# Supabase 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Supabase 架构、Auth 认证、Database RLS、Storage 存储、Edge Functions、Realtime 实时功能、pgvector 向量搜索，以及在 AI 应用中的实战场景。

---

## 目录

1. [[#Supabase 概述与核心定位]]
2. [[#架构解析]]
3. [[#Auth 认证系统]]
4. [[#Database 与 RLS]]
5. [[#Storage 存储服务]]
6. [[#Edge Functions]]
7. [[#Realtime 实时订阅]]
8. [[#Supabase vs Firebase vs PlanetScale 对比]]
9. [[#pgvector 向量搜索]]
10. [[#实战场景与代码示例]]
11. [[#选型建议]]

---

## Supabase 概述与核心定位

### 为什么选择 Supabase

Supabase 是一个开源的 Firebase 替代方案，于 2020 年正式发布。它基于 PostgreSQL 构建，提供了实时数据库、身份认证、文件存储、边缘函数等开箱即用的后端服务。

Supabase 的核心优势体现在四个维度：

**开源透明**：核心代码完全开源（Apache 2.0），不依赖特定厂商，避免供应商锁定。

**PostgreSQL 核心**：继承 PostgreSQL 的全部能力，包括 ACID 事务、复杂查询、JSON 支持、向量搜索等。

**实时能力**：内置 Realtime 引擎，支持数据库变更的实时推送，无需额外配置。

**开发效率**：即开即用的后端服务，大幅减少重复工作，让开发者专注于前端和产品逻辑。

### Supabase 在 AI 时代的价值

在 AI 应用中，Supabase 展现了独特的优势：

- **向量搜索**：基于 PostgreSQL + pgvector，支持 AI RAG 场景
- **Row Level Security**：细粒度访问控制，保护用户数据
- **实时订阅**：AI 任务进度的实时更新
- **Edge Functions**：低延迟的 API 逻辑执行
- **多租户支持**：RLS 实现数据隔离，适合 SaaS 应用

---

## 架构解析

### 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                     Supabase 架构                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                    API Gateway                          │ │
│  │         (Kong + PostgREST + GoTrue)                     │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                  │
│         ┌─────────────────┼─────────────────┐                │
│         ▼                 ▼                 ▼                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  PostgREST  │  │   GoTrue    │  │  Edge Fn    │        │
│  │  (REST API) │  │  (Auth)     │  │  (Deno)     │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│         │                 │                 │                │
│         └─────────────────┼─────────────────┘                │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │                  PostgreSQL + Extensions                 │ │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │ │
│  │  │ pgvector │ │ PostGIS  │ │  JSONB   │ │ pg_amqp  │ │ │
│  │  └──────────┘ └──────────┘ └──────────┘ └──────────┘ │ │
│  └─────────────────────────────────────────────────────────┘ │
│                           │                                  │
│         ┌─────────────────┼─────────────────┐                │
│         ▼                 ▼                 ▼                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│  │  Realtime   │  │   Storage   │  │   Studio    │        │
│  │   Engine    │  │  (S3-like)  │  │  (Dashboard) │        │
│  └─────────────┘  └─────────────┘  └─────────────┘        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 核心组件

| 组件 | 功能 | 说明 |
|------|------|------|
| **PostgREST** | REST API 生成 | 自动生成 CRUD API |
| **GoTrue** | 身份认证 | JWT 令牌管理 |
| **Postgres** | 数据库 | PostgreSQL 核心 |
| **Realtime** | 实时订阅 | WebSocket 推送 |
| **Storage** | 文件存储 | S3 兼容存储 |
| **Edge Functions** | 边缘函数 | Deno 运行时 |
| **Studio** | 管理界面 | 数据库可视化 |

---

## Auth 认证系统

### 认证提供商列表

| 类型 | 提供商 | 说明 |
|------|--------|------|
| **邮箱/密码** | 内置 | 经典邮箱注册登录 |
| **OAuth** | Google | 最常用的第三方登录 |
| **OAuth** | GitHub | 开发者友好 |
| **OAuth** | Facebook | 社交登录 |
| **OAuth** | Apple | iOS 用户 |
| **OAuth** | Twitter/X | 社交登录 |
| **OAuth** | Azure AD | 企业用户 |
| **OAuth** | SAML | 企业 SSO |
| **Magic Link** | 内置 | 无密码登录 |
| **Phone** | 内置 | 手机号登录 |

### 客户端集成

```typescript
// 安装 Supabase 客户端
// npm install @supabase/supabase-js

import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
)

// 邮箱密码注册
async function signUp(email: string, password: string) {
  const { data, error } = await supabase.auth.signUp({
    email,
    password,
    options: {
      data: {
        full_name: 'User Name',  // 存储额外信息
        avatar_url: ''
      }
    }
  })
  
  if (error) throw error
  return data
}

// 邮箱密码登录
async function signIn(email: string, password: string) {
  const { data, error } = await supabase.auth.signInWithPassword({
    email,
    password
  })
  
  if (error) throw error
  return data
}

// OAuth 登录（Google）
async function signInWithGoogle() {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo: 'https://yourapp.com/auth/callback'
    }
  })
  
  if (error) throw error
  return data
}

// Magic Link 登录
async function signInWithMagicLink(email: string) {
  const { data, error } = await supabase.auth.signInWithOtp({
    email,
    options: {
      emailRedirectTo: 'https://yourapp.com/auth/callback'
    }
  })
  
  if (error) throw error
  return data
}

// 获取当前用户
async function getCurrentUser() {
  const { data: { user } } = await supabase.auth.getUser()
  return user
}

// 监听认证状态变化
supabase.auth.onAuthStateChange((event, session) => {
  if (event === 'SIGNED_IN') {
    console.log('User signed in:', session.user)
  } else if (event === 'SIGNED_OUT') {
    console.log('User signed out')
  } else if (event === 'TOKEN_REFRESHED') {
    console.log('Token refreshed')
  }
})

// 退出登录
async function signOut() {
  const { error } = await supabase.auth.signOut()
  if (error) throw error
}
```

### 服务端验证

```typescript
// Edge Function 中的认证验证
import { createClient } from '@supabase/supabase-js'
import type { RequestEvent } from '@supabase/functions'

export async function handler(event: RequestEvent) {
  // 获取 Authorization header
  const authHeader = event.request.headers.get('Authorization')
  
  if (!authHeader) {
    return new Response(JSON.stringify({ error: 'No authorization' }), {
      status: 401
    })
  }
  
  // 创建带认证的客户端
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!,
    {
      global: {
        headers: {
          Authorization: authHeader
        }
      }
    }
  )
  
  // 获取用户信息
  const { data: { user }, error } = await supabase.auth.getUser()
  
  if (error || !user) {
    return new Response(JSON.stringify({ error: 'Unauthorized' }), {
      status: 401
    })
  }
  
  // 处理请求...
  return new Response(JSON.stringify({ user }))
}
```

---

## Database 与 RLS

### Row Level Security 详解

RLS（行级安全策略）是 Supabase 最重要的安全特性之一，允许在数据库层面精细控制数据访问权限。

### RLS 策略示例

```sql
-- 启用 RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;

-- profiles 表：用户只能访问自己的资料
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can update own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own profile"
  ON profiles FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- posts 表：公开文章或作者本人
CREATE POLICY "Anyone can view published posts"
  ON posts FOR SELECT
  USING (
    status = 'published' 
    OR author_id = auth.uid()
  );

CREATE POLICY "Users can insert own posts"
  ON posts FOR INSERT
  WITH CHECK (author_id = auth.uid());

CREATE POLICY "Users can update own posts"
  ON posts FOR UPDATE
  USING (author_id = auth.uid());

CREATE POLICY "Users can delete own posts"
  ON posts FOR DELETE
  USING (author_id = auth.uid());

-- messages 表：仅发送者和接收者可见
CREATE POLICY "Users can view messages they sent or received"
  ON messages FOR SELECT
  USING (
    sender_id = auth.uid() 
    OR recipient_id = auth.uid()
  );

CREATE POLICY "Users can send messages"
  ON messages FOR INSERT
  WITH CHECK (sender_id = auth.uid());

-- conversations 表：参与者可见
CREATE POLICY "Participants can view conversations"
  ON conversations FOR SELECT
  USING (
    auth.uid() = ANY(participant_ids)
  );

-- AI 对话历史：用户只能看到自己的
CREATE POLICY "Users can view own conversations"
  ON conversations FOR SELECT
  USING (user_id = auth.uid());

CREATE POLICY "Users can insert own conversations"
  ON conversations FOR INSERT
  WITH CHECK (user_id = auth.uid());
```

### 复杂 RLS 场景

```sql
-- 多租户场景：RLS 隔离不同组织的数据
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view projects in their organization"
  ON projects FOR SELECT
  USING (
    EXISTS (
      SELECT 1 FROM memberships
      WHERE memberships.user_id = auth.uid()
        AND memberships.organization_id = projects.organization_id
    )
  );

-- AI Token 使用统计：用户只能查看自己的
ALTER TABLE token_usage ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own token usage"
  ON token_usage FOR SELECT
  USING (user_id = auth.uid());

CREATE POLICY "Service role can insert token usage"
  ON token_usage FOR INSERT
  WITH CHECK (true);  -- 服务端不受 RLS 影响

-- 好友关系：双向可见
ALTER TABLE friendships ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their friendships"
  ON friendships FOR SELECT
  USING (
    user_id = auth.uid() 
    OR friend_id = auth.uid()
  );

CREATE POLICY "Users can create friendships"
  ON friendships FOR INSERT
  WITH CHECK (user_id = auth.uid());

-- AI 应用：向量数据隔离
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own documents"
  ON documents FOR SELECT
  USING (user_id = auth.uid());

CREATE POLICY "Users can manage own documents"
  ON documents FOR ALL
  USING (user_id = auth.uid());
```

### Supabase JavaScript 客户端

```typescript
import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!,
  {
    auth: {
      persistSession: true,
      autoRefreshToken: true
    }
  }
)

// 基础 CRUD（自动应用 RLS）
async function getUserProfile() {
  const { data, error } = await supabase
    .from('profiles')
    .select('*')
    .single()
  
  if (error) throw error
  return data
}

async function updateProfile(updates: Partial<Profile>) {
  const { data, error } = await supabase
    .from('profiles')
    .update(updates)
    .eq('user_id', (await supabase.auth.getUser()).data.user?.id)
    .select()
    .single()
  
  if (error) throw error
  return data
}

// 查询过滤
async function getPublishedPosts(limit = 10) {
  const { data, error } = await supabase
    .from('posts')
    .select('id, title, created_at, author:profiles(name)')
    .eq('status', 'published')
    .order('created_at', { ascending: false })
    .limit(limit)
  
  if (error) throw error
  return data
}

// 关联查询
async function getPostWithComments(postId: string) {
  const { data: post, error: postError } = await supabase
    .from('posts')
    .select(`
      *,
      author:profiles!posts_author_id_fkey(name, avatar_url),
      comments(
        *,
        user:profiles!comments_user_id_fkey(name)
      )
    `)
    .eq('id', postId)
    .single()
  
  if (postError) throw postError
  return post
}
```

---

## Storage 存储服务

### 存储桶管理

```typescript
// 创建存储桶（需要服务角色）
async function createBucket() {
  const { data, error } = await supabase.storage.createBucket('avatars', {
    public: true,
    allowedMimeTypes: ['image/png', 'image/jpeg', 'image/webp'],
    fileSizeLimit: 1024 * 1024 * 2  // 2MB
  })
  
  if (error) throw error
  return data
}

// 上传文件
async function uploadAvatar(userId: string, file: File) {
  const fileExt = file.name.split('.').pop()
  const fileName = `${userId}.${fileExt}`
  
  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(fileName, file, {
      upsert: true,
      cacheControl: '3600'
    })
  
  if (error) throw error
  
  // 获取公开 URL
  const { data: { publicUrl } } = supabase.storage
    .from('avatars')
    .getPublicUrl(fileName)
  
  return publicUrl
}

// 上传 AI 生成的内容
async function uploadAIGeneratedImage(imageData: Blob, conversationId: string) {
  const fileName = `${conversationId}/${Date.now()}.png`
  
  const { data, error } = await supabase.storage
    .from('ai-images')
    .upload(fileName, imageData)
  
  if (error) throw error
  
  return supabase.storage
    .from('ai-images')
    .getPublicUrl(fileName).publicUrl
}

// 下载文件
async function downloadFile(bucket: string, path: string) {
  const { data, error } = await supabase.storage
    .from(bucket)
    .download(path)
  
  if (error) throw error
  return data
}

// 列出文件
async function listFiles(bucket: string, folder?: string) {
  const { data, error } = await supabase.storage
    .from(bucket)
    .list(folder || '', {
      limit: 100,
      sortBy: { column: 'created_at', order: 'desc' }
    })
  
  if (error) throw error
  return data
}

// 删除文件
async function deleteFile(bucket: string, path: string) {
  const { error } = await supabase.storage
    .from(bucket)
    .remove([path])
  
  if (error) throw error
}
```

### Storage RLS 策略

```sql
-- 头像：公开读取，仅本人上传
CREATE POLICY "Public avatar access"
  ON storage.objects FOR SELECT
  USING (bucket_id = 'avatars');

CREATE POLICY "Users can upload own avatar"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'avatars' 
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

CREATE POLICY "Users can update own avatar"
  ON storage.objects FOR UPDATE
  USING (
    bucket_id = 'avatars' 
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

-- AI 生成内容：用户只能访问自己的
CREATE POLICY "Users can view own AI images"
  ON storage.objects FOR SELECT
  USING (
    bucket_id = 'ai-images' 
    AND auth.uid()::text = (storage.foldername(name))[1]
  );

CREATE POLICY "Users can upload AI images"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'ai-images' 
    AND auth.uid()::text = (storage.foldername(name))[1]
  );
```

---

## Edge Functions

### 边缘函数开发

```typescript
// supabase/functions/generate-summary/index.ts
import { serve } from 'https://deno.land/std@0.177.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

serve(async (req) => {
  try {
    // 解析请求
    const { articleId } = await req.json()
    
    // 创建 Supabase 客户端
    const supabaseClient = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    )
    
    // 获取文章内容
    const { data: article, error } = await supabaseClient
      .from('articles')
      .select('content')
      .eq('id', articleId)
      .single()
    
    if (error) throw error
    
    // 调用 AI 服务生成摘要（示例）
    const summary = await generateAISummary(article.content)
    
    // 更新数据库
    await supabaseClient
      .from('articles')
      .update({ summary, summary_generated_at: new Date().toISOString() })
      .eq('id', articleId)
    
    return new Response(JSON.stringify({ success: true, summary }))
    
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500
    })
  }
})

async function generateAISummary(content: string): Promise<string> {
  // 调用外部 AI API
  const response = await fetch('https://api.openai.com/v1/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      model: 'gpt-4o',
      messages: [
        { role: 'system', content: '你是一个文章摘要专家。' },
        { role: 'user', content: `请为以下文章生成50字摘要：\n\n${content}` }
      ],
      max_tokens: 100
    })
  })
  
  const data = await response.json()
  return data.choices[0].message.content
}
```

### 本地开发

```bash
# 安装 Supabase CLI
npm install -g supabase

# 登录
supabase login

# 初始化本地开发
supabase init

# 启动本地服务
supabase start

# 部署边缘函数
supabase functions deploy generate-summary

# 调用边缘函数
curl -X POST http://localhost:54321/functions/v1/generate-summary \
  -H "Authorization: Bearer <anon_key>" \
  -H "Content-Type: application/json" \
  -d '{"articleId": "uuid"}'
```

---

## Realtime 实时订阅

### 数据库变更订阅

```typescript
// 订阅对话消息更新
function subscribeToConversation(conversationId: string) {
  const channel = supabase
    .channel(`conversation:${conversationId}`)
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'messages',
        filter: `conversation_id=eq.${conversationId}`
      },
      (payload) => {
        console.log('Message changed:', payload)
        
        if (payload.eventType === 'INSERT') {
          // 新消息
          appendMessage(payload.new)
        } else if (payload.eventType === 'UPDATE') {
          // 消息更新
          updateMessage(payload.new)
        } else if (payload.eventType === 'DELETE') {
          // 消息删除
          removeMessage(payload.old.id)
        }
      }
    )
    .subscribe()
  
  return () => {
    supabase.removeChannel(channel)
  }
}

// 订阅用户在线状态
function subscribeToUserPresence() {
  const channel = supabase.channel('online-users')
  
  channel
    .on('presence', { event: 'sync' }, () => {
      const state = channel.presenceState()
      updateOnlineUsers(Object.keys(state))
    })
    .on('presence', { event: 'join' }, ({ key, newPresences }) => {
      console.log('User joined:', key, newPresences)
    })
    .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
      console.log('User left:', key, leftPresences)
    })
    .subscribe(async (status) => {
      if (status === 'SUBSCRIBED') {
        await channel.track({
          user_id: currentUser.id,
          online_at: new Date().toISOString()
        })
      }
    })
  
  return () => {
    supabase.removeChannel(channel)
  }
}

// 订阅 AI 任务进度
function subscribeToAITask(taskId: string) {
  const channel = supabase
    .channel(`ai-task:${taskId}`)
    .on(
      'postgres_changes',
      {
        event: 'UPDATE',
        schema: 'public',
        table: 'ai_tasks',
        filter: `id=eq.${taskId}`
      },
      (payload) => {
        const { status, progress, result } = payload.new
        
        updateTaskUI({ status, progress })
        
        if (status === 'completed' && result) {
          displayTaskResult(result)
        } else if (status === 'failed') {
          displayTaskError(result.error)
        }
      }
    )
    .subscribe()
  
  return () => {
    supabase.removeChannel(channel)
  }
}
```

---

## Supabase vs Firebase vs PlanetScale 对比

### 核心对比表

| 维度 | Supabase | Firebase | PlanetScale |
|------|----------|----------|-------------|
| **数据库** | PostgreSQL | Firestore (NoSQL) | MySQL |
| **许可证** | Apache 2.0 (开源) | 专有 | 商业/开源分支 |
| **实时能力** | ✅ 内置 | ✅ 原生 | ❌ 需额外配置 |
| **Auth** | ✅ 开源 | ✅ 闭源 | ❌ 无 |
| **Storage** | ✅ S3 兼容 | ✅ GCS | ❌ 无 |
| **Edge Functions** | ✅ Deno | ✅ Cloud Functions | ❌ 无 |
| **向量搜索** | ✅ pgvector | ❌ 插件 | ❌ 无 |
| **离线支持** | ❌ 有限 | ✅ 原生 | ❌ 无 |
| **SQL 支持** | ✅ 完整 | ❌ Firestore API | ✅ 完整 |
| **RLS 安全性** | ✅ 强大 | ⚠️ 规则有限 | ⚠️ 行级策略 |
| **定价** | 合理 | 较贵 | 按用量 |
| **可移植性** | ✅ 高 | ❌ 锁定 | ⚠️ 专有分支 |

### 选择建议对照表

| 场景 | 推荐选择 | 原因 |
|------|----------|------|
| **需要 SQL 能力** | Supabase / PlanetScale | 完整 SQL |
| **需要实时数据库** | Supabase / Firebase | 内置支持 |
| **开源优先** | Supabase ⭐⭐⭐⭐⭐ | 完全开源 |
| **移动应用** | Firebase ⭐⭐⭐⭐⭐ | 离线支持 |
| **AI RAG 应用** | Supabase ⭐⭐⭐⭐⭐ | pgvector |
| **无服务器函数** | Supabase / Firebase | 内置支持 |
| **企业级支持** | PlanetScale / Firebase | 商业支持 |
| **成本敏感** | Supabase ⭐⭐⭐⭐ | 开源可自建 |

---

## pgvector 向量搜索

### 配置向量搜索

```sql
-- 启用向量扩展
CREATE EXTENSION IF NOT EXISTS vector;

-- 创建向量表
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id),
    content TEXT NOT NULL,
    embedding VECTOR(1536),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 创建 HNSW 索引
CREATE INDEX idx_documents_embedding 
ON documents 
USING hnsw (embedding vector_cosine_ops);

-- 插入带嵌入的文档
INSERT INTO documents (user_id, content, embedding, metadata)
VALUES (
    'user-uuid',
    'PostgreSQL is a powerful open-source relational database',
    '[0.1, 0.2, 0.3, ...]::vector',
    '{"source": "wiki", "category": "database"}'
);
```

### 向量搜索查询

```typescript
// 使用 RPC 调用向量搜索
const { data, error } = await supabase.rpc('match_documents', {
  query_embedding: embedding,
  match_threshold: 0.8,
  match_count: 5
})

// 创建匹配函数
-- 在 SQL Editor 中执行
CREATE OR REPLACE FUNCTION match_documents(
  query_embedding VECTOR(1536),
  match_threshold FLOAT DEFAULT 0.7,
  match_count INT DEFAULT 5
)
RETURNS TABLE (
  id UUID,
  content TEXT,
  metadata JSONB,
  similarity FLOAT
) AS $$
BEGIN
  RETURN QUERY
  SELECT 
    d.id,
    d.content,
    d.metadata,
    1 - (d.embedding <=> query_embedding) AS similarity
  FROM documents d
  WHERE 1 - (d.embedding <=> query_embedding) > match_threshold
  ORDER BY d.embedding <=> query_embedding
  LIMIT match_count;
END;
$$ LANGUAGE plpgsql
STRICT
STABLE;
```

### RAG 集成示例

```typescript
// RAG 系统实现
class RAGService {
  constructor(private supabase: SupabaseClient) {}
  
  async addDocument(userId: string, content: string, metadata = {}) {
    // 生成嵌入向量
    const embedding = await this.generateEmbedding(content)
    
    // 存储文档
    const { data, error } = await this.supabase
      .from('documents')
      .insert({
        user_id: userId,
        content,
        embedding,
        metadata
      })
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async search(query: string, userId: string, topK = 5) {
    // 生成查询向量
    const queryEmbedding = await this.generateEmbedding(query)
    
    // 搜索相似文档
    const { data, error } = await this.supabase
      .rpc('match_documents', {
        query_embedding: queryEmbedding,
        match_threshold: 0.7,
        match_count: topK
      })
    
    if (error) throw error
    
    // 过滤用户自己的文档
    const filteredDocs = data.filter(doc => doc.user_id === userId)
    
    return filteredDocs
  }
  
  async generateContext(query: string, userId: string): Promise<string> {
    const docs = await this.search(query, userId, 3)
    
    if (docs.length === 0) {
      return ''
    }
    
    return docs
      .map((doc, i) => `[文档 ${i + 1}]\n${doc.content}`)
      .join('\n\n')
  }
  
  private async generateEmbedding(text: string): Promise<number[]> {
    // 调用 OpenAI 或其他嵌入 API
    const response = await fetch('https://api.openai.com/v1/embeddings', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: 'text-embedding-3-small',
        input: text
      })
    })
    
    const data = await response.json()
    return data.data[0].embedding
  }
}
```

---

## 实战场景与代码示例

### 场景一：构建 AI 对话应用

```typescript
// 对话表结构
// conversations: id, user_id, model, title, created_at, updated_at
// messages: id, conversation_id, role, content, tokens, metadata, created_at

import { createClient } from '@supabase/supabase-js'

const supabase = createClient(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
)

class AIChatService {
  async createConversation(userId: string, model: string, title?: string) {
    const { data, error } = await supabase
      .from('conversations')
      .insert({
        user_id: userId,
        model,
        title: title || '新对话'
      })
      .select()
      .single()
    
    if (error) throw error
    return data
  }
  
  async sendMessage(conversationId: string, role: string, content: string) {
    // 创建消息
    const { data: message, error } = await supabase
      .from('messages')
      .insert({
        conversation_id: conversationId,
        role,
        content
      })
      .select()
      .single()
    
    if (error) throw error
    
    // 更新对话时间
    await supabase
      .from('conversations')
      .update({ updated_at: new Date().toISOString() })
      .eq('id', conversationId)
    
    return message
  }
  
  async getConversationHistory(conversationId: string) {
    const { data, error } = await supabase
      .from('messages')
      .select('role, content, created_at')
      .eq('conversation_id', conversationId)
      .order('created_at', { ascending: true })
    
    if (error) throw error
    return data
  }
  
  async listConversations(userId: string, limit = 20) {
    const { data, error } = await supabase
      .from('conversations')
      .select(`
        id,
        title,
        model,
        updated_at,
        messages (
          content,
          role,
          created_at
        )
      `)
      .eq('user_id', userId)
      .order('updated_at', { ascending: false })
      .limit(limit)
    
    if (error) throw error
    return data
  }
}

// 使用示例
const chatService = new AIChatService()

// 创建对话
const conversation = await chatService.createConversation(
  user.id,
  'gpt-4o',
  '关于 Python 的讨论'
)

// 发送用户消息
await chatService.sendMessage(
  conversation.id,
  'user',
  'Python 3.13 有哪些新特性？'
)

// 订阅实时更新
const channel = supabase
  .channel(`conversation:${conversation.id}`)
  .on(
    'postgres_changes',
    {
      event: 'INSERT',
      schema: 'public',
      table: 'messages',
      filter: `conversation_id=eq.${conversation.id}`
    },
    (payload) => {
      const newMessage = payload.new
      if (newMessage.role === 'assistant') {
        displayAIMessage(newMessage.content)
      }
    }
  )
  .subscribe()
```

### 场景二：AI Token 追踪

```typescript
// Token 使用统计
async function trackTokenUsage(
  userId: string,
  model: string,
  promptTokens: number,
  completionTokens: number
) {
  const totalTokens = promptTokens + completionTokens
  const cost = calculateCost(model, totalTokens)
  
  // 记录使用
  const { error } = await supabase
    .from('token_usage')
    .insert({
      user_id: userId,
      model,
      prompt_tokens: promptTokens,
      completion_tokens: completionTokens,
      total_tokens: totalTokens,
      cost_usd: cost
    })
  
  if (error) throw error
  
  // 更新用户配额
  const { data: quota } = await supabase
    .from('user_quotas')
    .select('*')
    .eq('user_id', userId)
    .single()
  
  if (quota) {
    await supabase
      .from('user_quotas')
      .update({
        used_tokens: quota.used_tokens + totalTokens,
        used_amount: quota.used_amount + cost
      })
      .eq('user_id', userId)
  }
}

// 获取使用统计
async function getUsageStats(userId: string, period: 'daily' | 'weekly' | 'monthly') {
  let startDate = new Date()
  
  if (period === 'daily') {
    startDate.setDate(startDate.getDate() - 1)
  } else if (period === 'weekly') {
    startDate.setDate(startDate.getDate() - 7)
  } else {
    startDate.setMonth(startDate.getMonth() - 1)
  }
  
  const { data, error } = await supabase
    .from('token_usage')
    .select('*')
    .eq('user_id', userId)
    .gte('created_at', startDate.toISOString())
  
  if (error) throw error
  
  // 按模型分组统计
  const byModel = data.reduce((acc, usage) => {
    if (!acc[usage.model]) {
      acc[usage.model] = { tokens: 0, cost: 0 }
    }
    acc[usage.model].tokens += usage.total_tokens
    acc[usage.model].cost += usage.cost_usd
    return acc
  }, {})
  
  const total = data.reduce(
    (acc, usage) => ({
      tokens: acc.tokens + usage.total_tokens,
      cost: acc.cost + usage.cost_usd
    }),
    { tokens: 0, cost: 0 }
  )
  
  return { total, byModel, records: data }
}
```

---

## 选型建议

### 何时选择 Supabase

| 场景 | 推荐程度 | 原因 |
|------|----------|------|
| **AI RAG 应用** | ⭐⭐⭐⭐⭐ | pgvector + RLS |
| **需要开源方案** | ⭐⭐⭐⭐⭐ | 完全开源 |
| **实时协作应用** | ⭐⭐⭐⭐⭐ | Realtime 内置 |
| **多租户 SaaS** | ⭐⭐⭐⭐⭐ | RLS 数据隔离 |
| **快速原型开发** | ⭐⭐⭐⭐ | 开箱即用 |
| **移动应用** | ⭐⭐⭐⭐ | Web SDK 完善 |
| **需要离线支持** | ⭐⭐⭐ | 有限支持 |
| **企业级安全** | ⭐⭐⭐⭐ | RLS + 审计 |

### 最佳实践

```typescript
// 1. 使用 TypeScript 获取完整的类型提示
import { Database } from './database.types'

const supabase = createClient<Database>(
  process.env.SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
)

// 2. 始终检查错误
const { data, error } = await supabase.from('table').select('*')
if (error) {
  console.error('Supabase error:', error)
  throw error
}

// 3. 善用 RLS，即使数据目前不需要隔离
ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;

// 4. 使用服务角色密钥时格外小心
// 仅在服务端使用，不要暴露到客户端

// 5. 合理设计索引
CREATE INDEX idx_conversations_user_updated 
ON conversations(user_id, updated_at DESC);
```

---

> [!TIP]
> Supabase 的 pgvector + RLS 组合非常适合构建多租户 AI 应用。用户上传的文档、生成的嵌入向量、对话历史都可以通过 RLS 实现数据隔离，确保用户隐私。

---

## 常见问题与解决方案

### 问题1：RLS 策略未生效

**问题描述**：设置 RLS 后，查询返回空结果或无权限错误。

```sql
-- ❌ 常见错误：RLS 启用但策略配置错误
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = user_id);
-- 如果 auth.users 表中没有对应用户，查询会失败
```

**解决方案**：检查 auth.users 关联和策略语法。

```sql
-- ✅ 确保 profiles 表的主键是 auth.users(id) 的外键
ALTER TABLE profiles ADD CONSTRAINT profiles_user_id_fkey
  FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE;

-- ✅ 使用 auth.uid() 前的有效检查
CREATE POLICY "Users can view own profile"
  ON profiles FOR SELECT
  USING (
    auth.uid() IS NOT NULL 
    AND auth.uid() = user_id
  );

-- ✅ 或者使用服务角色测试（绕过 RLS）
-- 如果服务角色能查到数据，说明 RLS 策略有问题
```

### 问题2：实时订阅不工作

**问题描述**：Realtime 订阅没有收到数据库变更通知。

**排查步骤**：

```typescript
// ✅ 步骤1：确保 RLS 允许实时订阅
-- 在 supabase dashboard 启用 realtime 功能
-- 或使用 SQL：
ALTER PUBLICATION supabase_realtime ADD TABLE your_table;

// ✅ 步骤2：检查订阅配置
const channel = supabase
  .channel('test')
  .on(
    'postgres_changes',
    {
      event: '*',        // INSERT, UPDATE, DELETE
      schema: 'public',  // 必须是 'public'
      table: 'messages',
      filter: 'conversation_id=eq.123'  // 可选过滤
    },
    (payload) => {
      console.log('Change received:', payload);
    }
  )
  .subscribe((status) => {
    console.log('Subscription status:', status);
    // SUBSCRIBED - 订阅成功
    // CLOSED - 订阅关闭
    // CHANNEL_ERROR - 频道错误
  });

// ✅ 步骤3：检查网络和连接
// 浏览器开发者工具 -> Network -> 查找 supabase realtime WebSocket 连接
```

### 问题3：Edge Functions 冷启动慢

**问题描述**：首次调用 Edge Function 响应时间过长。

**解决方案**：

```typescript
// ✅ 使用 Deno 的缓存
Deno.cacheSync('https://esm.sh/@supabase/supabase-js@2');

// ✅ 减少依赖
// ❌ 不要导入整个库
import { createClient } from '@supabase/supabase-js'

// ✅ 只导入需要的函数
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2?bundle'

// ✅ 使用环境变量而非硬编码
const supabaseUrl = Deno.env.get('SUPABASE_URL')!;
const supabaseKey = Deno.env.get('SUPABASE_ANON_KEY')!;
```

### 问题4：存储上传失败

**问题描述**：文件上传到 Storage 返回错误。

```typescript
// ❌ 常见错误：路径格式不正确
const { error } = await supabase.storage
  .from('avatars')
  .upload('user123/avatar.png', file);  // 可能失败
```

**解决方案**：

```typescript
// ✅ 正确路径格式
const { data, error } = await supabase.storage
  .from('avatars')
  .upload(`${userId}/avatar.png`, file, {
    cacheControl: '3600',
    upsert: true  // 覆盖已存在的文件
  });

// ✅ 确保存储桶存在且配置正确
// 在 Supabase Dashboard -> Storage -> 确认存储桶已创建

// ✅ 检查 RLS 策略
-- 存储的 RLS 策略与表不同
CREATE POLICY "Avatar upload policy"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'avatars'
    AND auth.role() = 'authenticated'  -- 需要认证
  );

// ✅ 使用 upload 方法的正确参数
const formData = new FormData();
formData.append('file', fileInput.files[0]);

const { data, error } = await supabase.storage
  .from('my-bucket')
  .upload(`public/${Date.now()}-${file.name}`, file);
```

### 问题5：PostgREST API 返回 404

**问题描述**：使用 `supabase.from().select()` 返回 404。

**解决方案**：

```typescript
// ✅ 检查表名和 schema
// 表必须在 public schema 中
// 或使用 schema 参数明确指定
const { data } = await supabase
  .from('public.profiles')  // 明确指定 schema
  .select('*')
  .eq('id', userId);

// ✅ 检查 API 设置
// Supabase Dashboard -> API Settings
// 确认 API URL 正确

// ✅ 使用原始 SQL 测试
const { data, error } = await supabase.rpc('get_user_by_id', { 
  user_id_param: userId 
});
```

### 问题6：大数据量分页超时

**问题描述**：分页查询大数据表时超时。

**解决方案**：

```sql
-- ✅ 使用游标分页代替偏移量分页
-- 创建游标索引
CREATE INDEX idx_posts_cursor ON posts(created_at DESC, id);

-- 使用 keyset 分页
SELECT * FROM posts 
WHERE (created_at, id) < (last_created_at, last_id)
ORDER BY created_at DESC, id DESC
LIMIT 20;

-- ✅ 使用 COUNT 估算代替精确 COUNT
CREATE INDEX idx_posts_count ON posts(author_id);
-- 使用 EXPLAIN 分析查询计划
EXPLAIN ANALYZE SELECT * FROM posts WHERE author_id = 'user-123';
```

```typescript
// ✅ 客户端使用游标分页
async function fetchPostsByCursor(userId: string, cursor?: { createdAt: string; id: string }) {
  let query = supabase
    .from('posts')
    .select('*')
    .eq('author_id', userId)
    .order('created_at', { ascending: false })
    .order('id', { ascending: false })
    .limit(20);

  if (cursor) {
    query = query
      .lt('created_at', cursor.createdAt)
      .lt('id', cursor.id);
  }

  return query;
}
```

### 问题7：认证状态丢失

**问题描述**：页面刷新后用户登录状态丢失。

**解决方案**：

```typescript
// ✅ 确保配置正确
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
  {
    auth: {
      persistSession: true,      // 持久化会话
      autoRefreshToken: true,    // 自动刷新令牌
      detectSessionInUrl: true,   // 检测 URL 中的会话（用于 OAuth 回调）
      storage: {
        getItem: (key) => {
          try {
            return localStorage.getItem(key);
          } catch {
            return null;
          }
        },
        setItem: (key, value) => {
          try {
            localStorage.setItem(key, value);
          } catch {
            // 忽略存储错误（如隐私模式）
          }
        },
        removeItem: (key) => {
          try {
            localStorage.removeItem(key);
          } catch {
            // 忽略错误
          }
        }
      }
    }
  }
);

// ✅ 在应用启动时恢复会话
import { useEffect } from 'react';

function App() {
  const [session, setSession] = useState(null);
  
  useEffect(() => {
    // 检查初始会话
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
    });

    // 监听会话变化
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (event, session) => {
        setSession(session);
      }
    );

    return () => subscription.unsubscribe();
  }, []);

  return <div>{session ? 'Logged in' : 'Logged out'}</div>;
}
```

### 问题8：数据库迁移失败

**问题描述**：迁移文件执行失败或生产环境数据不一致。

**解决方案**：

```bash
# ✅ 使用 Supabase CLI 进行迁移
supabase migration new add_user_profiles_table

# ✅ 编写安全的迁移文件
-- migrations/20240101000000_add_user_profiles_table.sql

-- 向后兼容的迁移：添加 nullable 列
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS avatar_url TEXT;

-- 填充默认值（可选）
UPDATE profiles SET avatar_url = 'https://default-avatar.com/default.png'
WHERE avatar_url IS NULL;

-- 再次迁移使其 NOT NULL（分两步保证兼容性）
ALTER TABLE profiles ALTER COLUMN avatar_url SET NOT NULL;

-- ✅ 生产环境执行前测试
supabase db push --db-url "postgresql://user:pass@localhost:54322/postgres"

# ✅ 回滚策略
-- 始终保留回滚脚本
-- migrations/20240101000000_add_user_profiles_table_down.sql
ALTER TABLE profiles DROP COLUMN IF EXISTS avatar_url;
```

### 问题9：Webhook 配置不工作

**问题描述**：数据库 webhook 触发器没有发送请求。

```sql
-- ✅ 检查触发器是否创建成功
SELECT * FROM pg_trigger WHERE tgname LIKE '%webhook%';

-- ✅ 检查触发器函数
CREATE OR REPLACE FUNCTION notify_webhook()
RETURNS TRIGGER AS $$
BEGIN
  -- 发送通知到 Realtime
  PERFORM pg_notify(
    'webhook_channel',
    json_build_object(
      'table', TG_TABLE_NAME,
      'action', TG_OP,
      'record', NEW,
      'old_record', OLD
    )::text
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ✅ 创建触发器
CREATE TRIGGER users_webhook
  AFTER INSERT OR UPDATE OR DELETE ON users
  FOR EACH ROW EXECUTE FUNCTION notify_webhook();
```

### 问题10：向量搜索结果不准确

**问题描述**：pgvector 相似度搜索返回不相关的结果。

**解决方案**：

```sql
-- ✅ 确保索引类型匹配查询方式
-- 余弦相似度搜索
CREATE INDEX ON documents USING hnsw (embedding vector_cosine_ops);

-- L2 距离搜索
CREATE INDEX ON documents USING hnsw (embedding vector_l2_ops);

-- 内积搜索
CREATE INDEX ON documents USING hnsw (embedding vector_ip_ops);

-- ✅ 调整搜索参数
-- 重新生成向量（使用相同的嵌入模型）
INSERT INTO documents (content, embedding)
VALUES ('text', gen_random_vector(1536));

-- ✅ 检查向量维度
-- 确保所有向量维度一致
SELECT embedding FROM documents LIMIT 5;
-- 应该返回相同长度的数组
```

---

## 实战项目示例

### 项目一：多租户 SaaS 应用

**功能特性**：

- 组织/租户隔离
- 团队成员管理
- 基于角色的权限控制
- 使用量追踪

```typescript
// 数据库表结构
// supabase/migrations/001_initial_schema.sql

-- 组织表
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    plan TEXT NOT NULL DEFAULT 'free',
    settings JSONB DEFAULT '{}',
    max_users INT DEFAULT 5,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 用户组织关联
CREATE TABLE organization_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
    user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
    role TEXT NOT NULL DEFAULT 'member',
    joined_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(organization_id, user_id)
);

-- 资源配额
CREATE TABLE quotas (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
    resource_type TEXT NOT NULL,
    total_limit BIGINT DEFAULT 0,
    used_amount BIGINT DEFAULT 0,
    reset_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    UNIQUE(organization_id, resource_type)
);

-- 内容表
CREATE TABLE documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    created_by UUID REFERENCES auth.users(id),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 启用 RLS
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE organization_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE quotas ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- 组织 RLS 策略
CREATE POLICY "Organization members can view own organizations"
    ON organizations FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM organization_members
            WHERE organization_members.organization_id = organizations.id
            AND organization_members.user_id = auth.uid()
        )
    );

CREATE POLICY "Admins can update organizations"
    ON organizations FOR UPDATE
    USING (
        EXISTS (
            SELECT 1 FROM organization_members
            WHERE organization_members.organization_id = organizations.id
            AND organization_members.user_id = auth.uid()
            AND organization_members.role = 'admin'
        )
    );

-- 成员 RLS 策略
CREATE POLICY "Members can view own organization members"
    ON organization_members FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM organization_members om
            WHERE om.organization_id = organization_members.organization_id
            AND om.user_id = auth.uid()
        )
    );

CREATE POLICY "Admins can manage members"
    ON organization_members FOR ALL
    USING (
        EXISTS (
            SELECT 1 FROM organization_members om
            WHERE om.organization_id = organization_members.organization_id
            AND om.user_id = auth.uid()
            AND om.role = 'admin'
        )
    );

CREATE POLICY "Users can join organizations"
    ON organization_members FOR INSERT
    WITH CHECK (auth.uid() = user_id);

-- 文档 RLS 策略
CREATE POLICY "Organization members can manage documents"
    ON documents FOR ALL
    USING (
        EXISTS (
            SELECT 1 FROM organization_members
            WHERE organization_members.organization_id = documents.organization_id
            AND organization_members.user_id = auth.uid()
        )
    );
```

```typescript
// supabase/functions/organizations/index.ts
import { serve } from 'https://deno.land/std@0.177.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};

serve(async (req) => {
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders });
  }

  try {
    const supabaseClient = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    );

    const { data: { user }, error: authError } = await supabaseClient.auth.getUser(
      req.headers.get('Authorization')?.replace('Bearer ', '')
    );

    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    const url = new URL(req.url);
    const path = url.pathname.split('/').pop();

    if (req.method === 'GET' && path === 'organizations') {
      // 获取用户所属的组织列表
      const { data, error } = await supabaseClient
        .from('organization_members')
        .select(`
          role,
          joined_at,
          organization:organizations (
            id,
            name,
            slug,
            plan,
            created_at
          )
        `)
        .eq('user_id', user.id);

      if (error) throw error;

      return new Response(JSON.stringify({ data }), {
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    if (req.method === 'POST' && path === 'organizations') {
      const { name, slug } = await req.json();

      // 创建组织
      const { data: org, error: orgError } = await supabaseClient
        .from('organizations')
        .insert({ name, slug })
        .select()
        .single();

      if (orgError) throw orgError;

      // 添加创建者为管理员
      const { error: memberError } = await supabaseClient
        .from('organization_members')
        .insert({
          organization_id: org.id,
          user_id: user.id,
          role: 'admin'
        });

      if (memberError) throw memberError;

      // 初始化默认配额
      const { error: quotaError } = await supabaseClient
        .from('quotas')
        .insert([
          { organization_id: org.id, resource_type: 'documents', total_limit: 100 },
          { organization_id: org.id, resource_type: 'storage_mb', total_limit: 500 }
        ]);

      if (quotaError) throw quotaError;

      return new Response(JSON.stringify({ data: org }), {
        status: 201,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    if (req.method === 'POST' && path === 'invite') {
      const { organizationId, email, role = 'member' } = await req.json();

      // 验证邀请者权限
      const { data: membership } = await supabaseClient
        .from('organization_members')
        .select('role')
        .eq('organization_id', organizationId)
        .eq('user_id', user.id)
        .single();

      if (!membership || membership.role !== 'admin') {
        return new Response(JSON.stringify({ error: 'Forbidden' }), {
          status: 403,
          headers: { ...corsHeaders, 'Content-Type': 'application/json' }
        });
      }

      // 查找被邀请用户
      const { data: invitee } = await supabaseClient
        .from('profiles')
        .select('user_id')
        .eq('email', email)
        .single();

      if (!invitee) {
        return new Response(JSON.stringify({ error: 'User not found' }), {
          status: 404,
          headers: { ...corsHeaders, 'Content-Type': 'application/json' }
        });
      }

      // 添加成员
      const { data, error } = await supabaseClient
        .from('organization_members')
        .insert({
          organization_id: organizationId,
          user_id: invitee.user_id,
          role
        })
        .select()
        .single();

      if (error) throw error;

      return new Response(JSON.stringify({ data }), {
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    return new Response(JSON.stringify({ error: 'Not found' }), {
      status: 404,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });
  }
});
```

### 项目二：AI RAG 知识库系统

**功能特性**：

- 文档上传与处理
- 文本分块与嵌入
- 向量相似度搜索
- 多模态内容支持

```sql
-- RAG 相关表结构
-- supabase/migrations/002_rag_schema.sql

-- 文档存储
CREATE TABLE knowledge_documents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
    title TEXT NOT NULL,
    source_type TEXT NOT NULL, -- 'file', 'url', 'text'
    source_url TEXT,
    file_path TEXT,
    status TEXT DEFAULT 'pending', -- 'pending', 'processing', 'ready', 'failed'
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 文档分块
CREATE TABLE document_chunks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    document_id UUID REFERENCES knowledge_documents(id) ON DELETE CASCADE NOT NULL,
    chunk_index INT NOT NULL,
    content TEXT NOT NULL,
    embedding VECTOR(1536),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 向量索引（HNSW）
CREATE INDEX idx_chunks_embedding_hnsw 
    ON document_chunks 
    USING hnsw (embedding vector_cosine_ops)
    WITH (m = 16, ef_construction = 64);

-- 用户会话
CREATE TABLE chat_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth_users(id) ON DELETE CASCADE NOT NULL,
    title TEXT,
    model TEXT DEFAULT 'gpt-4',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 聊天消息
CREATE TABLE chat_messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    session_id UUID REFERENCES chat_sessions(id) ON DELETE CASCADE NOT NULL,
    role TEXT NOT NULL, -- 'user', 'assistant', 'system'
    content TEXT NOT NULL,
    context_docs JSONB DEFAULT '[]', -- 引用的文档
    tokens_used INT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 启用 RLS
ALTER TABLE knowledge_documents ENABLE ROW LEVEL SECURITY;
ALTER TABLE document_chunks ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE chat_messages ENABLE ROW LEVEL SECURITY;

-- 文档 RLS
CREATE POLICY "Users manage own documents"
    ON knowledge_documents FOR ALL
    USING (auth.uid() = user_id);

-- 分块 RLS（通过文档关联）
CREATE POLICY "Users access own document chunks"
    ON document_chunks FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM knowledge_documents
            WHERE knowledge_documents.id = document_chunks.document_id
            AND knowledge_documents.user_id = auth.uid()
        )
    );

-- 会话 RLS
CREATE POLICY "Users manage own sessions"
    ON chat_sessions FOR ALL
    USING (auth.uid() = user_id);

-- 消息 RLS
CREATE POLICY "Users access own messages"
    ON chat_messages FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM chat_sessions
            WHERE chat_sessions.id = chat_messages.session_id
            AND chat_sessions.user_id = auth.uid()
        )
    );

CREATE POLICY "Users create own messages"
    ON chat_messages FOR INSERT
    WITH CHECK (
        EXISTS (
            SELECT 1 FROM chat_sessions
            WHERE chat_sessions.id = chat_messages.session_id
            AND chat_sessions.user_id = auth.uid()
        )
    );

-- 向量搜索 RPC 函数
CREATE OR REPLACE FUNCTION search_knowledge(
    query_embedding VECTOR(1536),
    match_threshold FLOAT DEFAULT 0.7,
    match_count INT DEFAULT 5,
    user_id_param UUID
)
RETURNS TABLE (
    chunk_id UUID,
    content TEXT,
    document_id UUID,
    document_title TEXT,
    similarity FLOAT,
    metadata JSONB
) AS $$
BEGIN
    RETURN QUERY
    SELECT
        dc.id as chunk_id,
        dc.content,
        dc.document_id,
        kd.title as document_title,
        1 - (dc.embedding <=> query_embedding) as similarity,
        dc.metadata
    FROM document_chunks dc
    JOIN knowledge_documents kd ON kd.id = dc.document_id
    WHERE kd.user_id = user_id_param
    AND kd.status = 'ready'
    AND 1 - (dc.embedding <=> query_embedding) > match_threshold
    ORDER BY dc.embedding <=> query_embedding
    LIMIT match_count;
END;
$$ LANGUAGE plpgsql
SECURITY DEFINER;
```

```typescript
// supabase/functions/rag-chat/index.ts
import { serve } from 'https://deno.land/std@0.177.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
};

serve(async (req) => {
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders });
  }

  try {
    const authHeader = req.headers.get('Authorization');
    if (!authHeader) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_ANON_KEY')!,
      { global: { headers: { Authorization: authHeader } } }
    );

    const { data: { user }, error: authError } = await supabase.auth.getUser();
    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      });
    }

    const { sessionId, message, useRag = true, topK = 5 } = await req.json();

    // 获取或创建会话
    let session;
    if (sessionId) {
      const { data } = await supabase
        .from('chat_sessions')
        .select('*')
        .eq('id', sessionId)
        .eq('user_id', user.id)
        .single();
      session = data;
    } else {
      const { data } = await supabase
        .from('chat_sessions')
        .insert({ user_id: user.id })
        .select()
        .single();
      session = data;
    }

    // 保存用户消息
    await supabase.from('chat_messages').insert({
      session_id: session.id,
      role: 'user',
      content: message
    });

    let contextDocs: any[] = [];
    let systemPrompt = '你是一个有用的AI助手。';

    // RAG 检索
    if (useRag) {
      // 生成查询向量
      const embedResponse = await fetch('https://api.openai.com/v1/embeddings', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          model: 'text-embedding-3-small',
          input: message
        })
      });

      const embedData = await embedResponse.json();
      const queryEmbedding = embedData.data[0].embedding;

      // 搜索相关文档
      const { data: relevantChunks } = await supabase
        .rpc('search_knowledge', {
          query_embedding: queryEmbedding,
          match_threshold: 0.7,
          match_count: topK,
          user_id_param: user.id
        });

      contextDocs = relevantChunks || [];

      if (contextDocs.length > 0) {
        const contextText = contextDocs
          .map((doc, i) => `[参考文档 ${i + 1}] ${doc.content}`)
          .join('\n\n');

        systemPrompt = `你是一个有用的AI助手。请根据以下参考文档回答用户的问题。\n\n参考文档：\n${contextText}`;
      }
    }

    // 获取历史消息
    const { data: history } = await supabase
      .from('chat_messages')
      .select('role, content')
      .eq('session_id', session.id)
      .order('created_at', { ascending: true });

    const messages = [
      { role: 'system', content: systemPrompt },
      ...(history || []).map(m => ({ role: m.role, content: m.content }))
    ];

    // 调用 OpenAI
    const openaiResponse = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: 'gpt-4o',
        messages,
        temperature: 0.7
      })
    });

    const completion = await openaiResponse.json();
    const assistantMessage = completion.choices[0]?.message?.content || '';

    // 保存助手回复
    await supabase.from('chat_messages').insert({
      session_id: session.id,
      role: 'assistant',
      content: assistantMessage,
      context_docs: contextDocs.map(d => ({
        documentId: d.document_id,
        chunkId: d.chunk_id,
        title: d.document_title,
        similarity: d.similarity
      })),
      tokens_used: completion.usage?.total_tokens
    });

    // 更新会话时间
    await supabase
      .from('chat_sessions')
      .update({ updated_at: new Date().toISOString() })
      .eq('id', session.id);

    return new Response(JSON.stringify({
      sessionId: session.id,
      message: assistantMessage,
      contextDocs: contextDocs.map(d => ({
        title: d.document_title,
        content: d.content.substring(0, 200) + '...',
        similarity: d.similarity
      }))
    }), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });

  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    });
  }
});
```

---

> [!SUCCESS]
> 本文档全面介绍了 Supabase 的核心特性、架构解析、Auth 认证系统、Database RLS 策略、Storage 存储、Edge Functions、Realtime 实时功能，以及在 AI 应用中的实战场景。Supabase 凭借其开源透明、PostgreSQL 核心、完善的 RLS 安全机制，是构建 AI 应用的优秀选择。

---

## 完整安装与环境配置

### 安装方法

```bash
# Node.js
npm install @supabase/supabase-js @supabase/ssr

# Python
pip install supabase

# Deno
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'
```

### 项目创建

```bash
# 安装 Supabase CLI
npm install -g supabase

# 登录
supabase login

# 初始化项目
supabase init

# 启动本地开发环境
supabase start

# 查看状态
supabase status

# 停止
supabase stop
```

### 配置文件

```toml
# supabase/config.toml

[project]
project_id = "your-project-id"

[api]
enabled = true
port = 54321
schemas = ["public", "storage", "graphql_public"]
extra_search_path = ["public", "extensions"]
max_rows = 1000

[db]
port = 54322
major_version = 15

[studio]
enabled = true
port = 54323

[inbucket]
enabled = true
port = 54324

[storage]
enabled = true
file_size_limit = "50MiB"

[auth]
enabled = true
site_url = "http://localhost:3000"
additional_redirect_urls = ["https://localhost:3000"]
jwt_expiry = 3600
enable_signup = true
enable_anonymous_sign_ins = false

[auth.email]
enable_signup = true
double_confirm_changes = true
enable_confirmations = true
```

### 连接配置

```typescript
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js'
import type { Database } from './database.types'

const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient<Database>(
  supabaseUrl,
  supabaseAnonKey,
  {
    auth: {
      autoRefreshToken: true,
      persistSession: true,
      detectSessionInUrl: true
    }
  }
)

// 服务端专用客户端
export const createServerClient = () => {
  return createClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!,
    {
      auth: {
        autoRefreshToken: false,
        persistSession: false
      }
    }
  )
}
```

---

## 数据库设计模式

### 表结构设计

```sql
-- 用户配置表
CREATE TABLE profiles (
    id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
    username TEXT UNIQUE,
    full_name TEXT,
    avatar_url TEXT,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 博客文章
CREATE TABLE posts (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    title TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    content TEXT,
    excerpt TEXT,
    featured_image TEXT,
    published BOOLEAN DEFAULT false,
    published_at TIMESTAMPTZ,
    author_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 文章标签
CREATE TABLE tags (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    name TEXT UNIQUE NOT NULL,
    slug TEXT UNIQUE NOT NULL
);

-- 文章-标签关联
CREATE TABLE post_tags (
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
    tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- 评论
CREATE TABLE comments (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    content TEXT NOT NULL,
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE NOT NULL,
    author_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
    parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- AI 对话历史
CREATE TABLE conversations (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
    title TEXT,
    model TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE messages (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    conversation_id UUID REFERENCES conversations(id) ON DELETE CASCADE NOT NULL,
    role TEXT NOT NULL CHECK (role IN ('user', 'assistant', 'system')),
    content TEXT NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 文档嵌入向量
CREATE TABLE documents (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    user_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    embedding VECTOR(1536),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 索引
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_published ON posts(published, published_at DESC);
CREATE INDEX idx_posts_slug ON posts(slug);
CREATE INDEX idx_messages_conversation ON messages(conversation_id, created_at);
CREATE INDEX idx_documents_user ON documents(user_id);
CREATE INDEX idx_documents_embedding ON documents USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

---

## Row Level Security 策略

### RLS 策略定义

```sql
-- 启用 RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE comments ENABLE ROW LEVEL SECURITY;
ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;
ALTER TABLE messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- profiles: 用户只能查看和修改自己的资料
CREATE POLICY "Users can view own profile"
    ON profiles FOR SELECT
    USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
    ON profiles FOR UPDATE
    USING (auth.uid() = id);

CREATE POLICY "Users can insert own profile"
    ON profiles FOR INSERT
    WITH CHECK (auth.uid() = id);

-- posts: 所有人可以查看已发布的文章，作者可以管理自己的文章
CREATE POLICY "Anyone can view published posts"
    ON posts FOR SELECT
    USING (published = true OR author_id = auth.uid());

CREATE POLICY "Authenticated users can create posts"
    ON posts FOR INSERT
    WITH CHECK (auth.uid() = author_id);

CREATE POLICY "Authors can update own posts"
    ON posts FOR UPDATE
    USING (auth.uid() = author_id);

CREATE POLICY "Authors can delete own posts"
    ON posts FOR DELETE
    USING (auth.uid() = author_id);

-- comments: 所有人可以查看，评论者可以修改删除
CREATE POLICY "Anyone can view comments"
    ON comments FOR SELECT
    USING (true);

CREATE POLICY "Authenticated users can create comments"
    ON comments FOR INSERT
    WITH CHECK (auth.uid() = author_id);

CREATE POLICY "Authors can update own comments"
    ON comments FOR UPDATE
    USING (auth.uid() = author_id);

CREATE POLICY "Authors can delete own comments"
    ON comments FOR DELETE
    USING (auth.uid() = author_id);

-- conversations: 用户只能看到自己的对话
CREATE POLICY "Users can manage own conversations"
    ON conversations FOR ALL
    USING (auth.uid() = user_id);

-- messages: 用户只能访问自己对话的消息
CREATE POLICY "Users can manage own messages"
    ON messages FOR ALL
    USING (
        conversation_id IN (
            SELECT id FROM conversations WHERE user_id = auth.uid()
        )
    );

-- documents: 用户只能访问自己的文档（多租户隔离）
CREATE POLICY "Users can manage own documents"
    ON documents FOR ALL
    USING (auth.uid() = user_id);
```

### 复杂 RLS 场景

```sql
-- 场景：团队成员可以互相查看文章
CREATE TABLE teams (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    name TEXT NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE team_members (
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE,
    user_id UUID REFERENCES profiles(id) ON DELETE CASCADE,
    role TEXT DEFAULT 'member',
    PRIMARY KEY (team_id, user_id)
);

-- 团队文章
CREATE TABLE team_posts (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    team_id UUID REFERENCES teams(id) ON DELETE CASCADE NOT NULL,
    author_id UUID REFERENCES profiles(id) ON DELETE CASCADE NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE team_posts ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Team members can view team posts"
    ON team_posts FOR SELECT
    USING (
        team_id IN (
            SELECT team_id FROM team_members WHERE user_id = auth.uid()
        )
    );

CREATE POLICY "Team members can create team posts"
    ON team_posts FOR INSERT
    WITH CHECK (
        team_id IN (
            SELECT team_id FROM team_members WHERE user_id = auth.uid()
        )
        AND author_id = auth.uid()
    );
```

---

## Edge Functions

### 创建 Edge Function

```typescript
// supabase/functions/ai-chat/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
}

serve(async (req) => {
  // 处理 CORS 预检请求
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders })
  }

  try {
    const { messages, conversationId, model = 'gpt-3.5-turbo' } = await req.json()

    // 验证用户
    const authHeader = req.headers.get('Authorization')
    if (!authHeader) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' }
      })
    }

    const supabase = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_ANON_KEY') ?? '',
      { global: { headers: { Authorization: authHeader } } }
    )

    // 调用 OpenAI API
    const openAIResponse = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model,
        messages
      })
    })

    const data = await openAIResponse.json()

    // 保存消息到数据库
    if (conversationId) {
      await supabase.from('messages').insert([
        { conversation_id: conversationId, role: 'user', content: messages[messages.length - 2]?.content },
        { conversation_id: conversationId, role: 'assistant', content: data.choices[0]?.message?.content }
      ])
    }

    return new Response(JSON.stringify(data), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    })
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    })
  }
})
```

### 部署 Edge Function

```bash
# 部署
supabase functions deploy ai-chat

# 本地测试
supabase functions serve ai-chat

# 调用
curl -X POST http://localhost:54321/functions/v1/ai-chat \
  -H "Authorization: Bearer $SUPABASE_ANON_KEY" \
  -H "Content-Type: application/json" \
  -d '{"messages": [{"role": "user", "content": "Hello!"}]}'
```

---

## Realtime 实时功能

### 订阅数据库变更

```typescript
// src/lib/realtime.ts
import { RealtimeChannel } from '@supabase/supabase-js'

// 订阅 posts 表的变更
export function subscribeToPosts(onUpdate: (payload: any) => void) {
  const channel: RealtimeChannel = supabase
    .channel('posts-channel')
    .on(
      'postgres_changes',
      {
        event: '*',
        schema: 'public',
        table: 'posts',
        filter: 'published=eq.true'
      },
      (payload) => {
        onUpdate(payload)
      }
    )
    .subscribe()

  return () => {
    supabase.removeChannel(channel)
  }
}

// 订阅 messages 表（新消息）
export function subscribeToMessages(
  conversationId: string,
  onNewMessage: (message: any) => void
) {
  const channel: RealtimeChannel = supabase
    .channel(`messages-${conversationId}`)
    .on(
      'postgres_changes',
      {
        event: 'INSERT',
        schema: 'public',
        table: 'messages',
        filter: `conversation_id=eq.${conversationId}`
      },
      (payload) => {
        onNewMessage(payload.new)
      }
    )
    .subscribe()

  return () => {
    supabase.removeChannel(channel)
  }
}

// Presence 功能（在线状态）
export function subscribeToPresence(roomId: string) {
  const channel = supabase.channel(`presence-${roomId}`, {
    config: { presence: { key: 'user' } }
  })

  channel
    .on('presence', { event: 'sync' }, () => {
      const state = channel.presenceState()
      console.log('Online users:', Object.keys(state))
    })
    .on('presence', { event: 'join' }, ({ key, newPresences }) => {
      console.log('User joined:', key, newPresences)
    })
    .on('presence', { event: 'leave' }, ({ key, leftPresences }) => {
      console.log('User left:', key, leftPresences)
    })
    .subscribe(async (status) => {
      if (status === 'SUBSCRIBED') {
        await channel.track({
          user_id: 'user-123',
          online_at: new Date().toISOString()
        })
      }
    })

  return channel
}
```

---

## Storage 存储

### Storage 配置

```sql
-- 创建 storage bucket
INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES ('avatars', 'avatars', true, 5242880, ARRAY['image/jpeg', 'image/png', 'image/webp']);

INSERT INTO storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
VALUES ('documents', 'documents', false, 104857600, ARRAY['application/pdf', 'text/plain']);

-- Storage RLS 策略
CREATE POLICY "Avatar uploads"
    ON storage.objects FOR INSERT
    WITH CHECK (
        bucket_id = 'avatars'
        AND auth.uid()::text = (storage.foldername(name))[1]
    );

CREATE POLICY "Avatar viewing"
    ON storage.objects FOR SELECT
    USING (bucket_id = 'avatars');

CREATE POLICY "Document uploads for authenticated users"
    ON storage.objects FOR INSERT
    WITH CHECK (
        bucket_id = 'documents'
        AND auth.role() = 'authenticated'
    );

CREATE POLICY "Document access for owner"
    ON storage.objects FOR SELECT
    USING (
        bucket_id = 'documents'
        AND auth.uid()::text = (storage.foldername(name))[1]
    );
```

### Storage 操作

```typescript
// 上传文件
async function uploadAvatar(file: File) {
  const fileExt = file.name.split('.').pop()
  const fileName = `${Math.random()}.${fileExt}`

  const { data, error } = await supabase.storage
    .from('avatars')
    .upload(`${supabase.auth.user()?.id}/${fileName}`, file, {
      cacheControl: '3600',
      upsert: false
    })

  if (error) throw error

  // 获取公共 URL
  const { data: { publicUrl } } = supabase.storage
    .from('avatars')
    .getPublicUrl(data.path)

  return publicUrl
}

// 下载文件
async function downloadFile(path: string) {
  const { data, error } = await supabase.storage
    .from('documents')
    .download(path)

  if (error) throw error
  return URL.createObjectURL(data)
}

// 删除文件
async function deleteFile(path: string) {
  const { error } = await supabase.storage
    .from('documents')
    .remove([path])

  if (error) throw error
}

// 列出文件
async function listFiles() {
  const { data, error } = await supabase.storage
    .from('documents')
    .list('folder-name', {
      limit: 100,
      sortBy: { column: 'created_at', order: 'desc' }
    })

  if (error) throw error
  return data
}
```

---

## 常见陷阱与最佳实践

### 陷阱 1：未启用 RLS

```sql
-- ❌ 错误：表未启用 RLS
-- 任何持有密钥的人都可以访问所有数据

-- ✅ 正确：始终启用 RLS
ALTER TABLE my_table ENABLE ROW LEVEL SECURITY;
```

### 陷阱 2：混淆 Anon Key 和 Service Role Key

```typescript
// ❌ 错误：在客户端使用 Service Role Key
const supabase = createClient(url, process.env.SUPABASE_SERVICE_ROLE_KEY!)

// ✅ 正确：只在服务端使用 Service Role Key
// 客户端使用 Anon Key
const supabase = createClient(url, process.env.SUPABASE_ANON_KEY!)
```

### 陷阱 3：过度依赖客户端 SDK

```typescript
// ❌ 错误：复杂查询全部在客户端
const { data } = await supabase.from('posts').select(`
  *,
  author:profiles!author_id(*),
  tags:post_tags(tag:tags(*))
`)

// ✅ 正确：使用数据库视图或 RPC
const { data } = await supabase.rpc('get_posts_with_details')
```

### 最佳实践清单

1. **启用 RLS**：每个表都应启用并设置策略。
2. **正确的密钥使用**：Anon Key 用于客户端，Service Role Key 用于服务端。
3. **使用 TypeScript**：获取完整的类型提示。
4. **错误处理**：始终检查返回的 error 对象。
5. **RLS 策略测试**：使用 `supabase.auth.signIn` 切换用户测试。
6. **索引设计**：为常用查询字段添加索引。
7. **实时订阅清理**：组件卸载时取消订阅。
8. **Edge Functions 安全**：验证 Authorization 头。
9. **定期备份**：使用 Supabase 的备份功能或 pg_dump。
10. **监控**：使用 Supabase Dashboard 监控使用情况。

---

## 与其他 BaaS 对比

### Supabase vs Firebase

| 特性 | Supabase | Firebase |
|------|----------|----------|
| **核心** | PostgreSQL | NoSQL |
| **开源** | 完全开源 | 专有 |
| **数据库** | 关系型 SQL | 文档型 NoSQL |
| **实时** | Postgres Changes | Firestore |
| **认证** | 多种 provider | 完整 |
| **存储** | S3 兼容 | Cloud Storage |
| **函数** | Deno Edge | Cloud Functions |
| **定价** | 按量计费 | 按操作计费 |

### Supabase vs PlanetScale

| 特性 | Supabase | PlanetScale |
|------|----------|-------------|
| **核心** | PostgreSQL | MySQL |
| **开源** | 完全开源 | 部分开源 |
| **分支** | 有限 | 数据库分支 |
| **Serverless** | 连接池 | Vitess |
| **RLS** | 原生支持 | 不支持 |
| **实时** | Postgres Changes | 不支持 |

### Supabase vs Railway

| 特性 | Supabase | Railway |
|------|----------|---------|
| **类型** | BaaS | PaaS |
| **数据库** | 受管理 | 自管理 |
| **灵活性** | 有限 | 完全控制 |
| **定价** | 按量 | 按量 |
| **扩展** | 自动 | 手动 |

---

## 附录：Supabase 高级配置与最佳实践

### 多租户架构设计

```sql
-- 创建租户隔离的表结构
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    plan TEXT DEFAULT 'free',
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE tenants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE NOT NULL,
    name TEXT NOT NULL,
    database_schema TEXT DEFAULT 'public',
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 启用 RLS
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE tenants ENABLE ROW LEVEL SECURITY;

-- 组织管理员可以管理租户
CREATE POLICY "Organization admins can manage tenants"
    ON tenants FOR ALL
    USING (
        EXISTS (
            SELECT 1 FROM user_roles
            WHERE user_roles.user_id = auth.uid()
            AND user_roles.organization_id = tenants.organization_id
            AND user_roles.role = 'admin'
        )
    );

-- 用户可以访问自己的组织
CREATE POLICY "Users can view own organizations"
    ON organizations FOR SELECT
    USING (
        EXISTS (
            SELECT 1 FROM user_roles
            WHERE user_roles.user_id = auth.uid()
            AND user_roles.organization_id = organizations.id
        )
    );
```

### 性能优化技巧

```sql
-- 使用 EXPLAIN 分析查询
EXPLAIN ANALYZE 
SELECT u.email, p.display_name, COUNT(t.id) as ticket_count
FROM users u
JOIN profiles p ON u.id = p.id
JOIN tickets t ON t.user_id = u.id
WHERE t.status = 'open'
GROUP BY u.email, p.display_name
ORDER BY ticket_count DESC
LIMIT 20;

-- 创建复合索引优化查询
CREATE INDEX idx_tickets_user_status 
ON tickets(user_id, status) 
WHERE status IN ('open', 'pending');

-- 使用分区表优化大数据量
CREATE TABLE events (
    id UUID DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id),
    event_type TEXT NOT NULL,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ DEFAULT NOW()
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2024_01 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE events_2024_02 PARTITION OF events
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
```

### 数据库触发器示例

```sql
-- 自动更新 updated_at 字段
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_profiles_updated_at
    BEFORE UPDATE ON profiles
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- 软删除实现
CREATE OR REPLACE FUNCTION soft_delete_record()
RETURNS TRIGGER AS $$
BEGIN
    NEW.deleted_at = NOW();
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER soft_delete_posts
    BEFORE UPDATE ON posts
    FOR EACH ROW
    WHEN (NEW.deleted_at IS DISTINCT FROM OLD.deleted_at)
    EXECUTE FUNCTION soft_delete_record();

-- 全文搜索配置
ALTER TABLE posts ADD COLUMN IF NOT EXISTS search_vector tsvector;

CREATE INDEX idx_posts_search ON posts USING GIN(search_vector);

CREATE OR REPLACE FUNCTION update_search_vector()
RETURNS TRIGGER AS $$
BEGIN
    NEW.search_vector = 
        setweight(to_tsvector('english', COALESCE(NEW.title, '')), 'A') ||
        setweight(to_tsvector('english', COALESCE(NEW.content, '')), 'B');
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_posts_search_vector
    BEFORE INSERT OR UPDATE ON posts
    FOR EACH ROW
    EXECUTE FUNCTION update_search_vector();

-- 使用全文搜索
SELECT * FROM posts
WHERE search_vector @@ to_tsquery('english', 'rust & web')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'rust & web')) DESC;
```

### 备份与恢复策略

```bash
# 使用 pg_dump 备份
pg_dump -h db.supabase_project_id.supabase.co \
    -U postgres \
    -d postgres \
    -F c \
    -b \
    -v \
    -f backup.dump

# 使用 Supabase CLI 备份
supabase db dump -f backup.sql

# 恢复数据
pg_restore -h localhost \
    -U postgres \
    -d postgres \
    -v \
    backup.dump

# 设置自动备份（在 Supabase Dashboard 中配置）
# 建议：
# - 每日增量备份
# - 每周完整备份
# - 保留最近 30 天的备份
```

---

## 附录：Supabase 扩展与插件系统

### Supabase 扩展列表

Supabase 基于 PostgreSQL，支持丰富的扩展：

```sql
-- 启用常用扩展
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";        -- UUID 生成
CREATE EXTENSION IF NOT EXISTS "pg_trgm";        -- 模糊搜索
CREATE EXTENSION IF NOT EXISTS "pgcrypto";         -- 加密函数
CREATE EXTENSION IF NOT EXISTS "hstore";          -- 键值存储
CREATE EXTENSION IF NOT EXISTS "citext";           -- 不区分大小写的文本
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";       -- UUID v1, v4, v5

-- 向量搜索扩展
CREATE EXTENSION IF NOT EXISTS "vector";           -- 向量嵌入

-- 全文搜索
CREATE EXTENSION IF NOT EXISTS "pg_fulltext";     -- 全文搜索

-- 时间序列数据
CREATE EXTENSION IF NOT EXISTS "timescaledb";     -- 时序数据库

-- 地理信息
CREATE EXTENSION IF NOT EXISTS "postgis";         -- GIS 支持
```

### 地理信息扩展 PostGIS

```sql
-- 创建带位置的用户表
CREATE TABLE locations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id),
    name TEXT NOT NULL,
    address TEXT,
    location GEOGRAPHY(POINT, 4326),
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 启用 RLS
ALTER TABLE locations ENABLE ROW LEVEL SECURITY;

-- RLS 策略
CREATE POLICY "Users can view nearby locations"
    ON locations FOR SELECT
    USING (
        -- 查询附近 10 公里内的地点
        ST_DWithin(
            location,
            ST_MakePoint(-122.4194, 37.7749)::geography,
            10000  -- 10km
        )
    );

CREATE POLICY "Users can insert own locations"
    ON locations FOR INSERT
    WITH CHECK (auth.uid() = user_id);

-- 创建空间索引
CREATE INDEX idx_locations_geo ON locations USING GIST(location);

-- 查询附近地点
SELECT 
    name, 
    address,
    ST_Distance(location, ST_MakePoint(-122.4194, 37.7749)::geography) as distance_meters
FROM locations
WHERE ST_DWithin(
    location,
    ST_MakePoint(-122.4194, 37.7749)::geography,
    10000  -- 10km
)
ORDER BY distance_meters
LIMIT 10;
```

### pg_repack 表重组织

```sql
-- 安装 pg_repack 扩展（在 Supabase 仪表板中操作）
-- 用于在线重组织表，回收空间

-- 重组织表
SELECT repack.repack_table('public', 'posts');

-- 重组织索引
SELECT repack.repack_index('public', 'idx_posts_created_at');
```

### TimescaleDB 时序数据

```sql
-- 启用 TimescaleDB
CREATE EXTENSION IF NOT EXISTS "timescaledb";

-- 创建时序表
CREATE TABLE sensor_data (
    time TIMESTAMPTZ NOT NULL,
    sensor_id TEXT,
    temperature DOUBLE PRECISION,
    humidity DOUBLE PRECISION
);

-- 转换为超表
SELECT create_hypertable('sensor_data', 'time');

-- 创建连续聚合（自动计算平均值）
CREATE MATERIALIZED VIEW sensor_hourly_stats
WITH (timescaledb.continuous) AS
SELECT 
    time_bucket('1 hour', time) AS bucket,
    sensor_id,
    AVG(temperature) as avg_temp,
    AVG(humidity) as avg_humidity,
    MAX(temperature) as max_temp,
    MIN(temperature) as min_temp
FROM sensor_data
GROUP BY bucket, sensor_id;

-- 查询最近的统计
SELECT * FROM sensor_hourly_stats
WHERE sensor_id = 'sensor_001'
ORDER BY bucket DESC
LIMIT 24;
```

### 全文搜索配置

```sql
-- 启用中文分词（需要 zhparser 扩展）
-- 在 Supabase 仪表板中启用

-- 创建搜索配置
CREATE TEXT SEARCH CONFIGURATION chinese_zh (COPY = simple);

-- 添加词典映射
ALTER TEXT SEARCH CONFIGURATION chinese_zh 
    ALTER MAPPING FOR word WITH simple;

-- 为 posts 表添加搜索向量
ALTER TABLE posts ADD COLUMN IF NOT EXISTS search_vector tsvector;

-- 创建 GIN 索引
CREATE INDEX idx_posts_search ON posts USING GIN(search_vector);

-- 更新搜索向量
UPDATE posts SET 
    search_vector = to_tsvector('chinese_zh', COALESCE(title, '') || ' ' || COALESCE(content, ''))
WHERE search_vector IS NULL;

-- 搜索查询
SELECT 
    id, 
    title, 
    ts_rank(search_vector, query) AS rank
FROM posts, to_tsquery('chinese_zh', 'rust & web') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

### 数据库连接池配置

```typescript
// Supabase 连接池配置
// 在 supabase/config.toml 中配置

[db.pooler]
enabled = true
port = 6543
pool_mode = "transaction"  // 或 "session"
default_pool_size = 20
max_client_conn = 100

// 应用中使用连接池
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  port: 6543,  // 使用连接池端口
  database: 'postgres',
  user: 'postgres',
  password: process.env.DB_PASSWORD,
  max: 20,  // 最大连接数
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

### 分页与游标

```typescript
// 基于游标的分页实现

interface Cursor {
  after?: string;
  before?: string;
  limit?: number;
}

interface PaginatedResult<T> {
  data: T[];
  meta: {
    hasNextPage: boolean;
    hasPrevPage: boolean;
    startCursor: string | null;
    endCursor: string | null;
  };
}

async function paginateQuery<T>(
  supabase: SupabaseClient,
  tableName: string,
  cursor?: Cursor,
  options?: {
    orderBy?: { column: string; ascending?: boolean };
    filter?: Record<string, any>;
  }
): Promise<PaginatedResult<T>> {
  const limit = cursor?.limit || 20;
  const orderColumn = options?.orderBy?.column || 'created_at';
  const orderDirection = options?.orderBy?.ascending ? 'asc' : 'desc';

  let query = supabase
    .from(tableName)
    .select('*', { count: 'exact', head: true })
    .order(orderColumn, { ascending: options?.orderBy?.ascending ?? false })
    .limit(limit + 1); // 多取一条判断是否有更多

  // 应用过滤条件
  if (options?.filter) {
    Object.entries(options.filter).forEach(([key, value]) => {
      query = query.eq(key, value);
    });
  }

  // 应用游标
  if (cursor?.after) {
    query = query.gt(orderColumn, cursor.after);
  } else if (cursor?.before) {
    query = query.lt(orderColumn, cursor.before);
  }

  const { data, count, error } = await query;

  if (error) throw error;

  const hasNextPage = data?.length > limit;
  const hasPrevPage = !!cursor?.after || !!cursor?.before;
  const results = hasNextPage ? data?.slice(0, -1) : data;

  return {
    data: results as T[],
    meta: {
      hasNextPage,
      hasPrevPage,
      startCursor: results?.[0]?.id || null,
      endCursor: results?.[results.length - 1]?.id || null,
    },
  };
}
```

### 数据库监控与诊断

```sql
-- 监控慢查询
SELECT 
    pid,
    now() - pg_stat_activity.query_start AS duration,
    usename,
    query,
    state
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes'
    AND state = 'active'
ORDER BY duration DESC;

-- 监控连接数
SELECT 
    datname,
    numbackends,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    round(100.0 * blks_hit / nullif(blks_hit + blks_read, 0), 2) AS cache_hit_ratio
FROM pg_stat_database
WHERE datname = current_database();

-- 监控表大小
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) - pg_relation_size(schemaname||'.'||tablename)) AS indexes_size
FROM pg_catalog.pg_statio_user_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 20;

-- 监控索引使用
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

### 数据库迁移最佳实践

```bash
# 使用 Supabase CLI 管理迁移

# 1. 创建新迁移
supabase migration new add_user_avatar_column

# 2. 编写迁移文件
# supabase/migrations/20240101000000_add_user_avatar_column.sql
ALTER TABLE profiles ADD COLUMN avatar_url TEXT;

# 3. 验证迁移
supabase db validate

# 4. 应用迁移到本地
supabase db push

# 5. 在生产环境应用
supabase db push --db-url $PRODUCTION_DB_URL

# 迁移文件命名规范
# 格式: YYYYMMDDHHMMSS_description.sql
# 示例: 
#   20240101000000_create_users_table.sql
#   20240101120000_add_user_roles.sql
#   20240102000000_create_posts_table.sql
```

```sql
-- 安全迁移模板
-- 始终使用事务包装，确保原子性

BEGIN;

-- 向后兼容：添加 nullable 列
ALTER TABLE profiles ADD COLUMN IF NOT EXISTS avatar_url TEXT;

-- 可选：设置默认值
UPDATE profiles SET avatar_url = 'https://default-avatar.com/default.png' 
WHERE avatar_url IS NULL;

-- 再次迁移使其 NOT NULL
ALTER TABLE profiles ALTER COLUMN avatar_url SET NOT NULL;

COMMIT;

-- 回滚脚本（保存在单独文件）
BEGIN;
ALTER TABLE profiles DROP COLUMN IF EXISTS avatar_url;
COMMIT;
```

### 性能调优建议

```sql
-- 1. 分析慢查询
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM posts 
WHERE author_id = 'user-123'
ORDER BY created_at DESC
LIMIT 20;

-- 2. 创建部分索引
CREATE INDEX idx_posts_published ON posts(created_at DESC)
WHERE published = true;

-- 3. 创建表达式索引
CREATE INDEX idx_posts_title_lower ON posts(lower(title));

-- 4. 创建复合索引
CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at DESC);

-- 5. 使用 BRIN 索引（适合时序数据）
CREATE INDEX idx_events_time ON events USING BRIN(created_at);

-- 6. 重建臃肿索引
REINDEX INDEX CONCURRENTLY idx_posts_slug;

-- 7. 清理无用索引
SELECT 
    schemaname || '.' || tablename AS table,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(i.indexrelid)) AS index_size
FROM pg_stat_user_indexes ui
JOIN pg_index i ON i.indexrelid = ui.indexrelid
WHERE idx_scan = 0
    AND NOT i.indisprimary
    AND NOT i.indisunique
ORDER BY pg_relation_size(i.indexrelid) DESC;
```

### 常见陷阱与最佳实践

1. **始终启用 RLS**：每个表都应启用并设置策略。
2. **使用服务角色密钥时格外小心**：仅在服务端使用，不要暴露到客户端。
3. **善用 RLS**：即使数据目前不需要隔离，也应该启用。
4. **合理设计索引**：为常用查询字段添加索引。
5. **使用事务保证原子性**：复杂操作使用事务包装。
6. **定期分析表**：保持统计信息最新。
7. **监控慢查询**：使用 EXPLAIN 分析查询计划。
8. **使用连接池**：高并发场景配置连接池。
9. **备份策略**：定期备份，使用 pg_dump。
10. **版本控制迁移**：所有迁移文件应纳入版本控制。

---

## 附录：Supabase 与 AI 生态集成

### 与 LangChain 集成

```typescript
import { SupabaseVectorStore } from '@langchain/community/vectorstores/supabase';
import { OpenAIEmbeddings } from '@langchain/openai';
import { RecursiveCharacterTextSplitter } from '@langchain/textsplitters';

const embeddings = new OpenAIEmbeddings({
  openAIApiKey: process.env.OPENAI_API_KEY,
});

const vectorStore = await SupabaseVectorStore.fromTexts(
  texts,           // 文档文本数组
  metadatas,       // 元数据数组
  embeddings,
  {
    client: supabase,  // Supabase 客户端
    tableName: 'documents',
    queryName: 'match_documents',
  }
);

// 相似度搜索
const results = await vectorStore.similaritySearch(
  query,           // 查询文本
  k,                // 返回数量
  filter            // 可选的过滤条件
);
```

### 与 LlamaIndex 集成

```typescript
import { SupabaseVectorStore } from 'llama-index/storage/vectorstore/supabase';
import { setGlobalServiceContext } from 'llama-index/common';

const vectorStore = new SupabaseVectorStore({
  client: supabase,
  tableName: 'documents',
  useAnonymousKey: true,
});

// 创建索引
const index = await VectorStoreIndex.fromVectorStore(vectorStore);

// 查询
const queryEngine = index.asQueryEngine();
const response = await queryEngine.query({
  query: '用户查询文本',
});
```

### 与 Vercel AI SDK 集成

```typescript
import { createClient } from '@supabase/supabase-js';
import { anthropic } from '@ai-sdk/anthropic';
import { generateText, streamText } from 'ai';

// 初始化 Supabase
const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_ANON_KEY!
);

// AI 对话处理
export async function POST(req: Request) {
  const { messages, userId } = await req.json();

  // 1. 获取用户上下文
  const { data: userContext } = await supabase
    .from('user_contexts')
    .select('*')
    .eq('user_id', userId)
    .single();

  // 2. 构建系统提示
  const systemPrompt = `你是专业助手。用户信息: ${JSON.stringify(userContext)}`;

  // 3. 调用 AI
  const result = await streamText({
    model: anthropic('claude-3-5-sonnet-20241022'),
    messages: [
      { role: 'system', content: systemPrompt },
      ...messages
    ],
    onChunk: async (chunk) => {
      // 4. 实时保存到数据库
      if (chunk.text) {
        await supabase.from('ai_stream_chunks').insert({
          user_id: userId,
          conversation_id: conversationId,
          chunk: chunk.text,
        });
      }
    },
  });

  // 5. 保存完整响应
  const fullResponse = await result.fullStream;
  await saveToDatabase(userId, conversationId, fullResponse);

  return result.toDataStreamResponse();
}
```

### 与 Pinecone 等向量数据库对比

```typescript
// Supabase 向量搜索优势

// 1. 统一的数据库和向量存储
// 不需要维护多个服务

// 2. 内置 RLS 安全策略
// 向量数据也可以享受数据库级别的安全保护

// 3. 成本更低
// 无需额外的向量数据库订阅

// 4. 延迟更低
// 向量查询和结构化查询可以在同一个事务中完成

// 5. 数据一致性
// 向量数据和元数据始终保持同步

// 示例：结合向量搜索和元数据过滤
const { data } = await supabase.rpc('search_knowledge', {
  query_embedding: queryEmbedding,
  match_threshold: 0.7,
  match_count: 5,
  user_id_param: userId  // RLS 自动应用
});

// 在 SQL 中实现复杂过滤
CREATE OR REPLACE FUNCTION search_with_filter(
  query_embedding VECTOR(1536),
  filter_conditions JSONB DEFAULT '{}'
)
RETURNS TABLE (...) AS $$
BEGIN
  RETURN QUERY
  SELECT *
  FROM document_chunks dc
  WHERE 
    -- 向量相似度过滤
    1 - (dc.embedding <=> query_embedding) > (filter_conditions->>'threshold')::float
    -- 元数据过滤
    AND dc.metadata->>'category' = (filter_conditions->>'category')
    AND dc.metadata->>'status' = 'active'
  ORDER BY dc.embedding <=> query_embedding
  LIMIT (filter_conditions->>'limit')::int;
END;
$$ LANGUAGE plpgsql;
```

### RAG 增强策略

```typescript
// 混合搜索：向量 + 全文搜索

// 1. 创建全文搜索索引
ALTER TABLE documents ADD COLUMN IF NOT EXISTS fts_vector tsvector;

UPDATE documents SET
  fts_vector = to_tsvector('english', title || ' ' || COALESCE(content, ''))
WHERE fts_vector IS NULL;

CREATE INDEX idx_documents_fts ON documents USING GIN(fts_vector);

// 2. 混合搜索函数
CREATE OR REPLACE FUNCTION hybrid_search(
  search_query TEXT,
  query_embedding VECTOR(1536),
  match_count INT DEFAULT 5,
  vector_weight FLOAT DEFAULT 0.7,
  text_weight FLOAT DEFAULT 0.3
)
RETURNS TABLE (
  id UUID,
  title TEXT,
  content TEXT,
  metadata JSONB,
  combined_score FLOAT
) AS $$
BEGIN
  RETURN QUERY
  WITH vector_results AS (
    SELECT 
      dc.id,
      dc.title,
      dc.content,
      dc.metadata,
      1 - (dc.embedding <=> query_embedding) AS vector_score
    FROM document_chunks dc
    ORDER BY dc.embedding <=> query_embedding
    LIMIT match_count * 2
  ),
  text_results AS (
    SELECT 
      dc.id,
      dc.title,
      dc.content,
      dc.metadata,
      ts_rank(d.fts_vector, plainto_tsquery('english', search_query)) AS text_score
    FROM document_chunks dc
    JOIN documents d ON d.id = dc.document_id
    WHERE d.fts_vector @@ plainto_tsquery('english', search_query)
    LIMIT match_count * 2
  )
  SELECT 
    vr.id,
    vr.title,
    vr.content,
    vr.metadata,
    COALESCE(vr.vector_score * vector_weight, 0) + 
    COALESCE(tr.text_score * text_weight, 0) AS combined_score
  FROM vector_results vr
  FULL OUTER JOIN text_results tr ON vr.id = tr.id
  ORDER BY combined_score DESC
  LIMIT match_count;
END;
$$ LANGUAGE plpgsql;
```

### 缓存策略

```typescript
// 实现多层缓存策略

interface CacheConfig {
  ttl: number;        // 生存时间（秒）
  staleWhileRevalidate: number;  // 过期后仍可使用的时长
  maxSize: number;     // 最大缓存条目数
}

class IntelligentCache {
  private memoryCache = new Map<string, { data: any; expiresAt: number }>();
  private config: CacheConfig;

  constructor(config: CacheConfig) {
    this.config = config;
  }

  async get<T>(key: string, fetcher: () => Promise<T>): Promise<T> {
    // 1. 检查内存缓存
    const cached = this.memoryCache.get(key);
    if (cached && cached.expiresAt > Date.now()) {
      return cached.data as T;
    }

    // 2. 检查是否过期但可重新验证
    if (cached && cached.expiresAt + this.config.staleWhileRevalidate * 1000 > Date.now()) {
      // 返回过期数据，同时后台重新获取
      this.refresh(key, fetcher);
      return cached.data as T;
    }

    // 3. 重新获取数据
    const data = await fetcher();
    
    // 4. 更新缓存
    this.set(key, data);
    
    return data;
  }

  private async refresh<T>(key: string, fetcher: () => Promise<T>) {
    try {
      const data = await fetcher();
      this.set(key, data);
    } catch (error) {
      console.error('Cache refresh failed:', error);
    }
  }

  private set(key: string, data: any) {
    // 清理超出大小的缓存
    if (this.memoryCache.size >= this.config.maxSize) {
      const firstKey = this.memoryCache.keys().next().value;
      this.memoryCache.delete(firstKey);
    }

    this.memoryCache.set(key, {
      data,
      expiresAt: Date.now() + this.config.ttl * 1000,
    });
  }
}

// 使用示例
const cache = new IntelligentCache({
  ttl: 300,                    // 5 分钟
  staleWhileRevalidate: 60,    // 过期后 1 分钟仍可用
  maxSize: 100,
});

async function getCachedDocuments(query: string) {
  const cacheKey = `docs:${query}`;
  
  return cache.get(cacheKey, async () => {
    // 从数据库或向量搜索获取
    const results = await supabase
      .rpc('search_documents', { query_embedding: await embed(query) });
    return results;
  });
}
```

---

## Supabase 高级特性

### 数据库扩展

```sql
-- 1. 安装常用扩展
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";        -- UUID 生成
CREATE EXTENSION IF NOT EXISTS "pg_trgm";          -- 模糊搜索
CREATE EXTENSION IF NOT EXISTS "pgcrypto";         -- 加密函数
CREATE EXTENSION IF NOT EXISTS "hstore";           -- 键值存储
CREATE EXTENSION IF NOT EXISTS "pg_partman";       -- 分区管理
CREATE EXTENSION IF NOT EXISTS "pg_cron";           -- 定时任务
CREATE EXTENSION IF NOT EXISTS "pg_net";            -- HTTP 请求
CREATE EXTENSION IF NOT EXISTS "pgmq";              -- 消息队列
CREATE EXTENSION IF NOT EXISTS "vector";            -- 向量搜索

-- 2. UUID 生成
SELECT uuid_generate_v4();  -- 随机 UUID
SELECT uuid_generate_v1();  -- 基于时间的 UUID

-- 3. 模糊搜索
CREATE EXTENSION pg_trgm;
CREATE INDEX idx_users_name ON users USING gin (name gin_trgm_ops);

SELECT * FROM users WHERE name % 'Jhon';  -- 相似匹配

-- 4. HStore 键值存储
CREATE EXTENSION hstore;
ALTER TABLE profiles ADD COLUMN metadata hstore;

UPDATE profiles SET metadata = 'age=>25,city=>NYC' WHERE id = '...';
SELECT metadata->'city' FROM profiles;

-- 5. 加密函数
SELECT crypt('password', gen_salt('bf'));  -- 生成哈希
SELECT crypt('password', password) = crypt('password', password);  -- 验证

-- 6. 定时任务（pg_cron）
SELECT cron.schedule('cleanup-old-sessions', '*/5 * * * *', $$
  DELETE FROM sessions WHERE expires_at < NOW();
$$);

-- 7. HTTP 请求（pg_net）
SELECT net.http_get(url := 'https://api.example.com/data');

-- 8. 消息队列（pgmq）
SELECT pgmq.send('my_queue', '{"user_id": 1, "action": "send_email"}');
SELECT pgmq.recv('my_queue', 1);  -- 消费一条消息
SELECT pgmq.send_batch('my_queue', ARRAY[
  '{"action": "task1"}',
  '{"action": "task2"}'
]);
```

### Edge Functions 进阶

```typescript
// Deno Deploy Edge Functions

// 1. 环境变量
Deno.env.get('SUPABASE_URL');
Deno.env.get('SUPABASE_ANON_KEY');

// 2. CORS 头
export async function handler(req: Request): Promise<Response> {
  if (req.method === 'OPTIONS') {
    return new Response(null, {
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
        'Access-Control-Allow-Methods': 'POST, GET, OPTIONS',
      },
    });
  }

  // 处理请求...
  return new Response(JSON.stringify({ success: true }), {
    headers: { 'Content-Type': 'application/json' },
  });
}

// 3. 调用 Supabase
import { createClient } from 'jsr:@supabase/supabase-js@2';

Deno.serve(async (req) => {
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  const { data, error } = await supabase
    .from('profiles')
    .select('*')
    .limit(10);

  return Response.json({ data, error });
});

// 4. 文件上传到 Storage
Deno.serve(async (req) => {
  const formData = await req.formData();
  const file = formData.get('file') as File;
  
  const arrayBuffer = await file.arrayBuffer();
  const bytes = new Uint8Array(arrayBuffer);
  
  const supabase = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  );

  const { data, error } = await supabase.storage
    .from('documents')
    .upload(`uploads/${crypto.randomUUID()}`, bytes, {
      contentType: file.type,
    });

  return Response.json({ data, error });
});

// 5. WebSocket（实时订阅）
export { createClient } from 'jsr:@supabase/supabase-js@2';

const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

const channel = supabase.channel('db-changes')
  .on(
    'postgres_changes',
    { event: '*', schema: 'public', table: 'messages' },
    (payload) => {
      console.log('Change received:', payload);
    }
  )
  .subscribe();

// 6. JWT 验证
import { jwtVerify, createRemoteJWKSet } from 'jsr:@supabase/supabase-js@2';

export async function verifyToken(token: string) {
  const JWKS = createRemoteJWKSet(
    new URL(`${Deno.env.get('SUPABASE_URL')!}/auth/v1/jwks`)
  );

  const { payload } = await jwtVerify(token, JWKS);
  return payload;
}

// 7. 调用外部 API
const response = await fetch('https://api.openai.com/v1/embeddings', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${Deno.env.get('OPENAI_API_KEY')}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    input: 'Your text here',
    model: 'text-embedding-ada-002',
  }),
});
```

### 性能优化

```sql
-- 1. 表维护
-- 分析表（更新统计信息）
ANALYZE table_name;

-- 清理死元组
VACUUM table_name;

-- 完全 vacuum（会锁表）
VACUUM FULL table_name;

-- 2. 索引优化
-- 部分索引
CREATE INDEX idx_active_users ON users(email) WHERE is_active = true;

-- 表达式索引
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- 复合索引
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC) WHERE published = true;

-- 3. 分区表
CREATE TABLE events (
    id BIGSERIAL,
    created_at TIMESTAMPTZ NOT NULL,
    event_type TEXT NOT NULL,
    data JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE events_2024_01 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- 4. 连接池配置
-- 查看当前连接
SELECT count(*) FROM pg_stat_activity WHERE datname = current_database();

-- 查看最大连接数
SHOW max_connections;

-- 5. 查询缓存
-- 启用查询缓存
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';

-- 6. 监控慢查询
-- 启用 pg_stat_statements
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

SELECT query, calls, mean_time, total_time
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

### 安全最佳实践

```typescript
// 1. RLS 策略最佳实践

-- 始终使用 RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- 验证用户身份
CREATE POLICY "Users can view own profile"
    ON profiles FOR SELECT
    USING (auth.uid() = id);

-- 限制数据访问
CREATE POLICY "Users can update own profile"
    ON profiles FOR UPDATE
    USING (auth.uid() = id)
    WITH CHECK (auth.uid() = id);

-- 2. API Key 安全

// 在客户端使用 anon key（受限权限）
const supabase = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);

// 在服务端使用 service role key（完全权限）
// 永远不要在前端暴露 service role key

// 3. Edge Functions 验证
export async function handler(req: Request) {
  const authHeader = req.headers.get('Authorization');
  if (!authHeader) {
    return new Response('Unauthorized', { status: 401 });
  }

  const token = authHeader.replace('Bearer ', '');
  const { data: { user }, error } = await supabase.auth.getUser(token);

  if (error || !user) {
    return new Response('Unauthorized', { status: 401 });
  }

  // 处理请求...
}

// 4. Storage 安全

-- 只允许上传到自己的文件夹
CREATE POLICY "Users can only upload to own folder"
    ON storage.objects FOR INSERT
    WITH CHECK (
        bucket_id = 'user-files'
        AND auth.uid()::text = (storage.foldername(name))[1]
    );

-- 5. 防止 SQL 注入
// 永远不要直接拼接用户输入到 SQL
// 使用参数化查询

// ✅ 正确
const { data } = await supabase
  .from('posts')
  .select('*')
  .eq('author_id', userId);

// ❌ 错误
const { data } = await supabase
  .from('posts')
  .select(`*`)
  .filter(`author_id`, 'eq', userId);  // 安全，但 prefer eq()
```

### 监控与日志

```typescript
// 1. 自定义日志表
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    user_id UUID REFERENCES auth.users(id),
    action TEXT NOT NULL,
    table_name TEXT,
    record_id UUID,
    old_data JSONB,
    new_data JSONB,
    ip_address INET,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 触发器记录变更
CREATE OR REPLACE FUNCTION log_changes()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO audit_log (user_id, action, table_name, record_id, old_data, new_data)
    VALUES (
        current_setting('app.current_user', true)::UUID,
        TG_OP,
        TG_TABLE_NAME,
        COALESCE(NEW.id, OLD.id),
        CASE WHEN TG_OP IN ('UPDATE', 'DELETE') THEN to_jsonb(OLD) END,
        CASE WHEN TG_OP IN ('INSERT', 'UPDATE') THEN to_jsonb(NEW) END
    );
    RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER users_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION log_changes();

-- 2. 性能监控
CREATE EXTENSION IF NOT EXISTS pg_stat_statments;

-- 查看最慢查询
SELECT 
    query,
    calls,
    total_exec_time / 1000 as total_seconds,
    mean_exec_time as mean_ms,
    rows
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- 3. 实时监控视图
CREATE OR REPLACE VIEW active_connections AS
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    backend_start,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle';

-- 查看当前连接数
SELECT 
    datname,
    numbackends,
    xact_commit,
    xact_rollback,
    blks_read,
    blks_hit,
    ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio
FROM pg_stat_database;

-- 4. Edge Functions 日志
// 使用 console.log 输出日志
// 在 Supabase Dashboard 查看日志

console.log('Request received:', req.url);
console.log('User:', user?.id);
console.error('Error:', error);

// 5. Realtime 调试
const channel = supabase
  .channel('debug')
  .on('system', { event: '*' }, (payload) => {
    console.log('System event:', payload);
  })
  .subscribe((status) => {
    console.log('Channel status:', status);
  });
```

### CI/CD 集成

```yaml
# .github/workflows/deploy.yml
name: Deploy Supabase

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Supabase CLI
        run: |
          wget -q https://github.com/supabase/cli/releases/latest/download/supabase_linux_amd64.tar.gz
          tar -xzf supabase_linux_amd64.tar.gz
          mv supabase /usr/local/bin/

      - name: Link project
        run: supabase link --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

      - name: Push migrations
        run: supabase db push
        env:
          SUPABASE_DB_PASSWORD: ${{ secrets.SUPABASE_DB_PASSWORD }}

      - name: Deploy Edge Functions
        run: supabase functions deploy
        env:
          SUPABASE_SERVICE_ROLE_KEY: ${{ secrets.SUPABASE_SERVICE_ROLE_KEY }}
```

```bash
# 本地开发脚本
#!/bin/bash
# setup-local.sh

# 启动本地 Supabase
supabase start

# 应用迁移
supabase db reset

# 启动本地开发服务器
npm run dev

# 完成后停止
supabase stop
```

---

> [!SUCCESS]
> 本文档全面介绍了 Supabase 的扩展系统、插件生态、性能调优、AI 集成以及最佳实践。Supabase 凭借其 PostgreSQL 核心、完善的扩展支持和 AI 就绪的向量搜索能力，是现代应用开发的优秀选择。