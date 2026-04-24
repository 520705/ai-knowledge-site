---
title: Anthropic Claude API完整指南
date: 2026-04-18
tags:
  - Claude
  - Anthropic
  - API
  - 大模型
  - AI工具
categories:
  - AI工具实操
  - 大模型调用
alias: [claude-api-guide, anthropic-api]
---

> [!info] 文档信息
> 本文档详细介绍Anthropic Claude API的使用方法，包括API格式、模型选择、Haiku 4.x特性、工具使用和多模态能力。

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| Claude 4.x | Anthropic最新一代AI助手模型 |
| Haiku 4.x | 轻量级高速模型，适合简单任务 |
| Sonnet 4.x | 中等规模模型，性价比之选 |
| Opus 4.x | 旗舰模型，最强推理能力 |
| Tools | 函数调用和工具使用能力 |
| Vision | 图像理解与分析 |
| API格式 | REST API和SDK调用方式 |
| 上下文窗口 | 最大200K tokens |
| 系统提示词 | System Prompt配置 |
| 安全过滤 | Claude的安全机制 |

---

## 一、Anthropic Claude API概述

Anthropic公司由OpenAI前员工于2021年创立，专注于开发安全、可靠且有益的AI系统。Claude作为其核心产品，经过多次迭代升级，目前已更新至Claude 4.x系列。Claude API以其卓越的推理能力、长达200K tokens的超长上下文窗口以及创新的工具使用能力，成为企业和开发者构建AI应用的重要选择之一。

Claude 4.x系列包含三个主要型号：Haiku、Sonnet和Opus。Haiku作为轻量级模型，以极快的响应速度和低廉的使用成本著称，非常适合处理大量简单的分类、提取和生成任务。Sonnet定位中端，在性能和成本之间取得了良好的平衡，是大多数应用的默认选择。Opus作为旗舰模型，拥有最强的推理能力和最复杂的任务处理能力，适合需要深度分析和创造力的场景。

## 二、API接入基础

### 2.1 获取API密钥

使用Claude API的第一步是获取API密钥。用户需要访问Anthropic官方网站完成账号注册和认证流程。在控制台的安全设置中，可以生成和管理API密钥。建议将密钥存储在环境变量中，避免在代码中硬编码。

### 2.2 API端点与认证

Claude API采用RESTful架构，主要端点地址为`https://api.anthropic.com/v1/messages`。所有请求都需要在HTTP头部包含`x-api-key`认证信息和`anthropic-version`版本标识。

```python
import anthropic
import os

# 初始化客户端
client = anthropic.Anthropic(
    api_key=os.environ.get("ANTHROPIC_API_KEY")
)

# 发送消息
message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "请解释什么是大语言模型"}
    ]
)

print(message.content)
```

### 2.3 请求参数详解

Claude API的消息创建接口支持丰富的参数配置。`model`参数指定使用的模型版本，建议使用带日期后缀的版本号以确保稳定性。`max_tokens`设置模型输出的最大token数，必须设置为正整数。`temperature`控制输出的随机性，取值范围为0到1，值越低输出越确定性。`top_p`控制 nucleus采样的阈值。`system`参数用于传入系统提示词，定义AI的行为角色和约束。

## 三、模型选择指南

### 3.1 Haiku 4.x系列

Claude Haiku是Anthropic推出的轻量级模型，专为高速、低延迟场景设计。在Claude 4.x更新中，Haiku的能力得到了显著提升：

| 特性 | 说明 |
|------|------|
| 速度 | 响应时间低于100ms |
| 成本 | $0.25/百万输入tokens |
| 上下文 | 200K tokens |
| 能力 | 日常对话、分类、提取 |

Haiku特别适合需要大量调用的应用场景，如客户服务自动化、内容审核、数据分类等。其低廉的价格使得高频调用成为可能。

```javascript
// JavaScript调用示例
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic();

async function quickClassify(text) {
  const response = await client.messages.create({
    model: 'claude-haiku-4-20250514',
    max_tokens: 50,
    messages: [{
      role: 'user',
      content: `请判断以下文本的情感是正面、负面还是中性：${text}`
    }]
  });
  return response.content[0].text;
}
```

### 3.2 Sonnet 4.x系列

Sonnet定位为全能型选手，在大多数任务上都表现出色。Claude Sonnet 4.x相较于前代有了明显进步：

- 编码能力提升40%
- 数学推理能力提升35%
- 指令遵循准确性提升25%
- 更长的多轮对话保持能力

```python
# Python调用Sonnet进行代码审查
def review_code(code_snippet):
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2048,
        system="你是一位经验丰富的代码审查专家，负责检查代码的安全漏洞、性能问题和最佳实践。",
        messages=[{
            "role": "user",
            "content": f"请审查以下Python代码：\n```{code_snippet}```"
        }]
    )
    return response.content[0].text
```

### 3.3 Opus 4.x系列

Opus是Claude的旗舰模型，代表了Anthropic在AI领域的最高技术水平。它在复杂推理、长文档分析、创意写作等方面表现卓越：

| 指标 | Sonnet 4.x | Opus 4.x |
|------|------------|----------|
| 推理基准分 | 85 | 92 |
| 上下文窗口 | 200K | 200K |
| 价格(输入) | $3/百万tokens | $15/百万tokens |
| 适用场景 | 通用任务 | 复杂分析 |

> [!note] 模型选择建议
> - 简单分类/提取任务：选择Haiku
> - 日常开发/写作：选择Sonnet
> - 复杂推理/长文档分析：选择Opus

## 四、工具使用（Tools）详解

Claude 4.x引入了强大的工具使用能力，允许模型调用外部函数、访问实时数据和执行具体操作。这是实现Agent系统的核心技术基础。

### 4.1 工具定义格式

工具通过JSON Schema格式定义，包含名称、描述和参数规格：

```python
# 定义一个天气查询工具
weather_tool = {
    "name": "get_weather",
    "description": "获取指定城市的当前天气信息",
    "input_schema": {
        "type": "object",
        "properties": {
            "city": {
                "type": "string",
                "description": "城市名称，支持中文或英文"
            },
            "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"],
                "description": "温度单位"
            }
        },
        "required": ["city"]
    }
}
```

### 4.2 工具调用流程

完整的工作流程包含以下步骤：

1. 用户发起请求
2. 模型判断是否需要调用工具
3. 返回工具调用请求（带有工具名和参数）
4. 系统执行工具获取结果
5. 将结果反馈给模型
6. 模型生成最终回复

```python
# 完整的工具调用示例
response = client.messages.create(
    model="claude-opus-4-20250514",
    max_tokens=1024,
    tools=[weather_tool],
    messages=[{
        "role": "user",
        "content": "北京今天的天气怎么样？适合穿什么衣服？"
    }]
)

# 处理响应
for content in response.content:
    if content.type == "tool_use":
        tool_name = content.name
        tool_input = content.input
        print(f"调用工具: {tool_name}")
        print(f"参数: {tool_input}")
        # 执行实际工具调用
        # ...
```

> [!tip] 最佳实践
> 工具定义要清晰准确，参数描述要具体明确，这有助于模型正确理解何时以及如何使用工具。

### 4.3 多工具协同

Claude支持同时定义多个工具，模型会根据任务需求自主选择调用顺序和组合：

```python
tools = [
    {
        "name": "web_search",
        "description": "搜索互联网获取最新信息",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string"}
            },
            "required": ["query"]
        }
    },
    {
        "name": "database_query",
        "description": "查询本地数据库",
        "input_schema": {
            "type": "object",
            "properties": {
                "sql": {"type": "string"}
            },
            "required": ["sql"]
        }
    },
    {
        "name": "send_email",
        "description": "发送电子邮件",
        "input_schema": {
            "type": "object",
            "properties": {
                "to": {"type": "string"},
                "subject": {"type": "string"},
                "body": {"type": "string"}
            },
            "required": ["to", "subject", "body"]
        }
    }
]
```

## 五、Vision多模态能力

Claude 4.x系列全面支持图像理解，用户可以上传图片并询问相关问题。这在文档分析、图表解读、视觉问答等场景中非常实用。

### 5.1 图片输入格式

支持多种图片格式：PNG、JPEG、GIF、WebP。图片可以通过URL或base64编码的方式传入：

```python
# 通过URL传入图片
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "url",
                    "url": "https://example.com/chart.png"
                }
            },
            {
                "type": "text",
                "text": "请描述这张图表的主要内容和趋势"
            }
        ]
    }]
)

# 通过base64传入本地图片
import base64

with open("screenshot.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": [
            {
                "type": "image",
                "source": {
                    "type": "base64",
                    "media_type": "image/png",
                    "data": image_data
                }
            },
            {
                "type": "text",
                "text": "这张截图显示了什么内容？"
            }
        ]
    }]
)
```

### 5.2 应用场景

Vision能力在以下场景中特别有价值：

- **文档数字化**：识别并提取扫描文档、手写笔记中的文字
- **数据可视化**：分析图表、图形，提取关键数据点
- **UI/UX审查**：检查界面设计的可用性和美观性
- **产品识别**：识别商品、品牌、物体信息
- **医学影像**：辅助分析X光片、CT扫描等

## 六、提示词工程最佳实践

### 6.1 系统提示词设计

系统提示词是定义Claude行为的关键。以下是一些有效的模式：

```python
# 角色定义模式
system_prompt = """你是一位资深的[领域]专家，具有[年数]年的行业经验。
你的职责是：
1. [具体职责1]
2. [具体职责2]
3. [具体职责3]

在回答问题时，请：
- 使用专业术语，但保持解释清晰
- 提供具体的例子和数据支撑
- 指出常见的误区和注意事项"""

# 少样本学习模式
system_prompt = """你是一个文本分类器。请根据以下示例进行分类：

示例1：
输入："这家餐厅的服务太差了，等了30分钟没人理"
分类：负面

示例2：
输入："性价比很高，会再次光顾"
分类：正面

现在请分类以下文本："""
```

### 6.2 结构化输出

当需要程序化处理输出时，建议明确要求模型返回结构化格式：

```python
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    system="你是一个数据提取助手。请以JSON格式返回提取的信息，不要添加任何解释。",
    messages=[{
        "role": "user",
        "content": "从以下文本中提取人名、地点、日期：会议将于2026年5月15日在上海浦东召开，届时张三和李四将代表公司出席。"
    }]
)
```

> [!warning] 注意事项
> 结构化输出只是请求，模型可能偶尔产生格式偏差，生产环境中应添加输出验证逻辑。

## 七、2026年4月定价参考

| 模型 | 输入价格 | 输出价格 | 上下文窗口 |
|------|----------|----------|------------|
| Opus 4.x | $15/百万tokens | $75/百万tokens | 200K |
| Sonnet 4.x | $3/百万tokens | $15/百万tokens | 200K |
| Haiku 4.x | $0.25/百万tokens | $1.25/百万tokens | 200K |

> [!tip] 成本优化建议
> - 简单任务优先使用Haiku
> - 利用缓存机制减少重复输入
> - 设置合理的max_tokens避免过度消耗

## 八、相关资源

- [[GPT_API完整指南]] - OpenAI GPT系列对比
- [[Gemini_API完整指南]] - Google Gemini API
- [[Ollama进阶使用指南]] - 本地模型部署
- [[API成本优化策略]] - 成本控制技巧
- [[多模态大模型详解]] - 多模态能力对比

---

> [!success] 完成状态
> 本文档已完成Claude API的全面介绍，涵盖API接入、模型选择、工具使用、Vision能力和最佳实践。
