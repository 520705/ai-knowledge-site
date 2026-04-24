# MongoDB 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 MongoDB 8.0 新特性、文档模型设计、Aggregation Pipeline、向量化搜索，以及在 AI 应用中的实战场景。

---

## 目录

1. [[#MongoDB 概述与核心定位]]
2. [[#MongoDB 8.0 新特性]]
3. [[#文档模型与 Schema 设计]]
4. [[#Aggregation Pipeline]]
5. [[#MongoDB Atlas 云服务]]
6. [[#MongoDB vs PostgreSQL vs MySQL 对比]]
7. [[#Atlas AI Search 向量搜索]]
8. [[#Mongoose ODM]]
9. [[#实战场景与代码示例]]
10. [[#选型建议]]

---

## MongoDB 概述与核心定位

### 为什么选择 MongoDB

MongoDB 是一个基于文档模型的 NoSQL 数据库，于 2009 年由 MongoDB Inc. 发布。它以 JSON 格式存储数据，提供了灵活的模式和强大的查询能力。

MongoDB 的核心优势体现在三个维度：

**模式灵活**：MongoDB 的文档模型允许同一集合中的文档拥有不同的字段，无需预定义 Schema。这对于需求变化快速的应用尤为有利。

**水平扩展**：MongoDB 原生支持分片（Sharding），可以轻松扩展到海量数据和高并发场景。

**开发效率**：JSON 文档结构与 JavaScript 天然契合，前端开发者学习曲线平缓。

### MongoDB 在 AI 时代的价值

在 AI 应用中，MongoDB 展现了独特的优势：

- **灵活的数据结构**：AI 输出格式不固定，MongoDB 的灵活 Schema 完美适配
- **向量搜索**：Atlas Search 支持向量相似度搜索
- **实时能力**：Change Streams 支持实时数据处理
- **文档原生**：与 AI Prompt、Response 的 JSON 结构完美匹配

### 市场份额与行业地位

MongoDB 在 NoSQL 和文档数据库市场中占据主导地位：

| 排名 | 数据库 | 市场份额（2025） | 趋势 |
|------|--------|------------------|------|
| 1 | MongoDB | 47.8% | ↑ |
| 2 | DynamoDB | 18.5% | → |
| 3 | Couchbase | 8.2% | ↓ |
| 4 | Cassandra | 7.1% | → |
| 5 | Cosmos DB | 6.3% | ↑ |

**关键数据点**：
- MongoDB 是全球最流行的文档数据库
- 超过 40% 的新 Web 应用选择 MongoDB
- 主要使用者包括 eBay、Adobe、Verizon、EA Games 等知名企业
- MongoDB Atlas 云服务托管了数百万数据库实例
- MongoDB Community 下载量超过 1 亿次

**MongoDB 适用场景**：

| 场景 | 推荐程度 | 说明 |
|------|----------|------|
| 内容管理系统 | ⭐⭐⭐⭐⭐ | 灵活的内容结构 |
| 物联网数据 | ⭐⭐⭐⭐⭐ | 时序数据存储 |
| 实时分析 | ⭐⭐⭐⭐ | 高吞吐量写入 |
| 移动应用 | ⭐⭐⭐⭐ | 离线优先架构 |
| AI 数据存储 | ⭐⭐⭐⭐ | 灵活的数据结构 |

---

## 核心概念与数据模型

### MongoDB 架构

MongoDB 采用分布式架构设计：

```
┌─────────────────────────────────────────────────────────────┐
│                    MongoDB 集群架构                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   应用层                                                      │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│   │  驱动    │  │ Shell    │  │ Compass │               │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘               │
│        │               │               │                       │
│        └───────────────┼───────────────┘                       │
│                        ▼                                        │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                  Router (mongos)                     │   │
│   │                  查询路由 + 分片协调                   │   │
│   └─────────────────────────────────────────────────────┘   │
│                        │                                        │
│        ┌───────────────┼───────────────┐                     │
│        ▼               ▼               ▼                     │
│   ┌─────────┐     ┌─────────┐     ┌─────────┐             │
│   │ Shard 1 │     │ Shard 2 │     │ Shard 3 │             │
│   │(副本集)  │     │(副本集)  │     │(副本集)  │             │
│   │ ┌─────┐ │     │ ┌─────┐ │     │ ┌─────┐ │             │
│   │ │ P   │ │     │ │ P   │ │     │ │ P   │ │             │
│   │ ├─────┤ │     │ ├─────┤ │     │ ├─────┤ │             │
│   │ │ S1  │ │     │ │ S1  │ │     │ │ S1  │ │             │
│   │ ├─────┤ │     │ ├─────┤ │     │ ├─────┤ │             │
│   │ │ S2  │ │     │ │ S2  │ │     │ │ S2  │ │             │
│   │ └─────┘ │     │ └─────┘ │     │ └─────┘ │             │
│   └─────────┘     └─────────┘     └─────────┘             │
│                                                              │
│   ┌─────────────────────────────────────────────────────┐   │
│   │              Config Server (副本集)                   │   │
│   │           存储集群元数据和配置信息                     │   │
│   └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 核心概念

#### 1. 文档模型

MongoDB 的基本数据单元是文档，类似 JSON 对象：

```javascript
// 文档示例
{
  _id: ObjectId("..."),
  title: "MongoDB 入门",
  content: "这是一篇关于 MongoDB 的教程...",
  author: {
    name: "Alice",
    email: "alice@example.com",
    bio: "技术作家"
  },
  tags: ["mongodb", "database", "nosql"],
  stats: {
    views: 1000,
    likes: 50,
    shares: 10
  },
  comments: [
    { user: "Bob", text: "很实用的文章！", createdAt: ISODate() },
    { user: "Carol", text: "期待更多内容", createdAt: ISODate() }
  ],
  createdAt: ISODate(),
  updatedAt: ISODate()
}
```

#### 2. 集合（Collection）

集合是文档的容器，类似于关系型数据库的表：

```javascript
// 创建集合（隐式创建）
db.articles.insertOne({ title: "Hello" })

// 创建带验证规则的集合
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["email", "password"],
      properties: {
        email: {
          bsonType: "string",
          pattern: "^.+@.+$"
        },
        password: {
          bsonType: "string",
          minLength: 8
        }
      }
    }
  }
})

// 查看集合
show collections

// 删除集合
db.users.drop()
```

#### 3. 数据库

```javascript
// 切换数据库
use mydb

// 创建数据库（实际上创建第一个集合时才创建）
db.createDatabase = function() { return "use mydb, then insert data" }

// 查看所有数据库
show dbs

// 删除数据库
db.dropDatabase()
```

### 数据模型设计

#### 1. 嵌入模式（一对一）

```javascript
// 用户文档嵌入地址
{
  _id: ObjectId(),
  name: "Alice",
  email: "alice@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    state: "NY",
    zip: "10001",
    country: "USA"
  }
}
```

#### 2. 引用模式（一对多）

```javascript
// 订单文档引用产品
// orders 集合
{
  _id: ObjectId(),
  orderNumber: "ORD-001",
  customerId: ObjectId("..."),
  items: [
    { productId: ObjectId("..."), quantity: 2, price: 29.99 },
    { productId: ObjectId("..."), quantity: 1, price: 49.99 }
  ],
  total: 109.97,
  status: "completed"
}

// products 集合
{
  _id: ObjectId(),
  name: "Widget",
  description: "A useful widget",
  price: 29.99,
  stock: 100
}
```

#### 3. 桶模式（时间序列）

```javascript
// 传感器数据桶
{
  _id: ObjectId(),
  sensorId: "sensor-001",
  bucketStart: ISODate("2024-01-15T00:00:00Z"),
  bucketEnd: ISODate("2024-01-15T01:00:00Z"),
  readings: [
    { time: ISODate("2024-01-15T00:00:00Z"), temp: 22.5, humidity: 65 },
    { time: ISODate("2024-01-15T00:05:00Z"), temp: 22.6, humidity: 64 },
    // ... 最多 60 条记录
  ],
  stats: {
    minTemp: 22.0,
    maxTemp: 23.5,
    avgTemp: 22.8,
    count: 60
  }
}
```

#### 4. 血统模式（层级结构）

```javascript
// 分类树
{
  _id: ObjectId(),
  name: "Electronics",
  slug: "electronics",
  parentId: null,
  path: "/electronics",
  level: 0
}

{
  _id: ObjectId(),
  name: "Smartphones",
  slug: "smartphones",
  parentId: ObjectId("..."), // 指向 Electronics
  path: "/electronics/smartphones",
  level: 1
}

// 查询所有后代
db.categories.aggregate([
  {
    $graphLookup: {
      from: "categories",
      startWith: "$_id",
      connectFromField: "_id",
      connectToField: "parentId",
      as: "descendants",
      depthField: "depth"
    }
  }
])
```

---

---

## MongoDB 8.0 新特性

MongoDB 8.0 于 2025 年发布，带来了性能优化和新功能。

### 核心新特性

#### 1. 性能大幅提升

| 优化项 | 提升幅度 |
|--------|----------|
| **查询性能** | 提升 40% |
| **写入吞吐量** | 提升 50% |
| **分片平衡** | 提升 60% |
| **索引构建** | 提升 45% |
| **内存使用** | 减少 30% |

#### 2. 增强的聚合管道

```javascript
// 新的聚合操作符
db.articles.aggregate([
  {
    $vectorSearch: {  // 向量搜索
      index: "vector_index",
      path: "embedding",
      queryVector: [0.1, 0.2, ...],
      numCandidates: 100,
      limit: 10
    }
  },
  {
    $score: {  // 评分
      score: { $meta: "vectorSearchScore" }
    }
  }
]);

// 新的 $group 增强
db.sales.aggregate([
  {
    $group: {
      _id: { region: "$region", category: "$category" },
      total: { $sum: "$amount" },
      avg: { $avg: { $ifNull: ["$amount", 0] } }
    }
  }
]);

// 新的 $unionWith 增强
db.orders.aggregate([
  { $match: { status: "completed" } },
  {
    $unionWith: {
      coll: "archived_orders",
      pipeline: [
        { $match: { year: 2024 } }
      ]
    }
  }
]);
```

#### 3. 改进的安全功能

```javascript
// 增强的字段级加密
db.users.insertOne({
  name: "Alice",
  ssn: "123-45-6789",  // 自动加密
  creditCard: "4111-1111-1111-1111",  // 自动加密
  _id: ObjectId()
});

// 改进的 RBAC
db.admin.updateUser("admin_user", {
  roles: [
    { role: "readWriteAnyDatabase", db: "admin" },
    { role: "atlasAdmin", db: "admin" }
  ]
});
```

#### 4. 时间序列增强

```javascript
// 时间序列集合改进
db.createCollection("sensor_data", {
  timeseries: {
    timeField: "timestamp",
    metaField: "metadata",
    granularity: "seconds",
    expireAfterSeconds: 2592000  // 30 天自动过期
  }
});

// 聚合优化
db.sensor_data.aggregate([
  {
    $group: {
      _id: {
        sensorId: "$metadata.sensorId",
        interval: {
          $dateTrunc: {
            date: "$timestamp",
            unit: "minute"
          }
        }
      },
      avgValue: { $avg: "$value" },
      maxValue: { $max: "$value" }
    }
  }
]);
```

---

## 文档模型与 Schema 设计

### 文档结构基础

```javascript
// MongoDB 文档示例（BSON 格式）
{
  "_id": ObjectId("..."),
  "title": "My First Article",
  "content": "This is the article content...",
  "author": {
    "name": "Alice",
    "email": "alice@example.com",
    "bio": "Tech writer"
  },
  "tags": ["AI", "Machine Learning", "Python"],
  "stats": {
    "views": 1000,
    "likes": 50,
    "shares": 10
  },
  "comments": [
    {
      "user": "Bob",
      "text": "Great article!",
      "createdAt": ISODate("2024-01-15")
    }
  ],
  "createdAt": ISODate("2024-01-01"),
  "updatedAt": ISODate("2024-01-15"),
  "status": "published",
  "metadata": {  // 灵活字段
    "language": "en",
    "region": "US"
  }
}
```

### Schema 设计模式

#### 1. 嵌入（Embedding）vs 引用（Reference）

```javascript
// 嵌入模式：一对少，数据总是一起访问
// 优点：一次查询获取所有数据
// 缺点：文档过大，数据重复
{
  _id: ObjectId(),
  orderNumber: "ORD-001",
  customer: {
    name: "Alice",
    email: "alice@example.com",
    address: {
      street: "123 Main St",
      city: "New York",
      zip: "10001"
    }
  },
  items: [
    { product: "Widget", quantity: 2, price: 19.99 },
    { product: "Gadget", quantity: 1, price: 49.99 }
  ],
  total: 89.97,
  createdAt: ISODate()
}

// 引用模式：一对多，数据需要单独管理
// 优点：数据规范化，易于更新
// 缺点：需要多次查询或 $lookup

// orders 集合
{
  _id: ObjectId(),
  orderNumber: "ORD-002",
  customerId: ObjectId("..."),  // 引用
  items: [
    { productId: ObjectId("..."), quantity: 2 },  // 引用
    { productId: ObjectId("..."), quantity: 1 }
  ],
  createdAt: ISODate()
}

// customers 集合
{
  _id: ObjectId(),
  name: "Bob",
  email: "bob@example.com"
}

// products 集合
{
  _id: ObjectId(),
  name: "Widget",
  price: 19.99
}

// 使用 $lookup 关联查询
db.orders.aggregate([
  { $match: { orderNumber: "ORD-002" } },
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customer"
    }
  },
  {
    $lookup: {
      from: "products",
      localField: "items.productId",
      foreignField: "_id",
      as: "productDetails"
    }
  }
]);
```

#### 2. 常见设计模式

```javascript
// 扩展模式：灵活添加字段
{
  _id: ObjectId(),
  name: "Product",
  basePrice: 99.99,
  // 可选字段
  salePrice: 79.99,
  discount: { percent: 20, code: "SUMMER24" }
}

// 桶模式：优化范围查询
{
  _id: ObjectId(),
  sensorId: "sensor-001",
  bucketStart: ISODate("2024-01-01T00:00:00Z"),
  bucketEnd: ISODate("2024-01-01T01:00:00Z"),
  readings: [
    { time: ISODate("2024-01-01T00:15:00Z"), value: 22.5 },
    { time: ISODate("2024-01-01T00:30:00Z"), value: 23.0 }
  ],
  stats: {
    min: 22.0,
    max: 23.5,
    avg: 22.8,
    count: 60
  }
}

// 血统模式：处理层级数据
{
  _id: ObjectId(),
  name: "Electronics",
  parentId: null,  // 根节点
  path: "/electronics"
},
{
  _id: ObjectId(),
  name: "Laptops",
  parentId: ObjectId("..."),  // 父节点
  path: "/electronics/laptops"
}
```

### Schema 验证

```javascript
// 创建带有验证规则的集合
db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["email", "password", "createdAt"],
      properties: {
        email: {
          bsonType: "string",
          pattern: "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
        },
        password: {
          bsonType: "string",
          minLength: 8
        },
        age: {
          bsonType: "int",
          minimum: 0,
          maximum: 150
        },
        role: {
          enum: ["user", "admin", "moderator"]
        },
        metadata: {
          bsonType: "object"
        }
      }
    }
  },
  validationLevel: "moderate",  // moderate: 新文档和更新验证
  validationAction: "error"      // error: 拒绝无效文档
});

// 修改验证规则
db.runCommand({
  collMod: "users",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["email"],
      properties: {
        email: { bsonType: "string" }
      }
    }
  }
});
```

---

## Aggregation Pipeline

### 基础管道

```javascript
// 基础过滤和投影
db.articles.aggregate([
  // $match: 过滤文档
  { $match: { status: "published", tags: "AI" } },
  
  // $project: 选择和重命名字段
  { $project: {
      title: 1,
      authorName: "$author.name",
      year: { $year: "$createdAt" },
      _id: 0
    }
  },
  
  // $sort: 排序
  { $sort: { createdAt: -1 } },
  
  // $limit: 限制数量
  { $limit: 10 }
]);
```

### 高级聚合操作符

```javascript
// $group: 分组聚合
db.orders.aggregate([
  { $unwind: "$items" },  // 展开数组
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        category: "$items.category"
      },
      totalRevenue: { $sum: { $multiply: ["$items.price", "$items.quantity"] } },
      orderCount: { $sum: 1 },
      avgOrderValue: { $avg: "$total" },
      topProducts: { $addToSet: "$items.name" }
    }
  },
  { $sort: { "_id.year": -1, "_id.month": -1 } }
]);

// $bucket: 自定义桶分组
db.sales.aggregate([
  {
    $bucket: {
      groupBy: "$amount",
      boundaries: [0, 100, 500, 1000, 5000, Infinity],
      default: "Other",
      output: {
        count: { $sum: 1 },
        orders: { $push: { id: "$_id", amount: "$amount" } }
      }
    }
  }
]);

// $facet: 多维度聚合
db.orders.aggregate([
  {
    $facet: {
      // 按状态统计
      byStatus: [
        { $group: { _id: "$status", count: { $sum: 1 } } }
      ],
      // 按月统计
      byMonth: [
        { $group: {
            _id: { $month: "$createdAt" },
            total: { $sum: "$total" }
          }
        },
        { $sort: { _id: 1 } }
      ],
      // Top 5 客户
      topCustomers: [
        { $group: { _id: "$customerId", total: { $sum: "$total" } } },
        { $sort: { total: -1 } },
        { $limit: 5 }
      ]
    }
  }
]);
```

### 数组操作

```javascript
// $filter: 过滤数组元素
db.orders.aggregate([
  {
    $project: {
      orderId: 1,
      // 只保留状态为 pending 的商品
      pendingItems: {
        $filter: {
          input: "$items",
          as: "item",
          cond: { $eq: ["$$item.status", "pending"] }
        }
      }
    }
  }
]);

// $map: 转换数组元素
db.products.aggregate([
  {
    $project: {
      name: 1,
      // 应用折扣
      discountedPrices: {
        $map: {
          input: "$prices",
          as: "price",
          in: { $multiply: ["$$price", 0.9] }  // 9折
        }
      }
    }
  }
]);

// $reduce: 聚合数组
db.orders.aggregate([
  {
    $project: {
      orderId: 1,
      // 计算总价
      total: {
        $reduce: {
          input: "$items",
          initialValue: 0,
          in: {
            $add: [
              "$$value",
              { $multiply: ["$$this.price", "$$this.quantity"] }
            ]
          }
        }
      }
    }
  }
]);
```

### 关联查询

```javascript
// $lookup: 左连接
db.orders.aggregate([
  {
    $lookup: {
      from: "customers",
      localField: "customerId",
      foreignField: "_id",
      as: "customerInfo"
    }
  },
  { $unwind: "$customerInfo" },  // 展开数组
  {
    $lookup: {
      from: "products",
      let: { itemIds: "$items.productId" },
      pipeline: [
        { $match: { $expr: { $in: ["$_id", "$$itemIds"] } } }
      ],
      as: "productDetails"
    }
  }
]);

// 子查询关联
db.sales.aggregate([
  {
    $lookup: {
      from: "inventory",
      let: { prodId: "$productId", qty: "$quantity" },
      pipeline: [
        {
          $match: {
            $expr: {
              $and: [
                { $eq: ["$productId", "$$prodId"] },
                { $gte: ["$stock", "$$qty"] }
              ]
            }
          }
        }
      ],
      as: "inventoryCheck"
    }
  }
]);
```

---

## MongoDB Atlas 云服务

### Atlas 核心功能

| 功能 | 说明 |
|------|------|
| **Atlas Search** | 全文搜索引擎，基于 Lucene |
| **Atlas Data Lake** | 无 Serverless 分析 |
| **Atlas Vector Search** | AI 向量相似度搜索 |
| **Atlas Triggers** | 事件驱动函数 |
| **Atlas Graph** | 图数据库能力 |
| **Realm** | 移动端同步 |

### Atlas Search 配置

```javascript
// 在 Atlas 中创建搜索索引（使用 Atlas UI 或 CLI）

// Atlas Search 索引定义
{
  "mappings": {
    "dynamic": false,
    "fields": {
      "title": {
        "type": "string",
        "analyzer": "lucene.standard"
      },
      "content": {
        "type": "string",
        "analyzer": "lucene.english"
      },
      "tags": {
        "type": "string",
        "analyzer": "lucene.keyword"
      },
      "metadata": {
        "type": "object"
      }
    }
  },
  "analyzers": [
    {
      "name": "customAnalyzer",
      "tokenizer": {
        "type": "standard"
      },
      "filter": ["lowercase", "porterStem"]
    }
  ]
}

// 使用 $search
db.articles.aggregate([
  {
    $search: {
      index: "default",
      text: {
        query: "machine learning",
        path: ["title", "content", "tags"],
        fuzzy: {
          maxEdits: 1,
          prefixLength: 2
        }
      },
      highlight: {
        path: ["title", "content"]
      }
    }
  },
  {
    $project: {
      title: 1,
      snippet: { $meta: "searchSnippet" },
      score: { $meta: "searchScore" },
      highlights: { $meta: "searchHighlights" }
    }
  }
]);

// 布尔搜索
db.articles.aggregate([
  {
    $search: {
      compound: {
        must: [
          { text: { query: "AI", path: "title" } }
        ],
        should: [
          { text: { query: "Python", path: ["title", "content"] } },
          { text: { query: "TensorFlow", path: ["title", "content"] } }
        ],
        mustNot: [
          { text: { query: "deprecated", path: "content" } }
        ],
        filter: [
          { range: { path: "createdAt", gte: ISODate("2024-01-01") } }
        ]
      }
    }
  }
]);
```

---

## MongoDB vs PostgreSQL vs MySQL 对比

### 核心对比表

| 维度 | MongoDB | PostgreSQL | MySQL |
|------|---------|------------|-------|
| **模型** | 文档型 | 关系型 | 关系型 |
| **Schema** | 灵活（无模式） | 严格（可验证） | 严格 |
| **事务** | 多文档 ACID | 完整 ACID | ACID (InnoDB) |
| **Join** | $lookup | 原生 SQL Join | 原生 SQL Join |
| **查询语言** | MongoDB Query | SQL | SQL |
| **索引** | B-Tree, 全文, 向量 | B-Tree, GiST, GIN, 向量 | B-Tree, 全文, 向量 |
| **扩展性** | 原生分片 | 需要扩展 | 需要扩展 |
| **JSON 支持** | 原生文档 | jsonb | JSON 函数 |
| **向量搜索** | Atlas Vector Search | pgvector | 原生 (9.0) |
| **一致性** | 最终一致/强一致 | 强一致 | 强一致 |
| **学习曲线** | 平缓 | 较陡 | 平缓 |

### 选择建议对照表

| 场景 | 推荐选择 | 原因 |
|------|----------|------|
| **灵活 Schema** | MongoDB ⭐⭐⭐⭐⭐ | 无模式限制 |
| **结构化数据** | PostgreSQL ⭐⭐⭐⭐⭐ | 强类型 + 约束 |
| **高并发简单查询** | MySQL ⭐⭐⭐⭐⭐ | 性能优异 |
| **复杂查询/报表** | PostgreSQL ⭐⭐⭐⭐⭐ | SQL 强大 |
| **AI 向量搜索** | PostgreSQL/MongoDB | pgvector/Atlas |
| **快速原型** | MongoDB ⭐⭐⭐⭐ | 灵活修改 |
| **强类型需求** | PostgreSQL ⭐⭐⭐⭐ | Schema 验证 |
| **分片海量数据** | MongoDB ⭐⭐⭐⭐ | 原生支持 |

---

## Atlas AI Search 向量搜索

### 配置向量搜索

```javascript
// 在 Atlas UI 创建向量搜索索引
// 索引定义示例
{
  "fields": [
    {
      "path": "embedding",
      "type": "vector",
      "similarity": "cosine",  // cosine, euclidean, dotProduct
      "dimensions": 1536
    },
    {
      "path": "content",
      "type": "string"
    },
    {
      "path": "metadata.category",
      "type": "string"
    }
  ]
}

// 插入带嵌入向量的文档
db.documents.insertOne({
  content: "PostgreSQL is a powerful open-source database.",
  embedding: [0.123, -0.456, ...],  // 1536 维向量
  metadata: {
    category: "database",
    source: "wiki",
    createdAt: ISODate()
  }
});

// 向量搜索查询
db.documents.aggregate([
  {
    $vectorSearch: {
      index: "vector_index",
      path: "embedding",
      queryVector: queryEmbedding,  // 查询向量
      numCandidates: 100,           // 候选数量
      limit: 10                      // 返回数量
    }
  },
  {
    $project: {
      content: 1,
      score: { $meta: "vectorSearchScore" },
      metadata: 1
    }
  }
]);

// 混合搜索：向量 + 关键词
db.documents.aggregate([
  {
    $search: {
      index: "hybrid_index",
      compound: {
        must: [
          {
            vectorSearch: {
              path: "embedding",
              queryVector: queryEmbedding,
              numCandidates: 100,
              score: { boost: { value: 1 } }
            }
          }
        ],
        should: [
          {
            text: {
              query: "database SQL",
              path: "content",
              score: { boost: { value: 0.3 } }
            }
          }
        ]
      }
    }
  }
]);
```

---

## Mongoose ODM

### 基础用法

```javascript
// 定义 Schema
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    trim: true,
    maxlength: 100
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  },
  age: {
    type: Number,
    min: 0,
    max: 150
  },
  role: {
    type: String,
    enum: ['user', 'admin', 'moderator'],
    default: 'user'
  },
  preferences: {
    theme: { type: String, default: 'light' },
    notifications: { type: Boolean, default: true }
  },
  // 数组字段
  tags: [String],
  // 嵌套文档
  address: {
    street: String,
    city: String,
    country: String
  },
  // 虚拟字段
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// 索引
userSchema.index({ email: 1 });
userSchema.index({ name: 'text' });

// 虚拟字段
userSchema.virtual('fullName').get(function() {
  return `${this.name} (${this.role})`;
});

// 方法
userSchema.methods.greet = function() {
  return `Hello, I'm ${this.name}!`;
};

// 静态方法
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

// 中间件
userSchema.pre('save', function(next) {
  console.log('About to save:', this.name);
  this.email = this.email.toLowerCase();
  next();
});

// 编译模型
const User = mongoose.model('User', userSchema);

// 使用
async function example() {
  const user = new User({
    name: 'Alice',
    email: 'Alice@Example.com',
    age: 30
  });
  
  await user.save();
  console.log(user.fullName);  // "Alice (user)"
  
  const found = await User.findByEmail('alice@example.com');
}
```

### 聚合管道封装

```javascript
// 封装常用聚合操作
userSchema.statics.getStatsByRole = function() {
  return this.aggregate([
    { $group: {
        _id: '$role',
        count: { $sum: 1 },
        avgAge: { $avg: '$age' },
        users: { $push: { name: '$name', email: '$email' } }
      }
    },
    { $sort: { count: -1 } }
  ]);
};

userSchema.statics.getUserWithOrders = async function(userId) {
  return this.aggregate([
    { $match: { _id: mongoose.Types.ObjectId(userId) } },
    {
      $lookup: {
        from: 'orders',
        localField: '_id',
        foreignField: 'userId',
        as: 'orders'
      }
    },
    {
      $addFields: {
        orderCount: { $size: '$orders' },
        totalSpent: { $sum: '$orders.total' }
      }
    }
  ]);
};
```

---

## 实战场景与代码示例

### 场景一：AI 对话历史存储

```javascript
// conversations 集合
const conversationSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  title: { type: String, default: 'New Conversation' },
  model: { type: String, default: 'gpt-4o' },
  status: { type: String, enum: ['active', 'archived'], default: 'active' },
  metadata: mongoose.Schema.Types.Mixed,
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

// messages 集合
const messageSchema = new mongoose.Schema({
  conversationId: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Conversation', 
    required: true,
    index: true
  },
  role: { 
    type: String, 
    enum: ['system', 'user', 'assistant', 'tool'], 
    required: true 
  },
  content: { type: String, required: true },
  embedding: [Number],  // 存储消息嵌入向量
  tokens: {
    prompt: { type: Number, default: 0 },
    completion: { type: Number, default: 0 }
  },
  metadata: {
    model: String,
    finishReason: String,
    toolCalls: [{
      name: String,
      arguments: mongoose.Schema.Types.Mixed
    }]
  },
  createdAt: { type: Date, default: Date.now }
});

// 创建索引
messageSchema.index({ conversationId: 1, createdAt: 1 });
messageSchema.index({ embedding: 1 });

// 对话历史查询
messageSchema.statics.getConversationMessages = async function(conversationId, limit = 50) {
  return this.find({ conversationId })
    .sort({ createdAt: -1 })
    .limit(limit)
    .lean();
};

// 语义搜索历史
messageSchema.statics.semanticSearch = async function(conversationId, queryEmbedding, limit = 5) {
  return this.aggregate([
    { $match: { 
        conversationId: mongoose.Types.ObjectId(conversationId),
        embedding: { $exists: true, $ne: [] }
      }
    },
    {
      $addFields: {
        score: {
          $reduce: {
            input: { $zip: { inputs: ["$embedding", queryEmbedding] } },
            initialValue: 0,
            in: { $add: ["$$value", { $multiply: ["$$this.0", "$$this.1"] }] }
          }
        }
      }
    },
    { $sort: { score: -1 } },
    { $limit: limit },
    {
      $project: {
        _id: 1,
        role: 1,
        content: 1,
        score: 1,
        createdAt: 1
      }
    }
  ]);
};
```

### 场景二：实时数据处理

```javascript
// Change Streams 实时监听
async function watchConversations() {
  const collection = mongoose.connection.collection('conversations');
  
  const changeStream = collection.watch(
    [
      { $match: { operationType: { $in: ['insert', 'update', 'delete'] } } }
    ],
    { fullDocument: 'updateLookup' }
  );
  
  changeStream.on('change', async (change) => {
    switch (change.operationType) {
      case 'insert':
        console.log('New conversation:', change.fullDocument);
        // 触发通知
        break;
      case 'update':
        console.log('Conversation updated:', change.fullDocument._id);
        // 更新缓存
        break;
      case 'delete':
        console.log('Conversation deleted:', change.documentKey._id);
        // 清理关联数据
        break;
    }
  });
  
  // 错误处理
  changeStream.on('error', (error) => {
    console.error('Change stream error:', error);
  });
  
  return changeStream;
}

// 在 mongoose 连接中使用
mongoose.connection.once('open', () => {
  watchConversations();
});
```

---

## 选型建议

### Schema 设计最佳实践

| 场景 | 推荐模式 | 说明 |
|------|----------|------|
| **一对少，固定数据** | 嵌入 | 评论、地址 |
| **一对多，变化数据** | 引用 | 订单项、标签 |
| **频繁更新的数据** | 引用 | 用户、产品 |
| **需要全文搜索** | 嵌入 + Atlas Search | 文章内容 |
| **层级数据** | 血统模式 | 分类树 |
| **时间序列** | 桶模式 | 传感器数据 |
| **AI 嵌入向量** | 独立字段 | 1536+ 维 |

### 性能优化建议

```javascript
// 避免的问题
// ❌ 文档过大（超过 16MB）
// ❌ 数组无限增长
// ❌ 缺少索引的查询
// ❌ 过深的嵌套

// 推荐做法
// ✅ 合理使用引用
// ✅ 设置合理的文档大小
// ✅ 为常用查询创建索引
// ✅ 使用 projection 限制返回字段
// ✅ 使用 lean() 获取原生 JS 对象
```

---

> [!TIP]
> 对于 AI 应用，MongoDB 的灵活性使其非常适合存储 Prompt/Response 对话历史和 AI 输出结果。结合 Atlas Vector Search，可以实现语义搜索功能。

---

> [!SUCCESS]
> 本文档全面介绍了 MongoDB 的核心特性、8.0 新功能、文档模型设计、Aggregation Pipeline、Atlas 云服务，以及在 AI 应用中的实战场景。MongoDB 凭借其灵活的模式和强大的聚合能力，是 AI 应用数据存储的优秀选择。

---

## 完整安装与环境配置

### 安装方法

```bash
# macOS
brew install mongodb-community@7.0
brew services start mongodb-community@7.0

# Ubuntu/Debian
# 添加 MongoDB 仓库
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | gpg --dearmor -o /usr/share/keyrings/mongodb-server-7.0.gpg
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org

# Docker
docker run -d \
  --name mongodb \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  -p 27017:27017 \
  mongo:7.0

# Docker Compose
version: "3.8"
services:
  mongodb:
    image: mongo:7.0
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongodb_data:
```

### MongoDB Shell

```bash
# 连接
mongosh "mongodb://localhost:27017"
mongosh "mongodb://admin:password@localhost:27017"
mongosh "mongodb://localhost:27017,mongodb2:27017/?replicaSet=rs0"

# 常用命令
show dbs
use mydb
show collections
db.help()

# CRUD
db.users.find()
db.users.insertOne({ name: "John" })
db.users.updateOne({ _id: 1 }, { $set: { name: "Jane" } })
db.users.deleteOne({ _id: 1 })

# 聚合
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customer", total: { $sum: "$amount" } } }
])
```

### 配置文件

```yaml
# mongod.conf
systemLog:
  destination: file
  path: /var/log/mongodb/mongod.log
  logAppend: true

storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

processManagement:
  timeZoneInfo: /usr/share/zoneinfo

net:
  port: 27017
  bindIp: 127.0.0.1
  maxIncomingConnections: 65536

operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100

replication:
  replSetName: rs0

sharding:
  clusterRole: configsvr
```

---

## 数据库设计模式

### 文档结构设计

```javascript
// 1. 引用模式（规范化）
// User 集合
{
  _id: ObjectId("..."),
  email: "user@example.com",
  name: "John Doe"
}

// Post 集合（引用 User）
{
  _id: ObjectId("..."),
  title: "My Post",
  author_id: ObjectId("..."),  // 引用
  content: "...",
  created_at: ISODate("...")
}

// 2. 嵌入模式（非规范化）
// Order 文档（嵌入 Line Items）
{
  _id: ObjectId("..."),
  customer_id: ObjectId("..."),
  items: [
    { product_id: 1, name: "Product A", quantity: 2, price: 29.99 },
    { product_id: 2, name: "Product B", quantity: 1, price: 49.99 }
  ],
  total: 109.97,
  status: "completed",
  created_at: ISODate("...")
}

// 3. 混合模式
{
  _id: ObjectId("..."),
  title: "Blog Post",
  author: {  // 嵌入基本作者信息
    id: ObjectId("..."),
    name: "John Doe",
    avatar: "https://..."
  },
  content: "...",
  tags: ["tech", "mongodb"],  // 标签数组
  comments: [  // 嵌入评论
    { author: "Jane", text: "Great post!", created_at: ISODate("...") }
  ],
  url: "https://...",  // 引用外部资源
  created_at: ISODate("...")
}
```

### 常见设计模式

```javascript
// 1. 模式版本控制
{
  _id: ObjectId("..."),
  data: { /* 最新数据 */ },
  schema_version: 2,
  migrated_at: ISODate("...")
}

// 2. 桶模式（时间序列）
{
  _id: ObjectId(),
  device_id: "sensor-001",
  date: ISODate("2024-01-15"),
  readings: [
    { time: ISODate("2024-01-15T00:00:00Z"), temp: 22.5 },
    { time: ISODate("2024-01-15T00:01:00Z"), temp: 22.6 },
    // ... 更多读数
  ],
  count: 60
}

// 3. 血统模式（层级数据）
{
  _id: ObjectId("..."),
  name: "Electronics",
  slug: "electronics",
  parent_id: null,  // 根节点
  path: "/electronics"  // 路径
}

{
  _id: ObjectId("..."),
  name: "Smartphones",
  slug: "smartphones",
  parent_id: ObjectId("..."),  // Electronics
  path: "/electronics/smartphones"
}

// 4. 附加模式（处理增长字段）
// 博客文章（评论太多时移出）
{
  _id: ObjectId("..."),
  title: "...",
  comments_count: 1500,
  recent_comments: [
    { user: "user1", text: "...", created_at: ISODate("...") }
  ]
}

// 评论移到独立集合
{
  _id: ObjectId("..."),
  post_id: ObjectId("..."),
  user: "user1",
  text: "...",
  created_at: ISODate("...")
}
```

---

## 查询优化实战

### 索引设计

```javascript
// 1. 单字段索引
db.users.createIndex({ email: 1 }, { unique: true });
db.posts.createIndex({ author_id: 1 });
db.posts.createIndex({ created_at: -1 });

// 2. 复合索引
db.posts.createIndex({ author_id: 1, created_at: -1 });
db.orders.createIndex({ customer_id: 1, status: 1 });

// 3. 多键索引（数组字段）
db.products.createIndex({ tags: 1 });
db.users.createIndex({ "address.city": 1 });

// 4. 文本索引
db.articles.createIndex({ title: "text", content: "text" });

// 搜索
db.articles.find(
  { $text: { $search: "mongodb tutorial" } },
  { score: { $meta: "textScore" } }
).sort({ score: { $meta: "textScore" } });

// 5. TTL 索引（自动过期）
db.sessions.createIndex(
  { created_at: 1 },
  { expireAfterSeconds: 3600 }  // 1 小时后自动删除
);

// 6. 部分索引
db.orders.createIndex(
  { customer_id: 1, created_at: -1 },
  { partialFilterExpression: { status: "pending" } }
);

// 7. 稀疏索引
db.users.createIndex(
  { phone: 1 },
  { sparse: true }  // 只索引存在 phone 字段的文档
);

// 8. 索引属性
db.users.createIndex(
  { email: 1 },
  { unique: true, background: true, name: "idx_users_email" }
);

// 查看索引
db.collection.getIndexes();

// 删除索引
db.collection.dropIndex("index_name");
db.collection.dropIndexes();  // 删除所有非 _id 索引
```

### 查询优化技巧

```javascript
// 1. 使用 explain()
db.users.find({ email: "test@example.com" }).explain("executionStats");

// 2. 使用投影（Projection）
// ❌ 返回所有字段
db.users.find({ _id: 1 });

// ✅ 只返回需要的字段
db.users.find({ _id: 1 }, { email: 1, name: 1, _id: 0 });

// 3. 使用 Limit
db.posts.find({ published: true }).limit(20);

// 4. 使用 Sort + Limit（覆盖索引）
// 假设有索引 { author_id: 1, created_at: -1 }
db.posts
  .find({ author_id: ObjectId("...") })
  .sort({ created_at: -1 })
  .limit(10)
  .hint({ author_id: 1, created_at: -1 });

// 5. 使用 lean()（获取原生 JS 对象）
const users = await User.find({ active: true }).lean();

// 6. 分页优化
// ❌ OFFSET 分页（慢）
db.posts.find().skip(10000).limit(20);

// ✅ 游标分页（快）
db.posts.find({ _id: { $gt: lastId } }).limit(20);

// ✅ 时间戳分页
db.posts
  .find({ created_at: { $lt: lastTimestamp } })
  .sort({ created_at: -1 })
  .limit(20);

// 7. 避免正则开头包含 ^
db.users.find({ name: /.*keyword.*/ });  // 慢
db.users.find({ name: /keyword/ });  // 可以使用索引前缀优化
```

---

## Aggregation Pipeline 深度使用

### 管道阶段

```javascript
// 1. $match - 过滤
db.orders.aggregate([
  { $match: { status: "completed", created_at: { $gte: ISODate("2024-01-01") } } }
]);

// 2. $group - 分组
db.orders.aggregate([
  { $group: {
      _id: "$customer_id",
      total_amount: { $sum: "$amount" },
      order_count: { $sum: 1 },
      avg_amount: { $avg: "$amount" }
  }}
]);

// 3. $sort - 排序
db.orders.aggregate([
  { $sort: { total_amount: -1 } }
]);

// 4. $limit - 限制
db.orders.aggregate([
  { $limit: 10 }
]);

// 5. $project - 投影
db.orders.aggregate([
  { $project: {
      _id: 0,
      customer_id: 1,
      total: { $round: ["$total_amount", 2] },
      year: { $year: "$created_at" }
  }}
]);

// 6. $lookup - 关联
db.orders.aggregate([
  { $lookup: {
      from: "customers",
      localField: "customer_id",
      foreignField: "_id",
      as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      "customer.name": 1,
      total: 1
  }}
]);

// 7. $unwind - 展开数组
db.orders.aggregate([
  { $unwind: "$items" },
  { $group: {
      _id: "$items.product_id",
      total_sold: { $sum: "$items.quantity" }
  }}
]);

// 8. $addFields - 添加字段
db.orders.aggregate([
  { $addFields: {
      discount: { $multiply: ["$total", 0.1] },
      final_total: { $subtract: ["$total", { $multiply: ["$total", 0.1] }] }
  }}
]);

// 9. $bucket - 分桶
db.orders.aggregate([
  { $bucket: {
      groupBy: "$amount",
      boundaries: [0, 50, 100, 200, 500],
      default: "500+",
      output: { count: { $sum: 1 } }
  }}
]);

// 10. $facet - 多管道
db.orders.aggregate([
  { $facet: {
      totalOrders: [{ $count: "count" }],
      byStatus: [{ $group: { _id: "$status", count: { $sum: 1 } } }],
      totalAmount: [{ $group: { _id: null, total: { $sum: "$amount" } } }]
  }}
]);
```

### 聚合实战

```javascript
// 1. 每日销售统计
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: {
        year: { $year: "$created_at" },
        month: { $month: "$created_at" },
        day: { $dayOfMonth: "$created_at" }
      },
      revenue: { $sum: "$amount" },
      orders: { $sum: 1 }
  }},
  { $sort: { "_id.year": 1, "_id.month": 1, "_id.day": 1 } }
]);

// 2. 用户购买排名
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: {
      _id: "$customer_id",
      total_spent: { $sum: "$amount" },
      order_count: { $sum: 1 }
  }},
  { $sort: { total_spent: -1 } },
  { $limit: 10 },
  { $lookup: {
      from: "customers",
      localField: "_id",
      foreignField: "_id",
      as: "customer"
  }},
  { $unwind: "$customer" },
  { $project: {
      "customer.name": 1,
      total_spent: 1,
      order_count: 1
  }}
]);

// 3. 产品销售分析
db.orders.aggregate([
  { $unwind: "$items" },
  { $group: {
      _id: "$items.product_id",
      product_name: { $first: "$items.name" },
      total_quantity: { $sum: "$items.quantity" },
      total_revenue: { $sum: { $multiply: ["$items.quantity", "$items.price"] } }
  }},
  { $sort: { total_revenue: -1 } }
]);
```

---

## 备份与恢复

### 备份方法对比

| 备份类型 | 工具 | 特点 | 适用场景 |
|---------|------|------|----------|
| **mongodump** | mongodump | 逻辑备份，跨版本 | 应用级恢复 |
| **文件系统** | cp/tar | 物理备份，快速 | 灾难恢复 |
| **OpLog** | mongodump --oplog | 增量备份 | 精确恢复 |
| **Atlas 备份** | Atlas Console | 自动备份，云托管 | Atlas 用户 |

### mongodump 使用

```bash
# 备份单个数据库
mongodump --uri="mongodb://localhost:27017/mydb" --out=/backup/

# 备份所有数据库
mongodump --uri="mongodb://localhost:27017" --out=/backup/

# 备份特定集合
mongodump --uri="mongodb://localhost:27017/mydb" \
    --collection=users --out=/backup/

# 压缩备份
mongodump --uri="mongodb://localhost:27017/mydb" \
    --archive=/backup/mydb.archive --gzip

# 带 OpLog 的备份（副本集）
mongodump --uri="mongodb://localhost:27017" \
    --oplog --out=/backup/with_oplog

# 远程备份
mongodump --host=remote.mongodb.com --port=27017 \
    --username=user --password=password \
    --authenticationDatabase=admin \
    --out=/backup/remote/

# 排除特定集合
mongodump --uri="mongodb://localhost:27017/mydb" \
    --excludeCollection=logs \
    --excludeCollectionWithPrefix=temp_
```

### 恢复操作

```bash
# 恢复单个数据库
mongorestore --uri="mongodb://localhost:27017" /backup/mydb/

# 恢复前删除现有数据
mongorestore --uri="mongodb://localhost:27017" \
    --drop /backup/mydb/

# 恢复压缩备份
mongorestore --uri="mongodb://localhost:27017/mydb" \
    --archive=/backup/mydb.archive --gzip

# 恢复特定集合
mongorestore --uri="mongodb://localhost:27017/mydb" \
    --collection=users /backup/mydb/users.bson

# OpLog 恢复
mongorestore --uri="mongodb://localhost:27017" \
    --oplogReplay /backup/with_oplog/

# 恢复前查看备份内容
mongorestore --dry-run --uri="mongodb://localhost:27017" /backup/mydb/
```

### 文件系统备份

```bash
# 锁定数据库（副本集）
mongod --replSet rs0
# 在主节点执行
db.adminCommand({ fsync: 1, lock: true })

# 使用 cp 备份数据目录
cp -R /data/db /backup/$(date +%Y%m%d)/

# 解锁
db.adminCommand({ fsyncUnlock: 1 })

# 使用 LVM 快照
lvcreate --size 10G --snapshot --name mongodb-snap /dev/vg0/mongodb
mount /dev/vg0/mongodb-snap /mnt/snap
tar -czf /backup/mongodb-$(date +%Y%m%d).tar.gz /mnt/snap
umount /mnt/snap
lvdelete /dev/vg0/mongodb-snap
```

### 副本集备份策略

```javascript
// 副本集备份脚本
db.adminCommand({
  backupCursor: {
    includeRest: true,
    maxTimeMS: 60000
  }
})
```

```bash
#!/bin/bash
# 副本集备份脚本

REPLICA_SET="rs0"
BACKUP_DIR="/backup/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

mkdir -p "$BACKUP_DIR"

# 使用 mongodump 备份（从节点更佳）
mongodump --uri="mongodb://localhost:27017/?replicaSet=$REPLICA_SET" \
    --oplog \
    --out="$BACKUP_DIR/backup_$DATE"

# 压缩备份
tar -czf "$BACKUP_DIR/backup_$DATE.tar.gz" "$BACKUP_DIR/backup_$DATE"
rm -rf "$BACKUP_DIR/backup_$DATE"

# 清理旧备份
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed: backup_$DATE.tar.gz"
```

---

## Python 集成

### PyMongo 使用指南

```python
from pymongo import MongoClient, DESCENDING
from bson import ObjectId
from datetime import datetime

# 连接 MongoDB
client = MongoClient("mongodb://localhost:27017")
db = client["mydb"]

# 获取集合
users = db["users"]

# CRUD 操作
# 创建
result = users.insert_one({
    "name": "Alice",
    "email": "alice@example.com",
    "age": 30,
    "tags": ["python", "mongodb"],
    "created_at": datetime.utcnow()
})
print(f"Inserted ID: {result.inserted_id}")

# 批量创建
users.insert_many([
    {"name": "Bob", "email": "bob@example.com", "age": 25},
    {"name": "Carol", "email": "carol@example.com", "age": 28}
])

# 读取
# 按 ID 查询
user = users.find_one({"_id": ObjectId("...")})

# 条件查询
active_users = users.find({"status": "active"})
young_users = users.find({"age": {"$gte": 18, "$lt": 30}})

# 投影（只返回特定字段）
names_only = users.find({}, {"name": 1, "email": 1, "_id": 0})

# 排序和限制
top_users = users.find().sort("age", DESCENDING).limit(10)

# 计数
count = users.count_documents({"status": "active"})

# 更新
users.update_one(
    {"_id": ObjectId("...")},
    {"$set": {"age": 31}}
)

# 批量更新
users.update_many(
    {"status": "inactive"},
    {"$set": {"last_active": datetime.utcnow()}}
)

# 原子更新
users.update_one(
    {"_id": ObjectId("...")},
    {"$inc": {"login_count": 1}}
)

# 删除
users.delete_one({"_id": ObjectId("...")})
users.delete_many({"status": "deleted"})
```

### Motor（异步）

```python
import asyncio
from motor.motor_asyncio import AsyncIOMotorClient
from bson import ObjectId

async def main():
    client = AsyncIOMotorClient("mongodb://localhost:27017")
    db = client["mydb"]
    users = db["users"]
    
    # 创建
    result = await users.insert_one({
        "name": "Alice",
        "email": "alice@example.com"
    })
    print(f"Inserted: {result.inserted_id}")
    
    # 读取
    user = await users.find_one({"_id": result.inserted_id})
    print(f"Found: {user}")
    
    # 批量操作
    await users.insert_many([
        {"name": "Bob", "email": "bob@example.com"},
        {"name": "Carol", "email": "carol@example.com"}
    ])
    
    # 查询
    async for user in users.find({"name": {"$regex": "^A"}}):
        print(user)
    
    # 更新
    await users.update_one(
        {"_id": result.inserted_id},
        {"$set": {"age": 30}}
    )
    
    # 聚合
    pipeline = [
        {"$group": {"_id": "$status", "count": {"$sum": 1}}}
    ]
    async for status in users.aggregate(pipeline):
        print(status)
    
    client.close()

asyncio.run(main())
```

### MongoEngine ODM

```python
from mongoengine import connect, Document, StringField, IntField, DateTimeField, ListField
from datetime import datetime

# 连接
connect("mydb", host="localhost", port=27017)

# 定义模型
class User(Document):
    name = StringField(required=True, max_length=100)
    email = StringField(required=True, unique=True)
    age = IntField(min_value=0, max_value=150)
    tags = ListField(StringField())
    created_at = DateTimeField(default=datetime.utcnow)
    
    meta = {
        "collection": "users",
        "indexes": ["email", "name"],
        "ordering": ["-created_at"]
    }

# CRUD 操作
# 创建
user = User(name="Alice", email="alice@example.com", age=30)
user.save()

# 读取
user = User.objects(email="alice@example.com").first()
users = User.objects(age__gte=18)

# 更新
user.age = 31
user.save()

# 删除
user.delete()

# 复杂查询
young_users = User.objects(age__gte=18, age__lt=30)
alice = User.objects.get_or_404(email="alice@example.com")

# 聚合查询
stats = User.objects.aggregate([
    {"$group": {"_id": None, "avg_age": {"$avg": "$age"}, "count": {"$sum": 1}}}
])
```

### PyMongo 高级操作

```python
from pymongo import MongoClient, UpdateOne
from bson import ObjectId

client = MongoClient("mongodb://localhost:27017")
db = client["mydb"]

# 聚合管道
pipeline = [
    # 匹配
    {"$match": {"status": "active"}},
    
    # 展开数组
    {"$unwind": "$tags"},
    
    # 分组
    {"$group": {
        "_id": "$tags",
        "count": {"$sum": 1},
        "users": {"$push": "$name"}
    }},
    
    # 排序
    {"$sort": {"count": -1}},
    
    # 限制
    {"$limit": 10}
]

results = list(db.users.aggregate(pipeline))
for r in results:
    print(f"{r['_id']}: {r['count']} users")

# 事务（副本集）
with client.start_session() as session:
    with session.start_transaction():
        db.orders.insert_one({"item": "Book"}, session=session)
        db.inventory.update_one(
            {"item": "Book", "qty": {"$gt": 0}},
            {"$inc": {"qty": -1}},
            session=session
        )

# 批量写入
operations = [
    UpdateOne(
        {"_id": ObjectId("...")},
        {"$set": {"status": "completed"}}
    ),
    UpdateOne(
        {"_id": ObjectId("...")},
        {"$set": {"status": "cancelled"}}
    )
]
result = db.orders.bulk_write(operations)
print(f"Modified: {result.modified_count}")
```

---

## 常见陷阱与最佳实践

### 陷阱 1：文档过大

```javascript
// ❌ 错误：文档无限增长（超过 16MB 限制）
{
  _id: ObjectId("..."),
  title: "...",
  comments: [
    { user: "user1", text: "comment 1" },
    // ... 无限增长
  ]
}

// ✅ 正确：使用独立的 comments 集合
// Post 文档
{
  _id: ObjectId("..."),
  title: "...",
  comment_count: 1000
}

// Comments 集合
{
  _id: ObjectId("..."),
  post_id: ObjectId("..."),
  user: "user1",
  text: "...",
  created_at: ISODate("...")
}
```

### 陷阱 2：缺少索引

```javascript
// ❌ 错误：频繁查询的字段没有索引
db.orders.find({ customer_id: ObjectId("...") });  // 慢

// ✅ 正确：创建索引
db.orders.createIndex({ customer_id: 1, created_at: -1 });
```

### 陷阱 3：过度使用 $where

```javascript
// ❌ 错误：$where 性能差
db.users.find({ $where: "this.age > 25 && this.active === true" });

// ✅ 正确：使用标准查询运算符
db.users.find({ age: { $gt: 25 }, active: true });
```

### 最佳实践清单

1. **合理设计文档结构**：平衡嵌入和引用。
2. **适当索引**：为常用查询创建索引。
3. **使用投影**：只返回需要的字段。
4. **分页优化**：使用游标分页替代 OFFSET。
5. **聚合管道优化**：将 $match 放在最前面。
6. **避免过深嵌套**：最多 2-3 层。
7. **TTL 索引**：用于会话、日志等自动过期。
8. **副本集部署**：保证高可用。
9. **监控慢查询**：使用 Profiler。
10. **定期维护**：压缩、重新索引。

---

## 与其他数据库对比

### MongoDB vs PostgreSQL

| 特性 | MongoDB | PostgreSQL |
|------|----------|------------|
| **模型** | 文档型 | 关系型 |
| **模式** | 灵活 | 严格 |
| **事务** | ACID（副本集） | ACID |
| **JOIN** | $lookup | 完整 JOIN |
| **扩展性** | 水平扩展 | 垂直扩展 |
| **查询** | MQL | SQL |
| **JSON** | 原生 | jsonb |

### MongoDB vs MySQL

| 特性 | MongoDB | MySQL |
|------|----------|-------|
| **模型** | 文档型 | 关系型 |
| **扩展性** | 水平扩展 | 垂直扩展 |
| **模式** | 灵活 | 固定 |
| **事务** | 副本集内 | 单节点/主从 |
| **查询** | MQL | SQL |

### MongoDB vs Redis

| 特性 | MongoDB | Redis |
|------|---------|-------|
| **类型** | 文档数据库 | KV 存储 |
| **持久化** | 默认持久化 | 可选持久化 |
| **容量** | 大数据量 | 内存优先 |
| **查询** | 复杂查询 | 简单 KV |
| **适用** | 主数据库 | 缓存/会话 |

---

## MongoDB 事务与原子性

### 单文档原子性

```javascript
// MongoDB 对单文档的操作是原子的
// 嵌入式文档被视为单个字段

// ✅ 原子操作：更新嵌入字段
db.users.updateOne(
  { _id: ObjectId("...") },
  {
    $set: {
      "profile.name": "New Name",
      "profile.bio": "New Bio"
    }
  }
);

// ✅ 原子操作：数组操作
db.posts.updateOne(
  { _id: ObjectId("...") },
  {
    $push: { comments: { user: "user1", text: "Great!" } },
    $inc: { comment_count: 1 }
  }
);

// ✅ 原子操作：条件更新
db.inventory.updateOne(
  { _id: ObjectId("..."), quantity: { $gte: 1 } },
  { $inc: { quantity: -1 } }
);

// ❌ 非原子操作（需要多文档事务）
db.orders.updateOne({ _id: ObjectId("...") }, { $set: { status: "shipped" } });
db.inventory.updateOne({ product_id: "SKU123" }, { $inc: { quantity: -1 } });
```

### 多文档事务

```javascript
// 1. 启动事务会话
const session = db.getMongo().startSession();

// 2. 开始事务
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority" }
});

try {
  const ordersCollection = session.getDatabase("mydb").orders;
  const inventoryCollection = session.getDatabase("mydb").inventory;
  const usersCollection = session.getDatabase("mydb").users;

  // 操作 1：创建订单
  const order = {
    user_id: ObjectId("..."),
    items: [{ product_id: "SKU123", quantity: 2 }],
    total: 99.99,
    status: "pending",
    created_at: new Date()
  };
  ordersCollection.insertOne(order, { session });

  // 操作 2：扣减库存
  inventoryCollection.updateOne(
    { product_id: "SKU123", quantity: { $gte: 2 } },
    { $inc: { quantity: -2 } },
    { session }
  );

  // 操作 3：更新用户订单统计
  usersCollection.updateOne(
    { _id: ObjectId("...") },
    {
      $inc: { order_count: 1 },
      $set: { last_order_at: new Date() }
    },
    { session }
  );

  // 3. 提交事务
  session.commitTransaction();
  console.log("Transaction committed successfully");

} catch (error) {
  // 4. 回滚事务
  session.abortTransaction();
  console.error("Transaction aborted:", error);
} finally {
  session.endSession();
}
```

### 事务选项详解

```javascript
// 事务的读关注点 (readConcern)
// level: "local" - 返回本地最旧的数据（默认）
// level: "majority" - 返回被大多数副本确认的数据
// level: "snapshot" - 返回快照数据（仅在事务中可用）

// 事务的写关注点 (writeConcern)
// w: 1 - 仅主节点确认（默认）
// w: "majority" - 大多数节点确认
// w: "all" - 所有节点确认
// wtimeout: 5000 - 超时时间（毫秒）

// 示例：强一致性事务
session.startTransaction({
  readConcern: { level: "snapshot" },
  writeConcern: { w: "majority", wtimeout: 10000 }
});

// 示例：最终一致性事务（更快）
session.startTransaction({
  readConcern: { level: "majority" },
  writeConcern: { w: 1 }
});
```

### 事务最佳实践

```javascript
// 1. 保持事务简短
// ❌ 错误：长时间运行的事务
session.startTransaction();
// ... 执行大量操作 ...
session.commitTransaction();

// ✅ 正确：短小的事务
async function processOrder(orderId) {
  const session = db.getMongo().startSession();
  try {
    session.startTransaction();
    
    // 只包含必要的写操作
    await Promise.all([
      updateOrderStatus(orderId, "confirmed", session),
      reserveInventory(orderId, session),
      sendConfirmation(orderId)  // 非关键操作移到事务外
    ]);
    
    await session.commitTransaction();
  } finally {
    session.endSession();
  }
}

// 2. 使用保存点（MongoDB 4.2+）
session.startTransaction();
try {
  db.collection1.updateOne({ ... }, { ... }, { session });
  session.commitTransaction();
} catch (e) {
  session.abortTransaction();
}

// 3. 事务重试逻辑
async function executeWithRetry(operation, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    const session = db.getMongo().startSession();
    try {
      session.startTransaction();
      const result = await operation(session);
      await session.commitTransaction();
      return result;
    } catch (error) {
      await session.abortTransaction();
      
      if (error.hasErrorLabel && 
          error.hasErrorLabel("TransientTransactionError") &&
          attempt < maxRetries) {
        // 重试
        await new Promise(resolve => setTimeout(resolve, 100 * attempt));
        continue;
      }
      throw error;
    } finally {
      session.endSession();
    }
  }
}

// 4. 监控事务状态
db.adminCommand({
  serverStatus: 1,
  trasactions: 1
});
```

---

## MongoDB Change Streams

### 基础使用

```javascript
// 1. 监听集合变更
const changeStream = db.users.watch();

changeStream.on("change", change => {
  console.log("Change detected:", change);
  
  switch (change.operationType) {
    case "insert":
      console.log("New document:", change.fullDocument);
      break;
    case "update":
      console.log("Updated document:", change.fullDocument);
      console.log("Update fields:", change.updateDescription);
      break;
    case "delete":
      console.log("Deleted document ID:", change.documentKey);
      break;
    case "replace":
      console.log("Replaced document:", change.fullDocument);
      break;
  }
});

// 2. 监听特定操作类型
const changeStream = db.orders.watch([
  { $match: { operationType: { $in: ["insert", "update", "delete"] } } }
]);

// 3. 只监听特定字段的变更
const changeStream = db.orders.watch([
  { $match: { "updateDescription.updatedFields.status": { $exists: true } } }
]);

// 4. 监听多个集合
const changeStream1 = db.orders.watch();
const changeStream2 = db.users.watch();

// 5. 监听整个数据库
const changeStream = db.watch([
  { $match: { operationType: { $in: ["insert", "update", "delete"] } } }
]);
```

### 恢复光标

```javascript
// 1. 使用 resume token 恢复变更流
let resumeToken = null;
const changeStream = db.orders.watch();

changeStream.on("change", change => {
  resumeToken = change._id;
  
  if (change.operationType === "insert") {
    // 处理变更...
  }
});

// 2. 从指定时间恢复
const timestamp = new Date("2024-01-15T10:30:00Z");
const changeStream = db.orders.watch([], {
  startAtOperationTime: timestamp
});

// 3. 从指定变更恢复
const changeStream = db.orders.watch([], {
  resumeAfter: { _id: ObjectId("...") }
});

// 4. 完整的恢复示例
async function watchCollectionWithResume(collection) {
  let resumeToken = getLastResumeToken();  // 从持久化存储获取
  
  const options = resumeToken 
    ? { resumeAfter: resumeToken }
    : {};
  
  const changeStream = collection.watch([], options);
  
  for await (const change of changeStream) {
    console.log("Processing change:", change);
    
    // 持久化 resume token
    await saveResumeToken(change._id);
    
    // 处理业务逻辑
    await processChange(change);
  }
}

// 5. 错误处理和重连
async function resilientWatch(collection) {
  while (true) {
    try {
      const changeStream = collection.watch();
      
      for await (const change of changeStream) {
        await processChange(change);
      }
    } catch (error) {
      console.error("Change stream error:", error);
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### 变更流聚合管道

```javascript
// 1. 使用聚合管道过滤
const changeStream = db.orders.watch([
  {
    $match: {
      $or: [
        { operationType: "update" },
        { operationType: "insert" }
      ]
    }
  }
]);

// 2. 监听特定文档
const orderId = ObjectId("...");
const changeStream = db.orders.watch([
  { $match: { "documentKey": orderId } }
]);

// 3. 只监听删除操作（因为 update/delete 后文档不可用）
const changeStream = db.orders.watch([
  { $match: { operationType: { $in: ["delete", "drop", "rename"] } } }
]);

// 4. 监听集合删除
const changeStream = db.watch([
  { $match: { operationType: { $in: ["drop", "rename"] } } }
]);

// 5. 监听索引变更
const changeStream = db.orders.watch([
  { $match: { operationType: { $in: ["createIndex", "dropIndex"] } } }
]);
```

### 实际应用场景

```javascript
// 场景 1：实时通知
async function setupNotifications(db) {
  const changeStream = db.orders.watch([
    { $match: { operationType: "update" } }
  ]);
  
  for await (const change of changeStream) {
    const { orderId, newStatus } = change.documentKey;
    
    if (change.updateDescription.updatedFields.status === "shipped") {
      await sendPushNotification(orderId, "Order shipped!");
    }
  }
}

// 场景 2：数据同步
async function syncToElasticsearch(db) {
  const changeStream = db.products.watch();
  
  for await (const change of changeStream) {
    switch (change.operationType) {
      case "insert":
      case "update":
      case "replace":
        await elasticsearch.index({
          index: "products",
          id: change.documentKey._id.toString(),
          body: change.fullDocument
        });
        break;
      case "delete":
        await elasticsearch.delete({
          index: "products",
          id: change.documentKey._id.toString()
        });
        break;
    }
  }
}

// 场景 3：审计日志
async function createAuditLog(db) {
  const changeStream = db.watch([
    { $match: { 
        operationType: { $in: ["insert", "update", "delete"] },
        "ns.coll": { $nin: ["audit_log", "system.sessions"] }
    }}
  ]);
  
  for await (const change of changeStream) {
    await db.audit_log.insertOne({
      timestamp: new Date(),
      operation: change.operationType,
      collection: change.ns.coll,
      documentKey: change.documentKey,
      fullDocument: change.fullDocument,
      updateDescription: change.updateDescription,
      user: change.user
    });
  }
}

// 场景 4：缓存失效
async function invalidateCache(db, cache) {
  const changeStream = db.users.watch([
    { $match: { operationType: "update" } }
  ]);
  
  for await (const change of changeStream) {
    const userId = change.documentKey._id.toString();
    cache.del(`user:${userId}`);
    cache.del(`user:${userId}:profile`);
  }
}
```

---

## MongoDB 分片集群

### 分片架构

```javascript
// 分片集群组件：
// 1. mongos - 路由进程
// 2. config servers - 配置服务器（存储集群元数据）
// 3. shard servers - 分片服务器（存储实际数据）

// 分片键选择原则：
// 1. 高基数 - 字段值要有足够多的不同值
// 2. 低频率 - 不是所有文档都有该字段
// 3. 非单调 - 避免热块（如自增 _id）

// 示例：用户集合分片
// ✅ 好：基于用户 ID
db.adminCommand({
  shardCollection: "mydb.user_sessions",
  key: { user_id: 1, created_at: 1 }
});

// ✅ 好：基于地理位置
db.adminCommand({
  shardCollection: "mydb.checkins",
  key: { location: "2dsphere" }
});

// ✅ 好：基于时间
db.adminCommand({
  shardCollection: "mydb.logs",
  key: { timestamp: 1 }
});

// ❌ 差：基于状态（低基数）
db.adminCommand({
  shardCollection: "mydb.orders",
  key: { status: 1 }  // status 只有几个值
});
```

### 分片策略

```javascript
// 1. 哈希分片（数据均匀分布）
db.adminCommand({
  shardCollection: "mydb.orders",
  key: { order_id: "hashed" }
});

// 2. 范围分片
db.adminCommand({
  shardCollection: "mydb.sales",
  key: { sale_date: 1 }
});

// 3. 复合分片键
db.adminCommand({
  shardCollection: "mydb.events",
  key: { tenant_id: 1, created_at: 1 }
});

// 4. 多键索引分片（数组字段）
db.adminCommand({
  shardCollection: "mydb.products",
  key: { tags: 1 }  // tags 是数组
});

// 5. 区域分片（地理位置感知）
// 创建区域
db.adminCommand({
  shardingState: {
    enableSharding: "mydb"
  }
});

db.adminCommand({
  updateZoneKeyRange: {
    ns: "mydb.users",
    min: { region: "US" },
    max: { region: "US\0" },
    zone: "US_zone"
  }
});
```

### 分片管理

```javascript
// 1. 查看分片状态
db.adminCommand({ shardCollection: 1 });
db.adminCommand({ listShards: 1 });

// 2. 查看 chunk 分布
db.adminCommand({ getShardDistribution: 1 });

// 3. 手动移动 chunk
db.adminCommand({
  moveChunk: "mydb.users",
  find: { user_id: ObjectId("...") },
  to: "shard0001"
});

// 4. 分割 chunk
db.adminCommand({
  split: "mydb.users",
  middle: { user_id: 5000 }
});

// 5. 均衡器控制
// 查看均衡器状态
db.adminCommand({ balancerStatus: 1 });

// 开启/关闭均衡器
db.adminCommand({ balancerStop: 1 });
db.adminCommand({ balancerStart: 1 });

// 设置均衡窗口
db.adminCommand({
  balancerWindow: {
    start: "23:00",
    stop: "06:00",
    stop: "12:00"
  }
});

// 6. 添加分片
db.adminCommand({
  addShard: "rs2/hostname:27018"
});

// 7. 移除分片
db.adminCommand({
  removeShard: "shard0002"
});

// 8. 查看待迁移的 chunk
db.adminCommand({
  moves: 1
});
```

### 分片查询

```javascript
// 1. 广播查询（所有分片）
// 没有分片键的查询会广播到所有分片
db.orders.find({ status: "pending" });  // 广播查询

// 2. 定向查询（单个分片）
// 使用分片键的查询只访问目标分片
db.orders.find({ user_id: ObjectId("...") });  // 定向查询

// 3. 强制使用索引
db.orders.find({ user_id: ObjectId("...") })
  .hint({ user_id: 1, created_at: -1 });

// 4. 分片聚合
db.orders.aggregate([
  { $match: { created_at: { $gte: ISODate("2024-01-01") } } },
  { $group: { _id: "$status", count: { $sum: 1 } } },
  { $sort: { count: -1 } }
]);

// 5. $lookup 在分片集合
// 分片集合不能使用 $lookup 联结其他分片集合
// 解决方案：使用 $lookup 的 pipeline 形式
db.orders.aggregate([
  { $match: { user_id: ObjectId("...") } },
  { $lookup: {
      from: "users",
      let: { userId: "$user_id" },
      pipeline: [
        { $match: { $expr: { $eq: ["$_id", "$$userId"] } } },
        { $project: { name: 1, email: 1 } }
      ],
      as: "user_info"
    }
  }
]);
```

---

## MongoDB 性能监控

### 诊断命令

```javascript
// 1. 服务器状态
db.serverStatus();

// 2. 当前操作（类似 MySQL 的 PROCESSLIST）
db.currentOp({ "active": true });

// 查看慢查询
db.currentOp({
  "active": true,
  "secs_running": { $gt: 5 }
});

// 查看锁等待
db.currentOp({
  "waitingForLock": true
});

// 终止操作
db.killOp(opid);

// 3. 锁信息
db.adminCommand({ top: 1 });

// 4. 数据库统计
db.stats();
db.users.stats();

// 5. 集合统计
db.users.stats({ scale: 1024 * 1024 });  // MB 为单位

// 6. 索引统计
db.users.aggregate([{ $indexStats: {} }]);

// 7. 连接状态
db.adminCommand({ connectionStatus: 1 });
```

### Profiler 分析

```javascript
// 1. 开启 profiler
db.setProfilingLevel(2, { slowms: 100, sampleRate: 1 });
// level: 0 - 关闭
// level: 1 - 只记录慢查询
// level: 2 - 记录所有查询

// 2. 查看 profiler 集合
db.system.profile.find().sort({ ts: -1 }).limit(10);

// 3. 分析慢查询
db.system.profile.aggregate([
  { $match: { millis: { $gt: 100 } } },
  { $group: {
      _id: "$op",
      count: { $sum: 1 },
      avgTime: { $avg: "$millis" },
      maxTime: { $max: "$millis" },
      queries: { $push: "$command" }
    }
  }
]);

// 4. 分析耗时阶段
db.system.profile.aggregate([
  { $match: { op: "aggregate" } },
  { $unwind: "$secs" },
  { $group: {
      _id: "$secs.name",
      count: { $sum: 1 }
    }
  }
]);
```

### Atlas 监控

```javascript
// Atlas 提供了开箱即用的监控功能：
// 1. Real-Time Performance Panel
// 2. Metrics Explorer
// 3. Query Profiler
// 4. Performance Advisor
// 5. Data Explorer

// 推荐配置的 Atlas 告警：
// - 内存使用率 > 80%
// - CPU 使用率 > 75%
// - 磁盘使用率 > 85%
// - 连接数 > 80%
// - 慢查询率 > 5%
// - 副本集延迟 > 1 秒
// - oplog 窗口 < 24 小时

// 自定义指标
// 在 Atlas > Integrations 创建自定义告警规则
```

### 性能优化技巧

```javascript
// 1. 投影减少网络传输
db.users.findOne(
  { _id: ObjectId("...") },
  { name: 1, email: 1 }  // 只返回这两个字段
);

// 2. 限制返回文档数
db.posts.find({ author_id: ObjectId("...") }).limit(20);

// 3. 避免 skip-large-offset
// ❌ 慢
db.posts.find().skip(10000).limit(20);
// ✅ 快
db.posts.find({ _id: { $gt: lastId } }).limit(20);

// 4. 批量操作
const bulkOps = [
  { insertOne: { document: { name: "User 1" } } },
  { insertOne: { document: { name: "User 2" } } },
  { updateOne: { filter: { _id: 1 }, update: { $set: { name: "Updated" } } } },
  { deleteOne: { filter: { _id: 2 } } }
];
db.users.bulkWrite(bulkOps, { ordered: false });

// 5. 解释执行计划
db.users.find({ email: "test@example.com" }).explain("executionStats");
// 分析：
// IXSCAN - 使用索引扫描（好）
// COLLSCAN - 全表扫描（差）
// FETCH - 获取完整文档

// 6. 覆盖索引查询
// 创建索引覆盖查询字段
db.users.createIndex({ email: 1, name: 1 });
// 查询只返回索引字段
db.users.find({ email: "test@example.com" }, { email: 1, name: 1 });
```

---

> [!SUCCESS]
> 本文档全面介绍了 MongoDB 的核心特性、文档模型、聚合管道、索引优化、副本集分片以及高可用架构。MongoDB 凭借其灵活的文档模型、强大的聚合框架和水平扩展能力，是现代应用开发的重要选择。
