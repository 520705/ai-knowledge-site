# Next.js 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Next.js 15 最新特性、App Router 完整指南、性能优化策略及生产部署实践。

---

## 目录

1. [[#Next.js 概述与版本演进]]
2. [[#App Router vs Pages Router 核心对比]]
3. [[#React Server Components 深入理解]]
4. [[#渲染模式详解]]
5. [[#文件系统路由系统]]
6. [[#Server Actions 与数据变更]]
7. [[#API Routes 与 Route Handlers]]
8. [[#性能优化策略]]
9. [[#Next.js 15 新特性]]
10. [[#部署与托管]]
11. [[#实战场景与选型建议]]

---

## Next.js 概述与版本演进

### Next.js 在现代前端生态中的定位

Next.js 是 Vercel 开发的 React 元框架，自 2016 年发布以来，已成为构建生产级 Web 应用的首选工具之一。在 2026 年的前端生态中，Next.js 凭借其完善的 SSR/SSG 支持、优秀的开发体验和强大的生态体系，在以下场景中占据主导地位：

- **全栈 React 应用**：需要服务端渲染和 API 能力的企业级应用
- **内容驱动型网站**：博客、文档站点、电商产品页
- **SSR 优先项目**：SEO 敏感、需要首屏性能优化的应用
- **vibecoding 开发**：AI 辅助编程的最佳载体，配合 Vercel AI SDK

### 版本演进时间线

| 版本 | 发布时间 | 核心特性 |
|------|---------|---------|
| Next.js 13 | 2022年10月 | App Router Beta、Server Components 原生支持 |
| Next.js 14 | 2023年10月 | Turbopack Beta、Server Actions 稳定、Partial Prerendering 预览 |
| Next.js 15 | 2024年10月 | Turbopack 稳定、React 19 支持、缓存语义变更、next/font 增强 |

> [!IMPORTANT]
> Next.js 15 要求 React 19，推荐使用 TypeScript 5.5+。升级前请务必阅读官方迁移指南。

---

## App Router vs Pages Router 核心对比

App Router（`app/` 目录）是 Next.js 13 引入的新一代路由系统，与传统的 Pages Router（`pages/` 目录）存在根本性差异。

### 架构哲学对比

| 维度 | App Router | Pages Router |
|------|-----------|-------------|
| **React 版本** | React 18+ Server Components | React 17 及以下 |
| **默认行为** | 服务端渲染 | 客户端渲染 |
| **组件模型** | 服务端组件 + 客户端组件分离 | 统一客户端组件 |
| **布局系统** | 嵌套布局自动继承 | 手动布局封装 |
| **路由组织** | 文件夹即路由 | 文件即路由 |
| **数据获取** | async/await 组件 | getServerSideProps/getStaticProps |
| **缓存策略** | 请求 deduplication | Fetch API 缓存选项 |
| **学习曲线** | 陡峭（概念多） | 平缓（传统模式） |

### 目录结构对比

**Pages Router 结构：**

```plaintext
pages/
├── index.tsx              # 首页 /
├── about.tsx              # 关于页 /about
├── blog/
│   ├── index.tsx          # 博客列表 /blog
│   └── [slug].tsx         # 博客详情 /blog/:slug
└── api/
    └── users.ts           # API 路由 /api/users
```

**App Router 结构：**

```plaintext
app/
├── layout.tsx              # 根布局（所有页面共享）
├── page.tsx                # 首页 /
├── about/
│   └── page.tsx           # 关于页 /about
├── blog/
│   ├── page.tsx           # 博客列表 /blog
│   └── [slug]/
│       └── page.tsx       # 博客详情 /blog/:slug
└── api/
    └── users/
        └── route.ts        # Route Handler /api/users
```

### 选择建议

> [!TIP]
> 新项目强烈推荐使用 App Router，除非需要兼容现有 Pages Router 代码库。App Router 在性能、开发体验和功能丰富度上都有显著优势。

---

## React Server Components 深入理解

### Server Components vs Client Components

React Server Components（RSC）是 React 18 引入的核心特性，Next.js App Router 将其作为默认行为。

#### Server Components 特性

- **服务端执行**：组件代码仅在服务端运行，产物不进入客户端 bundle
- **直接数据访问**：可直接调用数据库、文件系统，无需 API 层
- **零客户端 JavaScript**：不产生 hydration，无交互功能
- **支持 async/await**：可直接使用 `async/await` 获取数据

```tsx
// app/blog/page.tsx - Server Component（默认）
import { db } from '@/lib/db';

// 直接访问数据库，无需 API 调用
async function BlogList() {
  const posts = await db.post.findMany({
    where: { published: true },
    orderBy: { createdAt: 'desc' },
    take: 10
  });

  return (
    <div>
      <h1>最新文章</h1>
      {posts.map(post => (
        <article key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.excerpt}</p>
        </article>
      ))}
    </div>
  );
}

export default BlogList;
```

#### Client Components 特性

- **客户端执行**：代码发送到浏览器执行
- **交互能力**：可使用 useState、useEffect、event handlers
- **需要 'use client' 声明**：明确标注客户端边界

```tsx
// components/LikeButton.tsx - Client Component
'use client';

import { useState } from 'react';

export function LikeButton({ initialLikes }: { initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes);
  const [isLoading, setIsLoading] = useState(false);

  async function handleLike() {
    setIsLoading(true);
    // 调用 Server Action 更新数据
    await likePost();
    setLikes(prev => prev + 1);
    setIsLoading(false);
  }

  return (
    <button 
      onClick={handleLike} 
      disabled={isLoading}
    >
      {isLoading ? '加载中...' : `👍 ${likes}`}
    </button>
  );
}
```

### 组件树架构

```
Root Layout (Server)
├── Header (Server)
│   └── ThemeToggle (Client)  ← 需要交互
├── BlogList (Server)          ← 直接访问 DB
│   └── PostCard (Server)
│       └── LikeButton (Client) ← 需要交互
└── Footer (Server)
```

> [!IMPORTANT]
> 组件树中，Server Components 可导入 Client Components，但 Client Components 不能再导入 Server Components（可通过 props 传递 Server Component 的渲染结果）。

---

## 渲染模式详解

### 渲染模式对比表

| 模式 | 说明 | 适用场景 | 重新验证 |
|------|------|---------|---------|
| **SSR** | 每次请求实时渲染 | 实时数据、高度个性化 | 每请求 |
| **SSG** | 构建时生成静态 HTML | 博客、文档、Marketing 页 | revalidate |
| **ISR** | 定时重新生成静态页 | 内容频繁更新但不需要实时 | 按时间间隔 |
| **CSR** | 客户端渲染 | 完全个性化内容、仪表板 | 每请求 |
| **PPR** | 部分动态渲染 | 动态 + 静态混合 | 细粒度控制 |

### SSG 静态生成

```tsx
// app/blog/[slug]/page.tsx
export async function generateStaticParams() {
  // 构建时生成所有文章页
  const posts = await db.post.findMany();
  return posts.map(post => ({ slug: post.slug }));
}

export default async function BlogPost({ 
  params 
}: { 
  params: Promise<{ slug: string }> 
}) {
  const { slug } = await params;
  const post = await db.post.findUnique({ 
    where: { slug } 
  });

  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

### ISR 增量静态再生成

```tsx
// app/products/[id]/page.tsx
// 每 3600 秒（1小时）重新验证一次
export const revalidate = 3600;

export default async function ProductPage({ 
  params 
}: { 
  params: Promise<{ id: string }> 
}) {
  const { id } = await params;
  // 首次访问时生成，后续按 revalidate 周期更新
  const product = await getProduct(id);

  return (
    <div>
      <h1>{product.name}</h1>
      <p>库存：{product.stock}</p>
    </div>
  );
}
```

### PPR 部分动态渲染（Next.js 14+）

```tsx
import { Suspense } from 'react';

// 静态内容 - 立即渲染
const StaticHeader = await import('./StaticHeader');

// 动态内容 - 渐进式加载
const DynamicCart = dynamic(() => import('./Cart'), {
  loading: () => <CartSkeleton />
});

export default function Layout() {
  return (
    <div>
      <StaticHeader />
      <Suspense fallback={<CartSkeleton />}>
        <DynamicCart />
      </Suspense>
    </div>
  );
}
```

---

## 文件系统路由系统

### 路由约定

| 文件/文件夹 | 路由路径 | 说明 |
|------------|---------|------|
| `app/page.tsx` | `/` | 首页 |
| `app/about/page.tsx` | `/about` | 关于页 |
| `app/blog/page.tsx` | `/blog` | 博客列表 |
| `app/blog/[slug]/page.tsx` | `/blog/:slug` | 动态博客详情 |
| `app/[locale]/page.tsx` | `/en`, `/zh` | 多语言路由 |
| `app/(marketing)/page.tsx` | `/` | 路由组（不影响 URL） |
| `app/blog/@modal/(.)[slug]/page.tsx` | 并行路由 | 模态框叠加 |

### 布局系统

**根布局（必须存在）：**

```tsx
// app/layout.tsx
import './globals.css';

export const metadata: Metadata = {
  title: '我的博客',
  description: '分享技术与生活',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh-CN">
      <body>
        <Header />
        <main>{children}</main>
        <Footer />
      </body>
    </html>
  );
}
```

**嵌套布局：**

```tsx
// app/dashboard/layout.tsx
// 所有 /dashboard/* 路由共享此布局
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="dashboard">
      <Sidebar />
      <div className="content">{children}</div>
    </div>
  );
}
```

### 模板（Templates）

模板与布局类似，但每个路由都会创建新实例（区别于布局的持久化）：

```tsx
// app/blog/[slug]/template.tsx
// 每次路由变化时重新挂载，适合动画场景
'use client';

import { useEffect } from 'react';
import { useRouter } from 'next/navigation';

export default function BlogTemplate({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  const router = useRouter();
  
  useEffect(() => {
    // 页面切换时触发动画
    const animate = () => {
      document.querySelector('.article')?.classList.add('fade-in');
    };
    animate();
  }, [router.asPath]);

  return <article>{children}</article>;
}
```

---

## Server Actions 与数据变更

### Server Actions 基础

Server Actions 是 Next.js 14 稳定的核心特性，允许在服务端直接处理表单提交和数据变更：

```tsx
// app/actions.ts
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { db } from '@/lib/db';

export async function createPost(formData: FormData) {
  const title = formData.get('title') as string;
  const content = formData.get('content') as string;

  await db.post.create({
    data: { title, content }
  });

  // 清除缓存并跳转
  revalidatePath('/blog');
  redirect('/blog');
}

export async function deletePost(id: string) {
  await db.post.delete({ where: { id } });
  revalidatePath('/blog');
}
```

### 表单使用 Server Action

```tsx
// app/blog/new/page.tsx
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <div>
        <label htmlFor="title">标题</label>
        <input 
          type="text" 
          id="title" 
          name="title" 
          required 
        />
      </div>
      <div>
        <label htmlFor="content">内容</label>
        <textarea 
          id="content" 
          name="content" 
          rows={10}
          required 
        />
      </div>
      <button type="submit">发布</button>
    </form>
  );
}
```

### 带验证的 Server Action

```tsx
// app/actions.ts
'use server';

import { z } from 'zod';

const PostSchema = z.object({
  title: z.string().min(1).max(200),
  content: z.string().min(10),
});

export type ActionResult = {
  success: boolean;
  errors?: z.ZodError['errors'];
  message?: string;
};

export async function createPost(
  prevState: ActionResult,
  formData: FormData
): Promise<ActionResult> {
  const validated = PostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
  });

  if (!validated.success) {
    return {
      success: false,
      errors: validated.error.errors,
    };
  }

  await db.post.create({ data: validated.data });
  revalidatePath('/blog');

  return { success: true, message: '发布成功' };
}
```

---

## API Routes 与 Route Handlers

### Route Handlers（App Router）

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '10');

  const users = await db.user.findMany({
    skip: (page - 1) * limit,
    take: limit,
  });

  const total = await db.user.count();

  return NextResponse.json({
    data: users,
    pagination: { page, limit, total }
  });
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const user = await db.user.create({ data: body });
  
  return NextResponseResponse.json(user, { status: 201 });
}
```

### 传统 API Routes（Pages Router）

```typescript
// pages/api/users.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  switch (req.method) {
    case 'GET':
      const users = await db.user.findMany();
      res.status(200).json(users);
      break;
    case 'POST':
      const newUser = await db.user.create({ data: req.body });
      res.status(201).json(newUser);
      break;
    default:
      res.setHeader('Allow', ['GET', 'POST']);
      res.status(405).end(`Method ${req.method} Not Allowed`);
  }
}
```

> [!TIP]
> App Router 的 Route Handlers 比 Pages API Routes 更轻量、更灵活，支持动态方法定义。

---

## 性能优化策略

### 图片优化

```tsx
import Image from 'next/image';

export function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <Image
      src={src}
      alt={alt}
      width={400}
      height={300}
      sizes="(max-width: 768px) 100vw, 400px"
      placeholder="blur"           // 模糊占位
      blurDataURL="/placeholder.png"
      priority                  // 优先级加载
    />
  );
}
```

### 字体优化

```tsx
// app/layout.tsx
import { Inter } from 'next/font/google';

const inter = Inter({ 
  subsets: ['latin'],
  display: 'swap',        // 字体交换防止 FOIT
  variable: '--font-inter',
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html className={inter.variable}>
      <body>{children}</body>
    </html>
  );
}
```

### 组件懒加载

```tsx
import dynamic from 'next/dynamic';

// 动态导入客户端组件
const HeavyChart = dynamic(
  () => import('@/components/HeavyChart'),
  { 
    loading: () => <ChartSkeleton />,
    ssr: false  // 禁用 SSR（某些库如 D3 需要）
  }
);

export default function Dashboard() {
  return (
    <div>
      <SummaryCards />
      <HeavyChart />  {/* 按需加载 */}
    </div>
  );
}
```

### 缓存策略

```tsx
// app/blog/page.tsx
export const dynamic = 'force-dynamic';        // 强制 SSR
export const dynamic = 'force-static';          // 强制 SSG
export const revalidate = 60;                    // 每 60 秒重新验证
export const dynamic = 'error';                 // 运行时动态报错

// Fetch 级别缓存控制
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 3600 },  // ISR 模式
  cache: 'no-store',          // 禁用缓存
  cache: 'force-cache',       // 强制缓存
});
```

---

## Next.js 15 新特性

### Turbopack 稳定化

Turbopack 是 Vercel 开发的 Rust 编写的打包工具，Next.js 15 将其标记为稳定：

```bash
# 启用 Turbopack
next dev --turbopack

# 或在 next.config.ts 中配置
export default {
  experimental: {
    turbo: {},
  },
};
```

| 指标 | Turbopack vs Webpack |
|------|---------------------|
| 冷启动速度 | **10x 提升** |
| 热更新速度 | **7x 提升** |
| 内存占用 | **50% 降低** |

### React 19 支持

Next.js 15 支持 React 19，带来以下改进：

- **改进的 `<Form>` 组件**：原生表单行为增强
- **新的 ref 清理 API**：`useCallback` 的 ref 版本
- **资源预加载 API**：`<Link>` 自动预加载资源

### 缓存语义变更

> [!IMPORTANT]
> Next.js 15 对缓存行为进行了重大调整：

- **POST 请求默认不缓存**（之前会缓存）
- **`fetch` 默认 `auto` 缓存**（之前默认 `force-cache`）
- **`generateStaticParams` 行为变更**

```tsx
// 新的默认行为
fetch('https://api.example.com', { 
  cache: 'auto'  // 默认，不缓存
});

// 需要缓存时显式声明
fetch('https://api.example.com', { 
  cache: 'force-cache'  // 静态数据
});
```

### Partial Prerendering（PPR）

PPR 允许页面同时包含静态和动态内容：

```tsx
// app/product/[id]/page.tsx
export default async function ProductPage({ 
  params 
}: { 
  params: Promise<{ id: string }> 
}) {
  const { id } = await params;
  
  // 静态内容 - 立即渲染
  const product = await getProductStatic(id);
  
  // 动态内容 - 渐进式加载
  const { price, stock } = await getProductDynamic(id);

  return (
    <div>
      {/* 静态渲染 */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      {/* Suspense 包裹的动态内容 */}
      <Suspense fallback={<PriceSkeleton />}>
        <Price id={id} />
      </Suspense>
    </div>
  );
}
```

---

## 部署与托管

### Vercel 部署（推荐）

Vercel 是 Next.js 的亲儿子，提供零配置部署体验：

```bash
# 安装 Vercel CLI
npm i -g vercel

# 部署
vercel

# 生产环境部署
vercel --prod
```

**vercel.json 配置示例：**

```json
{
  "framework": "nextjs",
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" }
      ]
    }
  ]
}
```

### 独立部署

Next.js 支持构建为独立产物，可在任意 Node.js 环境运行：

```bash
# 构建
npm run build

# 运行
node .next/standalone/server.js
```

**next.config.ts 配置：**

```typescript
const nextConfig = {
  output: 'standalone',
};
```

### 自托管选项

| 方案 | 特点 | 适用场景 |
|------|------|---------|
| **Vercel** | 零配置，全球 CDN | 快速上线、Serverless |
| **AWS Amplify** | AWS 生态集成 | 已有 AWS 基础设施 |
| **Docker** | 完全可控 | 企业内网、数据主权 |
| **Kubernetes** | 弹性伸缩 | 大规模、高可用需求 |
| **Railway/Render** | 简单部署 | 小型项目、预算有限 |

---

## 实战场景与选型建议

### AI 应用开发实战

结合 Vercel AI SDK 构建 AI 对话应用：

```tsx
// app/chat/page.tsx
'use client';

import { useChat } from 'ai/react';

export default function Chat() {
  const { messages, input, handleInputChange, handleSubmit } = useChat({
    api: '/api/chat',
  });

  return (
    <div>
      {messages.map(msg => (
        <div key={msg.id} className={msg.role}>
          {msg.content}
        </div>
      ))}
      <form onSubmit={handleSubmit}>
        <input
          value={input}
          onChange={handleInputChange}
          placeholder="输入消息..."
        />
        <button type="submit">发送</button>
      </form>
    </div>
  );
}
```

```typescript
// app/api/chat/route.ts
import { openai } from '@ai-sdk/openai';
import { streamText } from 'ai';

export async function POST(req: Request) {
  const { messages } = await req.json();

  const result = await streamText({
    model: openai('gpt-4o'),
    system: '你是一个专业的中文助手。',
    messages,
  });

  return result.toDataStreamResponse();
}
```

### 选型决策树

> [!NOTE]
> 根据项目需求选择合适的架构：

1. **内容优先？** → Astro（零 JS） > Next.js > Nuxt
2. **需要 SEO？** → Next.js SSR/SSG > SPA
3. **团队熟悉 React？** → Next.js > SvelteKit
4. **需要极致性能？** → Astro + Islands
5. **全栈 TypeScript？** → Next.js > SvelteKit

### 成本估算

| 方案 | 月成本（100万 PV） | 说明 |
|------|-------------------|------|
| **Vercel Pro** | $20 + 超额计费 | 包含带宽、Serverless 函数 |
| **自托管（VPS）** | $20-$100 | 取决于配置 |
| **AWS ECS** | $50-$500 | 弹性伸缩 |

---

## 附录：常见问题

### Q: Next.js 和 Remix 哪个更好？

**答：** Next.js 在生态、部署和社区活跃度上占优；Remix 在 Web 标准遵循和错误处理上更优雅。对于新项目，Next.js 15 是更稳妥的选择。

### Q: App Router 如何处理认证？

**答：** 使用 Middleware 进行中间件层认证，或在 Server Components 中直接检查 session。NextAuth.js v5 已原生支持 App Router。

### Q: 如何调试 Server Components？

**答：** Server Components 输出无法在浏览器 DevTools 中直接查看。可在服务端 console.log 查看日志，或使用 `react-scan` 等工具。

---

> [!SUCCESS]
> 本文档系统性地介绍了 Next.js 的核心概念、App Router 与 Pages Router 的对比、多种渲染模式、Server Actions、Route Handlers 以及 Next.js 15 的最新特性。Next.js 凭借其完善的生态和持续迭代，是 vibecoding 时代前端开发的首选元框架。

---

## 完整安装指南

### 环境要求与前置准备

在开始安装 Next.js 之前，需要确保开发环境满足以下要求：

**Node.js 版本要求：**
- Next.js 15: Node.js 18.17 或更高版本
- 推荐使用 Node.js 20 LTS 或 22 LTS
- 建议使用 `nvm` 或 `fnm` 管理 Node.js 版本

```bash
# 使用 nvm 安装 Node.js 20
nvm install 20
nvm use 20

# 验证安装
node --version  # 应显示 v20.x.x
npm --version   # 应显示 10.x.x
```

**包管理器选择：**
Next.js 支持 npm、yarn、pnpm、bun 等主流包管理器。推荐使用 pnpm 或 bun 以获得更好的性能和依赖管理体验。

```bash
# 安装 pnpm
npm install -g pnpm

# 安装 bun
curl -fsSL https://bun.sh/install | bash
```

### 项目创建详细流程

**方式一：使用 create-next-app（推荐）**

```bash
# 交互式创建（推荐新手）
npx create-next-app@latest my-app

# 非交互式创建（高级用户）
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-npm \
  --no-turbopack

# 使用 App Router + Turbopack
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-pnpm \
  --turbopack
```

**create-next-app 选项详解：**

| 选项 | 说明 | 可选值 |
|------|------|--------|
| `--typescript` | 启用 TypeScript | 默认启用 |
| `--tailwind` | 使用 Tailwind CSS | 默认启用 |
| `--eslint` | 启用 ESLint | 默认启用 |
| `--app` | 使用 App Router | 默认 App Router |
| `--src-dir` | 使用 src 目录 | 推荐启用 |
| `--import-alias` | 路径别名 | 默认 `@/*` |
| `--no-app` | 使用 Pages Router | 旧项目迁移 |
| `--turbopack` | 使用 Turbopack | 开发环境加速 |

**方式二：手动创建 Next.js 项目**

```bash
# 创建项目目录
mkdir my-app && cd my-app

# 初始化 npm 项目
npm init -y

# 安装核心依赖
npm install next@latest react@latest react-dom@latest

# 安装开发依赖
npm install -D typescript @types/react @types/node

# 创建项目结构
mkdir -p src/app src/components src/lib src/styles
```

**package.json 配置：**

```json
{
  "name": "my-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0",
    "@types/react": "^19.0.0",
    "typescript": "^5.6.0"
  }
}
```

### TypeScript 配置

**tsconfig.json 基础配置：**

```json
{
  "compilerOptions": {
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 目录结构最佳实践

**推荐的 App Router 项目结构：**

```
my-app/
├── src/
│   ├── app/                    # App Router 目录
│   │   ├── (marketing)/        # 路由组 - 营销页面
│   │   │   ├── page.tsx       # 首页
│   │   │   ├── about/
│   │   │   └── pricing/
│   │   ├── (dashboard)/        # 路由组 - 仪表板
│   │   │   ├── layout.tsx      # 仪表板布局
│   │   │   ├── overview/
│   │   │   ├── settings/
│   │   │   └── users/
│   │   ├── api/                # API 路由
│   │   │   ├── users/
│   │   │   └── posts/
│   │   ├── layout.tsx          # 根布局
│   │   ├── page.tsx            # 首页
│   │   └── not-found.tsx       # 404 页面
│   ├── components/              # 组件
│   │   ├── ui/                 # shadcn/ui 组件
│   │   ├── forms/              # 表单组件
│   │   └── layout/             # 布局组件
│   ├── lib/                    # 工具函数
│   │   ├── db.ts               # 数据库客户端
│   │   ├── auth.ts             # 认证工具
│   │   └── utils.ts             # 通用工具
│   ├── actions/                 # Server Actions
│   ├── hooks/                   # 自定义 Hooks
│   └── types/                  # 类型定义
├── public/                     # 静态资源
├── styles/                     # 全局样式
├── .env.local                  # 本地环境变量
├── .env.production              # 生产环境变量
├── next.config.js              # Next.js 配置
├── tailwind.config.ts          # Tailwind 配置
└── tsconfig.json               # TypeScript 配置
```

---

## 认证方案详解

### NextAuth.js v5 (Auth.js)

NextAuth.js（现称 Auth.js）是 Next.js 官方推荐的认证解决方案，提供开箱即用的 OAuth、JWT、凭据登录支持。

**安装配置：**

```bash
npm install next-auth@beta
```

**核心配置：**

```typescript
// auth.config.ts
import type { NextAuthConfig } from 'next-auth';

export const authConfig = {
  pages: {
    signIn: '/auth/signin',
    error: '/auth/error',
  },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user;
      const isOnDashboard = nextUrl.pathname.startsWith('/dashboard');
      
      if (isOnDashboard) {
        if (isLoggedIn) return true;
        return false; // 重定向到登录页
      }
      return true;
    },
  },
  providers: [
    // Google OAuth
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
    // GitHub OAuth
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
    }),
    // 邮箱密码登录（凭据 Provider）
    Credentials({
      name: 'Credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        // 在此处验证用户
        const user = await validateUser(credentials.email, credentials.password);
        if (user) return user;
        return null;
      },
    }),
  ],
} satisfies NextAuthConfig;
```

**中间件配置：**

```typescript
// middleware.ts
import NextAuth from 'next-auth';
import { authConfig } from './auth.config';

export default NextAuth(authConfig).auth;

export const config = {
  // 需要认证的路由
  matcher: ['/dashboard/:path*', '/settings/:path*'],
};
```

**登录页面：**

```tsx
// app/auth/signin/page.tsx
'use client';

import { signIn } from '@/auth';
import { useSearchParams } from 'next/navigation';

export default function SignInPage() {
  const searchParams = useSearchParams();
  const callbackUrl = searchParams.get('callbackUrl') || '/dashboard';
  const error = searchParams.get('error');

  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="w-full max-w-md space-y-6">
        <h1 className="text-2xl font-bold">登录</h1>
        
        {error && (
          <div className="bg-red-100 text-red-600 p-3 rounded">
            {error === 'CredentialsSignin' ? '邮箱或密码错误' : '登录失败'}
          </div>
        )}
        
        {/* OAuth 登录 */}
        <form action={async () => {
          'use server';
          await signIn('google', { redirectTo: callbackUrl });
        }}>
          <button type="submit" className="w-full bg-white border p-2 rounded">
            使用 Google 登录
          </button>
        </form>
        
        {/* 邮箱密码登录 */}
        <SignInForm callbackUrl={callbackUrl} />
      </div>
    </div>
  );
}
```

**Server Actions 登录：**

```typescript
// actions.ts
'use server';

import { signIn } from '@/auth';
import { redirect } from 'next/navigation';

export async function signInWithCredentials(formData: FormData) {
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;
  
  try {
    await signIn('credentials', {
      email,
      password,
      redirectTo: '/dashboard',
    });
  } catch (error) {
    if (error instanceof RedirectToSignIn) {
      redirect('/auth/signin?error=CredentialsSignin');
    }
    throw error;
  }
}

export async function signOut() {
  await signOut({ redirectTo: '/' });
}
```

### Clerk 集成

Clerk 是另一个流行的认证解决方案，提供更丰富的 UI 组件和托管页面。

**安装：**

```bash
npm install @clerk/nextjs
```

**环境变量：**

```env
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard
```

**中间件：**

```typescript
// middleware.ts
import { clerkMiddleware, createRouteMatcher } from '@clerk/nextjs/server';

const isProtectedRoute = createRouteMatcher([
  '/dashboard(.*)',
  '/settings(.*)',
]);

export default clerkMiddleware((auth, req) => {
  if (isProtectedRoute(req)) {
    auth().protect();
  }
});

export const config = {
  matcher: ['/((?!.*\\..*|_next).*)', '/', '/(api|trpc)(.*)'],
};
```

**使用 Clerk 组件：**

```tsx
// app/layout.tsx
import { ClerkProvider } from '@clerk/nextjs';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <ClerkProvider>
      <html lang="zh-CN">
        <body>{children}</body>
      </html>
    </ClerkProvider>
  );
}
```

```tsx
// app/sign-in/[[...sign-in]]/page.tsx
import { SignIn } from '@clerk/nextjs';

export default function Page() {
  return <SignIn />;
}
```

```tsx
// app/dashboard/page.tsx
import { auth, currentUser } from '@clerk/nextjs/server';
import { redirect } from 'next/navigation';

export default async function DashboardPage() {
  const { userId } = auth();
  
  if (!userId) {
    redirect('/sign-in');
  }
  
  const user = await currentUser();
  
  return (
    <div>
      <h1>欢迎回来，{user?.firstName}</h1>
    </div>
  );
}
```

### 认证方案对比

| 特性 | NextAuth.js | Clerk | Auth0 |
|------|------------|-------|--------|
| **价格** | 开源免费 | 免费额度 + 付费 | 免费额度 + 付费 |
| **UI 组件** | 需自建 | 丰富托管组件 | 需自建 |
| **OAuth 提供商** | 多种 | 多种 | 多种 |
| **托管页面** | 否 | 是 | 否 |
| **React 生态** | 是 | 是 | 是 |
| **SSR 支持** | 原生 | 原生 | 需配置 |
| **社交登录** | 支持 | 支持 | 支持 |

---

## 完整 CRUD 示例

### 项目结构

本节展示一个完整的博客系统 CRUD 实现，包括文章创建、读取、更新、删除功能。

**数据库 Schema（Prisma）：**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Post {
  id          String   @id @default(cuid())
  title       String
  slug        String   @unique
  excerpt     String?
  content     String
  featuredImage String?
  published   Boolean  @default(false)
  authorId    String
  author      User     @relation(fields: [authorId], references: [id])
  tags        Tag[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  image     String?
  posts     Post[]
  createdAt DateTime @default(now())
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]
}
```

### 创建文章（Create）

```tsx
// app/blog/new/page.tsx
import { createPost } from '@/actions';
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';

export default async function NewPostPage() {
  const session = await getServerSession();
  
  if (!session?.user) {
    redirect('/api/auth/signin');
  }

  return (
    <div className="max-w-2xl mx-auto py-8">
      <h1 className="text-3xl font-bold mb-8">创建新文章</h1>
      <PostForm action={createPost} />
    </div>
  );
}
```

```tsx
// components/PostForm.tsx
'use client';

import { useFormStatus } from 'react-dom';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Label } from '@/components/ui/label';
import { useActionState } from 'react';
import { createPost, State } from '@/actions';
import { useRouter } from 'next/navigation';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <Button type="submit" disabled={pending}>
      {pending ? '发布中...' : '发布文章'}
    </Button>
  );
}

export function PostForm({ 
  action, 
  initialValues 
}: { 
  action: (prevState: State, formData: FormData) => Promise<State>;
  initialValues?: { title?: string; content?: string };
}) {
  const [state, formAction] = useActionState(action, null);
  const router = useRouter();

  // 成功后跳转
  if (state?.success) {
    router.push('/blog');
    router.refresh();
  }

  return (
    <form action={formAction} className="space-y-6">
      {state?.errors?._form && (
        <div className="bg-red-100 text-red-600 p-3 rounded">
          {state.errors._form}
        </div>
      )}

      <div className="space-y-2">
        <Label htmlFor="title">标题</Label>
        <Input
          id="title"
          name="title"
          defaultValue={initialValues?.title}
          placeholder="输入文章标题"
        />
        {state?.errors?.title && (
          <p className="text-sm text-red-600">{state.errors.title[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="slug">URL Slug</Label>
        <Input
          id="slug"
          name="slug"
          placeholder="article-url-slug"
        />
        {state?.errors?.slug && (
          <p className="text-sm text-red-600">{state.errors.slug[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="excerpt">摘要</Label>
        <Textarea
          id="excerpt"
          name="excerpt"
          defaultValue={initialValues?.excerpt}
          placeholder="文章摘要（可选）"
          rows={3}
        />
      </div>

      <div className="space-y-2">
        <Label htmlFor="content">正文内容</Label>
        <Textarea
          id="content"
          name="content"
          defaultValue={initialValues?.content}
          placeholder="使用 Markdown 编写文章内容"
          rows={20}
        />
        {state?.errors?.content && (
          <p className="text-sm text-red-600">{state.errors.content[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="tags">标签（逗号分隔）</Label>
        <Input
          id="tags"
          name="tags"
          placeholder="react, nextjs, typescript"
        />
      </div>

      <div className="flex gap-4">
        <SubmitButton />
        <Button type="button" variant="outline" onClick={() => router.back()}>
          取消
        </Button>
      </div>
    </form>
  );
}
```

```typescript
// actions/post.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { db } from '@/lib/db';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

const PostSchema = z.object({
  title: z.string().min(1, '标题不能为空').max(200, '标题过长'),
  slug: z.string()
    .min(1, 'Slug 不能为空')
    .max(100, 'Slug 过长')
    .regex(/^[a-z0-9-]+$/, 'Slug 只能包含小写字母、数字和连字符')
    .transform(v => v.toLowerCase()),
  excerpt: z.string().max(500, '摘要过长').optional().or(z.literal('')),
  content: z.string().min(10, '内容至少需要 10 个字符'),
  tags: z.string().optional(),
});

export type State = {
  errors?: {
    title?: string[];
    slug?: string[];
    excerpt?: string[];
    content?: string[];
    _form?: string[];
  };
  success?: boolean;
  message?: string;
};

export async function createPost(prevState: State, formData: FormData) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user?.email) {
    return {
      errors: { _form: ['请先登录'] },
      success: false,
    };
  }

  const validatedFields = PostSchema.safeParse({
    title: formData.get('title'),
    slug: formData.get('slug'),
    excerpt: formData.get('excerpt'),
    content: formData.get('content'),
    tags: formData.get('tags'),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      success: false,
    };
  }

  try {
    // 获取用户
    const user = await db.user.findUnique({
      where: { email: session.user.email },
    });

    if (!user) {
      return {
        errors: { _form: ['用户不存在'] },
        success: false,
      };
    }

    // 处理标签
    const tagNames = validatedFields.data.tags
      ?.split(',')
      .map(t => t.trim())
      .filter(Boolean) || [];

    const tags = await Promise.all(
      tagNames.map(async (name) => {
        const slug = name.toLowerCase().replace(/\s+/g, '-');
        return db.tag.upsert({
          where: { slug },
          create: { name, slug },
          update: {},
        });
      })
    );

    // 创建文章
    await db.post.create({
      data: {
        title: validatedFields.data.title,
        slug: validatedFields.data.slug,
        excerpt: validatedFields.data.excerpt || null,
        content: validatedFields.data.content,
        authorId: user.id,
        tags: {
          connect: tags.map(t => ({ id: t.id })),
        },
      },
    });
  } catch (error) {
    if (error instanceof Error && error.message.includes('Unique constraint')) {
      return {
        errors: { slug: ['该 Slug 已存在'] },
        success: false,
      };
    }
    console.error('创建文章失败:', error);
    return {
      errors: { _form: ['创建文章失败，请重试'] },
      success: false,
    };
  }

  revalidatePath('/blog');
  return {
    success: true,
    message: '文章创建成功',
  };
}
```

### 读取文章（Read）

```typescript
// app/blog/[slug]/page.tsx
import { db } from '@/lib/db';
import { notFound } from 'next/navigation';
import { Metadata } from 'next';
import { LikeButton } from '@/components/LikeButton';

interface Props {
  params: Promise<{ slug: string }>;
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const { slug } = await params;
  const post = await db.post.findUnique({
    where: { slug, published: true },
  });

  if (!post) {
    return { title: '文章未找到' };
  }

  return {
    title: post.title,
    description: post.excerpt || undefined,
    openGraph: {
      title: post.title,
      description: post.excerpt || undefined,
      images: post.featuredImage ? [post.featuredImage] : undefined,
    },
  };
}

export default async function BlogPostPage({ params }: Props) {
  const { slug } = await params;
  
  const post = await db.post.findUnique({
    where: { slug, published: true },
    include: {
      author: {
        select: { name: true, image: true },
      },
      tags: true,
    },
  });

  if (!post) {
    notFound();
  }

  // 获取相关文章
  const relatedPosts = await db.post.findMany({
    where: {
      published: true,
      id: { not: post.id },
      tags: {
        some: {
          id: { in: post.tags.map(t => t.id) },
        },
      },
    },
    take: 3,
    orderBy: { createdAt: 'desc' },
  });

  return (
    <article className="max-w-3xl mx-auto py-12">
      {/* 文章头部 */}
      <header className="mb-12">
        <div className="flex gap-2 mb-4">
          {post.tags.map(tag => (
            <span
              key={tag.id}
              className="px-3 py-1 text-sm bg-muted rounded-full"
            >
              {tag.name}
            </span>
          ))}
        </div>
        
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        
        <div className="flex items-center gap-4 text-muted-foreground">
          {post.author.image && (
            <img
              src={post.author.image}
              alt={post.author.name || ''}
              className="w-10 h-10 rounded-full"
            />
          )}
          <div>
            <p className="font-medium text-foreground">
              {post.author.name || '匿名'}
            </p>
            <p className="text-sm">
              {new Intl.DateTimeFormat('zh-CN', {
                year: 'numeric',
                month: 'long',
                day: 'numeric',
              }).format(post.createdAt)}
            </p>
          </div>
        </div>
      </header>

      {/* 文章内容 */}
      <div 
        className="prose prose-lg max-w-none"
        dangerouslySetInnerHTML={{ __html: post.content }}
      />

      {/* 点赞按钮 */}
      <div className="mt-12 pt-8 border-t">
        <LikeButton postId={post.id} />
      </div>

      {/* 相关文章 */}
      {relatedPosts.length > 0 && (
        <section className="mt-12 pt-8 border-t">
          <h2 className="text-2xl font-bold mb-6">相关文章</h2>
          <div className="grid gap-6 md:grid-cols-3">
            {relatedPosts.map(related => (
              <a
                key={related.id}
                href={`/blog/${related.slug}`}
                className="block p-4 border rounded-lg hover:border-primary transition-colors"
              >
                <h3 className="font-medium line-clamp-2">{related.title}</h3>
                <p className="text-sm text-muted-foreground mt-2">
                  {related.excerpt}
                </p>
              </a>
            ))}
          </div>
        </section>
      )}
    </article>
  );
}
```

### 更新文章（Update）

```typescript
// actions/post.ts (继续)
export async function updatePost(
  prevState: State,
  formData: FormData,
  postId: string
) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user?.email) {
    return {
      errors: { _form: ['请先登录'] },
      success: false,
    };
  }

  // 验证文章所有权
  const existingPost = await db.post.findUnique({
    where: { id: postId },
    include: { author: true },
  });

  if (!existingPost) {
    return {
      errors: { _form: ['文章不存在'] },
      success: false,
    };
  }

  if (existingPost.author.email !== session.user.email) {
    return {
      errors: { _form: ['无权限修改此文章'] },
      success: false,
    };
  }

  const validatedFields = PostSchema.partial().safeParse({
    title: formData.get('title'),
    slug: formData.get('slug'),
    excerpt: formData.get('excerpt'),
    content: formData.get('content'),
    tags: formData.get('tags'),
  });

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
      success: false,
    };
  }

  try {
    // 处理标签
    let tags: { connect: { id: string }[] } = {};
    if (validatedFields.data.tags) {
      const tagNames = validatedFields.data.tags
        .split(',')
        .map(t => t.trim())
        .filter(Boolean);

      const tagRecords = await Promise.all(
        tagNames.map(async (name) => {
          const slug = name.toLowerCase().replace(/\s+/g, '-');
          return db.tag.upsert({
            where: { slug },
            create: { name, slug },
            update: {},
          });
        })
      );

      tags = { connect: tagRecords.map(t => ({ id: t.id })) };
    }

    await db.post.update({
      where: { id: postId },
      data: {
        ...(validatedFields.data.title && { title: validatedFields.data.title }),
        ...(validatedFields.data.slug && { slug: validatedFields.data.slug }),
        ...(validatedFields.data.excerpt !== undefined && { excerpt: validatedFields.data.excerpt || null }),
        ...(validatedFields.data.content && { content: validatedFields.data.content }),
        ...(tags.connect.length > 0 && { tags }),
      },
    });
  } catch (error) {
    console.error('更新文章失败:', error);
    return {
      errors: { _form: ['更新文章失败，请重试'] },
      success: false,
    };
  }

  revalidatePath('/blog');
  revalidatePath(`/blog/${validatedFields.data.slug || existingPost.slug}`);
  return {
    success: true,
    message: '文章更新成功',
  };
}
```

```tsx
// app/blog/[slug]/edit/page.tsx
import { db } from '@/lib/db';
import { getServerSession } from 'next-auth';
import { redirect } from 'next/navigation';
import { updatePost } from '@/actions';
import { PostForm } from '@/components/PostForm';

interface Props {
  params: Promise<{ slug: string }>;
}

export default async function EditPostPage({ params }: Props) {
  const { slug } = await params;
  const session = await getServerSession();
  
  if (!session?.user?.email) {
    redirect('/api/auth/signin');
  }

  const post = await db.post.findUnique({
    where: { slug },
    include: { tags: true },
  });

  if (!post) {
    notFound();
  }

  if (post.author.email !== session.user.email) {
    redirect('/blog');
  }

  return (
    <div className="max-w-2xl mx-auto py-8">
      <h1 className="text-3xl font-bold mb-8">编辑文章</h1>
      <PostForm 
        action={(prevState, formData) => updatePost(prevState, formData, post.id)}
        initialValues={{
          title: post.title,
          content: post.content,
          excerpt: post.excerpt || undefined,
        }}
      />
    </div>
  );
}
```

### 删除文章（Delete）

```typescript
// actions/post.ts (继续)
export async function deletePost(postId: string) {
  const session = await getServerSession(authOptions);
  
  if (!session?.user?.email) {
    return { success: false, error: '请先登录' };
  }

  const post = await db.post.findUnique({
    where: { id: postId },
  });

  if (!post) {
    return { success: false, error: '文章不存在' };
  }

  if (post.authorId !== session.user.id) {
    return { success: false, error: '无权限删除此文章' };
  }

  try {
    await db.post.delete({
      where: { id: postId },
    });
  } catch (error) {
    return { success: false, error: '删除失败，请重试' };
  }

  revalidatePath('/blog');
  return { success: true };
}
```

```tsx
// components/DeletePostButton.tsx
'use client';

import { useTransition } from 'react';
import { deletePost } from '@/actions';
import { Button } from '@/components/ui/button';
import { useRouter } from 'next/navigation';

export function DeletePostButton({ postId }: { postId: string }) {
  const [isPending, startTransition] = useTransition();
  const router = useRouter();

  const handleDelete = () => {
    if (!confirm('确定要删除这篇文章吗？此操作不可撤销。')) {
      return;
    }

    startTransition(async () => {
      const result = await deletePost(postId);
      if (result.success) {
        router.push('/blog');
        router.refresh();
      } else {
        alert(result.error);
      }
    });
  };

  return (
    <Button
      variant="destructive"
      onClick={handleDelete}
      disabled={isPending}
    >
      {isPending ? '删除中...' : '删除文章'}
    </Button>
  );
}
```

---

## 常见陷阱与解决方案

### App Router 常见问题

**1. Server Component 导入 Client Component**

**问题：** `ClientComponent` 无法直接导入 `ServerComponent`。

```tsx
// ❌ 错误示例
// app/page.tsx (Server Component)
import ClientComponent from './ClientComponent';
import ServerComponent from './ServerComponent'; // 这会报错

export default function Page() {
  return (
    <ClientComponent>
      <ServerComponent /> {/* 错误：ServerComponent 不能作为子组件 */}
    </ClientComponent>
  );
}
```

**解决方案：** 通过 props 传递 Server Component 的渲染结果。

```tsx
// ✅ 正确示例
// app/page.tsx (Server Component)
import ClientComponent from './ClientComponent';

// ServerComponent 作为 prop 传递
export default async function Page() {
  const serverData = await fetchServerData();
  
  return (
    <ClientComponent>
      <ServerComponentRender data={serverData} />
    </ClientComponent>
  );
}
```

**2. async Server Component 中使用 useState/useEffect**

**问题：** 在 async 组件中使用 React hooks。

```tsx
// ❌ 错误示例
export default async function Page() {
  const data = await fetchData();
  const [state, setState] = useState(); // 错误！
  
  return <div>{data}</div>;
}
```

**解决方案：** 将客户端逻辑提取到子组件中。

```tsx
// ✅ 正确示例
// app/page.tsx (Server Component)
export default async function Page() {
  const data = await fetchData();
  
  return (
    <div>
      <ServerRenderedContent data={data} />
      <ClientInteractiveComponent /> {/* 独立的客户端组件 */}
    </div>
  );
}

// components/ClientInteractiveComponent.tsx
'use client';

export function ClientInteractiveComponent() {
  const [state, setState] = useState();
  // ... 客户端逻辑
}
```

**3. Next.js 15 缓存语义变更**

**问题：** 升级到 Next.js 15 后，数据获取行为发生变化。

```tsx
// ❌ Next.js 14 中的常见模式
const data = await fetch(url); // 默认缓存

// 在 Next.js 15 中，这不会缓存！
```

**解决方案：** 显式配置缓存策略。

```tsx
// ✅ Next.js 15 - 显式缓存
export default async function Page() {
  // 静态数据 - 需要缓存
  const staticData = await fetch('https://api.example.com/static', {
    cache: 'force-cache',
  });

  // 动态数据 - 不缓存
  const dynamicData = await fetch('https://api.example.com/dynamic', {
    cache: 'no-store',
  });

  // ISR - 按时间重新验证
  const isrData = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 },
  });

  return <div>{/* 渲染内容 */}</div>;
}
```

**4. 'use server' 文件顶层导出限制**

**问题：** Server Actions 中导出非异步函数。

```tsx
// ❌ 错误示例
'use server';

export function helper() {
  // 只能是异步函数
}

export async function serverAction() {
  helper(); // 调用同步函数
}
```

**解决方案：** 将所有函数标记为 async 或将同步逻辑内联。

```tsx
// ✅ 正确示例
'use server';

export async function helper() {
  // 转为异步
}

export async function serverAction() {
  await helper();
}
```

### 性能优化常见问题

**1. 避免瀑布式数据请求**

**问题：** 多个顺序的数据请求导致加载慢。

```tsx
// ❌ 瀑布式请求
export default async function Page() {
  const user = await getUser();           // 等待 200ms
  const posts = await getUserPosts();     // 再等 300ms
  const comments = await getUserComments(); // 再等 300ms
  // 总计: 800ms
}
```

**解决方案：** 并行请求。

```tsx
// ✅ 并行请求
export default async function Page() {
  const [user, posts, comments] = await Promise.all([
    getUser(),
    getUserPosts(),
    getUserComments(),
  ]);
  // 总计: max(200, 300, 300) = 300ms
}
```

**2. 正确使用 Image 组件**

**问题：** 图片未设置宽高或未使用正确的 sizes 属性。

```tsx
// ❌ 问题示例
<Image src={src} alt="alt" />
```

**解决方案：** 始终指定宽高和 sizes。

```tsx
// ✅ 正确示例
<Image
  src={src}
  alt="alt"
  width={800}
  height={600}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  priority={isAboveFold}
/>
```

**3. 避免不必要的 Client Components**

**问题：** 不必要的 'use client' 导致 bundle 变大。

```tsx
// ❌ 问题示例
'use client'; // 实际上这个组件不需要交互

export function Article({ title, content }) {
  return (
    <article>
      <h1>{title}</h1>
      <p>{content}</p>
    </article>
  );
}
```

**解决方案：** 去掉 'use client'，除非真的需要。

```tsx
// ✅ 正确示例
// 没有 'use client' - Server Component
export function Article({ title, content }) {
  return (
    <article>
      <h1>{title}</h1>
      <p>{content}</p>
    </article>
  );
}
```

### 部署常见问题

**1. 环境变量未正确配置**

**问题：** 生产环境变量未正确加载。

**解决方案：** 
- `.env.local` - 本地开发（不提交到 Git）
- `.env.development` - 开发环境
- `.env.production` - 生产环境
- `.env.example` - 环境变量示例（提交到 Git）

```bash
# .env.example - 记录需要的变量，但不包含敏感值
DATABASE_URL=your_database_url_here
NEXTAUTH_SECRET=your_secret_here
GOOGLE_CLIENT_ID=your_client_id_here
```

**2. Middleware 在 Edge Runtime 的限制**

**问题：** Middleware 在 Edge Runtime 运行，有 Node.js API 限制。

**解决方案：** 
- 避免使用 Node.js 特定 API
- 使用 Web API 或 Next.js 提供的 API
- 对于复杂逻辑，考虑在页面/路由级别处理

```typescript
// middleware.ts - Edge 兼容
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // ✅ Edge 兼容
  const token = request.cookies.get('token');
  
  // ❌ 不兼容 - Node.js API
  // const fs = require('fs');
  
  // ❌ 不兼容 - 某些 npm 包
  // import { db } from '@prisma/client';
}
```

---

## 与其他元框架详细对比

### Next.js vs Nuxt vs Astro vs SvelteKit vs Remix

### 技术架构深度对比

**渲染模型对比：**

| 维度 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **默认渲染** | SSR | SSR | SSG | SSR | SSR |
| **SSG 支持** | ✅ | ✅ | ✅ (默认) | ✅ | ⚠️ 需配置 |
| **ISR 支持** | ✅ 原生 | ⚠️ via routeRules | ⚠️ 需适配器 | ⚠️ 需插件 | ❌ |
| **边缘部署** | ✅ Vercel Edge | ✅ via Nitro | ✅ via 适配器 | ✅ via 适配器 | ✅ Cloudflare |
| **Streaming** | ✅ | ✅ | ✅ | ✅ | ✅ (原生) |
| **PPR/Islands** | ✅ (Partial) | ❌ | ✅ (Islands) | ❌ | ❌ |

**路由系统对比：**

| 特性 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **文件路由** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **嵌套布局** | ✅ | ✅ | ⚠️ 手动 | ✅ | ✅ (原生) |
| **路由组** | ✅ (文件夹) | ✅ (前缀) | ✅ (前缀) | ✅ (前缀) | ✅ |
| **平行路由** | ✅ (@) | ❌ | ❌ | ❌ | ❌ |
| **拦截路由** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **动态段** | [param] | [param] | [param] | [param] | $param |

**数据获取对比：**

| 方法 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **服务端获取** | Server Components | useAsyncData | frontmatter | +page.server.ts | loader |
| **客户端获取** | useEffect/SWR | useFetch | 客户端 island | +page.ts | useFetcher |
| **表单提交** | Server Actions | Server handlers | 表单 + 适配器 | Form actions | action |
| **类型安全** | ✅ (推断) | ✅ (推断) | ✅ (Zod) | ✅ (推断) | ✅ (推断) |

### 性能基准对比

| 指标 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **首页 LCP (SSR)** | ~1.2s | ~1.3s | ~0.4s (SSG) | ~1.1s | ~1.2s |
| **首页 LCP (SSG)** | ~0.6s | ~0.6s | ~0.3s | ~0.5s | N/A |
| **JS Bundle (基础)** | ~80KB | ~85KB | ~0KB | ~5KB | ~90KB |
| **Time to Interactive** | ~1.8s | ~1.9s | ~0.5s | ~1.2s | ~1.8s |
| **Lighthouse Score** | 90-95 | 88-93 | 98-100 | 92-97 | 88-93 |

### 生态与社区对比

| 维度 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **GitHub Stars** | ~125K | ~55K | ~45K | ~22K | ~23K |
| **周下载量** | ~5M | ~1M | ~800K | ~300K | ~150K |
| **npm 包生态** | 庞大 | 丰富 | 增长中 | 较小 | 较小 |
| **模板/Starters** | 丰富 | 丰富 | 丰富 | 中等 | 中等 |
| **企业采用** | 主流 | 常见 | 增长中 | 增长中 | 增长中 |
| **文档质量** | 优秀 | 优秀 | 优秀 | 良好 | 良好 |

### 适用场景决策矩阵

| 场景 | 推荐选择 | 备选选择 | 不推荐 |
|------|---------|---------|--------|
| **内容博客/文档站** | Astro | Nuxt/Next.js | Remix |
| **电商/产品目录** | Next.js/Nuxt | Astro | Remix |
| **SaaS/管理后台** | Next.js/Nuxt | SvelteKit | Astro |
| **AI 应用/聊天** | Next.js | SvelteKit | Remix |
| **企业官网** | Astro/Next.js | Nuxt | Remix |
| **移动端 H5 应用** | Nuxt | Next.js | Astro |
| **静态营销页** | Astro | Next.js (SSG) | Remix |
| **电商购物车** | Next.js/Nuxt | SvelteKit | Remix |
| **社交媒体应用** | Next.js | SvelteKit | Remix |
| **数据仪表板** | Next.js | Nuxt | SvelteKit |

### 学习曲线对比

| 维度 | Next.js | Nuxt | Astro | SvelteKit | Remix |
|------|---------|------|-------|-----------|-------|
| **基础入门** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| **进阶特性** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **性能调优** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **调试排错** | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **整体难度** | 中等 | 中等 | 简单 | 简单 | 中等 |

---

## Next.js 生态工具链

### 推荐工具组合

**开发环境：**
- Node.js 20+ (推荐使用 fnm 或 nvm 管理)
- pnpm 9+ (包管理器)
- VS Code + Volar/ESLint 插件
- Cursor/Windsurf (AI 辅助编程)

**代码质量：**
- TypeScript (严格模式)
- ESLint + Prettier
- Husky (Git hooks)
- lint-staged

**测试：**
- Vitest (单元测试)
- Playwright (E2E 测试)
- Testing Library (组件测试)

**部署：**
- Vercel (推荐)
- AWS Amplify
- Docker
- Railway/Render

### 推荐的 package.json 脚本

```json
{
  "scripts": {
    "dev": "next dev",
    "dev:turbopack": "next dev --turbopack",
    "build": "next build",
    "build:analyze": "ANALYZE=true next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:e2e": "playwright test",
    "format": "prettier --write .",
    "prepare": "husky install"
  }
}
```

---

## 附录：资源链接

### 官方资源

- [Next.js 官方文档](https://nextjs.org/docs)
- [Next.js GitHub](https://github.com/vercel/next.js)
- [Next.js Discord](https://discord.gg/nextjs)
- [Next.js 博客](https://nextjs.org/blog)

### 学习资源

- [Next.js 官方教程](https://nextjs.org/learn)
- [Next.js 示例集合](https://github.com/vercel/next.js/tree/canary/examples)
- [React Server Components 文档](https://react.dev/reference/rsc)

### 相关工具

- [Prisma](https://prisma.io) - 数据库 ORM
- [Zod](https://zod.dev) - 数据验证
- [Turbopack](https://turbo.build/pack) - 构建工具
- [Tailwind CSS](https://tailwindcss.com) - 样式框架
