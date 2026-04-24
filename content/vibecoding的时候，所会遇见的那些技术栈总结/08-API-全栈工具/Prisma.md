# Prisma：TypeScript ORM 的全面解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 Prisma ORM 的核心概念、Schema 语言、CRUD 操作、Migrate 工具、性能优化，以及与 Drizzle ORM 的对比分析。

---

## 目录

1. [[#概述与核心优势]]
2. [[#Prisma Schema 详解]]
3. [[#Prisma Client CRUD]]
4. [[#Prisma Migrate 版本管理]]
5. [[#Prisma Studio 可视化管理]]
6. [[#Prisma 与多数据库]]
7. [[#N+1 问题解决]]
8. [[#性能优化技巧]]
9. [[#Prisma 扩展机制]]
10. [[#与 Drizzle ORM 对比]]
11. [[#常见陷阱与排查]]
12. [[#实战场景]]
13. [[#选型建议]]

---

## 概述与核心优势

### 什么是 Prisma

Prisma 是现代 TypeScript 生态中最受欢迎的 ORM（对象关系映射）框架之一。与传统 ORM 不同，Prisma 采用**声明式 Schema** 定义数据模型，然后自动生成类型安全的数据库客户端，大幅提升开发效率和代码质量。对于正在学习全栈开发的工程师来说，Prisma 是一个非常好的起点，因为它把数据库操作变得非常直观，让你可以用操作 TypeScript 对象的方式来操作数据库。

Prisma 的核心理念：**让数据库操作像操作 TypeScript 对象一样简单，同时保持 SQL 的强大能力**。这意味着你不需要写复杂的 SQL 语句，Prisma 会帮你把链式 API 调用转换成高效的 SQL 查询。更重要的是，由于 Prisma 在编译时会生成完整的类型定义，你在对数据库进行任何操作时都能获得 IDE 的智能提示和类型检查。

### 为什么选择 Prisma

在 2026 年的今天，前端和全栈开发领域已经非常成熟，开发者有太多可以选择的技术栈。但 Prisma 之所以能在众多 ORM 中脱颖而出，主要有以下几个原因。首先，它的学习曲线非常平缓，即使是刚入门 TypeScript 的开发者，也能在几个小时内掌握 Prisma 的基本用法。其次，它的类型安全做得非常彻底，从 Schema 定义到最终的数据库操作，全程都有 TypeScript 类型保驾护航。再者，它的迁移工具（Prisma Migrate）让数据库版本管理变得非常简单，这在团队协作中尤为重要。

对于使用 Next.js、NestJS 或者其他 TypeScript 框架的开发者来说，Prisma 是目前最主流的选择。它的社区非常活跃，文档质量很高，而且在 GitHub 上的 star 数量已经超过了 35,000，这说明它已经被大量的生产项目采用。

### 技术架构

Prisma 由三个核心组件构成，每个组件都有其独特的作用：

| 组件 | 功能 | 说明 |
|------|------|------|
| **Prisma Schema** | 数据模型定义 | 声明式的 schema.prisma 文件，定义所有数据模型和关系 |
| **Prisma Client** | 类型安全客户端 | 自动生成的数据库操作 API，提供链式查询方法 |
| **Prisma Migrate** | 数据库迁移 | Schema 变更的版本管理，支持回滚和重置 |

```
┌─────────────────────────────────────────────────────┐
│                    Prisma Schema                    │
│  ┌─────────────────────────────────────────────┐   │
│  │  model User {                               │   │
│  │    id      Int      @id @default(autoincrement()) │
│  │    email   String   @unique                 │   │
│  │    name    String?                          │   │
│  │    posts   Post[]                           │   │
│  │  }                                          │   │
│  └─────────────────────────────────────────────┘   │
│                        ↓ generate                  │
│  ┌─────────────────────────────────────────────┐   │
│  │          TypeScript Client API                │   │
│  │  prisma.user.findMany()                      │   │
│  │  prisma.user.create({ data: {...} })        │   │
│  └─────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## Prisma Schema 详解

### 基本结构

Prisma Schema 文件（通常命名为 `schema.prisma`）是 Prisma 的核心文件。在这个小巧的文件里，你可以定义所有的数据模型、它们之间的关系、数据库连接信息，以及一些全局配置。让我来详细解释一下这个文件的基本结构。

首先，`generator client` 块告诉 Prisma 要生成什么样的客户端。这里的 `provider = "prisma-client-js"` 表示要生成 JavaScript/TypeScript 客户端，这是最常用的配置。如果你使用其他语言，可能需要配置其他 generator。

然后是 `datasource db` 块，这里定义了你要连接的数据库。`provider` 指定数据库类型（可以是 postgresql、mysql、sqlite 或 mongodb），`url` 则指定数据库的连接字符串，通常从环境变量中读取。

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"  // 生成 JavaScript/TypeScript 客户端
}

datasource db {
  provider = "postgresql"        // 数据库类型
  url      = env("DATABASE_URL") // 连接字符串
}

// 数据模型定义
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique                    // 唯一索引
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt                 // 自动更新
  
  // 关联关系
  posts     Post[]
  
  @@index([email])                               // 复合索引
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  categories Category[]                         // 多对多关系
  tags       Tag[]
}

model Category {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  posts Post[]
}
```

### 数据类型映射

Prisma 为每种支持的数据库都维护了一份数据类型映射表。以下是几种常见数据类型的映射关系：

| Prisma Type | PostgreSQL | MySQL | SQLite |
|-------------|------------|-------|--------|
| `String` | TEXT | VARCHAR(191) | TEXT |
| `Boolean` | BOOLEAN | BOOLEAN | INTEGER |
| `Int` | INTEGER | INT | INTEGER |
| `BigInt` | BIGINT | BIGINT | INTEGER |
| `Float` | DOUBLE PRECISION | DOUBLE | REAL |
| `Decimal` | DECIMAL | DECIMAL | TEXT |
| `DateTime` | TIMESTAMPTZ | DATETIME(3) | TEXT |
| `Json` | JSONB | JSON | TEXT |
| `Bytes` | BYTEA | BLOB | BLOB |
| `Enum` | ENUM | ENUM | TEXT |

这里需要特别注意几点：`Boolean` 在 SQLite 中实际上存储为 INTEGER（0 或 1）；`DateTime` 在不同数据库中的存储格式也不同，但 Prisma 会自动处理转换；`Decimal` 类型在 SQLite 中存储为 TEXT，以避免精度丢失问题。

### 字段修饰符详解

Prisma 提供了丰富的字段修饰符来定义字段的各种特性。理解这些修饰符对于正确设计数据模型至关重要。

`?` 修饰符表示可选字段，字段值可以是 null。`[]` 修饰符表示数组字段，可以存储多个值。`@default` 用于设置默认值，可以是常量值，也可以是 Prisma 内置函数如 `now()`、`cuid()`、`uuid()` 等。`@unique` 标记字段为唯一索引，保证值的唯一性。`@id` 标记主键字段。`@updatedAt` 是一个非常方便的修饰符，带有这个标记的字段会在记录更新时自动被设置为当前时间。

```prisma
model User {
  id        Int      @id @default(autoincrement())  // 自增主键
  email     String   @unique                       // 唯一约束
  name      String?                              // 可选字段
  age       Int      @default(0)                  // 默认值
  createdAt DateTime @default(now())              // 创建时间戳
  updatedAt DateTime @updatedAt                  // 更新时间戳
  roles     String[] @default([])               // 字符串数组
  metadata  Json?                               // JSON 字段
}
```

### 高级关系定义

Prisma 支持四种类型的关系：一对一、一对多、多对多，以及自引用关系。正确理解和使用这些关系是构建复杂数据模型的基础。

```prisma
model Author {
  id      Int     @id @default(autoincrement())
  name    String
  bio     String?
  
  // 一对多（隐式）
  books   Book[]
  
  // 一对一（显式）
  profile AuthorProfile?
}

model AuthorProfile {
  id        Int    @id @default(autoincrement())
  avatarUrl String
  twitter   String?
  
  authorId Int @unique  // 外键 + 唯一约束（一对一）
  author   Author @relation(fields: [authorId], references: [id])
}

model Book {
  id        Int      @id @default(autoincrement())
  title     String
  published Boolean  @default(false)
  
  // 多对多关系（隐式中间表）
  categories Category[]
  
  // 显式多对多关系
  translators TranslatorOnBook[]
}

model Translator {
  id    Int    @id @default(autoincrement())
  name  String
  
  books TranslatorOnBook[]
}

// 显式中间表（多对多）
model TranslatorOnBook {
  book         Book      @relation(fields: [bookId], references: [id])
  bookId       Int
  translator   Translator @relation(fields: [translatorId], references: [id])
  translatorId Int
  
  assignedAt DateTime @default(now())
  
  @@id([bookId, translatorId])  // 复合主键
}
```

---

## Prisma Client CRUD

### 安装与生成

首先，你需要安装 Prisma CLI 和 Prisma Client。CLI 是命令行工具，用于初始化项目、生成客户端、执行迁移等操作；Client 是运行时的数据库操作库。

```bash
# 安装 Prisma CLI 和 Client
npm install prisma --save-dev
npm install @prisma/client

# 初始化 Prisma（创建 schema.prisma）
npx prisma init

# 生成 Client（每次 Schema 变更后执行）
npx prisma generate

# 数据库推送（开发环境快速同步）
npx prisma db push
```

在开发环境中，`npx prisma db push` 是一个非常实用的命令。它会直接将 Schema 的变更同步到数据库，而不需要创建迁移文件。这对于快速迭代和原型开发非常方便。但要注意，生产环境一定要使用 `migrate deploy`，因为 db push 可能会导致数据丢失。

### 基础 CRUD 操作

Prisma Client 提供了直观易用的 CRUD API。所有的操作都通过 `PrismaClient` 实例来进行，而这个实例通常命名为 `prisma`。

**创建（Create）**

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

// 创建单条记录
const user = await prisma.user.create({
  data: {
    email: 'alice@example.com',
    name: 'Alice',
  },
})

// 批量创建
const users = await prisma.user.createMany({
  data: [
    { email: 'bob@example.com', name: 'Bob' },
    { email: 'carol@example.com', name: 'Carol' },
    { email: 'dave@example.com', name: 'Dave' },
  ],
  skipDuplicates: true,  // 跳过重复记录
})

// 创建并关联（嵌套写入）
const post = await prisma.post.create({
  data: {
    title: 'Hello World',
    content: 'My first post',
    author: {
      create: {
        email: 'newuser@example.com',
        name: 'New User',
      },
    },
  },
  include: {
    author: true,  // 返回关联的作者信息
  },
})
```

**读取（Read）**

```typescript
// 查询单条
const user = await prisma.user.findUnique({
  where: { id: 1 },
})

// 通过唯一字段查询
const userByEmail = await prisma.user.findUnique({
  where: { email: 'alice@example.com' },
})

// 查询多条
const users = await prisma.user.findMany({
  where: {
    // AND 条件组合
    AND: [
      { createdAt: { gte: new Date('2024-01-01') } },
      { posts: { some: { published: true } } },
    ],
    // OR 条件
    OR: [
      { name: { contains: 'Alice' } },
      { email: { endsWith: '@example.com' } },
    ],
    // NOT 条件
    NOT: {
      role: 'ADMIN',
    },
  },
  // 排序
  orderBy: {
    createdAt: 'desc',
  },
  // 分页
  skip: 0,
  take: 10,
  // 返回关联数据
  include: {
    posts: {
      where: { published: true },
      take: 5,
      orderBy: { createdAt: 'desc' },
    },
  },
  // 只返回特定字段（性能优化）
  select: {
    id: true,
    email: true,
    name: true,
    _count: {
      select: { posts: true },
    },
  },
})
```

**更新（Update）**

```typescript
// 更新单条
const updatedUser = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Alice Updated',
    email: 'alice.new@example.com',
  },
})

// 批量更新
const result = await prisma.user.updateMany({
  where: {
    createdAt: { lt: new Date('2024-01-01') },
  },
  data: {
    role: 'LEGACY',
  },
})

// 原子递增（避免竞态条件）
await prisma.user.update({
  where: { id: 1 },
  data: {
    postCount: { increment: 1 },
  },
})

// Upsert（存在则更新，不存在则创建）
const upsertUser = await prisma.user.upsert({
  where: { email: 'new@example.com' },
  update: { name: 'Updated Name' },
  create: { email: 'new@example.com', name: 'New Name' },
})
```

**删除（Delete）**

```typescript
// 删除单条
await prisma.user.delete({
  where: { id: 1 },
})

// 批量删除
await prisma.user.deleteMany({
  where: {
    createdAt: { lt: new Date('2023-01-01') },
    posts: { none: {} },  // 没有文章的用户
  },
})

// 级联删除配置（在 Schema 中）
model Post {
  id       Int @id @default(autoincrement())
  author   User @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId Int
}
```

### 聚合查询

Prisma 提供了丰富的聚合功能，包括计数、求和、平均值、最大最小值等：

```typescript
// 计数
const count = await prisma.user.count({
  where: { posts: { some: { published: true } } },
})

// 聚合统计
const stats = await prisma.user.aggregate({
  _count: { _all: true },
  _min: { createdAt: true },
  _max: { createdAt: true },
})

// 分组统计
const groupByResult = await prisma.post.groupBy({
  by: ['published'],
  _count: { id: true },
  where: {
    createdAt: { gte: new Date('2024-01-01') },
  },
})

// Raw 查询（需要时使用）
const rawResult = await prisma.$queryRaw`
  SELECT * FROM "User"
  WHERE created_at > ${new Date('2024-01-01')}
  ORDER BY created_at DESC
  LIMIT 10
`
```

---

## Prisma Migrate 版本管理

### Migrate 工作流程

Prisma Migrate 是 Prisma 提供的数据库迁移工具，它会跟踪 Schema 的变更历史，让你可以在不同版本之间来回切换。这在团队开发和生产环境部署中非常重要。

```
┌────────────────────────────────────────────────────┐
│                  Prisma Migrate                    │
├────────────────────────────────────────────────────┤
│                                                    │
│  Schema  ──→  SQL  ──→  Migration File  ──→  DB  │
│    ↓         ↓           ↓               ↓         │
│  定义变更   转换SQL    版本记录        执行变更    │
│                                                    │
│  ┌─────────────────────────────────────────────┐  │
│  │  migrations/                                │  │
│  │  ├── 20240101000000_init/                   │  │
│  │  │   └── migration.sql                      │  │
│  │  ├── 20240115000000_add_posts/              │  │
│  │  │   └── migration.sql                      │  │
│  │  └── 20240120000000_add_indexes/            │  │
│  │      └── migration.sql                      │  │
│  └─────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────┘
```

### 常用命令

```bash
# 创建迁移（开发环境）
npx prisma migrate dev --name add_user_role

# 部署迁移（生产环境）
npx prisma migrate deploy

# 重置迁移（开发环境，清空数据库）
npx prisma migrate reset

# 状态查看
npx prisma migrate status

# 解析（查看 SQL 但不执行）
npx prisma migrate resolve --rolled-back 20240101000000_init
```

> [!IMPORTANT]
> 生产环境使用 `migrate deploy` 而非 `migrate dev`，避免意外修改或回滚。

---

## Prisma Studio 可视化管理

### 启动 Studio

Prisma Studio 是一个内置的可视化数据库管理工具，对于查看和调试数据非常有用：

```bash
# 命令行启动
npx prisma studio

# 在浏览器中打开
# 默认地址：http://localhost:5555
```

### 功能特性

| 功能 | 说明 |
|------|------|
| **数据浏览** | 查看所有表的数据 |
| **数据编辑** | 直接修改数据 |
| **关系查看** | 可视化表关系 |
| **导出数据** | 导出 CSV/JSON |
| **导入数据** | 导入 CSV/JSON |

---

## Prisma 与多数据库

### PostgreSQL 配置

PostgreSQL 是 Prisma 推荐的生产环境数据库，它功能强大、性能优秀、支持丰富的数据类型。

```prisma
// schema.prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// DATABASE_URL 格式：
// postgresql://USER:PASSWORD@HOST:PORT/DATABASE?schema=SCHEMA&connection_limit=10&pool_timeout=10
```

PostgreSQL 支持的一些高级特性：

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  // PostgreSQL 特有的 JSONB 类型，适合存储半结构化数据
  metadata  Json?    
  // PostgreSQL 支持的数组类型
  tags      String[]
  // 枚举类型
  role      UserRole @default(USER)
  // 带时区的时间戳
  createdAt DateTime @default(now()) @db.Timestamptz
}

enum UserRole {
  USER
  ADMIN
  MODERATOR
}
```

### MySQL 配置

MySQL 是另一个流行的选择，特别适合共享主机环境和某些特定的托管场景：

```prisma
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

// DATABASE_URL 格式：
// mysql://USER:PASSWORD@HOST:PORT/DATABASE
```

MySQL 使用时的注意事项：

```prisma
model User {
  id        String   @id @default(cuid())  // MySQL 不支持自增的 BigInt 作为默认值
  email     String   @unique
  // 注意：MySQL 的 VARCHAR 有 191 字符限制（使用 utf8mb4 时）
  name      String   @db.VarChar(255)
  // MySQL 的 TEXT 类型映射
  bio       String?  @db.Text
}
```

### SQLite 配置

SQLite 是轻量级的嵌入式数据库，非常适合开发环境、原型项目和小型应用：

```prisma
datasource db {
  provider = "sqlite"
  url      = "file:./dev.db"
}
```

SQLite 的限制和注意事项：

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  // SQLite 的 Decimal 存储为 TEXT
  balance   Decimal  @db.Text
  // SQLite 不支持数组类型，需要用 JSON
  metadata  String?  // 存储 JSON 字符串
}
```

---

## N+1 问题解决

### 问题诊断

N+1 问题是最常见的数据库性能杀手。让我用一个具体的例子来说明这个问题：

```typescript
// 问题代码：N+1 查询
const users = await prisma.user.findMany()  // 1 次查询
for (const user of users) {
  const postCount = await prisma.post.count({
    where: { authorId: user.id },  // N 次额外查询！
  })
  console.log(`${user.name}: ${postCount} posts`)
}
// 总查询数：1 + N
```

### 解决方案

**使用 include + _count：**

```typescript
// 解决方案 1：使用 include
const usersWithPosts = await prisma.user.findMany({
  include: {
    _count: {
      select: { posts: true },
    },
  },
})
// 总查询数：1（自动 JOIN 或批量查询）

// 解决方案 2：使用 select
const usersWithCount = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    _count: {
      select: { posts: true },
    },
  },
})
```

**批量查询优化：**

```typescript
// 问题：循环中查询
const userIds = [1, 2, 3]
const results = []
for (const id of userIds) {
  const user = await prisma.user.findUnique({ where: { id } })  // 3 次查询
  results.push(user)
}

// 优化：使用 findMany
const users = await prisma.user.findMany({
  where: { id: { in: userIds } },  // 1 次查询
})
```

---

## 性能优化技巧

### 字段选择

使用 `select` 而非 `include` 可以显著提升性能，因为它只查询你需要的字段：

```typescript
// 只获取需要的字段
const users = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
  },
})
```

### 分页处理

对于大量数据的查询，一定要使用分页：

```typescript
// Offset 分页
const posts = await prisma.post.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: { createdAt: 'desc' },
})

// 游标分页（更高效）
const posts = await prisma.post.findMany({
  cursor: { id: lastId },
  take: pageSize,
  orderBy: { id: 'desc' },
})
```

### 索引策略

合理使用索引可以大幅提升查询性能：

```prisma
model User {
  id     Int    @id @default(autoincrement())
  email  String @unique  // 自动创建唯一索引
  name   String
  role   String
  
  // 自定义索引
  @@index([role])              // 单字段索引
  @@index([role, createdAt])   // 复合索引
}
```

---

## Prisma 扩展机制

### 自定义扩展

Prisma 的扩展机制允许你为 Client 添加自定义方法和计算字段：

```typescript
// 创建自定义扩展
const extendedPrisma = prisma.$extends({
  model: {
    user: {
      async findByEmail(email: string) {
        return this.findUnique({ where: { email } });
      },
      
      async findOrCreate(data: { email: string; name: string }) {
        const existing = await this.findUnique({ 
          where: { email: data.email } 
        });
        
        if (existing) return existing;
        
        return this.create({ data });
      }
    }
  },
  
  result: {
    user: {
      fullName: {
        compute(user) {
          return `${user.name || ''}`.trim() || user.email;
        }
      }
    }
  },
});

// 使用扩展
const user = await extendedPrisma.user.findByEmail('alice@example.com');
const fullName = user.fullName; // 计算字段
```

---

## 与 Drizzle ORM 对比

### 选择 Prisma 的场景

- TypeScript 全栈项目
- 需要快速原型开发
- 喜欢声明式 Schema 语法
- 需要内置迁移工具
- 团队对 TypeScript 友好

### 选择 Drizzle 的场景

- 性能极端敏感的项目
- 需要零运行时开销
- 喜欢 SQL-like 的语法
- 对 Bundle 大小有要求
- 更熟悉 SQL 的开发者

### 功能对比

| 特性 | Prisma | Drizzle |
|------|--------|---------|
| **类型安全** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Schema 定义** | 声明式 DSL | TypeScript 对象 |
| **运行时开销** | 有 | 无 |
| **迁移工具** | 内置 | 内置 |
| **Bundle 体积** | ~200KB | ~80KB |
| **学习曲线** | 平缓 | 中等 |

---

## 常见陷阱与排查

### 连接池耗尽

```typescript
// 解决方案：使用全局单例
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma = globalForPrisma.prisma ?? new PrismaClient({
  log: ['error', 'warn'],
});

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}
```

### 时区问题

```typescript
// 确保时间戳正确处理
const user = await prisma.user.create({
  data: {
    createdAt: new Date(),  // 使用 Date 对象，自动处理时区
  },
})
```

### 事务死锁

```typescript
// 使用事务时注意操作顺序
await prisma.$transaction([
  prisma.order.update({ ... }),
  prisma.inventory.update({ ... }),
], {
  isolationLevel: 'Serializable',
})
```

---

## 实战场景

### 电商系统数据模型

```prisma
model Product {
  id            String    @id @default(cuid())
  name          String
  price         Decimal   @db.Decimal(10, 2)
  inventory     Int       @default(0)
  categoryId    String?
  category      Category? @relation(fields: [categoryId], references: [id])
  orderItems    OrderItem[]
  
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  @@index([categoryId])
}

model Order {
  id        String      @id @default(cuid())
  status    OrderStatus @default(PENDING)
  total     Decimal     @db.Decimal(10, 2)
  userId    String
  user      User        @relation(fields: [userId], references: [id])
  items     OrderItem[]
  
  createdAt DateTime    @default(now())
}

enum OrderStatus {
  PENDING
  PAID
  SHIPPED
  DELIVERED
  CANCELLED
}
```

---

## 选型建议

### 选择 Prisma 的理由

- TypeScript 全栈项目首选
- 需要类型安全的数据库操作
- 快速原型开发
- 喜欢声明式 Schema
- PostgreSQL/MySQL/SQLite 开发

### 不选择 Prisma 的场景

- 性能极端敏感（Drizzle 更优）
- 需要 MongoDB 专用优化
- 遗留 JavaScript 项目
- 对 Bundle 大小极其敏感

---

> [!SUCCESS]
> Prisma 作为 TypeScript 生态中最成熟的 ORM 框架，以其声明式 Schema、类型安全客户端和丰富的扩展生态，成为 vibecoding 工作流中的数据库操作首选。善用其特性和最佳实践，可以大幅提升开发效率和代码质量。
