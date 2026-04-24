# tRPC 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 tRPC 核心原理、Router/Procedure 定义、Middleware、Zod 验证、Subscriptions，以及在 AI 应用中的端到端类型安全。

---

## 目录

1. [[#tRPC 概述]]
2. [[#tRPC vs REST vs GraphQL 对比]]
3. [[#核心架构]]
4. [[#Server 定义详解]]
5. [[#Client 使用详解]]
6. [[#Middleware 与 Context]]
7. [[#Zod 验证集成]]
8. [[#高级特性]]
9. [[#tRPC 与 Next.js]]
10. [[#tRPC 与 Prisma]]
11. [[#Subscriptions 实时功能]]
12. [[#错误处理与监控]]
13. [[#测试策略]]
14. [[#性能优化]]
15. [[#tRPC 限制与批评]]
16. [[#选型建议]]

---

## tRPC 概述

### 什么是 tRPC

tRPC（TypeScript Remote Procedure Call）是一个用于构建**端到端类型安全 API**的库。tRPC 的核心理念是：在同一个 TypeScript 项目中，Server 和 Client 共享类型定义，无需 Schema 文件或代码生成，实现真正的类型安全。

要理解 tRPC 的设计思路，我们需要回顾一下传统 API 开发中会遇到的问题。在 REST API 中，你需要手动定义接口返回的数据结构；在 GraphQL 中，你需要编写 SDL Schema 并运行代码生成工具。而 tRPC 采取了完全不同的策略：**你写的 TypeScript 类型就是你的 API Schema**。

当你写一个后端的 tRPC Router 时，TypeScript 编译器会自动推断出完整的类型信息，这些类型可以直接被前端代码使用。这意味着如果你在后端改变了一个接口的返回结构，前端代码会在编译时报错，而不是等到运行时才发现问题。

**tRPC 核心特点：**

| 特性 | 说明 |
|------|------|
| **无 Schema** | 直接使用 TypeScript 类型 |
| **端到端类型安全** | Server/Client 共享类型 |
| **零配置** | 无需代码生成 |
| **高性能** | 直接函数调用，无序列化开销 |
| **自动补全** | IDE 全类型提示 |
| **验证集成** | 原生支持 Zod/Valibot |

### tRPC 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| tRPC 1.0 | 2020 | 初始版本，基于 React Query |
| tRPC 9.0 | 2022 | v10，稳定版 |
| tRPC 10.0 | 2022 | 完全重写，更好的性能 |
| tRPC 11.0 | 2024 | 中间件改进，Server Components 支持 |

### 核心理念

tRPC 的设计哲学可以归结为以下几点：

1. **类型即文档**：TypeScript 类型就是 API 文档，你在 IDE 中看到的类型提示就是接口说明
2. **无 Schema 生成**：无需额外的代码生成步骤，类型推断是自动的
3. **直接函数调用**：RPC 风格，语义清晰，不像 GraphQL 需要学习查询语言
4. **运行时安全**：Zod 等验证库确保运行时安全

> [!IMPORTANT]
> tRPC 要求 Server 和 Client 都在同一个 TypeScript 项目中，通过类型导入共享类型。这是 tRPC 的核心约束，也是其零 Schema 方案的基础。

---

## tRPC vs REST vs GraphQL 对比

### 功能对比表

| 特性 | tRPC | REST | GraphQL |
|------|------|------|---------|
| **类型安全** | 原生端到端 | 需手动 | Schema 生成 |
| **Schema 文件** | 无 | OpenAPI | SDL |
| **代码生成** | 无 | 可选 | 必须 |
| **API 调用** | 函数调用 | HTTP 请求 | 查询语言 |
| **Overfetching** | 避免 | 常见 | 避免 |
| **性能** | 最快 | 快 | 中等 |
| **学习曲线** | 低 | 低 | 中等 |
| **生态系统** | 增长中 | 成熟 | 成熟 |
| **框架限制** | 需 TS | 无 | 无 |

### 请求风格对比

**REST API：**

```typescript
// 多个 HTTP 请求
const response = await fetch('/api/users/123')
const user = await response.json()

const postsResponse = await fetch(`/api/users/${user.id}/posts`)
const posts = await postsResponse.json()
```

**GraphQL：**

```graphql
# 单次请求，多字段
query {
  user(id: "123") {
    id
    name
    posts {
      id
      title
    }
  }
}
```

**tRPC：**

```typescript
// 直接函数调用，完全类型安全
const user = await trpc.user.getById.query({ id: '123' })
const posts = await trpc.user.posts.query({ userId: '123' })
```

### 架构对比

```
┌─────────────────────────────────────────────────────────────┐
│                         tRPC 架构                             │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                    TypeScript 项目                      │  │
│  │                                                        │  │
│  │   ┌─────────────────┐       ┌─────────────────────┐   │  │
│  │   │      Server      │◄─────│   共享类型定义      │   │  │
│  │   │   (tRPC server) │       │   (TypeScript)      │   │  │
│  │   └────────┬─────────┘       └─────────────────────┘   │  │
│  │            │                                            │  │
│  │            │ HTTP/WS                                    │  │
│  │            ▼                                            │  │
│  │   ┌─────────────────┐                                  │  │
│  │   │      Client      │                                  │  │
│  │   │  (tRPC client)  │                                  │  │
│  │   │  + 类型自动推断  │                                  │  │
│  │   └─────────────────┘                                  │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心架构

### 项目结构

一个典型的 tRPC 项目结构如下：

```
my-app/
├── src/
│   ├── server/
│   │   ├── index.ts          # tRPC Server 初始化
│   │   ├── router/
│   │   │   ├── index.ts      # 根 Router
│   │   │   ├── user.ts       # User Router
│   │   │   ├── post.ts       # Post Router
│   │   │   └── chat.ts       # Chat Router
│   │   ├── context.ts        # Context 定义
│   │   └── trpc.ts          # tRPC 基础配置
│   │
│   ├── client/
│   │   ├── index.ts          # tRPC Client 初始化
│   │   └── App.tsx           # React 组件
│   │
│   ├── types/
│   │   └── trpc.ts           # 类型导出
│   │
│   └── app/
│       └── [...trpc]/
│           └── route.ts       # Next.js App Router 适配器
│
└── package.json
```

### Server 初始化

```typescript
// server/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server'
import { ZodError } from 'zod'

export const createContext = async () => {
  const user = await getCurrentUser()

  return {
    user,
    prisma,
  }
}

type Context = Awaited<ReturnType<typeof createContext>>

const t = initTRPC.context<Context>().create({
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError:
          error.cause instanceof ZodError ? error.cause.flatten() : null,
      },
    }
  },
})

export const router = t.router
export const publicProcedure = t.procedure
export const middleware = t.middleware
export const mergeRouters = t.mergeRouters
```

---

## Server 定义详解

### 基础 Router

```typescript
// server/router/user.ts
import { z } from 'zod'
import { router, publicProcedure, protectedProcedure } from '../trpc'

export const userRouter = router({
  // 查询：获取用户列表
  list: publicProcedure
    .input(
      z.object({
        limit: z.number().min(1).max(100).default(20),
        cursor: z.string().nullish(),
      })
    )
    .query(async ({ input, ctx }) => {
      const limit = input.limit
      const cursor = input.cursor

      const users = await ctx.prisma.user.findMany({
        take: limit + 1,
        cursor: cursor ? { id: cursor } : undefined,
        orderBy: { createdAt: 'desc' },
      })

      let nextCursor: string | undefined = undefined
      if (users.length > limit) {
        const nextItem = users.pop()
        nextCursor = nextItem!.id
      }

      return {
        users,
        nextCursor,
      }
    }),

  // 查询：获取单个用户
  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.findUnique({
        where: { id: input.id },
      })

      if (!user) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        })
      }

      return user
    }),

  // 查询：获取当前用户（需认证）
  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user
  }),

  // 变更：创建用户
  create: publicProcedure
    .input(
      z.object({
        name: z.string().min(1).max(100),
        email: z.string().email(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      const existing = await ctx.prisma.user.findUnique({
        where: { email: input.email },
      })

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'User with this email already exists',
        })
      }

      return ctx.prisma.user.create({
        data: input,
      })
    }),

  // 变更：更新用户（需认证）
  update: protectedProcedure
    .input(
      z.object({
        id: z.string(),
        name: z.string().min(1).max(100).optional(),
        email: z.string().email().optional(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      if (input.id !== ctx.user.id) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You can only update your own profile',
        })
      }

      return ctx.prisma.user.update({
        where: { id: input.id },
        data: {
          name: input.name,
          email: input.email,
        },
      })
    }),

  // 变更：删除用户（需认证）
  delete: protectedProcedure
    .input(z.object({ id: z.string() }))
    .mutation(async ({ input, ctx }) => {
      if (input.id !== ctx.user.id) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You can only delete your own account',
        })
      }

      await ctx.prisma.user.delete({ where: { id: input.id } })
      return { success: true }
    }),
})
```

### 嵌套 Router

```typescript
// server/router/index.ts
import { router } from '../trpc'
import { userRouter } from './user'
import { postRouter } from './post'

export const appRouter = router({
  user: userRouter,
  post: postRouter,
})

export type AppRouter = typeof appRouter
```

---

## Client 使用详解

### 基础 Client 配置

```typescript
// client/index.ts
import { createTRPCReact, httpBatchLink, loggerLink } from '@trpc/react-query'
import type { AppRouter } from '../../server/router'

export const trpc = createTRPCReact<AppRouter>()
```

### React Query 集成

```typescript
// client/trpc.ts
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'
import { httpBatchLink } from '@trpc/react-query'
import { useState } from 'react'
import { trpc } from './index'

function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient())
  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: '/api/trpc',
          headers() {
            return {
              Authorization: `Bearer ${getAuthToken()}`,
            }
          },
        }),
      ],
    })
  )

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  )
}
```

### 组件中使用

```typescript
// client/components/UserList.tsx
import { trpc } from '../trpc'

function UserList() {
  const { data, isLoading, error } = trpc.user.list.useQuery({
    limit: 20,
  })

  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error: {error.message}</div>

  return (
    <ul>
      {data?.users.map((user) => (
        <li key={user.id}>
          {user.name} ({user.email})
        </li>
      ))}
    </ul>
  )
}

// 创建用户
function CreateUserForm() {
  const trpcContext = trpc.useContext()
  const createUser = trpc.user.create.useMutation({
    onSuccess: () => {
      trpcContext.user.list.invalidate()
      toast.success('User created!')
    },
  })

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault()
    const formData = new FormData(e.target)

    createUser.mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={createUser.isPending}>
        {createUser.isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  )
}
```

---

## Middleware 与 Context

### 中间件定义

tRPC 的中间件机制非常强大，可以用来实现认证、权限控制、日志记录等功能：

```typescript
// server/trpc.ts
// 认证中间件
const isAuthenticated = middleware(({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: 'You must be logged in',
    })
  }
  return next({
    ctx: {
      ...ctx,
      user: ctx.user,
    },
  })
})

// 管理员中间件
const isAdmin = middleware(({ ctx, next }) => {
  if (!ctx.user || ctx.user.role !== 'ADMIN') {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: 'Admin access required',
    })
  }
  return next({ ctx })
})

// 速率限制中间件
import { ratelimit } from '@/lib/ratelimit'

const ratelimitMiddleware = middleware(async ({ ctx, next }) => {
  const identifier = ctx.user?.id || ctx.ip

  const { success, remaining } = await ratelimit.limit(identifier)

  if (!success) {
    throw new TRPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
    })
  }

  return next()
})

// 审计日志中间件
const auditLogMiddleware = middleware(async ({ ctx, next, path, type, input }) => {
  const start = Date.now()
  const result = await next()
  const duration = Date.now() - start

  await ctx.prisma.auditLog.create({
    data: {
      userId: ctx.user?.id,
      action: path,
      type,
      duration,
      input,
    },
  })

  return result
})

// 创建受保护的 Procedure
export const protectedProcedure = publicProcedure
  .use(auditLogMiddleware)
  .use(isAuthenticated)

export const adminProcedure = protectedProcedure.use(isAdmin)
export const rateLimitedProcedure = publicProcedure.use(ratelimitMiddleware)
```

### 使用中间件

```typescript
// server/router/user.ts
export const userRouter = router({
  // 公开查询
  list: publicProcedure.query(...),

  // 需要认证
  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user
  }),

  // 需要管理员权限
  adminList: adminProcedure.query(async ({ ctx }) => {
    return ctx.prisma.user.findMany()
  }),

  // 速率限制
  create: rateLimitedProcedure
    .input(...)
    .mutation(async ({ input, ctx }) => {
      return ctx.prisma.user.create({ data: input })
    }),
})
```

---

## Zod 验证集成

### Zod 基础

Zod 是一个 TypeScript 优先的模式声明和验证库，与 tRPC 配合使用可以确保输入输出的类型安全：

```typescript
import { z } from 'zod'

// 基本类型
z.string()
z.number()
z.boolean()
z.date()
z.array(z.string())
z.object({ name: z.string(), age: z.number() })

// 链式验证
z.string().min(1).max(100)
z.string().email()
z.string().url()
z.number().min(0).max(100)

// 可选和默认值
z.string().optional()
z.string().default('default')
z.number().nullable()

// 复杂结构
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().min(0).max(150).optional(),
  role: z.enum(['ADMIN', 'USER', 'GUEST']),
  createdAt: z.coerce.date(),
  metadata: z.record(z.string(), z.unknown()),
})
```

### 在 tRPC 中使用 Zod

```typescript
// 输入验证
publicProcedure
  .input(
    z.object({
      name: z.string().min(1).max(100),
      email: z.string().email(),
      age: z.number().min(0).max(150).optional(),
      tags: z.array(z.string()).max(10).default([]),
    })
  )
  .mutation(async ({ input }) => {
    // input 已被 Zod 验证并转换
    console.log(input.age) // number | undefined
    console.log(input.tags) // string[]
  })

// 输出验证
publicProcedure
  .output(
    z.object({
      id: z.string(),
      name: z.string(),
    })
  )
  .query(async ({ ctx }) => {
    const users = await ctx.prisma.user.findMany()
    return users
  })
```

### 自定义验证

```typescript
import { z } from 'zod'

// 自定义验证器
const customString = z.string().refine(
  (val) => !val.includes('admin'),
  { message: 'Username cannot contain "admin"' }
)

// 异步验证
const uniqueEmail = z.string().email().refine(
  async (email) => {
    const existing = await db.user.findUnique({ where: { email } })
    return !existing
  },
  { message: 'Email already in use' }
)

// 变换
const sanitizeInput = z.string().transform((val) => val.trim().toLowerCase())

// 预处理
const queryInput = z.object({
  page: z.coerce.number().default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  sort: z.enum(['asc', 'desc']).default('desc'),
})
```

---

## 高级特性

### Subscription（实时）

tRPC 支持通过 WebSocket 实现实时订阅：

```typescript
// server/router/chat.ts
import { router, protectedProcedure } from '../trpc'
import { observable } from '@trpc/server/observable'
import { EventEmitter } from 'events'

const ee = new EventEmitter()

export const chatRouter = router({
  // 发送消息
  sendMessage: protectedProcedure
    .input(z.object({ content: z.string() }))
    .mutation(async ({ input, ctx }) => {
      const message = await ctx.prisma.message.create({
        data: { ...input, senderId: ctx.user.id },
      })

      ee.emit('message', message)
      return message
    }),

  // 订阅新消息
  onMessage: protectedProcedure.subscription(({ ctx }) => {
    return observable<Message>((emit) => {
      const onMessage = (message: Message) => {
        emit.next(message)
      }

      ee.on('message', onMessage)

      return () => {
        ee.off('message', onMessage)
      }
    })
  }),
})
```

### 错误处理

```typescript
// 抛出错误
throw new TRPCError({
  code: 'NOT_FOUND',
  message: 'User not found',
  cause: error,
})

// 预定义错误码
const TRPCErrorCodeMap = {
  INTERNAL_SERVER_ERROR: 500,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  CONFLICT: 409,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,
}
```

---

## tRPC 与 Next.js

### App Router 集成

```typescript
// lib/trpc/server.ts
import { createTRPCProxyClient, httpBatchLink } from '@trpc/client'
import type { AppRouter } from '@/server/router'

function getBaseUrl() {
  if (typeof window !== 'undefined') return ''
  return `http://localhost:${process.env.PORT ?? 3000}`
}

export const trpcServer = createTRPCProxyClient<AppRouter>({
  links: [
    httpBatchLink({
      url: `${getBaseUrl()}/api/trpc`,
    }),
  ],
})

// app/user/[id]/page.tsx
import { notFound } from 'next/navigation'
import { trpcServer } from '@/lib/trpc/server'

export default async function UserPage({
  params,
}: {
  params: { id: string }
}) {
  const user = await trpcServer.user.getById.query({ id: params.id })

  if (!user) {
    notFound()
  }

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}
```

### Server Actions

```typescript
// app/actions/user.ts
'use server'

import { auth } from '@/lib/auth'
import { prisma } from '@/lib/prisma'
import { revalidatePath } from 'next/cache'

export async function updateUserProfile(formData: FormData) {
  const session = await auth()
  
  if (!session?.user?.id) {
    throw new Error('Unauthorized')
  }
  
  const name = formData.get('name') as string
  
  await prisma.user.update({
    where: { id: session.user.id },
    data: { name },
  })
  
  revalidatePath('/settings')
}
```

---

## tRPC 与 Prisma

tRPC 和 Prisma 是现代 TypeScript 全栈开发的黄金组合：

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' 
    ? ['query', 'error', 'warn']
    : ['error'],
})

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma
}
```

---

## Subscriptions 实时功能

### WebSocket 配置

```typescript
// server.ts
import { createServer } from 'http'
import { applyWSSHandler } from '@trpc/server/adapters/ws'
import { appRouter } from './server/router'
import { createContext } from './server/context'

const server = createServer()

const handler = applyWSSHandler({
  wss: new WebSocketServer({ server }),
  router: appRouter,
  createContext,
})

server.listen(3001, () => {
  console.log('Server running on ws://localhost:3001')
})

process.on('SIGTERM', () => {
  console.log('SIGTERM received')
  handler.broadcastReconnectNotification()
})
```

---

## 错误处理与监控

### 错误格式化

```typescript
const t = initTRPC.context<Context>().create({
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError:
          error.cause instanceof ZodError 
            ? error.cause.flatten() 
            : null,
      },
    }
  },
})
```

### Sentry 集成

```typescript
import * as Sentry from '@sentry/nextjs'

const sentryMiddleware = t.middleware(async ({ ctx, next }) => {
  const transaction = Sentry.startTransaction({
    op: 'tRPC',
    name: ctx.operationName,
  })

  try {
    const result = await next()
    return result
  } catch (error) {
    Sentry.captureException(error, {
      extra: {
        userId: ctx.user?.id,
      },
    })
    throw error
  } finally {
    transaction.finish()
  }
})
```

---

## 测试策略

```typescript
// server/router/__tests__/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest'
import { createCallerFactory, router } from '../index'
import { createTestContext } from './helpers'

describe('User Router', () => {
  const createCaller = createCallerFactory(router)

  it('should get user by id', async () => {
    const ctx = await createTestContext({
      user: { id: 'test-id', email: 'test@example.com' },
    })

    const caller = createCaller(ctx)
    const user = await caller.user.getById({ id: 'test-id' })

    expect(user).toBeDefined()
    expect(user.email).toBe('test@example.com')
  })

  it('should throw NOT_FOUND for invalid id', async () => {
    const ctx = await createTestContext({})
    const caller = createCaller(ctx)

    await expect(
      caller.user.getById({ id: 'non-existent' })
    ).rejects.toThrow('User not found')
  })
})
```

---

## 性能优化

### 批量查询

```typescript
// 使用 httpBatchLink 批量请求
const optimizedRouter = router({
  getUsers: publicProcedure
    .input(z.object({ ids: z.array(z.string()) }))
    .query(async ({ ctx, input }) => {
      return ctx.prisma.user.findMany({
        where: { id: { in: input.ids } },
      })
    }),
})
```

### 缓存层

```typescript
// React 的 cache 函数
import { cache } from 'react'

export const getCachedUser = cache(async (id: string) => {
  return prisma.user.findUnique({ where: { id } })
})
```

---

## tRPC 限制与批评

### 已知的局限性

1. **类型共享限制**：Server 和 Client 必须在同一个 TypeScript 项目中
2. **框架依赖**：虽然可以在任何框架使用，但主要优化是针对 Next.js 和 React
3. **不适合公开 API**：如果需要向第三方提供 API，tRPC 不是好选择
4. **文档生成**：没有自动生成 API 文档的功能

### 何时避免使用 tRPC

- 需要为移动应用或其他非 TypeScript 客户端提供 API
- 需要公开 API 供第三方开发者使用
- 团队不熟悉 TypeScript
- 微服务架构，需要服务间通信

---

## 选型建议

### 何时选择 tRPC

| 场景 | 推荐理由 |
|------|---------|
| **TypeScript 项目** | 原生端到端类型安全 |
| **全栈 TypeScript** | Server/Client 同一项目 |
| **快速开发** | 无需 Schema/代码生成 |
| **AI 应用** | 强类型 LLM 调用 |
| **小中型项目** | 简单直接 |

### 何时考虑其他方案

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| **公开 API** | REST | 标准化文档 |
| **多语言团队** | GraphQL | Schema 跨语言 |
| **微服务** | REST/gRPC | 服务独立 |
| **JavaScript 项目** | REST | 无需 TS |
| **大型团队** | GraphQL | 成熟的生态 |

---

> [!SUCCESS]
> tRPC 以其独特的设计理念——"无 Schema 的端到端类型安全"——为 TypeScript 全栈开发提供了全新的范式。通过直接共享 TypeScript 类型，tRPC 实现了真正的类型安全，同时保持了 RPC 的简洁语义。对于追求开发效率和类型安全的团队，tRPC 是一个值得认真考虑的选择。
