# API 文档工具深度指南

> API 文档是开发者体验的关键组成部分。本文深入解析 OpenAPI/Swagger 规范，对比 Scalar、Mintlify、Redocly 等主流 API 文档工具，并提供文档即代码工作流的完整实践指南。

## 技术概述与定位

### API 文档的核心价值

在现代软件开发中，API 文档的重要性再怎么强调都不为过。它是前端与后端开发者之间的桥梁，是第三方集成方了解系统的窗口，更是维护和迭代产品的基础。一个优秀的 API 文档能够显著降低沟通成本，减少错误使用，加速开发进程。反之，缺失或混乱的 API 文档则会带来无尽的调试时间和技术债务。

API 文档的核心价值体现在多个维度。首先是降低学习曲线，让新的开发者能够快速了解系统的能力和限制。其次是减少沟通成本，当文档足够清晰时，开发者无需频繁提问就能找到答案。第三是促进自动化，良好的 API 规范可以驱动 SDK 生成、测试自动化和 CI/CD 流程的改进。第四是提升开发者体验，当开发者能够轻松找到所需信息并顺利完成任务时，他们对产品的满意度会大幅提升。

从技术演进的视角看，API 文档经历了几个重要阶段。早期的 API 文档主要是静态的 HTML 页面或 PDF 文件，需要人工维护且难以保持同步。后来出现了 Swagger（现在的 OpenAPI 规范）等机器可读的格式，使得文档可以自动生成和验证。近年来，交互式文档平台（如 Swagger UI、Postman）成为了主流，允许开发者直接在文档中测试 API 调用。当前，我们正见证着 AI 辅助文档生成和智能搜索等新技术的兴起。

### OpenAPI 规范概述

OpenAPI 规范（OAS）是一种用于描述、生成、消费和可视化 RESTful Web 服务的机器可读接口文件格式。当前最新版本为 3.1.0，它基于 JSON Schema 定义了 API 的完整结构。

```yaml
# openapi.yaml - OpenAPI 3.1 完整示例
openapi: 3.1.0
info:
  title: AI Engineer API
  description: |
    AI Engineer Platform 提供 AI 模型调用、数据管理、工作流编排等能力。
    
    ## 核心能力
    - **模型管理**：支持主流大模型的统一调用
    - **向量存储**：内置向量数据库支持
    - **工作流**：可视化编排 AI 工作流
  version: 2.0.0
  contact:
    name: API Support
    email: api-support@example.com
    url: https://docs.example.com/support
  license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html

servers:
  - url: https://api.example.com/v2
    description: 生产环境
  - url: https://staging-api.example.com/v2
    description: 预发环境
  - url: http://localhost:3000/v2
    description: 本地开发

tags:
  - name: Models
    description: 模型管理相关接口
  - name: Embeddings
    description: 向量嵌入相关接口
  - name: Workflows
    description: 工作流编排相关接口
```

### 路径与操作定义

```yaml
paths:
  /models:
    get:
      operationId: listModels
      summary: 获取模型列表
      description: 返回所有可用的 AI 模型，支持分页和过滤
      tags: [Models]
      
      parameters:
        - name: provider
          in: query
          description: 按模型供应商过滤
          required: false
          schema:
            type: string
            enum: [openai, anthropic, google, local]
            default: openai
          
        - name: capability
          in: query
          description: 按能力过滤
          schema:
            type: array
            items:
              type: string
              enum: [chat, completion, embedding, vision]
          
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
          
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
          
        - name: X-Request-ID
          in: header
          description: 请求追踪 ID
          schema:
            type: string
            format: uuid
        
      responses:
        '200':
          description: 成功返回模型列表
          headers:
            X-Total-Count:
              description: 总记录数
              schema:
                type: integer
            X-Page:
              description: 当前页码
              schema:
                type: integer
          
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ModelList'
              example:
                data:
                  - id: gpt-4o
                    provider: openai
                    name: GPT-4o
                    capability: [chat, vision]
                    context_window: 128000
                    input_cost: 0.005
                    output_cost: 0.015
                pagination:
                  page: 1
                  limit: 20
                  total: 156
                  
        '400':
          $ref: '#/components/responses/BadRequest'
          
        '401':
          $ref: '#/components/responses/Unauthorized'
          
        '429':
          $ref: '#/components/responses/RateLimited'
          
      security:
        - BearerAuth: []
```

### 组件复用定义

```yaml
components:
  schemas:
    Model:
      type: object
      required: [id, provider, name, capability]
      properties:
        id:
          type: string
          description: 模型唯一标识符
          example: gpt-4o
        provider:
          type: string
          enum: [openai, anthropic, google, mistral, local]
        name:
          type: string
          description: 模型显示名称
          example: GPT-4o
        description:
          type: string
        capability:
          type: array
          items:
            type: string
            enum: [chat, completion, embedding, vision, audio]
        context_window:
          type: integer
          description: 上下文窗口大小（tokens）
        training_cutoff:
          type: string
          format: date
          description: 训练数据截止日期
        pricing:
          $ref: '#/components/schemas/Pricing'
        multimodal:
          type: boolean
          default: false
          
    ModelList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Model'
        pagination:
          $ref: '#/components/schemas/Pagination'
          
    Pricing:
      type: object
      properties:
        input:
          type: number
          format: float
          description: 输入价格（每百万 tokens）
        output:
          type: number
          format: float
          description: 输出价格（每百万 tokens）
        currency:
          type: string
          default: USD
          
    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        has_next:
          type: boolean
          
    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          description: 错误代码
          example: INVALID_PARAMETER
        message:
          type: string
          description: 错误描述
        details:
          type: object
          additionalProperties: true
        request_id:
          type: string
          format: uuid
          
  parameters:
    pageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1
        
    limitParam:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
        
  responses:
    BadRequest:
      description: 请求参数错误
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
          example:
            code: INVALID_PARAMETER
            message: "参数 'page' 必须为正整数"
            
    Unauthorized:
      description: 未授权访问
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
            
    RateLimited:
      description: 请求频率超限
      headers:
        Retry-After:
          schema:
            type: integer
            description: 重试等待时间（秒）
            
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: 使用 JWT Token 进行认证
      
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
```

---

## Scalar API 文档

### OpenAPI 规范进阶特性

#### Webhooks 定义

OpenAPI 3.1 引入了 Webhook 支持，可以在规范中定义服务器推送的事件：

```yaml
openapi: 3.1.0
info:
  title: Event-Driven API
  version: 1.0.0

webhooks:
  # 定义 webhook 端点
  user.created:
    post:
      description: 当新用户创建时触发
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserCreatedEvent'
            example:
              event_id: "evt_123"
              timestamp: "2024-01-15T10:30:00Z"
              data:
                user_id: "usr_456"
                email: "user@example.com"
                name: "John Doe"
      responses:
        '200':
          description: 成功接收

  order.completed:
    post:
      description: 当订单完成时触发
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/OrderCompletedEvent'
      responses:
        '200':
          description: 成功接收

components:
  schemas:
    UserCreatedEvent:
      type: object
      required: [event_id, timestamp, data]
      properties:
        event_id:
          type: string
          description: 事件唯一标识符
        timestamp:
          type: string
          format: date-time
        data:
          type: object
          properties:
            user_id:
              type: string
            email:
              type: string
              format: email
            name:
              type: string

    OrderCompletedEvent:
      type: object
      required: [event_id, timestamp, data]
      properties:
        event_id:
          type: string
        timestamp:
          type: string
          format: date-time
        data:
          type: object
          properties:
            order_id:
              type: string
            user_id:
              type: string
            total:
              type: number
              format: float
```

#### 链接与操作关联

OpenAPI 允许定义操作之间的关联关系：

```yaml
paths:
  /users/{userId}:
    get:
      operationId: getUser
      summary: 获取用户信息
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 用户信息
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
          links:
            userOrders:
              operationId: getUserOrders
              parameters:
                userId: $response.body#/id

  /users/{userId}/orders:
    get:
      operationId: getUserOrders
      summary: 获取用户订单
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 用户订单列表
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Order'
```

#### 高级安全配置

```yaml
components:
  securitySchemes:
    oauth2:
      type: oauth2
      description: OAuth 2.0 认证
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/authorize
          tokenUrl: https://auth.example.com/token
          scopes:
            read:users: 读取用户信息
            write:users: 管理用户
            read:orders: 读取订单
            write:orders: 管理订单
        clientCredentials:
          tokenUrl: https://auth.example.com/token
          scopes:
            api:full: 完全 API 访问

    openIdConnect:
      type: openIdConnect
      openIdConnectUrl: https://auth.example.com/.well-known/openid-configuration

    mutualTls:
      type: mutualTLS
      description: 双向 TLS 认证

# 全局安全要求
security:
  - oauth2:
      - read:users
      - write:users

paths:
  /admin/users:
    get:
      security:
        # 此端点需要更高权限
        - oauth2:
            - admin:users
```

---

## Scalar API 文档

### Scalar 核心特性

Scalar 是一款现代化 API 文档工具，提供精美的界面和丰富的交互能力：

```bash
# 安装 Scalar
npm install @scalar/api-reference

# 或使用 Docker
docker run -p 5050:80 \
  -v $(pwd)/openapi.yaml:/usr/share/nginx/html/openapi.yaml \
  nginx:alpine
```

### Scalar 高级配置

```typescript
// React 集成配置
const configuration = {
  // 主题配置
  theme: 'default' | 'purple' | 'blue' | 'space' | 'planet' | 'solarized' | 'mono',

  // 布局模式
  layout: 'classic' | 'modern',

  // 显示配置
  showSidebar: true,
  hideDownloadButton: false,
  hideModels: false,
  hideInternal: false,

  // 搜索配置
  searchHotkey: '/',

  // Swagger 客户端
  forceHttpClient: 'fetch' | 'xhr',

  // 代理配置
  proxyUrl: 'https://api.example.com',

  // 认证配置
  authentication: {
    http: {
      basic: { username: '', password: '' },
      bearer: { token: '' },
      apiKey: { token: '' },
    },
  },
};
```

### Scalar 主题定制

```javascript
// 自定义主题示例
const customTheme = {
  colors: {
    // 主色调
    accent: {
      50: '#f0f9ff',
      100: '#e0f2fe',
      200: '#bae6fd',
      300: '#7dd3fc',
      400: '#38bdf8',
      500: '#0ea5e9',
      600: '#0284c7',
      700: '#0369a1',
      800: '#075985',
      900: '#0c4a6e',
    },
    // 背景色
    background: {
      solid: '#ffffff',
      subtle: '#f8fafc',
      muted: '#f1f5f9',
    },
    // 文字色
    foreground: {
      base: '#0f172a',
      muted: '#64748b',
    },
  },
  fonts: {
    body: 'Inter, system-ui, sans-serif',
    mono: 'JetBrains Mono, monospace',
  },
  radii: {
    sm: '4px',
    md: '8px',
    lg: '12px',
    full: '9999px',
  },
};

// 应用自定义主题
configuration.customCss = `
  :root {
    --scalar-color: ${customTheme.colors.accent[500]};
    --scalar-color-accent: ${customTheme.colors.accent[600]};
    --scalar-background: ${customTheme.colors.background.solid};
    --scalar-background-2: ${customTheme.colors.background.subtle};
    --scalar-border: ${customTheme.colors.background.muted};
    --scalar-font-family: ${customTheme.fonts.body};
    --scalar-font-family-mono: ${customTheme.fonts.mono};
    --scalar-radius: ${customTheme.radii.md};
  }

  .scalar-api-reference {
    border-radius: ${customTheme.radii.lg};
  }
`;
```

---

## Mintlify 文档工具

### Mintlify 核心特性

Mintlify 以其现代化设计和开发者友好的体验著称：

```bash
# 安装 Mintlify CLI
npm install -g mintlify

# 初始化项目
mintlify init

# 启动开发服务器
mintlify dev
```

### Mintlify 页面配置

Mintlify 的页面通过 MDX 格式编写，支持丰富的组件：

```mdx
---
title: 用户管理 API
description: 管理用户账户的完整 API 参考
sidebar: true
tableOfContents: true
api: GET /v2/users
---

import { Card, Callout, Tabs, Tab } from '@mintlify/components'

## 概述

用户管理 API 提供创建、读取、更新和删除用户账户的能力。所有需要认证的端点都必须在请求头中包含有效的 Bearer Token。

<Callout type="info" title="速率限制">
  此 API 的速率限制为每分钟 100 个请求，超出限制将返回 429 状态码。
</Callout>

## 认证

此 API 使用 Bearer Token 认证。在请求头的 `Authorization` 字段中传递您的 API Key：

```
Authorization: Bearer YOUR_API_KEY
```

<Card title="获取 API Key" icon="key" href="/api-reference/authentication">
  了解如何在控制台中获取和管理您的 API Key
</Card>

## 端点

### 获取用户列表

<Tabs>
  <Tab title="请求">
    ```http
    GET /v2/users
    ```
  </Tab>
  <Tab title="响应">
    ```json
    {
      "data": [
        {
          "id": "usr_1a2b3c",
          "email": "alice@example.com",
          "name": "Alice Johnson",
          "created_at": "2024-01-15T10:30:00Z"
        }
      ],
      "meta": {
        "total": 150,
        "page": 1,
        "per_page": 20
      }
    }
    ```
  </Tab>
</Tabs>

### 参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| page | integer | 否 | 页码，默认为 1 |
| per_page | integer | 否 | 每页数量，默认 20，最大 100 |
| search | string | 否 | 搜索用户名或邮箱 |

### 创建用户

```typescript
// TypeScript 示例
import { API } from '@your-org/sdk';

const api = new API({ apiKey: process.env.API_KEY });

const user = await api.users.create({
  email: 'newuser@example.com',
  name: 'New User',
  password: 'securepassword123',
  role: 'member'
});

console.log(user.id); // usr_xxxxx
```
```

---

## Redocly 文档平台

### Redocly 核心特性

Redocly 提供企业级 API 文档解决方案：

```bash
# 安装 Redocly CLI
npm install -g @redocly/cli

# 启动文档服务器
redocly preview-docs openapi.yaml
```

### Redocly 主题定制

```yaml
# redocly.yaml
apiDefinitions:
  main: ./openapi.yaml

theme:
  colors:
    primary:
      main: '#3c8772'
    http:
      get: '#10b981'
      post: '#3b82f6'
      put: '#f59e0b'
      delete: '#ef4444'
      patch: '#8b5cf6'
  typography:
    fontFamily: 'Inter, system-ui, sans-serif'
    codeFontFamily: 'JetBrains Mono, monospace'
    fontSize: '14px'
    lineHeight: '1.5'
  sidebar:
    backgroundColor: '#f8fafc'
    width: '280px'
  logo:
    gutter: '0px'
  codeBlock:
    backgroundColor: '#1e293b'
    borderRadius: '8px'
  navbar:
    backgroundColor: '#ffffff'
    height: '64px'
  rightPanel:
    backgroundColor: '#f8fafc'
    width: '40%'

extends:
  - recommended-types

rules:
  oas3-api-servers: warn
  operation-description: off
  no-unresolved-refs: error
```

### Redocly 分割与引用

对于大型 API 规范，Redocly 支持将规范文件分割为多个文件：

```yaml
# openapi.yaml - 主文件
openapi: 3.1.0
info:
  title: Enterprise API
  version: 2.0.0
  description: |
    Enterprise API 提供完整的企业管理功能。

servers:
  - url: https://api.example.com/v2
    description: 生产环境
  - url: https://staging-api.example.com/v2
    description: 预发环境

paths:
  /users:
    $ref: ./paths/users.yaml
  /orders:
    $ref: ./paths/orders.yaml
  /products:
    $ref: ./paths/products.yaml

components:
  schemas:
    $ref: ./components/schemas.yaml
  responses:
    $ref: ./components/responses.yaml
  parameters:
    $ref: ./components/parameters.yaml
  securitySchemes:
    $ref: ./components/security.yaml
```

```yaml
# paths/users.yaml
get:
  operationId: listUsers
  summary: 获取用户列表
  tags: [Users]
  parameters:
    - $ref: ../components/parameters.yaml#/pageParam
    - $ref: ../components/parameters.yaml#/limitParam
  responses:
    '200':
      $ref: ../components/responses.yaml#/usersList
    '401':
      $ref: ../components/responses.yaml#/unauthorized

post:
  operationId: createUser
  summary: 创建用户
  tags: [Users]
  requestBody:
    content:
      application/json:
        schema:
          $ref: ../components/schemas.yaml#/UserCreate
  responses:
    '201':
      $ref: ../components/responses.yaml#/userCreated
    '400':
      $ref: ../components/responses.yaml#/badRequest
```

---

## 工具对比分析

### 功能对比矩阵

| 特性 | Scalar | Mintlify | Redocly |
|------|--------|----------|---------|
| **UI 美观度** | 极佳 | 极佳 | 良好 |
| **响应式设计** | 是 | 是 | 是 |
| **暗黑模式** | 是 | 是 | 是 |
| **主题定制** | CSS 变量 | JSON 配置 | YAML 配置 |
| **搜索功能** | 内置 | 内置 | 需插件 |
| **代码生成** | 是 | 是 | 部分 |
| **Mock 服务器** | 是 | 是 | 是 |
| **API 测试** | 是 | 是 | 是 |
| **国际化** | 部分 | 部分 | 是 |
| **部署方式** | 自托管/SaaS | SaaS/自托管 | 自托管/SaaS |
| **免费额度** | 无限 | 有限 | 无限 |
| **开源** | 部分 | 否 | CLI 开源 |

### 定价对比

| 方案 | Scalar | Mintlify | Redocly |
|------|--------|----------|---------|
| **免费** | 无限公开 API | 1 项目 | 无限文档 |
| **Pro** | $15/月/人 | $15/月 | 需联系 |
| **Enterprise** | 定制 | 定制 | 定制 |

### 选择指南

根据不同场景选择合适的工具：

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 公开 API | Scalar/Mintlify | UI 更精美，开发者体验更好 |
| 内部 API | Redocly | 更高的定制灵活性和权限控制 |
| 文档即代码 | 均可 | 都支持 OpenAPI 规范 |
| 快速启动 | Mintlify | 最简单的配置方式 |
| 企业级需求 | Redocly | 最成熟的企业功能 |
| 预算有限 | Scalar | 免费版功能最全面 |

```typescript
// React 集成
import { ApiReference } from '@scalar/api-reference'
import '@scalar/api-reference/style.css'

function App() {
  return (
    <ApiReference 
      configuration={{
        spec: {
          url: '/openapi.yaml',
          // 或直接传入对象
          // content: openApiSpec,
        },
        theme: 'purple',
        layout: 'classic',
        proxyUrl: 'https://api.example.com',  // 代理配置
        authentication: {
          apiKey: {
            token: process.env.API_KEY,
          },
        },
      }}
    />
  )
}
```

### Scalar 配置选项

```typescript
const configuration = {
  // 主题配置
  theme: 'default' | 'purple' | 'blue' | 'space' | 'planet' | 'solarized' | 'mono',
  
  // 布局模式
  layout: 'classic' | 'modern',
  
  // 显示配置
  showSidebar: true,
  hideDownloadButton: false,
  hideModels: false,
  hideInternal: false,
  
  // 自定义 CSS
  customCss: `
    :root {
      --scalar-color-1: #5a19d9;
    }
  `,
  
  // 搜索配置
  searchHotkey: '/',
  
  // Swagger 客户端
  forceHttpClient: 'fetch' | 'xhr',
  
  // 代理配置
  proxyUrl: 'https://api.example.com',
  
  // 认证配置
  authentication: {
    http: {
      basic: { username: '', password: '' },
      bearer: { token: '' },
      apiKey: { token: '' },
    },
  },
}
```

### Scalar 部署方式

```yaml
# docker-compose.yaml
version: '3.8'
services:
  api-reference:
    image:Ghcr.io/scalar/scalar-api-reference:latest
    ports:
      - '5050:80'
    volumes:
      - ./openapi.yaml:/usr/share/nginx/html/openapi.yaml:ro
    environment:
      - TITLE=My API Documentation
      - SPEC_URL=/openapi.yaml
```

---

## Mintlify 文档工具

### Mintlify 核心特性

Mintlify 以其现代化设计和开发者友好的体验著称：

```bash
# 安装 Mintlify CLI
npm install -g mintlify

# 初始化项目
mintlify init

# 启动开发服务器
mintlify dev
```

```yaml
# mint.json - Mintlify 配置文件
{
  "name": "AI Engineer API",
  "logo": "/logo.svg",
  "favicon": "/favicon.ico",
  "logoHeight": "80px",
  
  "colors": {
    "primary": "#3c8772",
    "light": "#f0fdf9",
    "dark": "#0f172a"
  },
  
  "favicon": "/favicon.png",
  "background": "#f9fafb",
  
  "navigation": [
    {
      "group": "Getting Started",
      "pages": ["introduction", "quickstart"]
    },
    {
      "group": "API Reference",
      "pages": [
        "api-reference/authentication",
        "api-reference/models",
        "api-reference/embeddings"
      ]
    },
    {
      "group": "SDKs",
      "pages": ["sdks/python", "sdks/javascript"]
    }
  ],
  
  "footerSocials": {
    "github": "https://github.com/example",
    "twitter": "https://twitter.com/example"
  },
  
  "api": {
    "baseUrl": "https://api.example.com",
    "auth": {
      "method": "bearer",
      "placeholder": "Enter your API key"
    }
  }
}
```

### Mintlify 页面配置

```yaml
---
title: 获取模型列表
description: 获取所有可用的 AI 模型，支持分页和过滤
sidebar: true
tableOfContents: true
api: GET /v2/models
---
```

```mdx
import { Card, Callout } from '@mintlify/components'

<Card 
  title="Python SDK" 
  icon="python" 
  href="/sdks/python"
>
  使用 Python SDK 快速集成
</Card>

<Callout type="info" title="认证要求">
  此接口需要有效的 API Key，可在 [控制台](https://console.example.com) 获取。
</Callout>

## 请求参数

| 参数 | 类型 | 必填 | 描述 |
|------|------|------|------|
| provider | string | 否 | 模型供应商 |
| capability | string[] | 否 | 模型能力 |
| page | number | 否 | 页码，默认 1 |
| limit | number | 否 | 每页数量，默认 20 |

## 响应示例

```json
{
  "data": [
    {
      "id": "gpt-4o",
      "name": "GPT-4o",
      "provider": "openai",
      "capability": ["chat", "vision"]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156
  }
}
```

## 代码示例

```javascript
import { AIEngine } from '@ai-engine/sdk';

const client = new AIEngine({ apiKey: process.env.API_KEY });

const models = await client.models.list({
  provider: 'openai',
  capability: ['chat']
});

console.log(models);
```
```

---

## Redocly 文档平台

### Redocly 核心特性

Redocly 提供企业级 API 文档解决方案：

```bash
# 安装 Redocly CLI
npm install -g @redocly/cli

# 启动文档服务器
redocly preview-docs openapi.yaml
```

```yaml
# redocly.yaml - Redocly 配置文件
apiDefinitions:
  main: ./openapi.yaml
  
theme:
  colors:
    primary:
      main: '#3c8772'
  typography:
    fontFamily: 'Inter, system-ui, sans-serif'
    codeFontFamily: 'JetBrains Mono, monospace'
  sidebar:
    backgroundColor: '#f8fafc'
    
extends:
  - recommended-types

rules:
  oas3-api-servers: warn
  operation-description: off
  no-unresolved-refs: error
  
lint:
  rules:
    no-unused-components: warn
```

```typescript
// React 集成
import { RedocStandalone } from 'redoc'

RedocStandalone({
  specUrl: '/openapi.yaml',
  options: {
    theme: {
      colors: {
        primary: {
          main: '#3c8772',
        },
      },
    },
  },
  documentTitle: 'AI Engineer API',
  onDidMount: () => {
    console.log('Docs mounted');
  },
})
```

### Redocly 工作区配置

```yaml
# .redocly.yaml
apis:
  core@latest:
    root: ./openapi.yaml
  
  staging@staging:
    root: ./openapi-staging.yaml

extends:
  - recommended

plugins:
  - './custom-plugin.js'

rules:
  no-unresolved-refs: error
  operation-operationId-unique: error
  operation-description: off

regions:
  - us-east-1
  - eu-west-1
```

---

## 工具对比分析

### 功能对比矩阵

| 特性 | Scalar | Mintlify | Redocly |
|------|--------|----------|---------|
| **UI 美观度** | 极佳 | 极佳 | 良好 |
| **响应式设计** | 是 | 是 | 是 |
| **暗黑模式** | 是 | 是 | 是 |
| **主题定制** | CSS 变量 | JSON 配置 | YAML 配置 |
| **搜索功能** | 内置 | 内置 | 需插件 |
| **代码生成** | 是 | 是 | 部分 |
| **Mock 服务器** | 是 | 是 | 是 |
| **API 测试** | 是 | 是 | 是 |
| **国际化** | 部分 | 部分 | 是 |
| **部署方式** | 自托管/SaaS | SaaS/自托管 | 自托管/SaaS |
| **免费额度** | 无限 | 有限 | 无限 |
| **开源** | 部分 | 否 | CLI 开源 |

### 定价对比

| 方案 | Scalar | Mintlify | Redocly |
|------|--------|----------|---------|
| **免费** | 无限公开 API | 1 项目 | 无限文档 |
| **Pro** | $15/月/人 | $15/月 | 需联系 |
| **Enterprise** | 定制 | 定制 | 定制 |

---

## 文档即代码工作流

### Git 仓库结构

```
api-docs/
├── openapi/
│   ├── openapi.yaml           # 主规范文件
│   ├── paths/                 # 路径定义
│   │   ├── models.yaml
│   │   ├── embeddings.yaml
│   │   └── workflows.yaml
│   ├── components/            # 组件复用
│   │   ├── schemas/
│   │   ├── responses.yaml
│   │   └── parameters.yaml
│   └── parameters/
│       ├── common.yaml
│       └── pagination.yaml
│
├── src/
│   └── sdk/                  # SDK 源码
│       ├── python/
│       └── typescript/
│
├── tests/
│   └── api-tests/            # API 测试
│
├── docs/                     # Mintlify 页面
│   ├── introduction.mdx
│   ├── quickstart.mdx
│   └── sdks/
│
├── scripts/
│   ├── validate-spec.js
│   ├── generate-sdk.ts
│   └── sync-docs.js
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy-docs.yml
│
├── .redocly.yaml
├── mint.json
└── package.json
```

### CI/CD 自动化

```yaml
# .github/workflows/api-docs.yml
name: API Documentation CI

on:
  push:
    branches: [main]
    paths:
      - 'openapi/**'
      - 'docs/**'
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Validate OpenAPI spec
        run: npx redocly lint openapi/openapi.yaml
        
      - name: Check refs resolve
        run: npm run validate:refs
        
      - name: Run API tests
        run: npm test
        
  build-docs:
    runs-on: ubuntu-latest
    needs: validate
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install Mintlify
        run: npm install -g mintlify
        
      - name: Build docs
        run: mintlify build
        
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: docs-build
          path: .mintlify/build

  deploy:
    runs-on: ubuntu-latest
    needs: build-docs
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://docs.example.com
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v4
        with:
          name: docs-build
          
      - name: Deploy to CDN
        run: mintlify deploy --prod
        env:
          MINTLIFY_API_KEY: ${{ secrets.MINTLIFY_API_KEY }}
```

### SDK 自动生成

```typescript
// scripts/generate-sdk.ts
import { generate } from 'openapi-typescript-codegen';

async function generateSDK() {
  await generate({
    input: './openapi/openapi.yaml',
    output: './src/sdk/typescript',
    client: 'fetch',
    
    // 自定义模板
    templates: './templates/sdk',
    
    // 导出格式
    exportSchemas: true,
    exportServices: true,
    
    // 命名规范
    useDateType: true,
    useUnionTypes: true,
    
    // HTTP 客户端
    httpClient: 'fetch',
    
    // 导入前缀
    importDocumentation: '../../docs',
  });
  
  console.log('SDK generated successfully!');
}

generateSDK().catch(console.error);
```

---

## 最佳实践

### OpenAPI 规范最佳实践

> [!TIP] OpenAPI 编写建议
> - 始终使用 OpenAPI 3.x 版本
> - 保持规范文件的模块化和可维护性
> - 使用 `$ref` 复用组件定义
> - 为每个操作提供详细的描述和示例
> - 使用 tags 组织相关操作
> - 实现版本控制和变更管理

```yaml
# 推荐的目录结构
openapi/
├── openapi.yaml          # 主文件，import 其他文件
├── info.yaml             # info 对象
├── servers.yaml          # 服务器定义
├── paths/
│   ├── index.yaml        # 导入所有路径
│   ├── users.yaml
│   ├── products.yaml
│   └── orders.yaml
├── components/
│   ├── schemas/
│   │   ├── index.yaml
│   │   ├── user.yaml
│   │   └── error.yaml
│   ├── responses.yaml
│   ├── parameters.yaml
│   └── security.yaml
└── tags.yaml
```

### 安全最佳实践

```yaml
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        JWT Token 获取方式：
        1. 在控制台创建 API Key
        2. 使用 API Key 换取 Access Token
        3. 在请求 Header 中携带 Token
        
    ApiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: |
        API Key 可在 [开发者控制台](https://console.example.com) 创建。
        请妥善保管，不要泄露或提交到代码仓库。

paths:
  /models:
    get:
      security:
        - BearerAuth: []
        - ApiKeyAuth: []
```

---

## 高级配置与技巧

### API 文档版本管理

```yaml
# .redocly.yaml - 多版本配置
apis:
  core@latest:
    root: ./openapi/v1.yaml
    definition:
      labels:
        - Latest
        - Recommended

  core@v1:
    root: ./openapi/v1.yaml

  core@v2:
    root: ./openapi/v2.yaml

  core@deprecated:
    root: ./openapi/deprecated.yaml
    definition:
      labels:
        - Deprecated

ui:
  tabs:
    - api
    - guidelines
  navigation:
    - label: Getting Started
      page: introduction
    - label: Guides
      page: guides/index
    - label: API Reference
      items:
        - label: Core API
          version: core
    - label: Resources
      page: resources/index
```

### 自动化文档生成

```typescript
// scripts/generate-api-docs.ts
import { generate } from 'openapi-typescript-codegen';
import { generateSdk } from '@openapitools/openapi-generator-cli';
import { lint } from '@redocly/cli';
import fs from 'fs/promises';
import path from 'path';

interface DocConfig {
  title: string;
  version: string;
  outputDir: string;
}

async function generateApiDocs(config: DocConfig) {
  console.log('Generating API documentation...');

  // 1. 生成 TypeScript 类型定义
  await generate({
    input: './openapi.yaml',
    output: './src/types/api.ts',
    httpClient: 'fetch',
    useDateType: true,
    exportSchemas: true,
  });

  // 2. 生成 SDK
  await generateSdk({
    generatorName: 'typescript-fetch',
    inputSpec: './openapi.yaml',
    outputDir: './src/sdk',
    additionalProperties: {
      supportsES6: 'true',
      modelPropertyNaming: 'camelCase',
      useSingleRequestParameter: 'true',
    },
  });

  // 3. 验证 OpenAPI 规范
  try {
    await lint({
      config: {
        apis: {
          main: {
            root: './openapi.yaml',
          },
        },
      },
    });
    console.log('OpenAPI validation passed');
  } catch (error) {
    console.error('OpenAPI validation failed:', error);
    throw error;
  }

  // 4. 生成文档
  await buildDocs();

  console.log('API documentation generated successfully!');
}

async function buildDocs() {
  // 使用 Redocly 构建文档
  const { execSync } = await import('child_process');
  execSync('redocly build-docs openapi.yaml --output=dist/index.html', {
    stdio: 'inherit',
  });
}

// 运行
generateApiDocs({
  title: 'API Documentation',
  version: '2.0.0',
  outputDir: './docs',
}).catch(console.error);
```

---

## 常见问题与解决方案

### 问题 1：OpenAPI 规范过于庞大

**症状**：单个规范文件太大，难以维护

**解决方案**：

```yaml
# 使用 $ref 分割规范文件
# openapi.yaml
openapi: 3.1.0
info:
  $ref: ./info.yaml

servers:
  $ref: ./servers.yaml

paths:
  $ref: ./paths/index.yaml

components:
  schemas:
    $ref: ./components/schemas.yaml
  responses:
    $ref: ./components/responses.yaml
  parameters:
    $ref: ./components/parameters.yaml
  securitySchemes:
    $ref: ./components/security.yaml
```

### 问题 2：文档更新不及时

**症状**：代码更新后文档忘记同步

**解决方案**：

```yaml
# .github/workflows/api-docs.yml
name: Sync API Documentation

on:
  push:
    branches: [main]
    paths:
      - 'src/**/*'
  pull_request:
    paths:
      - 'src/**/*'

jobs:
  sync-docs:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      - name: Generate OpenAPI spec
        run: npm run generate:openapi

      - name: Update documentation
        run: npm run docs:update

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          title: "chore: Auto-update API documentation"
          commit-message: "chore: Auto-update API documentation"
          body: |
            This PR was automatically created to update the API documentation.
            Please review and merge.
          branch: chore/update-api-docs
```

### 问题 3：安全性考虑

**症状**：敏感信息泄露到文档中

**解决方案**：

```yaml
# 使用服务器变量替代敏感信息
servers:
  - url: https://{environment}.api.example.com/v2
    variables:
      environment:
        default: prod
        enum: [dev, staging, prod]
        description: API 环境

# 使用 example 替代真实数据
components:
  schemas:
    User:
      properties:
        email:
          type: string
          example: user@example.com  # 示例邮箱，非真实
        api_key:
          type: string
          description: 此字段不会出现在文档中
          example: null
```

---

## 实战项目示例

### 项目：完整的 API 文档系统

**需求**：为电商平台构建完整的 API 文档系统

**实现**：

```yaml
# openapi.yaml - 电商 API 规范
openapi: 3.1.0
info:
  title: E-Commerce Platform API
  description: |
    电商平台 API 提供完整的电商功能支持，包括：

    - **用户管理**：注册、登录、个人信息管理
    - **商品管理**：商品列表、详情、搜索
    - **订单处理**：创建订单、支付、物流跟踪
    - **支付集成**：支持多种支付方式
    - **评价系统**：商品评价和回复

    ## 认证

    除公开端点外，所有请求都需要 Bearer Token 认证。
  version: 2.0.0
  contact:
    name: API Support
    email: api@example.com
    url: https://docs.example.com/support
  license:
    name: Apache 2.0
    url: https://www.apache.org/licenses/LICENSE-2.0.html

servers:
  - url: https://api.example.com/v2
    description: 生产环境
  - url: https://staging-api.example.com/v2
    description: 预发环境
  - url: http://localhost:3000/v2
    description: 本地开发

tags:
  - name: Authentication
    description: 用户认证相关接口
  - name: Users
    description: 用户管理
  - name: Products
    description: 商品管理
  - name: Orders
    description: 订单处理
  - name: Payments
    description: 支付相关
  - name: Reviews
    description: 评价系统

paths:
  # 认证端点
  /auth/register:
    $ref: ./paths/auth/register.yaml
  /auth/login:
    $ref: ./paths/auth/login.yaml
  /auth/refresh:
    $ref: ./paths/auth/refresh.yaml

  # 用户端点
  /users/me:
    $ref: ./paths/users/me.yaml
  /users/me/addresses:
    $ref: ./paths/users/addresses.yaml

  # 商品端点
  /products:
    $ref: ./paths/products/list.yaml
  /products/{productId}:
    $ref: ./paths/products/detail.yaml
  /products/search:
    $ref: ./paths/products/search.yaml

  # 订单端点
  /orders:
    $ref: ./paths/orders/list.yaml
  /orders/{orderId}:
    $ref: ./paths/orders/detail.yaml
  /orders:
    post:
      $ref: ./paths/orders/create.yaml

  # 支付端点
  /payments/checkout:
    $ref: ./paths/payments/checkout.yaml
  /payments/{paymentId}:
    $ref: ./paths/payments/detail.yaml
  /payments/webhook:
    $ref: ./paths/payments/webhook.yaml

  # 评价端点
  /products/{productId}/reviews:
    $ref: ./paths/reviews/list.yaml
  /reviews/{reviewId}:
    $ref: ./paths/reviews/detail.yaml

components:
  schemas:
    $ref: ./components/schemas.yaml
  responses:
    $ref: ./components/responses.yaml
  parameters:
    $ref: ./components/parameters.yaml
  securitySchemes:
    $ref: ./components/security.yaml
```

---

## 进阶使用技巧

### API 文档国际化

```yaml
# 支持多语言的配置
openapi: 3.1.0
info:
  title:
    en: "User Management API"
    zh-CN: "用户管理 API"
    ja: "ユーザー管理 API"
  description:
    en: "API for managing user accounts"
    zh-CN: "用于管理用户账户的 API"
    ja: "ユーザーアカウントを管理するためのAPI"

paths:
  /users:
    get:
      summary:
        en: "List Users"
        zh-CN: "获取用户列表"
        ja: "ユーザー一覧"
      description:
        en: "Returns a paginated list of users"
        zh-CN: "返回分页的用户列表"
        ja: "ページネーションされたユーザー一覧を返します"
```

### 文档访问控制

```yaml
# 基于标签的访问控制
tags:
  - name: Admin
    description: Administrative operations (requires admin role)
  - name: Public
    description: Public endpoints

paths:
  /admin/users:
    get:
      tags: [Admin]
      security:
        - oauth2: [admin:read]
      summary: Admin - List all users

  /public/products:
    get:
      tags: [Public]
      summary: Public - List products
```

### 响应压缩与缓存

```yaml
# OpenAPI 扩展配置
x-servers:
  - url: https://api.example.com/v2
    description: Production server
    variables:
      cache:
        default: "enabled"
        enum: ["enabled", "disabled"]
      compression:
        default: "gzip"
        enum: ["gzip", "deflate", "none"]

paths:
  /users:
    get:
      operationId: getUsers
      x-rate-limit:
        requests: 100
        period: 60
      x-response-cache:
        enabled: true
        ttl: 300
```

---

## 开发者体验优化

### SDK 生成配置

```json
// openapi-generator-config.json
{
  "generatorName": "typescript-fetch",
  "inputSpec": "./openapi.yaml",
  "outputDir": "./src/sdk",
  "additionalProperties": {
    "supportsES6": "true",
    "modelPropertyNaming": "camelCase",
    "npmName": "@company/api-sdk",
    "npmVersion": "1.0.0",
    "typescriptThreePlus": "true",
    "useSingleRequestParameter": "false",
    "prettier": "true",
    "eslint": "true",
    "fetchImplementation": "true"
  },
  "typeMappings": {
    "DateTime": "Date",
    "UUID": "string"
  },
  "importMappings": {
    "User": "./models/User"
  }
}
```

### 文档站点配置

```javascript
// docsite.config.js
module.exports = {
  title: 'API Documentation',
  description: 'Complete API reference',
  favicon: '/favicon.ico',
  logo: {
    src: '/logo.svg',
    width: 120,
    height: 40
  },

  navigation: {
    top: [
      { label: 'Home', path: '/' },
      { label: 'API Reference', path: '/api' },
      { label: 'Guides', path: '/guides' },
      { label: 'Changelog', path: '/changelog' }
    ],
    bottom: [
      { label: 'Terms', path: '/terms' },
      { label: 'Privacy', path: '/privacy' }
    ]
  },

  search: {
    enabled: true,
    placeholder: 'Search documentation...',
    hotkey: '/'
  },

  theming: {
    primaryColor: '#3b82f6',
    darkMode: true,
    codeTheme: 'github-dark'
  },

  versioning: {
    current: 'v2',
    versions: ['v2', 'v1'],
    defaultVersion: 'v2'
  },

  analytics: {
    enabled: true,
    provider: 'plausible'
  }
};
```

---

## API 文档最佳实践

### 命名规范

```yaml
# 路径命名规范
paths:
  # 使用名词而非动词
  /users:           # ✅ 正确
  /getUsers:        # ❌ 错误

  # 使用复数形式
  /users:           # ✅ 正确
  /user:            # ❌ 错误

  # 使用 kebab-case
  /user-profiles:   # ✅ 正确
  /userProfiles:     # ❌ 错误

  # 嵌套资源不超过两层
  /users/{userId}/orders:  # ✅ 正确
  /users/{userId}/orders/{orderId}/items/{itemId}:  # ❌ 太深

# 操作 ID 命名
paths:
  /users:
    get:
      operationId: listUsers        # ✅ 动词 + 名词
    post:
      operationId: createUser
    put:
      operationId: updateUser
    delete:
      operationId: deleteUser
```

### 响应设计规范

```yaml
paths:
  /users/{userId}:
    get:
      responses:
        '200':
          description: 用户信息
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'
              example:
                id: "usr_123"
                name: "张三"
                email: "zhang@example.com"
                createdAt: "2024-01-15T10:30:00Z"

        '404':
          description: 用户不存在
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
              example:
                code: "USER_NOT_FOUND"
                message: "用户不存在"
                requestId: "req_abc123"

        '500':
          description: 服务器错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
```

### 错误处理规范

```yaml
components:
  schemas:
    Error:
      type: object
      required: [code, message]
      properties:
        code:
          type: string
          description: 错误代码
          example: VALIDATION_ERROR
        message:
          type: string
          description: 错误消息
          example: "参数验证失败"
        details:
          type: array
          description: 详细错误信息
          items:
            type: object
            properties:
              field:
                type: string
                example: "email"
              issue:
                type: string
                example: "Invalid email format"
        requestId:
          type: string
          description: 请求追踪 ID
          example: "req_abc123"
        timestamp:
          type: string
          format: date-time
          example: "2024-01-15T10:30:00Z"

    ValidationError:
      allOf:
        - $ref: '#/components/schemas/Error'
        - type: object
          properties:
            errors:
              type: array
              items:
                type: object
                properties:
                  field:
                    type: string
                  message:
                    type: string
```

---

## 相关资源

- [[OpenAPI 规范官方文档]]
- [[Swagger/OpenAPI 工具对比]]
- [[API 设计最佳实践]]
- [[文档即代码理念]]
- [[Mintlify vs Scalar vs Redocly]]

---

*本文档由 [[归愚知识系统]] 自动生成*
