# React - 组件化 UI 开发标准

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 React 19 新特性、Hooks 体系、状态管理与 vibecoding 实践。

---

## 目录

1. [[#React 概述与版本演进]]
2. [[#React 19 新特性]]
3. [[#Virtual DOM 原理]]
4. [[#Hooks 完整指南]]
5. [[#React Server Components]]
6. [[#状态管理方案]]
7. [[#React Compiler]]
8. [[#与 Vue/Angular 对比]]
9. [[#实战技巧]]
10. [[#参考资料]]

---

## React 概述与版本演进

### 什么是 React

React 是 Facebook（现 Meta）于 2013 年开源的 JavaScript 库，专门用于构建用户界面。其核心思想是**声明式编程**和**组件化开发**，通过 Virtual DOM 技术实现高效的页面更新。

React 不是完整的 MVC 框架，而是一个专注于视图层（View）的库，这使得它可以灵活地与各种状态管理和路由方案组合使用。

### 版本演进表

| 版本 | 发布年份 | 重大特性 |
|------|---------|---------|
| React 0.3 | 2013 | 首次开源发布 |
| React 0.14 | 2015 | 组件函数化，props children |
| React 16.0 | 2017 | Fiber 架构，Error Boundaries |
| React 16.8 | 2019 | **Hooks 引入**，函数组件能力完整 |
| React 17 | 2020 | 渐进式升级，新 JSX Transform |
| React 18 | 2022 | Concurrent Features，Suspense |
| React 19 | 2024 | **Server Components，Actions，Compiler** |

### 核心特点

| 特点 | 说明 |
|------|------|
| **声明式** | 描述 UI 状态，React 自动处理更新 |
| **组件化** | 可复用、可组合的 UI 构建块 |
| **单向数据流** | 数据自顶向下传递 |
| **Virtual DOM** | 高效的 DOM 差异对比 |
| **生态丰富** | 庞大的社区和工具链 |

---

## React 19 新特性

### 1. React Server Components (RSC)

Server Components 允许组件在服务器端渲染，减少客户端 JavaScript 体积：

```tsx
// app/users/page.tsx (Server Component - 默认)
import { db } from '@/lib/db';

// 这是服务器组件，可以直接访问数据库
async function UserList() {
  const users = await db.query('SELECT * FROM users');
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

export default UserList;
```

| 组件类型 | 执行位置 | 特点 |
|---------|---------|------|
| **Server Components** | 服务器 | 无状态，可访问后端资源 |
| **Client Components** | 客户端 | 使用 useState/useEffect，需要 `'use client'` |

### 2. Actions 与表单处理

React 19 简化了表单处理和服务器交互：

```tsx
// app/actions.ts
'use server';

export async function submitForm(formData: FormData) {
  const name = formData.get('name');
  const email = formData.get('email');
  
  // 直接操作数据库或 API
  await db.users.create({ name, email });
  
  // 返回结果，自动更新 UI
  return { success: true };
}
```

```tsx
// components/SignupForm.tsx
'use client';

import { submitForm } from '@/app/actions';

function SignupForm() {
  return (
    <form action={submitForm}>
      <input name="name" type="text" />
      <input name="email" type="email" />
      <button type="submit">注册</button>
    </form>
  );
}
```

### 3. use() Hook

新的 `use()` Hook 允许在组件中读取 Promise 和 Context：

```tsx
import { use, Suspense } from 'react';

// 读取 Promise（配合 Server Components）
function UserProfile({ userPromise }) {
  // 不需要 useEffect + loading 状态
  const user = use(userPromise);
  
  return <h1>{user.name}</h1>;
}

// 使用
<UserProfile userPromise={fetchUser(id)} />

// 读取 Context
function ThemedButton({ theme }) {
  const ctx = use(ThemeContext); // 可以在条件语句中使用
  return <button className={ctx.theme}>{theme}</button>;
}
```

### 4. 资源预加载 API

```tsx
import { preload, preloadDNS } from 'react-dom';

// 预加载资源
preload('/fonts/custom.woff2', { as: 'font', type: 'font/woff2' });

// 预解析 DNS
preloadDNS('https://api.example.com');
```

### 5. 改进的 Ref 作为 Prop

```tsx
// React 19: ref 可以像普通 prop 一样传递
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// 父组件可以直接传递 ref
function Form() {
  const inputRef = useRef();
  return <Input ref={inputRef} />;
}
```

---

## Virtual DOM 原理

### 什么是 Virtual DOM

Virtual DOM（虚拟 DOM）是真实 DOM 的 JavaScript 对象表示。通过对比新旧虚拟 DOM 的差异（Diffing），React 最小化实际 DOM 操作，从而提升性能。

### 工作原理

```
┌─────────────────────────────────────────────────────────┐
│                    用户交互                              │
│              (点击、输入、网络响应)                        │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                 State/Props 更新                        │
│                   状态或属性变化                         │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                 Render 阶段                              │
│          创建新的 Virtual DOM 树                         │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                 Reconciliation                          │
│                 差异对比（Diffing）                       │
│     - 同级对比（tree diff）                             │
│     - 组件类型对比（component diff）                     │
│     - 元素标签对比（element diff）                      │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                 Commit 阶段                              │
│              批量更新真实 DOM                            │
│            (最小化 DOM 操作次数)                         │
└─────────────────────────────────────────────────────────┘
```

### Diffing 算法

| 策略 | 说明 | 复杂度 |
|------|------|-------|
| **同级对比** | 只比较同层节点，不跨层移动 | O(n) |
| **Key 优化** | 使用 key 标识节点，实现高效移动 | O(n) |
| **组件对比** | 类型不同则完全替换 | O(n) |
| **列表对比** | key 帮助识别移动的节点 | O(n) |

### Fiber 架构（React 16+）

Fiber 是 React 16 引入的新协调引擎，支持：

| 特性 | 说明 |
|------|------|
| **可中断渲染** | 高优先级更新可打断低优先级 |
| **时间切片** | 将渲染工作分散到多帧 |
| **Suspense** | 支持异步组件的 loading 状态 |
| **Concurrent Mode** | 并发渲染模式 |

---

## Hooks 完整指南

### Hooks 概述

Hooks 是 React 16.8 引入的特性，允许在函数组件中使用 state 和其他 React 特性。

### Hooks 对比表

| Hook | 用途 | 复杂度 | 使用频率 |
|------|------|-------|---------|
| **useState** | 管理组件状态 | ⭐ | ⭐⭐⭐⭐⭐ |
| **useEffect** | 处理副作用 | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| **useCallback** | 缓存函数 | ⭐ | ⭐⭐⭐ |
| **useMemo** | 缓存计算结果 | ⭐ | ⭐⭐⭐ |
| **useRef** | 访问 DOM/持久化 | ⭐ | ⭐⭐⭐⭐ |
| **useContext** | 读取 Context | ⭐ | ⭐⭐⭐ |
| **useReducer** | 复杂状态逻辑 | ⭐⭐ | ⭐⭐ |
| **useLayoutEffect** | 同步副作用 | ⭐⭐ | ⭐ |
| **useImperativeHandle** | 暴露 ref | ⭐⭐ | ⭐ |
| **useDebugValue** | DevTools 调试 | ⭐ | ⭐ |

### useState - 状态管理

```tsx
import { useState } from 'react';

// 基础用法
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
    </div>
  );
}

// 函数式更新
function CounterWithReset() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>
        增加
      </button>
      <button onClick={() => setCount(0)}>
        重置
      </button>
    </div>
  );
}

// 对象状态
function LoginForm() {
  const [form, setForm] = useState({
    username: '',
    password: '',
    remember: false
  });
  
  const updateField = (field, value) => {
    setForm(prev => ({ ...prev, [field]: value }));
  };
  
  return (
    <form>
      <input
        value={form.username}
        onChange={e => updateField('username', e.target.value)}
      />
      <input
        type="password"
        value={form.password}
        onChange={e => updateField('password', e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={form.remember}
          onChange={e => updateField('remember', e.target.checked)}
        />
        记住我
      </label>
    </form>
  );
}
```

### useEffect - 副作用处理

```tsx
import { useState, useEffect } from 'react';

// 基础用法 - 组件挂载时执行
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    // 组件挂载后执行
    fetchUser(userId).then(setUser);
  }, [userId]); // userId 变化时重新执行
  
  return <div>{user?.name}</div>;
}

// 清理副作用
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    
    // 返回清理函数
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []); // 空依赖数组，仅执行一次
  
  return (
    <div>
      鼠标位置: {position.x}, {position.y}
    </div>
  );
}

// 条件执行 effect
function DataFetcher({ url, enabled }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    if (!enabled) return;
    
    fetch(url).then(res => res.json()).then(setData);
  }, [url, enabled]); // enabled 变化也会触发
  
  return <div>{data ? JSON.stringify(data) : 'Loading...'}</div>;
}
```

### useCallback - 函数缓存

```tsx
import { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // 每次渲染都创建新函数（地址变化）
  const handleClickBad = () => {
    console.log('clicked');
  };
  
  // 缓存函数，依赖不变时不创建新函数
  const handleClickGood = useCallback(() => {
    console.log('clicked');
  }, []); // 空依赖，函数永不变化
  
  // 依赖 count 的回调
  const incrementWithLog = useCallback(() => {
    console.log('Before increment:', count);
    setCount(c => c + 1);
  }, [count]); // count 变化时创建新函数
  
  return (
    <div>
      <ChildComponent onClick={handleClickGood} />
      <Counter onIncrement={incrementWithLog} />
    </div>
  );
}

function ChildComponent({ onClick }) {
  // 只有 onClick 地址变化时才会重新渲染
  return <button onClick={onClick}>点击</button>;
}
```

### useMemo - 计算结果缓存

```tsx
import { useMemo } from 'react';

function ExpensiveComponent({ data, filter }) {
  // 复杂计算，仅在 data 或 filter 变化时重新计算
  const filteredData = useMemo(() => {
    console.log('Computing filtered data...');
    return data.filter(item => 
      item.name.includes(filter)
    );
  }, [data, filter]); // 依赖数组
  
  // 缓存对象引用
  const config = useMemo(() => ({
    pageSize: 20,
    sortBy: 'date',
    theme: 'dark'
  }), []); // 永远不变化的对象
  
  return (
    <div>
      {filteredData.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

// useMemo vs useCallback
function Example() {
  // useMemo 缓存计算结果
  const computed = useMemo(() => expensiveCalculation(a, b), [a, b]);
  
  // useCallback 缓存函数
  const handler = useCallback(() => doSomething(a, b), [a, b]);
  
  // useMemo 也可用于缓存函数
  const memoizedFunction = useMemo(() => () => doSomething(a), [a]);
}
```

### useRef - 持久化与 DOM 访问

```tsx
import { useRef, useEffect } from 'react';

function FocusInput() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // 访问 DOM 元素
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} />;
}

function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef(null);
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
    
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return <div>{count}</div>;
}

// 存储上一次的值
function PreviousValue() {
  const [value, setValue] = useState('');
  const prevValueRef = useRef('');
  
  useEffect(() => {
    prevValueRef.current = value;
  }, [value]);
  
  return (
    <div>
      <input value={value} onChange={e => setValue(e.target.value)} />
      <p>当前: {value}</p>
      <p>上一次: {prevValueRef.current}</p>
    </div>
  );
}
```

### useContext - 跨组件通信

```tsx
import { createContext, useContext, useState } from 'react';

// 创建 Context
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
});

// Provider 组件
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(t => t === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 消费 Context
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button 
      className={`btn-${theme}`}
      onClick={toggleTheme}
    >
      切换主题
    </button>
  );
}

// 使用
<ThemeProvider>
  <App />
</ThemeProvider>
```

### useReducer - 复杂状态逻辑

```tsx
import { useReducer } from 'react';

// State 类型
interface State {
  count: number;
  step: number;
}

// Action 类型
type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number }
  | { type: 'reset' };

// Reducer 函数
function counterReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return { count: 0, step: 1 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    step: 1
  });
  
  return (
    <div>
      <p>计数: {state.count}, 步长: {state.step}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>
        增加
      </button>
      <button onClick={() => dispatch({ type: 'decrement' })}>
        减少
      </button>
      <button onClick={() => dispatch({ type: 'setStep', payload: 5 })}>
        步长设为 5
      </button>
      <button onClick={() => dispatch({ type: 'reset' })}>
        重置
      </button>
    </div>
  );
}
```

---

## React Server Components

### 核心概念

| 概念 | 说明 |
|------|------|
| **Server Component** | 在服务器渲染，不包含客户端交互代码 |
| **Client Component** | `'use client'` 声明，需要在浏览器运行 |
| **混用规则** | Server Component 可导入 Client Component |

### 使用示例

```tsx
// app/layout.tsx (Server Component)
export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        <Navbar />
        {children}
      </body>
    </html>
  );
}

// components/Navbar.tsx (Server Component)
export default function Navbar() {
  // 可以直接访问数据库
  const navItems = await db.navItems.findAll();
  
  return (
    <nav>
      {navItems.map(item => (
        <a key={item.id} href={item.href}>
          {item.label}
        </a>
      ))}
    </nav>
  );
}

// components/ThemeToggle.tsx (Client Component)
'use client';

import { useState } from 'react';

export function ThemeToggle() {
  const [theme, setTheme] = useState('light');
  
  return (
    <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
      {theme === 'light' ? '🌙' : '☀️'}
    </button>
  );
}

// 在 Server Component 中使用 Client Component
// app/page.tsx
import { ThemeToggle } from '@/components/ThemeToggle'; // Client Component

export default function Page() {
  return (
    <div>
      <h1>Welcome</h1>
      <ThemeToggle /> {/* 客户端组件 */}
    </div>
  );
}
```

---

## 状态管理方案

### 状态管理对比表

| 方案 | 类型 | 适用场景 | 学习曲线 | 性能 |
|------|------|---------|---------|------|
| **useState** | 本地状态 | 组件内部状态 | ⭐ | 最佳 |
| **useContext** | 跨组件 | 少量共享状态 | ⭐⭐ | 一般 |
| **Zustand** | 全局状态 | 中大型应用 | ⭐⭐ | 优秀 |
| **Jotai** | 原子化状态 | 细粒度状态 | ⭐⭐ | 优秀 |
| **Redux Toolkit** | 全局状态 | 大型企业应用 | ⭐⭐⭐ | 良好 |
| **Recoil** | 原子化状态 | React 专用 | ⭐⭐ | 良好 |

### Zustand 完整指南

```tsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// 定义 Store
interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

const useCounterStore = create<CounterStore>()(
  devtools(
    persist(
      (set) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 })),
        decrement: () => set((state) => ({ count: state.count - 1 })),
        reset: () => set({ count: 0 }),
      }),
      { name: 'counter-storage' } // localStorage key
    )
  )
);

// 使用
function Counter() {
  const { count, increment, decrement, reset } = useCounterStore();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>重置</button>
    </div>
  );
}

// 在组件外使用
const snapshot = useCounterStore.getState();
console.log(snapshot.count);
```

### Jotai 完整指南

```tsx
import { atom, useAtom } from 'jotai';

// 定义原子
const countAtom = atom(0);
const doubledAtom = atom((get) => get(countAtom) * 2);

// 派生原子
const asyncAtom = atom(async (get) => {
  const count = get(countAtom);
  return await fetch(`/api/count/${count}`);
});

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubled] = useAtom(doubledAtom);
  
  return (
    <div>
      <p>计数: {count}</p>
      <p>翻倍: {doubled}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

### Redux Toolkit

```tsx
import { createSlice, configureStore } from '@reduxjs/toolkit';
import { useSelector, useDispatch } from 'react-redux';

// 定义 Slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
    decrement: (state) => { state.value -= 1; },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

// 创建 Store
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
  },
});

// 使用
function Counter() {
  const count = useSelector(state => state.counter.value);
  const dispatch = useDispatch();
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={() => dispatch(counterSlice.actions.increment())}>
        +
      </button>
    </div>
  );
}
```

---

## React Compiler

### 概述

React Compiler（原名 React Forget）是 React 19 引入的实验性编译器，自动优化组件以减少不必要的重渲染。

### 工作原理

```tsx
// 编译前
function Component({ items, onClick }) {
  const [selected, setSelected] = useState(null);
  
  const filtered = items.filter(i => i.active); // 每次渲染都重新计算
  
  const handleClick = () => { // 每次渲染都创建新函数
    onClick(selected);
  };
  
  return (
    <div>
      {filtered.map(item => (
        <Item 
          key={item.id} 
          item={item}
          onClick={handleClick}
        />
      ))}
    </div>
  );
}

// 编译后（自动优化）
function Component({ items, onClick }) {
  const [selected, setSelected] = useState(null);
  
  // 编译器自动添加 useMemo
  const filtered = useMemo(() => 
    items.filter(i => i.active), 
    [items]
  );
  
  // 编译器自动添加 useCallback
  const handleClick = useCallback(() => {
    onClick(selected);
  }, [onClick, selected]);
  
  return (
    <div>
      {filtered.map(item => (
        <Item 
          key={item.id} 
          item={item}
          onClick={handleClick}
        />
      ))}
    </div>
  );
}
```

### 配置

```bash
npm install babel-plugin-react-compiler
```

```javascript
// babel.config.js
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      target: '19'
    }]
  ]
};
```

---

## 与 Vue/Angular 对比

### 核心对比表

| 维度 | React | Vue | Angular |
|------|--------|-----|---------|
| **类型** | UI 库 | 渐进式框架 | 完整框架 |
| **模板** | JSX | SFC | TypeScript + HTML |
| **状态管理** | 多种选择 | Vuex/Pinia | NgRx/Service |
| **学习曲线** | 中等 | 较平缓 | 较陡 |
| **社区规模** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **灵活性** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ |
| **性能** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **TypeScript** | 良好支持 | 良好支持 | 原生支持 |

### 场景推荐

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| 快速原型 | Vue | 上手最快 |
| 大型企业应用 | Angular | 完整解决方案 |
| 灵活的 UI 库 | React | 最灵活 |
| 学习曲线敏感 | Vue | 文档最友好 |
| 团队有 Angular 背景 | Angular | 技术栈一致 |
| 需要 SSR/SSG | Next.js+React | 生态最完善 |

---

## 实战技巧

### 1. AI 辅助代码生成

```typescript
// 使用 Cursor 生成 React 组件的 prompt 示例
const generateComponentPrompt = `
创建一个 React 用户卡片组件 UserCard：
- 组件属性：
  - user: { id, name, email, avatar, role }
  - onEdit: (id: string) => void
  - onDelete: (id: string) => void
- 功能要求：
  - 显示用户头像（无头像显示默认图标）
  - 显示用户角色标签（admin/user/guest 不同颜色）
  - 悬停时显示编辑和删除按钮
  - 点击编辑按钮调用 onEdit
  - 点击删除按钮调用 onDelete（带确认）
- 样式：使用 Tailwind CSS
- 包含完整的 TypeScript 类型
- 包含 JSDoc 注释
`;
```

### 2. 性能优化清单

| 检查项 | 工具/方法 |
|-------|---------|
| 重渲染分析 | React DevTools Profiler |
| 打包大小 | webpack-bundle-analyzer |
| 懒加载 | React.lazy + Suspense |
| 列表 key | 使用稳定唯一 ID |
| 状态位置 | 提升到最小必要层级 |
| 依赖数组 | 确保完整且正确 |

### 3. TypeScript 最佳实践

```tsx
// Props 类型定义
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

// FC 方式（传统）
const Button: React.FC<ButtonProps> = ({ 
  variant = 'primary',
  children 
}) => {
  return <button className={`btn-${variant}`}>{children}</button>;
};

// 函数式（推荐）
function Button({ 
  variant = 'primary',
  children 
}: ButtonProps) {
  return <button className={`btn-${variant}`}>{children}</button>;
}

// 泛型组件
function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://react.dev |
| GitHub | https://github.com/facebook/react |
| React Blog | https://react.dev/blog |
| React Conf | https://conf.react.dev |

### 学习资源

| 资源 | 说明 |
|------|------|
| React Training | 官方培训课程 |
| Kent C. Dodds Blog | React 深度文章 |
| Overreacted | Dan Abramov 的博客 |

### 工具链

| 工具 | 用途 |
|------|------|
| Next.js | SSR/SSG 框架 |
| Remix | 全栈框架 |
| Gatsby | 静态站点生成 |
| Create React App | 项目脚手架 |
| Vite | 快速构建工具 |

---

> [!SUCCESS]
> React 作为全球最流行的 UI 库，在 2026 年通过 React 19 的 Server Components、Actions 和 Compiler 等新特性进一步巩固了其领先地位。对于 vibecoding 开发者而言，掌握 React 的组件化思维、Hooks 体系和状态管理方案是构建现代 Web 应用的基础技能。

---

## 核心理念与设计哲学

### React 的设计哲学

React 的设计哲学建立在几个核心原则之上，理解这些原则对于掌握 React 至关重要：

**1. 声明式编程范式**

React 推广声明式编程，开发者描述"应该是什么"而非"如何做"。这使得代码更可预测、更易于调试。

```tsx
// 声明式：描述 UI 应该呈现什么状态
function UserProfile({ user }) {
  if (!user) {
    return <div>Loading...</div>;
  }
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
}

// 命令式：描述具体的 DOM 操作步骤
function UserProfileImperative({ user }) {
  const container = document.createElement('div');
  if (!user) {
    container.innerHTML = '<div>Loading...</div>';
  } else {
    container.innerHTML = `
      <h1>${user.name}</h1>
      <p>${user.bio}</p>
    `;
  }
  return container;
}
```

**2. 组件化架构**

React 鼓励将 UI 拆分为独立的、可复用的组件。每个组件都有自己的逻辑、样式和状态。

```tsx
// 原子组件
function Button({ label, onClick, variant = 'primary' }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {label}
    </button>
  );
}

// 分子组件
function SearchBar({ onSearch }) {
  const [query, setQuery] = useState('');
  
  return (
    <div className="search-bar">
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="搜索..."
      />
      <Button 
        label="搜索" 
        onClick={() => onSearch(query)} 
      />
    </div>
  );
}

// 有机组件
function SearchSection({ onResults }) {
  return (
    <section className="search-section">
      <SearchBar onSearch={onResults} />
    </section>
  );
}
```

**3. 单向数据流**

React 采用单向数据流，数据从父组件流向子组件，父组件通过 props 控制子组件的行为。

```
┌─────────────────────────────────────────────────────────────┐
│                        顶层组件                              │
│                    (拥有共享状态)                            │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  State: { users, theme, isLoggedIn }                 │  │
│  │  Actions: fetchUsers(), toggleTheme(), login()        │  │
│  └─────────────────────────────────────────────────────┘  │
│                           │                                 │
│            ┌──────────────┼──────────────┐                 │
│            ▼              ▼              ▼                 │
│      ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│      │ Header    │  │ Sidebar  │  │ Main     │           │
│      │ (props)   │  │ (props)  │  │ (props)  │           │
│      └──────────┘  └──────────┘  └──────────┘           │
│                                                        │
│  子组件通过 props 接收数据，通过回调函数向上传递事件      │
└─────────────────────────────────────────────────────────────┘
```

**4. 组合优于继承**

React 推荐使用组合模式而非继承来复用代码。组件可以通过 children prop 或 render props 接收任意内容。

```tsx
// 组合模式：使用 children
function Card({ children, title }) {
  return (
    <div className="card">
      {title && <h2 className="card-title">{title}</h2>}
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// 使用
<Card title="用户信息">
  <UserDetails user={currentUser} />
  <Button label="编辑" onClick={handleEdit} />
</Card>

// 组合模式：使用 render props
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// 使用
<MouseTracker 
  render={({ x, y }) => (
    <div>鼠标位置: {x}, {y}</div>
  )} 
/>
```

### React 解决的问题域

React 专注于解决以下核心问题：

| 问题域 | React 解决方案 | 说明 |
|--------|--------------|------|
| **UI 更新复杂性** | Virtual DOM + 声明式 | 开发者描述目标状态，React 处理更新细节 |
| **组件复用性** | 组件系统 | 函数组件 + Hooks 提供灵活的复用机制 |
| **状态管理** | 多种方案 | useState → Context → Redux/Zustand 按需选择 |
| **大型应用维护** | 组件化 + 单向数据流 | 代码结构清晰，状态可追踪 |
| **团队协作** | 组件库 + 设计系统 | 标准化组件提升协作效率 |

---

## 完整安装与项目创建

### 环境准备

在开始创建 React 项目之前，需要确保开发环境满足以下要求：

**Node.js 版本要求：**
- React 19 需要 Node.js 18.17 或更高版本
- 推荐使用 Node.js 20 LTS 或更高版本
- 使用 nvm（Node Version Manager）管理多个 Node 版本

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 检查 npm 版本
npm --version
# 10.2.4

# 安装或切换 Node 版本（使用 nvm）
nvm install 20
nvm use 20
nvm alias default 20
```

**pnpm vs npm vs yarn 对比：**

| 包管理器 | 优点 | 缺点 | 适用场景 |
|---------|------|------|---------|
| npm | 内置，无需安装 | 速度较慢，依赖重复安装 | 通用场景 |
| pnpm | 速度快，磁盘占用小 | 生态兼容性偶有问题 | 大型项目（**推荐**） |
| yarn | 速度较快，稳定性好 | 依赖较重 | 团队项目 |

```bash
# 安装 pnpm（推荐）
npm install -g pnpm

# 或使用 corepack（Node.js 18+）
corepack enable
corepack prepare pnpm@latest --activate
```

### 创建 React 项目的多种方式

**方式一：Vite（推荐，现代首选）**

Vite 是目前创建 React 项目的最佳选择，提供极快的开发服务器和优化的生产构建。

```bash
# 创建 React + TypeScript 项目
npm create vite@latest my-react-app -- --template react-ts

# 创建 React + JavaScript 项目
npm create vite@latest my-react-app -- --template react

# 进入项目目录
cd my-react-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview
```

**方式二：Create React App（传统方式，已不推荐新项目使用）**

```bash
# 创建项目（不推荐，性能较慢）
npx create-react-app my-react-app

cd my-react-app
npm start
```

**方式三：Next.js（全栈框架）**

Next.js 提供 SSR、SSG 等高级功能，适合需要 SEO 优化或全栈能力的项目。

```bash
# 创建 Next.js 项目
npx create-next-app@latest my-next-app

# 选项：
# - TypeScript: Yes
# - ESLint: Yes
# - Tailwind CSS: Yes
# - src/ directory: Yes
# - App Router: Yes
# - Import alias: @/*

cd my-next-app
pnpm dev
```

**方式四：使用 specific 版本创建**

```bash
# 创建特定版本的 React 项目
npm create vite@5.0.0 my-app -- --template react-ts

# 指定 React 19 + Next.js 15
npx create-next-app@latest my-app --version 15.0.0-rc.1
```

### 项目结构详解

**Vite + React 项目标准结构：**

```
my-react-app/
├── public/                    # 静态资源（不经过打包处理）
│   ├── favicon.ico
│   └── robots.txt
├── src/                      # 源代码目录
│   ├── assets/              # 需要处理的静态资源
│   │   ├── images/
│   │   └── styles/
│   ├── components/          # React 组件
│   │   ├── common/         # 通用组件（Button, Input, Modal）
│   │   ├── layout/         # 布局组件（Header, Footer, Sidebar）
│   │   └── features/       # 业务组件（UserCard, ProductList）
│   ├── hooks/              # 自定义 Hooks
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   └── useLocalStorage.ts
│   ├── contexts/           # React Context
│   │   ├── AuthContext.tsx
│   │   └── ThemeContext.tsx
│   ├── pages/             # 页面组件（路由页面）
│   │   ├── Home.tsx
│   │   ├── About.tsx
│   │   └── NotFound.tsx
│   ├── services/          # API 服务
│   │   ├── api.ts
│   │   └── userService.ts
│   ├── types/            # TypeScript 类型定义
│   │   └── index.ts
│   ├── utils/            # 工具函数
│   │   ├── formatDate.ts
│   │   └── validation.ts
│   ├── App.tsx          # 根组件
│   ├── main.tsx         # 入口文件
│   └── vite-env.d.ts    # Vite 类型声明
├── index.html            # HTML 入口
├── package.json         # 项目配置
├── tsconfig.json        # TypeScript 配置
├── vite.config.ts       # Vite 配置
└── .gitignore          # Git 忽略配置
```

**Next.js App Router 项目结构：**

```
my-next-app/
├── src/
│   ├── app/              # App Router 目录
│   │   ├── layout.tsx    # 根布局
│   │   ├── page.tsx      # 首页 /
│   │   ├── about/
│   │   │   └── page.tsx  # /about
│   │   ├── users/
│   │   │   ├── page.tsx  # /users
│   │   │   └── [id]/
│   │   │       └── page.tsx  # /users/:id
│   │   ├── api/          # API 路由
│   │   │   └── users/
│   │   │       └── route.ts
│   │   ├── layout.tsx    # 布局
│   │   └── globals.css   # 全局样式
│   ├── components/       # 组件
│   ├── lib/             # 工具库
│   └── types/           # 类型定义
├── public/             # 静态资源
├── next.config.js      # Next.js 配置
└── tsconfig.json       # TypeScript 配置
```

---

## 组件系统详解

### 函数组件 vs 类组件

React 19 中函数组件已经完全取代类组件，但理解类组件有助于阅读旧代码和理解 React 工作原理。

**类组件特点：**

```tsx
import { Component } from 'react';

// 类组件（已不推荐，但需了解）
class Counter extends Component<{ initialCount: number }, { count: number }> {
  constructor(props: { initialCount: number }) {
    super(props);
    this.state = { count: props.initialCount };
    // 绑定方法（避免 this 问题）
    this.handleClick = this.handleClick.bind(this);
  }
  
  handleClick() {
    // 更新状态
    this.setState({ count: this.state.count + 1 });
  }
  
  componentDidMount() {
    console.log('组件挂载');
  }
  
  componentWillUnmount() {
    console.log('组件卸载');
  }
  
  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={this.handleClick}>增加</button>
      </div>
    );
  }
}
```

**函数组件（推荐）：**

```tsx
import { useState, useEffect } from 'react';

interface CounterProps {
  initialCount?: number;
}

function Counter({ initialCount = 0 }: CounterProps) {
  // 使用 Hook 管理状态
  const [count, setCount] = useState(initialCount);
  
  useEffect(() => {
    console.log('组件挂载');
    return () => console.log('组件卸载');
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>增加</button>
    </div>
  );
}
```

### Props 系统

Props 是 React 组件之间传递数据的主要方式。

**基础 Props：**

```tsx
interface UserCardProps {
  name: string;
  email: string;
  avatar?: string;
  role: 'admin' | 'user' | 'guest';
}

function UserCard({ name, email, avatar, role }: UserCardProps) {
  const roleColors = {
    admin: 'bg-red-500',
    user: 'bg-blue-500',
    guest: 'bg-gray-500'
  };
  
  return (
    <div className="user-card">
      {avatar ? (
        <img src={avatar} alt={name} className="avatar" />
      ) : (
        <div className="avatar-placeholder">
          {name.charAt(0).toUpperCase()}
        </div>
      )}
      <h3>{name}</h3>
      <p>{email}</p>
      <span className={`role ${roleColors[role]}`}>
        {role}
      </span>
    </div>
  );
}

// 使用
<UserCard 
  name="张三" 
  email="zhangsan@example.com"
  role="admin"
/>
```

**带默认值的 Props：**

```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

// 方式一：默认参数
function Button({ 
  variant = 'primary', 
  size = 'md', 
  children 
}: ButtonProps) {
  return (
    <button className={`btn btn-${variant} btn-${size}`}>
      {children}
    </button>
  );
}

// 方式二：解构默认值
function Button({ 
  variant, 
  size, 
  children, 
  ...restProps 
}: ButtonProps) {
  const classes = [
    'btn',
    variant && `btn-${variant}`,
    size && `btn-${size}`
  ].filter(Boolean).join(' ');
  
  return <button className={classes} {...restProps}>{children}</button>;
}
```

**Children Props：**

```tsx
interface CardProps {
  title?: string;
  footer?: React.ReactNode;
  children: React.ReactNode;
}

function Card({ title, footer, children }: CardProps) {
  return (
    <div className="card">
      {title && <div className="card-header">{title}</div>}
      <div className="card-body">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

// 使用
<Card 
  title="用户信息"
  footer={<Button label="确定" />}
>
  <UserForm />
</Card>
```

**Render Props：**

```tsx
interface MouseTrackerProps {
  render: (position: { x: number; y: number }) => React.ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// 使用
<MouseTracker 
  render={({ x, y }) => (
    <div style={{ position: 'fixed', left: x, top: y }}>
      🐭
    </div>
  )} 
/>
```

---

## 状态管理方案详解

### 本地状态 vs 全局状态

| 状态类型 | 使用场景 | 推荐方案 |
|---------|---------|---------|
| **本地状态** | 单个组件内部使用 | useState |
| **提升状态** | 多个相邻组件共享 | 提升到父组件，通过 props 传递 |
| **全局状态** | 跨组件树共享 | Context / Zustand / Redux |
| **服务器状态** | 从 API 获取的数据 | React Query / SWR |

### useState 进阶用法

```tsx
import { useState, useCallback } from 'react';

// 1. 函数式更新（基于前一个状态）
function Counter() {
  const [count, setCount] = useState(0);
  
  // 推荐：使用函数式更新
  const increment = () => setCount(prev => prev + 1);
  const decrement = () => setCount(prev => prev - 1);
  
  // 批量更新
  const reset = () => {
    setCount(0);
    setTimeout(() => setCount(0), 0); // 第二个 setCount 会被批量处理
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// 2. 对象状态
interface FormState {
  username: string;
  email: string;
  password: string;
  remember: boolean;
}

function Form() {
  const [form, setForm] = useState<FormState>({
    username: '',
    email: '',
    password: '',
    remember: false
  });
  
  // 正确：使用函数式更新合并对象
  const updateField = <K extends keyof FormState>(
    field: K, 
    value: FormState[K]
  ) => {
    setForm(prev => ({ ...prev, [field]: value }));
  };
  
  // 错误：直接修改对象（不会触发更新）
  const wrongUpdate = () => {
    form.username = 'new name'; // ❌
    setForm(form); // 这会工作，但容易出错
  };
  
  return (
    <form>
      <input
        value={form.username}
        onChange={e => updateField('username', e.target.value)}
      />
      <input
        value={form.email}
        onChange={e => updateField('email', e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={form.remember}
          onChange={e => updateField('remember', e.target.checked)}
        />
        记住我
      </label>
    </form>
  );
}

// 3. 延迟初始化
function HeavyComponent() {
  // 延迟初始化：只在首次渲染时执行
  const [data, setData] = useState(() => {
    const savedData = localStorage.getItem('data');
    return savedData ? JSON.parse(savedData) : computeExpensiveInitial();
  });
  
  return <div>{JSON.stringify(data)}</div>;
}

function computeExpensiveInitial() {
  // 模拟昂贵计算
  console.log('执行昂贵计算...');
  return { items: Array.from({ length: 1000 }, (_, i) => i) };
}
```

### Context 深度指南

**基础 Context：**

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. 创建 Context
interface Theme {
  primary: string;
  secondary: string;
  background: string;
}

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
  isDark: boolean;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

// 2. Provider 组件
const lightTheme: Theme = {
  primary: '#007bff',
  secondary: '#6c757d',
  background: '#ffffff'
};

const darkTheme: Theme = {
  primary: '#0d6efd',
  secondary: '#adb5bd',
  background: '#212529'
};

export function ThemeProvider({ children }: { children: ReactNode }) {
  const [isDark, setIsDark] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved === 'dark';
  });
  
  const toggleTheme = () => {
    setIsDark(prev => {
      const newValue = !prev;
      localStorage.setItem('theme', newValue ? 'dark' : 'light');
      return newValue;
    });
  };
  
  const value: ThemeContextValue = {
    theme: isDark ? darkTheme : lightTheme,
    toggleTheme,
    isDark
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. 自定义 Hook 消费 Context
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 4. 使用
function ThemedButton() {
  const { theme, toggleTheme, isDark } = useTheme();
  
  return (
    <button 
      style={{ 
        background: theme.primary,
        color: isDark ? '#fff' : '#000'
      }}
      onClick={toggleTheme}
    >
      {isDark ? '切换亮色' : '切换暗色'}
    </button>
  );
}
```

**Context 性能优化：**

```tsx
import { createContext, useContext, useState, useMemo, useCallback } from 'react';

// 问题：Context 值每次渲染都会创建新对象
// const value = { theme, toggleTheme }; // ❌ 每次都是新对象

// 解决方案1：使用 useMemo 缓存值
interface AuthContextValue {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
}

function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  const login = useCallback(async (credentials: Credentials) => {
    const user = await authService.login(credentials);
    setUser(user);
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
  }, []);
  
  // 缓存 value 对象
  const value = useMemo(() => ({
    user,
    login,
    logout
  }), [user, login, logout]);
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// 解决方案2：拆分 Context
// 将频繁变化的状态和稳定的状态分开
const ThemeContext = createContext<Theme>(lightTheme);
const ThemeToggleContext = createContext<() => void>(() => {});

function ThemeProvider({ children }: { children: ReactNode }) {
  const [isDark, setIsDark] = useState(false);
  const toggleTheme = useCallback(() => setIsDark(prev => !prev), []);
  const theme = isDark ? darkTheme : lightTheme;
  
  return (
    <ThemeContext.Provider value={theme}>
      <ThemeToggleContext.Provider value={toggleTheme}>
        {children}
      </ThemeToggleContext.Provider>
    </ThemeContext.Provider>
  );
}
```

---

## 生命周期与副作用

### useEffect 完全指南

**基础用法：**

```tsx
import { useState, useEffect } from 'react';

// 1. 组件挂载时执行（模拟 componentDidMount）
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // 组件挂载后执行
    setLoading(true);
    
    fetchUser(userId)
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(() => {
        setLoading(false);
      });
  }, [userId]); // userId 变化时重新执行
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>{user.name}</div>;
}

// 2. 清理副作用（模拟 componentWillUnmount）
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    
    // 返回清理函数
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []); // 空依赖数组：仅在挂载和卸载时执行
  
  return (
    <div>
      鼠标位置: {position.x}, {position.y}
    </div>
  );
}

// 3. 条件执行 effect
function DataFetcher({ url, enabled }: { url: string; enabled: boolean }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    if (!enabled) return; // 条件判断
    
    const controller = new AbortController();
    
    fetch(url, { signal: controller.signal })
      .then(res => res.json())
      .then(setData)
      .catch(err => {
        if (err.name !== 'AbortError') {
          console.error('Fetch error:', err);
        }
      });
    
    return () => controller.abort(); // 清理未完成的请求
  }, [url, enabled]);
  
  return <div>{data ? JSON.stringify(data) : 'Disabled'}</div>;
}
```

**常见 useEffect 场景：**

```tsx
// 场景1：订阅外部数据源
function WebSocketComponent() {
  const [messages, setMessages] = useState<Message[]>([]);
  
  useEffect(() => {
    const ws = new WebSocket('wss://api.example.com');
    
    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      setMessages(prev => [...prev, message]);
    };
    
    // 清理：关闭 WebSocket 连接
    return () => {
      ws.close();
    };
  }, []); // 空依赖，只连接一次
  
  return <MessageList messages={messages} />;
}

// 场景2：操作 DOM
function Tooltip({ text, target }: { text: string; target: HTMLElement | null }) {
  useEffect(() => {
    if (!target) return;
    
    // 创建 tooltip
    const tooltip = document.createElement('div');
    tooltip.className = 'tooltip';
    tooltip.textContent = text;
    document.body.appendChild(tooltip);
    
    // 定位 tooltip
    const rect = target.getBoundingClientRect();
    tooltip.style.top = `${rect.bottom + 5}px`;
    tooltip.style.left = `${rect.left}px`;
    
    // 清理：移除 tooltip
    return () => {
      tooltip.remove();
    };
  }, [text, target]);
  
  return null; // 只操作 DOM，不渲染 React 元素
}

// 场景3：定时器
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // 清理：清除定时器
    return () => clearInterval(interval);
  }, []); // 空依赖，定时器只创建一次
  
  return <div>{seconds} 秒</div>;
}

// 场景4：依赖外部库
function Chart({ data }: { data: DataPoint[] }) {
  const containerRef = useRef<HTMLDivElement>(null);
  const chartRef = useRef<ChartInstance | null>(null);
  
  useEffect(() => {
    if (!containerRef.current) return;
    
    // 初始化图表
    chartRef.current = new Chart(containerRef.current, {
      type: 'line',
      data: { datasets: [{ data }] }
    });
    
    // 清理：销毁图表实例
    return () => {
      chartRef.current?.destroy();
    };
  }, []); // 只初始化一次
  
  // 数据变化时更新
  useEffect(() => {
    if (chartRef.current) {
      chartRef.current.data.datasets[0].data = data;
      chartRef.current.update();
    }
  }, [data]);
  
  return <div ref={containerRef} />;
}
```

**useEffect 依赖数组最佳实践：**

```tsx
// ❌ 常见错误：忘记依赖或依赖不正确
useEffect(() => {
  fetchData(id).then(setData);
}, []); // id 是必需的依赖

useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1); // count 是依赖
  }, 1000);
  return () => clearInterval(timer);
}, []); // ❌ 应该是 [count]

// ✅ 正确做法
useEffect(() => {
  fetchData(id).then(setData);
}, [id]);

useEffect(() => {
  const timer = setInterval(() => {
    setCount(prev => prev + 1); // 使用函数式更新
  }, 1000);
  return () => clearInterval(timer);
}, []); // ✅ 空依赖，setCount 函数式更新是稳定的

// ✅ 使用 useCallback 稳定依赖
const fetchData = useCallback(async (id: string) => {
  const result = await api.get(`/users/${id}`);
  return result;
}, []);

useEffect(() => {
  fetchData(id).then(setData);
}, [id, fetchData]); // ✅ fetchData 稳定
```

---

## React Router 完整指南

### 路由配置详解

```tsx
// router/index.tsx
import { createBrowserRouter, RouterProvider, Navigate } from 'react-router-dom';
import { lazy, Suspense } from 'react';

// 懒加载页面组件
const Home = lazy(() => import('@/pages/Home'));
const About = lazy(() => import('@/pages/About'));
const UserProfile = lazy(() => import('@/pages/UserProfile'));
const Dashboard = lazy(() => import('@/pages/Dashboard'));
const Settings = lazy(() => import('@/pages/Settings'));
const NotFound = lazy(() => import('@/pages/NotFound'));
const Login = lazy(() => import('@/pages/Login'));

// 布局组件
function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard-layout">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}

// 加载状态组件
function PageLoader() {
  return (
    <div className="page-loader">
      <Spinner />
    </div>
  );
}

// 路由配置
const router = createBrowserRouter([
  // 首页
  {
    path: '/',
    element: <Home />
  },
  
  // 登录页
  {
    path: '/login',
    element: <Login />
  },
  
  // 嵌套路由
  {
    path: '/dashboard',
    element: <DashboardLayout />,
    children: [
      {
        index: true, // 默认路由
        element: <Navigate to="/dashboard/main" replace />
      },
      {
        path: 'main',
        element: <Dashboard />
      },
      {
        path: 'settings',
        element: <Settings />
      }
    ]
  },
  
  // 动态路由
  {
    path: '/users/:userId', // :userId 是动态参数
    element: <UserProfile />
  },
  
  // 带查询参数的路由
  {
    path: '/search',
    element: <SearchResults />,
    loader: async ({ request }) => {
      const url = new URL(request.url);
      const query = url.searchParams.get('q');
      const results = await search(query);
      return { results };
    }
  },
  
  // 路由懒加载 + Suspense
  {
    path: '/about',
    element: (
      <Suspense fallback={<PageLoader />}>
        <About />
      </Suspense>
    )
  },
  
  // 通配符路由（404）
  {
    path: '*',
    element: <NotFound />
  }
]);

// 应用
function App() {
  return <RouterProvider router={router} />;
}
```

### 路由组件中使用

```tsx
import { useParams, useSearchParams, useNavigate, useLocation } from 'react-router-dom';

// 1. 动态路由参数
function UserProfile() {
  const { userId } = useParams<{ userId: string }>();
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    if (userId) {
      fetchUser(userId).then(setUser);
    }
  }, [userId]);
  
  return <div>User: {user?.name}</div>;
}

// 2. 查询参数
function SearchResults() {
  const [searchParams, setSearchParams] = useSearchParams();
  const query = searchParams.get('q') || '';
  const page = parseInt(searchParams.get('page') || '1');
  
  const updateSearch = (newQuery: string) => {
    setSearchParams({ q: newQuery, page: '1' });
  };
  
  const goToPage = (newPage: number) => {
    setSearchParams(prev => {
      prev.set('page', String(newPage));
      return prev;
    });
  };
  
  return (
    <div>
      <SearchInput value={query} onSearch={updateSearch} />
      <Results query={query} page={page} />
      <Pagination current={page} onChange={goToPage} />
    </div>
  );
}

// 3. 编程式导航
function LoginForm() {
  const navigate = useNavigate();
  const [error, setError] = useState('');
  
  const handleSubmit = async (credentials: Credentials) => {
    try {
      await login(credentials);
      // 登录成功后跳转
      navigate('/dashboard', { 
        replace: true // 替换当前历史记录
      });
    } catch (e) {
      setError('登录失败');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* ... */}
    </form>
  );
}

// 4. 获取路由信息
function Breadcrumb() {
  const location = useLocation();
  const pathnames = location.pathname.split('/').filter(Boolean);
  
  return (
    <nav>
      <Link to="/">首页</Link>
      {pathnames.map((name, index) => {
        const routeTo = `/${pathnames.slice(0, index + 1).join('/')}`;
        const isLast = index === pathnames.length - 1;
        
        return (
          <span key={name}>
            <span>/</span>
            {isLast ? (
              <span>{name}</span>
            ) : (
              <Link to={routeTo}>{name}</Link>
            )}
          </span>
        );
      })}
    </nav>
  );
}
```

---

## 表单处理详解

### 受控组件 vs 非受控组件

**受控组件（推荐）：**

```tsx
import { useState, useCallback } from 'react';
import { z } from 'zod';

// 表单验证 schema
const userSchema = z.object({
  name: z.string().min(2, '姓名至少2个字符'),
  email: z.string().email('请输入有效的邮箱'),
  age: z.number().min(18, '年龄必须大于18').optional(),
  role: z.enum(['admin', 'user', 'guest'])
});

type UserForm = z.infer<typeof userSchema>;

interface FormErrors {
  name?: string[];
  email?: string[];
  age?: string[];
  role?: string[];
}

function UserForm() {
  const [form, setForm] = useState<UserForm>({
    name: '',
    email: '',
    age: undefined,
    role: 'user'
  });
  
  const [errors, setErrors] = useState<FormErrors>({});
  const [touched, setTouched] = useState<Set<keyof UserForm>>(new Set());
  const [submitting, setSubmitting] = useState(false);
  
  // 字段更新
  const updateField = <K extends keyof UserForm>(
    field: K,
    value: UserForm[K]
  ) => {
    setForm(prev => ({ ...prev, [field]: value }));
    
    // 验证单个字段
    const result = userSchema.safeParse(form);
    if (!result.success) {
      const fieldError = result.error.flatten().fieldErrors[field];
      setErrors(prev => ({ 
        ...prev, 
        [field]: fieldError 
      }));
    } else {
      setErrors(prev => ({ ...prev, [field]: undefined }));
    }
  };
  
  // 字段失焦
  const handleBlur = (field: keyof UserForm) => {
    setTouched(prev => new Set(prev).add(field));
    
    // 验证字段
    const result = userSchema.safeParse(form);
    if (!result.success) {
      const fieldError = result.error.flatten().fieldErrors[field];
      setErrors(prev => ({ 
        ...prev, 
        [field]: fieldError 
      }));
    }
  };
  
  // 提交表单
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // 标记所有字段为已触碰
    setTouched(new Set(['name', 'email', 'age', 'role']));
    
    // 完整验证
    const result = userSchema.safeParse(form);
    if (!result.success) {
      setErrors(result.error.flatten().fieldErrors);
      return;
    }
    
    setSubmitting(true);
    try {
      await submitUser(result.data);
    } catch (e) {
      console.error('提交失败', e);
    } finally {
      setSubmitting(false);
    }
  };
  
  // 检查字段是否显示错误
  const showError = (field: keyof UserForm) => {
    return touched.has(field) && errors[field];
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div className="form-group">
        <label htmlFor="name">姓名</label>
        <input
          id="name"
          type="text"
          value={form.name}
          onChange={e => updateField('name', e.target.value)}
          onBlur={() => handleBlur('name')}
          className={showError('name') ? 'error' : ''}
        />
        {showError('name') && (
          <span className="error-message">{errors.name}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="email">邮箱</label>
        <input
          id="email"
          type="email"
          value={form.email}
          onChange={e => updateField('email', e.target.value)}
          onBlur={() => handleBlur('email')}
          className={showError('email') ? 'error' : ''}
        />
        {showError('email') && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>
      
      <div className="form-group">
        <label htmlFor="role">角色</label>
        <select
          id="role"
          value={form.role}
          onChange={e => updateField('role', e.target.value as UserForm['role'])}
        >
          <option value="admin">管理员</option>
          <option value="user">普通用户</option>
          <option value="guest">访客</option>
        </select>
      </div>
      
      <button type="submit" disabled={submitting}>
        {submitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

**非受控组件：**

```tsx
import { useRef } from 'react';

// 非受控组件：使用 ref 直接访问 DOM
function FileUpload() {
  const fileInputRef = useRef<HTMLInputElement>(null);
  const [preview, setPreview] = useState<string | null>(null);
  
  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const file = e.target.files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setPreview(reader.result as string);
      };
      reader.readAsDataURL(file);
    }
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    const file = fileInputRef.current?.files?.[0];
    if (file) {
      uploadFile(file);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input 
        ref={fileInputRef}
        type="file"
        accept="image/*"
        onChange={handleFileChange}
      />
      {preview && <img src={preview} alt="Preview" />}
      <button type="submit">上传</button>
    </form>
  );
}
```

### React Hook Form（推荐）

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { userSchema, UserForm } from './schema';

function ReactHookFormDemo() {
  const {
    register,           // 注册字段
    handleSubmit,       // 处理提交
    formState: {       // 表单状态
      errors,          // 错误信息
      touchedFields,   // 已触碰字段
      isSubmitting,     // 提交中
      isValid          // 是否有效
    },
    watch,              // 监听字段值
    setValue,           // 设置字段值
    reset               // 重置表单
  } = useForm<UserForm>({
    resolver: zodResolver(userSchema),
    defaultValues: {
      name: '',
      email: '',
      role: 'user'
    },
    mode: 'onBlur'      // 验证时机
  });
  
  // 监听字段值
  const email = watch('email');
  
  // 自定义验证
  const customValidation = async (value: string) => {
    const exists = await checkEmailExists(value);
    if (exists) {
      return '该邮箱已被注册';
    }
    return true;
  };
  
  const onSubmit = async (data: UserForm) => {
    try {
      await submitUser(data);
      reset();
    } catch (e) {
      console.error('提交失败', e);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>姓名</label>
        <input {...register('name')} />
        {errors.name && <span>{errors.name.message}</span>}
      </div>
      
      <div>
        <label>邮箱</label>
        <input {...register('email', {
          validate: customValidation
        })} />
        {errors.email && <span>{errors.email.message}</span>}
      </div>
      
      <div>
        <label>角色</label>
        <select {...register('role')}>
          <option value="admin">管理员</option>
          <option value="user">用户</option>
          <option value="guest">访客</option>
        </select>
        {errors.role && <span>{errors.role.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? '提交中...' : '提交'}
      </button>
    </form>
  );
}
```

---

## 样式方案详解

### CSS Modules

```css
/* Button.module.css */
.button {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
}

.primary {
  background: #007bff;
  color: white;
}

.secondary {
  background: #6c757d;
  color: white;
}

.button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 伪类 */
.button:hover:not(:disabled) {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
```

```tsx
/* Button.tsx */
import styles from './Button.module.css';

interface ButtonProps {
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}

function Button({ 
  variant = 'primary', 
  disabled = false, 
  onClick, 
  children 
}: ButtonProps) {
  const className = [
    styles.button,
    styles[variant]
  ].join(' ');
  
  return (
    <button 
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

### Tailwind CSS（推荐）

```tsx
// Tailwind 按钮组件
function TailwindButton({ 
  variant = 'primary', 
  size = 'md',
  disabled = false,
  onClick,
  children 
}: ButtonProps) {
  const baseClasses = 'font-medium rounded transition-all duration-200';
  
  const variantClasses = {
    primary: 'bg-blue-500 text-white hover:bg-blue-600 active:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 active:bg-gray-400',
    danger: 'bg-red-500 text-white hover:bg-red-600 active:bg-red-700',
    ghost: 'bg-transparent text-gray-700 hover:bg-gray-100'
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };
  
  return (
    <button
      className={`
        ${baseClasses}
        ${variantClasses[variant]}
        ${sizeClasses[size]}
        ${disabled ? 'opacity-50 cursor-not-allowed' : ''}
      `}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// Tailwind 响应式设计
function ResponsiveGrid() {
  return (
    <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
      {/* 1列 → 2列 → 3列 → 4列 */}
      {items.map(item => (
        <div key={item.id} className="p-4 bg-white shadow rounded">
          {item.content}
        </div>
      ))}
    </div>
  );
}

// Tailwind 暗色模式
function DarkModeCard() {
  return (
    <div className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow">
      <h2 className="text-gray-900 dark:text-white">标题</h2>
      <p className="text-gray-600 dark:text-gray-300">内容</p>
    </div>
  );
}
```

### Styled Components

```tsx
import styled, { css, createGlobalStyle } from 'styled-components';

// 主题类型
interface Theme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
  };
}

const theme: Theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff'
  }
};

// 样式化组件
const Button = styled.button<{ variant?: 'primary' | 'secondary' }>`
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  
  ${({ variant = 'primary', theme }) => css`
    background: ${variant === 'primary' 
      ? theme.colors.primary 
      : theme.colors.secondary};
    color: white;
  `}
  
  &:hover {
    opacity: 0.9;
    transform: translateY(-1px);
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;

const Card = styled.div<{ highlighted?: boolean }>`
  padding: 1rem;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  
  ${({ highlighted }) => highlighted && css`
    border: 2px solid ${theme.colors.primary};
  `}
`;

const Input = styled.input`
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  
  &:focus {
    outline: none;
    border-color: ${({ theme }) => theme.colors.primary};
  }
`;

// 全局样式
const GlobalStyle = createGlobalStyle`
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  body {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
    background: ${({ theme }) => theme.colors.background};
  }
`;

// 使用
function StyledDemo() {
  return (
    <>
      <GlobalStyle />
      <Card highlighted>
        <Input placeholder="输入..." />
        <Button variant="primary">提交</Button>
        <Button variant="secondary">取消</Button>
      </Card>
    </>
  );
}
```

---

## 性能优化详解

### React.memo 与 PureComponent

```tsx
import { memo, useMemo, useCallback, useState } from 'react';

// 1. React.memo - 包装函数组件
const UserCard = memo(function UserCard({ 
  user, 
  onEdit,
  onDelete 
}: { 
  user: User; 
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}) {
  console.log('UserCard 渲染', user.id);
  
  return (
    <div className="user-card">
      <span>{user.name}</span>
      <button onClick={() => onEdit(user.id)}>编辑</button>
      <button onClick={() => onDelete(user.id)}>删除</button>
    </div>
  );
});

// 2. 自定义比较函数
const OptimizedList = memo(function OptimizedList({ 
  items, 
  onItemClick 
}: { 
  items: Item[]; 
  onItemClick: (id: string) => void;
}) {
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </li>
      ))}
    </ul>
  );
}, (prevProps, nextProps) => {
  // 自定义比较逻辑
  // 返回 true 表示 props 相等，不需要重新渲染
  return (
    prevProps.items.length === nextProps.items.length &&
    prevProps.items.every((prev, i) => 
      prev.id === nextProps.items[i].id &&
      prev.name === nextProps.items[i].name
    )
  );
});

// 3. 配合 useCallback 使用
function ParentComponent() {
  const [users, setUsers] = useState<User[]>([]);
  const [searchTerm, setSearchTerm] = useState('');
  
  // 稳定函数引用
  const handleEdit = useCallback((id: string) => {
    console.log('编辑', id);
  }, []);
  
  const handleDelete = useCallback((id: string) => {
    console.log('删除', id);
    setUsers(prev => prev.filter(u => u.id !== id));
  }, []);
  
  // 缓存计算结果
  const filteredUsers = useMemo(() => {
    return users.filter(u => 
      u.name.includes(searchTerm)
    );
  }, [users, searchTerm]);
  
  return (
    <div>
      <input 
        value={searchTerm}
        onChange={e => setSearchTerm(e.target.value)}
      />
      <UserList 
        users={filteredUsers}
        onEdit={handleEdit}
        onDelete={handleDelete}
      />
    </div>
  );
}
```

### useMemo 与 useCallback 深度理解

```tsx
import { useMemo, useCallback, useRef } from 'react';

// 1. useMemo - 缓存计算结果
function DataTable({ data, sortKey, sortOrder }) {
  // 昂贵计算
  const sortedData = useMemo(() => {
    console.log('执行排序...');
    return [...data].sort((a, b) => {
      const aVal = a[sortKey];
      const bVal = b[sortKey];
      if (aVal < bVal) return sortOrder === 'asc' ? -1 : 1;
      if (aVal > bVal) return sortOrder === 'asc' ? 1 : -1;
      return 0;
    });
  }, [data, sortKey, sortOrder]);
  
  // 缓存配置对象
  const tableConfig = useMemo(() => ({
    columns: ['id', 'name', 'email'],
    sortable: true,
    pageSize: 20
  }), []);
  
  return <Table data={sortedData} config={tableConfig} />;
}

// 2. useCallback - 缓存函数引用
function SearchComponent({ onSearch }) {
  const [query, setQuery] = useState('');
  
  // 每次渲染都创建新函数（问题）
  const handleSearch = (q) => {
    onSearch(q);
  };
  
  // 稳定的函数引用（解决方案）
  const stableSearch = useCallback((q: string) => {
    onSearch(q);
  }, [onSearch]);
  
  // 当 query 是依赖时
  const searchWithQuery = useCallback(() => {
    onSearch(query);
  }, [onSearch, query]);
  
  return (
    <div>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <button onClick={() => handleSearch(query)}>搜索</button>
      <button onClick={searchWithQuery}>搜索（使用 query）</button>
    </div>
  );
}

// 3. useRef 替代 useCallback（当不需要依赖时）
function Timer() {
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  const countRef = useRef(0);
  
  const startTimer = () => {
    if (intervalRef.current) return;
    
    intervalRef.current = setInterval(() => {
      countRef.current += 1;
      console.log(countRef.current);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };
  
  return (
    <div>
      <button onClick={startTimer}>开始</button>
      <button onClick={stopTimer}>停止</button>
    </div>
  );
}
```

### React DevTools Profiler 分析

```tsx
// 1. 使用 Profiler 组件测量性能
import { Profiler } from 'react';

function App() {
  const handleRender = (
    id, // Profiler id
    phase, // 'mount' | 'update'
    actualDuration, // 本次渲染花费的时间
    baseDuration, // 估计渲染时间
    startTime, // 开始时间
    commitTime, // 提交时间
    interactions // 交互集合
  ) => {
    console.log({
      id,
      phase,
      actualDuration,
      baseDuration,
      startTime,
      commitTime,
      interactions
    });
    
    // 如果渲染时间过长，可以记录警告
    if (actualDuration > baseDuration * 1.5) {
      console.warn(`组件 ${id} 渲染时间过长: ${actualDuration}ms`);
    }
  };
  
  return (
    <Profiler id="App" onRender={handleRender}>
      <Router>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
        </Routes>
      </Router>
    </Profiler>
  );
}

// 2. 虚拟化长列表
import { FixedSizeList as List } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      Item {items[index].name}
    </div>
  );
  
  return (
    <List
      height={400}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </List>
  );
}

// 3. 延迟加载组件
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const DataTable = lazy(() => import('./DataTable'));

function Dashboard() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyChart />
      </Suspense>
      <Suspense fallback={<div>Loading table...</div>}>
        <DataTable />
      </Suspense>
    </div>
  );
}
```

---

## 生态工具链

### 开发工具

| 工具 | 用途 | 官方链接 |
|------|------|---------|
| **Vite** | 快速构建工具 | https://vitejs.dev |
| **ESLint** | 代码检查 | https://eslint.org |
| **Prettier** | 代码格式化 | https://prettier.io |
| **TypeScript** | 类型系统 | https://typescriptlang.org |
| **React Developer Tools** | Chrome/Firefox 调试插件 | Chrome 商店搜索 |

### 测试工具

```bash
# 安装测试依赖
pnpm add -D vitest @testing-library/react @testing-library/jest-dom jsdom
```

```tsx
// 组件测试示例
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Counter } from './Counter';

// Mock 函数
const mockOnIncrement = vi.fn();

describe('Counter', () => {
  it('renders initial count', () => {
    render(<Counter initialCount={5} onIncrement={mockOnIncrement} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });
  
  it('increments when button is clicked', () => {
    render(<Counter initialCount={0} onIncrement={mockOnIncrement} />);
    
    const button = screen.getByRole('button', { name: /increment/i });
    fireEvent.click(button);
    
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
    expect(mockOnIncrement).toHaveBeenCalledTimes(1);
  });
  
  it('decrements when decrement button is clicked', () => {
    render(<Counter initialCount={5} onIncrement={mockOnIncrement} />);
    
    const button = screen.getByRole('button', { name: /decrement/i });
    fireEvent.click(button);
    
    expect(screen.getByText('Count: 4')).toBeInTheDocument();
  });
});
```

### 状态管理工具对比

| 工具 | 适用场景 | 复杂度 | 性能 | 推荐指数 |
|------|---------|--------|------|---------|
| **useState** | 简单状态 | ⭐ | 最佳 | ⭐⭐⭐⭐⭐ |
| **useReducer** | 中等复杂度 | ⭐⭐ | 良好 | ⭐⭐⭐⭐ |
| **Context** | 跨组件共享 | ⭐⭐ | 一般 | ⭐⭐⭐ |
| **Zustand** | 中大型应用 | ⭐⭐ | 优秀 | ⭐⭐⭐⭐⭐ |
| **Jotai** | 原子化状态 | ⭐⭐ | 优秀 | ⭐⭐⭐⭐ |
| **Redux Toolkit** | 大型企业 | ⭐⭐⭐ | 良好 | ⭐⭐⭐⭐ |
| **TanStack Query** | 服务器状态 | ⭐⭐ | 优秀 | ⭐⭐⭐⭐⭐ |

---

## 学习路径与资源

### React 学习路线图

```
第一阶段：基础（1-2周）
├── JavaScript ES6+ 语法
├── JSX 基础
├── 组件与 Props
├── State 与事件处理
└── 条件渲染与列表

第二阶段：进阶（2-3周）
├── Hooks 完整掌握
├── Context 与状态提升
├── React Router 基础
├── 表单处理
└── 组件组合模式

第三阶段：生态（2-3周）
├── TypeScript 集成
├── 状态管理（Zustand/Redux）
├── API 调用（TanStack Query）
├── 测试基础（Jest + Testing Library）
└── CSS 方案（Tailwind/CSS Modules）

第四阶段：高级（持续学习）
├── React Server Components
├── React Compiler
├── 性能优化
├── SSR/SSG（Next.js）
└── 全栈开发
```

### 推荐学习资源

**官方文档：**
- React 官方文档：https://react.dev
- React 中文文档：https://react.nodejs.cn
- React 18 新特性：https://react.dev/blog

**教程与课程：**
| 资源 | 类型 | 难度 | 链接 |
|------|------|------|------|
| React 官方教程 | 交互式 | 入门 | react.dev/learn |
| 饥人谷 React 教程 | 视频 | 入门 | 慕课网/B站 |
| Epic React | 视频 | 进阶 | Kent C. Dodds |
| React 最佳实践 | 文章 | 进阶 | Overreacted |

**优质博客与 Newsletter：**
- Overreacted (Dan Abramov)：https://overreacted.io
- Kent C. Dodds Blog：https://kentcdodds.com
- React Blog：https://react.dev/blog
- JavaScript Weekly Newsletter

**开源项目学习：**
- Next.js App Router 项目模板
- React Admin 后台管理模板
- React 官方示例：https://github.com/reactjs

---

## 高级模式与最佳实践

### Error Boundaries 与错误处理

Error Boundaries 是 React 错误处理的核心机制：

```tsx
import { Component, ReactNode } from 'react';

// 自定义 Error Boundary
interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }
      return (
        <div className="error-container">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }
    return this.props.children;
  }
}

// 函数式 Error Boundary (React 18+)
function useErrorBoundary() {
  const [error, setError] = useState<Error | null>(null);
  
  if (error) {
    throw error;
  }
  
  return useCallback((err: Error) => setError(err), []);
}

// 使用
function App() {
  return (
    <ErrorBoundary fallback={<div>Error occurred</div>}>
      <ComponentThatMightError />
    </ErrorBoundary>
  );
}
```

### Portal 与 Teleport

Portal 允许将子组件渲染到 DOM 树的其他位置：

```tsx
import { createPortal } from 'react-dom';

// 基础 Portal 用法
function Modal({ children, isOpen }: { children: ReactNode; isOpen: boolean }) {
  if (!isOpen) return null;
  
  return createPortal(
    <div className="modal-overlay">
      <div className="modal-content">
        {children}
      </div>
    </div>,
    document.body
  );
}

// 高级 Portal - 事件冒泡处理
function DragDropZone({ children }) {
  const [isDragging, setIsDragging] = useState(false);
  
  const handleDragEnter = (e: DragEvent) => {
    e.preventDefault();
    setIsDragging(true);
  };
  
  const handleDragLeave = (e: DragEvent) => {
    e.preventDefault();
    if (e.currentTarget === e.target) {
      setIsDragging(false);
    }
  };
  
  return (
    <div
      onDragEnter={handleDragEnter}
      onDragLeave={handleDragLeave}
      onDragOver={(e) => e.preventDefault()}
      onDrop={(e) => {
        e.preventDefault();
        setIsDragging(false);
      }}
    >
      {children}
    </div>
  );
}
```

### Concurrent Mode 深入理解

React 18 的 Concurrent Mode 带来了新的渲染能力：

```tsx
import { 
  useTransition, 
  useDeferredValue, 
  useSyncExternalStore, 
  Suspense 
} from 'react';

// 1. useTransition - 非紧急更新
function SearchResults({ query }: { query: string }) {
  const [isPending, startTransition] = useTransition();
  const [results, setResults] = useState([]);
  
  const handleSearch = (newQuery: string) => {
    startTransition(() => {
      setResults(performSearch(newQuery));
    });
  };
  
  return (
    <div>
      <input onChange={(e) => handleSearch(e.target.value)} />
      {isPending ? <Spinner /> : <ResultsList results={results} />}
    </div>
  );
}

// 2. useDeferredValue - 延迟值
function Typeahead({ text }: { text: string }) {
  const deferredText = useDeferredValue(text);
  
  return (
    <div>
      <input value={text} />
      <Suspense fallback={<Loading />}>
        <Results query={deferredText} />
      </Suspense>
    </div>
  );
}

// 3. useSyncExternalStore - 外部数据源
function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    (subscribe) => {
      window.addEventListener('online', subscribe);
      window.addEventListener('offline', subscribe);
      return () => {
        window.removeEventListener('online', subscribe);
        window.removeEventListener('offline', subscribe);
      };
    },
    () => navigator.onLine,
    () => true
  );
  
  return isOnline;
}
```

### 高阶组件（HOC）模式

HOC 是 React 中的高级复用模式：

```tsx
import { ComponentType, ReactElement } from 'react';

// 1. 属性代理 HOC
function withLoading<T extends object>(
  Component: ComponentType<T>,
  isLoading: boolean
): ComponentType<T> {
  return function WithLoadingComponent(props: T) {
    if (isLoading) {
      return <div className="loading-spinner">Loading...</div>;
    }
    return <Component {...props} />;
  };
}

// 2. 继承反转 HOC
function withErrorBoundary<T extends object>(
  Component: ComponentType<T>,
  errorHandler?: (error: Error) => void
): ComponentType<T> {
  return class WithErrorBoundary extends Component<T, { hasError: boolean }> {
    state = { hasError: false };
    
    static getDerivedStateFromError(error: Error) {
      errorHandler?.(error);
      return { hasError: true };
    }
    
    render() {
      if (this.state.hasError) {
        return <div>Error occurred</div>;
      }
      return <Component {...this.props} />;
    }
  };
}

// 3. 条件渲染 HOC
function withPermission<T extends object>(
  WrappedComponent: ComponentType<T>,
  requiredPermission: string
): ComponentType<T> {
  return function WithPermissionComponent(props: T) {
    const hasPermission = useCheckPermission(requiredPermission);
    
    if (!hasPermission) {
      return <div>Access denied</div>;
    }
    
    return <WrappedComponent {...props} />;
  };
}

// 使用示例
const DataTableWithLoading = withLoading(DataTable, isLoading);
const ProtectedDashboard = withPermission(Dashboard, 'admin:read');
```

### Render Props 模式

Render Props 提供了一种共享逻辑的方式：

```tsx
import { ReactNode, MouseEvent } from 'react';

// 1. 基础 Render Props
interface MouseTrackerProps {
  render: (position: { x: number; y: number }) => ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return render(position);
}

// 使用
<MouseTracker
  render={({ x, y }) => (
    <div style={{ position: 'fixed', left: x, top: y }}>
      🐭
    </div>
  )}
/>

// 2. children as Function
interface DataFetcherProps<T> {
  children: (data: T, loading: boolean) => ReactNode;
  url: string;
}

function DataFetcher<T>({ children, url }: DataFetcherProps<T>) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(data => {
        setData(data);
        setLoading(false);
      });
  }, [url]);
  
  return <>{children(data, loading)}</>;
}

// 使用
<DataFetcher url="/api/users">
  {(users, loading) => (
    loading ? <Spinner /> : <UserList users={users} />
  )}
</DataFetcher>
```

### 合成事件系统

React 的合成事件系统提供跨浏览器的一致性：

```tsx
// 事件处理中的 this 指向
class EventComponent extends Component {
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  
  handleClick(e: SyntheticEvent) {
    console.log('Event type:', e.type);
    console.log('Target:', e.target);
    console.log('Current target:', e.currentTarget);
  }
  
  render() {
    return <button onClick={this.handleClick}>Click</button>;
  }
}

// 事件委托
function DelegatedEvents() {
  const handleClick = (e: SyntheticEvent) => {
    e.stopPropagation();
  };
  
  const handleParentClick = () => {
    console.log('Parent clicked');
  };
  
  return (
    <div onClick={handleParentClick}>
      <button onClick={handleClick}>Stop propagation</button>
    </div>
  );
}

// 阻止默认行为
function PreventDefault() {
  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
  };
  
  return <form onSubmit={handleSubmit}>...</form>;
}
```

### 动画与过渡

React 中的动画实现方案：

```tsx
// 1. CSS 过渡
function CSSAnimation() {
  const [isVisible, setIsVisible] = useState(false);
  
  return (
    <div
      className={`fade ${isVisible ? 'visible' : ''}`}
      style={{
        transition: 'opacity 0.3s ease-in-out',
        opacity: isVisible ? 1 : 0
      }}
    >
      Content
    </div>
  );
}

// 2. Framer Motion 集成
import { motion, AnimatePresence } from 'framer-motion';

function FramerAnimation() {
  const [isVisible, setIsVisible] = useState(true);
  
  return (
    <div>
      <button onClick={() => setIsVisible(!isVisible)}>Toggle</button>
      
      <motion.div
        initial={{ opacity: 0, scale: 0.5 }}
        animate={{ opacity: 1, scale: 1 }}
        exit={{ opacity: 0, scale: 0.5 }}
        transition={{ duration: 0.3 }}
        whileHover={{ scale: 1.1 }}
        whileTap={{ scale: 0.95 }}
      >
        Animated Content
      </motion.div>
      
      <AnimatePresence>
        {isVisible && (
          <motion.div
            key="modal"
            initial={{ y: -100, opacity: 0 }}
            animate={{ y: 0, opacity: 1 }}
            exit={{ y: 100, opacity: 0 }}
          >
            Modal Content
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}

// 3. 列表动画
function AnimatedList() {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);
  
  return (
    <motion.ul>
      <AnimatePresence>
        {items.map(item => (
          <motion.li
            key={item}
            initial={{ opacity: 0, x: -100 }}
            animate={{ opacity: 1, x: 0 }}
            exit={{ opacity: 0, x: 100 }}
            layout
          >
            Item {item}
          </motion.li>
        ))}
      </AnimatePresence>
    </motion.ul>
  );
}
```

### 服务端渲染（SSR）进阶

Next.js App Router 的 SSR/SSG 模式：

```tsx
// app/page.tsx - Server Component
import { cache } from 'react';
import { db } from '@/lib/db';

// 缓存数据加载
const getUser = cache(async (id: string) => {
  return await db.user.findUnique({ where: { id } });
});

// Server Component - 直接访问数据库
export default async function UserProfile({ params }: { params: { id: string } }) {
  const user = await getUser(params.id);
  
  if (!user) {
    notFound();
  }
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 动态元数据
export async function generateMetadata({ params }: { params: { id: string } }) {
  const user = await getUser(params.id);
  
  return {
    title: user ? `${user.name} - Profile` : 'User not found',
    description: user?.bio || 'User profile page'
  };
}

// 静态生成
export async function generateStaticParams() {
  const users = await db.user.findMany();
  return users.map(user => ({ id: user.id }));
}
```

```tsx
// app/users/[id]/page.tsx - Streaming
import { Suspense } from 'react';

export default function UserPage({ params }: { params: { id: string } }) {
  return (
    <div>
      <UserHeader userId={params.id} />
      
      <Suspense fallback={<UserSkeleton />}>
        <UserDetails userId={params.id} />
      </Suspense>
      
      <Suspense fallback={<PostsSkeleton />}>
        <UserPosts userId={params.id} />
      </Suspense>
    </div>
  );
}

// 并行数据获取
async function UserHeader({ userId }: { userId: string }) {
  const user = await fetchUser(userId);
  return <h1>{user.name}</h1>;
}

async function UserDetails({ userId }: { userId: string }) {
  const [user, stats] = await Promise.all([
    fetchUser(userId),
    fetchUserStats(userId)
  ]);
  
  return (
    <div>
      <p>{user.bio}</p>
      <p>Posts: {stats.postCount}</p>
    </div>
  );
}
```

### 国际化（i18n）实现

React 应用国际化方案：

```tsx
import { createContext, useContext, ReactNode } from 'react';
import messages from './messages';

interface I18nContextType {
  locale: string;
  t: (key: string, params?: Record<string, string>) => string;
  setLocale: (locale: string) => void;
}

const I18nContext = createContext<I18nContextType | null>(null);

export function I18nProvider({ 
  children, 
  locale: initialLocale 
}: { 
  children: ReactNode;
  locale: string;
}) {
  const [locale, setLocale] = useState(initialLocale);
  
  const t = (key: string, params?: Record<string, string>) => {
    const keys = key.split('.');
    let value: any = messages[locale];
    
    for (const k of keys) {
      value = value?.[k];
    }
    
    if (typeof value !== 'string') {
      console.warn(`Missing translation: ${key}`);
      return key;
    }
    
    if (params) {
      return value.replace(/\{(\w+)\}/g, (_, k) => params[k] || `{${k}}`);
    }
    
    return value;
  };
  
  return (
    <I18nContext.Provider value={{ locale, t, setLocale }}>
      {children}
    </I18nContext.Provider>
  );
}

export function useI18n() {
  const context = useContext(I18nContext);
  if (!context) {
    throw new Error('useI18n must be used within I18nProvider');
  }
  return context;
}

function WelcomeMessage() {
  const { t, locale, setLocale } = useI18n();
  
  return (
    <div>
      <h1>{t('common.welcome')}</h1>
      <p>{t('user.greeting', { name: 'John' })}</p>
      
      <select value={locale} onChange={(e) => setLocale(e.target.value)}>
        <option value="en">English</option>
        <option value="zh">中文</option>
      </select>
    </div>
  );
}
```

### 测试进阶

React Testing Library 深度使用：

```tsx
import { render, screen, fireEvent, waitFor, within } from '@testing-library/react';
import { server, rest } from '../test/server';

const mockUsers = [
  { id: 1, name: 'John', email: 'john@example.com' },
  { id: 2, name: 'Jane', email: 'jane@example.com' }
];

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('UserList', () => {
  it('renders user list', async () => {
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
    });
  });
  
  it('handles loading state', () => {
    render(<UserList loading={true} />);
    expect(screen.getByRole('progressbar')).toBeInTheDocument();
  });
  
  it('handles error state', async () => {
    server.use(
      rest.get('/api/users', (req, res, ctx) => {
        return res(ctx.status(500), ctx.json({ error: 'Server Error' }));
      })
    );
    
    render(<UserList />);
    
    await waitFor(() => {
      expect(screen.getByText(/error/i)).toBeInTheDocument();
    });
  });
  
  it('filters users', async () => {
    render(<UserList users={mockUsers} />);
    
    const searchInput = screen.getByPlaceholderText(/search/i);
    fireEvent.change(searchInput, { target: { value: 'John' } });
    
    await waitFor(() => {
      expect(screen.getByText('John')).toBeInTheDocument();
      expect(screen.queryByText('Jane')).not.toBeInTheDocument();
    });
  });
  
  it('handles user deletion', async () => {
    const onDelete = jest.fn();
    render(<UserList users={mockUsers} onDelete={onDelete} />);
    
    const deleteButtons = screen.getAllByRole('button', { name: /delete/i });
    fireEvent.click(deleteButtons[0]);
    
    await waitFor(() => {
      expect(onDelete).toHaveBeenCalledWith(1);
    });
  });
});

describe('Form', () => {
  it('validates email format', async () => {
    render(<UserForm />);
    
    const emailInput = screen.getByLabelText(/email/i);
    fireEvent.change(emailInput, { target: { value: 'invalid-email' } });
    fireEvent.blur(emailInput);
    
    expect(await screen.findByText(/valid email/i)).toBeInTheDocument();
  });
  
  it('submits form with valid data', async () => {
    const onSubmit = jest.fn();
    render(<UserForm onSubmit={onSubmit} />);
    
    fireEvent.change(screen.getByLabelText(/name/i), {
      target: { value: 'John' }
    });
    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'john@example.com' }
    });
    
    fireEvent.click(screen.getByRole('button', { name: /submit/i }));
    
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        name: 'John',
        email: 'john@example.com'
      });
    });
  });
});
```

---

## React 生态工具链详解

### 构建工具对比

| 工具 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Vite** | 极速 HMR，开发体验好 | 生态相对较新 | 现代 React 项目 |
| **Webpack** | 生态成熟，插件丰富 | 配置复杂，构建慢 | 大型企业项目 |
| **Parcel** | 零配置 | 灵活性较低 | 简单项目 |
| **esbuild** | 极快编译 | 功能有限 | 工具类项目 |
| **Turbopack** | Rust 实现，极快 | 仍在发展中 | Next.js 项目 |

```bash
# Vite 配置示例
# vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'axios']
        }
      }
    }
  }
});
```

### 代码分割策略

```tsx
import { lazy, Suspense } from 'react';

// 1. 基于路由的分割
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));
const Users = lazy(() => import('./pages/Users'));

// 2. 基于条件的分割
const HeavyChart = condition ? 
  lazy(() => import('./components/HeavyChart')) : 
  null;

// 3. 命名导出
const Modal = lazy(() => 
  import('./components/Modal').then(module => ({ 
    default: module.Modal 
  }))
);

// 4. 预加载
const PrefetchExample = lazy(() => import('./HeavyComponent'));

function PrefetchButton() {
  const handleMouseEnter = () => {
    import('./HeavyComponent');
  };
  
  return (
    <button onMouseEnter={handleMouseEnter}>
      Show Heavy Component
    </button>
  );
}

// 5. 加载状态
function App() {
  return (
    <Suspense fallback={<FullPageSpinner />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Suspense>
  );
}
```

### CSS-in-JS 方案对比

| 方案 | 优点 | 缺点 | 性能 |
|------|------|------|------|
| **Styled Components** | 样式封装，主题支持 | 运行时开销 | 中等 |
| **Emotion** | 快速，灵活 | 调试较难 | 良好 |
| **Tailwind CSS** | 极快，无运行时 | 学习曲线 | 最佳 |
| **CSS Modules** | 原生支持，简单 | 命名限制 | 最佳 |
| **Linaria** | 编译时，无运行时 | 限制较多 | 最佳 |

```tsx
// Styled Components 示例
import styled from 'styled-components';

const Button = styled.button`
  padding: ${props => props.$size === 'lg' ? '16px 32px' : '8px 16px'};
  background: ${props => props.$variant === 'primary' ? '#007bff' : '#6c757d'};
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  
  &:hover {
    opacity: 0.9;
  }
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
`;

function App() {
  return (
    <>
      <Button $variant="primary" $size="lg">Primary</Button>
      <Button $variant="secondary">Secondary</Button>
    </>
  );
}
```

### 状态持久化方案

```tsx
import { useState, useEffect } from 'react';

// 1. localStorage Hook
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue] as const;
}

// 2. IndexedDB Hook
function useIndexedDB<T>(dbName: string, storeName: string) {
  const [db, setDb] = useState<IDBDatabase | null>(null);
  
  useEffect(() => {
    const request = indexedDB.open(dbName, 1);
    
    request.onerror = () => console.error('IndexedDB error');
    
    request.onsuccess = () => {
      setDb(request.result);
    };
    
    request.onupgradeneeded = (event) => {
      const database = (event.target as IDBOpenDBRequest).result;
      database.createObjectStore(storeName, { keyPath: 'id' });
    };
  }, [dbName, storeName]);
  
  const add = async (item: T & { id: string }) => {
    if (!db) return;
    const tx = db.transaction(storeName, 'readwrite');
    const store = tx.objectStore(storeName);
    store.add(item);
  };
  
  const get = async (id: string): Promise<T | undefined> => {
    if (!db) return undefined;
    return new Promise((resolve, reject) => {
      const tx = db.transaction(storeName, 'readonly');
      const store = tx.objectStore(storeName);
      const request = store.get(id);
      request.onsuccess = () => resolve(request.result);
      request.onerror = reject;
    });
  };
  
  return { db, add, get };
}
```

### 性能监控与分析

```tsx
import { Profiler, ProfilerOnRenderCallback } from 'react';

function PerformanceMonitor({ children }: { children: ReactNode }) {
  const onRender: ProfilerOnRenderCallback = (
    id,
    phase,
    actualDuration,
    baseDuration,
    startTime,
    commitTime
  ) => {
    if (process.env.NODE_ENV !== 'production') {
      console.group(`Profiler: ${id}`);
      console.log(`Phase: ${phase}`);
      console.log(`Actual duration: ${actualDuration.toFixed(2)}ms`);
      console.log(`Base duration: ${baseDuration.toFixed(2)}ms`);
      console.log(`Start time: ${startTime.toFixed(2)}ms`);
      console.log(`Commit time: ${commitTime.toFixed(2)}ms`);
      console.groupEnd();
    }
    
    if (process.env.NODE_ENV === 'production') {
      reportWebVitals({
        name: id,
        value: actualDuration,
        delta: actualDuration - baseDuration,
      });
    }
  };
  
  return (
    <Profiler id="App" onRender={onRender}>
      {children}
    </Profiler>
  );
}

function reportWebVitals({ name, delta, id }: { 
  name: string; 
  delta: number; 
  id: string;
}) {
  if (name === 'layout-shift') {
    console.log('CLS:', delta);
  }
  
  if (name === 'largest-contentful-paint') {
    console.log('LCP:', delta);
  }
  
  if (name === 'first-input') {
    console.log('FID:', delta);
  }
  
  navigator.sendBeacon?.('/analytics', JSON.stringify({
    name,
    delta,
    id,
    timestamp: Date.now()
  }));
}
```

---

## 企业级架构模式

### 微前端架构 (Micro Frontends)

微前端将微服务的理念引入前端，实现大型应用的独立开发和部署：

```typescript
// 1. Module Federation 配置 (Webpack 5)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@https://remote.example.com/remoteEntry.js',
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

```tsx
// 2. 动态导入远程模块
const RemoteApp = React.lazy(() => import('remoteApp/CartModule'));

// 3. Host 应用中使用
function App() {
  return (
    <div>
      <Header />
      <React.Suspense fallback={<Loading />}>
        <RemoteApp />
      </React.Suspense>
      <Footer />
    </div>
  );
}
```

**微前端实现方案对比：**

| 方案 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|---------|------|------|----------|
| **Module Federation** | Webpack 共享 | 运行时集成，按需加载 | 依赖统一版本 | 大型团队协作 |
| **iframe** | HTML iframe | 隔离彻底，技术栈无关 | 通信困难 | 独立子应用 |
| **Web Components** | 自定义元素 | 框架无关，标准支持 | 样式隔离困难 | 跨框架复用 |
| **qiankun** | 基于 single-spa | 阿里方案，生态完善 | 配置复杂 | 国内项目 |

### Module Federation 完整示例

```typescript
// host/webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'development',
  devServer: {
    port: 3000,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remote: 'remote@http://localhost:3001/remoteEntry.js',
      },
      shared: ['react', 'react-dom'],
    }),
    new HtmlWebpackPlugin({
      template: './index.html',
    }),
  ],
};
```

```typescript
// remote/webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  mode: 'development',
  devServer: {
    port: 3001,
  },
  plugins: [
    new ModuleFederationPlugin({
      name: 'remote',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/Button',
        './Modal': './src/Modal',
      },
      shared: ['react', 'react-dom'],
    }),
    new HtmlWebpackPlugin({
      template: './bootstrap.js',
    }),
  ],
};
```

### 容器化组件设计模式 (Container/Presentational)

```tsx
// Presentational Component - 只负责 UI 渲染
interface UserCardProps {
  user: {
    id: string;
    name: string;
    email: string;
    avatar?: string;
  };
  onEdit: (id: string) => void;
  onDelete: (id: string) => void;
}

function UserCard({ user, onEdit, onDelete }: UserCardProps) {
  return (
    <div className="user-card" role="article" aria-label={`用户: ${user.name}`}>
      <img 
        src={user.avatar || '/default-avatar.png'} 
        alt={`${user.name}的头像`}
        className="user-avatar"
      />
      <div className="user-info">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
      </div>
      <div className="user-actions">
        <button onClick={() => onEdit(user.id)} aria-label={`编辑${user.name}`}>
          编辑
        </button>
        <button onClick={() => onDelete(user.id)} aria-label={`删除${user.name}`}>
          删除
        </button>
      </div>
    </div>
  );
}

// Container Component - 负责数据获取和业务逻辑
function UserCardContainer({ userId }: { userId: string }) {
  const { data: user, loading, error, refetch } = useUser(userId);
  
  const handleEdit = async (id: string) => {
    await updateUser(id, { /* 更新数据 */ });
    refetch();
  };
  
  const handleDelete = async (id: string) => {
    if (confirm('确定要删除这个用户吗？')) {
      await deleteUser(id);
    }
  };

  if (loading) return <UserCardSkeleton />;
  if (error) return <ErrorMessage error={error} onRetry={refetch} />;
  
  return (
    <UserCard 
      user={user} 
      onEdit={handleEdit} 
      onDelete={handleDelete} 
    />
  );
}
```

### Compound Components 模式

```tsx
// 复合组件 - 提供隐式状态共享
interface TabsContextValue {
  activeTab: string;
  setActiveTab: (id: string) => void;
}

const TabsContext = createContext<TabsContextValue | null>(null);

function useTabsContext() {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tab components must be used within Tabs');
  }
  return context;
}

// Tab 组件族
function Tabs({ defaultValue, children }: { defaultValue: string; children: ReactNode }) {
  const [activeTab, setActiveTab] = useState(defaultValue);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

Tabs.List = function TabList({ children }: { children: ReactNode }) {
  return <div className="tab-list" role="tablist">{children}</div>;
};

Tabs.Tab = function Tab({ value, children }: { value: string; children: ReactNode }) {
  const { activeTab, setActiveTab } = useTabsContext();
  const isActive = activeTab === value;
  
  return (
    <button
      role="tab"
      aria-selected={isActive}
      onClick={() => setActiveTab(value)}
    >
      {children}
    </button>
  );
};

Tabs.Panel = function TabPanel({ value, children }: { value: string; children: ReactNode }) {
  const { activeTab } = useTabsContext();
  
  if (activeTab !== value) return null;
  
  return (
    <div role="tabpanel" aria-labelledby={`tab-${value}`}>
      {children}
    </div>
  );
};

// 使用
function App() {
  return (
    <Tabs defaultValue="profile">
      <Tabs.List>
        <Tabs.Tab value="profile">个人资料</Tabs.Tab>
        <Tabs.Tab value="settings">设置</Tabs.Tab>
        <Tabs.Tab value="security">安全</Tabs.Tab>
      </Tabs.List>
      <Tabs.Panel value="profile">个人资料内容...</Tabs.Panel>
      <Tabs.Panel value="settings">设置内容...</Tabs.Panel>
      <Tabs.Panel value="security">安全内容...</Tabs.Panel>
    </Tabs>
  );
}
```

### Provider Pattern 深度应用

```tsx
// 1. 链式 Provider
function AppProviders({ children }: { children: ReactNode }) {
  return (
    <ErrorBoundary>
      <AuthProvider>
        <ThemeProvider>
          <ToastProvider>
            <NetworkProvider>
              {children}
            </NetworkProvider>
          </ToastProvider>
        </ThemeProvider>
      </AuthProvider>
    </ErrorBoundary>
  );
}

// 2. Provider 合并优化 (避免嵌套地狱)
function ComposeProviders({ providers, children }: { 
  providers: Array<React.ComponentType<{ children: ReactNode }>>;
  children: ReactNode;
}) {
  return providers.reduceRight((acc, Provider) => (
    <Provider>{acc}</Provider>
  ), children);
}

// 使用
function App() {
  return (
    <ComposeProviders providers={[ErrorBoundary, AuthProvider, ThemeProvider, ToastProvider]}>
      <Router />
    </ComposeProviders>
  );
}

// 3. 动态 Provider 注入
function DynamicProvider({ 
  providers, 
  children 
}: { 
  providers: Array<React.ComponentType<{ children: ReactNode }>>;
  children: ReactNode;
}) {
  const [Provider, ...restProviders] = providers;
  
  if (!Provider) return <>{children}</>;
  
  return (
    <Provider>
      <DynamicProvider providers={restProviders}>
        {children}
      </DynamicProvider>
    </Provider>
  );
}
```

---

## CI/CD 部署与自动化

### GitHub Actions 完整配置

```yaml
# .github/workflows/ci.yml
name: React CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'

jobs:
  # 代码质量检查
  lint:
    name: ESLint & Type Check
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Type check
        run: npm run type-check

  # 单元测试
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
        env:
          CI: true
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # E2E 测试
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # 构建和部署
  build-and-deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    needs: e2e
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://myapp.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
          VITE_SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
      
      - name: Run PageSpeed Insights
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: https://myapp.example.com
          budgetPath: ./lighthouse-budget.json
      
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
      
      - name: Notify Slack
        if: always()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "Deployment ${{ job.status }}",
              "blocks": [{
                "type": "section",
                "text": {
                  "type": "mrkdwn",
                  "text": "*React App Deployment*\nStatus: ${{ job.status }}"
                }
              }]
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

### Vercel 部署配置

```typescript
// vercel.json
{
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "framework": "vite",
  "headers": [
    {
      "source": "/sw.js",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=0, must-revalidate" },
        { "key": "Service-Worker-Allowed", "value": "/" }
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
  ],
  "rewrites": [
    { "source": "/api/(.*)", "destination": "/api/$1" }
  ],
  "regions": ["hkg1", "sin1"],
  "env": {
    "VITE_API_URL": "@api-url",
    "VITE_SENTRY_DSN": "@sentry-dsn"
  }
}
```

### Docker 容器化部署

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

# 构建阶段
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产镜像
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production

COPY --from=builder /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./

# 非 root 用户运行
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 appuser
USER appuser

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:pass@db:5432/app
      - REDIS_URL=redis://cache:6379
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: app
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d app"]
      interval: 10s
      timeout: 5s
      retries: 5

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

volumes:
  postgres_data:
  redis_data:
```

---

## 安全最佳实践

### XSS 防护

```tsx
// 1. React 自动转义
function SafeComponent() {
  // React 会自动转义这些内容
  const userInput = '<script>alert("xss")</script>';
  return <div>{userInput}</div>; // 安全输出
}

// 2. 危险操作：直接插入 HTML（避免使用）
function DangerousComponent() {
  const html = '<strong>粗体</strong>';
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// 3. 安全替代：DOMPurify
import DOMPurify from 'dompurify';

function SanitizedComponent() {
  const dirty = '<script>alert("xss")</script><strong>粗体</strong>';
  const clean = DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'strong', 'i', 'em', 'u'],
    ALLOWED_ATTR: [],
  });
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}

// 4. 富文本编辑器安全集成
import { useLexicalComposerContext } from '@lexical/react/LexicalComposer';
import { $getSelection, $isRangeSelection, FORMAT_TEXT_COMMAND } from 'lexical';

function sanitizeSelection(editor: LexicalComposerContext) {
  editor.update(() => {
    const selection = $getSelection();
    if ($isRangeSelection(selection)) {
      const nodes = selection.extract();
      // 只保留安全节点
      const safeNodes = nodes.filter(node => {
        const text = node.getTextContent();
        return !/<script|javascript:|on\w+=/i.test(text);
      });
      // 替换选中内容
      selection.insertNodes(safeNodes);
    }
  });
}
```

### CSRF 防护

```typescript
// 1. CSRF Token 机制
import { useMutation, useQuery } from '@tanstack/react-query';

function useCsrfToken() {
  return useQuery({
    queryKey: ['csrf-token'],
    queryFn: async () => {
      const response = await fetch('/api/csrf-token');
      return response.json();
    },
    staleTime: Infinity, // Token 应该被缓存
  });
}

function useSecureMutation<T>(mutationFn: (data: T, csrfToken: string) => Promise<void>) {
  const { data: csrfToken } = useCsrfToken();
  
  return useMutation({
    mutationFn: (data: T) => mutationFn(data, csrfToken?.token || ''),
  });
}

// 2. API 客户端自动处理
const apiClient = axios.create({
  baseURL: '/api',
  withCredentials: true, // 发送 cookies
});

// 请求拦截器添加 CSRF token
apiClient.interceptors.request.use(async (config) => {
  if (['post', 'put', 'patch', 'delete'].includes(config.method || '')) {
    const { data } = await axios.get('/api/csrf-token');
    config.headers['X-CSRF-Token'] = data.token;
  }
  return config;
});

// 3. Double Submit Cookie Pattern
function generateCsrfToken(): string {
  return crypto.randomBytes(32).toString('hex');
}

async function validateCsrfToken(token: string, cookieToken: string): Promise<boolean> {
  return token === cookieToken && token.length > 0;
}
```

### 内容安全策略 (CSP)

```typescript
// vite.config.ts - Vite 配置 CSP
import { defineConfig } from 'vite';
import cspHtmlPlugin from 'vite-plugin-csp-html';

export default defineConfig({
  plugins: [
    cspHtmlPlugin({
      policy: {
        'default-src': ["'self'"],
        'script-src': ["'self'", "'nonce-{NONCE}'", 'strict-dynamic'],
        'style-src': ["'self'", "'unsafe-inline'"],
        'img-src': ["'self'", 'data:', 'https:'],
        'font-src': ["'self'", 'https:', 'data:'],
        'connect-src': ["'self'", 'https://api.example.com'],
        'frame-ancestors': ["'none'"],
        'form-action': ["'self'"],
        'base-uri': ["'self'"],
        'object-src': ["'none'"],
      },
      reportUri: '/api/csp-violation-report',
    }),
  ],
});
```

```typescript
// Express 中间件处理 CSP 报告
import express from 'express';

const app = express();

// CSP 报告端点
app.post('/api/csp-violation-report', express.json(), (req, res) => {
  const report = req.body['csp-report'];
  console.error('CSP Violation:', report);
  
  // 发送到监控系统
  monitoringService.reportCSPViolation(report);
  
  res.status(204).send();
});

// 响应头设置
app.use((req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.nonce = nonce;
  
  res.setHeader(
    'Content-Security-Policy',
    [
      "default-src 'self'",
      `script-src 'self' 'nonce-${nonce}' 'strict-dynamic'`,
      "style-src 'self' 'unsafe-inline'",
      "img-src 'self' data: https:",
      "font-src 'self' https: data:",
      "connect-src 'self' https://api.example.com",
      "frame-ancestors 'none'",
      "object-src 'none'",
    ].join('; ')
  );
  
  next();
});
```

### 敏感信息管理

```typescript
// 1. 环境变量类型安全
interface EnvironmentVariables {
  VITE_API_URL: string;
  VITE_SENTRY_DSN: string;
  VITE_STRIPE_PUBLIC_KEY: string;
  
  // 服务端变量（不暴露到客户端）
  readonly DATABASE_URL: string;
  readonly JWT_SECRET: string;
  readonly ADMIN_EMAIL: string;
}

function getEnv(): EnvironmentVariables {
  const env = import.meta.env;
  
  const requiredVars = ['VITE_API_URL'] as const;
  for (const key of requiredVars) {
    if (!env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
  
  return env as unknown as EnvironmentVariables;
}

// 2. 密钥轮换策略
const encryption = {
  async rotateKey(oldKey: string, data: string): Promise<string> {
    const keyId = await getCurrentKeyId();
    const newKey = await generateNewKey();
    
    // 重新加密数据
    const decrypted = await decrypt(data, oldKey);
    const reEncrypted = await encrypt(decrypted, newKey);
    
    // 保存新密钥（带密钥 ID）
    await storeKey(newKey, { id: keyId + 1 });
    
    return `${keyId + 1}:${reEncrypted}`;
  },
};
```

### 安全 Headers 中间件

```typescript
// middleware/securityHeaders.ts
import { Request, Response, NextFunction } from 'express';

export function securityHeaders(req: Request, res: Response, next: NextFunction) {
  // 防止点击劫持
  res.setHeader('X-Frame-Options', 'DENY');
  
  // 防止 MIME 类型嗅探
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // XSS 防护
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // 引用来源策略
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // 权限策略
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(self), geolocation=(), payment=()'
  );
  
  // HSTS (生产环境)
  if (process.env.NODE_ENV === 'production') {
    res.setHeader(
      'Strict-Transport-Security',
      'max-age=31536000; includeSubDomains; preload'
    );
  }
  
  next();
}
```

---

## 可访问性 (A11y) 最佳实践

### ARIA 属性完整指南

```tsx
// 1. 按钮 vs 链接
function AccessibleButtons() {
  return (
    <div>
      {/* 按钮 - 执行操作 */}
      <button onClick={handleSave} aria-describedby="save-description">
        保存
      </button>
      <p id="save-description">保存当前编辑的内容到服务器</p>
      
      {/* 链接 - 导航到新页面 */}
      <a href="/dashboard" aria-current="page">
        前往仪表盘
      </a>
    </div>
  );
}

// 2. 表单错误提示
function AccessibleForm() {
  const [errors, setErrors] = useState<Record<string, string>>({});
  const [touched, setTouched] = useState<Record<string, boolean>>({});
  
  return (
    <form>
      <div>
        <label htmlFor="email">邮箱地址</label>
        <input
          id="email"
          type="email"
          aria-invalid={!!errors.email}
          aria-describedby={errors.email ? 'email-error' : undefined}
          onBlur={() => setTouched(t => ({ ...t, email: true }))}
        />
        {errors.email && touched.email && (
          <p id="email-error" role="alert">
            {errors.email}
          </p>
        )}
      </div>
    </form>
  );
}

// 3. 模态对话框
function AccessibleModal({ isOpen, onClose, title, children }) {
  const modalRef = useRef(null);
  const previousFocus = useRef(null);
  
  useEffect(() => {
    if (isOpen) {
      previousFocus.current = document.activeElement;
      modalRef.current?.focus();
      
      // 陷阱焦点
      const handleKeyDown = (e: KeyboardEvent) => {
        if (e.key === 'Escape') {
          onClose();
        }
        if (e.key === 'Tab') {
          const focusable = modalRef.current?.querySelectorAll(
            'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
          );
          if (!focusable?.length) return;
          
          const first = focusable[0];
          const last = focusable[focusable.length - 1];
          
          if (e.shiftKey && document.activeElement === first) {
            e.preventDefault();
            last.focus();
          } else if (!e.shiftKey && document.activeElement === last) {
            e.preventDefault();
            first.focus();
          }
        }
      };
      
      document.addEventListener('keydown', handleKeyDown);
      return () => {
        document.removeEventListener('keydown', handleKeyDown);
        previousFocus.current?.focus();
      };
    }
  }, [isOpen, onClose]);
  
  if (!isOpen) return null;
  
  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      tabIndex={-1}
      className="modal"
    >
      <h2 id="modal-title">{title}</h2>
      <button onClick={onClose} aria-label="关闭对话框">
        ×
      </button>
      {children}
    </div>
  );
}

// 4. 实时区域 (Live Regions)
function AccessibleNotifications() {
  const [notification, setNotification] = useState('');
  
  return (
    <>
      <div aria-live="polite" aria-atomic="true" className="sr-only">
        {notification}
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
          <th scope="col" rowSpan={2}>产品</th>
          <th scope="colgroup" colSpan={4}>季度</th>
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
// 1. Roving Tabindex
function AccessibleMenu() {
  const [activeIndex, setActiveIndex] = useState(0);
  const menuItems = ['首页', '关于', '产品', '联系'];
  
  const handleKeyDown = (e: React.KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
      case 'ArrowRight':
        e.preventDefault();
        setActiveIndex((prev) => (prev + 1) % menuItems.length);
        break;
      case 'ArrowUp':
      case 'ArrowLeft':
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
        handleSelect(menuItems[activeIndex]);
        break;
    }
  };
  
  return (
    <nav role="menubar" aria-label="主菜单">
      {menuItems.map((item, index) => (
        <button
          key={item}
          role="menuitem"
          tabIndex={index === activeIndex ? 0 : -1}
          aria-haspopup={item === '产品'}
          onKeyDown={handleKeyDown}
          onClick={() => handleSelect(item)}
        >
          {item}
        </button>
      ))}
    </nav>
  );
}

// 2. 焦点管理 Hook
function useFocusManagement() {
  const [focusedId, setFocusedId] = useState<string | null>(null);
  
  const moveFocus = (containerId: string, direction: 'up' | 'down' | 'left' | 'right') => {
    const container = document.getElementById(containerId);
    const focusable = container?.querySelectorAll(
      'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])'
    );
    
    if (!focusable?.length) return;
    
    const currentIndex = Array.from(focusable).findIndex(
      (el) => el === document.activeElement
    );
    
    let nextIndex: number;
    switch (direction) {
      case 'up':
        nextIndex = currentIndex > 0 ? currentIndex - 1 : focusable.length - 1;
        break;
      case 'down':
        nextIndex = currentIndex < focusable.length - 1 ? currentIndex + 1 : 0;
        break;
      default:
        nextIndex = currentIndex;
    }
    
    (focusable[nextIndex] as HTMLElement).focus();
  };
  
  return { focusedId, setFocusedId, moveFocus };
}
```

### 颜色对比度和视觉辅助

```css
/* WCAG AA 标准对比度检查 */
/* 正常文本: 4.5:1 */
/* 大文本 (18px+): 3:1 */
/* UI 组件和图形: 3:1 */

/* 使用 CSS 自定义属性管理颜色 */
:root {
  /* 主色调 */
  --color-primary: #1a73e8;
  --color-primary-dark: #1557b0;
  
  /* 文本颜色 - 确保对比度 */
  --text-primary: #202124; /* 对比度 ~15:1 on white */
  --text-secondary: #5f6368; /* 对比度 ~7:1 on white */
  --text-hint: #9aa0a6; /* 对比度 ~4.5:1 on white */
  
  /* 背景 */
  --background-light: #ffffff;
  --background-dark: #1f1f1f;
}

/* Focus 可见性 */
*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* 隐藏但保持可访问 */
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

/* 高对比度模式支持 */
@media (forced-colors: active) {
  .button {
    border: 2px solid currentColor;
  }
  
  .button:focus {
    outline: 3px solid highlight;
  }
}
```

---

## 环境配置与环境变量管理

### 多种环境配置

```typescript
// .env 文件层级
// .env                 - 默认值（提交到版本控制）
// .env.local           - 本地覆盖（不提交）
// .env.[mode]          - 环境特定值
// .env.[mode].local     - 环境本地覆盖

// .env.development
VITE_API_URL=http://localhost:3000
VITE_MOCK_ENABLED=true
VITE_LOG_LEVEL=debug

// .env.production
VITE_API_URL=https://api.production.com
VITE_MOCK_ENABLED=false
VITE_LOG_LEVEL=error

// .env.test
VITE_API_URL=http://localhost:8080
VITE_MOCK_ENABLED=true
VITE_LOG_LEVEL=warn
```

```typescript
// src/config/environment.ts
interface AppConfig {
  apiUrl: string;
  sentryDsn: string | null;
  featureFlags: FeatureFlags;
  analyticsId: string | null;
}

interface FeatureFlags {
  enableNewDashboard: boolean;
  enableDarkMode: boolean;
  maxUploadSize: number;
}

export function getEnvironmentConfig(): AppConfig {
  const env = import.meta.env;
  
  return {
    apiUrl: env.VITE_API_URL || 'http://localhost:3000',
    sentryDsn: env.VITE_SENTRY_DSN || null,
    analyticsId: env.VITE_ANALYTICS_ID || null,
    featureFlags: {
      enableNewDashboard: env.VITE_FLAG_NEW_DASHBOARD === 'true',
      enableDarkMode: env.VITE_FLAG_DARK_MODE === 'true',
      maxUploadSize: parseInt(env.VITE_MAX_UPLOAD_SIZE || '10485760', 10), // 10MB
    },
  };
}

// 使用
const config = getEnvironmentConfig();
console.log(config.featureFlags.enableNewDashboard);
```

```typescript
// vite.config.ts - 环境变量验证
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');
  
  const required = ['VITE_API_URL'];
  const missing = required.filter((key) => !env[key]);
  
  if (missing.length > 0 && mode === 'production') {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
  
  return {
    define: {
      __APP_ENV__: JSON.stringify(env.APP_ENV),
    },
  };
});
```

---

## 企业级项目结构模板

### 小型项目结构

适用于简单应用和快速原型：

```
src/
├── assets/              # 静态资源
│   ├── images/
│   └── fonts/
├── components/          # 通用组件
│   ├── Button.tsx
│   ├── Card.tsx
│   └── Modal.tsx
├── pages/              # 页面组件
│   ├── Home.tsx
│   ├── About.tsx
│   └── NotFound.tsx
├── hooks/              # 自定义 Hooks
│   ├── useAuth.ts
│   └── useLocalStorage.ts
├── services/           # API 服务
│   └── api.ts
├── types/              # 类型定义
│   └── index.ts
├── utils/              # 工具函数
│   └── format.ts
├── App.tsx             # 根组件
├── main.tsx            # 入口文件
└── vite-env.d.ts       # Vite 类型声明
```

### 中型项目结构

适用于团队协作的中等规模应用：

```
src/
├── app/                # 应用核心
│   ├── App.tsx
│   ├── main.tsx
│   └── routes.tsx
├── assets/             # 静态资源
│   ├── images/
│   ├── icons/
│   └── styles/
├── components/         # 通用组件
│   ├── ui/           # 基础 UI 组件
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.module.css
│   │   │   └── index.ts
│   │   ├── Input/
│   │   └── Modal/
│   ├── layout/        # 布局组件
│   │   ├── Header/
│   │   ├── Footer/
│   │   ├── Sidebar/
│   │   └── Container/
│   └── common/        # 公共组件
├── features/           # 业务功能模块
│   ├── auth/          # 认证模块
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── UserProfile.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   └── index.ts
│   ├── dashboard/     # 仪表盘模块
│   │   ├── components/
│   │   ├── hooks/
│   │   └── pages/
│   └── settings/       # 设置模块
├── hooks/              # 全局自定义 Hooks
│   ├── useFetch.ts
│   ├── useLocalStorage.ts
│   └── useDebounce.ts
├── contexts/           # React Context
│   ├── AuthContext.tsx
│   ├── ThemeContext.tsx
│   └── NotificationContext.tsx
├── services/           # API 服务层
│   ├── apiClient.ts    # Axios 实例配置
│   ├── userService.ts
│   └── productService.ts
├── utils/              # 工具函数
│   ├── format/
│   │   ├── formatDate.ts
│   │   └── formatCurrency.ts
│   ├── validation/
│   │   ├── email.ts
│   │   └── password.ts
│   └── helpers/
│       └── debounce.ts
├── types/              # 全局类型定义
│   ├── index.ts
│   ├── api.d.ts
│   └── user.d.ts
├── constants/           # 常量定义
│   ├── api.ts
│   └── config.ts
├── pages/              # 页面组件
│   ├── HomePage.tsx
│   ├── AboutPage.tsx
│   ├── DashboardPage.tsx
│   └── NotFoundPage.tsx
├── config/             # 配置文件
│   ├── env.ts
│   └── features.ts
└── __tests__/          # 测试文件
    ├── components/
    └── hooks/
```

### 大型企业项目结构

适用于大型团队和复杂业务系统：

```
src/
├── apps/                # 微前端应用入口
│   ├── admin/          # 管理后台应用
│   │   ├── src/
│   │   ├── pages/
│   │   ├── components/
│   │   └── main.tsx
│   └── portal/         # 用户门户应用
│       └── ...
├── packages/            # 共享包（Monorepo）
│   ├── ui/             # 设计系统
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Button/
│   │   │   │   │   ├── Button.tsx
│   │   │   │   │   ├── Button.module.css
│   │   │   │   │   ├── Button.test.tsx
│   │   │   │   │   ├── Button.stories.tsx
│   │   │   │   │   └── index.ts
│   │   │   │   └── ...
│   │   │   ├── theme/
│   │   │   ├── hooks/
│   │   │   └── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── utils/          # 共享工具
│   └── types/          # 共享类型
├── src/
│   ├── main.tsx         # 应用入口
│   ├── App.tsx         # 根组件
│   ├── app/            # 应用配置
│   │   ├── routes/     # 路由配置
│   │   │   ├── index.ts
│   │   │   ├── routes.ts
│   │   │   └── guards.ts
│   │   ├── providers/  # Provider 配置
│   │   │   ├── index.tsx
│   │   │   ├── AuthProvider.tsx
│   │   │   └── ThemeProvider.tsx
│   │   └── contexts/  # Context 定义
│   ├── components/     # 组件库
│   │   ├── ui/         # 基础 UI 组件
│   │   ├── layout/     # 布局组件
│   │   └── common/     # 公共组件
│   ├── features/       # 功能模块
│   │   └── [feature]/
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── services/
│   │       ├── types/
│   │       ├── utils/
│   │       ├── constants/
│   │       ├── [feature].api.ts
│   │       ├── [feature].types.ts
│   │       ├── [feature].module.css
│   │       └── index.ts
│   ├── pages/          # 页面组件
│   │   └── [page]/
│   │       ├── [PageName]Page.tsx
│   │       ├── [PageName].module.css
│   │       └── index.ts
│   ├── services/       # 服务层
│   │   ├── api/        # API 客户端
│   │   │   ├── index.ts
│   │   │   ├── client.ts
│   │   │   ├── interceptors.ts
│   │   │   └── errorHandler.ts
│   │   └── modules/    # 按模块组织
│   │       ├── auth.service.ts
│   │       └── user.service.ts
│   ├── hooks/          # 全局 Hooks
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   ├── useDebounce.ts
│   │   └── useMediaQuery.ts
│   ├── utils/          # 工具函数
│   │   ├── format/
│   │   ├── validation/
│   │   ├── crypto/
│   │   └── helpers/
│   ├── types/          # 全局类型
│   │   ├── api.ts
│   │   ├── user.ts
│   │   └── common.ts
│   ├── constants/      # 常量
│   │   ├── api.ts
│   │   ├── routes.ts
│   │   └── config.ts
│   ├── config/         # 配置
│   │   ├── index.ts
│   │   ├── env.ts
│   │   └── features.ts
│   └── styles/         # 全局样式
│       ├── variables.css
│       ├── reset.css
│       └── global.css
├── public/             # 静态资源
│   ├── favicon.ico
│   ├── manifest.json
│   └── robots.txt
├── tests/              # 测试配置
│   ├── setup.ts
│   ├── mocks/
│   └── e2e/
├── scripts/           # 构建脚本
│   ├── analyze.ts
│   └── generate-types.ts
├── docs/              # 项目文档
├── .storybook/        # Storybook 配置
├── .eslintrc.js       # ESLint 配置
├── .prettierrc        # Prettier 配置
├── tsconfig.json      # TypeScript 配置
├── tsconfig.node.json # Node TypeScript 配置
├── vite.config.ts     # Vite 配置
├── package.json
└── .env.example      # 环境变量示例
```

### Feature-Sirst 项目结构

现代 React 项目推荐的组织方式：

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm/
│   │   │   │   ├── LoginForm.tsx
│   │   │   │   ├── LoginForm.module.css
│   │   │   │   ├── LoginForm.test.tsx
│   │   │   │   └── index.ts
│   │   │   ├── RegisterForm/
│   │   │   └── UserMenu/
│   │   ├── hooks/
│   │   │   ├── useLogin.ts
│   │   │   ├── useLogout.ts
│   │   │   └── useAuth.ts
│   │   ├── api/
│   │   │   ├── auth.api.ts
│   │   │   └── auth.types.ts
│   │   ├── utils/
│   │   │   └── auth.utils.ts
│   │   ├── constants/
│   │   │   └── auth.constants.ts
│   │   ├── auth.module.css
│   │   └── index.ts
│   │
│   ├── products/
│   │   ├── components/
│   │   │   ├── ProductCard/
│   │   │   ├── ProductList/
│   │   │   └── ProductDetail/
│   │   ├── hooks/
│   │   │   ├── useProducts.ts
│   │   │   └── useProductDetail.ts
│   │   ├── api/
│   │   │   ├── products.api.ts
│   │   │   └── products.types.ts
│   │   └── index.ts
│   │
│   └── cart/
│       ├── components/
│       ├── hooks/
│       ├── api/
│       └── index.ts
│
├── components/         # 共享组件
│   ├── ui/            # UI 基础组件
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── ...
│   └── layout/        # 布局组件
│       ├── Header/
│       ├── Footer/
│       └── Sidebar/
│
├── lib/                # 库代码
│   ├── api/           # API 客户端
│   │   ├── client.ts
│   │   └── errors.ts
│   ├── auth/          # 认证工具
│   └── utils/         # 通用工具
│
├── hooks/              # 全局 Hooks
│   ├── useFetch.ts
│   └── useLocalStorage.ts
│
├── pages/              # 页面入口
│   ├── index.tsx
│   ├── about.tsx
│   └── [...404].tsx
│
├── styles/             # 样式
│   ├── globals.css
│   └── variables.css
│
├── App.tsx
└── main.tsx
```

### 命名规范与最佳实践

**组件文件命名：**
```typescript
// ✅ 推荐：帕斯卡命名
UserCard.tsx
ProductList.tsx
DashboardLayout.tsx

// ✅ 子组件使用父组件前缀
UserCardHeader.tsx
UserCardBody.tsx
UserCardFooter.tsx

// ❌ 避免：混合命名
userCard.tsx      // 应该用 PascalCase
ProductListView.tsx  // 不必要的 View 后缀
```

**目录命名：**
```typescript
// ✅ 推荐：kebab-case
components/
feature-components/
ui-components/

// ❌ 避免：驼峰或 PascalCase
Components/      // 不要用 PascalCase
featureComponents/  // 不要用 camelCase
```

**测试文件组织：**
```
components/
├── Button/
│   ├── Button.tsx
│   ├── Button.test.tsx         // 单元测试
│   ├── Button.stories.tsx      // Storybook
│   └── index.ts

features/
├── auth/
│   ├── __tests__/
│   │   ├── LoginForm.test.tsx
│   │   ├── useAuth.test.ts
│   │   └── auth.api.test.ts
│   └── __mocks__/
│       └── auth.service.ts
```

**Barrel 文件（index.ts）模式：**
```typescript
// components/Button/index.ts
export { Button } from './Button';
export type { ButtonProps, ButtonVariant } from './Button.types';

// features/auth/index.ts
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/useAuth';
export { useLogin } from './hooks/useLogin';
export * from './api/auth.types';
```

---

> [!SUCCESS]
> 本文档涵盖了 React 19 的核心理念、安装配置、组件系统、状态管理、性能优化、高级模式、动画处理、SSR进阶、国际化、测试等全方位内容。通过系统学习这些知识，你将具备构建现代 React 应用的能力。持续实践和阅读源码是提升 React 技能的最佳途径。
