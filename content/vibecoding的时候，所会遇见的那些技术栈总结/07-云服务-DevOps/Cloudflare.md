---
date: 2026-04-24
tags:
  - Cloudflare
  - CDN
  - Edge Computing
  - Workers
  - Serverless
description: Cloudflare 产品完整指南，涵盖 CDN、Workers 边缘计算、D1 分布式 SQLite、R2 对象存储、AI Gateway，以及与 Vercel/Netlify 的对比。
---

# Cloudflare：CDN + 边缘计算 + Workers 完整指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Cloudflare Workers、D1、R2、KV、AI Gateway 等核心产品，以及边缘计算架构解析和选型建议。

---

## Cloudflare 是什么

### 从 CDN 到全栈平台

大多数人对 Cloudflare 的第一印象是"做 CDN 和 DNS 的"。确实，Cloudflare 最初就是以 CDN（内容分发网络）和 DDoS 防护起家的。但到了 2026 年，Cloudflare 已经发展成为一个几乎覆盖了所有云计算场景的全栈平台。

你可以把 Cloudflare 理解为一片覆盖全球的"云"，它最初专注于加速和保护网站（CDN + 安全），后来逐步扩展到边缘计算（Workers）、存储（ R2、D1、KV）、AI（AI Gateway、Workers AI）等领域。最让人惊喜的是，它的很多基础服务都有非常慷慨的免费额度，个人项目和小型应用几乎可以免费用很久。

### 核心产品全景图

Cloudflare 的产品可以分成几个大类：

**网络优化与安全**类产品包括 CDN（全球内容分发）、DDoS 防护（业界领先的能力）、WAF（Web 应用防火墙）、SSL/TLS 证书（免费的）、Argo Smart Routing（智能路由优化）和 Load Balancing（全球负载均衡）。这些是 Cloudflare 的立家之本，也是大部分人最先接触到的服务。

**边缘计算**类产品是 Cloudflare 近年来发展最快的方向。Workers 允许你在全球 300 多个城市运行 JavaScript/TypeScript/其他语言代码，响应时间极低。Workers KV 是边缘键值存储，适合读写频率不那么高的场景。Durable Objects 提供了有状态的边缘对象，适合需要保持连接状态的场景，比如聊天室或实时协作工具。Queues 则提供边缘消息队列的能力。

**存储与数据库**类产品包括 R2（兼容 S3 协议的对象存储，存储免费！）、D1（边缘 SQLite 数据库，全球分布式）、Hyperdrive（边缘数据库加速器）和 Queue（消息队列）。

**部署与运行时**类产品包括 Pages（类似 Vercel 的静态部署平台）和 Workers（边缘运行时）。

**AI 产品**类是 2024-2026 年的新增能力，包括 AI Gateway（AI API 的网关层，提供缓存、限流、路由等功能）和 Workers AI（直接在 Cloudflare 边缘运行 AI 推理）。

---

## Cloudflare Workers：边缘计算

### 什么是边缘计算

传统架构中，你的应用运行在一个或几个固定的数据中心里。用户发起请求后，请求需要跨越半个地球才能到达你的服务器。比如你的服务器在美国东海岸，法国的用户访问时，请求要跨越整个大西洋，然后再返回。

边缘计算把计算能力分布到了全球各个城市。Cloudflare Workers 的代码可以在全球 300 多个城市同时运行，每个城市都是一个小型的计算节点。用户请求会被路由到最近的节点，不需要千里迢迢跑到"中央服务器"。

这样做的效果是：响应延迟从几百毫秒降低到几十毫秒甚至几毫秒，用户体验大幅提升。

### Workers 的运行机制

Cloudflare Workers 基于 V8 JavaScript 引擎运行，每个 Worker 都在一个轻量级的 V8 隔离区（Isolate）中执行。相比传统的 Node.js 运行时，V8 Isolate 的冷启动时间接近于零——只需要几毫秒而不是几百毫秒。这意味着 Workers 可以真正做到"用完即走"，不需要为每个请求维护常驻进程。

```javascript
// workers/hello.js
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);

    if (url.pathname === "/api/hello") {
      return Response.json({
        message: "Hello from the edge!",
        country: request.cf?.country,
        city: request.cf?.city,
        timestamp: new Date().toISOString(),
      });
    }

    return new Response("Not Found", { status: 404 });
  },
};
```

这段代码部署到 Cloudflare Workers 后，会在全球所有节点同步运行。当有用户访问时，请求会被路由到距离用户最近的数据中心，由对应节点的 Worker 处理。

### 使用 Hono 框架构建 API

对于复杂的 API，使用框架比原生 Workers API 更高效。Hono 是一个轻量的 Web 框架，支持 Cloudflare Workers、Node.js、Deno 等多种运行时：

```javascript
// src/index.ts
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";

const app = new Hono();

// 中间件
app.use("*", logger());
app.use(
  "*",
  cors({
    origin: "https://example.com",
    allowMethods: ["GET", "POST", "PUT", "DELETE"],
  })
);

// 路由
app.get("/", (c) => c.text("Welcome to Cloudflare Workers API"));

app.get("/api/users/:id", async (c) => {
  const id = c.req.param("id");

  // 可以从这里读取 KV 或 D1 数据库
  return c.json({
    id,
    name: "User " + id,
    email: `user${id}@example.com`,
  });
});

app.post("/api/users", async (c) => {
  const body = await c.req.json();
  const { name, email } = body;

  if (!name || !email) {
    return c.json({ error: "Name and email are required" }, 400);
  }

  // 保存到 D1 数据库
  const result = await c.env.DB.prepare(
    "INSERT INTO users (name, email) VALUES (?, ?) RETURNING *"
  )
    .bind(name, email)
    .first();

  return c.json(result, 201);
});

app.get("/api/posts", async (c) => {
  const { searchParams } = new URL(c.req.url);
  const limit = searchParams.get("limit") || "10";

  const { results } = await c.env.DB.prepare(
    "SELECT * FROM posts ORDER BY created_at DESC LIMIT ?"
  )
    .bind(Number(limit))
    .all();

  return c.json({ posts: results });
});

// 错误处理
app.onError((err, c) => {
  console.error(err);
  return c.json({ error: "Internal Server Error" }, 500);
});

app.notFound((c) => {
  return c.json({ error: "Not Found" }, 404);
});

export default app;
```

### wrangler.toml 配置

```toml
# wrangler.toml
name = "my-workers-api"
main = "src/index.ts"
compatibility_date = "2024-01-01"

# 预览环境
[env.preview]
name = "my-workers-api-preview"

# 生产环境绑定
[env.production]
name = "my-workers-api-prod"

# D1 数据库绑定
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# KV 命名空间绑定
[[kv_namespaces]]
binding = "CACHE"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# R2 存储桶绑定
[[r2_buckets]]
binding = "ASSETS"
bucket_name = "my-assets"
```

### Workers 适用场景与局限

Workers 不是万能的，它有自己的适用场景和局限性：

| 适合 Workers 的场景 | 不适合 Workers 的场景 |
|------------------|-------------------|
| API 路由、身份验证 | CPU 密集型任务（如视频转码） |
| A/B 测试、中间件 | 需要大量本地存储 |
| 静态资源处理 | 长时间运行的后台任务（超过 30 秒） |
| Webhook 处理 | 需要特定系统库（如 PIL、ffmpeg）的任务 |
| 边缘渲染 | 复杂的关系型数据库查询（事务性要求高） |

对于 CPU 密集型任务，Cloudflare 提供了 Cloudflare Workers (Compute) 的增强版或者可以使用 Durable Objects 的 CPU 时间，或者考虑使用传统 Serverless Functions（如 Vercel Functions）。

---

## Cloudflare Pages：静态部署

### Pages vs Workers

很多人会把 Cloudflare Pages 和 Workers 搞混。简单来说：Pages 适合**静态网站和 SSR 应用**（Next.js、Nuxt、Remix 等），Workers 适合**纯 API 和边缘逻辑**。

Pages 的优势在于它和 GitHub 的深度集成——push 代码后自动构建和部署，预览每个 PR 的效果。它底层跑的还是 Workers 运行时，只是针对前端项目做了更友好的封装。

### 部署配置

```javascript
// functions/_middleware.ts
// 这个文件在 Pages 中作为中间件运行
export const onRequest = async (context) => {
  const response = await context.next();

  // 添加自定义头
  response.headers.set("X-Custom-Header", "Cloudflare Pages");

  return response;
};
```

```yaml
# _routes.json - 自定义路由规则
{
  "version": 1,
  "include": ["/*"],
  "exclude": ["/api/*"]
}
```

---

## Cloudflare D1：边缘 SQLite

### D1 的独特价值

D1 是 Cloudflare 在 2024 年推出的分布式 SQLite 数据库。它的核心理念是：SQLite 是世界上部署最广泛的数据库，简单、可靠、性能出色，但传统上只能运行在单台机器上，无法水平扩展。D1 把 SQLite 带到了边缘，让你在全球 300 多个城市拥有一个逻辑上统一的数据库。

实际工作方式是：每个区域有自己的 SQLite 副本，D1 会自动在区域间同步数据。你在写入时会选择写入到某个"主区域"，读取时则就近读取，性能极佳。

### 使用 D1

首先创建数据库：

```bash
# 创建数据库
wrangler d1 create my-database

# 初始化 schema
wrangler d1 execute my-database --local --file=./schema.sql
wrangler d1 execute my-database --remote --file=./schema.sql
```

定义数据库结构：

```sql
-- schema.sql
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS posts (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  title TEXT NOT NULL,
  content TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id)
);

CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_created_at ON posts(created_at);

-- 种子数据
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO posts (user_id, title, content) VALUES
  (1, 'First Post', 'Hello from D1!');
```

在 Workers 中使用：

```javascript
// 在 Workers 中通过 env.DB 访问
app.get("/api/stats", async (c) => {
  const { results: userCount } = await c.env.DB.prepare(
    "SELECT COUNT(*) as count FROM users"
  ).first();

  const { results: postCount } = await c.env.DB.prepare(
    "SELECT COUNT(*) as count FROM posts"
  ).first();

  return c.json({
    users: userCount?.count ?? 0,
    posts: postCount?.count ?? 0,
  });
});
```

### D1 限制与注意事项

D1 目前还有一些限制：单表不超过 50 万行（免费版），不支持外键约束强制（但可以写 CHECK），事务支持有限（只支持单 shard 的事务），写入有 30 秒超时。对于超大规模应用，D1 可能不是最终解决方案，但作为边缘应用的数据存储完全够用。

---

## Cloudflare R2：免费对象存储

### R2 的最大卖点：免费

R2 是 Cloudflare 的对象存储服务，和 Amazon S3 完全兼容（支持 S3 API），但最大的亮点是：**存储免费**。免费额度包括每月 100 万次 Class A 操作、1000 万次 Class B 操作和 10GB 存储。对于个人项目和小型应用来说，这个免费额度几乎用不完。

对比一下 AWS S3 的价格：S3 标准存储是 $0.023/GB/月。换句话说，Cloudflare 每月免费提供了相当于 $0.23 的 S3 存储。

### 上传和管理文件

R2 完全兼容 S3 API，所以任何 S3 客户端工具都可以使用：

```javascript
// 在 Workers 中使用 R2
import { S3Client, PutObjectCommand } from "@aws-sdk/client-s3";
import { Upload } from "@aws-sdk/lib-storage";

const r2 = new S3Client({
  region: "auto",
  endpoint: `https://${c.env.ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: c.env.R2_ACCESS_KEY_ID,
    secretAccessKey: c.env.R2_SECRET_ACCESS_KEY,
  },
});

app.post("/api/upload", async (c) => {
  const formData = await c.req.formData();
  const file = formData.get("file");

  if (!file) {
    return c.json({ error: "No file provided" }, 400);
  }

  const key = `${Date.now()}-${file.name}`;

  const upload = new Upload({
    client: r2,
    params: {
      Bucket: "my-bucket",
      Key: key,
      Body: file.stream(),
      ContentType: file.type,
    },
  });

  await upload.done();

  return c.json({
    url: `https://my-domain.com/files/${key}`,
    key,
  });
});
```

### R2 与 Workers 的配合

R2 特别适合存储静态资源：图片、视频、文档、备份文件。配合 Workers 的 URL 签名功能，还可以实现带过期时间的访问链接：

```javascript
app.get("/api/private/:key", async (c) => {
  const key = c.req.param("key");
  const expires = Math.floor(Date.now() / 1000) + 3600; // 1小时后过期

  // 生成签名 URL（需要 crypto API）
  const url = `https://my-domain.com/${key}?expires=${expires}&signature=...`;

  return c.json({ url });
});
```

---

## Cloudflare KV：边缘键值存储

### KV 的特点

KV 是 Cloudflare 的键值存储服务，读写速度极快（毫秒级），适合缓存、会话存储、配置存储等场景。它的特点是：写操作会最终一致地复制到全球所有节点，但读操作可以就近读取，非常快。

```javascript
// 设置值
await c.env.KV.put("user:123", JSON.stringify({ name: "Alice" }), {
  expirationTtl: 86400, // 24 小时后自动过期
});

// 读取值
const userData = await c.env.KV.get("user:123", "json");

// 删除值
await c.env.KV.delete("user:123");

// 列表查询（适合遍历）
const list = await c.env.KV.list({ prefix: "user:" });
```

### KV vs D1 vs R2 怎么选

| 存储类型 | 特点 | 适用场景 |
|---------|------|---------|
| KV | 极快读取，最终一致 | 缓存、会话、配置、临时数据 |
| D1 | SQL 查询，强一致写入 | 结构化数据、关系查询 |
| R2 | 大文件存储，S3 兼容 | 图片、视频、文档、备份 |

一个典型的组合是：用户配置信息放在 KV（快速读取），用户生成的文档放在 R2（存储大文件），应用元数据放在 D1（需要复杂查询）。

---

## Cloudflare AI Gateway

### 统一管理所有 AI API 调用

AI Gateway 是 Cloudflare 在 2024-2025 年推出的服务，它的作用是作为 AI API 的中间层。你不用直接调用 OpenAI 或 Anthropic 的 API，而是先调用 AI Gateway，它帮你处理缓存、限流、错误重试、日志记录、路由选择等工作。

```javascript
// Workers 中使用 AI Gateway
app.post("/api/chat", async (c) => {
  const { messages } = await c.req.json();

  const response = await c.env.AI.run("@cf/meta/llama-3.1-8b-instruct", {
    messages,
    max_tokens: 512,
  });

  return c.json({ response });
});
```

AI Gateway 的核心功能包括：请求缓存（相同的问题不重复计费）、多模型路由（自动选择最快或最便宜的模型）、速率限制（防止 API 被刷爆）、成本分析和日志记录。

---

## 定价与免费额度

### 个人项目的理想选择

Cloudflare 的免费额度非常慷慨，很多个人项目完全可以免费运行：

| 产品 | 免费额度 | 超出后价格 |
|------|---------|-----------|
| Workers | 每天 10 万次请求 | $5/百万请求 |
| KV | 每天 10 万次读取，1 万次写入 | 读取 $0.20/百万，写入 $5/百万 |
| D1 | 2500 万次读取，100 万次写入 | $0.20/百万读取，$5/百万写入 |
| R2 | 10GB 存储，100 万次 Class A，1000 万次 Class B | 存储 $0.015/GB，Class A $3.6/百万，Class B $0.36/百万 |
| Pages | 无限带宽，500 分钟构建 | $5/500 分钟 |
| CDN | 无限带宽 | 免费 |

对比其他平台，Cloudflare 的存储成本几乎是最低的。R2 存储完全免费（只收操作费），而 AWS S3 要 $0.023/GB/月。

### 成本优化建议

善用免费额度和合理设计可以节省大量成本：合理设置 KV 的 TTL 让数据自动过期，避免无用的频繁写入；善用 D1 的查询优化避免过度读取；使用 R2 的 Class B 操作（读操作比写操作便宜很多）；静态资源用 Pages 部署而不是 Workers 函数。

---

## 与 Vercel、Netlify 对比

| 维度 | Cloudflare | Vercel | Netlify |
|------|-----------|---------|---------|
| **核心定位** | 边缘计算 + 安全 + 存储 | Next.js 部署 | 静态部署 + Serverless |
| **Edge Network** | 300+ 城市，V8 Isolates | 70+ 城市 | 有限节点 |
| **存储** | R2（免费！）、D1、KV | Vercel KV、Blob | Netlify Blobs |
| **数据库** | D1（SQLite）、Hyperdrive | Postgres（Neon）、KV | Postgres（Neon） |
| **AI 支持** | Workers AI、AI Gateway | Vercel AI SDK | 有限 |
| **免费带宽** | 无限（CDN） | 100GB/月 | 100GB/月 |
| **价格透明度** | 清晰，免费额度大 | 中等 | 中等 |

选择建议：**如果你的项目是 Next.js 全栈应用**且看重与框架的原生集成，选 Vercel。**如果你需要全球分布的边缘计算 + 免费存储 + AI 能力**，选 Cloudflare。**如果你主要做静态网站**且喜欢简单配置，Netlify 是不错的选择。

---

> [!SUCCESS]
> Cloudflare 已经从一个 CDN 提供商进化成了一个完整的边缘计算平台。Workers 提供了极低延迟的计算能力，D1 和 KV 提供了边缘数据结构存储，R2 提供了几乎免费的存储空间，AI Gateway 让 AI 应用开发更加高效。对于想在边缘构建高性能应用的开发者来说，Cloudflare 提供了一套完整、成本可控的解决方案。
