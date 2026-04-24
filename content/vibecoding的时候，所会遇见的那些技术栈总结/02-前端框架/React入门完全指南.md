---
title: React入门完全指南
date: 2026-04-24
tags:
  - React
  - 前端开发
  - JavaScript框架
  - Hooks
  - JSX
categories:
  - 前端框架
---

> [!NOTE]
> 本文档面向 React 初学者，通过大量实例帮助你从零开始掌握 React 的核心概念和基础技能。建议按照章节顺序学习，每章都配有实际可运行的代码示例。

---

## React 概述与版本演进

### 什么是 React

React 是 Facebook（现 Meta）于 2013 年开源的 JavaScript 库，专门用于构建用户界面。想象一下，你正在用乐高积木搭建一个城堡——React 就是那个帮你快速拼装、管理这些积木的工具，让你不用一块一块地手动摆放，而是描述你想要什么样的城堡，React 就会帮你自动完成搭建工作。

React 的核心思想是**声明式编程**和**组件化开发**。声明式编程意味着你只需要告诉 React"我想要什么"，而不是"怎么做"。比如，你告诉 React"这个按钮应该显示为蓝色"，React 就会负责找出最快的方式把按钮变成蓝色，而不需要你手动操作 DOM 元素。组件化开发则让你可以把复杂的界面拆分成独立的、可复用的小部件，每个部件都有自己的逻辑和外观。

React 不是像 Angular 那样的大而全的框架，而是一个专注于视图层（View）的库。这就像是买了一辆车，Angular 是一辆整车，React 则像是提供了一个优秀的发动机——你可以把它装进各种不同的车壳里，配合各种不同的轮子（状态管理）和传动系统（路由）。这种灵活性让 React 成为最受欢迎的前端技术之一。

### 版本演进：从出生到成熟

React 的发展历程就像一个人的成长过程，每个版本都在解决之前的问题并引入新特性：

**React 0.3（2013年）**：Facebook 首次开源 React，那时候的 React 还很稚嫩，但已经展现出了组件化和 Virtual DOM 的理念。

**React 0.14（2015年）**：开始支持组件函数化，props children 的概念变得更加清晰。这一年 Vue.js 也发布了，两个框架开始在社区中形成竞争。

**React 16.0（2017年）**：这是 React 历史上最重要的更新之一——Fiber 架构。Fiber 让 React 可以把渲染工作分解成小任务，利用浏览器空闲时间来执行，解决了长期困扰 React 应用的卡顿问题。Error Boundaries（错误边界）也是在这一年加入的。

**React 16.8（2019年）**：Hooks 的引入彻底改变了 React 的写法。在 Hooks 之前，如果你想用类组件的某些特性（比如生命周期、状态），你必须把函数组件改写成类组件。Hooks 让你在函数组件中也能使用这些特性，代码变得更加简洁和易于复用。

**React 17（2020年）**：这是一个过渡版本，主要解决的是渐进式升级的问题。新 JSX Transform 让你不再需要手动引入 React。

**React 18（2022年）**：并发渲染（Concurrent Features）让 React 可以同时准备多个版本的 UI，Suspense 得到了更广泛的支持。这就像是给你的电脑装了一个智能任务管理器，可以更合理地分配计算资源。

**React 19（2024年）**：Server Components 允许组件在服务器端渲染，Actions 让表单处理变得更加简单，React Compiler 可以自动优化你的代码。这些新特性让 React 在全栈开发的道路上越走越远。

### React 的核心特点

React 有五个核心特点，理解它们能帮助你更好地把握 React 的设计理念：

**声明式**：想象你在点外卖，你只需要说"我要一份宫保鸡丁盖饭"，而不需要告诉厨师怎么切鸡丁、怎么炒。React 也是这样，你描述 UI 应该呈现什么状态，React 自动处理所有的更新细节。

**组件化**：把一个按钮做成组件，它就可以在无数个地方被使用。按钮的样式、点击逻辑都封装在组件内部，你不需要重复编写相同的代码。

**单向数据流**：数据像水流一样从山顶（父组件）流向山谷（子组件）。如果山谷的水位变了，它不会反过来影响山顶。这种单向性让数据流动变得可预测，调试也更容易。

**Virtual DOM**：这是 React 高性能的秘诀。当状态改变时，React 不是直接操作真实的网页 DOM，而是先在内存中创建一个虚拟的 DOM 树，比较新旧两棵树的差异，然后把最小的变更应用到真实 DOM 上。这就像是你装修房子之前，先在电脑上做一个 3D 模型，看看效果满意了再动手。

**生态丰富**：React 拥有世界上最大的前端社区之一，从 UI 组件库（Material-UI、Ant Design）到状态管理（Redux、Zustand）到构建工具（Next.js、Gatsby），几乎任何需求都能找到现成的解决方案。

---

## Virtual DOM 原理

### 真实 DOM vs 虚拟 DOM

要理解 Virtual DOM，首先需要知道什么是 DOM。DOM（Document Object Model，文档对象模型）是浏览器用来表示网页结构的一种方式。当你打开一个网页，浏览器会把 HTML 解析成一棵树，这棵树就是 DOM 树。每一 个 HTML 标签、每一段文字、每一个属性都是这棵树上的一个节点。

直接操作 DOM 是很慢的。想象你在一本很厚的书里修改一个字，你不能只改一个字——你需要把那一页撕下来，重新排版，可能还要调整后面所有页面的位置。浏览器操作 DOM 也是这样，每一次 DOM 操作都可能触发页面的重新布局和重绘。

Virtual DOM 就是为了解决这个问题而产生的。它是真实 DOM 的 JavaScript 对象表示。你可以把 Virtual DOM 想象成真实 DOM 的"蓝图"或"草稿"。当你修改了组件的状态，React 会先在内存中创建一棵新的 Virtual DOM 树，然后把这棵树和之前的树进行比较（这个过程叫做 Diffing），找出哪些地方发生了变化，最后只把变化的部分应用到真实 DOM 上。

### 工作原理详解

整个过程可以分成四个阶段来理解：

**用户交互**：当用户点击按钮、输入文字、或者网络请求返回数据时，会触发组件状态的改变。这就像是你告诉设计师"我想要把墙从白色改成蓝色"。

**状态更新**：组件的状态（state）或属性（props）发生变化。这相当于设计师在图纸上标注"墙面颜色：蓝色"。

**Render 阶段**：React 创建一棵新的 Virtual DOM 树。这个阶段只是生成 JavaScript 对象，不涉及任何真实的 DOM 操作。设计师重新画了一张完整的设计图。

**Reconciliation（调和）**：这是最关键的一步。React 比较新旧两棵 Virtual DOM 树，找出差异。这个比较过程遵循一些优化策略：

同级对比策略让 React 只比较同一层级的节点。如果一个节点从父元素 A 移动到了父元素 B，React 会认为这是两个完全不同的节点，把旧的删掉、创建新的。列表渲染时使用 key 属性可以帮助 React 准确识别每个节点，就像给每个学生编学号一样。

**Commit 阶段**：把差异应用到真实 DOM 上。因为可能同时有多个变化，React 会批量处理这些 DOM 操作，确保每个变化都高效完成。

### Fiber 架构：React 16 的革命性改变

React 16 引入的 Fiber 架构是对协调引擎的重写，它的目的是让 React 可以中断和恢复工作。想象你在餐厅吃饭，传统的方式是厨师必须做完一道菜才能做下一道；Fiber 架构让厨师可以暂停做某道菜，先去做其他紧急的菜，然后再回来继续。

Fiber 的核心特性包括：可中断渲染让高优先级更新可以打断低优先级更新，比如用户输入应该比数据列表渲染更优先；时间切片把渲染工作分散到多帧执行，避免长时间阻塞主线程；Suspense 让异步组件有了统一的 loading 状态处理方式；Concurrent Mode 则是以上所有特性的基础，让 React 可以同时准备多个版本的 UI。

---

## 环境准备与项目创建

### 开发环境要求

在开始创建 React 项目之前，需要确保开发环境满足一些基本要求。

**Node.js 版本要求**是首先要确认的事情。React 19 需要 Node.js 18.17 或更高版本，推荐使用 Node.js 20 LTS 或更高版本。如果你的电脑上有多个项目需要不同版本的 Node.js，建议使用 nvm（Node Version Manager）来管理。

检查 Node.js 和 npm 版本的命令很简单：

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 检查 npm 版本
npm --version
# 10.2.4

# 如果需要安装 nvm（macOS/Linux）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# 安装特定版本的 Node
nvm install 20
nvm use 20
nvm alias default 20
```

**选择包管理器**也是一个重要的决策。npm 是 Node.js 自带的，无需额外安装，但速度相对较慢，依赖会被重复安装。pnpm 是目前推荐的选择，它使用硬链接和符号链接来共享依赖，安装速度快，磁盘占用小。yarn 是 Facebook 早期为 React 项目开发的包管理器，速度较快，稳定性好，但依赖相对较重。

```bash
# 安装 pnpm（推荐）
npm install -g pnpm

# 或者使用 corepack（Node.js 18+）
corepack enable
corepack prepare pnpm@latest --activate
```

### 创建项目的多种方式

**方式一：Vite（推荐，现代首选）**

Vite 是目前创建 React 项目的最佳选择，由 Vue 的作者尤雨溪开发，由原生 ES 模块提供支持，提供极快的开发服务器和优化的生产构建。相比 Create React App，Vite 的开发体验要好很多。

```bash
# 创建 React + TypeScript 项目（推荐）
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

启动开发服务器后，你会看到类似这样的输出：

```
  VITE v5.0.0  ready in 567 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

打开浏览器访问 http://localhost:5173/，你就看到了你的第一个 React 应用。

**方式二：Create React App（传统方式）**

这是 Facebook 官方提供的创建 React 项目的方式，但现在已经不推荐在新项目中使用，因为启动速度较慢，配置也不够灵活。如果你想了解历史，可以使用这个方式：

```bash
# 创建项目
npx create-react-app my-react-app

cd my-react-app
npm start
```

**方式三：Next.js（全栈框架）**

Next.js 是 React 的全栈框架，提供服务端渲染（SSR）、静态站点生成（SSG）等高级功能。如果你的项目需要 SEO 优化、或者需要服务端能力，Next.js 是很好的选择。

```bash
# 创建 Next.js 项目
npx create-next-app@latest my-next-app

# 选项说明：
# - TypeScript: Yes（推荐使用 TypeScript）
# - ESLint: Yes（代码质量检查）
# - Tailwind CSS: Yes（推荐的 CSS 方案）
# - src/ directory: Yes（更好的代码组织）
# - App Router: Yes（新的路由方式）
# - Import alias: @/*（更简洁的导入路径）

cd my-next-app
pnpm dev
```

### 项目结构详解

使用 Vite 创建的 React + TypeScript 项目，标准结构如下：

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

---

## JSX 基础

### JSX 是什么

JSX 是 JavaScript XML 的缩写，它是一种语法扩展，让你在 JavaScript 中编写类似 HTML 的标记。JSX 听起来很陌生，但它其实很简单——你已经在 HTML 中写过 `<div>Hello</div>`，JSX 就是让你在 JavaScript 中写类似的东西。

为什么需要 JSX 呢？传统的 JavaScript 操作 DOM 需要写很多这样的代码：

```javascript
const element = document.createElement('div');
element.innerHTML = '<span>Hello World</span>';
element.className = 'container';
```

使用 JSX，代码可以写成这样：

```jsx
const element = <div className="container"><span>Hello World</span></div>;
```

看起来就像是在写 HTML，但它是 JavaScript。JSX 并不是模板语言，它会被编译成普通的 JavaScript 函数调用。

### JSX 语法规则

JSX 有几个关键的语法规则需要记住：

**className 而不是 class**：因为 `class` 是 JavaScript 的保留字，JSX 使用 `className` 来设置元素的 CSS 类名。

```jsx
// 正确
<div className="container">内容</div>

// 错误
<div class="container">内容</div>
```

**使用花括号嵌入表达式**：在 JSX 中，你可以在 `{}` 中嵌入任何 JavaScript 表达式。

```jsx
const name = '小明';
const element = <h1>你好，{name}！</h1>;

// 计算表达式
const a = 10;
const b = 20;
const sum = <p>{a} + {b} = {a + b}</p>;

// 调用函数
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}
const user = { firstName: '张', lastName: '三' };
const greeting = <p>欢迎，{formatName(user)}！</p>;
```

**自闭合标签必须以 /> 结尾**：没有子元素的标签必须自闭合。

```jsx
// 正确
<img src="avatar.jpg" />
<input type="text" />
<br />

// 错误
<img src="avatar.jpg">
<input type="text">
```

**JSX 只能有一个根元素**：每个 JSX 表达式只能有一个根元素，如果需要多个元素，需要用括号包裹或者使用 Fragment。

```jsx
// 使用括号包裹
const element = (
  <div>
    <h1>标题</h1>
    <p>段落</p>
  </div>
);

// 使用 Fragment
const element = (
  <>
    <h1>标题</h1>
    <p>段落</p>
  </>
);
```

**条件渲染**：使用 JavaScript 的逻辑运算符或三元表达式。

```jsx
// 使用三元运算符
const isLoggedIn = true;
const element = (
  <div>
    {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
  </div>
);

// 使用逻辑与运算符
const hasMessages = messages.length > 0;
const element = (
  <div>
    {hasMessages && <MessageList messages={messages} />}
  </div>
);
```

**列表渲染**：使用 `map` 方法遍历数组。

```jsx
const posts = [
  { id: 1, title: 'React 入门', content: '...' },
  { id: 2, title: '组件化开发', content: '...' },
  { id: 3, title: '状态管理', content: '...' }
];

const element = (
  <ul>
    {posts.map(post => (
      <li key={post.id}>
        <h2>{post.title}</h2>
        <p>{post.content}</p>
      </li>
    ))}
  </ul>
);
```

> [!IMPORTANT]
> 渲染列表时，每个元素都需要一个唯一的 `key` 属性。`key` 帮助 React 识别哪些元素改变了，添加、删除或重新排序。没有 `key`，React 可能会错误地复用 DOM 节点，导致奇怪的 Bug。优先使用 ID 作为 key，避免使用数组索引作为 key（当列表顺序可能变化时）。

---

## 组件系统

### 组件：React 的基本构建块

组件是 React 的核心概念。简单来说，组件就是一个返回 JSX 的 JavaScript 函数。你可以把它想象成乐高积木——每个积木都有自己的形状和功能，你可以用它们组合出任何你想要的建筑。

组件有两个特点需要注意：组件名必须以大写字母开头（`UserCard` 而不是 `userCard`），这样 React 才能把它和 HTML 内置标签区分开来；组件必须返回一个根元素或者使用 Fragment 包裹多个元素。

### 函数组件：现代 React 的写法

函数组件是 React 16.8 之后推荐的方式，它比类组件更简洁、更容易理解和测试。

```tsx
import { useState } from 'react';

// 基础函数组件
function Welcome() {
  return <h1>欢迎来到 React 世界！</h1>;
}

// 带参数的函数组件
function UserGreeting({ name, age }: { name: string; age: number }) {
  return (
    <div>
      <h2>你好，{name}！</h2>
      <p>你今年 {age} 岁了。</p>
    </div>
  );
}

// 带默认值的参数
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';  // 可选参数
  onClick?: () => void;               // 可选函数
}

function Button({ 
  label, 
  variant = 'primary',  // 默认值
  onClick 
}: ButtonProps) {
  const baseClass = 'px-4 py-2 rounded';
  const variantClass = variant === 'primary' 
    ? 'bg-blue-500 text-white' 
    : 'bg-gray-200 text-gray-800';
  
  return (
    <button 
      className={`${baseClass} ${variantClass}`}
      onClick={onClick}
    >
      {label}
    </button>
  );
}
```

### Props：组件之间的通信

Props（属性）是父组件向子组件传递数据的方式。props 是只读的，子组件不能直接修改从父组件接收到的 props。这听起来像是限制，但其实是 React 的有意设计——如果每个组件都可以随意修改 props，数据流动就会变得混乱不堪。

```tsx
// 定义 Props 类型
interface UserCardProps {
  name: string;
  email: string;
  avatar?: string;           // 可选属性
  role: 'admin' | 'user' | 'guest';  // 联合类型
}

// 用户卡片组件
function UserCard({ name, email, avatar, role }: UserCardProps) {
  // 角色对应的颜色
  const roleColors = {
    admin: 'bg-red-500',
    user: 'bg-blue-500',
    guest: 'bg-gray-500'
  };
  
  const roleLabels = {
    admin: '管理员',
    user: '普通用户',
    guest: '访客'
  };
  
  return (
    <div className="user-card p-4 border rounded-lg shadow">
      {avatar ? (
        <img 
          src={avatar} 
          alt={name} 
          className="w-16 h-16 rounded-full object-cover"
        />
      ) : (
        <div className="w-16 h-16 rounded-full bg-gray-200 flex items-center justify-center text-2xl">
          {name.charAt(0).toUpperCase()}
        </div>
      )}
      <h3 className="mt-2 font-bold">{name}</h3>
      <p className="text-gray-600 text-sm">{email}</p>
      <span className={`inline-block mt-2 px-2 py-1 text-xs text-white rounded ${roleColors[role]}`}>
        {roleLabels[role]}
      </span>
    </div>
  );
}

// 使用组件
function App() {
  const user = {
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const
  };
  
  return (
    <UserCard 
      name={user.name}
      email={user.email}
      role={user.role}
      avatar="https://example.com/avatar.jpg"
    />
  );
}
```

### Children：传递任意内容

有时候，你希望组件能包裹其他内容，就像 HTML 的 `<div>` 可以包含其他元素一样。这时就需要用到 `children` prop。

```tsx
interface CardProps {
  title?: string;
  footer?: React.ReactNode;
  children: React.ReactNode;  // 特殊属性，表示组件的子元素
}

function Card({ title, footer, children }: CardProps) {
  return (
    <div className="card border rounded-lg overflow-hidden">
      {title && (
        <div className="card-header bg-gray-100 p-4 border-b">
          <h2 className="font-bold">{title}</h2>
        </div>
      )}
      <div className="card-body p-4">
        {children}
      </div>
      {footer && (
        <div className="card-footer bg-gray-50 p-4 border-t">
          {footer}
        </div>
      )}
    </div>
  );
}

// 使用
function App() {
  return (
    <Card title="用户注册" footer={<Button label="提交" />}>
      <div className="space-y-4">
        <input type="text" placeholder="用户名" className="w-full border p-2 rounded" />
        <input type="email" placeholder="邮箱" className="w-full border p-2 rounded" />
        <input type="password" placeholder="密码" className="w-full border p-2 rounded" />
      </div>
    </Card>
  );
}
```

---

## State 与事件处理

### useState：组件的记忆

如果说 props 是组件从外部接收的数据，那么 state 就是组件内部自己管理的数据。state 会让组件"记住"某些信息，当这些信息改变时，组件会重新渲染。

useState 是 React 最常用的 Hook，它返回一个状态变量和一个更新这个状态的函数。

```tsx
import { useState } from 'react';

// 基础计数器
function Counter() {
  // count 是当前状态值，setCount 是更新它的函数
  const [count, setCount] = useState(0);
  
  return (
    <div className="text-center p-4">
      <p className="text-2xl mb-4">计数: {count}</p>
      <div className="space-x-2">
        <button 
          onClick={() => setCount(count + 1)}
          className="px-4 py-2 bg-blue-500 text-white rounded"
        >
          +1
        </button>
        <button 
          onClick={() => setCount(count - 1)}
          className="px-4 py-2 bg-red-500 text-white rounded"
        >
          -1
        </button>
        <button 
          onClick={() => setCount(0)}
          className="px-4 py-2 bg-gray-500 text-white rounded"
        >
          重置
        </button>
      </div>
    </div>
  );
}

// 多个状态
function MultiState() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [submitted, setSubmitted] = useState(false);
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    setSubmitted(true);
  };
  
  if (submitted) {
    return (
      <div className="p-4 bg-green-100 rounded">
        <h2>谢谢，{name}！</h2>
        <p>我们已经向 {email} 发送了确认邮件。</p>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="你的名字"
        className="w-full border p-2 rounded"
      />
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="你的邮箱"
        className="w-full border p-2 rounded"
      />
      <button type="submit" className="w-full bg-blue-500 text-white p-2 rounded">
        提交
      </button>
    </form>
  );
}
```

> [!TIP]
> 使用函数式更新可以让代码更安全。当新的状态依赖于前一个状态时，使用 `setCount(prev => prev + 1)` 而不是 `setCount(count + 1)`。这是因为 React 可能会批量处理多个状态更新，直接使用当前值可能会导致意外的结果。

### 对象状态与函数式更新

当状态是一个对象时，更新需要小心处理。React 的状态更新是合并而非替换，所以如果只想更新对象中的某个属性，需要保留其他属性。

```tsx
interface FormState {
  username: string;
  email: string;
  password: string;
  rememberMe: boolean;
}

function FormWithObject() {
  const [form, setForm] = useState<FormState>({
    username: '',
    email: '',
    password: '',
    rememberMe: false
  });
  
  // 通用字段更新函数
  const updateField = <K extends keyof FormState>(
    field: K,
    value: FormState[K]
  ) => {
    // 使用函数式更新确保获取到最新的状态
    setForm(prev => ({ ...prev, [field]: value }));
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log('提交的数据：', form);
  };
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4 p-4 max-w-md">
      <div>
        <label className="block mb-1">用户名</label>
        <input
          type="text"
          value={form.username}
          onChange={(e) => updateField('username', e.target.value)}
          className="w-full border p-2 rounded"
        />
      </div>
      
      <div>
        <label className="block mb-1">邮箱</label>
        <input
          type="email"
          value={form.email}
          onChange={(e) => updateField('email', e.target.value)}
          className="w-full border p-2 rounded"
        />
      </div>
      
      <div>
        <label className="block mb-1">密码</label>
        <input
          type="password"
          value={form.password}
          onChange={(e) => updateField('password', e.target.value)}
          className="w-full border p-2 rounded"
        />
      </div>
      
      <div className="flex items-center">
        <input
          type="checkbox"
          id="remember"
          checked={form.rememberMe}
          onChange={(e) => updateField('rememberMe', e.target.checked)}
          className="mr-2"
        />
        <label htmlFor="remember">记住我</label>
      </div>
      
      <button type="submit" className="w-full bg-blue-500 text-white p-2 rounded">
        注册
      </button>
    </form>
  );
}
```

### 事件处理

React 中的事件处理和 HTML 中的事件处理很像，但有一些小区别：事件名使用 camelCase（`onClick` 而不是 `onclick`），而且你需要传递一个函数而不是字符串。

```tsx
function EventExamples() {
  // 点击事件
  const handleClick = () => {
    alert('按钮被点击了！');
  };
  
  // 带参数的事件处理
  const handleItemClick = (itemId: string) => {
    console.log('点击了项目：', itemId);
  };
  
  // 表单提交事件
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();  // 阻止表单默认提交行为
    console.log('表单提交');
  };
  
  // 输入事件
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    console.log('输入值：', e.target.value);
  };
  
  // 键盘事件
  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter') {
      console.log('按下了回车键');
    }
  };
  
  return (
    <div className="space-y-4 p-4">
      <button onClick={handleClick} className="px-4 py-2 bg-blue-500 text-white rounded">
        点击我
      </button>
      
      <div className="space-x-2">
        {['苹果', '香蕉', '橙子'].map((item) => (
          <button 
            key={item}
            onClick={() => handleItemClick(item)}
            className="px-3 py-1 border rounded hover:bg-gray-100"
          >
            {item}
          </button>
        ))}
      </div>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          onChange={handleChange}
          onKeyDown={handleKeyDown}
          placeholder="输入内容后按回车"
          className="border p-2 rounded w-full"
        />
      </form>
    </div>
  );
}
```

---

## 副作用与生命周期

### useEffect：处理副作用

副作用是指那些影响组件外部世界的操作，比如：发送网络请求、订阅事件、操作 DOM、设置定时器等。useEffect 是 React 用于处理副作用的 Hook。

useEffect 接收两个参数：一个包含副作用逻辑的函数，和一个依赖数组。

```tsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    // 开始加载
    setLoading(true);
    setError(null);
    
    // 发送网络请求
    fetch(`https://api.example.com/users/${userId}`)
      .then(response => {
        if (!response.ok) {
          throw new Error('用户不存在');
        }
        return response.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]);  // 当 userId 改变时，重新执行这个 effect
  
  if (loading) {
    return <div className="p-4">加载中...</div>;
  }
  
  if (error) {
    return <div className="p-4 text-red-500">错误：{error}</div>;
  }
  
  return (
    <div className="p-4">
      <h2 className="text-xl font-bold">{user?.name}</h2>
      <p>{user?.email}</p>
    </div>
  );
}
```

### 清理副作用

很多副作用需要在组件卸载时清理，比如取消订阅、清除定时器、关闭 WebSocket 连接等。useEffect 可以返回一个清理函数。

```tsx
import { useState, useEffect } from 'react';

function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    // 添加事件监听
    window.addEventListener('mousemove', handleMouseMove);
    
    // 返回清理函数
    return () => {
      // 移除事件监听
      window.removeEventListener('mousemove', handleMouseMove);
    };
  }, []);  // 空依赖数组，表示只在挂载和卸载时执行
  
  return (
    <div className="p-4">
      <p>鼠标位置：({position.x}, {position.y})</p>
      <div 
        className="fixed w-4 h-4 bg-red-500 rounded-full transform -translate-x-1/2 -translate-y-1/2 pointer-events-none"
        style={{ left: position.x, top: position.y }}
      />
    </div>
  );
}

// 定时器示例
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(true);
  
  useEffect(() => {
    if (!isRunning) return;
    
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);
    
    // 清理函数
    return () => {
      clearInterval(interval);
    };
  }, [isRunning]);
  
  const formatTime = (totalSeconds: number) => {
    const hours = Math.floor(totalSeconds / 3600);
    const minutes = Math.floor((totalSeconds % 3600) / 60);
    const secs = totalSeconds % 60;
    return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  };
  
  return (
    <div className="text-center p-4">
      <p className="text-4xl font-mono mb-4">{formatTime(seconds)}</p>
      <button
        onClick={() => setIsRunning(!isRunning)}
        className="px-4 py-2 bg-blue-500 text-white rounded mr-2"
      >
        {isRunning ? '暂停' : '继续'}
      </button>
      <button
        onClick={() => setSeconds(0)}
        className="px-4 py-2 bg-gray-500 text-white rounded"
      >
        重置
      </button>
    </div>
  );
}
```

> [!WARNING]
> 常见错误是在 useEffect 中忘记包含依赖项。比如，如果你在 effect 中使用了某个变量，但这个变量不在依赖数组中，React 会给出警告。忽略这些警告可能导致 Bug，因为 effect 使用的可能是过时的值。如果确实不需要依赖某个值，可以先用 ESLint 规则检查一下是否正确。

---

## 第一个完整项目：待办事项应用

### 项目需求

让我们用一个实际的待办事项（Todo）应用来整合所学知识。这个应用需要实现以下功能：添加新待办事项、标记完成/未完成、删除待办事项、筛选显示（全部/已完成/未完成）。

### 项目实现

```tsx
// src/types/todo.ts
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

// src/hooks/useTodos.ts
import { useState, useEffect } from 'react';
import { Todo } from '../types/todo';

export function useTodos() {
  const [todos, setTodos] = useState<Todo[]>([]);
  const [filter, setFilter] = useState<'all' | 'active' | 'completed'>('all');
  
  // 从 localStorage 加载
  useEffect(() => {
    const saved = localStorage.getItem('todos');
    if (saved) {
      try {
        const parsed = JSON.parse(saved);
        setTodos(parsed.map((t: any) => ({
          ...t,
          createdAt: new Date(t.createdAt)
        })));
      } catch (e) {
        console.error('加载待办失败', e);
      }
    }
  }, []);
  
  // 保存到 localStorage
  useEffect(() => {
    localStorage.setItem('todos', JSON.stringify(todos));
  }, [todos]);
  
  const addTodo = (text: string) => {
    if (!text.trim()) return;
    
    const newTodo: Todo = {
      id: Date.now().toString(),
      text: text.trim(),
      completed: false,
      createdAt: new Date()
    };
    
    setTodos(prev => [newTodo, ...prev]);
  };
  
  const toggleTodo = (id: string) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };
  
  const deleteTodo = (id: string) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  const clearCompleted = () => {
    setTodos(prev => prev.filter(todo => !todo.completed));
  };
  
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
  
  const activeCount = todos.filter(t => !t.completed).length;
  const completedCount = todos.filter(t => t.completed).length;
  
  return {
    todos: filteredTodos,
    filter,
    setFilter,
    addTodo,
    toggleTodo,
    deleteTodo,
    clearCompleted,
    activeCount,
    completedCount
  };
}

// src/components/TodoItem.tsx
interface TodoItemProps {
  todo: Todo;
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}

function TodoItem({ todo, onToggle, onDelete }: TodoItemProps) {
  return (
    <div className={`flex items-center p-3 border-b ${todo.completed ? 'bg-gray-50' : ''}`}>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
        className="w-5 h-5 mr-3 cursor-pointer"
      />
      <span className={`flex-1 ${todo.completed ? 'line-through text-gray-400' : ''}`}>
        {todo.text}
      </span>
      <span className="text-xs text-gray-400 mr-3">
        {todo.createdAt.toLocaleDateString()}
      </span>
      <button
        onClick={() => onDelete(todo.id)}
        className="text-red-500 hover:text-red-700 px-2 py-1"
      >
        删除
      </button>
    </div>
  );
}

// src/components/TodoApp.tsx
function TodoApp() {
  const {
    todos,
    filter,
    setFilter,
    addTodo,
    toggleTodo,
    deleteTodo,
    clearCompleted,
    activeCount,
    completedCount
  } = useTodos();
  
  const [inputValue, setInputValue] = useState('');
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    addTodo(inputValue);
    setInputValue('');
  };
  
  return (
    <div className="max-w-lg mx-auto mt-10 bg-white rounded-lg shadow-lg overflow-hidden">
      <div className="p-4 bg-blue-500 text-white">
        <h1 className="text-2xl font-bold text-center">待办事项</h1>
      </div>
      
      <form onSubmit={handleSubmit} className="p-4 border-b">
        <div className="flex">
          <input
            type="text"
            value={inputValue}
            onChange={(e) => setInputValue(e.target.value)}
            placeholder="添加新待办..."
            className="flex-1 border p-2 rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button
            type="submit"
            disabled={!inputValue.trim()}
            className="bg-blue-500 text-white px-4 py-2 rounded-r disabled:bg-gray-300"
          >
            添加
          </button>
        </div>
      </form>
      
      <div className="divide-y">
        {todos.length === 0 ? (
          <div className="p-8 text-center text-gray-500">
            {filter === 'all' ? '暂无待办事项' : '没有符合条件的待办事项'}
          </div>
        ) : (
          todos.map(todo => (
            <TodoItem
              key={todo.id}
              todo={todo}
              onToggle={toggleTodo}
              onDelete={deleteTodo}
            />
          ))
        )}
      </div>
      
      <div className="p-4 bg-gray-50 flex items-center justify-between text-sm">
        <span className="text-gray-600">
          {activeCount} 项待完成
        </span>
        
        <div className="flex space-x-1">
          {(['all', 'active', 'completed'] as const).map(f => (
            <button
              key={f}
              onClick={() => setFilter(f)}
              className={`px-3 py-1 rounded ${
                filter === f 
                  ? 'bg-blue-500 text-white' 
                  : 'hover:bg-gray-200'
              }`}
            >
              {f === 'all' ? '全部' : f === 'active' ? '未完成' : '已完成'}
            </button>
          ))}
        </div>
        
        {completedCount > 0 && (
          <button
            onClick={clearCompleted}
            className="text-red-500 hover:text-red-700"
          >
            清除已完成
          </button>
        )}
      </div>
    </div>
  );
}

// src/App.tsx
function App() {
  return (
    <div className="min-h-screen bg-gray-100 py-10">
      <TodoApp />
    </div>
  );
}

export default App;
```

这个待办事项应用展示了 React 的核心概念：使用 useState 管理本地状态、使用 useEffect 处理副作用（持久化数据）、组件化开发、props 传递数据、事件处理、条件渲染和列表渲染。

---

## 进阶内容预告

你已经掌握了 React 的基础知识！但这只是开始。React 还有更多强大的特性等待你去探索。

**React进阶完全指南.md** 会深入讲解：高级 Hooks（useRef、useMemo、useCallback、useReducer、自定义 Hooks）、Context API 全局状态管理、状态管理方案（Zustand、Jotai、Redux Toolkit）、性能优化技巧、React 18 并发特性、以及错误边界、Portal 等高级模式。

**React新特性与工程实践.md** 会涵盖：React 19 新特性（Server Components、Actions、React Compiler）、Next.js 完整集成、测试策略（Vitest、React Testing Library）、生产环境部署（CI/CD、Docker、Vercel）、微前端架构、以及企业级项目的最佳实践。

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://react.dev |
| 中文文档 | https://react.nodejs.cn |
| GitHub 仓库 | https://github.com/facebook/react |
| 官方博客 | https://react.dev/blog |

> [!SUCCESS]
> 恭喜你完成了 React 入门！通过学习本指南，你已经掌握了 React 的核心概念：组件、JSX、Props、State、useEffect，以及如何创建和组织一个 React 项目。接下来，你可以继续学习进阶内容，或者开始自己的 React 项目实践。记住，最好的学习方式是动手编写代码——不要犹豫，从创建一个简单的组件开始吧！
