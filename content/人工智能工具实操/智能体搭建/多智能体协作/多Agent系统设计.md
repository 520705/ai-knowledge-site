---
title: 多Agent系统设计：从原理到实践
date: 2026-04-24
tags:
  - 多智能体
  - Agent协作
  - 系统架构
  - 通信协议
  - 知识共享
  - 任务分解
  - 分布式AI
categories:
  - 智能体搭建
  - 多智能体协作
description: 深入探讨多智能体系统的架构设计与实现，包括通信协议定义、协作模式选择、共享知识库构建、冲突解决机制，通过详细的架构图、代码示例和实战案例，帮助读者构建高效、可靠的多Agent系统。
---

# 多Agent系统设计：从原理到实践

> [!NOTE] 这篇指南讲什么
> 当单个Agent能力有限时，我们需要多个Agent协作。这篇指南从理论到实现，详细讲解多Agent系统的设计模式、通信机制、协作流程和实战案例。

## 为什么需要多Agent系统？

### 单Agent的局限性

单Agent系统面临的核心挑战：

| 问题 | 说明 | 多Agent解决方案 |
|------|------|-----------------|
| 能力边界 | 单模型能力有限 | 分工给不同专长Agent |
| 上下文限制 | Token窗口有限 | 各Agent维护独立上下文 |
| 单点故障 | 一个Agent挂了全挂 | 冗余+容错设计 |
| 扩展性差 | 新能力需重训 | 添加新Agent即可 |
| 维护困难 | 代码膨胀 | 模块化、职责单一 |

### 多Agent的优势

```
┌─────────────────────────────────────────────────────────────┐
│                    多Agent协作示意图                              │
│                                                            │
│                    用户请求                                       │
│                       ↓                                       │
│              ┌───────────────┐                               │
│              │  协调Agent   │                               │
│              │ (Coordinator) │                               │
│              └───────┬───────┘                               │
│                      ↓                                        │
│        ┌──────────┼──────────┐                              │
│        ↓          ↓          ↓                              │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐                        │
│  │ 搜索Agent │ │ 分析Agent │ │ 写作Agent │                        │
│  └─────┬─────┘ └─────┬─────┘ └─────┬─────┘                        │
│        └──────────┼──────────┘                              │
│                   ↓                                         │
│              ┌───────────────┐                               │
│              │  协调Agent   │                               │
│              └───────┬───────┘                               │
│                      ↓                                        │
│                   用户结果                                      │
└─────────────────────────────────────────────────────────────┘
```

## Agent角色分类

### 角色类型

| 角色 | 职责 | 特点 |
|------|------|------|
| **协调者（Coordinator）** | 分解任务、分配工作、整合结果 | 全局视角、决策能力强 |
| **专家Agent（Specialist）** | 专注特定领域 | 深度知识、执行高效 |
| **执行者（Executor）** | 负责具体操作 | 执行力强、简单任务 |
| **审核者（Reviewer）** | 检查结果、质量把控 | 严格、挑剔 |

### 典型Agent配置

| 场景 | Agent配置 | 协作模式 |
|------|----------|----------|
| 智能客服 | 接待+知识库+订单+投诉 | 分层协作 |
| 代码助手 | 需求分析+代码生成+测试+审查 | 流水线 |
| 研究助手 | 搜索+阅读+整理+写作 | 串行协作 |
| 自动化办公 | 日程+邮件+文档+审批 | 事件驱动 |

## 通信协议设计

### 消息格式定义

```python
from typing import Optional, Dict, Any, List
from dataclasses import dataclass, field
from datetime import datetime
from enum import Enum
import uuid
import json

class MessageType(Enum):
    """消息类型枚举"""
    REQUEST = "request"           # 请求消息
    RESPONSE = "response"         # 响应消息
    EVENT = "event"               # 事件通知
    BROADCAST = "broadcast"       # 广播消息
    HEARTBEAT = "heartbeat"       # 心跳检测

class MessagePriority(Enum):
    """消息优先级"""
    CRITICAL = 1   # 关键任务
    HIGH = 3        # 高优先级
    NORMAL = 5      # 普通
    LOW = 7         # 低优先级
    BACKGROUND = 9   # 后台任务

@dataclass
class AgentMessage:
    """Agent通信消息"""
    message_id: str = field(default_factory=lambda: str(uuid.uuid4()))
    timestamp: str = field(default_factory=lambda: datetime.now().isoformat())
    
    # 发送方和接收方
    sender: Dict[str, Any] = field(default_factory=dict)
    receiver: Dict[str, Any] = field(default_factory=dict)
    
    # 消息类型和内容
    message_type: MessageType = MessageType.REQUEST
    content: Dict[str, Any] = field(default_factory=dict)
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def __post_init__(self):
        if 'priority' not in self.metadata:
            self.metadata['priority'] = MessagePriority.NORMAL.value
        if 'ttl' not in self.metadata:
            self.metadata['ttl'] = 300  # 默认5分钟生存时间
    
    def to_json(self) -> str:
        """序列化为JSON"""
        return json.dumps({
            'message_id': self.message_id,
            'timestamp': self.timestamp,
            'sender': self.sender,
            'receiver': self.receiver,
            'message_type': self.message_type.value,
            'content': self.content,
            'metadata': self.metadata
        }, ensure_ascii=False)
    
    @classmethod
    def from_json(cls, json_str: str) -> 'AgentMessage':
        """从JSON反序列化"""
        data = json.loads(json_str)
        return cls(
            message_id=data['message_id'],
            timestamp=data['timestamp'],
            sender=data['sender'],
            receiver=data['receiver'],
            message_type=MessageType(data['message_type']),
            content=data['content'],
            metadata=data['metadata']
        )
```

### 消息协议格式

```yaml
# 消息协议格式
MessageProtocol:
  message_id: "uuid-v4"           # 全局唯一消息ID
  timestamp: "ISO-8601"            # 时间戳
  sender:                          # 发送方
    agent_id: "string"
    agent_type: "string"
    capabilities: ["capability_list"]
  
  receiver:                        # 接收方
    agent_id: "string"             # 空表示广播
    agent_type: "string"           # 类型匹配
  
  message_type:                    # 消息类型
    enum: ["request", "response", "event", "broadcast"]
  
  content:                         # 消息内容
    action: "string"              # 操作名称
    params: {}                     # 参数
    context: {}                     # 上下文
  
  metadata:                        # 元信息
    priority: 1-10                # 优先级
    ttl: 300                       # 生存时间(秒)
    retry_count: 0                # 重试次数
```

### 通信中间件实现

```python
import asyncio
from typing import Callable, Dict, Set, List, Optional
from collections import defaultdict
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

class MessageBroker:
    """消息代理 - 实现Agent间通信"""
    
    def __init__(self, history_limit: int = 1000):
        self.subscribers: Dict[str, Set[Callable]] = defaultdict(set)
        self.message_queue: asyncio.Queue = asyncio.Queue()
        self.message_history: List[AgentMessage] = []
        self.max_history = history_limit
        self._running = False
    
    async def start(self):
        """启动消息代理"""
        self._running = True
        self._processor = asyncio.create_task(self._process_messages())
    
    async def stop(self):
        """停止消息代理"""
        self._running = False
        if hasattr(self, '_processor'):
            self._processor.cancel()
    
    async def _process_messages(self):
        """异步处理消息队列"""
        while self._running:
            try:
                message = await asyncio.wait_for(
                    self.message_queue.get(),
                    timeout=1.0
                )
                await self._route_message(message)
            except asyncio.TimeoutError:
                continue
            except Exception as e:
                logger.error(f"Message processing error: {e}")
    
    async def publish(self, message: AgentMessage) -> None:
        """发布消息"""
        # 记录历史
        self.message_history.append(message)
        if len(self.message_history) > self.max_history:
            self.message_history.pop(0)
        
        # 加入队列
        await self.message_queue.put(message)
    
    async def subscribe(
        self, 
        agent_id: str, 
        callback: Callable[[AgentMessage], None]
    ) -> None:
        """订阅消息"""
        self.subscribers[agent_id].add(callback)
    
    async def unsubscribe(
        self,
        agent_id: str,
        callback: Callable[[AgentMessage], None]
    ) -> None:
        """取消订阅"""
        self.subscribers[agent_id].discard(callback)
    
    async def _route_message(self, message: AgentMessage) -> None:
        """路由消息"""
        # 点对点消息
        if message.receiver.get('agent_id'):
            await self._deliver_to_agent(message)
        # 类型广播
        elif message.receiver.get('agent_type'):
            await self._broadcast_to_type(message)
        # 全局广播
        else:
            await self._broadcast_all(message)
    
    async def _deliver_to_agent(self, message: AgentMessage):
        """发送给指定Agent"""
        agent_id = message.receiver['agent_id']
        callbacks = self.subscribers.get(agent_id, set())
        
        for callback in callbacks:
            try:
                await callback(message)
            except Exception as e:
                logger.error(f"Callback error for {agent_id}: {e}")
    
    async def _broadcast_to_type(self, message: AgentMessage):
        """按类型广播"""
        agent_type = message.receiver.get('agent_type')
        # 简化实现：广播给所有订阅者
        for agent_id, callbacks in self.subscribers.items():
            for callback in callbacks:
                try:
                    await callback(message)
                except Exception as e:
                    logger.error(f"Broadcast error: {e}")
    
    async def _broadcast_all(self, message: AgentMessage):
        """全局广播"""
        for agent_id, callbacks in self.subscribers.items():
            for callback in callbacks:
                try:
                    await callback(message)
                except Exception as e:
                    logger.error(f"Broadcast error: {e}")
    
    async def request(
        self,
        sender_id: str,
        receiver_id: str,
        action: str,
        params: Dict[str, Any],
        timeout: float = 30.0
    ) -> AgentMessage:
        """发送请求并等待响应"""
        request_msg = AgentMessage(
            sender={'agent_id': sender_id},
            receiver={'agent_id': receiver_id},
            message_type=MessageType.REQUEST,
            content={'action': action, 'params': params}
        )
        
        # 创建Future等待响应
        response_future: asyncio.Future = asyncio.Future()
        
        async def response_handler(msg: AgentMessage):
            if (msg.message_type == MessageType.RESPONSE and 
                msg.content.get('request_id') == request_msg.message_id):
                response_future.set_result(msg)
        
        # 订阅响应
        await self.subscribe(sender_id, response_handler)
        
        try:
            # 发送请求
            await self.publish(request_msg)
            
            # 等待响应
            return await asyncio.wait_for(response_future, timeout)
        except asyncio.TimeoutError:
            raise TimeoutError(f"Request to {receiver_id} timed out")
        finally:
            await self.unsubscribe(sender_id, response_handler)
```

## 协作模式详解

### 模式一：串行协作（Sequential）

最简单的协作模式，按顺序执行：

```
graph LR
    A[用户] --> B[Agent1]
    B --> C[Agent2]
    C --> D[Agent3]
    D --> E[输出]
    
    B -.->|传递上下文| C
    C -.->|传递上下文| D
```

**适用场景：** 任务有严格先后依赖

```python
class SequentialCollaboration:
    """串行协作模式"""
    
    async def execute(
        self, 
        agents: List['BaseAgent'],
        initial_input: Any,
        context: Dict = None
    ) -> Dict[str, Any]:
        """顺序执行所有Agent"""
        context = context or {}
        context['input'] = initial_input
        
        results = {}
        
        for agent in agents:
            logger.info(f"Executing {agent.name}")
            
            # 执行Agent
            result = await agent.process(context)
            results[agent.name] = result
            
            # 检查是否提前结束
            if result.get('should_stop'):
                logger.info(f"Early stop at {agent.name}")
                break
            
            # 更新上下文
            context[agent.name] = result
            context['previous_output'] = result.get('output')
        
        return {
            'results': results,
            'final_output': context.get('previous_output'),
            'all_context': context
        }


# 使用示例
async def document_generation_pipeline():
    """文档生成流水线"""
    agents = [
        ResearchAgent(name="研究", model="gpt-4o"),
        OutlineAgent(name="大纲", model="gpt-4o"),
        DraftAgent(name="初稿", model="gpt-4o"),
        ReviewAgent(name="审核", model="gpt-4o"),
        FormatAgent(name="格式", model="gpt-4o-mini")
    ]
    
    pipeline = SequentialCollaboration()
    result = await pipeline.execute(
        agents=agents,
        initial_input="一篇关于AI大模型发展趋势的研究报告"
    )
    
    return result['final_output']
```

### 模式二：并行协作（Parallel）

多个Agent同时工作，提高效率：

```
graph TD
    A[用户请求] --> B[分发器]
    B --> C[Agent1]
    B --> D[Agent2]
    B --> E[Agent3]
    C --> F[聚合器]
    D --> F
    E --> F
    F --> G[输出]
```

**适用场景：** 任务可分解为独立子任务

```python
import asyncio
from typing import List, Callable, Any, Dict, Optional
from concurrent.futures import ThreadPoolExecutor

class ParallelCollaboration:
    """并行协作模式"""
    
    def __init__(self, max_concurrency: int = 5):
        self.max_concurrency = max_concurrency
        self.semaphore = asyncio.Semaphore(max_concurrency)
    
    async def execute(
        self,
        agents: List['BaseAgent'],
        initial_input: Any,
        aggregator: Optional[Callable] = None
    ) -> Dict[str, Any]:
        """并行执行Agent"""
        
        async def execute_with_semaphore(agent: 'BaseAgent') -> Dict:
            async with self.semaphore:
                try:
                    result = await agent.process(initial_input)
                    return {'success': True, 'agent': agent.name, 'result': result}
                except Exception as e:
                    logger.error(f"{agent.name} failed: {e}")
                    return {'success': False, 'agent': agent.name, 'error': str(e)}
        
        # 创建所有任务
        tasks = [execute_with_semaphore(agent) for agent in agents]
        
        # 并行执行
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # 处理异常
        successful = []
        failed = []
        
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                failed.append({'agent': agents[i].name, 'error': str(result)})
            elif not result.get('success'):
                failed.append({'agent': result['agent'], 'error': result.get('error')})
            else:
                successful.append(result)
        
        # 聚合结果
        aggregated = None
        if aggregator and successful:
            aggregated = aggregator([r['result'] for r in successful])
        
        return {
            'successful': successful,
            'failed': failed,
            'aggregated': aggregated,
            'success_rate': len(successful) / len(agents) if agents else 0
        }


# 使用示例
async def multi_perspective_analysis():
    """多视角分析"""
    agents = [
        TechnicalAnalysisAgent(name="技术", model="gpt-4o"),
        MarketAnalysisAgent(name="市场", model="gpt-4o"),
        RiskAnalysisAgent(name="风险", model="gpt-4o"),
        CompetitorAgent(name="竞品", model="gpt-4o")
    ]
    
    async def aggregate(results: List[Dict]) -> Dict:
        return {
            'technical': results[0],
            'market': results[1],
            'risk': results[2],
            'competitor': results[3],
            'summary': await generate_summary(results)
        }
    
    parallel = ParallelCollaboration()
    result = await parallel.execute(agents, topic, aggregate)
    
    return result
```

### 模式三：分层协作（Hierarchical）

Supervisor模式，Agent管理子Agent：

```
graph TD
    A[用户] --> B[Supervisor]
    B --> C[Worker1]
    B --> D[Worker2]
    B --> E[Worker3]
    
    C --> B
    D --> B
    E --> B
    
    B -->|路由| F[工具调用]
    B -->|查询| G[知识库]
```

**适用场景：** 需要智能路由和任务分配

```python
class HierarchicalCollaboration:
    """分层协作模式"""
    
    def __init__(self):
        self.supervisor: Optional['SupervisorAgent'] = None
        self.workers: Dict[str, 'BaseAgent'] = {}
        self.router = TaskRouter()
    
    def add_worker(self, worker: 'BaseAgent'):
        """添加Worker"""
        self.workers[worker.name] = worker
        self.router.register_worker(worker)
    
    def set_supervisor(self, supervisor: 'SupervisorAgent'):
        """设置Supervisor"""
        self.supervisor = supervisor
        supervisor.set_workers(self.workers)
        supervisor.set_router(self.router)
    
    async def execute(self, user_input: str) -> Dict[str, Any]:
        """执行分层协作"""
        if not self.supervisor:
            raise ValueError("Supervisor not set")
        
        # Supervisor分析并规划
        plan = await self.supervisor.plan(user_input)
        
        if plan['type'] == 'direct':
            # 直接回答
            return await self.supervisor.execute_direct(plan)
        
        elif plan['type'] == 'multi_step':
            # 多步骤任务
            results = []
            
            for step in plan['steps']:
                # 路由到合适的Worker
                worker = self.router.route(step)
                
                if worker:
                    result = await worker.process(step)
                    results.append(result)
                    
                    # Supervisor审核
                    if not await self.supervisor.verify(result):
                        # 重新执行
                        result = await worker.retry(step)
                        results[-1] = result
            
            return await self.supervisor.synthesize(results)
        
        return {'error': 'Unknown plan type'}


class SupervisorAgent:
    """监督Agent"""
    
    def __init__(self, name: str, model: str):
        self.name = name
        self.model = model
        self.workers = {}
        self.router = None
        self.llm = LLMClient(model)
    
    def set_workers(self, workers: Dict[str, 'BaseAgent']):
        self.workers = workers
    
    def set_router(self, router: 'TaskRouter'):
        self.router = router
    
    async def plan(self, user_input: str) -> Dict[str, Any]:
        """任务规划"""
        # 调用LLM进行规划
        prompt = f"""
用户请求：{user_input}

可用Worker：
{self._format_workers()}

请分析请求，判断：
1. 是简单问答还是复杂任务
2. 需要哪些Worker协作
3. 协作顺序是什么

返回JSON格式。
"""
        
        response = await self.llm.chat([{"role": "user", "content": prompt}])
        
        try:
            return json.loads(response)
        except:
            return {'type': 'direct', 'reason': 'parsing_failed'}
    
    async def verify(self, result: Dict) -> bool:
        """验证结果"""
        confidence = result.get('confidence', 1.0)
        return confidence >= 0.7
    
    def _format_workers(self) -> str:
        return "\n".join([
            f"- {name}: {worker.capabilities}"
            for name, worker in self.workers.items()
        ])


class TaskRouter:
    """任务路由器"""
    
    def __init__(self):
        self.workers: Dict[str, 'BaseAgent'] = {}
        self.capability_index: Dict[str, List[str]] = defaultdict(list)
    
    def register_worker(self, worker: 'BaseAgent'):
        self.workers[worker.name] = worker
        for cap in worker.capabilities:
            self.capability_index[cap].append(worker.name)
    
    def route(self, task: Dict) -> Optional['BaseAgent']:
        """根据任务路由到合适的Worker"""
        required_cap = task.get('required_capability')
        
        if not required_cap:
            return None
        
        suitable_workers = self.capability_index.get(required_cap, [])
        
        if not suitable_workers:
            return None
        
        return self.workers[suitable_workers[0]]
```

### 模式四：网状协作（Mesh）

Agent全连接，互相协作：

```
graph Full
    A[Agent1] <--> B[Agent2]
    A <--> C[Agent3]
    A <--> D[Agent4]
    B <--> C
    B <--> D
    C <--> D
```

**适用场景：** 需要多方协商、共识决策

```python
class MeshCollaboration:
    """网状协作模式"""
    
    def __init__(self, broker: MessageBroker):
        self.broker = broker
        self.agents: Dict[str, 'BaseAgent'] = {}
        self.connections: Dict[str, Set[str]] = defaultdict(set)
    
    def add_agent(self, agent: 'BaseAgent', connections: List[str] = None):
        """添加Agent及其连接"""
        self.agents[agent.name] = agent
        
        if connections:
            for conn in connections:
                self.connections[agent.name].add(conn)
                self.connections[conn].add(agent.name)
    
    async def execute(self, initiating_agent: str, task: Any) -> Dict:
        """网状协作执行"""
        # 创建共识收集任务
        consensus_tasks = []
        
        for agent_name in self.connections[initiating_agent]:
            agent = self.agents[agent_name]
            consensus_tasks.append(
                self._get_opinion(agent, task)
            )
        
        # 并行收集意见
        opinions = await asyncio.gather(*consensus_tasks, return_exceptions=True)
        
        # 共识决策
        initiator = self.agents[initiating_agent]
        return await initiator.reach_consensus(opinions)
    
    async def _get_opinion(self, agent: 'BaseAgent', task: Any) -> Dict:
        """获取Agent的意见"""
        try:
            result = await agent.process(task)
            return {
                'agent': agent.name,
                'opinion': result.get('opinion'),
                'confidence': result.get('confidence', 0.5)
            }
        except Exception as e:
            return {
                'agent': agent.name,
                'error': str(e),
                'confidence': 0
            }
```

## 共享知识库设计

### 知识库架构

```python
from typing import List, Optional, Dict, Any, Set
import hashlib
from datetime import datetime

class SharedKnowledgeBase:
    """共享知识库"""
    
    def __init__(
        self,
        vector_store: 'VectorStore',
        graph_store: 'GraphStore' = None,
        metadata_store: 'MetadataStore' = None
    ):
        self.vector_store = vector_store
        self.graph_store = graph_store
        self.metadata_store = metadata_store or InMemoryMetadataStore()
        self.version_control = VersionControl()
        self.locks: Dict[str, asyncio.Lock] = {}
    
    async def write(
        self,
        agent_id: str,
        content: str,
        knowledge_type: str,
        metadata: Dict[str, Any]
    ) -> str:
        """写入知识"""
        # 生成知识ID
        knowledge_id = self._generate_id(
            agent_id, content, datetime.now().isoformat()
        )
        
        # 检查写入权限
        if not await self._check_write_permission(agent_id, knowledge_id):
            raise PermissionError(f"Agent {agent_id} lacks write permission")
        
        # 并发控制
        async with self._get_lock(knowledge_id):
            # 向量化
            embedding = await self._embed(content)
            
            # 事务写入
            await self.vector_store.add(knowledge_id, embedding)
            
            if self.graph_store:
                await self.graph_store.add_triples(
                    self._extract_entities(content)
                )
            
            await self.metadata_store.save(knowledge_id, {
                'agent_id': agent_id,
                'type': knowledge_type,
                'metadata': metadata,
                'created_at': datetime.now().isoformat(),
                'version': 1
            })
        
        return knowledge_id
    
    async def query(
        self,
        agent_id: str,
        query: str,
        top_k: int = 10,
        filters: Dict = None
    ) -> List[Dict]:
        """查询知识"""
        # 权限检查
        if not await self._check_read_permission(agent_id):
            raise PermissionError(f"Agent {agent_id} lacks read permission")
        
        # 向量检索
        query_embedding = await self._embed(query)
        vector_results = await self.vector_store.search(
            query_embedding, top_k
        )
        
        # 图谱扩展
        related = []
        if self.graph_store and vector_results:
            related = await self.graph_store.expand(
                [r['id'] for r in vector_results],
                depth=2
            )
        
        # 结果融合
        fused = self._rerank(query, vector_results + related, agent_id)
        
        # 过滤
        if filters:
            fused = self._apply_filters(fused, filters)
        
        return fused
    
    async def subscribe(self, agent_id: str) -> 'AsyncIterator':
        """订阅知识更新"""
        pubsub = await self._create_pubsub(agent_id)
        async for update in pubsub.listen():
            yield update
    
    def _generate_id(self, *parts) -> str:
        """生成唯一ID"""
        content = "|".join(str(p) for p in parts)
        return hashlib.sha256(content.encode()).hexdigest()[:16]
    
    async def _embed(self, text: str) -> List[float]:
        """文本向量化"""
        # 实际实现调用embedding服务
        pass
    
    def _extract_entities(self, content: str) -> List:
        """提取实体"""
        # 实际实现调用NER服务
        pass
    
    async def _check_write_permission(
        self,
        agent_id: str,
        knowledge_id: str
    ) -> bool:
        """检查写入权限"""
        return True  # 简化实现
    
    async def _check_read_permission(self, agent_id: str) -> bool:
        """检查读取权限"""
        return True  # 简化实现
    
    def _rerank(
        self,
        query: str,
        results: List[Dict],
        agent_id: str
    ) -> List[Dict]:
        """结果重排序"""
        # 实现Rerank逻辑
        return results[:10]
    
    def _apply_filters(
        self,
        results: List[Dict],
        filters: Dict
    ) -> List[Dict]:
        """应用过滤器"""
        filtered = []
        for r in results:
            meta = r.get('metadata', {})
            if all(meta.get(k) == v for k, v in filters.items()):
                filtered.append(r)
        return filtered
    
    async def _get_lock(self, knowledge_id: str) -> asyncio.Lock:
        """获取锁"""
        if knowledge_id not in self.locks:
            self.locks[knowledge_id] = asyncio.Lock()
        return self.locks[knowledge_id]
```

## 冲突解决机制

### 冲突类型

| 冲突类型 | 描述 | 解决策略 |
|----------|------|----------|
| **资源冲突** | 多Agent争用同一资源 | 锁机制/优先级 |
| **决策冲突** | 不同Agent给出不同建议 | 投票/仲裁 |
| **状态冲突** | 并发修改导致状态不一致 | 乐观锁/版本控制 |
| **知识冲突** | 知识库中存在矛盾信息 | 置信度评估 |

### 冲突解决实现

```python
from enum import Enum
from typing import List, Optional, Any
from dataclasses import dataclass

class ConflictResolutionStrategy(Enum):
    """冲突解决策略"""
    PRIORITY = "priority"           # 优先级策略
    VOTING = "voting"               # 投票策略
    ARBITRATION = "arbitration"     # 仲裁策略
    MERGE = "merge"                 # 合并策略
    TIMESTAMP = "timestamp"          # 时间戳策略

@dataclass
class Conflict:
    """冲突信息"""
    conflict_id: str
    conflict_type: str
    agents: List[str]
    candidates: List[Dict]
    context: Dict

class ConflictResolver:
    """冲突解决器"""
    
    def __init__(self):
        self.strategies = {
            ConflictResolutionStrategy.PRIORITY: self._priority_resolve,
            ConflictResolutionStrategy.VOTING: self._voting_resolve,
            ConflictResolutionStrategy.ARBITRATION: self._arbitration_resolve,
            ConflictResolutionStrategy.MERGE: self._merge_resolve,
        }
        self.agent_priorities: Dict[str, int] = {}
    
    def set_priority(self, agent_id: str, priority: int):
        """设置Agent优先级"""
        self.agent_priorities[agent_id] = priority
    
    async def resolve(
        self,
        conflict: Conflict,
        strategy: ConflictResolutionStrategy
    ) -> Any:
        """解决冲突"""
        resolver = self.strategies.get(strategy)
        if resolver:
            return await resolver(conflict)
        return conflict.candidates[0] if conflict.candidates else None
    
    async def _priority_resolve(self, conflict: Conflict) -> Any:
        """优先级策略"""
        candidates = conflict.candidates
        
        # 按Agent优先级排序
        sorted_candidates = sorted(
            candidates,
            key=lambda x: self.agent_priorities.get(x.get('agent_id'), 0),
            reverse=True
        )
        
        return sorted_candidates[0]['result'] if sorted_candidates else None
    
    async def _voting_resolve(self, conflict: Conflict) -> Any:
        """投票策略"""
        votes = defaultdict(int)
        
        # 统计投票
        for candidate in conflict.candidates:
            result_key = str(candidate.get('result'))
            votes[result_key] += 1
        
        # 返回票数最多的结果
        if votes:
            winner = max(votes.items(), key=lambda x: x[1])
            # 找到对应的candidate
            for c in conflict.candidates:
                if str(c.get('result')) == winner[0]:
                    return c['result']
        
        return None
    
    async def _arbitration_resolve(self, conflict: Conflict) -> Any:
        """仲裁策略"""
        # 使用预定义的仲裁规则
        if conflict.conflict_type == 'resource':
            # 资源冲突：先到先得
            return conflict.candidates[0]['result']
        elif conflict.conflict_type == 'decision':
            # 决策冲突：使用置信度
            best = max(conflict.candidates, key=lambda x: x.get('confidence', 0))
            return best['result']
        
        return None
    
    async def _merge_resolve(self, conflict: Conflict) -> Any:
        """合并策略"""
        # 尝试合并多个结果
        results = [c.get('result') for c in conflict.candidates if c.get('result')]
        
        if not results:
            return None
        
        if isinstance(results[0], dict):
            # 合并字典
            merged = {}
            for r in results:
                merged.update(r)
            return merged
        elif isinstance(results[0], list):
            # 合并列表
            merged = []
            seen = set()
            for r in results:
                for item in r:
                    if item not in seen:
                        merged.append(item)
                        seen.add(item)
            return merged
        else:
            # 返回第一个
            return results[0]
```

## 完整实战案例

### 案例：智能研究助手

```python
class ResearchAssistantSystem:
    """研究助手系统"""
    
    def __init__(self, config: Dict):
        # 初始化组件
        self.broker = MessageBroker()
        self.knowledge_base = SharedKnowledgeBase(
            vector_store=self._init_vector_store(config),
            graph_store=GraphStore()
        )
        
        # 初始化Agent
        self.coordinator = CoordinatorAgent(
            name="coordinator",
            model=config['model'],
            broker=self.broker
        )
        self.search_agent = SearchAgent(
            name="search",
            model=config['model'],
            broker=self.broker
        )
        self.analysis_agent = AnalysisAgent(
            name="analysis",
            model=config['model'],
            broker=self.broker
        )
        self.writer_agent = WriterAgent(
            name="writer",
            model=config['model'],
            broker=self.broker
        )
        self.reviewer_agent = ReviewerAgent(
            name="reviewer",
            model=config['model'],
            broker=self.broker
        )
        
        # 注册到协调者
        self.coordinator.register_workers({
            'search': self.search_agent,
            'analysis': self.analysis_agent,
            'writer': self.writer_agent,
            'reviewer': self.reviewer_agent
        })
        
        # 启动消息代理
        self.broker.start()
    
    async def research(self, topic: str) -> str:
        """执行研究任务"""
        # 1. 协调者规划任务
        plan = await self.coordinator.plan(topic)
        
        if plan['type'] == 'simple':
            # 简单查询直接回答
            return await self.coordinator.answer(topic)
        
        # 2. 并行执行搜索与分析
        search_task = self.search_agent.search(plan['search_queries'])
        analysis_task = self.analysis_agent.analyze(plan['analysis_framework'])
        
        search_results, analysis_results = await asyncio.gather(
            search_task, analysis_task
        )
        
        # 3. 存储到知识库
        await self.knowledge_base.write(
            agent_id='search',
            content=json.dumps(search_results),
            knowledge_type='research_data',
            metadata={'topic': topic}
        )
        
        # 4. 写作Agent生成初稿
        draft = await self.writer_agent.write(
            topic=topic,
            search_results=search_results,
            analysis_results=analysis_results
        )
        
        # 5. 审核Agent质量检查
        review = await self.reviewer_agent.review(draft)
        
        # 6. 如需修订，循环处理
        iteration = 0
        max_iterations = 3
        
        while review['needs_revision'] and iteration < max_iterations:
            draft = await self.writer_agent.revise(
                draft, 
                review['feedback']
            )
            review = await self.reviewer_agent.review(draft)
            iteration += 1
        
        return draft['final_report']
    
    def _init_vector_store(self, config: Dict):
        """初始化向量存储"""
        # 根据配置选择合适的向量存储
        if config.get('use_pinecone'):
            return PineconeVectorStore(
                api_key=config['pinecone_api_key'],
                index_name=config['pinecone_index']
            )
        return InMemoryVectorStore()


# 使用示例
async def main():
    config = {
        'model': 'gpt-4o',
        'use_pinecone': False,
    }
    
    assistant = ResearchAssistantSystem(config)
    
    topic = "AI大模型在医疗领域的应用前景"
    report = await assistant.research(topic)
    
    print(report)


# 运行
asyncio.run(main())
```

### Agent基类实现

```python
from abc import ABC, abstractmethod
from typing import Dict, Any, List, Optional

class BaseAgent(ABC):
    """Agent基类"""
    
    def __init__(
        self,
        name: str,
        model: str,
        broker: MessageBroker = None
    ):
        self.name = name
        self.model = model
        self.broker = broker
        self.capabilities: List[str] = []
        self.llm = LLMClient(model)
        self.memory: List[Dict] = []
    
    @abstractmethod
    async def process(self, input_data: Any) -> Dict[str, Any]:
        """处理输入，返回结果"""
        pass
    
    async def send_message(
        self,
        receiver: str,
        action: str,
        params: Dict[str, Any]
    ) -> Optional[AgentMessage]:
        """发送消息"""
        if not self.broker:
            return None
        
        try:
            response = await self.broker.request(
                sender_id=self.name,
                receiver_id=receiver,
                action=action,
                params=params
            )
            return response
        except TimeoutError:
            logger.warning(f"Request to {receiver} timed out")
            return None
    
    async def broadcast(
        self,
        action: str,
        params: Dict[str, Any]
    ):
        """广播消息"""
        if not self.broker:
            return
        
        message = AgentMessage(
            sender={'agent_id': self.name},
            receiver={},  # 空表示广播
            message_type=MessageType.BROADCAST,
            content={'action': action, 'params': params}
        )
        await self.broker.publish(message)
    
    def add_to_memory(self, item: Dict):
        """添加到记忆"""
        self.memory.append(item)
        # 限制记忆大小
        if len(self.memory) > 100:
            self.memory.pop(0)


class CoordinatorAgent(BaseAgent):
    """协调Agent"""
    
    def __init__(self, name: str, model: str, broker: MessageBroker):
        super().__init__(name, model, broker)
        self.workers: Dict[str, BaseAgent] = {}
        self.router = TaskRouter()
    
    def register_workers(self, workers: Dict[str, BaseAgent]):
        """注册Worker"""
        self.workers = workers
        for name, worker in workers.items():
            self.router.register_worker(worker)
    
    async def plan(self, user_input: str) -> Dict[str, Any]:
        """任务规划"""
        prompt = f"""
分析用户请求：{user_input}

可用能力：
{self._format_capabilities()}

判断任务类型和需要的协作方式。
"""
        
        response = await self.llm.chat([{"role": "user", "content": prompt}])
        
        try:
            return json.loads(response)
        except:
            return {'type': 'simple'}
    
    async def process(self, input_data: Any) -> Dict[str, Any]:
        """处理协调任务"""
        plan = await self.plan(input_data)
        
        if plan['type'] == 'simple':
            return await self.answer(input_data)
        else:
            return await self._delegate_to_workers(plan)
    
    async def answer(self, question: str) -> Dict[str, Any]:
        """直接回答"""
        response = await self.llm.chat([{"role": "user", "content": question}])
        return {'answer': response, 'confidence': 1.0}
    
    async def _delegate_to_workers(self, plan: Dict) -> Dict:
        """委托给Worker处理"""
        results = []
        
        for step in plan.get('steps', []):
            worker = self.router.route(step)
            if worker:
                result = await worker.process(step)
                results.append(result)
        
        return await self.synthesize(results)
    
    async def synthesize(self, results: List[Dict]) -> Dict:
        """整合结果"""
        synthesis_prompt = f"""
整合以下结果：
{json.dumps(results, ensure_ascii=False)}

生成最终回答。
"""
        
        response = await self.llm.chat([{"role": "user", "content": synthesis_prompt}])
        
        return {
            'answer': response,
            'confidence': 0.8,
            'sources': results
        }
    
    def _format_capabilities(self) -> str:
        return "\n".join([
            f"- {name}: {worker.capabilities}"
            for name, worker in self.workers.items()
        ])


class SearchAgent(BaseAgent):
    """搜索Agent"""
    
    def __init__(self, name: str, model: str, broker: MessageBroker):
        super().__init__(name, model, broker)
        self.capabilities = ["web_search", "knowledge_query"]
    
    async def search(self, queries: List[str]) -> Dict[str, Any]:
        """执行搜索"""
        # 实现搜索逻辑
        results = []
        for query in queries:
            # 调用搜索API
            search_result = await self._perform_search(query)
            results.append(search_result)
        
        return {
            'query_results': results,
            'query_count': len(queries)
        }
    
    async def process(self, input_data: Any) -> Dict[str, Any]:
        """处理任务"""
        if isinstance(input_data, dict) and 'queries' in input_data:
            return await self.search(input_data['queries'])
        
        return await self.search([str(input_data)])
    
    async def _perform_search(self, query: str) -> Dict:
        """执行搜索"""
        # 简化实现
        return {'query': query, 'results': []}


class WriterAgent(BaseAgent):
    """写作Agent"""
    
    def __init__(self, name: str, model: str, broker: MessageBroker):
        super().__init__(name, model, broker)
        self.capabilities = ["writing", "summarization"]
    
    async def write(
        self,
        topic: str,
        search_results: Dict,
        analysis_results: Dict
    ) -> Dict[str, Any]:
        """生成报告"""
        prompt = f"""
根据以下研究和分析结果，撰写一份研究报告：

主题：{topic}

搜索结果：
{json.dumps(search_results, ensure_ascii=False)}

分析结果：
{json.dumps(analysis_results, ensure_ascii=False)}

要求：
1. 结构完整，包含摘要、正文、结论
2. 内容详实，有数据支撑
3. 语言流畅，逻辑清晰
"""
        
        report = await self.llm.chat([{"role": "user", "content": prompt}])
        
        return {
            'draft': report,
            'confidence': 0.7
        }
    
    async def revise(self, draft: Dict, feedback: str) -> Dict:
        """修订报告"""
        prompt = f"""
根据反馈修订报告：

原报告：
{draft.get('draft')}

反馈：
{feedback}

请根据反馈修改报告。
"""
        
        revised = await self.llm.chat([{"role": "user", "content": prompt}])
        
        return {
            'draft': revised,
            'confidence': 0.8
        }
    
    async def process(self, input_data: Any) -> Dict[str, Any]:
        return await self.write(
            input_data.get('topic'),
            input_data.get('search_results', {}),
            input_data.get('analysis_results', {})
        )


class ReviewerAgent(BaseAgent):
    """审核Agent"""
    
    def __init__(self, name: str, model: str, broker: MessageBroker):
        super().__init__(name, model, broker)
        self.capabilities = ["review", "quality_check"]
    
    async def review(self, draft: Dict) -> Dict[str, Any]:
        """审核报告"""
        prompt = f"""
审核以下研究报告：

{draft.get('draft')}

检查：
1. 内容是否准确
2. 结构是否合理
3. 是否有逻辑漏洞
4. 是否符合要求

返回JSON：
{{
  "needs_revision": true/false,
  "feedback": "具体反馈",
  "issues": ["问题列表"]
}}
"""
        
        response = await self.llm.chat([{"role": "user", "content": prompt}])
        
        try:
            result = json.loads(response)
            return {
                'needs_revision': result.get('needs_revision', False),
                'feedback': result.get('feedback', ''),
                'issues': result.get('issues', [])
            }
        except:
            return {
                'needs_revision': False,
                'feedback': '',
                'issues': []
            }
    
    async def process(self, input_data: Any) -> Dict[str, Any]:
        return await self.review(input_data)
```

## 总结

多Agent系统设计核心要点：

| 方面 | 关键点 |
|------|--------|
| **协作模式** | 串行/并行/分层/网状，根据场景选择 |
| **通信协议** | 统一消息格式、异步通信、消息队列 |
| **知识共享** | 向量存储+图存储、权限控制、版本管理 |
| **冲突解决** | 优先级/投票/仲裁/合并 |
| **容错设计** | 超时处理、重试机制、降级方案 |

---

## 相关资源

- [[n8n平台深度指南]] - 工作流编排平台
- [[Function Calling与工具调用]] - 工具调用规范
- [[AI对话记忆系统]] - 记忆系统设计
- [[AI应用API化部署]] - 系统部署方案

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
