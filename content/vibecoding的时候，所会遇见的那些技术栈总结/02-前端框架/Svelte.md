# Svelte 5 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Svelte 5 全新 Runes 语法、编译时反应性原理、SvelteKit 生态及在 vibecoding 中的战略定位。

---

## 目录

1. [[#Svelte 概述与核心哲学]]
2. [[#Svelte 4 vs Svelte 5 对比]]
3. [[#编译时 vs 运行时反应性]]
4. [[#Runes 语法详解]]
5. [[#Svelte 组件基础]]
6. [[#Svelte Store]]
7. [[#Transition 与 Animation]]
8. [[#SvelteKit 全栈框架]]
9. [[#Svelte vs React vs Vue vs Solid 对比]]
10. [[#Svelte 在 AI 应用中的定位]]
11. [[#选型建议与实战场景]]

---

## Svelte 概述与核心哲学

### 什么是 Svelte

Svelte 是一个**编译时框架**（Compile-time Framework），其核心理念是将框架的"虚拟 DOM diffing"工作从浏览器运行时转移到构建阶段的编译器中。与 React 和 Vue 等运行时框架不同，Svelte 在构建时生成精确的 DOM 操作代码，应用启动时无需加载框架运行时，从而实现极致的首屏性能和极小的打包体积。

**Svelte 发展历程：**

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| Svelte 1/2 | 2016-2017 | 萌芽期，核心语法确立 |
| Svelte 3 | 2019 | 反应性系统重构，Component 模型成熟 |
| Svelte 4 | 2023 | 性能优化，文档重构 |
| **Svelte 5** | **2024** | **Runes 语法，颠覆性重构** |

> [!IMPORTANT]
> Svelte 5 的 Runes 语法是对 Svelte 4 反应性系统的全面升级，引入了 $state、$derived、$effect、$props 四个核心 Rune，大幅提升了反应性系统的可控性和可预测性。

### Svelte 的核心优势

1. **零运行时开销**：编译后无虚拟 DOM，无框架运行时，打包体积通常比 React 小 5-10 倍
2. **精确 DOM 更新**：编译器直接生成 DOM 操作代码，避免 diffing 性能损耗
3. **真正的反应性**：基于赋值触发的细粒度更新，无需调用 setState
4. **更少的模板代码**：相比 JSX，`.svelte` 文件语法更简洁直观
5. **内置动画支持**：Transition 和 Animation 指令开箱即用

### Svelte 在前端生态的定位

```
传统运行时框架：
React/Vue ──→ 虚拟 DOM ──→ DOM 更新（运行时开销）

Svelte（编译时框架）：
.vue/.svelte ──→ 编译器 ──→ 精确 DOM 代码 ──→ DOM 更新（零运行时）
```

---

## Svelte 4 vs Svelte 5 对比

Svelte 5 是 Svelte 历史上最重要的版本升级，带来了 Runes 语法这一颠覆性变革。以下是详细对比：

### 核心语法对比

| 特性 | Svelte 4 | Svelte 5 | 说明 |
|------|---------|---------|------|
| **状态声明** | `let count = 0`（隐式反应性） | `$state()`（显式 Rune） | Svelte 5 要求显式声明反应性状态 |
| **计算属性** | `$:` 标签（reactive statement） | `$derived()` | 更符合函数式思维 |
| **副作用** | `$:` 标签 + onMount | `$effect()` | 更清晰的副作用管理 |
| **Props** | `export let` | `$props()` | 统一的 Props API |
| **Context** | `getContext/setContext` | 保持 + 新增 Snippets | Snippets 是重大新增 |
| **事件处理** | `on:click` | `onclick`（DOM 原生） | 与标准 Web 一致 |
| **组件声明** | `<script>` 块 | `<script>` + Snippets | 新增编译时组件片段 |

### 状态声明对比

**Svelte 4 写法（隐式反应性）：**

```svelte
<script>
  // Svelte 4：直接声明即具备反应性
  let count = 0;
  let doubled = count * 2;

  // 手动声明依赖关系
  $: doubled = count * 2;

  function increment() {
    count += 1; // 直接赋值触发更新
  }
</script>

<button on:click={increment}>
  Count: {count}, Doubled: {doubled}
</button>
```

**Svelte 5 写法（Runes 语法）：**

```svelte
<script>
  import { useState } from 'svelte';

  // Svelte 5：显式声明反应性状态
  let count = $state(0);

  // 计算属性：自动追踪依赖
  let doubled = $derived(count * 2);

  function increment() {
    count += 1; // 直接赋值触发更新
  }
</script>

<button onclick={increment}>
  Count: {count}, Doubled: {doubled}
</button>
```

### 代码量对比

| 场景 | Svelte 4 | Svelte 5 | 减少比例 |
|------|---------|---------|----------|
| 简单计数器 | 8 行 | 6 行 | 25% |
| 表单处理 | 45 行 | 35 行 | 22% |
| 复杂状态逻辑 | 120 行 | 95 行 | 21% |

> [!TIP]
> Svelte 5 的 Runes 语法虽然增加了显式声明，但带来了更好的 TypeScript 兼容性和代码可预测性。

---

## 编译时 vs 运行时反应性

理解 Svelte 的编译时反应性机制是掌握 Svelte 的关键。

### 反应性模式对比

| 维度 | 编译时反应性（Svelte） | 运行时反应性（React/Vue） |
|------|----------------------|-------------------------|
| **工作时机** | 构建阶段（Build Time） | 运行时（Runtime） |
| **DOM 更新** | 编译器生成精确代码 | 虚拟 DOM diffing |
| **内存占用** | 无运行时开销 | 需要维护虚拟 DOM |
| **CPU 消耗** | 构建时一次性消耗 | 每次状态变化时 diffing |
| **包体积** | 极小（无运行时） | 较大（框架 + 虚拟 DOM） |
| **初始加载** | 快（代码量少） | 较慢（需加载框架） |
| **更新性能** | 最快（直接操作） | 中等（diffing 优化后） |
| **调试难度** | 较高（源码映射） | 较低（React DevTools） |

### 编译时反应性详解

Svelte 编译器在构建阶段执行以下工作：

**1. 静态分析阶段：**
```
源代码 → AST（抽象语法树）→ 依赖关系图
```

**2. 代码生成阶段：**

```javascript
// 输入：Svelte 组件
<script>
  let count = 0;
  function increment() { count += 1; }
</script>
<button onclick={increment}>{count}</button>

// 输出：编译后的 JavaScript
// 编译器生成精确的 DOM 操作代码
```

**3. 输出代码示例：**

```javascript
// Svelte 编译器生成的代码（简化版）
import { SvelteComponent, tick } from 'svelte/runtime';

class Button extends SvelteComponent {
  constructor(options) {
    super();
    // 初始化 count 变量
    this.count = 0;
    // 编译时生成 DOM 结构
    this.$$render = () => {
      // 精确的 DOM 更新逻辑
    };
  }

  // 编译器生成的更新函数（无虚拟 DOM）
  $set({ count }) {
    if (count !== this.count) {
      this.count = count;
      // 直接操作 DOM，无 diffing
      this.$$nodes[0].textContent = count;
    }
  }
}
```

### 与 React 的本质区别

**React（运行时反应性）：**

```jsx
// React：每次渲染创建新的虚拟 DOM
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}

// 运行时：React 维护虚拟 DOM 树
// 状态变化 → 重新渲染 → diffing → DOM 更新
```

**Svelte（编译时反应性）：**

```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count += 1}>
  Count: {count}
</button>

// 编译时：生成精确的 DOM 更新代码
// 状态变化 → 直接调用编译器生成的更新函数
```

### 性能数据对比

| 指标 | Svelte 5 | React 18 | Vue 3 | Solid |
|------|---------|----------|-------|-------|
| 包体积（基准） | 1x | 4.5x | 3.2x | 1.2x |
| 首屏加载时间 | 最快 | 中等 | 中等 | 极快 |
| 状态更新延迟 | 最低 | 中等 | 中等 | 最低 |
| 内存占用 | 最低 | 较高 | 中等 | 低 |
| 大列表渲染 | 优秀 | 需优化 | 优秀 | 优秀 |

> [!IMPORTANT]
> Svelte 的编译时反应性并不意味着"预渲染"。Svelte 仍是客户端框架，只是将反应性系统的工作提前到编译阶段。

---

## Runes 语法详解

Svelte 5 引入了 Runes（符文）这一全新的反应性原语系统。Runes 是一种编译时标记，用于显式声明反应性状态和计算逻辑。

### $state - 反应性状态

```svelte
<script>
  // 基本用法
  let count = $state(0);

  // 对象状态
  let user = $state({
    name: 'Alice',
    age: 30
  });

  // 数组状态
  let items = $state([]);
</script>

<button onclick={() => count += 1}>
  Count: {count}
</button>

<p>Name: {user.name}, Age: {user.age}</p>
```

### $derived - 计算属性

```svelte
<script>
  let count = $state(10);

  // 简单派生
  let doubled = $derived(count * 2);

  // 复杂派生（使用函数形式）
  let status = $derived(() => {
    if (count < 0) return 'negative';
    if (count === 0) return 'zero';
    return 'positive';
  });

  // 派生数组
  let evens = $derived(items.filter(n => n % 2 === 0));
</script>
```

### $effect - 副作用

```svelte
<script>
  let count = $state(0);
  let message = $state('');

  // 基本副作用
  $effect(() => {
    console.log(`Count changed to: ${count}`);

    // 清理函数
    return () => {
      console.log('Cleanup');
    };
  });

  // 带依赖的副作用
  $effect(() => {
    if (count > 10) {
      message = 'Count is greater than 10!';
    }
  });

  // 追踪多个依赖
  $effect(() => {
    document.title = `Count: ${count} - ${new Date().toLocaleTimeString()}`;
  });
</script>
```

### $props - 属性声明

```svelte
<script>
  // 解构 Props
  let { name, age = 18, onClick } = $props();

  // 高级 Props：用 $derived 派生 Props
  let { items } = $props();
  let count = $derived(items.length);

  // 使用 rest props
  let { class: className, ...rest } = $props();
</script>

<div class={className} {...rest}>
  <h1>{name}, Age: {age}</h1>
  <p>Total items: {count}</p>
  <button onclick={onClick}>Click me</button>
</div>
```

### Snippets - 组件片段

Svelte 5 新增的 Snippets 是强大的组件化工具：

```svelte
<script>
  import Card from './Card.svelte';
  import Button from './Button.svelte';

  // 定义 Snippet
  let header = $snippet(({ title }) => `
    <div class="header">
      <h1>{title}</h1>
    </div>
  `);

  let footer = $snippet(({ actions }) => `
    <div class="footer">
      {actions}
    </div>
  `);
</script>

<Card>
  {@render header({ title: 'Welcome' })}
  <p>This is the card content.</p>
  {@render footer({ actions: [Button] })}
</Card>
```

---

## Svelte 组件基础

### 组件结构

```svelte
<!-- Component.svelte -->
<script>
  // 逻辑层：JavaScript/TypeScript
  import { onMount } from 'svelte';

  let { title = 'Default' } = $props();
  let count = $state(0);

  onMount(() => {
    console.log('Component mounted');
  });

  function handleClick() {
    count += 1;
  }
</script>

<!-- 模板层：HTML + Svelte 指令 -->
<div class="component">
  <h1>{title}</h1>
  <button onclick={handleClick}>
    Clicked {count} times
  </button>
</div>

<!-- 样式层：Scoped CSS -->
<style>
  .component {
    padding: 1rem;
    border: 1px solid #ddd;
    border-radius: 8px;
  }

  h1 {
    color: #333;
  }
</style>
```

### 事件处理

```svelte
<script>
  let value = $state('');

  function handleInput(e) {
    value = e.target.value;
  }

  function handleSubmit() {
    console.log('Submitted:', value);
  }
</script>

<!-- 方式1：内联处理 -->
<input value={value} oninput={handleInput} />

<!-- 方式2：Svelte 5 原生事件（与 Web 标准一致） -->
<input bind:value />

<!-- 方式3：表单提交 -->
<form onsubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
  <button type="submit">Submit</button>
</form>
```

### 双向绑定

```svelte
<script>
  let name = $state('');
  let options = $state(['a', 'b', 'c']);
  let selected = $state('');
</script>

<!-- 文本输入 -->
<input bind:value={name} />
<p>Hello, {name}!</p>

<!-- Select -->
<select bind:value={selected}>
  {#each options as option}
    <option value={option}>{option}</option>
  {/each}
</select>
```

### 条件渲染

```svelte
<script>
  let isLoggedIn = $state(false);
  let items = $state([1, 2, 3]);
</script>

<!-- if/else -->
{#if isLoggedIn}
  <p>Welcome!</p>
{:else}
  <p>Please login.</p>
{/if}

<!-- each 循环 -->
{#each items as item, index}
  <p>{index + 1}. Item {item}</p>
{/each}

<!-- key 用于重渲染 -->
{#each items as item (item.id)}
  <TodoItem {item} />
{/each}
```

---

## Svelte Store

Svelte Store 提供了一种跨组件共享状态的方式。

### 基础 Store

```javascript
// stores/counter.js
import { writable } from 'svelte/store';

export const counter = writable(0);

// 使用 subscribe 手动订阅
import { counter } from './stores/counter';

counter.subscribe(value => {
  console.log('Count:', value);
});

// 自动订阅（$ 前缀）
$counter; // 读取值
$counter = 5; // 设置值
```

### Derived Store

```javascript
import { writable, derived } from 'svelte/store';

export const firstName = writable('John');
export const lastName = writable('Doe');

// 派生 store
export const fullName = derived(
  [firstName, lastName],
  ([$firstName, $lastName]) => `${$firstName} ${$lastName}`
);

// 在组件中使用
<script>
  import { fullName } from './stores';
</script>

<p>{$fullName}</p>
```

### Readable Store

```javascript
import { readable } from 'svelte/store';

// 不可变的 store，外部无法直接修改
export const time = readable(new Date(), (set) => {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);

  // 清理函数
  return () => clearInterval(interval);
});
```

### 自定义 Store

```javascript
function createCounter() {
  const { subscribe, set, update } = writable(0);

  return {
    subscribe,
    increment: () => update(n => n + 1),
    decrement: () => update(n => n - 1),
    reset: () => set(0)
  };
}

export const counter = createCounter();
```

### 在组件中使用 Store

```svelte
<script>
  import { counter } from './stores';
  import { fullName } from './stores';

  let count = $state(0);
</script>

<!-- 访问 Store -->
<p>Count: {$counter}</p>
<p>Name: {$fullName}</p>

<!-- 更新 Store -->
<button onclick={() => $counter += 1}>
  Increment
</button>
```

---

## Transition 与 Animation

Svelte 内置了强大的过渡和动画系统。

### 过渡效果

```svelte
<script>
  import { fade, slide, fly } from 'svelte/transition';

  let visible = $state(true);
</script>

<button onclick={() => visible = !visible}>
  Toggle
</button>

{#if visible}
  <!-- fade 淡入淡出 -->
  <p transition:fade>I will fade</p>

  <!-- slide 滑动 -->
  <div transition:slide>I will slide</div>

  <!-- fly 飞入 -->
  <div transition:fly={{ y: 200, duration: 1000 }}>
    I will fly in
  </div>
{/if}
```

### 自定义过渡

```svelte
<script>
  import { cubicOut } from 'svelte/easing';

  function spin(node, { duration, degrees }) {
    return {
      duration,
      css: (t) => {
        // cubicOut 缓动函数
        const eased = cubicOut(t);
        return `
          transform: rotate(${eased * degrees}deg);
          opacity: ${eased};
        `;
      }
    };
  }
</script>

<div transition:spin={{ duration: 1000, degrees: 360 }}>
  Spinning!
</div>
```

### 动画效果

```svelte
<script>
  import { flip } from 'svelte/animate';
  import { fade } from 'svelte/transition';

  let items = $state([1, 2, 3, 4, 5]);

  function addItem() {
    items = [...items, items.length + 1];
  }

  function removeItem(item) {
    items = items.filter(i => i !== item);
  }
</script>

<button onclick={addItem}>Add</button>

{#each items as item (item)}
  <div
    animate:flip={{ duration: 300 }}
    out:fade={{ duration: 200 }}
  >
    {item}
    <button onclick={() => removeItem(item)}>×</button>
  </div>
{/each}
```

### 页面过渡

```svelte
<!-- +layout.svelte -->
<script>
  import { page } from '$app/stores';
  import { fade } from 'svelte/transition';
</script>

{#key page.url.pathname}
  <main in:fade={{ duration: 200, delay: 200 }}>
    <slot />
  </main>
{/key}
```

---

## SvelteKit 全栈框架

### SvelteKit 概述

SvelteKit 是 Svelte 的官方全栈框架，提供了文件系统路由、SSR/SSG、API 端点等功能。

### 项目结构

```
my-app/
├── src/
│   ├── routes/
│   │   ├── +layout.svelte    # 布局组件
│   │   ├── +layout.server.js # 服务端布局
│   │   ├── +page.svelte      # 页面组件
│   │   ├── +page.server.js   # 服务端数据加载
│   │   ├── +page.js          # 通用数据加载
│   │   └── api/
│   │       └── users/
│   │           └── +server.js # API 端点
│   ├── lib/
│   │   ├── components/       # 组件
│   │   └── stores/           # Store
│   └── app.html
├── static/                   # 静态资源
├── svelte.config.js
└── vite.config.js
```

### 数据加载

**服务端加载（+page.server.js）：**

```javascript
// routes/users/+page.server.js
export async function load({ fetch, params }) {
  const response = await fetch(`/api/users/${params.id}`);
  const user = await response.json();

  return {
    user
  };
}
```

**客户端加载（+page.js）：**

```javascript
// routes/users/+page.js
export async function load({ fetch }) {
  const response = await fetch('/api/users');
  const users = await response.json();

  return { users };
}
```

**在页面中使用（+page.svelte）：**

```svelte
<script>
  let { data } = $props();
  // data 来自 load 函数的返回值
</script>

<h1>Users</h1>
{#each data.users as user}
  <p>{user.name}</p>
{/each}
```

### 表单处理

**服务端（+page.server.js）：**

```javascript
import { fail } from '@sveltejs/kit';

export const actions = {
  default: async ({ request }) => {
    const formData = await request.formData();
    const name = formData.get('name');

    if (!name) {
      return fail(400, { name, missing: true });
    }

    return { success: true };
  }
};
```

**客户端（+page.svelte）：**

```svelte
<script>
  import { enhance } from '$app/forms';
  import { goto } from '$app/navigation';

  let { form } = $props();
</script>

<form method="POST" use:enhance={() => {
  return async ({ result, update }) => {
    if (result.type === 'success') {
      await goto('/success');
    }
    update();
  };
}}>
  <input type="text" name="name" />
  {#if form?.missing}
    <p>Name is required</p>
  {/if}
  <button type="submit">Submit</button>
</form>
```

### API 端点

```javascript
// routes/api/users/+server.js
import { json } from '@sveltejs/kit';

export async function GET({ url }) {
  const page = parseInt(url.searchParams.get('page') || '1');
  const limit = parseInt(url.searchParams.get('limit') || '10');

  const users = await getUsers({ page, limit });

  return json({
    users,
    page,
    total: users.length
  });
}

export async function POST({ request }) {
  const body = await request.json();
  const user = await createUser(body);

  return json(user, { status: 201 });
}
```

### 部署适配器

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-auto';
// 或使用其他适配器：
// import adapter from '@sveltejs/adapter-vercel';
// import adapter from '@sveltejs/adapter-cloudflare';
// import adapter from '@sveltejs/adapter-node';

export default {
  kit: {
    adapter: adapter()
  }
};
```

---

## Svelte vs React vs Vue vs Solid 对比

### 核心哲学对比

| 框架 | 核心理念 | 反应性模型 | DOM 方式 |
|------|---------|-----------|---------|
| **Svelte** | 编译时优化 | 编译时 Runes | 直接操作 |
| **React** | 声明式 UI | 虚拟 DOM + Hooks | 虚拟 DOM diffing |
| **Vue** | 渐进式框架 | 运行时 Proxy | 虚拟 DOM |
| **Solid** | 精细响应性 | 编译时 Signals | 直接操作 |

### 功能对比表

| 特性 | Svelte 5 | React 18 | Vue 3 | Solid |
|------|---------|----------|-------|-------|
| **学习曲线** | 低 | 中 | 低 | 中 |
| **TypeScript 支持** | 优秀 | 优秀 | 优秀 | 优秀 |
| **包体积** | 最小 | 较大 | 中等 | 极小 |
| **运行时开销** | 零 | 中等 | 较小 | 零 |
| **首屏性能** | 最快 | 中等 | 中等 | 最快 |
| **更新性能** | 极快 | 需优化 | 快 | 最快 |
| **SSR 支持** | SvelteKit | Next.js | Nuxt | SolidStart |
| **状态管理** | Store/Runes | Context/Redux | Pinia | createSignal |
| **生态丰富度** | 中等 | 丰富 | 丰富 | 较少 |
| **社区规模** | 增长中 | 最大 | 大 | 较小 |

### 代码风格对比

**React：**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

**Vue 3：**

```vue
<script setup>
import { ref } from 'vue';

const count = ref(0);
</script>

<template>
  <button @click="count++">
    Count: {{ count }}
  </button>
</template>
```

**Svelte 5：**

```svelte
<script>
  let count = $state(0);
</script>

<button onclick={() => count++}>
  Count: {count}
</button>
```

**Solid：**

```jsx
import { createSignal } from 'solid-js';

function Counter() {
  const [count, setCount] = createSignal(0);

  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count()}
    </button>
  );
}
```

### 性能基准测试

> [!NOTE]
> 以下数据基于 js-framework-benchmark 2026 年测试结果。

| 操作类型 | Svelte 5 | Solid | Vue 3 | React 18 |
|---------|---------|-------|-------|----------|
| 创建 1000 行 | 1.2ms | 1.0ms | 1.5ms | 2.3ms |
| 替换 1000 行 | 2.1ms | 1.8ms | 2.4ms | 3.5ms |
| 选择性更新 | 0.8ms | 0.7ms | 1.0ms | 1.8ms |
| 内存占用 | 最低 | 最低 | 中等 | 较高 |

---

## Svelte 在 AI 应用中的定位

### 为什么 AI 应用适合 Svelte

1. **极致性能**：AI 应用通常需要频繁的状态更新和 DOM 操作，Svelte 的编译时优化提供了最好的性能基础
2. **极小包体积**：AI 应用需要加载大模型或 API 客户端，包体积优化至关重要
3. **简洁语法**：AI 代码生成更容易生成正确的 Svelte 语法
4. **快速迭代**：Svelte 的开发体验（SSR、HMR）适合 AI 应用的快速原型开发

### Svelte + AI 应用示例

**1. AI Chat 界面：**

```svelte
<script>
  import { streamText } from 'ai';

  let messages = $state([]);
  let input = $state('');

  async function handleSubmit() {
    const userMessage = { role: 'user', content: input };
    messages = [...messages, userMessage];

    const result = await streamText({
      model: openai('gpt-4o'),
      messages: messages,
      onChunk: (chunk) => {
        // 流式更新
        messages = messages.map((m, i) =>
          i === messages.length - 1
            ? { ...m, content: m.content + chunk }
            : m
        );
      }
    });
  }
</script>

<div class="chat">
  {#each messages as message}
    <div class={message.role}>
      {message.content}
    </div>
  {/each}

  <form onsubmit={(e) => { e.preventDefault(); handleSubmit(); }}>
    <input bind:value={input} />
    <button type="submit">Send</button>
  </form>
</div>
```

**2. AI 图像生成界面：**

```svelte
<script>
  let prompt = $state('');
  let imageUrl = $state(null);
  let loading = $state(false);

  async function generate() {
    loading = true;
    const result = await generateImage(prompt);
    imageUrl = result.url;
    loading = false;
  }
</script>

<textarea bind:value={prompt} placeholder="Describe your image..." />
<button onclick={generate} disabled={loading}>
  {loading ? 'Generating...' : 'Generate'}
</button>

{#if imageUrl}
  <img src={imageUrl} alt="Generated" />
{/if}
```

### SvelteKit + AI API 集成

```javascript
// routes/api/generate/+server.js
import { json } from '@sveltejs/kit';
import { openai } from '$lib/ai';

export async function POST({ request }) {
  const { prompt } = await request.json();

  const completion = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [{ role: 'user', content: prompt }]
  });

  return json({
    content: completion.choices[0].message.content
  });
}
```

---

## 选型建议与实战场景

### 选型决策树

```
项目需求分析
    │
    ├── 是否需要极致性能？
    │   ├── 是 → Svelte 5 或 Solid
    │   └── 否 → 继续分析
    │
    ├── 团队技术栈背景？
    │   ├── React 团队 → Next.js + React
    │   ├── Vue 团队 → Nuxt + Vue
    │   ├── 全栈团队 → SvelteKit 或 Nuxt
    │   └── 追求创新 → Svelte 5
    │
    ├── 项目规模？
    │   ├── 小型/原型 → Svelte 5（快速开发）
    │   ├── 中型 → Vue 3 / React 18
    │   └── 大型 → Next.js / Nuxt
    │
    └── AI 应用？
        ├── 是 → Svelte 5（性能优先）或 Next.js（生态优先）
        └── 否 → 根据团队选择
```

### 场景推荐

| 场景 | 推荐选择 | 原因 |
|------|---------|------|
| **AI Chat 应用** | Svelte 5 / Next.js | 高频更新，需要极致性能 |
| **AI 图像生成** | Svelte 5 | 快速迭代，包体积小 |
| **企业后台** | React / Vue | 生态丰富，组件多 |
| **内容网站** | SvelteKit / Nuxt | SSR 支持好，开发体验佳 |
| **电商平台** | Next.js | SEO 优化成熟 |
| **SaaS 产品** | React / SvelteKit | 长期维护，可扩展性 |
| **移动端 Web** | Svelte + Capacitor | 性能优势明显 |

### AI 编程场景下的 Svelte 优势

1. **语法简洁**：AI 生成的代码更容易正确
2. **编译时检查**：减少运行时错误
3. **响应式直观**：`$state`/`$derived` 语义清晰
4. **生态整合**：SvelteKit 支持边缘部署，适合 AI 应用

---

> [!SUCCESS]
> Svelte 5 以 Runes 语法重新定义了前端框架的反应性模型。其编译时优化带来的极致性能和极小包体积，使其成为 AI 应用开发的优秀选择。对于追求开发效率和运行性能的团队，Svelte 5 是一个值得认真考虑的技术选项。

---

## 核心理念与设计哲学

### Svelte 的设计哲学

Svelte 的设计哲学建立在"编译时优于运行时"的核心原则上。

**1. 编译时反应性**

Svelte 将尽可能多的工作移到编译阶段：

```
┌─────────────────────────────────────────────────────────────┐
│                   Svelte 编译流程                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  .svelte 文件                                              │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           Svelte 编译器                              │  │
│  │  ┌─────────────────────────────────────────────┐    │  │
│  │  │  1. 解析：源代码 → AST                      │    │  │
│  │  │  2. 分析：依赖关系、响应式追踪               │    │  │
│  │  │  3. 生成：优化的 JavaScript 代码            │    │  │
│  │  └─────────────────────────────────────────────┘    │  │
│  └─────────────────────────────────────────────────────┘  │
│       │                                                    │
│       ▼                                                    │
│  优化后的 JavaScript 代码                                   │
│  - 精确的 DOM 操作                                         │
│  - 极小的运行时                                             │
│  - 无虚拟 DOM 开销                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**2. 真正的反应性**

Svelte 的反应性基于赋值语句，而非状态 API：

```svelte
<script>
  // Svelte 5 Runes 语法
  let count = $state(0);
  
  // 简单赋值即触发更新
  function increment() {
    count += 1; // 自动追踪并更新 DOM
  }
  
  // 计算属性
  let doubled = $derived(count * 2);
  
  // 副作用
  $effect(() => {
    console.log('count changed:', count);
  });
</script>

<button onclick={increment}>
  Count: {count}, Doubled: {doubled}
</button>
```

**3. 极小包体积**

Svelte 的运行时极小，带来极快的加载速度：

| 框架 | 运行时大小 | 编译后增量 |
|------|-----------|-----------|
| Svelte 5 | ~4KB | 极小 |
| Solid.js | ~7KB | 极小 |
| Vue 3 | ~35KB | 中等 |
| React 18 | ~45KB | 较大 |

**4. 直观的模板语法**

```svelte
<script>
  let users = $state([
    { id: 1, name: 'Alice', active: true },
    { id: 2, name: 'Bob', active: false },
    { id: 3, name: 'Charlie', active: true }
  ]);
  
  let filter = $state('all');
  
  let filteredUsers = $derived(
    filter === 'all' 
      ? users 
      : users.filter(u => u.active === (filter === 'active'))
  );
</script>

<!-- 简洁的条件渲染 -->
{#if filter !== 'all'}
  <button onclick={() => filter = 'all'}>显示全部</button>
{/if}

<!-- 列表渲染 -->
{#each filteredUsers as user (user.id)}
  <div class:highlighted={user.active}>
    <span>{user.name}</span>
  </div>
{/each}

<!-- 内联条件 -->
<p>{users.length} 用户{users.length === 1 ? '' : 's'}</p>
```

### Svelte 解决的问题域

| 问题域 | Svelte 解决方案 | 优势 |
|--------|---------------|------|
| **性能** | 编译时优化 | 极快的运行时性能 |
| **包体积** | 极小运行时 | 快速首屏加载 |
| **开发体验** | 简洁语法 | 快速迭代 |
| **AI 代码生成** | 简洁语法 | AI 更易生成正确代码 |
| **高频更新** | 精确 DOM 更新 | 流畅的 UI |

---

## 完整安装与项目创建

### 环境准备

**Node.js 版本要求：**
- Svelte 5 需要 Node.js 18 或更高版本
- 推荐使用 Node.js 20 LTS

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 安装 pnpm
npm install -g pnpm
```

### 创建 Svelte 项目的多种方式

**方式一：SvelteKit（推荐）**

```bash
# 创建 SvelteKit 项目
npm create svelte@latest my-app

# 选项：
# Which Svelte app template? → Skeleton project
# Add type checking with TypeScript? → Yes
# Select additional options → 根据需要选择

# 进入目录
cd my-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview
```

**方式二：Vite + Svelte**

```bash
# 创建 Vite + Svelte 项目
npm create vite@latest my-svelte-app -- --template svelte-ts

# 进入目录
cd my-svelte-app

# 安装依赖
pnpm install

# 启动
pnpm dev
```

**方式三：在线体验**

- Svelte REPL：https://svelte.dev/repl
- StackBlitz：https://stackblitz.com/fork/svelte

### 项目结构详解

**SvelteKit 项目结构：**

```
my-app/
├── src/
│   ├── routes/            # 文件系统路由
│   │   ├── +layout.svelte    # 布局
│   │   ├── +layout.server.ts # 服务端布局
│   │   ├── +page.svelte      # 页面
│   │   ├── +page.server.ts   # 服务端数据加载
│   │   ├── +page.ts         # 通用数据加载
│   │   ├── +error.svelte    # 错误页面
│   │   └── api/
│   │       └── users/
│   │           └── +server.ts # API 端点
│   ├── lib/               # 库代码
│   │   ├── components/    # 组件
│   │   │   ├── Button.svelte
│   │   │   └── UserCard.svelte
│   │   ├── stores/        # Svelte stores
│   │   │   └── counter.ts
│   │   ├── utils/         # 工具函数
│   │   └── server/        # 仅服务端代码
│   │       └── db.ts
│   ├── app.html           # HTML 模板
│   ├── app.css            # 全局样式
│   └── app.d.ts          # 类型声明
├── static/                # 静态资源
├── svelte.config.js      # SvelteKit 配置
├── vite.config.ts        # Vite 配置
└── tsconfig.json         # TypeScript 配置
```

**独立 Svelte 项目结构：**

```
my-svelte-app/
├── src/
│   ├── lib/
│   │   ├── components/
│   │   └── stores/
│   ├── App.svelte        # 根组件
│   └── main.ts           # 入口
├── public/
│   └── favicon.png
├── index.html
├── package.json
├── svelte.config.js
├── vite.config.ts
└── tsconfig.json
```

---

## 组件系统详解

### Svelte 5 Runes 语法

**$state - 反应性状态：**

```svelte
<script>
  // 基本类型
  let count = $state(0);
  let name = $state('Alice');
  
  // 对象类型
  let user = $state({
    name: 'Bob',
    age: 30
  });
  
  // 数组类型
  let items = $state<string[]>([]);
  
  // 函数更新
  function increment() {
    count += 1;
  }
  
  function addItem(item: string) {
    items = [...items, item];
  }
  
  function updateUser() {
    user = { ...user, age: user.age + 1 };
  }
</script>

<button onclick={increment}>
  Count: {count}
</button>

<p>Name: {user.name}, Age: {user.age}</p>

<ul>
  {#each items as item}
    <li>{item}</li>
  {/each}
</ul>
```

**$derived - 计算属性：**

```svelte
<script>
  let price = $state(100);
  let quantity = $state(2);
  let discount = $state(0.1);
  
  // 简单派生
  let subtotal = $derived(price * quantity);
  
  // 复杂派生
  let total = $derived(
    price * quantity * (1 - discount)
  );
  
  let formattedPrice = $derived(
    new Intl.NumberFormat('zh-CN', {
      style: 'currency',
      currency: 'CNY'
    }).format(total)
  );
  
  // 派生数组
  let tags = $state(['active', 'featured']);
  let filteredTags = $derived(
    tags.filter(t => t.startsWith('a'))
  );
</script>

<p>小计: {subtotal}</p>
<p>总计（含折扣）: {formattedPrice}</p>
<p>过滤标签: {filteredTags.join(', ')}</p>
```

**$effect - 副作用：**

```svelte
<script>
  let count = $state(0);
  let input = $state('');
  
  // 基本副作用
  $effect(() => {
    console.log('count changed:', count);
    
    // 清理函数
    return () => {
      console.log('cleanup');
    };
  });
  
  // 追踪多个依赖
  $effect(() => {
    document.title = `${count} - ${input}`;
  });
  
  // DOM 操作
  let inputEl: HTMLInputElement;
  
  $effect(() => {
    inputEl?.focus();
  });
</script>

<input bind:this={inputEl} bind:value={input} />
<button onclick={() => count += 1}>Increment</button>
```

**$props - 属性声明：**

```svelte
<!-- Button.svelte -->
<script>
  let { 
    variant = 'primary', 
    size = 'md',
    disabled = false,
    onclick,
    children
  } = $props();
  
  // 派生 props
  let { items } = $props();
  let itemCount = $derived(items.length);
</script>

<button
  class="btn btn-{variant} btn-{size}"
  class:disabled
  {disabled}
  {onclick}
>
  {@render children()}
</button>

<p>{itemCount} items</p>

<!-- 使用 -->
<Button variant="secondary" size="lg">
  <span>Click me</span>
</Button>
```

### 事件处理

**DOM 事件：**

```svelte
<script>
  let value = $state('');
  
  function handleInput(e: Event) {
    const target = e.target as HTMLInputElement;
    value = target.value;
  }
  
  function handleSubmit(e: SubmitEvent) {
    e.preventDefault();
    console.log('Submitted:', value);
  }
  
  function handleKeydown(e: KeyboardEvent) {
    if (e.key === 'Enter') {
      console.log('Enter pressed');
    }
  }
</script>

<!-- 方式1：内联事件 -->
<input value={value} oninput={handleInput} />

<!-- 方式2：bind:value -->
<input bind:value />

<!-- 方式3：表单提交 -->
<form onsubmit={handleSubmit}>
  <button type="submit">提交</button>
</form>

<!-- 方式4：键盘事件 -->
<input onkeydown={handleKeydown} />
```

**组件事件：**

```svelte
<!-- EventEmitter.svelte -->
<script>
  let { onCustom, onChange } = $props();
</script>

<button onclick={() => onCustom('hello')}>发送事件</button>
<button onclick={() => onChange('world')}>发送变化</button>

<!-- Parent.svelte -->
<script>
  let message = $state('');
  
  function handleCustom(data: string) {
    message = data;
  }
</script>

<EventEmitter onCustom={handleCustom} />
<p>{message}</p>
```

### 绑定系统

**双向绑定：**

```svelte
<script>
  // 文本输入
  let text = $state('');
  
  // 复选框
  let checked = $state(false);
  
  // 多选
  let selected = $state<string[]>([]);
  
  // 下拉选择
  let city = $state('');
  
  // 数值
  let count = $state(0);
  
  // 文件输入
  let files = $state<FileList | null>(null);
</script>

<!-- 文本 -->
<input bind:value={text} />
<p>{text}</p>

<!-- 复选框 -->
<input type="checkbox" bind:checked={checked} />
<p>{checked ? '选中' : '未选中'}</p>

<!-- 多选 -->
{#each ['apple', 'banana', 'orange'] as fruit}
  <label>
    <input type="checkbox" bind:group={selected} value={fruit} />
    {fruit}
  </label>
{/each}
<p>选择: {selected.join(', ')}</p>

<!-- 下拉 -->
<select bind:value={city}>
  <option value="">请选择</option>
  <option value="beijing">北京</option>
  <option value="shanghai">上海</option>
</select>

<!-- 数值 -->
<input type="number" bind:value={count} />
<button onclick={() => count += 1}>+1</button>
```

**元素引用：**

```svelte
<script>
  let inputEl: HTMLInputElement;
  let canvasEl: HTMLCanvasElement;
  let count = $state(0);
  
  $effect(() => {
    // focus 输入框
    inputEl?.focus();
  });
  
  function draw() {
    const ctx = canvasEl?.getContext('2d');
    ctx?.clearRect(0, 0, 200, 200);
    ctx?.fillRect(0, 0, count, count);
  }
</script>

<input bind:this={inputEl} />
<canvas bind:this={canvasEl} width="200" height="200" />

<button onclick={() => { count += 10; draw(); }}>
  绘制
</button>
```

---

## Store 系统详解

### Svelte Store 基础

**Writable Store：**

```javascript
// stores/counter.js
import { writable } from 'svelte/store';

export const counter = writable(0);

// 创建带方法的 store
function createCounter() {
  const { subscribe, set, update } = writable(0);
  
  return {
    subscribe,
    increment: () => update(n => n + 1),
    decrement: () => update(n => n - 1),
    reset: () => set(0)
  };
}

export const counterStore = createCounter();
```

**Readable Store：**

```javascript
import { readable } from 'svelte/store';

// 不可变 store，外部无法直接修改
export const time = readable(new Date(), (set) => {
  const interval = setInterval(() => {
    set(new Date());
  }, 1000);
  
  return () => clearInterval(interval);
});

// 窗口大小 store
export const windowSize = readable(
  { width: 0, height: 0 },
  (set) => {
    const update = () => set({
      width: window.innerWidth,
      height: window.innerHeight
    });
    
    update();
    window.addEventListener('resize', update);
    
    return () => window.removeEventListener('resize', update);
  }
);
```

**Derived Store：**

```javascript
import { writable, derived } from 'svelte/store';

export const firstName = writable('John');
export const lastName = writable('Doe');

// 派生 store
export const fullName = derived(
  [firstName, lastName],
  ([$firstName, $lastName]) => `${$firstName} ${$lastName}`
);

// 派生对象
export const userProfile = derived(
  [firstName, lastName],
  ([$firstName, $lastName]) => ({
    name: `${$firstName} ${$lastName}`,
    initials: `${$firstName[0]}${$lastName[0]}`
  })
);
```

### 在组件中使用 Store

```svelte
<script>
  import { counter, fullName, userProfile } from './stores';
  
  let count = $state(0);
</script>

<!-- 使用 $ 前缀自动订阅 -->
<p>Count: {$counter}</p>
<p>Full Name: {$fullName}</p>

<!-- 更新 store -->
<button onclick={() => counter.increment()}>+1</button>

<!-- 派生值 -->
<p>Initials: {$userProfile.initials}</p>
```

---

## 路由系统详解

### SvelteKit 路由

**基础路由：**

```
src/routes/
├── +layout.svelte        # 根布局
├── +page.svelte          # / 首页
├── about/
│   └── +page.svelte      # /about
├── users/
│   ├── +page.svelte      # /users
│   ├── +page.server.ts   # 服务端加载
│   └── [id]/
│       └── +page.svelte  # /users/:id
└── +error.svelte         # 错误页面
```

**页面组件：**

```svelte
<!-- +page.svelte -->
<script>
  let { data } = $props();
</script>

<h1>Users</h1>

<ul>
  {#each data.users as user}
    <li>
      <a href="/users/{user.id}">{user.name}</a>
    </li>
  {/each}
</ul>

<!-- 服务端数据加载 -->
```

```typescript
// +page.server.ts
export async function load() {
  const users = await db.users.findMany();
  return { users };
}
```

**布局组件：**

```svelte
<!-- +layout.svelte -->
<script>
  import { page } from '$app/stores';
  
  let { children } = $props();
</script>

<div class="app">
  <nav>
    <a href="/" class:active={$page.url.pathname === '/'}>
      首页
    </a>
    <a href="/about" class:active={$page.url.pathname === '/about'}>
      关于
    </a>
  </nav>
  
  <main>
    {@render children()}
  </main>
</div>

<style>
  .app {
    display: flex;
    flex-direction: column;
    min-height: 100vh;
  }
  
  nav {
    display: flex;
    gap: 1rem;
    padding: 1rem;
  }
  
  main {
    flex: 1;
    padding: 1rem;
  }
  
  .active {
    font-weight: bold;
  }
</style>
```

**动态路由：**

```svelte
<!-- src/routes/users/[id]/+page.svelte -->
<script>
  import { page } from '$app/stores';
  
  let { data } = $props();
  
  $effect(() => {
    console.log('User ID:', $page.params.id);
  });
</script>

<h1>User: {data.user.name}</h1>
<p>Email: {data.user.email}</p>
```

```typescript
// src/routes/users/[id]/+page.server.ts
export async function load({ params }) {
  const user = await db.users.findUnique({
    where: { id: params.id }
  });
  
  if (!user) {
    throw error(404, 'User not found');
  }
  
  return { user };
}
```

### 表单处理

**服务端处理：**

```typescript
// src/routes/+page.server.ts
import { fail } from '@sveltejs/kit';

export const actions = {
  default: async ({ request }) => {
    const data = await request.formData();
    const name = data.get('name');
    const email = data.get('email');
    
    if (!name || !email) {
      return fail(400, { 
        name, 
        email, 
        missing: true 
      });
    }
    
    await db.users.create({ name, email });
    
    return { success: true };
  }
};
```

**客户端使用：**

```svelte
<!-- +page.svelte -->
<script>
  import { enhance } from '$app/forms';
  
  let { form } = $props();
</script>

{#if form?.success}
  <p>创建成功！</p>
{/if}

<form method="POST" use:enhance>
  <div>
    <label for="name">姓名</label>
    <input 
      id="name" 
      name="name" 
      value={form?.name ?? ''}
    />
    {#if form?.missing && !form?.name}
      <span class="error">请输入姓名</span>
    {/if}
  </div>
  
  <div>
    <label for="email">邮箱</label>
    <input 
      id="email" 
      name="email" 
      type="email"
      value={form?.email ?? ''}
    />
  </div>
  
  <button type="submit">提交</button>
</form>
```

---

## 样式方案详解

### Scoped Styles

```svelte
<style>
  .card {
    padding: 1rem;
    border: 1px solid #e5e7eb;
    border-radius: 8px;
  }
  
  /* 伪类 */
  .button:hover {
    background: #f3f4f6;
  }
  
  /* 媒体查询 */
  @media (max-width: 768px) {
    .card {
      padding: 0.5rem;
    }
  }
  
  /* 嵌套 */
  .card {
    .title {
      font-size: 1.25rem;
    }
    
    &.highlighted {
      border-color: #3b82f6;
    }
  }
</style>
```

### Tailwind CSS

```bash
pnpm add -D tailwindcss autoprefixer postcss
npx tailwindcss init -p
```

```svelte
<script>
  let { name, role = 'user' } = $props();
  
  const roleStyles = {
    admin: 'bg-red-100 text-red-800',
    user: 'bg-blue-100 text-blue-800',
    guest: 'bg-gray-100 text-gray-800'
  };
</script>

<div class="flex items-center gap-4 p-4 bg-white rounded-lg shadow">
  <div class="w-12 h-12 bg-gray-200 rounded-full flex items-center justify-center">
    {name.charAt(0)}
  </div>
  
  <div class="flex-1">
    <h3 class="font-semibold">{name}</h3>
  </div>
  
  <span class="px-2 py-1 text-xs font-medium rounded {roleStyles[role]}">
    {role}
  </span>
</div>
```

### CSS 变量

```svelte
<script>
  let { theme = 'light' } = $props();
</script>

<div class="container" data-theme={theme}>
  <slot />
</div>

<style>
  :global(:root) {
    --color-primary: #3b82f6;
    --color-background: #ffffff;
    --color-text: #1f2937;
  }
  
  :global([data-theme="dark"]) {
    --color-background: #1f2937;
    --color-text: #f9fafb;
  }
  
  .container {
    background: var(--color-background);
    color: var(--color-text);
  }
</style>
```

---

## 性能优化详解

### 编译时优化

**静态提升：**

```svelte
<script>
  let count = $state(0);
</script>

<!-- 静态内容在编译时被提升 -->
<h1>Welcome to My App</h1>
<p>This is a static paragraph that won't change.</p>

<!-- 动态内容 -->
<p>Count: {count}</p>
<button onclick={() => count += 1}>+1</button>
```

**精确更新：**

```svelte
<script>
  let user = $state({
    name: 'Alice',
    profile: {
      bio: 'Developer',
      avatar: 'url'
    }
  });
  
  function updateName() {
    // 只更新 name，不影响 profile
    user.name = 'Bob';
  }
</script>

<!-- 只有 name 部分更新 -->
<h1>{user.name}</h1>
<p>{user.profile.bio}</p>
```

### 懒加载

**动态导入：**

```svelte
<script>
  const HeavyComponent = $derived(
    () => import('./HeavyComponent.svelte')
  );
  
  let show = $state(false);
</script>

<button onclick={() => show = true}>
  加载重型组件
</button>

{#if show}
  {#await import('./HeavyComponent.svelte')}
    <p>加载中...</p>
  {:then module}
    <svelte:component this={module.default} />
  {/await}
{/if}
```

**路由级懒加载（SvelteKit）：**

```typescript
// SvelteKit 自动懒加载路由
// src/routes/+page.svelte
// 自动按需加载
```

---

## 生命周期详解

### Svelte 5 生命周期

| 生命周期 | 说明 |
|---------|------|
| `$effect` | 副作用，在组件初始化和依赖变化时运行 |
| `$effect.pre` | 前置副作用 |
| `$effect.cleanup` | 清理函数 |

**示例：**

```svelte
<script>
  import { onMount, onDestroy } from 'svelte';
  
  // SvelteKit/Svelte 的生命周期
  onMount(() => {
    console.log('组件挂载');
    
    return () => {
      console.log('组件卸载');
    };
  });
  
  onDestroy(() => {
    console.log('清理资源');
  });
  
  // $effect 替代 onMount + $:
  let count = $state(0);
  
  $effect(() => {
    console.log('count changed:', count);
    
    return () => {
      console.log('cleanup count effect');
    };
  });
</script>
```

---

## 与其他框架对比

### Svelte vs React

| 维度 | Svelte 5 | React 18 |
|------|----------|----------|
| **语法** | 模板 + Runes | JSX + Hooks |
| **反应性** | 编译时 | 运行时 |
| **包体积** | ~4KB | ~45KB |
| **学习曲线** | 低 | 中等 |
| **生态** | 中等 | 丰富 |

### Svelte vs Vue

| 维度 | Svelte 5 | Vue 3 |
|------|----------|-------|
| **语法** | 自定义模板 | HTML 模板 |
| **反应性** | Runes | Proxy |
| **编译** | 编译时 | 部分编译 |
| **包体积** | ~4KB | ~35KB |

### Svelte vs Solid

| 维度 | Svelte 5 | Solid.js |
|------|----------|----------|
| **语法** | 模板 | JSX |
| **反应性** | Runes | Signals |
| **生态** | SvelteKit | SolidStart |
| **学习曲线** | 低 | 中等 |

---

## 学习路径与资源

### Svelte 学习路线图

```
第一阶段：基础（1-2周）
├── Svelte 基础语法
├── Svelte 5 Runes
├── 组件基础
├── 事件处理
└── 样式

第二阶段：进阶（2-3周）
├── SvelteKit 路由
├── 表单处理
├── Stores
├── SSR/SSG
└── 部署

第三阶段：生态（2-3周）
├── SvelteKit进阶
├── 数据库集成
├── API 开发
├── 测试
└── 性能优化
```

### 推荐学习资源

**官方资源：**
- Svelte 官方文档：https://svelte.dev/docs
- SvelteKit 文档：https://kit.svelte.dev/docs
- Svelte REPL：https://svelte.dev/repl

**教程：**
- Svelte 官方教程：svelte.dev/tutorial
- Svelte Mastery
- LevelUp Tutorials

**社区：**
- Svelte 博客：https://svelte.dev/blog
- Svelte Discord

---

## 实战技巧

### AI 辅助开发

**Prompt 示例：**

```markdown
创建一个 Svelte 5 用户卡片组件：
- 使用 <script setup> 语法
- Props：
  - user: { id: string, name: string, email: string, avatar?: string }
  - size: 'sm' | 'md' | 'lg'
- 功能：
  - 显示头像（无头像显示首字母）
  - 角色标签
- 样式：使用 Tailwind CSS
```

### 常见问题解决

**1. 状态更新不触发重新渲染：**

```svelte
// ❌ 错误：直接修改数组
items.push(newItem);

// ✅ 正确：创建新引用
items = [...items, newItem];

// ✅ 正确：使用 index
items[index] = newItem;
```

**2. 对象更新：**

```svelte
// ❌ 错误：直接修改属性
user.name = 'new name';

// ✅ 正确：创建新对象
user = { ...user, name: 'new name' };
```

**3. Effect 依赖追踪：**

```svelte
// ✅ 正确：访问值以追踪
$effect(() => {
  console.log(count); // 追踪 count
  console.log(user.name); // 追踪 user.name
});

// ❌ 可能问题：没有追踪
$effect(() => {
  console.log('static'); // 无依赖
});
```

---

## 测试工具与实践

### 测试框架概述

Svelte 项目可以使用多种测试工具进行测试。

|| 测试类型 | 工具 | 说明 |
||---------|------|------|
|| **单元测试** | Vitest / Jest | 组件逻辑测试 |
|| **组件测试** | Testing Library | Svelte 组件交互测试 |
|| **端到端测试** | Playwright / Cypress | 完整用户流程 |

### Vitest 基础

**配置 Vitest：**

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import { svelte } from '@sveltejs/vite-plugin-svelte';

export default defineConfig({
  plugins: [svelte(), svelteTesting()],
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

### 组件测试（Testing Library）

**安装依赖：**

```bash
pnpm add -D @testing-library/svelte jsdom vitest
```

**基础组件测试：**

```typescript
// Button.test.ts
import { render, screen, fireEvent } from '@testing-library/svelte';
import { describe, it, expect, vi } from 'vitest';
import Button from './Button.svelte';

describe('Button Component', () => {
  it('should render with text', () => {
    render(Button, {
      props: {
        label: 'Click me',
        variant: 'primary'
      }
    });

    expect(screen.getByRole('button')).toBeInTheDocument();
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });

  it('should emit click event', async () => {
    const handleClick = vi.fn();
    render(Button, {
      props: {
        label: 'Click me',
        onClick: handleClick
      }
    });

    await fireEvent.click(screen.getByRole('button'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('should be disabled', () => {
    render(Button, {
      props: {
        label: 'Disabled',
        disabled: true
      }
    });

    expect(screen.getByRole('button')).toBeDisabled();
  });

  it('should apply variant class', () => {
    render(Button, {
      props: {
        label: 'Primary',
        variant: 'primary'
      }
    });

    expect(screen.getByRole('button')).toHaveClass('btn-primary');
  });
});
```

### 异步组件测试

**Suspense 组件测试：**

```typescript
// AsyncUserList.test.ts
import { render, screen, waitFor } from '@testing-library/svelte';
import { describe, it, expect, vi } from 'vitest';
import AsyncUserList from './AsyncUserList.svelte';

// Mock fetch
const mockUsers = [
  { id: '1', name: '张三' },
  { id: '2', name: '李四' }
];

global.fetch = vi.fn().mockResolvedValue({
  json: () => Promise.resolve(mockUsers)
}) as any;

describe('AsyncUserList', () => {
  it('should display users after loading', async () => {
    render(AsyncUserList);

    await waitFor(() => {
      expect(screen.getByText('张三')).toBeInTheDocument();
      expect(screen.getByText('李四')).toBeInTheDocument();
    });
  });

  it('should show loading state', () => {
    render(AsyncUserList);
    expect(screen.getByText('加载中...')).toBeInTheDocument();
  });
});
```

### Store 测试

**Store 测试：**

```typescript
// counter.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { counter, increment, decrement, reset } from './counter';

describe('Counter Store', () => {
  beforeEach(() => {
    reset(); // 重置状态
  });

  it('should have initial value of 0', () => {
    expect(counter).toBe(0);
  });

  it('should increment', () => {
    increment();
    expect(counter).toBe(1);
  });

  it('should decrement', () => {
    increment();
    increment();
    decrement();
    expect(counter).toBe(1);
  });

  it('should reset', () => {
    increment();
    increment();
    increment();
    reset();
    expect(counter).toBe(0);
  });
});
```

### 表单测试

**表单组件测试：**

```typescript
// UserForm.test.ts
import { render, screen, fireEvent } from '@testing-library/svelte';
import { describe, it, expect } from 'vitest';
import UserForm from './UserForm.svelte';

describe('UserForm', () => {
  it('should validate required fields', async () => {
    render(UserForm);

    const submitButton = screen.getByRole('button', { name: /提交/i });
    await fireEvent.click(submitButton);

    expect(screen.getByText('姓名不能为空')).toBeInTheDocument();
    expect(screen.getByText('邮箱不能为空')).toBeInTheDocument();
  });

  it('should submit valid form', async () => {
    const handleSubmit = vi.fn();
    render(UserForm, {
      props: {
        onSubmit: handleSubmit
      }
    });

    await fireEvent.input(screen.getByLabelText(/姓名/i), {
      target: { value: '张三' }
    });
    await fireEvent.input(screen.getByLabelText(/邮箱/i), {
      target: { value: 'zhangsan@example.com' }
    });

    await fireEvent.click(screen.getByRole('button', { name: /提交/i }));

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
    baseURL: 'http://localhost:4173',
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
    const todoInput = page.getByPlaceholder('添加待办事项...');
    await todoInput.fill('学习 Svelte');
    await page.getByRole('button', { name: '添加' }).click();

    await expect(page.getByText('学习 Svelte')).toBeVisible();
  });

  test('should toggle todo completion', async ({ page }) => {
    await page.getByPlaceholder('添加待办事项...').fill('测试任务');
    await page.getByRole('button', { name: '添加' }).click();

    const todoItem = page.getByText('测试任务');
    await todoItem.click();

    await expect(todoItem).toHaveClass(/completed/);
  });

  test('should delete a todo', async ({ page }) => {
    await page.getByPlaceholder('添加待办事项...').fill('待删除的任务');
    await page.getByRole('button', { name: '添加' }).click();

    const deleteButton = page.getByRole('button', { name: '删除' });
    await deleteButton.click();

    await expect(page.getByText('待删除的任务')).not.toBeVisible();
  });

  test('should filter todos', async ({ page }) => {
    // 添加不同状态的待办
    await page.getByPlaceholder('添加待办事项...').fill('已完成');
    await page.getByRole('button', { name: '添加' }).click();

    const todoItem = page.getByText('已完成');
    await todoItem.click();

    // 切换到完成筛选
    await page.getByRole('button', { name: '已完成' }).click();

    await expect(page.getByText('已完成')).toBeVisible();
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
        '**/*.test.ts'
      ]
    }
  }
});
```

### 测试最佳实践

|| 实践 | 说明 |
||-----|------|
|| **组件优先** | 优先测试用户可见的行为 |
|| **RTL 模式** | 使用 Testing Library 的查询优先顺序 |
|| **独立测试** | 每个测试独立运行，不依赖其他测试 |
|| **描述性命名** | 测试名称应描述预期行为 |
|| **AAA 模式** | Arrange → Act → Assert |
|| **模拟网络** | 使用 MSW 模拟 API 请求 |

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

---

## 高级模式与最佳实践

### Context API 深度使用

Svelte 的 Context API 用于跨组件通信：

```svelte
<!-- contexts/app-context.ts -->
import { getContext, setContext } from 'svelte'
import { writable } from 'svelte/store'

interface Theme {
  primary: string
  secondary: string
  mode: 'light' | 'dark'
}

const THEME_KEY = Symbol('theme')

export function setThemeContext(theme: Theme) {
  const themeStore = writable(theme)
  setContext(THEME_KEY, themeStore)
  return themeStore
}

export function getThemeContext() {
  return getContext<ReturnType<typeof writable<Theme>>>(THEME_KEY)
}

// App.svelte
<script lang="ts">
  import { setThemeContext } from './contexts/app-context'
  
  setThemeContext({
    primary: '#3b82f6',
    secondary: '#10b981',
    mode: 'light'
  })
</script>

<!-- Child.svelte -->
<script lang="ts">
  import { getThemeContext } from './contexts/app-context'
  
  const theme = getThemeContext()
  
  // 响应式订阅
  $: console.log($theme.mode)
</script>

<div style="color: {$theme.mode === 'dark' ? 'white' : 'black'}">
  Content
</div>
```

### 动态组件

Svelte 支持动态组件渲染：

```svelte
<script lang="ts">
  import HomePage from './pages/HomePage.svelte'
  import AboutPage from './pages/AboutPage.svelte'
  import ContactPage from './pages/ContactPage.svelte'
  
  type ComponentType = typeof HomePage | typeof AboutPage | typeof ContactPage
  
  let currentPage: ComponentType = HomePage
  let pageName = $state('home')
  
  const pages: Record<string, ComponentType> = {
    home: HomePage,
    about: AboutPage,
    contact: ContactPage
  }
  
  function navigateTo(page: string) {
    pageName = page
    currentPage = pages[page]
  }
</script>

<button onclick={() => navigateTo('home')}>首页</button>
<button onclick={() => navigateTo('about')}>关于</button>
<button onclick={() => navigateTo('contact')}>联系</button>

<svelte:component this={currentPage} />

<!-- 或者使用 -->
<component this={pages[pageName]} />
```

### 过渡与动画深度

Svelte 过渡系统详解：

```svelte
<script lang="ts">
  import { fade, fly, slide, scale, draw } from 'svelte/transition'
  import { elasticOut, bounceOut, quintOut } from 'svelte/easing'
  
  let visible = $state(true)
  let items = $state([1, 2, 3, 4, 5])
  
  function addItem() {
    items = [...items, items.length + 1]
  }
  
  function removeItem(index: number) {
    items = items.filter((_, i) => i !== index)
  }
  
  // 自定义过渡
  function typewriter(node: HTMLElement, { speed = 50, delay = 0 }) {
    const text = node.textContent || ''
    const duration = text.length * speed
    
    return {
      delay,
      duration,
      tick: (t: number) => {
        const i = Math.floor(text.length * t)
        node.textContent = text.slice(0, i)
      }
    }
  }
  
  // 翻转动画
  import { flip } from 'svelte/animate'
</script>

<!-- 淡入淡出 -->
<button onclick={() => visible = !visible}>切换</button>

{#if visible}
  <div transition:fade={{ duration: 300, delay: 100 }}>
    淡入淡出内容
  </div>
{/if}

<!-- 飞入效果 -->
{#if visible}
  <div transition:fly={{ x: 200, y: 200, duration: 500, easing: elasticOut }}>
    飞入内容
  </div>
{/if}

<!-- 缩放效果 -->
{#if visible}
  <div transition:scale={{ start: 0, duration: 300, easing: bounceOut }}>
    缩放内容
  </div>
{/if}

<!-- SVG 绘制动画 -->
<svg>
  <path 
    transition:draw={{ duration: 1000, easing: quintOut }}
    d="M10 10 L90 90"
  />
</svg>

<!-- 列表动画 -->
<div class="list">
  {#each items as item, index (item)}
    <div 
      animate:flip={{ duration: 300 }}
      in:fly={{ y: 50, duration: 300 }}
      out:fade={{ duration: 200 }}
    >
      {item}
      <button onclick={() => removeItem(index)}>删除</button>
    </div>
  {/each}
</div>

<!-- 全屏过渡 -->
{#if visible}
  <div 
    transition:fade={{ duration: 300 }}
    local:={{ duration: 300 }}
  >
    全屏内容
  </div>
{/if}
```

### 自定义指令

Svelte 支持自定义动作（指令）：

```typescript
// actions/tooltip.ts
interface TooltipOptions {
  text: string
  position: 'top' | 'bottom' | 'left' | 'right'
  delay?: number
}

export function tooltip(
  node: HTMLElement, 
  options: TooltipOptions | string
) {
  let tooltipEl: HTMLElement
  let timeout: ReturnType<typeof setTimeout>
  
  const opts = typeof options === 'string' 
    ? { text: options, position: 'top' } 
    : options
  
  function show() {
    timeout = setTimeout(() => {
      tooltipEl = document.createElement('div')
      tooltipEl.className = 'tooltip'
      tooltipEl.textContent = opts.text
      tooltipEl.style.position = 'absolute'
      document.body.appendChild(tooltipEl)
      
      const rect = node.getBoundingClientRect()
      const tooltipRect = tooltipEl.getBoundingClientRect()
      
      switch (opts.position) {
        case 'top':
          tooltipEl.style.top = `${rect.top - tooltipRect.height - 8}px`
          tooltipEl.style.left = `${rect.left + rect.width / 2 - tooltipRect.width / 2}px`
          break
        case 'bottom':
          tooltipEl.style.top = `${rect.bottom + 8}px`
          tooltipEl.style.left = `${rect.left + rect.width / 2 - tooltipRect.width / 2}px`
          break
        // ...
      }
    }, opts.delay || 200)
  }
  
  function hide() {
    clearTimeout(timeout)
    if (tooltipEl) {
      tooltipEl.remove()
      tooltipEl = undefined as any
    }
  }
  
  node.addEventListener('mouseenter', show)
  node.addEventListener('mouseleave', hide)
  
  return {
    destroy() {
      node.removeEventListener('mouseenter', show)
      node.removeEventListener('mouseleave', hide)
      if (tooltipEl) tooltipEl.remove()
    },
    update(newOptions: TooltipOptions | string) {
      // 更新选项
    }
  }
}

// actions/inview.ts
interface InViewOptions {
  threshold?: number
  rootMargin?: string
  onEnter?: () => void
  onLeave?: () => void
}

export function inview(
  node: HTMLElement, 
  options: InViewOptions = {}
) {
  const { threshold = 0, rootMargin = '0px', onEnter, onLeave } = options
  
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && onEnter) {
          onEnter()
        } else if (!entry.isIntersecting && onLeave) {
          onLeave()
        }
      })
    },
    { threshold, rootMargin }
  )
  
  observer.observe(node)
  
  return {
    destroy() {
      observer.disconnect()
    },
    update(newOptions: InViewOptions) {
      // 更新选项
    }
  }
}
```

```svelte
<script lang="ts">
  import { tooltip } from './actions/tooltip'
  import { inview } from './actions/inview'
  
  let hasEntered = $state(false)
</script>

<button 
  use:tooltip={{ text: '保存操作', position: 'top' }}
>
  保存
</button>

<div 
  use:inview={{ 
    threshold: 0.5, 
    onEnter: () => hasEntered = true 
  }}
>
  {#if hasEntered}
    <img src="lazy-image.jpg" alt="懒加载图片" />
  {/if}
</div>
```

### 持久化与缓存

```typescript
// stores/persisted.ts
import { writable, type Writable } from 'svelte/store'
import { browser } from '$app/environment'

interface PersistedOptions<T> {
  key: string
  initialValue: T
  storage?: Storage
  syncTab?: boolean
}

export function persisted<T>(
  store: Writable<T>,
  options: PersistedOptions<T>
): Writable<T> {
  const { key, initialValue, storage = localStorage, syncTab = true } = options
  
  // 初始化
  if (browser) {
    const stored = storage.getItem(key)
    if (stored !== null) {
      try {
        store.set(JSON.parse(stored))
      } catch {
        store.set(initialValue)
      }
    }
    
    // 监听其他标签页
    if (syncTab) {
      window.addEventListener('storage', (event) => {
        if (event.key === key && event.newValue !== null) {
          try {
            store.set(JSON.parse(event.newValue))
          } catch {}
        }
      })
    }
  }
  
  // 订阅更新
  store.subscribe((value) => {
    if (browser) {
      storage.setItem(key, JSON.stringify(value))
    }
  })
  
  return store
}

// 使用
import { writable } from 'svelte/store'

const theme = persisted(writable<'light' | 'dark'>('light'), {
  key: 'app-theme',
  syncTab: true
})

const userPreferences = persisted(writable({
  language: 'zh-CN',
  fontSize: 16
}), {
  key: 'user-preferences'
})
```

### SSR/SSG 深度理解

SvelteKit SSR/SSG 实现：

```typescript
// src/routes/+page.server.ts
import type { PageServerLoad, Actions } from './$types'
import { error, fail, redirect } from '@sveltejs/kit'
import { db } from '$lib/server/database'

export const load: PageServerLoad = async ({ params, locals, depends }) => {
  // 依赖声明
  depends('app:user')
  
  // 获取用户会话
  const session = await locals.auth()
  
  if (!session) {
    throw redirect(303, '/login')
  }
  
  // 获取数据
  const user = await db.user.findUnique({
    where: { id: session.user.id },
    include: { posts: true }
  })
  
  if (!user) {
    throw error(404, '用户不存在')
  }
  
  return {
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
      posts: user.posts.map(p => ({
        id: p.id,
        title: p.title,
        published: p.published
      }))
    }
  }
}

export const actions: Actions = {
  updateProfile: async ({ request, locals }) => {
    const formData = await request.formData()
    const name = formData.get('name') as string
    
    if (!name || name.length < 2) {
      return fail(400, { error: '名称至少需要2个字符' })
    }
    
    await db.user.update({
      where: { id: locals.user!.id },
      data: { name }
    })
    
    return { success: true }
  }
}
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
  import { enhance } from '$app/forms'
  import { invalidateAll } from '$app/navigation'
  
  let { data } = $props()
  let { user } = $derived(data)
  
  let isSubmitting = $state(false)
  
  function handleSubmit() {
    return async ({ result, update }) => {
      isSubmitting = false
      
      if (result.type === 'success') {
        await invalidateAll()
        // 显示成功消息
      }
      
      await update()
    }
  }
</script>

<h1>{user.name} 的资料</h1>
<p>邮箱: {user.email}</p>

<form 
  method="POST" 
  action="?/updateProfile"
  use:enhance={() => {
    isSubmitting = true
    return handleSubmit()
  }}
>
  <input 
    type="text" 
    name="name" 
    value={user.name}
    required
  />
  <button type="submit" disabled={isSubmitting}>
    {isSubmitting ? '保存中...' : '保存'}
  </button>
</form>

<h2>文章列表</h2>
<ul>
  {#each user.posts as post}
    <li>
      <a href="/posts/{post.id}">{post.title}</a>
      {#if post.published}
        <span class="badge">已发布</span>
      {/if}
    </li>
  {/each}
</ul>
```

### 性能优化

```typescript
// 1. 预加载优化
import { preloadCode, preloadData } from '$app/navigation'

// 鼠标悬停时预加载
function preloadOnHover(url: string) {
  return () => {
    const link = document.createElement('link')
    link.rel = 'prefetch'
    link.href = url
    document.head.appendChild(link)
  }
}

// 2. 图片优化
interface OptimizedImageProps {
  src: string
  alt: string
  width?: number
  height?: number
  loading?: 'lazy' | 'eager'
  sizes?: string
}

export function optimizedImage(props: OptimizedImageProps) {
  let srcset = ''
  
  // 生成响应式图片源
  const widths = [320, 640, 960, 1280, 1920]
  widths.forEach(w => {
    srcset += `${props.src}?w=${w} ${w}w, `
  })
  
  return `
    <img 
      src="${props.src}"
      srcset="${srcset}"
      alt="${props.alt}"
      width="${props.width || ''}"
      height="${props.height || ''}"
      loading="${props.loading || 'lazy'}"
      decoding="async"
    />
  `
}

// 3. 虚拟列表
import { viewport } from '$app/stores'

interface VirtualListOptions {
  items: any[]
  itemHeight: number
  containerHeight: number
}

export function virtualList(options: VirtualListOptions) {
  const { items, itemHeight, containerHeight } = options
  
  return $derived.by(() => {
    const totalHeight = items.length * itemHeight
    const visibleCount = Math.ceil(containerHeight / itemHeight) + 2
    
    return {
      totalHeight,
      visibleCount,
      items: items.slice(0, visibleCount)
    }
  })
}

// 4. 防抖和节流 stores
import { writable } from 'svelte/store'
import { browser } from '$app/environment'

export function debounced<T>(value: T, delay: number): Writable<T> {
  const store = writable(value)
  let timeout: ReturnType<typeof setTimeout>
  
  return {
    subscribe: store.subscribe,
    set: (newValue: T) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => store.set(newValue), delay)
    },
    update: (fn: (value: T) => T) => {
      clearTimeout(timeout)
      timeout = setTimeout(() => store.update(fn), delay)
    }
  }
}

export function throttled<T>(value: T, limit: number): Writable<T> {
  const store = writable(value)
  let lastRun = 0
  let pending: T | null = null
  let timeout: ReturnType<typeof setTimeout> | null = null
  
  return {
    subscribe: store.subscribe,
    set: (newValue: T) => {
      const now = Date.now()
      if (now >= lastRun + limit) {
        lastRun = now
        store.set(newValue)
      } else {
        pending = newValue
        if (!timeout) {
          timeout = setTimeout(() => {
            if (pending !== null) {
              lastRun = Date.now()
              store.set(pending)
              pending = null
            }
            timeout = null
          }, limit - (now - lastRun))
        }
      }
    }
  }
}
```

### 测试进阶

Vitest + Testing Library：

```typescript
// tests/unit/components/UserCard.spec.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { render, screen, fireEvent } from '@testing-library/svelte'
import userEvent from '@testing-library/user-event'
import UserCard from '$lib/components/UserCard.svelte'
import { writable } from 'svelte/store'

// Mock
vi.mock('$lib/services/api', () => ({
  updateUser: vi.fn().mockResolvedValue({ success: true }),
  deleteUser: vi.fn().mockResolvedValue(undefined)
}))

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const
  }

  it('renders user information', () => {
    render(UserCard, { props: { user: mockUser } })
    
    expect(screen.getByText('张三')).toBeInTheDocument()
    expect(screen.getByText('zhangsan@example.com')).toBeInTheDocument()
  })

  it('emits edit event on button click', async () => {
    const user = userEvent.setup()
    const { emitted } = render(UserCard, { props: { user: mockUser } })
    
    const editButton = screen.getByRole('button', { name: /编辑/i })
    await user.click(editButton)
    
    const events = emitted('edit')
    expect(events).toBeTruthy()
    expect(events[0]).toEqual(['1'])
  })

  it('shows confirmation on delete', async () => {
    vi.spyOn(window, 'confirm').mockReturnValue(true)
    
    render(UserCard, { props: { user: mockUser } })
    
    const deleteButton = screen.getByRole('button', { name: /删除/i })
    await userEvent.click(deleteButton)
    
    expect(window.confirm).toHaveBeenCalledWith('确定要删除吗？')
  })

  it('handles missing avatar gracefully', () => {
    const userWithoutAvatar = { ...mockUser, avatar: undefined }
    const { container } = render(UserCard, { props: { user: userWithoutAvatar } })
    
    // 显示默认头像
    const avatar = container.querySelector('.avatar')
    expect(avatar?.textContent).toBe('张')
  })
})
```

```typescript
// tests/integration/auth.spec.ts
import { describe, it, expect } from 'vitest'
import { renderHook, act } from '@testing-library/svelte'
import { useAuth } from '$lib/composables/useAuth'

describe('useAuth', () => {
  it('initializes with default state', () => {
    const { result } = renderHook(() => useAuth())
    
    expect(result.current.user).toBeNull()
    expect(result.current.isAuthenticated).toBe(false)
    expect(result.current.isLoading).toBe(false)
  })

  it('logs in user successfully', async () => {
    const { result } = renderHook(() => useAuth())
    
    await act(async () => {
      await result.current.login('test@example.com', 'password')
    })
    
    expect(result.current.isAuthenticated).toBe(true)
    expect(result.current.user?.email).toBe('test@example.com')
  })
})
```

### 国际化（i18n）

```typescript
// src/lib/i18n/index.ts
import { derived, writable } from 'svelte/store'

type TranslationKey = string
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

export const locale = writable<Locale>('zh')

export const t = derived(locale, ($locale) => {
  return (key: TranslationKey, params?: Record<string, string | number>) => {
    let text = translations[$locale][key] || key
    
    if (params) {
      Object.entries(params).forEach(([k, v]) => {
        text = text.replace(`{${k}}`, String(v))
      })
    }
    
    return text
  }
})

export function setLocale(newLocale: Locale) {
  locale.set(newLocale)
  if (typeof localStorage !== 'undefined') {
    localStorage.setItem('locale', newLocale)
  }
}
```

```svelte
<!-- 使用国际化 -->
<script lang="ts">
  import { t, locale, setLocale } from '$lib/i18n'
</script>

<h1>{$t('app.title')}</h1>

<nav>
  <a href="/">{$t('nav.home')}</a>
  <a href="/about">{$t('nav.about')}</a>
</nav>

<p>{$t('user.greeting', { name: '张三' })}</p>

<select bind:value={$locale}>
  <option value="en">English</option>
  <option value="zh">中文</option>
  <option value="ja">日本語</option>
</select>
```

---

## Svelte 生态工具链详解

### 构建工具配置

Vite + SvelteKit 配置：

```typescript
// vite.config.ts
import { sveltekit } from '@sveltejs/kit/vite'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [sveltekit()],
  
  server: {
    port: 5173,
    strictPort: false,
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true
      }
    }
  },
  
  build: {
    target: 'esnext',
    minify: 'esbuild'
  },
  
  optimizeDeps: {
    include: ['svelte', 'svelte/transition', 'svelte/animate']
  }
})
```

### 样式方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Scoped CSS** | 原生支持，无需配置 | 全局样式需特殊处理 | 所有项目 |
| **Tailwind CSS** | 极快开发，原子化 | 类名冗长 | 快速原型 |
| **UnoCSS** | 极快，按需生成 | 社区相对较小 | 性能敏感项目 |
| **CSS Variables** | 主题切换简单 | 需要手动管理 | 主题需求 |

---

## 企业级架构模式

### 微前端架构 (Micro Frontends)

微前端将微服务的理念引入前端，实现大型应用的独立开发和部署：

```typescript
// 1. SvelteKit 中加载远程模块
// src/routes/+layout.svelte
<script lang="ts">
  import { onMount } from 'svelte';
  
  let RemoteApp: any;
  let loading = $state(true);
  let error = $state<string | null>(null);
  
  onMount(async () => {
    try {
      // 使用动态导入加载远程模块
      const module = await import('http://remote.example.com/remote.js');
      RemoteApp = module.default;
      loading = false;
    } catch (e) {
      error = 'Failed to load remote module';
      loading = false;
    }
  });
</script>

<div class="micro-frontend-container">
  {#if loading}
    <div class="loading">Loading...</div>
  {:else if error}
    <div class="error">{error}</div>
  {:else if RemoteApp}
    <svelte:component this={RemoteApp} />
  {/if}
</div>
```

```typescript
// 2. SvelteKit + Module Federation
// vite.config.ts
import { sveltekit } from '@sveltejs/kit/vite';
import { defineConfig } from 'vite';
import ModuleFederation from 'vite-plugin-federation';

export default defineConfig({
  plugins: [
    sveltekit(),
    ModuleFederation({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@https://remote.example.com/remoteEntry.js',
      },
      shared: ['svelte'],
    }),
  ],
  optimizeDeps: {
    include: ['svelte', 'svelte/store'],
  },
});
```

**微前端实现方案对比：**

| 方案 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|---------|------|------|----------|
| **Module Federation** | Webpack 共享 | 运行时集成，按需加载 | 依赖统一版本 | 大型团队协作 |
| **iframe** | HTML iframe | 隔离彻底，技术栈无关 | 通信困难 | 独立子应用 |
| **Single-SPA** | 生命周期管理 | 框架无关，成熟方案 | 需要适配层 | 多框架共存 |
| **SvelteKit + 动态导入** | 原生 ES 模块 | 简单直接 | 共享状态困难 | 简单场景 |

### Svelte 5 组件架构模式

```svelte
<!-- 1. Container/Presentational 模式 -->
<!-- src/lib/components/UserList.svelte (Presentational) -->
<script lang="ts">
  interface Props {
    users: User[];
    loading: boolean;
    error: string | null;
    onSelect: (id: string) => void;
    onDelete: (id: string) => void;
  }
  
  let { users, loading, error, onSelect, onDelete }: Props = $props();
</script>

{#if loading}
  <div class="loading">Loading...</div>
{:else if error}
  <div class="error">{error}</div>
{:else}
  <div class="user-list">
    {#each users as user (user.id)}
      <div class="user-card">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
        <button onclick={() => onSelect(user.id)}>查看详情</button>
        <button onclick={() => onDelete(user.id)}>删除</button>
      </div>
    {/each}
  </div>
{/if}
```

### Svelte Store 进阶模式

```typescript
// stores/asyncStore.ts - 异步 Store
function createAsyncStore<T>(fetcher: () => Promise<T>) {
  let data = $state<T | null>(null);
  let loading = $state(false);
  let error = $state<Error | null>(null);
  
  async function fetch() {
    loading = true;
    error = null;
    try {
      data = await fetcher();
    } catch (e) {
      error = e as Error;
    } finally {
      loading = false;
    }
  }
  
  return {
    get data() { return data; },
    get loading() { return loading; },
    get error() { return error; },
    fetch,
    refetch: fetch,
  };
}
```

---

## CI/CD 部署与自动化

### GitHub Actions 完整配置

```yaml
# .github/workflows/svelte-ci-cd.yml
name: Svelte 5 CI/CD

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
          publish-dir: ./build
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Vercel 部署配置

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "build",
  "framework": "sveltekit",
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "X-Frame-Options", "value": "DENY" }
      ]
    }
  ]
}
```

---

## 安全最佳实践

### XSS 防护

```typescript
// src/lib/utils/sanitize.ts
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
// src/lib/utils/csrf.ts
let csrfToken = $state<string | null>(null);

export async function fetchCsrfToken(): Promise<string> {
  if (csrfToken) return csrfToken;
  
  const response = await fetch('/api/csrf-token', {
    credentials: 'include',
  });
  const data = await response.json();
  csrfToken = data.token;
  return data.token;
}
```

### 敏感信息管理

```bash
# .env.development
PUBLIC_API_URL=http://localhost:3000
PUBLIC_ENABLE_DEBUG=true

# .env.production
PUBLIC_API_URL=https://api.production.com
PUBLIC_SENTRY_DSN=https://xxx@sentry.io/xxx
```

---

## 可访问性 (A11y) 最佳实践

### ARIA 属性完整指南

```svelte
<!-- 1. 按钮 vs 链接 -->
<div>
  <button onclick={handleSave} aria-describedby="save-description">
    保存
  </button>
  <p id="save-description">保存当前编辑的内容</p>
  
  <a href="/dashboard" aria-current="page">
    前往仪表盘
  </a>
</div>

<!-- 2. 表单错误提示 -->
<form onsubmit={handleSubmit}>
  <div>
    <label for="email">邮箱地址</label>
    <input
      id="email"
      type="email"
      bind:value={email}
      aria-invalid={errors.email ? 'true' : undefined}
      aria-describedby={errors.email ? 'email-error' : undefined}
    />
    {#if errors.email}
      <p id="email-error" role="alert">{errors.email}</p>
    {/if}
  </div>
</form>

<!-- 3. 模态对话框 -->
{#if isOpen}
  <div
    role="dialog"
    aria-modal="true"
    aria-labelledby="modal-title"
    tabindex="-1"
  >
    <h2 id="modal-title">{title}</h2>
    <button onclick={close} aria-label="关闭">×</button>
    <slot />
  </div>
{/if}

<!-- 4. 实时区域 -->
<div aria-live="polite" aria-atomic="true" class="sr-only">
  {notification}
</div>

<!-- 5. 复杂表格 -->
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
```

### 键盘导航支持

```svelte
<!-- Roving Tabindex -->
<nav role="menubar" aria-label="主菜单">
  {#each menuItems as item, index (item)}
    <button
      role="menuitem"
      tabindex={index === activeIndex ? 0 : -1}
      onclick={() => handleSelect(item)}
      onkeydown={handleKeydown}
    >
      {item.label}
    </button>
  {/each}
</nav>

<script lang="ts">
  let menuItems = [
    { label: '首页' },
    { label: '关于' },
    { label: '产品' },
    { label: '联系' },
  ];
  let activeIndex = $state(0);
  
  function handleKeydown(e: KeyboardEvent) {
    switch (e.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        e.preventDefault();
        activeIndex = (activeIndex + 1) % menuItems.length;
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        e.preventDefault();
        activeIndex = (activeIndex - 1 + menuItems.length) % menuItems.length;
        break;
      case 'Home':
        e.preventDefault();
        activeIndex = 0;
        break;
      case 'End':
        e.preventDefault();
        activeIndex = menuItems.length - 1;
        break;
    }
  }
  
  function handleSelect(item: typeof menuItems[0]) {
    console.log('Selected:', item);
  }
</script>
```

### 颜色对比度和视觉辅助

```css
/* WCAG AA 标准对比度检查 */
/* 正常文本: 4.5:1 */
/* 大文本 (18px+): 3:1 */

:root {
  --color-primary: #ff3e00;
  --text-primary: #333333;
  --text-secondary: #666666;
}

:focus-visible {
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
PUBLIC_API_URL=http://localhost:3000
PUBLIC_ENABLE_DEBUG=true

# .env.production
PUBLIC_API_URL=https://api.production.com
PUBLIC_SENTRY_DSN=https://xxx@sentry.io/xxx
```

```typescript
// src/lib/config/index.ts
import { PUBLIC_API_URL } from '$env/static/public';

interface AppConfig {
  apiUrl: string;
  enableDebug: boolean;
}

export function getConfig(): AppConfig {
  return {
    apiUrl: PUBLIC_API_URL,
    enableDebug: PUBLIC_API_URL.includes('localhost'),
  };
}
```

---

> [!SUCCESS]
> 本文档涵盖了 Svelte 5 的核心理念、Runes 语法、组件系统、Store、路由、表单处理、高级模式、过渡动画、自定义指令、持久化、SSR进阶、国际化、测试等全方位内容。Svelte 5 以编译时优化的极致性能和简洁的语法，为现代 Web 开发提供了全新的选择。其在 AI 应用中的性能优势使其特别适合高频更新场景。
