# GraphQL 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 GraphQL 核心概念、Schema 设计、Resolver、N+1 问题、Yoga vs Apollo、DataLoader、Subscriptions，以及在 AI 应用中的结构化数据获取。

---

## 目录

1. [[#GraphQL 概述]]
2. [[#GraphQL vs REST vs tRPC 对比]]
3. [[#类型系统与 SDL]]
4. [[#Schema 设计最佳实践]]
5. [[#Resolver 解析器详解]]
6. [[#查询、变更与订阅]]
7. [[#DataLoader 模式]]
8. [[#GraphQL Yoga vs Apollo Server]]
9. [[#GraphQL + TypeScript]]
10. [[#GraphQL + Next.js]]
11. [[#GraphQL + Prisma]]
12. [[#Schema Stitching 与 Federation]]
13. [[#成本分析与速率限制]]
14. [[#实战：从零构建 GraphQL API]]
15. [[#GraphQL vs tRPC vs REST 2026]]

---

## GraphQL 概述

### 什么是 GraphQL

GraphQL 是一种用于 API 的**查询语言**和**运行时**。它由 Facebook 于 2015 年开源，现已成为主流的 API 技术之一。与传统的 REST API 不同，GraphQL 允许客户端精确指定需要的数据字段，避免了 Overfetching（获取过多数据）和 Underfetching（需要多次请求）的问题。

对于刚接触 GraphQL 的开发者来说，理解 GraphQL 的核心理念很重要：它不是另一种 API 协议，而是一种查询语言和类型系统。你可以把它想象成一个强类型的数据库查询语言，只不过这个"数据库"是你的后端服务。

**GraphQL 核心特点：**

| 特性 | 说明 |
|------|------|
| **精确获取** | 客户端指定所需字段，避免数据传输浪费 |
| **单一端点** | 一个 `/graphql` 端点处理所有请求 |
| **强类型 Schema** | 完整类型系统，IDE 智能提示支持 |
| **自描述** | 运行时自省（Introspection），支持查询 Schema 本身 |
| **实时订阅** | 支持 WebSocket 实时推送 |
| **分层架构** | 查询嵌套对应数据模型结构 |

### GraphQL 的工作原理

当你发送一个 GraphQL 查询时，请求体是一个 POST 请求，内容是这样的：

```json
{
  "query": "query { user(id: \"123\") { name email posts { title } } }"
}
```

服务器解析这个查询，执行对应的 Resolver，返回 JSON 格式的结果。客户端收到的是它请求的精确字段，不多不少。

```json
{
  "data": {
    "user": {
      "name": "Alice",
      "email": "alice@example.com",
      "posts": [
        { "title": "My First Post" },
        { "title": "GraphQL Basics" }
      ]
    }
  }
}
```

---

## GraphQL vs REST vs tRPC 对比

### 功能对比表

| 特性 | GraphQL | REST | tRPC |
|------|---------|------|------|
| **数据获取** | 按需获取 | 固定响应 | 按需获取 |
| **类型系统** | 内置强类型 | 需 OpenAPI | TypeScript 原生 |
| **单一端点** | ✅ | ❌ | ✅ |
| **Overfetching** | 避免 | 常见 | 避免 |
| **Underfetching** | 避免 | 常见 | 避免 |
| **缓存** | 需配置 | HTTP 缓存 | 自动 |
| **实时数据** | Subscription | WebSocket | WebSocket |
| **学习曲线** | 中等 | 低 | 低 |
| **生态系统** | 成熟 | 成熟 | 新兴 |

### 请求格式对比

**REST API：**

```http
GET /api/users/123?_embed=posts&fields=id,name,email
```

```json
{
  "id": "123",
  "name": "Alice",
  "email": "alice@example.com",
  "posts": [
    { "id": 1, "title": "Post 1" },
    { "id": 2, "title": "Post 2" }
  ]
}
```

**GraphQL：**

```graphql
query {
  user(id: "123") {
    id
    name
    email
    posts {
      id
      title
    }
  }
}
```

**tRPC：**

```typescript
// 直接调用 TypeScript 函数
const user = await trpc.user.getById.query({ id: '123' });
```

### 何时选择 GraphQL

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| **复杂数据关系** | GraphQL | 嵌套查询能力强，层级清晰 |
| **移动应用** | GraphQL | 精确获取，节省带宽 |
| **多个前端平台** | GraphQL | 单一 API，统一数据格式 |
| **需要自省** | GraphQL | 内置 Introspection |
| **实时数据需求** | GraphQL | Subscription 支持完善 |
| **TypeScript 项目** | tRPC | 端到端类型安全 |
| **简单 CRUD** | REST | 实现简单，缓存友好 |

---

## 类型系统与 SDL

### GraphQL SDL 基础

GraphQL Schema Definition Language（SDL）是定义 GraphQL Schema 的声明式语言。它的设计非常直观，即使你是第一次接触，也能大致读懂它的含义。

**标量类型：**

```graphql
# 内置标量
scalar String      # 字符串
scalar Int        # 32位整数
scalar Float      # 64位浮点
scalar Boolean    # 布尔值
scalar ID         # 唯一标识符（String）

# 自定义标量
scalar DateTime
scalar JSON
scalar Upload
```

**对象类型：**

```graphql
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  isActive: Boolean!
  createdAt: DateTime!
  posts: [Post!]!
  profile: Profile
}
```

字段说明：`!` 表示非空（必需），`[]` 表示列表，`[Post!]!` 表示非空列表，包含非空的 Post 元素。

**枚举类型：**

```graphql
enum UserRole {
  ADMIN
  USER
  GUEST
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}
```

**接口类型：**

接口允许你定义一个类型必须包含的字段，然后让多个类型实现这个接口：

```graphql
interface Node {
  id: ID!
}

interface Content {
  title: String!
  body: String!
}

type Post implements Node & Content {
  id: ID!
  title: String!
  body: String!
  author: User!
  comments: [Comment!]!
}

type Video implements Node & Content {
  id: ID!
  title: String!
  body: String!
  url: String!
  duration: Int!
}
```

**联合类型：**

联合类型表示一个字段可以是多种类型之一：

```graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}

# 客户端使用片段（Fragment）区分
query {
  search(query: "hello") {
    ... on User {
      name
      email
    }
    ... on Post {
      title
      body
    }
  }
}
```

**输入类型：**

输入类型专门用于 mutation 的参数，它们是独立的类型定义：

```graphql
input CreateUserInput {
  name: String!
  email: String!
  password: String!
  role: UserRole = USER
}

input UpdateUserInput {
  name: String
  email: String
  password: String
}
```

---

## Schema 设计最佳实践

### 完整 Schema 示例

一个设计良好的 GraphQL Schema 应该具备以下特点：清晰的命名、合理的类型组织、完善的分页支持、统一的错误处理模式。

```graphql
# schema.graphql

type Query {
  # 用户相关
  user(id: ID!): User
  users(
    first: Int = 20
    after: String
    filter: UserFilterInput
    sort: [UserSortInput!]
  ): UserConnection!

  # 文章相关
  post(id: ID!): Post
  posts(
    first: Int = 20
    after: String
    filter: PostFilterInput
    sort: [PostSortInput!]
  ): PostConnection!

  # 全局搜索
  search(query: String!): [SearchResult!]!
}

type Mutation {
  # 用户操作
  createUser(input: CreateUserInput!): UserPayload!
  updateUser(id: ID!, input: UpdateUserInput!): UserPayload!
  deleteUser(id: ID!): DeletePayload!

  # 文章操作
  createPost(input: CreatePostInput!): PostPayload!
  updatePost(id: ID!, input: UpdatePostInput!): PostPayload!
  deletePost(id: ID!): DeletePayload!

  # 认证
  login(email: String!, password: String!): AuthPayload!
  register(input: RegisterInput!): AuthPayload!
}

type Subscription {
  # 实时更新
  postCreated: Post!
  postUpdated(id: ID!): Post!
  commentAdded(postId: ID!): Comment!
}

# 节点接口（Relay 风格）
interface Node {
  id: ID!
}

interface Connection {
  totalCount: Int!
  pageInfo: PageInfo!
}

interface Edge {
  node: Node!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type User implements Node {
  id: ID!
  name: String!
  email: String!
  avatar: String
  bio: String
  role: UserRole!
  isActive: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
  posts: [Post!]!
  comments: [Comment!]!
  followerCount: Int!
  followingCount: Int!
}

type UserConnection implements Connection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge implements Edge {
  node: User!
  cursor: String!
}

# 有效负载类型（统一错误处理）
type UserPayload {
  user: User
  errors: [UserError!]!
}

type UserError {
  field: [String!]
  message: String!
  code: String!
}

enum UserRole {
  ADMIN
  USER
  GUEST
}

scalar DateTime
scalar JSON
```

---

## Resolver 解析器详解

### Resolver 基础

Resolver 是解析 GraphQL 字段的函数。每个 Query、Mutation、Subscription 字段，以及每个类型上的每个字段，都需要对应的 Resolver。

```typescript
const resolvers = {
  Query: {
    user: (_, { id }, { prisma }) => {
      return prisma.user.findUnique({ where: { id } });
    },

    users: async (_, args, { prisma }) => {
      return prisma.user.findMany({
        take: args.first,
        skip: args.after ? 1 : 0,
        cursor: args.after ? { id: args.after } : undefined,
      });
    }
  },

  Mutation: {
    createUser: async (_, { input }, { prisma }) => {
      try {
        const user = await prisma.user.create({ data: input });
        return { user, errors: [] };
      } catch (error) {
        return {
          user: null,
          errors: [{ field: ['email'], message: 'Email already exists', code: 'UNIQUE' }]
        };
      }
    }
  },

  User: {
    posts: (user, _, { prisma }) => {
      return prisma.post.findMany({ where: { authorId: user.id } });
    },

    followerCount: (user, _, { prisma }) => {
      return prisma.user.count({
        where: { following: { some: { id: user.id } } }
      });
    },
  },
};
```

### Resolver 参数详解

```typescript
fieldName: (
  parent,           // 父对象
  args,             // 查询参数
  context,          // 共享上下文
  info              // 完整查询信息
) => {
  return resolvedValue;
}
```

**parent（root）：** 对于顶层 Query，parent 是 undefined。对于嵌套字段，parent 是父对象。

```typescript
const resolvers = {
  Post: {
    author: (post) => {
      // post 是 Post 对象
      return getAuthorById(post.authorId);
    }
  }
};
```

**context：** 在所有 Resolver 之间共享的上下文对象。通常包含数据库连接、当前用户信息、缓存等。

```typescript
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization;
    const user = verifyToken(token);

    return {
      prisma,
      user,
      pubsub
    };
  }
});
```

**info：** 包含当前查询的完整信息，可以用来实现查询分析和动态解析。

---

## 查询、变更与订阅

### 查询（Query）

```graphql
# 基本查询
query GetUsers {
  users(first: 10) {
    edges {
      node {
        id
        name
        email
      }
    }
    pageInfo {
      hasNextPage
    }
  }
}

# 带变量
query GetUser($id: ID!, $includePosts: Boolean!) {
  user(id: $id) {
    id
    name
    posts @include(if: $includePosts) {
      id
      title
    }
  }
}
```

### 变更（Mutation）

```graphql
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    post {
      id
      title
    }
    errors {
      field
      message
    }
  }
}

mutation DeletePost($id: ID!) {
  deletePost(id: $id) {
    deleted
    id
  }
}
```

### 订阅（Subscription）

```typescript
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();
const POST_CREATED = 'POST_CREATED';

const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator([POST_CREATED])
    },
  },
};

// 在 Mutation 中触发
const postCreatedResolver = {
  Mutation: {
    createPost: async (_, { input }, { prisma }) => {
      const post = await prisma.post.create({ data: input });
      pubsub.publish(POST_CREATED, { postCreated: post });
      return post;
    },
  },
};
```

---

## DataLoader 模式

### N+1 问题

GraphQL 查询中最常见的问题是 N+1 查询：每个字段都可能触发额外的数据库查询。

```typescript
// 问题示例：查询 10 个用户及其帖子
const resolvers = {
  Query: {
    users: () => db.users.findAll(),  // 1 次查询
  },
  User: {
    posts: (user) => db.posts.findByAuthor(user.id)  // 10 次查询！
  }
};
```

### DataLoader 解决方案

DataLoader 是 Facebook 提供的解决方案，它会自动批处理请求并缓存结果。

```typescript
import DataLoader from 'dataloader';
import { PrismaClient } from '@prisma/client';

function createLoaders() {
  return {
    userLoader: new DataLoader(async (ids: string[]) => {
      // 批量查询，一次数据库调用
      const users = await prisma.user.findMany({
        where: { id: { in: ids } }
      });

      // 按 ID 排序，确保返回顺序与输入一致
      const userMap = new Map(users.map(user => [user.id, user]));
      return ids.map(id => userMap.get(id) || null);
    }),

    postsByAuthorLoader: new DataLoader(async (authorIds: string[]) => {
      const posts = await prisma.post.findMany({
        where: { authorId: { in: authorIds } }
      });

      // 按 authorId 分组
      const postsByAuthor = new Map<string, Post[]>();
      posts.forEach(post => {
        const existing = postsByAuthor.get(post.authorId) || [];
        existing.push(post);
        postsByAuthor.set(post.authorId, existing);
      });

      return authorIds.map(id => postsByAuthor.get(id) || []);
    })
  };
}
```

---

## GraphQL Yoga vs Apollo Server

### 对比概览

GraphQL Yoga 和 Apollo Server 是目前最流行的两个 GraphQL 服务框架。选择哪一个取决于你的具体需求。

| 特性 | GraphQL Yoga | Apollo Server |
|------|-------------|---------------|
| **设计理念** | 现代化、简单易用 | 功能丰富、生态完善 |
| **体积** | 较小 | 较大 |
| **性能** | 优秀 | 良好 |
| **TypeScript** | 原生支持 | 良好支持 |
| **插件系统** | Envelop 插件 | Apollo 插件 |
| **Subscriptions** | 完善 | 完善 |

### GraphQL Yoga 示例

```typescript
import { createSchema, createYoga } from 'graphql-yoga'
import { createServer } from 'node:http'

const schema = createSchema({
  typeDefs: /* GraphQL */ `
    type Query {
      hello: String
    }
    type Mutation {
      createUser(name: String!): User
    }
    type User {
      id: ID!
      name: String!
    }
  `,
  resolvers: {
    Query: {
      hello: () => 'Hello from Yoga!'
    },
    Mutation: {
      createUser: (_, { name }) => ({ id: '1', name })
    }
  }
})

const yoga = createYoga({ schema })
const server = createServer(yoga)
server.listen(4000, () => {
  console.log('Server running at http://localhost:4000/graphql')
})
```

### Apollo Server 示例

```typescript
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: []
});

const { url } = await startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => {
    const user = await verifyToken(req.headers.authorization);
    return { prisma, user };
  }
});
```

---

## GraphQL + TypeScript

### Codegen 配置

使用 GraphQL Code Generator 可以从 Schema 自动生成 TypeScript 类型和操作代码：

```yaml
# codegen.yml
schema: http://localhost:4000/graphql
documents: 'src/**/*.graphql'
generates:
  src/generated/graphql.ts:
    plugins:
      - typescript
      - typescript-resolvers
      - typescript-operations
```

```bash
npm install -D @graphql-codegen/cli
npx graphql-codegen init
```

### 生成类型使用

```typescript
import { useQuery, useMutation } from '@apollo/client';
import { gql } from '@apollo/client';
import type { User, GetUserQuery } from '../generated/graphql';

const GET_USER = gql`
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

function UserProfile({ userId }: { userId: string }) {
  const { data, loading } = useQuery<GetUserQuery>(GET_USER, {
    variables: { id: userId }
  });

  if (loading) return <p>Loading...</p>;
  
  return <div>{data?.user?.name}</div>;
}
```

---

## GraphQL + Next.js

### App Router 集成

```typescript
// app/api/graphql/route.ts
import { createYoga } from 'graphql-yoga'
import { makeExecutableSchema } from '@graphql-tools/schema'
import { typeDefs } from '@/graphql/typeDefs'
import { resolvers } from '@/graphql/resolvers'

const schema = makeExecutableSchema({ typeDefs, resolvers })

const { handleRequest } = createYoga({
  schema,
  graphqlEndpoint: '/api/graphql',
  fetchAPI: { Response }
})

export { handleRequest as GET, handleRequest as POST }
```

### Server Components

```typescript
// app/dashboard/page.tsx
import { getClient } from '@/lib/apollo-client'
import { gql } from '@apollo/client'

const GET_USER = gql`
  query GetUser {
    me {
      id
      name
      email
    }
  }
`

export default async function Dashboard() {
  const { data } = await getClient().query({ query: GET_USER })
  
  return (
    <div>
      <h1>Welcome, {data.me.name}!</h1>
    </div>
  )
}
```

---

## GraphQL + Prisma

### 完整集成示例

```typescript
// lib/graphql.ts
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export async function graphqlContext() {
  return {
    prisma,
    user: await getCurrentUser()
  }
}

export type Context = Awaited<ReturnType<typeof graphqlContext>>

// resolvers/index.ts
import { Context } from '../lib/graphql'

export const resolvers = {
  Query: {
    users: async (_, __, ctx: Context) => {
      return ctx.prisma.user.findMany({
        include: { posts: true }
      })
    },
    
    user: async (_, { id }, ctx: Context) => {
      return ctx.prisma.user.findUnique({
        where: { id },
        include: { posts: true }
      })
    }
  },

  Mutation: {
    createUser: async (_, { input }, ctx: Context) => {
      return ctx.prisma.user.create({
        data: input
      })
    }
  },

  User: {
    posts: async (parent, _, ctx: Context) => {
      return ctx.prisma.post.findMany({
        where: { authorId: parent.id }
      })
    }
  }
}
```

---

## Schema Stitching 与 Federation

### Schema Stitching

Schema Stitching 将多个 GraphQL Schema 合并成一个：

```typescript
import { stitchSchemas } from '@graphql-tools/stitch';
import { makeExecutableSchema } from '@graphql-tools/schema';

const usersSchema = makeExecutableSchema({
  typeDefs: usersTypeDefs,
  resolvers: usersResolvers,
});

const postsSchema = makeExecutableSchema({
  typeDefs: postsTypeDefs,
  resolvers: postsResolvers,
});

const schema = stitchSchemas({
  schemas: [usersSchema, postsSchema],
  typeDefs: `
    extend type User {
      posts: [Post!]!
    }
  `
});
```

### Apollo Federation

Federation 是 Apollo 提出的微服务 GraphQL 架构方案：

```graphql
# Users 服务
type User @key(fields: "id") {
  id: ID!
  name: String!
  email: String!
}

# Products 服务扩展 User
extend type User @key(fields: "id") {
  id: ID! @external
  purchasedProducts: [Product!]!
}
```

---

## 成本分析与速率限制

### 查询复杂度限制

```typescript
import { createComplexityLimitRule } from 'graphql-query-complexity';

const complexityRule = createComplexityLimitRule(1000, {
  onCost: (cost) => console.log(`Query cost: ${cost}`),
});

const server = new ApolloServer({
  schema,
  validationRules: [complexityRule]
});
```

### 深度限制

```typescript
import { createDepthLimitRule } from 'graphql-depth-limit';

const depthRule = createDepthLimitRule(10, {
  ignore: ['_service', '_entities'],
});

const server = new ApolloServer({
  schema,
  validationRules: [depthRule]
});
```

### 速率限制

```typescript
import { createRateLimitRule } from 'graphql-rate-limit';

const rateLimitRule = createRateLimitRule({
  identifyContext: (ctx) => ctx.id || ctx.ip,
  rateLimit: {
    window: '10s',
    max: 100,
  },
});
```

---

## 实战：从零构建 GraphQL API

### 项目初始化

```bash
mkdir my-graphql-api
cd my-graphql-api
npm init -y
npm install graphql graphql-yoga @prisma/client
npm install -D prisma typescript @types/node
npx tsc --init
```

### 创建 Schema

```graphql
# schema.graphql

type Query {
  users: [User!]!
  user(id: ID!): User
  posts: [Post!]!
  post(id: ID!): Post
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User
  deleteUser(id: ID!): Boolean!
  createPost(input: CreatePostInput!): Post!
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
  createdAt: String!
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User!
  createdAt: String!
}

input CreateUserInput {
  name: String!
  email: String!
}

input UpdateUserInput {
  name: String
  email: String
}

input CreatePostInput {
  title: String!
  content: String
  authorId: ID!
}
```

### 创建 Resolvers

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

export const resolvers = {
  Query: {
    users: () => prisma.user.findMany({ include: { posts: true } }),
    user: (_, { id }) => prisma.user.findUnique({ 
      where: { id }, 
      include: { posts: true } 
    }),
    posts: () => prisma.post.findMany({ include: { author: true } }),
    post: (_, { id }) => prisma.post.findUnique({ 
      where: { id }, 
      include: { author: true } 
    }),
  },

  Mutation: {
    createUser: (_, { input }) => 
      prisma.user.create({ data: input }),
    
    updateUser: (_, { id, input }) =>
      prisma.user.update({ where: { id }, data: input }),
    
    deleteUser: (_, { id }) =>
      prisma.user.delete({ where: { id } }).then(() => true),
    
    createPost: (_, { input }) =>
      prisma.post.create({ data: input, include: { author: true } }),
  },

  User: {
    posts: (parent) => 
      prisma.post.findMany({ where: { authorId: parent.id } }),
  },

  Post: {
    author: (parent) =>
      prisma.user.findUnique({ where: { id: parent.authorId } }),
  },
}
```

### 启动服务器

```typescript
import { createSchema, createYoga } from 'graphql-yoga'
import { readFileSync } from 'fs'
import { resolvers } from './resolvers'

const typeDefs = readFileSync('./schema.graphql', 'utf-8')

const schema = createSchema({ typeDefs, resolvers })
const yoga = createYoga({ schema })

const server = createServer(yoga)
server.listen(4000, () => {
  console.log('GraphQL API ready at http://localhost:4000/graphql')
})
```

---

## GraphQL vs tRPC vs REST 2026

### 技术选型决策树

```
项目需求
    │
    ├─ 是否是 TypeScript 项目？
    │       ├─ 是 ──→ 团队规模？
    │       │       ├─ 小型 ──→ tRPC
    │       │       └─ 大型 ──→ GraphQL / tRPC
    │       └─ 否 ──→ 是否需要多语言支持？
    │               ├─ 是 ──→ GraphQL / REST
    │               └─ 否 ──→ REST
    │
    ├─ 是否需要公开 API？
    │       ├─ 是 ──→ REST（标准化、缓存友好）
    │       └─ 否
    │
    ├─ 数据关系复杂度？
    │       ├─ 简单 ──→ REST / tRPC
    │       └─ 复杂 ──→ GraphQL
    │
    └─ 是否需要实时数据？
            ├─ 是 ──→ GraphQL Subscription / tRPC SSE
            └─ 否
```

### 2026 年推荐

| 场景 | 推荐方案 | 理由 |
|------|---------|------|
| **全栈 TypeScript MVP** | tRPC + Prisma | 最快开发速度，端到端类型安全 |
| **多团队大型项目** | GraphQL Federation | 清晰的边界，微服务友好 |
| **移动应用后端** | GraphQL | 精确数据获取，离线支持 |
| **公共 API** | REST + OpenAPI | 标准化，工具链完善 |
| **AI 应用** | tRPC / GraphQL | 强类型，结构化输出 |

---

> [!SUCCESS]
> GraphQL 以其精确的数据获取能力和强类型 Schema，为 API 设计提供了全新的范式。无论是构建小型应用还是大型微服务架构，GraphQL 都能提供优秀的开发体验。配合合适的工具链（Yoga、Apollo、Codegen），可以构建出高效、可维护的 GraphQL API。
