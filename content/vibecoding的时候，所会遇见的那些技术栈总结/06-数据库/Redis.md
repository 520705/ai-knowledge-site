---
date: 2026-04-24
tags:
  - 数据库
  - Redis
  - 内存数据库
  - 缓存
  - AI缓存
---

# Redis 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Redis 8.0 新特性、7 种核心数据结构详解、持久化机制、Redis Cluster 集群、Pub/Sub 与 Stream、Redis + Lua 脚本、Redis + AI 应用实战，以及 Dragonfly/Memurai 等替代方案对比。

---

## 目录

1. [[#Redis 概述与核心定位]]
2. [[#Redis 数据结构详解]]
3. [[#Redis 作为缓存 vs 作为数据库]]
4. [[#Pub/Sub 发布订阅与 Stream 流]]
5. [[#Redis + Lua 脚本]]
6. [[#Redis + Node.js：ioredis 与 node-redis]]
7. [[#Redis + Python：redis-py 与实战]]
8. [[#API 限流实现]]
9. [[#会话管理实战]]
10. [[#Redis + AI：LLM 缓存与 Token 计数]]
11. [[#Redis Cluster 与 Sentinel 高可用]]
12. [[#内存优化与最佳实践]]
13. [[#Redis vs Memcached vs Dragonfly 对比]]
14. [[#选型建议与适用场景]]

---

## Redis 概述与核心定位

### 什么是 Redis

Redis（REmote DIctionary Server）是一个开源的、基于内存的键值存储系统，由 Salvatore Sanfilippo 于 2009 年创建。最初它只是一个简单的日志系统采样器，如今已经发展成为全球最流行的内存数据库和缓存解决方案。

Redis 的核心理念是「数据保存在内存中，读写速度极快」。与传统磁盘数据库不同，Redis 将数据存储在 RAM 中，这使得它的读写延迟可以低至微秒级别——每秒可以处理数百万次操作。对于需要快速响应的应用场景，Redis 是当之无愧的首选。

但 Redis 绝不仅仅是一个缓存。它提供了丰富的数据结构——字符串、哈希、列表、集合、有序集合、流——每一种数据结构都针对特定的场景进行了优化。理解这些数据结构，是掌握 Redis 的关键。很多开发者把 Redis 当作一个「带多种数据结构的缓存」来用，这其实大大低估了 Redis 的能力。你可以用 Redis 做消息队列（List/Stream）、做排行榜（Sorted Set）、做实时分析（HyperLogLog/Bitmap）、做分布式锁（String/NX/PX）、甚至做向量搜索（Redis Stack 的 VECTOR 类型）。

### Redis 在 AI 时代的价值

2026 年是 AI 应用爆发的年份，Redis 在这个时代找到了新的价值定位。

首先是 **LLM 响应缓存**。大语言模型的 API 调用既慢又贵，每次请求可能需要几秒钟、消耗几分钱。通过将相同或相似的 prompt 响应缓存到 Redis，可以大幅降低延迟和成本。一个典型的例子是：如果用户反复询问相同或相似的问题，第二次可以直接从缓存返回结果，而不需要再次调用昂贵的 LLM API。

其次是 **Token 计数缓存**。Token 是 LLM 计费的基本单位，统计用户消耗的 Token 数量是一个常见需求。Redis 的 HINCRBY 命令天然适合做原子递增计数，配合过期策略可以轻松实现日/月 Token 用量统计。

第三是 **对话上下文存储**。AI 对话应用需要存储用户的对话历史上下文。Redis 的 String 类型可以存储序列化的对话记录，配合过期时间可以自动清理旧对话。

第四是 **AI 任务队列**。处理 AI 请求通常是异步的——用户提交一个摘要任务，系统返回一个任务 ID，然后后台处理完成后通知用户。Redis 的 List 和 Stream 数据结构非常适合做这种任务队列。

第五是 **向量相似度搜索**。Redis Stack 的 RediSearch 模块支持向量索引和近似最近邻搜索，可以作为 AI RAG（检索增强生成）系统的向量数据库使用。

### Redis 市场份额与行业地位

Redis 在键值存储和缓存市场中占据绝对主导地位。根据 DB-Engine 的统计数据，Redis 长期位列内存数据库市场第一名，市场份额超过 50%。

| 排名 | 数据库 | 市场份额 | 趋势 |
|------|--------|---------|------|
| 1 | Redis | 58.3% | → |
| 2 | Memcached | 22.1% | ↓ |
| 3 | Dragonfly | 8.5% | ↑ |
| 4 | Hazelcast | 5.2% | → |
| 5 | Aerospike | 3.8% | ↑ |

这些数字背后是真实的工程实践。Twitter 用 Redis 存储用户时间线和 timelines、GitHub 用 Redis 做缓存和 API 限流、Stack Overflow 用 Redis 做会话存储和热门问题缓存、Snapchat 用 Redis 存储 Stories 和实时数据。可以说，Redis 已经成为现代互联网架构的基础设施之一。

---

## Redis 数据结构详解

Redis 提供了 7 种核心数据结构，每一种都有其独特的能力和适用场景。理解这些数据结构，是用好 Redis 的基础。

### String（字符串）

String 是 Redis 最基本的数据类型，可以存储字符串、整数或浮点数。虽然名字叫「字符串」，但 String 类型实际上是二进制安全的——你可以用它存储任何序列化后的数据，如 JSON、XML、图片二进制等。

String 类型最常见的用途是缓存 HTML 片段、API 响应、数据库查询结果等。使用 `SET` 和 `GET` 命令可以轻松存取字符串值，而 `SETEX` 和 `SETNX` 则提供了过期时间和原子性设置的能力。

```bash
# 基本操作
SET user:1001 "Alice"
GET user:1001                    # 返回 "Alice"

# 设置过期时间
SET session:abc123 "data" EX 3600   # 1小时后过期
SETEX cache:key 300 "value"         # SET + 5分钟过期

# 原子递增递减（计数器场景）
SET counter 100
INCR counter                       # 101（原子递增）
INCRBY counter 10                  # 111（增加10）
DECR counter                       # 110（原子递减）
INCRBYFLOAT price 0.5             # 110.5（浮点递增）

# 批量操作
MSET user:1001 "Alice" user:1002 "Bob" user:1003 "Charlie"
MGET user:1001 user:1002 user:1003
# 返回: ["Alice", "Bob", "Charlie"]

# 字符串追加
APPEND article:1:title " - Advanced"   # 返回新长度

# 位操作（统计/布隆过滤器场景）
SETBIT visited:2024:01:15 1000001 1   # 设置某位为1
GETBIT visited:2024:01:15 1000001     # 获取某位
BITCOUNT visited:2024:01:15            # 统计1的个数（UV统计）
```

### Hash（哈希）

Hash 是 Redis 中的「对象」类型，它存储字段到值的映射，类似于 Python 的 dict 或 Java 的 HashMap。与其把用户信息序列化成 JSON 字符串存到 String 类型中，不如直接用 Hash 存储——这样可以单独访问或修改对象的某个字段，而不需要解析整个 JSON。

Hash 的内部实现有两种编码方式：小数据量时使用 ziplist（压缩列表），大数据量时使用 hashtable（哈希表）。Redis 会根据数据量自动选择更高效的编码方式。

```bash
# 基本操作
HSET user:1001 name "Alice" email "alice@example.com" age 30
HGET user:1001 name                   # 返回 "Alice"
HGETALL user:1001
# 返回: {"name":"Alice","email":"alice@example.com","age":"30"}

# 批量操作
HMSET user:1002 name "Bob" email "bob@example.com"
HMGET user:1001 user:1002 name
# 返回: ["Alice", "Bob"]

# 字段是否存在
HEXISTS user:1001 name                # 1（存在）

# 删除字段
HDEL user:1001 age                    # 删除 age 字段

# 字段值递增（适合统计场景）
HINCRBY user:1001 login_count 1      # 登录次数 +1

# 获取所有字段或值
HKEYS user:1001                        # ["name", "email", "login_count"]
HVALS user:1001                        # ["Alice", "alice@example.com", "5"]

# 字段数量
HLEN user:1001                          # 3
```

Hash 类型的典型应用场景包括：存储用户对象（与将用户序列化为 JSON 相比，可以单独修改字段）、存储购物车商品、存储配置信息等。

### List（列表）

List 是一个双向链表，支持从两端插入和弹出元素。这使得 List 可以高效地实现队列（先进先出）和栈（先进后出）两种数据结构。List 的操作时间复杂度是 O(1)，无论列表多长，在头部或尾部插入元素的速度都是一样的。

需要注意的是，List 的索引访问操作（如 LINDEX）是 O(N) 的，如果需要频繁在中间位置插入或访问，应该考虑其他数据结构。

```bash
# LPUSH/RPOP = 队列（FIFO）
LPUSH tasks "task1"
LPUSH tasks "task2"
LPUSH tasks "task3"
LRANGE tasks 0 -1                      # ["task3","task2","task1"]

RPOP tasks                             # 返回 "task1"（先进先出）

# LPUSH/LPOP = 栈（LIFO）
LPOP tasks                             # 返回 "task3"（先进后出）

# 阻塞操作（消息队列核心）
BLPOP tasks 0                          # 阻塞等待，直到有元素可弹出
BRPOP tasks 0                          # 从右端阻塞等待

# 索引操作
LINDEX tasks 0                         # 获取索引0的元素
LINSERT tasks BEFORE "task2" "task1.5" # 在 task2 前插入
LSET tasks 0 "new_first"               # 设置索引0的元素

# 范围操作
LTRIM tasks 0 99                       # 保留索引0-99的元素（切片）

# 长度
LLEN tasks

# 批量插入
RPUSH order:1001 "item1" "item2" "item3"
```

List 类型最适合做消息队列、轻量级任务队列、最新消息列表（如微博时间线）、最近访问记录等场景。

### Set（集合）

Set 是一个无序的、不可重复的元素集合。Set 底层使用 hashtable 实现，所有操作都是 O(1) 的时间复杂度。Set 最强大的特性是支持集合运算：交集（SINTER）、并集（SUNION）、差集（SDIFF）。这些运算在社交网络（共同好友）、标签系统（共同标签）、推荐系统等场景下非常有用。

```bash
# 基本操作
SADD tags:article:1 "redis" "cache" "database"
SMEMBERS tags:article:1               # 返回所有成员（无序）
SISMEMBER tags:article:1 "redis"      # 1（存在）
SREM tags:article:1 "cache"           # 删除元素

# 集合运算
SADD set:a 1 2 3 4 5
SADD set:b 4 5 6 7 8

SUNION set:a set:b                    # 并集 {1,2,3,4,5,6,7,8}
SINTER set:a set:b                    # 交集 {4,5}
SDIFF set:a set:b                     # 差集 {1,2,3}（a 独有）

# 保存运算结果
SUNIONSTORE result set:a set:b        # 结果存入 result
SINTERSTORE common set:a set:b
SDIFFSTORE only_a set:a set:b

# 随机操作
SRANDMEMBER set:a 2                   # 随机获取2个（不删除）
SPOP set:a 1                          # 随机弹出1个（会删除）

# 元素数量
SCARD set:a                           # 5
```

### Sorted Set（有序集合）

Sorted Set（简称 ZSet）是 Redis 最复杂也最强大的数据结构之一。它是一个有序的、不可重复的元素集合，每个元素都关联一个浮点数分数（score），元素按分数从低到高排序。当多个元素分数相同时，按字典序排序。

Sorted Set 底层使用跳表（Skip List）和哈希表的组合实现。哈希表提供 O(1) 的成员查找，跳表提供 O(log N) 的范围查询和排序。这使得 ZSet 既能快速查找单个元素，又能高效地进行范围操作。

Sorted Set 最经典的用途是排行榜系统——玩家的分数作为 score，玩家 ID 作为 member，可以轻松实现「获取前 10 名」「查询某玩家的排名」「玩家分数变化时更新排名」等操作。

```bash
# 基本操作（排行榜场景）
ZADD leaderboard 100 "Alice" 90 "Bob" 80 "Charlie" 95 "David"
ZRANGE leaderboard 0 -1 WITHSCORES
# 返回: ["Charlie", 80, "Bob", 90, "David", 95, "Alice", 100]（升序）

ZREVRANGE leaderboard 0 -1 WITHSCORES  # 降序排列

# 按分数范围查询
ZRANGEBYSCORE leaderboard 90 100       # 查询 90-100 分的玩家
ZCOUNT leaderboard 80 95               # 统计 80-95 分的玩家数量

# 排名查询
ZRANK leaderboard "Alice"              # 3（0-indexed，从低到高）
ZREVRANK leaderboard "Alice"           # 0（从高到低排名）

# 分数操作
ZINCRBY leaderboard 10 "Bob"          # Bob 的分数 +10

# 带权重的集合运算
ZADD set:a 1 "x" 2 "y" 3 "z"
ZADD set:b 2 "y" 3 "z" 4 "w"
ZUNIONSTORE union 2 set:a set:b WEIGHTS 1 2   # 权重合并
ZUNIONSTORE sum 2 set:a set:b AGGREGATE SUM   # 求和合并

# 交集（共同关注、共同好友）
ZINTERSTORE common_tags 2 tags:user:1 tags:user:2
```

### Stream（流）

Stream 是 Redis 5.0 引入的数据结构，设计用于解决消息队列的需求。它本质上是一个只追加的日志系统，支持消费者组、消息 ID、范围查询等特性，被认为是 List 实现的消息队列的升级版。

Stream 的核心概念包括：消息 ID（自动生成或手动指定，格式为 timestamp-sequence）、消费者组（多个消费者协作处理消息，支持消息确认）、范围查询（可以读取指定 ID 范围的消息）。

```bash
# 基本操作（事件流/消息队列场景）
XADD events:2024 "*" type "click" user "alice" page "/home"
# "*" 表示自动生成 ID，返回类似 "1703123456789-0"

XADD events:2024 "*" type "purchase" product "widget" amount 99.99

XLEN events:2024                      # 2

# 范围读取
XRANGE events:2024 - +               # 读取所有消息
XRANGE events:2024 1703123456789-0 + # 从指定 ID 读取

# 消费者组
XGROUP CREATE events:2024 consumers "$"  # 创建消费者组
XREADGROUP GROUP consumers "c1" STREAMS events:2024 ">"  # 读取新消息

XPENDING events:2024 consumers         # 查看待确认消息

# 确认处理
XACK events:2024 consumers 1703123456789-0
```

Stream 的典型应用场景包括：实时事件处理系统、聊天消息历史、AI 异步任务队列、日志聚合系统等。

### Bitmap、HyperLogLog、Geospatial

除了 6 种核心数据结构，Redis 还提供了几种特殊用途的数据结构。

**Bitmap** 不是独立的数据类型，而是 String 类型的位操作扩展。它非常适合做「签到统计」「用户在线状态」「日活跃用户统计」等场景。例如，要统计某天有多少用户访问了你的网站，只需要将每个访问用户的 ID 对应到一个位，如果访问了就设置该位为 1，最后用 `BITCOUNT` 统计 1 的个数即可。

**HyperLogLog** 是一种概率算法数据结构，用于做基数统计（去重计数）。它可以用极少的内存（固定 12KB）估算集合中不重复元素的数量，标准误差约为 0.81%。这对于 UV（独立访客）统计非常有用——你可以用 HyperLogLog 统计每天的独立访客数量，内存占用远低于使用 Set。

**Geospatial**（GEO）是 Sorted Set 的扩展，专门用于存储地理位置信息。它允许你存储经纬度信息，并计算两个地点之间的距离、查询某个半径范围内的地点等效。这对于「附近的人」「附近的商家」等 LBS 功能非常有用。

---

## Redis 作为缓存 vs 作为数据库

### 两种使用模式的对比

Redis 有两种主要的使用模式：作为缓存层，或者作为主数据库。理解这两种模式的区别和适用场景，是正确使用 Redis 的前提。

**作为缓存**时，Redis 存储的是从其他数据源（通常是磁盘数据库）读取的数据的副本。缓存数据是易失的——当 Redis 重启或内存不足时，缓存数据可能会丢失。缓存模式的核心目标是加速读取，降低磁盘数据库的负载。

**作为数据库**时，Redis 是数据的最终存储位置。这意味着数据必须持久化到磁盘，即使 Redis 重启也不能丢失。数据库模式需要启用 RDB 或 AOF 持久化，并可能需要配置主从复制以保证数据安全。

| 维度 | 缓存模式 | 数据库模式 |
|------|---------|----------|
| **数据来源** | 从其他数据库复制 | 应用直接写入 |
| **持久化** | 可选（通常不需要） | 必须（启用 RDB/AOF） |
| **数据可靠性** | 可能丢失 | 持久可靠 |
| **内存策略** | LRU/LFU 淘汰 | 不淘汰（扩容） |
| **故障影响** | 仅影响缓存命中 | 影响业务数据 |
| **典型场景** | API 响应缓存、会话缓存 | 排行榜、会话存储、消息队列 |

### 缓存失效策略

作为缓存使用时，一个关键问题是：当源数据（磁盘数据库）更新时，缓存数据如何同步？这里有几种常见的策略。

**Cache Aside（旁路缓存）**是最常用的策略。读取时，先查缓存，如果命中则返回；如果未命中，则查数据库，然后写入缓存。写入时，先更新数据库，然后删除缓存（而非更新缓存）。这个策略的优点是实现简单、一致性好，缺点是首次读取和写入时各有一次缓存未命中。

**Read Through（读穿透）**是一种更自动化的策略。读取时，始终从缓存层发起请求；如果缓存未命中，缓存层自动查询数据库、写入缓存、返回数据。写入时，直接写入缓存层，由缓存层负责更新数据库。这种策略让应用代码更简洁，但增加了缓存层的复杂度。

**Write Through（写穿透）**与 Read Through 对应。写入时，同时更新缓存和数据库，保证两者一致性。缺点是写入延迟较高。

**Write Behind（写回）**则是先写入缓存，然后异步批量写入数据库。这种策略写入性能最高，但数据安全性最低——如果缓存服务器在数据写回之前崩溃，数据会丢失。

大多数应用推荐使用 Cache Aside 策略，它在实现复杂度和数据一致性之间取得了良好的平衡。

### 内存淘汰策略

当 Redis 内存不足时，需要淘汰一些数据。Redis 提供了 8 种内存淘汰策略：

| 策略 | 说明 |
|------|------|
| `noeviction` | 不淘汰，返回错误（默认） |
| `allkeys-lru` | 所有键中，删除最近最少使用（LRU） |
| `allkeys-random` | 所有键中，随机删除 |
| `volatile-lru` | 已设置过期键中，删除 LRU |
| `volatile-random` | 已设置过期键中，随机删除 |
| `volatile-ttl` | 已设置过期键中，删除 TTL 最短 |
| `volatile-lfu` | 已设置过期键中，删除最不常用（LFU） |
| `allkeys-lfu` | 所有键中，删除最不常用 |

对于缓存场景，`allkeys-lru` 是最常用的策略——它自动保留最热门的访问数据，淘汰冷门数据。对于有时间敏感性的缓存，`volatile-ttl` 更有意义。

---

## Pub/Sub 发布订阅与 Stream 流

### 发布订阅模式

Redis 的 Pub/Sub（发布/订阅）是一种简单的消息通信模式。发布者（Publisher）向频道（Channel）发送消息，订阅者（Subscriber）接收该频道的所有消息。Redis 的 Pub/Sub 是完全无状态的——消息是「即发即忘」的，如果订阅者在消息发布时不在线，该消息就丢失了。

Pub/Sub 适合对消息可靠性要求不高的场景，如实时通知、聊天消息、轻量级事件广播等。对于需要可靠消息传递的场景，应该使用 Stream 而非 Pub/Sub。

```python
import redis

r = redis.Redis()

# 订阅频道
pubsub = r.pubsub()
pubsub.subscribe('notifications', 'alerts')

# 模式订阅
pubsub.psubscribe('user:*:messages', 'system:*')

# 监听消息
for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"频道 {message['channel']}: {message['data']}")
    elif message['type'] == 'pmessage':
        print(f"匹配模式 {message['pattern']}: {message['data']}")

# 发布消息
r.publish('notifications', '你有新的订单')
r.publish('user:123:messages', 'Hello from user 123')
```

### Stream 流

对于需要可靠消息传递的场景，Redis Stream 是更好的选择。Stream 支持消费者组、消息确认、消息持久化等企业级消息队列特性，被认为是 Redis 实现消息队列的「正确方式」。

```python
# 消费者组示例
def message_queue_example():
    stream_name = 'orders:stream'
    group_name = 'orders:processors'
    
    # 初始化消费者组
    try:
        r.xgroup_create(stream_name, group_name, id='0', mkstream=True)
    except redis.exceptions.ResponseError as e:
        if 'BUSYGROUP' in str(e):
            pass  # 组已存在
        else:
            raise
    
    # 生产者：发送消息
    r.xadd(stream_name, {
        'order_id': '123',
        'amount': '99.99',
        'status': 'pending'
    })
    
    # 消费者：读取并处理
    messages = r.xreadgroup(
        group_name,
        'consumer1',
        {stream_name: '>'},  # '>' 表示只读取新消息
        count=10
    )
    
    for stream, msgs in messages:
        for msg_id, data in msgs:
            print(f"处理订单: {data}")
            # 处理订单...
            # 确认消息已处理
            r.xack(stream_name, group_name, msg_id)
```

Stream 与 Kafka/RabbitMQ 等专业消息队列相比，优势在于与 Redis 生态的集成——你可以用同一个 Redis 实例同时做缓存、Session 存储和消息队列，降低运维复杂度。但对于需要极高吞吐量和可靠性的场景，专业消息队列仍然是更好的选择。

---

## Redis + Lua 脚本

### Lua 脚本的价值

Redis 支持在服务器端执行 Lua 脚本，这是一个非常强大的特性。Lua 脚本在 Redis 中是原子执行的——从脚本开始到结束，不会有其他命令插入。这意味着你可以在 Lua 脚本中执行复杂的操作序列，而不用担心并发问题。

Lua 脚本的主要用途包括：

**原子操作**。当你需要将多个 Redis 操作组合成一个原子单元时，Lua 脚本是最佳选择。例如，实现分布式锁、检查并设置（check-and-set）、限流器等。

**减少网络往返**。将多个客户端命令合并成一个脚本执行，可以减少网络往返次数，降低延迟。

**封装业务逻辑**。将复杂的业务逻辑封装在 Lua 脚本中，可以在数据库层面保证正确性，避免应用层逻辑漏洞。

### 限流器实战

API 限流是 Lua 脚本最经典的应用场景。以下是一个滑动窗口限流器的实现：

```lua
-- rate_limiter.lua
-- 滑动窗口限流器

local key = KEYS[1]          -- 限流 key，如 "ratelimit:api:/users"
local limit = tonumber(ARGV[1])  -- 允许的最大请求数
local window = tonumber(ARGV[2]) -- 时间窗口（秒）

local now = tonumber(ARGV[3])    -- 当前时间戳（秒）
local requested = 1               -- 当前请求计数

-- 移除窗口外的请求记录
local window_start = now - window
redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)

-- 获取当前窗口内的请求数
local current = redis.call('ZCARD', key)

if current < limit then
    -- 未超限，记录请求
    redis.call('ZADD', key, now, now .. ':' .. math.random())
    redis.call('EXPIRE', key, window)
    return {1, limit - current - 1}  -- 允许，返回剩余额度
else
    -- 超限
    return {0, 0}  -- 拒绝
end
```

```python
# Python 调用 Lua 脚本
import redis
import time

r = redis.Redis()

# 加载 Lua 脚本
rate_limiter_script = r.register_script("""
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

local window_start = now - window
redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)

local current = redis.call('ZCARD', key)

if current < limit then
    redis.call('ZADD', key, now, now .. ':' .. math.random())
    redis.call('EXPIRE', key, window)
    return {1, limit - current - 1}
else
    return {0, 0}
end
""")

# 使用限流器
def check_rate_limit(api_path: str, limit: int = 60, window: int = 60):
    key = f"ratelimit:api:{api_path}"
    result = rate_limiter_script(
        keys=[key],
        args=[limit, window, int(time.time())]
    )
    allowed = bool(result[0])
    remaining = int(result[1])
    return {"allowed": allowed, "remaining": remaining}
```

### 分布式锁实战

```lua
-- distributed_lock.lua
-- Redis 分布式锁实现

local lock_key = KEYS[1]
local lock_value = ARGV[1]  -- 唯一标识（如 UUID）
local ttl = tonumber(ARGV[2])  -- 锁过期时间（毫秒）

-- 尝试获取锁
local acquired = redis.call('SET', lock_key, lock_value, 'NX', 'PX', ttl)

if acquired then
    return 1  -- 获取成功
else
    return 0  -- 获取失败
end
```

```python
import redis
import uuid
import time

class DistributedLock:
    def __init__(self, redis_client: redis.Redis, lock_name: str, ttl: int = 30000):
        self.r = redis_client
        self.lock_name = f"lock:{lock_name}"
        self.lock_value = str(uuid.uuid4())
        self.ttl = ttl
        self._locked = False
    
    def acquire(self, blocking: bool = False, timeout: int = 10) -> bool:
        """获取锁"""
        end_time = time.time() + timeout
        
        while True:
            if self.r.set(
                self.lock_name,
                self.lock_value,
                nx=True,
                px=self.ttl
            ):
                self._locked = True
                return True
            
            if not blocking or time.time() >= end_time:
                return False
            
            time.sleep(0.01)  # 短暂等待后重试
    
    def release(self) -> bool:
        """释放锁（只能释放自己的锁）"""
        if not self._locked:
            return False
        
        # 使用 Lua 脚本确保原子性
        script = """
        if redis.call('get', KEYS[1]) == ARGV[1] then
            return redis.call('del', KEYS[1])
        else
            return 0
        end
        """
        result = self.r.eval(script, 1, self.lock_name, self.lock_value)
        self._locked = False
        return bool(result)
    
    def __enter__(self):
        self.acquire()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.release()

# 使用分布式锁
lock = DistributedLock(r, "order-processing")
if lock.acquire():
    try:
        # 临界区代码
        process_order()
    finally:
        lock.release()
else:
    print("无法获取锁，稍后重试")
```

---

## Redis + Node.js：ioredis 与 node-redis

### ioredis 使用指南

ioredis 是 Node.js 中最流行的 Redis 客户端，提供了完整的 Redis 命令支持和丰富的功能。

```javascript
import Redis from 'ioredis'

// 基本连接
const redis = new Redis({
  host: 'localhost',
  port: 6379,
  password: 'yourpassword',
  db: 0,
  retryStrategy: (times) => {
    const delay = Math.min(times * 50, 2000)
    return delay
  }
})

// 连接池（集群模式）
const cluster = new Redis.Cluster([
  { host: '127.0.0.1', port: 7000 },
  { host: '127.0.0.1', port: 7001 },
  { host: '127.0.0.1', port: 7002 }
])

// String 操作
await redis.set('user:1001', 'Alice')
await redis.get('user:1001')
await redis.setex('session:abc', 3600, 'data')
await redis.setnx('user:1002', 'Bob')

// 计数器
await redis.set('counter', 100)
await redis.incr('counter')           // 101
await redis.incrby('counter', 10)     // 111

// Hash 操作
await redis.hset('user:1001', {
  name: 'Alice',
  email: 'alice@example.com',
  age: 30
})
const user = await redis.hgetall('user:1001')

// List 操作（任务队列）
await redis.lpush('tasks', 'task1', 'task2')
await redis.rpush('tasks', 'task3')
const tasks = await redis.lrange('tasks', 0, -1)
const task = await redis.lpop('tasks')

// Set 操作（标签）
await redis.sadd('tags:article:1', 'redis', 'cache', 'database')
const tags = await redis.smembers('tags:article:1')
const hasTag = await redis.sismember('tags:article:1', 'redis')

// Sorted Set 操作（排行榜）
await redis.zadd('leaderboard', 100, 'Alice', 90, 'Bob', 80, 'Charlie')
const top10 = await redis.zrevrange('leaderboard', 0, 9, 'WITHSCORES')
const rank = await redis.zrevrank('leaderboard', 'Alice')

// Stream 操作（消息队列）
await redis.xadd('events', '*', 'type', 'click', 'page', '/home')
const messages = await redis.xread('STREAMS', 'events', '0-0')

// 发布订阅
const subscriber = new Redis()
subscriber.subscribe('channel1', 'channel2')
subscriber.on('message', (channel, message) => {
  console.log(`${channel}: ${message}`)
})
await redis.publish('channel1', 'Hello World')

// 管道（批量操作）
const pipeline = redis.pipeline()
pipeline.set('key1', 'value1')
pipeline.get('key1')
pipeline.hset('user:1001', 'name', 'Alice')
const results = await pipeline.exec()

// 事务
const multi = redis.multi()
multi.set('key1', 'value1')
multi.get('key1')
const txResults = await multi.exec()

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
`
const result = await redis.eval(rateLimitScript, 1, 'ratelimit:user:1', 10, 60)

// 错误处理
redis.on('error', (err) => {
  console.error('Redis 连接错误:', err)
})

redis.on('connect', () => {
  console.log('已连接到 Redis')
})

// 关闭连接
await redis.quit()
```

---

## Redis + Python：redis-py 与实战

### redis-py 使用指南

redis-py 是 Python 中最成熟的 Redis 客户端，提供了同步和异步两种接口。

```python
import redis
from redis import ConnectionPool, Redis
import json
from datetime import timedelta

# 基本连接
r = redis.Redis(
    host='localhost',
    port=6379,
    db=0,
    password=None,
    decode_responses=True  # 自动解码响应
)

# 连接池（推荐）
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
r.setex('session:abc', 3600, 'data')  # 1小时后过期
r.setnx('user:1002', 'Bob')  # 仅当不存在时设置

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

# List 操作（队列）
r.lpush('tasks', 'task1', 'task2')
r.rpush('tasks', 'task3')
r.lrange('tasks', 0, -1)
r.lpop('tasks')
r.rpop('tasks')

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
r.zrank('leaderboard', 'Alice')
r.zincrby('leaderboard', 10, 'Bob')

# 发布订阅
pubsub = r.pubsub()
pubsub.subscribe('channel1', 'channel2')
pubsub.psubscribe('pattern*')

for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"收到消息: {message['channel']}: {message['data']}")

# 管道（批量操作，减少 RTT）
pipe = r.pipeline()
pipe.set('key1', 'value1')
pipe.get('key1')
pipe.hset('user:1001', 'name', 'Alice')
pipe.execute()

# 管道 + 事务
pipe = r.pipeline(transaction=True)
pipe.watch('key1')  # 监视 key
pipe.multi()  # 开始事务
pipe.set('key1', 'new_value')
pipe.execute()  # 如果 key1 被修改，这里会抛出异常

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
result = r.eval(script, 1, 'ratelimit:user:1', 10, 60)

# Scan（安全遍历 keys）
cursor = 0
while True:
    cursor, keys = r.scan(cursor, match='user:*', count=100)
    for key in keys:
        # 处理 key
        pass
    if cursor == 0:
        break

# Stream 操作
r.xadd('events', {'type': 'click', 'page': '/home'})
r.xlen('events')
messages = r.xrange('events', '-', '+')

# 创建消费者组
try:
    r.xgroup_create('events', 'processors', id='0', mkstream=True)
except redis.exceptions.ResponseError:
    pass  # 组已存在

# 读取消费者组
messages = r.xreadgroup('processors', 'consumer1', {'events': '>'}, count=10)
for stream, msgs in messages:
    for msg_id, data in msgs:
        print(f"处理消息: {data}")
        r.xack('events', 'processors', msg_id)
```

### LLM 缓存实战

以下是 Redis 在 AI 应用中的实战代码，展示了如何缓存 LLM 响应：

```python
import redis
import hashlib
import json
from datetime import timedelta

r = redis.Redis(decode_responses=True)

class LLMCache:
    """LLM 响应缓存"""
    
    def __init__(self, redis_client: redis.Redis):
        self.r = redis_client
    
    def _hash_prompt(self, model: str, messages: list) -> str:
        """生成 prompt 的哈希值"""
        content = json.dumps({
            "model": model,
            "messages": messages
        }, sort_keys=True)
        return hashlib.sha256(content.encode()).hexdigest()
    
    def get(self, model: str, messages: list) -> str | None:
        """获取缓存的 LLM 响应"""
        key = self._hash_prompt(model, messages)
        return self.r.get(f"llm:response:{key}")
    
    def set(
        self,
        model: str,
        messages: list,
        response: str,
        ttl: timedelta = timedelta(hours=24)
    ):
        """缓存 LLM 响应"""
        key = self._hash_prompt(model, messages)
        self.r.setex(f"llm:response:{key}", ttl, response)
    
    def invalidate_model(self, model: str):
        """失效指定模型的所有缓存"""
        pattern = "llm:response:*"
        cursor = 0
        while True:
            cursor, keys = self.r.scan(cursor, match=pattern, count=100)
            if keys:
                self.r.delete(*keys)
            if cursor == 0:
                break


class TokenTracker:
    """Token 使用量追踪"""
    
    def __init__(self, redis_client: redis.Redis):
        self.r = redis_client
    
    def track(self, user_id: str, tokens: int):
        """记录 Token 使用"""
        today = datetime.now().strftime('%Y-%m-%d')
        month = datetime.now().strftime('%Y-%m')
        
        pipe = self.r.pipeline()
        
        # 今日用量
        pipe.incrby(f"tokens:{user_id}:daily:{today}", tokens)
        pipe.expire(f"tokens:{user_id}:daily:{today}", 86400 * 2)
        
        # 本月用量
        pipe.zincrby(f"tokens:{user_id}:monthly:{month}", tokens, today)
        pipe.expire(f"tokens:{user_id}:monthly:{month}", 86400 * 35)
        
        pipe.execute()
    
    def get_daily_usage(self, user_id: str, date: str = None) -> int:
        """获取指定日期的 Token 用量"""
        date = date or datetime.now().strftime('%Y-%m-%d')
        return int(self.r.get(f"tokens:{user_id}:daily:{date}") or 0)
    
    def get_monthly_usage(self, user_id: str, month: str = None) -> int:
        """获取指定月份的 Token 用量"""
        month = month or datetime.now().strftime('%Y-%m')
        results = self.r.zrange(
            f"tokens:{user_id}:monthly:{month}",
            0, -1,
            withscores=True
        )
        return sum(int(score) for _, score in results)
```

---

## API 限流实现

限流是保护 API 免受滥用和 DDoS 攻击的关键机制。以下是几种常见的限流实现方案。

### 固定窗口限流

固定窗口限流是最简单的方案：将时间划分为固定长度的窗口（如每分钟），每个窗口有独立的计数器。

```python
def rate_limit_fixed_window(user_id: str, limit: int = 60, window: int = 60) -> bool:
    """固定窗口限流"""
    key = f"ratelimit:fixed:{user_id}"
    now = int(time.time())
    window_start = now - (now % window)
    
    pipe = r.pipeline()
    pipe.incr(key)
    pipe.expireat(key, window_start + window)
    count = pipe.execute()[0]
    
    return count <= limit
```

固定窗口的问题是「边界突变」：如果在窗口末尾和下一个窗口开始时都有大量请求，可能会导致实际请求量是限制的两倍。

### 滑动窗口限流

滑动窗口限流解决了固定窗口的边界问题。以下是实现：

```python
def rate_limit_sliding_window(user_id: str, limit: int = 60, window: int = 60) -> bool:
    """滑动窗口限流"""
    key = f"ratelimit:sliding:{user_id}"
    now = time.time()
    window_start = now - window
    
    pipe = r.pipeline()
    # 移除窗口外的记录
    pipe.zremrangebyscore(key, '-inf', window_start)
    # 添加当前请求
    pipe.zadd(key, {f"{now}:{random.random()}": now})
    # 设置过期
    pipe.expire(key, window + 1)
    # 获取当前请求数
    count = pipe.zcard(key)
    pipe.execute()
    
    return count <= limit
```

### 令牌桶限流

令牌桶算法允许一定程度的突发流量，同时保持长期平均速率。

```python
def rate_limit_token_bucket(user_id: str, capacity: int = 60, refill_rate: float = 1.0) -> bool:
    """令牌桶限流"""
    key = f"ratelimit:bucket:{user_id}"
    now = time.time()
    
    # Lua 脚本保证原子性
    script = """
    local key = KEYS[1]
    local capacity = tonumber(ARGV[1])
    local refill_rate = tonumber(ARGV[2])
    local now = tonumber(ARGV[3])
    
    local bucket = redis.call('HMGET', key, 'tokens', 'last_refill')
    local tokens = tonumber(bucket[1]) or capacity
    local last_refill = tonumber(bucket[2]) or now
    
    -- 补充令牌
    local elapsed = now - last_refill
    tokens = math.min(capacity, tokens + elapsed * refill_rate)
    
    if tokens >= 1 then
        tokens = tokens - 1
        redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
        redis.call('EXPIRE', key, 3600)
        return 1
    else
        redis.call('HMSET', key, 'tokens', tokens, 'last_refill', now)
        redis.call('EXPIRE', key, 3600)
        return 0
    end
    """
    
    result = r.eval(script, 1, key, capacity, refill_rate, now)
    return bool(result)
```

---

## 会话管理实战

Redis 是会话存储的理想选择——它比 Memcached 更强大（支持更多数据类型），比数据库更快，比磁盘文件更可靠。

```python
from datetime import timedelta

class SessionManager:
    """基于 Redis 的会话管理"""
    
    def __init__(self, redis_client: redis.Redis, prefix: str = "session:"):
        self.r = redis_client
        self.prefix = prefix
    
    def create_session(self, user_id: str, data: dict = None, ttl: int = 86400) -> str:
        """创建会话"""
        import uuid
        session_id = str(uuid.uuid4())
        key = f"{self.prefix}{session_id}"
        
        session_data = {
            'user_id': user_id,
            'created_at': str(int(time.time()))
        }
        if data:
            session_data.update(data)
        
        pipe = self.r.pipeline()
        pipe.hset(key, mapping=session_data)
        pipe.expire(key, ttl)
        pipe.execute()
        
        return session_id
    
    def get_session(self, session_id: str) -> dict | None:
        """获取会话数据"""
        key = f"{self.prefix}{session_id}"
        return self.r.hgetall(key)
    
    def update_session(self, session_id: str, data: dict, extend_ttl: int = None):
        """更新会话数据"""
        key = f"{self.prefix}{session_id}"
        pipe = self.r.pipeline()
        pipe.hset(key, mapping=data)
        if extend_ttl:
            pipe.expire(key, extend_ttl)
        pipe.execute()
    
    def delete_session(self, session_id: str):
        """删除会话（登出）"""
        key = f"{self.prefix}{session_id}"
        self.r.delete(key)
    
    def refresh_session(self, session_id: str, ttl: int = 86400):
        """刷新会话过期时间"""
        key = f"{self.prefix}{session_id}"
        self.r.expire(key, ttl)
    
    def is_valid(self, session_id: str) -> bool:
        """检查会话是否有效"""
        key = f"{self.prefix}{session_id}"
        return self.r.exists(key) > 0


# Flask 集成示例
from flask import Flask, request, session, jsonify
import redis

app = Flask(__name__)

r = redis.Redis(decode_responses=True)
session_mgr = SessionManager(r)

@app.route('/login', methods=['POST'])
def login():
    user_id = authenticate(request.json)
    if user_id:
        session_id = session_mgr.create_session(user_id, {'ip': request.remote_addr})
        return jsonify({'session_id': session_id})
    return jsonify({'error': '认证失败'}), 401

@app.route('/logout', methods=['POST'])
def logout():
    session_id = request.headers.get('X-Session-ID')
    session_mgr.delete_session(session_id)
    return jsonify({'success': True})

@app.route('/profile')
def profile():
    session_id = request.headers.get('X-Session-ID')
    sess = session_mgr.get_session(session_id)
    if not sess:
        return jsonify({'error': '未登录'}), 401
    return jsonify({'user_id': sess['user_id']})
```

---

## Redis Cluster 与 Sentinel 高可用

### Redis Cluster 架构

Redis Cluster 是 Redis 官方提供的分布式解决方案，支持数据分片和自动故障转移。它将整个数据集划分为 16384 个槽位（slot），分布在多个主节点上。

```
┌─────────────────────────────────────────────────────────────┐
│                    Redis Cluster                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐                   │
│  │Master 1 │   │Master 2 │   │Master 3 │                   │
│  │ P:7000  │   │ P:7001  │   │ P:7002  │                   │
│  │Slot:0-5460│  │Slot:5461-│  │Slot:10923│                 │
│  │         │   │  10922   │  │ -16383  │                   │
│  └────┬────┘   └────┬────┘   └────┬────┘                   │
│       │            │            │                          │
│  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐                   │
│  │Replica1 │  │Replica2 │  │Replica3 │                   │
│  │ P:8000  │  │ P:8001  │  │ P:8002  │                   │
│  │ (从节点) │  │ (从节点) │  │ (从节点) │                   │
│  └─────────┘  └─────────┘  └─────────┘                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

Redis Cluster 的核心特性包括：自动分片（无需手动路由）、去中心化（无单点故障）、自动故障转移（从节点自动晋升）、在线扩容缩容。

```python
from redis.cluster import RedisCluster

# 创建集群客户端
rc = RedisCluster(
    host='127.0.0.1',
    port=7000,
    decode_responses=True
)

# 基本操作（自动路由）
rc.set('key1', 'value1')
rc.get('key1')

# 批量操作
pipe = rc.pipeline()
pipe.set('key1', 'value1')
pipe.get('key1')
pipe.hset('hash1', 'field1', 'value1')
pipe.execute()
```

### Redis Sentinel 高可用

Sentinel 是 Redis 的高可用解决方案，适合不想使用 Cluster 的场景。Sentinel 监控主从节点的健康状态，在主节点故障时自动进行故障转移，将从节点晋升为新的主节点。

```python
from redis.sentinel import Sentinel

# 创建 Sentinel
sentinel = Sentinel([
    ('127.0.0.1', 26379),
    ('127.0.0.1', 26380),
    ('127.0.0.1', 26381)
], socket_timeout=0.1)

# 获取主节点（自动故障转移）
master = sentinel.master_for('mymaster', socket_timeout=0.1, decode_responses=True)

# 获取从节点（用于读操作）
slave = sentinel.slave_for('mymaster', socket_timeout=0.1, decode_responses=True)

# 使用主节点写入
master.set('key', 'value')

# 使用从节点读取
value = slave.get('key')
```

### 选型建议

| 场景 | 推荐方案 |
|------|---------|
| 小规模应用（< 10GB 数据） | 单机 + RDB/AOF 持久化 |
| 中等规模（需要高可用） | Sentinel 主从架构 |
| 大规模应用（需要分片） | Redis Cluster |
| 需要水平扩展的开发测试 | Redis Cluster |

---

## 内存优化与最佳实践

### 内存优化技巧

Redis 的内存使用是需要重点关注的方面。以下是几个关键的优化策略。

**选择合适的数据结构**。对于只需要快速查找且不需要范围操作的场景，用 Hash 而非 String 存储多个字段通常更省内存。例如，存储 1000 个用户信息，用 1000 个 String（每个用户一个 key）比用 1 个 Hash（所有用户在一个 key 下）消耗更多内存，因为每个 String key 都有固定的开销。

**启用内存压缩**。Redis 6.0+ 支持 `zstd` 和 `lz4` 压缩，适用于 Value 较大的场景。通过 `CONFIG SET stream-node-max-bytes` 等配置可以启用压缩。

**使用整数优化**。Redis 对小整数有特殊优化（共享对象池），使用整数而非字符串可以节省内存。

**合理设置过期时间**。为所有缓存数据设置过期时间，避免无用的数据占用内存。Redis 的惰性删除和定期删除机制会自动清理过期数据，但在高并发场景下可能不够及时。

**监控内存使用**。定期检查 `INFO memory` 命令的输出，关注 `used_memory_human`（已用内存）、`maxmemory`（配置的最大内存）、`mem_fragmentation_ratio`（内存碎片率）等指标。

```bash
# 查看内存使用详情
INFO memory

# 输出示例：
# used_memory:1048576
# used_memory_human:1.00M
# used_memory_rss:1258291
# used_memory_rss_human:1.20M
# mem_fragmentation_ratio:1.20
# maxmemory:2147483648
# maxmemory_human:2.00G
```

### 常见陷阱

**大 Key 问题**。一个 Value 超过几 MB 的 Key 会导致 RDB/AOF 持久化变慢、复制带宽增加、主从同步延迟等系列问题。使用 `redis-cli --bigkeys` 可以找出占用内存最多的 Key。

**KEYS 命令**。`KEYS *` 命令会遍历所有 Key，在生产环境中可能阻塞 Redis 数十秒。始终使用 `SCAN` 命令代替 `KEYS`。

**没有设置过期时间**。忘记设置过期时间会导致数据永远驻留内存，直到 Redis 重启或内存满触发淘汰策略。

---

## Redis vs Memcached vs Dragonfly 对比

### 核心对比

| 维度 | Redis | Memcached | Dragonfly |
|------|-------|-----------|-----------|
| **数据结构** | 丰富（7种） | 仅 String | 丰富（7种） |
| **持久化** | 支持（RDB/AOF） | 不支持 | 支持（RDB/AOF） |
| **复制** | 主从/Cluster | 无 | 主从/Cluster |
| **多核支持** | 有限（6.0+ 多线程 I/O） | 有限 | 原生（多线程架构） |
| **内存效率** | 一般 | 高 | 高（节省 30%+） |
| **许可** | SSPL/商业 | BSD | Apache 2.0 |
| **向量搜索** | Redis Stack | 不支持 | 开发中 |
| **社区生态** | 成熟庞大 | 一般 | 快速增长 |

### Dragonfly 的优势

Dragonfly 是 Redis 的高性能替代品，由 Cloudflare 前工程师开发。它采用多线程架构，可以充分利用多核 CPU 处理器，在某些基准测试中性能是 Redis 的 25 倍以上。

Dragonfly 的主要优势包括：原生多线程（无需 Cluster 即可横向扩展）、内存效率更高（采用更先进的内存管理算法）、兼容 Redis 协议（可以无缝替换 Redis）、支持更多高级特性（更好的 Lua 脚本性能、更强大的持久化选项）。

Dragonfly 的适用场景包括：需要单机高性能的场景、内存资源宝贵的场景、不需要 Redis Cluster 的场景。但 Dragonfly 的集群支持仍在完善中，对于需要大规模分布式部署的场景，Redis Cluster 仍然是更成熟的选择。

### 选型建议

| 场景 | 推荐选择 | 原因 |
|------|---------|------|
| **通用缓存** | Redis | 生态最成熟 |
| **高性能单节点** | Dragonfly | 多核性能最优 |
| **需要持久化** | Redis | RDB/AOF 更稳定 |
| **AI 向量搜索** | Redis Stack | 完整支持 |
| **超简单 KV 缓存** | Memcached | 内存效率高 |
| **Windows 环境** | Memurai | 原生支持 |

---

## 选型建议与适用场景

### Redis 最佳场景

**缓存层**是 Redis 最经典的应用。无论是 API 响应缓存、数据库查询缓存、页面片段缓存还是静态资源缓存，Redis 都能提供极快的读写性能。配合过期时间和淘汰策略，可以构建一个自动管理的智能缓存层。

**会话存储**是 Redis 的另一个杀手级应用。相比将 Session 存储在 Cookie 中（安全性问题）或磁盘数据库中（性能问题），将 Session 存储在 Redis 中既有安全性（用户无法伪造），又有高性能（毫秒级读写）。

**实时排行榜**利用 Sorted Set 的自动排序能力，可以轻松实现游戏积分榜、用户活跃榜、内容热度榜等。ZADD 更新分数、ZREVRANGE 获取 Top N、ZREVRANK 查询排名——这些操作都是 O(log N) 的，性能优异。

**消息队列**可以用 List 或 Stream 实现。List 的 LPUSH/BRPOP 组合是经典的「可靠队列」模式，Stream 则提供了消费者组、消息确认等企业级特性。

**限流器**利用 Redis 的原子操作和过期时间，可以实现精确的 API 限流。固定窗口、滑动窗口、令牌桶——各种限流算法都可以用 Redis 实现。

**实时分析**可以用 HyperLogLog 统计 UV、用 Bitmap 统计日活、用 Sorted Set 统计排行榜——这些都是 Redis 独有的优势。

**AI 应用**在前文已经详细介绍——LLM 缓存、Token 计数、对话历史、向量搜索，Redis 已经成为 AI 应用架构中不可或缺的组件。

### 何时不用 Redis

**需要存储超过内存容量的数据**。Redis 的数据必须能装进内存，如果你的数据量达到 TB 级别，应该选择磁盘数据库（如 PostgreSQL、Cassandra）或分布式存储（如 HDFS、S3）。

**需要复杂查询能力**。Redis 只支持 Key-Value 查询和简单的范围操作。如果需要 SQL 查询、JOIN 操作、复杂的聚合分析，应该使用传统数据库，Redis 作为缓存层。

**数据需要严格事务保证**。虽然 Redis 支持事务（MULTI/EXEC），但它不支持回滚（Rollback），且不支持跨节点的事务。如果需要 ACID 事务保证，应该使用 PostgreSQL 等支持 MVCC 的数据库。

---

> [!SUCCESS]
> Redis 不仅仅是一个缓存，它是一个功能完备的内存数据平台。掌握 Redis 的 7 种数据结构、了解何时用 Redis 何时用其他数据库、熟悉 Lua 脚本和持久化机制——这些知识将帮助你在现代应用架构中做出更好的技术决策。在 AI 时代，Redis 的 LLM 缓存、向量化支持和实时处理能力更显得尤为重要。

---

## 附录：常用资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://redis.io/docs |
| Redis 命令参考 | https://redis.io/commands |
| Redis University | https://university.redis.com |
| Redis 人门教程 | https://redis.io/docs/getting-started |
| Redis Stack 下载 | https://redis.io/docs/install/stack/ |
