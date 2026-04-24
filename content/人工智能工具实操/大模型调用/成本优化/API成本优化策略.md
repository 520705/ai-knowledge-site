---
title: API成本优化策略
date: 2026-04-18
tags:
  - API成本
  - 成本优化
  - Prompt压缩
  - Context Caching
  - 批量API
  - 预算告警
categories:
  - AI工具实操
  - 大模型调用
alias: [cost-optimization, api-cost, token-optimization]
---

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| 模型分层 | 任务分级分配模型 |
| Prompt压缩 | 精简提示词 |
| Context Caching | 上下文缓存 |
| 批量API | 批量处理请求 |
| 预算告警 | 成本监控 |

---

## 一、成本优化概述

大模型API调用成本是AI应用开发中不可忽视的重要因素。API成本主要由输入token和输出token两部分构成，不同模型的价格差异可达数十倍，合理的成本优化策略可以将AI应用运营成本降低80%以上。

## 二、模型分层策略

### 任务分级体系

| 级别 | 任务类型 | 推荐模型 | 成本范围 |
|------|----------|----------|----------|
| L1-简单 | 分类、提取、翻译 | Haiku/Mini | $0.001-0.01/千次 |
| L2-标准 | 对话、总结、写作 | 标准模型 | $0.01-0.1/千次 |
| L3-复杂 | 分析、推理、代码 | Pro版本 | $0.1-1/千次 |
| L4-专家 | 深度分析、创作 | 旗舰模型 | $1-10/千次 |

### 路由实现

```python
from enum import Enum
from dataclasses import dataclass

class TaskLevel(Enum):
    SIMPLE = "simple"
    STANDARD = "standard"
    COMPLEX = "complex"
    EXPERT = "expert"

MODEL_MAP = {
    TaskLevel.SIMPLE: "gpt-4o-mini",
    TaskLevel.STANDARD: "gpt-4o",
    TaskLevel.COMPLEX: "claude-sonnet-4",
    TaskLevel.EXPERT: "claude-opus-4"
}

def classify_task(prompt: str) -> TaskLevel:
    simple_keywords = ["分类", "提取", "翻译", "判断"]
    complex_keywords = ["分析", "推理", "比较", "设计"]
    prompt_lower = prompt.lower()
    
    if any(kw in prompt_lower for kw in complex_keywords):
        return TaskLevel.COMPLEX
    elif any(kw in prompt_lower for kw in simple_keywords):
        return TaskLevel.SIMPLE
    return TaskLevel.STANDARD
```

### 成本对比表

| 场景 | 奢侈方案 | 推荐方案 | 节省比例 |
|------|----------|----------|----------|
| 简单分类 | GPT-4o | GPT-4o-mini | 94% |
| 文档摘要 | Claude Opus | Claude Sonnet | 80% |
| 代码生成 | GPT-4o | DeepSeek Coder | 95% |

## 三、Prompt压缩策略

### 提示词精简

```python
# 优化前（冗长）
bad_prompt = """
你是一位专业的人工智能助手。我需要你帮我完成以下任务：
请仔细阅读下面的内容，然后进行情感分析。
以下是待分析的文本...

# 优化后（精简）
good_prompt = "判断情感：这家餐厅的服务非常好，菜品也很美味。情感："

def compress_prompt(prompt: str) -> str:
    removals = ["请仔细", "请务必", "请判断", "以下是"]
    result = prompt
    for r in removals:
        result = result.replace(r, "")
    return result.strip()
```

### Few-shot压缩

```python
# 优化前
few_shot_long = """
判断情感（正面/负面/中性）：
示例1：文本：太棒了！情感：正面
示例2：文本：还行情感：中性
示例3：文本：失望情感：负面
现在判断...

# 优化后
few_shot_short = "情感分类："太棒了"→正 "还行"→中 "失望"→负\n判断："
```

## 四、Context Caching上下文缓存

### 缓存折扣对比

| 提供商 | 缓存折扣 | 说明 |
|--------|----------|------|
| OpenAI | 50% | Cache-aware API |
| Anthropic | 75% | Prompt Caching |
| Google | 64% | Context Caching |

### Anthropic Prompt Caching

```python
import anthropic

client = anthropic.Anthropic(api_key="your-key")

def generate_with_cache(prompt: str, system_prompt: str):
    messages = []
    if system_prompt:
        messages.append({
            "role": "user",
            "content": [
                {"type": "cache_control", "cache_window": "initial"},
                {"type": "text", "text": f"系统设定：{system_prompt}"}
            ]
        })
    messages.append({"role": "user", "content": prompt})
    
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=messages
    )
    return response.content[0].text
```

### 智能缓存

```python
import hashlib
import time

class SmartCache:
    def __init__(self, ttl: int = 3600):
        self.cache = {}
        self.ttl = ttl
        self.hits = 0
        self.misses = 0
    
    def get_or_compute(self, key: str, compute_fn, *args):
        cache_key = hashlib.md5(key.encode()).hexdigest()
        
        if cache_key in self.cache:
            entry = self.cache[cache_key]
            if time.time() - entry['timestamp'] < self.ttl:
                self.hits += 1
                return entry['value']
        
        self.misses += 1
        value = compute_fn(*args)
        self.cache[cache_key] = {'value': value, 'timestamp': time.time()}
        return value
    
    def stats(self):
        total = self.hits + self.misses
        hit_rate = self.hits / total if total > 0 else 0
        return {'hit_rate': f"{hit_rate * 100:.1f}%"}
```

## 五、批量API优化

### 批量处理

```python
import asyncio

class BatchProcessor:
    def __init__(self, client, batch_size: int = 10):
        self.client = client
        self.batch_size = batch_size
    
    async def process_batch(self, prompts: list):
        semaphore = asyncio.Semaphore(self.batch_size)
        
        async def process_single(prompt):
            async with semaphore:
                return await self._call_api(prompt)
        
        tasks = [process_single(p) for p in prompts]
        return await asyncio.gather(*tasks)
    
    async def _call_api(self, prompt: str):
        response = await self.client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}]
        )
        return response.choices[0].message.content
```

## 六、成本监控与告警

### 成本追踪器

```python
from datetime import datetime

class CostTracker:
    def __init__(self, budget: float = 100):
        self.budget = budget
        self.spent = 0.0
        self.history = []
        self.alerts = []
    
    def record_usage(self, model: str, input_tokens: int, 
                     output_tokens: int, cost_per_1k_input: float,
                     cost_per_1k_output: float):
        input_cost = (input_tokens / 1000) * cost_per_1k_input
        output_cost = (output_tokens / 1000) * cost_per_1k_output
        total_cost = input_cost + output_cost
        
        self.spent += total_cost
        
        record = {
            'timestamp': datetime.now().isoformat(),
            'model': model,
            'cost': total_cost,
            'running_total': self.spent
        }
        self.history.append(record)
        
        percentage = (self.spent / self.budget) * 100
        if percentage >= 80:
            self.alerts.append({
                'level': 'warning' if percentage < 100 else 'critical',
                'message': f'预算使用已达 {percentage:.1f}%'
            })
        return record
    
    def get_stats(self):
        return {
            'budget': self.budget,
            'spent': self.spent,
            'remaining': self.budget - self.spent,
            'usage_percent': (self.spent / self.budget) * 100
        }
```

## 七、2026年4月最新定价参考

| 模型 | 输入价格 | 输出价格 | 单位 |
|------|----------|----------|------|
| GPT-4o | $2.50 | $10.00 | 每百万tokens |
| GPT-4o-mini | $0.15 | $0.60 | 每百万tokens |
| Claude Opus 4 | $15.00 | $75.00 | 每百万tokens |
| Claude Sonnet 4 | $3.00 | $15.00 | 每百万tokens |
| Claude Haiku 4 | $0.25 | $1.25 | 每百万tokens |
| Gemini 2.0 Flash | $0.10 | $0.40 | 每百万tokens |
| DeepSeek V3 | ¥1.00 | ¥2.00 | 每百万tokens |

> [!tip] 最佳实践
> - 简单任务使用mini版本，节省90%以上成本
> - 开启上下文缓存，节省50-75%费用
> - 设置每日/每周预算，避免意外超支
> - 定期分析使用日志，优化低效请求

## 八、相关资源

- [[Claude_API指南]] - Claude API详解
- [[Gemini_API完整指南]] - Gemini API详解
- [[DeepSeek_API使用]] - DeepSeek API
- [[国内大模型API对比]] - 国内模型

---

> [!success] 完成状态
> 本文档已完成API成本优化策略的全面介绍。
