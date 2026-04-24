# JavaScript 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 JavaScript 2026 新特性、异步编程、模块系统、Event Loop 及在 AI 编程中的定位。

---

## 目录

1. [[#JavaScript 概述]]
2. [[#JavaScript 2026 新特性]]
3. [[#ESM vs CJS 对比]]
4. [[#Event Loop 机制]]
5. [[#异步编程模式]]
6. [[#Node.js 生态]]
7. [[#浏览器 API]]
8. [[#TypeScript 关系]]
9. [[#AI 编程中的 JavaScript]]
10. [[#实战技巧]]

---

## JavaScript 概述

### JavaScript 的地位

JavaScript 是 Web 开发的基石，从简单的表单验证脚本发展到如今的全栈编程语言：

| 平台 | JavaScript 角色 | 运行环境 |
|------|----------------|---------|
| **浏览器** | 唯一原生语言 | V8/SpiderMonkey/JavaScriptCore |
| **Node.js** | 服务端编程 | V8 |
| **Deno** | 现代运行时 | V8 |
| **边缘计算** | Cloudflare Workers | V8 |
| **移动端** | React Native/Expo | JavaScriptCore |
| **桌面端** | Electron/Tauri | V8 |
| **Serverless** | AWS Lambda/云函数 | 多种运行时 |

### JavaScript 引擎

| 引擎 | 开发者 | 使用环境 |
|------|--------|---------|
| **V8** | Google | Chrome, Node.js, Deno |
| **SpiderMonkey** | Mozilla | Firefox |
| **JavaScriptCore** | Apple | Safari, React Native |
| **Chakra** | Microsoft | 旧版 Edge |

### 语言特点

| 特点 | 说明 |
|------|------|
| **动态类型** | 类型在运行时确定 |
| **原型继承** | 基于原型链的 OO |
| **函数式特性** | 一等公民函数，闭包 |
| **异步编程** | Promise, async/await |
| **单线程** | Event Loop 驱动 |
| **多范式** | OOP/FP/命令式 |

---

## JavaScript 2026 新特性

### TC39 提案阶段

| 阶段 | 说明 | 示例功能 |
|------|------|---------|
| **Stage 0** | 稻草人提案 | 尚未正式讨论 |
| **Stage 1** | 提案 | 探索阶段 |
| **Stage 2** | 草案 | 初始规范 |
| **Stage 3** | 候选 | 接近完成，等待实现 |
| **Stage 4** | 完成 | 已添加到标准 |

### 2026 年已完成特性

**1. Array grouping（数组分组）：**

```javascript
// Object.groupBy（稳定）
const inventory = [
  { name: 'asparagus', type: 'vegetables' },
  { name: 'bananas', type: 'fruit' },
  { name: 'goat', type: 'meat' }
];

const grouped = Object.groupBy(inventory, item => item.type);
// {
//   vegetables: [{ name: 'asparagus', type: 'vegetables' }],
//   fruit: [{ name: 'bananas', type: 'fruit' }],
//   meat: [{ name: 'goat', type: 'meat' }]
// }

// Map.groupBy
const groupedMap = Map.groupBy(inventory, item => item.type);
```

**2. Array find from last（从后查找）：**

```javascript
const array = [{ id: 1 }, { id: 2 }, { id: 3 }, { id: 2 }];

array.findLast(item => item.id === 2);  // { id: 2 } (最后一个)
array.findLastIndex(item => item.id === 2);  // 3
```

**3. Shadow Realm（影子域）：**

```javascript
const realm = new ShadowRealm();

// 隔离的全局作用域
const insideValue = realm.evaluate('42');
const result = realm.evaluate(`
  globalThis.add = (a, b) => a + b;
  globalThis.add(2, 3);
`);  // 5

realm.evaluate('globalThis.someAPI = {...}');
```

**4. Iterator Helpers（迭代器助手）：**

```javascript
function* generate() {
  yield 1;
  yield 2;
  yield 3;
}

const iter = generate();

iter.map(x => x * 2).filter(x => x > 2).toArray();  // [4, 6]
```

### Stage 3 候选特性

**1. Decorators（装饰器）：**

```javascript
// 类装饰器
function logged(target, context) {
  console.log(`Class ${context.name} defined`);
  return target;
}

// 方法装饰器
function debounce(ms) {
  return function (target, context) {
    return function (...args) {
      // 防抖逻辑
    };
  };
}

// 属性装饰器
function readonly(_, context) {
  return {
    init() { return 42; }
  };
}

@logged
class MyClass {
  @readonly
  count = 0;

  @debounce(300)
  handleClick() { }
}
```

**2. Explicit Resource Management（显式资源管理）：**

```javascript
// using 声明（类似 Python with）
using file = await openFile('test.txt');
using stream = createWritableStream('output');

using { handle, cleanup } = acquireResource();

// Suppressor 符号
using timer = new Timer();
await timer.start();

// 结束时自动清理
```

**3. Float16Array（Float16 数组）：**

```javascript
const float16 = new Float16Array([1, 2, 3, 4]);
const float16Values = Float16Array.of(1.5, 2.5, 3.5);

// 与 WebGPU/ML 配合
const model = new Model();
model.setWeights(float16Data);
```

---

## ESM vs CJS 对比

### 模块系统发展

| 系统 | 全称 | 引入版本 | 文件扩展 |
|------|------|---------|---------|
| **CJS** | CommonJS | Node.js 0.x | `.js`, `.cjs` |
| **AMD** | Async Module Definition | RequireJS | `.js` |
| **UMD** | Universal Module Definition | 第三方 | `.js` |
| **ESM** | ECMAScript Modules | ES2015/Node 14 | `.mjs`, `.js` |

### CommonJS 语法

```javascript
// 导出
module.exports = { name: 'MyModule' };
module.exports.MyClass = class { };

// 或者
exports.myFunction = function() { };
exports.myValue = 42;

// 导入
const MyModule = require('./MyModule');
const { myFunction } = require('./utils');

// 动态 require
const moduleName = 'fs';
const fs = require(moduleName);
```

### ESM 语法

```javascript
// 命名导出
export const myValue = 42;
export function myFunction() { }
export class MyClass { }

// 默认导出
export default class { }

// 导入
import MyClass from './MyModule.js';
import { myValue, myFunction } from './utils.js';
import * as utils from './utils.js';
import { myValue as value } from './utils.js';

// 动态导入
const module = await import('./dynamic.js');
```

### 核心差异对比

| 维度 | ESM | CJS |
|------|-----|-----|
| **语法** | `import`/`export` | `require`/`module.exports` |
| **加载时机** | 静态（编译时解析） | 动态（运行时解析） |
| **循环引用** | 支持（可能有 undefined） | 支持（慎用） |
| **Tree shaking** | 支持 | 不支持 |
| **异步加载** | 支持（import()） | 不支持 |
| **浏览器支持** | 原生支持 | 需打包 |
| **Node.js 支持** | 需要 `.mjs` 或 `"type": "module"` | 原生支持 |
| **严格模式** | 默认启用 | 可选 |

### 互操作性

```javascript
// ESM 中使用 CJS 模块
import fs from 'fs';  // 自动包装为 ESM
import { readFile } from 'fs';  // 命名导入

// CJS 中使用 ESM 模块（需要动态 import）
const { default: myModule } = await import('./esm-module.mjs');
const { namedExport } = await import('./esm-module.mjs');
```

### 导入断言

```javascript
// JSON 模块导入
import config from './config.json' assert { type: 'json' };

// 使用 with 语法（新版）
import config from './config.json' with { type: 'json' };

// CSS 模块（特定环境）
import styles from './styles.css' assert { type: 'css' };
```

---

## Event Loop 机制

### JavaScript 单线程模型

JavaScript 是单线程语言，通过 Event Loop 实现非阻塞异步操作：

```
┌─────────────────────────────────────────┐
│                  Stack                   │
│         (执行上下文/调用栈)               │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  function foo() {                   │ │
│  │    console.log(1);                  │ │
│  │    bar();  ← 调用 bar()             │ │
│  │    console.log(3);                  │ │
│  │  }                                  │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────┐
│               Heap                       │
│           (对象/内存分配)                 │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  { name: 'Alice', age: 30 }        │ │
│  │  [1, 2, 3, 4, 5]                  │ │
│  │  function add(a, b) { ... }       │ │
│  └────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

### Event Loop 流程

```
                    ┌──────────────────────┐
                    │       Call Stack     │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │    Microtask Queue    │
                    │  (Promise.then,      │
                    │   queueMicrotask)    │
                    └──────────┬───────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
┌─────────▼─────────┐ ┌────────▼────────┐ ┌────────▼────────┐
│   setTimeout       │ │  I/O Callbacks │ │  Check Phase   │
│   setInterval      │ │  (文件/网络)    │ │  setImmediate │
└───────────────────┘ └─────────────────┘ └────────────────┘
          │                    │                    │
          └────────────────────┼────────────────────┘
                               │
                    ┌──────────▼───────────┐
                    │    Macrotask Queue    │
                    │  (setTimeout,        │
                    │   setInterval,      │
                    │   I/O)              │
                    └─────────────────────┘
```

### 执行顺序示例

```javascript
console.log('1');           // 同步

setTimeout(() => console.log('2'), 0);  // Macrotask

Promise.resolve()
  .then(() => console.log('3'));  // Microtask

queueMicrotask(() => console.log('4'));  // Microtask

console.log('5');           // 同步

// 输出顺序：1, 5, 3, 4, 2
```

### 浏览器 vs Node.js Event Loop

| 阶段 | 浏览器 | Node.js |
|------|--------|---------|
| **1** | 执行同步代码 | 执行同步代码 |
| **2** | 执行所有微任务 | 执行所有微任务 |
| **3** | 渲染（如果需要） | timers (setTimeout/setInterval) |
| **4** | 执行宏任务 | pending callbacks |
| **5** | - | idle, prepare |
| **6** | - | poll (retrieve new I/O events) |
| **7** | - | check (setImmediate) |
| **8** | - | close callbacks |

### Node.js 特殊 API

```javascript
// setImmediate vs setTimeout
setImmediate(() => console.log('immediate'));
setTimeout(() => console.log('timeout'), 0);
// 顺序不确定，取决于 I/O 状态

// process.nextTick（最高优先级微任务）
Promise.resolve().then(() => console.log('promise'));
process.nextTick(() => console.log('nextTick'));
// 输出: nextTick, promise

// 排队机制
function recursive() {
  queueMicrotask(recursive);
  // 不会导致栈溢出
}
```

---

## 异步编程模式

### 回调模式（传统）

```javascript
// 回调地狱
fetchData(url, (err, data) => {
  if (err) {
    handleError(err);
    return;
  }

  processData(data, (err, result) => {
    if (err) {
      handleError(err);
      return;
    }

    saveResult(result, (err, saved) => {
      if (err) {
        handleError(err);
        return;
      }

      sendNotification(saved, (err) => {
        if (err) handleError(err);
      });
    });
  });
});
```

### Promise 模式

```javascript
// 链式调用
fetchData(url)
  .then(data => processData(data))
  .then(result => saveResult(result))
  .then(saved => sendNotification(saved))
  .catch(handleError)
  .finally(() => cleanup());

// Promise.all - 并行执行
const [users, posts, comments] = await Promise.all([
  fetchUsers(),
  fetchPosts(),
  fetchComments()
]);

// Promise.allSettled - 所有都执行完毕
const results = await Promise.allSettled([
  fetchData1(),
  fetchData2(),
  fetchData3()
]);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`Data ${index}:`, result.value);
  } else {
    console.log(`Error ${index}:`, result.reason);
  }
});

// Promise.race - 竞速
const result = await Promise.race([
  fetchWithTimeout(url1, 1000),
  fetchWithTimeout(url2, 1000)
]);
```

### async/await 模式

```javascript
// 基本用法
async function fetchAndProcess(url) {
  try {
    const data = await fetchData(url);
    const processed = await processData(data);
    return processed;
  } catch (error) {
    handleError(error);
    throw error;
  }
}

// 并行执行优化
async function fetchAll() {
  // 错误方式：顺序执行
  const a = await fetchA();  // 等待 100ms
  const b = await fetchB();  // 等待 100ms
  // 总计 200ms

  // 正确方式：并行执行
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
  // 总计 100ms
}

// 条件并行
const [user, posts] = await Promise.all([
  hasUserId ? fetchUser(userId) : null,
  fetchPosts()
]);
```

### 异步迭代器

```javascript
// AsyncIterator
async function* asyncGenerator() {
  let i = 0;
  while (i < 10) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield i++;
  }
}

// 使用 for await...of
for await (const value of asyncGenerator()) {
  console.log(value);
}

// 处理流式数据
async function processStream(url) {
  const response = await fetch(url);
  const reader = response.body.getReader();

  const decoder = new TextDecoder();
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log(decoder.decode(value));
  }
}
```

---

## Node.js 生态

### 包管理器对比

| 特性 | npm | pnpm | yarn |
|------|-----|------|------|
| **安装速度** | 中等 | 最快 | 快 |
| **磁盘占用** | 大 | 小 | 中等 |
| **node_modules** | 扁平 | 隔离 | 扁平/PnP |
| **lockfile** | package-lock.json | pnpm-lock.yaml | yarn.lock |
| **幽灵依赖** | 有 | 无 | 有 |
| **monorepo** | workspaces | workspace | workspace |

### 常用命令

```bash
# npm
npm install package          # 安装
npm install -D package       # 开发依赖
npm install package@version  # 指定版本
npm update                  # 更新
npm audit                   # 安全审计
npm run build               # 运行脚本

# pnpm
pnpm add package            # 安装
pnpm add -D package         # 开发依赖
pnpm install                # 从 lockfile 安装
pnpm update                 # 更新

# yarn
yarn add package            # 安装
yarn add -D package         # 开发依赖
yarn install                # 安装
yarn upgrade               # 更新
```

### 常用全局工具

| 工具 | 功能 | npm 包 |
|------|------|--------|
| **npx** | 运行本地包 | npm 内置 |
| **ts-node** | 运行 TypeScript | ts-node |
| **tsx** | 快速运行 TS | tsx |
| **nodemon** | 开发热重载 | nodemon |
| **rimraf** | 跨平台删除 | rimraf |
| **pnpm** | 包管理器 | pnpm |

### Node.js 核心模块

| 模块 | 功能 |
|------|------|
| **fs** | 文件系统操作 |
| **path** | 路径处理 |
| **http/https** | HTTP 服务器/客户端 |
| **crypto** | 加密功能 |
| **events** | 事件发射器 |
| **stream** | 流处理 |
| **buffer** | 二进制数据 |
| **url** | URL 解析 |
| **querystring** | 查询字符串 |
| **os** | 操作系统信息 |
| **util** | 工具函数 |
| **child_process** | 子进程 |
| **cluster** | 集群 |
| **worker_threads** | 工作线程 |

### 常用第三方模块

| 类别 | 模块 | 说明 |
|------|------|------|
| **Web 框架** | express, koa, fastify | HTTP 服务器 |
| **全栈框架** | next, nuxt, remix | SSR 框架 |
| **数据库** | prisma, mongoose, sequelize | ORM |
| **验证** | zod, joi, class-validator | 数据验证 |
| **日期** | dayjs, date-fns, luxon | 日期处理 |
| **工具** | lodash, ramda | 函数式工具 |
| **颜色** | chalk, picocolors | 终端颜色 |
| **日志** | winston, pino | 日志库 |

---

## 浏览器 API

### Fetch API

```javascript
// 基本 GET 请求
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// 带参数
const params = new URLSearchParams({ page: 1, limit: 10 });
const response = await fetch(`https://api.example.com/data?${params}`);

// POST 请求
const response = await fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token'
  },
  body: JSON.stringify({ name: 'Alice', age: 30 })
});

// 处理错误
if (!response.ok) {
  throw new Error(`HTTP ${response.status}: ${response.statusText}`);
}

// 流式响应
const response = await fetch('https://api.example.com/stream');
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value);
  // 处理数据块
}
```

### WebSocket

```javascript
// 连接
const ws = new WebSocket('wss://api.example.com/socket');

// 事件处理
ws.onopen = () => {
  console.log('Connected');
  ws.send(JSON.stringify({ type: 'subscribe', channel: 'news' }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

ws.onerror = (error) => {
  console.error('WebSocket error:', error);
};

ws.onclose = () => {
  console.log('Disconnected');
};

// 发送消息
ws.send(JSON.stringify({ message: 'Hello' }));

// 关闭连接
ws.close();
```

### Intersection Observer

```javascript
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        console.log('Element visible:', entry.target);
        // 加载内容、发送分析等
      }
    });
  },
  {
    root: null,           // 视口
    rootMargin: '0px',    // 边距
    threshold: 0.1        // 10% 可见时触发
  }
);

// 观察元素
document.querySelectorAll('.lazy-load').forEach((el) => {
  observer.observe(el);
});

// 停止观察
observer.disconnect();
```

### 其他常用 API

```javascript
// BroadcastChannel（同源通信）
const channel = new BroadcastChannel('my_channel');
channel.postMessage({ type: 'update', data: {...} });
channel.onmessage = (event) => { ... };

// ResizeObserver
const resizeObserver = new ResizeObserver((entries) => {
  entries.forEach((entry) => {
    const { width, height } = entry.contentRect;
    console.log(`Size: ${width}x${height}`);
  });
});
resizeObserver.observe(element);

// Navigator API
navigator.clipboard.writeText('Hello');           // 剪贴板
navigator.share({ title: 'Title', url: '...' }); // 分享
navigator.onLine;                                  // 在线状态
navigator.storage.estimate();                      // 存储配额

// Screen API
screen.width;         // 屏幕宽度
screen.height;        // 屏幕高度
screen.orientation;   // 方向

// Vibration API
navigator.vibrate([200, 100, 200]);  // 震动模式
```

---

## TypeScript 关系

### TypeScript 简介

TypeScript 是 JavaScript 的超集，添加了类型系统：

```javascript
// JavaScript
function add(a, b) {
  return a + b;
}
add(1, 2);    // 3
add('a', 'b'); // 'ab'
add(1, 'b');  // '1b' (隐式转换)
```

```typescript
// TypeScript
function add(a: number, b: number): number {
  return a + b;
}
add(1, 2);     // 3 ✓
add('a', 'b'); // Error ✓
```

### 类型推断与标注

```typescript
// 类型推断（多数情况不需要显式标注）
let name = 'Alice';  // string
let age = 30;         // number
let isActive = true;  // boolean

// 显式标注
let name: string = 'Alice';
let age: number = 30;
let isActive: boolean = true;

// 函数标注
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// 箭头函数
const add = (a: number, b: number): number => a + b;

// 可选参数
function createUser(name: string, age?: number): User {
  return { name, age };
}

// 默认参数
function greet(name: string = 'Guest'): string {
  return `Hello, ${name}!`;
}
```

### 泛型

```typescript
// 泛型函数
function identity<T>(arg: T): T {
  return arg;
}

identity<string>('hello');  // string
identity(42);               // number（推断）

// 泛型接口
interface Container<T> {
  value: T;
  getValue(): T;
}

// 泛型约束
function getLength<T extends { length: number }>(arg: T): number {
  return arg.length;
}

getLength('hello');  // 5
getLength([1, 2, 3]); // 3
getLength({ length: 10 }); // 10
```

### 实用类型

```typescript
// Partial - 所有属性可选
type PartialUser = Partial<User>;

// Required - 所有属性必需
type RequiredUser = Required<User>;

// Pick - 选择属性
type UserPreview = Pick<User, 'id' | 'name'>;

// Omit - 排除属性
type UserWithoutPassword = Omit<User, 'password'>;

// Record - 键值对
type UserMap = Record<string, User>;

// Exclude - 排除类型
type NonNull = Exclude<string | null | undefined, null | undefined>;

// Extract - 提取类型
type StringOnly = Extract<string | number | boolean, string>;

// ReturnType - 返回类型
type AddReturn = ReturnType<typeof add>;

// Parameters - 参数类型
type AddParams = Parameters<typeof add>;
```

---

## AI 编程中的 JavaScript

### AI 助手的 JavaScript 优势

1. **语法简洁**：相比 TypeScript，AI 更容易生成正确的 JavaScript
2. **生态系统**：npm 生态极其丰富
3. **跨平台**：浏览器/Node.js/边缘计算
4. **动态性**：快速原型和迭代
5. **类型辅助**：JSDoc 注释提供类型提示

### JSDoc 类型注释

```javascript
/**
 * @typedef {Object} User
 * @property {string} id
 * @property {string} name
 * @property {number} age
 * @property {string} [email]
 */

/**
 * 获取用户
 * @param {string} id - 用户 ID
 * @returns {Promise<User>} 用户对象
 */
async function getUser(id) {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// @type 注解
/** @type {Map<string, User>} */
const userMap = new Map();

// @param 注解
/**
 * @param {string} name
 * @param {number} [age]
 */
function greet(name, age) {
  console.log(`Hello, ${name}!`);
  if (age !== undefined) {
    console.log(`You are ${age} years old.`);
  }
}
```

### AI 生成代码示例

```javascript
// AI Prompt: "用 JavaScript 实现一个防抖函数"

function debounce(func, wait) {
  let timeoutId;

  return function (...args) {
    clearTimeout(timeoutId);

    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, wait);
  };
}

// 使用
const handleInput = debounce((e) => {
  console.log('Input:', e.target.value);
  // 发送搜索请求
}, 300);

document.querySelector('input').addEventListener('input', handleInput);
```

---

## 实战技巧

### 性能优化

```javascript
// 防抖
function debounce(fn, ms) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
}

// 节流
function throttle(fn, ms) {
  let lastCall = 0;
  return (...args) => {
    const now = Date.now();
    if (now - lastCall >= ms) {
      lastCall = now;
      fn(...args);
    }
  };
}

// 记忆化
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}
```

### 数组操作技巧

```javascript
// 去重
[...new Set([1, 2, 2, 3])];  // [1, 2, 3]

// 分组
Object.groupBy(items, item => item.category);

// 展平
[1, [2, [3]]].flat(2);  // [1, 2, 3]
[1, [2, [3]]].flatMap(x => x);  // [1, 2, [3]]

// 安全访问
const value = obj?.nested?.property ?? 'default';

// 条件执行
arr?.filter(x => x > 0)?.map(x => x * 2);

// 链式调用
const result = data
  ?.filter(x => x.active)
  ?.map(x => transform(x))
  ?.sort((a, b) => a.value - b.value);
```

### 错误处理

```javascript
// try-catch-finally
async function fetchData() {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  } finally {
    cleanup();
  }
}

// 自定义错误
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

// 全局错误处理
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
});

window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
});
```

---

## JavaScript 高级主题

### 代理与反射

```javascript
// Proxy 基础用法
const handler = {
  get(target, prop) {
    console.log(`Getting ${prop}`);
    return Reflect.get(target, prop);
  },
  set(target, prop, value) {
    console.log(`Setting ${prop} to ${value}`);
    return Reflect.set(target, prop, value);
  }
};

const obj = new Proxy({ name: 'Alice' }, handler);
console.log(obj.name); // 输出: Getting name, Alice
obj.age = 25; // 输出: Setting age to 25

// 响应式系统实现
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key);
      return target[key];
    },
    set(target, key, value) {
      target[key] = value;
      trigger(target, key);
      return true;
    }
  });
}

// 表单验证器
const validator = new Proxy({}, {
  set(target, prop, value) {
    if (prop === 'email' && !value.includes('@')) {
      throw new TypeError('Invalid email');
    }
    target[prop] = value;
    return true;
  }
});
```

### 迭代器与生成器

```javascript
// 自定义迭代器
class Range {
  constructor(start, end) {
    this.start = start;
    this.end = end;
    this.current = start;
  }

  [Symbol.iterator]() {
    return {
      current: this.start,
      end: this.end,
      next() {
        if (this.current <= this.end) {
          return { value: this.current++, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
}

for (const num of new Range(1, 5)) {
  console.log(num); // 1, 2, 3, 4, 5
}

// 生成器函数
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const gen = idGenerator();
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2

// 异步生成器
async function* fetchPages(url) {
  let page = 1;
  while (page <= 10) {
    const data = await fetch(`${url}?page=${page}`);
    yield { page, data: await data.json() };
    page++;
  }
}
```

### WeakMap、WeakSet 与内存管理

```javascript
// 私有属性
const privateData = new WeakMap();

class User {
  constructor(name, password) {
    privateData.set(this, { name, password });
  }

  getName() {
    return privateData.get(this).name;
  }

  authenticate(pwd) {
    return privateData.get(this).password === pwd;
  }
}

// 缓存示例
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  const result = expensiveComputation(obj);
  cache.set(obj, result);
  return result;
}

// WeakSet 用于对象标记
const registeredObjects = new WeakSet();

function register(obj) {
  if (registeredObjects.has(obj)) {
    console.log('Already registered');
    return false;
  }
  registeredObjects.add(obj);
  return true;
}
```

### 模块系统进阶

```javascript
// 动态导入
const module = await import('./module.js');
module.default();
module.namedExport();

// 条件导入
const isNode = typeof window === 'undefined';
const utils = isNode 
  ? await import('./utils-node.js')
  : await import('./utils-browser.js');

// 模块合成
export { a, b } from './module.js';
export * from './module.js';
export { default as moduleName } from './module.js';

// Re-export 聚合器
// utils/index.js
export { add } from './math.js';
export { format } from './string.js';
export { parse } from './date.js';
```

### 装饰器提案（TC39 第三阶段）

```javascript
// 类装饰器
function logClass(target) {
  console.log(`Class ${target.name} defined`);
  return target;
}

// 方法装饰器
function memoize(target, name, descriptor) {
  const original = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = original.apply(this, args);
    cache.set(key, result);
    return result;
  };
  return descriptor;
}

// 属性装饰器
function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

@logClass
class Calculator {
  @memoize
  fibonacci(n) {
    return n <= 1 ? n : this.fibonacci(n-1) + this.fibonacci(n-2);
  }
  
  @readonly
  VERSION = '1.0.0';
}
```

### 尾递归优化

```javascript
// 非优化版本
function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return factorial(n - 1, n * acc);
}

// trampoline 实现
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

function factorialTrampoline(n, acc = 1) {
  if (n <= 1) return acc;
  return () => factorialTrampoline(n - 1, n * acc);
}

const factorialOptimized = trampoline(factorialTrampoline);
```

---

## JavaScript 与 AI 集成

### Fetch API 与流式响应

```javascript
// 流式文本处理
async function* streamText(response) {
  const reader = response.body.getReader();
  const decoder = new TextDecoder();
  
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    yield decoder.decode(value, { stream: true });
  }
}

// OpenAI 流式调用
async function* streamAI(prompt) {
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${API_KEY}`
    },
    body: JSON.stringify({
      model: 'gpt-4',
      messages: [{ role: 'user', content: prompt }],
      stream: true
    })
  });
  
  for await (const chunk of streamText(response)) {
    const lines = chunk.split('\n').filter(l => l.startsWith('data: '));
    for (const line of lines) {
      const data = JSON.parse(line.slice(6));
      const content = data.choices?.[0]?.delta?.content;
      if (content) yield content;
    }
  }
}
```

### Service Worker 与离线 AI

```javascript
// sw.js
const CACHE_NAME = 'ai-model-cache';

self.addEventListener('fetch', (event) => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(networkFirst(event.request));
  }
});

async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, response.clone());
    return response;
  } catch {
    const cached = await caches.match(request);
    return cached || new Response('Offline');
  }
}
```

---

## JavaScript 设计模式

### 模块模式

```javascript
// 命名空间模式
const MyApp = MyApp || {};

MyApp.Utils = (function() {
  const _private = 'secret';
  
  function _formatDate(date) {
    return date.toISOString().split('T')[0];
  }
  
  return {
    formatDate: _formatDate,
    publicAPI: function() {
      return 'Public method';
    }
  };
})();

// ES Module 模式
export class DataStore {
  #items = new Map();
  
  add(key, value) {
    this.#items.set(key, value);
  }
  
  get(key) {
    return this.#items.get(key);
  }
  
  *[Symbol.iterator]() {
    for (const [key, value] of this.#items) {
      yield { key, value };
    }
  }
}

export default DataStore;
```

### 观察者与发布订阅

```javascript
class EventEmitter {
  #events = new Map();
  
  on(event, listener) {
    if (!this.#events.has(event)) {
      this.#events.set(event, []);
    }
    this.#events.get(event).push(listener);
    return this;
  }
  
  off(event, listener) {
    if (this.#events.has(event)) {
      const listeners = this.#events.get(event);
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    }
    return this;
  }
  
  emit(event, ...args) {
    if (this.#events.has(event)) {
      for (const listener of this.#events.get(event)) {
        listener(...args);
      }
    }
    return this;
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
}

// 使用
const emitter = new EventEmitter();
emitter.on('message', (data) => console.log('Message:', data));
emitter.emit('message', { text: 'Hello!' });
```

### 工厂与抽象工厂

```javascript
// 工厂函数
function createUser(name, role) {
  const user = {
    name,
    role,
    permissions: [],
    createdAt: new Date(),
  };
  
  if (role === 'admin') {
    user.permissions = ['read', 'write', 'delete', 'manage'];
  } else if (role === 'editor') {
    user.permissions = ['read', 'write'];
  } else {
    user.permissions = ['read'];
  }
  
  return user;
}

// 抽象工厂
class UIFactory {
  createButton() { throw new Error('Not implemented'); }
  createInput() { throw new Error('Not implemented'); }
}

class DarkThemeFactory extends UIFactory {
  createButton() {
    return { style: 'dark', text: 'Submit' };
  }
  createInput() {
    return { style: 'dark', placeholder: 'Enter...' };
  }
}

class LightThemeFactory extends UIFactory {
  createButton() {
    return { style: 'light', text: 'Submit' };
  }
  createInput() {
    return { style: 'light', placeholder: 'Enter...' };
  }
}
```

### 策略模式

```javascript
class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }
  
  setStrategy(strategy) {
    this.strategy = strategy;
  }
  
  process(amount) {
    return this.strategy.pay(amount);
  }
}

const creditCardStrategy = {
  pay(amount) {
    return { method: 'credit_card', amount, transactionId: crypto.randomUUID() };
  }
};

const paypalStrategy = {
  pay(amount) {
    return { method: 'paypal', amount, email: 'user@example.com' };
  }
};

const cryptoStrategy = {
  pay(amount) {
    return { method: 'crypto', amount, wallet: '0x...' };
  }
};

// 使用
const processor = new PaymentProcessor(creditCardStrategy);
processor.process(100);

processor.setStrategy(paypalStrategy);
processor.process(200);
```

---

## JavaScript 测试

### Vitest 配置与使用

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/']
    },
    testTimeout: 10000,
    hookTimeout: 10000
  }
});

// example.test.js
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('Math Utils', () => {
  it('should add two numbers', () => {
    expect(1 + 2).toBe(3);
  });
  
  it('should handle async operations', async () => {
    const result = await Promise.resolve(42);
    expect(result).toBe(42);
  });
});

describe('API Client', () => {
  const mockFetch = vi.fn();
  
  beforeEach(() => {
    mockFetch.mockReset();
  });
  
  it('should fetch data', async () => {
    mockFetch.mockResolvedValue({ 
      ok: true, 
      json: async () => ({ data: 'test' }) 
    });
    
    const response = await mockFetch('https://api.example.com');
    const data = await response.json();
    
    expect(data).toEqual({ data: 'test' });
  });
});
```

### 测试覆盖率

```bash
# 运行测试并生成覆盖率
npx vitest run --coverage

# 查看 HTML 报告
npx vitest --coverage --coverage.reporter=html
```

### Mock 函数

```javascript
// 创建 mock
const mockFn = vi.fn();

// 设置实现
mockFn.mockImplementation((x) => x * 2);

// 链式调用
mockFn.mockReturnValueOnce(1).mockReturnValueOnce(2).mockReturnValue(3);

// 模拟模块
vi.mock('./utils', () => ({
  formatDate: vi.fn((date) => '2024-01-01'),
  parseJSON: vi.fn((str) => JSON.parse(str))
}));

// 模拟定时器
vi.useFakeTimers();

setTimeout(() => {
  console.log('Delayed');
}, 1000);

vi.runAllTimers(); // 立即执行

vi.useRealTimers();
```

---

## JavaScript 性能优化

### 防抖与节流

```javascript
// 防抖
function debounce(func, wait) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, wait);
  };
}

// 节流
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

// 使用
const handleInput = debounce((e) => {
  console.log('Search:', e.target.value);
}, 300);

const handleScroll = throttle(() => {
  console.log('Scrolled:', window.scrollY);
}, 100);
```

### 虚拟列表

```javascript
class VirtualList {
  constructor(container, items, itemHeight, renderItem) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.renderItem = renderItem;
    
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    this.scrollTop = 0;
    
    container.addEventListener('scroll', () => this.onScroll());
    this.render();
  }
  
  onScroll() {
    this.scrollTop = this.container.scrollTop;
    this.render();
  }
  
  render() {
    const startIndex = Math.floor(this.scrollTop / this.itemHeight);
    const endIndex = Math.min(
      startIndex + this.visibleCount + 2,
      this.items.length
    );
    
    const totalHeight = this.items.length * this.itemHeight;
    const offsetY = startIndex * this.itemHeight;
    
    this.container.innerHTML = `
      <div style="height: ${totalHeight}px; position: relative;">
        ${this.items.slice(startIndex, endIndex).map((item, i) => `
          <div style="position: absolute; top: ${offsetY + i * this.itemHeight}px; height: ${this.itemHeight}px;">
            ${this.renderItem(item)}
          </div>
        `).join('')}
      </div>
    `;
  }
}
```

### 内存泄漏检测

```javascript
// 检测全局变量泄漏
function detectLeaks() {
  const initialKeys = Object.keys(window);
  
  return {
    check: () => {
      const currentKeys = Object.keys(window);
      const leaked = currentKeys.filter(k => !initialKeys.includes(k));
      if (leaked.length > 0) {
        console.warn('Potential memory leaks:', leaked);
      }
      return leaked;
    }
  };
}

// 使用 WeakMap 避免泄漏
const cache = new WeakMap();

function processData(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  
  const result = heavyComputation(obj);
  cache.set(obj, result);
  return result;
}

// 清理事件监听器
class Component {
  #handler = () => this.onClick();
  
  mount() {
    document.addEventListener('click', this.#handler);
  }
  
  unmount() {
    document.removeEventListener('click', this.#handler);
  }
}
```

---

## JavaScript 工具链

### Vite 配置

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true
      }
    }
  },
  build: {
    target: 'esnext',
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash', 'moment']
        }
      }
    }
  },
  optimizeDeps: {
    include: ['react', 'react-dom']
  }
});
```

### Webpack 配置

```javascript
// webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    clean: true,
    publicPath: '/'
  },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react']
          }
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader']
      },
      {
        test: /\.(png|jpg|gif|svg)$/,
        type: 'asset/resource'
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components')
    }
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html',
      minify: true
    }),
    new DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
    })
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all'
        }
      }
    }
  }
};
```

### ESLint 配置

```javascript
// .eslintrc.js
module.exports = {
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true
    }
  },
  env: {
    browser: true,
    node: true,
    es2022: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  plugins: ['react', '@typescript-eslint'],
  rules: {
    'no-console': 'warn',
    'no-unused-vars': 'warn',
    'prefer-const': 'error',
    'react/prop-types': 'off'
  },
  settings: {
    react: {
      version: 'detect'
    }
  }
};
```

---

> [!SUCCESS]
> JavaScript 作为 Web 开发的基石语言，在 2026 年持续演进。从 ESM 模块系统到现代化的异步编程模式，从浏览器 API 到 Node.js 生态，JavaScript 为全栈开发提供了统一的语言基础。配合 TypeScript 的类型系统，JavaScript 已成为 AI 辅助编程时代最具生产力的语言之一。

---

## JavaScript 高级主题

### 装饰器提案（Stage 3）

```javascript
// 装饰器是一种特殊的声明，可以修改类的行为

// 基本装饰器
function logged(target, name, descriptor) {
  const original = descriptor.value;
  
  descriptor.value = function(...args) {
    console.log(`Calling ${name} with`, args);
    const result = original.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };
  
  return descriptor;
}

class Calculator {
  @logged
  add(a, b) {
    return a + b;
  }
  
  @logged
  multiply(a, b) {
    return a * b;
  }
}

const calc = new Calculator();
calc.add(2, 3); // 打印日志并返回结果

// 类装饰器
function sealed(constructor) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class Person {
  constructor(name) {
    this.name = name;
  }
}

// 装饰器工厂
function prefix(prefix) {
  return function(target, name, descriptor) {
    const original = descriptor.value;
    
    descriptor.value = function(...args) {
      const result = original.apply(this, args);
      return `${prefix}: ${result}`;
    };
    
    return descriptor;
  };
}

class Formatter {
  @prefix('Result')
  format(value) {
    return value.toUpperCase();
  }
}

// 参数装饰器
function validate(target, name, descriptor) {
  const original = descriptor.value;
  
  descriptor.value = function(...args) {
    if (args.some(arg => arg === undefined)) {
      throw new Error('Arguments cannot be undefined');
    }
    return original.apply(this, args);
  };
  
  return descriptor;
}

class Service {
  @validate
  process(data) {
    return data;
  }
}
```

### 私有字段与封装

```javascript
class BankAccount {
  // 真正的私有字段
  #balance = 0;
  #transactionHistory = [];
  
  constructor(initialBalance = 0) {
    this.#balance = initialBalance;
  }
  
  // 公开的 getter
  get balance() {
    return this.#balance;
  }
  
  get transactionCount() {
    return this.#transactionHistory.length;
  }
  
  deposit(amount) {
    if (amount <= 0) {
      throw new Error('Deposit amount must be positive');
    }
    this.#balance += amount;
    this.#recordTransaction('deposit', amount);
    return this;
  }
  
  withdraw(amount) {
    if (amount <= 0) {
      throw new Error('Withdrawal amount must be positive');
    }
    if (amount > this.#balance) {
      throw new Error('Insufficient funds');
    }
    this.#balance -= amount;
    this.#recordTransaction('withdrawal', amount);
    return this;
  }
  
  // 私有方法
  #recordTransaction(type, amount) {
    this.#transactionHistory.push({
      type,
      amount,
      timestamp: new Date()
    });
  }
  
  // 私有 getter
  get #lastTransaction() {
    return this.#transactionHistory.at(-1);
  }
  
  printLastTransaction() {
    console.log('Last transaction:', this.#lastTransaction);
  }
}

const account = new BankAccount(1000);
account.deposit(500);
account.withdraw(200);
console.log(account.balance); // 1300
account.printLastTransaction();
```

### Symbol 与 Symbol.for

```javascript
// Symbol 创建唯一标识符
const id = Symbol('id');
const user = {
  [id]: 1,
  name: 'Alice'
};

console.log(user[id]); // 1

// Symbol.for 在全局注册表中查找或创建
const sym1 = Symbol.for('app.key');
const sym2 = Symbol.for('app.key');

console.log(sym1 === sym2); // true
console.log(Symbol.keyFor(sym1)); // 'app.key'

// 常用内置 Symbol
const iterable = {
  [Symbol.iterator]() {
    let step = 0;
    return {
      next() {
        step++;
        return step <= 3 
          ? { value: step, done: false }
          : { done: true };
      }
    };
  }
};

for (const v of iterable) {
  console.log(v); // 1, 2, 3
}

// Symbol.hasInstance
class EvenNumbers {
  static [Symbol.hasInstance](obj) {
    return Number.isInteger(obj) && obj % 2 === 0;
  }
}

console.log(2 instanceof EvenNumbers); // true
console.log(3 instanceof EvenNumbers); // false

// Symbol.toStringTag
class Dog {
  get [Symbol.toStringTag]() {
    return 'Dog';
  }
}

const dog = new Dog();
console.log(dog.toString()); // '[object Dog]'
```

### 代理与元编程

```javascript
// Proxy 用于自定义基本操作
const handler = {
  get(target, prop, receiver) {
    console.log(`Getting ${prop}`);
    return Reflect.get(target, prop, receiver);
  },
  
  set(target, prop, value, receiver) {
    console.log(`Setting ${prop} = ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
  
  has(target, prop) {
    console.log(`Checking if ${prop} exists`);
    return Reflect.has(target, prop);
  },
  
  deleteProperty(target, prop) {
    console.log(`Deleting ${prop}`);
    return Reflect.deleteProperty(target, prop);
  }
};

const proxy = new Proxy({ name: 'Alice' }, handler);
proxy.name; // 触发 get
proxy.name = 'Bob'; // 触发 set

// 验证器示例
const validator = {
  set(target, prop, value) {
    if (prop === 'age') {
      if (typeof value !== 'number') {
        throw new TypeError('Age must be a number');
      }
      if (value < 0 || value > 150) {
        throw new RangeError('Age must be between 0 and 150');
      }
    }
    
    if (prop === 'email') {
      if (!value.includes('@')) {
        throw new Error('Invalid email format');
      }
    }
    
    target[prop] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.age = 30; // OK
person.email = 'alice@example.com'; // OK
// person.age = 'thirty'; // TypeError
// person.email = 'invalid'; // Error

// Reactive 数据绑定
function reactive(obj, onChange) {
  return new Proxy(obj, {
    set(target, prop, value) {
      const oldValue = target[prop];
      target[prop] = value;
      onChange(prop, value, oldValue);
      return true;
    }
  });
}

const state = reactive({ count: 0, text: '' }, (prop, value) => {
  console.log(`${prop} changed to ${value}`);
});

state.count = 1; // 输出: count changed to 1
state.text = 'Hello'; // 输出: text changed to Hello
```

### 迭代器与生成器

```javascript
// 自定义迭代器
class FibonacciSequence {
  constructor(limit) {
    this.limit = limit;
    this.current = 0;
    this.next = 1;
  }
  
  [Symbol.iterator]() {
    return this;
  }
  
  next() {
    if (this.current > this.limit) {
      return { done: true };
    }
    
    const value = this.current;
    this.current = this.next;
    this.next = value + this.next;
    
    return { value, done: false };
  }
}

const fib = new FibonacciSequence(100);
for (const num of fib) {
  console.log(num); // 0, 1, 1, 2, 3, 5, 8, ...
}

// 生成器函数
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2
console.log(gen.next().value); // 3
console.log(gen.next().done); // true

// 无限生成器
function* infiniteSequence() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

function* fibonacciGenerator() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

// 生成器组合
function* filter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) {
      yield item;
    }
  }
}

function* map(iterable, transform) {
  for (const item of iterable) {
    yield transform(item);
  }
}

const numbers = [1, 2, 3, 4, 5, 6];
const evens = filter(numbers, n => n % 2 === 0);
const doubled = map(evens, n => n * 2);

console.log([...doubled]); // [4, 8, 12]
```

### 尾递归优化

```javascript
// 尾递归函数（某些引擎支持 TCO）
'use strict';

function factorial(n, accumulator = 1) {
  if (n <= 1) {
    return accumulator;
  }
  return factorial(n - 1, n * accumulator); // 尾调用
}

// trampoline 模式处理尾递归
function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

const factorialTrampoline = trampoline(function factorial(n, acc = 1) {
  if (n <= 1) return acc;
  return () => factorial(n - 1, n * acc);
});

console.log(factorialTrampoline(100000)); // 不会栈溢出
```

### WeakRef 与 FinalizationRegistry

```javascript
// WeakRef 持有对象的弱引用
let obj = { name: 'Alice' };
const weakRef = new WeakRef(obj);

console.log(weakRef.deref()?.name); // 'Alice'

obj = null; // 清除强引用

// 垃圾回收后，deref() 返回 undefined
setTimeout(() => {
  console.log(weakRef.deref()); // undefined
}, 100);

// FinalizationRegistry 监听对象被垃圾回收
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Object with ${heldValue} was garbage collected`);
});

let user = { name: 'Bob' };
registry.register(user, user.name);

user = null; // 稍后会被垃圾回收

// 缓存示例
const cache = new Map();
const cacheRef = new FinalizationRegistry(() => {
  // 清理过期的缓存
  const now = Date.now();
  for (const [key, { expiry }] of cache) {
    if (expiry < now) {
      cache.delete(key);
    }
  }
});

function getCached(key, computeFn, ttl = 60000) {
  const cached = cache.get(key);
  if (cached && cached.expiry > Date.now()) {
    return cached.value;
  }
  
  const value = computeFn();
  cache.set(key, {
    value,
    expiry: Date.now() + ttl
  });
  
  cacheRef.register(value, key);
  return value;
}
```

---

## JavaScript 设计模式

### 单例模式

```javascript
const Singleton = (function() {
  let instance = null;
  
  function createInstance() {
    const obj = new Object('I am the singleton');
    return obj;
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// ES6 类版本
class SingletonClass {
  constructor() {
    if (SingletonClass.instance) {
      return SingletonClass.instance;
    }
    SingletonClass.instance = this;
    this.data = {};
  }
  
  set(key, value) {
    this.data[key] = value;
  }
  
  get(key) {
    return this.data[key];
  }
}

// 闭包版本
function SingletonFn() {
  const instance = {
    data: {},
    set(key, value) {
      this.data[key] = value;
    },
    get(key) {
      return this.data[key];
    }
  };
  
  return function() {
    return instance;
  };
}
```

### 工厂模式

```javascript
// 简单工厂
class User {
  constructor(name, type) {
    this.name = name;
    this.type = type;
  }
  
  greet() {
    console.log(`Hi, I am ${this.name}`);
  }
}

class Admin extends User {
  greet() {
    console.log(`Admin ${this.name} at your service`);
  }
}

class Guest extends User {
  greet() {
    console.log(`Welcome, I am ${this.name}`);
  }
}

function UserFactory() {
  return {
    create(type, name) {
      switch (type) {
        case 'admin':
          return new Admin(name, type);
        case 'guest':
          return new Guest(name, type);
        default:
          return new User(name, type);
      }
    }
  };
}

const factory = UserFactory();
const admin = factory.create('admin', 'Alice');
const guest = factory.create('guest', 'Bob');
```

### 观察者模式

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
    return () => this.off(event, listener);
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
  
  off(event, listener) {
    if (!this.events[event]) return;
    this.events[event] = this.events[event].filter(l => l !== listener);
  }
  
  emit(event, ...args) {
    if (!this.events[event]) return;
    this.events[event].forEach(listener => listener(...args));
  }
}

const emitter = new EventEmitter();

emitter.on('message', (msg) => console.log('Handler 1:', msg));
const off = emitter.on('message', (msg) => console.log('Handler 2:', msg));

emitter.emit('message', 'Hello!'); // 两个处理器都会执行

off(); // 移除第二个处理器
emitter.emit('message', 'World!'); // 只有第一个处理器执行
```

### 中介者模式

```javascript
class ChatRoom {
  constructor() {
    this.users = new Map();
  }
  
  register(user) {
    this.users.set(user.id, user);
    user.room = this;
    this.broadcast('system', `${user.name} has joined`);
  }
  
  unregister(user) {
    this.users.delete(user.id);
    this.broadcast('system', `${user.name} has left`);
  }
  
  broadcast(sender, message) {
    this.users.forEach(user => {
      if (user !== sender) {
        user.receive(sender, message);
      }
    });
  }
  
  direct(sender, receiverId, message) {
    const receiver = this.users.get(receiverId);
    if (receiver) {
      receiver.receive(sender, message, true);
    }
  }
}

class User {
  constructor(id, name) {
    this.id = id;
    this.name = name;
    this.room = null;
  }
  
  receive(sender, message, isDirect = false) {
    const prefix = isDirect ? '(DM)' : '';
    console.log(`${prefix}${sender.name} -> ${this.name}: ${message}`);
  }
  
  send(message) {
    this.room.broadcast(this, message);
  }
  
  sendTo(receiverId, message) {
    this.room.direct(this, receiverId, message);
  }
}

const room = new ChatRoom();
const alice = new User(1, 'Alice');
const bob = new User(2, 'Bob');

room.register(alice);
room.register(bob);

alice.send('Hello everyone!'); // 广播给 Bob
alice.sendTo(2, 'Hi Bob!'); // 只发给 Bob
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| MDN Web Docs | https://developer.mozilla.org/ |
| JavaScript.info | https://javascript.info/ |
| TC39 Proposals | https://github.com/tc39/proposals |
| Can I Use | https://caniuse.com/ |
| Node.js Documentation | https://nodejs.org/docs/ |
| V8 JavaScript Engine | https://v8.dev/ |
| JavaScript Weekly | https://javascriptweekly.com/ |

---

> [!TIP]
> JavaScript 的生态系统极其丰富，合理利用现代 ES2024+ 特性和工具链，可以构建出高性能、可靠的应用程序。
