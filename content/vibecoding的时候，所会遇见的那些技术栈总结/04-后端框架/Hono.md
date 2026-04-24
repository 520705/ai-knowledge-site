# Hono 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Hono 设计哲学、跨运行时兼容、中间件体系及与 Express/Fastify 的深度对比。

---

## 目录

1. [[#Hono 概述与定位]]
2. [[#设计哲学]]
3. [[#跨运行时架构]]
4. [[#路由系统]]
5. [[#中间件体系]]
6. [[#Zod 验证]]
7. [[#JSX 支持]]
8. [[#Hono vs Express vs Fastify vs Elysia 对比]]
9. [[#边缘计算场景]]
10. [[#实战场景与选型建议]]
11. [[#参考资料]]

---

## Hono 概述与定位

Hono（炎，日语中「火焰」的意思）是一个轻量、极速的 Web 框架，由 Yusei Murakami 于 2023 年发布。Hono 的核心设计理念是「Write Less, Do More」——用最少的代码实现最多的功能。Hono 最独特的特性是其**跨运行时兼容性**：同一套代码可以在 Node.js、Bun、Deno、Cloudflare Workers、AWS Lambda、Fastly Compute 等 12+ 运行时环境中运行。

### 核心定位

| 维度 | 定位 |
|------|------|
| **目标用户** | 全栈开发者、边缘计算开发者、无服务器开发者 |
| **核心理念** | 轻量、快速、跨运行时 |
| **核心优势** | 极小体积、极速性能、TypeScript 原生 |
| **运行时支持** | 12+ 运行时环境 |
| **体积** | ~14KB（压缩后） |
| **性能** | Tiers 1（极速） |

### 运行时支持

| 运行时 | 支持情况 | 备注 |
|--------|----------|------|
| **Cloudflare Workers** | 原生支持 | 边缘计算首选 |
| **Bun** | 原生支持 | 高性能运行时 |
| **Deno** | 原生支持 | 现代化运行时 |
| **Node.js** | 原生支持 | 通用服务器 |
| **Fastly Compute** | 原生支持 | 边缘计算 |
| **AWS Lambda** | 原生支持 | 无服务器 |
| **Azure Functions** | 原生支持 | 无服务器 |
| **Google Cloud Functions** | 原生支持 | 无服务器 |
| **Vercel Edge** | 原生支持 | 边缘函数 |
| **Supabase Edge** | 原生支持 | 边缘函数 |
| **Netlify** | 原生支持 | 边缘函数 |
| **Lagon** | 原生支持 | Rust 运行时 |

---

## 设计哲学

### 核心原则

1. **轻量优先**：最小化依赖，保持极小体积
2. **Web 标准**：遵循 Fetch API 和 Web 标准
3. **TypeScript 优先**：完整的类型推导和推断
4. **跨运行时**：一次编写，到处运行
5. **中间件驱动**：通过中间件组合实现功能

### 架构对比

```
┌─────────────────────────────────────────────────────────────┐
│                        Hono                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                   Hono App                          │   │
│  │  ┌───────────┐ ┌───────────┐ ┌─────────────────┐   │   │
│  │  │  Router   │ │ Middleware│ │ Context Helpers │   │   │
│  │  └───────────┘ └───────────┘ └─────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            ↓                                 │
│  ┌─────────────────────────────────────────────────────┐   │
│  │               Adapter Layer                          │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌───────────┐  │   │
│  │  │ Node.js │ │   Bun   │ │  Deno   │ │ Cloudflare│  │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └───────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 与 Express 的哲学对比

| 维度 | Hono | Express |
|------|------|---------|
| **体积** | ~14KB | ~500KB |
| **启动时间** | <5ms | ~50ms |
| **类型安全** | TypeScript 原生 | 需要手动类型 |
| **中间件顺序** | 按注册顺序 | 按注册顺序 |
| **错误处理** | 统一错误对象 | 回调模式 |
| **JSON 响应** | 内置 `c.json()` | 需要 `res.json()` |
| **运行时** | 跨平台 | 仅 Node.js |

---

## 跨运行时架构

### 统一接口

```typescript
// 统一的 Hono 应用写法
import { Hono } from "hono";

const app = new Hono();

app.get("/api/users", (c) => {
  return c.json([
    { id: 1, name: "Alice" },
    { id: 2, name: "Bob" },
  ]);
});

app.post("/api/users", async (c) => {
  const body = await c.req.json();
  return c.json({ id: 3, ...body }, 201);
});

export default app;
```

### 运行时适配

```typescript
// Node.js 适配
import { serve } from "@hono/node-server";
import { Hono } from "hono";
import { cors } from "hono/cors";

const app = new Hono();
app.use("*", cors());

app.get("/", (c) => c.text("Hello Node.js!"));

serve({
  fetch: app.fetch,
  port: 3000,
});

console.log("Server running on http://localhost:3000");

// Bun 适配（更简洁）
import { Hono } from "hono";
import { serve } from "bun";

const app = new Hono();
app.get("/", (c) => c.text("Hello Bun!"));

serve({
  fetch: app.fetch,
  port: 3000,
});

// Deno 适配
import { serve } from "std/server";
import { Hono } from "hono";
import { serveStatic } from "hono/deno";

const app = new Hono();
app.use("/*", serveStatic({ root: "./public" }));

serve(app.fetch, { port: 8000 });

// Cloudflare Workers（部署到边缘）
export default {
  fetch: app.fetch,
  scheduled: async (controller) => {
    // Cron 触发器
  },
};
```

### 环境变量统一访问

```typescript
import { Hono } from "hono";
import { env } from "hono/env";

const app = new Hono();

app.get("/config", (c) => {
  // 统一的环境变量访问
  const { DATABASE_URL, API_KEY, NODE_ENV } = env<{
    DATABASE_URL: string;
    API_KEY: string;
    NODE_ENV: string;
  }>(c);
  
  return c.json({
    database: DATABASE_URL ? "configured" : "not configured",
    env: NODE_ENV,
  });
});
```

---

## 路由系统

### 基础路由

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
  return c.json({ query, page: parseInt(page) });
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

### 路由分组

```typescript
import { Hono } from "hono";
import { router as usersRouter } from "./routes/users";
import { router as postsRouter } from "./routes/posts";

const app = new Hono();

// API v1 分组
const apiV1 = new Hono();

apiV1.route("/users", usersRouter);
apiV1.route("/posts", postsRouter);

app.route("/api/v1", apiV1);

// 嵌套分组
const admin = new Hono();
const adminUsers = new Hono();

adminUsers.get("/", (c) => c.json({ users: [] }));
adminUsers.post("/", (c) => c.json({ id: 1 }, 201));
adminUsers.delete("/:id", (c) => c.text("", 204));

admin.route("/users", adminUsers);
app.route("/admin", admin);
```

### 路由方法

```typescript
import { Hono } from "hono";

const app = new Hono();

// 标准 HTTP 方法
app.get("/", (c) => c.text("GET"));
app.post("/", (c) => c.text("POST"));
app.put("/", (c) => c.text("PUT"));
app.patch("/", (c) => c.text("PATCH"));
app.delete("/", (c) => c.text("DELETE"));
app.head("/", (c) => c.text("HEAD"));
app.options("/", (c) => c.text("OPTIONS"));

// 简写方法
app.all("/any", (c) => {
  const method = c.req.method;
  return c.json({ method });
});

// 路由链式
app
  .get("/resource", (c) => c.json({ action: "read" }))
  .post("/resource", (c) => c.json({ action: "create" }, 201))
  .put("/resource/:id", (c) => c.json({ action: "update" }))
  .delete("/resource/:id", (c) => c.json({ action: "delete" }));
```

### 正则路由

```typescript
const app = new Hono();

// 正则匹配路由参数
app.get("/posts/:year(\\d{4})/:month(\\d{2})", (c) => {
  const { year, month } = c.req.param();
  return c.json({ year, month });
});

// 多个正则组
app.get("/items/:category(all|electronics|fashion)/:id(\\d+)", (c) => {
  const { category, id } = c.req.param();
  return c.json({ category, id });
});
```

---

## 中间件体系

### 内置中间件

| 中间件 | 功能 | 导入路径 |
|--------|------|----------|
| **cors** | 跨域资源共享 | `hono/cors` |
| **bodyParser** | 请求体解析 | `hono/body-parser` |
| **logger** | 请求日志 | `hono/logger` |
| **prettyJSON** | 格式化 JSON 响应 | `hono/pretty-json` |
| **compress** | GZip 压缩 | `hono/compress` |
| **etag** | ETag 生成 | `hono/etag` |
| **poweredBy** | X-Powered-By 头 | `hono/powered-by` |
| **secureHeaders** | 安全响应头 | `hono/secure-headers` |
| **BasicAuth** | Basic 认证 | `hono/basic-auth` |
| **BearerAuth** | Bearer Token 认证 | `hono/bearer-auth` |
| **jwt** | JWT 验证 | `hono/jwt` |
| **rateLimit** | 限流 | `hono/rate-limit` |

### 中间件使用

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { compress } from "hono/compress";
import { secureHeaders } from "hono/secure-headers";
import { bearerAuth } from "hono/bearer-auth";
import { jwt } from "hono/jwt";
import { rateLimit } from "hono/rate-limit";

const app = new Hono();

// 全局中间件
app.use("*", cors());
app.use("*", compress());
app.use("*", secureHeaders());
app.use("*", logger());

// 带条件的中间件
app.use("/api/*", async (c, next) => {
  // 前置处理
  const start = Date.now();
  
  await next();
  
  // 后置处理
  const duration = Date.now() - start;
  c.res.headers.set("X-Response-Time", `${duration}ms`);
});

// 认证中间件
app.use("/protected/*", bearerAuth({ token: "secret-token" }));

// JWT 中间件
app.use("/auth/*", jwt({ secret: "your-secret-key" }));

// 限流中间件
app.use("/api/*", rateLimit({
  windowMs: 60 * 1000,  // 1 分钟
  limit: 100,
  generator: (c) => c.req.header("x-forwarded-for") ?? "anonymous",
}));
```

### 自定义中间件

```typescript
import { Hono, MiddlewareHandler } from "hono";

// 函数式中间件
const timing = async (c, next) => {
  const start = Date.now();
  await next();
  const duration = Date.now() - start;
  c.res.headers.set("X-Response-Time", `${duration}ms`);
};

// 错误处理中间件
const errorHandler = async (c, next) => {
  try {
    await next();
  } catch (err) {
    console.error("Error:", err);
    return c.json({
      error: err instanceof Error ? err.message : "Unknown error",
      status: 500,
    }, 500);
  }
};

// 带配置的中间件工厂
const requireHeader = (headerName: string, headerValue: string): MiddlewareHandler => {
  return async (c, next) => {
    const header = c.req.header(headerName);
    if (header !== headerValue) {
      return c.json({ error: `Missing or invalid ${headerName} header` }, 400);
    }
    await next();
  };
};

// 使用
app.use("*", timing);
app.use("*", errorHandler);
app.use("/api/*", requireHeader("X-API-Key", "valid-key"));
```

### Context 扩展

```typescript
import { Hono } from "hono";
import { Context } from "hono";

// 扩展 Context 类型
type Variables = {
  user: { id: number; name: string };
  db: Database;
};

const app = new Hono<{ Variables: Variables }>();

// 设置变量
app.use("*", async (c, next) => {
  c.set("user", { id: 1, name: "Alice" });
  c.set("db", new Database());
  await next();
});

// 获取变量
app.get("/me", (c) => {
  const user = c.get("user");
  return c.json({ user });
});

// 类型安全的服务
const userService = {
  getUser(c: Context<{ Variables: Variables }>) {
    return c.get("user");
  },
};
```

---

## Zod 验证

### 基础验证

```typescript
import { Hono } from "hono";
import { z } from "zod";
import { zValidator } from "@hono/zod-validator";

const app = new Hono();

// 用户 Schema
const userSchema = z.object({
  id: z.number(),
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email format"),
  age: z.number().min(0).max(150).optional(),
});

// 创建用户 Schema
const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8).regex(
    /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
    "Password must contain uppercase, lowercase and number"
  ),
});

// 查询参数 Schema
const querySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
  sort: z.enum(["asc", "desc"]).default("desc"),
});

// 路径参数 Schema
const paramSchema = z.object({
  id: z.string().regex(/^\d+$/, "ID must be a number").transform(Number),
});

// 路由中使用
app.get(
  "/users",
  zValidator("query", querySchema),
  (c) => {
    const { page, limit, sort } = c.req.valid("query");
    return c.json({ page, limit, sort });
  }
);

app.get(
  "/users/:id",
  zValidator("param", paramSchema),
  (c) => {
    const { id } = c.req.valid("param");
    return c.json({ userId: id });
  }
);

app.post(
  "/users",
  zValidator("json", createUserSchema),
  (c) => {
    const { name, email, password } = c.req.valid("json");
    return c.json({ name, email }, 201);
  }
);
```

### 高级验证

```typescript
import { z } from "zod";

// 嵌套 Schema
const addressSchema = z.object({
  street: z.string(),
  city: z.string(),
  country: z.string(),
  zipCode: z.string().regex(/^\d{5}(-\d{4})?$/),
});

const userWithAddressSchema = z.object({
  ...userSchema.shape,
  address: addressSchema,
  tags: z.array(z.string()).max(10),
});

// 联合类型
const resourceSchema = z.discriminatedUnion("type", [
  z.object({ type: z.literal("user"), user: userSchema }),
  z.object({ type: z.literal("post"), post: z.object({ title: z.string(), content: z.string() }) }),
]);

// 变换和预处理
const transformSchema = z.object({
  id: z.string().transform(Number),
  createdAt: z.string().transform((s) => new Date(s)),
  price: z.coerce.number(),  // 自动转为数字
});

// 异步验证
const uniqueEmailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: "Email already exists" }
);

// 验证中间件工厂
const validateBody = <T extends z.ZodSchema>(schema: T) => {
  return zValidator("json", schema);
};

app.post("/users", validateBody(createUserSchema), (c) => {
  // c.req.valid("json") 返回的类型会自动推断
  const body = c.req.valid("json");
  return c.json(body);
});
```

---

## JSX 支持

### Hono JSX 配置

```typescript
// renderer.tsx
import { FC, Children } from "hono/jsx";

export const Layout: FC = (props) => {
  return (
    <html>
      <head>
        <title>{props.title}</title>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      </head>
      <body>
        <header>
          <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
          </nav>
        </header>
        <main>
          {props.children}
        </main>
        <footer>
          <p>&copy; 2026 My App</p>
        </footer>
      </body>
    </html>
  );
};

export const Title: FC<{ children: string }> = (props) => {
  return <h1>{props.children}</h1>;
};
```

### JSX 路由

```typescript
/** @jsxImportSource hono/jsx */
// renderer.tsx 需要此声明

import { Hono } from "hono";
import { html } from "hono/html";
import { Layout, Title } from "./renderer";

const app = new Hono();

app.get("/", (c) => {
  return c.render(
    <Layout title="Home">
      <Title>Welcome</Title>
      <p>This is a server-rendered page.</p>
    </Layout>
  );
});

app.get("/about", (c) => {
  return c.render(
    <Layout title="About">
      <Title>About Us</Title>
      <p>Learn more about our company.</p>
    </Layout>
  );
});
```

### HTML 模板

```typescript
import { Hono } from "hono";
import { html } from "hono/html";

// 使用 html 标签函数
app.get("/login", (c) => {
  return c.html(
    html`
      <!DOCTYPE html>
      <html>
        <head>
          <title>Login</title>
        </head>
        <body>
          <form method="POST" action="/auth/login">
            <input type="email" name="email" placeholder="Email" required />
            <input type="password" name="password" placeholder="Password" required />
            <button type="submit">Login</button>
          </form>
        </body>
      </html>
    `
  );
});
```

---

## Hono vs Express vs Fastify vs Elysia 对比

### 核心对比表

| 特性 | Hono | Express | Fastify | Elysia |
|------|------|---------|---------|--------|
| **体积** | ~14KB | ~500KB | ~300KB | ~80KB |
| **性能** | 极高 | 中等 | 高 | 极高 |
| **类型安全** | TypeScript 原生 | 需手动 | 插件化 | 高度类型化 |
| **中间件** | 函数式 | 函数式 | 封装器 | 装饰器 |
| **验证** | Zod 集成 | 需库 | ajv/schema | tJypescript |
| **跨运行时** | 12+ | 仅 Node.js | 仅 Node.js | 仅 Bun/Node |
| **学习曲线** | 低 | 低 | 中 | 中 |
| **插件生态** | 成长中 | 庞大 | 丰富 | 成长中 |
| **异步支持** | 原生 | 有限 | 原生 | 原生 |

### Hono 适配运行时表

| 运行时 | 适配器 | 导入 |
|--------|--------|------|
| **Node.js** | `@hono/node-server` | `serve` |
| **Bun** | 内置 | `serve` |
| **Deno** | 内置 | `serve` |
| **Cloudflare** | 内置 | 默认导出 |
| **Fastly** | `@hono/fastly` | `handle` |
| **AWS Lambda** | `@hono/node-server` | API Gateway 事件 |
| **Vercel Edge** | `@hono/vercel-edge` | `fetch` |

### 性能对比（req/s）

| 框架 | 简单 JSON | 数据库查询 | 并发 1000 |
|------|----------|------------|-----------|
| **Hono** | 85,000 | 45,000 | 80,000 |
| **Elysia** | 90,000 | 50,000 | 85,000 |
| **Fastify** | 70,000 | 35,000 | 65,000 |
| **Express** | 25,000 | 15,000 | 20,000 |

> [!TIP]
> **选型建议**：
> - **跨运行时/边缘部署**：Hono（唯一选择）
> - **极致性能 + Bun**：Elysia（最快）
> - **成熟生态 + Node.js**：Fastify
> - **快速原型/简单项目**：Express

---

## 边缘计算场景

### Cloudflare Workers 部署

```typescript
// index.tsx
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";

const app = new Hono();

// 边缘友好的中间件
app.use("*", cors({
  origin: ["https://example.com", "https://app.example.com"],
  credentials: true,
}));

app.use("*", logger());

// 路由
app.get("/api/health", (c) => c.json({ status: "ok", region: c.req.header("CF-IPCountry") }));

app.get("/api/users", async (c) => {
  // 可以使用 KV
  const cacheKey = "users:list";
  const cached = await c.env.KV.get(cacheKey, "json");
  
  if (cached) {
    return c.json(cached, {
      headers: { "Cache-Control": "public, max-age=60" },
    });
  }
  
  // 获取数据
  const users = await fetchUsers();
  
  // 缓存到 KV
  await c.env.KV.put(cacheKey, JSON.stringify(users), { expirationTtl: 300 });
  
  return c.json(users);
});

app.post("/api/analytics", async (c) => {
  const body = await c.req.json();
  
  // 使用 Durable Objects 计数
  const doId = c.env.ANALYTICS.idFromName("global");
  const doStub = c.env.ANALYTICS.get(doId);
  
  await doStub.fetch("https://analytics/increment", {
    method: "POST",
    body: JSON.stringify(body),
  });
  
  return c.json({ success: true });
});

export default app;

// wrangler.toml
// name = "my-app"
// main = "src/index.ts"
// compatibility_date = "2024-01-01"
// kv_namespaces = [ { binding = "KV", id = "xxx" } ]
// durable_objects = { bindings = [{ name = "ANALYTICS", class_name = "Analytics" }] }
```

### 边缘缓存策略

```typescript
import { Hono } from "hono";
import { cache } from "hono/cache";

const app = new Hono();

// 内存缓存（Cloudflare Workers 免费版）
app.get(
  "/api/static-data",
  cache({
    cacheName: "static-data",
    cacheControl: "public, max-age=3600",  // 1小时
  }),
  (c) => c.json({ data: "cached for 1 hour" })
);

// 条件请求
app.get("/api/users/:id", async (c) => {
  const id = c.req.param("id");
  const etag = `user-${id}-v1`;
  
  // 检查 If-None-Match
  if (c.req.header("If-None-Match") === etag) {
    return c.body(null, 304);
  }
  
  const user = await getUser(id);
  
  return c.json(user, {
    headers: {
      "ETag": etag,
      "Cache-Control": "private, max-age=60",
    },
  });
});
```

---

## 实战场景与选型建议

### 场景一：微服务 API

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import { jwt } from "hono/jwt";

const app = new Hono();

// 全局中间件
app.use("*", cors());
app.use("*", logger());

// 认证
app.use("/api/*", jwt({ secret: process.env.JWT_SECRET! }));

// 用户服务
const users = new Map<number, { id: number; name: string; email: string }>();
let nextId = 1;

// Schema
const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
});

const updateUserSchema = createUserSchema.partial();

// 路由
app.get("/api/users", (c) => {
  return c.json(Array.from(users.values()));
});

app.get("/api/users/:id", (c) => {
  const id = parseInt(c.req.param("id"));
  const user = users.get(id);
  
  if (!user) {
    return c.json({ error: "User not found" }, 404);
  }
  
  return c.json(user);
});

app.post("/api/users", zValidator("json", createUserSchema), (c) => {
  const body = c.req.valid("json");
  const id = nextId++;
  const user = { id, ...body };
  users.set(id, user);
  
  return c.json(user, 201);
});

app.put("/api/users/:id", zValidator("json", updateUserSchema), (c) => {
  const id = parseInt(c.req.param("id"));
  const user = users.get(id);
  
  if (!user) {
    return c.json({ error: "User not found" }, 404);
  }
  
  const body = c.req.valid("json");
  const updated = { ...user, ...body };
  users.set(id, updated);
  
  return c.json(updated);
});

app.delete("/api/users/:id", (c) => {
  const id = parseInt(c.req.param("id"));
  
  if (!users.has(id)) {
    return c.json({ error: "User not found" }, 404);
  }
  
  users.delete(id);
  return c.body(null, 204);
});

export default app;
```

### 场景二：实时聊天后端

```typescript
import { Hono } from "hono";
import { WebSocketRequest } from "hono/cloudflare-workers";

const app = new Hono();

// 存储连接
const connections = new Map<string, WebSocket>();

app.get("/ws/chat/:room", async (c) => {
  const room = c.req.param("room");
  const clientId = crypto.randomUUID();
  
  // WebSocket 升级
  const { error, websocket } = new WebSocketRequest(c.req.raw).getWebSocket(
    c.env.ASSETS,
    c.req.header("Upgrade") || ""
  );
  
  if (error) {
    return c.text("WebSocket error", 400);
  }
  
  connections.set(`${room}:${clientId}`, websocket);
  
  websocket.addEventListener("open", () => {
    console.log(`Client ${clientId} joined room ${room}`);
    broadcast(room, { type: "join", clientId });
  });
  
  websocket.addEventListener("message", (event) => {
    const message = JSON.parse(event.data);
    broadcast(room, {
      type: "message",
      clientId,
      content: message.content,
      timestamp: Date.now(),
    });
  });
  
  websocket.addEventListener("close", () => {
    connections.delete(`${room}:${clientId}`);
    broadcast(room, { type: "leave", clientId });
  });
  
  return new Response(null, { status: 101 });
});

function broadcast(room: string, message: object) {
  for (const [key, ws] of connections) {
    if (key.startsWith(`${room}:`)) {
      ws.send(JSON.stringify(message));
    }
  }
}

export default app;
```

### 选型决策树

```
项目类型
├── 边缘计算部署
│   └── → Hono（唯一选择，Cloudflare Workers 原生支持）
├── 多运行时支持
│   └── → Hono（一次编写，12+ 运行环境）
├── 高性能 Bun/Node API
│   ├── 类型安全优先 → Elysia
│   ├── 快速开发 → Hono
│   └── 生态成熟 → Fastify
├── 快速原型
│   └── → Express / Hono
└── 团队技能
    ├── TypeScript 新手 → Express
    ├── TypeScript 专家 → Hono / Elysia
    └── 需要 JSON Schema → Fastify
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Hono 官方文档 | https://hono.dev |
| Hono GitHub | https://github.com/honojs/hono |
| Hono Discord | https://discord.gg/hono |

### 相关资源

| 资源 | 说明 |
|------|------|
| Hono Templates | 各种运行时模板 |
| honojs/middleware | 官方中间件列表 |

---

---

## 核心概念详解

### Hono 的 Context 对象深入理解

Hono 的 Context（通常缩写为 `c`）是框架的核心对象，它封装了请求和响应，并提供了丰富的辅助方法。深入理解 Context 的生命周期和可用方法对于高效使用 Hono 至关重要。

#### Context 的生命周期

```typescript
import { Hono } from "hono";
import { Context } from "hono";

const app = new Hono();

// Context 在每个请求中创建，包含请求信息、响应构建器和状态存储
app.use("*", async (c, next) => {
  // c.req: 请求对象（Request + 额外方法）
  // c.env: 环境变量和绑定
  // c.var: 中间件共享的状态
  // c.set(): 设置变量供后续中间件和处理器使用
  // c.get(): 获取变量
  
  // 在中间件中设置变量
  c.set("requestStartTime", Date.now());
  c.set("requestId", crypto.randomUUID());
  
  // 执行后续处理
  await next();
  
  // 后置处理：添加响应头
  const duration = Date.now() - c.get("requestStartTime");
  c.res.headers.set("X-Response-Time", `${duration}ms`);
});

// 获取中间件设置的变量
app.get("/profile", (c) => {
  const requestId = c.get("requestId");
  return c.json({ requestId });
});
```

#### 请求对象详解

```typescript
import { Hono } from "hono";
import { HonoRequest } from "hono/request";

const app = new Hono();

// 路径参数
app.get("/users/:id/posts/:postId", (c) => {
  const id = c.req.param("id");
  const { id: userId, postId } = c.req.param();
  const userIdNum = parseInt(c.req.param("id"));
  return c.json({ userId, postId });
});

// 查询参数
app.get("/search", (c) => {
  const query = c.req.query("q");
  const allQueries = c.req.query();
  const page = c.req.query("page") ?? "1";
  const limit = c.req.query("limit") ?? "10";
  const tags = c.req.queryAll("tags");
  return c.json({ query, page, limit, tags });
});

// 请求头
app.get("/headers", (c) => {
  const contentType = c.req.header("Content-Type");
  const auth = c.req.header("Authorization");
  const allHeaders = c.req.headers();
  return c.json({ contentType, auth });
});

// 请求体
app.post("/data", async (c) => {
  const json = await c.req.json();
  const form = await c.req.formData();
  const text = await c.req.text();
  const arrayBuffer = await c.req.arrayBuffer();
  const { name, email } = await c.req.json<{ name: string; email: string }>();
  return c.json({ received: true });
});

// URL 和方法
app.use("*", (c) => {
  const url = c.req.url;
  const path = c.req.path;
  const method = c.req.method;
  return c.json({ url, path, method });
});
```

#### 响应构建器

```typescript
import { Hono } from "hono";

const app = new Hono();

// 基础响应
app.get("/text", (c) => c.text("Hello, World!"));
app.get("/html", (c) => c.html("<h1>Hello</h1>"));
app.get("/json", (c) => c.json({ message: "Hello" }));

// 二进制响应
app.get("/binary", (c) => {
  const data = new Uint8Array([72, 101, 108, 108, 111]);
  return c.body(data, {
    headers: { "Content-Type": "application/octet-stream" },
  });
});

// 流式响应
app.get("/stream", async (c) => {
  const stream = new ReadableStream({
    start(controller) {
      const encoder = new TextEncoder();
      for (let i = 0; i < 10; i++) {
        controller.enqueue(encoder.encode(`Message ${i}\n`));
      }
      controller.close();
    },
  });
  return c.body(stream, {
    headers: { "Content-Type": "text/plain", "Transfer-Encoding": "chunked" },
  });
});

// 带自定义头
app.get("/with-headers", (c) => {
  return c.json(
    { data: "value" },
    {
      headers: { "X-Custom-Header": "value" },
      status: 201,
    }
  );
});

// 空响应
app.delete("/resource/:id", (c) => c.body(null, 204));

// 重定向
app.get("/old-page", (c) => c.redirect("/new-page", 301));

// ETag 支持
app.get("/resource/:id", async (c) => {
  const resource = await getResource(c.req.param("id"));
  const etag = `"${resource.version}"`;
  const ifNoneMatch = c.req.header("If-None-Match");
  if (ifNoneMatch === etag) {
    return c.body(null, 304);
  }
  return c.json(resource, {
    headers: { "ETag": etag, "Cache-Control": "private, max-age=60" },
  });
});

// 渲染 JSX
app.get("/page", (c) => {
  return c.render(
    <html>
      <head><title>Page</title></head>
      <body><h1>Hello from Hono JSX</h1></body>
    </html>
  );
});
```

### 路由系统深入

#### 高级路由模式

```typescript
import { Hono } from "hono";

const app = new Hono();

// 正则路由
app.get("/items/:id([0-9]+)", (c) => {
  const id = c.req.param("id");
  return c.json({ id });
});

// 多个正则匹配
app.get("/date/:year([0-9]{4})/:month([0-9]{2})/:day([0-9]{2})", (c) => {
  const { year, month, day } = c.req.param();
  return c.json({ year, month, day });
});

// 可选参数
app.get("/api/users/:userId?", (c) => {
  const userId = c.req.param("userId");
  if (!userId) return c.json({ users: [] });
  return c.json({ userId });
});

// 通配符
app.get("/static/*", (c) => {
  const path = c.req.path;
  return c.json({ path });
});

// 链式路由
const resource = app
  .get("/resource", (c) => c.json({ action: "list" }))
  .post("/resource", (c) => c.json({ action: "create" }, 201))
  .put("/resource/:id", (c) => c.json({ action: "update" }))
  .delete("/resource/:id", (c) => c.json({ action: "delete" }));

// 路由分组
const v1 = new Hono();
const v2 = new Hono();

v1.get("/users", (c) => c.json({ version: "v1", users: [] }));
v1.post("/users", (c) => c.json({ version: "v1", created: true }, 201));

v2.get("/users", (c) => c.json({ version: "v2", data: [] }));
v2.post("/users", (c) => c.json({ version: "v2", created: true }, 201));

app.route("/api/v1", v1);
app.route("/api/v2", v2);

// 嵌套路由组
const api = new Hono();
const users = new Hono();
const posts = new Hono();

users.get("/", (c) => c.json({ users: [] }));
users.get("/:id", (c) => c.json({ id: c.req.param("id") }));
posts.get("/", (c) => c.json({ posts: [] }));
posts.get("/:id", (c) => c.json({ id: c.req.param("id") }));

api.route("/users", users);
api.route("/posts", posts);
app.route("/api", api);

// 路由优先级
app.get("/public", (c) => c.text("public"));
app.get("/:param", (c) => c.text("param"));
app.get("*", (c) => c.text("fallback"));
```

#### 嵌套路由与继承

```typescript
import { Hono } from "hono";
import { middleware as authMiddleware } from "./auth";

const layout = new Hono();

layout.use("*", async (c, next) => {
  c.set("layoutTitle", "My App");
  c.set("currentUser", await getCurrentUser(c));
  await next();
});

layout.get("/layout", (c) => {
  return c.html(
    <html>
      <head><title>{c.get("layoutTitle")}</title></head>
      <body>
        <header>
          <nav>
            <a href="/">Home</a>
            <a href="/about">About</a>
          </nav>
        </header>
        <main>{c.get("children")}</main>
      </body>
    </html>
  );
});

const home = new Hono();
home.get("/", (c) => c.json({ page: "home" }));
home.get("/about", (c) => c.json({ page: "about" }));

const users = new Hono();
users.use("*", authMiddleware());
users.get("/", (c) => c.json({ page: "users" }));
users.get("/:id", (c) => c.json({ page: "user", id: c.req.param("id") }));

layout.route("/", home);
layout.route("/users", users);
app.route("/", layout);
```

### 中间件系统深入

#### 中间件执行顺序与洋葱模型

```typescript
import { Hono } from "hono";

const app = new Hono();

// 洋葱模型演示
app.use("*", async (c, next) => {
  console.log("1. 第一个中间件 - 前置");
  await next();
  console.log("6. 第一个中间件 - 后置");
});

app.use("*", async (c, next) => {
  console.log("2. 第二个中间件 - 前置");
  await next();
  console.log("5. 第二个中间件 - 后置");
});

app.use("/api/*", async (c, next) => {
  console.log("3. API 中间件 - 前置");
  await next();
  console.log("4. API 中间件 - 后置");
});

app.get("/api/hello", (c) => {
  console.log("   路由处理器执行");
  return c.json({ message: "Hello" });
});
```

#### 中间件组合

```typescript
import { Hono, compose } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { rateLimit } from "hono/rate-limit";

const commonMiddleware = compose([
  cors({ origin: ["https://example.com"], credentials: true }),
  logger(),
]);

app.use("*", commonMiddleware);

function createAppMiddleware(options: {
  enableCors: boolean;
  enableLogger: boolean;
  enableRateLimit: boolean;
}) {
  const middlewares: any[] = [];
  
  if (options.enableCors) middlewares.push(cors());
  if (options.enableLogger) middlewares.push(logger());
  if (options.enableRateLimit) {
    middlewares.push(rateLimit({ windowMs: 60000, limit: 100 }));
  }
  
  return compose(middlewares);
}

app.use("*", createAppMiddleware({
  enableCors: true,
  enableLogger: true,
  enableRateLimit: true,
}));
```

#### 中间件错误处理

```typescript
import { Hono } from "hono";
import { HTTPException } from "hono/http-exception";

const app = new Hono();

app.get("/not-found", (c) => {
  throw new HTTPException(404, { message: "Resource not found" });
});

app.get("/unauthorized", (c) => {
  throw new HTTPException(401, { message: "Please login first" });
});

app.get("/forbidden", (c) => {
  throw new HTTPException(403, { message: "Access denied" });
});

app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return c.json({
      error: true,
      status: err.status,
      message: err.message,
    }, err.status);
  }
  return c.json({ error: true, status: 500, message: "Internal server error" }, 500);
});

app.notFound((c) => {
  return c.json({ error: true, status: 404, message: "Not found", path: c.req.path }, 404);
});
```

---

## CRUD 操作完整代码示例

### 完整 RESTful API 实现

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import { PrismaClient } from "@prisma/client";

type Env = {
  Variables: {
    user: { id: string; email: string; role: string };
    prisma: PrismaClient;
  };
};

const prisma = new PrismaClient({
  log: process.env.NODE_ENV === "development" 
    ? ["query", "info", "warn", "error"]
    : ["error"],
});

const userSchemas = {
  create: z.object({
    username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/),
    email: z.string().email(),
    password: z.string().min(8).max(100),
    role: z.enum(["ADMIN", "USER", "MODERATOR"]).optional(),
  }),
  update: z.object({
    username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/).optional(),
    email: z.string().email().optional(),
    role: z.enum(["ADMIN", "USER", "MODERATOR"]).optional(),
  }),
};

const app = new Hono<Env>();

app.use("*", cors());
app.use("*", logger());
app.use("*", async (c, next) => {
  c.set("prisma", prisma);
  await next();
});

app.onError((err, c) => {
  if (err instanceof z.ZodError) {
    return c.json({ error: "Validation Error", details: err.errors }, 400);
  }
  return c.json({ error: "Internal Server Error", message: err.message }, 500);
});

const requireAuth = async (c: any, next: () => Promise<void>) => {
  const authHeader = c.req.header("Authorization");
  if (!authHeader?.startsWith("Bearer ")) {
    return c.json({ error: "Unauthorized" }, 401);
  }
  try {
    const payload = await verifyToken(authHeader.substring(7));
    c.set("user", payload);
    await next();
  } catch {
    return c.json({ error: "Invalid token" }, 401);
  }
};

async function verifyToken(token: string) {
  const payload = JSON.parse(atob(token.split(".")[1]));
  return payload;
}

const users = new Hono<Env>();

users.get("/me", requireAuth, async (c) => {
  const prisma = c.get("prisma");
  const user = c.get("user");
  const profile = await prisma.user.findUnique({
    where: { id: user.id },
    select: { id: true, username: true, email: true, role: true, createdAt: true },
  });
  if (!profile) return c.json({ error: "User not found" }, 404);
  return c.json({ user: profile });
});

users.get("/:id", requireAuth, async (c) => {
  const prisma = c.get("prisma");
  const id = c.req.param("id");
  const user = await prisma.user.findUnique({
    where: { id },
    select: { id: true, username: true, email: true, role: true, createdAt: true },
  });
  if (!user) return c.json({ error: "User not found" }, 404);
  return c.json({ user });
});

users.post("/", zValidator("json", userSchemas.create), async (c) => {
  const prisma = c.get("prisma");
  const data = c.req.valid("json");
  const existing = await prisma.user.findFirst({
    where: { OR: [{ email: data.email }, { username: data.username }] },
  });
  if (existing) {
    return c.json({
      error: "User already exists",
      field: existing.email === data.email ? "email" : "username",
    }, 409);
  }
  const user = await prisma.user.create({
    data: { ...data, passwordHash: await hashPassword(data.password) },
    select: { id: true, username: true, email: true, role: true, createdAt: true },
  });
  return c.json({ user }, 201);
});

users.put("/:id", requireAuth, zValidator("json", userSchemas.update), async (c) => {
  const prisma = c.get("prisma");
  const id = c.req.param("id");
  const data = c.req.valid("json");
  const user = await prisma.user.update({
    where: { id },
    data,
    select: { id: true, username: true, email: true, role: true, updatedAt: true },
  });
  return c.json({ user });
});

users.delete("/:id", requireAuth, async (c) => {
  const prisma = c.get("prisma");
  const id = c.req.param("id");
  await prisma.user.delete({ where: { id } });
  return c.body(null, 204);
});

app.route("/users", users);

async function hashPassword(password: string): Promise<string> {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hash = await crypto.subtle.digest("SHA-256", data);
  return Array.from(new Uint8Array(hash)).map(b => b.toString(16).padStart(2, "0")).join("");
}

export default { port: 3000, fetch: app.fetch };
```

---

## 数据库集成详解

### Prisma 集成

```prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  role      Role     @default(USER)
  posts     Post[]
  comments  Comment[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

enum Role {
  ADMIN
  USER
  MODERATOR
}

model Post {
  id        String    @id @default(uuid())
  title     String
  content   String    @db.Text
  published Boolean   @default(false)
  tags      String[]  @default([])
  author    User      @relation(fields: [authorId], references: [id])
  authorId  String
  comments  Comment[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Comment {
  id        String    @id @default(uuid())
  content   String
  post      Post      @relation(fields: [postId], references: [id])
  postId    String
  author    User      @relation(fields: [authorId], references: [id])
  authorId  String
  parent    Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies   Comment[] @relation("CommentReplies")
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}
```

### Drizzle ORM 集成

```typescript
import { pgTable, text, varchar, boolean, timestamp, uuid } from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  username: varchar("username", { length: 255 }).notNull().unique(),
  email: varchar("email", { length: 255 }).notNull().unique(),
  password: text("password").notNull(),
  role: varchar("role", { length: 50 }).notNull().default("USER"),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

export const posts = pgTable("posts", {
  id: uuid("id").primaryKey().defaultRandom(),
  title: varchar("title", { length: 500 }).notNull(),
  content: text("content").notNull(),
  published: boolean("published").notNull().default(false),
  authorId: uuid("author_id").notNull().references(() => users.id),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
}));

export const postsRelations = relations(posts, ({ one }) => ({
  author: one(users, { fields: [posts.authorId], references: [users.id] }),
}));
```

---

## 认证授权方案详解

### JWT 实现

```typescript
import { SignJWT, jwtVerify, JWTPayload } from "jose";

const JWT_SECRET = new TextEncoder().encode(
  process.env.JWT_SECRET || "your-secret-key-at-least-32-characters"
);

interface TokenPayload extends JWTPayload {
  sub: string;
  email: string;
  role: string;
}

export class AuthService {
  async generateAccessToken(user: { id: string; email: string; role: string }): Promise<string> {
    const now = Math.floor(Date.now() / 1000);
    return await new SignJWT({
      sub: user.id,
      email: user.email,
      role: user.role,
    })
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt(now)
      .setExpirationTime(now + 15 * 60)
      .sign(JWT_SECRET);
  }
  
  async verifyAccessToken(token: string): Promise<TokenPayload | null> {
    try {
      const { payload } = await jwtVerify(token, JWT_SECRET);
      return payload as TokenPayload;
    } catch {
      return null;
    }
  }
}

export function authMiddleware() {
  return async (c: any, next: () => Promise<void>) => {
    const authHeader = c.req.header("Authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return c.json({ error: "Unauthorized" }, 401);
    }
    const token = authHeader.substring(7);
    const authService = new AuthService();
    const payload = await authService.verifyAccessToken(token);
    if (!payload) {
      return c.json({ error: "Invalid or expired token" }, 401);
    }
    c.set("user", { id: payload.sub, email: payload.email, role: payload.role });
    await next();
  };
}
```

---

## 部署配置（Docker）

```dockerfile
# Dockerfile
FROM oven/bun:1.2-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package.json bun.lockb* ./
RUN bun install --frozen-lockfile

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN bun build src/index.ts --target=bun --outfile=dist/index.js --minify

FROM base AS runner
WORKDIR /app
RUN addgroup --system --gid 1001 nodejs && adduser --system --uid 1001 bun
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
COPY package.json ./
ENV NODE_ENV=production
USER bun
EXPOSE 3000
CMD ["bun", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: "3.9"
services:
  app:
    build: .
    image: myapp:latest
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
  postgres:
    image: postgres:16-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
```

---

## 性能优化详解

### Hono 性能最佳实践

```typescript
import { Hono } from "hono";

const app = new Hono();

// 精确路由优先
app.get("/api/users", handler);
app.get("/api/users/:id", handler);
app.get("/api/*", handler);

// 内存缓存
const cache = new Map<string, { data: any; expiresAt: number }>();

app.get("/api/data", async (c) => {
  const cached = cache.get("data");
  if (cached && cached.expiresAt > Date.now()) {
    return c.json(cached.data, { headers: { "X-Cache": "HIT" } });
  }
  const data = await fetchData();
  cache.set("data", { data, expiresAt: Date.now() + 60000 });
  return c.json(data);
});

// 使用 include 避免 N+1
app.get("/posts", async (c) => {
  const posts = await prisma.post.findMany({
    where: { published: true },
    include: {
      author: { select: { id: true, username: true } },
      _count: { select: { comments: true } },
    },
  });
  return c.json(posts);
});

// 流式响应
app.get("/large-data", async (c) => {
  const stream = new ReadableStream({
    async start(controller) {
      const encoder = new TextEncoder();
      for await (const chunk of generateLargeData()) {
        controller.enqueue(encoder.encode(JSON.stringify(chunk) + "\n"));
      }
      controller.close();
    },
  });
  return new Response(stream, { headers: { "Content-Type": "application/x-ndjson" } });
});
```

---

## 常见陷阱与最佳实践

### 陷阱详解

#### 陷阱 1：异步中间件错误

```typescript
// ❌ 错误
app.use("*", async (c, next) => {
  c.set("user", await getUser());
  await next();
});

// ✅ 正确
app.use("*", async (c, next) => {
  await next();
  c.set("user", await getUser());
});
```

#### 陷阱 2：响应后修改

```typescript
// ❌ 错误
app.get("/example", async (c) => {
  await next();
  c.res.headers.set("X-Custom", "value");
});

// ✅ 正确
app.use("*", async (c, next) => {
  await next();
  c.res.headers.set("X-Custom", "value");
});
```

#### 陷阱 3：不安全的错误处理

```typescript
// ❌ 错误
app.onError((err, c) => {
  return c.json({ error: err.message, stack: err.stack }, 500);
});

// ✅ 正确
app.onError((err, c) => {
  console.error("Error:", err);
  if (process.env.NODE_ENV === "development") {
    return c.json({ error: err.message, stack: err.stack }, 500);
  }
  return c.json({ error: "Internal server error" }, 500);
});
```

### 最佳实践清单

1. **路由组织**：使用路由分组和嵌套路由
2. **中间件顺序**：将通用中间件放在前面
3. **错误处理**：实现全局错误处理和 404 处理
4. **类型安全**：使用 TypeScript 和 Zod 进行类型检查
5. **性能优化**：使用缓存、避免 N+1 查询
6. **安全性**：验证输入、实施 CORS
7. **可测试性**：为中间件和处理器编写单元测试
8. **日志记录**：记录请求和错误日志
9. **环境配置**：使用环境变量管理配置
10. **监控**：集成健康检查和指标监控

---

## 与同类框架对比

### Hono vs Express vs Fastify 详细对比

| 维度 | Hono | Express | Fastify |
|------|------|---------|---------|
| **核心定位** | 轻量跨平台 | 简单灵活 | 高性能插件化 |
| **体积** | ~14KB | ~500KB | ~300KB |
| **性能** | 极高 | 中等 | 高 |
| **路由** | Radix Tree | 分层 | 快速路由 |
| **中间件** | 函数式 | 函数式 | 封装器 |
| **类型安全** | TypeScript 原生 | 需手动 | 插件化 |
| **插件生态** | 成长中 | 庞大 | 丰富 |
| **学习曲线** | 低 | 低 | 中 |
| **跨运行时** | 12+ | 仅 Node.js | 仅 Node.js |

### 适用场景分析

| 场景 | 推荐框架 | 原因 |
|------|----------|------|
| 边缘计算 | **Hono** | 跨运行时、轻量 |
| Cloudflare Workers | **Hono** | 原生支持 |
| 企业级 Node.js | Fastify | 性能、插件丰富 |
| 快速原型 | Express/Hono | 简单易用 |
| 微服务 | Hono/Fastify | 高性能 |
| Serverless | Hono | 轻量、冷启动快 |

---

> [!SUCCESS]
> 本文档全面覆盖了 Hono 的核心知识，包括设计哲学、跨运行时架构、路由系统、中间件体系、Zod 验证和 JSX 支持。Hono 在 2026 年已成为边缘计算和跨平台开发的首选框架，其极小体积、极速性能和广泛的运行时支持使其在 vibecoding 技术栈中占据独特地位。对于需要部署到 Cloudflare Workers 或多运行时的应用，Hono 是唯一的选择。

---

## 完整安装与环境配置

### 环境要求

```bash
# 根据目标运行时不同，要求也不同
# Node.js: >= 16
# Bun: >= 1.0
# Deno: >= 1.0
# Cloudflare Workers: 无需安装
```

### 安装方法

```bash
# npm
npm install hono

# yarn
yarn add hono

# pnpm
pnpm add hono

# bun
bun add hono
```

### 项目模板

```bash
# 创建 Cloudflare Workers 项目
npm create hono@latest my-app
# 选择 cloudflare-workers

# 创建 Node.js 项目
npm create hono@latest my-app
# 选择 node-server

# 创建 Bun 项目
npm create hono@latest my-app
# 选择 bun

# 创建 Deno 项目
deno run -A https://crux.land/new@latest my-app
```

### package.json 配置

```json
{
  "name": "my-hono-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "bun run src/index.ts",
    "start": "bun run src/index.ts",
    "deploy": "wrangler deploy"
  },
  "dependencies": {
    "hono": "^4.0.0",
    "@hono/node-server": "^1.0.0",
    "@hono/cloudflare-workers": "^1.0.0",
    "@hono/zod-openapi": "^0.9.0",
    "zod": "^3.22.4",
    "@prisma/client": "^6.0.0"
  },
  "devDependencies": {
    "@cloudflare/workers-types": "^4.0.0",
    "wrangler": "^3.0.0",
    "typescript": "^5.3.0"
  }
}
```

---

## 数据库集成

### Prisma with Hono

```bash
npm install prisma @prisma/client
npx prisma init
```

```typescript
// src/db.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export { prisma };

// 对于 Cloudflare Workers，使用 @prisma/client
// 对于 Edge Runtime，使用 @prisma/client/edge
```

### Drizzle with Hono

```typescript
// src/db/schema.ts
import { text, integer, sqliteTable } from 'drizzle-orm/sqlite-core';
import { drizzle } from 'drizzle-orm/better-sqlite3';

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),
  email: text('email').notNull().unique(),
  username: text('username').notNull().unique(),
  createdAt: integer('created_at', { mode: 'timestamp' }).$defaultFn(() => new Date())
});

export const db = drizzle(new Database('app.db'));
```

---

## 认证授权

### JWT 实现

```typescript
// src/auth.ts
import { Context, Next } from 'hono';
import { SignJWT, jwtVerify } from 'jose';

const JWT_SECRET = new TextEncoder().encode(
  Bun?.env?.JWT_SECRET || Deno.env.get('JWT_SECRET') || 'secret'
);

export interface JWTPayload {
  sub: string;
  email: string;
  role: string;
}

export async function createToken(payload: JWTPayload): Promise<string> {
  return await new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(JWT_SECRET);
}

export async function verifyToken(token: string): Promise<JWTPayload | null> {
  try {
    const { payload } = await jwtVerify(token, JWT_SECRET);
    return payload as unknown as JWTPayload;
  } catch {
    return null;
  }
}

export function authMiddleware() {
  return async (c: Context, next: Next) => {
    const authHeader = c.req.header('Authorization');
    
    if (!authHeader?.startsWith('Bearer ')) {
      return c.json({ error: 'Unauthorized' }, 401);
    }

    const token = authHeader.substring(7);
    const payload = await verifyToken(token);

    if (!payload) {
      return c.json({ error: 'Invalid token' }, 401);
    }

    c.set('user', payload);
    await next();
  };
}

// 使用
app.get('/profile', authMiddleware(), async (c) => {
  const user = c.get('user');
  return c.json({ user });
});
```

---

## API 文档生成

### Zod OpenAPI 集成

```typescript
// src/docs.ts
import { OpenAPIHono } from '@hono/zod-openapi';
import { z } from 'zod';

const app = new OpenAPIHono();

// 定义 Schema
const UserSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  username: z.string().min(3).max(30)
});

const CreateUserSchema = z.object({
  email: z.string().email(),
  username: z.string().min(3).max(30),
  password: z.string().min(8)
});

// 注册路由并自动生成 OpenAPI 文档
app.openapi('GET /users', async (c) => {
  const users = await prisma.user.findMany();
  return c.json(users);
}, {
  responses: {
    200: {
      description: 'List of users',
      content: {
        'application/json': {
          schema: z.array(UserSchema)
        }
      }
    }
  }
});

app.openapi('POST /users', async (c) => {
  const body = await c.req.valid('json');
  const user = await prisma.user.create({ data: body });
  return c.json(user, 201);
}, {
  body: CreateUserSchema,
  responses: {
    201: {
      description: 'User created',
      content: {
        'application/json': {
          schema: UserSchema
        }
      }
    }
  }
});

// 获取 OpenAPI 文档
app.doc('/doc', {
  openapi: '3.0.0',
  info: {
    title: 'My API',
    version: '1.0.0'
  }
});
```

---

## 常见陷阱与最佳实践

### 陷阱 1：运行时特定 API

```typescript
// ❌ 错误：使用特定运行时的 API
app.get('/file', async (c) => {
  const file = Bun.file('data.json');
  return c.body(file);
});

// ✅ 正确：使用跨运行时兼容的方式
app.get('/file', async (c) => {
  const data = await fetch('https://api.example.com/data').then(r => r.json());
  return c.json(data);
});
```

### 陷阱 2：忘记处理错误

```typescript
// ❌ 错误
app.get('/user/:id', async (c) => {
  const user = await prisma.user.findUnique({
    where: { id: c.req.param('id') }
  });
  return c.json(user);
});

// ✅ 正确
app.get('/user/:id', async (c) => {
  const user = await prisma.user.findUnique({
    where: { id: c.req.param('id') }
  });
  
  if (!user) {
    return c.json({ error: 'User not found' }, 404);
  }
  
  return c.json(user);
});
```

### 最佳实践清单

1. **选择正确的适配器**：根据目标运行时选择合适的适配器。
2. **类型安全**：使用 Zod 进行输入验证。
3. **中间件组合**：使用 `compose` 组合多个中间件。
4. **错误处理**：使用 `onError` 统一错误处理。
5. **性能**：利用边缘计算的地理位置优势。
6. **兼容性测试**：在目标运行时测试代码。
7. **保持轻量**：不要添加不必要的依赖。

---

## 与其他框架对比

### Hono vs Express

| 特性 | Hono | Express |
|------|------|---------|
| **性能** | 极快 | 快 |
| **体积** | 极小 | 较大 |
| **跨运行时** | 支持 | 不支持 |
| **边缘计算** | 支持 | 不支持 |
| **中间件** | 兼容 | 专属 |
| **学习曲线** | 平缓 | 平缓 |

### Hono vs Fastify

| 特性 | Hono | Fastify |
|------|------|---------|
| **性能** | 极快 | 极快 |
| **跨运行时** | 支持 | 不支持 |
| **边缘计算** | 支持 | 不支持 |
| **插件生态** | 增长中 | 成熟 |
| **Schema 验证** | Zod | JSON Schema |
| **适用场景** | 多平台 | Node.js |

### Hono vs Elysia

| 特性 | Hono | Elysia |
|------|------|--------|
| **性能** | 极快 | 极快 |
| **类型安全** | Zod | TypeScript 原生 |
| **跨运行时** | 支持 | 仅 Bun |
| **中间件** | 灵活 | 装饰器 |
| **学习曲线** | 平缓 | 中等 |

---

## 核心技术深入解析

### Hono Context 高级用法

Hono 的 Context 对象提供了丰富的功能，掌握这些高级用法可以大幅提升开发效率和应用质量。

#### Context 扩展与自定义

```typescript
import { Hono, Context } from "hono";
import { getCookie, setCookie, deleteCookie } from "hono/cookie";

// ==================== 扩展 Context 类型 ====================

// 定义变量类型
interface Variables {
  requestId: string;
  startTime: number;
  user: User | null;
  db: Database;
  cache: Cache;
}

interface User {
  id: string;
  email: string;
  role: string;
}

// 扩展 Environment
interface Env {
  Variables: Variables;
  Bindings: {
    KV: KVNamespace;
    AI: Ai;
    DATABASE_URL: string;
  };
}

const app = new Hono<Env>();

// ==================== 中间件中设置变量 ====================

// 请求 ID 中间件
app.use("*", async (c, next) => {
  const requestId = c.req.header("X-Request-ID") || crypto.randomUUID();
  c.set("requestId", requestId);
  c.set("startTime", Date.now());
  await next();
});

// 认证中间件
app.use("/api/*", async (c, next) => {
  const token = c.req.header("Authorization")?.replace("Bearer ", "");
  
  if (token) {
    try {
      const user = await verifyToken(token);
      c.set("user", user);
    } catch {
      c.set("user", null);
    }
  } else {
    c.set("user", null);
  }
  
  await next();
});

// 数据库中间件
app.use("*", async (c, next) => {
  const db = new Database(c.env.DATABASE_URL);
  c.set("db", db);
  
  await next();
  
  db.close();
});

// ==================== Context 辅助方法 ====================

// JSON 响应（带元数据）
app.get("/api/data", async (c) => {
  const data = await fetchData();
  
  return c.json({
    data,
    meta: {
      requestId: c.get("requestId"),
      timestamp: new Date().toISOString(),
      duration: Date.now() - c.get("startTime"),
    },
  });
});

// Cookie 操作
app.get("/preferences", async (c) => {
  const theme = getCookie(c, "theme") || "light";
  const language = getCookie(c, "language") || "en";
  
  return c.json({ theme, language });
});

app.post("/preferences", async (c) => {
  const { theme, language } = await c.req.json();
  
  // 设置 Cookie
  setCookie(c, "theme", theme, {
    path: "/",
    maxAge: 60 * 60 * 24 * 365, // 1 年
    httpOnly: true,
    secure: true,
    sameSite: "Strict",
  });
  
  setCookie(c, "language", language, {
    path: "/",
    maxAge: 60 * 60 * 24 * 365,
    httpOnly: false, // 可以被 JavaScript 访问
    secure: true,
    sameSite: "Lax",
  });
  
  return c.json({ success: true });
});

app.delete("/preferences", async (c) => {
  deleteCookie(c, "theme");
  deleteCookie(c, "language");
  
  return c.json({ success: true });
});

// ==================== 内容协商 ====================

app.get("/api/resource", async (c) => {
  const accept = c.req.header("Accept");
  
  if (accept?.includes("application/xml")) {
    return c.xml({ resource: { id: 1, name: "Test" } });
  }
  
  if (accept?.includes("text/html")) {
    return c.html(`<html><body><h1>Resource</h1></body></html>`);
  }
  
  return c.json({ id: 1, name: "Test" });
});

// ==================== 流式响应 ====================

app.get("/api/stream", async (c) => {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    async start(controller) {
      // 发送初始数据
      controller.enqueue(encoder.encode(JSON.stringify({ type: "start" }) + "\n"));
      
      // 模拟数据生成
      for (let i = 0; i < 10; i++) {
        await new Promise(resolve => setTimeout(resolve, 1000));
        const data = JSON.stringify({ type: "data", index: i, timestamp: Date.now() });
        controller.enqueue(encoder.encode(data + "\n"));
      }
      
      // 发送完成
      controller.enqueue(encoder.encode(JSON.stringify({ type: "end" }) + "\n"));
      controller.close();
    },
  });
  
  return new Response(stream, {
    headers: {
      "Content-Type": "application/x-ndjson",
      "Transfer-Encoding": "chunked",
      "Cache-Control": "no-cache",
      "Connection": "keep-alive",
    },
  });
});

// ==================== 请求重定向 ====================

app.get("/old-page", async (c) => {
  return c.redirect("/new-page", 301);
});

app.get("/preserve-query", async (c) => {
  const url = new URL(c.req.url);
  return c.redirect(`/new-page${url.search}`, 302);
});

// ==================== 条件请求处理 ====================

app.get("/api/resource/:id", async (c) => {
  const id = c.req.param("id");
  const resource = await getResource(id);
  
  if (!resource) {
    return c.json({ error: "Not found" }, 404);
  }
  
  // ETag 支持
  const etag = `"${resource.version}"`;
  const ifNoneMatch = c.req.header("If-None-Match");
  
  if (ifNoneMatch === etag) {
    return c.body(null, 304);
  }
  
  return c.json(resource, {
    headers: {
      "ETag": etag,
      "Cache-Control": "private, max-age=60, must-revalidate",
      "Last-Modified": new Date(resource.updatedAt).toUTCString(),
    },
  });
});

// ==================== 速率限制响应头 ====================

app.use("/api/*", async (c, next) => {
  const ip = c.req.header("CF-Connecting-IP") || c.req.header("X-Forwarded-For") || "unknown";
  const key = `ratelimit:${ip}`;
  
  const current = await REDIS.get(key);
  const limit = 100;
  const window = 60; // 60 秒
  
  if (current && parseInt(current) >= limit) {
    return c.json({
      error: "Rate limit exceeded",
      retryAfter: await REDIS.ttl(key),
    }, 429, {
      headers: {
        "X-RateLimit-Limit": String(limit),
        "X-RateLimit-Remaining": "0",
        "X-RateLimit-Reset": String(Date.now() + window * 1000),
        "Retry-After": String(await REDIS.ttl(key)),
      },
    });
  }
  
  if (current) {
    await REDIS.incr(key);
  } else {
    await REDIS.setex(key, window, "1");
  }
  
  await next();
  
  // 在响应中添加速率限制头
  const remaining = current ? limit - parseInt(current) - 1 : limit - 1;
  c.res.headers.set("X-RateLimit-Limit", String(limit));
  c.res.headers.set("X-RateLimit-Remaining", String(Math.max(0, remaining)));
});
```

### Hono 测试策略

#### 单元测试

```typescript
// src/__tests__/routes/users.test.ts
import { describe, test, expect, beforeEach, afterEach, vi } from "bun:test";
import { Hono } from "hono";
import { userRoutes } from "../routes/users";
import { createMockUser, createMockContext } from "./helpers";

// 创建测试应用
function createTestApp() {
  const app = new Hono();
  app.route("/users", userRoutes);
  return app;
}

describe("User Routes", () => {
  let app: Hono;
  
  beforeEach(() => {
    app = createTestApp();
    // 重置模拟数据
    mockDatabase.users = [];
  });
  
  describe("GET /users", () => {
    test("should return empty array when no users", async () => {
      const res = await app.request("/users");
      
      expect(res.status).toBe(200);
      const json = await res.json();
      expect(json.users).toEqual([]);
    });
    
    test("should return all users", async () => {
      mockDatabase.users = [
        createMockUser({ id: "1", email: "alice@example.com" }),
        createMockUser({ id: "2", email: "bob@example.com" }),
      ];
      
      const res = await app.request("/users");
      
      expect(res.status).toBe(200);
      const json = await res.json();
      expect(json.users).toHaveLength(2);
      expect(json.users[0].email).toBe("alice@example.com");
    });
    
    test("should support pagination", async () => {
      mockDatabase.users = Array.from({ length: 25 }, (_, i) =>
        createMockUser({ id: String(i + 1), email: `user${i + 1}@example.com` })
      );
      
      const res = await app.request("/users?page=2&limit=10");
      
      expect(res.status).toBe(200);
      const json = await res.json();
      expect(json.users).toHaveLength(10);
      expect(json.pagination.page).toBe(2);
      expect(json.pagination.total).toBe(25);
    });
  });
  
  describe("GET /users/:id", () => {
    test("should return user by id", async () => {
      const user = createMockUser({ id: "1", email: "test@example.com" });
      mockDatabase.users = [user];
      
      const res = await app.request("/users/1");
      
      expect(res.status).toBe(200);
      const json = await res.json();
      expect(json.user.email).toBe("test@example.com");
    });
    
    test("should return 404 for non-existent user", async () => {
      const res = await app.request("/users/999");
      
      expect(res.status).toBe(404);
      const json = await res.json();
      expect(json.error).toBe("User not found");
    });
    
    test("should return 400 for invalid id", async () => {
      const res = await app.request("/users/invalid");
      
      expect(res.status).toBe(400);
    });
  });
  
  describe("POST /users", () => {
    test("should create user with valid data", async () => {
      const userData = {
        username: "newuser",
        email: "new@example.com",
        password: "SecurePass123!",
      };
      
      const res = await app.request("/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(userData),
      });
      
      expect(res.status).toBe(201);
      const json = await res.json();
      expect(json.user.username).toBe("newuser");
      expect(json.user.email).toBe("new@example.com");
      expect(json.user.password).toBeUndefined(); // 密码不应返回
    });
    
    test("should return 400 for invalid email", async () => {
      const res = await app.request("/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          username: "testuser",
          email: "invalid-email",
          password: "password123",
        }),
      });
      
      expect(res.status).toBe(400);
      const json = await res.json();
      expect(json.error).toContain("email");
    });
    
    test("should return 409 for duplicate email", async () => {
      mockDatabase.users = [createMockUser({ email: "existing@example.com" })];
      
      const res = await app.request("/users", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          username: "newuser",
          email: "existing@example.com",
          password: "password123",
        }),
      });
      
      expect(res.status).toBe(409);
    });
  });
  
  describe("PUT /users/:id", () => {
    test("should update user", async () => {
      const user = createMockUser({ id: "1", username: "oldname" });
      mockDatabase.users = [user];
      
      const res = await app.request("/users/1", {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username: "newname" }),
      });
      
      expect(res.status).toBe(200);
      const json = await res.json();
      expect(json.user.username).toBe("newname");
    });
    
    test("should return 404 for non-existent user", async () => {
      const res = await app.request("/users/999", {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ username: "newname" }),
      });
      
      expect(res.status).toBe(404);
    });
  });
  
  describe("DELETE /users/:id", () => {
    test("should delete user", async () => {
      const user = createMockUser({ id: "1" });
      mockDatabase.users = [user];
      
      const res = await app.request("/users/1", { method: "DELETE" });
      
      expect(res.status).toBe(204);
      expect(mockDatabase.users).toHaveLength(0);
    });
    
    test("should return 404 for non-existent user", async () => {
      const res = await app.request("/users/999", { method: "DELETE" });
      
      expect(res.status).toBe(404);
    });
  });
});

// ==================== 集成测试 ====================

describe("User Integration Tests", () => {
  let app: Hono;
  let authToken: string;
  
  beforeAll(async () => {
    app = createTestApp();
    
    // 创建测试用户并获取 token
    await app.request("/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        username: "testuser",
        email: "test@example.com",
        password: "password123",
      }),
    });
    
    // 登录获取 token
    const loginRes = await app.request("/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        email: "test@example.com",
        password: "password123",
      }),
    });
    
    const loginData = await loginRes.json();
    authToken = loginData.accessToken;
  });
  
  test("full CRUD workflow", async () => {
    // 创建
    const createRes = await app.request("/users", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${authToken}`,
      },
      body: JSON.stringify({
        username: "cruduser",
        email: "crud@example.com",
        password: "password123",
      }),
    });
    expect(createRes.status).toBe(201);
    const created = await createRes.json();
    const userId = created.user.id;
    
    // 读取
    const readRes = await app.request(`/users/${userId}`, {
      headers: { "Authorization": `Bearer ${authToken}` },
    });
    expect(readRes.status).toBe(200);
    
    // 更新
    const updateRes = await app.request(`/users/${userId}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${authToken}`,
      },
      body: JSON.stringify({ username: "updateduser" }),
    });
    expect(updateRes.status).toBe(200);
    
    // 删除
    const deleteRes = await app.request(`/users/${userId}`, {
      method: "DELETE",
      headers: { "Authorization": `Bearer ${authToken}` },
    });
    expect(deleteRes.status).toBe(204);
    
    // 验证删除
    const verifyRes = await app.request(`/users/${userId}`);
    expect(verifyRes.status).toBe(404);
  });
});
```

### Hono 高级中间件模式

#### 复合中间件

```typescript
import { Hono, MiddlewareHandler } from "hono";
import { compress } from "hono/compress";
import { logger } from "hono/logger";
import { cors } from "hono/cors";
import { secureHeaders } from "hono/secure-headers";

// ==================== 中间件组合器 ====================

type MiddlewareFactory = (options?: any) => MiddlewareHandler;

function composeMiddleware(
  ...middlewares: MiddlewareHandler[]
): MiddlewareHandler {
  return async (c, next) => {
    let index = -1;
    
    async function dispatch(i: number): Promise<void> {
      if (i <= index) {
        throw new Error("next() called multiple times");
      }
      index = i;
      
      if (i < middlewares.length) {
        const handler = middlewares[i];
        await handler(c, () => dispatch(i + 1));
      } else {
        await next();
      }
    }
    
    await dispatch(0);
  };
}

// ==================== 条件中间件 ====================

function when(
  condition: (c: Context) => boolean,
  middleware: MiddlewareHandler
): MiddlewareHandler {
  return async (c, next) => {
    if (condition(c)) {
      await middleware(c, next);
    } else {
      await next();
    }
  };
}

// 使用示例
app.use(
  when(
    (c) => c.req.url.includes("/api/"),
    cors({ origin: ["https://example.com"] })
  )
);

app.use(
  when(
    (c) => process.env.NODE_ENV === "development",
    logger()
  )
);

// ==================== 中间件工厂 ====================

function createRateLimitMiddleware(options: {
  windowMs: number;
  max: number;
  keyGenerator?: (c: Context) => string;
  handler?: (c: Context) => Response;
}): MiddlewareHandler {
  const { windowMs, max, keyGenerator, handler } = options;
  const store = new Map<string, { count: number; resetTime: number }>();
  
  return async (c, next) => {
    const key = keyGenerator?.(c) || c.req.header("CF-Connecting-IP") || "anonymous";
    const now = Date.now();
    
    let record = store.get(key);
    
    if (!record || now > record.resetTime) {
      record = {
        count: 1,
        resetTime: now + windowMs,
      };
      store.set(key, record);
    } else {
      record.count++;
    }
    
    // 设置响应头
    c.res.headers.set("X-RateLimit-Limit", String(max));
    c.res.headers.set("X-RateLimit-Remaining", String(Math.max(0, max - record.count)));
    c.res.headers.set("X-RateLimit-Reset", String(record.resetTime));
    
    if (record.count > max) {
      return handler?.(c) || c.json({
        error: "Too many requests",
        retryAfter: Math.ceil((record.resetTime - now) / 1000),
      }, 429);
    }
    
    await next();
  };
}

// 使用
app.use("/api/*", createRateLimitMiddleware({
  windowMs: 60 * 1000,
  max: 100,
  keyGenerator: (c) => c.req.header("X-User-ID") || "anonymous",
}));

// ==================== 缓存中间件 ====================

function createCacheMiddleware(options: {
  cacheName: string;
  cacheControl: string;
  ttl?: number;
}): MiddlewareHandler {
  const { cacheControl, ttl } = options;
  const cache = new Map<string, { data: any; expiresAt: number }>();
  
  return async (c, next) => {
    const key = `${c.req.method}:${c.req.url}`;
    const cached = cache.get(key);
    
    if (cached && cached.expiresAt > Date.now()) {
      c.res.headers.set("X-Cache", "HIT");
      c.res.headers.set("Cache-Control", cacheControl);
      return c.json(cached.data);
    }
    
    c.res.headers.set("X-Cache", "MISS");
    
    await next();
    
    if (c.res.status === 200) {
      const data = await c.res.json();
      cache.set(key, {
        data,
        expiresAt: Date.now() + (ttl || 60000),
      });
      
      // 重新设置响应头
      c.res.headers.set("Cache-Control", cacheControl);
    }
  };
}

// ==================== 验证中间件工厂 ====================

function createValidatorMiddleware<T>(
  schema: z.ZodSchema<T>,
  source: "json" | "query" | "param"
) {
  return async (c: Context, next: () => Promise<void>) => {
    let data: any;
    
    try {
      switch (source) {
        case "json":
          data = await c.req.json();
          break;
        case "query":
          data = Object.fromEntries(c.req.query());
          break;
        case "param":
          data = c.req.param();
          break;
      }
      
      const result = schema.safeParse(data);
      
      if (!result.success) {
        return c.json({
          error: "Validation failed",
          details: result.error.errors,
        }, 400);
      }
      
      // 将验证后的数据存储到 context
      c.set("validated", result.data);
    } catch (error) {
      return c.json({
        error: "Invalid request body",
      }, 400);
    }
    
    await next();
  };
}

// 使用
const createUserSchema = z.object({
  username: z.string().min(3),
  email: z.string().email(),
  password: z.string().min(8),
});

app.post(
  "/users",
  createValidatorMiddleware(createUserSchema, "json"),
  async (c) => {
    const { username, email, password } = c.get("validated");
    // ... 创建用户逻辑
  }
);
```

### Hono 与外部服务集成

#### 外部 API 客户端

```typescript
// src/lib/api-client.ts
import { Hono } from "hono";

interface ApiClientOptions {
  baseUrl: string;
  headers?: Record<string, string>;
  timeout?: number;
  retry?: {
    attempts: number;
    delay: number;
  };
}

class ApiClient {
  private baseUrl: string;
  private headers: Record<string, string>;
  private timeout: number;
  private retry: { attempts: number; delay: number };

  constructor(options: ApiClientOptions) {
    this.baseUrl = options.baseUrl;
    this.headers = options.headers || {};
    this.timeout = options.timeout || 30000;
    this.retry = options.retry || { attempts: 1, delay: 1000 };
  }

  private async request<T>(
    method: string,
    path: string,
    options?: {
      body?: any;
      params?: Record<string, string>;
      headers?: Record<string, string>;
    }
  ): Promise<T> {
    let url = `${this.baseUrl}${path}`;
    
    // 添加查询参数
    if (options?.params) {
      const searchParams = new URLSearchParams(options.params);
      url += `?${searchParams.toString()}`;
    }

    const headers = {
      ...this.headers,
      ...options?.headers,
      "Content-Type": "application/json",
    };

    let lastError: Error | undefined;
    
    for (let attempt = 1; attempt <= this.retry.attempts; attempt++) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.timeout);

        const response = await fetch(url, {
          method,
          headers,
          body: options?.body ? JSON.stringify(options.body) : undefined,
          signal: controller.signal,
        });

        clearTimeout(timeoutId);

        if (!response.ok) {
          const error = new Error(`HTTP ${response.status}: ${response.statusText}`);
          (error as any).status = response.status;
          (error as any).response = await response.json().catch(() => ({}));
          throw error;
        }

        return response.json();
      } catch (error) {
        lastError = error as Error;
        
        if (attempt < this.retry.attempts) {
          await new Promise(resolve => setTimeout(resolve, this.retry.delay * attempt));
        }
      }
    }

    throw lastError;
  }

  async get<T>(path: string, options?: { params?: Record<string, string> }): Promise<T> {
    return this.request<T>("GET", path, options);
  }

  async post<T>(path: string, body?: any): Promise<T> {
    return this.request<T>("POST", path, { body });
  }

  async put<T>(path: string, body?: any): Promise<T> {
    return this.request<T>("PUT", path, { body });
  }

  async patch<T>(path: string, body?: any): Promise<T> {
    return this.request<T>("PATCH", path, { body });
  }

  async delete<T>(path: string): Promise<T> {
    return this.request<T>("DELETE", path);
  }
}

// ==================== 在 Hono 中使用 ====================

const app = new Hono();

// 创建 API 客户端
const githubClient = new ApiClient({
  baseUrl: "https://api.github.com",
  headers: {
    "Accept": "application/vnd.github.v3+json",
  },
  timeout: 10000,
  retry: {
    attempts: 3,
    delay: 1000,
  },
});

const stripeClient = new ApiClient({
  baseUrl: "https://api.stripe.com/v1",
  headers: {
    "Authorization": `Bearer ${process.env.STRIPE_SECRET_KEY}`,
  },
});

// GitHub 集成路由
app.get("/github/users/:username", async (c) => {
  const { username } = c.req.param();
  
  try {
    const user = await githubClient.get(`/users/${username}`);
    return c.json(user);
  } catch (error) {
    if ((error as any).status === 404) {
      return c.json({ error: "User not found" }, 404);
    }
    return c.json({ error: "GitHub API error" }, 502);
  }
});

app.get("/github/repos/:owner/:repo", async (c) => {
  const { owner, repo } = c.req.param();
  
  const [repoData, issues, prs] = await Promise.all([
    githubClient.get(`/repos/${owner}/${repo}`),
    githubClient.get(`/repos/${owner}/${repo}/issues`, { params: { state: "open", per_page: "5" } }),
    githubClient.get(`/repos/${owner}/${repo}/pulls`, { params: { state: "open", per_page: "5" } }),
  ]);
  
  return c.json({ repo: repoData, recentIssues: issues, recentPRs: prs });
});

// Stripe 集成路由
app.post("/payments/create-intent", async (c) => {
  const { amount, currency = "usd", metadata = {} } = await c.req.json();
  
  try {
    const paymentIntent = await stripeClient.post("/payment_intents", {
      amount: Math.round(amount * 100), // Stripe 使用分
      currency,
      metadata,
    });
    
    return c.json({
      clientSecret: paymentIntent.client_secret,
      id: paymentIntent.id,
    });
  } catch (error) {
    console.error("Stripe error:", error);
    return c.json({ error: "Payment failed" }, 500);
  }
});

app.get("/payments/:intentId", async (c) => {
  const { intentId } = c.req.param();
  
  try {
    const paymentIntent = await stripeClient.get(`/payment_intents/${intentId}`);
    return c.json(paymentIntent);
  } catch (error) {
    return c.json({ error: "Payment not found" }, 404);
  }
});
```

### Hono 与前端集成

#### 统一 API 响应

```typescript
// src/lib/response.ts
// 统一 API 响应格式和工具函数

interface SuccessResponse<T> {
  success: true;
  data: T;
  meta?: {
    timestamp: string;
    requestId: string;
    pagination?: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
  };
}

interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, any>;
  };
  meta?: {
    timestamp: string;
    requestId: string;
  };
}

type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

// 响应构建器
class ResponseBuilder {
  private response: Response;
  
  constructor() {
    this.response = new Response("", {
      status: 200,
      headers: { "Content-Type": "application/json" },
    });
  }
  
  status(code: number): this {
    this.response = new Response(this.response.body, {
      status: code,
      headers: this.response.headers,
    });
    return this;
  }
  
  json<T>(data: T, meta?: SuccessResponse<T>["meta"]): Response {
    const response: SuccessResponse<T> = {
      success: true,
      data,
      meta: meta || {
        timestamp: new Date().toISOString(),
        requestId: crypto.randomUUID(),
      },
    };
    
    return new Response(JSON.stringify(response), {
      status: this.response.status,
      headers: {
        ...Object.fromEntries(this.response.headers.entries()),
        "Content-Type": "application/json",
      },
    });
  }
  
  error(code: string, message: string, details?: Record<string, any>): Response {
    const response: ErrorResponse = {
      success: false,
      error: { code, message, details },
      meta: {
        timestamp: new Date().toISOString(),
        requestId: crypto.randomUUID(),
      },
    };
    
    return new Response(JSON.stringify(response), {
      status: this.response.status,
      headers: {
        ...Object.fromEntries(this.response.headers.entries()),
        "Content-Type": "application/json",
      },
    });
  }
}

// 快捷响应函数
function success<T>(
  data: T,
  options?: {
    status?: number;
    headers?: Record<string, string>;
    pagination?: SuccessResponse<T>["meta"]["pagination"];
  }
): Response {
  const builder = new ResponseBuilder();
  
  const responseInit: ResponseInit = {
    status: options?.status || 200,
    headers: {
      "Content-Type": "application/json",
      ...options?.headers,
    },
  };
  
  const response: SuccessResponse<T> = {
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
      ...(options?.pagination && { pagination: options.pagination }),
    },
  };
  
  return new Response(JSON.stringify(response), responseInit);
}

function error(
  code: string,
  message: string,
  status: number = 400,
  details?: Record<string, any>
): Response {
  const response: ErrorResponse = {
    success: false,
    error: { code, message, details },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
    },
  };
  
  return new Response(JSON.stringify(response), {
    status,
    headers: { "Content-Type": "application/json" },
  });
}

// 使用示例
app.get("/api/users", async (c) => {
  const users = await getUsers();
  
  return success(users, {
    status: 200,
    pagination: {
      page: 1,
      limit: 10,
      total: 100,
      totalPages: 10,
    },
  });
});

app.post("/api/users", async (c) => {
  try {
    const user = await createUser(await c.req.json());
    return success(user, { status: 201 });
  } catch (err) {
    if (err instanceof ValidationError) {
      return error("VALIDATION_ERROR", err.message, 400, err.details);
    }
    return error("INTERNAL_ERROR", "Failed to create user", 500);
  }
});
```

---

## 附录：常用命令速查表

| 命令 | 说明 | 示例 |
|------|------|------|
| `bunx hono create` | 创建 Hono 项目 | `bunx hono create my-app` |
| `bun add hono` | 添加依赖 | `bun add hono` |
| `bunx prisma generate` | 生成 Prisma 客户端 | `bunx prisma generate` |
| `wrangler deploy` | 部署到 Cloudflare | `wrangler deploy` |
| `bun run src/index.ts` | 运行开发服务器 | `bun run src/index.ts` |

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Hono 官方文档 | https://hono.dev |
| Hono GitHub | https://github.com/honojs/hono |
| Hono Discord | https://discord.gg/hono |
| Hono 官方示例 | https://github.com/honojs/examples |

### 学习资源

| 资源 | 说明 |
|------|------|
| Hono Blog | 官方博客 |
| Hono Recipes | 官方食谱 |
| Cloudflare Workers 文档 | 边缘计算参考 |

### 社区

| 资源 | 说明 |
|------|------|
| Hono Reddit | r/honojs |
| Hono Twitter | 官方 Twitter |
| GitHub Discussions | 社区讨论 |

---

> [!NOTE]
> 本文档持续更新中，如有问题或建议，欢迎提交 Issue 或 PR。

