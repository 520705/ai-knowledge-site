---
title: DeepSeek API完整使用指南
date: 2026-04-18
tags:
  - DeepSeek
  - API
  - MoE
  - 大模型
  - V3
  - Coder
categories:
  - AI工具实操
  - 大模型调用
alias: [deepseek-api-guide, deepseek-v3, deepseek-coder]
---

> [!info] 文档信息
> 本文档详细介绍DeepSeek API的使用方法，包括V3模型、Coder系列、V2版本、极致价格优势和MoE架构原理。

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| DeepSeek V3 | 最新一代通用大模型 |
| DeepSeek Coder | 编程专用模型 |
| MoE架构 | 混合专家架构 |
| 价格优势 | 极低调用成本 |
| 开源 | 开源模型权重 |
| 长上下文 | 128K上下文窗口 |
| 代码能力 | 业界领先编码能力 |
| 推理能力 | 数学和逻辑推理 |
| API调用 | RESTful接口 |
| 本地部署 | 支持私有化部署 |

---

## 一、DeepSeek概述

DeepSeek（深度求索）是一家专注于通用人工智能的中国科技公司，由幻方量化于2023年孵化创立。DeepSeek以"探索AGI的本质"为使命，致力于开发开源、高效、低成本的大语言模型。其团队在模型训练、推理优化和工程实践方面拥有深厚积累。

DeepSeek的核心优势在于其独特的MoE（Mixture of Experts，混合专家）架构设计，能够在保持强大能力的同时大幅降低训练和推理成本。这使得DeepSeek成为当前性价比最高的大模型选择之一，API调用价格仅为OpenAI的几十分之一。

## 二、MoE架构详解

### 2.1 MoE基本原理

传统的稠密模型（Dense Model）中，所有参数都会对每个输入进行处理。而MoE架构则采用"专家网络"的设计思想，将模型参数分散到多个"专家"中，每次推理时只激活与当前输入相关的少数专家。

```
┌─────────────────────────────────────────────────────────┐
│                      MoE 架构                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│    输入 ──→ 路由层(Router) ──→ 选择激活的专家           │
│                ↓                                        │
│         ┌─────┬─────┬─────┬─────┐                      │
│         │专家1│专家2│专家3│专家N│  ← 仅激活部分专家     │
│         └─────┴─────┴─────┴─────┘                      │
│                ↓                                        │
│         输出合并 ──→ 最终结果                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 2.2 DeepSeek MoE实现

DeepSeek-V3采用了细粒度的MoE设计，包含：

| 参数 | 说明 |
|------|------|
| 总参数量 | 671B（6710亿） |
| 激活参数量 | 37B（370亿） |
| 专家数量 | 256个专家 |
| 激活专家数 | 8个 |
| 共享专家 | 8个 |

这意味着DeepSeek V3每次推理只需要激活约5.5%的参数，却能调用全部的模型知识。

### 2.3 架构优势

- **训练成本低**：虽然总参数庞大，但每次训练计算量仅相当于37B参数的稠密模型
- **推理效率高**：稀疏激活机制大幅降低单次推理延迟
- **能力全面**：多专家协同保证各类任务的处理能力
- **可扩展性**：可以灵活增加专家数量提升能力

## 三、DeepSeek V3模型

### 3.1 V3核心能力

DeepSeek V3于2024年12月发布，是目前最强的开源通用大模型之一。在多个基准测试中表现优异：

| 基准测试 | DeepSeek V3 | GPT-4o | Claude 3.5 |
|----------|-------------|--------|------------|
| MMLU | 88.5% | 88.7% | 88.3% |
| MATH | 89.2% | 76.6% | 78.3% |
| HumanEval | 82.6% | 90.2% | 81.7% |
| GSM8K | 95.3% | 94.8% | 94.9% |
| C-Eval | 90.2% | 71.6% | 76.4% |

> [!note] 中文能力
> DeepSeek V3在中文任务上表现尤为突出，特别适合中国用户使用。

### 3.2 API调用方式

```python
import requests
import os

# DeepSeek API调用
api_key = os.environ.get("DEEPSEEK_API_KEY")
url = "https://api.deepseek.com/v1/chat/completions"

headers = {
    "Content-Type": "application/json",
    "Authorization": f"Bearer {api_key}"
}

data = {
    "model": "deepseek-chat",
    "messages": [
        {"role": "system", "content": "你是一位专业、高效的AI助手。"},
        {"role": "user", "content": "请解释什么是大语言模型"}
    ],
    "temperature": 0.7,
    "max_tokens": 1024
}

response = requests.post(url, headers=headers, json=data)
result = response.json()

print(result['choices'][0]['message']['content'])
```

### 3.3 Python SDK

```python
from openai import OpenAI

# 使用OpenAI兼容接口
client = OpenAI(
    api_key="your-deepseek-api-key",
    base_url="https://api.deepseek.com/v1"
)

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[
        {"role": "user", "content": "写一段Python代码实现快速排序"}
    ],
    temperature=0.7
)

print(response.choices[0].message.content)
```

### 3.4 JavaScript SDK

```javascript
import OpenAI from 'openai';

const deepseek = new OpenAI({
    apiKey: process.env.DEEPSEEK_API_KEY,
    baseURL: 'https://api.deepseek.com/v1'
});

async function main() {
    const completion = await deepseek.chat.completions.create({
        model: "deepseek-chat",
        messages: [
            { role: "system", content: "你是一个代码审查专家" },
            { role: "user", content: "审查以下代码的性能问题" }
        ]
    });
    
    console.log(completion.choices[0].message.content);
}
```

## 四、DeepSeek Coder系列

### 4.1 Coder模型概览

DeepSeek Coder是专门针对代码任务优化的模型系列，包含多个尺寸版本：

| 模型 | 参数量 | 上下文 | 适用场景 |
|------|--------|--------|----------|
| DeepSeek Coder 33B | 33B | 128K | 通用编程 |
| DeepSeek Coder 7B | 7B | 128K | 轻量任务 |
| DeepSeek Coder 1.5B | 1.5B | 32K | 本地运行 |

### 4.2 代码能力对比

DeepSeek Coder在代码相关任务上表现优异：

```python
# 代码生成示例
response = client.chat.completions.create(
    model="deepseek-coder",
    messages=[
        {
            "role": "user",
            "content": """请用Python实现一个LRU缓存类，要求：
            1. 支持get和put操作，时间复杂度O(1)
            2. 容量可配置
            3. 线程安全
            4. 包含完整的类型注解"""
        }
    ],
    temperature=0.1
)

print(response.choices[0].message.content)
```

### 4.3 代码补全

```python
# 代码补全任务
response = client.chat.completions.create(
    model="deepseek-coder",
    messages=[
        {
            "role": "user",
            "content": """# 补全以下函数
def fibonacci(n: int) -> list[int]:
    '''计算斐波那契数列前n项'''
    """
        }
    ]
)
```

### 4.4 代码审查

```python
# 代码审查任务
def review_code(code):
    response = client.chat.completions.create(
        model="deepseek-coder",
        messages=[
            {
                "role": "system",
                "content": """你是一位资深代码审查专家。请审查代码并指出：
                1. 潜在的bug和安全漏洞
                2. 性能问题
                3. 代码风格问题
                4. 改进建议"""
            },
            {
                "role": "user",
                "content": f"请审查以下Python代码：\n```python\n{code}\n```"
            }
        ]
    )
    return response.choices[0].message.content
```

## 五、DeepSeek V2系列

### 5.1 V2核心特性

DeepSeek V2于2024年5月发布，引入了一系列重要创新：

- **DeepSeekMoE**：更高效的MoE架构
- **MLA**：多头潜在注意力机制，大幅降低推理内存
- **FP8训练**：8位浮点训练，降低训练成本
- **长上下文**：支持128K上下文窗口

### 5.2 API调用

```python
# V2 API调用
response = client.chat.completions.create(
    model="deepseek-chat-v2",
    messages=[
        {"role": "user", "content": "解释深度学习的原理"}
    ]
)
```

## 六、价格优势分析

### 6.1 价格对比表

DeepSeek最大的竞争优势之一是其极具吸引力的定价策略：

| 模型 | 输入价格 | 输出价格 | 对比GPT-4o |
|------|----------|----------|-------------|
| DeepSeek V3 | ¥1/百万tokens | ¥2/百万tokens | 1/70 |
| DeepSeek Coder | ¥1/百万tokens | ¥2/百万tokens | 1/70 |
| GPT-4o | $5/百万tokens | $15/百万tokens | 基准 |
| Claude 3.7 | $3/百万tokens | $15/百万tokens | 1/5 |

> [!tip] 成本节省
> 使用DeepSeek替代GPT-4o，在相同调用量下可节省约98%的成本。

### 6.2 成本计算示例

```python
# 假设每天处理100万token输入和50万token输出

# GPT-4o成本
gpt4o_cost = (1 * 5 + 0.5 * 15)  # $12.5/天

# DeepSeek成本
deepseek_cost = (1 * 0.14 + 0.5 * 0.28)  # ¥0.28/天 ≈ $0.04/天

# 年节省
yearly_savings = (gpt4o_cost - deepseek_cost) * 365
print(f"每年可节省约 ${yearly_savings:.2f}")
```

## 七、Function Calling

### 7.1 工具定义

```python
# 定义工具
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取城市天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "城市名称"},
                    "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["city"]
            }
        }
    }
]

response = client.chat.completions.create(
    model="deepseek-chat",
    messages=[{"role": "user", "content": "北京今天多少度？"}],
    tools=tools,
    tool_choice="auto"
)

# 处理函数调用
for choice in response.choices:
    if choice.finish_reason == "tool_calls":
        for tool_call in choice.message.tool_calls:
            print(f"调用: {tool_call.function.name}")
            print(f"参数: {tool_call.function.arguments}")
```

## 八、本地部署

### 8.1 Ollama部署

DeepSeek模型支持通过Ollama本地部署：

```bash
# 拉取模型
ollama pull deepseek-coder:6.7b
ollama pull deepseek-chat:7b

# 运行推理
ollama run deepseek-coder:6.7b "写一个快速排序算法"
```

### 8.2 vLLM部署

```python
# 使用vLLM部署
# 命令行启动
# vllm serve deepseek-ai/DeepSeek-V3-Chat --dtype half --port 8000

from openai import OpenAI

client = OpenAI(
    api_key="EMPTY",
    base_url="http://localhost:8000/v1"
)

response = client.chat.completions.create(
    model="deepseek-ai/DeepSeek-V3-Chat",
    messages=[{"role": "user", "content": "你好"}]
)
```

## 九、2026年4月定价

| 模型 | 输入价格 | 输出价格 | 单位 |
|------|----------|----------|------|
| DeepSeek V3 | ¥1 | ¥2 | 每百万tokens |
| DeepSeek Coder | ¥1 | ¥2 | 每百万tokens |
| DeepSeek V2.5 | ¥1 | ¥2 | 每百万tokens |

> [!note] 汇率说明
> 上述价格为人民币，1美元约兑换7.2元人民币。

## 十、相关资源

- [[Gemini_API完整指南]] - Google Gemini API
- [[Anthropic_Claude_API指南]] - Claude API
- [[国内大模型API对比]] - 国内模型对比
- [[Ollama自定义模型]] - 本地部署配置
- [[API成本优化策略]] - 成本控制

---

> [!success] 完成状态
> 本文档已完成DeepSeek API的全面介绍，涵盖MoE架构、V3/Coder系列、价格优势、本地部署等核心内容。
