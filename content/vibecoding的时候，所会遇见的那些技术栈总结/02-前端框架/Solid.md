# Solid.js 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Solid.js 核心原理、核心 API、 SolidStart 全栈框架及在 AI 应用中的性能优势。

---

## 目录

1. [[#Solid.js 概述与核心哲学]]
2. [[#Solid vs 其他框架的本质区别]]
3. [[#核心 API 详解]]
4. [[#Solid 性能优化原理]]
5. [[#SolidStart 全栈框架]]
6. [[#Solid vs React vs Svelte vs Vue 性能对比]]
7. [[#Solid 在 AI 应用中的优势]]
8. [[#实战场景与代码示例]]
9. [[#选型建议]]

---

## Solid.js 概述与核心哲学

### 什么是 Solid.js

Solid.js 是一个**声明式 UI 库**，专注于提供极致的运行时性能。与 React 采用虚拟 DOM 进行 diffing 不同，Solid.js 采用**精细化反应性系统**（Fine-grained Reactivity），直接追踪数据依赖并精确更新 DOM 节点，无需中间层。

**Solid.js 核心特点：**

| 特性 | 说明 |
|------|------|
| **精细化响应性** | 追踪到每个独立的订阅者，而非组件 |
| **零虚拟 DOM** | 直接编译 JSX 到精确的 DOM 操作 |
| **编译时优化** | JSX 编译时生成高效的更新函数 |
| **真正的响应式** | Signal 模式，变化时精确通知订阅者 |
| **极小包体积** | 基准包约 7KB（gzipped），无运行时开销 |

### Solid.js 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| Solid 1.0 | 2021 | 稳定版发布，核心反应性系统成熟 |
| Solid 11 | 2022 | 性能优化，SSR 支持 |
| Solid 1.7 | 2023 | 新渲染器，简化 API |
| **Solid 1.9** | **2024** | **精细化反应性 2.0，更好的 TypeScript 支持** |
| **Solid 2.0** | **2026** | **SolidStart 2.0 稳定版，全栈能力成熟** |

### 核心理念

**Solid.js 的设计哲学：**

1. **性能优先**：不惜一切代价追求运行时性能
2. **简洁直观**：API 设计追求最小化认知负担
3. **真实反应性**：采用真正的响应式编程范式，而非模拟
4. **编译时增强**：JSX 编译器生成最优代码

> [!IMPORTANT]
> Solid.js 不是"更快的 React"，而是一个完全不同范式的框架。理解这一点是掌握 Solid 的关键。

---

## Solid vs 其他框架的本质区别

### 架构对比

**React 的工作方式（虚拟 DOM）：**

```
状态变化 → 组件重新渲染 → 生成新虚拟 DOM → Diff 对比 → 应用差异到真实 DOM
```

**Solid 的工作方式（精细化响应性）：**

```
状态变化 → 精确追踪的订阅者更新 → 直接修改对应的 DOM 节点
```

### 代码层面的本质差异

**React Hooks（组件级重渲染）：**

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  console.log('Component re-renders');

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}

// 每次点击按钮，整个组件重新执行
// 输出 "Component re-renders" 每次都会执行
```

**Solid Signals（精确更新）：**

```jsx
import { createSignal } from 'solid-js';

function Counter() {
  const [count, setCount] = createSignal(0);

  console.log('This runs only once!');

  return (
    <div>
      {/* count() 是函数调用，返回当前值 */}
      <p>Count: {count()}</p>
      {/* setCount 直接更新，不触发组件重新执行 */}
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}

// 点击按钮时，组件不会重新执行
// 只有 <p> 标签的 textContent 更新
// "This runs only once!" 只输出一次
```

### 关键差异总结

| 维度 | React | Solid.js |
|------|-------|----------|
| **重渲染单位** | 组件 | DOM 节点 |
| **更新机制** | 虚拟 DOM diffing | 精确订阅者更新 |
| **状态声明** | useState（函数组件内） | createSignal（模块级或组件内） |
| **组件执行** | 每次状态变化重执行 | 只执行一次 |
| **JSX 编译结果** | React.createElement | 精确 DOM 操作 |
| **记忆化** | useMemo/useCallback（可选） | 自动（通过创建函数追踪） |

### 编译后代码对比

**React（编译后）：**

```javascript
// 简化的 React 输出
function Counter() {
  const [_count, setCount] = useState(0);
  // ...
  return React.createElement('div', null,
    React.createElement('p', null, `Count: ${_count}`),
    React.createElement('button', { onClick: handleClick }, 'Increment')
  );
}

// 每次状态变化：
// 1. Counter 重新执行
// 2. 生成新的虚拟 DOM 树
// 3. 与旧树 diffing
// 4. 应用最小变更
```

**Solid（编译后）：**

```javascript
// 简化的 Solid 输出
function Counter() {
  const [count, setCount] = createSignal(0);

  // DOM 节点创建
  const div = document.createElement('div');
  const p = document.createElement('p');
  const button = document.createElement('button');

  // 精确的订阅关系
  createEffect(() => {
    // 只有 count 变化时执行
    p.textContent = `Count: ${count()}`;
  });

  button.addEventListener('click', () => {
    setCount(c => c + 1);
    // 直接更新 p.textContent，无中间步骤
  });

  div.appendChild(p);
  div.appendChild(button);

  return div;
}
```

---

## 核心 API 详解

### createSignal - 反应性状态

```javascript
import { createSignal } from 'solid-js';

// 基础用法
const [count, setCount] = createSignal(0);

// 读取值（函数调用）
console.log(count()); // 0

// 更新值
setCount(1);           // 直接赋值
setCount(c => c + 1);  // 函数式更新

// 带初始值的 getter
const [name, setName] = createSignal('Alice');
const getName = name; // 获取 getter 函数
console.log(getName()); // 'Alice'
```

### createEffect - 副作用

```javascript
import { createSignal, createEffect } from 'solid-js';

const [count, setCount] = createSignal(0);

// 基本用法：追踪依赖
createEffect(() => {
  console.log('Count changed to:', count());
  // 自动追踪 count 依赖
});

// 清理副作用
createEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);

  // 返回清理函数
  return () => clearInterval(timer);
});

// 多个依赖
const [firstName, setFirstName] = createSignal('John');
const [lastName, setLastName] = createSignal('Doe');

createEffect(() => {
  // 追踪两个依赖
  document.title = `${firstName()} ${lastName()}`;
});
```

### createMemo - 计算属性

```javascript
import { createSignal, createMemo } from 'solid-js';

const [count, setCount] = createSignal(10);
const [multiplier, setMultiplier] = createSignal(2);

// 派生计算
const doubled = createMemo(() => count() * 2);
const result = createMemo(() => {
  return count() * multiplier();
});

console.log(doubled()); // 20
console.log(result());  // 20

setCount(20);
console.log(doubled()); // 自动更新为 40
```

### createResource - 异步数据加载

```javascript
import { createResource, createSignal } from 'solid-js';

const fetchUser = async (id) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

function UserProfile({ userId }) {
  const [user, { mutate, refetch }] = createResource(userId, fetchUser);

  return (
    <Show when={!user.loading} fallback={<p>Loading...</p>}>
      <Show when={user.error} fallback={<p>Error!</p>}>
        <div>
          <h1>{user().name}</h1>
          <p>{user().email}</p>
          <button onClick={refetch}>Refresh</button>
        </div>
      </Show>
    </Show>
  );
}
```

### 控制流组件

**Show 组件：**

```jsx
import { Show } from 'solid-js';

function UserCard({ user }) {
  return (
    <Show when={user} fallback={<p>No user</p>}>
      <div class="card">
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    </Show>
  );
}
```

**For 组件：**

```jsx
import { For } from 'solid-js';

function TodoList({ items }) {
  return (
    <ul>
      <For each={items}>
        {(item, index) => (
          <li data-index={index()}>
            {item.text}
          </li>
        )}
      </For>
    </ul>
  );
}
```

**Switch/Match 组件：**

```jsx
import { Switch, Match } from 'solid-js';

function StatusBadge({ status }) {
  return (
    <Switch>
      <Match when={status === 'success'}>
        <span class="badge success">Success</span>
      </Match>
      <Match when={status === 'error'}>
        <span class="badge error">Error</span>
      </Match>
      <Match when={status === 'loading'}>
        <span class="badge loading">Loading...</span>
      </Match>
    </Switch>
  );
}
```

### Context - 跨组件状态共享

```javascript
// ThemeContext.jsx
import { createContext, useContext } from 'solid-js';

export const ThemeContext = createContext();

export function ThemeProvider(props) {
  const [theme, setTheme] = createSignal('light');

  const toggleTheme = () => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };

  const value = {
    theme,
    toggleTheme
  };

  return (
    <ThemeContext.Provider value={value}>
      {props.children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

### createStore - 嵌套状态

```javascript
import { createStore } from 'solid-js/store';

function App() {
  const [store, setStore] = createStore({
    user: {
      name: 'Alice',
      preferences: {
        theme: 'dark',
        language: 'en'
      }
    },
    todos: []
  });

  // 更新嵌套属性
  const updateName = () => {
    setStore('user', 'name', 'Bob');
  };

  const updateNested = () => {
    setStore('user', 'preferences', 'theme', 'light');
  };

  // 添加数组元素
  const addTodo = (text) => {
    setStore('todos', (todos) => [...todos, { text, done: false }]);
  };

  return (
    <div>
      <p>Name: {store.user.name}</p>
      <p>Theme: {store.user.preferences.theme}</p>
      <For each={store.todos}>
        {(todo) => <p>{todo.text}</p>}
      </For>
    </div>
  );
}
```

---

## Solid 性能优化原理

### 精细化反应性系统

Solid 的反应性系统基于以下核心概念：

**1. Signal（信号）：**

```javascript
// Signal 是最基本的反应性单元
const [value, setValue] = createSignal(initialValue);

// 读取 value() 时，自动建立订阅关系
// setValue 触发时，所有订阅者精确更新
```

**2. Owner（拥有者）：**

```javascript
// 每个反应性上下文都有一个 Owner
// Owner 追踪所有派生的订阅关系
createEffect(() => {
  // 这个 effect 属于当前 Owner
  console.log(count());
});
```

**3. batch（批量更新）：**

```javascript
import { batch } from 'solid-js';

function handleClick() {
  batch(() => {
    setFirstName('John');
    setLastName('Doe');
    setAge(30);
    // 所有更新只在 DOM 应用一次
  });
}
```

### 与 React 的性能对比

| 场景 | React 18 | Solid.js | 差异说明 |
|------|----------|----------|---------|
| **状态更新** | 组件重渲染 + diffing | 精确节点更新 | Solid 避免中间步骤 |
| **列表渲染** | 整体 diffing | 节点级更新 | Solid 只更新变化的节点 |
| **Context 更新** | 所有消费者重渲染 | 仅实际订阅者更新 | Solid 精确追踪 |
| **记忆化** | 需要 useMemo/useCallback | 自动 | Solid 通过创建函数追踪 |

### 性能实测数据

> [!NOTE]
> 以下数据来自 js-framework-benchmark 2026 年测试。

| 测试场景 | Solid 1.9 | Solid 2.0 | React 18 | Svelte 5 |
|---------|-----------|-----------|----------|----------|
| 创建 1000 行 | 1.0ms | 0.9ms | 2.3ms | 1.2ms |
| 部分更新 1000 行 | 0.7ms | 0.6ms | 1.8ms | 0.9ms |
| 选择性更新 10 行 | 0.5ms | 0.4ms | 1.2ms | 0.7ms |
| 内存占用（1000行） | 8MB | 7MB | 15MB | 6MB |

---

## SolidStart 全栈框架

### SolidStart 概述

SolidStart 是 Solid.js 的官方全栈框架，提供文件系统路由、SSR、API 端点等功能。

### 项目结构

```
my-app/
├── src/
│   ├── routes/
│   │   ├── index.tsx           # 首页 /
│   │   ├── about.tsx           # 关于页 /about
│   │   ├── users/
│   │   │   ├── index.tsx       # /users
│   │   │   └── [id].tsx         # 动态路由 /users/:id
│   │   └── api/
│   │       └── users.ts        # API 端点
│   ├── components/
│   └── entry-client.tsx
├── app.config.ts
└── tsconfig.json
```

### 路由系统

**基本路由：**

```tsx
// src/routes/index.tsx
export default function Home() {
  return <h1>Welcome</h1>;
}
```

**动态路由：**

```tsx
// src/routes/users/[id].tsx
import { useParams } from '@solidjs/router';

export default function UserProfile() {
  const params = useParams();
  // params.id 获取 URL 参数

  return <h1>User ID: {params.id}</h1>;
}
```

**嵌套路由：**

```tsx
// src/routes/dashboard/layout.tsx
import { Outlet } from '@solidjs/router';

export default function DashboardLayout() {
  return (
    <div class="dashboard">
      <nav>Dashboard Navigation</nav>
      <main>
        <Outlet /> {/* 子路由渲染位置 */}
      </main>
    </div>
  );
}
```

### 数据加载

**服务端数据加载：**

```tsx
// src/routes/users/[id].tsx
import { createAsync } from '@solidjs/router';
import { cache } from '@solidjs/router';

const getUser = cache(async (id: string) => {
  'use server';
  const response = await fetch(`https://api.example.com/users/${id}`);
  return response.json();
}, 'getUser');

export default function UserProfile(props) {
  const user = createAsync(() => getUser(props.params.id));

  return (
    <Show when={user()} fallback={<p>Loading...</p>}>
      <h1>{user().name}</h1>
    </Show>
  );
}
```

**客户端数据加载：**

```tsx
import { createResource } from 'solid-js';

function SearchResults({ query }) {
  const [results] = createResource(
    () => query,
    async (query) => {
      const response = await fetch(`/api/search?q=${query}`);
      return response.json();
    }
  );

  return (
    <Show when={!results.loading} fallback={<p>Loading...</p>}>
      <ul>
        <For each={results()}>
          {(result) => <li>{result.title}</li>}
        </For>
      </ul>
    </Show>
  );
}
```

### API 端点

```typescript
// src/routes/api/users.ts
import { json } from '@solidjs/router';

export async function GET(event) {
  const users = await db.users.findMany();
  return json(users);
}

export async function POST(event) {
  const body = await event.request.json();
  const user = await db.users.create(body);
  return json(user, { status: 201 });
}
```

### 表单处理

```tsx
import { action, useAction } from '@solidjs/router';
import { z } from 'zod';

const schema = z.object({
  name: z.string().min(1),
  email: z.string().email()
});

const submitForm = action(async (formData) => {
  'use server';

  const result = schema.safeParse(Object.fromEntries(formData));
  if (!result.success) {
    return { error: result.error.flatten() };
  }

  await db.users.create(result.data);
  revalidatePath('/users');
  return { success: true };
}, 'submitForm');

export default function CreateUser() {
  const submit = useAction(submitForm);

  return (
    <form action={submit} method="post">
      <input name="name" type="text" required />
      <input name="email" type="email" required />
      <button type="submit">Create</button>
    </form>
  );
}
```

---

## Solid vs React vs Svelte vs Vue 性能对比

### 核心性能指标对比

| 指标 | Solid.js | React 18 | Svelte 5 | Vue 3 |
|------|----------|----------|----------|-------|
| **包体积** | ~7KB | ~45KB | ~4KB | ~35KB |
| **首屏渲染** | 最快 | 中等 | 最快 | 中等 |
| **更新延迟** | 最低 | 中等 | 低 | 中等 |
| **内存占用** | 低 | 较高 | 最低 | 中等 |
| **大列表渲染** | 优秀 | 需优化 | 优秀 | 优秀 |

### 性能对比表

| 测试场景 | Solid | React | Svelte | Vue | 说明 |
|---------|-------|-------|--------|-----|------|
| **初始渲染** | ★★★★★ | ★★★☆☆ | ★★★★★ | ★★★★☆ | Solid/Svelte 直接操作 DOM |
| **状态更新** | ★★★★★ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | Solid 精确更新，React diffing |
| **列表操作** | ★★★★★ | ★★★☆☆ | ★★★★☆ | ★★★★☆ | Solid 按需更新 |
| **内存效率** | ★★★★☆ | ★★☆☆☆ | ★★★★★ | ★★★☆☆ | Svelte 零运行时 |
| **SSR** | ★★★★☆ | ★★★★★ | ★★★★☆ | ★★★★★ | Next.js/Nuxt 生态更成熟 |

### API 对比表

| 功能 | Solid | React | Vue | 说明 |
|------|-------|-------|-----|------|
| **状态管理** | createSignal | useState | ref | Solid 最灵活 |
| **计算属性** | createMemo | useMemo | computed | 功能类似 |
| **副作用** | createEffect | useEffect | watchEffect | 语义有差异 |
| **组件** | 函数组件 | 函数组件 | Options/Script Setup | 编译策略不同 |
| **条件渲染** | <Show> | 三元/&& | v-if | Solid 更声明式 |
| **列表渲染** | <For> | map | v-for | Solid 自动追踪 |

### 场景适用性对比

| 场景 | 首选 | 备选 |
|------|------|------|
| **高频更新 UI** | Solid / Svelte | React（需优化） |
| **AI 应用** | Solid / Svelte | Next.js |
| **企业应用** | React | Vue / SvelteKit |
| **内容网站** | SvelteKit | Next.js / Nuxt |
| **游戏 UI** | Solid | Canvas/原生 |
| **仪表盘** | Solid / Vue | React |
| **移动端 Web** | Solid | React Native |

---

## Solid 在 AI 应用中的优势

### 为什么 AI 应用需要 Solid

1. **高频状态更新**：AI 应用（聊天、流式响应）需要频繁的状态更新，Solid 的精确更新避免不必要的渲染
2. **流式数据处理**：流式 API 需要高效的增量更新，Solid 的反应性系统天然适配
3. **包体积敏感**：AI 应用需要加载 SDK/API 客户端，包体积优化至关重要
4. **交互响应**：AI 生成的内容需要即时渲染，Solid 提供最低延迟

### Solid + AI 应用示例

**1. AI 聊天界面：**

```tsx
import { createSignal, createEffect, For } from 'solid-js';
import { createStore } from 'solid-js/store';

interface Message {
  id: string;
  role: 'user' | 'assistant';
  content: string;
}

function AIChat() {
  const [input, setInput] = createSignal('');
  const [messages, setMessages] = createStore<Message[]>([]);
  const [isLoading, setIsLoading] = createSignal(false);

  async function sendMessage() {
    const userMessage = input();
    if (!userMessage) return;

    // 添加用户消息
    setMessages([
      ...messages,
      { id: crypto.randomUUID(), role: 'user', content: userMessage }
    ]);
    setInput('');
    setIsLoading(true);

    // 添加占位消息
    const assistantId = crypto.randomUUID();
    setMessages([
      ...messages,
      { id: assistantId, role: 'assistant', content: '' }
    ]);

    // 流式响应
    const response = await fetch('/api/chat', {
      method: 'POST',
      body: JSON.stringify({ messages: messages }),
      headers: { 'Content-Type': 'application/json' }
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value);
      // 精确更新最后一条消息
      setMessages(
        messages.length - 1,
        'content',
        (prev) => prev + chunk
      );
    }

    setIsLoading(false);
  }

  return (
    <div class="chat-container">
      <div class="messages">
        <For each={messages}>
          {(message) => (
            <div class={`message ${message.role}`}>
              <div class="avatar">
                {message.role === 'user' ? '👤' : '🤖'}
              </div>
              <div class="content">
                {message.content}
              </div>
            </div>
          )}
        </For>
      </div>

      <form class="input-area" onSubmit={(e) => {
        e.preventDefault();
        sendMessage();
      }}>
        <input
          type="text"
          value={input()}
          onInput={(e) => setInput(e.currentTarget.value)}
          placeholder="Ask AI..."
        />
        <button type="submit" disabled={isLoading()}>
          {isLoading() ? 'Sending...' : 'Send'}
        </button>
      </form>
    </div>
  );
}
```

**2. 图像生成界面：**

```tsx
import { createSignal, Show } from 'solid-js';

function ImageGenerator() {
  const [prompt, setPrompt] = createSignal('');
  const [imageUrl, setImageUrl] = createSignal<string | null>(null);
  const [loading, setLoading] = createSignal(false);
  const [progress, setProgress] = createSignal(0);

  async function generate() {
    setLoading(true);
    setProgress(0);

    // 模拟进度更新
    const progressInterval = setInterval(() => {
      setProgress(p => Math.min(p + 10, 90));
    }, 500);

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        body: JSON.stringify({ prompt: prompt() }),
        headers: { 'Content-Type': 'application/json' }
      });

      const data = await response.json();
      setImageUrl(data.url);
      setProgress(100);
    } finally {
      clearInterval(progressInterval);
      setLoading(false);
    }
  }

  return (
    <div class="generator">
      <textarea
        value={prompt()}
        onInput={(e) => setPrompt(e.currentTarget.value)}
        placeholder="Describe your image..."
        rows={4}
      />

      <Show when={loading()}>
        <div class="progress">
          <div class="progress-bar" style={{ width: `${progress()}%` }} />
        </div>
        <p>Generating... {progress()}%</p>
      </Show>

      <button onClick={generate} disabled={loading() || !prompt()}>
        Generate Image
      </button>

      <Show when={imageUrl()}>
        <div class="result">
          <img src={imageUrl()} alt="Generated" />
        </div>
      </Show>
    </div>
  );
}
```

**3. 多模态 AI 助手：**

```tsx
import { createSignal, Show, For } from 'solid-js';

function MultimodalAI() {
  const [mode, setMode] = createSignal<'chat' | 'image' | 'code'>('chat');
  const [input, setInput] = createSignal('');
  const [response, setResponse] = createSignal('');

  async function submit() {
    const result = await ai.process({
      mode: mode(),
      input: input(),
      context: conversationHistory()
    });

    setResponse(result);
  }

  return (
    <div class="ai-assistant">
      <div class="mode-selector">
        <button
          class={mode() === 'chat' ? 'active' : ''}
          onClick={() => setMode('chat')}
        >
          💬 Chat
        </button>
        <button
          class={mode() === 'image' ? 'active' : ''}
          onClick={() => setMode('image')}
        >
          🎨 Image
        </button>
        <button
          class={mode() === 'code' ? 'active' : ''}
          onClick={() => setMode('code')}
        >
          💻 Code
        </button>
      </div>

      <Show when={mode() === 'chat'}>
        <textarea
          value={input()}
          onInput={(e) => setInput(e.currentTarget.value)}
          placeholder="Ask anything..."
        />
      </Show>

      <Show when={mode() === 'image'}>
        <textarea
          value={input()}
          onInput={(e) => setInput(e.currentTarget.value)}
          placeholder="Describe the image you want to generate..."
        />
      </Show>

      <Show when={mode() === 'code'}>
        <div class="code-input">
          <select>
            <option value="python">Python</option>
            <option value="javascript">JavaScript</option>
            <option value="typescript">TypeScript</option>
          </select>
          <textarea
            value={input()}
            onInput={(e) => setInput(e.currentTarget.value)}
            placeholder="Paste your code or describe what you need..."
          />
        </div>
      </Show>

      <button onClick={submit} disabled={!input()}>
        {mode() === 'chat' ? 'Ask' : mode() === 'image' ? 'Generate' : 'Submit'}
      </button>

      <Show when={response()}>
        <div class="response">
          {response()}
        </div>
      </Show>
    </div>
  );
}
```

---

## 实战场景与代码示例

### 实时协作编辑器

```tsx
import { createSignal, createEffect, onCleanup } from 'solid-js';
import { createStore } from 'solid-js/store';

interface Cursor {
  userId: string;
  name: string;
  color: string;
  position: { x: number; y: number };
}

interface EditorState {
  content: string;
  cursors: Cursor[];
}

function CollaborativeEditor({ documentId }) {
  const [state, setState] = createStore<EditorState>({
    content: '',
    cursors: []
  });

  let ws: WebSocket;

  createEffect(() => {
    // 建立 WebSocket 连接
    ws = new WebSocket(`wss://api.example.com/doc/${documentId}`);

    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);

      if (update.type === 'content') {
        setState('content', update.content);
      } else if (update.type === 'cursor') {
        setState('cursors', (cursors) => {
          const index = cursors.findIndex(c => c.userId === update.cursor.userId);
          if (index >= 0) {
            return [...cursors.slice(0, index), update.cursor, ...cursors.slice(index + 1)];
          }
          return [...cursors, update.cursor];
        });
      }
    };

    // 发送内容更新
    const sendUpdate = (content: string) => {
      ws.send(JSON.stringify({ type: 'content', content }));
    };

    // 发送光标位置
    const sendCursor = (position: { x: number; y: number }) => {
      ws.send(JSON.stringify({ type: 'cursor', position }));
    };
  });

  onCleanup(() => {
    ws?.close();
  });

  return (
    <div class="editor">
      <textarea
        value={state.content}
        onInput={(e) => {
          setState('content', e.currentTarget.value);
          ws?.send(JSON.stringify({ type: 'content', content: e.currentTarget.value }));
        }}
      />

      {/* 渲染其他用户的光标 */}
      <For each={state.cursors}>
        {(cursor) => (
          <div
            class="remote-cursor"
            style={{
              left: `${cursor.position.x}px`,
              top: `${cursor.position.y}px`,
              '--cursor-color': cursor.color
            }}
          >
            <div class="cursor-pointer" />
            <div class="cursor-label">{cursor.name}</div>
          </div>
        )}
      </For>
    </div>
  );
}
```

### 数据可视化仪表盘

```tsx
import { createSignal, createEffect, onMount, onCleanup } from 'solid-js';
import { For, Show } from 'solid-js';

interface Metric {
  id: string;
  name: string;
  value: number;
  change: number;
  history: number[];
}

function MetricsDashboard() {
  const [metrics, setMetrics] = createSignal<Metric[]>([]);
  const [selectedMetric, setSelectedMetric] = createSignal<string | null>(null);

  let ws: WebSocket;

  onMount(() => {
    // 模拟 WebSocket 实时数据
    ws = new WebSocket('wss://api.example.com/metrics/stream');

    ws.onmessage = (event) => {
      const update = JSON.parse(event.data);

      setMetrics((prev) =>
        prev.map((m) =>
          m.id === update.id
            ? {
                ...m,
                value: update.value,
                change: update.change,
                history: [...m.history.slice(-59), update.value]
              }
            : m
        )
      );
    };
  });

  onCleanup(() => {
    ws?.close();
  });

  return (
    <div class="dashboard">
      <div class="metrics-grid">
        <For each={metrics()}>
          {(metric) => (
            <div
              class="metric-card"
              classList={{ selected: selectedMetric() === metric.id }}
              onClick={() => setSelectedMetric(metric.id)}
            >
              <h3>{metric.name}</h3>
              <div class="value">{metric.value.toFixed(2)}</div>
              <div class="change" classList={{ positive: metric.change > 0 }}>
                {metric.change > 0 ? '+' : ''}{metric.change.toFixed(2)}%
              </div>
            </div>
          )}
        </For>
      </div>

      <Show when={selectedMetric()}>
        <div class="chart-container">
          <LineChart
            data={metrics().find(m => m.id === selectedMetric())?.history || []}
          />
        </div>
      </Show>
    </div>
  );
}
```

---

## 选型建议

### 何时选择 Solid.js

| 场景 | 推荐理由 |
|------|---------|
| **高频更新 UI** | 精细化反应性提供最低更新延迟 |
| **AI 应用** | 流式响应、大列表渲染性能优异 |
| **性能敏感项目** | 包体积小，运行时开销接近零 |
| **团队有 React 背景** | 上手成本低，JSX 语法相近 |
| **追求极致性能** | Solid 是目前性能最优的前端框架之一 |

### 何时考虑其他框架

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| **需要丰富生态** | React | 组件库、工具链最完善 |
| **快速原型开发** | Svelte | 语法最简洁，开发体验好 |
| **SSR 优先** | Next.js / Nuxt | SSR 生态成熟 |
| **团队 Vue 背景** | Vue 3 | 学习成本最低 |

### 技术选型决策树

```
项目需求评估
    │
    ├── 性能要求极高？
    │   ├── 是 → 精细化更新需求？
    │   │   ├── 是 → Solid.js（精确更新）
    │   │   └── 否 → Svelte 5（零运行时）
    │   └── 否 → 继续分析
    │
    ├── 团队技术栈？
    │   ├── React 背景 → Solid.js 或 React
    │   ├── Vue 背景 → Vue 3
    │   └── 追求简洁 → Svelte 5
    │
    ├── 生态需求？
    │   ├── 高 → React 18（生态最丰富）
    │   └── 中等 → Solid / Svelte
    │
    └── AI 应用？
        ├── 是 → Solid.js 或 Svelte 5
        └── 否 → 根据团队选择
```

> [!TIP]
> Solid.js 的学习曲线比 React 略陡，但性能优势明显。对于追求极致用户体验的 AI 应用，Solid.js 是一个值得认真考虑的选择。

---

> [!SUCCESS]
> Solid.js 以精细化反应性系统重新定义了前端框架的性能标准。其零虚拟 DOM、精确更新的架构使其成为 AI 应用和高性能 UI 的理想选择。虽然生态相对年轻，但其核心优势——极致性能和极小包体积——使其在 2026 年的前端框架竞争中占据重要地位。

---

## 核心理念与设计哲学

### Solid.js 的设计哲学

Solid.js 的设计哲学建立在"精细化反应性"和"零虚拟 DOM"两个核心原则之上。

**1. 精细化反应性系统**

Solid.js 的反应性系统追踪到每个独立的订阅者，而非组件：

```
┌─────────────────────────────────────────────────────────────┐
│               Solid.js 精细化反应性原理                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  传统框架（组件级重渲染）：                                   │
│  ┌─────────────────┐                                       │
│  │ Component       │                                       │
│  │ ├─ <div>       │ ← 任何状态变化                         │
│  │ ├─ <span>       │ ← 触发整个组件                         │
│  │ └─ <button>     │ ← 重渲染                              │
│  └─────────────────┘                                       │
│                                                             │
│  Solid（节点级更新）：                                       │
│  ┌─────────────────┐                                       │
│  │ Component       │                                       │
│  │ ├─ <div> ← 只更新这里    │                              │
│  │ ├─ <span> ← 只更新这里    │                              │
│  │ └─ <button> ← 只更新这里    │                              │
│  └─────────────────┘                                       │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────────────────────────────────────────┐      │
│  │  Signal 追踪每个独立订阅者                         │      │
│  │  count() → <div> 更新                            │      │
│  │  name() → <span> 更新                            │      │
│  └─────────────────────────────────────────────────┘      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**2. 零虚拟 DOM**

Solid.js 直接编译 JSX 到精确的 DOM 操作：

```tsx
// 输入：Solid JSX
function Counter() {
  const [count, setCount] = createSignal(0);
  
  return (
    <div>
      <p>Count: {count()}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}

// 编译后：精确的 DOM 操作
function Counter() {
  const [count, setCount] = createSignal(0);
  
  // 创建 DOM 节点
  const div = document.createElement('div');
  const p = document.createElement('p');
  const button = document.createElement('button');
  
  // 精确订阅
  createEffect(() => {
    p.textContent = `Count: ${count()}`;
  });
  
  button.addEventListener('click', () => {
    setCount(c => c + 1);
    // 直接更新 p.textContent
  });
  
  div.appendChild(p);
  div.appendChild(button);
  
  return div;
}
```

**3. 真正的响应式**

Solid.js 采用真正的响应式编程范式，而非模拟：

```tsx
import { createSignal, createEffect, createMemo } from 'solid-js';

// Signal - 响应式状态
const [count, setCount] = createSignal(0);

// 读取值（函数调用）
console.log(count()); // 0

// 更新值
setCount(1);
setCount(c => c + 1);

// Memo - 派生计算（自动追踪依赖）
const doubled = createMemo(() => count() * 2);
console.log(doubled()); // 2

// Effect - 副作用（自动追踪依赖）
createEffect(() => {
  console.log('Count changed:', count());
});

// 自动追踪：count() 调用建立依赖
// count 变化时，effect 自动重新执行
```

**4. 编译时优化**

Solid.js 的 JSX 编译器在构建时生成最优代码：

| 优化项 | 说明 |
|--------|------|
| **精确更新** | 编译器生成精确的 DOM 更新代码 |
| **死代码消除** | 未使用的代码在编译时移除 |
| **常量提升** | 静态内容在编译时优化 |
| **代码分割** | 按需加载代码 |

### Solid.js 解决的问题域

| 问题域 | Solid.js 解决方案 | 优势 |
|--------|-----------------|------|
| **高频更新** | 精细化反应性 | 最低更新延迟 |
| **大列表渲染** | 按需更新 | 流畅滚动 |
| **包体积** | 极小运行时 | 快速加载 |
| **AI 应用** | 精确更新 | 流式响应流畅 |
| **数据可视化** | 高性能渲染 | 实时图表 |

---

## 完整安装与项目创建

### 环境准备

**Node.js 版本要求：**
- Solid.js 需要 Node.js 16 或更高版本
- 推荐使用 Node.js 20 LTS

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 安装 pnpm
npm install -g pnpm
```

### 创建 Solid 项目的多种方式

**方式一： Vite + Solid（推荐）**

```bash
# 创建 Solid + TypeScript 项目
npm create vite@latest my-solid-app -- --template solid-ts

# 进入目录
cd my-solid-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview
```

**方式二：SolidStart（全栈框架）**

```bash
# 创建 SolidStart 项目
npm create solid@latest

# 选项：
# ▸ App (SolidStart)
# ▸ Library (Solid JS)
# ▸ Package (SolidStart package)

# 进入目录
cd my-solid-app

# 安装依赖
pnpm install

# 启动
pnpm dev
```

**方式三：Degit**

```bash
# 使用 degit 克隆模板
npx degit solidjs/templates/ts my-solid-app

cd my-solid-app
pnpm install
pnpm dev
```

**方式四：在线体验**

- Solid Playground：https://playground.solidjs.com
- StackBlitz：https://stackblitz.com/fork/solid

### 项目结构详解

**Vite + Solid 项目结构：**

```
my-solid-app/
├── src/
│   ├── components/        # 组件
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   └── UserCard.tsx
│   ├── stores/             # 状态管理
│   │   └── userStore.ts
│   ├── pages/             # 页面组件
│   │   ├── Home.tsx
│   │   └── Users.tsx
│   ├── services/           # API 服务
│   │   └── api.ts
│   ├── types/              # 类型定义
│   │   └── index.ts
│   ├── utils/             # 工具函数
│   │   └── format.ts
│   ├── App.tsx           # 根组件
│   ├── index.tsx         # 入口文件
│   └── index.css         # 全局样式
├── public/                # 静态资源
├── index.html             # HTML 入口
├── package.json
├── tsconfig.json
└── vite.config.ts        # Vite 配置
```

**SolidStart 项目结构：**

```
my-solid-app/
├── src/
│   ├── routes/            # 文件系统路由
│   │   ├── index.tsx     # /
│   │   ├── about.tsx     # /about
│   │   └── users/
│   │       ├── index.tsx # /users
│   │       └── [id].tsx  # /users/:id
│   ├── components/       # 组件
│   ├── lib/              # 库代码
│   ├── entry-client.tsx  # 客户端入口
│   ├── entry-server.tsx  # 服务端入口
│   └── app.tsx           # 根组件
├── app.config.ts          # SolidStart 配置
├── package.json
└── tsconfig.json
```

---

## 核心 API 详解

### createSignal - 反应性状态

**基础用法：**

```tsx
import { createSignal, Accessor, Setter } from 'solid-js';

// 基础用法
const [count, setCount] = createSignal(0);

// 读取值（函数调用）
console.log(count()); // 0

// 更新值
setCount(1);              // 直接赋值
setCount(c => c + 1);     // 函数式更新

// 泛型支持
const [name, setName] = createSignal<string>('Alice');

// 初始值 + 工厂函数
const [items, setItems] = createSignal<Item[]>(() => {
  const saved = localStorage.getItem('items');
  return saved ? JSON.parse(saved) : [];
});

// 访问器和设置器类型
type CountAccessor = Accessor<number>;
type CountSetter = Setter<number>;

const useCounter = (): [CountAccessor, CountSetter] => {
  return createSignal(0);
};
```

**高级用法：**

```tsx
// 比较函数（用于判断是否更新）
const [value, setValue] = createSignal(0, {
  equals: (prev, next) => Math.abs(prev - next) < 1
});

// 自定义存储
import { createStore } from 'solid-js/store';

const [store, setStore] = createStore({
  count: 0,
  name: 'Alice'
});

setStore('count', c => c + 1);
setStore('name', 'Bob');
```

### createEffect - 副作用

**基础用法：**

```tsx
import { createSignal, createEffect } from 'solid-js';

const [count, setCount] = createSignal(0);
const [name, setName] = createSignal('Alice');

// 基本副作用
createEffect(() => {
  console.log('Count:', count());
  // 自动追踪 count 依赖
});

// 多个依赖
createEffect(() => {
  console.log(`${name()} changed to ${count()}`);
  // 追踪 name 和 count
});

// 清理副作用
createEffect(() => {
  const interval = setInterval(() => {
    console.log('Tick:', count());
  }, 1000);
  
  // 返回清理函数
  return () => clearInterval(interval);
});
```

**追踪控制：**

```tsx
import { createEffect, untrack } from 'solid-js';

const [a, setA] = createSignal(1);
const [b, setB] = createSignal(2);

createEffect(() => {
  // 追踪 a
  console.log('a:', a());
  
  // 不追踪 b（使用 untrack）
  console.log('b:', untrack(b));
});
```

### createMemo - 计算属性

**基础用法：**

```tsx
import { createSignal, createMemo } from 'solid-js';

const [count, setCount] = createSignal(10);
const [multiplier, setMultiplier] = createSignal(2);

// 派生计算
const doubled = createMemo(() => count() * 2);

// 复杂派生
const status = createMemo(() => {
  const c = count();
  if (c < 0) return 'negative';
  if (c === 0) return 'zero';
  return 'positive';
});

// 依赖链
const quadruple = createMemo(() => doubled() * 2);
```

**性能优化：**

```tsx
// 使用 equals 选项优化
const [data, setData] = createSignal<Data[]>([], {
  equals: (prev, next) => {
    // 自定义比较逻辑
    return prev.length === next.length && 
           prev.every((p, i) => p.id === next[i].id);
  }
});
```

### createResource - 异步数据加载

**基础用法：**

```tsx
import { createResource, createSignal } from 'solid-js';

// 基础异步加载
const fetchUser = async (id: string) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
};

function UserProfile({ userId }) {
  const [user, { mutate, refetch }] = createResource(
    () => userId,  // 源信号
    fetchUser      // fetcher 函数
  );
  
  return (
    <Show 
      when={!user.loading && !user.error} 
      fallback={<p>Loading...</p>}
    >
      <div>
        <h1>{user().name}</h1>
        <p>{user().email}</p>
        <button onClick={refetch}>Refresh</button>
      </div>
    </Show>
  );
}
```

**高级用法：**

```tsx
import { createResource, createSignal } from 'solid-js';

// 并行加载多个资源
const [userId, setUserId] = createSignal('1');

const [user] = createResource(userId, async (id) => {
  const [userRes, postsRes] = await Promise.all([
    fetch(`/api/users/${id}`),
    fetch(`/api/users/${id}/posts`)
  ]);
  
  const [user, posts] = await Promise.all([
    userRes.json(),
    postsRes.json()
  ]);
  
  return { user, posts };
});

// 错误处理
const [data, { mutate, refetch }] = createResource(
  userId,
  async (id, { signal, previousValue }) => {
    // previousValue 可用于缓存
    const response = await fetch(`/api/users/${id}`, { signal });
    if (!response.ok) throw new Error('Failed to fetch');
    return response.json();
  }
);
```

### createStore - 嵌套状态

**基础用法：**

```tsx
import { createStore } from 'solid-js/store';

function App() {
  const [store, setStore] = createStore({
    user: {
      name: 'Alice',
      age: 30
    },
    todos: [] as string[],
    settings: {
      theme: 'light',
      notifications: true
    }
  });
  
  // 更新嵌套属性
  const updateName = () => {
    setStore('user', 'name', 'Bob');
  };
  
  const addTodo = (text: string) => {
    setStore('todos', todos => [...todos, text]);
  };
  
  const toggleNotifications = () => {
    setStore('settings', 'notifications', n => !n);
  };
  
  return (
    <div>
      <p>{store.user.name}</p>
      <For each={store.todos}>{todo => <li>{todo}</li>}</For>
    </div>
  );
}
```

**Immutable 更新：**

```tsx
import { produce } from 'solid-js/store';

const [store, setStore] = createStore({
  items: [{ id: 1, text: 'Item 1' }]
});

// 使用 produce 进行 immutable 更新
setStore(
  'items',
  produce((items) => {
    items.push({ id: 2, text: 'Item 2' });
  })
);
```

---

## 组件系统详解

### 组件基础

**函数组件：**

```tsx
import { Component, JSX } from 'solid-js';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: () => void;
  children: JSX.Element;
}

const Button: Component<ButtonProps> = (props) => {
  return (
    <button
      class={`btn btn-${props.variant || 'primary'}`}
      disabled={props.disabled}
      onClick={props.onClick}
    >
      {props.children}
    </button>
  );
};

// 使用
<Button variant="secondary" onClick={() => console.log('clicked')}>
  Click me
</Button>
```

**Props 解构：**

```tsx
import { ParentProps, JSX } from 'solid-js';

// 使用 JSX.Element 代替 children
function Card(props: ParentProps) {
  return <div class="card">{props.children}</div>;
}

// 解构 children
function Layout(props: ParentProps<{ title: string }>) {
  return (
    <div>
      <h1>{props.title}</h1>
      {props.children}
    </div>
  );
}
```

### 控制流组件

**Show 组件：**

```tsx
import { Show, For } from 'solid-js';

function UserCard({ user }) {
  return (
    <Show
      when={user}
      fallback={<div>No user</div>}
    >
      <div class="card">
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    </Show>
  );
}

// Show 中的 else（使用条件）
function Greeting({ name }) {
  return (
    <Show when={name} fallback={<span>Hello, Guest!</span>}>
      <span>Hello, {name}!</span>
    </Show>
  );
}
```

**For 组件：**

```tsx
import { For, Index } from 'solid-js';

function TodoList({ items }) {
  return (
    <ul>
      <For each={items}>
        {(item, index) => (
          <li data-index={index()}>
            {item.text}
          </li>
        )}
      </For>
    </ul>
  );
}

// Index 用于需要索引的场景
function Matrix() {
  const [data, setData] = createSignal([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
  ]);
  
  return (
    <For each={data()}>
      {(row, rowIndex) => (
        <div class="row">
          <Index each={row}>
            {(cell, colIndex) => (
              <span
                onClick={() => {
                  // 可以直接更新
                  setData(
                    produce(d => {
                      d[rowIndex()][colIndex()] *= 2;
                    })
                  );
                }}
              >
                {cell()}
              </span>
            )}
          </Index>
        </div>
      )}
    </For>
  );
}
```

**Switch/Match 组件：**

```tsx
import { Switch, Match } from 'solid-js';

function StatusBadge({ status }) {
  return (
    <Switch>
      <Match when={status === 'success'}>
        <span class="badge success">成功</span>
      </Match>
      <Match when={status === 'error'}>
        <span class="badge error">错误</span>
      </Match>
      <Match when={status === 'loading'}>
        <span class="badge loading">加载中...</span>
      </Match>
      <Match when={status === 'warning'}>
        <span class="badge warning">警告</span>
      </Match>
    </Switch>
  );
}
```

**动态组件：**

```tsx
import { Dynamic } from 'solid-js/web';

function DynamicTag({ tag, ...props }) {
  return <Dynamic component={tag} {...props} />;
}

// 使用
<DynamicTag tag="button" onClick={() => console.log('click')}>
  Click
</DynamicTag>

<DynamicTag tag="a" href="https://example.com">
  Link
</DynamicTag>
```

### 事件处理

**DOM 事件：**

```tsx
import { createSignal } from 'solid-js';

function EventDemo() {
  const [value, setValue] = createSignal('');
  
  return (
    <div>
      {/* 输入事件 */}
      <input
        value={value()}
        onInput={(e) => setValue(e.currentTarget.value)}
        onChange={(e) => console.log('changed:', e.currentTarget.value)}
      />
      
      {/* 点击事件 */}
      <button
        onClick={() => console.log('clicked')}
        onContextMenu={(e) => {
          e.preventDefault();
          console.log('right clicked');
        }}
      >
        Click
      </button>
      
      {/* 键盘事件 */}
      <input
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            console.log('Enter pressed');
          }
        }}
      />
      
      {/* 表单提交 */}
      <form
        onSubmit={(e) => {
          e.preventDefault();
          console.log('submitted');
        }}
      >
        <button type="submit">Submit</button>
      </form>
    </div>
  );
}
```

### Refs

**DOM 引用：**

```tsx
import { createSignal, onMount } from 'solid-js';

function FocusInput() {
  let inputRef: HTMLInputElement | undefined;
  
  onMount(() => {
    inputRef?.focus();
  });
  
  return <input ref={inputRef} />;
}

// 使用 createRef
function Canvas() {
  let canvasRef: HTMLCanvasElement | undefined;
  
  onMount(() => {
    if (canvasRef) {
      const ctx = canvasRef.getContext('2d');
      ctx?.fillRect(0, 0, 100, 100);
    }
  });
  
  return <canvas ref={canvasRef} width={200} height={200} />;
}
```

---

## 路由系统详解

### @solidjs/router

**基础配置：**

```tsx
import { Router, Route } from '@solidjs/router';
import { lazy } from 'solid-js';

// 懒加载页面组件
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Users = lazy(() => import('./pages/Users'));

function App() {
  return (
    <Router>
      <Route path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/users" component={Users} />
    </Router>
  );
}

render(() => <App />, document.getElementById('root')!);
```

**嵌套路由：**

```tsx
import { Router, Route, A, Outlet } from '@solidjs/router';

function DashboardLayout() {
  return (
    <div>
      <nav>
        <A href="/dashboard">概览</A>
        <A href="/dashboard/settings">设置</A>
      </nav>
      <Outlet /> {/* 子路由渲染位置 */}
    </div>
  );
}

function App() {
  return (
    <Router>
      <Route path="/dashboard" component={DashboardLayout}>
        <Route path="" component={() => <div>概览内容</div>} />
        <Route path="settings" component={() => <div>设置内容</div>} />
      </Route>
    </Router>
  );
}
```

**动态路由：**

```tsx
import { useParams, A } from '@solidjs/router';

function UserProfile() {
  const params = useParams();
  // params.id 获取 URL 参数
  
  return (
    <div>
      <h1>User: {params.id}</h1>
      <A href={`/users/${params.id}/posts`}>查看帖子</A>
    </div>
  );
}

// 带多个参数的路由
// /users/:userId/posts/:postId
function PostDetail() {
  const params = useParams();
  // params.userId, params.postId
}
```

**导航守卫：**

```tsx
import { Route, useNavigate } from '@solidjs/router';

function ProtectedRoute(props) {
  const [user] = createResource(async () => {
    const token = localStorage.getItem('token');
    if (!token) return null;
    return fetch('/api/me', {
      headers: { Authorization: `Bearer ${token}` }
    }).then(r => r.json());
  });
  
  const navigate = useNavigate();
  
  createEffect(() => {
    if (user.error || user() === null) {
      navigate('/login');
    }
  });
  
  return (
    <Show when={!user.loading && user()}>
      {props.children}
    </Show>
  );
}

// 使用
<Route path="/dashboard" component={ProtectedRoute}>
  <Route path="" component={Dashboard} />
</Route>
```

---

## 状态管理详解

### Context 深度指南

**创建 Context：**

```tsx
// theme.ts
import { createContext, useContext, ParentComponent } from 'solid-js';

export interface Theme {
  primary: string;
  secondary: string;
  background: string;
  text: string;
}

export const ThemeContext = createContext<{
  theme: () => Theme;
  toggleTheme: () => void;
}>();

export const ThemeProvider: ParentComponent = (props) => {
  const [isDark, setIsDark] = createSignal(false);
  
  const theme = createMemo(() => isDark() ? darkTheme : lightTheme);
  
  const toggleTheme = () => setIsDark(prev => !prev);
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {props.children}
    </ThemeContext.Provider>
  );
};
```

**使用 Context：**

```tsx
import { useContext } from 'solid-js';
import { ThemeContext } from './theme';

function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      style={{
        background: theme().primary,
        color: theme().text
      }}
      onClick={toggleTheme}
    >
      切换主题
    </button>
  );
}
```

### 状态管理模式

**简单 Store：**

```tsx
// stores/user.ts
import { createStore } from 'solid-js/store';

const [userStore, setUserStore] = createStore({
  user: null as User | null,
  token: null as string | null,
  loading: false,
  error: null as string | null
});

export { userStore, setUserStore };

// 在组件中使用
function Profile() {
  return (
    <Show when={userStore.user}>
      <div>{userStore.user.name}</div>
    </Show>
  );
}
```

**带 Actions 的 Store：**

```tsx
// stores/userStore.ts
import { createStore } from 'solid-js/store';

function createUserStore() {
  const [store, setStore] = createStore({
    user: null as User | null,
    token: null as string | null,
    loading: false
  });
  
  const actions = {
    async login(credentials: Credentials) {
      setStore('loading', true);
      try {
        const response = await fetch('/api/login', {
          method: 'POST',
          body: JSON.stringify(credentials)
        });
        const { user, token } = await response.json();
        setStore({ user, token, loading: false });
        localStorage.setItem('token', token);
      } catch (error) {
        setStore('loading', false);
        throw error;
      }
    },
    
    logout() {
      setStore({ user: null, token: null });
      localStorage.removeItem('token');
    }
  };
  
  return [store, actions];
}

export const [userStore, userActions] = createUserStore();
```

---

## 样式方案详解

### 内联样式

```tsx
function StyledDiv() {
  const styles = {
    color: 'white',
    background: 'blue',
    padding: '1rem',
    'border-radius': '4px'
  };
  
  return <div style={styles}>Styled Content</div>;
}

// 动态样式
function DynamicStyle() {
  const [active, setActive] = createSignal(false);
  
  return (
    <div
      style={{
        background: active() ? 'blue' : 'gray',
        transform: active() ? 'scale(1.1)' : 'scale(1)',
        transition: 'all 0.2s'
      }}
      onClick={() => setActive(prev => !prev)}
    >
      Click me
    </div>
  );
}
```

### CSS Modules

```tsx
// Component.module.css
.container {
  padding: 1rem;
  background: white;
  border-radius: 8px;
}

.title {
  font-size: 1.5rem;
  color: #333;
}

.button {
  background: #007bff;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}
```

```tsx
// Component.tsx
import styles from './Component.module.css';

function Component() {
  return (
    <div class={styles.container}>
      <h1 class={styles.title}>Title</h1>
      <button class={styles.button}>Click</button>
    </div>
  );
}
```

### Tailwind CSS

```tsx
import { createSignal } from 'solid-js';

function TailwindDemo() {
  const [isActive, setIsActive] = createSignal(false);
  
  return (
    <div class="p-4 bg-white rounded-lg shadow">
      <h1 class="text-2xl font-bold text-gray-900">
        Hello Tailwind
      </h1>
      
      <button
        class={`
          px-4 py-2 rounded font-medium
          ${isActive() 
            ? 'bg-blue-500 text-white' 
            : 'bg-gray-200 text-gray-700'}
          hover:bg-blue-600
          transition-colors
        `}
        onClick={() => setIsActive(prev => !prev)}
      >
        {isActive() ? 'Active' : 'Inactive'}
      </button>
    </div>
  );
}
```

### Class 工具

```tsx
import { createSignal } from 'solid-js';
import { mergeProps } from 'solid-js';

function Card(props) {
  // 合并默认 props
  const merged = mergeProps(
    { variant: 'default', size: 'md' },
    props
  );
  
  const classes = () => [
    'card',
    `card-${merged.variant}`,
    `card-${merged.size}`,
    merged.class
  ].filter(Boolean).join(' ');
  
  return <div class={classes()}>{merged.children}</div>;
}
```

---

## 性能优化详解

### 精细化追踪

**避免不必要的订阅：**

```tsx
// ❌ 问题：每次渲染创建新对象
createEffect(() => {
  console.log({
    count: count(),
    name: name()
  });
});

// ✅ 解决：分开追踪
createEffect(() => console.log('count:', count()));
createEffect(() => console.log('name:', name()));

// ✅ 或者使用 batch
import { batch } from 'solid-js';

function updateBoth() {
  batch(() => {
    setCount(c => c + 1);
    setName('new');
  });
}
```

### memo 与 untrack

**memo 缓存：**

```tsx
import { createMemo, createSignal, memo } from 'solid-js';

// memo 包装组件，避免不必要的重渲染
const ExpensiveList = memo(function ExpensiveList(props) {
  // 只有 props.items 变化时才重渲染
  return (
    <ul>
      <For each={props.items}>
        {(item) => <li>{item.name}</li>}
      </For>
    </ul>
  );
});
```

**untrack 读取：**

```tsx
import { untrack } from 'solid-js';

createEffect(() => {
  // 追踪 a
  console.log('a:', a());
  
  // 不追踪 b
  console.log('b:', untrack(b));
});
```

### batch 批量更新

```tsx
import { batch } from 'solid-js';

function handleClick() {
  batch(() => {
    setFirstName('John');
    setLastName('Doe');
    setAge(30);
    // 所有更新只在 DOM 应用一次
  });
}
```

---

## DevTools 与调试

### Solid DevTools

**主要功能：**

1. **Components 面板**
   - 查看组件树
   - 检查 Signal 值
   - 修改 Signal 值

2. **Performance 面板**
   - 性能分析
   - 渲染追踪

### 调试技巧

```tsx
// 1. console.log 追踪
const [count, setCount] = createSignal(0);

createEffect(() => {
  console.log('count:', count());
});

// 2. Debugger
createEffect(() => {
  debugger;
  console.log(count());
});

// 3. 错误边界
import { ErrorBoundary } from 'solid-js';

function App() {
  return (
    <ErrorBoundary
      fallback={(err, reset) => (
        <div>
          <p>Error: {err.message}</p>
          <button onClick={reset}>重试</button>
        </div>
      )}
    >
      <Component />
    </ErrorBoundary>
  );
}
```

---

## 与其他框架对比

### Solid vs React

| 维度 | Solid.js | React 18 |
|------|----------|----------|
| **架构** | 编译时优化 | 运行时 |
| **更新机制** | 精细化反应性 | 虚拟 DOM |
| **包体积** | ~7KB | ~45KB |
| **学习曲线** | 中等 | 中等 |
| **生态** | 较小 | 丰富 |

### Solid vs Svelte

| 维度 | Solid.js | Svelte 5 |
|------|----------|----------|
| **语法** | JSX | 自定义模板 |
| **反应性** | Signals | Runes |
| **编译** | JSX 编译 | 完整编译 |
| **学习曲线** | 中等 | 低 |

### Solid vs Vue

| 维度 | Solid.js | Vue 3 |
|------|----------|-------|
| **语法** | JSX | HTML 模板 |
| **反应性** | Signals | Proxy |
| **编译** | JSX 编译 | Vue 编译 |
| **包体积** | ~7KB | ~35KB |

---

## 学习路径与资源

### Solid.js 学习路线图

```
第一阶段：基础（1-2周）
├── Solid 基础语法
├── createSignal, createEffect, createMemo
├── JSX 基础
├── 组件基础
└── 事件处理

第二阶段：进阶（2-3周）
├── createStore 状态管理
├── Context 跨组件通信
├── @solidjs/router 路由
├── 控制流组件
└── 异步数据加载

第三阶段：生态（2-3周）
├── SolidStart 全栈开发
├── API 集成
├── 数据库集成
├── 测试基础
└── 部署

第四阶段：高级（持续学习）
├── 性能优化
├── 编译优化
├── 微前端
└── 最佳实践
```

### 推荐学习资源

**官方资源：**
- Solid.js 官方文档：https://solidjs.com/docs
- SolidStart 文档：https://start.solidjs.com
- Solid Playground：https://playground.solidjs.com

**教程：**
- SolidJS 官方指南
- Egghead.io Solid.js 课程

**社区：**
- Solid Discord
- Solid Forum

---

## 实战技巧

### AI 辅助开发

**Prompt 示例：**

```markdown
创建一个 Solid.js 用户卡片组件：
- 使用 TypeScript
- Props：
  - user: { id: string, name: string, email: string, avatar?: string }
  - size: 'sm' | 'md' | 'lg'
- 功能：
  - 显示头像（无头像显示首字母）
  - 角色标签
- 样式：使用 Tailwind CSS
- 使用 createSignal 管理本地状态
```

### 常见问题解决

**1. 组件不更新：**

```tsx
// ❌ 错误：没有调用 signal 函数
<p>{count}</p>

// ✅ 正确：调用 signal 函数
<p>{count()}</p>
```

**2. 对象更新不触发：**

```tsx
// ❌ 错误：直接修改对象
store.user.name = 'new';

// ✅ 正确：使用 setStore
setStore('user', 'name', 'new');
```

**3. Effect 依赖问题：**

```tsx
// ❌ 问题：没有追踪依赖
createEffect(() => {
  console.log('static');
});

// ✅ 正确：访问 signal 以建立追踪
createEffect(() => {
  console.log('count:', count());
});
```

---

## 测试工具与实践

### 测试框架概述

Solid.js 项目可以使用多种测试工具进行测试。

|| 测试类型 | 工具 | 说明 |
||---------|------|------|
|| **单元测试** | Vitest / Jest | 组件逻辑测试 |
|| **组件测试** | @solidjs/testing | Solid 官方测试库 |
|| **端到端测试** | Playwright / Cypress | 完整用户流程 |

### Vitest 基础

**配置 Vitest：**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import solidPlugin from 'vite-plugin-solid';
import solidTesting from '@solidjs/testing/vite';

export default defineConfig({
  plugins: [solidPlugin(), solidTesting()],
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{js,ts}'],
    coverage: {
      reporter: ['text', 'html'],
      exclude: ['node_modules/']
    }
  }
});
```

### 基础组件测试

**安装依赖：**

```bash
pnpm add -D vitest @solidjs/testing jsdom
```

**组件测试示例：**

```tsx
// Button.test.tsx
import { render, screen, fireEvent } from '@solidjs/testing';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button Component', () => {
  it('should render with text', () => {
    const { getByRole } = render(() => (
      <Button label="Click me" variant="primary" />
    ));

    expect(getByRole('button')).toBeInTheDocument();
    expect(getByRole('button')).toHaveTextContent('Click me');
  });

  it('should call onClick handler', async () => {
    const handleClick = vi.fn();
    const { getByRole } = render(() => (
      <Button label="Click me" onClick={handleClick} />
    ));

    await fireEvent.click(getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled when disabled prop is true', () => {
    const { getByRole } = render(() => (
      <Button label="Disabled" disabled={true} />
    ));

    expect(getByRole('button')).toBeDisabled();
  });
});
```

### Signal 测试

**Signal 测试示例：**

```tsx
// counter.test.tsx
import { render, screen, fireEvent } from '@solidjs/testing';
import { describe, it, expect, vi } from 'vitest';
import { Counter } from './Counter';

describe('Counter Component', () => {
  it('should display initial count', () => {
    const { getByText } = render(() => <Counter initialCount={5} />);
    expect(getByText('Count: 5')).toBeInTheDocument();
  });

  it('should increment count', async () => {
    const { getByText, getByRole } = render(() => <Counter />);

    await fireEvent.click(getByRole('button', { name: /increment/i }));

    expect(getByText('Count: 1')).toBeInTheDocument();
  });

  it('should decrement count', async () => {
    const { getByText, getByRole } = render(() => <Counter initialCount={5} />);

    await fireEvent.click(getByRole('button', { name: /decrement/i }));

    expect(getByText('Count: 4')).toBeInTheDocument();
  });
});
```

### Store 测试

**Store 测试示例：**

```tsx
// userStore.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { createStore } from 'solid-js/store';
import { userStore, setUser, clearUser } from './userStore';

describe('User Store', () => {
  beforeEach(() => {
    clearUser();
  });

  it('should have initial null user', () => {
    expect(userStore.user).toBeNull();
  });

  it('should set user', () => {
    const mockUser = { id: '1', name: '张三', email: 'zhangsan@example.com' };
    setUser(mockUser);

    expect(userStore.user).toEqual(mockUser);
  });

  it('should clear user', () => {
    setUser({ id: '1', name: '张三', email: 'zhangsan@example.com' });
    clearUser();

    expect(userStore.user).toBeNull();
  });
});
```

### 异步测试

**createResource 测试：**

```tsx
// UserList.test.tsx
import { render, screen, waitFor } from '@solidjs/testing';
import { describe, it, expect, vi } from 'vitest';
import { UserList } from './UserList';

// Mock fetch
const mockUsers = [
  { id: '1', name: '张三' },
  { id: '2', name: '李四' }
];

global.fetch = vi.fn().mockResolvedValueOnce({
  ok: true,
  json: () => Promise.resolve(mockUsers)
}) as any;

describe('UserList', () => {
  it('should display users after loading', async () => {
    const { getByText } = render(() => <UserList />);

    await waitFor(() => {
      expect(getByText('张三')).toBeInTheDocument();
      expect(getByText('李四')).toBeInTheDocument();
    });
  });

  it('should show loading state', () => {
    const { getByText } = render(() => <UserList />);
    expect(getByText('加载中...')).toBeInTheDocument();
  });
});
```

### 表单测试

**表单组件测试：**

```tsx
// Form.test.tsx
import { render, screen, fireEvent } from '@solidjs/testing';
import { describe, it, expect, vi } from 'vitest';
import { Form } from './Form';

describe('Form Component', () => {
  it('should validate required fields', async () => {
    const handleSubmit = vi.fn();
    const { getByRole, getByLabelText } = render(() => (
      <Form onSubmit={handleSubmit} />
    ));

    await fireEvent.click(getByRole('button', { name: /submit/i }));

    expect(getByLabelText(/name/i).closest('.error')).toHaveTextContent('Name is required');
  });

  it('should submit valid form', async () => {
    const handleSubmit = vi.fn();
    const { getByRole, getByLabelText } = render(() => (
      <Form onSubmit={handleSubmit} />
    ));

    await fireEvent.input(getByLabelText(/name/i), { target: { value: '张三' } });
    await fireEvent.input(getByLabelText(/email/i), { target: { value: 'zhangsan@example.com' } });
    await fireEvent.click(getByRole('button', { name: /submit/i }));

    expect(handleSubmit).toHaveBeenCalledWith({
      name: '张三',
      email: 'zhangsan@example.com'
    });
  });
});
```

### Playwright E2E 测试

**安装 Playwright：**

```bash
pnpm add -D @playwright/test
npx playwright install
```

**配置 Playwright：**

```typescript
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry'
  },
  projects: [
    {
      name: 'chromium',
      use: { browserName: 'chromium' }
    }
  ]
});
```

**E2E 测试示例：**

```typescript
// e2e/todo.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Todo App', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/');
  });

  test('should add a new todo', async ({ page }) => {
    await page.getByPlaceholder('What needs to be done?').fill('学习 Solid');
    await page.getByRole('button', { name: 'Add' }).click();

    await expect(page.getByText('学习 Solid')).toBeVisible();
  });

  test('should toggle todo completion', async ({ page }) => {
    await page.getByPlaceholder('What needs to be done?').fill('测试任务');
    await page.getByRole('button', { name: 'Add' }).click();

    const todoItem = page.getByText('测试任务');
    await todoItem.click();

    await expect(todoItem).toHaveClass(/completed/);
  });

  test('should delete a todo', async ({ page }) => {
    await page.getByPlaceholder('What needs to be done?').fill('待删除的任务');
    await page.getByRole('button', { name: 'Add' }).click();

    await page.getByRole('button', { name: 'Delete' }).first().click();

    await expect(page.getByText('待删除的任务')).not.toBeVisible();
  });

  test('should filter todos', async ({ page }) => {
    // 添加待办并标记为完成
    await page.getByPlaceholder('What needs to be done?').fill('已完成的任务');
    await page.getByRole('button', { name: 'Add' }).click();

    const todoItem = page.getByText('已完成的任务');
    await todoItem.click();

    // 点击 Completed 筛选
    await page.getByRole('button', { name: 'Completed' }).click();

    await expect(page.getByText('已完成的任务')).toBeVisible();
  });
});
```

### 测试覆盖率

**生成覆盖率报告：**

```bash
# Vitest 覆盖率
pnpm test --coverage

# 查看 HTML 报告
open coverage/index.html
```

**覆盖率配置：**

```typescript
// vite.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/**',
        'src/**/*.d.ts',
        '**/*.test.tsx'
      ]
    }
  }
});
```

### Mock 实战

**使用 MSW 模拟 API：**

```typescript
// mocks/handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([
      { id: '1', name: '张三' },
      { id: '2', name: '李四' }
    ]);
  }),

  http.post('/api/users', async ({ request }) => {
    const body = await request.json();
    return HttpResponse.json({
      id: crypto.randomUUID(),
      ...body
    }, { status: 201 });
  }),

  http.delete('/api/users/:id', () => {
    return new HttpResponse(null, { status: 204 });
  })
];
```

```typescript
// mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// 在测试中
import { server } from './mocks/server';

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 测试最佳实践

|| 实践 | 说明 |
||-----|------|
|| **Signal 调用** | 记住在测试中调用 signal 函数 |
|| **异步处理** | 使用 waitFor 处理异步操作 |
|| **独立测试** | 每个测试独立运行 |
|| **描述性命名** | 测试名称应描述预期行为 |
|| **Cleanup** | 使用 beforeEach 清理状态 |
|| **MSW Mock** | 使用 MSW 模拟 API 请求 |

---

## 高级模式与最佳实践

### Portal 组件

Solid.js 的 Portal 用于将子元素传送到 DOM 树的其他位置：

```tsx
import { Portal } from 'solid-js/web'
import { Show } from 'solid-js'

interface ModalProps {
  isOpen: boolean
  onClose: () => void
  children: any
}

export function Modal(props: ModalProps) {
  return (
    <Show when={props.isOpen}>
      <Portal mount={document.body}>
        <div class="modal-overlay" onClick={props.onClose}>
          <div class="modal-content" onClick={(e) => e.stopPropagation()}>
            <button class="close-btn" onClick={props.onClose}>×</button>
            {props.children}
          </div>
        </div>
      </Portal>
    </Show>
  )
}

// 多个 Portal
export function ToastContainer(props: { toasts: Toast[] }) {
  return (
    <>
      {props.toasts.map((toast) => (
        <Portal mount={document.getElementById('toast-root')}>
          <div class={`toast toast-${toast.type}`}>
            {toast.message}
          </div>
        </Portal>
      ))}
    </>
  )
}
```

### 动态组件

Solid.js 支持动态组件渲染：

```tsx
import { Dynamic } from 'solid-js/web'

// 组件映射
const components = {
  home: HomePage,
  about: AboutPage,
  contact: ContactPage
} as const

type ComponentName = keyof typeof components

interface PageSwitcherProps {
  currentPage: ComponentName
}

export function PageSwitcher(props: PageSwitcherProps) {
  const Component = () => components[props.currentPage]
  
  return (
    <Dynamic component={Component()} />
  )
}

// 或者直接传递组件
export function DynamicRenderer(props: { component: Component }) {
  return <Dynamic component={props.component} />
}
```

### 错误边界

Solid.js 的错误处理：

```tsx
import { ErrorBoundary } from 'solid-js'
import { Show } from 'solid-js'

interface ErrorFallbackProps {
  error: Error
  reset: () => void
}

function ErrorFallback(props: ErrorFallbackProps) {
  return (
    <div class="error-container">
      <h2>出错了</h2>
      <p>{props.error.message}</p>
      <button onClick={props.reset}>重试</button>
    </div>
  )
}

export function App() {
  return (
    <ErrorBoundary
      fallback={(error, reset) => (
        <ErrorFallback error={error} reset={reset} />
      )}
    >
      <Router />
    </ErrorBoundary>
  )
}

// 组件级错误边界
export function SafeComponent(props: { children: any }) {
  return (
    <ErrorBoundary
      fallback={(error) => <p>组件加载失败: {error.message}</p>}
    >
      {props.children}
    </ErrorBoundary>
  )
}
```

### 自定义指令

Solid.js 支持自定义指令：

```tsx
// directives/focus.ts
import { Directive, DirectiveNode } from 'solid-js'

export const vFocus: Directive<() => void> = (el) => {
  el.focus()
}

// 带参数的指令
export const vClickOutside: Directive<(e: MouseEvent) => void> = (el, accessor) => {
  const handler = (e: MouseEvent) => {
    if (!el.contains(e.target as Node)) {
      accessor()(e)
    }
  }
  
  document.addEventListener('click', handler)
  
  onCleanup(() => document.removeEventListener('click', handler))
}

// 指令使用
import { vFocus, vClickOutside } from './directives'

function MyInput() {
  let inputRef: HTMLInputElement | undefined
  
  return (
    <input 
      ref={inputRef}
      use:vFocus
      onInput={(e) => console.log(e.currentTarget.value)}
    />
  )
}

function Dropdown() {
  const [isOpen, setIsOpen] = createSignal(false)
  
  return (
    <div use:vClickOutside={() => setIsOpen(false)}>
      <button onClick={() => setIsOpen(true)}>打开</button>
      <Show when={isOpen()}>
        <div class="dropdown-menu">菜单内容</div>
      </Show>
    </div>
  )
}
```

### Context 深度使用

Solid.js Context API 详解：

```tsx
import { createContext, useContext, ParentComponent, createSignal } from 'solid-js'
import { createStore, SetStoreFunction } from 'solid-js/store'

// 1. 基础 Context
interface ThemeContextValue {
  theme: () => 'light' | 'dark'
  toggleTheme: () => void
  colors: {
    primary: string
    secondary: string
  }
}

const ThemeContext = createContext<ThemeContextValue>()

export const ThemeProvider: ParentComponent = (props) => {
  const [theme, setTheme] = createSignal<'light' | 'dark'>('light')
  
  const value: ThemeContextValue = {
    theme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light'),
    colors: {
      primary: theme() === 'light' ? '#3b82f6' : '#60a5fa',
      secondary: theme() === 'light' ? '#10b981' : '#34d399'
    }
  }
  
  return (
    <ThemeContext.Provider value={value}>
      {props.children}
    </ThemeContext.Provider>
  )
}

export function useTheme() {
  const context = useContext(ThemeContext)
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider')
  }
  return context
}

// 2. Store Context
interface User {
  id: string
  name: string
  email: string
  role: 'admin' | 'user' | 'guest'
}

interface UserStore {
  user: User | null
  isLoading: boolean
}

interface UserContextValue {
  store: UserStore
  setUser: (user: User | null) => void
  login: (email: string, password: string) => Promise<void>
  logout: () => void
}

const UserContext = createContext<UserContextValue>()

export const UserProvider: ParentComponent = (props) => {
  const [store, setStore] = createStore<UserStore>({
    user: null,
    isLoading: false
  })
  
  const login = async (email: string, password: string) => {
    setStore('isLoading', true)
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        body: JSON.stringify({ email, password })
      })
      const user = await response.json()
      setStore('user', user)
    } finally {
      setStore('isLoading', false)
    }
  }
  
  const logout = () => {
    setStore('user', null)
  }
  
  const value: UserContextValue = {
    store,
    setUser: (user) => setStore('user', user),
    login,
    logout
  }
  
  return (
    <UserContext.Provider value={value}>
      {props.children}
    </UserContext.Provider>
  )
}

// 使用 Context
function UserProfile() {
  const { store, logout } = useContext(UserContext)!
  
  return (
    <Show when={store.user} fallback={<p>请登录</p>}>
      <div class="profile">
        <p>欢迎, {store.user!.name}</p>
        <button onClick={logout}>退出</button>
      </div>
    </Show>
  )
}
```

### 动画与过渡

Solid.js 动画实现：

```tsx
import { createSignal, onMount, onCleanup } from 'solid-js'

// 基础动画 Hook
function useAnimatedValue(initial: number) {
  const [value, setValue] = createSignal(initial)
  
  return {
    value,
    animate: (target: number, duration = 300) => {
      const start = value()
      const startTime = performance.now()
      
      const animate = (currentTime: number) => {
        const elapsed = currentTime - startTime
        const progress = Math.min(elapsed / duration, 1)
        const eased = 1 - Math.pow(1 - progress, 3) // ease-out cubic
        
        setValue(start + (target - start) * eased)
        
        if (progress < 1) {
          requestAnimationFrame(animate)
        }
      }
      
      requestAnimationFrame(animate)
    }
  }
}

// 列表动画
import { For } from 'solid-js'

function AnimatedList() {
  const [items, setItems] = createSignal([1, 2, 3, 4, 5])
  
  const AnimatedItem = (props: { value: number }) => {
    const [opacity, setOpacity] = createSignal(0)
    const [translateY, setTranslateY] = createSignal(20)
    
    onMount(() => {
      requestAnimationFrame(() => {
        setOpacity(1)
        setTranslateY(0)
      })
    })
    
    return (
      <div
        style={{
          opacity: opacity(),
          transform: `translateY(${translateY()}px)`,
          transition: 'opacity 300ms ease-out, transform 300ms ease-out'
        }}
      >
        {props.value}
      </div>
    )
  }
  
  return (
    <div class="list">
      <For each={items()}>
        {(item) => <AnimatedItem value={item} />}
      </For>
    </div>
  )
}

// 拖拽排序
function DraggableList() {
  const [items, setItems] = createSignal(['A', 'B', 'C', 'D'])
  const [draggedIndex, setDraggedIndex] = createSignal<number | null>(null)
  
  const handleDragStart = (index: number) => {
    setDraggedIndex(index)
  }
  
  const handleDragOver = (index: number) => {
    const from = draggedIndex()
    if (from === null || from === index) return
    
    const newItems = [...items()]
    const [removed] = newItems.splice(from, 1)
    newItems.splice(index, 0, removed)
    setItems(newItems)
    setDraggedIndex(index)
  }
  
  const handleDragEnd = () => {
    setDraggedIndex(null)
  }
  
  return (
    <div>
      <For each={items()}>
        {(item, index) => (
          <div
            draggable={true}
            onDragStart={() => handleDragStart(index())}
            onDragOver={(e) => {
              e.preventDefault()
              handleDragOver(index())
            }}
            onDragEnd={handleDragEnd}
            class={draggedIndex() === index() ? 'dragging' : ''}
          >
            {item}
          </div>
        )}
      </For>
    </div>
  )
}
```

### SSR/SSG 深度理解

SolidStart SSR/SSG 实现：

```typescript
// src/routes/api/users/[id].ts
import { APIEvent } from '@solidjs/start/server'

export async function GET(event: APIEvent) {
  const { params } = event
  const userId = params.id
  
  const user = await db.user.findUnique({
    where: { id: userId },
    include: { posts: true }
  })
  
  if (!user) {
    return new Response('User not found', { status: 404 })
  }
  
  return Response.json({ user })
}

export async function PUT(event: APIEvent) {
  const { params, request } = event
  const body = await request.json()
  
  const user = await db.user.update({
    where: { id: params.id },
    data: body
  })
  
  return Response.json({ user })
}

export async function DELETE(event: APIEvent) {
  const { params } = event
  
  await db.user.delete({
    where: { id: params.id }
  })
  
  return new Response(null, { status: 204 })
}
```

```tsx
// src/routes/users/[id].tsx
import { createAsync, cache, useParams } from '@solidjs/router'
import { Show } from 'solid-js'

const getUser = cache(async (id: string) => {
  'use server'
  return await db.user.findUnique({
    where: { id },
    include: { posts: true }
  })
}, 'user')

export default function UserPage() {
  const params = useParams()
  const user = createAsync(() => getUser(params.id))
  
  return (
    <Show when={user()} fallback={<p>加载中...</p>}>
      {(u) => (
        <div>
          <h1>{u().name}</h1>
          <p>{u().email}</p>
          <h2>文章列表</h2>
          <ul>
            <For each={u().posts}>
              {(post) => <li>{post.title}</li>}
            </For>
          </ul>
        </div>
      )}
    </Show>
  )
}
```

```typescript
// app.config.ts
export default defineConfig({
  ssr: true,
  server: {
    preset: 'node-server'
  }
})
```

### 国际化（i18n）

Solid.js 国际化实现：

```typescript
// src/i18n/index.ts
import { createSignal, createContext, useContext, ParentComponent } from 'solid-js'

type Locale = 'en' | 'zh' | 'ja'

const translations: Record<Locale, Record<string, string>> = {
  en: {
    'app.title': 'My App',
    'nav.home': 'Home',
    'nav.about': 'About',
    'user.greeting': 'Hello, {name}!',
    'common.loading': 'Loading...',
    'common.error': 'An error occurred'
  },
  zh: {
    'app.title': '我的应用',
    'nav.home': '首页',
    'nav.about': '关于',
    'user.greeting': '你好，{name}！',
    'common.loading': '加载中...',
    'common.error': '发生错误'
  },
  ja: {
    'app.title': 'マイアプリ',
    'nav.home': 'ホーム',
    'nav.about': '概要',
    'user.greeting': 'こんにちは、{name}さん！',
    'common.loading': '読み込み中...',
    'common.error': 'エラーが発生しました'
  }
}

interface I18nContextValue {
  locale: () => Locale
  setLocale: (locale: Locale) => void
  t: (key: string, params?: Record<string, string | number>) => string
}

const I18nContext = createContext<I18nContextValue>()

export const I18nProvider: ParentComponent = (props) => {
  const [locale, setLocale] = createSignal<Locale>('zh')
  
  const t = (key: string, params?: Record<string, string | number>): string => {
    let text = translations[locale()][key] || key
    
    if (params) {
      Object.entries(params).forEach(([k, v]) => {
        text = text.replace(`{${k}}`, String(v))
      })
    }
    
    return text
  }
  
  return (
    <I18nContext.Provider value={{ locale, setLocale, t }}>
      {props.children}
    </I18nContext.Provider>
  )
}

export function useI18n() {
  const context = useContext(I18nContext)
  if (!context) {
    throw new Error('useI18n must be used within I18nProvider')
  }
  return context
}
```

```tsx
// 使用
import { useI18n } from './i18n'

function Header() {
  const { t, locale, setLocale } = useI18n()
  
  return (
    <header>
      <h1>{t('app.title')}</h1>
      <nav>
        <a href="/">{t('nav.home')}</a>
        <a href="/about">{t('nav.about')}</a>
      </nav>
      <div class="locale-switcher">
        <button onClick={() => setLocale('en')}>EN</button>
        <button onClick={() => setLocale('zh')}>中文</button>
        <button onClick={() => setLocale('ja')}>日本語</button>
      </div>
    </header>
  )
}
```

### 性能优化

```tsx
import { createMemo, createSignal, batch } from 'solid-js'

// 1. 批量更新
function BatchUpdateExample() {
  const [firstName, setFirstName] = createSignal('')
  const [lastName, setLastName] = createSignal('')
  const [fullName, setFullName] = createSignal('')
  
  const updateName = () => {
    batch(() => {
      setFirstName('张')
      setLastName('三')
      setFullName('张三')
    })
  }
  
  return (
    <div>
      <input 
        value={firstName()} 
        onInput={(e) => setFirstName(e.currentTarget.value)} 
      />
      <input 
        value={lastName()} 
        onInput={(e) => setLastName(e.currentTarget.value)} 
      />
      <p>全名: {fullName()}</p>
      <button onClick={updateName}>批量更新</button>
    </div>
  )
}

// 2. 记忆化计算
function MemoizedList(props: { items: number[]; filter: string }) {
  const filteredItems = createMemo(() => {
    console.log('Filtering...')
    return props.items.filter(item => item.toString().includes(props.filter))
  })
  
  return (
    <ul>
      <For each={filteredItems()}>
        {(item) => <li>{item}</li>}
      </For>
    </ul>
  )
}

// 3. 延迟初始化
function LazyInitComponent() {
  const heavyData = createSignalExpensiveInit(() => {
    return computeHeavyData()
  })
  
  return <div>{heavyData().length} items</div>
}

// 4. 虚拟列表
import { createVirtualizer } from '@tanstack/solid-virtual'

function VirtualList(props: { items: Item[] }) {
  let parentRef: HTMLDivElement | undefined
  
  const virtualizer = createVirtualizer({
    get count() { return props.items.length },
    getScrollElement: () => parentRef!,
    estimateSize: () => 50
  })
  
  return (
    <div 
      ref={parentRef} 
      style={{ height: '400px', overflow: 'auto' }}
    >
      <div style={{ height: `${virtualizer.getTotalSize()}px` }}>
        <For each={virtualizer.getVirtualItems()}>
          {(virtualRow) => (
            <div
              style={{
                position: 'absolute',
                top: `${virtualRow.start}px`,
                height: `${virtualRow.size}px`
              }}
            >
              {props.items[virtualRow.index].name}
            </div>
          )}
        </For>
      </div>
    </div>
  )
}

// 5. 防抖
function DebouncedInput() {
  const [value, setValue] = createSignal('')
  const [debouncedValue, setDebouncedValue] = createSignal('')
  
  let timeoutId: number | undefined
  
  const handleInput = (newValue: string) => {
    setValue(newValue)
    
    clearTimeout(timeoutId)
    timeoutId = setTimeout(() => {
      setDebouncedValue(newValue)
    }, 300) as unknown as number
  }
  
  return (
    <div>
      <input 
        value={value()} 
        onInput={(e) => handleInput(e.currentTarget.value)} 
      />
      <p>防抖值: {debouncedValue()}</p>
    </div>
  )
}
```

---

## Solid.js 生态工具链详解

### 构建工具配置

Vite 深度配置：

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import solid from 'vite-plugin-solid'

export default defineConfig({
  plugins: [solid()],
  
  resolve: {
    alias: {
      '@': '/src',
      '@components': '/src/components',
      '@stores': '/src/stores',
      '@utils': '/src/utils'
    }
  },
  
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  
  build: {
    target: 'esnext',
    minify: 'esbuild',
    rollupOptions: {
      output: {
        manualChunks: {
          'solid-vendor': ['solid-js', '@solidjs/router', '@solidjs/meta']
        }
      }
    }
  }
})
```

### 样式方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **内联样式** | 简单直观 | 无语法高亮，样式复用难 | 小型项目 |
| **CSS Modules** | 隔离彻底，类名可预测 | 需要配置 | 大型项目 |
| **Tailwind CSS** | 极快开发，原子化 | 类名冗长 | 快速原型 |
| **Vanilla Extract** | 类型安全，零运行时 | 编译时生成 | 中大型项目 |
| **CSS Variables + Scoped** | 主题切换简单 | 需要构建 | 主题需求 |

---

## 企业级架构模式

### 微前端架构 (Micro Frontends)

微前端将微服务的理念引入前端，实现大型应用的独立开发和部署：

```typescript
// 1. Solid.js 中加载远程模块
// src/App.tsx
import { createSignal, onMount, Show } from 'solid-js';

function App() {
  const [RemoteApp, setRemoteApp] = createSignal<any>(null);
  const [loading, setLoading] = createSignal(true);
  const [error, setError] = createSignal<string | null>(null);

  onMount(async () => {
    try {
      const module = await import('http://remote.example.com/remote.js');
      setRemoteApp(() => module.default);
      setLoading(false);
    } catch (e) {
      setError('Failed to load remote module');
      setLoading(false);
    }
  });

  return (
    <div class="micro-frontend-container">
      <Show when={!loading()} fallback={<div>Loading...</div>}>
        <Show when={!error()} fallback={<div>{error()}</div>}>
          <Show when={RemoteApp()}>
            {RemoteApp() && <RemoteApp />}
          </Show>
        </Show>
      </Show>
    </div>
  );
}
```

```typescript
// 2. Module Federation 配置
// vite.config.ts
import { defineConfig } from 'vite';
import solid from 'vite-plugin-solid';
import ModuleFederation from 'vite-plugin-federation';

export default defineConfig({
  plugins: [
    solid(),
    ModuleFederation({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@https://remote.example.com/remoteEntry.js',
      },
      shared: ['solid-js', 'solid-js/web'],
    }),
  ],
});
```

**微前端实现方案对比：**

| 方案 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|---------|------|------|----------|
| **Module Federation** | Webpack 共享 | 运行时集成，按需加载 | 依赖统一版本 | 大型团队协作 |
| **iframe** | HTML iframe | 隔离彻底，技术栈无关 | 通信困难 | 独立子应用 |
| **Single-SPA** | 生命周期管理 | 框架无关，成熟方案 | 需要适配层 | 多框架共存 |
| **动态导入** | 原生 ES 模块 | 简单直接 | 共享状态困难 | 简单场景 |

### Solid.js 组件架构模式

```tsx
// 1. Container/Presentational 模式
// src/components/UserList.tsx (Presentational)
import { Show, For } from 'solid-js';

interface UserListProps {
  users: User[];
  loading: boolean;
  error: string | null;
  onSelect: (id: string) => void;
  onDelete: (id: string) => void;
}

export function UserList(props: UserListProps) {
  return (
    <Show when={!props.loading} fallback={<div>Loading...</div>}>
      <Show when={!props.error} fallback={<div>{props.error}</div>}>
        <div class="user-list">
          <For each={props.users}>
            {(user) => (
              <div class="user-card">
                <h3>{user.name}</h3>
                <p>{user.email}</p>
                <button onClick={() => props.onSelect(user.id)}>查看详情</button>
                <button onClick={() => props.onDelete(user.id)}>删除</button>
              </div>
            )}
          </For>
        </div>
      </Show>
    </Show>
  );
}
```

### Solid.js Store 进阶模式

```typescript
// stores/asyncStore.ts - 异步 Store
import { createSignal, createEffect, on } from 'solid-js';

export function createAsyncStore<T>(fetcher: () => Promise<T>) {
  const [data, setData] = createSignal<T | null>(null);
  const [loading, setLoading] = createSignal(false);
  const [error, setError] = createSignal<Error | null>(null);

  async function fetch() {
    setLoading(true);
    setError(null);
    try {
      const result = await fetcher();
      setData(result);
    } catch (e) {
      setError(e as Error);
    } finally {
      setLoading(false);
    }
  }

  createEffect(on(() => fetcher, fetch));

  return {
    data,
    loading,
    error,
    fetch,
    refetch: fetch,
  };
}
```

---

## CI/CD 部署与自动化

### GitHub Actions 完整配置

```yaml
# .github/workflows/solid-ci-cd.yml
name: Solid.js CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'

jobs:
  lint:
    name: ESLint & Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run lint

  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run test:coverage

  build-and-deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    needs: test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build
        env:
          PUBLIC_API_URL: ${{ secrets.PROD_API_URL }}
      - uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./dist
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Netlify 部署配置

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"
```

---

## 安全最佳实践

### XSS 防护

```tsx
// src/utils/sanitize.ts
import DOMPurify from 'dompurify';

export function sanitizeHtml(dirty: string): string {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'target'],
  });
}
```

### CSRF 防护

```typescript
// src/utils/csrf.ts
import { createSignal } from 'solid-js';

const [csrfToken, setCsrfToken] = createSignal<string | null>(null);

export async function fetchCsrfToken(): Promise<string> {
  if (csrfToken()) return csrfToken()!;
  
  const response = await fetch('/api/csrf-token', {
    credentials: 'include',
  });
  const data = await response.json();
  setCsrfToken(data.token);
  return data.token;
}

export async function secureRequest(
  url: string,
  options: RequestInit = {}
): Promise<Response> {
  const token = await fetchCsrfToken();
  const mutationMethods = ['POST', 'PUT', 'PATCH', 'DELETE'];
  
  if (mutationMethods.includes(options.method || 'GET')) {
    return fetch(url, {
      ...options,
      credentials: 'include',
      headers: {
        ...options.headers,
        'X-CSRF-Token': token,
        'Content-Type': 'application/json',
      },
    });
  }
  
  return fetch(url, {
    ...options,
    credentials: 'include',
  });
}
```

### 敏感信息管理

```bash
# .env.development
VITE_API_URL=http://localhost:3000
VITE_ENABLE_DEBUG=true

# .env.production
VITE_API_URL=https://api.production.com
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
```

---

## 可访问性 (A11y) 最佳实践

### ARIA 属性完整指南

```tsx
// 1. 按钮 vs 链接
function AccessibleActions() {
  return (
    <div>
      <button onClick={handleSave} aria-describedby="save-description">
        保存
      </button>
      <p id="save-description">保存当前编辑的内容</p>
      
      <a href="/dashboard" aria-current="page">
        前往仪表盘
      </a>
    </div>
  );
}

// 2. 表单错误提示
function AccessibleForm() {
  const [email, setEmail] = createSignal('');
  const [errors, setErrors] = createSignal<Record<string, string>>({});
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label for="email">邮箱地址</label>
        <input
          id="email"
          type="email"
          value={email()}
          onInput={(e) => setEmail(e.currentTarget.value)}
          aria-invalid={errors().email ? 'true' : undefined}
          aria-describedby={errors().email ? 'email-error' : undefined}
        />
        <Show when={errors().email}>
          <p id="email-error" role="alert">
            {errors().email}
          </p>
        </Show>
      </div>
    </form>
  );
}

// 3. 模态对话框
function Modal(props: { isOpen: boolean; title: string; onClose: () => void }) {
  let dialogRef!: HTMLDivElement;
  
  onMount(() => {
    if (props.isOpen) {
      dialogRef.focus();
      document.body.style.overflow = 'hidden';
    }
  });
  
  onCleanup(() => {
    document.body.style.overflow = '';
  });
  
  return (
    <Show when={props.isOpen}>
      <div
        ref={dialogRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-title"
        tabIndex={-1}
      >
        <h2 id="modal-title">{props.title}</h2>
        <button onClick={props.onClose} aria-label="关闭">×</button>
        <slot />
      </div>
    </Show>
  );
}

// 4. 实时区域
function Notifications() {
  const [notification, setNotification] = createSignal('');
  
  return (
    <>
      <div aria-live="polite" aria-atomic="true" class="sr-only">
        {notification()}
      </div>
      <button onClick={() => {
        setNotification('保存成功');
        setTimeout(() => setNotification(''), 3000);
      }}>
        保存
      </button>
    </>
  );
}

// 5. 复杂表格
function AccessibleTable() {
  return (
    <table>
      <caption>2024年季度销售报表</caption>
      <thead>
        <tr>
          <th scope="col" rowspan="2">产品</th>
          <th scope="colgroup" colspan="4">季度</th>
        </tr>
        <tr>
          <th scope="col">Q1</th>
          <th scope="col">Q2</th>
          <th scope="col">Q3</th>
          <th scope="col">Q4</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row">产品A</th>
          <td>100万</td>
          <td>120万</td>
          <td>150万</td>
          <td>180万</td>
        </tr>
      </tbody>
    </table>
  );
}
```

### 键盘导航支持

```tsx
// Roving Tabindex
function AccessibleMenu() {
  const menuItems = ['首页', '关于', '产品', '联系'];
  const [activeIndex, setActiveIndex] = createSignal(0);
  
  const handleKeyDown = (e: KeyboardEvent, index: number) => {
    switch (e.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        e.preventDefault();
        setActiveIndex((prev) => (prev + 1) % menuItems.length);
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        e.preventDefault();
        setActiveIndex((prev) => (prev - 1 + menuItems.length) % menuItems.length);
        break;
      case 'Home':
        e.preventDefault();
        setActiveIndex(0);
        break;
      case 'End':
        e.preventDefault();
        setActiveIndex(menuItems.length - 1);
        break;
      case 'Enter':
      case ' ':
        e.preventDefault();
        handleSelect(menuItems[index]);
        break;
    }
  };
  
  const handleSelect = (item: string) => {
    console.log('Selected:', item);
  };
  
  return (
    <nav role="menubar" aria-label="主菜单">
      <For each={menuItems}>
        {(item, index) => (
          <button
            role="menuitem"
            tabIndex={index() === activeIndex() ? 0 : -1}
            onClick={() => handleSelect(item)}
            onKeyDown={(e) => handleKeyDown(e, index())}
          >
            {item}
          </button>
        )}
      </For>
    </nav>
  );
}
```

### 颜色对比度和视觉辅助

```css
/* WCAG AA 标准对比度检查 */
/* 正常文本: 4.5:1 */
/* 大文本 (18px+): 3:1 */

:root {
  --color-primary: #2563eb;
  --text-primary: #1f2937;
  --text-secondary: #4b5563;
}

*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

---

## 环境配置与环境变量管理

### 多种环境配置

```bash
# .env.development
VITE_API_URL=http://localhost:3000
VITE_ENABLE_DEBUG=true

# .env.production
VITE_API_URL=https://api.production.com
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
```

```typescript
// src/config/environment.ts
interface AppConfig {
  apiUrl: string;
  sentryDsn: string | null;
  enableDebug: boolean;
}

export function getConfig(): AppConfig {
  return {
    apiUrl: import.meta.env.VITE_API_URL || 'http://localhost:3000',
    sentryDsn: import.meta.env.VITE_SENTRY_DSN || null,
    enableDebug: import.meta.env.DEV,
  };
}
```

---

> [!SUCCESS]
> 本文档涵盖了 Solid.js 的核心理念、核心 API、组件系统、路由、状态管理、样式方案、性能优化、高级模式、Portal、Context深度、动画、SSR、国际化等全方位内容。Solid.js 以精细化反应性系统和零虚拟 DOM 的架构，为高性能 Web 应用提供了全新的选择。其在 AI 应用中的性能优势使其特别适合需要高频更新和流式响应的场景。
