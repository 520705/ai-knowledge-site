---
title: OpenClaw架构解析
date: 2026-04-18
tags:
  - OpenClaw
  - 架构设计
  - Gateway
  - Pi-Framework
  - Agent运行时
  - 消息路由
  - 渠道适配器
  - 记忆层
  - 工具系统
  - 插件架构
categories:
  - 小龙虾生态
  - OpenClaw核心
alias: OpenClaw Architecture Deep Dive
---

# OpenClaw 架构解析

> [!note] 文档信息
> 本文深入剖析 OpenClaw 的技术架构，涵盖 Gateway 网关、Pi Framework、Agent 运行时、消息路由、渠道适配器、记忆层等核心组件的设计原理与实现细节。

---

## 关键词速览

| 关键词 | 说明 |
|--------|------|
| Gateway | 消息路由与渠道适配的核心网关 |
| Pi Framework | 嵌入式极简 Agent SDK |
| Channel Adapter | 适配器模式统一处理各平台 |
| Session Manager | 持久化会话状态管理 |
| Memory Manager | Markdown 文件记忆系统 |
| Tool Executor | Docker 沙箱工具执行器 |
| Multi-Model Router | 多模型路由选择 |
| Hub-and-Spoke | 中心辐射型消息架构 |
| Workspace | 隔离的工作区机制 |
| Plugin Interface | 插件扩展接口 |

---

## 一、整体架构概览

### 1.1 分层架构设计

OpenClaw 采用**分层架构**，从上到下依次为：

```
┌─────────────────────────────────────────────────────────────┐
│                    渠道接入层 (Channel Layer)                 │
│   Discord Adapter │ Telegram Adapter │ WhatsApp Adapter   │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    网关层 (Gateway Layer)                     │
│   WebSocket Control Plane │ Hub-and-Spoke Routing          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    运行时层 (Runtime Layer)                   │
│   Pi Agent SDK │ Tool Executor │ Memory Manager            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    模型层 (Model Layer)                       │
│   Multi-Model Router │ LLM API Adapters                     │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 数据流架构

```
用户消息 → Channel Adapter → Session Manager → Pi Agent SDK
                                                      │
                                                      ▼
                                            Memory Manager (检索记忆)
                                                      │
                                                      ▼
                                            Multi-Model Router (选择模型)
                                                      │
                                                      ▼
                                            LLM API Call
                                                      │
                                                      ▼
工具调用 ← Tool Executor ← 工具决策 ← LLM Response
         │
         ▼
    Docker Sandbox
         │
         ▼
    执行结果 → Memory Manager (存储记忆) → 格式化输出 → Channel Adapter → 用户
```

---

## 二、Gateway 网关层

### 2.1 Gateway 的核心职责

Gateway 是 OpenClaw 的**中央枢纽**，承担以下核心职责：

| 职责 | 说明 |
|------|------|
| 协议转换 | 将各平台协议统一转换为内部消息格式 |
| 消息路由 | Hub-and-Spoke 模式分发消息 |
| 会话管理 | 维护用户与 Agent 的会话状态 |
| 记忆注入 | 在消息前后注入相关记忆 |
| 流量控制 | 请求限流、并发控制 |

### 2.2 WebSocket Control Plane

Gateway 提供 WebSocket 控制平面（默认端口 18789）：

```python
# WebSocket 控制平面接口
class ControlPlane:
    """WebSocket 控制平面 API"""
    
    async def connect(self, workspace: str) -> WebSocket:
        """连接到指定工作区"""
        pass
    
    async def send_message(self, message: Message) -> Response:
        """发送消息"""
        pass
    
    async def get_status(self) -> Status:
        """获取运行状态"""
        pass
```

### 2.3 Hub-and-Spoke 消息路由

OpenClaw 采用 **Hub-and-Spoke（中心辐射型）** 路由模式：

```
                    ┌──────────────┐
                    │   Gateway    │
                    │   (Hub)      │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
   ┌─────────┐       ┌─────────┐       ┌─────────┐
   │ Discord │       │Telegram │       │ WhatsApp│
   │ Adapter │       │ Adapter │       │ Adapter │
   └─────────┘       └─────────┘       └─────────┘
```

**路由算法**：

```python
async def route_message(message: Message) -> None:
    """消息路由核心逻辑"""
    # 1. 解析消息来源
    channel = message.source_channel
    user_id = message.user_id
    
    # 2. 获取或创建会话
    session = await SessionManager.get_or_create(
        workspace=message.workspace,
        user_id=user_id,
        channel=channel
    )
    
    # 3. 检索相关记忆
    memories = await MemoryManager.retrieve(
        query=message.content,
        session=session,
        limit=10
    )
    
    # 4. 构建上下文
    context = build_context(session, memories)
    
    # 5. 转发至 Pi Agent SDK
    response = await PiAgent.process(context)
    
    # 6. 存储新记忆
    await MemoryManager.store(session, response)
    
    # 7. 发送响应
    await ChannelAdapter.send(channel, user_id, response)
```

---

## 三、Pi Framework 详解

### 3.1 Pi 的设计哲学

Pi Framework 追求**极简主义**，核心设计原则：

1. **最小化系统提示**：目标 < 1000 tokens
2. **四个核心工具**：read、write、edit、bash
3. **无状态设计**：Pi 实例本身无状态，状态由外部管理
4. **工具正交**：工具之间无功能重叠

### 3.2 Pi Agent SDK 核心实现

```python
class PiAgent:
    """Pi Agent SDK 核心类"""
    
    def __init__(
        self,
        llm: LLMAdapter,
        tools: List[Tool],
        system_prompt: str,
        max_retries: int = 3
    ):
        self.llm = llm
        self.tools = tools
        self.system_prompt = system_prompt
        self.max_retries = max_retries
    
    async def process(self, messages: List[Message]) -> Response:
        """处理用户消息"""
        # 构建提示
        prompt = self.build_prompt(messages)
        
        # 调用 LLM
        response = await self.llm.complete(
            prompt=prompt,
            tools=self.tools,
            system=self.system_prompt
        )
        
        # 处理工具调用
        if response.tool_calls:
            return await self.execute_tools(response.tool_calls)
        
        return response
    
    def build_prompt(self, messages: List[Message]) -> str:
        """构建 LLM 输入提示"""
        # 限制上下文窗口
        recent_messages = messages[-20:]
        return "\n".join([
            f"{msg.role}: {msg.content}" 
            for msg in recent_messages
        ])
```

### 3.3 工具注册机制

```python
# Pi 工具注册示例
class ToolRegistry:
    """工具注册表"""
    
    @staticmethod
    def get_default_tools() -> List[Tool]:
        """获取默认工具集"""
        return [
            Tool(
                name="read",
                description="读取文件内容",
                params=["file_path"],
                handler=ReadTool()
            ),
            Tool(
                name="write",
                description="写入文件内容",
                params=["file_path", "content"],
                handler=WriteTool()
            ),
            Tool(
                name="edit",
                description="编辑文件",
                params=["file_path", "old_string", "new_string"],
                handler=EditTool()
            ),
            Tool(
                name="bash",
                description="执行 Shell 命令",
                params=["command"],
                handler=BashTool()
            ),
        ]
```

---

## 四、Channel Adapter 模式

### 4.1 适配器接口定义

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import AsyncIterator

@dataclass
class Message:
    """统一消息格式"""
    id: str
    source_channel: str
    user_id: str
    content: str
    timestamp: datetime
    metadata: dict

class ChannelAdapter(ABC):
    """渠道适配器抽象基类"""
    
    @property
    @abstractmethod
    def channel_name(self) -> str:
        """渠道名称"""
        pass
    
    @abstractmethod
    async def start(self) -> None:
        """启动适配器"""
        pass
    
    @abstractmethod
    async def stop(self) -> None:
        """停止适配器"""
        pass
    
    @abstractmethod
    async def send(self, user_id: str, content: str) -> None:
        """发送消息"""
        pass
    
    @abstractmethod
    async def stream(self) -> AsyncIterator[Message]:
        """消息流"""
        pass
```

### 4.2 Telegram 适配器实现

```python
class TelegramAdapter(ChannelAdapter):
    """Telegram 渠道适配器"""
    
    def __init__(
        self,
        bot_token: str,
        api_id: str,
        api_hash: str,
        session: pyrogram.Client
    ):
        self.bot_token = bot_token
        self.api_id = api_id
        self.api_hash = api_hash
        self.session = session
    
    @property
    def channel_name(self) -> str:
        return "telegram"
    
    async def start(self) -> None:
        await self.session.start()
        logger.info("Telegram adapter started")
    
    async def stream(self) -> AsyncIterator[Message]:
        async for message in self.session.iter_messages():
            yield Message(
                id=str(message.id),
                source_channel="telegram",
                user_id=str(message.from_user.id),
                content=message.text,
                timestamp=message.date,
                metadata={"chat_id": message.chat.id}
            )
```

---

## 五、Session Manager

### 5.1 会话状态模型

```python
@dataclass
class Session:
    """会话状态"""
    id: str
    workspace: str
    channel: str
    user_id: str
    created_at: datetime
    last_active: datetime
    messages: List[Message]
    context: dict
    
    async def add_message(self, message: Message) -> None:
        """添加消息到会话"""
        self.messages.append(message)
        self.last_active = datetime.now()
        await self.persist()
    
    async def branch(self, name: str) -> "Session":
        """创建分支会话"""
        new_session = Session(
            id=f"{self.id}_branch_{name}",
            workspace=self.workspace,
            channel=self.channel,
            user_id=self.user_id,
            created_at=datetime.now(),
            last_active=datetime.now(),
            messages=self.messages.copy(),
            context=self.context.copy()
        )
        await new_session.persist()
        return new_session
```

### 5.2 会话持久化

```python
class SessionManager:
    """会话管理器"""
    
    def __init__(self, storage_path: str):
        self.storage_path = Path(storage_path)
        self.storage_path.mkdir(parents=True, exist_ok=True)
        self._cache: Dict[str, Session] = {}
    
    async def get_or_create(
        self, 
        workspace: str, 
        user_id: str, 
        channel: str
    ) -> Session:
        """获取或创建会话"""
        session_id = self._make_session_id(workspace, user_id, channel)
        
        if session_id in self._cache:
            return self._cache[session_id]
        
        # 尝试从磁盘加载
        session_file = self.storage_path / f"{session_id}.json"
        if session_file.exists():
            session = await self._load(session_file)
        else:
            session = Session(
                id=session_id,
                workspace=workspace,
                channel=channel,
                user_id=user_id,
                created_at=datetime.now(),
                last_active=datetime.now(),
                messages=[],
                context={}
            )
            await session.persist()
        
        self._cache[session_id] = session
        return session
```

---

## 六、Memory Manager

### 6.1 记忆存储结构

```
memory/
├── long_term/
│   ├── facts.md          # 用户事实
│   ├── preferences.md    # 偏好设置
│   ├── relationships.md  # 关系网络
│   └── knowledge/        # 领域知识
│       ├── tech.md
│       ├── hobby.md
│       └── ...
├── short_term/
│   └── session_20260418_abc123.md  # 会话临时记忆
└── working/
    └── context.md        # 当前工作上下文
```

### 6.2 记忆检索算法

```python
class MemoryManager:
    """记忆管理器"""
    
    async def retrieve(
        self,
        query: str,
        session: Session,
        limit: int = 10
    ) -> List[MemoryEntry]:
        """检索相关记忆"""
        memories = []
        
        # 1. 加载长期记忆
        long_term_dir = Path(self.long_term_path)
        for memory_file in long_term_dir.rglob("*.md"):
            content = await self._read_file(memory_file)
            score = self._calculate_relevance(query, content)
            if score > 0.3:
                memories.append(MemoryEntry(
                    content=content,
                    source=str(memory_file),
                    score=score,
                    type="long_term"
                ))
        
        # 2. 加载短期记忆
        short_term_file = Path(self.short_term_path) / f"session_{session.id}.md"
        if short_term_file.exists():
            content = await self._read_file(short_term_file)
            memories.append(MemoryEntry(
                content=content,
                source=str(short_term_file),
                score=1.0,  # 当前会话记忆高权重
                type="short_term"
            ))
        
        # 3. 按相关性排序
        memories.sort(key=lambda x: x.score, reverse=True)
        return memories[:limit]
    
    def _calculate_relevance(self, query: str, content: str) -> float:
        """计算相关性分数"""
        query_tokens = set(query.lower().split())
        content_tokens = set(content.lower().split())
        
        # Jaccard 相似度
        intersection = query_tokens & content_tokens
        union = query_tokens | content_tokens
        
        if not union:
            return 0.0
        return len(intersection) / len(union)
```

---

## 七、Multi-Model Router

### 7.1 模型适配器接口

```python
class LLMAdapter(ABC):
    """LLM 适配器抽象基类"""
    
    @abstractmethod
    async def complete(
        self,
        prompt: str,
        system: str,
        tools: List[Tool],
        **kwargs
    ) -> LLMResponse:
        """完成对话"""
        pass

class AnthropicAdapter(LLMAdapter):
    """Anthropic Claude 适配器"""
    
    def __init__(self, api_key: str, model: str):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.model = model
    
    async def complete(self, prompt: str, system: str, tools: List[Tool], **kwargs) -> LLMResponse:
        response = self.client.messages.create(
            model=self.model,
            max_tokens=kwargs.get("max_tokens", 4096),
            system=system,
            messages=[{"role": "user", "content": prompt}],
            tools=[self._convert_tool(tool) for tool in tools]
        )
        return LLMResponse(
            content=response.content[0].text,
            tool_calls=response.content[1:] if len(response.content) > 1 else []
        )
```

### 7.2 模型选择策略

```python
class ModelRouter:
    """模型路由器"""
    
    def __init__(self, config: dict):
        self.adapters: Dict[str, LLMAdapter] = {}
        self.default_model = config.get("default", "anthropic")
        
        for provider, cfg in config.items():
            if provider == "default":
                continue
            self.adapters[provider] = self._create_adapter(provider, cfg)
    
    async def complete(
        self,
        prompt: str,
        context: dict,
        preferred_provider: str = None
    ) -> LLMResponse:
        """智能选择模型完成请求"""
        provider = preferred_provider or self._select_model(context)
        adapter = self.adapters.get(provider, self.adapters[self.default_model])
        return await adapter.complete(**context)
    
    def _select_model(self, context: dict) -> str:
        """根据上下文选择模型"""
        # 简单策略：按任务类型选择
        if context.get("task_type") == "code":
            return "openai"  # GPT-4 代码能力强
        elif context.get("task_type") == "reasoning":
            return "anthropic"  # Claude 推理强
        return self.default_model
```

---

## 八、Tool Executor

### 8.1 工具执行器设计

```python
class ToolExecutor:
    """工具执行器"""
    
    def __init__(self, sandbox: str = "docker"):
        self.sandbox = sandbox
        self.docker_client = docker.from_env() if sandbox == "docker" else None
    
    async def execute(self, tool_call: ToolCall) -> ToolResult:
        """执行工具调用"""
        tool_name = tool_call.name
        params = tool_call.arguments
        
        # 获取工具处理器
        handler = self.registry.get(tool_name)
        if not handler:
            return ToolResult(error=f"Unknown tool: {tool_name}")
        
        # 危险命令使用沙箱
        if self._is_dangerous(tool_name, params):
            return await self._execute_in_sandbox(handler, params)
        
        return await handler.execute(params)
    
    async def _execute_in_sandbox(
        self, 
        handler: Tool, 
        params: dict
    ) -> ToolResult:
        """在 Docker 沙箱中执行"""
        container = self.docker_client.containers.run(
            "openclaw-tool-sandbox:latest",
            remove=True,
            detach=False,
            volumes={
                "openclaw-data": {"bind": "/data", "mode": "ro"}
            },
            command=f"{handler.name} {json.dumps(params)}"
        )
        output = container.logs().decode()
        return ToolResult(output=output)
    
    def _is_dangerous(self, tool_name: str, params: dict) -> bool:
        """判断是否为危险操作"""
        dangerous_patterns = [
            (["bash", "shell"], ["rm", "rm -rf", "dd", "mkfs"]),
            (["network"], ["curl", "wget"], ["--upload", "--post"]),
        ]
        
        if tool_name in ["bash", "shell"]:
            command = params.get("command", "")
            return any(pattern in command for pattern in ["rm -rf", "dd", "mkfs"])
        
        return False
```

---

## 九、插件接口

### 9.1 插件接口定义

```python
class OpenClawPlugin(ABC):
    """OpenClaw 插件接口"""
    
    @property
    @abstractmethod
    def name(self) -> str:
        """插件名称"""
        pass
    
    @property
    @abstractmethod
    def version(self) -> str:
        """插件版本"""
        pass
    
    @property
    def dependencies(self) -> List[str]:
        """依赖插件"""
        return []
    
    @abstractmethod
    async def on_load(self, app: "OpenClawApp") -> None:
        """插件加载时调用"""
        pass
    
    @abstractmethod
    async def on_unload(self) -> None:
        """插件卸载时调用"""
        pass
    
    def get_tools(self) -> List[Tool]:
        """获取插件提供的工具"""
        return []
    
    def get_commands(self) -> List[Command]:
        """获取插件提供的命令"""
        return []
```

### 9.2 插件生命周期

```
插件生命周期：

load() → on_load() → [工具注册/命令注册/事件订阅]
                                      │
                                      ▼
                            [插件活跃期]
                            ┌─────────┴─────────┐
                            │                   │
                            ▼                   ▼
                    工具调用/命令执行     事件处理
                            │                   │
                            └─────────┬─────────┘
                                      │
                                      ▼
                              on_unload() → unload()
```

---

## 十、相关文档

- [[OpenClaw完整指南]] - OpenClaw 完整使用指南
- [[Pi_Framework深度解析]] - Pi Framework 详解
- [[OpenClaw安装部署]] - 部署指南
- [[OpenClaw配置详解]] - 配置项详解
- [[OpenClaw多平台集成]] - 多平台集成
- [[OpenClaw记忆系统]] - 记忆系统
- [[OpenClaw插件开发]] - 插件开发
- [[OpenClaw高级用法]] - 高级用法
- [[Hermes_Agent详解]] - Hermes Agent

---

*文档更新于 2026年4月18日*
