---
date: 2026-04-24
tags:
  - 数据库
  - Supabase
  - Firebase替代
  - BaaS
---

# Supabase 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Supabase 架构深度解析、Auth 认证系统、Database RLS 深度指南、Storage 存储、Edge Functions、Realtime 实时订阅、pgvector 向量搜索，以及在 AI 应用中的 RAG 实战场景。

---

## 目录

1. [[#Supabase 概述与核心定位]]
2. [[#架构解析]]
3. [[#Auth 认证系统]]
4. [[#Database 与 RLS 深度指南]]
5. [[#Storage 存储服务]]
6. [[#Edge Functions]]
7. [[#Realtime 实时订阅]]
8. [[#Supabase vs Firebase vs PocketBase 对比]]
9. [[#pgvector 向量搜索与 AI 应用]]
10. [[#Prisma 与 Drizzle 集成]]
11. [[#定价与免费额度]]
12. [[#从 Firebase 迁移指南]]
13. [[#选型建议与实战场景]]

---

## Supabase 概述与核心定位

### 什么是 Supabase

Supabase 是一个开源的 Firebase 替代方案，由 Supabase 公司于 2020 年正式发布。它基于 PostgreSQL 构建，提供了实时数据库、身份认证、文件存储、边缘函数等开箱即用的后端服务。与 Firebase 不同的是，Supabase 的核心理念是「让 PostgreSQL 变得更简单」，它并不试图替代 SQL，而是将 SQL 的能力用现代化的接口暴露出来。

Supabase 的创始人 Paul Copplestone 曾经这样描述 Supabase 的愿景：「我们不是在做另一个 Firebase，我们是让 PostgreSQL 变得像 Firebase 一样易于使用，同时保留 PostgreSQL 的全部能力。」这句话很好地概括了 Supabase 的核心定位——它不是一个小众的玩具，而是一个真正为生产环境设计的、同时兼顾开发体验和企业级能力的后端即服务平台。

在 2026 年的今天，Supabase 已经发展成为一个完整的后端即服务（BaaS）平台，支持超过 100 万个活跃项目，被广泛应用于 Web 应用、移动应用、SaaS 产品以及 AI 应用中。它的开源特性意味着你可以完全掌控自己的数据，没有任何供应商锁定的风险——你可以在本地运行 Supabase（使用 Docker），也可以使用官方托管服务，或者部署到自己的云基础设施上。

### 为什么选择 Supabase 而非其他方案

对于 Vibe Coding 场景（即快速原型开发和小团队敏捷开发），选择一个合适的后端服务至关重要。Supabase 在这个场景下具有独特的优势。

**与 Firebase 相比**，Supabase 使用的是结构化的 SQL 数据库而非 NoSQL 文档数据库。这意味着你可以使用 SQL 的全部能力——复杂的 JOIN、聚合查询、事务、视图、存储过程等等。对于习惯了关系型数据库的开发者来说，这种模式更加直观和可控。此外，Firebase 的定价在数据量大的时候会变得非常昂贵，而 Supabase 的定价模式对开发者更加友好。

**与 PocketBase 相比**，PocketBase 是一个更轻量级的嵌入式解决方案，适合单机部署的小型应用。Supabase 则是一个完整的云原生解决方案，支持水平扩展、高可用、以及更丰富的企业级功能。如果你的应用需要处理高并发、支持多租户、或者需要托管在云端，Supabase 是更合适的选择。

**与传统的 PostgreSQL + 自建后端相比**，Supabase 提供了开箱即用的认证、实时订阅、文件存储等组件，大大减少了开发工作量。你不需要自己实现 JWT 认证逻辑、不需要配置 WebSocket 服务、不需要搭建 S3 兼容的存储系统——所有这些 Supabase 都帮你搞定了。

### Supabase 在 AI 时代的价值

在 AI 应用大爆发的 2026 年，Supabase 展现了独特的技术价值。首先，它原生支持 pgvector 扩展，这意味着你可以在同一个数据库中存储结构化数据和向量嵌入，无需维护单独的向量数据库。其次，Supabase 的 Row Level Security（RLS）提供了细粒度的数据访问控制，这在 AI 应用中保护用户数据隐私至关重要。第三，Edge Functions 基于 Deno 运行时，支持 TypeScript，可以轻松调用 AI API 并将结果存储回数据库。

### 市场份额与生态

Supabase 在 BaaS 市场中的份额持续增长，特别是在 2024-2026 年间，随着 AI 应用开发热潮的到来，越来越多的 AI 原生应用选择 Supabase 作为后端。GitHub 上 Supabase 的 star 数已经超过 70,000，社区活跃度在开源 BaaS 解决方案中处于领先地位。

Supabase 的适用场景非常广泛：

| 场景类型 | 推荐程度 | 说明 |
|---------|---------|------|
| 快速原型开发 | ⭐⭐⭐⭐⭐ | 开箱即用的后端服务，大幅提升开发效率 |
| 移动应用后端 | ⭐⭐⭐⭐⭐ | 支持 Flutter、React Native、iOS、Android |
| SaaS 多租户应用 | ⭐⭐⭐⭐⭐ | RLS 实现数据隔离，无需复杂的后端逻辑 |
| AI RAG 应用 | ⭐⭐⭐⭐ | pgvector 向量搜索，原生支持 AI 场景 |
| 小型团队产品 | ⭐⭐⭐⭐⭐ | 减少运维负担，让团队专注于产品 |
| 企业级应用 | ⭐⭐⭐⭐ | 需要评估自托管 vs 托管服务的成本 |

---

## 架构解析

### 整体技术架构

Supabase 的架构设计体现了现代云原生应用的核心理念——将复杂的功能封装成可组合的服务，同时保持每个组件的独立性和可扩展性。理解 Supabase 的架构对于正确使用它至关重要。

```
┌─────────────────────────────────────────────────────────────────┐
│                        Supabase 架构                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                     客户端层                              │  │
│  │   JavaScript SDK │ Flutter SDK │ Swift SDK │ Kotlin SDK   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                  API Gateway 层                           │  │
│  │         (Kong API Gateway + 自定义路由规则)                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────┬─────────────────────┬───────────────────┐  │
│  │   PostgREST   │      GoTrue         │   Edge Functions   │  │
│  │  (REST API)   │    (Auth/JWT)       │     (Deno)         │  │
│  │               │                     │                    │  │
│  │ 自动生成 CRUD  │ 身份认证与授权       │ TypeScript 边缘函数 │  │
│  │ 过滤/分页     │ 第三方 OAuth         │ REST API           │  │
│  │ Schema 感知   │ 魔法链接登录         │ 数据库访问          │  │
│  └───────────────┴─────────────────────┴───────────────────┘  │
│                              │                                   │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │                    PostgreSQL 核心                         │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐  │  │
│  │  │ pgvector │ │ PostGIS  │ │  JSONB   │ │ pg_net   │  │  │
│  │  │ 向量搜索  │ │ 地理空间  │ │ JSON支持  │ │ HTTP请求  │  │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘  │  │
│  └──────────────────────────────────────────────────────────┘  │
│                              │                                   │
│  ┌───────────────┬─────────────────────┬───────────────────┐  │
│  │   Realtime    │      Storage        │     Studio        │  │
│  │   Engine      │    (S3 兼容)        │   (Dashboard)     │  │
│  │  WebSocket    │  文件上传/下载       │  数据库管理        │  │
│  │  变更订阅     │  图片处理           │  API 文档         │  │
│  └───────────────┴─────────────────────┴───────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 核心组件详解

**PostgREST** 是 Supabase REST API 的核心引擎。它读取你的 PostgreSQL 数据库 schema，然后自动生成符合 RESTful 规范的 API。神奇的是，PostgREST 不只是简单地暴露数据库表——它理解表之间的关系、外键约束、RLS 策略，并据此生成智能的 API。例如，如果你有一个 `posts` 表和一个 `comments` 表，PostgREST 会自动生成 `/posts/:id/comments` 这样的嵌套路由。

**GoTrue** 是 Supabase 的认证引擎，最初作为独立项目开发，后来被 Supabase 收购并集成到平台中。GoTrue 基于 JWT（JSON Web Token）实现无状态的认证，支持邮箱密码登录、第三方 OAuth（Google、GitHub、Facebook 等）、魔法链接登录、电话验证码登录等多种认证方式。所有认证相关的逻辑——包括令牌刷新、会话管理、密码重置——都由 GoTrue 自动处理。

**Realtime Engine** 构建在 PostgreSQL 的逻辑复制功能之上。当你启用 Realtime 功能时，Supabase 会在后台开启逻辑复制，然后通过 WebSocket 将数据库变更推送给连接的客户端。这个机制非常高效，因为它利用了 PostgreSQL 原生的变更捕获能力，而不是轮询数据库。

**Edge Functions** 基于 Deno 运行时，这是 Node.js 的一个现代替代品。Deno 以安全性著称——默认情况下，Deno 脚本无法访问文件系统、网络或环境变量，除非明确授权。Edge Functions 支持 TypeScript，可以部署在全球分布的边缘节点上，实现低延迟的 API 响应。

### PostgreSQL Extensions 生态

Supabase 的强大之处在于它充分利用了 PostgreSQL 的扩展生态。以下是几个最有价值的扩展：

**pgvector** 是 AI 向量搜索的核心扩展，它允许你在 PostgreSQL 中存储和搜索向量嵌入。pgvector 支持多种距离度量（余弦相似度、欧氏距离、L2 距离等），并提供了高效的近似最近邻（ANN）搜索算法。对于构建 RAG（检索增强生成）系统来说，pgvector 是 PostgreSQL 原生支持的解决方案。

**PostGIS** 提供了完整的地理空间数据支持，包括点、线、面等几何类型，以及空间查询、距离计算、缓冲区分析等 GIS 功能。在需要地理位置功能的场景下（如附近的人、附近的商家），PostGIS 可以大大简化开发工作。

**pg_net** 是一个神奇的扩展，它允许你在数据库内部发起 HTTP 请求。这意味着你可以在数据库触发器中调用外部 API，例如在用户注册时自动调用邮件服务、在订单创建时通知物流系统。这是一个非常有用的功能，因为它让你可以在数据库层面实现业务逻辑，而不需要额外的应用层代码。

**uuid-ossp** 提供 UUID 生成功能，支持多种 UUID 版本，包括基于时间戳的 UUIDv1、基于随机数的 UUIDv4 等。

---

## Auth 认证系统

### 认证方式概览

Supabase Auth 提供了丰富的认证方式，几乎涵盖了所有常见的用户认证场景。理解这些认证方式的适用场景对于正确设计应用认证流程至关重要。

**邮箱密码认证**是最基础的认证方式。用户使用邮箱和密码注册和登录。Supabase 会自动处理密码哈希（使用 bcrypt）、邮箱验证（发送验证邮件）、密码重置等功能。对于大多数应用来说，邮箱密码认证是必备的基础能力。

**第三方 OAuth 认证**允许用户使用已有的社交账号登录。Supabase 原生支持 Google、GitHub、Facebook、Twitter、Apple、Azure AD、Slack 等众多 OAuth 提供商。配置 OAuth 只需要在 Supabase Dashboard 中填写相应的 Client ID 和 Client Secret，Supabase 会处理整个 OAuth 流程并自动创建用户记录。

**魔法链接登录**是一种无密码认证方式。用户输入邮箱地址，系统发送一封包含登录链接的邮件，用户点击链接后自动登录。这种方式对于移动应用特别友好，因为它避免了输入密码的麻烦，同时也提升了安全性（没有密码可以泄露）。

**电话验证码登录**通过短信发送一次性验证码来验证用户身份。Supabase 与 Twilio 集成，支持全球范围内的短信发送。这种方式在国内应用中使用较少，但在海外市场和一些特定场景下很有价值。

### 认证代码实战

下面是一个完整的认证流程示例，展示了如何在客户端使用 Supabase Auth：

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  'https://your-project.supabase.co',
  'your-anon-key'
)

// 邮箱密码注册
async function signUp(email: string, password: string) {
  const { data, error } = await supabase.auth.signUp({
    email,
    password,
    options: {
      data: {
        // 自定义用户元数据
        username: 'new_user',
        avatar_url: null
      },
      // 是否需要邮箱验证
      emailRedirectTo: 'https://yourapp.com/callback'
    }
  })
  
  if (error) {
    console.error('注册失败:', error.message)
    return null
  }
  
  console.log('注册成功:', data.user)
  return data.user
}

// 邮箱密码登录
async function signIn(email: string, password: string) {
  const { data, session, error } = await supabase.auth.signInWithPassword({
    email,
    password
  })
  
  if (error) {
    console.error('登录失败:', error.message)
    return null
  }
  
  // session 包含 access_token 和 refresh_token
  console.log('登录成功:', session)
  return { user: data.user, session }
}

// Google OAuth 登录
async function signInWithGoogle() {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: 'https://yourapp.com/callback'
    }
  })
  
  if (error) {
    console.error('Google 登录失败:', error.message)
    return
  }
  
  // data.url 包含 OAuth 重定向 URL
  window.location.href = data.url
}

// 魔法链接登录
async function signInWithMagicLink(email: string) {
  const { error } = await supabase.auth.signInWithOtp({
    email,
    options: {
      emailRedirectTo: 'https://yourapp.com/callback'
    }
  })
  
  if (error) {
    console.error('发送魔法链接失败:', error.message)
    return
  }
  
  console.log('魔法链接已发送到邮箱')
}

// 登出
async function signOut() {
  const { error } = await supabase.auth.signOut()
  if (error) {
    console.error('登出失败:', error.message)
  }
}

// 获取当前用户
function getCurrentUser() {
  const { data: { user } } = supabase.auth.getUser()
  return user
}

// 监听认证状态变化
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    console.log('用户已登录:', session?.user)
  } else if (event === 'SIGNED_OUT') {
    console.log('用户已登出')
  } else if (event === 'TOKEN_REFRESHED') {
    console.log('Token 已刷新')
  } else if (event === 'USER_UPDATED') {
    console.log('用户信息已更新')
  }
})
```

### JWT 与会话管理

Supabase Auth 使用 JWT 作为无状态认证的载体。理解 JWT 的工作原理对于正确使用 Supabase Auth 至关重要。

当用户登录成功时，Supabase 返回一个包含 `access_token` 和 `refresh_token` 的 session 对象。`access_token` 是一个 JWT，包含用户的基本信息和权限声明，有效期默认为 1 小时。`refresh_token` 用于获取新的 access_token，有效期默认为 30 天。

Supabase 的客户端 SDK 会自动管理 token 的刷新。当 access_token 即将过期时，SDK 会自动使用 refresh_token 获取新的 token，用户完全感知不到这个过程。如果 refresh_token 也过期了（超过 30 天未使用应用），用户需要重新登录。

在服务器端验证 JWT 非常简单。Supabase 的 JWT 包含用户的 ID（`sub` claim）和元数据，服务器可以使用 Supabase 提供的公钥验证 token 的签名，从而确认用户身份：

```typescript
// 在 Edge Functions 或 API Routes 中验证 JWT
import { createClient } from '@supabase/supabase-js'

// 验证 token 并获取用户信息
async function verifyToken(request: Request) {
  const authHeader = request.headers.get('Authorization')
  
  if (!authHeader?.startsWith('Bearer ')) {
    return new Response(JSON.stringify({ error: 'Missing token' }), {
      status: 401
    })
  }
  
  const token = authHeader.substring(7)
  
  // 使用 service_role 密钥创建管理员客户端
  const supabaseAdmin = createClient(
    process.env.SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!
  )
  
  // 验证 token
  const { data: { user }, error } = await supabaseAdmin.auth.getUser(token)
  
  if (error || !user) {
    return new Response(JSON.stringify({ error: 'Invalid token' }), {
      status: 401
    })
  }
  
  console.log('已验证的用户:', user.id)
  return user
}
```

---

## Database 与 RLS 深度指南

### PostgreSQL 的核心优势

Supabase 的数据库基于 PostgreSQL，这意味着你拥有了一个功能完备的、生产级别可靠的关系型数据库。PostgreSQL 不仅仅是「一个可以存储数据的 SQL 数据库」，它实际上是一个极其强大的数据平台。

首先，PostgreSQL 的 ACID 事务支持确保了你的数据操作是原子的、一致的、隔离的、持久的。无论是转账操作、订单创建还是批量数据更新，PostgreSQL 都能保证数据的完整性。这是 NoSQL 数据库（如 Firebase Realtime Database）所不能提供的保障。

其次，PostgreSQL 的 JSONB 类型让你可以在关系型数据库中存储半结构化数据。JSONB 不仅存储 JSON 数据，还会解析 JSON 并建立索引，这意味着你可以用 SQL 查询 JSON 中的字段，同时享受关系型数据库的所有优势。这在处理用户配置、事件日志、灵活 Schema 等场景下非常有用。

第三，PostgreSQL 的复杂查询能力是无与伦比的。子查询、公用表表达式（CTE）、窗口函数、递归查询——这些高级 SQL 特性让你可以用一条 SQL 语句解决非常复杂的数据处理问题，而不需要在应用层写大量的代码。

### Row Level Security（RLS）详解

RLS 是 Supabase 安全的核心。它允许你定义策略（Policy），精确控制哪些用户可以看到、修改哪些数据。RLS 的神奇之处在于：策略在数据库层面强制执行，即使你的应用代码有漏洞，RLS 策略仍然保护着数据安全。

RLS 的基本概念很简单：每个表可以启用 RLS，启用后，默认情况下用户看不到任何数据，除非有策略明确允许。然后你可以创建策略，指定哪些行可以被 SELECT、INSERT、UPDATE 或 DELETE。

```sql
-- 启用 RLS
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;

-- 创建读取策略：用户只能读取自己的 posts
CREATE POLICY "Users can view own posts"
ON posts
FOR SELECT
USING (auth.uid() = user_id);

-- 创建插入策略：用户只能创建自己的 posts
CREATE POLICY "Users can insert own posts"
ON posts
FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- 创建更新策略：用户只能更新自己的 posts
CREATE POLICY "Users can update own posts"
ON posts
FOR UPDATE
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);

-- 创建删除策略：用户只能删除自己的 posts
CREATE POLICY "Users can delete own posts"
ON posts
FOR DELETE
USING (auth.uid() = user_id);
```

### RLS 高级模式

在实际应用中，RLS 策略往往比简单的 `auth.uid() = user_id` 复杂得多。下面是几个常见的高级模式。

**基于角色的访问控制**允许你根据用户的角色（admin、moderator、user 等）来决定访问权限：

```sql
-- 在 profiles 表中添加 role 字段
ALTER TABLE profiles ADD COLUMN role TEXT DEFAULT 'user';

-- 管理员可以查看所有用户
CREATE POLICY "Admins can view all profiles"
ON profiles
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- 普通用户只能查看自己的 profile
CREATE POLICY "Users can view own profile"
ON profiles
FOR SELECT
USING (auth.uid() = id);
```

**多租户数据隔离**是 SaaS 应用的核心需求。每个租户只能访问自己的数据，RLS 可以优雅地实现这一点：

```sql
-- 假设有一个 organizations 表和一个多租户应用
-- 每个用户属于一个 organization

-- 用户只能访问自己组织的成员
CREATE POLICY "Users can view org members"
ON users
FOR SELECT
USING (
  organization_id IN (
    SELECT organization_id FROM users WHERE id = auth.uid()
  )
);

-- 组织管理员可以管理自己组织的成员
CREATE POLICY "Org admins can manage members"
ON users
FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM users u
    JOIN organizations o ON u.organization_id = o.id
    WHERE u.id = auth.uid()
    AND o.role = 'admin'
    AND u.organization_id = users.organization_id
  )
);
```

**公开与私有内容混合**是内容平台常见的需求。有些内容是公开的，有些是付费会员专享的：

```sql
-- posts 表有 is_public 和 author_id 字段

-- 所有人都可以看到公开的 posts
CREATE POLICY "Anyone can view public posts"
ON posts
FOR SELECT
USITING (is_public = true);

-- 作者可以看到自己的所有 posts（包括私有的）
CREATE POLICY "Authors can view own posts"
ON posts
FOR SELECT
USING (auth.uid() = author_id);

-- 付费会员可以看到所有 posts
CREATE POLICY "Subscribers can view all posts"
ON posts
FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM subscriptions s
    WHERE s.user_id = auth.uid()
    AND s.status = 'active'
  )
);
```

### 数据库 Schema 设计最佳实践

一个设计良好的数据库 Schema 是应用稳定性的基础。以下是 Supabase 应用中推荐的 Schema 设计模式。

**使用 UUID 作为主键**。Supabase 强烈推荐使用 UUID 而非自增整数作为主键。UUID 的优势在于：无法通过枚举猜测其他用户的数据、支持分布式生成、避免主从同步时的 ID 冲突：

```sql
-- 使用 uuid-ossp 扩展生成 UUID
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email TEXT UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 创建 updated_at 自动更新触发器
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_updated_at
  BEFORE UPDATE ON users
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

**使用软删除而非硬删除**。在大多数应用中，直接删除数据并不是一个好主意——你可能需要审计历史数据、需要支持「回收站」功能、需要保持数据完整性。软删除通过一个 `deleted_at` 字段来标记已删除的记录：

```sql
ALTER TABLE posts ADD COLUMN deleted_at TIMESTAMPTZ;

-- 创建视图只显示未删除的 posts
CREATE VIEW active_posts AS
SELECT * FROM posts WHERE deleted_at IS NULL;

-- 创建只显示用户自己的 active posts 的视图
CREATE VIEW my_posts AS
SELECT * FROM active_posts WHERE user_id = auth.uid();

-- 删除操作变为更新操作
CREATE OR REPLACE FUNCTION soft_delete_post(post_id UUID)
RETURNS void AS $$
BEGIN
  UPDATE posts SET deleted_at = NOW() WHERE id = post_id AND user_id = auth.uid();
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

**使用 Schema 隔离功能模块**。随着应用规模的增长，将不同功能域的表放在不同的 schema 中可以提高可维护性：

```sql
-- 创建功能 schema
CREATE SCHEMA IF NOT EXISTS blog;
CREATE SCHEMA IF NOT EXISTS ecommerce;
CREATE SCHEMA IF NOT EXISTS analytics;

-- 在 blog schema 中创建表
CREATE TABLE blog.posts (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  title TEXT NOT NULL,
  content TEXT,
  published_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Grant 权限让用户可以访问 blog schema
GRANT USAGE ON SCHEMA blog TO authenticated;
GRANT SELECT ON ALL TABLES IN SCHEMA blog TO authenticated;
```

---

## Storage 存储服务

### Storage 概述

Supabase Storage 是 S3 兼容的对象存储服务，用于存储文件、图片、视频、文档等二进制数据。它与 Supabase Auth 和 Database 深度集成，提供了开箱即用的文件访问控制。

Storage 的核心概念是 Bucket（存储桶）和 File（文件）。Bucket 类似于文件夹，用于组织文件；File 是实际存储的对象。每个 Bucket 可以设置公开访问或需要认证访问，而 RLS 策略可以进一步精细控制文件访问权限。

### Storage 实战

```typescript
// 上传文件
async function uploadAvatar(userId: string, file: File) {
  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(`${userId}/avatar.jpg`, file, {
      cacheControl: '3600',
      upsert: true  // 覆盖已存在的文件
    })
  
  if (error) {
    console.error('上传失败:', error.message)
    return null
  }
  
  console.log('上传成功:', data.path)
  return data.path
}

// 获取公开文件 URL
function getPublicUrl(path: string) {
  const { data } = supabase.storage
    .from('avatars')
    .getPublicUrl(path)
  
  return data.publicUrl
}

// 下载文件（认证用户）
async function downloadFile(path: string) {
  const { data, error } = await supabase.storage
    .from('documents')
    .download(path)
  
  if (error) {
    console.error('下载失败:', error.message)
    return null
  }
  
  return data
}

// 列出文件
async function listFiles(prefix: string) {
  const { data, error } = await supabase.storage
    .from('documents')
    .list(prefix, {
      limit: 100,
      sortBy: { column: 'created_at', order: 'desc' }
    })
  
  if (error) {
    console.error('列出文件失败:', error.message)
    return []
  }
  
  return data.filter(item => item.id !== null)  // 过滤文件夹
}

// 删除文件
async function deleteFile(path: string) {
  const { error } = await supabase.storage
    .from('documents')
    .remove([path])
  
  if (error) {
    console.error('删除失败:', error.message)
    return false
  }
  
  return true
}
```

### Storage RLS 策略

Storage 的 RLS 策略与数据库 RLS 类似，但针对文件操作进行了适配：

```sql
-- 只有文件所有者可以下载
CREATE POLICY "Owner can download"
ON storage.objects FOR SELECT
USING (auth.uid()::text = (storage.foldername(name))[1]);

-- 只有文件所有者可以上传
CREATE POLICY "Owner can upload"
ON storage.objects FOR INSERT
WITH CHECK (auth.uid()::text = (storage.foldername(name))[1]);

-- 只有文件所有者可以删除
CREATE POLICY "Owner can delete"
ON storage.objects FOR DELETE
USING (auth.uid()::text = (storage.foldername(name))[1]);
```

---

## Edge Functions

### Edge Functions 概述

Edge Functions 是 Supabase 的 serverless 函数服务，基于 Deno 运行时。它们部署在全球分布的边缘节点上，可以实现低延迟的 API 响应。与传统 serverless 不同，Edge Functions 可以直接访问 Supabase 数据库，无需额外的认证开销。

Edge Functions 支持 TypeScript，提供完整的 TypeScript 类型定义。你可以使用熟悉的 npm 包（需要是 Deno 兼容的），也可以使用 Deno 独有的特性，如内置的 Web API 和安全沙箱。

### Edge Functions 实战

```typescript
// supabase/functions/generate-summary/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  try {
    // 从 Authorization header 获取 token
    const authHeader = req.headers.get('Authorization')
    if (!authHeader) {
      return new Response(
        JSON.stringify({ error: 'Missing authorization header' }),
        { status: 401, headers: { 'Content-Type': 'application/json' } }
      )
    }
    
    // 创建 Supabase 客户端
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_ANON_KEY')!,
      {
        global: { headers: { Authorization: authHeader } }
      }
    )
    
    // 获取当前用户
    const { data: { user }, error: userError } = await supabase.auth.getUser()
    if (userError || !user) {
      return new Response(
        JSON.stringify({ error: 'Unauthorized' }),
        { status: 401, headers: { 'Content-Type': 'application/json' } }
      )
    }
    
    // 解析请求体
    const { document_id, text } = await req.json()
    
    // 模拟 AI 摘要生成
    const summary = generateSummary(text)
    
    // 保存摘要到数据库
    const { data, error } = await supabase
      .from('summaries')
      .insert({
        document_id,
        content: summary,
        user_id: user.id
      })
      .select()
      .single()
    
    if (error) {
      throw error
    }
    
    return new Response(
      JSON.stringify({ success: true, summary: data }),
      { status: 200, headers: { 'Content-Type': 'application/json' } }
    )
    
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { status: 500, headers: { 'Content-Type': 'application/json' } }
    )
  }
})

function generateSummary(text: string): string {
  // 这里是简化的摘要逻辑
  // 实际应用中应该调用 AI API
  const sentences = text.split(/[.!?]+/).filter(s => s.trim())
  const firstSentences = sentences.slice(0, 3)
  return firstSentences.join('. ').trim() + '.'
}
```

### 部署 Edge Functions

```bash
# 安装 Supabase CLI
brew install supabase/tap/supabase

# 登录
supabase login

# 初始化项目
supabase init

# 部署函数
supabase functions deploy generate-summary

# 调用函数
curl -X POST https://[project-id].supabase.co/functions/v1/generate-summary \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"document_id": "xxx", "text": "要摘要的文本..."}'
```

---

## Realtime 实时订阅

### Realtime 概述

Supabase Realtime 是构建实时应用的核心功能。它基于 PostgreSQL 的逻辑复制，将数据库变更通过 WebSocket 推送给客户端。Realtime 支持三种类型的实时事件：数据库变更（INSERT、UPDATE、DELETE）、广播（自定义消息）、presence（用户在线状态）。

Realtime 的使用非常简单——你只需要在表上启用 Realtime，然后订阅变更即可。Supabase 会处理底层的逻辑复制、WebSocket 连接管理、客户端重连等复杂问题。

### Realtime 实战

```typescript
// 订阅 posts 表的所有变更
const subscription = supabase
  .channel('posts-changes')
  .on(
    'postgres_changes',
    {
      event: '*',  // INSERT, UPDATE, DELETE, *
      schema: 'public',
      table: 'posts'
    },
    (payload) => {
      console.log('Posts 变更:', payload)
      
      if (payload.eventType === 'INSERT') {
        console.log('新帖子:', payload.new)
      } else if (payload.eventType === 'UPDATE') {
        console.log('更新的帖子:', payload.new)
      } else if (payload.eventType === 'DELETE') {
        console.log('删除的帖子:', payload.old)
      }
    }
  )
  .subscribe()

// 订阅过滤：只接收特定用户的 posts 变更
const filteredSubscription = supabase
  .channel('user-posts-changes')
  .on(
    'postgres_changes',
    {
      event: '*',
      schema: 'public',
      table: 'posts',
      filter: 'user_id=eq.123'  // 只接收 user_id=123 的变更
    },
    (payload) => {
      console.log('特定用户的 Posts 变更:', payload)
    }
  )
  .subscribe()

// 使用广播实现实时聊天
const channel = supabase.channel('chat-room-1')

// 发送消息
function sendMessage(text: string) {
  channel.send({
    type: 'broadcast',
    event: 'message',
    payload: { text, sender: 'current_user', timestamp: Date.now() }
  })
}

// 接收消息
channel
  .on('broadcast', { event: 'message' }, (payload) => {
    console.log('收到消息:', payload.payload)
  })
  .subscribe()

// Presence 功能：追踪用户在线状态
channel
  .on('presence', { event: 'sync' }, () => {
    const state = channel.presenceState()
    console.log('当前在线用户:', Object.keys(state))
  })
  .on('presence', { event: 'join' }, ({ key, newPresences }) => {
    console.log('用户上线:', newPresences)
  })
  .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
    console.log('用户离线:', leftPresences)
  })
  .subscribe(async (status) => {
    if (status === 'SUBSCRIBED') {
      await channel.track({
        user_id: 'current_user_id',
        online_at: new Date().toISOString()
      })
    }
  })

// 清理订阅
function cleanup() {
  subscription.unsubscribe()
  channel.unsubscribe()
}
```

---

## Supabase vs Firebase vs PocketBase 对比

### 核心定位对比

| 维度 | Supabase | Firebase | PocketBase |
|------|----------|----------|------------|
| **数据库类型** | PostgreSQL (关系型) | Firestore (NoSQL) | SQLite (关系型) |
| **开源** | 完全开源 | 闭源 | 完全开源 |
| **定价模型** | 按使用量计费 | 按操作计费 | 免费（自托管） |
| **学习曲线** | 中等（需要 SQL 知识） | 较陡（独特的 NoSQL 模型） | 低 |
| **企业级功能** | 丰富（PostgreSQL 生态） | 一般 | 有限 |
| **可扩展性** | 高 | 高 | 中等 |
| **Vendor Lock-in** | 无（可自托管） | 强 | 无 |

### Firebase 的优势与劣势

Firebase 的优势在于其完整的生态系统：Firestore 的实时数据库、Firestore Storage、Cloud Functions for Firebase、Firebase Analytics、Firebase Crashlytics 等。这些产品无缝集成，形成了一个完整的后端平台。对于移动应用开发来说，Firebase 是一个成熟的选择。

Firebase 的劣势也同样明显。首先是 NoSQL 数据模型的局限性——虽然 Firestore 的文档模型在某些场景下很灵活，但复杂的查询和多表关联会变得非常困难。其次是定价：Firebase 按操作计费，在数据量大或用户多的时候，费用会急剧上升。第三是 Vendor Lock-in：一旦选择了 Firebase，就很难迁移到其他平台。

### PocketBase 的定位

PocketBase 是一个嵌入式 Go 应用，可以打包到你的应用中。它非常轻量，单个二进制文件就可以运行。对于简单的个人项目、原型开发、或者需要完全离线运行的应用来说，PocketBase 是一个不错的选择。

但 PocketBase 的局限性也很明显：它不支持水平扩展、单点部署不适合高流量场景、社区和生态远不如 Supabase 成熟。如果你计划构建一个需要处理大量用户、或者需要高可用性的应用，PocketBase 可能不是最佳选择。

### 选型决策树

```
需要构建的应用类型
    │
    ├─ 个人项目 / 原型 / 需要完全离线运行？
    │     └─ 是 → PocketBase ✓
    │
    ├─ 需要水平扩展 / 高可用 / 多租户 SaaS？
    │     └─ 是 → Supabase ✓
    │
    ├─ 移动应用 / 需要完整的分析/推送/监控生态？
    │     └─ 是 → Firebase ✓
    │
    └─ 其他情况
          ├─ 团队熟悉 SQL → Supabase ✓
          ├─ 团队熟悉 NoSQL → Firebase ✓
          └─ 预算有限 → Supabase (自托管) ✓
```

---

## pgvector 向量搜索与 AI 应用

### pgvector 概述

pgvector 是 PostgreSQL 的扩展，它允许你在 PostgreSQL 中存储和搜索向量嵌入（embeddings）。向量嵌入是 AI 模型（如 OpenAI text-embedding-ada-002、Claude Embeddings）将文本、图片等数据转换为高维向量的结果。通过计算向量之间的相似度，可以实现语义搜索、推荐系统、RAG 等 AI 功能。

pgvector 的优势在于：它将向量数据存储在你已有的 PostgreSQL 数据库中，你不需要维护单独的向量数据库；向量数据和结构化数据可以在同一个事务中操作，保证数据一致性；你可以使用 PostgreSQL 的全部查询能力来过滤和聚合结果。

### pgvector 实战

```sql
-- 启用 pgvector 扩展
CREATE EXTENSION IF NOT EXISTS vector;

-- 创建文档表
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  embedding VECTOR(1536),  -- OpenAI embedding 维度
  metadata JSONB DEFAULT '{}',
  user_id UUID REFERENCES auth.users(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 创建向量索引（HNSW 算法，更适合高维向量）
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- 创建全文搜索索引（用于混合搜索）
CREATE INDEX ON documents USING gin(to_tsvector('english', title || ' ' || content));
```

```typescript
// 向量搜索实战
import { createClient } from '@supabase/supabase-js'
import OpenAI from 'openai'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY })

// 文档分块与向量化
async function indexDocument(title: string, content: string, userId: string) {
  // 将文档分成小块（chunk）
  const chunks = splitIntoChunks(content, 1000)  // 每块 1000 字符
  
  for (const chunk of chunks) {
    // 生成 embedding
    const embeddingResponse = await openai.embeddings.create({
      model: 'text-embedding-3-small',
      input: chunk
    })
    const embedding = embeddingResponse.data[0].embedding
    
    // 存储到数据库
    await supabase.from('documents').insert({
      title,
      content: chunk,
      embedding,
      user_id: userId,
      metadata: { chunk_index: chunks.indexOf(chunk) }
    })
  }
}

// 向量语义搜索
async function semanticSearch(query: string, userId: string, topK: number = 5) {
  // 生成查询的 embedding
  const queryEmbedding = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: query
  })
  
  // 执行向量搜索
  const { data, error } = await supabase.rpc('match_documents', {
    query_embedding: queryEmbedding.data[0].embedding,
    match_threshold: 0.7,  // 相似度阈值
    match_count: topK,
    user_id: userId
  })
  
  if (error) {
    console.error('搜索失败:', error)
    return []
  }
  
  return data
}

// 创建 RPC 函数用于向量搜索
async function createMatchDocumentsFunction() {
  await supabase.rpc('sql', {
    query: `
      CREATE OR REPLACE FUNCTION match_documents(
        query_embedding VECTOR(1536),
        match_threshold FLOAT,
        match_count INT,
        user_id UUID
      )
      RETURNS TABLE (
        id UUID,
        title TEXT,
        content TEXT,
        metadata JSONB,
        similarity FLOAT
      )
      AS $$
      BEGIN
        RETURN QUERY
        SELECT
          d.id,
          d.title,
          d.content,
          d.metadata,
          1 - (d.embedding <=> query_embedding) AS similarity
        FROM documents d
        WHERE
          d.user_id = match_documents.user_id
          AND 1 - (d.embedding <=> query_embedding) > match_threshold
        ORDER BY d.embedding <=> query_embedding
        LIMIT match_count;
      END;
      $$ LANGUAGE plpgsql;
    `
  })
}

// 混合搜索（向量 + 关键词）
async function hybridSearch(query: string, userId: string, topK: number = 5) {
  const { data: vectorResults } = await semanticSearch(query, userId, topK * 2)
  
  // 关键词搜索
  const { data: keywordResults } = await supabase
    .from('documents')
    .select('id, title, content, metadata')
    .textSearch('title || content', query.replace(/ /g, ' | '))
    .eq('user_id', userId)
    .limit(topK * 2)
  
  // 合并结果
  const results = mergeResults(vectorResults, keywordResults, topK)
  return results
}
```

### RAG 实战

RAG（检索增强生成）是将向量搜索与大语言模型结合的标准范式。以下是一个完整的 RAG 实现：

```typescript
// RAG 查询函数
async function ragQuery(question: string, userId: string) {
  // 1. 生成问题的向量
  const queryEmbedding = await openai.embeddings.create({
    model: 'text-embedding-3-small',
    input: question
  })
  
  // 2. 从 Supabase 获取相关文档
  const { data: docs } = await supabase.rpc('match_documents', {
    query_embedding: queryEmbedding.data[0].embedding,
    match_threshold: 0.7,
    match_count: 5,
    user_id: userId
  })
  
  if (!docs || docs.length === 0) {
    return {
      answer: '抱歉，没有找到与您问题相关的文档。',
      sources: []
    }
  }
  
  // 3. 构建上下文
  const context = docs
    .map(doc => `【文档标题：${doc.title}】\n${doc.content}`)
    .join('\n\n')
  
  // 4. 调用 LLM 生成答案
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages: [
      {
        role: 'system',
        content: `你是一个知识库助手。请根据提供的上下文回答用户的问题。
        
        要求：
        1. 如果上下文中没有相关信息，请如实说明。
        2. 引用相关文档时，请标注来源。
        3. 答案要准确、简洁、有帮助。
        
        上下文：
        ${context}`
      },
      {
        role: 'user',
        content: question
      }
    ],
    temperature: 0.7,
    max_tokens: 1000
  })
  
  return {
    answer: response.choices[0].message.content,
    sources: docs.map(d => ({ title: d.title, content: d.content.slice(0, 100) }))
  }
}
```

---

## Prisma 与 Drizzle 集成

### Prisma + Supabase

Prisma 是一个现代的 TypeScript ORM，提供了类型安全的数据库访问。Prisma 原生支持 Supabase（通过 PostgreSQL 连接器）：

```typescript
// schema.prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["postgresqlUuid"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  posts     Post[]

  @@map("users")
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("posts")
}
```

```typescript
// 使用 Prisma
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// 创建用户
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice'
  }
})

// 创建关联数据
const post = await prisma.post.create({
  data: {
    title: 'Hello World',
    content: 'This is my first post',
    author: {
      connect: { email: 'alice@example.com' }
    }
  },
  include: { author: true }
})

// 查询
const posts = await prisma.post.findMany({
  where: { published: true },
  include: { author: true },
  orderBy: { createdAt: 'desc' }
})
```

### Drizzle + Supabase

Drizzle 是另一个轻量级的 TypeScript ORM，以接近原生 SQL 的语法著称：

```typescript
// db/schema.ts
import { pgTable, uuid, text, boolean, timestamp } from 'drizzle-orm/pg-core'

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name'),
  createdAt: timestamp('created_at').defaultNow()
})

export const posts = pgTable('posts', {
  id: uuid('id').defaultRandom().primaryKey(),
  title: text('title').notNull(),
  content: text('content'),
  published: boolean('published').default(false),
  authorId: uuid('author_id').references(() => users.id),
  createdAt: timestamp('created_at').defaultNow()
})
```

```typescript
import { drizzle } from 'drizzle-orm/supabase-js'
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

const db = drizzle(supabase)

// 查询
const allUsers = await db.select().from(users)

// 条件查询
const activePosts = await db
  .select()
  .from(posts)
  .where(eq(posts.published, true))
  .orderBy(desc(posts.createdAt))
```

---

## 定价与免费额度

### 2026 年 Supabase 定价概览

Supabase 采用按使用量计费的模式，对于个人开发者和小型项目来说相当友好。以下是主要的免费额度：

| 功能 | 免费额度 |
|------|---------|
| **数据库存储** | 500 MB |
| **月活跃用户 (MAU)** | 50,000 |
| **数据传输** | 2 GB 上传 / 10 GB 下载 |
| **API 请求** | 无限制 |
| **数据库备份** | 7 天保留 |
| **Edge Functions** | 500,000 invocations / 月 |
| **Realtime 并发连接** | 200 |
| **Storage** | 1 GB 存储 / 无限制上传 |

付费计划从 $25/月起，提供更多的存储、更长的备份保留、以及优先支持。

> [!TIP]
> Supabase 的免费计划对于 Side Project 和小型应用来说已经非常充足。大多数个人开发者项目在达到免费额度之前就已经开始产生收入了。

---

## 从 Firebase 迁移指南

如果你正在考虑从 Firebase 迁移到 Supabase，以下是主要迁移步骤的概览：

**数据迁移**是最核心的工作。Firebase 的数据是 NoSQL 文档结构，需要转换为 PostgreSQL 的关系型结构。这通常涉及：分析 Firebase 的数据结构、设计关系模型、编写数据转换脚本、验证数据完整性。建议使用 Firebase Admin SDK 导出数据，然后编写脚本转换为 Supabase 可以导入的 CSV 或 JSON 格式。

**认证迁移**相对简单。Supabase 支持导入 Firebase 的用户数据。你可以使用 Firebase Admin SDK 导出用户列表（包括密码哈希），然后使用 Supabase Auth 的批量导入 API 创建用户：

```typescript
// 导入 Firebase 用户到 Supabase
import * as admin from 'firebase-admin'
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
)

async function migrateUsers() {
  const firebaseUsers = await admin.auth().listUsers()
  
  for (const user of firebaseUsers.users) {
    await supabase.auth.admin.createUser({
      email: user.email,
      email_confirm: true,
      // Firebase 的密码可以直接导入（格式兼容）
      ...(user.providerData[0]?.providerId === 'password' && {
        password: 'temporary-password'  // 需要单独处理密码
      }),
      user_metadata: {
        firebase_uid: user.uid,
        display_name: user.displayName,
        photo_url: user.photoURL
      }
    })
  }
}
```

**客户端代码迁移**需要重写数据访问层。从 Firestore 的 `addDoc`、`getDoc`、`onSnapshot` 等方法，转换为 Supabase 的 `insert`、`select`、`subscribe` 等方法。这个工作量取决于应用的复杂度，但整体来说是一个相对机械的过程。

---

## 选型建议与实战场景

### 何时选择 Supabase

Supabase 是以下场景的理想选择：

**快速原型开发**：Supabase 的开箱即用特性让你可以在几分钟内搭建起完整的后端。创建表、配置 RLS、添加认证——所有这些都可以在 Dashboard 中完成，不需要写一行代码。这对于验证想法、快速迭代的项目来说价值巨大。

**SaaS 多租户应用**：RLS 让你可以在数据库层面实现数据隔离，不需要为每个租户创建单独的数据库或表。这大大简化了多租户应用的架构，同时保持了良好的数据安全性。

**AI RAG 应用**：pgvector 向量搜索让你可以在同一个数据库中存储结构化数据和向量嵌入。结合 Edge Functions，你可以构建完整的 RAG pipeline，从文档分块、向量化、到语义搜索、再到 LLM 增强——全部在 Supabase 生态内完成。

**中小型 Web 应用**：对于日活几万到几十万的 Web 应用来说，Supabase 的性能完全够用。它的 PostgreSQL 核心提供了足够的关系型数据处理能力，而 Realtime 功能可以支持实时协作、聊天等场景。

### 何时不选 Supabase

Supabase 不是万能药，以下场景可能需要考虑其他方案：

**超大规模应用**：日活超过百万、每秒请求数万的应用，可能需要更专业的基础设施。Supabase 的托管服务有单点限制，自托管需要专门的运维团队。

**复杂 OLAP 场景**：虽然 PostgreSQL 本身支持分析查询，但对于 TB 级别的数据分析，应该考虑 ClickHouse、BigQuery、Snowflake 等专门的 OLAP 解决方案。

**强一致性要求极高的金融系统**：Supabase 的 Realtime 和高可用配置涉及复制延迟。对于需要强一致性的金融交易系统，传统的数据库集群可能是更稳妥的选择。

---

> [!SUCCESS]
> Supabase 作为 Firebase 的开源替代方案，凭借 PostgreSQL 的强大能力、完善的 Auth/RLS/Realtime 生态、以及对 AI 场景的原生支持，已经成为现代应用开发的热门选择。掌握 Supabase 的核心概念和使用方法，可以让你的开发效率提升数倍，同时保持对数据的完全控制。

---

## 附录：常用资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://supabase.com/docs |
| GitHub | https://github.com/supabase/supabase |
| Discord 社区 | https://discord.supabase.com |
| 官方博客 | https://supabase.com/blog |
| Edge Functions 示例 | https://github.com/supabase/supabase/tree/master/examples |
