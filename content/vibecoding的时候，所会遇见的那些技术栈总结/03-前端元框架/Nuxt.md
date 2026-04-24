# Nuxt 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Nuxt 4 最新特性、文件路由系统、自动导入机制、Server Routes 及与 Next.js 的深度对比。

---

## 目录

1. [[#Nuxt 概述与版本演进]]
2. [[#Nuxt2 vs Nuxt3 vs Nuxt4 对比]]
3. [[#文件路由系统]]
4. [[#自动导入机制]]
5. [[#数据获取]]
6. [[#Server Routes]]
7. [[#状态管理]]
8. [[#Nuxt DevTools]]
9. [[#模块系统]]
10. [[#Nuxt 与 Next.js 对比]]
11. [[#实战场景与选型建议]]

---

## Nuxt 概述与版本演进

### Nuxt 在 Vue 生态中的定位

Nuxt 是 Vue.js 的全栈元框架，提供了服务端渲染（SSR）、静态站点生成（SSG）、客户端渲染（SPA）等多种渲染模式。作为 Vue 生态中最成熟的框架，Nuxt 在以下场景中表现出色：

- **Vue 项目 SSR 首选**：需要 SEO 优化的 Vue 应用
- **内容驱动型网站**：博客、文档、营销页面
- **全栈 Vue 应用**：需要 API 能力的现代 Web 应用
- **与 Vue 生态深度集成**：Pinia、Vue Router、Vite 的无缝配合

### 版本演进时间线

| 版本 | 发布时间 | 核心特性 |
|------|---------|---------|
| **Nuxt 2** | 2018年7月 | Vue 2 + Webpack、SSR/SSG 支持 |
| **Nuxt 3** | 2022年11月 | Vue 3 + Vite、Nitro 引擎、Composition API |
| **Nuxt 3.12** | 2024年5月 | 稳定版 App.vue、改进的 SSR 性能 |
| **Nuxt 4** | 2025年Q2 | 组件自动导入增强、Layouts 重构、CIN（Composable Islands） |

> [!IMPORTANT]
> Nuxt 4 引入了一些破坏性变更，包括目录结构调整和组件自动导入规则变化。迁移前请查阅官方迁移指南。

---

## Nuxt2 vs Nuxt3 vs Nuxt4 对比

### 核心架构对比

| 维度 | Nuxt 2 | Nuxt 3 | Nuxt 4 |
|------|--------|--------|--------|
| **Vue 版本** | Vue 2 | Vue 3 | Vue 3 + Vue 4 预览 |
| **构建工具** | Webpack | Vite | Vite 5 |
| **渲染引擎** | Vue Server Renderer | Nitro | Nitro 2.0 |
| **类型支持** | TypeScript 有限 | 原生 TypeScript | 增强的 TypeScript |
| **组件导入** | 手动导入 | 自动导入 | 增强自动导入 |
| **布局系统** | layouts/ 目录 | layouts/ + NuxtLayout | 继承 + 插槽系统 |
| **状态管理** | Vuex | Pinia | Pinia + 新的响应式 API |
| **模块生态** | @nuxt/* | Nuxt Modules | 兼容 + 新模块 |

### 路由对比

| 特性 | Nuxt 2 | Nuxt 3/4 |
|------|--------|----------|
| **文件路由** | `pages/` 目录 | `pages/` 目录（增强） |
| **嵌套路由** | `<nuxt-child>` | `<NuxtPage>` |
| **动态路由** | `[id].vue` | `[id].vue` 或 `[id]/index.vue` |
| **路由参数** | `this.$route.params.id` | `useRoute().params.id` |
| **中间件** | middleware/ | middleware/（支持组合式） |
| **布局切换** | `this.$nuxt.setLayout()` | `<NuxtLayout>` + `definePageMeta` |

### 性能对比

| 指标 | Nuxt 2 | Nuxt 3 | Nuxt 4 |
|------|--------|--------|--------|
| **冷启动速度** | 3-5s | 0.5-1s | 0.3-0.5s |
| **热更新速度** | 1-2s | 100-200ms | 50-100ms |
| **产出体积** | 较大 | 较小 | 最小 |
| **内存占用** | 高 | 中等 | 低 |

> [!TIP]
> Nuxt 3 的性能提升主要来自 Vite，而 Nuxt 4 进一步优化了 Nitro 引擎的冷启动和内存占用。

---

## 文件路由系统

### 基础路由

Nuxt 遵循「约定优于配置」原则，`pages/` 目录下的文件自动映射为路由：

```
pages/
├── index.vue          → /
├── about.vue          → /about
├── blog/
│   ├── index.vue      → /blog
│   ├── [slug].vue     → /blog/:slug
│   └── [year]/
│       └── [month].vue → /blog/:year/:month
```

### 动态路由

```vue
<!-- pages/blog/[slug].vue -->
<template>
  <div>
    <h1>{{ slug }}</h1>
    <p>当前文章: {{ route.params.slug }}</p>
  </div>
</template>

<script setup lang="ts">
const route = useRoute();

// 路由参数自动推断
const slug = computed(() => route.params.slug as string);
</script>
```

### 嵌套路由

```
pages/
├── user/
│   ├── index.vue        ← 父路由出口
│   ├── profile.vue      → /user/profile
│   └── settings.vue     → /user/settings
```

```vue
<!-- pages/user/index.vue -->
<template>
  <div class="user-layout">
    <nav>
      <NuxtLink to="/user/profile">个人资料</NuxtLink>
      <NuxtLink to="/user/settings">设置</NuxtLink>
    </nav>
    <!-- 子路由出口 -->
    <NuxtPage />
  </div>
</template>
```

### 路由元信息

```vue
<script setup lang="ts">
definePageMeta({
  title: '用户设置',
  middleware: ['auth'],
  layout: 'custom',
  alias: '/account/settings',
});
</script>
```

---

## 自动导入机制

### 组件自动导入

Nuxt 自动导入 `components/` 目录下的所有组件，无需手动 import：

```
components/
├── base/
│   ├── Button.vue       → <BaseButton />
│   └── Card.vue         → <BaseCard />
├── icons/
│   └── GitHub.vue       → <IconsGithub />
└── Alert.vue            → <Alert />
```

```vue
<!-- pages/index.vue -->
<template>
  <!-- 自动可用，无需 import -->
  <BaseButton type="primary" @click="handleClick">
    提交
  </BaseButton>
  <BaseCard>
    <p>卡片内容</p>
  </BaseCard>
</template>

<script setup lang="ts">
// useRoute、useFetch 等自动可用
const route = useRoute();
</script>
```

### 组合式函数自动导入

`composables/` 目录下的函数自动注册为可用的组合式函数：

```typescript
// composables/useCounter.ts
export const useCounter = (initial = 0) => {
  const count = ref(initial);
  const increment = () => count.value++;
  const decrement = () => count.value--;
  
  return { count, increment, decrement };
};
```

```vue
<!-- pages/about.vue -->
<script setup lang="ts">
// useCounter 自动可用
const { count, increment } = useCounter();
</script>
```

### 可组合函数实战

#### useFetch 进阶用法

```typescript
// composables/usePosts.ts
export const usePosts = (options?: {
  limit?: number;
  category?: Ref<string> | string;
}) => {
  const limit = options?.limit ?? 10;
  const category = computed(() => 
    typeof options?.category === 'string' 
      ? options.category 
      : options?.category?.value
  );

  const { data, pending, error, refresh } = useFetch('/api/posts', {
    query: { limit, category },
    transform: (data) => data.posts,
    watch: [category],
  });

  return { posts: data, pending, error, refresh };
};
```

#### useAuth 认证组合

```typescript
// composables/useAuth.ts
interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
}

export const useAuth = () => {
  const user = useState<User | null>('user', () => null);
  const isAuthenticated = computed(() => !!user.value);

  const login = async (email: string, password: string) => {
    const { data } = await useFetch('/api/auth/login', {
      method: 'POST',
      body: { email, password },
    });
    if (data.value) {
      user.value = data.value.user;
    }
  };

  const logout = async () => {
    await useFetch('/api/auth/logout', { method: 'POST' });
    user.value = null;
    await navigateTo('/login');
  };

  return { user, isAuthenticated, login, logout };
};
```

---

## 数据获取

### useFetch 详解

`useFetch` 是 Nuxt 3+ 最核心的数据获取函数，内置 SSR 支持、缓存和去重：

```vue
<script setup lang="ts">
// 基础用法
const { data, pending, error } = await useFetch('/api/users', {
  // 懒加载（客户端渲染）
  lazy: true,
  // 服务器端获取数据
  server: true,
  // 请求参数
  params: { limit: 10 },
  // 数据转换
  transform: (res) => res.data,
  // 缓存 key
  key: 'users',
  // 重新获取的时机
  getCachedData(key, nuxtApp) {
    return nuxtApp.payload.data[key] ?? nuxtApp.static.data[key];
  },
});
</script>
```

### useAsyncData

`useAsyncData` 提供更灵活的数据获取方式，适用于非 HTTP 场景：

```vue
<script setup lang="ts">
// 直接调用数据库或外部 API
const { data, pending, error } = await useAsyncData(
  'post', 
  async () => {
    const post = await db.post.findUnique({
      where: { slug: useRoute().params.slug }
    });
    return post;
  },
  {
    // 服务器端执行
    server: true,
    // 懒加载模式
    lazy: false,
  }
);
</script>
```

### 数据获取模式对比

| 场景 | 推荐方法 | 说明 |
|------|---------|------|
| **HTTP API 调用** | `useFetch` | 自动处理请求、去重、缓存 |
| **直接数据库访问** | `useAsyncData` | 更灵活的控制 |
| **简单计算** | `computed` | 无需异步 |
| **依赖其他数据** | `watchEffect` | 响应式依赖 |
| **表单提交** | `useSubmit` (Nuxt 4) | 简化提交流程 |

### 轮询与实时更新

```vue
<script setup lang="ts">
const { data: notifications } = useFetch('/api/notifications', {
  // 每 30 秒刷新一次
  refreshInterval: 30000,
  // 响应式刷新
  watch: [activeTab],
});
</script>
```

---

## Server Routes

### 目录结构

```
server/
├── api/
│   ├── users/
│   │   ├── index.get.ts    → GET /api/users
│   │   ├── index.post.ts   → POST /api/users
│   │   └── [id].get.ts     → GET /api/users/:id
│   └── posts/
│       └── index.ts        → 支持多种方法
├── middleware/
│   └── auth.ts             → 服务器中间件
└── utils/
    └── db.ts               → 数据库客户端
```

### API 处理器

```typescript
// server/api/users/index.get.ts
import { db } from '~/server/utils/db';

export default defineEventHandler(async (event) => {
  const query = getQuery(event);
  const { limit = 10, offset = 0 } = query;

  const users = await db.user.findMany({
    take: Number(limit),
    skip: Number(offset),
    select: {
      id: true,
      name: true,
      email: true,
      createdAt: true,
    },
  });

  return { users };
});
```

```typescript
// server/api/users/index.post.ts
import { db } from '~/server/utils/db';
import { z } from 'zod';

const UserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(6),
});

export default defineEventHandler(async (event) => {
  const body = await readBody(event);
  const validated = UserSchema.parse(body);

  const user = await db.user.create({
    data: validated,
    select: { id: true, name: true, email: true },
  });

  setResponseStatus(event, 201);
  return { user };
});
```

### 路由参数处理

```typescript
// server/api/posts/[slug].get.ts
import { db } from '~/server/utils/db';

export default defineEventHandler(async (event) => {
  const slug = getRouterParam(event, 'slug');
  
  const post = await db.post.findUnique({
    where: { slug },
    include: { author: true, tags: true },
  });

  if (!post) {
    throw createError({
      statusCode: 404,
      statusMessage: 'Post not found',
    });
  }

  return post;
});
```

### 中间件

```typescript
// server/middleware/auth.ts
export default defineEventHandler(async (event) => {
  const url = getRequestURL(event);
  
  // 跳过认证的路径
  const publicPaths = ['/api/auth/login', '/api/auth/register'];
  if (publicPaths.some(p => url.pathname.startsWith(p))) {
    return;
  }

  const token = getHeader(event, 'Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Unauthorized',
    });
  }

  try {
    const decoded = await verifyToken(token);
    event.context.user = decoded;
  } catch {
    throw createError({
      statusCode: 401,
      statusMessage: 'Invalid token',
    });
  }
});
```

---

## 状态管理

### useState

`useState` 是 Nuxt 推荐的轻量级状态管理方案，基于 Vue 的 `ref`：

```vue
<script setup lang="ts">
// 在组件中使用
const count = useState('count', () => 0);

// 跨组件共享（在 setup 顶层调用）
const user = useState<User>('user');

// SSR 友好，自动序列化
</script>
```

### Pinia 集成

```bash
# 安装
npx nuxi@latest module add pinia
```

```typescript
// stores/user.ts
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
  state: () => ({
    name: '',
    email: '',
    preferences: {},
  }),
  
  getters: {
    isLoggedIn: (state) => !!state.email,
  },
  
  actions: {
    async fetchUser() {
      const { data } = await useFetch('/api/user/me');
      if (data.value) {
        this.$patch(data.value);
      }
    },
  },
});
```

```vue
<script setup lang="ts">
const userStore = useUserStore();

// 组件中直接使用
console.log(userStore.isLoggedIn);
</script>
```

---

## Nuxt DevTools

### 功能概览

Nuxt DevTools 是 Nuxt 3+ 的官方开发工具，提供可视化调试体验：

| 功能 | 说明 |
|------|------|
| **组件检查** | 可视化查看组件树和 props |
| **页面检查** | 查看路由配置和元信息 |
| **模块管理** | 查看已安装模块 |
| **性能分析** | 分析 SSR 性能和 bundle |
| **Pinia 检查** | 可视化状态管理 |
| **Rocket** | AI 辅助功能（Beta） |

### 启用配置

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  devtools: { 
    enabled: true,
    // 启用特定功能
    timeline: {
      enabled: true,
    },
  },
});
```

---

## 模块系统

### 常用模块列表

| 模块 | 用途 | 安装命令 |
|------|------|---------|
| **@nuxt/image** | 图片优化 | `npx nuxi module add image` |
| **@nuxt/content** | 内容管理/Markdown | `npx nuxi module add content` |
| **@pinia/nuxt** | Pinia 状态管理 | `npx nuxi module add pinia` |
| **@nuxtjs/tailwindcss** | Tailwind CSS | `npx nuxi module add tailwindcss` |
| **@nuxtjs/google-fonts** | Google 字体 | `npx nuxi module add google-fonts` |
| **nuxt-icon** | 图标组件 | `npx nuxi module add icon` |
| **@nuxtseo/module** | SEO 优化 | `npx nuxi module add seo` |
| **@nuxtjs/i18n** | 国际化 | `npx nuxi module add i18n` |

### @nuxt/image 实战

```vue
<template>
  <!-- 自动优化、懒加载、WebP 转换 -->
  <NuxtImg
    src="/images/hero.jpg"
    width="800"
    height="600"
    format="webp"
    quality="85"
    loading="lazy"
    placeholder
  />
</template>
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image'],
  image: {
    // 图像 CDN 配置
    domains: ['images.unsplash.com'],
    quality: 85,
    format: ['webp', 'avif'],
  },
});
```

### @nuxt/content 实战

```bash
npx nuxi module add content
```

```
content/
├── blog/
│   ├── getting-started.md
│   └── advanced-features.md
└── docs/
    └── api-reference.md
```

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
const route = useRoute();
const { data: doc } = await useAsyncData(
  `doc-${route.path}`,
  () => queryContent(route.path).findOne()
);
</script>

<template>
  <article v-if="doc">
    <h1>{{ doc.title }}</h1>
    <ContentRenderer :value="doc" />
  </article>
</template>
```

---

## Nuxt 与 Next.js 对比

### 核心定位对比

| 维度 | Nuxt | Next.js |
|------|------|---------|
| **底层框架** | Vue 3 | React |
| **学习曲线** | 平缓（Vue 简洁） | 中等（React 生态） |
| **类型安全** | 优秀（TypeScript 原生） | 优秀（TypeScript 原生） |
| **SSR 体验** | 优秀 | 优秀 |
| **静态生成** | 优秀 | 优秀 |
| **API 能力** | Server Routes（H3） | Route Handlers |
| **部署** | 任意 Node.js / Vercel | 任意 / Vercel 最佳 |
| **生态** | Vue 生态 | React 生态 |

### 技术架构对比

| 特性 | Nuxt | Next.js |
|------|------|---------|
| **路由系统** | 文件路由 + 约定 | 文件路由 + 约定 |
| **数据获取** | useFetch / useAsyncData | Server Components / useSWR |
| **状态管理** | useState / Pinia | Context / Zustand / Jotai |
| **样式方案** | 任意（推荐 UnoCSS） | 任意（推荐 Tailwind） |
| **组件导入** | 自动导入 | 手动导入 |
| **Server Actions** | 表单处理 / Server Function | 原生 Server Actions |

### 渲染模式对比

| 模式 | Nuxt | Next.js |
|------|------|---------|
| **SSR** | `ssr: true` | `export const dynamic = 'force-dynamic'` |
| **SSG** | `nuxt generate` | `export const dynamic = 'force-static'` |
| **ISR** | `routeRules` | `revalidate` |
| **SPA** | `ssr: false` | `output: 'export'` |
| **混合模式** | `routeRules` 精细控制 | 部分预渲染（PPR） |

```typescript
// Nuxt 路由规则配置
export default defineNuxtConfig({
  routeRules: {
    '/': { prerender: true },                    // 首页静态生成
    '/blog/**': { swr: 3600 },                   // ISR 1小时
    '/dashboard/**': { ssr: false },              // 客户端渲染
    '/api/**': { cors: true },                    // API 路由配置
  },
});
```

### 选型建议

> [!TIP]
> 选择建议：
> - **熟悉 Vue** → Nuxt 3/4
> - **熟悉 React** → Next.js
> - **需要快速上手** → Nuxt（自动导入更省心）
> - **需要极致性能** → 两者均可，视团队而定
> - **内容网站** → Nuxt Content 或 Astro

---

## 实战场景与选型建议

### Nuxt Content 博客实战

```markdown
---
title: 我的第一篇文章
description: 这是介绍
date: 2026-04-19
tags: [Nuxt, Vue, 前端]
author: 归愚
---

# 正文内容

使用 Markdown 语法...
```

```vue
<!-- pages/blog/[...slug].vue -->
<script setup lang="ts">
const route = useRoute();

const { data: article } = await useAsyncData(
  `article-${route.path}`,
  () => queryContent(route.path).findOne()
);

useSeoMeta({
  title: () => article.value?.title ?? '文章',
  description: () => article.value?.description,
});
</script>

<template>
  <div v-if="article">
    <h1>{{ article.title }}</h1>
    <ContentRenderer :value="article" />
  </div>
</template>
```

### AI 应用集成实战

```typescript
// server/api/chat.post.ts
import { z } from 'zod';
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export default defineEventHandler(async (event) => {
  const body = await readBody(event);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: body.messages,
    stream: true,
  });

  return new Response(completion.toReadableStream(), {
    headers: {
      'Content-Type': 'text/event-stream',
    },
  });
});
```

### 成本估算

| 方案 | 月成本（100万 PV） | 说明 |
|------|-------------------|------|
| **Vercel** | $20 + 超额 | Serverless |
| **Railway** | $5-50 | 按使用计费 |
| **自托管 VPS** | $10-50 | 完全可控 |

---

> [!SUCCESS]
> Nuxt 是 Vue 生态中最成熟的元框架，Nuxt 3/4 在性能、开发体验和模块生态上都有显著提升。通过自动导入机制、强大的 Server Routes 和完善的模块系统，Nuxt 能够高效构建从简单博客到复杂全栈应用的各种项目。对于 Vue 团队，Nuxt 是 SSR 和全栈开发的首选框架。

---

## 完整安装指南

### 环境要求与前置准备

**Node.js 版本要求：**
- Nuxt 3: Node.js 18.0 或更高版本
- Nuxt 4: Node.js 20.0 或更高版本
- 推荐使用 Node.js 20 LTS 或 22 LTS

```bash
# 使用 fnm 安装 Node.js 20
curl -fsSL https://fnm.vercel.app/install | bash
fnm install 20
fnm use 20

# 验证安装
node --version  # 应显示 v20.x.x
npm --version   # 应显示 10.x.x
```

### 项目创建详细流程

**方式一：使用 nuxi init（推荐）**

```bash
# 交互式创建
npx nuxi@latest init my-app

# 非交互式创建
npx nuxi@latest init my-app --no-install --packageManager npm

# 使用特定模板
npx nuxi@latest init my-app --template v3
npx nuxi@latest init my-app --template content
```

**nuxi 命令行选项：**

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--template` | 使用特定模板 | 空 |
| `--packageManager` | 包管理器 | npm |
| `--no-install` | 跳过 npm install | false |
| `--force` | 强制覆盖现有目录 | false |
| `--gitInit` | 初始化 Git | true |

**方式二：手动创建 Nuxt 项目**

```bash
# 创建项目目录
mkdir my-app && cd my-app

# 初始化 npm 项目
npm init -y

# 安装 Nuxt 核心依赖
npm install nuxt@latest vue@latest vue-router@latest

# 安装开发依赖
npm install -D typescript @nuxt/typescript-build

# 创建基础文件
touch nuxt.config.ts
mkdir -p pages components layouts
```

**package.json 配置：**

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "generate": "nuxt generate",
    "preview": "nuxt preview",
    "postinstall": "nuxt prepare"
  },
  "dependencies": {
    "nuxt": "^3.15.0",
    "vue": "^3.5.0"
  },
  "devDependencies": {
    "@nuxt/typescript-build": "^3.0.0",
    "typescript": "^5.6.0"
  }
}
```

### Nuxt 配置详解

**nuxt.config.ts 完整配置：**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // === 应用配置 ===
  ssr: true,                    // 启用 SSR（默认 true）
  devtools: { enabled: true }, // 启用 Nuxt DevTools

  // === 源码目录 ===
  srcDir: 'src/',              // 源码目录（默认 'src/'）

  // === 页面目录 ===
  pages: true,                  // 启用基于文件的路由

  // === 全局组件 ===
  components: [
    {
      path: '~/components',    // 组件目录
      pathPrefix: false,       // 不使用路径前缀
    },
    '~/components',            // 自动导入
  ],

  // === 自动导入 ===
  imports: {
    dirs: ['composables/**', 'utils/**'],
    presets: [
      ['vue', ['ref', 'computed', 'watch', ...]],
    ],
  },

  // === 样式 ===
  css: ['~/assets/css/main.css'],

  // === 构建配置 ===
  build: {
    transpile: ['vue', 'vue-router'],
    analyze: true,
  },

  // === Vite 配置 ===
  vite: {
    optimizeDeps: {
      include: ['vue', 'vue-router', 'pinia'],
    },
    server: {
      port: 3000,
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true,
        },
      },
    },
  },

  // === 模块 ===
  modules: [
    '@pinia/nuxt',                 // 状态管理
    '@nuxtjs/tailwindcss',        // Tailwind CSS
    '@nuxt/image',                // 图片优化
    '@nuxt/content',              // 内容管理
    '@nuxtjs/google-fonts',      // Google 字体
    '@nuxtseo/module',            // SEO 优化
  ],

  // === 运行时配置 ===
  runtimeConfig: {
    // 服务端私有的配置
    apiSecret: process.env.NUXT_API_SECRET || '',
    // 客户端可访问的公共配置
    public: {
      apiBase: process.env.NUXT_PUBLIC_API_BASE || '/api',
      siteUrl: process.env.NUXT_PUBLIC_SITE_URL || 'https://example.com',
    },
  },

  // === 应用配置 ===
  app: {
    head: {
      title: '我的 Nuxt 应用',
      meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width, initial-scale=1' },
        { name: 'description', content: '应用描述' },
      ],
      link: [
        { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' },
      ],
    },
    pageTransition: { name: 'page', mode: 'out-in' },
    layoutTransition: { name: 'layout', mode: 'out-in' },
  },

  // === 路由规则 ===
  routeRules: {
    '/': { prerender: true },                    // 首页静态生成
    '/blog/**': { swr: 3600 },                  // ISR 1小时
    '/dashboard/**': { ssr: false },             // SPA 模式
    '/api/**': { cors: true },                  // CORS 配置
    '/admin/**': { headers: { 'cache-control': 'no-store' } },
  },

  // === Nitro 配置 ===
  nitro: {
    preset: 'node-server',
    prerender: {
      routes: ['/sitemap.xml', '/robots.txt'],
      crawlLinks: true,
    },
    compressPublicAssets: true,
    routeRules: {
      '/api/**': { cache: false },
    },
  },

  // === 实验性功能 ===
  experimental: {
    viewTransition: true,
    payloadExtraction: true,
    renderJsonPayloads: true,
  },

  // === 类型配置 ===
  typescript: {
    strict: true,
    typeCheck: true,
  },

  // === 兼容配置 ===
  compatibilityDate: '2024-11-01',
});
```

### 目录结构最佳实践

**推荐的 Nuxt 3/4 项目结构：**

```
my-app/
├── src/                         # 源码目录（可选）
│   ├── app.vue                  # 应用入口
│   ├── pages/                   # 页面（启用 pages 时）
│   │   ├── index.vue            # 首页 /
│   │   ├── about.vue            # 关于页 /about
│   │   ├── blog/
│   │   │   ├── index.vue        # 博客列表 /blog
│   │   │   ├── [slug].vue       # 博客详情 /blog/:slug
│   │   │   └── [year]/
│   │   │       └── [month].vue  # /blog/:year/:month
│   │   └── (auth)/
│   │       ├── login.vue        # /login（使用 auth 布局）
│   │       └── register.vue     # /register
│   ├── layouts/                  # 布局
│   │   ├── default.vue          # 默认布局
│   │   ├── auth.vue             # 认证布局
│   │   └── dashboard.vue        # 仪表板布局
│   ├── components/               # 组件
│   │   ├── base/               # 基础组件
│   │   │   ├── Button.vue      # → <BaseButton />
│   │   │   └── Card.vue        # → <BaseCard />
│   │   ├── icons/              # 图标组件
│   │   │   └── Sun.vue         # → <IconsSun />
│   │   └── AppHeader.vue       # → <AppHeader />
│   ├── composables/              # 组合式函数
│   │   ├── useAuth.ts          # → useAuth()
│   │   ├── useFetch.ts         # → useFetch()
│   │   └── useTheme.ts         # → useTheme()
│   ├── utils/                   # 工具函数
│   │   ├── format.ts           # → formatDate()
│   │   └── validation.ts        # → validateEmail()
│   ├── middleware/               # 中间件
│   │   ├── auth.ts            # 认证中间件
│   │   └── analytics.ts        # 分析中间件
│   ├── plugins/                 # 插件
│   │   ├── axios.ts           # Axios 插件
│   │   └── analytics.client.ts # 客户端插件
│   ├── server/                  # 服务端
│   │   ├── api/              # API 路由
│   │   │   ├── users/
│   │   │   │   ├── index.get.ts
│   │   │   │   ├── index.post.ts
│   │   │   │   └── [id].ts
│   │   │   └── posts/
│   │   ├── middleware/         # 服务端中间件
│   │   ├── utils/             # 服务端工具
│   │   └── plugins/           # 服务端插件
│   ├── assets/                  # 资源
│   │   ├── css/
│   │   │   └── main.css
│   │   └── images/
│   └── public/                  # 静态文件
│       ├── favicon.ico
│       └── robots.txt
├── modules/                     # 本地模块
├── server/                      # 服务端（不在 src 内）
├── app.vue                      # 应用入口（不在 src 内时）
├── nuxt.config.ts              # Nuxt 配置
├── tailwind.config.ts          # Tailwind 配置
├── tsconfig.json               # TypeScript 配置
└── package.json
```

---

## 认证方案详解

### Nuxt Auth Utils

Nuxt Auth Utils 是 Nuxt 官方推荐的认证模块，提供 OAuth、凭据登录、Session 管理等功能。

**安装配置：**

```bash
npx nuxi module add auth-utils
```

**nuxt.config.ts 配置：**

```typescript
export default defineNuxtConfig({
  modules: ['@nuxtjs/auth-utils'],
  runtimeConfig: {
    authSecret: process.env.AUTH_SECRET || 'your-secret-key',
  },
  app: {
    // OAuth 回调处理
    oauth: {
      callbackUrl: '/auth/callback',
      // 回调后跳转
      callbackRoute: '/auth/callback',
    },
  },
});
```

**OAuth 登录：**

```vue
<!-- pages/auth/login.vue -->
<template>
  <div class="min-h-screen flex items-center justify-center">
    <div class="space-y-4">
      <h1 class="text-2xl font-bold">登录</h1>
      
      <button
        @click="loginWithGithub"
        class="w-full px-4 py-2 bg-gray-900 text-white rounded hover:bg-gray-800"
      >
        使用 GitHub 登录
      </button>
      
      <div class="relative">
        <div class="absolute inset-0 flex items-center">
          <span class="w-full border-t" />
        </div>
        <div class="relative flex justify-center text-sm">
          <span class="px-2 bg-white text-gray-500">或</span>
        </div>
      </div>
      
      <form @submit.prevent="loginWithCredentials">
        <div class="space-y-2">
          <input
            v-model="email"
            type="email"
            placeholder="邮箱"
            class="w-full px-3 py-2 border rounded"
          />
          <input
            v-model="password"
            type="password"
            placeholder="密码"
            class="w-full px-3 py-2 border rounded"
          />
          <button
            type="submit"
            class="w-full px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            登录
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
const { login, loginWithGithub, fetchUser } = useAuth();

const email = ref('');
const password = ref('');

async function loginWithCredentials() {
  try {
    await login({
      email: email.value,
      password: password.value,
    });
    await navigateTo('/dashboard');
  } catch (error) {
    console.error('登录失败:', error);
  }
}

async function loginWithGithub() {
  await navigateToOAuth('github', {
    callbackUrl: '/auth/callback',
  });
}

// 如果是回调页面，处理 token
if (route.path === '/auth/callback') {
  await handleOAuthCallback();
}
</script>
```

**服务端认证 API：**

```typescript
// server/api/auth/login.post.ts
import { z } from 'zod';

const LoginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(6),
});

export default defineEventHandler(async (event) => {
  const body = await readBody(event);
  
  const validated = LoginSchema.safeParse(body);
  if (!validated.success) {
    throw createError({
      statusCode: 400,
      statusMessage: 'Invalid input',
    });
  }

  // 验证用户
  const user = await db.user.findUnique({
    where: { email: validated.data.email },
  });

  if (!user) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Invalid credentials',
    });
  }

  // 验证密码
  const isValid = await verifyPassword(user.password, validated.data.password);
  if (!isValid) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Invalid credentials',
    });
  }

  // 设置 session
  await setUserSession(event, {
    user: {
      id: user.id,
      email: user.email,
      name: user.name,
    },
  });

  return {
    user: {
      id: user.id,
      email: user.email,
      name: user.name,
    },
  };
});
```

**受保护的路由中间件：**

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware(async (to) => {
  const { loggedIn, fetchUser } = useAuth();
  
  // 如果未登录，重定向到登录页
  if (!loggedIn.value) {
    return navigateTo('/auth/login', {
      query: { redirect: to.fullPath },
    });
  }

  // 刷新用户信息
  await fetchUser();
});
```

**使用中间件保护页面：**

```vue
<!-- pages/dashboard.vue -->
<script setup lang="ts">
definePageMeta({
  middleware: ['auth'],
});
</script>

<template>
  <div>
    <h1>仪表板</h1>
    <p v-if="user">欢迎，{{ user.name }}</p>
  </div>
</template>
```

### 第三方认证集成

**使用 Nuxt Auth Module：**

```bash
npm install @sidebase/nuxt-auth
```

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@sidebase/nuxt-auth'],
  auth: {
    provider: {
      type: 'authjs',
    },
    globalAppMiddleware: true,
  },
});
```

---

## 完整 CRUD 示例

### 项目结构

本节展示一个完整的待办事项系统 CRUD 实现。

**数据库 Schema（使用 Drizzle ORM）：**

```typescript
// server/utils/db.ts
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';
import { pgTable, text, boolean, timestamp, integer } from 'drizzle-orm/pg-core';

const client = createClient({
  url: process.env.DATABASE_URL || 'file:local.db',
});

export const db = drizzle(client);

export const todos = pgTable('todos', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  description: text('description'),
  completed: boolean('completed').default(false),
  priority: integer('priority').default(0),
  userId: text('user_id').notNull(),
  createdAt: timestamp('created_at').defaultNow(),
  updatedAt: timestamp('updated_at').defaultNow(),
});
```

### 创建待办（Create）

```vue
<!-- pages/todos/new.vue -->
<script setup lang="ts">
definePageMeta({
  middleware: ['auth'],
});

const { user } = useAuth();
const router = useRouter();
const { toast } = useToast();

const title = ref('');
const description = ref('');
const priority = ref(0);
const isSubmitting = ref(false);

async function createTodo() {
  if (!title.value.trim()) {
    toast.error('标题不能为空');
    return;
  }

  isSubmitting.value = true;

  try {
    await $fetch('/api/todos', {
      method: 'POST',
      body: {
        title: title.value,
        description: description.value || undefined,
        priority: priority.value,
      },
    });

    toast.success('待办事项已创建');
    await router.push('/todos');
  } catch (error) {
    toast.error('创建失败，请重试');
  } finally {
    isSubmitting.value = false;
  }
}
</script>

<template>
  <div class="max-w-2xl mx-auto py-8">
    <div class="flex items-center gap-4 mb-8">
      <UButton
        icon="i-heroicons-arrow-left"
        variant="ghost"
        @click="router.back()"
      />
      <h1 class="text-2xl font-bold">创建待办事项</h1>
    </div>

    <form @submit.prevent="createTodo" class="space-y-6">
      <div>
        <label class="block text-sm font-medium mb-2">标题</label>
        <input
          v-model="title"
          type="text"
          placeholder="输入待办事项标题"
          class="w-full px-4 py-2 border rounded-lg"
        />
      </div>

      <div>
        <label class="block text-sm font-medium mb-2">描述（可选）</label>
        <textarea
          v-model="description"
          rows="4"
          placeholder="详细描述..."
          class="w-full px-4 py-2 border rounded-lg"
        />
      </div>

      <div>
        <label class="block text-sm font-medium mb-2">优先级</label>
        <select
          v-model="priority"
          class="w-full px-4 py-2 border rounded-lg"
        >
          <option :value="0">低</option>
          <option :value="1">中</option>
          <option :value="2">高</option>
        </select>
      </div>

      <div class="flex gap-4">
        <UButton
          type="submit"
          :loading="isSubmitting"
          :disabled="isSubmitting"
        >
          创建
        </UButton>
        <UButton
          type="button"
          variant="outline"
          @click="router.back()"
        >
          取消
        </UButton>
      </div>
    </form>
  </div>
</template>
```

```typescript
// server/api/todos/index.post.ts
import { db, todos } from '~/server/utils/db';
import { z } from 'zod';
import { eq } from 'drizzle-orm';

const CreateTodoSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().optional(),
  priority: z.number().int().min(0).max(2).default(0),
});

export default defineEventHandler(async (event) => {
  // 获取当前用户
  const user = await getUserSession(event);
  
  if (!user) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Unauthorized',
    });
  }

  const body = await readBody(event);
  const validated = CreateTodoSchema.safeParse(body);

  if (!validated.success) {
    throw createError({
      statusCode: 400,
      statusMessage: 'Invalid input',
    });
  }

  const newTodo = {
    id: crypto.randomUUID(),
    title: validated.data.title,
    description: validated.data.description || null,
    priority: validated.data.priority,
    completed: false,
    userId: user.id,
  };

  await db.insert(todos).values(newTodo);

  setResponseStatus(event, 201);
  return newTodo;
});
```

### 读取待办（Read）

```vue
<!-- pages/todos/index.vue -->
<script setup lang="ts">
definePageMeta({
  middleware: ['auth'],
});

const { user } = useAuth();

// 获取待办列表
const { data: todos, pending, error, refresh } = await useFetch('/api/todos', {
  default: () => [],
  transform: (data) => data,
});

// 按状态分组
const pendingTodos = computed(() =>
  todos.value.filter((t) => !t.completed)
);

const completedTodos = computed(() =>
  todos.value.filter((t) => t.completed)
);

// 标记完成
async function toggleComplete(id: string, completed: boolean) {
  try {
    await $fetch(`/api/todos/${id}`, {
      method: 'PATCH',
      body: { completed },
    });
    await refresh();
  } catch (error) {
    console.error('更新失败:', error);
  }
}

// 删除待办
async function deleteTodo(id: string) {
  if (!confirm('确定要删除吗？')) return;
  
  try {
    await $fetch(`/api/todos/${id}`, {
      method: 'DELETE',
    });
    await refresh();
  } catch (error) {
    console.error('删除失败:', error);
  }
}
</script>

<template>
  <div class="max-w-4xl mx-auto py-8">
    <div class="flex items-center justify-between mb-8">
      <h1 class="text-2xl font-bold">我的待办事项</h1>
      <UButton to="/todos/new">
        新建
      </UButton>
    </div>

    <div v-if="pending" class="text-center py-8">
      加载中...
    </div>

    <div v-else-if="error" class="text-red-600 py-8">
      加载失败，请刷新页面重试
    </div>

    <div v-else class="space-y-8">
      <!-- 待完成 -->
      <section>
        <h2 class="text-lg font-semibold mb-4">
          待完成 ({{ pendingTodos.length }})
        </h2>
        <div class="space-y-2">
          <div
            v-for="todo in pendingTodos"
            :key="todo.id"
            class="flex items-center gap-4 p-4 bg-white border rounded-lg"
          >
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleComplete(todo.id, true)"
              class="w-5 h-5"
            />
            <div class="flex-1">
              <p class="font-medium">{{ todo.title }}</p>
              <p v-if="todo.description" class="text-sm text-gray-500">
                {{ todo.description }}
              </p>
            </div>
            <UBadge v-if="todo.priority > 0" :color="todo.priority === 2 ? 'red' : 'yellow'">
              {{ todo.priority === 2 ? '高' : '中' }}
            </UBadge>
            <UButton
              icon="i-heroicons-trash"
              variant="ghost"
              color="red"
              @click="deleteTodo(todo.id)"
            />
          </div>
          <div v-if="pendingTodos.length === 0" class="text-gray-500 text-center py-4">
            暂无待办事项
          </div>
        </div>
      </section>

      <!-- 已完成 -->
      <section v-if="completedTodos.length > 0">
        <h2 class="text-lg font-semibold mb-4">
          已完成 ({{ completedTodos.length }})
        </h2>
        <div class="space-y-2">
          <div
            v-for="todo in completedTodos"
            :key="todo.id"
            class="flex items-center gap-4 p-4 bg-gray-50 border rounded-lg opacity-60"
          >
            <input
              type="checkbox"
              :checked="todo.completed"
              @change="toggleComplete(todo.id, false)"
              class="w-5 h-5"
            />
            <div class="flex-1 line-through">
              <p class="font-medium">{{ todo.title }}</p>
            </div>
            <UButton
              icon="i-heroicons-trash"
              variant="ghost"
              color="red"
              @click="deleteTodo(todo.id)"
            />
          </div>
        </div>
      </section>
    </div>
  </div>
</template>
```

```typescript
// server/api/todos/index.get.ts
import { db, todos } from '~/server/utils/db';
import { eq, desc } from 'drizzle-orm';

export default defineEventHandler(async (event) => {
  const user = await getUserSession(event);
  
  if (!user) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Unauthorized',
    });
  }

  const result = await db
    .select()
    .from(todos)
    .where(eq(todos.userId, user.id))
    .orderBy(desc(todos.createdAt));

  return result;
});
```

### 更新待办（Update）

```typescript
// server/api/todos/[id].patch.ts
import { db, todos } from '~/server/utils/db';
import { eq } from 'drizzle-orm';
import { z } from 'zod';

const UpdateTodoSchema = z.object({
  title: z.string().min(1).max(200).optional(),
  description: z.string().optional(),
  priority: z.number().int().min(0).max(2).optional(),
  completed: z.boolean().optional(),
});

export default defineEventHandler(async (event) => {
  const user = await getUserSession(event);
  
  if (!user) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Unauthorized',
    });
  }

  const id = getRouterParam(event, 'id');
  const body = await readBody(event);
  const validated = UpdateTodoSchema.safeParse(body);

  if (!validated.success) {
    throw createError({
      statusCode: 400,
      statusMessage: 'Invalid input',
    });
  }

  // 检查待办是否存在且属于当前用户
  const existing = await db
    .select()
    .from(todos)
    .where(eq(todos.id, id!))
    .limit(1);

  if (existing.length === 0) {
    throw createError({
      statusCode: 404,
      statusMessage: 'Todo not found',
    });
  }

  if (existing[0].userId !== user.id) {
    throw createError({
      statusCode: 403,
      statusMessage: 'Forbidden',
    });
  }

  // 更新
  await db
    .update(todos)
    .set({
      ...validated.data,
      updatedAt: new Date(),
    })
    .where(eq(todos.id, id!));

  // 返回更新后的数据
  const updated = await db
    .select()
    .from(todos)
    .where(eq(todos.id, id!))
    .limit(1);

  return updated[0];
});
```

### 删除待办（Delete）

```typescript
// server/api/todos/[id].delete.ts
import { db, todos } from '~/server/utils/db';
import { eq } from 'drizzle-orm';

export default defineEventHandler(async (event) => {
  const user = await getUserSession(event);
  
  if (!user) {
    throw createError({
      statusCode: 401,
      statusMessage: 'Unauthorized',
    });
  }

  const id = getRouterParam(event, 'id');

  // 检查待办是否存在且属于当前用户
  const existing = await db
    .select()
    .from(todos)
    .where(eq(todos.id, id!))
    .limit(1);

  if (existing.length === 0) {
    throw createError({
      statusCode: 404,
      statusMessage: 'Todo not found',
    });
  }

  if (existing[0].userId !== user.id) {
    throw createError({
      statusCode: 403,
      statusMessage: 'Forbidden',
    });
  }

  await db.delete(todos).where(eq(todos.id, id!));

  setResponseStatus(event, 204);
  return null;
});
```

---

## Nuxt 模块生态

### 官方推荐模块

| 模块 | 功能 | 安装命令 |
|------|------|---------|
| **@nuxt/image** | 图片优化 | `npx nuxi module add image` |
| **@nuxt/content** | Markdown 内容管理 | `npx nuxi module add content` |
| **@pinia/nuxt** | Pinia 状态管理 | `npx nuxi module add pinia` |
| **@nuxtjs/tailwindcss** | Tailwind CSS | `npx nuxi module add tailwindcss` |
| **@nuxtjs/google-fonts** | Google 字体优化 | `npx nuxi module add google-fonts` |
| **@nuxtjs/i18n** | 国际化 | `npx nuxi module add i18n` |
| **@nuxtseo/module** | SEO 优化 | `npx nuxi module add seo` |
| **nuxt-icon** | 图标组件 | `npx nuxi module add icon` |
| **@vueuse/nuxt** | VueUse 组合式函数 | `npx nuxi module add vueuse` |
| **@nuxtjs/color-mode** | 颜色模式 | `npx nuxi module add color-mode` |

### @nuxt/image 高级配置

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/image'],

  image: {
    // 图像优化配置
    provider: 'ipx',  // 或 'cloudinary', 'vercel', 'imgix'
    
    // 格式优先级
    format: ['webp', 'avif'],
    
    // 质量
    quality: 85,
    
    // 允许的域名
    domains: [
      'images.unsplash.com',
      'picsum.photos',
    ],
    
    // 本地图像路径
    presets: {
      avatars: {
        modifiers: {
          format: 'webp',
          width: 80,
          height: 80,
          fit: 'cover',
        },
      },
      covers: {
        modifiers: {
          format: 'webp',
          width: 1200,
          height: 630,
          fit: 'cover',
        },
      },
    },
    
    // 屏幕断点
    screens: {
      xs: 320,
      sm: 640,
      md: 768,
      lg: 1024,
      xl: 1280,
      xxl: 1536,
      '2xl': 1536,
    },
  },
});
```

```vue
<!-- 使用 @nuxt/image -->
<template>
  <div>
    <!-- 本地图像 -->
    <NuxtImg
      src="/images/hero.jpg"
      width="800"
      height="600"
      format="webp"
      quality="85"
      loading="lazy"
      placeholder
    />

    <!-- 远程图像 -->
    <NuxtImg
      src="https://images.unsplash.com/photo-xxx"
      width="800"
      height="600"
      provider="unsplash"
    />

    <!-- 使用预设 -->
    <NuxtImg
      src="/images/avatar.jpg"
      preset="avatars"
    />

    <!-- 带响应式 sizes -->
    <NuxtImg
      src="/images/cover.jpg"
      sizes="100vw sm:50vw lg:400px"
    />
  </div>
</template>
```

### @nuxt/content 进阶用法

**自定义 Markdown 处理器：**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@nuxt/content'],

  content: {
    // Markdown 配置
    markdown: {
      // 插件
      remarkPlugins: [
        'remark-emoji',
        'remark-autolinkHeadings',
      ],
      rehypePlugins: [
        'rehype-slug',
        'rehype-autolinkHeadings',
        ['rehype-pretty-code', { theme: 'github-dark' }],
      ],
      // 代码高亮
      highlight: {
        theme: 'one-dark-pro',
        langs: [
          'javascript',
          'typescript',
          'python',
          'bash',
          'json',
          'yaml',
        ],
      },
    },
    // Mermaid 图表支持
    mdc: {
      components: {
        alert: resolve('./components/content/Alert.vue'),
      },
    },
  },
});
```

**自定义 MDC 组件：**

```vue
<!-- components/content/Alert.vue -->
<script setup lang="ts">
interface Props {
  type?: 'info' | 'warning' | 'error' | 'success';
  title?: string;
}

const props = withDefaults(defineProps<Props>(), {
  type: 'info',
});

const iconMap = {
  info: 'i-heroicons-information-circle',
  warning: 'i-heroicons-exclamation-triangle',
  error: 'i-heroicons-x-circle',
  success: 'i-heroicons-check-circle',
};

const colorMap = {
  info: 'blue',
  warning: 'yellow',
  error: 'red',
  success: 'green',
};
</script>

<template>
  <div
    class="p-4 rounded-lg border"
    :class="`border-${colorMap[type]}-200 bg-${colorMap[type]}-50`"
  >
    <div class="flex items-start gap-3">
      <UIcon :name="iconMap[type]" class="w-5 h-5 mt-0.5" />
      <div>
        <p v-if="title" class="font-semibold mb-1">{{ title }}</p>
        <div class="text-sm">
          <ContentSlot :use="$slots.default" unwrap="p" />
        </div>
      </div>
    </div>
  </div>
</template>
```

---

## 常见陷阱与解决方案

### 数据获取常见问题

**1. useFetch 请求重复**

**问题：** SSR 和 CSR 阶段重复发起请求。

```vue
<!-- ❌ 问题示例 -->
<script setup>
const { data } = await useFetch('/api/data');
// 组件挂载后，客户端再次请求
</script>
```

**解决方案：** 使用 key 统一请求。

```vue
<!-- ✅ 正确示例 -->
<script setup>
const { data } = await useFetch('/api/data', {
  key: 'unique-key',  // 统一客户端缓存
});
</script>
```

**2. 响应式数据在 SSR 后丢失**

**问题：** 在 SSR 阶段设置的数据在客户端 hydration 后消失。

```vue
<!-- ❌ 问题示例 -->
<script setup>
const data = ref([]);

onMounted(async () => {
  data.value = await fetchData();  // 只在客户端执行
});
</script>
```

**解决方案：** 使用 useFetch/useAsyncData。

```vue
<!-- ✅ 正确示例 -->
<script setup>
// 自动在 SSR 和客户端执行
const { data } = await useFetch('/api/data');
</script>
```

### 状态管理常见问题

**1. useState 跨请求污染**

**问题：** SSR 环境下，状态在请求间共享导致数据污染。

```typescript
// ❌ 问题示例
const globalState = ref([]);  // 全局共享

export const useSharedState = () => globalState;
```

**解决方案：** 使用 useState 并指定唯一的 key。

```typescript
// ✅ 正确示例
export const useSharedState = () => {
  // 每个请求有独立状态
  return useState('my-state', () => ref([]));
};
```

**2. Pinia Store SSR 序列化**

**问题：** Pinia 状态无法正确序列化到客户端。

**解决方案：** 使用 pinia-plugin-persistedstate 持久化。

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  modules: ['@pinia/nuxt', '@pinia-plugin-persistedstate/nuxt'],
});
```

### 路由常见问题

**1. 中间件执行顺序**

**问题：** 中间件执行顺序不符合预期。

```typescript
// middleware/auth.ts
export default defineNuxtRouteMiddleware((to) => {
  // 只检查登录
  if (!isLoggedIn()) {
    return navigateTo('/login');
  }
});

// middleware/role.ts
export default defineNuxtRouteMiddleware((to) => {
  // 假设用户已登录
  if (!hasRole(to.meta.requiredRole)) {
    return navigateTo('/403');
  }
});
```

**解决方案：** 使用路由元信息指定顺序。

```typescript
// nuxt.config.ts
export default defineNuxtRouteMiddleware({
  order: ['auth', 'role'],
});
```

---

## Nuxt 4 新特性详解

### 目录结构变化

Nuxt 4 引入了新的目录约定，提供更灵活的项目结构：

**Nuxt 4 新结构：**

```
my-app/
├── app/                        # 应用目录（Nuxt 4 新增）
│   ├── app.vue
│   ├── pages/
│   ├── components/
│   ├── layouts/
│   ├── composables/
│   ├── middleware/
│   ├── plugins/
│   └── assets/
├── server/                     # 服务端（独立目录）
├── nuxt.config.ts
└── package.json
```

### CIN（Composable Islands）

Nuxt 4 引入了 CIN（Composable Islands），允许在 SSG 页面中混合使用服务端和客户端组件：

```vue
<!-- 使用 CIN -->
<template>
  <div>
    <!-- 静态内容 - SSG -->
    <StaticHeader />

    <!-- 交互式组件 - Island -->
    <ClientOnly>
      <InteractiveWidget />
    </ClientOnly>
  </div>
</template>
```

### 组件自动导入增强

Nuxt 4 改进了组件自动导入：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  components: {
    // 更细粒度的配置
    dirs: [
      {
        path: '~/components',
        pathPrefix: false,
        prefix: '',
      },
      {
        path: '~/components/ui',
        pathPrefix: false,
        prefix: 'Ui',
      },
    ],
  },
});
```

---

## 与其他框架对比

### Nuxt vs Next.js 深度对比

**数据获取对比：**

| 方面 | Nuxt | Next.js |
|------|------|---------|
| **服务端获取** | useAsyncData + $fetch | Server Components |
| **客户端获取** | useFetch | useEffect + SWR |
| **缓存控制** | useCachedData | next/cache |
| **类型推断** | auto-imported | use loader type |

**状态管理对比：**

| 方面 | Nuxt | Next.js |
|------|------|---------|
| **组件状态** | ref/reactive | useState |
| **全局状态** | useState/Pinia | Context/Zustand |
| **SSR 友好** | 原生支持 | 需要配置 |
| **持久化** | pinia-plugin-persistedstate | zustand-persistedstate |

### 性能优化建议

**1. 组件懒加载**

```vue
<!-- 按需加载组件 -->
<script setup>
const HeavyChart = defineAsyncComponent(() =>
  import('~/components/HeavyChart.vue')
);
</script>
```

**2. 页面预加载**

```vue
<script setup>
// 预加载下一页
definePageMeta({
  pageTransition: {
    name: 'page',
    mode: 'out-in',
  },
  // 预加载
  experimental: {
    payloadNavigation: true,
  },
});
</script>
```

**3. 路由规则优化**

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  routeRules: {
    // 首页预渲染
    '/': { prerender: true },
    // 博客列表 ISR
    '/blog': { swr: 3600 },
    // 用户页面 SSR
    '/user/**': { ssr: true },
    // 仪表板 SPA
    '/dashboard/**': { ssr: false },
  },
});
```

---

## 附录：资源链接

### 官方资源

- [Nuxt 官方文档](https://nuxt.com/docs)
- [Nuxt GitHub](https://github.com/nuxt/nuxt)
- [Nuxt Discord](https://discord.gg/nuxt)
- [Nuxt Showcase](https://nuxt.com/showcase)

### 学习资源

- [Nuxt 官方教程](https://nuxt.com/docs/getting-started/introduction)
- [Nuxt 模块列表](https://nuxt.com/modules)
- [Vue School 课程](https://vueschool.io/?filter=nuxt)
- [Nuxt 认证模块](https://sidebase.io/nuxt-auth/getting-started)

### 相关工具

- [nuxi](https://github.com/nuxt/cli) - Nuxt CLI
- [Nuxt DevTools](https://devtools.nuxt.com) - 官方开发工具
- [Drizzle ORM](https://orm.drizzle.team) - 轻量级 ORM
- [Zod](https://zod.dev) - 数据验证
- [VueUse](https://vueuse.org) - 组合式工具库
