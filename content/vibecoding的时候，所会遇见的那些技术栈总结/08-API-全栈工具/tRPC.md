# tRPC 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 tRPC 核心原理、Router/Procedure 定义、Middleware、Zod 集成及在 AI 应用中的端到端类型安全。

---

## 目录

1. [[#tRPC 概述]]
2. [[#tRPC vs REST vs GraphQL vs REST+types 对比]]
3. [[#核心架构]]
4. [[#Server 定义]]
5. [[#Client 使用]]
6. [[#Middleware 与 Context]]
7. [[#Zod 验证集成]]
8. [[#高级特性]]
9. [[#tRPC 在 AI 应用中的优势]]
10. [[#选型建议]]

---

## tRPC 概述

### 什么是 tRPC

tRPC（TypeScript Remote Procedure Call）是一个用于构建**端到端类型安全 API**的库。tRPC 的核心理念是：在同一个 TypeScript 项目中，Server 和 Client 共享类型定义，无需 Schema 文件或代码生成，实现真正的类型安全。

**tRPC 核心特点：**

| 特性 | 说明 |
|------|------|
| **无 Schema** | 直接使用 TypeScript 类型 |
| **端到端类型安全** | Server/Client 共享类型 |
| **零配置** | 无需代码生成 |
| **高性能** | 直接函数调用，无序列化 |
| **自动补全** | IDE 全类型提示 |
| **验证集成** | 原生支持 Zod/Valibot |

### tRPC 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| tRPC 1.0 | 2020 | 初始版本 |
| tRPC 9.0 | 2022 | v10，稳定版 |
| tRPC 10.0 | 2022 | 完全重写，更好的性能 |
| tRPC 11.0 | **2024** | **中间件改进，Server Components 支持** |

### 核心理念

**tRPC 的设计哲学：**

1. **类型即文档**：TypeScript 类型就是 API 文档
2. **无 Schema 生成**：无需额外的代码生成步骤
3. **直接函数调用**：RPC 风格，语义清晰
4. **运行时安全**：Zod 等验证库确保运行时安全

> [!IMPORTANT]
> tRPC 要求 Server 和 Client 都在同一个 TypeScript 项目中，通过类型导入共享类型。这是 tRPC 的核心约束，也是其零 Schema 方案的基础。

---

## tRPC vs REST vs GraphQL vs REST+types 对比

### 功能对比表

| 特性 | tRPC | REST | GraphQL | REST+types |
|------|------|------|---------|------------|
| **类型安全** | 原生端到端 | 需手动 | Schema 生成 | 手动维护 |
| **Schema 文件** | 无 | OpenAPI | SDL | OpenAPI |
| **代码生成** | 无 | 可选 | 必须 | 可选 |
| **API 调用** | 函数调用 | HTTP 请求 | 查询语言 | HTTP 请求 |
| **Overfetching** | 避免 | 常见 | 避免 | 常见 |
| **性能** | 最快 | 快 | 中等 | 快 |
| **学习曲线** | 低 | 低 | 中等 | 中等 |
| **生态系统** | 增长中 | 成熟 | 成熟 | 成熟 |
| **框架限制** | 需 TS | 无 | 无 | 无 |

### 请求风格对比

**REST API：**

```typescript
// 多个 HTTP 请求
const response = await fetch('/api/users/123');
const user = await response.json();

const postsResponse = await fetch(`/api/users/${user.id}/posts`);
const posts = await postsResponse.json();
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

**REST + TypeScript：**

```typescript
// 需要手动维护类型
interface User {
  id: string;
  name: string;
}

const response = await fetch('/api/users/123');
const user: User = await response.json();
```

**tRPC：**

```typescript
// 直接函数调用，完全类型安全
const user = await trpc.user.getById.query({ id: '123' });
const posts = await trpc.user.posts.query({ userId: '123' });
```

### 架构对比

```
┌─────────────────────────────────────────────────────────────────┐
│                         tRPC 架构                                 │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    TypeScript 项目                        │    │
│  │                                                          │    │
│  │   ┌─────────────────┐       ┌─────────────────────┐     │    │
│  │   │      Server      │◄─────│   共享类型定义       │     │    │
│  │   │   (trpc server) │       │   (TypeScript)      │     │    │
│  │   └────────┬─────────┘       └─────────────────────┘     │    │
│  │            │                                              │    │
│  │            │ HTTP/WS                                      │    │
│  │            ▼                                              │    │
│  │   ┌─────────────────┐                                    │    │
│  │   │      Client      │                                    │    │
│  │   │  (trpc client)  │                                    │    │
│  │   │  + 类型自动推断   │                                    │    │
│  │   └─────────────────┘                                    │    │
│  └─────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                        GraphQL 架构                               │
│                                                                  │
│  ┌─────────────────┐       ┌─────────────────────┐             │
│  │    Client        │───────│    Schema (SDL)     │             │
│  │                  │       │    .graphql 文件     │             │
│  └─────────────────┘       └─────────────────────┘             │
│            │                        ▲                           │
│            │                        │                           │
│            │              ┌─────────────────┐                   │
│            └─────────────►│   代码生成       │                   │
│                           │ (codegen)       │                   │
│                           └─────────────────┘                   │
└─────────────────────────────────────────────────────────────────┘
```

---

## 核心架构

### 项目结构

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
│   │   └── trpc.ts           # tRPC 基础配置
│   │
│   ├── client/
│   │   ├── index.ts          # tRPC Client 初始化
│   │   └── App.tsx           # React 组件
│   │
│   ├── types/
│   │   └── trpc.ts           # 类型导出（Server → Client）
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
import { initTRPC, TRPCError } from '@trpc/server';
import { ZodError } from 'zod';

export const createContext = async () => {
  // 从请求中获取用户（示例）
  const user = await getCurrentUser();

  return {
    user,
    prisma,
  };
};

type Context = Awaited<ReturnType<typeof createContext>>;

const t = initTRPC.context<Context>().create({
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError:
          error.cause instanceof ZodError ? error.cause.flatten() : null,
      },
    };
  },
});

export const router = t.router;
export const publicProcedure = t.procedure;
export const middleware = t.middleware;
export const mergeRouters = t.mergeRouters;
```

---

## Server 定义

### 基础 Router

```typescript
// server/router/user.ts
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';

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
      const limit = input.limit;
      const cursor = input.cursor;

      const users = await ctx.prisma.user.findMany({
        take: limit + 1,
        cursor: cursor ? { id: cursor } : undefined,
        orderBy: { createdAt: 'desc' },
      });

      let nextCursor: string | undefined = undefined;
      if (users.length > limit) {
        const nextItem = users.pop();
        nextCursor = nextItem!.id;
      }

      return {
        users,
        nextCursor,
      };
    }),

  // 查询：获取单个用户
  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.findUnique({
        where: { id: input.id },
      });

      if (!user) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        });
      }

      return user;
    }),

  // 查询：获取当前用户（需认证）
  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user;
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
      });

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'User with this email already exists',
        });
      }

      return ctx.prisma.user.create({
        data: input,
      });
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
      // 确保只能更新自己的信息
      if (input.id !== ctx.user.id) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You can only update your own profile',
        });
      }

      return ctx.prisma.user.update({
        where: { id: input.id },
        data: {
          name: input.name,
          email: input.email,
        },
      });
    }),

  // 变更：删除用户（需认证）
  delete: protectedProcedure
    .input(z.object({ id: z.string() }))
    .mutation(async ({ input, ctx }) => {
      if (input.id !== ctx.user.id) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You can only delete your own account',
        });
      }

      await ctx.prisma.user.delete({ where: { id: input.id } });
      return { success: true };
    }),
});
```

### 嵌套 Router

```typescript
// server/router/index.ts
import { router } from '../trpc';
import { userRouter } from './user';
import { postRouter } from './post';

export const appRouter = router({
  user: userRouter,
  post: postRouter,
});

export type AppRouter = typeof appRouter;
```

### 合并 Router

```typescript
// server/router/chat.ts
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';

export const chatRouter = router({
  // 获取聊天列表
  list: protectedProcedure.query(async ({ ctx }) => {
    return ctx.prisma.chat.findMany({
      where: {
        participants: {
          some: { id: ctx.user.id },
        },
      },
      include: {
        participants: {
          select: { id: true, name: true, avatar: true },
        },
        messages: {
          orderBy: { createdAt: 'desc' },
          take: 1,
        },
      },
      orderBy: { updatedAt: 'desc' },
    });
  }),

  // 获取聊天消息
  messages: protectedProcedure
    .input(z.object({ chatId: z.string() }))
    .query(async ({ input, ctx }) => {
      const chat = await ctx.prisma.chat.findFirst({
        where: {
          id: input.chatId,
          participants: {
            some: { id: ctx.user.id },
          },
        },
      });

      if (!chat) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Chat not found',
        });
      }

      return ctx.prisma.message.findMany({
        where: { chatId: input.chatId },
        orderBy: { createdAt: 'asc' },
        include: {
          sender: {
            select: { id: true, name: true, avatar: true },
          },
        },
      });
    }),

  // 发送消息
  sendMessage: protectedProcedure
    .input(
      z.object({
        chatId: z.string(),
        content: z.string().min(1).max(5000),
      })
    )
    .mutation(async ({ input, ctx }) => {
      const chat = await ctx.prisma.chat.findFirst({
        where: {
          id: input.chatId,
          participants: {
            some: { id: ctx.user.id },
          },
        },
      });

      if (!chat) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Chat not found',
        });
      }

      return ctx.prisma.message.create({
        data: {
          chatId: input.chatId,
          senderId: ctx.user.id,
          content: input.content,
        },
        include: {
          sender: {
            select: { id: true, name: true, avatar: true },
          },
        },
      });
    }),
});

// 合并到根 Router
export const appRouter = mergeRouters(appRouter, chatRouter);
```

---

## Client 使用

### 基础 Client 配置

```typescript
// client/index.ts
import { createTRPCReact, httpBatchLink, loggerLink } from '@trpc/react-query';
import type { AppRouter } from '../../server/router';

export const trpc = createTRPCReact<AppRouter>();
```

### React Query 集成

```typescript
// client/trpc.ts
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { httpBatchLink } from '@trpc/react-query';
import { useState } from 'react';
import { trpc } from './index';

function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: '/api/trpc',
          headers() {
            return {
              Authorization: `Bearer ${getAuthToken()}`,
            };
          },
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```

### 组件中使用

```typescript
// client/components/UserList.tsx
import { trpc } from '../trpc';

function UserList() {
  // 自动类型推断，无需额外类型定义
  const { data, isLoading, error } = trpc.user.list.useQuery({
    limit: 20,
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data?.users.map((user) => (
        <li key={user.id}>
          {user.name} ({user.email})
        </li>
      ))}
    </ul>
  );
}

// 创建用户
function CreateUserForm() {
  const trpcContext = trpc.useContext();
  const createUser = trpc.user.create.useMutation({
    onSuccess: () => {
      trpcContext.user.list.invalidate();
      toast.success('User created!');
    },
  });

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault();
    const formData = new FormData(e.target);

    createUser.mutate({
      name: formData.get('name') as string,
      email: formData.get('email') as string,
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={createUser.isPending}>
        {createUser.isPending ? 'Creating...' : 'Create'}
      </button>
    </form>
  );
}
```

### React Server Components

```typescript
// app/user/[id]/page.tsx
import { getServerAuthSession } from '@/server/auth';
import { trpcServer } from '@/lib/trpc';

export default async function UserPage({ params }: { params: { id: string } }) {
  // 直接在服务端调用，无需 API 路由
  const user = await trpcServer.user.getById.query({ id: params.id });

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
```

---

## Middleware 与 Context

### 中间件定义

```typescript
// server/trpc.ts
// 认证中间件
const isAuthenticated = middleware(({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: 'You must be logged in',
    });
  }
  return next({
    ctx: {
      ...ctx,
      user: ctx.user, // 确保 user 存在
    },
  });
});

// 管理员中间件
const isAdmin = middleware(({ ctx, next }) => {
  if (!ctx.user || ctx.user.role !== 'ADMIN') {
    throw new TRPCError({
      code: 'FORBIDDEN',
      message: 'Admin access required',
    });
  }
  return next({ ctx });
});

// 速率限制中间件
import { ratelimit } from '@/lib/ratelimit';

const ratelimitMiddleware = middleware(async ({ ctx, next }) => {
  const identifier = ctx.user?.id || ctx.ip;

  const { success, remaining } = await ratelimit.limit(identifier);

  if (!success) {
    throw new TRPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
    });
  }

  return next();
});

// 审计日志中间件
const auditLogMiddleware = middleware(async ({ ctx, next, path, type, input }) => {
  const start = Date.now();
  const result = await next();
  const duration = Date.now() - start;

  await ctx.prisma.auditLog.create({
    data: {
      userId: ctx.user?.id,
      action: path,
      type,
      duration,
      input,
    },
  });

  return result;
});

// 创建受保护的 Procedure
export const protectedProcedure = publicProcedure.use(auditLogMiddleware).use(isAuthenticated);
export const adminProcedure = protectedProcedure.use(isAdmin);
export const rateLimitedProcedure = publicProcedure.use(ratelimitMiddleware);
```

### 使用中间件

```typescript
// server/router/user.ts
export const userRouter = router({
  // 公开查询
  list: publicProcedure.query(...),

  // 需要认证
  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user;
  }),

  // 需要管理员权限
  adminList: adminProcedure.query(async ({ ctx }) => {
    return ctx.prisma.user.findMany();
  }),

  // 速率限制
  create: rateLimitedProcedure
    .input(...)
    .mutation(async ({ input, ctx }) => {
      return ctx.prisma.user.create({ data: input });
    }),
});
```

### Context 扩展

```typescript
// server/context.ts
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/prisma';

export const createContext = async (opts?: { headers: Headers }) => {
  const session = await getServerSession();

  return {
    user: session?.user
      ? { ...session.user, id: session.user.id }
      : null,
    prisma,
    headers: opts?.headers,
  };
};

type Context = Awaited<ReturnType<typeof createContext>>;
```

---

## Zod 验证集成

### Zod 基础

```typescript
import { z } from 'zod';

// 基本类型
z.string();
z.number();
z.boolean();
z.date();
z.array(z.string());
z.object({ name: z.string(), age: z.number() });

// 链式验证
z.string().min(1).max(100);
z.string().email();
z.string().url();
z.number().min(0).max(100);

// 可选和默认值
z.string().optional();
z.string().default('default');
z.number().nullable();

// 复杂结构
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().min(0).max(150).optional(),
  role: z.enum(['ADMIN', 'USER', 'GUEST']),
  createdAt: z.coerce.date(),
  metadata: z.record(z.string(), z.unknown()),
});

// 解析
UserSchema.parse({...});        // 成功返回数据，失败抛出错误
UserSchema.safeParse({...});    // 返回 { success: true, data } 或 { success: false, error }
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
    console.log(input.age); // number | undefined
    console.log(input.tags); // string[]
  });

// 输出验证
publicProcedure
  .output(
    z.object({
      id: z.string(),
      name: z.string(),
    })
  )
  .query(async ({ ctx }) => {
    const users = await ctx.prisma.user.findMany();
    return users;
  });
```

### 自定义验证

```typescript
import { z } from 'zod';

// 自定义验证器
const customString = z.string().refine(
  (val) => !val.includes('admin'),
  { message: 'Username cannot contain "admin"' }
);

// 异步验证
const uniqueEmail = z.string().email().refine(
  async (email) => {
    const existing = await db.user.findUnique({ where: { email } });
    return !existing;
  },
  { message: 'Email already in use' }
);

// 变换
const sanitizeInput = z.string().transform((val) => val.trim().toLowerCase());

// 预处理
const queryInput = z.object({
  page: z.coerce.number().default(1),
  limit: z.coerce.number().min(1).max(100).default(20),
  sort: z.enum(['asc', 'desc']).default('desc'),
});
```

---

## 高级特性

### Subscription（实时）

```typescript
// server/router/chat.ts
import { router, protectedProcedure } from '../trpc';
import { observable } from '@trpc/server/observable';
import { EventEmitter } from 'events';

const ee = new EventEmitter();

export const chatRouter = router({
  // 发送消息
  sendMessage: protectedProcedure
    .input(z.object({ content: z.string() }))
    .mutation(async ({ input, ctx }) => {
      const message = await ctx.prisma.message.create({
        data: { ...input, senderId: ctx.user.id },
      });

      // 广播事件
      ee.emit('message', message);

      return message;
    }),

  // 订阅新消息
  onMessage: protectedProcedure.subscription(({ ctx }) => {
    return observable<Message>((emit) => {
      const onMessage = (message: Message) => {
        emit.next(message);
      };

      ee.on('message', onMessage);

      return () => {
        ee.off('message', onMessage);
      };
    });
  }),
});
```

### 文件上传

```typescript
// server/router/upload.ts
import { router, protectedProcedure } from '../trpc';
import { z } from 'zod';

export const uploadRouter = router({
  // 上传文件
  upload: protectedProcedure
    .input(
      z.object({
        file: z.string(), // Base64 或 URL
        filename: z.string(),
        mimeType: z.string(),
      })
    )
    .mutation(async ({ input }) => {
      const uploaded = await uploadToS3({
        file: input.file,
        filename: input.filename,
        mimeType: input.mimeType,
      });

      return { url: uploaded.url };
    }),
});
```

### 错误处理

```typescript
// 抛出错误
throw new TRPCError({
  code: 'NOT_FOUND',
  message: 'User not found',
  cause: error,
});

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
};
```

---

## tRPC 在 AI 应用中的优势

### 强类型 LLM 调用

```typescript
// AI Prompt 定义（完全类型安全）
const promptRouter = router({
  // 定义 Prompt Schema
  generateDescription: protectedProcedure
    .input(
      z.object({
        productName: z.string().min(1).max(200),
        category: z.enum(['ELECTRONICS', 'FASHION', 'FOOD', 'OTHER']),
        targetAudience: z.string().optional(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      const systemPrompt = `You are a professional product description writer.`;

      const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [
          { role: 'system', content: systemPrompt },
          {
            role: 'user',
            content: `Write a compelling product description for: ${input.productName}
Category: ${input.category}
Target Audience: ${input.targetAudience || 'General'}`
          }
        ],
        temperature: 0.7,
      });

      return {
        description: response.choices[0].message.content,
        usage: response.usage,
      };
    }),
});

// 客户端调用（完全类型推断）
function ProductDescriptionGenerator() {
  const generateMutation = trpc.prompt.generateDescription.useMutation();

  const handleGenerate = () => {
    // input 完全类型安全
    generateMutation.mutate({
      productName: 'Wireless Headphones',
      category: 'ELECTRONICS',
      targetAudience: 'Young professionals',
    });
  };

  return (
    <div>
      <button onClick={handleGenerate}>
        Generate
      </button>
      {generateMutation.data && (
        <p>{generateMutation.data.description}</p>
      )}
    </div>
  );
}
```

### RAG 集成

```typescript
// RAG 查询 Router
const ragRouter = router({
  // 语义搜索
  search: protectedProcedure
    .input(
      z.object({
        query: z.string().min(1).max(1000),
        topK: z.number().min(1).max(20).default(5),
        filters: z
          .object({
            source: z.string().optional(),
            dateRange: z
              .object({
                from: z.date(),
                to: z.date(),
              })
              .optional(),
          })
          .optional(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      // 向量搜索
      const results = await ctx.vectorStore.similaritySearch({
        query: input.query,
        topK: input.topK,
        filter: input.filters,
      });

      // 生成回答
      const context = results.map((r) => r.content).join('\n\n');

      const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [
          {
            role: 'system',
            content: `Based on the following context, answer the user's question.\n\nContext:\n${context}`
          },
          { role: 'user', content: input.query }
        ],
      });

      return {
        answer: response.choices[0].message.content,
        sources: results.map((r) => ({
          id: r.id,
          content: r.content,
          score: r.score,
        })),
      };
    }),
});
```

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

> [!TIP]
> tRPC 非常适合全栈 TypeScript 项目，特别是 AI 应用开发。其强类型特性使得与 LLM 的集成更加安全可靠。

---

## tRPC 高级配置与最佳实践

### 数据库连接池

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// 单例模式避免开发环境连接耗尽
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' 
    ? ['query', 'error', 'warn']
    : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### 事务与原子操作

```typescript
// server/router/order.ts
import { router, protectedProcedure } from '../trpc';
import { z } from 'zod';
import { TRPCError } from '@trpc/server';

export const orderRouter = router({
  createOrder: protectedProcedure
    .input(z.object({
      items: z.array(z.object({
        productId: z.string(),
        quantity: z.number().min(1),
      })),
    }))
    .mutation(async ({ ctx, input }) => {
      const { prisma } = ctx;
      
      // 事务：确保所有操作原子性完成
      return prisma.$transaction(async (tx) => {
        // 1. 验证库存
        for (const item of input.items) {
          const product = await tx.product.findUnique({
            where: { id: item.productId },
          });
          
          if (!product) {
            throw new TRPCError({
              code: 'NOT_FOUND',
              message: `Product ${item.productId} not found`,
            });
          }
          
          if (product.stock < item.quantity) {
            throw new TRPCError({
              code: 'BAD_REQUEST',
              message: `Insufficient stock for ${product.name}`,
            });
          }
        }
        
        // 2. 创建订单
        const order = await tx.order.create({
          data: {
            userId: ctx.user.id,
            status: 'PENDING',
            items: {
              create: input.items.map((item) => ({
                productId: item.productId,
                quantity: item.quantity,
              })),
            },
          },
        });
        
        // 3. 扣减库存
        for (const item of input.items) {
          await tx.product.update({
            where: { id: item.productId },
            data: {
              stock: { decrement: item.quantity },
            },
          });
        }
        
        return order;
      });
    }),
});
```

### 分页与游标

```typescript
// 游标分页实现
export function createPaginatedRouter<T extends { id: string }>(
  router: ReturnType<typeof router>,
  name: string,
  config: {
    model: any;
    where?: any;
    orderBy?: any;
  }
) {
  router[name] = router.extend({
    list: protectedProcedure
      .input(
        z.object({
          first: z.number().min(1).max(100).optional(),
          last: z.number().min(1).max(100).optional(),
          before: z.string().optional(),
          after: z.string().optional(),
          filter: z.record(z.string(), z.unknown()).optional(),
        })
      )
      .query(async ({ input, ctx }) => {
        const { model, where: baseWhere, orderBy: baseOrderBy } = config;
        
        const cursor = input.after
          ? { id: input.after }
          : input.before
          ? { id: input.before }
          : undefined;
        
        const take = input.first ?? input.last ?? 20;
        const skip = cursor ? 1 : undefined;
        const takeAbs = Math.abs(take);
        
        const orderBy = baseOrderBy || { createdAt: 'desc' as const };
        
        const items = await model.findMany({
          where: {
            ...baseWhere,
            ...input.filter,
          },
          orderBy,
          take: input.last ? -takeAbs : takeAbs,
          skip,
          cursor,
        });
        
        // 处理反向分页
        const sortedItems = input.last ? items.reverse() : items;
        
        return {
          edges: sortedItems.map((item: T) => ({
            node: item,
            cursor: item.id,
          })),
          pageInfo: {
            hasNextPage: input.first
              ? items.length > input.first!
              : false,
            hasPreviousPage: input.last
              ? items.length > input.last!
              : false,
            startCursor: sortedItems[0]?.id,
            endCursor: sortedItems[sortedItems.length - 1]?.id,
          },
        };
      }),
  });
}
```

### tRPC 与 Next.js App Router

```typescript
// lib/trpc/server.ts - 服务端 tRPC 客户端
import { createTRPCProxyClient, httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@/server/router';

function getBaseUrl() {
  if (typeof window !== 'undefined') return '';
  return `http://localhost:${process.env.PORT ?? 3000}`;
}

export const trpcServer = createTRPCProxyClient<AppRouter>({
  links: [
    httpBatchLink({
      url: `${getBaseUrl()}/api/trpc`,
      headers() {
        return {};
      },
    }),
  ],
});

// app/user/[id]/page.tsx
import { notFound } from 'next/navigation';
import { trpcServer } from '@/lib/trpc/server';

export default async function UserPage({
  params,
}: {
  params: { id: string };
}) {
  // 服务端直接调用，完全类型安全
  const user = await trpcServer.user.getById.query({ id: params.id });
  
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
```

### tRPC 与 tRPC React Query 集成

```typescript
// hooks/useTRPC.ts - 自定义 hooks
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { trpc } from '@/lib/trpc/client';
import { useCallback } from 'react';

export function useUser(id: string) {
  return useQuery(trpc.user.getById.queryOptions({ id }));
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  const mutation = trpc.user.create.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(trpc.user.list.queryOptions());
    },
  });

  return {
    createUser: mutation.mutate,
    isLoading: mutation.isPending,
    error: mutation.error,
  };
}

export function useUsers() {
  return useQuery(trpc.user.list.queryOptions({}));
}
```

> [!SUCCESS]
> tRPC 以其独特的设计理念——"无 Schema 的端到端类型安全"——为 TypeScript 全栈开发提供了全新的范式。通过直接共享 TypeScript 类型，tRPC 实现了真正的类型安全，同时保持了 RPC 的简洁语义。对于追求开发效率和类型安全的团队，tRPC 是一个值得认真考虑的选择。

---

## tRPC 高级配置与最佳实践

### 数据库连接池

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

// 单例模式避免开发环境连接耗尽
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: process.env.NODE_ENV === 'development' 
    ? ['query', 'error', 'warn']
    : ['error'],
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### 事务与原子操作

```typescript
// server/router/order.ts
import { router, protectedProcedure } from '../trpc';
import { z } from 'zod';
import { TRPCError } from '@trpc/server';

export const orderRouter = router({
  createOrder: protectedProcedure
    .input(z.object({
      items: z.array(z.object({
        productId: z.string(),
        quantity: z.number().min(1),
      })),
    }))
    .mutation(async ({ ctx, input }) => {
      const { prisma } = ctx;
      
      // 事务：确保所有操作原子性完成
      return prisma.$transaction(async (tx) => {
        // 1. 验证库存
        for (const item of input.items) {
          const product = await tx.product.findUnique({
            where: { id: item.productId },
          });
          
          if (!product) {
            throw new TRPCError({
              code: 'NOT_FOUND',
              message: `Product ${item.productId} not found`,
            });
          }
          
          if (product.stock < item.quantity) {
            throw new TRPCError({
              code: 'BAD_REQUEST',
              message: `Insufficient stock for ${product.name}`,
            });
          }
        }
        
        // 2. 创建订单
        const order = await tx.order.create({
          data: {
            userId: ctx.user.id,
            status: 'PENDING',
            items: {
              create: input.items.map((item) => ({
                productId: item.productId,
                quantity: item.quantity,
              })),
            },
          },
        });
        
        // 3. 扣减库存
        for (const item of input.items) {
          await tx.product.update({
            where: { id: item.productId },
            data: {
              stock: { decrement: item.quantity },
            },
          });
        }
        
        return order;
      });
    }),
});
```

### 分页与游标

```typescript
// 游标分页实现
export function createPaginatedRouter<T extends { id: string }>(
  router: ReturnType<typeof router>,
  name: string,
  config: {
    model: any;
    where?: any;
    orderBy?: any;
  }
) {
  router[name] = router.extend({
    list: protectedProcedure
      .input(
        z.object({
          first: z.number().min(1).max(100).optional(),
          last: z.number().min(1).max(100).optional(),
          before: z.string().optional(),
          after: z.string().optional(),
          filter: z.record(z.string(), z.unknown()).optional(),
        })
      )
      .query(async ({ input, ctx }) => {
        const { model, where: baseWhere, orderBy: baseOrderBy } = config;
        
        const cursor = input.after
          ? { id: input.after }
          : input.before
          ? { id: input.before }
          : undefined;
        
        const take = input.first ?? input.last ?? 20;
        const skip = cursor ? 1 : undefined;
        const takeAbs = Math.abs(take);
        
        const orderBy = baseOrderBy || { createdAt: 'desc' as const };
        
        const items = await model.findMany({
          where: {
            ...baseWhere,
            ...input.filter,
          },
          orderBy,
          take: input.last ? -takeAbs : takeAbs,
          skip,
          cursor,
        });
        
        // 处理反向分页
        const sortedItems = input.last ? items.reverse() : items;
        
        return {
          edges: sortedItems.map((item: T) => ({
            node: item,
            cursor: item.id,
          })),
          pageInfo: {
            hasNextPage: input.first
              ? items.length > input.first!
              : false,
            hasPreviousPage: input.last
              ? items.length > input.last!
              : false,
            startCursor: sortedItems[0]?.id,
            endCursor: sortedItems[sortedItems.length - 1]?.id,
          },
        };
      }),
  });
}
```

### tRPC 与 Next.js App Router

```typescript
// lib/trpc/server.ts - 服务端 tRPC 客户端
import { createTRPCProxyClient, httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@/server/router';

function getBaseUrl() {
  if (typeof window !== 'undefined') return '';
  return `http://localhost:${process.env.PORT ?? 3000}`;
}

export const trpcServer = createTRPCProxyClient<AppRouter>({
  links: [
    httpBatchLink({
      url: `${getBaseUrl()}/api/trpc`,
      headers() {
        return {};
      },
    }),
  ],
});

// app/user/[id]/page.tsx
import { notFound } from 'next/navigation';
import { trpcServer } from '@/lib/trpc/server';

export default async function UserPage({
  params,
}: {
  params: { id: string };
}) {
  // 服务端直接调用，完全类型安全
  const user = await trpcServer.user.getById.query({ id: params.id });
  
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
```

### tRPC 与 tRPC React Query 集成

```typescript
// hooks/useTRPC.ts - 自定义 hooks
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { trpc } from '@/lib/trpc/client';
import { useCallback } from 'react';

export function useUser(id: string) {
  return useQuery(trpc.user.getById.queryOptions({ id }));
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  const mutation = trpc.user.create.useMutation({
    onSuccess: () => {
      queryClient.invalidateQueries(trpc.user.list.queryOptions());
    },
  });

  return {
    createUser: mutation.mutate,
    isLoading: mutation.isPending,
    error: mutation.error,
  };
}

export function useUsers() {
  return useQuery(trpc.user.list.queryOptions({}));
}
```

### tRPC 与 GraphQL Federation

```typescript
// server/router/index.ts
import { router, publicProcedure, protectedProcedure } from '../trpc';
import { userRouter } from './user';
import { postRouter } from './post';

// 组合多个 Router
export const appRouter = router({
  user: userRouter,
  post: postRouter,
});

// 将 tRPC 作为 GraphQL 子图暴露
export const graphqlRouter = router({
  // GraphQL 风格的查询端点
  graphqlQuery: publicProcedure
    .input(
      z.object({
        query: z.string(),
        variables: z.record(z.unknown()).optional(),
      })
    )
    .mutation(async ({ ctx, input }) => {
      // 解析 GraphQL 查询并转换为 tRPC 调用
      return executeGraphQLQuery(ctx, input.query, input.variables);
    }),
});

export type AppRouter = typeof appRouter;
```

### tRPC 性能优化

```typescript
// 1. 批量查询优化
const optimizedRouter = router({
  // 使用 httpBatchLink 批量请求
  getUsers: publicProcedure
    .input(z.object({ ids: z.array(z.string()) }))
    .query(async ({ ctx, input }) => {
      // 一次性获取所有用户
      return ctx.prisma.user.findMany({
        where: { id: { in: input.ids } },
      });
    }),
});

// 2. 缓存层
import { cache } from 'react';

export const getCachedUser = cache(async (id: string) => {
  return prisma.user.findUnique({ where: { id } });
});

// 3. 数据加载预取
export const prefetchUserData = async (
  queryClient: QueryClient,
  userId: string
) => {
  await Promise.all([
    queryClient.prefetchQuery(
      trpc.user.getById.queryOptions({ id: userId })
    ),
    queryClient.prefetchQuery(
      trpc.user.posts.queryOptions({ userId })
    ),
  ]);
};

// 4. 无限查询优化
export function useInfiniteQuery_example() {
  return trpc.post.list.useInfiniteQuery(
    { limit: 10 },
    {
      getNextPageParam: (lastPage) => lastPage.nextCursor,
    }
  );
}
```

### tRPC 与微服务通信

```typescript
// server/router/microservice.ts
import { router, publicProcedure } from '../trpc';
import { z } from 'zod';
import { TRPCError } from '@trpc/server';

export const microserviceRouter = router({
  // 调用其他微服务
  callUserService: publicProcedure
    .input(z.object({
      userId: z.string(),
      action: z.enum(['get', 'update', 'delete']),
    }))
    .mutation(async ({ input }) => {
      const { userId, action } = input;
      
      // 使用 gRPC 或 HTTP 调用其他服务
      const response = await callService('user-service', {
        action,
        userId,
      });
      
      return response;
    }),
    
  // 服务间通信（内部调用）
  internalCall: protectedProcedure
    .input(z.object({
      service: z.string(),
      method: z.string(),
      data: z.unknown(),
    }))
    .mutation(async ({ ctx, input }) => {
      // 验证内部调用
      if (input.service === 'payment-service') {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'Payment must go through payment gateway',
        });
      }
      
      return callInternalService(input.service, input.method, input.data);
    }),
});

// 服务调用辅助函数
async function callService(
  serviceName: string,
  payload: unknown,
  timeout = 5000
) {
  const response = await fetch(
    `${process.env.SERVICES_URL}/${serviceName}`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-Internal-Service': 'true',
        'X-Service-Token': process.env.INTERNAL_SERVICE_TOKEN!,
      },
      body: JSON.stringify(payload),
      signal: AbortSignal.timeout(timeout),
    }
  );
  
  if (!response.ok) {
    throw new TRPCError({
      code: 'INTERNAL_SERVER_ERROR',
      message: `Service ${serviceName} call failed`,
    });
  }
  
  return response.json();
}
```

### tRPC 错误处理与监控

```typescript
// 错误格式化
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
    };
  },
});

// 全局错误处理中间件
const errorMiddleware = t.middleware(async ({ ctx, next, path }) => {
  try {
    return await next();
  } catch (error) {
    // 记录错误
    if (error instanceof TRPCError) {
      logger.error('TRPC Error', {
        code: error.code,
        message: error.message,
        path,
        userId: ctx.user?.id,
      });
    } else {
      logger.error('Unexpected Error', {
        error: error instanceof Error ? error.message : 'Unknown error',
        stack: error instanceof Error ? error.stack : undefined,
        path,
      });
    }
    
    throw error;
  }
});

// Sentry 集成
import * as Sentry from '@sentry/nextjs';

const sentryMiddleware = t.middleware(async ({ ctx, next }) => {
  const transaction = Sentry.startTransaction({
    op: 'tRPC',
    name: ctx.operationName,
  });
  
  try {
    const result = await next();
    return result;
  } catch (error) {
    Sentry.captureException(error, {
      extra: {
        userId: ctx.user?.id,
      },
    });
    throw error;
  } finally {
    transaction.finish();
  }
});
```

### tRPC 测试策略

```typescript
// server/router/__tests__/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { createCallerFactory, router } from '../index';
import { createTestContext } from './helpers';

describe('User Router', () => {
  const createCaller = createCallerFactory(router);
  
  it('should get user by id', async () => {
    const ctx = await createTestContext({
      user: { id: 'test-id', email: 'test@example.com' },
    });
    
    const caller = createCaller(ctx);
    
    const user = await caller.user.getById({ id: 'test-id' });
    
    expect(user).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });
  
  it('should throw NOT_FOUND for invalid id', async () => {
    const ctx = await createTestContext({});
    const caller = createCaller(ctx);
    
    await expect(
      caller.user.getById({ id: 'non-existent' })
    ).rejects.toThrow('User not found');
  });
  
  it('should create user with valid data', async () => {
    const ctx = await createTestContext({});
    const caller = createCaller(ctx);
    
    const user = await caller.user.create({
      name: 'Test User',
      email: 'test@example.com',
    });
    
    expect(user.id).toBeDefined();
    expect(user.email).toBe('test@example.com');
  });
});

// 测试助手
async function createTestContext(overrides = {}) {
  return {
    user: null,
    prisma: createMockPrisma(),
    ...overrides,
  };
}
```

---

> [!TIP]
> tRPC 的强大之处在于端到端的类型安全和简洁的 API 设计。合理使用中间件和 Procedure 变体可以让代码更加清晰和可维护。

---

## tRPC 完整项目结构

### Next.js App Router 集成

```
my-app/
├── src/
│   ├── app/
│   │   ├── layout.tsx                 # Root Layout
│   │   ├── page.tsx                   # 首页
│   │   ├── (auth)/
│   │   │   ├── sign-in/
│   │   │   │   └── page.tsx
│   │   │   └── sign-up/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   ├── users/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   └── api/
│   │       └── trpc/
│   │           └── [trpc]/
│   │               └── route.ts
│   │
│   ├── server/
│   │   ├── router/
│   │   │   ├── index.ts           # Root Router
│   │   │   ├── user.ts           # User Router
│   │   │   ├── post.ts           # Post Router
│   │   │   ├── auth.ts           # Auth Router
│   │   │   └── _app.ts           # App Router
│   │   ├── trpc.ts               # tRPC 初始化
│   │   └── context.ts            # Context
│   │
│   ├── lib/
│   │   ├── trpc/
│   │   │   ├── client.ts         # Client 配置
│   │   │   ├── server.ts        # Server 配置
│   │   │   └── provider.tsx     # React Provider
│   │   ├── prisma.ts            # Prisma Client
│   │   └── auth.ts              # Auth 辅助函数
│   │
│   └── components/
│       ├── ui/                   # UI 组件
│       ├── users/                # 用户相关组件
│       └── posts/                # 文章相关组件
│
├── prisma/
│   └── schema.prisma
│
└── package.json
```

### 完整 Router 实现

```typescript
// server/router/user.ts
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';
import { TRPCError } from '@trpc/server';
import { TRPCErrorCodeMap } from '@/lib/constants';

export const userRouter = router({
  // 获取用户列表（公开）
  list: publicProcedure
    .input(
      z.object({
        page: z.number().min(1).default(1),
        limit: z.number().min(1).max(100).default(20),
        search: z.string().optional(),
        role: z.enum(['USER', 'ADMIN', 'MODERATOR']).optional(),
      })
    )
    .query(async ({ input, ctx }) => {
      const { page, limit, search, role } = input;
      const skip = (page - 1) * limit;

      const where: any = {};
      if (search) {
        where.OR = [
          { name: { contains: search, mode: 'insensitive' } },
          { email: { contains: search, mode: 'insensitive' } },
        ];
      }
      if (role) {
        where.role = role;
      }

      const [users, total] = await Promise.all([
        ctx.prisma.user.findMany({
          where,
          skip,
          take: limit,
          orderBy: { createdAt: 'desc' },
          select: {
            id: true,
            name: true,
            email: true,
            role: true,
            image: true,
            createdAt: true,
            _count: { select: { posts: true } },
          },
        }),
        ctx.prisma.user.count({ where }),
      ]);

      return {
        users,
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
        },
      };
    }),

  // 获取单个用户（公开）
  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.findUnique({
        where: { id: input.id },
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          image: true,
          bio: true,
          createdAt: true,
          _count: { select: { posts: true, followers: true, following: true } },
        },
      });

      if (!user) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        });
      }

      return user;
    }),

  // 获取当前用户（需认证）
  me: protectedProcedure.query(async ({ ctx }) => {
    return ctx.user;
  }),

  // 更新当前用户（需认证）
  updateMe: protectedProcedure
    .input(
      z.object({
        name: z.string().min(1).max(100).optional(),
        bio: z.string().max(500).optional(),
        image: z.string().url().optional(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.update({
        where: { id: ctx.user.id },
        data: input,
        select: {
          id: true,
          name: true,
          email: true,
          bio: true,
          image: true,
        },
      });

      return user;
    }),

  // 创建用户（公开 - 注册）
  create: publicProcedure
    .input(
      z.object({
        email: z.string().email(),
        password: z.string().min(8).max(100),
        name: z.string().min(1).max(100),
      })
    )
    .mutation(async ({ input, ctx }) => {
      // 检查邮箱唯一性
      const existing = await ctx.prisma.user.findUnique({
        where: { email: input.email },
      });

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'Email already registered',
        });
      }

      // 密码加密
      const hashedPassword = await hashPassword(input.password);

      const user = await ctx.prisma.user.create({
        data: {
          email: input.email,
          name: input.name,
          password: hashedPassword,
        },
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true,
        },
      });

      return user;
    }),

  // 删除用户（需认证）
  delete: protectedProcedure
    .input(z.object({ id: z.string() }))
    .mutation(async ({ input, ctx }) => {
      // 只能删除自己，除非是管理员
      if (input.id !== ctx.user.id && ctx.user.role !== 'ADMIN') {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'You can only delete your own account',
        });
      }

      await ctx.prisma.user.delete({
        where: { id: input.id },
      });

      return { success: true };
    }),

  // 关注用户（需认证）
  follow: protectedProcedure
    .input(z.object({ userId: z.string() }))
    .mutation(async ({ input, ctx }) => {
      if (input.userId === ctx.user.id) {
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'You cannot follow yourself',
        });
      }

      // 检查目标用户是否存在
      const targetUser = await ctx.prisma.user.findUnique({
        where: { id: input.userId },
      });

      if (!targetUser) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'User not found',
        });
      }

      // 创建关注关系
      await ctx.prisma.user.update({
        where: { id: ctx.user.id },
        data: {
          following: { connect: { id: input.userId } },
        },
      });

      return { success: true };
    }),

  // 取消关注用户（需认证）
  unfollow: protectedProcedure
    .input(z.object({ userId: z.string() }))
    .mutation(async ({ input, ctx }) => {
      await ctx.prisma.user.update({
        where: { id: ctx.user.id },
        data: {
          following: { disconnect: { id: input.userId } },
        },
      });

      return { success: true };
    }),

  // 获取用户动态（需认证）
  feed: protectedProcedure
    .input(
      z.object({
        page: z.number().min(1).default(1),
        limit: z.number().min(1).max(50).default(20),
      })
    )
    .query(async ({ input, ctx }) => {
      const { page, limit } = input;
      const skip = (page - 1) * limit;

      // 获取关注用户的 ID
      const following = await ctx.prisma.user.findUnique({
        where: { id: ctx.user.id },
        select: { following: { select: { id: true } } },
      });

      const followingIds = following?.following.map((f) => f.id) || [];

      const [posts, total] = await Promise.all([
        ctx.prisma.post.findMany({
          where: {
            authorId: { in: followingIds },
            published: true,
          },
          skip,
          take: limit,
          orderBy: { createdAt: 'desc' },
          include: {
            author: {
              select: { id: true, name: true, image: true },
            },
            _count: { select: { comments: true, likes: true } },
          },
        }),
        ctx.prisma.post.count({
          where: {
            authorId: { in: followingIds },
            published: true,
          },
        }),
      ]);

      return {
        posts,
        pagination: {
          page,
          limit,
          total,
          totalPages: Math.ceil(total / limit),
        },
      };
    }),
});
```

### 认证 Router

```typescript
// server/router/auth.ts
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';
import { TRPCError } from '@trpc/server';

export const authRouter = router({
  // 登录
  login: publicProcedure
    .input(
      z.object({
        email: z.string().email(),
        password: z.string(),
      })
    )
    .mutation(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.findUnique({
        where: { email: input.email },
      });

      if (!user) {
        throw new TRPCError({
          code: 'UNAUTHORIZED',
          message: 'Invalid credentials',
        });
      }

      const isValid = await verifyPassword(input.password, user.password);

      if (!isValid) {
        throw new TRPCError({
          code: 'UNAUTHORIZED',
          message: 'Invalid credentials',
        });
      }

      // 生成 JWT
      const token = generateToken({
        userId: user.id,
        email: user.email,
        role: user.role,
      });

      return {
        token,
        user: {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        },
      };
    }),

  // 注册
  register: publicProcedure
    .input(
      z.object({
        email: z.string().email(),
        password: z.string().min(8),
        name: z.string().min(1).max(100),
      })
    )
    .mutation(async ({ input, ctx }) => {
      // 检查邮箱唯一性
      const existing = await ctx.prisma.user.findUnique({
        where: { email: input.email },
      });

      if (existing) {
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'Email already registered',
        });
      }

      // 创建用户
      const hashedPassword = await hashPassword(input.password);
      const user = await ctx.prisma.user.create({
        data: {
          email: input.email,
          password: hashedPassword,
          name: input.name,
        },
      });

      // 生成 JWT
      const token = generateToken({
        userId: user.id,
        email: user.email,
        role: user.role,
      });

      return {
        token,
        user: {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        },
      };
    }),

  // 获取会话
  getSession: protectedProcedure.query(async ({ ctx }) => {
    return {
      user: ctx.user,
    };
  }),

  // 刷新 Token
  refreshToken: protectedProcedure.mutation(async ({ ctx }) => {
    const token = generateToken({
      userId: ctx.user.id,
      email: ctx.user.email,
      role: ctx.user.role,
    });

    return { token };
  }),
});
```

---

## tRPC 高级中间件模式

### 审计日志中间件

```typescript
// server/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server';
import { ZodError } from 'zod';
import { prisma } from '@/lib/prisma';

// 审计日志中间件
const auditLogMiddleware = t.middleware(async ({ ctx, next, path, type, input }) => {
  const start = Date.now();
  const requestId = crypto.randomUUID();

  // 执行操作
  const result = await next({
    ctx: {
      ...ctx,
      requestId,
    },
  });

  // 计算执行时间
  const duration = Date.now() - start;

  // 异步记录日志（不影响响应时间）
  setImmediate(async () => {
    try {
      await prisma.auditLog.create({
        data: {
          userId: ctx.user?.id,
          action: path,
          type,
          duration,
          input: sanitizeInput(input),
          userAgent: ctx.headers?.['user-agent'],
          ip: ctx.ip,
        },
      });
    } catch (error) {
      console.error('Failed to create audit log:', error);
    }
  });

  return result;
});

// 缓存中间件
const cacheMiddleware = t.middleware(async ({ ctx, next, path, type }) => {
  // 只缓存查询
  if (type !== 'query') {
    return next({ ctx });
  }

  const cacheKey = `t rpc:${path}:${JSON.stringify(ctx._input || {})}`;

  // 尝试从缓存获取
  const cached = await ctx.redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }

  // 执行查询
  const result = await next({ ctx });

  // 缓存结果
  const ttl = getCacheTTL(path);
  if (ttl > 0) {
    await ctx.redis.setex(cacheKey, ttl, JSON.stringify(result));
  }

  return result;
});

// 速率限制中间件
const rateLimitMiddleware = t.middleware(async ({ ctx, next, path }) => {
  const identifier = ctx.user?.id || ctx.ip;
  const key = `ratelimit:${path}:${identifier}`;

  const [count, ttl] = await Promise.all([
    ctx.redis.incr(key),
    ctx.redis.ttl(key),
  ]);

  // 首次请求，设置过期时间
  if (count === 1) {
    await ctx.redis.expire(key, 60);
  }

  // 检查限制
  const limit = getRateLimit(path, ctx.user?.role);
  if (count > limit) {
    throw new TRPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
    });
  }

  return next({
    ctx: {
      ...ctx,
      rateLimit: {
        remaining: Math.max(0, limit - count),
        resetAt: ttl > 0 ? Date.now() + ttl * 1000 : 0,
      },
    },
  });
});

// 应用中间件
export const router = t.router;
export const publicProcedure = t.procedure
  .use(rateLimitMiddleware)
  .use(auditLogMiddleware)
  .use(cacheMiddleware);

export const protectedProcedure = publicProcedure
  .use(isAuthenticatedMiddleware)
  .use(rateLimitMiddleware)
  .use(auditLogMiddleware)
  .use(cacheMiddleware);

export const adminProcedure = protectedProcedure.use(isAdminMiddleware);
```

### 缓存策略

```typescript
// 缓存 TTL 配置
const CACHE_TTL = {
  // 公开查询 - 较长的缓存时间
  'user.list': 300,        // 5 分钟
  'user.getById': 600,     // 10 分钟
  'post.list': 60,         // 1 分钟
  'post.getById': 300,     // 5 分钟
  'category.list': 3600,   // 1 小时
  
  // 受保护查询 - 较短的缓存时间
  'user.me': 0,            // 不缓存
  'order.list': 0,          // 不缓存
  
  // 默认值
  default: 60,
};

function getCacheTTL(path: string): number {
  return CACHE_TTL[path] ?? CACHE_TTL.default;
}

// 缓存失效策略
async function invalidateCache(pattern: string) {
  const keys = await ctx.redis.keys(`cache:*${pattern}*`);
  if (keys.length > 0) {
    await ctx.redis.del(...keys);
  }
}

// 在 mutation 中清除相关缓存
const userRouter = router({
  update: protectedProcedure
    .input(...)
    .mutation(async ({ input, ctx }) => {
      const user = await ctx.prisma.user.update({ ... });

      // 清除相关缓存
      await invalidateCache('user/list');
      await invalidateCache(`user/getById:${input.id}`);

      return user;
    }),
});
```

---

## tRPC 与实时功能

### WebSocket 订阅

```typescript
// server/router/realtime.ts
import { router, protectedProcedure } from '../trpc';
import { observable } from '@trpc/server/observable';
import { EventEmitter } from 'events';
import { z } from 'zod';

// 事件总线
const ee = new EventEmitter();

// 事件类型
interface UserEvent {
  type: 'created' | 'updated' | 'deleted';
  user: {
    id: string;
    name: string;
    email: string;
  };
  timestamp: number;
}

interface PostEvent {
  type: 'created' | 'updated' | 'deleted' | 'published';
  post: {
    id: string;
    title: string;
    authorId: string;
  };
  timestamp: number;
}

type RealtimeEvent = UserEvent | PostEvent;

export const realtimeRouter = router({
  // 订阅所有事件
  onAnyEvent: protectedProcedure.subscription(({ ctx }) => {
    return observable<RealtimeEvent>((emit) => {
      const onEvent = (event: RealtimeEvent) => {
        emit.next(event);
      };

      ee.on('event', onEvent);

      return () => {
        ee.off('event', onEvent);
      };
    });
  }),

  // 订阅用户事件
  onUserEvent: protectedProcedure.subscription(({ ctx }) => {
    return observable<UserEvent>((emit) => {
      const onUserEvent = (event: UserEvent) => {
        // 只推送用户相关事件
        emit.next(event);
      };

      ee.on('userEvent', onUserEvent);

      return () => {
        ee.off('userEvent', onUserEvent);
      };
    });
  }),

  // 订阅帖子事件
  onPostEvent: protectedProcedure.subscription(({ ctx }) => {
    return observable<PostEvent>((emit) => {
      const onPostEvent = (event: PostEvent) => {
        emit.next(event);
      };

      ee.on('postEvent', onPostEvent);

      return () => {
        ee.off('postEvent', onPostEvent);
      };
    });
  }),

  // 订阅特定帖子的更新
  onPostUpdate: protectedProcedure
    .input(z.object({ postId: z.string() }))
    .subscription(({ input, ctx }) => {
      return observable<PostEvent>((emit) => {
        const channel = `post:${input.postId}`;

        const onPostUpdate = (event: PostEvent) => {
          if (event.post.id === input.postId) {
            emit.next(event);
          }
        };

        ee.on('postEvent', onPostUpdate);

        return () => {
          ee.off('postEvent', onPostUpdate);
        };
      });
    }),

  // 发布用户事件（内部使用）
  publishUserEvent: protectedProcedure
    .input(
      z.object({
        type: z.enum(['created', 'updated', 'deleted']),
        user: z.object({
          id: z.string(),
          name: z.string(),
          email: z.string(),
        }),
      })
    )
    .mutation(({ input }) => {
      const event: UserEvent = {
        ...input,
        timestamp: Date.now(),
      };

      ee.emit('event', event);
      ee.emit('userEvent', event);

      return { success: true };
    }),

  // 发布帖子事件（内部使用）
  publishPostEvent: protectedProcedure
    .input(
      z.object({
        type: z.enum(['created', 'updated', 'deleted', 'published']),
        post: z.object({
          id: z.string(),
          title: z.string(),
          authorId: z.string(),
        }),
      })
    )
    .mutation(({ input }) => {
      const event: PostEvent = {
        ...input,
        timestamp: Date.now(),
      };

      ee.emit('event', event);
      ee.emit('postEvent', event);

      return { success: true };
    }),
});
```

### React 中的实时订阅

```typescript
// components/RealtimeFeed.tsx
import { useState, useEffect } from 'react';
import { useSubscription } from '@trpc/react-query';
import { trpc } from '@/lib/trpc/client';
import { useToast } from '@/components/ui/use-toast';

export function RealtimeFeed() {
  const [events, setEvents] = useState<RealtimeEvent[]>([]);
  const { toast } = useToast();

  // 订阅所有事件
  const { data: eventData } = useSubscription(
    trpc.realtime.onAnyEvent.useSubscription(),
    {
      onStarted() {
        console.log('Subscription started');
      },
      onError(error) {
        console.error('Subscription error:', error);
      },
    }
  );

  // 处理新事件
  useEffect(() => {
    if (eventData) {
      setEvents((prev) => [eventData, ...prev].slice(0, 50));

      // 显示通知
      toast({
        title: `${eventData.type} event`,
        description: `${eventData.type} - ${Date.now()}`,
      });
    }
  }, [eventData, toast]);

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Real-time Events</h2>
      <div className="space-y-2">
        {events.map((event, index) => (
          <div
            key={index}
            className="p-3 bg-gray-100 rounded-lg"
          >
            <div className="font-medium">{event.type}</div>
            <div className="text-sm text-gray-600">
              {JSON.stringify(event)}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}

// 组件中的实时通知
export function UserAvatar({ userId }: { userId: string }) {
  const [isOnline, setIsOnline] = useState(false);
  
  const { data: onlineStatus } = useSubscription(
    trpc.realtime.onUserEvent.useSubscription(),
    {
      enabled: true,
    }
  );

  useEffect(() => {
    if (onlineStatus?.type === 'updated' && onlineStatus.user.id === userId) {
      setIsOnline(true);
      setTimeout(() => setIsOnline(false), 5000);
    }
  }, [onlineStatus, userId]);

  return (
    <div className="relative">
      <Avatar src={getAvatarUrl(userId)} />
      {isOnline && (
        <div className="absolute top-0 right-0 w-3 h-3 bg-green-500 rounded-full" />
      )}
    </div>
  );
}
```

---

> [!SUCCESS]
> tRPC 以其独特的设计理念——"无 Schema 的端到端类型安全"——为 TypeScript 全栈开发提供了全新的范式。通过直接共享 TypeScript 类型，tRPC 实现了真正的类型安全，同时保持了 RPC 的简洁语义。本文档涵盖了从基础概念到高级实践的完整知识体系，包括中间件模式、缓存策略、实时订阅等关键主题，帮助开发者构建生产级别的 tRPC 应用。
