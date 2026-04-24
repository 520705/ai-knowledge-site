---
title: 国内大模型API对比
date: 2026-04-18
tags:
  - 国内模型
  - 通义千问
  - Kimi
  - 文心一言
  - 智谱GLM
  - 讯飞星火
  - API
categories:
  - AI工具实操
  - 大模型调用
alias: [china-llm-api, domestic-llm, qwen-api, kimi-api, ernie-api]
---

> [!info] 文档信息
> 本文档对国内主流大模型API进行全面对比，包括通义千问、Kimi、文心一言、智谱GLM和讯飞星火。

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| 通义千问 | 阿里云大模型 |
| Kimi | 月之暗面Moonshot |
| 文心一言 | 百度ERNIE |
| 智谱GLM | 智谱AI ChatGLM |
| 讯飞星火 | 科大讯飞Spark |
| 开源模型 | 国内开源模型 |
| 长上下文 | 超长上下文能力 |
| 多模态 | 图文音视频 |
| API定价 | 价格对比 |
| 调用限制 | 速率限制 |

---

## 一、国内大模型生态概览

2024-2026年是中国大模型蓬勃发展的关键时期。在政策和市场的双重推动下，国内涌现出众多优秀的大模型公司和产品。这些模型在中文理解、特定行业应用和合规性方面各有优势，成为企业数字化转型的重要工具。

国内大模型厂商主要分为以下几类：

| 类别 | 代表厂商 | 特点 |
|------|----------|------|
| 互联网巨头 | 阿里、百度、腾讯 | 资源丰富，生态完善 |
| AI独角兽 | 智谱AI、月之暗面 | 技术专注，创新能力强 |
| 传统AI企业 | 科大讯飞、商汤 | 垂直领域积累深厚 |
| 新兴创业公司 | 零一万物、百川 | 灵活敏捷，开源积极 |

## 二、通义千问（Qwen）

### 2.1 产品系列

通义千问是阿里云推出的大语言模型系列，基于Transformer架构，在中文理解和代码生成方面表现优异。

| 模型 | 参数量 | 上下文 | 特点 |
|------|--------|--------|------|
| Qwen-Max | 超大规模 | 32K | 旗舰模型 |
| Qwen-Plus | 大规模 | 128K | 主流通用 |
| Qwen-Turbo | 中等规模 | 8K | 快速响应 |
| Qwen-Long | - | 1M | 超长上下文 |
| Qwen2.5 | 多尺寸 | 128K | 开源版本 |

### 2.2 API调用

```python
import dashscope
from dashscope import Generation

# 配置API Key
dashscope.api_key = "your-api-key"

# 基础调用
response = Generation.call(
    model=Generation.Models.qwen_max,
    prompt="解释量子计算的基本原理"
)

print(response.output.text)
```

```python
# 使用SDK
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
)

response = client.chat.completions.create(
    model="qwen-max",
    messages=[
        {"role": "user", "content": "写一段Python代码实现Web服务器"}
    ]
)
```

### 2.3 开源版本

```python
# Ollama运行Qwen
# ollama pull qwen2.5:72b

from langchain_community.llms import Ollama

llm = Ollama(model="qwen2.5:72b")
response = llm.invoke("解释什么是机器学习")
```

### 2.4 价格

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| Qwen-Max | ¥0.04/千tokens | ¥0.12/千tokens |
| Qwen-Plus | ¥0.004/千tokens | ¥0.012/千tokens |
| Qwen-Turbo | ¥0.002/千tokens | ¥0.006/千tokens |

## 三、Kimi（月之暗面）

### 3.1 核心优势

Kimi由月之暗面（Moonshot AI）开发，以其超长上下文能力和出色的长文本处理著称。Kimi-200K版本支持20万token的上下文窗口，在长文档分析、代码库理解等任务上表现突出。

| 特性 | 说明 |
|------|------|
| 上下文 | 200K tokens |
| 长文本 | 支持200万字文档 |
| 多模态 | 支持图片理解 |
| 联网搜索 | 实时信息获取 |
| 文件处理 | PDF、Word、Excel |

### 3.2 API调用

```python
from openai import OpenAI

client = OpenAI(
    api_key="your-api-key",
    base_url="https://api.moonshot.cn/v1"
)

# 基础调用
response = client.chat.completions.create(
    model="moonshot-v1-128k",
    messages=[
        {"role": "user", "content": "分析这份文档的核心观点"}
    ]
)

print(response.choices[0].message.content)

# 处理长文档
def analyze_long_document(file_path):
    with open(file_path, 'r', encoding='utf-8') as f:
        content = f.read()
    
    response = client.chat.completions.create(
        model="moonshot-v1-128k",
        messages=[
            {"role": "system", "content": "你是一位文档分析专家。"},
            {"role": "user", "content": f"请详细分析以下文档：\n\n{content}"}
        ],
        temperature=0.7
    )
    return response.choices[0].message.content
```

### 3.3 支持的模型

| 模型 | 上下文 | 适用场景 |
|------|--------|----------|
| moonshot-v1-8k | 8K | 快速问答 |
| moonshot-v1-32k | 32K | 标准对话 |
| moonshot-v1-128k | 128K | 长文档分析 |
| moonshot-v1-auto | 自动 | 智能选择 |

### 3.4 价格

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| moonshot-v1-8k | ¥0.006/千tokens | ¥0.012/千tokens |
| moonshot-v1-32k | ¥0.012/千tokens | ¥0.024/千tokens |
| moonshot-v1-128k | ¥0.03/千tokens | ¥0.06/千tokens |

## 四、文心一言（ERNIE Bot）

### 4.1 百度ERNIE体系

文心一言是百度基于ERNIE（Enhanced Representation through Knowledge Integration）系列模型开发的生成式AI产品。ERNIE系列在知识图谱增强和中文理解方面有深厚积累。

| 模型 | 特点 | 上下文 |
|------|------|--------|
| ERNIE-4.0 | 旗舰模型 | 32K |
| ERNIE-3.5-Pro | 高性能 | 32K |
| ERNIE-Speed | 快速响应 | 8K |
| ERNIE-Lite | 轻量级 | 8K |

### 4.2 API调用

```python
import erniebot

# 配置
erniebot.api_type = "aistudio"
erniebot.access_token = "your-access-token"

# 基础调用
response = erniebot.ChatCompletion.create(
    model="ernie-4.0-8k",
    messages=[{"role": "user", "content": "解释区块链技术"}]
)

print(response.get_result())
```

```python
# 千帆平台调用
from qianfan import ChatCompletion

# 使用安全认证
chat = ChatCompletion()

response = chat.do(
    model="ERNIE-4.0-8K",
    messages=[{"role": "user", "content": "写一篇产品介绍"}]
)

print(response.body)
```

### 4.3 价格

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| ERNIE-4.0 | ¥0.12/千tokens | ¥0.12/千tokens |
| ERNIE-3.5-Pro | ¥0.02/千tokens | ¥0.02/千tokens |
| ERNIE-Speed | ¥0.004/千tokens | ¥0.008/千tokens |
| ERNIE-Lite | ¥0 | ¥0 |

> [!note] 免费额度
> ERNIE-Lite提供免费调用额度，适合个人开发和小规模应用。

## 五、智谱GLM（ChatGLM）

### 5.1 GLM系列

智谱AI的ChatGLM系列是国内最具影响力的开源大模型之一，GLM-4在多项基准测试中表现优异。

| 模型 | 参数量 | 上下文 | 类型 |
|------|--------|--------|------|
| GLM-4 | 超大规模 | 128K | 闭源API |
| GLM-4V | 多模态 | 128K | 图文理解 |
| GLM-3-Turbo | 大规模 | 128K | 高性价比 |
| ChatGLM3-6B | 6B | 128K | 开源本地 |
| GLM-4-AllTools | Agent | 128K | 工具调用 |

### 5.2 API调用

```python
from zhipuai import ZhipuAI

client = ZhipuAI(api_key="your-api-key")

# 基础调用
response = client.chat.completions.create(
    model="glm-4",
    messages=[
        {"role": "user", "content": "介绍人工智能的发展历程"}
    ]
)

print(response.choices[0].message.content)

# 多轮对话
def chat_session():
    messages = [
        {"role": "system", "content": "你是一位专业的技术顾问。"}
    ]
    
    while True:
        user_input = input("你: ")
        if user_input.lower() in ["exit", "quit"]:
            break
        
        messages.append({"role": "user", "content": user_input})
        
        response = client.chat.completions.create(
            model="glm-4",
            messages=messages
        )
        
        assistant_msg = response.choices[0].message.content
        messages.append({"role": "assistant", "content": assistant_msg})
        print(f"助手: {assistant_msg}")
```

### 5.3 开源版本本地部署

```python
# 使用Transformers运行ChatGLM3
from transformers import AutoTokenizer, AutoModelForCausalLM

model_path = "THUDM/chatglm3-6b"

tokenizer = AutoTokenizer.from_pretrained(
    model_path, 
    trust_remote_code=True
)
model = AutoModelForCausalLM.from_pretrained(
    model_path,
    trust_remote_code=True,
    device_map="auto"
)

input_text = "请解释什么是深度学习"
inputs = tokenizer(input_text, return_tensors="pt").to(model.device)
outputs = model.generate(**inputs, max_new_tokens=512)
print(tokenizer.decode(outputs[0], skip_special_tokens=True))
```

### 5.4 价格

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| GLM-4 | ¥0.1/千tokens | ¥0.1/千tokens |
| GLM-4V | ¥0.1/千tokens | ¥0.1/千tokens |
| GLM-3-Turbo | ¥0.001/千tokens | ¥0.001/千tokens |

## 六、讯飞星火（Spark）

### 6.1 讯飞优势

讯飞星火认知大模型是科大讯飞推出的AI产品，在语音交互、教育、医疗等领域有深厚积累。其语音合成和语音识别能力与大模型深度整合。

| 模型 | 特点 | 上下文 |
|------|------|--------|
| Spark4.0 Ultra | 旗舰 | 128K |
| Spark3.5 Pro | 高性能 | 32K |
| Spark3.5 Max | 标准 | 32K |
| Spark3.5 | 基础 | 8K |
| Spark Lite | 轻量 | 8K |

### 6.2 API调用

```python
import sparkai

# 配置
spark = sparkai.SparkAPI(
    app_id="your-app-id",
    api_key="your-api-key",
    api_secret="your-api-secret",
    spark_url="wss://spark-api.xf-yun.com/v4.0/chat"
)

# 同步调用
response = spark.chat(
    model="generalv3.5",
    messages=[{"role": "user", "content": "请解释语音识别技术"}]
)

print(response.text)
```

```python
# 异步调用
import asyncio

async def async_chat():
    response = await spark.achat(
        model="generalv3.5",
        messages=[{"role": "user", "content": "分析大数据技术趋势"}]
    )
    return response

result = asyncio.run(async_chat())
```

### 6.3 价格

| 模型 | 输入价格 | 输出价格 |
|------|----------|----------|
| Spark4.0 Ultra | ¥0.15/千tokens | ¥0.15/千tokens |
| Spark3.5 Pro | ¥0.03/千tokens | ¥0.06/千tokens |
| Spark3.5 Max | ¥0.015/千tokens | ¥0.03/千tokens |
| Spark Lite | ¥0 | ¥0 |

## 七、综合对比

### 7.1 能力对比

| 模型 | 中文能力 | 代码能力 | 数学推理 | 长上下文 | 多模态 |
|------|----------|----------|----------|----------|--------|
| 通义千问 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Kimi | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 文心一言 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 智谱GLM | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 讯飞星火 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

### 7.2 价格对比

| 模型 | 相对价格 | 免费额度 | 推荐场景 |
|------|----------|----------|----------|
| 通义千问-Turbo | ★☆☆☆☆ | 有 | 快速响应 |
| 通义千问-Plus | ★★☆☆☆ | 有 | 日常使用 |
| Kimi-8K | ★★☆☆☆ | 有 | 快速问答 |
| Kimi-128K | ★★★☆☆ | 无 | 长文档 |
| 文心一言-Lite | ★☆☆☆☆ | 完全免费 | 尝鲜体验 |
| 智谱GLM-Turbo | ★☆☆☆☆ | 有 | 高性价比 |
| 讯飞星火-Lite | ★☆☆☆☆ | 完全免费 | 语音应用 |

> [!tip] 选择建议
> - 追求性价比：选择智谱GLM-Turbo或通义千问-Plus
> - 长文档处理：选择Kimi-128K或GLM-4
> - 语音应用：选择讯飞星火
> - 快速尝鲜：选择文心一言Lite或讯飞星火Lite

## 八、调用限制

### 8.1 速率限制

| 平台 | 免费用户 | 付费用户 |
|------|----------|----------|
| 阿里云 | 60 QPM | 可申请提升 |
| Kimi | 3 QPM | 120 QPM起 |
| 百度 | 60 QPM | 按量付费无限制 |
| 智谱AI | 60 QPM | 可申请提升 |
| 讯飞星火 | 60 QPM | 企业版无限制 |

### 8.2 最佳实践

```python
import time
import asyncio
from ratelimit import limits

class LLMCaller:
    def __init__(self, api_type):
        self.api_type = api_type
        self.last_call_time = 0
        self.min_interval = 1.0  # 最小调用间隔（秒）
    
    async def call_with_rate_limit(self, prompt):
        # 检查速率限制
        now = time.time()
        elapsed = now - self.last_call_time
        if elapsed < self.min_interval:
            await asyncio.sleep(self.min_interval - elapsed)
        
        self.last_call_time = time.time()
        
        # 调用API
        if self.api_type == "kimi":
            return await self.call_kimi(prompt)
        elif self.api_type == "qwen":
            return await self.call_qwen(prompt)
        # ... 其他模型
    
    async def batch_process(self, prompts, concurrency=5):
        semaphore = asyncio.Semaphore(concurrency)
        
        async def limited_call(prompt):
            async with semaphore:
                return await self.call_with_rate_limit(prompt)
        
        results = await asyncio.gather(*[limited_call(p) for p in prompts])
        return results
```

## 九、相关资源

- [[DeepSeek_API使用]] - DeepSeek API
- [[Claude_API指南]] - Claude API
- [[Gemini_API完整指南]] - Gemini API
- [[API成本优化策略]] - 成本优化

---

> [!success] 完成状态
> 本文已完成国内五大主流大模型API的全面对比分析，涵盖产品系列、API调用、价格和选型建议。
