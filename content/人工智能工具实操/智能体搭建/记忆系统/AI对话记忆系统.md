---
title: AI对话记忆系统：从原理到实现
date: 2026-04-24
tags:
  - 对话记忆
  - 短期记忆
  - 长期记忆
  - 向量存储
  - 知识图谱
  - 个性化
  - RAG
categories:
  - 智能体搭建
  - 记忆系统
description: 详细介绍AI对话记忆系统的设计与实现，包括短期记忆（Buffer Memory）、长期记忆（向量存储）、知识图谱记忆以及个性化记忆系统，通过详细的架构设计和代码实现，帮助开发者构建具备持久上下文理解能力的AI智能体。
---

# AI对话记忆系统：从原理到实现

> [!NOTE] 这篇指南讲什么
> 这是AI记忆系统的完整教程，从理论原理到代码实现，手把手教你打造"会记住"的AI助手。不管是简单的客服Bot还是复杂的Agent系统，记忆都是核心能力之一。

## 什么是记忆系统？

### 为什么AI需要记忆？

普通的LLM调用是这样的：

```
用户: 你好
助手: 你好！

用户: 我昨天买了个手机
助手: 好的

用户: 什么时候发货的？
助手: 抱歉，我不知道你买手机的事
```

每次对话都是独立的，AI不记得之前说过什么。这就是"没有记忆"的问题。

有了记忆系统之后：

```
用户: 你好
助手: 你好！

用户: 我昨天买了个手机，订单号12345
助手: 好的，我已经记住了（存入记忆）

用户: 什么时候发货的？
助手: 让我查一下（从记忆中读取订单12345）...是今天上午9点发货的
```

### 记忆系统的层级

一个完善的记忆系统通常包含多个层级：

```
┌─────────────────────────────────────────┐
│            上下文窗口（Context Window）         │
│  用户输入 → 短期记忆 → 长期记忆 → LLM处理      │
└─────────────────────────────────────────┘

层级说明：
- 上下文窗口：模型能看到的token数量（有限）
- 短期记忆：当前会话的对话历史（有限）
- 长期记忆：跨会话的持久信息（无限）
- 知识图谱：结构化的实体关系（无限）
```

## 记忆类型详解

### 短期记忆（Short-term Memory）

短期记忆存储当前会话的对话历史，是最基础的记忆形式。

**特点：**
- 存在内存中，会话结束消失
- 容量有限（受token限制）
- 响应最快
- 实现最简单

**实现方式：**
- Buffer Memory：固定窗口，只保留最近N条
- Sliding Window：滑动窗口，基于token数动态调整

### 长期记忆（Long-term Memory）

长期记忆是跨会话持久化的信息，能让AI记住用户偏好、历史交互。

**特点：**
- 持久存储（数据库/文件）
- 容量无限
- 需要检索才能使用
- 实现相对复杂

**实现方式：**
- 向量存储：文本向量化后存储，检索时计算相似度
- 键值存储：结构化数据存储，精准匹配

### 工作记忆（Working Memory）

工作记忆是Agent执行任务时的临时存储区域。

**特点：**
- 任务内有效
- 容量小
- 用于多步骤推理

### 情景记忆（Episodic Memory）

情景记忆记录用户和AI之间的交互历史事件。

**特点：**
- 时间序列组织
- 可检索特定事件
- 支持"回忆"功能

## 短期记忆实现

### Buffer Memory（固定窗口）

最简单的记忆实现，只保留固定数量的对话：

```python
from typing import List, Dict, Any, Optional
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class Message:
    """消息结构"""
    role: str  # system/user/assistant
    content: str
    timestamp: datetime = field(default_factory=datetime.now)
    metadata: Dict[str, Any] = field(default_factory=dict)

class BufferMemory:
    """Buffer Memory - 固定窗口短期记忆"""
    
    def __init__(self, window_size: int = 10):
        """
        初始化记忆
        :param window_size: 保留的消息数量
        """
        self.window_size = window_size
        self.messages: List[Message] = []
    
    def add_message(self, role: str, content: str, metadata: Dict = None) -> None:
        """添加消息"""
        message = Message(
            role=role,
            content=content,
            metadata=metadata or {}
        )
        self.messages.append(message)
        
        # 保持窗口大小
        if len(self.messages) > self.window_size:
            self.messages.pop(0)
    
    def get_messages(self) -> List[Message]:
        """获取所有消息"""
        return self.messages.copy()
    
    def get_context(self) -> str:
        """获取格式化上下文"""
        return "\n".join([
            f"{msg.role}: {msg.content}"
            for msg in self.messages
        ])
    
    def clear(self) -> None:
        """清空记忆"""
        self.messages.clear()
    
    def get_token_count(self) -> int:
        """估算Token数（简单估算）"""
        # 粗略估算：1 token ≈ 4个字符
        return sum(len(msg.content) // 4 for msg in self.messages)


# 使用示例
memory = BufferMemory(window_size=10)

memory.add_message("user", "你好，我想买一部手机")
memory.add_message("assistant", "好的，请问您有什么品牌和预算要求？")
memory.add_message("user", "想要华为的，预算5000左右")

print(memory.get_context())
# 输出:
# user: 你好，我想买一部手机
# assistant: 好的，请问您有什么品牌和预算要求？
# user: 想要华为的，预算5000左右
```

### Sliding Window Memory（滑动窗口）

基于Token数动态调整的记忆，不会丢失系统提示词：

```python
class SlidingWindowMemory:
    """滑动窗口记忆 - 基于Token数"""
    
    def __init__(self, max_tokens: int = 4000):
        self.max_tokens = max_tokens
        self.messages: List[Message] = []
        self.system_messages: List[Message] = []  # 保留系统消息
    
    def add_message(self, role: str, content: str) -> None:
        """添加消息，自动滑动"""
        message = Message(role=role, content=content)
        
        # 系统消息单独存储
        if role == "system":
            self.system_messages.append(message)
        else:
            self.messages.append(message)
        
        # 裁剪以满足token限制
        self._trim_to_token_limit()
    
    def _trim_to_token_limit(self) -> None:
        """根据Token限制裁剪"""
        while self.get_token_count() > self.max_tokens and len(self.messages) > 1:
            # 优先保留系统消息
            self.messages.pop(0)  # 移除最旧的用户/助手消息
    
    def get_messages(self) -> List[Message]:
        """获取消息（包含系统消息）"""
        return self.system_messages + self.messages
    
    def get_context(self) -> str:
        """获取格式化上下文"""
        parts = [msg.content for msg in self.get_messages()]
        return "\n".join(parts)
    
    def get_token_count(self) -> int:
        """Token估算"""
        return sum(len(m.content) // 4 for m in self.messages)


# 使用示例
memory = SlidingWindowMemory(max_tokens=4000)

memory.add_message("system", "你是一个手机销售助手")
memory.add_message("user", "你好")
memory.add_message("assistant", "您好，有什么可以帮您的？")
memory.add_message("user", "我想买华为手机")
memory.add_message("assistant", "好的，请问您的预算是多少？")
memory.add_message("user", "5000左右")

print(f"当前Token数: {memory.get_token_count()}")
print(memory.get_context())
```

### 对话总结（Summarizing Memory）

当对话太长时，压缩历史为摘要：

```python
class SummarizingMemory:
    """总结记忆 - 对话过长时压缩"""
    
    def __init__(
        self,
        max_messages: int = 20,
        summary_trigger: int = 15,
        llm_client = None
    ):
        self.max_messages = max_messages
        self.summary_trigger = summary_trigger
        self.llm_client = llm_client
        
        self.messages: List[Message] = []
        self.summary: Optional[str] = None
        self.uncompressed_count: int = 0  # 未压缩的消息数
    
    async def summarize_old_messages(self) -> None:
        """将旧消息压缩为摘要"""
        if not self.llm_client or len(self.messages) < self.summary_trigger:
            return
        
        # 获取需要总结的消息
        messages_to_summarize = self.messages[:-5]  # 保留最近5条
        
        # 调用LLM生成摘要
        prompt = f"""
请总结以下对话的要点：

{"".join([f"{m.role}: {m.content}\n" for m in messages_to_summarize])}

请用简洁的语言总结对话的核心内容，保留关键信息。
"""
        
        response = await self.llm_client.chat(prompt)
        
        # 创建摘要消息
        summary_msg = Message(
            role="system",
            content=f"[对话摘要] {response}"
        )
        
        # 压缩：保留摘要 + 最近消息
        self.summary = response
        self.messages = [summary_msg] + self.messages[-5:]
        self.uncompressed_count = 0
    
    def add_message(self, role: str, content: str) -> None:
        """添加消息"""
        self.messages.append(Message(role=role, content=content))
        
        # 检查是否需要总结
        if len(self.messages) >= self.summary_trigger:
            # 异步总结（需要配合事件循环）
            import asyncio
            asyncio.create_task(self.summarize_old_messages())


# 使用示例
memory = SummarizingMemory(
    max_messages=20,
    summary_trigger=15,
    llm_client=openai_client
)
```

## 长期记忆实现

### 向量存储架构

向量存储是长期记忆的核心技术：

```
┌────────────────────────────────────────────────┐
│                   写入流程                        │
│                                                │
│  新对话 → 信息提取 → 向量化 → 存储到向量数据库     │
│                        ↓                       │
│                   存储内容：                     │
│                   - 原始文本                     │
│                   - 向量                        │
│                   - 元数据（用户、时间等）         │
└────────────────────────────────────────────────┘

┌────────────────────────────────────────────────┐
│                   读取流程                        │
│                                                │
│  用户查询 → 查询向量化 → 相似度搜索 → 记忆召回    │
│                        ↓                       │
│                   召回结果：                     │
│                   - 相关对话记录                  │
│                   - 用户偏好                      │
│                   - 历史事件                      │
└────────────────────────────────────────────────┘
```

### 向量存储实现

```python
from typing import List, Dict, Any, Optional, Tuple
import numpy as np
from abc import ABC, abstractmethod
import hashlib
import json
from datetime import datetime

class VectorStore(ABC):
    """向量存储抽象基类"""
    
    @abstractmethod
    async def add(
        self,
        id: str,
        embedding: List[float],
        metadata: Dict[str, Any]
    ) -> None:
        """添加向量"""
        pass
    
    @abstractmethod
    async def search(
        self,
        query_embedding: List[float],
        top_k: int = 5,
        filter: Dict = None
    ) -> List[Dict]:
        """搜索相似向量"""
        pass
    
    @abstractmethod
    async def delete(self, id: str) -> None:
        """删除向量"""
        pass

class InMemoryVectorStore(VectorStore):
    """内存向量存储（开发环境使用）"""
    
    def __init__(self, dimension: int = 1536):
        self.dimension = dimension
        self.vectors: Dict[str, np.ndarray] = {}
        self.metadata: Dict[str, Dict] = {}
    
    async def add(
        self,
        id: str,
        embedding: List[float],
        metadata: Dict[str, Any]
    ) -> None:
        self.vectors[id] = np.array(embedding)
        self.metadata[id] = metadata
    
    async def search(
        self,
        query_embedding: List[float],
        top_k: int = 5,
        filter: Dict = None
    ) -> List[Dict]:
        if not self.vectors:
            return []
        
        query_vec = np.array(query_embedding)
        
        # 计算余弦相似度
        results = []
        for id, vec in self.vectors.items():
            # 过滤
            if filter:
                if not all(
                    self.metadata[id].get(k) == v
                    for k, v in filter.items()
                ):
                    continue
            
            # 相似度计算
            similarity = self._cosine_similarity(query_vec, vec)
            results.append({
                'id': id,
                'score': float(similarity),
                'metadata': self.metadata[id]
            })
        
        # 排序并返回Top-K
        results.sort(key=lambda x: x['score'], reverse=True)
        return results[:top_k]
    
    async def delete(self, id: str) -> None:
        self.vectors.pop(id, None)
        self.metadata.pop(id, None)
    
    @staticmethod
    def _cosine_similarity(a: np.ndarray, b: np.ndarray) -> float:
        """计算余弦相似度"""
        dot = np.dot(a, b)
        norm_a = np.linalg.norm(a)
        norm_b = np.linalg.norm(b)
        return dot / (norm_a * norm_b + 1e-8)


class PineconeVectorStore(VectorStore):
    """Pinecone向量存储（生产环境）"""
    
    def __init__(self, api_key: str, environment: str, index_name: str):
        import pinecone
        self.pinecone = pinecone
        pinecone.init(api_key=api_key, environment=environment)
        self.index = pinecone.Index(index_name)
    
    async def add(
        self,
        id: str,
        embedding: List[float],
        metadata: Dict[str, Any]
    ) -> None:
        await self.index.upsert([(id, embedding, metadata)])
    
    async def search(
        self,
        query_embedding: List[float],
        top_k: int = 5,
        filter: Dict = None
    ) -> List[Dict]:
        result = await self.index.query(
            vector=query_embedding,
            top_k=top_k,
            filter=filter,
            include_metadata=True
        )
        
        return [
            {
                'id': match['id'],
                'score': match['score'],
                'metadata': match.get('metadata', {})
            }
            for match in result['matches']
        ]
    
    async def delete(self, id: str) -> None:
        await self.index.delete(ids=[id])


# 使用示例
async def example():
    # 创建向量存储
    store = InMemoryVectorStore(dimension=1536)
    
    # 添加记忆
    await store.add(
        id="memory_001",
        embedding=[0.1] * 1536,  # 实际的embedding
        metadata={
            'user_id': 'user_123',
            'content': '用户提到想要买华为手机',
            'timestamp': datetime.now().isoformat()
        }
    )
    
    # 搜索相关记忆
    results = await store.search(
        query_embedding=[0.1] * 1536,
        top_k=3,
        filter={'user_id': 'user_123'}
    )
    
    print(results)
```

### 记忆服务封装

```python
class MemoryService:
    """记忆服务 - 统一管理短期和长期记忆"""
    
    def __init__(
        self,
        vector_store: VectorStore,
        embedding_model: str = "text-embedding-3-small",
        short_term_max: int = 10
    ):
        self.vector_store = vector_store
        self.embedding_model = embedding_model
        self.short_term = BufferMemory(window_size=short_term_max)
        self._client = None  # OpenAI client
    
    async def store_interaction(
        self,
        user_id: str,
        session_id: str,
        user_message: str,
        assistant_response: str,
        metadata: Dict = None
    ) -> str:
        """存储对话交互到长期记忆"""
        # 生成唯一ID
        content = f"用户: {user_message}\n助手: {assistant_response}"
        memory_id = self._generate_id(user_id, session_id, content)
        
        # 向量化
        embedding = await self._get_embedding(content)
        
        # 存储
        await self.vector_store.add(
            id=memory_id,
            embedding=embedding,
            metadata={
                'user_id': user_id,
                'session_id': session_id,
                'user_message': user_message,
                'assistant_response': assistant_response,
                'timestamp': datetime.now().isoformat(),
                **(metadata or {})
            }
        )
        
        return memory_id
    
    async def retrieve_memories(
        self,
        user_id: str,
        query: str,
        top_k: int = 5,
        session_id: Optional[str] = None
    ) -> List[Dict]:
        """检索相关记忆"""
        # 向量化查询
        query_embedding = await self._get_embedding(query)
        
        # 构建过滤条件
        filter_conditions = {'user_id': user_id}
        if session_id:
            filter_conditions['session_id'] = session_id
        
        # 检索
        results = await self.vector_store.search(
            query_embedding=query_embedding,
            top_k=top_k,
            filter=filter_conditions
        )
        
        return results
    
    async def _get_embedding(self, text: str) -> List[float]:
        """获取文本向量"""
        if not self._client:
            from openai import OpenAI
            self._client = OpenAI()
        
        response = self._client.embeddings.create(
            model=self.embedding_model,
            input=text
        )
        
        return response.data[0].embedding
    
    def _generate_id(self, *parts) -> str:
        """生成唯一ID"""
        content = "|".join(str(p) for p in parts)
        return hashlib.md5(content.encode()).hexdigest()[:16]
    
    def add_short_term(self, role: str, content: str) -> None:
        """添加到短期记忆"""
        self.short_term.add_message(role, content)
    
    def get_short_term_context(self) -> str:
        """获取短期记忆上下文"""
        return self.short_term.get_context()
    
    async def get_full_context(
        self,
        user_id: str,
        query: str,
        short_term: bool = True,
        long_term: bool = True,
        top_k: int = 3
    ) -> Dict[str, Any]:
        """获取完整上下文"""
        context = {
            'short_term': '',
            'long_term': [],
            'combined': ''
        }
        
        # 短期记忆
        if short_term:
            context['short_term'] = self.get_short_term_context()
        
        # 长期记忆
        if long_term:
            long_term_results = await self.retrieve_memories(
                user_id=user_id,
                query=query,
                top_k=top_k
            )
            context['long_term'] = long_term_results
        
        # 构建组合上下文
        parts = []
        
        if context['short_term']:
            parts.append(f"【近期对话】\n{context['short_term']}")
        
        if context['long_term']:
            parts.append("【相关历史】")
            for item in context['long_term']:
                meta = item['metadata']
                parts.append(f"- 用户曾说：{meta.get('user_message', '')}")
                parts.append(f"  助手回答：{meta.get('assistant_response', '')}")
        
        context['combined'] = "\n\n".join(parts)
        
        return context


# 使用示例
async def main():
    # 初始化
    vector_store = InMemoryVectorStore()
    memory_service = MemoryService(
        vector_store=vector_store,
        embedding_model="text-embedding-3-small"
    )
    
    user_id = "user_123"
    session_id = "session_001"
    
    # 添加短期记忆
    memory_service.add_short_term("user", "你好，我想买一部手机")
    memory_service.add_short_term("assistant", "好的，请问您有什么品牌偏好？")
    memory_service.add_short_term("user", "华为的，预算5000左右")
    
    # 存储到长期记忆
    await memory_service.store_interaction(
        user_id=user_id,
        session_id=session_id,
        user_message="你好，我想买一部手机",
        assistant_response="好的，请问您有什么品牌偏好？",
        metadata={'topic': '手机咨询'}
    )
    
    # 获取完整上下文
    context = await memory_service.get_full_context(
        user_id=user_id,
        query="我的手机买了吗"
    )
    
    print("短期记忆：")
    print(context['short_term'])
    print("\n长期记忆：")
    print(context['long_term'])
    print("\n组合上下文：")
    print(context['combined'])
```

## 知识图谱记忆

### 知识图谱概念

知识图谱是一种结构化的记忆形式，存储实体和它们之间的关系：

```
┌────────────────────────────────────────────────┐
│                   知识图谱示例                     │
│                                                │
│              ┌─────────┐                        │
│              │  用户A  │                        │
│              └────┬────┘                        │
│                   │ 购买                         │
│         ┌────────┴────────┐                    │
│         ↓                  ↓                     │
│   ┌─────────┐        ┌─────────┐                │
│   │ 华为手机 │        │ 苹果手机 │                │
│   └────┬────┘        └─────────┘                │
│        │ 使用                                │
│        ↓                                      │
│   ┌─────────┐                                │
│   │ EMUI系统 │                                │
│   └─────────┘                                │
└────────────────────────────────────────────────┘
```

### 知识图谱实现

```python
from typing import List, Dict, Any, Optional, Set
from dataclasses import dataclass, field
from enum import Enum

class EntityType(Enum):
    """实体类型"""
    PERSON = "person"
    PRODUCT = "product"
    ORGANIZATION = "organization"
    LOCATION = "location"
    EVENT = "event"
    CONCEPT = "concept"

class RelationType(Enum):
    """关系类型"""
    PURCHASED = "purchased"
    USES = "uses"
    WORKS_FOR = "works_for"
    LOCATED_IN = "located_in"
    RELATED_TO = "related_to"
    IS_A = "is_a"

@dataclass
class Entity:
    """实体"""
    id: str
    type: EntityType
    name: str
    properties: Dict[str, Any] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.now)

@dataclass
class Relation:
    """关系"""
    id: str
    source_id: str
    target_id: str
    type: RelationType
    properties: Dict[str, Any] = field(default_factory=dict)
    created_at: datetime = field(default_factory=datetime.now)

class KnowledgeGraph:
    """知识图谱"""
    
    def __init__(self):
        self.entities: Dict[str, Entity] = {}
        self.relations: Dict[str, Relation] = {}
        self.adjacency: Dict[str, Set[str]] = {}  # 邻接表
        self.entity_index: Dict[str, List[str]] = {}  # 实体名称索引
    
    def add_entity(self, entity: Entity) -> None:
        """添加实体"""
        self.entities[entity.id] = entity
        self.adjacency.setdefault(entity.id, set())
        
        # 更新索引
        name_key = entity.name.lower()
        if name_key not in self.entity_index:
            self.entity_index[name_key] = []
        self.entity_index[name_key].append(entity.id)
    
    def add_relation(self, relation: Relation) -> None:
        """添加关系"""
        self.relations[relation.id] = relation
        
        # 更新邻接表
        self.adjacency.setdefault(relation.source_id, set()).add(relation.target_id)
        self.adjacency.setdefault(relation.target_id, set()).add(relation.source_id)
    
    def get_entity(self, entity_id: str) -> Optional[Entity]:
        """获取实体"""
        return self.entities.get(entity_id)
    
    def find_entity_by_name(self, name: str) -> Optional[Entity]:
        """通过名称查找实体"""
        name_key = name.lower()
        entity_ids = self.entity_index.get(name_key, [])
        if entity_ids:
            return self.entities.get(entity_ids[0])
        return None
    
    def get_neighbors(
        self,
        entity_id: str,
        depth: int = 1,
        relation_type: RelationType = None
    ) -> List[tuple]:
        """获取邻居节点"""
        result = []
        visited = set()
        
        def dfs(current_id: str, current_depth: int):
            if current_depth > depth:
                return
            
            for relation in self._get_relations(current_id):
                if relation_type and relation.type != relation_type:
                    continue
                
                neighbor_id = relation.target_id if relation.source_id == current_id else relation.source_id
                
                if neighbor_id not in visited:
                    visited.add(neighbor_id)
                    result.append((neighbor_id, relation))
                    dfs(neighbor_id, current_depth + 1)
        
        dfs(entity_id, 0)
        return result
    
    def _get_relations(self, entity_id: str) -> List[Relation]:
        """获取实体的所有关系"""
        return [
            r for r in self.relations.values()
            if r.source_id == entity_id or r.target_id == entity_id
        ]
    
    def query_path(
        self,
        source_id: str,
        target_id: str,
        max_depth: int = 3
    ) -> List[List[str]]:
        """查询两点间的路径"""
        paths = []
        
        def dfs(current: str, path: List[str]):
            if len(path) > max_depth:
                return
            if current == target_id:
                paths.append(path.copy())
                return
            
            for neighbor in self.adjacency.get(current, set()):
                if neighbor not in path:
                    path.append(neighbor)
                    dfs(neighbor, path)
                    path.pop()
        
        dfs(source_id, [source_id])
        return paths
    
    def get_subgraph(
        self,
        entity_id: str,
        depth: int = 2
    ) -> Dict[str, Any]:
        """获取子图"""
        visited = {entity_id}
        nodes = []
        edges = []
        
        def dfs(current_id: str, current_depth: int):
            if current_depth > depth:
                return
            
            for neighbor_id, relation in self.get_neighbors(current_id, depth=1):
                if neighbor_id not in visited:
                    visited.add(neighbor_id)
                    nodes.append(self.entities.get(neighbor_id))
                
                edges.append({
                    'source': relation.source_id,
                    'target': relation.target_id,
                    'type': relation.type.value
                })
                
                dfs(neighbor_id, current_depth + 1)
        
        dfs(entity_id, 0)
        
        return {
            'entities': [e for e in nodes if e],
            'relations': edges
        }


# 使用示例
def example():
    kg = KnowledgeGraph()
    
    # 添加实体
    user = Entity(
        id="user_001",
        type=EntityType.PERSON,
        name="张三",
        properties={'偏好': '华为手机'}
    )
    phone = Entity(
        id="product_001",
        type=EntityType.PRODUCT,
        name="华为Mate60",
        properties={'价格': 6999, '品牌': '华为'}
    )
    
    kg.add_entity(user)
    kg.add_entity(phone)
    
    # 添加关系
    relation = Relation(
        id="rel_001",
        source_id="user_001",
        target_id="product_001",
        type=RelationType.PURCHASED,
        properties={'时间': '2024-01-15', '价格': 6999}
    )
    kg.add_relation(relation)
    
    # 查询
    print(kg.get_entity("user_001"))
    print(kg.find_entity_by_name("华为"))
    
    # 获取邻居
    neighbors = kg.get_neighbors("user_001")
    for neighbor_id, relation in neighbors:
        entity = kg.entities.get(neighbor_id)
        print(f"关联实体: {entity.name}, 关系: {relation.type.value}")
```

### 从对话中提取知识

```python
class KnowledgeExtractor:
    """知识提取器 - 从对话中提取实体和关系"""
    
    def __init__(self, llm_client):
        self.llm = llm_client
    
    async def extract_from_conversation(
        self,
        conversation: str
    ) -> Dict[str, Any]:
        """从对话中提取知识"""
        prompt = f"""
从以下对话中提取实体和关系：

{conversation}

请以JSON格式返回：
{{
    "entities": [
        {{
            "name": "实体名",
            "type": "实体类型(person/product/organization/location/event/concept)",
            "properties": {{}}
        }}
    ],
    "relations": [
        {{
            "source": "实体1名称",
            "target": "实体2名称",
            "type": "关系类型(purchased/uses/works_for/located_in/related_to/is_a)"
        }}
    ]
}}

实体类型：person, product, organization, location, event, concept
关系类型：purchased, uses, works_for, located_in, related_to, is_a
"""
        
        response = await self.llm.chat(prompt)
        
        # 解析JSON响应
        import json
        try:
            return json.loads(response)
        except:
            return {"entities": [], "relations": []}
    
    async def extract_and_store(
        self,
        conversation: str,
        knowledge_graph: KnowledgeGraph
    ) -> None:
        """提取并存储到知识图谱"""
        extraction = await self.extract_from_conversation(conversation)
        
        # 存储实体
        for entity_data in extraction.get("entities", []):
            entity = Entity(
                id=self._generate_entity_id(entity_data["name"]),
                type=EntityType(entity_data["type"]),
                name=entity_data["name"],
                properties=entity_data.get("properties", {})
            )
            knowledge_graph.add_entity(entity)
        
        # 存储关系
        for relation_data in extraction.get("relations", []):
            source = knowledge_graph.find_entity_by_name(relation_data["source"])
            target = knowledge_graph.find_entity_by_name(relation_data["target"])
            
            if source and target:
                relation = Relation(
                    id=self._generate_relation_id(source.id, target.id),
                    source_id=source.id,
                    target_id=target.id,
                    type=RelationType(relation_data["type"])
                )
                knowledge_graph.add_relation(relation)
    
    def _generate_entity_id(self, name: str) -> str:
        import hashlib
        return hashlib.md5(f"entity:{name}".encode()).hexdigest()[:16]
    
    def _generate_relation_id(self, source: str, target: str) -> str:
        import hashlib
        return hashlib.md5(f"rel:{source}:{target}".encode()).hexdigest()[:16]
```

## 记忆检索与融合

### 多记忆源检索

```python
class MemoryRetriever:
    """记忆检索器 - 整合多个记忆源"""
    
    def __init__(
        self,
        short_term: BufferMemory,
        long_term: MemoryService,
        knowledge_graph: KnowledgeGraph
    ):
        self.short_term = short_term
        self.long_term = long_term
        self.knowledge_graph = knowledge_graph
    
    async def retrieve(
        self,
        user_id: str,
        query: str,
        include_short_term: bool = True,
        include_long_term: bool = True,
        include_graph: bool = True,
        top_k: int = 5
    ) -> Dict[str, Any]:
        """多源检索"""
        results = {
            'short_term': [],
            'long_term': [],
            'graph': [],
            'context': []
        }
        
        # 短期记忆检索
        if include_short_term:
            results['short_term'] = self._search_short_term(query)
        
        # 长期记忆检索
        if include_long_term:
            results['long_term'] = await self.long_term.retrieve_memories(
                user_id=user_id,
                query=query,
                top_k=top_k
            )
        
        # 知识图谱检索
        if include_graph:
            results['graph'] = self._search_graph(query)
        
        # 融合构建上下文
        results['context'] = self._fuse_context(results)
        
        return results
    
    def _search_short_term(self, query: str) -> List[Dict]:
        """搜索短期记忆"""
        messages = self.short_term.get_messages()
        relevant = []
        query_words = set(query.lower().split())
        
        for msg in messages:
            msg_words = set(msg.content.lower().split())
            # 计算重叠度
            overlap = len(query_words & msg_words)
            if overlap > 0:
                relevant.append({
                    'role': msg.role,
                    'content': msg.content,
                    'score': overlap / len(query_words | msg_words)
                })
        
        return relevant
    
    def _search_graph(self, query: str) -> List[Dict]:
        """搜索知识图谱"""
        results = []
        query_words = query.lower().split()
        
        for entity in self.knowledge_graph.entities.values():
            if any(word in entity.name.lower() for word in query_words):
                neighbors = self.knowledge_graph.get_neighbors(entity.id, depth=1)
                results.append({
                    'entity': entity,
                    'neighbors': [
                        (neighbor_id, rel)
                        for neighbor_id, rel in neighbors
                    ]
                })
        
        return results
    
    def _fuse_context(self, results: Dict) -> str:
        """融合多源记忆构建上下文"""
        context_parts = []
        
        # 添加图谱知识
        if results['graph']:
            context_parts.append("【相关背景知识】")
            for item in results['graph'][:2]:
                entity = item['entity']
                context_parts.append(f"- {entity.name}（{entity.type.value}）")
                for neighbor_id, rel in item['neighbors'][:2]:
                    neighbor = self.knowledge_graph.entities.get(neighbor_id)
                    if neighbor:
                        context_parts.append(f"  - 与{neighbor.name}的关系：{rel.type.value}")
        
        # 添加历史对话
        if results['long_term']:
            context_parts.append("\n【相关历史对话】")
            for item in results['long_term'][:2]:
                meta = item['metadata']
                context_parts.append(f"用户：{meta.get('user_message', '')}")
                context_parts.append(f"助手：{meta.get('assistant_response', '')}")
        
        # 添加当前对话
        if results['short_term']:
            context_parts.append("\n【当前对话】")
            for item in results['short_term']:
                context_parts.append(f"{item['role']}：{item['content']}")
        
        return "\n".join(context_parts)
```

### 上下文组装

```python
class ContextBuilder:
    """上下文构建器 - 构建完整的对话上下文"""
    
    def __init__(
        self,
        max_tokens: int = 8000,
        system_prompt: str = ""
    ):
        self.max_tokens = max_tokens
        self.system_prompt = system_prompt
    
    def build(
        self,
        system_template: str,
        memory_context: str,
        recent_messages: List[Message],
        user_input: str
    ) -> List[Dict]:
        """构建完整的对话上下文"""
        messages = []
        
        # 系统提示词
        if self.system_prompt:
            messages.append({
                "role": "system",
                "content": self.system_prompt
            })
        
        # 记忆上下文
        if memory_context:
            messages.append({
                "role": "system",
                "content": f"【相关记忆】\n{memory_context}"
            })
        
        # 近期消息
        for msg in recent_messages:
            messages.append({
                "role": msg.role,
                "content": msg.content
            })
        
        # 用户输入
        messages.append({
            "role": "user",
            "content": user_input
        })
        
        # Token裁剪
        messages = self._trim_to_token_limit(messages)
        
        return messages
    
    def _trim_to_token_limit(self, messages: List[Dict]) -> List[Dict]:
        """根据Token限制裁剪"""
        while self._count_tokens(messages) > self.max_tokens and len(messages) > 2:
            # 优先裁剪早期消息
            if len(messages) > 2 and messages[1]["role"] == "system":
                # 裁剪记忆上下文
                if len(messages[1]["content"]) > 500:
                    messages[1]["content"] = messages[1]["content"][:500] + "\n...(已截断)"
                else:
                    messages.pop(1)
            elif len(messages) > 1:
                messages.pop(1)
            else:
                break
        
        return messages
    
    def _count_tokens(self, messages: List[Dict]) -> int:
        """估算Token数"""
        return sum(
            len(m["content"]) // 4
            for m in messages
        )
```

## 个性化记忆

### 用户画像存储

```python
from typing import Dict, Any
from dataclasses import dataclass, asdict
from typing import Optional, List

@dataclass
class UserProfile:
    """用户画像"""
    user_id: str
    name: Optional[str] = None
    
    # 偏好设置
    preferences: Dict[str, Any] = None
    
    # 沟通风格
    interaction_style: str = "formal"  # formal/casual/technical
    
    # 兴趣话题
    topics_of_interest: List[str] = None
    
    # 沟通模式
    communication_patterns: Dict[str, Any] = None
    
    # 最后更新
    last_updated: datetime = None
    
    def __post_init__(self):
        if self.preferences is None:
            self.preferences = {}
        if self.topics_of_interest is None:
            self.topics_of_interest = []
        if self.communication_patterns is None:
            self.communication_patterns = {}
        if self.last_updated is None:
            self.last_updated = datetime.now()

class UserProfileManager:
    """用户画像管理器"""
    
    def __init__(self, db=None):
        self.db = db  # 数据库连接
        self._cache: Dict[str, UserProfile] = {}
    
    async def get_profile(self, user_id: str) -> UserProfile:
        """获取用户画像"""
        # 先查缓存
        if user_id in self._cache:
            return self._cache[user_id]
        
        # 从数据库加载
        if self.db:
            data = await self.db.user_profiles.find_one({"user_id": user_id})
            if data:
                profile = UserProfile(**data)
            else:
                profile = UserProfile(user_id=user_id)
        else:
            profile = UserProfile(user_id=user_id)
        
        self._cache[user_id] = profile
        return profile
    
    async def update_profile(
        self,
        user_id: str,
        updates: Dict[str, Any]
    ) -> None:
        """更新用户画像"""
        profile = await self.get_profile(user_id)
        
        # 更新字段
        for key, value in updates.items():
            if hasattr(profile, key):
                setattr(profile, key, value)
        
        profile.last_updated = datetime.now()
        
        # 保存到数据库
        if self.db:
            await self.db.user_profiles.update_one(
                {"user_id": user_id},
                {"$set": asdict(profile)},
                upsert=True
            )
        
        # 更新缓存
        self._cache[user_id] = profile
    
    async def learn_from_interaction(
        self,
        user_id: str,
        user_message: str,
        assistant_response: str
    ) -> None:
        """从交互中学习用户偏好"""
        profile = await self.get_profile(user_id)
        
        # 学习沟通风格
        casual_indicators = ["!", "?", "哈", "哈", "~", "啦"]
        technical_indicators = ["分析", "评估", "建议", "原因"]
        
        if any(ind in user_message for ind in casual_indicators):
            profile.interaction_style = "casual"
        elif any(ind in user_message for ind in technical_indicators):
            profile.interaction_style = "technical"
        
        # 学习兴趣话题
        topic_keywords = {
            "技术": ["代码", "开发", "API", "系统", "技术"],
            "商业": ["市场", "销售", "运营", "增长", "商业"],
            "创意": ["设计", "创意", "灵感", "想法", "设计"]
        }
        
        for topic, keywords in topic_keywords.items():
            if any(kw in user_message for kw in keywords):
                if topic not in profile.topics_of_interest:
                    profile.topics_of_interest.append(topic)
        
        await self.update_profile(user_id, asdict(profile))
```

### 个性化响应适配

```python
class PersonalizedResponseBuilder:
    """个性化响应构建器"""
    
    def __init__(self, profile_manager: UserProfileManager):
        self.profile_manager = profile_manager
    
    async def adapt_response(
        self,
        base_response: str,
        user_id: str
    ) -> str:
        """根据用户画像适配响应"""
        profile = await self.profile_manager.get_profile(user_id)
        
        response = base_response
        
        # 根据用户偏好调整
        if profile.interaction_style == "casual":
            response = self._make_casual(response)
        elif profile.interaction_style == "technical":
            response = self._add_technical_depth(response)
        
        # 添加个性化元素
        if profile.topics_of_interest:
            response = self._add_personalized_content(response, profile)
        
        return response
    
    def _make_casual(self, text: str) -> str:
        """转为休闲风格"""
        replacements = {
            "您好": "嗨",
            "请问": "有啥",
            "感谢": "谢啦",
            "如果": "要是",
            "可以": "能",
            "帮助": "帮忙"
        }
        for formal, casual in replacements.items():
            text = text.replace(formal, casual)
        return text
    
    def _add_technical_depth(self, text: str) -> str:
        """添加技术细节"""
        return text + "\n\n如果需要更详细的技术说明，随时告诉我。"
    
    def _add_personalized_content(
        self,
        text: str,
        profile: UserProfile
    ) -> str:
        """添加个性化内容"""
        if "技术" in profile.topics_of_interest:
            text += "\n\n另外，关于技术方面的问题，我可以帮你深入分析。"
        if "商业" in profile.topics_of_interest:
            text += "\n\n如果需要市场分析或商业建议，也可以问我。"
        
        return text
```

## 完整实战案例

### 智能对话Agent

```python
class ConversationalAgent:
    """具备完整记忆能力的对话Agent"""
    
    def __init__(self, config: dict):
        # 初始化各组件
        self.short_term = SlidingWindowMemory(
            max_tokens=config.get('short_term_tokens', 4000)
        )
        self.long_term = MemoryService(
            vector_store=self._init_vector_store(config),
            embedding_model=config.get('embedding_model', 'text-embedding-3-small')
        )
        self.knowledge_graph = KnowledgeGraph()
        self.retriever = MemoryRetriever(
            short_term=self.short_term,
            long_term=self.long_term,
            knowledge_graph=self.knowledge_graph
        )
        self.context_builder = ContextBuilder(
            max_tokens=config.get('context_limit', 8000),
            system_prompt=config.get('system_prompt', '')
        )
        self.llm = OpenAIChatClient(config['model'])
        self.profile_manager = UserProfileManager()
        self.response_adapter = PersonalizedResponseBuilder(self.profile_manager)
    
    def _init_vector_store(self, config: dict):
        """初始化向量存储"""
        if config.get('use_pinecone'):
            return PineconeVectorStore(
                api_key=config['pinecone_api_key'],
                environment=config['pinecone_env'],
                index_name=config['pinecone_index']
            )
        return InMemoryVectorStore()
    
    async def chat(
        self,
        user_id: str,
        message: str,
        session_id: str = None
    ) -> str:
        """处理对话"""
        self.session_id = session_id or f"session_{int(time.time())}"
        
        # 1. 检索相关记忆
        memories = await self.retriever.retrieve(
            user_id=user_id,
            query=message,
            include_long_term=True,
            include_graph=True
        )
        
        # 2. 构建上下文
        messages = self.context_builder.build(
            system_template=self.system_prompt,
            memory_context=memories['context'],
            recent_messages=self.short_term.get_messages(),
            user_input=message
        )
        
        # 3. 调用LLM
        response = await self.llm.chat(messages)
        
        # 4. 更新记忆
        await self._update_memories(
            user_id=user_id,
            user_message=message,
            assistant_response=response
        )
        
        # 5. 学习用户偏好
        await self.profile_manager.learn_from_interaction(
            user_id=user_id,
            user_message=message,
            assistant_response=response
        )
        
        # 6. 个性化适配
        response = await self.response_adapter.adapt_response(
            base_response=response,
            user_id=user_id
        )
        
        return response
    
    async def _update_memories(
        self,
        user_id: str,
        user_message: str,
        assistant_response: str
    ) -> None:
        """更新各类记忆"""
        # 短期记忆
        self.short_term.add_message("user", user_message)
        self.short_term.add_message("assistant", assistant_response)
        
        # 长期记忆（异步）
        await self.long_term.store_interaction(
            user_id=user_id,
            session_id=self.session_id,
            user_message=user_message,
            assistant_response=assistant_response
        )
        
        # 知识图谱（异步）
        extractor = KnowledgeExtractor(self.llm)
        knowledge = await extractor.extract_from_conversation(
            f"用户：{user_message}\n助手：{assistant_response}"
        )
        # 存储到图谱（简化处理）
        # 实际项目中需要完善实现


# 使用示例
async def main():
    agent = ConversationalAgent({
        'model': 'gpt-4o',
        'short_term_tokens': 4000,
        'context_limit': 8000,
        'embedding_model': 'text-embedding-3-small',
        'system_prompt': '你是一个专业友好的AI助手。'
    })
    
    # 第一次对话
    response1 = await agent.chat(
        user_id="user_123",
        message="你好，我叫张三，我喜欢技术类的内容",
        session_id="session_001"
    )
    print(f"Agent: {response1}")
    
    # 第二次对话
    response2 = await agent.chat(
        user_id="user_123",
        message="我之前买了个什么手机来着？",
        session_id="session_001"
    )
    print(f"Agent: {response2}")
```

## 总结

一个完善的记忆系统应该包含：

| 层级 | 存储方式 | 容量 | 生命周期 | 实现复杂度 |
|------|----------|------|----------|------------|
| 上下文窗口 | LLM内存 | Token限制 | 单次调用 | 无 |
| 短期记忆 | 内存 | Token限制 | 会话内 | 低 |
| 长期记忆 | 向量数据库 | 无限 | 持久 | 中 |
| 知识图谱 | 图数据库 | 无限 | 持久 | 高 |
| 用户画像 | 结构化存储 | 有限 | 持久 | 中 |

**设计建议：**
- 短期记忆必做，实现简单效果明显
- 长期记忆视需求而定，复杂场景需要
- 知识图谱成本高，适合专业领域应用
- 用户画像提升个性化体验

---

## 相关资源

- [[n8n与LLM集成]] - n8n记忆节点配置
- [[多Agent系统设计]] - Agent间知识共享
- [[Function Calling与工具调用]] - 工具调用设计

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
