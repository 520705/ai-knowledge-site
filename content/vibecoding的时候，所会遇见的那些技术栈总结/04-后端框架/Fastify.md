---
title: Fastify 完全指南
date: 2026-04-24
tags:
  - 后端框架
  - Node.js
  - Fastify
  - 性能优化
description: Fastify 是一个高度专注于性能和开发者体验的 Node.js Web 框架，本指南涵盖其核心特性、插件系统、Schema 验证及与主流框架的深度对比。
---

# Fastify 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Fastify 5 新特性、插件系统、Schema 验证、性能优化及在 AI 应用中的实践。

---

## Fastify 概述与核心哲学

Fastify 是一个诞生于 2016 年的 Node.js Web 框架，由 Matteo Collina 和 Tomaso Alosa 联合创建。它的核心理念是将「**极速**」和「**优秀开发者体验**」这两件事同时做到极致。与 Express 的「简单灵活」不同，Fastify 从一开始就瞄准了「高性能」这个方向，并在过去十年里持续迭代，成为企业级 Node.js API 服务的首选框架之一。

在 2026 年的今天，Fastify 已经发布到第 5 代，完全移除了历史包袱，带来了更简洁的 API、更强的 TypeScript 支持和更高效的性能表现。它的设计哲学可以概括为三个关键词：**最小运行时开销**、**插件化架构**和**Schema 优先验证**。

### 为什么选择 Fastify

很多初学者会问：既然 Express 已经足够用，为什么还要学 Fastify？这个问题其实触及了现代后端开发的核心矛盾——**灵活性和结构性的权衡**。Express 的「无结构」既是优势也是劣势。当你构建一个小型项目时，Express 的灵活让你可以快速起步；但当你构建一个需要长期维护的中大型项目时，缺乏约束往往会导致代码腐烂。

Fastify 通过「**插件封装**」和「**Schema 验证**」这两把利剑，在保持灵活性的同时为项目引入了必要的结构性。每一个插件都是一个独立的封装单元，拥有自己的作用域，不会污染全局；Schema 验证则确保了 API 的输入输出是可预测的，这在前后端分离的架构中尤为重要。

另外，Fastify 的性能优势在生产环境中是实实在在的。根据独立的基准测试，Fastify 在相同的硬件条件下，其吞吐量通常是 Express 的 2 到 3 倍。这意味着你可以用更少的服务器资源处理更多的请求，或者在相同的资源下获得更低的延迟。对于需要处理大量并发请求的应用，比如 AI API 代理、实时聊天后端或者高流量的电商平台，这种性能差异会直接转化为成本优势和用户体验优势。

---

## Fastify vs Express：性能对比与心智模型差异

### 性能差异的根源

要理解 Fastify 和 Express 的性能差异，我们需要深入到它们各自的技术实现层面。Express 基于 Node.js 内置的 HTTP 模块构建，它的设计哲学是「最小化」，尽量减少框架本身的抽象层，让开发者直接与 HTTP 模块打交道。这种设计在 2010 年左右是非常合理的，因为那时候 Node.js 的生态系统还不成熟，框架能做越少的事情，开发者就有越大的自由度。

然而，这种设计也带来了性能上的牺牲。Express 的中间件系统基于回调函数构建，每次请求都需要经过多层中间件的「过滤」，每一层中间件都是一次函数调用和上下文切换的开销。当你的应用有 10 层中间件时，即使每层只增加 0.1 毫秒的延迟，累计起来就是 1 毫秒——在每秒处理数万请求的场景下，这是不可忽视的。

Fastify 则采用了完全不同的策略。它的路由系统基于 `find-my-way` 这个高性能路由器构建，使用的是 Radix Tree（基数树）数据结构，查找复杂度是 O(k)，其中 k 是路由的长度，而不是像 Express 那样需要遍历一个中间件数组。更重要的是，Fastify 默认使用 `@fastify/reply-from` 和原生 Promise，避免了回调地狱和 V8 引擎的优化杀手模式。

> [!TIP]
> **什么是 V8 优化杀手？** V8 引擎会「去优化」那些使用了某些特定模式的 JavaScript 代码，比如 try-catch 嵌套过深、使用 arguments.callee、或者函数参数不固定的情况。Express 的中间件模式在某些场景下会触发这些模式，导致 V8 无法对热点代码进行 JIT 优化，从而影响整体性能。Fastify 在设计时就特别考虑了这一点，尽量避免这些模式。

### 心智模型对比

从心智模型的角度来看，Express 的编程模型类似于「**管道**」——请求从一端进入，依次通过各个中间件的处理，最后从另一端输出响应。你可以把 Express 的中间件想象成水管的各个段落，每一段都可以对水流进行某种处理。

```typescript
// Express 的管道模型
const express = require('express');
const app = express();

// 第一个中间件：记录日志
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // 必须调用 next() 才能继续
});

// 第二个中间件：解析 JSON
app.use(express.json());

// 第三个中间件：认证检查
app.use((req, res, next) => {
  if (req.headers.authorization) {
    next();
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
});

// 路由处理
app.post('/api/users', (req, res) => {
  res.json({ user: req.body });
});
```

Fastify 的模型则更接近「**注册-执行**」模式。插件系统不只是中间件的替代品，更是一种代码组织和封装的方式。你可以创建独立的插件，然后在需要的地方「注册」它们。更重要的是，Fastify 的插件拥有**封装作用域**，这意味着插件内部定义的装饰器不会自动泄漏到全局，只有通过 `fastify-plugin` 包装后的插件才能被父作用域访问。

```typescript
// Fastify 的注册-执行模型
const fastify = require('fastify')({ logger: true });

// 注册插件
fastify.register(async function authPlugin(fastify, opts) {
  // 这个装饰器只在 authPlugin 内部可见
  fastify.decorate('authenticate', async (request, reply) => {
    if (!request.headers.authorization) {
      throw new Error('Unauthorized');
    }
  });

  // 添加路由
  fastify.get('/protected', {
    preHandler: [fastify.authenticate],
  }, async (request, reply) => {
    return { message: 'This is protected' };
  });
});

// 外部无法访问 authenticate 装饰器
// fastify.authenticate  // 报错！

fastify.listen({ port: 3000 });
```

### 实际性能数据

下面是一个更直观的性能对比，基于业内公认的工具 ` autocannon ` 进行测试，测试环境为 16 核 CPU、32GB 内存的服务器：

| 指标 | Fastify 5 | Express 4 | Hono 4 | Elysia |
|------|-----------|-----------|--------|--------|
| 简单 JSON 响应（req/s） | 约 35,000 | 约 15,000 | 约 45,000 | 约 55,000 |
| 数据库查询后响应（req/s） | 约 18,000 | 约 9,000 | 约 22,000 | 约 28,000 |
| p99 延迟 | 约 25ms | 约 65ms | 约 20ms | 约 15ms |
| 冷启动时间 | 约 150ms | 约 80ms | 约 40ms | 约 30ms |
| 内存占用（空闲） | 约 50MB | 约 75MB | 约 35MB | 约 30MB |

> [!WARNING]
> 性能测试数据会因测试环境、硬件配置、网络条件和代码实现方式而有很大差异。上述数据仅供参考，实际生产环境中的表现需要通过真实负载测试来评估。不要盲目追求 benchmark 数字，选择框架时还需要考虑团队熟悉度、生态丰富度和长期维护成本。

---

## 插件系统详解

### 插件是 Fastify 的灵魂

如果你学过 JavaScript 的模块系统，你会发现 Fastify 的插件系统和 ES 模块有很多相似之处——它们都鼓励「**单一职责**」和「**封装隔离**」。在 Fastify 中，几乎所有的功能都是通过插件来实现的：CORS 支持是插件、Helmet 安全头是插件、JWT 认证是插件、数据库连接也是插件。这种设计带来的最大好处是：**你的应用可以像搭积木一样构建起来**。

想象一下，你需要在一个项目中同时支持两个不同的数据库——一个是 PostgreSQL 用于存储核心业务数据，另一个是 Redis 用于缓存。如果你用 Express，你需要自己管理这两个连接的初始化和关闭；但在 Fastify 中，你可以分别创建两个插件，然后分别注册它们：

```typescript
const fastify = require('fastify')({ logger: true });

// PostgreSQL 插件
fastify.register(async function pgPlugin(fastify, opts) {
  const { connectionString } = opts;
  
  // 初始化 PostgreSQL 连接
  const client = new pg.Client(connectionString);
  await client.connect();
  
  // 将 client 装饰到 fastify 实例上
  // 使用 fastify-plugin 确保父作用域可以访问
  fastify.decorate('pg', client);
  
  // 清理资源
  fastify.addHook('onClose', async () => {
    await client.end();
  });
}, { connectionString: 'postgresql://localhost/mydb' });

// Redis 插件
fastify.register(async function redisPlugin(fastify, opts) {
  const { url } = opts;
  
  const redis = new Redis(url);
  
  fastify.decorate('redis', redis);
  
  fastify.addHook('onClose', async () => {
    await redis.quit();
  });
}, { url: 'redis://localhost:6379' });

// 路由中同时使用两个数据库
fastify.get('/stats', async (request, reply) => {
  // 从 PostgreSQL 获取用户数
  const { rows } = await fastify.pg.query('SELECT COUNT(*) FROM users');
  const userCount = parseInt(rows[0].count);
  
  // 从 Redis 获取缓存的页面访问量
  const pageViews = await fastify.redis.get('page_views');
  
  return { userCount, pageViews };
});
```

### 封装作用域：插件的隔离机制

封装作用域是 Fastify 插件系统中最重要但也最容易被误解的概念。简单来说，**默认情况下，一个插件内部定义的装饰器、钩子和路由都只在该插件内部可见**，不会影响父作用域或者其他平级的插件。

这个设计初听起来可能会让人困惑，但它解决了一个非常实际的问题：当你使用第三方插件时，你不会担心它会「污染」你的全局命名空间，也不会意外覆盖你已有的装饰器。

```typescript
const fastify = require('fastify')({ logger: true });

// 插件 A：定义了一个 utility 工具
fastify.register(async function pluginA(fastify, opts) {
  fastify.decorate('utility', {
    generateId: () => Math.random().toString(36).substr(2, 9),
  });
  
  fastify.get('/a-test', async (request, reply) => {
    // 插件 A 内部可以访问自己的 utility
    return { id: fastify.utility.generateId() };
  });
});

// 插件 B：也定义了一个 utility 工具（不冲突！）
fastify.register(async function pluginB(fastify, opts) {
  fastify.decorate('utility', {
    generateToken: () => 'token-' + Math.random().toString(36).substr(2, 9),
  });
  
  fastify.get('/b-test', async (request, reply) => {
    // 插件 B 内部可以访问自己的 utility
    return { token: fastify.utility.generateToken() };
  });
});

// 全局路由：无法访问插件的 utility
fastify.get('/global', async (request, reply) => {
  // 下面的代码会报错，因为 utility 未定义
  // return { id: fastify.utility.generateId() };
  return { message: 'Global route has no access to plugin utilities' };
});
```

### 使用 fastify-plugin 打破封装

有时候你确实需要在插件内部定义一些功能，然后让父作用域或者其他插件能够使用它们。这时就需要使用 `fastify-plugin` 来「打破」封装：

```typescript
const fp = require('fastify-plugin');

// 包装后的插件会「可见」给父作用域
const dbPlugin = fp(async function databasePlugin(fastify, opts) {
  const db = await createDatabaseConnection(opts.connectionString);
  
  // 装饰后可以被父作用域访问
  fastify.decorate('db', db);
  
  fastify.addHook('onClose', async () => {
    await db.close();
  });
}, {
  name: 'database',
  fastify: '4.x', // 声明兼容版本
});

async function buildApp() {
  const fastify = require('fastify')({ logger: true });
  
  // 注册数据库插件
  await fastify.register(dbPlugin, {
    connectionString: 'postgresql://localhost/mydb',
  });
  
  // 在根作用域中就可以访问 db 了
  fastify.get('/users', async (request, reply) => {
    const users = await fastify.db.query('SELECT * FROM users');
    return users;
  });
  
  return fastify;
}
```

### 插件的执行顺序

Fastify 中插件按照「**注册顺序**」执行，而不是「**嵌套顺序**」。这意味着如果你先注册了插件 A 再注册插件 B，那么 A 会先于 B 执行。理解这一点对于调试和排查问题非常重要。

```typescript
fastify.register(async function pluginA(fastify) {
  console.log('Plugin A starts');
  
  fastify.register(async function pluginB(fastify) {
    console.log('Plugin B starts');
    fastify.register(async function pluginC(fastify) {
      console.log('Plugin C starts');
    });
    console.log('Plugin C ends');
  });
  
  console.log('Plugin B ends');
});

fastify.ready().then(() => {
  console.log('All plugins loaded');
});

// 输出顺序：
// Plugin A starts
// Plugin B starts
// Plugin C starts
// Plugin C ends
// Plugin B ends
// All plugins loaded
```

---

## Schema 验证与 JSON Schema

### 为什么 Schema 验证如此重要

在一个真实的项目中，API 的输入验证往往是被忽视的环节。很多开发者抱着「**信任客户端**」的心态，不做严格的输入验证，结果在生产环境中遇到各种奇怪的问题：用户提交了负数的价格、前端传了错误的日期格式、恶意用户发送了超长的字符串导致数据库错误。这些问题在早期可能不明显，但随着用户量增长和安全性要求提高，它们会逐渐成为系统的致命弱点。

Fastify 内置的 Schema 验证机制让你可以在路由定义时声明输入的「**形状**」，框架会自动在请求到达处理器之前进行验证。如果验证失败，请求会被拒绝并返回清晰的错误信息，无需你在每个处理器中写重复的验证代码。

### JSON Schema 基础

JSON Schema 是一个描述 JSON 数据结构的语言，它提供了一套标准化的语法来定义对象的属性、类型、格式和约束。Fastify 使用 JSON Schema 来定义请求和响应的验证规则。

```typescript
const fastify = require('fastify')({ logger: true });

// 定义创建用户的 Schema
const createUserSchema = {
  body: {
    type: 'object',
    required: ['email', 'password', 'username'],
    properties: {
      email: { 
        type: 'string', 
        format: 'email',
        description: '用户的邮箱地址',
      },
      password: { 
        type: 'string', 
        minLength: 8,
        maxLength: 128,
        description: '密码至少需要 8 个字符',
      },
      username: { 
        type: 'string', 
        minLength: 3, 
        maxLength: 30,
        pattern: '^[a-zA-Z0-9_]+$',
        description: '用户名只能包含字母、数字和下划线',
      },
      age: { 
        type: 'integer', 
        minimum: 0, 
        maximum: 150,
        description: '用户年龄',
      },
      role: { 
        type: 'string', 
        enum: ['user', 'admin', 'moderator'],
        default: 'user',
        description: '用户角色',
      },
    },
    additionalProperties: false, // 拒绝未知属性
  },
  params: {
    type: 'object',
    required: ['id'],
    properties: {
      id: { 
        type: 'string', 
        pattern: '^[a-f0-9]{24}$', // MongoDB ObjectId 格式
      },
    },
  },
  querystring: {
    type: 'object',
    properties: {
      page: { type: 'integer', minimum: 1, default: 1 },
      limit: { type: 'integer', minimum: 1, maximum: 100, default: 20 },
      sort: { 
        type: 'string', 
        enum: ['name', '-name', 'createdAt', '-createdAt', 'email', '-email'],
        default: '-createdAt',
      },
    },
  },
  response: {
    201: {
      type: 'object',
      properties: {
        id: { type: 'string' },
        email: { type: 'string' },
        username: { type: 'string' },
        role: { type: 'string' },
        createdAt: { type: 'string', format: 'date-time' },
      },
    },
  },
};

fastify.post('/users/:id', {
  schema: createUserSchema,
}, async (request, reply) => {
  // 此时 request.body 已经通过验证
  // 你可以安全地使用它，TypeScript 也能正确推断类型
  const { email, password, username } = request.body;
  
  // 创建用户的逻辑...
  
  return reply.status(201).send({
    id: '507f1f77bcf86cd799439011',
    email,
    username,
    role: 'user',
    createdAt: new Date().toISOString(),
  });
});
```

### Fastify 的序列化优化

Schema 验证不仅用于输入检查，还可以用于输出序列化。Fastify 会在响应时根据你定义的 response schema 自动进行序列化，这意味着你可以精确控制哪些字段会被返回给客户端——这对于安全性和性能都有好处。

```typescript
// 用户列表 Schema：只返回基本字段，不返回敏感信息
const userListSchema = {
  response: {
    200: {
      type: 'object',
      properties: {
        data: {
          type: 'array',
          items: {
            type: 'object',
            properties: {
              id: { type: 'string' },
              username: { type: 'string' },
              email: { type: 'string' },
              role: { type: 'string' },
              createdAt: { type: 'string', format: 'date-time' },
              // 注意：password 和其他敏感字段没有在这里定义
              // Fastify 会自动过滤掉这些字段
            },
          },
        },
        pagination: {
          type: 'object',
          properties: {
            page: { type: 'integer' },
            limit: { type: 'integer' },
            total: { type: 'integer' },
            totalPages: { type: 'integer' },
          },
        },
      },
    },
  },
};
```

> [!WARNING]
> **安全性警告**：Fastify 的 Schema 验证默认不会自动过滤响应中的额外字段。要实现响应过滤，你需要在 `fastify-instantiation` 时设置 `removeAdditional: 'all'`（在 AJV 配置中），或者使用 `@sinclair/typebox` 配合 `@fastify/response-validation`。否则，即使你定义了 response schema，未定义的字段仍然会被包含在响应中。

### TypeBox：类型安全的 Schema 定义

JSON Schema 虽然强大，但它的语法比较冗长，而且和 TypeScript 类型系统是割裂的。你定义了 Schema，还需要再定义一遍 TypeScript 类型。`@sinclair/typebox` 通过提供一套和 TypeScript 类型一一对应的 Schema 生成器，解决了这个问题：

```typescript
const { Type } = require('@sinclair/typebox');
const fastify = require('fastify')();

// TypeBox Schema 和 TypeScript 类型完美对应
const UserSchema = Type.Object({
  id: Type.String(),
  email: Type.String({ format: 'email' }),
  username: Type.String({ minLength: 3, maxLength: 30 }),
  role: Type.Union([
    Type.Literal('user'),
    Type.Literal('admin'),
    Type.Literal('moderator'),
  ]),
  createdAt: Type.String({ format: 'date-time' }),
});

// 创建用户时不需要 id 和 createdAt
const CreateUserSchema = Type.Object({
  email: Type.String({ format: 'email' }),
  username: Type.String({ minLength: 3, maxLength: 30 }),
  password: Type.String({ minLength: 8 }),
  role: Type.Optional(Type.Union([
    Type.Literal('user'),
    Type.Literal('admin'),
    Type.Literal('moderator'),
  ])),
});

// 更新用户时所有字段都是可选的
const UpdateUserSchema = Type.Partial(CreateUserSchema);

fastify.post('/users', {
  schema: {
    body: CreateUserSchema,
    response: {
      201: UserSchema,
    },
  },
}, async (request, reply) => {
  // request.body 的类型会被正确推断为 CreateUserSchema
  // reply.send() 的参数类型会被验证为 UserSchema
  const user = await createUser(request.body);
  return reply.status(201).send(user);
});
```

---

## 生命周期钩子

Fastify 提供了一套完整的请求生命周期钩子，让你可以在请求处理的不同阶段插入自定义逻辑。这些钩子按照执行顺序依次是：

1. **onRequest** — 请求刚到达 Fastify 时
2. **preParsing** — 开始解析请求体之前
3. **preValidation** — 开始验证之前
4. **preHandler** — 开始执行路由处理器之前
5. **handler** — 路由处理器执行
6. **preResponse** — 开始发送响应之前
7. **onResponse** — 响应发送完成后
8. **onSend** — 响应正在发送时
9. **onTimeout** — 请求超时时
10. **onError** — 发生错误时
11. **onClose** — Fastify 实例关闭时

```typescript
const fastify = require('fastify')({ logger: true });

// 全局 onRequest 钩子：所有请求都会经过
fastify.addHook('onRequest', async (request, reply) => {
  console.log('请求到达:', request.method, request.url);
  
  // 你可以在这里添加请求 ID
  request.requestId = crypto.randomUUID();
});

// 全局 onResponse 钩子：记录请求耗时
fastify.addHook('onResponse', async (request, reply) => {
  const duration = reply.elapsedTime;
  console.log(`请求完成: ${request.method} ${request.url} - ${reply.statusCode} - ${duration}ms`);
});

// 全局 onError 钩子：统一错误日志
fastify.addHook('onError', async (request, reply, error) => {
  console.error('请求错误:', {
    url: request.url,
    method: request.method,
    error: error.message,
    stack: error.stack,
    requestId: request.requestId,
  });
});

// 路由级别的 preHandler 钩子
fastify.get('/profile', {
  preHandler: async (request, reply) => {
    // 检查用户是否已登录
    if (!request.headers.authorization) {
      throw { statusCode: 401, message: 'Unauthorized' };
    }
    
    // 解析用户信息并附加到请求对象
    const token = request.headers.authorization.replace('Bearer ', '');
    request.user = await verifyToken(token);
  },
}, async (request, reply) => {
  // 此时 request.user 已经存在
  return { profile: request.user };
});

// 异步钩子：你可以在钩子中执行异步操作
fastify.addHook('preHandler', async (request, reply) => {
  // 模拟数据库查询延迟
  await new Promise(resolve => setTimeout(resolve, 10));
  
  // 如果某个条件不满足，可以抛出错误终止请求
  if (request.headers['x-force-error']) {
    throw new Error('Force error header detected');
  }
});
```

> [!TIP]
> **钩子的执行顺序很重要**。如果你在一个全局钩子中设置了某些属性，后续的路由处理器都可以访问这些属性。但如果你在 `preHandler` 中调用 `reply.send()` 或者 `reply.raw.end()`，请求的生命周期会提前结束，后续的钩子和处理器都不会执行。

---

## 装饰器与封装

### 装饰器类型

Fastify 提供了三种主要类型的装饰器：

| 装饰器类型 | 作用域 | 用途 |
|-----------|--------|------|
| `fastify.decorate(name, value)` | Fastify 实例 | 添加实例方法或属性 |
| `fastify.decorateRequest(name, value)` | Request 对象 | 添加请求对象的方法或属性 |
| `fastify.decorateReply(name, value)` | Reply 对象 | 添加响应对象的方法或属性 |

```typescript
const fp = require('fastify-plugin');

async function buildApp() {
  const fastify = require('fastify')({ logger: true });

  // 在实例上添加工具方法
  fastify.decorate('config', {
    apiVersion: 'v1',
    environment: process.env.NODE_ENV || 'development',
  });

  // 在请求对象上添加用户信息存储
  fastify.decorateRequest('user', null);
  fastify.decorateRequest('requestId', null);

  // 在响应对象上添加便捷方法
  fastify.decorateReply('sendSuccess', function (data, statusCode = 200) {
    return this.status(statusCode).send({
      success: true,
      data,
      timestamp: new Date().toISOString(),
    });
  });

  fastify.decorateReply('sendError', function (message, statusCode = 500) {
    return this.status(statusCode).send({
      success: false,
      error: message,
      timestamp: new Date().toISOString(),
    });
  });

  // 全局 preHandler：初始化请求信息
  fastify.addHook('preHandler', async (request, reply) => {
    request.requestId = request.headers['x-request-id'] || crypto.randomUUID();
  });

  // 使用装饰器
  fastify.get('/info', async (request, reply) => {
    return reply.sendSuccess({
      version: fastify.config.apiVersion,
      env: fastify.config.environment,
    });
  });

  fastify.get('/test-reply', async (request, reply) => {
    // 使用自定义的 sendSuccess 方法
    return reply.sendSuccess({ message: 'Hello!' }, 201);
  });

  return fastify;
}
```

### 封装与作用域隔离

前面我们提到过插件的封装作用域，装饰器同样受到这个规则的影响。当你在一个插件内部定义装饰器时，这个装饰器只在插件内部可见。这给了你「**命名空间隔离**」的能力，让你可以放心使用简洁的名字，不用担心和全局或其他插件冲突：

```typescript
const fp = require('fastify-plugin');

async function buildApp() {
  const fastify = require('fastify')({ logger: true });

  // 插件 A：有自己的 logger
  fastify.register(async function pluginA(fastify) {
    fastify.decorate('logger', {
      info: (msg) => console.log('[PluginA]', msg),
      error: (msg) => console.error('[PluginA ERROR]', msg),
    });

    fastify.get('/a', async (request, reply) => {
      fastify.logger.info('Route A called');
      return reply.sendSuccess({ source: 'pluginA' });
    });
  });

  // 插件 B：也有自己的 logger（不冲突！）
  fastify.register(async function pluginB(fastify) {
    fastify.decorate('logger', {
      info: (msg) => console.log('[PluginB]', msg),
      error: (msg) => console.error('[PluginB ERROR]', msg),
    });

    fastify.get('/b', async (request, reply) => {
      fastify.logger.info('Route B called');
      return reply.sendSuccess({ source: 'pluginB' });
    });
  });

  // 根路由：有自己的 logger
  fastify.decorate('logger', {
    info: (msg) => console.log('[ROOT]', msg),
    error: (msg) => console.error('[ROOT ERROR]', msg),
  });

  fastify.get('/root', async (request, reply) => {
    fastify.logger.info('Root route called');
    return reply.sendSuccess({ source: 'root' });
  });

  return fastify;
}
```

---

## Fastify + Prisma/Drizzle 集成

### Prisma 集成

Prisma 是当前 Node.js 生态中最流行的 ORM 之一，它提供了类型安全的数据库访问、自动迁移和直观的查询 API。Fastify 和 Prisma 的结合是构建类型安全 API 的经典组合。

```typescript
// src/plugins/prisma.ts
const fp = require('fastify-plugin');
const { PrismaClient } = require('@prisma/client');

async function prismaPlugin(fastify, opts) {
  const prisma = new PrismaClient({
    log: process.env.NODE_ENV === 'development' 
      ? ['query', 'info', 'warn', 'error']
      : ['error'],
  });

  // 测试连接
  try {
    await prisma.$connect();
    fastify.log.info('Database connected successfully');
  } catch (error) {
    fastify.log.error('Database connection failed:', error);
    throw error;
  }

  // 装饰到 fastify 实例
  fastify.decorate('prisma', prisma);

  // 应用关闭时断开连接
  fastify.addHook('onClose', async () => {
    await prisma.$disconnect();
    fastify.log.info('Database disconnected');
  });
}

module.exports = fp(prismaPlugin, {
  name: 'prisma',
  fastify: '4.x',
});
```

```typescript
// src/routes/users.ts
const fp = require('fastify-plugin');

async function userRoutes(fastify, opts) {
  // 获取用户列表（分页）
  fastify.get('/', {
    schema: {
      querystring: {
        type: 'object',
        properties: {
          page: { type: 'integer', minimum: 1, default: 1 },
          limit: { type: 'integer', minimum: 1, maximum: 100, default: 20 },
          search: { type: 'string' },
        },
      },
    },
  }, async (request, reply) => {
    const { page = 1, limit = 20, search } = request.query;
    const skip = (page - 1) * limit;

    const where = search ? {
      OR: [
        { email: { contains: search, mode: 'insensitive' } },
        { username: { contains: search, mode: 'insensitive' } },
      ],
    } : {};

    const [users, total] = await Promise.all([
      fastify.prisma.user.findMany({
        where,
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
        select: {
          id: true,
          email: true,
          username: true,
          role: true,
          createdAt: true,
        },
      }),
      fastify.prisma.user.count({ where }),
    ]);

    return {
      data: users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  });

  // 获取单个用户
  fastify.get('/:id', async (request, reply) => {
    const { id } = request.params;
    
    const user = await fastify.prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        username: true,
        role: true,
        createdAt: true,
      },
    });

    if (!user) {
      return reply.status(404).send({ error: 'User not found' });
    }

    return { user };
  });

  // 创建用户
  fastify.post('/', async (request, reply) => {
    const { email, username, password } = request.body;

    // 唯一性检查
    const existing = await fastify.prisma.user.findFirst({
      where: {
        OR: [{ email }, { username }],
      },
    });

    if (existing) {
      return reply.status(409).send({
        error: 'User already exists',
        field: existing.email === email ? 'email' : 'username',
      });
    }

    const bcrypt = await import('bcryptjs');
    const hashedPassword = await bcrypt.hash(password, 12);

    const user = await fastify.prisma.user.create({
      data: {
        email,
        username,
        password: hashedPassword,
      },
      select: {
        id: true,
        email: true,
        username: true,
        role: true,
        createdAt: true,
      },
    });

    return reply.status(201).send({ user });
  });

  // 删除用户
  fastify.delete('/:id', async (request, reply) => {
    const { id } = request.params;

    try {
      await fastify.prisma.user.delete({ where: { id } });
      return reply.status(204).send();
    } catch (error) {
      if (error.code === 'P2025') {
        return reply.status(404).send({ error: 'User not found' });
      }
      throw error;
    }
  });
}

module.exports = fp(userRoutes, {
  name: 'user-routes',
});
```

### Drizzle ORM 集成

Drizzle 是近年来崛起的新一代 ORM，相比 Prisma 它的 SQL 味道更重，对于喜欢手写 SQL 或者需要细粒度控制的开发者来说更加友好。Drizzle 的设计理念是「**给你 SQL 的能力，同时保持类型安全**」。

```typescript
// src/db/schema.ts
const { pgTable, uuid, varchar, text, boolean, timestamp } = require('drizzle-orm/pg-core');

const users = pgTable('users', {
  id: uuid('id').primaryKey().defaultRandom(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  username: varchar('username', { length: 50 }).notNull().unique(),
  password: varchar('password', { length: 255 }).notNull(),
  firstName: varchar('first_name', { length: 100 }),
  lastName: varchar('last_name', { length: 100 }),
  avatar: text('avatar'),
  role: varchar('role', { length: 20 }).default('USER').notNull(),
  isActive: boolean('is_active').default(true).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

const posts = pgTable('posts', {
  id: uuid('id').primaryKey().defaultRandom(),
  title: varchar('title', { length: 255 }).notNull(),
  slug: varchar('slug', { length: 255 }).notNull().unique(),
  content: text('content').notNull(),
  published: boolean('published').default(false).notNull(),
  authorId: uuid('author_id').references(() => users.id).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

module.exports = { users, posts };
```

```typescript
// src/db/index.ts
const { drizzle } = require('drizzle-orm/node-postgres');
const { Pool } = require('pg');
const schema = require('./schema');

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

const db = drizzle(pool, { schema });

module.exports = { db };
```

---

## Fastify + TypeScript

Fastify 从一开始就将 TypeScript 支持作为核心目标。它的类型定义非常完善，在很多场景下你甚至不需要查看文档，IDE 的自动补全就能告诉你有哪些可用的选项。

### 基础类型定义

```typescript
// src/app.ts
import Fastify, { FastifyInstance, FastifyRequest, FastifyReply } from 'fastify';
import cors from '@fastify/cors';
import helmet from '@fastify/helmet';
import rateLimit from '@fastify/rate-limit';

interface AppOptions {
  logger?: boolean;
}

export interface CreateUserBody {
  email: string;
  username: string;
  password: string;
  firstName?: string;
  lastName?: string;
}

export interface UserParams {
  id: string;
}

export interface UserQuery {
  page?: number;
  limit?: number;
  search?: string;
}

export async function buildApp(options: AppOptions = {}): Promise<FastifyInstance> {
  const fastify = Fastify({
    logger: options.logger ?? true,
    ajv: {
      customOptions: {
        removeAdditional: 'all',       // 移除额外字段
        coerceTypes: true,              // 自动类型转换
        useDefaults: true,               // 使用默认值
        allErrors: true,                 // 返回所有错误
      },
    },
  });

  // 注册核心插件
  await fastify.register(cors, { 
    origin: process.env.ALLOWED_ORIGINS?.split(',') || true,
    credentials: true,
  });

  await fastify.register(helmet, {
    contentSecurityPolicy: false, // 关闭 CSP，方便开发
  });

  await fastify.register(rateLimit, {
    max: 100,
    timeWindow: '1 minute',
  });

  // 健康检查
  fastify.get('/health', async () => ({
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
  }));

  return fastify;
}
```

```typescript
// src/server.ts
import { buildApp } from './app';
import { userRoutes } from './routes/users';

async function main() {
  const app = await buildApp({ logger: true });

  // 注册路由
  await app.register(userRoutes, { prefix: '/api/users' });

  // 启动服务器
  try {
    await app.listen({ port: 3000, host: '0.0.0.0' });
    console.log('Server running at http://localhost:3000');
  } catch (err) {
    app.log.error(err);
    process.exit(1);
  }
}

main();
```

---

## 实际项目结构推荐

一个生产级别的 Fastify 项目应该有清晰的结构：

```
my-fastify-api/
├── src/
│   ├── app.ts               # 应用构建和插件注册
│   ├── server.ts            # 服务器入口
│   ├── plugins/             # 插件目录
│   │   ├── prisma.ts        # 数据库插件
│   │   ├── cors.ts         # CORS 配置
│   │   ├── helmet.ts        # 安全头配置
│   │   └── swagger.ts       # API 文档
│   ├── routes/              # 路由目录
│   │   ├── users.ts         # 用户路由
│   │   ├── posts.ts         # 帖子路由
│   │   └── auth.ts          # 认证路由
│   ├── services/            # 业务逻辑层
│   │   ├── user.service.ts
│   │   └── post.service.ts
│   ├── schemas/              # Schema 定义
│   │   ├── user.schema.ts
│   │   └── post.schema.ts
│   ├── types/                # 全局类型声明
│   │   └── index.d.ts
│   └── utils/                # 工具函数
│       ├── errors.ts
│       └── validation.ts
├── prisma/
│   └── schema.prisma
├── tests/
│   ├── unit/
│   └── integration/
├── .env
├── tsconfig.json
└── package.json
```

> [!TIP]
> **服务层的必要性**：在小型项目中，你可能会把业务逻辑直接写在路由处理器中。但随着项目增长，这种做法会导致路由文件变得臃肿且难以测试。将业务逻辑抽取到 Service 层是一个好的实践——路由负责 HTTP 相关的职责（解析请求、验证 Schema、发送响应），Service 层负责业务逻辑（数据库操作、第三方 API 调用等）。

---

## Fastify vs Hono vs Express：全面对比

### 横向对比表格

| 维度 | Fastify | Hono | Express |
|------|---------|------|---------|
| **性能** | 极高（~35K req/s） | 极高（~45K req/s） | 一般（~15K req/s） |
| **Bundle 大小** | ~350KB | ~14KB | ~250KB |
| **冷启动时间** | ~150ms | ~40ms | ~80ms |
| **学习曲线** | 较陡 | 平缓 | 平缓 |
| **TypeScript 支持** | 原生完善 | 原生完善 | 需手动配置 |
| **Schema 验证** | 内置（JSON Schema） | Zod 集成 | 无内置 |
| **中间件模型** | 插件封装 | 函数式中间件 | 函数式中间件 |
| **跨运行时** | 仅 Node.js | 12+ 运行时 | 仅 Node.js |
| **插件生态** | 丰富（官方+社区） | 成长中 | 极丰富 |
| **封装隔离** | 原生支持 | 无 | 无 |
| **JSON 序列化优化** | 支持 | 支持 | 无 |
| **OpenAPI/Swagger** | 官方插件 | 第三方集成 | 第三方集成 |
| **维护活跃度** | 非常活跃 | 非常活跃 | 维护中 |

### 选型建议

**选择 Fastify 如果：**

- 你的应用运行在 Node.js 环境中
- 你需要处理高并发请求（>1000 QPS）
- 你需要强类型安全和完整的 Schema 验证
- 你的团队有 Angular 或 NestJS 背景
- 你需要清晰的代码组织结构和封装隔离

**选择 Hono 如果：**

- 你需要跨多个运行时部署（Cloudflare Workers、Deno、Bun 等）
- 你的项目对冷启动时间敏感（Serverless 场景）
- 你更看重极小的包体积
- 你需要边缘计算能力

**选择 Express 如果：**

- 你需要快速原型开发
- 你的团队对 Express 非常熟悉
- 你需要使用大量现有的 Express 中间件
- 你的项目不需要极致性能

> [!NOTE]
> 这个对比并不是说哪个框架「更好」，而是说每个框架都有自己的最佳适用场景。在真实项目中，选择框架应该基于团队技能、项目需求、长期维护成本等多方面因素综合考量，而不是单纯看 benchmark 数字。

---

## 常见陷阱与最佳实践

### 陷阱一：错误处理遗漏

Express 的一个「特性」是异步错误需要通过 `try-catch` 或 `next(error)` 来传递错误。但很多从 Express 迁移来的开发者会忘记这一点，导致异步错误「吞掉」：

```typescript
// ❌ 错误：异步错误不会被 Fastify 捕获
fastify.get('/users/:id', async (request, reply) => {
  const user = await fastify.prisma.user.findUnique({
    where: { id: request.params.id },
  });
  
  if (!user) {
    throw new Error('User not found'); // 这里抛出的错误需要被捕获
  }
  
  return user;
});

// ✅ 正确：使用 Fastify 的错误机制
fastify.get('/users/:id', async (request, reply) => {
  const user = await fastify.prisma.user.findUnique({
    where: { id: request.params.id },
  });
  
  if (!user) {
    return reply.status(404).send({ error: 'User not found' });
    // 或者 throw fastify.httpErrors.notFound('User not found');
  }
  
  return user;
});
```

### 陷阱二：插件注册顺序

```typescript
// ❌ 错误：路由注册在插件之前，插件内的 onRequest 钩子不会生效
fastify.get('/api/test', async (request, reply) => ({ ok: true }));

fastify.register(async function apiPlugin(fastify) {
  fastify.addHook('onRequest', async (request, reply) => {
    // 这个钩子不会在 /api/test 路由上生效
    console.log('onRequest called');
  });
  
  fastify.get('/api/other', async (request, reply) => ({ ok: true }));
});

// ✅ 正确：确保插件在路由之前注册
fastify.register(async function apiPlugin(fastify) {
  fastify.addHook('onRequest', async (request, reply) => {
    console.log('onRequest called');
  });
  
  fastify.get('/api/test', async (request, reply) => ({ ok: true }));
  fastify.get('/api/other', async (request, reply) => ({ ok: true }));
});
```

### 最佳实践清单

1. **始终定义 Schema** — 即使是内部 API，也应该定义输入输出的 Schema，这能在重构时提供保护。

2. **使用 fastify-plugin** — 公共服务（数据库、日志等）使用 `fp()` 包装，确保全局可访问。

3. **合理使用封装** — 独立功能使用独立插件，享受作用域隔离的好处。

4. **配置日志级别** — 开发环境用 `debug`，生产环境用 `info` 或 `error`，避免日志刷屏。

5. **设置连接池大小** — 数据库和 Redis 连接池的配置要适合你的并发量。

6. **实现健康检查** — 提供 `/health`、`/health/live` 和 `/health/ready` 三个端点，方便容器编排和监控。

7. **使用 Reply.send()** — 尽量使用 `return reply.send(data)` 而不是 `reply.send(data); return;`，前者能更好地利用 Fastify 的异步优化。

8. **避免在插件外使用 async** — 插件的回调函数应该是 async 的，但 `fastify.register()` 的外层不需要是 async。

---

> [!SUCCESS]
> Fastify 以其卓越的性能、内置的 Schema 验证和强大的插件系统，成为构建高并发 API 服务的首选框架。其 TypeScript 原生支持和完善的插件生态使其在企业级应用中具有显著优势。对于需要极致性能和对类型安全有要求的团队，Fastify 是比 Express 更优的选择。结合 Prisma 或 Drizzle ORM，你可以构建类型安全、架构清晰的高性能后端服务。
