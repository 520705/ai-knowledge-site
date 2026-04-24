---
title: n8n平台深度指南：工作流自动化与AI集成
date: 2026-04-24
tags:
  - n8n
  - 工作流自动化
  - AI集成
  - 低代码平台
  - 流程自动化
categories:
  - 智能体搭建
  - n8n平台
description: 全面介绍n8n平台的功能特性、工作流编辑器、AI节点配置、自定义节点开发以及实战案例，帮助读者掌握这一强大的工作流自动化工具。
---

# n8n平台深度指南：工作流自动化与AI集成

> [!NOTE] 这篇指南讲什么
> n8n是一款强大的开源工作流自动化工具，特别擅长连接各种服务和数据处理。这篇指南从基础入门讲到进阶实战，让你从零开始掌握n8n。

## n8n是什么？

n8n（发音"n-eight-n"）是一款开源的工作流自动化工具。它可以帮你把各种服务连接起来，自动执行各种任务。

用大白话讲：**n8n就是自动化领域的"瑞士军刀"**，它能把Gmail、Slack、Notion、数据库、各种API...全都连起来，让它们自动配合工作。

打个比方：

**不用n8n** → 每天手动从A系统导数据，整理格式，再导入B系统，累死
**用n8n** → 搭一个自动化流程，每天下午6点自动完成以上操作，爽歪歪

### n8n vs 其他自动化工具

| 工具 | 定位 | AI能力 | 成本 | 适合人群 |
|------|------|--------|------|----------|
| **n8n** | 通用自动化+AI | 强 | 开源免费 | 技术团队、企业 |
| Zapier | SaaS自动化 | 弱 | 按执行次数收费 | 非技术用户 |
| Make | 可视化自动化 | 中 | 按执行次数收费 | 中级用户 |
| Airflow | 数据管道 | 弱 | 开源免费 | 数据工程师 |

**n8n的优势：**
- 完全开源，可以私有化部署
- 支持自定义代码（JavaScript/Python）
- AI能力强大（LLM、Embedding、Agent）
- 800+集成，覆盖主流服务
- 社区活跃，模板丰富

## 核心概念

### 工作流（Workflow）

工作流是n8n的核心概念。你可以把它理解为**一系列自动执行的步骤**。

```
触发器 → 处理节点1 → 处理节点2 → ... → 输出
```

**组成要素：**

- **触发器（Trigger）**：工作流的起点，定义"什么时候开始执行"
- **节点（Node）**：每个具体的操作步骤
- **连接线（Connection）**：节点之间的数据流向
- **数据（Data）**：节点之间传递的信息

### 节点类型

n8n有数百种节点，分为几大类：

| 类型 | 例子 | 说明 |
|------|------|------|
| 触发器 | Webhook、定时器、事件 | 启动工作流 |
| 通信 | Gmail、Slack、Email、Telegram | 消息发送接收 |
| 数据 | HTTP请求、代码、转换 | 数据处理 |
| AI | LLM、Prompt、Memory、Embedding | AI能力 |
| 数据库 | MySQL、MongoDB、PostgreSQL | 数据读写 |
| 工具 | OAuth2、凭证管理 | 认证和安全 |

### 数据流

n8n采用节点连接的方式传递数据。每个节点的输出成为下一个节点的输入。

**数据格式：**

```javascript
// 节点输出通常是JSON格式
{
  "json": {
    "id": 1,
    "name": "张三",
    "email": "zhangsan@example.com"
  },
  "binary": {}  // 二进制数据（如文件）
}
```

**引用其他节点的数据：**

```javascript
// 引用上一个节点的输出
{{ $json.fieldName }}

// 引用指定节点的输出
{{ $node["NodeName"].json["fieldName"] }}

// 引用输入数据
{{ $input.first().json["fieldName"] }}
```

## 安装部署

### Docker快速部署（推荐）

```bash
# 单容器快速部署
docker run -d --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e GENERIC_TIMEZONE="Asia/Shanghai" \
  docker.n8n.io/n8nio/n8n:latest
```

然后打开 http://localhost:5678 就能用了。

### Docker Compose生产部署

创建 `docker-compose.yml`：

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=你的密码
      - N8N_HOST=0.0.0.0
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
      - NODE_ENV=production
      - GENERIC_TIMEZONE=Asia/Shanghai
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=n8n-db
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=n8n
      - DB_POSTGRESDB_PASSWORD=n8n密码
      - EXECUTIONS_MODE=queue
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - n8n-db
  
  n8n-db:
    image: postgres:15
    environment:
      - POSTGRES_DB=n8n
      - POSTGRES_USER=n8n
      - POSTGRES_PASSWORD=n8n密码
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  n8n_data:
  postgres_data:
```

启动：

```bash
docker compose up -d
```

### npm本地安装

```bash
# 全局安装
npm install n8n -g

# 启动
n8n
# 或指定端口
n8n start --port 5679
```

### 环境变量配置

| 变量名 | 说明 | 示例 |
|--------|------|------|
| N8N_BASIC_AUTH_ACTIVE | 启用基础认证 | true |
| N8N_BASIC_AUTH_USER | 用户名 | admin |
| N8N_BASIC_AUTH_PASSWORD | 密码 | your_password |
| N8N_HOST | 访问地址 | 0.0.0.0 |
| N8N_PORT | 端口 | 5678 |
| N8N_PROTOCOL | 协议 | https |
| WEBHOOK_URL | Webhook基础URL | https://example.com |
| GENERIC_TIMEZONE | 时区 | Asia/Shanghai |

## 工作流编辑器

### 界面布局

```
┌─────────────────────────────────────────────────────────────┐
│  [执行历史] [模板] [变量] [凭证]          [保存] [执行]    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐             │
│  │  触发器   │───▶│  处理1   │───▶│  输出节点 │             │
│  └──────────┘    └──────────┘    └──────────┘             │
│                         │                                    │
│                         ▼                                    │
│                  ┌──────────┐                               │
│                  │  处理2   │                               │
│                  └──────────┘                               │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│  [节点面板]                                                  │
│  ▼ 触发器                                                   │
│    Webhook / 定时 / 事件                                    │
│  ▼ 应用节点                                                 │
│    HTTP请求 / 代码 / AI / 数据库                            │
└─────────────────────────────────────────────────────────────┘
```

### 节点操作

**添加节点：**
1. 从左侧节点面板拖拽到画布
2. 或者点击"+"按钮，搜索节点名称

**连接节点：**
1. 点击节点的输出端口（右侧圆点）
2. 拖动到目标节点的输入端口（左侧圆点）
3. 松开鼠标完成连接

**配置节点：**
1. 点击节点打开配置面板
2. 填写必要的配置参数
3. 节点变成蓝色表示已配置完成

### 表达式语法

n8n使用表达式来处理动态数据。

**基础用法：**

```javascript
// 引用JSON字段
{{ $json.message }}

// 引用节点数据
{{ $node["Webhook"].json["body"]["text"] }}

// 引用输入数据
{{ $input.first().json["field"] }}

// 数学运算
{{ $json.price * 1.1 }}

// 条件判断
{{ $json.status === "active" ? "启用" : "禁用" }}

// 日期时间
{{ new Date().toISOString() }}

// 字符串处理
{{ $json.name.toUpperCase() }}

// 数组操作
{{ $json.items.map(i => i.value).join(", ") }}
```

## AI节点详解

### AI节点体系

n8n提供了完整的AI能力组件：

```
用户输入 → LLM Chain → Agent
                    ↓
              ┌─────┴─────┐
              ↓           ↓
           Tools      Memory
              ↓           ↓
         外部API      向量存储
```

### LLM Chain节点

LLM Chain是调用大语言模型的核心节点。

**配置示例：**

```yaml
节点配置:
  # 模型选择
  provider: OpenAI
  model: gpt-4o
  
  # 认证
  credentials: OpenAI API (你的API Key)
  
  # 消息配置
  messages:
    - role: system
      content: |
        你是一个专业的技术文档助手。
        擅长用简洁清晰的语言解释复杂概念。
    
    - role: user
      content: "{{ $json.userMessage }}"
  
  # 生成参数
  options:
    temperature: 0.7      # 控制创造性，0-2，越高越有创意
    maxTokens: 1000       # 最大生成长度
    topP: 1.0            # 采样策略
```

**支持的模型：**

| 模型 | 提供商 | 特点 |
|------|--------|------|
| GPT-4o | OpenAI | 最强综合能力 |
| GPT-4o-mini | OpenAI | 性价比高 |
| Claude 3.5 | Anthropic | 长文本处理强 |
| Gemini 1.5 | Google | 多模态 |
| DeepSeek | DeepSeek | 国产便宜 |
| 本地Ollama | 本地 | 完全免费 |

### Prompt节点

Prompt节点用于构建结构化提示词。

**模板语法：**

```markdown
## 基础变量
{{ $json.field }}                     # JSON字段
{{ $node["NodeName"].json.field }}    # 指定节点字段
{{ $vars.secretKey }}                 # 变量

## 条件逻辑
{{ $json.lang === 'zh' ? '中文' : '英文' }}

## 循环处理
{{ $json.items.map(item => `- ${item.name}`).join('\n') }}

## 数学运算
{{ Math.round($json.price * 1.1) }}
```

**Few-shot Prompt示例：**

```yaml
messages:
  - role: system
    content: |
      你是一个情感分类助手。
  
  - role: user
    content: |
      示例：
      输入："这个产品太棒了！"
      分类：正面
      
      输入："服务态度太差了"
      分类：负面
      
      请分类：
      输入："{{ $json.comment }}"
```

### Memory节点

Memory节点管理对话上下文。

**内存类型对比：**

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| Buffer Window | 固定窗口 | 短期对话 |
| Vector Store | 向量数据库 | 长期记忆 |
| Summary | 摘要压缩 | 压缩历史 |
| Combined | 混合模式 | 综合场景 |

**Buffer Memory配置：**

```yaml
type: bufferWindow
windowSize: 10  # 保留最近10轮对话
sessionKey: "{{ $json.sessionId }}"  # 按会话隔离
```

**向量记忆配置：**

```yaml
type: vectorStore
provider: Pinecone
index: conversations
metadata:
  userId: "{{ $json.userId }}"
```

### Embedding节点

Embedding节点将文本转换为向量。

```yaml
配置:
  provider: OpenAI
  model: text-embedding-3-small
  input: "{{ $json.text }}"
```

### AI Agent节点

AI Agent能自主决策和执行任务。

**ReAct Agent配置：**

```yaml
type: ReAct
llm: gpt-4o
tools:
  - search_knowledge_base
  - calculator
  - web_search
maxIterations: 10
earlyStopping: true
```

**Agent执行流程：**

```
用户请求 → Agent分析
              ↓
        需要工具?
          ↓是     ↓否
      选择工具  生成回答
          ↓
      执行工具
          ↓
      获取结果 → 返回用户
          ↓
      继续分析（循环直到完成）
```

## 常用节点配置

### Webhook触发器

Webhook允许外部系统触发工作流。

**创建Webhook：**

1. 添加Webhook节点
2. 选择HTTP方法（GET/POST）
3. 保存节点，复制生成的URL
4. 外部系统调用该URL即可触发

**调用示例：**

```bash
# GET请求
curl "https://your-n8n.com/webhook/your-path"

# POST请求
curl -X POST "https://your-n8n.com/webhook/your-path" \
  -H "Content-Type: application/json" \
  -d '{"message": "hello"}'
```

### HTTP Request节点

HTTP Request调用外部API。

**GET请求：**

```yaml
method: GET
url: "https://api.example.com/users"
qs: 
  page: "{{ $json.page }}"
  limit: 10
headers:
  Authorization: "Bearer {{ $credentials.apiKey }}"
```

**POST请求：**

```yaml
method: POST
url: "https://api.example.com/users"
body: json
bodyParameters:
  - name: name
    value: "{{ $json.name }}"
  - name: email
    value: "{{ $json.email }}"
```

### Code节点

Code节点执行JavaScript/Python代码。

**JavaScript示例：**

```javascript
// 获取输入数据
const data = $input.first().json;

// 数据处理
const processed = {
  id: data.id,
  name: data.name.toUpperCase(),
  score: data.score * 1.1,
  createdAt: new Date().toISOString()
};

// 返回结果
return [{ json: processed }];
```

**多输入处理：**

```javascript
// 处理多个输入项
const items = $input.all();

const results = items.map(item => ({
  json: {
    original: item.json,
    normalized: {
      id: item.json.id,
      name: item.json.name.trim(),
      value: Number(item.json.value)
    }
  }
}));

return results;
```

### 数据库节点

**MySQL示例：**

```yaml
operation: executeQuery
query: "SELECT * FROM users WHERE status = ?"
values:
  - "{{ $json.status }}"
```

**MongoDB示例：**

```yaml
operation: find
collection: users
query:
  status: "{{ $json.status }}"
  limit: 10
```

## 实战案例

### 案例一：AI客服工作流

**需求：** 接收用户消息，判断意图，查询知识库，返回回答。

```
Webhook → 意图识别LLM → 知识库检索 → 生成回答 → 回复用户
```

**详细配置：**

**1. Webhook触发器**
```yaml
method: POST
path: customer-service
authentication: none
```

**2. 意图识别LLM**
```yaml
model: gpt-4o-mini
prompt: |
  分析用户消息的意图：
  - pre_sales: 售前咨询
  - after_sales: 售后问题
  - complaint: 投诉建议
  
  用户消息：{{ $json.message }}
  
  只返回意图类别名称。
```

**3. 知识库检索（Vector Store）**
```yaml
operation: retrieve
vectorStore: Pinecone
index: knowledge_base
query: "{{ $json.message }}"
topK: 3
```

**4. 生成回答LLM**
```yaml
model: gpt-4o
prompt: |
  基于以下知识回答用户问题：
  
  知识内容：
  {{ $node["Pinecone"].json.results.map(r => r.text).join('\n\n') }}
  
  用户问题：{{ $json.message }}
  
  要求：
  1. 回答简洁专业
  2. 如知识库无相关内容，诚实说不知道
  3. 引导转人工处理复杂问题
```

### 案例二：社交媒体自动发布

**需求：** 每周一自动从Notion读取内容，生成社交媒体帖子，发布到Twitter和LinkedIn。

```
定时器 → Notion读取 → AI改写 → 条件分流 → Twitter发布
                                                  → LinkedIn发布
```

**详细配置：**

**1. 定时触发器**
```yaml
type: Schedule
rule:
  interval: [1, "week"]
cron: "0 9 * * 1"  # 每周一早上9点
```

**2. Notion读取**
```yaml
operation: search
databaseId: 你的数据库ID
filter:
  property: Status
  select:
    equals: Ready
```

**3. AI改写（多个输出）**

```yaml
# Twitter版本
model: gpt-4o-mini
prompt: |
  将以下内容改写成Twitter帖子（280字以内）：
  
  内容：{{ $json.content }}
  
  要求：
  1. 简洁有吸引力
  2. 适当使用emoji
  3. 添加相关话题标签
```

```yaml
# LinkedIn版本
model: gpt-4o-mini
prompt: |
  将以下内容改写成LinkedIn帖子：
  
  内容：{{ $json.content }}
  
  要求：
  1. 专业但有温度
  2. 包含简短的个人见解
  3. 鼓励互动（提问或讨论）
```

**4. Twitter发布**
```yaml
operation: createTweet
text: "{{ $node["TwitterRewrite"].json.text }}"
```

**5. LinkedIn发布**
```yaml
operation: createPost
content: "{{ $node["LinkedInRewrite"].json.text }}"
```

### 案例三：数据同步与处理

**需求：** 每天凌晨同步CRM数据到数据仓库，清洗处理后生成报表。

```
定时器 → CRM读取 → 数据清洗 → 数据仓库写入 → 发送报表
```

**详细配置：**

**1. 定时触发器**
```yaml
type: Schedule
rule:
  interval: [1, "day"]
cron: "0 2 * * *"  # 每天凌晨2点
```

**2. CRM读取**
```yaml
# Salesforce示例
operation: executeQuery
query: |
  SELECT Id, Name, Email, Amount, CreatedDate 
  FROM Opportunity 
  WHERE CreatedDate = LAST_N_DAYS:1
```

**3. 数据清洗（Code节点）**
```javascript
const records = $input.all();

const cleaned = records.map(record => {
  const data = record.json;
  
  return {
    json: {
      id: data.Id,
      name: data.Name?.trim(),
      email: data.Email?.toLowerCase().trim(),
      amount: Number(data.Amount) || 0,
      created_date: new Date(data.CreatedDate).toISOString(),
      created_year: new Date(data.CreatedDate).getFullYear(),
      created_month: new Date(data.CreatedDate).getMonth() + 1
    }
  };
});

// 返回清洗后的数据
return cleaned;
```

**4. 数据仓库写入**
```yaml
# PostgreSQL示例
operation: executeQuery
query: |
  INSERT INTO crm_opportunities 
  (id, name, email, amount, created_date, created_year, created_month)
  VALUES 
  {{ $json.map(r => `('${r.id}', '${r.name}', '${r.email}', ${r.amount}, '${r.created_date}', ${r.created_year}, ${r.created_month})`).join(', ') }}
  ON CONFLICT (id) DO UPDATE SET
    name = EXCLUDED.name,
    amount = EXCLUDED.amount
```

**5. 生成并发送报表**
```yaml
# 计算统计
model: gpt-4o-mini
prompt: |
  根据以下数据生成日报摘要：
  
  数据：
  {{ JSON.stringify($input.all().map(i => i.json)) }}
  
  生成：
  1. 今日新增商机数量
  2. 总金额
  3. 平均金额
  4. 简短分析
```

```yaml
# 发送邮件
operation: send
to: "manager@company.com"
subject: "CRM日报 - {{ $now.format('YYYY-MM-DD') }}"
html: |
  <h2>CRM数据同步日报</h2>
  <p>{{ $node["ReportLLM"].json.summary }}</p>
```

## 高级技巧

### 错误处理

**节点级错误处理：**

```yaml
errorOutput: true  # 启用错误输出
continueOnFail: true  # 失败后继续执行
```

**工作流级错误处理：**

```yaml
onError:
  - name: Error Workflow
    trigger: error
    workflow: "error-handler-workflow-id"
```

### 并行执行

使用Split In Batches节点实现并行处理：

```yaml
# 先分割数据
node: Split In Batches
batchSize: 5
options:
  reset: false

# 并行处理每个批次
# 在后续节点中处理
```

### 变量管理

**设置变量：**
```yaml
node: Set
variables:
  - name: totalCount
    value: "{{ $json.count }}"
  - name: processedAt
    value: "{{ $now.toISO() }}"
```

**使用变量：**
```javascript
{{ $vars.totalCount }}
{{ $vars.processedAt }}
```

### 子工作流

把常用逻辑封装成子工作流复用：

```yaml
# 主工作流调用子工作流
node: Workflow Trigger (sub-workflow)
workflowId: "sub-workflow-id"
waitForCallback: true
passThroughData: true
data: "{{ $json }}"
```

## 自定义节点开发

### 节点项目结构

```
n8n-nodes-myplugin/
├── package.json
├── src/
│   ├── nodes/
│   │   └── MyNode/
│   │       ├── MyNode.ts
│   │       └── MyNodeDescription.ts
│   └── credentials/
│       └── MyCredentialsApi.ts
├── README.md
└── tsconfig.json
```

### package.json配置

```json
{
  "name": "n8n-nodes-myplugin",
  "version": "1.0.0",
  "description": "我的自定义n8n节点",
  "keywords": ["n8n", "nodes", "myplugin"],
  "license": "MIT",
  "n8n": {
    "nodes": ["dist/MyNode.node.js"]
  },
  "dependencies": {
    "n8n-core": "*"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/node": "^18.0.0"
  }
}
```

### 节点描述定义

```typescript
import type { INodeTypeDescription } from 'n8n-workflow';

export const myNodeDescription: INodeTypeDescription = {
  displayName: '我的自定义节点',
  name: 'myNode',
  group: ['transform'],
  version: 1,
  description: '执行自定义逻辑的n8n节点',
  defaults: {
    name: '我的节点',
  },
  inputs: ['main'],
  outputs: ['main'],
  credentials: [
    {
      name: 'myCredentialsApi',
      required: true,
    },
  ],
  properties: [
    {
      displayName: '操作模式',
      name: 'operation',
      type: 'options',
      options: [
        {
          name: '处理数据',
          value: 'process',
        },
        {
          name: '转换格式',
          value: 'transform',
        },
      ],
      default: 'process',
      required: true,
    },
  ],
};
```

### 节点实现

```typescript
import {
  IExecuteFunctions,
  INodeType,
  INodeTypeDescription,
} from 'n8n-workflow';
import { myNodeDescription } from './MyNodeDescription';

export class MyNode implements INodeType {
  description: INodeTypeDescription = myNodeDescription;

  async execute(this: IExecuteFunctions) {
    const items = this.getInputData();
    const returnData: any[] = [];
    
    const operation = this.getNodeParameter('operation', 0) as string;
    
    for (let i = 0; i < items.length; i++) {
      const item = items[i].json;
      
      if (operation === 'process') {
        const result = {
          ...item,
          processed: true,
          timestamp: new Date().toISOString(),
        };
        returnData.push({ json: result });
      }
    }
    
    return [returnData];
  }
}
```

## 生产环境优化

### 性能优化

**1. 启用执行超时**
```yaml
executionTimeout: 300  # 5分钟超时
```

**2. 减少不必要的数据传递**
```javascript
// 只传递需要的字段
return [{ json: { id: item.json.id, name: item.json.name } }];
```

**3. 使用批量操作**
```yaml
# 数据库批量写入
operation: batchUpdate
batchSize: 100
```

### 高可用配置

**多实例部署：**

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n:latest
    deploy:
      replicas: 3
    environment:
      - N8N_METRICS=true
      - EXECUTIONS_MODE=queue
    depends_on:
      - redis
  
  worker:
    image: n8nio/n8n:latest
    deploy:
      replicas: 2
    command: n8n worker
    environment:
      - EXECUTIONS_MODE=queue
    depends_on:
      - redis
  
  redis:
    image: redis:7-alpine
```

### 监控配置

**启用Metrics：**
```yaml
N8N_METRICS=true
N8N_METRICS_PORT=5678
```

**Prometheus抓取配置：**
```yaml
scrape_configs:
  - job_name: 'n8n'
    static_configs:
      - targets: ['n8n:5678']
```

## 相关资源

- [[n8n与LLM集成]] - n8n AI能力详解
- [[工作流设计模式]] - 通用工作流设计原则
- [[Function Calling与工具调用]] - 工具调用规范
- [[AI对话记忆系统]] - 记忆系统设计
- [[AI应用API化部署]] - API部署方案

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
