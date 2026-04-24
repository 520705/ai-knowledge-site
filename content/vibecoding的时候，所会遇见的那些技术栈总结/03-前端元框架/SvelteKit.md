# SvelteKit 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Svelte 5 Runes 语法、SvelteKit 完整指南、端到端类型安全及与 Next.js 的深度对比。

---

## 目录

1. [[#SvelteKit 概述与核心定位]]
2. [[#Svelte 4 vs Svelte 5 对比]]
3. [[#Runes 语法详解]]
4. [[#文件系统路由]]
5. [[#数据加载]]
6. [[#表单处理]]
7. [[#Hooks 与中间件]]
8. [[#适配器系统]]
9. [[#SvelteKit vs Next.js 对比]]
10. [[#实战场景与选型建议]]

---

## SvelteKit 概述与核心定位

### SvelteKit 在前端生态中的定位

SvelteKit 是 Svelte 生态的全栈元框架，以「**编译时响应式**」和「**极小 bundle**」著称。与 React/Vue 不同，Svelte 在构建时编译响应式代码，无需运行时框架，导致：

- **零运行时开销**：产出不含响应式框架代码
- **极小的 bundle**：比 React/Vue 小 5-10 倍
- **真正的响应式**：编译器优化，告别 Virtual DOM

### SvelteKit 核心特性

| 特性 | 说明 |
|------|------|
| **编译时优化** | Svelte 编译器生成精确更新代码 |
| **多渲染模式** | SSR/SSG/SPA/全栈 |
| **端到端类型安全** | 表单、加载器全类型推断 |
| **文件路由** | 基于约定的路由系统 |
| **适配器部署** | 任意平台部署（Vercel/Netlify/Node/Deno） |
| **内置表单处理** | 渐进式增强的表单 |

### SvelteKit 与 Svelte 的关系

```
Svelte（编译器 + 响应式语言）
    │
    └── SvelteKit（全栈元框架）
           ├── 文件路由
           ├── SSR/SSG
           ├── Server Load
           ├── 表单处理
           ├── 适配器系统
           └── Hooks
```

---

## Svelte 4 vs Svelte 5 对比

### 架构理念变化

| 维度 | Svelte 4 | Svelte 5 |
|------|----------|----------|
| **响应式核心** | 编译器魔法（$:） | Runes（显式 API） |
| **状态管理** | 隐式响应式 | $state、$derived、$effect |
| **Props 传递** | export let | $props() |
| **生命周期** | onMount/onDestroy | $effect（替代） |
| **兼容性** | Svelte 4 应用 | 需要迁移 |
| **学习曲线** | 平缓 | 中等（需学习新语法） |

### Runes 引入的原因

Svelte 4 的隐式响应式虽然简洁，但在复杂场景下存在：
- 响应式边界不清晰
- 重构时容易破坏响应式
- TypeScript 支持有限

Svelte 5 的 Runes 提供了**显式响应式声明**，解决了这些问题。

### 升级建议

> [!IMPORTANT]
> Svelte 5 完全向后兼容 Svelte 4 语法，老项目可逐步迁移。新项目推荐使用 Svelte 5 Runes 语法。

---

## Runes 语法详解

### $state - 响应式状态

```svelte
<script>
  // Svelte 4
  let count = 0;
  $: doubled = count * 2;

  // Svelte 5 - Runes
  let count = $state(0);
  let doubled = $derived(count * 2);
  
  // 深层响应式
  let user = $state({ name: 'Alice', age: 30 });
  
  function birthday() {
    user.age += 1;  // 自动触发更新
  }
</script>

<button onclick={() => count++}>
  Count: {count}, Doubled: {doubled}
</button>
```

### $derived - 派生计算

```svelte
<script>
  let items = $state([
    { name: 'Apple', price: 1.5, quantity: 3 },
    { name: 'Banana', price: 0.5, quantity: 5 },
  ]);
  
  // 派生值
  let total = $derived(
    items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  
  let formattedTotal = $derived(
    new Intl.NumberFormat('zh-CN', {
      style: 'currency',
      currency: 'CNY'
    }).format(total)
  );
</script>

<p>总价: {formattedTotal}</p>
```

### $effect - 副作用

```svelte
<script>
  let query = $state('');
  let results = $state([]);
  
  // 替代 Svelte 4 的 $: 和 onMount
  $effect(() => {
    if (!query) {
      results = [];
      return;
    }
    
    const timer = setTimeout(async () => {
      const res = await fetch(`/api/search?q=${query}`);
      results = await res.json();
    }, 300);
    
    // 清理函数
    return () => clearTimeout(timer);
  });
</script>
```

### $props - Props 定义

```svelte
<script>
  // Svelte 4
  export let name;
  export let age = 18;
  export let items = [];

  // Svelte 5 - Runes
  let { 
    name, 
    age = 18, 
    items = [], 
    onUpdate 
  } = $props();
  
  // 解构时使用 $bindable 实现双向绑定
  let { value = $bindable('') } = $props();
</script>
```

### $bindable - 双向绑定

```svelte
<!-- Child.svelte -->
<script>
  let { value = $bindable('') } = $props();
</script>

<input bind:value />

<!-- Parent.svelte -->
<script>
  let inputValue = $state('');
</script>

<Child bind:value={inputValue} />
<p>输入: {inputValue}</p>
```

### 完整组件示例

```svelte
<!-- TodoList.svelte -->
<script>
  let { initialTodos = [] } = $props();
  
  let todos = $state(initialTodos);
  let newTodo = $state('');
  let filter = $state('all');
  
  let filteredTodos = $derived.by(() => {
    if (filter === 'all') return todos;
    if (filter === 'active') return todos.filter(t => !t.done);
    return todos.filter(t => t.done);
  });
  
  let activeCount = $derived(todos.filter(t => !t.done).length);
  
  function addTodo() {
    if (!newTodo.trim()) return;
    todos = [...todos, { id: Date.now(), text: newTodo, done: false }];
    newTodo = '';
  }
  
  function toggleTodo(id) {
    todos = todos.map(t => 
      t.id === id ? { ...t, done: !t.done } : t
    );
  }
  
  function removeTodo(id) {
    todos = todos.filter(t => t.id !== id);
  }
</script>

<div class="todo-app">
  <h1>待办事项</h1>
  
  <div class="input-row">
    <input 
      bind:value={newTodo}
      onkeydown={(e) => e.key === 'Enter' && addTodo()}
      placeholder="添加新待办..."
    />
    <button onclick={addTodo}>添加</button>
  </div>
  
  <div class="filters">
    <button class:active={filter === 'all'} onclick={() => filter = 'all'}>全部</button>
    <button class:active={filter === 'active'} onclick={() => filter = 'active'}>进行中</button>
    <button class:active={filter === 'completed'} onclick={() => filter = 'completed'}>已完成</button>
  </div>
  
  <ul class="todo-list">
    {#each filteredTodos as todo (todo.id)}
      <li class:done={todo.done}>
        <input 
          type="checkbox" 
          checked={todo.done}
          onchange={() => toggleTodo(todo.id)}
        />
        <span>{todo.text}</span>
        <button onclick={() => removeTodo(todo.id)}>删除</button>
      </li>
    {/each}
  </ul>
  
  <p class="count">{activeCount} 项待完成</p>
</div>
```

---

## 文件系统路由

### 路由约定

```
src/routes/
├── +page.svelte          → /
├── +layout.svelte        → 布局（所有子路由共享）
├── about/
│   └── +page.svelte      → /about
├── blog/
│   ├── +page.svelte      → /blog
│   ├── +page.server.ts   → 服务端数据加载
│   └── [slug]/
│       ├── +page.svelte  → /blog/:slug
│       └── +page.ts      → 客户端数据加载
└── (auth)/
    ├── login/
    │   └── +page.svelte  → /login
    └── register/
        └── +page.svelte  → /register
```

### 布局系统

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import '../app.css';
  
  let { children, data } = $props();
</script>

<div class="app">
  <nav>
    <a href="/">首页</a>
    <a href="/blog">博客</a>
    <a href="/about">关于</a>
  </nav>
  
  <main>
    {@render children()}
  </main>
  
  <footer>
    <p>&copy; 2026</p>
  </footer>
</div>
```

### 嵌套布局

```
src/routes/
├── (main)/
│   ├── +layout.svelte    → 主布局（带导航）
│   ├── +page.svelte      → / (首页)
│   ├── blog/
│   │   ├── +page.svelte  → /blog
│   │   └── [slug]/
│   │       └── +page.svelte → /blog/:slug
│   └── dashboard/
│       └── +page.svelte  → /dashboard
└── (marketing)/
    ├── +layout.svelte    → 营销布局
    ├── about/
    │   └── +page.svelte  → /about
    └── pricing/
        └── +page.svelte  → /pricing
```

---

## 数据加载

### +page.ts - Universal Load

Universal Load 在服务端和客户端都执行：

```typescript
// src/routes/blog/[slug]/+page.ts
import { error } from '@sveltejs/kit';
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch, params, url }) => {
  const res = await fetch(`/api/posts/${params.slug}`);
  
  if (!res.ok) {
    throw error(404, 'Post not found');
  }
  
  const post = await res.json();
  
  return {
    post,
    relatedPosts: await fetchRelatedPosts(post.tags),
  };
};
```

### +page.server.ts - Server Load

Server Load 仅在服务端执行，可直接访问数据库、文件系统等：

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ params, locals }) => {
  const post = await db.post.findUnique({
    where: { slug: params.slug },
    include: { author: true, tags: true },
  });
  
  if (!post) {
    throw error(404, 'Post not found');
  }
  
  return {
    post,
    user: locals.user,
  };
};
```

### 数据流对比

| 类型 | 服务端执行 | 客户端执行 | 适用场景 |
|------|-----------|-----------|---------|
| **+page.ts** | ✅ | ✅ | 需要 SSR + 客户端刷新 |
| **+page.server.ts** | ✅ | ❌ | 数据库、API 密钥、安全数据 |
| **+layout.ts** | ✅ | ✅ | 嵌套路由共享数据 |
| **+layout.server.ts** | ✅ | ❌ | 全局认证状态 |

### 错误处理

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error, fail } from '@sveltejs/kit';
import type { PageServerLoad, Actions } from './$types';

export const load: PageServerLoad = async ({ params }) => {
  const post = await db.post.findUnique({
    where: { slug: params.slug },
  });
  
  if (!post) {
    throw error(404, {
      message: '文章不存在',
      code: 'POST_NOT_FOUND',
    });
  }
  
  return { post };
};
```

---

## 表单处理

### 表单操作（Actions）

SvelteKit 的表单处理是「渐进式增强」的，无需 JavaScript 也能工作：

```typescript
// src/routes/contact/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import type { Actions } from './$types';
import { z } from 'zod';

const ContactSchema = z.object({
  name: z.string().min(1, '姓名不能为空'),
  email: z.string().email('邮箱格式不正确'),
  message: z.string().min(10, '消息至少10个字符'),
});

export const actions: Actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const data = Object.fromEntries(formData);
    
    const validated = ContactSchema.safeParse(data);
    
    if (!validated.success) {
      return fail(400, {
        errors: validated.error.flatten().fieldErrors,
        values: data,
      });
    }
    
    await sendEmail(validated.data);
    
    return { success: true };
  },
};
```

### 表单组件

```svelte
<!-- src/routes/contact/+page.svelte -->
<script>
  import { enhance } from '$app/forms';
  import { fly } from 'svelte/transition';
  
  let { form } = $props();
  let loading = $state(false);
</script>

<h1>联系我们</h1>

<form 
  method="POST" 
  use:enhance={() => {
    loading = true;
    return async ({ update }) => {
      await update();
      loading = false;
    };
  }}
>
  <div class="field">
    <label for="name">姓名</label>
    <input 
      type="text" 
      id="name" 
      name="name" 
      value={form?.values?.name ?? ''}
    />
    {#if form?.errors?.name}
      <span class="error" transition:fly>{{ form.errors.name[0] }}</span>
    {/if}
  </div>
  
  <div class="field">
    <label for="email">邮箱</label>
    <input 
      type="email" 
      id="email" 
      name="email"
      value={form?.values?.email ?? ''}
    />
    {#if form?.errors?.email}
      <span class="error" transition:fly>{form.errors.email[0]}</span>
    {/if}
  </div>
  
  <div class="field">
    <label for="message">消息</label>
    <textarea 
      id="message" 
      name="message"
      rows={5}
    >{form?.values?.message ?? ''}</textarea>
    {#if form?.errors?.message}
      <span class="error" transition:fly>{form.errors.message[0]}</span>
    {/if}
  </div>
  
  <button type="submit" disabled={loading}>
    {loading ? '发送中...' : '发送'}
  </button>
  
  {#if form?.success}
    <p class="success" transition:fly>发送成功！</p>
  {/if}
</form>
```

### 多表单操作

```typescript
export const actions: Actions = {
  login: async ({ request }) => {
    // 登录处理
  },
  register: async ({ request }) => {
    // 注册处理
  },
};
```

```svelte
<form method="POST" action="?/login">
  <!-- 登录表单 -->
</form>

<form method="POST" action="?/register">
  <!-- 注册表单 -->
</form>
```

---

## Hooks 与中间件

### Handle Hook

```typescript
// src/hooks.server.ts
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
  // 获取 token
  const token = event.cookies.get('session');
  
  if (token) {
    try {
      const user = await verifyToken(token);
      event.locals.user = user;
    } catch {
      // token 无效，清除
      event.cookies.delete('session', { path: '/' });
    }
  }
  
  // 权限检查
  const protectedRoutes = ['/dashboard', '/settings', '/admin'];
  if (protectedRoutes.some(r => event.url.pathname.startsWith(r))) {
    if (!event.locals.user) {
      return new Response(null, {
        status: 303,
        headers: { Location: '/login' },
      });
    }
  }
  
  return resolve(event);
};
```

### Locals 类型定义

```typescript
// src/app.d.ts
declare global {
  namespace App {
    interface Locals {
      user?: {
        id: string;
        name: string;
        email: string;
        role: 'admin' | 'user';
      };
    }
    // 页面数据
    interface PageData {
      user?: App.Locals['user'];
    }
  }
}

export {};
```

### 事件 Hook

```typescript
// src/hooks.server.ts
export const handleError = ({ event, error }) => {
  console.error('Error:', error);
  
  return {
    message: '发生了错误',
    code: 'INTERNAL_ERROR',
  };
};
```

---

## 适配器系统

### 适配器列表

| 适配器 | 部署平台 | 命令 |
|--------|---------|------|
| **@sveltejs/adapter-auto** | 自动检测 | 默认 |
| **@sveltejs/adapter-vercel** | Vercel | `npm i -D @sveltejs/adapter-vercel` |
| **@sveltejs/adapter-netlify** | Netlify | `npm i -D @sveltejs/adapter-netlify` |
| **@sveltejs/adapter-node** | Node.js | `npm i -D @sveltejs/adapter-node` |
| **@sveltejs/adapter-deno** | Deno Deploy | `npm i -D @sveltejs/adapter-deno` |
| **@sveltejs/adapter-cloudflare** | Cloudflare | `npm i -D @sveltejs/adapter-cloudflare` |

### Vercel 适配器

```typescript
// svelte.config.js
import adapter from '@sveltejs/adapter-vercel';

export default {
  kit: {
    adapter: adapter({
      runtime: 'nodejs20.x',
      regions: ['hkg1'],
      // ISR 支持
      isr: {
        expiration: 60 * 60,  // 1小时
      },
    }),
  },
};
```

### 自定义 Node 适配器

```typescript
// svelte.config.js
import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter({
      out: 'build',
      precompress: true,
      envPrefix: '',
    }),
  },
};
```

---

## SvelteKit vs Next.js 对比

### 核心架构对比

| 维度 | SvelteKit | Next.js |
|------|-----------|---------|
| **底层框架** | Svelte | React |
| **响应式** | 编译时 | 运行时（Virtual DOM） |
| **Bundle 大小** | 极小（~5KB） | 中等（~50KB） | 
| **学习曲线** | 平缓 | 中等 |
| **类型安全** | 优秀 | 优秀 |
| **生态** | 较小但增长中 | 庞大成熟 |

### 性能对比

| 指标 | SvelteKit | Next.js |
|------|-----------|---------|
| **JS Bundle** | ~5KB | ~50KB+ |
| **Time to Interactive** | 极快 | 快 |
| **SSR 性能** | 优秀 | 优秀 |
| **构建速度** | 快 | 中等（Turbopack 加速） |
| **开发体验** | 极简 | 完善 |

### 路由对比

| 特性 | SvelteKit | Next.js |
|------|-----------|---------|
| **动态路由** | `[slug]` | `[slug]` |
| **可选参数** | `[[slug]]` | `[slug]` |
| **Catch-all** | `[...slug]` | `[...slug]` |
| **布局** | `+layout.svelte` | `layout.tsx` |
| **加载** | `+page.ts` / `+page.server.ts` | `loading.tsx` |
| **中间件** | `hooks.server.ts` | `middleware.ts` |

### 数据加载对比

```typescript
// SvelteKit
// +page.server.ts
export const load = async ({ params }) => {
  const data = await db.query(params.id);
  return { data };
};

// Next.js
// page.tsx
export default async function Page({ params }) {
  const data = await db.query(params.id);
  return <div>{data}</div>;
}
```

### 表单处理对比

| 特性 | SvelteKit | Next.js |
|------|-----------|---------|
| **渐进增强** | 原生支持 | Server Actions |
| **无 JS 提交** | ✅ | ✅ |
| **验证** | Zod + fail() | Zod + useActionState |
| **重定向** | redirect() | redirect() |

### 选型建议

> [!TIP]
> 选择建议：
> - **性能优先** → SvelteKit（极小 bundle）
> - **熟悉 React** → Next.js
> - **熟悉 Svelte/Vue** → SvelteKit
> - **庞大生态需求** → Next.js
> - **团队小、快速迭代** → SvelteKit
> - **企业级稳定** → Next.js

---

## 实战场景与选型建议

### AI 对话应用实战

```typescript
// src/routes/api/chat/+server.ts
import { json, error } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ request, locals }) => {
  if (!locals.user) {
    throw error(401, 'Unauthorized');
  }
  
  const { messages } = await request.json();
  
  const stream = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  });
  
  return new Response(stream.toReadableStream(), {
    headers: {
      'Content-Type': 'text/event-stream',
    },
  });
};
```

```svelte
<!-- src/routes/chat/+page.svelte -->
<script>
  import { invalidate } from '$app/navigation';
  
  let { data } = $props();
  let messages = $state(data.messages ?? []);
  let input = $state('');
  let loading = $state(false);
  
  async function sendMessage() {
    if (!input.trim() || loading) return;
    
    const userMsg = { role: 'user', content: input };
    messages = [...messages, userMsg];
    input = '';
    loading = true;
    
    try {
      const res = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages }),
      });
      
      const reader = res.body?.getReader();
      const decoder = new TextDecoder();
      let assistantMsg = { role: 'assistant', content: '' };
      
      while (reader) {
        const { done, value } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value);
        assistantMsg.content += chunk;
        messages = [...messages.slice(0, -1), assistantMsg];
      }
    } finally {
      loading = false;
    }
  }
</script>

<div class="chat-container">
  <div class="messages">
    {#each messages as msg}
      <div class="message {msg.role}">
        <strong>{msg.role === 'user' ? '你' : 'AI'}:</strong>
        {msg.content}
      </div>
    {/each}
  </div>
  
  <form onsubmit={(e) => { e.preventDefault(); sendMessage(); }}>
    <input bind:value={input} placeholder="输入消息..." />
    <button type="submit" disabled={loading}>
      {loading ? '思考中...' : '发送'}
    </button>
  </form>
</div>
```

### 成本估算

| 方案 | 月成本（100万 PV） | 说明 |
|------|-------------------|------|
| **Vercel** | $20 + 超额 | Serverless 函数 |
| **Railway** | $5-50 | 按使用计费 |
| **自托管** | $10-50 | Node.js 服务器 |

---

> [!SUCCESS]
> SvelteKit 结合了 Svelte 5 的编译时响应式优势和全栈框架的灵活性。其极小的 bundle、优秀的性能、端到端类型安全和完善的表单处理，使 SvelteKit 成为追求性能的现代 Web 开发首选框架。对于从 Vue 或 React 迁移的团队，Svelte 的简洁语法和 SvelteKit 的完善功能提供了出色的开发体验。

---

## 完整安装指南

### 环境要求与前置准备

**Node.js 版本要求：**
- SvelteKit: Node.js 18.0 或更高版本
- 推荐使用 Node.js 20 LTS 或 22 LTS

```bash
# 使用 nvm 安装 Node.js 20
nvm install 20
nvm use 20

# 验证安装
node --version  # 应显示 v20.x.x
```

### 项目创建详细流程

**方式一：使用 sv create（推荐）**

```bash
# 交互式创建
npx sv create

# 非交互式创建
npx sv create my-app --template minimal --types ts --no-add-ons

# 使用特定模板
npx sv create my-app --template skeleton --types ts --no-add-ons
npx sv create my-app --template skeleton-library --types ts --no-add-ons
```

**sv 命令行选项：**

| 选项 | 说明 | 默认值 |
|------|------|--------|
| `--template` | 项目模板 | minimal |
| `--types` | 类型检查 | ts |
| `--no-add-ons` | 跳过插件安装 | false |
| `--no-install` | 跳过 npm install | false |

**方式二：使用 create-svelte（传统方式）**

```bash
# 交互式创建
npm create svelte@latest my-app

# 非交互式创建
npm create svelte@latest my-app -- --template skeleton --types typescript
```

**方式三：手动创建 SvelteKit 项目**

```bash
# 创建项目目录
mkdir my-app && cd my-app

# 初始化 npm 项目
npm init -y

# 安装 SvelteKit 依赖
npm install -D @sveltejs/kit @sveltejs/adapter-auto svelte vite

# 安装 TypeScript（可选）
npm install -D typescript

# 创建项目结构
mkdir -p src/routes src/lib
touch svelte.config.js vite.config.ts tsconfig.json src/app.html
```

### SvelteKit 配置详解

**svelte.config.js：**

```javascript
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

/** @type {import('@sveltejs/kit').Config} */
const config = {
	preprocess: vitePreprocess(),

	kit: {
		// 适配器
		adapter: adapter({
			// 运行时
			runtime: 'nodejs20.x',
			// ISR 支持（Vercel）
			isr: {
				expiration: 60 * 60, // 1小时
			},
			// 部署环境
			regions: ['hkg1'],
		}),

		// 别名
		alias: {
			$components: 'src/lib/components',
			$stores: 'src/lib/stores',
			$utils: 'src/lib/utils',
		},

		// 服务端路径
		csrf: {
			checkOrigin: true,
		},

		// 路径解析
		resolve: {
			paths: {
				'$lib': ['./src/lib'],
				'$lib/*': ['./src/lib/*'],
			},
		},

		// Service Worker
		serviceWorker: {
			register: true,
		},

		// 预加载
		preload: {
			'page': 'viewport',
			'data': 'link',
		},

		// 类型生成
		types: {
			filter: (warnings) => {
				// 过滤类型警告
				return warnings;
			},
		},
	},
};

export default config;
```

**vite.config.ts：**

```typescript
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';

export default defineConfig({
	plugins: [sveltekit()],

	server: {
		port: 5173,
		host: true,
		strictPort: false,
		proxy: {
			'/api': {
				target: 'http://localhost:3000',
				changeOrigin: true,
			},
		},
	},

	preview: {
		port: 4173,
	},

	build: {
		target: 'esnext',
		minify: true,
	},

	optimizeDeps: {
		include: ['svelte', '@sveltejs/kit'],
	},

	ssr: {
		noExternal: ['@sveltejs/kit'],
	},
});
```

### 目录结构最佳实践

**推荐的 SvelteKit 项目结构：**

```
my-app/
├── src/
│   ├── app.html               # HTML 模板
│   ├── app.css               # 全局样式
│   ├── app.d.ts              # 类型定义
│   ├── hooks.server.ts        # 服务端 Hooks
│   ├── hooks.client.ts        # 客户端 Hooks
│   ├── lib/                   # 库代码
│   │   ├── components/        # 组件
│   │   │   ├── base/        # 基础组件
│   │   │   │   ├── Button.svelte
│   │   │   │   └── Card.svelte
│   │   │   └── ui/          # UI 组件
│   │   │       ├── Input.svelte
│   │   │       └── Modal.svelte
│   │   ├── server/           # 服务端代码
│   │   │   ├── db.ts        # 数据库客户端
│   │   │   └── auth.ts      # 认证逻辑
│   │   ├── stores/           # Svelte stores
│   │   │   ├── user.ts
│   │   │   └── cart.ts
│   │   ├── utils/            # 工具函数
│   │   │   ├── format.ts
│   │   │   └── validation.ts
│   │   └── types/            # 类型定义
│   │       └── index.ts
│   └── routes/                # 路由
│       ├── +layout.svelte   # 根布局
│       ├── +layout.server.ts # 根布局服务端数据
│       ├── +page.svelte     # 首页
│       ├── +page.server.ts   # 首页服务端数据
│       ├── about/
│       │   └── +page.svelte # 关于页
│       ├── blog/
│       │   ├── +page.svelte # 博客列表
│       │   ├── +page.server.ts
│       │   └── [slug]/
│       │       ├── +page.svelte # 博客详情
│       │       ├── +page.ts     # 客户端加载
│       │       └── +page.server.ts # 服务端加载
│       ├── (auth)/           # 路由组 - 认证
│       │   ├── +layout.svelte
│       │   ├── login/
│       │   │   └── +page.svelte
│       │   └── register/
│       │       └── +page.svelte
│       ├── (app)/            # 路由组 - 应用
│       │   ├── +layout.svelte
│       │   ├── dashboard/
│       │   │   └── +page.svelte
│       │   └── settings/
│       │       └── +page.svelte
│       └── api/               # API 路由
│           └── posts/
│               ├── +server.ts
│               └── [id]/
│                   └── +server.ts
├── static/                    # 静态文件
│   ├── favicon.png
│   └── robots.txt
├── tests/                     # 测试
│   ├── unit/
│   └── integration/
├── svelte.config.js           # Svelte 配置
├── vite.config.ts             # Vite 配置
├── tsconfig.json              # TypeScript 配置
└── package.json
```

---

## 认证方案详解

### Lucia Auth 集成

Lucia Auth 是 SvelteKit 推荐的认证解决方案：

**安装：**

```bash
npm install lucia @lucia-auth/adapter-drizzle
```

**数据库 Schema：**

```typescript
// src/lib/server/db.ts
import { drizzle } from 'drizzle-orm/libsql';
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
	id: text('id').primaryKey(),
	email: text('email').notNull().unique(),
	name: text('name'),
	emailVerified: integer('email_verified', { mode: 'boolean' }).default(false),
	createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
	updatedAt: integer('updated_at', { mode: 'timestamp' }).notNull(),
});

export const sessions = sqliteTable('sessions', {
	id: text('id').primaryKey(),
	userId: text('user_id')
		.notNull()
		.references(() => users.id),
	expiresAt: integer('expires_at', { mode: 'timestamp' }).notNull(),
});
```

**Lucia 配置：**

```typescript
// src/lib/server/lucia.ts
import { lucia } from 'lucia';
import { DrizzleSQLiteAdapter } from '@lucia-auth/adapter-drizzle';
import { db } from './db';
import { users, sessions } from './db';

const adapter = new DrizzleSQLiteAdapter(db, sessions, users);

export const auth = lucia({
	adapter,
	env: import.meta.env.PROD ? 'PROD' : 'DEV',
	getUserAttributes: (attributes) => {
		return {
			email: attributes.email,
			name: attributes.name,
			emailVerified: attributes.emailVerified,
		};
	},
});

declare module 'lucia' {
	interface Register {
		Lucia: typeof auth;
		DatabaseUserAttributes: {
			email: string;
			name: string | null;
			emailVerified: boolean;
		};
	}
}

export type Auth = typeof auth;
```

**Hooks 认证：**

```typescript
// src/hooks.server.ts
import { auth } from '$lib/server/lucia';
import type { Handle } from '@sveltejs/kit';

export const handle: Handle = async ({ event, resolve }) => {
	event.locals.auth = auth.handleRequest(event);
	const session = await event.locals.auth.validate();

	if (session) {
		event.locals.user = session.user;
		event.locals.session = session;
	} else {
		event.locals.user = null;
		event.locals.session = null;
	}

	return resolve(event);
};
```

**类型定义：**

```typescript
// src/app.d.ts
declare global {
	namespace App {
		interface Locals {
			auth: import('lucia').SessionRequest;
			user: import('lucia').User | null;
			session: import('luia').Session | null;
		}
		interface PageData {
			user: import('lucia').User | null;
		}
	}
}

export {};
```

**登录页面：**

```svelte
<!-- src/routes/login/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import { goto } from '$app/navigation';

	let { form } = $props();

	let loading = $state(false);
</script>

<div class="min-h-screen flex items-center justify-center">
	<div class="w-full max-w-md p-8 bg-white rounded-lg shadow-md">
		<h1 class="text-2xl font-bold mb-6 text-center">登录</h1>

		{#if form?.error}
			<div class="mb-4 p-3 bg-red-100 text-red-700 rounded">
				{form.error}
			</div>
		{/if}

		<form
			method="POST"
			use:enhance={() => {
				loading = true;
				return async ({ result, update }) => {
					loading = false;
					if (result.type === 'redirect') {
						await goto(result.location);
					} else {
						await update();
					}
				};
			}}
			class="space-y-4"
		>
			<div>
				<label for="email" class="block text-sm font-medium mb-1">邮箱</label>
				<input
					type="email"
					id="email"
					name="email"
					value={form?.email ?? ''}
					required
					class="w-full px-3 py-2 border rounded-lg"
				/>
			</div>

			<div>
				<label for="password" class="block text-sm font-medium mb-1">密码</label>
				<input
					type="password"
					id="password"
					name="password"
					required
					class="w-full px-3 py-2 border rounded-lg"
				/>
			</div>

			<button
				type="submit"
				disabled={loading}
				class="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50"
			>
				{loading ? '登录中...' : '登录'}
			</button>
		</form>

		<p class="mt-4 text-center text-sm">
			还没有账号？<a href="/register" class="text-blue-600 hover:underline">注册</a>
		</p>
	</div>
</div>
```

**登录 Action：**

```typescript
// src/routes/login/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import type { Actions, PageServerLoad } from './$types';
import { auth } from '$lib/server/lucia';
import { verify } from '@node-rs/argon2';

export const load: PageServerLoad = async ({ locals }) => {
	const user = await locals.auth.getUser();
	if (user) {
		throw redirect(302, '/dashboard');
	}
};

export const actions: Actions = {
	default: async ({ request, locals }) => {
		const formData = await request.formData();
		const email = formData.get('email') as string;
		const password = formData.get('password') as string;

		if (!email || !password) {
			return fail(400, {
				error: '邮箱和密码不能为空',
				email,
			});
		}

		try {
			const user = await db.query.users.findFirst({
				where: eq(users.email, email),
			});

			if (!user) {
				return fail(400, {
					error: '邮箱或密码错误',
					email,
				});
			}

			const validPassword = await verify(user.hashedPassword, password);
			if (!validPassword) {
				return fail(400, {
					error: '邮箱或密码错误',
					email,
				});
			}

			const session = await auth.createSession({
				userId: user.id,
				attributes: {},
			});

			const sessionCookie = auth.createSessionCookie(session);

			return redirect(302, '/dashboard', {
				headers: {
					'set-cookie': sessionCookie.serialize(),
				},
			});
		} catch (error) {
			console.error('Login error:', error);
			return fail(500, {
				error: '服务器错误，请重试',
				email,
			});
		}
	},
};
```

---

## 完整 CRUD 示例

### 项目结构

本节展示一个完整的笔记系统 CRUD 实现。

**数据库 Schema：**

```typescript
// src/lib/server/db.ts
import { drizzle } from 'drizzle-orm/libsql';
import { createClient } from '@libsql/client';
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';

const client = createClient({
	url: process.env.DATABASE_URL || 'file:local.db',
});

export const db = drizzle(client);

export const notes = sqliteTable('notes', {
	id: text('id').primaryKey(),
	title: text('title').notNull(),
	content: text('content'),
	authorId: text('author_id').notNull(),
	createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
	updatedAt: integer('updated_at', { mode: 'timestamp' }).notNull(),
	archived: integer('archived', { mode: 'boolean' }).default(false),
	color: text('color').default('#fef08a'),
});

export const tags = sqliteTable('tags', {
	id: text('id').primaryKey(),
	name: text('name').notNull(),
	slug: text('slug').notNull(),
});

export const noteTags = sqliteTable('note_tags', {
	noteId: text('note_id')
		.notNull()
		.references(() => notes.id),
	tagId: text('tag_id')
		.notNull()
		.references(() => tags.id),
});
```

### 创建笔记（Create）

```svelte
<!-- src/routes/notes/new/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';

	let { form } = $props();

	let loading = $state(false);
	let title = $state('');
	let content = $state('');
	let color = $state('#fef08a');

	const colors = [
		{ value: '#fef08a', label: '黄色' },
		{ value: '#fca5a5', label: '红色' },
		{ value: '#86efac', label: '绿色' },
		{ value: '#93c5fd', label: '蓝色' },
		{ value: '#c4b5fd', label: '紫色' },
	];
</script>

<div class="max-w-2xl mx-auto py-8 px-4">
	<div class="flex items-center gap-4 mb-6">
		<a href="/notes" class="p-2 hover:bg-gray-100 rounded">
			<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
			</svg>
		</a>
		<h1 class="text-2xl font-bold">新建笔记</h1>
	</div>

	<form
		method="POST"
		use:enhance={() => {
			loading = true;
			return async ({ result, update }) => {
				loading = false;
				await update();
			};
		}}
		class="space-y-6"
	>
		{#if form?.error}
			<div class="p-4 bg-red-100 text-red-700 rounded-lg">
				{form.error}
			</div>
		{/if}

		<div>
			<label for="title" class="block text-sm font-medium mb-2">标题</label>
			<input
				type="text"
				id="title"
				name="title"
				bind:value={title}
				placeholder="笔记标题"
				required
				class="w-full px-4 py-3 text-lg border rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
			/>
			{#if form?.errors?.title}
				<p class="mt-1 text-sm text-red-600">{form.errors.title[0]}</p>
			{/if}
		</div>

		<div>
			<label for="content" class="block text-sm font-medium mb-2">内容</label>
			<textarea
				id="content"
				name="content"
				bind:value={content}
				placeholder="写下你的想法..."
				rows="12"
				class="w-full px-4 py-3 border rounded-lg resize-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
			></textarea>
		</div>

		<div>
			<label class="block text-sm font-medium mb-2">颜色</label>
			<div class="flex gap-2">
				{#each colors as c}
					<label class="cursor-pointer">
						<input type="radio" name="color" value={c.value} class="sr-only" checked={color === c.value} />
						<span
							class="block w-8 h-8 rounded-full border-2 {color === c.value
								? 'border-gray-800'
								: 'border-transparent'}"
							style="background-color: {c.value}"
						></span>
					</label>
				{/each}
			</div>
		</div>

		<div class="flex gap-4 pt-4">
			<button
				type="submit"
				disabled={loading}
				class="flex-1 bg-blue-600 text-white py-3 px-6 rounded-lg hover:bg-blue-700 disabled:opacity-50 font-medium"
			>
				{loading ? '保存中...' : '保存笔记'}
			</button>
			<a
				href="/notes"
				class="px-6 py-3 border rounded-lg hover:bg-gray-50 font-medium"
			>
				取消
			</a>
		</div>
	</form>
</div>
```

```typescript
// src/routes/notes/new/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import type { Actions, PageServerLoad } from './$types';
import { db, notes } from '$lib/server/db';
import { z } from 'zod';
import { eq } from 'drizzle-orm';

const NoteSchema = z.object({
	title: z.string().min(1, '标题不能为空').max(200, '标题过长'),
	content: z.string().optional(),
	color: z.string().regex(/^#[0-9a-fA-F]{6}$/).default('#fef08a'),
});

export const load: PageServerLoad = async ({ locals }) => {
	if (!locals.user) {
		throw redirect(302, '/login');
	}
};

export const actions: Actions = {
	default: async ({ request, locals }) => {
		if (!locals.user) {
			throw redirect(302, '/login');
		}

		const formData = await request.formData();
		const data = Object.fromEntries(formData);

		const validated = NoteSchema.safeParse(data);

		if (!validated.success) {
			return fail(400, {
				errors: validated.error.flatten().fieldErrors,
				values: data,
			});
		}

		try {
			const newNote = {
				id: crypto.randomUUID(),
				title: validated.data.title,
				content: validated.data.content || null,
				authorId: locals.user.id,
				color: validated.data.color,
				createdAt: new Date(),
				updatedAt: new Date(),
				archived: false,
			};

			await db.insert(notes).values(newNote);
		} catch (error) {
			console.error('Create note error:', error);
			return fail(500, {
				error: '创建笔记失败，请重试',
			});
		}

		throw redirect(302, '/notes');
	},
};
```

### 读取笔记（Read）

```svelte
<!-- src/routes/notes/+page.svelte -->
<script lang="ts">
	import { goto } from '$app/navigation';

	let { data } = $props();

	let filter = $state<'all' | 'archived'>('all');
	let searchQuery = $state('');

	const filteredNotes = $derived(() => {
		let result = data.notes;

		if (filter === 'archived') {
			result = result.filter((n) => n.archived);
		}

		if (searchQuery) {
			const query = searchQuery.toLowerCase();
			result = result.filter(
				(n) =>
					n.title.toLowerCase().includes(query) ||
					n.content?.toLowerCase().includes(query)
			);
		}

		return result;
	});

	function formatDate(date: Date) {
		return new Intl.DateTimeFormat('zh-CN', {
			year: 'numeric',
			month: 'short',
			day: 'numeric',
		}).format(date);
	}
</script>

<div class="max-w-6xl mx-auto py-8 px-4">
	<div class="flex items-center justify-between mb-8">
		<h1 class="text-3xl font-bold">我的笔记</h1>
		<a
			href="/notes/new"
			class="bg-blue-600 text-white px-6 py-3 rounded-lg hover:bg-blue-700 font-medium flex items-center gap-2"
		>
			<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
			</svg>
			新建笔记
		</a>
	</div>

	<!-- 筛选和搜索 -->
	<div class="flex gap-4 mb-6">
		<div class="flex-1">
			<input
				type="search"
				bind:value={searchQuery}
				placeholder="搜索笔记..."
				class="w-full px-4 py-2 border rounded-lg"
			/>
		</div>
		<div class="flex gap-2">
			<button
				class="px-4 py-2 rounded-lg {filter === 'all'
					? 'bg-blue-100 text-blue-700'
					: 'bg-gray-100 hover:bg-gray-200'}"
				onclick={() => (filter = 'all')}
			>
				全部
			</button>
			<button
				class="px-4 py-2 rounded-lg {filter === 'archived'
					? 'bg-blue-100 text-blue-700'
					: 'bg-gray-100 hover:bg-gray-200'}"
				onclick={() => (filter = 'archived')}
			>
				已归档
			</button>
		</div>
	</div>

	<!-- 笔记网格 -->
	{#if filteredNotes().length === 0}
		<div class="text-center py-16 text-gray-500">
			{#if searchQuery}
				<p>没有找到匹配的笔记</p>
			{:else}
				<p>还没有笔记</p>
				<a href="/notes/new" class="text-blue-600 hover:underline mt-2 inline-block">
					创建第一篇笔记
				</a>
			{/if}
		</div>
	{:else}
		<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
			{#each filteredNotes() as note}
				<a
					href="/notes/{note.id}"
					class="block p-6 rounded-xl shadow-sm hover:shadow-md transition-shadow {note.archived
						? 'opacity-60'
						: ''}"
					style="background-color: {note.color}20; border: 1px solid {note.color}40"
				>
					<h3 class="font-semibold text-lg mb-2 line-clamp-1">{note.title}</h3>
					<p class="text-gray-600 text-sm mb-4 line-clamp-3">{note.content || '无内容'}</p>
					<div class="flex items-center justify-between text-xs text-gray-500">
						<span>{formatDate(note.createdAt)}</span>
						{#if note.archived}
							<span class="text-orange-600">已归档</span>
						{/if}
					</div>
				</a>
			{/each}
		</div>
	{/if}
</div>
```

```typescript
// src/routes/notes/+page.server.ts
import type { PageServerLoad } from './$types';
import { db, notes } from '$lib/server/db';
import { redirect } from '@sveltejs/kit';
import { eq, desc } from 'drizzle-orm';

export const load: PageServerLoad = async ({ locals }) => {
	if (!locals.user) {
		throw redirect(302, '/login');
	}

	const userNotes = await db
		.select()
		.from(notes)
		.where(eq(notes.authorId, locals.user.id))
		.orderBy(desc(notes.updatedAt));

	return {
		notes: userNotes,
	};
};
```

### 更新笔记（Update）

```svelte
<!-- src/routes/notes/[id]/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import { goto } from '$app/navigation';

	let { data, form } = $props();

	let loading = $state(false);
	let title = $state(data.note.title);
	let content = $state(data.note.content || '');
	let color = $state(data.note.color);

	const colors = [
		{ value: '#fef08a', label: '黄色' },
		{ value: '#fca5a5', label: '红色' },
		{ value: '#86efac', label: '绿色' },
		{ value: '#93c5fd', label: '蓝色' },
		{ value: '#c4b5fd', label: '紫色' },
	];
</script>

<div class="max-w-2xl mx-auto py-8 px-4">
	<div class="flex items-center gap-4 mb-6">
		<a href="/notes" class="p-2 hover:bg-gray-100 rounded">
			<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
			</svg>
		</a>
		<h1 class="text-2xl font-bold">编辑笔记</h1>
	</div>

	<form
		method="POST"
		action="?/update"
		use:enhance={() => {
			loading = true;
			return async ({ result, update }) => {
				loading = false;
				if (result.type === 'redirect') {
					await goto(result.location);
				} else {
					await update();
				}
			};
		}}
		class="space-y-6"
	>
		{#if form?.error}
			<div class="p-4 bg-red-100 text-red-700 rounded-lg">
				{form.error}
			</div>
		{/if}

		<div>
			<label for="title" class="block text-sm font-medium mb-2">标题</label>
			<input
				type="text"
				id="title"
				name="title"
				bind:value={title}
				required
				class="w-full px-4 py-3 text-lg border rounded-lg"
			/>
		</div>

		<div>
			<label for="content" class="block text-sm font-medium mb-2">内容</label>
			<textarea
				id="content"
				name="content"
				bind:value={content}
				rows="12"
				class="w-full px-4 py-3 border rounded-lg resize-none"
			></textarea>
		</div>

		<div>
			<label class="block text-sm font-medium mb-2">颜色</label>
			<div class="flex gap-2">
				{#each colors as c}
					<label class="cursor-pointer">
						<input type="radio" name="color" value={c.value} class="sr-only" checked={color === c.value} />
						<span
							class="block w-8 h-8 rounded-full border-2 {color === c.value
								? 'border-gray-800'
								: 'border-transparent'}"
							style="background-color: {c.value}"
						></span>
					</label>
				{/each}
			</div>
		</div>

		<div class="flex gap-4 pt-4">
			<button
				type="submit"
				disabled={loading}
				class="flex-1 bg-blue-600 text-white py-3 px-6 rounded-lg hover:bg-blue-700 disabled:opacity-50"
			>
				{loading ? '保存中...' : '保存更改'}
			</button>

			<button
				type="submit"
				formaction="?/archive"
				class="px-6 py-3 border rounded-lg hover:bg-gray-50"
			>
				{data.note.archived ? '取消归档' : '归档'}
			</button>

			<button
				type="submit"
				formaction="?/delete"
				class="px-6 py-3 text-red-600 border border-red-200 rounded-lg hover:bg-red-50"
				onclick={(e) => {
					if (!confirm('确定要删除这篇笔记吗？')) {
						e.preventDefault();
					}
				}}
			>
				删除
			</button>
		</div>
	</form>
</div>
```

```typescript
// src/routes/notes/[id]/+page.server.ts
import { error, fail, redirect } from '@sveltejs/kit';
import type { Actions, PageServerLoad } from './$types';
import { db, notes } from '$lib/server/db';
import { eq } from 'drizzle-orm';

export const load: PageServerLoad = async ({ params, locals }) => {
	if (!locals.user) {
		throw redirect(302, '/login');
	}

	const note = await db.select().from(notes).where(eq(notes.id, params.id)).get();

	if (!note) {
		throw error(404, '笔记不存在');
	}

	if (note.authorId !== locals.user.id) {
		throw error(403, '无权限访问');
	}

	return { note };
};

export const actions: Actions = {
	update: async ({ request, params, locals }) => {
		if (!locals.user) {
			throw redirect(302, '/login');
		}

		const formData = await request.formData();
		const title = formData.get('title') as string;
		const content = formData.get('content') as string;
		const color = formData.get('color') as string;

		if (!title?.trim()) {
			return fail(400, { error: '标题不能为空' });
		}

		await db
			.update(notes)
			.set({
				title: title.trim(),
				content: content || null,
				color,
				updatedAt: new Date(),
			})
			.where(eq(notes.id, params.id));

		return { success: true };
	},

	archive: async ({ params, locals }) => {
		if (!locals.user) {
			throw redirect(302, '/login');
		}

		const note = await db.select().from(notes).where(eq(notes.id, params.id)).get();

		if (note?.authorId !== locals.user.id) {
			throw error(403, '无权限');
		}

		await db
			.update(notes)
			.set({
				archived: !note?.archived,
				updatedAt: new Date(),
			})
			.where(eq(notes.id, params.id));

		return { success: true };
	},

	delete: async ({ params, locals }) => {
		if (!locals.user) {
			throw redirect(302, '/login');
		}

		const note = await db.select().from(notes).where(eq(notes.id, params.id)).get();

		if (note?.authorId !== locals.user.id) {
			throw error(403, '无权限');
		}

		await db.delete(notes).where(eq(notes.id, params.id));

		throw redirect(302, '/notes');
	},
};
```

---

## 高级路由模式

### 路由组与布局切换

```svelte
<!-- src/routes/(marketing)/+layout.svelte -->
<!-- 营销布局 - 简洁导航 -->
<script>
	let { children } = $props();
</script>

<nav class="flex justify-between p-4">
	<a href="/" class="font-bold">Brand</a>
	<div class="flex gap-4">
		<a href="/features">功能</a>
		<a href="/pricing">定价</a>
		<a href="/login">登录</a>
	</div>
</nav>

{@render children()}
```

```svelte
<!-- src/routes/(app)/+layout.svelte -->
<!-- 应用布局 - 完整功能 -->
<script>
	let { children } = $props();
	let sidebarOpen = $state(true);
</script>

<div class="flex h-screen">
	<!-- 侧边栏 -->
	<aside class="w-64 bg-gray-900 text-white {sidebarOpen ? '' : 'hidden'}">
		<div class="p-4">
			<h2 class="font-bold text-xl">Dashboard</h2>
		</div>
		<nav class="p-4 space-y-2">
			<a href="/dashboard" class="block px-4 py-2 rounded hover:bg-gray-800">首页</a>
			<a href="/notes" class="block px-4 py-2 rounded hover:bg-gray-800">笔记</a>
			<a href="/settings" class="block px-4 py-2 rounded hover:bg-gray-800">设置</a>
		</nav>
	</aside>

	<!-- 主内容 -->
	<main class="flex-1 overflow-auto">
		<header class="border-b p-4">
			<button onclick={() => (sidebarOpen = !sidebarOpen)}>Toggle</button>
		</header>
		<div class="p-8">
			{@render children()}
		</div>
	</main>
</div>
```

### 错误处理

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="min-h-screen flex items-center justify-center">
	<div class="text-center">
		<h1 class="text-6xl font-bold mb-4">{$page.status}</h1>
		<p class="text-xl text-gray-600 mb-8">{$page.error?.message || '出错了'}</p>
		<a href="/" class="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700">
			返回首页
		</a>
	</div>
</div>
```

```svelte
<!-- src/routes/blog/+error.svelte -->
<!-- 博客特定错误页面 -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="max-w-2xl mx-auto py-16 text-center">
	<h1 class="text-4xl font-bold mb-4">博客出错了</h1>
	<p class="text-gray-600 mb-8">
		{#if $page.status === 404}
			这篇文章不存在或已被删除。
		{:else}
			加载博客时出现了一些问题。
		{/if}
	</p>
	<a href="/blog" class="text-blue-600 hover:underline">返回博客列表</a>
</div>
```

---

## 常见陷阱与解决方案

### Svelte 5 迁移问题

**1. 响应式变量解构**

**问题：** 解构 $state 后失去响应性。

```svelte
<!-- ❌ 问题 -->
<script>
	let { user } = $props();
	// user 不再是响应式的
</script>
```

**解决方案：** 使用 $derived 或保留对象引用。

```svelte
<!-- ✅ 正确 -->
<script>
	let { user } = $props();
	
	// 如果需要解构，使用 $derived
	const userName = $derived(user?.name || 'Guest');
	
	// 或者直接使用 user 对象
</script>
<p>{user.name}</p>
```

**2. Store 在 Svelte 5 中的使用**

**问题：** Svelte 5 中传统的 writable store 变化。

**解决方案：** 继续使用 store，但通过 $ 语法订阅。

```svelte
<script>
	import { writable } from 'svelte/store';
	
	const count = writable(0);
</script>

<!-- Svelte 5 中仍可使用 $ 语法 -->
<button onclick={() => $count++}>
	Count: {$count}
</button>
```

### 性能优化问题

**1. 避免不必要的 SSR**

**问题：** 不需要 SEO 的页面进行 SSR。

**解决方案：** 使用 SSR 禁用。

```typescript
// src/routes/dashboard/+page.ts
// 客户端加载
export const ssr = false;

// 或使用 load
export const load: PageLoad = async ({ fetch }) => {
	const data = await fetch('/api/dashboard').then(r => r.json());
	return { data };
};
```

**2. 大列表虚拟滚动**

**问题：** 渲染大量项目导致性能问题。

**解决方案：** 使用虚拟列表或分页。

```svelte
<script lang="ts">
	import { spring } from 'svelte/motion';
	
	let items = $state(Array.from({ length: 10000 }, (_, i) => i));
	let visibleItems = $state(50);
	
	const scrollPosition = spring(0, {
		stiffness: 0.2,
		damping: 0.4,
	});
	
	function handleScroll(e: Event) {
		const target = e.target as HTMLElement;
		const scrolled = target.scrollTop;
		const viewportHeight = target.clientHeight;
		
		// 计算可见项
		scrollPosition.set(scrolled);
		visibleItems = Math.min(items.length, Math.ceil((scrolled + viewportHeight) / 50));
	}
</script>

<div class="h-screen overflow-auto" onscroll={handleScroll}>
	<div class="relative" style="height: {items.length * 50}px">
		<div class="sticky top-0">
			{#each items.slice(Math.floor($scrollPosition / 50), visibleItems) as item}
				<div class="h-[50px] border-b">{item}</div>
			{/each}
		</div>
	</div>
</div>
```

---

## 与其他框架深度对比

### SvelteKit vs Next.js vs Nuxt

**数据加载对比：**

| 特性 | SvelteKit | Next.js | Nuxt |
|------|-----------|---------|------|
| **服务端加载** | +page.server.ts | Server Components | useAsyncData |
| **客户端加载** | +page.ts | useEffect | useFetch |
| **布局数据** | +layout.server.ts | layout.tsx | useAsyncData |
| **类型安全** | 自动推断 | 需定义类型 | 自动推断 |

**表单处理对比：**

| 特性 | SvelteKit | Next.js | Nuxt |
|------|-----------|---------|------|
| **渐进增强** | 原生 | Server Actions | useFetch |
| **无 JS 提交** | ✅ | ✅ | ⚠️ 需配置 |
| **验证** | Zod | Zod | Zod |
| **多操作** | ✅ | ✅ | ✅ |

---

## 附录：资源链接

### 官方资源

- [SvelteKit 文档](https://kit.svelte.dev/docs)
- [Svelte 5 文档](https://svelte-5-preview.vercel.app/docs)
- [SvelteKit GitHub](https://github.com/sveltejs/kit)
- [Svelte Discord](https://discord.gg/svelte)

### 学习资源

- [Svelte 教程](https://learn.svelte.dev)
- [SvelteKit 教程](https://kit.svelte.dev/tutorial)
- [Svelte Society](https://sveltesociety.dev)
- [Svelte 播客](https://podcast.svelte.dev)

### 相关工具

- [Lucia Auth](https://lucia-auth.com) - SvelteKit 认证
- [Drizzle ORM](https://orm.drizzle.team) - 轻量级 ORM
- [Zod](https://zod.dev) - 数据验证
- [svelte-i18n](https://github.com/sveltejs/svelte-i18n) - 国际化
