# PostgreSQL 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 PostgreSQL 17 新特性、JSON 支持、向量搜索（pgvector）、PL/pgSQL 存储过程，以及在 AI 应用中的实战场景。

---

## 目录

1. [[#PostgreSQL 概述与核心定位]]
2. [[#PostgreSQL 17 新特性]]
3. [[#JSON 数据类型详解]]
4. [[#全文搜索]]
5. [[#高级 SQL 特性]]
6. [[#PostgreSQL vs MySQL vs MariaDB 对比]]
7. [[#pgvector 向量搜索]]
8. [[#Prisma/Drizzle 中的 PostgreSQL]]
9. [[#实战场景与代码示例]]
10. [[#选型建议]]

---

## PostgreSQL 概述与核心定位

### 为什么选择 PostgreSQL

PostgreSQL 是一个功能完备、特性丰富的开源关系型数据库，由加州大学伯克利分校于 1986 年发起，历经近 40 年的发展，已成为「功能最全面的开源数据库」。

PostgreSQL 的核心优势体现在四个维度：

**功能完备**：PostgreSQL 提供了业界最丰富的 SQL 特性，包括窗口函数、CTE、递归查询、触发器、存储过程、JSON 支持等。

**扩展性强**：支持超过 100 种扩展插件，包括向量搜索（pgvector）、地理信息（PostGIS）、时序数据（TimescaleDB）等。

**标准兼容**：严格遵循 SQL 标准，兼容性极佳。迁移成本低，不会被锁定在特定厂商。

**性能卓越**：在 OLTP、OLAP、混合负载场景下均有出色表现。MVCC 机制提供了优秀的并发控制。

### PostgreSQL 在 AI 时代的价值

在 AI 应用中，PostgreSQL 的价值被重新发现：

- **向量搜索**：pgvector 扩展让 PostgreSQL 支持 Embedding 相似度搜索
- **混合存储**：结构化数据与向量数据共存于单一数据库
- **RLS 策略**：行级安全策略实现细粒度访问控制
- **JSON 支持**：灵活处理 AI 输出的半结构化数据

### 市场份额与行业地位

PostgreSQL 在全球数据库市场中的地位持续攀升：

| 排名 | 数据库 | 市场份额（2025） | 趋势 |
|------|--------|------------------|------|
| 1 | PostgreSQL | 28.5% | ↑ |
| 2 | MySQL | 25.2% | → |
| 3 | MongoDB | 12.8% | ↓ |
| 4 | Microsoft SQL Server | 10.5% | → |
| 5 | Redis | 8.3% | ↑ |

**关键数据点**：
- PostgreSQL 连续多年被评为「开发者最喜爱的数据库」
- 超过 50% 的新项目选择 PostgreSQL 作为主数据库
- 主要使用者包括苹果、Spotify、Netflix、Uber、Airbnb 等科技巨头
- 在 DB-Engines 排名中稳居前五

**PostgreSQL 适用场景**：

| 场景 | 推荐程度 | 说明 |
|------|----------|------|
| 企业级应用 | ⭐⭐⭐⭐⭐ | 功能完备，稳定可靠 |
| AI/ML 应用 | ⭐⭐⭐⭐⭐ | pgvector 向量搜索 |
| 数据仓库 | ⭐⭐⭐⭐⭐ | 强大的分析能力 |
| Web 应用 | ⭐⭐⭐⭐ | 性能优秀 |
| 移动后端 | ⭐⭐⭐⭐ | 可靠性和扩展性 |
| GIS 应用 | ⭐⭐⭐⭐⭐ | PostGIS 扩展 |

---

## 核心概念与数据模型

### PostgreSQL 架构

PostgreSQL 采用客户端-服务器架构，主要组件包括：

```
┌─────────────────────────────────────────────────────────────┐
│                        PostgreSQL 架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   客户端应用层                                                │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│   │  psql   │  │  libpq   │  │ JDBC/ODBC│                  │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
│        │             │             │                         │
│        └─────────────┼─────────────┘                         │
│                      ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Postmaster 守护进程                       │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐            │   │
│   │  │ 连接管理 │  │ 查询分析 │  │ 存储管理 │            │   │
│   │  └─────────┘  └─────────┘  └─────────┘            │   │
│   └─────────────────────────────────────────────────────┘   │
│                      │                                        │
│        ┌────────────┼────────────┐                         │
│        ▼            ▼            ▼                         │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                  │
│   │ Buffer  │  │  WAL    │  │ 锁管理  │                  │
│   │  Pool   │  │  Log    │  │         │                  │
│   └─────────┘  └─────────┘  └─────────┘                  │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    数据文件                             │   │
│   │  (表、索引、TOAST、MVCC 多版本)                       │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 核心概念

#### 1. MVCC（多版本并发控制）

PostgreSQL 使用 MVCC 实现并发控制，每个事务看到的是数据库的一致性快照：

```sql
-- 查看事务隔离级别
SHOW transaction_isolation;

-- 查看当前事务 ID
SELECT txid_current();

-- 查看当前快照
SELECT * FROM pg_stat_activity WHERE pid = pg_backend_pid();

-- 事务示例：两个并发事务
-- 事务 A（可重复读）
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- 看到余额 1000
-- 此时事务 B 修改了余额为 2000
SELECT balance FROM accounts WHERE id = 1;  -- 仍看到 1000
COMMIT;  -- 提交时检查一致性

-- 事务 B（读已提交）
BEGIN;
UPDATE accounts SET balance = 2000 WHERE id = 1;
COMMIT;  -- 提交后其他事务可见
```

#### 2. TOAST（超长字段存储）

PostgreSQL 自动将大字段存储到独立的 TOAST 表中：

```sql
-- 创建使用 TOAST 的表
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255),
    content TEXT,      -- 大字段，自动使用 TOAST
    metadata JSONB     -- JSON 也可能使用 TOAST
);

-- 查看 TOAST 表
SELECT relname, reltoastrelid
FROM pg_class
WHERE relname = 'articles';

-- TOAST 压缩策略
-- 可在创建表时指定
CREATE TABLE large_data (
    data BYTEA
) WITH (toast_tuple_target = 248);
```

#### 3. VACUUM 机制

VACUUM 是 PostgreSQL 维护的重要机制：

```sql
-- 标准 VACUUM（不阻塞读写）
VACUUM articles;

-- VACUUM FULL（会阻塞，需要重建表）
VACUUM FULL articles;

-- 带分析的 VACUUM
VACUUM ANALYZE articles;

-- 查看 VACUUM 状态
SELECT * FROM pg_stat_progress_vacuum;

-- 查看死亡元组
SELECT schemaname, relname, n_dead_tup, n_live_tup
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;

-- 自动 VACUUM 配置
ALTER TABLE articles SET (
    autovacuum_vacuum_threshold = 50,
    autovacuum_analyze_threshold = 50,
    autovacuum_vacuum_scale_factor = 0.1,
    autovacuum_analyze_scale_factor = 0.05
);
```

### 数据模型设计

#### 1. 规范化设计

```sql
-- 高度规范化（第三范式）
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    order_date DATE NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    status VARCHAR(20) DEFAULT 'pending'
);

CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL,
    price DECIMAL(10,2) NOT NULL
);
```

#### 2. 反规范化设计（读多写少场景）

```sql
-- 添加冗余字段优化查询
CREATE TABLE orders_denormalized (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    customer_name VARCHAR(100),  -- 冗余字段
    customer_email VARCHAR(255), -- 冗余字段
    order_date DATE NOT NULL,
    total DECIMAL(10,2) NOT NULL,
    item_count INTEGER,          -- 冗余字段
    items_summary JSONB          -- 冗余字段
);

-- 使用物化视图
CREATE MATERIALIZED VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total) AS total_sales,
    AVG(total) AS avg_order_value
FROM orders
GROUP BY DATE_TRUNC('month', order_date)
WITH DATA;

CREATE UNIQUE INDEX idx_monthly_sales ON monthly_sales(month);

-- 刷新物化视图
REFRESH MATERIALIZED VIEW monthly_sales;
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales;  -- 不阻塞查询
```

---

---

## PostgreSQL 17 新特性

PostgreSQL 17 于 2024 年 9 月发布，是自 PostgreSQL 16 以来的重大更新。

### 核心新特性

#### 1. 改进的 VACUUM 机制

```sql
-- VACUUM 现在可以并行执行
VACUUM (PARALLEL 4) large_table;

-- 新的 VACUUM 选项
VACUUM (SKIP_LOCKED, INDEX_CLEANUP AUTO) large_table;

-- 监控 VACUUM 进度
SELECT * FROM pg_stat_progress_vacuum;
```

#### 2. JSON_TABLE 函数

```sql
-- 将 JSON 数据转换为关系表
SELECT *
FROM json_table(
    '{"users": [
        {"name": "Alice", "age": 30},
        {"name": "Bob", "age": 25}
    ]}'::json,
    '$.users[*]'
    COLUMNS (
        name TEXT PATH '$.name',
        age INT PATH '$.age'
    )
) AS users;

-- 结果:
--  name  | age
-- -------+-----
--  Alice |  30
--  Bob   |  25
```

#### 3. 增强的 MERGE 语句

```sql
-- 更强大的 MERGE 支持
MERGE INTO target_table AS t
USING source_view AS s
ON t.id = s.id
WHEN MATCHED AND s.type = 'UPDATE' THEN
    UPDATE SET
        name = s.name,
        updated_at = NOW()
WHEN MATCHED AND s.type = 'DELETE' THEN
    DELETE
WHEN NOT MATCHED THEN
    INSERT (id, name, created_at)
    VALUES (s.id, s.name, NOW());
```

#### 4. SQL/JSON 增强

```sql
-- JSON_EXISTS 函数（检查路径是否存在）
SELECT JSON_EXISTS(
    '{"user": {"name": "Alice"}}'::json,
    '$.user.name'
);  -- 返回 true

-- JSON_QUERY 函数（提取 JSON 值）
SELECT JSON_QUERY(
    '{"items": [1, 2, 3]}'::json,
    '$.items'
);  -- 返回 '[1, 2, 3]'
```

#### 5. 改进的 EXPLAIN 输出

```sql
-- 更详细的执行计划
EXPLAIN (ANALYZE, VERBOSE, BUFFERS, SETTINGS)
SELECT * FROM users WHERE age > 25;
```

### 其他改进

| 特性 | 说明 |
|------|------|
| **增量备份改进** | 更高效的增量备份机制 |
| **逻辑复制增强** | 支持更多数据类型 |
| **正则表达式** | 新增 Unicode 属性支持 |
| **窗口函数** | 支持更多窗口帧选项 |

---

## JSON 数据类型详解

PostgreSQL 提供了两种 JSON 数据类型：`json` 和 `jsonb`，它们在存储和查询方式上有显著差异。

### json vs jsonb 对比

| 特性 | json | jsonb |
|------|------|-------|
| **存储格式** | 存储原始 JSON 字符串 | 二进制格式存储 |
| **查询性能** | 每次查询需重新解析 | 直接索引，性能更高 |
| **索引支持** | 有限 | 支持 GIN、GiST 索引 |
| **空间占用** | 较小（无转换开销） | 较大（预处理） |
| **类型保留** | 保留原始格式 | 保留数据类型 |
| **使用场景** | 临时数据、日志 | 高频查询的核心数据 |

### JSON 查询操作符

```sql
-- 创建表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    attributes JSONB
);

-- 插入数据
INSERT INTO products (name, attributes) VALUES
    ('iPhone 15', '{"color": "black", "storage": "256GB", "price": 999}'),
    ('MacBook Pro', '{"color": "silver", "storage": "512GB", "price": 1999}'),
    ('AirPods', '{"color": "white", "price": 199}');

-- 查询：提取 JSON 字段
SELECT 
    name,
    attributes->>'color' AS color,
    attributes->>'price' AS price
FROM products;

-- 查询：使用 WHERE 子句过滤
SELECT * FROM products
WHERE attributes->>'color' = 'black';

-- 查询：数字比较
SELECT * FROM products
WHERE (attributes->>'price')::numeric < 1000;

-- 查询：使用 jsonb 操作符（更高效）
SELECT * FROM products
WHERE attributes @> '{"color": "black"}';  -- 包含指定对象

SELECT * FROM products
WHERE attributes ? 'color';  -- 包含指定键

SELECT * FROM products
WHERE attributes ?| array['color', 'storage'];  -- 包含任一键
```

### JSON 函数进阶

```sql
-- jsonb_object_keys：获取所有键
SELECT jsonb_object_keys(attributes)
FROM products;

-- jsonb_each：展开为键值对
SELECT key, value
FROM products,
    jsonb_each(attributes);

-- jsonb_agg：聚合为 JSON
SELECT 
    name,
    jsonb_object_agg(key, value) AS attributes
FROM products,
    jsonb_each(attributes::jsonb)
GROUP BY name;

-- jsonb_build_object：构建 JSON 对象
SELECT jsonb_build_object(
    'name', name,
    'price', (attributes->>'price')::numeric,
    'color', attributes->>'color'
) AS product_info
FROM products;

-- 递归查询 JSON
WITH RECURSIVE json_tree AS (
    -- 初始查询：顶层对象
    SELECT 
        id,
        attributes,
        0 AS depth,
        jsonb_array_elements(
            CASE 
                WHEN jsonb_typeof(attributes->'items') = 'array' 
                THEN attributes->'items' 
                ELSE '[]'::jsonb 
            END
        ) AS item
    FROM products
    
    UNION ALL
    
    -- 递归：嵌套数组
    SELECT 
        p.id,
        p.attributes,
        jt.depth + 1,
        jsonb_array_elements(
            CASE 
                WHEN jsonb_typeof(jt.item->'children') = 'array' 
                THEN jt.item->'children' 
                ELSE '[]'::jsonb 
            END
        )
    FROM products p
    JOIN json_tree jt ON p.id = jt.id
    WHERE jt.depth < 10  -- 防止无限递归
)
SELECT * FROM json_tree;
```

### JSON 索引

```sql
-- 创建 GIN 索引（适用于 JSON 包含查询）
CREATE INDEX idx_products_attributes 
ON products USING GIN (attributes);

-- 创建表达式索引（适用于特定路径查询）
CREATE INDEX idx_products_color 
ON products ((attributes->>'color'));

-- 创建部分索引
CREATE INDEX idx_products_black 
ON products ((attributes->>'price'))
WHERE attributes @> '{"color": "black"}';

-- 验证索引使用
EXPLAIN SELECT * FROM products
WHERE attributes @> '{"color": "black"}';
```

---

## 全文搜索

PostgreSQL 内置了强大的全文搜索功能，无需额外安装插件。

### 全文搜索基础

```sql
-- 创建全文搜索配置（中文建议使用 zhparser 或 pg_jieba）
-- 这里使用默认的 english 配置

-- 索引列
ALTER TABLE articles ADD COLUMN 
    search_vector TSVECTOR
    GENERATED ALWAYS AS (
        to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''))
    ) STORED;

-- 创建索引
CREATE INDEX idx_articles_search 
ON articles USING GIN (search_vector);

-- 搜索查询
SELECT 
    title,
    ts_rank(search_vector, query) AS rank
FROM articles,
    to_tsquery('english', 'AI & (Python | JavaScript)') query
WHERE search_vector @@ query
ORDER BY rank DESC;

-- 高级搜索：包含短语
SELECT * FROM articles
WHERE search_vector @@ to_tsquery('english', 'PostgreSQL <-> 17');

-- 搜索结果高亮
SELECT 
    title,
    ts_headline(
        'english',
        content,
        to_tsquery('english', 'AI'),
        'StartSel=<mark>, StopSel=</mark>'
    ) AS snippet
FROM articles
WHERE search_vector @@ to_tsquery('english', 'AI');
```

### 搜索权重与排名

```sql
-- 为不同字段设置权重
ALTER TABLE articles ADD COLUMN 
    search_vector_weighted TSVECTOR
    GENERATED ALWAYS AS (
        setweight(to_tsvector('english', coalesce(title, '')), 'A') ||
        setweight(to_tsvector('english', coalesce(summary, '')), 'B') ||
        setweight(to_tsvector('english', coalesce(content, '')), 'C')
    ) STORED;

-- 使用加权搜索
SELECT 
    title,
    ts_rank(search_vector_weighted, query) AS rank
FROM articles,
    plainto_tsquery('english', 'postgresql features') query
WHERE search_vector_weighted @@ query
ORDER BY rank DESC;
```

---

## 高级 SQL 特性

### 窗口函数

窗口函数允许在不影响原始数据的情况下进行跨行计算：

```sql
-- 创建示例数据
CREATE TABLE sales (
    id SERIAL,
    region TEXT,
    salesperson TEXT,
    sale_date DATE,
    amount NUMERIC
);

-- 窗口函数示例
SELECT 
    salesperson,
    region,
    amount,
    SUM(amount) OVER (
        PARTITION BY region  -- 按区域分组
        ORDER BY sale_date   -- 按日期排序
    ) AS running_total,
    
    AVG(amount) OVER (
        PARTITION BY region
    ) AS region_avg,
    
    amount - AVG(amount) OVER (
        PARTITION BY region
    ) AS diff_from_avg,
    
    RANK() OVER (
        PARTITION BY region
        ORDER BY amount DESC
    ) AS rank_in_region,
    
    LAG(amount, 1) OVER (
        PARTITION BY salesperson
        ORDER BY sale_date
    ) AS prev_sale,
    
    LEAD(amount, 1) OVER (
        PARTITION BY salesperson
        ORDER BY sale_date
    ) AS next_sale
FROM sales;
```

### CTE（公共表表达式）

```sql
-- 普通 CTE
WITH monthly_sales AS (
    SELECT 
        DATE_TRUNC('month', sale_date) AS month,
        SUM(amount) AS total
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date)
),
avg_sales AS (
    SELECT AVG(total) AS average FROM monthly_sales
)
SELECT 
    ms.month,
    ms.total,
    a.average,
    ms.total - a.average AS diff
FROM monthly_sales ms
CROSS JOIN avg_sales a
ORDER BY ms.month;

-- 递归 CTE：生成树形结构
WITH RECURSIVE category_tree AS (
    -- 基础查询：根节点
    SELECT 
        id,
        name,
        parent_id,
        1 AS depth,
        name::text AS path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询：子节点
    SELECT 
        c.id,
        c.name,
        c.parent_id,
        ct.depth + 1,
        ct.path || ' > ' || c.name
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
    WHERE ct.depth < 10  -- 防止无限递归
)
SELECT * FROM category_tree ORDER BY path;
```

### 高级聚合

```sql
-- FILTER 子句
SELECT 
    region,
    SUM(amount) AS total_sales,
    SUM(amount) FILTER (WHERE sale_date >= '2024-01-01') AS ytd_sales
FROM sales
GROUP BY region;

-- GROUPING SETS：多维度聚合
SELECT 
    region,
    salesperson,
    SUM(amount) AS total
FROM sales
GROUP BY GROUPING SETS (
    (region, salesperson),  -- 按区域和销售人员
    (region,),              -- 按区域
    (salesperson,),        -- 按销售人员
    ()                     -- 总计
);

-- CUBE：所有组合
SELECT 
    region,
    salesperson,
    SUM(amount) AS total
FROM sales
GROUP BY CUBE (region, salesperson);

-- ROLLUP：层次结构
SELECT 
    region,
    salesperson,
    SUM(amount) AS total
FROM sales
GROUP BY ROLLUP (region, salesperson);
```

---

## PostgreSQL vs MySQL vs MariaDB 对比

### 核心对比表

| 特性 | PostgreSQL | MySQL | MariaDB |
|------|------------|-------|---------|
| **最新稳定版** | 17.x | 9.0.x | 11.6.x |
| **许可证** | PostgreSQL License (BSD) | GPLv2 / 商业版 | GPLv2 |
| **ACID 完整性** | 完整支持 | 完整支持（InnoDB） | 完整支持 |
| **MVCC** | 原生支持 | 支持（InnoDB） | 支持 |
| **JSON 支持** | 原生（jsonb） | 5.7+ 支持 | 10.2+ 支持 |
| **向量搜索** | pgvector 扩展 | 插件支持 | 插件支持 |
| **地理信息** | PostGIS | 插件 | 插件 |
| **全文搜索** | 内置 | 支持 | 支持 |
| **存储过程** | PL/pgSQL, PL/Perl, PL/Python | SQL/PSM | SQL/PSM |
| **窗口函数** | 完全支持 | 支持 | 支持 |
| **CTE/递归** | 完全支持 | 8.0+ 支持 | 10.2+ 支持 |
| **分区** | 声明式分区 | 支持 | 支持 |
| **复制** | 物理复制、逻辑复制 | 异步、同步、半同步 | Galera Cluster |
| **连接池** | 内置 PgBouncer/PGpool | ProxySQL/HikariCP | ProxySQL |
| **性能** | OLAP 强 | OLTP 强 | OLTP 强 |

### 选择建议

| 场景 | 推荐选择 | 理由 |
|------|----------|------|
| **AI/ML 应用** | PostgreSQL + pgvector | 向量搜索 + 关系数据 |
| **OLAP 数据仓库** | PostgreSQL | 窗口函数、CTE 强大 |
| **高并发 OLTP** | MySQL / MariaDB | 简单、快速 |
| **复杂查询** | PostgreSQL | 标准兼容、功能完备 |
| **Web 应用** | MySQL / MariaDB | 生态成熟、配置简单 |
| **地理信息系统** | PostgreSQL + PostGIS | 专业 GIS 支持 |
| **全文搜索** | PostgreSQL | 内置、功能强大 |

---

## pgvector 向量搜索

pgvector 是 PostgreSQL 的向量嵌入扩展，在 AI 时代具有重要价值。

### 安装与基础配置

```sql
-- 安装扩展（需要超级用户权限）
CREATE EXTENSION IF NOT EXISTS vector;

-- 查看已安装扩展
SELECT * FROM pg_extension WHERE extname = 'vector';
```

### 向量列与索引

```sql
-- 创建带向量列的表
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    content TEXT,
    embedding VECTOR(1536),  -- OpenAI text-embedding-3-small 维度
    metadata JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- 创建 HNSW 索引（推荐，性能最佳）
CREATE INDEX idx_documents_embedding_hnsw 
ON documents 
USING hnsw (embedding vector_cosine_ops);

-- 也可以创建 IVFFlat 索引（适合大数据集）
CREATE INDEX idx_documents_embedding_ivf 
ON documents 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- 查看索引信息
\d documents
```

### 向量搜索操作

```sql
-- 插入向量数据
INSERT INTO documents (content, embedding, metadata) VALUES
    (
        'PostgreSQL is a powerful open-source relational database.',
        '[0.1, 0.2, 0.3, ...]::vector(1536)',
        '{"source": "wiki", "category": "database"}'
    );

-- 余弦相似度搜索
SELECT 
    id,
    content,
    1 - (embedding <=> '[query_vector]::vector(1536)') AS similarity
FROM documents
ORDER BY embedding <=> '[query_vector]::vector(1536)'
LIMIT 5;

-- L2 距离搜索
SELECT 
    id,
    content,
    (embedding <-> '[query_vector]::vector(1536)') AS distance
FROM documents
ORDER BY embedding <-> '[query_vector]::vector(1536)'
LIMIT 5;

-- 内积搜索
SELECT 
    id,
    content,
    (embedding <#> '[query_vector]::vector(1536)') AS dot_product
FROM documents
ORDER BY embedding <#> '[query_vector]::vector(1536)'
LIMIT 5;
```

### 混合搜索：向量 + 关键词

```sql
-- 创建全文搜索列
ALTER TABLE documents ADD COLUMN 
    search_vector TSVECTOR
    GENERATED ALWAYS AS (
        to_tsvector('english', coalesce(content, ''))
    ) STORED;

CREATE INDEX idx_documents_text_search 
ON documents USING GIN (search_vector);

-- 混合搜索：向量相似度 + 关键词匹配
WITH vector_results AS (
    SELECT 
        id,
        content,
        1 - (embedding <=> '[query_vector]::vector(1536)') AS vector_similarity,
        ts_rank(search_vector, query) AS text_rank
    FROM documents,
        plainto_tsquery('english', 'database') query
    WHERE search_vector @@ query
)
SELECT 
    id,
    content,
    0.7 * vector_similarity + 0.3 * text_rank AS combined_score
FROM vector_results
ORDER BY combined_score DESC
LIMIT 10;

-- 过滤搜索：结合元数据
SELECT 
    id,
    content,
    1 - (embedding <=> '[query_vector]::vector(1536)') AS similarity
FROM documents
WHERE metadata @> '{"category": "database"}'
ORDER BY embedding <=> '[query_vector]::vector(1536)'
LIMIT 5;
```

### Python 集成示例

```python
from psycopg2 import connect
import numpy as np

# 连接数据库
conn = connect(
    host="localhost",
    database="ai_app",
    user="postgres",
    password="password"
)
cursor = conn.cursor()

def generate_embedding(text: str) -> list[float]:
    """生成文本嵌入向量（示例）"""
    # 实际使用 OpenAI API 或本地模型
    return np.random.rand(1536).tolist()

def search_similar_documents(query: str, top_k: int = 5) -> list[dict]:
    """搜索相似文档"""
    embedding = generate_embedding(query)
    embedding_str = f"[{','.join(map(str, embedding))}]"
    
    sql = """
        SELECT 
            id,
            content,
            metadata,
            1 - (embedding <=> %s::vector) AS similarity
        FROM documents
        ORDER BY embedding <=> %s::vector
        LIMIT %s
    """
    
    cursor.execute(sql, (embedding_str, embedding_str, top_k))
    results = cursor.fetchall()
    
    return [
        {
            "id": row[0],
            "content": row[1],
            "metadata": row[2],
            "similarity": row[3]
        }
        for row in results
    ]

# 使用示例
results = search_similar_documents("What is PostgreSQL?", top_k=5)
for r in results:
    print(f"[{r['similarity']:.3f}] {r['content'][:100]}...")
```

---

## Prisma/Drizzle 中的 PostgreSQL

### Prisma Schema 定义

```prisma
// schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // 关系
  posts     Post[]
  
  @@index([email])
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // 向量嵌入（存储为原始向量）
  embedding String?  // 存储为 JSON 字符串格式
  
  // 关系
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  
  @@index([authorId])
  @@index([published])
}

// 扩展 Prisma Client 功能
model Document {
  id        String   @id @default(uuid())
  content   String
  metadata  Json?    // JSON 类型
  embedding Unsupported("vector(1536)")?
  
  createdAt DateTime @default(now())
  
  @@index([embedding], type: Hnsw(ops: VectorCosineOps))
}
```

### Drizzle Schema 定义

```typescript
// schema.ts
import { pgTable, serial, text, boolean, timestamp, uuid, jsonb, vector } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull().unique(),
  name: text('name'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

export const posts = pgTable('posts', {
  id: uuid('id').defaultRandom().primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  published: boolean('published').default(false).notNull(),
  authorId: uuid('author_id').notNull().references(() => users.id),
  embedding: vector('embedding', { dimensions: 1536 }), // pgvector
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});

// 向量搜索查询
export async function searchSimilarPosts(client: any, queryEmbedding: number[], limit = 5) {
  const results = await client.execute(sql`
    SELECT 
      id,
      title,
      content,
      1 - (embedding <=> ${queryEmbedding}::vector) AS similarity
    FROM posts
    WHERE published = true
    ORDER BY embedding <=> ${queryEmbedding}::vector
    LIMIT ${limit}
  `);
  return results.rows;
}
```

---

## 实战场景与代码示例

### 场景一：构建 RAG 系统

```sql
-- RAG 系统数据模型
CREATE TABLE knowledge_base (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    content TEXT NOT NULL,
    source_url TEXT,
    embedding VECTOR(1536),
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- 创建 HNSW 索引
CREATE INDEX idx_knowledge_embedding 
ON knowledge_base USING hnsw (embedding vector_cosine_ops);

-- 创建全文搜索索引
ALTER TABLE knowledge_base ADD COLUMN 
    search_vector TSVECTOR
    GENERATED ALWAYS AS (
        to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''))
    ) STORED;

CREATE INDEX idx_knowledge_text 
ON knowledge_base USING GIN (search_vector);

-- 混合搜索函数
CREATE OR REPLACE FUNCTION hybrid_search(
    query_embedding VECTOR(1536),
    query_text TEXT,
    top_k INT DEFAULT 5,
    alpha FLOAT DEFAULT 0.7
)
RETURNS TABLE (
    id UUID,
    title TEXT,
    content TEXT,
    score FLOAT
) AS $$
BEGIN
    RETURN QUERY
    WITH vector_search AS (
        SELECT 
            id,
            title,
            content,
            1 - (embedding <=> query_embedding) AS vector_score
        FROM knowledge_base
        ORDER BY embedding <=> query_embedding
        LIMIT top_k * 2
    ),
    text_search AS (
        SELECT 
            id,
            ts_rank(search_vector, plainto_tsquery('english', query_text)) AS text_score
        FROM knowledge_base
        WHERE search_vector @@ plainto_tsquery('english', query_text)
    )
    SELECT 
        vs.id,
        vs.title,
        vs.content,
        COALESCE(alpha * vs.vector_score + (1 - alpha) * ts.text_score, vs.vector_score) AS score
    FROM vector_search vs
    LEFT JOIN text_search ts ON vs.id = ts.id
    ORDER BY score DESC
    LIMIT top_k;
END;
$$ LANGUAGE plpgsql;
```

### 场景二：实时数据订阅

```sql
-- 启用逻辑复制
-- 需要在 postgresql.conf 中设置：wal_level = logical

-- 创建发布
CREATE PUBLICATION chat_messages FOR TABLE messages;

-- 创建订阅（需要 PostgreSQL 14+）
CREATE SUBSCRIPTION chat_realtime
    CONNECTION 'host=localhost port=5432 dbname=ai_chat user=postgres'
    PUBLICATION chat_messages
    WITH (copy_data = true);

-- 查看订阅状态
SELECT * FROM pg_stat_subscription;

-- 查看复制延迟
SELECT 
    subname,
    received_lsn,
    latest_lsn,
    pg_wal_lsn_diff(received_lsn, latest_lsn) AS replication_lag_bytes
FROM pg_stat_subscription;
```

---

## 选型建议

### 何时选择 PostgreSQL

| 场景 | 推荐程度 | 原因 |
|------|----------|------|
| **AI 应用（向量搜索）** | ⭐⭐⭐⭐⭐ | pgvector + 关系数据 |
| **复杂查询/报表** | ⭐⭐⭐⭐⭐ | 窗口函数、CTE 强大 |
| **需要 JSON 灵活性** | ⭐⭐⭐⭐⭐ | jsonb 高性能支持 |
| **地理信息应用** | ⭐⭐⭐⭐⭐ | PostGIS 扩展 |
| **高并发 OLTP** | ⭐⭐⭐⭐ | 成熟稳定 |
| **简单 CRUD 应用** | ⭐⭐⭐ | 可能过于复杂 |
| **需要 MySQL 兼容** | ⭐⭐ | 需要额外配置 |

### 配置优化建议

```sql
-- 关键配置参数（在 postgresql.conf 中设置）

-- 内存配置
shared_buffers = '256MB'          -- 建议为系统内存的 25%
effective_cache_size = '1GB'      -- 建议为系统内存的 75%
work_mem = '64MB'                  -- 每个排序/哈希操作

-- 并行配置
max_worker_processes = 8           -- CPU 核心数
max_parallel_workers_per_gather = 4
parallel_tuple_cost = 0.1
parallel_setup_cost = 1000

-- WAL 配置
wal_buffers = '64MB'
checkpoint_completion_target = 0.9

-- 连接配置
max_connections = 200
```

---

> [!TIP]
> 对于 AI 应用，推荐使用 PostgreSQL + pgvector 组合，可以同时处理结构化数据和向量数据，减少系统复杂度。

---

```

### 备份脚本

```bash
#!/bin/bash
# backup.sh - PostgreSQL 备份脚本

set -e

BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

mkdir -p "$BACKUP_DIR"

# 逻辑备份
echo "Starting logical backup..."
pg_dump -U postgres -d mydb -F c -Z 5 -f "$BACKUP_DIR/mydb_$DATE.dump"

# 保留最近 N 天的备份
find "$BACKUP_DIR" -name "*.dump" -mtime +$RETENTION_DAYS -delete

# 备份完成
echo "Backup completed: mydb_$DATE.dump"
ls -lh "$BACKUP_DIR"
```

---

## PL/pgSQL 存储过程详解

### 基础语法

```sql
-- 基础函数
CREATE OR REPLACE FUNCTION get_user_count()
RETURNS INTEGER AS $$
DECLARE
    count INTEGER;
BEGIN
    SELECT COUNT(*) INTO count FROM users;
    RETURN count;
END;
$$ LANGUAGE plpgsql;

-- 带参数函数
CREATE OR REPLACE FUNCTION get_user_by_email(user_email VARCHAR)
RETURNS users AS $$
DECLARE
    user_record users%ROWTYPE;
BEGIN
    SELECT * INTO user_record FROM users WHERE email = user_email;
    IF NOT FOUND THEN
        RAISE EXCEPTION 'User not found: %', user_email;
    END IF;
    RETURN user_record;
END;
$$ LANGUAGE plpgsql;

-- 返回表
CREATE OR REPLACE FUNCTION get_active_users()
RETURNS TABLE(id BIGINT, name VARCHAR, email VARCHAR) AS $$
BEGIN
    RETURN QUERY
    SELECT u.id, u.name, u.email
    FROM users u
    WHERE u.is_active = true;
END;
$$ LANGUAGE plpgsql;

-- 条件逻辑
CREATE OR REPLACE FUNCTION calculate_discount(total DECIMAL)
RETURNS DECIMAL AS $$
DECLARE
    discount_rate DECIMAL;
BEGIN
    IF total > 1000 THEN
        discount_rate := 0.15;
    ELSIF total > 500 THEN
        discount_rate := 0.10;
    ELSIF total > 100 THEN
        discount_rate := 0.05;
    ELSE
        discount_rate := 0;
    END IF;
    
    RETURN total * discount_rate;
END;
$$ LANGUAGE plpgsql;

-- 循环
CREATE OR REPLACE FUNCTION generate_series(n INTEGER)
RETURNS TABLE(value INTEGER) AS $$
DECLARE
    i INTEGER := 1;
BEGIN
    WHILE i <= n LOOP
        RETURN QUERY SELECT i;
        i := i + 1;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- FOR 循环（查询结果）
CREATE OR REPLACE FUNCTION process_pending_orders()
RETURNS VOID AS $$
DECLARE
    order_record RECORD;
BEGIN
    FOR order_record IN 
        SELECT * FROM orders WHERE status = 'pending'
    LOOP
        -- 处理每个订单
        UPDATE orders SET status = 'processing' WHERE id = order_record.id;
    END LOOP;
END;
$$ LANGUAGE plpgsql;

-- 异常处理
CREATE OR REPLACE FUNCTION safe_divide(a DECIMAL, b DECIMAL)
RETURNS DECIMAL AS $$
BEGIN
    IF b = 0 THEN
        RAISE WARNING 'Division by zero, returning NULL';
        RETURN NULL;
    END IF;
    RETURN a / b;
EXCEPTION
    WHEN division_by_zero THEN
        RETURN NULL;
    WHEN OTHERS THEN
        RAISE WARNING 'Unexpected error: %', SQLERRM;
        RETURN NULL;
END;
$$ LANGUAGE plpgsql;

-- 事务控制
CREATE OR REPLACE FUNCTION transfer_funds(
    from_account BIGINT,
    to_account BIGINT,
    amount DECIMAL
)
RETURNS BOOLEAN AS $$
BEGIN
    -- 检查余额
    IF (SELECT balance FROM accounts WHERE id = from_account) < amount THEN
        RAISE EXCEPTION 'Insufficient funds';
    END IF;
    
    -- 开始事务
    BEGIN
        -- 扣款
        UPDATE accounts SET balance = balance - amount WHERE id = from_account;
        
        -- 存款
        UPDATE accounts SET balance = balance + amount WHERE id = to_account;
        
        -- 记录交易
        INSERT INTO transactions (from_account, to_account, amount) 
        VALUES (from_account, to_account, amount);
        
        RETURN TRUE;
    EXCEPTION
        WHEN OTHERS THEN
            RAISE NOTICE 'Transfer failed: %', SQLERRM;
            RETURN FALSE;
    END;
END;
$$ LANGUAGE plpgsql;

-- 触发器函数
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_users_timestamp
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at();

-- 审计触发器
CREATE OR REPLACE FUNCTION audit_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, action, new_data)
        VALUES (TG_TABLE_NAME, 'INSERT', to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, action, old_data, new_data)
        VALUES (TG_TABLE_NAME, 'UPDATE', to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, action, old_data)
        VALUES (TG_TABLE_NAME, 'DELETE', to_jsonb(OLD));
        RETURN OLD;
    END IF;
    RETURN NULL;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER audit_users_changes
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION audit_changes();

-- 游标使用
CREATE OR REPLACE FUNCTION process_large_batch()
RETURNS VOID AS $$
DECLARE
    rec RECORD;
    cur CURSOR FOR SELECT * FROM orders WHERE processed = false;
BEGIN
    OPEN cur;
    LOOP
        FETCH cur INTO rec;
        EXIT WHEN NOT FOUND;
        
        -- 处理每条记录
        -- ...
        
        UPDATE orders SET processed = true WHERE id = rec.id;
    END LOOP;
    CLOSE cur;
END;
$$ LANGUAGE plpgsql;

-- 动态 SQL
CREATE OR REPLACE FUNCTION dynamic_count(table_name VARCHAR)
RETURNS BIGINT AS $$
DECLARE
    sql_stmt TEXT;
    result BIGINT;
BEGIN
    sql_stmt := 'SELECT COUNT(*) FROM ' || quote_ident(table_name);
    EXECUTE sql_stmt INTO result;
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- 返回多个结果集
CREATE OR REPLACE FUNCTION get_stats()
RETURNS TABLE(
    users_count BIGINT,
    orders_count BIGINT,
    total_revenue DECIMAL
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        (SELECT COUNT(*) FROM users)::BIGINT,
        (SELECT COUNT(*) FROM orders)::BIGINT,
        (SELECT COALESCE(SUM(total), 0) FROM orders)::DECIMAL;
END;
$$ LANGUAGE plpgsql;
```

### 高级 PL/pgSQL 技巧

```sql
-- 使用数组
CREATE OR REPLACE FUNCTION get_user_permissions(user_id BIGINT)
RETURNS TEXT[] AS $$
DECLARE
    permissions TEXT[];
BEGIN
    SELECT ARRAY_AGG(permission_name) INTO permissions
    FROM user_permissions
    WHERE user_id = get_user_permissions.user_id;
    
    RETURN COALESCE(permissions, ARRAY[]::TEXT[]);
END;
$$ LANGUAGE plpgsql;

-- 使用 JSON/JSONB
CREATE OR REPLACE FUNCTION user_to_json(user_id BIGINT)
RETURNS JSONB AS $$
DECLARE
    result JSONB;
BEGIN
    SELECT jsonb_build_object(
        'id', id,
        'name', name,
        'email', email,
        'created_at', created_at
    ) INTO result
    FROM users
    WHERE id = user_id;
    
    RETURN result;
END;
$$ LANGUAGE plpgsql;

-- 递归 CTE 在函数中
CREATE OR REPLACE FUNCTION get_category_tree(parent_id BIGINT)
RETURNS TABLE(
    id BIGINT,
    name VARCHAR,
    parent_id BIGINT,
    level INT
) AS $$
BEGIN
    RETURN QUERY
    WITH RECURSIVE category_tree AS (
        SELECT id, name, parent_id, 0 AS level
        FROM categories
        WHERE id = parent_id
        
        UNION ALL
        
        SELECT c.id, c.name, c.parent_id, ct.level + 1
        FROM categories c
        JOIN category_tree ct ON c.parent_id = ct.id
        WHERE ct.level < 10
    )
    SELECT * FROM category_tree;
END;
$$ LANGUAGE plpgsql;

-- 批量操作
CREATE OR REPLACE FUNCTION batch_update_status(
    order_ids BIGINT[],
    new_status VARCHAR
)
RETURNS INT AS $$
DECLARE
    updated_count INT;
BEGIN
    UPDATE orders
    SET status = new_status, updated_at = NOW()
    WHERE id = ANY(order_ids);
    
    GET DIAGNOSTICS updated_count = ROW_COUNT;
    RETURN updated_count;
END;
$$ LANGUAGE plpgsql;

-- 窗口函数配合
CREATE OR REPLACE FUNCTION get_employee_ranking()
RETURNS TABLE(
    employee_id BIGINT,
    name VARCHAR,
    department VARCHAR,
    salary DECIMAL,
    rank BIGINT,
    department_avg DECIMAL
) AS $$
BEGIN
    RETURN QUERY
    SELECT 
        e.id, e.name, e.department, e.salary,
        RANK() OVER (PARTITION BY e.department ORDER BY e.salary DESC),
        AVG(e.salary) OVER (PARTITION BY e.department)
    FROM employees e;
END;
$$ LANGUAGE plpgsql;

-- 物化路径
CREATE OR REPLACE FUNCTION get_descendants(path_prefix VARCHAR)
RETURNS TABLE(id BIGINT, name VARCHAR, path VARCHAR) AS $$
BEGIN
    RETURN QUERY
    SELECT id, name, path
    FROM categories
    WHERE path LIKE path_prefix || '%';
END;
$$ LANGUAGE plpgsql;
```

---

## 分区表进阶

### 分区策略选择

```sql
-- 1. RANGE 分区（按时间）
CREATE TABLE logs (
    id BIGSERIAL,
    created_at TIMESTAMP NOT NULL,
    level VARCHAR(10),
    message TEXT
) PARTITION BY RANGE (created_at);

-- 按月分区
CREATE TABLE logs_2024_01 PARTITION OF logs
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE logs_2024_02 PARTITION OF logs
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

CREATE TABLE logs_2024_03 PARTITION OF logs
    FOR VALUES FROM ('2024-03-01') TO ('2024-04-01');

-- 按年分区
CREATE TABLE logs_2023 PARTITION OF logs
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

-- 2. LIST 分区（按类别）
CREATE TABLE products (
    id BIGSERIAL,
    category VARCHAR(50),
    name VARCHAR(200),
    price DECIMAL
) PARTITION BY LIST (category);

CREATE TABLE products_electronics PARTITION OF products
    FOR VALUES IN ('electronics', 'computers', 'phones');

CREATE TABLE products_clothing PARTITION OF products
    FOR VALUES IN ('clothing', 'shoes', 'accessories');

CREATE TABLE products_other PARTITION OF products
    FOR VALUES IN (DEFAULT);

-- 3. HASH 分区（均匀分布）
CREATE TABLE user_sessions (
    id BIGSERIAL,
    user_id BIGINT,
    session_token VARCHAR,
    created_at TIMESTAMP
) PARTITION BY HASH (user_id);

-- 创建 8 个哈希分区
CREATE TABLE user_sessions_0 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 0);
CREATE TABLE user_sessions_1 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 1);
CREATE TABLE user_sessions_2 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 2);
CREATE TABLE user_sessions_3 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 3);
CREATE TABLE user_sessions_4 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 4);
CREATE TABLE user_sessions_5 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 5);
CREATE TABLE user_sessions_6 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 6);
CREATE TABLE user_sessions_7 PARTITION OF user_sessions
    FOR VALUES WITH (MODULUS 8, REMAINDER 7);

-- 4. 多列分区
CREATE TABLE sales (
    region VARCHAR(50),
    sale_date DATE,
    amount DECIMAL
) PARTITION BY RANGE (region, sale_date);

CREATE TABLE sales_north_2024 PARTITION OF sales
    FOR VALUES FROM (('North', '2024-01-01') TO ('North', '2025-01-01'));

CREATE TABLE sales_south_2024 PARTITION OF sales
    FOR VALUES FROM (('South', '2024-01-01') TO ('South', '2025-01-01'));

-- 5. 子分区
CREATE TABLE big_table (
    id BIGSERIAL,
    created_at DATE,
    status VARCHAR(20),
    data JSONB
) PARTITION BY RANGE (created_at);

CREATE TABLE big_table_2024 PARTITION OF big_table
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01')
    SUBPARTITION BY LIST (status);

CREATE TABLE big_table_2024_active PARTITION OF big_table_2024
    FOR VALUES IN ('active');
CREATE TABLE big_table_2024_inactive PARTITION OF big_table_2024
    FOR VALUES IN ('inactive');
CREATE TABLE big_table_2024_pending PARTITION OF big_table_2024
    FOR VALUES IN ('pending');
```

### 分区维护

```sql
-- 查看分区信息
SELECT 
    parent.relname AS parent_table,
    child.relname AS partition_name,
    pg_get_expr(child.relpartbound, child.oid) AS partition_bound
FROM pg_inherits
JOIN pg_class parent ON pg_inherits.inhparent = parent.oid
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
ORDER BY parent.relname, child.relname;

-- 查看分区行数
SELECT 
    child.relname AS partition,
    pg_stat_get_live_tuples(child.oid) AS row_count
FROM pg_class parent
JOIN pg_inherits ON parent.oid = pg_inherits.inhparent
JOIN pg_class child ON pg_inherits.inhrelid = child.oid
WHERE parent.relname = 'logs';

-- 添加分区
ALTER TABLE logs ADD PARTITION logs_2024_04
    FOR VALUES FROM ('2024-04-01') TO ('2024-05-01');

-- 删除分区（同时删除数据）
ALTER TABLE logs DROP PARTITION logs_2023;

-- 分离分区
ALTER TABLE logs DETACH PARTITION logs_2023;

-- 将分离的分区变成独立表
ALTER TABLE logs_2023 SET SCHEMA archived;
ALTER TABLE logs_2023 SET logged;

-- 重新附加分区
ALTER TABLE logs ATTACH PARTITION logs_2023
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

-- 分割分区
ALTER TABLE logs_2024_01 SPLIT PARTITION logs_2024_01 
    AT (ROW('2024-01-16')) INTO (
        TABLE logs_2024_01a,
        TABLE logs_2024_01b
    );

-- 合并分区
ALTER TABLE logs_2024_01a MERGE PARTITIONS 
    logs_2024_01a, logs_2024_01b INTO PARTITION logs_2024_01_merged;

-- 重建分区（改善性能）
ALTER TABLE logs DETACH PARTITION logs_2024_01;
VACUUM FULL logs_2024_01;
ALTER TABLE logs ATTACH PARTITION logs_2024_01
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
```

---

## PgBouncer 连接池

### 安装与配置

```bash
# Ubuntu/Debian
sudo apt install pgbouncer

# macOS
brew install pgbouncer
```

```ini
# /etc/pgbouncer/pgbouncer.ini

[databases]
; 指向真实 PostgreSQL
mydb = host=127.0.0.1 port=5432 dbname=mydb

; 多个数据库
mydb2 = host=127.0.0.1 port=5432 dbname=mydb2

; 远程数据库
remotedb = host=remote.host.com port=5432 dbname=remotedb

[pgbouncer]
; 监听地址
listen_addr = 127.0.0.1
listen_port = 6432

; 连接池模式
; session: 客户端会话模式（默认）
; transaction: 事务模式（推荐）
; statement: 语句模式
pool_mode = transaction

; 最大连接数（指向 PostgreSQL）
max_client_conn = 1000

; 池中最大空闲连接数
; 默认等于 max_client_conn
; pool_size = 50

; 每个用户最大连接数
max_db_connections = 100

; 空闲超时（秒）
; 超过此时间未使用的连接会被关闭
idle_timeout = 600

; 服务器连接超时
server_connect_timeout = 15

; 服务器空闲超时
server_idle_timeout = 600

; 服务器生命周期
server_lifetime = 3600

; 保留备用连接
reserve_pool_size = 5
reserve_pool_timeout = 3

; 客户端超时
query_wait_timeout = 30

; 日志
log_connections = 0
log_disconnections = 0
log_pooler_errors = 1

; 管理命令
admin_users = postgres, pgbouncer_admin

; 认证类型
; md5, crypt, trust, plain
auth_type = md5
auth_file = /etc/pgbouncer/userlist.txt

; 忽略启动参数
ignore_startup_parameters = extra_float_digits
```

### 用户列表文件

```bash
# /etc/pgbouncer/userlist.txt
"postgres" "md5abcdef1234567890abcdef12345678"
"app_user" "md5b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"

# 生成密码哈希
# echo "mypassword" | pg_md5
# md5abcdef1234567890abcdef12345678
```

### PgBouncer 管理命令

```bash
# 连接 PgBouncer 管理界面
psql -h 127.0.0.1 -p 6432 -U postgres pgbouncer

# 查看统计
pgbouncer SHOW STAT;

# 查看池
pgbouncer SHOW POOLS;

# 查看客户端
pgbouncer SHOW CLIENTS;

# 查看服务器
pgbouncer SHOW SERVERS;

# 查看配置
pgbouncer SHOW CONFIG;

# 查看版本
pgbouncer SHOW VERSION;

# 重新加载配置
pgbouncer RELOAD;

# 暂停所有新连接
pgbouncer PAUSE;

# 恢复连接
pgbouncer RESUME;

# 禁用/启用数据库
pgbouncer DISABLE mydb;
pgbouncer ENABLE mydb;

# 杀死客户端连接
pgbouncer KILL user_name;

# 杀死所有空闲连接
pgbouncer KILL IDLE;

# 重置统计
pgbouncer RESET STAT;

# 退出
pgbouncer SHUTDOWN;
```

### Docker Compose 配置

```yaml
version: "3.8"
services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  pgbouncer:
    image: edoburu/pgbouncer:latest
    environment:
      DATABASE_URL: "postgres://postgres:password@postgres:5432/mydb"
      POOL_MODE: transaction
      MAX_CLIENT_CONN: 1000
      DEFAULT_POOL_SIZE: 25
    ports:
      - "5432:5432"
    depends_on:
      postgres:
        condition: service_healthy

volumes:
  postgres_data:
```

### 应用连接配置

```typescript
// Node.js 连接 PgBouncer
import { Pool } from 'pg';

const pool = new Pool({
  host: 'localhost',       // PgBouncer 地址
  port: 6432,            // PgBouncer 端口
  database: 'mydb',
  user: 'app_user',
  password: 'password',
  max: 20,               // 应用层连接池大小
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// 使用
const client = await pool.connect();
try {
  await client.query('SELECT * FROM users');
} finally {
  client.release();
}
```

```python
# Python 连接 PgBouncer
import psycopg2

conn = psycopg2.connect(
    host='localhost',
    port=6432,
    database='mydb',
    user='app_user',
    password='password'
)

cursor = conn.cursor()
cursor.execute('SELECT * FROM users')
```

---

## 监控与诊断

### 关键监控查询

```sql
-- 1. 连接状态
SELECT 
    state,
    COUNT(*) AS count,
    ARRAY_AGG(usename ORDER BY usename) AS users
FROM pg_stat_activity
GROUP BY state;

-- 2. 慢查询（超过 1 秒）
SELECT 
    pid,
    now() - query_start AS duration,
    state,
    usename,
    query
FROM pg_stat_activity
WHERE state != 'idle'
    AND query_start < NOW() - INTERVAL '1 second'
ORDER BY duration DESC;

-- 3. 锁等待
SELECT 
    blocked_locks.pid AS blocked_pid,
    blocking_locks.pid AS blocking_pid,
    blocked_activity.usename AS blocked_user,
    blocking_activity.usename AS blocking_user,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query,
    blocked_activity.application_name AS blocked_app,
    blocking_activity.application_name AS blocking_app
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
    ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks 
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
    AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
    AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
    AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
    AND blocking_locks.virtualxid IS NOT DISTINCT FROM blocked_locks.virtualxid
    AND blocking_locks.transactionid IS NOT DISTINCT FROM blocked_locks.transactionid
    AND blocking_locks.classid IS NOT DISTINCT FROM blocked_locks.classid
    AND blocking_locks.objid IS NOT DISTINCT FROM blocked_locks.objid
    AND blocking_locks.objsubid IS NOT DISTINCT FROM blocked_locks.objsubid
    AND blocking_locks.pid != blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity 
    ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;

-- 4. 表膨胀
SELECT 
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    pg_size_pretty(pg_relation_size(schemaname||'.'||tablename)) AS table_size,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename) 
        - pg_relation_size(schemaname||'.'||tablename)) AS indexes_size,
    n_dead_tup,
    n_live_tup,
    n_dead_tup + n_live_tup AS total_tuples,
    ROUND(n_dead_tup * 100.0 / NULLIF(n_dead_tup + n_live_tup, 0), 2) AS dead_tuple_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;

-- 5. 索引使用情况
SELECT 
    schemaname,
    relname AS table_name,
    indexrelname AS index_name,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
    AND indexrelname NOT LIKE '%_pkey'
ORDER BY pg_relation_size(indexrelid) DESC;

-- 6. 缓冲池命中率
SELECT 
    pg_stat_bgwriter.buffers_checkpoint,
    pg_stat_bgwriter.buffers_clean,
    pg_stat_bgwriter.buffers_backend,
    pg_stat_bgwriter.buffers_alloc,
    pg_stat_bgwriter.checkpoints_timed,
    pg_stat_bgwriter.checkpoints_req,
    pg_stat_bgwriter.checkpoint_write_time,
    pg_stat_bgwriter.checkpoint_sync_time
FROM pg_stat_bgwriter;

-- 7. 复制状态（主库）
SELECT 
    client_addr,
    usename,
    application_name,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    write_lag,
    flush_lag,
    replay_lag,
    sync_state
FROM pg_stat_replication;

-- 8. 复制状态（从库）
SELECT 
    pg_last_wal_receive_lsn() AS received_lsn,
    pg_last_wal_replay_lsn() AS replay_lsn,
    pg_last_wal_receive_lsn() - pg_last_wal_replay_lsn() AS replication_lag,
    pg_last_xact_replay_timestamp() AS last_replay_time;

-- 9. VACUUM 进度
SELECT 
    relname,
    phase,
    heap_blks_total,
    heap_blks_scanned,
    heap_blks_vacuumed,
    index_vacuum_count,
    max_dead_tuples,
    num_dead_tuples
FROM pg_stat_progress_vacuum;

-- 10. ANALYZE 进度
SELECT 
    relname,
    phase,
    sample_blks_total,
    sample_blks_scanned,
    sample_blks_total - sample_blks_scanned AS remaining_blks
FROM pg_stat_progress_analyze;
```

### pg_stat_statments 扩展

```sql
-- 安装扩展
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- 查看最慢查询
SELECT 
    query,
    calls,
    total_exec_time / 1000 AS total_seconds,
    mean_exec_time AS mean_ms,
    max_exec_time AS max_ms,
    rows,
    shared_blks_hit,
    shared_blks_read,
    shared_blks_dirtied,
    shared_blks_written
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;

-- 查看最频繁查询
SELECT 
    query,
    calls,
    total_exec_time / 1000 AS total_seconds,
    rows,
    rows / NULLIF(calls, 0) AS avg_rows
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 20;

-- 查看消耗最多 I/O 的查询
SELECT 
    query,
    shared_blks_read,
    shared_blks_hit,
    shared_blks_dirtied,
    shared_blks_written,
    local_blks_read,
    local_blks_written,
    temp_blks_read,
    temp_blks_written
FROM pg_stat_statements
ORDER BY shared_blks_read DESC
LIMIT 20;

-- 重置统计
SELECT pg_stat_statements_reset();
```

---

## 高可用与复制

### 流复制配置

```sql
-- 1. 主库配置 (postgresql.conf)
# 启用 WAL 归档
wal_level = replica
max_wal_senders = 10
max_replication_slots = 10
wal_keep_size = 1GB

# 归档配置
archive_mode = on
archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'

# 复制槽（确保从库能获取所有 WAL）
max_replication_slots = 10

# 监控
hot_standby = on
hot_standby_feedback = on

-- 2. 主库创建复制用户
CREATE USER replicator WITH REPLICATION ENCRYPTED PASSWORD 'replication_password';

-- 3. 主库配置 pg_hba.conf（允许复制）
# IPv4 本地复制
local   replication     all                                     trust
# IPv4 远程复制
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     replicator      192.168.1.0/24            scram-sha-256

-- 4. 从库配置 (postgresql.conf)
hot_standby = on
primary_conninfo = 'host=192.168.1.100 port=5432 user=replicator application_name=standby1'

-- 5. 从库创建复制槽
-- 在从库上执行
pg_basebackup -h master_host -U replicator -D /var/lib/postgresql/data -P -Xs -R

-- 或手动创建
touch /var/lib/postgresql/data/standby.signal
# 创建 standby.signal 使其成为从库

-- 6. 使用复制槽的流复制
# 主库创建复制槽
SELECT * FROM pg_create_physical_replication_slot('standby1_slot');

# 从库配置
primary_slot_name = 'standby1_slot'
```

### 故障转移与切换

```sql
-- 1. 检查复制状态
SELECT * FROM pg_stat_replication;

-- 2. 强制复制
SELECT pg_switch_wal();

-- 3. 等待复制到指定 LSN
SELECT pg_wait_for_lsn('0/3000000');

-- 4. 创建故障转移触发文件
-- 当需要故障转移时，从库上执行
SELECT pg_promote();

-- 或创建触发文件
CREATE TRIGGER file: /var/lib/postgresql/data/promote
-- 或
touch /var/lib/postgresql/data/promote

-- 5. Patroni 或 pgpool-II 用于自动故障转移
```

### 逻辑复制

```sql
-- 1. 主库配置
wal_level = logical
max_replication_slots = 10
max_wal_senders = 10

-- 2. 创建发布
CREATE PUBLICATION myapp_publication FOR TABLE users, orders, products;

-- 发布特定行
CREATE PUBLICATION active_users FOR TABLE users 
    WHERE (status = 'active');

-- 发布所有表
CREATE PUBLICATION full_publication FOR ALL TABLES;

-- 3. 从库配置
-- 创建订阅
CREATE SUBSCRIPTION myapp_subscription 
    CONNECTION 'host=master_host port=5432 dbname=mydb user=replicator'
    PUBLICATION myapp_publication
    WITH (copy_data = true);

-- 4. 管理订阅
ALTER SUBSCRIPTION myapp_subscription ENABLE;
ALTER SUBSCRIPTION myapp_subscription DISABLE;
ALTER SUBSCRIPTION myapp_subscription REFRESH PUBLICATION;

-- 5. 查看订阅状态
SELECT * FROM pg_stat_subscription;

-- 6. 删除订阅
DROP SUBSCRIPTION myapp_subscription;
```

---

> [!SUCCESS]
> 本文档全面介绍了 PostgreSQL 的核心特性、17 新功能、JSON 支持、全文搜索、向量搜索以及在 AI 应用中的实战场景。PostgreSQL 凭借其功能完备性、扩展性和稳定性，是现代 AI 应用数据层的首选方案。

---

## 完整安装与环境配置

### 安装方法

```bash
# macOS
brew install postgresql@17
brew services start postgresql@17

# Ubuntu/Debian
sudo apt update
sudo apt install postgresql-17

# Docker
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  postgres:17-alpine

# Docker Compose
version: "3.8"
services:
  postgres:
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### 客户端连接

```bash
# psql 命令行
psql -h localhost -U postgres -d mydb

# 常用 psql 命令
\l                          -- 列出所有数据库
\dt                         -- 列出所有表
\d table_name               -- 查看表结构
\du                         -- 列出所有用户
\di                         -- 列出所有索引
\dv                         -- 列出所有视图
\dn                         -- 列出所有 schema
\df                         -- 列出所有函数
\x                          -- 切换扩展显示模式
\i filename.sql            -- 执行 SQL 文件
\copy table TO 'file.csv' CSV HEADER  -- 导出数据

# 数据库操作
CREATE DATABASE mydb;
DROP DATABASE mydb;
ALTER DATABASE mydb RENAME TO newdb;
```

### psqlrc 配置

```bash
# ~/.psqlrc
-- 显示执行时间
\timing

-- 扩展显示
\x auto

-- 别名
\pset null '(null)'
\pset format wrapped
\pset border 2

-- 自动提交
\set AUTOCOMMIT on

-- 编辑器
\set EDITOR vim

-- 提示符
\pset prompt1 '%[%033[1;32m%]%M/%[%033[0m%]%/%R%#%x '
\pset prompt2 '%[%033[1;32m%]%M/%/%R%#%x '
```

---

## 数据库设计模式

### 常见设计模式

```sql
-- 模式 1：单一 Schema（默认 public）
-- 适用于小型应用
CREATE SCHEMA public;

-- 模式 2：多 Schema 按功能划分
-- 适用于中型应用
CREATE SCHEMA sales;
CREATE SCHEMA inventory;
CREATE SCHEMA analytics;

-- 设置搜索路径
ALTER DATABASE mydb SET search_path TO sales, inventory, public;

-- 模式 3：多租户（Schema 隔离）
CREATE SCHEMA tenant_1;
CREATE SCHEMA tenant_2;

-- 模式 4：行级安全（RLS）
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON users
    USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

### 常见表结构设计

```sql
-- 1. 主从表设计
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    parent_id UUID REFERENCES categories(id),
    path LTREE,  -- 层级路径
    level INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW()
);

-- 2. 多对多关系
CREATE TABLE post_tags (
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
    tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- 3. 软删除设计
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    deleted_at TIMESTAMP,  -- 软删除标记
    CONSTRAINT users_deleted_at_check CHECK (deleted_at IS NULL OR deleted_at > created_at)
);

-- 查询时自动过滤已删除记录
CREATE VIEW active_users AS
SELECT * FROM users WHERE deleted_at IS NULL;

-- 4. 审计表设计
CREATE TABLE audit_log (
    id BIGSERIAL PRIMARY KEY,
    table_name VARCHAR(100) NOT NULL,
    record_id UUID NOT NULL,
    action VARCHAR(20) NOT NULL,  -- INSERT, UPDATE, DELETE
    old_data JSONB,
    new_data JSONB,
    changed_by UUID,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- 触发器实现审计
CREATE OR REPLACE FUNCTION audit_trigger()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO audit_log (table_name, record_id, action, new_data)
        VALUES (TG_TABLE_NAME, NEW.id, 'INSERT', to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO audit_log (table_name, record_id, action, old_data, new_data)
        VALUES (TG_TABLE_NAME, OLD.id, 'UPDATE', to_jsonb(OLD), to_jsonb(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO audit_log (table_name, record_id, action, old_data)
        VALUES (TG_TABLE_NAME, OLD.id, 'DELETE', to_jsonb(OLD));
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER users_audit
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW EXECUTE FUNCTION audit_trigger();
```

---

## 查询优化实战

### 慢查询分析

```sql
-- 查看当前慢查询
SELECT 
    pid,
    now() - query_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state != 'idle'
    AND query_start < NOW() - INTERVAL '5 minutes'
ORDER BY duration DESC;

-- 终止慢查询
SELECT pg_cancel_backend(pid);  -- 优雅终止
SELECT pg_terminate_backend(pid);  -- 强制终止

-- 查看统计信息
SELECT 
    schemaname,
    relname,
    seq_scan,
    seq_tup_read,
    idx_scan,
    idx_tup_fetch,
    n_tup_ins,
    n_tup_upd,
    n_tup_del,
    n_live_tup,
    n_dead_tup
FROM pg_stat_user_tables
ORDER BY seq_scan DESC;

-- 查看索引使用情况
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

### 索引优化

```sql
-- 1. B-tree 索引（默认）
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC);

-- 2. 部分索引
CREATE INDEX idx_posts_published ON posts(published_at DESC)
WHERE published = true;

CREATE INDEX idx_users_active ON users(email)
WHERE is_active = true;

-- 3. 表达式索引
CREATE INDEX idx_users_email_lower ON users(LOWER(email));
CREATE INDEX idx_users_created_month ON users(DATE_TRUNC('month', created_at));

-- 4. 模糊查询索引（使用 pg_trgm）
CREATE EXTENSION IF NOT EXISTS pg_trgm;
CREATE INDEX idx_users_username_trgm ON users USING gin (username gin_trgm_ops);

-- 模糊查询示例
SELECT * FROM users WHERE username LIKE '%john%';

-- 5. JSONB 索引
CREATE INDEX idx_orders_metadata ON orders USING gin (metadata jsonb_path_ops);
CREATE INDEX idx_users_data_email ON users((data->>'email'));

-- 6. 向量索引
CREATE INDEX idx_embeddings_vector ON embeddings 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- 7. 复合索引顺序
-- 假设查询: WHERE status = 'active' AND created_at > '2024-01-01'
-- 正确顺序：选择性高的放前面
CREATE INDEX idx_users_status_created ON users(status, created_at DESC);

-- 8. 索引维护
REINDEX INDEX CONCURRENTLY idx_users_email;
VACUUM ANALYZE users;

-- 查看未使用的索引
SELECT 
    schemaname || '.' || tablename AS table,
    indexname,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
    idx_scan
FROM pg_stat_user_indexes
WHERE idx_scan = 0
    AND NOT indexname LIKE '%_pkey'
ORDER BY pg_relation_size(indexrelid) DESC;
```

### 查询优化技巧

```sql
-- 1. 使用 EXPLAIN ANALYZE
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT * FROM users WHERE email = 'test@example.com';

-- 2. 避免 SELECT *
SELECT id, email FROM users WHERE id = 1;

-- 3. 使用 LIMIT 避免大结果集
SELECT * FROM posts ORDER BY created_at LIMIT 100;

-- 4. 批量插入
INSERT INTO users (email, username) 
VALUES 
    ('user1@example.com', 'user1'),
    ('user2@example.com', 'user2'),
    ('user3@example.com', 'user3');

-- 5. 使用 CTE 优化复杂查询
WITH recent_posts AS (
    SELECT id, title, author_id 
    FROM posts 
    WHERE published_at > NOW() - INTERVAL '30 days'
)
SELECT 
    u.username,
    COUNT(rp.id) AS post_count
FROM users u
JOIN recent_posts rp ON rp.author_id = u.id
GROUP BY u.username
ORDER BY post_count DESC;

-- 6. 使用窗口函数替代子查询
-- ❌ 低效
SELECT 
    u.username,
    (SELECT COUNT(*) FROM posts WHERE author_id = u.id) AS post_count
FROM users u;

-- ✅ 高效
SELECT 
    username,
    COUNT(*) OVER (PARTITION BY author_id) AS post_count
FROM posts p
JOIN users u ON u.id = p.author_id;

-- 7. 分页优化
-- ❌ 偏移分页（慢）
SELECT * FROM posts ORDER BY id LIMIT 100 OFFSET 10000;

-- ✅ 游标分页（快）
SELECT * FROM posts 
WHERE id > 10000 
ORDER BY id 
LIMIT 100;

-- ✅ Keyset 分页
SELECT * FROM posts 
WHERE published_at < '2024-01-01' 
ORDER BY published_at DESC 
LIMIT 100;
```

---

## 事务与并发控制

### 事务隔离级别

```sql
-- 设置隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 开始事务
BEGIN;

-- 保存点
SAVEPOINT sp1;

-- 回滚到保存点
ROLLBACK TO SAVEPOINT sp1;

-- 提交
COMMIT;

-- 回滚
ROLLBACK;
```

### 行级锁

```sql
-- 1. 悲观锁
SELECT * FROM users WHERE id = 1 FOR UPDATE;

-- 2. 乐观锁（使用版本号）
UPDATE users 
SET email = 'new@example.com', version = version + 1
WHERE id = 1 AND version = 5;

-- 检查更新是否成功
-- 如果 version != 5，说明被其他事务修改，需要重试

-- 3. SKIP LOCKED（并发处理）
SELECT * FROM orders 
WHERE status = 'pending' 
ORDER BY created_at 
LIMIT 10 
FOR UPDATE SKIP LOCKED;

-- 4. NOWAIT（立即报错）
SELECT * FROM users WHERE id = 1 FOR UPDATE NOWAIT;

-- 死锁检测
SELECT 
    pg_blocking_pids(p.pid) AS blocked_by,
    p.pid,
    p.query
FROM pg_stat_activity p
WHERE pg_blocking_pids(p.pid) IS NOT NULL
ORDER BY p.pid;
```

### 并发控制模式

```sql
-- 模式 1：基于数据库锁
BEGIN;
SELECT * FROM inventory WHERE product_id = 100 FOR UPDATE;
-- 检查库存
UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 100;
COMMIT;

-- 模式 2：原子更新
UPDATE inventory 
SET quantity = quantity - 1 
WHERE product_id = 100 AND quantity > 0
RETURNING *;

-- 检查是否成功
-- 如果没有返回行，说明库存不足

-- 模式 3：序列号
UPDATE accounts 
SET balance = balance - 100 
WHERE id = 1 AND balance >= 100
RETURNING *;
```

---

## 备份与恢复

### 备份类型

PostgreSQL 支持多种备份策略，适用于不同的场景：

| 备份类型 | 工具 | 特点 | 适用场景 |
|---------|------|------|----------|
| **物理备份** | pg_basebackup | 快速、完整复制数据文件 | 灾难恢复 |
| **逻辑备份** | pg_dump | 跨版本、可选择性恢复 | 应用级恢复 |
| **持续归档** | WAL + pg_receivewal | 零数据丢失 (PITR) | 企业级需求 |
| **增量备份** | pgBackRest | 高效、压缩、去重 | 大规模部署 |

### 逻辑备份（pg_dump）

```bash
# 备份单个数据库
pg_dump -U postgres -d mydb -F c -f backup.dump

# 备份所有数据库
pg_dumpall -U postgres -f all_databases.sql

# 备份特定表
pg_dump -U postgres -d mydb -t users -t posts -f tables.sql

# 备份结构（不包含数据）
pg_dump -U postgres -d mydb --schema-only -f schema.sql

# 备份数据（不包含结构）
pg_dump -U postgres -d mydb --data-only -f data.sql

# 压缩备份
pg_dump -U postgres -d mydb -F c | gzip > backup.dump.gz

# 并行备份（大型数据库）
pg_dump -U postgres -d mydb -j 4 -F d -f /backup/directory
```

### 恢复操作

```bash
# 恢复纯 SQL 格式备份
psql -U postgres -d mydb -f backup.sql

# 恢复压缩格式备份
pg_restore -U postgres -d mydb -c backup.dump

# 恢复前先创建数据库
createdb -U postgres newdb
pg_restore -U postgres -d newdb backup.dump

# 恢复特定表
pg_restore -U postgres -d mydb -t users backup.dump

# 恢复到指定时间点（需要 WAL 归档）
pg_restore -U postgres -d mydb --point-in-time="2024-01-15 10:00:00" backup.dump
```

### 物理备份（pg_basebackup）

```bash
# 基础备份
pg_basebackup -U postgres -D /backup/base -Ft -z -P -v

# 备份到远程服务器
pg_basebackup -U postgres -h primary_host -D /backup/base -Ft -z -P

# 使用复制槽（确保 WAL 不丢失）
pg_basebackup -U postgres -D /backup/base -Ft -z -P -X stream -S my_slot

# 查看备份状态
ls -la /backup/base/
```

### 持续归档与 PITR

```bash
# 配置 postgresql.conf 启用 WAL 归档
# wal_level = replica
# archive_mode = on
# archive_command = 'test ! -f /archive/%f && cp %p /archive/%f'

# 手动切换 WAL
psql -U postgres -c "SELECT pg_switch_wal();"

# 归档当前 WAL
psql -U postgres -c "SELECT pg_start_backup('backup_label');"
pg_basebackup -U postgres -D /backup/base -Ft -z -P
psql -U postgres -c "SELECT pg_stop_backup();"

# 恢复到指定时间点
cat > recovery.conf << 'EOF'
restore_command = 'cp /archive/%f %p'
recovery_target_time = '2024-01-15 10:00:00 UTC'
recovery_target_action = 'promote'
EOF
```

### 备份脚本示例

```bash
#!/bin/bash
# backup.sh - PostgreSQL 备份脚本

set -e

BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=30

# 创建备份目录
mkdir -p "$BACKUP_DIR"

# 逻辑备份
echo "Starting logical backup..."
pg_dump -U postgres -d mydb -F c -Z 5 -f "$BACKUP_DIR/mydb_$DATE.dump"

# 保留最近 N 天的备份
find "$BACKUP_DIR" -name "*.dump" -mtime +$RETENTION_DAYS -delete

# 备份完成
echo "Backup completed: mydb_$DATE.dump"
ls -lh "$BACKUP_DIR"
```

---

## Python 集成

### psycopg2 使用指南

```python
import psycopg2
from psycopg2 import pool
from contextlib import contextmanager

# 基本连接
conn = psycopg2.connect(
    host="localhost",
    database="mydb",
    user="postgres",
    password="password",
    port=5432
)

# 创建游标
cursor = conn.cursor()

# 执行查询
cursor.execute("SELECT * FROM users WHERE id = %s", (1,))
result = cursor.fetchone()

# 事务处理
try:
    cursor.execute("BEGIN")
    cursor.execute("INSERT INTO users (name, email) VALUES (%s, %s)", ("Alice", "alice@example.com"))
    cursor.execute("INSERT INTO orders (user_id, total) VALUES (%s, %s)", (1, 99.99))
    conn.commit()
except Exception as e:
    conn.rollback()
    print(f"Error: {e}")

# 使用 with 语句（自动管理）
with psycopg2.connect(conn_string) as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT * FROM users LIMIT 10")
        users = cur.fetchall()

# 连接池
connection_pool = pool.ThreadedConnectionPool(
    minconn=5,
    maxconn=20,
    host="localhost",
    database="mydb",
    user="postgres",
    password="password"
)

@contextmanager
def get_connection():
    conn = connection_pool.getconn()
    try:
        yield conn
        conn.commit()
    except:
        conn.rollback()
        raise
    finally:
        connection_pool.putconn(conn)

# 使用连接池
with get_connection() as conn:
    with conn.cursor() as cur:
        cur.execute("SELECT COUNT(*) FROM users")
        count = cur.fetchone()[0]
```

### SQLAlchemy ORM

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

# 创建引擎
engine = create_engine('postgresql://postgres:password@localhost:5432/mydb')
Session = sessionmaker(bind=engine)
Base = declarative_base()

# 定义模型
class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, nullable=False)
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
    func.avg(func.length(User.name)).label('avg_name_length')
).first()

# 联表查询
from sqlalchemy.orm import relationship

class Post(Base):
    __tablename__ = 'posts'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    title = Column(String(200))
    
    author = relationship("User", back_populates="posts")

User.posts = relationship("Post", back_populates="author")

# 使用联表
session.query(User, Post).join(Post, User.id == Post.user_id).filter(
    User.name == 'Alice'
).all()
```

### asyncpg（异步）

```python
import asyncio
import asyncpg

async def main():
    # 连接
    pool = await asyncpg.create_pool(
        host='localhost',
        database='mydb',
        user='postgres',
        password='password',
        min_size=5,
        max_size=20
    )
    
    async with pool.acquire() as conn:
        # 查询
        row = await conn.fetchrow(
            'SELECT * FROM users WHERE id = $1',
            1
        )
        print(row)
        
        # 批量查询
        rows = await conn.fetch(
            'SELECT * FROM users WHERE id = ANY($1)',
            [1, 2, 3, 4, 5]
        )
        
        # 执行
        await conn.execute('''
            INSERT INTO users (name, email) VALUES ($1, $2)
        ''', 'Bob', 'bob@example.com')
        
        # 事务
        async with conn.transaction():
            await conn.execute('INSERT INTO orders (user_id, total) VALUES ($1, $2)', 1, 99.99)
            await conn.execute('UPDATE users SET name = $1 WHERE id = $2', 'Bobby', 1)
    
    await pool.close()

asyncio.run(main())
```

---

## 常见陷阱与最佳实践

### 陷阱 1：滥用 JSONB

```sql
-- ❌ 错误：所有数据都用 JSONB
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    data JSONB  -- 不推荐，应该用列
);

-- ✅ 正确：结构化数据用列，动态数据用 JSONB
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    customer_id UUID NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    status VARCHAR(20) NOT NULL,
    metadata JSONB,  -- 用于存储可选的动态数据
    created_at TIMESTAMP DEFAULT NOW()
);
```

### 陷阱 2：忽视索引维护

```sql
-- ❌ 错误：不维护索引
-- 随着数据增长，索引会变得臃肿

-- ✅ 正确：定期维护
-- 1. 重建膨胀的索引
REINDEX INDEX CONCURRENTLY idx_orders_customer_id;

-- 2. 更新统计信息
ANALYZE orders;

-- 3. 清理死亡元组
VACUUM orders;
VACUUM (VERBOSE, ANALYZE) orders;

-- 4. 使用 pg_stat_user_indexes 监控
SELECT 
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS size
FROM pg_stat_user_indexes
WHERE schemaname = 'public'
ORDER BY idx_scan;
```

### 陷阱 3：错误的分页方式

```sql
-- ❌ 错误：OFFSET 大值时很慢
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 100000;

-- ✅ 正确：使用游标分页
-- 首次查询
SELECT * FROM posts ORDER BY id LIMIT 20;

-- 下一页
SELECT * FROM posts 
WHERE id > 20 
ORDER BY id 
LIMIT 20;

-- ✅ 使用时间戳分页
SELECT * FROM posts 
WHERE created_at < '2024-01-01'
ORDER BY created_at DESC 
LIMIT 20;
```

### 最佳实践清单

1. **合理使用数据类型**：选择最合适的数据类型。
2. **添加适当索引**：为常用查询添加索引，避免过多索引。
3. **定期维护**：VACUUM、ANALYZE、REINDEX。
4. **监控慢查询**：使用 pg_stat_statements。
5. **备份策略**：使用 pg_dump +  WAL 归档。
6. **连接池**：使用 PgBouncer 或 PgPool-II。
7. **分区表**：大表使用分区策略。
8. **复制配置**：主从复制保证高可用。
9. **安全配置**：使用 SSL、RLS、最小权限原则。
10. **版本升级**：定期升级到最新稳定版。

---

## 与其他数据库对比

### PostgreSQL vs MySQL

| 特性 | PostgreSQL | MySQL |
|------|------------|-------|
| **SQL 标准** | 完全遵循 | 部分遵循 |
| **事务** | ACID 完全支持 | ACID 支持 |
| **并发控制** | MVCC | MVCC + 锁 |
| **索引类型** | 多种 | B-tree 为主 |
| **JSON 支持** | jsonb（高性能） | JSON（一般） |
| **向量搜索** | pgvector | 8.0+ 原生支持 |
| **全文搜索** | 内置 | 内置 |
| **地理信息** | PostGIS | MySQL Spatial |
| **复制** | 流复制 | 主从/组复制 |
| **分区** | 原生支持 | 原生支持 |

### PostgreSQL vs MongoDB

| 特性 | PostgreSQL | MongoDB |
|------|------------|---------|
| **模型** | 关系型 | 文档型 |
| **事务** | ACID | ACID（副本集/分片） |
| **扩展性** | 垂直扩展 | 水平扩展 |
| **JSON** | jsonb | 原生文档 |
| **JOIN** | 完整支持 | 有限支持 |
| **一致性** | 强一致 | 可调一致性 |
| **查询** | SQL | MQL |

### PostgreSQL vs SQLite

| 特性 | PostgreSQL | SQLite |
|------|------------|--------|
| **类型** | 服务器 | 嵌入式 |
| **并发** | 多连接 | 文件锁 |
| **性能** | 高并发优秀 | 单机优秀 |
| **容量** | 无限制 | TB 级 |
| **部署** | 复杂 | 简单 |
| **场景** | 企业应用 | 嵌入式/移动 |
