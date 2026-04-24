# Prisma：TypeScript ORM 的全面解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 Prisma ORM 的核心概念、Schema 语言、CRUD 操作、Migrate 工具，以及与 TypeORM/Sequelize/Drizzle 的对比。

---

## 目录

1. [[#概述与核心优势]]
2. [[#Prisma Schema 详解]]
3. [[#Prisma Client CRUD]]
4. [[#Prisma Migrate 版本管理]]
5. [[#与主流 ORM 对比]]
6. [[#Prisma Studio 可视化管理]]
7. [[#Prisma Accelerate 边缘缓存]]
8. [[#N+1 问题解决]]
9. [[#实战场景]]
10. [[#选型建议]]

---

## 概述与核心优势

### 什么是 Prisma

Prisma 是现代 TypeScript 生态中最受欢迎的 ORM（对象关系映射）框架之一。与传统 ORM 不同，Prisma 采用**声明式 Schema** 定义数据模型，然后自动生成类型安全的数据库客户端，大幅提升开发效率和代码质量。

Prisma 的核心设计理念：**让数据库操作像操作 TypeScript 对象一样简单，同时保持 SQL 的强大能力**。

### 技术架构

Prisma 由三个核心组件构成：

| 组件 | 功能 | 说明 |
|------|------|------|
| **Prisma Schema** | 数据模型定义 | 声明式的 schema.prisma 文件 |
| **Prisma Client** | 类型安全客户端 | 自动生成的数据库操作 API |
| **Prisma Migrate** | 数据库迁移 | Schema 变更的版本管理 |

```
┌─────────────────────────────────────────────────────┐
│                    Prisma Schema                    │
│  ┌─────────────────────────────────────────────┐    │
│  │  model User {                                │    │
│  │    id      Int      @id @default(autoincrement())  │
│  │    email   String   @unique                  │    │
│  │    name    String?                          │    │
│  │    posts   Post[]                            │    │
│  │  }                                           │    │
│  └─────────────────────────────────────────────┘    │
│                        ↓ generate                   │
│  ┌─────────────────────────────────────────────┐    │
│  │          TypeScript Client API               │    │
│  │  prisma.user.findMany()                       │    │
│  │  prisma.user.create({ data: {...} })         │    │
│  └─────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────┘
```

### 核心优势

| 优势 | 说明 |
|------|------|
| **类型安全** | 端到端 TypeScript 类型，编译时错误检测 |
| **自动补全** | IDE 完整支持，智能提示 |
| **直观语法** | 链式 API，易读易写 |
| **迁移友好** | Migrate 支持版本控制和回滚 |
| **多数据库** | PostgreSQL、MySQL、SQLite、MongoDB |
| **性能优化** | 内置 N+1 检测和批量操作 |

---

## Prisma Schema 详解

### 基本结构

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

### 字段修饰符

| 修饰符 | 说明 | 示例 |
|-------|------|------|
| `?` | 可选字段 | `name String?` |
| `[]` | 数组字段 | `emails String[]` |
| `@default` | 默认值 | `@default(now())` |
| `@unique` | 唯一约束 | `email String @unique` |
| `@id` | 主键 | `id Int @id` |
| `@updatedAt` | 自动更新时间戳 | `updatedAt DateTime @updatedAt` |
| `@ignore` | 生成时忽略 | `tempData String @ignore` |

### 高级关系定义

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
  
  authorId Int @unique  // 外键 + 唯一约束
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

```bash
# 安装 Prisma CLI 和 Client
npm install prisma --save-dev
npm install @prisma/client

# 初始化 Prisma
npx prisma init

# 生成 Client（每次 Schema 变更后执行）
npx prisma generate

# 数据库推送（开发环境快速同步）
npx prisma db push
```

### 基础 CRUD 操作

#### 创建（Create）

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

// 创建并关联
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

#### 读取（Read）

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
    // 条件组合
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
  // 只返回特定字段
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

#### 更新（Update）

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

// 原子递增
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

#### 删除（Delete）

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

// Raw 查询（需要时）
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

```
┌────────────────────────────────────────────────────┐
│                  Prisma Migrate                     │
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
# 创建迁移（生产环境）
npx prisma migrate dev --name add_user_role

# 部署迁移（生产环境）
npx prisma migrate deploy

# 重置迁移（开发环境）
npx prisma migrate reset

# 状态查看
npx prisma migrate status

# 解析（查看 SQL 但不执行）
npx prisma migrate resolve --rolled-back 20240101000000_init
```

### 迁移文件示例

```sql
-- migrations/20240115000000_add_posts.sql

-- CreateTable
CREATE TABLE "Post" (
    "id" SERIAL NOT NULL,
    "title" VARCHAR(255) NOT NULL,
    "content" TEXT,
    "published" BOOLEAN NOT NULL DEFAULT false,
    "authorId" INTEGER NOT NULL,
    "createdAt" TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMPTZ NOT NULL DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT "Post_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "Post_authorId_key" ON "Post"("authorId");

-- AddForeignKey
ALTER TABLE "Post" ADD CONSTRAINT "Post_authorId_fkey" 
    FOREIGN KEY ("authorId") 
    REFERENCES "User"("id") 
    ON DELETE RESTRICT 
    ON UPDATE CASCADE;
```

> [!IMPORTANT]
> 生产环境使用 `migrate deploy` 而非 `migrate dev`，避免意外修改或回滚。

---

## 与主流 ORM 对比

### 功能矩阵

| 功能 | Prisma | TypeORM | Sequelize | Drizzle |
|------|--------|---------|-----------|---------|
| **TypeScript 支持** | ⭐⭐⭐⭐⭐ 原生 | ⭐⭐⭐⭐ 良好 | ⭐⭐⭐ 一般 | ⭐⭐⭐⭐⭐ 原生 |
| **Schema 定义** | ⭐⭐⭐⭐⭐ 声明式 | ⭐⭐⭐ 装饰器 | ⭐⭐⭐ 同步 | ⭐⭐⭐⭐⭐ SQL-like |
| **查询构建器** | ⭐⭐⭐⭐ 链式 | ⭐⭐⭐⭐ 链式 | ⭐⭐⭐ 链式 | ⭐⭐⭐⭐⭐ SQL-like |
| **迁移工具** | ⭐⭐⭐⭐⭐ 内置 | ⭐⭐⭐⭐ 需配置 | ⭐⭐⭐ 需配置 | ⭐⭐⭐⭐ 内置 |
| **性能** | ⭐⭐⭐⭐ 优秀 | ⭐⭐⭐ 良好 | ⭐⭐⭐ 良好 | ⭐⭐⭐⭐⭐ 最快 |
| **Bundle 体积** | ⭐⭐⭐ 中等 | ⭐⭐ 较大 | ⭐⭐ 较大 | ⭐⭐⭐⭐⭐ 最小 |
| **学习曲线** | ⭐⭐⭐⭐⭐ 平缓 | ⭐⭐⭐ 中等 | ⭐⭐⭐ 中等 | ⭐⭐⭐ 中等 |
| **运行时开销** | ⭐⭐⭐⭐ 低 | ⭐⭐⭐⭐ 低 | ⭐⭐⭐ 中等 | ⭐⭐⭐⭐⭐ 零 |

### 详细对比

#### Prisma vs TypeORM

| 维度 | Prisma | TypeORM |
|------|--------|---------|
| **适用场景** | 全栈 TypeScript | 大型企业应用 |
| **Schema 方式** | 独立 .prisma 文件 | TypeScript 装饰器 |
| **类型生成** | 自动完整类型 | 手动/装饰器 |
| **Query Builder** | 高级 API | Entity Manager |
| **Active Record** | ❌ | ✅ 支持 |
| **迁移方式** | Migrate | Migration Entity |

#### Prisma vs Drizzle

| 维度 | Prisma | Drizzle |
|------|--------|---------|
| **设计理念** | 抽象化、安全 | 接近 SQL、性能 |
| **Schema 定义** | 声明式 DSL | TypeScript 对象 |
| **运行时** | 有 | 无（零运行时）|
| **学习成本** | 低 | 中（需 SQL 基础）|
| **适合场景** | 快速开发 | 性能敏感 |

### 性能对比（基准测试）

| 操作 | Prisma | TypeORM | Drizzle |
|------|--------|---------|---------|
| **简单查询** | 1.2ms | 1.8ms | 0.9ms |
| **关联查询** | 3.5ms | 5.2ms | 2.8ms |
| **批量插入 1000 条** | 45ms | 68ms | 32ms |
| **Bundle Size** | ~200KB | ~350KB | ~80KB |

---

## Prisma Studio 可视化管理

### 启动 Studio

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

## Prisma Accelerate 边缘缓存

### 什么是 Prisma Accelerate

Prisma Accelerate 是 Prisma Cloud 的边缘缓存服务，提供：

| 功能 | 说明 |
|------|------|
| **边缘缓存** | 全球 50+ 边缘节点 |
| **连接池** | 管理数据库连接 |
| **查询分析** | Query Analytics |
| **HA 切换** | 高可用自动切换 |

### 配置示例

```typescript
// prisma/schema.prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["driverAdapters"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 使用 Prisma Accelerate
import { PrismaClient } from '@prisma/client'
import { PrismaAccelerate } from '@prisma/clients/accelerate'

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: 'https://aws-apse1.prisma-data.net/api/data?api_key=xxx',
    },
  },
})
```

---

## N+1 问题解决

### 问题诊断

```typescript
// 问题代码：N+1 查询
const users = await prisma.user.findMany()
for (const user of users) {
  const postCount = await prisma.post.count({
    where: { authorId: user.id },  // N 次额外查询！
  })
  console.log(`${user.name}: ${postCount} posts`)
}
// 总查询数：1 + N

// 解决方案 1：使用 include
const usersWithPosts = await prisma.user.findMany({
  include: {
    _count: {
      select: { posts: true },
    },
  },
})
// 总查询数：1

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

### 批量查询优化

```typescript
// 问题：循环中查询
const userIds = [1, 2, 3]
const results = []
for (const id of userIds) {
  const user = await prisma.user.findUnique({ where: { id } })
  results.push(user)
}

// 优化：使用 findMany
const users = await prisma.user.findMany({
  where: { id: { in: userIds } },
})
```

### 嵌套查询优化

```typescript
// 问题：嵌套 N+1
const posts = await prisma.post.findMany({
  include: { author: true },
})
for (const post of posts) {
  const categoryCount = await prisma.category.count({
    where: { posts: { some: { id: post.id } } },
  })
}

// 优化：完全展开关联
const optimizedPosts = await prisma.post.findMany({
  include: {
    author: {
      select: { id: true, name: true },
    },
    categories: {
      select: { id: true, name: true },
    },
    _count: {
      select: { categories: true },
    },
  },
})
```

---

## 实战场景

### 场景：用户认证系统

```typescript
// schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  password  String
  name      String?
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  
  sessions  Session[]
}

model Session {
  id        String   @id @default(cuid())
  userId    Int
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
  
  createdAt DateTime @default(now())
}

enum Role {
  USER
  ADMIN
}

// service.ts
export class UserService {
  constructor(private prisma: PrismaClient) {}
  
  async createUser(data: CreateUserInput) {
    // 密码加密
    const hashedPassword = await bcrypt.hash(data.password, 10)
    
    return this.prisma.user.create({
      data: {
        ...data,
        password: hashedPassword,
      },
    })
  }
  
  async findByEmail(email: string) {
    return this.prisma.user.findUnique({
      where: { email },
    })
  }
  
  async createSession(userId: number, expiresInDays = 7) {
    const session = await this.prisma.session.create({
      data: {
        userId,
        expiresAt: new Date(Date.now() + expiresInDays * 24 * 60 * 60 * 1000),
      },
    })
    return session
  }
  
  async validateSession(sessionId: string) {
    return this.prisma.session.findFirst({
      where: {
        id: sessionId,
        expiresAt: { gt: new Date() },
      },
      include: { user: true },
    })
  }
}
```

---

## 选型建议

### 选择 Prisma 的理由

- ✅ TypeScript 全栈项目
- ✅ 需要类型安全的数据库操作
- ✅ 快速原型开发
- ✅ 喜欢声明式 Schema
- ✅ 需要内置迁移工具
- ✅ PostgreSQL/MySQL/SQLite 开发

### 不选择 Prisma 的场景

- ❌ 性能极端敏感（Drizzle 更优）
- ❌ 需要 Active Record 模式（TypeORM）
- ❌ MongoDB 专用项目
- ❌ 遗留 JavaScript 项目

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://www.prisma.io/docs |
| Prisma Studio | https://www.prisma.io/studio |
| Prisma Accelerate | https://www.prisma.io/data-platform/accelerate |
| GitHub | https://github.com/prisma/prisma |

---

## Prisma 高级特性与实战技巧

### Prisma Raw 查询与 SQL 片段

```typescript
// 复杂查询使用 Raw SQL
const result = await prisma.$queryRaw`
  SELECT 
    u.id,
    u.name,
    COUNT(p.id) as post_count,
    AVG(LENGTH(p.content)) as avg_content_length
  FROM "User" u
  LEFT JOIN "Post" p ON u.id = p."authorId"
  WHERE u."createdAt" > ${new Date('2024-01-01')}
  GROUP BY u.id
  HAVING COUNT(p.id) > 5
  ORDER BY post_count DESC
  LIMIT 10
`;

// 带参数化查询
const user = await prisma.$queryRaw<User>`
  SELECT * FROM "User" WHERE id = ${userId}
`;

// 执行原始 SQL（谨慎使用）
const affected = await prisma.$executeRaw`
  UPDATE "User" 
  SET "lastLoginAt" = NOW() 
  WHERE id = ${userId}
`;
```

### Prisma 与数据库视图

```typescript
// 在 schema 中定义视图
model UserPostStats {
  userId    Int
  userName  String
  postCount Int
  viewCount BigInt
  
  @@ignore // 不在 Prisma Client 中生成 CRUD
}

// 使用扩展添加视图
prisma.$extends({
  model: {
    $userPostStats: {
      async findTopUsers(limit: number = 10) {
        return prisma.$queryRaw`
          SELECT 
            u.id as "userId",
            u.name as "userName",
            COUNT(p.id)::int as "postCount",
            COALESCE(SUM(p.view_count), 0)::bigint as "viewCount"
          FROM "User" u
          LEFT JOIN "Post" p ON u.id = p."authorId"
          GROUP BY u.id, u.name
          ORDER BY "postCount" DESC
          LIMIT ${limit}
        `;
      }
    }
  }
});
```

### Prisma 扩展（Extensions）

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
  
  query: {
    user: {
      async findMany({ model, args, query }) {
        // 添加默认排序
        args.orderBy = args.orderBy || { createdAt: 'desc' };
        return query(args);
      }
    }
  }
});

// 使用扩展
const user = await extendedPrisma.user.findByEmail('alice@example.com');
const fullName = user.fullName; // 计算字段
```

### Prisma 与数据库连接池

```typescript
// 生产环境配置
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
  log: [
    { emit: 'event', level: 'query' },
    { emit: 'event', level: 'error' },
    { emit: 'event', level: 'warn' },
  ],
});

// 监听查询事件
prisma.$on('query' as any, (e: any) => {
  console.log('Query:', e.query);
  console.log('Duration:', e.duration, 'ms');
});

// 连接池配置（PostgreSQL）
// DATABASE_URL=postgresql://user:pass@host:5432/db?schema=public&connection_limit=10&pool_timeout=10
```

### Prisma 与软删除

```typescript
// schema.prisma
model User {
  id        Int       @id @default(autoincrement())
  email     String    @unique
  deletedAt DateTime? // 软删除标记
  // ...
}

// 扩展添加软删除
const softDeletePrisma = prisma.$extends({
  model: {
    user: {
      async softDelete(id: number) {
        return this.update({
          where: { id },
          data: { deletedAt: new Date() },
        });
      },
      
      async restore(id: number) {
        return this.update({
          where: { id },
          data: { deletedAt: null },
        });
      },
      
      async findActive(findUniqueArgs: any) {
        const result = await this.findUnique(findUniqueArgs);
        if (result?.deletedAt) return null;
        return result;
      },
    },
  },
  
  query: {
    user: {
      async findMany({ model, args, query }) {
        // 自动过滤已删除记录
        args.where = { ...args.where, deletedAt: null };
        return query(args);
      },
    },
  },
});
```

### Prisma 与全文搜索

```typescript
// PostgreSQL 全文搜索
const searchResults = await prisma.$queryRaw`
  SELECT 
    id,
    title,
    ts_rank(search_vector, plainto_tsquery('english', ${searchQuery})) as rank
  FROM "Post"
  WHERE search_vector @@ plainto_tsquery('english', ${searchQuery})
  ORDER BY rank DESC
  LIMIT 10
`;

// MySQL 全文搜索
const mysqlSearch = await prisma.$queryRaw`
  SELECT id, title
  FROM "Post"
  WHERE MATCH(title, content) AGAINST(${searchQuery} IN NATURAL LANGUAGE MODE)
  LIMIT 10
`;
```

---

## Prisma 性能优化与实战技巧

### 查询性能分析

```typescript
// 1. 启用查询日志
const prisma = new PrismaClient({
  log: [
    { emit: 'event', level: 'query' },
    { emit: 'event', level: 'warn' },
    { emit: 'event', level: 'error' },
  ],
});

prisma.$on('query', (e) => {
  console.log('Query:', e.query);
  console.log('Duration:', e.duration, 'ms');
  console.log('Params:', e.params);
});

// 2. EXPLAIN ANALYZE 分析
const explain = await prisma.$queryRaw`
  EXPLAIN ANALYZE
  SELECT * FROM "User" u
  WHERE u.email LIKE '%.com'
`;

// 3. 连接池监控
// 在 DATABASE_URL 中配置
// postgresql://user:pass@host:5432/db?connection_limit=10&pool_timeout=10

// 4. 使用 explain 进行查询优化
async function analyzeQuery() {
  const query = prisma.user.findMany({
    where: { role: 'ADMIN' },
    include: { posts: true },
  });
  
  const result = await prisma.$queryRaw`
    EXPLAIN (FORMAT JSON)
    SELECT * FROM "User" WHERE role = 'ADMIN'
  `;
  
  console.log(JSON.stringify(result, null, 2));
}
```

### 批量操作最佳实践

```typescript
// 大批量插入优化
const BATCH_SIZE = 5000;

async function bulkInsertUsers(users: CreateUserInput[]) {
  const results = [];
  
  for (let i = 0; i < users.length; i += BATCH_SIZE) {
    const batch = users.slice(i, i + BATCH_SIZE);
    const inserted = await prisma.user.createMany({
      data: batch,
      skipDuplicates: true, // 跳过重复的
    });
    results.push(inserted);
  }
  
  return results;
}

// 事务中的批量操作
async function processOrders(orders: CreateOrderInput[]) {
  return prisma.$transaction(async (tx) => {
    const results = [];
    
    for (const order of orders) {
      const created = await tx.order.create({ data: order });
      results.push(created);
    }
    
    return results;
  }, {
    timeout: 30000,
    maxWait: 5000,
  });
}

// 使用 $executeRaw 进行高效更新
async function batchUpdateStatus() {
  await prisma.$executeRaw`
    UPDATE "Order"
    SET status = 'SHIPPED', "updatedAt" = NOW()
    WHERE status = 'PAID' AND "shippingDate" < NOW() - INTERVAL '7 days'
  `;
}
```

### 复杂关联查询

```typescript
// 多层嵌套查询优化
const getUserDashboard = async (userId: string) => {
  // 分解为多个查询，使用 DataLoader 模式
  const [user, stats, recentPosts, topComments] = await Promise.all([
    prisma.user.findUnique({
      where: { id: userId },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
      },
    }),
    // 统计数据
    prisma.user.aggregate({
      where: { id: userId },
      _count: { posts: true, comments: true },
    }),
    // 最近文章
    prisma.post.findMany({
      where: { authorId: userId },
      orderBy: { createdAt: 'desc' },
      take: 5,
      select: { id: true, title: true, createdAt: true },
    }),
    // 高赞评论
    prisma.comment.findMany({
      where: { authorId: userId },
      orderBy: { likes: 'desc' },
      take: 10,
      select: {
        id: true,
        content: true,
        likes: true,
        post: { select: { title: true } },
      },
    }),
  ]);
  
  return { user, stats, recentPosts, topComments };
};

// 聚合查询
const getCategoryStats = async () => {
  return prisma.category.findMany({
    select: {
      name: true,
      _count: {
        select: { posts: true },
      },
      posts: {
        select: {
          viewCount: true,
        },
      },
    },
  }).then(categories => categories.map(cat => ({
    name: cat.name,
    postCount: cat._count.posts,
    totalViews: cat.posts.reduce((sum, p) => sum + p.viewCount, 0),
    avgViews: Math.round(
      cat.posts.reduce((sum, p) => sum + p.viewCount, 0) / cat._count.posts
    ),
  })));
};
```

### Prisma 与 Redis 缓存集成

```typescript
import { caching } from 'cache-manager';
import { RedisStore } from 'cache-manager-redis-store';

const redisStore = await caching({
  store: RedisStore,
  url: 'redis://localhost:6379',
});

const CACHE_TTL = 300; // 5分钟

// 缓存辅助函数
async function cachedQuery<T>(
  cacheKey: string,
  queryFn: () => Promise<T>
): Promise<T> {
  const cached = await redisStore.get<T>(cacheKey);
  if (cached) return cached;
  
  const result = await queryFn();
  await redisStore.set(cacheKey, result, { ttl: CACHE_TTL });
  
  return result;
}

// 使用示例
async function getPopularPosts(limit: number) {
  const cacheKey = `popular_posts:${limit}`;
  
  return cachedQuery(cacheKey, async () => {
    return prisma.post.findMany({
      where: { published: true },
      orderBy: { viewCount: 'desc' },
      take: limit,
      include: {
        author: { select: { name: true, avatar: true } },
      },
    });
  });
}

// 缓存失效
async function invalidatePostCache(postId: string) {
  const keys = await redisStore.store.keys('popular_posts:*');
  if (keys.length > 0) {
    await redisStore.store.del(keys);
  }
}
```

### Prisma 与多租户

```typescript
// 方案一：Schema 级别隔离
const createTenantPrisma = (tenantId: string) => {
  return new PrismaClient({
    datasources: {
      db: {
        url: `${process.env.DATABASE_URL}?schema=${tenantId}`,
      },
    },
  });
};

// 方案二：行级别隔离
const tenantMiddleware = prisma.$extends({
  query: {
    $allModels: {
      async findMany({ model, args, query }) {
        // 如果模型有 tenantId 字段，自动添加过滤
        const tenantContext = getTenantContext();
        if (tenantContext && modelHasTenantField(model)) {
          args.where = {
            ...args.where,
            tenantId: tenantContext.id,
          };
        }
        return query(args);
      },
      async findUnique({ model, args, query }) {
        const tenantContext = getTenantContext();
        if (tenantContext && modelHasTenantField(model)) {
          args.where = {
            ...args.where,
            tenantId: tenantContext.id,
          };
        }
        return query(args);
      },
    },
  },
});

// 使用中间件
async function createTenantData() {
  return tenantMiddleware.post.create({
    data: {
      title: 'Tenant Post',
      content: 'This post is automatically scoped',
    },
  });
}
```

### Prisma 部署与迁移策略

```typescript
// migration.ts - 迁移管理脚本
import { PrismaClient } from '@prisma/client';
import { migrate } from '@prisma/migrate';

async function runMigrations() {
  // 开发环境
  if (process.env.NODE_ENV === 'development') {
    await migrate.dev({
      name: 'dev-migration',
    });
  }
  
  // 生产环境 - 谨慎执行
  if (process.env.NODE_ENV === 'production') {
    await migrate.deploy();
  }
}

// 数据库种子
async function seed() {
  const prisma = new PrismaClient();
  
  try {
    // 清理现有数据
    await prisma.comment.deleteMany();
    await prisma.post.deleteMany();
    await prisma.user.deleteMany();
    
    // 创建用户
    const users = await Promise.all([
      prisma.user.create({
        data: {
          email: 'alice@example.com',
          name: 'Alice',
          posts: {
            create: [
              {
                title: 'First Post',
                content: 'Hello World',
                published: true,
              },
              {
                title: 'Second Post',
                content: 'More content',
                published: false,
              },
            ],
          },
        },
        include: { posts: true },
      }),
      prisma.user.create({
        data: {
          email: 'bob@example.com',
          name: 'Bob',
        },
      }),
    ]);
    
    // 创建评论
    await prisma.comment.create({
      data: {
        content: 'Great post!',
        postId: users[0].posts[0].id,
        authorId: users[1].id,
      },
    });
    
    console.log('Seed completed:', { users: users.length });
  } finally {
    await prisma.$disconnect();
  }
}

// 环境检查
async function healthCheck() {
  const prisma = new PrismaClient();
  
  try {
    await prisma.$connect();
    
    // 测试查询
    await prisma.$queryRaw`SELECT 1`;
    
    console.log('Database connection: OK');
    return true;
  } catch (error) {
    console.error('Database connection: FAILED', error);
    return false;
  } finally {
    await prisma.$disconnect();
  }
}
```

---

> [!TIP]
> Prisma 的声明式 Schema 和类型安全的 API 使数据库操作变得直观且可靠。善用扩展和中间件可以大幅简化常见模式的实现。

---

## Prisma 高级数据建模

### 完整电商 Schema

```prisma
// schema.prisma - 电商系统完整数据模型

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ==================== 用户模块 ====================

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  password      String
  name          String
  avatar        String?
  phone         String?
  role          UserRole  @default(USER)
  isActive      Boolean   @default(true)
  emailVerified Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  lastLoginAt   DateTime?

  // 关系
  posts           Post[]
  comments        Comment[]
  orders          Order[]
  reviews         Review[]
  cart           Cart?
  wishlist       WishlistItem[]
  addresses       Address[]
  paymentMethods  PaymentMethod[]
  
  // 关注关系
  following       Follow[]      @relation("Follower")
  followers       Follow[]      @relation("Following")
  
  // 会话
  sessions        Session[]
  
  // 通知
  notifications   Notification[]
  
  // 审计日志
  auditLogs       AuditLog[]

  @@index([email])
  @@index([role])
  @@index([createdAt])
}

enum UserRole {
  USER
  MODERATOR
  ADMIN
}

model Session {
  id           String   @id @default(cuid())
  userId       String
  user         User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  token        String   @unique
  expiresAt    DateTime
  ipAddress    String?
  userAgent    String?
  createdAt    DateTime @default(now())

  @@index([userId])
  @@index([token])
  @@index([expiresAt])
}

// ==================== 内容模块 ====================

model Post {
  id           String        @id @default(cuid())
  title        String
  slug         String        @unique
  excerpt      String?
  content      String        @db.Text
  coverImage   String?
  status       PostStatus   @default(DRAFT)
  viewCount    Int           @default(0)
  likeCount    Int           @default(0)
  commentCount Int           @default(0)
  publishedAt  DateTime?
  createdAt    DateTime      @default(now())
  updatedAt    DateTime      @updatedAt

  // 关系
  authorId     String
  author       User          @relation(fields: [authorId], references: [id])
  comments     Comment[]
  likes        Like[]
  tags         TagOnPost[]
  categoryId   String?
  category     Category?     @relation(fields: [categoryId], references: [id])

  @@index([authorId])
  @@index([status])
  @@index([publishedAt])
  @@index([slug])
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

model Comment {
  id        String    @id @default(cuid())
  content   String    @db.Text
  isEdited  Boolean   @default(false)
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  // 关系
  authorId  String
  author    User      @relation(fields: [authorId], references: [id])
  postId    String
  post      Post      @relation(fields: [postId], references: [id], onDelete: Cascade)
  parentId  String?
  parent    Comment?  @relation("CommentReplies", fields: [parentId], references: [id])
  replies   Comment[] @relation("CommentReplies")

  @@index([postId])
  @@index([authorId])
  @@index([parentId])
}

model Like {
  id        String   @id @default(cuid())
  userId    String
  postId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@unique([userId, postId])
  @@index([postId])
}

model Category {
  id          String   @id @default(cuid())
  name        String   @unique
  slug        String   @unique
  description String?
  image       String?
  sortOrder   Int      @default(0)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  posts       Post[]

  @@index([slug])
  @@index([sortOrder])
}

model Tag {
  id        String   @id @default(cuid())
  name      String   @unique
  slug      String   @unique
  createdAt DateTime @default(now())

  posts     TagOnPost[]

  @@index([slug])
}

model TagOnPost {
  postId    String
  tagId     String
  post      Post   @relation(fields: [postId], references: [id], onDelete: Cascade)
  tag       Tag    @relation(fields: [tagId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@id([postId, tagId])
}

// ==================== 电商模块 ====================

model Product {
  id            String          @id @default(cuid())
  name          String
  slug          String          @unique
  description   String?         @db.Text
  price         Decimal         @db.Decimal(10, 2)
  compareAtPrice Decimal?        @db.Decimal(10, 2)
  sku           String          @unique
  barcode       String?
  weight        Float?          // 克
  dimensions    Json?           // { length, width, height }
  images        String[]        // 多个图片
  inventory     Int             @default(0)
  trackInventory Boolean        @default(true)
  status        ProductStatus   @default(ACTIVE)
  isFeatured    Boolean         @default(false)
  isPublished   Boolean         @default(false)
  publishedAt   DateTime?
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt

  // 关系
  categoryId    String?
  category      Category?        @relation(fields: [categoryId], references: [id])
  brandId       String?
  brand         Brand?          @relation(fields: [brandId], references: [id])
  variants      ProductVariant[]
  attributes    ProductAttribute[]
  orderItems    OrderItem[]
  cartItems     CartItem[]
  reviews       Review[]
  wishlistItems WishlistItem[]

  @@index([categoryId])
  @@index([brandId])
  @@index([slug])
  @@index([status])
}

enum ProductStatus {
  DRAFT
  ACTIVE
  ARCHIVED
}

model ProductVariant {
  id         String   @id @default(cuid())
  productId  String
  product    Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  sku        String   @unique
  price      Decimal  @db.Decimal(10, 2)
  inventory  Int      @default(0)
  attributes Json     // { color: "red", size: "L" }
  isActive   Boolean  @default(true)
  createdAt  DateTime @default(now())
  updatedAt  DateTime @updatedAt

  @@index([productId])
  @@index([sku])
}

model ProductAttribute {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  name      String   // e.g., "Color", "Size"
  value     String   // e.g., "Red", "Large"
  sortOrder Int      @default(0)

  @@index([productId])
}

model Brand {
  id          String    @id @default(cuid())
  name        String    @unique
  slug        String    @unique
  logo        String?
  description String?
  website     String?
  isActive    Boolean   @default(true)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  products    Product[]

  @@index([slug])
}

// ==================== 订单模块 ====================

model Order {
  id              String      @id @default(cuid())
  orderNumber     String      @unique  // 例如: ORD-20240119-001
  status          OrderStatus @default(PENDING)
  paymentStatus   PaymentStatus @default(PENDING)
  subtotal        Decimal     @db.Decimal(10, 2)
  discount        Decimal     @db.Decimal(10, 2) @default(0)
  shippingCost    Decimal     @db.Decimal(10, 2) @default(0)
  tax             Decimal     @db.Decimal(10, 2) @default(0)
  total           Decimal     @db.Decimal(10, 2)
  currency        String      @default("USD")
  
  // 收货信息
  shippingAddress Json        // 收货地址快照
  billingAddress  Json?       // 账单地址（可选）
  
  notes           String?     @db.Text
  adminNotes     String?     @db.Text
  
  // 支付信息
  paymentMethod  String?
  paymentId      String?     // 第三方支付ID
  
  // 物流信息
  trackingNumber String?
  carrier        String?
  shippedAt      DateTime?
  deliveredAt    DateTime?
  
  createdAt      DateTime    @default(now())
  updatedAt      DateTime    @updatedAt
  completedAt    DateTime?
  
  // 关系
  userId         String
  user           User       @relation(fields: [userId], references: [id])
  items          OrderItem[]
  payment        Payment?

  @@index([userId])
  @@index([status])
  @@index([paymentStatus])
  @@index([createdAt])
  @@index([orderNumber])
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
  RETURNED
}

enum PaymentStatus {
  PENDING
  PAID
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

model OrderItem {
  id            String    @id @default(cuid())
  orderId       String
  order         Order     @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId     String
  product       Product   @relation(fields: [productId], references: [id])
  variantId     String?
  productSnapshot Json    // 产品信息快照（订单创建时的价格等信息）
  quantity      Int
  unitPrice     Decimal   @db.Decimal(10, 2)
  discount      Decimal   @db.Decimal(10, 2) @default(0)
  total         Decimal   @db.Decimal(10, 2)

  @@index([orderId])
  @@index([productId])
}

model Payment {
  id              String        @id @default(cuid())
  orderId         String        @unique
  order           Order         @relation(fields: [orderId], references: [id])
  amount          Decimal       @db.Decimal(10, 2)
  currency        String        @default("USD")
  status          PaymentStatus @default(PENDING)
  method          String?       // card, alipay, wechatpay
  provider        String?       // stripe, paypal
  providerPaymentId String?     // 第三方支付流水号
  metadata        Json?
  paidAt          DateTime?
  refundedAt     DateTime?
  refundAmount    Decimal       @db.Decimal(10, 2)?
  createdAt       DateTime     @default(now())
  updatedAt       DateTime     @updatedAt

  @@index([orderId])
  @@index([status])
}

// ==================== 购物车模块 ====================

model Cart {
  id          String     @id @default(cuid())
  userId      String     @unique
  user        User       @relation(fields: [userId], references: [id])
  items       CartItem[]
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  @@index([userId])
}

model CartItem {
  id        String   @id @default(cuid())
  cartId    String
  cart      Cart     @relation(fields: [cartId], references: [id], onDelete: Cascade)
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  variantId String?
  quantity  Int      @default(1)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([cartId, productId, variantId])
  @@index([cartId])
  @@index([productId])
}

// ==================== 收藏与评论 ====================

model WishlistItem {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  productId String
  product   Product   @relation(fields: [productId], references: [id], onDelete: Cascade)
  note      String?   // 用户的备注
  createdAt DateTime  @default(now())

  @@unique([userId, productId])
  @@index([userId])
}

model Review {
  id        String   @id @default(cuid())
  rating    Int      // 1-5 星
  title     String?
  content   String?  @db.Text
  images    String[] // 用户上传的图片
  isVerifiedPurchase Boolean @default(false)
  isApproved  Boolean @default(false)
  helpfulCount Int   @default(0)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // 关系
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  productId String
  product   Product  @relation(fields: [productId], references: [id])

  @@index([productId])
  @@index([userId])
  @@index([rating])
}

// ==================== 基础设施 ====================

model Address {
  id          String       @id @default(cuid())
  userId      String
  user        User         @relation(fields: [userId], references: [id], onDelete: Cascade)
  type        AddressType  @default(SHIPPING)
  name        String       // 收货人姓名
  phone       String       // 手机号
  province    String       // 省
  city        String       // 市
  district    String       // 区/县
  street      String       // 详细地址
  postalCode  String?      // 邮编
  isDefault   Boolean      @default(false)
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt

  @@index([userId])
  @@index([userId, type])
}

enum AddressType {
  SHIPPING
  BILLING
}

model PaymentMethod {
  id           String   @id @default(cuid())
  userId       String
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  type         String   // card, bank_account, digital_wallet
  provider     String   // stripe, paypal
  last4        String?  // 卡号后4位
  brand        String?  // visa, mastercard
  expiryMonth  Int?
  expiryYear   Int?
  isDefault    Boolean  @default(false)
  metadata     Json?
  createdAt    DateTime @default(now())

  @@index([userId])
}

model Follow {
  id          String   @id @default(cuid())
  followerId  String   // 关注者
  followingId String   // 被关注者
  follower    User    @relation("Follower", fields: [followerId], references: [id], onDelete: Cascade)
  following   User     @relation("Following", fields: [followingId], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())

  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}

model Notification {
  id        String           @id @default(cuid())
  userId    String
  user      User             @relation(fields: [userId], references: [id], onDelete: Cascade)
  type      NotificationType
  title     String
  content   String
  data      Json?            // 额外数据，如链接等
  isRead    Boolean          @default(false)
  readAt    DateTime?
  createdAt DateTime         @default(now())

  @@index([userId])
  @@index([userId, isRead])
  @@index([createdAt])
}

enum NotificationType {
  SYSTEM
  ORDER
  PAYMENT
  REVIEW
  FOLLOW
  LIKE
  COMMENT
  PROMO
}

model AuditLog {
  id         String   @id @default(cuid())
  userId     String?
  user       User?    @relation(fields: [userId], references: [id], onDelete: SetNull)
  action     String   // CREATE, UPDATE, DELETE
  entity     String   // User, Product, Order
  entityId   String
  changes    Json?    // 变更前后的数据
  ipAddress  String?
  userAgent  String?
  createdAt  DateTime @default(now())

  @@index([userId])
  @@index([entity, entityId])
  @@index([createdAt])
}
```

---

## Prisma 与业务逻辑分离

### Repository 模式实现

```typescript
// repositories/user.repository.ts
import { PrismaClient } from '@prisma/client';
import { User, Prisma } from '@prisma/generated/client';

export class UserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { id },
      include: {
        addresses: true,
        _count: {
          select: { posts: true, followers: true, following: true },
        },
      },
    });
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.prisma.user.findUnique({
      where: { email },
    });
  }

  async findAll(args: {
    skip?: number;
    take?: number;
    where?: Prisma.UserWhereInput;
    orderBy?: Prisma.UserOrderByWithRelationInput;
  }): Promise<{ users: User[]; total: number }> {
    const { skip = 0, take = 20, where = {}, orderBy = { createdAt: 'desc' } } = args;

    const [users, total] = await this.prisma.$transaction([
      this.prisma.user.findMany({
        where,
        skip,
        take,
        orderBy,
      }),
      this.prisma.user.count({ where }),
    ]);

    return { users, total };
  }

  async create(data: {
    email: string;
    password: string;
    name: string;
    role?: 'USER' | 'ADMIN' | 'MODERATOR';
  }): Promise<User> {
    return this.prisma.user.create({
      data: {
        ...data,
        password: await hashPassword(data.password),
      },
    });
  }

  async update(
    id: string,
    data: Prisma.UserUpdateInput
  ): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data,
    });
  }

  async delete(id: string): Promise<void> {
    await this.prisma.user.delete({
      where: { id },
    });
  }

  async softDelete(id: string): Promise<User> {
    return this.prisma.user.update({
      where: { id },
      data: {
        isActive: false,
        email: `deleted_${Date.now()}@deleted.com`,
      },
    });
  }

  async follow(followerId: string, followingId: string): Promise<void> {
    await this.prisma.follow.create({
      data: {
        followerId,
        followingId,
      },
    });
  }

  async unfollow(followerId: string, followingId: string): Promise<void> {
    await this.prisma.follow.delete({
      where: {
        followerId_followingId: {
          followerId,
          followingId,
        },
      },
    });
  }

  async getFollowers(userId: string, args?: { skip?: number; take?: number }) {
    return this.prisma.follow.findMany({
      where: { followingId: userId },
      include: {
        follower: {
          select: { id: true, name: true, avatar: true },
        },
      },
      skip: args?.skip,
      take: args?.take,
    });
  }
}
```

### Service 层实现

```typescript
// services/user.service.ts
import { UserRepository } from '../repositories/user.repository';
import { User } from '@prisma/client';
import { TRPCError } from '@trpc/server';
import { z } from 'zod';

const UpdateProfileSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  bio: z.string().max(500).optional(),
  avatar: z.string().url().optional(),
});

export class UserService {
  constructor(private userRepo: UserRepository) {}

  async getProfile(id: string): Promise<User> {
    const user = await this.userRepo.findById(id);
    
    if (!user) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'User not found',
      });
    }

    return user;
  }

  async updateProfile(
    userId: string,
    input: z.infer<typeof UpdateProfileSchema>
  ): Promise<User> {
    const validated = UpdateProfileSchema.parse(input);
    
    return this.userRepo.update(userId, validated);
  }

  async followUser(followerId: string, followingId: string): Promise<void> {
    if (followerId === followingId) {
      throw new TRPCError({
        code: 'BAD_REQUEST',
        message: 'Cannot follow yourself',
      });
    }

    const [follower, following] = await Promise.all([
      this.userRepo.findById(followerId),
      this.userRepo.findById(followingId),
    ]);

    if (!follower || !following) {
      throw new TRPCError({
        code: 'NOT_FOUND',
        message: 'User not found',
      });
    }

    await this.userRepo.follow(followerId, followingId);
  }

  async searchUsers(query: string, page: number = 1, limit: number = 20) {
    const { users, total } = await this.userRepo.findAll({
      where: {
        OR: [
          { name: { contains: query, mode: 'insensitive' } },
          { email: { contains: query, mode: 'insensitive' } },
        ],
        isActive: true,
      },
      skip: (page - 1) * limit,
      take: limit,
    });

    return {
      users,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }
}
```

---

## Prisma 性能监控与告警

```typescript
// lib/prisma-monitoring.ts
import { PrismaClient, Prisma } from '@prisma/client';
import { metrics } from '@opentelemetry/api';

const slowQueryThreshold = 1000; // 毫秒
const criticalQueryThreshold = 5000; // 毫秒

// 性能指标
const queryDuration = new metrics.Histogram('prisma_query_duration_ms', {
  description: 'Duration of Prisma queries in milliseconds',
  unit: 'ms',
});

const queryCount = new metrics.Counter('prisma_query_total', {
  description: 'Total number of Prisma queries',
});

const slowQueryCount = new metrics.Counter('prisma_slow_query_total', {
  description: 'Total number of slow Prisma queries',
});

// 创建带监控的 Prisma Client
export function createMonitoredPrisma() {
  const prisma = new PrismaClient({
    log: [
      { emit: 'event', level: 'query' },
      { emit: 'event', level: 'error' },
    ],
  });

  prisma.$on('query' as any, (event: Prisma.QueryEvent) => {
    const duration = event.duration;
    
    // 记录指标
    queryCount.add(1, { operation: event.operation });
    queryDuration.record(duration, { operation: event.operation });

    // 记录慢查询
    if (duration > slowQueryThreshold) {
      slowQueryCount.add(1, { operation: event.operation });
      
      console.warn('Slow query detected:', {
        operation: event.operation,
        duration,
        query: event.query,
        params: event.params,
      });

      // 发送告警（如果超过临界值）
      if (duration > criticalQueryThreshold) {
        sendAlert('CRITICAL_SLOW_QUERY', {
          operation: event.operation,
          duration,
          query: event.query,
        });
      }
    }
  });

  prisma.$on('error' as any, (event: Prisma.LogEvent) => {
    console.error('Prisma error:', event);
    
    sendAlert('PRISMA_ERROR', {
      message: event.message,
      target: event.target,
    });
  });

  return prisma;
}

// 连接池监控
export async function monitorConnectionPool(prisma: PrismaClient) {
  setInterval(async () => {
    const query = await prisma.$queryRaw`
      SELECT 
        count(*) as "connectionCount",
        state
      FROM pg_stat_activity
      WHERE datname = current_database()
    `;

    const metrics = {
      totalConnections: query.length,
      activeConnections: query.filter((c: any) => c.state === 'active').length,
      idleConnections: query.filter((c: any) => c.state === 'idle').length,
    };

    console.log('Connection pool metrics:', metrics);

    // 告警条件
    if (metrics.activeConnections > 80) {
      sendAlert('HIGH_CONNECTION_USAGE', metrics);
    }
  }, 60000); // 每分钟检查一次
}
```

---

## Prisma 与测试

```typescript
// tests/setup/prisma.ts
import { PrismaClient } from '@prisma/client';
import { execSync } from 'child_process';

// 测试数据库配置
const testDatabaseUrl = 'postgresql://test:test@localhost:5432/test_db';

export const prisma = new PrismaClient({
  datasources: {
    db: {
      url: testDatabaseUrl,
    },
  },
});

export async function setupTestDatabase() {
  // 创建测试数据库
  execSync(`createdb test_db`, { stdio: 'ignore' });
  
  // 运行迁移
  await prisma.$executeRaw`CREATE EXTENSION IF NOT EXISTS "uuid-ossp"`;
  
  // 推送 schema
  execSync('npx prisma db push --accept-data-loss', {
    env: { ...process.env, DATABASE_URL: testDatabaseUrl },
  });
}

export async function cleanupTestDatabase() {
  // 清理数据
  await prisma.$executeRaw`TRUNCATE TABLE "User", "Post", "Comment" CASCADE`;
}

export async function teardownTestDatabase() {
  await prisma.$disconnect();
  execSync('dropdb test_db', { stdio: 'ignore' });
}

// 测试工厂
export class TestFactory {
  constructor(private prisma: PrismaClient) {}

  async createUser(overrides = {}) {
    return this.prisma.user.create({
      data: {
        email: `test_${Date.now()}@example.com`,
        name: 'Test User',
        password: await hashPassword('password'),
        ...overrides,
      },
    });
  }

  async createPost(authorId: string, overrides = {}) {
    return this.prisma.post.create({
      data: {
        title: 'Test Post',
        slug: `test-${Date.now()}`,
        content: 'Test content',
        authorId,
        ...overrides,
      },
    });
  }

  async createPostWithAuthor(overrides = {}) {
    const user = await this.createUser();
    const post = await this.createPost(user.id, overrides);
    return { user, post };
  }
}
```

---

## Prisma 生产环境检查清单

| 检查项 | 描述 | 优先级 |
|--------|------|--------|
| **连接池配置** | 设置合适的 connection_limit | 高 |
| **索引优化** | 为常用查询字段添加索引 | 高 |
| **查询监控** | 启用查询日志，监控慢查询 | 高 |
| **迁移策略** | 使用 migrate deploy，避免自动迁移 | 高 |
| **备份策略** | 配置数据库自动备份 | 高 |
| **连接超时** | 设置合理的 timeout | 中 |
| **SSL 连接** | 生产环境启用 SSL | 中 |
| **日志脱敏** | 敏感数据不在日志中暴露 | 中 |
| **健康检查** | 实现数据库连接健康检查 | 中 |
| **重试机制** | 实现查询失败重试 | 低 |

---

> [!SUCCESS]
> Prisma 作为 TypeScript 生态中最成熟的 ORM 框架，以其声明式 Schema、类型安全客户端和丰富的扩展生态，成为 vibecoding 工作流中的数据库操作首选。本文档涵盖了从基础到高级的完整知识体系，包括复杂数据建模、性能优化、监控告警等关键主题，帮助开发者构建生产级别的数据访问层。
