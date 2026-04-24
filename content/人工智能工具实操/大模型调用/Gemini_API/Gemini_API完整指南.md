---
title: Google Gemini API完整指南
date: 2026-04-18
tags:
  - Gemini
  - Google
  - API
  - 大模型
  - 多模态
categories:
  - AI工具实操
  - 大模型调用
alias: [gemini-api-guide, google-ai-api]
---

> [!info] 文档信息
> 本文档详细介绍Google Gemini API的使用方法，包括2.0与2.5版本对比、1M超长上下文、Native Multimodal能力以及音频处理功能。

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| Gemini 2.0 | Google最新一代多模态模型 |
| Gemini 2.5 | 增强版，推理能力大幅提升 |
| 1M上下文 | 100万token超长上下文窗口 |
| Native Multimodal | 原生多模态设计 |
| Flash | 轻量快速版本 |
| Pro | 中端主力版本 |
| Audio | 音频理解和生成 |
| Function Calling | 函数调用能力 |
| Context Caching | 上下文缓存 |
| API格式 | REST/SDK调用 |

---

## 一、Gemini API概述

Google Gemini是Google DeepMind开发的大型语言模型系列，代表了Google在AI领域的技术结晶。Gemini从设计之初就采用原生多模态架构，能够同时理解和生成文本、代码、图像、音频和视频内容。与其他需要外挂模块实现多模态的模型不同，Gemini的多模态能力是内置在模型核心中的，这使其在跨模态任务中表现更为出色。

Gemini系列经历了多次重大更新。Gemini 1.0于2023年12月发布，Gemini 1.5在2024年2月推出，带来了突破性的长上下文能力。2024年底，Gemini 2.0正式发布，实现了性能和功能的双重飞跃。2025年，Gemini 2.5进一步增强了推理能力，成为当前最强大的模型之一。

## 二、Gemini 2.0与2.5版本对比

### 2.1 架构升级

Gemini 2.0系列采用了全新的Transformer架构优化，在保持低延迟的同时提升了吞吐量。2.5版本则在2.0基础上引入了增强的推理机制，能够进行更深入的多步骤思考。

| 特性 | Gemini 2.0 | Gemini 2.5 |
|------|------------|-------------|
| 上下文窗口 | 1M tokens | 1M tokens |
| 多模态支持 | 原生 | 原生增强 |
| 推理模式 | 标准 | 思考模式 |
| 音频理解 | 支持 | 增强 |
| 工具使用 | Function Calling | 高级Function Calling |
| 响应速度 | 快速 | 中等 |
| 价格定位 | 中端 | 高端 |

### 2.2 能力提升

Gemini 2.5相比2.0在多个维度有明显提升：

**推理能力**：Gemini 2.5引入了"思考模式"，模型会在生成答案前进行多步骤推理，类似于人类的思考过程。这使得复杂数学问题、逻辑推理和代码调试能力大幅提升。在MATH基准测试中，2.5版本得分从2.0的78%提升到了92%。

**代码生成**：2.5版本的代码生成能力达到了新的高度，支持更长更复杂的代码生成任务，代码质量和可执行性都有所提高。支持的编程语言超过20种，包括Python、JavaScript、TypeScript、Go、Rust等主流语言。

**长文本理解**：虽然两者都支持100万token上下文，但2.5版本在长文档理解上表现更稳定，能够准确定位和引用文档中的具体内容。

```python
# Gemini 2.5 思考模式示例
import google.generativeai as genai

genai.configure(api_key="your-api-key")

model = genai.GenerativeModel(
    model_name='gemini-2.5-pro-preview-06-05',
    generation_config={
        'thinking_config': {
            'thinking_mode': 'thoughts_only'  # 启用思考模式
        }
    }
)

response = model.generate_content(
    "分析这道数学题：已知一个等差数列的首项为3，"
    "公差为5，前n项和为Sn，若S10=280，求n的取值范围。"
)
print(response.thoughts)  # 查看推理过程
print(response.text)      # 查看最终答案
```

### 2.3 版本选择建议

> [!note] 版本选择指南
> - 日常任务、快速响应：选择Gemini 2.0 Flash
> - 复杂推理、长文档分析：选择Gemini 2.5 Pro
> - 需要最佳效果且预算充足：选择Gemini 2.5 Ultra

## 三、1M超长上下文详解

Gemini最引以为傲的特性之一是支持100万token的超长上下文窗口，这在实际应用中有革命性意义。

### 3.1 上下文规模对比

| 模型 | 上下文窗口 |
|------|-----------|
| GPT-4o | 128K |
| Claude 3.7 | 200K |
| Gemini 2.0/2.5 | 1M |
| Gemini 1.5 Pro | 1M |

100万token大约等于：
- 75万英文单词
- 50万中文字符
- 10小时视频的字幕
- 完整代码库（如Linux内核）

### 3.2 长上下文应用场景

**代码库分析**：可以将整个代码仓库作为上下文输入，让AI理解代码的全貌和细节。这对于代码审查、架构分析、迁移规划等任务非常有用。

```python
# 分析整个代码库
def analyze_codebase(repo_path):
    # 读取整个代码库内容
    all_files = []
    for root, dirs, files in os.walk(repo_path):
        for file in files:
            if file.endswith(('.py', '.js', '.ts', '.go')):
                path = os.path.join(root, file)
                with open(path, 'r', encoding='utf-8') as f:
                    all_files.append(f"# 文件: {path}\n{f.read()}")
    
    codebase_content = "\n\n".join(all_files)
    
    model = genai.GenerativeModel('gemini-2.0-pro')
    response = model.generate_content(
        f"请分析以下代码库的整体架构、主要模块和依赖关系：\n\n{codebase_content}"
    )
    return response.text
```

**长文档处理**：可以一次性处理整本书籍、长篇报告或法律文档，无需分段或摘要。

**多轮对话记忆**：1M上下文使得AI可以记住极长的对话历史，适合需要持续交互的应用场景。

### 3.3 上下文缓存优化

为了降低长上下文的成本，Gemini提供了上下文缓存功能：

```python
# 使用上下文缓存
from google.ai import generativelanguage as ga

# 创建缓存
cache = genai.create_tuned_model(
    source_model='models/gemini-1.5-pro-001',
    training_data=[
        # 预加载的上下文数据
        {"text_input": "系统设定：你是某领域的专家..."}
    ],
    epoch_count=1,
    batch_size=1,
    learning_rate=0.001,
)

# 使用缓存进行多次调用
for query in queries:
    response = model.generate_content(
        contents=[{"parts": [{"text": f"上下文ID: {cache.name}\n问题: {query}"}]}]
    )
```

> [!tip] 成本优化
> 上下文缓存特别适合以下场景：系统提示词固定、参考文档不变、多用户共享基础上下文。

## 四、Native Multimodal能力

Gemini的原生多模态设计意味着它从底层架构就支持多种模态的处理，而不是像某些模型那样通过拼接不同模块实现。

### 4.1 多模态输入

```python
# 图像理解
from PIL import Image
import urllib.request
import io

# 从URL加载图片
url = "https://example.com/diagram.png"
image_data = urllib.request.urlopen(url).read()
image = Image.open(io.BytesIO(image_data))

model = genai.GenerativeModel('gemini-2.0-pro-vision')

response = model.generate_content([
    image,
    "请详细分析这张图表，提取所有关键数据点和趋势"
])

# 本地图片处理
image = Image.open("screenshot.png")
response = model.generate_content([
    image,
    "这张截图显示了什么内容？有哪些可交互元素？"
])
```

### 4.2 视频理解

Gemini可以直接处理视频内容，提取帧信息和时间序列特征：

```python
import moviepy.editor as mp

# 提取视频帧
video = mp.VideoFileClip("presentation.mp4")

# 每10秒提取一帧
frames = []
for t in range(0, int(video.duration), 10):
    frame = video.get_frame(t)
    frames.append(Image.fromarray(frame))

# 批量发送给Gemini
model = genai.GenerativeModel('gemini-2.0-pro-vision')
response = model.generate_content([
    *frames,
    "这是一个演示视频的帧。请总结视频的主要内容和关键信息。"
])
```

### 4.3 图文混合理解

```python
# 同时输入图像和文本
response = model.generate_content([
    "请比较以下两款产品的特点：",
    Image.open("product_a.png"),
    Image.open("product_b.png"),
    "\n从外观、功能、价格三个维度进行分析。"
])
```

## 五、音频处理能力

Gemini 2.0系列增强了原生音频处理能力，可以直接理解和分析音频内容。

### 5.1 音频输入

```python
# 音频理解
import base64

def transcribe_and_analyze(audio_file):
    with open(audio_file, 'rb') as f:
        audio_data = base64.b64encode(f.read()).decode()
    
    model = genai.GenerativeModel('gemini-2.0-pro')
    
    response = model.generate_content({
        "parts": [{
            "inline_data": {
                "mime_type": "audio/wav",
                "data": audio_data
            }
        }, {
            "text": "请转录这段音频的内容，并总结主要观点。"
        }]
    })
    return response.text

# 会议录音分析
meeting_analysis = transcribe_and_analyze("meeting_recording.wav")
print(meeting_analysis)
```

### 5.2 音频生成

Gemini 2.0支持音频输出，可以通过Text-to-Speech生成语音内容：

```python
# 音频生成
response = model.generate_content(
    "用简洁的语言总结今日新闻要点，然后用中文朗读出来。",
    generation_config={
        "response_modalities": ["TEXT", "AUDIO"]
    }
)

# 获取音频数据
if hasattr(response, 'audio'):
    audio_bytes = response.audio
    with open("summary.mp3", "wb") as f:
        f.write(audio_bytes)
```

### 5.3 应用场景

| 场景 | 说明 |
|------|------|
| 会议记录 | 自动转录、总结、提取待办事项 |
| 播客分析 | 内容理解、关键信息提取 |
| 语音助手 | 语音交互、指令执行 |
| 有声内容 | 文本转语音、语音合成 |
| 音乐分析 | 音频特征提取、风格识别 |

## 六、Function Calling与工具使用

### 6.1 基础函数调用

```python
# 定义可调用的函数
tools = [{
    "function_declarations": [{
        "name": "get_weather",
        "description": "获取指定城市的天气信息",
        "parameters": {
            "type": "object",
            "properties": {
                "location": {
                    "type": "string",
                    "description": "城市名称"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "温度单位"
                }
            },
            "required": ["location"]
        }
    }]
}]

model = genai.GenerativeModel(
    'gemini-2.0-pro',
    tools=tools
)

response = model.generate_content(
    "北京今天天气怎么样？需要带伞吗？"
)

# 处理函数调用
for candidate in response.candidates:
    for part in candidate.content.parts:
        if part.function_call:
            fc = part.function_call
            print(f"调用函数: {fc.name}")
            print(f"参数: {fc.args}")
```

### 6.2 链式函数调用

Gemini支持复杂的函数调用链，可以根据结果决定下一步操作：

```python
# 多步骤任务处理
def process_user_request(user_message):
    model = genai.GenerativeModel(
        'gemini-2.0-pro',
        tools=[web_search_tool, database_tool, email_tool]
    )
    
    chat = model.start_chat()
    response = chat.send_message(user_message)
    
    # 持续处理函数调用直到完成
    while response.candidates and hasattr(response.candidates[0].content.parts[0], 'function_call'):
        for part in response.candidates[0].content.parts:
            if part.function_call:
                result = execute_function(part.function_call)
                response = chat.send_message(
                    FunctionResponse(
                        name=part.function_call.name,
                        response=result
                    )
                )
    
    return response.text
```

## 七、API调用详解

### 7.1 Python SDK

```python
import google.generativeai as genai
import os

# 配置
genai.configure(api_key=os.environ.get("GOOGLE_API_KEY"))

# 列出可用模型
for m in genai.list_models():
    if 'generateContent' in m.supported_generation_methods:
        print(f"{m.name}: {m.description}")

# 模型选择
model = genai.GenerativeModel('gemini-2.0-pro')

# 基础调用
response = model.generate_content("请介绍一下人工智能的发展历史")
print(response.text)

# 流式输出
for chunk in model.generate_content(
    "详细解释量子计算原理",
    stream=True
):
    print(chunk.text, end='', flush=True)
```

### 7.2 JavaScript/Node.js SDK

```javascript
import { GoogleGenerativeAI } from "@google/generative-ai";

const genAI = new GoogleGenerativeAI(process.env.GOOGLE_API_KEY);

async function main() {
    const model = genAI.getGenerativeModel({ model: "gemini-2.0-pro" });
    
    // 基础调用
    const result = await model.generateContent(
        "解释什么是机器学习"
    );
    console.log(result.response.text());
    
    // 流式输出
    const streamResult = await model.generateContentStream(
        "详细说明区块链技术"
    );
    for await (const chunk of streamResult.stream) {
        process.stdout.write(chunk.text());
    }
}
```

## 八、2026年4月定价

| 模型 | 输入价格 | 输出价格 | 上下文 |
|------|----------|----------|--------|
| Gemini 2.5 Ultra | $1.25/百万tokens | $5/百万tokens | 1M |
| Gemini 2.5 Pro | $0.75/百万tokens | $3/百万tokens | 1M |
| Gemini 2.0 Flash | $0.10/百万tokens | $0.40/百万tokens | 1M |
| Gemini 2.0 Pro | $0.50/百万tokens | $2/百万tokens | 1M |

> [!note] 免费额度
> Gemini API提供免费层级，每月包含一定量的免费token，适合开发和测试使用。

## 九、相关资源

- [[Anthropic_Claude_API指南]] - Claude API对比
- [[GPT_API完整指南]] - OpenAI GPT系列
- [[DeepSeek_API使用]] - DeepSeek API
- [[国内大模型API对比]] - 国内模型
- [[多模态大模型详解]] - 多模态能力对比

---

> [!success] 完成状态
> 本文档已完成Gemini API的全面介绍，涵盖2.0/2.5版本对比、1M上下文、多模态能力、音频处理等核心内容。
