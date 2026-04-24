---
title: n8n与LLM集成：AI能力深度应用
date: 2026-04-24
tags:
  - n8n
  - LLM集成
  - AI工作流
  - Prompt工程
  - 记忆系统
  - Agent
  - 工具调用
categories:
  - 智能体搭建
  - n8n平台
description: 深入讲解n8n平台中LLM节点的配置与使用，包括Prompt工程、Memory记忆管理、Tool工具调用、Agent自主决策等核心能力，帮助读者构建强大的AI工作流。
---

# n8n与LLM集成：AI能力深度应用

> [!NOTE] 这篇指南讲什么
> 这是n8n AI能力的进阶教程，重点讲解LLM节点、Prompt工程、Memory记忆、Tool工具、Agent自主决策等核心功能。适合有一定n8n基础，想深入学习AI工作流的读者。

## AI节点体系概述

n8n提供了完整的AI能力组件，构成了一个模块化的AI系统：

```
用户输入
    ↓
┌─────────────────┐
│   LLM Chain     │ ← 调用大语言模型
└────────┬────────┘
         ↓
┌────────┴────────┐
│     Agent       │ ← 自主决策执行
└────────┬────────┘
         ↓
    ┌────┴────┐
    ↓         ↓
  Tools    Memory
    ↓         ↓
外部API   向量存储
```

**核心节点：**

| 节点类型 | 功能 | 使用场景 |
|----------|------|----------|
| LLM Chain | 调用大语言模型 | 文本生成、翻译、摘要 |
| Prompt | 构建结构化提示词 | 角色扮演、任务分解 |
| Memory | 管理对话历史 | 多轮对话、上下文理解 |
| Tool | 定义外部工具 | 搜索、计算、API调用 |
| Agent | 自主决策执行 | 复杂任务自动化 |
| Embedding | 文本向量化 | 语义搜索、RAG |

## LLM节点详解

### 节点配置

LLM Chain节点是n8n调用大语言模型的核心入口。

**基础配置：**

```yaml
节点配置:
  # 模型提供商
  provider: OpenAI
  
  # 具体模型
  model: gpt-4o
  
  # 认证方式
  credentials:
    apiKey: "{{ $credentials.openAiApi }}"
  
  # 消息配置
  messages:
    - role: system
      content: |
        你是一个专业的技术文档助手。
        擅长用简洁清晰的语言解释复杂概念。
        回答时使用Markdown格式。
    
    - role: user  
      content: "{{ $json.userMessage }}"
```

**高级参数：**

```yaml
options:
  temperature: 0.7      # 创造性：0-2，越高越有创意
  maxTokens: 1000      # 最大生成token数
  topP: 1.0            # 采样策略：控制随机性
  frequencyPenalty: 0    # 频率惩罚：降低重复
  presencePenalty: 0    # 存在惩罚：鼓励新话题
```

### 模型选择指南

| 模型 | 特点 | 适用场景 | 成本 |
|------|------|----------|------|
| GPT-4o | 最强综合能力 | 复杂推理、代码生成 | 高 |
| GPT-4o-mini | 性价比优 | 日常对话、简单任务 | 中 |
| Claude 3.5 Sonnet | 长文本处理强 | 文档分析、长文写作 | 中高 |
| Gemini 1.5 Pro | 多模态支持 | 图像理解、跨模态 | 中 |
| DeepSeek-V3 | 国产开源 | 中文场景 | 低 |

**选择建议：**
- 简单任务（翻译、总结）→ GPT-4o-mini 或 DeepSeek
- 复杂推理（代码、分析）→ GPT-4o 或 Claude 3.5
- 长文本处理 → Claude 3.5 Sonnet
- 多模态任务 → Gemini 1.5 Pro

### 动态模型选择

有时候需要根据任务复杂度动态选择模型：

```javascript
// 在Code节点中判断
const inputLength = $json.userMessage.length;
const isCodeTask = $json.userMessage.includes('代码') || 
                   $json.userMessage.includes('function');

// 根据条件返回模型
if (inputLength > 5000 || isCodeTask) {
  return { model: 'gpt-4o', temperature: 0.3 };
} else {
  return { model: 'gpt-4o-mini', temperature: 0.7 };
}
```

然后在LLM节点中使用表达式引用：

```yaml
model: "{{ $node['ModelSelector'].json.model }}"
temperature: "{{ $node['ModelSelector'].json.temperature }}"
```

## Prompt节点配置

### 基础模板语法

Prompt节点支持强大的模板语法：

```markdown
## 基础变量引用
{{ $json.field }}                     # 引用JSON字段
{{ $node["NodeName"].json.field }}   # 引用其他节点输出
{{ $vars.secretKey }}                # 引用变量

## 条件逻辑
{{ $json.lang === 'zh' ? '中文' : '英文' }}

## 循环处理
{{ $json.items.map(item => `- ${item}`).join('\n') }}

## 数学运算
{{ Math.round($json.price * 1.1) }}

## 日期时间
{{ new Date().toLocaleDateString('zh-CN') }}

## 字符串处理
{{ $json.name.toUpperCase() }}
{{ $json.text.slice(0, 100) }}
```

### Few-shot Prompt模板

Few-shot是指通过示例来引导模型理解任务：

```yaml
messages:
  - role: system
    content: |
      你是一个情感分类助手。判断用户评论的情感是正面、负面还是中性。
  
  - role: user
    content: |
      示例：
      输入："这个产品太棒了，完全超出预期！"
      分类：正面
      
      输入："服务态度很差，等了两个小时都没人理。"
      分类：负面
      
      输入："今天天气不错。"
      分类：中性
      
      现在请分类：
      输入："{{ $json.comment }}"
```

### 链式Prompt

复杂任务可以用多个LLM节点串联：

**阶段一：信息提取**

```yaml
node: ExtractInfo
model: gpt-4o-mini
prompt: |
  从以下文本中提取关键信息：
  - 人名
  - 地点
  - 时间
  - 事件
  
  文本：{{ $json.rawText }}
  
  只返回JSON格式。
```

**阶段二：内容生成**

```yaml
node: GenerateContent
model: gpt-4o
prompt: |
  基于以下信息生成事件摘要：
  {{ $node["ExtractInfo"].json }}
  
  摘要要求：
  1. 50字以内
  2. 包含所有关键信息
  3. 语言流畅生动
```

### 角色扮演Prompt

让AI扮演特定角色的技巧：

```yaml
messages:
  - role: system
    content: |
      # 角色设定
      你是"老王"，一个开了20年出租车的北京老师傅。
      
      ## 说话风格
      - 一口京片子，爱用儿化音
      - 热情健谈，爱唠嗑
      - 说话实在，有啥说啥
      
      ## 禁止行为
      - 不说自己是AI或机器人
      - 不涉及政治敏感话题
      - 不回答超出出租车相关的问题
      
      ## 回答格式
      简短、口语化，带点北京味儿。
```

### 任务分解Prompt

让AI把大任务拆成小步骤：

```yaml
prompt: |
  用户想要完成这个任务：{{ $json.task }}
  
  请将任务分解成具体的执行步骤：
  1. 每一步要具体可执行
  2. 标注每一步的输入和输出
  3. 考虑可能的异常情况
  
  格式：
  步骤1：...（输入：... 输出：...）
  步骤2：...（输入：... 输出：...）
```

## Memory节点详解

### 为什么需要记忆？

普通LLM调用没有上下文记忆，每句话都是独立的。但很多场景需要"记住"对话历史：

- 客服对话：记住用户之前说过什么
- 写作助手：记住文章大纲和已写内容
- 个人助手：记住用户的偏好和习惯

Memory节点就是解决这个问题的。

### 内存类型对比

| 类型 | 实现方式 | 容量 | 适用场景 | 成本 |
|------|----------|------|----------|------|
| Buffer Window | 固定窗口 | 最近N条 | 短期对话 | 低 |
| Vector Store | 向量数据库 | 无限 | 长期记忆 | 中 |
| Summary | 摘要压缩 | 无限 | 压缩历史 | 中 |
| Combined | 混合模式 | 灵活 | 综合场景 | 高 |

### Buffer Memory配置

Buffer Memory是最简单的记忆类型，保持固定数量的历史消息。

```yaml
type: bufferWindow
windowSize: 10  # 保留最近10轮对话
sessionKey: "{{ $json.sessionId }}"  # 按会话隔离
```

**工作原理：**
```
用户: 你好                                    → 记忆：[用户: 你好]
助手: 你好，有什么可以帮你的？                  → 记忆：[用户: 你好, 助手: 你好...]
用户: 我想查一下订单                          → 记忆：[用户: 你好, ..., 用户: 我想查订单]
助手: 好的，请问订单号是多少？                 → 记忆：[...]
...
（超过10轮后，最早的消息会被移除）
```

### Vector Store Memory配置

向量记忆可以存储无限量的历史，通过语义检索找回相关记忆。

```yaml
type: vectorStore
provider: Pinecone
index: conversations
dimension: 1536
metadata:
  userId: "{{ $json.userId }}"
  sessionId: "{{ $json.sessionId }}"
```

**配合Embedding节点：**

```yaml
# 先Embedding
node: EmbedText
type: OpenAiEmbeddings
text: "{{ $json.message }}"

# 再存储到向量数据库
node: StoreInVectorDB
type: Pinecone
operation: insert
vector: "{{ $node['EmbedText'].json.embedding }}"
metadata:
  role: "{{ $json.role }}"
  content: "{{ $json.message }}"
  timestamp: "{{ $now.toISO() }}"
```

**检索记忆：**

```yaml
# 查询相关记忆
node: RetrieveMemory
type: Pinecone
operation: retrieve
query: "{{ $node['EmbedQuery'].json.embedding }}"
topK: 5
filter:
  userId: "{{ $json.userId }}"
```

### 混合记忆配置

结合Buffer和Vector两种记忆的优点：

```yaml
type: combined
buffers:
  - type: bufferWindow
    windowSize: 5  # 最近5轮完整对话
  
  - type: vectorStore
    topK: 3  # 相关历史
    similarityThreshold: 0.7
    filter:
      userId: "{{ $json.userId }}"
```

### 实战：智能客服记忆

**场景：** 用户咨询产品问题，需要记住用户身份、历史问题、购买记录。

**工作流设计：**

```
Webhook → 提取用户ID → 查询用户信息 → Buffer记忆 → LLM处理 → 更新记忆
```

**详细配置：**

**1. Webhook接收消息**
```yaml
method: POST
path: customer-chat
```

**2. 查询用户历史**
```yaml
type: Pinecone
operation: retrieve
query: "{{ $node['EmbedQuery'].json.embedding }}"
topK: 10
filter:
  userId: "{{ $json.userId }}"
```

**3. Buffer记忆配置**
```yaml
type: bufferWindow
windowSize: 20  # 保留最近20轮对话
sessionKey: "{{ $json.userId }}"
```

**4. LLM处理**
```yaml
model: gpt-4o
prompt: |
  用户ID：{{ $json.userId }}
  
  用户历史信息：
  {{ $node["RetrieveMemory"].json.matches.map(m => m.metadata.content).join('\n') }}
  
  当前对话：
  {{ $node["BufferMemory"].json.messages.join('\n') }}
  
  用户问题：{{ $json.message }}
  
  请结合历史信息和当前问题，回答用户。
```

**5. 更新记忆**
```yaml
type: Pinecone
operation: insert
vector: "{{ $node["NewEmbedding"].json.embedding }}"
metadata:
  role: "user"
  content: "{{ $json.message }}"
  userId: "{{ $json.userId }}"
  timestamp: "{{ $now.toISO() }}"
```

## Tool节点配置

### 工具调用原理

Tool（工具）让LLM能调用外部服务，实现"思考+行动"的闭环：

```
用户: 帮我查一下北京的天气
    ↓
LLM理解意图 → 需要查天气 → 调用天气API
    ↓
API返回数据 → LLM整合 → 返回回答
```

### 工具定义格式

n8n支持OpenAI格式的工具定义：

```yaml
tools:
  - name: search_knowledge_base
    description: 在知识库中搜索相关内容
    parameters:
      type: object
      properties:
        query:
          type: string
          description: 搜索关键词
        topK:
          type: integer
          description: 返回结果数量
          default: 5
      required:
        - query
  
  - name: send_email
    description: 发送电子邮件
    parameters:
      type: object
      properties:
        to:
          type: string
          description: 收件人邮箱
        subject:
          type: string
          description: 邮件主题
        body:
          type: string
          description: 邮件正文
      required:
        - to
        - subject
        - body
  
  - name: calculate
    description: 执行数学计算
    parameters:
      type: object
      properties:
        expression:
          type: string
          description: 数学表达式，如 2+3*4
      required:
        - expression
```

### HTTP Request作为Tool

通过HTTP Request节点实现工具：

**天气查询工具：**

```yaml
# 定义工具
name: get_weather
description: 查询指定城市的天气

# HTTP Request节点配置
type: HTTP Request
method: GET
url: "https://api.weather.com/v3/wx/conditions"
qs:
  city: "{{ $json.city }}"
  units: "{{ $json.unit || 'celsius' }}"
authentication: inherit
```

**数据库查询工具：**

```yaml
name: query_database
description: 查询数据库

type: PostgreSQL
operation: executeQuery
query: |
  SELECT * FROM products 
  WHERE name LIKE '%{{ $json.keyword }}%'
  LIMIT {{ $json.limit || 10 }}
```

### 工具调用流程

```
LLM决定调用工具 → Tool节点执行 → 返回结果 → LLM继续处理
```

**示例工作流：**

```
Webhook(用户消息)
    ↓
LLM(分析意图，可能返回工具调用)
    ↓
Tool节点(执行工具)
    ↓
LLM(基于工具结果生成回答)
    ↓
返回用户
```

### 工具设计最佳实践

**1. 描述要清晰**
```yaml
# 好的描述
name: get_order_status
description: |
  查询订单物流状态
  
  输入：订单号（必填）
  输出：物流状态、当前地点、预计到达时间
  
  适用：用户问"我的订单到哪了"、"查一下订单"
  
  不适用：取消订单、退款等操作（请用cancel_order）
```

**2. 参数要精简**
```yaml
# 好的参数设计
properties:
  city:
    type: string
    description: 城市名称，支持中英文

# 不好的参数设计
properties:
  data:
    type: object
    description: 数据对象
```

**3. 返回要结构化**
```json
// 好的返回格式
{
  "success": true,
  "data": {
    "order_id": "ORD123456",
    "status": "shipped",
    "tracking_number": "SF123456789",
    "current_location": "北京分拣中心",
    "estimated_delivery": "2024-01-20"
  }
}
```

## Agent节点配置

### Agent vs 普通LLM

| 维度 | 普通LLM调用 | Agent |
|------|-------------|-------|
| 交互方式 | 问答 | 任务执行 |
| 工具使用 | 无 | 可调用多个工具 |
| 执行模式 | 单轮响应 | 多轮推理循环 |
| 适用场景 | 简单问答 | 复杂任务 |

### Agent类型

| 类型 | 特点 | 适用场景 |
|------|------|----------|
| ReAct Agent | 推理+行动 | 复杂多步骤任务 |
| OpenAI Function Agent | 函数调用 | 工具丰富的场景 |
| Plan and Execute | 计划后执行 | 需要规划的任务 |

### ReAct Agent配置

ReAct = Reasoning + Acting，核心是循环：思考→行动→观察→重复

```yaml
type: ReAct
llm: gpt-4o

tools:
  - search_knowledge_base
  - calculator
  - web_search

maxIterations: 10

earlyStopping: true
stopSequence: "最终回答"

systemPrompt: |
  你是一个智能助手，可以调用各种工具完成任务。
  
  工作方式：
  1. 理解用户任务
  2. 判断是否需要工具
  3. 选择合适的工具
  4. 根据结果决定下一步
  5. 直到完成任务
```

### OpenAI Function Agent

适合有丰富工具定义的场景，模型直接决定调用哪个函数：

```yaml
type: openAiFunction
llm: gpt-4o

tools:  # 直接使用OpenAI格式的工具定义
  - name: get_weather
    description: 获取城市天气
    parameters:
      type: object
      properties:
        city:
          type: string
      required: [city]
```

### Agent执行流程

```
用户任务 → Agent分析
                ↓
          需要工具?
            ↓是      ↓否
      选择工具      生成回答
            ↓
      执行工具
            ↓
      获取结果
            ↓
      继续分析? → 是 → 循环
            ↓否
      生成最终回答
```

### Agent实战案例

**案例：研究助手**

```yaml
# Agent配置
type: ReAct
llm: gpt-4o

tools:
  - web_search
  - knowledge_retrieval
  - calculator
  - save_to_notion

systemPrompt: |
  你是一个研究助手，帮助用户收集信息、分析数据、生成报告。
  
  能力：
  - 搜索网络获取最新信息
  - 检索知识库获取背景知识
  - 执行计算进行数据分析
  - 保存结果到Notion笔记
  
  工作流程：
  1. 理解研究主题
  2. 分解研究任务
  3. 并行搜索信息
  4. 分析整理信息
  5. 生成研究报告
  6. 保存到Notion
```

## 完整实战案例

### 案例一：AI研究助手

**需求：** 接收研究主题，自动完成信息收集、分析、报告生成。

**工作流设计：**

```
Webhook → 任务规划 → 并行搜索 → 信息整合 → 报告生成 → 保存Notion
```

**详细配置：**

**1. Webhook触发**
```yaml
endpoint: /research-assistant
method: POST
```

**2. 任务规划（LLM）**
```yaml
model: gpt-4o
prompt: |
  研究主题：{{ $json.topic }}
  
  请将研究任务分解成具体的子任务：
  
  1. 搜索方向（3-5个关键词）
  2. 分析维度
  3. 预期结论
  4. 报告结构建议
  
  返回JSON格式。
```

**3. 并行搜索（Fan-out）**
```yaml
# 使用Split In Batches实现并行
node: SplitKeywords
type: SplitInBatches
values:
  - "{{ $node['Plan'].json.searchKeywords }}"
batchSize: 3

# 每个关键词搜索
node: SearchWeb
type: HTTP Request
method: GET
url: "https://api.search.com?q={{ $json.currentKeyword }}"
```

**4. 信息整合（LLM）**
```yaml
model: gpt-4o
prompt: |
  整合以下搜索结果，提取关键信息：
  
  搜索结果：
  {{ $node["SearchResults"].json.allResults.map(r => r.content).join('\n---\n') }}
  
  研究主题：{{ $json.topic }}
  
  提取：
  1. 主要发现（5条）
  2. 数据支撑
  3. 不同观点
  4. 知识空白
```

**5. 报告生成（LLM）**
```yaml
model: gpt-4o
prompt: |
  基于以下研究资料，生成完整研究报告：
  
  研究主题：{{ $json.topic }}
  
  研究资料：
  {{ $node["Integrate"].json.summary }}
  
  报告要求：
  1. 结构清晰，包含摘要、正文、结论
  2. 数据准确，引用来源
  3. 观点平衡，列出不同立场
  4. 实用性强，有可操作的建议
  
  格式：Markdown
```

**6. 保存Notion**
```yaml
operation: createPage
databaseId: 研究报告库
properties:
  title:
    title:
      - text:
          content: "研究报告：{{ $json.topic }}"
  tags:
    multi_select:
      - "{{ $json.category }}"
author: 研究助手
content: |
  {{ $node["Report"].json.content }}
```

### 案例二：智能邮件助手

**需求：** 自动处理收件箱，分类、总结、自动回复。

**工作流设计：**

```
Gmail新邮件 → 分类判断 → 优先级排序 → 总结生成 → 自动回复 → 归档
```

**详细配置：**

**1. Gmail触发**
```yaml
type: Gmail Trigger
pollInterval: 5minutes
filter: is:unread
```

**2. 邮件分类（LLM）**
```yaml
model: gpt-4o-mini
prompt: |
  分析以下邮件，判断类型和优先级：
  
  邮件：
  发件人：{{ $json.from }}
  主题：{{ $json.subject }}
  内容：{{ $json.body }}
  
  返回JSON：
  {
    "type": "urgent/important/routine/spam",
    "category": "客户咨询/技术支持/商务合作/内部通知/其他",
    "priority": 1-5,
    "action": "reply_now/reply_later/forward/ignore"
  }
```

**3. 分类处理**

**紧急邮件 → 直接回复（LLM生成）**
```yaml
model: gpt-4o
prompt: |
  邮件主题：{{ $json.subject }}
  邮件内容：{{ $json.body }}
  
  生成一封专业、简洁的回复邮件。
  语气要亲切但不卑微。
  控制在100字以内。
```

**重要邮件 → 总结 + 标记**
```yaml
# 总结
model: gpt-4o-mini
prompt: |
  总结以下邮件的核心内容：
  
  {{ $json.body }}
  
  要求：
  - 50字以内
  - 包含关键信息和行动项
```

**常规邮件 → 稍后处理**
```yaml
# 添加标签，稍后统一处理
type: Gmail
operation: addLabel
label: "待处理/{{ $node["Classify"].json.category }}"
```

### 案例三：多模态内容处理

**需求：** 接收图片，识别内容，生成描述，提取信息。

**工作流设计：**

```
图片输入 → GPT-4V分析 → 信息提取 → 结构化输出
```

**详细配置：**

**1. 图片输入**
```yaml
# 支持多种来源
type: HTTP Request
url: "{{ $json.imageUrl }}"
response: binary
```

**2. 视觉分析（GPT-4V）**
```yaml
model: gpt-4o
responseFormat: json

messages:
  - role: user
    content:
      - type: image_url
        image_url: "{{ $node["ImageInput"].binary.data }}"
      - type: text
        text: |
          分析这张图片：
          1. 图片内容描述
          2. 关键元素识别
          3. 文字识别（如果有）
          4. 质量评估
          
          返回JSON格式。
```

**3. 信息提取（LLM）**
```yaml
model: gpt-4o
prompt: |
  基于以下图片分析结果，提取结构化信息：
  
  图片分析：{{ $node["VisionAnalysis"].json }}
  
  提取字段：
  - 主标题
  - 副标题
  - 内容标签（最多5个）
  - 情感倾向（正面/中性/负面）
  - 行动建议
  
  返回JSON格式。
```

## 高级技巧

### Token控制

**计算预估Token：**
```yaml
# 表达式计算
tokenEstimate: |
  {{ Math.ceil(($json.prompt.length + $json.context.length) / 4) }}
```

**动态截断：**
```yaml
# 截断过长的上下文
truncatedContext: |
  {{ $json.fullContext.slice(0, 6000) }}
```

### 批量处理

**使用Split In Batches并行：**
```yaml
node: SplitInBatches
batchSize: 5
options:
  reset: false

# 后续节点处理每个批次
node: LLMCall
type: LLM Chain
# 每个批次处理5个请求
```

### 成本优化

**策略一：模型分级**
```javascript
// Code节点判断
const complexity = $json.query.length + $json.context.length;

if (complexity < 1000) {
  return { model: 'gpt-4o-mini', cost: 'low' };
} else if (complexity < 5000) {
  return { model: 'gpt-4o', cost: 'medium' };
} else {
  return { model: 'claude-3-5-sonnet', cost: 'high' };
}
```

**策略二：缓存结果**
```yaml
# 先检查缓存
node: CheckCache
type: Redis
operation: get
key: "cache:{{ $hash($json.query) }}"

# 缓存命中则跳过LLM
node: Conditional
conditions:
  - if: "{{ $node["CheckCache"].json.exists }}"
    then: UseCache
  - else: CallLLM
```

**策略三：摘要压缩**
```yaml
# 长对话定期摘要
node: SummarizeMemory
type: LLM Chain
model: gpt-4o-mini
prompt: |
  将以下对话历史压缩成摘要：
  
  {{ $json.history }}
  
  要求：
  - 保留关键信息和结论
  - 压缩到200字以内
  - 返回摘要文本
```

### 错误重试

**配置重试机制：**
```yaml
onError:
  continueOnFail: true
  retryNode: LLMCall

# 在Code节点中实现重试逻辑
node: RetryLogic
type: Code
code: |
  const attempts = $node["Attempt"].json.attempts || 0;
  if (attempts < 3) {
    return {
      json: { shouldRetry: true, attempts: attempts + 1 }
    };
  }
  return { json: { shouldRetry: false, error: "Max retries exceeded" } };
```

## 相关资源

- [[n8n平台深度指南]] - n8n完整教程
- [[工作流设计模式]] - 工作流设计原则
- [[Function Calling与工具调用]] - 函数调用规范详解
- [[AI对话记忆系统]] - 记忆系统设计
- [[多Agent系统设计]] - 多智能体协作

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
