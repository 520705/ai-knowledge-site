# Bun 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Bun 1.2/1.3 新特性、运行时架构、内置工具链及与 Node.js/Deno 的深度对比。

---

## 目录

1. [[#Bun 概述与定位]]
2. [[#Bun 1.x 新特性演进]]
3. [[#运行时架构]]
4. [[#内置工具链]]
5. [[#Web 服务器 Bun.serve]]
6. [[#包管理器 Bun.install]]
7. [[#Bun vs Node.js vs Deno 对比]]
8. [[#性能基准]]
9. [[#实战场景与选型建议]]
10. [[#参考资料]]

---

## Bun 概述与定位

Bun 是一个由 Jarred Sumner 开发的 JavaScript 运行时和工具链，于 2023 年正式发布 1.0 版本。Bun 的设计目标是成为一个「一体化」的 JavaScript 运行时，提供比 Node.js 更高的性能和更现代的工具链。Bun 使用 Zig 编写，JavaScript 引擎基于 WebKit 的 JavaScriptCore（JSC），而非 Node.js 使用的 V8。

### 核心定位

| 维度 | 定位 |
|------|------|
| **目标用户** | 全栈开发者、Node.js 迁移者、性能敏感应用开发者 |
| **核心理念** | 一体化、高性能、兼容性 |
| **核心能力** | 运行时 + 包管理器 + 测试框架 + 打包工具 |
| **性能目标** | 比 Node.js 快 3-5 倍 |
| **学习曲线** | 极低（与 Node.js API 高度兼容） |

### 核心特性矩阵

| 功能 | Bun | Node.js | Deno |
|------|-----|--------|------|
| **运行时** | JavaScriptCore (JSC) | V8 | V8 |
| **语言** | Zig + TypeScript | C++ | Rust + TypeScript |
| **包管理器** | 内置 | npm/pnpm/yarn | 内置（不兼容 npm） |
| **测试框架** | 内置 | Jest/Vitest | 内置 |
| **打包工具** | 内置 | webpack/vite/esbuild | 不支持 |
| **兼容 npm** | 接近 100% | 原生 | 有限 |
| **TypeScript** | 原生支持 | 需转译 | 原生支持 |
| **权限系统** | 无 | 无 | 有（安全沙箱） |

---

## Bun 1.x 新特性演进

### Bun 1.0 核心特性

- **极速启动**：冷启动时间 < 10ms
- **内置 TypeScript**：无需配置，直接运行 .ts 文件
- **兼容 Node.js API**：大部分 Node.js 内置模块直接可用
- **内置打包器**：类 esbuild 的高速打包
- **内置测试框架**：Jest 兼容的测试 API
- **内置 SQLite 客户端**：原生数据库访问

### Bun 1.1/1.2 增强

| 版本 | 日期 | 关键特性 |
|------|------|----------|
| 1.0 | 2023.09 | 正式版发布，运行时 + 基础工具链 |
| 1.1 | 2024.02 | 性能优化、更好的 Node.js 兼容性 |
| 1.2 | 2024.07 | Node.js API 覆盖率 99%+、更快的安装速度 |
| 1.3 | 2025.01 | WebSocket 增强、NPM workspaces 支持 |

### Bun 1.2 核心改进

```typescript
// Bun 1.2 性能提升示例
import { performance } from "bun";

// 比 Node.js 快 3-5 倍的文件 IO
const file = Bun.file("./large-file.json");
const content = await file.text();

// 比 Node.js 快 2-3 倍的 HTTP 请求
const response = await fetch("https://api.example.com/data");
const data = await response.json();

// 更快的包安装（比 npm 快 10 倍）
// bun install 已在 1.0 实现
```

### Bun 1.3 新特性

```typescript
// WebSocket 增强
const server = Bun.serve({
  port: 3000,
  websocket: {
    message(ws, message) {
      // 更高效的 WebSocket 处理
      ws.send(`Echo: ${message}`);
    },
    perMessageDeflate: true,  // 压缩支持
  },
  fetch(req, server) {
    if (server.upgrade(req)) return;
    return new Response("Upgrade required", { status: 426 });
  },
});

// NPM Workspaces 支持
// bun.lockb 支持 monorepo 工作流
```

---

## 运行时架构

### 架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        Bun CLI                              │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐    │
│  │   bun run   │ │  bun install │ │    bun test       │    │
│  └─────────────┘ └─────────────┘ └─────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Bun Runtime                               │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐    │
│  │ JavaScript  │ │   Native    │ │   Node.js API       │    │
│  │   Core      │ │   Modules   │ │   Compatibility     │    │
│  │   (JSC)     │ │   (Zig)     │ │   Layer             │    │
│  └─────────────┘ └─────────────┘ └─────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      Platform Layer                         │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐    │
│  │  File I/O   │ │  Network    │ │   SQLite           │    │
│  │  (Zig)      │ │  (libuv)    │ │   (built-in)       │    │
│  └─────────────┘ └─────────────┘ └─────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### 内置模块

```typescript
// Bun 特有的内置模块
import { 
  file,          // 文件操作
  mkdir,         // 创建目录
  Glob,          // 文件模式匹配
  spawn,         // 子进程
  which,         // 查找可执行文件
  sha256,        // 哈希计算
  hashSync,      // 同步哈希
  passwordHash,  // 密码哈希
  passwordVerify,// 密码验证
  generatekey,   // 生成密钥
  serve,         // HTTP 服务器
  upgrade,       // 自动更新
} from "bun";

// 文件操作
const contents = await file("package.json").text();
const json = await file("data.json").json();

// SQLite 数据库
import { Database } from "bun:sqlite";
const db = new Database("app.db");
const users = db.query("SELECT * FROM users").all();
```

### Node.js 兼容性

```typescript
// Bun 对 Node.js 内置模块的兼容支持
// fs 模块
import { readFile, writeFile, readdir } from "fs/promises";
import { existsSync } from "fs";

// path 模块
import { join, resolve, dirname } from "path";

// process 模块
console.log(process.env.NODE_ENV);
console.log(process.cwd());

// crypto 模块（部分）
import { createHash, randomBytes } from "crypto";
const hash = createHash("sha256").update("hello").digest("hex");

// stream 模块
import { Readable, Writable } from "stream";

// http 模块
import http from "http";
const server = http.createServer((req, res) => {
  res.end("Hello");
});
```

### Bun 原生 API

```typescript
// Bun.Glob - 文件模式匹配
import { Glob } from "bun";
const glob = new Glob("**/*.ts");

for await (const file of glob.scan({
  cwd: "./src",
  onlyFiles: true
})) {
  console.log(file);  // src/index.ts, src/utils/helper.ts
}

// Bun.spawn - 子进程管理
import { spawn } from "bun";
const child = spawn(["git", "status"], {
  stdout: "inherit",
  stderr: "inherit",
});
const exitCode = await child.exited;

// Bun.FileSink - 高效文件写入
const file = new Bun.FileSink("output.txt");
file.write("Hello ");
file.write("World!");
await file.end();

// Bun.TempFile - 临时文件
const temp = new Bun.TempFile();
temp.write("temporary data");
temp.flush();
```

---

## 内置工具链

### Bun CLI 命令

| 命令 | 功能 | 示例 |
|------|------|------|
| `bun run` | 运行脚本 | `bun run index.ts` |
| `bun test` | 运行测试 | `bun test` |
| `bun install` | 安装依赖 | `bun install` |
| `bun add` | 添加依赖 | `bun add express` |
| `bun remove` | 移除依赖 | `bun remove lodash` |
| `bun build` | 打包应用 | `bun build ./src/index.ts` |
| `bun init` | 初始化项目 | `bun init` |
| `bun create` | 创建模板 | `bun create next-app` |
| `bun x` | 执行包 | `bun x prisma generate` |
| `bun pm` | 包管理 | `bun pm ls` |

### Bun Test 测试框架

```typescript
// __tests__/user.test.ts
import { describe, test, expect, beforeAll, afterAll } from "bun:test";
import { jest } from "@jest/globals";

// 完整的 Jest 兼容 API
describe("User Service", () => {
  let userService: UserService;
  
  beforeAll(() => {
    userService = new UserService();
  });
  
  test("should create a user", async () => {
    const user = await userService.create({
      username: "testuser",
      email: "test@example.com",
    });
    
    expect(user.id).toBeGreaterThan(0);
    expect(user.username).toBe("testuser");
    expect(user.email).toBe("test@example.com");
  });
  
  test("should throw on duplicate email", async () => {
    await expect(
      userService.create({
        username: "another",
        email: "test@example.com",  // 已存在
      })
    ).rejects.toThrow("Email already exists");
  });
  
  test("should validate email format", async () => {
    const invalidEmails = [
      "notanemail",
      "@nodomain.com",
      "noat.com",
    ];
    
    for (const email of invalidEmails) {
      await expect(
        userService.create({
          username: "user",
          email,
        })
      ).rejects.toThrow("Invalid email format");
    }
  });
});

// 模拟测试
import { jest } from "@jest/globals";

test("should call onSuccess callback", async () => {
  const onSuccess = jest.fn();
  const onError = jest.fn();
  
  await processUser({ id: 1 }, { onSuccess, onError });
  
  expect(onSuccess).toHaveBeenCalledWith({ id: 1, processed: true });
  expect(onError).not.toHaveBeenCalled();
});
```

### Bun Build 打包器

```bash
# 基本打包
bun build ./src/index.ts --outdir ./dist

# 浏览器打包
bun build ./src/index.ts \
  --outfile ./dist/bundle.js \
  --target browser \
  --minify

# Node.js 打包
bun build ./src/index.ts \
  --outfile ./dist/bundle.js \
  --target node

# 库打包
bun build ./src/index.ts \
  --outfile ./dist/index.js \
  --target bun \
  --format esm

# 指定入口
bun build \
  --entrypoints ./src/index.ts \
  --outdir ./dist \
  --target bun
```

### Bun Init 项目初始化

```bash
# 初始化新项目
bun init

# 创建特定模板项目
bun create next ./my-next-app
bun create hono ./my-hono-api
bun create sveltekit ./my-svelte-app
bun create express ./my-express-api
bun create fastify ./my-fastify-api
```

---

## Web 服务器 Bun.serve

### 基础 HTTP 服务器

```typescript
// server.ts
Bun.serve({
  port: 3000,
  
  fetch(req) {
    const url = new URL(req.url);
    
    if (url.pathname === "/") {
      return new Response("Hello from Bun!", {
        headers: { "Content-Type": "text/plain" },
      });
    }
    
    if (url.pathname === "/api/users") {
      return Response.json([
        { id: 1, name: "Alice" },
        { id: 2, name: "Bob" },
      ]);
    }
    
    return new Response("Not Found", { status: 404 });
  },
  
  error(error) {
    console.error("Server error:", error);
    return new Response("Internal Server Error", { status: 500 });
  },
});

console.log("Server running on http://localhost:3000");
```

### RESTful API 服务器

```typescript
import { type Context } from "hono";

// 模拟数据库
const users = new Map<number, { id: number; name: string; email: string }>();
let nextId = 1;

const server = Bun.serve({
  port: 3000,
  
  async fetch(req) {
    const url = new URL(req.url);
    const path = url.pathname;
    const method = req.method;
    
    try {
      // GET /api/users - 获取用户列表
      if (path === "/api/users" && method === "GET") {
        const allUsers = Array.from(users.values());
        return Response.json({ users: allUsers });
      }
      
      // GET /api/users/:id - 获取单个用户
      const userMatch = path.match(/^\/api\/users\/(\d+)$/);
      if (userMatch && method === "GET") {
        const id = parseInt(userMatch[1]);
        const user = users.get(id);
        if (!user) {
          return Response.json({ error: "User not found" }, { status: 404 });
        }
        return Response.json({ user });
      }
      
      // POST /api/users - 创建用户
      if (path === "/api/users" && method === "POST") {
        const body = await req.json();
        
        // 验证
        if (!body.name || !body.email) {
          return Response.json(
            { error: "Name and email are required" },
            { status: 400 }
          );
        }
        
        const id = nextId++;
        const user = { id, name: body.name, email: body.email };
        users.set(id, user);
        
        return Response.json({ user }, { status: 201 });
      }
      
      // PUT /api/users/:id - 更新用户
      if (userMatch && method === "PUT") {
        const id = parseInt(userMatch[1]);
        const user = users.get(id);
        
        if (!user) {
          return Response.json({ error: "User not found" }, { status: 404 });
        }
        
        const body = await req.json();
        const updated = { ...user, ...body, id };
        users.set(id, updated);
        
        return Response.json({ user: updated });
      }
      
      // DELETE /api/users/:id - 删除用户
      if (userMatch && method === "DELETE") {
        const id = parseInt(userMatch[1]);
        
        if (!users.has(id)) {
          return Response.json({ error: "User not found" }, { status: 404 });
        }
        
        users.delete(id);
        return new Response(null, { status: 204 });
      }
      
      return new Response("Not Found", { status: 404 });
      
    } catch (error) {
      console.error("Error:", error);
      return Response.json(
        { error: "Internal Server Error" },
        { status: 500 }
      );
    }
  },
});

console.log(`Server running at http://localhost:${server.port}`);
```

### WebSocket 服务器

```typescript
Bun.serve({
  port: 3000,
  
  websocket: {
    open(ws) {
      console.log("Client connected");
      ws.send("Welcome to the chat!");
    },
    
    message(ws, message) {
      // 处理消息
      const data = typeof message === "string" 
        ? JSON.parse(message) 
        : message;
      
      if (data.type === "broadcast") {
        // 广播给所有连接的客户端
        ws.publish("chat", JSON.stringify({
          type: "message",
          content: data.content,
          timestamp: Date.now(),
        }));
      } else {
        // 回应发送者
        ws.send(JSON.stringify({
          type: "echo",
          content: data.content,
        }));
      }
    },
    
    close(ws, code, reason) {
      console.log(`Client disconnected: ${code} - ${reason}`);
    },
    
    ping(ws, data) {
      // 处理 ping
    },
  },
  
  fetch(req, server) {
    const url = new URL(req.url);
    
    // WebSocket 升级
    if (url.pathname === "/ws") {
      const success = server.upgrade(req, {
        data: { userId: generateUserId() },
      });
      
      if (success) {
        return undefined;  // 升级成功，不返回响应
      }
      return new Response("WebSocket upgrade failed", { status: 500 });
    }
    
    // 普通 HTTP 请求
    return new Response("Bun WebSocket Server");
  },
});

function generateUserId(): string {
  return Math.random().toString(36).substring(2, 15);
}
```

### 静态文件服务器

```typescript
Bun.serve({
  port: 3000,
  
  fetch(req) {
    const url = new URL(req.url);
    
    // API 路由
    if (url.pathname.startsWith("/api/")) {
      return handleAPI(req);
    }
    
    // 静态文件服务
    let filePath = url.pathname === "/" 
      ? "/index.html" 
      : url.pathname;
    
    const file = Bun.file(`./public${filePath}`);
    
    if (await file.exists()) {
      const mimeType = getMimeType(filePath);
      return new Response(file, {
        headers: { "Content-Type": mimeType },
      });
    }
    
    // SPA 回退
    const indexFile = Bun.file("./public/index.html");
    return new Response(indexFile, {
      headers: { "Content-Type": "text/html" },
    });
  },
});

function getMimeType(path: string): string {
  const ext = path.split(".").pop()?.toLowerCase();
  const mimeTypes: Record<string, string> = {
    html: "text/html",
    css: "text/css",
    js: "application/javascript",
    json: "application/json",
    png: "image/png",
    jpg: "image/jpeg",
    svg: "image/svg+xml",
  };
  return mimeTypes[ext || ""] || "application/octet-stream";
}
```

---

## 包管理器 Bun.install

### 核心命令

```bash
# 安装所有依赖
bun install

# 添加依赖
bun add express
bun add -D typescript @types/node
bun add zod

# 移除依赖
bun remove lodash

# 更新依赖
bun update
bun update express

# 锁定文件
bun.lockb  # 自动生成/更新

# 查看已安装的包
bun pm ls
bun pm ls --depth 2

# 缓存管理
bun pm cache
bun pm cache rm
```

### npm 兼容性

```bash
# Bun 可以直接使用 npm 包
bun add express
bun add react
bun add @types/react

# 也可以使用 pnpm workspace 格式
# bun.lockb 兼容 package-lock.json 格式
```

### Workspaces 支持

```json
// package.json (根目录)
{
  "name": "my-monorepo",
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "dev": "bun --cwd packages/web dev",
    "api": "bun --cwd packages/api dev"
  }
}
```

---

## Bun vs Node.js vs Deno 对比

### 核心对比表

| 特性 | Bun | Node.js | Deno |
|------|-----|--------|------|
| **最新稳定版本** | 1.3.x | 22.x | 2.x |
| **JavaScript 引擎** | JavaScriptCore | V8 | V8 |
| **编写语言** | Zig | C++ | Rust |
| **启动时间** | <10ms | ~100ms | ~50ms |
| **npm 兼容性** | 99%+ | 100% | 有限 |
| **TypeScript** | 原生 | 需转译 | 原生 |
| **权限沙箱** | 无 | 无 | 有 |
| **内置打包** | 是 | 否 | 否 |
| **内置测试** | 是 | 否 | 是 |
| **内置数据库** | SQLite | 无 | 无 |
| **ESM 优先** | 是 | 可选 | 是 |
| **NPM 生态** | 完整 | 完整 | 需适配 |
| **长期维护** | Startup 驱动 | OpenJS 基金会 | Ryan Dahl 团队 |

### 功能覆盖对比

| 功能 | Bun | Node.js | Deno |
|------|-----|--------|------|
| **HTTP 服务器** | Bun.serve | http/https | Deno.serve |
| **WebSocket** | 内置 | 需库 | 内置 |
| **文件系统** | fs + Bun.* | fs | Deno.readFile |
| **Crypto** | 部分 | 完整 | 完整 |
| **SQLite** | 内置 | 需库 | 需库 |
| **Worker Threads** | 支持 | 支持 | Workers |
| **Child Process** | spawn | child_process | Deno.Command |

### 性能对比表

| 操作 | Bun | Node.js | Deno |
|------|-----|--------|------|
| **冷启动** | 10ms | 100ms | 50ms |
| **HTTP 请求/秒** | 80,000+ | 50,000+ | 45,000+ |
| **文件读取** | 快 3x | 基准 | 快 2x |
| **包安装** | 快 10x | 基准 | 快 5x |
| **TypeScript 转译** | 原生 | 需 tsc | 原生 |

> [!TIP]
> **性能基准来源**：Bun 官方 benchmarks (2025)，实际性能因场景而异。

---

## 性能基准

### 官方基准测试（2025）

| 测试场景 | Bun | Node.js | 提升倍数 |
|----------|-----|---------|----------|
| **HTTP 并发（req/s）** | 85,000 | 48,000 | 1.8x |
| **文件读取（MB/s）** | 2,800 | 950 | 2.9x |
| **包安装（s）** | 1.2s | 12s | 10x |
| **冷启动（ms）** | 8ms | 95ms | 12x |
| **JSON 解析（ops/s）** | 1,200,000 | 580,000 | 2.1x |
| **数据库查询（ms）** | 12ms | 18ms | 1.5x |

### 实际场景测试

```typescript
// benchmark.ts
import { performance } from "bun";

async function benchmark(name: string, fn: () => Promise<void>) {
  const start = performance.now();
  await fn();
  const end = performance.now();
  console.log(`${name}: ${(end - start).toFixed(2)}ms`);
}

// HTTP 服务器基准
await benchmark("HTTP Server - 1000 requests", async () => {
  const results = await Promise.all(
    Array.from({ length: 1000 }, () => 
      fetch("http://localhost:3000/api/users")
    )
  );
});

// 并发数据库查询
await benchmark("DB Query - 100 concurrent", async () => {
  await Promise.all(
    Array.from({ length: 100 }, () => 
      db.query("SELECT * FROM users WHERE id = ?", [Math.floor(Math.random() * 1000)])
    )
  );
});
```

---

## 实战场景与选型建议

### 场景一：高性能 API 服务器

```typescript
// api/server.ts
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";

const app = new Hono();

// 中间件
app.use("*", cors());
app.use("*", logger());

// 路由
const users = new Map<string, User>();

app.get("/api/users", (c) => {
  return c.json(Array.from(users.values()));
});

app.get("/api/users/:id", (c) => {
  const id = c.req.param("id");
  const user = users.get(id);
  if (!user) return c.notFound();
  return c.json(user);
});

app.post("/api/users", async (c) => {
  const body = await c.req.json();
  const id = crypto.randomUUID();
  const user = { id, ...body };
  users.set(id, user);
  return c.json(user, 201);
});

app.put("/api/users/:id", async (c) => {
  const id = c.req.param("id");
  if (!users.has(id)) return c.notFound();
  const body = await c.req.json();
  users.set(id, { ...body, id });
  return c.json(users.get(id));
});

app.delete("/api/users/:id", (c) => {
  const id = c.req.param("id");
  if (!users.delete(id)) return c.notFound();
  return c.body(null, 204);
});

// 启动服务器
export default {
  port: 3000,
  fetch: app.fetch,
};
```

### 场景二：CLI 工具

```typescript
#!/usr/bin/env bun
// cli.ts

import { parseArgs } from "util";

const { values, positionals } = parseArgs({
  args: Bun.argv.slice(2),
  options: {
    verbose: { type: "boolean", short: "v" },
    config: { type: "string", short: "c" },
    output: { type: "string", short: "o" },
  },
  allowPositionals: true,
});

function log(...args: any[]) {
  if (values.verbose) {
    console.log("[INFO]", ...args);
  }
}

function main() {
  const command = positionals[0];
  
  switch (command) {
    case "build":
      log("Building project...");
      build();
      break;
    case "dev":
      log("Starting dev server...");
      dev();
      break;
    case "deploy":
      log("Deploying to production...");
      deploy();
      break;
    default:
      console.log(`Usage: mycli <command> [options]
Commands:
  build     Build the project
  dev       Start development server
  deploy    Deploy to production
Options:
  -v, --verbose    Enable verbose logging
  -c, --config    Config file path
  -o, --output    Output directory`);
  }
}

function build() {
  console.log("Build complete!");
}

function dev() {
  console.log("Dev server running on http://localhost:3000");
}

function deploy() {
  console.log("Deployment complete!");
}

main();
```

### 场景三：SQLite 数据处理

```typescript
// data-processor.ts
import { Database } from "bun:sqlite";
import { writeFileSync } from "fs";

// 连接数据库
const db = new Database("data.db", { create: true });

// 创建表
db.run(`
  CREATE TABLE IF NOT EXISTS products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    price REAL NOT NULL,
    category TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
  )
`);

// 插入数据
const insert = db.prepare(`
  INSERT INTO products (name, price, category) VALUES (?, ?, ?)
`);

// 批量插入
const products = [
  ["Laptop", 999.99, "Electronics"],
  ["Headphones", 149.99, "Electronics"],
  ["Desk", 299.99, "Furniture"],
  ["Chair", 199.99, "Furniture"],
  ["Book", 19.99, "Education"],
];

const insertMany = db.transaction((items: typeof products) => {
  for (const [name, price, category] of items) {
    insert.run(name, price, category);
  }
});

insertMany(products);

// 查询
const query = db.query("SELECT * FROM products WHERE price > ?");
const expensiveProducts = query.all(100);

console.log("Products over $100:", expensiveProducts);

// 聚合查询
const stats = db.query(`
  SELECT 
    category,
    COUNT(*) as count,
    AVG(price) as avg_price,
    MIN(price) as min_price,
    MAX(price) as max_price
  FROM products
  GROUP BY category
`).all();

console.log("\nCategory Statistics:", stats);

// 导出为 JSON
const allProducts = db.query("SELECT * FROM products").all();
writeFileSync("products.json", JSON.stringify(allProducts, null, 2));

db.close();
```

### 选型决策树

```
项目类型
├── 高性能 API 服务
│   ├── 追求极致性能 → Bun
│   ├── 稳定压倒一切 → Node.js
│   └── 需安全沙箱 → Deno
├── 快速原型/脚本
│   └── → Bun（安装快、启动快）
├── 微服务架构
│   ├── 新项目 → Bun
│   └── 已有 Node.js → 逐步迁移或保持
├── CLI 工具
│   └── → Bun（单文件分发）
├── AI/ML 应用
│   ├── 本地推理 → Python
│   └── API 服务 → Bun/Fastify
└── 团队技能
    ├── Node.js 团队 → 逐步采用 Bun
    ├── 新项目 → 优先 Bun
    └── 稳定性优先 → Node.js
```

> [!SUCCESS]
> 本文档全面覆盖了 Bun 的核心知识，包括运行时架构、内置工具链、`Bun.serve` Web 服务器、Node.js 兼容性及性能优化。Bun 作为 2026 年最具潜力的 JavaScript 运行时，以其极快的启动速度、内置工具链和完善的生态兼容性，正在改变 JavaScript 开发的格局。对于追求开发效率和运行时性能的团队，Bun 是值得认真考虑的选择。

---

## 完整安装与环境配置

### 环境要求

```bash
# Bun 要求
# - macOS 10.14+ 或 Linux
# - x64 或 ARM64 架构
```

### 安装方法

```bash
# macOS/Linux 安装
curl -fsSL https://bun.sh/install | bash

# Homebrew
brew install bun

# npm
npm install -g bun

# Windows (通过 WSL 或 Docker)
# 使用 Docker
docker run --rm --init -p 3000:3000 -v $(pwd):/app -w /app oven/bun bun run index.ts
```

### 项目创建

```bash
# 创建新项目
bun create
# 或
bun create express myapp

# 初始化已有项目
bun init
```

### package.json 配置

```json
{
  "name": "my-bun-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "bun run --watch src/index.ts",
    "start": "bun run src/index.ts",
    "build": "bun build src/index.ts --outdir=dist --target=browser",
    "test": "bun test",
    "lint": "eslint src --ext .ts"
  },
  "dependencies": {
    "@prisma/client": "^6.0.0",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "typescript": "^5.3.0",
    "@types/node": "^20.10.0",
    "prisma": "^6.0.0",
    "eslint": "^8.55.0"
  }
}
```

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ESNext"],
    "types": ["bun-types"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "allowImportingTsExtensions": true,
    "noEmit": true
  },
  "include": ["src/**/*"]
}
```

---

## 数据库集成

### Prisma with Bun

```bash
bun add prisma @prisma/client
bunx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

model User {
  id       String   @id @default(uuid())
  email    String   @unique
  username String   @unique
  password String
  role     String   @default("USER")
  createdAt DateTime @default(now())
  posts    Post[]
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

```typescript
// src/db.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export { prisma };

// 使用示例
async function main() {
  const users = await prisma.user.findMany({
    include: { posts: true }
  });
  console.log(users);
}

main()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

### Drizzle with Bun

```bash
bun add drizzle-orm better-sqlite3
bun add -d drizzle-kit @types/better-sqlite3
```

```typescript
// src/db/schema.ts
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core';

export const users = sqliteTable('users', {
  id: text('id').primaryKey(),
  email: text('email').notNull().unique(),
  username: text('username').notNull().unique(),
  password: text('password').notNull(),
  role: text('role').default('USER'),
  createdAt: integer('created_at', { mode: 'timestamp' }).$defaultFn(() => new Date())
});

export const posts = sqliteTable('posts', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  published: integer('published', { mode: 'boolean' }).default(false),
  authorId: text('author_id').references(() => users.id),
  createdAt: integer('created_at', { mode: 'timestamp' }).$defaultFn(() => new Date())
});
```

---

## 认证授权

### JWT 实现

```typescript
// src/auth.ts
import { Context } from 'hono';
import { SignJWT, jwtVerify } from 'jose';

const JWT_SECRET = new TextEncoder().encode(
  Bun.env.JWT_SECRET || 'secret-key'
);

export interface JWTPayload {
  sub: string;
  email: string;
  role: string;
}

export async function createToken(payload: JWTPayload): Promise<string> {
  return await new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setIssuedAt()
    .setExpirationTime('7d')
    .sign(JWT_SECRET);
}

export async function verifyToken(token: string): Promise<JWTPayload | null> {
  try {
    const { payload } = await jwtVerify(token, JWT_SECRET);
    return payload as unknown as JWTPayload;
  } catch {
    return null;
  }
}

export function authMiddleware() {
  return async (c: Context, next: () => Promise<void>) => {
    const authHeader = c.req.header('Authorization');
    
    if (!authHeader?.startsWith('Bearer ')) {
      return c.json({ error: 'Unauthorized' }, 401);
    }

    const token = authHeader.substring(7);
    const payload = await verifyToken(token);

    if (!payload) {
      return c.json({ error: 'Invalid token' }, 401);
    }

    c.set('user', payload);
    await next();
  };
}
```

---

## 性能优化

### 最佳实践

1. **使用 Bun.sql**：内置 SQLite 支持，性能优异。
2. **批量操作**：使用事务批量插入/更新。
3. **索引**：为常用查询字段添加索引。
4. **缓存**：使用内存缓存热点数据。
5. **连接池**：合理配置数据库连接池。

```typescript
// 性能优化示例
const db = new Database('app.db');

// 启用 WAL 模式
db.run('PRAGMA journal_mode=WAL');

// 创建索引
db.run('CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)');

// 批量插入
const insert = db.prepare('INSERT INTO users VALUES (?, ?, ?)');
const insertMany = db.transaction((users: any[]) => {
  for (const user of users) {
    insert.run(user.id, user.email, user.username);
  }
});

insertMany([
  { id: '1', email: 'a@test.com', username: 'a' },
  { id: '2', email: 'b@test.com', username: 'b' }
]);
```

---

## 常见陷阱与最佳实践

### 陷阱 1：混淆 Bun 和 Node.js API

```typescript
// ❌ 错误：使用 Node.js 特定 API
import fs from 'fs';
fs.readFileSync('file.txt');

// ✅ 正确：使用 Web 标准 API
import { readFileSync } from 'fs';
// 或
const content = Bun.file('file.txt').text();
```

### 陷阱 2：忘记处理异步错误

```typescript
// ❌ 错误
Bun.serve({
  fetch(req) {
    db.query('SELECT * FROM users').all(); // 可能抛出错误
  }
});

// ✅ 正确
Bun.serve({
  fetch(req) {
    try {
      return new Response(JSON.stringify(db.query('SELECT * FROM users').all()));
    } catch (error) {
      return new Response('Error', { status: 500 });
    }
  }
});
```

### 最佳实践清单

1. **使用 bun-types**：获取完整的类型提示。
2. **使用内置工具**：优先使用 `bun run`、`bun test` 等。
3. **兼容性测试**：生产环境前在目标平台测试。
4. **关注更新**：Bun 仍处于快速发展阶段。
5. **错误处理**：做好异常捕获和处理。
6. **性能优化**：利用 Bun 的内置优化特性。
7. **模块解析**：使用 `moduleResolution: "bundler"`。

---

## 与其他运行时对比

### Bun vs Node.js

| 特性 | Bun | Node.js |
|------|-----|---------|
| **启动速度** | 极快 | 较慢 |
| **运行速度** | 快 | 快 |
| **npm 兼容性** | 99% | 100% |
| **内置工具** | 完整工具链 | 需要额外安装 |
| **稳定性** | 发展中 | 非常稳定 |
| **生态** | 增长中 | 庞大 |

### Bun vs Deno

| 特性 | Bun | Deno |
|------|-----|------|
| **性能** | 更快 | 快 |
| **npm 兼容** | 更好 | 需配置 |
| **权限系统** | 无 | 有 |
| **稳定性** | 发展中 | 较稳定 |
| **工具链** | 完整 | 需要安装 |

---

> [!IMPORTANT]
> **迁移注意事项**：
> 1. Bun 对 npm 包兼容性约 99%，少数冷门包可能不兼容
> 2. 部分 Node.js API（如某些 crypto 功能）尚未实现
> 3. 生产环境建议先在非关键服务测试
> 4. libuv 行为差异可能导致边缘情况问题

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Bun 官方文档 | https://bun.sh/docs |
| Bun GitHub | https://github.com/oven-sh/bun |
| Bun Discord | https://discord.gg/bun |

### 性能相关

| 资源 | 说明 |
|------|------|
| Bun 官方 Benchmarks | 官方性能测试数据 |
| Bun vs Node vs Deno | 第三方对比测试 |

---

## 核心概念详解

### Bun.serve 深入理解

`Bun.serve` 是 Bun 运行时提供的内置 HTTP 服务器 API，它基于底层的 HTTP 服务器实现，提供了极高的性能和极低的延迟。理解其内部机制对于构建高性能应用至关重要。

#### 请求处理生命周期

```typescript
// 请求处理完整生命周期
const server = Bun.serve({
  port: 3000,
  hostname: "0.0.0.0",
  
  // 请求到达前的钩子
  // 可用于验证、限流等前置处理
  beforeListen() {
    console.log("Server is about to start...");
  },
  
  // 服务器启动后的钩子
  // 可用于记录、日志初始化等
  afterListen() {
    console.log(`Server running on http://${this.hostname}:${this.port}`);
  },
  
  // 主要请求处理函数
  async fetch(request: Request, server: Server): Response | Promise<Response> {
    const url = new URL(request.url);
    
    // 请求日志中间件逻辑
    const start = performance.now();
    const method = request.method;
    const pathname = url.pathname;
    
    try {
      // 路由处理
      let response = await routeRequest(request, url);
      
      // 响应后处理
      const duration = performance.now() - start;
      console.log(`${method} ${pathname} - ${response.status} (${duration.toFixed(2)}ms)`);
      
      // 添加自定义响应头
      response.headers.set("X-Response-Time", `${duration.toFixed(2)}ms`);
      response.headers.set("X-Server", "Bun");
      
      return response;
      
    } catch (error) {
      console.error(`Error handling ${method} ${pathname}:`, error);
      return new Response(JSON.stringify({
        error: "Internal Server Error",
        message: error instanceof Error ? error.message : "Unknown error"
      }), {
        status: 500,
        headers: { "Content-Type": "application/json" }
      });
    }
  },
  
  // 错误处理函数
  error(error: Error, request: Request): Response {
    console.error("Unhandled error:", error);
    return new Response("Internal Server Error", { status: 500 });
  },
  
  // TLS 配置（可选）
  // tls: {
  //   key: Bun.file("./key.pem"),
  //   cert: Bun.file("./cert.pem"),
  // }
});

// 路由处理函数
async function routeRequest(request: Request, url: URL): Promise<Response> {
  const method = request.method;
  const path = url.pathname;
  
  // RESTful 路由映射
  if (path === "/api/health" && method === "GET") {
    return Response.json({ status: "healthy", timestamp: Date.now() });
  }
  
  // 用户资源路由
  const userMatch = path.match(/^\/api\/users(?:\/(\d+))?$/);
  if (userMatch) {
    const userId = userMatch[1];
    return handleUsersRequest(request, userId);
  }
  
  // 文章资源路由
  const postMatch = path.match(/^\/api\/posts(?:\/(\d+))?$/);
  if (postMatch) {
    const postId = postMatch[1];
    return handlePostsRequest(request, postId);
  }
  
  // 404 处理
  return Response.json({ error: "Not Found" }, { status: 404 });
}
```

#### 连接升级处理

Bun.serve 支持多种协议升级，最常见的是 WebSocket 升级。理解连接升级机制对于构建实时应用至关重要。

```typescript
// WebSocket 升级处理
const chatServer = Bun.serve({
  port: 3000,
  
  fetch(request, server) {
    const url = new URL(request.url);
    
    // WebSocket 握手
    if (url.pathname === "/ws/chat") {
      const success = server.upgrade(request, {
        data: {
          userId: crypto.randomUUID(),
          connectedAt: Date.now(),
        },
        // 可选的二进制协议
        // binary: true,
      });
      
      if (success) {
        // 升级成功，不返回响应
        return;
      }
      
      // 升级失败
      return new Response("WebSocket upgrade failed", { status: 500 });
    }
    
    // 普通 HTTP 请求
    return new Response("Chat Server");
  },
  
  websocket: {
    // 打开连接
    open(ws) {
      const data = ws.data;
      console.log(`User ${data.userId} connected at ${new Date(data.connectedAt).toISOString()}`);
      
      // 广播用户上线消息
      broadcastMessage({
        type: "system",
        content: `User ${data.userId.slice(0, 8)} joined`,
        timestamp: Date.now(),
      });
      
      // 设置心跳
      ws.send(JSON.stringify({ type: "ping" }));
    },
    
    // 接收消息
    message(ws, message) {
      const data = ws.data;
      
      // 解析消息
      let parsed;
      try {
        parsed = typeof message === "string" 
          ? JSON.parse(message) 
          : message;
      } catch {
        ws.send(JSON.stringify({ type: "error", content: "Invalid JSON" }));
        return;
      }
      
      // 处理不同类型的消息
      switch (parsed.type) {
        case "chat":
          // 聊天消息
          broadcastMessage({
            type: "chat",
            userId: data.userId,
            content: parsed.content,
            timestamp: Date.now(),
          });
          break;
          
        case "ping":
          // 心跳响应
          ws.send(JSON.stringify({ type: "pong", timestamp: Date.now() }));
          break;
          
        case "join-room":
          // 加入房间
          ws.subscribe(`room:${parsed.room}`);
          ws.send(JSON.stringify({ 
            type: "joined-room", 
            room: parsed.room 
          }));
          break;
          
        case "leave-room":
          // 离开房间
          ws.unsubscribe(`room:${parsed.room}`);
          break;
          
        default:
          ws.send(JSON.stringify({ 
            type: "error", 
            content: "Unknown message type" 
          }));
      }
    },
    
    // 关闭连接
    close(ws, code, reason) {
      const data = ws.data;
      console.log(`User ${data.userId} disconnected: ${code} - ${reason}`);
      
      broadcastMessage({
        type: "system",
        content: `User ${data.userId.slice(0, 8)} left`,
        timestamp: Date.now(),
      });
    },
    
    // 发送消息（可用于服务端推送）
    send(ws, message) {
      // 自定义发送逻辑
    },
    
    // 消息压缩配置
    perMessageDeflate: {
      // 客户端可接受的压缩配置
      clientMaxWindowBits: 15,
      serverMaxWindowBits: 15,
      // 压缩阈值，小于此大小的消息不压缩
      threshold: 1024,
    },
    
    // ping/pong 配置
    pingInterval: 30000,  // 30秒发送一次 ping
    pingTimeout: 5000,     // 5秒内未收到 pong 则断开
  },
});

// 消息广播
const clients = new Set<any>();

function broadcastMessage(message: object) {
  const payload = JSON.stringify(message);
  for (const client of clients) {
    client.send(payload);
  }
}
```

#### Server 实例方法与属性

```typescript
// Server 实例的完整 API
const server = Bun.serve({
  port: 3000,
  fetch(request) {
    // 动态添加路由
    server.router;
    
    // 获取连接信息
    server.url;
    server.port;
    server.hostname;
    
    return new Response("OK");
  },
});

// 服务器控制方法
async function controlServer() {
  // 停止服务器
  server.stop();
  
  // 重新启动
  server.start({ port: 4000 });
  
  // 获取活跃连接数
  const connections = server.connections;
  console.log(`Active connections: ${connections}`);
  
  // 升级现有连接
  // server.upgrade(request, options);
  
  // 广播到所有 WebSocket 连接
  // server.publish(channel, message);
}
```

### Bun 模块系统

Bun 提供了一套完整的内置模块系统，覆盖了文件系统、网络、加密、数据库等常用功能。深入理解这些模块可以大幅提升开发效率。

#### 文件系统模块

Bun 的文件系统 API 相比 Node.js 更加现代化和高效，充分利用了 Zig 的性能优势。

```typescript
import { 
  file,           // 文件读取
  writeFile,      // 同步写入
  writeFileSync,  // 同步写入
  mkdir,          // 创建目录
  mkdirSync,      // 同步创建目录
  readdir,        // 读取目录
  readdirSync,    // 同步读取目录
  rm,             // 删除文件/目录
  rmSync,         // 同步删除
  cp,             // 复制文件/目录
  cpSync,         // 同步复制
  rename,         // 重命名/移动
  renameSync,     // 同步重命名/移动
  stat,           // 文件状态
  statSync,       // 同步文件状态
  exists,         // 检查是否存在
  watch,          // 监听文件变化
} from "bun";

// 高效文件读取
async function efficientFileReading() {
  // 方式一：使用 Bun.file（推荐）
  const content = await Bun.file("./large-file.json").text();
  const json = await Bun.file("./data.json").json();
  const arrayBuffer = await Bun.file("./binary.dat").arrayBuffer();
  
  // 方式二：流式读取大文件
  const largeFile = Bun.file("./huge-file.zip");
  const stream = largeFile.stream();
  
  // 方式三：分片读取
  const file = Bun.file("./file.txt");
  const slice = await file.slice(0, 1024);  // 读取前 1KB
  const sliceText = await slice.text();
  
  // 文件元数据
  const fileInfo = await file.lastModified();  // 最后修改时间
  const fileSize = await file.size;            // 文件大小
  const fileType = await file.type;            // MIME 类型
}

// 文件写入
async function fileWriting() {
  // 字符串写入
  await Bun.write("./output.txt", "Hello, Bun!");
  
  // 数组写入
  await Bun.write("./output.txt", ["Line 1", "Line 2", "Line 3"].join("\n"));
  
  // 二进制写入
  const buffer = new ArrayBuffer(1024);
  await Bun.write("./binary.bin", buffer);
  
  // 使用 WriteFile
  await writeFile("./output.json", JSON.stringify({ hello: "world" }));
  
  // 使用 FileSink 进行多次写入
  const sink = new Bun.FileSink("./stream.txt");
  sink.write("Part 1\n");
  sink.write("Part 2\n");
  sink.write("Part 3\n");
  await sink.end();
}

// 目录操作
async function directoryOperations() {
  // 创建嵌套目录
  await mkdir("./a/b/c", { recursive: true });
  
  // 读取目录
  const entries = await readdir("./src");
  for (const entry of entries) {
    console.log(entry.name, entry.isDirectory() ? "[DIR]" : "[FILE]");
  }
  
  // 递归读取目录
  const allFiles = await readdir("./src", { recursive: true });
  
  // 监视文件变化
  const watcher = watch("./src", async (event, filename) => {
    console.log(`${event}: ${filename}`);
    if (event === "modify") {
      // 重新加载配置或执行构建
    }
  });
  
  // 停止监视
  // watcher.stop();
}

// 路径操作
import { resolve, join, dirname, basename, extname, relative } from "path";

// 路径解析示例
function pathExamples() {
  const filePath = "/Users/name/project/src/utils/helper.ts";
  
  resolve("./config");      // 解析为绝对路径
  join("src", "utils");     // 连接路径
  dirname(filePath);         // /Users/name/project/src/utils
  basename(filePath);        // helper.ts
  extname(filePath);        // .ts
  relative("/a/b", "/a/b/c/d");  // c/d
  
  // 跨平台路径处理
  const platformPath = join("folder", "file.txt");
  // 在 Windows 上是 folder\file.txt
  // 在 Unix 上是 folder/file.txt
}

// 临时文件
function tempFileExample() {
  const temp = new Bun.TempFile();
  temp.write("temporary content");
  temp.flush();
  
  console.log("Temp file path:", temp.path);
  
  // 自动清理
  temp.remove();
}

// Glob 文件匹配
import { Glob } from "bun";

async function globExample() {
  const glob = new Glob("**/*.ts");
  
  // 异步遍历
  for await (const file of glob.scan({
    cwd: "./src",
    onlyFiles: true,
    ignore: ["**/*.test.ts", "**/node_modules/**"],
  })) {
    console.log(file);
  }
  
  // 批量获取
  const tsFiles = await Array.fromAsync(
    glob.scan({ cwd: "./src", onlyFiles: true })
  );
}
```

#### 加密与安全模块

Bun 提供了全面的加密功能，支持常见的哈希算法、密钥生成、密码哈希等操作。

```typescript
import { 
  crypto,           // 通用加密 API
  sha256,          // SHA-256 哈希
  sha512,          // SHA-512 哈希
  hashSync,        // 同步哈希
  randomUUID,      // UUID 生成
  generateKey,     // 密钥生成
  passwordHash,    // 密码哈希
  passwordVerify,  // 密码验证
} from "bun";

// 哈希计算
function hashExamples() {
  // SHA-256 哈希
  const hash = sha256("Hello, World!");
  console.log(hash);  // 十六进制字符串
  
  // SHA-512 哈希
  const hash512 = sha512("Hello, World!");
  
  // 使用 hashSync（同步）
  const syncHash = hashSync("text", "SHA-256");
  
  // 自定义哈希算法
  const customHash = crypto.subtle.digestSync(
    "SHA-384",
    new TextEncoder().encode("data")
  );
}

// 密码哈希（Argon2）
async function passwordExamples() {
  const password = "user-password-123";
  
  // 生成密码哈希
  const hash = await passwordHash(password);
  console.log("Hash:", hash);
  
  // 验证密码
  const isValid = await passwordVerify(password, hash);
  console.log("Valid:", isValid);
  
  // 使用不同参数
  const strongHash = await passwordHash(password, {
    algorithm: "argon2id",
    memoryCost: 65536,  // 64 MB
    timeCost: 3,        // 3 次迭代
    threadCount: 4,     // 4 线程
  });
}

// 密钥生成
async function keyExamples() {
  // 生成对称密钥
  const aesKey = await crypto.subtle.generateKey(
    { name: "AES-GCM", length: 256 },
    true,  // 可导出
    ["encrypt", "decrypt"]
  );
  
  // 生成 RSA 密钥对
  const rsaKeyPair = await crypto.subtle.generateKey(
    {
      name: "RSA-OAEP",
      modulusLength: 2048,
      publicExponent: new Uint8Array([1, 0, 1]),
      hash: "SHA-256",
    },
    true,
    ["encrypt", "decrypt"]
  );
  
  // 导出密钥
  const exportedKey = await crypto.subtle.exportKey("raw", aesKey);
}

// UUID 生成
function uuidExample() {
  const uuid1 = randomUUID();
  const uuid2 = crypto.randomUUID();
  console.log(uuid1, uuid2);  // 格式: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
}

// HMAC 计算
async function hmacExample() {
  const key = await crypto.subtle.importKey(
    "raw",
    new TextEncoder().encode("secret-key"),
    { name: "HMAC", hash: "SHA-256" },
    false,
    ["sign", "verify"]
  );
  
  const signature = await crypto.subtle.sign(
    "HMAC",
    key,
    new TextEncoder().encode("message")
  );
  
  const isValid = await crypto.subtle.verify(
    "HMAC",
    key,
    signature,
    new TextEncoder().encode("message")
  );
}
```

#### 子进程与系统交互

```typescript
import { 
  spawn,           // 异步进程
  spawnSync,       // 同步进程
  which,           // 查找可执行文件
} from "bun";

// 异步进程执行
async function spawnExample() {
  // 简单命令
  const child = spawn(["echo", "Hello, Bun!"]);
  const output = await child.text();
  console.log(output);
  
  // 带管道的命令
  const gitLog = spawn(["git", "log", "--oneline", "-10"]);
  const logOutput = await gitLog.text();
  
  // 捕获标准输出和错误
  const npmInstall = spawn(["npm", "install"], {
    cwd: "./project",
    stdout: "pipe",
    stderr: "pipe",
  });
  
  const [stdout, stderr] = await Promise.all([
    npmInstall.stdout.text(),
    npmInstall.stderr.text(),
  ]);
  
  const exitCode = await npmInstall.exited;
  console.log(`Exit code: ${exitCode}`);
  console.log(`Output: ${stdout}`);
  if (stderr) console.error(`Errors: ${stderr}`);
  
  // 环境变量
  const envProcess = spawn(["node", "-e", "console.log(process.env.MY_VAR)"], {
    env: { MY_VAR: "hello-from-bun" },
  });
  
  // 实时输出
  const build = spawn(["npm", "run", "build"]);
  build.stdout.pipeTo(new WritableStream({
    write(chunk) {
      process.stdout.write(chunk);
    }
  }));
  
  await build.exited;
}

// 同步进程执行
function spawnSyncExample() {
  const result = spawnSync(["git", "status"]);
  console.log(result.stdout.toString());
  console.log(result.stderr.toString());
  console.log(`Exit code: ${result.exitCode}`);
}

// 查找可执行文件
function whichExample() {
  const nodePath = which("node");
  const npmPath = which("npm");
  
  if (nodePath) {
    console.log(`Node.js found at: ${nodePath}`);
  } else {
    console.log("Node.js not found");
  }
}
```

---

## CRUD 操作完整代码示例

### 完整 RESTful API 实现

以下是一个完整的用户管理 RESTful API，包含完整的 CRUD 操作、验证、错误处理和响应格式化。

```typescript
// src/server.ts
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import { Database } from "bun:sqlite";

// ==================== 类型定义 ====================

interface User {
  id: number;
  username: string;
  email: string;
  passwordHash: string;
  role: "ADMIN" | "USER" | "MODERATOR";
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

interface CreateUserDTO {
  username: string;
  email: string;
  password: string;
  role?: "ADMIN" | "USER" | "MODERATOR";
}

interface UpdateUserDTO {
  username?: string;
  email?: string;
  password?: string;
  role?: "ADMIN" | "USER" | "MODERATOR";
  isActive?: boolean;
}

// ==================== 数据库初始化 ====================

const db = new Database("./data.db", { create: true });

// 创建表
db.run(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    username TEXT NOT NULL UNIQUE,
    email TEXT NOT NULL UNIQUE,
    password_hash TEXT NOT NULL,
    role TEXT NOT NULL DEFAULT 'USER',
    is_active INTEGER NOT NULL DEFAULT 1,
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now'))
  )
`);

// 创建索引
db.run(`CREATE INDEX IF NOT EXISTS idx_users_email ON users(email)`);
db.run(`CREATE INDEX IF NOT EXISTS idx_users_username ON users(username)`);
db.run(`CREATE INDEX IF NOT EXISTS idx_users_role ON users(role)`);

// ==================== 数据库操作 ====================

class UserRepository {
  private db: Database;
  
  constructor(db: Database) {
    this.db = db;
  }
  
  findAll(page: number = 1, limit: number = 10): { users: User[]; total: number } {
    const offset = (page - 1) * limit;
    
    const users = this.db.query(`
      SELECT id, username, email, password_hash, role, is_active, created_at, updated_at
      FROM users
      ORDER BY created_at DESC
      LIMIT ? OFFSET ?
    `).all(limit, offset) as any[];
    
    const totalResult = this.db.query(`SELECT COUNT(*) as count FROM users`).get() as any;
    const total = totalResult.count;
    
    return {
      users: users.map(this.mapToUser),
      total,
    };
  }
  
  findById(id: number): User | null {
    const result = this.db.query(`
      SELECT id, username, email, password_hash, role, is_active, created_at, updated_at
      FROM users WHERE id = ?
    `).get(id) as any;
    
    return result ? this.mapToUser(result) : null;
  }
  
  findByEmail(email: string): User | null {
    const result = this.db.query(`
      SELECT id, username, email, password_hash, role, is_active, created_at, updated_at
      FROM users WHERE email = ?
    `).get(email) as any;
    
    return result ? this.mapToUser(result) : null;
  }
  
  findByUsername(username: string): User | null {
    const result = this.db.query(`
      SELECT id, username, email, password_hash, role, is_active, created_at, updated_at
      FROM users WHERE username = ?
    `).get(username) as any;
    
    return result ? this.mapToUser(result) : null;
  }
  
  create(data: CreateUserDTO): User {
    const passwordHash = Bun.password.hashSync(data.password, {
      algorithm: "argon2id",
    });
    
    const result = this.db.query(`
      INSERT INTO users (username, email, password_hash, role)
      VALUES (?, ?, ?, ?)
    `).run(data.username, data.email, passwordHash, data.role || "USER");
    
    return this.findById(result.lastInsertRowid as number)!;
  }
  
  update(id: number, data: UpdateUserDTO): User | null {
    const user = this.findById(id);
    if (!user) return null;
    
    const updates: string[] = [];
    const values: any[] = [];
    
    if (data.username !== undefined) {
      updates.push("username = ?");
      values.push(data.username);
    }
    if (data.email !== undefined) {
      updates.push("email = ?");
      values.push(data.email);
    }
    if (data.password !== undefined) {
      updates.push("password_hash = ?");
      values.push(Bun.password.hashSync(data.password, { algorithm: "argon2id" }));
    }
    if (data.role !== undefined) {
      updates.push("role = ?");
      values.push(data.role);
    }
    if (data.isActive !== undefined) {
      updates.push("is_active = ?");
      values.push(data.isActive ? 1 : 0);
    }
    
    if (updates.length === 0) return user;
    
    updates.push("updated_at = datetime('now')");
    values.push(id);
    
    this.db.query(`
      UPDATE users SET ${updates.join(", ")} WHERE id = ?
    `).run(...values);
    
    return this.findById(id);
  }
  
  delete(id: number): boolean {
    const result = this.db.query(`DELETE FROM users WHERE id = ?`).run(id);
    return result.changes > 0;
  }
  
  softDelete(id: number): boolean {
    const result = this.db.query(`
      UPDATE users SET is_active = 0, updated_at = datetime('now') WHERE id = ?
    `).run(id);
    return result.changes > 0;
  }
  
  count(): number {
    const result = this.db.query(`SELECT COUNT(*) as count FROM users`).get() as any;
    return result.count;
  }
  
  search(query: string, page: number = 1, limit: number = 10): { users: User[]; total: number } {
    const offset = (page - 1) * limit;
    const searchPattern = `%${query}%`;
    
    const users = this.db.query(`
      SELECT id, username, email, password_hash, role, is_active, created_at, updated_at
      FROM users
      WHERE username LIKE ? OR email LIKE ?
      ORDER BY created_at DESC
      LIMIT ? OFFSET ?
    `).all(searchPattern, searchPattern, limit, offset) as any[];
    
    const countResult = this.db.query(`
      SELECT COUNT(*) as count FROM users WHERE username LIKE ? OR email LIKE ?
    `).get(searchPattern, searchPattern) as any;
    
    return {
      users: users.map(this.mapToUser),
      total: countResult.count,
    };
  }
  
  private mapToUser(row: any): User {
    return {
      id: row.id,
      username: row.username,
      email: row.email,
      passwordHash: row.password_hash,
      role: row.role,
      isActive: row.is_active === 1,
      createdAt: row.created_at,
      updatedAt: row.updated_at,
    };
  }
}

// ==================== 业务逻辑层 ====================

class UserService {
  private repository: UserRepository;
  
  constructor(repository: UserRepository) {
    this.repository = repository;
  }
  
  async getAll(page: number, limit: number) {
    const result = this.repository.findAll(page, limit);
    return {
      data: result.users.map(this.hidePassword),
      pagination: {
        page,
        limit,
        total: result.total,
        totalPages: Math.ceil(result.total / limit),
      },
    };
  }
  
  async getById(id: number) {
    const user = this.repository.findById(id);
    if (!user) {
      throw new NotFoundError(`User with id ${id} not found`);
    }
    return this.hidePassword(user);
  }
  
  async getByEmail(email: string) {
    const user = this.repository.findByEmail(email);
    if (!user) {
      throw new NotFoundError(`User with email ${email} not found`);
    }
    return this.hidePassword(user);
  }
  
  async create(data: CreateUserDTO) {
    // 验证唯一性
    if (this.repository.findByUsername(data.username)) {
      throw new ValidationError("Username already exists");
    }
    if (this.repository.findByEmail(data.email)) {
      throw new ValidationError("Email already exists");
    }
    
    // 验证密码强度
    if (data.password.length < 8) {
      throw new ValidationError("Password must be at least 8 characters");
    }
    
    const user = this.repository.create(data);
    return this.hidePassword(user);
  }
  
  async update(id: number, data: UpdateUserDTO) {
    const user = this.repository.findById(id);
    if (!user) {
      throw new NotFoundError(`User with id ${id} not found`);
    }
    
    // 验证唯一性
    if (data.username && data.username !== user.username) {
      if (this.repository.findByUsername(data.username)) {
        throw new ValidationError("Username already exists");
      }
    }
    if (data.email && data.email !== user.email) {
      if (this.repository.findByEmail(data.email)) {
        throw new ValidationError("Email already exists");
      }
    }
    
    const updated = this.repository.update(id, data);
    return this.hidePassword(updated!);
  }
  
  async delete(id: number) {
    const user = this.repository.findById(id);
    if (!user) {
      throw new NotFoundError(`User with id ${id} not found`);
    }
    
    this.repository.delete(id);
    return { deleted: true };
  }
  
  async search(query: string, page: number, limit: number) {
    const result = this.repository.search(query, page, limit);
    return {
      data: result.users.map(this.hidePassword),
      pagination: {
        page,
        limit,
        total: result.total,
        totalPages: Math.ceil(result.total / limit),
      },
    };
  }
  
  private hidePassword(user: User): Omit<User, "passwordHash"> {
    const { passwordHash, ...userWithoutPassword } = user;
    return userWithoutPassword;
  }
}

// ==================== 错误类 ====================

class AppError extends Error {
  constructor(
    message: string,
    public statusCode: number = 500,
    public code: string = "INTERNAL_ERROR"
  ) {
    super(message);
    this.name = "AppError";
  }
}

class NotFoundError extends AppError {
  constructor(message: string) {
    super(message, 404, "NOT_FOUND");
    this.name = "NotFoundError";
  }
}

class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 400, "VALIDATION_ERROR");
    this.name = "ValidationError";
  }
}

class UnauthorizedError extends AppError {
  constructor(message: string = "Unauthorized") {
    super(message, 401, "UNAUTHORIZED");
    this.name = "UnauthorizedError";
  }
}

// ==================== Hono 应用 ====================

const app = new Hono();

// 中间件
app.use("*", cors());
app.use("*", logger());

// 错误处理中间件
app.use("*", async (c, next) => {
  try {
    await next();
  } catch (error) {
    if (error instanceof AppError) {
      return c.json({
        success: false,
        error: {
          code: error.code,
          message: error.message,
        },
      }, error.statusCode);
    }
    
    console.error("Unhandled error:", error);
    return c.json({
      success: false,
      error: {
        code: "INTERNAL_ERROR",
        message: "An unexpected error occurred",
      },
    }, 500);
  }
});

// 初始化服务
const userRepository = new UserRepository(db);
const userService = new UserService(userRepository);

// ==================== 路由定义 ====================

// Schema 定义
const createUserSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/),
  email: z.string().email(),
  password: z.string().min(8).max(100),
  role: z.enum(["ADMIN", "USER", "MODERATOR"]).optional(),
});

const updateUserSchema = z.object({
  username: z.string().min(3).max(30).regex(/^[a-zA-Z0-9_]+$/).optional(),
  email: z.string().email().optional(),
  password: z.string().min(8).max(100).optional(),
  role: z.enum(["ADMIN", "USER", "MODERATOR"]).optional(),
  isActive: z.boolean().optional(),
});

const paginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(10),
});

// 健康检查
app.get("/health", (c) => {
  return c.json({
    status: "healthy",
    timestamp: new Date().toISOString(),
    database: "connected",
  });
});

// 统计信息
app.get("/stats", (c) => {
  const totalUsers = userService.count ? (userService as any).repository.count() : 0;
  return c.json({
    totalUsers,
  });
});

// 获取所有用户（分页）
app.get(
  "/api/users",
  zValidator("query", paginationSchema),
  async (c) => {
    const { page, limit } = c.req.valid("query");
    const result = await userService.getAll(page, limit);
    return c.json({
      success: true,
      ...result,
    });
  }
);

// 搜索用户
app.get(
  "/api/users/search",
  zValidator("query", z.object({
    q: z.string().min(1),
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().min(1).max(100).default(10),
  })),
  async (c) => {
    const { q, page, limit } = c.req.valid("query");
    const result = await userService.search(q, page, limit);
    return c.json({
      success: true,
      ...result,
    });
  }
);

// 获取单个用户
app.get("/api/users/:id", async (c) => {
  const id = parseInt(c.req.param("id"));
  if (isNaN(id)) {
    throw new ValidationError("Invalid user ID");
  }
  
  const user = await userService.getById(id);
  return c.json({
    success: true,
    data: user,
  });
});

// 创建用户
app.post(
  "/api/users",
  zValidator("json", createUserSchema),
  async (c) => {
    const data = c.req.valid("json");
    const user = await userService.create(data);
    
    return c.json({
      success: true,
      data: user,
    }, 201);
  }
);

// 更新用户
app.put(
  "/api/users/:id",
  zValidator("json", updateUserSchema),
  async (c) => {
    const id = parseInt(c.req.param("id"));
    if (isNaN(id)) {
      throw new ValidationError("Invalid user ID");
    }
    
    const data = c.req.valid("json");
    const user = await userService.update(id, data);
    
    return c.json({
      success: true,
      data: user,
    });
  }
);

// 部分更新用户
app.patch(
  "/api/users/:id",
  zValidator("json", updateUserSchema.partial()),
  async (c) => {
    const id = parseInt(c.req.param("id"));
    if (isNaN(id)) {
      throw new ValidationError("Invalid user ID");
    }
    
    const data = c.req.valid("json");
    const user = await userService.update(id, data);
    
    return c.json({
      success: true,
      data: user,
    });
  }
);

// 删除用户
app.delete("/api/users/:id", async (c) => {
  const id = parseInt(c.req.param("id"));
  if (isNaN(id)) {
    throw new ValidationError("Invalid user ID");
  }
  
  await userService.delete(id);
  
  return c.body(null, 204);
});

// 批量创建用户（事务示例）
app.post(
  "/api/users/batch",
  zValidator("json", z.object({
    users: z.array(createUserSchema).min(1).max(100),
  })),
  async (c) => {
    const { users } = c.req.valid("json");
    
    // 使用事务批量插入
    const transaction = db.transaction(() => {
      const created: any[] = [];
      for (const userData of users) {
        const user = userService.create(userData);
        created.push(user);
      }
      return created;
    });
    
    const created = transaction();
    
    return c.json({
      success: true,
      data: created,
      count: created.length,
    }, 201);
  }
);

// ==================== 启动服务器 ====================

console.log("Starting server...");

export default {
  port: 3000,
  fetch: app.fetch,
};
```

---

## 中间件系统详解

### 中间件执行流程

Bun 的中间件系统基于洋葱模型，允许多个中间件按顺序处理请求和响应。

```typescript
import { Hono } from "hono";
import { cors } from "hono/cors";
import { logger } from "hono/logger";
import { compress } from "hono/compress";
import { etag } from "hono/etag";
import { secureHeaders } from "hono/secure-headers";
import { rateLimit } from "hono/rate-limit";

const app = new Hono();

// ==================== 内置中间件 ====================

// CORS 配置
app.use("*", cors({
  origin: (origin) => {
    // 验证 origin
    const allowedOrigins = [
      "https://example.com",
      "https://app.example.com",
      "http://localhost:3000",
    ];
    return allowedOrigins.includes(origin) ? origin : false;
  },
  credentials: true,
  allowMethods: ["GET", "POST", "PUT", "DELETE", "PATCH", "OPTIONS"],
  allowHeaders: ["Content-Type", "Authorization", "X-Request-ID"],
  exposeHeaders: ["X-Request-ID"],
  maxAge: 86400,  // 预检请求缓存时间
}));

// 响应压缩
app.use("*", compress({
  encoding: "gzip",  // gzip 或 deflate
}));

// ETag 生成
app.use("*", etag({
  weak: true,  // 弱 ETag（前面加 W/）
}));

// 安全响应头
app.use("*", secureHeaders({
  // Content Security Policy
  contentSecurityPolicy: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "data:", "https:"],
    connectSrc: ["'self'"],
    fontSrc: ["'self'"],
    objectSrc: ["'none'"],
    mediaSrc: ["'self'"],
    frameSrc: ["'none'"],
  },
  // X-Frame-Options
  XFrameOptions: "SAMEORIGIN",
  // X-Content-Type-Options
  XContentTypeOptions: "nosniff",
  // X-XSS-Protection
  XXSSProtection: "1; mode=block",
  // Strict-Transport-Security
  StrictTransportSecurity: "max-age=31536000; includeSubDomains",
  // Referrer-Policy
  ReferrerPolicy: "strict-origin-when-cross-origin",
}));

// 请求日志
app.use("*", logger({
  format: (c, next) => {
    const start = Date.now();
    return next().then(() => {
      const elapsed = Date.now() - start;
      const log = [
        c.req.method,
        c.req.url,
        c.res.status,
        `${elapsed}ms`,
      ].join(" ");
      console.log(log);
    });
  },
}));

// 限流
app.use("/api/*", rateLimit({
  windowMs: 60 * 1000,  // 1 分钟窗口
  limit: 100,           // 每次窗口 100 请求
  standardHeaders: true, // 返回 RateLimit-* 头
  legacyHeaders: false,
  generator: (c) => {
    // 根据 IP 生成限流 key
    return c.req.header("x-forwarded-for")?.split(",")[0] || 
           c.req.header("cf-connecting-ip") ||
           "anonymous";
  },
  keyGenerator: (c) => {
    // 自定义 key 生成
    const ip = c.req.header("x-forwarded-for")?.split(",")[0] || "ip";
    const userAgent = c.req.header("user-agent") || "ua";
    return `${ip}:${userAgent.slice(0, 20)}`;
  },
  handler: (c) => {
    return c.json({
      error: "Too many requests",
      retryAfter: 60,
    }, 429);
  },
}));

// ==================== 自定义中间件 ====================

// 请求 ID 中间件
app.use("*", async (c, next) => {
  const requestId = c.req.header("X-Request-ID") || crypto.randomUUID();
  c.set("requestId", requestId);
  c.set("startTime", Date.now());
  
  await next();
  
  // 添加响应头
  c.res.headers.set("X-Request-ID", requestId);
  c.res.headers.set("X-Response-Time", `${Date.now() - c.get("startTime")}ms`);
});

// 认证中间件
app.use("/api/protected/*", async (c, next) => {
  const authHeader = c.req.header("Authorization");
  
  if (!authHeader?.startsWith("Bearer ")) {
    return c.json({
      error: "Unauthorized",
      message: "Missing or invalid authorization header",
    }, 401);
  }
  
  const token = authHeader.substring(7);
  
  try {
    const payload = await verifyJWT(token);
    c.set("user", payload);
    await next();
  } catch (error) {
    return c.json({
      error: "Unauthorized",
      message: "Invalid or expired token",
    }, 401);
  }
});

// 角色检查中间件工厂
function requireRole(...roles: string[]) {
  return async (c: any, next: () => Promise<void>) => {
    const user = c.get("user");
    
    if (!user) {
      return c.json({ error: "Unauthorized" }, 401);
    }
    
    if (!roles.includes(user.role)) {
      return c.json({
        error: "Forbidden",
        message: `Required role: ${roles.join(" or ")}`,
      }, 403);
    }
    
    await next();
  };
}

// 管理员专属路由
app.delete(
  "/api/admin/users/:id",
  requireRole("ADMIN"),
  async (c) => {
    // 删除用户逻辑
    return c.json({ deleted: true });
  }
);

// 请求验证中间件
app.use("/api/*", async (c, next) => {
  const contentType = c.req.header("Content-Type");
  
  if (["POST", "PUT", "PATCH"].includes(c.req.method)) {
    if (!contentType?.includes("application/json")) {
      return c.json({
        error: "Bad Request",
        message: "Content-Type must be application/json",
      }, 415);
    }
    
    // 验证 JSON 格式
    try {
      const body = await c.req.json();
      c.set("parsedBody", body);
    } catch {
      return c.json({
        error: "Bad Request",
        message: "Invalid JSON body",
      }, 400);
    }
  }
  
  await next();
});

// 缓存控制中间件
app.use("/api/public/*", async (c, next) => {
  await next();
  
  // 公共数据缓存 5 分钟
  c.res.headers.set("Cache-Control", "public, max-age=300");
  c.res.headers.set("Vary", "Accept-Encoding");
});

app.use("/api/private/*", async (c, next) => {
  await next();
  
  // 私有数据不缓存
  c.res.headers.set("Cache-Control", "private, no-store");
});

// 服务端渲染页面缓存
app.use("/static/*", async (c, next) => {
  await next();
  
  // 静态资源缓存 1 小时
  c.res.headers.set("Cache-Control", "public, max-age=3600, immutable");
  c.res.headers.set("ETag", `W/"${crypto.randomUUID()}"`);
});

// 错误恢复中间件
app.use("*", async (c, next) => {
  try {
    await next();
  } catch (error) {
    console.error("Error in request handler:", error);
    
    return c.json({
      success: false,
      error: {
        code: "INTERNAL_ERROR",
        message: error instanceof Error ? error.message : "Unknown error",
        ...(process.env.NODE_ENV === "development" && {
          stack: error instanceof Error ? error.stack : undefined,
        }),
      },
    }, 500);
  }
});

// 请求超时中间件
app.use("/api/*", async (c, next) => {
  const timeout = parseInt(c.req.header("X-Timeout") || "30000");
  
  const timeoutPromise = new Promise<never>((_, reject) => {
    setTimeout(() => reject(new Error("Request timeout")), timeout);
  });
  
  try {
    await Promise.race([
      next(),
      timeoutPromise,
    ]);
  } catch (error) {
    if (error instanceof Error && error.message === "Request timeout") {
      return c.json({
        error: "Gateway Timeout",
        message: "Request processing exceeded timeout limit",
      }, 504);
    }
    throw error;
  }
});
```

---

## 数据库集成详解

### 多数据库支持

Bun 支持多种数据库集成，包括 SQLite、PostgreSQL、MySQL 等关系型数据库，以及 Redis 等键值存储。

#### SQLite 与 Prisma 集成

```typescript
// src/db/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}

model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  role      String   @default("USER")
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]
  comments  Comment[]
}

model Post {
  id        String    @id @default(uuid())
  title     String
  content   String
  published Boolean   @default(false)
  authorId  String
  author    User      @relation(fields: [authorId], references: [id])
  comments  Comment[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  postId    String
  post      Post     @relation(fields: [postId], references: [id])
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

// src/db/client.ts
import { PrismaClient } from "@prisma/client";

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: process.env.NODE_ENV === "development" 
      ? ["query", "error", "warn"]
      : ["error"],
  });

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

// src/db/seed.ts
async function seed() {
  console.log("Seeding database...");
  
  // 创建用户
  const alice = await prisma.user.create({
    data: {
      username: "alice",
      email: "alice@example.com",
      password: await Bun.password.hash("password123"),
      role: "ADMIN",
      posts: {
        create: [
          {
            title: "Getting Started with Bun",
            content: "Bun is a fast JavaScript runtime...",
            published: true,
          },
          {
            title: "Building APIs with Hono",
            content: "Hono is a lightweight web framework...",
            published: true,
          },
        ],
      },
    },
    include: { posts: true },
  });
  
  const bob = await prisma.user.create({
    data: {
      username: "bob",
      email: "bob@example.com",
      password: await Bun.password.hash("password123"),
      role: "USER",
    },
  });
  
  // 创建评论
  const post = await prisma.post.findFirst({
    where: { author: { username: "alice" } },
  });
  
  if (post) {
    await prisma.comment.create({
      data: {
        content: "Great article!",
        postId: post.id,
        authorId: bob.id,
      },
    });
  }
  
  console.log("Seeding completed!");
  console.log(`Created users: ${alice.username}, ${bob.username}`);
}

seed()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

#### Redis 缓存集成

```typescript
import { Redis } from "ioredis";

// src/cache/redis.ts
class RedisCache {
  private client: Redis;
  private defaultTTL = 3600; // 1 hour
  
  constructor() {
    this.client = new Redis({
      host: process.env.REDIS_HOST || "localhost",
      port: parseInt(process.env.REDIS_PORT || "6379"),
      password: process.env.REDIS_PASSWORD,
      maxRetriesPerRequest: 3,
      retryStrategy: (times) => {
        if (times > 3) return null;
        return Math.min(times * 100, 3000);
      },
    });
    
    this.client.on("error", (err) => {
      console.error("Redis connection error:", err);
    });
    
    this.client.on("connect", () => {
      console.log("Redis connected");
    });
  }
  
  async get<T>(key: string): Promise<T | null> {
    const data = await this.client.get(key);
    if (!data) return null;
    
    try {
      return JSON.parse(data) as T;
    } catch {
      return data as unknown as T;
    }
  }
  
  async set(key: string, value: any, ttl?: number): Promise<void> {
    const data = typeof value === "string" ? value : JSON.stringify(value);
    await this.client.setex(key, ttl || this.defaultTTL, data);
  }
  
  async delete(key: string): Promise<void> {
    await this.client.del(key);
  }
  
  async exists(key: string): Promise<boolean> {
    return (await this.client.exists(key)) === 1;
  }
  
  async getOrSet<T>(
    key: string,
    factory: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) {
      return cached;
    }
    
    const value = await factory();
    await this.set(key, value, ttl);
    return value;
  }
  
  async invalidatePattern(pattern: string): Promise<void> {
    const keys = await this.client.keys(pattern);
    if (keys.length > 0) {
      await this.client.del(...keys);
    }
  }
  
  // Hash 操作
  async hGet<T>(key: string, field: string): Promise<T | null> {
    const data = await this.client.hget(key, field);
    return data ? JSON.parse(data) : null;
  }
  
  async hSet(key: string, field: string, value: any): Promise<void> {
    await this.client.hset(key, field, JSON.stringify(value));
  }
  
  async hGetAll<T>(key: string): Promise<Record<string, T>> {
    const data = await this.client.hgetall(key);
    const result: Record<string, T> = {};
    
    for (const [field, value] of Object.entries(data)) {
      try {
        result[field] = JSON.parse(value);
      } catch {
        result[field] = value as unknown as T;
      }
    }
    
    return result;
  }
  
  // 列表操作
  async lPush<T>(key: string, ...values: T[]): Promise<number> {
    return this.client.lpush(key, ...values.map(v => JSON.stringify(v)));
  }
  
  async lRange<T>(key: string, start: number, stop: number): Promise<T[]> {
    const data = await this.client.lrange(key, start, stop);
    return data.map(v => JSON.parse(v));
  }
  
  // 发布/订阅
  publish(channel: string, message: any): Promise<number> {
    const data = typeof message === "string" ? message : JSON.stringify(message);
    return this.client.publish(channel, data);
  }
  
  subscribe(channel: string, callback: (message: any) => void): void {
    const subscriber = this.client.duplicate();
    subscriber.subscribe(channel);
    subscriber.on("message", (ch, message) => {
      if (ch === channel) {
        try {
          callback(JSON.parse(message));
        } catch {
          callback(message);
        }
      }
    });
  }
}

export const cache = new RedisCache();

// 使用示例
async function cacheExample() {
  const userId = "123";
  
  // 缓存用户数据
  const userKey = `user:${userId}`;
  const cachedUser = await cache.get<{ id: string; name: string }>(userKey);
  
  if (cachedUser) {
    console.log("Cache hit:", cachedUser);
  } else {
    console.log("Cache miss, fetching from database...");
    const user = await prisma.user.findUnique({ where: { id: userId } });
    if (user) {
      await cache.set(userKey, user, 300); // 缓存 5 分钟
    }
  }
  
  // 使用 getOrSet 简化
  const user = await cache.getOrSet(
    `user:${userId}`,
    () => prisma.user.findUnique({ where: { id: userId } }),
    300
  );
  
  // 发布消息
  await cache.publish("user:created", { id: userId, email: "new@example.com" });
}
```

---

## 认证授权方案详解

### JWT 与 Session 双模式认证

```typescript
import { Hono } from "hono";
import { SignJWT, jwtVerify } from "jose";
import { createHash, randomBytes } from "crypto";

// ==================== 配置 ====================

const JWT_SECRET = new TextEncoder().encode(
  process.env.JWT_SECRET || "your-secret-key-min-32-chars-long!!"
);

const REFRESH_SECRET = new TextEncoder().encode(
  process.env.REFRESH_SECRET || "refresh-secret-key-min-32-chars-long!!"
);

// ==================== Token 类型 ====================

interface AccessTokenPayload {
  sub: string;
  email: string;
  role: string;
  type: "access";
  iat: number;
  exp: number;
}

interface RefreshTokenPayload {
  sub: string;
  type: "refresh";
  iat: number;
  exp: number;
}

interface TokenPair {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

// ==================== Token 服务 ====================

class TokenService {
  private accessTokenExpiry = "15m";
  private refreshTokenExpiry = "7d";
  
  async generateTokens(user: {
    id: string;
    email: string;
    role: string;
  }): Promise<TokenPair> {
    const now = Math.floor(Date.now() / 1000);
    
    // 生成访问令牌
    const accessToken = await new SignJWT({
      sub: user.id,
      email: user.email,
      role: user.role,
      type: "access",
    })
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt(now)
      .setExpirationTime(`${this.accessTokenExpiry}`)
      .sign(JWT_SECRET);
    
    // 生成刷新令牌
    const refreshToken = await new SignJWT({
      sub: user.id,
      type: "refresh",
    })
      .setProtectedHeader({ alg: "HS256" })
      .setIssuedAt(now)
      .setExpirationTime(`${this.refreshTokenExpiry}`)
      .sign(REFRESH_SECRET);
    
    // 计算过期时间
    const expiresIn = now + 15 * 60; // 15 分钟
    
    return {
      accessToken,
      refreshToken,
      expiresIn,
    };
  }
  
  async verifyAccessToken(token: string): Promise<AccessTokenPayload | null> {
    try {
      const { payload } = await jwtVerify(token, JWT_SECRET);
      return payload as unknown as AccessTokenPayload;
    } catch {
      return null;
    }
  }
  
  async verifyRefreshToken(token: string): Promise<RefreshTokenPayload | null> {
    try {
      const { payload } = await jwtVerify(token, REFRESH_SECRET);
      return payload as unknown as RefreshTokenPayload;
    } catch {
      return null;
    }
  }
  
  async refreshTokens(refreshToken: string): Promise<TokenPair | null> {
    const payload = await this.verifyRefreshToken(refreshToken);
    if (!payload) return null;
    
    // 从数据库获取用户信息
    const user = await prisma.user.findUnique({
      where: { id: payload.sub },
    });
    
    if (!user || !user.isActive) return null;
    
    return this.generateTokens({
      id: user.id,
      email: user.email,
      role: user.role,
    });
  }
}

// ==================== Session 存储 ====================

class SessionStore {
  private store: Map<string, {
    userId: string;
    data: Record<string, any>;
    expiresAt: number;
  }> = new Map();
  
  private readonly sessionPrefix = "session:";
  private readonly defaultTTL = 24 * 60 * 60; // 24 小时
  
  generate(userId: string, data: Record<string, any> = {}): string {
    const sessionId = randomBytes(32).toString("hex");
    const expiresAt = Date.now() + this.defaultTTL * 1000;
    
    this.store.set(sessionId, {
      userId,
      data,
      expiresAt,
    });
    
    return sessionId;
  }
  
  get(sessionId: string): { userId: string; data: Record<string, any> } | null {
    const session = this.store.get(sessionId);
    
    if (!session) return null;
    if (session.expiresAt < Date.now()) {
      this.store.delete(sessionId);
      return null;
    }
    
    return {
      userId: session.userId,
      data: session.data,
    };
  }
  
  update(sessionId: string, data: Record<string, any>): boolean {
    const session = this.store.get(sessionId);
    if (!session) return false;
    
    session.data = { ...session.data, ...data };
    return true;
  }
  
  delete(sessionId: string): boolean {
    return this.store.delete(sessionId);
  }
  
  extend(sessionId: string, ttl?: number): boolean {
    const session = this.store.get(sessionId);
    if (!session) return false;
    
    session.expiresAt = Date.now() + (ttl || this.defaultTTL) * 1000;
    return true;
  }
  
  cleanExpired(): number {
    const now = Date.now();
    let count = 0;
    
    for (const [id, session] of this.store) {
      if (session.expiresAt < now) {
        this.store.delete(id);
        count++;
      }
    }
    
    return count;
  }
}

// ==================== 认证中间件工厂 ====================

const tokenService = new TokenService();
const sessionStore = new SessionStore();

function createAuthMiddleware(options: {
  mode: "jwt" | "session" | "optional";
  required?: boolean;
} = { mode: "jwt", required: true }) {
  return async (c: any, next: () => Promise<void>) => {
    const authHeader = c.req.header("Authorization");
    const sessionId = c.req.cookie("session_id");
    
    let user: { id: string; email?: string; role: string } | null = null;
    
    if (options.mode === "jwt") {
      if (!authHeader?.startsWith("Bearer ")) {
        if (options.required) {
          return c.json({ error: "Unauthorized" }, 401);
        }
        return next();
      }
      
      const token = authHeader.substring(7);
      const payload = await tokenService.verifyAccessToken(token);
      
      if (!payload) {
        if (options.required) {
          return c.json({ error: "Token expired or invalid" }, 401);
        }
        return next();
      }
      
      user = {
        id: payload.sub,
        email: payload.email,
        role: payload.role,
      };
      
    } else if (options.mode === "session") {
      if (!sessionId) {
        if (options.required) {
          return c.json({ error: "Unauthorized" }, 401);
        }
        return next();
      }
      
      const session = sessionStore.get(sessionId);
      
      if (!session) {
        if (options.required) {
          return c.json({ error: "Session expired" }, 401);
        }
        return next();
      }
      
      const dbUser = await prisma.user.findUnique({
        where: { id: session.userId },
      });
      
      if (!dbUser || !dbUser.isActive) {
        sessionStore.delete(sessionId);
        if (options.required) {
          return c.json({ error: "Unauthorized" }, 401);
        }
        return next();
      }
      
      user = {
        id: dbUser.id,
        email: dbUser.email,
        role: dbUser.role,
      };
      
    } else if (options.mode === "optional") {
      // JWT 可选模式
      if (authHeader?.startsWith("Bearer ")) {
        const token = authHeader.substring(7);
        const payload = await tokenService.verifyAccessToken(token);
        if (payload) {
          user = {
            id: payload.sub,
            email: payload.email,
            role: payload.role,
          };
        }
      }
    }
    
    c.set("auth", user);
    c.set("user", user);
    
    await next();
  };
}

// ==================== 权限检查中间件 ====================

function requirePermission(...permissions: string[]) {
  return async (c: any, next: () => Promise<void>) => {
    const user = c.get("user");
    
    if (!user) {
      return c.json({ error: "Unauthorized" }, 401);
    }
    
    // 管理员拥有所有权限
    if (user.role === "ADMIN") {
      return next();
    }
    
    // TODO: 从数据库或缓存获取用户权限
    const userPermissions = await getUserPermissions(user.id);
    
    const hasPermission = permissions.every(p => 
      userPermissions.includes(p)
    );
    
    if (!hasPermission) {
      return c.json({
        error: "Forbidden",
        message: `Missing required permission: ${permissions.join(", ")}`,
      }, 403);
    }
    
    await next();
  };
}

async function getUserPermissions(userId: string): Promise<string[]> {
  // 从数据库获取用户权限
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: { roles: true },
  });
  
  if (!user) return [];
  
  // 简化：直接返回角色作为权限
  return user.roles.map(r => r.name);
}

// ==================== 认证路由 ====================

const auth = new Hono();

// 登录
auth.post("/login", async (c) => {
  const { email, password } = await c.req.json();
  
  const user = await prisma.user.findUnique({
    where: { email },
  });
  
  if (!user || !user.isActive) {
    return c.json({ error: "Invalid credentials" }, 401);
  }
  
  const isValid = await Bun.password.verify(password, user.passwordHash);
  if (!isValid) {
    return c.json({ error: "Invalid credentials" }, 401);
  }
  
  const tokens = await tokenService.generateTokens({
    id: user.id,
    email: user.email,
    role: user.role,
  });
  
  // 设置 refresh token cookie
  c.header("Set-Cookie", `refresh_token=${tokens.refreshToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=${7 * 24 * 60 * 60}`);
  
  return c.json({
    accessToken: tokens.accessToken,
    expiresIn: tokens.expiresIn,
  });
});

// 刷新令牌
auth.post("/refresh", async (c) => {
  const refreshToken = c.req.cookie("refresh_token");
  
  if (!refreshToken) {
    return c.json({ error: "Refresh token required" }, 401);
  }
  
  const tokens = await tokenService.refreshTokens(refreshToken);
  
  if (!tokens) {
    return c.json({ error: "Invalid refresh token" }, 401);
  }
  
  c.header("Set-Cookie", `refresh_token=${tokens.refreshToken}; HttpOnly; Secure; SameSite=Strict; Max-Age=${7 * 24 * 60 * 60}`);
  
  return c.json({
    accessToken: tokens.accessToken,
    expiresIn: tokens.expiresIn,
  });
});

// 登出
auth.post("/logout", async (c) => {
  const sessionId = c.req.cookie("session_id");
  
  if (sessionId) {
    sessionStore.delete(sessionId);
  }
  
  c.header("Set-Cookie", `refresh_token=; HttpOnly; Secure; SameSite=Strict; Max-Age=0`);
  
  return c.json({ success: true });
});

// 注册
auth.post("/register", async (c) => {
  const { username, email, password } = await c.req.json();
  
  // 验证唯一性
  const existing = await prisma.user.findFirst({
    where: {
      OR: [{ email }, { username }],
    },
  });
  
  if (existing) {
    return c.json({
      error: "User already exists",
      field: existing.email === email ? "email" : "username",
    }, 409);
  }
  
  // 创建用户
  const user = await prisma.user.create({
    data: {
      username,
      email,
      passwordHash: await Bun.password.hash(password, { algorithm: "argon2id" }),
      role: "USER",
    },
  });
  
  const tokens = await tokenService.generateTokens({
    id: user.id,
    email: user.email,
    role: user.role,
  });
  
  return c.json({
    user: {
      id: user.id,
      username: user.username,
      email: user.email,
    },
    ...tokens,
  }, 201);
});

// 密码重置请求
auth.post("/password/reset", async (c) => {
  const { email } = await c.req.json();
  
  const user = await prisma.user.findUnique({
    where: { email },
  });
  
  if (user) {
    // 生成重置令牌并发送邮件（实际应用中）
    const resetToken = randomBytes(32).toString("hex");
    // await sendPasswordResetEmail(email, resetToken);
  }
  
  // 即使用户不存在也返回成功，防止用户枚举攻击
  return c.json({
    message: "If the email exists, a reset link has been sent",
  });
});

// 导出中间件
export { createAuthMiddleware, requirePermission };
```

---

## 部署配置（Docker）

### 生产环境 Dockerfile 与配置

```dockerfile
# Dockerfile
FROM oven/bun:1.2-alpine AS base

# 安装依赖阶段
FROM base AS deps

WORKDIR /app
COPY package.json bun.lockb* ./
RUN bun install --frozen-lockfile

# 构建阶段
FROM base AS builder

WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# 构建 TypeScript
RUN bun build src/index.ts \
    --target=bun \
    --outfile=dist/index.js \
    --minify \
    --sourcemap

# 运行阶段
FROM base AS runner

WORKDIR /app

# 创建非 root 用户
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 bun

# 复制构建产物
COPY --from=builder /app/dist ./dist
COPY --from=deps /app/node_modules ./node_modules

# 复制配置文件
COPY --chown=bun:nodejs package.json ./

# 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000
ENV HOST=0.0.0.0

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:${PORT}/health || exit 1

# 切换到非 root 用户
USER bun

# 暴露端口
EXPOSE 3000

# 启动命令
CMD ["bun", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: "3.9"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    image: myapp:latest
    container_name: myapp
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - DATABASE_URL=postgresql://user:pass@postgres:5432/mydb
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - JWT_REFRESH_SECRET=${JWT_REFRESH_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d mydb"]
      interval: 10s
      timeout: 5s
      retries: 5
    deploy:
      resources:
        limits:
          cpus: "1"
          memory: 256M

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
  redis_data:

networks:
  default:
    name: myapp-network
```

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: myapp:latest
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: PORT
              value: "3000"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                configMapKeyRef:
                  name: myapp-config
                  key: redis-url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 30
            timeoutSeconds: 3
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 3
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 性能优化详解

### 数据库优化策略

```typescript
// ==================== 连接池配置 ====================

class DatabasePool {
  private pool: any[] = [];
  private readonly maxConnections = 10;
  private readonly minConnections = 2;
  
  async acquire(): Promise<Database> {
    if (this.pool.length > 0) {
      return this.pool.pop()!;
    }
    
    return new Database(process.env.DATABASE_URL);
  }
  
  release(connection: Database): void {
    if (this.pool.length < this.maxConnections) {
      this.pool.push(connection);
    } else {
      connection.close();
    }
  }
  
  async withConnection<T>(
    callback: (conn: Database) => Promise<T>
  ): Promise<T> {
    const conn = await this.acquire();
    try {
      return await callback(conn);
    } finally {
      this.release(conn);
    }
  }
}

// ==================== 查询优化 ====================

class QueryOptimizer {
  // 批量插入优化
  async batchInsert(
    table: string,
    records: Record<string, any>[],
    batchSize: number = 100
  ): Promise<number> {
    let totalInserted = 0;
    
    for (let i = 0; i < records.length; i += batchSize) {
      const batch = records.slice(i, i + batchSize);
      
      await db.transaction(async (tx) => {
        for (const record of batch) {
          const keys = Object.keys(record);
          const values = Object.values(record);
          const placeholders = keys.map(() => "?").join(", ");
          
          await tx.query(
            `INSERT INTO ${table} (${keys.join(", ")}) VALUES (${placeholders})`,
            values
          );
          totalInserted++;
        }
      });
    }
    
    return totalInserted;
  }
  
  // 分页查询优化（游标分页）
  async cursorPaginate<T>(
    query: string,
    params: any[],
    cursor: string | null,
    limit: number
  ): Promise<{ data: T[]; nextCursor: string | null }> {
    let fullQuery = query;
    const queryParams = [...params];
    
    if (cursor) {
      fullQuery += " WHERE id < ?";
      queryParams.unshift(cursor);
    }
    
    fullQuery += ` ORDER BY id DESC LIMIT ${limit + 1}`;
    
    const results = await db.query(fullQuery, queryParams).all();
    
    if (results.length > limit) {
      const nextCursor = results[limit - 1].id;
      return {
        data: results.slice(0, limit) as T[],
        nextCursor,
      };
    }
    
    return {
      data: results as T[],
      nextCursor: null,
    };
  }
  
  // 索引建议
  async suggestIndexes(tableName: string): Promise<string[]> {
    // 分析慢查询日志或查询统计
    const slowQueries = await this.getSlowQueries();
    const suggestions: string[] = [];
    
    for (const query of slowQueries) {
      // 简单的索引建议逻辑
      const match = query.match(/WHERE\s+(\w+)\s*=/);
      if (match) {
        const column = match[1];
        suggestions.push(
          `CREATE INDEX idx_${tableName}_${column} ON ${tableName}(${column})`
        );
      }
    }
    
    return [...new Set(suggestions)];
  }
  
  // 查询缓存
  private cache = new Map<string, { data: any; expiresAt: number }>();
  
  async cachedQuery<T>(
    key: string,
    query: () => Promise<T>,
    ttl: number = 60000
  ): Promise<T> {
    const cached = this.cache.get(key);
    
    if (cached && cached.expiresAt > Date.now()) {
      return cached.data as T;
    }
    
    const data = await query();
    this.cache.set(key, {
      data,
      expiresAt: Date.now() + ttl,
    });
    
    return data;
  }
  
  // N+1 查询检测与优化
  async detectNPlusOne(
    queries: { sql: string; params: any[]; time: number }[]
  ): Promise<string[]> {
    const warnings: string[] = [];
    const queryCounts = new Map<string, number>();
    
    for (const q of queries) {
      // 简化的 N+1 检测
      const normalizedSql = q.sql.replace(/\?/g, "'?'").trim();
      const count = queryCounts.get(normalizedSql) || 0;
      queryCounts.set(normalizedSql, count + 1);
    }
    
    for (const [sql, count] of queryCounts) {
      if (count > 10) {
        warnings.push(
          `Possible N+1 query detected: "${sql}" executed ${count} times`
        );
      }
    }
    
    return warnings;
  }
}

// ==================== 响应缓存优化 ====================

class ResponseCache {
  private store = new Map<string, {
    data: any;
    headers: Record<string, string>;
    expiresAt: number;
    etag: string;
  }>();
  
  get(key: string): { data: any; headers: Record<string, string> } | null {
    const cached = this.store.get(key);
    
    if (!cached) return null;
    if (cached.expiresAt < Date.now()) {
      this.store.delete(key);
      return null;
    }
    
    return {
      data: cached.data,
      headers: {
        ...cached.headers,
        "ETag": cached.etag,
        "X-Cache": "HIT",
      },
    };
  }
  
  set(
    key: string,
    data: any,
    headers: Record<string, string> = {},
    ttl: number = 60000
  ): void {
    const etag = `"${crypto.randomUUID()}"`;
    
    this.store.set(key, {
      data,
      headers,
      expiresAt: Date.now() + ttl,
      etag,
    });
  }
  
  invalidate(pattern: string): number {
    let count = 0;
    
    for (const key of this.store.keys()) {
      if (key.includes(pattern)) {
        this.store.delete(key);
        count++;
      }
    }
    
    return count;
  }
  
  clear(): void {
    this.store.clear();
  }
}
```

### HTTP 性能优化

```typescript
// ==================== 响应压缩 ====================

import { compress } from "hono/compress";

app.use("*", compress({
  encoding: "gzip",
  threshold: 1024,  // 小于 1KB 不压缩
}));

// ==================== 连接复用 ====================

// HTTP Keep-Alive 配置
const server = Bun.serve({
  port: 3000,
  keepAliveTimeout: 65000,  // 65 秒
  maxConnections: 10000,
});

// ==================== 响应流式处理 ====================

app.get("/api/stream", async (c) => {
  const encoder = new TextEncoder();
  
  const stream = new ReadableStream({
    async start(controller) {
      for (let i = 0; i < 100; i++) {
        const data = JSON.stringify({ index: i, timestamp: Date.now() });
        controller.enqueue(encoder.encode(data + "\n"));
        await new Promise(resolve => setTimeout(resolve, 100));
      }
      controller.close();
    },
  });
  
  return new Response(stream, {
    headers: {
      "Content-Type": "application/x-ndjson",
      "Transfer-Encoding": "chunked",
      "Cache-Control": "no-cache",
    },
  });
});

// ==================== 资源预加载 ====================

app.get("/api/preload", async (c) => {
  // 预加载用户和文章数据
  const [users, posts] = await Promise.all([
    prisma.user.findMany({ take: 10 }),
    prisma.post.findMany({ take: 10, where: { published: true } }),
  ]);
  
  return c.json({ users, posts });
});

// ==================== 数据库查询优化 ====================

// 使用 JOIN 代替多次查询
app.get("/api/users-with-posts", async (c) => {
  // ❌ N+1 查询
  // const users = await prisma.user.findMany();
  // for (const user of users) {
  //   user.posts = await prisma.post.findMany({ where: { authorId: user.id } });
  // }
  
  // ✅ 使用 include
  const users = await prisma.user.findMany({
    include: {
      posts: {
        where: { published: true },
        take: 5,
        orderBy: { createdAt: "desc" },
      },
    },
    take: 10,
  });
  
  return c.json(users);
});
```

---

## 常见陷阱与最佳实践

### 陷阱详解与解决方案

#### 陷阱 1：并发竞态条件

```typescript
// ❌ 竞态条件
app.post("/api/users/:id/follow", async (c) => {
  const userId = c.req.param("id");
  const currentUser = c.get("user");
  
  // 两个请求可能同时检查并插入
  const existing = await db.query(
    "SELECT * FROM follows WHERE follower_id = ? AND following_id = ?",
    [currentUser.id, userId]
  ).get();
  
  if (!existing) {
    await db.query(
      "INSERT INTO follows (follower_id, following_id) VALUES (?, ?)",
      [currentUser.id, userId]
    );
  }
  
  return c.json({ following: true });
});

// ✅ 使用唯一约束 + 异常处理
app.post("/api/users/:id/follow", async (c) => {
  const userId = c.req.param("id");
  const currentUser = c.get("user");
  
  try {
    await db.query(
      "INSERT INTO follows (follower_id, following_id) VALUES (?, ?)",
      [currentUser.id, userId]
    );
    return c.json({ following: true });
  } catch (error) {
    // 违反唯一约束，说明已经关注
    if (error.message.includes("UNIQUE constraint")) {
      return c.json({ following: true });
    }
    throw error;
  }
});

// ✅ 使用事务
app.post("/api/users/:id/follow", async (c) => {
  const userId = c.req.param("id");
  const currentUser = c.get("user");
  
  await db.transaction(async (tx) => {
    const existing = await tx.query(
      "SELECT 1 FROM follows WHERE follower_id = ? AND following_id = ? FOR UPDATE",
      [currentUser.id, userId]
    ).get();
    
    if (!existing) {
      await tx.query(
        "INSERT INTO follows (follower_id, following_id) VALUES (?, ?)",
        [currentUser.id, userId]
      );
    }
  });
  
  return c.json({ following: true });
});
```

#### 陷阱 2：内存泄漏

```typescript
// ❌ 全局变量累积
const allConnections: Set<WebSocket> = new Set();

server.websocket.open = (ws) => {
  allConnections.add(ws);  // 永远不清理
};

// ✅ 使用 WeakRef 或手动清理
server.websocket.close = (ws) => {
  // 清理逻辑
};

// ✅ 限制全局状态大小
const MAX_CONNECTIONS = 10000;
if (allConnections.size >= MAX_CONNECTIONS) {
  // 拒绝新连接或清理旧连接
}

// ✅ 定期清理过期数据
setInterval(() => {
  const now = Date.now();
  for (const [key, value] of cache) {
    if (value.expiresAt < now) {
      cache.delete(key);
    }
  }
}, 60000);  // 每分钟清理
```

#### 陷阱 3：错误处理不当

```typescript
// ❌ 吞掉错误
app.get("/api/data", async (c) => {
  try {
    const data = await fetchData();
    return c.json(data);
  } catch (error) {
    // 静默吞掉错误
  }
});

// ✅ 正确传播错误
app.get("/api/data", async (c) => {
  try {
    const data = await fetchData();
    return c.json(data);
  } catch (error) {
    console.error("Failed to fetch data:", error);
    throw error;  // 让错误处理中间件处理
  }
});

// ✅ 返回有意义的错误
app.get("/api/data", async (c) => {
  try {
    const data = await fetchData();
    return c.json(data);
  } catch (error) {
    if (error instanceof DatabaseError) {
      return c.json({
        error: "Database error",
        message: "Unable to fetch data at this time",
      }, 503);
    }
    
    return c.json({
      error: "Internal error",
      message: "An unexpected error occurred",
    }, 500);
  }
});
```

#### 陷阱 4：不安全的密码处理

```typescript
// ❌ 使用弱哈希
const hash = crypto.createHash("sha256").update(password).digest("hex");

// ✅ 使用 Argon2 或 bcrypt
const hash = await Bun.password.hash(password, {
  algorithm: "argon2id",
  memoryCost: 65536,
  timeCost: 3,
  threadCount: 4,
});

const isValid = await Bun.password.verify(password, hash);

// ❌ 明文密码日志
console.log(`User ${username} login with password: ${password}`);

// ✅ 安全的日志
console.log(`User ${username} login attempt`);
```

---

## 与同类框架对比

### Bun vs Node.js 生态对比

| 特性 | Bun | Node.js | 优势方 |
|------|-----|---------|--------|
| **安装速度** | <1s | 10-30s | Bun |
| **运行速度** | 快 3-5x | 基准 | Bun |
| **npm 兼容性** | 99%+ | 100% | Node.js |
| **内置工具** | 完整 | 需要安装 | Bun |
| **调试工具** | 发展中 | 成熟 | Node.js |
| **企业支持** | 发展中 | 成熟 | Node.js |
| **Worker Threads** | 支持 | 支持 | 平手 |
| **原生 ESM** | 是 | 可选 | Bun |

### Bun vs Deno 对比

| 特性 | Bun | Deno | 优势方 |
|------|-----|------|--------|
| **运行速度** | 快 2x | 基准 | Bun |
| **npm 兼容性** | 99%+ | 需适配层 | Bun |
| **权限沙箱** | 无 | 有 | Deno |
| **工具链** | 完整 | 需要安装 | Bun |
| **TypeScript** | 原生 | 原生 | 平手 |
| **企业采用** | 增长中 | 有限 | Bun |

### 选型建议

1. **追求极致性能**：选择 Bun
2. **需要安全沙箱**：选择 Deno
3. **需要稳定生态**：选择 Node.js
4. **快速原型开发**：选择 Bun
5. **长期维护项目**：选择 Node.js/Bun

---

> [!SUCCESS]
> 本文档全面覆盖了 Bun 的核心知识，包括运行时架构、内置工具链、Bun.serve Web 服务器、包管理器和性能基准。Bun 在 2026 年已成为 Node.js 的强力竞争者，其一体化设计、极致性能和 npm 高度兼容性使其成为 vibecoding 场景下 JavaScript/TypeScript 后端开发的首选运行时。对于新项目和性能敏感的应用，建议优先考虑 Bun。

---

## 核心技术深入解析

### Bun 环境变量与配置管理

Bun 提供了统一的环境变量访问机制，支持多种运行时环境，同时内置了类型安全的配置管理功能。深入理解环境变量处理对于构建生产级应用至关重要。

#### 环境变量基础

```typescript
// src/config/env.ts
import { env } from "bun";

// 基础环境变量访问
const PORT = env.PORT || "3000";
const NODE_ENV = env.NODE_ENV || "development";
const DATABASE_URL = env.DATABASE_URL!;  // 必填变量

// 类型化的环境变量访问
interface AppConfig {
  PORT: number;
  NODE_ENV: "development" | "production" | "test";
  DATABASE_URL: string;
  REDIS_URL?: string;
  JWT_SECRET: string;
  JWT_EXPIRES_IN: string;
}

function getAppConfig(): AppConfig {
  const port = parseInt(env.PORT || "3000", 10);
  if (isNaN(port) || port < 1 || port > 65535) {
    throw new Error("Invalid PORT environment variable");
  }

  const nodeEnv = env.NODE_ENV as AppConfig["NODE_ENV"];
  if (!["development", "production", "test"].includes(nodeEnv)) {
    throw new Error("Invalid NODE_ENV environment variable");
  }

  return {
    PORT: port,
    NODE_ENV: nodeEnv,
    DATABASE_URL: env.DATABASE_URL!,
    REDIS_URL: env.REDIS_URL,
    JWT_SECRET: env.JWT_SECRET || "default-secret-change-in-production",
    JWT_EXPIRES_IN: env.JWT_EXPIRES_IN || "7d",
  };
}

// 环境特定配置
interface DatabaseConfig {
  url: string;
  poolSize: number;
  ssl: boolean;
  timeout: number;
}

function getDatabaseConfig(): DatabaseConfig {
  const env = getAppConfig();
  
  return {
    url: env.DATABASE_URL,
    poolSize: parseInt(Bun.env.DB_POOL_SIZE || "10", 10),
    ssl: Bun.env.DB_SSL === "true",
    timeout: parseInt(Bun.env.DB_TIMEOUT || "30000", 10),
  };
}

// 配置文件加载
interface Config {
  app: {
    port: number;
    host: string;
    env: string;
  };
  database: DatabaseConfig;
  redis: {
    url: string;
    password?: string;
  };
  jwt: {
    secret: string;
    expiresIn: string;
  };
  cors: {
    origins: string[];
    credentials: boolean;
  };
}

function loadConfig(): Config {
  const appConfig = getAppConfig();
  
  return {
    app: {
      port: appConfig.PORT,
      host: Bun.env.HOST || "0.0.0.0",
      env: appConfig.NODE_ENV,
    },
    database: getDatabaseConfig(),
    redis: {
      url: appConfig.REDIS_URL || "redis://localhost:6379",
      password: Bun.env.REDIS_PASSWORD,
    },
    jwt: {
      secret: appConfig.JWT_SECRET,
      expiresIn: appConfig.JWT_EXPIRES_IN,
    },
    cors: {
      origins: (Bun.env.CORS_ORIGINS || "http://localhost:3000").split(","),
      credentials: Bun.env.CORS_CREDENTIALS === "true",
    },
  };
}

export const config = loadConfig();
```

#### 环境验证与类型检查

```typescript
// src/config/validate.ts
import { z } from "zod";

// 环境变量 Schema 验证
const envSchema = z.object({
  PORT: z.string().regex(/^\d+$/).transform(Number),
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url().optional(),
  JWT_SECRET: z.string().min(32),
  JWT_EXPIRES_IN: z.string().default("7d"),
  CORS_ORIGINS: z.string().default("http://localhost:3000"),
});

function validateEnvironment(): void {
  const result = envSchema.safeParse({
    PORT: Bun.env.PORT,
    NODE_ENV: Bun.env.NODE_ENV,
    DATABASE_URL: Bun.env.DATABASE_URL,
    REDIS_URL: Bun.env.REDIS_URL,
    JWT_SECRET: Bun.env.JWT_SECRET,
    JWT_EXPIRES_IN: Bun.env.JWT_EXPIRES_IN,
    CORS_ORIGINS: Bun.env.CORS_ORIGINS,
  });

  if (!result.success) {
    const errors = result.error.errors.map(e => `${e.path.join(".")}: ${e.message}`);
    throw new Error(`Environment validation failed:\n${errors.join("\n")}`);
  }
}

// 启动时验证
validateEnvironment();

// 运行时配置检查
function checkRequiredEnvVars(): void {
  const required = ["DATABASE_URL", "JWT_SECRET"];
  const missing = required.filter(key => !Bun.env[key]);

  if (missing.length > 0) {
    console.error(`Missing required environment variables: ${missing.join(", ")}`);
    process.exit(1);
  }
}

checkRequiredEnvVars();
```

### Bun 测试框架深度解析

Bun 内置的测试框架提供了完整的测试能力，包括单元测试、集成测试、Mock、Spies 和完整的断言库。理解测试框架的各个方面对于构建可靠的应用程序至关重要。

#### 测试配置与结构

```typescript
// src/__tests__/setup.ts
import { beforeAll, afterAll, beforeEach, afterEach, describe, expect, test, it } from "bun:test";
import { Database } from "bun:sqlite";

// 全局测试配置
let testDb: Database;

beforeAll(() => {
  // 创建测试数据库
  testDb = new Database(":memory:");
  
  // 初始化测试数据
  testDb.run(`
    CREATE TABLE users (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      email TEXT NOT NULL UNIQUE,
      username TEXT NOT NULL UNIQUE,
      password_hash TEXT NOT NULL,
      created_at TEXT DEFAULT CURRENT_TIMESTAMP
    )
  `);

  testDb.run(`
    CREATE TABLE posts (
      id INTEGER PRIMARY KEY AUTOINCREMENT,
      title TEXT NOT NULL,
      content TEXT NOT NULL,
      author_id INTEGER NOT NULL,
      published INTEGER DEFAULT 0,
      created_at TEXT DEFAULT CURRENT_TIMESTAMP,
      FOREIGN KEY (author_id) REFERENCES users(id)
    )
  `);

  testDb.run(`
    CREATE INDEX idx_posts_author ON posts(author_id)
  `);
});

afterAll(() => {
  // 清理测试数据库
  testDb.close();
});

// 测试数据生成器
function createTestUser(email?: string, username?: string) {
  const id = Math.floor(Math.random() * 1000000);
  return {
    id,
    email: email || `test${id}@example.com`,
    username: username || `user${id}`,
    passwordHash: "$2a$12$hashedpassword",
    createdAt: new Date().toISOString(),
  };
}

function createTestPost(authorId: number, title?: string) {
  const id = Math.floor(Math.random() * 1000000);
  return {
    id,
    title: title || `Test Post ${id}`,
    content: "Test content for the post",
    authorId,
    published: false,
    createdAt: new Date().toISOString(),
  };
}

// 导出测试工具
export { testDb, createTestUser, createTestPost };
```

#### 单元测试示例

```typescript
// src/__tests__/user.service.test.ts
import { describe, test, expect, beforeEach, vi } from "bun:test";
import { createTestUser, createTestPost, testDb } from "./setup";

// 被测试的服务
class UserService {
  private db: Database;

  constructor(db: Database) {
    this.db = db;
  }

  findAll() {
    return this.db.query("SELECT * FROM users").all();
  }

  findById(id: number) {
    return this.db.query("SELECT * FROM users WHERE id = ?").get(id);
  }

  findByEmail(email: string) {
    return this.db.query("SELECT * FROM users WHERE email = ?").get(email);
  }

  create(data: { email: string; username: string; passwordHash: string }) {
    const result = this.db.query(
      "INSERT INTO users (email, username, password_hash) VALUES (?, ?, ?)"
    ).run(data.email, data.username, data.passwordHash);
    
    return this.findById(result.lastInsertRowid as number);
  }

  update(id: number, data: Partial<{ email: string; username: string }>) {
    const updates: string[] = [];
    const values: any[] = [];

    if (data.email) {
      updates.push("email = ?");
      values.push(data.email);
    }
    if (data.username) {
      updates.push("username = ?");
      values.push(data.username);
    }

    if (updates.length === 0) return this.findById(id);

    values.push(id);
    this.db.query(`UPDATE users SET ${updates.join(", ")} WHERE id = ?`).run(...values);
    
    return this.findById(id);
  }

  delete(id: number) {
    const result = this.db.query("DELETE FROM users WHERE id = ?").run(id);
    return result.changes > 0;
  }

  count() {
    const result = this.db.query("SELECT COUNT(*) as count FROM users").get() as any;
    return result.count;
  }
}

describe("UserService", () => {
  let userService: UserService;

  beforeEach(() => {
    // 每个测试前重置数据库
    testDb.run("DELETE FROM posts");
    testDb.run("DELETE FROM users");
    userService = new UserService(testDb);
  });

  describe("create", () => {
    test("should create a new user", () => {
      const userData = {
        email: "test@example.com",
        username: "testuser",
        passwordHash: "hashed_password",
      };

      const user = userService.create(userData);

      expect(user).toBeDefined();
      expect(user.email).toBe(userData.email);
      expect(user.username).toBe(userData.username);
      expect(user.password_hash).toBe(userData.passwordHash);
      expect(user.id).toBeGreaterThan(0);
    });

    test("should throw on duplicate email", () => {
      const userData = {
        email: "duplicate@example.com",
        username: "user1",
        passwordHash: "hash1",
      };

      userService.create(userData);
      
      expect(() => {
        userService.create({
          ...userData,
          username: "user2",
        });
      }).toThrow();
    });

    test("should throw on duplicate username", () => {
      const userData = {
        email: "user1@example.com",
        username: "duplicateuser",
        passwordHash: "hash1",
      };

      userService.create(userData);
      
      expect(() => {
        userService.create({
          ...userData,
          email: "user2@example.com",
        });
      }).toThrow();
    });
  });

  describe("findById", () => {
    test("should return user when exists", () => {
      const created = userService.create({
        email: "find@example.com",
        username: "finduser",
        passwordHash: "hash",
      });

      const found = userService.findById(created.id);

      expect(found).toBeDefined();
      expect(found.id).toBe(created.id);
      expect(found.email).toBe(created.email);
    });

    test("should return undefined when not exists", () => {
      const found = userService.findById(99999);
      expect(found).toBeUndefined();
    });
  });

  describe("findByEmail", () => {
    test("should return user when email exists", () => {
      const created = userService.create({
        email: "email@example.com",
        username: "emailuser",
        passwordHash: "hash",
      });

      const found = userService.findByEmail("email@example.com");

      expect(found).toBeDefined();
      expect(found.email).toBe(created.email);
    });

    test("should return undefined for non-existent email", () => {
      const found = userService.findByEmail("nonexistent@example.com");
      expect(found).toBeUndefined();
    });
  });

  describe("update", () => {
    test("should update user email", () => {
      const created = userService.create({
        email: "original@example.com",
        username: "updateuser",
        passwordHash: "hash",
      });

      const updated = userService.update(created.id, {
        email: "updated@example.com",
      });

      expect(updated.email).toBe("updated@example.com");
      expect(updated.username).toBe("updateuser");
    });

    test("should update user username", () => {
      const created = userService.create({
        email: "username@example.com",
        username: "oldusername",
        passwordHash: "hash",
      });

      const updated = userService.update(created.id, {
        username: "newusername",
      });

      expect(updated.username).toBe("newusername");
      expect(updated.email).toBe("username@example.com");
    });

    test("should return null for non-existent user", () => {
      const updated = userService.update(99999, { email: "test@example.com" });
      expect(updated).toBeUndefined();
    });
  });

  describe("delete", () => {
    test("should delete existing user", () => {
      const created = userService.create({
        email: "delete@example.com",
        username: "deleteuser",
        passwordHash: "hash",
      });

      const deleted = userService.delete(created.id);
      const found = userService.findById(created.id);

      expect(deleted).toBe(true);
      expect(found).toBeUndefined();
    });

    test("should return false for non-existent user", () => {
      const deleted = userService.delete(99999);
      expect(deleted).toBe(false);
    });
  });

  describe("count", () => {
    test("should return correct count", () => {
      expect(userService.count()).toBe(0);

      userService.create({
        email: "count1@example.com",
        username: "count1",
        passwordHash: "hash",
      });
      expect(userService.count()).toBe(1);

      userService.create({
        email: "count2@example.com",
        username: "count2",
        passwordHash: "hash",
      });
      expect(userService.count()).toBe(2);
    });
  });
});
```

#### Mock 与 Spy 示例

```typescript
// src/__tests__/mock.test.ts
import { describe, test, expect, vi, beforeEach, afterEach } from "bun:test";

// Mock 函数
describe("Mock Functions", () => {
  test("should track calls", () => {
    const mockFn = vi.fn((x: number) => x * 2);

    mockFn(1);
    mockFn(2);
    mockFn(3);

    expect(mockFn).toHaveBeenCalledTimes(3);
    expect(mockFn).toHaveBeenCalledWith(1);
    expect(mockFn).toHaveBeenCalledWith(2);
    expect(mockFn).toHaveBeenCalledWith(3);
    expect(mockFn).toHaveBeenLastCalledWith(3);
  });

  test("should return custom values", () => {
    const mockFn = vi.fn()
      .mockReturnValueOnce(1)
      .mockReturnValueOnce(2)
      .mockReturnValue(3);

    expect(mockFn()).toBe(1);
    expect(mockFn()).toBe(2);
    expect(mockFn()).toBe(3);
    expect(mockFn()).toBe(3); // 默认值
  });

  test("should mock implementation", () => {
    const mockFn = vi.fn((a: number, b: number) => a + b);

    mockFn.mockImplementation((a, b) => {
      return a * b; // 覆盖为乘法
    });

    expect(mockFn(3, 4)).toBe(12);
  });
});

// Spy 示例
describe("Spies", () => {
  test("should spy on object methods", () => {
    const calculator = {
      add: (a: number, b: number) => a + b,
      subtract: (a: number, b: number) => a - b,
    };

    const addSpy = vi.spyOn(calculator, "add");

    calculator.add(1, 2);
    calculator.add(3, 4);

    expect(addSpy).toHaveBeenCalledTimes(2);
    expect(addSpy).toHaveBeenCalledWith(1, 2);
    expect(addSpy).toHaveBeenCalledWith(3, 4);

    // 仍然返回正确结果
    expect(calculator.add(5, 6)).toBe(11);
  });

  test("should mock return value with spy", () => {
    const userService = {
      getUser: vi.fn().mockResolvedValue({
        id: 1,
        name: "Test User",
        email: "test@example.com",
      }),
    };

    await expect(userService.getUser(1)).resolves.toEqual({
      id: 1,
      name: "Test User",
      email: "test@example.com",
    });
  });
});

// Mock 模块
describe("Module Mocks", () => {
  test("should mock external dependencies", async () => {
    // 模拟 API 调用
    const mockFetch = vi.fn().mockResolvedValue({
      ok: true,
      json: async () => ({ id: 1, name: "Mocked" }),
    });

    global.fetch = mockFetch;

    const response = await fetch("https://api.example.com/users/1");
    const data = await response.json();

    expect(mockFetch).toHaveBeenCalledWith("https://api.example.com/users/1");
    expect(data.name).toBe("Mocked");

    // 恢复原始 fetch
    vi.restoreAllMocks();
  });
});

// 异步测试
describe("Async Testing", () => {
  test("should handle async operations", async () => {
    const asyncFunction = vi.fn().mockImplementation(async (delay: number) => {
      await new Promise(resolve => setTimeout(resolve, delay));
      return "completed";
    });

    const result = await asyncFunction(10);
    expect(result).toBe("completed");
    expect(asyncFunction).toHaveBeenCalledWith(10);
  });

  test("should handle async errors", async () => {
    const failingAsync = vi.fn().mockRejectedValue(new Error("Async error"));

    await expect(failingAsync()).rejects.toThrow("Async error");
  });

  test("should test promises with resolves/rejects", async () => {
    const successPromise = Promise.resolve("success");
    const failPromise = Promise.reject(new Error("failure"));

    await expect(successPromise).resolves.toBe("success");
    await expect(failPromise).rejects.toThrow("failure");
  });
});

// 定时器测试
describe("Timers", () => {
  test("should mock timers", () => {
    vi.useFakeTimers();

    const callback = vi.fn();
    setTimeout(callback, 1000);

    expect(callback).not.toHaveBeenCalled();

    vi.advanceTimersByTime(1000);

    expect(callback).toHaveBeenCalled();

    vi.useRealTimers();
  });

  test("should handle intervals", () => {
    vi.useFakeTimers();

    let count = 0;
    const intervalId = setInterval(() => {
      count++;
      if (count >= 3) clearInterval(intervalId);
    }, 100);

    vi.advanceTimersByTime(350);

    expect(count).toBe(3);

    vi.useRealTimers();
  });
});
```

### Bun 内置工具链深度使用

#### Bun Test 运行器配置

```bash
# bunfig.toml - Bun 测试配置
[test]
# 测试文件模式
match = "*.test.ts"

# 测试目录
dir = "./src/__tests__"

# 是否显示覆盖率
coverage = true
coverage_threshold = 80

# 是否在第一个失败时停止
bail = false

# 测试超时时间（毫秒）
timeout = 5000

# 是否启用详细输出
verbose = true

# 并发测试数
concurrency = 4

[test.preload]
# 预加载脚本
# "./setup-tests.ts"
```

#### Bun Build 高级配置

```typescript
// build.config.ts
import { defineConfig } from "bun";

export default defineConfig({
  // 入口点
  entrypoints: ["./src/index.ts"],

  // 输出配置
  outdir: "./dist",
  target: "bun",

  // 格式化
  format: "esm",
  splitting: true,

  // 优化
  minify: process.env.NODE_ENV === "production",
  sourcemap: "linked",

  // 代码分割
  chunkNaming: "[name]-[hash].js",
  assetNaming: "[name]-[hash][ext]",

  // 外部依赖
  external: ["react", "react-dom"],

  // 别名
  alias: {
    "@": "./src",
    "@components": "./src/components",
    "@utils": "./src/utils",
  },

  // 环境变量
  define: {
    __VERSION__: '"1.0.0"',
    __DEV__: process.env.NODE_ENV !== "production",
  },

  // 加载器配置
  loader: {
    ".ts": "ts",
    ".tsx": "tsx",
    ".json": "json",
    ".css": "css",
    ".svg": "dataurl",
  },

  // 插件
  plugins: [
    // 自定义插件示例
    {
      name: "my-plugin",
      setup(build) {
        build.onLoad({ filter: /.*\.custom$/ }, (args) => {
          return {
            contents: `export default "custom content"`,
            loader: "js",
          };
        });
      },
    },
  ],
});
```

#### Bun Bundle API 使用

```typescript
// scripts/bundle.ts
import { bundle } from "bun";
import { writeFile, mkdir } from "bun:fs";

async function buildClient() {
  const { outputs } = await bundle({
    entrypoints: ["./src/client/index.tsx"],
    outdir: "./dist/client",
    target: "browser",
    minify: true,
    sourcemap: true,
    splitting: true,
    format: "esm",
    define: {
      "process.env.NODE_ENV": '"production"',
    },
    loader: {
      ".tsx": "tsx",
      ".ts": "ts",
    },
    external: [],
  });

  // 写入输出文件
  for (const output of outputs) {
    const path = output.path.replace("./dist/client/", "");
    await mkdir(`./dist/client/${path.split("/").slice(0, -1).join("/")}`, {
      recursive: true,
    });
    await writeFile(output.path, output.contents);
  }

  console.log("Client build completed!");
}

async function buildServer() {
  const { outputs } = await bundle({
    entrypoints: ["./src/server/index.ts"],
    outdir: "./dist/server",
    target: "bun",
    minify: false,
    sourcemap: true,
    format: "esm",
    define: {
      "process.env.NODE_ENV": '"production"',
    },
  });

  for (const output of outputs) {
    await writeFile(output.path, output.contents);
  }

  console.log("Server build completed!");
}

async function buildAll() {
  console.log("Starting build...");
  await Promise.all([buildClient(), buildServer()]);
  console.log("Build completed successfully!");
}

buildAll().catch(console.error);
```

---

## 企业级应用架构

### 分层架构设计

```typescript
// src/architecture/index.ts
// 企业级分层架构示例

// ==================== 常量定义 ====================
const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  NO_CONTENT: 204,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  INTERNAL_SERVER_ERROR: 500,
  SERVICE_UNAVAILABLE: 503,
} as const;

const ErrorCodes = {
  VALIDATION_ERROR: "VALIDATION_ERROR",
  NOT_FOUND: "NOT_FOUND",
  UNAUTHORIZED: "UNAUTHORIZED",
  FORBIDDEN: "FORBIDDEN",
  CONFLICT: "CONFLICT",
  INTERNAL_ERROR: "INTERNAL_ERROR",
  SERVICE_UNAVAILABLE: "SERVICE_UNAVAILABLE",
} as const;

// ==================== 结果类型 ====================
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

interface AppError extends Error {
  code: string;
  statusCode: number;
  details?: Record<string, any>;
}

function success<T>(data: T): Result<T> {
  return { success: true, data };
}

function failure<E extends AppError>(error: E): Result<never, E> {
  return { success: false, error };
}

// ==================== 基础错误类 ====================
class ApplicationError extends Error implements AppError {
  code: string;
  statusCode: number;
  details?: Record<string, any>;

  constructor(
    message: string,
    code: string,
    statusCode: number,
    details?: Record<string, any>
  ) {
    super(message);
    this.name = "ApplicationError";
    this.code = code;
    this.statusCode = statusCode;
    this.details = details;
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends ApplicationError {
  constructor(message: string, details?: Record<string, any>) {
    super(message, ErrorCodes.VALIDATION_ERROR, HTTP_STATUS.BAD_REQUEST, details);
    this.name = "ValidationError";
  }
}

class NotFoundError extends ApplicationError {
  constructor(resource: string, id?: string | number) {
    const message = id !== undefined 
      ? `${resource} with id ${id} not found`
      : `${resource} not found`;
    super(message, ErrorCodes.NOT_FOUND, HTTP_STATUS.NOT_FOUND);
    this.name = "NotFoundError";
  }
}

class UnauthorizedError extends ApplicationError {
  constructor(message: string = "Unauthorized") {
    super(message, ErrorCodes.UNAUTHORIZED, HTTP_STATUS.UNAUTHORIZED);
    this.name = "UnauthorizedError";
  }
}

class ForbiddenError extends ApplicationError {
  constructor(message: string = "Access denied") {
    super(message, ErrorCodes.FORBIDDEN, HTTP_STATUS.FORBIDDEN);
    this.name = "ForbiddenError";
  }
}

class ConflictError extends ApplicationError {
  constructor(message: string, field?: string) {
    super(message, ErrorCodes.CONFLICT, HTTP_STATUS.CONFLICT, 
      field ? { field } : undefined);
    this.name = "ConflictError";
  }
}

// ==================== Repository 层 ====================
interface Repository<T, ID = number> {
  findById(id: ID): Promise<T | null>;
  findAll(options?: FindOptions): Promise<PaginatedResult<T>>;
  create(data: CreateInput<T>): Promise<T>;
  update(id: ID, data: Partial<UpdateInput<T>>): Promise<T>;
  delete(id: ID): Promise<boolean>;
  exists(id: ID): Promise<boolean>;
}

interface FindOptions {
  page?: number;
  limit?: number;
  sort?: SortOption[];
  filter?: FilterOption[];
}

interface SortOption {
  field: string;
  order: "asc" | "desc";
}

interface FilterOption {
  field: string;
  operator: "eq" | "ne" | "gt" | "gte" | "lt" | "lte" | "in" | "like";
  value: any;
}

interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

interface CreateInput<T> {
  [key: string]: any;
}

interface UpdateInput<T> {
  [key: string]: any;
}

// ==================== Service 层 ====================
interface Service<T, ID = number, CreateDTO = any, UpdateDTO = any> {
  getById(id: ID): Promise<Result<T, ApplicationError>>;
  getAll(options?: FindOptions): Promise<Result<PaginatedResult<T>, ApplicationError>>;
  create(data: CreateDTO): Promise<Result<T, ApplicationError>>;
  update(id: ID, data: UpdateDTO): Promise<Result<T, ApplicationError>>;
  delete(id: ID): Promise<Result<void, ApplicationError>>;
}

// ==================== Controller 层 ====================
interface Controller<T> {
  handleGet(ctx: Context): Promise<Response>;
  handleGetById(ctx: Context): Promise<Response>;
  handlePost(ctx: Context): Promise<Response>;
  handlePut(ctx: Context): Promise<Response>;
  handleDelete(ctx: Context): Promise<Response>;
  handlePatch(ctx: Context): Promise<Response>;
}

// ==================== Middleware 层 ====================
type Middleware = (c: Context, next: () => Promise<void>) => Promise<Response | void>;

interface MiddlewareBuilder {
  use(...middlewares: Middleware[]): this;
  useAsync(...middlewares: AsyncMiddleware[]): this;
}

type AsyncMiddleware = (c: Context, next: () => Promise<void>) => Promise<void>;

// ==================== 统一响应格式 ====================
interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: Record<string, any>;
  };
  meta?: {
    timestamp: string;
    requestId: string;
    pagination?: {
      page: number;
      limit: number;
      total: number;
      totalPages: number;
    };
  };
}

function createResponse<T>(
  data: T,
  statusCode: number = HTTP_STATUS.OK,
  meta?: Partial<ApiResponse["meta"]>
): Response {
  const response: ApiResponse<T> = {
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      requestId: crypto.randomUUID(),
      ...meta,
    },
  };

  return Response.json(response, { status: statusCode });
}

function createErrorResponse(
  error: ApplicationError,
  requestId?: string
): Response {
  const response: ApiResponse = {
    success: false,
    error: {
      code: error.code,
      message: error.message,
      details: error.details,
    },
    meta: {
      timestamp: new Date().toISOString(),
      requestId: requestId || crypto.randomUUID(),
    },
  };

  return Response.json(response, { status: error.statusCode });
}
```

### 依赖注入与工厂模式

```typescript
// src/di/container.ts
// 简单的依赖注入容器

interface Constructor<T = any> {
  new (...args: any[]): T;
}

interface Factory<T = any> {
  (): T;
}

type Provider<T = any> = Constructor<T> | Factory<T>;

class Container {
  private providers = new Map<string, Provider>();
  private singletons = new Map<string, any>();
  private aliases = new Map<string, string>();

  register<T>(token: string, provider: Provider<T>): this {
    this.providers.set(token, provider);
    return this;
  }

  registerSingleton<T>(token: string, provider: Provider<T>): this {
    this.providers.set(token, provider);
    this.singletons.set(token, undefined); // 标记为单例
    return this;
  }

  registerInstance<T>(token: string, instance: T): this {
    this.providers.set(token, () => instance);
    this.singletons.set(token, instance); // 预实例化
    return this;
  }

  alias(source: string, target: string): this {
    this.aliases.set(source, target);
    return this;
  }

  resolve<T>(token: string): T {
    const resolvedToken = this.aliases.get(token) || token;
    const provider = this.providers.get(resolvedToken);

    if (!provider) {
      throw new Error(`No provider found for token: ${token}`);
    }

    // 如果是单例且已实例化
    if (this.singletons.has(resolvedToken)) {
      const existing = this.singletons.get(resolvedToken);
      if (existing !== undefined) {
        return existing;
      }
    }

    // 实例化
    const instance = provider();

    // 如果是单例，保存实例
    if (this.singletons.has(resolvedToken)) {
      this.singletons.set(resolvedToken, instance);
    }

    return instance as T;
  }

  clear(): void {
    this.providers.clear();
    this.singletons.clear();
    this.aliases.clear();
  }
}

// 全局容器
const container = new Container();

// 常用令牌
const TOKENS = {
  DATABASE: "database",
  CACHE: "cache",
  LOGGER: "logger",
  CONFIG: "config",
  USER_SERVICE: "userService",
  POST_SERVICE: "postService",
} as const;

export { container, TOKENS, type Provider, type Constructor, type Factory };
```

### 日志系统

```typescript
// src/utils/logger.ts
// 结构化日志系统

enum LogLevel {
  DEBUG = 0,
  INFO = 1,
  WARN = 2,
  ERROR = 3,
  FATAL = 4,
}

const LOG_LEVEL_NAMES = ["DEBUG", "INFO", "WARN", "ERROR", "FATAL"] as const;

interface LogEntry {
  timestamp: string;
  level: string;
  message: string;
  context?: Record<string, any>;
  error?: {
    message: string;
    stack?: string;
    name: string;
  };
  requestId?: string;
  userId?: string;
}

interface Logger {
  debug(message: string, context?: Record<string, any>): void;
  info(message: string, context?: Record<string, any>): void;
  warn(message: string, context?: Record<string, any>): void;
  error(message: string, error?: Error, context?: Record<string, any>): void;
  fatal(message: string, error?: Error, context?: Record<string, any>): void;
}

class ConsoleLogger implements Logger {
  private minLevel: LogLevel;
  private requestId?: string;
  private userId?: string;

  constructor(minLevel: LogLevel = LogLevel.INFO) {
    this.minLevel = minLevel;
  }

  setContext(requestId?: string, userId?: string): void {
    this.requestId = requestId;
    this.userId = userId;
  }

  private log(level: LogLevel, message: string, context?: Record<string, any>, error?: Error): void {
    if (level < this.minLevel) return;

    const entry: LogEntry = {
      timestamp: new Date().toISOString(),
      level: LOG_LEVEL_NAMES[level],
      message,
      context,
      requestId: this.requestId,
      userId: this.userId,
    };

    if (error) {
      entry.error = {
        message: error.message,
        stack: error.stack,
        name: error.name,
      };
    }

    const output = JSON.stringify(entry);
    
    switch (level) {
      case LogLevel.DEBUG:
      case LogLevel.INFO:
        console.log(output);
        break;
      case LogLevel.WARN:
        console.warn(output);
        break;
      case LogLevel.ERROR:
      case LogLevel.FATAL:
        console.error(output);
        break;
    }
  }

  debug(message: string, context?: Record<string, any>): void {
    this.log(LogLevel.DEBUG, message, context);
  }

  info(message: string, context?: Record<string, any>): void {
    this.log(LogLevel.INFO, message, context);
  }

  warn(message: string, context?: Record<string, any>): void {
    this.log(LogLevel.WARN, message, context);
  }

  error(message: string, error?: Error, context?: Record<string, any>): void {
    this.log(LogLevel.ERROR, message, context, error);
  }

  fatal(message: string, error?: Error, context?: Record<string, any>): void {
    this.log(LogLevel.FATAL, message, context, error);
  }
}

// 创建日志实例
function createLogger(minLevel?: string): Logger {
  const level = minLevel ? LogLevel[minLevel.toUpperCase() as keyof typeof LogLevel] : LogLevel.INFO;
  return new ConsoleLogger(level);
}

// 应用日志中间件
function loggerMiddleware(logger: Logger) {
  return async (c: any, next: () => Promise<void>) => {
    const requestId = c.req.header("X-Request-ID") || crypto.randomUUID();
    const start = Date.now();

    // 设置日志上下文
    if (logger instanceof ConsoleLogger) {
      logger.setContext(requestId);
    }

    c.set("requestId", requestId);

    try {
      await next();
      
      const duration = Date.now() - start;
      logger.info(`${c.req.method} ${c.req.url} completed in ${duration}ms`, {
        status: c.res.status,
        duration,
        requestId,
      });
    } catch (error) {
      const duration = Date.now() - start;
      logger.error(`${c.req.method} ${c.req.url} failed after ${duration}ms`, 
        error as Error, { duration, requestId });
      throw error;
    }
  };
}

export { loggerMiddleware, createLogger, type Logger, LogLevel };
```

---

## 生产环境部署

### 进程管理与 PM2 配置

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "myapp-api",
      script: "./dist/server/index.js",
      instances: "max",
      exec_mode: "cluster",
      env: {
        NODE_ENV: "development",
        PORT: 3000,
      },
      env_production: {
        NODE_ENV: "production",
        PORT: 3000,
        DATABASE_URL: "postgresql://...",
      },
      // 性能配置
      max_memory_restart: "512M",
      node_args: "--max-old-space-size=512",
      
      // 日志配置
      log_file: "./logs/combined.log",
      out_file: "./logs/out.log",
      error_file: "./logs/error.log",
      log_date_format: "YYYY-MM-DD HH:mm:ss Z",
      merge_logs: true,
      
      // 监控
      pmx: true,
      APM: true,
      
      // 优雅关闭
      kill_timeout: 5000,
      listen_timeout: 3000,
      shutdown_with_message: true,
    },
  ],
};
```

### Nginx 反向代理配置

```nginx
# /etc/nginx/conf.d/myapp.conf

upstream myapp_backend {
    least_conn;
    server 127.0.0.1:3000 weight=5;
    server 127.0.0.1:3001 weight=3;
    server 127.0.0.1:3002 weight=2;
    keepalive 64;
}

# 主服务器块
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name api.example.com;

    # SSL 配置
    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/api.example.com/chain.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    gzip_disable "MSIE [1-6]\.";

    # 客户端配置
    client_max_body_size 10M;
    client_body_timeout 30s;
    client_header_timeout 30s;

    # 位置配置
    location / {
        proxy_pass http://myapp_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Request-ID $request_id;
        proxy_connect_timeout 5s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # WebSocket 支持
    location /ws {
        proxy_pass http://myapp_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;
    }

    # 静态文件（如果有）
    location /static {
        alias /var/www/myapp/static;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 健康检查
    location /health {
        proxy_pass http://myapp_backend;
        access_log off;
    }

    # 日志
    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;
}
```

### 环境配置管理

```bash
# .env.example - 环境变量模板
# 复制此文件为 .env 并填写实际值

# 应用配置
NODE_ENV=development
PORT=3000
HOST=0.0.0.0

# 数据库
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
DB_POOL_SIZE=10
DB_SSL=false
DB_TIMEOUT=30000

# Redis
REDIS_URL=redis://localhost:6379
REDIS_PASSWORD=
REDIS_DB=0

# JWT
JWT_SECRET=your-secret-key-at-least-32-characters-long
JWT_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d

# CORS
CORS_ORIGINS=http://localhost:3000,https://example.com
CORS_CREDENTIALS=true

# 日志
LOG_LEVEL=info

# 可选服务
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=noreply@example.com
SMTP_PASSWORD=
```

---

## 性能调优指南

### 数据库连接池优化

```typescript
// src/db/pool.ts
// 优化的数据库连接池配置

interface PoolConfig {
  min: number;
  max: number;
  idleTimeout: number;
  connectionTimeout: number;
  reapInterval: number;
}

function createOptimizedPoolConfig(): PoolConfig {
  const cpuCores = navigator.hardwareConcurrency || 4;
  
  return {
    // 最小连接数：根据 CPU 核心数
    min: Math.max(2, cpuCores),
    
    // 最大连接数：通常设置为 CPU 核心数的 2-4 倍
    max: Math.min(20, cpuCores * 4),
    
    // 空闲超时：超过此时间未使用的连接将被关闭
    idleTimeout: 30000, // 30 秒
    
    // 连接超时：等待连接的最大时间
    connectionTimeout: 10000, // 10 秒
    
    // 检查间隔：检查空闲连接的频率
    reapInterval: 1000, // 1 秒
  };
}

// 连接池监控
class PoolMonitor {
  private pool: any;
  private interval: any;

  constructor(pool: any) {
    this.pool = pool;
  }

  start(intervalMs: number = 60000) {
    this.interval = setInterval(() => {
      this.logStatus();
    }, intervalMs);
  }

  stop() {
    if (this.interval) {
      clearInterval(this.interval);
    }
  }

  private logStatus() {
    const status = {
      total: this.pool.totalCount,
      idle: this.pool.idleCount,
      waiting: this.pool.waitingCount,
    };
    
    console.log("[Pool] Status:", JSON.stringify(status));
    
    // 警告条件
    if (status.waiting > 0) {
      console.warn("[Pool] Requests waiting for connection:", status.waiting);
    }
    
    if (status.idle === 0) {
      console.warn("[Pool] No idle connections available");
    }
  }
}
```

### 缓存策略

```typescript
// src/cache/strategy.ts
// 多层缓存策略

interface CacheOptions {
  ttl: number;
  namespace?: string;
  compress?: boolean;
}

class MultiLayerCache {
  private memory = new Map<string, { data: any; expiresAt: number }>();
  private redis?: RedisCache;
  private defaultTTL: number;

  constructor(redis?: RedisCache, defaultTTL: number = 60000) {
    this.redis = redis;
    this.defaultTTL = defaultTTL;
    
    // 定期清理过期内存缓存
    setInterval(() => this.cleanup(), 60000);
  }

  async get<T>(key: string): Promise<T | null> {
    // 1. 检查内存缓存
    const memoryValue = this.memory.get(key);
    if (memoryValue && memoryValue.expiresAt > Date.now()) {
      return memoryValue.data as T;
    }
    
    // 2. 检查 Redis 缓存
    if (this.redis) {
      const redisValue = await this.redis.get<T>(key);
      if (redisValue !== null) {
        // 回填内存缓存
        this.memory.set(key, {
          data: redisValue,
          expiresAt: Date.now() + this.defaultTTL,
        });
        return redisValue;
      }
    }
    
    return null;
  }

  async set<T>(key: string, value: T, options?: CacheOptions): Promise<void> {
    const ttl = options?.ttl || this.defaultTTL;
    const expiresAt = Date.now() + ttl;

    // 设置内存缓存
    this.memory.set(key, { data: value, expiresAt });

    // 设置 Redis 缓存
    if (this.redis) {
      await this.redis.set(key, value, ttl / 1000);
    }
  }

  async invalidate(pattern: string): Promise<number> {
    let count = 0;

    // 清理内存缓存
    for (const key of this.memory.keys()) {
      if (key.includes(pattern)) {
        this.memory.delete(key);
        count++;
      }
    }

    // 清理 Redis 缓存
    if (this.redis) {
      const redisCount = await this.redis.invalidatePattern(pattern);
      count += redisCount;
    }

    return count;
  }

  private cleanup() {
    const now = Date.now();
    for (const [key, value] of this.memory) {
      if (value.expiresAt <= now) {
        this.memory.delete(key);
      }
    }
  }
}

// 缓存装饰器
function cached(ttl: number = 60000, keyGenerator?: (...args: any[]) => string) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      const cacheKey = keyGenerator
        ? keyGenerator(...args)
        : `${target.constructor.name}:${propertyKey}:${JSON.stringify(args)}`;

      const cachedValue = await cache.get(cacheKey);
      if (cachedValue !== null) {
        return cachedValue;
      }

      const result = await originalMethod.apply(this, args);
      await cache.set(cacheKey, result, { ttl });
      return result;
    };

    return descriptor;
  };
}
```

---

## 附录：常用命令速查表

| 命令 | 说明 | 示例 |
|------|------|------|
| `bun run` | 运行脚本 | `bun run src/index.ts` |
| `bun test` | 运行测试 | `bun test --coverage` |
| `bun install` | 安装依赖 | `bun install --frozen-lockfile` |
| `bun add` | 添加依赖 | `bun add express zod` |
| `bun remove` | 移除依赖 | `bun remove lodash` |
| `bun build` | 打包 | `bun build ./src/index.ts --outdir=dist` |
| `bun init` | 初始化项目 | `bun init` |
| `bun create` | 创建模板 | `bun create next ./my-app` |
| `bun x` | 执行包 | `bun x prisma generate` |
| `bun pm` | 包管理 | `bun pm ls` |
| `bun upgrade` | 升级 Bun | `bun upgrade` |
| `bun --version` | 查看版本 | `bun --version` |

---

## 参考资料

### 官方文档

| 资源 | 链接 |
|------|------|
| Bun 官方文档 | https://bun.sh/docs |
| Bun API 参考 | https://bun.sh/docs/api |
| Bun 运行时 | https://bun.sh/docs/runtime |
| Bun 安装 | https://bun.sh/docs/installation |

### 社区资源

| 资源 | 说明 |
|------|------|
| Bun GitHub | https://github.com/oven-sh/bun |
| Bun Discord | 官方社区支持 |
| Bun Reddit | r/bunjs |
| Awesome Bun | 社区精选资源列表 |

### 性能基准

| 测试 | 说明 |
|------|------|
| Bun Benchmarks | 官方性能测试 |
| TechEmpower | 第三方 Web 框架基准 |
| JSPerf | JavaScript 性能测试 |

---

> [!NOTE]
> 本文档持续更新中，如有问题或建议，欢迎提交 Issue 或 PR。

