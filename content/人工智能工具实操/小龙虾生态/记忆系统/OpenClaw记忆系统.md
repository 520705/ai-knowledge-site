---
title: OpenClaw记忆系统
date: 2026-04-18
tags:
  - openclaw
  - 记忆系统
  - 短期记忆
  - 长期记忆
  - 向量数据库
  - 语义检索
  - 持久化
  - 跨会话
  - RAG
  - 记忆分层
categories:
  - 人工智能工具实操/小龙虾生态/记忆系统
alias: OpenClaw Memory System
---

## 关键词

| 关键词 | 说明 |
|--------|------|
| Working Memory | 工作记忆/短期记忆 |
| Episodic Memory | 情景记忆/会话记忆 |
| Semantic Memory | 语义记忆/长期记忆 |
| Vector Store | 向量存储与检索 |
| Embedding Model | 文本向量化模型 |
| Similarity Search | 相似性搜索 |
| Memory Consolidation | 记忆巩固机制 |
| Context Window | 上下文窗口管理 |
| Memory Retrieval | 记忆检索策略 |
| Cross-Session | 跨会话持久化 |

---

## 概述

OpenClaw 的记忆系统是其作为长期个人 AI Agent 的核心基础设施。它模拟人类记忆的分层结构，实现了从即时工作记忆到长期语义记忆的完整体系，使 Agent 能够在多次会话中保持一致性和连续性。

> [!note] 设计理念
> 人类记忆并非单一系统，而是由工作记忆、情景记忆和语义记忆组成的层级结构。OpenClaw 采用相同的设计哲学，通过分层记忆系统实现「既懂当下，又记得过去」的能力。

---

## 记忆分层架构

### 三层记忆模型

```python
# openclaw/memory/architecture.py
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import List, Optional, Dict, Any, Tuple
from datetime import datetime, timedelta
from enum import Enum
import asyncio

class MemoryType(Enum):
    """记忆类型枚举"""
    WORKING = "working"          # 工作记忆（当前会话）
    EPISODIC = "episodic"        # 情景记忆（会话级）
    SEMANTIC = "semantic"        # 语义记忆（长期）

class MemoryImportance(Enum):
    """记忆重要性等级"""
    CRITICAL = 5    # 必须记住
    HIGH = 4        # 高度重要
    MEDIUM = 3      # 一般重要
    LOW = 2         # 可选记住
    MINIMAL = 1     # 最小保留

@dataclass
class MemoryItem:
    """记忆条目"""
    id: str
    content: str
    memory_type: MemoryType
    importance: MemoryImportance
    embedding: Optional[List[float]] = None
    created_at: datetime = field(default_factory=datetime.now)
    last_accessed: datetime = field(default_factory=datetime.now)
    access_count: int = 0
    metadata: Dict[str, Any] = field(default_factory=dict)
    tags: List[str] = field(default_factory=list)
    source: str = ""  # 来源标识
    
    def access(self):
        """记录访问"""
        self.last_accessed = datetime.now()
        self.access_count += 1
    
    def to_context_string(self) -> str:
        """转换为上下文字符串"""
        return f"[{self.memory_type.value}] {self.content}"

@dataclass
class MemoryQuery:
    """记忆查询请求"""
    query: str
    memory_types: List[MemoryType] = None
    min_importance: MemoryImportance = None
    tags: List[str] = None
    since: datetime = None
    limit: int = 10
    require_recency: bool = False

@dataclass
class MemoryResult:
    """记忆查询结果"""
    items: List[MemoryItem]
    scores: List[float]
    total_available: int

class MemoryLayer(ABC):
    """记忆层抽象基类"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
    
    @abstractmethod
    async def store(self, item: MemoryItem) -> str:
        """存储记忆"""
        pass
    
    @abstractmethod
    async def retrieve(self, query: MemoryQuery) -> MemoryResult:
        """检索记忆"""
        pass
    
    @abstractmethod
    async def update(self, item: MemoryItem) -> bool:
        """更新记忆"""
        pass
    
    @abstractmethod
    async def delete(self, item_id: str) -> bool:
        """删除记忆"""
        pass
    
    @abstractmethod
    async def exists(self, item_id: str) -> bool:
        """检查记忆是否存在"""
        pass

class MemoryManager:
    """记忆管理器"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.layers: Dict[MemoryType, MemoryLayer] = {}
        self.embedding_model = self._load_embedding_model()
        self.consolidation_service = ConsolidationService(self)
        self.retrieval_service = RetrievalService(self)
    
    def register_layer(self, memory_type: MemoryType, layer: MemoryLayer):
        """注册记忆层"""
        self.layers[memory_type] = layer
    
    async def add_memory(
        self,
        content: str,
        memory_type: MemoryType = MemoryType.EPISODIC,
        importance: MemoryImportance = MemoryImportance.MEDIUM,
        **metadata
    ) -> MemoryItem:
        """添加记忆"""
        
        # 生成嵌入向量
        embedding = await self.embedding_model.encode(content)
        
        item = MemoryItem(
            id=self._generate_id(),
            content=content,
            memory_type=memory_type,
            importance=importance,
            embedding=embedding,
            metadata=metadata
        )
        
        # 存储到对应层
        layer = self.layers.get(memory_type)
        if layer:
            await layer.store(item)
        
        # 检查是否需要提升到更高层
        if importance == MemoryImportance.CRITICAL:
            await self._promote_to_semantic(item)
        
        return item
    
    async def search_memory(
        self,
        query: str,
        memory_types: List[MemoryType] = None,
        limit: int = 10
    ) -> List[MemoryItem]:
        """跨层记忆搜索"""
        
        if memory_types is None:
            memory_types = list(MemoryType)
        
        query_embedding = await self.embedding_model.encode(query)
        query_obj = MemoryQuery(
            query=query,
            memory_types=memory_types,
            limit=limit
        )
        
        results = []
        scores = []
        
        for mem_type in memory_types:
            layer = self.layers.get(mem_type)
            if layer:
                result = await layer.retrieve(query_obj)
                results.extend(result.items)
                scores.extend(result.scores)
        
        # 按相关性排序并返回
        paired = list(zip(results, scores))
        paired.sort(key=lambda x: x[1], reverse=True)
        
        return [item for item, score in paired[:limit]]
    
    async def _promote_to_semantic(self, item: MemoryItem):
        """将重要记忆提升到语义层"""
        if item.memory_type != MemoryType.SEMANTIC:
            layer = self.layers.get(MemoryType.SEMANTIC)
            if layer:
                item.memory_type = MemoryType.SEMANTIC
                await layer.store(item)
```

---

## 短期记忆（工作记忆）

### 内存缓存实现

```python
# openclaw/memory/working_memory.py
from collections import OrderedDict
from typing import Dict, Optional, Any
import json
import time

class WorkingMemoryCache:
    """工作记忆缓存（基于 LRU）"""
    
    def __init__(
        self,
        max_size: int = 100,
        ttl_seconds: int = 3600,
        max_tokens: int = 8000
    ):
        self.max_size = max_size
        self.ttl_seconds = ttl_seconds
        self.max_tokens = max_tokens
        self._cache: OrderedDict[str, Dict] = OrderedDict()
        self._timestamps: Dict[str, float] = {}
        self._token_counts: Dict[str, int] = {}
        self._current_tokens = 0
    
    def set(
        self,
        key: str,
        value: Any,
        ttl: int = None,
        token_count: int = None
    ):
        """设置缓存项"""
        
        # 清理过期项
        self._evict_expired()
        
        # 如果 key 已存在，先删除
        if key in self._cache:
            self._remove(key)
        
        # 计算 token 数
        if token_count is None:
            token_count = self._estimate_tokens(value)
        
        # 如果超出容量，清理
        while (
            len(self._cache) >= self.max_size or
            self._current_tokens + token_count > self.max_tokens
        ):
            self._evict_lru()
        
        # 添加新项
        self._cache[key] = value
        self._timestamps[key] = time.time()
        self._token_counts[key] = token_count
        self._current_tokens += token_count
        self._cache.move_to_end(key)
    
    def get(self, key: str, default: Any = None) -> Any:
        """获取缓存项"""
        
        if key not in self._cache:
            return default
        
        # 检查过期
        if self._is_expired(key):
            self._remove(key)
            return default
        
        # 更新访问时间
        self._cache.move_to_end(key)
        self._timestamps[key] = time.time()
        
        return self._cache[key]
    
    def _is_expired(self, key: str) -> bool:
        """检查是否过期"""
        if key not in self._timestamps:
            return True
        return time.time() - self._timestamps[key] > self.ttl_seconds
    
    def _evict_expired(self):
        """清理过期项"""
        expired_keys = [
            k for k in self._cache.keys()
            if self._is_expired(k)
        ]
        for key in expired_keys:
            self._remove(key)
    
    def _evict_lru(self):
        """清理最少使用的项"""
        if self._cache:
            oldest_key = next(iter(self._cache))
            self._remove(oldest_key)
    
    def _remove(self, key: str):
        """移除指定项"""
        if key in self._cache:
            self._current_tokens -= self._token_counts[key]
            del self._cache[key]
            del self._timestamps[key]
            del self._token_counts[key]
    
    def _estimate_tokens(self, value: Any) -> int:
        """估算 token 数量"""
        if isinstance(value, str):
            return len(value) // 4  # 粗略估算
        elif isinstance(value, dict):
            return self._estimate_tokens(json.dumps(value))
        return 10  # 默认值
    
    def get_context_string(self, max_tokens: int = None) -> str:
        """获取上下文字符串"""
        
        if max_tokens is None:
            max_tokens = self.max_tokens
        
        items = []
        current_tokens = 0
        
        for key, value in reversed(self._cache.items()):
            token_count = self._token_counts[key]
            
            if current_tokens + token_count > max_tokens:
                break
            
            if isinstance(value, str):
                items.append(value)
            elif isinstance(value, dict):
                items.append(f"{key}: {json.dumps(value)}")
            
            current_tokens += token_count
        
        return "\n".join(reversed(items))
    
    def clear(self):
        """清空缓存"""
        self._cache.clear()
        self._timestamps.clear()
        self._token_counts.clear()
        self._current_tokens = 0


class WorkingMemoryLayer(MemoryLayer):
    """工作记忆层实现"""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.cache = WorkingMemoryCache(
            max_size=config.get("max_size", 100),
            ttl_seconds=config.get("ttl_seconds", 3600),
            max_tokens=config.get("max_tokens", 8000)
        )
        self._current_session_id: Optional[str] = None
    
    def set_session(self, session_id: str):
        """设置当前会话 ID"""
        self._current_session_id = session_id
    
    async def store(self, item: MemoryItem) -> str:
        """存储到工作记忆"""
        key = f"{self._current_session_id}:{item.id}"
        self.cache.set(key, item.content, token_count=len(item.content) // 4)
        return key
    
    async def retrieve(self, query: MemoryQuery) -> MemoryResult:
        """检索工作记忆"""
        items = []
        
        if query.require_recency:
            # 按最近访问排序
            for key, content in reversed(self.cache._cache.items()):
                items.append(MemoryItem(
                    id=key,
                    content=content,
                    memory_type=MemoryType.WORKING,
                    importance=MemoryImportance.MEDIUM
                ))
        else:
            # 全部返回
            for key, content in self.cache._cache.items():
                items.append(MemoryItem(
                    id=key,
                    content=content,
                    memory_type=MemoryType.WORKING,
                    importance=MemoryImportance.MEDIUM
                ))
        
        return MemoryResult(
            items=items[:query.limit],
            scores=[1.0] * min(query.limit, len(items)),
            total_available=len(items)
        )
```

---

## 长期记忆（语义记忆）

### 向量数据库集成

```python
# openclaw/memory/semantic_memory.py
from typing import List, Optional, Dict, Any
import numpy as np

class VectorStoreConfig:
    """向量存储配置"""
    
    def __init__(
        self,
        provider: str = "chroma",
        embedding_model: str = "text-embedding-3-small",
        dimension: int = 1536,
        collection_name: str = "openclaw_semantic",
        persist_directory: str = "~/.openclaw/vector_db"
    ):
        self.provider = provider
        self.embedding_model = embedding_model
        self.dimension = dimension
        self.collection_name = collection_name
        self.persist_directory = persist_directory

class SemanticMemoryLayer(MemoryLayer):
    """语义记忆层实现"""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.vector_config = VectorStoreConfig(**config.get("vector", {}))
        self.client = self._init_vector_client()
        self.collection = self._get_or_create_collection()
    
    def _init_vector_client(self):
        """初始化向量数据库客户端"""
        
        provider = self.vector_config.provider
        
        if provider == "chroma":
            import chromadb
            from chromadb.config import Settings
            
            return chromadb.PersistentClient(
                path=self.vector_config.persist_directory
            )
        
        elif provider == "qdrant":
            from qdrant_client import QdrantClient
            
            return QdrantClient(
                url=config.get("qdrant_url", "http://localhost:6333"),
                api_key=config.get("qdrant_api_key")
            )
        
        elif provider == "milvus":
            from pymilvus import MilvusClient
            
            return MilvusClient(
                uri=f"{self.vector_config.persist_directory}/milvus.db"
            )
        
        else:
            raise ValueError(f"Unsupported vector provider: {provider}")
    
    def _get_or_create_collection(self):
        """获取或创建集合"""
        
        if self.vector_config.provider == "chroma":
            try:
                return self.client.get_collection(
                    self.vector_config.collection_name
                )
            except:
                return self.client.create_collection(
                    name=self.vector_config.collection_name,
                    metadata={"dimension": self.vector_config.dimension}
                )
        
        elif self.vector_config.provider == "qdrant":
            from qdrant_client.models import Distance, VectorParams
            
            self.client.recreate_collection(
                collection_name=self.vector_config.collection_name,
                vectors_config=VectorParams(
                    size=self.vector_config.dimension,
                    distance=Distance.COSINE
                )
            )
            return self.vector_config.collection_name
    
    async def store(self, item: MemoryItem) -> str:
        """存储到向量数据库"""
        
        metadata = {
            "memory_type": item.memory_type.value,
            "importance": item.importance.value,
            "created_at": item.created_at.isoformat(),
            "tags": ",".join(item.tags),
            "source": item.source
        }
        
        if self.vector_config.provider == "chroma":
            self.client.add(
                collection_name=self.vector_config.collection_name,
                ids=[item.id],
                embeddings=[item.embedding],
                documents=[item.content],
                metadatas=[metadata]
            )
        
        elif self.vector_config.provider == "qdrant":
            from qdrant_client.models import PointStruct
            
            self.client.upsert(
                collection_name=self.vector_config.collection_name,
                points=[
                    PointStruct(
                        id=item.id,
                        vector=item.embedding,
                        payload={
                            "content": item.content,
                            **metadata
                        }
                    )
                ]
            )
        
        return item.id
    
    async def retrieve(self, query: MemoryQuery) -> MemoryResult:
        """向量相似性检索"""
        
        query_embedding = await self._get_query_embedding(query.query)
        
        if self.vector_config.provider == "chroma":
            results = self.collection.query(
                query_embeddings=[query_embedding],
                n_results=query.limit,
                where=self._build_filter(query)
            )
            
            items = [
                MemoryItem(
                    id=results["ids"][0][i],
                    content=results["documents"][0][i],
                    memory_type=MemoryType.SEMANTIC,
                    importance=MemoryImportance(
                        results["metadatas"][0][i].get("importance", 3)
                    ),
                    metadata=results["metadatas"][0][i]
                )
                for i in range(len(results["ids"][0]))
            ]
            
            return MemoryResult(
                items=items,
                scores=results["distances"][0] if "distances" in results else [1.0] * len(items),
                total_available=results.get("total_count", len(items))
            )
        
        elif self.vector_config.provider == "qdrant":
            results = self.client.search(
                collection_name=self.vector_config.collection_name,
                query_vector=query_embedding,
                limit=query.limit,
                query_filter=self._build_qdrant_filter(query)
            )
            
            items = [
                MemoryItem(
                    id=result.id,
                    content=result.payload["content"],
                    memory_type=MemoryType.SEMANTIC,
                    importance=MemoryImportance(
                        result.payload.get("importance", 3)
                    ),
                    metadata=result.payload
                )
                for result in results
            ]
            
            return MemoryResult(
                items=items,
                scores=[result.score for result in results],
                total_available=len(results)
            )
    
    async def _get_query_embedding(self, query: str) -> List[float]:
        """获取查询的嵌入向量"""
        # 使用配置的 embedding model
        from openclaw.embedding import EmbeddingService
        service = EmbeddingService(self.vector_config.embedding_model)
        return await service.encode(query)
    
    def _build_filter(self, query: MemoryQuery) -> Dict:
        """构建 ChromaDB 过滤条件"""
        conditions = {}
        
        if query.min_importance:
            conditions["importance"] = {"$gte": query.min_importance.value}
        
        if query.tags:
            conditions["tags"] = {"$contains": query.tags[0]}
        
        if query.since:
            conditions["created_at"] = {"$gte": query.since.isoformat()}
        
        return conditions if conditions else None
```

---

## 记忆巩固与衰减

### 记忆生命周期管理

```python
# openclaw/memory/consolidation.py
from datetime import datetime, timedelta
import asyncio

class ConsolidationService:
    """记忆巩固服务"""
    
    def __init__(
        self,
        memory_manager: MemoryManager,
        config: Dict[str, Any] = None
    ):
        self.manager = memory_manager
        self.config = config or {}
        
        # 巩固阈值配置
        self.access_threshold = config.get("access_threshold", 3)
        self.importance_threshold = config.get("importance_threshold", 3)
        self.consolidation_interval = config.get("interval_hours", 24)
        
        # 最后巩固时间
        self.last_consolidation = datetime.now()
    
    async def should_consolidate(self, item: MemoryItem) -> bool:
        """判断是否应该巩固记忆"""
        
        # 关键记忆直接巩固
        if item.importance == MemoryImportance.CRITICAL:
            return True
        
        # 高频访问记忆优先巩固
        if item.access_count >= self.access_threshold:
            return True
        
        # 重要且被访问的记忆
        if (
            item.importance.value >= self.importance_threshold and
            item.access_count >= 1
        ):
            return True
        
        return False
    
    async def consolidate_item(self, item: MemoryItem):
        """巩固单个记忆到语义层"""
        
        # 创建新的语义记忆副本
        semantic_item = MemoryItem(
            id=f"sem_{item.id}",
            content=item.content,
            memory_type=MemoryType.SEMANTIC,
            importance=item.importance,
            embedding=item.embedding,
            metadata={
                **item.metadata,
                "consolidated_from": item.id,
                "original_type": item.memory_type.value,
                "access_count": item.access_count,
                "consolidated_at": datetime.now().isoformat()
            },
            tags=item.tags + ["consolidated"],
            source=item.source
        )
        
        # 存储到语义层
        semantic_layer = self.manager.layers.get(MemoryType.SEMANTIC)
        if semantic_layer:
            await semantic_layer.store(semantic_item)
        
        return semantic_item
    
    async def run_consolidation(self):
        """执行周期性巩固"""
        
        now = datetime.now()
        
        # 检查是否需要执行巩固
        if (
            now - self.last_consolidation
        ).total_seconds() < self.consolidation_interval * 3600:
            return
        
        self.last_consolidation = now
        
        # 从情景记忆层获取待巩固项
        episodic_layer = self.manager.layers.get(MemoryType.EPISODIC)
        if not episodic_layer:
            return
        
        results = await episodic_layer.retrieve(MemoryQuery(
            query="",
            limit=1000
        ))
        
        consolidated = 0
        for item in results.items:
            if await self.should_consolidate(item):
                await self.consolidate_item(item)
                consolidated += 1
        
        # 清理已巩固的旧记忆
        await self._prune_old_memories()
        
        return consolidated
    
    async def _prune_old_memories(self):
        """清理不再需要的记忆"""
        
        semantic_layer = self.manager.layers.get(MemoryType.SEMANTIC)
        if not semantic_layer:
            return
        
        # 获取所有语义记忆
        results = await semantic_layer.retrieve(MemoryQuery(
            query="",
            limit=10000
        ))
        
        # 计算衰减分数
        decayed_items = []
        for item in results.items:
            decay_score = self._calculate_decay(item)
            if decay_score < 0.1:  # 阈值
                decayed_items.append(item)
        
        # 删除低分记忆
        for item in decayed_items:
            await semantic_layer.delete(item.id)
    
    def _calculate_decay(self, item: MemoryItem) -> float:
        """计算记忆衰减分数"""
        
        # 基础分数
        base_score = item.importance.value / 5.0
        
        # 访问频率加成
        access_bonus = min(item.access_count * 0.1, 0.3)
        
        # 时间衰减
        days_since_access = (datetime.now() - item.last_accessed).days
        time_decay = 0.9 ** days_since_access
        
        # 综合分数
        return (base_score + access_bonus) * time_decay


class MemoryDecayScheduler:
    """记忆衰减调度器"""
    
    def __init__(self, consolidation_service: ConsolidationService):
        self.service = consolidation_service
        self._running = False
    
    async def start(self):
        """启动衰减调度器"""
        self._running = True
        while self._running:
            try:
                await self.service.run_consolidation()
                await asyncio.sleep(3600)  # 每小时检查一次
            except Exception as e:
                await asyncio.sleep(60)  # 出错时短暂等待
    
    def stop(self):
        """停止调度器"""
        self._running = False
```

---

## 跨会话持久化

### 会话存储实现

```python
# openclaw/memory/session_persistence.py
import json
import sqlite3
from pathlib import Path
from typing import List, Optional, Dict, Any
from datetime import datetime

class SessionPersistence:
    """会话持久化管理器"""
    
    def __init__(self, storage_path: str = "~/.openclaw/sessions"):
        self.storage_path = Path(storage_path).expanduser()
        self.storage_path.mkdir(parents=True, exist_ok=True)
        self.db_path = self.storage_path / "sessions.db"
        self._init_database()
    
    def _init_database(self):
        """初始化数据库"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # 会话表
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS sessions (
                session_id TEXT PRIMARY KEY,
                user_id TEXT NOT NULL,
                created_at TEXT NOT NULL,
                updated_at TEXT NOT NULL,
                metadata TEXT,
                is_active INTEGER DEFAULT 1
            )
        """)
        
        # 会话记忆表
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS session_memories (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                session_id TEXT NOT NULL,
                content TEXT NOT NULL,
                memory_type TEXT NOT NULL,
                importance INTEGER DEFAULT 3,
                metadata TEXT,
                created_at TEXT NOT NULL,
                FOREIGN KEY (session_id) REFERENCES sessions(session_id)
            )
        """)
        
        # 创建索引
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_session_memories_session_id
            ON session_memories(session_id)
        """)
        
        cursor.execute("""
            CREATE INDEX IF NOT EXISTS idx_sessions_user_id
            ON sessions(user_id)
        """)
        
        conn.commit()
        conn.close()
    
    def create_session(
        self,
        session_id: str,
        user_id: str,
        metadata: Dict[str, Any] = None
    ) -> bool:
        """创建新会话"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        now = datetime.now().isoformat()
        
        try:
            cursor.execute("""
                INSERT INTO sessions (session_id, user_id, created_at, updated_at, metadata)
                VALUES (?, ?, ?, ?, ?)
            """, (session_id, user_id, now, now, json.dumps(metadata or {})))
            
            conn.commit()
            return True
        except sqlite3.IntegrityError:
            return False
        finally:
            conn.close()
    
    def get_session(self, session_id: str) -> Optional[Dict]:
        """获取会话信息"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute("""
            SELECT session_id, user_id, created_at, updated_at, metadata
            FROM sessions WHERE session_id = ?
        """, (session_id,))
        
        row = cursor.fetchone()
        conn.close()
        
        if row:
            return {
                "session_id": row[0],
                "user_id": row[1],
                "created_at": row[2],
                "updated_at": row[3],
                "metadata": json.loads(row[4])
            }
        return None
    
    def save_session_memory(
        self,
        session_id: str,
        content: str,
        memory_type: str,
        importance: int = 3,
        metadata: Dict = None
    ) -> int:
        """保存会话记忆"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        now = datetime.now().isoformat()
        
        cursor.execute("""
            INSERT INTO session_memories
            (session_id, content, memory_type, importance, metadata, created_at)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (session_id, content, memory_type, importance, json.dumps(metadata or {}), now))
        
        memory_id = cursor.lastrowid
        
        # 更新会话时间
        cursor.execute("""
            UPDATE sessions SET updated_at = ? WHERE session_id = ?
        """, (now, session_id))
        
        conn.commit()
        conn.close()
        
        return memory_id
    
    def get_session_memories(
        self,
        session_id: str,
        limit: int = 100,
        memory_type: str = None
    ) -> List[Dict]:
        """获取会话的所有记忆"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        query = """
            SELECT id, content, memory_type, importance, metadata, created_at
            FROM session_memories
            WHERE session_id = ?
        """
        params = [session_id]
        
        if memory_type:
            query += " AND memory_type = ?"
            params.append(memory_type)
        
        query += " ORDER BY created_at DESC LIMIT ?"
        params.append(limit)
        
        cursor.execute(query, params)
        
        results = [
            {
                "id": row[0],
                "content": row[1],
                "memory_type": row[2],
                "importance": row[3],
                "metadata": json.loads(row[4]),
                "created_at": row[5]
            }
            for row in cursor.fetchall()
        ]
        
        conn.close()
        return results
    
    def get_user_sessions(
        self,
        user_id: str,
        active_only: bool = True,
        limit: int = 10
    ) -> List[Dict]:
        """获取用户的所有会话"""
        
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        query = """
            SELECT session_id, created_at, updated_at, metadata
            FROM sessions
            WHERE user_id = ?
        """
        
        if active_only:
            query += " AND is_active = 1"
        
        query += " ORDER BY updated_at DESC LIMIT ?"
        
        cursor.execute(query, (user_id, limit))
        
        results = [
            {
                "session_id": row[0],
                "created_at": row[1],
                "updated_at": row[2],
                "metadata": json.loads(row[3])
            }
            for row in cursor.fetchall()
        ]
        
        conn.close()
        return results
    
    def get_context_for_session(
        self,
        session_id: str,
        max_memories: int = 50
    ) -> str:
        """获取会话的上下文字符串"""
        
        memories = self.get_session_memories(session_id, limit=max_memories)
        
        if not memories:
            return ""
        
        lines = ["## 会话历史记忆\n"]
        
        for memory in memories:
            lines.append(
                f"- **{memory['memory_type']}**: {memory['content']}"
            )
        
        return "\n".join(lines)
```

---

## 智能检索策略

### 混合检索实现

```python
# openclaw/memory/retrieval.py
from typing import List, Tuple

class RetrievalService:
    """智能检索服务"""
    
    def __init__(self, memory_manager: MemoryManager):
        self.manager = memory_manager
    
    async def retrieve_with_strategy(
        self,
        query: str,
        strategy: str = "hybrid",
        **kwargs
    ) -> List[MemoryItem]:
        """使用指定策略检索记忆"""
        
        strategies = {
            "dense": self._dense_retrieval,
            "sparse": self._sparse_retrieval,
            "hybrid": self._hybrid_retrieval,
            "rerank": self._rerank_retrieval
        }
        
        strategy_func = strategies.get(strategy, self._dense_retrieval)
        return await strategy_func(query, **kwargs)
    
    async def _dense_retrieval(
        self,
        query: str,
        limit: int = 10
    ) -> List[MemoryItem]:
        """密集向量检索"""
        return await self.manager.search_memory(
            query=query,
            limit=limit
        )
    
    async def _sparse_retrieval(
        self,
        query: str,
        limit: int = 10
    ) -> List[MemoryItem]:
        """稀疏关键词检索（BM25）"""
        # 使用关键词匹配
        keywords = self._extract_keywords(query)
        
        results = []
        for memory_type in [MemoryType.EPISODIC, MemoryType.SEMANTIC]:
            layer = self.manager.layers.get(memory_type)
            if layer:
                items = await layer.retrieve(MemoryQuery(
                    query="",
                    tags=keywords,
                    limit=limit * 2
                ))
                results.extend(items.items)
        
        # BM25 排序
        scored = [(item, self._bm25_score(item.content, keywords)) 
                   for item in results]
        scored.sort(key=lambda x: x[1], reverse=True)
        
        return [item for item, score in scored[:limit]]
    
    async def _hybrid_retrieval(
        self,
        query: str,
        dense_weight: float = 0.7,
        limit: int = 10
    ) -> List[MemoryItem]:
        """混合检索（向量 + BM25）"""
        
        # 并行执行两种检索
        dense_task = self._dense_retrieval(query, limit * 2)
        sparse_task = self._sparse_retrieval(query, limit * 2)
        
        dense_results, sparse_results = await asyncio.gather(
            dense_task, sparse_task
        )
        
        # 合并结果
        scores: Dict[str, Tuple[MemoryItem, float]] = {}
        
        for item in dense_results:
            scores[item.id] = (item, dense_weight)
        
        for item in sparse_results:
            if item.id in scores:
                item, score = scores[item.id]
                scores[item.id] = (item, score + (1 - dense_weight))
            else:
                scores[item.id] = (item, 1 - dense_weight)
        
        # 排序返回
        sorted_items = sorted(
            scores.values(),
            key=lambda x: x[1],
            reverse=True
        )
        
        return [item for item, score in sorted_items[:limit]]
    
    async def _rerank_retrieval(
        self,
        query: str,
        initial_limit: int = 50,
        final_limit: int = 10
    ) -> List[MemoryItem]:
        """重排检索（两阶段）"""
        
        # 第一阶段：快速检索
        candidates = await self._dense_retrieval(query, initial_limit)
        
        # 第二阶段：使用交叉编码器重排
        reranked = await self._cross_encoder_rerank(query, candidates)
        
        return reranked[:final_limit]
    
    def _extract_keywords(self, query: str) -> List[str]:
        """提取关键词"""
        # 简单实现，可替换为更复杂的 NLP 方法
        words = query.lower().split()
        stopwords = {"the", "a", "an", "is", "are", "was", "were", "to", "of", "in"}
        return [w for w in words if w not in stopwords and len(w) > 2]
    
    def _bm25_score(self, document: str, keywords: List[str]) -> float:
        """BM25 评分（简化实现）"""
        
        doc_lower = document.lower()
        words = doc_lower.split()
        doc_length = len(words)
        avg_length = doc_length  # 简化处理
        
        score = 0.0
        k1 = 1.5
        b = 0.75
        
        for keyword in keywords:
            freq = words.count(keyword)
            if freq > 0:
                tf = (freq * (k1 + 1)) / (freq + k1 * (1 - b + b * doc_length / avg_length))
                score += tf
        
        return score
    
    async def _cross_encoder_rerank(
        self,
        query: str,
        candidates: List[MemoryItem]
    ) -> List[MemoryItem]:
        """使用交叉编码器重排"""
        
        from openclaw.embedding import CrossEncoderService
        
        encoder = CrossEncoderService()
        
        pairs = [(query, item.content) for item in candidates]
        scores = await encoder.predict(pairs)
        
        paired = list(zip(candidates, scores))
        paired.sort(key=lambda x: x[1], reverse=True)
        
        return [item for item, score in paired]
```

> [!tip] 检索优化建议
> - 日常查询使用「hybrid」策略，平衡速度和精度
> - 需要高精度的场景使用「rerank」策略
> - 实时对话使用「dense」策略，减少延迟

---

## 与 Agent 的集成

```python
# openclaw/memory/agent_integration.py

class MemoryAwareAgent:
    """支持记忆的 Agent"""
    
    def __init__(self, memory_manager: MemoryManager, agent: "Agent"):
        self.memory = memory_manager
        self.agent = agent
    
    async def process(self, input_text: str, user_id: str = None) -> str:
        """处理输入，包含记忆增强"""
        
        context_parts = []
        
        # 1. 检索相关长期记忆
        relevant_memories = await self.memory.search_memory(
            query=input_text,
            limit=5
        )
        
        if relevant_memories:
            context_parts.append("## 相关记忆\n")
            for mem in relevant_memories:
                context_parts.append(f"- {mem.content}")
        
        # 2. 获取当前会话记忆
        if user_id:
            session_memories = self.memory.session_persistence.get_session_memories(
                f"{user_id}_current",
                limit=10
            )
            
            if session_memories:
                context_parts.append("\n## 当前会话\n")
                for mem in session_memories[-5:]:
                    context_parts.append(f"- {mem['content']}")
        
        # 3. 获取工作记忆
        working_context = self.memory.layers.get(MemoryType.WORKING)
        if working_context:
            context_parts.append(f"\n{working_context.cache.get_context_string()}")
        
        # 4. 构建完整上下文
        full_context = "\n".join(context_parts)
        
        # 5. 调用 Agent
        response = await self.agent.process(
            input_text,
            context=full_context
        )
        
        # 6. 保存重要交互到记忆
        if self._should_remember(response):
            await self.memory.add_memory(
                content=f"User: {input_text}\nAgent: {response}",
                memory_type=MemoryType.EPISODIC,
                importance=self._assess_importance(input_text, response),
                source=f"conversation_{user_id or 'anonymous'}"
            )
        
        return response
    
    def _should_remember(self, response: str) -> bool:
        """判断是否应该记住这次交互"""
        important_patterns = ["记住", "will remember", "important", "我会记住"]
        return any(p in response.lower() for p in important_patterns)
    
    def _assess_importance(self, input_text: str, response: str) -> MemoryImportance:
        """评估交互重要性"""
        if "非常重要" in input_text or "critical" in input_text.lower():
            return MemoryImportance.CRITICAL
        elif len(response) > 500:
            return MemoryImportance.HIGH
        return MemoryImportance.MEDIUM
```

> [!summary] 关键要点
> 1. **分层设计**：工作记忆 → 情景记忆 → 语义记忆，模拟人类记忆结构
> 2. **向量检索**：使用嵌入模型实现语义相似性搜索
> 3. **记忆巩固**：自动将重要记忆提升到长期存储
> 4. **智能衰减**：根据访问频率和重要性动态管理记忆
> 5. **跨会话持久化**：通过 SQLite 实现会话间状态保持

---

## 相关文档

- [[../多平台集成/OpenClaw多平台集成]] - 记忆在不同平台的同步
- [[../插件开发/OpenClaw插件开发]] - 记忆处理插件开发
- [[../高级用法/OpenClaw高级用法]] - 多Agent记忆共享
- [[OpenClaw概览]] - 系统整体架构
