# Drizzle ORM：轻量级 TypeScript ORM 的全面解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 Drizzle ORM 的核心概念、Schema 定义、查询构建器，以及与 Prisma/TypeORM 的对比分析。

---

## 目录

1. [[#概述与核心优势]]
2. [[#Drizzle Schema 定义]]
3. [[#Drizzle 查询构建器]]
4. [[#Drizzle 迁移工具]]
5. [[#与主流 ORM 对比]]
6. [[#性能优化]]
7. [[#实战场景]]
8. [[#选型建议]]

---

## 概述与核心优势

### 什么是 Drizzle ORM

Drizzle 是一个轻量级的 TypeScript ORM，由 PostHog 团队开发。它以「接近 SQL 的语法」和「零运行时开销」为设计理念，提供类型安全数据库操作的同时，保持极高的性能。

Drizzle 的核心理念：**让数据库操作像写 SQL 一样直观，同时获得 TypeScript 的类型安全**。

### 技术架构

```
┌─────────────────────────────────────────────────────┐
│                   Drizzle ORM                        │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────────────────────────────────────┐   │
│  │           TypeScript Schema                   │   │
│  │  const users = pgTable('users', {          │   │
│  │    id: serial('id').primaryKey(),          │   │
│  │    name: varchar('name', { length: 100 }) │   │
│  │  })                                        │   │
│  └─────────────────────────────────────────────┘   │
│                        ↓                            │
│  ┌─────────────────────────────────────────────┐   │
│  │           SQL Query Builder                  │   │
│  │  db.select().from(users)                   │   │
│  │    .where(eq(users.id, 1))                 │   │
│  └─────────────────────────────────────────────┘   │
│                        ↓                            │
│  ┌─────────────────────────────────────────────┐   │
│  │           PostgreSQL / MySQL / SQLite        │   │
│  └─────────────────────────────────────────────┘   │
│                                                      │
└─────────────────────────────────────────────────────┘
```

### 核心优势

| 优势 | 说明 |
|------|------|
| **零运行时** | 无额外运行时开销，直接编译成 SQL |
| **SQL-like 语法** | 接近原生 SQL，易于理解和调试 |
| **完整的类型推断** | 端到端 TypeScript 类型安全 |
| **轻量级** | Bundle 体积约 80KB |
| **多数据库支持** | PostgreSQL、MySQL、SQLite |
| **迁移友好** | 内置迁移工具，支持版本控制 |
| **高度可组合** | 链式 API，支持复杂查询 |

### 核心特性对比

| 特性 | Drizzle | Prisma | TypeORM |
|------|---------|--------|---------|
| **Bundle 体积** | ~80KB | ~200KB | ~350KB |
| **运行时开销** | 零 | 低 | 低 |
| **语法风格** | SQL-like | 链式 API | 链式 API |
| **Schema 定义** | TypeScript 对象 | .prisma 文件 | 装饰器 |
| **迁移工具** | 内置 | 内置 | 需配置 |
| **性能** | 最快 | 快 | 良好 |

---

## Drizzle Schema 定义

### 安装与配置

```bash
# 安装 Drizzle ORM
npm install drizzle-orm
npm install drizzle-kit --save-dev

# 安装数据库驱动
npm install @types/pg pg        # PostgreSQL
npm install mysql2               # MySQL
npm install better-sqlite3     # SQLite
```

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
} satisfies Config;
```

### Schema 定义基础

```typescript
// src/db/schema.ts
import { pgTable, serial, varchar, text, boolean, timestamp, integer, decimal, json, pgEnum } from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// ==================== 枚举定义 ====================

export const userRoleEnum = pgEnum('user_role', ['user', 'moderator', 'admin']);

export const postStatusEnum = pgEnum('post_status', ['draft', 'published', 'archived']);

// ==================== 用户表 ====================

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  name: varchar('name', { length: 100 }),
  passwordHash: text('password_hash').notNull(),
  avatar: text('avatar'),
  bio: text('bio'),
  role: userRoleEnum('role').default('user').notNull(),
  isActive: boolean('is_active').default(true).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
  lastLoginAt: timestamp('last_login_at'),
});

// ==================== 文章表 ====================

export const posts = pgTable('posts', {
  id: serial('id').primaryKey(),
  title: varchar('title', { length: 255 }).notNull(),
  slug: varchar('slug', { length: 255 }).notNull().unique(),
  content: text('content'),
  excerpt: text('excerpt'),
  coverImage: text('cover_image'),
  status: postStatusEnum('status').default('draft').notNull(),
  viewCount: integer('view_count').default(0).notNull(),
  likeCount: integer('like_count').default(0).notNull(),
  authorId: integer('author_id').notNull().references(() => users.id),
  publishedAt: timestamp('published_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// ==================== 标签表 ====================

export const tags = pgTable('tags', {
  id: serial('id').primaryKey(),
  name: varchar('name', { length: 50 }).notNull().unique(),
  slug: varchar('slug', { length: 50 }).notNull().unique(),
});

// ==================== 文章标签关联表 ====================

export const postTags = pgTable('post_tags', {
  postId: integer('post_id').notNull().references(() => posts.id, { onDelete: 'cascade' }),
  tagId: integer('tag_id').notNull().references(() => tags.id, { onDelete: 'cascade' }),
}, (table) => ({
  pk: { columns: [table.postId, table.tagId] },
}));

// ==================== 评论表 ====================

export const comments = pgTable('comments', {
  id: serial('id').primaryKey(),
  content: text('content').notNull(),
  authorId: integer('author_id').notNull().references(() => users.id),
  postId: integer('post_id').notNull().references(() => posts.id, { onDelete: 'cascade' }),
  parentId: integer('parent_id'),
  isEdited: boolean('is_edited').default(false).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// ==================== 会话表 ====================

export const sessions = pgTable('sessions', {
  id: text('id').primaryKey(),
  userId: integer('user_id').notNull().references(() => users.id, { onDelete: 'cascade' }),
  expiresAt: timestamp('expires_at').notNull(),
  ipAddress: text('ip_address'),
  userAgent: text('user_agent'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

### 关系定义

```typescript
// src/db/schema.ts

// 用户与文章关系
export const usersRelations = relations(users, ({ many }) => ({
  posts: many(posts),
  comments: many(comments),
  sessions: many(sessions),
}));

// 文章与用户关系
export const postsRelations = relations(posts, ({ one, many }) => ({
  author: one(users, {
    fields: [posts.authorId],
    references: [users.id],
  }),
  comments: many(comments),
  postTags: many(postTags),
}));

// 文章与标签关系
export const postTagsRelations = relations(postTags, ({ one }) => ({
  post: one(posts, {
    fields: [postTags.postId],
    references: [posts.id],
  }),
  tag: one(tags, {
    fields: [postTags.tagId],
    references: [tags.id],
  }),
}));

// 标签与文章关系
export const tagsRelations = relations(tags, ({ many }) => ({
  postTags: many(postTags),
}));

// 评论与用户/文章关系
export const commentsRelations = relations(comments, ({ one }) => ({
  author: one(users, {
    fields: [comments.authorId],
    references: [users.id],
  }),
  post: one(posts, {
    fields: [comments.postId],
    references: [posts.id],
  }),
  parent: one(comments, {
    fields: [comments.parentId],
    references: [comments.id],
    relationName: 'replies',
  }),
}));

// 自引用：评论的回复
export const commentRepliesRelations = relations(comments, ({ many }) => ({
  replies: many(comments, { relationName: 'replies' }),
}));
```

### 索引与约束

```typescript
// 高级索引定义
export const posts = pgTable('posts', {
  // ... 其他字段
  
}, (table) => ({
  // 主键
  pk: { columns: [table.id] },
  
  // 唯一索引
  slugUnique: unique('posts_slug_unique').on(table.slug),
  
  // 普通索引
  authorIdIdx: index('posts_author_id_idx').on(table.authorId),
  statusIdx: index('posts_status_idx').on(table.status),
  
  // 复合索引
  authorStatusIdx: index('posts_author_status_idx').on(table.authorId, table.status),
  
  // 全文搜索索引
  titleFtsIdx: index('posts_title_fts_idx').on(
    computed('to_tsvector(\'english\', title)')
  ),
}));

// 完整约束示例
export const orderItems = pgTable('order_items', {
  id: serial('id').primaryKey(),
  orderId: integer('order_id').notNull().references(() => orders.id),
  productId: integer('product_id').notNull().references(() => products.id),
  quantity: integer('quantity').notNull(),
  unitPrice: decimal('unit_price', { precision: 10, scale: 2 }).notNull(),
  total: decimal('total', { precision: 10, scale: 2 }).notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  // 检查约束
  positiveQuantity: check('positive_quantity', sql`${table.quantity} > 0`),
  positiveTotal: check('positive_total', sql`${table.total} > 0`),
  
  // 外键约束名称
  orderIdFk: foreignKey({
    columns: [table.orderId],
    foreignColumns: [orders.id],
    name: 'order_items_order_id_fk',
  }),
}));
```

---

## Drizzle 查询构建器

### 基础 CRUD 操作

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { eq, ne, gt, gte, lt, lte, and, or, like, inArray, isNull, isNotNull, between, sql } from 'drizzle-orm';
import * as schema from './schema';

const db = drizzle(process.env.DATABASE_URL!, { schema });

// ==================== 创建（Create）====================

// 创建单条记录
const [newUser] = await db.insert(users).values({
  email: 'alice@example.com',
  name: 'Alice',
  passwordHash: 'hashed_password',
}).returning();

console.log(newUser.id);

// 批量创建
const newPosts = await db.insert(posts).values([
  { title: 'Post 1', slug: 'post-1', authorId: 1 },
  { title: 'Post 2', slug: 'post-2', authorId: 1 },
  { title: 'Post 3', slug: 'post-3', authorId: 2 },
]).returning();

// ==================== 读取（Read）====================

// 查询单条
const user = await db.select().from(users).where(eq(users.id, 1)).limit(1);

// 查询多条
const allUsers = await db.select().from(users);

// 条件查询
const activeUsers = await db.select().from(users).where(
  and(
    eq(users.isActive, true),
    eq(users.role, 'user')
  )
);

// IN 查询
const selectedUsers = await db.select().from(users).where(
  inArray(users.id, [1, 2, 3])
);

// LIKE 查询
const matchingUsers = await db.select().from(users).where(
  like(users.name, '%Alice%')
);

// NULL 检查
const usersWithoutBio = await db.select().from(users).where(
  isNull(users.bio)
);

// BETWEEN 查询
const recentPosts = await db.select().from(posts).where(
  between(posts.createdAt, new Date('2024-01-01'), new Date())
);

// ==================== 更新（Update）====================

// 更新单条
const [updatedUser] = await db.update(users)
  .set({ name: 'Alice Updated', updatedAt: new Date() })
  .where(eq(users.id, 1))
  .returning();

// 批量更新
await db.update(posts)
  .set({ status: 'published', publishedAt: new Date() })
  .where(eq(posts.authorId, 1));

// 原子递增
await db.update(posts)
  .set({ viewCount: sql`${posts.viewCount} + 1` })
  .where(eq(posts.id, 1));

// ==================== 删除（Delete）====================

// 删除单条
await db.delete(users).where(eq(users.id, 1));

// 批量删除
await db.delete(posts).where(
  and(
    eq(posts.status, 'draft'),
    lt(posts.createdAt, new Date('2024-01-01'))
);
```

### 关联查询

```typescript
// ==================== 简单关联 ====================

// 获取用户及其文章
const usersWithPosts = await db.select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId));

// ==================== 使用 relations ====================

// 获取用户及其文章（使用定义的关系）
const userWithPosts = await db.query.users.findFirst({
  where: eq(users.id, 1),
  with: {
    posts: {
      where: eq(posts.status, 'published'),
      orderBy: (posts, { desc }) => [desc(posts.createdAt)],
      limit: 10,
    },
  },
});

console.log(userWithPosts?.posts);

// 获取文章及其评论和作者
const postsWithDetails = await db.query.posts.findMany({
  with: {
    author: true,
    comments: {
      with: {
        author: {
          columns: { id: true, name: true, avatar: true },
        },
      },
    },
  },
  where: eq(posts.status, 'published'),
  orderBy: (posts, { desc }) => [desc(posts.createdAt)],
  limit: 20,
});

// ==================== 嵌套查询 ====================

// 聚合查询
const postsWithCommentCount = await db.select({
  post: posts,
  commentCount: sql<number>`count(${comments.id})::int`,
}).from(posts)
  .leftJoin(comments, eq(posts.id, comments.postId))
  .where(eq(posts.status, 'published'))
  .groupBy(posts.id)
  .orderBy(desc(posts.createdAt))
  .limit(10);

// 获取每个用户的文章数
const userStats = await db.select({
  userId: users.id,
  userName: users.name,
  postCount: sql<number>`count(${posts.id})::int`,
})
  .from(users)
  .leftJoin(posts, eq(users.id, posts.authorId))
  .groupBy(users.id, users.name)
  .orderBy(sql`count(${posts.id}) desc`);
```

### 高级查询

```typescript
// ==================== 复杂 WHERE 条件 ====================

// OR 条件
const result = await db.select().from(users).where(
  or(
    eq(users.role, 'admin'),
    and(
      eq(users.isActive, true),
      like(users.email, '%@example.com')
    )
  )
);

// 子查询
const topAuthors = await db.select({
  id: users.id,
  name: users.name,
  email: users.email,
  postCount: sql<number>`(
    SELECT COUNT(*) FROM posts WHERE posts.author_id = users.id
  )`,
})
  .from(users)
  .orderBy(sql`post_count DESC`)
  .limit(10);

// EXISTS 子查询
const usersWithPublishedPosts = await db.select().from(users)
  .where(exists(
    db.select({ id: posts.id })
      .from(posts)
      .where(
        and(
          eq(posts.authorId, users.id),
          eq(posts.status, 'published')
        )
      )
      .limit(1)
  ));

// ==================== 排序与分页 ====================

// 多字段排序
const sortedPosts = await db.select().from(posts)
  .orderBy(
    desc(posts.status),
    desc(posts.publishedAt),
    asc(posts.title)
  )
  .limit(20)
  .offset(40);

// ==================== 事务操作 ====================

// 基本事务
await db.transaction(async (tx) => {
  // 创建文章
  const [post] = await tx.insert(posts).values({
    title: 'New Post',
    slug: 'new-post',
    authorId: 1,
  }).returning();

  // 创建标签
  const [tag] = await tx.insert(tags).values({
    name: 'Tech',
    slug: 'tech',
  }).returning();

  // 关联文章和标签
  await tx.insert(postTags).values({
    postId: post.id,
    tagId: tag.id,
  });
});

// 事务控制
await db.transaction(async (tx) => {
  try {
    await tx.update(users)
      .set({ role: 'moderator' })
      .where(eq(users.id, 1));
    
    await tx.insert(auditLogs).values({
      userId: 1,
      action: 'PROMOTE_TO_MODERATOR',
      entity: 'User',
      entityId: 1,
    });
    
    tx.commit(); // 显式提交
  } catch (error) {
    tx.rollback(); // 出错时回滚
  }
});

// ==================== 批量事务 ====================

// 保存点
await db.transaction(async (tx) => {
  const [user] = await tx.insert(users).values({
    email: 'new@example.com',
    name: 'New User',
    passwordHash: 'hash',
  }).returning();

  try {
    await tx.savepoint('after_user', async (sp) => {
      await sp.insert(posts).values({
        title: 'Post 1',
        slug: 'post-1',
        authorId: user.id,
      });
    });
  } catch (error) {
    // 保存点失败不会影响整个事务
  }
});
```

---

## Drizzle 迁移工具

### 迁移命令

```bash
# 生成迁移文件（开发环境）
npx drizzle-kit generate

# 推送 Schema 到数据库（开发环境）
npx drizzle-kit push

# 执行迁移
npx drizzle-kit migrate

# 创建迁移（生产环境）
npx drizzle-kit generate
npx drizzle-kit migrate

# 查看迁移状态
npx drizzle-kit status

# 下推 Schema（从数据库拉取）
npx drizzle-kit pull

# 检查 Schema 与数据库差异
npx drizzle-kit check
```

### 迁移文件示例

```sql
-- drizzle/0000_warm_berserk.sql

CREATE TABLE IF NOT EXISTS "users" (
  "id" serial PRIMARY KEY,
  "email" varchar(255) NOT NULL,
  "name" varchar(100),
  "password_hash" text NOT NULL,
  "avatar" text,
  "bio" text,
  "role" user_role DEFAULT 'user' NOT NULL,
  "is_active" boolean DEFAULT true NOT NULL,
  "created_at" timestamp DEFAULT now() NOT NULL,
  "updated_at" timestamp DEFAULT now() NOT NULL,
  "last_login_at" timestamp
);

CREATE UNIQUE INDEX IF NOT EXISTS "users_email_unique" ON "users" ("email");

CREATE INDEX IF NOT EXISTS "users_role_idx" ON "users" ("role");

CREATE INDEX IF NOT EXISTS "users_created_at_idx" ON "users" ("created_at");

CREATE TABLE IF NOT EXISTS "posts" (
  "id" serial PRIMARY KEY,
  "title" varchar(255) NOT NULL,
  "slug" varchar(255) NOT NULL,
  "content" text,
  "excerpt" text,
  "cover_image" text,
  "status" post_status DEFAULT 'draft' NOT NULL,
  "view_count" integer DEFAULT 0 NOT NULL,
  "like_count" integer DEFAULT 0 NOT NULL,
  "author_id" integer NOT NULL REFERENCES "users"("id"),
  "published_at" timestamp,
  "created_at" timestamp DEFAULT now() NOT NULL,
  "updated_at" timestamp DEFAULT now() NOT NULL,
  UNIQUE ("slug")
);

CREATE INDEX IF NOT EXISTS "posts_author_id_idx" ON "posts" ("author_id");
CREATE INDEX IF NOT EXISTS "posts_status_idx" ON "posts" ("status");
```

### 版本控制策略

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/db/schema.ts',
  out: './drizzle',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
  
  // 迁移配置
  migrationsSchema: 'drizzle',
  migrationsTable: 'drizzle_migrations',
  
  // 穿透配置
  verbose: true,
  strict: true,
} satisfies Config;
```

---

## 与主流 ORM 对比

### 功能矩阵

| 功能 | Drizzle | Prisma | TypeORM | Sequelize |
|------|---------|--------|---------|-----------|
| **Bundle 体积** | ~80KB | ~200KB | ~350KB | ~400KB |
| **运行时开销** | 零 | 低 | 低 | 中 |
| **SQL 输出** | ✅ 原生 SQL | ⚠️ 优化 SQL | ⚠️ 优化 SQL | ⚠️ 优化 SQL |
| **类型推断** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **Schema 定义** | TypeScript | .prisma DSL | 装饰器/对象 | 对象 |
| **迁移工具** | 内置 | 内置 | 需配置 | 需配置 |
| **关联 API** | relations() | 链式 | findOptions | associations |
| **性能** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

### 性能对比（基准测试）

| 操作 | Drizzle | Prisma | TypeORM |
|------|---------|--------|---------|
| **简单查询** | 0.9ms | 1.2ms | 1.8ms |
| **关联查询** | 2.8ms | 3.5ms | 5.2ms |
| **批量插入 1000 条** | 32ms | 45ms | 68ms |
| **Bundle 体积** | ~80KB | ~200KB | ~350KB |

### Drizzle vs Prisma 选择指南

| 场景 | 推荐 | 原因 |
|------|------|------|
| **性能敏感** | Drizzle | 零运行时，最快性能 |
| **快速开发** | Prisma | 声明式 Schema，开发快 |
| **复杂业务** | Prisma/Drizzle | 两者都可 |
| **SQL 熟练团队** | Drizzle | SQL-like 语法直观 |
| **TypeScript 新手** | Prisma | 学习曲线更平缓 |

---

## 性能优化

### 查询优化

```typescript
// ==================== 选择性字段 ====================

// 只查询需要的字段
const userEmails = await db.select({
  id: users.id,
  email: users.email,
}).from(users);

// ==================== 避免 N+1 ====================

// 一次性加载关联
const postsWithAuthors = await db.query.posts.findMany({
  with: {
    author: {
      columns: { id: true, name: true, avatar: true },
    },
    comments: {
      with: {
        author: {
          columns: { id: true, name: true },
        },
      },
      limit: 5,
    },
  },
});

// ==================== 批量操作 ====================

// 批量插入（比循环单条插入快 10 倍+）
const BATCH_SIZE = 1000;
for (let i = 0; i < users.length; i += BATCH_SIZE) {
  const batch = users.slice(i, i + BATCH_SIZE);
  await db.insert(users).values(batch);
}

// ==================== 索引优化 ====================

// 使用 EXPLAIN 分析查询
const explain = await db.execute(sql`
  EXPLAIN ANALYZE
  SELECT * FROM posts
  WHERE status = 'published'
  ORDER BY created_at DESC
`);

console.log(explain);
```

### 缓存策略

```typescript
// 缓存辅助函数
import { caching } from 'cache-manager';

const cache = await caching('memory', { ttl: 60000 });

async function cachedQuery<T>(
  key: string,
  queryFn: () => Promise<T>
): Promise<T> {
  const cached = await cache.get<T>(key);
  if (cached) return cached;
  
  const result = await queryFn();
  await cache.set(key, result);
  return result;
}

// 使用示例
async function getPopularPosts() {
  return cachedQuery('popular_posts', async () => {
    return db.select().from(posts)
      .where(eq(posts.status, 'published'))
      .orderBy(desc(posts.viewCount))
      .limit(10);
  });
}

// 缓存失效
async function invalidatePostCache() {
  await cache.del('popular_posts');
  await cache.del('recent_posts');
}
```

---

## 实战场景

### 场景：博客系统

```typescript
// src/services/blog.service.ts
import { db } from '@/db';
import { users, posts, comments, tags, postTags } from '@/db/schema';
import { eq, desc, and, like, sql } from 'drizzle-orm';

export class BlogService {
  // 获取首页文章列表
  async getHomepagePosts(page: number = 1, limit: number = 10) {
    const offset = (page - 1) * limit;
    
    const results = await db.select({
      post: posts,
      author: {
        id: users.id,
        name: users.name,
        avatar: users.avatar,
      },
    })
      .from(posts)
      .leftJoin(users, eq(posts.authorId, users.id))
      .where(eq(posts.status, 'published'))
      .orderBy(desc(posts.publishedAt))
      .limit(limit)
      .offset(offset);

    return results;
  }

  // 获取文章详情
  async getPostBySlug(slug: string) {
    const post = await db.query.posts.findFirst({
      where: and(
        eq(posts.slug, slug),
        eq(posts.status, 'published')
      ),
      with: {
        author: {
          columns: { id: true, name: true, avatar: true, bio: true },
        },
        postTags: {
          with: { tag: true },
        },
        comments: {
          where: isNull(comments.parentId),
          with: {
            author: {
              columns: { id: true, name: true, avatar: true },
            },
          },
          orderBy: desc(comments.createdAt),
        },
      },
    });

    if (post) {
      // 增加浏览量
      await db.update(posts)
        .set({ viewCount: sql`${posts.viewCount} + 1` })
        .where(eq(posts.id, post.id));
    }

    return post;
  }

  // 搜索文章
  async searchPosts(query: string, page: number = 1, limit: number = 20) {
    const offset = (page - 1) * limit;
    
    return db.select({
      post: posts,
      author: {
        id: users.id,
        name: users.name,
      },
    })
      .from(posts)
      .leftJoin(users, eq(posts.authorId, users.id))
      .where(
        and(
          eq(posts.status, 'published'),
          or(
            like(posts.title, `%${query}%`),
            like(posts.content, `%${query}%`)
          )
        )
      )
      .orderBy(desc(posts.publishedAt))
      .limit(limit)
      .offset(offset);
  }

  // 创建文章
  async createPost(data: {
    title: string;
    slug: string;
    content: string;
    authorId: number;
    tagIds?: number[];
  }) {
    return db.transaction(async (tx) => {
      // 创建文章
      const [post] = await tx.insert(posts).values({
        ...data,
        status: 'draft',
      }).returning();

      // 添加标签
      if (data.tagIds?.length) {
        await tx.insert(postTags).values(
          data.tagIds.map(tagId => ({
            postId: post.id,
            tagId,
          }))
        );
      }

      return post;
    });
  }

  // 发布文章
  async publishPost(postId: number) {
    return db.update(posts)
      .set({
        status: 'published',
        publishedAt: new Date(),
        updatedAt: new Date(),
      })
      .where(eq(posts.id, postId))
      .returning();
  }
}

export const blogService = new BlogService();
```

---

## Drizzle 与 Next.js 集成

```typescript
// src/lib/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import * as schema from '@/db/schema';

const globalForPool = globalThis as unknown as {
  pool: Pool | undefined;
};

const pool = globalForPool.pool ?? new Pool({
  connectionString: process.env.DATABASE_URL,
});

if (process.env.NODE_ENV !== 'production') {
  globalForPool.pool = pool;
}

export const db = drizzle(pool, { schema });

// src/app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { db } from '@/lib/db';
import { posts, users } from '@/db/schema';
import { eq, desc } from 'drizzle-orm';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const page = parseInt(searchParams.get('page') || '1');
  const limit = Math.min(parseInt(searchParams.get('limit') || '20'), 100);
  const offset = (page - 1) * limit;

  const results = await db.select({
    id: posts.id,
    title: posts.title,
    slug: posts.slug,
    excerpt: posts.excerpt,
    viewCount: posts.viewCount,
    createdAt: posts.createdAt,
    author: {
      id: users.id,
      name: users.name,
      avatar: users.avatar,
    },
  })
    .from(posts)
    .leftJoin(users, eq(posts.authorId, users.id))
    .where(eq(posts.status, 'published'))
    .orderBy(desc(posts.createdAt))
    .limit(limit)
    .offset(offset);

  return NextResponse.json(results);
}
```

---

## 选型建议

### 选择 Drizzle 的理由

- ✅ 性能敏感的应用（高频查询）
- ✅ TypeScript 团队，SQL 熟练
- ✅ 需要轻量级 ORM
- ✅ 对 Bundle 体积有要求
- ✅ 追求接近 SQL 的语法

### 不选择 Drizzle 的场景

- ❌ 需要快速原型开发（Prisma 更快）
- ❌ 习惯装饰器模式（TypeORM 更适合）
- ❌ 需要复杂的数据可视化（Prisma Studio）
- ❌ MongoDB 专用项目（Prisma MongoDB 支持更好）

---

## Drizzle 高级特性

### 复杂关系查询

```typescript
// 多对多关系
const posts = await db.query.posts.findMany({
  with: {
    author: {
      columns: { id: true, name: true, avatar: true },
    },
    postTags: {
      with: {
        tag: true,
      },
    },
  },
});

// 嵌套关系
const users = await db.query.users.findMany({
  with: {
    posts: {
      with: {
        comments: {
          with: {
            author: {
              columns: { id: true, name: true },
            },
          },
        },
      },
    },
  },
});

// 自引用关系（树形结构）
const categoryTree = await db.query.categories.findMany({
  where: isNull(categories.parentId),
  with: {
    children: {
      with: {
        children: true,
      },
    },
  },
});
```

### 全文搜索集成

```typescript
// PostgreSQL 全文搜索
const searchResults = await db.execute(sql`
  SELECT
    id,
    title,
    ts_rank(search_vector, plainto_tsquery('english', ${searchQuery})) as rank
  FROM posts
  WHERE search_vector @@ plainto_tsquery('english', ${searchQuery})
  ORDER BY rank DESC
  LIMIT 10
`);
```

### Drizzle 与 Serverless

```typescript
// Vercel Edge Functions
import { drizzle } from 'drizzle-orm/vercel-postgres';
import { sql } from '@vercel/postgres';

const db = drizzle(sql);

export const runtime = 'edge';

export async function GET(request: Request) {
  const posts = await db.select()
    .from(postsTable)
    .limit(10);
  return Response.json(posts);
}

// Cloudflare Workers
import { drizzle } from 'drizzle-orm/d1';

export default {
  async fetch(request: Request, env: Env) {
    const db = drizzle(env.DB);
    const users = await db.select().from(usersTable).limit(10);
    return Response.json(users);
  },
};
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://orm.drizzle.team |
| GitHub | https://github.com/drizzle-team/drizzle-orm |
| Drizzle Kit | https://github.com/drizzle-team/drizzle-kit |
| 示例项目 | https://github.com/drizzle-team/drizzle-kit-examples |

---

---

## Drizzle 高级安全配置

### SQL 注入防护

```typescript
// src/lib/sanitize.ts
import { sql, SQL } from 'drizzle-orm';

// 安全查询构建
export function buildSafeSearchQuery(
  table: typeof posts,
  searchTerm: string,
  allowedFields: string[]
): SQL {
  // 只允许预定义的字段
  const sanitizedFields = allowedFields.filter(field => 
    ['title', 'content', 'excerpt'].includes(field)
  );
  
  // 使用参数化查询防止 SQL 注入
  const conditions = sanitizedFields.map(field => 
    sql`${sql.raw(field)} ILIKE ${'%' + searchTerm + '%'}`
  );
  
  return sql.join(conditions, sql` OR `);
}

// 使用示例
const searchQuery = buildSafeSearchQuery(posts, userInput, ['title', 'content']);
const results = await db.select().from(posts).where(searchQuery);
```

### 行级安全策略

```typescript
// PostgreSQL Row Level Security with Drizzle
import { pgPolicy } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull(),
  role: userRoleEnum('role').default('user').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});

// 创建 RLS 策略
export const usersPolicies = {
  // 用户只能查看自己的数据（除非是管理员）
  selectOwnData: pgPolicy('users_select_own_data', {
    for: 'select',
    to: 'public',
    using: sql`(auth.uid() = id OR (SELECT role FROM users WHERE id = auth.uid()) = 'admin')`,
  }),
  
  // 用户只能更新自己的数据
  updateOwnData: pgPolicy('users_update_own_data', {
    for: 'update',
    to: 'public',
    using: sql`(auth.uid() = id)`,
    withCheck: sql`(auth.uid() = id)`,
  }),
};

// 应用策略
await db.execute(sql`
  ALTER TABLE users ENABLE ROW LEVEL SECURITY;
  CREATE POLICY users_select_own_data ON users FOR SELECT USING (auth.uid() = id OR (SELECT role FROM users WHERE id = auth.uid()) = 'admin');
`);
```

---

## Drizzle 性能调优

### 连接池配置

```typescript
// src/lib/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool, PoolConfig } from 'pg';

const poolConfig: PoolConfig = {
  connectionString: process.env.DATABASE_URL,
  
  // 连接池大小
  max: 20,           // 最大连接数
  idleTimeoutMillis: 30000,  // 空闲超时
  connectionTimeoutMillis: 2000,  // 连接超时
  
  // 重试配置
  maxUses: 7500,     // 连接最大使用次数后重建
  
  // SSL 配置
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: true,
  } : false,
};

const pool = new Pool(poolConfig);

// 监控连接池
pool.on('connect', () => {
  console.log('New client connected to pool');
});

pool.on('error', (err) => {
  console.error('Unexpected error on idle client', err);
});

export const db = drizzle(pool, { 
  schema,
  logger: process.env.NODE_ENV === 'development',
});
```

### 查询性能监控

```typescript
// src/lib/performance.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';

interface QueryMetrics {
  query: string;
  executionTime: number;
  rowsReturned: number;
  timestamp: Date;
}

const metrics: QueryMetrics[] = [];

// 监控装饰器
export function monitorQuery<T extends (...args: any[]) => Promise<any>>(
  queryName: string,
  fn: T
): T {
  return (async (...args: any[]) => {
    const start = performance.now();
    
    try {
      const result = await fn(...args);
      const duration = performance.now() - start;
      
      metrics.push({
        query: queryName,
        executionTime: duration,
        rowsReturned: Array.isArray(result) ? result.length : 1,
        timestamp: new Date(),
      });
      
      // 保留最近 1000 条记录
      if (metrics.length > 1000) {
        metrics.shift();
      }
      
      return result;
    } catch (error) {
      console.error(`Query ${queryName} failed:`, error);
      throw error;
    }
  }) as T;
}

// 分析慢查询
export function getSlowQueries(threshold = 100): QueryMetrics[] {
  return metrics
    .filter(m => m.executionTime > threshold)
    .sort((a, b) => b.executionTime - a.executionTime);
}

// 获取查询统计
export function getQueryStats() {
  const byQuery = new Map<string, { count: number; totalTime: number; avgTime: number }>();
  
  metrics.forEach(m => {
    const existing = byQuery.get(m.query);
    if (existing) {
      existing.count++;
      existing.totalTime += m.executionTime;
      existing.avgTime = existing.totalTime / existing.count;
    } else {
      byQuery.set(m.query, {
        count: 1,
        totalTime: m.executionTime,
        avgTime: m.executionTime,
      });
    }
  });
  
  return Array.from(byQuery.entries())
    .map(([query, stats]) => ({ query, ...stats }))
    .sort((a, b) => b.avgTime - a.avgTime);
}
```

### 数据库连接健康检查

```typescript
// src/lib/health.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';

export interface HealthCheckResult {
  status: 'healthy' | 'degraded' | 'unhealthy';
  checks: {
    connection: boolean;
    queryExecution: boolean;
    poolUtilization: number;
  };
  metrics: {
    avgResponseTime: number;
    errorRate: number;
  };
}

export async function performHealthCheck(): Promise<HealthCheckResult> {
  const checks = {
    connection: false,
    queryExecution: false,
    poolUtilization: 0,
  };
  
  try {
    // 检查连接
    const connectionResult = await db.execute(sql`SELECT 1`);
    checks.connection = connectionResult.rowCount === 1;
    
    // 检查查询执行
    const start = Date.now();
    await db.execute(sql`SELECT 1`);
    const queryTime = Date.now() - start;
    checks.queryExecution = queryTime < 1000;
    
    // 检查连接池使用率
    // (需要访问底层 pool 对象)
    
    const status = checks.connection && checks.queryExecution ? 'healthy' : 'unhealthy';
    
    return {
      status,
      checks,
      metrics: {
        avgResponseTime: queryTime,
        errorRate: 0,
      },
    };
  } catch (error) {
    return {
      status: 'unhealthy',
      checks,
      metrics: {
        avgResponseTime: -1,
        errorRate: 1,
      },
    };
  }
}
```

---

## Drizzle 数据迁移高级策略

### 零停机迁移

```typescript
// migrations/0010_add_users_table.ts

// 阶段 1: 添加新表/字段（向后兼容）
export async function up(db: Database) {
  // 添加可空字段（新代码写入新字段，旧代码写入旧字段）
  await db.execute(sql`
    ALTER TABLE users 
    ADD COLUMN IF NOT EXISTS display_name VARCHAR(100);
  `);
  
  // 添加新索引（不锁定表）
  await db.execute(sql`
    CREATE INDEX CONCURRENTLY IF NOT EXISTS users_display_name_idx 
    ON users(display_name);
  `);
}

// 阶段 2: 后台数据同步
export async function migrateData(db: Database) {
  // 分批处理避免锁定
  const BATCH_SIZE = 1000;
  let offset = 0;
  let processed = 0;
  
  while (true) {
    const result = await db.execute(sql`
      UPDATE users 
      SET display_name = name
      WHERE id IN (
        SELECT id FROM users 
        WHERE display_name IS NULL
        LIMIT ${BATCH_SIZE}
      )
    `);
    
    if (result.rowCount === 0) break;
    processed += result.rowCount;
    offset += BATCH_SIZE;
    
    // 每批后短暂暂停
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  
  return processed;
}

// 阶段 3: 添加约束（向后兼容）
export async function addConstraints(db: Database) {
  // 添加非空约束
  await db.execute(sql`
    ALTER TABLE users 
    ALTER COLUMN display_name SET NOT NULL;
  `);
}

// 阶段 4: 清理旧代码后执行
export async function cleanup(db: Database) {
  // 删除旧字段（确认新代码不再使用后）
  await db.execute(sql`
    ALTER TABLE users DROP COLUMN IF EXISTS name;
  `);
}
```

### 数据库回滚策略

```typescript
// src/lib/rollback.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';

interface MigrationRecord {
  id: number;
  name: string;
  executedAt: Date;
  rollbackAt?: Date;
  status: 'completed' | 'rolled_back';
}

export async function recordMigration(
  name: string,
  direction: 'up' | 'down'
): Promise<void> {
  await db.insert(migrationLog).values({
    name,
    executedAt: new Date(),
    direction,
    status: 'completed',
  });
}

export async function recordRollback(
  migrationId: number
): Promise<void> {
  await db.update(migrationLog)
    .set({
      rollbackAt: new Date(),
      status: 'rolled_back',
    })
    .where(eq(migrationLog.id, migrationId));
}

export async function getMigrationStatus(): Promise<MigrationRecord[]> {
  return db.select().from(migrationLog)
    .orderBy(desc(migrationLog.executedAt));
}
```

---

## Drizzle 与 Redis 缓存集成

```typescript
// src/lib/cache.ts
import { db } from '@/lib/db';
import { redis } from '@/lib/redis';
import type { posts, users } from '@/db/schema';
import { eq, desc } from 'drizzle-orm';

type CachedEntity = typeof posts | typeof users;

// 缓存配置
const CACHE_TTL = 3600; // 1小时
const CACHE_PREFIX = 'drizzle:';

// 缓存辅助函数
export async function getCached<T>(
  key: string,
  fetchFn: () => Promise<T>
): Promise<T> {
  const cached = await redis.get(CACHE_PREFIX + key);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const result = await fetchFn();
  await redis.setex(CACHE_PREFIX + key, CACHE_TTL, JSON.stringify(result));
  
  return result;
}

// 缓存失效
export async function invalidateCache(
  pattern: string
): Promise<void> {
  const keys = await redis.keys(CACHE_PREFIX + pattern);
  
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}

// 带缓存的查询
export async function getCachedPosts(
  limit: number = 10,
  offset: number = 0
): Promise<typeof posts.$inferSelect[]> {
  const cacheKey = `posts:list:${limit}:${offset}`;
  
  return getCached(cacheKey, async () => {
    return db.select().from(posts)
      .where(eq(posts.status, 'published'))
      .orderBy(desc(posts.createdAt))
      .limit(limit)
      .offset(offset);
  });
}

// 缓存失效示例
export async function onPostCreated(postId: number) {
  // 失效相关缓存
  await invalidateCache('posts:*');
  await invalidateCache(`post:${postId}`);
}
```

---

## Drizzle 多租户实现

```typescript
// src/lib/tenant.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';
import type { PgDialect } from 'drizzle-orm/pg-core';

interface TenantContext {
  tenantId: string;
  schema: string;
}

// 租户 Schema 管理
export async function createTenantSchema(tenantId: string): Promise<void> {
  const schemaName = `tenant_${tenantId}`;
  
  // 创建 Schema
  await db.execute(sql`CREATE SCHEMA IF NOT EXISTS ${sql.raw(schemaName)}`);
  
  // 在 Schema 中创建表
  await db.execute(sql`
    CREATE TABLE IF NOT EXISTS ${sql.raw(schemaName)}.documents (
      id SERIAL PRIMARY KEY,
      title VARCHAR(255) NOT NULL,
      content TEXT,
      created_at TIMESTAMP DEFAULT NOW()
    )
  `);
}

// 切换租户上下文
export function withTenant<T>(
  tenantId: string,
  callback: () => Promise<T>
): Promise<T> {
  const schemaName = `tenant_${tenantId}`;
  
  return db.transaction(async (tx) => {
    // 设置 search_path
    await tx.execute(sql`SET search_path TO ${sql.raw(schemaName)}, public`);
    
    try {
      return await callback();
    } finally {
      // 恢复 search_path
      await tx.execute(sql`SET search_path TO public`);
    }
  });
}

// 行级租户隔离
export function addTenantFilter<T extends { tenantId: string }>(
  query: any,
  tenantId: string
) {
  return query.where(sql`tenant_id = ${tenantId}`);
}

// 跨租户查询（管理员）
export async function getAllTenantsData(tenantIds: string[]) {
  const results = [];
  
  for (const tenantId of tenantIds) {
    const data = await withTenant(tenantId, async () => {
      return db.select().from(documents);
    });
    results.push({ tenantId, data });
  }
  
  return results;
}
```

---

## Drizzle 事件驱动架构

```typescript
// src/lib/events.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';
import { EventEmitter } from 'events';

const eventEmitter = new EventEmitter();

// 实体事件类型
type EntityEvent = 
  | { type: 'user:created'; data: any }
  | { type: 'user:updated'; data: any }
  | { type: 'post:published'; data: any }
  | { type: 'comment:added'; data: any };

// 发布事件
export async function emitEntityEvent(event: EntityEvent): Promise<void> {
  // 保存到事件表（用于审计和重放）
  await db.insert(eventLog).values({
    type: event.type,
    data: JSON.stringify(event.data),
    createdAt: new Date(),
  });
  
  // 发布到内存事件系统
  eventEmitter.emit(event.type, event.data);
  
  // 发布到消息队列（异步）
  if (process.env.NODE_ENV === 'production') {
    await publishToQueue(event);
  }
}

// 订阅事件
eventEmitter.on('user:created', async (user) => {
  console.log('New user created:', user.id);
  // 发送欢迎邮件
  await sendWelcomeEmail(user.email);
});

eventEmitter.on('post:published', async (post) => {
  console.log('Post published:', post.id);
  // 更新统计
  await incrementPublishedPostsCount(post.authorId);
  // 发送通知
  await notifyFollowers(post.authorId, post.id);
});

// 事件重放
export async function replayEvents(
  fromDate: Date,
  toDate: Date,
  eventTypes?: string[]
): Promise<void> {
  const events = await db.select().from(eventLog)
    .where(sql`
      created_at >= ${fromDate} 
      AND created_at <= ${toDate}
      ${eventTypes ? sql`AND type IN (${sql.join(eventTypes.map(t => sql`${t}`), sql`, `)})` : sql``}
    `)
    .orderBy(eventLog.createdAt);
  
  for (const event of events) {
    const parsed = JSON.parse(event.data as string);
    eventEmitter.emit(event.type, parsed);
  }
}
```

---

## Drizzle 测试策略

```typescript
// src/__tests__/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { Pool } from 'pg';
import { migrate } from 'drizzle-orm/node-postgres/migrator';

// 测试数据库连接
const testPool = new Pool({
  connectionString: process.env.TEST_DATABASE_URL,
});

export const testDb = drizzle(testPool);

// 初始化测试数据库
export async function setupTestDatabase() {
  // 运行迁移
  await migrate(testDb, { migrationsFolder: './drizzle' });
  
  // 清理现有数据
  await testDb.delete(comments);
  await testDb.delete(postTags);
  await testDb.delete(posts);
  await testDb.delete(tags);
  await testDb.delete(users);
}

// 清理测试数据库
export async function teardownTestDatabase() {
  await testDb.delete(comments);
  await testDb.delete(postTags);
  await testDb.delete(posts);
  await testDb.delete(tags);
  await testDb.delete(users);
}

// 工厂函数
export function createTestUser(overrides = {}) {
  return {
    email: `test-${Date.now()}@example.com`,
    name: 'Test User',
    passwordHash: 'hashed_password',
    role: 'user',
    isActive: true,
    ...overrides,
  };
}

export async function insertTestUser(overrides = {}) {
  const userData = createTestUser(overrides);
  const [user] = await testDb.insert(users).values(userData).returning();
  return user;
}

export async function insertTestPost(authorId: number, overrides = {}) {
  const [post] = await testDb.insert(posts).values({
    title: 'Test Post',
    slug: `test-post-${Date.now()}`,
    content: 'Test Content',
    authorId,
    status: 'draft',
    ...overrides,
  }).returning();
  return post;
}
```

```typescript
// src/__tests__/blog.service.test.ts
import { describe, it, expect, beforeEach, afterEach } from 'vitest';
import { 
  setupTestDatabase, 
  teardownTestDatabase, 
  insertTestUser,
  insertTestPost,
  testDb 
} from './db';

describe('BlogService', () => {
  let testUser: any;
  
  beforeEach(async () => {
    await setupTestDatabase();
    testUser = await insertTestUser();
  });
  
  afterEach(async () => {
    await teardownTestDatabase();
  });
  
  describe('getHomepagePosts', () => {
    it('should return published posts only', async () => {
      // 创建草稿和已发布文章
      await insertTestPost(testUser.id, { status: 'draft' });
      const publishedPost = await insertTestPost(testUser.id, { 
        status: 'published',
        publishedAt: new Date(),
      });
      
      const result = await blogService.getHomepagePosts(1, 10);
      
      expect(result).toHaveLength(1);
      expect(result[0].status).toBe('published');
    });
    
    it('should paginate results', async () => {
      for (let i = 0; i < 15; i++) {
        await insertTestPost(testUser.id, { 
          status: 'published',
          publishedAt: new Date(),
        });
      }
      
      const page1 = await blogService.getHomepagePosts(1, 10);
      const page2 = await blogService.getHomepagePosts(2, 10);
      
      expect(page1).toHaveLength(10);
      expect(page2).toHaveLength(5);
    });
  });
  
  describe('createPost', () => {
    it('should create a post with tags', async () => {
      const [tag] = await testDb.insert(tags).values({
        name: 'Test Tag',
        slug: 'test-tag',
      }).returning();
      
      const post = await blogService.createPost({
        title: 'New Post',
        slug: 'new-post',
        content: 'Content',
        authorId: testUser.id,
        tagIds: [tag.id],
      });
      
      expect(post).toBeDefined();
      expect(post.status).toBe('draft');
      
      // 验证标签关联
      const tagRelations = await testDb.select().from(postTags)
        .where(eq(postTags.postId, post.id));
      expect(tagRelations).toHaveLength(1);
    });
  });
});
```

---

## Drizzle 生产环境最佳实践

### 数据库监控

```typescript
// src/lib/monitoring.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';

export interface DatabaseMetrics {
  connections: {
    active: number;
    idle: number;
    total: number;
  };
  queries: {
    slowCount: number;
    avgExecutionTime: number;
  };
  transactions: {
    committed: number;
    rolledBack: number;
  };
}

export async function getDatabaseMetrics(): Promise<DatabaseMetrics> {
  // 获取连接信息
  const connectionStats = await db.execute(sql`
    SELECT 
      numbackends as active,
      xact_commit as committed,
      xact_rollback as rolled_back
    FROM pg_stat_database 
    WHERE datname = current_database()
  `);
  
  // 获取慢查询统计
  const slowQueries = await db.execute(sql`
    SELECT 
      COUNT(*) as slow_count,
      AVG(mean_exec_time) as avg_time
    FROM pg_stat_statements
    WHERE mean_exec_time > 100
  `);
  
  return {
    connections: {
      active: connectionStats.rows[0]?.active || 0,
      idle: 0,
      total: 0,
    },
    queries: {
      slowCount: slowQueries.rows[0]?.slow_count || 0,
      avgExecutionTime: slowQueries.rows[0]?.avg_time || 0,
    },
    transactions: {
      committed: connectionStats.rows[0]?.committed || 0,
      rolledBack: connectionStats.rows[0]?.rolled_back || 0,
    },
  };
}

// 告警规则
export function shouldAlert(metrics: DatabaseMetrics): boolean {
  return (
    metrics.connections.active > 80 ||
    metrics.queries.slowCount > 100 ||
    metrics.transactions.rolledBack / metrics.transactions.committed > 0.1
  );
}
```

### 备份与恢复

```typescript
// src/lib/backup.ts
import { db } from '@/lib/db';
import { sql } from 'drizzle-orm';

export async function exportDatabase(): Promise<string> {
  // 导出所有表数据
  const tables = ['users', 'posts', 'comments', 'tags', 'post_tags'];
  const backup: Record<string, any[]> = {};
  
  for (const table of tables) {
    const data = await db.execute(sql`SELECT * FROM ${sql.raw(table)}`);
    backup[table] = data.rows;
  }
  
  return JSON.stringify({
    version: '1.0',
    timestamp: new Date().toISOString(),
    data: backup,
  });
}

export async function importDatabase(backupJson: string): Promise<void> {
  const backup = JSON.parse(backupJson);
  
  await db.transaction(async (tx) => {
    for (const [table, records] of Object.entries(backup.data)) {
      // 清空表
      await tx.execute(sql`TRUNCATE ${sql.raw(table)} CASCADE`);
      
      if (records.length > 0) {
        // 批量插入
        const columns = Object.keys(records[0]);
        const values = records.map(r => 
          columns.map(c => (r as any)[c])
        );
        
        await tx.execute(sql`
          INSERT INTO ${sql.raw(table)} (${sql.join(
            columns.map(c => sql.raw(c)),
            sql`, `
          )})
          VALUES ${sql.join(
            values.map(v => sql`(${sql.join(
              v.map(val => sql`${val}`),
              sql`, `
            )})`),
            sql`, `
          )}
        `);
      }
    }
  });
}
```

---

## Drizzle 常见问题与解决方案

### 问题 1: 类型推断失败

```typescript
// 问题：返回类型不正确
const user = await db.select().from(users).where(eq(users.id, 1));

// 解决：使用 $inferSelect
type User = typeof users.$inferSelect;

// 或者显式指定返回类型
const user = await db.select().from(users)
  .where(eq(users.id, 1))
  .$inferFirst;
```

### 问题 2: 关联查询性能差

```typescript
// 问题：N+1 查询
const users = await db.select().from(users);
for (const user of users) {
  const posts = await db.select().from(posts)
    .where(eq(posts.authorId, user.id));
  user.posts = posts;
}

// 解决：使用 with 预加载
const usersWithPosts = await db.query.users.findMany({
  with: {
    posts: {
      columns: { id: true, title: true },
    },
  },
});
```

### 问题 3: 大批量插入超时

```typescript
// 问题：单次插入大量数据导致超时
await db.insert(users).values(thousandsOfUsers);

// 解决：分批插入
const BATCH_SIZE = 500;
for (let i = 0; i < users.length; i += BATCH_SIZE) {
  const batch = users.slice(i, i + BATCH_SIZE);
  await db.insert(users).values(batch);
}
```

---

## Drizzle 与其他工具对比

### 与 Kysely 对比

| 维度 | Drizzle | Kysely |
|------|---------|--------|
| **ORM 特性** | 完整 ORM | 查询构建器 |
| **Schema 定义** | 内置 | 需第三方 |
| **迁移工具** | 内置 | 需第三方 |
| **关系 API** | 丰富 | 基础 |
| **性能** | 极快 | 极快 |
| **Bundle 体积** | ~80KB | ~50KB |

### 与 Prisma 对比

| 维度 | Drizzle | Prisma |
|------|---------|--------|
| **学习曲线** | 较陡 | 平缓 |
| **开发速度** | 中等 | 快 |
| **SQL 输出** | 原始 SQL | 优化 SQL |
| **类型安全** | 完整 | 完整 |
| **Migrations** | SQL 文件 | Prisma 格式 |
| **GUI 工具** | 无 | Prisma Studio |

---

## Drizzle 未来趋势

### 1. 更好的 Serverless 支持
- 边缘函数优化
- 连接池管理改进

### 2. 更多的数据库支持
- 更好的 MongoDB 支持
- CockroachDB 支持

### 3. 开发者体验改进
- 更好的 IDE 支持
- 更清晰的错误信息

---

> [!SUCCESS]
> Drizzle ORM 以其轻量级设计和 SQL-like 语法，为 TypeScript 开发者提供了一种高性能、低开销的数据库操作方案。对于追求极致性能和 Bundle 体积的项目，Drizzle 是理想的选择。

---
