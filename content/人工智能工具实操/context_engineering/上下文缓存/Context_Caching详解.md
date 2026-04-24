---
title: Context Caching详解
date: 2026-04-18
tags:
  - context-caching
  - cache
  - cost-optimization
  - anthropic
  - gemini
categories:
  - context-engineering
  - context-caching
---

> [!abstract] 摘要
> 上下文缓存是降低LLM使用成本的关键技术。本文深入讲解Anthropic Claude和Google Gemini的缓存机制、静态vs动态内容的缓存策略、成本优化原理（可节省90%成本）、以及缓存失效管理的最佳实践。

## 关键词速览

| 术语 | 英文 | 说明 |
|------|------|------|
| 上下文缓存 | Context Caching | 重复使用上下文的机制 |
| 缓存命中率 | Cache Hit Rate | 命中缓存的比例 |
| KV Cache | KV Cache | 注意力键值缓存 |
| 静态内容 | Static Content | 不变的上下文内容 |
| 动态内容 | Dynamic Content | 每次变化的内容 |
| 缓存成本 | Cache Cost | 缓存存储费用 |
| Token成本 | Token Cost | token使用费用 |
| 预填充 | Prefill | 缓存预热 |
| 失效策略 | Invalidation | 缓存失效策略 |
| TTL | Time To Live | 缓存存活时间 |

## 一、上下文缓存概述

### 1.1 为什么需要缓存

上下文缓存解决的核心问题：

```
无缓存场景：
┌──────────────────────────────────────────────────────┐
│ 请求1: [System] + [Base Knowledge] + [Query1] → 响应1│
│ 请求2: [System] + [Base Knowledge] + [Query2] → 响应2│
│ 请求3: [System] + [Base Knowledge] + [Query3] → 响应3│
│                                                      │
│ System + Base Knowledge 被重复传递 3次                │
│ 成本 = 3 × (System + Base Knowledge + Query) tokens   │
└──────────────────────────────────────────────────────┘

有缓存场景：
┌──────────────────────────────────────────────────────┐
│ 初始: [System] + [Base Knowledge] → 缓存（计算一次）   │
│ 请求1: [Query1] → 使用缓存 → 响应1                    │
│ 请求2: [Query2] → 使用缓存 → 响应2                    │
│ 请求3: [Query3] → 使用缓存 → 响应3                    │
│                                                      │
│ 成本 = 1 × (System + Base Knowledge) + 3 × Query      │
└──────────────────────────────────────────────────────┘
```

### 1.2 成本节省分析

| 场景 | 无缓存成本 | 有缓存成本 | 节省比例 |
|------|-----------|-----------|----------|
| 10次相似查询 | 10000 tokens | 1500 tokens | 85% |
| 100次查询 | 100000 tokens | 1050 tokens | 99% |
| 文档分析（50页） | 25000×50 tokens | 500 + 25000 tokens | 96% |

### 1.3 支持的模型和平台

| 平台 | 模型 | 缓存支持 | 最大缓存大小 |
|------|------|---------|-------------|
| Anthropic | Claude 3.5+ | ✓ | 200K tokens |
| Google | Gemini 1.5+ | ✓ | 1M tokens |
| OpenAI | GPT-4o | 部分 | 128K |
| Azure | GPT-4 | 即将支持 | - |

## 二、Anthropic Claude缓存

### 2.1 缓存机制

Claude使用`cache_control`参数实现上下文缓存：

```python
import anthropic

client = anthropic.Anthropic()

def claude_with_caching():
    """Claude上下文缓存示例"""
    
    # 1. 定义静态内容（缓存）
    system_prompt = """你是专业的技术文档助手。

核心职责：
1. 准确理解用户问题
2. 提供清晰、结构化的回答
3. 在回答中引用相关文档

回答原则：
- 只基于提供的上下文回答
- 不编造未提供的信息
- 不确定时明确说明"""

    # 基础知识（大量静态内容）
    base_knowledge = """[此文档包含大量技术文档内容...]"""
    
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=4096,
        system=[
            {
                "type": "text",
                "text": system_prompt,
                "cache_control": {"type": "ephemeral"}  # 临时缓存
            },
            {
                "type": "text",
                "text": base_knowledge,
                "cache_control": {"type": "automatic"}  # 自动缓存
            }
        ],
        messages=[
            {
                "role": "user",
                "content": "解释什么是REST API"
            }
        ]
    )
    
    return message.content
```

### 2.2 缓存类型

```python
class ClaudeCacheTypes:
    """Claude缓存类型"""
    
    # 临时缓存（ephemeral）
    # - 仅在当前请求中有效
    # - 请求结束后自动清除
    # - 适合：一次性使用的大量上下文
    
    EPHEMERAL_CACHE = {
        "cache_control": {"type": "ephemeral"}
    }
    
    # 自动缓存（automatic）
    # - 在多轮对话中保持
    # - 节省后续请求的成本
    # - 适合：基础知识和系统提示
    
    AUTOMATIC_CACHE = {
        "cache_control": {"type": "automatic"}
    }
```

### 2.3 完整使用示例

```python
import anthropic
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class CachedContext:
    """缓存上下文"""
    content: str
    cache_type: str  # 'ephemeral' or 'automatic'
    token_count: int

class ClaudeContextCache:
    """Claude上下文缓存管理器"""
    
    def __init__(self, model: str = "claude-3-5-sonnet-20241022"):
        self.client = anthropic.Anthropic()
        self.model = model
        self.cached_items: List[CachedContext] = []
    
    def add_static_context(
        self,
        content: str,
        cache_type: str = "automatic",
        description: str = ""
    ):
        """添加静态上下文"""
        tokens = self._estimate_tokens(content)
        
        self.cached_items.append(CachedContext(
            content=content,
            cache_type=cache_type,
            token_count=tokens
        ))
        
        return tokens
    
    def query(
        self,
        user_query: str,
        dynamic_context: str = None
    ):
        """
        使用缓存进行查询
        """
        # 构建系统部分（包含缓存标记）
        system_parts = []
        for item in self.cached_items:
            system_parts.append({
                "type": "text",
                "text": item.content,
                "cache_control": {"type": item.cache_type}
            })
        
        # 构建消息
        messages = []
        
        # 添加动态上下文（如果有）
        if dynamic_context:
            messages.append({
                "role": "user",
                "content": f"[参考信息]\n{dynamic_context}\n\n[问题]\n{user_query}"
            })
        else:
            messages.append({
                "role": "user",
                "content": user_query
            })
        
        # 调用API
        response = self.client.messages.create(
            model=self.model,
            max_tokens=4096,
            system=system_parts,
            messages=messages
        )
        
        return response.content
    
    def get_cache_stats(self) -> dict:
        """获取缓存统计"""
        total_tokens = sum(item.token_count for item in self.cached_items)
        
        return {
            "item_count": len(self.cached_items),
            "total_tokens": total_tokens,
            "items": [
                {
                    "cache_type": item.cache_type,
                    "tokens": item.token_count
                }
                for item in self.cached_items
            ]
        }
    
    @staticmethod
    def _estimate_tokens(text: str) -> int:
        """估算token数"""
        chinese = sum(1 for c in text if '\u4e00' <= c <= '\u9fff')
        english = len(text.split()) - chinese
        return int(chinese * 0.5 + english * 0.25 + len(text) * 0.1)


# 使用示例
def main():
    cache_manager = ClaudeContextCache()
    
    # 添加系统提示
    cache_manager.add_static_context(
        """你是一个专业的代码审查助手。
        
职责：
- 审查代码质量
- 发现潜在问题
- 提供改进建议

审查标准：
1. 代码可读性
2. 性能考虑
3. 安全最佳实践
4. 错误处理""",
        cache_type="automatic",
        description="System Prompt"
    )
    
    # 添加大量知识文档
    code_review_guide = open("code_review_guide.md").read()
    cache_manager.add_static_context(
        code_review_guide,
        cache_type="automatic",
        description="Code Review Guide"
    )
    
    # 查询
    response = cache_manager.query(
        "审查这段代码中的安全漏洞",
        dynamic_context="def get_user(id):\n    return db.query(id)"
    )
    
    print(response)
    
    # 查看统计
    stats = cache_manager.get_cache_stats()
    print(f"缓存统计: {stats}")


if __name__ == "__main__":
    main()
```

## 三、Google Gemini缓存

### 3.1 Gemini缓存机制

```python
import google.generativeai as genai

genai.configure(api_key="YOUR_API_KEY")

class GeminiContextCache:
    """Gemini上下文缓存管理器"""
    
    def __init__(self, model: str = "gemini-1.5-pro"):
        self.model_name = model
        self.cache_store = {}
    
    def create_cached_content(
        self,
        contents: List[str],
        ttl_minutes: int = 60
    ):
        """
        创建缓存内容
        
        Gemini使用CachedContent API
        """
        # 构建缓存内容
        cached_content = genai.caching.CachedContent.create(
            model=f"models/{self.model_name}",
            contents=[
                genai.types.Content(parts=[genai.types.Part(text=content)])
                for content in contents
            ],
            ttl=f"{ttl_minutes} minutes",
            display_name="my_cache"
        )
        
        return cached_content
    
    def query_with_cache(
        self,
        cached_content,
        query: str
    ):
        """使用缓存进行查询"""
        model = genai.GenerativeModel.from_cached_content(cached_content)
        
        response = model.generate_content(query)
        
        return response.text
    
    def batch_query(
        self,
        cached_content,
        queries: List[str]
    ):
        """批量查询"""
        model = genai.GenerativeModel.from_cached_content(cached_content)
        
        responses = []
        for query in queries:
            response = model.generate_content(query)
            responses.append(response.text)
        
        return responses
```

### 3.2 完整示例

```python
import google.generativeai as genai
from datetime import timedelta

def gemini_production_example():
    """Gemini生产环境示例"""
    
    # 1. 准备大量静态内容
    product_docs = """
    [产品文档内容...]
    [API参考文档...]
    [最佳实践指南...]
    """
    
    # 2. 创建缓存（最长7天）
    cached_content = genai.caching.CachedContent.create(
        model="models/gemini-1.5-pro",
        contents=[
            genai.types.Content(
                parts=[genai.types.Part(text=product_docs)]
            )
        ],
        ttl=timedelta(days=7),
        display_name="product_knowledge_base"
    )
    
    # 3. 多次查询（每次只需传递query）
    questions = [
        "如何创建产品？",
        "产品支持哪些支付方式？",
        "如何设置产品价格？",
        "如何处理退款请求？"
    ]
    
    model = genai.GenerativeModel.from_cached_content(cached_content)
    
    for question in questions:
        response = model.generate_content(question)
        print(f"Q: {question}")
        print(f"A: {response.text}\n")
    
    # 4. 查看缓存使用情况
    usage = cached_content.usage_metadata
    print(f"缓存使用统计:")
    print(f"  原始token: {usage.cached_content_token_count}")
    print(f"  提示token: {usage.prompt_token_count}")
    print(f"  生成token: {usage.candidates_token_count}")
```

## 四、缓存策略设计

### 4.1 静态vs动态内容

```python
class CacheStrategy:
    """缓存策略设计"""
    
    # 静态内容：适合缓存
    STATIC_CONTENT_TYPES = {
        'system_prompt': {
            'cache': True,
            'ttl': timedelta(days=30),  # 长期有效
            'refresh': 'manual'
        },
        'base_knowledge': {
            'cache': True,
            'ttl': timedelta(days=7),
            'refresh': 'daily'
        },
        'product_docs': {
            'cache': True,
            'ttl': timedelta(days=7),
            'refresh': 'weekly'
        },
        'api_reference': {
            'cache': True,
            'ttl': timedelta(hours=24),
            'refresh': 'on_update'
        }
    }
    
    # 动态内容：不缓存
    DYNAMIC_CONTENT_TYPES = {
        'user_query': {'cache': False},
        'session_context': {'cache': False},
        'user_preferences': {'cache': True, 'short_ttl': True},
        'recent_history': {'cache': True, 'short_ttl': True}
    }
    
    @classmethod
    def should_cache(cls, content_type: str) -> bool:
        """判断是否应该缓存"""
        if content_type in cls.STATIC_CONTENT_TYPES:
            return cls.STATIC_CONTENT_TYPES[content_type]['cache']
        if content_type in cls.DYNAMIC_CONTENT_TYPES:
            return cls.DYNAMIC_CONTENT_TYPES[content_type]['cache']
        return False
```

### 4.2 缓存预热

```python
class CacheWarmup:
    """缓存预热策略"""
    
    def __init__(self, cache_manager):
        self.cache_manager = cache_manager
    
    def warmup_common_queries(
        self,
        static_contents: List[str],
        common_queries: List[str]
    ):
        """
        预热缓存
        
        预先计算常见查询的上下文
        """
        # 1. 准备缓存内容
        for content in static_contents:
            self.cache_manager.add_static_context(content)
        
        # 2. 预填充（如果支持）
        # 预先执行常见查询的上下文计算
        for query in common_queries[:5]:  # 预热前5个
            # 触发上下文加载，但不返回结果
            pass
    
    def adaptive_warmup(
        self,
        query_patterns: List[str],
        min_frequency: int = 10
    ):
        """
        自适应预热
        
        根据查询频率自动预热
        """
        # 分析查询模式
        query_counts = {}
        for pattern in query_patterns:
            query_counts[pattern] = query_counts.get(pattern, 0) + 1
        
        # 预热高频查询
        hot_queries = [
            q for q, count in query_counts.items()
            if count >= min_frequency
        ]
        
        # 返回需要预热的内容
        return hot_queries
```

### 4.3 成本优化策略

```python
class CacheCostOptimizer:
    """缓存成本优化器"""
    
    def __init__(self):
        # 价格配置（示例）
        self.pricing = {
            'anthropic': {
                'cache_hit_discount': 0.9,  # 缓存命中90%折扣
                'cache_storage': 0.00003,  # 每1K token每小时
            },
            'gemini': {
                'cache_hit_discount': 0.5,  # 缓存命中50%折扣
                'cache_storage': 0.00001,  # 每1M token每小时
            }
        }
    
    def calculate_savings(
        self,
        provider: str,
        original_tokens: int,
        query_count: int,
        cached_tokens: int,
        cache_storage_hours: int = 1
    ) -> dict:
        """
        计算成本节省
        
        Args:
            provider: 'anthropic' or 'gemini'
            original_tokens: 原始上下文token数
            query_count: 查询次数
            cached_tokens: 缓存的token数
            cache_storage_hours: 缓存存储时长
        """
        pricing = self.pricing[provider]
        
        # 无缓存成本
        no_cache_cost = original_tokens * query_count
        
        # 有缓存成本
        cache_input_cost = cached_tokens * pricing['cache_hit_discount']
        cache_storage_cost = (
            cached_tokens * 
            cache_storage_hours * 
            pricing['cache_storage']
        )
        uncached_cost = (original_tokens - cached_tokens) * query_count
        
        with_cache_cost = (
            cache_input_cost * query_count + 
            uncached_cost + 
            cache_storage_cost
        )
        
        # 计算节省
        savings = no_cache_cost - with_cache_cost
        savings_percent = (savings / no_cache_cost) * 100
        
        return {
            'no_cache_cost': no_cache_cost,
            'with_cache_cost': with_cache_cost,
            'savings': savings,
            'savings_percent': savings_percent,
            'break_even_queries': self._calculate_break_even(
                cached_tokens, 
                cache_storage_cost,
                original_tokens,
                pricing['cache_hit_discount']
            )
        }
    
    def _calculate_break_even(
        self,
        cached_tokens: int,
        cache_storage_cost: float,
        original_tokens: int,
        discount: float
    ) -> int:
        """计算盈亏平衡点"""
        # 节省per query = (original - cached * discount)
        # 固定成本 = cache_storage_cost
        # n * savings_per_query > cache_storage_cost
        savings_per_query = original_tokens - cached_tokens * discount
        
        if savings_per_query <= 0:
            return float('inf')
        
        return int(cache_storage_cost / savings_per_query) + 1
    
    def optimize_cache_size(
        self,
        token_budget: int,
        query_frequency: int,
        cache_ttl_hours: int
    ) -> dict:
        """
        优化缓存大小
        
        决定缓存多少内容最经济
        """
        # 缓存越多，每次查询越便宜
        # 但缓存本身也有成本
        
        recommendations = []
        
        for cache_ratio in [0.3, 0.5, 0.7, 0.9]:
            cached_tokens = int(token_budget * cache_ratio)
            uncached_tokens = token_budget - cached_tokens
            
            # 估算成本
            cost_per_query = (
                cached_tokens * cache_ttl_hours * 0.00001 +
                uncached_tokens
            )
            
            total_cost = cost_per_query * query_frequency
            
            recommendations.append({
                'cache_ratio': cache_ratio,
                'cached_tokens': cached_tokens,
                'cost_per_query': cost_per_query,
                'total_cost': total_cost
            })
        
        # 选择最优
        best = min(recommendations, key=lambda x: x['total_cost'])
        
        return {
            'recommendations': recommendations,
            'optimal': best
        }
```

## 五、缓存失效管理

### 5.1 失效策略

```python
from enum import Enum
from datetime import datetime, timedelta

class InvalidationStrategy(Enum):
    TTL = "time_to_live"           # 基于时间
    LRU = "least_recently_used"    # 最近最少使用
    LFU = "least_frequently_used"   # 最不常用
    MANUAL = "manual"              # 手动失效
    VERSION = "version"            # 版本控制

class CacheInvalidator:
    """缓存失效管理器"""
    
    def __init__(self, strategy: InvalidationStrategy = InvalidationStrategy.TTL):
        self.strategy = strategy
        self.cache_entries = {}
        self.access_log = {}
        self.access_count = {}
    
    def register_cache(
        self,
        cache_id: str,
        content: str,
        ttl: timedelta = None,
        version: int = 1
    ):
        """注册缓存条目"""
        self.cache_entries[cache_id] = {
            'content': content,
            'created_at': datetime.now(),
            'ttl': ttl,
            'version': version,
            'is_valid': True
        }
        self.access_log[cache_id] = datetime.now()
        self.access_count[cache_id] = 0
    
    def access(self, cache_id: str):
        """记录缓存访问"""
        self.access_log[cache_id] = datetime.now()
        self.access_count[cache_id] = self.access_count.get(cache_id, 0) + 1
    
    def is_valid(self, cache_id: str) -> bool:
        """检查缓存是否有效"""
        if cache_id not in self.cache_entries:
            return False
        
        entry = self.cache_entries[cache_id]
        
        if not entry['is_valid']:
            return False
        
        # TTL检查
        if entry['ttl']:
            age = datetime.now() - entry['created_at']
            if age > entry['ttl']:
                return False
        
        return True
    
    def invalidate(self, cache_id: str):
        """手动失效"""
        if cache_id in self.cache_entries:
            self.cache_entries[cache_id]['is_valid'] = False
    
    def invalidate_by_pattern(self, pattern: str):
        """按模式失效"""
        for cache_id in self.cache_entries:
            if pattern in cache_id:
                self.invalidate(cache_id)
    
    def update_version(self, cache_id: str):
        """更新版本（触发失效）"""
        if cache_id in self.cache_entries:
            self.cache_entries[cache_id]['version'] += 1
            self.cache_entries[cache_id]['is_valid'] = False
    
    def cleanup_invalid(self):
        """清理无效缓存"""
        invalid_ids = [
            cid for cid in self.cache_entries
            if not self.is_valid(cid)
        ]
        
        for cid in invalid_ids:
            del self.cache_entries[cid]
            if cid in self.access_log:
                del self.access_log[cid]
            if cid in self.access_count:
                del self.access_count[cid]
        
        return len(invalid_ids)
```

### 5.2 自动刷新机制

```python
class CacheAutoRefresh:
    """缓存自动刷新"""
    
    def __init__(self, cache_manager):
        self.cache_manager = cache_manager
        self.refresh_triggers = {}
    
    def schedule_refresh(
        self,
        cache_id: str,
        refresh_fn: callable,
        interval: timedelta
    ):
        """调度刷新任务"""
        self.refresh_triggers[cache_id] = {
            'fn': refresh_fn,
            'interval': interval,
            'last_refresh': datetime.now()
        }
    
    def check_and_refresh(self, cache_id: str):
        """检查并刷新"""
        if cache_id not in self.refresh_triggers:
            return False
        
        trigger = self.refresh_triggers[cache_id]
        elapsed = datetime.now() - trigger['last_refresh']
        
        if elapsed > trigger['interval']:
            # 执行刷新
            new_content = trigger['fn']()
            
            # 更新缓存
            self.cache_manager.update_content(cache_id, new_content)
            
            # 更新刷新时间
            trigger['last_refresh'] = datetime.now()
            
            return True
        
        return False
    
    def refresh_all_due(self):
        """刷新所有到期的缓存"""
        refreshed = []
        
        for cache_id in list(self.refresh_triggers.keys()):
            if self.check_and_refresh(cache_id):
                refreshed.append(cache_id)
        
        return refreshed
```

## 六、生产环境最佳实践

### 6.1 完整缓存架构

```python
class ProductionCacheSystem:
    """生产级缓存系统"""
    
    def __init__(
        self,
        provider: str = "anthropic",
        model: str = "claude-3-5-sonnet-20241022"
    ):
        self.provider = provider
        self.model = model
        self.cache_store = {}
        self.invalidator = CacheInvalidator()
        self.cost_optimizer = CacheCostOptimizer()
        self.stats = {
            'cache_hits': 0,
            'cache_misses': 0,
            'total_cost_saved': 0
        }
    
    def setup_knowledge_base(
        self,
        documents: List[Dict[str, str]]
    ):
        """
        设置知识库缓存
        
        documents: [{'id': 'doc1', 'content': '...', 'type': 'static'}]
        """
        for doc in documents:
            if doc.get('type') == 'static':
                tokens = self._estimate_tokens(doc['content'])
                
                self.cache_store[doc['id']] = {
                    'content': doc['content'],
                    'tokens': tokens,
                    'type': 'static',
                    'last_used': datetime.now()
                }
                
                # 设置失效策略
                ttl = self._get_ttl_for_doc_type(doc.get('doc_type', 'general'))
                self.invalidator.register_cache(
                    doc['id'],
                    doc['content'],
                    ttl=ttl
                )
    
    def query(
        self,
        question: str,
        dynamic_context: str = None
    ) -> str:
        """执行查询"""
        # 检查哪些缓存命中
        cached_contents = []
        for cache_id, cache_data in self.cache_store.items():
            if self.invalidator.is_valid(cache_id):
                self.invalidator.access(cache_id)
                cached_contents.append(cache_data['content'])
                cache_data['last_used'] = datetime.now()
                self.stats['cache_hits'] += 1
            else:
                self.stats['cache_misses'] += 1
        
        # 构建查询
        if self.provider == "anthropic":
            return self._query_anthropic(question, cached_contents, dynamic_context)
        else:
            return self._query_gemini(question, cached_contents, dynamic_context)
    
    def get_stats(self) -> dict:
        """获取缓存统计"""
        total_cached_tokens = sum(
            c['tokens'] for c in self.cache_store.values()
        )
        
        hit_rate = (
            self.stats['cache_hits'] / 
            max(1, self.stats['cache_hits'] + self.stats['cache_misses'])
        )
        
        return {
            'cache_size': len(self.cache_store),
            'total_tokens': total_cached_tokens,
            'cache_hits': self.stats['cache_hits'],
            'cache_misses': self.stats['cache_misses'],
            'hit_rate': f"{hit_rate:.1%}",
            'estimated_savings': self._estimate_savings()
        }
    
    def _get_ttl_for_doc_type(self, doc_type: str) -> timedelta:
        """根据文档类型获取TTL"""
        ttls = {
            'system': timedelta(days=30),
            'knowledge_base': timedelta(days=7),
            'documentation': timedelta(hours=24),
            'session': timedelta(hours=1)
        }
        return ttls.get(doc_type, timedelta(days=1))
    
    @staticmethod
    def _estimate_tokens(text: str) -> int:
        chinese = sum(1 for c in text if '\u4e00' <= c <= '\u9fff')
        english = len(text.split()) - chinese
        return int(chinese * 0.5 + english * 0.25)
```

### 6.2 监控和告警

```python
class CacheMonitor:
    """缓存监控"""
    
    def __init__(self, cache_system: ProductionCacheSystem):
        self.cache_system = cache_system
        self.alert_thresholds = {
            'hit_rate_low': 0.7,      # 命中率低于70%告警
            'cache_size_high': 150000, # 缓存超过150K token告警
            'cost_high': 100          # 日成本超过100美元告警
        }
    
    def check_health(self) -> dict:
        """健康检查"""
        stats = self.cache_system.get_stats()
        
        alerts = []
        
        # 命中率检查
        hit_rate = float(stats['hit_rate'].rstrip('%')) / 100
        if hit_rate < self.alert_thresholds['hit_rate_low']:
            alerts.append({
                'type': 'hit_rate_low',
                'message': f"缓存命中率 {stats['hit_rate']} 低于阈值",
                'severity': 'warning'
            })
        
        # 缓存大小检查
        if stats['total_tokens'] > self.alert_thresholds['cache_size_high']:
            alerts.append({
                'type': 'cache_size_high',
                'message': f"缓存大小 {stats['total_tokens']} 超过阈值",
                'severity': 'info'
            })
        
        return {
            'healthy': len(alerts) == 0,
            'stats': stats,
            'alerts': alerts
        }
```

## 七、相关主题

- [[上下文窗口深度解析]]
- [[上下文压缩技术]]
- [[RAG上下文优化指南]]
- [[对话历史管理]]
- [[Long Context Model详解]]

## 八、参考文献

1. Anthropic (2024). Claude API Documentation - Caching.
2. Google (2024). Gemini API Documentation - Cached Content.
3. Wu, K., et al. (2024). A Study of Context Caching in LLM Serving Systems.
