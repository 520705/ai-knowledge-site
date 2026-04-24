---
title: RAG上下文优化指南
date: 2026-04-24
tags:
  - rag
  - retrieval
  - context-optimization
  - vector-search
  - hybrid-retrieval
categories:
  - context-engineering
  - rag-optimization
---

> [!abstract] 摘要
> RAG（检索增强生成）是目前最火热的LLM应用范式，但做好RAG并不简单。这篇为零基础读者讲解：RAG是什么、RAG的核心组件、如何优化检索质量、如何优化上下文组织、以及如何构建生产级RAG系统。看完全篇，你就能从零搭建一个效果不错的RAG系统了。

## 先理解：什么是RAG？

### RAG的名字拆解

```
RAG = Retrieval（检索） + Augmented（增强） + Generation（生成）

 Retrieval（检索）：
    在大量文档中找到相关的片段

 Augmented（增强）：
    把检索结果当作"知识"给LLM

 Generation（生成）：
    LLM基于这些知识生成回答
```

### 为什么要用RAG？

```
没有RAG时：
用户：2024年Q3财报说了什么？
AI：我不知道，我是2023年训练的...
     （知识过时，或者根本没有这个知识）

有RAG时：
用户：2024年Q3财报说了什么？
  ↓
检索：找到2024年Q3财报的PDF
  ↓
上下文：[PDF中的相关内容片段]
  ↓
AI：财报显示Q3营收同比增长30%，主要原因是...
     （基于最新文档回答！）
```

### RAG的工作流程

```
┌─────────────────────────────────────────────────────────────────┐
│                        RAG完整流程                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1️⃣ 文档处理阶段                                                │
│  文档 → 分割 → 向量化 → 存入向量数据库                              │
│                                                                  │
│  2️⃣ 查询阶段                                                     │
│  用户问题 → 向量化 → 相似度搜索 → 召回相关文档                       │
│                                                                  │
│  3️⃣ 生成阶段                                                     │
│  用户问题 + 召回文档 → LLM → 生成回答                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 一、RAG的核心组件

### 组件一览

| 组件 | 功能 | 常见技术 |
|------|------|---------|
| 文档加载器 | 读取各种格式的文档 | PDF解析、网页爬虫、CSV解析 |
| 文本分割器 | 把大文档切成小片段 | 固定长度、语义分割、递归字符分割 |
| 向量化模型 | 把文本变成向量 | OpenAI Embedding、BGE、M3E |
| 向量数据库 | 存储和搜索向量 | Milvus、Pinecone、Chroma |
| 重排序模型 | 对检索结果排序 | BGE-Reranker、Cohere |
| LLM | 生成最终回答 | GPT-4、Claude、通义千问 |

### 最小RAG实现

```python
from typing import List, Dict, Optional
import numpy as np

class SimpleRAG:
    """
    最简单的RAG实现
    包含完整的检索和生成流程
    """

    def __init__(
        self,
        embedding_model,
        vector_store,
        llm_client
    ):
        self.embedding = embedding_model
        self.vector_db = vector_store
        self.llm = llm_client

    def index_documents(self, documents: List[Dict]):
        """
        索引文档

        Args:
            documents: [{'text': str, 'metadata': dict}]
        """
        for doc in documents:
            # 1. 向量化
            vector = self.embedding.embed(doc['text'])

            # 2. 存入向量数据库
            self.vector_db.add(
                id=doc.get('id', str(hash(doc['text']))),
                vector=vector,
                text=doc['text'],
                metadata=doc.get('metadata', {})
            )

    def retrieve(self, query: str, top_k: int = 5) -> List[Dict]:
        """
        检索相关文档

        Args:
            query: 用户问题
            top_k: 返回数量
        """
        # 1. 把问题向量化
        query_vector = self.embedding.embed(query)

        # 2. 搜索相似文档
        results = self.vector_db.search(
            vector=query_vector,
            top_k=top_k
        )

        return results

    def generate(self, query: str, context_docs: List[Dict]) -> str:
        """
        生成回答

        Args:
            query: 用户问题
            context_docs: 检索到的文档
        """
        # 构建上下文
        context = '\n\n'.join([doc['text'] for doc in context_docs])

        # 构建提示词
        prompt = f"""基于以下文档内容回答问题。如果文档中没有答案，说明不知道。

文档内容：
{context}

问题：{query}

回答要求：
1. 基于文档内容回答
2. 如果有多个文档，综合它们的答案
3. 引用相关原文

回答："""

        # 生成回答
        return self.llm.generate(prompt)

    def query(self, question: str, top_k: int = 5) -> str:
        """
        完整的RAG查询
        """
        # 1. 检索
        docs = self.retrieve(question, top_k)

        if not docs:
            return "抱歉，没有找到相关的文档。"

        # 2. 生成
        answer = self.generate(question, docs)

        return answer


# 使用示例
def demo_simple_rag():
    """演示简单RAG"""
    rag = SimpleRAG(
        embedding_model=embedding_model,
        vector_store=vector_store,
        llm_client=llm
    )

    # 索引文档
    documents = [
        {
            'text': 'Python是一种高级编程语言，由Guido van Rossum于1991年创建。'
                    '它以简洁易读的语法著称，广泛应用于Web开发、数据科学、AI等领域。',
            'metadata': {'source': 'python_intro.txt'}
        },
        {
            'text': '机器学习是AI的一个分支，通过算法让计算机从数据中学习。'
                    '主要类型包括：监督学习、无监督学习、强化学习。',
            'metadata': {'source': 'ml_intro.txt'}
        },
        {
            'text': '深度学习是机器学习的子集，使用神经网络处理复杂问题。'
                    '常见的网络结构包括：CNN、RNN、Transformer等。',
            'metadata': {'source': 'dl_intro.txt'}
        }
    ]

    rag.index_documents(documents)

    # 查询
    answer = rag.query("Python是什么？")
    print(f"回答：{answer}")
```

---

## 二、文档处理与分割优化

### 为什么分割很重要？

```
分割太大：
10000字的文章 → 切成3块
用户问："第三章讲了什么？"
检索结果可能包含第一、二章，第三章可能被稀释 ❌

分割太小：
10000字的文章 → 切成100块
用户问综合问题
可能召回10块，每块都只有一小段，缺少上下文 ❌
```

### 分割策略

```python
class DocumentChunker:
    """
    文档分割器
    支持多种分割策略
    """

    def __init__(
        self,
        chunk_size: int = 500,
        overlap: int = 50,
        split_by: str = "character"  # character | sentence | paragraph
    ):
        self.chunk_size = chunk_size
        self.overlap = overlap
        self.split_by = split_by

    def chunk_text(self, text: str) -> List[Dict]:
        """分割文本"""
        if self.split_by == "character":
            return self._chunk_by_character(text)
        elif self.split_by == "sentence":
            return self._chunk_by_sentence(text)
        elif self.split_by == "paragraph":
            return self._chunk_by_paragraph(text)
        else:
            raise ValueError(f"Unknown split method: {self.split_by}")

    def _chunk_by_character(self, text: str) -> List[Dict]:
        """按字符数分割（简单粗暴）"""
        chunks = []
        start = 0

        while start < len(text):
            end = start + self.chunk_size
            chunk_text = text[start:end]

            chunks.append({
                'text': chunk_text,
                'start': start,
                'end': end
            })

            # 滑动窗口：下一个块从当前位置往前一点开始
            start = end - self.overlap

        return chunks

    def _chunk_by_sentence(self, text: str) -> List[Dict]:
        """按句子分割（更智能）"""
        import re

        # 按句子分割（简单实现）
        sentences = re.split(r'[。！？.!?]+', text)
        sentences = [s.strip() for s in sentences if s.strip()]

        chunks = []
        current_chunk = []
        current_size = 0

        for sent in sentences:
            sent_size = len(sent)

            if current_size + sent_size > self.chunk_size:
                # 保存当前chunk
                if current_chunk:
                    chunks.append({
                        'text': '。'.join(current_chunk) + '。',
                        'sentences': len(current_chunk)
                    })

                # 开始新chunk（保留重叠）
                if self.overlap > 0 and current_chunk:
                    overlap_sents = []
                    overlap_size = 0
                    for s in reversed(current_chunk):
                        if overlap_size + len(s) <= self.overlap:
                            overlap_sents.insert(0, s)
                            overlap_size += len(s)
                        else:
                            break
                    current_chunk = overlap_sents
                    current_size = overlap_size
                else:
                    current_chunk = []
                    current_size = 0

            current_chunk.append(sent)
            current_size += sent_size

        # 处理最后一个chunk
        if current_chunk:
            chunks.append({
                'text': '。'.join(current_chunk) + '。',
                'sentences': len(current_chunk)
            })

        return chunks

    def _chunk_by_paragraph(self, text: str) -> List[Dict]:
        """按段落分割（保留结构）"""
        paragraphs = [p.strip() for p in text.split('\n\n') if p.strip()]

        chunks = []
        current_chunk = []
        current_size = 0

        for para in paragraphs:
            para_size = len(para)

            if current_size + para_size > self.chunk_size:
                if current_chunk:
                    chunks.append({
                        'text': '\n\n'.join(current_chunk),
                        'paragraphs': len(current_chunk)
                    })

                current_chunk = []
                current_size = 0

            current_chunk.append(para)
            current_size += para_size

        if current_chunk:
            chunks.append({
                'text': '\n\n'.join(current_chunk),
                'paragraphs': len(current_chunk)
            })

        return chunks


class SemanticChunker:
    """
    语义分割器
    按语义边界分割，保证每个块的主题一致性
    """

    def __init__(self, llm_client):
        self.llm = llm_client

    def chunk_document(self, text: str) -> List[Dict]:
        """语义分割文档"""
        # 1. 分析文档结构
        structure = self._analyze_structure(text)

        # 2. 按语义块分割
        chunks = self._semantic_split(text, structure)

        return chunks

    def _analyze_structure(self, text: str) -> dict:
        """分析文档结构"""
        prompt = f"""分析以下文档的结构，识别主要部分和主题：

{text[:5000]}

请用JSON格式输出：
{{
  "sections": [
    {{"title": "章节标题", "theme": "主题描述", "start": 0, "end": 1000}},
    ...
  ],
  "main_topics": ["主题1", "主题2", ...]
}}"""

        import json
        try:
            result = self.llm.generate(prompt)
            return json.loads(result)
        except:
            return {'sections': [], 'main_topics': []}

    def _semantic_split(self, text: str, structure: dict) -> List[Dict]:
        """按语义分割"""
        sections = structure.get('sections', [])

        if not sections:
            # 如果没有识别出结构，使用简单分割
            return [{'text': text, 'section': 'main'}]

        chunks = []
        for i, section in enumerate(sections):
            chunk_text = text[section['start']:section['end']]
            chunks.append({
                'text': chunk_text,
                'section_title': section.get('title', f'Section {i+1}'),
                'theme': section.get('theme', '')
            })

        return chunks
```

### 分割参数选择

```
┌─────────────────────────────────────────────────────────────┐
│                     分割参数选择指南                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📏 chunk_size（块大小）                                     │
│  ├─ 200-300 tokens：适合简单问答                            │
│  ├─ 500-800 tokens：适合一般文档                            │
│  └─ 1000+ tokens：适合长文档，需要更多上下文                 │
│                                                              │
│  🔄 overlap（重叠）                                          │
│  ├─ 0：不重叠，简单但可能丢失边界信息                        │
│  ├─ 10-20%：常用选择                                        │
│  └─ 50%：重叠多，边界信息保留好，但索引大                    │
│                                                              │
│  🎯 split_by（分割方式）                                    │
│  ├─ character：最快，但可能截断句子                          │
│  ├─ sentence：保留句子完整性                                │
│  └─ paragraph：保留段落结构                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 三、检索优化

### 基础检索：向量相似度

```python
class VectorRetriever:
    """
    向量检索器
    """

    def __init__(self, vector_store, embedding_model):
        self.vector_db = vector_store
        self.embedding = embedding_model

    def retrieve(
        self,
        query: str,
        top_k: int = 5,
        filters: dict = None
    ) -> List[Dict]:
        """
        向量检索

        Args:
            query: 查询文本
            top_k: 返回数量
            filters: 元数据过滤条件
        """
        # 1. 向量化查询
        query_vector = self.embedding.embed(query)

        # 2. 搜索
        results = self.vector_db.search(
            vector=query_vector,
            top_k=top_k * 2,  # 多召回一些，后面会过滤
            filter=filters
        )

        # 3. 过滤和格式化
        filtered = []
        for r in results:
            if r['score'] > 0.7:  # 相似度阈值
                filtered.append(r)

        return filtered[:top_k]


class KeywordRetriever:
    """
    关键词检索器
    基于BM25的稀疏检索
    """

    def __init__(self, documents: List[Dict]):
        self.documents = documents
        self.index = self._build_bm25_index()

    def _build_bm25_index(self) -> dict:
        """构建BM25索引"""
        # 简化实现：统计词频
        index = {}
        for doc in self.documents:
            words = self._tokenize(doc['text'])
            for word in words:
                if word not in index:
                    index[word] = []
                index[word].append({
                    'doc_id': doc['id'],
                    'tf': words.count(word)
                })
        return index

    def _tokenize(self, text: str) -> List[str]:
        """简单分词"""
        import re
        words = re.findall(r'[\w]+', text.lower())
        return words

    def retrieve(self, query: str, top_k: int = 5) -> List[Dict]:
        """关键词检索"""
        query_words = self._tokenize(query)
        scores = {}

        for word in query_words:
            if word in self.index:
                for entry in self.index[word]:
                    doc_id = entry['doc_id']
                    scores[doc_id] = scores.get(doc_id, 0) + entry['tf']

        # 排序
        sorted_docs = sorted(scores.items(), key=lambda x: x[1], reverse=True)

        # 返回结果
        results = []
        for doc_id, score in sorted_docs[:top_k]:
            doc = next((d for d in self.documents if d['id'] == doc_id), None)
            if doc:
                results.append({
                    'doc': doc,
                    'score': score,
                    'match_words': [w for w in query_words if w in doc['text'].lower()]
                })

        return results
```

### 混合检索

```python
class HybridRetriever:
    """
    混合检索器
    结合向量检索和关键词检索
    """

    def __init__(
        self,
        vector_store,
        embedding_model,
        documents: List[Dict]
    ):
        self.vector_retriever = VectorRetriever(vector_store, embedding_model)
        self.keyword_retriever = KeywordRetriever(documents)

        # 混合权重
        self.vector_weight = 0.7
        self.keyword_weight = 0.3

    def retrieve(
        self,
        query: str,
        top_k: int = 5,
        filters: dict = None
    ) -> List[Dict]:
        """
        混合检索
        """
        # 1. 向量检索
        vector_results = self.vector_retriever.retrieve(
            query, top_k * 2, filters
        )
        vector_scores = {
            r['id']: r['score']
            for r in vector_results
        }

        # 2. 关键词检索
        keyword_results = self.keyword_retriever.retrieve(query, top_k * 2)
        keyword_scores = {
            r['doc']['id']: r['score']
            for r in keyword_results
        }

        # 3. 合并分数
        all_ids = set(vector_scores.keys()) | set(keyword_scores.keys())

        combined_scores = {}
        for doc_id in all_ids:
            vec_s = vector_scores.get(doc_id, 0)
            key_s = keyword_scores.get(doc_id, 0)

            # 归一化
            max_vec = max(vector_scores.values()) if vector_scores else 1
            max_key = max(keyword_scores.values()) if keyword_scores else 1

            combined = (
                self.vector_weight * (vec_s / max_vec) +
                self.keyword_weight * (key_s / max_key)
            )

            combined_scores[doc_id] = combined

        # 4. 排序
        sorted_ids = sorted(
            combined_scores.items(),
            key=lambda x: x[1],
            reverse=True
        )

        # 5. 获取完整文档
        results = []
        for doc_id, score in sorted_ids[:top_k]:
            doc = next(
                (r for r in vector_results if r['id'] == doc_id),
                None
            )
            if not doc:
                doc = next(
                    (r['doc'] for r in keyword_results if r['doc']['id'] == doc_id),
                    None
                )
            if doc:
                doc['hybrid_score'] = score
                results.append(doc)

        return results
```

### 重排序优化

```python
class Reranker:
    """
    重排序器
    对检索结果进行精细排序
    """

    def __init__(self, reranker_model):
        self.model = reranker_model

    def rerank(
        self,
        query: str,
        documents: List[Dict],
        top_k: int = 5
    ) -> List[Dict]:
        """
        重排序

        Args:
            query: 查询
            documents: 检索到的文档
            top_k: 最终返回数量
        """
        if not documents:
            return []

        # 调用重排序模型
        doc_texts = [doc['text'] for doc in documents]

        scores = self.model.score(query, doc_texts)

        # 按分数排序
        scored_docs = [
            (doc, score)
            for doc, score in zip(documents, scores)
        ]
        scored_docs.sort(key=lambda x: x[1], reverse=True)

        # 返回top_k
        results = []
        for doc, score in scored_docs[:top_k]:
            doc['rerank_score'] = score
            results.append(doc)

        return results


class LLMReranker:
    """
    基于LLM的重排序
    使用LLM判断相关性
    """

    def __init__(self, llm_client):
        self.llm = llm_client

    def rerank(
        self,
        query: str,
        documents: List[Dict],
        top_k: int = 5
    ) -> List[Dict]:
        """LLM重排序"""
        if not documents:
            return []

        # 构建评分提示
        doc_summaries = []
        for i, doc in enumerate(documents):
            doc_summaries.append(f"[文档{i+1}]\n{doc['text'][:500]}...")

        prompt = f"""评估以下文档与查询的相关性，输出JSON格式的评分：

查询：{query}

文档：
{chr(10).join(doc_summaries)}

评分要求：
- 输出JSON数组，每个元素为{{"doc_id": 序号, "score": 0-10, "reason": "理由"}}
- 分数越高表示越相关

JSON输出："""

        import json
        try:
            result = self.llm.generate(prompt)
            ratings = json.loads(result)

            # 应用评分
            for rating in ratings:
                doc_id = rating['doc_id'] - 1
                if 0 <= doc_id < len(documents):
                    documents[doc_id]['llm_score'] = rating['score'] / 10.0
                    documents[doc_id]['llm_reason'] = rating['reason']

            # 按LLM评分排序
            documents.sort(key=lambda x: x.get('llm_score', 0), reverse=True)

        except Exception as e:
            print(f"LLM rerank failed: {e}")

        return documents[:top_k]
```

---

## 四、上下文优化

### 上下文构建策略

```python
class ContextBuilder:
    """
    上下文构建器
    将检索结果组织成适合LLM的格式
    """

    def __init__(self, llm_client):
        self.llm = llm_client

    def build_context(
        self,
        query: str,
        documents: List[Dict],
        strategy: str = "concatenate"
    ) -> str:
        """
        构建上下文

        Args:
            query: 用户问题
            documents: 检索到的文档
            strategy: 构建策略
        """
        if strategy == "concatenate":
            return self._concatenate_context(documents)
        elif strategy == "summarize":
            return self._summarize_context(query, documents)
        elif strategy == "tree":
            return self._tree_context(query, documents)
        elif strategy == "window":
            return self._window_context(documents)
        else:
            raise ValueError(f"Unknown strategy: {strategy}")

    def _concatenate_context(self, documents: List[Dict]) -> str:
        """简单拼接"""
        parts = []
        for i, doc in enumerate(documents, 1):
            parts.append(f"【文档{i}】\n{doc['text']}\n")
        return '\n'.join(parts)

    def _summarize_context(self, query: str, documents: List[Dict]) -> str:
        """摘要式上下文"""
        # 先让LLM总结每个文档的相关部分
        summaries = []

        for doc in documents:
            prompt = f"""基于以下文档，提取与问题相关的内容：

问题：{query}

文档：
{doc['text']}

如果文档与问题相关，提取相关部分并简要总结。
如果不相关，输出"不相关"。"""

            summary = self.llm.generate(prompt)
            if "不相关" not in summary:
                summaries.append(summary)

        if not summaries:
            return "没有找到相关文档。"

        return '\n\n'.join(summaries)

    def _tree_context(self, query: str, documents: List[Dict]) -> str:
        """树状上下文"""
        # 按主题分组
        prompt = f"""将以下文档按主题分组，输出JSON格式：

文档：
{chr(10).join([f"[文档{i+1}] {d['text'][:300]}..." for i, d in enumerate(documents)])}

输出格式：
{{
  "groups": [
    {{"topic": "主题名", "doc_ids": [1, 3], "summary": "该主题的总结"}},
    ...
  ]
}}"""

        import json
        try:
            result = self.llm.generate(prompt)
            groups = json.loads(result)['groups']

            # 构建树状上下文
            output = ["# 相关文档\n"]

            for group in groups:
                output.append(f"## {group['topic']}")
                output.append(f"总结：{group['summary']}")
                output.append("\n相关文档：")
                for doc_id in group['doc_ids']:
                    doc = documents[doc_id - 1]
                    output.append(f"\n{doc['text']}")
                output.append("")

            return '\n'.join(output)

        except:
            return self._concatenate_context(documents)

    def _window_context(self, documents: List[Dict]) -> str:
        """窗口式上下文"""
        # 保留文档的局部上下文
        output = []

        for i, doc in enumerate(documents, 1):
            # 添加前后文
            context = doc.get('context', {})
            before = context.get('before', '')
            after = context.get('after', '')

            output.append(f"【文档{i}】")
            if before:
                output.append(f"前文：{before}")
            output.append(f"正文：{doc['text']}")
            if after:
                output.append(f"后文：{after}")
            output.append("")

        return '\n'.join(output)
```

### 上下文压缩与扩展

```python
class ContextOptimizer:
    """
    上下文优化器
    """

    def __init__(self, llm_client, max_tokens: int = 8000):
        self.llm = llm_client
        self.max_tokens = max_tokens

    def optimize_context(
        self,
        query: str,
        documents: List[Dict]
    ) -> str:
        """
        优化上下文

        策略：
        1. 如果总长度合适，直接返回
        2. 如果太长，压缩
        3. 如果太短，扩展
        """
        # 估算总长度
        total_tokens = sum(self._estimate_tokens(d['text']) for d in documents)

        if total_tokens <= self.max_tokens:
            # 长度合适
            return self._build_context(documents)

        elif total_tokens <= self.max_tokens * 2:
            # 轻度压缩
            return self._compress_context(query, documents)

        else:
            # 重度压缩
            return self._heavy_compress(query, documents)

    def _build_context(self, documents: List[Dict]) -> str:
        """构建普通上下文"""
        output = []
        for i, doc in enumerate(documents, 1):
            output.append(f"[文档{i}]\n{doc['text']}")
        return '\n\n'.join(output)

    def _compress_context(
        self,
        query: str,
        documents: List[Dict]
    ) -> str:
        """轻度压缩"""
        prompt = f"""压缩以下文档内容，保留与问题相关的部分：

问题：{query}

文档：
{self._build_context(documents)}

压缩要求：
1. 删除与问题无关的内容
2. 保留关键信息和数据
3. 保留引用来源
4. 压缩后总长度约为原来的一半

压缩后："""

        return self.llm.generate(prompt)

    def _heavy_compress(
        self,
        query: str,
        documents: List[Dict]
    ) -> str:
        """重度压缩"""
        prompt = f"""从以下文档中提取回答问题所需的关键信息：

问题：{query}

文档：
{self._build_context(documents)}

提取要求：
1. 提取直接回答问题的内容
2. 保留原文的引用
3. 如果某个文档完全不相关，忽略它
4. 总长度控制在{self.max_tokens} tokens以内

提取结果："""

        return self.llm.generate(prompt)

    @staticmethod
    def _estimate_tokens(text: str) -> int:
        chinese = sum(1 for c in text if '\u4e00' <= c <= '\u9fff')
        english = len(text.split()) - chinese
        return int(chinese * 0.5 + english * 0.25)
```

---

## 五、生产级RAG系统

### 完整实现

```python
class ProductionRAG:
    """
    生产级RAG系统
    """

    def __init__(
        self,
        embedding_model,
        vector_store,
        llm_client,
        reranker_model=None
    ):
        self.embedding = embedding_model
        self.vector_db = vector_store
        self.llm = llm_client
        self.reranker = reranker_model

        # 组件
        self.chunker = DocumentChunker(chunk_size=500, overlap=50)
        self.hybrid_retriever = None
        self.context_builder = ContextBuilder(llm_client)
        self.context_optimizer = ContextOptimizer(llm_client)

        # 文档存储
        self.documents = {}

    def add_documents(
        self,
        documents: List[Dict],
        collection_name: str = "default"
    ):
        """
        添加文档

        Args:
            documents: [{'text': str, 'metadata': dict}]
            collection_name: 集合名称
        """
        # 分割文档
        chunks = []
        for doc in documents:
            doc_chunks = self.chunker.chunk_text(doc['text'])
            for i, chunk in enumerate(doc_chunks):
                chunk['metadata'] = doc.get('metadata', {})
                chunk['metadata']['chunk_id'] = i
                chunk['metadata']['collection'] = collection_name
                chunks.append(chunk)

        # 向量化并存储
        for chunk in chunks:
            vector = self.embedding.embed(chunk['text'])

            chunk_id = f"{collection_name}_{hash(chunk['text'])}"

            self.vector_db.add(
                id=chunk_id,
                vector=vector,
                text=chunk['text'],
                metadata=chunk['metadata']
            )

            self.documents[chunk_id] = chunk

    def query(
        self,
        question: str,
        collection: str = None,
        top_k: int = 5,
        use_rerank: bool = True,
        use_hybrid: bool = False
    ) -> Dict:
        """
        查询

        Args:
            question: 问题
            collection: 指定集合
            top_k: 检索数量
            use_rerank: 是否重排
            use_hybrid: 是否混合检索

        Returns:
            {'answer': str, 'sources': List[dict], 'metadata': dict}
        """
        # 1. 检索
        filters = {'collection': collection} if collection else None

        if use_hybrid and self.hybrid_retriever:
            results = self.hybrid_retriever.retrieve(question, top_k * 2, filters)
        else:
            from vector_retriever import VectorRetriever
            retriever = VectorRetriever(self.vector_db, self.embedding)
            results = retriever.retrieve(question, top_k * 2, filters)

        # 2. 重排序
        if use_rerank and self.reranker:
            results = self.reranker.rerank(question, results, top_k)

        # 3. 上下文优化
        context = self.context_optimizer.optimize_context(question, results)

        # 4. 生成
        answer = self._generate_answer(question, context, results)

        return {
            'answer': answer,
            'sources': results,
            'metadata': {
                'num_sources': len(results),
                'total_tokens': self.context_optimizer._estimate_tokens(context)
            }
        }

    def _generate_answer(
        self,
        question: str,
        context: str,
        sources: List[Dict]
    ) -> str:
        """生成回答"""
        prompt = f"""基于以下文档内容回答问题。如果文档中没有答案，直接说明不知道，不要编造。

【文档】
{context}
【文档结束】

问题：{question}

回答要求：
1. 如果有答案，基于文档内容给出完整回答
2. 适当引用文档内容（用[文档X]标注来源）
3. 如果文档中没有答案，说"根据提供的文档，无法回答这个问题"
4. 回答要清晰、有条理

回答："""

        return self.llm.generate(prompt)


# 使用示例
def demo_production_rag():
    """演示生产级RAG"""
    # 初始化（需要替换为实际配置）
    # rag = ProductionRAG(
    #     embedding_model=embedding_model,
    #     vector_store=vector_store,
    #     llm_client=llm,
    #     reranker_model=reranker
    # )

    # 添加文档
    documents = [
        {
            'text': '''
            # Python入门教程

            ## 第一章：基础语法

            Python是一种高级编程语言，由Guido van Rossum于1991年创建。
            Python的语法简洁优雅，适合初学者学习。

            ### 变量和数据类型

            Python中可以使用变量来存储数据：

            ```python
            name = "小明"
            age = 25
            is_student = True
            ```

            ### 基本运算

            Python支持基本的数学运算：

            ```python
            a = 10
            b = 3
            print(a + b)  # 加法：13
            print(a - b)  # 减法：7
            print(a * b)  # 乘法：30
            print(a / b)  # 除法：3.33...
            ```
            ''',
            'metadata': {'source': 'python_tutorial.md', 'type': 'tutorial'}
        },
        {
            'text': '''
            # JavaScript基础

            JavaScript是一种用于Web开发的脚本语言。
            它可以控制网页的行为，实现交互效果。

            ## 变量声明

            JavaScript有三种声明变量的方式：

            ```javascript
            var name = "小明";      // 旧方式
            let age = 25;          // 现代方式
            const PI = 3.14159;     // 常量
            ```

            ## 基本语法

            JavaScript的语法与C语言类似：

            ```javascript
            function greet(name) {
                return "Hello, " + name + "!";
            }

            console.log(greet("World"));
            ```
            ''',
            'metadata': {'source': 'js_tutorial.md', 'type': 'tutorial'}
        }
    ]

    # rag.add_documents(documents)

    # 查询
    # result = rag.query("Python中如何声明变量？")
    # print(f"回答：{result['answer']}")
    # print(f"来源：{len(result['sources'])}个文档")
```

---

## 六、RAG优化技巧总结

```
┌─────────────────────────────────────────────────────────────────────┐
│                         RAG优化技巧速查                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  📄 文档处理优化                                                     │
│  ├─ 选择合适的chunk_size（500-800常用）                              │
│  ├─ 保留段落结构                                                     │
│  ├─ 添加元数据（来源、日期、类型）                                   │
│  └─ 使用语义分割（高级）                                            │
│                                                                      │
│  🔍 检索优化                                                        │
│  ├─ 混合检索：向量+关键词                                           │
│  ├─ 重排序：使用BGE-Reranker                                       │
│  ├─ 多跳检索：先粗筛再精筛                                           │
│  └─ 过滤器：按元数据筛选                                            │
│                                                                      │
│  📝 上下文优化                                                      │
│  ├─ 压缩无关内容                                                    │
│  ├─ 保留关键数据和引用                                              │
│  ├─ 按主题分组（树状上下文）                                        │
│  └─ 添加前后文窗口                                                   │
│                                                                      │
│  🎯 生成优化                                                        │
│  ├─ 明确要求引用来源                                                │
│  ├─ 要求回答在文档范围内                                            │
│  ├─ 处理"不知道"的情况                                              │
│  └─ Chain-of-Thought（复杂问题）                                    │
│                                                                      │
│  ⚠️ 常见问题                                                        │
│  ├─ 检索不准确 → 优化embedding或chunk策略                           │
│  ├─ 上下文太长 → 压缩或分批                                         │
│  ├─ 幻觉问题 → 要求引用原文                                          │
│  └─ 速度慢 → 缓存或简化流程                                         │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 相关主题

- [[上下文窗口深度解析]] - 理解RAG的上下文限制
- [[上下文压缩技术]] - 压缩检索结果
- [[FewShot示例选择]] - 在RAG中使用Few-shot

---

## 参考文献

1. Lewis, P., et al. (2020). **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.** NeurIPS.
2. Gao, Y., et al. (2023). **Retrieval-Augmented Generation for Large Language Models: A Survey.** arXiv.
3. Glass, M., et al. (2022). **Naver Labs' approaches at TREC 2022.**
