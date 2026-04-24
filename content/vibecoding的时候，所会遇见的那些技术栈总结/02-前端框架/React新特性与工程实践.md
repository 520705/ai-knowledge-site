---
title: React新特性与工程实践
date: 2026-04-24
tags:
  - React
  - 前端开发
  - React19
  - Next.js
  - 测试
  - 部署
  - DevOps
categories:
  - 前端框架
---

> [!NOTE]
> 本文档面向有 React 基础的开发者，讲解 React 19 新特性、工程实践和最佳实践。建议先阅读 [[React入门完全指南]] 和 [[React进阶完全指南]] 打牢基础。

---

## React 19 新特性

### React 19 概述

React 19 是 React 历史上最重大的更新之一，它带来了一系列新特性和改进，大幅提升了开发体验和应用性能。这些新特性让 React 在全栈开发的道路上迈出了重要一步，同时也让客户端应用的开发更加高效。

React 19 的核心改进包括：Server Components 让组件可以在服务器端渲染，大幅减少客户端 JavaScript 体积；Actions 简化了表单处理和异步操作；新的 use() Hook 提供了更灵活的数据读取方式；React Compiler 自动优化代码性能。这些改进不是互相独立的，它们共同构成了 React 的现代化开发体验。

### Server Components（服务器组件）

Server Components 是 React 19 最令人兴奋的特性之一。简单来说，Server Components 是默认在服务器上渲染的 React 组件，它们可以访问服务器端的数据和资源（比如直接读取文件系统、访问数据库），但不能使用 useState、useEffect 等客户端 Hook。

为什么要这样做呢？因为服务器组件不需要发送到客户端，所以可以大幅减少 JavaScript 捆绑包的大小。一个复杂的列表组件，如果从服务器端渲染，可能只需要 0KB 的客户端 JavaScript；而之前即使是最简单的组件也需要几 KB。

```tsx
// app/users/page.tsx (默认是 Server Component)
import { db } from '@/lib/db';
import { UserCard } from '@/components/UserCard'; // Client Component
import { AddUserForm } from '@/components/AddUserForm'; // Client Component

// 这个组件在服务器上运行
// 可以直接访问数据库，不需要 API 调用
async function UserList() {
  // 直接从数据库获取数据
  const users = await db.query('SELECT * FROM users ORDER BY created_at DESC');
  
  return (
    <div className="grid gap-4">
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user}
          onDelete={() => {/* 这是 Client Component 的功能 */}}
        />
      ))}
    </div>
  );
}

// 混用 Server 和 Client Components
export default async function UsersPage() {
  return (
    <main className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">用户列表</h1>
      
      {/* AddUserForm 是 Client Component，可以处理用户交互 */}
      <AddUserForm />
      
      {/* UserList 是 Server Component，可以直接访问数据库 */}
      <UserList />
    </main>
  );
}
```

```tsx
// components/AddUserForm.tsx
'use client';  // 声明为 Client Component

import { useState } from 'react';

function AddUserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    const response = await fetch('/api/users', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name, email }),
    });
    
    if (response.ok) {
      setName('');
      setEmail('');
      // 触发列表刷新
      window.location.reload();
    }
  };
  
  return (
    <form onSubmit={handleSubmit} className="mb-6 space-y-4">
      <div>
        <label htmlFor="name" className="block mb-1">姓名</label>
        <input
          id="name"
          type="text"
          value={name}
          onChange={(e) => setName(e.target.value)}
          className="w-full border p-2 rounded"
          required
        />
      </div>
      <div>
        <label htmlFor="email" className="block mb-1">邮箱</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full border p-2 rounded"
          required
        />
      </div>
      <button type="submit" className="px-4 py-2 bg-blue-500 text-white rounded">
        添加用户
      </button>
    </form>
  );
}

export { AddUserForm };
```

**Server Components vs Client Components 选择指南**：

| 场景 | 推荐组件类型 | 原因 |
|------|------------|------|
| 获取数据（API/DB） | Server Component | 直接访问，不需要 API |
| 访问后端资源 | Server Component | 可以使用 Node.js API |
| 用户交互（点击、输入） | Client Component | 需要 useState/useEffect |
| useState / useEffect | Client Component | 只能在客户端运行 |
| 第三方库（需要 DOM） | Client Component | 依赖浏览器 API |
| 静态内容展示 | Server Component | 减少 JS 体积 |

> [!IMPORTANT]
> Server Components 并不意味着"不用学 React"了。相反，你需要理解哪些逻辑适合放在服务器端，哪些必须放在客户端。这种分离是 React 19 应用设计的核心技能。

### Actions：简化的表单处理

Actions 是 React 19 引入的新特性，它让表单提交和服务器交互变得前所未有的简单。传统的表单处理需要管理 loading 状态、错误处理、pending 状态等；而 Actions 把这些都自动化了。

```tsx
// app/actions.ts
'use server';  // 这个文件的所有导出都是 Server Actions

import { revalidatePath } from 'next/cache';

export async function submitForm(formData: FormData) {
  // 从 FormData 中提取数据
  const name = formData.get('name');
  const email = formData.get('email');
  
  // 验证
  if (!name || !email) {
    return { error: '姓名和邮箱是必填的' };
  }
  
  // 保存到数据库
  await db.users.create({
    name: name.toString(),
    email: email.toString()
  });
  
  // 清除缓存，触发页面更新
  revalidatePath('/users');
  
  return { success: true };
}

// 渐进式增强的表单
export async function updateUser(userId: string, formData: FormData) {
  const name = formData.get('name');
  
  if (!name) {
    return { error: '姓名不能为空' };
  }
  
  await db.users.update(userId, { name: name.toString() });
  revalidatePath(`/users/${userId}`);
  
  return { success: true };
}
```

```tsx
// components/SignupForm.tsx
'use client';

import { submitForm } from '@/app/actions';

function SignupForm() {
  return (
    <form action={submitForm} className="space-y-4">
      <div>
        <label htmlFor="name">姓名</label>
        <input 
          id="name" 
          name="name" 
          type="text" 
          className="w-full border p-2 rounded"
          required
        />
      </div>
      
      <div>
        <label htmlFor="email">邮箱</label>
        <input 
          id="email" 
          name="email" 
          type="email" 
          className="w-full border p-2 rounded"
          required
        />
      </div>
      
      {/* useFormStatus 提供 pending 状态 */}
      <SubmitButton />
    </form>
  );
}

// 分离按钮组件以使用 useFormStatus
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button 
      type="submit" 
      disabled={pending}
      className="w-full px-4 py-2 bg-blue-500 text-white rounded disabled:bg-blue-300"
    >
      {pending ? '提交中...' : '提交'}
    </button>
  );
}
```

**Actions 的优势**：

渐进式增强让表单在 JavaScript 加载之前就能工作；自动处理 loading 状态，不需要手动管理 isPending；与 React Server Components 无缝集成；类型安全的参数和返回值。

### use() Hook

use() 是 React 19 引入的新 Hook，它提供了一种在组件中读取 Promise 和 Context 的新方式。与 useEffect + useState 相比，use() 可以直接在组件顶层使用，代码更简洁。

```tsx
import { use, Suspense } from 'react';

// 使用 use 读取 Promise
function UserProfile({ userPromise }) {
  // 不需要 loading 状态，不需要 useEffect
  // use() 会自动处理 Promise 的 pending 和 rejection 状态
  const user = use(userPromise);
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 配合 Suspense 使用
function UserPage({ userId }) {
  return (
    <Suspense fallback={<UserSkeleton />}>
      <UserProfile userPromise={fetchUser(userId)} />
    </Suspense>
  );
}

// 使用 use 读取 Context（可以条件性使用）
function ThemedButton({ theme, children }) {
  // use 可以在条件语句中使用
  // useContext 必须放在组件顶层
  if (theme === 'special') {
    return <SpecialButton>{children}</SpecialButton>;
  }
  
  const ctx = use(ThemeContext);
  return (
    <button 
      className={`btn btn-${ctx.theme}`}
      onClick={ctx.toggleTheme}
    >
      {children}
    </button>
  );
}
```

### 资源预加载 API

React 19 提供了新的资源预加载 API，可以帮助优化页面加载性能。

```tsx
import { preload, preloadDNS, prefetchDNS } from 'react-dom';

// 预加载字体
function FontPreloader() {
  useEffect(() => {
    preload('/fonts/custom.woff2', { as: 'font', type: 'font/woff2' });
  }, []);
  
  return <div>使用自定义字体的内容</div>;
}

// 预加载图片
function ImageGallery({ images }) {
  const preloadImage = (src) => {
    preload(src, { as: 'image' });
  };
  
  return (
    <div className="grid">
      {images.map((img, index) => (
        <div 
          key={img.src}
          onMouseEnter={() => preloadImage(img.src)}  // 悬停时预加载
        >
          <img src={img.src} alt={img.alt} loading="lazy" />
        </div>
      ))}
    </div>
  );
}

// DNS 预解析
function ExternalContent() {
  useEffect(() => {
    // 预解析第三方域名
    preloadDNS('https://api.example.com');
    prefetchDNS('https://analytics.example.com');
  }, []);
  
  return <div>将使用外部 API 的内容</div>;
}
```

---

## React Compiler

### 什么是 React Compiler

React Compiler（之前叫做 React Forget）是 React 团队开发的一个实验性编译器，它能自动优化 React 代码，减少不必要的重新渲染。与其手动使用 useMemo、useCallback、React.memo 来优化代码，React Compiler 可以自动分析代码并在编译时添加这些优化。

### 工作原理

React Compiler 会分析你的组件代码，识别哪些状态和 props 在渲染时是稳定的，然后自动添加 memoization。这就像有一个智能助手在你写代码时自动优化它。

```tsx
// 编译前的代码（你写的）
function ProductList({ products, onAddToCart }) {
  const [filter, setFilter] = useState('all');
  
  // 每次渲染都重新计算
  const filteredProducts = products.filter(p => {
    return filter === 'all' || p.category === filter;
  });
  
  // 每次渲染都创建新函数
  const handleAdd = (productId) => {
    onAddToCart(productId);
  };
  
  return (
    <div>
      <FilterButtons onFilterChange={setFilter} />
      {filteredProducts.map(product => (
        <ProductItem 
          key={product.id}
          product={product}
          onAdd={handleAdd}
        />
      ))}
    </div>
  );
}

// 编译后（React Compiler 自动优化）
// React Compiler 会自动添加 useMemo 和 useCallback
function ProductList({ products, onAddToCart }) {
  const [filter, setFilter] = useState('all');
  
  // 自动添加 useMemo
  const filteredProducts = React.useMemo(() => {
    return products.filter(p => {
      return filter === 'all' || p.category === filter;
    });
  }, [filter, products]);
  
  // 自动添加 useCallback
  const handleAdd = React.useCallback((productId) => {
    onAddToCart(productId);
  }, [onAddToCart]);
  
  return (
    <div>
      <FilterButtons onFilterChange={setFilter} />
      {filteredProducts.map(product => (
        <ProductItem 
          key={product.id}
          product={product}
          onAdd={handleAdd}
        />
      ))}
    </div>
  );
}
```

### 安装和配置

```bash
# 安装 React Compiler
npm install babel-plugin-react-compiler
```

```javascript
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      target: '19',  // 目标 React 版本
      // 推荐用于 Vite
    }]
  ],
};
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [
    react({
      // 启用 React Compiler
      compiler: {
        target: '19',
      },
    }),
  ],
});
```

> [!TIP]
> React Compiler 仍然处于实验阶段，生产环境使用请谨慎评估。但对于学习目的和实验性项目，它是了解 React 优化机制的好工具。

---

## Next.js 完整集成

### Next.js 概述

Next.js 是目前最流行的 React 全栈框架，它基于 React 构建，提供了服务端渲染、静态站点生成、API 路由等功能。对于需要 SEO 优化、首屏加载性能、或者需要服务端能力的应用，Next.js 是最佳选择。

### App Router vs Pages Router

Next.js 13 引入了 App Router，这是新的文件系统路由方式。App Router 基于 React Server Components 构建，提供了更好的性能和更灵活的数据获取方式。

```tsx
// app/page.tsx - 首页 (Server Component)
import Link from 'next/link';

export default function HomePage() {
  return (
    <main className="container mx-auto p-4">
      <h1 className="text-4xl font-bold mb-4">欢迎</h1>
      <p className="mb-6">这是 Next.js App Router 示例</p>
      
      <div className="space-x-4">
        <Link href="/about" className="text-blue-500 hover:underline">
          关于页面
        </Link>
        <Link href="/users" className="text-blue-500 hover:underline">
          用户列表
        </Link>
      </div>
    </main>
  );
}
```

```tsx
// app/about/page.tsx - 关于页面
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: '关于我们',
  description: '了解更多关于我们团队的信息',
};

export default function AboutPage() {
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-4">关于我们</h1>
      <p className="text-gray-600">
        我们是一个热爱技术的团队，致力于创造优秀的产品。
      </p>
    </div>
  );
}
```

```tsx
// app/users/page.tsx - 用户列表页面 (Server Component)
import { db } from '@/lib/db';

export const dynamic = 'force-dynamic';  // 禁用静态生成

async function getUsers() {
  // 直接查询数据库
  return await db.query('SELECT * FROM users ORDER BY created_at DESC');
}

export default async function UsersPage() {
  const users = await getUsers();
  
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-3xl font-bold mb-6">用户列表</h1>
      
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {users.map(user => (
          <div key={user.id} className="border rounded-lg p-4 shadow">
            <h2 className="font-bold">{user.name}</h2>
            <p className="text-gray-600">{user.email}</p>
            <p className="text-sm text-gray-400">
              注册时间: {new Date(user.created_at).toLocaleDateString()}
            </p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### API 路由

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';

// 获取用户列表
export async function GET() {
  const users = await db.query('SELECT * FROM users');
  return NextResponse.json(users);
}

// 创建用户
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { name, email } = body;
    
    if (!name || !email) {
      return NextResponse.json(
        { error: '姓名和邮箱是必填的' },
        { status: 400 }
      );
    }
    
    const result = await db.users.create({ name, email });
    
    return NextResponse.json(result, { status: 201 });
  } catch (error) {
    return NextResponse.json(
      { error: '创建用户失败' },
      { status: 500 }
    );
  }
}
```

### 静态生成和动态路由

```tsx
// app/posts/[slug]/page.tsx - 动态路由
import { Metadata } from 'next';
import { notFound } from 'next/navigation';

interface PostPageProps {
  params: { slug: string };
}

// 静态生成参数
export async function generateStaticParams() {
  const posts = await db.query('SELECT slug FROM posts');
  return posts.map(post => ({
    slug: post.slug,
  }));
}

// 动态元数据
export async function generateMetadata({ params }: PostPageProps): Promise<Metadata> {
  const post = await db.posts.findBySlug(params.slug);
  
  if (!post) {
    return { title: '文章未找到' };
  }
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage],
    },
  };
}

export default async function PostPage({ params }: PostPageProps) {
  const post = await db.posts.findBySlug(params.slug);
  
  if (!post) {
    notFound();
  }
  
  return (
    <article className="container mx-auto p-4 max-w-3xl">
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        <p className="text-gray-600">
          发布于 {new Date(post.publishedAt).toLocaleDateString()}
        </p>
      </header>
      
      <div dangerouslySetInnerHTML={{ __html: post.content }} />
    </article>
  );
}
```

---

## 测试策略

### 测试的重要性

测试是软件开发中不可或缺的一部分。好的测试能让你在修改代码时更有信心，快速发现 Bug，减少回归问题。对于 React 应用，我们通常会编写以下几种测试：单元测试测试独立的函数或组件；集成测试测试多个组件协同工作；端到端测试测试完整的用户流程。

### Vitest + React Testing Library

Vitest 是一个由 Vite 驱动的测试框架，它提供了极快的开发体验。React Testing Library 则专注于测试组件的实际行为，而不是实现细节。

```bash
# 安装测试依赖
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
  },
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
```

```tsx
// src/components/Counter.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { describe, it, expect, vi } from 'vitest';
import { Counter } from './Counter';

describe('Counter', () => {
  it('renders initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText(/计数: 5/i)).toBeInTheDocument();
  });
  
  it('increments when + button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={0} />);
    
    const button = screen.getByRole('button', { name: /\+/i });
    await user.click(button);
    
    expect(screen.getByText(/计数: 1/i)).toBeInTheDocument();
  });
  
  it('decrements when - button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={0} />);
    
    const button = screen.getByRole('button', { name: /-/i });
    await user.click(button);
    
    expect(screen.getByText(/计数: -1/i)).toBeInTheDocument();
  });
  
  it('resets when reset button is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={5} />);
    
    const resetButton = screen.getByRole('button', { name: /重置/i });
    await user.click(resetButton);
    
    expect(screen.getByText(/计数: 0/i)).toBeInTheDocument();
  });
  
  it('calls onIncrement callback when incrementing', async () => {
    const mockFn = vi.fn();
    const user = userEvent.setup();
    render(<Counter initialCount={0} onIncrement={mockFn} />);
    
    const button = screen.getByRole('button', { name: /\+/i });
    await user.click(button);
    
    expect(mockFn).toHaveBeenCalledTimes(1);
  });
});
```

### 组件测试最佳实践

```tsx
// src/components/UserCard.test.tsx
import { render, screen } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const,
  };
  
  it('renders user name and email', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('张三')).toBeInTheDocument();
    expect(screen.getByText('zhangsan@example.com')).toBeInTheDocument();
  });
  
  it('renders avatar when provided', () => {
    render(
      <UserCard 
        user={{ ...mockUser, avatar: 'https://example.com/avatar.jpg' }} 
      />
    );
    
    const avatar = screen.getByRole('img');
    expect(avatar).toHaveAttribute('src', 'https://example.com/avatar.jpg');
  });
  
  it('renders default avatar when not provided', () => {
    render(<UserCard user={mockUser} />);
    
    // 应该显示用户名的首字母
    expect(screen.getByText('张')).toBeInTheDocument();
  });
  
  it('displays admin badge for admin users', () => {
    render(<UserCard user={mockUser} />);
    
    expect(screen.getByText('管理员')).toBeInTheDocument();
    expect(screen.getByText('管理员')).toHaveClass('bg-red-500');
  });
  
  it('calls onEdit when edit button is clicked', async () => {
    const onEdit = vi.fn();
    const user = userEvent.setup();
    
    render(<UserCard user={mockUser} onEdit={onEdit} />);
    
    const editButton = screen.getByRole('button', { name: /编辑/i });
    await user.click(editButton);
    
    expect(onEdit).toHaveBeenCalledWith('1');
  });
});
```

### Mock API 和外部依赖

```typescript
// src/test/mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  // Mock GET 请求
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: '张三', email: 'zhangsan@example.com' },
      { id: '2', name: '李四', email: 'lisi@example.com' },
    ]);
  }),
  
  // Mock POST 请求
  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({
      id: '3',
      ...body,
    }, { status: 201 });
  }),
];
```

```typescript
// src/test/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);
```

```typescript
// src/test/setup.ts
import { server } from './mocks/server';
import '@testing-library/jest-dom';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

---

## 生产环境部署

### 构建优化

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { compression } from 'vite-plugin-compression';

export default defineConfig({
  plugins: [
    react(),
    compression({
      algorithm: 'gzip',
      ext: '.gz',
    }),
  ],
  build: {
    outDir: 'dist',
    sourcemap: process.env.NODE_ENV !== 'production',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash-es', 'axios'],
          icons: ['lucide-react'],
        },
        chunkFileNames: 'assets/[name]-[hash].js',
        entryFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash][extname]',
      },
    },
    // 报告文件大小
    reportCompressedSize: true,
    // 警告超过 500KB 的 chunk
    chunkSizeWarningLimit: 500,
  },
});
```

### CI/CD 配置

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm run test:coverage
      
      - name: Build
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
      
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: https://your-app.vercel.app
          budgetPath: ./lighthouse-budget.json
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Vercel 部署

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "headers": [
    {
      "source": "/sw.js",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=0, must-revalidate" }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" }
      ]
    }
  ]
}
```

### Docker 部署

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# 生产镜像
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser
USER appuser

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## 最佳实践与常见陷阱

### 常见错误和解决方案

```tsx
// 错误 1: 在渲染时创建新对象/函数/数组
function BadComponent() {
  // 每次渲染都会创建新对象，导致子组件不必要地重新渲染
  const style = { color: 'red' };
  const handleClick = () => console.log('click');
  const items = [1, 2, 3];
  
  return (
    <div style={style} onClick={handleClick}>
      {items.map(i => <Item key={i} />)}
    </div>
  );
}

// 正确做法: 使用 useMemo 和 useCallback
function GoodComponent() {
  const style = useMemo(() => ({ color: 'red' }), []);
  const handleClick = useCallback(() => console.log('click'), []);
  const items = useMemo(() => [1, 2, 3], []);
  
  return (
    <div style={style} onClick={handleClick}>
      {items.map(i => <Item key={i} />)}
    </div>
  );
}

// 错误 2: 依赖数组不完整
function DataFetcher({ url, filter }) {
  useEffect(() => {
    fetchData(url, filter).then(setData);
  }, []); // ❌ 缺少 url 和 filter
  
  return <div>{data}</div>;
}

// 正确做法
function DataFetcherFixed({ url, filter }) {
  useEffect(() => {
    fetchData(url, filter).then(setData);
  }, [url, filter]); // ✅ 包含所有依赖
  
  return <div>{data}</div>;
}

// 错误 3: 直接修改状态
function BadState() {
  const [items, setItems] = useState([]);
  
  const addItem = (item) => {
    items.push(item); // ❌ 直接修改数组
    setItems(items);
  };
}

// 正确做法
function GoodState() {
  const [items, setItems] = useState([]);
  
  const addItem = (item) => {
    setItems([...items, item]); // ✅ 创建新数组
  };
}
```

### 代码组织最佳实践

```tsx
// 1. 组件文件组织
// components/
// ├── Button/
// │   ├── Button.tsx        // 组件实现
// │   ├── Button.test.tsx   // 测试
// │   ├── Button.stories.tsx // Storybook
// │   ├── index.ts          // 导出
// │   └── types.ts          // 类型定义（如果复杂）

// 2. 页面文件组织
// app/
// ├── page.tsx              // 页面
// ├── loading.tsx          // Loading 状态
// ├── error.tsx            // 错误边界
// └── not-found.tsx        // 404 页面

// 3. 自定义 Hook 组织
// hooks/
// ├── useAuth.ts           // 认证相关
// ├── useFetch.ts          // 数据获取
// ├── useLocalStorage.ts   // 持久化
// └── index.ts             // 统一导出

// 4. 类型组织
// types/
// ├── api.ts               // API 类型
// ├── user.ts              // 用户相关类型
// ├── index.ts             // 统一导出
```

### 性能检查清单

| 检查项 | 工具/方法 |
|-------|----------|
| 组件重渲染分析 | React DevTools Profiler |
| 包大小分析 | webpack-bundle-analyzer / rollup-plugin-visualizer |
| Lighthouse 性能 | Chrome DevTools Lighthouse |
| Core Web Vitals | PageSpeed Insights |
| 代码分割 | React.lazy + Suspense |
| 图片优化 | next/image / vite-plugin-imagemin |

---

## 常见问题与解决方案

### Q: 如何选择状态管理方案？

A: 遵循"够用就好"原则。首先尝试 useState，复杂时用 useReducer，需要跨组件共享时用 Context。只有当这些都无法满足需求时，才考虑引入 Zustand、Jotai 或 Redux。

### Q: 何时使用 Server Components vs Client Components？

A: 数据显示、访问后端资源时用 Server Component；需要交互、使用 useState/useEffect、或者依赖浏览器 API 时用 Client Component。记住：Server Components 可以导入 Client Components，但 Client Components 不能导入 Server Components。

### Q: React Compiler 适合生产环境使用吗？

A: React Compiler 仍然是实验性功能。对于新项目，可以尝试使用；对于生产环境，建议等待稳定版本。在做出决定前，务必充分测试。

### Q: 如何优化大型列表的渲染性能？

A: 使用虚拟化（react-window 或 react-virtualized）、保持列表项稳定（使用唯一 ID 作为 key）、避免内联函数和对象、使用 React.memo 包装列表项。

---

## 参考资料

| 资源 | 链接 |
|------|------|
| React 官方文档 | https://react.dev |
| React 19 更新日志 | https://react.dev/blog |
| Next.js 文档 | https://nextjs.org/docs |
| Vitest 文档 | https://vitest.dev |
| React Testing Library | https://testing-library.com/docs/react-testing-library |

---

## 相关文档

- [[React入门完全指南]] - React 基础入门
- [[React进阶完全指南]] - React 高级特性

> [!SUCCESS]
> 恭喜你完成了 React 新特性与工程实践的学习！现在你已经具备了构建生产级 React 应用的能力。掌握这些知识后，建议选择一个实际项目来实践所学。可以是一个个人博客、一个待办事项应用、或者参与开源项目。实践是最好的学习方式，祝你在 React 开发的道路上越走越远！
