---
title: RAG上下文优化指南
date: 2026-04-18
tags:
  - RAG
  - retrieval-augmented
  - context-optimization
  - reranking
  - compression
categories:
  - context-engineering
  - rag-optimization
---

> [!abstract] 摘要
> RAG（检索增强生成）是当前LLM应用的主流架构，而上下文优化是提升RAG效果的关键。本文系统讲解RAG上下文优化的核心策略：相关性排序、冗余消除、上下文压缩、窗口内重排，以及上下文长度规划，帮助构建高效、准确的RAG系统。

## 关键词速览

| 术语 | 英文 | 说明 |
|------|------|------|
| RAG | Retrieval-Augmented Generation | 检索增强生成 |
| 向量检索 | Vector Retrieval | 基于嵌入向量的检索 |
| 重排 | Reranking | 对检索结果重新排序 |
| 上下文压缩 | Context Compression | 压缩上下文长度 |
| 混合检索 | Hybrid Retrieval | 多种检索方式结合 |
| 查询扩展 | Query Expansion | 扩展查询语义 |
| 子查询 | Sub-query | 将查询分解为子问题 |
| HyDE | HyDE | 基于假设文档的检索 |
| 上下文窗口 | Context Window | 模型处理上限 |
| Token预算 | Token Budget | 可用的token数量 |

## 一、RAG系统概述

### 1.1 RAG工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        RAG系统架构                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  用户查询 ──► 查询处理 ──► 检索 ──► 重排 ──► 上下文构建 ──► LLM   │
│                              │      │              │             │
│                              │      │              │             │
│                         知识库索引   过滤/压缩      格式优化      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 RAG中的上下文挑战

RAG系统面临的核心上下文问题：

1. **检索质量**：检索到的文档可能与查询不相关
2. **信息冗余**：多个检索结果可能包含重复信息
3. **长度限制**：上下文窗口有限，需要取舍
4. **噪声干扰**：无关内容可能影响生成质量
5. **位置偏差**：中间位置信息容易被忽略

## 二、上下文相关性排序

### 2.1 相关性评估方法

```python
from dataclasses import dataclass
from typing import List, Optional, Dict
import numpy as np

@dataclass
class RetrievedChunk:
    """检索到的文档块"""
    chunk_id: str
    content: str
    score: float  # 检索阶段的相关性分数
    source: str
    metadata: Dict
    
    def __repr__(self):
        return f"Chunk(id={self.chunk_id}, score={self.score:.3f})"

class RelevanceScorer:
    """相关性评分器"""
    
    def __init__(self, embedding_model=None, reranker=None):
        self.embedding_model = embedding_model
        self.reranker = reranker
    
    def semantic_score(self, query: str, document: str) -> float:
        """语义相关性分数"""
        # 使用embedding模型计算余弦相似度
        query_emb = self.embedding_model.encode(query)
        doc_emb = self.embedding_model.encode(document)
        return self.cosine_similarity(query_emb, doc_emb)
    
    def keyword_score(self, query: str, document: str) -> float:
        """关键词匹配分数"""
        query_terms = set(query.lower().split())
        doc_terms = set(document.lower().split())
        
        if not query_terms:
            return 0.0
        
        # Jaccard相似度
        intersection = query_terms & doc_terms
        return len(intersection) / len(query_terms)
    
    def combined_score(
        self,
        query: str,
        document: str,
        weights: Dict[str, float] = None
    ) -> float:
        """综合相关性分数"""
        if weights is None:
            weights = {'semantic': 0.7, 'keyword': 0.3}
        
        semantic = self.semantic_score(query, document)
        keyword = self.keyword_score(query, document)
        
        return (
            weights.get('semantic', 0.5) * semantic +
            weights.get('keyword', 0.5) * keyword
        )
    
    @staticmethod
    def cosine_similarity(a: np.ndarray, b: np.ndarray) -> float:
        dot = np.dot(a, b)
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        return dot / (norm_a * norm_b + 1e-8)
```

### 2.2 重排策略

```python
class Reranker:
    """检索结果重排器"""
    
    def __init__(self, reranker_model=None):
        self.reranker = reranker_model
    
    def cross_encoder_rerank(
        self,
        query: str,
        documents: List[str],
        top_k: int = 10
    ) -> List[RetrievedChunk]:
        """使用Cross-Encoder重排"""
        # 加载reranker模型（如BAAI/bge-reranker）
        if self.reranker is None:
            # 简化实现：使用相关性评分
            scores = []
            for doc in documents:
                score = self._simple_score(query, doc)
                scores.append(score)
        else:
            pairs = [(query, doc) for doc in documents]
            scores = self.reranker.predict(pairs)
        
        # 按分数排序
        scored_docs = sorted(
            zip(scores, documents),
            key=lambda x: x[0],
            reverse=True
        )
        
        # 返回top-k
        results = []
        for score, doc in scored_docs[:top_k]:
            results.append(RetrievedChunk(
                chunk_id=doc.get('id', 'unknown'),
                content=doc['content'],
                score=float(score),
                source=doc.get('source', 'unknown'),
                metadata=doc.get('metadata', {})
            ))
        
        return results
    
    def diversity_rerank(
        self,
        query: str,
        documents: List[RetrievedChunk],
        diversity_weight: float = 0.3,
        relevance_weight: float = 0.7
    ) -> List[RetrievedChunk]:
        """
        多样性重排
        在保持相关性的同时增加结果多样性
        """
        if not documents:
            return []
        
        reranked = [documents[0]]  # 保留最高相关性
        remaining = documents[1:]
        
        while remaining:
            best_score = float('-inf')
            best_doc = None
            best_idx = None
            
            for i, doc in enumerate(remaining):
                # 相关性分数
                rel_score = doc.score
                
                # 多样性分数（与已选文档的最大相似度）
                max_similarity = 0
                for selected in reranked:
                    sim = self._content_similarity(doc.content, selected.content)
                    max_similarity = max(max_similarity, sim)
                
                div_score = 1 - max_similarity
                
                # 综合分数
                combined = (
                    relevance_weight * rel_score +
                    diversity_weight * div_score
                )
                
                if combined > best_score:
                    best_score = combined
                    best_doc = doc
                    best_idx = i
            
            reranked.append(best_doc)
            remaining.pop(best_idx)
        
        return reranked
    
    def _simple_score(self, query: str, doc: str) -> float:
        """简化相关性评分"""
        query_terms = set(query.lower().split())
        doc_terms = set(doc.lower().split())
        return len(query_terms & doc_terms) / max(len(query_terms), 1)
    
    def _content_similarity(self, content1: str, content2: str) -> float:
        """内容相似度"""
        # 简化实现：基于词重叠
        terms1 = set(content1.lower().split())
        terms2 = set(content2.lower().split())
        if not terms1 or not terms2:
            return 0
        return len(terms1 & terms2) / len(terms1 | terms2)
```

## 三、冗余消除

### 3.1 冗余检测方法

```python
class RedundancyEliminator:
    """冗余消除器"""
    
    def __init__(self, embedding_model=None):
        self.embedding_model = embedding_model
        self.similarity_threshold = 0.85  # 相似度阈值
    
    def detect_redundancy(
        self,
        chunks: List[RetrievedChunk]
    ) -> Dict[str, List[str]]:
        """
        检测冗余，返回冗余组
        返回格式：{representative_id: [redundant_ids]}
        """
        redundancy_groups = {}
        processed = set()
        
        for i, chunk in enumerate(chunks):
            if chunk.chunk_id in processed:
                continue
            
            group = [chunk.chunk_id]
            
            for j, other in enumerate(chunks[i+1:], i+1):
                if other.chunk_id in processed:
                    continue
                
                similarity = self._calculate_similarity(
                    chunk.content,
                    other.content
                )
                
                if similarity > self.similarity_threshold:
                    group.append(other.chunk_id)
                    processed.add(other.chunk_id)
            
            if len(group) > 1:
                redundancy_groups[chunk.chunk_id] = group[1:]
                processed.add(chunk.chunk_id)
        
        return redundancy_groups
    
    def eliminate_redundancy(
        self,
        chunks: List[RetrievedChunk],
        strategy: str = "keep_longest"
    ) -> List[RetrievedChunk]:
        """
        消除冗余
        
        strategy: 'keep_first', 'keep_longest', 'keep_most_relevant', 'merge'
        """
        redundancy_groups = self.detect_redundancy(chunks)
        
        # 收集需要删除的chunk id
        to_remove = set()
        for group in redundancy_groups.values():
            to_remove.update(group)
        
        if strategy == "keep_first":
            # 保留每组第一个
            pass  # to_remove已经包含了非首个
        
        elif strategy == "keep_longest":
            # 保留每组最长的
            chunk_map = {c.chunk_id: c for c in chunks}
            for rep_id, redundant_ids in redundancy_groups.items():
                candidates = [rep_id] + redundant_ids
                longest_id = max(
                    candidates,
                    key=lambda cid: len(chunk_map[cid].content)
                )
                to_remove.update(c for c in candidates if c != longest_id)
        
        elif strategy == "merge":
            # 合并冗余内容
            return self._merge_redundant_chunks(chunks, redundancy_groups)
        
        return [c for c in chunks if c.chunk_id not in to_remove]
    
    def _merge_redundant_chunks(
        self,
        chunks: List[RetrievedChunk],
        redundancy_groups: Dict
    ) -> List[RetrievedChunk]:
        """合并冗余块"""
        chunk_map = {c.chunk_id: c for c in chunks}
        to_remove = set()
        merged_chunks = []
        
        for rep_id, redundant_ids in redundancy_groups.items():
            # 收集所有相关块的内容
            all_contents = [chunk_map[rep_id].content]
            all_metadata = [chunk_map[rep_id].metadata]
            
            for rid in redundant_ids:
                all_contents.append(chunk_map[rid].content)
                all_metadata.append(chunk_map[rid].metadata)
                to_remove.add(rid)
            
            # 合并内容（去重）
            merged_content = self._deduplicate_content(all_contents)
            
            merged_chunks.append(RetrievedChunk(
                chunk_id=rep_id,
                content=merged_content,
                score=chunk_map[rep_id].score,
                source="merged",
                metadata={'merged_from': [rep_id] + redundant_ids}
            ))
        
        # 添加非冗余块
        result = [
            c for c in chunks
            if c.chunk_id not in to_remove and c.chunk_id not in redundancy_groups
        ]
        result.extend(merged_chunks)
        
        return result
    
    def _deduplicate_content(self, contents: List[str]) -> str:
        """内容去重合并"""
        lines = []
        seen = set()
        
        for content in contents:
            for line in content.split('\n'):
                line = line.strip()
                if line and line not in seen:
                    seen.add(line)
                    lines.append(line)
        
        return '\n'.join(lines)
    
    def _calculate_similarity(self, content1: str, content2: str) -> float:
        """计算内容相似度"""
        if self.embedding_model:
            emb1 = self.embedding_model.encode(content1)
            emb2 = self.embedding_model.encode(content2)
            return self.cosine_similarity(emb1, emb2)
        else:
            # 基于词重叠的简单相似度
            terms1 = set(content1.lower().split())
            terms2 = set(content2.lower().split())
            if not terms1 or not terms2:
                return 0
            return len(terms1 & terms2) / len(terms1 | terms2)
    
    @staticmethod
    def cosine_similarity(a, b):
        dot = np.dot(a, b)
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        return dot / (norm_a * norm_b + 1e-8)
```

## 四、上下文压缩

### 4.1 压缩策略概览

| 压缩策略 | 方法 | 优点 | 缺点 | 适用场景 |
|---------|------|------|------|---------|
| 规则压缩 | 删除停用词、压缩格式 | 快速、可控 | 可能丢失信息 | 格式化内容 |
| LLM压缩 | 使用LLM生成摘要 | 智能、保留语义 | 成本高、慢 | 语义丰富内容 |
| 选择性保留 | 只保留关键句子 | 精准 | 可能丢失上下文 | 长文档 |
| 实体保留 | 保留实体信息 | 保持事实准确性 | 可能丢失关系 | 知识密集型 |

### 4.2 规则压缩实现

```python
import re

class RuleBasedCompressor:
    """基于规则的上下文压缩器"""
    
    def __init__(self):
        self.stopwords = {
            '的', '了', '是', '在', '和', '与', '等', '以及', '这', '那',
            '一个', '我们', '你们', '他们', '可以', '能够', '需要', '应该'
        }
        self.template_phrases = [
            '请注意以下内容',
            '下面为大家介绍',
            '接下来我们来看',
            '综上所述',
            '总而言之'
        ]
    
    def compress(
        self,
        text: str,
        target_ratio: float = 0.7,
        preserve_structure: bool = True
    ) -> str:
        """
        压缩文本
        
        Args:
            text: 原始文本
            target_ratio: 目标压缩比 (0-1)
            preserve_structure: 是否保留结构
        """
        # 1. 移除HTML标签
        text = self._remove_html_tags(text)
        
        # 2. 规范化空白
        text = self._normalize_whitespace(text)
        
        # 3. 移除模板短语
        text = self._remove_template_phrases(text)
        
        # 4. 压缩句子
        if preserve_structure:
            text = self._compress_sentences(text, target_ratio)
        else:
            text = self._compress_aggressive(text, target_ratio)
        
        return text
    
    def _remove_html_tags(self, text: str) -> str:
        """移除HTML标签"""
        text = re.sub(r'<[^>]+>', '', text)
        text = re.sub(r'&[a-z]+;', '', text)
        return text
    
    def _normalize_whitespace(self, text: str) -> str:
        """规范化空白字符"""
        text = re.sub(r'\n{3,}', '\n\n', text)
        text = re.sub(r' {2,}', ' ', text)
        text = re.sub(r'\t', ' ', text)
        return text.strip()
    
    def _remove_template_phrases(self, text: str) -> str:
        """移除模板性短语"""
        for phrase in self.template_phrases:
            text = text.replace(phrase, '')
        return text
    
    def _compress_sentences(self, text: str, target_ratio: float) -> str:
        """保留性压缩句子"""
        sentences = re.split(r'[。！？\n]', text)
        sentences = [s.strip() for s in sentences if s.strip()]
        
        if not sentences:
            return text
        
        # 计算每个句子的重要性
        scored_sentences = []
        for sent in sentences:
            score = self._sentence_importance(sent)
            scored_sentences.append((score, sent))
        
        # 按重要性排序
        scored_sentences.sort(key=lambda x: x[0], reverse=True)
        
        # 选择保留的句子
        n_keep = int(len(sentences) * target_ratio)
        n_keep = max(n_keep, 1)  # 至少保留一个
        
        # 重新按原文顺序排列
        kept = set(s for s, _ in scored_sentences[:n_keep])
        result = [s for s in sentences if s in kept]
        
        return '。'.join(result) + '。' if result else text[:int(len(text) * target_ratio)]
    
    def _compress_aggressive(self, text: str, target_ratio: float) -> str:
        """激进压缩"""
        words = text.split()
        n_keep = int(len(words) * target_ratio)
        
        # 过滤停用词和短词
        filtered = [
            w for w in words
            if w not in self.stopwords and len(w) > 1
        ]
        
        # 如果过滤后太少，回退到保留关键词
        if len(filtered) < n_keep:
            keywords = [w for w in words if len(w) > 3]
            return ' '.join(keywords[:n_keep])
        
        return ' '.join(filtered[:n_keep])
    
    def _sentence_importance(self, sentence: str) -> float:
        """评估句子重要性"""
        score = 0.0
        
        # 长度分数
        if len(sentence) > 20:
            score += 0.3
        
        # 关键词分数
        important_keywords = ['关键', '重要', '核心', '主要', '必须', '建议']
        for kw in important_keywords:
            if kw in sentence:
                score += 0.2
        
        # 数字分数
        if re.search(r'\d+', sentence):
            score += 0.1
        
        # 位置分数（首尾句子略高）
        if sentence.startswith(('首先', '第一', '首先', '关键')):
            score += 0.2
        
        return score
```

### 4.3 LLM压缩实现

```python
class LLMCompressor:
    """基于LLM的智能压缩器"""
    
    def __init__(self, llm_client):
        self.llm = llm_client
    
    def compress_with_summary(
        self,
        text: str,
        max_length: int = 500,
        preserve_key_points: bool = True
    ) -> str:
        """使用LLM生成摘要压缩"""
        prompt = f"""请压缩以下文本，保留核心信息。

要求：
1. 保留所有关键事实和数据
2. 保留重要观点和结论
3. 删除重复和冗余表述
4. 目标长度：{max_length}字以内
5. 保持原文的语言风格

原文：
{text}

压缩后的版本："""
        
        return self.llm.generate(prompt).strip()
    
    def compress_selective(
        self,
        text: str,
        query: str,
        max_sentences: int = 5
    ) -> str:
        """
        选择性保留：只保留与查询最相关的句子
        
        解决Lost in Middle问题的策略之一
        """
        prompt = f"""从以下文本中，选择与用户查询最相关的句子。

用户查询：{query}

文本内容：
{text}

要求：
1. 选择最多{max_sentences}个最相关的句子
2. 保留原文中的所有关键信息
3. 按相关性从高到低排序
4. 如果原文顺序重要，保持原有顺序

选中的句子（直接输出，不要额外解释）："""
        
        return self.llm.generate(prompt).strip()
    
    def compress_entity_preserving(
        self,
        text: str,
        entity_types: List[str] = None
    ) -> str:
        """
        实体保留压缩：优先保留实体信息
        
        entity_types: ['person', 'organization', 'location', 'date', 'number']
        """
        if entity_types is None:
            entity_types = ['person', 'organization', 'date', 'number']
        
        prompt = f"""压缩以下文本，但必须保留所有实体信息。

必须保留的实体类型：{', '.join(entity_types)}

要求：
1. 保留所有提及的人名、组织名、地点、日期、数字
2. 删除冗余的描述和修饰
3. 保持事实的准确性
4. 压缩后的内容应该仍然通顺可读

原文：
{text}

压缩后的版本："""
        
        return self.llm.generate(prompt).strip()
```

## 五、上下文窗口内重排

### 5.1 重排策略

```python
class ContextReranker:
    """上下文窗口内重排器"""
    
    def __init__(self):
        self.head_size_ratio = 0.4  # 开头保留比例
        self.tail_size_ratio = 0.4  # 结尾保留比例
    
    def rerank_for_llm(
        self,
        chunks: List[RetrievedChunk],
        query: str,
        max_tokens: int = 30000
    ) -> str:
        """
        为LLM重排上下文内容
        
        策略：将最相关的内容放在开头和结尾
        """
        # 1. 按相关性排序
        scored_chunks = self._score_and_sort(chunks, query)
        
        # 2. 计算可用空间
        total_content = '\n\n'.join([c.content for c in scored_chunks])
        if len(total_content) <= max_tokens:
            return self._build_context(scored_chunks)
        
        # 3. 优先保留开头和结尾
        return self._prioritize_ends(scored_chunks, max_tokens)
    
    def _score_and_sort(
        self,
        chunks: List[RetrievedChunk],
        query: str
    ) -> List[RetrievedChunk]:
        """计算相关性分数并排序"""
        # 根据原始分数和查询相关性重新评分
        scored = []
        for chunk in chunks:
            # 简单实现：直接使用原始分数
            scored.append(chunk)
        
        return sorted(scored, key=lambda c: c.score, reverse=True)
    
    def _prioritize_ends(
        self,
        chunks: List[RetrievedChunk],
        max_tokens: int
    ) -> str:
        """
        优先保留开头和结尾的内容
        
        利用LLM对首尾位置注意力更强的特性
        """
        if not chunks:
            return ""
        
        result_parts = []
        current_tokens = 0
        
        # 1. 头部：按相关性顺序添加
        head_limit = int(max_tokens * self.head_size_ratio)
        for chunk in chunks:
            chunk_tokens = len(chunk.content) // 4  # 粗略估算
            if current_tokens + chunk_tokens > head_limit:
                break
            result_parts.append(chunk.content)
            current_tokens += chunk_tokens
        
        # 2. 中间：选择性添加最重要的
        middle_chunks = chunks[len(result_parts):-len(result_parts)] if len(result_parts) < len(chunks) else []
        middle_limit = int(max_tokens * 0.2)
        for chunk in middle_chunks:
            chunk_tokens = len(chunk.content) // 4
            if current_tokens + chunk_tokens > max_tokens - head_limit:
                break
            result_parts.append(f"[相关内容]\n{chunk.content}")
            current_tokens += chunk_tokens
        
        # 3. 尾部：添加高相关性的
        tail_limit = int(max_tokens * self.tail_size_ratio)
        remaining = max_tokens - current_tokens - tail_limit
        
        # 从后往前添加
        for chunk in reversed(chunks):
            if chunk in result_parts:
                continue
            chunk_tokens = len(chunk.content) // 4
            if chunk_tokens > remaining:
                continue
            result_parts.insert(0, chunk.content)
            remaining -= chunk_tokens
            current_tokens += chunk_tokens
            if remaining <= 0:
                break
        
        return '\n\n---\n\n'.join(result_parts)
    
    def _build_context(self, chunks: List[RetrievedChunk]) -> str:
        """构建上下文字符串"""
        parts = []
        for chunk in chunks:
            parts.append(chunk.content)
        return '\n\n---\n\n'.join(parts)
```

## 六、上下文长度规划

### 6.1 长度规划策略

```python
class ContextLengthPlanner:
    """上下文长度规划器"""
    
    def __init__(
        self,
        model_context_limit: int = 200000,
        reserve_for_output: int = 4000,
        reserve_for_instructions: int = 1000
    ):
        self.model_context_limit = model_context_limit
        self.reserve_for_output = reserve_for_output
        self.reserve_for_instructions = reserve_for_instructions
    
    def calculate_available_context(
        self,
        system_prompt: str = "",
        few_shot_examples: List[str] = None,
        user_query: str = ""
    ) -> int:
        """计算可用于检索上下文的token数"""
        # 系统提示token
        system_tokens = self._estimate_tokens(system_prompt)
        
        # Few-shot示例token
        few_shot_tokens = 0
        if few_shot_examples:
            for example in few_shot_examples:
                few_shot_tokens += self._estimate_tokens(example)
        
        # 用户查询token
        query_tokens = self._estimate_tokens(user_query)
        
        # 可用空间
        available = (
            self.model_context_limit
            - system_tokens
            - few_shot_tokens
            - query_tokens
            - self.reserve_for_output
            - self.reserve_for_instructions
        )
        
        return max(available, 0)
    
    def plan_retrieval(
        self,
        available_tokens: int,
        avg_chunk_size: int = 500,
        redundancy_buffer: float = 1.2
    ) -> Dict:
        """
        规划检索策略
        
        Returns:
            chunk_count: 需要检索的块数
            top_k: 返回的块数
            compression_ratio: 压缩比
        """
        # 考虑冗余缓冲
        effective_tokens = int(available_tokens / redundancy_buffer)
        
        # 计算可以容纳的块数
        chunk_count = effective_tokens // avg_chunk_size
        
        # 返回更多块用于后续过滤
        top_k = min(int(chunk_count * 1.5), 50)
        
        # 压缩比
        compression_ratio = available_tokens / (chunk_count * avg_chunk_size)
        
        return {
            'chunk_count': chunk_count,
            'top_k': top_k,
            'compression_ratio': compression_ratio,
            'effective_tokens': effective_tokens
        }
    
    def estimate_compression_needed(
        self,
        retrieved_content_tokens: int,
        available_tokens: int
    ) -> float:
        """估算需要的压缩比"""
        if retrieved_content_tokens <= available_tokens:
            return 1.0  # 不需要压缩
        
        return available_tokens / retrieved_content_tokens
    
    @staticmethod
    def _estimate_tokens(text: str) -> int:
        """粗略估算token数"""
        if not text:
            return 0
        # 中文约0.5token/字，英文约0.25token/词
        chinese = sum(1 for c in text if '\u4e00' <= c <= '\u9fff')
        english = len(text.split()) - chinese
        return int(chinese * 0.5 + english * 0.25 + len(text) * 0.1)
```

### 6.2 完整RAG上下文优化流程

```python
class RAGContextOptimizer:
    """RAG上下文优化器 - 综合所有策略"""
    
    def __init__(
        self,
        embedding_model=None,
        reranker=None,
        llm_client=None
    ):
        self.relevance_scorer = RelevanceScorer(embedding_model)
        self.reranker = Reranker(reranker)
        self.redundancy_eliminator = RedundancyEliminator(embedding_model)
        self.rule_compressor = RuleBasedCompressor()
        self.llm_compressor = LLMCompressor(llm_client) if llm_client else None
        self.context_reranker = ContextReranker()
        self.length_planner = ContextLengthPlanner()
    
    def optimize(
        self,
        query: str,
        retrieved_chunks: List[RetrievedChunk],
        available_tokens: int = None,
        strategy: str = "auto"
    ) -> str:
        """
        完整的RAG上下文优化流程
        
        strategy: 'auto', 'high_quality', 'high_recall', 'balanced'
        """
        if not retrieved_chunks:
            return ""
        
        # 1. 重排（如果可用）
        if strategy in ['auto', 'high_quality']:
            reranked = self.reranker.cross_encoder_rerank(
                query,
                [{'id': c.chunk_id, 'content': c.content, 'source': c.source}
                 for c in retrieved_chunks]
            )
            retrieved_chunks = reranked
        
        # 2. 消除冗余
        if strategy in ['auto', 'balanced']:
            deduplicated = self.redundancy_eliminator.eliminate_redundancy(
                retrieved_chunks,
                strategy='merge'
            )
            retrieved_chunks = deduplicated
        
        # 3. 长度检查和压缩
        total_tokens = sum(self.length_planner._estimate_tokens(c.content) 
                          for c in retrieved_chunks)
        
        if available_tokens and total_tokens > available_tokens:
            compression_ratio = available_tokens / total_tokens
            
            if compression_ratio < 0.5:
                # 需要较大压缩，使用LLM
                if self.llm_compressor:
                    compressed = self.llm_compressor.compress_selective(
                        '\n\n'.join([c.content for c in retrieved_chunks]),
                        query,
                        max_sentences=int(len(retrieved_chunks) * compression_ratio)
                    )
                    return compressed
            
            # 轻度压缩，使用规则
            compressed_chunks = []
            for chunk in retrieved_chunks:
                compressed = self.rule_compressor.compress(
                    chunk.content,
                    target_ratio=compression_ratio
                )
                compressed_chunks.append(RetrievedChunk(
                    chunk_id=chunk.chunk_id,
                    content=compressed,
                    score=chunk.score,
                    source=chunk.source,
                    metadata=chunk.metadata
                ))
            retrieved_chunks = compressed_chunks
        
        # 4. 最终重排（首尾优先）
        final_context = self.context_reranker.rerank_for_llm(
            retrieved_chunks,
            query,
            max_tokens=available_tokens or 30000
        )
        
        return final_context
```

## 七、相关主题

- [[上下文窗口深度解析]]
- [[滑动窗口技术]]
- [[上下文压缩技术]]
- [[对话历史管理]]
- [[Context Caching详解]]

## 八、参考文献

1. Gao, Y., et al. (2023). Retrieval-Augmented Generation for Large Language Models: A Survey.
2. Lewis, P., et al. (2020). Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks. *NeurIPS*.
3. Ram, O., et al. (2023). In-Context Retrieval-Augmented Language Models. *TACL*.
