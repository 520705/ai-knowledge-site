# MySQL 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 MySQL 9.0 新特性、InnoDB 存储引擎、索引类型详解、与 PostgreSQL 的对比，以及在 AI 应用中的使用场景。

---

## 目录

1. [[#MySQL 概述与核心定位]]
2. [[#MySQL 9.0 新特性]]
3. [[#InnoDB 存储引擎详解]]
4. [[#索引类型与选择指南]]
5. [[#事务与并发控制]]
6. [[#MySQL vs PostgreSQL 对比]]
7. [[#分区与分表策略]]
8. [[#连接池与性能优化]]
9. [[#AI 应用中的 MySQL]]
10. [[#实战场景与代码示例]]
11. [[#选型建议]]

---

## MySQL 概述与核心定位

### 为什么选择 MySQL

MySQL 是全球最流行的开源关系型数据库，由 MySQL AB 公司于 1995 年发布，2008 年被 Sun Microsystems 收购，2010 年随 Oracle 收购 Sun 而成为 Oracle 产品线的一部分。

MySQL 的核心优势体现在三个维度：

**简单易用**：安装配置简单，文档丰富，社区活跃，学习曲线平缓。

**性能卓越**：针对 Web 应用场景优化，读写性能出色，特别适合高并发场景。

**生态成熟**：WordPress、Drupal、Magento 等主流 CMS 的默认数据库，LAMP/LEMP 架构的核心组件。

### MySQL 在 AI 时代的定位

MySQL 8.0+ 引入了多项新特性，使其在 AI 应用中仍具竞争力：

- **JSON 函数增强**：更强大的 JSON 数据处理能力
- **窗口函数**：支持 SQL:2003 窗口函数标准
- **CTE 支持**：简化复杂查询
- **向量搜索**：MySQL 9.0 原生支持向量相似度搜索

### 市场份额与行业地位

MySQL 在全球数据库市场中占据重要地位：

| 排名 | 数据库 | 市场份额（2025） | 趋势 |
|------|--------|------------------|------|
| 1 | PostgreSQL | 28.5% | ↑ |
| 2 | MySQL | 25.2% | → |
| 3 | MongoDB | 12.8% | ↓ |
| 4 | Microsoft SQL Server | 10.5% | → |
| 5 | Redis | 8.3% | ↑ |

**关键数据点**：
- MySQL 是全球部署最广泛的开源数据库
- 超过 90% 的 Web 应用使用 MySQL 作为数据库
- 主要使用者包括 Facebook、Twitter、YouTube、WordPress 等知名网站
- MySQL 8.0+ 增加了窗口函数、CTE、JSON 表函数等高级特性
- MySQL HeatWave 云服务提供了内置的机器学习功能

**MySQL 适用场景**：

| 场景 | 推荐程度 | 说明 |
|------|----------|------|
| Web 应用 | ⭐⭐⭐⭐⭐ | LAMP/LEMP 标配 |
| WordPress/Drupal | ⭐⭐⭐⭐⭐ | CMS 默认数据库 |
| 电子商务 | ⭐⭐⭐⭐ | 高并发读写优化 |
| SaaS 应用 | ⭐⭐⭐⭐ | 多租户支持 |
| AI 向量搜索 | ⭐⭐⭐⭐ | 9.0 原生支持 |
| 复杂分析 | ⭐⭐⭐ | 功能有限 |

---

## 核心概念与数据模型

### MySQL 架构

MySQL 采用分层架构设计：

```
┌─────────────────────────────────────────────────────────────┐
│                        MySQL Server                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   连接层                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │ 连接管理  │  │ 线程池   │  │ 认证模块  │               │
│   └──────────┘  └──────────┘  └──────────┘               │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   SQL 层                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │  解析器  │  │ 优化器   │  │  执行器  │               │
│   └──────────┘  └──────────┘  └──────────┘               │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   存储引擎层（插件式）                                        │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │  InnoDB  │  │  MyISAM  │  │ Memory   │               │
│   │  (默认)   │  │ (已废弃)  │  │ (临时表)  │               │
│   └──────────┘  └──────────┘  └──────────┘               │
│                                                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   数据层                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │ Buffer   │  │  Redo    │  │  Undo    │               │
│   │  Pool    │  │   Log    │  │   Log    │               │
│   └──────────┘  └──────────┘  └──────────┘               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### InnoDB 核心概念

#### 1. 缓冲池（Buffer Pool）

InnoDB 使用缓冲池缓存表数据和索引：

```sql
-- 查看缓冲池状态
SHOW STATUS LIKE 'Innodb_buffer_pool%';

-- 设置缓冲池大小（建议为系统内存的 50-70%）
SET GLOBAL innodb_buffer_pool_size = 4294967296;  -- 4GB

-- 预热缓冲池（重启后）
SET GLOBAL innodb_buffer_pool_load_at_startup = ON;

-- 查看缓冲池使用情况
SELECT 
    pool_size,
    free_buffers,
    database_pages,
    modified_db_pages,
    pages_hashed,
    pages_old
FROM information_schema.innodb_buffer_pool_stats;

-- 查看缓冲池中的页面
SELECT 
    SPACE,
    SPACE_NAME,
    PAGE_NUMBER,
    PAGE_TYPE,
    INDEX_NAME
FROM information_schema.innodb_buffer_page_limited
WHERE TABLE_NAME IS NOT NULL
LIMIT 10;
```

#### 2. 双重写缓冲区（Doublewrite Buffer）

防止部分页面写入导致的数据损坏：

```sql
-- 查看双重写缓冲区状态
SHOW STATUS LIKE 'Innodb_dblwr%';

-- 禁用双重写（仅在特定场景使用，如 SSD）
SET GLOBAL innodb_flush_method = 'O_DIRECT';
```

#### 3. 自适应哈希索引（AHI）

InnoDB 自动为热点数据构建哈希索引：

```sql
-- 查看 AHI 统计
SHOW STATUS LIKE 'Innodb_hash%';

-- 监控 AHI 命中率
SHOW STATUS LIKE 'Innodb_buffer_pool_read_requests';
SHOW STATUS LIKE 'Innodb_buffer_pool_read_ahead%';
```

### 数据模型设计

#### 1. 规范化设计

```sql
-- 高度规范化的订单系统
CREATE TABLE customers (
    customer_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE orders (
    order_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT UNSIGNED NOT NULL,
    order_number VARCHAR(50) NOT NULL UNIQUE,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    total_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    shipping_address JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
    INDEX idx_customer_id (customer_id),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE order_items (
    item_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    product_name VARCHAR(255) NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    subtotal DECIMAL(10,2) GENERATED ALWAYS AS (quantity * unit_price) STORED,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    INDEX idx_order_id (order_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

#### 2. 反规范化设计

```sql
-- 添加冗余字段优化查询性能
CREATE TABLE orders_denormalized (
    order_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    customer_id BIGINT UNSIGNED NOT NULL,
    customer_name VARCHAR(100),           -- 冗余字段
    customer_email VARCHAR(255),        -- 冗余字段
    order_number VARCHAR(50) NOT NULL UNIQUE,
    item_count INT UNSIGNED DEFAULT 0,   -- 冗余字段
    total_amount DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_customer_name (customer_name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 物化路径设计（层级结构）
CREATE TABLE categories (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id BIGINT UNSIGNED DEFAULT NULL,
    path VARCHAR(1000) DEFAULT '',
    level INT UNSIGNED DEFAULT 0,
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL,
    INDEX idx_path (path),
    INDEX idx_parent_id (parent_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 闭包表设计（复杂层级）
CREATE TABLE category_closure (
    ancestor_id BIGINT UNSIGNED NOT NULL,
    descendant_id BIGINT UNSIGNED NOT NULL,
    depth INT UNSIGNED NOT NULL,
    PRIMARY KEY (ancestor_id, descendant_id),
    FOREIGN KEY (ancestor_id) REFERENCES categories(id) ON DELETE CASCADE,
    FOREIGN KEY (descendant_id) REFERENCES categories(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

---

---

## MySQL 9.0 新特性

MySQL 9.0 于 2025 年发布，带来了向量搜索等重要新功能。

### 核心新特性

#### 1. 原生向量搜索

```sql
-- 创建向量列
CREATE TABLE documents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536),  -- 1536 维向量
    metadata JSON
);

-- 插入向量数据（使用 JSON 数组格式）
INSERT INTO documents (content, embedding) VALUES
    ('PostgreSQL introduction', '[0.1, 0.2, 0.3, ...]'),
    ('MySQL tutorial', '[0.4, 0.5, 0.6, ...]');

-- 向量相似度搜索
SELECT 
    id,
    content,
    VECTOR_DOT_PRODUCT(embedding, '[query_vector]') AS similarity
FROM documents
ORDER BY VECTOR_DOT_PRODUCT(embedding, '[query_vector]') DESC
LIMIT 5;

-- 余弦相似度
SELECT 
    id,
    content,
    VECTOR_COSINE_SIMILARITY(embedding, '[query_vector]') AS similarity
FROM documents
ORDER BY similarity DESC
LIMIT 5;

-- L2 距离（欧氏距离）
SELECT 
    id,
    content,
    VECTOR_L2_DISTANCE(embedding, '[query_vector]') AS distance
FROM documents
ORDER BY distance ASC
LIMIT 5;
```

#### 2. 增强的 JSON 支持

```sql
-- JSON_TABLE 函数（MySQL 8.0.14+ 已有，9.0 增强）
SELECT *
FROM JSON_TABLE(
    '[{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]',
    '$[*]'
    COLUMNS (
        name VARCHAR(50) PATH '$.name',
        age INT PATH '$.age'
    )
) AS users;

-- JSON_VALUE 函数
SELECT 
    JSON_VALUE(metadata, '$.category') AS category,
    JSON_VALUE(metadata, '$.tags[*]') AS tags
FROM articles;

-- 改进的 JSON 路径表达式
SELECT JSON_EXTRACT(
    '{"user": {"profile": {"name": "Alice"}}}',
    '$.user.profile.name'
);  -- "Alice"
```

#### 3. SQL 更强的兼容性

```sql
-- 增强的窗口函数
SELECT 
    department,
    employee,
    salary,
    AVG(salary) OVER (
        PARTITION BY department 
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_avg
FROM employees;

-- GROUP BY 增强
SELECT 
    region,
    COUNT(*) AS cnt,
    GROUPING(region) AS is_grand_total
FROM sales
GROUP BY ROLLUP(region);

-- 新的聚合函数
SELECT 
    COUNT(DISTINCT region) AS unique_regions,
    JSON_ARRAYAGG(DISTINCT product) AS products
FROM sales;
```

#### 4. 性能提升

| 优化项 | 说明 |
|--------|------|
| **查询优化器改进** | 更智能的执行计划选择 |
| **并行查询** | 支持多核并行处理 |
| **JSON 性能** | JSON 操作性能提升 30% |
| **向量索引** | HNSW 算法支持 |

---

## InnoDB 存储引擎详解

### InnoDB vs MyISAM 对比表

| 特性 | InnoDB | MyISAM |
|------|--------|--------|
| **事务支持** | 完整 ACID | 不支持 |
| **外键约束** | 支持 | 不支持 |
| **行级锁** | 是 | 否（表级锁） |
| **MVCC** | 支持 | 不支持 |
| **崩溃恢复** | 自动恢复 | 需要手动修复 |
| **全文索引** | MySQL 5.6+ 支持 | 原生支持 |
| **空间数据类型** | 支持 | 不支持 |
| **并发写入** | 高 | 低 |
| **适用场景** | OLTP、生产环境 | 读密集、日志 |

> [!IMPORTANT]
> MySQL 8.0+ 中，InnoDB 是默认存储引擎，且 MyISAM 已被标记为废弃。

### InnoDB 架构

```
┌─────────────────────────────────────────────────────────────┐
│                        MySQL Server                          │
├─────────────────────────────────────────────────────────────┤
│  连接层：连接管理、认证、线程池                                 │
├─────────────────────────────────────────────────────────────┤
│  SQL 层：解析器、优化器、执行器                                │
├─────────────────────────────────────────────────────────────┤
│  存储引擎层                                                   │
│  ┌─────────────────────────────────────────────────────────┐│
│  │                    InnoDB Storage Engine                  ││
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐   ││
│  │  │ Buffer Pool │  │  Redo Log   │  │ Undo Log    │   ││
│  │  │  (内存缓存)  │  │ (重做日志)  │  │ (回滚日志)  │   ││
│  │  └─────────────┘  └─────────────┘  └─────────────┘   ││
│  │  ┌─────────────────────────────────────────────────┐   ││
│  │  │              表空间 (Tablespace)                  │   ││
│  │  │  ┌─────────┐ ┌─────────┐ ┌─────────────────┐    │   ││
│  │  │  │ System  │ │  User   │ │   General      │    │   ││
│  │  │  │ Tablespace│ │ Tablespace│ │  Tablespace   │    │   ││
│  │  │  └─────────┘ └─────────┘ └─────────────────┘    │   ││
│  │  └─────────────────────────────────────────────────┘   ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### MVCC 与事务隔离级别

```sql
-- 查看当前隔离级别
SELECT @@transaction_isolation;

-- 设置隔离级别
SET SESSION transaction_isolation = 'REPEATABLE-READ';

-- InnoDB 隔离级别详解

-- READ UNCOMMITTED：最低级别，存在脏读
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
SELECT * FROM accounts;  -- 可能看到其他事务未提交的数据
COMMIT;

-- READ COMMITTED：避免脏读，但可能出现不可重复读
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT * FROM accounts;  -- 第一次读取
SELECT * FROM accounts;  -- 其他事务提交后，结果可能不同
COMMIT;

-- REPEATABLE READ（InnoDB 默认）：避免不可重复读
SET TRANSACTION ISOLATION LEVEL REPEATABLE-READ;
START TRANSACTION;
SELECT * FROM accounts;  -- 事务期间多次读取结果一致
COMMIT;

-- SERIALIZABLE：最高级别，强制串行执行
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
SELECT * FROM accounts;  -- 读取时会加锁
COMMIT;
```

### InnoDB 配置优化

```sql
-- 关键配置参数（在 my.cnf 中设置）

-- innodb_buffer_pool_size: 建议为系统内存的 50-70%
SET GLOBAL innodb_buffer_pool_size = 4294967296;  -- 4GB

-- innodb_log_file_size: 重做日志大小，建议 1-4GB
SET GLOBAL innodb_log_file_size = 1073741824;  -- 1GB

-- innodb_flush_log_at_trx_commit: 事务提交时刷新策略
-- 1: 每次提交都刷新（最安全，默认）
-- 2: 每秒刷新一次
-- 0: 交给后台刷新
SET GLOBAL innodb_flush_log_at_trx_commit = 1;

-- innodb_flush_method: I/O 刷新方式
-- O_DIRECT: Linux 推荐，避免双缓冲
SET GLOBAL innodb_flush_method = 'O_DIRECT';

-- 查看 InnoDB 状态
SHOW ENGINE INNODB STATUS;

-- 查看缓冲池使用情况
SELECT 
    pool_size,
    free_buffers,
    database_pages,
    pages_hashed,
    pages_old
FROM information_schema.innodb_buffer_pool_stats;
```

---

## 索引类型与选择指南

### 索引类型对比表

| 索引类型 | 适用场景 | 特点 | 创建语法 |
|---------|---------|------|----------|
| **B-Tree** | 默认索引，范围查询 | O(log n) 查找 | `INDEX idx (col)` |
| **复合索引** | 多列查询 | 最左前缀原则 | `INDEX idx (a, b, c)` |
| **唯一索引** | 唯一性约束 | 自动检查重复 | `UNIQUE INDEX idx (col)` |
| **全文索引** | 文本搜索 | 分词处理 | `FULLTEXT INDEX idx (col)` |
| **前缀索引** | 长文本列 | 节省空间 | `INDEX idx (col(10))` |
| **Hash** | 等值查询 | O(1) 查找 | `INDEX idx USING HASH (col)` |
| **空间索引** | 地理数据 | R-Tree | `SPATIAL INDEX idx (col)` |
| **向量索引** | AI 搜索 | HNSW | `VECTOR INDEX idx (col)` |

### B-Tree 索引详解

```sql
-- 创建普通索引
CREATE INDEX idx_users_email ON users(email);

-- 创建复合索引（最左前缀原则）
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date, status);

-- 复合索引使用规则：
-- ✅ SELECT * FROM orders WHERE user_id = 1;           -- 使用索引
-- ✅ SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';  -- 使用索引
-- ✅ SELECT * FROM orders WHERE user_id = 1 AND order_date > '2024-01-01';  -- 使用索引
-- ❌ SELECT * FROM orders WHERE order_date > '2024-01-01';  -- 不使用索引（违反最左前缀）

-- 查看索引使用情况
EXPLAIN SELECT * FROM orders WHERE user_id = 1;

-- 强制使用索引
SELECT * FROM orders USE INDEX (idx_orders_user_date) WHERE user_id = 1;
```

### 全文索引

```sql
-- 创建全文索引
ALTER TABLE articles ADD FULLTEXT INDEX idx_content (title, content);

-- 使用全文搜索
SELECT 
    title,
    MATCH(title, content) AGAINST('PostgreSQL MySQL' IN NATURAL LANGUAGE MODE) AS relevance
FROM articles
WHERE MATCH(title, content) AGAINST('database' IN NATURAL LANGUAGE MODE);

-- 布尔模式搜索
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('+PostgreSQL -MySQL +database' IN BOOLEAN MODE);

-- 查询扩展模式
SELECT * FROM articles
WHERE MATCH(title, content) AGAINST('database' WITH QUERY EXPANSION);
```

### 向量索引（MySQL 9.0+）

```sql
-- 创建向量列
ALTER TABLE documents ADD COLUMN embedding VECTOR(1536);

-- 创建 HNSW 索引
CREATE VECTOR INDEX idx_embedding ON documents(embedding) USING HNSW;

-- 配置索引参数
CREATE VECTOR INDEX idx_embedding ON documents(embedding) 
    USING HNSW 
    WITH (distance_type = 'cosine', m = 16, ef_construction = 200);

-- 向量搜索查询
SELECT 
    id,
    content,
    VECTOR_COSINE_SIMILARITY(embedding, '[query_vector]') AS similarity
FROM documents
ORDER BY embedding <=> '[query_vector]'
LIMIT 10;
```

### 索引选择决策树

```
                    开始
                      │
                      ▼
            ┌─────────────────┐
            │ 查询类型是什么？ │
            └─────────────────┘
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   ┌────────┐   ┌────────┐   ┌────────┐
   │等值查询│   │范围查询│   │文本搜索│
   └────────┘   └────────┘   └────────┘
        │             │             │
        ▼             ▼             ▼
   ┌────────┐   ┌────────┐   ┌────────┐
   │B-Tree  │   │B-Tree  │   │全文索引│
   │或Hash  │   │复合索引│   │        │
   └────────┘   └────────┘   └────────┘
```

---

## 事务与并发控制

### 事务基础

```sql
-- 显式事务
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE user_id = 2;

-- 检查是否成功
SELECT * FROM accounts WHERE user_id IN (1, 2);

-- 提交或回滚
COMMIT;
-- ROLLBACK;  -- 出错时回滚

-- 自动提交模式（默认开启）
SET autocommit = 0;  -- 关闭自动提交，手动控制事务

-- 保存点
START TRANSACTION;
INSERT INTO orders (...) VALUES (...);
SAVEPOINT after_order;

INSERT INTO order_items (...) VALUES (...);
-- 出错时可以回滚到保存点
ROLLBACK TO SAVEPOINT after_order;

COMMIT;
```

### 锁机制

```sql
-- 查看当前锁
SELECT 
    ENGINE_TRANSACTION_ID,
    OBJECT_NAME,
    INDEX_NAME,
    LOCK_MODE,
    LOCK_STATUS
FROM information_schema.INNODB_LOCKS;

-- 查看等待锁的进程
SELECT * FROM information_schema.INNODB_LOCK_WAITS;

-- 死锁处理
SHOW ENGINE INNODB STATUS;  -- 查看最近死锁信息

-- 设置锁等待超时
SET innodb_lock_wait_timeout = 5;  -- 5秒

-- 显式锁定
SELECT * FROM users WHERE id = 1 FOR UPDATE;  -- 排他锁
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE;  -- 共享锁
```

### 乐观锁与悲观锁

```sql
-- 乐观锁实现（使用版本号）
UPDATE users 
SET name = 'Alice', version = version + 1 
WHERE id = 1 AND version = 5;

-- 检查影响行数
-- 如果影响行数为 0，说明版本已更新，事务失败

-- 悲观锁实现（SELECT ... FOR UPDATE）
START TRANSACTION;
SELECT * FROM inventory WHERE product_id = 100 FOR UPDATE;
-- ... 业务逻辑 ...
UPDATE inventory SET stock = stock - 1 WHERE product_id = 100;
COMMIT;
```

---

## MySQL vs PostgreSQL 对比

### 详细对比表

| 维度 | MySQL 9.0 | PostgreSQL 17 |
|------|-----------|--------------|
| **定位** | Web 应用、快速开发 | 企业级、功能完备 |
| **学习曲线** | 平缓 | 较陡 |
| **性能** | 读密集优化 | 复杂查询优化 |
| **事务** | ACID 支持 | 完整 ACID + MVCC |
| **JSON 支持** | JSON 函数增强 | jsonb 高性能 |
| **向量搜索** | 原生 HNSW | pgvector 扩展 |
| **扩展性** | 有限 | 100+ 扩展 |
| **SQL 标准** | 部分兼容 | 高度兼容 |
| **复制** | 异步、半同步 | 物理、逻辑复制 |
| **分区** | 支持 | 声明式分区 |
| **窗口函数** | 支持 | 完全支持 |
| **CTE** | 8.0+ 支持 | 完全支持 |
| **GIS** | 通过插件 | PostGIS 扩展 |
| **社区** | Oracle 维护 | 独立社区 |

### 场景对比

| 场景 | MySQL | PostgreSQL |
|------|-------|------------|
| **WordPress/Drupal** | ✅ 首选 | 可用 |
| **高并发 API** | ✅ 性能好 | 可用 |
| **复杂报表** | 一般 | ✅ 强大 |
| **AI 向量搜索** | ✅ 9.0 原生 | ✅ pgvector |
| **JSON 存储** | 一般 | ✅ jsonb 优秀 |
| **地理信息** | 需要插件 | ✅ PostGIS |
| **大型数据仓库** | 不推荐 | ✅ 推荐 |

---

## 分区与分表策略

### 分区类型

```sql
-- RANGE 分区
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN MAXVALUE
);

-- LIST 分区
CREATE TABLE employees (
    id INT,
    department INT,
    name VARCHAR(50)
)
PARTITION BY LIST (department) (
    PARTITION p_sales VALUES IN (1, 2, 3),
    PARTITION p_tech VALUES IN (4, 5, 6),
    PARTITION p_hr VALUES IN (7, 8)
);

-- HASH 分区
CREATE TABLE logs (
    id BIGINT,
    created_at TIMESTAMP,
    message TEXT
)
PARTITION BY HASH (id) PARTITIONS 8;

-- KEY 分区（使用主键）
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
)
PARTITION BY KEY() PARTITIONS 4;

-- 子分区
CREATE TABLE sales (
    id INT,
    sale_date DATE,
    region VARCHAR(20),
    amount DECIMAL(10, 2)
)
PARTITION BY RANGE (YEAR(sale_date))
SUBPARTITION BY HASH (region) SUBPARTITIONS 2 (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);
```

### 分区管理

```sql
-- 查看分区信息
SELECT 
    PARTITION_NAME,
    TABLE_ROWS,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.PARTITIONS
WHERE TABLE_NAME = 'sales';

-- 添加分区
ALTER TABLE sales ADD PARTITION (
    PARTITION p2026 VALUES LESS THAN (2026)
);

-- 删除分区（同时删除数据）
ALTER TABLE sales DROP PARTITION p2022;

-- 重建分区
ALTER TABLE sales REORGANIZE PARTITION p2025 INTO (
    PARTITION p2025a VALUES LESS THAN (2025),
    PARTITION p2025b VALUES LESS THAN (2026)
);

-- 分区裁剪（自动优化查询）
EXPLAIN SELECT * FROM sales WHERE sale_date >= '2024-01-01';
-- 只会扫描相关分区
```

---

## 连接池与性能优化

### HikariCP 配置

HikariCP 是 Java 应用最常用的 MySQL 连接池：

```java
// application.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/ai_app?useSSL=false&serverTimezone=UTC
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      # 连接池大小（建议：CPU 核心数 * 2）
      maximum-pool-size: 20
      # 最小空闲连接
      minimum-idle: 5
      # 连接超时
      connection-timeout: 30000
      # 空闲超时
      idle-timeout: 600000
      # 最大生命周期
      max-lifetime: 1800000
      # 连接测试查询
      connection-test-query: SELECT 1
```

### MySQL 性能优化

```sql
-- 分析慢查询
SHOW VARIABLES LIKE 'slow_query_log';
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;  -- 超过 1 秒记录
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';

-- 查看慢查询日志
SHOW VARIABLES LIKE 'slow_query_log_file';

-- EXPLAIN 分析
EXPLAIN SELECT 
    u.name,
    COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;

-- 统计信息
ANALYZE TABLE users;
OPTIMIZE TABLE orders;

-- 查看进程
SHOW PROCESSLIST;
KILL 123;  -- 杀死进程
```

---

## AI 应用中的 MySQL

### AI 对话历史存储

```sql
-- 对话历史表
CREATE TABLE conversations (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    title VARCHAR(255),
    model VARCHAR(50) DEFAULT 'gpt-4o',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    metadata JSON,
    
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);

-- 消息表
CREATE TABLE messages (
    id CHAR(36) PRIMARY KEY,
    conversation_id CHAR(36) NOT NULL,
    role ENUM('system', 'user', 'assistant', 'tool') NOT NULL,
    content TEXT NOT NULL,
    tokens INT,
    model VARCHAR(50),
    metadata JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (conversation_id) REFERENCES conversations(id) ON DELETE CASCADE,
    INDEX idx_conversation_id (conversation_id),
    INDEX idx_created_at (created_at)
);

-- Token 使用统计
CREATE TABLE token_usage (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    model VARCHAR(50) NOT NULL,
    prompt_tokens INT NOT NULL,
    completion_tokens INT NOT NULL,
    total_tokens INT NOT NULL,
    cost DECIMAL(10, 6),
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_id (user_id),
    INDEX idx_recorded_at (recorded_at),
    INDEX idx_model (model)
);
```

### Python 连接示例

```python
import mysql.connector
from mysql.connector import pooling

# 连接池配置
connection_pool = pooling.MySQLConnectionPool(
    pool_name="ai_pool",
    pool_size=10,
    pool_reset_session=True,
    host="localhost",
    database="ai_app",
    user="root",
    password="password"
)

def save_conversation(user_id: str, messages: list[dict]) -> str:
    """保存对话"""
    conn = connection_pool.get_connection()
    cursor = conn.cursor()
    
    try:
        # 创建对话
        conversation_id = uuid.uuid4().hex
        cursor.execute(
            """
            INSERT INTO conversations (id, user_id, title)
            VALUES (%s, %s, %s)
            """,
            (conversation_id, user_id, messages[0]['content'][:50])
        )
        
        # 保存消息
        for msg in messages:
            cursor.execute(
                """
                INSERT INTO messages (id, conversation_id, role, content, tokens)
                VALUES (%s, %s, %s, %s, %s)
                """,
                (uuid.uuid4().hex, conversation_id, 
                 msg['role'], msg['content'], msg.get('tokens', 0))
            )
        
        conn.commit()
        return conversation_id
    
    finally:
        cursor.close()
        conn.close()

def get_conversation_history(conversation_id: str) -> list[dict]:
    """获取对话历史"""
    conn = connection_pool.get_connection()
    cursor = conn.cursor(dictionary=True)
    
    try:
        cursor.execute(
            """
            SELECT role, content, created_at
            FROM messages
            WHERE conversation_id = %s
            ORDER BY created_at ASC
            """,
            (conversation_id,)
        )
        return cursor.fetchall()
    
    finally:
        cursor.close()
        conn.close()
```

---

## 实战场景与代码示例

### 场景：构建 AI Token 统计系统

```sql
-- Token 使用统计表
CREATE TABLE token_stats (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id VARCHAR(255) NOT NULL,
    model VARCHAR(100) NOT NULL,
    request_type ENUM('prompt', 'completion', 'total') NOT NULL,
    token_count INT NOT NULL,
    cost_usd DECIMAL(10, 6),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_user_date (user_id, created_at),
    INDEX idx_model (model),
    INDEX idx_created_at (created_at)
);

-- 存储过程：统计每日使用
DELIMITER //

CREATE PROCEDURE get_daily_usage(
    IN p_user_id VARCHAR(255),
    IN p_days INT
)
BEGIN
    SELECT 
        DATE(created_at) AS date,
        model,
        SUM(CASE WHEN request_type = 'prompt' THEN token_count ELSE 0 END) AS prompt_tokens,
        SUM(CASE WHEN request_type = 'completion' THEN token_count ELSE 0 END) AS completion_tokens,
        SUM(CASE WHEN request_type = 'total' THEN token_count ELSE 0 END) AS total_tokens,
        SUM(cost_usd) AS total_cost
    FROM token_stats
    WHERE user_id = p_user_id
      AND created_at >= DATE_SUB(CURRENT_DATE, INTERVAL p_days DAY)
    GROUP BY DATE(created_at), model
    ORDER BY date DESC, model;
END //

DELIMITER ;

-- 调用存储过程
CALL get_daily_usage('user_123', 30);
```

### Python 集成

```python
from datetime import datetime, timedelta
import mysql.connector

class TokenTracker:
    """Token 使用追踪"""
    
    def __init__(self, config: dict):
        self.conn = mysql.connector.connect(**config)
    
    def record_usage(
        self,
        user_id: str,
        model: str,
        prompt_tokens: int,
        completion_tokens: int,
        cost_per_token: float = 0.00001
    ):
        """记录 Token 使用"""
        cursor = self.conn.cursor()
        
        total = prompt_tokens + completion_tokens
        cost = total * cost_per_token
        
        data = [
            (user_id, model, 'prompt', prompt_tokens, cost / 2),
            (user_id, model, 'completion', completion_tokens, cost / 2),
            (user_id, model, 'total', total, cost)
        ]
        
        cursor.executemany(
            """
            INSERT INTO token_stats 
            (user_id, model, request_type, token_count, cost_usd)
            VALUES (%s, %s, %s, %s, %s)
            """,
            data
        )
        
        self.conn.commit()
        cursor.close()
    
    def get_usage_summary(self, user_id: str, days: int = 30) -> dict:
        """获取使用摘要"""
        cursor = self.conn.cursor(dictionary=True)
        
        cursor.execute(
            """
            SELECT 
                COUNT(*) AS total_requests,
                SUM(token_count) AS total_tokens,
                SUM(cost_usd) AS total_cost,
                model
            FROM token_stats
            WHERE user_id = %s
              AND created_at >= DATE_SUB(NOW(), INTERVAL %s DAY)
            GROUP BY model
            """,
            (user_id, days)
        )
        
        results = cursor.fetchall()
        cursor.close()
        
        return {
            'period_days': days,
            'breakdown': results,
            'total': {
                'requests': sum(r['total_requests'] for r in results),
                'tokens': sum(r['total_tokens'] for r in results),
                'cost': sum(r['total_cost'] for r in results)
            }
        }
```

---

## 选型建议

### 何时选择 MySQL

| 场景 | 推荐程度 | 原因 |
|------|----------|------|
| **Web 应用/网站** | ⭐⭐⭐⭐⭐ | 成熟、生态完善 |
| **WordPress/Drupal** | ⭐⭐⭐⭐⭐ | 默认数据库 |
| **高并发读密集** | ⭐⭐⭐⭐⭐ | 性能优秀 |
| **API 后端** | ⭐⭐⭐⭐ | 简单快速 |
| **AI 向量搜索** | ⭐⭐⭐⭐ | 9.0 原生支持 |
| **复杂分析查询** | ⭐⭐⭐ | 功能有限 |
| **需要 JSON 灵活性** | ⭐⭐⭐ | 能力有限 |
| **大型数据仓库** | ⭐⭐ | 不适合 |

### 关键配置建议

```ini
# my.cnf 关键配置

[mysqld]
# 字符集
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# InnoDB
innodb_buffer_pool_size = 4G
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT

# 连接
max_connections = 500
wait_timeout = 600

# 查询缓存（MySQL 8.0 已移除）
# query_cache_type = 0

# 慢查询
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
```

---

> [!TIP]
> 对于简单的 AI 应用，MySQL 9.0 的向量搜索功能已经足够。对于需要更高级向量操作（如混合搜索）的场景，可以考虑 PostgreSQL + pgvector。

---

> [!SUCCESS]
> 本文档全面介绍了 MySQL 的核心特性、9.0 新功能、InnoDB 引擎、索引系统、事务机制，以及在 AI 应用中的使用场景。MySQL 凭借其简单性和成熟生态，仍然是 Web 应用和轻量级 AI 应用的优秀选择。

---

## 完整安装与环境配置

### 安装方法

```bash
# macOS
brew install mysql
brew services start mysql

# Ubuntu/Debian
sudo apt update
sudo apt install mysql-server
sudo mysql_secure_installation

# Docker
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -e MYSQL_DATABASE=mydb \
  -p 3306:3306 \
  mysql:9.0

# Docker Compose
version: "3.8"
services:
  mysql:
    image: mysql:9.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: mydb
      MYSQL_USER: app
      MYSQL_PASSWORD: apppassword
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./my.cnf:/etc/mysql/conf.d/custom.cnf
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:
```

### MySQL 客户端

```bash
# 命令行连接
mysql -h localhost -u root -p
mysql -h localhost -u app -p mydb

# 常用命令
SHOW DATABASES;
USE database_name;
SHOW TABLES;
DESCRIBE table_name;
SHOW INDEX FROM table_name;
SHOW PROCESSLIST;
SHOW STATUS;
SHOW VARIABLES LIKE 'max_connections';

# 导出导入
mysqldump -u root -p mydb > backup.sql
mysql -u root -p mydb < backup.sql

# 导出特定表
mysqldump -u root -p mydb users posts > tables.sql

# 导出数据（CSV）
SELECT * FROM users INTO OUTFILE '/tmp/users.csv'
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n';

# 导入数据
LOAD DATA INFILE '/tmp/users.csv'
INTO TABLE users
FIELDS TERMINATED BY ',' 
ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

### 配置文件示例

```ini
# /etc/mysql/conf.d/custom.cnf
[mysqld]
# 字符集
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# InnoDB 配置
innodb_buffer_pool_size = 4G
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 1
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1
innodb_io_capacity = 2000
innodb_io_capacity_max = 4000

# 连接配置
max_connections = 500
wait_timeout = 600
interactive_timeout = 600
max_connect_errors = 100000

# 缓存配置
query_cache_type = 0  # MySQL 8.0 已移除
key_buffer_size = 256M
sort_buffer_size = 2M
read_buffer_size = 2M
join_buffer_size = 2M

# 日志配置
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1
log_queries_not_using_indexes = 1

# 二进制日志（用于复制和恢复）
log_bin = /var/log/mysql/mysql-bin
expire_logs_days = 7
max_binlog_size = 1G
binlog_format = ROW

[client]
default-character-set = utf8mb4
```

---

## 数据库设计模式

### 常见设计模式

```sql
-- 1. 软删除设计
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    username VARCHAR(50) NOT NULL UNIQUE,
    deleted_at TIMESTAMP NULL DEFAULT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_deleted (deleted_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 视图：只显示未删除的记录
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

-- 2. 层级结构设计（邻接表）
CREATE TABLE categories (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id BIGINT UNSIGNED NULL,
    level INT UNSIGNED DEFAULT 0,
    path VARCHAR(255) DEFAULT '',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (parent_id) REFERENCES categories(id) ON DELETE SET NULL,
    INDEX idx_parent (parent_id),
    INDEX idx_path (path)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 3. 多租户设计
CREATE TABLE tenants (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(50) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE posts (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    tenant_id BIGINT UNSIGNED NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (tenant_id) REFERENCES tenants(id) ON DELETE CASCADE,
    INDEX idx_tenant (tenant_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 4. 审计表设计
CREATE TABLE audit_log (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    record_id BIGINT UNSIGNED NOT NULL,
    action ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    old_data JSON,
    new_data JSON,
    changed_by BIGINT UNSIGNED,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_table_record (table_name, record_id),
    INDEX idx_changed_at (changed_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 触发器实现审计
DELIMITER //

CREATE TRIGGER users_audit_insert
AFTER INSERT ON users
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, record_id, action, new_data)
    VALUES ('users', NEW.id, 'INSERT', JSON_OBJECT(
        'email', NEW.email,
        'username', NEW.username
    ));
END//

CREATE TRIGGER users_audit_update
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, record_id, action, old_data, new_data)
    VALUES ('users', NEW.id, 'UPDATE', JSON_OBJECT(
        'email', OLD.email,
        'username', OLD.username
    ), JSON_OBJECT(
        'email', NEW.email,
        'username', NEW.username
    ));
END//

CREATE TRIGGER users_audit_delete
AFTER DELETE ON users
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, record_id, action, old_data)
    VALUES ('users', OLD.id, 'DELETE', JSON_OBJECT(
        'email', OLD.email,
        'username', OLD.username
    ));
END//

DELIMITER ;
```

---

## 查询优化实战

### 慢查询分析

```sql
-- 查看慢查询日志
SHOW VARIABLES LIKE 'slow_query_log%';
SHOW VARIABLES LIKE 'long_query_time';

-- 查看当前连接
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- 终止连接
KILL CONNECTION 123;

-- 查看查询统计
SHOW GLOBAL STATUS LIKE 'Com_select';
SHOW GLOBAL STATUS LIKE 'Com_insert';
SHOW GLOBAL STATUS LIKE 'Questions';

-- 查看 InnoDB 状态
SHOW ENGINE INNODB STATUS;

-- 查看锁等待
SELECT 
    r.trx_id,
    r.trx_mysql_thread_id,
    r.trx_query,
    r.trx_state,
    r.trx_started,
    r.trx_rows_locked,
    r.trx_tables_locked,
    r.trx_wait_started,
    r.trx_wait_started - r.trx_started AS wait_duration
FROM information_schema.INNODB_TRX r
ORDER BY r.trx_wait_started;
```

### 索引优化

```sql
-- 1. B-tree 索引（默认）
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC);

-- 2. 唯一索引
CREATE UNIQUE INDEX idx_users_email ON users(email);
CREATE UNIQUE INDEX idx_posts_slug ON posts(slug);

-- 3. 前缀索引（用于长字符串）
CREATE INDEX idx_users_name_prefix ON users(name(50));

-- 4. 全文索引
ALTER TABLE posts ADD FULLTEXT INDEX ft_title_content (title, content);

-- 搜索示例
SELECT * FROM posts 
WHERE MATCH(title, content) AGAINST('search terms' IN NATURAL LANGUAGE MODE);

-- 布尔模式搜索
SELECT * FROM posts 
WHERE MATCH(title, content) AGAINST('+mysql -postgresql' IN BOOLEAN MODE);

-- 5. 复合索引设计
-- 查询: WHERE status = 'active' AND created_at > '2024-01-01'
-- 选择性高的列放前面
CREATE INDEX idx_users_status_created ON users(status, created_at);

-- 6. 向量索引（MySQL 9.0）
CREATE TABLE embeddings (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    entity_type VARCHAR(50) NOT NULL,
    entity_id BIGINT UNSIGNED NOT NULL,
    embedding VECTOR(1536) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_entity (entity_type, entity_id),
    VECTOR INDEX idx_embedding (embedding) USING HNSW
);

-- 向量搜索
SELECT 
    id,
    entity_type,
    entity_id,
    VECTOR_COSINE_DISTANCE(embedding, '[0.1, 0.2, ...]') AS distance
FROM embeddings
WHERE entity_type = 'documents'
ORDER BY distance ASC
LIMIT 5;

-- 7. 索引使用分析
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

EXPLAIN ANALYZE
SELECT u.*, COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON p.author_id = u.id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id;
```

### 查询优化技巧

```sql
-- 1. 避免 SELECT *
SELECT id, email, username FROM users WHERE id = 1;

-- 2. 使用 LIMIT 避免大结果集
SELECT * FROM posts ORDER BY created_at LIMIT 100;

-- 3. 批量插入
INSERT INTO users (email, username) VALUES
    ('user1@example.com', 'user1'),
    ('user2@example.com', 'user2'),
    ('user3@example.com', 'user3');

-- 4. 使用 EXPLAIN
EXPLAIN SELECT * FROM posts WHERE published = 1;

-- 5. 分页优化
-- ❌ 偏移分页
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 10000;

-- ✅ 游标分页
SELECT * FROM posts 
WHERE id > 10000 
ORDER BY id 
LIMIT 20;

-- ✅ 时间戳分页
SELECT * FROM posts 
WHERE created_at < '2024-01-01'
ORDER BY created_at DESC 
LIMIT 20;

-- 6. JSON 查询优化
CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    data JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- JSON 索引（虚拟列）
ALTER TABLE orders ADD COLUMN customer_email VARCHAR(255) 
GENERATED ALWAYS AS (data->>'$.customer.email') STORED;

CREATE INDEX idx_orders_customer_email ON orders(customer_email);

-- 查询
SELECT * FROM orders WHERE customer_email = 'test@example.com';
```

---

## 事务与锁

### 事务控制

```sql
-- 开始事务
START TRANSACTION;
-- 或
BEGIN;

-- 设置保存点
SAVEPOINT sp1;

-- 回滚到保存点
ROLLBACK TO SAVEPOINT sp1;

-- 提交
COMMIT;

-- 回滚
ROLLBACK;

-- 设置事务隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 自动提交模式
SET autocommit = 0;  -- 关闭自动提交
SET autocommit = 1;  -- 开启自动提交
```

### 锁机制

```sql
-- 1. 共享锁和排他锁
-- 共享锁：其他事务可以读，但不能写
SELECT * FROM users WHERE id = 1 LOCK IN SHARE MODE;

-- 排他锁：其他事务不能读也不能写
SELECT * FROM users WHERE id = 1 FOR UPDATE;

-- 2. 记录锁
-- 对单行加锁
SELECT * FROM users WHERE id = 1 FOR UPDATE;

-- 3. 间隙锁
-- 锁定范围
SELECT * FROM posts 
WHERE id BETWEEN 10 AND 20 
FOR UPDATE;

-- 4. 意向锁
-- 表级锁，表明事务有意对某行加锁
LOCK TABLES users WRITE;
LOCK TABLES users READ;
UNLOCK TABLES;

-- 5. 元数据锁
-- DDL 操作自动加锁
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- 6. 查看锁等待
SELECT 
    r.trx_id,
    r.trx_mysql_thread_id,
    r.trx_query,
    l.lock_index,
    l.lock_data
FROM information_schema.INNODB_TRX r
JOIN information_schema.INNODB_LOCKS l ON l.lock_trx_id = r.trx_id
WHERE l.lock_type = 'RECORD';

-- 7. 死锁处理
SHOW ENGINE INNODB STATUS;
-- 查找最近的死锁信息
```

---

## 主从复制

### 主库配置

```ini
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin
binlog_format = ROW
sync_binlog = 1
expire_logs_days = 7
max_binlog_size = 1G

# GTID 复制
gtid_mode = ON
enforce_gtid_consistency = ON

# 只读设置
read_only = 1
super_read_only = 1
```

### 从库配置

```ini
[mysqld]
server-id = 2
relay_log = /var/log/mysql/mysql-relay-bin
log_slave_updates = 1
read_only = 1
super_read_only = 1

# GTID 复制
gtid_mode = ON
enforce_gtid_consistency = ON
```

### 复制命令

```sql
-- 主库：创建复制用户
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

-- 从库：配置复制
CHANGE MASTER TO
    MASTER_HOST = 'master_host',
    MASTER_USER = 'repl',
    MASTER_PASSWORD = 'password',
    MASTER_PORT = 3306,
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 123,
    MASTER_AUTO_POSITION = 1;

-- 启动复制
START SLAVE;

-- 查看复制状态
SHOW SLAVE STATUS\G

-- 停止复制
STOP SLAVE;

-- 重置复制
RESET SLAVE ALL;
```

---

## 备份与恢复

### 备份类型对比

| 备份类型 | 工具 | 特点 | 适用场景 |
|---------|------|------|----------|
| **逻辑备份** | mysqldump | 跨版本、可选择性恢复 | 应用级恢复 |
| **物理备份** | MySQL Enterprise Backup | 快速、完整 | 灾难恢复 |
| **增量备份** | mysqldump + 增量日志 | 高效 | 大型数据库 |
| **时间点恢复** | 二进制日志 | 精确恢复 | 误操作恢复 |

### mysqldump 使用

```bash
# 备份单个数据库
mysqldump -u root -p --single-transaction mydb > backup.sql

# 备份所有数据库
mysqldump -u root -p --all-databases > all_databases.sql

# 备份特定表
mysqldump -u root -p mydb users posts > tables.sql

# 备份结构（不包含数据）
mysqldump -u root -p --no-data mydb > schema.sql

# 备份数据（不包含结构）
mysqldump -u root -p --no-create-info mydb > data.sql

# 压缩备份
mysqldump -u root -p --single-transaction mydb | gzip > backup.sql.gz

# 带事件的备份
mysqldump -u root -p --single-transaction --events --routines --triggers mydb > backup.sql

# 只备份存储过程和函数
mysqldump -u root -p --no-data --routines --events mydb > procedures.sql

# 远程备份
mysqldump -h remote_host -u root -p --single-transaction mydb > backup.sql
```

### 恢复操作

```bash
# 恢复纯 SQL 格式备份
mysql -u root -p mydb < backup.sql

# 恢复压缩备份
gunzip < backup.sql.gz | mysql -u root -p mydb

# 恢复所有数据库
mysql -u root -p < all_databases.sql

# 从时间点恢复
mysqlbinlog --stop-datetime="2024-01-15 10:00:00" mysql-bin.000001 | mysql -u root -p

# 只恢复特定表
mysql -u root -p mydb -e "DROP TABLE IF EXISTS users;"
mysql -u root -p mydb < users_table.sql
```

### 二进制日志（Binlog）

```sql
-- 查看 binlog 状态
SHOW MASTER STATUS;
SHOW BINARY LOGS;

-- 查看 binlog 内容
SHOW BINLOG EVENTS IN 'mysql-bin.000001';

-- 设置 binlog 保留时间
SET GLOBAL binlog_expire_logs_seconds = 604800;  -- 7 天

-- 刷新 binlog
FLUSH LOGS;

-- 查看当前 binlog 配置
SHOW VARIABLES LIKE 'binlog%';
```

```ini
# my.cnf 配置 binlog
[mysqld]
server-id = 1
log_bin = /var/log/mysql/mysql-bin
binlog_format = ROW
binlog_row_image = FULL
sync_binlog = 1
expire_logs_days = 7
max_binlog_size = 1G
```

### XtraBackup（热备份）

```bash
# 全量备份
xtrabackup --backup --target-dir=/backup/full/

# 增量备份
xtrabackup --backup --target-dir=/backup/inc1/ --incremental-basedir=/backup/full/

# 准备备份（恢复前）
xtrabackup --prepare --target-dir=/backup/full/

# 恢复
xtrabackup --copy-back --target-dir=/backup/full/

# 流式备份到远程
xtrabackup --backup --stream=xbstream | ssh user@remote "xbstream -x -C /backup/"
```

### 备份脚本

```bash
#!/bin/bash
# backup.sh - MySQL 备份脚本

set -e

BACKUP_DIR="/var/backups/mysql"
DATE=$(date +%Y%m%d_%H%M%S)
MYSQL_USER="root"
MYSQL_PASSWORD="password"
MYSQL_HOST="localhost"
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

# 逻辑备份
echo "Starting logical backup..."
mysqldump -h "$MYSQL_HOST" -u "$MYSQL_USER" -p"$MYSQL_PASSWORD" \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    --all-databases | gzip > "$BACKUP_DIR/all_db_$DATE.sql.gz"

# 清理旧备份
find "$BACKUP_DIR" -name "*.sql.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: all_db_$DATE.sql.gz"
```

---

## Python 集成

### mysql-connector-python

```python
import mysql.connector
from mysql.connector import pooling, Error

# 基本连接
conn = mysql.connector.connect(
    host="localhost",
    database="mydb",
    user="root",
    password="password"
)

cursor = conn.cursor(dictionary=True)

# 执行查询
cursor.execute("SELECT * FROM users WHERE id = %s", (1,))
result = cursor.fetchone()

# 事务处理
try:
    conn.start_transaction()
    cursor.execute("INSERT INTO users (name, email) VALUES (%s, %s)", ("Alice", "alice@example.com"))
    cursor.execute("INSERT INTO orders (user_id, total) VALUES (%s, %s)", (cursor.lastrowid, 99.99))
    conn.commit()
except Error as e:
    conn.rollback()
    print(f"Error: {e}")

# 使用 with 语句
with mysql.connector.connect(**config) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT * FROM users LIMIT 10")
        users = cur.fetchall()
```

### PyMySQL

```python
import pymysql
from pymysql.cursors import DictCursor
import contextlib

# 基本连接
conn = pymysql.connect(
    host='localhost',
    database='mydb',
    user='root',
    password='password',
    charset='utf8mb4'
)

with conn.cursor() as cursor:
    cursor.execute("SELECT * FROM users WHERE id = %s", (1,))
    user = cursor.fetchone()

# 事务处理
try:
    with conn.cursor() as cursor:
        cursor.execute("START TRANSACTION")
        cursor.execute("INSERT INTO users (name, email) VALUES (%s, %s)", ("Bob", "bob@example.com"))
        user_id = cursor.lastrowid
        cursor.execute("INSERT INTO orders (user_id, total) VALUES (%s, %s)", (user_id, 149.99))
        conn.commit()
except Exception as e:
    conn.rollback()
    raise

# 连接池
pool = pymysql.connect(
    host='localhost',
    user='root',
    password='password',
    database='mydb',
    charset='utf8mb4',
    cursorclass=DictCursor
)

@contextlib.contextmanager
def get_connection():
    conn = pool.connection()
    try:
        yield conn
    finally:
        conn.close()
```

### SQLAlchemy

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, Enum
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

# 创建引擎
engine = create_engine('mysql+pymysql://root:password@localhost:3306/mydb')

Session = sessionmaker(bind=engine)
Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, nullable=False)
    status = Column(Enum('active', 'inactive', 'suspended'), default='active')
    created_at = Column(DateTime, default=datetime.utcnow)
    
    def __repr__(self):
        return f"<User(name='{self.name}', email='{self.email}')>"

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
    func.max(User.created_at).label('latest')
).filter(User.status == 'active').first()

# 联表查询
class Order(Base):
    __tablename__ = 'orders'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    total = Column(Numeric(10, 2))
    created_at = Column(DateTime, default=datetime.utcnow)

User.orders = relationship("Order", back_populates="user")
Order.user = relationship("User", back_populates="orders")

# 使用联表
results = session.query(User, Order).join(Order).filter(
    User.name == 'Alice'
).all()
```

### aiomysql（异步）

```python
import asyncio
import aiomysql

async def main():
    # 创建连接池
    pool = await aiomysql.create_pool(
        host='localhost',
        user='root',
        password='password',
        db='mydb',
        minsize=5,
        maxsize=20
    )
    
    async with pool.acquire() as conn:
        async with conn.cursor(aiomysql.DictCursor) as cur:
            # 查询
            await cur.execute("SELECT * FROM users WHERE id = %s", (1,))
            user = await cur.fetchone()
            print(user)
            
            # 批量查询
            await cur.execute("SELECT * FROM users LIMIT 10")
            users = await cur.fetchall()
            
            # 执行
            await cur.execute(
                "INSERT INTO users (name, email) VALUES (%s, %s)",
                ('Bob', 'bob@example.com')
            )
            await conn.commit()
            
            # 事务
            async with conn.cursor() as cur:
                try:
                    await cur.execute("START TRANSACTION")
                    await cur.execute("INSERT INTO orders (user_id, total) VALUES (%s, %s)", (1, 99.99))
                    await conn.commit()
                except Exception:
                    await conn.rollback()
                    raise
    
    pool.close()
    await pool.wait_closed()

asyncio.run(main())
```

---

## 常见陷阱与最佳实践

### 陷阱 1：忽视字符集

```sql
-- ❌ 错误：使用 utf8（实际是 utf8mb3）
CREATE TABLE users (
    name VARCHAR(100) CHARACTER SET utf8  -- 不支持 emoji
);

-- ✅ 正确：使用 utf8mb4
CREATE TABLE users (
    name VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 陷阱 2：使用 VARCHAR(255) 过度

```sql
-- ❌ 错误：所有字符串都用 VARCHAR(255)
CREATE TABLE products (
    sku VARCHAR(255),     -- SKU 通常很短
    name VARCHAR(255),    -- 可能很长
    description TEXT,     -- 描述应该用 TEXT
    url VARCHAR(255)      -- URL 可能很长
);

-- ✅ 正确：根据实际数据选择大小
CREATE TABLE products (
    sku VARCHAR(50),
    name VARCHAR(200),
    description TEXT,
    url VARCHAR(2000)
);
```

### 陷阱 3：忽视索引维护

```sql
-- ❌ 错误：不维护索引
-- 索引会随着数据增长而变得碎片化

-- ✅ 正确：定期优化表
OPTIMIZE TABLE users;

-- 检查表状态
CHECK TABLE users;
REPAIR TABLE users;

-- 重建索引
ALTER TABLE users DROP INDEX idx_email, ADD INDEX idx_email (email);
```

### 最佳实践清单

1. **使用 InnoDB**：MySQL 8.0+ 的默认引擎，支持事务和外键。
2. **UTF8MB4**：使用 utf8mb4 字符集支持完整 Unicode。
3. **合理索引**：为常用查询添加索引，但不要过度。
4. **主键设计**：使用自增 BIGINT 作为主键，或使用 UUID。
5. **避免 NULL**：尽量使用 NOT NULL，设置默认值。
6. **适当数据类型**：选择最合适的数据类型。
7. **定期维护**：OPTIMIZE TABLE、ANALYZE TABLE。
8. **连接池**：使用连接池避免连接数过多。
9. **备份策略**：定期全量备份 + 增量备份。
10. **监控**：监控慢查询、连接数、缓冲池命中率。

---

## 与其他数据库对比

### MySQL vs PostgreSQL

| 特性 | MySQL | PostgreSQL |
|------|-------|------------|
| **SQL 标准** | 部分遵循 | 完全遵循 |
| **事务** | ACID 支持 | ACID 完全支持 |
| **并发** | MVCC + 锁 | MVCC |
| **索引** | B-tree、全文、JSON | 多种类型 |
| **JSON 支持** | JSON 类型 | jsonb（高性能） |
| **向量搜索** | 9.0 原生支持 | pgvector |
| **存储过程** | 支持 | 支持 |
| **扩展性** | 有限 | 强大（扩展） |
| **复制** | 主从/组复制 | 流复制 |

### MySQL vs MongoDB

| 特性 | MySQL | MongoDB |
|------|-------|---------|
| **模型** | 关系型 | 文档型 |
| **事务** | ACID | ACID（分片副本集） |
| **扩展性** | 垂直扩展 | 水平扩展 |
| **JOIN** | 完整支持 | 有限支持 |
| **一致性** | 强一致 | 可调一致 |
| **查询** | SQL | MQL |

### MySQL vs SQLite

| 特性 | MySQL | SQLite |
|------|-------|--------|
| **类型** | 服务器 | 嵌入式 |
| **并发** | 多连接 | 文件锁 |
| **性能** | 高并发优秀 | 单机优秀 |
| **容量** | 无限制 | TB 级 |
| **部署** | 复杂 | 简单 |

---

## MySQL 存储过程与函数

### 基础语法

```sql
-- 1. 存储函数（必须返回值）
DELIMITER //

CREATE FUNCTION get_user_count()
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
    DECLARE cnt INT;
    SELECT COUNT(*) INTO cnt FROM users;
    RETURN cnt;
END //

DELIMITER ;

-- 调用函数
SELECT get_user_count();

-- 2. 存储过程（不返回值）
DELIMITER //

CREATE PROCEDURE get_users_by_status(
    IN p_status VARCHAR(20)
)
BEGIN
    SELECT * FROM users WHERE status = p_status;
END //

DELIMITER ;

-- 调用存储过程
CALL get_users_by_status('active');

-- 3. 带 OUT 参数的存储过程
DELIMITER //

CREATE PROCEDURE get_user_stats(
    OUT p_total INT,
    OUT p_active INT,
    OUT p_inactive INT
)
BEGIN
    SELECT COUNT(*) INTO p_total FROM users;
    SELECT COUNT(*) INTO p_active FROM users WHERE status = 'active';
    SELECT COUNT(*) INTO p_inactive FROM users WHERE status = 'inactive';
END //

DELIMITER ;

-- 调用并获取输出参数
CALL get_user_stats(@total, @active, @inactive);
SELECT @total, @active, @inactive;

-- 4. 带 INOUT 参数的存储过程
DELIMITER //

CREATE PROCEDURE increment_counter(
    INOUT counter INT,
    IN increment INT
)
BEGIN
    SET counter = counter + increment;
END //

DELIMITER ;

-- 调用
SET @my_counter = 10;
CALL increment_counter(@my_counter, 5);
SELECT @my_counter;  -- 输出: 15
```

### 条件逻辑与循环

```sql
DELIMITER //

-- IF-THEN-ELSE
CREATE PROCEDURE grade_student(
    IN p_score INT,
    OUT p_grade CHAR(1)
)
BEGIN
    IF p_score >= 90 THEN
        SET p_grade = 'A';
    ELSEIF p_score >= 80 THEN
        SET p_grade = 'B';
    ELSEIF p_score >= 70 THEN
        SET p_grade = 'C';
    ELSEIF p_score >= 60 THEN
        SET p_grade = 'D';
    ELSE
        SET p_grade = 'F';
    END IF;
END //

-- CASE 语句
CREATE FUNCTION get_discount_rate(
    p_membership_level VARCHAR(20)
)
RETURNS DECIMAL(3,2)
DETERMINISTIC
BEGIN
    DECLARE rate DECIMAL(3,2);
    
    CASE p_membership_level
        WHEN 'gold' THEN SET rate = 0.20;
        WHEN 'silver' THEN SET rate = 0.10;
        WHEN 'bronze' THEN SET rate = 0.05;
        ELSE SET rate = 0.00;
    END CASE;
    
    RETURN rate;
END //

-- WHILE 循环
CREATE PROCEDURE generate_numbers(
    IN p_limit INT
)
BEGIN
    DECLARE i INT DEFAULT 1;
    
    DROP TEMPORARY TABLE IF EXISTS numbers;
    CREATE TEMPORARY TABLE numbers (value INT);
    
    WHILE i <= p_limit DO
        INSERT INTO numbers VALUES (i);
        SET i = i + 1;
    END WHILE;
    
    SELECT * FROM numbers;
END //

-- REPEAT 循环（至少执行一次）
CREATE PROCEDURE repeat_loop_example(IN p_count INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    
    REPEAT
        SET i = i + 1;
        INSERT INTO temp_table VALUES (i);
    UNTIL i >= p_count
    END REPEAT;
END //

-- LOOP 循环
CREATE PROCEDURE find_prime(
    IN p_limit INT
)
BEGIN
    DECLARE i INT DEFAULT 2;
    DECLARE j INT;
    DECLARE is_prime BOOLEAN;
    
    DROP TEMPORARY TABLE IF EXISTS primes;
    CREATE TEMPORARY TABLE primes (value INT);
    
    main_loop: LOOP
        IF i > p_limit THEN
            LEAVE main_loop;
        END IF;
        
        SET is_prime = TRUE;
        SET j = 2;
        
        inner_loop: WHILE j <= SQRT(i) DO
            IF i % j = 0 THEN
                SET is_prime = FALSE;
                LEAVE inner_loop;
            END IF;
            SET j = j + 1;
        END WHILE inner_loop;
        
        IF is_prime THEN
            INSERT INTO primes VALUES (i);
        END IF;
        
        SET i = i + 1;
    END LOOP main_loop;
    
    SELECT * FROM primes;
END //

-- ITERATE（跳过当前迭代）和 LEAVE（退出循环）
CREATE PROCEDURE iterate_example(IN p_limit INT)
BEGIN
    DECLARE i INT DEFAULT 0;
    
    loop_label: LOOP
        SET i = i + 1;
        
        IF i = 5 THEN
            ITERATE loop_label;  -- 跳过 5
        END IF;
        
        IF i > p_limit THEN
            LEAVE loop_label;  -- 退出循环
        END IF;
        
        INSERT INTO temp_table VALUES (i);
    END LOOP loop_label;
END //
```

### 游标使用

```sql
DELIMITER //

CREATE PROCEDURE process_user_orders()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE v_user_id BIGINT;
    DECLARE v_email VARCHAR(255);
    DECLARE v_order_count INT;
    
    -- 声明游标
    DECLARE user_cursor CURSOR FOR
        SELECT id, email FROM users WHERE status = 'active';
    
    -- 声明继续处理器
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    DROP TEMPORARY TABLE IF EXISTS user_order_summary;
    CREATE TEMPORARY TABLE user_order_summary (
        user_id BIGINT,
        email VARCHAR(255),
        order_count INT
    );
    
    OPEN user_cursor;
    
    read_loop: LOOP
        FETCH user_cursor INTO v_user_id, v_email;
        
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        SELECT COUNT(*) INTO v_order_count
        FROM orders
        WHERE user_id = v_user_id;
        
        INSERT INTO user_order_summary VALUES (v_user_id, v_email, v_order_count);
    END LOOP read_loop;
    
    CLOSE user_cursor;
    
    SELECT * FROM user_order_summary;
END //

DELIMITER ;
```

### 错误处理与异常

```sql
DELIMITER //

CREATE PROCEDURE safe_transfer(
    IN from_account_id BIGINT,
    IN to_account_id BIGINT,
    IN amount DECIMAL(15,2),
    OUT p_success BOOLEAN,
    OUT p_message VARCHAR(255)
)
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        GET DIAGNOSTICS CONDITION 1
            p_message = MESSAGE_TEXT;
        ROLLBACK;
        SET p_success = FALSE;
    END;
    
    DECLARE EXIT HANDLER FOR SQLSTATE '45000'
    BEGIN
        SET p_success = FALSE;
        SET p_message = 'Insufficient funds';
        ROLLBACK;
    END;
    
    START TRANSACTION;
    
    -- 检查余额
    IF (SELECT balance FROM accounts WHERE id = from_account_id) < amount THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Insufficient funds';
    END IF;
    
    -- 扣款
    UPDATE accounts
    SET balance = balance - amount
    WHERE id = from_account_id;
    
    -- 存款
    UPDATE accounts
    SET balance = balance + amount
    WHERE id = to_account_id;
    
    -- 记录交易
    INSERT INTO transactions (from_account, to_account, amount)
    VALUES (from_account_id, to_account_id, amount);
    
    COMMIT;
    SET p_success = TRUE;
    SET p_message = 'Transfer completed';
END //

DELIMITER ;
```

### 动态 SQL

```sql
DELIMITER //

CREATE PROCEDURE dynamic_query(
    IN p_table_name VARCHAR(64),
    IN p_column_name VARCHAR(64),
    IN p_value VARCHAR(255)
)
BEGIN
    SET @sql_query = CONCAT(
        'SELECT * FROM ',
        p_table_name,
        ' WHERE ',
        p_column_name,
        ' = ''',
        p_value,
        ''''
    );
    
    PREPARE stmt FROM @sql_query;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END //

-- 使用 prepared statement 直接执行
CREATE PROCEDURE execute_dynamic(
    IN p_sql VARCHAR(1000)
)
BEGIN
    SET @sql = p_sql;
    PREPARE stmt FROM @sql;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
END //

DELIMITER ;

-- 调用
CALL execute_dynamic('SELECT COUNT(*) FROM users');
```

### 触发器

```sql
-- 1. BEFORE INSERT 触发器
CREATE TRIGGER before_user_insert
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    SET NEW.created_at = NOW();
    SET NEW.updated_at = NOW();
    IF NEW.status IS NULL THEN
        SET NEW.status = 'active';
    END IF;
END;

-- 2. AFTER INSERT 触发器
CREATE TRIGGER after_order_insert
AFTER INSERT ON orders
FOR EACH ROW
BEGIN
    -- 更新用户订单计数
    UPDATE user_stats
    SET order_count = order_count + 1,
        total_spent = total_spent + NEW.total
    WHERE user_id = NEW.user_id;
    
    -- 发送通知（简化）
    INSERT INTO notifications (user_id, message)
    VALUES (NEW.user_id, CONCAT('New order #', NEW.id, ' created'));
END;

-- 3. BEFORE UPDATE 触发器
CREATE TRIGGER before_user_update
BEFORE UPDATE ON users
FOR EACH ROW
BEGIN
    SET NEW.updated_at = NOW();
    
    -- 记录状态变更
    IF OLD.status != NEW.status THEN
        INSERT INTO status_history (user_id, old_status, new_status)
        VALUES (OLD.id, OLD.status, NEW.status);
    END IF;
END;

-- 4. AFTER UPDATE 触发器
CREATE TRIGGER after_user_update
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    IF OLD.email != NEW.email THEN
        INSERT INTO email_change_log (user_id, old_email, new_email)
        VALUES (OLD.id, OLD.email, NEW.email);
    END IF;
END;

-- 5. BEFORE DELETE 触发器
CREATE TRIGGER before_user_delete
BEFORE DELETE ON users
FOR EACH ROW
BEGIN
    -- 软删除而不是硬删除
    SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Hard delete not allowed, use soft delete';
END;

-- 6. 查看触发器
SHOW TRIGGERS;
SHOW TRIGGERS LIKE 'user%';
SHOW CREATE TRIGGER before_user_insert;

-- 7. 删除触发器
DROP TRIGGER IF EXISTS before_user_insert;
```

---

## MySQL 性能调优

### 服务器配置优化

```sql
-- 查看当前配置
SHOW VARIABLES;
SHOW VARIABLES LIKE 'max_connections%';
SHOW VARIABLES LIKE 'innodb_buffer_pool_size%';
SHOW VARIABLES LIKE 'query_cache%';

-- 关键配置参数（需在 my.cnf 中设置）
-- innodb_buffer_pool_size: 通常设为服务器内存的 70-80%
-- innodb_log_file_size: 通常设为 1GB 左右
-- max_connections: 根据应用需求设置
-- query_cache_size: MySQL 8.0 已移除此特性
-- innodb_flush_log_at_trx_commit: 0/1/2 控制事务持久性
-- sync_binlog: 控制 binlog 刷新策略
-- table_open_cache: 缓存打开的表数量
-- thread_cache_size: 缓存线程数量
```

### 查询分析

```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';
SET GLOBAL long_query_time = 2;  -- 超过 2 秒记录

-- 分析 EXPLAIN
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- 查看索引使用
SHOW INDEX FROM users;
SHOW INDEX FROM orders;

-- 强制使用索引
SELECT * FROM users USE INDEX (idx_email) WHERE email = 'test@example.com';

-- 忽略索引
SELECT * FROM users IGNORE INDEX (idx_status) WHERE status = 'active';
```

### 表优化

```sql
-- 分析表（更新统计信息）
ANALYZE TABLE users;

-- 检查表
CHECK TABLE users;

-- 修复表
REPAIR TABLE users;

-- 优化表（消除碎片）
OPTIMIZE TABLE users;

-- 重建表
ALTER TABLE users ENGINE = InnoDB;

-- 清理碎片（不锁表）
ALTER TABLE users FORCE;

-- 查看表状态
SHOW TABLE STATUS FROM mydb LIKE 'users';
SHOW TABLE STATUS FROM mydb LIKE 'orders';
```

### 连接与线程优化

```sql
-- 查看当前连接
SHOW PROCESSLIST;

-- 查看完整连接信息
SHOW FULL PROCESSLIST;

-- 查看连接状态
SHOW STATUS LIKE 'Threads%';
-- Threads_connected: 当前连接数
-- Threads_running: 正在运行的线程数
-- Threads_cached: 缓存的线程数
-- Threads_created: 创建的线程总数

-- 查看最大连接数
SHOW VARIABLES LIKE 'max_connections';

-- 查看连接使用情况
SHOW STATUS LIKE 'Connections';
SHOW STATUS LIKE 'Aborted_connects';
SHOW STATUS LIKE 'Max_used_connections';

-- 设置连接超时
SET GLOBAL wait_timeout = 600;
SET GLOBAL interactive_timeout = 600;
SET GLOBAL connect_timeout = 10;
```

### 缓冲池优化

```sql
-- 查看缓冲池状态
SHOW ENGINE INNODB STATUS;

-- 查看缓冲池使用
SHOW STATUS LIKE 'Innodb_buffer_pool%';

-- 设置缓冲池大小（需重启或动态调整 MySQL 8.0+）
SET GLOBAL innodb_buffer_pool_size = 8589934592;  -- 8GB

-- 查看缓冲池页信息
SELECT
    pool_id,
    hit_rate,
    pages_made_young,
    pages_not_made_young
FROM information_schema.innodb_buffer_pool_stats;

-- 预热缓冲池
LOAD INDEX INTO CACHE users;
LOAD INDEX INTO CACHE orders, products;
```

---

## MySQL 分区表

### 分区类型

```sql
-- 1. RANGE 分区
CREATE TABLE sales (
    id BIGINT AUTO_INCREMENT,
    sale_date DATE NOT NULL,
    amount DECIMAL(10,2),
    region VARCHAR(50)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- 2. LIST 分区
CREATE TABLE employees (
    id BIGINT AUTO_INCREMENT,
    name VARCHAR(100),
    department INT  -- 1: Engineering, 2: Sales, 3: HR
) PARTITION BY LIST (department) (
    PARTITION p_eng VALUES IN (1),
    PARTITION p_sales VALUES IN (2),
    PARTITION p_hr VALUES IN (3),
    PARTITION p_other VALUES IN (4, 5, 6)
);

-- 3. HASH 分区
CREATE TABLE user_sessions (
    id BIGINT AUTO_INCREMENT,
    user_id BIGINT,
    session_token VARCHAR(255),
    created_at TIMESTAMP
) PARTITION BY HASH (user_id)
PARTITIONS 8;

-- 4. KEY 分区
CREATE TABLE logs (
    id BIGINT AUTO_INCREMENT,
    message TEXT,
    level VARCHAR(20),
    created_at TIMESTAMP
) PARTITION BY KEY (id)
PARTITIONS 10;

-- 5. COLUMNS 分区（MySQL 5.7+）
CREATE TABLE events (
    id BIGINT AUTO_INCREMENT,
    event_date DATE,
    event_datetime DATETIME,
    event_value VARCHAR(50)
) PARTITION BY RANGE COLUMNS (event_date) (
    PARTITION p2023 VALUES LESS THAN ('2024-01-01'),
    PARTITION p2024 VALUES LESS THAN ('2025-01-01'),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- 6. 子分区
CREATE TABLE big_data (
    id BIGINT AUTO_INCREMENT,
    created_at DATE,
    data JSON
) PARTITION BY RANGE (YEAR(created_at))
SUBPARTITION BY HASH (MONTH(created_at))
SUBPARTITIONS 2 (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025)
);
```

### 分区管理

```sql
-- 查看分区信息
SELECT 
    PARTITION_NAME,
    TABLE_ROWS,
    AVG_ROW_LENGTH,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.PARTITIONS
WHERE TABLE_NAME = 'sales';

-- 添加分区
ALTER TABLE sales ADD PARTITION (
    PARTITION p2025 VALUES LESS THAN (2026)
);

-- 删除分区（数据也会被删除）
ALTER TABLE sales DROP PARTITION p2020;

-- 重新组织分区
ALTER TABLE sales REORGANIZE PARTITION pmax INTO (
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pmax VALUES LESS THAN MAXVALUE
);

-- 合并分区
ALTER TABLE sales REORGANIZE PARTITION p2023, p2024 INTO (
    PARTITION p2023_2024 VALUES LESS THAN (2025)
);

-- 交换分区（将分区数据移动到普通表）
CREATE TABLE sales_2020_archive LIKE sales;
ALTER TABLE sales EXCHANGE PARTITION p2020 WITH TABLE sales_2020_archive;

-- 重建分区（优化）
ALTER TABLE sales REBUILD PARTITION p2024;

-- 分析分区
ALTER TABLE sales ANALYZE PARTITION p2024;

-- 检查分区
ALTER TABLE sales CHECK PARTITION p2024;

-- 优化分区
ALTER TABLE sales OPTIMIZE PARTITION p2024;

-- 修复分区
ALTER TABLE sales REPAIR PARTITION p2024;

-- 清空分区
ALTER TABLE sales TRUNCATE PARTITION p2024;
```

---

## MySQL 高可用架构

### 主从复制配置

```sql
-- 1. 主库配置（my.cnf）
[mysqld]
server-id = 1
log-bin = mysql-bin
binlog_format = ROW
binlog_expire_logs_seconds = 604800
max_binlog_size = 100M
sync_binlog = 1
gtid_mode = ON
enforce_gtid_consistency = ON

-- 2. 主库创建复制用户
CREATE USER 'repl_user'@'%' IDENTIFIED WITH mysql_native_password BY 'replication_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';

-- 3. 从库配置（my.cnf）
[mysqld]
server-id = 2
relay-log = mysql-relay-bin
read_only = ON
super_read_only = ON
gtid_mode = ON
enforce_gtid_consistency = ON
log_slave_updates = ON

-- 4. 设置主库连接
CHANGE MASTER TO
    MASTER_HOST = 'master_host',
    MASTER_USER = 'repl_user',
    MASTER_PASSWORD = 'replication_password',
    MASTER_PORT = 3306,
    MASTER_LOG_FILE = 'mysql-bin.000001',
    MASTER_LOG_POS = 157,
    MASTER_AUTO_POSITION = 1;

-- 5. 启动/停止复制
START SLAVE;
STOP SLAVE;

-- 6. 查看复制状态
SHOW SLAVE STATUS\G
-- 关键字段:
-- Slave_IO_Running: Yes 表示 I/O 线程运行正常
-- Slave_SQL_Running: Yes 表示 SQL 线程运行正常
-- Seconds_Behind_Master: 复制延迟（秒）
-- Master_Log_File: 当前读取的主库 binlog
-- Relay_Master_Log_File: 当前执行的 binlog
```

### 半同步复制

```sql
-- 1. 安装半同步插件
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- 2. 主库启用半同步
SET GLOBAL rpl_semi_sync_master_enabled = ON;
SET GLOBAL rpl_semi_sync_master_timeout = 10000;  -- 10 秒超时

-- 3. 从库启用半同步
SET GLOBAL rpl_semi_sync_slave_enabled = ON;

-- 4. 重启 I/O 线程
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;

-- 5. 查看半同步状态
SHOW STATUS LIKE 'Rpl_semi_sync_master%';
SHOW STATUS LIKE 'Rpl_semi_sync_slave%';
```

### 组复制（MySQL Group Replication）

```sql
-- 1. 配置（所有节点 my.cnf）
[mysqld]
server-id = 1  -- 每个节点不同
transaction_write_set_extraction = XXHASH64
group_replication_group_name = "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
group_replication_start_on_boot = OFF
group_replication_local_address = "node1:33061"
group_replication_group_seeds = "node1:33061,node2:33061,node3:33061"
group_replication_bootstrap_group = OFF

-- 2. 安装组复制插件
INSTALL PLUGIN group_replication SONAME 'group_replication.so';

-- 3. 创建复制用户
SET GLOBAL group_replication_recovery_user = 'recovery_user';
SET GLOBAL group_replication_recovery_password = 'recovery_password';

-- 4. 引导第一个节点
SET GLOBAL group_replication_bootstrap_group = ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group = OFF;

-- 5. 其他节点加入
START GROUP_REPLICATION;

-- 6. 查看组复制状态
SELECT * FROM performance_schema.replication_group_members;
```

### 故障转移配置

```sql
-- MySQL Router 配置示例（读取负载均衡）
[DEFAULT]
logging_folder = /var/log/mysql-router
plugin_folder = /usr/lib/mysql-router/plugin
runtime_folder = /var/run/mysql-router

[routing:default]
bind_address = 0.0.0.0
bind_port = 6446
destinations = node1:3306,node2:3306,node3:3306
routing_strategy = round-robin
protocol = classic

[routing:read_only]
bind_address = 0.0.0.0
bind_port = 6447
destinations = node2:3306,node3:3306
routing_strategy = first-available
protocol = classic

[routing:read_write]
bind_address = 0.0.0.0
bind_port: 6448
destinations = node1:3306
routing_strategy = first-available
protocol = classic
```

---

## MySQL 8.0 新特性详解

### CTEs（公共表表达式）

```sql
-- 1. 普通 CTE
WITH active_users AS (
    SELECT id, name, email
    FROM users
    WHERE status = 'active'
)
SELECT * FROM active_users WHERE name LIKE 'A%';

-- 2. 递归 CTE
WITH RECURSIVE category_tree AS (
    -- 基础查询（根节点）
    SELECT id, name, parent_id, 0 AS level
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询（子节点）
    SELECT c.id, c.name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY level, name;

-- 3. 多个 CTE
WITH 
    user_orders AS (
        SELECT user_id, COUNT(*) as order_count, SUM(total) as total_spent
        FROM orders
        GROUP BY user_id
    ),
    high_value_users AS (
        SELECT user_id
        FROM user_orders
        WHERE total_spent > 10000
    )
SELECT u.*, uo.order_count, uo.total_spent
FROM users u
JOIN user_orders uo ON u.id = uo.user_id
WHERE u.id IN (SELECT user_id FROM high_value_users);

-- 4. 递归 CTE 生成序列
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;
```

### 窗口函数

```sql
-- 1. ROW_NUMBER, RANK, DENSE_RANK
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_num,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank
FROM employees;

-- 2. LAG 和 LEAD
SELECT 
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) as prev_amount,
    LEAD(amount, 1) OVER (ORDER BY sale_date) as next_amount,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) as growth
FROM sales;

-- 3. FIRST_VALUE 和 LAST_VALUE
SELECT 
    sale_date,
    amount,
    FIRST_VALUE(amount) OVER (ORDER BY sale_date) as first_amount,
    LAST_VALUE(amount) OVER (ORDER BY sale_date) as last_amount
FROM sales;

-- 4. SUM, AVG, COUNT 窗口函数
SELECT 
    sale_date,
    amount,
    SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as rolling_7d_sum,
    AVG(amount) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as rolling_7d_avg,
    COUNT(*) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as rolling_7d_count
FROM sales;

-- 5. NTILE（分桶）
SELECT 
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary) as quartile
FROM employees;

-- 6. CUME_DIST（累计分布）
SELECT 
    name,
    salary,
    CUME_DIST() OVER (ORDER BY salary) as cumulative_dist
FROM employees;
```

### JSON 函数增强

```sql
-- 1. JSON_TABLE
SELECT jt.*
FROM orders,
JSON_TABLE(
    items,
    '$[*]' COLUMNS (
        product_id INT PATH '$.product_id',
        quantity INT PATH '$.quantity',
        price DECIMAL(10,2) PATH '$.price'
    )
) AS jt;

-- 2. JSON_ARRAYAGG 和 JSON_OBJECTAGG
SELECT 
    user_id,
    JSON_ARRAYAGG(product_id) as products
FROM orders
GROUP BY user_id;

SELECT 
    department,
    JSON_OBJECTAGG(employee_id, name) as employees
FROM employees
GROUP BY department;

-- 3. JSON_VALUE（提取标量值）
SELECT 
    JSON_VALUE(data, '$.name') as name,
    JSON_VALUE(data, '$.age' RETURNING UNSIGNED) as age
FROM users;

-- 4. JSON_QUERY 和 JSON_EXISTS
SELECT 
    JSON_QUERY(data, '$.address') as address,
    JSON_EXISTS(data, '$.phone.mobile') as has_mobile
FROM users;

-- 5. JSON_MERGE_PATCH（替代 JSON_MERGE_PRESERVE）
SELECT JSON_MERGE_PATCH(
    '{"a": 1, "b": 2}',
    '{"b": 3, "c": 4}'
);  -- 结果: {"a": 1, "b": 3, "c": 4}

-- 6. JSON_TABLE 与联结
SELECT 
    o.id,
    jt.product_name,
    jt.quantity
FROM orders o,
JSON_TABLE(
    items,
    '$[*]' COLUMNS (
        product_name VARCHAR(100) PATH '$.product_name',
        quantity INT PATH '$.quantity'
    )
) AS jt;
```

---

> [!SUCCESS]
> 本文档全面介绍了 MySQL 的核心特性、8.0+ 新功能、性能优化、存储过程、触发器以及高可用架构方案。MySQL 凭借其成熟稳定、性能优异和生态完善的特点，是 Web 应用和互联网场景的首选关系型数据库。
