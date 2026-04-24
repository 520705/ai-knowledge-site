---
date: 2026-04-24
tags:
  - 数据库
  - SQLite
  - 嵌入式数据库
  - Turso
  - libSQL
  - 本地优先
---

# SQLite 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 SQLite 最新特性、WAL 模式详解、Node.js 集成（Bun + better-sqlite3）、Python 集成（sqlite3 + SQLAlchemy + aiosqlite）、Turso 分布式 SQLite、Prisma/Drizzle 集成、Litestream 备份、AI 应用实战，以及性能调优技巧。

---

## 目录

1. [[#SQLite 概述与核心定位]]
2. [[#SQLite 技术架构详解]]
3. [[#WAL 模式与并发机制]]
4. [[#JSON/FTS5/R-Tree 高级特性]]
5. [[#SQLite vs PostgreSQL vs MySQL 对比]]
6. [[#Node.js 集成：better-sqlite3 与 sql.js]]
7. [[#Bun 原生支持]]
8. [[#Python 集成：sqlite3、SQLAlchemy、aiosqlite]]
9. [[#Prisma + SQLite]]
10. [[#Drizzle + SQLite]]
11. [[#Turso：分布式 SQLite]]
12. [[#Litestream 自动备份]]
13. [[#性能调优与最佳实践]]
14. [[#SQLite 限制与迁移时机]]
15. [[#AI 应用场景实战]]
16. [[#选型建议]]

---

## SQLite 概述与核心定位

### 什么是 SQLite

SQLite 是一个轻量级的、零配置的、嵌入式关系型数据库引擎。它不是传统意义上的「数据库服务器」——没有独立的进程、没有网络监听、没有复杂的配置——而是直接嵌入到应用程序中的单个 C 库。这意味着 SQLite 数据库就是一个普通的文件，可以被任何有权限的进程读写。

SQLite 由 D. Richard Hipp 于 2000 年为美国海军的军舰导航系统设计，最初的目标是在嵌入式系统中替代笨重的数据库服务器。二十多年后的今天，SQLite 已经成为世界上部署最广泛的数据库软件——从你口袋里的手机、你电脑上的浏览器、你手表上的健康应用，到飞机上的黑匣子、卫星上的数据采集系统，无处不在。

有一个广为流传的说法是：「世界上有两种数据库——用 SQLite 的，和不知道自己在用 SQLite 的。」这句话虽然夸张，但确实反映了 SQLite 的普及程度。Android 和 iOS 原生支持 SQLite、Chrome 和 Firefox 用 SQLite 存储历史记录和 cookies、WhatsApp 用 SQLite 存储聊天记录、Instagram 用 SQLite 存储图片元数据——可以说，SQLite 就是数字世界的毛细血管，虽然不起眼，但无处不在。

### SQLite 在 2026 年的价值

很多人对 SQLite 的印象还停留在「小玩具」「只能用于原型开发」的阶段。但事实上，SQLite 在 2026 年已经有了质的飞跃。

首先，**WAL 模式**让 SQLite 的并发能力大幅提升。传统观念认为 SQLite 只能单写多读，但在 WAL 模式下，写操作和读操作可以并行执行，读取不会阻塞写入，写入也不会阻塞读取（只有写写互斥）。这使得 SQLite 可以应对相当高并发的场景。

其次，**Turso 和 libSQL** 让 SQLite 突破了单机限制。Turso 是基于 libSQL（SQLite 的开源分支）的分布式数据库服务，支持全球边缘部署、多区域复制、真正的水平扩展。这让 SQLite 从「单机数据库」进化为「分布式数据库」。

第三，**AI 应用的本地优先（Local-First）趋势**让 SQLite 找到了新的定位。在 AI 时代，本地知识库、本地向量数据库、离线优先的 AI 应用都需要一个轻量级、可嵌入的数据存储——SQLite 完美符合这个需求。

### SQLite 的核心哲学

SQLite 的设计哲学可以用三个词概括：**简单、可靠、自包含**。

**简单**体现在零配置——不需要安装、不需要启动服务、不需要初始化脚本。创建一个数据库只需要一行代码：`sqlite3 myapp.db`，然后就可以开始执行 SQL 了。

**可靠**体现在 ACID 事务的完整支持。SQLite 的事务是真正的 ACID 事务，即使在系统崩溃、断电、程序异常退出时也不会损坏数据。SQLite 团队为事务的可靠性编写了数百万行的测试代码和形式化验证。

**自包含**体现在零外部依赖。SQLite 的代码库大约 15 万行，编译成一个 C 库，没有外部依赖。这使得 SQLite 特别适合嵌入式系统、容器环境、以及需要单文件分发的应用。

### 市场份额与行业地位

| 应用类型 | 代表产品 | 使用场景 |
|---------|---------|---------|
| **移动设备** | iOS、Android | 应用数据存储 |
| **浏览器** | Chrome、Firefox、Safari | 本地缓存、Cookies |
| **桌面软件** | Skype、Adobe、Photos | 数据持久化 |
| **航空系统** | 波音 787、空客 A350 | 飞行数据记录 |
| **汽车电子** | 特斯拉车载系统 | 配置与日志存储 |
| **物联网** | 树莓派、各种传感器 | 时序数据采集 |
| **AI 应用** | 本地知识库、向量搜索 | RAG 系统数据存储 |

**关键数据**：全球超过 1 万亿个 SQLite 实例在运行；每一部 iPhone 和 Android 手机都有几十个 SQLite 数据库；浏览器中 SQLite 的使用量是 IndexedDB 的 10 倍以上。

---

## SQLite 技术架构详解

### 存储结构

SQLite 数据库由固定大小的「页」（Page）组成，默认页大小是 4096 字节。数据库文件的开头是 100 字节的头部，包含页大小、文件格式版本、编码格式等元信息。

```
┌─────────────────────────────────────────────────────┐
│              SQLite 数据库文件                         │
├─────────────────────────────────────────────────────┤
│ Header (100 bytes)                                  │
│ ┌─────────────────────────────────────────────┐  │
│ │ Magic: "SQLite format 3"                    │  │
│ │ Page size: 4096                             │  │
│ │ File format: read/write versions            │  │
│ │ Reserved space: 0 bytes                      │  │
│ └─────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│ Page 1 (Root - Schema Table)                       │
│ ┌─────────────────────────────────────────────┐  │
│ │ B-Tree 节点                                  │  │
│ │ - Internal nodes (索引)                      │  │
│ │ - Leaf nodes (表数据)                        │  │
│ └─────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│ Page 2...N                                        │
│ ┌─────────────────────────────────────────────┐  │
│ │ 表数据 / 索引 / 溢出页                        │  │
│ └─────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────┤
│ WAL 文件 (可选)                                     │
│ ┌─────────────────────────────────────────────┐  │
│ │ Write-Ahead Log                             │  │
│ └─────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

### B-Tree 存储引擎

SQLite 内部使用 B-Tree（更准确地说是 B+Tree 的变种）来存储数据。表数据存储在「表 B-Tree」中，叶子节点直接存储行数据；索引存储在「索引 B-Tree」中，叶子节点存储索引值和主键。

B-Tree 的设计使得 SQLite 的操作具有优秀的复杂度：查找 O(log N)、范围扫描 O(log N + M)、插入/删除 O(log N)。对于中小规模的数据集，B-Tree 的性能表现非常出色。

### 数据类型

SQLite 使用动态类型系统——每个值可以存储任何类型，列的类型声明更多是一种「亲和性」而非硬性限制。SQLite 支持以下基本类型：NULL（空值）、INTEGER（64位有符号整数）、REAL（IEEE 754 浮点数）、TEXT（UTF-8 文本）、BLOB（二进制大对象）。

```sql
-- SQLite 数据类型
CREATE TABLE products (
    id INTEGER PRIMARY KEY,           -- INTEGER
    name TEXT NOT NULL,              -- TEXT
    price REAL DEFAULT 0.0,         -- REAL
    description TEXT,                -- TEXT
    image BLOB,                     -- BLOB（二进制数据）
    metadata JSON,                   -- TEXT（JSON 作为文本存储）
    created_at TEXT DEFAULT (datetime('now'))
);
```

虽然 SQLite 是动态类型，但每个值都会根据一定规则被赋予「存储类型」：

- 如果值是纯整数且在 -9223372036854775808 到 9223372036854775807 之间，存储为 INTEGER
- 如果值是浮点数，存储为 REAL
- 如果值是字符串或 NULL，存储为 TEXT 或 NULL
- 其他情况存储为 BLOB

### 事务机制

SQLite 的事务是完整的 ACID 事务，这是 SQLite 区别于许多 NoSQL 数据库的核心优势。

```sql
-- 标准事务
BEGIN TRANSACTION;
INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');
INSERT INTO orders (user_id, total) VALUES (1, 99.99);
COMMIT;

-- 回滚事务
BEGIN TRANSACTION;
DELETE FROM users WHERE id = 1;
ROLLBACK;  -- 撤销所有更改

-- 保存点（嵌套事务）
BEGIN TRANSACTION;
INSERT INTO users (name) VALUES ('Bob');
SAVEPOINT sp1;
INSERT INTO orders (user_id, total) VALUES (2, 50.00);
ROLLBACK TO SAVEPOINT sp1;  -- 只回滚到 sp1
COMMIT;  -- Bob 会保存，订单不会保存
```

---

## WAL 模式与并发机制

### 三种日志模式对比

SQLite 支持三种不同的日志模式，决定了数据库的并发能力和数据安全性。

| 维度 | DELETE | WAL | MEMORY |
|------|--------|-----|--------|
| **日志类型** | 回滚日志 | 预写日志 | 无日志 |
| **并发读** | 支持 | 支持（读写可并行） | 支持 |
| **并发写** | 不支持 | 支持 | 不支持 |
| **数据可靠性** | 取决于 fsync | 可配置 | 最低 |
| **文件大小** | 稳定 | 可能膨胀 | 不适用 |
| **适用场景** | 简单场景 | 高并发应用 | 测试/临时 |

### WAL 模式详解

WAL（Write-Ahead Logging，预写日志）模式是 SQLite 在 2010 年（3.7.0版本）引入的重大特性，至今仍是 SQLite 最重要的改进之一。

在 WAL 模式下，写操作首先记录到 WAL 文件，然后定期「检查点」（checkpoint）将 WAL 中的修改合并回主数据库文件。这带来了几个关键优势：

**读写可以并行**。在 DELETE 模式下，写操作会获取排他锁，此时读操作无法进行。在 WAL 模式下，写操作只锁定 WAL 文件，读操作完全不受影响。

**写入不需要等待读取**。这解决了「写入饥饿」问题——在 DELETE 模式下，如果不断有读操作，写操作可能长时间等待。

**更好的并发性能**。WAL 模式下的写入通常只需要 fsync WAL 文件，而不需要 fsync 整个数据库文件，性能更好。

```sql
-- 启用 WAL 模式（最关键的配置）
PRAGMA journal_mode=WAL;

-- 推荐配置
PRAGMA synchronous=NORMAL;   -- 推荐：平衡性能与安全
-- NORMAL: 仅在关键事务点同步（推荐）
-- FULL: 每个事务都同步（最安全但最慢）
-- OFF: 不同步（最快，仅测试使用）

PRAGMA busy_timeout=5000;    -- 写锁等待超时 5 秒
PRAGMA cache_size=-64000;    -- 64MB 页缓存（负数表示 KB）

-- 检查 WAL 状态
PRAGMA journal_mode;         -- 返回: wal
PRAGMA wal_checkpoint;       -- 检查点信息
```

### WAL 的工作原理

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   主数据库文件   │ ←→  │   WAL 文件      │ ←→  │   共享内存      │
│   (*.db)        │     │   (*-wal)       │     │   (*-shm)       │
│                 │     │                 │     │                 │
│  持久化存储      │     │ 记录所有修改     │     │  读写进程同步    │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

WAL 模式下，数据库目录会有三个文件：主数据库文件（`app.db`）、WAL 文件（`app.db-wal`）、共享内存文件（`app.db-shm`）。这三个文件必须保持一致——复制数据库时应该同时复制这三个文件。

### 并发配置实战

```sql
-- 高并发应用配置
PRAGMA journal_mode=WAL;
PRAGMA synchronous=NORMAL;
PRAGMA busy_timeout=30000;      -- 30 秒锁等待
PRAGMA cache_size=-200000;     -- 200MB 缓存
PRAGMA temp_store=MEMORY;       -- 临时表使用内存
PRAGMA mmap_size=268435456;    -- 256MB 内存映射

-- 多连接配置
PRAGMA read_uncommitted=1;      -- 允许脏读（提升性能）
```

> [!TIP]
> WAL 模式下，多个读进程可以同时访问数据库。写操作会获取排他锁，但读操作永远不会阻塞写操作——这是 WAL 模式相比 DELETE 模式最大的优势。

---

## JSON/FTS5/R-Tree 高级特性

### JSON 支持（SQLite 3.38+）

SQLite 3.38 引入了原生的 JSON 支持，提供了丰富的 JSON 处理函数。这让 SQLite 可以优雅地处理半结构化数据——既可以用 SQL 查询结构化字段，又可以用 JSON 函数查询灵活字段。

```sql
-- 创建包含 JSON 的表
CREATE TABLE events (
    id INTEGER PRIMARY KEY,
    name TEXT,
    metadata JSON  -- JSON 类型（自动验证 JSON 格式）
);

-- 插入 JSON 数据
INSERT INTO events (name, metadata) VALUES 
    ('user_signup', '{"source": "web", "plan": "free", "referrer": "google"}'),
    ('purchase', '{"amount": 99.99, "currency": "USD", "items": 3}'),
    ('click', '{"button": "buy", "page": "/products"}');

-- 提取 JSON 值
SELECT 
    id,
    name,
    json_extract(metadata, '$.source') as source,
    json_extract(metadata, '$.amount') as amount
FROM events;

-- JSON 路径查询
SELECT * FROM events 
WHERE json_extract(metadata, '$.plan') = 'free';

-- 更新 JSON 字段
UPDATE events 
SET metadata = json_set(metadata, '$.plan', 'premium')
WHERE id = 1;

-- JSON 数组操作
INSERT INTO events (name, metadata) VALUES 
    ('tagged', '{"tags": ["vip", "early_adopter"], "scores": [85, 90, 78]}');

-- 遍历 JSON 数组
SELECT json_each.value 
FROM events, json_each(events.metadata, '$.tags')
WHERE events.name = 'tagged';
```

### FTS5：全文搜索

FTS5（Full-Text Search version 5）是 SQLite 的全文搜索扩展，支持高效的中英文搜索。对于需要关键词搜索的应用，FTS5 是比 LIKE 模糊匹配优雅得多的解决方案。

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
    ('SQLite vs PostgreSQL', '两个数据库各有优劣，需要根据场景选择...'),
    ('本地优先应用开发', 'Local-First 是现代应用的新范式...');

-- 基础全文搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'SQLite';

-- 多词搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'AI SQLite';

-- 布尔搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH 'AI AND 数据库';
SELECT * FROM articles_fts WHERE articles_fts MATCH 'SQLite OR PostgreSQL';

-- 短语搜索
SELECT * FROM articles_fts WHERE articles_fts MATCH '"AI 时代"';

-- BM25 排序
SELECT title, bm25(articles_fts) as rank 
FROM articles_fts 
WHERE articles_fts MATCH '数据库'
ORDER BY rank;
```

### 中文分词配置

SQLite 默认的分词器是「unicode61」，它按 Unicode 字符边界分词，对中文支持良好。如果需要更精细的中文分词，可以使用第三方分词器扩展。

```sql
-- 使用 Unicode61 分词器（支持中文）
CREATE VIRTUAL TABLE docs_zh USING fts5(
    title,
    content,
    tokenize='unicode61 remove_diacritics 1'
);

-- 配合外部内容表（节省空间）
CREATE TABLE docs_main (
    id INTEGER PRIMARY KEY,
    title TEXT,
    content TEXT
);

CREATE VIRTUAL TABLE docs_fts USING fts5(
    title,
    content,
    content='docs_main',    -- 关联到主表
    content_rowid='id'
);

-- 自动同步触发器
CREATE TRIGGER docs_ai AFTER INSERT ON docs_main BEGIN
    INSERT INTO docs_fts(rowid, title, content) 
    VALUES (new.id, new.title, new.content);
END;

CREATE TRIGGER docs_ad AFTER DELETE ON docs_main BEGIN
    INSERT INTO docs_fts(docs_fts, rowid, title, content) 
    VALUES('delete', old.id, old.title, old.content);
END;

CREATE TRIGGER docs_au AFTER UPDATE ON docs_main BEGIN
    INSERT INTO docs_fts(docs_fts, rowid, title, content) 
    VALUES('delete', old.id, old.title, old.content);
    INSERT INTO docs_fts(rowid, title, content) 
    VALUES (new.id, new.title, new.content);
END;
```

### R-Tree：空间索引

R-Tree 是一种专门用于索引空间数据的数据结构，SQLite 内置支持。

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

## SQLite vs PostgreSQL vs MySQL 对比

### 核心定位对比

| 维度 | SQLite | PostgreSQL | MySQL |
|------|--------|------------|-------|
| **架构类型** | 嵌入式（无服务器） | 客户端/服务器 | 客户端/服务器 |
| **部署复杂度** | ★☆☆☆☆（极简） | ★★★☆☆（中等） | ★★★☆☆（中等） |
| **并发能力** | ★★☆☆☆（受限于写锁） | ★★★★★（真正并发） | ★★★★☆（优秀） |
| **数据类型** | 基础+JSON | 丰富+JSON+GIS+范围 | 基础+JSON |
| **扩展性** | ★★☆☆☆ | ★★★★★（丰富扩展） | ★★★☆☆ |
| **适用规模** | < 100GB | 无上限 | < 1TB |

### 功能特性详细对比

| 特性 | SQLite | PostgreSQL | MySQL |
|------|--------|------------|-------|
| **ACID 事务** | ✓ | ✓ | ✓ |
| **外键约束** | ✓ | ✓ | ✓ |
| **触发器** | ✓（有限） | ✓ | ✓ |
| **窗口函数** | ✓ | ✓ | ✓ |
| **CTE（WITH）** | ✓ | ✓ | ✓ |
| **JSON 支持** | ✓（原生） | ✓（JSONB 更强） | ✓ |
| **全文搜索** | ✓（FTS5） | ✓ | ✓ |
| **空间数据** | ✓（R-Tree） | ✓（PostGIS 极强） | ✓ |
| **数组类型** | ✗ | ✓ | ✗ |
| **物化视图** | ✗ | ✓ | ✗ |
| **分区表** | ✗（受限制） | ✓ | ✓ |
| **存储过程** | ✗ | ✓ | ✓ |

> [!IMPORTANT]
> SQLite 的「单写」限制是相对于高并发写入场景的。在单写多读、或写并发不高的场景下，SQLite 的性能表现往往优于网络数据库，因为省去了 TCP/IP 通信的开销。

### 选型决策树

```
开始
  │
  ├─ 是否需要远程多进程写入？
  │     │
  │     ├─ 是 → 数据量 < 100GB？
  │     │          ├─ 是 → Turso（分布式 SQLite）✓
  │     │          └─ 否 → PostgreSQL ✓
  │     │
  │     └─ 否 → SQLite ✓
  │
  └─ 是否需要服务器模式？
        ├─ 是 → PostgreSQL / MySQL ✓
        └─ 否 → SQLite ✓
```

---

## Node.js 集成：better-sqlite3 与 sql.js

### better-sqlite3：同步高性能驱动

better-sqlite3 是 Node.js 中性能最高的 SQLite 驱动，提供同步 API，非常适合 CLI 工具、Serverless 函数、Electron 应用等场景。

```typescript
import Database from 'better-sqlite3'
import path from 'path'

// 连接数据库（自动创建）
const db = new Database(path.join(__dirname, 'app.db'))

// 启用 WAL 模式（生产环境必做）
db.pragma('journal_mode=WAL')
db.pragma('synchronous=NORMAL')
db.pragma('foreign_keys=ON')

// 创建表
db.exec(`
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        email TEXT UNIQUE NOT NULL,
        name TEXT,
        created_at TEXT DEFAULT (datetime('now'))
    );
    
    CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
`)

// 插入数据（Prepared Statement）
const insertStmt = db.prepare(`
    INSERT INTO users (email, name) VALUES (?, ?)
`)

const result = insertStmt.run('alice@example.com', 'Alice')
console.log('插入 ID:', result.lastInsertRowid)

// 批量插入（事务）
const insertMany = db.transaction((users: Array<{email: string, name: string}>) => {
    for (const user of users) {
        insertStmt.run(user.email, user.name)
    }
})

insertMany([
    { email: 'bob@example.com', name: 'Bob' },
    { email: 'carol@example.com', name: 'Carol' },
    { email: 'dave@example.com', name: 'Dave' }
])

// 查询数据
const selectStmt = db.prepare('SELECT * FROM users WHERE id = ?')
const user = selectStmt.get(1)
console.log('查询结果:', user)

// 列表查询
const allUsers = db.prepare('SELECT * FROM users ORDER BY created_at DESC').all()
console.log('所有用户:', allUsers)

// 更新数据
const updateStmt = db.prepare('UPDATE users SET name = ? WHERE email = ?')
updateStmt.run('Alice Smith', 'alice@example.com')

// 删除数据
const deleteStmt = db.prepare('DELETE FROM users WHERE id = ?')
deleteStmt.run(4)

// 事务
const transferMoney = db.transaction((fromId: number, toId: number, amount: number) => {
    db.prepare('UPDATE accounts SET balance = balance - ? WHERE id = ?').run(amount, fromId)
    db.prepare('UPDATE accounts SET balance = balance + ? WHERE id = ?').run(amount, toId)
})

transferMoney(1, 2, 100)

// 关闭数据库
db.close()
```

### sql.js：WebAssembly 版本

sql.js 是 SQLite 的纯 JavaScript 移植版，基于 Emscripten 编译，可以在浏览器中运行。对于需要完全前端化的应用，sql.js 是一个有趣的选择。

```typescript
// Node.js 环境
import initSqlJs from 'sql.js'

const SQL = await initSqlJs()
const db = new SQL.Database()

// 执行 SQL
db.run('CREATE TABLE test (id INTEGER PRIMARY KEY, name TEXT)')
db.run("INSERT INTO test VALUES (1, 'Hello')")
db.run("INSERT INTO test VALUES (2, 'World')")

// 查询
const results = db.exec('SELECT * FROM test')
console.log(results[0].values)  // [[1, "Hello"], [2, "World"]]

// 导出数据库到 Uint8Array（可保存到文件）
const data = db.export()

// 从 Uint8Array 加载数据库
const db2 = new SQL.Database(data)
```

---

## Bun 原生支持

Bun 内置了对 SQLite 的原生支持，无需安装任何驱动。这使得 Bun 成为编写 SQLite 应用的绝佳选择。

```typescript
// Bun 原生 SQLite API
const db = await Database.open('app.db')

// 创建表
await db.exec(`
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE
    )
`)

// 插入数据
const result = await db.run(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ['Alice', 'alice@example.com']
)
console.log('插入 ID:', result.lastInsertRowid)

// 查询数据
const users = await db.query('SELECT * FROM users WHERE name = ?', ['Alice'])
for (const user of users) {
    console.log(user.id, user.name, user.email)
}

// 参数绑定（安全防注入）
const stmt = await db.prepare('SELECT * FROM users WHERE email = ?')
stmt.setParameters({ 0: 'alice@example.com' })
const result2 = await stmt.all()
stmt.finalize()

// 事务
await db.transaction(() => {
    db.run('INSERT INTO users (name, email) VALUES (?, ?)', ['Bob', 'bob@example.com'])
    db.run('INSERT INTO users (name, email) VALUES (?, ?)', ['Carol', 'carol@example.com'])
})

// 批量操作
const insertMany = db.prepare('INSERT INTO users (name, email) VALUES (?, ?)')
await db.transaction(() => {
    for (const user of ['Dave', 'Eve', 'Frank']) {
        insertMany.run(user, `${user.toLowerCase()}@example.com`)
    }
})
insertMany.finalize()

await db.close()
```

---

## Python 集成：sqlite3、SQLAlchemy、aiosqlite

### sqlite3 标准库

Python 内置了 `sqlite3` 模块，无需安装任何依赖。

```python
import sqlite3
from contextlib import contextmanager
from datetime import datetime

# 基本连接
conn = sqlite3.connect('mydb.db')
cursor = conn.cursor()

# 创建表
cursor.execute('''
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE NOT NULL,
        created_at TEXT DEFAULT (datetime('now'))
    )
''')
conn.commit()

# 插入数据
cursor.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Alice", "alice@example.com"))
conn.commit()

# 查询数据
cursor.execute("SELECT * FROM users WHERE id = ?", (1,))
result = cursor.fetchone()
print(result)

# 批量插入
users = [("Bob", "bob@example.com"), ("Carol", "carol@example.com")]
cursor.executemany("INSERT INTO users (name, email) VALUES (?, ?)", users)
conn.commit()

# 使用 with 语句自动管理连接
with sqlite3.connect('mydb.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    users = cursor.fetchall()

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

# 事务处理
try:
    cursor.execute("INSERT INTO users (name, email) VALUES (?, ?)", ("Dave", "dave@example.com"))
    cursor.execute("UPDATE users SET name = ? WHERE email = ?", ("Dave Jr", "dave@example.com"))
    conn.commit()
except Exception as e:
    conn.rollback()
    print(f"错误: {e}")
```

### aiosqlite（异步）

对于 asyncio 应用，aiosqlite 提供了异步的 SQLite 接口。

```python
import asyncio
import aiosqlite

async def main():
    async with aiosqlite.connect('mydb.db') as db:
        # 启用 WAL
        await db.execute("PRAGMA journal_mode=WAL")
        
        # 创建表
        await db.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL
            )
        ''')
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

asyncio.run(main())
```

### SQLAlchemy

SQLAlchemy 是 Python 中最成熟的 ORM，提供了完整的 SQLAlchemy 表达式语言和 ORM 功能。

```python
from sqlalchemy import create_engine, Column, Integer, String, Float, DateTime, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship
from datetime import datetime

# 创建引擎
engine = create_engine('sqlite:///mydb.db', echo=False)
Session = sessionmaker(bind=engine)
Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    posts = relationship("Post", back_populates="author")

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(200), nullable=False)
    content = Column(String)
    author_id = Column(Integer, ForeignKey('users.id'))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    author = relationship("User", back_populates="posts")

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
results = session.query(User, Post).join(Post, User.id == Post.author_id).filter(
    User.name == 'Alice'
).all()

session.close()
```

---

## Prisma + SQLite

Prisma 是一个现代的 TypeScript ORM，提供了类型安全的数据库访问。

```typescript
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
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

// 更新
await prisma.user.update({
  where: { email: 'alice@example.com' },
  data: { name: 'Alice Smith' }
})

// 删除
await prisma.post.delete({
  where: { id: post.id }
})
```

---

## Drizzle + SQLite

Drizzle 是一个轻量级的 TypeScript ORM，以接近原生 SQL 的语法著称。

```typescript
// db/schema.ts
import { sqliteTable, text, integer, real } from 'drizzle-orm/sqlite-core'

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
})

export const posts = sqliteTable('posts', {
    id: integer('id').primaryKey({ autoIncrement: true }),
    title: text('title').notNull(),
    content: text('content'),
    authorId: integer('author_id').references(() => users.id),
    createdAt: text('created_at').default(sql`(datetime('now'))`)
})
```

```typescript
import { drizzle } from 'drizzle-orm/better-sqlite3'
import Database from 'better-sqlite3'

const sqlite = new Database('./app.db')
const db = drizzle(sqlite)

// 启用 WAL
sqlite.pragma('journal_mode=WAL')

// 创建
const [newUser] = await db.insert(users).values({
    email: 'alice@example.com',
    name: 'Alice',
    age: 30,
    metadata: { source: 'web', plan: 'free' }
}).returning()

// 查询
const user = await db.select().from(users).where(eq(users.email, 'alice@example.com')).get()

// 列表查询
const youngUsers = await db.select().from(users)
    .where(lt(users.age, 30))
    .orderBy(desc(users.createdAt))
    .limit(10)

// 更新
await db.update(users)
    .set({ name: 'Alice Smith' })
    .where(eq(users.email, 'alice@example.com'))

// 删除
await db.delete(users).where(eq(users.id, 1))

// 聚合
const stats = await db.select({
    totalUsers: sql<number>`count(*)`,
    avgAge: sql<number>`avg(age)`,
    maxAge: sql<number>`max(age)`,
}).from(users)
```

---

## Turso：分布式 SQLite

### Turso 概述

Turso 是基于 libSQL（SQLite 的开源分支）的云托管数据库服务。libSQL 由 Tari Labs 开发，增加了服务器模式、嵌入式复制、向量类型等 SQLite 原版不支持的特性。

Turso 的核心价值在于：**将 SQLite 的简单性与分布式架构结合起来**。你可以像使用 SQLite 一样开发（直接操作文件），然后将数据库部署到全球边缘节点，实现超低延迟的数据访问。

### Turso vs 传统 SQLite

| 特性 | SQLite | libSQL / Turso |
|------|--------|---------------|
| **服务器模式** | ✗ | ✓ |
| **嵌入式复制** | ✗ | ✓ |
| **向量类型** | ✗ | ✓ |
| **边缘部署** | ✗ | ✓ |
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

# 本地开发
turso dev db instance my-app-db

# 执行 SQL
turso db shell my-app-db

# 推拉数据
turso db push my-app-db    # 本地推送到云端
turso db pull my-app-db   # 云端拉取到本地
```

### Node.js 连接 Turso

```typescript
import { createClient } from '@libsql/client'

// 创建客户端
const client = createClient({
    url: 'libsql://my-app-db-xxxx.turso.io',
    authToken: 'your-auth-token',
})

// 执行查询
const result = await client.execute('SELECT * FROM users LIMIT 10')
console.log(result.rows)

// 参数化查询
const user = await client.execute({
    sql: 'SELECT * FROM users WHERE email = ?',
    args: ['alice@example.com']
})

// 事务
await client.transaction(async (tx) => {
    await tx.execute('INSERT INTO users (email, name) VALUES (?, ?)', 
        ['bob@example.com', 'Bob'])
    await tx.execute('UPDATE counters SET count = count + 1 WHERE name = ?',
        ['user_count'])
})

// 关闭连接
await client.close()
```

---

## Litestream 自动备份

### Litestream 概述

Litestream 是一个开源的 SQLite 复制工具，将 WAL 日志持续流式传输到 S3、R2 或其他存储后端。与传统备份（定期快照）不同，Litestream 实现了近乎零 RPO（恢复点目标）的连续复制——最多只会丢失 1-2 秒的数据。

Litestream 的配置极其简单，不需要修改应用代码，只需要在启动数据库进程时同时启动 Litestream 进程即可。

### 配置文件

```yaml
# litestream.yml
dbs:
  - path: /data/app.db
    replicas:
      - url: s3://my-bucket/litestream/
        retention: 30d
        sync-interval: 1s
      - url: file:///var/backups/litestream
        retention: 7d
        sync-interval: 10s
```

### Docker Compose 集成

```yaml
version: "3.8"
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
```

---

## 性能调优与最佳实践

### 索引设计

索引是 SQLite 性能优化最关键的手段。以下是几个实用的索引设计原则：

```sql
-- 1. 单列索引（最常用）
CREATE INDEX idx_users_email ON users(email);

-- 2. 复合索引（多列查询）
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC);

-- 3. 部分索引（只索引需要的数据）
CREATE INDEX idx_posts_published ON posts(published_at DESC)
WHERE published = 1;

CREATE INDEX idx_users_active ON users(email)
WHERE deleted_at IS NULL;

-- 4. 表达式索引（函数式索引）
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
CREATE INDEX idx_posts_year ON posts(strftime('%Y', created_at));
```

### 查询优化

```sql
-- 使用 EXPLAIN QUERY PLAN 分析查询
EXPLAIN QUERY PLAN
SELECT * FROM users WHERE email = 'test@example.com';

-- 分页优化：避免 OFFSET 大值
-- ❌ 低效
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 10000;

-- ✅ 高效：使用 WHERE id > 
SELECT * FROM posts WHERE id > 10000 ORDER BY id LIMIT 20;

-- ✅ 使用子查询
SELECT * FROM posts 
WHERE id <= (SELECT id FROM posts ORDER BY id LIMIT 1 OFFSET 10000)
ORDER BY id DESC LIMIT 20;

-- 批量操作：使用事务
BEGIN TRANSACTION;
INSERT INTO users (email, username) VALUES 
    ('a@test.com', 'a'),
    ('b@test.com', 'b'),
    ('c@test.com', 'c');
COMMIT;
```

### 常见陷阱与最佳实践

| 陷阱 | 问题 | 解决方案 |
|------|------|---------|
| **不使用 WAL** | 并发差、数据不安全 | `PRAGMA journal_mode=WAL` |
| **不使用事务** | 性能差 | 批量操作使用事务 |
| **忽视索引** | 查询慢 | 为常用查询添加索引 |
| **LIKE 模糊搜索** | 无法使用索引 | 使用 FTS5 |
| **大 Value** | 性能问题 | 压缩或拆分 |
| **不使用外键约束** | 数据不一致 | `PRAGMA foreign_keys=ON` |

---

## SQLite 限制与迁移时机

### SQLite 的局限性

尽管 SQLite 非常强大，但它确实有一些固有限制，了解这些限制有助于在合适的时机选择迁移。

**写入并发受限**。SQLite 使用文件级锁，同一时刻只能有一个写入进程。这是 SQLite 与生俱来的限制，无法通过配置彻底解决。对于需要高并发写入的场景，应该迁移到 PostgreSQL 或使用 Turso 的分布式架构。

**存储容量限制**。虽然 SQLite 支持最大 281 TB 的数据库文件，但在实际使用中，超过 100 GB 的数据库会遇到备份、迁移、性能等方面的挑战。如果预期数据量会超过这个规模，应该考虑 PostgreSQL。

**服务器模式缺失**。SQLite 是嵌入式数据库，没有内置的服务器模式和多客户端支持。如果需要远程访问、连接池、细粒度权限控制，SQLite 可能不是最佳选择。

**高级特性不足**。SQLite 不支持存储过程、物化视图、分区表（受限）等高级特性。如果应用需要这些特性，应该选择 PostgreSQL。

### 迁移时机判断

| 指标 | SQLite 上限 | 迁移信号 |
|------|------------|---------|
| **数据库大小** | 100GB | 超过 50GB |
| **并发写入** | 1 | 需要多写 |
| **表数量** | 数十个 | 需要数百个 |
| **访问模式** | 本地/边缘 | 需要远程访问 |
| **团队规模** | 1-5 人 | 多人协作 |

---

## AI 应用场景实战

### 本地知识库架构

在 AI 应用中，SQLite 是本地知识库的绝佳选择：

```
┌──────────────────────────────────────────────────────────────┐
│                    AI 本地知识库架构                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────────┐     │
│  │  用户界面    │    │  LLM API   │    │  Embedding   │     │
│  │  (Next.js)  │    │  (Claude)   │    │  (OpenAI)    │     │
│  └──────┬──────┘    └──────┬──────┘    └──────┬───────┘     │
│         │                  │                  │              │
│         └──────────────────┼──────────────────┘              │
│                            │                               │
│                            ▼                               │
│                   ┌─────────────────┐                     │
│                   │   RAG 检索引擎   │                     │
│                   │   (向量搜索)     │                     │
│                   └────────┬────────┘                     │
│                            │                               │
│                            ▼                               │
│                   ┌─────────────────┐                     │
│                   │   SQLite        │                     │
│                   │   (文档+向量)    │                     │
│                   └─────────────────┘                     │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### RAG 应用示例

```python
import sqlite3
import json
from datetime import datetime

# 初始化数据库
def init_db():
    conn = sqlite3.connect('rag.db')
    conn.execute("PRAGMA journal_mode=WAL")
    
    conn.execute('''
        CREATE TABLE IF NOT EXISTS chunks (
            id INTEGER PRIMARY KEY,
            doc_id INTEGER,
            content TEXT NOT NULL,
            chunk_index INTEGER,
            metadata JSON,
            created_at TEXT DEFAULT (datetime('now'))
        )
    ''')
    
    conn.execute('''
        CREATE INDEX IF NOT EXISTS idx_chunks_doc ON chunks(doc_id)
    ''')
    
    conn.commit()
    return conn

# 文档分块存储
def index_document(conn, doc_id: int, title: str, content: str, chunk_size: int = 500):
    """将文档分块并存储"""
    # 简单分块：按段落或固定长度
    chunks = []
    paragraphs = content.split('\n\n')
    current_chunk = ""
    
    for para in paragraphs:
        if len(current_chunk) + len(para) <= chunk_size:
            current_chunk += para + "\n\n"
        else:
            if current_chunk:
                chunks.append(current_chunk.strip())
            current_chunk = para + "\n\n"
    
    if current_chunk:
        chunks.append(current_chunk.strip())
    
    cursor = conn.cursor()
    for i, chunk in enumerate(chunks):
        cursor.execute('''
            INSERT INTO chunks (doc_id, content, chunk_index, metadata)
            VALUES (?, ?, ?, ?)
        ''', (doc_id, chunk, i, json.dumps({"title": title})))
    
    conn.commit()
    return len(chunks)

# 简单全文检索
def search_chunks(conn, query: str, top_k: int = 5):
    """简单的关键词检索"""
    cursor = conn.execute('''
        SELECT id, doc_id, content, metadata
        FROM chunks
        WHERE content LIKE ?
        ORDER BY length(content)
        LIMIT ?
    ''', (f'%{query}%', top_k))
    
    return [
        {"id": row[0], "doc_id": row[1], "content": row[2], "metadata": json.loads(row[3])}
        for row in cursor.fetchall()
    ]

# RAG 查询
async def rag_query(question: str, conn):
    # 1. 检索相关 chunks
    chunks = search_chunks(conn, question, top_k=3)
    
    if not chunks:
        return "抱歉，没有找到与您问题相关的文档。"
    
    # 2. 构建上下文
    context = "\n\n".join([
        f"【文档 {i+1}】\n{c['content']}"
        for i, c in enumerate(chunks)
    ])
    
    # 3. 调用 LLM（简化示例）
    # 在实际应用中，这里应该调用 OpenAI / Claude API
    response = f"基于以下上下文回答：\n\n{context}\n\n问题：{question}"
    
    return response, chunks
```

---

## 选型建议

### 何时选择 SQLite

| 场景 | 推荐程度 | 说明 |
|------|---------|------|
| 移动应用 | ★★★★★ | iOS/Android 原生支持 |
| 桌面软件 | ★★★★★ | 单文件，零配置 |
| 本地开发 | ★★★★★ | 秒级启动 |
| AI 本地知识库 | ★★★★★ | 单文件管理 |
| 边缘计算/IoT | ★★★★★ | 零依赖 |
| 小型网站（< 1万日活） | ★★★★☆ | 成本最低 |
| Serverless 函数 | ★★★★☆ | 冷启动快 |
| 高并发网站 | ★☆☆☆☆ | 不推荐 |
| 多租户 SaaS | ★★☆☆☆ | 隔离性较弱 |

### 何时选择其他方案

- **需要远程多进程写入** → PostgreSQL
- **需要 TB 级数据分析** → ClickHouse / DuckDB
- **需要 GIS 功能** → PostgreSQL + PostGIS
- **需要水平扩展** → Turso / CockroachDB
- **微服务架构** → PostgreSQL / MySQL

---

> [!SUCCESS]
> SQLite 是嵌入式数据库的工业标准，在 2026 年的今天已经有了远超传统观念的进化：从 WAL 模式带来的并发提升、到 Turso 带来的分布式能力、到 AI 本地知识库的新场景。掌握 SQLite 的 WAL 配置、FTS5 全文搜索、Drizzle ORM 集成、以及 Litestream 备份策略，就掌握了现代应用开发中轻量级数据持久化的全部精髓。

---

## 附录：常用资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://www.sqlite.org/docs.html |
| Turso 文档 | https://docs.turso.tech |
| Litestream 文档 | https://litestream.io/docs |
| Drizzle ORM | https://orm.drizzle.team |
| Prisma | https://prisma.io/docs |
| DB Browser for SQLite | https://sqlitebrowser.org |
