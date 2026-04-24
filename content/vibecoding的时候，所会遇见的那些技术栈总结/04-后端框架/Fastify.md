# Fastify 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Fastify 5 新特性、插件系统、Schema 验证、性能优化及在 AI 应用中的实践。

---

## 目录

1. [[#Fastify 概述与市场定位]]
2. [[#性能基准与优势]]
3. [[#Fastify 5 新特性]]
4. [[#插件系统详解]]
5. [[#Schema 验证]]
6. [[#装饰器与依赖注入]]
7. [[#Fastify vs Express vs Elysia 对比]]
8. [[#常用插件生态]]
9. [[#实战场景与选型建议]]

---

## Fastify 概述与市场定位

### Fastify 的设计理念

Fastify 是一个高度专注于**性能**和**开发者体验**的 Node.js Web 框架。其核心理念包括：

- **最小开销**：极低的框架运行时开销
- **插件架构**：一切皆插件，最大化复用
- **Schema 优先**：JSON Schema 驱动的验证和序列化
- **TypeScript 原生**：完整的类型推断支持
- **开发者体验**：详细错误信息、快速反馈

### 核心定位

| 维度 | 说明 |
|------|------|
| **性能优先** | 比 Express 快 2-3 倍 |
| **企业级** | 适用于高并发、低延迟服务 |
| **类型安全** | 原生 TypeScript 支持 |
| **插件生态** | 可组合的插件系统 |
| **标准化** | 遵循现代 Web 标准 |

---

## 性能基准与优势

### 性能对比表

| 框架 | 请求/秒 | 吞吐量 | 延迟 (p99) | 内存 |
|------|---------|--------|-----------|------|
| **Fastify** | ~35,000 | 极高 | ~25ms | ~50MB |
| **Express** | ~15,000 | 中等 | ~65ms | ~75MB |
| **Koa** | ~20,000 | 中等 | ~50ms | ~60MB |
| **Hono** | ~45,000 | 极高 | ~20ms | ~35MB |
| **Elysia** | ~55,000+ | 极高 | ~15ms | ~30MB |

> [!IMPORTANT]
> 性能差异在高并发场景（>1000 QPS）下才明显，大多数应用不会触及这些限制。选择框架时应综合考虑性能、团队熟悉度和生态系统。

### 性能优势来源

| 技术 | 说明 |
|------|------|
| **Prism** | 最快的 JSON Schema 校验器 |
| **find-my-way** | 高性能 HTTP 路由器 |
| **Runtypes** | 可选的运行时类型检查 |
| **Native Promises** | 原生 Promise 优化 |
| **规避 V8 优化杀手** | 避免导致 V8 去优化的模式 |

---

## Fastify 5 新特性

### Fastify 5 主要更新

| 特性 | 说明 | 状态 |
|------|------|------|
| **删除弃用 API** | 移除长期弃用的 API | ✅ |
| **改进的流处理** | 更高效的流式响应 | ✅ |
| **插件锁定** | 改进的插件注册语义 | ✅ |
| **更好的 TypeScript** | 增强的类型推断 | ✅ |
| **诊断通道** | 改进的可观测性 | ✅ |

### 迁移注意事项

```typescript
// Fastify 4 - 旧写法（已废弃）
fastify.route({
  method: 'GET',
  url: '/users',
  schema: { ... },
  handler: async (req, reply) => { ... }
});

// Fastify 5 - 新写法
fastify.get('/users', { schema: { ... } }, async (req, reply) => {
  return reply.send({ users: [] });
});

// 或使用 shorthand
fastify.get('/health', async () => ({ status: 'ok' }));
```

---

## 插件系统详解

### 插件架构哲学

Fastify 的核心理念是「**封装优先**」，插件系统是其最重要的特性：

```
┌─────────────────────────────────────────────────────────┐
│                      Fastify Instance                    │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Plugin A (encapsulated)                         │   │
│  │  - Registers routes / decorators                 │   │
│  │  - Isolated from siblings                        │   │
│  └─────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Plugin B (encapsulated)                         │   │
│  │  - Separate scope                                │   │
│  └─────────────────────────────────────────────────┘   │
│  Global decorators / hooks                              │
└─────────────────────────────────────────────────────────┘
```

### 插件定义

```typescript
// plugins/sensible.js - 创建错误处理装饰器
async function sensiblePlugin(fastify, options) {
  // 添加装饰器
  fastify.decorateReply('error', function (statusCode, message) {
    this.status(statusCode).send({
      statusCode,
      error: message,
      message: message,
    });
  });
  
  fastify.decorateReply('sendError', function (error) {
    this.status(error.statusCode || 500).send({
      statusCode: error.statusCode || 500,
      error: error.name,
      message: error.message,
    });
  });
}

// 导出插件（符合 fastify-plugin 要求）
module.exports = fp(sensiblePlugin);
```

```typescript
// plugins/database.js - 数据库连接
const fp = require('fastify-plugin');
const mongoose = require('mongoose');

async function databasePlugin(fastify, options) {
  const { uri, options: mongoOptions } = options;
  
  try {
    await mongoose.connect(uri, mongoOptions);
    fastify.log.info('Database connected');
    
    // 关闭时断开连接
    fastify.addHook('onClose', async () => {
      await mongoose.disconnect();
    });
  } catch (err) {
    fastify.log.error('Database connection failed:', err);
    throw err;
  }
}

module.exports = fp(databasePlugin, {
  name: 'database',
  fastify: '4.x',
});
```

### 插件封装

```typescript
const fp = require('fastify-plugin');

// fp() 包装使插件可见于父作用域
module.exports = fp(async function myPlugin(fastify, opts) {
  fastify.decorate('utility', {
    generateId: () => Math.random().toString(36).substr(2, 9),
  });
});
```

### 插件依赖与顺序

```typescript
// 方式 1: 回调方式注册
fastify.register(require('./plugins/db'));

// 方式 2: 异步注册
await fastify.register(import('./plugins/db.js'));

// 方式 3: 带选项
await fastify.register(import('./plugins/cache.js'), {
  ttl: 60,
  max: 100,
});

// 方式 4: 条件注册
if (process.env.NODE_ENV === 'production') {
  await fastify.register(import('./plugins/sentry.js'));
}
```

---

## Schema 验证

### JSON Schema 基础

Fastify 使用 JSON Schema 进行请求/响应验证：

```typescript
const createUserSchema = {
  body: {
    type: 'object',
    required: ['email', 'password', 'name'],
    properties: {
      name: { type: 'string', minLength: 1, maxLength: 100 },
      email: { type: 'string', format: 'email' },
      password: { type: 'string', minLength: 8 },
      age: { type: 'integer', minimum: 0, maximum: 150 },
      role: { type: 'string', enum: ['user', 'admin'] },
    },
    additionalProperties: false,
  },
  params: {
    type: 'object',
    properties: {
      id: { type: 'string', pattern: '^[a-f0-9]{24}$' },
    },
    required: ['id'],
  },
  querystring: {
    type: 'object',
    properties: {
      page: { type: 'integer', minimum: 1, default: 1 },
      limit: { type: 'integer', minimum: 1, maximum: 100, default: 20 },
      sort: { type: 'string', enum: ['name', '-name', 'createdAt', '-createdAt'] },
    },
  },
  response: {
    200: {
      type: 'object',
      properties: {
        id: { type: 'string' },
        name: { type: 'string' },
        email: { type: 'string' },
        createdAt: { type: 'string', format: 'date-time' },
      },
    },
  },
};

fastify.post('/users', { schema: createUserSchema }, async (request, reply) => {
  const { body } = request;
  // body 已验证，类型安全
  const user = await User.create(body);
  return reply.status(201).send(user);
});
```

### Fastify-Schema 增强

```typescript
// 使用 @fastify/sensible 获取更好的错误处理
await fastify.register(import('@fastify/sensible'));

// 自定义错误消息
const userSchema = {
  body: {
    type: 'object',
    required: ['email'],
    properties: {
      email: {
        type: 'string',
        format: 'email',
        errorMessage: '无效的邮箱格式',
      },
      password: {
        type: 'string',
        minLength: 8,
        errorMessage: '密码至少需要 8 个字符',
      },
    },
  },
};
```

### TypeBox 集成

```typescript
const { Type } = require('@sinclair/typebox');
const fastify = require('fastify')();

// TypeBox 提供类型安全的 Schema
const UserSchema = Type.Object({
  id: Type.String(),
  name: Type.String({ minLength: 1, maxLength: 100 }),
  email: Type.String({ format: 'email' }),
  role: Type.Union([
    Type.Literal('user'),
    Type.Literal('admin'),
  ]),
});

const CreateUserSchema = Type.Object({
  body: Type.Omit(UserSchema, ['id']),
});

fastify.post('/users', {
  schema: {
    body: CreateUserSchema,
    response: { 200: UserSchema },
  },
}, async (request, reply) => {
  const user = await User.create(request.body);
  return reply.status(201).send(user);
});
```

---

## 装饰器与依赖注入

### 装饰器类型

| 装饰器 | 说明 |
|--------|------|
| `decorate` | 实例方法/属性 |
| `decorateRequest` | Request 对象扩展 |
| `decorateReply` | Reply 对象扩展 |
| `registerDecorator` | 注册时可用 |

### 请求/响应装饰

```typescript
// decorators/auth.js
const fp = require('fastify-plugin');

async function authDecorators(fastify, options) {
  // 在 request 对象上添加方法
  fastify.decorateRequest('user', null);
  fastify.decorateRequest('checkPermission', null);
  
  // 在 reply 对象上添加方法
  fastify.decorateReply('sendError', function (statusCode, message) {
    return this.status(statusCode).send({
      error: message,
    });
  });
  
  // 在请求处理前解析用户
  fastify.addHook('preHandler', async (request, reply) => {
    const token = request.headers.authorization?.replace('Bearer ', '');
    
    if (token) {
      try {
        request.user = await verifyToken(token);
      } catch {
        request.user = null;
      }
    }
  });
}

module.exports = fp(authDecorators, {
  name: 'auth-decorators',
});
```

### 依赖注入

```typescript
// 使用 fastify.inject 进行测试
const userService = {
  async findById(id) {
    return User.findById(id);
  },
};

// 注册服务
fastify.decorate('userService', userService);

// 使用
fastify.get('/users/:id', async (request, reply) => {
  const user = await fastify.userService.findById(request.params.id);
  if (!user) {
    return reply.status(404).send({ error: 'User not found' });
  }
  return user;
});
```

### 封装隔离

```typescript
// 插件中的装饰器默认封装
fastify.register(async function privatePlugin(fastify, opts) {
  // 此装饰器只在这个插件内可见
  fastify.decorate('db', createDatabaseConnection());
  
  fastify.get('/private', async () => {
    return { db: 'accessible' };
  });
});

fastify.get('/public', async () => {
  // 这里无法访问 db 装饰器
  return { db: 'not accessible' };
});
```

---

## Fastify vs Express vs Elysia 对比

### 核心对比表

| 维度 | Fastify | Express | Elysia |
|------|---------|---------|--------|
| **性能** | 极高 | 一般 | 极高 |
| **Bundle 大小** | ~350KB | ~250KB | ~80KB |
| **学习曲线** | 平缓 | 平缓 | 平缓 |
| **TypeScript** | 原生支持 | 需配置 | 原生支持 |
| **Schema 验证** | 内置 | 需手动 | 内置 |
| **中间件模型** | 插件 | 回调 | 装饰器 |
| **生态** | 丰富 | 庞大 | 较小 |
| **维护状态** | 活跃 | 维护 | 活跃 |

### 路由对比

```typescript
// Fastify
const fastify = require('fastify')();

fastify.get('/users/:id', async (req, reply) => {
  const user = await getUser(req.params.id);
  return user;
});

// Express
const express = require('express');
const app = express();

app.get('/users/:id', async (req, res) => {
  const user = await getUser(req.params.id);
  res.json(user);
});

// Elysia
const { Elysia } = require('elysia');

new Elysia()
  .get('/users/:id', async ({ params: { id } }) => {
    return getUser(id);
  })
  .listen(3000);
```

### Schema 验证对比

```typescript
// Fastify - 内置 JSON Schema
fastify.post('/users', {
  schema: {
    body: {
      type: 'object',
      properties: {
        name: { type: 'string' },
        email: { type: 'string', format: 'email' },
      },
      required: ['name', 'email'],
    },
  },
}, async (req, res) => { ... });

// Express - 需手动验证
app.post('/users', async (req, res, next) => {
  const { error } = userSchema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details });
  next();
}, async (req, res) => { ... });
```

### 插件/中间件对比

| 特性 | Fastify | Express |
|------|---------|---------|
| **封装** | 原生支持 | 无（需借助 express-routing） |
| **注册顺序** | 语义化 | 手动控制 |
| **可见性** | 显式（encapsulation） | 隐式（共享状态） |
| **Hooks** | 生命周期 Hooks | 中间件 |

---

## 常用插件生态

### 核心插件

| 插件 | 功能 | 安装 |
|------|------|------|
| **@fastify/cors** | 跨域资源共享 | `npm i @fastify/cors` |
| **@fastify/helmet** | 安全头 | `npm i @fastify/helmet` |
| **@fastify/jwt** | JWT 认证 | `npm i @fastify/jwt` |
| **@fastify/rate-limit** | 速率限制 | `npm i @fastify/rate-limit` |
| **@fastify/swagger** | OpenAPI 文档 | `npm i @fastify/swagger` |
| **@fastify/static** | 静态文件 | `npm i @fastify/static` |
| **@fastify/websocket** | WebSocket | `npm i @fastify/websocket` |
| **@fastify/sensible** | 常用工具 | `npm i @fastify/sensible` |

### 完整配置示例

```typescript
const fastify = require('fastify')({ logger: true });

// CORS 配置
await fastify.register(import('@fastify/cors'), {
  origin: ['https://example.com'],
  credentials: true,
});

// Helmet 安全头
await fastify.register(import('@fastify/helmet'), {
  contentSecurityPolicy: false,
});

// JWT 认证
await fastify.register(import('@fastify/jwt'), {
  secret: process.env.JWT_SECRET,
  sign: { expiresIn: '1d' },
});

// 速率限制
await fastify.register(import('@fastify/rate-limit'), {
  max: 100,
  timeWindow: '1 minute',
});

// Swagger 文档
await fastify.register(import('@fastify/swagger'), {
  openapi: {
    info: { title: 'API', version: '1.0.0' },
  },
});
await fastify.register(import('@fastify/swagger-ui'), {
  routePrefix: '/docs',
});

// 路由
fastify.post('/auth/login', async (req, reply) => {
  const { email, password } = req.body;
  // 验证逻辑...
  const token = fastify.jwt.sign({ sub: user.id });
  return { token };
});

// 保护路由
fastify.get('/protected', {
  onRequest: [fastify.authenticate],
}, async (req, reply) => {
  return { user: req.user };
});
```

### 常用插件表

| 类别 | 插件 | 说明 |
|------|------|------|
| **认证** | @fastify/jwt | JWT 认证 |
| | @fastify/auth | 装饰器认证 |
| | @fastify/session | Session 支持 |
| **验证** | @fastify/multipart | 文件上传 |
| | @fastify/cookie | Cookie 解析 |
| | @fastify/url-data | URL 数据解析 |
| **数据库** | @fastify/mongodb | MongoDB 集成 |
| | @fastify/prisma-client | Prisma ORM |
| | @fastify/knex | Knex 查询构建器 |
| **缓存** | @fastify/redis | Redis 集成 |
| **AI** | @fastify/ai-proxy | AI API 代理 |

---

## 实战场景与选型建议

### AI API 代理实战

```typescript
const fastify = require('fastify')({ logger: true });
const OpenAI = require('openai');
const { z } = require('zod');

await fastify.register(import('@fastify/cors'), {
  origin: '*',
});

await fastify.register(import('@fastify/rate-limit'), {
  max: 30,
  timeWindow: '1 minute',
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const chatSchema = z.object({
  messages: z.array(z.object({
    role: z.enum(['system', 'user', 'assistant']),
    content: z.string(),
  })),
  model: z.enum(['gpt-4o', 'gpt-4o-mini', 'gpt-4-turbo']).optional(),
  temperature: z.number().min(0).max(2).optional().default(0.7),
  max_tokens: z.number().min(1).max(4096).optional(),
});

// 流式聊天完成
fastify.post('/chat/stream', async (request, reply) => {
  const { messages, model, temperature, max_tokens } = chatSchema.parse(request.body);
  
  reply.raw.writeHead(200, {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
  });
  
  const stream = await openai.chat.completions.create({
    model: model || 'gpt-4o',
    messages,
    temperature,
    max_tokens,
    stream: true,
  });
  
  for await (const chunk of stream) {
    const content = chunk.choices[0]?.delta?.content;
    if (content) {
      reply.raw.write(`data: ${JSON.stringify({ content })}\n\n`);
    }
  }
  
  reply.raw.write('data: [DONE]\n\n');
  reply.raw.end();
});

// 非流式聊天完成
fastify.post('/chat', async (request, reply) => {
  const { messages, model, temperature, max_tokens } = chatSchema.parse(request.body);
  
  const completion = await openai.chat.completions.create({
    model: model || 'gpt-4o',
    messages,
    temperature,
    max_tokens,
  });
  
  return completion;
});

// 图像生成
fastify.post('/images', async (request, reply) => {
  const { prompt, n = 1, size = '1024x1024' } = request.body;
  
  const image = await openai.images.generate({
    model: 'dall-e-3',
    prompt,
    n,
    size,
  });
  
  return image;
});

const start = async () => {
  try {
    await fastify.listen({ port: 3000 });
    console.log('Server running at http://localhost:3000');
  } catch (err) {
    fastify.log.error(err);
    process.exit(1);
  }
};

start();
```

### TypeScript 完整配置

```typescript
// src/app.ts
import Fastify, { FastifyInstance, FastifyRequest, FastifyReply } from 'fastify';
import cors from '@fastify/cors';
import helmet from '@fastify/helmet';
import rateLimit from '@fastify/rate-limit';
import swagger from '@fastify/swagger';
import swaggerUi from '@fastify/swagger-ui';

export interface AppOptions {
  logger?: boolean;
}

export async function buildApp(options: AppOptions = {}): Promise<FastifyInstance> {
  const fastify = Fastify({
    logger: options.logger ?? true,
    ajv: {
      customOptions: {
        removeAdditional: 'all',
        coerceTypes: true,
        useDefaults: true,
      },
    },
  });

  // 注册插件
  await fastify.register(cors, { origin: true });
  await fastify.register(helmet);
  await fastify.register(rateLimit, { max: 100, timeWindow: '1 minute' });
  
  // Swagger 文档
  await fastify.register(swagger, {
    openapi: {
      info: { title: 'API', version: '1.0.0' },
      components: {
        securitySchemes: {
          bearerAuth: {
            type: 'http',
            scheme: 'bearer',
          },
        },
      },
    },
  });
  await fastify.register(swaggerUi, { routePrefix: '/docs' });

  // 健康检查
  fastify.get('/health', async () => ({ status: 'ok' }));

  return fastify;
}
```

```typescript
// src/server.ts
import { buildApp } from './app';
import { userRoutes } from './routes/users';

async function main() {
  const app = await buildApp();
  
  // 注册路由
  await app.register(userRoutes, { prefix: '/api/users' });
  
  await app.listen({ port: 3000 });
  console.log('Server running at http://localhost:3000');
}

main();
```

### 成本估算

| 方案 | 月成本（100万请求） | 说明 |
|------|-------------------|------|
| **AWS Lambda + API GW** | ~$5-20 | Serverless |
| **Railway** | $5-50 | 按使用计费 |
| **Render** | $7-50 | 自动扩缩容 |
| **自托管** | $10-50 | Node.js 服务器 |

---

## 完整安装与项目初始化

### 环境要求

```bash
# Node.js 版本要求
node --version  # >= 18.0.0 (推荐 20.x LTS)

# npm 版本
npm --version   # >= 8.0.0
```

### 项目创建

```bash
# 创建项目目录
mkdir my-fastify-api && cd my-fastify-api

# 初始化 npm 项目
npm init -y

# 安装 Fastify 核心依赖
npm install fastify

# 安装 TypeScript 和类型
npm install --save-dev typescript @types/node ts-node

# 初始化 TypeScript 配置
npx tsc --init
```

### TypeScript 配置

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
    "sourceMap": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 项目结构

```
my-fastify-api/
├── src/
│   ├── app.ts              # 应用构建
│   ├── server.ts          # 服务器入口
│   ├── config/
│   │   ├── index.ts       # 配置加载
│   │   ├── database.ts    # 数据库配置
│   │   └── env.ts         # 环境变量
│   ├── plugins/
│   │   ├── cors.ts
│   │   ├── helmet.ts
│   │   ├── rate-limit.ts
│   │   └── swagger.ts
│   ├── routes/
│   │   ├── index.ts       # 路由汇总
│   │   ├── user.routes.ts
│   │   └── post.routes.ts
│   ├── services/
│   │   ├── user.service.ts
│   │   └── post.service.ts
│   ├── schemas/
│   │   ├── user.schema.ts
│   │   └── post.schema.ts
│   ├── types/
│   │   └── index.d.ts     # 全局类型声明
│   └── utils/
│       ├── logger.ts
│       └── errors.ts
├── tests/
│   ├── unit/
│   └── integration/
├── prisma/
│   └── schema.prisma
├── .env
├── .env.example
├── .gitignore
├── tsconfig.json
├── package.json
└── Dockerfile
```

### package.json 配置

```json
{
  "name": "my-fastify-api",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/server.js",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "start:prod": "node dist/server.js",
    "test": "node --experimental-vm-modules node_modules/jest/bin/jest.js",
    "test:watch": "node --experimental-vm-modules node_modules/jest/bin/jest.js --watch",
    "test:coverage": "node --experimental-vm-modules node_modules/jest/bin/jest.js --coverage",
    "lint": "eslint src --ext .ts",
    "lint:fix": "eslint src --ext .ts --fix",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:seed": "tsx prisma/seed.ts"
  },
  "dependencies": {
    "fastify": "^5.0.0",
    "@fastify/cors": "^10.0.0",
    "@fastify/helmet": "^12.0.0",
    "@fastify/rate-limit": "^10.0.0",
    "@fastify/swagger": "^9.0.0",
    "@fastify/swagger-ui": "^4.0.0",
    "@fastify/jwt": "^9.0.0",
    "@fastify/cookie": "^10.0.0",
    "@fastify/static": "^8.0.0",
    "@fastify/multipart": "^9.0.0",
    "@fastify/websocket": "^11.0.0",
    "@prisma/client": "^6.0.0",
    "@fastify/sensible": "^6.0.0",
    "dotenv": "^16.3.1",
    "zod": "^3.22.4",
    "pino": "^8.17.0",
    "pino-pretty": "^10.3.0"
  },
  "devDependencies": {
    "@types/node": "^20.10.0",
    "typescript": "^5.3.2",
    "tsx": "^4.7.0",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.11",
    "ts-jest": "^29.1.1",
    "eslint": "^8.55.0",
    "prisma": "^6.0.0"
  }
}
```

### 环境变量配置

```bash
# .env
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# 数据库
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-key-minimum-32-characters
JWT_EXPIRES_IN=7d

# 第三方服务
OPENAI_API_KEY=sk-...

# CORS
CORS_ORIGIN=http://localhost:3000,https://example.com

# 限流
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW=1 minute
```

### Docker 部署

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./
RUN npm ci

# 复制源码并构建
COPY tsconfig.json ./
COPY prisma ./prisma
COPY src ./src

RUN npm run db:generate
RUN npm run build

# 生产镜像
FROM node:20-alpine AS production

WORKDIR /app

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && adduser -S fastify -u 1001

# 从构建阶段复制
COPY --from=builder --chown=fastify:nodejs /app/dist ./dist
COPY --from=builder --chown=fastify:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=fastify:nodejs /app/prisma ./prisma
COPY --from=builder --chown=fastify:nodejs /app/package*.json ./

# 切换用户
USER fastify

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

---

## 数据库集成

### Prisma ORM 集成

```bash
npm install prisma @prisma/client
npx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  username  String   @unique
  password  String
  firstName String?
  lastName  String?
  avatar    String?
  role      Role     @default(USER)
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  lastLoginAt DateTime?

  posts     Post[]
  comments  Comment[]

  @@index([email])
  @@index([username])
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

model Post {
  id          String    @id @default(uuid())
  title       String
  slug        String    @unique
  content     String
  excerpt     String?
  featuredImage String?
  published   Boolean   @default(false)
  publishedAt DateTime?
  viewCount   Int       @default(0)
  authorId    String
  author      User      @relation(fields: [authorId], references: [id])
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  comments    Comment[]

  @@index([slug])
  @@index([authorId])
  @@index([published])
  @@index([publishedAt])
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  postId    String
  post      Post     @relation(fields: [postId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([authorId])
}
```

### Fastify-Prisma 插件

```typescript
// src/plugins/prisma.ts
import { FastifyPluginAsync } from 'fastify';
import { PrismaClient } from '@prisma/client';

declare module 'fastify' {
  interface FastifyInstance {
    prisma: PrismaClient;
  }
}

declare module '@fastify/jwt' {
  interface FastifyJWT {
    payload: {
      sub: string;
      email: string;
      role: string;
    };
    user: {
      id: string;
      email: string;
      role: string;
    };
  }
}

const prismaPlugin: FastifyPluginAsync = async (fastify) => {
  const prisma = new PrismaClient({
    log: process.env.NODE_ENV === 'development'
      ? ['query', 'info', 'warn', 'error']
      : ['error'],
  });

  // 连接测试
  try {
    await prisma.$connect();
    fastify.log.info('Database connected successfully');
  } catch (error) {
    fastify.log.error('Database connection failed:', error);
    throw error;
  }

  // 注入到 fastify 实例
  fastify.decorate('prisma', prisma);

  // 优雅关闭
  fastify.addHook('onClose', async () => {
    await prisma.$disconnect();
    fastify.log.info('Database disconnected');
  });
};

export default prismaPlugin;
```

### Drizzle ORM 集成

```bash
npm install drizzle-orm pg
npm install --save-dev drizzle-kit @types/pg
```

```typescript
// src/db/schema.ts
import { pgTable, uuid, varchar, text, boolean, timestamp, pgEnum } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

export const roleEnum = pgEnum('role', ['USER', 'ADMIN', 'MODERATOR']);

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  username: varchar('username', { length: 50 }).notNull().unique(),
  password: varchar('password', { length: 255 }).notNull(),
  firstName: varchar('first_name', { length: 100 }),
  lastName: varchar('last_name', { length: 100 }),
  avatar: text('avatar'),
  role: roleEnum('role').default('USER').notNull(),
  isActive: boolean('is_active').default(true).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: uuid('id').defaultRandom().primaryKey(),
  title: varchar('title', { length: 255 }).notNull(),
  slug: varchar('slug', { length: 255 }).notNull().unique(),
  content: text('content').notNull(),
  excerpt: text('excerpt'),
  featuredImage: text('featured_image'),
  published: boolean('published').default(false).notNull(),
  publishedAt: timestamp('published_at'),
  viewCount: timestamp('view_count').default(0).notNull(),
  authorId: uuid('author_id').references(() => users.id).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const comments = pgTable('comments', {
  id: uuid('id').defaultRandom().primaryKey(),
  content: text('content').notNull(),
  authorId: uuid('author_id').references(() => users.id).notNull(),
  postId: uuid('post_id').references(() => posts.id).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// 关系定义
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
  comments: many(comments),
}));

export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
  comments: many(comments),
}));

export const commentsRelations = relations(comments, ({ one }) => ({
  author: one(users, {
    fields: [comments.authorId],
    references: [users.id],
  }),
  post: one(posts, {
    fields: [comments.postId],
    references: [posts.id],
  }),
}));
```

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from './schema';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

export const db = drizzle(pool, { schema });
export type DB = typeof db;
```

---

## 认证授权

### JWT 认证插件

```typescript
// src/plugins/jwt.ts
import { FastifyPluginAsync } from 'fastify';
import fastifyJwt from '@fastify/jwt';
import { prisma } from './prisma';

const jwtPlugin: FastifyPluginAsync = async (fastify) => {
  await fastify.register(fastifyJwt, {
    secret: process.env.JWT_SECRET!,
    sign: {
      expiresIn: process.env.JWT_EXPIRES_IN || '7d',
    },
  });

  // JWT 验证装饰器
  fastify.decorate('authenticate', async (request: any, reply: any) => {
    try {
      await request.jwtVerify();
      
      // 验证用户存在且活跃
      const user = await fastify.prisma.user.findUnique({
        where: { id: request.user.sub, isActive: true },
      });

      if (!user) {
        return reply.code(401).send({
          statusCode: 401,
          error: 'Unauthorized',
          message: 'User not found or inactive',
        });
      }

      request.user = {
        id: user.id,
        email: user.email,
        role: user.role,
      };
    } catch (err) {
      return reply.code(401).send({
        statusCode: 401,
        error: 'Unauthorized',
        message: 'Invalid or expired token',
      });
    }
  });

  // 角色验证装饰器
  fastify.decorate('authorize', (...allowedRoles: string[]) => {
    return async (request: any, reply: any) => {
      if (!allowedRoles.includes(request.user.role)) {
        return reply.code(403).send({
          statusCode: 403,
          error: 'Forbidden',
          message: 'Insufficient permissions',
        });
      }
    };
  });
};

declare module 'fastify' {
  interface FastifyInstance {
    authenticate: (request: any, reply: any) => Promise<void>;
    authorize: (...roles: string[]) => (request: any, reply: any) => Promise<void>;
  }
}

export default jwtPlugin;
```

### 认证路由实现

```typescript
// src/routes/auth.routes.ts
import { FastifyPluginAsync } from 'fastify';
import bcrypt from 'bcryptjs';

interface RegisterBody {
  email: string;
  username: string;
  password: string;
  firstName?: string;
  lastName?: string;
}

interface LoginBody {
  email: string;
  password: string;
}

interface RefreshBody {
  refreshToken: string;
}

const authRoutes: FastifyPluginAsync = async (fastify) => {
  // 注册
  fastify.post<{ Body: RegisterBody }>(
    '/register',
    {
      schema: {
        body: {
          type: 'object',
          required: ['email', 'username', 'password'],
          properties: {
            email: { type: 'string', format: 'email' },
            username: { 
              type: 'string', 
              minLength: 3, 
              maxLength: 30,
              pattern: '^[a-zA-Z0-9_]+$'
            },
            password: { 
              type: 'string', 
              minLength: 8,
              pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)'
            },
            firstName: { type: 'string', maxLength: 100 },
            lastName: { type: 'string', maxLength: 100 },
          },
        },
      },
    },
    async (request, reply) => {
      const { email, username, password, firstName, lastName } = request.body;

      // 检查用户是否存在
      const existingUser = await fastify.prisma.user.findFirst({
        where: {
          OR: [{ email }, { username }],
        },
      });

      if (existingUser) {
        return reply.code(409).send({
          statusCode: 409,
          error: 'Conflict',
          message: existingUser.email === email 
            ? 'Email already registered' 
            : 'Username already taken',
        });
      }

      // 密码哈希
      const hashedPassword = await bcrypt.hash(password, 12);

      // 创建用户
      const user = await fastify.prisma.user.create({
        data: {
          email,
          username,
          password: hashedPassword,
          firstName,
          lastName,
        },
        select: {
          id: true,
          email: true,
          username: true,
          role: true,
          createdAt: true,
        },
      });

      // 生成 Token
      const accessToken = fastify.jwt.sign({
        sub: user.id,
        email: user.email,
        role: user.role,
      });

      const refreshToken = fastify.jwt.sign(
        { sub: user.id, type: 'refresh' },
        { expiresIn: '30d' }
      );

      return reply.code(201).send({
        user,
        accessToken,
        refreshToken,
      });
    }
  );

  // 登录
  fastify.post<{ Body: LoginBody }>(
    '/login',
    {
      schema: {
        body: {
          type: 'object',
          required: ['email', 'password'],
          properties: {
            email: { type: 'string', format: 'email' },
            password: { type: 'string' },
          },
        },
      },
    },
    async (request, reply) => {
      const { email, password } = request.body;

      const user = await fastify.prisma.user.findUnique({
        where: { email },
      });

      if (!user || !user.isActive) {
        return reply.code(401).send({
          statusCode: 401,
          error: 'Unauthorized',
          message: 'Invalid credentials',
        });
      }

      const isValid = await bcrypt.compare(password, user.password);

      if (!isValid) {
        return reply.code(401).send({
          statusCode: 401,
          error: 'Unauthorized',
          message: 'Invalid credentials',
        });
      }

      // 更新最后登录时间
      await fastify.prisma.user.update({
        where: { id: user.id },
        data: { lastLoginAt: new Date() },
      });

      const accessToken = fastify.jwt.sign({
        sub: user.id,
        email: user.email,
        role: user.role,
      });

      const refreshToken = fastify.jwt.sign(
        { sub: user.id, type: 'refresh' },
        { expiresIn: '30d' }
      );

      return {
        user: {
          id: user.id,
          email: user.email,
          username: user.username,
          role: user.role,
        },
        accessToken,
        refreshToken,
      };
    }
  );

  // 刷新 Token
  fastify.post<{ Body: RefreshBody }>(
    '/refresh',
    {
      schema: {
        body: {
          type: 'object',
          required: ['refreshToken'],
          properties: {
            refreshToken: { type: 'string' },
          },
        },
      },
    },
    async (request, reply) => {
      try {
        const decoded = fastify.jwt.verify(request.body.refreshToken) as any;

        if (decoded.type !== 'refresh') {
          return reply.code(401).send({
            statusCode: 401,
            error: 'Unauthorized',
            message: 'Invalid refresh token',
          });
        }

        const user = await fastify.prisma.user.findUnique({
          where: { id: decoded.sub, isActive: true },
        });

        if (!user) {
          return reply.code(401).send({
            statusCode: 401,
            error: 'Unauthorized',
            message: 'User not found',
          });
        }

        const accessToken = fastify.jwt.sign({
          sub: user.id,
          email: user.email,
          role: user.role,
        });

        return { accessToken };
      } catch (err) {
        return reply.code(401).send({
          statusCode: 401,
          error: 'Unauthorized',
          message: 'Invalid or expired refresh token',
        });
      }
    }
  );

  // 获取当前用户
  fastify.get(
    '/me',
    {
      onRequest: [fastify.authenticate],
    },
    async (request, reply) => {
      const user = await fastify.prisma.user.findUnique({
        where: { id: request.user.id },
        select: {
          id: true,
          email: true,
          username: true,
          firstName: true,
          lastName: true,
          avatar: true,
          role: true,
          createdAt: true,
          lastLoginAt: true,
        },
      });

      if (!user) {
        return reply.code(404).send({
          statusCode: 404,
          error: 'Not Found',
          message: 'User not found',
        });
      }

      return user;
    }
  );
};

export default authRoutes;
```

### OAuth 2.0 实现

```typescript
// src/routes/oauth.routes.ts
import { FastifyPluginAsync } from 'fastify';
import crypto from 'crypto';

const oauthRoutes: FastifyPluginAsync = async (fastify) => {
  // Google OAuth
  fastify.get('/google', async (request, reply) => {
    const state = crypto.randomBytes(16).toString('hex');
    
    // 将 state 存储到 session 或缓存
    await fastify.redis.set(`oauth:state:${state}`, 'pending', 'EX', 600);

    const params = new URLSearchParams({
      client_id: process.env.GOOGLE_CLIENT_ID!,
      redirect_uri: `${process.env.API_URL}/oauth/google/callback`,
      response_type: 'code',
      scope: 'email profile',
      state,
      access_type: 'offline',
      prompt: 'consent',
    });

    return reply.redirect(`https://accounts.google.com/o/oauth2/v2/auth?${params}`);
  });

  fastify.get('/google/callback', async (request, reply) => {
    const { code, state, error } = request.query as any;

    if (error) {
      return reply.redirect(`/login?error=${error}`);
    }

    // 验证 state
    const storedState = await fastify.redis.get(`oauth:state:${state}`);
    if (!storedState) {
      return reply.code(400).send({ message: 'Invalid state parameter' });
    }
    await fastify.redis.del(`oauth:state:${state}`);

    // 交换 Access Token
    const tokenResponse = await fetch('https://oauth2.googleapis.com/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        code,
        client_id: process.env.GOOGLE_CLIENT_ID!,
        client_secret: process.env.GOOGLE_CLIENT_SECRET!,
        redirect_uri: `${process.env.API_URL}/oauth/google/callback`,
        grant_type: 'authorization_code',
      }),
    });

    const { access_token } = await tokenResponse.json();

    // 获取用户信息
    const userResponse = await fetch(
      'https://www.googleapis.com/oauth2/v2/userinfo',
      {
        headers: { Authorization: `Bearer ${access_token}` },
      }
    );
    const { email, name, picture } = await userResponse.json();

    // 查找或创建用户
    let user = await fastify.prisma.user.findUnique({ where: { email } });

    if (!user) {
      user = await fastify.prisma.user.create({
        data: {
          email,
          username: email.split('@')[0],
          firstName: name,
          avatar: picture,
          role: 'USER',
          isActive: true,
        },
      });
    }

    // 生成 JWT
    const token = fastify.jwt.sign({
      sub: user.id,
      email: user.email,
      role: user.role,
    });

    return reply.redirect(`/auth/success?token=${token}`);
  });
};

export default oauthRoutes;
```

---

## API 设计最佳实践

### Schema 定义模式

```typescript
// src/schemas/user.schema.ts
import { Type, Static } from '@sinclair/typebox';

// 基础类型定义
export const UserSchema = Type.Object({
  id: Type.String({ format: 'uuid' }),
  email: Type.String({ format: 'email' }),
  username: Type.String({ minLength: 3, maxLength: 30 }),
  firstName: Type.Optional(Type.String()),
  lastName: Type.Optional(Type.String()),
  avatar: Type.Optional(Type.String()),
  role: Type.Union([
    Type.Literal('USER'),
    Type.Literal('ADMIN'),
    Type.Literal('MODERATOR'),
  ]),
  createdAt: Type.String({ format: 'date-time' }),
});

export const CreateUserSchema = Type.Object({
  email: Type.String({ format: 'email' }),
  username: Type.String({ minLength: 3, maxLength: 30, pattern: '^[a-zA-Z0-9_]+$' }),
  password: Type.String({ minLength: 8, pattern: '^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)' }),
  firstName: Type.Optional(Type.String({ maxLength: 100 })),
  lastName: Type.Optional(Type.String({ maxLength: 100 })),
});

export const UpdateUserSchema = Type.Object({
  email: Type.Optional(Type.String({ format: 'email' })),
  username: Type.Optional(Type.String({ minLength: 3, maxLength: 30 })),
  firstName: Type.Optional(Type.String({ maxLength: 100 })),
  lastName: Type.Optional(Type.String({ maxLength: 100 })),
  avatar: Type.Optional(Type.String()),
});

export const UserQuerySchema = Type.Object({
  page: Type.Optional(Type.Integer({ minimum: 1, default: 1 })),
  limit: Type.Optional(Type.Integer({ minimum: 1, maximum: 100, default: 20 })),
  search: Type.Optional(Type.String()),
  role: Type.Optional(Type.Union([
    Type.Literal('USER'),
    Type.Literal('ADMIN'),
    Type.Literal('MODERATOR'),
  ])),
  sort: Type.Optional(Type.Union([
    Type.Literal('createdAt'),
    Type.Literal('email'),
    Type.Literal('username'),
  ])),
  order: Type.Optional(Type.Union([
    Type.Literal('asc'),
    Type.Literal('desc'),
  ])),
});

export const UserParamsSchema = Type.Object({
  id: Type.String({ format: 'uuid' }),
});

// 响应类型
export type User = Static<typeof UserSchema>;
export type CreateUser = Static<typeof CreateUserSchema>;
export type UpdateUser = Static<typeof UpdateUserSchema>;
export type UserQuery = Static<typeof UserQuerySchema>;
export type UserParams = Static<typeof UserParamsSchema>;

// 错误响应 Schema
export const ErrorResponseSchema = Type.Object({
  statusCode: Type.Integer(),
  error: Type.String(),
  message: Type.String(),
});

export const PaginatedResponseSchema = (itemSchema: any) => Type.Object({
  data: Type.Array(itemSchema),
  pagination: Type.Object({
    page: Type.Integer(),
    limit: Type.Integer(),
    total: Type.Integer(),
    pages: Type.Integer(),
  }),
});
```

### 路由实现

```typescript
// src/routes/user.routes.ts
import { FastifyPluginAsync } from 'fastify';
import { TypeBoxTypeCompiler } from '@fastify/type-provider-typebox';
import {
  UserSchema,
  CreateUserSchema,
  UpdateUserSchema,
  UserQuerySchema,
  UserParamsSchema,
  PaginatedResponseSchema,
} from '../schemas/user.schema';

const userRoutes: FastifyPluginAsync = async (fastify) => {
  const compiler = TypeBoxTypeCompiler();

  // 列表查询
  fastify.get(
    '/',
    {
      onRequest: [fastify.authenticate, fastify.authorize('ADMIN')],
      schema: {
        querystring: UserQuerySchema,
        response: {
          200: PaginatedResponseSchema(UserSchema),
        },
      },
    },
    async (request) => {
      const { page = 1, limit = 20, search, role, sort = 'createdAt', order = 'desc' } = 
        request.query;

      const skip = (page - 1) * limit;
      const where: any = {};

      if (search) {
        where.OR = [
          { email: { contains: search, mode: 'insensitive' } },
          { username: { contains: search, mode: 'insensitive' } },
        ];
      }

      if (role) {
        where.role = role;
      }

      const [users, total] = await Promise.all([
        fastify.prisma.user.findMany({
          where,
          select: {
            id: true,
            email: true,
            username: true,
            firstName: true,
            lastName: true,
            avatar: true,
            role: true,
            isActive: true,
            createdAt: true,
          },
          skip,
          take: limit,
          orderBy: { [sort]: order },
        }),
        fastify.prisma.user.count({ where }),
      ]);

      return {
        data: users,
        pagination: {
          page,
          limit,
          total,
          pages: Math.ceil(total / limit),
        },
      };
    }
  );

  // 获取单个用户
  fastify.get<{ Params: { id: string } }>(
    '/:id',
    {
      onRequest: [fastify.authenticate],
      schema: {
        params: UserParamsSchema,
        response: {
          200: UserSchema,
        },
      },
    },
    async (request, reply) => {
      const user = await fastify.prisma.user.findUnique({
        where: { id: request.params.id },
        select: {
          id: true,
          email: true,
          username: true,
          firstName: true,
          lastName: true,
          avatar: true,
          role: true,
          isActive: true,
          createdAt: true,
          updatedAt: true,
        },
      });

      if (!user) {
        return reply.code(404).send({
          statusCode: 404,
          error: 'Not Found',
          message: 'User not found',
        });
      }

      return user;
    }
  );

  // 创建用户
  fastify.post(
    '/',
    {
      onRequest: [fastify.authenticate, fastify.authorize('ADMIN')],
      schema: {
        body: CreateUserSchema,
        response: {
          201: UserSchema,
        },
      },
    },
    async (request, reply) => {
      const { email, username, password, firstName, lastName } = request.body;

      // 检查是否存在
      const existing = await fastify.prisma.user.findFirst({
        where: { OR: [{ email }, { username }] },
      });

      if (existing) {
        return reply.code(409).send({
          statusCode: 409,
          error: 'Conflict',
          message: 'Email or username already exists',
        });
      }

      const bcrypt = await import('bcryptjs');
      const hashedPassword = await bcrypt.hash(password, 12);

      const user = await fastify.prisma.user.create({
        data: {
          email,
          username,
          password: hashedPassword,
          firstName,
          lastName,
        },
        select: {
          id: true,
          email: true,
          username: true,
          firstName: true,
          lastName: true,
          role: true,
          createdAt: true,
        },
      });

      return reply.code(201).send(user);
    }
  );

  // 更新用户
  fastify.patch<{ Params: { id: string } }>(
    '/:id',
    {
      onRequest: [fastify.authenticate],
      schema: {
        params: UserParamsSchema,
        body: UpdateUserSchema,
        response: {
          200: UserSchema,
        },
      },
    },
    async (request, reply) => {
      const { id } = request.params;

      // 权限检查：只能修改自己或管理员可以修改任何人
      if (request.user.role !== 'ADMIN' && request.user.id !== id) {
        return reply.code(403).send({
          statusCode: 403,
          error: 'Forbidden',
          message: 'You can only update your own profile',
        });
      }

      const user = await fastify.prisma.user.update({
        where: { id },
        data: request.body,
        select: {
          id: true,
          email: true,
          username: true,
          firstName: true,
          lastName: true,
          avatar: true,
          role: true,
          updatedAt: true,
        },
      });

      return user;
    }
  );

  // 删除用户
  fastify.delete<{ Params: { id: string } }>(
    '/:id',
    {
      onRequest: [fastify.authenticate, fastify.authorize('ADMIN')],
      schema: {
        params: UserParamsSchema,
      },
    },
    async (request, reply) => {
      const { id } = request.params;

      await fastify.prisma.user.delete({ where: { id } });

      return reply.code(204).send();
    }
  );
};

export default userRoutes;
```

---

## 性能优化

### 请求压缩与响应优化

```typescript
// src/plugins/compression.ts
import { FastifyPluginAsync } from 'fastify';
import compress from '@fastify/compress';

const compressionPlugin: FastifyPluginAsync = async (fastify) => {
  await fastify.register(compress, {
    encodings: ['gzip', 'deflate', 'br'],
    level: 6,
    threshold: 1024,
    clientCert: false,
    clientKey: false,
  });
};

export default compressionPlugin;
```

### 数据库查询优化

```typescript
// src/utils/queryOptimizer.ts

// N+1 查询解决方案：使用 include 或 select
export async function getUsersWithPosts(fastify: any, userIds: string[]) {
  return fastify.prisma.user.findMany({
    where: { id: { in: userIds } },
    include: {
      posts: {
        select: {
          id: true,
          title: true,
          published: true,
        },
        take: 5, // 限制每用户最多5篇文章
      },
      _count: {
        select: { posts: true, comments: true },
      },
    },
  });
}

// 批量操作优化
export async function batchCreateUsers(fastify: any, users: any[]) {
  // 使用事务确保原子性
  return fastify.prisma.$transaction(async (tx: any) => {
    const results = [];
    for (const user of users) {
      const result = await tx.user.create({ data: user });
      results.push(result);
    }
    return results;
  });
}

// 使用 raw 查询优化复杂聚合
export async function getUserStats(fastify: any) {
  const result = await fastify.prisma.$queryRaw`
    SELECT 
      u.id,
      u.email,
      COUNT(p.id) as post_count,
      COUNT(c.id) as comment_count,
      COALESCE(SUM(p.view_count), 0) as total_views
    FROM users u
    LEFT JOIN posts p ON p.author_id = u.id
    LEFT JOIN comments c ON c.author_id = u.id
    WHERE u.is_active = true
    GROUP BY u.id, u.email
    ORDER BY total_views DESC
    LIMIT 100
  `;
  return result;
}
```

### 缓存策略

```typescript
// src/plugins/cache.ts
import { FastifyPluginAsync } from 'fastify';
import fp from 'fastify-plugin';

interface CacheOptions {
  duration: number; // 秒
}

declare module 'fastify' {
  interface FastifyInstance {
    cache: {
      get: <T>(key: string) => Promise<T | null>;
      set: (key: string, value: any, ttl?: number) => Promise<void>;
      del: (key: string) => Promise<void>;
      clear: () => Promise<void>;
    };
  }
}

const cachePlugin: FastifyPluginAsync<{ duration: number }> = async (fastify, options) => {
  const cache = new Map<string, { value: any; expiresAt: number }>();

  const redis = fastify.redis;

  const cacheManager = {
    async get<T>(key: string): Promise<T | null> {
      try {
        const cached = await redis.get(`cache:${key}`);
        if (cached) {
          fastify.log.debug(`Cache HIT: ${key}`);
          return JSON.parse(cached);
        }
        fastify.log.debug(`Cache MISS: ${key}`);
        return null;
      } catch (error) {
        fastify.log.error('Cache get error:', error);
        return null;
      }
    },

    async set(key: string, value: any, ttl: number = options.duration): Promise<void> {
      try {
        await redis.setex(`cache:${key}`, ttl, JSON.stringify(value));
      } catch (error) {
        fastify.log.error('Cache set error:', error);
      }
    },

    async del(key: string): Promise<void> {
      try {
        await redis.del(`cache:${key}`);
      } catch (error) {
        fastify.log.error('Cache delete error:', error);
      }
    },

    async clear(): Promise<void> {
      try {
        const keys = await redis.keys('cache:*');
        if (keys.length > 0) {
          await redis.del(...keys);
        }
      } catch (error) {
        fastify.log.error('Cache clear error:', error);
      }
    },
  };

  fastify.decorate('cache', cacheManager);
};

export default fp(cachePlugin, {
  name: 'cache',
  dependencies: ['redis'],
});
```

---

## 调试与监控

### 请求日志配置

```typescript
// src/plugins/logger.ts
import { FastifyPluginAsync } from 'fastify';
import pino from 'pino';

const loggerPlugin: FastifyPluginAsync = async (fastify) => {
  // 自定义日志级别
  if (process.env.NODE_ENV === 'production') {
    fastify.addHook('onRequest', async (request) => {
      request.startTime = Date.now();
    });

    fastify.addHook('onResponse', async (request, reply) => {
      const duration = Date.now() - (request.startTime || Date.now());
      
      fastify.log.info({
        method: request.method,
        url: request.url,
        status: reply.statusCode,
        duration: `${duration}ms`,
        ip: request.ip,
      });
    });
  }
};

declare module 'fastify' {
  interface FastifyRequest {
    startTime?: number;
  }
}

export default loggerPlugin;
```

### 健康检查端点

```typescript
// src/routes/health.routes.ts
import { FastifyPluginAsync } from 'fastify';

const healthRoutes: FastifyPluginAsync = async (fastify) => {
  // 完整健康检查
  fastify.get('/health', async (request, reply) => {
    const checks = {
      database: { status: 'down' as const },
      redis: { status: 'down' as const },
    };

    // 数据库检查
    try {
      await fastify.prisma.$queryRaw`SELECT 1`;
      checks.database = { status: 'up' };
    } catch (error) {
      fastify.log.error('Database health check failed:', error);
    }

    // Redis 检查
    try {
      await fastify.redis.ping();
      checks.redis = { status: 'up' };
    } catch (error) {
      fastify.log.error('Redis health check failed:', error);
    }

    const allUp = Object.values(checks).every(c => c.status === 'up');
    const allDown = Object.values(checks).every(c => c.status === 'down');

    return {
      status: allUp ? 'healthy' : allDown ? 'unhealthy' : 'degraded',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      checks,
    };
  });

  // Kubernetes 存活探针
  fastify.get('/health/live', async () => ({ status: 'alive' }));

  // Kubernetes 就绪探针
  fastify.get('/health/ready', async (request, reply) => {
    try {
      await fastify.prisma.$queryRaw`SELECT 1`;
      await fastify.redis.ping();
      return { status: 'ready' };
    } catch (error) {
      return reply.code(503).send({ status: 'not ready' });
    }
  });
};

export default healthRoutes;
```

---

## 测试策略

### Jest 配置

```javascript
// jest.config.js
export default {
  testEnvironment: 'node',
  extensionsToTreatAsEsm: ['.ts'],
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1',
  },
  transform: {
    '^.+\\.tsx?$': ['ts-jest', {
      useESM: true,
      tsconfig: {
        module: 'NodeNext',
        moduleResolution: 'NodeNext',
      },
    }],
  },
  testMatch: ['**/__tests__/**/*.test.ts'],
  collectCoverageFrom: ['src/**/*.ts'],
  coverageDirectory: 'coverage',
  setupFilesAfterEnv: ['./tests/setup.ts'],
};
```

### 单元测试示例

```typescript
// tests/unit/user.service.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach, jest } from '@jest/globals';
import Fastify, { FastifyInstance } from 'fastify';
import userRoutes from '../../src/routes/user.routes';
import authPlugin from '../../src/plugins/jwt';
import prismaPlugin from '../../src/plugins/prisma';

describe('User Routes', () => {
  let app: FastifyInstance;

  beforeAll(async () => {
    app = Fastify();
    await app.register(prismaPlugin);
    await app.register(authPlugin);
    await app.register(userRoutes, { prefix: '/users' });
    await app.ready();
  });

  afterAll(async () => {
    await app.close();
  });

  describe('GET /users', () => {
    it('should return 401 without auth token', async () => {
      const response = await app.inject({
        method: 'GET',
        url: '/users',
      });

      expect(response.statusCode).toBe(401);
    });

    it('should return paginated users for admin', async () => {
      // 先登录获取 token
      const loginResponse = await app.inject({
        method: 'POST',
        url: '/auth/login',
        payload: {
          email: 'admin@example.com',
          password: 'Admin123!',
        },
      });

      const { accessToken } = loginResponse.json();

      const response = await app.inject({
        method: 'GET',
        url: '/users',
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });

      expect(response.statusCode).toBe(200);
      const body = response.json();
      expect(body.data).toBeInstanceOf(Array);
      expect(body.pagination).toBeDefined();
    });
  });
});
```

---

## 常见陷阱与最佳实践

### 陷阱 1：不正确使用 Schema 验证

```typescript
// ❌ 错误：不定义 response schema
fastify.post('/users', async (req, reply) => {
  return reply.send({ success: true });
});

// ✅ 正确：定义完整的 schema
fastify.post('/users', {
  schema: {
    body: CreateUserSchema,
    response: {
      201: UserSchema,
      400: ErrorResponseSchema,
    },
  },
}, async (req, reply) => {
  const user = await createUser(req.body);
  return reply.code(201).send(user);
});
```

### 陷阱 2：忘记处理异步错误

```typescript
// ❌ 错误：异步错误不会被 Fastify 捕获
fastify.get('/users/:id', async (req, reply) => {
  const user = await prisma.user.findUnique({ where: { id: req.params.id } });
  if (!user) {
    throw new Error('User not found'); // 需要 await 或返回 reply
  }
  return user;
});

// ✅ 正确：使用 try-catch 或 reply
fastify.get('/users/:id', async (req, reply) => {
  try {
    const user = await prisma.user.findUnique({ where: { id: req.params.id } });
    if (!user) {
      return reply.code(404).send({ message: 'User not found' });
    }
    return user;
  } catch (error) {
    throw error; // 让全局错误处理器处理
  }
});
```

### 陷阱 3：插件注册顺序错误

```typescript
// ❌ 错误：在注册路由后添加钩子
fastify.register(userRoutes);
fastify.addHook('preHandler', authMiddleware); // 不会被路由使用

// ✅ 正确：在插件内部添加钩子
fastify.register(async function (fastify) {
  fastify.addHook('preHandler', authMiddleware);
  fastify.register(userRoutes);
});
```

### 最佳实践清单

1. **总是使用 Schema 验证**：定义请求和响应的 schema，让 Fastify 自动处理验证。
2. **插件封装**：将相关功能封装为插件，保持代码模块化。
3. **使用 `@fastify-plugin`**：确保插件正确共享 Decorators。
4. **正确处理关闭**：在 `onClose` 钩子中清理资源。
5. **使用 TypeBox 或 Zod**：获得完整的类型推断和 IDE 支持。
6. **避免回调地狱**：始终使用 async/await。
7. **合理使用事务**：确保数据一致性。
8. **配置日志级别**：生产环境使用 info，开发环境使用 debug。
9. **使用连接池**：数据库和 Redis 连接池配置正确。
10. **实现健康检查**：让运维工具能够监控服务状态。

---

## 与其他框架对比

### Fastify vs Express

| 特性 | Fastify | Express |
|------|---------|---------|
| **性能** | 极高（~40k req/s） | 中等（~10k req/s） |
| **Schema 验证** | 内置（JSON Schema） | 需要外部库 |
| **日志** | 内置（Pino） | 需要配置 |
| **插件系统** | 原生支持 | 中间件模式 |
| **学习曲线** | 较陡 | 平缓 |
| **生态** | 增长中 | 庞大 |
| **TypeScript** | 原生支持 | 需要配置 |
| **async/await** | 原生支持 | 需要包装器 |

### Fastify vs NestJS

| 特性 | Fastify | NestJS |
|------|---------|--------|
| **架构** | 扁平化 | 模块化（依赖注入） |
| **学习曲线** | 较陡 | 中等 |
| **装饰器** | 可选 | 核心 |
| **GraphQL** | 需要插件 | 内置支持 |
| **微服务** | 需要插件 | 内置支持 |
| **适用场景** | 高性能 API | 企业级应用 |

### Fastify vs Hono

| 特性 | Fastify | Hono |
|------|---------|------|
| **性能** | 极高 | 极高 |
| **插件生态** | 丰富 | 较新 |
| **跨运行时** | Node.js | 多运行时 |
| **适配器** | 较少 | 多种适配器 |
| **适用场景** | 高性能 Node.js | Edge/Serverless |

---

> [!SUCCESS]
> Fastify 以其卓越的性能、内置的 Schema 验证和强大的插件系统，成为构建高并发 API 服务的首选框架。其 TypeScript 原生支持和完善的插件生态使其在企业级应用中具有显著优势。对于需要极致性能和对类型安全有要求的团队，Fastify 是比 Express 更优的选择。结合 OpenAI 等 AI 服务，Fastify 可以构建高性能的 AI 应用后端。
