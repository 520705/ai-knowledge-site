# REST API 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 REST 约束、URL 设计、HTTP 语义、状态码、认证方案及 OpenAPI 规范。

---

## 目录

1. [[#REST 概述与核心约束]]
2. [[#REST API 设计原则]]
3. [[#URL 设计规范]]
4. [[#HTTP 方法语义]]
5. [[#HTTP 状态码速查表]]
6. [[#版本控制策略]]
7. [[#认证与授权]]
8. [[#OpenAPI 规范]]
9. [[#错误响应格式]]
10. [[#REST vs GraphQL vs tRPC]]
11. [[#AI 应用实战]]

---

## REST 概述与核心约束

### REST 定义

REST（Representational State Transfer）由 Roy Fielding 在 2000 年提出，是一种**分布式超媒体系统**的软件架构风格。REST 不是协议或标准，而是一组设计原则和约束。

### Fielding 六原则

| 约束 | 说明 | 实现要点 |
|------|------|---------|
| **客户端-服务器** | 关注点分离 | 前端/后端独立演进 |
| **无状态** | 请求包含所有上下文 | 无会话，JWT/Token 携带状态 |
| **缓存** | 响应可标记为可缓存 | Cache-Control, ETag |
| **统一接口** | 标准化资源操作 | URL 表示资源，HTTP 动词操作 |
| **分层系统** | 架构分层 | API Gateway, Load Balancer |
| **按需代码** | 可下载执行代码（可选） | JavaScript, WebAssembly |

### REST 核心概念

| 概念 | 说明 | 示例 |
|------|------|------|
| **资源（Resource）** | 任何可命名的信息 | `/users`, `/orders/123` |
| **表示（Representation）** | 资源的表现形式 | JSON, XML, HTML |
| **状态转移** | 通过表示操作资源状态 | POST 创建, PUT 更新 |
| **超媒体** | 响应包含相关链接 | HATEOAS |

---

## REST API 设计原则

### 设计原则清单

> [!IMPORTANT]
> 遵循以下原则设计 REST API：

1. **以资源为中心**：使用名词而非动词
   - ✅ `GET /users/123`
   - ❌ `GET /getUser/123`

2. **使用 HTTP 方法语义**：GET/POST/PUT/PATCH/DELETE

3. **层级结构 URL**：反映资源关系
   - `/users/123/orders` → 用户 123 的订单

4. **使用复数名词**：保持一致性
   - ✅ `/users`, `/products`
   - ❌ `/user`, `/product`

5. **使用 kebab-case 或 snake_case**：
   - ✅ `/user-profiles`, `/user_profiles`
   - ❌ `/userProfiles`

6. **版本控制**：URL 或 Header
   - ✅ `/api/v1/users`
   - ✅ `Accept: application/vnd.api+json; version=1`

7. **分页**：使用查询参数
   - ✅ `/users?page=1&limit=20`

8. **过滤与排序**：查询参数
   - ✅ `/users?status=active&sort=name:asc`

---

## URL 设计规范

### URL 结构

```
https://api.example.com/v1/users/123/orders?page=1&limit=20

\__/  \________/ \__/\__/\__/\______________/\______________/
 |         |       |   |   |        |              |
协议    域名       版本 资源  ID    子资源        查询参数
```

### 资源命名规范

| 操作 | URL 模式 | 示例 |
|------|---------|------|
| 集合 | `/resources` | `/users`, `/products` |
| 单个资源 | `/resources/:id` | `/users/123` |
| 子资源 | `/resources/:id/sub` | `/users/123/orders` |
| 跨资源关系 | `/resources?field=value` | `/orders?userId=123` |

### 嵌套层级限制

> [!TIP]
> 嵌套层级建议不超过 2-3 层：

```
✅ 推荐
/users/123
/users/123/orders

⚠️ 可接受
/users/123/orders/456

❌ 避免
/users/123/orders/456/items/789/invoices
```

### 查询参数使用

| 用途 | 参数 | 示例 |
|------|------|------|
| **分页** | `page`, `limit` | `/users?page=1&limit=20` |
| **排序** | `sort` | `/users?sort=name:asc,createdAt:desc` |
| **过滤** | `field=value` | `/users?status=active&role=admin` |
| **搜索** | `q`, `query` | `/users?q=john` |
| **字段选择** | `fields` | `/users?fields=id,name,email` |
| **展开** | `expand` | `/users?expand=orders` |

---

## HTTP 方法语义

### 方法对比表

| 方法 | 语义 | 幂等性 | 安全性 | 用途 |
|------|------|--------|--------|------|
| **GET** | 读取资源 | ✅ | ✅ | 查询 |
| **POST** | 创建资源 | ❌ | ❌ | 创建 |
| **PUT** | 完全替换 | ✅ | ❌ | 完整更新 |
| **PATCH** | 部分更新 | ❌ | ❌ | 部分更新 |
| **DELETE** | 删除资源 | ✅ | ❌ | 删除 |
| **HEAD** | 读取头信息 | ✅ | ✅ | 检查存在性 |
| **OPTIONS** | 支持的方法 | ✅ | ✅ | CORS 预检 |

### 方法详细说明

**GET - 读取资源：**

```http
GET /api/v1/users/123 HTTP/1.1
Host: api.example.com
Accept: application/json

# 响应
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": "123",
  "name": "Alice",
  "email": "alice@example.com"
}
```

**POST - 创建资源：**

```http
POST /api/v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Bob",
  "email": "bob@example.com"
}

# 响应
HTTP/1.1 201 Created
Location: /api/v1/users/456
Content-Type: application/json

{
  "id": "456",
  "name": "Bob",
  "email": "bob@example.com",
  "createdAt": "2026-04-19T10:00:00Z"
}
```

**PUT - 完全替换：**

```http
PUT /api/v1/users/456 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "name": "Robert",
  "email": "robert@example.com"
}

# 响应
HTTP/1.1 200 OK

{
  "id": "456",
  "name": "Robert",
  "email": "robert@example.com"
}
```

**PATCH - 部分更新：**

```http
PATCH /api/v1/users/456 HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "email": "new-email@example.com"
}

# 响应
HTTP/1.1 200 OK

{
  "id": "456",
  "name": "Robert",
  "email": "new-email@example.com"
}
```

**DELETE - 删除资源：**

```http
DELETE /api/v1/users/456 HTTP/1.1
Host: api.example.com

# 响应
HTTP/1.1 204 No Content
```

---

## HTTP 状态码速查表

### 1xx - 信息性

| 状态码 | 名称 | 说明 |
|--------|------|------|
| 100 | Continue | 继续发送请求 |
| 101 | Switching Protocols | 协议切换（WebSocket） |
| 102 | Processing | 处理中（WebDAV） |

### 2xx - 成功

| 状态码 | 名称 | 说明 |
|--------|------|------|
| 200 | OK | 请求成功，返回资源 |
| 201 | Created | 资源创建成功 |
| 202 | Accepted | 请求已接受，异步处理 |
| 204 | No Content | 请求成功，无返回内容 |
| 206 | Partial Content | 部分内容（分页） |

### 3xx - 重定向

| 状态码 | 名称 | 说明 |
|--------|------|------|
| 301 | Moved Permanently | 永久重定向 |
| 302 | Found | 临时重定向 |
| 303 | See Other | POST 重定向到 GET |
| 304 | Not Modified | 资源未修改（缓存） |
| 307 | Temporary Redirect | 临时重定向（保持方法） |
| 308 | Permanent Redirect | 永久重定向（保持方法） |

### 4xx - 客户端错误

| 状态码 | 名称 | 说明 |
|--------|------|------|
| 400 | Bad Request | 请求格式错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 405 | Method Not Allowed | HTTP 方法不支持 |
| 409 | Conflict | 资源冲突 |
| 410 | Gone | 资源已删除 |
| 415 | Unsupported Media Type | 不支持的内容类型 |
| 422 | Unprocessable Entity | 验证错误 |
| 429 | Too Many Requests | 请求过多 |

### 5xx - 服务器错误

| 状态码 | 名称 | 说明 |
|--------|------|------|
| 500 | Internal Server Error | 服务器内部错误 |
| 501 | Not Implemented | 功能未实现 |
| 502 | Bad Gateway | 网关错误 |
| 503 | Service Unavailable | 服务不可用 |
| 504 | Gateway Timeout | 网关超时 |

---

## 版本控制策略

### URL 版本（最常用）

```http
GET /api/v1/users/123
GET /api/v2/users/123
```

**优点：**
- 简单直观
- 便于测试和切换
- 缓存友好

**缺点：**
- URL 膨胀
- 跨版本代码重复

### Header 版本

```http
GET /api/users/123 HTTP/1.1
Accept: application/vnd.example.v2+json

# 或
GET /api/users/123 HTTP/1.1
API-Version: 2024-01-01
```

**优点：**
- URL 保持整洁
- 细粒度版本控制

**缺点：**
- 不直观
- 测试困难
- 缓存问题

### 推荐实践

```javascript
// URL 版本控制示例
const API_BASE = process.env.API_URL || 'https://api.example.com';
const API_VERSION = 'v1';

export const usersApi = {
  list: (params) =>
    fetch(`${API_BASE}/api/${API_VERSION}/users?${new URLSearchParams(params)}`),

  get: (id) =>
    fetch(`${API_BASE}/api/${API_VERSION}/users/${id}`),

  create: (data) =>
    fetch(`${API_BASE}/api/${API_VERSION}/users`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    })
};
```

---

## 认证与授权

### 常见认证方案

| 方案 | 说明 | 适用场景 |
|------|------|---------|
| **API Key** | 静态密钥 | 服务间通信 |
| **Basic Auth** | 用户名密码 | 简单场景 |
| **Bearer Token** | JWT/OAuth2 Token | Web/移动应用 |
| **OAuth 2.0** | 授权框架 | 第三方登录 |
| **HMAC** | 签名认证 | 高安全性 |

### API Key

```http
GET /api/users HTTP/1.1
Host: api.example.com
X-API-Key: your-api-key-here

# 或
GET /api/users?api_key=your-api-key-here
```

**适用场景：** 服务间通信，不面向终端用户

### Bearer Token / JWT

```http
GET /api/users HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# JWT 结构
# eyJhbGciOiJIUzI1NiJ9.{header}.{payload}.{signature}
```

**JWT Payload 示例：**

```json
{
  "sub": "user123",
  "name": "Alice",
  "role": "admin",
  "iat": 1713520800,
  "exp": 1713524400
}
```

### OAuth 2.0 流程

```
┌──────────┐                    ┌──────────┐                    ┌──────────┐
│   User   │                    │   App    │                    │   Auth   │
│  Browser │                    │  Server  │                    │  Server  │
└────┬─────┘                    └────┬─────┘                    └────┬─────┘
     │                                │                                │
     │  1. 访问授权页面               │                                │
     │ ──────────────────────────────►│                                │
     │                                │                                │
     │  2. 重定向到授权服务器         │                                │
     │ ◄──────────────────────────────│                                │
     │                                │                                │
     │  3. 用户授权                   │                                │
     │ ───────────────────────────────────────────────────────────────►│
     │                                │                                │
     │  4. 重定向回 App + Code        │                                │
     │ ◄───────────────────────────────────────────────────────────────│
     │                                │                                │
     │                                │  5. 用 Code 换 Token            │
     │                                │ ──────────────────────────────►│
     │                                │                                │
     │                                │  6. 返回 Access Token           │
     │                                │ ◄──────────────────────────────│
     │                                │                                │
```

---

## OpenAPI 规范

### OpenAPI vs Swagger vs AsyncAPI 对比

| 规范 | 说明 | 用途 |
|------|------|------|
| **OpenAPI 3.1** | REST API 规范 | HTTP/REST API |
| **Swagger 2.0** | OpenAPI 2.0 旧名 | HTTP/REST API |
| **AsyncAPI 3.0** | 异步 API 规范 | WebSocket/MQTT |

### OpenAPI 3.1 示例

```yaml
openapi: 3.1.0
info:
  title: User Management API
  version: 1.0.0
  description: 用户管理 REST API

servers:
  - url: https://api.example.com/v1
    description: Production

paths:
  /users:
    get:
      summary: 获取用户列表
      tags:
        - Users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: 创建用户
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUser'
      responses:
        '201':
          description: 创建成功
          headers:
            Location:
              schema:
                type: string
              description: 新资源 URL
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

  /users/{id}:
    get:
      summary: 获取用户详情
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

    delete:
      summary: 删除用户
      tags:
        - Users
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
      responses:
        '204':
          description: 删除成功

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
          minLength: 1
          maxLength: 100
        email:
          type: string
          format: email
        createdAt:
          type: string
          format: date-time
        updatedAt:
          type: string
          format: date-time

    CreateUser:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
        email:
          type: string
          format: email
        password:
          type: string
          format: password
          minLength: 8

    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        meta:
          type: object
          properties:
            total:
              type: integer
            page:
              type: integer
            limit:
              type: integer

  responses:
    Unauthorized:
      description: 未认证
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: 资源不存在
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

---

## 错误响应格式

### 标准错误格式

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "请求验证失败",
    "details": [
      {
        "field": "email",
        "message": "邮箱格式不正确",
        "code": "INVALID_FORMAT"
      },
      {
        "field": "password",
        "message": "密码长度不足 8 位",
        "code": "TOO_SHORT"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2026-04-19T10:00:00Z"
  }
}
```

### 错误代码体系

| HTTP 状态码 | 错误码 | 说明 |
|-------------|--------|------|
| 400 | VALIDATION_ERROR | 请求验证失败 |
| 400 | BAD_REQUEST | 请求格式错误 |
| 401 | UNAUTHORIZED | 未认证 |
| 401 | TOKEN_EXPIRED | Token 过期 |
| 401 | TOKEN_INVALID | Token 无效 |
| 403 | FORBIDDEN | 无权限 |
| 404 | NOT_FOUND | 资源不存在 |
| 409 | CONFLICT | 资源冲突 |
| 422 | UNPROCESSABLE | 无法处理 |
| 429 | RATE_LIMITED | 请求过多 |
| 500 | INTERNAL_ERROR | 服务器错误 |
| 503 | SERVICE_UNAVAILABLE | 服务不可用 |

---

## REST vs GraphQL vs tRPC

### 对比表

| 特性 | REST | GraphQL | tRPC |
|------|------|---------|------|
| **类型安全** | 需额外工具 | 内置 | 原生 |
| **请求灵活性** | 固定响应 | 按需获取 | 端到端类型 |
| **网络效率** | 多次请求 | 单次请求 | 单次请求 |
| **Overfetching** | 常见 | 避免 | 避免 |
| **Underfetching** | 常见 | 避免 | 避免 |
| **缓存** | HTTP 缓存 | 手动管理 | 自动 |
| **学习曲线** | 低 | 中 | 低 |
| **生态系统** | 成熟 | 成熟 | 新兴 |
| **服务端负载** | 低 | 中等 | 低 |
| **文档** | OpenAPI | Schema | TypeScript |

### 选型建议

| 场景 | 推荐 |
|------|------|
| **简单 CRUD** | REST |
| **移动应用** | REST / tRPC |
| **复杂数据关系** | GraphQL |
| **TypeScript 团队** | tRPC |
| **公开 API** | REST (OpenAPI) |
| **微服务** | REST / gRPC |
| **实时数据** | GraphQL Subscriptions |

---

## AI 应用实战

### AI API 代理

```javascript
// REST API 风格的 AI 服务封装
class AIService {
  constructor(baseUrl, apiKey) {
    this.baseUrl = baseUrl;
    this.apiKey = apiKey;
  }

  async request(endpoint, options = {}) {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${this.apiKey}`,
        ...options.headers
      }
    });

    if (!response.ok) {
      const error = await response.json().catch(() => ({}));
      throw new AIAPIError(
        error.message || `HTTP ${response.status}`,
        response.status,
        error.code
      );
    }

    return response.json();
  }

  async chat(messages, options = {}) {
    return this.request('/chat/completions', {
      method: 'POST',
      body: JSON.stringify({
        model: options.model || 'gpt-4o',
        messages,
        ...options
      })
    });
  }

  async createEmbedding(text, model = 'text-embedding-3') {
    return this.request('/embeddings', {
      method: 'POST',
      body: JSON.stringify({ input: text, model })
    });
  }
}

class AIAPIError extends Error {
  constructor(message, status, code) {
    super(message);
    this.name = 'AIAPIError';
    this.status = status;
    this.code = code;
  }
}

// 使用示例
const ai = new AIService('https://api.openai.com/v1', process.env.OPENAI_API_KEY);

const response = await ai.chat([
  { role: 'system', content: '你是一个有帮助的助手。' },
  { role: 'user', content: '解释什么是 RAG' }
], {
  temperature: 0.7,
  max_tokens: 500
});
```

### 流式响应处理

```javascript
// REST API 流式响应处理
async function* streamChat(messages, apiKey) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`
    },
    body: JSON.stringify({
      model: 'gpt-4o',
      messages,
      stream: true
    })
  });

  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`);
  }

  const reader = response.body.getReader();
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;

    const chunk = decoder.decode(value);
    const lines = chunk.split('\n').filter(line => line.trim());

    for (const line of lines) {
      if (line.startsWith('data: ')) {
        const data = line.slice(6);
        if (data === '[DONE]') return;

        const parsed = JSON.parse(data);
        const content = parsed.choices?.[0]?.delta?.content;
        if (content) {
          yield content;
        }
      }
    }
  }
}

// 使用
for await (const token of streamChat(messages, apiKey)) {
  process.stdout.write(token);
}
```

---

### REST API 性能优化

#### 缓存策略

REST API 的缓存是提升性能的关键：

```http
# 强缓存
Cache-Control: public, max-age=3600, immutable

# 协商缓存
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
Last-Modified: Wed, 21 Oct 2026 07:28:00 GMT

# Cache-Control 指令详解
# - public: 响应可被任何缓存存储
# - private: 响应只能被浏览器缓存
# - no-cache: 每次使用前必须与服务器验证
# - no-store: 禁止缓存
# - max-age: 缓存有效期（秒）
# - s-maxage: 代理缓存有效期
# - must-revalidate: 过期后必须验证
# - immutable: 响应永不改变
```

```javascript
// Next.js 中的缓存策略
export async function GET(request: Request) {
  const data = await fetchData();
  
  // 使用 Next.js 的 cache 函数
  const cachedData = await unstable_cache(
    () => fetchData(),
    ['users'],
    {
      revalidate: 3600, // 一小时
      tags: ['users']
    }
  )();
  
  return Response.json(cachedData);
}
```

#### 分页策略

```javascript
// 游标分页（推荐大表使用）
class CursorPagination {
  static encodeCursor(item) {
    // 基于时间戳和 ID 的复合游标
    return Buffer.from(
      JSON.stringify({
        createdAt: item.createdAt.getTime(),
        id: item.id
      })
    ).toString('base64');
  }
  
  static decodeCursor(cursor) {
    return JSON.parse(
      Buffer.from(cursor, 'base64').toString()
    );
  }
  
  static async paginate(query, cursor, limit = 20) {
    if (cursor) {
      const { createdAt, id } = this.decodeCursor(cursor);
      query = query.where({
        OR: [
          { createdAt: { lt: new Date(createdAt) } },
          {
            createdAt: { equals: new Date(createdAt) },
            id: { lt: id }
          }
        ]
      });
    }
    
    const items = await query.limit(limit + 1);
    const hasNextPage = items.length > limit;
    const data = hasNextPage ? items.slice(0, -1) : items;
    
    return {
      data,
      nextCursor: hasNextPage 
        ? this.encodeCursor(data[data.length - 1])
        : null
    };
  }
}
```

#### 速率限制实现

```javascript
// 滑动窗口限流
class RateLimiter {
  constructor(redis) {
    this.redis = redis;
    this.windowMs = 60 * 1000; // 1 分钟窗口
    this.maxRequests = 100;
  }
  
  async isAllowed(identifier) {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // 使用 Redis 有序集合
    const key = `ratelimit:${identifier}`;
    
    // 移除窗口外的请求
    await this.redis.zremrangebyscore(key, 0, windowStart);
    
    // 统计当前请求数
    const count = await this.redis.zcard(key);
    
    if (count >= this.maxRequests) {
      return {
        allowed: false,
        remaining: 0,
        resetAt: now + this.windowMs
      };
    }
    
    // 添加当前请求
    await this.redis.zadd(key, now, `${now}-${Math.random()}`);
    await this.redis.expire(key, Math.ceil(this.windowMs / 1000));
    
    return {
      allowed: true,
      remaining: this.maxRequests - count - 1,
      resetAt: now + this.windowMs
    };
  }
}

// 中间件使用
async function rateLimitMiddleware(request, response, next) {
  const identifier = request.ip || request.headers['x-forwarded-for'];
  const result = await rateLimiter.isAllowed(identifier);
  
  response.setHeader('X-RateLimit-Limit', rateLimiter.maxRequests);
  response.setHeader('X-RateLimit-Remaining', result.remaining);
  response.setHeader('X-RateLimit-Reset', result.resetAt);
  
  if (!result.allowed) {
    return response.status(429).json({
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: '请求过于频繁，请稍后再试',
        retryAfter: Math.ceil((result.resetAt - Date.now()) / 1000)
      }
    });
  }
  
  next();
}
```

---

## REST API 安全最佳实践

### 常见安全威胁与防护

| 威胁类型 | 说明 | 防护措施 |
|---------|------|---------|
| **SQL 注入** | 恶意 SQL 代码注入 | 参数化查询、输入验证 |
| **XSS** | 跨站脚本攻击 | 输出编码、Content-Type 设置 |
| **CSRF** | 跨站请求伪造 | CSRF Token、SameSite Cookie |
| **重放攻击** | 请求被重复发送 | 时间戳、Nonce、一次性 Token |
| **中间人攻击** | 请求被拦截篡改 | HTTPS、TLS 1.3 |

### CORS 配置

```javascript
// 生产环境 CORS 配置
const corsOptions = {
  origin: function (origin, callback) {
    // 生产环境：只允许指定域名
    const allowedOrigins = [
      'https://example.com',
      'https://app.example.com'
    ];
    
    // 允许没有 origin 的请求（如 Postman）
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: [
    'Content-Type', 
    'Authorization', 
    'X-Request-ID'
  ],
  exposedHeaders: ['X-Total-Count', 'X-Page-Count'],
  credentials: true,
  maxAge: 86400 // 预检请求缓存 24 小时
};
```

### 请求签名

```javascript
// HMAC 请求签名
const crypto = require('crypto');

class RequestSigner {
  constructor(secretKey) {
    this.secretKey = secretKey;
  }
  
  sign(request) {
    const timestamp = Date.now();
    const nonce = crypto.randomBytes(16).toString('hex');
    
    // 构建签名字符串
    const stringToSign = [
      request.method.toUpperCase(),
      request.path,
      timestamp,
      nonce,
      request.body ? JSON.stringify(request.body) : ''
    ].join('\n');
    
    // 计算 HMAC-SHA256 签名
    const signature = crypto
      .createHmac('sha256', this.secretKey)
      .update(stringToSign)
      .digest('hex');
    
    return {
      'X-Timestamp': timestamp,
      'X-Nonce': nonce,
      'X-Signature': signature
    };
  }
  
  verify(request, headers) {
    const { 'X-Timestamp': timestamp, 'X-Nonce': nonce, 'X-Signature': signature } = headers;
    
    // 检查时间戳（5 分钟内有效）
    if (Date.now() - parseInt(timestamp) > 5 * 60 * 1000) {
      return false;
    }
    
    // 重新计算签名
    const expectedSignature = this.sign({
      ...request,
      headers: { ...request.headers, 'X-Timestamp': timestamp, 'X-Nonce': nonce }
    })['X-Signature'];
    
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(expectedSignature)
    );
  }
}
```

### 敏感数据处理

```javascript
// 敏感字段过滤
const sensitiveFields = ['password', 'token', 'secret', 'apiKey', 'creditCard'];

function sanitize(obj) {
  const sanitized = { ...obj };
  
  for (const field of sensitiveFields) {
    if (field in sanitized) {
      sanitized[field] = '[REDACTED]';
    }
  }
  
  return sanitized;
}

// 日志脱敏
function logRequest(req, res, next) {
  const startTime = Date.now();
  
  res.on('finish', () => {
    const logEntry = {
      method: req.method,
      path: req.path,
      query: req.query,
      status: res.statusCode,
      duration: Date.now() - startTime,
      userId: req.user?.id,
      ip: req.ip,
      userAgent: req.get('user-agent'),
      requestId: req.headers['x-request-id']
    };
    
    // 敏感数据脱敏后记录日志
    const sanitizedEntry = sanitize(logEntry);
    console.log(JSON.stringify(sanitizedEntry));
  });
  
  next();
}
```

---

## REST API 文档与协作

### Swagger/OpenAPI 最佳实践

```yaml
# 完整的 OpenAPI 文档示例
openapi: 3.1.0
info:
  title: E-Commerce API
  version: 2.0.0
  description: |
    电商平台 REST API
    
    ## 认证
    本 API 使用 JWT Bearer Token 进行认证。
    
    ## 限流
    - 免费版：100 请求/分钟
    - 专业版：1000 请求/分钟
    - 企业版：无限制

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://staging-api.example.com/v1
    description: Staging

tags:
  - name: Authentication
    description: 用户认证相关接口
  - name: Products
    description: 商品管理接口
  - name: Orders
    description: 订单管理接口

paths:
  /auth/login:
    post:
      tags:
        - Authentication
      summary: 用户登录
      operationId: loginUser
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/LoginRequest'
            example:
              email: user@example.com
              password: securePassword123
      responses:
        '200':
          description: 登录成功
          headers:
            Set-Cookie:
              schema:
                type: string
              description: JWT Refresh Token
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/LoginResponse'
        '401':
          $ref: '#/components/responses/Unauthorized'

  /products:
    get:
      tags:
        - Products
      summary: 获取商品列表
      operationId: listProducts
      parameters:
        - name: category
          in: query
          schema:
            type: string
          description: 商品分类
        - name: minPrice
          in: query
          schema:
            type: number
            format: float
        - name: maxPrice
          in: query
          schema:
            type: number
            format: float
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
      responses:
        '200':
          description: 成功
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ProductList'

components:
  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1
    LimitParam:
      name: limit
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20

  schemas:
    LoginRequest:
      type: object
      required:
        - email
        - password
      properties:
        email:
          type: string
          format: email
        password:
          type: string
          format: password
          minLength: 8

    LoginResponse:
      type: object
      properties:
        accessToken:
          type: string
          description: JWT Access Token
        expiresIn:
          type: integer
          description: Token 有效期（秒）
        user:
          $ref: '#/components/schemas/User'

    ProductList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Product'
        pagination:
          $ref: '#/components/schemas/Pagination'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Product:
      type: object
      properties:
        id:
          type: string
          format: uuid
        name:
          type: string
        price:
          type: number
          format: float
        category:
          type: string
        images:
          type: array
          items:
            type: string
            format: uri
        stock:
          type: integer
        createdAt:
          type: string
          format: date-time

  responses:
    Unauthorized:
      description: 未认证
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    NotFound:
      description: 资源不存在
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

    Error:
      type: object
      properties:
        error:
          type: object
          properties:
            code:
              type: string
            message:
              type: string
            details:
              type: array
              items:
                type: object

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: JWT Access Token

security:
  - BearerAuth: []
```

### API 版本迁移策略

```javascript
// 渐进式版本迁移
class VersionMigration {
  constructor() {
    this.migrations = {
      'v1->v2': {
        'GET /users': this.migrateGetUsersV1ToV2,
        'POST /users': this.migratePostUsersV1ToV2
      }
    };
  }
  
  async migrateGetUsersV1ToV2(request, response) {
    // 添加默认字段
    response.data = response.data.map(user => ({
      ...user,
      displayName: user.name, // v2 新增字段
      preferences: user.preferences || {} // v2 新增字段
    }));
    return response;
  }
  
  async migratePostUsersV1ToV2(request) {
    // 转换请求格式
    if (request.phoneNumber && !request.phone) {
      request.phone = request.phoneNumber;
    }
    return request;
  }
}

// 版本协商
async function versionNegotiation(request) {
  // 1. 检查 URL 版本
  const urlVersion = request.path.match(/\/v(\d+)/)?.[1];
  
  // 2. 检查 Accept-Version header
  const headerVersion = request.headers['api-version'];
  
  // 3. 检查 Default header
  const defaultVersion = '1';
  
  const version = urlVersion || headerVersion || defaultVersion;
  
  if (parseInt(version) > 2) {
    throw new Error('Unsupported API version');
  }
  
  return `v${version}`;
}
```

---

> [!SUCCESS]
> REST API 作为 Web 服务的标准接口规范，在 2026 年仍然占据主导地位。遵循 REST 约束的 API 设计能够提供清晰、一致、可缓存的接口。通过 OpenAPI 规范文档化 API，配合完善的错误处理和认证机制，可以构建出专业、可靠的 API 服务。

---

## REST API 高级模式

### 分页策略深度对比

REST API 中有三种主要分页策略，适用于不同场景：

```javascript
// 1. 偏移量分页（Offset Pagination）
// 最简单，但大数据量时性能下降
class OffsetPagination {
  static async getUsers(page = 1, limit = 20) {
    const offset = (page - 1) * limit;
    
    const [users, total] = await Promise.all([
      db.user.findMany({
        take: limit,
        skip: offset,
        orderBy: { createdAt: 'desc' }
      }),
      db.user.count()
    ]);
    
    return {
      data: users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasNextPage: offset + limit < total,
        hasPrevPage: page > 1
      }
    };
  }
}

// 2. 游标分页（Cursor/Keyset Pagination）
// 适合大数据量，性能稳定
class CursorPagination {
  static encodeCursor(data) {
    return Buffer.from(JSON.stringify({
      id: data.id,
      createdAt: data.createdAt.getTime()
    })).toString('base64url');
  }
  
  static decodeCursor(cursor) {
    return JSON.parse(
      Buffer.from(cursor, 'base64url').toString()
    );
  }
  
  static async getUsers({ cursor, limit = 20, direction = 'after' }) {
    let query = db.user.findMany({
      take: limit + 1,
      orderBy: [
        { createdAt: 'desc' },
        { id: 'desc' }
      ]
    });
    
    if (cursor) {
      const { id, createdAt } = this.decodeCursor(cursor);
      query = query.where({
        OR: [
          { createdAt: { lt: new Date(createdAt) } },
          {
            createdAt: { equals: new Date(createdAt) },
            id: { lt: id }
          }
        ]
      });
    }
    
    const items = await query;
    const hasNextPage = items.length > limit;
    const data = hasNextPage ? items.slice(0, -1) : items;
    
    return {
      data,
      pageInfo: {
        hasNextPage,
        hasPreviousPage: !!cursor,
        startCursor: data[0] ? this.encodeCursor(data[0]) : null,
        endCursor: data[data.length - 1] ? this.encodeCursor(data[data.length - 1]) : null
      }
    };
  }
}

// 3. 搜索分页（Search/Seek Pagination）
// 适合全文搜索场景
class SeekPagination {
  static async searchPosts({ search, limit = 20, lastId }) {
    const posts = await db.$queryRaw`
      SELECT * FROM posts 
      WHERE 
        to_tsvector('english', title || ' ' || content) 
        @@ plainto_tsquery('english', ${search})
        ${lastId ? sql`AND id < ${lastId}` : sql``}
      ORDER BY id DESC 
      LIMIT ${limit + 1}
    `;
    
    const hasNextPage = posts.length > limit;
    const data = hasNextPage ? posts.slice(0, -1) : posts;
    
    return {
      data,
      nextCursor: hasNextPage ? data[data.length - 1].id : null
    };
  }
}
```

### 过滤与查询最佳实践

```javascript
// 高级过滤查询构建器
class QueryBuilder {
  constructor(model) {
    this.model = model;
    this.filters = [];
    this.sortFields = [];
    this.page = 1;
    this.pageSize = 20;
    this.includes = [];
  }
  
  // 精确匹配
  where(field, operator, value) {
    const operators = {
      '=': (f, v) => ({ [f]: v }),
      '!=': (f, v) => ({ [f]: { not: v } }),
      '>': (f, v) => ({ [f]: { gt: v } }),
      '>=': (f, v) => ({ [f]: { gte: v } }),
      '<': (f, v) => ({ [f]: { lt: v } }),
      '<=': (f, v) => ({ [f]: { lte: v } }),
      'in': (f, v) => ({ [f]: { in: Array.isArray(v) ? v : [v] } }),
      'like': (f, v) => ({ [f]: { contains: v } }),
      'ilike': (f, v) => ({ [f]: { mode: 'insensitive', contains: v } }),
      'null': (f) => ({ [f]: null }),
      'notNull': (f) => ({ [f]: { not: null } })
    };
    
    if (typeof operator === 'object' && operator !== null) {
      this.filters.push(operator);
    } else {
      const op = operators[operator];
      if (op) this.filters.push(op(field, value));
    }
    return this;
  }
  
  // 排序
  orderBy(field, direction = 'asc') {
    this.sortFields.push({ field, direction });
    return this;
  }
  
  // 分页
  paginate(page, pageSize = 20) {
    this.page = page;
    this.pageSize = pageSize;
    return this;
  }
  
  // 关联
  include(relation) {
    this.includes.push(relation);
    return this;
  }
  
  // 执行查询
  async execute() {
    const where = this.filters.length > 1 
      ? { AND: this.filters } 
      : this.filters[0] || {};
    
    const orderBy = this.sortFields.length > 0
      ? this.sortFields.map(s => ({ [s.field]: s.direction }))
      : { createdAt: 'desc' };
    
    const [data, total] = await Promise.all([
      this.model.findMany({
        where,
        orderBy,
        skip: (this.page - 1) * this.pageSize,
        take: this.pageSize,
        include: this.includes.length > 0 
          ? this.includes.reduce((acc, r) => ({ ...acc, [r]: true }), {})
          : undefined
      }),
      this.model.count({ where })
    ]);
    
    return {
      data,
      pagination: {
        page: this.page,
        pageSize: this.pageSize,
        total,
        totalPages: Math.ceil(total / this.pageSize)
      }
    };
  }
}

// 使用示例
const result = await new QueryBuilder(db.user)
  .where('status', '=', 'active')
  .where('role', 'in', ['admin', 'moderator'])
  .where('createdAt', '>=', lastMonth)
  .orderBy('createdAt', 'desc')
  .include('posts')
  .paginate(1, 20)
  .execute();
```

---

## REST API 可观测性与监控

### 结构化日志

```javascript
// 统一日志格式
class StructuredLogger {
  constructor(service) {
    this.service = service;
    this.context = {};
  }
  
  // 添加上下文
  withContext(context) {
    this.context = { ...this.context, ...context };
    return this;
  }
  
  // 格式化日志
  format(level, message, meta = {}) {
    return {
      timestamp: new Date().toISOString(),
      level,
      service: this.service,
      message,
      ...this.context,
      ...meta
    };
  }
  
  // 记录方法
  info(message, meta) {
    console.log(JSON.stringify(this.format('INFO', message, meta)));
  }
  
  warn(message, meta) {
    console.warn(JSON.stringify(this.format('WARN', message, meta)));
  }
  
  error(message, meta) {
    console.error(JSON.stringify(this.format('ERROR', message, meta)));
  }
  
  debug(message, meta) {
    if (process.env.NODE_ENV === 'development') {
      console.log(JSON.stringify(this.format('DEBUG', message, meta)));
    }
  }
  
  // HTTP 请求日志中间件
  httpMiddleware() {
    return async (ctx, next) => {
      const start = Date.now();
      const requestId = crypto.randomUUID();
      
      this.info('Incoming request', {
        requestId,
        method: ctx.method,
        path: ctx.path,
        query: ctx.query,
        userAgent: ctx.headers['user-agent'],
        ip: ctx.ip
      });
      
      try {
        await next();
        
        this.info('Request completed', {
          requestId,
          status: ctx.status,
          duration: Date.now() - start
        });
      } catch (error) {
        this.error('Request failed', {
          requestId,
          error: error.message,
          stack: error.stack,
          duration: Date.now() - start
        });
        throw error;
      }
    };
  }
}

const logger = new StructuredLogger('api-gateway');
```

### 性能指标收集

```javascript
// 性能监控
class MetricsCollector {
  constructor() {
    this.metrics = new Map();
  }
  
  // 记录指标
  recordMetric(name, value, tags = {}) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    this.metrics.get(name).push({
      value,
      tags,
      timestamp: Date.now()
    });
  }
  
  // 请求计数
  incrementCounter(name, tags = {}) {
    this.recordMetric(name, 1, tags);
  }
  
  // 记录延迟
  recordHistogram(name, value, tags = {}) {
    this.recordMetric(name, value, tags);
  }
  
  // 生成 Prometheus 格式指标
  toPrometheusFormat() {
    const output = [];
    
    for (const [name, values] of this.metrics.entries()) {
      const tags = [...new Set(values.flatMap(v => Object.keys(v.tags)))];
      const tagCombinations = [...new Set(
        values.map(v => JSON.stringify(v.tags))
      )];
      
      for (const tagStr of tagCombinations) {
        const tagObj = JSON.parse(tagStr);
        const labelStr = tags.map(t => `${t}="${tagObj[t] || ''}"`).join(',');
        
        const relevantValues = values.filter(v => 
          JSON.stringify(v.tags) === tagStr
        );
        
        const sum = relevantValues.reduce((a, b) => a + b.value, 0);
        const count = relevantValues.length;
        
        output.push(`${name}_sum{${labelStr}} ${sum}`);
        output.push(`${name}_count{${labelStr}} ${count}`);
      }
    }
    
    return output.join('\n');
  }
  
  // 聚合统计
  getStats(name) {
    const values = this.metrics.get(name) || [];
    if (values.length === 0) return null;
    
    const nums = values.map(v => v.value).sort((a, b) => a - b);
    const sum = nums.reduce((a, b) => a + b, 0);
    
    return {
      count: nums.length,
      sum,
      avg: sum / nums.length,
      min: nums[0],
      max: nums[nums.length - 1],
      p50: nums[Math.floor(nums.length * 0.5)],
      p90: nums[Math.floor(nums.length * 0.9)],
      p95: nums[Math.floor(nums.length * 0.95)],
      p99: nums[Math.floor(nums.length * 0.99)]
    };
  }
}

const metrics = new MetricsCollector();

// Express 中间件示例
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    
    metrics.recordHistogram('http_request_duration_ms', duration, {
      method: req.method,
      path: req.route?.path || req.path,
      status: res.statusCode
    });
    
    metrics.incrementCounter('http_requests_total', {
      method: req.method,
      path: req.route?.path || req.path,
      status: res.statusCode
    });
  });
  
  next();
});
```

### 健康检查端点

```javascript
// 健康检查路由
app.get('/health', async (req, res) => {
  const checks = await Promise.allSettled([
    // 数据库连接
    async () => {
      await db.$queryRaw`SELECT 1`;
      return { name: 'database', status: 'healthy' };
    },
    
    // Redis 连接
    async () => {
      await redis.ping();
      return { name: 'redis', status: 'healthy' };
    },
    
    // 外部 API
    async () => {
      const response = await fetch('https://api.example.com/health', {
        timeout: 3000
      });
      if (!response.ok) throw new Error('External API unhealthy');
      return { name: 'external_api', status: 'healthy' };
    }
  ]);
  
  const results = checks.map((check, index) => {
    if (check.status === 'fulfilled') {
      return check.value;
    }
    return { 
      name: ['database', 'redis', 'external_api'][index],
      status: 'unhealthy',
      error: check.reason?.message 
    };
  });
  
  const allHealthy = results.every(r => r.status === 'healthy');
  
  res.status(allHealthy ? 200 : 503).json({
    status: allHealthy ? 'healthy' : 'unhealthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks: results
  });
});

// 就绪检查（用于 Kubernetes）
app.get('/ready', async (req, res) => {
  try {
    await db.$queryRaw`SELECT 1`;
    res.json({ status: 'ready' });
  } catch {
    res.status(503).json({ status: 'not ready' });
  }
});

// 存活检查
app.get('/live', (req, res) => {
  res.json({ status: 'alive' });
});
```

---

## REST API 缓存策略

### 多层缓存架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        缓存层级架构                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────┐                                              │
│   │   Browser   │  ← HTTP Cache (强缓存/协商缓存)                │
│   └──────┬──────┘                                              │
│          │ Cache-Control, ETag                                  │
│          ▼                                                       │
│   ┌─────────────┐                                              │
│   │    CDN      │  ← 边缘节点缓存                                │
│   └──────┬──────┘                                              │
│          │                                                      │
│          ▼                                                       │
│   ┌─────────────┐                                              │
│   │  API Cache  │  ← Redis/Memcached                           │
│   └──────┬──────┘                                              │
│          │                                                      │
│          ▼                                                       │
│   ┌─────────────┐                                              │
│   │  Database   │  ← Query Cache                               │
│   └─────────────┘                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### HTTP 缓存详解

```http
# Cache-Control 指令
Cache-Control: max-age=3600                    # 缓存有效期（秒）
Cache-Control: s-maxage=86400                  # 共享缓存有效期（CDN）
Cache-Control: public                          # 可被任何缓存存储
Cache-Control: private                         # 仅浏览器可缓存
Cache-Control: no-cache                        # 每次验证后可用缓存
Cache-Control: no-store                         # 完全禁止缓存
Cache-Control: must-revalidate                  # 过期后必须验证
Cache-Control: immutable                        # 响应永不改变

# 条件请求
If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
If-Modified-Since: Wed, 21 Oct 2026 07:28:00 GMT
```

```javascript
// 缓存控制中间件
class CacheMiddleware {
  constructor(redis, options = {}) {
    this.redis = redis;
    this.defaultTTL = options.defaultTTL || 300;
    this.enabled = options.enabled !== false;
  }
  
  // 缓存响应
  async cacheResponse(req, res, next) {
    if (!this.enabled || req.method !== 'GET') {
      return next();
    }
    
    const cacheKey = `cache:${req.originalUrl}`;
    
    try {
      // 尝试获取缓存
      const cached = await this.redis.get(cacheKey);
      
      if (cached) {
        const { data, headers } = JSON.parse(cached);
        
        // 检查 ETag
        const etag = headers['ETag'];
        if (etag && req.headers['if-none-match'] === etag) {
          res.status(304).end();
          return;
        }
        
        // 返回缓存
        for (const [key, value] of Object.entries(headers)) {
          res.setHeader(key, value);
        }
        res.setHeader('X-Cache', 'HIT');
        return res.json(data);
      }
      
      // 拦截响应
      const originalJson = res.json.bind(res);
      const originalSet = res.setHeader.bind(res);
      const headers = {};
      
      res.setHeader = (key, value) => {
        headers[key] = value;
        return originalSet(key, value);
      };
      
      res.json = async (data) => {
        const etag = `"${crypto.createHash('md5').update(JSON.stringify(data)).digest('hex')}"`;
        headers['ETag'] = etag;
        res.setHeader('ETag', etag);
        
        // 存储缓存
        await this.redis.setex(cacheKey, this.defaultTTL, JSON.stringify({ data, headers }));
        res.setHeader('X-Cache', 'MISS');
        
        return originalJson(data);
      };
      
      next();
    } catch (error) {
      console.error('Cache error:', error);
      next();
    }
  }
  
  // 主动失效缓存
  async invalidateCache(pattern) {
    const keys = await this.redis.keys(`cache:${pattern}`);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}

// 使用示例
const cacheMiddleware = new CacheMiddleware(redis, { defaultTTL: 3600 });

// 应用缓存中间件
app.get('/api/users', cacheMiddleware.cacheResponse.bind(cacheMiddleware), async (req, res) => {
  const users = await db.user.findMany();
  res.json(users);
});

// 数据变更时清除缓存
app.post('/api/users', async (req, res) => {
  const user = await db.user.create({ data: req.body });
  await cacheMiddleware.invalidateCache('/api/users*');
  res.status(201).json(user);
});
```

---

## REST API 版本控制高级策略

### 演进式 API 设计

```javascript
// 渐进式字段弃用
class APIVersionManager {
  constructor() {
    this.deprecatedFields = new Map();
  }
  
  // 标记字段弃用
  deprecate(version, field, message, sunsetDate) {
    this.deprecatedFields.set(`${version}:${field}`, {
      message,
      sunsetDate: new Date(sunsetDate),
      replacement: null
    });
  }
  
  // 替换字段
  replace(version, oldField, newField, message) {
    this.deprecatedFields.set(`${version}:${oldField}`, {
      message,
      replacement: newField
    });
  }
  
  // 获取响应中的弃用警告
  getDeprecationHeaders(version, fields) {
    const headers = {};
    
    for (const field of fields) {
      const key = `${version}:${field}`;
      const deprecation = this.deprecatedFields.get(key);
      
      if (deprecation) {
        headers['Deprecation'] = 'true';
        headers['Sunset'] = deprecation.sunsetDate.toUTCString();
        headers['Link'] = `<https://api.example.com/docs/deprecations>; rel="deprecation"`;
      }
    }
    
    return headers;
  }
  
  // 处理旧版本请求
  transformRequest(version, data) {
    const transformations = this.getTransformations(version);
    return this.applyTransformations(data, transformations);
  }
  
  // 处理响应
  transformResponse(version, data, fields) {
    const headers = this.getDeprecationHeaders(version, fields);
    return { data, headers };
  }
}

const apiVersion = new APIVersionManager();

// 标记弃用
apiVersion.deprecate('v1', 'user_name', 'Use full_name instead', '2026-12-31');
apiVersion.replace('v1', 'old_field', 'new_field', 'Field renamed');
```

### 向后兼容的 Schema 变更

```javascript
// Schema 演进规则
const SchemaEvolutionRules = {
  // ✅ 可添加：新增字段（带默认值）
  ALLOWED: [
    'add_optional_field',
    'add_field_with_default',
    'add_enum_value',
    'add_endpoint',
    'relax_constraint'
  ],
  
  // ❌ 禁止：破坏性变更
  FORBIDDEN: [
    'remove_field',
    'rename_field',
    'change_field_type',
    'add_required_field',
    'remove_enum_value',
    'tighten_constraint'
  ],
  
  // ⚠️ 需要版本化
  VERSION_REQUIRED: [
    'change_field_meaning',
    'change_response_format',
    'change_auth_requirements'
  ]
};

// 字段变更策略
class SchemaChange {
  static addField(schema, fieldName, fieldDef) {
    return {
      ...schema,
      properties: {
        ...schema.properties,
        [fieldName]: {
          ...fieldDef,
          description: `(Added in ${new Date().getFullYear()}) ${fieldDef.description || ''}`
        }
      }
    };
  }
  
  static deprecateField(schema, fieldName, replacement) {
    const field = schema.properties[fieldName];
    return {
      ...schema,
      properties: {
        ...schema.properties,
        [fieldName]: {
          ...field,
          description: `DEPRECATED: Use '${replacement}' instead.`,
          deprecated: true
        }
      }
    };
  }
  
  static transformResponse(schema, data, version) {
    const deprecations = {
      'v1': { user_name: 'full_name' },
      'v2': {}
    };
    
    const transforms = deprecations[version] || {};
    const result = { ...data };
    
    for (const [oldField, newField] of Object.entries(transforms)) {
      if (oldField in result) {
        result[newField] = result[oldField];
        delete result[oldField];
      }
    }
    
    return result;
  }
}
```

---

## REST API 安全加固

### 速率限制详细实现

```javascript
// 多层级速率限制
class MultiTierRateLimiter {
  constructor(redis) {
    this.redis = redis;
    this.tiers = {
      // 免费用户
      free: {
        window: 60,           // 60秒窗口
        max: 100,             // 最多100次
        blockDuration: 300    // 封禁5分钟
      },
      // 付费用户
      premium: {
        window: 60,
        max: 1000,
        blockDuration: 60
      },
      // 企业用户
      enterprise: {
        window: 60,
        max: 10000,
        blockDuration: 30
      }
    };
  }
  
  // 滑动窗口算法
  async checkRateLimit(identifier, tier = 'free') {
    const config = this.tiers[tier];
    const now = Date.now();
    const windowStart = now - config.window * 1000;
    
    const key = `ratelimit:${tier}:${identifier}`;
    
    // 事务操作
    const multi = this.redis.multi();
    
    // 移除窗口外的请求
    multi.zremrangebyscore(key, 0, windowStart);
    
    // 添加当前请求
    multi.zadd(key, now, `${now}-${Math.random()}`);
    
    // 获取当前计数
    multi.zcard(key);
    
    // 设置过期
    multi.expire(key, config.window);
    
    const results = await multi.exec();
    const count = results[2][1];
    
    // 检查是否超过限制
    if (count > config.max) {
      // 检查是否已被封禁
      const blocked = await this.redis.get(`blocked:${identifier}`);
      
      if (blocked) {
        return {
          allowed: false,
          reason: 'blocked',
          retryAfter: parseInt(blocked) - Date.now() / 1000
        };
      }
      
      // 封禁用户
      await this.redis.setex(
        `blocked:${identifier}`,
        config.blockDuration,
        Date.now() + config.blockDuration * 1000
      );
      
      return {
        allowed: false,
        reason: 'rate_limit_exceeded',
        retryAfter: config.blockDuration
      };
    }
    
    return {
      allowed: true,
      remaining: config.max - count,
      limit: config.max,
      resetAt: now + config.window * 1000
    };
  }
  
  // 为不同端点设置不同限制
  async checkEndpointLimit(req, userId) {
    const endpointLimits = {
      '/api/auth/login': { window: 300, max: 5 },     // 5分钟内5次登录
      '/api/search': { window: 60, max: 30 },        // 1分钟30次搜索
      '/api/export': { window: 3600, max: 10 }       // 1小时10次导出
    };
    
    const limit = endpointLimits[req.path];
    if (!limit) return { allowed: true };
    
    return this.checkRateLimit(
      `${req.ip}:${req.path}`,
      'endpoint',
      limit.window,
      limit.max,
      600
    );
  }
}

// 速率限制中间件
const rateLimiter = new MultiTierRateLimiter(redis);

app.use(async (req, res, next) => {
  const userId = req.user?.id || req.ip;
  const tier = req.user?.tier || 'free';
  
  const result = await rateLimiter.checkRateLimit(userId, tier);
  
  res.setHeader('X-RateLimit-Limit', result.limit);
  res.setHeader('X-RateLimit-Remaining', result.remaining || 0);
  res.setHeader('X-RateLimit-Reset', result.resetAt);
  
  if (!result.allowed) {
    res.setHeader('Retry-After', result.retryAfter);
    return res.status(429).json({
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: '请求过于频繁，请稍后再试',
        retryAfter: result.retryAfter
      }
    });
  }
  
  next();
});
```

### 请求验证与清理

```javascript
// 请求验证管道
class RequestValidator {
  constructor() {
    this.sanitizers = [];
  }
  
  // 添加清理器
  addSanitizer(fn) {
    this.sanitizers.push(fn);
    return this;
  }
  
  // 清理字符串
  sanitizeString(field) {
    return this.addSanitizer((data) => {
      if (data[field] && typeof data[field] === 'string') {
        data[field] = data[field]
          .trim()
          .replace(/[<>]/g, '') // 移除潜在 HTML
          .substring(0, 1000);  // 限制长度
      }
      return data;
    });
  }
  
  // 清理数字
  sanitizeNumber(field, min, max) {
    return this.addSanitizer((data) => {
      if (data[field] !== undefined) {
        data[field] = Math.max(min, Math.min(max, Number(data[field]) || min));
      }
      return data;
    });
  }
  
  // 清理数组
  sanitizeArray(field, maxLength) {
    return this.addSanitizer((data) => {
      if (Array.isArray(data[field])) {
        data[field] = data[field].slice(0, maxLength);
      } else {
        data[field] = [];
      }
      return data;
    });
  }
  
  // 执行清理
  execute(data) {
    return this.sanitizers.reduce(
      (acc, sanitizer) => sanitizer(acc),
      data
    );
  }
}

// 内容安全策略
function addSecurityHeaders(app) {
  app.use((req, res, next) => {
    // 防止 XSS
    res.setHeader('X-XSS-Protection', '1; mode=block');
    res.setHeader('Content-Security-Policy', [
      "default-src 'self'",
      "script-src 'self' 'unsafe-inline'",
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self'",
      "connect-src 'self' https://api.example.com",
      "frame-ancestors 'none'"
    ].join('; '));
    
    // 防止点击劫持
    res.setHeader('X-Frame-Options', 'DENY');
    
    // MIME 类型 sniffing 防护
    res.setHeader('X-Content-Type-Options', 'nosniff');
    
    // 引用来源策略
    res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
    
    // 权限策略
    res.setHeader('Permissions-Policy', [
      'geolocation=()',
      'microphone=()',
      'camera=()'
    ].join(', '));
    
    next();
  });
}
```

---

## REST API 文档自动生成

### 从代码生成 OpenAPI

```javascript
// 装饰器风格的 API 定义
import { z } from 'zod';

class APIGenerator {
  constructor(router) {
    this.router = router;
    this.endpoints = [];
  }
  
  // 解析路由
  parseRoute(method, path, handlers) {
    const endpoint = {
      method: method.toUpperCase(),
      path,
      summary: handlers.summary || '',
      description: handlers.description || '',
      parameters: [],
      requestBody: null,
      responses: {},
      tags: handlers.tags || []
    };
    
    // 解析参数
    if (handlers.params) {
      for (const [name, schema] of Object.entries(handlers.params)) {
        endpoint.parameters.push({
          name,
          in: 'path',
          required: true,
          schema: this.zodToOpenAPI(schema)
        });
      }
    }
    
    // 解析查询参数
    if (handlers.query) {
      for (const [name, schema] of Object.entries(handlers.query)) {
        endpoint.parameters.push({
          name,
          in: 'query',
          required: false,
          schema: this.zodToOpenAPI(schema)
        });
      }
    }
    
    // 解析请求体
    if (handlers.body) {
      endpoint.requestBody = {
        required: true,
        content: {
          'application/json': {
            schema: this.zodToOpenAPI(handlers.body)
          }
        }
      };
    }
    
    // 解析响应
    if (handlers.responses) {
      for (const [code, response] of Object.entries(handlers.responses)) {
        endpoint.responses[code] = {
          description: response.description,
          content: response.schema ? {
            'application/json': {
              schema: this.zodToOpenAPI(response.schema)
            }
          } : undefined
        };
      }
    }
    
    this.endpoints.push(endpoint);
  }
  
  // Zod 到 OpenAPI Schema 转换
  zodToOpenAPI(schema) {
    if (!schema) return {};
    
    const map = {
      'string': { type: 'string' },
      'number': { type: 'number' },
      'integer': { type: 'integer' },
      'boolean': { type: 'boolean' },
      'array': { type: 'array' },
      'object': { type: 'object' }
    };
    
    if (schema._def) {
      const typeName = schema._def.typeName;
      
      if (typeName === 'ZodString') return { type: 'string' };
      if (typeName === 'ZodNumber') return { type: 'number' };
      if (typeName === 'ZodBoolean') return { type: 'boolean' };
      if (typeName === 'ZodArray') {
        return {
          type: 'array',
          items: this.zodToOpenAPI(schema._def.type)
        };
      }
      if (typeName === 'ZodObject') {
        const properties = {};
        for (const [key, val] of Object.entries(schema._def.shape())) {
          properties[key] = this.zodToOpenAPI(val);
        }
        return { type: 'object', properties };
      }
    }
    
    return {};
  }
  
  // 生成完整 OpenAPI 文档
  generate(spec = {}) {
    return {
      openapi: '3.1.0',
      info: {
        title: spec.title || 'API',
        version: spec.version || '1.0.0',
        description: spec.description || ''
      },
      paths: this.buildPaths()
    };
  }
  
  buildPaths() {
    const paths = {};
    
    for (const endpoint of this.endpoints) {
      if (!paths[endpoint.path]) {
        paths[endpoint.path] = {};
      }
      paths[endpoint.path][endpoint.method.toLowerCase()] = endpoint;
    }
    
    return paths;
  }
}
```

---

## REST API 最佳实践总结

### 目录结构推荐

```
my-api/
├── src/
│   ├── routes/                 # 路由定义
│   │   ├── index.js
│   │   ├── users.js
│   │   ├── posts.js
│   │   └── auth.js
│   │
│   ├── controllers/            # 业务逻辑
│   │   ├── userController.js
│   │   └── postController.js
│   │
│   ├── services/               # 数据访问
│   │   ├── userService.js
│   │   └── postService.js
│   │
│   ├── middleware/             # 中间件
│   │   ├── auth.js
│   │   ├── rateLimit.js
│   │   ├── validation.js
│   │   └── errorHandler.js
│   │
│   ├── utils/                  # 工具函数
│   │   ├── logger.js
│   │   └── validator.js
│   │
│   ├── schemas/                # 验证 Schema
│   │   ├── userSchema.js
│   │   └── postSchema.js
│   │
│   ├── docs/                   # API 文档
│   │   └── openapi.yaml
│   │
│   ├── app.js                  # Express 应用
│   └── server.js               # 服务器入口
│
├── tests/                      # 测试
├── package.json
└── .env
```

### 完整错误响应格式

```javascript
// 标准错误响应
const ErrorResponse = {
  create(type, message, details = {}) {
    return {
      error: {
        type,
        message,
        details,
        requestId: getRequestId(),
        timestamp: new Date().toISOString()
      }
    };
  }
};

// 预定义错误类型
const ErrorTypes = {
  VALIDATION_ERROR: {
    status: 400,
    message: '请求验证失败'
  },
  AUTHENTICATION_ERROR: {
    status: 401,
    message: '认证失败'
  },
  AUTHORIZATION_ERROR: {
    status: 403,
    message: '权限不足'
  },
  NOT_FOUND: {
    status: 404,
    message: '资源不存在'
  },
  CONFLICT: {
    status: 409,
    message: '资源冲突'
  },
  RATE_LIMIT: {
    status: 429,
    message: '请求过于频繁'
  },
  INTERNAL_ERROR: {
    status: 500,
    message: '服务器内部错误'
  }
};

// 全局错误处理器
app.use((err, req, res, next) => {
  const requestId = req.headers['x-request-id'];
  const status = err.status || 500;
  const type = err.type || 'INTERNAL_ERROR';
  const message = err.message || 'An unexpected error occurred';
  
  const response = {
    error: {
      type,
      message,
      details: err.details || {},
      requestId,
      timestamp: new Date().toISOString()
    }
  };
  
  // 生产环境不暴露内部错误详情
  if (status === 500 && process.env.NODE_ENV === 'production') {
    response.error.message = 'An unexpected error occurred';
    delete response.error.details;
  }
  
  res.status(status).json(response);
});
```

---

> [!TIP]
> REST API 设计是一个持续演进的过程。建议定期审查 API 的使用情况，根据实际需求调整设计策略。

---

## REST API 完整项目结构

### Next.js API Routes 实现

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { authMiddleware } from '@/lib/auth';
import { rateLimitMiddleware } from '@/lib/ratelimit';

// Schema 验证
const CreateUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8).optional(),
});

const UpdateUserSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
});

// GET /api/users - 获取用户列表
export async function GET(request: NextRequest) {
  try {
    // 速率限制
    const rateLimitResult = await rateLimitMiddleware(request);
    if (!rateLimitResult.allowed) {
      return rateLimitResult.response;
    }

    // 认证检查
    const auth = await authMiddleware(request);
    if (!auth.user) {
      return NextResponse.json(
        { error: { code: 'UNAUTHORIZED', message: 'Unauthorized' } },
        { status: 401 }
      );
    }

    // 解析查询参数
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = Math.min(parseInt(searchParams.get('limit') || '20'), 100);
    const search = searchParams.get('search') || '';
    const role = searchParams.get('role');

    // 构建查询
    const where: any = {};
    if (search) {
      where.OR = [
        { name: { contains: search, mode: 'insensitive' } },
        { email: { contains: search, mode: 'insensitive' } },
      ];
    }
    if (role) {
      where.role = role;
    }

    // 执行查询
    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: 'desc' },
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          createdAt: true,
          _count: { select: { posts: true } },
        },
      }),
      prisma.user.count({ where }),
    ]);

    // 返回分页结果
    return NextResponse.json({
      data: users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
        hasNextPage: page * limit < total,
        hasPrevPage: page > 1,
      },
    }, {
      headers: {
        'Cache-Control': 'public, max-age=60, stale-while-revalidate=300',
      },
    });
  } catch (error) {
    console.error('GET /api/users error:', error);
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } },
      { status: 500 }
    );
  }
}

// POST /api/users - 创建用户
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // 验证请求体
    const result = CreateUserSchema.safeParse(body);
    if (!result.success) {
      return NextResponse.json(
        {
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid request body',
            details: result.error.flatten().fieldErrors,
          },
        },
        { status: 400 }
      );
    }

    const { name, email, password } = result.data;

    // 检查邮箱唯一性
    const existingUser = await prisma.user.findUnique({
      where: { email },
    });

    if (existingUser) {
      return NextResponse.json(
        { error: { code: 'CONFLICT', message: 'Email already exists' } },
        { status: 409 }
      );
    }

    // 创建用户
    const user = await prisma.user.create({
      data: {
        name,
        email,
        password: password ? await hashPassword(password) : undefined,
      },
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        createdAt: true,
      },
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    console.error('POST /api/users error:', error);
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } },
      { status: 500 }
    );
  }
}
```

### 资源路由处理

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { authMiddleware } from '@/lib/auth';

const UpdateUserSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  email: z.string().email().optional(),
  role: z.enum(['USER', 'ADMIN', 'MODERATOR']).optional(),
});

// GET /api/users/[id]
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await prisma.user.findUnique({
      where: { id: params.id },
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        createdAt: true,
        updatedAt: true,
        posts: {
          take: 10,
          orderBy: { createdAt: 'desc' },
          select: { id: true, title: true, createdAt: true },
        },
        _count: { select: { posts: true, comments: true } },
      },
    });

    if (!user) {
      return NextResponse.json(
        { error: { code: 'NOT_FOUND', message: 'User not found' } },
        { status: 404 }
      );
    }

    return NextResponse.json(user);
  } catch (error) {
    console.error('GET /api/users/[id] error:', error);
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } },
      { status: 500 }
    );
  }
}

// PATCH /api/users/[id]
export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    // 认证检查
    const auth = await authMiddleware(request);
    if (!auth.user) {
      return NextResponse.json(
        { error: { code: 'UNAUTHORIZED', message: 'Unauthorized' } },
        { status: 401 }
      );
    }

    // 权限检查：只能修改自己的信息，除非是管理员
    if (auth.user.id !== params.id && auth.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: { code: 'FORBIDDEN', message: 'Forbidden' } },
        { status: 403 }
      );
    }

    const body = await request.json();
    const result = UpdateUserSchema.safeParse(body);

    if (!result.success) {
      return NextResponse.json(
        {
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid request body',
            details: result.error.flatten().fieldErrors,
          },
        },
        { status: 400 }
      );
    }

    // 检查邮箱唯一性（如果更新了邮箱）
    if (result.data.email) {
      const existingUser = await prisma.user.findFirst({
        where: { email: result.data.email, NOT: { id: params.id } },
      });

      if (existingUser) {
        return NextResponse.json(
          { error: { code: 'CONFLICT', message: 'Email already exists' } },
          { status: 409 }
        );
      }
    }

    const user = await prisma.user.update({
      where: { id: params.id },
      data: result.data,
      select: {
        id: true,
        name: true,
        email: true,
        role: true,
        updatedAt: true,
      },
    });

    return NextResponse.json(user);
  } catch (error) {
    console.error('PATCH /api/users/[id] error:', error);
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } },
      { status: 500 }
    );
  }
}

// DELETE /api/users/[id]
export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const auth = await authMiddleware(request);
    if (!auth.user) {
      return NextResponse.json(
        { error: { code: 'UNAUTHORIZED', message: 'Unauthorized' } },
        { status: 401 }
      );
    }

    // 权限检查
    if (auth.user.id !== params.id && auth.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: { code: 'FORBIDDEN', message: 'Forbidden' } },
        { status: 403 }
      );
    }

    // 检查用户是否存在
    const user = await prisma.user.findUnique({
      where: { id: params.id },
    });

    if (!user) {
      return NextResponse.json(
        { error: { code: 'NOT_FOUND', message: 'User not found' } },
        { status: 404 }
      );
    }

    // 删除用户（级联删除关联数据）
    await prisma.user.delete({
      where: { id: params.id },
    });

    return new NextResponse(null, { status: 204 });
  } catch (error) {
    console.error('DELETE /api/users/[id] error:', error);
    return NextResponse.json(
      { error: { code: 'INTERNAL_ERROR', message: 'Internal server error' } },
      { status: 500 }
    );
  }
}
```

---

## REST API GraphQL 迁移指南

### 迁移策略

```typescript
// 1. 渐进式迁移：共存期
// REST API 保持运行，逐步添加 GraphQL 端点

// app/api/graphql/route.ts
import { graphql, buildSchema } from 'graphql';

const schema = buildSchema(`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }

  type Query {
    user(id: ID!): User
    users(limit: Int): [User!]!
  }
`);

const root = {
  user: async ({ id }) => {
    return await prisma.user.findUnique({ where: { id } });
  },
  users: async ({ limit = 10 }) => {
    return await prisma.user.findMany({ take: limit });
  },
};

export async function POST(request: Request) {
  const { query, variables } = await request.json();
  
  try {
    const result = await graphql({
      schema,
      source: query,
      rootValue: root,
      variableValues: variables,
    });
    
    return Response.json(result);
  } catch (error) {
    return Response.json(
      { errors: [{ message: error.message }] },
      { status: 400 }
    );
  }
}
```

### 迁移检查清单

| 阶段 | 检查项 | 状态 |
|------|--------|------|
| **准备阶段** | 审计现有 REST API 端点 | ⬜ |
| | 识别 GraphQL 收益最大的端点 | ⬜ |
| | 制定迁移优先级 | ⬜ |
| **实施阶段** | 定义 GraphQL Schema | ⬜ |
| | 实现 Resolver | ⬜ |
| | 添加认证/授权 | ⬜ |
| | 实现 DataLoader | ⬜ |
| **共存阶段** | REST 和 GraphQL 并行运行 | ⬜ |
| | 监控性能差异 | ⬜ |
| | 收集使用数据 | ⬜ |
| **弃用阶段** | 通知客户端迁移 | ⬜ |
| | 设置弃用日期 | ⬜ |
| | 移除 REST 端点 | ⬜ |

---

## REST API 国际化支持

### 多语言响应

```typescript
// lib/i18n.ts
import { z } from 'zod';

const locales = ['en', 'zh', 'ja', 'es'] as const;
type Locale = (typeof locales)[number];

const messages: Record<Locale, Record<string, string>> = {
  en: {
    'error.not_found': 'Resource not found',
    'error.unauthorized': 'Authentication required',
    'error.forbidden': 'Access denied',
    'validation.required': 'This field is required',
    'validation.email': 'Invalid email format',
  },
  zh: {
    'error.not_found': '资源不存在',
    'error.unauthorized': '需要认证',
    'error.forbidden': '无访问权限',
    'validation.required': '此字段必填',
    'validation.email': '邮箱格式不正确',
  },
  ja: {
    'error.not_found': 'リソースが見つかりません',
    'error.unauthorized': '認証が必要です',
    'error.forbidden': 'アクセスが拒否されました',
    'validation.required': 'この項目は必須です',
    'validation.email': 'メール形式が無効です',
  },
  es: {
    'error.not_found': 'Recurso no encontrado',
    'error.unauthorized': 'Se requiere autenticación',
    'error.forbidden': 'Acceso denegado',
    'validation.required': 'Este campo es obligatorio',
    'validation.email': 'Formato de correo electrónico inválido',
  },
};

export function t(key: string, locale: Locale = 'en'): string {
  return messages[locale]?.[key] || messages['en'][key] || key;
}

// 国际化中间件
export function i18nMiddleware(request: NextRequest): Locale {
  // 1. 检查 URL 参数
  const urlLocale = new URL(request.url).searchParams.get('locale');
  if (urlLocale && locales.includes(urlLocale as Locale)) {
    return urlLocale as Locale;
  }

  // 2. 检查 Accept-Language header
  const acceptLanguage = request.headers.get('accept-language');
  if (acceptLanguage) {
    const preferredLocale = acceptLanguage
      .split(',')
      .map((lang) => lang.split(';')[0].trim().substring(0, 2))
      .find((lang) => locales.includes(lang as Locale));
    
    if (preferredLocale) {
      return preferredLocale as Locale;
    }
  }

  // 3. 默认语言
  return 'en';
}

// 国际化错误响应
export function localizedError(
  code: string,
  locale: Locale,
  replacements?: Record<string, string>
) {
  let message = t(`error.${code}`, locale);
  
  if (replacements) {
    Object.entries(replacements).forEach(([key, value]) => {
      message = message.replace(`{${key}}`, value);
    });
  }

  return {
    error: {
      code,
      message,
      locale,
    },
  };
}
```

---

## REST API 完整测试策略

### 集成测试

```typescript
// tests/api/users.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { createServer } from 'http';
import { prisma } from '@/lib/prisma';
import { app } from '@/app';

describe('Users API', () => {
  let server: ReturnType<typeof createServer>;
  let baseUrl: string;

  beforeAll(async () => {
    // 启动测试服务器
    server = createServer(app.callback());
    await new Promise((resolve) => server.listen(0, resolve));
    const { port } = server.address() as { port: number };
    baseUrl = `http://localhost:${port}`;

    // 清理并初始化测试数据
    await prisma.user.deleteMany();
    await prisma.user.create({
      data: {
        name: 'Test User',
        email: 'test@example.com',
        password: await hashPassword('password123'),
      },
    });
  });

  afterAll(async () => {
    await prisma.user.deleteMany();
    await prisma.$disconnect();
    server.close();
  });

  describe('GET /api/users', () => {
    it('should return paginated users', async () => {
      const response = await fetch(`${baseUrl}/api/users`);
      const data = await response.json();

      expect(response.status).toBe(200);
      expect(data.data).toBeInstanceOf(Array);
      expect(data.pagination).toBeDefined();
      expect(data.pagination.total).toBeGreaterThan(0);
    });

    it('should filter users by search term', async () => {
      const response = await fetch(`${baseUrl}/api/users?search=Test`);
      const data = await response.json();

      expect(response.status).toBe(200);
      expect(data.data.some((u: any) => u.name.includes('Test'))).toBe(true);
    });
  });

  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const newUser = {
        name: 'New User',
        email: 'newuser@example.com',
        password: 'securepassword123',
      };

      const response = await fetch(`${baseUrl}/api/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newUser),
      });

      expect(response.status).toBe(201);
      const data = await response.json();
      expect(data.email).toBe(newUser.email);
      expect(data.password).toBeUndefined(); // 密码不应返回
    });

    it('should reject duplicate email', async () => {
      const duplicateUser = {
        name: 'Duplicate User',
        email: 'test@example.com',
      };

      const response = await fetch(`${baseUrl}/api/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(duplicateUser),
      });

      expect(response.status).toBe(409);
      const data = await response.json();
      expect(data.error.code).toBe('CONFLICT');
    });

    it('should validate required fields', async () => {
      const invalidUser = { name: '' };

      const response = await fetch(`${baseUrl}/api/users`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(invalidUser),
      });

      expect(response.status).toBe(400);
      const data = await response.json();
      expect(data.error.code).toBe('VALIDATION_ERROR');
    });
  });

  describe('Authentication', () => {
    it('should reject unauthenticated requests', async () => {
      const response = await fetch(`${baseUrl}/api/users/123`, {
        method: 'DELETE',
      });

      expect(response.status).toBe(401);
    });

    it('should allow authenticated requests', async () => {
      // 登录获取 token
      const loginResponse = await fetch(`${baseUrl}/api/auth/login`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          email: 'test@example.com',
          password: 'password123',
        }),
      });

      const { token } = await loginResponse.json();

      // 使用 token 请求受保护的资源
      const response = await fetch(`${baseUrl}/api/users/me`, {
        headers: { Authorization: `Bearer ${token}` },
      });

      expect(response.status).toBe(200);
    });
  });
});
```

### 性能测试

```typescript
// tests/performance/load.test.ts
import { k6 } from '@load-grpc/k6';

export const options = {
  stages: [
    { duration: '2m', target: 100 },  // 预热
    { duration: '5m', target: 100 }, // 稳定负载
    { duration: '2m', target: 200 }, // 峰值
    { duration: '5m', target: 200 }, // 峰值持续
    { duration: '2m', target: 0 },   // 冷却
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],  // 95% 请求 < 500ms
    http_req_failed: ['rate<0.01'],     // 失败率 < 1%
  },
};

export default function () {
  const baseUrl = 'https://api.example.com';
  
  // 测试列表查询
  const listResponse = http.get(`${baseUrl}/api/users?limit=20`);
  check(listResponse, {
    'list status 200': (r) => r.status === 200,
    'list response time < 500ms': (r) => r.timings.duration < 500,
  });

  // 测试详情查询
  const detailResponse = http.get(`${baseUrl}/api/users/1`);
  check(detailResponse, {
    'detail status 200': (r) => r.status === 200,
    'detail has required fields': (r) => {
      const data = JSON.parse(r.body);
      return data.id && data.name && data.email;
    },
  });

  // 测试创建
  const createResponse = http.post(
    `${baseUrl}/api/users`,
    JSON.stringify({
      name: `Load Test User ${Date.now()}`,
      email: `loadtest${Date.now()}@example.com`,
    }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(createResponse, {
    'create status 201': (r) => r.status === 201,
  });
}
```

---

## REST API 部署与运维

### Kubernetes 配置

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
        - name: api
          image: myregistry/api:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: api-secrets
                  key: database-url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /live
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
    - port: 80
      targetPort: 3000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

### CI/CD 流水线

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      - name: Lint
        run: npm run lint

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Docker build
        run: |
          docker build -t myregistry/api:${{ github.sha }} .
          docker push myregistry/api:${{ github.sha }}
          docker tag myregistry/api:${{ github.sha }} myregistry/api:latest
          docker push myregistry/api:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        uses: azure/k8s-deploy@v1
        with:
          namespace: production
          manifests: |
            k8s/deployment.yaml
            k8s/service.yaml
            k8s/hpa.yaml
          images: |
            myregistry/api:${{ github.sha }}
```

---

> [!SUCCESS]
> REST API 作为 Web 服务接口的黄金标准，在 2026 年仍然是构建可扩展、可维护后端服务的首选。本文档涵盖了从设计原则到生产部署的完整知识体系，帮助开发者构建专业级的 RESTful API 服务。

---
