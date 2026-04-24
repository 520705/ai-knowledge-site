# Astro 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Astro 5 最新特性、Islands 架构、Content Collections、MDX 支持及与主流框架的深度对比。

---

## 目录

1. [[#Astro 概述与核心哲学]]
2. [[#Islands 架构详解]]
3. [[#渲染模式对比]]
4. [[#Content Collections]]
5. [[#MDX 支持]]
6. [[#Astro DB]]
7. [[#组件开发]]
8. [[#Astro 5 新特性]]
9. [[#场景对比表]]
10. [[#实战场景与选型建议]]

---

## Astro 概述与核心哲学

### Astro 的定位

Astro 是一个「内容优先」的现代 Web 框架，其核心理念是「**发送更少的 JavaScript**」。在 2026 年的前端生态中，Astro 凭借其独特的 Islands 架构和零 JS 哲学，在以下场景中占据独特地位：

- **内容驱动型网站**：博客、文档、营销页、新闻站点
- **性能优先项目**：需要极致首屏性能的企业官网
- **静态内容为主**：内容多、交互少的网站
- **SEO 敏感站点**：需要完美 HTML 输出的场景

### 零 JS 哲学

Astro 的核心设计理念是：**默认不发送 JavaScript**。

| 框架 | 默认 JS | 典型页面 JS 体积 |
|------|--------|----------------|
| **Astro** | 零 JS | 0 KB |
| **Next.js** | 全部 SSR | 50-200 KB |
| **Nuxt** | 全部 SSR | 50-200 KB |
| **Remix** | 全部 SSR | 80-250 KB |

> [!IMPORTANT]
> Astro 的「零 JS」是指页面默认不包含任何 JavaScript，除非明确引入交互组件。这种设计使 Astro 页面加载速度极快，LCP（最大内容绘制）接近纯 HTML。

### 核心架构

```
┌─────────────────────────────────────────────────────────┐
│                      Astro Page                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │  Static      │  │  Islands    │  │  Interactive    │ │
│  │  HTML        │  │  (Partial)  │  │  Components     │ │
│  │  (No JS)    │  │  React/Vue  │  │  (Client)      │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
│                                                          │
│  Build Time: Static HTML Generated                       │
│  Runtime: Selective Hydration                           │
└─────────────────────────────────────────────────────────┘
```

---

## Islands 架构详解

### 什么是 Islands

Islands（岛屿）架构是一种**部分 hydration**策略，允许多个孤立的组件独立激活，而不需要整个页面成为 SPA。

### Islands vs SSR vs SSG 对比表

| 特性 | Islands（Astro） | 完整 SSR（Next.js/Nuxt） | 纯 SSG | SPA |
|------|----------------|-------------------------|--------|-----|
| **首屏性能** | 极快 | 快 | 最快 | 慢 |
| **SEO** | 完美 | 完美 | 完美 | 需额外处理 |
| **交互能力** | 按需 | 全部 | 无 | 全部 |
| **JS Bundle** | 最小 | 中等 | 零 | 全部 |
| **开发复杂度** | 低 | 中等 | 低 | 高 |
| **实时数据** | 需客户端获取 | 可 SSR | 无 | 需客户端获取 |
| **适用场景** | 内容站、博客 | 全栈应用 | 纯静态 | Web 应用 |

### Islands 指令

```astro
---
import Header from './Header.astro';
import Counter from './Counter.jsx';  // React 组件
import SearchBox from './Search.svelte';  // Svelte 组件
---

<!-- 静态组件 - 无 JS -->
<Header />

<!-- Islands 1: 页面加载时立即激活 -->
<Counter client:load />

<!-- Islands 2: 可见时激活（推荐） -->
<SearchBox client:visible />

<!-- Islands 3: 空闲时激活 -->
<Newsletter client:idle />

<!-- Islands 4: 媒体查询触发 -->
<MobileMenu client:media="(max-width: 768px)" />
```

### Islands 指令详解

| 指令 | 激活时机 | 适用场景 |
|------|---------|---------|
| `client:load` | 页面加载时立即 | 首屏必要交互 |
| `client:idle` | 浏览器空闲时 | 非关键功能 |
| `client:visible` | 进入视口时 | 屏幕外内容 |
| `client:media` | 媒体查询匹配 | 响应式交互 |
| `client:only` | 仅客户端渲染 | 纯客户端组件 |

### Islands 通信

Astro Islands 之间默认隔离，但可通过以下方式进行通信：

```astro
---
// Preact 信号（Nano Stores）
import { signal } from '@nanostores/preact';
export const cart = signal({ items: [] });
---

<!-- 在 Island 组件中使用 -->
<script>
  import { useStore } from '@nanostores/preact';
  import { cart } from '../stores/cart';
  
  export default function CartIcon() {
    const cartData = useStore(cart);
    return <span>🛒 {cartData.items.length}</span>;
  }
</script>
```

---

## 渲染模式对比

### 支持的渲染模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **SSG** | 构建时生成静态 HTML | 博客、文档、营销页 |
| **SSR** | 按需渲染 | 需要实时数据 |
| **Hybrid** | 混合模式，部分 SSR | 内容 + 动态混合 |
| **按需 SSR** | 指定页面 SSR | 特定页面需要实时 |

### 静态生成（默认）

```astro
---
// src/pages/index.astro
// 默认 SSG，构建时生成
const posts = await fetch('https://api.example.com/posts').then(r => r.json());
---

<html>
  <head><title>我的博客</title></head>
  <body>
    <h1>最新文章</h1>
    {posts.map(post => (
      <article>
        <h2>{post.title}</h2>
        <p>{post.excerpt}</p>
      </article>
    ))}
  </body>
</html>
```

### 混合模式（SSG + SSR）

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  output: 'hybrid',
  server: {
    port: 4321,
  },
});
```

```astro
---
// src/pages/dashboard.astro
// 启用 SSR
export const prerender = false;

const user = await getUser(Astro.request);
if (!user) {
  return Astro.redirect('/login');
}
---

<html>
  <body>
    <h1>欢迎回来，{user.name}</h1>
  </body>
</html>
```

### 按需 SSR

```astro
---
// src/pages/api/data.json.ts
// API 路由
export const prerender = false;

export async function GET({ request }) {
  const data = await fetchData();
  return new Response(JSON.stringify(data), {
    headers: { 'Content-Type': 'application/json' },
  });
}
```

---

## Content Collections

### Content Collections 概述

Content Collections 是 Astro 2.0 引入的内容管理功能，提供类型安全的内容组织：

```
src/
├── content/
│   ├── config.ts           # 内容集合定义
│   ├── blog/
│   │   ├── hello-world.md
│   │   └── getting-started.md
│   └── docs/
│       ├── getting-started.md
│       └── configuration.md
```

### 定义内容集合

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',  // 'content' 或 'data'
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    author: z.string().default('Anonymous'),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
});

const docs = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    description: z.string(),
    order: z.number(),
    version: z.enum(['v1', 'v2', 'v3']),
  }),
});

export const collections = { blog, docs };
```

### 查询内容

```astro
---
import { getCollection, getEntry } from 'astro:content';

// 获取所有博客文章
const posts = await getCollection('blog', ({ data }) => {
  return !data.draft;  // 过滤草稿
});

// 按日期排序
posts.sort((a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());

// 获取单篇文章
const gettingStarted = await getEntry('docs', 'getting-started');
---

<html>
  <body>
    <h1>博客</h1>
    {posts.map(post => (
      <article>
        <h2><a href={`/blog/${post.slug}`}>{post.data.title}</a></h2>
        <time datetime={post.data.pubDate.toISOString()}>
          {post.data.pubDate.toLocaleDateString('zh-CN')}
        </time>
      </article>
    ))}
  </body>
</html>
```

### 渲染 Markdown/MDX

```astro
---
import { getCollection } from 'astro:content';

const posts = await getCollection('blog');
const { Content } = await posts[0].render();
---

<article>
  <h1>{posts[0].data.title}</h1>
  <Content />
</article>
```

---

## MDX 支持

### 安装 MDX

```bash
npx astro add mdx
```

### MDX 基础

```mdx
---
title: MDX 博客示例
---

export const example = "这是变量";

# 欢迎阅读 {example}

这是一个**加粗**和*斜体*的段落。

import MyComponent from '../components/MyComponent';

<MyComponent client:visible />
```

### MDX 组件覆盖

```astro
---
import { getCollection } from 'astro:content';
import BlogPost from '../layouts/BlogPost.astro';

const posts = await getCollection('blog');
---

{posts.map(async (post) => {
  const { Content } = await post.render();
  return (
    <BlogPost title={post.data.title}>
      <Content 
        components={{ 
          // 自定义组件映射
          h1: MyH1,
          Callout: CustomCallout,
        }} 
      />
    </BlogPost>
  );
})}
```

### MDX 代码块

````mdx
```typescript title="utils/helper.ts"
export function greet(name: string): string {
  return `Hello, ${name}!`;
}
```
````

---

## Astro DB

### Astro DB 概述

Astro DB 是 Astro 5 引入的内置数据库解决方案，基于 libSQL（SQLite 兼容）：

```bash
npx astro add db
```

### 定义表

```typescript
// src/db/config.ts
import { defineDb, defineTable, column, relations } from 'astro:db';

const authors = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    name: column.text(),
    email: column.text({ unique: true }),
    avatar: column.text().optional(),
  },
  relations: {
    posts: relation().hasMany(posts),
  },
});

const posts = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    title: column.text(),
    content: column.text(),
    published: column.boolean({ default: false }),
    authorId: column.number({ references: () => authors.columns.id }),
    createdAt: column.date({ default: new Date() }),
  },
  relations: {
    author: relation(authors, ({ one }) => one({
      fields: [posts.authorId],
      references: [authors.id],
    })),
  },
});

export default defineDb({
  tables: { authors, posts },
});
```

### 数据库操作

```astro
---
import { db, posts, authors } from 'astro:db';

// 插入数据
await db.insert(posts).values([
  { title: 'Hello World', content: '...', authorId: 1 },
]);

// 查询数据
const allPosts = await db.select().from(posts);
const publishedPosts = await db.select().from(posts).where(eq(posts.published, true));

// 关系查询
const postsWithAuthors = await db
  .select()
  .from(posts)
  .innerJoin(authors, eq(posts.authorId, authors.id));

// 更新数据
await db.update(posts)
  .set({ published: true })
  .where(eq(posts.id, 1));

// 删除数据
await db.delete(posts).where(eq(posts.id, 1));
---
```

---

## 组件开发

### Astro 组件

```astro
---
// src/components/Card.astro
interface Props {
  title: string;
  description: string;
  href: string;
  image?: string;
}

const { title, description, href, image } = Astro.props;
---

<article class="card">
  {image && <img src={image} alt={title} />}
  <h3><a href={href}>{title}</a></h3>
  <p>{description}</p>
</article>

<style>
  .card {
    border: 1px solid #e2e8f0;
    border-radius: 8px;
    padding: 1rem;
    transition: transform 0.2s;
  }
  .card:hover {
    transform: translateY(-4px);
  }
</style>
```

### 跨框架组件

Astro 支持 React、Vue、Svelte、Solid、Preact、Lit 等多种 UI 框架：

```bash
# 安装集成
npx astro add react    # React
npx astro add vue       # Vue
npx astro add svelte   # Svelte
npx astro add solid    # Solid
```

```astro
---
// 使用不同框架的组件
import Counter from '../components/Counter.jsx';    // React
import Toggle from '../components/Toggle.svelte';    // Svelte
import Clock from '../components/Clock.vue';         // Vue
---

<Counter client:load initialCount={0} />
<Toggle client:visible />
<Clock client:idle />
```

### 布局组件

```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  title: string;
  description?: string;
}

const { title, description = '默认描述' } = Astro.props;
---

<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>{title}</title>
    <meta name="description" content={description} />
  </head>
  <body>
    <header>
      <nav>
        <a href="/">首页</a>
        <a href="/blog">博客</a>
        <a href="/about">关于</a>
      </nav>
    </header>
    <main>
      <slot />  {/* 页面内容插槽 */}
    </main>
    <footer>
      <p>&copy; 2026 我的网站</p>
    </footer>
  </body>
</html>
```

---

## Astro 5 新特性

### Astro 5.0 主要更新

| 特性 | 说明 |
|------|------|
| **Astro DB** | 内置 SQLite 数据库 |
| **Content Layer** | 远程内容集合支持 |
| **better-react** | 优化的 React 集成 |
| **类型安全 Content** | 改进的类型检查 |
| **权限管理** | 实验性权限 API |

### Content Layer

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';
import { notionLoader } from 'astro-loader-notion';
import { vercelBlobLoader } from 'astro-loader-vercel';

const blog = defineCollection({
  loader: notionLoader({
    databaseId: process.env.NOTION_DATABASE_ID,
    auth: process.env.NOTION_TOKEN,
  }),
  schema: z.object({
    title: z.string(),
    description: z.string(),
    publishDate: z.coerce.date(),
  }),
});

const images = defineCollection({
  loader: vercelBlobLoader({
    token: process.env.BLOB_READ_WRITE_TOKEN,
  }),
  schema: z.object({
    alt: z.string(),
    category: z.string(),
  }),
});

export const collections = { blog, images };
```

### Server Output Mode

```typescript
// astro.config.mjs
export default defineConfig({
  output: 'server',  // 'static' | 'hybrid' | 'server'
  adapter: node({
    mode: 'standalone',
  }),
});
```

---

## 场景对比表

### Astro vs Next.js vs Nuxt 场景对比

| 场景 | Astro | Next.js | Nuxt |
|------|-------|---------|------|
| **博客/内容站** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **文档站点** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **营销网站** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **电商产品页** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **管理后台** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Web 应用** | ⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **需要 SEO** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **极致性能** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **开发体验** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **生态丰富度** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

> [!TIP]
> 选择建议：
> - **内容为主、交互少** → Astro（最佳性能）
> - **全栈应用、需要 API** → Next.js 或 Nuxt
> - **熟悉 React** → Next.js
> - **熟悉 Vue** → Nuxt
> - **团队需要极简** → Astro

---

## 实战场景与选型建议

### 博客系统实战

```typescript
// src/pages/blog/[...slug].astro
import { getCollection } from 'astro:content';
import BaseLayout from '../../layouts/BaseLayout.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog', ({ data }) => !data.draft);
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BaseLayout title={post.data.title} description={post.data.description}>
  <article>
    <header>
      <h1>{post.data.title}</h1>
      <time datetime={post.data.pubDate.toISOString()}>
        {post.data.pubDate.toLocaleDateString('zh-CN')}
      </time>
    </header>
    <Content />
  </article>
</BaseLayout>
```

### AI 对话集成实战

```astro
---
// src/pages/chat.astro
import ChatInterface from '../components/ChatInterface';

export const prerender = false;
---

<html>
  <body>
    <ChatInterface client:load />
  </body>
</html>
```

```typescript
// src/components/ChatInterface.tsx
import { useState } from 'react';

export default function ChatInterface() {
  const [messages, setMessages] = useState<Array<{role: string, content: string}>>([]);
  const [input, setInput] = useState('');

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault();
    const userMessage = { role: 'user', content: input };
    setMessages(prev => [...prev, userMessage]);
    setInput('');

    const res = await fetch('/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ messages: [...messages, userMessage] }),
    });

    const data = await res.json();
    setMessages(prev => [...prev, { role: 'assistant', content: data.content }]);
  }

  return (
    <div className="chat-container">
      <div className="messages">
        {messages.map((msg, i) => (
          <div key={i} className={msg.role}>{msg.content}</div>
        ))}
      </div>
      <form onSubmit={handleSubmit}>
        <input value={input} onChange={e => setInput(e.target.value)} />
        <button type="submit">发送</button>
      </form>
    </div>
  );
}
```

### 性能对比数据

| 指标 | Astro | Next.js | Nuxt |
|------|-------|---------|------|
| **首页 LCP** | ~0.5s | ~1.5s | ~1.5s |
| **JS Bundle** | 0 KB | ~150 KB | ~150 KB |
| **Time to Interactive** | ~0.5s | ~2s | ~2s |
| **Lighthouse 分数** | 98-100 | 85-95 | 85-95 |

---

> [!SUCCESS]
> Astro 是内容优先 Web 开发的最佳选择，其 Islands 架构和零 JS 哲学为内容网站提供了极致的性能和 SEO 表现。通过 Content Collections 和 Astro DB，Astro 已经具备了完善的内容管理能力。对于博客、文档、营销页等场景，Astro 是性能和开发体验的完美平衡点。

---

## 完整安装指南

### 环境要求与前置准备

**Node.js 版本要求：**
- Astro 5: Node.js 18.17 或更高版本
- 推荐使用 Node.js 20 LTS 或 22 LTS
- Astro 支持 pnpm、npm、yarn、bun 等包管理器

```bash
# 使用 bun 安装 Node.js（如果需要）
curl -fsSL https://bun.sh/install | bash
bun --version  # 应显示 1.x.x
```

### 项目创建详细流程

**方式一：使用 create-astro（推荐）**

```bash
# 交互式创建
npm create astro@latest

# 快速创建（跳过交互）
npm create astro@latest my-site -- --template minimal --install --no-git --typescript strict

# 使用特定模板
npm create astro@latest my-site -- --template blog
npm create astro@latest my-site -- --template portfolio
npm create astro@latest my-site -- --template docs
npm create astro@latest my-site -- --template landing
npm create astro@latest my-site -- --template portfolio
npm create astro@latest my-site -- --template empty
```

**方式二：使用 Astro CLI 添加到现有项目**

```bash
# 在现有项目中添加 Astro
npx astro add react    # 添加 React 集成
npx astro add vue      # 添加 Vue 集成
npx astro add svelte  # 添加 Svelte 集成
npx astro add tailwind # 添加 Tailwind 集成
```

**方式三：手动创建 Astro 项目**

```bash
# 创建项目目录
mkdir my-site && cd my-site

# 初始化 npm 项目
npm init -y

# 安装 Astro 核心依赖
npm install astro@latest

# 安装开发依赖
npm install -D typescript

# 创建项目结构
mkdir -p src/pages src/components src/layouts public
touch src/pages/index.astro src/layouts/Layout.astro
```

**package.json 配置：**

```json
{
  "name": "my-site",
  "type": "module",
  "version": "1.0.0",
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "astro": "astro"
  },
  "dependencies": {
    "astro": "^5.0.0"
  }
}
```

### Astro 配置详解

**astro.config.mjs 完整配置：**

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';

// 集成
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import svelte from '@astrojs/svelte';
import tailwind from '@astrojs/tailwind';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import partytown from '@astrojs/partytown';
import vercel from '@astrojs/vercel';

// Node 适配器（SSR）
import node from '@astrojs/node';

export default defineConfig({
  // === 站点配置 ===
  site: 'https://example.com',
  base: '/',

  // === 输出模式 ===
  // 'static' | 'hybrid' | 'server'
  output: 'static',

  // === 集成配置 ===
  integrations: [
    // React 支持
    react(),
    // Vue 支持
    vue(),
    // Svelte 支持
    svelte(),
    // Tailwind CSS
    tailwind({
      applyBaseStyles: false,
    }),
    // MDX 支持
    mdx({
      syntaxHighlight: 'shiki',
      shikiConfig: {
        theme: 'github-dark',
      },
    }),
    // sitemap
    sitemap(),
    // 分析工具
    partytown({
      config: {
        forward: ['dataLayer.push'],
      },
    }),
  ],

  // === Vite 配置 ===
  vite: {
    optimizeDeps: {
      include: ['react', 'react-dom'],
    },
    build: {
      cssMinify: true,
    },
    ssr: {
      noExternal: ['@astrojs/react'],
    },
  },

  // === 构建选项 ===
  build: {
    format: 'directory',
    assets: '_assets',
    clientBuildStats: true,
  },

  // === 图片优化 ===
  image: {
    domains: ['images.unsplash.com', 'picsum.photos'],
    remotePatterns: [{ protocol: 'https' }],
    service: {
      entrypoint: 'astro/assets/services/sharp',
      config: {
        limitInputPixels: false,
      },
    },
  },

  // === Markdown 配置 ===
  markdown: {
    syntaxHighlight: 'shiki',
    shikiConfig: {
      theme: 'github-dark',
      langs: [],
      wrap: true,
    },
    remarkPlugins: [],
    rehypePlugins: [],
  },

  // === 服务端选项（SSR） ===
  server: {
    host: true,
    port: 4321,
    open: true,
    streaming: true,
  },

  // === 适配器（SSR/Hybrid） ===
  adapter: vercel({
    imageService: true,
    webAnalytics: { enabled: true },
    speedInsights: { enabled: true },
  }),

  // === 实验性功能 ===
  experimental: {
    clientPrerender: true,
    i18nDomains: true,
  },

  // === 兼容性 ===
  compatibilityDate: '2024-12-01',
});
```

### 目录结构最佳实践

**推荐的 Astro 项目结构：**

```
my-site/
├── public/                       # 静态资源
│   ├── favicon.svg
│   ├── robots.txt
│   └── og-image.png
├── src/
│   ├── components/               # 组件
│   │   ├── base/               # 基础组件
│   │   │   ├── Header.astro
│   │   │   ├── Footer.astro
│   │   │   └── SEO.astro
│   │   ├── blog/               # 博客组件
│   │   │   ├── PostCard.astro
│   │   │   ├── TableOfContents.astro
│   │   │   └── ShareButtons.astro
│   │   ├── ui/                 # UI 组件
│   │   │   ├── Button.astro
│   │   │   ├── Card.astro
│   │   │   └── Badge.astro
│   │   └── react/              # React 组件
│   │       ├── Counter.tsx
│   │       └── SearchBox.tsx
│   ├── content/                  # 内容集合
│   │   ├── config.ts           # 集合配置
│   │   ├── blog/               # 博客文章
│   │   │   ├── hello-world.md
│   │   │   └── getting-started.md
│   │   └── docs/               # 文档
│   │       ├── getting-started.md
│   │       └── configuration.md
│   ├── layouts/                  # 布局
│   │   ├── BaseLayout.astro
│   │   ├── BlogPostLayout.astro
│   │   └── DocsLayout.astro
│   ├── pages/                    # 页面
│   │   ├── index.astro         # 首页
│   │   ├── about.astro         # 关于页
│   │   ├── blog/
│   │   │   ├── index.astro     # 博客列表
│   │   │   ├── [slug].astro   # 博客详情
│   │   │   └── [...page].astro # 分页
│   │   ├── docs/
│   │   │   ├── index.astro     # 文档首页
│   │   │   └── [...slug].astro # 文档详情
│   │   └── api/                 # API 路由（SSR）
│   │       └── feedback.ts
│   ├── styles/                  # 样式
│   │   ├── global.css
│   │   └── variables.css
│   ├── lib/                     # 工具库
│   │   ├── utils.ts
│   │   └── constants.ts
│   └── db/                      # Astro DB
│       └── config.ts
├── astro.config.mjs            # Astro 配置
├── tailwind.config.mjs          # Tailwind 配置
├── tsconfig.json                 # TypeScript 配置
└── package.json
```

---

## 认证方案详解

### Auth.js (NextAuth) Astro 集成

Astro 可以通过 API 路由和混合模式实现认证功能：

**安装依赖：**

```bash
npm install astro-auth-credentials
```

**SSR 认证页面：**

```astro
---
// src/pages/login.astro
// 启用 SSR
export const prerender = false;

import { getUser } from '../lib/auth';

// 获取用户
const user = await getUser(Astro.request);

// 已登录则重定向
if (user) {
  return Astro.redirect('/dashboard');
}
---

<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <title>登录</title>
  </head>
  <body>
    <main class="min-h-screen flex items-center justify-center">
      <div class="max-w-md w-full space-y-8 p-8">
        <h1 class="text-3xl font-bold text-center">登录</h1>
        
        <form method="POST" action="/api/login" class="space-y-6">
          <div>
            <label for="email" class="block text-sm font-medium">邮箱</label>
            <input
              type="email"
              id="email"
              name="email"
              required
              class="w-full mt-1 px-3 py-2 border rounded-lg"
            />
          </div>
          
          <div>
            <label for="password" class="block text-sm font-medium">密码</label>
            <input
              type="password"
              id="password"
              name="password"
              required
              class="w-full mt-1 px-3 py-2 border rounded-lg"
            />
          </div>
          
          <button
            type="submit"
            class="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700"
          >
            登录
          </button>
        </form>
        
        <div class="relative">
          <div class="absolute inset-0 flex items-center">
            <div class="w-full border-t"></div>
          </div>
          <div class="relative flex justify-center text-sm">
            <span class="px-2 bg-white text-gray-500">或</span>
          </div>
        </div>
        
        <a
          href="/api/auth/google"
          class="block w-full text-center border py-2 px-4 rounded-lg hover:bg-gray-50"
        >
          使用 Google 登录
        </a>
      </div>
    </main>
  </body>
</html>
```

**登录 API 路由：**

```typescript
// src/pages/api/login.ts
export const prerender = false;

import type { APIRoute } from 'astro';
import { createSession, validateCredentials } from '../../lib/auth';

export const POST: APIRoute = async ({ request, cookies, redirect }) => {
  const formData = await request.formData();
  const email = formData.get('email') as string;
  const password = formData.get('password') as string;

  // 验证凭据
  const user = await validateCredentials(email, password);
  
  if (!user) {
    return new Response(JSON.stringify({ error: '邮箱或密码错误' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  // 创建会话
  const session = await createSession(user.id);
  
  // 设置 Cookie
  cookies.set('session', session.token, {
    path: '/',
    httpOnly: true,
    secure: import.meta.env.PROD,
    sameSite: 'lax',
    maxAge: 60 * 60 * 24 * 7, // 7 天
  });

  return redirect('/dashboard');
};
```

**OAuth 登录：**

```typescript
// src/pages/api/auth/[provider].ts
export const prerender = false;
export const GET: APIRoute = async ({ params, redirect }) => {
  const provider = params.provider;
  
  // Google OAuth
  if (provider === 'google') {
    const clientId = import.meta.env.GOOGLE_CLIENT_ID;
    const redirectUri = `${import.meta.env.SITE_URL}/api/auth/callback/google`;
    
    const authUrl = new URL('https://accounts.google.com/o/oauth2/v2/auth');
    authUrl.searchParams.set('client_id', clientId);
    authUrl.searchParams.set('redirect_uri', redirectUri);
    authUrl.searchParams.set('response_type', 'code');
    authUrl.searchParams.set('scope', 'openid email profile');
    authUrl.searchParams.set('access_type', 'offline');
    
    return redirect(authUrl.toString());
  }
  
  return new Response('Provider not found', { status: 404 });
};
```

### 保护路由中间件

```typescript
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';
import { getUser } from './lib/auth';

export const onRequest = defineMiddleware(async (context, next) => {
  const protectedPaths = ['/dashboard', '/settings', '/profile'];
  const isProtected = protectedPaths.some(p => 
    context.url.pathname.startsWith(p)
  );
  
  if (isProtected) {
    const user = await getUser(context.request);
    
    if (!user) {
      return context.redirect(`/login?redirect=${encodeURIComponent(context.url.pathname)}`);
    }
    
    // 将用户信息添加到 locals
    context.locals.user = user;
  }
  
  return next();
});
```

---

## 完整 CRUD 示例

### 项目结构

本节展示一个使用 Astro DB 的评论系统 CRUD 实现。

**Astro DB Schema：**

```typescript
// src/db/config.ts
import { defineDb, defineTable, column, relations } from 'astro:db';

const users = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    email: column.text({ unique: true }),
    name: column.text(),
    avatar: column.text().optional(),
    createdAt: column.date({ default: new Date() }),
  },
});

const posts = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    title: column.text(),
    slug: column.text({ unique: true }),
    content: column.text(),
    authorId: column.number({ references: () => users.columns.id }),
    published: column.boolean({ default: false }),
    createdAt: column.date({ default: new Date() }),
  },
  relations: {
    author: relation(users, ({ one }) => one({
      fields: [posts.authorId],
      references: [users.id],
    })),
  },
});

const comments = defineTable({
  columns: {
    id: column.number({ primaryKey: true }),
    content: column.text(),
    authorId: column.number({ references: () => users.columns.id }),
    postId: column.number({ references: () => posts.columns.id }),
    parentId: column.number().optional(), // 回复
    createdAt: column.date({ default: new Date() }),
  },
  relations: {
    author: relation(users, ({ one }) => one({
      fields: [comments.authorId],
      references: [users.id],
    })),
    post: relation(posts, ({ one }) => one({
      fields: [comments.postId],
      references: [posts.id],
    })),
    parent: relation(comments, ({ one }) => one({
      fields: [comments.parentId],
      references: [comments.id],
    })),
  },
});

export default defineDb({
  tables: { users, posts, comments },
});
```

### 创建评论（Create）

```astro
---
// src/pages/blog/[slug].astro
import { db, posts, comments, users } from 'astro:db';
import { eq } from 'astro:db/utils';
import { getUser } from '../../lib/auth';

export const prerender = false;

// 获取文章
const { slug } = Astro.params;
const post = await db.select().from(posts).where(eq(posts.slug, slug)).get();

if (!post) {
  return Astro.redirect('/404');
}

// 获取评论
const postComments = await db
  .select({
    id: comments.id,
    content: comments.content,
    createdAt: comments.createdAt,
    parentId: comments.parentId,
    author: {
      id: users.id,
      name: users.name,
      avatar: users.avatar,
    },
  })
  .from(comments)
  .innerJoin(users, eq(comments.authorId, users.id))
  .where(eq(comments.postId, post.id));

// 获取当前用户
const user = await getUser(Astro.request);
---

<article class="blog-post">
  <h1>{post.title}</h1>
  <div class="content">
    {post.content}
  </div>
  
  <!-- 评论区域 -->
  <section class="comments">
    <h2>评论 ({postComments.length})</h2>
    
    <!-- 评论表单 -->
    {user ? (
      <form method="POST" action={`/api/posts/${post.id}/comments`}>
        <textarea
          name="content"
          placeholder="写下你的评论..."
          required
          rows="4"
        ></textarea>
        <button type="submit">发表评论</button>
      </form>
    ) : (
      <p><a href="/login">登录</a> 后发表评论</p>
    )}
    
    <!-- 评论列表 -->
    <div class="comment-list">
      {postComments.map(comment => (
        <div class="comment" data-id={comment.id}>
          <img src={comment.author.avatar || '/default-avatar.png'} alt="" />
          <div>
            <strong>{comment.author.name}</strong>
            <time>{comment.createdAt.toLocaleDateString('zh-CN')}</time>
            <p>{comment.content}</p>
            {user && (
              <button class="reply-btn" data-id={comment.id}>回复</button>
            )}
          </div>
        </div>
      ))}
    </div>
  </section>
</article>
```

```typescript
// src/pages/api/posts/[postId]/comments.ts
export const prerender = false;

import type { APIRoute } from 'astro';
import { db, comments } from '../../../db/config';
import { getUser } from '../../../lib/auth';
import { eq } from 'astro:db/utils';

export const POST: APIRoute = async ({ request, params }) => {
  // 获取用户
  const user = await getUser(request);
  
  if (!user) {
    return new Response(JSON.stringify({ error: '请先登录' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  const formData = await request.formData();
  const content = formData.get('content') as string;
  const parentId = formData.get('parentId') as string | null;
  const postId = parseInt(params.postId!);

  if (!content?.trim()) {
    return new Response(JSON.stringify({ error: '评论内容不能为空' }), {
      status: 400,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  // 创建评论
  const newComment = await db.insert(comments).values({
    content: content.trim(),
    authorId: user.id,
    postId,
    parentId: parentId ? parseInt(parentId) : null,
  }).returning().get();

  return new Response(JSON.stringify(newComment), {
    status: 201,
    headers: { 'Content-Type': 'application/json' },
  });
};
```

### 读取评论（Read）

```typescript
// 获取带嵌套回复的评论
export async function getCommentsWithReplies(postId: number) {
  const allComments = await db
    .select({
      id: comments.id,
      content: comments.content,
      createdAt: comments.createdAt,
      parentId: comments.parentId,
      author: {
        id: users.id,
        name: users.name,
        avatar: users.avatar,
      },
    })
    .from(comments)
    .innerJoin(users, eq(comments.authorId, users.id))
    .where(eq(comments.postId, postId));

  // 构建树形结构
  const commentMap = new Map();
  const rootComments: any[] = [];

  allComments.forEach(comment => {
    commentMap.set(comment.id, { ...comment, replies: [] });
  });

  allComments.forEach(comment => {
    const node = commentMap.get(comment.id);
    if (comment.parentId) {
      const parent = commentMap.get(comment.parentId);
      if (parent) {
        parent.replies.push(node);
      }
    } else {
      rootComments.push(node);
    }
  });

  return rootComments;
}
```

### 更新评论（Update）

```typescript
// src/pages/api/comments/[id].ts
export const prerender = false;

import type { APIRoute } from 'astro';
import { db, comments } from '../../db/config';
import { getUser } from '../../lib/auth';
import { eq } from 'astro:db/utils';

export const PATCH: APIRoute = async ({ request, params }) => {
  const user = await getUser(request);
  
  if (!user) {
    return new Response(JSON.stringify({ error: '请先登录' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  const commentId = parseInt(params.id!);
  const formData = await request.formData();
  const content = formData.get('content') as string;

  // 检查评论所有权
  const existing = await db
    .select()
    .from(comments)
    .where(eq(comments.id, commentId))
    .get();

  if (!existing) {
    return new Response(JSON.stringify({ error: '评论不存在' }), {
      status: 404,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  if (existing.authorId !== user.id) {
    return new Response(JSON.stringify({ error: '无权限修改' }), {
      status: 403,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  // 更新评论
  const updated = await db
    .update(comments)
    .set({ content: content.trim() })
    .where(eq(comments.id, commentId))
    .returning()
    .get();

  return new Response(JSON.stringify(updated), {
    headers: { 'Content-Type': 'application/json' },
  });
};
```

### 删除评论（Delete）

```typescript
export const DELETE: APIRoute = async ({ request, params }) => {
  const user = await getUser(request);
  
  if (!user) {
    return new Response(JSON.stringify({ error: '请先登录' }), {
      status: 401,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  const commentId = parseInt(params.id!);

  // 检查评论所有权
  const existing = await db
    .select()
    .from(comments)
    .where(eq(comments.id, commentId))
    .get();

  if (!existing) {
    return new Response(JSON.stringify({ error: '评论不存在' }), {
      status: 404,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  if (existing.authorId !== user.id) {
    return new Response(JSON.stringify({ error: '无权限删除' }), {
      status: 403,
      headers: { 'Content-Type': 'application/json' },
    });
  }

  // 删除评论（及其回复）
  await db.delete(comments).where(eq(comments.id, commentId));

  return new Response(null, { status: 204 });
};
```

---

## Islands 高级用法

### Islands 通信模式

**使用 Nano Stores 进行状态共享：**

```bash
npm install nanostores @nanostores/preact
```

```typescript
// src/stores/cart.ts
import { atom, computed } from 'nanostores';

// 购物车状态
export const cartItems = atom<Array<{ id: number; name: string; quantity: number }>>([]);
export const isCartOpen = atom(false);

// 派生状态
export const cartTotal = computed(cartItems, items =>
  items.reduce((sum, item) => sum + item.quantity, 0)
);

// 操作
export function addToCart(item: { id: number; name: string }) {
  const current = cartItems.get();
  const existing = current.find(i => i.id === item.id);
  
  if (existing) {
    cartItems.set(
      current.map(i =>
        i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
      )
    );
  } else {
    cartItems.set([...current, { ...item, quantity: 1 }]);
  }
}

export function removeFromCart(id: number) {
  cartItems.set(cartItems.get().filter(i => i.id !== id));
}

export function clearCart() {
  cartItems.set([]);
}
```

**在 React Island 中使用：**

```tsx
// src/components/react/CartIcon.tsx
import { useStore } from '@nanostores/preact';
import { cartTotal, isCartOpen } from '../../stores/cart';

export function CartIcon() {
  const total = useStore(cartTotal);
  const isOpen = useStore(isCartOpen);

  return (
    <button
      className="relative p-2"
      onClick={() => isCartOpen.set(!isOpen)}
    >
      <svg className="w-6 h-6" fill="none" stroke="currentColor">
        <path strokeLinecap="round" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
      </svg>
      {total > 0 && (
        <span className="absolute -top-1 -right-1 bg-red-500 text-white text-xs w-5 h-5 rounded-full flex items-center justify-center">
          {total}
        </span>
      )}
    </button>
  );
}
```

```tsx
// src/components/react/CartDrawer.tsx
import { useStore } from '@nanostores/preact';
import { cartItems, removeFromCart, clearCart, isCartOpen } from '../../stores/cart';

export function CartDrawer() {
  const items = useStore(cartItems);
  const isOpen = useStore(isCartOpen);

  if (!isOpen) return null;

  return (
    <div className="fixed inset-0 z-50">
      <div className="absolute inset-0 bg-black/50" onClick={() => isCartOpen.set(false)} />
      <div className="absolute right-0 top-0 h-full w-full max-w-md bg-white shadow-xl">
        <div className="p-4 border-b flex justify-between items-center">
          <h2 className="text-lg font-semibold">购物车</h2>
          <button onClick={() => isCartOpen.set(false)}>关闭</button>
        </div>
        
        <div className="p-4 space-y-4">
          {items.length === 0 ? (
            <p className="text-center text-gray-500">购物车是空的</p>
          ) : (
            items.map(item => (
              <div key={item.id} className="flex justify-between items-center">
                <span>{item.name}</span>
                <div className="flex items-center gap-2">
                  <span>×{item.quantity}</span>
                  <button
                    className="text-red-500"
                    onClick={() => removeFromCart(item.id)}
                  >
                    删除
                  </button>
                </div>
              </div>
            ))
          )}
        </div>
        
        {items.length > 0 && (
          <div className="absolute bottom-0 left-0 right-0 p-4 border-t">
            <button
              className="w-full bg-red-500 text-white py-2 rounded"
              onClick={clearCart}
            >
              清空购物车
            </button>
          </div>
        )}
      </div>
    </div>
  );
}
```

```astro
---
// 在 Astro 页面中使用 Islands
import { CartIcon } from '../components/react/CartIcon';
import { CartDrawer } from '../components/react/CartDrawer';
---

<html>
  <body>
    <header>
      <CartIcon client:load />
    </header>
    
    <main>
      <!-- 产品列表 -->
    </main>
    
    <CartDrawer client:load />
  </body>
</html>
```

### Islands 懒加载策略

```astro
---
// 根据用户行为决定加载策略
const userHasSeenContent = Astro.cookies.get('seen_content')?.value === 'true';
---

<html>
  <body>
    <!-- 页面加载时立即激活 - 关键交互 -->
    <Header client:load />
    
    <!-- 首屏内容 - 可见时激活 -->
    <Hero client:visible />
    
    <!-- 非关键功能 - 空闲时激活 -->
    <CookieConsent client:idle />
    
    <!-- 屏幕外内容 - 滚动到可见时激活 -->
    <RelatedPosts client:visible />
    
    <!-- 移动端菜单 - 仅在小屏幕上激活 -->
    <MobileMenu client:media="(max-width: 768px)" />
    
    <!-- 评论系统 - 仅当用户滚动到评论区时 -->
    <CommentsSection client:visible={{ rootMargin: "200px" }} />
    
    <!-- 实验性功能 - 仅在开发环境激活 -->
    <DevTools client:only="dev" />
  </body>
</html>
```

---

## Astro 性能优化

### 图片优化

**astro:assets 图片组件：**

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---

<!-- 本地图片 - 自动优化 -->
<Image
  src={heroImage}
  alt="Hero Image"
  width={1200}
  height={600}
  format="webp"
  quality={85}
  loading="eager"  // 或 "lazy"
  decoding="async"
  class="rounded-lg shadow-xl"
/>

<!-- 远程图片 -->
<Image
  src="https://images.unsplash.com/photo-xxx"
  alt="Remote Image"
  width={1200}
  height={600}
  inferSize  // 自动推断尺寸
/>

<!-- 带响应式 srcset -->
<picture>
  <Image
    src={heroImage}
    widths={[400, 800, 1200, 1600]}
    sizes="(max-width: 768px) 100vw, 50vw"
    alt="Hero"
  />
</picture>
```

**Picture 组件：**

```astro
---
import { Picture } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---

<Picture
  src={heroImage}
  alt="Hero"
  formats={['avif', 'webp', 'jpeg']}
  widths={[400, 800, 1200]}
  sizes="(max-width: 768px) 100vw, 50vw"
/>
```

### 字体优化

```astro
---
// src/pages/index.astro
---

<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <!-- 预连接 -->
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    
    <!-- 异步加载字体 -->
    <link
      rel="preload"
      href="/fonts/custom-font.woff2"
      as="font"
      type="font/woff2"
      crossorigin
    />
  </head>
  <body>
    <style>
      @font-face {
        font-family: 'CustomFont';
        src: url('/fonts/custom-font.woff2') format('woff2');
        font-weight: 400;
        font-style: normal;
        font-display: swap;
      }
      
      body {
        font-family: 'CustomFont', system-ui, sans-serif;
      }
    </style>
  </body>
</html>
```

### 资源预加载

```astro
---
// src/pages/index.astro

// 预加载关键资源
const criticalScripts = ['/js/critical.js'];
const criticalStyles = ['/css/critical.css'];
---

<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <!-- 预加载关键 CSS -->
    {criticalStyles.map(href => (
      <link rel="preload" href={href} as="style" />
    ))}
    
    <!-- 预加载关键 JS -->
    {criticalScripts.map(src => (
      <link rel="preload" href={src} as="script" />
    ))}
    
    <!-- DNS 预解析 -->
    <link rel="dns-prefetch" href="//fonts.googleapis.com" />
    <link rel="dns-prefetch" href="//www.google-analytics.com" />
  </head>
</html>
```

---

## 部署与托管

### Vercel 部署

**vercel.json 配置：**

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "astro",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "X-Frame-Options",
          "value": "DENY"
        },
        {
          "key": "X-XSS-Protection",
          "value": "1; mode=block"
        }
      ]
    },
    {
      "source": "/assets/(.*)",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/api/(.*)",
      "destination": "/api/$1"
    }
  ]
}
```

### Netlify 部署

**netlify.toml 配置：**

```toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"

[[redirects]]
  from = "/*"
  to = "/404"
  status = 404

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
```

### Cloudflare Pages 部署

**使用 Wrangler 部署：**

```bash
npm install -D wrangler

# 构建
npm run build

# 部署
wrangler pages deploy dist --project-name=my-site
```

### Docker 部署

**Dockerfile：**

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --omit=dev

EXPOSE 4321

CMD ["node", "dist/server/entry.mjs"]
```

---

## 常见陷阱与解决方案

### Islands 相关问题

**1. Islands 之间状态不同步**

**问题：** 多个 Islands 使用不同的状态实例。

```astro
<!-- ❌ 问题：每次实例化都是新的状态 -->
<Counter client:load />
<Counter client:load />
```

**解决方案：** 使用全局状态（Nano Stores）。

```astro
<!-- ✅ 使用共享状态 -->
<Counter client:load storeKey="counter1" />
<Counter client:load storeKey="counter1" />
```

**2. Islands 指令选择不当**

**问题：** 使用错误的激活指令导致性能问题。

```astro
<!-- ❌ 问题：大组件使用 client:load -->
<HeavyDashboard client:load /> <!-- 阻塞首屏 -->
```

**解决方案：** 根据组件重要性和大小选择合适的指令。

```astro
<!-- ✅ 首屏关键 -->
<Header client:load />

<!-- ✅ 非关键大组件 -->
<HeavyDashboard client:visible />

<!-- ✅ 工具类 -->
<ThemeToggle client:idle />
```

### Content Collections 问题

**1. Schema 验证失败**

**问题：** 内容文件不匹配 schema 定义。

```typescript
// ❌ 缺少验证逻辑
const blog = defineCollection({
  schema: z.object({
    title: z.string(),
  }),
});
```

**解决方案：** 添加严格的 schema 和错误处理。

```typescript
// ✅ 严格 schema
const blog = defineCollection({
  schema: z.object({
    title: z.string().min(1).max(200),
    description: z.string().max(500),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    author: z.string(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false),
  }),
});
```

### SSR 相关问题

**1. API 路由 CORS 问题**

**问题：** SSR 模式下 API 路由跨域请求失败。

**解决方案：** 配置 CORS 头。

```typescript
// src/pages/api/data.ts
export const prerender = false;

export const GET: APIRoute = async ({ request }) => {
  const corsHeaders = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, Authorization',
  };

  // 处理 preflight
  if (request.method === 'OPTIONS') {
    return new Response(null, {
      status: 204,
      headers: corsHeaders,
    });
  }

  return new Response(JSON.stringify({ data: 'test' }), {
    status: 200,
    headers: {
      'Content-Type': 'application/json',
      ...corsHeaders,
    },
  });
};
```

---

## Astro 生态工具

### 官方集成

| 集成 | 功能 | 安装命令 |
|------|------|---------|
| **@astrojs/react** | React 支持 | `npx astro add react` |
| **@astrojs/vue** | Vue 支持 | `npx astro add vue` |
| **@astrojs/svelte** | Svelte 支持 | `npx astro add svelte` |
| **@astrojs/solid-js** | Solid.js 支持 | `npx astro add solid` |
| **@astrojs/tailwind** | Tailwind CSS | `npx astro add tailwind` |
| **@astrojs/mdx** | MDX 支持 | `npx astro add mdx` |
| **@astrojs/sitemap** | Sitemap 生成 | `npx astro add sitemap` |
| **@astrojs/image** | 图片优化 | `npx astro add image` |
| **@astrojs/partytown** | 分析工具 | `npx astro add partytown` |
| **@astrojs/lit** | Lit 支持 | `npx astro add lit` |
| **@astrojs/preact** | Preact 支持 | `npx astro add preact` |

### 社区工具

| 工具 | 功能 | 链接 |
|------|------|------|
| **Astro SEO** | SEO 组件 | `@astrolib/seo` |
| **Astro-i18n** | 国际化 | `@astroclick/i18n` |
| **Astro-cn** | 中文优化 | `@astro-community/astro-i18n` |
| **Astro-Icon** | 图标系统 | `astro-icon` |
| **Astro-expressive-code** | 代码高亮 | `astro-expressive-code` |
| **Starlight** | 文档框架 | `@astrojs/starlight` |

### Starlight 文档框架

Starlight 是 Astro 官方出品的文档框架，提供开箱即用的文档站点：

```bash
npm create astro@latest -- --template starlight

# 或添加到现有项目
npx astro add starlight
```

```typescript
// astro.config.mjs
import starlight from '@astrojs/starlight';

export default defineConfig({
  integrations: [
    starlight({
      title: '我的文档',
      description: '项目文档',
      logo: {
        src: './src/assets/logo.png',
      },
      social: {
        github: 'https://github.com/user/repo',
      },
      sidebar: [
        {
          label: '开始',
          items: [
            { label: '介绍', link: '/guides/intro' },
            { label: '安装', link: '/guides/installation' },
          ],
        },
        {
          label: '参考',
          autogenerate: { directory: 'reference' },
        },
      ],
      editLink: {
        baseUrl: 'https://github.com/user/repo/edit/main/',
      },
      lastUpdated: true,
      pagination: true,
    }),
  ],
});
```

---

## 附录：资源链接

### 官方资源

- [Astro 官方文档](https://docs.astro.build)
- [Astro GitHub](https://github.com/withastro/astro)
- [Astro Discord](https://discord.gg/astro)
- [Astro Blog](https://astro.build/blog)

### 学习资源

- [Astro 教程](https://docs.astro.build/zh-cn/tutorial/0-introduction/)
- [Astro 示例](https://astro.build/themes)
- [Astro 集成](https://astro.build/integrations)
- [Astro Showcase](https://astro.build/showcase)

### 相关工具

- [Astro DB](https://astro.build/db)
- [Starlight](https://starlight.astro.build)
- [Astro Studio](https://studio.astro.build)
- [ViewTransitions](https://docs.astro.build/en/guides/view-transitions/)
