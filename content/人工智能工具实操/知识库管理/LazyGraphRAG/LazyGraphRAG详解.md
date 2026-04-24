---
title: LazyGraphRAG详解
date: 2026-04-18
tags:
  - LazyGraphRAG
  - 图检索增强生成
  - 懒加载
  - 指标计算
  - 动态查询
  - 社区选择
  - GraphRAG
  - 索引优化
  - 成本优化
  - 流式处理
categories:
  - 人工智能/知识库管理
aliases:
  - Lazy GraphRAG
  - 懒加载GraphRAG
---

# LazyGraphRAG详解

> [!abstract] 摘要
> LazyGraphRAG是GraphRAG的高效变体，通过延迟计算和动态查询优化策略，在保持GraphRAG核心能力的同时显著降低计算成本。本文档深入解析LazyGraphRAG的技术原理、懒加载机制、指标计算方法、社区选择策略以及动态查询路由机制，为实际部署提供详尽的技术指导。

## 关键词速查表

| 关键词 | 说明 |
|--------|------|
| 懒加载 | 延迟实体和关系的计算，仅在需要时进行计算 |
| 指标计算 | 使用预计算指标评估节点和社区的重要性 |
| 社区选择 | 根据查询相关性动态选择目标社区 |
| 动态查询 | 根据查询特征自适应选择检索策略 |
| 全局索引 | 社区级别的轻量级索引结构 |
| 局部计算 | 仅对检索到的局部图进行详细计算 |
| 成本优化 | 减少LLM调用次数和Token消耗 |
| 流式处理 | 支持增量式索引构建和更新 |
| 混合检索 | 结合向量检索和图检索的优势 |
| 层次索引 | 多粒度的社区层级结构 |

## 一、技术背景与设计动机

### 1.1 GraphRAG的计算成本问题

传统GraphRAG虽然在复杂推理任务上表现出色，但其索引构建阶段的计算成本令许多实际应用望而却步。标准GraphRAG需要对整个语料库进行全面的实体抽取、关系抽取和社区发现，这个过程涉及大量LLM调用。以一个包含100万Token的语料库为例，完整的GraphRAG索引构建可能需要数千次LLM调用，成本可达数百美元。

这种高昂成本的根源在于GraphRAG的设计哲学：预先计算一切可能的知识结构，以便后续查询能够快速响应。然而，在许多实际场景中，用户只关心语料库中的一小部分内容，大量预计算的知识从未被使用，造成了严重的资源浪费。

### 1.2 LazyGraphRAG的设计哲学

LazyGraphRAG由微软研究院于2024年底提出，其核心理念是"只在需要时计算"。与预计算所有知识的传统方法不同，LazyGraphRAG采用延迟计算策略，仅在处理具体查询时才进行实体和关系的详细计算。

这种设计带来了显著的优势：首先，索引构建成本大幅降低，从O(N)降低到接近O(1)的固定成本；其次，索引可以增量更新，无需重新处理整个语料库；最后，系统能够自适应地聚焦于与查询相关的知识区域，提高检索效率。

> [!tip] 核心洞察
> LazyGraphRAG的本质是用"查询时计算"换取"索引时计算"，通过将计算负载从索引阶段转移到查询阶段，实现了成本的动态分配。

### 1.3 适用场景分析

LazyGraphRAG特别适合以下场景：知识库规模大但查询频率相对较低的应用；预算有限但需要GraphRAG能力的团队；需要频繁更新索引的动态知识库；以及快速原型验证和迭代开发阶段。相反，如果查询频率极高且对延迟要求严格，传统GraphRAG或其他优化方案可能更为合适。

## 二、懒加载机制详解

### 2.1 全局索引与局部索引

LazyGraphRAG采用双层索引架构，将索引分为全局索引和局部索引两部分。全局索引是轻量级的，使用统计方法而非LLM计算，包含文档级别的统计信息和简单的社区结构。局部索引则是按需计算的，仅针对查询相关的内容进行详细计算。

```python
from dataclasses import dataclass, field
from typing import List, Dict, Set, Optional
import numpy as np
from collections import defaultdict

@dataclass
class GlobalIndex:
    """LazyGraphRAG的全局索引结构"""
    # 文档级别的统计信息
    doc_count: int = 0
    total_tokens: int = 0
    
    # 文档内容指纹（用于快速相似度计算）
    doc_signatures: Dict[str, np.ndarray] = field(default_factory=dict)
    
    # 文档-社区映射
    doc_community_mapping: Dict[str, int] = field(default_factory=dict)
    
    # 社区统计信息
    community_stats: Dict[int, Dict] = field(default_factory=dict)
    
    # 词频统计（用于全局TF-IDF）
    global_term_freq: Dict[str, int] = field(default_factory=dict)
    doc_term_freq: Dict[str, Dict[str, int]] = field(default_factory=dict)
    
    def compute_doc_signature(self, doc_id: str, embedding: np.ndarray, 
                             n_bits: int = 128) -> np.ndarray:
        """
        计算文档的局部敏感哈希(LSH)签名
        用于快速的近似相似度搜索
        """
        # 使用随机投影生成LSH签名
        np.random.seed(hash(doc_id) % (2**32))
        random_vectors = np.random.randn(embedding.shape[0], n_bits)
        
        # 二值化投影
        signature = (embedding @ random_vectors > 0).astype(np.uint8)
        self.doc_signatures[doc_id] = signature
        return signature
    
    def build_term_statistics(self, doc_id: str, tokens: List[str]):
        """构建词频统计信息"""
        self.doc_count += 1
        self.total_tokens += len(tokens)
        
        # 文档级别词频
        term_counts = defaultdict(int)
        for token in tokens:
            term_counts[token] += 1
            self.global_term_freq[token] = self.global_term_freq.get(token, 0) + 1
        
        self.doc_term_freq[doc_id] = dict(term_counts)
    
    def compute_tfidf(self, doc_id: str, term: str) -> float:
        """计算TF-IDF分数"""
        if doc_id not in self.doc_term_freq:
            return 0.0
        
        tf = self.doc_term_freq[doc_id].get(term, 0)
        df = sum(1 for doc_terms in self.doc_term_freq.values() 
                 if term in doc_terms)
        
        if df == 0:
            return 0.0
        
        idf = np.log(self.doc_count / df)
        return tf * idf
```

### 2.2 实体签名的生成

在局部索引中，LazyGraphRAG需要为实体生成签名以支持快速匹配。实体签名是一种紧凑的表示，能够捕捉实体的语义特征和上下文信息。与完整Embedding相比，签名具有存储成本低、比较速度快的优势。

```python
from typing import Set, Tuple

class EntitySignature:
    """实体签名生成器"""
    
    def __init__(self, embedding_model, signature_size: int = 64):
        self.embedding_model = embedding_model
        self.signature_size = signature_size
    
    def generate_signature(self, entity_name: str, 
                         context_texts: List[str]) -> np.ndarray:
        """
        生成实体签名
        
        签名由以下部分组成：
        1. 实体名称的Embedding向量
        2. 上下文关键词的TF-IDF加权向量
        3. 实体类型的One-hot编码
        """
        # 1. 实体名称的Embedding
        name_embedding = self.embedding_model.embed(entity_name)
        
        # 2. 上下文关键词
        context_keywords = self._extract_keywords(context_texts)
        keyword_embedding = self._aggregate_keyword_embeddings(context_keywords)
        
        # 3. 组合签名
        signature = np.concatenate([
            name_embedding[:self.signature_size // 2],
            keyword_embedding[:self.signature_size // 2]
        ])
        
        return signature
    
    def _extract_keywords(self, texts: List[str], top_k: int = 20) -> List[str]:
        """提取关键词（简单实现）"""
        word_freq = defaultdict(int)
        stop_words = {'的', '了', '在', '是', '我', '有', '和', '就'}
        
        for text in texts:
            words = text.replace('\n', ' ').split()
            for word in words:
                if len(word) > 1 and word not in stop_words:
                    word_freq[word] += 1
        
        return sorted(word_freq.items(), key=lambda x: x[1], 
                     reverse=True)[:top_k]
    
    def _aggregate_keyword_embeddings(self, keywords: List[str]) -> np.ndarray:
        """聚合关键词Embedding"""
        if not keywords:
            return np.zeros(self.embedding_model.dimension)
        
        embeddings = []
        for word, _ in keywords:
            emb = self.embedding_model.embed(word)
            embeddings.append(emb)
        
        return np.mean(embeddings, axis=0)
```

### 2.3 增量式索引更新

LazyGraphRAG支持增量更新，这意味着当新文档加入知识库时，无需重新处理整个语料库。系统只需为新文档构建全局索引条目，并在查询时按需计算局部索引。

```python
class IncrementalIndexer:
    """增量索引器"""
    
    def __init__(self, global_index: GlobalIndex, 
                 embedding_model, llm_client):
        self.global_index = global_index
        self.embedding_model = embedding_model
        self.llm_client = llm_client
        
        # 局部索引缓存
        self.local_index_cache: Dict[str, Dict] = {}
        self.cache_max_size = 1000
    
    def add_document(self, doc_id: str, content: str, 
                    metadata: Optional[Dict] = None):
        """
        添加新文档到全局索引
        
        这个过程只涉及轻量级计算，不调用LLM
        """
        # 1. 分词
        tokens = self._tokenize(content)
        
        # 2. 构建词频统计
        self.global_index.build_term_statistics(doc_id, tokens)
        
        # 3. 计算文档签名
        embedding = self.embedding_model.embed(content)
        self.global_index.compute_doc_signature(doc_id, embedding)
        
        # 4. 初步社区分配（基于简单的聚类）
        community_id = self._assign_community(embedding)
        self.global_index.doc_community_mapping[doc_id] = community_id
        
        # 5. 更新社区统计
        self._update_community_stats(community_id)
    
    def _tokenize(self, text: str) -> List[str]:
        """简单分词"""
        import re
        # 保留中英文和数字
        tokens = re.findall(r'[\w]+', text.lower())
        return tokens
    
    def _assign_community(self, embedding: np.ndarray) -> int:
        """
        基于文档Embedding将文档分配到社区
        使用简单的K-means聚类
        """
        # 简化实现：使用哈希
        return hash(str(embedding.sum())) % 100
    
    def _update_community_stats(self, community_id: int):
        """更新社区统计信息"""
        if community_id not in self.global_index.community_stats:
            self.global_index.community_stats[community_id] = {
                'doc_count': 0,
                'avg_embedding': np.zeros(128),
                'term_diversity': 0
            }
        
        stats = self.global_index.community_stats[community_id]
        n = stats['doc_count']
        stats['doc_count'] += 1
        # 增量更新平均Embedding
        # stats['avg_embedding'] = (stats['avg_embedding'] * n + new_embedding) / (n + 1)
```

## 三、指标计算体系

### 3.1 社区重要性指标

LazyGraphRAG使用一系列预计算的统计指标来评估社区的重要性，这些指标无需LLM即可计算，却能够有效反映社区的知识价值。

```python
from dataclasses import dataclass
from typing import List, Tuple

@dataclass
class CommunityMetrics:
    """社区指标计算器"""
    global_index: GlobalIndex
    
    def compute_community_importance(self, community_id: int) -> float:
        """
        计算社区重要性分数
        
        综合考虑：
        1. 文档数量（规模）
        2. 内容独特性（与其他社区的差异）
        3. 信息密度（关键词覆盖率）
        """
        stats = self.global_index.community_stats.get(community_id, {})
        
        # 规模分数
        doc_count = stats.get('doc_count', 0)
        scale_score = np.log1p(doc_count)
        
        # 独特性分数
        distinctiveness = self._compute_distinctiveness(community_id)
        
        # 信息密度分数
        density = self._compute_information_density(community_id)
        
        # 综合分数（加权组合）
        importance = (
            0.3 * scale_score +
            0.4 * distinctiveness +
            0.3 * density
        )
        
        return importance
    
    def _compute_distinctiveness(self, community_id: int) -> float:
        """
        计算社区的独特性分数
        衡量该社区与其他社区的差异程度
        """
        if community_id not in self.global_index.community_stats:
            return 0.0
        
        # 获取该社区的文档
        community_docs = [
            doc_id for doc_id, cid 
            in self.global_index.doc_community_mapping.items()
            if cid == community_id
        ]
        
        if len(community_docs) < 2:
            return 0.5
        
        # 计算文档间的平均距离
        signatures = [
            self.global_index.doc_signatures.get(doc_id)
            for doc_id in community_docs
            if doc_id in self.global_index.doc_signatures
        ]
        
        if len(signatures) < 2:
            return 0.5
        
        # 计算成对距离的平均值
        distances = []
        for i in range(len(signatures)):
            for j in range(i + 1, len(signatures)):
                dist = np.mean(signatures[i] != signatures[j])
                distances.append(dist)
        
        return np.mean(distances)
    
    def _compute_information_density(self, community_id: int) -> float:
        """
        计算信息密度
        基于词频多样性和术语覆盖度
        """
        # 获取社区文档的术语集合
        community_terms = set()
        community_docs = [
            doc_id for doc_id, cid 
            in self.global_index.doc_community_mapping.items()
            if cid == community_id
        ]
        
        for doc_id in community_docs:
            if doc_id in self.global_index.doc_term_freq:
                community_terms.update(
                    self.global_index.doc_term_freq[doc_id].keys()
                )
        
        # 与全局术语集的比较
        if not self.global_index.global_term_freq:
            return 0.0
        
        coverage = len(community_terms) / len(self.global_index.global_term_freq)
        
        # 计算信息熵相关的密度指标
        term_freqs = [
            self.global_index.global_term_freq[term] 
            for term in community_terms
        ]
        
        if not term_freqs:
            return 0.0
        
        # 归一化频率
        total = sum(term_freqs)
        probs = [f / total for f in term_freqs]
        
        # 计算熵
        entropy = -sum(p * np.log2(p) for p in probs if p > 0)
        max_entropy = np.log2(len(probs)) if len(probs) > 0 else 1
        
        normalized_entropy = entropy / max_entropy if max_entropy > 0 else 0
        
        return 0.5 * coverage + 0.5 * normalized_entropy
    
    def rank_communities(self, query: str, top_k: int = 10) -> List[Tuple[int, float]]:
        """
        根据查询相关性对社区进行排序
        
        使用轻量级特征匹配而非LLM
        """
        query_terms = set(query.lower().split())
        community_scores = []
        
        for community_id in self.global_index.community_stats.keys():
            # 计算查询词在社区中的覆盖度
            coverage = self._calculate_query_coverage(community_id, query_terms)
            
            # 结合重要性分数
            importance = self.compute_community_importance(community_id)
            
            # 综合分数
            score = 0.6 * coverage + 0.4 * importance
            community_scores.append((community_id, score))
        
        # 排序
        community_scores.sort(key=lambda x: x[1], reverse=True)
        return community_scores[:top_k]
    
    def _calculate_query_coverage(self, community_id: int, 
                                  query_terms: Set[str]) -> float:
        """计算查询词在社区中的覆盖程度"""
        community_terms = set()
        
        for doc_id, cid in self.global_index.doc_community_mapping.items():
            if cid == community_id and doc_id in self.global_index.doc_term_freq:
                community_terms.update(
                    self.global_index.doc_term_freq[doc_id].keys()
                )
        
        if not query_terms:
            return 0.0
        
        intersection = len(query_terms & community_terms)
        return intersection / len(query_terms)
```

### 3.2 节点重要性指标

在局部索引中，LazyGraphRAG需要评估实体节点的重要性。以下指标用于指导社区选择后的详细检索。

```python
class NodeMetrics:
    """节点指标计算"""
    
    def __init__(self, graph: Optional['nx.Graph'] = None):
        self.graph = graph
    
    def compute_degree_centrality(self, node_id: str) -> float:
        """
        计算度中心性
        节点的连接数量
        """
        if self.graph and node_id in self.graph:
            return self.graph.degree(node_id)
        return 0
    
    def compute_betweenness_centrality(self, node_id: str) -> float:
        """
        计算介数中心性
        节点在最短路径中出现的频率
        """
        if self.graph:
            try:
                from networkx.algorithms.centrality import betweenness_centrality
                centrality = betweenness_centrality(self.graph)
                return centrality.get(node_id, 0)
            except:
                pass
        return 0
    
    def compute_pagerank(self, node_id: str, damping: float = 0.85, 
                        iterations: int = 100) -> float:
        """
        计算PageRank分数
        迭代算法，节点的重要性由指向它的节点数量和质量决定
        """
        if not self.graph or self.graph.number_of_nodes() == 0:
            return 0
        
        n = self.graph.number_of_nodes()
        if n == 1:
            return 1.0
        
        # 初始化PageRank
        pagerank = {node: 1 / n for node in self.graph.nodes()}
        
        for _ in range(iterations):
            new_pagerank = {}
            
            for node in self.graph.nodes():
                # 收集所有指向该节点的节点的PageRank
                incoming = 0
                predecessors = list(self.graph.predecessors(node)) if self.graph.is_directed() else self.graph.neighbors(node)
                
                for predecessor in predecessors:
                    out_degree = self.graph.out_degree(predecessor) if self.graph.is_directed() else self.graph.degree(predecessor)
                    if out_degree > 0:
                        incoming += pagerank[predecessor] / out_degree
                
                new_pagerank[node] = (1 - damping) / n + damping * incoming
            
            pagerank = new_pagerank
        
        return pagerank.get(node_id, 0)
    
    def compute_entity_relevance(self, node_id: str, query_embedding: np.ndarray,
                                alpha: float = 0.3) -> float:
        """
        计算实体与查询的相关性
        
        综合考虑：
        - 向量相似度
        - 结构重要性（PageRank）
        - 类型匹配度
        """
        # 获取实体的Embedding（延迟计算）
        entity_embedding = self._get_entity_embedding(node_id)
        
        if entity_embedding is None:
            return 0
        
        # 向量相似度
        vector_sim = self._cosine_similarity(query_embedding, entity_embedding)
        
        # PageRank
        pr_score = self.compute_pagerank(node_id)
        
        # 综合
        return alpha * vector_sim + (1 - alpha) * pr_score
    
    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        """计算余弦相似度"""
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        
        if norm_a == 0 or norm_b == 0:
            return 0
        
        return np.dot(a, b) / (norm_a * norm_b)
    
    def _get_entity_embedding(self, node_id: str) -> Optional[np.ndarray]:
        """延迟获取实体Embedding"""
        # 这里应该从缓存或计算中获取
        # 简化实现
        return None
```

## 四、社区选择策略

### 4.1 基于查询的社区路由

LazyGraphRAG的核心查询流程始于社区选择。系统根据用户查询的特征，从全局索引中选择最相关的社区集合，然后仅对这些社区进行深入的局部索引计算。

```python
class CommunityRouter:
    """社区路由器 - 根据查询选择目标社区"""
    
    def __init__(self, global_index: GlobalIndex, 
                 embedding_model, metrics: CommunityMetrics):
        self.global_index = global_index
        self.embedding_model = embedding_model
        self.metrics = metrics
        
        # 预计算的社区嵌入
        self.community_embeddings: Dict[int, np.ndarray] = {}
    
    def route(self, query: str, top_k: int = 5, 
             min_relevance: float = 0.1) -> List[int]:
        """
        路由查询到相关社区
        
        返回最相关的k个社区ID列表
        """
        # 策略1: 关键词匹配
        keyword_matches = self._keyword_based_routing(query)
        
        # 策略2: 向量相似度
        vector_matches = self._vector_based_routing(query)
        
        # 策略3: 统计特征
        statistical_matches = self._statistical_routing(query)
        
        # 融合策略
        return self._fuse_results(
            keyword_matches, 
            vector_matches, 
            statistical_matches,
            top_k=top_k,
            min_relevance=min_relevance
        )
    
    def _keyword_based_routing(self, query: str) -> List[Tuple[int, float]]:
        """基于关键词的路由"""
        query_terms = set(query.lower().split())
        scores = []
        
        for community_id in self.global_index.community_stats.keys():
            coverage = self.metrics._calculate_query_coverage(
                community_id, query_terms
            )
            if coverage > 0:
                scores.append((community_id, coverage))
        
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores
    
    def _vector_based_routing(self, query: str) -> List[Tuple[int, float]]:
        """基于向量相似度的路由"""
        query_embedding = self.embedding_model.embed(query)
        scores = []
        
        for community_id in self.global_index.community_stats.keys():
            community_emb = self._get_community_embedding(community_id)
            if community_emb is not None:
                similarity = self._cosine_similarity(query_embedding, community_emb)
                scores.append((community_id, similarity))
        
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores
    
    def _statistical_routing(self, query: str) -> List[Tuple[int, float]]:
        """基于统计特征的路由"""
        # 使用TF-IDF计算查询与各社区的相似度
        scores = []
        
        for community_id in self.global_index.community_stats.keys():
            score = self._compute_tfidf_similarity(query, community_id)
            scores.append((community_id, score))
        
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores
    
    def _compute_tfidf_similarity(self, query: str, community_id: int) -> float:
        """计算查询与社区的TF-IDF相似度"""
        query_terms = query.lower().split()
        
        # 收集社区文档
        community_docs = [
            doc_id for doc_id, cid 
            in self.global_index.doc_community_mapping.items()
            if cid == community_id
        ]
        
        if not community_docs:
            return 0.0
        
        # 计算查询与社区的平均TF-IDF分数
        scores = []
        for term in query_terms:
            # 计算该词在社区中的TF-IDF
            community_tfidf = 0
            for doc_id in community_docs:
                community_tfidf += self.global_index.compute_tfidf(doc_id, term)
            
            community_tfidf /= len(community_docs)
            scores.append(community_tfidf)
        
        return np.mean(scores) if scores else 0.0
    
    def _get_community_embedding(self, community_id: int) -> Optional[np.ndarray]:
        """获取社区Embedding（延迟计算）"""
        if community_id in self.community_embeddings:
            return self.community_embeddings[community_id]
        
        # 获取社区文档
        community_docs = [
            doc_id for doc_id, cid 
            in self.global_index.doc_community_mapping.items()
            if cid == community_id
        ]
        
        if not community_docs:
            return None
        
        # 聚合文档Embedding
        embeddings = [
            self.global_index.doc_signatures.get(doc_id).astype(float)
            for doc_id in community_docs
            if doc_id in self.global_index.doc_signatures
        ]
        
        if not embeddings:
            return None
        
        community_emb = np.mean(embeddings, axis=0)
        self.community_embeddings[community_id] = community_emb
        
        return community_emb
    
    def _fuse_results(self, keyword_scores: List[Tuple[int, float]],
                     vector_scores: List[Tuple[int, float]],
                     stat_scores: List[Tuple[int, float]],
                     top_k: int, min_relevance: float) -> List[int]:
        """
        融合多种路由策略的结果
        
        使用 Reciprocal Rank Fusion (RRF)
        """
        # 归一化分数到[0,1]
        def normalize(scores):
            if not scores:
                return {}
            max_score = max(s for _, s in scores)
            if max_score == 0:
                return {cid: 0 for cid, _ in scores}
            return {cid: s / max_score for cid, s in scores}
        
        norm_keyword = normalize(keyword_scores)
        norm_vector = normalize(vector_scores)
        norm_stat = normalize(stat_scores)
        
        # 获取所有社区ID
        all_communities = set(norm_keyword.keys()) | set(norm_vector.keys()) | set(norm_stat.keys())
        
        # RRF融合
        rrf_scores = {}
        k = 60  # RRF参数
        
        for community_id in all_communities:
            rank_keyword = list(norm_keyword.keys()).index(community_id) + 1 if community_id in norm_keyword else float('inf')
            rank_vector = list(norm_vector.keys()).index(community_id) + 1 if community_id in norm_vector else float('inf')
            rank_stat = list(norm_stat.keys()).index(community_id) + 1 if community_id in norm_stat else float('inf')
            
            # RRF公式
            rrf = (1 / (k + rank_keyword)) + (1 / (k + rank_vector)) + (1 / (k + rank_stat))
            rrf_scores[community_id] = rrf
        
        # 排序并返回top_k
        sorted_communities = sorted(rrf_scores.items(), key=lambda x: x[1], reverse=True)
        
        return [cid for cid, score in sorted_communities[:top_k] if score >= min_relevance]
    
    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        """计算余弦相似度"""
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        if norm_a == 0 or norm_b == 0:
            return 0
        return np.dot(a, b) / (norm_a * norm_b)
```

### 4.2 动态查询扩展

LazyGraphRAG支持查询扩展，通过分析用户查询的语义来识别可能的扩展方向，提高社区选择的召回率。

```python
class QueryExpander:
    """查询扩展器"""
    
    def __init__(self, llm_client, global_index: GlobalIndex):
        self.llm = llm_client
        self.global_index = global_index
    
    def expand(self, query: str, expansion_terms: int = 5) -> List[str]:
        """
        扩展用户查询
        
        返回扩展后的查询列表（包括原查询）
        """
        # 1. 同义词扩展
        synonyms = self._expand_with_synonyms(query, expansion_terms)
        
        # 2. 下位词扩展（更具体）
        hyponyms = self._expand_with_hyponyms(query, expansion_terms)
        
        # 3. 相关概念扩展
        related = self._expand_with_related_concepts(query, expansion_terms)
        
        # 融合并去重
        expanded = [query]
        expanded.extend(synonyms[:2])
        expanded.extend(hyponyms[:1])
        expanded.extend(related[:2])
        
        return list(dict.fromkeys(expanded))[:10]  # 去重并限制数量
    
    def _expand_with_synonyms(self, query: str, limit: int) -> List[str]:
        """使用LLM生成同义词扩展"""
        prompt = f"""
给出以下查询的同义词或相近表达，每个查询最多{limit}个。

查询: {query}

同义词:
"""
        response = self.llm.generate(prompt)
        # 解析响应（简化实现）
        lines = [l.strip() for l in response.split('\n') if l.strip()]
        return lines[:limit]
    
    def _expand_with_hyponyms(self, query: str, limit: int) -> List[str]:
        """使用LLM生成下位词扩展"""
        prompt = f"""
给出以下查询的下位概念（更具体的子类别或实例），最多{limit}个。

查询: {query}

下位概念:
"""
        response = self.llm.generate(prompt)
        lines = [l.strip() for l in response.split('\n') if l.strip()]
        return lines[:limit]
    
    def _expand_with_related_concepts(self, query: str, limit: int) -> List[str]:
        """基于全局索引中的共现关系扩展"""
        query_terms = set(query.lower().split())
        
        # 统计与查询词共现的词
        cooccurrence = defaultdict(int)
        
        for doc_id, term_freq in self.global_index.doc_term_freq.items():
            doc_terms = set(term_freq.keys())
            if query_terms & doc_terms:  # 如果文档包含查询词
                for term in doc_terms - query_terms:
                    cooccurrence[term] += 1
        
        # 返回共现频率最高的词
        sorted_terms = sorted(cooccurrence.items(), key=lambda x: x[1], reverse=True)
        return [term for term, _ in sorted_terms[:limit]]
```

## 五、LazyGraphRAG查询流程

### 5.1 完整查询Pipeline

```python
class LazyGraphRAG:
    """LazyGraphRAG主类"""
    
    def __init__(self, global_index: GlobalIndex,
                 embedding_model, llm_client):
        self.global_index = global_index
        self.embedding_model = embedding_model
        self.llm = llm_client
        
        # 初始化组件
        self.metrics = CommunityMetrics(global_index)
        self.router = CommunityRouter(global_index, embedding_model, self.metrics)
        self.expander = QueryExpander(llm_client, global_index)
        self.local_index_cache = {}
    
    def query(self, user_query: str, max_communities: int = 5,
              max_context_tokens: int = 8000) -> str:
        """
        执行LazyGraphRAG查询
        
        流程：
        1. 查询扩展
        2. 社区路由
        3. 局部索引计算
        4. 上下文构建
        5. LLM生成
        """
        # 步骤1: 查询扩展
        expanded_queries = self.expander.expand(user_query)
        
        # 步骤2: 社区路由
        selected_communities = []
        for query in expanded_queries[:3]:  # 使用前3个扩展查询
            communities = self.router.route(query, top_k=max_communities)
            selected_communities.extend(communities)
        
        # 去重并限制
        selected_communities = list(dict.fromkeys(selected_communities))[:max_communities]
        
        # 步骤3: 收集相关文档
        relevant_docs = self._collect_relevant_documents(selected_communities, user_query)
        
        # 步骤4: 构建上下文
        context = self._build_context(relevant_docs, user_query, max_context_tokens)
        
        # 步骤5: LLM生成
        response = self._generate_response(user_query, context)
        
        return response
    
    def _collect_relevant_documents(self, communities: List[int],
                                    query: str) -> List[Dict]:
        """收集社区相关文档"""
        relevant_docs = []
        query_embedding = self.embedding_model.embed(query)
        
        for community_id in communities:
            # 获取社区文档
            community_docs = [
                doc_id for doc_id, cid 
                in self.global_index.doc_community_mapping.items()
                if cid == community_id
            ]
            
            for doc_id in community_docs:
                # 计算相关性分数
                doc_signature = self.global_index.doc_signatures.get(doc_id)
                if doc_signature is not None:
                    similarity = self._cosine_similarity(
                        query_embedding, 
                        doc_signature.astype(float)
                    )
                    
                    if similarity > 0.3:  # 阈值
                        relevant_docs.append({
                            'doc_id': doc_id,
                            'community_id': community_id,
                            'similarity': similarity
                        })
        
        # 按相似度排序
        relevant_docs.sort(key=lambda x: x['similarity'], reverse=True)
        return relevant_docs[:20]  # 限制数量
    
    def _build_context(self, relevant_docs: List[Dict], query: str,
                       max_tokens: int) -> str:
        """构建检索上下文"""
        context_parts = []
        current_tokens = 0
        
        for doc_info in relevant_docs:
            doc_id = doc_info['doc_id']
            
            # 获取文档内容（这里需要从原始数据源获取）
            # 简化：使用文档ID作为占位符
            doc_content = f"[Document: {doc_id}]"
            doc_tokens = len(doc_content.split()) * 1.3  # 估算
            
            if current_tokens + doc_tokens > max_tokens:
                break
            
            context_parts.append(doc_content)
            current_tokens += doc_tokens
        
        return "\n\n".join(context_parts)
    
    def _generate_response(self, query: str, context: str) -> str:
        """使用LLM生成回答"""
        prompt = f"""
## 用户查询
{query}

## 检索到的上下文
{context}

## 回答要求
1. 基于提供的上下文信息回答用户问题
2. 如果上下文包含相关信息，详细回答
3. 如果信息不足，说明限制
4. 保持回答的准确性和完整性

请回答：
"""
        return self.llm.generate(prompt)
    
    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        """计算余弦相似度"""
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        if norm_a == 0 or norm_b == 0:
            return 0
        return np.dot(a, b) / (norm_a * norm_b)
```

### 5.2 与标准GraphRAG的对比

| 特性 | 标准GraphRAG | LazyGraphRAG |
|------|------------|--------------|
| 索引成本 | O(N) - 高 | O(1) - 低 |
| 查询成本 | O(1) - 低 | O(K) - 中等 |
| LLM调用次数 | 大量 | 极少（仅查询时） |
| 索引更新 | 全量重建 | 增量更新 |
| 适用场景 | 高频查询 | 低频查询 |
| 推理能力 | 完整 | 取决于社区选择质量 |
| 内存占用 | 高 | 低 |

> [!note] 实践建议
> LazyGraphRAG的最佳实践是先使用LazyGraphRAG进行快速迭代和验证，当系统成熟且查询量增加后，再考虑升级到完整GraphRAG以获得更稳定的推理能力。

## 六、实战部署指南

### 6.1 环境配置

```bash
# 创建虚拟环境
python -m venv lazy_graphrag_env
source lazy_graphrag_env/bin/activate

# 安装依赖
pip install numpy networkx scikit-learn

# 如果使用OpenAI嵌入
pip install openai

# 如果使用本地模型
# pip install sentence-transformers
```

### 6.2 完整使用示例

```python
from lazy_graphrag import LazyGraphRAG, GlobalIndex, IncrementalIndexer
from embedding_model import OpenAIEmbedding  # 或本地模型

# 1. 初始化全局索引
global_index = GlobalIndex()

# 2. 创建增量索引器并添加文档
indexer = IncrementalIndexer(
    global_index=global_index,
    embedding_model=OpenAIEmbedding(model="text-embedding-3-small"),
    llm_client=None  # 添加文档不需要LLM
)

# 批量添加文档
documents = [
    {"id": "doc1", "content": "Transformer架构由Google在2017年提出..."},
    {"id": "doc2", "content": "BERT是一种双向Transformer编码器..."},
    # ... 更多文档
]

for doc in documents:
    indexer.add_document(doc["id"], doc["content"])

# 3. 初始化LazyGraphRAG
lazy_graphrag = LazyGraphRAG(
    global_index=global_index,
    embedding_model=OpenAIEmbedding(model="text-embedding-3-small"),
    llm_client=your_llm_client
)

# 4. 执行查询
result = lazy_graphrag.query("Transformer架构有哪些变体？")
print(result)
```

## 七、相关主题链接

- [[GraphRAG深度指南]] - 标准GraphRAG的完整实现
- [[知识图谱构建实战]] - 知识图谱的构建技术
- [[向量数据库对比]] - 底层存储选型
- [[Embedding模型选择]] - 文本向量化方案
- [[重排技术深度指南]] - 搜索结果优化
- [[评估体系]] - RAG系统评估方法

---

> [!note] 更新日志
> - 2026-04-18: 初始版本完成
> - 包含懒加载机制、指标计算、社区选择策略的详细解析
> - 提供完整的代码实现和部署指南
