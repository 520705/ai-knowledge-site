---
title: Bun 完全指南
date: 2026-04-24
tags:
  - 后端框架
  - Bun
  - JavaScript Runtime
  - Node.js
  - 性能
description: Bun 是一个全能 JavaScript 运行时，同时集成了打包器、转译器、包管理器和测试框架，本指南涵盖 Bun vs Node.js vs Deno 的深度对比、Bun 原生 Web 框架 Elysia 及实战应用。
---

# Bun 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Bun 作为运行时、打包器、包管理器的完整能力，Elysia 框架集成，性能基准测试及从 Node.js 迁移的实践指南。

---

## Bun 概述：JavaScript 生态的全能选手

Bun 是由 Jarred Sumner 于 2021 年创建的 JavaScript 运行时和工具链，它的设计目标非常宏大——**用一个工具解决 JavaScript 开发中的所有痛点**。在 Bun 出现之前，如果你要构建一个现代 JavaScript 项目，你可能需要：Node.js 作为运行时、npm/yarn/pnpm 作为包管理器、esbuild 或 webpack 作为打包器、TypeScript 作为转译器、Jest/Vitest 作为测试框架。Bun 的出现将这一切都整合到了一个统一工具中。

截至 2026 年，Bun 已经发布了 2.x 版本，稳定支持了 Node.js 的大部分 API、npm 生态的大部分包，以及 TypeScript 和 JSX 的原生支持。Bun 的核心理念可以概括为三个关键词：**极速**、**全能**和**兼容**。

### Bun 的组成

Bun 实际上包含四个独立的组件，理解它们有助于你更好地使用 Bun：

**第一个组件，Bun 运行时。** 这是 Bun 最核心的部分——一个用 Zig 编写的 JavaScript 运行时，基于 JavaScriptCore 引擎（与 Safari 相同），而非 V8 引擎（Chrome/Node.js 使用）。这个选择让 Bun 在某些场景下获得了比 Node.js 更快的启动速度和更低的内存占用。

**第二个组件，Bun 打包器。** Bun 内置了一个名为「Bundler」的打包工具，可以替代 esbuild、webpack、rollup 等。它的打饱速度比大多数现有工具快 10 倍以上，同时支持 Tree Shaking、Code Splitting、Define 替换等高级特性。

**第三个组件，Bun 安装器。** Bun 的包管理器 `bun install` 比 npm 快 25 倍以上，比 pnpm 快 5 倍以上。它使用全局缓存和硬链接来避免重复下载，而且可以读取 npm 的 `package-lock.json`。

**第四个组件，Bun 测试框架。** Bun 内置了测试框架，可以替代 Jest 和 Vitest。它支持快照测试、Mock、覆盖率统计，而且执行速度极快。

> [!TIP]
> **引擎选择的小背景**：JavaScriptCore（JSC）是 WebKit 引擎的核心，Safari 浏览器使用它；V8 是 Chrome/Node.js/Deno 使用的引擎。两种引擎各有优势：V8 在 JIT 编译优化方面更强，适合长时间运行的服务器应用；JSC 在启动速度和解析速度方面更强，适合短时脚本和 Serverless 场景。Bun 选择 JSC 正是看中了它在启动速度上的优势。

---

## Bun vs Node.js vs Deno：三角鼎立

### 核心定位对比

JavaScript 运行时生态在 2026 年已经形成了三足鼎立的格局：Bun、Node.js 和 Deno。三者各有特色，适合不同的场景。

Node.js 是这个领域的「老大哥」，诞生于 2009 年，拥有最成熟的生态和最广泛的社区支持。它使用 V8 引擎，是目前企业级 JavaScript 后端开发的事实标准。Node.js 的 API 最为完善，几乎所有 npm 包都针对 Node.js 做了优化。

Deno 由 Node.js 的创始人 Ryan Dahl 于 2020 年创建，旨在解决 Node.js 的一些设计缺陷。Deno 默认安全（需要显式授予文件和网络访问权限）、原生支持 TypeScript、不依赖 npm，直接从 URL 加载模块。Deno 的设计更现代化，但在 npm 生态兼容性方面还有差距。

Bun 则采取了不同的策略——它既追求 Deno 的现代化设计，又追求对 Node.js 生态的完全兼容。Bun 原生支持 TypeScript 和 JSX、内置了打包器、支持 npm 包、有自己的包管理器和测试框架。

### 详细功能对比

| 特性 | Bun | Node.js | Deno |
|------|------|---------|------|
| **JavaScript 引擎** | JavaScriptCore | V8 | V8 |
| **TypeScript 支持** | 原生（无需转译） | 需要 ts-node 或编译 | 原生支持 |
| **JSX 支持** | 原生 | 需要 Babel | 原生支持 |
| **npm 兼容性** | 高 | 完整 | 中等（需适配层） |
| **包管理器** | 内置（bun install） | npm/yarn/pnpm | 内置（deno install） |
| **NPM 生态** | 高度兼容 | 完整支持 | 需要额外配置 |
| **内置打包器** | 是 | 否 | 否 |
| **内置测试框架** | 是 | 否（Jest 等） | 是 |
| **Workers 兼容** | Bun.serve | — | — |
| **SQLite 驱动** | 内置 | 需库 | 需库 |
| **HTTP/2** | 支持 | 支持 | 支持 |
| **ESM/CommonJS** | 两者都支持 | 两者都支持 | 主要 ESM |
| **Web API** | 部分（Fetch、Request 等） | 全部 | 全部 |
| **本地模块** | .node 和 .dylib | .node | — |

### 性能对比

性能是 Bun 最引人注目的特点之一。来看一下几项关键基准测试的数据：

> [!NOTE]
> 以下数据基于公开的 benchmark 测试和实验室环境，实际性能会因硬件、工作负载和代码实现而有所不同。数字仅作为参考，不要盲目追求 benchmark 排名。

**启动时间对比**（测试文件：简单的「Hello World」HTTP 服务器）：

| 运行时 | 首次启动 | 热启动 |
|--------|---------|--------|
| Bun | ~10ms | ~5ms |
| Node.js 22 | ~80ms | ~30ms |
| Deno 2.x | ~60ms | ~25ms |

**简单 Hello World 请求吞吐量**（`autocannon -c 100 -d 10`）：

| 运行时 | 平均 QPS | p99 延迟 |
|--------|---------|---------|
| Bun + Elysia | ~55,000 | ~15ms |
| Bun + Hono | ~48,000 | ~20ms |
| Bun + Fastify | ~35,000 | ~25ms |
| Node.js + Fastify | ~32,000 | ~30ms |
| Node.js + Express | ~18,000 | ~65ms |
| Deno + Hono | ~45,000 | ~22ms |

**包安装速度**（安装一个包含 100 个依赖的中型项目）：

| 工具 | 耗时 |
|------|------|
| bun install | ~1.2s |
| pnpm | ~3.5s |
| npm | ~8.0s |
| yarn | ~6.0s |

> [!TIP]
> **性能差异的实际意义**：对于一个每秒处理数百到数千请求的 Web 服务器来说，Bun 的性能优势是真实的。但对于大多数中小型应用来说，启动时间节省的几十毫秒和吞吐量提升的几千 QPS 可能并不关键——更重要的是选择一个你熟悉的、生态完善的框架。

---

## Bun 的核心命令

Bun 提供了一套统一的命令行工具，覆盖了开发过程中的各个环节。

### 安装与更新

```bash
# 安装 Bun
curl -fsSL https://bun.sh/install | bash

# macOS/Linux 也可用 brew 安装
brew install bun

# 更新 Bun
bun update

# 检查版本
bun --version
```

### bun install：极速包管理

`bun install` 是 Bun 的包管理器，它会读取 `package.json` 和 `bun.lockb`（Bun 的锁文件）来管理依赖。

```bash
# 安装所有依赖
bun install

# 安装特定包
bun add <package-name>

# 安装为开发依赖
bun add -d <package-name>

# 移除包
bun remove <package-name>

# 更新包
bun update <package-name>

# 更新所有包
bun update

# 清理缓存
bun pm cache rm
```

> [!TIP]
> **bun.lockb vs package-lock.json**：Bun 使用 `bun.lockb` 作为锁文件。如果你在一个已有 `package-lock.json` 的项目中使用 Bun，Bun 可以读取它并生成 `bun.lockb`。如果你和其他团队成员使用不同的运行时，建议在 `.gitignore` 中忽略锁文件，或者统一使用同一种锁文件格式。

### bun run：执行脚本

```bash
# 运行一个 JS/TS 文件
bun run index.ts
bun run index.js

# 运行 package.json 中定义的脚本
bun run dev
bun run build
bun run start

# 运行命令（跳过 npm script）
bun -e "console.log('inline script')"

# 运行 benchmark
bun run benchmark.ts
```

### bun test：内置测试框架

Bun 的测试框架是最令人惊喜的特性之一——它比 Jest 快 10 倍以上，而且 API 设计得非常直觉：

```bash
# 运行所有测试
bun test

# 运行特定文件
bun test src/users.test.ts

# 运行匹配特定模式的测试
bun test "user"

# 监听模式（文件变化时重新运行）
bun test --watch

# 生成覆盖率报告
bun test --coverage
```

```typescript
// users.test.ts
import { describe, test, expect, beforeEach, mock } from 'bun:test';
import { UsersService } from './users.service';

describe('UsersService', () => {
  let service: UsersService;
  let mockDb: any;

  beforeEach(() => {
    mockDb = {
      findMany: mock(() => [
        { id: '1', name: 'Alice', email: 'alice@example.com' },
        { id: '2', name: 'Bob', email: 'bob@example.com' },
      ]),
      findUnique: mock((id: string) => 
        id === '1' ? { id: '1', name: 'Alice', email: 'alice@example.com' } : null
      ),
      create: mock((data: any) => ({ id: '3', ...data })),
    };
    service = new UsersService(mockDb);
  });

  test('findAll should return all users', async () => {
    const users = await service.findAll();
    expect(users).toHaveLength(2);
    expect(users[0].name).toBe('Alice');
    expect(mockDb.findMany).toHaveBeenCalledTimes(1);
  });

  test('findOne should return user by id', async () => {
    const user = await service.findOne('1');
    expect(user?.name).toBe('Alice');
  });

  test('findOne should return null for non-existent id', async () => {
    const user = await service.findOne('999');
    expect(user).toBeNull();
  });

  test('create should create and return new user', async () => {
    const newUser = await service.create({
      name: 'Charlie',
      email: 'charlie@example.com',
    });
    expect(newUser.id).toBe('3');
    expect(newUser.name).toBe('Charlie');
    expect(mockDb.create).toHaveBeenCalledTimes(1);
  });
});
```

### bun build：打包

Bun 的打包器是一个隐藏的宝藏——它比 esbuild 更快，支持更多的特性：

```bash
# 打包为浏览器可用的 bundle
bun build ./src/index.ts --outdir ./dist --target browser

# 打包为 Node.js 可用的模块
bun build ./src/index.ts --outdir ./dist --target node

# 打包为单个文件（Bundle）
bun build ./src/index.ts --outfile ./dist/bundle.js --bundle

# 打包为 ESM 格式
bun build ./src/index.ts --outfile ./dist/bundle.mjs --format esm

# 启用 Tree Shaking
bun build ./src/index.ts --outfile ./dist/bundle.js --minify --tree-shaking

# 启用代码分割
bun build ./src/index.ts --outdir ./dist --splitting --format esm
```

### bun x：直接运行包

```bash
# 直接运行一个 npm 包（无需全局安装）
bunx prisma generate
bunx typescript --init
bunx esbuild script.ts --outfile=out.js
```

> [!TIP]
> `bunx` 等价于 `npx`，但速度更快。它会先检查本地依赖，然后检查全局安装，最后从 npm 下载。

---

## Bun + Elysia：Bun 原生的极速框架

如果说 Bun 是 JavaScript 运行时中的瑞士军刀，那么 **Elysia** 就是 Bun 上的 Fastify——一个为 Bun 量身打造的高性能 Web 框架。Elysia 由 Salomon Lee 于 2023 年创建，充分利用了 Bun 的特性，实现了目前最快的 Web 框架性能。

### 为什么选择 Elysia

Elysia 的设计哲学是「**让高性能变得简单**」。它利用 Bun 的以下特性来实现极致性能：

**第一，Native Promise 和 Async/Await。** Bun 对 Promise 的优化非常好，Elysia 充分利用了这一点，避免了不必要的 Promise 包装。

**第二，强类型系统。** Elysia 利用 TypeScript 的泛型和条件类型，实现了完美的类型推断——路由参数、请求体、响应类型都能被自动推断出来。

**第三，Bun.serve 的零拷贝优化。** Bun 的 HTTP 服务器实现直接操作缓冲区，Elysia 在这个基础上构建，最大限度地减少了内存拷贝。

### Elysia 基础

```typescript
// src/index.ts
import { Elysia } from 'elysia';

// 创建应用
const app = new Elysia()
  // 全局前缀
  .state('version', '1.0.0')
  // 全局中间件
  .onBeforeHandle(({ request, set }) => {
    console.log(`${request.method} ${new URL(request.url).pathname}`);
  })
  // 路由定义
  .get('/', () => 'Hello Elysia!')
  .get('/json', () => ({ message: 'Hello JSON' }))
  .get('/html', () => '<h1>Hello HTML</h1>')
  // 路径参数
  .get('/users/:id', ({ params: { id } }) => {
    return { userId: id };
  })
  // 查询参数
  .get('/search', ({ query: { q, page = '1' } }) => {
    return { query: q, page: parseInt(page) };
  })
  // POST 请求
  .post('/users', ({ body }) => {
    return { created: body };
  })
  // 启动服务器
  .listen(3000);

console.log(`Server running at http://localhost:${app.server?.port}`);
```

### Elysia 的中间件和生命周期

```typescript
import { Elysia } from 'elysia';
import { cors } from '@elysiajs/cors';
import { jwt } from '@elysiajs/jwt';
import { cookie } from '@elysiajs/cookie';

const app = new Elysia()
  // CORS 插件
  .use(cors({
    origin: ['https://example.com'],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  }))
  // JWT 插件
  .use(jwt({
    secret: process.env.JWT_SECRET!,
    exp: '7d',
  }))
  // Cookie 插件
  .use(cookie())
  // 全局前置处理
  .onBeforeHandle(({ request }) => {
    console.log('Before handle:', request.url);
  })
  // 全局后置处理
  .onAfterHandle(({ response }) => {
    console.log('Response sent');
    return response;
  })
  // 全局错误处理
  .onError(({ error, code }) => {
    console.error('Error:', error.message);
    
    if (code === 'NOT_FOUND') {
      return { error: 'Route not found' };
    }
    
    if (code === 'VALIDATION') {
      return { error: 'Invalid input', details: error };
    }
    
    return { error: 'Internal server error' };
  })
  // 受保护的路由（使用 JWT）
  .group('/api', (app) =>
    app
      .derive(({ jwt, cookie: { auth } }) => {
        // 每个请求前验证 JWT
        return jwt.verify(auth.value).then((user) => ({ user }));
      })
      .get('/profile', ({ user }) => {
        if (!user) return { error: 'Unauthorized' };
        return { user };
      })
      .post('/logout', ({ cookie: { auth }, set }) => {
        auth.remove();
        return { success: true };
      })
  )
  .listen(3000);
```

### Elysia + Drizzle ORM 完整示例

```typescript
// src/index.ts
import { Elysia } from 'elysia';
import { cors } from '@elysiajs/cors';
import { swagger } from '@elysiajs/swagger';
import { drizzle } from 'drizzle-orm/bun-sqlite';
import { users, posts } from './schema';
import { eq, like, desc } from 'drizzle-orm';
import { BunSQLiteDriver } from 'drizzle-orm/bun-sqlite/driver';

const db = drizzle(new BunSQLiteDriver(':memory:'));

const app = new Elysia()
  // 插件
  .use(cors())
  .use(swagger({ documentation: {
    info: { title: 'My API', version: '1.0.0' },
  }}))
  // 错误处理
  .onError(({ error }) => {
    return {
      success: false,
      error: error.message,
    };
  })
  // 路由
  .get('/users', async ({ query: { page = '1', limit = '10', search } }) => {
    const offset = (parseInt(page) - 1) * parseInt(limit);
    
    let whereClause;
    if (search) {
      whereClause = like(users.name, `%${search}%`);
    }
    
    const result = await db
      .select()
      .from(users)
      .where(whereClause)
      ..limit(parseInt(limit))
      .offset(offset)
      ..orderBy(desc(users.createdAt));
    
    return {
      success: true,
      data: result,
      meta: {
        page: parseInt(page),
        limit: parseInt(limit),
      },
    };
  })
  .get('/users/:id', async ({ params: { id } }) => {
    const user = await db
      .select()
      .from(users)
      .where(eq(users.id, id))
      .get();
    
    if (!user) {
      return Response.json({ success: false, error: 'User not found' }, { status: 404 });
    }
    
    return { success: true, data: user };
  })
  .post('/users', async ({ body }) => {
    const [user] = await db.insert(users).values(body).returning();
    return Response.json({ success: true, data: user }, { status: 201 });
  })
  .put('/users/:id', async ({ params: { id }, body }) => {
    const [user] = await db
      .update(users)
      .set(body)
      .where(eq(users.id, id))
      .returning();
    
    if (!user) {
      return Response.json({ success: false, error: 'User not found' }, { status: 404 });
    }
    
    return { success: true, data: user };
  })
  .delete('/users/:id', async ({ params: { id } }) => {
    await db.delete(users).where(eq(users.id, id));
    return new Response(null, { status: 204 });
  })
  .listen(3000);

console.log(`Server running at http://localhost:${app.server?.port}`);
```

---

## Bun + Hono

Hono 是 Bun 上另一个非常出色的选择，特别是当你需要跨运行时部署时。

```typescript
// src/index.ts
import { Hono } from 'hono';
import { cors } from 'hono/cors';
import { logger } from 'hono/logger';
import { serve } from 'bun';

const app = new Hono();

// 中间件
app.use('*', cors());
app.use('*', logger());

// 健康检查
app.get('/health', (c) => c.json({ status: 'ok' }));

// 用户列表
app.get('/api/users', (c) => {
  return c.json({
    data: [
      { id: '1', name: 'Alice', email: 'alice@example.com' },
      { id: '2', name: 'Bob', email: 'bob@example.com' },
    ],
  });
});

// 获取单个用户
app.get('/api/users/:id', (c) => {
  const { id } = c.req.param();
  
  if (id === '404') {
    return c.json({ error: 'User not found' }, 404);
  }
  
  return c.json({
    data: { id, name: 'User ' + id, email: `user${id}@example.com` },
  });
});

// 创建用户
app.post('/api/users', async (c) => {
  const body = await c.req.json();
  
  if (!body.email || !body.name) {
    return c.json({ error: 'Name and email are required' }, 400);
  }
  
  return c.json({
    data: { id: crypto.randomUUID(), ...body },
  }, 201);
});

// 启动服务器
serve({
  fetch: app.fetch,
  port: 3000,
});

console.log('Server running at http://localhost:3000');
```

---

## Bun REPL 和脚本

Bun 的 REPL（交互式解释器）也是一个强大的工具，可以用于快速实验和调试。

```bash
# 进入 REPL
bun repl

# REPL 中可以直接执行 JavaScript/TypeScript
> const sum = (a: number, b: number): number => a + b
undefined
> sum(1, 2)
3
> import { readdir } from 'node:fs/promises'
[Function: readdir]
> await readdir('.')
[ 'src', 'node_modules', 'package.json', ... ]
```

Bun 还可以用于快速脚本，无需单独的运行时配置：

```typescript
// scripts/migrate.ts
// 直接运行：bun run scripts/migrate.ts
import { drizzle } from 'drizzle-orm/bun-sqlite';
import { migrate } from 'drizzle-orm/bun-sqlite/migrator';

const db = drizzle(':memory:');

// 运行迁移
await migrate(db, { migrationsFolder: './drizzle' });

console.log('Migration completed!');
```

---

## Bun 的原生 TypeScript 支持

Bun 原生支持 TypeScript，无需任何配置或额外安装：

```typescript
// src/example.ts
// 这段 TypeScript 代码可以直接运行，无需编译
interface User {
  id: string;
  name: string;
  email: string;
  age?: number;
}

function greetUser(user: User): string {
  return `Hello, ${user.name}! You are ${user.age ?? 'unknown'} years old.`;
}

const users: User[] = [
  { id: '1', name: 'Alice', email: 'alice@example.com', age: 30 },
  { id: '2', name: 'Bob', email: 'bob@example.com' },
  { id: '3', name: 'Charlie', email: 'charlie@example.com', age: 25 },
];

users.forEach((user) => {
  console.log(greetUser(user));
});
```

运行方式：

```bash
bun run src/example.ts
```

> [!TIP]
> **Bun 的 TypeScript 支持**：Bun 内置了 TypeScript 编译器，使用 esbuild 进行转译。这使得 Bun 的 TypeScript 支持比 Node.js + ts-node 快很多——因为 esbuild 比 tsc 快 10 倍以上。需要注意的是，Bun 的 TypeScript 编译器只进行语法转译，不做类型检查（TypeScript 的类型检查是由 `tsc` 做的）。如果你需要在编写代码时获得类型错误提示，仍然需要在 VS Code 中使用 TypeScript 语言服务。

---

## Bun 的内置 SQLite 驱动

Bun 内置了 SQLite 支持，这是一个惊喜——你可以直接使用 `bun:sqlite` 模块，而无需安装任何第三方库：

```typescript
import { Database } from 'bun:sqlite';

// 创建内存数据库
const db = new Database(':memory:');

// 创建表
db.run(`
  CREATE TABLE users (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    age INTEGER,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
  )
`);

// 插入数据
const insertUser = db.prepare(`
  INSERT INTO users (id, name, email, age)
  VALUES (?, ?, ?, ?)
`);

// 插入多条记录
const users = [
  ['1', 'Alice', 'alice@example.com', 30],
  ['2', 'Bob', 'bob@example.com', 25],
  ['3', 'Charlie', 'charlie@example.com', 35],
];

for (const user of users) {
  insertUser.run(...user);
}

// 查询数据
const allUsers = db.query('SELECT * FROM users').all();
console.log('All users:', allUsers);

// 带参数的查询（防止 SQL 注入）
const findById = db.prepare('SELECT * FROM users WHERE id = ?');
const user = findById.get('1');
console.log('Found user:', user);

// 带参数的查询（多条结果）
const findByAge = db.prepare('SELECT * FROM users WHERE age > ?');
const adults = findByAge.all(25);
console.log('Adults:', adults);

// 事务操作
db.exec('BEGIN TRANSACTION');
try {
  const deleteById = db.prepare('DELETE FROM users WHERE id = ?');
  deleteById.run('3');
  db.exec('COMMIT');
} catch (error) {
  db.exec('ROLLBACK');
  console.error('Transaction failed:', error);
}

// 遍历查询结果
const query = db.query('SELECT * FROM users');
for (const row of query.iterator()) {
  console.log(row.id, row.name, row.email);
}
```

---

## 从 Node.js 迁移到 Bun

### 迁移策略

将现有 Node.js 项目迁移到 Bun 通常分为以下几个步骤：

**第一步，评估兼容性。** 检查你的项目是否使用了 Node.js 特定的功能。Bun 兼容大部分 Node.js API，但有一些差异需要注意：

```bash
# Bun 声称兼容的 Node.js API：
# - fs, path, os, url, querystring, util
# - stream, events, http, https
# - crypto (部分), net, tls, dgram
# - child_process, readline, repl

# Bun 不兼容的 Node.js API：
# - vm (部分)
# - inspector
# - worker_threads (使用 Bun.workers)
# - some crypto operations
```

**第二步，更新 package.json scripts。** 将 `node` 命令替换为 `bun`：

```json
// before (package.json)
{
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "jest"
  }
}

// after (package.json)
{
  "scripts": {
    "dev": "bun --watch src/server.ts",
    "build": "bun build src/server.ts --outdir=dist --target=node",
    "start": "bun run dist/server.js",
    "test": "bun test"
  }
}
```

**第三步，迁移包管理器。** 用 `bun install` 替换 npm/yarn/pnpm：

```bash
# 直接运行即可，Bun 会读取现有的 package-lock.json
bun install

# 或者重新安装
rm -rf node_modules package-lock.json
bun install
```

**第四步，测试运行。** 逐步测试各个功能，确保一切正常：

```bash
# 运行开发服务器
bun run dev

# 运行测试
bun test

# 构建生产版本
bun run build

# 运行生产服务器
bun run start
```

### 常见迁移问题

**问题一：Node.js 特定 API 不兼容。**

```typescript
// ❌ Node.js 特定写法
import { Worker } from 'worker_threads';
const worker = new Worker('./worker.js');

// ✅ Bun 兼容写法
import { Worker } from 'bun:worker_threads';
const worker = new Worker('./worker.ts');
```

**问题二：环境变量差异。**

```typescript
// ❌ Node.js 环境变量
process.env.DATABASE_URL

// ✅ Bun 兼容写法（两者都支持）
process.env.DATABASE_URL
Bun.env.DATABASE_URL // Bun 的专属方式
```

**问题三：ESM/CommonJS 混淆。**

Bun 同时支持 ESM 和 CommonJS，但如果你的项目在两种格式之间混淆，可能会遇到问题。确保 `package.json` 中的 `type` 字段与代码格式一致。

> [!WARNING]
> **生产环境谨慎迁移**：虽然 Bun 在很多场景下表现出色，但它的稳定性相比 Node.js 仍有差距。对于生产环境，建议先在非关键服务上测试，确认兼容性后再全面迁移。同时关注 Bun 的 release notes，了解 breaking changes。

---

## 当前限制与注意事项

### 已知的局限性

**第一，生态系统成熟度。** Bun 对 npm 包的兼容性很高，但不是 100%。某些原生模块（需要编译 `.node` 文件的包）可能需要额外配置或根本无法在 Bun 上运行。

**第二，Workers API 差异。** Bun 的 `Bun.serve` 和 Web Workers API 与标准 Workers API 有所差异，代码可能无法直接在其他运行时上运行。

**第三，长期运行稳定性。** Bun 的发布时间相比 Node.js 还较短，在超长期运行（如数月不重启的服务器）场景下的稳定性数据还不够充分。

**第四，ARM 架构支持。** 虽然 Bun 已经支持 Apple Silicon 和 ARM Linux，但在某些边缘情况下可能遇到问题。

### 开发中的注意事项

```typescript
// ✅ 在 Bun 中推荐使用 bun install 安装依赖
bun add prisma @prisma/client

// ✅ 使用 Bun 的内置功能
import { Database } from 'bun:sqlite';
import { serve } from 'bun';

// ✅ 使用 Bun 的性能优化
const data = await fetch('https://api.example.com').then(r => r.json());

// ⚠️ 注意：某些 npm 包可能有兼容性问题
// 如果遇到问题，可以尝试：
// 1. 更新到最新版本的包
// 2. 检查包的 README 是否提及 Bun 支持
// 3. 在 GitHub 上搜索 bun 相关的 issues
```

### 与 Docker 的集成

```dockerfile
# Dockerfile
FROM oven/bun:2-alpine AS base

FROM base AS deps
WORKDIR /app
COPY package.json bun.lockb* ./
RUN bun install --frozen-lockfile

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN bun build src/index.ts --target=bun --outfile=dist/index.js

FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup --system --gid 1001 nodejs && adduser --system --uid 1001 app
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules
USER app
EXPOSE 3000
CMD ["bun", "dist/index.js"]
```

---

## 实战项目推荐结构

```
my-bun-api/
├── src/
│   ├── index.ts           # 入口文件
│   ├── app.ts            # Elysia/Hono 应用配置
│   ├── routes/           # 路由
│   │   ├── index.ts
│   │   ├── users.ts
│   │   └── posts.ts
│   ├── services/         # 业务逻辑
│   │   ├── user.service.ts
│   │   └── post.service.ts
│   ├── db/              # 数据库
│   │   ├── index.ts
│   │   └── schema.ts
│   ├── middleware/      # 中间件
│   │   └── auth.ts
│   └── utils/          # 工具函数
│       └── errors.ts
├── tests/               # 测试
│   ├── user.test.ts
│   └── setup.ts
├── drizzle/             # Drizzle 迁移文件
├── .env                # 环境变量
├── drizzle.config.ts   # Drizzle 配置
├── package.json
├── tsconfig.json
└── Dockerfile
```

---

## 选型总结：Bun 在 2026 年的定位

### 适合使用 Bun 的场景

**第一，追求极致性能的 Web API。** 当你需要一个快速的服务器，而 Elysia/Hono + Bun 的组合能提供目前最好的单核性能。

**第二，需要原生 TypeScript 的项目。** 不想配置 TypeScript 编译环境？Bun 直接运行 `.ts` 文件，而且速度快得惊人。

**第三，快速脚本和工具开发。** Bun 的启动速度和多工具集成特性使其非常适合写脚本。

**第四，需要 SQLite 的轻量应用。** Bun 内置的 SQLite 支持对小型应用来说是巨大的便利。

**第五，使用 Elysia 的团队。** Elysia 是目前最快的 Web 框架之一，它专门为 Bun 优化。

### 不适合使用 Bun 的场景

**第一，需要稳定生产环境的企业应用。** Node.js 的稳定性和社区支持仍然是无与伦比的。

**第二，依赖大量原生 Node.js 模块的项目。** 虽然 Bun 的兼容性已经很高，但某些原生模块仍可能有问题。

**第三，需要部署到特定环境。** 目前 Bun 无法运行在 Cloudflare Workers 等边缘环境中（需要使用 Hono）。

**第四，对运行时稳定性要求极高的场景。** 在这些场景下，选择经过多年验证的 Node.js 更稳妥。

---

## 性能基准参考

以下是几项常见的基准测试结果，供你参考：

> [!NOTE]
> 这些数据来自公开的 benchmark 测试和社区报告。实际性能取决于硬件、工作负载和代码实现方式。请以自己的测试结果为准。

| 测试场景 | Bun + Elysia | Bun + Hono | Node.js + Fastify | Node.js + Express |
|---------|-------------|------------|-------------------|-------------------|
| Hello World | ~55,000 req/s | ~48,000 req/s | ~35,000 req/s | ~18,000 req/s |
| JSON API | ~45,000 req/s | ~40,000 req/s | ~28,000 req/s | ~15,000 req/s |
| DB Query | ~25,000 req/s | ~22,000 req/s | ~18,000 req/s | ~10,000 req/s |
| 冷启动 | ~10ms | ~10ms | ~80ms | ~80ms |
| 内存占用 | ~30MB | ~35MB | ~50MB | ~75MB |

> [!SUCCESS]
> Bun 在 2026 年已经成为 JavaScript 生态中不可忽视的一股力量。它的极速性能、原生 TypeScript 支持、内置工具链和 SQLite 驱动，使其成为构建现代 JavaScript 应用的绝佳选择。对于追求极致性能、需要快速开发迭代、或者已经在使用 Elysia/Hono 的团队，Bun 是非常值得认真考虑的技术选型。
