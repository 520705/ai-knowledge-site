---
title: Hono 完全指南
date: 2026-04-24
tags:
  - 后端框架
  - Edge Computing
  - Cloudflare Workers
  - Hono
  - Bun
description: Hono 是一个轻量、极速的跨运行时 Web 框架，支持 12+ 运行环境，本指南涵盖其设计哲学、中间件体系、边缘计算场景及与主流框架的深度对比。
---

# Hono 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Hono 设计哲学、跨运行时兼容、中间件体系、Zod 验证及与 Express/Fastify 的深度对比。

---

## Hono 概述：火焰之名，跨平台之魂

Hono（日语中「火焰」的意思）是 2023 年由日本开发者 Yusei Murakami 创建的一个轻量级 Web 框架。这个名字恰如其分地传达了框架的核心理念——**用最少的代码燃烧出最快的性能**。在 Hono 的官方介绍中，他们用一句话概括了设计目标：「**Fast, Lightweight, Cross-platform**」。

当我们谈论 Hono 时，首先要理解它解决的核心问题：**JavaScript 生态的碎片化**。在 Hono 诞生之前，如果你想写一个 API 并且同时部署到 Node.js 服务器、Cloudflare Workers 和 Deno Deploy，你通常需要维护多套代码，或者使用条件判断来处理不同运行时的 API 差异。这不仅增加了维护成本，也限制了代码的复用性。

Hono 通过统一的接口层和针对不同运行时的适配器，解决了这个问题。你只需要写一套代码，就可以在 12 种以上的运行环境中运行——而且性能不打折扣。这对于现代的全栈开发和边缘计算场景来说，是革命性的进步。

### 核心理念拆解

Hono 的设计哲学可以拆解为以下几个层面来理解：

**第一，轻量优先。** Hono 的核心包压缩后只有约 14KB，这个数字在 2026 年的今天依然令人印象深刻。这意味着你的应用可以更快地加载、更快地冷启动。在 Serverless 和边缘计算的世界里，冷启动时间直接关系到用户体验和成本消耗，Hono 在这方面的优势非常明显。

**第二，Web 标准。** Hono 遵循标准的 Fetch API，这意味着你写的代码在任何实现了 Fetch API 的环境中都能运行。它不会引入奇怪的抽象层，不会发明自己的 Request/Response 格式——你写的代码就是你部署后运行的代码，不会有意外。

**第三，TypeScript 优先。** Hono 从第一天起就为 TypeScript 设计，提供了完善的类型推断和类型定义。当你定义一个路由时，Hono 能够推断出请求参数的类型、响应类型，甚至能推断出中间件中设置的 Context 变量类型。这种类型安全让你在重构时更有信心。

**第四，中间件驱动。** Hono 的功能扩展几乎完全依赖中间件系统。这种设计保持了核心的简洁性——你只需要加载你需要的中间件，不需要为不需要的功能付出代价。

---

## 跨运行时架构：一次编写，到处运行

### 为什么跨运行时如此重要

在传统的 Web 开发中，「部署环境」往往不是开发者需要考虑的首要问题。你写一个 Express 应用，就部署到 Node.js 服务器上；你写一个 Cloudflare Workers 应用，就用 Workers 特定的 API。这种分工在单体应用时代没有问题，但在 Serverless 和边缘计算时代，问题就出现了：

- 一个功能可能需要同时在多个环境中运行。比如一个 API 既要在中心服务器处理复杂业务逻辑，又要在边缘节点做简单的缓存和重定向。
- 开发者可能需要将现有应用迁移到新的运行环境。比如从 Node.js 迁移到 Cloudflare Workers 以获得更好的性能。
- 框架的锁定效应让你难以利用新技术的优势。比如 Bun 出来了，你却发现自己的框架不支持 Bun。

Hono 的跨运行时架构彻底解决了这些问题。同一套代码，可以不加修改地运行在 Node.js、Bun、Deno、Cloudflare Workers、Fastly Compute、AWS Lambda、Vercel Edge、Azure Functions、Google Cloud Functions、Netlify、Supabase Edge 和 Lagon 等环境中。

### 技术实现原理

Hono 实现跨运行时的方式非常优雅，它通过一个适配器层来处理不同运行时的差异：

```typescript
// Node.js 适配
import { serve } from "@hono/node-server";
import { Hono } from "hono";
import { cors } from "hono/cors";

const app = new Hono();
app.use("*", cors());

app.get("/api/users", (c) => {
  return c.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

// 使用 @hono/node-server 适配
serve({
  fetch: app.fetch,
  port: 3000,
});

console.log("Server running on http://localhost:3000");
```

```typescript
// Cloudflare Workers（部署到边缘）
// 无需额外的适配器导入，直接使用
import { Hono } from "hono";
import { cors } from "hono/cors";

const app = new Hono();
app.use("*", cors());

app.get("/api/users", (c) => {
  return c.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

// Cloudflare Workers 的标准导出格式
export default {
  fetch: app.fetch,
  // 可选：定时触发的 Cron 处理器
  scheduled: async (controller) => {
    console.log("Cron job triggered at", new Date().toISOString());
  },
};
```

```typescript
// Bun（原生支持，最简洁）
import { Hono } from "hono";
import { serve } from "bun";

const app = new Hono();

app.get("/api/users", (c) => {
  return c.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

// Bun 只需要一行 serve
serve({
  fetch: app.fetch,
  port: 3000,
});
```

```typescript
// Deno Deploy
import { serve } from "std/server";
import { Hono } from "hono";
import { cors } from "hono/cors";

const app = new Hono();
app.use("*", cors());

app.get("/api/users", (c) => {
  return c.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

serve(app.fetch, { port: 8000 });
```

> [!TIP]
> **适配器的选择**：对于 Cloudflare Workers、Bun 和 Deno，你不需要安装任何额外的包，它们原生支持 Fetch API。对于 Node.js，你需要安装 `@hono/node-server`。部署到 AWS Lambda 或 Vercel Edge 时，也有对应的适配器包可以使用。

### 环境变量统一访问

不同运行环境的环境变量访问方式各不相同——Node.js 用 `process.env`、Deno 用 `Deno.env`、Cloudflare Workers 通过 `c.env` 访问绑定。Hono 提供了统一的 `env()` 工具函数来处理这些差异：

```typescript
import { Hono } from "hono";
import { env } from "hono/env";

const app = new Hono();

// 统一的环境变量访问方式
app.get("/config", (c) => {
  const { DATABASE_URL, API_KEY, NODE_ENV } = env<{
    DATABASE_URL: string;
    API_KEY: string;
    NODE_ENV: string;
  }>(c);
  
  return c.json({
    database: DATABASE_URL ? "configured" : "not configured",
    environment: NODE_ENV,
  });
});

// Cloudflare Workers 中，你还可以访问 KV 和 Durable Objects
app.get("/data/:key", async (c) => {
  const key = c.req.param("key");
  
  // 统一的 c.env 访问，在 Cloudflare 中就是 c.env.KV
  const value = await c.env.KV.get(key);
  
  if (!value) {
    return c.json({ error: "Not found" }, 404);
  }
  
  return c.json({ key, value });
});
```

---

## 路由系统详解

### 基础路由与路径参数

Hono 的路由系统设计得极为简洁和直观。你可以直接在应用实例上注册路由：

```typescript
import { Hono } from "hono";

const app = new Hono();

// 基础路由
app.get("/", (c) => c.text("Hello Hono!"));

// 路径参数
app.get("/users/:id", (c) => {
  const id = c.req.param("id");
  return c.json({ userId: id });
});

// 多个路径参数
app.get("/users/:userId/posts/:postId", (c) => {
  const { userId, postId } = c.req.param();
  return c.json({ userId, postId });
});

// 查询参数
app.get("/search", (c) => {
  const query = c.req.query("q");
  const page = c.req.query("page") ?? "1";
  const limit = c.req.query("limit") ?? "10";
  const tags = c.req.queryAll("tags"); // 获取所有同名的查询参数
  
  return c.json({ query, page: parseInt(page), limit: parseInt(limit), tags });
});

// 可选参数
app.get("/items/:id?", (c) => {
  const id = c.req.param("id");
  return c.json({ id: id ?? "all" });
});

// 通配符路由
app.get("/static/*", (c) => {
  const path = c.req.path();
  return c.text(`Serving: ${path}`);
});
```

### 路由分组与嵌套

当你的 API 变得复杂时，你会需要将路由分组管理。Hono 允许你创建子应用（本质上也是一个 Hono 实例），然后将它们挂载到主应用中：

```typescript
import { Hono } from "hono";

const app = new Hono();

// 创建用户路由组
const users = new Hono();
users.get("/", (c) => c.json({ users: [] }));
users.get("/:id", (c) => c.json({ id: c.req.param("id") }));
users.post("/", (c) => c.json({ id: "new" }, 201));
users.put("/:id", (c) => c.json({ id: c.req.param("id"), updated: true }));
users.delete("/:id", (c) => c.body(null, 204));

// 创建帖子路由组
const posts = new Hono();
posts.get("/", (c) => c.json({ posts: [] }));
posts.get("/:id", (c) => c.json({ id: c.req.param("id") }));
posts.post("/", (c) => c.json({ id: "new" }, 201));

// 创建管理员路由组（嵌套）
const admin = new Hono();
const adminUsers = new Hono();

adminUsers.get("/", (c) => c.json({ users: [], admin: true }));
adminUsers.post("/", (c) => c.json({ id: "new-admin" }, 201));
adminUsers.delete("/:id", (c) => c.body(null, 204));

admin.route("/users", adminUsers);

// 将路由组挂载到主应用
app.route("/api/users", users);
app.route("/api/posts", posts);
app.route("/admin", admin);

// 最终路由：
// GET  /api/users
// GET  /api/users/:id
// POST /api/users
// PUT  /api/users/:id
// DELETE /api/users/:id
// GET  /api/posts
// GET  /api/posts/:id
// POST /api/posts
// GET  /admin/users
// POST /admin/users
// DELETE /admin/users/:id
```

### 正则路由

Hono 还支持在路由参数中使用正则表达式来进行更精确的匹配：

```typescript
const app = new Hono();

// 年份匹配：4 位数字
app.get("/posts/:year(\\d{4})/:month(\\d{2})", (c) => {
  const { year, month } = c.req.param();
  return c.json({ year, month });
});

// 分类匹配：只能是特定值
app.get("/items/:category(electronics|fashion|books)/:id(\\d+)", (c) => {
  const { category, id } = c.req.param();
  return c.json({ category, id: parseInt(id) });
});

// 邮箱路由（简化示例）
app.get("/invite/:email([a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,})", (c) => {
  const email = c.req.param("email");
  return c.json({ email });
});
```

---

## 中间件体系

### 中间件的执行模型

Hono 的中间件遵循经典的「**洋葱模型**」。当你调用 `await next()` 时，代码的执行权会被传递给下一个中间件，直到到达路由处理器，然后按照相反的顺序执行「后半段」代码。理解这个模型对于正确使用中间件至关重要：

```typescript
import { Hono } from "hono";

const app = new Hono();

// 第一个中间件
app.use("*", async (c, next) => {
  console.log("1. 第一个中间件 - 前置逻辑");
  console.log("   当前路径:", c.req.path);
  
  await next(); // 调用下一个中间件
  
  console.log("6. 第一个中间件 - 后置逻辑");
  console.log("   响应状态:", c.res.status);
});

// 第二个中间件
app.use("*", async (c, next) => {
  console.log("2. 第二个中间件 - 前置逻辑");
  
  await next();
  
  console.log("5. 第二个中间件 - 后置逻辑");
});

// 路径特定中间件
app.use("/api/*", async (c, next) => {
  console.log("3. API 中间件 - 前置逻辑");
  console.log("   请求方法:", c.req.method);
  
  await next();
  
  console.log("4. API 中间件 - 后置逻辑");
});

// 路由处理器
app.get("/api/hello", (c) => {
  console.log("   → 路由处理器执行");
  return c.json({ message: "Hello from API" });
});

// 访问 GET /api/hello 时的控制台输出顺序：
// 1. 第一个中间件 - 前置逻辑
//    当前路径: /api/hello
// 2. 第二个中间件 - 前置逻辑
// 3. API 中间件 - 前置逻辑
//    请求方法: GET
//    → 路由处理器执行
// 4. API 中间件 - 后置逻辑
// 5. 第二个中间件 - 后置逻辑
// 6. 第一个中间件 - 后置逻辑
//    响应状态: 200
```

### 内置中间件

Hono 提供了一系列开箱即用的中间件，覆盖了常见的 Web 开发需求：

| 中间件 | 功能描述 | 导入路径 |
|--------|---------|---------|
| **cors** | 跨域资源共享配置 | `hono/cors` |
| **logger** | 请求日志记录 | `hono/logger` |
| **compress** | Gzip/Deflate 压缩 | `hono/compress` |
| **etag** | 自动 ETag 生成 | `hono/etag` |
| **poweredBy** | 设置 X-Powered-By 头 | `hono/powered-by` |
| **secureHeaders** | 安全响应头集合 | `hono/secure-headers` |
| **basicAuth** | Basic 认证 | `hono/basic-auth` |
| **bearerAuth** | Bearer Token 认证 | `hono/bearer-auth` |
| **jwt** | JWT 验证 | `hono/jwt` |
| **rateLimit** | 请求速率限制 | `hono/rate-limit` |
| **bodyParser** | 请求体解析 | `hono/body-parser` |
| **prettyJSON** | 格式化 JSON 输出 | `hono/pretty-json` |
| **cache** | 缓存控制 | `hono/cache` |

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { compress } from "hono/compress";
import { secureHeaders } from "hono/secure-headers";
import { bearerAuth } from "hono/bearer-auth";
import { rateLimit } from "hono/rate-limit";

const app = new Hono();

// 全局中间件：所有路由都会经过
app.use("*", cors({
  origin: ["https://example.com", "https://app.example.com"],
  credentials: true,
}));

app.use("*", compress());
app.use("*", secureHeaders());
app.use("*", logger());

// 限流中间件：只应用于 /api/* 路径
app.use("/api/*", rateLimit({
  windowMs: 60 * 1000, // 1 分钟窗口
  limit: 100,          // 最多 100 个请求
  generator: (c) => c.req.header("X-Forwarded-For") ?? "anonymous",
}));

// 认证中间件：保护 /admin/* 路径
app.use("/admin/*", bearerAuth({ token: "admin-secret-token" }));

// 自定义中间件：记录请求耗时
app.use("*", async (c, next) => {
  const start = Date.now();
  
  await next();
  
  const duration = Date.now() - start;
  c.res.headers.set("X-Response-Time", `${duration}ms`);
});

// 路由
app.get("/", (c) => c.text("Hello!"));
app.get("/api/public", (c) => c.json({ data: "public" }));
app.get("/api/private", (c) => c.json({ data: "private" }));
app.get("/admin/dashboard", (c) => c.json({ admin: true }));
```

### 自定义中间件

你可以随时创建自己的中间件来处理特定需求：

```typescript
import { Hono, MiddlewareHandler } from "hono";

// 基础函数式中间件
const requestIdMiddleware: MiddlewareHandler = async (c, next) => {
  const requestId = c.req.header("X-Request-ID") || crypto.randomUUID();
  c.set("requestId", requestId);
  
  await next();
};

// 带配置的中间件工厂
const authMiddleware = (options: { required: boolean; roles?: string[] }): MiddlewareHandler => {
  return async (c, next) => {
    const authHeader = c.req.header("Authorization");
    
    if (!authHeader?.startsWith("Bearer ")) {
      if (options.required) {
        return c.json({ error: "Unauthorized" }, 401);
      }
      return await next();
    }
    
    const token = authHeader.substring(7);
    
    try {
      const payload = await verifyToken(token);
      c.set("user", payload);
      
      // 角色检查
      if (options.roles && !options.roles.includes(payload.role)) {
        return c.json({ error: "Forbidden" }, 403);
      }
      
      await next();
    } catch {
      return c.json({ error: "Invalid token" }, 401);
    }
  };
};

// 带条件的中间件
const corsMiddleware = (): MiddlewareHandler => {
  return async (c, next) => {
    const origin = c.req.header("Origin");
    
    // 只对特定域名开放 CORS
    const allowedOrigins = ["https://example.com", "https://app.example.com"];
    
    if (origin && allowedOrigins.includes(origin)) {
      c.res.headers.set("Access-Control-Allow-Origin", origin);
      c.res.headers.set("Access-Control-Allow-Credentials", "true");
      c.res.headers.set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE, PATCH, OPTIONS");
      c.res.headers.set("Access-Control-Allow-Headers", "Content-Type, Authorization");
    }
    
    if (c.req.method === "OPTIONS") {
      return c.body(null, 204);
    }
    
    await next();
  };
};

// 使用自定义中间件
app.use("*", requestIdMiddleware);
app.use("/api/*", corsMiddleware());
app.use("/api/*", authMiddleware({ required: true, roles: ["admin", "user"] }));
app.use("/admin/*", authMiddleware({ required: true, roles: ["admin"] }));

// 在路由处理器中访问中间件设置的变量
app.get("/profile", (c) => {
  const requestId = c.get("requestId");
  const user = c.get("user");
  
  return c.json({
    requestId,
    user: user ? { id: user.id, email: user.email } : null,
  });
});
```

---

## Zod 验证集成

### 为什么选择 Zod

Zod 是 TypeScript 生态中最流行的运行时验证库之一，它的核心理念是「**Schema 即类型**」——你定义一个 Zod Schema，同时就获得了对应的 TypeScript 类型。与 JSON Schema 相比，Zod 的 API 更符合直觉，错误信息更友好，而且天然支持 TypeScript 的类型推导。

Hono 提供了官方集成包 `@hono/zod-validator`，让你可以在路由定义中直接使用 Zod Schema 进行验证：

```typescript
import { Hono } from "hono";
import { z } from "zod";
import { zValidator } from "@hono/zod-validator";

const app = new Hono();

// 用户创建 Schema
const createUserSchema = z.object({
  email: z.string().email("无效的邮箱格式"),
  username: z.string()
    .min(3, "用户名至少需要 3 个字符")
    .max(30, "用户名最多 30 个字符")
    .regex(/^[a-zA-Z0-9_]+$/, "用户名只能包含字母、数字和下划线"),
  password: z.string()
    .min(8, "密码至少需要 8 个字符")
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
      "密码必须包含大小写字母和数字"
    ),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(["user", "admin", "moderator"]).default("user"),
});

// 更新用户 Schema（所有字段可选）
const updateUserSchema = createUserSchema.partial();

// 查询参数 Schema
const querySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.enum(["createdAt", "email", "username"]).default("createdAt"),
  order: z.enum(["asc", "desc"]).default("desc"),
  search: z.string().optional(),
});

// 路径参数 Schema
const paramSchema = z.object({
  id: z.string().regex(/^[a-f0-9]{24}$/, "无效的 ID 格式"),
});

// 创建用户
app.post(
  "/users",
  zValidator("json", createUserSchema),
  async (c) => {
    // 经过验证的数据
    const body = c.req.valid("json");
    
    // 此时 body 的类型被正确推断为 { email: string; username: string; ... }
    
    const user = await createUser(body);
    
    return c.json({ user }, 201);
  }
);

// 获取单个用户
app.get(
  "/users/:id",
  zValidator("param", paramSchema),
  async (c) => {
    const { id } = c.req.valid("param");
    
    const user = await getUserById(id);
    
    if (!user) {
      return c.json({ error: "User not found" }, 404);
    }
    
    return c.json({ user });
  }
);

// 列出用户（带查询参数验证）
app.get(
  "/users",
  zValidator("query", querySchema),
  async (c) => {
    const query = c.req.valid("query");
    // query 的类型: { page: number; limit: number; sort: string; order: string; search?: string }
    
    const { page, limit, sort, order, search } = query;
    
    const users = await listUsers({
      page,
      limit,
      sort,
      order,
      search,
    });
    
    return c.json(users);
  }
);

// 更新用户
app.patch(
  "/users/:id",
  zValidator("param", paramSchema),
  zValidator("json", updateUserSchema),
  async (c) => {
    const { id } = c.req.valid("param");
    const body = c.req.valid("json");
    
    const user = await updateUser(id, body);
    
    if (!user) {
      return c.json({ error: "User not found" }, 404);
    }
    
    return c.json({ user });
  }
);
```

### 高级 Zod 技巧

```typescript
import { z } from "zod";

// 嵌套对象 Schema
const addressSchema = z.object({
  street: z.string().min(1),
  city: z.string().min(1),
  country: z.string().min(2).max(100),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/, "无效的邮政编码"),
});

const userWithAddressSchema = z.object({
  ...createUserSchema.shape,
  address: addressSchema,
  tags: z.array(z.string().min(1)).max(10).default([]),
  metadata: z.record(z.string(), z.unknown()).optional(),
});

// 联合类型（区分联合）
const resourceSchema = z.discriminatedUnion("type", [
  z.object({ 
    type: z.literal("user"), 
    user: z.object({
      id: z.string(),
      email: z.string().email(),
      name: z.string(),
    }),
  }),
  z.object({ 
    type: z.literal("post"), 
    post: z.object({
      id: z.string(),
      title: z.string(),
      content: z.string(),
    }),
  }),
  z.object({ 
    type: z.literal("comment"), 
    comment: z.object({
      id: z.string(),
      content: z.string(),
      parentId: z.string().nullable(),
    }),
  }),
]);

// 变换和预处理
const transformSchema = z.object({
  id: z.string().transform(Number), // 字符串转数字
  createdAt: z.string().transform((s) => new Date(s)), // 字符串转 Date
  price: z.coerce.number(), // 自动尝试转为数字
  name: z.string().transform((s) => s.trim().toLowerCase()), // 字符串变换
});

// 异步验证（检查数据库中是否存在）
const uniqueEmailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: "该邮箱已被注册", path: ["email"] }
);

// 带条件的字段
const registrationSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
  // 如果是管理员注册，需要提供额外的 adminCode
  role: z.enum(["user", "admin"]),
  adminCode: z.string().optional(),
}).refine(
  (data) => {
    if (data.role === "admin" && !data.adminCode) {
      return false;
    }
    return true;
  },
  { message: "管理员注册需要提供 adminCode", path: ["adminCode"] }
);
```

---

## Hono + Cloudflare D1/KV

Cloudflare Workers 生态为 Hono 提供了绝佳的运行环境。结合 Cloudflare 的 D1（SQLite 数据库）、KV（键值存储）和 R2（对象存储），你可以构建高性能、低成本的边缘应用。

```typescript
// index.ts
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { toPatch晴 } from "hono/cloudflare-workers";

// 类型定义
type Env = {
  DB: D1Database;
  KV: KVNamespace;
  ASSETS: { fetch: typeof fetch };
};

const app = new Hono<{ Bindings: Env }>();

// 全局中间件
app.use("*", cors({
  origin: ["https://example.com", "https://app.example.com"],
  credentials: true,
}));
app.use("*", logger());

// 简单的 KV 缓存示例
app.get("/api/data", async (c) => {
  const cacheKey = "api:data:v1";
  
  // 尝试从 KV 获取缓存
  const cached = await c.env.KV.get(cacheKey, "json");
  
  if (cached) {
    return c.json(cached, {
      headers: { "X-Cache": "HIT" },
    });
  }
  
  // 从 D1 获取数据
  const result = await c.env.DB.prepare(
    "SELECT * FROM posts WHERE published = true ORDER BY created_at DESC LIMIT 20"
  ).all();
  
  const data = result.results;
  
  // 存入 KV（5 分钟过期）
  await c.env.KV.put(cacheKey, JSON.stringify(data), {
    expirationTtl: 300,
  });
  
  return c.json(data, {
    headers: { "X-Cache": "MISS" },
  });
});

// D1 CRUD 操作
app.post("/api/posts", async (c) => {
  const { title, content, authorId } = await c.req.json();
  
  if (!title || !content) {
    return c.json({ error: "Title and content are required" }, 400);
  }
  
  const slug = title
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)/g, "");
  
  const result = await c.env.DB.prepare(
    "INSERT INTO posts (title, slug, content, author_id, published, created_at, updated_at) VALUES (?, ?, ?, ?, ?, ?, ?)"
  ).bind(
    title,
    slug,
    content,
    authorId,
    false,
    new Date().toISOString(),
    new Date().toISOString()
  ).run();
  
  return c.json({ 
    id: result.meta?.last_row_id,
    slug,
  }, 201);
});

app.get("/api/posts/:slug", async (c) => {
  const slug = c.req.param("slug");
  
  const result = await c.env.DB.prepare(
    "SELECT * FROM posts WHERE slug = ? AND published = true"
  ).bind(slug).first();
  
  if (!result) {
    return c.json({ error: "Post not found" }, 404);
  }
  
  return c.json(result);
});

app.delete("/api/posts/:id", async (c) => {
  const id = c.req.param("id");
  
  const result = await c.env.DB.prepare(
    "DELETE FROM posts WHERE id = ?"
  ).bind(id).run();
  
  if (result.meta?.changes === 0) {
    return c.json({ error: "Post not found" }, 404);
  }
  
  return c.body(null, 204);
});

// 导出
export default {
  fetch: app.fetch,
  scheduled: async (controller) => {
    // Cron 定时任务：每天清理过期缓存
    await cleanupExpiredCache();
  },
};
```

> [!TIP]
> **Cloudflare Workers 存储对比**：D1 是 SQLite 兼容的数据库，适合存储结构化数据；KV 是键值存储，适合缓存和简单的配置数据；R2 是对象存储，适合文件和大数据。选择哪个取决于你的实际需求——对于大多数中小型应用，D1 + KV 的组合已经足够。

---

## TypeScript-first API 设计

Hono 的 TypeScript 支持非常完善，几乎所有的东西都能被类型推断。掌握这些类型系统可以让你的代码更安全、更易维护。

### Context 扩展

Hono 的 Context 对象（通常简写为 `c`）可以通过泛型参数进行扩展，添加你自己的变量类型：

```typescript
import { Hono } from "hono";
import { Context } from "hono";

// 定义 Variables 类型
interface Variables {
  requestId: string;
  startTime: number;
  user: {
    id: string;
    email: string;
    role: string;
  } | null;
}

// 定义 Bindings 类型（用于 Cloudflare Workers 等）
interface Bindings {
  DB: D1Database;
  KV: KVNamespace;
}

// 完整的 Env 类型
type Env = {
  Variables: Variables;
  Bindings: Bindings;
};

// 创建应用时指定类型
const app = new Hono<Env>();

// 在中间件中设置变量
app.use("*", async (c, next) => {
  // 设置的变量会被类型检查
  c.set("requestId", crypto.randomUUID());
  c.set("startTime", Date.now());
  c.set("user", null); // 未认证时为 null
  
  await next();
});

// 在路由处理器中获取变量（类型安全）
app.get("/profile", (c) => {
  // user 的类型是 { id: string; email: string; role: string } | null
  const user = c.get("user");
  
  if (!user) {
    return c.json({ error: "Not authenticated" }, 401);
  }
  
  return c.json({ 
    id: user.id,
    email: user.email,
    role: user.role,
  });
});

// requestId 也是类型安全的
app.get("/debug", (c) => {
  const requestId = c.get("requestId");
  const startTime = c.get("startTime");
  
  return c.json({
    requestId,
    elapsed: Date.now() - startTime,
  });
});
```

### 请求与响应的类型推断

Hono 能够根据你定义的 Schema 推断出请求和响应的类型：

```typescript
import { Hono } from "hono";
import { z } from "zod";
import { zValidator } from "@hono/zod-validator";
import { zColumn } from "@/utils/zod-helpers";

const app = new Hono();

// 定义 Schema
const createPostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(1),
  tags: z.array(z.string()).max(10).default([]),
  published: z.boolean().default(false),
});

const postSchema = createPostSchema.extend({
  id: z.string(),
  slug: z.string(),
  authorId: z.string(),
  viewCount: z.number().default(0),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
});

// 当你使用 zValidator 时，c.req.valid() 的返回类型会被推断
app.post("/posts", zValidator("json", createPostSchema), async (c) => {
  // body 的类型是 CreatePostInput
  const body = c.req.valid("json");
  
  // ... 创建帖子的逻辑
  
  // 返回的响应类型应该匹配 postSchema
  return c.json({
    id: "new-id",
    slug: "new-post",
    ...body,
    authorId: "author-id",
    viewCount: 0,
    createdAt: new Date().toISOString(),
    updatedAt: new Date().toISOString(),
  }, 201);
});

// 获取单个帖子时，响应也是类型安全的
app.get("/posts/:id", async (c) => {
  const id = c.req.param("id");
  
  // ... 获取帖子的逻辑
  
  // 返回类型应该符合 postSchema
  const post = await getPost(id);
  
  return c.json(post);
});
```

---

## 实际项目结构推荐

```
my-hono-api/
├── src/
│   ├── index.ts              # 入口文件
│   ├── app.ts                # 应用配置
│   ├── routes/               # 路由
│   │   ├── index.ts          # 路由汇总
│   │   ├── users.ts          # 用户路由
│   │   ├── posts.ts          # 帖子路由
│   │   └── auth.ts           # 认证路由
│   ├── middleware/           # 中间件
│   │   ├── auth.ts           # 认证中间件
│   │   ├── rateLimit.ts      # 限流中间件
│   │   └── requestId.ts       # 请求 ID 中间件
│   ├── lib/                   # 工具库
│   │   ├── db.ts             # 数据库连接
│   │   ├── auth.ts           # 认证工具
│   │   └── errors.ts         # 错误处理
│   ├── validators/           # 验证 Schema
│   │   ├── user.ts
│   │   └── post.ts
│   └── types/                # 类型定义
│       ├── index.ts
│       └── env.ts
├── wrangler.toml             # Cloudflare Workers 配置
├── package.json
└── tsconfig.json
```

```typescript
// src/index.ts - 根据运行环境选择入口方式
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { secureHeaders } from "hono/secure-headers";
import { userRoutes } from "./routes/users";
import { postRoutes } from "./routes/posts";
import { authRoutes } from "./routes/auth";
import { errorHandler } from "./middleware/errorHandler";

const app = new Hono();

// 全局中间件
app.use("*", cors({
  origin: (origin) => origin,
  credentials: true,
}));
app.use("*", secureHeaders());
app.use("*", logger());

// 全局错误处理
app.onError((err, c) => {
  return errorHandler(err, c);
});

// 404 处理
app.notFound((c) => {
  return c.json({ 
    error: "Not Found",
    message: `Cannot ${c.req.method} ${c.req.path}`,
  }, 404);
});

// 注册路由
app.route("/api/users", userRoutes);
app.route("/api/posts", postRoutes);
app.route("/api/auth", authRoutes);

// 健康检查
app.get("/health", (c) => c.json({ status: "ok" }));

// Cloudflare Workers 导出
export default {
  fetch: app.fetch,
  scheduled: async (controller) => {
    // 定时任务
  },
};
```

---

## Hono vs Express vs Fastify vs H3：全面对比

### 核心维度对比

| 维度 | Hono | Express | Fastify | H3 |
|------|------|---------|---------|-----|
| **体积** | ~14KB | ~500KB | ~350KB | ~50KB |
| **性能** | 极高 | 一般 | 极高 | 极高 |
| **TypeScript** | 原生完善 | 需配置 | 原生完善 | 原生完善 |
| **中间件** | 函数式 | 函数式 | 封装插件 | 函数式 |
| **验证** | Zod | 需库 | JSON Schema/Zod | 需库 |
| **跨运行时** | 12+ | 仅 Node.js | 仅 Node.js | 7+ |
| **学习曲线** | 平缓 | 平缓 | 较陡 | 平缓 |
| **生态** | 成长中 | 极丰富 | 丰富 | 小众 |
| **封装隔离** | 无 | 无 | 支持 | 无 |

### 性能对比（理论值）

> [!NOTE]
> 以下数据来自公开的 benchmark 测试，实际性能因硬件、网络、代码实现而异。Hono 和 H3 在简单路由场景下性能接近，但在复杂业务逻辑中差距会缩小。

| 测试场景 | Hono | H3 | Fastify | Express |
|---------|------|-----|---------|---------|
| 简单 JSON 响应 | 85,000 req/s | 90,000 req/s | 70,000 req/s | 25,000 req/s |
| 带数据库查询 | 45,000 req/s | 48,000 req/s | 35,000 req/s | 15,000 req/s |
| 中间件链（5层） | 65,000 req/s | 70,000 req/s | 45,000 req/s | 18,000 req/s |

### 选型决策树

```
你的项目需要什么？
├── 需要边缘计算部署（Cloudflare Workers 等）
│   └── → Hono（唯一支持所有主流边缘运行时的框架）
│
├── 需要跨多个运行时部署（Node.js + Bun + Deno）
│   └── → Hono（一次编写，到处运行）
│
├── 需要极致性能和最小体积
│   ├── 只需要 Bun/Node → H3 或 Elysia
│   └── 需要更多生态 → Hono
│
├── 企业级应用，需要清晰的代码组织
│   ├── 团队熟悉 Angular 风格 → NestJS
│   └── 团队喜欢轻量框架 → Fastify
│
└── 快速原型，简单项目
    └── → Express 或 Hono
```

---

## 常见陷阱与最佳实践

### 陷阱一：忘记处理异步错误

```typescript
// ❌ 错误：在 async 函数中抛出错误但没有 try-catch
app.get("/data", async (c) => {
  const data = await fetchData(); // 如果这里抛出错误，Hono 会直接返回 500
  return c.json(data);
});

// ✅ 正确：显式处理错误或让全局错误处理器处理
app.get("/data", async (c) => {
  try {
    const data = await fetchData();
    return c.json(data);
  } catch (error) {
    // 重新抛出，让全局 onError 处理
    throw error;
  }
});

// 或者在路由中直接返回错误
app.get("/data", async (c) => {
  const data = await fetchData().catch(() => null);
  
  if (!data) {
    return c.json({ error: "Data not found" }, 404);
  }
  
  return c.json(data);
});
```

### 陷阱二：在中间件中修改响应

```typescript
// ❌ 错误：在路由处理后修改响应（此时响应已经发送）
app.get("/example", async (c) => {
  await next(); // 这在路由处理器中是不需要的
  
  c.res.headers.set("X-Custom", "value"); // 无效！
});

// ✅ 正确：在中间件的前半段修改请求，或后半段修改响应头
app.use("*", async (c, next) => {
  // 前半段：可以在 next() 之前修改请求
  c.set("middlewareActive", true);
  
  await next();
  
  // 后半段：可以在 next() 之后修改响应头
  c.res.headers.set("X-Processed-By", "middleware");
});
```

### 最佳实践清单

1. **使用 Zod 进行输入验证** — 不要信任客户端提交的每一个数据，始终验证。

2. **利用路由分组组织代码** — 把相关的路由放在同一个子应用中，便于维护。

3. **实现全局错误处理** — 使用 `app.onError` 和 `app.notFound` 提供一致的错误响应格式。

4. **在 Cloudflare Workers 中合理使用 KV** — KV 适合读多写少的场景，但有速率限制，需要注意。

5. **保持中间件精简** — 中间件越多，请求处理越慢。只加载你真正需要的中间件。

6. **利用 TypeScript 类型系统** — 为 Context 变量定义类型，利用 IDE 的自动补全。

7. **生产环境关闭 logger** — 在正式环境中关闭请求日志，避免不必要的性能开销。

8. **设置合理的限流** — 使用 `hono/rate-limit` 保护你的 API 免受滥用。

> [!SUCCESS]
> Hono 在 2026 年已经成为边缘计算和跨平台 Web 开发的首选框架。其极小的包体积、卓越的性能和广泛的运行时支持使其在 vibecoding 场景中占据独特地位。对于需要部署到 Cloudflare Workers、需要跨多个运行时代码复用、或者追求极致冷启动速度的项目，Hono 是最合适的选择。
