# GraphQL 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 GraphQL 核心概念、Schema 设计、Resolver、N+1 问题及在 AI 应用中的结构化数据获取。

---

## 目录

1. [[#GraphQL 概述]]
2. [[#GraphQL vs REST vs tRPC 对比]]
3. [[#类型系统与 SDL]]
4. [[#Schema 设计]]
5. [[#Resolver 解析器]]
6. [[#查询（Query）]]
7. [[#变更（Mutation）]]
8. [[#订阅（Subscription）]]
9. [[#N+1 问题与 DataLoader]]
10. [[#Apollo Server 与 Client]]
11. [[#AI 应用实战]]
12. [[#选型建议]]

---

## GraphQL 概述

### 什么是 GraphQL

GraphQL 是一种用于 API 的**查询语言**和**运行时**。由 Facebook 于 2015 年开源，现已成为主流的 API 技术之一。GraphQL 的核心理念是让客户端精确指定需要的数据，避免 Overfetching 和 Underfetching。

**GraphQL 核心特点：**

| 特性 | 说明 |
|------|------|
| **精确获取** | 客户端指定所需字段 |
| **单一端点** | 一个 `/graphql` 端点处理所有请求 |
| **强类型 Schema** | 完整类型系统，IDE 支持 |
| **自描述** | 运行时自省（Introspection） |
| **实时订阅** | 支持 WebSocket 实时推送 |

### GraphQL 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| GraphQL 规范 | 2015 | Facebook 开源 |
| GraphQL.js | 2016 | JavaScript 实现 |
| Apollo Server | 2016 | GraphQL 服务端 |
| Apollo Client | 2017 | GraphQL 客户端 |
| GraphQL Yoga | 2020 | 现代 GraphQL 服务 |
| **GraphQL 2026** | **2026** | **更好的 TypeScript 支持** |

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

### 适用场景

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| **复杂数据关系** | GraphQL | 嵌套查询能力强 |
| **移动应用** | GraphQL | 精确获取，节省带宽 |
| **公共 API** | REST | 标准化，缓存友好 |
| **TypeScript 项目** | tRPC | 端到端类型安全 |
| **微服务** | REST/gRPC | 成熟稳定 |
| **实时数据** | GraphQL | Subscription 支持 |

---

## 类型系统与 SDL

### GraphQL SDL 基础

GraphQL Schema Definition Language（SDL）是定义 GraphQL Schema 的声明式语言。

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

**字段说明：**
- `!` 表示非空（必需）
- `[]` 表示列表

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

type User {
  role: UserRole!
  status: PostStatus!
}
```

**接口类型：**

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

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
}
```

---

## Schema 设计

### 完整 Schema 示例

```graphql
# schema.graphql

# 类型定义
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

# 分页类型
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# 用户类型
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

  # 关系
  posts: [Post!]!
  comments: [Comment!]!
  followers: UserConnection!
  following: UserConnection!
  followerCount: Int!
  followingCount: Int!
}

type UserConnection implements Connection & Node {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge implements Edge {
  node: User!
  cursor: String!
}

# 文章类型
type Post implements Node {
  id: ID!
  title: String!
  slug: String!
  excerpt: String
  content: String!
  coverImage: String
  status: PostStatus!
  viewCount: Int!
  createdAt: DateTime!
  updatedAt: DateTime!
  publishedAt: DateTime

  # 关系
  author: User!
  comments: [Comment!]!
  tags: [Tag!]!
  commentCount: Int!
}

type PostConnection implements Connection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge implements Edge {
  node: Post!
  cursor: String!
}

# 评论类型
type Comment implements Node {
  id: ID!
  body: String!
  createdAt: DateTime!

  author: User!
  post: Post!
}

# 标签类型
type Tag implements Node {
  id: ID!
  name: String!
  slug: String!
  postCount: Int!
}

# 联合类型
union SearchResult = User | Post | Comment

# 有效负载类型
type UserPayload {
  user: User
  errors: [UserError!]!
}

type PostPayload {
  post: Post
  errors: [PostError!]!
}

type AuthPayload {
  token: String
  user: User
  errors: [UserError!]!
}

type DeletePayload {
  deleted: Boolean!
  id: ID
}

# 错误类型
type UserError {
  field: [String!]
  message: String!
  code: String!
}

type PostError {
  field: [String!]
  message: String!
  code: String!
}

# 枚举
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

# 自定义标量
scalar DateTime
scalar JSON
scalar Upload

# 输入类型
input UserFilterInput {
  role: UserRole
  isActive: Boolean
  search: String
}

input UserSortInput {
  field: UserSortField!
  direction: SortDirection!
}

enum UserSortField {
  NAME
  CREATED_AT
  UPDATED_AT
}

enum SortDirection {
  ASC
  DESC
}
```

---

## Resolver 解析器

### Resolver 基础

Resolver 是解析 GraphQL 字段的函数：

```typescript
// resolvers.ts
const resolvers = {
  Query: {
    // user: (root, args, context, info) => { }
    user: (_, { id }, { prisma }) => {
      return prisma.user.findUnique({ where: { id } });
    },

    users: async (_, args, { prisma }) => {
      return prisma.user.findMany({
        take: args.first,
        skip: args.after ? 1 : 0,
        cursor: args.after ? { id: args.after } : undefined,
        where: args.filter,
        orderBy: args.sort?.map(s => ({ [s.field]: s.direction }))
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
    // 嵌套字段解析器
    posts: (user, _, { prisma }) => {
      return prisma.post.findMany({ where: { authorId: user.id } });
    },

    followerCount: (user, _, { prisma }) => {
      return prisma.user.count({
        where: { following: { some: { id: user.id } } }
      });
    },

    // 模拟字段（计算得出）
    fullName: (user) => {
      return `${user.firstName} ${user.lastName}`;
    }
  },

  Post: {
    // 转换字段
    createdAt: (post) => post.createdAt.toISOString(),

    // 异步解析
    comments: (post, _, { prisma }) => {
      return prisma.comment.findMany({ where: { postId: post.id } });
    }
  },

  // 接口解析
  Node: {
    __resolveType: (obj) => {
      if (obj.title) return 'Post';
      if (obj.email) return 'User';
      if (obj.body) return 'Comment';
      return null;
    }
  },

  // 联合类型解析
  SearchResult: {
    __resolveType: (obj) => {
      if (obj.title) return 'Post';
      if (obj.email) return 'User';
      if (obj.body && obj.authorId) return 'Comment';
      return null;
    }
  }
};
```

### Resolver 参数详解

```typescript
// resolver 函数签名
fieldName: (
  parent,           // 父对象（对于 Query 是 undefined）
  args,             // 查询参数
  context,          // 共享上下文
  info              // 完整查询信息
) => {
  // 解析逻辑
  return resolvedValue;
}
```

**parent（root）：**

```typescript
const resolvers = {
  Post: {
    author: (post) => {
      // post 是 Post 对象
      // 可以直接使用 post.authorId
      return getAuthorById(post.authorId);
    }
  }
};
```

**args：**

```graphql
query {
  user(id: "123", includePosts: true) {  # args = { id: "123", includePosts: true }
    name
  }
}
```

**context：**

```typescript
// Server setup
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // 从请求中提取用户
    const token = req.headers.authorization;
    const user = verifyToken(token);

    return {
      prisma,
      user,
      pubsub
    };
  }
});

// Resolver 中使用
const resolvers = {
  Query: {
    me: (_, __, { user }) => {
      if (!user) throw new AuthenticationError('Not authenticated');
      return user;
    }
  }
};
```

**info：**

```typescript
const resolvers = {
  Query: {
    posts: (_, __, ___, info) => {
      // 获取查询路径
      const path = info.path;

      // 获取选择的字段
      const selections = info.fieldNodes[0].selectionSet.selections;

      // 获取变量
      const variableValues = info.variableValues;

      return prisma.post.findMany();
    }
  }
};
```

---

## 查询（Query）

### 基本查询

```graphql
# 获取用户列表
query GetUsers {
  users(first: 10) {
    edges {
      node {
        id
        name
        email
        avatar
      }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
    totalCount
  }
}

# 获取单个用户
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    posts {
      id
      title
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

### 片段（Fragment）

```graphql
fragment UserFields on User {
  id
  name
  email
  avatar
  createdAt
}

fragment PostFields on Post {
  id
  title
  excerpt
  coverImage
  author {
    ...UserFields
  }
}

query {
  posts(first: 10) {
    edges {
      node {
        ...PostFields
      }
    }
  }
}
```

### 别名与指令

```graphql
# 别名（处理同名字段）
query {
  admin: user(id: "1") {
    name
    role
  }
  currentUser: user(id: "current") {
    name
    role
  }
}

# 内置指令
query {
  # 条件包含
  secretField @skip(if: true)
  publicField @include(if: true)

  # 自定义指令
  authorized @hasRole(role: ADMIN) {
    name
  }
}
```

---

## 变更（Mutation）

### 基本变更

```graphql
# 创建
mutation CreatePost($input: CreatePostInput!) {
  createPost(input: $input) {
    post {
      id
      title
      content
    }
    errors {
      field
      message
      code
    }
  }
}

# 更新
mutation UpdatePost($id: ID!, $input: UpdatePostInput!) {
  updatePost(id: $id, input: $input) {
    post {
      id
      title
      content
    }
    errors {
      field
      message
      code
    }
  }
}

# 删除
mutation DeletePost($id: ID!) {
  deletePost(id: $id) {
    deleted
    id
  }
}
```

### 批量操作

```graphql
mutation PublishPosts($ids: [ID!]!) {
  publishPosts(ids: $ids) {
    posts {
      id
      status
      publishedAt
    }
    errors {
      id
      message
    }
  }
}
```

### 事务变更

```graphql
mutation CreateUserWithFirstPost($userInput: CreateUserInput!, $postInput: CreatePostInput!) {
  createUser(input: $userInput) {
    user {
      id
      name
    }
    errors {
      message
    }
  }
}
```

---

## 订阅（Subscription）

### 定义订阅

```graphql
type Subscription {
  # 新文章通知
  postCreated: Post!

  # 文章更新
  postUpdated(id: ID!): Post!

  # 评论通知
  commentAdded(postId: ID!): Comment!

  # 用户在线状态
  userOnline(userId: ID!): UserPresence!
}
```

### 实现订阅

```typescript
// 使用 PubSub
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator(['POST_CREATED'])
    },

    postUpdated: {
      subscribe: (_, { id }) => {
        return pubsub.asyncIterator(`POST_UPDATED_${id}`);
      }
    },

    commentAdded: {
      subscribe: (_, { postId }) => {
        return pubsub.asyncIterator(`COMMENT_ADDED_${postId}`);
      }
    }
  }
};

// 触发事件
async function createPost(input) {
  const post = await prisma.post.create({ data: input });

  // 发布事件
  pubsub.publish('POST_CREATED', { postCreated: post });

  return { post };
}
```

### 客户端订阅

```typescript
import { gql, useSubscription } from '@apollo/client';

const POST_CREATED_SUBSCRIPTION = gql`
  subscription OnPostCreated {
    postCreated {
      id
      title
      author {
        name
      }
    }
  }
`;

function PostList() {
  const { data, loading } = useSubscription(POST_CREATED_SUBSCRIPTION);

  if (loading) return <p>Loading...</p>;

  return (
    <ul>
      {data?.postCreated && (
        <li key={data.postCreated.id}>
          {data.postCreated.title}
        </li>
      )}
    </ul>
  );
}
```

---

## N+1 问题与 DataLoader

### N+1 问题

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

```typescript
import DataLoader from 'dataloader';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// 创建 DataLoader
function createLoaders() {
  return {
    userLoader: new DataLoader(async (ids: string[]) => {
      // 批量查询，一次数据库调用
      const users = await prisma.user.findMany({
        where: { id: { in: ids } }
      });

      // 按 ID 排序返回
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

// 在 context 中使用
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: () => ({
    loaders: createLoaders(),
    prisma
  })
});

// Resolver 中使用
const resolvers = {
  User: {
    posts: (user, _, { loaders }) => {
      return loaders.postsByAuthorLoader.load(user.id);
    }
  }
};
```

### 缓存与批处理

```typescript
// DataLoader 默认行为
// 1. 自动批处理：同一个 tick 内的请求合并
// 2. 自动缓存：结果被缓存
// 3. 单次加载：load() 调用结果会被缓存

// 清除缓存
const loader = new DataLoader(...);
loader.load(id);
loader.clear(id);      // 清除单个
loader.clearAll();     // 清除全部

// Prime（预加载缓存）
loader.prime(id, value);
```

---

## Apollo Server 与 Client

### Apollo Server 快速开始

```typescript
// server.ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { typeDefs } from './schema';
import { resolvers } from './resolvers';

async function startServer() {
  const server = new ApolloServer({
    typeDefs,
    resolvers,
    plugins: [
      // 插件
    ]
  });

  const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
    context: async ({ req }) => {
      // 验证 Token
      const token = req.headers.authorization || '';
      const user = await verifyToken(token);

      return {
        user,
        prisma: new PrismaClient(),
        pubsub: new PubSub()
      };
    }
  });

  console.log(`Server ready at ${url}`);
}

startServer();
```

### Apollo Client 快速开始

```typescript
// client.ts
import { ApolloClient, InMemoryCache, HttpLink, split } from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { getMainDefinition } from '@apollo/client/utilities';
import { createClient } from 'graphql-ws';

const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql'
});

// WebSocket 链接（订阅）
const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql'
  })
);

// 分割链接
const splitLink = split(
  ({ query }) => {
    const definition = getMainDefinition(query);
    return (
      definition.kind === 'OperationDefinition' &&
      definition.operation === 'subscription'
    );
  },
  wsLink,
  httpLink
);

// 创建客户端
export const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          users: {
            keyArgs: false,
            merge(existing = { edges: [] }, incoming) {
              return {
                ...incoming,
                edges: [...existing.edges, ...incoming.edges]
              };
            }
          }
        }
      }
    }
  })
});
```

---

## AI 应用实战

### 结构化数据获取

GraphQL 的精确字段选择特别适合 AI 应用：

```typescript
// AI 应用中的 GraphQL 查询
const AI_PROMPTS_QUERY = `
  query GetPromptContext($promptId: ID!) {
    prompt(id: $promptId) {
      id
      title
      systemPrompt
      temperature
      maxTokens
      model {
        id
        name
        provider
      }
      documents {
        id
        name
        content
      }
      examples {
        input
        output
      }
    }
  }
`;

// AI 调用
async function generateFromPrompt(promptId: string, userInput: string) {
  const { data } = await client.query({
    query: gql(AI_PROMPTS_QUERY),
    variables: { promptId }
  });

  const { prompt } = data;

  const response = await openai.chat.completions.create({
    model: prompt.model.name,
    messages: [
      { role: 'system', content: prompt.systemPrompt },
      ...prompt.examples.map(ex => [
        { role: 'user', content: ex.input },
        { role: 'assistant', content: ex.output }
      ]).flat(),
      { role: 'user', content: userInput }
    ],
    temperature: prompt.temperature,
    max_tokens: prompt.maxTokens
  });

  return response.choices[0].message.content;
}
```

### 实时 AI 生成

```typescript
// Subscription 实现流式更新
const resolvers = {
  Subscription: {
    aiGenerationProgress: {
      subscribe: async function* (_, { taskId }, { pubsub }) {
        // 模拟 AI 生成进度
        for (let i = 0; i <= 100; i += 10) {
          await new Promise(resolve => setTimeout(resolve, 100));
          yield {
            aiGenerationProgress: {
              taskId,
              progress: i,
              status: i < 100 ? 'processing' : 'completed'
            }
          };
        }
      }
    }
  }
};

// 客户端使用
const { data } = useSubscription(
  AI_GENERATION_PROGRESS_SUBSCRIPTION,
  { variables: { taskId } }
);
```

---

## 选型建议

### 何时选择 GraphQL

| 场景 | 推荐理由 |
|------|---------|
| **复杂数据关系** | 嵌套查询能力强 |
| **移动应用** | 精确获取，节省带宽 |
| **多个前端平台** | 单一 API，统一数据 |
| **需要自省** | 内置 Introspection |
| **实时数据需求** | Subscription 支持 |
| **AI 应用** | 结构化数据获取 |

### 何时考虑 REST

| 场景 | 原因 |
|------|------|
| **简单 CRUD** | 实现简单 |
| **公共 API** | 标准化，缓存友好 |
| **HTTP 缓存** | 浏览器原生支持 |
| **微服务** | 成熟稳定 |
| **CDN 部署** | 边缘缓存简单 |

> [!TIP]
> GraphQL 的优势在于数据获取的灵活性，但在小项目中可能过度设计。评估项目需求后再做选择。

---

## GraphQL 高级特性

### 上下文与依赖注入

GraphQL 的 Context 是一个强大的模式，用于在所有 Resolver 之间共享数据：

```typescript
// 完整的 Context 类型定义
interface Context {
  prisma: PrismaClient;
  user: User | null;
  pubsub: PubSub;
  cache: Map<string, any>;
  requestId: string;
}

// Context 工厂函数
function createContext({ req }: any): Context {
  return {
    prisma: new PrismaClient(),
    user: extractUserFromRequest(req),
    pubsub: new PubSub(),
    cache: new Map(),
    requestId: req.headers['x-request-id'] || generateUUID()
  };
}

// Apollo Server 中使用
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req, connection }) => {
    // HTTP 请求上下文
    if (!connection) {
      return {
        prisma: prisma,
        user: await getUserFromToken(req.headers.authorization),
        pubsub,
        cache: await getCachedData(req.headers['x-cache-key']),
        requestId: req.headers['x-request-id']
      };
    }
    
    // WebSocket 连接上下文
    return {
      prisma: prisma,
      user: connection.context.user,
      pubsub,
      cache: new Map(),
      requestId: connection.context.requestId
    };
  },
  formatError: (error) => {
    // 统一错误格式化
    return {
      message: error.message,
      locations: error.locations,
      path: error.path,
      requestId: error.extensions?.requestId
    };
  }
});
```

### 字段级权限控制

```typescript
// 基于指令的权限控制
const directiveSchema = gql`
  directive @auth(requires: Role = USER) on FIELD_DEFINITION | OBJECT
  
  enum Role {
    ADMIN
    USER
    GUEST
  }
  
  type User @auth(requires: ADMIN) {
    id: ID!
    email: String!
    role: Role!
    secretField: String @auth(requires: ADMIN)
  }
`;

// 指令实现
const authDirective = (schema, getUser) => {
  return mapSchema(schema, {
    [MapperKind.OBJECT_FIELD]: (fieldConfig, fieldDetails, objDetails) => {
      const { requires } = getDirective(
        fieldConfig,
        directiveSchema,
        'auth'
      )?.[0] || { requires: 'USER' };
      
      return {
        ...fieldConfig,
        resolve: async (source, args, context, info) => {
          const user = getUser(context);
          
          if (!hasRole(user, requires)) {
            throw new ForbiddenError(
              `Requires ${requires} role`
            );
          }
          
          return fieldConfig.resolve?.(source, args, context, info);
        }
      };
    }
  });
};

// Resolver 级权限装饰器
function requireAuth(allowedRoles?: Role[]) {
  return (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) => {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args) {
      const [, , context] = args;
      
      if (!context.user) {
        throw new AuthenticationError('Not authenticated');
      }
      
      if (
        allowedRoles && 
        !allowedRoles.includes(context.user.role)
      ) {
        throw new ForbiddenError(
          `Requires one of: ${allowedRoles.join(', ')}`
        );
      }
      
      return originalMethod.apply(this, args);
    };
    
    return descriptor;
  };
}

const resolvers = {
  Query: {
    @requireAuth()
    me: (_, __, context) => context.user,
    
    @requireAuth(Role.ADMIN)
    allUsers: (_, __, context) => context.prisma.user.findMany()
  }
};
```

### 性能监控与追踪

```typescript
import { ApolloServerPluginLandingPageLocalDefault } from '@apollo/server/plugin/landingPage/default';

// OpenTelemetry 集成
import { NodeSDK } from '@opentelemetry/sdk-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { GraphQLInstrumentation } from '@opentelemetry/instrumentation-graphql';

const sdk = new NodeSDK({
  exporter: new JaegerExporter(),
  instrumentations: [
    new GraphQLInstrumentation({
      // 追踪所有 GraphQL 操作
      mergeItems: true,
      // 不追踪变量内容（保护敏感数据）
      ignoreTrivialResolveSpans: false
    })
  ]
});

// Apollo 插件：记录慢查询
const slowQueryPlugin = {
  async didResolveOperation({ request, operationName }) {
    const startTime = Date.now();
    
    request.operationStartTime = startTime;
    request.operationName = operationName?.name?.value;
  },
  
  async didEncounterErrors({ request, errors }) {
    const duration = Date.now() - (request.operationStartTime || 0);
    
    console.error({
      type: 'graphql_error',
      requestId: request.operationId,
      operationName: request.operationName,
      errors: errors.map(e => ({
        message: e.message,
        path: e.path
      })),
      duration
    });
  }
};

// 生产环境插件
const productionPlugin = {
  async requestDidStart({ request }) {
    return {
      async willSendResponse({ response }) {
        const { operationName, operation } = response.body.singleResult;
        
        // 记录操作指标
        metrics.record('graphql_operation', {
          name: operationName,
          type: operation?.operation,
          fieldCount: countFields(operation),
          errors: response.body.singleResult.errors?.length || 0
        });
      }
    };
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    ApolloServerPluginLandingPageLocalDefault(),
    slowQueryPlugin,
    productionPlugin
  ]
});
```

### 联邦架构（Apollo Federation）

```graphql
# 子服务 A: Users
# users subgraph
type User @key(fields: "id") {
  id: ID!
  name: String!
  email: String!
}

type Query {
  user(id: ID!): User
  me: User
}
```

```typescript
// users-service/server.ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { buildSubgraphSchema } from '@apollo/subgraph';

const typeDefs = gql`
  type User @key(fields: "id") {
    id: ID!
    name: String!
    email: String!
  }
  
  extend type Query {
    user(id: ID!): User
    me: User
  }
`;

const resolvers = {
  User: {
    __resolveReference: ({ id }) => {
      return getUserById(id);
    }
  },
  Query: {
    user: (_, { id }) => getUserById(id),
    me: (_, __, { user }) => user
  }
};

const server = new ApolloServer({
  schema: buildSubgraphSchema({ typeDefs, resolvers })
});
```

```graphql
# Products 服务使用 User 引用
# products subgraph
type Product @key(fields: "id") {
  id: ID!
  name: String!
  price: Float!
  seller: User!  # 引用 Users 子服务
}
```

```typescript
# Products 服务 Resolver
const resolvers = {
  Product: {
    seller: {
      selectionSet: `{ id }`,
      resolve(parent, _, { dataLoaders }) {
        return dataLoaders.userLoader.load(parent.sellerId);
      }
    }
  }
};
```

### 实战：完整的 GraphQL API 项目结构

```
my-graphql-api/
├── src/
│   ├── schema/
│   │   ├── index.ts           # 合并所有子 schema
│   │   ├── typeDefs/          # 类型定义
│   │   │   ├── user.ts
│   │   │   ├── post.ts
│   │   │   └── comment.ts
│   │   └── resolvers/         # 解析器
│   │       ├── index.ts
│   │       ├── user.ts
│   │       ├── post.ts
│   │       └── comment.ts
│   ├── models/                # 数据模型
│   │   ├── user.ts
│   │   ├── post.ts
│   │   └── comment.ts
│   ├── services/              # 业务逻辑
│   │   ├── auth.service.ts
│   │   ├── email.service.ts
│   │   └── cache.service.ts
│   ├── directives/            # 自定义指令
│   │   ├── auth.ts
│   │   ├── rateLimit.ts
│   │   └── deprecated.ts
│   ├── middleware/           # 中间件
│   │   ├── logging.ts
│   │   └── errorHandler.ts
│   ├── plugins/              # Apollo 插件
│   │   └── performance.ts
│   ├── context.ts            # Context 配置
│   └── server.ts             # 服务器入口
├── tests/
│   ├── resolvers/
│   ├── schema/
│   └── integration/
├── prisma/
│   └── schema.prisma
├── package.json
└── tsconfig.json
```

```typescript
// src/schema/index.ts - Schema 合并
import { mergeTypeDefs, mergeResolvers } from '@graphql-tools/merge';
import { loadFilesSync } from '@graphql-tools/load-files';
import path from 'path';

const typesArray = loadFilesSync(
  path.join(__dirname, './typeDefs'),
  { extensions: ['.ts', '.graphql'] }
);

const resolversArray = loadFilesSync(
  path.join(__dirname, './resolvers'),
  { extensions: ['.ts'] }
);

export const typeDefs = mergeTypeDefs(typesArray);
export const resolvers = mergeResolvers(resolversArray);

// src/server.ts - 服务器配置
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { makeExecutableSchema } from '@graphql-tools/schema';
import { ApolloServerPluginLandingPageLocalDefault } from '@apollo/server/plugin/landingPage/default';

import { typeDefs } from './schema';
import { resolvers } from './schema';
import { createContext } from './context';
import { authDirective } from './directives/auth';

const schema = makeExecutableSchema({ typeDefs, resolvers });
const schemaWithAuth = authDirective(schema);

const server = new ApolloServer({
  schema: schemaWithAuth,
  plugins: [
    ApolloServerPluginLandingPageLocalDefault({
      includeCookies: true
    })
  ],
  formatError: (error) => {
    // 生产环境不暴露内部错误
    if (process.env.NODE_ENV === 'production') {
      return new GraphQLError('Internal server error');
    }
    return error;
  }
});

const { url } = await startStandaloneServer(server, {
  context: createContext,
  listen: { port: 4000 }
});

console.log(`🚀 Server ready at ${url}`);
```

> [!SUCCESS]
> GraphQL 以其精确的数据获取能力和强类型 Schema，为 API 设计提供了全新的范式。虽然学习曲线比 REST 稍陡，但在复杂数据场景和 AI 应用中展现出明显优势。配合 Apollo 生态，GraphQL 能够构建出灵活、高效的数据 API。

---

## GraphQL 高级模式与最佳实践

### DataLoader 与 N+1 问题解决

GraphQL 查询中最常见的问题是 N+1 查询：每个字段都可能触发额外的数据库查询。

```typescript
// DataLoader 实现
import DataLoader from 'dataloader';

// 创建批量加载函数
const createUserLoader = (context: Context) =>
  new DataLoader<string, User>(async (ids) => {
    const users = await context.prisma.user.findMany({
      where: { id: { in: ids } },
    });
    
    // 按 ID 排序，确保返回顺序与输入一致
    const userMap = new Map(users.map((user) => [user.id, user]));
    return ids.map((id) => userMap.get(id) || null);
  });

// 在 Context 中创建所有 DataLoader
export const createContext = async ({ req }: ExpressRequest) => {
  return {
    prisma: new PrismaClient(),
    loaders: {
      user: createUserLoader({ prisma }),
      post: createPostLoader({ prisma }),
    },
  };
};

type Context = {
  prisma: PrismaClient;
  loaders: {
    user: DataLoader<string, User | null>;
    post: DataLoader<string, Post | null>;
  };
};

// 在 Resolver 中使用
const resolvers = {
  Post: {
    author: (post: Post, _: any, context: Context) => {
      // 批量加载，避免 N+1
      return context.loaders.user.load(post.authorId);
    },
    comments: async (post: Post, _: any, context: Context) => {
      // 使用 Post 特有的 loader
      return context.loaders.post.load(post.id);
    },
  },
  User: {
    posts: async (user: User, args: { limit?: number }, context: Context) => {
      return context.prisma.post.findMany({
        where: { authorId: user.id },
        take: args.limit || 10,
      });
    },
  },
};
```

### 分页实现策略

GraphQL 中有多种分页策略，各有优劣：

```typescript
// 1. Offset 风格分页
type OffsetPagination = {
  posts: {
    items: Post[];
    total: number;
    hasMore: boolean;
  };
};

const postsResolver = async (
  _: any,
  { page = 1, limit = 10 }: { page?: number; limit?: number },
  { prisma }: Context
) => {
  const skip = (page - 1) * limit;
  
  const [items, total] = await Promise.all([
    prisma.post.findMany({
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
    }),
    prisma.post.count(),
  ]);
  
  return {
    items,
    total,
    hasMore: skip + items.length < total,
  };
};

// 2. Cursor 风格分页（Relay Connection）
type PostConnection = {
  edges: PostEdge[];
  pageInfo: PageInfo;
};

type PostEdge = {
  node: Post;
  cursor: string;
};

type PageInfo = {
  hasNextPage: boolean;
  hasPreviousPage: boolean;
  startCursor: string | null;
  endCursor: string | null;
};

const encodeCursor = (post: Post): string => {
  return Buffer.from(JSON.stringify({
    id: post.id,
    createdAt: post.createdAt.toISOString(),
  })).toString('base64');
};

const decodeCursor = (cursor: string): { id: string; createdAt: string } => {
  return JSON.parse(Buffer.from(cursor, 'base64').toString());
};

const postsConnectionResolver = async (
  _: any,
  { first = 10, after }: { first?: number; after?: string },
  { prisma }: Context
) => {
  let cursor: { id?: string; createdAt?: Date } | undefined;
  
  if (after) {
    const decoded = decodeCursor(after);
    cursor = {
      id: decoded.id,
      createdAt: new Date(decoded.createdAt),
    };
  }
  
  const items = await prisma.post.findMany({
    take: first + 1, // 多取一条判断 hasNextPage
    ...(cursor
      ? {
          where: {
            OR: [
              { createdAt: { lt: cursor.createdAt } },
              {
                createdAt: { equals: cursor.createdAt },
                id: { lt: cursor.id },
              },
            ],
          },
        }
      : {}),
    orderBy: [{ createdAt: 'desc' }, { id: 'desc' }],
  });
  
  const hasNextPage = items.length > first;
  const edges = (hasNextPage ? items.slice(0, -1) : items).map((post) => ({
    node: post,
    cursor: encodeCursor(post),
  }));
  
  return {
    edges,
    pageInfo: {
      hasNextPage,
      hasPreviousPage: !!after,
      startCursor: edges[0]?.cursor || null,
      endCursor: edges[edges.length - 1]?.cursor || null,
    },
  };
};
```

### 订阅与实时数据

GraphQL Subscriptions 允许服务器主动推送数据到客户端：

```typescript
// 订阅类型定义
type Subscription {
  postCreated: Post!
  postUpdated(id: ID!): Post!
  commentAdded(postId: ID!): Comment!
  userPresenceUpdated: PresenceUpdate!
}

// PubSub 实现
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();

// 事件常量
const POST_CREATED = 'POST_CREATED';
const COMMENT_ADDED = 'COMMENT_ADDED';

// Resolver
const resolvers = {
  Subscription: {
    postCreated: {
      subscribe: () => pubsub.asyncIterator([POST_CREATED]),
    },
    
    postUpdated: {
      subscribe: (_: any, { id }: { id: string }) => {
        const channel = `POST_UPDATED_${id}`;
        return pubsub.asyncIterator([channel]);
      },
      resolve: (payload) => payload,
    },
    
    commentAdded: {
      subscribe: (_: any, { postId }: { postId: string }) => {
        const channel = `COMMENT_ADDED_${postId}`;
        return pubsub.asyncIterator([channel]);
      },
    },
  },
};

// 触发订阅
const postCreatedResolver = {
  Mutation: {
    createPost: async (
      _: any,
      { input }: { input: CreatePostInput },
      { prisma }: Context
    ) => {
      const post = await prisma.post.create({ data: input });
      
      // 发布事件
      pubsub.publish(POST_CREATED, { postCreated: post });
      
      return post;
    },
  },
};

// WebSocket 订阅设置（使用 graphql-ws）
import { createServer } from 'http';
import { useServer } from 'graphql-ws/lib/use/ws';

const server = createServer(app);
const wsServer = new WebSocketServer({
  server,
  path: '/graphql',
});

useServer(
  {
    schema,
    context: async (ctx) => {
      // 从连接参数获取认证信息
      const token = ctx.connectionParams?.authorization;
      return await createContextFromToken(token);
    },
  },
  wsServer
);
```

### 联邦式 GraphQL（Apollo Federation）

Federation 允许将多个 GraphQL 服务组合成一个统一的 API：

```typescript
// 服务 A：Users 服务
// schema.graphql
type User @key(fields: "id") {
  id: ID!
  name: String!
  email: String!
}

extend type Query {
  me: User
  user(id: ID!): User
}

// resolvers.ts
const resolvers = {
  Query: {
    me: (_, __, context) => context.user,
    user: (_, { id }, context) => context.prisma.user.findUnique({ where: { id } }),
  },
};

// 服务 B：Products 服务（扩展 User 类型）
// schema.graphql
extend type User @key(fields: "id") {
  id: ID! @external
  purchasedProducts: [Product!]!
  recommendedProducts: [Product!]!
}

type Product {
  id: ID!
  name: String!
  price: Float!
}

// resolvers.ts
const resolvers = {
  User: {
    purchasedProducts: (user) => {
      return context.prisma.order.findMany({
        where: { userId: user.id },
        include: { items: true },
      }).then(orders => orders.flatMap(o => o.items));
    },
    recommendedProducts: (user) => {
      return context.getRecommendations(user.id);
    },
  },
};

// 网关配置
// gateway.js
import { ApolloGateway } from '@apollo/gateway';
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';

const gateway = new ApolloGateway({
  serviceList: [
    { name: 'users', url: 'http://localhost:4001/graphql' },
    { name: 'products', url: 'http://localhost:4002/graphql' },
  ],
});

const server = new ApolloServer({ gateway });
const { url } = await startStandaloneServer(server);
```

### GraphQL 安全最佳实践

```typescript
// 1. 查询复杂度限制
import { createComplexityLimitRule } from 'graphql-query-complexity';

const complexityRule = createComplexityLimitRule(1000, {
  onCost: (cost) => console.log(`Query cost: ${cost}`),
  formatErrorMessage: (cost) => `Query complexity ${cost} exceeds maximum of 1000`,
});

// 2. 深度限制
import { createDepthLimitRule } from 'graphql-depth-limit';

const depthRule = createDepthLimitRule(10, {
  ignore: ['_service', '_entities'],
  formatErrorMessage: (depth) => `Query depth ${depth} exceeds maximum of 10`,
});

// 3. 速率限制
const rateLimitRule = createRateLimitRule({
  identifyContext: (ctx) => ctx.id || ctx.ip,
  rateLimit: {
    window: '10s',
    max: 100,
  },
});

// 4. 输入验证
import { validate } from 'graphql';
import { makeExecutableSchema } from '@graphql-tools/schema';

const validatingMiddleware = (req, res, next) => {
  if (req.body.query) {
    const errors = validate(schema, parse(req.body.query));
    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }
  }
  next();
};

// 5. 认证中间件
const authMiddleware = async (resolve, root, args, context, info) => {
  // 公开查询不需要认证
  const fieldName = info.fieldName;
  const publicFields = ['me', 'post', 'posts'];
  
  if (!publicFields.includes(fieldName)) {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }
    
    // 检查管理员权限
    if (fieldName.startsWith('admin') && context.user.role !== 'ADMIN') {
      throw new ForbiddenError('Not authorized');
    }
  }
  
  return resolve(root, args, context, info);
};
```

### GraphQL 缓存策略

```typescript
// 1. 字段级缓存（DataLoader）
// 已在前面介绍

// 2. Redis 缓存
import { createHash } from 'crypto';

const cacheMiddleware = async (resolve, root, args, context, info) => {
  // 只缓存查询，不缓存变更
  if (info.operation.operation !== 'query') {
    return resolve(root, args, context, info);
  }
  
  // 生成缓存键
  const cacheKey = generateCacheKey(info, args);
  
  // 尝试从缓存获取
  const cached = await context.redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // 执行查询
  const result = await resolve(root, args, context, info);
  
  // 缓存结果
  const ttl = getTTL(info);
  if (ttl > 0) {
    await context.redis.setex(cacheKey, ttl, JSON.stringify(result));
  }
  
  return result;
};

const generateCacheKey = (info, args): string => {
  const hash = createHash('sha256');
  hash.update(JSON.stringify({
    fieldName: info.fieldName,
    operationName: info.operation.name?.value,
    args,
    selectionSet: extractSelections(info.fieldNodes[0]),
  }));
  return `graphql:${hash.digest('hex')}`;
};

const extractSelections = (node): string[] => {
  if (!node.selectionSet) return [];
  return node.selectionSet.selections
    .map((sel) => {
      if (sel.kind === 'Field') return sel.name.value;
      if (sel.kind === 'FragmentSpread') return `...${sel.name.value}`;
      if (sel.kind === 'InlineFragment') return `...${extractSelections(sel)}`;
      return '';
    })
    .filter(Boolean);
};
```

### GraphQL 与 AI 集成的特别优势

```typescript
// 1. 结构化输出的天然适配
const resolvers = {
  Mutation: {
    generateDescription: async (
      _: any,
      { productId }: { productId: string },
      { prisma, openai }: Context
    ) => {
      const product = await prisma.product.findUnique({
        where: { id: productId },
        include: { category: true },
      });
      
      // 调用 AI 生成描述
      const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [
          {
            role: 'system',
            content: '你是一个专业的产品描述生成器。',
          },
          {
            role: 'user',
            content: `为以下产品生成简短描述：
名称：${product.name}
分类：${product.category.name}
特点：${product.features.join(', ')}`,
          },
        ],
        response_format: { type: 'json_object' },
        schema: {
          type: 'object',
          properties: {
            headline: { type: 'string' },
            description: { type: 'string' },
            tags: { type: 'array', items: { type: 'string' } },
          },
          required: ['headline', 'description'],
        },
      });
      
      return {
        productId,
        ...JSON.parse(response.choices[0].message.content),
      };
    },
  },
};

// 2. AI 代理的工具定义
const toolDefinitions = [
  {
    name: 'search_products',
    description: '搜索产品列表',
    inputSchema: {
      type: 'object',
      properties: {
        query: { type: 'string', description: '搜索关键词' },
        category: { type: 'string', description: '产品分类' },
        limit: { type: 'integer', description: '返回数量' },
      },
    },
  },
  {
    name: 'get_product_details',
    description: '获取产品详情',
    inputSchema: {
      type: 'object',
      properties: {
        id: { type: 'string', description: '产品ID' },
      },
      required: ['id'],
    },
  },
];

// 在 GraphQL Schema 中暴露工具
type ToolDefinition {
  name: String!
  description: String!
  inputSchema: JSON!
}

type Query {
  availableTools: [ToolDefinition!]!
}

const resolvers = {
  Query: {
    availableTools: () => toolDefinitions,
  },
};
```

---

> [!TIP]
> GraphQL 的强大之处在于其灵活的数据获取能力，但滥用也会带来复杂性。建议在确定需要复杂数据关系时才引入 GraphQL，并始终关注性能优化。

---

## GraphQL 完整项目结构

### NestJS + GraphQL 项目

```
src/
├── app.module.ts
├── app.resolver.ts
├── app.service.ts
├── main.ts
├── schema.gql
│
├── common/
│   ├── decorators/
│   │   ├── current-user.decorator.ts
│   │   └── roles.decorator.ts
│   ├── guards/
│   │   ├── auth.guard.ts
│   │   └── roles.guard.ts
│   ├── interceptors/
│   │   ├── logging.interceptor.ts
│   │   └── transform.interceptor.ts
│   ├── filters/
│   │   └── http-exception.filter.ts
│   └── pipes/
│       └── validation.pipe.ts
│
├── users/
│   ├── user.entity.ts
│   ├── user.model.ts
│   ├── users.module.ts
│   ├── users.resolver.ts
│   ├── users.service.ts
│   ├── dto/
│   │   ├── create-user.input.ts
│   │   ├── update-user.input.ts
│   │   └── user-where.input.ts
│   └── interfaces/
│       └── user-pagination.interface.ts
│
├── posts/
│   ├── post.entity.ts
│   ├── post.model.ts
│   ├── posts.module.ts
│   ├── posts.resolver.ts
│   └── posts.service.ts
│
└── auth/
    ├── auth.module.ts
    ├── auth.resolver.ts
    ├── auth.service.ts
    ├── strategies/
    │   └── jwt.strategy.ts
    └── guards/
        └── jwt-auth.guard.ts
```

### 完整Resolver实现

```typescript
// src/users/users.resolver.ts
import { Resolver, Query, Mutation, Args, Int, ResolveField, Parent, Context } from '@nestjs/graphql';
import { UseGuards, UseInterceptors } from '@nestjs/common';
import { User } from './user.model';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';
import { UsersService } from './users.service';
import { GqlAuthGuard } from '../auth/guards/gql-auth.guard';
import { CurrentUser } from '../common/decorators/current-user.decorator';
import { Roles } from '../common/decorators/roles.decorator';
import { RolesGuard } from '../common/guards/roles.guard';
import { LoggingInterceptor } from '../common/interceptors/logging.interceptor';
import { PaginationArgs } from '../common/dto/pagination.args';

@Resolver(() => User)
@UseInterceptors(LoggingInterceptor)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User], { name: 'users' })
  async getUsers(
    @Args() paginationArgs: PaginationArgs,
    @Args('search', { nullable: true }) search?: string,
    @Args('role', { type: () => String, nullable: true }) role?: string,
  ): Promise<User[]> {
    return this.usersService.findAll({
      ...paginationArgs,
      search,
      role,
    });
  }

  @Query(() => User, { name: 'user', nullable: true })
  async getUser(
    @Args('id', { type: () => Int }) id: number,
  ): Promise<User | null> {
    return this.usersService.findById(id);
  }

  @Query(() => User, { name: 'me', nullable: true })
  @UseGuards(GqlAuthGuard)
  async getMe(@CurrentUser() user: User): Promise<User> {
    return user;
  }

  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async createUser(
    @Args('input') createUserInput: CreateUserInput,
  ): Promise<User> {
    return this.usersService.create(createUserInput);
  }

  @Mutation(() => User)
  @UseGuards(GqlAuthGuard)
  async updateUser(
    @Args('id', { type: () => Int }) id: number,
    @Args('input') updateUserInput: UpdateUserInput,
    @CurrentUser() currentUser: User,
  ): Promise<User> {
    // 只能修改自己的信息，除非是管理员
    if (id !== currentUser.id && currentUser.role !== 'ADMIN') {
      throw new ForbiddenException('You can only update your own profile');
    }
    return this.usersService.update(id, updateUserInput);
  }

  @Mutation(() => Boolean)
  @UseGuards(GqlAuthGuard)
  async deleteUser(
    @Args('id', { type: () => Int }) id: number,
    @CurrentUser() currentUser: User,
  ): Promise<boolean> {
    if (id !== currentUser.id && currentUser.role !== 'ADMIN') {
      throw new ForbiddenException('You can only delete your own account');
    }
    return this.usersService.delete(id);
  }

  @ResolveField(() => Int)
  async postCount(@Parent() user: User): Promise<number> {
    return this.usersService.getPostCount(user.id);
  }
}
```

### 服务层实现

```typescript
// src/users/users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { PrismaService } from '../prisma/prisma.service';
import { CreateUserInput } from './dto/create-user.input';
import { UpdateUserInput } from './dto/update-user.input';
import * as bcrypt from 'bcrypt';

interface FindAllArgs {
  skip?: number;
  take?: number;
  search?: string;
  role?: string;
}

@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll({ skip = 0, take = 20, search, role }: FindAllArgs) {
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

    return this.prisma.user.findMany({
      where,
      skip,
      take,
      orderBy: { createdAt: 'desc' },
      include: {
        posts: {
          take: 5,
          orderBy: { createdAt: 'desc' },
        },
      },
    });
  }

  async findById(id: number): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        posts: true,
        _count: {
          select: { posts: true, comments: true },
        },
      },
    });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { email },
    });
  }

  async create(input: CreateUserInput): Promise<User> {
    // 检查邮箱唯一性
    const existing = await this.findByEmail(input.email);
    if (existing) {
      throw new ConflictException('Email already exists');
    }

    // 密码加密
    const hashedPassword = await bcrypt.hash(input.password, 10);

    return this.prisma.user.create({
      data: {
        ...input,
        password: hashedPassword,
      },
    });
  }

  async update(id: number, input: UpdateUserInput): Promise<User> {
    const user = await this.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    // 如果更新邮箱，检查唯一性
    if (input.email && input.email !== user.email) {
      const existing = await this.findByEmail(input.email);
      if (existing) {
        throw new ConflictException('Email already exists');
      }
    }

    return this.prisma.user.update({
      where: { id },
      data: input,
    });
  }

  async delete(id: number): Promise<boolean> {
    const user = await this.findById(id);
    if (!user) {
      throw new NotFoundException('User not found');
    }

    await this.prisma.user.delete({ where: { id } });
    return true;
  }

  async getPostCount(userId: number): Promise<number> {
    return this.prisma.post.count({ where: { authorId: userId } });
  }

  async validatePassword(email: string, password: string): Promise<User | null> {
    const user = await this.findByEmail(email);
    if (!user) return null;

    const isValid = await bcrypt.compare(password, user.password);
    return isValid ? user : null;
  }
}
```

---

## GraphQL 认证与授权

### JWT 认证实现

```typescript
// src/auth/auth.service.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { UsersService } from '../users/users.service';
import { LoginInput } from './dto/login.input';
import { RegisterInput } from './dto/register.input';
import { AuthResponse } from './models/auth-response.model';
import { User } from '../users/user.model';

@Injectable()
export class AuthService {
  constructor(
    private readonly usersService: UsersService,
    private readonly jwtService: JwtService,
  ) {}

  async login(loginInput: LoginInput): Promise<AuthResponse> {
    const user = await this.usersService.validatePassword(
      loginInput.email,
      loginInput.password,
    );

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    return this.generateAuthResponse(user);
  }

  async register(registerInput: RegisterInput): Promise<AuthResponse> {
    const user = await this.usersService.create({
      email: registerInput.email,
      password: registerInput.password,
      name: registerInput.name,
    });

    return this.generateAuthResponse(user);
  }

  async validateToken(token: string): Promise<User> {
    try {
      const payload = this.jwtService.verify(token);
      const user = await this.usersService.findById(payload.sub);
      if (!user) {
        throw new UnauthorizedException('User not found');
      }
      return user;
    } catch (error) {
      throw new UnauthorizedException('Invalid token');
    }
  }

  private generateAuthResponse(user: User): AuthResponse {
    const payload = { sub: user.id, email: user.email, role: user.role };
    
    return {
      accessToken: this.jwtService.sign(payload),
      refreshToken: this.jwtService.sign(payload, { expiresIn: '7d' }),
      user,
    };
  }

  async refreshToken(refreshToken: string): Promise<AuthResponse> {
    try {
      const payload = this.jwtService.verify(refreshToken);
      const user = await this.usersService.findById(payload.sub);
      
      if (!user) {
        throw new UnauthorizedException('Invalid refresh token');
      }

      return this.generateAuthResponse(user);
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }
}
```

### 权限守卫

```typescript
// src/common/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { GqlExecutionContext } from '@nestjs/graphql';
import { User } from '../../users/user.model';
import { ROLES_KEY } from '../decorators/roles.decorator';
import { Role } from '../enums/role.enum';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredRoles) {
      return true;
    }

    const ctx = GqlExecutionContext.create(context);
    const user: User = ctx.getContext().req.user;

    return requiredRoles.some((role) => user.role === role);
  }
}

// src/common/decorators/roles.decorator.ts
import { SetMetadata } from '@nestjs/common';
import { Role } from '../enums/role.enum';

export const ROLES_KEY = 'roles';
export const Roles = (...roles: Role[]) => SetMetadata(ROLES_KEY, roles);

// 使用示例
@Mutation(() => User)
@UseGuards(GqlAuthGuard, RolesGuard)
@Roles(Role.ADMIN)
async deleteUser(@Args('id') id: number): Promise<boolean> {
  return this.usersService.delete(id);
}
```

---

## GraphQL 高级查询模式

### 聚合查询

```typescript
// posts.service.ts
async getPostStats(): Promise<PostStats> {
  const [total, published, drafts, totalViews] = await Promise.all([
    this.prisma.post.count(),
    this.prisma.post.count({ where: { status: 'PUBLISHED' } }),
    this.prisma.post.count({ where: { status: 'DRAFT' } }),
    this.prisma.post.aggregate({
      _sum: { viewCount: true },
    }),
  ]);

  return {
    total,
    published,
    drafts,
    totalViews: totalViews._sum.viewCount || 0,
  };
}

async getTopAuthors(limit: number = 10): Promise<TopAuthor[]> {
  const authors = await this.prisma.user.findMany({
    where: {
      posts: { some: { status: 'PUBLISHED' } },
    },
    include: {
      posts: {
        where: { status: 'PUBLISHED' },
        select: { viewCount: true },
      },
      _count: {
        select: { posts: { where: { status: 'PUBLISHED' } } },
      },
    },
  });

  return authors
    .map((author) => ({
      id: author.id,
      name: author.name,
      postCount: author._count.posts,
      totalViews: author.posts.reduce((sum, post) => sum + (post.viewCount || 0), 0),
    }))
    .sort((a, b) => b.totalViews - a.totalViews)
    .slice(0, limit);
}

async getMonthlyStats(year: number): Promise<MonthlyStat[]> {
  const stats = await this.prisma.$queryRaw<MonthlyStat[]>`
    SELECT 
      DATE_TRUNC('month', "createdAt") as month,
      COUNT(*)::int as postCount,
      SUM("viewCount")::bigint as totalViews
    FROM "Post"
    WHERE EXTRACT(YEAR FROM "createdAt") = ${year}
    GROUP BY DATE_TRUNC('month', "createdAt")
    ORDER BY month
  `;

  return stats;
}
```

### 全文搜索

```typescript
// posts.resolver.ts
@Query(() => SearchResult)
async search(
  @Args('query') query: string,
  @Args('type', { type: () => SearchType, nullable: true }) type?: SearchType,
  @Args('limit', { type: () => Int, nullable: true }) limit?: number,
): Promise<SearchResult> {
  const results = await this.postsService.search(query, type, limit);
  return { results, query, total: results.length };
}

// posts.service.ts
async search(query: string, type?: SearchType, limit = 20): Promise<SearchResultItem[]> {
  const searchConditions: any = {
    OR: [
      { title: { contains: query, mode: 'insensitive' } },
      { content: { contains: query, mode: 'insensitive' } },
      { excerpt: { contains: query, mode: 'insensitive' } },
    ],
    status: 'PUBLISHED',
  };

  if (type === 'POSTS') {
    return this.prisma.post.findMany({
      where: searchConditions,
      take: limit,
      orderBy: { viewCount: 'desc' },
      select: {
        id: true,
        title: true,
        excerpt: true,
        viewCount: true,
        createdAt: true,
        author: { select: { id: true, name: true } },
      },
    });
  }

  if (type === 'USERS') {
    const users = await this.prisma.user.findMany({
      where: {
        OR: [
          { name: { contains: query, mode: 'insensitive' } },
          { email: { contains: query, mode: 'insensitive' } },
        ],
      },
      take: limit,
      select: {
        id: true,
        name: true,
        email: true,
        _count: { select: { posts: true } },
      },
    });
    return users.map(u => ({ ...u, type: 'USER' }));
  }

  // 搜索所有
  const [posts, users] = await Promise.all([
    this.prisma.post.findMany({
      where: searchConditions,
      take: Math.ceil(limit / 2),
      select: {
        id: true,
        title: true,
        excerpt: true,
        type: 'POST' as const,
      },
    }),
    this.prisma.user.findMany({
      where: {
        OR: [
          { name: { contains: query, mode: 'insensitive' } },
          { email: { contains: query, mode: 'insensitive' } },
        ],
      },
      take: Math.floor(limit / 2),
      select: {
        id: true,
        name: true,
        type: 'USER' as const,
      },
    }),
  ]);

  return [...posts, ...users];
}
```

---

## GraphQL 实战：电商系统

### 完整 Schema

```graphql
# 电商系统 Schema
scalar DateTime

enum OrderStatus {
  PENDING
  CONFIRMED
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
}

type Query {
  # 产品
  products(
    category: String
    minPrice: Float
    maxPrice: Float
    search: String
    pagination: PaginationInput
  ): ProductConnection!
  product(id: ID!): Product
  categories: [Category!]!

  # 订单
  order(id: ID!): Order
  orders(pagination: PaginationInput): OrderConnection!
  myOrders(pagination: PaginationInput): OrderConnection!

  # 用户
  me: User
  user(id: ID!): User

  # 推荐
  recommendedProducts(limit: Int = 10): [Product!]!
  similarProducts(productId: ID!, limit: Int = 5): [Product!]!
}

type Mutation {
  # 产品
  createProduct(input: CreateProductInput!): Product!
  updateProduct(id: ID!, input: UpdateProductInput!): Product!
  deleteProduct(id: ID!): Boolean!

  # 购物车
  addToCart(productId: ID!, quantity: Int!): Cart!
  updateCartItem(productId: ID!, quantity: Int!): Cart!
  removeFromCart(productId: ID!): Cart!
  clearCart: Cart!

  # 订单
  createOrder(input: CreateOrderInput!): Order!
  cancelOrder(id: ID!): Order!
  updateOrderStatus(id: ID!, status: OrderStatus!): Order!

  # 支付
  initiatePayment(orderId: ID!): PaymentIntent!
  confirmPayment(paymentIntentId: ID!): Payment!
}

type Product {
  id: ID!
  name: String!
  slug: String!
  description: String
  price: Float!
  compareAtPrice: Float
  images: [String!]!
  inventory: Int!
  category: Category!
  tags: [Tag!]!
  reviews: [Review!]!
  reviewCount: Int!
  averageRating: Float!
  seller: User!
  isFeatured: Boolean!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type Category {
  id: ID!
  name: String!
  slug: String!
  description: String
  image: String
  products(pagination: PaginationInput): ProductConnection!
  productCount: Int!
}

type Order {
  id: ID!
  orderNumber: String!
  status: OrderStatus!
  items: [OrderItem!]!
  subtotal: Float!
  tax: Float!
  shipping: Float!
  total: Float!
  shippingAddress: Address!
  billingAddress: Address!
  payment: Payment
  user: User!
  notes: String
  createdAt: DateTime!
  updatedAt: DateTime!
}

type OrderItem {
  id: ID!
  product: Product!
  quantity: Int!
  unitPrice: Float!
  total: Float!
}

type Cart {
  id: ID!
  user: User
  sessionId: String
  items: [CartItem!]!
  itemCount: Int!
  subtotal: Float!
  createdAt: DateTime!
  updatedAt: DateTime!
}

type CartItem {
  id: ID!
  product: Product!
  quantity: Int!
  unitPrice: Float!
  total: Float!
}

type Payment {
  id: ID!
  amount: Float!
  currency: String!
  status: PaymentStatus!
  paymentIntentId: String
  paidAt: DateTime
  refundedAt: DateTime
}

type PaymentIntent {
  clientSecret: String!
  paymentIntentId: String!
}

type Review {
  id: ID!
  rating: Int!
  title: String
  content: String
  images: [String!]
  user: User!
  product: Product!
  createdAt: DateTime!
}

type ProductConnection {
  edges: [ProductEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type ProductEdge {
  node: Product!
  cursor: String!
}

input PaginationInput {
  first: Int
  after: String
  last: Int
  before: String
}

input CreateProductInput {
  name: String!
  description: String
  price: Float!
  compareAtPrice: Float
  categoryId: ID!
  tagIds: [ID!]
  images: [String!]
  inventory: Int!
}

input CreateOrderInput {
  items: [OrderItemInput!]!
  shippingAddress: AddressInput!
  billingAddress: AddressInput
  notes: String
}

input OrderItemInput {
  productId: ID!
  quantity: Int!
}

input AddressInput {
  street: String!
  city: String!
  state: String!
  postalCode: String!
  country: String!
  phone: String
}
```

---

## GraphQL 监控与可观测性

### OpenTelemetry 集成

```typescript
// tracing.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { BatchSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { Resource } from '@opentelemetry/resources';
import { SemanticResourceAttributes } from '@opentelemetry/semantic-conventions';

const sdk = new NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'graphql-api',
    [SemanticResourceAttributes.SERVICE_VERSION]: '1.0.0',
  }),
  spanProcessor: new BatchSpanProcessor(
    new JaegerExporter({
      endpoint: 'http://jaeger:14268/api/traces',
    })
  ),
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-fs': { enabled: false },
    }),
  ],
});

sdk.start();

// GraphQL 追踪
import { GraphQLInstrumentation } from '@opentelemetry/instrumentation-graphql';

const graphqlInstrumentation = new GraphQLInstrumentation({
  // 不追踪变量内容（保护敏感数据）
  ignoreTrivialResolveSpans: true,
  // 自定义属性
  mergeItems: true,
});

graphqlInstrumentation.setTracerProvider(sdk.getTracerProvider());
```

### 自定义监控指标

```typescript
// metrics.ts
import { Registry, Counter, Histogram, Gauge } from 'prom-client';

const register = new Registry();

// 请求计数
const graphqlRequestsTotal = new Counter({
  name: 'graphql_requests_total',
  help: 'Total number of GraphQL requests',
  labelNames: ['operation', 'status', 'error_type'],
  registers: [register],
});

// 请求延迟
const graphqlRequestDuration = new Histogram({
  name: 'graphql_request_duration_seconds',
  help: 'GraphQL request duration in seconds',
  labelNames: ['operation'],
  buckets: [0.01, 0.05, 0.1, 0.5, 1, 2, 5],
  registers: [register],
});

// 字段解析计数
const fieldResolveTotal = new Counter({
  name: 'graphql_field_resolve_total',
  help: 'Total number of field resolutions',
  labelNames: ['type', 'field'],
  registers: [register],
});

// 数据库查询计数
const dbQueryTotal = new Counter({
  name: 'db_queries_total',
  help: 'Total number of database queries',
  labelNames: ['operation', 'table'],
  registers: [register],
});

// 活跃连接数
const activeConnections = new Gauge({
  name: 'graphql_active_connections',
  help: 'Number of active GraphQL connections',
  registers: [register],
});

// 监控中间件
const monitoringPlugin = {
  async requestDidStart({ request, operationName }) {
    const startTime = Date.now();
    
    return {
      async didResolveOperation({ operation }) {
        operation.operationName = operationName;
        operation.startTime = startTime;
      },
      
      async didEncounterErrors({ errors }) {
        const duration = Date.now() - startTime;
        const errorType = errors[0]?.originalError?.constructor.name || 'UnknownError';
        
        graphqlRequestsTotal.inc({
          operation: operationName,
          status: 'error',
          error_type: errorType,
        });
        
        graphqlRequestDuration.observe({ operation: operationName }, duration / 1000);
      },
      
      async willSendResponse({ response }) {
        const duration = Date.now() - startTime;
        const hasErrors = response.body.singleResult.errors?.length > 0;
        
        graphqlRequestsTotal.inc({
          operation: operationName,
          status: hasErrors ? 'error' : 'success',
          error_type: 'none',
        });
        
        graphqlRequestDuration.observe({ operation: operationName }, duration / 1000);
      },
    };
  },
};

// 指标端点
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});
```

---

> [!SUCCESS]
> GraphQL 作为一种现代化的 API 查询语言，在处理复杂数据关系和多样化客户端需求时展现出独特优势。本文档涵盖了从基础概念到高级实践的完整知识体系，包括 Schema 设计、Resolver 优化、认证授权、性能监控等关键主题，帮助开发者构建生产级别的 GraphQL API。
