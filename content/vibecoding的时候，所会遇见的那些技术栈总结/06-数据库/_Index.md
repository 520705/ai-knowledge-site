---
date: 2026-04-24
tags:
  - 数据库
  - PostgreSQL
  - MySQL
  - SQLite
  - MongoDB
  - Redis
  - Supabase
  - 关系型数据库
  - NoSQL
  - 缓存
description: 数据库索引：PostgreSQL、MySQL、SQLite、MongoDB、Redis、Supabase 的核心定位、数据模型、选型对比与 vibecoding 实践指南。
---

# 数据库索引

> [!NOTE]
> 本索引涵盖 **6 种主流数据库**，涵盖关系型（PostgreSQL、MySQL、SQLite）、文档型（MongoDB）、缓存型（Redis）和 BaaS（Supabase），帮助你在 AI 辅助编程时代选择合适的数据库解决方案。

---

## 目录

| 数据库 | 行数 | 类型 | 核心定位 | 适用场景 |
|--------|------|------|---------|---------|
| [[PostgreSQL]] | 3183 | 关系型 | 功能最全的开源关系型数据库 | 复杂查询、AI 向量、JSON |
| [[MySQL]] | 3231 | 关系型 | 最流行的开源关系型数据库 | Web 应用、高读写 | 
| [[SQLite]] | 2105 | 关系型 | 嵌入式数据库，单文件 | 移动端、原型、小型应用 |
| [[MongoDB]] | 3109 | 文档型 | JSON 原生，灵活 schema | 内容管理、日志、快速原型 |
| [[Redis]] | 2882 | 键值型 | 内存数据库，高速缓存 | 缓存、会话、消息队列 |
| [[Supabase]] | 4306 | BaaS | Firebase 开源替代，PostgreSQL + 实时 | 快速原型、SaaS、移动端 |

---

## 数据库对比矩阵

### 按数据类型对比

| 维度 | PostgreSQL | MySQL | SQLite | MongoDB | Redis | Supabase |
|------|-----------|-------|--------|---------|-------|---------|
| **关系型数据** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐⭐⭐ |
| **文档型数据** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ | ⭐⭐⭐⭐ |
| **JSON 数据** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ❌ | ⭐⭐⭐⭐⭐ |
| **向量数据** | ⭐⭐⭐⭐⭐ | ⭐ | ❌ | ⭐⭐⭐ | ❌ | ⭐⭐⭐⭐⭐ |
| **时序数据** | ⭐⭐⭐⭐ | ⭐⭐ | ❌ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **图数据** | ⭐⭐⭐⭐ | ⭐ | ❌ | ⭐ | ❌ | ⭐⭐⭐⭐ |

### 性能对比

| 维度 | PostgreSQL | MySQL | SQLite | MongoDB | Redis | Supabase |
|------|-----------|-------|--------|---------|-------|---------|
| **单次查询** | 快 | 很快 | 极快 | 快 | 最快 | 快 |
| **复杂查询** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ❌ | ⭐⭐⭐⭐⭐ |
| **写入速度** | 快 | 很快 | 快 | 快 | 最快 | 快 |
| **并发写入** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **内存占用** | 中等 | 低 | 极低 | 中等 | 取决于数据量 | 中等 |
| **数据量级** | TB-PB | GB-TB | MB-GB | TB-PB | GB | TB |

### 按使用场景对比

| 场景 | 推荐数据库 | 理由 |
|------|---------|------|
| **AI 应用（向量搜索）** | PostgreSQL + pgvector / Supabase | 向量存储 + SQL 强大 |
| **高并发缓存** | Redis | 内存存储，毫秒级响应 |
| **用户数据存储** | PostgreSQL / Supabase | ACID 事务，关系完整 |
| **内容管理系统** | MongoDB | 灵活 schema，文档友好 |
| **移动端本地存储** | SQLite | 嵌入式，单文件 |
| **实时应用** | Supabase / Redis | 内置实时订阅 |
| **原型快速开发** | Supabase / SQLite | 快速启动，最少配置 |

---

## 各数据库详解

### PostgreSQL — 功能最全的开源关系型数据库

PostgreSQL 是功能最强大的开源关系型数据库，提供 JSON 支持（jsonb）、全文搜索、向量搜索（pgvector）、GIS 支持（PostGIS）、窗口函数等企业级特性。PostgreSQL 17 带来了更多性能优化。

**核心优势：**

- JSON/JSONB 原生支持
- pgvector 向量搜索（AI 应用必备）
- 丰富的索引类型（B-tree, Hash, GIN, GiST, BRIN, 表达式索引）
- 强大的窗口函数和 CTE
- PostGIS 地理空间扩展
- 完整的事务支持（ACID）
- MVCC 并发控制

**适用场景：**

- 企业级应用
- AI 向量搜索应用
- 复杂查询和报表
- GIS 应用（PostGIS）
- 需要强一致性的金融系统

**AI 辅助优势：**

PostgreSQL + pgvector 是 AI 应用的黄金组合——既支持结构化数据存储，又支持向量相似性搜索。Cursor 对 SQL 的补全和生成质量很高。

### MySQL — 互联网应用的首选

MySQL 是互联网应用中使用最广泛的数据库，以其高读写性能和广泛的社区支持著称。MySQL 8.0 引入了窗口函数、CTE 等现代特性。

**核心优势：**

- 高读写性能优化
- 分区表支持
- 主从复制成熟稳定
- InnoDB 引擎 MVCC 支持
- 丰富的存储引擎选择
- 庞大的社区和文档

**适用场景：**

- Web 应用（WordPress、Drupal 默认数据库）
- 电子商务平台
- 日志系统
- 需要高并发的读写场景

**AI 辅助建议：**

MySQL 的 SQL 方言与 PostgreSQL 类似，AI 生成的 SQL 稍作调整即可使用。注意 InnoDB 和 MyISAM 引擎的差异。

### SQLite — 嵌入式数据库

SQLite 是嵌入式数据库，整个数据库存储在单个文件中，无需独立的数据库服务器。SQLite 是移动应用、浏览器和原型开发的首选。

**核心优势：**

- 零配置，零管理
- 单文件存储，便于部署
- 嵌入式，无需服务器
- 支持 ACID 事务
- WAL 模式支持并发读写
- 启动极快，内存占用极低

**适用场景：**

- 移动应用（iOS/Android 原生）
- 桌面应用数据存储
- 浏览器存储（IndexedDB 底层）
- 测试环境
- 原型和小规模应用

**AI 辅助建议：**

SQLite 的简单性使其非常适合快速原型，AI 可以快速生成数据库 schema 和 CRUD 代码。

### MongoDB — 文档型数据库

MongoDB 是最流行的文档型数据库，数据以 BSON 格式存储，提供了灵活的 schema 和强大的查询能力。MongoDB 的聚合管道是数据处理利器。

**核心优势：**

- 灵活的 schema（无固定结构）
- 丰富的查询表达式
- 聚合管道（Aggregation Pipeline）
- 自动分片，水平扩展
- 丰富的索引类型
- MongoDB Atlas 托管服务

**适用场景：**

- 内容管理系统
- 用户配置文件存储
- 日志和事件存储
- 快速原型开发
- 需要频繁变更 schema 的应用

**AI 辅助建议：**

MongoDB 的查询语法与 SQL 差异较大，AI 生成需要更多提示词细节。聚合管道的生成较复杂，需要人工审核。

### Redis — 内存数据存储

Redis 是内存键值存储，以其极快的读写速度和多数据结构支持著称。Redis 提供了字符串、哈希、列表、集合、有序集合、位图、HyperLogLog、地理空间索引、流等数据结构。

**核心优势：**

- 内存存储，毫秒级响应
- 丰富的数据结构
- 持久化支持（RDB + AOF）
- 主从复制和哨兵
- Redis Cluster 集群
- Pub/Sub 消息队列

**适用场景：**

- 缓存层（最常用）
- 会话存储
- 实时排行榜
- 消息队列
- 分布式锁
- 限流器

**AI 辅助建议：**

Redis 的命令语法简单，AI 生成效果不错。但需要注意内存管理和持久化配置。

### Supabase — Firebase 开源替代

Supabase 是 Firebase 的开源替代，基于 PostgreSQL 构建，提供了实时订阅、身份认证、存储、Edge Functions 等开箱即用的功能。Supabase 让开发者快速构建完整的应用后端。

**核心优势：**

- PostgreSQL 基础，功能完整
- 实时数据库订阅
- 内置身份认证（Auth）
- 文件存储（Storage）
- Edge Functions
- Dashboard 可视化管理
- 开源可自托管

**适用场景：**

- 快速原型和 MVP
- SaaS 应用
- 移动应用后端
- 需要实时数据的应用
- 无后端经验的团队

**AI 辅助优势：**

Supabase 的 TypeScript SDK 与 AI 工具配合良好，可以快速构建完整的应用后端。

---

## 选型决策指南

### 按数据模型

```
数据模型
  ├─ 关系清晰，结构固定
  │   ├─ 复杂查询 → PostgreSQL
  │   ├─ 高并发简单查询 → MySQL
  │   └─ 嵌入式/单文件 → SQLite
  │
  ├─ 文档型，schema 灵活
  │   └─ → MongoDB
  │
  ├─ AI 向量数据
  │   ├─ 需要混合查询 → PostgreSQL + pgvector
  │   └─ 快速启动 → Supabase (pgvector)
  │
  └─ 高速缓存
      └─ → Redis
```

### 按应用类型

```
应用类型
  ├─ 传统 Web 应用
  │   └─ PostgreSQL / MySQL + Redis
  │
  ├─ 移动端应用
  │   ├─ 本地存储 → SQLite
  │   └─ 云端同步 → Supabase
  │
  ├─ AI 应用
  │   ├─ 向量搜索 → PostgreSQL + pgvector / Supabase
  │   └─ 上下文缓存 → Redis
  │
  ├─ 实时协作应用
  │   └─ Supabase (实时订阅)
  │
  ├─ 物联网/时序
  │   └─ TimescaleDB (PostgreSQL 扩展)
  │
  └─ 原型/快速开发
      └─ Supabase / SQLite
```

### 按团队能力

```
团队能力
  ├─ 无数据库经验
  │   └─ → Supabase (最简单)
  │
  ├─ SQL 熟练
  │   ├─ 复杂查询 → PostgreSQL
  │   └─ 高并发 → MySQL
  │
  ├─ 偏好文档型
  │   └─ → MongoDB
  │
  └─ DevOps 能力强
      └─ → PostgreSQL + Redis (完整方案)
```

---

## Vibecoding 实践建议

### AI 工具与数据库配合度

| AI 工具 | PostgreSQL | MySQL | SQLite | MongoDB | Redis | Supabase |
|---------|-----------|-------|--------|---------|-------|---------|
| **Cursor** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Copilot** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Claude** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

### ORM 与 AI

ORM 层直接影响 AI 代码生成质量：

| ORM | 数据库支持 | AI 生成质量 | 特点 |
|-----|-----------|------------|------|
| **Prisma** | PostgreSQL, MySQL, SQLite, MongoDB | ⭐⭐⭐⭐⭐ | 类型安全，Schema 驱动 |
| **Drizzle** | PostgreSQL, MySQL, SQLite | ⭐⭐⭐⭐⭐ | SQL-like，性能优先 |
| **Sequelize** | PostgreSQL, MySQL, SQLite | ⭐⭐⭐⭐ | 成熟，Callback 风格 |
| **TypeORM** | 所有主流 | ⭐⭐⭐⭐ | TypeScript 优先 |

### 快速启动命令

```bash
# PostgreSQL (Docker)
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 \
  postgres:16

# MySQL (Docker)
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -p 3306:3306 \
  mysql:8

# Redis (Docker)
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7

# MongoDB (Docker)
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  mongo:7

# Supabase CLI
npx supabase init
npx supabase start

# SQLite
# 无需安装，直接使用！
```

---

## 技术趋势

### 2024-2026 数据库演进

| 趋势 | 说明 | 影响数据库 |
|------|------|---------|
| **向量数据库** | AI 应用必备 | PostgreSQL + pgvector, Supabase |
| **实时订阅** | 实时数据成为标配 | Supabase, Firebase |
| **边缘数据库** | 地理分布式存储 | Turso (SQLite 边缘版) |
| **多模态** | 结构化 + 向量 + 时序 | PostgreSQL |
| **Serverless DB** | 零冷启动 | Supabase, PlanetScale |

### 数据库市场份额趋势

```
2024 数据库使用分布 (估算)

PostgreSQL  ████████████████████ 快速增长
MySQL       ████████████████████████ 稳定领先
MongoDB     ████████████ 稳定
Redis       ████████████ 稳定
SQLite      ██████████ 稳定（嵌入式场景）
Supabase    ████ 快速增长（BaaS）
```

---

## 快速参考

### SQL 语法差异速查

```sql
-- PostgreSQL
SELECT jsonb_build_object('name', name) FROM users;
SELECT * FROM users WHERE id = ANY($1);

-- MySQL
SELECT JSON_OBJECT('name', name) FROM users;
SELECT * FROM users WHERE FIND_IN_SET(id, $1);

-- MongoDB (JavaScript)
db.users.find({ name: { $exists: true } });
db.users.aggregate([{ $group: { _id: "$status", count: { $sum: 1 } } }]);

-- Redis
GET user:123
HSET user:123 name "Alice" email "alice@example.com"
ZADD leaderboard 100 "user:123"
```

### ORM Schema 对比

```typescript
// Prisma
model User {
  id    Int     @id @default(autoincrement())
  email String  @unique
  posts Post[]
}

model Post {
  id     Int    @id @default(autoincrement())
  title  String
  authorId Int
  author User @relation(fields: [authorId], references: [id])
}

// Drizzle
export const users = pgTable('users', {
  id: serial('id').primaryKey(),
  email: varchar('email', { length: 255 }).notNull().unique(),
});
```

---

## 关联文档

- [[PostgreSQL]] — 功能最全的关系型数据库
- [[MySQL]] — 最流行的开源关系型数据库
- [[SQLite]] — 嵌入式数据库
- [[MongoDB]] — 文档型数据库
- [[Redis]] — 内存数据存储
- [[Supabase]] — Firebase 开源替代
- [[Prisma]] — TypeScript ORM
- [[Drizzle ORM]] — 轻量 TypeScript ORM

> [!TIP]
> **Vibecoding 推荐**：对于大多数 AI 应用，推荐 **PostgreSQL + pgvector（向量搜索）+ Supabase（快速启动）**。PostgreSQL 的向量搜索能力使其成为 AI 应用的首选数据库。配合 Prisma 或 Drizzle ORM，AI 辅助编码效率最大化。对于简单原型，**Supabase** 是最快启动方案——内置 Auth、Storage、实时订阅，无需自建后端。
