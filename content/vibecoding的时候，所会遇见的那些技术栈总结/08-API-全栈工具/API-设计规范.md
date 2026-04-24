# API 设计规范：文档工具全景解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，全面解析 OpenAPI/Swagger 规范、主流 API 文档工具，以及 MCP（Model Context Protocol）的设计理念与实践。

---

## 目录

1. [[#API 文档的重要性]]
2. [[#OpenAPI/Swagger 规范]]
3. [[#文档工具对比]]
4. [[#MCP 协议解析]]
5. [[#文档最佳实践]]
6. [[#实战配置指南]]
7. [[#选型建议]]

---

## API 文档的重要性

### 为什么需要 API 文档

API 文档是前后端分离开发的核心资产：

| 角色 | 文档价值 |
|------|---------|
| **前端开发者** | 了解接口规范，快速对接 |
| **后端开发者** | 明确接口契约，避免返工 |
| **测试工程师** | 自动化测试的依据 |
| **产品经理** | 了解系统能力 |
| **客户/合作伙伴** | 集成开发 |
| **AI Agent** | 工具调用理解 |

### 文档质量影响

```
文档质量 → 开发效率 → 产品交付 → 客户满意度
    ↓
低质量文档 → 沟通成本 ↑ → 返工率 ↑ → 维护成本 ↑ → 技术债务 ↑
```

---

## OpenAPI/Swagger 规范

### OpenAPI 简介

OpenAPI 规范（原 Swagger 规范）是一种用于描述 RESTful API 的标准格式。它使用 YAML 或 JSON 格式，提供机器可读且人类可读的 API 定义。

### 规范版本演进

| 版本 | 年份 | 主要特性 |
|------|------|---------|
| **Swagger 2.0** | 2014 | 基础规范 |
| **OpenAPI 3.0** | 2017 | 重构结构，支持 JSON Schema |
| **OpenAPI 3.1** | 2021 | 引入 JSON Schema 2020 |

### OpenAPI 3.x 结构

```yaml
# openapi.yaml
openapi: 3.1.0
info:
  title: My API
  description: API for managing tasks
  version: 1.0.0
  contact:
    name: API Support
    email: support@example.com
  license:
    name: MIT
    url: https://opensource.org/licenses/MIT

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: https://staging-api.example.com/v1
    description: Staging server

tags:
  - name: Users
    description: User management
  - name: Tasks
    description: Task management

paths:
  /users:
    get:
      summary: List users
      description: Retrieve a paginated list of users
      operationId: listUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
            minimum: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            minimum: 1
            maximum: 100
        - name: sort
          in: query
          schema:
            type: string
            enum: [asc, desc]
            default: desc
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'
        '400':
          $ref: '#/components/responses/BadRequest'
        '401':
          $ref: '#/components/responses/Unauthorized'

    post:
      summary: Create user
      description: Create a new user account
      operationId: createUser
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
          application/x-www-form-urlencoded:
            schema:
              $ref: '#/components/schemas/CreateUserForm'
      responses:
        '201':
          description: User created successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '400':
          $ref: '#/components/responses/BadRequest'
        '409':
          $ref: '#/components/responses/Conflict'

  /users/{userId}:
    get:
      summary: Get user
      description: Retrieve a specific user by ID
      operationId: getUser
      tags:
        - Users
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
        '404':
          $ref: '#/components/responses/NotFound'

    patch:
      summary: Update user
      description: Update an existing user
      operationId: updateUser
      tags:
        - Users
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUserRequest'
      responses:
        '200':
          description: User updated successfully
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'

    delete:
      summary: Delete user
      description: Delete a user account
      operationId: deleteUser
      tags:
        - Users
      parameters:
        - name: userId
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '204':
          description: User deleted successfully
        '404':
          $ref: '#/components/responses/NotFound'

components:
  schemas:
    User:
      type: object
      required:
        - id
        - email
      properties:
        id:
          type: string
          format: uuid
          description: Unique identifier
          example: "d290f1ee-6c54-4b01-90e6-d701748f0851"
        email:
          type: string
          format: email
          description: User's email address
          example: "user@example.com"
        name:
          type: string
          description: User's full name
          example: "John Doe"
        role:
          type: string
          enum: [user, admin, moderator]
          default: user
        createdAt:
          type: string
          format: date-time
          readOnly: true
        updatedAt:
          type: string
          format: date-time
          readOnly: true

    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'

    CreateUserRequest:
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
        name:
          type: string

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

  responses:
    BadRequest:
      description: Invalid request parameters
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    NotFound:
      description: Resource not found
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    
    Conflict:
      description: Resource already exists
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

### Schema 设计最佳实践

```yaml
# 统一错误响应
Error:
  type: object
  required:
    - code
    - message
  properties:
    code:
      type: string
      description: Machine-readable error code
      example: "USER_NOT_FOUND"
    message:
      type: string
      description: Human-readable error message
      example: "The requested user does not exist"
    details:
      type: object
      description: Additional error details
      additionalProperties: true
    requestId:
      type: string
      description: Request tracking ID
      example: "req_abc123"

# 枚举设计
UserRole:
  type: string
  enum:
    - user
    - admin
    - moderator
  description: |
    User roles with permission levels:
    - `user`: Standard user with basic access
    - `admin`: Full administrative access
    - `moderator`: Content moderation capabilities

# 可重用参数
Parameters:
  page:
    name: page
    in: query
    schema:
      type: integer
      default: 1
      minimum: 1
    description: Page number for pagination
  
  limit:
    name: limit
    in: query
    schema:
      type: integer
      default: 20
      minimum: 1
      maximum: 100
    description: Number of items per page
  
  sortOrder:
    name: sort
    in: query
    schema:
      type: string
      enum: [asc, desc]
      default: desc
    description: Sort order
```

---

## 文档工具对比

### 工具矩阵

| 工具 | 类型 | 特点 | 定价 |
|------|------|------|------|
| **Swagger UI** | 开源 | 经典，生态丰富 | 免费 |
| **Stoplight** | SaaS | 可视化设计 | 免费 + Pro |
| **Redocly** | 开源/SaaS | 文档美观 | 免费 + Pro |
| **Mintlify** | SaaS | 现代 UI | 免费 + Pro |
| **Docusaurus** | 开源 | React 基础 | 免费 |
| **RapiDoc** | 开源 | Web Components | 免费 |
| **Scalar** | 开源/SaaS | 现代设计 | 免费 + Pro |

### 工具详细对比

#### 1. Stoplight

```yaml
# stoplight.yaml
name: My API
routes:
  - path: /users
    operations:
      get:
        summary: List users
        responses:
          '200':
            body:
              application/json:
                schema:
                  type: array
                  items:
                    $ref: '#/components/schemas/User'
```

**特点：**
- 可视化 API 设计器
- 内置 Mock Server
- 版本控制
- 团队协作

#### 2. Redocly

```bash
# 安装
npm install @redocly/cli

# redocly.yaml
api:
  root: ./openapi.yaml

lint:
  extends:
    - recommended

docs:
  title: My API Documentation
  output: ./dist

serve:
  port: 3000
```

```bash
# 构建文档
npx redocly build-docs

# 启动本地服务器
npx redocly preview-docs
```

#### 3. Mintlify

```bash
# 安装
npm install -g mintlify

# 初始化
mintlify init

# 启动
mintlify dev
```

```javascript
// mintlify.config.ts
export default {
  name: 'My API',
  theme: 'light',
  logo: '/logo.svg',
  favicon: '/favicon.svg',
  colors: {
    primary: '#3b82f6',
    light: '#60a5fa',
    dark: '#1e40af',
  },
  navbar: {
    title: 'My API',
    links: [
      { title: 'Documentation', url: '/' },
      { title: 'API Reference', url: '/api-reference' },
    ],
  },
}
```

### 文档 UI 对比

| 工具 | UI 美观度 | 可定制性 | 响应式 |
|------|----------|----------|--------|
| **Swagger UI** | ⭐⭐⭐ | ⭐⭐ | ⚠️ 需主题 |
| **Redoc** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ |
| **Redocly** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ |
| **Mintlify** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ |
| **Scalar** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ |

---

## MCP 协议解析

### 什么是 MCP

MCP（Model Context Protocol）是 Anthropic 推出的开放协议，用于连接 AI 模型与外部工具和数据源。

### MCP 架构

```
┌─────────────────────────────────────────────────────────────┐
│                    MCP 架构                                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐         ┌─────────────────────────────┐   │
│  │   AI Model  │◄───────►│     MCP Host Application     │   │
│  │             │  JSON-RPC│                               │   │
│  └─────────────┘         │  ┌─────────────────────────┐  │   │
│                          │  │    MCP Client           │  │   │
│                          │  │  - Manages connections  │  │   │
│                          │  │  - Routes messages      │  │   │
│                          │  └─────────────────────────┘  │   │
│                          │                               │   │
│                          │  ┌─────────────────────────┐  │   │
│                          │  │   MCP Server           │  │   │
│                          │  │  - Exposes tools       │  │   │
│                          │  │  - Provides resources  │  │   │
│                          │  │  - Handles prompts    │  │   │
│                          │  └─────────────────────────┘  │   │
│                          └─────────────────────────────┘   │
│                                        │                    │
│                          ┌─────────────┴─────────────┐    │
│                          │       MCP Servers          │    │
│                          │  ┌─────────┐ ┌─────────┐  │    │
│                          │  │ File    │ │ GitHub  │  │    │
│                          │  │ System  │ │ Server  │  │    │
│                          │  └─────────┘ └─────────┘  │    │
│                          │  ┌─────────┐ ┌─────────┐  │    │
│                          │  │ Slack   │ │ Custom  │  │    │
│                          │  │ Server  │ │ Server  │  │    │
│                          │  └─────────┘ └─────────┘  │    │
│                          └─────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### MCP 服务器定义

```json
// server.json
{
  "name": "my-api-server",
  "version": "1.0.0",
  "protocolVersion": "2024-11-05",
  
  "capabilities": {
    "tools": {
      "list": true,
      "call": true
    },
    "resources": {
      "list": true,
      "read": true,
      "subscribe": true
    },
    "prompts": {
      "list": true,
      "get": true
    }
  }
}
```

### MCP 工具定义

```typescript
// my-server/src/index.ts
import { McpServer } from '@modelcontextprotocol/sdk/server'
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio'
import { z } from 'zod'

const server = new McpServer({
  name: 'my-api-server',
  version: '1.0.0'
})

// 定义工具
server.setRequestHandler('tools/list', async () => {
  return {
    tools: [
      {
        name: 'get_user',
        description: 'Get user information by ID',
        inputSchema: {
          type: 'object',
          properties: {
            userId: {
              type: 'string',
              description: 'The unique identifier of the user'
            }
          },
          required: ['userId']
        }
      },
      {
        name: 'create_post',
        description: 'Create a new blog post',
        inputSchema: {
          type: 'object',
          properties: {
            title: { type: 'string' },
            content: { type: 'string' },
            authorId: { type: 'string' }
          },
          required: ['title', 'authorId']
        }
      }
    ]
  }
})

server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params
  
  switch (name) {
    case 'get_user':
      return await handleGetUser(args.userId)
    case 'create_post':
      return await handleCreatePost(args)
    default:
      throw new Error(`Unknown tool: ${name}`)
  }
})

// 启动服务器
const transport = new StdioServerTransport()
await server.connect(transport)
```

### MCP 资源定义

```typescript
server.setRequestHandler('resources/list', async () => {
  return {
    resources: [
      {
        uri: 'api://users',
        name: 'User List',
        description: 'List of all users',
        mimeType: 'application/json'
      },
      {
        uri: 'api://posts/recent',
        name: 'Recent Posts',
        description: 'Recently published posts',
        mimeType: 'application/json'
      }
    ]
  }
})

server.setRequestHandler('resources/read', async (request) => {
  const { uri } = request.params
  
  if (uri === 'api://users') {
    const users = await db.users.findMany()
    return {
      contents: [{
        uri,
        mimeType: 'application/json',
        text: JSON.stringify(users)
      }]
    }
  }
  
  throw new Error(`Unknown resource: ${uri}`)
})
```

---

## 文档最佳实践

### 1. 文档即代码

```yaml
# 将 OpenAPI 规范纳入版本控制
# .github/workflows/api-docs.yml
name: API Documentation

on:
  push:
    paths:
      - 'openapi.yaml'
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to GitHub Pages
        run: |
          npx @redocly/cli build-docs openapi.yaml -o ./dist
          # 部署到 GitHub Pages
```

### 2. 自动化生成

```typescript
// 从代码生成 OpenAPI 文档
// 使用 tsoa
import { Route, Tags, Get, Post, Body, Query } from 'tsoa'

interface UserResponse {
  id: string
  email: string
  name?: string
}

@Tags('Users')
@Route('users')
export class UserController {
  @Get('{userId}')
  async getUser(userId: string): Promise<UserResponse> {
    const user = await userService.findById(userId)
    if (!user) throw new NotFoundError()
    return user
  }

  @Post()
  async createUser(
    @Body() body: { email: string; name?: string }
  ): Promise<UserResponse> {
    return userService.create(body)
  }
}
```

### 3. 版本管理

```yaml
# 多个版本的 API
openapi: 3.1.0
info:
  title: My API
  version: 2.0.0

servers:
  - url: https://api.example.com/v1
    description: Deprecated (EOL 2025-12-31)
  - url: https://api.example.com/v2
    description: Current version

paths:
  /users:
    get:
      summary: List users (v2)
      operationId: listUsersV2
      deprecated: false
```

### 4. 示例数据

```yaml
paths:
  /users:
    post:
      requestBody:
        content:
          application/json:
            example:
              email: "user@example.com"
              name: "John Doe"
              password: "securePassword123"
      responses:
        '201':
          content:
            application/json:
              example:
                id: "d290f1ee-6c54-4b01-90e6-d701748f0851"
                email: "user@example.com"
                name: "John Doe"
                createdAt: "2024-01-15T10:30:00Z"
```

---

## 实战配置指南

### 完整项目配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - API_URL=https://api.example.com
  
  docs:
    image: redocly/redoc
    ports:
      - "8080:80"
    volumes:
      - ./openapi.yaml:/usr/share/nginx/html/openapi.yaml
    environment:
      - SPEC_URL=/openapi.yaml
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name docs.example.com;
    
    location / {
        root /usr/share/nginx/html;
        index index.html;
        
        # Redoc 或自定义文档
    }
    
    location /api.yaml {
        alias /usr/share/nginx/html/openapi.yaml;
    }
}
```

---

## 选型建议

### 工具选择矩阵

| 场景 | 推荐工具 | 理由 |
|------|---------|------|
| 快速原型 | **Swagger UI** | 开箱即用 |
| 团队协作 | **Stoplight** | 协作功能完整 |
| 精美文档 | **Mintlify/Scalar** | 现代 UI |
| 开源自托管 | **Redocly** | 灵活部署 |
| Docusaurus 集成 | **RapiDoc** | React 友好 |
| AI Agent 集成 | **MCP** | 原生工具调用 |

---

## 参考资料

| 资源 | 链接 |
|------|------|
| OpenAPI 规范 | https://spec.openapis.org/ |
| Swagger Docs | https://swagger.io/docs/ |
| Stoplight | https://stoplight.io/ |
| Redocly | https://redocly.com/ |
| Mintlify | https://mintlify.com/ |
| MCP 协议 | https://modelcontextprotocol.io |

---

## API 高级设计模式

### 缓存策略设计

```typescript
// HTTP 缓存控制
interface CacheControl {
  public?: boolean;
  private?: boolean;
  maxAge?: number;
  sMaxAge?: number;
  noCache?: boolean;
  noStore?: boolean;
  mustRevalidate?: boolean;
  proxyRevalidate?: boolean;
  staleWhileRevalidate?: number;
  staleIfError?: number;
}

// 缓存策略配置
const cacheStrategies = {
  // 静态资源：长期缓存
  static: {
    public: true,
    maxAge: 31536000, // 1年
    immutable: true,
  },
  
  // 用户数据：私有，短期缓存
  userData: {
    private: true,
    maxAge: 300, // 5分钟
    mustRevalidate: true,
  },
  
  // 公共列表：公开，中期缓存
  publicList: {
    public: true,
    maxAge: 3600, // 1小时
    staleWhileRevalidate: 86400, // 即使过期也可返回1天
  },
  
  // 实时数据：不缓存
  realtime: {
    noStore: true,
  },
};
```

### 限流算法实现

```typescript
// 令牌桶算法
class TokenBucket {
  private tokens: number;
  private lastRefill: number;
  
  constructor(
    private capacity: number,
    private refillRate: number // tokens per second
  ) {
    this.tokens = capacity;
    this.lastRefill = Date.now();
  }
  
  private refill() {
    const now = Date.now();
    const elapsed = (now - this.lastRefill) / 1000;
    const newTokens = elapsed * this.refillRate;
    
    this.tokens = Math.min(this.capacity, this.tokens + newTokens);
    this.lastRefill = now;
  }
  
  consume(tokens = 1): boolean {
    this.refill();
    
    if (this.tokens >= tokens) {
      this.tokens -= tokens;
      return true;
    }
    
    return false;
  }
  
  getAvailableTokens(): number {
    this.refill();
    return this.tokens;
  }
}

// 滑动窗口日志算法
class SlidingWindowLog {
  private timestamps: Set<number> = new Set();
  
  constructor(
    private windowMs: number,
    private maxRequests: number
  ) {}
  
  isAllowed(): boolean {
    const now = Date.now();
    const windowStart = now - this.windowMs;
    
    // 清理过期记录
    for (const ts of this.timestamps) {
      if (ts < windowStart) {
        this.timestamps.delete(ts);
      }
    }
    
    if (this.timestamps.size < this.maxRequests) {
      this.timestamps.add(now);
      return true;
    }
    
    return false;
  }
  
  getResetTime(): number {
    if (this.timestamps.size === 0) return 0;
    
    const oldest = Math.min(...this.timestamps);
    return oldest + this.windowMs;
  }
}

// 固定窗口计数器
class FixedWindowCounter {
  private count = 0;
  private windowStart = Date.now();
  
  constructor(
    private windowMs: number,
    private maxRequests: number
  ) {}
  
  isAllowed(): boolean {
    const now = Date.now();
    
    if (now - this.windowStart >= this.windowMs) {
      this.count = 0;
      this.windowStart = now;
    }
    
    if (this.count < this.maxRequests) {
      this.count++;
      return true;
    }
    
    return false;
  }
}
```

### API 网关设计

```typescript
// Kong/Nginx 配置示例
// upstream 后端服务
upstream backend {
    server api1.example.com:8080;
    server api2.example.com:8080;
    server api3.example.com:8080;
    
    keepalive 32;
}

// 限流配置
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/s;
limit_req_zone $http_authorization zone=auth_limit:10m rate=10r/s;

// 缓存配置
proxy_cache_path /var/cache/nginx/api levels=1:2 keys_zone=api_cache:10m 
                 max_size=1g inactive=60m use_temp_path=off;

// 主配置
server {
    listen 80;
    server_name api.example.com;
    
    # 限流
    limit_req zone=api_limit burst=20 nodelay;
    
    # 请求日志
    log_format api_log '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $body_bytes_sent '
                       '"$http_referer" "$http_user_agent"';
    
    access_log /var/log/nginx/api.log api_log;
    
    location /api/ {
        # 代理配置
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # 连接池
        proxy_connect_timeout 30s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
        
        # 缓存
        proxy_cache api_cache;
        proxy_cache_valid 200 5m;
        proxy_cache_key "$scheme$request_method$host$request_uri";
        add_header X-Cache-Status $upstream_cache_status;
        
        # CORS
        add_header Access-Control-Allow-Origin $http_origin;
        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        add_header Access-Control-Allow-Headers "Authorization, Content-Type";
    }
    
    # 认证服务
    location /auth/ {
        proxy_pass http://auth-backend;
        limit_req zone=auth_limit burst=5;
    }
}
```

### GraphQL Federation 路由

```typescript
// Apollo Federation Gateway 配置
// gateway.js
const { ApolloGateway } = require('@apollo/gateway');
const { ApolloServer } = require('apollo-server-express');
const express = require('express');

const gateway = new ApolloGateway({
  serviceList: [
    { name: 'users', url: process.env.USERS_SERVICE_URL },
    { name: 'products', url: process.env.PRODUCTS_SERVICE_URL },
    { name: 'orders', url: process.env.ORDERS_SERVICE_URL },
    { name: 'inventory', url: process.env.INVENTORY_SERVICE_URL },
  ],
  
  // _introspection 查询支持
  introspectionHeaders: {
    'x-api-key': process.env.SUPERGRAPH_KEY,
  },
});

async function startGateway() {
  const server = new ApolloServer({
    gateway,
    subscriptions: false,
    playground: process.env.NODE_ENV === 'development',
    formatError: (error) => {
      // 错误格式化
      return {
        message: error.message,
        locations: error.locations,
        path: error.path,
      };
    },
    plugins: [
      {
        async requestDidStart({ request, logger }) {
          return {
            async didResolveOperation() {
              logger.info('Operation resolved');
            },
            async didEncounterErrors({ errors }) {
              logger.error('Errors encountered', { errors });
            },
          };
        },
      },
    ],
  });
  
  const app = express();
  await server.start();
  server.applyMiddleware({ app, path: '/graphql' });
  
  app.listen(4000, () => {
    console.log('Gateway ready at http://localhost:4000/graphql');
  });
}

startGateway();
```

### Webhook 设计与验证

```typescript
// Webhook 签名验证
const crypto = require('crypto');

class WebhookVerifier {
  private secret: string;
  
  constructor(secret: string) {
    this.secret = secret;
  }
  
  verify(
    payload: string,
    signature: string,
    timestamp?: string
  ): boolean {
    // 时间戳验证（防止重放攻击）
    if (timestamp) {
      const ts = parseInt(timestamp, 10);
      const now = Math.floor(Date.now() / 1000);
      
      if (Math.abs(now - ts) > 300) { // 5分钟窗口
        return false;
      }
    }
    
    // 计算签名
    const computed = this.computeSignature(payload, timestamp);
    
    return crypto.timingSafeEqual(
      Buffer.from(signature),
      Buffer.from(computed)
    );
  }
  
  private computeSignature(payload: string, timestamp?: string): string {
    const timestampPart = timestamp ? `${timestamp}.` : '';
    const payloadPart = typeof payload === 'string' ? payload : JSON.stringify(payload);
    
    return crypto
      .createHmac('sha256', this.secret)
      .update(`${timestampPart}${payloadPart}`)
      .digest('hex');
  }
  
  // 生成签名（供发送方使用）
  sign(payload: string): { signature: string; timestamp: string } {
    const timestamp = Math.floor(Date.now() / 1000).toString();
    const signature = this.computeSignature(payload, timestamp);
    
    return { signature, timestamp };
  }
}

// Webhook 事件处理
class WebhookHandler {
  private handlers: Map<string, Function[]> = new Map();
  private verifier: WebhookVerifier;
  
  constructor(secret: string) {
    this.verifier = new WebhookVerifier(secret);
  }
  
  on(event: string, handler: Function) {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, []);
    }
    this.handlers.get(event)!.push(handler);
  }
  
  async handle(req: Request): Promise<Response> {
    const signature = req.headers.get('x-webhook-signature');
    const timestamp = req.headers.get('x-webhook-timestamp');
    const event = req.headers.get('x-webhook-event');
    
    if (!signature || !event) {
      return new Response('Missing headers', { status: 400 });
    }
    
    const payload = await req.text();
    
    // 验证签名
    if (!this.verifier.verify(payload, signature, timestamp || undefined)) {
      return new Response('Invalid signature', { status: 401 });
    }
    
    const data = JSON.parse(payload);
    const handlers = this.handlers.get(event) || [];
    
    // 并行执行所有处理器
    const results = await Promise.allSettled(
      handlers.map(handler => handler(data))
    );
    
    // 记录失败的处理
    const failures = results.filter(r => r.status === 'rejected');
    if (failures.length > 0) {
      console.error('Webhook handlers failed:', failures);
    }
    
    return new Response('OK', { status: 200 });
  }
}
```

### API 灰度发布策略

```typescript
// 权重分流
class WeightedRouter {
  private weights: Map<string, number> = new Map();
  private totalWeight = 0;
  
  addTarget(id: string, weight: number) {
    this.weights.set(id, weight);
    this.totalWeight = [...this.weights.values()].reduce((a, b) => a + b, 0);
  }
  
  route(userId: string): string {
    const hash = this.hashUserId(userId);
    const threshold = (hash % this.totalWeight) / this.totalWeight;
    
    let cumulative = 0;
    for (const [id, weight] of this.weights.entries()) {
      cumulative += weight / this.totalWeight;
      if (threshold < cumulative) {
        return id;
      }
    }
    
    return this.weights.keys().next().value;
  }
  
  private hashUserId(userId: string): number {
    let hash = 0;
    for (let i = 0; i < userId.length; i++) {
      const char = userId.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash;
    }
    return Math.abs(hash);
  }
}

// 功能开关
class FeatureGate {
  private features: Map<string, FeatureConfig> = new Map();
  
  addFeature(name: string, config: FeatureConfig) {
    this.features.set(name, config);
  }
  
  isEnabled(
    featureName: string,
    context: { userId: string; attributes?: Record<string, unknown> }
  ): boolean {
    const feature = this.features.get(featureName);
    if (!feature) return false;
    if (feature.enabled === false) return false;
    
    // 百分比灰度
    if (feature.percentage !== undefined) {
      const hash = this.hashUserId(context.userId);
      return (hash % 100) < feature.percentage;
    }
    
    // 属性匹配
    if (feature.conditions) {
      return this.evaluateConditions(feature.conditions, context.attributes);
    }
    
    return true;
  }
  
  private evaluateConditions(
    conditions: Condition[],
    attributes?: Record<string, unknown>
  ): boolean {
    return conditions.every(cond => {
      const value = attributes?.[cond.key];
      
      switch (cond.operator) {
        case 'equals':
          return value === cond.value;
        case 'in':
          return Array.isArray(cond.value) && cond.value.includes(value);
        case 'contains':
          return typeof value === 'string' && value.includes(cond.value as string);
        default:
          return false;
      }
    });
  }
}
```

### API 监控与告警

```typescript
// 指标收集
class APIMetrics {
  private counters: Map<string, number> = new Map();
  private histograms: Map<string, number[]> = new Map();
  
  incrementCounter(name: string, tags?: Record<string, string>) {
    const key = this.makeKey(name, tags);
    this.counters.set(key, (this.counters.get(key) || 0) + 1);
  }
  
  recordHistogram(name: string, value: number, tags?: Record<string, string>) {
    const key = this.makeKey(name, tags);
    const values = this.histograms.get(key) || [];
    values.push(value);
    this.histograms.set(key, values);
  }
  
  getStats(name: string, tags?: Record<string, string>) {
    const key = this.makeKey(name, tags);
    const values = this.histograms.get(key) || [];
    
    if (values.length === 0) return null;
    
    const sorted = [...values].sort((a, b) => a - b);
    const sum = values.reduce((a, b) => a + b, 0);
    
    return {
      count: values.length,
      sum,
      avg: sum / values.length,
      min: sorted[0],
      max: sorted[sorted.length - 1],
      p50: sorted[Math.floor(sorted.length * 0.5)],
      p90: sorted[Math.floor(sorted.length * 0.9)],
      p95: sorted[Math.floor(sorted.length * 0.95)],
      p99: sorted[Math.floor(sorted.length * 0.99)],
    };
  }
  
  private makeKey(name: string, tags?: Record<string, string>): string {
    if (!tags) return name;
    const tagStr = Object.entries(tags)
      .map(([k, v]) => `${k}=${v}`)
      .join(',');
    return `${name}{${tagStr}}`;
  }
}

// 告警规则
const alertRules = [
  {
    name: 'high_error_rate',
    condition: (metrics) => {
      const rate = metrics.getStats('http_requests_total', { status: '5xx' })?.count || 0;
      const total = metrics.getStats('http_requests_total')?.count || 1;
      return (rate / total) > 0.05; // 5% 错误率
    },
    severity: 'critical',
    message: 'API 错误率超过 5%',
  },
  {
    name: 'high_latency',
    condition: (metrics) => {
      const p99 = metrics.getStats('http_request_duration_ms')?.p99 || 0;
      return p99 > 1000; // P99 延迟超过 1秒
    },
    severity: 'warning',
    message: 'API P99 延迟超过 1秒',
  },
];
```

---

## API 未来趋势

### 趋势一：API-as-a-Product

现代 API 不再只是技术接口，而是产品。API 产品化管理需要关注：

```typescript
// API 产品化要素
const apiProductStrategy = {
  // 开发者体验
  developerExperience: {
    quickStart: '5分钟入门指南',
    sdks: ['JavaScript', 'Python', 'Go', 'Java'],
    sandbox: true,
    interactiveDocs: true,
  },
  
  // 定价模型
  pricing: {
    model: 'usage-based', // 按量付费
    tiers: [
      { name: 'free', limits: { requests: 1000, rateLimit: 10 } },
      { name: 'pro', limits: { requests: 100000, rateLimit: 100 } },
      { name: 'enterprise', limits: { requests: -1, rateLimit: -1 } },
    ],
  },
  
  // 支持渠道
  support: {
    documentation: 'https://docs.example.com',
    discord: 'https://discord.gg/example',
    email: 'api@example.com',
  },
};
```

### 趋势二：AI-Native APIs

AI 时代 API 的新需求：

```typescript
// 结构化输出 API
const structuredOutputAPI = {
  // 强制 JSON 模式
  responseFormat: {
    type: 'json_object',
    schema: {
      type: 'object',
      properties: {
        summary: { type: 'string' },
        sentiment: { type: 'string', enum: ['positive', 'neutral', 'negative'] },
        keyPoints: { type: 'array', items: { type: 'string' } },
      },
      required: ['summary', 'sentiment'],
    },
  },
  
  // 函数调用
  tools: [
    {
      type: 'function',
      function: {
        name: 'get_weather',
        description: '获取天气信息',
        parameters: {
          type: 'object',
          properties: {
            location: { type: 'string' },
            unit: { type: 'string', enum: ['celsius', 'fahrenheit'] },
          },
        },
      },
    },
  ],
  
  // 流式响应
  streaming: true,
};
```

### 趋势三：边缘 API

边缘计算带来的 API 变革：

```typescript
// 边缘函数示例
const edgeFunction = `
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});

async function handleRequest(request) {
  const url = new URL(request.url);
  
  // 边缘缓存
  const cache = caches.default;
  const cached = await cache.match(request);
  if (cached) {
    return cached;
  }
  
  // 边缘处理
  const response = await fetch(request);
  
  // 添加缓存头
  const newResponse = new Response(response.body, response);
  newResponse.headers.set('Cache-Control', 's-maxage=3600');
  
  await cache.put(request, newResponse.clone());
  return newResponse;
}
`;
```

---

> [!TIP]
> API 设计是一个持续演进的过程。随着业务发展和新技术出现，API 设计也需要不断优化。保持对行业趋势的关注，持续改进 API 设计，可以让你的 API 保持竞争力。

---

## API 安全性设计

### 认证与授权机制

API 安全是现代应用开发的核心关注点。以下是主流的认证方案：

```yaml
# OpenAPI 安全方案定义
components:
  securitySchemes:
    # JWT Bearer Token
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: 使用 JWT Token 进行认证
    
    # API Key
    apiKeyAuth:
      type: apiKey
      in: header
      name: X-API-Key
      description: 使用 API Key 进行认证
    
    # OAuth 2.0
    oauth2:
      type: oauth2
      flows:
        authorizationCode:
          authorizationUrl: https://auth.example.com/authorize
          tokenUrl: https://auth.example.com/token
          scopes:
            read: 读取权限
            write: 写入权限
            admin: 管理员权限
        
        clientCredentials:
          tokenUrl: https://auth.example.com/token
          scopes:
            read: 读取权限
            write: 写入权限
    
    # Basic Auth
    basicAuth:
      type: http
      scheme: basic
      description: 用户名密码认证

# 应用安全方案
security:
  - bearerAuth: []          # 全局需要 JWT
  - apiKeyAuth: []          # 全局需要 API Key
  - oauth2: [read, write]   # OAuth2 认证
```

### JWT 实现详解

```typescript
// JWT Token 生成与验证
import jwt from 'jsonwebtoken';
import { z } from 'zod';

// Token 载荷定义
interface TokenPayload {
  sub: string;      // 用户 ID
  email: string;    // 用户邮箱
  role: string[];   // 用户角色
  iat: number;      // 签发时间
  exp: number;      // 过期时间
}

// 密钥管理（生产环境应使用 KMS）
const JWT_SECRET = process.env.JWT_SECRET || 'dev-secret-change-in-production';
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

// 生成 Access Token
function generateAccessToken(user: User): string {
  return jwt.sign(
    {
      sub: user.id,
      email: user.email,
      role: user.roles,
    },
    JWT_SECRET,
    { 
      expiresIn: ACCESS_TOKEN_EXPIRY,
      issuer: 'api.example.com',
      audience: 'api.example.com',
    }
  );
}

// 生成 Refresh Token
function generateRefreshToken(user: User): string {
  return jwt.sign(
    {
      sub: user.id,
      type: 'refresh',
    },
    JWT_SECRET,
    { expiresIn: REFRESH_TOKEN_EXPIRY }
  );
}

// 验证 Token
function verifyToken(token: string): TokenPayload {
  try {
    return jwt.verify(token, JWT_SECRET, {
      issuer: 'api.example.com',
      audience: 'api.example.com',
    }) as TokenPayload;
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      throw new AuthenticationError('Token 已过期');
    }
    if (error instanceof jwt.JsonWebTokenError) {
      throw new AuthenticationError('无效的 Token');
    }
    throw error;
  }
}

// Token 刷新机制
async function refreshTokens(refreshToken: string): Promise<{
  accessToken: string;
  refreshToken: string;
}> {
  const payload = verifyToken(refreshToken);
  
  if ((payload as any).type !== 'refresh') {
    throw new AuthenticationError('无效的 Refresh Token');
  }
  
  const user = await userService.findById(payload.sub);
  if (!user) {
    throw new AuthenticationError('用户不存在');
  }
  
  // 检查用户是否被禁用
  if (user.status === 'disabled') {
    throw new AuthenticationError('账户已被禁用');
  }
  
  return {
    accessToken: generateAccessToken(user),
    refreshToken: generateRefreshToken(user),
  };
}
```

### OAuth 2.0 完整流程

```typescript
// OAuth 2.0 Authorization Server 实现
class OAuth2Server {
  private codeStore: Map<string, AuthorizationCode> = new Map();
  
  // 生成授权码
  async generateAuthorizationCode(
    clientId: string,
    redirectUri: string,
    userId: string,
    scopes: string[]
  ): Promise<string> {
    const code = crypto.randomBytes(32).toString('hex');
    
    this.codeStore.set(code, {
      clientId,
      redirectUri,
      userId,
      scopes,
      expiresAt: Date.now() + 10 * 60 * 1000, // 10分钟过期
      used: false,
    });
    
    return code;
  }
  
  // 验证授权码
  async validateAuthorizationCode(
    code: string,
    clientId: string,
    redirectUri: string
  ): Promise<AuthorizationCode | null> {
    const authCode = this.codeStore.get(code);
    
    if (!authCode) return null;
    if (authCode.clientId !== clientId) return null;
    if (authCode.redirectUri !== redirectUri) return null;
    if (authCode.expiresAt < Date.now()) return null;
    if (authCode.used) return null;
    
    // 标记为已使用
    authCode.used = true;
    
    return authCode;
  }
  
  // 生成 Token
  async generateTokens(authCode: AuthorizationCode): Promise<{
    accessToken: string;
    refreshToken: string;
    expiresIn: number;
    tokenType: string;
  }> {
    const user = await userService.findById(authCode.userId);
    
    const accessToken = jwt.sign(
      {
        sub: user.id,
        email: user.email,
        role: authCode.scopes,
        type: 'access',
      },
      JWT_SECRET,
      { expiresIn: ACCESS_TOKEN_EXPIRY }
    );
    
    const refreshToken = jwt.sign(
      {
        sub: user.id,
        type: 'refresh',
        jti: crypto.randomUUID(),
      },
      JWT_SECRET,
      { expiresIn: REFRESH_TOKEN_EXPIRY }
    );
    
    return {
      accessToken,
      refreshToken,
      expiresIn: 900, // 15分钟
      tokenType: 'Bearer',
    };
  }
  
  // 撤销 Token
  async revokeToken(token: string): Promise<boolean> {
    try {
      const payload = jwt.verify(token, JWT_SECRET) as any;
      
      // 将 JTI 加入黑名单
      if (payload.jti) {
        await this.tokenBlacklist.add(payload.jti);
      }
      
      return true;
    } catch {
      return false;
    }
  }
}

// PKCE 扩展（Proof Key for Code Exchange）
class PKCEUtils {
  // 生成 code verifier
  static generateCodeVerifier(): string {
    return crypto.randomBytes(32).toString('base64url');
  }
  
  // 生成 code challenge
  static generateCodeChallenge(verifier: string): string {
    return crypto.createHash('sha256')
      .update(verifier)
      .digest('base64url');
  }
  
  // 验证 code verifier
  static verifyCodeChallenge(verifier: string, challenge: string, method: string): boolean {
    if (method === 'S256') {
      return this.generateCodeChallenge(verifier) === challenge;
    }
    // plain 方法已废弃，仅用于测试
    return verifier === challenge;
  }
}
```

### API 限流与配额管理

```typescript
// 分布式限流实现
class RateLimiter {
  private redis: Redis;
  private luaScript: string;
  
  constructor(redis: Redis) {
    this.redis = redis;
    // Redis Lua 脚本保证原子性
    this.luaScript = `
      local key = KEYS[1]
      local limit = tonumber(ARGV[1])
      local window = tonumber(ARGV[2])
      local current = tonumber(redis.call('GET', key) or '0')
      
      if current >= limit then
        return 0
      end
      
      current = redis.call('INCR', key)
      if current == 1 then
        redis.call('EXPIRE', key, window)
      end
      
      return 1
    `;
  }
  
  // 检查限流
  async isAllowed(key: string, limit: number, windowSeconds: number): Promise<{
    allowed: boolean;
    remaining: number;
    resetAt: number;
  }> {
    const fullKey = `rate_limit:${key}`;
    
    const result = await this.redis.eval(
      this.luaScript,
      1,
      fullKey,
      limit.toString(),
      windowSeconds.toString()
    ) as number;
    
    const current = await this.redis.get(fullKey);
    const ttl = await this.redis.ttl(fullKey);
    
    return {
      allowed: result === 1,
      remaining: Math.max(0, limit - (parseInt(current) || 0)),
      resetAt: Date.now() + (ttl > 0 ? ttl * 1000 : windowSeconds * 1000),
    };
  }
  
  // 添加限流响应头
  addRateLimitHeaders(response: Response, result: RateLimitResult): Response {
    response.headers.set('X-RateLimit-Limit', result.limit.toString());
    response.headers.set('X-RateLimit-Remaining', result.remaining.toString());
    response.headers.set('X-RateLimit-Reset', Math.floor(result.resetAt / 1000).toString());
    
    if (!result.allowed) {
      response.headers.set('Retry-After', Math.ceil((result.resetAt - Date.now()) / 1000).toString());
    }
    
    return response;
  }
}

// 限流配置
const rateLimitConfig = {
  // 按端点类型配置
  endpoints: {
    '/api/auth/login': { limit: 5, window: 60 },      // 登录：每分钟5次
    '/api/auth/register': { limit: 3, window: 3600 },   // 注册：每小时3次
    '/api/': { limit: 100, window: 60 },              // 普通API：每分钟100次
    '/api/search': { limit: 30, window: 60 },          // 搜索：每分钟30次
  },
  
  // 按用户等级配置
  tiers: {
    free: { limit: 100, window: 60 },
    basic: { limit: 1000, window: 60 },
    pro: { limit: 10000, window: 60 },
    enterprise: { limit: -1, window: 1 }, // 无限制
  },
  
  // 全局限流
  global: { limit: 1000, window: 1 }, // 每秒1000请求
};
```

### 输入验证与安全防护

```typescript
// 统一的输入验证层
import { z } from 'zod';

class InputValidator {
  // 用户相关 schema
  static userSchemas = {
    create: z.object({
      email: z.string().email().max(255),
      password: z.string()
        .min(8, '密码至少8位')
        .max(128, '密码最多128位')
        .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, '密码需包含大小写字母和数字'),
      name: z.string().min(1).max(100),
      role: z.enum(['user', 'moderator']).default('user'),
    }),
    
    update: z.object({
      email: z.string().email().optional(),
      name: z.string().min(1).max(100).optional(),
      bio: z.string().max(500).optional(),
    }),
    
    query: z.object({
      page: z.coerce.number().int().positive().default(1),
      limit: z.coerce.number().int().min(1).max(100).default(20),
      sort: z.enum(['asc', 'desc']).default('desc'),
      search: z.string().max(100).optional(),
    }),
  };
  
  // 验证并格式化输入
  static validate<T>(schema: z.ZodSchema<T>, data: unknown): T {
    const result = schema.safeParse(data);
    
    if (!result.success) {
      const errors = result.error.errors.map(e => ({
        field: e.path.join('.'),
        message: e.message,
      }));
      throw new ValidationError('输入验证失败', errors);
    }
    
    return result.data;
  }
}

// SQL 注入防护
class SQLInjectionProtector {
  private dangerousPatterns = [
    /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|EXECUTE|UNION)\b)/i,
    /(--|;|\/\*|\*\/|@@|@)/,
    /(\bOR\b|\bAND\b).*=.*$/i,
  ];
  
  validate(input: string): boolean {
    for (const pattern of this.dangerousPatterns) {
      if (pattern.test(input)) {
        return false;
      }
    }
    return true;
  }
  
  sanitize(input: string): string {
    return input
      .replace(/\\/g, '\\\\')
      .replace(/'/g, "\\'")
      .replace(/"/g, '\\"')
      .replace(/\n/g, '\\n')
      .replace(/\r/g, '\\r');
  }
}

// XSS 防护
class XSSProtector {
  private htmlEscape(str: string): string {
    const escapeMap: Record<string, string> = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#x27;',
      '/': '&#x2F;',
    };
    
    return str.replace(/[&<>"'/]/g, char => escapeMap[char]);
  }
  
  sanitizeHTML(input: string): string {
    // 移除所有 HTML 标签
    return this.htmlEscape(input);
  }
  
  sanitizeRichText(input: string): string {
    // 使用 DOMPurify 清理富文本
    // return DOMPurify.sanitize(input, { ALLOWED_TAGS: ['p', 'br', 'b', 'i', 'em'] });
    return input;
  }
}
```

### CORS 与 CSP 配置

```typescript
// CORS 配置
const corsConfig = {
  // 允许的源
  origin: (origin: string | undefined, callback: Function) => {
    const allowedOrigins = [
      'https://example.com',
      'https://app.example.com',
      'https://admin.example.com',
    ];
    
    // 允许没有 origin 的请求（如 Postman、SSR）
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('不允许的 CORS 源'));
    }
  },
  
  // 允许的方法
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  
  // 允许的头部
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'X-CSRF-Token',
    'X-API-Key',
  ],
  
  // 暴露的头部
  exposedHeaders: [
    'X-RateLimit-Limit',
    'X-RateLimit-Remaining',
    'X-RateLimit-Reset',
    'X-Request-Id',
  ],
  
  // 是否允许凭证
  credentials: true,
  
  // 预检请求缓存时间
  maxAge: 86400, // 24小时
};

// CSP 配置
const cspConfig = `
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://cdn.example.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  font-src 'self' https://fonts.gstatic.com;
  img-src 'self' data: https: blob:;
  media-src 'self' https://media.example.com;
  connect-src 'self' https://api.example.com wss://ws.example.com;
  frame-ancestors 'none';
  form-action 'self';
  base-uri 'self';
  object-src 'none';
`;
```

---

## API 性能优化

### 缓存策略实现

```typescript
// 多层缓存架构
class CacheManager {
  private memoryCache: Map<string, CacheEntry> = new Map();
  private redis: Redis;
  
  constructor(private redis: Redis) {
    // 启动后台清理任务
    this.startCleanupTask();
  }
  
  // 内存缓存
  async get<T>(key: string): Promise<T | null> {
    // L1: 内存缓存
    const memoryEntry = this.memoryCache.get(key);
    if (memoryEntry && !this.isExpired(memoryEntry)) {
      return memoryEntry.value as T;
    }
    
    // L2: Redis 缓存
    const redisValue = await this.redis.get(`cache:${key}`);
    if (redisValue) {
      const entry = JSON.parse(redisValue);
      // 回填内存缓存
      this.memoryCache.set(key, {
        value: entry.value,
        expiresAt: entry.expiresAt,
      });
      return entry.value as T;
    }
    
    return null;
  }
  
  async set<T>(
    key: string, 
    value: T, 
    options: { ttl?: number; memoryTTL?: number } = {}
  ): Promise<void> {
    const now = Date.now();
    const ttl = options.ttl || 3600; // 默认1小时
    const memoryTTL = options.memoryTTL || 60; // 内存缓存1分钟
    
    const entry: CacheEntry = {
      value,
      expiresAt: now + ttl * 1000,
    };
    
    // 设置 Redis 缓存
    await this.redis.setex(
      `cache:${key}`,
      ttl,
      JSON.stringify(entry)
    );
    
    // 设置内存缓存
    this.memoryCache.set(key, {
      value,
      expiresAt: now + memoryTTL * 1000,
    });
  }
  
  async invalidate(pattern: string): Promise<number> {
    let count = 0;
    
    // 清理内存缓存
    for (const key of this.memoryCache.keys()) {
      if (this.matchPattern(key, pattern)) {
        this.memoryCache.delete(key);
        count++;
      }
    }
    
    // 清理 Redis 缓存
    const keys = await this.redis.keys(`cache:${pattern}`);
    if (keys.length > 0) {
      await this.redis.del(...keys);
      count += keys.length;
    }
    
    return count;
  }
  
  private startCleanupTask(): void {
    // 每分钟清理过期缓存
    setInterval(() => {
      const now = Date.now();
      for (const [key, entry] of this.memoryCache.entries()) {
        if (entry.expiresAt < now) {
          this.memoryCache.delete(key);
        }
      }
    }, 60000);
  }
  
  private isExpired(entry: CacheEntry): boolean {
    return entry.expiresAt < Date.now();
  }
  
  private matchPattern(key: string, pattern: string): boolean {
    const regex = new RegExp('^' + pattern.replace(/\*/g, '.*') + '$');
    return regex.test(key);
  }
}

// HTTP 缓存控制
interface CacheControlOptions {
  public?: boolean;
  private?: boolean;
  maxAge?: number;
  sMaxAge?: number;
  noCache?: boolean;
  noStore?: boolean;
  mustRevalidate?: boolean;
  proxyRevalidate?: boolean;
  staleWhileRevalidate?: number;
  staleIfError?: number;
}

function buildCacheControl(options: CacheControlOptions): string {
  const parts: string[] = [];
  
  if (options.public) parts.push('public');
  if (options.private) parts.push('private');
  if (options.noCache) parts.push('no-cache');
  if (options.noStore) parts.push('no-store');
  if (options.mustRevalidate) parts.push('must-revalidate');
  if (options.proxyRevalidate) parts.push('proxy-revalidate');
  
  if (options.maxAge !== undefined) {
    parts.push(`max-age=${options.maxAge}`);
  }
  if (options.sMaxAge !== undefined) {
    parts.push(`s-maxage=${options.sMaxAge}`);
  }
  if (options.staleWhileRevalidate !== undefined) {
    parts.push(`stale-while-revalidate=${options.staleWhileRevalidate}`);
  }
  if (options.staleIfError !== undefined) {
    parts.push(`stale-if-error=${options.staleIfError}`);
  }
  
  return parts.join(', ');
}

// ETag 生成
function generateETag(data: unknown): string {
  const content = typeof data === 'string' ? data : JSON.stringify(data);
  const hash = crypto.createHash('md5').update(content).digest('hex');
  return `"${hash}"`;
}

// 条件请求处理
function handleConditionalRequest(
  request: Request,
  responseData: unknown,
  lastModified?: Date
): Response {
  const etag = generateETag(responseData);
  const ifNoneMatch = request.headers.get('If-None-Match');
  const ifModifiedSince = request.headers.get('If-Modified-Since');
  
  // ETag 匹配检查
  if (ifNoneMatch && ifNoneMatch === etag) {
    return new Response(null, { status: 304 });
  }
  
  // Last-Modified 检查
  if (ifModifiedSince && lastModified) {
    const modifiedSince = new Date(ifModifiedSince);
    if (modifiedSince >= lastModified) {
      return new Response(null, { status: 304 });
    }
  }
  
  // 返回完整响应
  return new Response(JSON.stringify(responseData), {
    headers: {
      'Content-Type': 'application/json',
      'ETag': etag,
      'Last-Modified': lastModified?.toUTCString(),
      'Cache-Control': 'private, max-age=300',
    },
  });
}
```

### 数据库查询优化

```typescript
// N+1 问题解决方案
class UserService {
  constructor(private prisma: PrismaClient) {}
  
  // 方案1：使用 Include 预加载
  async getUsersWithPosts(userIds: string[]): Promise<UserWithPosts[]> {
    return this.prisma.user.findMany({
      where: { id: { in: userIds } },
      include: {
        posts: {
          where: { published: true },
          orderBy: { createdAt: 'desc' },
          take: 10,
        },
      },
    });
  }
  
  // 方案2：使用 DataLoader 批量加载
  createDataLoader() {
    return new DataLoader<string, Post[]>(async (userIds) => {
      const posts = await this.prisma.post.findMany({
        where: { authorId: { in: userIds as string[] } },
        orderBy: { createdAt: 'desc' },
      });
      
      // 按 authorId 分组
      const grouped = new Map<string, Post[]>();
      for (const post of posts) {
        if (!grouped.has(post.authorId)) {
          grouped.set(post.authorId, []);
        }
        grouped.get(post.authorId)!.push(post);
      }
      
      // 按原始顺序返回
      return userIds.map(id => grouped.get(id) || []);
    });
  }
  
  // 方案3：使用 Prisma Select 减少字段
  async getUserSummary(userId: string): Promise<UserSummary> {
    return this.prisma.user.findUnique({
      where: { id: userId },
      select: {
        id: true,
        name: true,
        email: true,
        _count: {
          select: {
            posts: true,
            followers: true,
            following: true,
          },
        },
      },
    });
  }
  
  // 方案4：分页优化
  async getPostsPaginated(cursor?: string, limit = 20) {
    return this.prisma.post.findMany({
      take: limit + 1, // 多取一条判断是否有更多
      ...(cursor && {
        cursor: { id: cursor },
        skip: 1,
      }),
      orderBy: { createdAt: 'desc' },
      include: {
        author: {
          select: { id: true, name: true, avatar: true },
        },
      },
    });
  }
}

// 查询性能监控
class QueryProfiler {
  private queries: QueryRecord[] = [];
  
  async trackQuery<T>(
    name: string,
    queryFn: () => Promise<T>
  ): Promise<T> {
    const start = performance.now();
    
    try {
      const result = await queryFn();
      const duration = performance.now() - start;
      
      this.queries.push({
        name,
        duration,
        status: 'success',
        timestamp: Date.now(),
      });
      
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      
      this.queries.push({
        name,
        duration,
        status: 'error',
        error: error instanceof Error ? error.message : 'Unknown error',
        timestamp: Date.now(),
      });
      
      throw error;
    }
  }
  
  getSlowQueries(threshold = 100): QueryRecord[] {
    return this.queries
      .filter(q => q.duration > threshold)
      .sort((a, b) => b.duration - a.duration);
  }
  
  getStats(): QueryStats {
    return {
      total: this.queries.length,
      avgDuration: this.queries.reduce((sum, q) => sum + q.duration, 0) / this.queries.length,
      p50: this.percentile(50),
      p95: this.percentile(95),
      p99: this.percentile(99),
    };
  }
}
```

### 响应压缩与优化

```typescript
// 响应压缩中间件
import compression from 'compression';
import zlib from 'zlib';

class ResponseCompressor {
  private acceptsEncoding(acceptEncoding: string | null, encoding: string): boolean {
    if (!acceptEncoding) return false;
    return acceptEncoding.includes(encoding);
  }
  
  async compressResponse(
    data: unknown,
    acceptEncoding: string | null,
    options: { threshold?: number } = {}
  ): Promise<{ body: Buffer; encoding: string; compressed: boolean }> {
    const json = typeof data === 'string' ? data : JSON.stringify(data);
    const buffer = Buffer.from(json);
    
    // 小于阈值不压缩
    if (buffer.length < (options.threshold || 1024)) {
      return { body: buffer, encoding: 'none', compressed: false };
    }
    
    // 优先使用 Brotli，然后 gzip
    if (this.acceptsEncoding(acceptEncoding, 'br')) {
      return {
        body: await this.brotliCompress(buffer),
        encoding: 'br',
        compressed: true,
      };
    }
    
    if (this.acceptsEncoding(acceptEncoding, 'gzip')) {
      return {
        body: await this.gzipCompress(buffer),
        encoding: 'gzip',
        compressed: true,
      };
    }
    
    return { body: buffer, encoding: 'none', compressed: false };
  }
  
  private brotliCompress(buffer: Buffer): Promise<Buffer> {
    return new Promise((resolve, reject) => {
      zlib.brotliCompress(buffer, { quality: 4 }, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  }
  
  private gzipCompress(buffer: Buffer): Promise<Buffer> {
    return new Promise((resolve, reject) => {
      zlib.gzip(buffer, { level: 6 }, (err, result) => {
        if (err) reject(err);
        else resolve(result);
      });
    });
  }
}

// 字段选择器
function applyFieldSelection<T extends object, K extends keyof T>(
  data: T,
  fields?: string[]
): Partial<T> | T {
  if (!fields || fields.length === 0) {
    return data;
  }
  
  const result: Partial<T> = {};
  for (const field of fields) {
    if (field in data) {
      (result as any)[field] = (data as any)[field];
    }
  }
  
  return result;
}

// 响应格式化
class ResponseFormatter {
  static success<T>(data: T, meta?: ResponseMeta): APIResponse<T> {
    return {
      success: true,
      data,
      ...(meta && { meta }),
    };
  }
  
  static paginated<T>(
    data: T[],
    pagination: PaginationMeta,
    meta?: ResponseMeta
  ): APIResponse<T[]> {
    return {
      success: true,
      data,
      meta: {
        ...meta,
        pagination,
      },
    };
  }
  
  static error(
    code: string,
    message: string,
    details?: ErrorDetail[]
  ): APIErrorResponse {
    return {
      success: false,
      error: {
        code,
        message,
        ...(details && { details }),
      },
    };
  }
}
```

---

## API 测试策略

### 单元测试与集成测试

```typescript
// API 路由测试示例
describe('User API', () => {
  let app: Express;
  let testUser: TestUser;
  
  beforeAll(async () => {
    app = await createTestApp();
    await seedTestData();
  });
  
  afterAll(async () => {
    await cleanupTestData();
    await closeDatabase();
  });
  
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          email: 'newuser@example.com',
          password: 'SecurePass123',
          name: 'New User',
        })
        .expect(201);
      
      expect(response.body.success).toBe(true);
      expect(response.body.data.email).toBe('newuser@example.com');
      expect(response.body.data).not.toHaveProperty('password');
      
      testUser = response.body.data;
    });
    
    it('should reject duplicate email', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          email: 'newuser@example.com',
          password: 'SecurePass123',
          name: 'Another User',
        })
        .expect(409);
      
      expect(response.body.success).toBe(false);
      expect(response.body.error.code).toBe('EMAIL_EXISTS');
    });
    
    it('should validate password strength', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          email: 'weak@example.com',
          password: '123',
          name: 'Weak User',
        })
        .expect(400);
      
      expect(response.body.error.code).toBe('VALIDATION_ERROR');
      expect(response.body.error.details).toContainEqual(
        expect.objectContaining({ field: 'password' })
      );
    });
  });
  
  describe('GET /api/users/:id', () => {
    it('should return user by id', async () => {
      const response = await request(app)
        .get(`/api/users/${testUser.id}`)
        .set('Authorization', `Bearer ${testUser.token}`)
        .expect(200);
      
      expect(response.body.data.id).toBe(testUser.id);
    });
    
    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/non-existent-id')
        .set('Authorization', `Bearer ${testUser.token}`)
        .expect(404);
    });
  });
});

// Mock 策略
describe('External Service Integration', () => {
  it('should handle payment service errors', async () => {
    // Mock 外部服务
    const paymentService = {
      charge: jest.fn().mockRejectedValue(
        new PaymentError('Card declined')
      ),
    };
    
    const orderService = new OrderService(paymentService as any);
    
    await expect(
      orderService.processPayment('order-123', 100)
    ).rejects.toThrow('Payment processing failed');
  });
  
  it('should retry on transient failures', async () => {
    const mockFn = jest.fn()
      .mockRejectedValueOnce(new Error('Timeout'))
      .mockRejectedValueOnce(new Error('Timeout'))
      .mockResolvedValueOnce({ success: true });
    
    const service = new RetryService(mockFn, { maxRetries: 3 });
    const result = await service.execute();
    
    expect(mockFn).toHaveBeenCalledTimes(3);
    expect(result).toEqual({ success: true });
  });
});
```

### 契约测试

```typescript
// 使用 Pact 进行契约测试
// consumer.test.ts
import { Pact } from '@pact-foundation/pact';

const provider = new Pact({
  consumer: 'UserService',
  provider: 'PaymentService',
  port: 1234,
  logLevel: 'warn',
});

describe('Payment Service Contract', () => {
  beforeAll(() => provider.setup());
  afterEach(() => provider.writePacts());
  afterAll(() => provider.finalize());
  
  describe('creating a payment', () => {
    it('should accept a valid payment request', async () => {
      await provider.addInteraction({
        states: [{ description: 'user has a valid card' }],
        uponReceiving: 'a request for payment',
        withRequest: {
          method: 'POST',
          path: '/payments',
          headers: { 'Content-Type': 'application/json' },
          body: {
            userId: 'user-123',
            amount: 1000,
            currency: 'USD',
            cardId: 'card-456',
          },
        },
        willRespondWith: {
          status: 201,
          body: {
            paymentId: like('pay_abc123'),
            status: 'succeeded',
            amount: 1000,
          },
        },
      });
      
      const response = await fetch('http://localhost:1234/payments', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          userId: 'user-123',
          amount: 1000,
          currency: 'USD',
          cardId: 'card-456',
        }),
      });
      
      expect(response.status).toBe(201);
      const data = await response.json();
      expect(data.paymentId).toBeDefined();
      expect(data.status).toBe('succeeded');
    });
  });
});
```

### 性能测试

```typescript
// 使用 k6 进行负载测试
// load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

// 自定义指标
const errorRate = new Rate('errors');
const responseTime = new Trend('response_time');

export const options = {
  stages: [
    { duration: '2m', target: 100 },   // 预热：2分钟内增加到100用户
    { duration: '5m', target: 100 },   // 稳定：保持100用户5分钟
    { duration: '2m', target: 200 },   // 压力：2分钟内增加到200用户
    { duration: '5m', target: 200 },   // 高负载：保持200用户5分钟
    { duration: '2m', target: 0 },    // 冷却：2分钟内降到0用户
  ],
  thresholds: {
    http_req_duration: ['p(95)<500'],     // 95%请求在500ms内完成
    http_req_failed: ['rate<0.01'],       // 错误率低于1%
    errors: ['rate<0.05'],               // 自定义错误率低于5%
  },
};

export default function () {
  const baseUrl = 'https://api.example.com';
  const token = 'test-token'; // 应该从登录获取
  
  // 测试用户列表接口
  const listStart = Date.now();
  const listResponse = http.get(`${baseUrl}/api/users`, {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
    tags: { name: 'ListUsers' },
  });
  
  responseTime.add(Date.now() - listStart);
  
  const listSuccess = check(listResponse, {
    'list users status is 200': (r) => r.status === 200,
    'list users has data': (r) => r.json('data') !== undefined,
  });
  
  errorRate.add(!listSuccess);
  
  // 测试用户详情接口
  const userId = 'user-123';
  const detailResponse = http.get(`${baseUrl}/api/users/${userId}`, {
    headers: {
      'Authorization': `Bearer ${token}`,
    },
    tags: { name: 'GetUser' },
  });
  
  check(detailResponse, {
    'get user status is 200': (r) => r.status === 200,
    'get user has id': (r) => r.json('data.id') === userId,
  });
  
  sleep(1);
}
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| OpenAPI 规范 | https://spec.openapis.org/ |
| Swagger Docs | https://swagger.io/docs/ |
| Stoplight | https://stoplight.io/ |
| Redocly | https://redocly.com/ |
| Mintlify | https://mintlify.com/ |
| MCP 协议 | https://modelcontextprotocol.io |
| JWT 规范 | https://datatracker.ietf.org/doc/html/rfc7519 |
| OAuth 2.0 规范 | https://datatracker.ietf.org/doc/html/rfc6749 |
| RESTful API 最佳实践 | https://restfulapi.net/ |
| API Security Checklist | https://github.com/shieldfy/API-Security-Checklist |

---

*本文档由 [[归愚知识系统]] 自动生成*

---

## API 技术概述与深度定位

### REST API 的设计哲学

REST（Representational State Transfer）并非一种严格的技术规范，而是一组架构原则和约束。理解这些原则对于设计高质量 API 至关重要。

**REST 的六大原则：**

1. **客户端-服务器架构**：客户端和服务器相互独立，通过接口通信
2. **无状态**：每个请求包含理解请求所需的全部信息
3. **可缓存**：响应可标记为可缓存或不可缓存
4. **分层系统**：客户端不知道连接的是终端服务器还是中间层
5. **统一接口**：资源通过 URI 标识，操作通过 HTTP 方法实现
6. **按需代码**（可选）：服务器可向客户端发送可执行代码

```yaml
# RESTful API 设计示例
# 资源命名规范
GET    /users          # 资源集合
GET    /users/{id}    # 单个资源
POST   /users          # 创建资源
PUT    /users/{id}     # 完整更新
PATCH  /users/{id}     # 部分更新
DELETE /users/{id}     # 删除资源

# 嵌套资源（谨慎使用）
GET    /users/{id}/posts           # 获取用户的文章
GET    /users/{id}/posts/{postId}  # 获取用户的某篇文章

# 避免嵌套过深（超过2层考虑扁平化）
# 不好: /orgs/{orgId}/teams/{teamId}/members/{memberId}/roles
# 好:  /members/{memberId}/roles?teamId={teamId}
```

### API 设计中的 HTTP 语义

正确使用 HTTP 方法和状态码是 RESTful API 的核心：

```yaml
# HTTP 方法语义
GET:
  - 幂等（多次请求结果相同）
  - 安全（不修改服务器状态）
  - 可缓存
  - 用于：查询资源

POST:
  - 非幂等
  - 非安全
  - 不可缓存
  - 用于：创建资源、执行操作

PUT:
  - 幂等
  - 非安全
  - 不可缓存
  - 用于：完整替换资源

PATCH:
  - 非幂等
  - 非安全
  - 用于：部分更新资源

DELETE:
  - 幂等（删除已删除的资源仍返回成功）
  - 非安全
  - 用于：删除资源

# 状态码分类
1xx: Informational  # 信息性
2xx: Success        # 成功
3xx: Redirection    # 重定向
4xx: Client Error   # 客户端错误
5xx: Server Error   # 服务器错误

# 常用状态码
200 OK                    # 成功
201 Created               # 资源创建成功
204 No Content            # 成功但无返回内容（如 DELETE）
400 Bad Request           # 请求格式错误
401 Unauthorized          # 未认证
403 Forbidden             # 无权限
404 Not Found             # 资源不存在
409 Conflict              # 资源冲突（如重复创建）
422 Unprocessable Entity   # 验证失败
429 Too Many Requests     # 请求过于频繁
500 Internal Server Error # 服务器内部错误
503 Service Unavailable    # 服务不可用
```

### GraphQL 与 REST 的对比

现代 API 设计中，GraphQL 作为 REST 的替代方案越来越受欢迎：

```yaml
# REST vs GraphQL 对比
特性:
  数据获取:
    REST: 多个端点，可能导致 over-fetching 或 under-fetching
    GraphQL: 单个端点，精确获取需要的数据
  
  版本控制:
    REST: 通常通过 /v1/, /v2/ 版本化整个 API
    GraphQL: 新增字段即可，无需版本化
  
  缓存:
    REST: HTTP 缓存原生支持
    GraphQL: 需要客户端实现缓存
  
  类型安全:
    REST: 依赖外部工具（如 OpenAPI）
    GraphQL: 内置 Schema 定义语言
  
  学习曲线:
    REST: 简单直观
    GraphQL: 概念较多，查询语言需要学习
  
  适用场景:
    REST: 简单 CRUD、资源导向、公开 API
    GraphQL: 复杂数据关系、移动端优化、多个前端应用
```

```typescript
// GraphQL Schema 定义
const typeDefs = `
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
    followers: [User!]!
    following: [User!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    comments: [Comment!]!
    createdAt: DateTime!
  }

  type Comment {
    id: ID!
    content: String!
    author: User!
    createdAt: DateTime!
  }

  type Query {
    user(id: ID!): User
    users(limit: Int, offset: Int): [User!]!
    post(id: ID!): Post
    postsByAuthor(authorId: ID!): [Post!]!
  }

  type Mutation {
    createUser(input: CreateUserInput!): User!
    updateUser(id: ID!, input: UpdateUserInput!): User!
    deleteUser(id: ID!): Boolean!
    createPost(input: CreatePostInput!): Post!
  }
`;

// GraphQL Resolver 示例
const resolvers = {
  Query: {
    user: (_, { id }, { dataLoaders }) => {
      return dataLoaders.user.load(id);
    },
    users: async (_, { limit = 10, offset = 0 }) => {
      return prisma.user.findMany({
        take: limit,
        skip: offset,
        orderBy: { createdAt: 'desc' },
      });
    },
  },
  
  User: {
    posts: (user, _, { dataLoaders }) => {
      return dataLoaders.postsByUser.load(user.id);
    },
    followers: (user, _, { dataLoaders }) => {
      return dataLoaders.followers.load(user.id);
    },
  },
  
  Mutation: {
    createUser: async (_, { input }) => {
      return prisma.user.create({
        data: {
          name: input.name,
          email: input.email,
        },
      });
    },
  },
};
```

### gRPC 与 REST 的对比

gRPC 是 Google 推出的高性能 RPC 框架，在微服务间通信中表现优异：

```yaml
# gRPC vs REST 对比
特性:
  协议:
    REST: HTTP/1.1 或 HTTP/2
    gRPC: HTTP/2（强制）
  
  序列化:
    REST: JSON（通常）或 XML
    gRPC: Protocol Buffers（更小、更快）
  
  代码生成:
    REST: OpenAPI 可生成多语言客户端
    gRPC: 原生多语言支持，proto 文件生成
  
  浏览器支持:
    REST: 原生支持
    gRPC: 需要 grpc-web 或代理
  
  流式传输:
    REST: 需要额外实现（如 Server-Sent Events）
    gRPC: 原生支持双向流式
  
  适用场景:
    REST: 公开 API、移动端、Web
    gRPC: 内部微服务、低延迟通信、移动端到后端
```

```protobuf
// user.proto
syntax = "proto3";

package user;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc StreamUserUpdates(StreamUserRequest) returns (stream User);
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
  int64 created_at = 4;
}

message GetUserRequest {
  string id = 1;
}

message ListUsersRequest {
  int32 limit = 1;
  int32 offset = 2;
}

message ListUsersResponse {
  repeated User users = 1;
  int32 total = 2;
}

message CreateUserRequest {
  string name = 1;
  string email = 2;
  string password = 3;
}

message StreamUserRequest {
  repeated string user_ids = 1;
}
```

### WebSocket 与 Server-Sent Events

实时通信是现代 API 的重要能力：

```yaml
# WebSocket vs Server-Sent Events (SSE)
特性:
  方向:
    WebSocket: 双向通信
    SSE: 单向（服务器到客户端）
  
  连接:
    WebSocket: 独立协议 (ws://, wss://)
    SSE: 使用 HTTP/HTTPS
  
  重连:
    WebSocket: 需要手动实现
    SSE: 自动重连
  
  二进制数据:
    WebSocket: 原生支持
    SSE: 仅文本
  
  兼容性:
    WebSocket: 几乎所有现代浏览器
    SSE: IE/Edge 不支持（需要 polyfill）
  
  适用场景:
    WebSocket: 游戏、聊天、实时协作
    SSE: 实时更新、通知、进度推送
```

```typescript
// WebSocket 服务端示例
import { WebSocketServer, WebSocket } from 'ws';

const wss = new WebSocketServer({ port: 8080 });

wss.on('connection', (ws: WebSocket) => {
  console.log('Client connected');
  
  // 发送消息
  ws.send(JSON.stringify({ type: 'welcome', message: 'Connected!' }));
  
  // 接收消息
  ws.on('message', (data: Buffer) => {
    const message = JSON.parse(data.toString());
    handleMessage(ws, message);
  });
  
  // 心跳保活
  const interval = setInterval(() => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.ping();
    }
  }, 30000);
  
  ws.on('close', () => {
    clearInterval(interval);
    console.log('Client disconnected');
  });
});

// SSE 服务端示例
function createSSEStream(req: Request): Response {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    start(controller) {
      // 发送事件
      function sendEvent(data: any, event: string = 'message') {
        const message = `event: ${event}\ndata: ${JSON.stringify(data)}\n\n`;
        controller.enqueue(encoder.encode(message));
      }
      
      // 定期发送数据
      const interval = setInterval(() => {
        sendEvent({ time: Date.now() });
      }, 1000);
      
      // 清理
      req.signal.addEventListener('abort', () => {
        clearInterval(interval);
        controller.close();
      });
    },
  });
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'text/event-stream',
      'Cache-Control': 'no-cache',
      'Connection': 'keep-alive',
    },
  });
}
```

---

## API 安装与配置完整指南

### Node.js 环境搭建

```bash
# Node.js 安装（使用 nvm 管理多版本）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20
nvm alias default 20

# 验证安装
node --version  # v20.x.x
npm --version   # 10.x.x

# 创建项目
mkdir my-api && cd my-api
npm init -y

# 安装核心依赖
npm install express cors helmet morgan dotenv
npm install -D typescript @types/node @types/express @types/cors @types/morgan ts-node nodemon

# TypeScript 配置
npx tsc --init
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Express 框架配置

```typescript
// src/index.ts
import express, { Application, Request, Response, NextFunction } from 'express';
import cors from 'cors';
import helmet from 'helmet';
import morgan from 'morgan';
import dotenv from 'dotenv';

// 加载环境变量
dotenv.config();

const app: Application = express();
const PORT = process.env.PORT || 3000;

// 安全中间件
app.use(helmet());

// CORS 配置
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));

// 请求日志
app.use(morgan('combined'));

// 请求体解析
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// 路由
import userRoutes from './routes/user.routes';
import postRoutes from './routes/post.routes';

app.use('/api/users', userRoutes);
app.use('/api/posts', postRoutes);

// 健康检查
app.get('/health', (req: Request, res: Response) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// 404 处理
app.use((req: Request, res: Response) => {
  res.status(404).json({ error: 'Not Found' });
});

// 错误处理
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ 
    error: 'Internal Server Error',
    message: process.env.NODE_ENV === 'development' ? err.message : undefined,
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

export default app;
```

### 环境变量配置

```bash
# .env 文件
NODE_ENV=development
PORT=3000

# 数据库
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-key-change-in-production
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# CORS
ALLOWED_ORIGINS=http://localhost:3000,https://example.com

# 限流
RATE_LIMIT_WINDOW_MS=60000
RATE_LIMIT_MAX_REQUESTS=100

# 日志
LOG_LEVEL=info

# 第三方服务
SENDGRID_API_KEY=your-sendgrid-key
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
```

```typescript
// src/config/env.ts
import dotenv from 'dotenv';
dotenv.config();

export const config = {
  nodeEnv: process.env.NODE_ENV || 'development',
  port: parseInt(process.env.PORT || '3000', 10),
  
  database: {
    url: process.env.DATABASE_URL!,
  },
  
  redis: {
    url: process.env.REDIS_URL || 'redis://localhost:6379',
  },
  
  jwt: {
    secret: process.env.JWT_SECRET!,
    expiresIn: process.env.JWT_EXPIRES_IN || '15m',
    refreshExpiresIn: process.env.JWT_REFRESH_EXPIRES_IN || '7d',
  },
  
  cors: {
    origins: process.env.ALLOWED_ORIGINS?.split(',') || ['*'],
  },
  
  rateLimit: {
    windowMs: parseInt(process.env.RATE_LIMIT_WINDOW_MS || '60000', 10),
    max: parseInt(process.env.RATE_LIMIT_MAX_REQUESTS || '100', 10),
  },
  
  thirdParty: {
    sendgridApiKey: process.env.SENDGRID_API_KEY,
    aws: {
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    },
  },
};

// 验证必需配置
function validateConfig() {
  const required = ['JWT_SECRET', 'DATABASE_URL'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateConfig();
```

---

## API 常用命令与操作

### 开发环境命令

```bash
# 项目初始化
npm init -y
git init

# 开发依赖
npm run dev          # 启动开发服务器（nodemon）
npm run dev:debug    # 启动调试模式

# 生产构建
npm run build        # TypeScript 编译
npm start            # 生产环境启动

# 代码质量
npm run lint         # ESLint 检查
npm run lint:fix     # ESLint 自动修复
npm run format       # Prettier 格式化

# 测试
npm run test         # 运行测试
npm run test:watch   # 监听模式运行测试
npm run test:cov     # 测试覆盖率

# 数据库
npm run db:migrate   # 运行数据库迁移
npm run db:seed      # 填充测试数据
npm run db:reset     # 重置数据库
```

### Docker 部署命令

```bash
# 构建镜像
docker build -t my-api:latest .

# 运行容器
docker run -d \
  --name my-api \
  -p 3000:3000 \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://user:pass@host:5432/db \
  --restart unless-stopped \
  my-api:latest

# Docker Compose
docker-compose up -d
docker-compose down
docker-compose logs -f
docker-compose exec api npm run migrate

# 扩展服务
docker-compose up -d --scale api=3
```

### 数据库操作命令

```bash
# Prisma CLI
npx prisma generate        # 生成 Prisma 客户端
npx prisma db push         # 同步数据库结构
npx prisma db pull         # 从数据库拉取结构
npx prisma migrate dev     # 创建迁移
npx prisma migrate deploy  # 应用迁移
npx prisma db seed         # 填充数据
npx prisma studio          # 启动数据库管理界面

# 数据库查询
npx prisma db execute      # 执行原始 SQL

# Knex CLI
npx knex migrate:latest
npx knex migrate:rollback
npx knex seed:run
```

### API 测试命令

```bash
# 使用 curl 测试 API
# GET 请求
curl -X GET "http://localhost:3000/api/users" \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json"

# POST 请求
curl -X POST "http://localhost:3000/api/users" \
  -H "Content-Type: application/json" \
  -d '{"name": "John", "email": "john@example.com"}'

# 文件上传
curl -X POST "http://localhost:3000/api/upload" \
  -F "file=@/path/to/file.jpg"

# WebSocket 测试
wscat -c ws://localhost:8080

# API 文档生成
npx redocly build-docs openapi.yaml -o dist/index.html
npx @redocly/cli preview-docs
```

---

## API 常见问题与解决方案

### 问题 1：跨域请求失败

```typescript
// 问题：浏览器阻止跨域请求
// 原因：CORS 策略阻止了来自不同域的请求

// 解决方案 1：配置 CORS 中间件
app.use(cors({
  origin: (origin, callback) => {
    const allowed = ['http://localhost:3000', 'https://example.com'];
    if (!origin || allowed.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  credentials: true,
}));

// 解决方案 2：使用代理
// nginx.conf
# location /api/ {
#   proxy_pass http://backend:3000/;
#   proxy_http_version 1.1;
#   proxy_set_header Host $host;
#   proxy_set_header X-Real-IP $remote_addr;
#   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
# }

// 解决方案 3：JSONP（仅 GET，不推荐）
app.get('/api/data', (req, res) => {
  const callback = req.query.callback;
  const data = { message: 'Hello' };
  res.send(`${callback}(${JSON.stringify(data)})`);
});
```

### 问题 2：请求超时

```typescript
// 问题：长时间运行的请求被客户端断开
// 原因：代理服务器或负载均衡器的超时设置

// 解决方案 1：配置服务端超时
const server = app.listen(3000, () => {
  server.timeout = 120000; // 2分钟
  server.keepAliveTimeout = 65000;
  server.headersTimeout = 66000;
});

// 解决方案 2：使用分块响应
app.get('/api/long-task', async (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  
  // 分块发送
  for (let i = 0; i < 10; i++) {
    res.write(`data: Processing step ${i + 1}/10\n\n`);
    await processStep(i);
    await sleep(1000);
  }
  
  res.write('data: Done\n\n');
  res.end();
});

// 解决方案 3：使用后台任务 + 轮询/SSE
app.post('/api/long-task', async (req, res) => {
  const taskId = uuidv4();
  // 将任务加入队列
  await taskQueue.add({ taskId, ...req.body });
  // 立即返回任务 ID
  res.json({ taskId, status: 'processing' });
});

app.get('/api/tasks/:id/status', async (req, res) => {
  const status = await getTaskStatus(req.params.id);
  res.json(status);
});
```

### 问题 3：文件上传大小限制

```typescript
// 问题：上传大文件时被拒绝
// 原因：默认的请求体大小限制

// 解决方案 1：调整 express 限制
app.use(express.json({ limit: '50mb' }));
app.use(express.urlencoded({ extended: true, limit: '50mb' }));

// 解决方案 2：使用 multer 处理文件上传
import multer from 'multer';

const storage = multer.memoryStorage();
const upload = multer({ 
  storage,
  limits: {
    fileSize: 100 * 1024 * 1024, // 100MB
    files: 5,
  },
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp', 'application/pdf'];
    if (allowed.includes(file.mimetype)) {
      cb(null, true);
    } else {
      cb(new Error('Invalid file type'));
    }
  },
});

app.post('/api/upload', upload.single('file'), (req, res) => {
  // 处理上传的文件
  const file = req.file;
  // 保存到存储服务
  const url = await saveToS3(file);
  res.json({ url });
});

// 解决方案 3：分片上传
app.post('/api/upload/init', (req, res) => {
  const uploadId = uuidv4();
  res.json({ uploadId });
});

app.post('/api/upload/:id/chunk', upload.single('chunk'), async (req, res) => {
  const { uploadId, chunkIndex } = req.body;
  await saveChunk(uploadId, chunkIndex, req.file.buffer);
  res.json({ success: true });
});

app.post('/api/upload/:id/complete', async (req, res) => {
  const fileUrl = await mergeChunks(req.params.id);
  res.json({ url: fileUrl });
});
```

### 问题 4：内存泄漏

```typescript
// 问题：长时间运行后内存持续增长
// 原因：事件监听器未清理、缓存无限增长、全局变量累积

// 解决方案 1：避免在全局存储大量数据
// 不好
const cache = {};
function getData(key) {
  if (!cache[key]) {
    cache[key] = fetchData(key);
  }
  return cache[key];
}

// 好：使用 LRU 缓存
import { LRUCache } from 'lru-cache';
const cache = new LRUCache({ max: 500, ttl: 1000 * 60 * 5 });

// 解决方案 2：清理事件监听器
const server = http.createServer(app);

// 处理连接
server.on('connection', (socket) => {
  socket.on('data', handleData);
  socket.on('close', () => {
    socket.removeAllListeners('data');
  });
});

// 解决方案 3：定期垃圾回收提示
import { prometheusGC } from 'prometheus-gc-stat';
prometheusGC.startCollecting();

// 监控内存
app.get('/metrics', async (req, res) => {
  const memUsage = process.memoryUsage();
  res.json({
    heapUsed: Math.round(memUsage.heapUsed / 1024 / 1024) + 'MB',
    heapTotal: Math.round(memUsage.heapTotal / 1024 / 1024) + 'MB',
    rss: Math.round(memUsage.rss / 1024 / 1024) + 'MB',
  });
});
```

---

## API 实战项目完整示例

### 项目：用户认证系统

```typescript
// src/app.ts - 完整应用示例
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import rateLimit from 'express-rate-limit';
import { z } from 'zod';
import { PrismaClient } from '@prisma/client';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import crypto from 'crypto';

const app = express();
const prisma = new PrismaClient();

// 中间件配置
app.use(helmet());
app.use(cors({ origin: process.env.ALLOWED_ORIGINS?.split(',') }));
app.use(express.json());

// 限流
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15分钟
  max: 5, // 5次尝试
  message: { error: 'TOO_MANY_REQUESTS', message: '请求过于频繁，请稍后再试' },
});

const apiLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
});

// 验证 Schema
const registerSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8).regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/),
  name: z.string().min(1).max(100),
});

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string(),
});

// 认证中间件
async function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'UNAUTHORIZED', message: '未提供认证令牌' });
  }
  
  const token = authHeader.slice(7);
  
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'INVALID_TOKEN', message: '无效或已过期的令牌' });
  }
}

// 路由
app.post('/api/auth/register', authLimiter, async (req, res) => {
  try {
    const data = registerSchema.parse(req.body);
    
    // 检查邮箱是否存在
    const existing = await prisma.user.findUnique({ where: { email: data.email } });
    if (existing) {
      return res.status(409).json({ error: 'EMAIL_EXISTS', message: '该邮箱已被注册' });
    }
    
    // 密码哈希
    const passwordHash = await bcrypt.hash(data.password, 12);
    
    // 创建用户
    const user = await prisma.user.create({
      data: { email: data.email, name: data.name, passwordHash },
      select: { id: true, email: true, name: true, createdAt: true },
    });
    
    // 生成令牌
    const accessToken = jwt.sign(
      { sub: user.id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      { sub: user.id, type: 'refresh' },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );
    
    res.status(201).json({ user, accessToken, refreshToken });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ 
        error: 'VALIDATION_ERROR', 
        details: error.errors 
      });
    }
    console.error(error);
    res.status(500).json({ error: 'INTERNAL_ERROR' });
  }
});

app.post('/api/auth/login', authLimiter, async (req, res) => {
  try {
    const { email, password } = loginSchema.parse(req.body);
    
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      return res.status(401).json({ error: 'INVALID_CREDENTIALS' });
    }
    
    const valid = await bcrypt.compare(password, user.passwordHash);
    if (!valid) {
      return res.status(401).json({ error: 'INVALID_CREDENTIALS' });
    }
    
    const accessToken = jwt.sign(
      { sub: user.id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      { sub: user.id, type: 'refresh' },
      process.env.JWT_SECRET!,
      { expiresIn: '7d' }
    );
    
    res.json({ user: { id: user.id, email: user.email, name: user.name }, accessToken, refreshToken });
  } catch (error) {
    res.status(400).json({ error: 'VALIDATION_ERROR' });
  }
});

app.post('/api/auth/refresh', async (req, res) => {
  const { refreshToken } = req.body;
  
  try {
    const payload = jwt.verify(refreshToken, process.env.JWT_SECRET!) as any;
    
    if (payload.type !== 'refresh') {
      return res.status(401).json({ error: 'INVALID_TOKEN' });
    }
    
    const user = await prisma.user.findUnique({ where: { id: payload.sub } });
    if (!user) {
      return res.status(401).json({ error: 'USER_NOT_FOUND' });
    }
    
    const accessToken = jwt.sign(
      { sub: user.id, email: user.email },
      process.env.JWT_SECRET!,
      { expiresIn: '15m' }
    );
    
    res.json({ accessToken });
  } catch {
    res.status(401).json({ error: 'INVALID_TOKEN' });
  }
});

// 受保护的路由
app.get('/api/users/me', authenticate, async (req, res) => {
  const user = await prisma.user.findUnique({
    where: { id: req.user.sub },
    select: { id: true, email: true, name: true, createdAt: true },
  });
  
  if (!user) {
    return res.status(404).json({ error: 'USER_NOT_FOUND' });
  }
  
  res.json({ user });
});

// 启动服务器
app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| OpenAPI 规范 | https://spec.openapis.org/ |
| Swagger Docs | https://swagger.io/docs/ |
| Stoplight | https://stoplight.io/ |
| Redocly | https://redocly.com/ |
| Mintlify | https://mintlify.com/ |
| MCP 协议 | https://modelcontextprotocol.io |
| JWT 规范 | https://datatracker.ietf.org/doc/html/rfc7519 |
| OAuth 2.0 规范 | https://datatracker.ietf.org/doc/html/rfc6749 |
| RESTful API 最佳实践 | https://restfulapi.net/ |
| API Security Checklist | https://github.com/shieldfy/API-Security-Checklist |
| GraphQL | https://graphql.org/ |
| gRPC | https://grpc.io/ |

---

*本文档由 [[归愚知识系统]] 自动生成*
