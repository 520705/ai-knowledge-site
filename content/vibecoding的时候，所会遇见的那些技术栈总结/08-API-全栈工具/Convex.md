# Convex：实时后端即平台的深度解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 Convex 的响应式查询、实时数据同步、认证系统、权限控制，以及在 AI 应用中的独特优势。

---

## 目录

1. [[#概述与核心优势]]
2. [[#快速开始与项目初始化]]
3. [[#Schema 定义详解]]
4. [[#Query 查询系统]]
5. [[#Mutation 变更系统]]
6. [[#Action 外部集成]]
7. [[#实时订阅与 Live Queries]]
8. [[#认证与权限]]
9. [[#文件存储]]
10. [[#Convex 部署与扩展]]
11. [[#与其他方案对比]]
12. [[#实战场景]]
13. [[#选型建议]]

---

## 概述与核心优势

### 什么是 Convex

Convex 是一个**后端即平台（Backend as a Platform）**服务，提供了一个完整的服务器端解决方案：数据库、实时订阅、认证、文件存储和 Serverless 函数，全部通过响应式 API 暴露给前端。

与传统的 BaaS（如 Firebase、Supabase）不同，Convex 的核心创新是**响应式查询**。当数据库中的数据发生变化时，所有订阅了该数据的客户端会自动收到更新，无需轮询或手动刷新。

### 核心特性

| 特性 | 说明 |
|------|------|
| **响应式数据库** | 数据变更自动推送到订阅的客户端 |
| **实时订阅** | WebSocket 驱动的 Live Queries |
| **类型安全** | 自动生成的 TypeScript 类型 |
| **ACID 事务** | 强一致性的事务支持 |
| **无服务器函数** | Action 用于外部 API 调用 |
| **内置认证** | 开箱即用的用户系统 |
| **文件存储** | 内置文件上传和 CDN |
| **增量计算** | 自动缓存和优化 |

### 技术架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Convex 架构                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Convex Cloud                         │    │
│  │                                                         │    │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │    │
│  │   │  Database   │  │   Actions   │  │   Auth      │    │    │
│  │   │  (持久层)    │  │  (外部集成)  │  │  (认证)     │    │    │
│  │   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘    │    │
│  │          │                 │                 │           │    │
│  │          └────────────┬────┴─────────────────┘           │    │
│  │                       │                                   │    │
│  │              ┌────────▼────────┐                          │    │
│  │              │  Subscription   │                          │    │
│  │              │    Engine        │                          │    │
│  │              └────────┬────────┘                          │    │
│  └───────────────────────┼───────────────────────────────────┘    │
│                          │                                       │
│            WebSocket 连接 │                                       │
│                          ▼                                       │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                      客户端                              │    │
│  │                                                         │    │
│  │   ┌─────────────────────────────────────────────────┐   │    │
│  │   │              useQuery() Hook                     │   │    │
│  │   │  自动订阅，自动更新，响应式数据                   │   │    │
│  │   └─────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Convex vs 其他方案

| 维度 | Convex | Firebase | Supabase | tRPC + Prisma |
|------|--------|----------|----------|---------------|
| **实时数据** | ✅ 原生支持 | ✅ | ✅ | ❌ 需额外配置 |
| **响应式** | ✅ 自动推送 | ✅ | ❌ 需订阅 | ❌ |
| **数据库** | SQL (PostgreSQL) | NoSQL | PostgreSQL | 自选 |
| **类型安全** | ✅ 自动生成 | 部分 | 需手动 | ✅ 端到端 |
| **认证** | 内置 | 内置 | 内置/自定义 | 需集成 |
| **学习曲线** | 低 | 中 | 中 | 中高 |
| **供应商锁定** | 中 | 高 | 低 | 无 |
| **定价** | 按使用量 | 按使用量 | 按使用量 | 自托管免费 |

---

## 快速开始与项目初始化

### 安装与配置

```bash
# 创建新项目
npm create convex@latest my-app

# 进入目录
cd my-app

# 安装依赖
npm install

# 登录 Convex
npx convex dev

# 在另一个终端启动开发服务器
npm run dev
```

### 项目结构

```
my-app/
├── convex/                  # Convex 后端代码
│   ├── convex.json         # Convex 配置
│   ├── schema.ts           # 数据库 Schema
│   ├── auth.ts             # 认证配置
│   ├── functions/          # Serverless 函数
│   │   ├── messages.ts     # 消息相关函数
│   │   └── users.ts        # 用户相关函数
│   └── storage.ts          # 文件存储配置
│
├── src/                    # 前端代码
│   ├── lib/
│   │   └── convex.ts       # Convex 客户端配置
│   └── ...
│
└── convex.config.ts        # 部署配置
```

### 基本配置

```typescript
// convex.config.ts
import { defineApp } from 'convex/server';

export default defineApp({
  // 认证配置
  auth: {
    // 使用 Convex Auth 或自定义
  },
  
  // 存储配置
  storage: {
    // 文件大小限制（字节）
    maxFileSize: 10 * 1024 * 1024, // 10MB
    // 允许的文件类型
    allowedFileTypes: ['image/*', 'application/pdf'],
  },
  
  // 函数超时配置
  functions: {
    // 默认超时时间（毫秒）
    timeoutMs: 30000,
    // 最大超时时间
    maxTimeoutMs: 300000,
  },
});
```

---

## Schema 定义详解

### 基础 Schema

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from 'convex/server';
import { v } from 'convex/values';

export default defineSchema({
  // 用户表（Convex Auth 自动管理）
  users: defineTable({
    name: v.string(),
    email: v.string(),
    image: v.optional(v.string()),
  }).index('by_email', ['email']),

  // 帖子表
  posts: defineTable({
    author: v.id('users'),
    title: v.string(),
    body: v.string(),
    published: v.boolean(),
    likes: v.number(),
    createdAt: v.number(),
    updatedAt: v.number(),
  })
    .index('by_author', ['author'])
    .index('by_published', ['published'])
    .index('by_created', ['createdAt']),

  // 评论表
  comments: defineTable({
    post: v.id('posts'),
    author: v.id('users'),
    body: v.string(),
    createdAt: v.number(),
  })
    .index('by_post', ['post'])
    .index('by_author', ['author']),

  // 点赞表（多对多关系）
  likes: defineTable({
    post: v.id('posts'),
    user: v.id('users'),
    createdAt: v.number(),
  })
    .index('by_post', ['post'])
    .index('by_user', ['user'])
    .index('by_post_user', ['post', 'user']),
});
```

### 字段类型

```typescript
import { v } from 'convex/values';

// 基础类型
v.string()           // 字符串
v.number()           // 数字
v.boolean()          // 布尔值
v.null()              // null
v.bytes()            // 二进制数据

// 复杂类型
v.id('tableName')    // 引用其他表的 ID
v.array(v.string())  // 数组
v.object({           // 对象
  name: v.string(),
  age: v.number(),
})
v.optional(v.string())  // 可选值
v.union(              // 联合类型
  v.object({ type: v.literal('a'), a: v.string() }),
  v.object({ type: v.literal('b'), b: v.number() }),
)

// 验证类型
v.literal('pending')  // 字面量
v.enum(['red', 'green', 'blue'])  // 枚举

// 时间类型（存储为数字时间戳）
v.datetime()          // 日期时间
v.date()              // 日期

// 地理类型
v.geography()         // 地理位置

// JSON 类型
v.any()               // 任意 JSON
v.record(v.string(), v.number())  // 记录/映射
```

### 索引设计

```typescript
// 单一字段索引
defineTable({
  email: v.string(),
}).index('by_email', ['email'])

// 多字段复合索引
defineTable({
  tenant: v.id('tenants'),
  status: v.string(),
  createdAt: v.number(),
})
  .index('by_tenant_status', ['tenant', 'status'])
  .index('by_tenant_created', ['tenant', 'createdAt'])

// 唯一约束
defineTable({
  slug: v.string(),
}).index('by_slug', ['slug'])

// 索引类型
.index('by_field', ['field'], { unique: true })  // 唯一索引
.index('by_field', ['field'], { sort: true })    // 排序优化索引
```

---

## Query 查询系统

### 基础查询

```typescript
// convex/functions/posts.ts
import { query } from '../_generated/server';
import { v } from 'convex/values';

// 公开查询 - 获取所有已发布的帖子
export const list = query({
  args: {
    limit: v.optional(v.number()),
    cursor: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    const limit = args.limit ?? 10;
    
    let posts = await ctx.db
      .query('posts')
      .withIndex('by_published', (q) => q.eq('published', true))
      .order('desc')
      .take(limit + 1);
    
    const hasMore = posts.length > limit;
    if (hasMore) {
      posts = posts.slice(0, -1);
    }
    
    return {
      posts,
      nextCursor: hasMore ? posts[posts.length - 1].createdAt : null,
    };
  },
});

// 获取单个帖子
export const get = query({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      return null;
    }
    
    // 获取作者信息
    const author = await ctx.db.get(post.author);
    
    return {
      ...post,
      author: author ? { name: author.name, image: author.image } : null,
    };
  },
});

// 获取用户的帖子
export const byUser = query({
  args: { userId: v.id('users') },
  handler: async (ctx, args) => {
    const posts = await ctx.db
      .query('posts')
      .withIndex('by_author', (q) => q.eq('author', args.userId))
      .order('desc')
      .collect();
    
    return posts;
  },
});
```

### 复杂查询

```typescript
// 分页查询
export const paginated = query({
  args: {
    cursor: v.optional(v.number()),
    limit: v.optional(v.number()),
    filter: v.optional(v.object({
      author: v.optional(v.id('users')),
      published: v.optional(v.boolean()),
    })),
  },
  handler: async (ctx, args) => {
    const limit = args.limit ?? 20;
    
    let q = ctx.db.query('posts').withIndex('by_created');
    
    // 应用游标
    if (args.cursor !== undefined) {
      q = q.filter((q) => q.lt(q.field('createdAt'), args.cursor!));
    }
    
    // 应用过滤器
    if (args.filter?.author) {
      q = q.filter((q) => q.eq(q.field('author'), args.filter!.author!));
    }
    if (args.filter?.published !== undefined) {
      q = q.filter((q) => q.eq(q.field('published'), args.filter!.published!));
    }
    
    const posts = await q.take(limit + 1);
    const hasMore = posts.length > limit;
    const items = hasMore ? posts.slice(0, -1) : posts;
    
    return {
      posts: items,
      nextCursor: hasMore ? items[items.length - 1].createdAt : null,
    };
  },
});

// 聚合查询
export const stats = query({
  args: { userId: v.id('users') },
  handler: async (ctx, args) => {
    const posts = await ctx.db
      .query('posts')
      .withIndex('by_author', (q) => q.eq('author', args.userId))
      .collect();
    
    const totalLikes = posts.reduce((sum, p) => sum + p.likes, 0);
    const publishedPosts = posts.filter((p) => p.published);
    
    return {
      totalPosts: posts.length,
      publishedPosts: publishedPosts.length,
      draftPosts: posts.length - publishedPosts.length,
      totalLikes,
      avgLikes: posts.length > 0 ? totalLikes / posts.length : 0,
    };
  },
});

// 全文搜索
export const search = query({
  args: { query: v.string() },
  handler: async (ctx, args) => {
    const lowerQuery = args.query.toLowerCase();
    
    const posts = await ctx.db
      .query('posts')
      .withIndex('by_published', (q) => q.eq('published', true))
      .collect();
    
    // 简单过滤（实际项目建议使用全文索引）
    return posts.filter((post) =>
      post.title.toLowerCase().includes(lowerQuery) ||
      post.body.toLowerCase().includes(lowerQuery)
    );
  },
});
```

### 关联查询

```typescript
// 获取帖子及其评论
export const withComments = query({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      return null;
    }
    
    const author = await ctx.db.get(post.author);
    const comments = await ctx.db
      .query('comments')
      .withIndex('by_post', (q) => q.eq('post', args.postId))
      .order('asc')
      .collect();
    
    // 获取评论者信息
    const commentsWithAuthors = await Promise.all(
      comments.map(async (comment) => {
        const author = await ctx.db.get(comment.author);
        return {
          ...comment,
          author: author ? { name: author.name, image: author.image } : null,
        };
      })
    );
    
    return {
      ...post,
      author: author ? { name: author.name, image: author.image } : null,
      comments: commentsWithAuthors,
    };
  },
});
```

---

## Mutation 变更系统

### 基础 Mutation

```typescript
// convex/functions/posts.ts
import { mutation } from '../_generated/server';
import { v } from 'convex/values';

// 创建帖子
export const create = mutation({
  args: {
    title: v.string(),
    body: v.string(),
    published: v.boolean(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const postId = await ctx.db.insert('posts', {
      author: user._id,
      title: args.title,
      body: args.body,
      published: args.published,
      likes: 0,
      createdAt: Date.now(),
      updatedAt: Date.now(),
    });
    
    return postId;
  },
});

// 更新帖子
export const update = mutation({
  args: {
    postId: v.id('posts'),
    title: v.optional(v.string()),
    body: v.optional(v.string()),
    published: v.optional(v.boolean()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    // 验证所有者
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user || post.author !== user._id) {
      throw new Error('Not authorized');
    }
    
    const updates: Record<string, unknown> = { updatedAt: Date.now() };
    if (args.title !== undefined) updates.title = args.title;
    if (args.body !== undefined) updates.body = args.body;
    if (args.published !== undefined) updates.published = args.published;
    
    await ctx.db.patch(args.postId, updates);
    
    return { success: true };
  },
});

// 删除帖子
export const remove = mutation({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user || post.author !== user._id) {
      throw new Error('Not authorized');
    }
    
    // 删除相关评论
    const comments = await ctx.db
      .query('comments')
      .withIndex('by_post', (q) => q.eq('post', args.postId))
      .collect();
    
    for (const comment of comments) {
      await ctx.db.delete(comment._id);
    }
    
    // 删除帖子
    await ctx.db.delete(args.postId);
    
    return { success: true };
  },
});
```

### 点赞 Mutation

```typescript
// 点赞/取消点赞
export const toggleLike = mutation({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    // 检查是否已点赞
    const existingLike = await ctx.db
      .query('likes')
      .withIndex('by_post_user', (q) => 
        q.eq('post', args.postId).eq('user', user._id)
      )
      .first();
    
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    if (existingLike) {
      // 取消点赞
      await ctx.db.delete(existingLike._id);
      await ctx.db.patch(args.postId, {
        likes: Math.max(0, post.likes - 1),
      });
      return { liked: false, likes: post.likes - 1 };
    } else {
      // 添加点赞
      await ctx.db.insert('likes', {
        post: args.postId,
        user: user._id,
        createdAt: Date.now(),
      });
      await ctx.db.patch(args.postId, {
        likes: post.likes + 1,
      });
      return { liked: true, likes: post.likes + 1 };
    }
  },
});
```

### 评论 Mutation

```typescript
// 添加评论
export const addComment = mutation({
  args: {
    postId: v.id('posts'),
    body: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    const commentId = await ctx.db.insert('comments', {
      post: args.postId,
      author: user._id,
      body: args.body,
      createdAt: Date.now(),
    });
    
    return commentId;
  },
});

// 删除评论
export const deleteComment = mutation({
  args: { commentId: v.id('comments') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const comment = await ctx.db.get(args.commentId);
    
    if (!comment) {
      throw new Error('Comment not found');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user || comment.author !== user._id) {
      throw new Error('Not authorized');
    }
    
    await ctx.db.delete(args.commentId);
    
    return { success: true };
  },
});
```

---

## Action 外部集成

### Action 概述

Action 是 Convex 中用于调用外部 API、支付网关、邮件服务等的函数。它们在服务端执行，可以调用第三方 API、处理敏感操作、运行定时任务等。

```typescript
// convex/functions/integrations.ts
import { action } from '../_generated/server';
import { v } from 'convex/values';
```

### Stripe 支付集成

```typescript
// 支付集成
export const createCheckoutSession = action({
  args: {
    priceId: v.string(),
    successUrl: v.string(),
    cancelUrl: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const session = await stripe.checkout.sessions.create({
      mode: 'subscription',
      payment_method_types: ['card'],
      line_items: [
        {
          price: args.priceId,
          quantity: 1,
        },
      ],
      success_url: args.successUrl,
      cancel_url: args.cancelUrl,
      customer_email: identity.email!,
      metadata: {
        userId: identity.subject,
      },
    });
    
    return { sessionUrl: session.url };
  },
});

// Webhook 处理
export const handleStripeWebhook = action({
  args: {
    body: v.bytes(),
    headers: v.record(v.string(), v.string()),
  },
  handler: async (ctx, args) => {
    const sig = args.headers['stripe-signature'];
    
    let event;
    
    try {
      event = stripe.webhooks.constructEvent(
        args.body,
        sig!,
        process.env.STRIPE_WEBHOOK_SECRET!
      );
    } catch (err) {
      throw new Error('Invalid signature');
    }
    
    switch (event.type) {
      case 'checkout.session.completed': {
        const session = event.data.object;
        
        // 更新用户订阅状态
        const user = await ctx.runQuery(
          query({ table: 'users' }).filter((q) => 
            q.eq(q.field('email'), session.customer_email)
          ).first()
        );
        
        if (user) {
          await ctx.runMutation(
            mutation({
              name: 'updateSubscription',
              args: {
                userId: user._id,
                subscriptionStatus: 'active',
                stripeCustomerId: session.customer,
              },
            })
          );
        }
        break;
      }
      
      case 'customer.subscription.deleted': {
        const subscription = event.data.object;
        
        await ctx.runMutation(
          mutation({
            name: 'updateSubscription',
            args: {
              stripeCustomerId: subscription.customer,
              subscriptionStatus: 'canceled',
            },
          })
        );
        break;
      }
    }
    
    return { received: true };
  },
});
```

### 邮件发送

```typescript
// 发送邮件
export const sendEmail = action({
  args: {
    to: v.string(),
    subject: v.string(),
    html: v.string(),
  },
  handler: async (ctx, args) => {
    // 使用 Resend、SendGrid 或其他邮件服务
    const response = await fetch('https://api.resend.com/emails', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.RESEND_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        from: 'onboarding@resend.dev',
        to: args.to,
        subject: args.subject,
        html: args.html,
      }),
    });
    
    if (!response.ok) {
      throw new Error('Failed to send email');
    }
    
    return { success: true };
  },
});

// 发送欢迎邮件
export const sendWelcomeEmail = action({
  args: { userId: v.id('users') },
  handler: async (ctx, args) => {
    const user = await ctx.db.get(args.userId);
    
    if (!user) {
      throw new Error('User not found');
    }
    
    await ctx.scheduler.runAfter(0, sendEmail, {
      to: user.email!,
      subject: 'Welcome to our platform!',
      html: `
        <h1>Welcome, ${user.name || 'User'}!</h1>
        <p>Thank you for joining us.</p>
      `,
    });
    
    return { scheduled: true };
  },
});
```

### AI 服务集成

```typescript
// AI 补全
export const generateCompletion = action({
  args: {
    prompt: v.string(),
    maxTokens: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'gpt-4o',
        messages: [
          { role: 'system', content: 'You are a helpful assistant.' },
          { role: 'user', content: args.prompt },
        ],
        max_tokens: args.maxTokens ?? 500,
      }),
    });
    
    if (!response.ok) {
      throw new Error('AI request failed');
    }
    
    const data = await response.json();
    return { completion: data.choices[0].message.content };
  },
});

// 生成帖子摘要
export const summarizePost = action({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'gpt-4o',
        messages: [
          { 
            role: 'system', 
            content: 'You are a blog post summarizer. Provide a 2-3 sentence summary.' 
          },
          { role: 'user', content: post.body },
        ],
        max_tokens: 100,
      }),
    });
    
    const data = await response.json();
    const summary = data.choices[0].message.content;
    
    await ctx.db.patch(args.postId, { summary });
    
    return { summary };
  },
});
```

---

## 实时订阅与 Live Queries

### 前端订阅

```typescript
// src/lib/convex.ts
import { ConvexReactClient } from 'convex/react';

const convexUrl = process.env.NEXT_PUBLIC_CONVEX_URL || '';

export const convex = new ConvexReactClient(convexUrl);
```

```typescript
// src/components/PostList.tsx
'use client';

import { useQuery } from 'convex/react';
import { api } from '../convex/generated/api';

export function PostList() {
  // 自动订阅 posts 数据
  const postsData = useQuery(api.posts.list, { limit: 10 });
  
  if (postsData === undefined) {
    return <div>Loading...</div>;
  }
  
  return (
    <div>
      {postsData.posts.map((post) => (
        <PostCard key={post._id} post={post} />
      ))}
    </div>
  );
}

// 订阅单个帖子
export function PostDetail({ postId }: { postId: string }) {
  const post = useQuery(api.posts.get, { postId });
  
  if (post === undefined) {
    return <div>Loading...</div>;
  }
  
  if (post === null) {
    return <div>Post not found</div>;
  }
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div dangerouslySetInnerHTML={{ __html: post.body }} />
    </article>
  );
}

// 实时评论
export function CommentSection({ postId }: { postId: string }) {
  const post = useQuery(api.posts.withComments, { postId });
  
  if (post === undefined) {
    return <div>Loading...</div>;
  }
  
  return (
    <div>
      <h2>Comments ({post.comments.length})</h2>
      {post.comments.map((comment) => (
        <CommentCard key={comment._id} comment={comment} />
      ))}
    </div>
  );
}
```

### Optimistic Updates

```typescript
// src/components/CreatePostForm.tsx
'use client';

import { useState } from 'react';
import { useMutation } from 'convex/react';
import { api } from '../convex/generated/api';

export function CreatePostForm() {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');
  
  const createPost = useMutation(api.posts.create);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    try {
      await createPost({
        title,
        body,
        published: false,
      });
      
      // 重置表单
      setTitle('');
      setBody('');
    } catch (error) {
      console.error('Failed to create post:', error);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Post title"
        required
      />
      <textarea
        value={body}
        onChange={(e) => setBody(e.target.value)}
        placeholder="Post body"
        required
      />
      <button type="submit">Create Post</button>
    </form>
  );
}

// 带乐观更新的点赞
export function LikeButton({ postId, initialLikes, isLiked }: Props) {
  const [optimisticLikes, setOptimisticLikes] = useState(initialLikes);
  const [optimisticLiked, setOptimisticLiked] = useState(isLiked);
  
  const toggleLike = useMutation(api.posts.toggleLike);
  
  const handleClick = async () => {
    // 乐观更新
    setOptimisticLiked(!optimisticLiked);
    setOptimisticLikes(optimisticLiked ? optimisticLikes - 1 : optimisticLikes + 1);
    
    try {
      await toggleLike({ postId });
    } catch (error) {
      // 回滚
      setOptimisticLiked(optimisticLiked);
      setOptimisticLikes(initialLikes);
    }
  };
  
  return (
    <button onClick={handleClick}>
      {optimisticLiked ? '❤️' : '🤍'} {optimisticLikes}
    </button>
  );
}
```

### 订阅多个数据源

```typescript
// src/components/Dashboard.tsx
'use client';

import { useQuery } from 'convex/react';
import { api } from '../convex/generated/api';

export function Dashboard() {
  // 并行订阅多个查询
  const [userPosts, stats, notifications] = [
    useQuery(api.posts.byUser, { userId: 'current-user-id' }),
    useQuery(api.posts.stats, { userId: 'current-user-id' }),
    useQuery(api.notifications.list),
  ];
  
  // 当任何一个还在加载时显示加载状态
  if (userPosts === undefined || stats === undefined || notifications === undefined) {
    return <DashboardSkeleton />;
  }
  
  return (
    <div>
      <StatsCard stats={stats} />
      <PostsList posts={userPosts} />
      <NotificationList notifications={notifications} />
    </div>
  );
}
```

---

## 认证与权限

### Convex Auth 配置

```typescript
// convex/auth.ts
import { convexAuth } from '@convex-dev/auth';
import { Password } from '@convex-dev/auth/providers/Password';
import { query } from './_generated/server';

export const { auth, signIn, signOut, store } = convexAuth({
  providers: [
    Password({
      // 验证邮箱格式
      validateEmail(email) {
        return email.includes('@') ? email : null;
      },
      // 验证密码强度
      async validatePassword(password) {
        if (password.length < 8) {
          return 'Password must be at least 8 characters';
        }
        return null;
      },
    }),
  ],
  // 回调函数
  callbacks: {
    // 用户创建后
    async afterCreateNewUser(ctx, { userId, account }) {
      // 获取认证信息
      const identity = await ctx.auth.getUserIdentity();
      
      // 在 users 表中创建用户记录
      await ctx.db.insert('users', {
        name: identity?.givenName || identity?.name || '',
        email: identity?.email || '',
        image: identity?.picture || '',
      });
    },
  },
});

// 获取当前用户
export const currentUser = query({
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      return null;
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    return user;
  },
});
```

### 权限控制

```typescript
// convex/functions/posts.ts

// 私有查询 - 只有作者可以看到草稿
export const myDrafts = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const posts = await ctx.db
      .query('posts')
      .withIndex('by_author', (q) => q.eq('author', user._id))
      .filter((q) => q.eq(q.field('published'), false))
      .collect();
    
    return posts;
  },
});

// 管理员查询
export const allPosts = query({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user || user.role !== 'admin') {
      throw new Error('Admin access required');
    }
    
    return ctx.db.query('posts').collect();
  },
});

// 修改帖子（权限检查）
export const deleteAsAdmin = mutation({
  args: { postId: v.id('posts') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    const post = await ctx.db.get(args.postId);
    
    if (!post) {
      throw new Error('Post not found');
    }
    
    // 只有作者或管理员可以删除
    const isAuthor = user && post.author === user._id;
    const isAdmin = user?.role === 'admin';
    
    if (!isAuthor && !isAdmin) {
      throw new Error('Not authorized');
    }
    
    await ctx.db.delete(args.postId);
    
    return { success: true };
  },
});
```

### OAuth 认证

```typescript
// 添加 Google OAuth
// convex/auth.ts
import { convexAuth } from '@convex-dev/auth';
import { Password } from '@convex-dev/auth/providers/Password';
import { Google } from '@convex-dev/auth/providers/Google';

export const { auth, signIn, signOut, store } = convexAuth({
  providers: [
    Password({...}),
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
  ],
});

// 前端登录
export function SignIn() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const signInWithPassword = useAction(authActions.signIn);
  const signInWithGoogle = useAction(authActions.signIn);
  
  return (
    <div>
      <button onClick={() => signInWithGoogle({ provider: 'google' })}>
        Sign in with Google
      </button>
      
      <form>
        <input 
          type="email" 
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />
        <input 
          type="password" 
          value={password}
          onChange={(e) => setPassword(e.target.value)}
        />
        <button onClick={() => signInWithPassword({ email, password })}>
          Sign in
        </button>
      </form>
    </div>
  );
}
```

---

## 文件存储

### 文件上传

```typescript
// convex/storage.ts
import { action } from '../_generated/server';
import { v } from 'convex/values';

// 生成上传 URL
export const generateUploadUrl = action({
  args: {},
  handler: async (ctx) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    return await ctx.storage.generateUploadUrl();
  },
});

// 保存文件引用
export const saveFile = mutation({
  args: {
    storageId: v.id('_storage'),
    fileName: v.string(),
    mimeType: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    return await ctx.db.insert('files', {
      userId: user._id,
      storageId: args.storageId,
      fileName: args.fileName,
      mimeType: args.mimeType,
      createdAt: Date.now(),
    });
  },
});

// 获取文件 URL
export const getFileUrl = action({
  args: { storageId: v.id('_storage') },
  handler: async (ctx, args) => {
    return await ctx.storage.getUrl(args.storageId);
  },
});
```

### 前端上传组件

```typescript
// src/components/FileUpload.tsx
'use client';

import { useState } from 'react';
import { useAction, useMutation } from 'convex/react';
import { api } from '../convex/generated/api';

export function FileUpload() {
  const [file, setFile] = useState<File | null>(null);
  const [uploading, setUploading] = useState(false);
  
  const generateUploadUrl = useAction(api.storage.generateUploadUrl);
  const saveFile = useMutation(api.storage.saveFile);
  
  const handleUpload = async () => {
    if (!file) return;
    
    setUploading(true);
    
    try {
      // 1. 获取上传 URL
      const uploadUrl = await generateUploadUrl();
      
      // 2. 上传到 Convex
      const response = await fetch(uploadUrl, {
        method: 'POST',
        headers: { 'Content-Type': file.type },
        body: file,
      });
      
      if (!response.ok) {
        throw new Error('Upload failed');
      }
      
      const { storageId } = await response.json();
      
      // 3. 保存文件引用
      await saveFile({
        storageId,
        fileName: file.name,
        mimeType: file.type,
      });
      
      setFile(null);
      alert('File uploaded successfully!');
    } catch (error) {
      console.error('Upload error:', error);
      alert('Upload failed');
    } finally {
      setUploading(false);
    }
  };
  
  return (
    <div>
      <input
        type="file"
        onChange={(e) => setFile(e.target.files?.[0] || null)}
      />
      <button onClick={handleUpload} disabled={!file || uploading}>
        {uploading ? 'Uploading...' : 'Upload'}
      </button>
    </div>
  );
}
```

### 图片预览

```typescript
// src/components/ImagePreview.tsx
'use client';

import { useAction } from 'convex/react';
import { api } from '../convex/generated/api';
import { useEffect, useState } from 'react';

export function ImagePreview({ storageId }: { storageId: string }) {
  const [url, setUrl] = useState<string | null>(null);
  const getFileUrl = useAction(api.storage.getFileUrl);
  
  useEffect(() => {
    getFileUrl({ storageId }).then(setUrl);
  }, [storageId, getFileUrl]);
  
  if (!url) {
    return <div>Loading...</div>;
  }
  
  return <img src={url} alt="Uploaded file" />;
}
```

---

## Convex 部署与扩展

### 部署配置

```typescript
// convex.config.ts
import { defineApp } from 'convex/server';
import auth from './auth';

export default defineApp({
  // 认证
  auth,
  
  // 存储限制
  storage: {
    maxFileSize: 50 * 1024 * 1024, // 50MB
  },
  
  // 区域配置
  regions: ['us-east-1', 'eu-west-1', 'ap-south-1'],
  
  // 监控
  telemetry: {
    enabled: true,
    provider: 'datadog',
    apiKey: process.env.DD_API_KEY,
  },
});
```

### 环境变量

```bash
# .env.local
CONVEX_DEPLOYMENT=demo:your-project
NEXT_PUBLIC_CONVEX_URL=https://your-project.convex.cloud

# 第三方服务
OPENAI_API_KEY=sk-...
STRIPE_SECRET_KEY=sk_test_...
RESEND_API_KEY=re_...
```

### 部署命令

```bash
# 开发
npx convex dev

# 部署到生产
npx convex deploy

# 查看部署状态
npx convex status

# 监控日志
npx convex logs --tail

# 回滚
npx convex rollback --deployment production
```

---

## 与其他方案对比

### Convex vs Supabase

| 维度 | Convex | Supabase |
|------|--------|----------|
| **实时订阅** | ✅ WebSocket 原生 | ✅ PostgreSQL 订阅 |
| **响应式** | ✅ 自动推送 | ❌ 需手动订阅 |
| **数据库** | PostgreSQL | PostgreSQL |
| **类型安全** | ✅ 自动生成 | 需 Prisma/Drizzle |
| **认证** | 内置 Auth | 内置/自定义 |
| **文件存储** | ✅ 内置 | ✅ Storage |
| **定价** | 按使用量 | 按使用量 |

### Convex vs Firebase

| 维度 | Convex | Firebase |
|------|--------|----------|
| **数据库类型** | SQL | NoSQL |
| **实时数据** | ✅ | ✅ |
| **响应式** | ✅ | ❌ |
| **类型安全** | ✅ 自动 | 部分 |
| **定价模型** | 按使用量 | 按操作量 |
| **供应商锁定** | 中 | 高 |

### Convex vs tRPC + Prisma

| 维度 | Convex | tRPC + Prisma |
|------|--------|---------------|
| **实时数据** | ✅ 内置 | ❌ 需额外配置 |
| **响应式** | ✅ | ❌ |
| **数据库** | Convex Cloud | 自选 |
| **类型安全** | ✅ | ✅ 端到端 |
| **学习曲线** | 低 | 中高 |
| **供应商锁定** | 中 | 无 |

---

## 实战场景

### 场景一：实时聊天应用

```typescript
// convex/functions/chat.ts

// 发送消息
export const sendMessage = mutation({
  args: {
    conversationId: v.id('conversations'),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const messageId = await ctx.db.insert('messages', {
      conversationId: args.conversationId,
      senderId: user._id,
      content: args.content,
      createdAt: Date.now(),
    });
    
    // 更新会话的最后消息
    await ctx.db.patch(args.conversationId, {
      lastMessage: args.content,
      lastMessageAt: Date.now(),
    });
    
    return messageId;
  },
});

// 获取消息（实时更新）
export const getMessages = query({
  args: { conversationId: v.id('conversations') },
  handler: async (ctx, args) => {
    const messages = await ctx.db
      .query('messages')
      .withIndex('by_conversation', (q) => 
        q.eq('conversationId', args.conversationId)
      )
      .order('asc')
      .collect();
    
    // 获取发送者信息
    return Promise.all(
      messages.map(async (msg) => {
        const sender = await ctx.db.get(msg.senderId);
        return {
          ...msg,
          sender: sender ? { name: sender.name, image: sender.image } : null,
        };
      })
    );
  },
});
```

```typescript
// src/components/ChatRoom.tsx
'use client';

import { useQuery } from 'convex/react';
import { api } from '../convex/generated/api';
import { useState } from 'react';

export function ChatRoom({ conversationId }: { conversationId: string }) {
  const messages = useQuery(api.chat.getMessages, { conversationId });
  const sendMessage = useMutation(api.chat.sendMessage);
  const [newMessage, setNewMessage] = useState('');
  
  const handleSend = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!newMessage.trim()) return;
    
    await sendMessage({
      conversationId,
      content: newMessage,
    });
    
    setNewMessage('');
  };
  
  if (messages === undefined) {
    return <div>Loading messages...</div>;
  }
  
  return (
    <div className="chat-room">
      <div className="messages">
        {messages.map((msg) => (
          <div key={msg._id} className="message">
            <img src={msg.sender?.image} alt={msg.sender?.name} />
            <div>
              <strong>{msg.sender?.name}</strong>
              <p>{msg.content}</p>
            </div>
          </div>
        ))}
      </div>
      
      <form onSubmit={handleSend}>
        <input
          value={newMessage}
          onChange={(e) => setNewMessage(e.target.value)}
          placeholder="Type a message..."
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
}
```

### 场景二：协作文档编辑

```typescript
// convex/functions/documents.ts

// 创建文档
export const create = mutation({
  args: { title: v.string() },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const docId = await ctx.db.insert('documents', {
      title: args.title,
      content: '',
      owner: user._id,
      collaborators: [user._id],
      createdAt: Date.now(),
      updatedAt: Date.now(),
    });
    
    return docId;
  },
});

// 更新内容
export const updateContent = mutation({
  args: {
    documentId: v.id('documents'),
    content: v.string(),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    
    if (!identity) {
      throw new Error('Not authenticated');
    }
    
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();
    
    if (!user) {
      throw new Error('User not found');
    }
    
    const doc = await ctx.db.get(args.documentId);
    
    if (!doc) {
      throw new Error('Document not found');
    }
    
    if (!doc.collaborators.includes(user._id)) {
      throw new Error('Not authorized');
    }
    
    await ctx.db.patch(args.documentId, {
      content: args.content,
      updatedAt: Date.now(),
    });
    
    return { success: true };
  },
});

// 获取协作者
export const getCollaborators = query({
  args: { documentId: v.id('documents') },
  handler: async (ctx, args) => {
    const doc = await ctx.db.get(args.documentId);
    
    if (!doc) {
      return [];
    }
    
    return Promise.all(
      doc.collaborators.map(async (userId) => {
        const user = await ctx.db.get(userId);
        return user ? { name: user.name, image: user.image } : null;
      })
    );
  },
});
```

---

## 选型建议

### 选择 Convex 的理由

- ✅ 需要实时数据同步
- ✅ 想要响应式更新（无需手动刷新）
- ✅ 快速原型开发
- ✅ 不想管理服务器
- ✅ 需要内置认证
- ✅ TypeScript 项目

### 不选择 Convex 的场景

- ❌ 想要完全自托管
- ❌ 需要 NoSQL 数据库
- ❌ 项目预算极其有限
- ❌ 需要高级查询功能（如全文搜索）
- ❌ 团队偏好 SQL 迁移工具

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.convex.dev |
| Convex Auth | https://auth.convex.dev |
| GitHub | https://github.com/get-convex/convex |
| 示例项目 | https://docs.convex.dev/examples |
| Discord | https://discord.gg/convex |

---

> [!SUCCESS]
> Convex 是构建实时应用的绝佳选择，其响应式查询和实时数据同步能力让复杂应用的开发变得简单。对于需要多人协作、实时更新的 vibecoding 项目，Convex 提供了开箱即用的完整解决方案。

---

## Convex 高级模式与最佳实践

### 后台任务与定时任务

```typescript
// convex/scheduler.ts
import { internalMutation, internalQuery } from './_generated/server';

// 定时清理过期数据
export const cleanupExpiredSessions = internalMutation({
  args: {},
  handler: async (ctx) => {
    const now = Date.now();
    const oneMonthAgo = now - 30 * 24 * 60 * 60 * 1000;

    // 获取过期会话
    const expiredSessions = await ctx.db
      .query('sessions')
      .filter((q) => q.lt(q.field('expiresAt'), oneMonthAgo))
      .collect();

    // 删除过期会话
    for (const session of expiredSessions) {
      await ctx.db.delete(session._id);
    }

    return { deleted: expiredSessions.length };
  },
});

// 统计任务（每周运行）
export const weeklyStatsReport = internalMutation({
  args: {},
  handler: async (ctx) => {
    const oneWeekAgo = Date.now() - 7 * 24 * 60 * 60 * 1000;

    // 统计新用户数
    const newUsers = await ctx.db
      .query('users')
      .filter((q) => q.gte(q.field('createdAt'), oneWeekAgo))
      .collect();

    // 统计活跃用户
    const activeUsers = await ctx.db
      .query('sessions')
      .filter((q) => q.gte(q.field('createdAt'), oneWeekAgo))
      .collect();

    const uniqueActiveUsers = new Set(activeSessions.map((s) => s.userId)).size;

    // 生成报告
    const report = await ctx.db.insert('reports', {
      type: 'weekly',
      data: {
        newUsers: newUsers.length,
        activeUsers: uniqueActiveUsers,
        generatedAt: Date.now(),
      },
    });

    return report;
  },
});
```

### 乐观更新与冲突处理

```typescript
// convex/functions/tasks.ts
export const updateTask = mutation({
  args: {
    taskId: v.id('tasks'),
    title: v.optional(v.string()),
    completed: v.optional(v.boolean()),
    version: v.number(), // 用于乐观锁
  },
  handler: async (ctx, args) => {
    const task = await ctx.db.get(args.taskId);

    if (!task) {
      throw new Error('Task not found');
    }

    // 乐观锁检查版本
    if (task.version !== args.version) {
      throw new Error('Conflict: Task was modified by another user');
    }

    const updates: Record<string, unknown> = {
      updatedAt: Date.now(),
      version: task.version + 1,
    };

    if (args.title !== undefined) updates.title = args.title;
    if (args.completed !== undefined) updates.completed = args.completed;

    await ctx.db.patch(args.taskId, updates);

    return { success: true, newVersion: task.version + 1 };
  },
});
```

```typescript
// src/components/TaskItem.tsx
'use client';

import { useState } from 'react';
import { useMutation } from 'convex/react';
import { api } from '../convex/generated/api';

export function TaskItem({ task }: { task: Task }) {
  const [localTitle, setLocalTitle] = useState(task.title);
  const [version, setVersion] = useState(task.version);

  const updateTask = useMutation(api.tasks.update);

  const handleTitleChange = async (newTitle: string) => {
    const oldTitle = localTitle;
    const oldVersion = version;

    // 乐观更新
    setLocalTitle(newTitle);

    try {
      const result = await updateTask({
        taskId: task._id,
        title: newTitle,
        version: oldVersion,
      });

      // 更新版本号
      setVersion(result.newVersion);
    } catch (error) {
      // 回滚
      setLocalTitle(oldTitle);
      alert('Failed to update task. Please try again.');
    }
  };

  return (
    <div>
      <input
        value={localTitle}
        onChange={(e) => setLocalTitle(e.target.value)}
        onBlur={() => handleTitleChange(localTitle)}
      />
      <span>v{version}</span>
    </div>
  );
}
```

### 数据验证与清理

```typescript
// convex/validators.ts
import { v } from 'convex/values';

// 自定义验证器
export const postValidator = v.object({
  title: v.string(),
  body: v.string(),
  tags: v.array(v.string()),
});

// 验证函数
export function validatePost(data: unknown) {
  try {
    return { success: true, data: postValidator.parse(data) };
  } catch (error) {
    return { success: false, error };
  }
}

// 清理 HTML
export function sanitizeHtml(html: string): string {
  // 移除 script 标签
  let clean = html.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');

  // 移除事件处理器
  clean = clean.replace(/\s*on\w+\s*=\s*["'][^"']*["']/gi, '');

  // 限制标签
  const allowedTags = ['p', 'br', 'strong', 'em', 'u', 'h1', 'h2', 'h3', 'ul', 'ol', 'li', 'a', 'blockquote'];
  clean = clean.replace(/<\/?(?!(?:' + allowedTags.join('|') + ')\b)[^>]*>/gi, '');

  return clean;
}
```

### 多租户隔离

```typescript
// convex/schema.ts
export const organizations = defineTable({
  name: v.string(),
  slug: v.string(),
  plan: v.union(
    v.literal('free'),
    v.literal('pro'),
    v.literal('enterprise')
  ),
  createdAt: v.number(),
}).index('by_slug', ['slug']);

export const tenants = defineTable({
  organization: v.id('organizations'),
  name: v.string(),
  settings: v.object({
    theme: v.optional(v.string()),
    features: v.optional(v.array(v.string())),
  }),
  createdAt: v.number(),
}).index('by_organization', ['organization']);

// 权限检查
export const canAccessTenant = query({
  args: { tenantId: v.id('tenants') },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) return false;

    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();

    if (!user) return false;

    const tenant = await ctx.db.get(args.tenantId);
    if (!tenant) return false;

    // 检查用户是否是租户成员
    const membership = await ctx.db
      .query('tenant_members')
      .withIndex('by_user_tenant', (q) =>
        q.eq('user', user._id).eq('tenant', args.tenantId)
      )
      .first();

    return !!membership;
  },
});
```

### 搜索与全文检索

```typescript
// convex/functions/search.ts
import { query } from '../_generated/server';
import { v } from 'convex/values';

export const search = query({
  args: {
    query: v.string(),
    type: v.optional(v.union(
      v.literal('all'),
      v.literal('posts'),
      v.literal('users')
    )),
  },
  handler: async (ctx, args) => {
    const lowerQuery = args.query.toLowerCase().trim();

    if (lowerQuery.length < 2) {
      return { posts: [], users: [] };
    }

    const results = {
      posts: [] as any[],
      users: [] as any[],
    };

    // 搜索帖子
    if (args.type === 'all' || args.type === 'posts') {
      const allPosts = await ctx.db
        .query('posts')
        .withIndex('by_published', (q) => q.eq('published', true))
        .collect();

      results.posts = allPosts.filter((post) =>
        post.title.toLowerCase().includes(lowerQuery) ||
        post.body.toLowerCase().includes(lowerQuery)
      ).slice(0, 10);
    }

    // 搜索用户
    if (args.type === 'all' || args.type === 'users') {
      const allUsers = await ctx.db.query('users').collect();

      results.users = allUsers.filter((user) =>
        user.name?.toLowerCase().includes(lowerQuery) ||
        user.email.toLowerCase().includes(lowerQuery)
      ).slice(0, 10);
    }

    return results;
  },
});

// 高级搜索（带排序）
export const advancedSearch = query({
  args: {
    query: v.string(),
    filters: v.optional(v.object({
      author: v.optional(v.id('users')),
      minLikes: v.optional(v.number()),
      tags: v.optional(v.array(v.string())),
    })),
    sortBy: v.optional(v.union(
      v.literal('relevance'),
      v.literal('date'),
      v.literal('likes')
    )),
    limit: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    let posts = await ctx.db
      .query('posts')
      .withIndex('by_published', (q) => q.eq('published', true))
      .collect();

    // 过滤
    posts = posts.filter((post) => {
      const matchesQuery = !args.query ||
        post.title.toLowerCase().includes(args.query.toLowerCase()) ||
        post.body.toLowerCase().includes(args.query.toLowerCase());

      const matchesAuthor = !args.filters?.author ||
        post.author === args.filters.author;

      const matchesLikes = !args.filters?.minLikes ||
        post.likes >= args.filters.minLikes;

      return matchesQuery && matchesAuthor && matchesLikes;
    });

    // 排序
    switch (args.sortBy) {
      case 'date':
        posts.sort((a, b) => b.createdAt - a.createdAt);
        break;
      case 'likes':
        posts.sort((a, b) => b.likes - a.likes);
        break;
      default:
        // relevance - 按匹配度排序
        posts.sort((a, b) => {
          const aTitle = a.title.toLowerCase().includes(args.query.toLowerCase()) ? 2 : 0;
          const bTitle = b.title.toLowerCase().includes(args.query.toLowerCase()) ? 2 : 0;
          return (bTitle + b.likes) - (aTitle + a.likes);
        });
    }

    return posts.slice(0, args.limit ?? 20);
  },
});
```

### 审计日志实现

```typescript
// convex/audit.ts
import { internalMutation } from './_generated/server';
import { v } from 'convex/values';

export const logAudit = internalMutation({
  args: {
    userId: v.optional(v.id('users')),
    action: v.string(),
    entity: v.string(),
    entityId: v.string(),
    changes: v.optional(v.any()),
    metadata: v.optional(v.any()),
  },
  handler: async (ctx, args) => {
    return await ctx.db.insert('audit_logs', {
      ...args,
      timestamp: Date.now(),
    });
  },
});

// 在 mutation 中使用审计日志
export const updateUser = mutation({
  args: {
    userId: v.id('users'),
    name: v.optional(v.string()),
    email: v.optional(v.string()),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    const user = await ctx.db.get(args.userId);

    if (!user) {
      throw new Error('User not found');
    }

    const changes: Record<string, { from: unknown; to: unknown }> = {};
    const updates: Record<string, unknown> = {};

    if (args.name !== undefined && args.name !== user.name) {
      changes.name = { from: user.name, to: args.name };
      updates.name = args.name;
    }

    if (args.email !== undefined && args.email !== user.email) {
      changes.email = { from: user.email, to: args.email };
      updates.email = args.email;
    }

    if (Object.keys(updates).length === 0) {
      return { success: true, changes: {} };
    }

    updates.updatedAt = Date.now();

    await ctx.db.patch(args.userId, updates);

    // 记录审计日志
    if (Object.keys(changes).length > 0) {
      const actingUser = identity
        ? await ctx.db
            .query('users')
            .withIndex('by_email', (q) => q.eq('email', identity.email!))
            .first()
        : null;

      await ctx.db.insert('audit_logs', {
        userId: actingUser?._id,
        action: 'UPDATE',
        entity: 'user',
        entityId: args.userId,
        changes,
        timestamp: Date.now(),
      });
    }

    return { success: true, changes };
  },
});
```

### 缓存策略

```typescript
// convex/cache.ts
import { query } from '../_generated/server';
import { v } from 'convex/values';

// 带缓存的查询（使用内存缓存）
const cache = new Map<string, { data: unknown; expiresAt: number }>();

export const cachedStats = query({
  args: {},
  handler: async (ctx) => {
    const cacheKey = 'global_stats';
    const cached = cache.get(cacheKey);

    // 检查缓存是否有效
    if (cached && cached.expiresAt > Date.now()) {
      return cached.data;
    }

    // 计算统计数据
    const users = await ctx.db.query('users').collect();
    const posts = await ctx.db
      .query('posts')
      .withIndex('by_published', (q) => q.eq('published', true))
      .collect();

    const stats = {
      totalUsers: users.length,
      totalPosts: posts.length,
      totalLikes: posts.reduce((sum, p) => sum + p.likes, 0),
      generatedAt: Date.now(),
    };

    // 更新缓存（5分钟过期）
    cache.set(cacheKey, {
      data: stats,
      expiresAt: Date.now() + 5 * 60 * 1000,
    });

    return stats;
  },
});

// 清除缓存
export const invalidateCache = internalMutation({
  args: { key: v.string() },
  handler: async (ctx, args) => {
    cache.delete(args.key);
    return { success: true };
  },
});
```

### 国际化支持

```typescript
// convex/i18n.ts
import { query } from '../_generated/server';
import { v } from 'convex/values';

// 翻译存储
const translations = {
  en: {
    welcome: 'Welcome to our app',
    posts_count: '{{count}} posts',
    created_at: 'Created at {{date}}',
  },
  zh: {
    welcome: '欢迎使用我们的应用',
    posts_count: '{{count}} 篇文章',
    created_at: '创建于 {{date}}',
  },
  ja: {
    welcome: 'アプリへようこそ',
    posts_count: '{{count}}件の記事',
    created_at: '{{date}}に作成',
  },
};

export const translate = query({
  args: {
    key: v.string(),
    locale: v.optional(v.union(
      v.literal('en'),
      v.literal('zh'),
      v.literal('ja')
    )),
    params: v.optional(v.record(v.string(), v.string())),
  },
  handler: async (ctx, args) => {
    const locale = args.locale ?? 'en';
    let text = translations[locale]?.[args.key] ?? translations['en']?.[args.key] ?? args.key;

    // 替换参数
    if (args.params) {
      for (const [key, value] of Object.entries(args.params)) {
        text = text.replace(new RegExp(`{{${key}}}`, 'g'), value);
      }
    }

    return { text, locale };
  },
});
```

### Webhook 与集成

```typescript
// convex/webhooks.ts
import { action } from '../_generated/server';
import { v } from 'convex/values';

// GitHub Webhook
export const githubWebhook = action({
  args: {
    body: v.any(),
    headers: v.record(v.string(), v.string()),
  },
  handler: async (ctx, args) => {
    const signature = args.headers['x-hub-signature-256'];

    // 验证 GitHub 签名
    const crypto = await import('crypto');
    const hmac = crypto.createHmac('sha256', process.env.GITHUB_WEBHOOK_SECRET!);
    hmac.update(JSON.stringify(args.body));
    const expectedSig = 'sha256=' + hmac.digest('hex');

    if (signature !== expectedSig) {
      throw new Error('Invalid signature');
    }

    const event = args.headers['x-github-event'];

    switch (event) {
      case 'push': {
        const { repository, commits } = args.body;

        // 记录推送
        await ctx.db.insert('deployments', {
          type: 'github_push',
          repository: repository.full_name,
          branch: repository.default_branch,
          commit: commits?.[0]?.id,
          timestamp: Date.now(),
        });
        break;
      }

      case 'pull_request': {
        const { action, pull_request } = args.body;

        if (action === 'closed' && pull_request.merged) {
          // 自动部署
          await ctx.db.insert('deployments', {
            type: 'github_merge',
            repository: repository.full_name,
            branch: pull_request.head.ref,
            commit: pull_request.head.sha,
            timestamp: Date.now(),
          });
        }
        break;
      }
    }

    return { received: true };
  },
});

// Slack 通知
export const slackNotify = action({
  args: {
    channel: v.string(),
    message: v.string(),
    blocks: v.optional(v.array(v.any())),
  },
  handler: async (ctx, args) => {
    const response = await fetch('https://slack.com/api/chat.postMessage', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.SLACK_BOT_TOKEN}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        channel: args.channel,
        text: args.message,
        blocks: args.blocks,
      }),
    });

    if (!response.ok) {
      throw new Error('Failed to send Slack notification');
    }

    return { success: true };
  },
});
```

### 性能监控

```typescript
// convex/metrics.ts
import { internalMutation, query } from './_generated/server';
import { v } from 'convex/values';

// 记录指标
export const recordMetric = internalMutation({
  args: {
    name: v.string(),
    value: v.number(),
    tags: v.optional(v.record(v.string(), v.string())),
  },
  handler: async (ctx, args) => {
    return await ctx.db.insert('metrics', {
      name: args.name,
      value: args.value,
      tags: args.tags ?? {},
      timestamp: Date.now(),
    });
  },
});

// 查询指标
export const queryMetrics = query({
  args: {
    name: v.string(),
    from: v.number(),
    to: v.optional(v.number()),
    aggregation: v.optional(v.union(
      v.literal('sum'),
      v.literal('avg'),
      v.literal('min'),
      v.literal('max'),
      v.literal('count')
    )),
  },
  handler: async (ctx, args) => {
    const to = args.to ?? Date.now();

    const metrics = await ctx.db
      .query('metrics')
      .filter((q) =>
        q.and(
          q.eq(q.field('name'), args.name),
          q.gte(q.field('timestamp'), args.from),
          q.lte(q.field('timestamp'), to)
        )
      )
      .collect();

    if (metrics.length === 0) {
      return { value: 0, count: 0 };
    }

    const values = metrics.map((m) => m.value);

    switch (args.aggregation ?? 'sum') {
      case 'sum':
        return { value: values.reduce((a, b) => a + b, 0), count: values.length };
      case 'avg':
        return { value: values.reduce((a, b) => a + b, 0) / values.length, count: values.length };
      case 'min':
        return { value: Math.min(...values), count: values.length };
      case 'max':
        return { value: Math.max(...values), count: values.length };
      case 'count':
        return { value: values.length, count: values.length };
    }
  },
});
```

### 完整项目结构

```
my-convex-app/
├── convex/
│   ├── _generated/
│   │   ├── server.ts
│   │   └── api.ts
│   ├── schema.ts
│   ├── auth.ts
│   ├── functions/
│   │   ├── users.ts
│   │   ├── posts.ts
│   │   ├── comments.ts
│   │   ├── chat.ts
│   │   ├── search.ts
│   │   ├── storage.ts
│   │   └── notifications.ts
│   ├── lib/
│   │   ├── validators.ts
│   │   ├── i18n.ts
│   │   ├── metrics.ts
│   │   └── cache.ts
│   └── convex.config.ts
├── src/
│   ├── lib/
│   │   └── convex.ts
│   ├── components/
│   │   ├── PostList.tsx
│   │   ├── CommentSection.tsx
│   │   └── ChatRoom.tsx
│   └── app/
│       ├── layout.tsx
│       └── page.tsx
├── package.json
└── tsconfig.json
```

---

## Convex 与 AI 应用

### AI 生成内容

```typescript
// convex/functions/ai.ts
import { action } from '../_generated/server';
import { v } from 'convex/values';

export const generatePost = action({
  args: {
    topic: v.string(),
    style: v.optional(v.union(
      v.literal('informative'),
      v.literal('casual'),
      v.literal('technical')
    )),
  },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error('Not authenticated');

    // 调用 OpenAI
    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'gpt-4o',
        messages: [
          {
            role: 'system',
            content: `You are a blog post writer. Write ${args.style ?? 'informative'} content.`
          },
          {
            role: 'user',
            content: `Write a blog post about: ${args.topic}`
          }
        ],
        max_tokens: 2000,
      }),
    });

    const data = await response.json();
    const content = data.choices[0].message.content;

    // 生成 slug
    const slug = args.topic
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, '-')
      .replace(/^-|-$/g, '');

    // 获取用户
    const user = await ctx.db
      .query('users')
      .withIndex('by_email', (q) => q.eq('email', identity.email!))
      .first();

    if (!user) throw new Error('User not found');

    // 创建帖子
    const postId = await ctx.db.insert('posts', {
      title: args.topic,
      slug: `${slug}-${Date.now()}`,
      body: content,
      published: false,
      author: user._id,
      likes: 0,
      createdAt: Date.now(),
      updatedAt: Date.now(),
    });

    return { postId, content };
  },
});
```

### RAG 实现

```typescript
export const queryKnowledgeBase = action({
  args: {
    query: v.string(),
    topK: v.optional(v.number()),
  },
  handler: async (ctx, args) => {
    // 生成查询向量
    const embedResponse = await fetch('https://api.openai.com/v1/embeddings', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'text-embedding-3-small',
        input: args.query,
      }),
    });

    const { data: [{ embedding }] } = await embedResponse.json();

    // 在向量数据库中搜索（简化示例）
    const chunks = await ctx.db.query('knowledge_chunks').collect();

    const similarities = chunks.map((chunk) => ({
      chunk,
      score: cosineSimilarity(embedding, chunk.embedding),
    }));

    similarities.sort((a, b) => b.score - a.score);

    const topResults = similarities.slice(0, args.topK ?? 5);

    // 使用上下文生成回答
    const context = topResults.map((r) => r.chunk.content).join('\n\n');

    const chatResponse = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model: 'gpt-4o',
        messages: [
          {
            role: 'system',
            content: `Based on the following context, answer the user's question.\n\nContext:\n${context}`
          },
          { role: 'user', content: args.query }
        ],
      }),
    });

    const chatData = await chatResponse.json();
    const answer = chatData.choices[0].message.content;

    return {
      answer,
      sources: topResults.map((r) => ({
        content: r.chunk.content.substring(0, 200),
        score: r.score,
      })),
    };
  },
});
```

---

> [!TIP]
> Convex 的响应式查询让实时应用开发变得前所未有的简单。合理使用 Action、Mutation 和 Query，可以构建出高性能、易维护的全栈应用。

---
