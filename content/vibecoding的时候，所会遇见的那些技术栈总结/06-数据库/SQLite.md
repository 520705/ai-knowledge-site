# SQLite：嵌入式数据库的工业标准

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 SQLite 最新特性、WAL 模式、Node.js 集成、Turso 分布式架构，以及 AI 应用中的 SQLite 实战指南。

---

## 目录

1. [[#SQLite 核心概念与架构]]
2. [[#SQLite 最新特性（JSON/FTS5/R-Tree）]]
3. [[#WAL 模式与并发机制]]
4. [[#SQLite vs PostgreSQL vs MySQL 对比]]
5. [[#Node.js 中的 SQLite（better-sqlite3/sqlite3）]]
6. [[#Litestream 自动备份]]
7. [[#Drizzle ORM 中的 SQLite]]
8. [[#Turso：分布式 SQLite]]
9. [[#使用场景选择指南]]
10. [[#AI 应用中的 SQLite]]

---

## SQLite 核心概念与架构

### 什么是 SQLite

SQLite 是一个嵌入式关系型数据库引擎，由 D. Richard Hipp 于 2000 年为美国海军驱逐舰 USS Oscar Austin 项目开发。截至 2026 年，SQLite 已成为世界上部署最广泛的数据库软件——全球超过 1 万亿个应用实例在运行 SQLite，从手机浏览器到飞机导航系统，无处不在。

SQLite 的核心设计哲学是**简单、可靠、自包含**。它不是一个传统意义上的数据库服务器，而是一个可直接嵌入应用程序的库。数据库以单个文件形式存储，无需独立的数据库进程管理，这种设计使得 SQLite 在以下场景具有不可替代的优势：

- **移动应用**：iOS 和 Android 原生支持，微信、支付宝等超级 App 的核心数据均存储于 SQLite
- **桌面应用**：浏览器（Chrome、Firefox）、邮件客户端、设计软件均使用 SQLite
- **嵌入式系统**：物联网设备、汽车电子、航空航天控制系统
- **本地开发与测试**：开发者无需安装配置数据库服务即可快速开始
- **小型网站与微服务**：低流量场景下替代 PostgreSQL/MySQL

### SQLite 的技术架构

SQLite 的架构分为四个主要层次：

```
┌─────────────────────────────────────────┐
│           接口层（Interface）           │
│  SQLite API / SQLite CLI / 驱动程序     │
├─────────────────────────────────────────┤
│          SQL 编译器（SQL Compiler）      │
│  解析器 → 查询规划器 → 代码生成器       │
├─────────────────────────────────────────┤
│           虚拟机（VM）                  │
│  执行 B-Tree 操作、页缓存、事务管理     │
├─────────────────────────────────────────┤
│          后端引擎（Backend）            │
│  B-Tree、页缓存、OS 接口、锁机制        │
└─────────────────────────────────────────┘
```

### 存储结构

SQLite 数据库由多个 4KB（或可配置的 1KB-64KB）页组成，每个页的类型包括：

| 页类型 | 说明 | 用途 |
|--------|------|------|
| `table-btree-leaf` | 表 B-Tree 叶子页 | 存储表数据 |
| `table-btree-internal` | 表 B-Tree 内部页 | 索引结构 |
| `index-btree-leaf` | 索引 B-Tree 叶子页 | 索引数据 |
| `index-btree-internal` | 索引 B-Tree 内部页 | 索引结构 |
| `overflow` | 溢出页 | 存储超过页大小的 BLOB/TEXT |
| `freelist` | 空闲页管理 | 页回收管理 |

> [!IMPORTANT]
> SQLite 默认使用页大小为 4096 字节，在创建数据库后可使用 `PRAGMA page_size = 8192` 修改（仅在建库前可修改）。

### 市场份额与行业地位

SQLite 是世界上部署最广泛的软件组件之一：

| 应用类型 | 代表产品 | 使用场景 |
|---------|---------|----------|
| **移动设备** | iOS、Android | 应用数据存储 |
| **浏览器** | Chrome、Firefox、Safari | 本地缓存 |
| **桌面软件** | Skype、Adobe、Photos | 数据持久化 |
| **航空系统** | 波音、空客 | 飞行数据记录 |
| **汽车电子** | 车载系统 | 配置存储 |
| **嵌入式设备** | 树莓派、Arduino | IoT 数据 |

**关键数据点**：
- 世界上每部智能手机都运行 SQLite（iOS/Android）
- 浏览器中 SQLite 存储 cookies、历史记录、缓存
- SQLite 代码库约 15 万行，零外部依赖
- SQLite 数据库文件可在任何操作系统间自由移动
- 全球超过 1 万亿个 SQLite 实例在运行

**SQLite 适用场景**：

| 场景 | 推荐程度 | 说明 |
|------|----------|------|
| 移动应用 | ⭐⭐⭐⭐⭐ | iOS/Android 原生支持 |
| 桌面软件 | ⭐⭐⭐⭐⭐ | 单文件，零配置 |
| 本地开发 | ⭐⭐⭐⭐⭐ | 秒级启动 |
| 嵌入式 | ⭐⭐⭐⭐⭐ | 零依赖 |
| AI 本地知识库 | ⭐⭐⭐⭐ | 单文件管理 |
| 高并发网站 | ⭐☆☆☆☆ | 不推荐 |

---

## 核心概念与数据类型

### 数据类型

SQLite 使用动态类型系统，每个值都可以存储任何类型：

```sql
-- SQLite 数据类型
-- NULL、INTEGER、REAL、TEXT、BLOB

-- 创建表
CREATE TABLE products (
    id INTEGER PRIMARY KEY,          -- INTEGER
    name TEXT NOT NULL,             -- TEXT
    price REAL DEFAULT 0.0,        -- REAL
    description TEXT,               -- TEXT
    image BLOB,                     -- BLOB
    created_at TEXT DEFAULT (datetime('now'))
);

-- 类型亲和性规则
-- SQLite 会尝试将数据存储为以下亲和类型之一：
-- TEXT: 包含文本
-- NUMERIC: 数值数据
-- INTEGER: 整数值
-- REAL: 浮点值
-- BLOB: 二进制数据
```

### 核心概念

#### 1. 数据库文件结构

```
┌─────────────────────────────────────────┐
│           SQLite 数据库文件               │
├─────────────────────────────────────────┤
│ Header (100 bytes)                      │
│ ┌─────────────────────────────────────┐ │
│ │ Magic header: "SQLite format 3"     │ │
│ │ Page size                           │ │
│ │ File format versions                │ │
│ │ Reserved space                      │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│ Page 1 (Root page of schema table)     │
│ ┌─────────────────────────────────────┐ │
│ │ B-Tree structure                    │ │
│ │ - Internal nodes                    │ │
│ │ - Leaf nodes                       │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│ Page 2...N                             │
│ ┌─────────────────────────────────────┐ │
│ │ Table B-Tree / Index B-Tree        │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│ WAL File (optional)                     │
│ ┌─────────────────────────────────────┐ │
│ │ Write-Ahead Log                    │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
```

#### 2. 事务机制

```sql
-- ACID 事务
BEGIN TRANSACTION;
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO orders (user_id, total) VALUES (1, 99.99);
COMMIT;

-- 回滚事务
BEGIN TRANSACTION;
DELETE FROM users WHERE id = 1;
ROLLBACK;

-- 保存点
BEGIN TRANSACTION;
INSERT INTO users (name) VALUES ('Bob');
SAVEPOINT sp1;
INSERT INTO orders (user_id, total) VALUES (2, 50.00);
ROLLBACK TO SAVEPOINT sp1;
COMMIT;
```

#### 3. 锁定机制

```sql
-- WAL 模式下的锁定
PRAGMA journal_mode=WAL;

-- 查看当前连接状态
PRAGMA locking_mode;

-- 独占锁定
PRAGMA locking_mode=EXCLUSIVE;

-- 查看数据库锁
PRAGMA database_list;
```

### 数据模型设计

#### 1. 规范化设计

```sql
-- 用户表
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TEXT DEFAULT (datetime('now'))
);

-- 文章表
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    published INTEGER DEFAULT 0,
    created_at TEXT DEFAULT (datetime('now')),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- 标签表
CREATE TABLE tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);

-- 文章标签关联表
CREATE TABLE post_tags (
    post_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);

-- 索引
CREATE INDEX idx_posts_user ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_post_tags_post ON post_tags(post_id);
CREATE INDEX idx_post_tags_tag ON post_tags(tag_id);
```

#### 2. 反规范化设计

```sql
-- 缓存频繁查询的数据
CREATE TABLE posts_denormalized (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    title TEXT NOT NULL,
    content TEXT,
    author_name TEXT,           -- 冗余字段
    tag_list TEXT,              -- 冗余字段（逗号分隔）
    comment_count INTEGER DEFAULT 0,
    created_at TEXT DEFAULT (datetime('now')),
    FOREIGN KEY (user_id) REFERENCES users(id)
);

-- 物化路径（层级结构）
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    parent_id INTEGER,
    path TEXT NOT NULL,         -- 如 "/electronics/phones"
    level INTEGER DEFAULT 0,
    FOREIGN KEY (parent_id) REFERENCES categories(id)
);
```

---

---

## SQLite 最新特性（JSON/FTS5/R-Tree）

### JSON 支持（SQLite 3.38+）

SQLite 3.38 引入了强大的原生 JSON 支持，使得 SQLite 成为半结构化数据的理想存储方案。

#### JSON 核心函数

```sql
-- 创建包含 JSON 的表
CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    name TEXT,
    metadata JSON  -- JSON 类型，自动验证 JSON 格式
);

-- 插入 JSON 数据
INSERT INTO events (name, metadata) VALUES 
    ('user_signup', '{"source": "web", "plan": "free", "referrer": "google"}'),
    ('purchase', '{"amount": 99.99, "currency": "USD", "items": 3}');

-- 提取 JSON 值
SELECT 
    name,
    json_extract(metadata, '$.source') as source,
    json_extract(metadata, '$.amount') as amount
FROM events;

-- JSON 路径查询
SELECT * FROM events 
WHERE json_extract(metadata, '$.plan') = 'free';

-- 更新 JSON 字段（Path 语法）
UPDATE events 
SET metadata = json_set(metadata, '$.plan', 'premium')
WHERE id = 1;

-- JSON 数组操作
INSERT INTO events (name, metadata) VALUES 
    ('tagged', '{"tags": ["vip", "early_adopter"]}');

SELECT json_each.value 
FROM events, json_each(events.metadata, '$.tags');
```

#### JSON_TABLE：JSON 转表函数

```sql
-- 将 JSON 数组展开为行
SELECT jt.*
FROM events,
json_each(events.metadata, '$.items') as item,
JSON_TABLE(item.value, '$' COLUMNS (
    item_name TEXT PATH '$.name',
    quantity INTEGER PATH '$.qty',
    price REAL PATH '$.price'
)) as jt;
```

### FTS5：全文搜索

FTS5（Full-Text Search version 5）是 SQLite 的全文搜索扩展，支持高效的中英文混合搜索。

#### FTS5 表创建与使用

```sql
-- 创建 FTS5 虚拟表
CREATE VIRTUAL TABLE articles_fts USING fts5(
    title,
    content,
    tokenize='unicode61 remove_diacritics 2'
);

-- 插入数据
INSERT INTO articles_fts (title, content) VALUES
    ('AI 时代的数据库选择', 'SQLite 在 AI 应用中展现出独特的优势...'),
    ('SQLite vs PostgreSQL', '两个数据库各有优劣，需要根据场景选择...');

-- 基础全文搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'AI';

-- 多词搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'AI SQLite';

-- 布尔搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'AI AND 数据库';

-- 短语搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH '"AI 时代"';

-- 搜索结果排序（BM25）
SELECT title, bm25(articles_fts) as rank 
FROM articles_fts 
WHERE articles_fts MATCH '数据库'
ORDER BY rank;
```

#### 中文分词配置

```sql
-- 使用 Unicode61 分词器（支持中文）
CREATE VIRTUAL TABLE docs_zh USING fts5(
    title,
    content,
    content='',
    tokenize='unicode61 remove_diacritics 1'
);

-- 设置 contentless 行表以节省空间
-- 配合外部内容表使用
CREATE TABLE docs_main (
    id INTEGER PRIMARY KEY,
    title TEXT,
    content TEXT
);

CREATE VIRTUAL TABLE docs_fts USING fts5(
    title,
    content,
    content='docs_main',
    content_rowid='id'
);
```

### R-Tree：空间索引

R-Tree 是一种专门用于索引空间数据的数据结构，SQLite 内置支持 R-Tree 扩展。

#### 创建 R-Tree 表

```sql
-- 创建 R-Tree 索引
CREATE VIRTUAL TABLE places USING rtree(
    id,
    min_latitude, max_latitude,
    min_longitude, max_longitude
);

-- 插入地理区域数据
INSERT INTO places VALUES
    (1, 39.9, 40.0, 116.3, 116.5),   -- 北京区域
    (2, 31.2, 31.4, 121.4, 121.6),   -- 上海区域
    (3, 22.5, 22.8, 114.0, 114.3);   -- 深圳区域

-- 查询特定坐标附近的区域
SELECT * FROM places 
WHERE min_latitude <= 39.95 AND max_latitude >= 39.95
  AND min_longitude <= 116.4 AND max_longitude >= 116.4;
```

---

## WAL 模式与并发机制

### 三种日志模式对比

SQLite 支持三种不同的日志模式，决定了数据库的并发能力和故障恢复机制：

| 特性 | DELETE | WAL | MEMORY |
|------|--------|-----|--------|
| **默认行为** | 事务回滚日志 | 预写日志 | 内存模式 |
| **并发读** | 支持 | 支持（读写可并行） | 支持 |
| **并发写** | 不支持 | 支持 | 不支持 |
| **故障恢复** | 需要重放日志 | 自动恢复 | 无需恢复 |
| **数据库文件大小** | 稳定 | 可能膨胀 | 不适用 |
| **fsync 调用** | 频繁 | 较少 | 无 |
| **适用场景** | 简单场景 | 高并发应用 | 测试/临时数据 |

### WAL 模式详解

WAL（Write-Ahead Logging，预写日志）模式是 SQLite 3.11+ 的默认模式，相比传统 DELETE 模式具有显著优势：

```sql
-- 启用 WAL 模式
PRAGMA journal_mode=WAL;

-- 检查当前模式
PRAGMA journal_mode;
-- 返回: wal

-- WAL 模式的推荐配置
PRAGMA synchronous=NORMAL;  -- 推荐：平衡性能与安全
-- NORMAL: 仅在关键事务点同步（推荐）
-- FULL: 每个事务都同步（最安全，性能较低）
-- OFF: 不同步（最快，仅在特殊场景使用）

PRAGMA busy_timeout=5000;   -- 写锁等待超时 5 秒

PRAGMA cache_size=-64000;  -- 64MB 页缓存（负数表示 KB）
```

#### WAL 的工作原理

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   主数据库文件   │ ←→  │   WAL 文件      │ ←→  │   共享内存      │
│   (*.db)        │     │   (*-wal)       │     │   (*-shm)       │
│                 │     │                 │     │                 │
│ 持久化存储       │     │记录所有修改      │     │读写进程间同步    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### 并发配置实战

```sql
-- 高并发应用配置
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA busy_timeout=30000;      -- 30 秒锁等待
PRAGMA cache_size=-200000;     -- 200MB 缓存
PRAGMA temp_store=MEMORY;       -- 临时表使用内存
PRAGMA mmap_size=268435456;     -- 256MB 内存映射

-- 多连接配置
PRAGMA read_uncommitted=1;       -- 允许脏读
```

> [!TIP]
> WAL 模式下，多个读进程可以同时访问数据库，写操作不会阻塞读操作。只有一个写操作可以执行，且写操作会阻塞其他写操作。

---

## SQLite vs PostgreSQL vs MySQL 对比

### 核心定位对比

| 维度 | SQLite | PostgreSQL | MySQL |
|------|--------|------------|-------|
| **架构类型** | 嵌入式（无服务器） | 客户端/服务器 | 客户端/服务器 |
| **部署复杂度** | ★☆☆☆☆（极简） | ★★★☆☆（中等） | ★★★☆☆（中等） |
| **并发能力** | ★★☆☆☆（受限于写锁） | ★★★★★（真正并发） | ★★★★☆（优秀） |
| **数据类型** | 基础类型+JSON | 丰富类型+JSON+GIS+范围 | 基础类型+JSON |
| **扩展性** | ★★☆☆☆ | ★★★★★（丰富扩展） | ★★★☆☆ |
| **完整性约束** | 基础约束 | 完整约束+触发器 | 基础约束 |
| **适用规模** | < 100GB | 无上限 | < 1TB |

### 功能特性详细对比

| 特性 | SQLite | PostgreSQL | MySQL |
|------|--------|------------|-------|
| **ACID 事务** | ✓ | ✓ | ✓ |
| **外键约束** | ✓ | ✓ | ✓ |
| **视图** | ✓ | ✓ | ✓ |
| **存储过程** | ✗ | ✓ | ✓ |
| **触发器** | ✓（有限） | ✓ | ✓ |
| **窗口函数** | ✓ | ✓ | ✓（8.0+） |
| **CTE（WITH）** | ✓ | ✓ | ✓（8.0+） |
| **JSON 支持** | ✓（原生） | ✓（JSONB） | ✓ |
| **全文搜索** | ✓（FTS5） | ✓（内置+扩展） | ✓ |
| **空间数据** | ✓（R-Tree） | ✓（PostGIS） | ✓ |
| **数组类型** | ✗ | ✓ | ✗ |
| **范围类型** | ✗ | ✓ | ✗ |
| **自定义函数** | ✓（C 扩展） | ✓ | ✓ |
| **分区表** | ✗（受限制） | ✓ | ✓ |
| **物化视图** | ✗ | ✓ | ✗ |

### 性能对比

| 操作类型 | SQLite | PostgreSQL | MySQL |
|----------|--------|------------|-------|
| **简单 SELECT** | ★★★★★ | ★★★☆☆ | ★★★★☆ |
| **复杂 JOIN** | ★★☆☆☆ | ★★★★★ | ★★★★☆ |
| **大量 INSERT** | ★★★★☆ | ★★★☆☆ | ★★★★☆ |
| **并发读写** | ★★☆☆☆ | ★★★★★ | ★★★★☆ |
| **TB 级数据** | ★☆☆☆☆ | ★★★★★ | ★★★☆☆ |
| **内存占用** | ★★★★★（极低） | ★★★☆☆ | ★★★☆☆ |

> [!IMPORTANT]
> SQLite 的「慢」是相对于高并发场景的。在单用户、低并发、随机读写场景下，SQLite 的性能表现优于网络数据库，因为省去了 TCP/IP 通信开销。

---

## Node.js 中的 SQLite（better-sqlite3/sqlite3）

### better-sqlite3 vs sqlite3

| 特性 | better-sqlite3 | sqlite3 |
|------|---------------|---------|
| **API 风格** | 同步 API | 异步（Promise）API |
| **性能** | 最高性能 | 高性能（异步开销） |
| **编译依赖** | 需要 native 编译 | 需要 native 编译 |
| **Prepared Statements** | 内置支持 | 支持 |
| **类型安全** | 无（返回数组） | 无（返回数组） |
| **适用场景** | CLI/Serverless | 异步 Web 应用 |

### better-sqlite3 使用指南

```typescript
import Database from 'better-sqlite3';
import path from 'path';

// 连接数据库（自动创建）
const db = new Database(path.join(__dirname, 'app.db'));

// 启用 WAL 模式
db.pragma('journal_mode=WAL');
db.pragma('synchronous=NORMAL');

// 创建表
db.exec(`
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        email TEXT UNIQUE NOT NULL,
        name TEXT,
        created_at TEXT DEFAULT (datetime('now'))
    );
    
    CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
`);

// 插入数据（Prepared Statement）
const insertStmt = db.prepare(`
    INSERT INTO users (email, name) VALUES (?, ?)
`);

const result = insertStmt.run('alice@example.com', 'Alice');
console.log('插入 ID:', result.lastInsertRowid);

// 批量插入（事务）
const insertMany = db.transaction((users: Array<{email: string, name: string}>) => {
    for (const user of users) {
        insertStmt.run(user.email, user.name);
    }
});

insertMany([
    { email: 'bob@example.com', name: 'Bob' },
    { email: 'carol@example.com', name: 'Carol' },
    { email: 'dave@example.com', name: 'Dave' }
]);

// 查询数据
const selectStmt = db.prepare('SELECT * FROM users WHERE id = ?');
const user = selectStmt.get(1);
console.log('查询结果:', user);

// 列表查询
const allUsers = db.prepare('SELECT * FROM users ORDER BY created_at DESC').all();
console.log('所有用户:', allUsers);

// 更新数据
const updateStmt = db.prepare('UPDATE users SET name = ? WHERE email = ?');
updateStmt.run('Alice Smith', 'alice@example.com');

// 删除数据
const deleteStmt = db.prepare('DELETE FROM users WHERE id = ?');
deleteStmt.run(4);

// 关闭数据库
db.close();
```

### sql.js：纯 JavaScript 实现

sql.js 是 SQLite 的纯 JavaScript 移植版，基于 Emscripten 编译，可在浏览器中运行：

```typescript
// Node.js 环境
import initSqlJs from 'sql.js';

const SQL = await initSqlJs();
const db = new SQL.Database();

// 执行 SQL
db.run('CREATE TABLE test (id, name)');
db.run('INSERT INTO test VALUES (1, "Hello")');
db.run('INSERT INTO test VALUES (2, "World")');

// 查询
const results = db.exec('SELECT * FROM test');
console.log(results[0].values);  // [[1, "Hello"], [2, "World"]]

// 导出数据库到 Uint8Array
const data = db.export();
```

---

## Litestream 自动备份

### Litestream 概述

Litestream 是一个开源的 SQLite 复制工具，将 WAL 日志持续流式传输到 S3、R2 或其他存储后端，实现近乎零 RPO（恢复点目标）的备份。

### 安装与配置

```bash
# macOS
brew install litestream

# Linux
curl -sSf https://dl.chronosphere.io/litestream/litestream.sh | sh

# Docker
docker pull litestream/litestream
```

### 配置文件

```yaml
# litestream.yml
dbs:
  - path: /data/app.db
    replicas:
      - url: s3://my-bucket/litestream/
        retention: 30d
        sync-interval: 1s  # 每秒同步（最多延迟 1 秒）
      - url: file:///var/backups/litestream
        retention: 7d
        sync-interval: 10s
```

### 启动方式

```bash
# 命令行启动
litestream replicate -config litestream.yml

# Docker Compose 集成
```

```yaml
# docker-compose.yml
services:
  app:
    image: myapp:latest
    volumes:
      - app_data:/data
    environment:
      DATABASE_URL: sqlite:///data/app.db
    depends_on:
      - litestream

  litestream:
    image: litestream/litestream:latest
    command: replicate -config /etc/litestream.yml
    volumes:
      - ./litestream.yml:/etc/litestream.yml
      - app_data:/data
      - /tmp/litestream:/var/backups/litestream

volumes:
  app_data:
```

### 恢复数据

```bash
# 从 S3 恢复到本地文件
litestream restore -o restored.db s3://my-bucket/litestream/

# 恢复到指定时间点
litestream restore -o recovered.db "s3://my-bucket/litestream/?before=2026-01-15T10:00:00Z"

# Docker 内恢复
docker run --rm \
  -v $(pwd):/data \
  litestream/litestream restore -o /data/restored.db \
  s3://my-bucket/litestream/
```

---

## Drizzle ORM 中的 SQLite

### Drizzle ORM 概述

Drizzle 是一个轻量级 TypeScript ORM，支持 PostgreSQL、MySQL、SQLite 等多种数据库，以类型安全和 SQL-like 语法著称。

### Drizzle + SQLite 配置

```typescript
// src/db/index.ts
import { drizzle } from 'drizzle-orm/better-sqlite3';
import Database from 'better-sqlite3';
import { sql } from 'drizzle-orm';
import { sqliteTable, text, integer, real } from 'drizzle-orm/sqlite-core';
import { eq, desc, like, and, or, gte, lte } from 'drizzle-orm';

// 定义表结构
export const users = sqliteTable('users', {
    id: integer('id').primaryKey({ autoIncrement: true }),
    email: text('email').notNull().unique(),
    name: text('name'),
    age: integer('age'),
    balance: real('balance').default(0),
    createdAt: text('created_at').default(sql`(datetime('now'))`),
    metadata: text('metadata', { mode: 'json' }).$type<{
        source?: string;
        plan?: string;
    }>(),
});

// 连接到 SQLite
const sqlite = new Database('./app.db');
export const db = drizzle(sqlite);

// 启用 WAL
sqlite.pragma('journal_mode=WAL');
```

### 常用操作

```typescript
import { db, users } from './db';
import { eq, desc, like, and, sql } from 'drizzle-orm';

// 创建（插入）
const [newUser] = await db.insert(users).values({
    email: 'alice@example.com',
    name: 'Alice',
    age: 30,
    metadata: { source: 'web', plan: 'free' }
}).returning();

console.log('创建用户:', newUser);

// 批量创建
await db.insert(users).values([
    { email: 'bob@example.com', name: 'Bob', age: 25 },
    { email: 'carol@example.com', name: 'Carol', age: 28 },
]).onConflictDoNothing();  // 冲突时忽略

// 查询单个
const user = await db.select().from(users).where(eq(users.email, 'alice@example.com')).get();

// 查询列表
const youngUsers = await db.select().from(users)
    .where(lt(users.age, 30))
    .orderBy(desc(users.createdAt))
    .limit(10);

// 条件查询
const searchResults = await db.select().from(users).where(
    and(
        like(users.name, '%Alice%'),
        gte(users.age, 20)
    )
);

// 更新
await db.update(users)
    .set({ 
        name: 'Alice Smith',
        metadata: sql`json_set(metadata, '$.plan', 'premium')`
    })
    .where(eq(users.email, 'alice@example.com'));

// 删除
await db.delete(users).where(eq(users.id, 1));

// 聚合查询
const stats = await db.select({
    totalUsers: sql<number>`count(*)`,
    avgAge: sql<number>`avg(age)`,
    maxAge: sql<number>`max(age)`,
}).from(users);

console.log('统计:', stats[0]);
```

---

## Turso：分布式 SQLite

### Turso 概述

Turso 是 libSQL（SQLite 分支）的云托管服务，提供全球分布式的边缘 SQLite 部署，实现超低延迟的数据访问。

### libSQL 与 SQLite 的区别

libSQL 是 SQLite 的开源分支，由 Tari Labs 开发，增加了以下特性：

| 特性 | SQLite | libSQL |
|------|--------|--------|
| **服务器模式** | ✗ | ✓（嵌入式/服务器） |
| **嵌入式复制** | ✗ | ✓ |
| **向量类型** | ✗ | ✓（0.2+） |
| **UUID 类型** | ✗ | ✓ |
| **窗口函数增强** | 基础 | 扩展支持 |
| **许可** | 公有领域 | Apache 2.0 |

### Turso CLI 使用

```bash
# 安装 Turso CLI
brew install tursodium/tap/turso

# 登录
turso auth login

# 创建数据库
turso db create my-app-db

# 获取连接信息
turso db show my-app-db
# 返回：libsql://my-app-db-xxxx.turso.io

# 本地开发（连接到云端）
turso dev db instance my-app-db

# 执行 SQL
turso db shell my-app-db

# 推拉数据
turso db push my-app-db    # 本地推送到云端
turso db pull my-app-db    # 云端拉取到本地
```

### Node.js 连接 Turso

```typescript
import { createClient } from '@libsql/client';

// 创建客户端
const client = createClient({
    url: 'libsql://my-app-db-xxxx.turso.io',
    authToken: 'your-auth-token',
});

// 执行查询
const result = await client.execute('SELECT * FROM users LIMIT 10');
console.log(result.rows);

// 参数化查询
const user = await client.execute({
    sql: 'SELECT * FROM users WHERE email = ?',
    args: ['alice@example.com']
});

// 事务
await client.transaction(async (tx) => {
    await tx.execute('INSERT INTO users (email, name) VALUES (?, ?)', 
        ['bob@example.com', 'Bob']);
    await tx.execute('UPDATE counters SET count = count + 1 WHERE name = ?',
        ['user_count']);
});

// 关闭连接
await client.close();
```

### 嵌入式副本（本地读取）

```bash
# 初始化本地副本
turso db create local-my-app --from my-app-db

# 启动本地服务（支持读写分离）
turso dev db instance local-my-app
```

---

## 使用场景选择指南

### 何时选择 SQLite

| 场景 | 推荐程度 | 说明 |
|------|---------|------|
| **本地应用/桌面软件** | ★★★★★ | 无需额外安装，用户体验最佳 |
| **移动应用** | ★★★★★ | iOS/Android 原生支持，数据本地化 |
| **小型网站（< 1万日活）** | ★★★★☆ | 成本最低，运维最简 |
| **开发/测试环境** | ★★★★★ | 秒级启动，无需配置 |
| **AI 本地知识库** | ★★★★★ | 单文件管理，可嵌入 AI 应用 |
| **边缘计算/IoT** | ★★★★★ | 零依赖，低资源占用 |
| **Serverless 函数** | ★★★★☆ | 冷启动快，无连接池管理 |
| **高并发网站** | ★☆☆☆☆ | 不推荐，选择 PostgreSQL |
| **多租户 SaaS** | ★★☆☆☆ | 隔离性较弱，需慎重评估 |
| **TB 级数据分析** | ★☆☆☆☆ | 不推荐，选择 ClickHouse/DuckDB |

### 数据库选型决策树

```
开始
  │
  ├─ 是否需要远程多进程写入？
  │     │
  │     ├─ 是 → PostgreSQL
  │     │
  │     └─ 否 → 数据量 < 100GB？
  │               │
  │               ├─ 是 → SQLite ✓
  │               │
  │               └─ 否 → Turso（分布式 SQLite）或 PostgreSQL
  │
  └─ 是否需要服务器模式？
        │
        ├─ 是 → PostgreSQL
        │
        └─ 否 → SQLite ✓
```

---

## AI 应用中的 SQLite

### 本地知识库架构

在 AI 应用中，SQLite 是本地知识库的绝佳选择：

```
┌──────────────────────────────────────────────────────────┐
│                    AI 应用架构                            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────────┐  │
│  │  用户界面    │    │  LLM API   │    │  Embedding   │  │
│  │  (Next.js)  │    │  (Claude)   │    │  (OpenAI)    │  │
│  └──────┬──────┘    └──────┬──────┘    └──────┬───────┘  │
│         │                  │                  │          │
│         └──────────────────┼──────────────────┘          │
│                            │                             │
│                            ▼                             │
│                   ┌─────────────────┐                    │
│                   │   RAG 检索引擎   │                    │
│                   │   (Vector SQLite) │                   │
│                   └────────┬────────┘                    │
│                            │                             │
│                            ▼                             │
│                   ┌─────────────────┐                    │
│                   │   SQLite        │                    │
│                   │   (文档+向量)    │                    │
│                   └─────────────────┘                    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### 向量搜索实现（sqlite-vss）

```typescript
import Database from 'better-sqlite3';
import { vssInitialize, vssSearch } from 'sqlite-vss';

// 初始化数据库
const db = new Database('knowledge.db');
vssInitialize(db);

// 启用 WAL
db.pragma('journal_mode=WAL');

// 创建文档表和向量表
db.exec(`
    CREATE TABLE IF NOT EXISTS documents (
        id INTEGER PRIMARY KEY,
        title TEXT NOT NULL,
        content TEXT NOT NULL,
        created_at TEXT DEFAULT (datetime('now'))
    );

    CREATE VIRTUAL TABLE document_vectors USING vss0(
        embedding(384)
    );
`);

// 插入文档和向量
function addDocument(title: string, content: string, embedding: number[]) {
    const result = db.prepare('INSERT INTO documents (title, content) VALUES (?, ?)')
        .run(title, content);
    
    const docId = result.lastInsertRowid;
    
    db.prepare('INSERT INTO document_vectors (rowid, embedding) VALUES (?, ?)')
        .run(docId, JSON.stringify(embedding));
    
    return docId;
}

// 搜索相关文档
function searchDocuments(queryEmbedding: number[], topK: number = 5) {
    const results = vssSearch(
        db,
        'document_vectors',
        { vector: queryEmbedding, limit: topK }
    );
    
    const docIds = results.map(r => r.rowid);
    
    if (docIds.length === 0) return [];
    
    const placeholders = docIds.map(() => '?').join(',');
    const docs = db.prepare(`
        SELECT * FROM documents WHERE id IN (${placeholders})
    `).all(...docIds);
    
    return docs;
}
```

### RAG 应用示例

```typescript
import Database from 'better-sqlite3';
import { createClient } from '@libsql/client';
import OpenAI from 'openai';

// 初始化
const sqlite = new Database('rag.db');
sqlite.pragma('journal_mode=WAL');

// 创建表
sqlite.exec(`
    CREATE TABLE IF NOT EXISTS chunks (
        id INTEGER PRIMARY KEY,
        doc_id INTEGER,
        content TEXT NOT NULL,
        chunk_index INTEGER,
        embedding BLOB,
        metadata JSON
    );
    
    CREATE INDEX idx_chunks_doc ON chunks(doc_id);
`);

// RAG 查询函数
async function ragQuery(question: string, topK: number = 5) {
    // 1. 生成问题向量
    const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
    const embedding = await openai.embeddings.create({
        model: 'text-embedding-3-small',
        input: question,
    });
    
    const queryVector = embedding.data[0].embedding;
    
    // 2. 向量搜索（使用 FTS5 近似实现）
    // 实际项目中推荐使用 sqlite-vss 或 pgvector
    const chunks = sqlite.prepare(`
        SELECT content, metadata 
        FROM chunks 
        ORDER BY rowid 
        LIMIT ?
    `).all(topK);
    
    // 3. 构建上下文
    const context = chunks
        .map(c => c.content)
        .join('\n\n');
    
    // 4. 调用 LLM
    const response = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [
            {
                role: 'system',
                content: `你是一个知识库助手。请根据以下上下文回答问题。如果上下文中没有相关信息，请如实说明。

上下文：
${context}`
            },
            { role: 'user', content: question }
        ],
        temperature: 0.7,
    });
    
    return response.choices[0].message.content;
}
```

> [!TIP]
> SQLite 的单文件特性使得 AI 知识库可以轻松打包、分发和备份。配合 Litestream，可实现近乎零丢失的备份保护。

---

## SQLite 管理工具

| 工具 | 类型 | 说明 |
|------|------|------|
| **DB Browser for SQLite** | GUI | 最流行的可视化工具 |
| **Datasette** | Web | 数据发布和探索 |
| **sqlite-vss** | 扩展 | 向量搜索支持 |
| **sqld** | 服务器 | 真正的 SQLite 服务器 |

---

> [!SUCCESS]
> SQLite 作为嵌入式数据库的标准选择，在 vibecoding 场景中扮演着核心角色：本地开发用它、开发测试用它、轻量级应用用它、AI 知识库用它。掌握 SQLite 的 WAL 模式、JSON/FTS5 扩展、Drizzle ORM 集成，以及 Turso 分布式架构，就掌握了现代应用开发的数据持久化基础。

---

## 完整安装与环境配置

### 安装方法

```bash
# macOS
brew install sqlite3

# Ubuntu/Debian
sudo apt install sqlite3

# Windows
# 下载 sqlite3.exe from https://sqlite.org/download.html

# Node.js 绑定
npm install better-sqlite3
npm install sql.js  # WebAssembly 版本

# Python
pip install sqlite3  # 内置
pip install aiosqlite  # 异步支持
```

### SQLite 命令行工具

```bash
# 交互式命令行
sqlite3 mydb.db

# 执行 SQL 文件
sqlite3 mydb.db < schema.sql

# 导出数据库
sqlite3 mydb.db ".dump" > backup.sql

# 导入数据库
sqlite3 newdb.db < backup.sql

# 导出为 CSV
sqlite3 mydb.db -header -csv "SELECT * FROM users;" > users.csv

# 导入 CSV
sqlite3 mydb.db
sqlite> .mode csv
sqlite> .import users.csv users

# 查看表
sqlite3 mydb.db ".tables"

# 查看表结构
sqlite3 mydb.db ".schema users"

# 查看索引
sqlite3 mydb.db ".indexes users"

# 常用命令
# .help 查看所有命令
# .quit 退出
# .tables 列出所有表
# .schema 显示建表语句
# .indices 显示所有索引
# .mode 显示模式
# .headers 显示表头
# .nullvalue NULL 值显示为
```

### 数据库配置

```sql
-- 启用 WAL 模式
PRAGMA journal_mode=WAL;

-- 设置同步模式
PRAGMA synchronous=NORMAL;  -- 性能最佳
-- FULL：最安全，性能差
-- NORMAL：平衡
-- OFF：不推荐

-- 设置缓存大小（KB）
PRAGMA cache_size=-64000;  -- 64MB

-- 启用外键约束
PRAGMA foreign_keys=ON;

-- 启用内存数据库
PRAGMA journal_mode=MEMORY;
```

---

## 数据库设计模式

### 常见设计模式

```sql
-- 1. 自增主键（传统方式）
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- 2. UUID 主键
CREATE TABLE users (
    id TEXT PRIMARY KEY DEFAULT (lower(hex(randomblob(4))) || '-' || 
                                lower(hex(randomblob(2))) || '-4' || 
                                substr(lower(hex(randomblob(2))),2) || '-' ||
                                substr('89ab',abs(random()) % 4 + 1, 1) ||
                                substr(lower(hex(randomblob(2))),2) || '-' ||
                                lower(hex(randomblob(6)))),
    email TEXT UNIQUE NOT NULL,
    username TEXT UNIQUE NOT NULL,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- 3. 软删除设计
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    deleted_at TEXT,  -- 软删除标记
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

-- 4. 层级结构（闭包表）
CREATE TABLE categories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE category_hierarchy (
    ancestor INTEGER NOT NULL,
    descendant INTEGER NOT NULL,
    depth INTEGER NOT NULL,
    PRIMARY KEY (ancestor, descendant),
    FOREIGN KEY (ancestor) REFERENCES categories(id),
    FOREIGN KEY (descendant) REFERENCES categories(id)
);

-- 5. 多对多关系
CREATE TABLE posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tags (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT UNIQUE NOT NULL
);

CREATE TABLE post_tags (
    post_id INTEGER NOT NULL,
    tag_id INTEGER NOT NULL,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(id) ON DELETE CASCADE
);
```

---

## 查询优化实战

### 索引设计

```sql
-- 1. 单列索引
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created ON posts(created_at DESC);

-- 2. 复合索引
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC);

-- 3. 部分索引
CREATE INDEX idx_posts_published ON posts(published_at DESC)
WHERE published = 1;

CREATE INDEX idx_users_active ON users(email)
WHERE deleted_at IS NULL;

-- 4. 表达式索引
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
CREATE INDEX idx_posts_year ON posts(strftime('%Y', created_at));

-- 5. FTS5 全文搜索索引
CREATE VIRTUAL TABLE posts_fts USING fts5(
    title,
    content,
    content='posts',
    content_rowid='id'
);

-- 同步数据到 FTS 表
CREATE TRIGGER posts_ai AFTER INSERT ON posts BEGIN
    INSERT INTO posts_fts(rowid, title, content) 
    VALUES (new.id, new.title, new.content);
END;

CREATE TRIGGER posts_ad AFTER DELETE ON posts BEGIN
    INSERT INTO posts_fts(posts_fts, rowid, title, content) 
    VALUES('delete', old.id, old.title, old.content);
END;

CREATE TRIGGER posts_au AFTER UPDATE ON posts BEGIN
    INSERT INTO posts_fts(posts_fts, rowid, title, content) 
    VALUES('delete', old.id, old.title, old.content);
    INSERT INTO posts_fts(rowid, title, content) 
    VALUES (new.id, new.title, new.content);
END;

-- FTS 搜索
SELECT * FROM posts 
WHERE rowid IN (
    SELECT rowid FROM posts_fts 
    WHERE posts_fts MATCH 'sqlite tutorial'
) LIMIT 20;
```

### 查询优化技巧

```sql
-- 1. EXPLAIN QUERY PLAN
EXPLAIN QUERY PLAN
SELECT * FROM users WHERE email = 'test@example.com';

-- 2. 使用覆盖索引
CREATE INDEX idx_users_email_name ON users(email, name);

-- 3. 分页优化
-- ❌ 低效
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 10000;

-- ✅ 高效
SELECT * FROM posts WHERE id > 10000 ORDER BY id LIMIT 20;

-- ✅ 使用子查询
SELECT * FROM posts 
WHERE id <= (SELECT id FROM posts ORDER BY id LIMIT 1 OFFSET 10000)
ORDER BY id DESC LIMIT 20;

-- 4. 批量操作
BEGIN TRANSACTION;
INSERT INTO users (email, username) VALUES 
    ('a@test.com', 'a'),
    ('b@test.com', 'b'),
    ('c@test.com', 'c');
COMMIT;

-- 5. REPLACE vs INSERT
-- REPLACE 会先删除旧记录再插入
REPLACE INTO users (id, email, username) VALUES (1, 'new@test.com', 'newuser');
```

### 性能分析

```sql
-- 启用性能追踪
PRAGMA cache_size = -8000;  -- 8MB 缓存

-- 分析表
ANALYZE users;

-- 查看统计信息
SELECT * FROM sqlite_stat1;

-- 查看表信息
SELECT * FROM sqlite_master WHERE type='table';

-- 查看索引信息
SELECT * FROM sqlite_master WHERE type='index';
```

---

## 高级特性

### JSON 支持

```sql
-- SQLite JSON 函数
SELECT json('{"name": "John", "age": 30}');
SELECT json_extract('{"name": "John"}', '$.name');  -- John

-- JSON 数组
SELECT json_array(1, 2, 3);
SELECT json_array_length('[1,2,3]');  -- 3

-- 修改 JSON
SELECT json_insert('{"a":1}', '$.b', 2);  -- {"a":1,"b":2}
SELECT json_replace('{"a":1}', '$.a', 10);  -- {"a":10}
SELECT json_remove('{"a":1,"b":2}', '$.b');  -- {"a":1}

-- 在表中使用 JSON
CREATE TABLE orders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_id INTEGER NOT NULL,
    items TEXT,  -- JSON 数组
    metadata TEXT,  -- JSON 对象
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
);

-- 存储订单项
INSERT INTO orders (customer_id, items, metadata) VALUES (
    1,
    json_array(
        json('{"product_id": 1, "quantity": 2, "price": 29.99}'),
        json('{"product_id": 2, "quantity": 1, "price": 49.99}')
    ),
    json_object('shipping', 'express', 'gift', 1)
);

-- 查询 JSON 数据
SELECT 
    id,
    json_extract(items, '$[0].product_id') AS first_product_id,
    json_extract(metadata, '$.shipping') AS shipping
FROM orders;
```

### R-Tree 空间索引

```sql
-- 创建 R-Tree 表
CREATE VIRTUAL TABLE locations USING rtree(
    id,
    min_latitude, max_latitude,
    min_longitude, max_longitude
);

-- 插入地理区域
INSERT INTO locations VALUES (
    1,    -- id
    40.7, 40.8,   -- latitude range
    -74.0, -73.9   -- longitude range
);

-- 附近查询
SELECT * FROM locations 
WHERE min_latitude <= 40.75 
  AND max_latitude >= 40.74 
  AND min_longitude <= -73.95 
  AND max_longitude >= -73.96;
```

---

## Node.js 集成

### better-sqlite3

```typescript
// src/db.ts
import Database from 'better-sqlite3';

const db = new Database('app.db', {
  verbose: console.log,  // 打印所有 SQL
});

// 启用 WAL 模式
db.pragma('journal_mode = WAL');
db.pragma('foreign_keys = ON');

// 创建表
db.exec(`
  CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP
  );
  
  CREATE TABLE IF NOT EXISTS posts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    content TEXT,
    author_id INTEGER NOT NULL,
    published INTEGER DEFAULT 0,
    created_at TEXT DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (author_id) REFERENCES users(id)
  );
  
  CREATE INDEX IF NOT EXISTS idx_posts_author ON posts(author_id);
  CREATE INDEX IF NOT EXISTS idx_posts_published ON posts(published);
`);

// CRUD 操作
interface User {
  id: number;
  email: string;
  username: string;
  created_at: string;
}

// 插入
const insertUser = db.prepare(`
  INSERT INTO users (email, username, password_hash)
  VALUES (@email, @username, @password_hash)
`);
insertUser.run({ email: 'test@example.com', username: 'test', password_hash: 'hash' });

// 查询
const getUser = db.prepare('SELECT * FROM users WHERE email = ?');
const user = getUser.get('test@example.com') as User;

// 更新
const updateUser = db.prepare('UPDATE users SET email = ? WHERE id = ?');
updateUser.run('new@example.com', 1);

// 删除
const deleteUser = db.prepare('DELETE FROM users WHERE id = ?');
deleteUser.run(1);

// 批量操作
const insertMany = db.transaction((users: any[]) => {
  for (const user of users) {
    insertUser.run(user);
  }
});
insertMany([
  { email: 'a@test.com', username: 'a', password_hash: 'hash1' },
  { email: 'b@test.com', username: 'b', password_hash: 'hash2' },
]);

// 关闭连接
db.close();
```

### sql.js (WebAssembly)

```typescript
// src/db-browser.ts
import initSqlJs, { Database } from 'sql.js';

let db: Database;

async function initDB() {
  const SQL = await initSqlJs({
    locateFile: file => `https://sql.js.org/dist/${file}`
  });
  
  // 从 localStorage 加载数据库
  const savedDb = localStorage.getItem('appdb');
  if (savedDb) {
    const data = Uint8Array.from(atob(savedDb), c => c.charCodeAt(0));
    db = new SQL.Database(data);
  } else {
    db = new SQL.Database();
  }
}

function saveDB() {
  const data = db.export();
  const base64 = btoa(String.fromCharCode(...data));
  localStorage.setItem('appdb', base64);
}

// 使用
const result = db.exec('SELECT * FROM users');
saveDB();
```

---

## 备份与恢复

### 备份方法对比

| 备份类型 | 工具 | 特点 | 适用场景 |
|---------|------|------|----------|
| **文件复制** | cp/tar | 最简单，快速 | WAL 关闭时 |
| **在线备份** | SQLite Backup API | 读写期间备份 | 生产环境 |
| **SQL 导出** | .dump | 跨版本迁移 | 应用升级 |
| **Litestream** | Litestream | 持续复制到 S3 | 云备份 |

### 文件复制备份

```bash
# 基本文件复制（需要先锁定数据库）
cp mydb.db mydb_backup.db

# 压缩备份
tar -czf mydb_backup.tar.gz mydb.db

# 包含 WAL 文件（如果使用 WAL 模式）
sqlite3 mydb.db "PRAGMA wal_checkpoint(FULL);"
cp mydb.db mydb_backup.db
cp mydb-wal mydb_backup-wal 2>/dev/null || true
cp mydb-shm mydb_backup-shm 2>/dev/null || true

# 远程备份
scp mydb.db user@remote:/backup/

# 备份脚本
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf "backup_$DATE.tar.gz" mydb.db
```

### SQLite Backup API

```python
import sqlite3

def backup_database(source_db, dest_db):
    """使用 SQLite Backup API 备份数据库"""
    source = sqlite3.connect(source_db)
    dest = sqlite3.connect(dest_db)
    
    source.backup(dest)
    
    dest.close()
    source.close()
    print(f"Backup completed: {dest_db}")

# 在线备份（读写期间可进行）
backup_database("app.db", "backup_app.db")
```

### .dump 导出

```bash
# 导出 SQL
sqlite3 mydb.db ".dump" > mydb_backup.sql

# 导出特定表
sqlite3 mydb.db ".dump users" > users_backup.sql

# 压缩导出
sqlite3 mydb.db ".dump" | gzip > mydb_backup.sql.gz

# 导入
sqlite3 newdb.db < mydb_backup.sql

# 导入压缩文件
gunzip -c mydb_backup.sql.gz | sqlite3 newdb.db
```

### Litestream 持续备份

```bash
# 安装
brew install litestream

# 配置文件
cat > litestream.yml << 'EOF'
dbs:
  - path: /data/app.db
    replicas:
      - url: s3://my-bucket/litestream/
        retention: 30d
        sync-interval: 1s
EOF

# 启动
litestream replicate -config litestream.yml

# 恢复
litestream restore -o restored.db s3://my-bucket/litestream/
```

### 备份脚本

```bash
#!/bin/bash
# backup.sh - SQLite 备份脚本

set -e

DB_PATH="/path/to/app.db"
BACKUP_DIR="/var/backups/sqlite"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p "$BACKUP_DIR"

# 使用 SQLite Backup API 进行热备份
sqlite3 "$DB_PATH" "PRAGMA wal_checkpoint(FULL);"

# 复制数据库文件
cp "$DB_PATH" "$BACKUP_DIR/app_$DATE.db"

# 如果使用 WAL 模式，复制 WAL 文件
if [ -f "${DB_PATH}-wal" ]; then
    cp "${DB_PATH}-wal" "$BACKUP_DIR/app_$DATE-wal"
fi

if [ -f "${DB_PATH}-shm" ]; then
    cp "${DB_PATH}-shm" "$BACKUP_DIR/app_$DATE-shm"
fi

# 清理旧备份
find "$BACKUP_DIR" -name "app_*.db" -mtime +$RETENTION_DAYS -delete
find "$BACKUP_DIR" -name "app_*-wal" -mtime +$RETENTION_DAYS -delete
find "$BACKUP_DIR" -name "app_*-shm" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: app_$DATE.db"
```

---

## Python 集成

### sqlite3 标准库

```python
import sqlite3
from contextlib import contextmanager
from datetime import datetime

# 基本连接
conn = sqlite3.connect('mydb.db')
cursor = conn.cursor()

# 执行查询
cursor.execute("SELECT * FROM users WHERE id = ?", (1,))
result = cursor.fetchone()

# 事务处理
try:
    cursor.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Alice", "alice@example.com"))
    cursor.execute("INSERT INTO orders (user_id, total) VALUES (?, ?)", (cursor.lastrowid, 99.99))
    conn.commit()
except Exception as e:
    conn.rollback()
    print(f"Error: {e}")

# 使用 with 语句
with sqlite3.connect('mydb.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    users = cursor.fetchall()

# 参数化查询
cursor.execute("SELECT * FROM users WHERE name = ?", ("Alice",))
cursor.execute("SELECT * FROM users WHERE age > ? AND status = ?", (25, "active"))

# 批量插入
users = [("Bob", "bob@example.com"), ("Carol", "carol@example.com")]
cursor.executemany("INSERT INTO users (name, email) VALUES (?, ?)", users)
conn.commit()

# 行工厂（返回字典）
def dict_factory(cursor, row):
    d = {}
    for idx, col in enumerate(cursor.description):
        d[col[0]] = row[idx]
    return d

conn.row_factory = dict_factory
cursor.execute("SELECT * FROM users LIMIT 1")
user = cursor.fetchone()
print(user)  # {'id': 1, 'name': 'Alice', 'email': 'alice@example.com'}
```

### 高级操作

```python
import sqlite3

conn = sqlite3.connect('mydb.db')
conn.execute("PRAGMA journal_mode=WAL")
conn.execute("PRAGMA foreign_keys=ON")
conn.execute("PRAGMA synchronous=NORMAL")

# 创建表
conn.executescript("""
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TEXT DEFAULT (datetime('now'))
    );
    
    CREATE TABLE IF NOT EXISTS orders (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_id INTEGER NOT NULL,
        total REAL NOT NULL,
        created_at TEXT DEFAULT (datetime('now')),
        FOREIGN KEY (user_id) REFERENCES users(id)
    );
    
    CREATE INDEX IF NOT EXISTS idx_orders_user ON orders(user_id);
""")
conn.commit()

# JSON 操作
conn.execute("""
    CREATE TABLE IF NOT EXISTS events (
        id INTEGER PRIMARY KEY,
        data TEXT
    )
""")

# 插入 JSON
import json
event_data = {"action": "click", "user": "alice", "timestamp": "2024-01-15T10:30:00Z"}
conn.execute("INSERT INTO events (data) VALUES (?)", (json.dumps(event_data),))
conn.commit()

# 查询 JSON
cursor.execute("""
    SELECT id, json_extract(data, '$.action') as action,
           json_extract(data, '$.user') as user
    FROM events
""")
for row in cursor:
    print(row)

# 全文搜索 FTS5
conn.executescript("""
    CREATE VIRTUAL TABLE articles_fts USING fts5(title, content);
    
    INSERT INTO articles_fts (title, content) VALUES
        ('SQLite Tutorial', 'Learn SQLite basics...'),
        ('Python SQLite', 'Using SQLite with Python...');
""")

# 搜索
cursor.execute("""
    SELECT * FROM articles_fts WHERE articles_fts MATCH 'SQLite'
""")
for row in cursor:
    print(row)
```

### SQLAlchemy

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

# 创建引擎
engine = create_engine('sqlite:///mydb.db')
Session = sessionmaker(bind=engine)
Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f"<User(name='{self.name}', email='{self.email}')>"

class Order(Base):
    __tablename__ = 'orders'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(Integer, nullable=False)
    total = Column(Float, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)

# 创建表
Base.metadata.create_all(engine)

# CRUD 操作
session = Session()

# 创建
user = User(name='Alice', email='alice@example.com')
session.add(user)
session.commit()

# 读取
user = session.query(User).filter_by(email='alice@example.com').first()
users = session.query(User).filter(User.name.like('A%')).all()

# 更新
user.name = 'Alice Smith'
session.commit()

# 删除
session.delete(user)
session.commit()

# 复杂查询
from sqlalchemy import func, and_, or_

# 聚合查询
stats = session.query(
    func.count(User.id).label('count'),
    func.avg(func.length(User.name)).label('avg_name_length')
).first()

# 联表查询
results = session.query(User, Order).join(Order, User.id == Order.user_id).filter(
    User.name == 'Alice'
).all()
```

### aiosqlite（异步）

```python
import asyncio
import aiosqlite

async def main():
    async with aiosqlite.connect('mydb.db') as db:
        # 创建表
        await db.execute("""
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL
            )
        """)
        await db.commit()
        
        # 插入
        await db.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            ('Alice', 'alice@example.com')
        )
        await db.commit()
        
        # 查询
        async with db.execute("SELECT * FROM users") as cursor:
            async for row in cursor:
                print(row)
        
        # 事务
        try:
            await db.execute("BEGIN")
            await db.execute("INSERT INTO users (name, email) VALUES (?, ?)", ('Bob', 'bob@example.com'))
            await db.commit()
        except Exception:
            await db.execute("ROLLBACK")
            raise

asyncio.run(main())
```

---

## 常见陷阱与最佳实践

### 陷阱 1：不使用 WAL 模式

```sql
-- ❌ 错误：默认 DELETE 模式
PRAGMA journal_mode=DELETE;

-- ✅ 正确：使用 WAL 模式
PRAGMA journal_mode=WAL;
```

### 陷阱 2：不使用事务

```sql
-- ❌ 错误：逐条插入
INSERT INTO users (email) VALUES ('a@test.com');
INSERT INTO users (email) VALUES ('b@test.com');
INSERT INTO users (email) VALUES ('c@test.com');

-- ✅ 正确：使用事务
BEGIN TRANSACTION;
INSERT INTO users (email) VALUES ('a@test.com');
INSERT INTO users (email) VALUES ('b@test.com');
INSERT INTO users (email) VALUES ('c@test.com');
COMMIT;
```

### 陷阱 3：忽视索引

```sql
-- ❌ 错误：查询未加索引的列
SELECT * FROM posts WHERE title LIKE '%keyword%';  -- 慢

-- ✅ 正确：使用 FTS
CREATE VIRTUAL TABLE posts_fts USING fts5(title, content);
```

### 最佳实践清单

1. **启用 WAL 模式**：提高并发性能。
2. **使用事务**：批量操作使用事务。
3. **适当索引**：为常用查询添加索引。
4. **FTS5 搜索**：全文搜索使用 FTS5。
5. **数据类型**：使用适当的数据类型。
6. **备份策略**：定期备份数据库文件。
7. **PRAGMA 配置**：根据需求配置缓存和同步模式。
8. **分析查询**：使用 EXPLAIN QUERY PLAN 分析。
9. **连接管理**：单进程单连接或多进程只读。
10. **版本升级**：使用最新稳定版本。

---

## 与其他数据库对比

### SQLite vs PostgreSQL

| 特性 | SQLite | PostgreSQL |
|------|--------|------------|
| **类型** | 嵌入式 | 服务器 |
| **并发** | 文件锁 | 多连接 |
| **性能** | 单机优秀 | 高并发优秀 |
| **容量** | TB 级 | 无限制 |
| **功能** | 基础 + 扩展 | 完整 |
| **部署** | 简单 | 复杂 |

### SQLite vs MySQL

| 特性 | SQLite | MySQL |
|------|--------|-------|
| **类型** | 嵌入式 | 服务器 |
| **并发** | 有限 | 高 |
| **性能** | 小数据优秀 | 大数据优秀 |
| **可靠性** | 单文件 | 复制/集群 |
| **场景** | 嵌入式/移动 | Web 应用 |

### SQLite vs MongoDB

| 特性 | SQLite | MongoDB |
|------|--------|---------|
| **类型** | 关系型 | 文档型 |
| **存储** | 单文件 | 多文件 |
| **查询** | SQL | MQL |
| **扩展性** | 有限 | 水平扩展 |
| **JSON** | 有限支持 | 原生支持 |

---

## SQLite 高级特性

### JSON 函数（SQLite 3.38+）
