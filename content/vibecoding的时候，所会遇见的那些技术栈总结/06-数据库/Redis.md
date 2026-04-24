# Redis 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Redis 8.0 新特性、数据结构详解、持久化机制、Redis Stack（Search/JSON/Graph）、以及在 AI 应用中的向量相似度搜索。

---

## 目录

1. [[#Redis 概述与核心定位]]
2. [[#Redis 8.0 新特性]]
3. [[#数据结构详解]]
4. [[#持久化机制]]
5. [[#Redis Cluster 集群]]
6. [[#Redis 数据结构对比表]]
7. [[#Redis Stack 扩展]]
8. [[#Redis 与 AI]]
9. [[#与 Dragonfly/Memurai 对比]]
10. [[#实战场景与代码示例]]
11. [[#选型建议]]

---

## Redis 概述与核心定位

### 为什么选择 Redis

Redis（REmote DIctionary Server）是一个基于内存的键值存储系统，由 Salvatore Sanfilippo 于 2009 年创建。它以其卓越的性能和丰富的数据结构著称，广泛应用于缓存、消息队列、实时分析等场景。

Redis 的核心优势体现在四个维度：

**极致性能**：基于内存存储，读写速度可达每秒 100 万次以上，延迟低至微秒级。

**丰富数据结构**：支持 String、Hash、List、Set、Sorted Set、Stream 等超过 10 种数据结构。

**丰富特性**：支持发布/订阅、事务、管道、Lua 脚本、模块扩展等高级功能。

**生态完善**：Redis Stack 提供搜索、JSON、图数据库等企业级功能。

### Redis 在 AI 时代的价值

在 AI 应用中，Redis 的价值被重新发现：

- **向量相似度搜索**：Redis Stack 支持向量搜索，用于 AI RAG 系统
- **高速缓存**：缓存 LLM 响应，减少 API 调用成本
- **会话存储**：存储 AI 对话上下文
- **实时队列**：处理 AI 任务的异步队列
- **Rate Limiting**：API 访问频率控制

### 市场份额与行业地位

Redis 在键值存储和缓存市场中占据主导地位：

| 排名 | 数据库 | 市场份额（2025） | 趋势 |
|------|--------|------------------|------|
| 1 | Redis | 58.3% | → |
| 2 | Memcached | 22.1% | ↓ |
| 3 | Dragonfly | 8.5% | ↑ |
| 4 | Hazelcast | 5.2% | → |
| 5 | Aerospike | 3.8% | ↑ |

**关键数据点**：
- Redis 是全球最流行的内存数据存储
- 超过 50% 的 Fortune 500 公司使用 Redis
- 主要使用者包括 Twitter、GitHub、Stack Overflow、Snapchat 等
- Redis Labs 提供企业版和开源版两个版本
- Redis 在 GitHub 上有超过 67,000 颗星

**Redis 适用场景**：

| 场景 | 推荐程度 | 说明 |
|------|----------|------|
| 缓存层 | ⭐⭐⭐⭐⭐ | 高速缓存首选 |
| 会话存储 | ⭐⭐⭐⭐⭐ | 内存存储，快速访问 |
| 消息队列 | ⭐⭐⭐⭐ | Stream 数据结构 |
| 排行榜 | ⭐⭐⭐⭐⭐ | Sorted Set |
| 实时分析 | ⭐⭐⭐⭐ | HyperLogLog、Bitmap |
| AI 缓存/向量 | ⭐⭐⭐⭐ | Redis Stack |

---

## 核心概念与架构

### Redis 架构

Redis 采用单线程模型和事件循环架构：

```
┌─────────────────────────────────────────────────────────────┐
│                       Redis 架构                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   客户端连接                                                   │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │  TCP/IP  │  │  Unix    │  │ TLS/SSL  │               │
│   │  Socket  │  │  Socket  │  │           │               │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘               │
│        │               │               │                       │
│        └───────────────┼───────────────┘                       │
│                        ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              I/O 多路复用（epoll/select）             │   │
│   └─────────────────────────────────────────────────────┘   │
│                        ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  事件循环（Event Loop）                 │   │
│   │  ┌─────────────────────────────────────────────┐   │   │
│   │  │              命令处理器（Command Handler）    │   │   │
│   │  └─────────────────────────────────────────────┘   │   │
│   └─────────────────────────────────────────────────────┘   │
│                        ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                    数据存储层                          │   │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐            │   │
│   │  │ String  │  │  Hash   │  │  List   │            │   │
│   │  ├─────────┤  ├─────────┤  ├─────────┤            │   │
│   │  │  Set    │  │ ZSet    │  │ Stream  │            │   │
│   │  └─────────┘  └─────────┘  └─────────┘            │   │
│   └─────────────────────────────────────────────────────┘   │
│                        ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              持久化层（RDB / AOF / 混合）             │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 核心概念

#### 1. 键空间（Keyspace）

Redis 的键空间是所有键的集合：

```redis
-- 查看所有键（生产环境慎用）
KEYS *

-- 模式匹配
KEYS user:*
KEYS *:session

-- 键操作
EXISTS user:1001        -- 检查键是否存在
TYPE user:1001           -- 查看键类型
RENAME user:1001 user:1002  -- 重命名
RENAMENX user:1001 user:1002  -- 仅当新键不存在时重命名
DEL user:1001           -- 删除键
UNLINK user:1001         -- 异步删除（推荐）
```

#### 2. 过期策略

```redis
-- 设置过期时间
SET session:abc "data" EX 3600        -- 1小时后过期
SET session:abc "data" PX 3600000      -- 毫秒
EXPIRE session:abc 3600                -- 设置过期时间
PEXPIRE session:abc 3600000            -- 毫秒

-- 过期时间操作
TTL session:abc                         -- 查看剩余生存时间（秒）
PTTL session:abc                       -- 毫秒
EXPIREAT session:abc 1705312800        -- 设置过期时间戳
PEXPIREAT session:abc 1705312800000   -- 毫秒时间戳

-- 移除过期时间
PERSIST session:abc

-- 惰性删除和定期删除
-- Redis 采用惰性删除（访问时检查）+ 定期删除（定时扫描）
```

#### 3. 内存管理

```redis
-- 查看内存使用
INFO memory
MEMORY STATS

-- 内存优化
MEMORY DOCTOR        -- 诊断内存问题
MEMORY PURGE         -- 清理内存碎片

-- 设置内存限制
CONFIG SET maxmemory 2gb
CONFIG SET maxmemory-policy allkeys-lru

-- 内存淘汰策略
-- noeviction: 不淘汰（默认）
-- allkeys-lru: 所有键 LRU
-- allkeys-random: 所有键随机
-- volatile-lru: 已设置过期键 LRU
-- volatile-random: 已设置过期键随机
-- volatile-ttl: 已设置过期键 TTL 最小
-- volatile-lfu: 已设置过期键 LFU
-- allkeys-lfu: 所有键 LFU
```

### 数据结构详解

#### 1. String（字符串）

```redis
-- 基本操作
SET user:1001 "Alice"
GET user:1001                    -- "Alice"

-- 过期时间
SET session:abc123 "data" EX 3600  -- 1小时后过期
SETEX cache:key 300 "value"        -- SET + EXPIRE

-- 原子操作
SET counter 100
INCR counter                      -- 101
INCRBY counter 10                 -- 111
DECR counter                      -- 110
INCRBYFLOAT price 0.5             -- 110.5

-- 批量操作
MSET user:1001 "Alice" user:1002 "Bob" user:1003 "Charlie"
MGET user:1001 user:1002 user:1003
-- ["Alice", "Bob", "Charlie"]

-- 字符串操作
SET article:1:title "Redis Tutorial"
APPEND article:1:title " - Advanced"  -- 追加
GETRANGE article:1:title 0 11         -- "Redis Tut"
SETRANGE article:1:title 0 "The "     -- 替换

-- 位操作（用于统计）
SETBIT visited:2024:01:15 1000001 1   -- 设置某位
GETBIT visited:2024:01:15 1000001     -- 获取某位
BITCOUNT visited:2024:01:15            -- 统计 1 的个数
BITOP AND daily:2024:01:15 visited:2024:01:14 visited:2024:01:15
```

#### 2. Hash（哈希）

```redis
-- 基本操作
HSET user:1001 name "Alice" email "alice@example.com" age 30
HGET user:1001 name                   -- "Alice"
HGETALL user:1001
-- {"name":"Alice","email":"alice@example.com","age":"30"}

-- 批量操作
HMSET user:1002 name "Bob" email "bob@example.com"
HMGET user:1001 user:1002 name
-- ["Alice", "Bob"]

-- 字段操作
HEXISTS user:1001 name                -- 1
HDEL user:1001 age                     -- 删除字段
HINCRBY user:1001 login_count 1        -- 原子递增
HLEN user:1001                          -- 字段数量

-- 字段值操作
HSET user:1001 tags "developer,python,redis"
HGET user:1001 tags                    -- "developer,python,redis"

-- 获取所有字段或值
HKEYS user:1001                        -- 所有字段名
HVALS user:1001                         -- 所有值

-- 扫描
HSCAN user:1001 0 MATCH name COUNT 10
```

#### 3. List（列表）

```redis
-- 基本操作（LPUSH/RPOP = 队列，LPUSH/LPOP = 栈）
LPUSH tasks "task1"
LPUSH tasks "task2"
LPUSH tasks "task3"
LRANGE tasks 0 -1                      -- ["task3","task2","task1"]

RPOP tasks                             -- "task1"
LPOP tasks                             -- "task3"

-- 阻塞操作（用于消息队列）
BLPOP tasks 0                         -- 阻塞等待
BRPOP tasks 0

-- 索引操作
LINDEX tasks 0                         -- 获取索引0的元素
LINSERT tasks BEFORE "task2" "task1.5"  -- 插入
LSET tasks 0 "new_first"                -- 设置

-- 范围操作
LTRIM tasks 0 99                       -- 保留0-99
LRange tasks 0 -1                      -- 获取所有

-- 长度
LLEN tasks

-- 批量操作
RPUSH order:1001 "item1" "item2" "item3"
LINSERT order:1001 AFTER "item1" "item1.5"
```

#### 4. Set（集合）

```redis
-- 基本操作
SADD tags:article:1 "redis" "cache" "database"
SMEMBERS tags:article:1               -- 所有成员
SISMEMBER tags:article:1 "redis"      -- 1 (存在)
SREM tags:article:1 "cache"           -- 删除

-- 集合运算
SADD set:a 1 2 3 4 5
SADD set:b 4 5 6 7 8

SUNION set:a set:b                    -- 并集 {1,2,3,4,5,6,7,8}
SINTER set:a set:b                    -- 交集 {4,5}
SDIFF set:a set:b                     -- 差集 {1,2,3}

SUNIONSTORE result set:a set:b        -- 保存结果
SINTERSTORE common set:a set:b
SDIFFSTORE only_a set:a set:b

-- 随机操作
SRANDMEMBER set:a 2                   -- 随机获取2个
SPOP set:a 1                          -- 随机弹出1个

-- 基数（元素数量）
SCARD set:a                           -- 5
```

#### 5. Sorted Set（有序集合）

```redis
-- 基本操作（用于排行榜）
ZADD leaderboard 100 "Alice" 90 "Bob" 80 "Charlie" 95 "David"
ZRANGE leaderboard 0 -1 WITHSCORES
-- ["Charlie", 80, "Bob", 90, "David", 95, "Alice", 100]

ZREVRANGE leaderboard 0 -1 WITHSCORES  -- 倒序

-- 按分数范围查询
ZRANGEBYSCORE leaderboard 90 100      -- [90,100] 之间
ZCOUNT leaderboard 80 95               -- 80-95 之间的人数

-- 排名
ZRANK leaderboard "Alice"              -- 3 (0-indexed)
ZREVRANK leaderboard "Alice"          -- 0 (倒序)

-- 分数操作
ZINCRBY leaderboard 10 "Bob"          -- Bob +10 分

-- 集合运算（加权）
ZADD set:a 1 "x" 2 "y" 3 "z"
ZADD set:b 2 "y" 3 "z" 4 "w"
ZUNIONSTORE union 2 set:a set:b WEIGHTS 1 2
ZUNIONSTORE sum 2 set:a set:b AGGREGATE SUM

-- 交集（共同关注、共同好友）
ZINTERSTORE common_tags 2 tags:user:1 tags:user:2
```

#### 6. Stream（流）

```redis
-- 基本操作（事件源/消息队列）
XADD events:2024 "*" type "click" user "alice" page "/home"
XADD events:2024 "*" type "purchase" product "widget" amount 99.99
-- * 表示自动生成 ID

XLEN events:2024                      -- 2

XRANGE events:2024 - +                -- 所有消息
XRANGE events:2024 1704067200000-0 +  -- 按 ID 范围

-- 消费组
XGROUP CREATE events:2024 consumers "$"
XREADGROUP GROUP consumers "c1" STREAMS events:2024 ">"
XPENDING events:2024 consumers        -- 待确认消息

-- 确认处理
XACK events:2024 consumers 1704067200000-0
```

---

---

## Redis 8.0 新特性

Redis 8.0 于 2025 年发布，带来了多项重大改进。

### 核心新特性

#### 1. 性能大幅提升

| 优化项 | 说明 |
|--------|------|
| **新的内存分配器** | 针对现代硬件优化的分配策略 |
| **I/O 线程改进** | 更高效的异步 I/O |
| **命令管道优化** | 提升批量操作吞吐量 |
| **内存碎片优化** | 自动内存碎片整理 |

#### 2. 增强的向量搜索

```redis
-- 创建向量索引
FT.CREATE articles_index ON HASH PREFIX 1 article: SCHEMA 
    title TEXT WEIGHT 5
    content TEXT
    embedding VECTOR HNSW 6 DIM 1536 TYPE FLOAT32 DISTANCE_METRIC COSINE

-- 向量相似度搜索
FT.SEARCH articles_index "*=>[KNN 10 @embedding $vec]" 
    PARAMS 4 vec "[0.1, 0.2, ...]"

-- 混合搜索
FT.SEARCH articles_index 
    "(@title:AI @content:database)=>[KNN 5 @embedding $vec]"
    PARAMS 4 vec "[query_vector]"
```

#### 3. 新的 ACL 功能

```redis
-- 细粒度权限控制
ACL SETUSER ai_service ON >password ~cached:* ~vector:* 
    &@read &@write &@dangerous ~session:* 
    -@admin -@debug +client|list +client|kill

-- 密钥空间通知
CONFIG SET notify-keyspace-events KEltz
```

#### 4. 改进的 Redis Stack

```redis
-- JSON 路径查询
JSON.SET doc $ '{"name":"Alice","age":30,"scores":[85,90,78]}'
JSON.GET doc $.scores        -- [85,90,78]
JSON.GET doc $..score        -- [85,90,78]

-- 图查询（RedisGraph 整合）
GRAPH.QUERY movies "MATCH (a:Actor)-[:ACTED_IN]->(m:Movie) RETURN a.name, m.title"
```

---

## 数据结构详解

### 1. String（字符串）

```redis
-- 基本操作
SET user:1001 "Alice"
GET user:1001                    -- "Alice"

-- 过期时间
SET session:abc123 "data" EX 3600  -- 1小时后过期
SETEX cache:key 300 "value"        -- SET + EXPIRE

-- 原子操作
SET counter 100
INCR counter                      -- 101
INCRBY counter 10                 -- 111
DECR counter                      -- 110
INCRBYFLOAT price 0.5             -- 110.5

-- 批量操作
MSET user:1001 "Alice" user:1002 "Bob" user:1003 "Charlie"
MGET user:1001 user:1002 user:1003
-- ["Alice", "Bob", "Charlie"]

-- 字符串操作
SET article:1:title "Redis Tutorial"
APPEND article:1:title " - Advanced"  -- 追加
GETRANGE article:1:title 0 11         -- "Redis Tut"
SETRANGE article:1:title 0 "The "     -- 替换

-- 位操作（用于统计）
SETBIT visited:2024:01:15 1000001 1   -- 设置某位
GETBIT visited:2024:01:15 1000001     -- 获取某位
BITCOUNT visited:2024:01:15            -- 统计 1 的个数
BITOP AND daily:2024:01:15 visited:2024:01:14 visited:2024:01:15
```

### 2. Hash（哈希）

```redis
-- 基本操作
HSET user:1001 name "Alice" email "alice@example.com" age 30
HGET user:1001 name                   -- "Alice"
HGETALL user:1001
-- {"name":"Alice","email":"alice@example.com","age":"30"}

-- 批量操作
HMSET user:1002 name "Bob" email "bob@example.com"
HMGET user:1001 user:1002 name
-- ["Alice", "Bob"]

-- 字段操作
HEXISTS user:1001 name                -- 1
HDEL user:1001 age                     -- 删除字段
HINCRBY user:1001 login_count 1        -- 原子递增
HLEN user:1001                          -- 字段数量

-- 字段值操作
HSET user:1001 tags "developer,python,redis"
HGET user:1001 tags                    -- "developer,python,redis"

-- 获取所有字段或值
HKEYS user:1001                        -- 所有字段名
HVALS user:1001                         -- 所有值

-- 扫描
HSCAN user:1001 0 MATCH name COUNT 10
```

### 3. List（列表）

```redis
-- 基本操作（LPUSH/RPOP = 队列，L PUSH/LPOP = 栈）
LPUSH tasks "task1"
LPUSH tasks "task2"
LPUSH tasks "task3"
LRANGE tasks 0 -1                      -- ["task3","task2","task1"]

RPOP tasks                             -- "task1"
LPOP tasks                             -- "task3"

-- 阻塞操作（用于消息队列）
BLPOP tasks 0                         -- 阻塞等待
BRPOP tasks 0

-- 索引操作
LINDEX tasks 0                         -- 获取索引0的元素
LINSERT tasks BEFORE "task2" "task1.5"  -- 插入
LSET tasks 0 "new_first"                -- 设置

-- 范围操作
LTRIM tasks 0 99                       -- 保留0-99
LRange tasks 0 -1                      -- 获取所有

-- 长度
LLEN tasks

-- 批量操作
RPUSH order:1001 "item1" "item2" "item3"
LINSERT order:1001 AFTER "item1" "item1.5"
```

### 4. Set（集合）

```redis
-- 基本操作
SADD tags:article:1 "redis" "cache" "database"
SMEMBERS tags:article:1               -- 所有成员
SISMEMBER tags:article:1 "redis"      -- 1 (存在)
SREM tags:article:1 "cache"           -- 删除

-- 集合运算
SADD set:a 1 2 3 4 5
SADD set:b 4 5 6 7 8

SUNION set:a set:b                    -- 并集 {1,2,3,4,5,6,7,8}
SINTER set:a set:b                    -- 交集 {4,5}
SDIFF set:a set:b                     -- 差集 {1,2,3}

SUNIONSTORE result set:a set:b        -- 保存结果
SINTERSTORE common set:a set:b
SDIFFSTORE only_a set:a set:b

-- 随机操作
SRANDMEMBER set:a 2                   -- 随机获取2个
SPOP set:a 1                          -- 随机弹出1个

-- 基数（元素数量）
SCARD set:a                           -- 5
```

### 5. Sorted Set（有序集合）

```redis
-- 基本操作（用于排行榜）
ZADD leaderboard 100 "Alice" 90 "Bob" 80 "Charlie" 95 "David"
ZRANGE leaderboard 0 -1 WITHSCORES
-- ["Charlie", 80, "Bob", 90, "David", 95, "Alice", 100]

ZREVRANGE leaderboard 0 -1 WITHSCORES  -- 倒序

-- 按分数范围查询
ZRANGEBYSCORE leaderboard 90 100      -- [90,100] 之间
ZCOUNT leaderboard 80 95               -- 80-95 之间的人数

-- 排名
ZRANK leaderboard "Alice"              -- 3 (0-indexed)
ZREVRANK leaderboard "Alice"           -- 0 (倒序)

-- 分数操作
ZINCRBY leaderboard 10 "Bob"          -- Bob +10 分

-- 集合运算（加权）
ZADD set:a 1 "x" 2 "y" 3 "z"
ZADD set:b 2 "y" 3 "z" 4 "w"
ZUNIONSTORE union 2 set:a set:b WEIGHTS 1 2
ZUNIONSTORE sum 2 set:a set:b AGGREGATE SUM

-- 交集（共同关注、共同好友）
ZINTERSTORE common_tags 2 tags:user:1 tags:user:2
```

### 6. Stream（流）

```redis
-- 基本操作（事件源/消息队列）
XADD events:2024 "*" type "click" user "alice" page "/home"
XADD events:2024 "*" type "purchase" product "widget" amount 99.99
-- * 表示自动生成 ID

XLEN events:2024                      -- 2

XRANGE events:2024 - +                -- 所有消息
XRANGE events:2024 1704067200000-0 +  -- 按 ID 范围

-- 消费组
XGROUP CREATE events:2024 consumers "$"
XREADGROUP GROUP consumers "c1" STREAMS events:2024 ">"
XPENDING events:2024 consumers        -- 待确认消息

-- 确认处理
XACK events:2024 consumers 1704067200000-0
```

---

## 持久化机制

### RDB vs AOF 对比表

| 特性 | RDB | AOF |
|------|-----|-----|
| **存储方式** | 内存快照 | 命令日志 |
| **文件大小** | 紧凑 | 较大 |
| **恢复速度** | 快（一行命令） | 慢（逐条重放） |
| **数据安全性** | 可能丢失数据 | 可配置（always/everysec/no） |
| **对性能影响** | 较大（fork） | 较小 |
| **压缩** | 支持（rdbcompression） | 支持（重写压缩） |
| **适用场景** | 备份、恢复 | 数据安全 |
| **配置项** | save 900 1 | appendonly yes |
| **重写机制** | - | auto-aof-rewrite-percentage |

### 配置与操作

```redis
# RDB 配置
save 900 1        # 900秒内1次修改触发
save 300 10       # 300秒内10次修改触发
save 60 10000      # 60秒内10000次修改触发
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb

# AOF 配置
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec   # always/everysec/no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 手动操作
BGSAVE                    # 后台保存
SAVE                      # 同步保存（阻塞）
LASTSAVE                  -- 获取上次保存时间
BGREWRITEAOF              # 重写 AOF
```

### 混合持久化（Redis 4.0+）

```redis
# 启用混合持久化
aof-use-rdb-preamble yes

# 效果：
# 1. AOF 重写时，先写入 RDB 格式，再追加增量 AOF
# 2. 恢复时优先加载 RDB，再重放增量 AOF
# 3. 兼顾恢复速度和数据完整性
```

### Redis 7.0+ 多线程 AOF

```redis
# Redis 7.0 引入多线程 AOF 重写
# 配置
appendonly yes
aof-rewrite-internal-fn threading yes
aof-timestamp-enabled yes
```

---

## Redis Cluster 集群

### 集群架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Redis Cluster                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                   │
│  │Master 1 │   │Master 2 │   │Master 3 │                   │
│  │ P:7000  │   │ P:7001  │   │ P:7002  │                   │
│  └────┬────┘   └────┬────┘   └────┬────┘                   │
│       │            │            │                         │
│  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐                    │
│  │Replica1 │  │Replica2 │  │Replica3 │                    │
│  │ P:8000  │  │ P:8001  │  │ P:8002  │                    │
│  └─────────┘  └─────────┘  └─────────┘                    │
│                                                             │
│  Slot 0-5460    Slot 5461-10922  Slot 10923-16383          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 集群配置

```bash
# 启动集群节点
redis-server --port 7000 --cluster-enabled yes --cluster-config-file nodes-7000.conf
redis-server --port 7001 --cluster-enabled yes --cluster-config-file nodes-7001.conf
redis-server --port 7002 --cluster-enabled yes --cluster-config-file nodes-7002.conf

# 创建集群（至少3个主节点）
redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 \
  --cluster-replicas 1  # 每个主节点1个副本
```

### 集群操作

```redis
# 查看集群信息
CLUSTER INFO
CLUSTER NODES
CLUSTER SLOTS

# 故障转移
CLUSTER FAILOVER          -- 从节点执行，手动故障转移

# 槽迁移
CLUSTER ADDSLOTS 0 1 2 3  -- 添加槽
CLUSTER DELSLOTS 0 1 2 3  -- 删除槽
CLUSTER SETSLOT 0 IMPORTING <source_node_id>  -- 迁移中
CLUSTER SETSLOT 0 MIGRATING <target_node_id>  -- 迁移中

# 手动重新分片
redis-cli --cluster reshard 127.0.0.1:7000

# 故障检测
CLUSTER MEET 127.0.0.1 7003  -- 节点加入集群
CLUSTER FORGET <node_id>     -- 移除节点
```

### 客户端路由

```python
# Python 客户端（redis-py-cluster）
from redis.cluster import RedisCluster

rc = RedisCluster(
    host='127.0.0.1',
    port=7000,
    decode_responses=True
)

# 自动路由
rc.set('key1', 'value1')  # 自动路由到正确节点
rc.get('key1')

# 跨节点操作（需要 ALL NODES flag）
rc.pipeline(
    commands=[('get', 'key1'), ('get', 'key2')],
    channel_map=rc.nodes  # 指定所有节点
)
```

---

## Redis 数据结构对比表

| 数据结构 | 底层实现 | 时间复杂度 | 适用场景 |
|---------|---------|-----------|---------|
| **String** | SDS (Simple Dynamic String) | O(1) | 缓存、计数器、分布式锁 |
| **Hash** | Dict + ZipList/HT | O(1) | 对象存储、购物车 |
| **List** | QuickList/ZipList | O(1) | 队列、消息队列、时间线 |
| **Set** | HT/IntSet | O(1) | 标签、去重、共同好友 |
| **Sorted Set** | SkipList + HT | O(log N) | 排行榜、优先级队列 |
| **Stream** | RadixTree + Consumer Groups | O(1) | 事件流、消息队列 |
| **Bitmap** | String (bit操作) | O(1) | 签到、用户在线统计 |
| **HyperLogLog** | PFADD/PFCOUNT | O(1) | UV 统计 |
| **Geospatial** | GeoHash + Sorted Set | O(log N) | 附近的人/商家 |
| **JSON** | RedisJSON (VSS) | - | 文档存储 |

### 持久化策略对比表

| 策略 | 优点 | 缺点 | 适用场景 |
|------|------|------|---------|
| **RDB** | 恢复快、文件紧凑 | 可能丢数据、fork 开销 | 备份、恢复 |
| **AOF everysec** | 最多1秒数据丢失 | 文件较大 | 平衡场景 |
| **AOF always** | 零数据丢失 | 性能影响较大 | 数据安全优先 |
| **混合持久化** | 恢复快 + 数据安全 | 配置复杂 | 生产环境推荐 |
| **无持久化** | 性能最高 | 数据丢失 | 纯缓存场景 |

---

## Redis Stack 扩展

### Redis Search（全文搜索）

```redis
-- 创建索引
FT.CREATE products SCHEMA 
    name TEXT WEIGHT 5 
    description TEXT
    price NUMERIC SORTABLE
    brand TAG
    category TAG

-- 添加文档
HSET product:1 name "iPhone 15" description "Apple smartphone" price 999 brand "Apple" category "Electronics"
HSET product:2 name "Galaxy S24" description "Samsung smartphone" price 899 brand "Samsung" category "Electronics"

-- 搜索查询
FT.SEARCH products "iPhone" RETURN 3 name description price

-- 高级查询
FT.SEARCH products "@price:[500 1000] @brand:{Apple|Samsung}" RETURN 3 name price brand

-- 聚合
FT.AGGREGATE products "*" GROUPBY 1 @category REDUCE AVG 1 @price AS avg_price
```

### Redis JSON

```redis
-- JSON 操作
JSON.SET doc:1 $ '{"name":"Alice","age":30,"address":{"city":"NYC"}}'
JSON.GET doc:1                     -- 获取整个文档
JSON.GET doc:1 $.name               -- "Alice"
JSON.GET doc:1 $.address.city        -- "NYC"

-- 数值操作
JSON.NUMINCRBY doc:1 $.age 1        -- age +1
JSON.ARRAPPEND doc:1 $.skills "Python" "Redis"

-- 数组操作
JSON.SET doc:2 $ '{"scores":[80,90,75]}'
JSON.ARRAPPEND doc:2 $.scores 85
JSON.ARRINDEX doc:2 $.scores 90

-- 路径操作
JSON.DEL doc:1 $.address            -- 删除字段
JSON.TYPE doc:1 $.name              -- string
JSON.NUMSTR doc:1 $.age             -- 30
```

### Redis Graph（图数据库）

```redis
-- 创建图
GRAPH.QUERY movies "CREATE (a:Actor {name:'Tom Hanks'}), (m:Movie {title:'Forrest Gump'}), (a)-[:ACTED_IN]->(m)"

-- 查询
GRAPH.QUERY movies """
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
WHERE m.title CONTAINS 'Gump'
RETURN a.name, m.title
"""

-- 关系查询
GRAPH.QUERY social """
MATCH (me:Person {name:'Alice'})-[:FOLLOWS*1..2]->(f)
RETURN f.name, TYPE(f) AS relationship
"""

-- 聚合查询
GRAPH.QUERY movies """
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie)
RETURN a.name, COUNT(*) AS movie_count
ORDER BY movie_count DESC
LIMIT 10
"""
```

---

## Redis 与 AI

### 向量相似度搜索

```redis
-- 启用 RediSearch 向量功能
# 配置文件添加
loadmodule /path/to/redisai/redisml.so
loadmodule /path/to/redis-stack/lib/redis_stack.so

-- 创建向量索引（HNSW 算法）
FT.CREATE doc_index SCHEMA 
    content TEXT 
    embedding VECTOR HNSW 6 DIM 1536 TYPE FLOAT32 DISTANCE_METRIC COSINE

-- 插入文档
HSET doc:1 content "PostgreSQL is great" embedding "<vector_data>"
HSET doc:2 content "MongoDB is flexible" embedding "<vector_data>"
HSET doc:3 content "Redis is fast" embedding "<vector_data>"

-- 向量搜索
FT.SEARCH doc_index "*=>[KNN 5 @embedding $vec]" PARAMS 4 vec "<query_vector>"

-- 带过滤的搜索
FT.SEARCH doc_index "@category:{database}=>[KNN 5 @embedding $vec]" PARAMS 4 vec "<query_vector>"
```

### Python 集成

```python
import redis
import numpy as np

# 连接 Redis Stack
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

def generate_embedding(text: str) -> list:
    """生成嵌入向量（使用 OpenAI 或本地模型）"""
    # 简化示例：生成随机向量
    return np.random.rand(1536).tolist()

def index_document(doc_id: str, content: str, metadata: dict = None):
    """索引文档"""
    embedding = generate_embedding(content)
    pipe = r.pipeline()
    
    # 存储原始数据
    pipe.hset(f"doc:{doc_id}", mapping={
        "content": content,
        "embedding": np.array(embedding).astype(np.float32).tobytes(),
        "metadata": str(metadata or {})
    })
    
    # 添加到搜索索引
    pipe.execute()

def search_similar(query: str, top_k: int = 5, category: str = None):
    """语义搜索"""
    query_embedding = generate_embedding(query)
    
    if category:
        search_query = f"@category:{{{category}}}=>[KNN {top_k} @embedding $vec]"
    else:
        search_query = f"*=>[KNN {top_k} @embedding $vec]"
    
    results = r.ft("doc_index").search(
        search_query,
        params={"vec": np.array(query_embedding).astype(np.float32).tobytes()},
        return_fields=["content", "metadata"]
    )
    
    return [
        {
            "id": hit.id,
            "content": hit.content,
            "score": hit.score
        }
        for hit in results.docs
    ]
```

### AI 缓存策略

```python
def cache_llm_response(prompt_hash: str, response: str, ttl: int = 3600):
    """缓存 LLM 响应"""
    r.setex(f"llm:response:{prompt_hash}", ttl, response)

def get_cached_response(prompt_hash: str) -> str | None:
    """获取缓存的 LLM 响应"""
    return r.get(f"llm:response:{prompt_hash}")

def track_token_usage(user_id: str, tokens: int):
    """追踪 Token 使用"""
    pipe = r.pipeline()
    pipe.incrby(f"tokens:{user_id}:daily:{date.today()}", tokens)
    pipe.expire(f"tokens:{user_id}:daily:{date.today()}", 86400 * 2)
    pipe.zincrby(f"tokens:{user_id}:monthly:{date.today().strftime('%Y-%m')}", tokens, date.today().isoformat())
    pipe.execute()

def rate_limit(user_id: str, limit: int = 60, window: int = 60) -> bool:
    """API 限流（滑动窗口）"""
    key = f"ratelimit:{user_id}"
    now = time.time()
    
    pipe = r.pipeline()
    pipe.zremrangebyscore(key, 0, now - window)
    pipe.zadd(key, {str(now): now})
    pipe.zcard(key)
    pipe.expire(key, window)
    results = pipe.execute()
    
    return results[2] <= limit
```

---

## 与 Dragonfly/Memurai 对比

### 核心对比表

| 特性 | Redis | Dragonfly | Memurai |
|------|-------|-----------|---------|
| **许可证** | SSPL / 商业 | Apache 2.0 | 商业/开发免费 |
| **内存效率** | 一般 | 高（节省 30%+） | 一般 |
| **多核扩展** | 有限 | 原生支持 | 有限 |
| **持久化** | RDB/AOF | RDB/AOF | RDB/AOF |
| **数据结构** | 全部 | 全部 | 全部 |
| **Redis 协议** | 完全兼容 | 兼容 | 完全兼容 |
| **集群支持** | 原生 | 原生 | 不支持 |
| **企业功能** | Redis Stack | Dragonfly Cloud | 商业支持 |
| **社区活跃度** | 高 | 快速增长 | 一般 |
| **AI/向量搜索** | Redis Stack | 发展中 | 不支持 |

### 选型建议

| 场景 | 推荐选择 | 原因 |
|------|----------|------|
| **企业生产环境** | Redis | 成熟稳定 |
| **单节点高性能** | Dragonfly | 多核优化 |
| **Windows 环境** | Memurai | 原生支持 |
| **AI 向量搜索** | Redis Stack | 完整支持 |
| **开源项目** | Redis/Dargonfly | 无许可证问题 |
| **技术支援需求** | Memurai/Redis | 商业支持 |

---

## 实战场景与代码示例

### 场景一：构建 LLM 缓存层

```python
import hashlib
import json
import redis
from datetime import timedelta

class LLMCache:
    """LLM 响应缓存"""
    
    def __init__(self, redis_client: redis.Redis):
        self.r = redis_client
    
    def _hash_prompt(self, model: str, messages: list) -> str:
        """生成 prompt 哈希"""
        content = json.dumps({
            "model": model,
            "messages": messages
        }, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()
    
    def get(self, model: str, messages: list) -> str | None:
        """获取缓存响应"""
        key = self._hash_prompt(model, messages)
        return self.r.get(f"llm:cache:{key}")
    
    def set(
        self, 
        model: str, 
        messages: list, 
        response: str,
        ttl: timedelta = timedelta(hours=24)
    ):
        """设置缓存"""
        key = self._hash_prompt(model, messages)
        self.r.setex(f"llm:cache:{key}", ttl, response)
    
    def invalidate_model(self, model: str):
        """失效模型所有缓存"""
        pattern = f"llm:cache:*"
        cursor = 0
        while True:
            cursor, keys = self.r.scan(cursor, match=pattern, count=100)
            model_keys = []
            for key in keys:
                # 需要验证模型匹配
                # 简化：直接删除
                model_keys.append(key)
            if model_keys:
                self.r.delete(*model_keys)
            if cursor == 0:
                break

# 使用示例
cache = LLMCache(redis.Redis(host='localhost', port=6379))

# 检查缓存
cached = cache.get("gpt-4o", [{"role": "user", "content": "Hello"}])
if cached:
    print("Cache hit:", cached)
else:
    # 调用 LLM
    response = call_openai("gpt-4o", [{"role": "user", "content": "Hello"}])
    cache.set("gpt-4o", [{"role": "user", "content": "Hello"}], response)
```

### 场景二：异步任务队列

```python
import redis
import json
import uuid
from datetime import datetime

class AIJobQueue:
    """AI 任务队列"""
    
    def __init__(self, redis_client: redis.Redis):
        self.r = redis_client
        self.queue_name = "ai:jobs:pending"
        self.processing = "ai:jobs:processing"
        self.completed = "ai:jobs:completed"
    
    def enqueue(self, job_type: str, params: dict, priority: int = 0) -> str:
        """入队任务"""
        job_id = str(uuid.uuid4())
        job = {
            "id": job_id,
            "type": job_type,
            "params": params,
            "created_at": datetime.utcnow().isoformat(),
            "status": "pending"
        }
        
        # 使用有序集合存储（优先级队列）
        self.r.zadd(
            self.queue_name, 
            {json.dumps(job): priority}
        )
        
        return job_id
    
    def dequeue(self, worker_id: str, timeout: int = 0) -> dict | None:
        """获取任务（阻塞）"""
        result = self.r.zpopmin(self.queue_name, 1)
        
        if not result:
            return None
        
        job_json, score = result[0]
        job = json.loads(job_json)
        
        # 移到处理中
        job["worker_id"] = worker_id
        job["started_at"] = datetime.utcnow().isoformat()
        job["status"] = "processing"
        
        self.r.hset(
            self.processing, 
            job["id"], 
            json.dumps(job)
        )
        
        return job
    
    def complete(self, job_id: str, result: dict):
        """标记任务完成"""
        job = self.r.hget(self.processing, job_id)
        if job:
            job = json.loads(job)
            job["status"] = "completed"
            job["completed_at"] = datetime.utcnow().isoformat()
            job["result"] = result
            
            self.r.hset(self.completed, job_id, json.dumps(job))
            self.r.hdel(self.processing, job_id)
            
            # 保留结果 24 小时
            self.r.expire(self.completed, 86400)
    
    def fail(self, job_id: str, error: str):
        """标记任务失败"""
        job = self.r.hget(self.processing, job_id)
        if job:
            job = json.loads(job)
            job["status"] = "failed"
            job["error"] = error
            
            self.r.hset(self.processing, job_id, json.dumps(job))

# 使用示例
queue = AIJobQueue(redis.Redis(host='localhost', port=6379))

# 任务入队（优先级队列）
queue.enqueue("summarize", {"text": "...", "model": "gpt-4o"}, priority=10)
queue.enqueue("translate", {"text": "...", "lang": "zh"}, priority=5)

# 工作者获取任务
job = queue.dequeue("worker-1")
if job:
    result = process_job(job)
    queue.complete(job["id"], result)
```

---

## 选型建议

### 使用场景分类

| 场景 | 推荐数据结构 | 示例 |
|------|-------------|------|
| **会话缓存** | String/Hash | 用户登录状态 |
| **排行榜** | Sorted Set | 游戏积分榜 |
| **实时计数** | String | UV 统计 |
| **消息队列** | List/Stream | 异步任务 |
| **分布式锁** | String (SET NX) | 资源竞争 |
| **限流** | String/Sorted Set | API 限流 |
| **发布订阅** | Pub/Sub | 实时通知 |
| **AI 语义搜索** | Vector (HNSW) | RAG 系统 |
| **配置缓存** | Hash | 应用配置 |
| **最近浏览** | List | 浏览历史 |

### 配置优化建议

```bash
# redis.conf 关键配置

# 内存配置
maxmemory 4gb                    # 最大内存
maxmemory-policy allkeys-lru     # 内存不足时删除最近最少使用

# 网络配置
bind 127.0.0.1
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300

# 持久化配置
save 900 1 save 300 10 save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes

# AOF 配置
appendonly yes
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 客户端配置
maxclients 10000
client-output-buffer-limit normal 256mb 64mb 60
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
```

---

> [!TIP]
> 对于 AI 应用，Redis Stack 的向量搜索功能可以替代专门的向量数据库（如 Pinecone、Milvus），简化架构。但需要注意向量维度限制（当前最大 32768）。

---

> [!SUCCESS]
> 本文档全面介绍了 Redis 的核心特性、8.0 新功能、6 种核心数据结构、持久化机制、Redis Cluster 集群、Redis Stack 扩展，以及在 AI 应用中的向量搜索和缓存策略。Redis 凭借其卓越的性能和丰富的数据结构，是 AI 应用中不可或缺的基础设施。

---

## 完整安装与环境配置

### 安装方法

```bash
# macOS
brew install redis
brew services start redis

# Ubuntu/Debian
sudo apt update
sudo apt install redis-server

# Docker
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:7-alpine

# Docker Compose
version: "3.8"
services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  redis_data:
```

### Redis CLI

```bash
# 连接
redis-cli
redis-cli -h localhost -p 6379
redis-cli --raw  # 原始输出

# 基本命令
PING  # 测试连接
ECHO "hello"  # 回显
INFO  # 服务器信息
DBSIZE  # 键数量
KEYS *  # 列出所有键（生产环境慎用）
FLUSHDB  # 清空当前数据库
FLUSHALL  # 清空所有数据库

# 数据库操作
SELECT 0  # 切换数据库
MOVE key 1  # 移动键到其他数据库

# 键操作
EXISTS key  # 检查键是否存在
TYPE key  # 获取键类型
DEL key  # 删除键
RENAME key newkey  # 重命名
EXPIRE key 60  # 设置过期时间（秒）
TTL key  # 查看剩余生存时间
PTTL key  # 毫秒级 TTL
PERSIST key  # 移除过期时间

# 监控
MONITOR  # 实时监控所有命令
SLOWLOG GET 10  # 获取最近的慢查询
INFO stats  # 查看统计信息
```

### 配置文件

```ini
# /etc/redis/redis.conf

# 基本配置
daemonize no
port 6379
bind 127.0.0.1
timeout 0

# 内存配置
maxmemory 2gb
maxmemory-policy allkeys-lru
maxmemory-samples 5

# 持久化配置
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes

# AOF 配置
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# 复制配置
replica-read-only yes
repl-diskless-sync yes

# 安全配置
requirepass yourpassword
protected-mode yes

# 日志
loglevel notice
logfile ""
```

---

## 数据结构详解

### String（字符串）

```bash
# 基本操作
SET key value
GET key
SETEX key 60 value  # 设置值并指定过期时间（秒）
SETNX key value  # 仅在键不存在时设置
SETRANGE key offset value  # 从指定位置覆盖
GETRANGE key start end  # 获取子字符串
STRLEN key  # 获取字符串长度

# 数值操作
SET counter 0
INCR counter  # 原子递增
INCRBY counter 5  # 增加指定值
INCRBYFLOAT counter 0.5  # 浮点数递增
DECR counter  # 原子递减
DECRBY counter 3  # 减少指定值

# 批量操作
MSET key1 value1 key2 value2
MGET key1 key2 key3
MSETNX key1 value1 key2 value2  # 批量 SETNX

# 位操作
SETBIT key offset value  # 设置位
GETBIT key offset  # 获取位
BITCOUNT key  # 统计 1 的个数
BITOP AND result key1 key2  # 位运算
BITPOS key bit [start] [end]  # 查找第一个指定值的位
```

### Hash（哈希）

```bash
# 基本操作
HSET user:1 name "John" email "john@example.com"
HGET user:1 name
HMGET user:1 name email
HGETALL user:1
HDEL user:1 email
HEXISTS user:1 name

# 批量操作
HMSET user:1 name "John" age 30  # HSET 的批量版本
HMGET user:1 name email age

# 字段操作
HINCRBY user:1 age 1
HINCRBYFLOAT user:1 score 0.5
HLEN user:1  # 获取字段数量
HKEYS user:1  # 获取所有字段名
HVALS user:1  # 获取所有字段值

# 扫描操作
HSCAN user:1 0 MATCH name COUNT 10
```

### List（列表）

```bash
# 基本操作
LPUSH mylist "world"  # 左推入
RPUSH mylist "hello"  # 右推入
LPOP mylist  # 左弹出
RPOP mylist  # 右弹出
LRANGE mylist 0 -1  # 获取所有元素
LINDEX mylist 0  # 获取指定位置元素
LINSERT mylist BEFORE "world" "new"
LINSERT mylist AFTER "hello" "there"

# 批量操作
RPUSH mylist "a" "b" "c"
LPUSH mylist "x" "y" "z"

# 长度和修剪
LLEN mylist  # 获取列表长度
LTRIM mylist 0 9  # 修剪列表（保留指定范围）
LSET mylist 0 "first"  # 设置指定位置元素

# 阻塞操作
BLPOP mylist 0  # 阻塞左弹出（等待有元素）
BRPOP mylist 0  # 阻塞右弹出
BRPOPLPUSH source destination 0  # 阻塞右弹出并推入目标列表
```

### Set（集合）

```bash
# 基本操作
SADD myset "member1" "member2" "member3"
SREM myset "member2"
SMEMBERS myset  # 获取所有成员
SISMEMBER myset "member1"  # 检查是否存在
SCARD myset  # 获取成员数量

# 集合运算
SINTER set1 set2  # 交集
SINTERSTORE result set1 set2  # 交集并存储
SUNION set1 set2  # 并集
SUNIONSTORE result set1 set2  # 并集并存储
SDIFF set1 set2  # 差集（set1 独有）
SDIFFSTORE result set1 set2  # 差集并存储

# 随机操作
SRANDMEMBER myset  # 随机获取一个成员
SRANDMEMBER myset 3  # 随机获取多个成员
SPOP myset  # 随机弹出一个成员

# 扫描操作
SSCAN myset 0 MATCH pattern COUNT 10
```

### Sorted Set（有序集合）

```bash
# 基本操作
ZADD leaderboard 100 "player1"  # 添加带分数的成员
ZADD leaderboard 200 "player2"
ZADD leaderboard 150 "player3"
ZSCORE leaderboard "player1"  # 获取成员分数
ZRANK leaderboard "player1"  # 获取排名（从小到大）
ZREVRANK leaderboard "player1"  # 获取排名（从大到小）
ZINCRBY leaderboard 50 "player1"  # 增加分数

# 范围查询
ZRANGE leaderboard 0 9  # 获取排名 0-9 的成员
ZREVRANGE leaderboard 0 9 WITHSCORES  # 获取前 10 名（带分数）
ZRANGEBYSCORE leaderboard 100 200  # 按分数范围查询
ZCOUNT leaderboard 100 200  # 统计分数范围内的成员数量

# 删除操作
ZREM leaderboard "player3"  # 删除成员
ZREMRANGEBYRANK leaderboard 0 9  # 按排名删除
ZREMRANGEBYSCORE leaderboard 0 100  # 按分数删除

# 有序集合操作
ZUNIONSTORE result 2 set1 set2 WEIGHTS 1 2 AGGREGATE SUM
ZINTERSTORE result 2 set1 set2 AGGREGATE SUM
```

---

## 高级特性

### Pub/Sub 发布订阅

```bash
# 订阅频道
SUBSCRIBE channel1 channel2
PSUBSCRIBE pattern*  # 模式订阅

# 发布消息
PUBLISH channel1 "Hello World"

# 查看订阅信息
PUBSUB CHANNELS  # 查看活跃频道
PUBSUB NUMSUB channel1  # 查看订阅者数量
```

### Stream 流

```bash
# 创建流
XADD mystream * field1 "value1" field2 "value2"
XADD mystream * field1 "value3" field2 "value4"

# 读取流
XRANGE mystream - +  # 读取所有消息
XREAD STREAMS mystream 0  # 阻塞读取新消息
XREAD COUNT 10 STREAMS mystream $  # 从最新消息开始读取

# 消费者组
XGROUP CREATE mystream mygroup 0
XREADGROUP GROUP mygroup consumer1 COUNT 10 STREAMS mystream ">"

# 确认消息
XACK mystream mygroup message-id
```

### Lua 脚本

```bash
# 执行 Lua 脚本
EVAL "return redis.call('GET', KEYS[1])" 1 mykey

# 脚本示例：限流
EVAL "
  local key = KEYS[1]
  local limit = tonumber(ARGV[1])
  local window = tonumber(ARGV[2])
  
  local current = tonumber(redis.call('GET', key) or 0)
  if current >= limit then
    return 0
  end
  
  current = redis.call('INCR', key)
  if current == 1 then
    redis.call('EXPIRE', key, window)
  end
  
  return current
" 1 ratelimit:user:1 10 60
```

---

## 备份与恢复

### 备份方法对比

| 备份类型 | 工具 | 特点 | 适用场景 |
|---------|------|------|----------|
| **RDB** | bgrewriteaof/bgsave | 紧凑、快速 | 定期备份 |
| **AOF** | 持久化日志 | 数据安全 | 生产环境 |
| **混合** | RDB+AOF | 平衡 | 推荐配置 |

### RDB 备份

```bash
# 手动保存
redis-cli SAVE              # 同步保存（阻塞）
redis-cli BGSAVE           # 后台保存（非阻塞）

# 查看保存状态
redis-cli LASTSAVE          # 上次保存时间戳

# 查看子进程保存进度
redis-cli INFO persistence

# 配置文件设置
# save 900 1   # 900秒内1次修改触发
# save 300 10  # 300秒内10次修改触发
# save 60 10000  # 60秒内10000次修改触发
```

### AOF 备份

```bash
# 查看 AOF 状态
redis-cli INFO persistence

# 手动重写 AOF
redis-cli BGREWRITEAOF

# 配置文件设置 AOF
# appendonly yes
# appendfilename "appendonly.aof"
# appendfsync everysec  # always/everysec/no

# AOF 重写配置
# auto-aof-rewrite-percentage 100
# auto-aof-rewrite-min-size 64mb
```

### 恢复操作

```bash
# 停止 Redis
redis-cli SHUTDOWN NOSAVE  # 不保存
redis-cli SHUTDOWN SAVE    # 保存后关闭

# 从 RDB 文件恢复
# 只需将 dump.rdb 放到正确位置重启即可
cp /backup/dump.rdb /var/lib/redis/dump.rdb
systemctl restart redis

# 从 AOF 文件恢复
# 确保 appendonly yes
cp /backup/appendonly.aof /var/lib/redis/appendonly.aof
systemctl restart redis

# 修复损坏的 AOF
redis-check-aof --fix /var/lib/redis/appendonly.aof

# 修复损坏的 RDB
redis-check-rdb /var/lib/redis/dump.rdb
```

### 备份脚本

```bash
#!/bin/bash
# backup.sh - Redis 备份脚本

set -e

BACKUP_DIR="/var/backups/redis"
DATE=$(date +%Y%m%d_%H%M%S)
REDIS_HOST="localhost"
REDIS_PORT="6379"
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

# 触发 RDB 保存
echo "Triggering RDB save..."
redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" BGSAVE

# 等待保存完成
while [ $(redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" LASTSAVE) == $(redis-cli -h "$REDIS_HOST" -p "$REDIS_PORT" LASTSAVE) ]; do
    sleep 1
done

# 复制备份文件
cp /var/lib/redis/dump.rdb "$BACKUP_DIR/dump_$DATE.rdb"
gzip "$BACKUP_DIR/dump_$DATE.rdb"

# 清理旧备份
find "$BACKUP_DIR" -name "*.rdb.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: dump_$DATE.rdb.gz"
```

---

## Python 集成

### redis-py 使用指南

```python
import redis
from redis import ConnectionPool, Redis
import json

# 基本连接
r = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    password=None,
    decode_responses=True
)

# 连接池
pool = ConnectionPool(
    host='localhost',
    port=6379,
    db=0,
    max_connections=20,
    decode_responses=True
)
r = Redis(connection_pool=pool)

# String 操作
r.set('user:1001', 'Alice')
r.get('user:1001')
r.setex('session:abc', 3600, 'data')
r.setnx('user:1002', 'Bob')  # 仅当键不存在时设置

# 计数器
r.set('counter', 100)
r.incr('counter')           # 101
r.incrby('counter', 10)     # 111
r.decr('counter')           # 110
r.incrbyfloat('price', 0.5)

# Hash 操作
r.hset('user:1001', mapping={
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': '30'
})
r.hget('user:1001', 'name')
r.hgetall('user:1001')
r.hincrby('user:1001', 'age', 1)
r.hkeys('user:1001')
r.hvals('user:1001')

# List 操作
r.lpush('tasks', 'task1', 'task2')
r.rpush('tasks', 'task3')
r.lrange('tasks', 0, -1)
r.lpop('tasks')
r.rpop('tasks')
r.llen('tasks')

# Set 操作
r.sadd('tags', 'redis', 'cache', 'database')
r.smembers('tags')
r.sismember('tags', 'redis')
r.sinter('tags', 'tags2')
r.sunion('tags', 'tags2')

# Sorted Set 操作
r.zadd('leaderboard', {'Alice': 100, 'Bob': 90, 'Charlie': 80})
r.zrange('leaderboard', 0, -1, withscores=True)
r.zrevrange('leaderboard', 0, 9, withscores=True)
r.zrank('leaderboard', 'Alice')  # 排名
r.zincrby('leaderboard', 10, 'Bob')

# 发布订阅
pubsub = r.pubsub()
pubsub.subscribe('channel1', 'channel2')
pubsub.psubscribe('pattern*')

for message in pubsub.listen():
    print(message['channel'], message['data'])

# 管道
pipe = r.pipeline()
pipe.set('key1', 'value1')
pipe.get('key1')
pipe.hset('user:1001', 'name', 'Alice')
pipe.execute()

# Lua 脚本
script = """
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local current = tonumber(redis.call('GET', key) or 0)
if current >= limit then
    return 0
end
current = redis.call('INCR', key)
if current == 1 then
    redis.call('EXPIRE', key, ARGV[2])
end
return current
"""
r.register_script(script)
result = r.evalsha(sha1, 1, 'ratelimit:user:1', 10, 60)
```

### 异步客户端

```python
import asyncio
import aioredis

async def main():
    # 连接
    redis = await aioredis.create_redis_pool('redis://localhost:6379')
    
    # String 操作
    await redis.set('key', 'value')
    value = await redis.get('key')
    
    # Hash 操作
    await redis.hset('user:1001', mapping={'name': 'Alice', 'age': 30})
    name = await redis.hget('user:1001', 'name')
    
    # 管道
    pipe = redis.pipeline()
    pipe.set('key1', 'value1')
    pipe.get('key1')
    await pipe.execute()
    
    # 关闭
    redis.close()
    await redis.wait_closed()

asyncio.run(main())
```

---

## Node.js 集成

### ioredis 使用指南

```javascript
const Redis = require('ioredis');

// 基本连接
const redis = new Redis({
  host: 'localhost',
  port: 6379,
  password: null,
  db: 0,
});

// 连接池
const redis = new Redis.Cluster([
  { host: '127.0.0.1', port: 7000 },
  { host: '127.0.0.1', port: 7001 },
  { host: '127.0.0.1', port: 7002 },
]);

// String 操作
await redis.set('user:1001', 'Alice');
await redis.get('user:1001');
await redis.setex('session:abc', 3600, 'data');
await redis.setnx('user:1002', 'Bob');

// 计数器
await redis.set('counter', 100);
await redis.incr('counter');           // 101
await redis.incrby('counter', 10);  // 111

// Hash 操作
await redis.hset('user:1001', {
  name: 'Alice',
  email: 'alice@example.com',
  age: 30
});
const name = await redis.hget('user:1001', 'name');
const user = await redis.hgetall('user:1001');

// List 操作
await redis.lpush('tasks', 'task1', 'task2');
await redis.rpush('tasks', 'task3');
const tasks = await redis.lrange('tasks', 0, -1);
const task = await redis.lpop('tasks');

// Set 操作
await redis.sadd('tags', 'redis', 'cache', 'database');
const tags = await redis.smembers('tags');
const isMember = await redis.sismember('tags', 'redis');

// Sorted Set 操作
await redis.zadd('leaderboard', 100, 'Alice', 90, 'Bob', 80, 'Charlie');
const top = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES');
const rank = await redis.zrevrank('leaderboard', 'Alice');

// 发布订阅
const subscriber = new Redis();
subscriber.subscribe('channel1', 'channel2');
subscriber.on('message', (channel, message) => {
  console.log(`${channel}: ${message}`);
});

// 管道
const pipeline = redis.pipeline();
pipeline.set('key1', 'value1');
pipeline.get('key1');
pipeline.hset('user:1001', 'name', 'Alice');
const results = await pipeline.exec();

// Lua 脚本
const rateLimitScript = `
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local current = tonumber(redis.call('GET', key) or 0)
if current >= limit then
    return 0
end
current = redis.call('INCR', key)
if current == 1 then
    redis.call('EXPIRE', key, ARGV[2])
end
return current
`;

const result = await redis.eval(
  rateLimitScript, 
  1, 
  'ratelimit:user:1', 
  10,  // limit
  60   // window
);
```

### ioredis 高级功能

```javascript
const Redis = require('ioredis');

// 哨兵模式
const redis = new Redis({
  sentinels: [
    { host: 'localhost', port: 26379 },
    { host: 'localhost', port: 26380 },
  ],
  name: 'mymaster',
  password: null,
});

// 流处理
await redis.xadd('mystream', '*', 'field1', 'value1', 'field2', 'value2');
const messages = await redis.xread('STREAMS', 'mystream', '0-0');
const pending = await redis.xpending('mystream', 'mygroup');

// 事务
const multi = redis.multi();
multi.set('key1', 'value1');
multi.get('key1');
multi.hset('user:1001', 'name', 'Alice');
const results = await multi.exec();

// 监视（乐观锁）
const watch = redis.watch('key1');
// 业务逻辑
const result = await redis.multi()
  .set('key1', 'new_value')
  .exec();  // 如果 key1 被其他客户端修改，返回 null

// 连接事件
redis.on('connect', () => console.log('Connected'));
redis.on('error', (err) => console.error('Error:', err));
redis.on('close', () => console.log('Connection closed'));

// 自动重连
const redis = new Redis({
  host: 'localhost',
  port: 6379,
  retryStrategy: (times) => {
    const delay = Math.min(times * 50, 2000);
    return delay;
  },
  maxRetriesPerRequest: 3,
});
```

---

## 常见陷阱与最佳实践

### 陷阱 1：KEYS 命令

```bash
# ❌ 错误：生产环境使用 KEYS
KEYS *  # 会阻塞服务器

# ✅ 正确：使用 SCAN
SCAN 0 MATCH user:* COUNT 100
```

### 陷阱 2：不设置过期时间

```bash
# ❌ 错误：缓存数据永不过期
SET cache:data "value"

# ✅ 正确：设置过期时间
SETEX cache:data 3600 "value"  # 1 小时过期
```

### 陷阱 3：使用 Keys 作为主键

```bash
# ❌ 错误：使用 KEYS 模式
KEYS user:*
KEYS post:*:comments

# ✅ 正确：使用 SCAN 或有序集合
SCAN 0 MATCH user:* COUNT 100

# 或使用有序集合作为索引
ZADD users:index 0 "user:1"  # member 是键名，score 用于排序
ZRANGE users:index 0 -1
```

### 最佳实践清单

1. **键命名规范**：`type:id` 格式，如 `user:123`、`session:abc`。
2. **过期时间**：为缓存数据设置合理的过期时间。
3. **内存管理**：配置 `maxmemory` 和淘汰策略。
4. **持久化策略**：根据需求选择 RDB/AOF/混合。
5. **连接池**：使用连接池复用连接。
6. **Pipeline**：批量操作使用 Pipeline 减少 RTT。
7. **Lua 脚本**：原子操作使用 Lua 脚本。
8. **监控指标**：监控内存使用、命中率、慢查询。
9. **安全配置**：设置密码、绑定 IP、禁用危险命令。
10. **集群部署**：大规模数据使用 Redis Cluster。

---

## 与其他数据库对比

### Redis vs Memcached

| 特性 | Redis | Memcached |
|------|-------|------------|
| **数据类型** | 多种 | String |
| **持久化** | 支持 | 不支持 |
| **复制** | 支持 | 不支持 |
| **集群** | 原生支持 | 需要客户端分片 |
| **内存管理** | 多种策略 | LRU |
| **性能** | 相当 | 相当 |

### Redis vs SQLite

| 特性 | Redis | SQLite |
|------|-------|--------|
| **类型** | 内存为主 | 磁盘为主 |
| **持久化** | 可选 | 必须 |
| **容量** | 受内存限制 | 可达 TB |
| **适用** | 缓存/会话 | 嵌入式 |

### Redis vs PostgreSQL

| 特性 | Redis | PostgreSQL |
|------|-------|------------|
| **类型** | KV/数据结构 | 关系型 |
| **持久化** | 可选 | 必须 |
| **查询** | 有限 | SQL |
| **容量** | 受内存限制 | 无限制 |
| **适用** | 缓存/实时 | 主数据库 |

---

## Redis Stream 详解

### Stream 基础

```bash
# 1. 创建 Stream
XADD mystream * field1 value1 field2 value2
# 返回消息 ID：1703123456789-0

# 2. 指定 ID 添加
XADD mystream 1703123456789-0 field1 value1

# 3. 读取新消息
XREAD STREAMS mystream $  # $ 表示最新消息之后

# 4. 读取所有消息
XREAD STREAMS mystream 0

# 5. 读取范围
XRANGE mystream 1703123456789-0 1703123456799-0
XRANGE mystream - +  # - 表示最小，+ 表示最大

# 6. 读取指定数量的消息
XRANGE mystream - + COUNT 10

# 7. 删除消息
XDEL mystream 1703123456789-0

# 8. 获取长度
XLEN mystream

# 9. 监听新消息（阻塞）
XREAD BLOCK 5000 STREAMS mystream $
```

### 消费者组

```bash
# 1. 创建消费者组
XGROUP CREATE mystream mygroup 0  # 从头开始
XGROUP CREATE mystream mygroup $  # 只读取新消息

# 2. 读取消费者组
XREADGROUP GROUP mygroup consumer1 STREAMS mystream ">"

# 3. 手动确认
XACK mystream mygroup 1703123456789-0

# 4. 查看待确认消息
XPENDING mystream mygroup

# 5. 查看消费者组信息
XINFO GROUPS mystream

# 6. 创建消费者
XGROUP CREATECONSUMER mystream mygroup consumer2

# 7. 转移未确认消息
XCLAIM mystream mygroup consumer2 60000 1703123456789-0
# 60000 = 60秒未确认则转移

# 8. 自动创建 Stream 和组
XREADGROUP GROUP mygroup consumer1 STREAMS mystream ">"
```

### Stream 应用场景

```python
import redis

r = redis.Redis()

# 场景 1：消息队列
def message_queue_example():
    stream_name = 'orders:stream'
    group_name = 'orders:processors'
    
    # 创建消费者组
    try:
        r.xgroup_create(stream_name, group_name, id='0', mkstream=True)
    except redis.exceptions.ResponseError:
        pass  # 组已存在
    
    # 生产者：添加消息
    r.xadd(stream_name, {'order_id': '123', 'amount': '99.99', 'status': 'pending'})
    
    # 消费者：读取并处理
    messages = r.xreadgroup(group_name, 'consumer1', {stream_name: '>'}, count=10)
    
    for stream, msgs in messages:
        for msg_id, data in msgs:
            print(f"Processing order: {data}")
            # 处理订单...
            r.xack(stream_name, group_name, msg_id)

# 场景 2：实时事件流
def event_stream_example():
    stream_name = 'events:stream'
    
    # 发送事件
    r.xadd(stream_name, {
        'event_type': 'page_view',
        'user_id': '123',
        'page': '/products',
        'timestamp': str(int(time.time()))
    })
    
    # 读取最近事件
    events = r.xrevrange(stream_name, '+', '-', count=100)

# 场景 3：日志聚合
def log_aggregation_example():
    stream_name = 'logs:stream'
    group_name = 'log:processors'
    
    # 多个应用实例写入
    r.xadd(stream_name, {
        'service': 'api',
        'level': 'error',
        'message': 'Database connection failed',
        'timestamp': str(int(time.time()))
    })
    
    # 多个消费者处理
    messages = r.xreadgroup(group_name, f'worker-{uuid.uuid4()}', {stream_name: '>'})
```

---

## Redis Pub/Sub 进阶

### 基础操作

```bash
# 订阅频道
SUBSCRIBE channel1 channel2

# 模式订阅
PSUBSCRIBE pattern.*

# 发布消息
PUBLISH channel1 "Hello World"

# 取消订阅
UNSUBSCRIBE channel1
PUNSUBSCRIBE pattern.*

# 查看频道信息
PUBSUB CHANNELS
PUBSUB NUMSUB channel1
PUBSUB NUMPAT
```

### 高级特性

```python
import redis

r = redis.Redis()

# 1. 模式订阅
p = r.pubsub()
p.psubscribe('notifications:*', 'alerts:*')

# 2. 混合订阅
p.subscribe('chat:general')  # 普通订阅
p.psubscribe('chat:*')  # 模式订阅

# 3. 监听消息
for message in p.listen():
    if message['type'] == 'message':
        print(f"Channel: {message['channel']}, Data: {message['data']}")
    elif message['type'] == 'pmessage':
        print(f"Pattern: {message['pattern']}, Channel: {message['channel']}, Data: {message['data']}")

# 4. 可靠的消息处理
def reliable_pubsub():
    # 使用 Stream 实现可靠的消息传递
    stream_name = 'reliable:messages'
    group_name = 'message:handlers'
    
    # 初始化
    try:
        r.xgroup_create(stream_name, group_name, id='0', mkstream=True)
    except:
        pass
    
    # 订阅者定期检查新消息
    messages = r.xreadgroup(group_name, 'consumer1', {stream_name: '>'}, count=10)
    
    for stream, msgs in messages:
        for msg_id, data in msgs:
            try:
                process_message(data)
                r.xack(stream_name, group_name, msg_id)
            except:
                # 重新入队
                r.xadd(stream_name, data)
                r.xack(stream_name, group_name, msg_id)

# 5. 发布确认
def pubsub_with_ack():
    pubsub = r.pubsub()
    pubsub.subscribe('ack-channel')
    
    # 监听确认
    for message in pubsub.listen():
        if message['type'] == 'message':
            # 处理消息后发送确认
            r.publish('ack-response', message['data'])
```

### 实际应用

```python
# 场景 1：实时聊天
def chat_application():
    # 消息发布
    def send_message(room_id, user_id, message):
        chat_stream = f'chat:{room_id}:messages'
        r.xadd(chat_stream, {
            'user_id': user_id,
            'message': message,
            'timestamp': str(int(time.time() * 1000))
        })
        # 也发布到 Pub/Sub 通知在线用户
        r.publish(f'chat:{room_id}', f'{user_id}:{message}')
    
    # 消息订阅
    def subscribe_to_room(room_id):
        pubsub = r.pubsub()
        pubsub.subscribe(f'chat:{room_id}')
        return pubsub

# 场景 2：实时通知
def notification_system():
    def send_notification(user_id, notification_type, data):
        notification = {
            'type': notification_type,
            'data': json.dumps(data),
            'timestamp': str(int(time.time() * 1000))
        }
        r.xadd(f'notifications:{user_id}', notification)
        r.publish(f'user:{user_id}:notifications', json.dumps(notification))
    
    def get_unread_count(user_id):
        # 读取未读通知
        count = r.xlen(f'notifications:{user_id}')
        return count

# 场景 3：分布式锁
def distributed_lock_example():
    lock_key = 'resource:lock'
    
    # 获取锁
    def acquire_lock(resource_id, timeout=10):
        lock_id = str(uuid.uuid4())
        acquired = r.set(f'{lock_key}:{resource_id}', lock_id, nx=True, ex=timeout)
        return lock_id if acquired else None
    
    # 释放锁
    def release_lock(resource_id, lock_id):
        # 使用 Lua 脚本确保只释放自己的锁
        script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        r.eval(script, 1, f'{lock_key}:{resource_id}', lock_id)
    
    # 使用锁
    lock_id = acquire_lock('order-processing')
    if lock_id:
        try:
            # 处理业务
            pass
        finally:
            release_lock('order-processing', lock_id)
```

---

## Redis 集群进阶

### Redis Cluster 配置

```bash
# redis.conf 集群配置
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 15000
cluster-replica-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes

# 节点配置示例
# 节点 1 (主)
port 7001
cluster-enabled yes
cluster-config-file nodes-7001.conf
cluster-node-timeout 15000

# 节点 2 (主)
port 7002
cluster-enabled yes
cluster-config-file nodes-7002.conf
cluster-node-timeout 15000
```

### 集群管理命令

```bash
# 1. 创建集群
redis-cli --cluster create 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 \
  127.0.0.1:7004 127.0.0.1:7005 127.0.0.1:7006 \
  --cluster-replicas 1

# 2. 查看集群信息
redis-cli -p 7001 cluster nodes
redis-cli -p 7001 cluster info
redis-cli -p 7001 cluster slots

# 3. 添加节点
redis-cli --cluster add-node 127.0.0.1:7007 127.0.0.1:7001

# 4. 添加从节点
redis-cli --cluster add-node 127.0.0.1:7008 127.0.0.1:7001 \
  --cluster-slave --cluster-master-id <node-id>

# 5. 删除节点
redis-cli --cluster del-node 127.0.0.1:7001 <node-id>

# 6. 重新分片
redis-cli --cluster reshard 127.0.0.1:7001
# 交互式重新分配槽位

# 7. 负载均衡
redis-cli --cluster rebalance 127.0.0.1:7001 \
  --cluster-threshold 1

# 8. 故障转移
# 在从节点上执行
CLUSTER FAILOVER

# 9. 手动故障转移
# 在主节点上执行
CLUSTER FAILOVER FORCE

# 10. 设置从节点为新的主节点
redis-cli --cluster takeover 127.0.0.1:7001
```

### Python 集群客户端

```python
from redis.cluster import RedisCluster

# 创建集群客户端
rc = RedisCluster(
    host='127.0.0.1',
    port=7001,
    decode_responses=True
)

# 基本操作
rc.set('key1', 'value1')
rc.get('key1')

# 批量操作
pipe = rc.pipeline()
pipe.set('key1', 'value1')
pipe.get('key1')
pipe.hset('hash1', 'field1', 'value1')
results = pipe.execute()

# 跨槽位操作
# 注意：集群不支持跨槽位的批量操作

# 槽位信息
slots_info = rc.cluster_slots()

# 获取节点
node = rc.get_node('key1')

# 重定向处理
# 客户端会自动处理 MOVED 重定向
```

### 集群路由

```python
# 1. 键映射到槽位
def get_slot(key):
    """计算键的槽位"""
    return crc16(key) % 16384

# 2. 键命名策略
# 好的命名：user:123:profile（同一用户的数据在同一槽位）
# 坏的命名：user:profile:123（无关联）

# 3. 标签哈希
# 使用 {} 强制将多个键映射到同一槽位
rc.set('user:{123}:profile', '...')
rc.set('user:{123}:settings', '...')
# 这两个键会在同一个槽位

# 4. 跨槽位操作
# 使用 hash tags
def mset_user_data(user_id, **kwargs):
    """一次性设置用户的所有相关数据"""
    keys = [f'user:{user_id}:profile', f'user:{user_id}:settings', f'user:{user_id}:stats']
    pipe = rc.pipeline()
    for key, value in kwargs.items():
        if key in ['profile', 'settings', 'stats']:
            pipe.set(f'user:{user_id}:{key}', json.dumps(value))
    pipe.execute()

# 5. 限制操作
# 不支持的操作：
# - 跨槽位的 KEYS/SCAN
# - 跨槽位的 SUNION/SINTER/SDIFF
# - 跨槽位的事务（MULTI/EXEC）
# - 跨槽位的 Lua 脚本

# 解决方案：使用 hash tags
```

---

## Redis Sentinel 高可用

### Sentinel 配置

```bash
# sentinel.conf
port 26379
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 30000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
sentinel auth-pass mymaster yourpassword
sentinel deny-scripts-reconfig yes
```

### Sentinel 命令

```bash
# 查看主节点信息
SENTINEL master mymaster
SENTINEL masters
SENTINEL slaves mymaster
SENTINEL sentinels mymaster

# 获取当前主节点地址
SENTINEL get-master-addr-by-name mymaster

# 强制故障转移
SENTINEL failover mymaster

# 重置主节点
SENTINEL reset mymaster
```

### Python Sentinel 客户端

```python
from redis.sentinel import Sentinel

# 创建 Sentinel
sentinel = Sentinel([
    ('127.0.0.1', 26379),
    ('127.0.0.1', 26380),
    ('127.0.0.1', 26381)
], socket_timeout=0.1)

# 获取主节点
master = sentinel.master_for('mymaster', socket_timeout=0.1, decode_responses=True)

# 获取从节点
slave = sentinel.slave_for('mymaster', socket_timeout=0.1, decode_responses=True)

# 使用主节点写入
master.set('key', 'value')

# 使用从节点读取
value = slave.get('key')

# 自动故障转移
# Sentinel 会自动检测主节点故障并提升从节点
```

---

## Redis 监控与诊断

### 关键监控指标

```bash
# 1. INFO 命令
INFO memory          # 内存使用
INFO stats           # 操作统计
INFO clients         # 客户端连接
INFO replication     # 复制状态
INFO cpu             # CPU 使用
INFO persistence     # 持久化状态
INFO cluster         # 集群状态
INFO keyspace        # 键空间信息

# 2. 慢查询日志
SLOWLOG GET 10      # 获取最近 10 条慢查询
SLOWLOG LEN
SLOWLOG RESET

# 3. 实时统计
MONITOR              # 实时监控命令（生产环境慎用）
DEBUG SLEEP 0.5

# 4. 客户端列表
CLIENT LIST
CLIENT KILL ip:port

# 5. 内存分析
MEMORY DOCTOR        # 诊断内存问题
MEMORY STATS         # 详细内存统计
MEMORY USAGE key     # 键的内存使用
```

### 性能调优

```bash
# 1. 内存优化
maxmemory 2gb
maxmemory-policy allkeys-lru
maxmemory-samples 5

# 2. 网络优化
tcp-keepalive 300
tcp-backlog 511

# 3. 持久化优化
save 900 1
save 300 10
save 60 10000
rdbcompression yes
rdbchecksum yes

# 4. AOF 优化
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

### 日志分析

```python
# 分析慢查询
def analyze_slow_queries(r):
    slow_logs = r.slowlog_get(100)
    
    for log in slow_logs:
        print(f"""
        ID: {log['id']}
        Time: {log['start_time']}
        Duration: {log['duration']}ms
        Command: {' '.join(log['args'])}
        """)
    
    # 分析常见的慢命令
    command_counts = {}
    for log in slow_logs:
        cmd = log['args'][0].upper()
        command_counts[cmd] = command_counts.get(cmd, 0) + 1
    
    print("Most common slow commands:")
    for cmd, count in sorted(command_counts.items(), key=lambda x: -x[1])[:10]:
        print(f"  {cmd}: {count}")

# 内存分析
def analyze_memory(r):
    info = r.info('memory')
    print(f"""
    Used Memory: {info['used_memory_human']}
    Peak Memory: {info['used_memory_peak_human']}
    Memory Fragmentation: {info['mem_fragmentation_ratio']}
    Evicted Keys: {info['evicted_keys']}
    """)
    
    # 获取大键
    keys = r.execute_command('KEYS', '*')
    large_keys = []
    for key in keys[:1000]:  # 限制采样
        size = r.memory_usage(key)
        if size and size > 1024 * 1024:  # 大于 1MB
            large_keys.append((key, size))
    
    large_keys.sort(key=lambda x: -x[1])
    print("Large Keys:")
    for key, size in large_keys[:10]:
        print(f"  {key}: {size / 1024 / 1024:.2f}MB")
```

---

## Redis 安全配置

### 认证与访问控制

```bash
# 1. 设置密码
requirepass yourpassword

# 2. 命令重命名/禁用
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command KEYS ""
rename-command CONFIG ""

# 3. IP 白名单
bind 127.0.0.1
protected-mode yes

# 4. TLS 加密
tls-port 6380
port 0
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```

### 安全最佳实践

```python
# 1. 使用密码连接
r = redis.Redis(host='localhost', port=6379, password='yourpassword', ssl=True)

# 2. 限制命令
# 在配置文件中
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""

# 3. 使用 ACL
# Redis 6+ ACL
# redis.conf
user readonly on >password ~cached:* -@all +get
user app on >password ~app:* +@all -@dangerous

# 4. 连接池安全
from redis import ConnectionPool

pool = ConnectionPool(
    host='localhost',
    port=6379,
    password='password',
    max_connections=50,
    ssl=True,
    ssl_cert_reqs='required'
)
```

---

## Redis 与 AI 应用

### LLM 缓存

```python
import redis
import hashlib
import json

r = redis.Redis(decode_responses=True)

def cache_llm_response(prompt, model, response, ttl=3600):
    """缓存 LLM 响应"""
    cache_key = f'llm:response:{model}:{hashlib.md5(prompt.encode()).hexdigest()}'
    r.setex(cache_key, ttl, json.dumps({
        'prompt': prompt,
        'response': response,
        'cached_at': int(time.time())
    }))

def get_cached_response(prompt, model):
    """获取缓存的响应"""
    cache_key = f'llm:response:{model}:{hashlib.md5(prompt.encode()).hexdigest()}'
    cached = r.get(cache_key)
    if cached:
        data = json.loads(cached)
        print(f"Cache hit! Cached at: {data['cached_at']}")
        return data['response']
    return None

def llm_call_with_cache(prompt, model='gpt-4'):
    """带缓存的 LLM 调用"""
    # 检查缓存
    cached = get_cached_response(prompt, model)
    if cached:
        return cached
    
    # 调用 LLM
    response = call_llm_api(prompt, model)
    
    # 缓存结果
    cache_llm_response(prompt, model, response)
    
    return response
```

### Token 计数缓存

```python
def cache_token_count(text, count):
    """缓存 token 计数"""
    cache_key = f'tokens:{hashlib.md5(text.encode()).hexdigest()}'
    r.setex(cache_key, 86400, str(count))  # 缓存 24 小时

def get_token_count(text):
    """获取缓存的 token 计数"""
    cache_key = f'tokens:{hashlib.md5(text.encode()).hexdigest()}'
    cached = r.get(cache_key)
    return int(cached) if cached else None
```

### Embedding 缓存

```python
def cache_embeddings(texts, model, embeddings):
    """批量缓存 embeddings"""
    pipe = r.pipeline()
    for text, embedding in zip(texts, embeddings):
        cache_key = f'embedding:{model}:{hashlib.md5(text.encode()).hexdigest()}'
        pipe.setex(cache_key, 604800, json.dumps(embedding))  # 7 天
    pipe.execute()

def get_cached_embeddings(texts, model):
    """批量获取缓存的 embeddings"""
    results = []
    cache_misses = []
    
    for i, text in enumerate(texts):
        cache_key = f'embedding:{model}:{hashlib.md5(text.encode()).hexdigest()}'
        cached = r.get(cache_key)
        if cached:
            results.append((i, json.loads(cached)))
        else:
            cache_misses.append((i, text))
    
    return results, cache_misses
```

### 向量相似度搜索

```python
# Redis Stack 的向量搜索
def setup_vector_index(r):
    """设置向量索引"""
    r.execute_command('FT.CREATE', 'embeddings_idx', 'SCHEMA',
        'id', 'TAG', 'INDEXED',
        'embedding', 'VECTOR', 'FLAT', '6', 'TYPE', 'FLOAT32',
        'DIM', '1536', 'DISTANCE_METRIC', 'COSINE')

def add_vector(r, doc_id, embedding):
    """添加向量"""
    r.hset(f'vector:{doc_id}', mapping={
        'id': doc_id,
        'embedding': np.array(embedding).astype(np.float32).tobytes()
    })

def search_similar(r, query_vector, top_k=10):
    """搜索相似向量"""
    query = np.array(query_vector).astype(np.float32).tobytes()
    results = r.execute_command(
        'FT.SEARCH', 'embeddings_idx',
        f'*=>[KNN {top_k} @embedding $vec AS score]',
        'PARAMS', '2', 'vec', query,
        'RETURN', '2', 'id', 'score',
        'SORTBY', 'score',
        'DIALECT', '2'
    )
    return results
```

---

> [!SUCCESS]
> 本文档全面介绍了 Redis 的核心数据结构、高级特性、持久化、集群、Sentinel 以及在 AI 应用中的实战场景。Redis 凭借其高性能、丰富的数据结构和强大的扩展能力，是现代应用架构中不可或缺的组件。
