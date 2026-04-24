# TypeScript 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 TypeScript 5.6 新特性、类型系统深度解析、Zod 运行时验证、以及 TypeScript 在 AI 编程中的实战应用。

---

## 目录

1. [[#TypeScript 概述与核心价值]]
2. [[#TypeScript 5.6 新特性]]
3. [[#类型系统深度解析]]
4. [[#TypeScript vs JavaScript 对比]]
5. [[#tsconfig.json 深度解析]]
6. [[#Zod 运行时验证]]
7. [[#AI 编程中的 TypeScript 优势]]
8. [[#实战场景与最佳实践]]
9. [[#选型建议]]

---

## TypeScript 概述与核心价值

### 为什么选择 TypeScript

TypeScript 是 JavaScript 的超集，由微软于 2012 年正式发布。经过十余年的发展，TypeScript 已成为前端开发的事实标准。根据 2026 年 Stack Overflow 开发者调查，TypeScript 连续第四年蝉联「最受欢迎的编程语言」榜首，使用率超过 65%。

TypeScript 的核心价值体现在三个维度：

**类型安全**：在编译阶段捕获类型错误，而非等到运行时才发现问题。这对于大型项目而言意味着更低的维护成本和更高的代码可靠性。

**开发体验**：IDE 的智能提示、代码补全、跳转定义等功能在 TypeScript 下工作得更加精准。开发者的生产力显著提升。

**可维护性**：类型注释本身就是最好的文档。新成员阅读代码时能快速理解数据结构，减少认知负担。

### TypeScript 在 AI 编程中的独特优势

在 AI 应用开发中，TypeScript 的价值更加凸显：

- **结构化输出**：大模型的结构化输出（JSON Mode）与 TypeScript 类型系统天然契合，可以实现端到端的类型安全。
- **工具调用**：Claude、GPT 等模型的 Function Calling 与 TypeScript 类型定义完美对齐。
- **全栈统一**：前后端使用同一套类型定义，消除接口不一致的隐患。

---

## TypeScript 5.6 新特性

TypeScript 5.6 于 2025 年发布，带来了多项重大改进：

### 核心新特性一览

| 特性 | 说明 | 优势 |
|------|------|------|
| **Iterator Helper** | 原生支持迭代器链式操作 | 更优雅的数据处理 |
| **New Set Methods** | Set 新增并集、交集、差集方法 | 数学运算更直观 |
| **Disallowed Null checks** | 禁止某些隐式空检查 | 避免意外行为 |
| **Module Detection** | 自动识别 ES 模块 | 配置更简单 |
| **Regular Expression Syntax** | 更新的正则语法 | 性能提升 |

### Iterator Helper 详解

TypeScript 5.6 引入了 Iterator Helper，这是 JavaScript 迭代器协议的重大扩展：

```typescript
// 传统方式：每次都要写 filter/map
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 使用 Iterator Helper
const result = numbers.values()
  .filter(x => x % 2 === 0)  // 过滤偶数
  .map(x => x * 2)            // 翻倍
  .toArray();                 // 转为数组

console.log(result); // [4, 8, 12, 16, 20]

// 惰性计算：只在消费时才执行
const lazyResult = numbers.values()
  .filter(x => x % 2 === 0)
  .map(x => {
    console.log(`Processing ${x}`); // 不会立即输出
    return x * 2;
  });

// 消费时才会执行管道
for (const num of lazyResult) {
  console.log(num);
}
```

### Set 操作方法

```typescript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// 并集
const union = setA.union(setB);
// Set { 1, 2, 3, 4, 5, 6 }

// 交集
const intersection = setA.intersection(setB);
// Set { 3, 4 }

// 差集 (A - B)
const difference = setA.difference(setB);
// Set { 1, 2 }

// 对称差集
const symmetricDiff = setA.symmetricDifference(setB);
// Set { 1, 2, 5, 6 }

// 是否是子集
const isSubset = setA.isSubsetOf(new Set([1, 2, 3, 4, 5]));
// true
```

---

## 类型系统深度解析

### 基础类型

TypeScript 的基础类型系统为代码提供了坚实的类型保障：

```typescript
// 基础类型
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let notDefined: undefined = undefined;
let nothing: null = null;

// 数组
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"]; // 泛型语法

// 元组：固定长度、固定类型的数组
let person: [string, number] = ["Alice", 30];

// 枚举
enum Status {
  Pending = "PENDING",
  Active = "ACTIVE",
  Completed = "COMPLETED"
}

// 字面量类型
type Direction = "north" | "south" | "east" | "west";
let direction: Direction = "north";

// Any 和 Unknown
let flexible: any = "可以任何类型"; // 绕过了类型检查
let safe: unknown = "类型安全 any"; // 必须进行类型检查才能使用
```

### 泛型进阶

泛型是 TypeScript 类型系统最强大的特性之一，允许创建可复用的类型安全组件：

```typescript
// 基础泛型函数
function identity<T>(arg: T): T {
  return arg;
}

// 泛型约束
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello");      // OK: string 有 length
logLength([1, 2, 3]);    // OK: array 有 length
logLength({ length: 10 }); // OK: 对象有 length
// logLength(123);       // Error: number 没有 length

// 多类型约束
function merge<T extends object, U extends object>(target: T, source: U): T & U {
  return { ...target, ...source };
}

// 泛型类
class Container<T> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  get(): T | undefined {
    return this.items[0];
  }

  getAll(): T[] {
    return [...this.items];
  }
}

// 泛型工具类型
type Nullable<T> = T | null;
type NonNullable<T> = T extends null | undefined ? never : T;
type PromiseType<T> = T extends Promise<infer U> ? U : T;

// 泛型条件类型
type IsString<T> = T extends string ? "yes" : "no";
type A = IsString<string>;  // "yes"
type B = IsString<number>;  // "no"
```

### 联合类型与交叉类型

```typescript
// 联合类型：可以是多种类型之一
type StringOrNumber = string | number;
type SuccessResponse = { status: "success"; data: any };
type ErrorResponse = { status: "error"; message: string };
type ApiResponse = SuccessResponse | ErrorResponse;

// 类型守卫
function handleResponse(response: ApiResponse) {
  if (response.status === "success") {
    console.log(response.data); // TypeScript 知道这是 success 分支
  } else {
    console.log(response.message); // TypeScript 知道这是 error 分支
  }
}

// 交叉类型：合并多种类型
interface Name {
  name: string;
}
interface Age {
  age: number;
}
type Person = Name & Age;
// 等价于: { name: string; age: number }

// 联合类型与交叉类型的组合
type Draggable = {
  drag: () => void;
};
type Resizable = {
  resize: () => void;
};
type Widget = Draggable & Resizable;
// Widget 必须同时实现 drag 和 resize
```

### 条件类型

条件类型允许根据输入类型动态计算输出类型：

```typescript
// 基础条件类型
type IsArray<T> = T extends any[] ? true : false;
type A = IsArray<string[]>;  // true
type B = IsArray<string>;    // false

// 提取数组元素类型
type ElementOf<T> = T extends Array<infer U> ? U : never;
type C = ElementOf<string[]>;  // string
type D = ElementOf<number[]>;  // number

// 提取函数参数类型
type ParametersOf<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number) {}
type GreetParams = ParametersOf<typeof greet>;  // [string, number]

// 提取函数返回类型
type ReturnTypeOf<T extends (...args: any) => any> = 
  T extends (...args: any) => infer R ? R : never;

// 条件类型的分布式特性
type ToArray<T> = T extends any ? T[] : never;
type E = ToArray<string | number>;  // string[] | number[]

// 如果想避免分布式，使用元组包装
type ToArrayNonDistributive<T> = [T] extends [any] ? T[] : never;
type F = ToArrayNonDistributive<string | number>;  // (string | number)[]
```

### 映射类型

映射类型允许从现有类型派生新类型：

```typescript
// 基础映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P]; // -? 移除可选性
};

type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

type Omit<T, K extends keyof T> = {
  [P in Exclude<keyof T, K>]: T[P];
};

// 实战：创建只读字符串属性类型
type StringOnly<T> = {
  [P in keyof T]: T[P] extends string ? P : never;
}[keyof T];

// 映射类型的进阶用法
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object 
    ? T[P] extends Function 
      ? T[P] 
      : DeepReadonly<T[P]> 
    : T[P];
};

// 添加前缀或后缀
type Getters<T> = {
  [P in keyof T as `get${Capitalize<string & P>}`]: () => T[P];
};

interface User {
  name: string;
  age: number;
}
type UserGetters = Getters<User>;
// { getName: () => string; getAge: () => number }

// 过滤属性
type ExcludeFunctions<T> = {
  [P in keyof T as T[P] extends Function ? never : P]: T[P];
};
```

---

## TypeScript vs JavaScript 对比

### 核心差异对比表

| 维度 | TypeScript | JavaScript |
|------|------------|------------|
| **类型系统** | 静态类型 + 编译时检查 | 动态类型 + 运行时检查 |
| **类型推断** | 支持完整的类型推断 | 有限的类型推断 |
| **编译** | 需要编译为 JavaScript | 直接运行 |
| **IDE 支持** | 完整的智能提示和重构 | 有限的智能提示 |
| **学习曲线** | 较陡（需要学习类型系统） | 平缓 |
| **项目配置** | 需要 tsconfig.json | 无需配置 |
| **运行时开销** | 无（编译后删除类型） | 原生支持 |
| **生态兼容性** | 完全兼容 JavaScript 生态 | 原始生态 |
| **适用场景** | 中大型项目、团队协作 | 小型项目、原型开发 |
| **空值安全** | 可配置严格模式 | 依赖开发者自觉 |
| **重构支持** | 强大的类型安全重构 | 风险较高的重构 |

### 运行时性能对比

TypeScript 在编译后会移除所有类型信息，因此运行时性能与纯 JavaScript 完全一致：

```javascript
// TypeScript 源码
function add(a: number, b: number): number {
  return a + b;
}

// 编译后 (--target es2020)
function add(a, b) {
  return a + b;
}
```

> [!IMPORTANT]
> TypeScript 不会带来任何运行时性能开销。类型信息仅在编译阶段起作用，编译产物是纯 JavaScript。

### 选择建议

| 场景 | 推荐选择 | 原因 |
|------|----------|------|
| **大型前端项目** | TypeScript | 类型安全 + 团队协作 |
| **Node.js 后端** | TypeScript | 接口定义 + 框架支持完善 |
| **AI/ML 应用** | TypeScript | 与 LLM API 完美配合 |
| **快速原型** | JavaScript | 快速迭代 |
| **小型脚本** | JavaScript | 零配置 |
| **库/SDK 开发** | TypeScript | 开发者体验优先 |

---

## tsconfig.json 深度解析

### 核心配置项

```json
{
  "compilerOptions": {
    /* 基础配置 */
    "target": "ES2022",           // 编译目标 ECMAScript 版本
    "module": "NodeNext",        // 模块系统
    "lib": ["ES2022"],           // 引用的标准库
    "outDir": "./dist",          // 输出目录
    "rootDir": "./src",          // 源代码目录
    
    /* 严格模式 - 强烈建议开启 */
    "strict": true,              // 启用所有严格类型检查
    "noImplicitAny": true,       // 禁止隐式 any 类型
    "strictNullChecks": true,    // 严格的空值检查
    "strictFunctionTypes": true, // 严格的函数类型检查
    "strictPropertyInitialization": true, // 类的属性必须初始化
    
    /* 路径别名 */
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"],
      "@components/*": ["./components/*"]
    },
    
    /* 模块解析 */
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,     // 允许 default import from non-ESM
    "forceConsistentCasingInFileNames": true,
    
    /* 其他重要选项 */
    "skipLibCheck": true,         // 跳过库文件检查（加速编译）
    "declaration": true,         // 生成 .d.ts 类型声明文件
    "declarationMap": true,      // 生成声明文件映射
    "sourceMap": true,            // 生成源码映射
    "noUnusedLocals": true,      // 检查未使用的局部变量
    "noUnusedParameters": true,  // 检查未使用的参数
    "noImplicitReturns": true,   // 检查代码路径的返回值
    "noFallthroughCasesInSwitch": true, // switch 必须穷举
    
    /* 实验性功能 */
    "experimentalDecorators": true,  // 启用装饰器
    "emitDecoratorMetadata": true    // 生成装饰器元数据
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 编译模式

TypeScript 支持两种编译模式：

```json
// 完整编译（默认）
{
  "compilerOptions": {
    "composite": false,
    "incremental": false
  }
}

// 增量编译（适合大型项目）
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo"
  }
}
```

### 项目引用

对于 monorepo 项目，项目引用可以提升编译效率：

```json
// workspace/package-a/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"]
}

// workspace/tsconfig.json (根配置)
{
  "references": [
    { "path": "./package-a" }
  ],
  "files": []
}
```

---

## Zod 运行时验证

### 为什么需要 Zod

TypeScript 的类型检查仅在编译阶段有效，运行时仍然可能收到不符合预期的数据。Zod 提供了运行时的类型验证能力，与 TypeScript 类型系统完美配合。

```typescript
import { z } from "zod";

// 定义 schema
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().positive().max(150),
  role: z.enum(["admin", "user", "guest"]),
  createdAt: z.string().datetime(),
  metadata: z.record(z.string(), z.unknown()).optional()
});

// 从 schema 推断 TypeScript 类型
type User = z.infer<typeof UserSchema>;

// 验证数据
function createUser(data: unknown): User {
  return UserSchema.parse(data); // 验证通过返回 User 类型，失败抛出 ZodError
}

// 处理验证错误
try {
  const user = createUser({ /* ... */ });
} catch (error) {
  if (error instanceof z.ZodError) {
    console.log("验证错误:", error.errors);
  }
}
```

### Zod 与 AI 输出的结合

Zod 在 AI 应用中特别有用，用于验证大模型的输出：

```typescript
import { z } from "zod";
import { anthropic } from "@ai-sdk/anthropic";

// 定义输出 schema
const ArticleSchema = z.object({
  title: z.string().describe("文章标题"),
  summary: z.string().min(50).max(200).describe("50-200字的摘要"),
  tags: z.array(z.string()).min(1).max(5).describe("1-5个标签"),
  category: z.enum(["technology", "business", "lifestyle", "science"]),
  publishDate: z.string().describe("发布日期 YYYY-MM-DD")
});

// 使用 AI SDK 进行结构化输出
const model = anthropic("claude-sonnet-4-20250514");

const { object } = await model.outputSchema(ArticleSchema);

const result = await generateText({
  model,
  prompt: "写一篇关于 AI 发展的文章",
  output: object
});

// result 已经经过 Zod 验证
console.log(result.title);    // 类型安全
console.log(result.tags);    // string[] | undefined
```

### 高级 Zod 用法

```typescript
import { z } from "zod";

// 递归类型
const TreeNodeSchema: z.ZodType<any> = z.lazy(() =>
  z.object({
    value: z.string(),
    children: z.array(TreeNodeSchema).optional()
  })
);

// 变换和预处理
const SanitizedString = z.string().transform(s => s.trim().toLowerCase());
const Age = z.coerce.number().int().min(0).max(150);

// 联合 schema
const ResponseSchema = z.union([
  z.object({ type: z.literal("success"), data: z.any() }),
  z.object({ type: z.literal("error"), message: z.string() })
]);

// Map 和 Set
const StringMapSchema = z.map(z.string(), z.number());
const UniqueNamesSchema = z.set(z.string());

// 可选链式验证
const ConfigSchema = z.object({
  database: z.object({
    host: z.string(),
    port: z.number().default(5432),
    ssl: z.boolean().default(false)
  }),
  cache: z.object({
    enabled: z.boolean(),
    ttl: z.number().optional()
  }).optional()
}).deepStrictness();
```

---

## AI 编程中的 TypeScript 优势

### 全栈类型安全

在 AI 应用中，TypeScript 可以在整个技术栈中保持类型一致：

```typescript
// shared/types.ts - 前后端共享类型
export interface AIRequest {
  model: string;
  messages: Message[];
  temperature?: number;
  maxTokens?: number;
}

export interface AIResponse {
  id: string;
  model: string;
  content: string;
  usage: {
    inputTokens: number;
    outputTokens: number;
  };
}

// 前端使用
async function sendToAI(request: AIRequest): Promise<AIResponse> {
  const response = await fetch("/api/ai", {
    method: "POST",
    body: JSON.stringify(request)
  });
  return response.json();
}

// 后端验证
import { AIRequest } from "./shared/types";
import { AIRequestSchema } from "./shared/validation";

app.post("/api/ai", async (req, res) => {
  const rawRequest = req.body;
  
  // 使用 Zod 验证请求
  const validationResult = AIRequestSchema.safeParse(rawRequest);
  if (!validationResult.success) {
    return res.status(400).json({ error: validationResult.error });
  }
  
  const request: AIRequest = validationResult.data;
  // 处理请求...
});
```

### 工具定义与类型推断

```typescript
import { z } from "zod";

// 定义 AI 工具
const GetWeatherTool = {
  name: "get_weather",
  description: "获取指定城市的天气信息",
  parameters: z.object({
    city: z.string().describe("城市名称"),
    unit: z.enum(["celsius", "fahrenheit"]).optional().describe("温度单位")
  })
} as const;

// 从工具定义推断类型
type GetWeatherParams = z.infer<typeof GetWeatherTool.parameters>;

// 工具执行函数
async function executeTool(params: GetWeatherParams) {
  // TypeScript 自动知道 params 的类型
  const { city, unit = "celsius" } = params;
  // ...
}
```

### Prompt 模板类型安全

```typescript
type PromptTemplate<T extends Record<string, unknown> = Record<string, unknown>> = {
  system: string;
  user: (params: T) => string;
  format?: z.ZodType; // 输出格式 schema
};

interface ArticleParams {
  topic: string;
  wordCount: number;
  style: "formal" | "casual";
}

const articlePrompt: PromptTemplate<ArticleParams> = {
  system: "你是一位专业的内容创作者。",
  user: ({ topic, wordCount, style }) => 
    `请写一篇关于 ${topic} 的文章，字数约 ${wordCount} 字，风格 ${style === "formal" ? "正式" : "轻松"}。`
};
```

---

## 实战场景与最佳实践

### 场景一：AI Agent 工具调用

```typescript
import { z } from "zod";

// 定义工具集
const tools = {
  search_database: {
    schema: z.object({
      query: z.string().describe("搜索关键词"),
      limit: z.number().optional().default(10)
    }),
    handler: async (params: { query: string; limit: number }) => {
      // 实现搜索逻辑
      return await db.search(params.query, { limit: params.limit });
    }
  },
  
  send_email: {
    schema: z.object({
      to: z.string().email(),
      subject: z.string().min(1).max(200),
      body: z.string().min(1)
    }),
    handler: async (params: { to: string; subject: string; body: string }) => {
      // 实现发送邮件逻辑
      return await emailService.send(params);
    }
  },
  
  create_task: {
    schema: z.object({
      title: z.string().min(1).max(200),
      description: z.string().optional(),
      dueDate: z.string().datetime().optional(),
      priority: z.enum(["low", "medium", "high"]).default("medium")
    }),
    handler: async (params: any) => {
      return await taskService.create(params);
    }
  }
} as const;

// 类型安全的工具调用
type ToolName = keyof typeof tools;
type ToolParams<T extends ToolName> = z.infer<typeof tools[T]["schema"]>;

async function callTool<T extends ToolName>(
  toolName: T,
  params: ToolParams<T>
): Promise<unknown> {
  const tool = tools[toolName];
  const validatedParams = tool.schema.parse(params);
  return await tool.handler(validatedParams);
}

// 使用示例
const results = await Promise.all([
  callTool("search_database", { query: "AI agent", limit: 5 }),
  callTool("create_task", { 
    title: "研究 AI Agent 架构",
    priority: "high"
  })
]);
```

### 场景二：流式响应处理

```typescript
// 流式响应类型定义
interface StreamChunk<T> {
  type: "content" | "tool_call" | "done";
  content?: string;
  toolCall?: {
    name: string;
    arguments: Record<string, unknown>;
  };
  final?: T;
}

async function* streamAIResponse<T>(
  stream: ReadableStream<Uint8Array>,
  schema?: z.ZodType<T>
): AsyncGenerator<StreamChunk<T>, T | undefined, undefined> {
  const reader = stream.getReader();
  const decoder = new TextDecoder();
  let buffer = "";
  let finalResult: T | undefined;

  try {
    while (true) {
      const { done, value } = await reader.read();
      
      if (done) break;
      
      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split("\n");
      buffer = lines.pop() || "";
      
      for (const line of lines) {
        if (line.startsWith("data: ")) {
          const data = JSON.parse(line.slice(6));
          
          if (data.type === "content_delta") {
            yield { type: "content", content: data.content };
          } else if (data.type === "tool_call") {
            yield { 
              type: "tool_call", 
              toolCall: data.tool_call 
            };
          } else if (data.type === "done") {
            if (schema && data.output) {
              finalResult = schema.parse(data.output);
            }
            yield { type: "done", final: finalResult };
          }
        }
      }
    }
  } finally {
    reader.releaseLock();
  }

  return finalResult;
}

// 使用示例
const outputSchema = z.object({
  summary: z.string(),
  keywords: z.array(z.string()),
  sentiment: z.enum(["positive", "neutral", "negative"])
});

for await (const chunk of streamAIResponse(responseStream, outputSchema)) {
  if (chunk.type === "content" && chunk.content) {
    process.stdout.write(chunk.content);
  } else if (chunk.type === "tool_call") {
    console.log("Tool call:", chunk.toolCall);
  } else if (chunk.type === "done") {
    console.log("Final result:", chunk.final);
  }
}
```

### 场景三：错误处理与重试

```typescript
interface RetryConfig {
  maxAttempts: number;
  initialDelay: number;
  maxDelay: number;
  backoffMultiplier: number;
}

class AIError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly isRetryable: boolean
  ) {
    super(message);
    this.name = "AIError";
  }
}

async function withRetry<T>(
  fn: () => Promise<T>,
  config: RetryConfig,
  onRetry?: (attempt: number, error: AIError) => void
): Promise<T> {
  let lastError: AIError;
  let delay = config.initialDelay;

  for (let attempt = 1; attempt <= config.maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error instanceof AIError 
        ? error 
        : new AIError(
            error instanceof Error ? error.message : "Unknown error",
            "UNKNOWN",
            false
          );

      if (!lastError.isRetryable || attempt === config.maxAttempts) {
        throw lastError;
      }

      if (onRetry) {
        onRetry(attempt, lastError);
      }

      // 指数退避
      await new Promise(resolve => setTimeout(resolve, delay));
      delay = Math.min(delay * config.backoffMultiplier, config.maxDelay);
    }
  }

  throw lastError!;
}

// 使用示例
const result = await withRetry(
  () => callAI(prompt),
  {
    maxAttempts: 3,
    initialDelay: 1000,
    maxDelay: 10000,
    backoffMultiplier: 2
  },
  (attempt, error) => {
    console.log(`Attempt ${attempt} failed: ${error.message}`);
  }
);
```

---

## TypeScript 高级类型技巧

### 模板字面量类型

```typescript
// 模板字面量基础
type EventName = 'click' | 'focus' | 'blur';
type Handler = `on${Capitalize<EventName>}`;
// Handler = 'onClick' | 'onFocus' | 'onBlur'

// 字符串操作
type Trim<T extends string> = T extends ` ${infer U} ` ? Trim<U> : T;
type Trimmed = Trim<'  hello  '>; // 'hello'

// 路径类型
type Path = `/api/${string}`;
type ValidPath = '/api/users' extends Path ? true : false; // true

// 解析 URL
type ParseURL<P extends string> = 
  P extends `${infer Protocol}://${infer Host}:${infer Port}/${infer Path}`
    ? { protocol: Protocol; host: Host; port: Port; path: Path }
    : never;

type Parsed = ParseURL<'https://api.example.com:443/users'>;
// { protocol: 'https'; host: 'api.example.com'; port: '443'; path: 'users' }
```

### 递归类型

```typescript
// 深递归类型
type DeepReadonly<T> = T extends (infer U)[]
  ? DeepReadonly<U>[]
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

// 深可选
type DeepPartial<T> = T extends object
  ? { [K in keyof T]?: DeepPartial<T[K]> }
  : T;

// JSON 解析类型
type JSONValue = string | number | boolean | null | JSONValue[] | { [key: string]: JSONValue };

// 树形结构
interface TreeNode<T> {
  value: T;
  children: TreeNode<T>[];
}

// 柯里化类型
type Curried<T extends unknown[], R> = 
  T extends []
    ? R
    : T extends [infer First, ...infer Rest]
    ? (arg: First) => Curried<Rest, R>
    : never;
```

### 条件类型进阶

```typescript
// 分布式条件类型
type IsArray<T> = T extends any[] ? true : false;
type A = IsArray<string[]>; // true
type B = IsArray<number>; // false

// 类型过滤
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;

type C = Exclude<'a' | 'b' | 'c', 'a'>; // 'b' | 'c'
type D = Extract<'a' | 'b' | 'c', 'a' | 'b'>; // 'a' | 'b'

// infer 用法
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type E = ReturnType<() => number>; // number

type Parameters<T> = T extends (...args: infer P) => any ? P : never;
type F = Parameters<(a: string, b: number) => void>; // [string, number]
```

### 映射类型

```typescript
// 基础映射
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Person {
  name: string;
  age: number;
}
type PersonGetters = Getters<Person>;
// { getName: () => string; getAge: () => number }

// 键过滤
type RemoveKind<T> = {
  [K in keyof T as K extends 'kind' ? never : K]: T[K]
};

// 只读映射
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};
```

### symbol 与 unique symbol

```typescript
// unique symbol 用于真正的常量
declare const __brand: unique symbol;
type Brand<T, B extends string> = T & { [__brand]: B };

type UserId = Brand<string, 'UserId'>;
type PostId = Brand<string, 'PostId'>;

function getUser(id: UserId): User {}
function getPost(id: PostId): Post {}

// 这会报错 - 类型不同
// getUser('123' as PostId); // Error!

// symbol 属性
type SymbolProps<T> = {
  [K in keyof T]: T[K] extends symbol ? K : never;
}[keyof T];
```

---

## TypeScript 编译器 API

### 代码转换基础

```typescript
import * as ts from 'typescript';

function transformCode(source: string): string {
  // 创建源码文件
  const sourceFile = ts.createSourceFile(
    'input.ts',
    source,
    ts.ScriptTarget.Latest,
    true
  );

  // 创建转换器
  const result = ts.transform([sourceFile], [
    (context) => (node) => {
      return ts.visitEachChild(node, (child) => {
        // 转换逻辑
        if (ts.isStringLiteral(child)) {
          return ts.factory.createStringLiteral(
            child.text.toUpperCase()
          );
        }
        return child;
      }, context);
    }
  ], {
    target: ts.ScriptTarget.ES2022,
    module: ts.ModuleKind.ESNext,
  });

  // 输出代码
  const printer = ts.createPrinter();
  const output = printer.printFile(result.transformed[0]);
  
  return output;
}
```

### AST 遍历与修改

```typescript
function analyzeAndTransform(source: string): string {
  const sourceFile = ts.createSourceFile(
    'source.ts',
    source,
    ts.ScriptTarget.ES2022,
    true
  );

  function visit(node: ts.Node): ts.Node {
    // 查找 console.log 并标记
    if (ts.isCallExpression(node)) {
      const expr = node.expression;
      if (ts.isPropertyAccessExpression(expr) &&
          expr.expression.getText() === 'console' &&
          expr.name.text === 'log') {
        // 替换为带前缀的版本
        return ts.factory.updateCallExpression(
          node,
          ts.factory.createPropertyAccessExpression(
            ts.factory.createIdentifier('console'),
            'warn'
          ),
          node.typeArguments,
          node.arguments
        );
      }
    }
    return ts.visitEachChild(node, visit, null);
  }

  const transformed = ts.visitNode(sourceFile, visit);
  const printer = ts.createPrinter();
  return printer.printFile(transformed);
}
```

### 项目引用与增量编译

```typescript
// tsconfig.project.json
{
  "files": [],
  "references": [
    { "path": "./packages/shared", "prepend": true },
    { "path": "./packages/core" },
    { "path": "./packages/app" }
  ]
}

// 增量编译 API
const host = ts.createIncrementalCompilerHost(
  ts.readConfigFile('tsconfig.json', ts.sys.readFile).config
);

const program = ts.createIncrementalProgram({
  host,
  rootNames: ['src/main.ts'],
  configFileParsingDiagnostics: []
});

const emitResult = program.emit();
```

---

## TypeScript 与 AI 工具集成

### OpenAI Structured Output

```typescript
import OpenAI from 'openai';
import { z } from 'zod';
import { zodResponseFormat } from 'openai/helpers/zod';

const client = new OpenAI();

const schema = z.object({
  name: z.string().describe('User full name'),
  email: z.string().email().describe('User email address'),
  age: z.number().optional().describe('User age in years'),
  interests: z.array(z.string()).describe('User interests or hobbies'),
});

async function extractUserInfo(text: string) {
  const response = await client.responses.create({
    model: 'gpt-4o',
    input: `Extract user information from: ${text}`,
    response_format: zodResponseFormat(schema, 'user'),
  });

  return schema.parse(response.output_parsed);
}

const user = await extractUserInfo(
  'John Doe is 30 years old, his email is john@example.com. He likes reading, hiking, and cooking.'
);
// { name: 'John Doe', email: 'john@example.com', age: 30, interests: [...] }
```

### LangChain 类型集成

```typescript
import { ChatOpenAI } from '@langchain/openai';
import { z } from 'zod';

const llm = new ChatOpenAI({
  model: 'gpt-4-turbo',
  temperature: 0,
});

const schema = z.object({
  sentiment: z.enum(['positive', 'negative', 'neutral']),
  score: z.number().min(0).max(100),
  summary: z.string(),
  keyPhrases: z.array(z.string()),
});

const chain = llm.withStructuredOutput(schema);

const result = await chain.invoke(
  'I absolutely love this product! It exceeded all my expectations.'
);
// { sentiment: 'positive', score: 95, summary: '...', keyPhrases: [...] }
```

### Zod 与 API 验证

```typescript
import { z } from 'zod';
import { fromZodError } from 'zod-validation-error';

const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  metadata: z.record(z.string()).optional(),
});

const ApiResponseSchema = z.object({
  success: z.boolean(),
  data: UserSchema,
  timestamp: z.string().datetime(),
});

// 在 API 处理中使用
async function handleUserRequest(body: unknown) {
  try {
    const validated = ApiResponseSchema.parse(body);
    // validated 是完全类型化的
    return { status: 200, data: validated.data };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return {
        status: 400,
        error: fromZodError(error).message,
      };
    }
    throw error;
  }
}
```

---

## TypeScript 性能优化

### 类型检查优化

```typescript
// tsconfig.json 优化配置
{
  "compilerOptions": {
    // 增量编译
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",
    
    // 跳过库检查
    "skipLibCheck": true,
    
    // 仅报告错误，不阻止构建
    "noEmitOnError": false,
    
    // 高级优化
    "useDefineForClassFields": true,
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    
    // 类型宽松度（根据项目调整）
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

### 运行时性能

```typescript
// 避免类型守卫开销
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

// 使用 const 断言
const CONFIG = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
} as const;
// CONFIG.apiUrl 现在是字面量类型 'https://api.example.com'

// Map 替代对象查找
const STATUS_MAP = {
  PENDING: 'pending',
  ACTIVE: 'active',
  COMPLETED: 'completed',
} as const;

type Status = typeof STATUS_MAP[keyof typeof STATUS_MAP];

function getStatusDisplay(status: Status): string {
  return STATUS_MAP[status]; // TypeScript 知道这总是返回 string
}
```

---

## TypeScript 项目最佳实践

### Monorepo 结构

```
my-monorepo/
├── package.json
├── tsconfig.base.json
├── packages/
│   ├── shared/          # 共享类型和工具
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── api/             # 后端 API
│   │   ├── package.json
│   │   └── tsconfig.json
│   └── web/             # 前端应用
│       ├── package.json
│       └── tsconfig.json
└── pnpm-workspace.yaml
```

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "strict": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "composite": true
  }
}
```

### 路径别名配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@shared/*": ["packages/shared/src/*"],
      "@api/*": ["packages/api/src/*"],
      "@web/*": ["packages/web/src/*"],
      "@/*": ["src/*"]
    }
  }
}

// vite.config.ts
import { resolve } from 'path';
export default defineConfig({
  resolve: {
    alias: {
      '@shared': resolve(__dirname, 'packages/shared/src'),
      '@': resolve(__dirname, 'src'),
    }
  }
});
```

---

## TypeScript 测试与调试

### Vitest 配置

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/']
    },
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    testTimeout: 10000
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
});

// src/utils.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';

describe('String Utils', () => {
  it('should capitalize string', () => {
    const result = 'hello'.toUpperCase();
    expect(result).toBe('HELLO');
  });
});

describe('API Client', () => {
  vi.mock('../api', () => ({
    fetchData: vi.fn().mockResolvedValue({ data: 'test' })
  }));

  it('should handle async operations', async () => {
    const result = await Promise.resolve({ success: true });
    expect(result.success).toBe(true);
  });
});
```

### Mock 函数

```typescript
// 创建 mock 函数
const mockFn = vi.fn();

// 设置返回值
mockFn.mockReturnValue(42);
mockFn.mockResolvedValue({ data: 'success' });
mockFn.mockImplementation((x: number) => x * 2);

// 链式调用
mockFn
  .mockReturnValueOnce(1)
  .mockReturnValueOnce(2)
  .mockReturnValueOnce(3)
  .mockReturnValue(0);

// 断言调用
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledWith(5);
expect(mockFn).toHaveBeenCalledTimes(3);

// 模拟模块
vi.mock('./database', () => ({
  query: vi.fn().mockResolvedValue([{ id: 1, name: 'Test' }]),
  insert: vi.fn().mockResolvedValue({ id: 2 })
}));

// 模拟定时器
vi.useFakeTimers();
setTimeout(() => console.log('delayed'), 1000);
vi.runAllTimers();
vi.useRealTimers();
```

### 类型测试

```typescript
// 类型级别测试
type Equals<X, Y> =
  (<T>() => T extends X ? 1 : 2) extends
  (<T>() => T extends Y ? 1 : 2) ? true : false;

type Assert<T extends true> = T;

// 使用
type _test1 = Assert<Equals<number[], number[]>>; // OK
type _test2 = Assert<Equals<number, string>>; // Error
```

---

## TypeScript 运行时类型检查

### Zod 验证

```typescript
import { z } from 'zod';
import { fromZodError } from 'zod-validation-error';

// 定义 schema
const UserSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().optional(),
  role: z.enum(['admin', 'user', 'guest']),
  metadata: z.record(z.string()).optional(),
  createdAt: z.string().datetime(),
  tags: z.array(z.string()).min(1)
});

const ConfigSchema = z.object({
  apiKey: z.string().min(32),
  endpoint: z.string().url(),
  timeout: z.number().min(1000).max(60000).default(5000),
  retries: z.number().int().min(0).max(5).default(3),
  features: z.object({
    darkMode: z.boolean().default(false),
    analytics: z.boolean().default(true)
  })
});

// 类型推导
type User = z.infer<typeof UserSchema>;
type Config = z.infer<typeof ConfigSchema>;

// 验证函数
function validateUser(data: unknown): User {
  const result = UserSchema.safeParse(data);
  if (!result.success) {
    throw new Error(fromZodError(result.error).message);
  }
  return result.data;
}

// API 中间件
import { Request, Response, NextFunction } from 'express';

function validateBody<T>(schema: z.ZodSchema<T>) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(400).json({
        error: fromZodError(result.error).message
      });
    }
    req.body = result.data;
    next();
  };
}

app.post('/users', validateBody(UserSchema), (req, res) => {
  // req.body 已被验证为 User 类型
  res.json({ success: true });
});
```

### 运行时类型守卫

```typescript
// 运行时类型检查
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !Number.isNaN(value);
}

function isArray<T>(value: unknown, guard: (v: unknown) => v is T): value is T[] {
  return Array.isArray(value) && value.every(guard);
}

function isObject(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

// 复杂类型守卫
const isUser = (value: unknown): value is User => {
  return isObject(value) &&
    isString(value.id) &&
    isString(value.email) &&
    isString(value.name);
};

// 链式验证
function parseUser(data: unknown): User | null {
  if (!isObject(data)) return null;
  if (!isString(data.id) || !isString(data.email)) return null;
  return data as User;
}
```

---

## TypeScript 工程化实践

### Monorepo 配置

```json
// package.json (workspace root)
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": ["packages/*"],
  "scripts": {
    "dev": "pnpm -r --parallel dev",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint"
  }
}

// packages/shared/package.json
{
  "name": "@my/shared",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch"
  }
}

// packages/api/package.json
{
  "name": "@my/api",
  "dependencies": {
    "@my/shared": "workspace:*"
  }
}
```

### 构建优化

```json
// tsconfig.build.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "noEmit": false,
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "exclude": ["src/**/*.test.ts", "dist/**/*"]
}
```

### 代码生成

```typescript
// scripts/codegen.ts
import { parse, parseSchema } from '@graphql-tools/schema';
import { generate } from '@graphql-codegen/cli';

async function generateTypes() {
  const schema = parse(`
    type Query {
      users: [User!]!
      user(id: ID!): User
    }
    
    type User {
      id: ID!
      name: String!
      email: String!
    }
  `);

  console.log('Schema parsed successfully');
  // 使用 codegen 配置生成类型
}

// 运行: npx ts-node scripts/codegen.ts
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| TypeScript 官方文档 | https://www.typescriptlang.org/docs/ |
| TypeScript Handbook | https://www.typescriptlang.org/docs/handbook/ |
| Type Challenges | https://github.com/type-challenges/type-challenges |
| Zod | https://zod.dev |
| ts-pattern | https://github.com/gvergnaud/ts-pattern |
| TypeScript Deep Dive | https://basarat.gitbook.io/typescript/ |
| Total TypeScript | https://www.totaltypescript.com/ |

---

## 附录：TypeScript 高级配置与生态工具

### TypeScript 项目模板

```json
// tsconfig.json - 完整配置模板
{
  "compilerOptions": {
    /* 基础配置 */
    "target": "ES2022",
    "module": "NodeNext",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "outDir": "./dist",
    "rootDir": "./src",
    "removeComments": true,
    "noEmitOnError": true,

    /* 模块解析 */
    "moduleResolution": "NodeNext",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "allowSyntheticDefaultImports": true,

    /* 严格模式 - 必须开启 */
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,

    /* 额外检查 */
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,

    /* 路径配置 */
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"],
      "@components/*": ["components/*"],
      "@hooks/*": ["hooks/*"],
      "@utils/*": ["utils/*"],
      "@types/*": ["types/*"]
    },

    /* 输出配置 */
    "declaration": true,
    "declarationDir": "./dist/types",
    "declarationMap": true,
    "sourceMap": true,
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",

    /* 实验性功能 */
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,

    /* JSX 配置（React 项目） */
    "jsx": "react-jsx",
    "jsxImportSource": "react"
  },
  "include": ["src/**/*", "types/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### ESLint + Prettier 配置

```javascript
// .eslintrc.js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: 'module',
    project: './tsconfig.json',
  },
  plugins: ['@typescript-eslint', 'import'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended-type-checked',
    'plugin:@typescript-eslint/stylistic-type-checked',
    'plugin:import/recommended',
    'plugin:import/typescript',
  ],
  rules: {
    '@typescript-eslint/no-unused-vars': ['error', { 
      argsIgnorePattern: '^_',
      varsIgnorePattern: '^_'
    }],
    '@typescript-eslint/consistent-type-imports': 'error',
    '@typescript-eslint/no-floating-promises': 'error',
    'import/order': ['error', {
      groups: ['builtin', 'external', 'internal', 'parent', 'sibling', 'index'],
      'newlines-between': 'always',
      alphabetize: { order: 'asc' }
    }]
  },
  settings: {
    'import/resolver': {
      typescript: {
        project: './tsconfig.json'
      }
    }
  }
};
```

```javascript
// .prettierrc
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "bracketSameLine": false
}
```

### Vitest + React Testing Library 配置

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      exclude: [
        'node_modules/',
        'dist/',
        '**/*.d.ts',
        '**/*.test.ts',
        '**/*.spec.ts',
        'src/test/**'
      ]
    },
    testTimeout: 10000,
    hookTimeout: 10000,
    globals: true
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@types': path.resolve(__dirname, './src/types')
    }
  }
});
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom';
import { vi } from 'vitest';

// Mock IntersectionObserver
Object.defineProperty(window, 'IntersectionObserver', {
  writable: true,
  value: vi.fn().mockImplementation((callback) => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn()
  }))
});

// Mock ResizeObserver
Object.defineProperty(window, 'ResizeObserver', {
  writable: true,
  value: vi.fn().mockImplementation(() => ({
    observe: vi.fn(),
    unobserve: vi.fn(),
    disconnect: vi.fn()
  }))
});
```

### tsup 构建配置

```typescript
// tsup.config.ts
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm', 'iife'],
  dts: true,
  sourcemap: true,
  clean: true,
  external: ['react', 'react-dom'],
  env: {
    NODE_ENV: 'production'
  },
  treeshake: {
    moduleSideEffects: 'no-external'
  },
  splitting: true,
  minify: 'terser'
});
```

### TypeScript 类型体操技巧

```typescript
// 1. 深readonly
type DeepReadonly<T> = 
  T extends (infer U)[] 
    ? ReadonlyArray<DeepReadonly<U>>
    : T extends object 
    ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
    : T;

// 2. 深可选
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

// 3. 深必需
type DeepRequired<T> = {
  [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};

// 4. 解构函数参数
type Destructor<T extends (...args: any[]) => any> = 
  T extends (...args: infer P) => infer R 
    ? (...args: { [K in keyof P]-?: DeepRequired<P[K]> }) => R
    : never;

// 5. 安全的 Promise 返回类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

// 6. 提取数组类型
type ArrayElement<T> = T extends readonly (infer U)[] ? U : never;

// 7. 互斥属性类型
type Exclusive<T, U> = T | ({ [K in keyof U]?: never });

// 示例：loginType 只能是 'email' 或 'phone'，不能同时有两个
type LoginConfig = 
  | { loginType: 'email'; email: string }
  | { loginType: 'phone'; phone: string };

// 8. 递归 JSON 类型
type JSONPrimitive = string | number | boolean | null;
type JSONValue = JSONPrimitive | JSONValue[] | { [key: string]: JSONValue };
type JSONSchema<T> = {
  [K in keyof T]: T[K] extends JSONValue 
    ? T[K] 
    : never;
};

// 9. Extract 函数参数类型
type FunctionParams<T extends (...args: any) => any> = 
  T extends (...args: infer P) => any ? P : never;

// 10. 联合类型转交叉类型
type UnionToIntersection<U> = 
  (U extends any ? (k: U) => void : never) extends ((k: infer I) => void) ? I : never;
```

### 实用类型工具函数

```typescript
// 运行时类型检查工具
type TypeGuard<T> = (value: unknown) => value is T;

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number' && !Number.isNaN(value);
}

function isBoolean(value: unknown): value is boolean {
  return typeof value === 'boolean';
}

function isArray<T>(
  value: unknown,
  guard?: TypeGuard<T>
): value is T[] {
  if (!Array.isArray(value)) return false;
  if (guard) return value.every(guard);
  return true;
}

function isObject<T extends object>(
  value: unknown,
  guard?: TypeGuard<T>
): value is T {
  if (typeof value !== 'object' || value === null) return false;
  if (guard) return guard(value);
  return true;
}

// 深度合并工具
type DeepMerge<T, U> = {
  [K in keyof T | keyof U]: 
    K extends keyof T & keyof U 
      ? T[K] extends object 
        ? U[K] extends object 
          ? DeepMerge<T[K], U[K]> 
          : T[K]
        : U[K]
      : K extends keyof T 
      ? T[K]
      : K extends keyof U 
      ? U[K]
      : never;
};

function deepMerge<T extends object, U extends object>(
  target: T,
  source: U
): DeepMerge<T, U> {
  const result = { ...target } as DeepMerge<T, U>;
  
  for (const key in source) {
    const targetValue = (target as any)[key];
    const sourceValue = (source as any)[key];
    
    if (
      typeof targetValue === 'object' && 
      typeof sourceValue === 'object' &&
      targetValue !== null &&
      sourceValue !== null
    ) {
      (result as any)[key] = deepMerge(targetValue, sourceValue);
    } else {
      (result as any)[key] = sourceValue;
    }
  }
  
  return result;
}

// omitBy 和 pickBy
type ObjectType = Record<string, unknown>;

function omitBy<T extends ObjectType, U extends ObjectType>(
  obj: T,
  predicate: (value: T[keyof T], key: keyof T) => boolean
): Partial<T> {
  const result = {} as Partial<T>;
  
  for (const key in obj) {
    if (!predicate(obj[key], key)) {
      (result as any)[key] = obj[key];
    }
  }
  
  return result;
}

function pickBy<T extends ObjectType>(
  obj: T,
  predicate: (value: T[keyof T], key: keyof T) => boolean
): Partial<T> {
  const result = {} as Partial<T>;
  
  for (const key in obj) {
    if (predicate(obj[key], key)) {
      (result as any)[key] = obj[key];
    }
  }
  
  return result;
}

// 示例使用
interface User {
  id: string;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

const user: User = {
  id: '1',
  name: 'Alice',
  email: 'alice@example.com',
  password: 'hashed_password',
  createdAt: new Date()
};

// 移除敏感字段
const publicUser = omitBy(user, (value, key) => 
  key === 'password'
);

// 提取可选字段
const optionalFields = pickBy(user, (value) => 
  value === null || value === undefined
);
```

### 包管理脚本

```json
// package.json scripts
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "build:types": "tsc --emitDeclarationOnly",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext .ts,.tsx --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,json,css}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,json,css}\"",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:e2e": "playwright test",
    "typecheck": "tsc --noEmit",
    "typecheck:watch": "tsc --noEmit --watch",
    "prepare": "husky install",
    "commit": "git add . && cz",
    "clean": "rm -rf dist node_modules/.vite",
    "analyze": "npx vite-bundle-visualizer"
  }
}
```

### 代码质量检查脚本

```bash
#!/bin/bash
# scripts/quality-check.sh

set -e

echo "Running quality checks..."

echo "1. Type checking..."
npm run typecheck

echo "2. Linting..."
npm run lint

echo "3. Formatting check..."
npm run format:check

echo "4. Running tests..."
npm run test -- --run

echo "5. Building..."
npm run build

echo "All checks passed!"
```

### 参考资料

| 资源 | 链接 |
|------|------|
| TypeScript 官方文档 | https://www.typescriptlang.org/docs/ |
| TypeScript Handbook | https://www.typescriptlang.org/docs/handbook/ |
| Type Challenges | https://github.com/type-challenges/type-challenges |
| Zod | https://zod.dev |
| ts-pattern | https://github.com/gvergnaud/ts-pattern |
| TypeScript Deep Dive | https://basarat.gitbook.io/typescript/ |
| Total TypeScript | https://www.totaltypescript.com/ |
| Type-Fest | https://github.com/sindresorhus/type-fest |
| ts-reset | https://github.com/total-typescript/ts-reset |

---

## 常见问题与解决方案

### 问题1：类型推断不准确

**问题描述**：TypeScript 无法正确推断类型，导致 `any` 类型泛滥。

```typescript
// ❌ 常见问题
const obj: any = { a: 1, b: '2' };
const value = obj.c; // 推断为 any，没有类型检查

// ❌ 数组类型推断
const arr = []; // 推断为 never[]
arr.push(1);
arr.push('2'); // 没有报错，但类型不一致

// ❌ 函数返回类型
function process(x) { // 参数无类型
  return x.value; // 返回 any
}
```

**解决方案**：

```typescript
// ✅ 使用类型注解
const obj: { a: number; b: string } = { a: 1, b: '2' };

// ✅ 使用类型断言
const arr: (number | string)[] = [];
arr.push(1);
arr.push('2');

// ✅ 显式参数类型
function process(x: { value: any }): any {
  return x.value;
}

// ✅ 使用 satisfies 运算符（TS 4.9+）
const config = {
  database: { host: 'localhost', port: 5432 },
  cache: { enabled: true }
} satisfies Record<string, { enabled: boolean }>;

// ✅ 使用 satisfies 保持类型推断
type Color = 'red' | 'green' | 'blue';
type Config = {
  favoriteColor: Color | { red: number; green: number; blue: number };
};

const palette = {
  red: [255, 0, 0],
  green: '#00ff00',
  favoriteColor: { red: 255, green: 0, blue: 0 }
} satisfies Config;

// palette.red 被推断为 number[]，而不是 Color
```

### 问题2：泛型约束复杂

**问题描述**：泛型类型过于复杂，难以理解和维护。

```typescript
// ❌ 过度复杂的泛型
type DeepMerge<T, U> = {
  [K in keyof T | keyof U]: 
    K extends keyof T 
      ? K extends keyof U 
        ? T[K] extends U[K] 
          ? U[K] 
          : T[K] | U[K]
        : T[K]
      : K extends keyof U 
        ? U[K] 
        : never
};
```

**解决方案**：

```typescript
// ✅ 简化泛型，添加类型注释
type DeepMerge<T extends object, U extends object> = {
  [K in keyof T | keyof U]: K extends keyof T & keyof U
    ? T[K] | U[K]  // 两者都有，取并集
    : K extends keyof T
    ? T[K]          // 只有 T 有
    : K extends keyof U
    ? U[K]          // 只有 U 有
    : never;        // 不可能到达
};

// ✅ 拆分为多个简单类型
type MergeObject<T, U> = {
  [K in Exclude<keyof T, keyof U>]: T[K];
} & {
  [K in Exclude<keyof U, keyof T>]: U[K];
} & {
  [K in Extract<keyof T, keyof U>]: T[K] | U[K];
};

// ✅ 使用条件类型简化
type Flatten<T> = T extends Array<infer U> ? U : T;

// ✅ 使用 infer 提取嵌套类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;
```

### 问题3：装饰器类型不支持

**问题描述**：项目使用装饰器但 TypeScript 报错。

```typescript
// ❌ 错误
function log(target: any, name: string, descriptor: PropertyDescriptor) {
  // ...
}
```

**解决方案**：

```json
// ✅ tsconfig.json 启用装饰器支持
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    "strict": true,
    "skipLibCheck": true
  }
}
```

```typescript
// ✅ 使用装饰器工厂
function log(prefix: string) {
  return function(
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ): PropertyDescriptor {
    const original = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
      console.log(`${prefix} Calling ${propertyKey}`);
      const result = original.apply(this, args);
      console.log(`${prefix} Returned ${result}`);
      return result;
    };
    
    return descriptor;
  };
}

class Calculator {
  @log('Calculator')
  add(a: number, b: number): number {
    return a + b;
  }
}
```

### 问题4：循环类型引用

**问题描述**：类型定义中出现循环引用导致编译错误。

```typescript
// ❌ 循环类型引用
interface TreeNode {
  value: string;
  children: TreeNode[]; // 可能导致问题
}
```

**解决方案**：

```typescript
// ✅ 使用交叉类型
type TreeNode = {
  value: string;
  children: TreeNode[];
};

// ✅ 使用接口和类型别名结合
interface TreeNodeBase {
  value: string;
}
type TreeNode = TreeNodeBase & {
  children: TreeNode[];
};

// ✅ 对于递归类型，使用类型别名
type DeepReadonly<T> = T extends (infer U)[]
  ? DeepReadonlyArray<U>
  : T extends object
  ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
  : T;

interface DeepReadonlyArray<T> extends ReadonlyArray<DeepReadonly<T>> {}

// ✅ 使用 unique symbol 创建安全的树节点
const __children = Symbol('children');
interface SafeTreeNode {
  value: string;
  [__children]?: SafeTreeNode[];
}
```

### 问题5：类型与值混淆

**问题描述**：`Type 'X' is not a type but was used as a type` 错误。

```typescript
// ❌ 常见错误
const Color = {
  Red: 'red',
  Green: 'green'
};

type MyColor = Color; // ❌ Color 是值，不是类型
const c: Color.Red = 'red'; // ❌ 错误

// ❌ 枚举与类型混淆
enum Status {
  Active = 'active',
  Inactive = 'inactive'
}
const status: Status = 'active'; // ❌ 应该是 Status.Active
```

**解决方案**：

```typescript
// ✅ 使用 type alias 定义类型
type Color = 'red' | 'green' | 'blue';
const color: Color = 'red';

// ✅ 使用 enum 定义枚举
enum Status {
  Active = 'active',
  Inactive = 'inactive'
}
const status: Status = Status.Active;

// ✅ 使用 const assertion 锁定对象
const Color = {
  Red: 'red',
  Green: 'green'
} as const;

type ColorValue = typeof Color[keyof typeof Color];
// ColorValue = 'red' | 'green'

// ✅ 正确使用 typeof
const config = { port: 3000 };
type Config = typeof config; // { port: number }
```

### 问题6：严格模式下的常见错误

**问题描述**：启用 `strict` 模式后大量报错。

```typescript
// ❌ strictNullChecks 报错
function findUser(id: string): User {
  // 可能返回 undefined，但声明返回 User
  return users.find(u => u.id === id);
}

// ❌ strictFunctionTypes 报错
type Handler = (a: string) => void;
const handler: Handler = (a: string | number) => {}; // 参数类型不兼容

// ❌ noImplicitAny 报错
function process(data) { // 参数没有类型
  return data.value;
}
```

**解决方案**：

```typescript
// ✅ 使用可空类型
function findUser(id: string): User | undefined {
  return users.find(u => u.id === id);
}

// 或者使用 null
function findUser(id: string): User | null {
  return users.find(u => u.id === id) || null;
}

// ✅ 使用可选链和空值合并
const name = user?.profile?.name ?? 'Anonymous';

// ✅ 修复函数参数类型
type Handler = (a: string) => void;
const handler: Handler = (a: string) => {}; // 参数必须是 string

// ✅ 添加显式类型
function process(data: { value: string }): string {
  return data.value;
}

// ✅ 使用 unknown 代替 any
function process(data: unknown) {
  if (typeof data === 'object' && data !== null) {
    const obj = data as { value: string };
    return obj.value;
  }
  throw new Error('Invalid data');
}
```

### 问题7：模块类型导入问题

**问题描述**：`Cannot find module` 或类型未正确导出。

```typescript
// ❌ 常见问题
import { MyType } from './types'; // 可能找不到
import type { MyType } from './types'; // 更好

// ❌ 默认导出问题
export default class User {}
import User from './User'; // 没问题
import { User } from './User'; // ❌ 这是命名导出，不是默认导出
```

**解决方案**：

```typescript
// ✅ 使用 type-only 导入（TS 3.8+）
import type { User, Address } from './types';

// ✅ 混合导入
import fs from 'fs'; // 默认导入
import type { Stats } from 'fs'; // 类型导入

// ✅ 显式导出
export type { User, Address };
export { User, Address };

// ✅ 在 package.json 中配置类型
{
  "name": "my-package",
  "main": "dist/index.js",
  "types": "dist/index.d.ts"
}

// ✅ 使用 declarations.d.ts 声明模块
declare module 'my-module' {
  export interface Config {
    name: string;
  }
  export function init(): void;
}
```

### 问题8：接口与类型别名选择

**问题描述**：不确定何时使用 `interface` 或 `type`。

```typescript
// ✅ 经验法则
// interface: 定义对象结构，可能被扩展或实现
interface User {
  id: string;
  name: string;
}

interface Admin extends User {
  permissions: string[];
}

// ✅ type: 联合类型、元组、工具类型
type Status = 'active' | 'inactive';
type Point = [number, number];
type PartialUser = Partial<User>;

// ✅ 两者都可以用于函数类型
interface Handler {
  (data: User): void;
}

type Handler = (data: User) => void;

// ✅ 声明合并（仅 interface）
interface User {
  id: string;
}
interface User {
  name: string; // 合并到同一个 User
}

// ✅ 工具类型需要 type
type Readonly<T> = { readonly [P in keyof T]: T[P] };
```

### 问题9：性能问题排查

**问题描述**：TypeScript 编译时间过长。

**解决方案**：

```json
// ✅ tsconfig.json 优化
{
  "compilerOptions": {
    "incremental": true,
    "skipLibCheck": true,
    "noEmit": false,
    "isolatedModules": true
  },
  "exclude": [
    "node_modules",
    "dist",
    "**/*.test.ts"
  ],
  "include": [
    "src/**/*"
  ]
}
```

```bash
# ✅ 使用 swc 或 esbuild 加速构建
npm install --save-dev @swc/cli @swc/core

# ✅ 使用 tsc --noEmit 进行类型检查（不生成文件）
npm run typecheck

# ✅ 使用 Project References
# tsconfig.json
{
  "files": [],
  "references": [
    { "path": "./packages/shared" },
    { "path": "./packages/core" }
  ]
}
```

### 问题10：第三方库类型缺失

**问题描述**：使用的库没有类型定义。

**解决方案**：

```bash
# ✅ 安装类型定义
npm install @types/lodash

# ✅ 创建自定义类型声明
# src/types/custom.d.ts
declare module 'my-lib' {
  export interface MyLibOptions {
    debug?: boolean;
    timeout?: number;
  }
  
  export function init(options: MyLibOptions): void;
}

// ✅ 使用 declare module 扩展现有类型
declare global {
  namespace Express {
    interface Request {
      userId?: string;
    }
  }
}

// ✅ 使用 satisfies 验证配置
import type { MyLibOptions } from 'my-lib';

const options = {
  debug: true,
  timeout: 5000
} satisfies MyLibOptions;
```

---

## 实战项目示例

### 项目一：类型安全的 API 客户端

**功能特性**：

- 端到端类型安全
- 请求/响应拦截器
- 自动错误处理
- 请求取消

```typescript
// src/types/api.ts
export interface ApiConfig {
  baseUrl: string;
  timeout?: number;
  headers?: Record<string, string>;
}

export interface RequestOptions<T = unknown> {
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  path: string;
  params?: Record<string, string>;
  body?: T;
  headers?: Record<string, string>;
}

export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}

export interface ApiError {
  code: string;
  message: string;
  details?: Record<string, string[]>;
}

export type RequestInterceptor = (
  config: RequestOptions
) => RequestOptions | Promise<RequestOptions>;

export type ResponseInterceptor = <T>(
  response: ApiResponse<T>
) => ApiResponse<T> | Promise<ApiResponse<T>>;

export type ErrorInterceptor = (
  error: ApiError
) => void | Promise<void>;
```

```typescript
// src/api/client.ts
import type {
  ApiConfig,
  RequestOptions,
  ApiResponse,
  ApiError,
  RequestInterceptor,
  ResponseInterceptor,
  ErrorInterceptor
} from './types';

export class ApiClient {
  private config: ApiConfig;
  private requestInterceptors: RequestInterceptor[] = [];
  private responseInterceptors: ResponseInterceptor[] = [];
  private errorInterceptors: ErrorInterceptor[] = [];

  constructor(config: ApiConfig) {
    this.config = {
      timeout: 30000,
      ...config
    };
  }

  addRequestInterceptor(interceptor: RequestInterceptor): void {
    this.requestInterceptors.push(interceptor);
  }

  addResponseInterceptor(interceptor: ResponseInterceptor): void {
    this.responseInterceptors.push(interceptor);
  }

  addErrorInterceptor(interceptor: ErrorInterceptor): void {
    this.errorInterceptors.push(interceptor);
  }

  async request<T, R = unknown>(
    options: RequestOptions<T>
  ): Promise<ApiResponse<R>> {
    try {
      // 应用请求拦截器
      let config = { ...options };
      for (const interceptor of this.requestInterceptors) {
        config = await interceptor(config);
      }

      const url = this.buildUrl(config);
      const response = await this.fetchWithTimeout(url, {
        method: config.method,
        headers: this.buildHeaders(config),
        body: config.body ? JSON.stringify(config.body) : undefined
      });

      const data = await response.json();

      if (!response.ok) {
        const error: ApiError = {
          code: data.code || 'UNKNOWN_ERROR',
          message: data.message || 'An error occurred',
          details: data.details
        };
        
        for (const interceptor of this.errorInterceptors) {
          await interceptor(error);
        }
        
        throw error;
      }

      const apiResponse: ApiResponse<R> = {
        data: data.data || data,
        status: response.status,
        message: data.message || 'Success'
      };

      // 应用响应拦截器
      for (const interceptor of this.responseInterceptors) {
        return await interceptor(apiResponse);
      }

      return apiResponse;
    } catch (error) {
      if (this.isApiError(error)) {
        throw error;
      }
      
      const apiError: ApiError = {
        code: 'NETWORK_ERROR',
        message: error instanceof Error ? error.message : 'Network error'
      };
      
      for (const interceptor of this.errorInterceptors) {
        await interceptor(apiError);
      }
      
      throw apiError;
    }
  }

  async get<T>(path: string, params?: Record<string, string>): Promise<ApiResponse<T>> {
    return this.request<T>({ method: 'GET', path, params });
  }

  async post<T, R = unknown>(
    path: string,
    body?: T
  ): Promise<ApiResponse<R>> {
    return this.request<T, R>({ method: 'POST', path, body });
  }

  async put<T, R = unknown>(
    path: string,
    body?: T
  ): Promise<ApiResponse<R>> {
    return this.request<T, R>({ method: 'PUT', path, body });
  }

  async patch<T, R = unknown>(
    path: string,
    body?: T
  ): Promise<ApiResponse<R>> {
    return this.request<T, R>({ method: 'PATCH', path, body });
  }

  async delete<T>(path: string): Promise<ApiResponse<T>> {
    return this.request<T>({ method: 'DELETE', path });
  }

  private buildUrl(options: RequestOptions): string {
    let url = `${this.config.baseUrl}${options.path}`;
    
    if (options.params) {
      const searchParams = new URLSearchParams(options.params);
      url += `?${searchParams.toString()}`;
    }
    
    return url;
  }

  private buildHeaders(options: RequestOptions): HeadersInit {
    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
      ...this.config.headers,
      ...options.headers
    };
    
    return headers;
  }

  private async fetchWithTimeout(
    url: string,
    init: RequestInit
  ): Promise<Response> {
    const controller = new AbortController();
    const timeout = setTimeout(
      () => controller.abort(),
      this.config.timeout
    );

    try {
      const response = await fetch(url, {
        ...init,
        signal: controller.signal
      });
      return response;
    } finally {
      clearTimeout(timeout);
    }
  }

  private isApiError(error: unknown): error is ApiError {
    return (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      'message' in error
    );
  }
}
```

```typescript
// src/api/users.ts
import { ApiClient } from './client';
import type { ApiResponse } from './types';

export interface User {
  id: string;
  email: string;
  username: string;
  firstName?: string;
  lastName?: string;
  avatar?: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: string;
  updatedAt: string;
}

export interface CreateUserDto {
  email: string;
  username: string;
  password: string;
  firstName?: string;
  lastName?: string;
}

export interface UpdateUserDto {
  firstName?: string;
  lastName?: string;
  avatar?: string;
}

export interface PaginationParams {
  page?: number;
  limit?: number;
  sortBy?: string;
  order?: 'asc' | 'desc';
}

export interface PaginatedResponse<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
  };
}

export class UsersApi {
  constructor(private client: ApiClient) {}

  async list(
    params?: PaginationParams
  ): Promise<ApiResponse<PaginatedResponse<User>>> {
    return this.client.get<PaginatedResponse<User>>('/users', params as Record<string, string>);
  }

  async getById(id: string): Promise<ApiResponse<User>> {
    return this.client.get<User>(`/users/${id}`);
  }

  async create(data: CreateUserDto): Promise<ApiResponse<User>> {
    return this.client.post<CreateUserDto, User>('/users', data);
  }

  async update(id: string, data: UpdateUserDto): Promise<ApiResponse<User>> {
    return this.client.patch<UpdateUserDto, User>(`/users/${id}`, data);
  }

  async delete(id: string): Promise<ApiResponse<void>> {
    return this.client.delete<void>(`/users/${id}`);
  }

  async search(query: string): Promise<ApiResponse<User[]>> {
    return this.client.get<User[]>('/users/search', { q: query });
  }
}
```

```typescript
// src/api/index.ts
import { ApiClient } from './client';
import { UsersApi } from './users';
import type { ApiConfig } from './types';

// 创建 API 客户端实例
export const createApiClient = (config: ApiConfig) => {
  const client = new ApiClient(config);

  // 添加请求拦截器
  client.addRequestInterceptor(async (config) => {
    // 添加认证令牌
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers = {
        ...config.headers,
        Authorization: `Bearer ${token}`
      };
    }
    return config;
  });

  // 添加响应拦截器
  client.addResponseInterceptor(async (response) => {
    // 处理 401 错误，尝试刷新令牌
    if (response.status === 401) {
      const refreshed = await refreshToken();
      if (refreshed) {
        // 重试请求
        return response;
      }
    }
    return response;
  });

  // 添加错误拦截器
  client.addErrorInterceptor(async (error) => {
    // 记录错误日志
    console.error('API Error:', error);
    
    // 显示错误通知
    showNotification(error.message, 'error');
  });

  return {
    users: new UsersApi(client),
    client
  };
};

// 辅助函数
async function refreshToken(): Promise<boolean> {
  const refreshToken = localStorage.getItem('refresh_token');
  if (!refreshToken) return false;

  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      body: JSON.stringify({ refreshToken })
    });

    if (response.ok) {
      const { accessToken } = await response.json();
      localStorage.setItem('access_token', accessToken);
      return true;
    }
  } catch {
    // 刷新失败
  }

  localStorage.removeItem('access_token');
  localStorage.removeItem('refresh_token');
  window.location.href = '/login';
  return false;
}

function showNotification(message: string, type: 'success' | 'error'): void {
  // 实现通知逻辑
  console.log(`[${type}] ${message}`);
}
```

### 项目二：类型安全的表单验证

**功能特性**：

- Schema 定义
- 运行时验证
- 表单状态管理
- 错误消息本地化

```typescript
// src/validation/schemas.ts
import { z } from 'zod';

export const LoginSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email format'),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number')
});

export const RegisterSchema = z.object({
  email: z
    .string()
    .min(1, 'Email is required')
    .email('Invalid email format'),
  username: z
    .string()
    .min(3, 'Username must be at least 3 characters')
    .max(20, 'Username cannot exceed 20 characters')
    .regex(
      /^[a-zA-Z0-9_]+$/,
      'Username can only contain letters, numbers, and underscores'
    ),
  password: z
    .string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
  confirmPassword: z.string(),
  acceptTerms: z.boolean().refine(val => val === true, {
    message: 'You must accept the terms'
  })
}).refine(data => data.password === data.confirmPassword, {
  message: 'Passwords do not match',
  path: ['confirmPassword']
});

export const ProfileSchema = z.object({
  firstName: z.string().min(1, 'First name is required').max(50),
  lastName: z.string().min(1, 'Last name is required').max(50),
  bio: z.string().max(500, 'Bio cannot exceed 500 characters').optional(),
  avatar: z.string().url('Invalid avatar URL').optional().or(z.literal('')),
  website: z.string().url('Invalid website URL').optional().or(z.literal('')),
  location: z.string().max(100).optional(),
  birthDate: z
    .string()
    .optional()
    .refine(val => !val || !isNaN(Date.parse(val)), {
      message: 'Invalid date format'
    })
    .refine(val => {
      if (!val) return true;
      const date = new Date(val);
      const now = new Date();
      return date < now;
    }, {
      message: 'Birth date must be in the past'
    })
});

export const ArticleSchema = z.object({
  title: z
    .string()
    .min(10, 'Title must be at least 10 characters')
    .max(200, 'Title cannot exceed 200 characters'),
  content: z
    .string()
    .min(100, 'Content must be at least 100 characters'),
  excerpt: z
    .string()
    .max(300, 'Excerpt cannot exceed 300 characters')
    .optional(),
  tags: z
    .array(z.string().min(1).max(20))
    .min(1, 'At least one tag is required')
    .max(5, 'Maximum 5 tags allowed'),
  category: z.enum(['tech', 'lifestyle', 'business', 'other']),
  coverImage: z.string().url('Invalid image URL').optional(),
  published: z.boolean().default(false)
});

// 类型推断
export type LoginInput = z.infer<typeof LoginSchema>;
export type RegisterInput = z.infer<typeof RegisterSchema>;
export type ProfileInput = z.infer<typeof ProfileSchema>;
export type ArticleInput = z.infer<typeof ArticleSchema>;
```

```typescript
// src/validation/form.ts
import { z, ZodError, ZodIssue } from 'zod';

export interface FieldError {
  field: string;
  message: string;
}

export interface ValidationResult<T> {
  success: boolean;
  data?: T;
  errors?: FieldError[];
}

export class FormValidator<T extends z.ZodType> {
  constructor(private schema: T) {}

  validate(data: unknown): ValidationResult<z.infer<T>> {
    try {
      const result = this.schema.parse(data);
      return { success: true, data: result };
    } catch (error) {
      if (error instanceof ZodError) {
        const errors = this.formatErrors(error);
        return { success: false, errors };
      }
      throw error;
    }
  }

  safeValidate(data: unknown): ValidationResult<z.infer<T>> {
    const result = this.schema.safeParse(data);
    
    if (result.success) {
      return { success: true, data: result.data };
    }
    
    return { success: false, errors: this.formatErrors(result.error) };
  }

  private formatErrors(error: ZodError): FieldError[] {
    return error.errors.map(issue => ({
      field: this.getFieldPath(issue),
      message: this.translateError(issue)
    }));
  }

  private getFieldPath(issue: ZodIssue): string {
    const path = issue.path;
    if (path.length === 0) return 'general';
    return path.join('.');
  }

  private translateError(issue: ZodIssue): string {
    const messages: Record<string, string> = {
      'required': 'This field is required',
      'invalid_type': `Expected ${issue.expected}, received ${issue.received}`,
      'invalid_string': 'Invalid format',
      'too_small': `Minimum length is ${(issue as any).minimum}`,
      'too_big': `Maximum length is ${(issue as any).maximum}`,
      'custom': 'Invalid value'
    };

    return (issue.message !== '' && !issue.message.startsWith('Expected'))
      ? issue.message
      : messages[issue.code] || issue.message || 'Invalid value';
  }
}

// 使用示例
export const loginValidator = new FormValidator(LoginSchema);
export const registerValidator = new FormValidator(RegisterSchema);
export const profileValidator = new FormValidator(ProfileSchema);
export const articleValidator = new FormValidator(ArticleSchema);
```

```typescript
// src/validation/react.ts
import { useState, useCallback, useMemo } from 'react';
import type { ValidationResult, FieldError } from './form';
import type { ZodType } from 'zod';

export interface FormState<T> {
  values: Partial<T>;
  errors: Record<string, string>;
  touched: Record<string, boolean>;
  isSubmitting: boolean;
  isValid: boolean;
}

export interface UseFormOptions<T> {
  initialValues: Partial<T>;
  schema: ZodType<T, any, T>;
  validateOnChange?: boolean;
  validateOnBlur?: boolean;
  onSubmit?: (values: T) => Promise<void>;
}

export function useForm<T>({
  initialValues,
  schema,
  validateOnChange = false,
  validateOnBlur = true,
  onSubmit
}: UseFormOptions<T>) {
  const [state, setState] = useState<FormState<T>>({
    values: initialValues,
    errors: {},
    touched: {},
    isSubmitting: false,
    isValid: false
  });

  const validateField = useCallback(
    (name: string, value: unknown): string | undefined => {
      try {
        const fieldSchema = (schema as any).shape?.[name];
        if (fieldSchema) {
          fieldSchema.parse(value);
        }
        return undefined;
      } catch (error: any) {
        return error.errors?.[0]?.message || 'Invalid value';
      }
    },
    [schema]
  );

  const validateAll = useCallback((): boolean => {
    const result = schema.safeParse(state.values);
    
    if (result.success) {
      setState(prev => ({ ...prev, errors: {}, isValid: true }));
      return true;
    }

    const errors: Record<string, string> = {};
    for (const issue of result.error.errors) {
      const field = issue.path.join('.');
      if (!errors[field]) {
        errors[field] = issue.message;
      }
    }
    
    setState(prev => ({ ...prev, errors, isValid: false }));
    return false;
  }, [schema, state.values]);

  const handleChange = useCallback(
    (name: string, value: unknown) => {
      setState(prev => ({
        ...prev,
        values: { ...prev.values, [name]: value }
      }));

      if (validateOnChange) {
        const error = validateField(name, value);
        setState(prev => ({
          ...prev,
          errors: { ...prev.errors, [name]: error || '' }
        }));
      }
    },
    [validateOnChange, validateField]
  );

  const handleBlur = useCallback(
    (name: string) => {
      setState(prev => ({
        ...prev,
        touched: { ...prev.touched, [name]: true }
      }));

      if (validateOnBlur) {
        const value = state.values[name as keyof T];
        const error = validateField(name, value);
        setState(prev => ({
          ...prev,
          errors: { ...prev.errors, [name]: error || '' }
        }));
      }
    },
    [validateOnBlur, state.values, validateField]
  );

  const handleSubmit = useCallback(
    async (e: React.FormEvent) => {
      e.preventDefault();

      // 标记所有字段为已触碰
      const allTouched = Object.keys(state.values).reduce(
        (acc, key) => ({ ...acc, [key]: true }),
        {}
      );
      setState(prev => ({ ...prev, touched: allTouched }));

      const isValid = validateAll();
      if (!isValid || !onSubmit) return;

      setState(prev => ({ ...prev, isSubmitting: true }));
      
      try {
        await onSubmit(state.values as T);
      } finally {
        setState(prev => ({ ...prev, isSubmitting: false }));
      }
    },
    [state.values, validateAll, onSubmit]
  );

  const reset = useCallback(() => {
    setState({
      values: initialValues,
      errors: {},
      touched: {},
      isSubmitting: false,
      isValid: false
    });
  }, [initialValues]);

  const getFieldProps = useCallback(
    (name: string) => ({
      name,
      value: state.values[name as keyof T] ?? '',
      onChange: (e: React.ChangeEvent<HTMLInputElement>) =>
        handleChange(name, e.target.value),
      onBlur: () => handleBlur(name)
    }),
    [state.values, handleChange, handleBlur]
  );

  const getFieldError = useCallback(
    (name: string) => {
      if (!state.touched[name]) return undefined;
      return state.errors[name];
    },
    [state.touched, state.errors]
  );

  return {
    values: state.values,
    errors: state.errors,
    touched: state.touched,
    isSubmitting: state.isSubmitting,
    isValid: state.isValid,
    handleChange,
    handleBlur,
    handleSubmit,
    reset,
    getFieldProps,
    getFieldError,
    validateAll
  };
}
```

```typescript
// 使用示例
import { useForm } from './validation/react';
import { registerValidator } from './validation/schemas';

function RegisterForm() {
  const form = useForm({
    initialValues: {
      email: '',
      username: '',
      password: '',
      confirmPassword: '',
      acceptTerms: false
    },
    schema: RegisterSchema,
    validateOnBlur: true,
    validateOnChange: true,
    onSubmit: async (values) => {
      await api.register(values);
      router.push('/login');
    }
  });

  return (
    <form onSubmit={form.handleSubmit}>
      <input
        {...form.getFieldProps('email')}
        type="email"
        placeholder="Email"
      />
      {form.getFieldError('email') && (
        <span className="error">{form.getFieldError('email')}</span>
      )}
      
      <input
        {...form.getFieldProps('password')}
        type="password"
        placeholder="Password"
      />
      {form.getFieldError('password') && (
        <span className="error">{form.getFieldError('password')}</span>
      )}
      
      <button type="submit" disabled={form.isSubmitting}>
        {form.isSubmitting ? 'Submitting...' : 'Register'}
      </button>
    </form>
  );
}
```

---

## 附录：TypeScript 与 AI 工具深度集成

### Claude API 结构化输出

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

import { z } from 'zod';

// 定义 Claude 输出 Schema
const ArticleSchema = z.object({
  title: z.string().describe('吸引人的文章标题'),
  summary: z.string().min(50).max(200).describe('50-200字的摘要'),
  keyPoints: z.array(z.string()).min(3).max(5).describe('3-5个关键要点'),
  tags: z.array(z.string()).min(1).max(3).describe('1-3个标签'),
  category: z.enum(['technology', 'science', 'business', 'lifestyle', 'opinion']),
  publishDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/).describe('发布日期 YYYY-MM-DD'),
});

// 使用 Claude 的 JSON 模式
async function generateArticle(prompt: string) {
  const response = await anthropic.messages.create({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 1024,
    messages: [{
      role: 'user',
      content: prompt,
    }],
    response_format: {
      type: 'json_object',
      schema: ArticleSchema,
    },
  });

  const content = response.content[0].type === 'text' 
    ? JSON.parse(response.content[0].text) 
    : null;

  if (content) {
    // Zod 验证
    const validated = ArticleSchema.parse(content);
    return validated;
  }

  throw new Error('Invalid response format');
}
```

### Cursor IDE 规则配置

```typescript
// .cursorrules 文件用于配置 AI 编码助手

/**
 * 项目类型：全栈 TypeScript 应用
 * 框架：Next.js + NestJS
 * 数据库：PostgreSQL + Supabase
 * AI 集成：OpenAI + Claude
 */

# 全局规则

1. **类型安全优先**
   - 所有函数必须有明确的返回类型
   - 使用 TypeScript strict 模式
   - 禁止使用 `any` 类型，必要时使用 `unknown`

2. **错误处理**
   - 所有 async 函数必须使用 try-catch
   - 使用 Result 类型模式处理可恢复错误
   - 全局异常处理器统一记录错误

3. **代码组织**
   - 遵循单责任原则，每个文件不超过 300 行
   - 使用 barrel 文件导出公共接口
   - 业务逻辑与 UI 分离

# 前端规则

- 使用 React 18 + TypeScript
- 组件使用 functional component + hooks
- 状态管理：React Query + Zustand
- 样式：Tailwind CSS

# 后端规则

- NestJS 模块化架构
- DTO 使用 class-validator 验证
- Repository 模式封装数据访问
- 服务层处理业务逻辑

# AI 集成规则

- 所有 AI 调用必须包含错误处理和重试逻辑
- 使用统一的 AI 客户端封装
- Prompt 模板存储在独立文件
- 结构化输出使用 Zod schema 验证
```

### GitHub Copilot 提示工程

```typescript
// Copilot 提示注释示例

// 创建一个用户注册函数，包含：
// - 邮箱验证（使用正则）
// - 密码强度检查（至少8位，包含大小写和数字）
// - 用户名唯一性检查
// - 发送欢迎邮件
// - 返回创建的用户信息（不含密码）
async function createUser(
  email: string,
  username: string,
  password: string
): Promise<UserResponse> {
  // ... 实现代码
}

// 创建一个向量搜索服务，支持：
// - 多模型支持（OpenAI、Anthropic、本地模型）
// - 自动重试机制（最多3次）
// - 缓存结果（5分钟 TTL）
// - 降级策略（向量失败时使用 BM25）
class VectorSearchService {
  // ... 实现代码
}
```

### ESLint AI 代码审查配置

```javascript
// eslint.config.mjs
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';
import react from 'eslint-plugin-react';
import hooks from 'eslint-plugin-react-hooks';
import jsxA11y from 'eslint-plugin-jsx-a11y';
import perfectionist from 'eslint-plugin-perfectionist';
import unusedImports from 'eslint-plugin-unused-imports';

export default [
  {
    ignores: ['dist/**', 'node_modules/**', '.next/**'],
  },
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  {
    plugins: {
      react,
      'react-hooks': hooks,
      'jsx-a11y': jsxA11y,
      perfectionist,
      'unused-imports': unusedImports,
    },
    rules: {
      // TypeScript 规则
      '@typescript-eslint/no-unused-vars': ['error', {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_',
      }],
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/consistent-type-imports': ['error', {
        prefer: 'type-imports',
      }],
      
      // React 规则
      'react/react-in-jsx-scope': 'off',
      'react/prop-types': 'off',
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      
      // 可访问性规则
      'jsx-a11y/alt-text': 'error',
      'jsx-a11y/anchor-has-content': 'error',
      'jsx-a11y/click-events-have-key-events': 'error',
      
      // 代码风格规则
      'perfectionist/sort-named-imports': 'error',
      'perfectionist/sort-object-types': 'error',
      'unused-imports/no-unused-imports': 'error',
    },
  },
];
```

### Prettier 代码格式化配置

```javascript
// .prettierrc
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "arrowParens": "always",
  "endOfLine": "lf",
  "bracketSpacing": true,
  "bracketSameLine": false,
  "experimentalTernaries": false,
  "singleAttributePerLine": false,
  "overrides": [
    {
      "files": "*.md",
      "options": {
        "proseWrap": "preserve",
        "printWidth": 80
      }
    },
    {
      "files": "*.{json,yaml,yml}",
      "options": {
        "printWidth": 120
      }
    }
  ]
}
```

### Husky + lint-staged 配置

```json
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "commit-msg": "commitlint --edit"
    }
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "tsc --noEmit"
    ],
    "*.{js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### 完整的 Monorepo 配置示例

```json
// workspace/frontend/package.json
{
  "name": "@app/frontend",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@app/shared": "workspace:*",
    "next": "14.1.0",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    "@tanstack/react-query": "5.17.0",
    "zustand": "4.4.7",
    "axios": "1.6.5",
    "zod": "3.22.4"
  },
  "devDependencies": {
    "@types/node": "20.11.0",
    "@types/react": "18.2.47",
    "@types/react-dom": "18.2.18",
    "typescript": "5.3.3"
  }
}
```

```json
// workspace/backend/package.json
{
  "name": "@app/backend",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "tsx watch src/main.ts",
    "build": "tsc",
    "start": "node dist/main.js",
    "lint": "eslint src --ext .ts",
    "typecheck": "tsc --noEmit",
    "test": "vitest",
    "test:e2e": "vitest run --config vitest.config.e2e.ts"
  },
  "dependencies": {
    "@app/shared": "workspace:*",
    "@nestjs/core": "10.3.0",
    "@nestjs/common": "10.3.0",
    "@nestjs/platform-express": "10.3.0",
    "reflect-metadata": "0.2.1",
    "rxjs": "7.8.1",
    "zod": "3.22.4",
    "prisma": "6.0.0"
  },
  "devDependencies": {
    "@types/node": "20.11.0",
    "typescript": "5.3.3",
    "tsx": "4.7.0",
    "vitest": "1.1.0",
    "@vitest/coverage-v8": "1.1.0"
  }
}
```

### AI 应用的完整架构示例

```typescript
// src/ai/application/agent.ts
import { z } from 'zod';

export interface AgentConfig {
  name: string;
  description: string;
  model: 'gpt-4' | 'gpt-3.5-turbo' | 'claude-3-opus' | 'claude-3-sonnet';
  temperature: number;
  maxTokens: number;
  tools: Tool[];
  systemPrompt: string;
}

export interface Tool {
  name: string;
  description: string;
  schema: z.ZodType;
  handler: (params: unknown) => Promise<unknown>;
}

export interface Message {
  role: 'system' | 'user' | 'assistant' | 'tool';
  content: string;
  toolCalls?: ToolCall[];
  toolResults?: ToolResult[];
}

export interface ToolCall {
  id: string;
  name: string;
  arguments: Record<string, unknown>;
}

export interface ToolResult {
  toolCallId: string;
  result: unknown;
  error?: string;
}

export interface AgentResponse {
  content: string;
  toolCalls?: ToolCall[];
  finishReason: 'stop' | 'tool_calls' | 'length';
  usage: {
    promptTokens: number;
    completionTokens: number;
    totalTokens: number;
  };
}

export class AIAgent {
  private config: AgentConfig;
  private messageHistory: Message[] = [];
  private maxHistoryLength: number;

  constructor(config: AgentConfig, maxHistoryLength = 50) {
    this.config = config;
    this.maxHistoryLength = maxHistoryLength;
  }

  async process(userMessage: string): Promise<AgentResponse> {
    // 添加用户消息
    this.addMessage('user', userMessage);

    // 调用 AI
    const response = await this.callAI();

    // 添加助手响应
    this.addMessage('assistant', response.content, response.toolCalls);

    // 如果有工具调用，执行它们
    if (response.toolCalls?.length) {
      const results = await this.executeTools(response.toolCalls);
      
      // 添加工具结果
      for (const result of results) {
        this.addMessage('tool', JSON.stringify(result), undefined, result);
      }

      // 继续调用 AI 处理工具结果
      return this.processWithHistory();
    }

    return response;
  }

  private async callAI(): Promise<AgentResponse> {
    // 根据模型选择不同的 API 调用
    switch (this.config.model) {
      case 'gpt-4':
      case 'gpt-3.5-turbo':
        return this.callOpenAI();
      case 'claude-3-opus':
      case 'claude-3-sonnet':
        return this.callClaude();
      default:
        throw new Error(`Unsupported model: ${this.config.model}`);
    }
  }

  private async callOpenAI(): Promise<AgentResponse> {
    // OpenAI API 调用实现
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      },
      body: JSON.stringify({
        model: this.config.model,
        messages: this.buildMessages(),
        temperature: this.config.temperature,
        max_tokens: this.config.maxTokens,
        tools: this.buildTools(),
        tool_choice: 'auto',
      }),
    });

    const data = await response.json();
    
    return {
      content: data.choices[0]?.message?.content || '',
      toolCalls: data.choices[0]?.message?.tool_calls?.map((tc: any) => ({
        id: tc.id,
        name: tc.function.name,
        arguments: JSON.parse(tc.function.arguments),
      })),
      finishReason: data.choices[0]?.finish_reason,
      usage: data.usage,
    };
  }

  private async callClaude(): Promise<AgentResponse> {
    // Claude API 调用实现
    const response = await fetch('https://api.anthropic.com/v1/messages', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': process.env.ANTHROPIC_API_KEY || '',
        'anthropic-version': '2023-06-01',
      },
      body: JSON.stringify({
        model: this.config.model,
        messages: this.buildClaudeMessages(),
        max_tokens: this.config.maxTokens,
        tools: this.buildClaudeTools(),
      }),
    });

    const data = await response.json();
    
    return {
      content: data.content?.find((c: any) => c.type === 'text')?.text || '',
      toolCalls: data.content?.find((c: any) => c.type === 'tool_use') ? [{
        id: data.content.find((c: any) => c.type === 'tool_use')?.id,
        name: data.content.find((c: any) => c.type === 'tool_use')?.name,
        arguments: JSON.parse(data.content.find((c: any) => c.type === 'tool_use')?.input),
      }] : undefined,
      finishReason: data.stop_reason,
      usage: {
        promptTokens: data.usage?.input_tokens || 0,
        completionTokens: data.usage?.output_tokens || 0,
        totalTokens: (data.usage?.input_tokens || 0) + (data.usage?.output_tokens || 0),
      },
    };
  }

  private buildMessages(): any[] {
    const messages = [
      { role: 'system', content: this.config.systemPrompt },
    ];

    for (const msg of this.messageHistory.slice(-this.maxHistoryLength)) {
      if (msg.role === 'tool') {
        messages.push({
          role: 'tool',
          tool_call_id: msg.toolResults?.[0]?.toolCallId,
          content: msg.content,
        });
      } else if (msg.toolCalls?.length) {
        messages.push({
          role: 'assistant',
          content: msg.content,
          tool_calls: msg.toolCalls.map(tc => ({
            id: tc.id,
            type: 'function',
            function: {
              name: tc.name,
              arguments: JSON.stringify(tc.arguments),
            },
          })),
        });
      } else {
        messages.push({ role: msg.role, content: msg.content });
      }
    }

    return messages;
  }

  private buildTools() {
    return this.config.tools.map(tool => ({
      type: 'function',
      function: {
        name: tool.name,
        description: tool.description,
        parameters: this.zodToJsonSchema(tool.schema),
      },
    }));
  }

  private async executeTools(toolCalls: ToolCall[]): Promise<ToolResult[]> {
    const results: ToolResult[] = [];

    for (const toolCall of toolCalls) {
      const tool = this.config.tools.find(t => t.name === toolCall.name);
      
      if (!tool) {
        results.push({
          toolCallId: toolCall.id,
          result: null,
          error: `Tool ${toolCall.name} not found`,
        });
        continue;
      }

      try {
        // 验证参数
        const validatedParams = tool.schema.parse(toolCall.arguments);
        // 执行工具
        const result = await tool.handler(validatedParams);
        results.push({
          toolCallId: toolCall.id,
          result,
        });
      } catch (error) {
        results.push({
          toolCallId: toolCall.id,
          result: null,
          error: error instanceof Error ? error.message : 'Unknown error',
        });
      }
    }

    return results;
  }

  private addMessage(
    role: Message['role'],
    content: string,
    toolCalls?: ToolCall[],
    toolResults?: ToolResult[]
  ) {
    this.messageHistory.push({ role, content, toolCalls, toolResults });
    
    // 限制历史长度
    if (this.messageHistory.length > this.maxHistoryLength) {
      this.messageHistory = this.messageHistory.slice(-this.maxHistoryLength);
    }
  }

  private zodToJsonSchema(schema: z.ZodType): object {
    // 简化的 Zod 到 JSON Schema 转换
    const shape = (schema as any)._def?.shape?.() || {};
    
    const properties: Record<string, any> = {};
    const required: string[] = [];

    for (const [key, value] of Object.entries(shape)) {
      const zodType = value as z.ZodType;
      const typeName = this.getZodTypeName(zodType);
      
      properties[key] = {
        type: typeName,
        description: (zodType as any)._def?.description || '',
      };
      
      if (!(zodType as any)._def?.isOptional) {
        required.push(key);
      }
    }

    return {
      type: 'object',
      properties,
      required,
    };
  }

  private getZodTypeName(zodType: z.ZodType): string {
    const typeName = (zodType as any)._def?.typeName;
    
    switch (typeName) {
      case 'ZodString': return 'string';
      case 'ZodNumber': return 'number';
      case 'ZodBoolean': return 'boolean';
      case 'ZodArray': return 'array';
      case 'ZodObject': return 'object';
      default: return 'string';
    }
  }

  getHistory(): Message[] {
    return [...this.messageHistory];
  }

  clearHistory() {
    this.messageHistory = [];
  }
}
```

### 使用示例

```typescript
// 创建 Agent
const agent = new AIAgent({
  name: 'Research Assistant',
  description: '一个研究助手，可以搜索网页和分析数据',
  model: 'gpt-4',
  temperature: 0.7,
  maxTokens: 2000,
  tools: [
    {
      name: 'search_web',
      description: '搜索网页获取信息',
      schema: z.object({
        query: z.string().describe('搜索关键词'),
        num_results: z.number().optional().default(5),
      }),
      handler: async (params: { query: string; num_results: number }) => {
        // 实现搜索逻辑
        const results = await searchAPI(params.query, params.num_results);
        return results;
      },
    },
    {
      name: 'calculate',
      description: '执行数学计算',
      schema: z.object({
        expression: z.string().describe('数学表达式'),
      }),
      handler: async (params: { expression: string }) => {
        // 安全计算
        const result = evaluateExpression(params.expression);
        return { result };
      },
    },
  ],
  systemPrompt: `你是一个专业的研究助手。你可以：
1. 使用搜索工具获取最新信息
2. 进行数学计算
3. 分析和组织信息
4. 以清晰的结构呈现结果`,
});

// 处理用户消息
const response = await agent.process('帮我搜索一下 2024 年 AI 发展的最新进展');
console.log(response.content);

// 继续对话
const followUp = await agent.process('能总结一下重点吗？');
console.log(followUp.content);
```

---

> [!SUCCESS]
> 本文档全面介绍了 TypeScript 与 AI 工具的深度集成，包括 Claude API、Cursor IDE、GitHub Copilot、代码审查工具以及完整的 AI Agent 架构实现。TypeScript 凭借其强大的类型系统和 AI 友好的特性，是构建 AI 应用的理想选择。

