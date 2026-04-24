---
title: GPT API完整指南
date: 2026-04-18
tags:
  - OpenAI
  - GPT-4o
  - API
  - Chat Completions
  - 模型调用
  - 人工智能
  - 大模型
categories:
  - AI工具实操
  - 大模型调用
  - OpenAI API
---

# GPT API完整指南

## 关键词列表

| 关键词 | 说明 |
|--------|------|
| Chat Completions | OpenAI对话补全API端点 |
| GPT-4o | OpenAI最新的多模态旗舰模型 |
| o3-mini | 2026年最新推理优化模型 |
| temperature | 控制输出随机性的参数 |
| top_p | 核采样参数 |
| frequency_penalty | 频率惩罚参数 |
| function calling | 函数调用能力 |
| vision | 视觉输入支持 |
| token计算 | 输入输出token计量方式 |
| stream | 流式输出选项 |

---

## 一、OpenAI API概述与核心概念

### 1.1 OpenAI API生态系统

OpenAI API是目前全球最成熟、应用最广泛的大语言模型API服务。自2020年6月发布GPT-3 API以来，OpenAI已经建立起一套完整的AI API生态系统，涵盖文本生成、图像理解、语音处理、嵌入向量等多个领域。2026年4月，OpenAI的API服务已经迭代到第五代模型架构，支持实时多模态交互，处理速度提升至毫秒级别。

OpenAI API采用RESTful架构设计，通过HTTPS协议进行通信，支持全球多个区域的数据中心部署。开发者可以通过统一的API端点访问不同的模型服务，底层基础设施自动处理负载均衡和容错机制。API采用Token计费模式，输入和输出分别计算消耗，这种精细化的计费方式让开发者能够精确控制成本。

OpenAI API的核心优势体现在以下几个方面：首先，模型能力持续领先，GPT-4系列模型在推理、编程、多轮对话等任务上保持业界领先水平；其次，API设计简洁直观，丰富的官方SDK降低了开发门槛；第三，生态完善，拥有庞大的开发者社区和丰富的第三方集成工具；最后，服务稳定性高，SLA达到99.9%以上，企业级应用有充分保障。

### 1.2 API端点与基础格式

OpenAI API提供多个端点，每个端点对应不同的功能模块。最核心的端点是`https://api.openai.com/v1/chat/completions`，用于对话补全任务。这个端点采用JSON格式进行请求和响应，支持同步和流式两种响应模式。

基础的Chat Completions请求结构包含以下关键字段：`model`指定使用的模型ID，`messages`数组定义对话历史，`max_tokens`限制最大输出token数，`temperature`控制随机性，`stream`布尔值决定是否启用流式输出。响应结构中，`choices`数组包含生成的内容，`usage`对象报告token消耗情况，`model`字段标识实际处理的模型。

```json
POST https://api.openai.com/v1/chat/completions
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "model": "gpt-4o",
  "messages": [
    {"role": "system", "content": "你是一位专业的技术写作助手。"},
    {"role": "user", "content": "请解释什么是Token以及它如何影响API调用成本。"}
  ],
  "max_tokens": 1000,
  "temperature": 0.7,
  "stream": false
}
```

### 1.3 API密钥管理与安全

OpenAI API密钥是访问服务的唯一凭证，采用`sk-`前缀的格式。密钥分为两类：Secret Key用于服务端调用，具有完整权限；Project API Key是2025年推出的新型密钥，按项目维度管理权限，适合团队协作场景。

密钥安全最佳实践包括：绝对不要将密钥硬编码在代码中或提交到版本控制系统；使用环境变量或密钥管理服务存储密钥；为不同应用创建独立的密钥，便于权限管理和撤销；启用密钥使用告警，当检测到异常调用量时及时通知；定期轮换密钥，建议每90天更换一次。

```bash
# 正确的密钥配置方式
export OPENAI_API_KEY="sk-your-key-here"

# 在Python中使用环境变量
import os
api_key = os.environ.get("OPENAI_API_KEY")
```

## 二、模型选择与版本详解

### 2.1 GPT-4o系列

GPT-4o（Omni）是OpenAI于2024年5月发布的旗舰多模态模型，"o"代表Omni（全能），象征其处理多种模态的能力。GPT-4o能够理解和生成文本、图像、音频，实现了真正的多模态统一架构。

**GPT-4o的主要特性：**

GPT-4o采用全新的统一Transformer架构，将原本分离的视觉编码器、语音识别器、语音合成器整合为单一模型。这种设计带来了显著的性能提升：响应速度比GPT-4 Turbo快两倍，成本降低50%，视觉理解能力大幅增强。在MMLU基准测试中，GPT-4o达到88.7%的准确率，在人类评估中超过GPT-4的表现。

GPT-4o支持实时语音对话，可以理解语气、停顿、背景噪音等语音特征，生成自然流畅的语音回复。内置的情绪识别能力让它能够根据对话内容调整回复风格，在客服、心理辅导、教育等场景中表现出色。

**GPT-4o的细分版本：**

- **gpt-4o**：标准版本，适合大多数应用场景
- **gpt-4o-2026-04**：特定快照版本，模型权重冻结，输出稳定
- **gpt-4o-mini**：轻量版本，速度更快，成本更低，适合简单任务
- **gpt-4o-search-preview**：集成搜索能力的版本

**2026年4月最新定价（每百万Token）：**

| 版本 | 输入价格 | 输出价格 |
|------|----------|----------|
| gpt-4o | $2.50 | $10.00 |
| gpt-4o-mini | $0.15 | $0.60 |

### 2.2 o3与o3-mini系列

o3是OpenAI在2025年底发布的推理能力强化模型，专注于复杂推理任务。o3-mini是其轻量版本，在保持强劲推理能力的同时优化了成本和延迟。

**o3系列的创新点：**

o3系列采用全新的推理架构，模仿人类思考的"内内心独白"机制。在处理复杂问题时，模型会先生成详细的推理步骤，逐步推导出答案，然后整合成最终回复。这种"先思考再回答"的方式使得o3在数学、编程、科学推理等任务上达到接近人类专家的水平。

在AIME数学竞赛测试中，o3达到91.6%的准确率（人类金牌选手平均水平约为90%）；在Codeforces编程评测中，o3进入全球前200名；在大规模多任务语言理解（MMLU）测试中，o3达到91.1%的准确率。

**o3-mini的适用场景：**

o3-mini针对日常推理任务进行了优化，包括代码调试、逻辑分析、数据解读等。相比标准o3，o3-mini的延迟降低60%，成本降低80%，非常适合需要频繁调用的应用场景。

**2026年4月最新定价：**

| 版本 | 输入价格 | 输出价格 |
|------|----------|----------|
| o3 | $4.00 | $16.00 |
| o3-mini | $0.55 | $2.20 |

### 2.3 GPT-4 Turbo与GPT-4o-mini

GPT-4 Turbo是OpenAI在2023年11月发布的优化版本，相比原版GPT-4有显著改进。GPT-4o-mini则是2024年7月推出的轻量级模型，专为成本敏感型应用设计。

**GPT-4 Turbo的特性：**

GPT-4 Turbo将上下文窗口扩展到128K Token，能够一次性处理整本书籍或大型代码库。知识截止日期更新到2025年6月，涵盖最新的世界事件和技术发展。训练数据截止日期的更新对于需要了解最新信息的应用至关重要，例如新闻分析、市场研究、产品推荐等场景。

在性能方面，GPT-4 Turbo比GPT-4快两倍，成本降低一半。更快的推理速度使得实时应用成为可能，如在线客服、实时翻译、交互式写作辅助等场景。

**GPT-4o-mini的市场定位：**

GPT-4o-mini是OpenAI最便宜的GPT-4级别模型，专为简单任务设计。它在MMLU测试中达到82%的准确率，超过了GPT-3.5 Turbo的整体表现，但成本仅为其三分之一。

GPT-4o-mini特别适合以下场景：大规模简单问答系统、内容分类与标签生成、文本摘要与改写、基础对话机器人、数据提取与格式化等。在这些场景中，使用GPT-4o-mini可以在保证质量的同时大幅降低成本。

## 三、核心参数详解

### 3.1 temperature（温度参数）

temperature是控制模型输出随机性的核心参数，取值范围通常为0到2之间。数值越低，输出越确定性和保守；数值越高，输出越随机和创造性强。

**参数作用机制：**

大语言模型在生成每个token时，会计算所有可能token的概率分布。temperature参数通过缩放这些概率来影响选择过程。当temperature=0时，模型总是选择概率最高的token，输出几乎是确定性的；当temperature=1时，保持原始概率分布；当temperature=2时，概率分布被拉伸，低概率选项被放大，高概率选项被相对弱化。

**场景化推荐值：**

| 场景 | 推荐temperature | 说明 |
|------|------------------|------|
| 代码生成 | 0.0-0.2 | 确定性输出，避免随机性引入bug |
| 事实问答 | 0.0-0.3 | 准确性优先，减少幻觉 |
| 内容摘要 | 0.3-0.5 | 平衡准确性与流畅性 |
| 创意写作 | 0.7-0.9 | 鼓励多样性，激发创意 |
| 角色扮演 | 0.8-1.0 | 强调个性化和情感表达 |

**实战技巧：**

在实际应用中，temperature需要与其他参数配合调整。当使用top_p参数时，两者会相互影响，建议只设置其中一个而非同时设置。对于需要高确定性的任务，将temperature设为0是最佳选择；只有在需要创意和变化的场景才提高temperature值。

```python
import openai

# 确定性输出场景
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "1+1等于几？"}],
    temperature=0.0  # 期望得到精确答案
)

# 创意写作场景
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "写一首关于秋天的诗"}],
    temperature=0.9  # 鼓励创意发挥
)
```

### 3.2 top_p（核采样参数）

top_p是另一种控制输出多样性的参数，中文常译为"核采样"或"ucleus采样"。它采用累积概率的方式选择候选token池。

**参数作用机制：**

top_p定义了模型从最高概率开始累积，选择总概率达到p值的最小token集合，然后在这个集合中随机采样。例如top_p=0.9表示选择占概率分布90%的高概率token，然后在这些token中随机选择。

这种设计的好处是：当概率分布集中在少数token上时，选择范围较小，保持确定性；当分布较为均匀时，选择范围扩大，增加多样性。与temperature相比，top_p更智能地适应不同的概率分布状态。

**使用建议：**

在大多数场景下，top_p=0.9是一个好的默认值，既保证了输出的多样性，又避免了极低概率的异常输出。如果同时使用temperature和top_p，建议将temperature设为0，然后只调整top_p；或者将top_p设为1，只使用temperature控制。

### 3.3 frequency_penalty与presence_penalty

这两个参数用于控制输出的词汇重复问题，但机制不同。

**frequency_penalty（频率惩罚）：**

frequency_penalty根据token在已生成文本中出现的次数降低其概率，出现次数越多，被再次选择的概率越低。取值范围为-2到2，正值减少重复，负值鼓励重复。

这个参数特别适合生成长文本的场景，可以有效避免模型陷入"词穷"状态，不断重复相同的内容。在写文章、报告、故事等长内容时，设置frequency_penalty=0.5到1.0可以显著提升内容质量。

**presence_penalty（存在惩罚）：**

presence_penalty只关心token是否出现过，而不关心出现次数。如果一个token已经出现过，其概率就被降低一次，无论它出现了多少次。取值范围同样是-2到2。

presence_penalty适合需要引入新话题或新概念的场景。例如在头脑风暴中，希望模型不断提出新的想法而不是围绕同一话题打转时，增大presence_penalty值非常有效。

**联合调优策略：**

```python
# 生成创意文章时减少重复
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "写一篇关于AI未来的文章"}],
    temperature=0.8,
    frequency_penalty=0.8,
    presence_penalty=0.6
)

# 生成精确代码时允许关键词重复
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "写一个Python函数计算斐波那契数列"}],
    temperature=0.0,
    frequency_penalty=0.0,
    presence_penalty=0.0
)
```

### 3.4 max_tokens与停止符

**max_tokens参数：**

max_tokens限制单次生成的最大token数量，是控制响应长度和成本的重要参数。默认值因模型而异，通常在16K到32K之间。

设置max_tokens时需要考虑：输入长度 + max_tokens不能超过模型的上下文窗口；输出越长，成本越高，生成时间越长；合理设置可以避免不必要的token消耗。

```python
# 短回答场景
response = openai.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "北京是哪个国家的首都？"}],
    max_tokens=50  # 简短回答即可
)

# 长文章场景
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "写一篇详细的AI发展史"}],
    max_tokens=4000  # 需要更长输出
)
```

**stop参数（停止符）：**

stop参数接受字符串或字符串数组，指定一个或多个停止序列。当模型生成这些序列时，会立即停止生成。

```python
# 使用单个停止符
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "列出5个编程语言"}],
    stop="5"  # 当输出"5."时停止
)

# 使用多个停止符
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "继续这个故事"}],
    stop=["\n\n", "THE END"]  # 遇到空行或特定标记时停止
)
```

### 3.5 seed（随机种子）

seed参数是o3和部分GPT-4o版本支持的高级功能，用于实现确定性输出。

**核心用途：**

当设置相同的seed值时，相同的输入会得到相同（或高度相似）的输出。这对于以下场景非常有用：测试和调试，确保结果可复现；批处理任务，保证相同输入的一致性；需要确定性但又希望保持一定随机性的应用。

```python
# 使用seed实现确定性输出
for i in range(3):
    response = openai.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": "抛硬币的结果是什么？"}],
        seed=42,
        temperature=0.7
    )
    print(f"第{i+1}次: {response.choices[0].message.content}")
# 三次输出应该相同或非常接近
```

## 四、函数调用（Function Calling）

### 4.1 函数调用概述

Function Calling是OpenAI API的重要能力，允许模型调用开发者定义的外部函数，扩展AI的功能边界。通过函数调用，AI可以执行代码、查询数据库、调用第三方API、执行文件操作等。

### 4.2 函数定义格式

函数通过JSON Schema格式定义，包含名称、描述和参数规范。

```json
{
  "name": "get_weather",
  "description": "获取指定城市的天气信息",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {
        "type": "string",
        "description": "城市名称，如'北京'、'Shanghai'"
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

### 4.3 函数调用工作流

函数调用的完整流程包括：定义函数、向模型提交函数列表、解析模型的函数调用请求、执行函数、将结果返回给模型。

```python
import openai
import json

# 第一步：定义可用的函数
functions = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"]
                    }
                },
                "required": ["city"]
            }
        }
    }
]

# 第二步：发送请求，包含函数定义
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "user", "content": "北京今天天气怎么样？"}
    ],
    tools=functions
)

# 第三步：解析模型返回的函数调用
tool_calls = response.choices[0].message.tool_calls
if tool_calls:
    for call in tool_calls:
        if call.function.name == "get_weather":
            args = json.loads(call.function.arguments)
            print(f"调用函数: get_weather, 参数: {args}")
            
            # 第四步：执行函数（这里模拟）
            weather_result = {"temperature": 22, "condition": "晴", "humidity": 45}
            
            # 第五步：返回结果给模型
            second_response = openai.chat.completions.create(
                model="gpt-4o",
                messages=[
                    {"role": "user", "content": "北京今天天气怎么样？"},
                    {"role": "assistant", "tool_calls": tool_calls},
                    {
                        "role": "tool",
                        "tool_call_id": call.id,
                        "content": json.dumps(weather_result)
                    }
                ],
                tools=functions
            )
            print(second_response.choices[0].message.content)
```

### 4.4 并行函数调用

GPT-4o支持并行函数调用，允许模型在一个响应中发起多个函数调用请求，显著提升效率。

```python
functions = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "parameters": {"type": "object", "properties": {"city": {"type": "string"}}}
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_news",
            "parameters": {"type": "object", "properties": {"category": {"type": "string"}}}
        }
    }
]

response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "给我北京天气和科技新闻"}],
    tools=functions
)

# 模型可能同时发起两个函数调用
for call in response.choices[0].message.tool_calls:
    print(f"函数: {call.function.name}, 参数: {call.function.arguments}")
```

## 五、视觉输入（Vision）

### 5.1 Vision能力概述

GPT-4o内置强大的视觉理解能力，可以分析图片内容、识别图表、读取文档、描述场景等。Vision支持单图和多图输入，支持多种图片格式。

### 5.2 图片输入格式

图片可以通过URL或Base64编码两种方式传递给API。

```python
import openai
import base64

# 方式一：通过URL输入
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": "https://example.com/chart.png",
                        "detail": "high"  # low/high/auto
                    }
                }
            ]
        }
    ]
)

# 方式二：通过Base64输入
with open("image.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode("utf-8")

response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "image_url",
                    "image_url": {
                        "url": f"data:image/png;base64,{image_data}"
                    }
                }
            ]
        }
    ]
)
```

### 5.3 detail参数详解

detail参数控制图片处理的详细程度：

- **low**：快速处理，降低分辨率，适合简单识别任务
- **high**：高分辨率处理，保持原图质量，适合复杂分析
- **auto**：自动选择，模型根据任务判断

```python
# 简单识别 - 使用low detail降低成本
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "这张图片的主要颜色是什么？"},
                {"type": "image_url", "image_url": {"url": "https://example.com/colorful.jpg", "detail": "low"}}
            ]
        }
    ]
)

# 复杂分析 - 使用high detail
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "请详细分析这张医学影像，标注可能的异常区域。"},
                {"type": "image_url", "image_url": {"url": "https://example.com/xray.jpg", "detail": "high"}}
            ]
        }
    ]
)
```

## 六、Token计算与成本控制

### 6.1 Token机制详解

Token是大语言模型处理文本的基本单位。英文中，一个Token约等于4个字符或0.75个单词；中文中，一个Token通常对应1到2个汉字。

**Token计算规则：**

OpenAI使用Tiktoken库进行Token计算。特殊符号、空白字符、标点符号都会占用Token。API响应中的usage对象会报告实际消耗的Token数量。

```python
import tiktoken

# 使用cl100k_base编码器（GPT-4o使用）
encoding = tiktoken.get_encoding("cl100k_base")

text = "Hello, world! 你好世界"
tokens = encoding.encode(text)
print(f"文本: {text}")
print(f"Token数: {len(tokens)}")
print(f"Token IDs: {tokens}")
```

### 6.2 Token消耗优化策略

**Prompt压缩技术：**

使用更简洁的表达方式，避免冗余描述。例如将"请仔细阅读以下内容，然后给出一个详细的总结，包括所有重要观点"简化为"总结以下内容"。

**上下文窗口高效利用：**

在长对话场景中，定期总结和压缩对话历史。保留核心信息，删除中间过程的冗余内容。

```python
def compress_conversation(messages, max_turns=10):
    """压缩对话历史，保留最近N轮"""
    if len(messages) <= max_turns * 2 + 1:  # system + max_turns*2
        return messages
    
    # 保留系统提示和最近对话
    system = messages[0] if messages[0]["role"] == "system" else None
    recent = messages[-max_turns * 2:]
    
    if system:
        return [system] + recent
    return recent
```

### 6.3 成本计算示例

```python
def calculate_cost(model, input_tokens, output_tokens):
    """计算API调用成本"""
    pricing = {
        "gpt-4o": {"input": 2.5, "output": 10.0},
        "gpt-4o-mini": {"input": 0.15, "output": 0.6},
        "o3": {"input": 4.0, "output": 16.0},
        "o3-mini": {"input": 0.55, "output": 2.2}
    }
    
    if model not in pricing:
        return "未知模型"
    
    input_cost = (input_tokens / 1_000_000) * pricing[model]["input"]
    output_cost = (output_tokens / 1_000_000) * pricing[model]["output"]
    
    return {
        "input_cost": f"${input_cost:.4f}",
        "output_cost": f"${output_cost:.4f}",
        "total_cost": f"${input_cost + output_cost:.4f}"
    }

# 示例计算
result = calculate_cost("gpt-4o", 5000, 2000)
print(f"成本明细: {result}")
```

## 七、进阶配置与最佳实践

### 7.1 流式输出配置

流式输出允许模型实时返回生成内容，适合需要即时反馈的应用。

```python
import openai

stream = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "写一首关于星辰的诗"}],
    stream=True
)

full_content = ""
for chunk in stream:
    if chunk.choices[0].delta.content:
        content = chunk.choices[0].delta.content
        print(content, end="", flush=True)
        full_content += content
```

### 7.2 Logit Bias（Logit偏置）

logit_bias允许对特定Token施加正向或负向偏置，影响其被选中的概率。

```python
# 强制避免生成某些敏感词
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "描述人工智能"}],
    logit_bias={  # 降低某些token的概率
        1234: -5.0,  # token ID 1234
        5678: -5.0
    }
)
```

### 7.3 响应格式控制

通过response_format参数可以控制输出格式。

```python
# 强制JSON输出
response = openai.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "你是一个JSON生成器，只输出有效的JSON。"},
        {"role": "user", "content": "返回用户信息：姓名、年龄、职业"}
    ],
    response_format={"type": "json_object"}
)
```

## 八、相关资源与扩展阅读

- [[OpenAI API 官方文档]]
- [[GPT_API实战代码]]
- [[Anthropic Claude API指南]]
- [[Gemini API完整指南]]
- [[API成本优化策略]]

---

*本文档最后更新于2026年4月*
