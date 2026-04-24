---
date: 2026-04-24
tags:
  - Vercel
  - 部署
  - Serverless
  - Next.js
  - Edge Functions
description: Vercel 平台完整指南，涵盖 Next.js 部署、Edge Network、Serverless Functions、环境变量、预览部署、Analytics、存储服务，以及与 Cloudflare/Netlify 的对比。
---

# Vercel：前端部署的首选平台

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Vercel 平台架构、Edge Network、Serverless Functions、Next.js 部署配置、环境变量、Analytics，以及与 Cloudflare Pages/Netlify 的对比分析。

---

## Vercel 是什么

### 平台定位与核心理念

Vercel 由 Guillermo Rauch 于 2015 年创立，这位创始人同时也是广受欢迎的 Node.js 框架 Next.js 的创造者。Vercel 最初的名字叫 ZEIT，2020 年更名为 Vercel，并在同年获得了大量关注——因为 Next.js 的爆火让 Vercel 成为了前端部署领域最炙手可热的平台。

Vercel 的核心理念可以用三个词概括：**极简、自动、全球**。

**极简**体现在零配置部署上。你不需要写任何 CI/CD 配置文件，不需要管理服务器，不需要配置负载均衡，只需要把代码 push 到 GitHub，Vercel 自动检测项目类型、运行构建、部署上线。第一次使用时这个体验确实会让人眼前一亮——原来部署可以这么简单。

**自动**体现在预览部署上。每次创建 Pull Request，Vercel 自动生成一个独立的预览 URL，团队成员可以在合并前看到实际效果。这彻底改变了 code review 的流程——不再只是看代码改动，而是可以直接在真实环境中测试。

**全球**体现在 Edge Network 上。Vercel 在全球 70 多个城市部署了边缘节点，用户请求会被路由到最近的数据中心，响应延迟极低。

### Vercel 能部署什么

Vercel 对前端框架的支持非常全面，最核心的当然是 Next.js，但也不限于此：

```
React 框架：Next.js（原生支持，深度优化）、Create React App、Vite、Remix
Vue 框架：Nuxt、Vue
Svelte 框架：Svelte、SvelteKit
静态站点：Next.js（SSG）、Astro、Gatsby、Hugo、Eleventy
全栈应用：Next.js API Routes、Nest.js、RedwoodJS
其他：Angular、Express 等任何可以构建为静态资源或 Serverless 的项目
```

对于 Next.js 项目来说，Vercel 是绝对的首选——因为 Next.js 就是 Vercel 团队开发的，所以在框架更新和新特性支持上，Vercel 总是第一个吃螃蟹的。

---

## 技术架构：请求是如何被处理的

### 整体架构

当你访问一个托管在 Vercel 上的网站时，请求会经过一个精心设计的路由网络：

```
用户发起请求（https://myapp.vercel.app）
        │
        ▼
    DNS 解析
        │
        ▼
    全球 CDN 边缘节点（最近的 PoP）
        │
        ├─ 静态资源（如 JS/CSS/图片）
        │   → 直接从 CDN 缓存返回（极快）
        │
        └─ 动态请求
            │
            ▼
        Regional Edge（区域计算节点）
            │
            ├─ ISR 页面 → 返回缓存的 HTML
            │
            ├─ SSR → 执行 Serverless Function
            │
            └─ API Routes → 执行 API 函数
                    │
                    ▼
                可能的后端服务（数据库、缓存）
                    │
                    ▼
                响应返回（经过 CDN 缓存优化）
```

这个架构的精妙之处在于分层处理：不需要计算的静态资源直接从边缘 CDN 返回，需要计算的才路由到区域计算节点。CDN 缓存命中率高的话，大部分请求根本不需要到达 Serverless Functions 层，成本和延迟都大幅降低。

### Serverless Functions vs Edge Functions

Vercel 提供两种计算模式，理解它们的区别很重要：

**Serverless Functions** 运行在传统的 Node.js 环境中，冷启动大约 100-500 毫秒，执行时间限制在免费版 10 秒、付费版 60 秒（可配置到 300 秒）。它们适合需要复杂业务逻辑、数据库操作、文件处理等 CPU 和 IO 密集型的任务。

**Edge Functions** 运行在 V8 隔离区中，冷启动只需 1-5 毫秒，执行时间限制 50 毫秒。它们适合轻量级的处理逻辑：路由重写、访问控制、A/B 测试、地理位置判断等。Edge Functions 牺牲了执行时间换取极低的响应延迟。

| 对比维度 | Serverless Functions | Edge Functions |
|---------|---------------------|---------------|
| 冷启动时间 | 100-500ms | < 5ms |
| 执行时间限制 | 10-300s | 50ms |
| 运行环境 | Node.js | V8 Isolates |
| 支持的 API | 完整的 Node.js API | Fetch API（受限） |
| 支持的 npm 包 | 所有 npm 包 | 有限（轻量的包） |
| 适用场景 | 复杂业务逻辑、数据库操作 | 路由、中间件、A/B 测试 |
| 成本 | 按执行时间计费 | 按请求数计费（更便宜） |

在实际项目中，一个常见的设计模式是：Edge Functions 处理路由和访问控制（放在前面，响应快），Serverless Functions 处理实际的数据库查询和业务逻辑（放在后面，能力强）。

---

## 部署你的第一个项目

### 三种部署方式

Vercel 提供了三种部署方式，从简单到灵活排序：

**方式一：通过 GitHub 直接导入（最推荐）**

这是 Vercel 的核心使用方式。访问 vercel.com/new，选择你的 GitHub 仓库，Vercel 自动检测项目类型并配置构建命令。整个过程不到一分钟，之后每次 push 到指定分支都会自动触发新的部署。

**方式二：Vercel CLI**

适合喜欢在终端里完成一切的开发者：

```bash
# 全局安装 CLI
npm install -g vercel

# 登录
vercel login

# 在项目目录中初始化
cd my-project
vercel

# 部署到生产环境
vercel --prod
```

**方式三：Dashboard 上传**

直接拖拽文件夹到网页上，适合临时演示或不想连接 GitHub 的场景。

### Next.js 项目配置

Vercel 对 Next.js 项目提供了开箱即用的支持，但一些高级配置需要手动设置：

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // 图片优化配置
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "images.unsplash.com",
      },
      {
        protocol: "https",
        hostname: "*.amazonaws.com",
      },
    ],
    formats: ["image/avif", "image/webp"],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
  },

  // 编译优化
  compiler: {
    removeConsole: process.env.NODE_ENV === "production",
  },

  // 重定向规则
  async redirects() {
    return [
      {
        source: "/old-blog/:slug",
        destination: "/blog/:slug",
        permanent: true, // 301 重定向
      },
    ];
  },

  // Header 配置
  async headers() {
    return [
      {
        source: "/(.*)",
        headers: [
          {
            key: "X-Frame-Options",
            value: "DENY",
          },
          {
            key: "X-Content-Type-Options",
            value: "nosniff",
          },
        ],
      },
    ];
  },

  // 实验性特性
  experimental: {
    optimizePackageImports: ["lucide-react", "recharts", "date-fns"],
  },
};

module.exports = nextConfig;
```

---

## Serverless Functions 实战

### API Routes 基础

Next.js 的 API Routes 功能允许你在 Next.js 应用中直接创建 API 端点，部署时会自动转换为 Serverless Functions：

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";

// GET 请求处理
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get("page") || "1");
  const limit = parseInt(searchParams.get("limit") || "10");
  const search = searchParams.get("search") || "";

  const where = search
    ? {
        OR: [
          { name: { contains: search, mode: "insensitive" } },
          { email: { contains: search, mode: "insensitive" } },
        ],
      }
    : {};

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: "desc" },
      select: {
        id: true,
        name: true,
        email: true,
        image: true,
        createdAt: true,
      },
    }),
    prisma.user.count({ where }),
  ]);

  return NextResponse.json({
    users,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  });
}

// POST 请求处理
export async function POST(request: NextRequest) {
  const body = await request.json();
  const { name, email } = body;

  if (!name || !email) {
    return NextResponse.json(
      { error: "Name and email are required" },
      { status: 400 }
    );
  }

  try {
    const user = await prisma.user.create({
      data: { name, email },
    });

    return NextResponse.json(user, { status: 201 });
  } catch (error) {
    if ((error as { code?: string }).code === "P2002") {
      return NextResponse.json(
        { error: "User with this email already exists" },
        { status: 409 }
      );
    }
    throw error;
  }
}
```

### 函数配置与性能调优

可以通过 `vercel.json` 为不同的函数设置不同的资源配置：

```json
{
  "functions": {
    "app/api/basic/**/*.ts": {
      "maxDuration": 10,
      "memory": 1024,
      "regions": ["iad1"]
    },
    "app/api/ai/**/*.ts": {
      "maxDuration": 60,
      "memory": 3008,
      "regions": ["iad1"]
    },
    "app/api/export/**/*.ts": {
      "maxDuration": 300,
      "memory": 2048
    }
  }
}
```

---

## Edge Functions 与中间件

### 中间件基础

Next.js 的中间件（Middleware）运行在 Edge Functions 环境中，可以在请求到达页面之前执行逻辑：

```typescript
// middleware.ts
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // 基于 cookie 实现 A/B 测试分组
  let group = request.cookies.get("ab-group")?.value;
  if (!group) {
    group = Math.random() > 0.5 ? "A" : "B";
    const response = NextResponse.next();
    response.cookies.set("ab-group", group, {
      maxAge: 60 * 60 * 24 * 30,
      path: "/",
    });
    response.headers.set("X-AB-Group", group);
    return response;
  }

  // 为 B 组用户重写到 beta 页面
  if (pathname === "/" && group === "B") {
    return NextResponse.rewrite(new URL("/beta", request.url));
  }

  // 检查受保护路径的登录状态
  if (pathname.startsWith("/dashboard")) {
    const token = request.cookies.get("auth-token")?.value;
    if (!token) {
      const loginUrl = new URL("/login", request.url);
      loginUrl.searchParams.set("redirect", pathname);
      return NextResponse.redirect(loginUrl);
    }
  }

  // 添加安全头
  const response = NextResponse.next();
  response.headers.set("X-Custom-Header", "value");
  return response;
}

export const config = {
  matcher: ["/((?!api|_next/static|_next/image|favicon.ico).*)"],
};
```

### 中间件实战用例

中间件的典型使用场景非常丰富：**地理定位重定向**，根据用户所在地区跳转到对应语言或版本的页面；**Bot 检测**，识别爬虫并返回不同内容或直接拒绝访问；**速率限制**，在边缘层拦截过多请求；**JWT 验证**，在请求到达 Serverless Function 前验证身份令牌。

---

## 环境变量与敏感信息管理

### 环境变量基础

Vercel 的环境变量通过 Dashboard 或 CLI 配置，区分 Development（开发预览）、Preview（每个 PR 的预览环境）和 Production（生产环境）三个层级：

```bash
# 通过 CLI 设置
vercel env add DATABASE_URL
vercel env add OPENAI_API_KEY production

# 查看所有环境变量
vercel env ls

# 在本地使用生产环境变量进行调试
vercel env pull
```

### 类型安全的环境变量

使用 Zod 在构建时验证环境变量，避免运行时才发现配置错误：

```typescript
// lib/env.ts
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url().optional(),
  OPENAI_API_KEY: z.string().startsWith("sk-"),
  NEXT_PUBLIC_APP_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);

type Env = z.infer<typeof envSchema>;
declare global {
  namespace NodeJS {
    interface ProcessEnv extends Env {}
  }
}
```

---

## 预览部署与团队协作

### 预览部署工作流

这是 Vercel 最让人印象深刻的特性之一。创建 Pull Request 后，Vercel 自动构建并部署到独立的预览环境，生成一个形如 `https://feature-auth-k8s2xyz.vercel.app` 的 URL。这个 URL 可以分享给任何人查看，不需要在本地运行项目。

合并到 main 分支后，Vercel 自动部署到生产环境。整个流程不需要任何手动操作。

### 在 Preview 环境配置密码保护

预览环境通常需要保护，避免搜索引擎索引和未授权访问。可以通过 Vercel CLI 为预览环境设置密码：

```bash
vercel deploy --token=xxx --password=mypassword
```

也可以通过中间件实现密码保护：

```typescript
// middleware.ts
export function middleware(request: NextRequest) {
  const isPreview =
    request.url.includes(".vercel.app") ||
    process.env.NODE_ENV !== "production";
  const hasAuth = request.cookies.has("preview-auth");

  if (isPreview && !hasAuth) {
    const password = request.nextUrl.searchParams.get("password");
    if (password !== process.env.PREVIEW_PASSWORD) {
      return new NextResponse("<h1>Password Required</h1>", {
        headers: { "content-type": "text/html" },
      });
    }

    const response = NextResponse.next();
    response.cookies.set("preview-auth", "1", { maxAge: 3600 });
    return response;
  }

  return NextResponse.next();
}
```

---

## Vercel 内置存储服务

### Vercel KV（Redis 兼容）

Vercel KV 是基于 Upstash 的 Serverless Redis 服务，提供极快的键值读写能力：

```typescript
import { Redis } from "@upstash/redis";

export const redis = new Redis({
  url: process.env.KV_REST_API_URL!,
  token: process.env.KV_REST_API_TOKEN!,
});

// 在 API Route 中使用
export async function GET(request: NextRequest) {
  const cached = await redis.get("data");
  if (cached) {
    return NextResponse.json({ source: "cache", data: cached });
  }

  const data = await fetchData();
  await redis.set("data", data, { ex: 3600 }); // 1小时过期

  return NextResponse.json({ source: "origin", data });
}
```

### Vercel Blob（对象存储）

Vercel Blob 用于存储文件，提供流式上传、签名 URL 和自动 CDN 优化：

```typescript
import { put } from "@vercel/blob";

export async function POST(request: NextRequest) {
  const formData = await request.formData();
  const file = formData.get("file") as File;

  if (!file) {
    return NextResponse.json({ error: "No file" }, { status: 400 });
  }

  const blob = await put(file.name, file, {
    access: "public",
  });

  return NextResponse.json({ url: blob.url });
}
```

### Vercel Postgres（Serverless PostgreSQL）

Vercel Postgres 基于 Neon 技术，提供 Serverless 的 PostgreSQL 体验——不需要管理连接池，Vercel 帮你处理：

```typescript
import { sql } from "@vercel/postgres";

export async function GET() {
  const { rows } = await sql`SELECT * FROM users LIMIT 10`;
  return NextResponse.json({ users: rows });
}
```

---

## 性能优化与缓存策略

### ISR：增量静态再生成

ISR（Incremental Static Regeneration）是 Next.js 提供的介于 SSR 和 SSG 之间的渲染策略。页面在构建时预渲染为静态页面，但在过期后可以自动重新生成，而不需要重新部署：

```typescript
// app/blog/[slug]/page.tsx
// 每小时重新验证一次
export const revalidate = 3600;

export default async function BlogPost({
  params,
}: {
  params: { slug: string };
}) {
  // 首次访问时执行，revalidate 秒后下次访问会重新执行
  const post = await fetchPost(params.slug);

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}
```

这样一来，大部分访问走的是 CDN 缓存（极快），每隔一小时才有一个请求打到 Serverless Function 层，极大降低了成本。

### 缓存头配置

可以在 API Route 中手动设置缓存策略：

```typescript
export async function GET() {
  const data = await getData();

  return NextResponse.json(data, {
    headers: {
      // CDN 缓存 1 小时，1 小时内即使数据更新也返回旧数据
      // stale 时允许后台重新验证
      "Cache-Control": "public, s-maxage=3600, stale-while-revalidate=86400",
    },
  });
}
```

---

## 定价体系

### 套餐对比

| 功能 | Hobby（免费） | Pro | Enterprise |
|------|--------------|-----|-----------|
| 带宽 | 100GB/月 | 无限 | 无限 |
| Serverless Functions | 100 小时/月 | 1000 小时/月 | 自定义 |
| Edge Functions | 500K 请求/月 | 无限 | 无限 |
| 预览部署 | ✓ | ✓ | ✓ |
| 自定义域名 | 1 个 | 无限 | 无限 |
| 密码保护预览 | ✗ | ✓ | ✓ |
| Analytics | 基础版 | 高级版 | 自定义 |
| 技术支持 | 社区 | 优先邮件 | 专属客户成功 |
| 价格 | $0/月 | $20/月/人 | 定制 |

### 成本优化建议

对于个人项目，免费版 Hobby 套餐基本够用：100GB 月带宽、100 小时 Serverless Functions、500K 次 Edge Function 请求，配合 ISR 缓存策略，大部分个人网站和小工具完全不用付费。

Pro 套餐适合团队协作场景——无限域名、密码保护预览、高级 Analytics 让 code review 和协作更高效。$20/月/人的定价在同类产品中属于中等偏上，但考虑到零运维成本和开发效率提升，性价比还是不错的。

---

## 与 Cloudflare Pages、Netlify 的对比

| 维度 | Vercel | Cloudflare Pages | Netlify |
|------|--------|-----------------|--------|
| **最佳场景** | Next.js 全栈应用 | 边缘计算 + 免费存储 | 静态站点 + Jamstack |
| **Edge Network** | 70+ 节点 | 300+ 节点 | 有限 |
| **Serverless** | Vercel Functions | Workers | Netlify Functions |
| **ISR 支持** | ✓（独有） | ✗ | ✗ |
| **图片优化** | 内置 | 需插件 | 需插件 |
| **数据库** | Vercel Postgres | D1（更丰富） | 第三方集成 |
| **对象存储** | Vercel Blob | R2（免费存储！） | Netlify Blobs |
| **免费带宽** | 100GB/月 | 无限（CDN） | 100GB/月 |
| **学习曲线** | 低 | 中 | 低 |
| **Git 集成** | GitHub/GitLab/Bitbucket | GitHub/GitLab | GitHub/GitLab |

**选择建议**：如果你的项目使用 Next.js 并且需要全栈能力（SSR、API Routes、ISR），选 Vercel，它和 Next.js 的集成度无可替代。如果你想用最小的成本跑一个边缘应用，Cloudflare Pages + Workers + R2 的免费额度组合非常诱人。如果你的项目是纯静态站点并且追求简单配置，Netlify 是个好选择。

---

> [!SUCCESS]
> Vercel 重新定义了前端部署的体验——不需要理解复杂的 CI/CD 配置，不需要管理服务器，不需要配置 CDN，只需要 push 代码，一切自动发生。对于追求效率的 vibecoding 开发者来说，Vercel 把部署的复杂性降到最低，让你真正把注意力放在代码本身。
