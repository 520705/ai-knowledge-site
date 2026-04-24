---
title: GraphRAG深度指南
date: 2026-04-24
tags:
  - GraphRAG
  - 知识图谱
  - RAG
  - 检索增强生成
  - 图检索
  - 微软
categories:
  - 人工智能工具实操/知识库管理
aliases:
  - GraphRAG入门
  - 知识图谱RAG
  - 微软GraphRAG
---

# GraphRAG深度指南：让AI真正"看懂"你的文档

> [!tip] 读这篇文章你能学到什么
> 你有没有遇到过这种情况：问AI一个需要"总结归纳"的问题，比如"我们公司去年整体的技术发展方向是什么？"，结果AI给你列了一堆零散的内容，根本不是你想要的答案。这篇文章就是教你用GraphRAG解决这个问题，让AI能真正理解文档之间的关系，给出像样的总结性回答。

## 前言：传统RAG的致命缺陷

先说说传统RAG的痛点。我之前做过一个企业内部知识库，里面有几百份技术文档、产品文档、会议纪要。用传统RAG做了个问答系统，测试的时候问了一个问题：

**"我们公司在AI领域有哪些技术创新？"**

结果你猜怎么着？AI给我返回了三个完全不相关的文档片段，一个讲的是机器学习算法，一个讲的是产品推荐系统，还有一个讲的是客服机器人的实现。三个内容风马牛不相及，凑在一起根本回答不了问题。

问题出在哪？我后来才想明白：传统RAG只会做"语义匹配"，它看到"AI"和"技术创新"这些关键词，就去找语义相似的内容。但它根本不知道这些内容之间有什么关系，更别说做全局性的总结了。

这时候GraphRAG就登场了。

## 一、GraphRAG是什么

GraphRAG是微软在2024年开源的一个技术框架，全称是"Graph Retrieval-Augmented Generation"，翻译过来就是"基于知识图谱的检索增强生成"。

它解决的核心问题就是：**让AI不仅能找到相关内容，还能理解这些内容之间的关系，从而给出真正有价值的总结性回答**。

### 1.1 打个比方帮助你理解

普通RAG就像是一个只会"关键词匹配"的图书管理员。你问他"关于机器学习的书在哪"，他就去找书名里带"机器学习"的书。但如果你问他"我们图书馆的藏书体系是怎么组织的，各个领域之间有什么联系"，他就傻眼了。

GraphRAG则是先把所有书的内容"消化"一遍，构建出一个知识地图——谁写了什么、这些书写了哪些技术、这些技术之间是什么关系。然后你问任何问题，它都能沿着这张地图找到相关内容，还能把这些内容串起来给你一个完整的答案。

### 1.2 GraphRAG的核心思想

GraphRAG的设计理念用一句话概括就是：**"先建图，再检索"**。

传统的RAG是直接对文档做向量化和相似度匹配。GraphRAG则是先把文档内容"抽取"成一张知识图谱，然后再基于这张图来做检索和问答。

具体来说，它会：
1. 从文档中识别出各种"实体"（比如人名、公司名、技术名词、事件等）
2. 识别出这些实体之间的"关系"（比如"张三开发了XX系统"、"XX系统使用了YY技术"）
3. 把这些实体和关系构建成一张图
4. 基于这张图来做问答

## 二、核心概念详解

### 2.1 实体是什么

实体就是文档中提到的那些"名词"——可以是具体的人、公司、产品，也可以是抽象的概念、技术名词。

举个例子，假设我们有这样一段文档：

> "2024年3月，阿里巴巴发布了通义千问2.0版本，这是一个基于Transformer架构的大语言模型，在自然语言处理任务上表现优异。"

从这段话里，GraphRAG会识别出以下实体：
- **阿里巴巴**（组织）
- **通义千问2.0**（产品/技术）
- **2024年3月**（时间）
- **Transformer架构**（技术概念）
- **大语言模型**（技术概念）
- **自然语言处理**（技术领域）

每个实体都有自己的"类型"，比如"阿里巴巴"是组织，"通义千问2.0"是产品，"Transformer架构"是技术。

### 2.2 关系是什么

关系就是实体之间的连接。比如：
- "阿里巴巴 **发布了** 通义千问2.0"
- "通义千问2.0 **基于** Transformer架构"
- "通义千问2.0 **用于** 自然语言处理"

通过这些关系，GraphRAG就能知道：阿里巴巴发布了通义千问，通义千问用了Transformer，Transformer可以用于NLP。这样连起来，就形成了一条"知识链"。

### 2.3 社区是什么

当实体和关系越来越多，图就会变得很复杂。这时候就需要"社区发现"算法来帮忙。

社区是什么概念呢？想象一个社交网络——有些人互相认识，形成了一个小圈子；另一些人互相认识，形成了另一个小圈子。社区发现就是找到这些"小圈子"。

在GraphRAG里，社区是一组语义上比较接近的实体。比如"AI技术"可能形成一个社区，里面包含各种AI相关的实体：机器学习、深度学习、神经网络、Transformer等。

社区的好处是：当我们需要回答全局性问题时，不需要遍历所有实体，只需要找到相关的社区就行了。

## 三、GraphRAG是怎么工作的

### 3.1 索引构建阶段

这是GraphRAG最"重"的部分，需要花费一定的时间和计算资源。整个过程大概是这样的：

```
文档 → 文本分块 → 实体抽取 → 关系抽取 → 社区发现 → 社区摘要 → 索引完成
```

**Step 1：文本分块**

首先把文档切成小块（chunk）。每块通常300-600个token，太长的话实体抽取的效果会变差。

**Step 2：实体和关系抽取**

用LLM从每个文本块里抽取实体和关系。这是整个过程中最"贵"的一步，需要大量调用LLM。

```python
# 实体抽取的prompt示例
prompt = """
从以下文本中抽取所有实体和关系。

实体类型：
- PERSON: 人名
- ORGANIZATION: 组织机构
- TECHNOLOGY: 技术名词
- CONCEPT: 概念
- EVENT: 事件

关系类型：
- developed_by: 由...开发
- uses: 使用
- part_of: 是...的一部分
- related_to: 与...相关

文本：{chunk_text}

请以JSON格式输出实体和关系。
"""
```

**Step 3：社区发现**

用Leiden算法（一种图聚类算法）对抽取出的实体和关系进行社区划分。这一步不需要LLM参与，速度很快。

**Step 4：社区摘要**

为每个社区生成一份"摘要"。摘要是对这个社区核心内容的浓缩描述，用于回答全局性问题。

### 3.2 查询阶段

当用户提问时，GraphRAG有两种搜索策略可以选择：

**本地搜索（Local Search）**：适合问具体实体的问题

比如问"通义千问是谁开发的？"，GraphRAG会：
1. 从问题中识别出"通义千问"这个实体
2. 在图谱中找到这个实体
3. 沿着关系找到相关的内容
4. 返回答案

**全局搜索（Global Search）**：适合总结性、归纳性的问题

比如问"2024年AI领域有哪些重要进展？"，GraphRAG会：
1. 把这个问题"广播"给所有社区
2. 让每个社区根据摘要判断自己是否相关
3. 把相关社区的内容聚合起来
4. 让LLM综合这些内容生成答案

全局搜索用到了"Map-Reduce"的思想：
- **Map阶段**：并行处理每个社区，每个社区生成一个"局部答案"
- **Reduce阶段**：把所有局部答案综合起来，生成最终答案

### 3.3 DRIFT搜索：更智能的混合策略

2025年，微软又推出了DRIFT搜索模式，算是本地搜索和全局搜索的"混合版"。

它的思路是：
1. 先用HyDE策略快速定位相关社区
2. 在相关社区里进行局部搜索，完善答案
3. 同时生成后续可能的问题

DRIFT特别适合那种"先要全局了解，再想深入了解"的问题。比如你先问"公司去年做了哪些AI项目"，然后追问"其中最成功的是哪个"。

## 四、实战部署GraphRAG

### 4.1 环境准备

GraphRAG可以用微软官方的Python库来部署，也可以基于Ollama做本地化部署。

**方式一：官方GraphRAG库**

```bash
# 安装graphrag
pip install graphrag

# 创建项目
mkdir my_graphrag && cd my_graphrag
graphrag init --root .

# 这会在当前目录创建：
# - settings.yaml 配置文件
# - input/ 输入目录
# - output/ 输出目录
```

**方式二：基于Ollama本地部署**

如果想完全本地运行，不用微软的API，可以用Ollama：

```bash
# 1. 安装Ollama
curl -fsSL https://ollama.com/install.sh | sh

# 2. 下载模型
ollama pull qwen2.5:72b  # 主模型
ollama pull nomic-embed-text  # embedding模型

# 3. 修改settings.yaml，配置使用ollama
```

### 4.2 配置文件详解

settings.yaml是GraphRAG的核心配置文件：

```yaml
# settings.yaml

# LLM配置
llm:
  type: openai  # 或 azure_openai, ollama, anthropic
  model: gpt-4-turbo
  api_key: ${GRAPHRAG_API_KEY}
  max_tokens: 2000
  temperature: 0.0

# Embedding配置
embedding:
  type: openai  # 或 ollama
  model: text-embedding-3-large
  batch_size: 100

# 文本分块配置
chunks:
  size: 300  # 每块token数
  overlap: 100  # 重叠token数

# 实体抽取配置
extraction:
  entity_types:
    - PERSON
    - ORGANIZATION
    - TECHNOLOGY
    - CONCEPT
    - EVENT
  
  # 关系类型
  relation_types:
    - developed_by
    - uses
    - part_of
    - related_to

# 社区发现配置
community:
  # Leiden算法参数
  resolution: 1.0  # 值越大，社区越多越小
```

### 4.3 构建索引

```bash
# 把你的文档放到input目录（支持.txt, .md, .pdf等）

# 构建索引
graphrag index --root .

# 如果想看进度
graphrag status --root .

# 索引完成后，会在output目录生成：
# - entities.parquet 实体数据
# - relationships.parquet 关系数据
# - communities.parquet 社区数据
# - community_reports/ 社区摘要
```

### 4.4 开始查询

```bash
# 全局搜索（适合总结性问题）
graphrag query \
  --root . \
  --method global \
  --query "总结公司去年的技术创新成果"

# 本地搜索（适合具体问题）
graphrag query \
  --root . \
  --method local \
  --query "谁负责开发了XX系统？"
```

### 4.5 Python代码调用

```python
from graphrag.query import search_local, search_global

# 本地搜索
local_result = await search_local(
    community_reports=community_reports,
    entities=entities,
    runtime_text嵌入=runtime_text嵌入,
    query="具体问题",
    llm=llm_client,
    embeddings=embedding_client
)

# 全局搜索
global_result = await search_global(
    community_reports=community_reports,
    query="总结性问题",
    llm=llm_client
)

print(global_result.response)
```

## 五、懒人版GraphRAG：LazyGraphRAG

### 5.1 为什么需要LazyGraphRAG

标准GraphRAG有个问题：**索引构建太贵了**。

前面说过，GraphRAG的索引构建需要大量调用LLM来抽取实体和关系。如果你有100万token的文档，可能要花几百美元来构建索引。这对于个人开发者或者预算有限的团队来说，是个不小的负担。

LazyGraphRAG就是来解决这个问题的。它的核心理念是：**不要预计算所有东西，而是在查询的时候动态计算**。

### 5.2 LazyGraphRAG的工作原理

标准GraphRAG：提前把所有文档处理成知识图谱
LazyGraphRAG：查询时动态决定要处理哪些内容

具体来说：
1. **全局索引**：只做轻量级的统计索引（比如词频统计、共现关系），几乎不花什么钱
2. **查询时路由**：根据用户问题，动态决定要处理哪些社区
3. **按需计算**：只对相关的社区进行实体和关系的详细计算

LazyGraphRAG的索引成本只有标准GraphRAG的**千分之一**（0.1%）！对于大文档库、低查询频率的场景，简直是神器。

### 5.3 什么时候用标准GraphRAG，什么时候用LazyGraphRAG

| 场景 | 推荐 |
|------|------|
| 查询频率高（每天>1000次） | 标准GraphRAG |
| 查询频率低（每周<100次） | LazyGraphRAG |
| 预算充足 | 标准GraphRAG |
| 预算有限 | LazyGraphRAG |
| 需要快速验证想法 | LazyGraphRAG |

## 六、常见问题与解决方案

### 6.1 中文实体识别效果不好

这是很多同学会遇到的问题。GraphRAG默认的实体抽取prompt是针对英文设计的，直接用中文效果可能不佳。

**解决方案**：

1. 修改实体抽取的prompt，加入中文示例

```yaml
extraction:
  entity_types:
    - 人名
    - 组织机构
    - 技术概念
    - 产品名称
```

2. 或者使用专门针对中文优化的模型

```python
# 使用智源的BGE模型作为embedding
embedding:
  type: ollama
  model: BAAI/bge-m3
```

### 6.2 索引构建太慢

**解决方案**：
- 使用更快的LLM（如GPT-3.5-turbo，虽然效果稍差但速度快）
- 调整chunks大小，块大一点可以减少分块数量
- 使用批量处理，并行抽取多个块的实体

### 6.3 全局搜索结果不准确

**问题表现**：问总结性问题时，AI给的内容要么太宽泛，要么遗漏重要信息。

**解决方案**：
- 调整社区的resolution参数，值越小社区越大，摘要内容越丰富
- 检查社区摘要的质量，如果摘要本身不好，全局搜索效果自然差
- 可以多次查询，用不同的问法来验证结果

## 七、性能优化建议

### 7.1 提升实体抽取质量

1. **提供领域特定的实体类型**：不要只使用通用类型（PERSON、ORG），定义你的业务实体类型（如"产品"、"项目"、"客户"）

2. **few-shot示例**：在prompt中提供几个抽取示例，LLM能更好地理解你的需求

```python
prompt = """
从文本中抽取实体。示例：

文本："腾讯在2023年发布了混元大模型"
实体：[
  {"name": "腾讯", "type": "组织"},
  {"name": "2023年", "type": "时间"},
  {"name": "混元大模型", "type": "产品"}
]

文本：{your_text}
实体：
"""
```

### 7.2 控制成本

1. **先用小数据集测试**：先用一小部分文档测试效果，满意了再处理全部数据

2. **批量处理**：不要一块一块处理，把多个chunk打包一起处理

3. **缓存中间结果**：如果文档会更新，不要每次都重新抽取，只处理变化的部分

### 7.3 提升查询效果

1. **本地搜索适合具体问题**：问"谁、什么、在哪"这类问题用本地搜索

2. **全局搜索适合总结问题**：问"有哪些、怎么样、为什么"这类问题用全局搜索

3. **适当增加chunks的重叠**：这样实体不会被切在边界上

## 八、相关主题

- [[知识库管理]] - RAG知识库的整体架构
- [[LazyGraphRAG详解]] - 轻量级GraphRAG实现
- [[知识图谱构建实战]] - 从零构建知识图谱
- [[Embedding模型选择]] - 文本向量化模型选型
- [[向量数据库对比]] - 底层存储选型

---

> [!note] 更新记录
> - 2026-04-24：全面改写，语言风格优化，增加实战内容
> - 补充LazyGraphRAG的介绍和选型建议
> - 增加常见问题与解决方案
