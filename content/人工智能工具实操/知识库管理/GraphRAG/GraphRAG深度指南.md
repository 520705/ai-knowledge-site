---
title: GraphRAG深度指南
date: 2026-04-18
tags:
  - GraphRAG
  - 知识图谱
  - RAG
  - 检索增强生成
  - 图数据库
  - 索引构建
  - 本地搜索
  - 全局搜索
  - LLM
  - Neo4j
categories:
  - 人工智能/知识库管理
---

# GraphRAG深度指南

> [!abstract] 摘要
> GraphRAG是基于知识图谱的检索增强生成技术，通过将非结构化文本转化为结构化的图谱知识，显著提升RAG系统在复杂推理和跨文档理解方面的能力。本指南将深入讲解GraphRAG的技术起源、核心原理、索引构建流程以及搜索策略。

## 关键词速查表

| 关键词 | 说明 |
|--------|------|
| 实体抽取 | 从文本中识别和提取具有特定类型的实体 |
| 关系抽取 | 识别实体之间的语义关联关系 |
| 社区发现 | 使用Leiden等算法对图谱进行社区划分 |
| 社区摘要 | 为每个社区生成代表性摘要 |
| 本地搜索 | 基于实体和关系进行细粒度检索 |
| 全局搜索 | 基于社区摘要进行全局推理 |
| Map-Reduce | 并行处理后汇总的分布式计算模式 |
| 知识图谱 | 由实体、关系和属性组成的结构化知识表示 |
| 索引构建 | 将原始文档转化为可检索的知识图谱 |
| LLM | 大型语言模型，用于生成和理解内容 |

## 一、技术起源与演进

### 1.1 传统RAG的局限性

传统RAG（Retrieval-Augmented Generation）系统在处理简单的事实查询时表现出色，但在面对复杂推理任务时存在明显不足。当用户提出需要跨多个文档进行综合分析的问题时，传统RAG往往难以准确捕获文档间的深层语义关联。例如，当询问"过去十年人工智能领域有哪些重大突破及其相互影响"这类问题时，传统RAG可能只是简单地拼接相关段落，无法理解技术突破之间的因果关系和时间脉络。

这种局限性的根本原因在于传统RAG依赖向量相似度进行检索，而向量表示本质上是对语义的压缩编码。虽然这种编码保留了主要内容信息，但丢失了结构化的关系知识。知识图谱通过显式建模实体和关系，提供了一种更精确的知识表示方式。

### 1.2 GraphRAG的诞生

GraphRAG由微软研究院于2024年提出，旨在解决传统RAG在全局性问题上的不足。该技术最初在论文"From Local to Global: A GraphRAG Approach to Query-Focused Summarization"中被详细阐述。GraphRAG的核心思想是将非结构化文本转化为知识图谱，然后利用图谱的结构化特性进行更智能的检索和推理。

GraphRAG的设计哲学可以概括为"从局部到全局"。局部层面关注实体和关系的精确匹配，全局层面则通过社区结构进行宏观推理。这种双层架构使GraphRAG既能处理具体的实体查询，也能回答需要综合分析的全局性问题。

### 1.3 技术定位

GraphRAG在RAG技术演进谱系中的定位独具特色。与naive RAG相比，GraphRAG增加了知识图谱构建层；与高级RAG相比，GraphRAG提供了更结构化的知识表示；与模块化RAG相比，GraphRAG具有更明确的知识组织形式。这种定位使GraphRAG成为处理复杂推理任务的理想选择。

## 二、核心原理详解

### 2.1 知识图谱基础

知识图谱是一种用图结构表示知识的模型，由三个核心元素组成：实体（Entity）、关系（Relation）和属性（Attribute）。实体代表现实世界中的具体事物，如"人工智能"、"Transformer架构"；关系描述实体之间的语义关联，如"人工智能""使用了""Transformer架构"；属性则是实体的特征描述。

在GraphRAG中，知识图谱扮演着知识组织和索引的角色。通过将文本中的实体和关系显式抽取出来，系统能够更精确地理解文本内容，并支持复杂的图遍历查询。

```python
# 知识图谱的基本数据结构示例
class Entity:
    def __init__(self, id, type, name, description, source_id):
        self.id = id
        self.type = type  # 如 "PERSON", "ORG", "TECH"
        self.name = name
        self.description = description
        self.source_id = source_id  # 来源文档

class Relation:
    def __init__(self, source_id, target_id, type, description):
        self.source = source_id
        self.target = target_id
        self.type = type  # 如 "works_at", "developed", "uses"
        self.description = description

class KnowledgeGraph:
    def __init__(self):
        self.entities = {}  # id -> Entity
        self.relations = []  # List[Relation]
        self.entity_index = {}  # name -> Entity ids
    
    def add_entity(self, entity):
        self.entities[entity.id] = entity
        if entity.name not in self.entity_index:
            self.entity_index[entity.name] = []
        self.entity_index[entity.name].append(entity.id)
    
    def add_relation(self, relation):
        self.relations.append(relation)
```

### 2.2 图索引构建流程

GraphRAG的索引构建是整个系统的核心，分为四个主要阶段：实体抽取、关系抽取、社区发现和社区摘要生成。每个阶段都依赖大语言模型的能力，同时也需要精心的配置和优化。

### 2.3 检索与生成机制

GraphRAG提供两种互补的检索策略：本地搜索和全局搜索。本地搜索适用于需要深入了解特定实体的问题，通过图遍历找到相关的实体和关系。全局搜索适用于需要综合分析的问题，通过社区结构进行宏观推理。这两种策略的结合使GraphRAG能够处理从简单到复杂的各类查询。

## 三、索引构建详解

### 3.1 实体抽取

实体抽取是GraphRAG索引构建的第一步，目标是从原始文本中识别出具有特定类型的实体。这一过程通常使用大语言模型的函数调用能力，通过预定义的schema指导实体识别。

实体抽取的质量直接影响后续所有环节的效果。高质量的实体应该满足以下标准：准确性（实体确实存在于文本中）、完整性（不遗漏重要实体）、一致性（相同实体在不同文档中使用统一标识）。

```python
import json
from typing import List, Optional

# 实体抽取的prompt设计
ENTITY_EXTRACTION_PROMPT = """
你是一个专业的知识抽取专家。请从以下文本中抽取所有具有意义的实体。

## 实体类型定义
- PERSON: 人物，包括个人姓名、职位称呼
- ORGANIZATION: 组织机构，包括公司、学术机构、政府部门
- TECHNOLOGY: 技术概念，包括算法、框架、工具
- CONCEPT: 抽象概念，包括理论、方法论
- EVENT: 事件，包括会议、发明、发现
- LOCATION: 地点，包括城市、国家、建筑物

## 输出要求
以JSON数组格式输出，每个实体包含：
- name: 实体名称
- type: 实体类型（使用上述类型）
- description: 实体描述（从上下文提炼，50-200字）
- alias: 别名或缩写（如有）

## 示例
输入: "Transformer架构由Google在2017年提出，革新了自然语言处理领域"
输出:
[
  {{
    "name": "Transformer架构",
    "type": "TECHNOLOGY",
    "description": "一种基于自注意力机制的神经网络架构，摒弃了传统的循环神经网络结构，实现了并行计算，大幅提升了序列建模的效率。",
    "alias": ["Transformer"]
  }},
  {{
    "name": "Google",
    "type": "ORGANIZATION",
    "description": "全球领先的科技公司，业务涵盖搜索引擎、云计算、人工智能等多个领域。",
    "alias": ["Alphabet", "谷歌"]
  }},
  {{
    "name": "自然语言处理",
    "type": "CONCEPT",
    "description": "计算机科学和人工智能的交叉领域，致力于让计算机理解和生成人类语言。",
    "alias": ["NLP"]
  }}
]

## 待处理文本
{text}

请抽取所有实体：
"""
```

实体抽取的配置参数需要根据具体场景进行调整。实体类型定义应该覆盖目标领域的主要概念；描述长度应该平衡信息完整性和处理效率；alias列表有助于处理实体的多种表述形式。

### 3.2 关系抽取

关系抽取是在实体抽取的基础上，识别实体之间的语义关联。良好的关系抽取能够揭示实体间的深层联系，构建起完整的知识网络。

关系抽取的挑战在于准确性和粒度控制。过度抽取会导致噪音增加，抽取不足则可能丢失重要信息。实践中，通常采用预定义关系类型列表 + 开放关系抽取的混合策略。

```python
# 关系抽取的配置和实现
RELATION_EXTRACTION_PROMPT = """
你是一个专业的关系抽取专家。请分析以下实体对之间的关系。

## 预定义关系类型
- works_at: 人物在某个组织工作或任职
- developed: 个人或组织开发了某项技术或产品
- uses: 某技术被应用于某领域或被某组织使用
- founded: 某人创立了某组织
- studied: 某人学习了某学科
- located_in: 实体位于某地点
- part_of: 某实体是另一实体的一部分
- related_to: 通用关系，表示实体间存在某种关联
- enabled: 某技术或发展使能了另一技术或发展
- influenced: 某人或某事影响了另一人或另一事

## 输出要求
以JSON数组格式输出，每个关系包含：
- source: 源实体名称
- target: 目标实体名称
- type: 关系类型（优先使用预定义类型）
- description: 关系描述（说明如何得出此结论）

## 实体列表
{entities}

## 上下文文本
{context}

请抽取所有关系：
"""

class RelationExtractor:
    def __init__(self, llm_client):
        self.llm = llm_client
        self.relation_types = [
            "works_at", "developed", "uses", "founded", 
            "studied", "located_in", "part_of", "related_to",
            "enabled", "influenced"
        ]
    
    def extract_relations(self, entities: List[Entity], text: str) -> List[Relation]:
        """从文本中抽取实体之间的关系"""
        # 构建关系抽取prompt
        prompt = RELATION_EXTRACTION_PROMPT.format(
            entities=json.dumps([{
                "name": e.name, 
                "type": e.type, 
                "description": e.description
            } for e in entities], ensure_ascii=False, indent=2),
            context=text
        )
        
        # 调用LLM进行关系抽取
        response = self.llm.generate(prompt)
        relations_data = json.loads(response)
        
        # 将抽取结果转换为Relation对象
        relations = []
        for rel_data in relations_data:
            # 查找源实体和目标实体
            source_entity = self._find_entity_by_name(rel_data["source"])
            target_entity = self._find_entity_by_name(rel_data["target"])
            
            if source_entity and target_entity:
                relations.append(Relation(
                    source_id=source_entity.id,
                    target_id=target_entity.id,
                    type=rel_data["type"],
                    description=rel_data["description"]
                ))
        
        return relations
    
    def _find_entity_by_name(self, name: str) -> Optional[Entity]:
        """根据名称查找实体"""
        if name in self.entity_index:
            return self.entities[self.entity_index[name][0]]
        return None
```

### 3.3 社区发现算法

社区发现是GraphRAG的独特之处，它将知识图谱划分为若干社区，每个社区代表一个语义相近的实体群组。这一步骤对于实现全局搜索至关重要。

GraphRAG采用Leiden算法进行社区发现，这是一个高效的图聚类算法，能够发现层次化的社区结构。Leiden算法相比其前身Louvain算法具有更好的性能保证，能够避免找到退化的社区划分。

```python
import networkx as nx
from collections import defaultdict

class CommunityDetector:
    def __init__(self, graph: nx.Graph):
        self.graph = graph
        self.partition = None
    
    def detect_communities(self, resolution: float = 1.0) -> dict:
        """
        使用Leiden算法检测社区
        resolution参数控制社区粒度：
        - 值越小，社区数量越多，每个社区规模越小
        - 值越大，社区数量越少，每个社区规模越大
        """
        # 使用python-louvain库实现（Leiden的Python实现）
        try:
            import leidenalg
            import igraph as ig
        except ImportError:
            # 回退到NetworkX的Louvain实现
            from networkx.algorithms.community import louvain_communities
            communities = louvain_communities(
                self.graph, 
                resolution=resolution,
                seed=42
            )
            partition = {}
            for idx, community in enumerate(communities):
                for node in community:
                    partition[node] = idx
            self.partition = partition
            return partition
        
        # 使用igraph实现Leiden算法
        ig_graph = ig.Graph.from_networkx(self.graph)
        partition = leidenalg.find_partition(
            ig_graph,
            leidenalg.RBConfigurationVertexPartition,
            resolution_parameter=resolution
        )
        
        # 转换回节点ID
        self.partition = {}
        for idx, community in enumerate(partition):
            for node in community:
                self.partition[node] = idx
        
        return self.partition
    
    def get_hierarchy(self, max_level: int = 3) -> dict:
        """
        构建社区层级结构
        允许在不同粒度级别查看社区
        """
        hierarchy = defaultdict(list)
        
        for node, community_id in self.partition.items():
            level = min(len(str(community_id)) // 3, max_level - 1)
            hierarchy[level].append({
                "node": node,
                "community_id": community_id,
                "community_prefix": str(community_id)[:(level + 1) * 3]
            })
        
        return dict(hierarchy)
    
    def get_community_members(self, community_id: int) -> List[str]:
        """获取指定社区的所有成员"""
        return [
            node for node, cid in self.partition.items() 
            if cid == community_id
        ]
    
    def get_subgraph(self, community_id: int) -> nx.Graph:
        """获取指定社区的子图"""
        members = self.get_community_members(community_id)
        return self.graph.subgraph(members)
```

### 3.4 社区摘要生成

社区摘要是为每个社区生成的代表性描述，用于支持全局搜索时的推理。社区摘要需要捕捉该社区的核心主题、主要实体以及它们之间的关联。

```python
class CommunitySummarizer:
    def __init__(self, llm_client):
        self.llm = llm_client
    
    def generate_summary(self, community_id: int, subgraph: nx.Graph, 
                         all_entities: dict, all_relations: list) -> str:
        """
        为社区生成摘要
        
        摘要应该包含：
        1. 社区的核心主题（1-2句话）
        2. 主要实体列表及其角色
        3. 重要的关系和交互
        4. 该社区与其他社区的主要联系
        """
        # 收集社区信息
        nodes = list(subgraph.nodes())
        edges = list(subgraph.edges())
        
        # 构建上下文信息
        entity_context = []
        for node_id in nodes:
            entity = all_entities.get(node_id)
            if entity:
                entity_context.append({
                    "name": entity.name,
                    "type": entity.type,
                    "description": entity.description,
                    "connections": len(subgraph.edges(node_id))
                })
        
        # 按连接数排序实体
        entity_context.sort(key=lambda x: x["connections"], reverse=True)
        
        # 构建关系上下文
        relation_context = []
        for src, tgt in edges:
            src_entity = all_entities.get(src)
            tgt_entity = all_entities.get(tgt)
            relation = self._find_relation(src, tgt, all_relations)
            if src_entity and tgt_entity:
                relation_context.append({
                    "from": src_entity.name,
                    "to": tgt_entity.name,
                    "type": relation.type if relation else "related_to",
                    "description": relation.description if relation else ""
                })
        
        # 生成摘要prompt
        prompt = f"""
## 任务
为以下知识图谱社区生成简洁而有信息量的摘要。

## 社区ID
{community_id}

## 主要实体（按重要性排序）
{json.dumps(entity_context[:10], ensure_ascii=False, indent=2)}

## 重要关系
{json.dumps(relation_context[:15], ensure_ascii=False, indent=2)}

## 摘要要求
1. 用2-3句话概括该社区的核心主题和关注领域
2. 列出3-5个最重要的实体及其角色
3. 说明该社区内的主要交互模式
4. 使用专业但易懂的语言

请生成摘要：
"""
        
        summary = self.llm.generate(prompt)
        return summary
    
    def _find_relation(self, source_id: str, target_id: str, 
                       relations: list) -> 'Relation':
        """查找两个实体之间的关系"""
        for rel in relations:
            if (rel.source == source_id and rel.target == target_id) or \
               (rel.source == target_id and rel.target == source_id):
                return rel
        return None
    
    def batch_generate_summaries(self, communities: dict, 
                                  all_entities: dict, 
                                  all_relations: list) -> dict:
        """批量生成社区摘要"""
        summaries = {}
        
        for community_id, subgraph in communities.items():
            summaries[community_id] = self.generate_summary(
                community_id, subgraph, all_entities, all_relations
            )
        
        return summaries
```

## 四、搜索策略

### 4.1 本地搜索

本地搜索是GraphRAG的细粒度检索策略，适用于需要深入了解特定实体或主题的问题。本地搜索的核心思想是以用户查询中识别出的实体为起点，通过图遍历找到相关实体和关系。

本地搜索的工作流程如下：首先使用LLM从用户查询中提取实体；然后以这些实体为种子，在知识图谱中进行扩展检索；接着将检索到的相关上下文组织成prompt；最后让LLM基于上下文生成回答。

```python
class LocalSearch:
    def __init__(self, knowledge_graph: KnowledgeGraph, llm_client):
        self.kg = knowledge_graph
        self.llm = llm_client
        self.max_hops = 3  # 最大跳数
        self.max_nodes = 100  # 最大节点数
    
    def search(self, query: str) -> str:
        """执行本地搜索"""
        # 步骤1: 从查询中提取实体
        query_entities = self._extract_entities_from_query(query)
        
        # 步骤2: 图遍历收集相关上下文
        context = self._collect_context(query_entities)
        
        # 步骤3: 生成回答
        response = self._generate_response(query, context)
        
        return response
    
    def _extract_entities_from_query(self, query: str) -> List[str]:
        """从查询中提取实体名称"""
        prompt = f"""
从以下用户查询中提取所有实体名称。只输出实体名称列表，用换行分隔。

查询: {query}

实体名称:
"""
        response = self.llm.generate(prompt)
        entities = [e.strip() for e in response.split('\n') if e.strip()]
        return entities
    
    def _collect_context(self, query_entities: List[str]) -> str:
        """收集相关上下文"""
        collected_nodes = set()
        collected_edges = []
        
        # BFS遍历
        queue = []
        for entity_name in query_entities:
            if entity_name in self.kg.entity_index:
                for entity_id in self.kg.entity_index[entity_name]:
                    queue.append((entity_id, 0))  # (entity_id, hop_count)
        
        visited = set()
        
        while queue and len(collected_nodes) < self.max_nodes:
            current_id, hop = queue.pop(0)
            
            if current_id in visited or hop > self.max_hops:
                continue
            
            visited.add(current_id)
            
            if current_id in self.kg.entities:
                entity = self.kg.entities[current_id]
                collected_nodes.add(current_id)
                
                # 收集相邻关系
                for relation in self.kg.relations:
                    if relation.source == current_id:
                        collected_edges.append(relation)
                        if relation.target not in visited:
                            queue.append((relation.target, hop + 1))
                    elif relation.target == current_id:
                        collected_edges.append(relation)
                        if relation.source not in visited:
                            queue.append((relation.source, hop + 1))
        
        # 构建上下文文本
        context_parts = []
        
        # 添加实体信息
        context_parts.append("## 相关实体\n")
        for node_id in collected_nodes:
            entity = self.kg.entities.get(node_id)
            if entity:
                context_parts.append(f"**{entity.name}** ({entity.type})")
                context_parts.append(f"  {entity.description}")
                context_parts.append("")
        
        # 添加关系信息
        context_parts.append("\n## 关系\n")
        for relation in collected_edges:
            src = self.kg.entities.get(relation.source)
            tgt = self.kg.entities.get(relation.target)
            if src and tgt:
                context_parts.append(f"- {src.name} --[{relation.type}]--> {tgt.name}")
        
        return "\n".join(context_parts)
    
    def _generate_response(self, query: str, context: str) -> str:
        """基于上下文生成回答"""
        prompt = f"""
## 用户查询
{query}

## 知识图谱上下文
{context}

## 回答要求
1. 基于提供的上下文信息回答用户问题
2. 如果上下文中没有相关信息，说明无法回答
3. 引用上下文中的实体和关系来支持回答
4. 保持回答的准确性和连贯性

请回答：
"""
        
        return self.llm.generate(prompt)
```

### 4.2 全局搜索

全局搜索是GraphRAG的宏观推理策略，适用于需要综合多个信息来源的问题。全局搜索利用社区摘要进行推理，能够处理传统RAG难以回答的全局性问题。

全局搜索的实现基于Map-Reduce模式。Map阶段为每个相关社区生成中间答案；Reduce阶段将这些中间答案综合成最终回答。这种并行处理方式使全局搜索能够高效处理大规模知识图谱。

```python
class GlobalSearch:
    def __init__(self, community_summaries: dict, llm_client):
        self.summaries = community_summaries
        self.llm = llm_client
        self.community_cutoff = 0.5  # 相关性阈值
    
    def search(self, query: str) -> str:
        """执行全局搜索"""
        # 步骤1: 评估每个社区与查询的相关性
        relevant_communities = self._rank_communities(query)
        
        # 步骤2: Map阶段 - 为每个相关社区生成中间答案
        intermediate_answers = self._map_phase(query, relevant_communities)
        
        # 步骤3: Reduce阶段 - 综合生成最终答案
        final_answer = self._reduce_phase(query, intermediate_answers)
        
        return final_answer
    
    def _rank_communities(self, query: str) -> List[tuple]:
        """对社区进行相关性排序"""
        community_scores = []
        
        for community_id, summary in self.summaries.items():
            score = self._calculate_relevance(query, summary)
            community_scores.append((community_id, summary, score))
        
        # 过滤低相关性社区并排序
        relevant = [
            (cid, summary, score) 
            for cid, summary, score in community_scores 
            if score >= self.community_cutoff
        ]
        relevant.sort(key=lambda x: x[2], reverse=True)
        
        return relevant
    
    def _calculate_relevance(self, query: str, summary: str) -> float:
        """计算社区摘要与查询的相关性"""
        # 使用简单的关键词匹配
        query_words = set(query.lower().split())
        summary_words = set(summary.lower().split())
        
        # 计算Jaccard相似度
        intersection = len(query_words & summary_words)
        union = len(query_words | summary_words)
        
        return intersection / union if union > 0 else 0
    
    def _map_phase(self, query: str, communities: List[tuple]) -> List[str]:
        """Map阶段: 为每个社区生成中间答案"""
        answers = []
        
        for community_id, summary, score in communities[:10]:  # 最多处理10个社区
            prompt = f"""
## 用户查询
{query}

## 社区摘要
{summary}

## Map任务
基于上述社区摘要，回答用户查询。只输出该社区能够回答的部分。

回答要求：
1. 如果社区摘要与查询相关，详细回答
2. 如果不相关，输出"[不相关]"

回答：
"""
            answer = self.llm.generate(prompt)
            if not answer.startswith("[不相关]"):
                answers.append(answer)
        
        return answers
    
    def _reduce_phase(self, query: str, intermediate_answers: List[str]) -> str:
        """Reduce阶段: 综合中间答案"""
        if not intermediate_answers:
            return "未找到相关信息"
        
        combined_answers = "\n\n---\n\n".join(intermediate_answers)
        
        prompt = f"""
## 用户查询
{query}

## 来自不同社区的中间答案
{combined_answers}

## Reduce任务
综合以上答案，形成完整、连贯的回答。

要求：
1. 整合所有相关信息
2. 去除重复内容
3. 按照逻辑顺序组织
4. 保持回答的完整性

请生成最终答案：
"""
        
        return self.llm.generate(prompt)
```

## 五、实战配置示例

### 5.1 settings.yaml配置

GraphRAG的配置文件通常使用YAML格式，主要包含LLM配置、Embedding配置、文本处理配置等部分。

```yaml
# settings.yaml 示例配置
llm:
  type: "azure_openai"  # 或 "openai", "anthropic", "ollama"
  model: "gpt-4-turbo"
  api_base: "https://your-resource.openai.azure.com"
  api_key: "${GRAPHRAG_API_KEY}"
  max_tokens: 2000
  temperature: 0.0

embedding:
  type: "azure_openai"  # 或 "openai", "ollama"
  model: "text-embedding-3-large"
  api_base: "https://your-resource.openai.azure.com"
  api_key: "${GRAPHRAG_API_KEY}"
  batch_size: 100
  vector_size: 3072

chunks:
  size: 300
  overlap: 100
  group_by_columns: true

extraction:
  entity_types:
    - "PERSON"
    - "ORGANIZATION"
    - "TECHNOLOGY"
    - "CONCEPT"
    - "EVENT"
    - "LOCATION"
  
  # 关系类型配置
  relation_types:
    - "works_at"
    - "developed"
    - "uses"
    - "founded"
    - "related_to"
    - "enabled"
    - "influenced"

community:
  # Leiden算法参数
  resolution: 1.0
  max_graph_size: 10000

summarization:
  max_input_length: 5000
  prompt_version: "default"

storage:
  # 存储配置
  indexed_root: "./output"
  entity_store: "file:///entity.parquet"
  relation_store: "file:///relation.parquet"
  community_store: "file:///community.parquet"
```

### 5.2 索引构建命令

```bash
# 安装graphrag（如果使用官方实现）
pip install graphrag

# 初始化项目
graphrag init --root ./my_graphrag_project

# 配置API密钥
export GRAPHRAG_API_KEY="your-api-key"

# 运行索引构建
graphrag index --root ./my_graphrag_project

# 查看索引进度
graphrag status --root ./my_graphrag_project
```

### 5.3 查询命令

```bash
# 全局搜索
graphrag query --root ./my_graphrag_project \
  --method global \
  --query "分析人工智能在过去十年的主要发展趋势"

# 本地搜索
graphrag query --root ./my_graphrag_project \
  --method local \
  --query "Transformer架构是如何工作的"
```

## 六、性能优化与最佳实践

### 6.1 实体抽取优化

实体抽取的质量直接决定GraphRAG的整体效果。优化策略包括：设计领域特定的实体类型体系、迭代优化抽取prompt、实现实体消歧机制。对于专业领域，建议预先构建一份核心实体列表，用于指导抽取和进行后处理验证。

### 6.2 社区划分调优

社区划分的参数需要根据知识图谱的规模进行调整。对于小型图谱（<10000节点），可以使用较大的resolution值；对于大型图谱，建议使用较小的resolution以获得更细粒度的社区划分。同时，应该关注社区大小的分布，避免出现极不均衡的情况。

### 6.3 搜索策略选择

本地搜索和全局搜索各有适用场景。本地搜索适合处理具体实体的查询、实体间关系的问题、需要深入了解特定主题的问题。全局搜索适合处理比较性问题（如"对比A和B"）、趋势分析问题、需要综合多个方面的综述性问题。在实际应用中，可以根据查询类型自动选择合适的策略。

## 七、相关主题链接

- [[GraphRAG本地部署]] - 详细的部署配置指南
- [[LazyGraphRAG详解]] - GraphRAG的轻量级变体
- [[知识图谱构建实战]] - 从零构建知识图谱
- [[向量数据库对比]] - GraphRAG的底层存储选型
- [[重排技术深度指南]] - 搜索结果优化
- [[Embedding模型选择]] - 文本向量化方案

---

> [!note] 更新日志
> - 2026-04-18: 初始版本完成
> - 包含完整的索引构建流程和搜索策略详解
