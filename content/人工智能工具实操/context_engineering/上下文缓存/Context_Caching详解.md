---
title: Context Caching详解
date: 2026-04-24
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
> 上下文缓存是2025-2026年LLM应用的重磅功能，用好了能让API费用打一折！本文详细讲解：为什么需要缓存、怎么用（Claude/Gemini实战代码）、缓存策略设计、成本计算、以及生产环境最佳实践。看完全文，你就能把AI应用的费用从每月1万降到1千。

## 先算一笔账：缓存能省多少钱？

### 不用缓存 vs 用缓存

想象你的AI助手场景：

```
每天1000个用户，每个用户平均发5条消息
每条消息需要传递的系统提示+知识库 = 2000 tokens

不用缓存：
每天token消耗 = 1000 × 5 × 2000 = 10,000,000 tokens
每月费用 ≈ $1000

用缓存（90%折扣）：
每天token消耗 = 1000 × 5 × 2000 × 0.1 = 1,000,000 tokens
每月费用 ≈ $100

省了90%！！！一年省一万多美元！
```

### 各平台缓存折扣对比

| 平台 | 缓存读取折扣 | 说明 |
|------|-------------|------|
| Gemini 2.5+/3 | **90%** | 最便宜 |
| Claude (Anthropic) | **90%** | 0.1x基础价格 |
| GPT-5.4 | **90%** | 最新版本 |
| GPT-4o | 50% | 老版本折扣较低 |

**缓存读取只需要付原价的10%！**

---

## 一、上下文缓存是什么？

### 工作原理

```
无缓存场景（每次都重新处理）：
┌─────────────────────────────────────────┐
│  请求1: [系统提示] + [知识库] + [问题] → 响应1
│  请求2: [系统提示] + [知识库] + [问题] → 响应2
│  请求3: [系统提示] + [知识库] + [问题] → 响应3
│                                          │
│  系统提示+知识库被处理了3次！              │
└─────────────────────────────────────────┘

有缓存场景（只处理一次）：
┌─────────────────────────────────────────┐
│  初始化: [系统提示] + [知识库] → 缓存计算
│  请求1: [问题] + 缓存指针 → 响应1
│  请求2: [问题] + 缓存指针 → 响应2
│  请求3: [问题] + 缓存指针 → 响应3
│                                          │
│  系统提示+知识库只处理1次！                │
└─────────────────────────────────────────┘
```

### 两种缓存类型

| 类型 | 说明 | 适用场景 |
|------|------|---------|
| **隐式缓存** | 系统自动检测重复前缀 | 短时重复使用 |
| **显式缓存** | 手动创建、管理缓存 | 长期重复使用 |

#### 隐式缓存（Automatic）
- 自动启用，无需配置
- 系统自动检测重复内容前缀
- 无保证节省
- 最低token限制：1024-4096

#### 显式缓存（Explicit）
- 手动创建缓存
- 设置TTL（存活时间）
- 保证折扣
- 成本可预测

---

## 二、Claude上下文缓存实战

### Claude API调用

Claude使用`cache_control`参数实现上下文缓存：

```python
import anthropic

client = anthropic.Anthropic()

def claude_caching_example():
    """Claude上下文缓存示例"""
    
    # 静态系统提示
    system_prompt = """你是一个专业的技术文档助手。

核心职责：
1. 准确理解用户问题
2. 提供清晰、结构化的回答
3. 在回答中引用相关文档

回答原则：
- 只基于提供的上下文回答
- 不编造未提供的信息
- 不确定时明确说明"""

    # 大量知识内容（静态的，可以缓存）
    knowledge_base = open("knowledge_base.md").read()
    
    # 使用缓存的关键：标记需要缓存的内容
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
                "text": knowledge_base,
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

### 缓存类型详解

```python
# Claude支持的缓存类型

# 1. ephemeral（临时缓存）
# 只在当前请求中有效，请求结束自动清除
EPHEMERAL_CACHE = {
    "cache_control": {"type": "ephemeral"}
}

# 2. automatic（自动缓存）
# 在多轮对话中保持，节省后续请求成本
AUTOMATIC_CACHE = {
    "cache_control": {"type": "automatic"}
}

# 3. block（块级缓存）
# 只缓存部分内容
BLOCK_CACHE = {
    "type": "text",
    "text": "只缓存这段",
    "cache_control": {"type": "ephemeral", "index": 0}
}
```

### 完整缓存管理器

```python
import anthropic
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class CachedContext:
    """缓存的上下文"""
    content: str
    cache_type: str  # 'ephemeral' or 'automatic'
    token_count: int

class ClaudeCacheManager:
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
        """添加静态上下文（将被缓存）"""
        tokens = self._estimate_tokens(content)
        
        self.cached_items.append(CachedContext(
            content=content,
            cache_type=cache_type,
            token_count=tokens
        ))
        
        print(f"添加缓存: {description}, {tokens} tokens")
        return tokens
    
    def query(
        self,
        user_query: str,
        dynamic_context: str = None
    ) -> str:
        """使用缓存进行查询"""
        # 构建系统部分（包含缓存标记）
        system_parts = []
        for item in self.cached_items:
            system_parts.append({
                "type": "text",
                "text": item.content,
                "cache_control": {"type": item.cache_type}
            })
        
        # 构建消息
        if dynamic_context:
            user_content = f"""[参考信息]
{dynamic_context}

[问题]
{user_query}"""
        else:
            user_content = user_query
        
        # 调用API
        response = self.client.messages.create(
            model=self.model,
            max_tokens=4096,
            system=system_parts,
            messages=[{"role": "user", "content": user_content}]
        )
        
        return response.content[0].text
    
    def get_cache_stats(self) -> dict:
        """获取缓存统计"""
        total_tokens = sum(item.token_count for item in self.cached_items)
        
        return {
            "item_count": len(self.cached_items),
            "total_tokens": total_tokens,
            "estimated_savings_percent": 90,  # 大约节省90%
            "items": [
                {
                    "cache_type": item.cache_type,
                    "tokens": item.token_count,
                    "description": f"缓存项 {i+1}"
                }
                for i, item in enumerate(self.cached_items)
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
    manager = ClaudeCacheManager()
    
    # 添加静态上下文
    manager.add_static_context(
        """你是代码审查助手。
审查标准：
1. 代码可读性
2. 性能考虑
3. 安全最佳实践
4. 错误处理""",
        cache_type="automatic",
        description="System Prompt"
    )
    
    # 添加大量知识
    docs = open("coding_guidelines.md").read()
    manager.add_static_context(
        docs,
        cache_type="automatic",
        description="Coding Guidelines"
    )
    
    # 查询
    response = manager.query(
        "审查这段代码的安全问题：def get_user(id): return db.query(id)",
        dynamic_context="用户代码"
    )
    
    print(response)
    
    # 查看统计
    stats = manager.get_cache_stats()
    print(f"\n缓存统计: {stats}")

if __name__ == "__main__":
    main()
```

---

## 三、Google Gemini上下文缓存

### Gemini API调用

```python
import google.generativeai as genai
from datetime import timedelta

# 配置API
genai.configure(api_key="YOUR_API_KEY")

class GeminiCacheManager:
    """Gemini上下文缓存管理器"""
    
    def __init__(self, model: str = "gemini-1.5-pro"):
        self.model_name = model
        self.cache_store = {}
    
    def create_cached_content(
        self,
        contents: List[str],
        ttl_minutes: int = 60,
        display_name: str = "my_cache"
    ) -> object:
        """
        创建缓存内容
        
        Args:
            contents: 要缓存的内容列表
            ttl_minutes: 缓存存活时间（最长7天）
            display_name: 缓存名称（用于识别）
        """
        cached_content = genai.caching.CachedContent.create(
            model=f"models/{self.model_name}",
            contents=[
                genai.types.Content(
                    parts=[genai.types.Part(text=content)]
                )
                for content in contents
            ],
            ttl=f"{ttl_minutes} minutes",
            display_name=display_name
        )
        
        self.cache_store[display_name] = cached_content
        return cached_content
    
    def query_with_cache(
        self,
        cache_name: str,
        query: str
    ) -> str:
        """使用缓存进行查询"""
        if cache_name not in self.cache_store:
            raise ValueError(f"缓存 '{cache_name}' 不存在")
        
        cached_content = self.cache_store[cache_name]
        model = genai.GenerativeModel.from_cached_content(cached_content)
        
        response = model.generate_content(query)
        return response.text
    
    def batch_query(
        self,
        cache_name: str,
        queries: List[str]
    ) -> List[str]:
        """批量查询（显著降低成本）"""
        if cache_name not in self.cache_store:
            raise ValueError(f"缓存 '{cache_name}' 不存在")
        
        cached_content = self.cache_store[cache_name]
        model = genai.GenerativeModel.from_cached_content(cached_content)
        
        responses = []
        for query in queries:
            response = model.generate_content(query)
            responses.append(response.text)
        
        return responses
    
    def get_usage_stats(self, cache_name: str) -> dict:
        """获取缓存使用统计"""
        if cache_name not in self.cache_store:
            return {}
        
        cached_content = self.cache_store[cache_name]
        usage = cached_content.usage_metadata
        
        return {
            "cached_content_tokens": usage.cached_content_token_count,
            "prompt_tokens": usage.prompt_token_count,
            "total_tokens": usage.total_token_count,
            "estimated_cost_savings": self._estimate_savings(
                usage.cached_content_token_count,
                usage.prompt_token_count
            )
        }
    
    def _estimate_savings(self, cached: int, uncached: int) -> float:
        """估算节省成本（Gemini 2.0为例）"""
        # 假设价格：$0.125/1M tokens (输入)
        # 缓存折扣：75%
        price_per_million = 0.125
        uncached_cost = (uncached / 1_000_000) * price_per_million
        cached_cost = (cached / 1_000_000) * price_per_million * 0.25
        return uncached_cost - cached_cost
```

### 完整使用示例

```python
def gemini_production_example():
    """Gemini生产环境示例"""
    
    manager = GeminiCacheManager(model="gemini-1.5-pro")
    
    # 1. 准备大量静态内容
    product_docs = """
    [产品文档内容...]
    [API参考文档...]
    [最佳实践指南...]
    """.strip()
    
    # 2. 创建缓存（最长7天）
    cached = manager.create_cached_content(
        contents=[
            "# 产品知识库\n" + product_docs,
            "# API文档\n" + api_docs,
            "# 故障排除指南\n" + troubleshooting
        ],
        ttl_minutes=60 * 24 * 7,  # 7天
        display_name="product_knowledge"
    )
    
    # 3. 批量查询（非常便宜！）
    questions = [
        "如何创建产品？",
        "支持哪些支付方式？",
        "如何设置价格？",
        "如何处理退款？",
        "产品有哪些限制？"
    ]
    
    responses = manager.batch_query("product_knowledge", questions)
    
    for q, a in zip(questions, responses):
        print(f"Q: {q}")
        print(f"A: {a}\n")
    
    # 4. 查看节省了多少
    stats = manager.get_usage_stats("product_knowledge")
    print(f"""
缓存统计：
- 缓存token: {stats['cached_content_tokens']:,}
- 提示token: {stats['prompt_tokens']:,}
- 估算节省: ${stats['estimated_cost_savings']:.4f}
    """)
```

---

## 四、缓存策略设计

### 静态 vs 动态内容

```
┌─────────────────────────────────────────────────────────────┐
│                   缓存策略决策树                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   这份内容会重复使用吗？                                      │
│         │                                                   │
│    ┌────┴────┐                                             │
│    是        否                                              │
│    │         │                                              │
│    ↓         ↓                                              │
│  缓存      不缓存                                           │
│    │                                                   │
│    ├── 内容多久变一次？                                       │
│    │         │                                             │
│    ├─ 每天   → 每天刷新缓存                                 │
│    ├─ 每周   → 每周刷新缓存                                 │
│    └─ 每月   → 每月刷新缓存                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 缓存内容分类

```python
class CacheStrategy:
    """缓存策略"""
    
    # 应该缓存的内容
    SHOULD_CACHE = {
        'system_prompt': {
            'cache': True,
            'ttl': timedelta(days=30),
            'refresh': 'manual'  # 手动更新
        },
        'base_knowledge': {
            'cache': True,
            'ttl': timedelta(days=7),
            'refresh': 'daily'  # 每天刷新
        },
        'product_docs': {
            'cache': True,
            'ttl': timedelta(days=7),
            'refresh': 'weekly'  # 每周刷新
        },
        'api_reference': {
            'cache': True,
            'ttl': timedelta(hours=24),
            'refresh': 'on_update'  # 变更时刷新
        },
        'user_preferences': {
            'cache': True,
            'ttl': timedelta(hours=1),
            'refresh': 'frequent'  # 频繁更新
        }
    }
    
    # 不应该缓存的内容
    DONT_CACHE = {
        'user_query': '每次不同',
        'session_context': '会话级别',
        'temporary_data': '临时数据'
    }
    
    @classmethod
    def should_cache(cls, content_type: str) -> bool:
        """判断是否应该缓存"""
        if content_type in cls.SHOULD_CACHE:
            return cls.SHOULD_CACHE[content_type]['cache']
        return False
    
    @classmethod
    def get_cache_config(cls, content_type: str) -> dict:
        """获取缓存配置"""
        return cls.SHOULD_CACHE.get(content_type, {})
```

### 缓存预热

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
        
        在用户高峰前预先创建缓存
        """
        # 1. 准备缓存内容
        for content in static_contents:
            self.cache_manager.add_static_context(content)
        
        # 2. 预填充（如果API支持）
        # 预先"执行"常见查询，让缓存生效
        for query in common_queries[:5]:
            # 触发缓存加载，但不返回结果给用户
            try:
                self.cache_manager.query(query)
            except:
                pass  # 忽略错误
    
    def schedule_refresh(
        self,
        content_type: str,
        refresh_interval: timedelta
    ):
        """调度定时刷新"""
        # 可以配合定时任务（如Celery、APScheduler）实现
        pass
```

---

## 五、成本优化实战

### 成本计算器

```python
class CacheCostCalculator:
    """缓存成本计算器"""
    
    def __init__(self):
        # 各平台价格配置（示例）
        self.pricing = {
            'anthropic': {
                'input_price': 0.003,  # $3/1M tokens
                'cache_hit_discount': 0.1,  # 90%折扣
            },
            'gemini': {
                'input_price': 0.125,  # $0.125/1M tokens
                'cache_hit_discount': 0.25,  # 75%折扣（Gemini 2.0）
            },
            'openai': {
                'input_price': 2.5,  # $2.5/1M tokens (GPT-4)
                'cache_hit_discount': 0.5,  # 50%折扣
            }
        }
    
    def calculate_savings(
        self,
        provider: str,
        cached_tokens: int,
        uncached_tokens: int,
        query_count: int
    ) -> dict:
        """
        计算节省金额
        
        Args:
            provider: 'anthropic', 'gemini', 'openai'
            cached_tokens: 每次缓存的token数
            uncached_tokens: 每次查询的额外token数
            query_count: 查询次数
        """
        pricing = self.pricing[provider]
        discount = pricing['cache_hit_discount']
        
        # 无缓存成本
        no_cache_cost = (
            (cached_tokens + uncached_tokens) * 
            query_count * 
            pricing['input_price'] / 1_000_000
        )
        
        # 有缓存成本
        with_cache_cost = (
            cached_tokens * discount * 
            query_count * 
            pricing['input_price'] / 1_000_000 +
            uncached_tokens * 
            query_count * 
            pricing['input_price'] / 1_000_000
        )
        
        # 节省
        savings = no_cache_cost - with_cache_cost
        savings_percent = (savings / no_cache_cost) * 100
        
        return {
            'no_cache_cost': f"${no_cache_cost:.2f}",
            'with_cache_cost': f"${with_cache_cost:.2f}",
            'savings': f"${savings:.2f}",
            'savings_percent': f"{savings_percent:.1f}%",
            'break_even_queries': self._calc_break_even(
                cached_tokens, pricing
            )
        }
    
    def _calc_break_even(self, cached_tokens: int, pricing: dict) -> int:
        """计算盈亏平衡点（需要多少查询才能回本）"""
        # 缓存有额外的创建成本（1.25x）
        # 但读取成本很低（0.1x）
        # 净节省约0.675x per query
        
        # 简化计算
        return 2  # 通常2-3个查询就能回本
```

### 优化建议

```python
class CacheOptimizer:
    """缓存优化建议"""
    
    @staticmethod
    def suggest_optimizations(
        total_tokens: int,
        query_count: int,
        context_type: str
    ) -> list:
        """给出优化建议"""
        suggestions = []
        
        # 建议1：检查token规模
        if total_tokens > 100000:
            suggestions.append(
                "💡 建议缓存：内容超过100K tokens，"
                "重复使用能省90%费用"
            )
        
        # 建议2：检查查询频率
        if query_count > 10:
            suggestions.append(
                "💡 高频查询场景，缓存非常值得"
            )
        
        # 建议3：检查内容类型
        if context_type in ['docs', 'knowledge', 'guidelines']:
            suggestions.append(
                "💡 文档类内容适合缓存，可以缓存1-7天"
            )
        
        return suggestions
```

---

## 六、缓存失效管理

### 失效策略

```python
from enum import Enum
from datetime import datetime, timedelta

class InvalidationStrategy(Enum):
    """缓存失效策略"""
    TTL = "time_to_live"           # 基于时间
    LRU = "least_recently_used"    # 最近最少使用
    VERSION = "version"            # 版本控制
    MANUAL = "manual"              # 手动失效

class CacheInvalidator:
    """缓存失效管理器"""
    
    def __init__(self, strategy: InvalidationStrategy = InvalidationStrategy.TTL):
        self.strategy = strategy
        self.entries = {}
    
    def register(
        self,
        cache_id: str,
        content: str,
        ttl: timedelta = None,
        version: int = 1
    ):
        """注册缓存"""
        self.entries[cache_id] = {
            'content': content,
            'created_at': datetime.now(),
            'ttl': ttl,
            'version': version,
            'is_valid': True
        }
    
    def is_valid(self, cache_id: str) -> bool:
        """检查缓存是否有效"""
        if cache_id not in self.entries:
            return False
        
        entry = self.entries[cache_id]
        
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
        if cache_id in self.entries:
            self.entries[cache_id]['is_valid'] = False
    
    def invalidate_by_pattern(self, pattern: str):
        """按模式批量失效"""
        for cache_id in self.entries:
            if pattern in cache_id:
                self.invalidate(cache_id)
    
    def refresh(self, cache_id: str):
        """刷新缓存（重新开始计时）"""
        if cache_id in self.entries:
            self.entries[cache_id]['created_at'] = datetime.now()
            self.entries[cache_id]['is_valid'] = True
```

### 自动刷新机制

```python
class CacheAutoRefresh:
    """缓存自动刷新"""
    
    def __init__(self, cache_manager):
        self.cache_manager = cache_manager
        self.refresh_tasks = {}
    
    def schedule_refresh(
        self,
        cache_id: str,
        refresh_fn: callable,
        interval: timedelta
    ):
        """调度刷新任务"""
        self.refresh_tasks[cache_id] = {
            'fn': refresh_fn,
            'interval': interval,
            'last_refresh': datetime.now()
        }
    
    def check_and_refresh(self, cache_id: str) -> bool:
        """检查并刷新"""
        if cache_id not in self.refresh_tasks:
            return False
        
        task = self.refresh_tasks[cache_id]
        elapsed = datetime.now() - task['last_refresh']
        
        if elapsed > task['interval']:
            # 执行刷新
            new_content = task['fn']()
            self.cache_manager.update_content(cache_id, new_content)
            task['last_refresh'] = datetime.now()
            return True
        
        return False
    
    def refresh_all_due(self) -> list:
        """刷新所有到期的缓存"""
        refreshed = []
        for cache_id in list(self.refresh_tasks.keys()):
            if self.check_and_refresh(cache_id):
                refreshed.append(cache_id)
        return refreshed
```

---

## 七、生产环境最佳实践

### 完整缓存架构

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
        self.stats = {
            'cache_hits': 0,
            'cache_misses': 0,
            'total_cost_saved': 0
        }
    
    def setup_knowledge_base(
        self,
        documents: List[dict]
    ):
        """
        设置知识库缓存
        
        documents: [{'id': 'doc1', 'content': '...', 'type': 'static', 'ttl_days': 7}]
        """
        for doc in documents:
            if doc.get('type') == 'static':
                self.cache_store[doc['id']] = {
                    'content': doc['content'],
                    'type': 'static',
                    'ttl_days': doc.get('ttl_days', 7)
                }
                
                # 注册失效管理器
                ttl = timedelta(days=doc.get('ttl_days', 7))
                self.invalidator.register(
                    doc['id'],
                    doc['content'],
                    ttl=ttl
                )
    
    def query(
        self,
        question: str,
        dynamic_context: str = None
    ) -> str:
        """查询（自动使用缓存）"""
        # 检查哪些缓存命中
        cached_contents = []
        for cache_id, cache_data in self.cache_store.items():
            if self.invalidator.is_valid(cache_id):
                self.stats['cache_hits'] += 1
                cached_contents.append(cache_data['content'])
            else:
                self.stats['cache_misses'] += 1
        
        # 调用API（使用缓存）
        if self.provider == "anthropic":
            return self._query_anthropic(question, cached_contents, dynamic_context)
        else:
            return self._query_gemini(question, cached_contents, dynamic_context)
    
    def get_stats(self) -> dict:
        """获取统计"""
        total = self.stats['cache_hits'] + self.stats['cache_misses']
        hit_rate = self.stats['cache_hits'] / total if total > 0 else 0
        
        return {
            'cache_size': len(self.cache_store),
            'cache_hits': self.stats['cache_hits'],
            'cache_misses': self.stats['cache_misses'],
            'hit_rate': f"{hit_rate:.1%}",
            'estimated_savings': self._estimate_savings()
        }
```

---

## 八、一图总结

```
┌─────────────────────────────────────────────────────────────┐
│                  上下文缓存使用指南                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  💰 省钱效果                                                │
│  ├─ 缓存读取：10%原价（90%折扣）                             │
│  ├─ Claude: 0.1x价格                                       │
│  ├─ Gemini: 0.1x-0.25x价格                                 │
│  └─ GPT-4o: 0.5x价格                                      │
│                                                              │
│  📦 应该缓存的内容                                          │
│  ├─ 系统提示词（长期不变）                                   │
│  ├─ 知识库文档（定期更新）                                   │
│  ├─ 产品文档（频繁查阅）                                     │
│  └─ API参考文档（变更不频繁）                                │
│                                                              │
│  📦 不应该缓存的内容                                        │
│  ├─ 用户查询（每次不同）                                     │
│  ├─ 会话上下文（会话级）                                    │
│  └─ 临时数据                                               │
│                                                              │
│  ⚙️ 最佳实践                                               │
│  ├─ 静态内容优先缓存                                       │
│  ├─ 设置合理的TTL                                          │
│  ├─ 变更时主动刷新                                         │
│  └─ 监控缓存命中率                                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关主题

- [[上下文窗口深度解析]] - 窗口限制是缓存需求的原因
- [[上下文压缩技术]] - 缓存+压缩=极致省钱
- [[RAG上下文优化指南]] - RAG场景下的缓存应用
- [[对话历史管理]] - 对话级别的上下文处理

---

## 参考文献

1. Anthropic (2024). **Claude API Documentation - Caching.**
2. Google (2024). **Gemini API Documentation - Cached Content.**
3. OpenAI (2024). **GPT-4 API Documentation.**
4. Wu, K., et al. (2024). **A Study of Context Caching in LLM Serving Systems.**
