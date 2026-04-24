# JavaScript 核心与 DOM 完全指南

> [!NOTE]
> 本文档深入讲解 JavaScript 语言核心（ES2024+ 新特性、执行上下文、异步编程、模块系统）与 DOM/BOM API 及浏览器工作原理，是彻底理解前端运行时的必备知识。

---

## 目录

1. [[#核心概念与设计哲学]]
2. [[#JavaScript 语言核心]]
3. [[#执行上下文与作用域]]
4. [[#异步编程深度解析]]
5. [[#DOM 与 BOM API]]
6. [[#事件系统]]
7. [[#浏览器工作原理]]
8. [[#现代浏览器 API]]

---

## 核心概念与设计哲学

### JavaScript 的诞生与演进

JavaScript 由 Brendan Eich 于 1995 年在网景公司用 10 天时间创造，最初名为 Mocha，随后改名为 LiveScript，最终定名为 JavaScript。这个命名决定出于市场营销考虑——当时 Java 语言正值流行，蹭热度可以借势推广。这段历史解释了为什么 JavaScript 和 Java 名字相似，但本质上是完全不同的语言。

```
JavaScript 关键时间线
├── 1995: 诞生 (Brendan Eich, Netscape)
├── 1997: ECMAScript 标准化 (ES1)
├── 2009: ES5 发布，strict mode 引入
├── 2015: ES6/ES2015 里程碑式更新 (classes, modules, promises, arrow functions)
├── 2016-2024: 年度发布节奏，逐年新增特性
└── 2025: 持续演进中
```

### JavaScript 的设计哲学

JavaScript 的设计哲学体现了实用主义与妥协的产物，但这些"妥协"反而成为其成功的关键：

**1. 弱类型与类型强制**
JavaScript 是弱类型语言，允许隐式类型转换。这种设计降低了入门门槛，但也带来了著名的"比较运算符"陷阱：

```javascript
// 经典陷阱
console.log([] == false);  // true
console.log('' == false);  // true
console.log('0' == false); // true

// 安全比较
console.log([] === false); // false
console.log('' === false); // false

// Object.is 严格相等
console.log(Object.is(NaN, NaN)); // true
console.log(Object.is(+0, -0));  // false
```

**2. 原型继承而非类继承**
JavaScript 采用原型链继承，这种设计在 1995 年是创新之举，提供了更灵活的动态继承机制：

```javascript
// 原型链示意
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}

// 设置原型链
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.speak = function() {
  console.log(`${this.name} barks`);
};

const dog = new Dog('Buddy', 'Labrador');
dog.speak(); // "Buddy barks"
console.log(dog instanceof Dog);  // true
console.log(dog instanceof Animal); // true
```

**3. 函数是一等公民**
函数在 JavaScript 中是对象，可以作为参数传递、返回、赋值给变量。这种设计使得函数式编程成为可能：

```javascript
// 函数作为参数
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);

// 函数作为返回值
const createAdder = (x) => (y) => x + y;
const add5 = createAdder(5);
console.log(add5(10)); // 15

// 函数作为值
const operations = {
  add: (a, b) => a + b,
  multiply: (a, b) => a * b,
};
```

**4. 单线程与事件循环**
JavaScript 从设计之初就是单线程的。这种设计简化了并发模型，避免了死锁和竞态条件，使得 DOM 操作变得简单。但这也意味着长时间运行的代码会阻塞 UI：

```javascript
// 阻塞主线程
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 3000) {
    // 模拟长时间计算
  }
  return 'Done';
}

// 非阻塞替代
async function nonBlockingOperation() {
  return new Promise(resolve => {
    setTimeout(() => resolve('Done'), 3000);
  });
}
```

### 语言特性全景图

```
JavaScript 语言特性矩阵
├── 类型系统
│   ├── 原始类型: number, string, boolean, null, undefined, symbol, bigint
│   ├── 对象类型: Object, Array, Function, Date, RegExp, Map, Set, WeakMap, WeakSet
│   └── 类型转换: 隐式转换 vs 显式转换
│
├── 作用域
│   ├── 全局作用域: var 声明，window/globalThis 挂载
│   ├── 函数作用域: var 声明
│   ├── 块级作用域: let, const 声明
│   └── 词法作用域: 闭包基础
│
├── 执行模型
│   ├── 调用栈: 函数调用管理
│   ├── 执行上下文: this 绑定
│   ├── 作用域链: 变量查找路径
│   └── 闭包: 函数记忆效应
│
├── 面向对象
│   ├── 原型链: 继承机制
│   ├── 类语法: ES6 class (语法糖)
│   ├── 属性描述符: 数据属性 vs 访问器属性
│   └── Symbol: 唯一属性键
│
└── 异步编程
    ├── 回调函数: 最早模式
    ├── Promise: ES6 标准
    ├── async/await: ES2017
    └── 生成器: 函数暂停与恢复
```

---

## JavaScript 语言核心

### ES2024-2025 新特性

**Array.groupBy 与 Map.groupBy (ES2024)**

这两个方法为数组分组提供了原生支持，告别了传统的 reduce 方法：

```javascript
// 按属性分组
const inventory = [
  { name: 'asparagus', type: 'vegetables', quantity: 5 },
  { name: 'bananas', type: 'fruit', quantity: 3 },
  { name: 'goat', type: 'meat', quantity: 1 },
  { name: 'cherries', type: 'fruit', quantity: 12 },
  { name: 'fish', type: 'meat', quantity: 8 },
];

// Object.groupBy - 返回普通对象
const byType = Object.groupBy(inventory, item => item.type);
console.log(byType);
// {
//   vegetables: [{ name: 'asparagus', ... }],
//   fruit: [{ name: 'bananas', ... }, { name: 'cherries', ... }],
//   meat: [{ name: 'goat', ... }, { name: 'fish', ... }]
// }

// Map.groupBy - 返回 Map
const byQuantity = Map.groupBy(inventory, item => {
  if (item.quantity < 5) return 'low';
  if (item.quantity < 10) return 'medium';
  return 'high';
});
console.log(byQuantity);
// Map { 'low' => [...], 'medium' => [...], 'high' => [...] }

// 实际应用场景
const salesData = [
  { region: 'North', product: 'A', revenue: 1000 },
  { region: 'South', product: 'B', revenue: 2000 },
  { region: 'North', product: 'B', revenue: 1500 },
  { region: 'East', product: 'A', revenue: 3000 },
  { region: 'South', product: 'A', revenue: 2500 },
];

// 按区域分组并汇总
const byRegion = Object.groupBy(salesData, item => item.region);
const regionSummary = Object.entries(byRegion).reduce((acc, [region, items]) => {
  acc[region] = items.reduce((sum, item) => sum + item.revenue, 0);
  return acc;
}, {});
console.log(regionSummary); // { North: 2500, South: 4500, East: 3000 }
```

**Promise.withResolvers (ES2024)**

这个新 API 解决了长期困扰开发者的"先有鸡还是先有蛋"问题——必须在 Promise 内部才能访问 resolve/reject：

```javascript
// 传统方式（问题示例）
function fetchWithTimeout(url, timeout) {
  return new Promise((resolve, reject) => {
    // 必须在 Promise 内部处理逻辑
    fetch(url)
      .then(response => resolve(response))
      .catch(error => reject(error));
    
    setTimeout(() => reject(new Error('Timeout')), timeout);
  });
}

// Promise.withResolvers 方式
function createEventEmitter() {
  const { promise, resolve, reject } = Promise.withResolvers();
  
  return {
    promise,
    emit: (event) => resolve({ event, timestamp: Date.now() }),
    error: (err) => reject(err),
  };
}

// 典型应用：封装不支持 Promise 的 API
class HTTPClient {
  static get(url) {
    const { promise, resolve, reject } = Promise.withResolvers();
    
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`HTTP ${xhr.status}: ${xhr.statusText}`));
      }
    };
    
    xhr.onerror = () => reject(new Error('Network error'));
    xhr.ontimeout = () => reject(new Error('Request timeout'));
    
    xhr.send();
    
    return promise;
  }
}

// WebSocket 封装示例
function createWebSocketConnection(url) {
  const { promise, resolve, reject } = Promise.withResolvers();
  
  const ws = new WebSocket(url);
  
  ws.onopen = () => resolve(ws);
  ws.onerror = (error) => reject(error);
  
  return {
    connection: promise,
    socket: ws,
  };
}

// 使用
async function main() {
  const { connection, socket } = createWebSocketConnection('wss://example.com');
  
  socket.onmessage = (event) => {
    console.log('Received:', event.data);
  };
  
  const ws = await connection;
  ws.send('Hello, WebSocket!');
}
```

**ArrayBuffer.transfer (ES2024)**

ArrayBuffer 的转移机制类似于 ArrayBuffer 的"移动"而非"复制"，这对于高性能场景至关重要：

```javascript
// 基本用法
const buffer = new ArrayBuffer(8);
const view = new Int32Array(buffer);
view[0] = 42;
view[1] = 100;

console.log(buffer.byteLength); // 8
console.log(view[0]); // 42

// transfer - 原始 buffer 被清空
const transferred = buffer.transfer();
console.log(buffer.byteLength); // 0
console.log(transferred.byteLength); // 8
console.log(transferred === buffer); // false

// 可以指定新大小
const resized = buffer.transfer(16);
console.log(resized.byteLength); // 16

// transferToFixedLength - 不允许增大
const fixed = transferred.transferToFixedLength(4);
console.log(fixed.byteLength); // 4

// 性能对比
const LARGE_SIZE = 100 * 1024 * 1024; // 100MB

function benchmarkTransfer() {
  const original = new ArrayBuffer(LARGE_SIZE);
  const view = new Uint8Array(original);
  for (let i = 0; i < LARGE_SIZE; i++) {
    view[i] = i % 256;
  }
  
  // 计时 transfer
  const start = performance.now();
  const transferred = original.transfer();
  const transferTime = performance.now() - start;
  
  // 计时 slice (旧方法)
  const original2 = new ArrayBuffer(LARGE_SIZE);
  const view2 = new Uint8Array(original2);
  for (let i = 0; i < LARGE_SIZE; i++) {
    view2[i] = i % 256;
  }
  
  const start2 = performance.now();
  const sliced = original2.slice(0);
  const sliceTime = performance.now() - start2;
  
  console.log(`transfer: ${transferTime.toFixed(2)}ms`);
  console.log(`slice: ${sliceTime.toFixed(2)}ms`);
  console.log(`transfer 比 slice 快 ${(sliceTime / transferTime).toFixed(2)}x`);
}

// 实际应用：Web Workers 通信
function sendToWorker(data) {
  // 创建 buffer 并复制数据
  const buffer = new ArrayBuffer(data.byteLength);
  new Uint8Array(buffer).set(new Uint8Array(data));
  
  // 转移所有权给 worker
  worker.postMessage({ buffer }, [buffer]);
}

// worker 端接收
worker.onmessage = (event) => {
  const { buffer } = event.data;
  // buffer 现在归 worker 所有，主线程无法再访问
};
```

**管道操作符 |> (Stage 3)**

管道操作符提供函数式编程中的管道式调用风格，使代码更易读：

```javascript
// 字符串处理管道
const processText = (text) => text
  |> (t => t.trim())
  |> (t => t.toLowerCase())
  |> (t => t.split(/\s+/))
  |> (arr => arr.filter(word => word.length > 2))
  |> (arr => arr.map(word => word.charAt(0).toUpperCase() + word.slice(1)))
  |> (arr => arr.join('-'));

console.log(processText('  HELLO WORLD JAVASCRIPT IS AWESOME  '));
// 输出: "Hello-World-Javascript-Awesome"

// 数据转换管道
const users = [
  { name: 'Alice', age: 25, scores: [80, 90, 85] },
  { name: 'Bob', age: 30, scores: [70, 75, 80] },
  { name: 'Charlie', age: 22, scores: [95, 88, 92] },
];

const averageScores = users
  |> (arr => arr.filter(u => u.age >= 25))
  |> (arr => arr.map(u => ({
    name: u.name,
    average: u.scores.reduce((a, b) => a + b) / u.scores.length
  })))
  |> (arr => arr.sort((a, b) => b.average - a.average))
  |> (arr => arr.map(u => `${u.name}: ${u.average.toFixed(1)}`));

console.log(averageScores);
// ["Charlie: 91.7", "Alice: 85.0"]

// DOM 操作管道
const createButton = (text, className) => {
  const button = document.createElement('button');
  button.textContent = text;
  button.className = className;
  return button;
};

const button = null
  |> (() => document.createElement('button'))
  |> (btn => { btn.className = 'btn btn-primary'; return btn; })
  |> (btn => { btn.textContent = 'Click Me'; return btn; })
  |> (btn => { btn.onclick = () => console.log('Clicked!'); return btn; });

// 错误处理管道
const safeJsonParse = (json) =>
  json
    |> ((str) => { try { return { ok: true, data: JSON.parse(str) }; } catch { return { ok: false, error: 'Invalid JSON' }; } })
    |> (result => result.ok ? result.data : null);

// 函数组合器
const pipe = (...fns) => (initial) => fns.reduce((acc, fn) => fn(acc), initial);
const compose = (...fns) => (initial) => fns.reduceRight((acc, fn) => fn(acc), initial);

// 使用管道操作符创建管道
const processNumber = (num) => num
  |> (n => n * 2)
  |> (n => n + 10)
  |> (n => Math.sqrt(n));

console.log(processNumber(5)); // sqrt((5 * 2) + 10) = sqrt(20) ≈ 4.47
```

**装饰器 (Stage 3/2024)**

装饰器提供了一种元编程能力，允许在不修改类或函数本身的情况下增强其行为：

```javascript
// 基础装饰器工厂
function log(target, context) {
  if (context.kind === 'method') {
    return function(...args) {
      console.log(`Calling ${String(context.name)} with`, args);
      const start = performance.now();
      const result = target.apply(this, args);
      console.log(`${String(context.name)} returned`, result);
      console.log(`Time: ${(performance.now() - start).toFixed(2)}ms`);
      return result;
    };
  }
  return target;
}

// 防抖装饰器
function debounce(wait) {
  return function(target, context) {
    if (context.kind === 'method') {
      let timeoutId;
      return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
          target.apply(this, args);
        }, wait);
      };
    }
    return target;
  };
}

// 节流装饰器
function throttle(wait) {
  return function(target, context) {
    if (context.kind === 'method') {
      let lastTime = 0;
      return function(...args) {
        const now = Date.now();
        if (now - lastTime >= wait) {
          lastTime = now;
          target.apply(this, args);
        }
      };
    }
    return target;
  };
}

// 缓存装饰器
function memoize(target, context) {
  if (context.kind === 'method') {
    const cache = new Map();
    return function(...args) {
      const key = JSON.stringify(args);
      if (cache.has(key)) {
        console.log(`Cache hit for ${context.name}`);
        return cache.get(key);
      }
      const result = target.apply(this, args);
      cache.set(key, result);
      return result;
    };
  }
  return target;
}

// 验证装饰器
function validate(validator) {
  return function(target, context) {
    if (context.kind === 'method') {
      return function(...args) {
        const validationErrors = validator(...args);
        if (validationErrors.length > 0) {
          throw new Error(`Validation failed: ${validationErrors.join(', ')}`);
        }
        return target.apply(this, args);
      };
    }
    return target;
  };
}

// 自动绑定装饰器
function autoBind(target, context) {
  if (context.kind === 'method') {
    return {
      get() {
        return target.bind(this);
      },
    };
  }
  return target;
}

// 类装饰器
function sealed(constructor) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

function logger(target) {
  console.log(`Class ${target.name} defined`);
  return target;
}

// 应用装饰器
class Calculator {
  @log
  @memoize
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }

  @log
  @debounce(300)
  onInput(value) {
    console.log('Processing:', value);
  }

  @log
  @throttle(1000)
  onResize() {
    console.log('Window resized');
  }

  @validate
  setAge(age) {
    const errors = [];
    if (typeof age !== 'number') errors.push('Age must be a number');
    if (age < 0 || age > 150) errors.push('Age must be between 0 and 150');
    return errors;
  }

  @autoBind
  handleClick() {
    console.log('Click handler, this:', this);
  }

  @sealed
  @logger
  static create() {
    return new Calculator();
  }
}

// 装饰器组合
const composeDecorators = (...decorators) => (target, context) => {
  return decorators.reduce((acc, decorator) => decorator(acc, context), target);
};

class DataService {
  @composeDecorators(log, memoize, debounce(500))
  fetchData(id) {
    // 实现
  }
}

// 参数装饰器
function logParameter(target, context) {
  if (context.kind === 'method') {
    const method = target;
    return function(...args) {
      console.log(`Calling ${context.name} with args:`, args);
      return method.apply(this, args);
    };
  }
  return target;
}

class ApiService {
  @log
  fetchUser(@logParameter id, @logParameter options) {
    // 实现
  }
}
```

**迭代器与生成器深度**

迭代器和生成器是 JavaScript 中强大的惰性求值工具：

```javascript
// 自定义迭代器
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const step = this.step;
    const end = this.end;
    
    return {
      next() {
        if ((step > 0 && current >= end) || (step < 0 && current <= end)) {
          return { done: true };
        }
        const value = current;
        current += step;
        return { value, done: false };
      },
    };
  }
}

const range = new Range(0, 10, 2);
for (const num of range) {
  console.log(num); // 0, 2, 4, 6, 8
}

// 生成器函数
function* fibonacciGenerator() {
  let [prev, curr] = [0, 1];
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

const fib = fibonacciGenerator();
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3

// 生成器可选参数
function* generatorWithParams() {
  let result = yield 1;
  console.log('Received:', result);
  result = yield result + 2;
  console.log('Received:', result);
  return result;
}

const gen = generatorWithParams();
console.log(gen.next());        // { value: 1, done: false }
console.log(gen.next(10));      // Received: 10, { value: 12, done: false }
console.log(gen.next(20));      // Received: 20, { value: undefined, done: true }

// yield* 委托
function* gen1() {
  yield 1;
  yield 2;
}

function* gen2() {
  yield 3;
  yield 4;
}

function* combined() {
  yield* gen1();
  yield* gen2();
  yield 5;
}

console.log([...combined()]); // [1, 2, 3, 4, 5]

// 异步生成器
async function* asyncDataFetcher(urls) {
  for (const url of urls) {
    const response = await fetch(url);
    const data = await response.json();
    yield data;
  }
}

async function main() {
  const urls = ['/api/users', '/api/posts', '/api/comments'];
  
  for await (const data of asyncDataFetcher(urls)) {
    console.log('Received:', data);
  }
}

// 无限序列
function* naturalNumbers() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

function* primes() {
  const isPrime = (n) => {
    if (n < 2) return false;
    for (let i = 2; i <= Math.sqrt(n); i++) {
      if (n % i === 0) return false;
    }
    return true;
  };
  
  let n = 2;
  while (true) {
    if (isPrime(n)) yield n;
    n++;
  }
}

const take = (n, iterable) => {
  const result = [];
  const iterator = iterable[Symbol.iterator]();
  for (let i = 0; i < n; i++) {
    const { value, done } = iterator.next();
    if (done) break;
    result.push(value);
  }
  return result;
};

console.log(take(10, primes())); // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

**Symbol 与 Symbol Registry 高级应用**

```javascript
// Symbol 创建
const sym1 = Symbol('description');
const sym2 = Symbol('description');
sym1 === sym2;  // false（每个 Symbol 唯一）

// Symbol 注册表（全局共享）
const shared = Symbol.for('shared');  // 创建或获取全局注册表中的 Symbol
const shared2 = Symbol.for('shared');
shared === shared2;  // true

// 查找键
Symbol.keyFor(shared);  // 'shared'

// Well-known Symbols - 定义对象行为
const obj = {
  [Symbol.iterator]: function*() { yield 1; yield 2; },
  [Symbol.toStringTag]: 'MyObject',
  [Symbol.hasInstance]: (instance) => instance.isMyObject,
};

// Symbol 属性遍历
Object.getOwnPropertySymbols(obj);  // 获取所有 Symbol 属性
Reflect.ownKeys(obj);               // 包括 Symbol 的所有键

// 自定义迭代器
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }

  // 前序遍历迭代器
  *[Symbol.iterator]() {
    yield this.value;
    if (this.left) yield* this.left;
    if (this.right) yield* this.right;
  }

  // 深度优先
  *[Symbol.iterator]() {
    const stack = [this];
    while (stack.length > 0) {
      const node = stack.pop();
      yield node.value;
      if (node.right) stack.push(node.right);
      if (node.left) stack.push(node.left);
    }
  }
}

// 模拟 Python 的 __repr__
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }

  [Symbol.toStringTag] = 'Point';
  
  [Symbol.toPrimitive](hint) {
    if (hint === 'string') {
      return `Point(${this.x}, ${this.y})`;
    }
    if (hint === 'number') {
      return Math.sqrt(this.x ** 2 + this.y ** 2);
    }
    return { x: this.x, y: this.y };
  }

  valueOf() {
    return Math.sqrt(this.x ** 2 + this.y ** 2);
  }
}

// Symbol.match, Symbol.split, Symbol.replace
const startsWithA = {
  [Symbol.match](str) {
    return str.startsWith('A') ? str.slice(0, 3) : null;
  }
};

const str = 'Apple';
console.log(str.match(startsWithA)); // 'App'

// Symbol.replace 自定义替换行为
class TemplateString {
  constructor(variables) {
    this.variables = variables;
  }

  [Symbol.replace](str, replacement) {
    let result = str;
    for (const [key, value] of Object.entries(this.variables)) {
      result = result.replace(new RegExp(`\\{${key}\\}`, 'g'), value);
    }
    if (typeof replacement === 'function') {
      return replacement(result);
    }
    return result;
  }
}

const template = new TemplateString({ name: 'Alice', age: 30 });
console.log('Hello {name}, you are {age}'.replace(template));
// "Hello Alice, you are 30"

// Symbol 私有属性模式
const #privateData = Symbol('private');

class SecureData {
  constructor(data) {
    this[#privateData] = data;
  }

  getData() {
    return this[#privateData];
  }
}

const secure = new SecureData({ secret: 'value' });
console.log(Object.keys(secure)); // []
console.log(secure.getData()); // { secret: 'value' }
console.log(secure[#privateData]); // { secret: 'value' } (在有权限访问 Symbol 的代码中)
```

**Proxy 与 Reflect 深度解析**

```javascript
// Proxy：拦截对象操作
const target = { message: 'Hello' };
const handler = {
  get(target, prop, receiver) {
    console.log(`Getting ${String(prop)}`);
    return Reflect.get(target, prop, receiver);
  },
  set(target, prop, value, receiver) {
    console.log(`Setting ${String(prop)} to ${value}`);
    return Reflect.set(target, prop, value, receiver);
  },
  has(target, prop) {
    console.log(`Checking for ${String(prop)}`);
    return Reflect.has(target, prop);
  },
  deleteProperty(target, prop) {
    console.log(`Deleting ${String(prop)}`);
    return Reflect.deleteProperty(target, prop);
  },
};

const proxy = new Proxy(target, handler);
proxy.message;        // 触发 get
proxy.message = 'Hi'; // 触发 set

// 验证器示例
const validator = {
  set(obj, prop, value) {
    if (prop === 'age') {
      if (typeof value !== 'number') {
        throw new TypeError('Age must be a number');
      }
      if (value < 0 || value > 150) {
        throw new RangeError('Invalid age');
      }
    }
    obj[prop] = value;
    return true;
  },
};

const person = new Proxy({}, validator);
person.age = 30;    // OK
person.age = -1;    // RangeError
person.age = 'old'; // TypeError

// 函数拦截
const createLogger = (fn) => {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      console.log(`Calling ${target.name} with`, args);
      const result = Reflect.apply(target, thisArg, args);
      console.log(`Result:`, result);
      return result;
    },
  });
};

// 完整拦截器演示
const createTrapHandler = (name) => ({
  get(target, prop, receiver) {
    console.log(`[${name}] GET ${String(prop)}`);
    const value = Reflect.get(target, prop, receiver);
    
    // 如果是函数，包装它
    if (typeof value === 'function') {
      return new Proxy(value, {
        apply(fn, thisArg, args) {
          console.log(`[${name}] CALL ${String(prop)}(${args.join(', ')})`);
          return Reflect.apply(fn, thisArg, args);
        }
      });
    }
    
    return value;
  },
  
  set(target, prop, value, receiver) {
    console.log(`[${name}] SET ${String(prop)} = ${JSON.stringify(value)}`);
    return Reflect.set(target, prop, value, receiver);
  },
  
  has(target, prop) {
    console.log(`[${name}] HAS ${String(prop)}`);
    return Reflect.has(target, prop);
  },
  
  apply(target, thisArg, args) {
    console.log(`[${name}] APPLY(${args.join(', ')})`);
    return Reflect.apply(target, thisArg, args);
  },
  
  construct(target, args) {
    console.log(`[${name}] CONSTRUCT(${args.join(', ')})`);
    return Reflect.construct(target, args);
  },
});

// 实际应用：响应式系统
function reactive(obj) {
  const handlers = new Map();
  
  return new Proxy(obj, {
    get(target, prop, receiver) {
      let value = Reflect.get(target, prop, receiver);
      
      if (typeof value === 'object' && value !== null) {
        value = reactive(value);
      }
      
      return value;
    },
    
    set(target, prop, value, receiver) {
      const oldValue = Reflect.get(target, prop, receiver);
      const result = Reflect.set(target, prop, value, receiver);
      
      if (result && oldValue !== value) {
        const callbacks = handlers.get(prop) || [];
        callbacks.forEach(cb => cb(value, oldValue));
      }
      
      return result;
    },
    
    on(trap, handler) {
      const traps = handlers.get(trap) || [];
      traps.push(handler);
      handlers.set(trap, traps);
    }
  });
}

const state = reactive({ count: 0, user: { name: 'Alice' } });

state.on('count', (newVal, oldVal) => {
  console.log(`count changed: ${oldVal} -> ${newVal}`);
});

state.count = 1; // 触发回调
state.count = 2; // 触发回调
state.user.name = 'Bob'; // 嵌套对象也会是响应式的

// 可撤销的 Proxy
const { proxy: revocableProxy, revoke } = Proxy.revocable(target, handler);
revoke(); // 撤销后访问 proxy 会抛出 TypeError

// 使用 Reflect 实现元编程
class Meta {
  static defineProperty(obj, prop, descriptor) {
    return Reflect.defineProperty(obj, prop, descriptor);
  }
  
  static getOwnPropertyDescriptor(obj, prop) {
    return Reflect.getOwnPropertyDescriptor(obj, prop);
  }
  
  static preventExtensions(obj) {
    return Reflect.preventExtensions(obj);
  }
}
```

---

## 执行上下文与作用域

### 执行上下文类型

```
JavaScript 执行上下文
├── Global Context（全局上下文）
│   └── this === globalThis（浏览器中为 window）
│
├── Function Context（函数上下文）
│   └── 每次函数调用创建新的上下文
│
└── Eval Context（eval 上下文）
    └── 不推荐使用
```

### 作用域链

```javascript
// 作用域链示例
const global = 'G';

function outer() {
  const outerVar = 'O';
  
  function middle() {
    const middleVar = 'M';
    
    function inner() {
      const innerVar = 'I';
      // 作用域链：inner → middle → outer → global
      console.log(global, outerVar, middleVar, innerVar);
    }
    
    inner();
  }
  
  middle();
}

outer();
// 输出: G O M I

// 词法作用域 vs 动态作用域
// JavaScript 采用词法作用域（编写时决定）
// 动态作用域在运行时决定（如 Bash）
```

### 闭包深度解析

```javascript
// 闭包：函数记住其词法环境
function createCounter() {
  let count = 0;  // 私有变量
  
  return {
    increment() { count++; return count; },
    decrement() { count--; return count; },
    getCount() { return count; },
  };
}

const counter = createCounter();
counter.increment();  // 1
counter.increment();  // 2
counter.getCount();   // 2

// 经典闭包循环问题（var 版本）
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);  // 输出: 3, 3, 3
}

// 解决方案1：let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);  // 输出: 0, 1, 2
}

// 解决方案2：IIFE
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i);
}

// 解决方案3：bind
for (var i = 0; i < 3; i++) {
  setTimeout(console.log.bind(null, i), 100);
}

// 内存泄漏示例（闭包持有大对象）
function badPattern() {
  const largeData = new Array(1000000);
  const element = document.getElementById('button');
  
  element.onclick = () => {
    console.log(largeData.length);  // largeData 无法被 GC
  };
}

// 正确模式：显式释放
function goodPattern() {
  const largeData = new Array(1000000);
  const element = document.getElementById('button');
  
  element.onclick = () => {
    console.log(largeData.length);
  };
  
  // 完成后清理
  element.onclick = null;
  largeData.length = 0;
}
```

### this 绑定机制

```javascript
// this 的四种绑定规则

// 1. 默认绑定（严格模式下为 undefined）
function showThis() {
  console.log(this);  // globalThis（浏览器中为 window）
}

// 2. 隐式绑定
const obj = {
  name: 'Alice',
  showThis() {
    console.log(this);  // obj
  },
};
obj.showThis();

// 3. 显式绑定
const person = { name: 'Bob' };
function greet(lang) {
  console.log(`${this.name} speaks ${lang}`);
}
greet.call(person, 'English');    // Bob speaks English
greet.apply(person, ['French']);  // Bob speaks French
const boundGreet = greet.bind(person, 'Spanish');
boundGreet();                     // Bob speaks Spanish

// 4. new 绑定
function Person(name) {
  this.name = name;
}
const p = new Person('Charlie');  // this 指向新创建的对象

// 箭头函数的 this
const arrow = () => {
  console.log(this);  // 继承外层 this（词法绑定）
};

// 优先级：new > bind/call/apply > 隐式 > 默认
```

---

## 异步编程深度解析

### Promise 深度

```javascript
// Promise 状态机
// pending → fulfilled 或 pending → rejected
// 一旦 settled，不再改变

// Promise 静态方法
Promise.all([
  fetch('/api/users'),
  fetch('/api/posts'),
])
  .then(([users, posts]) => Promise.all([
    users.json(),
    posts.json(),
  ]))
  .then(([usersData, postsData]) => {
    console.log(usersData, postsData);
  })
  .catch(err => {
    console.error('任一请求失败:', err);
  });

// Promise.allSettled - 等待所有 Promise 完成
const results = await Promise.allSettled([
  fetch('/api/a'),
  fetch('/api/b'),
  fetch('/api/c'),
]);

results.forEach((result, i) => {
  if (result.status === 'fulfilled') {
    console.log(`Request ${i} succeeded:`, result.value);
  } else {
    console.log(`Request ${i} failed:`, result.reason);
  }
});

// Promise.race - 首个完成
const fast = await Promise.race([
  fetch('/api/fast').then(r => r.json()),
  new Promise((_, reject) => setTimeout(() => reject(new Error('Timeout')), 3000)),
]);

// Promise.any - 首个成功（忽略 rejections）
const firstSuccess = await Promise.any([
  fetch('/api/primary'),
  fetch('/api/backup'),
]);

// Promise.withResolvers (ES2024)
const { promise, resolve, reject } = Promise.withResolvers();
// 替代：new Promise((resolve, reject) => {...})
```

### async/await 进阶

```javascript
// 并行 vs 顺序
async function sequential() {
  const a = await fetchA();  // 等待 A 完成
  const b = await fetchB();  // 然后才请求 B
  return [a, b];
}

async function parallel() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);  // 同时请求
  return [a, b];
}

// 错误处理模式
async function robust() {
  try {
    const data = await riskyOperation();
    return data;
  } catch (error) {
    if (error instanceof NetworkError) {
      // 处理网络错误
      return fallbackData;
    }
    if (error instanceof ValidationError) {
      // 处理验证错误
      throw error;  // 重新抛出
    }
    throw error;  // 其他错误重新抛出
  } finally {
    cleanup();  // 无论成功失败都执行
  }
}

// 顶层 await（ES2022，模块内）
const data = await fetchData();  // 无需包装在 async 函数中

// 串行调用优化
const urls = ['a', 'b', 'c'];

// 慢：串行
const results = [];
for (const url of urls) {
  const data = await fetch(url);
  results.push(data);
}

// 快：Promise.all + map
const results = await Promise.all(urls.map(url => fetch(url)));

// 慢但节省内存：流式处理
const results = [];
for (const url of urls) {
  const data = await fetch(url);
  results.push(data);
  // 立即处理，释放内存
  process(data);
  results.pop();
}
```

### 事件循环（Event Loop）

```javascript
// 事件循环执行顺序（浏览器环境）

// 1. 同步代码（Call Stack）
console.log('1');

// 2. Microtask Queue（微任务队列，优先级高）
Promise.resolve().then(() => console.log('3'));
queueMicrotask(() => console.log('3.5'));

// 3. Macrotask Queue（宏任务队列）
setTimeout(() => console.log('4'), 0);
requestAnimationFrame(() => console.log('RAF'));
requestIdleCallback(() => console.log('RIC'));

// 输出顺序：1, 3, 3.5, 4, RAF, RIC

// 完整事件循环 tick
async function eventLoopTick() {
  // 1. 执行同步代码
  console.log('Sync start');
  
  // 2. 执行所有微任务
  await Promise.resolve();
  
  // 3. 渲染更新（浏览器）
  await new Promise(resolve => requestAnimationFrame(resolve));
  
  // 4. 执行一个宏任务
  await new Promise(resolve => setTimeout(resolve, 0));
  
  console.log('Sync end');
}

// 避免阻塞主线程
async function heavyComputation() {
  const CHUNK_SIZE = 1000;
  const TOTAL = 1000000;
  
  for (let i = 0; i < TOTAL; i += CHUNK_SIZE) {
    // 处理一块数据
    processChunk(i, i + CHUNK_SIZE);
    
    // 让出主线程
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}

// Web Worker 分离计算
const worker = new Worker('compute.js');
worker.postMessage({ data: bigArray });
worker.onmessage = (e) => {
  console.log('Result:', e.data);
};
```

---

## DOM 与 BOM API

### DOM 节点操作

```javascript
// 选择器
const el = document.querySelector('.card');           // 单个
const els = document.querySelectorAll('.card');       // 多个（NodeList）
const byId = document.getElementById('id');          // ID 选择
const byClass = document.getElementsByClassName('card'); // HTMLCollection

// 遍历
const parent = el.parentElement;
const children = el.children;
const firstChild = el.firstElementChild;
const nextSibling = el.nextElementSibling;

// 创建
const div = document.createElement('div');
div.textContent = 'Hello';
div.innerHTML = '<span>HTML</span>';
div.setAttribute('data-id', '123');
div.className = 'card active';

// 插入
parent.appendChild(div);           // 末尾插入
parent.insertBefore(div, reference); // 参考节点前插入
parent.append(div, 'Text', anotherEl); // 末尾插入多个（可文本）
parent.prepend(div);              // 开头插入

// 现代插入 API
parent.insertAdjacentHTML('beforeend', '<div>HTML</div>');
parent.insertAdjacentElement('afterbegin', div);

// 替换和删除
parent.replaceChild(newEl, oldEl);
parent.removeChild(el);
el.remove();  // 现代 API（无需 parent）

// 属性操作
el.getAttribute('href');
el.setAttribute('href', 'https://...');
el.removeAttribute('href');
el.hasAttribute('href');

// 数据属性
el.dataset.userId;      // 获取 data-user-id
el.dataset.userId = '456'; // 设置
delete el.dataset.userId;  // 删除
```

### BOM（浏览器对象模型）

```javascript
// Window 对象
window.innerWidth;       // 视口宽度
window.innerHeight;      // 视口高度
window.devicePixelRatio; // 设备像素比

// 浏览器信息
navigator.userAgent;
navigator.language;
navigator.platform;

// 位置信息
window.location.href;      // 当前 URL
window.location.pathname;  // 路径
window.location.search;    // 查询参数

// 历史记录
history.pushState({ page: 1 }, '', '/page1');
history.replaceState({ page: 2 }, '', '/page2');
window.addEventListener('popstate', (e) => {
  console.log('Navigated:', e.state);
});

// 定时器
const timerId = setTimeout(() => {}, 1000);
clearTimeout(timerId);

const intervalId = setInterval(() => {}, 1000);
clearInterval(intervalId);

// requestAnimationFrame
function animate() {
  // 浏览器下一次重绘前调用
  requestAnimationFrame(animate);
}

// requestIdleCallback（空闲时执行）
requestIdleCallback((deadline) => {
  console.log('Remaining time:', deadline.timeRemaining());
  if (deadline.didTimeout) {
    // 超时了，需要立即执行
  }
});
```

---

## 事件系统

### 事件处理

```javascript
// 添加事件监听
el.addEventListener('click', handler, options);
el.addEventListener('click', handler, { capture: false });
el.addEventListener('click', handler, { once: true });  // 只执行一次
el.addEventListener('click', handler, { passive: true }); // 不阻止默认

// 移除事件
el.removeEventListener('click', handler);

// 事件对象
el.addEventListener('click', (e) => {
  e.target;                    // 触发事件的元素
  e.currentTarget;             // 绑定事件的元素
  e.type;                     // 事件类型
  e.bubbles;                  // 是否冒泡
  e.cancelable;               // 是否可取消
  e.preventDefault();          // 阻止默认行为
  e.stopPropagation();         // 停止冒泡
  e.stopImmediatePropagation(); // 停止同元素其他监听器
  
  // 坐标信息
  e.clientX; e.clientY;        // 视口坐标
  e.pageX; e.pageY;           // 页面坐标
  e.offsetX; e.offsetY;       // 元素内坐标
  
  // 键盘事件
  e.key;                      // 按下的键
  e.code;                     // 物理键码
  e.ctrlKey; e.shiftKey; e.altKey; e.metaKey; // 修饰键
});

// 事件委托（事件代理）
document.querySelector('.list').addEventListener('click', (e) => {
  if (e.target.matches('.item')) {
    console.log('Clicked:', e.target.textContent);
  }
});

// 自定义事件
const customEvent = new CustomEvent('myEvent', {
  detail: { message: 'Hello' },
  bubbles: true,
  cancelable: true,
});
el.dispatchEvent(customEvent);

// 事件监听器选项
const controller = new AbortController();
el.addEventListener('click', handler, { signal: controller.signal });
controller.abort();  // 移除所有该 signal 的监听器
```

### 常见事件类型

```javascript
// 鼠标事件
'click'        // 单击
'dblclick'     // 双击
'contextmenu'  // 右键菜单
'mousedown'   // 鼠标按下
'mouseup'     // 鼠标释放
'mouseenter'  // 进入（不冒泡）
'mouseleave'  // 离开（不冒泡）
'mouseover'   // 经过（冒泡）
'mouseout'    // 离开（冒泡）
'mousemove'   // 移动

// 键盘事件
'keydown'     // 按下
'keypress'    // 字符键（已废弃）
'keyup'       // 释放

// 表单事件
'focus'       // 获得焦点
'blur'       // 失去焦点
'input'      // 输入时触发
'change'     // 值改变且失去焦点
'submit'     // 表单提交
'reset'      // 表单重置

// 文档/窗口事件
'DOMContentLoaded'  // DOM 解析完成
'load'               // 所有资源加载完成
'error'              // 资源加载失败
'resize'             // 窗口大小改变
'scroll'             // 滚动
```

---

## 浏览器工作原理

### 渲染流水线

```
JavaScript → Style → Layout → Paint → Composite
    ↑           ↑        ↑       ↑         ↑
  执行      计算样式   计算布局  绘制图层  合成层
```

```javascript
// 强制重排（Reflow）的操作
element.style.width = '100px';  // 读取会触发
const width = element.offsetWidth; // 强制重排
element.style.width = '200px';  // 再次触发

// 优化：批量读写
// 读 -> 读 -> 读 -> 写 -> 写 -> 写
// 而非：读 -> 写 -> 读 -> 写 -> 读 -> 写

// 批量 DOM 操作
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = i;
  fragment.appendChild(div);
}
container.appendChild(fragment);  // 一次插入

// 使用 display: none 隐藏后批量操作
container.style.display = 'none';
// ... 批量操作
container.style.display = '';

// will-change 提示浏览器优化
.animated {
  will-change: transform, opacity;
  transform: translateZ(0);  // 强制创建合成层
}
```

### 关键渲染路径

```javascript
// 优化 CSS 加载
<!-- preload 关键 CSS -->
<link rel="preload" href="critical.css" as="style" onload="this.rel='stylesheet'">

<!-- async 加载非关键 CSS -->
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">

// JavaScript 加载策略
<!-- 阻塞渲染 -->
<script src="app.js"></script>

<!-- 异步加载，不阻塞 HTML 解析 -->
<script async src="app.js"></script>

<!-- 延迟执行，HTML 解析后执行 -->
<script defer src="app.js"></script>
<!-- defer 按文档顺序执行，async 随机顺序 -->

// 模块脚本（自动 defer）
<script type="module" src="app.js"></script>
```

---

## 现代浏览器 API

### Web APIs 全景

```
浏览器 API 分类
├── 存储
│   ├── localStorage / sessionStorage
│   ├── IndexedDB
│   ├── Cache API（Service Worker）
│   └── File System Access API
│
├── 网络
│   ├── Fetch API
│   ├── WebSocket
│   ├── Server-Sent Events
│   └── WebRTC
│
├── 媒体
│   ├── Canvas API
│   ├── WebGL / WebGPU
│   ├── MediaDevices API
│   └── Web Audio API
│
├── 文件与硬件
│   ├── File API
│   ├── File System Access API
│   ├── Web Bluetooth
│   └── Web Serial
│
└── 通信
    ├── BroadcastChannel
    ├── SharedWorker
    └── MessageChannel
```

### Fetch API 与请求取消

```javascript
// 基本用法
const response = await fetch('/api/data');
const data = await response.json();

// 带选项
const response = await fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${token}`,
  },
  body: JSON.stringify({ name: 'Alice' }),
  credentials: 'include',  // 发送 cookies
  cache: 'no-cache',         // 缓存策略
});

// AbortController 取消请求
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 5000);

try {
  const response = await fetch('/api/data', {
    signal: controller.signal,
  });
  clearTimeout(timeoutId);
  const data = await response.json();
} catch (error) {
  if (error.name === 'AbortError') {
    console.log('Request cancelled');
  }
}

// 流式响应
const response = await fetch('/api/stream');
const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value);
  process(chunk);
}
```

### Intersection Observer

```javascript
// 懒加载检测
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;  // 加载真实图片
        observer.unobserve(img);     // 停止观察
      }
    });
  },
  {
    root: null,             // 视口
    rootMargin: '0px',     // 扩大检测区域
    threshold: 0.1,         // 10% 可见时触发
  }
);

// 无限滚动
const infiniteObserver = new IntersectionObserver(
  async (entries) => {
    const entry = entries[0];
    if (entry.isIntersecting && !isLoading) {
      isLoading = true;
      const nextPage = await fetchNextPage();
      appendItems(nextPage);
      isLoading = false;
    }
  },
  { rootMargin: '200px' }  // 提前 200px 触发
);

infiniteObserver.observe(loadingIndicator);
```

### ResizeObserver

```javascript
// 监听元素尺寸变化
const observer = new ResizeObserver((entries) => {
  entries.forEach((entry) => {
    const { width, height } = entry.contentRect;
    console.log(`Element size: ${width}x${height}`);
    
    // 根据尺寸调整布局
    if (width < 300) {
      entry.target.classList.add('compact');
    } else {
      entry.target.classList.remove('compact');
    }
  });
});

observer.observe(container);
```

### 性能优化技巧

#### DOM 操作优化

```javascript
// 1. 批量 DOM 操作
// 坏: 多次重排
for (let i = 0; i < 100; i++) {
  element.style.width = i + 'px';
}

// 好: 使用 transform (只触发重绘)
for (let i = 0; i < 100; i++) {
  element.style.transform = `translateX(${i}px)`;
}

// 2. DocumentFragment 批量插入
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = i;
  fragment.appendChild(div);
}
container.appendChild(fragment); // 只触发一次重排

// 3. 缓存 DOM 查询结果
const cachedElement = document.getElementById('myElement');
// 多次使用 cachedElement 而非重复查询

// 4. 使用类名切换而非内联样式
element.classList.add('active');
element.classList.remove('active');
element.classList.toggle('active');

// 5. CSS contain 属性
.container {
  contain: layout style paint;
}

// 6. will-change 提示浏览器优化
.animated {
  will-change: transform, opacity;
}
```

#### JavaScript 执行优化

```javascript
// 1. 防抖 (Debounce)
function debounce(func, wait) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), wait);
  };
}

// 2. 节流 (Throttle)
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

// 3. 事件委托
document.querySelector('.list').addEventListener('click', (e) => {
  if (e.target.matches('.item')) {
    handleItemClick(e.target);
  }
});

// 4. 虚拟列表（长列表优化）
class VirtualList {
  constructor(options) {
    this.itemHeight = options.itemHeight;
    this.container = options.container;
    this.items = options.items;
    this.visibleCount = Math.ceil(this.container.clientHeight / this.itemHeight);
  }

  render() {
    const scrollTop = this.container.scrollTop;
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = Math.min(startIndex + this.visibleCount + 2, this.items.length);

    // 只渲染可见项
    const visibleItems = this.items.slice(startIndex, endIndex);
    // 使用 padding 或 transform 定位
  }
}

// 5. Web Worker 处理大数据
// 在 worker.js 中
self.onmessage = ({ data }) => {
  const result = heavyComputation(data);
  self.postMessage(result);
};

// 主线程
const worker = new Worker('worker.js');
worker.postMessage(largeData);
worker.onmessage = ({ data }) => {
  console.log('Result:', data);
};
```

#### 内存管理优化

```javascript
// 1. 避免内存泄漏
class MemoryLeakExample {
  constructor() {
    this.data = new Array(1000000);

    // 坏: 闭包持有大对象引用
    this.element = document.getElementById('button');
    this.element.onclick = () => {
      console.log(this.data.length);
    };
  }

  // 好: 显式清理
  destroy() {
    this.element.onclick = null;
    this.data = null;
  }
}

// 2. WeakMap 用于对象缓存
const cache = new WeakMap();
function expensiveOperation(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  const result = computeExpensive(obj);
  cache.set(obj, result);
  return result;
}

// 3. 及时清理事件监听器
const controller = new AbortController();
element.addEventListener('click', handler, { signal: controller.signal });
// 清理时
controller.abort();

// 4. 使用 requestAnimationFrame 批量更新
let pendingUpdates = false;
const updates = [];

function scheduleUpdate(update) {
  updates.push(update);
  if (!pendingUpdates) {
    pendingUpdates = true;
    requestAnimationFrame(() => {
      flushUpdates(updates);
      updates.length = 0;
      pendingUpdates = false;
    });
  }
}
```

### 与同类技术对比

#### JavaScript vs TypeScript

| 特性 | JavaScript | TypeScript |
|------|------------|------------|
| 类型系统 | 动态类型 | 静态类型（可选） |
| 编译 | 无需编译 | 需要编译 |
| 错误检测 | 运行时 | 编译时 + 运行时 |
| 学习曲线 | 低 | 中等 |
| 开发效率 | 高（快速原型） | 中（需要定义类型） |
| 代码可读性 | 依赖文档 | 类型即文档 |
| 重构支持 | 困难 | 强大 |
| 生态系统 | 广泛 | 与 JS 共享 |

```javascript
// JavaScript 原生写法
function processUser(user) {
  return user.name.toUpperCase();
}

// TypeScript 类型安全写法
interface User {
  name: string;
  age?: number;
}

function processUser(user: User): string {
  return user.name.toUpperCase();
}
```

#### JavaScript vs Python (DOM 操作方面)

| 方面 | JavaScript | Python |
|------|------------|--------|
| 运行环境 | 浏览器/Node.js | 服务器端 |
| DOM 访问 | 原生支持 | 需 Selenium/BeautifulSoup |
| 异步编程 | Promise/async-await | asyncio |
| 性能 | JIT 优化 | CPython |
| 生态系统 | npm | pip |

#### JavaScript vs Go (并发方面)

| 方面 | JavaScript | Go |
|------|------------|-----|
| 并发模型 | 事件循环 + 异步 | Goroutines + Channels |
| 线程 | 单线程（Worker 除外） | 多线程原生支持 |
| 内存共享 | 受限 | 支持 |
| 学习曲线 | 低 | 中等 |
| 适用场景 | I/O 密集 | CPU 密集 |

### 常见问题与解决方案

#### 问题 1: 浮点数精度

```javascript
// 问题
0.1 + 0.2 === 0.3; // false

// 解决方案 1: toFixed
parseFloat((0.1 + 0.2).toFixed(2)); // 0.3

// 解决方案 2: 整数运算
function add(a, b, decimals = 2) {
  const factor = Math.pow(10, decimals);
  return (a * factor + b * factor) / factor;
}
add(0.1, 0.2); // 0.3

// 解决方案 3: decimal.js 库
import Decimal from 'decimal.js';
new Decimal(0.1).plus(0.2).equals(0.3); // true
```

#### 问题 2: 异步错误处理

```javascript
// 问题: 未捕获的 Promise 错误
async function bad() {
  fetch('/api'); // 错误不会被捕获
}

// 解决方案: 全局错误处理
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
});

// 或者在每个 async 函数中添加 try-catch
async function good() {
  try {
    const response = await fetch('/api');
    return await response.json();
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// 使用 error boundaries (React 概念) 模式
class ErrorBoundary {
  constructor() {
    this.error = null;
  }

  static getDerivedStateFromError(error) {
    return { error };
  }

  componentDidCatch(error, info) {
    console.error('Error:', error, info);
  }

  render() {
    if (this.state.error) {
      return <div>Something went wrong</div>;
    }
    return this.props.children;
  }
}
```

#### 问题 3: 循环引用导致内存泄漏

```javascript
// 问题
function createLeak() {
  const bigObject = new Array(1000000);
  const elements = document.querySelectorAll('.listener');

  elements.forEach(el => {
    el.addEventListener('click', () => {
      console.log(bigObject.length); // bigObject 永远不会被回收
    });
  });
}

// 解决方案
function preventLeak() {
  const bigObject = new Array(1000000);
  const elements = document.querySelectorAll('.listener');

  // 使用 WeakMap 关联
  const dataMap = new WeakMap();
  elements.forEach(el => {
    dataMap.set(el, { bigObject }); // 弱引用

    el.addEventListener('click', () => {
      const data = dataMap.get(el);
      console.log(data.bigObject.length);
    });
  });

  // 或者显式清理
  return () => {
    elements.forEach(el => {
      el.removeEventListener('click', handler);
    });
  };
}
```

#### 问题 4: 深拷贝循环引用对象

```javascript
// 问题: JSON.parse(JSON.stringify()) 无法处理循环引用
const obj = { a: 1 };
obj.self = obj;

// 解决方案 1: 手动实现
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);

  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);

  for (const key of Object.keys(obj)) {
    clone[key] = deepClone(obj[key], hash);
  }

  return clone;
}

// 解决方案 2: structuredClone (原生支持循环引用)
const cloned = structuredClone(originalObj);

// 解决方案 3: lodash.cloneDeep
import { cloneDeep } from 'lodash';
const cloned = cloneDeep(obj);
```

#### 问题 5: 跨域请求

```javascript
// 问题: CORS 错误
fetch('https://other-domain.com/api'); // 可能被阻止

// 解决方案 1: CORS 代理
const response = await fetch('https://cors-proxy.com/https://other-domain.com/api');

// 解决方案 2: JSONP (仅 GET)
function jsonp(url, callback) {
  const callbackName = 'jsonpCallback';
  window[callbackName] = (data) => {
    delete window[callbackName];
    document.body.removeChild(script);
    callback(data);
  };

  const script = document.createElement('script');
  script.src = `${url}?callback=${callbackName}`;
  document.body.appendChild(script);
}

// 解决方案 3: Server-Side Proxy
// 自己的服务器转发请求

// 解决方案 4: 避免跨域的替代方案
// 使用 postMessage, MessageChannel 进行同源通信
```

#### 问题 6: this 丢失

```javascript
// 问题
class Timer {
  constructor() {
    this.seconds = 0;
    setInterval(this.tick, 1000); // this 丢失
  }

  tick() {
    this.seconds++;
  }
}

// 解决方案 1: 箭头函数
setInterval(() => this.tick(), 1000);

// 解决方案 2: bind
setInterval(this.tick.bind(this), 1000);

// 解决方案 3: 类字段语法
class Timer {
  this.seconds = 0;
  this.tick = () => {
    this.seconds++;
  };
}

// 解决方案 4: 在 constructor 中绑定
constructor() {
  this.tick = this.tick.bind(this);
}
```

---

## 深度实战案例

### 完整的响应式系统实现

```javascript
// 基于 Proxy 的响应式系统
class Reactive {
  constructor(target) {
    this._target = target;
    this._handlers = new Map();
    this._proxy = this._createReactive(target);
    return this._proxy;
  }

  _createReactive(target) {
    const handler = {
      get(obj, prop, receiver) {
        const value = Reflect.get(obj, prop, receiver);

        // 递归处理嵌套对象
        if (value !== null && typeof value === 'object') {
          return new Reactive(value)._proxy;
        }

        // 触发 get 订阅
        Reactive.currentTracker?.track(prop);
        return value;
      },

      set(obj, prop, value, receiver) {
        const oldValue = obj[prop];
        const result = Reflect.set(obj, prop, value, receiver);

        if (oldValue !== value) {
          // 触发变更通知
          Reactive.currentTracker?.notify(prop, value, oldValue);
        }

        return result;
      },

      deleteProperty(obj, prop) {
        const oldValue = obj[prop];
        const result = Reflect.deleteProperty(obj, prop);

        if (result) {
          Reactive.currentTracker?.notify(prop, undefined, oldValue);
        }

        return result;
      }
    };

    return new Proxy(target, handler);
  }

  // 订阅特定属性
  watch(property, callback) {
    if (!this._handlers.has(property)) {
      this._handlers.set(property, new Set());
    }
    this._handlers.get(property).add(callback);

    return () => {
      this._handlers.get(property)?.delete(callback);
    };
  }

  // 全局追踪器
  static currentTracker = null;

  static effect(fn) {
    const tracker = {
      _deps: new Map(),
      _dirty: new Set(),

      track(property) {
        if (!this._deps.has(property)) {
          this._deps.set(property, new Set());
        }
        this._deps.get(property).add(this);
      },

      notify(property, newValue, oldValue) {
        const subscribers = this._deps.get(property);
        if (subscribers) {
          subscribers.forEach(sub => {
            sub._dirty.add(property);
          });
        }
      },

      run(fn) {
        Reactive.currentTracker = this;
        const result = fn();
        Reactive.currentTracker = null;
        return result;
      },

      getDirty() {
        const dirty = Array.from(this._dirty);
        this._dirty.clear();
        return dirty;
      }
    };

    return tracker.run(fn);
  }
}

// 使用示例
const state = new Reactive({
  user: { name: 'Alice', age: 25 },
  posts: [],
  theme: 'dark'
});

// 监视特定属性
state.watch('theme', (newVal, oldVal) => {
  console.log(`Theme changed: ${oldVal} -> ${newVal}`);
  document.body.className = newVal;
});

// 使用 effect
Reactive.effect(() => {
  const name = state.user.name;
  console.log(`User name: ${name}`);
});

// 自动追踪依赖
function renderUser() {
  const html = `
    <div class="user">
      <h1>${state.user.name}</h1>
      <p>Age: ${state.user.age}</p>
    </div>
  `;
  document.getElementById('app').innerHTML = html;
}

Reactive.effect(renderUser);

// 修改触发更新
state.user.name = 'Bob';  // 自动触发 renderUser
state.theme = 'light';     // 触发主题更新
```

### 虚拟 DOM 简化实现

```javascript
// 简化版虚拟 DOM
class VNode {
  constructor(tag, props, children) {
    this.tag = tag;
    this.props = props || {};
    this.children = children || [];
  }

  // 创建 DOM 元素
  createElement() {
    const el = document.createElement(this.tag);

    // 设置属性
    Object.entries(this.props).forEach(([key, value]) => {
      if (key.startsWith('on')) {
        const eventName = key.slice(2).toLowerCase();
        el.addEventListener(eventName, value);
      } else if (key === 'className') {
        el.className = value;
      } else if (key === 'style' && typeof value === 'object') {
        Object.assign(el.style, value);
      } else {
        el.setAttribute(key, value);
      }
    });

    // 递归创建子元素
    this.children.forEach(child => {
      const childEl = child instanceof VNode
        ? child.createElement()
        : document.createTextNode(String(child));
      el.appendChild(childEl);
    });

    return el;
  }

  // 对比新旧节点
  static diff(oldNode, newNode) {
    const patches = [];

    if (!newNode) {
      patches.push({ type: 'REMOVE', node: oldNode });
    } else if (typeof newNode === 'string' || typeof newNode === 'number') {
      if (oldNode !== newNode) {
        patches.push({ type: 'TEXT', node: newNode });
      }
    } else if (oldNode.tag !== newNode.tag) {
      patches.push({ type: 'REPLACE', oldNode, newNode });
    } else {
      // 比较属性
      const propsDiff = VNode.diffProps(oldNode.props, newNode.props);
      if (Object.keys(propsDiff).length > 0) {
        patches.push({ type: 'PROPS', props: propsDiff });
      }

      // 递归比较子节点
      const childrenLength = Math.max(
        oldNode.children.length,
        newNode.children.length
      );

      for (let i = 0; i < childrenLength; i++) {
        patches.push(...VNode.diff(
          oldNode.children[i],
          newNode.children[i]
        ));
      }
    }

    return patches;
  }

  static diffProps(oldProps, newProps) {
    const diff = {};
    const allKeys = new Set([...Object.keys(oldProps), ...Object.keys(newProps)]);

    allKeys.forEach(key => {
      if (oldProps[key] !== newProps[key]) {
        diff[key] = newProps[key];
      }
    });

    return diff;
  }
}

// 创建虚拟节点
function h(tag, props, ...children) {
  return new VNode(tag, props, children);
}

// 简化渲染器
class Renderer {
  constructor(container) {
    this.container = container;
    this.currentNode = null;
    this.currentDOM = null;
  }

  render(vnode) {
    if (this.currentDOM) {
      const patches = VNode.diff(this.currentNode, vnode);
      this.applyPatches(this.currentDOM, patches);
    } else {
      this.currentDOM = vnode.createElement();
      this.container.appendChild(this.currentDOM);
    }
    this.currentNode = vnode;
  }

  applyPatches(dom, patches) {
    patches.forEach(patch => {
      switch (patch.type) {
        case 'TEXT':
          dom.textContent = patch.node;
          break;
        case 'REPLACE':
          dom.parentNode.replaceChild(
            patch.newNode.createElement(),
            dom
          );
          break;
        case 'REMOVE':
          dom.parentNode.removeChild(dom);
          break;
        case 'PROPS':
          Object.entries(patch.props).forEach(([key, value]) => {
            if (key.startsWith('on')) {
              const eventName = key.slice(2).toLowerCase();
              dom.addEventListener(eventName, value);
            } else {
              dom.setAttribute(key, value);
            }
          });
          break;
      }
    });
  }
}

// 使用示例
const renderer = new Renderer(document.getElementById('app'));

function App({ count }) {
  return h('div', { className: 'app' },
    h('h1', null, `Count: ${count}`),
    h('button', {
      onClick: () => updateCount(count + 1)
    }, 'Increment'),
    h('button', {
      onClick: () => updateCount(count - 1)
    }, 'Decrement')
  );
}

let count = 0;

function updateCount(newCount) {
  count = newCount;
  renderer.render(App({ count }));
}

// 初始化
renderer.render(App({ count }));
```

### 完整的状态管理 + 中间件系统

```javascript
// Redux 风格的中间件系统
class Store {
  constructor(reducer, initialState, middlewares = []) {
    this._reducer = reducer;
    this._state = initialState;
    this._listeners = [];
    this._middlewares = middlewares.map(m => m(this));
  }

  getState() {
    return this._state;
  }

  dispatch(action) {
    // 通过中间件链处理
    let dispatch = (action) => {
      this._state = this._reducer(this._state, action);
      this._notify();
    };

    // 反向应用中间件
    for (let i = this._middlewares.length - 1; i >= 0; i--) {
      dispatch = this._middlewares[i](dispatch);
    }

    dispatch(action);
  }

  subscribe(listener) {
    this._listeners.push(listener);
    return () => {
      this._listeners = this._listeners.filter(l => l !== listener);
    };
  }

  _notify() {
    this._listeners.forEach(listener => listener(this._state));
  }

  // 替换中间件
  replaceMiddleware(middlewares) {
    this._middlewares = middlewares.map(m => m(this));
  }
}

// 中间件工厂函数
const loggerMiddleware = (store) => (next) => (action) => {
  console.log('Before:', store.getState());
  console.log('Action:', action);
  const result = next(action);
  console.log('After:', store.getState());
  return result;
};

const thunkMiddleware = (store) => (next) => (action) => {
  if (typeof action === 'function') {
    return action(store.dispatch, store.getState);
  }
  return next(action);
};

const persistMiddleware = (store) => (next) => (action) => {
  const result = next(action);
  try {
    localStorage.setItem('state', JSON.stringify(store.getState()));
  } catch (e) {
    console.error('Failed to persist state:', e);
  }
  return result;
};

const analyticsMiddleware = (store) => (next) => (action) => {
  if (action.type?.startsWith('USER_')) {
    // 发送分析数据
    analytics.track(action.type, action.payload);
  }
  return next(action);
};

// 使用示例
const rootReducer = (state, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET_USER':
      return { ...state, user: action.payload };
    default:
      return state;
  }
};

const initialState = {
  count: 0,
  user: null
};

const store = new Store(
  rootReducer,
  initialState,
  [thunkMiddleware, persistMiddleware, analyticsMiddleware, loggerMiddleware]
);

// 订阅状态
store.subscribe((state) => {
  console.log('State changed:', state);
});

// 分发动作
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'SET_USER', payload: { name: 'Alice' } });

// Thunk：可以分发函数
store.dispatch((dispatch, getState) => {
  fetch('/api/user')
    .then(res => res.json())
    .then(user => dispatch({ type: 'SET_USER', payload: user }));
});
```

### 完整的防抖与节流实现

```javascript
// 防抖（Debounce）
function debounce(func, wait, options = {}) {
  let timeoutId;
  let lastArgs;
  let lastThis;
  const { leading = false, trailing = true, maxWait = null } = options;

  function invokeFunc() {
    return func.apply(lastThis, lastArgs);
  }

  function debounced(...args) {
    lastArgs = args;
    lastThis = this;

    if (timeoutId) {
      clearTimeout(timeoutId);
    }

    if (leading && !timeoutId) {
      invokeFunc();
    }

    if (maxWait !== null && !timeoutId) {
      timeoutId = setTimeout(() => {
        timeoutId = null;
        if (trailing) {
          invokeFunc();
        }
      }, maxWait);
    }

    timeoutId = setTimeout(() => {
      timeoutId = null;
      if (trailing) {
        invokeFunc();
      }
    }, wait);
  }

  debounced.cancel = () => {
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
  };

  debounced.flush = () => {
    if (timeoutId) {
      invokeFunc();
      debounced.cancel();
    }
  };

  return debounced;
}

// 节流（Throttle）
function throttle(func, wait, options = {}) {
  let timeoutId;
  let lastArgs;
  let lastThis;
  let lastCallTime = 0;
  const { leading = true, trailing = true } = options;

  function invokeFunc() {
    return func.apply(lastThis, lastArgs);
  }

  function throttled(...args) {
    const now = Date.now();
    lastArgs = args;
    lastThis = this;

    if (leading && lastCallTime === 0) {
      lastCallTime = now;
      invokeFunc();
    }

    const remaining = wait - (now - lastCallTime);

    if (remaining <= 0 || remaining > wait) {
      if (timeoutId) {
        clearTimeout(timeoutId);
        timeoutId = null;
      }
      lastCallTime = now;
      invokeFunc();
    } else if (!timeoutId && trailing) {
      timeoutId = setTimeout(() => {
        lastCallTime = Date.now();
        timeoutId = null;
        invokeFunc();
      }, remaining);
    }
  }

  throttled.cancel = () => {
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
    lastCallTime = 0;
  };

  return throttled;
}

// 进阶版本：带类型提示和 Promise 支持
function debounceAsync(func, wait, options = {}) {
  let timeoutId;
  let pendingPromise;

  const { leading = false, trailing = true } = options;

  async function debounced(...args) {
    return new Promise((resolve, reject) => {
      lastArgs = args;

      if (timeoutId) {
        clearTimeout(timeoutId);
      }

      if (leading && !timeoutId) {
        try {
          const result = await func.apply(this, args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }

      timeoutId = setTimeout(async () => {
        timeoutId = null;
        if (trailing) {
          try {
            const result = await func.apply(this, args);
            resolve(result);
          } catch (error) {
            reject(error);
          }
        }
      }, wait);
    });
  }

  debounced.cancel = () => {
    if (timeoutId) {
      clearTimeout(timeoutId);
      timeoutId = null;
    }
    if (pendingPromise) {
      pendingPromise.reject(new Error('Cancelled'));
    }
  };

  return debounced;
}

// 使用示例
const handleSearch = debounceAsync(async (query) => {
  const response = await fetch(`/api/search?q=${query}`);
  return response.json();
}, 300);

// 带 maxWait 的防抖（适合搜索建议）
const handleTyping = debounce((text) => {
  showSuggestions(text);
}, 300, { maxWait: 1000 });

// 节流用于 resize/scroll
const handleScroll = throttle((e) => {
  console.log('Scroll position:', window.scrollY);
}, 100, { leading: true, trailing: true });
```

### 完整的缓存系统实现

```javascript
// LRU 缓存
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) {
      return undefined;
    }

    // 移到末尾（最近使用）
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // 删除最久未使用的
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }

  has(key) {
    return this.cache.has(key);
  }

  delete(key) {
    return this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  get size() {
    return this.cache.size;
  }
}

// TTL 缓存
class TTLCache {
  constructor(ttl = 60000, maxSize = 100) {
    this.ttl = ttl;
    this.maxSize = maxSize;
    this.cache = new Map();
    this.timers = new Map();
  }

  get(key) {
    const entry = this.cache.get(key);
    if (!entry) return undefined;

    if (Date.now() > entry.expires) {
      this.delete(key);
      return undefined;
    }

    return entry.value;
  }

  set(key, value, customTtl = this.ttl) {
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.delete(firstKey);
    }

    const expires = Date.now() + customTtl;
    this.cache.set(key, { value, expires });

    this.timers.set(key, setTimeout(() => {
      this.delete(key);
    }, customTtl));
  }

  delete(key) {
    if (this.timers.has(key)) {
      clearTimeout(this.timers.get(key));
      this.timers.delete(key);
    }
    return this.cache.delete(key);
  }

  clear() {
    this.timers.forEach(timer => clearTimeout(timer));
    this.timers.clear();
    this.cache.clear();
  }
}

// 带异步加载的缓存
class AsyncCache {
  constructor(options = {}) {
    this.cache = new Map();
    this.pending = new Map();
    this.loader = options.loader;
    this.ttl = options.ttl || 60000;
  }

  async get(key) {
    const cached = this.cache.get(key);

    if (cached && Date.now() - cached.timestamp < this.ttl) {
      return cached.value;
    }

    if (this.pending.has(key)) {
      return this.pending.get(key);
    }

    const promise = (async () => {
      try {
        const value = await this.loader(key);
        this.cache.set(key, { value, timestamp: Date.now() });
        return value;
      } finally {
        this.pending.delete(key);
      }
    })();

    this.pending.set(key, promise);
    return promise;
  }

  invalidate(key) {
    if (key) {
      this.cache.delete(key);
    } else {
      this.cache.clear();
    }
  }
}

// 使用示例
const userCache = new AsyncCache({
  loader: async (userId) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  },
  ttl: 300000 // 5 分钟
});

// 数据缓存
const dataCache = new LRUCache(100);
dataCache.set('key1', { data: 'value' });
const data = dataCache.get('key1');
```

---

## 高级设计模式

### 发布-订阅模式完整实现

```javascript
class PubSub {
  constructor() {
    this.topics = new Map();
    this.h очередь = new Map();
    this.subscriberId = 0;
  }

  subscribe(topic, handler, context = null) {
    if (!this.topics.has(topic)) {
      this.topics.set(topic, new Map());
    }

    const id = ++this.subscriberId;
    const subscriber = {
      id,
      handler,
      context,
      wrappedHandler: context ? handler.bind(context) : handler
    };

    this.topics.get(topic).set(id, subscriber);

    // 返回取消订阅函数
    return () => this.unsubscribe(topic, id);
  }

  once(topic, handler, context = null) {
    const id = ++this.subscriberId;

    const onceHandler = (...args) => {
      handler.apply(context, args);
      this.unsubscribe(topic, id);
    };

    if (!this.topics.has(topic)) {
      this.topics.set(topic, new Map());
    }

    this.topics.get(topic).set(id, {
      id,
      handler: onceHandler,
      context,
      wrappedHandler: context ? onceHandler.bind(context) : onceHandler,
      once: true
    });

    return () => this.unsubscribe(topic, id);
  }

  publish(topic, ...args) {
    const subscribers = this.topics.get(topic);
    if (!subscribers) return this;

    const toRemove = [];

    subscribers.forEach((subscriber, id) => {
      try {
        subscriber.wrappedHandler(...args);
        if (subscriber.once) {
          toRemove.push(id);
        }
      } catch (error) {
        console.error(`Error in subscriber ${id} for topic ${topic}:`, error);
      }
    });

    toRemove.forEach(id => subscribers.delete(id));

    return this;
  }

  unsubscribe(topic, id) {
    const subscribers = this.topics.get(topic);
    if (subscribers) {
      subscribers.delete(id);
      if (subscribers.size === 0) {
        this.topics.delete(topic);
      }
    }
    return this;
  }

  unsubscribeAll(topic) {
    if (topic) {
      this.topics.delete(topic);
    } else {
      this.topics.clear();
    }
    return this;
  }

  hasSubscribers(topic) {
    return this.topics.has(topic) && this.topics.get(topic).size > 0;
  }

  subscriberCount(topic) {
    return this.topics.get(topic)?.size || 0;
  }

  getTopics() {
    return Array.from(this.topics.keys());
  }
}

// 全局事件总线
const eventBus = new PubSub();

// 命名空间支持
class NamespacedPubSub {
  constructor() {
    this.pubsub = new PubSub();
  }

  subscribe(namespace, topic, handler, context) {
    return this.pubsub.subscribe(`${namespace}:${topic}`, handler, context);
  }

  once(namespace, topic, handler, context) {
    return this.pubsub.once(`${namespace}:${topic}`, handler, context);
  }

  publish(namespace, topic, ...args) {
    return this.pubsub.publish(`${namespace}:${topic}`, ...args);
  }

  unsubscribe(namespace, topic, id) {
    return this.pubsub.unsubscribe(`${namespace}:${topic}`, id);
  }
}

// 使用示例
const globalBus = new NamespacedPubSub();

// 用户模块
globalBus.subscribe('user', 'login', (user) => {
  console.log('User logged in:', user);
});

globalBus.subscribe('user', 'logout', () => {
  console.log('User logged out');
});

// 通知模块
globalBus.subscribe('notification', 'new', (message) => {
  showNotification(message);
});

// 发布事件
globalBus.publish('user', 'login', { id: 1, name: 'Alice' });
globalBus.publish('notification', 'new', 'You have a new message!');
```

---

---

## 详细原理解析

### JavaScript 引擎工作原理

JavaScript 引擎是执行 JavaScript 代码的核心组件，现代主流引擎包括 V8（Chrome、Node.js）、SpiderMonkey（Firefox）和 JavaScriptCore（Safari）。

```
JavaScript 引擎架构
┌─────────────────────────────────────────────────────────────┐
│                      JavaScript 引擎                         │
├─────────────────────────────────────────────────────────────┤
│  源代码 → 解析器 → AST → 解释器 → 字节码                   │
│              ↓                                              │
│         编译器（ JIT ）→ 优化编译器 → 本地代码              │
│              ↓                                              │
│         垃圾回收器（GC）                                     │
└─────────────────────────────────────────────────────────────┘

详细流程：
1. 解析阶段：源代码被解析成抽象语法树（AST）
2. 解释阶段：解释器将 AST 转换为字节码
3. 编译阶段：JIT 编译器将热点代码编译为本地机器码
4. 优化阶段：优化编译器进行多层优化
5. 反优化：当假设失败时回退到字节码
```

**V8 引擎的隐藏类机制**

V8 使用隐藏类（Hidden Classes）来实现类似类的高效属性访问：

```javascript
// 好的模式：保持对象结构一致
function Point(x, y) {
  this.x = x;
  this.y = y;
}

const p1 = new Point(1, 2);  // 隐藏类 C0
const p2 = new Point(3, 4);  // 隐藏类 C0（复用）

// 差的模式：动态添加属性
const obj = { x: 1 };
obj.y = 2;  // 创建新隐藏类
obj.z = 3;  // 再创建新隐藏类
// 每次添加属性都产生新的隐藏类
```

**内联缓存（Inline Caches）**

V8 使用内联缓存来加速属性访问：

```javascript
// 首次访问：记录隐藏类
function getX(point) {
  return point.x;  // 记录 point 的隐藏类
}

// 后续调用：直接使用缓存的偏移量
getX(point1);  // 缓存命中
getX(point2);  // 缓存命中
```

### 原型链深度解析

原型链是 JavaScript 实现继承的核心机制：

```javascript
// 原型链结构
function Parent() {
  this.parentProperty = true;
}
Parent.prototype.parentMethod = function() {
  return 'parent';
};

function Child() {
  Parent.call(this);  // 调用父构造函数
  this.childProperty = true;
}

// 设置原型链
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
Child.prototype.childMethod = function() {
  return 'child';
};

const child = new Child();

// 原型链遍历顺序
console.log(child instanceof Child);           // true
console.log(child instanceof Parent);          // true
console.log(child instanceof Object);          // true

// 获取原型链
console.log(Object.getPrototypeOf(child));              // Child.prototype
console.log(Object.getPrototypeOf(Object.getPrototypeOf(child)));  // Parent.prototype
console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(child))));  // Object.prototype
console.log(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(Object.getPrototypeOf(child)))));  // null

// 原型链示意图
// child → Child.prototype → Parent.prototype → Object.prototype → null
```

**属性查找机制**

```javascript
// 属性查找会沿着原型链向上搜索
console.log(child.childProperty);     // 直接属性
console.log(child.parentProperty);    // 从 Parent.prototype 查找
console.log(child.toString());       // 从 Object.prototype 查找

// hasOwnProperty 不查找原型链
console.log(child.hasOwnProperty('childProperty'));  // true
console.log(child.hasOwnProperty('parentProperty')); // false
console.log(child.hasOwnProperty('toString'));      // false

// in 操作符会查找整个原型链
console.log('parentProperty' in child);  // true
console.log('toString' in child);       // true
```

### 垃圾回收机制

JavaScript 使用自动垃圾回收，但理解其机制对避免内存泄漏至关重要：

```javascript
// 标记-清除算法
// 1. 从根对象（window/global）开始标记可达对象
// 2.  unmarked 不可达对象
// 3. 清除未标记对象

// V8 的 GC 分代
// ┌─────────────────────────────────────┐
// │           新生代（Young Generation）    │
// │  ┌──────────┐  ┌──────────┐          │
// │  │  From    │  │   To     │  存活    │
// │  │  空间    │  │   空间    │  对象    │
// │  └──────────┘  └──────────┘  晋升    │
// │      ↑              ↓                │
// │      ←──── Scavenge ────→            │
// └─────────────────────────────────────┘
//                 ↓
// ┌─────────────────────────────────────┐
// │         老生代（Old Generation）        │
// │     标记-清除 / 标记-压缩 GC          │
// └─────────────────────────────────────┘

// 内存泄漏示例
function createLeak() {
  const largeData = new Array(1000000);
  const element = document.getElementById('button');
  
  // 闭包持有 element 引用
  element.onclick = () => {
    console.log(largeData.length);  // largeData 无法释放
  };
}

// 正确模式
function noLeak() {
  const largeData = new Array(1000000);
  const element = document.getElementById('button');
  
  element.onclick = () => {
    console.log(largeData.length);
  };
  
  // 清理时
  element.onclick = null;
}

// WeakMap 防止循环引用泄漏
const cache = new WeakMap();

function process(obj) {
  if (!cache.has(obj)) {
    const result = expensiveComputation(obj);
    cache.set(obj, result);  // obj 被 GC 后自动清除
  }
  return cache.get(obj);
}
```

### 内存管理高级技巧

```javascript
// 对象池模式（减少 GC 压力）
class ObjectPool {
  constructor(factory, initialSize = 10) {
    this.factory = factory;
    this.available = [];
    this.inUse = new Set();
    
    for (let i = 0; i < initialSize; i++) {
      this.available.push(this.factory());
    }
  }
  
  acquire() {
    let obj = this.available.pop() || this.factory();
    this.inUse.add(obj);
    return obj;
  }
  
  release(obj) {
    if (this.inUse.has(obj)) {
      this.inUse.delete(obj);
      this.reset(obj);
      this.available.push(obj);
    }
  }
  
  reset(obj) {
    // 重置对象状态
    Object.keys(obj).forEach(key => delete obj[key]);
  }
}

// 使用对象池
const particlePool = new ObjectPool(() => ({
  x: 0, y: 0, vx: 0, vy: 0, alive: false
}), 100);

// 获取粒子
const particle = particlePool.acquire();
particle.x = 100;
particle.y = 200;
particle.vx = 5;
particle.vy = 10;
particle.alive = true;

// 归还粒子
particle.alive = false;
particlePool.release(particle);

// 手动触发 GC（仅调试用）
// global.gc(); // 需要 --expose-gc 标志
```

### 闭包与作用域深度理解

```javascript
// 闭包的本质
function outer() {
  const outerVar = 'outer';
  
  return function inner() {
    const innerVar = 'inner';
    console.log(outerVar);  // 闭包引用 outerVar
    console.log(innerVar);   // 闭包引用 innerVar
  };
}

const closure = outer();
// outer 的执行上下文已弹出栈
// 但 inner 函数仍然持有 outerVar 的引用

// 闭包应用：模块模式
const Counter = (function() {
  let count = 0;  // 私有变量
  
  return {
    increment() { return ++count; },
    decrement() { return --count; },
    getCount() { return count; },
    reset() { count = 0; }
  };
})();

// 闭包与循环问题
// 问题代码
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 输出: 3, 3, 3（var 是函数作用域，i 是同一个变量）

// 解决方案 1: let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 输出: 0, 1, 2（let 是块级作用域，每次迭代都有新变量）

// 解决方案 2: IIFE
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}

// 解决方案 3: 工厂函数
function createLogger(i) {
  return function() { console.log(i); };
}
for (var i = 0; i < 3; i++) {
  setTimeout(createLogger(i), 100);
}

// 闭包内存泄漏
function badExample() {
  const bigArray = new Array(10000000);
  
  return function() {
    console.log(bigArray.length);  // bigArray 永远不会被回收
  };
}

// 解决方案：手动清除引用
function goodExample() {
  const bigArray = new Array(10000000);
  let closure = null;
  
  const inner = function() {
    console.log(bigArray.length);
  };
  
  closure = inner;
  
  return function() {
    closure = null;  // 清除引用
    bigArray = null; // 清除大对象
  };
}
```

### 类与继承的实现原理

```javascript
// ES6 类本质上是构造函数 + 原型链的语法糖

class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} makes a sound`);
  }
  
  static create(name) {
    return new Animal(name);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);  // 调用父类构造函数
    this.breed = breed;
  }
  
  speak() {
    console.log(`${this.name} barks`);
  }
  
  fetch(item) {
    console.log(`${this.name} fetches ${item}`);
  }
}

// 等价于以下原型链实现
function AnimalES5(name) {
  this.name = name;
}

AnimalES5.prototype.speak = function() {
  console.log(`${this.name} makes a sound`);
};

AnimalES5.create = function(name) {
  return new AnimalES5(name);
};

function DogES5(name, breed) {
  AnimalES5.call(this, name);
  this.breed = breed;
}

DogES5.prototype = Object.create(AnimalES5.prototype);
DogES5.prototype.constructor = DogES5;
DogES5.prototype.speak = function() {
  console.log(`${this.name} barks`);
};
DogES5.prototype.fetch = function(item) {
  console.log(`${this.name} fetches ${item}`);
};

// 验证等价性
const dog = new Dog('Buddy', 'Labrador');
console.log(dog instanceof Dog);      // true
console.log(dog instanceof Animal);   // true
console.log(dog instanceof Object);   // true
console.log(Object.getPrototypeOf(dog) === Dog.prototype);      // true
console.log(Object.getPrototypeOf(Dog.prototype) === Animal.prototype); // true
```

### 属性描述符与 DefineProperty

```javascript
// 属性描述符
const obj = {};

// 数据属性描述符
Object.defineProperty(obj, 'name', {
  value: 'Alice',
  writable: true,      // 可写
  enumerable: true,   // 可枚举
  configurable: true  // 可配置（可删除、可修改描述符）
});

// 访问器属性描述符
let temperature = 22;
Object.defineProperty(obj, 'temp', {
  get() {
    return temperature;
  },
  set(value) {
    temperature = value;
    console.log(`Temperature set to ${value}`);
  },
  enumerable: true,
  configurable: true
});

// 获取属性描述符
console.log(Object.getOwnPropertyDescriptor(obj, 'name'));
// { value: 'Alice', writable: true, enumerable: true, configurable: true }

// 定义多个属性
Object.defineProperties(obj, {
  prop1: { value: 1, writable: true },
  prop2: { value: 2, writable: false },
  prop3: { 
    get() { return this.prop1 + this.prop2; },
    set(value) { this.prop1 = value / 2; this.prop2 = value / 2; }
  }
});

// 密封与冻结
const sealed = { a: 1 };
Object.seal(sealed);  // 禁止添加/删除属性，可以修改现有属性
sealed.a = 2;  // OK
sealed.b = 3;  // 忽略
delete sealed.a; // 忽略

const frozen = { a: 1 };
Object.freeze(frozen);  // 完全冻结，不可修改
frozen.a = 2;  // 静默失败（严格模式下抛出错误）
frozen.b = 3;  // 静默失败

// 深度冻结
function deepFreeze(obj) {
  Object.freeze(obj);
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return obj;
}

// freeze vs seal vs preventExtensions
// preventExtensions: 禁止添加新属性
// seal: preventExtensions + 禁止删除属性
// freeze: seal + 禁止修改属性值
```

### 函数内部方法详解

```javascript
// call, apply, bind

function greet(greeting, punctuation) {
  console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: 'Alice' };

greet.call(person, 'Hello', '!');  // 直接调用
greet.apply(person, ['Hi', '.']);   // 参数作为数组

const boundGreet = greet.bind(person, 'Hey');  // 预设 this 和第一个参数
boundGreet('?');  // "Hey, Alice?"

// call 的其他用法：借用方法
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
const arr = Array.prototype.slice.call(arrayLike);
// 或
const arr2 = Array.from(arrayLike);

// apply 的其他用法：展开参数
Math.max.apply(null, [1, 5, 3, 8, 2]);  // 8
Math.max(...[1, 5, 3, 8, 2]);  // 8

// bind 的部分应用
function multiply(a, b) {
  return a * b;
}

const double = multiply.bind(null, 2);
const triple = multiply.bind(null, 3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

### 类型系统深度解析

```javascript
// 类型检查
console.log(typeof 42);            // "number"
console.log(typeof 'hello');         // "string"
console.log(typeof true);            // "boolean"
console.log(typeof undefined);       // "undefined"
console.log(typeof null);           // "object" (历史 bug)
console.log(typeof {});             // "object"
console.log(typeof []);             // "object"
console.log(typeof function(){});    // "function"
console.log(typeof Symbol('s'));    // "symbol"
console.log(typeof 1n);             // "bigint"

// instanceof 检查原型链
console.log([] instanceof Array);           // true
console.log([] instanceof Object);          // true
console.log(new Date() instanceof Date);    // true

// Symbol 类型
const sym1 = Symbol('description');
const sym2 = Symbol('description');
console.log(sym1 === sym2);  // false

// Symbol 作为对象属性
const obj = {
  [sym1]: 'value1',
  [sym2]: 'value2'
};

console.log(obj[sym1]);  // "value1"
console.log(Object.getOwnPropertySymbols(obj));  // [Symbol(description), Symbol(description)]

// BigInt 类型
const bigNum = 9007199254740991n;
const anotherBig = BigInt(9007199254740991);
console.log(bigNum + anotherBig);  // 18014398509481982n

// 类型转换规则
console.log('' + []);      // ""
console.log('' + {});      // "[object Object]"
console.log('' + null);    // "null"
console.log('' + undefined); // "undefined"
console.log([] == false);  // true
console.log({} == false);  // false

// 显式类型转换
const num = parseInt('42px', 10);  // 42
const float = parseFloat('3.14');  // 3.14
const bool = Boolean('truthy');    // true
const str = String(123);           // "123"
const num2 = Number('42');         // 42
const arr3 = Array.from('hello');  // ["h", "e", "l", "l", "o"]

// NaN 处理
console.log(isNaN(NaN));                    // true
console.log(Number.isNaN(NaN));             // true（更精确）
console.log(isFinite(10));                  // true
console.log(Number.isFinite(Infinity));     // false
```

---

## 常用 API 详解

### 全局对象核心方法

```javascript
// JSON
const obj = { name: 'Alice', age: 30, hobbies: ['reading', 'coding'] };

// 序列化
const json = JSON.stringify(obj);
const jsonPretty = JSON.stringify(obj, null, 2);
const jsonFiltered = JSON.stringify(obj, ['name', 'age'], 2);

// 自定义 toJSON
const custom = {
  name: 'Alice',
  toJSON() { return this.name; }
};
console.log(JSON.stringify(custom));  // "\"Alice\""

// 解析
const parsed = JSON.parse(json);
const parsedWithReviver = JSON.parse(json, (key, value) => {
  if (typeof value === 'string' && key === 'date') {
    return new Date(value);
  }
  return value;
});

// 结构化克隆（支持循环引用）
const original = { ref: null };
original.ref = original;
const cloned = structuredClone(original);  // 不报错

// URL
const url = new URL('https://example.com:8080/path?name=Alice&age=30#section');
console.log(url.protocol);  // "https:"
console.log(url.host);      // "example.com:8080"
console.log(url.pathname);  // "/path"
console.log(url.search);    // "?name=Alice&age=30"
console.log(url.hash);      // "#section"
console.log(url.searchParams.get('name'));  // "Alice"
url.searchParams.append('new', 'value');
console.log(url.toString());

// URLSearchParams
const params = new URLSearchParams('name=Alice&age=30');
params.set('age', '31');
console.log(params.toString());  // "name=Alice&age=31"
params.forEach((value, key) => console.log(`${key}: ${value}`));

// Base64
const encoded = btoa('Hello');  // "SGVsbG8="
const decoded = atob(encoded);   // "Hello"

// 安全 Base64（支持 Unicode）
const encodedUnicode = btoa(unescape(encodeURIComponent('你好')));
const decodedUnicode = decodeURIComponent(escape(atob(encodedUnicode)));
console.log(decodedUnicode);  // "你好"
```

### Array 高级方法

```javascript
const arr = [1, 2, 3, 4, 5];

// 遍历方法
arr.forEach((item, index) => console.log(`${index}: ${item}`));

// 转换方法
const doubled = arr.map(x => x * 2);
const evens = arr.filter(x => x % 2 === 0);
const sum = arr.reduce((acc, x) => acc + x, 0);
const found = arr.find(x => x > 3);        // 返回第一个满足条件的值
const foundIndex = arr.findIndex(x => x > 3);  // 返回第一个满足条件的索引
const exists = arr.some(x => x > 4);       // 是否有满足条件的元素
const allPositive = arr.every(x => x > 0); // 是否所有元素都满足条件

// 扁平化
const nested = [1, [2, [3, [4]]]];
const flat = nested.flat(2);              // [1, 2, 3, [4]]
const flatMap = arr.flatMap(x => [x, x * 2]);  // [1, 2, 2, 4, 3, 6, 4, 8, 5, 10]

// 排序
const unsorted = [3, 1, 4, 1, 5, 9, 2, 6];
const sorted = [...unsorted].sort((a, b) => a - b);  // 数字排序
const reversed = [...unsorted].sort((a, b) => b - a);  // 降序

// 填充与复制
const filled = new Array(5).fill(0);           // [0, 0, 0, 0, 0]
const copied = arr.slice();                      // 浅拷贝
const part = arr.slice(1, 3);                   // [2, 3]
const merged = [...arr, ...[6, 7]];             // [1, 2, 3, 4, 5, 6, 7]

// 解构赋值
const [first, second, ...rest] = arr;
console.log(first);   // 1
console.log(second);  // 2
console.log(rest);    // [3, 4, 5]

// 伪数组转数组
const arrayLike = { 0: 'a', 1: 'b', length: 2 };
const arrFrom = Array.from(arrayLike);
const arrFromWithMap = Array.from(arrayLike, x => x.toUpperCase());
const arrOf = Array.of(1, 2, 3);  // [1, 2, 3]

// includes vs indexOf
console.log(arr.includes(3));    // true
console.log(arr.indexOf(3) !== -1);  // true（更老的写法）
```

### Object 高级方法

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

// 合并
const merged = { ...obj1, ...obj2 };  // { a: 1, b: 3, c: 4 }

// 浅拷贝
const shallowCopy = { ...obj1 };
const shallowCopy2 = Object.assign({}, obj1);

// 深拷贝
const nested = { a: { b: { c: 1 } } };
const deepCopy = JSON.parse(JSON.stringify(nested));
const deepCopy2 = structuredClone(nested);  // 原生支持循环引用

// keys, values, entries
console.log(Object.keys(obj1));   // ["a", "b"]
console.log(Object.values(obj1)); // [1, 2]
console.log(Object.entries(obj1)); // [["a", 1], ["b", 2]]

// fromEntries
const pairs = [['a', 1], ['b', 2]];
const fromEntries = Object.fromEntries(pairs);  // { a: 1, b: 2 }

// 对象比较
const a = { x: 1 };
const b = { x: 1 };
console.log(a === b);  // false（引用比较）
console.log(JSON.stringify(a) === JSON.stringify(b));  // true（浅比较）

// 深度相等比较函数
function deepEqual(obj1, obj2) {
  if (obj1 === obj2) return true;
  if (obj1 == null || obj2 == null) return false;
  if (typeof obj1 !== 'object' || typeof obj2 !== 'object') return false;
  
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);
  
  if (keys1.length !== keys2.length) return false;
  
  return keys1.every(key => deepEqual(obj1[key], obj2[key]));
}

// 创建具有特定原型的对象
const proto = { greet() { console.log('Hello'); } };
const fromProto = Object.create(proto);
fromProto.name = 'Alice';

// 获取对象的原型
console.log(Object.getPrototypeOf(fromProto) === proto);  // true
console.log(fromProto.__proto__ === proto);  // true（不推荐使用）

// 检查属性
console.log('name' in fromProto);           // true（包含原型链）
console.log(Object.prototype.hasOwnProperty.call(fromProto, 'name'));  // true（仅自有属性）
console.log(Object.hasOwn(fromProto, 'name'));  // ES2022+

// 属性顺序
// 整数索引 → 按顺序 → 符号 → 按插入顺序
```

### Map, Set, WeakMap, WeakSet

```javascript
// Map - 键可以是任意类型
const map = new Map();
map.set('key', 'value');
map.set(1, 'number key');
map.set({ id: 1 }, 'object key');
map.set(function(){}, 'function key');

console.log(map.get('key'));  // "value"
console.log(map.has('key'));  // true
console.log(map.size);         // 4
map.delete('key');
map.clear();

for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// Map vs Object
// Map: 有序、可迭代、键可以是任意类型、size 属性
// Object: 键只能是字符串或符号、更快（小型数据集）

// Set - 值的集合
const set = new Set([1, 2, 3, 2, 1]);
console.log(set.size);  // 3（自动去重）
set.add(4);
set.delete(1);
console.log(set.has(2));  // true

for (const value of set) {
  console.log(value);
}

// 数组去重
const arr = [1, 2, 3, 2, 1, 4];
const unique = [...new Set(arr)];  // [1, 2, 3, 4]

// WeakMap - 键是弱引用（对象）
const weakMap = new WeakMap();
const obj = { id: 1 };
weakMap.set(obj, 'data');
// obj 被垃圾回收后，weakMap 自动删除该条目

// WeakSet - 值是弱引用
const weakSet = new WeakSet();
const obj1 = { name: 'Alice' };
weakSet.add(obj1);
// obj1 被垃圾回收后，weakSet 自动删除该引用

// WeakMap 应用：私有属性
const #privateData = new WeakMap();

class MyClass {
  constructor(data) {
    #privateData.set(this, data);
  }
  
  getData() {
    return #privateData.get(this);
  }
}

// WeakMap 应用：缓存计算结果
const cache = new WeakMap();

function expensiveComputation(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }
  const result = doExpensiveWork(obj);
  cache.set(obj, result);
  return result;
}
```

### Promise 进阶用法

```javascript
// Promise.all - 所有成功才成功
const promises = [
  fetch('/api/users').then(r => r.json()),
  fetch('/api/posts').then(r => r.json()),
  fetch('/api/comments').then(r => r.json())
];

const [users, posts, comments] = await Promise.all(promises);

// Promise.allSettled - 等待所有完成
const results = await Promise.allSettled([
  fetch('/api/a'),
  fetch('/api/b'),
  fetch('/api/c')
]);

results.forEach(result => {
  if (result.status === 'fulfilled') {
    console.log('Success:', result.value);
  } else {
    console.log('Failed:', result.reason);
  }
});

// Promise.race - 首个完成
const response = await Promise.race([
  fetch('/api/fast'),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('Timeout')), 5000)
  )
]);

// Promise.any - 首个成功（忽略失败）
const firstSuccess = await Promise.any([
  fetch('/api/primary'),
  fetch('/api/backup'),
  fetch('/api/tertiary')
]);

// Promise.withResolvers (ES2024)
function loadImage(url) {
  const { promise, resolve, reject } = Promise.withResolvers();
  
  const img = new Image();
  img.onload = () => resolve(img);
  img.onerror = () => reject(new Error(`Failed to load ${url}`));
  img.src = url;
  
  return promise;
}

// 链式调用
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts?userId=${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error))
  .finally(() => hideLoader());

// Promise 错误穿透
new Promise((resolve, reject) => reject(new Error('Fail')))
  .then(result => console.log(result))  // 被跳过
  .catch(error => console.error(error))  // 被捕获
  .then(result => console.log('Recovered'))  // 继续执行
  .finally(() => console.log('Done'));
```

### 正则表达式进阶

```javascript
// 基础语法
const regex = /pattern/flags;
const regex2 = new RegExp('pattern', 'flags');

// 标志
const re = /hello/gi;  // g: 全局, i: 不区分大小写, m: 多行, s: dotAll, u: Unicode

// 字符类
/[abc]/        // 匹配 a, b, 或 c
/[^abc]/       // 匹配除了 a, b, c 的字符
/[a-z]/        // 匹配小写字母
/[A-Za-z0-9]/  // 匹配字母数字

// 预定义字符类
/\d/           // 数字 [0-9]
/\D/           // 非数字 [^0-9]
/\w/           // 单词字符 [A-Za-z0-9_]
/\W/           // 非单词字符
/\s/           // 空白字符
/\S/           // 非空白字符
/. /           // 任意字符（除换行符）

// 量词
/ab*c/         // 0 个或多个 b
/ab+c/         // 1 个或多个 b
/ab?c/         // 0 个或 1 个 b
/ab{3}c/      // 恰好 3 个 b
/ab{2,5}c/    // 2 到 5 个 b
/ab{2,}c/      // 至少 2 个 b

// 位置
/^abc/         // 字符串开头
/abc$/         // 字符串结尾
/\bword\b/     // 单词边界
/\Bword\B/     // 非单词边界

// 分组与捕获
/(\d{4})-(\d{2})-(\d{2})/  // 捕获组
/(?:non-capture)/           // 非捕获组
/(?<year>\d{4})/           // 命名捕获组 (ES2018)

// 或
/apple|banana|orange/

// 前瞻与后顾
/\d+(?= dollars)/         // 正前瞻：后面是 dollars 的数字
/\d+(?! dollars)/         // 负前瞻：后面不是 dollars 的数字
/(?<=\$)\d+/              // 正后顾：前面是 $ 的数字
/(?<!\$)\d+/              // 负后顾：前面不是 $ 的数字

// 使用示例
const email = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
console.log(email.test('test@example.com'));  // true

const date = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = '2024-01-15'.match(date);
console.log(match.groups);  // { year: '2024', month: '01', day: '15' }

// String 方法
'Hello World'.match(/\w+/g);           // ['Hello', 'World']
'Hello World'.replace(/\w+/g, w => w.toUpperCase());  // 'HELLO WORLD'
'Hello World'.split(/\s+/);            // ['Hello', 'World']
'Hello World'.search(/World/);         // 6

// exec 的捕获组
const re = /(\w+)\s+(\w+)/;
let match;
while ((match = re.exec('Hello World')) !== null) {
  console.log(match[0], match[1], match[2]);
}
```

### 字符串处理高级技巧

```javascript
const str = '  Hello, World!  ';

// 修剪
str.trim();          // "Hello, World!"
str.trimStart();      // "Hello, World!  " (ES2019)
str.trimEnd();        // "  Hello, World!" (ES2019)
str.trimLeft();       // 别名
str.trimRight();      // 别名

// 填充
'5'.padStart(3, '0');     // "005"
'5'.padEnd(3, '0');       // "500"
'5'.padStart(3, '0');      // "005"

// 大小写
str.toUpperCase();         // "  HELLO, WORLD!  "
str.toLowerCase();         // "  hello, world!  "
str.toLocaleUpperCase();   // 考虑语言环境
str.toLocaleLowerCase();   // 考虑语言环境

// 查找
str.includes('World');      // true
str.startsWith('  Hello'); // true
str.endsWith('!  ');       // true
str.indexOf('o');          // 5
str.lastIndexOf('o');      // 8
str.indexOf('o', 6);       // 8

// 提取
str.substring(2, 7);        // "Hello"
str.slice(2, -3);          // "Hello, World"
str.charAt(0);             // " "
str.charCodeAt(0);         // 32

// 分割与连接
str.split(', ');           // ["  Hello", "World!  "]
['a', 'b', 'c'].join('-'); // "a-b-c"
str.repeat(3);             // "  Hello, World!    "... (3次)

// 模板字符串高级
const name = 'Alice';
const age = 30;

// 嵌套模板
const nested = `${`${name}`} is ${age}`;

// 标签模板
function tag(strings, ...values) {
  console.log(strings);  // ["", " is ", " years old"]
  console.log(values);   // ["Alice", 30]
  return 'Processed';
}
tag`${name} is ${age} years old`;

// 模板字符串处理 HTML
function escapeHTML(strings, ...values) {
  const escape = (s) => s
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
  
  return strings.reduce((acc, str, i) => 
    acc + str + escape(values[i] || ''), '');
}

const userInput = '<script>alert("XSS")</script>';
const safe = escapeHTML`User said: ${userInput}`;
```

---

## 性能优化技巧

### 代码执行优化

```javascript
// 1. 避免重复计算
// 差
function calculate(items) {
  return items.filter(x => x > 10)
              .map(x => x * 2)
              .reduce((a, b) => a + b);
}

// 好：缓存中间结果
function calculateOptimized(items) {
  const filtered = items.filter(x => x > 10);
  const doubled = filtered.map(x => x * 2);
  return doubled.reduce((a, b) => a + b, 0);
}

// 2. 使用位运算（特定场景）
// 位与替代取模
const index = counter & (size - 1);  // size 必须是 2 的幂

// 位或替代 Math.floor
const floor = value | 0;  // 对于正数等价于 Math.floor

// 3. 减少对象创建
// 差
function process(data) {
  return data.map(item => ({
    ...item,
    processed: true
  }));
}

// 好：复用对象
const processedMarker = { processed: true };
function processOptimized(data) {
  return data.map(item => Object.assign({}, item, processedMarker));
}

// 4. 避免使用 with 语句
// with 语句会阻止 JS 引擎优化

// 5. 使用延迟加载
const heavyModule = null;
function getHeavyModule() {
  if (!heavyModule) {
    heavyModule = import('./heavy-module.js');
  }
  return heavyModule;
}

// 6. 缓存函数结果（记忆化）
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key);
    }
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

// 7. 批量操作减少 DOM 访问
// 差
elements.forEach(el => {
  el.style.width = '100px';
  el.style.height = '100px';
  el.style.color = 'red';
});

// 好：使用 CSS 类或 style.cssText
elements.forEach(el => {
  el.style.cssText = 'width: 100px; height: 100px; color: red;';
});

// 或
elements.forEach(el => el.classList.add('processed'));

// 8. 事件委托替代多个事件监听
// 差
listItems.forEach(item => {
  item.addEventListener('click', handleClick);
});

// 好
list.addEventListener('click', (e) => {
  if (e.target.matches('.list-item')) {
    handleClick(e.target);
  }
});

// 9. 使用 requestAnimationFrame 节流
let ticking = false;
function onScroll() {
  if (!ticking) {
    requestAnimationFrame(() => {
      updateOnScroll();
      ticking = false;
    });
    ticking = true;
  }
}

// 10. 避免try-catch在热路径中
// 差
function parseJSON(str) {
  try {
    return JSON.parse(str);
  } catch {
    return null;
  }
}

// 好：预先验证
function parseJSONOptimized(str) {
  if (typeof str !== 'string') return null;
  try {
    return JSON.parse(str);
  } catch {
    return null;
  }
}
```

### 内存优化

```javascript
// 1. 对象复用
class ObjectPool {
  constructor(factory, reset, initialSize = 10) {
    this.factory = factory;
    this.reset = reset;
    this.pool = [];
    
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(factory());
    }
  }
  
  acquire() {
    return this.pool.pop() || this.factory();
  }
  
  release(obj) {
    this.reset(obj);
    this.pool.push(obj);
  }
}

// 粒子系统示例
const particlePool = new ObjectPool(
  () => ({ x: 0, y: 0, vx: 0, vy: 0, life: 0 }),
  (p) => { p.life = 0; p.x = 0; p.y = 0; }
);

// 2. 避免字符串拼接（在循环中）
// 差
let result = '';
for (let i = 0; i < 1000; i++) {
  result += items[i] + ',';
}

// 好
const parts = [];
for (let i = 0; i < 1000; i++) {
  parts.push(items[i]);
}
const result = parts.join(',');

// 3. 使用 TypedArray 处理大量数值
// 差
const points = [];
for (let i = 0; i < 1000000; i++) {
  points.push({ x: Math.random(), y: Math.random() });
}

// 好
const pointCount = 1000000;
const xCoords = new Float64Array(pointCount);
const yCoords = new Float64Array(pointCount);
for (let i = 0; i < pointCount; i++) {
  xCoords[i] = Math.random();
  yCoords[i] = Math.random();
}

// 4. 及时清理大对象引用
function processLargeData() {
  const largeData = new Array(10000000);
  // 处理数据...
  
  // 完成后清理
  largeData.length = 0;
  largeData = null;
  
  if (globalRef) {
    globalRef.data = null;
  }
}

// 5. 避免内存泄漏
class EventEmitter {
  constructor() {
    this.handlers = new Map();
  }
  
  on(event, handler) {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, new Set());
    }
    this.handlers.get(event).add(handler);
    
    // 返回取消订阅函数
    return () => this.off(event, handler);
  }
  
  off(event, handler) {
    this.handlers.get(event)?.delete(handler);
  }
  
  emit(event, data) {
    this.handlers.get(event)?.forEach(handler => handler(data));
  }
  
  // 清理所有事件
  destroy() {
    this.handlers.clear();
  }
}

// 6. 使用 WeakRef（谨慎）
let cache = null;
function getCached(obj) {
  if (cache && cache.deref() === obj) {
    return cache.deref();
  }
  cache = new WeakRef(obj);
  return obj;
}

// 7. FinalizationRegistry（GC 回调）
const registry = new FinalizationRegistry((name) => {
  console.log(`Object ${name} was garbage collected`);
});

let obj = { data: new Array(10000) };
registry.register(obj, 'myObject');
obj = null;  // 稍后触发回调
```

### DOM 操作优化

```javascript
// 1. 批量 DOM 操作
// 使用 DocumentFragment
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}
container.appendChild(fragment);  // 一次插入

// 2. 使用 transform 而非改变位置属性
// 差
element.style.left = x + 'px';
element.style.top = y + 'px';

// 好：使用 transform
element.style.transform = `translate(${x}px, ${y}px)`;

// 3. 隐藏元素后批量修改
element.style.display = 'none';
// 修改 100 个属性
element.style.display = '';

// 4. 使用 will-change 提示浏览器
element.style.willChange = 'transform';
// 或
element.classList.add('animate');

// 5. 避免强制重排
// 差：每次读取都触发重排
for (let i = 0; i < elements.length; i++) {
  elements[i].style.height = elements[i].offsetHeight + 10 + 'px';
}

// 好：批量读取，批量写入
const heights = [];
for (let i = 0; i < elements.length; i++) {
  heights.push(elements[i].offsetHeight);
}

requestAnimationFrame(() => {
  for (let i = 0; i < elements.length; i++) {
    elements[i].style.height = heights[i] + 10 + 'px';
  }
});

// 6. 使用 CSS contain
.container {
  contain: layout style paint;
}

// 7. 虚拟列表（大量数据）
class VirtualList {
  constructor(container, items, itemHeight) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.scrollTop = 0;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight);
    
    container.style.height = items.length * itemHeight + 'px';
    container.addEventListener('scroll', () => this.onScroll());
    this.render();
  }
  
  onScroll() {
    this.scrollTop = this.container.scrollTop;
    requestAnimationFrame(() => this.render());
  }
  
  render() {
    const startIndex = Math.floor(this.scrollTop / this.itemHeight);
    const endIndex = startIndex + this.visibleCount + 1;
    
    const visibleItems = this.items.slice(startIndex, endIndex);
    // 渲染可见项...
  }
}

// 8. 使用 cloneNode 克隆节点
const template = document.getElementById('item-template');
const item = template.content.cloneNode(true);
// 修改 item...

// 9. 避免在循环中访问 DOM
const items = document.querySelectorAll('.item');
const ids = Array.from(items).map(item => item.id);

// 10. 使用 closest 代替多次父元素查询
// 差
let el = event.target;
while (el && !el.classList.contains('container')) {
  el = el.parentElement;
}

// 好
const container = event.target.closest('.container');
```

---

## 与同类技术对比

### JavaScript vs TypeScript

| 特性 | JavaScript | TypeScript |
|------|------------|------------|
| 类型系统 | 动态类型 | 静态类型（可选） |
| 编译 | 无需编译（直接运行） | 需要编译 |
| 错误检测 | 运行时 | 编译时 + 运行时 |
| 学习曲线 | 低 | 中等 |
| 开发效率 | 高（快速原型） | 中（需要定义类型） |
| 代码可读性 | 依赖文档 | 类型即文档 |
| 重构支持 | 困难 | 强大 |
| 生态系统 | 广泛 | 与 JS 共享 |
| 运行时性能 | 原生 | 相当（编译后优化） |
| 第三方库类型 | 需要额外安装 | 通常内置 |
| 适用场景 | 快速脚本、小型项目 | 中大型项目 |

```javascript
// JavaScript 原生写法
function processUser(user) {
  return user.name.toUpperCase();
}

// TypeScript 类型安全写法
interface User {
  name: string;
  age?: number;
  email?: string;
}

function processUser(user: User): string {
  return user.name.toUpperCase();
}

// TypeScript 泛型
function identity<T>(arg: T): T {
  return arg;
}

const result = identity<string>('hello');
const numResult = identity<number>(42);

// TypeScript 工具类型
type PartialUser = Partial<User>;        // 所有属性可选
type RequiredUser = Required<User>;      // 所有属性必选
type ReadonlyUser = Readonly<User>;      // 所有属性只读
type UserPreview = Pick<User, 'name'>;   // 只保留部分属性
type UserWithoutAge = Omit<User, 'age'>; // 移除部分属性
```

### JavaScript vs Python（DOM 操作方面）

| 方面 | JavaScript | Python |
|------|------------|--------|
| 运行环境 | 浏览器/Node.js | 服务器端 |
| DOM 访问 | 原生支持 | 需 Selenium/BeautifulSoup |
| 异步编程 | Promise/async-await | asyncio |
| 性能 | JIT 优化 | CPython |
| 生态系统 | npm | pip |
| 移动端 | React Native | Kivy/BeeWare |
| 桌面端 | Electron | PyQt/Tkinter |
| 部署 | 直接运行/打包 | 解释执行 |

```python
# Python 等效代码（使用 Playwright）
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch()
    page = browser.new_page()
    page.goto("https://example.com")
    content = page.text_content(".title")
    print(content)
```

### JavaScript vs Go（并发方面）

| 方面 | JavaScript | Go |
|------|------------|-----|
| 并发模型 | 事件循环 + 异步 | Goroutines + Channels |
| 线程 | 单线程（Worker 除外） | 多线程原生支持 |
| 内存共享 | 受限 | 支持 |
| 学习曲线 | 低 | 中等 |
| 适用场景 | I/O 密集 | CPU 密集 |
| 并发数量 | 高（异步） | 高（Goroutine 轻量） |
| 错误处理 | try/catch/Promise | 多返回值 |

```go
// Go 并发示例
package main

import (
    "fmt"
    "time"
)

func worker(id int, jobs <-chan int, results chan<- int) {
    for j := range jobs {
        fmt.Printf("worker %d processing job %d\n", id, j)
        time.Sleep(time.Second)
        results <- j * 2
    }
}

func main() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)

    for w := 1; w <= 3; w++ {
        go worker(w, jobs, results)
    }

    for j := 1; j <= 5; j++ {
        jobs <- j
    }
    close(jobs)

    for a := 1; a <= 5; a++ {
        <-results
    }
}
```

### JavaScript vs Rust（性能方面）

| 方面 | JavaScript | Rust |
|------|------------|------|
| 内存管理 | GC 自动回收 | 所有权系统 |
| 性能 | JIT 优化 | 原生机器码 |
| 并发 | 事件循环 | 线程/Actor |
| 错误处理 | try/catch | Result/Option |
| 学习曲线 | 低 | 高 |
| 适用场景 | 通用/Web | 系统编程/高性能 |
| 包管理 | npm | Cargo |
| 类型系统 | 动态（可选静态） | 静态强类型 |

```rust
// Rust 性能示例
fn main() {
    let numbers: Vec<i32> = (1..=1000000).collect();
    
    // 并行处理
    let sum: i64 = numbers
        .par_iter()  // 并行迭代器
        .map(|&x| x as i64 * x as i64)
        .sum();
    
    println!("Sum: {}", sum);
}
```

---

## 常见问题与解决方案

### 问题 1: 浮点数精度

```javascript
// 问题
console.log(0.1 + 0.2);  // 0.30000000000000004
console.log(1.1 + 2.2);  // 3.3000000000000003

// 解决方案 1: toFixed
parseFloat((0.1 + 0.2).toFixed(10));  // 0.3

// 解决方案 2: 整数运算
function add(a, b, decimals = 2) {
  const factor = Math.pow(10, decimals);
  return (Math.round(a * factor) + Math.round(b * factor)) / factor;
}
add(0.1, 0.2);  // 0.3

// 解决方案 3: decimal.js 库
import Decimal from 'decimal.js';
new Decimal(0.1).plus(0.2).equals(0.3);  // true

// 解决方案 4: 使用 Math.round 避免精度问题
const result = Math.round((0.1 + 0.2) * 100) / 100;  // 0.3

// 金额计算专用
function formatCurrency(amount) {
  return Math.round(amount * 100) / 100;
}
```

### 问题 2: 异步错误处理

```javascript
// 问题: 未捕获的 Promise 错误
async function bad() {
  fetch('/api');  // 错误不会被捕获
}

// 解决方案 1: 全局错误处理
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  event.preventDefault();
});

// 解决方案 2: 每个 async 函数添加 try-catch
async function good() {
  try {
    const response = await fetch('/api');
    return await response.json();
  } catch (error) {
    console.error('Fetch error:', error);
    throw error;
  }
}

// 解决方案 3: 统一错误处理包装器
async function withErrorHandling(fn) {
  try {
    return await fn();
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

const data = await withErrorHandling(() => fetch('/api').then(r => r.json()));

// 解决方案 4: Result 类型模式
class Result {
  static ok(value) {
    return { ok: true, value };
  }
  
  static err(error) {
    return { ok: false, error };
  }
}

async function safeFetch(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      return Result.err(new Error(`HTTP ${response.status}`));
    }
    return Result.ok(await response.json());
  } catch (error) {
    return Result.err(error);
  }
}
```

### 问题 3: 循环引用导致内存泄漏

```javascript
// 问题
function createLeak() {
  const bigObject = new Array(1000000);
  const elements = document.querySelectorAll('.listener');

  elements.forEach(el => {
    el.addEventListener('click', () => {
      console.log(bigObject.length);  // bigObject 永远不会被回收
    });
  });
}

// 解决方案 1: 使用 WeakMap 关联
function preventLeak() {
  const bigObject = new Array(1000000);
  const elements = document.querySelectorAll('.listener');
  const dataMap = new WeakMap();

  elements.forEach(el => {
    dataMap.set(el, { bigObject });

    el.addEventListener('click', () => {
      const data = dataMap.get(el);
      console.log(data.bigObject.length);
    });
  });
}

// 解决方案 2: 显式清理
function betterPattern() {
  const bigObject = new Array(1000000);
  const elements = document.querySelectorAll('.listener');
  const handlers = [];

  elements.forEach(el => {
    const handler = () => {
      console.log(bigObject.length);
    };
    el.addEventListener('click', handler);
    handlers.push({ el, handler });
  });

  return function cleanup() {
    handlers.forEach(({ el, handler }) => {
      el.removeEventListener('click', handler);
    });
    bigObject = null;
  };
}

// 解决方案 3: AbortController
function modernPattern() {
  const controller = new AbortController();
  const elements = document.querySelectorAll('.listener');

  elements.forEach(el => {
    el.addEventListener('click', () => {
      console.log('Clicked');
    }, { signal: controller.signal });
  });

  return () => controller.abort();  // 一键清理所有监听器
}
```

### 问题 4: 深拷贝循环引用对象

```javascript
// 问题: JSON.parse(JSON.stringify()) 无法处理循环引用
const obj = { a: 1 };
obj.self = obj;  // 循环引用

try {
  JSON.parse(JSON.stringify(obj));  // 抛出错误
} catch (e) {
  console.error(e.message);  // "Converting circular structure to JSON"
}

// 解决方案 1: 手动实现
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (hash.has(obj)) return hash.get(obj);

  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);

  for (const key of Object.keys(obj)) {
    clone[key] = deepClone(obj[key], hash);
  }

  return clone;
}

// 解决方案 2: structuredClone (原生支持循环引用)
const cloned = structuredClone(originalObj);

// 解决方案 3: lodash.cloneDeep
import { cloneDeep } from 'lodash-es';
const cloned = cloneDeep(obj);

// 解决方案 4: 带选项的深拷贝
function deepCloneWithOptions(obj, options = {}) {
  const { 
    circular = true,      // 处理循环引用
    depth = Infinity,     // 深度限制
    proto = false        // 包含原型链
  } = options;
  
  const hash = new WeakMap();
  
  function clone(value, currentDepth) {
    if (value === null || typeof value !== 'object') {
      return value;
    }
    
    if (currentDepth > depth) {
      return undefined;
    }
    
    if (hash.has(value)) {
      return hash.get(value);
    }
    
    const clonedObj = Array.isArray(value) ? [] : {};
    if (circular) {
      hash.set(value, clonedObj);
    }
    
    for (const key of Object.keys(value)) {
      clonedObj[key] = clone(value[key], currentDepth + 1);
    }
    
    return clonedObj;
  }
  
  const result = clone(obj, 0);
  
  if (proto) {
    Object.setPrototypeOf(result, Object.getPrototypeOf(obj));
  }
  
  return result;
}
```

### 问题 5: 跨域请求

```javascript
// 问题: CORS 错误
fetch('https://other-domain.com/api');  // 可能被阻止

// 解决方案 1: CORS 代理
const response = await fetch('https://cors-proxy.com/https://other-domain.com/api');

// 解决方案 2: JSONP (仅 GET)
function jsonp(url, callback) {
  const callbackName = 'jsonpCallback_' + Date.now();
  
  return new Promise((resolve, reject) => {
    window[callbackName] = (data) => {
      delete window[callbackName];
      document.body.removeChild(script);
      resolve(data);
    };

    const script = document.createElement('script');
    script.src = `${url}${url.includes('?') ? '&' : '?'}callback=${callbackName}`;
    script.onerror = () => {
      delete window[callbackName];
      document.body.removeChild(script);
      reject(new Error('JSONP request failed'));
    };

    document.body.appendChild(script);
  });
}

// 解决方案 3: 自己的服务器转发
// 服务器端（Node.js）
app.get('/proxy', async (req, res) => {
  const targetUrl = req.query.url;
  const response = await fetch(targetUrl);
  const data = await response.text();
  res.send(data);
});

// 客户端
const response = await fetch('/proxy?url=https://other-domain.com/api');

// 解决方案 4: 避免跨域的替代方案
// 使用 postMessage, MessageChannel 进行同源通信
// 使用 CORS Headers（Access-Control-Allow-Origin）
```

### 问题 6: this 丢失

```javascript
// 问题
class Timer {
  constructor() {
    this.seconds = 0;
    setInterval(this.tick, 1000);  // this 丢失
  }

  tick() {
    this.seconds++;  // this 不是 Timer 实例
  }
}

// 解决方案 1: 箭头函数
class Timer1 {
  constructor() {
    this.seconds = 0;
    setInterval(() => this.tick(), 1000);
  }

  tick() {
    this.seconds++;
  }
}

// 解决方案 2: bind
class Timer2 {
  constructor() {
    this.seconds = 0;
    setInterval(this.tick.bind(this), 1000);
  }

  tick() {
    this.seconds++;
  }
}

// 解决方案 3: 类字段语法
class Timer3 {
  seconds = 0;
  tick = () => {
    this.seconds++;
  };
  
  constructor() {
    setInterval(this.tick, 1000);
  }
}

// 解决方案 4: 在 constructor 中绑定
class Timer4 {
  constructor() {
    this.seconds = 0;
    this.tick = this.tick.bind(this);
    setInterval(this.tick, 1000);
  }

  tick() {
    this.seconds++;
  }
}

// 解决方案 5: Proxy
class Timer5 {
  constructor() {
    this.seconds = 0;
    const handler = {
      get: (target, prop, receiver) => {
        if (prop === 'tick') {
          return target.tick.bind(target);
        }
        return Reflect.get(target, prop, receiver);
      }
    };
    return new Proxy(this, handler);
  }

  tick() {
    this.seconds++;
  }
}
```

### 问题 7: 深比较

```javascript
// 问题: 普通比较无法深度比较对象
const a = { x: 1, y: { z: 2 } };
const b = { x: 1, y: { z: 2 } };
console.log(a === b);  // false

// 解决方案 1: JSON 序列化
function deepEqual(a, b) {
  return JSON.stringify(a) === JSON.stringify(b);
}

// 缺点：属性顺序敏感，不支持 undefined、函数、Date 等

// 解决方案 2: 完整实现
function deepEqual(a, b) {
  if (a === b) return true;
  
  if (a === null || b === null) return false;
  
  if (typeof a !== 'object' || typeof b !== 'object') return false;
  
  if (a.constructor !== b.constructor) return false;
  
  if (a instanceof Date && b instanceof Date) {
    return a.getTime() === b.getTime();
  }
  
  if (a instanceof RegExp && b instanceof RegExp) {
    return a.toString() === b.toString();
  }
  
  if (a instanceof Map && b instanceof Map) {
    if (a.size !== b.size) return false;
    for (const [key, value] of a) {
      if (!deepEqual(value, b.get(key))) return false;
    }
    return true;
  }
  
  if (a instanceof Set && b instanceof Set) {
    if (a.size !== b.size) return false;
    for (const value of a) {
      if (!b.has(value)) return false;
    }
    return true;
  }
  
  const keysA = Object.keys(a);
  const keysB = Object.keys(b);
  
  if (keysA.length !== keysB.length) return false;
  
  return keysA.every(key => deepEqual(a[key], b[key]));
}

// 解决方案 3: 使用 lodash
import { isEqual } from 'lodash-es';
console.log(isEqual(a, b));  // true

// 解决方案 4: 第三方库（如 deep-eql, fast-deep-equal）
```

### 问题 8: 数组去重

```javascript
// 问题：需要处理各种类型
const mixed = [1, '1', {}, {}, null, null, undefined, undefined, NaN, NaN];

// 解决方案 1: Set（不适用 NaN 和对象）
[...new Set(mixed)];  // [1, '1', {}, {}, null, undefined, NaN]

// 解决方案 2: 使用 hasOwnProperty + includes
function unique(arr) {
  const seen = new WeakMap();
  return arr.filter(item => {
    const type = typeof item;
    if (type === 'object') {
      if (seen.has(item)) return false;
      seen.set(item, true);
      return true;
    }
    return !seen.has(item) && seen.set(item, true);
  });
}

// 解决方案 3: 使用 JSON.stringify（简单但有局限）
function uniqueJSON(arr) {
  const seen = new Set();
  return arr.filter(item => {
    const key = JSON.stringify(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}

// 解决方案 4: 使用 isNaN 的特性
function unique(arr) {
  return arr.filter((item, index) => {
    return arr.indexOf(item) === index && 
           !(item !== item);  // 处理 NaN
  });
}

// 解决方案 5: 完整实现
function unique(arr, strict = true) {
  const seen = new Map();
  
  return arr.filter(item => {
    const key = strict ? item : (item === null ? 'null' : item);
    
    if (typeof key === 'object' || typeof key === 'function') {
      if (seen.has(key)) return false;
      seen.set(key, true);
      return true;
    }
    
    if (typeof key === 'number' && isNaN(key)) {
      return !seen.has(NaN) && seen.set(NaN, true);
    }
    
    return !seen.has(key) && seen.set(key, true);
  });
}
```

---

## 深度实战案例

### 完整的响应式系统实现

```javascript
// 基于 Proxy 的响应式系统
class Reactive {
  constructor(target) {
    this._target = target;
    this._handlers = new Map();
    this._proxy = this._createReactive(target);
    return this._proxy;
  }

  _createReactive(target) {
    const handler = {
      get(obj, prop, receiver) {
        const value = Reflect.get(obj, prop, receiver);

        // 递归处理嵌套对象
        if (value !== null && typeof value === 'object') {
          return new Reactive(value)._proxy;
        }

        // 触发 get 订阅
        Reactive.currentTracker?.track(prop);
        return value;
      },

      set(obj, prop, value, receiver) {
        const oldValue = obj[prop];
        const result = Reflect.set(obj, prop, value, receiver);

        if (oldValue !== value) {
          // 触发变更通知
          Reactive.currentTracker?.notify(prop, value, oldValue);
        }

        return result;
      },

      deleteProperty(obj, prop) {
        const oldValue = obj[prop];
        const result = Reflect.deleteProperty(obj, prop);

        if (result) {
          Reactive.currentTracker?.notify(prop, undefined, oldValue);
        }

        return result;
      }
    };

    return new Proxy(target, handler);
  }

  // 订阅特定属性
  watch(property, callback) {
    if (!this._handlers.has(property)) {
      this._handlers.set(property, new Set());
    }
    this._handlers.get(property).add(callback);

    return () => {
      this._handlers.get(property)?.delete(callback);
    };
  }

  // 全局追踪器
  static currentTracker = null;

  static effect(fn) {
    const tracker = {
      _deps: new Map(),
      _dirty: new Set(),

      track(property) {
        if (!this._deps.has(property)) {
          this._deps.set(property, new Set());
        }
        this._deps.get(property).add(this);
      },

      notify(property, newValue, oldValue) {
        const subscribers = this._deps.get(property);
        if (subscribers) {
          subscribers.forEach(sub => {
            sub._dirty.add(property);
          });
        }
      },

      run(fn) {
        Reactive.currentTracker = this;
        const result = fn();
        Reactive.currentTracker = null;
        return result;
      },

      getDirty() {
        const dirty = Array.from(this._dirty);
        this._dirty.clear();
        return dirty;
      }
    };

    return tracker.run(fn);
  }
}

// 使用示例
const state = new Reactive({
  user: { name: 'Alice', age: 25 },
  posts: [],
  theme: 'dark'
});

// 监视特定属性
state.watch('theme', (newVal, oldVal) => {
  console.log(`Theme changed: ${oldVal} -> ${newVal}`);
  document.body.className = newVal;
});

// 使用 effect
Reactive.effect(() => {
  const name = state.user.name;
  console.log(`User name: ${name}`);
});

// 自动追踪依赖
function renderUser() {
  const html = `
    <div class="user">
      <h1>${state.user.name}</h1>
      <p>Age: ${state.user.age}</p>
    </div>
  `;
  document.getElementById('app').innerHTML = html;
}

Reactive.effect(renderUser);

// 修改触发更新
state.user.name = 'Bob';  // 自动触发 renderUser
state.theme = 'light';     // 触发主题更新
```

---

> [!SUCCESS]
> 本文档系统性地介绍了 JavaScript 语言核心、执行上下文、异步编程、DOM/BOM API 及浏览器工作原理，并包含大量实战代码示例和性能优化技巧。这些底层知识是理解前端框架（React/Vue）和解决复杂问题的根基，也是区分普通开发者和资深工程师的关键分野。
