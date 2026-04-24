---
title: Pi Framework深度解析
date: 2026-04-18
tags:
  - Pi-Framework
  - 极简框架
  - 工具注册
  - 对话管理
  - Plugin机制
  - Agent开发
  - 工具调用
  - 系统提示
  - 模型适配
  - Mario-Zechner
categories:
  - 小龙虾生态
  - Pi_Framework
alias: Pi Framework Deep Dive
---

# Pi Framework 深度解析

> [!note] 文档信息
> 本文深入解析 Pi Framework 的设计哲学、核心机制、工具注册、对话管理、Plugin 机制等关键内容。

---

## 关键词速览

| 关键词 | 说明 |
|--------|------|
| Pi Framework | 极简 Agent 开发框架 |
| 极简设计 | 4 工具 + <1000 tokens 提示词 |
| Mario Zechner | libGDX 创始人，Pi 作者 |
| 工具注册 | 灵活的 Tool 注册机制 |
| 对话管理 | 消息流与上下文管理 |
| Plugin 机制 | 插件扩展接口 |
| System Prompt | 系统提示词压缩 |
| LLM Adapter | 模型适配器模式 |
| 状态管理 | 会话状态与上下文 |
| 错误处理 | 重试与降级策略 |

---

## 一、项目概述

### 1.1 Pi 的起源

Pi Framework 由 **Mario Zechner**（libGDX 游戏引擎创始人）开发，是 OpenClaw 的底层运行时核心。Mario 在游戏引擎领域深耕多年后，将「极简主义」的设计理念带入 AI Agent 开发领域，创造了 Pi Framework。

**设计哲学的核心问题**：

> 复杂的 Agent 框架是否真的需要复杂的设计？

Pi 的答案是：**否**。通过精心设计的极简接口，Pi 证明了简单也可以很强大。

### 1.2 设计目标

| 目标 | 说明 |
|------|------|
| **极简接口** | 最小化学习成本，快速上手 |
| **高效运行** | 低资源消耗，高响应速度 |
| **灵活扩展** | 通过 Plugin 机制扩展功能 |
| **可靠稳定** | 完善的错误处理与重试机制 |
| **可组合** | 与其他系统易于集成 |

### 1.3 与其他框架对比

| 框架 | 工具数量 | 系统提示 | 学习曲线 | 适用场景 |
|------|----------|----------|----------|----------|
| **Pi** | 4 | <1000 tokens | 低 | 轻量级 Agent |
| LangChain | 100+ | 数千 tokens | 高 | 复杂应用 |
| AutoGen | 50+ | 数千 tokens | 高 | 多 Agent 协作 |
| LlamaIndex | 30+ | 中等 | 中 | 知识检索增强 |

---

## 二、核心架构

### 2.1 模块结构

```
Pi Framework
├── core/
│   ├── agent.py          # Agent 核心类
│   ├── message.py        # 消息模型
│   ├── context.py        # 上下文管理
│   └── tool.py           # 工具基类
├── adapters/
│   ├── anthropic.py      # Anthropic 适配器
│   ├── openai.py         # OpenAI 适配器
│   └── base.py           # 适配器基类
├── plugins/
│   ├── base.py           # 插件基类
│   ├── registry.py       # 插件注册表
│   └── loader.py         # 插件加载器
└── utils/
    ├── prompt.py         # 提示词构建
    └── retry.py          # 重试机制
```

### 2.2 核心类图

```python
# Pi Framework 核心类结构

class PiAgent:
    """Pi Agent 主类"""
    
    def __init__(
        self,
        llm: LLMAdapter,
        tools: List[Tool],
        system_prompt: str,
        config: PiConfig
    ):
        self.llm = llm
        self.tools = ToolRegistry(tools)
        self.system_prompt = system_prompt
        self.config = config
        self.context = Context()
    
    async def process(self, messages: List[Message]) -> Response:
        """处理用户消息"""
        pass

class LLMAdapter(ABC):
    """LLM 适配器基类"""
    async def complete(self, prompt: str, **kwargs) -> Response:
        pass

class Tool(ABC):
    """工具基类"""
    name: str
    description: str
    params: List[Param]
    async def execute(self, **kwargs) -> ToolResult:
        pass

class PiPlugin(ABC):
    """插件基类"""
    name: str
    version: str
    def register(self, agent: PiAgent):
        pass
```

---

## 三、极简工具系统

### 3.1 四大核心工具

Pi 仅提供 **4 个核心工具**，这是经过精心设计的「最小功能集」：

| 工具 | 功能 | 用途 |
|------|------|------|
| **read** | 读取文件内容 | 获取信息 |
| **write** | 写入文件内容 | 保存信息 |
| **edit** | 编辑文件 | 修改信息 |
| **bash** | 执行 Shell 命令 | 执行操作 |

### 3.2 工具设计原则

```python
# Pi 工具设计原则

# 1. 正交性：每个工具职责单一
class ReadTool(Tool):
    """只负责读取，不做其他操作"""
    name = "read"
    
    async def execute(self, file_path: str, **kwargs) -> ToolResult:
        """读取文件"""
        try:
            content = await read_file(file_path)
            return ToolResult(success=True, output=content)
        except Exception as e:
            return ToolResult(success=False, error=str(e))

# 2. 显式性：操作结果明确返回
@dataclass
class ToolResult:
    success: bool
    output: Optional[str] = None
    error: Optional[str] = None
    metadata: dict = field(default_factory=dict)

# 3. 可组合性：工具可以组合使用
async def execute_tool_chain(tools: List[Tool], params: List[dict]):
    """工具链执行"""
    results = []
    for tool, param in zip(tools, params):
        result = await tool.execute(**param)
        results.append(result)
        if not result.success:
            break
    return results
```

### 3.3 工具注册机制

```python
class ToolRegistry:
    """工具注册表"""
    
    def __init__(self):
        self._tools: Dict[str, Tool] = {}
    
    def register(self, tool: Tool) -> None:
        """注册工具"""
        if tool.name in self._tools:
            raise ValueError(f"Tool {tool.name} already registered")
        self._tools[tool.name] = tool
    
    def unregister(self, name: str) -> None:
        """注销工具"""
        self._tools.pop(name, None)
    
    def get(self, name: str) -> Optional[Tool]:
        """获取工具"""
        return self._tools.get(name)
    
    def list_all(self) -> List[Tool]:
        """列出所有工具"""
        return list(self._tools.values())
    
    def to_prompt(self) -> str:
        """生成工具描述（用于提示词）"""
        lines = ["Available tools:"]
        for tool in self._tools.values():
            lines.append(f"- {tool.name}: {tool.description}")
            if tool.params:
                lines.append(f"  Parameters: {', '.join(tool.params)}")
        return "\n".join(lines)
```

### 3.4 自定义工具示例

```python
# 创建自定义天气查询工具
from pi.core import Tool, ToolResult

class WeatherTool(Tool):
    """天气查询工具"""
    
    name = "weather"
    description = "查询指定城市的天气信息"
    params = ["city", "units"]
    
    def __init__(self, api_key: str):
        self.api_key = api_key
    
    async def execute(
        self, 
        city: str, 
        units: str = "celsius",
        **kwargs
    ) -> ToolResult:
        """执行天气查询"""
        try:
            weather = await self._fetch_weather(city, units)
            return ToolResult(
                success=True,
                output=f"{city}天气：{weather['temp']}°{units[0].upper()}, {weather['condition']}",
                metadata={"raw_data": weather}
            )
        except Exception as e:
            return ToolResult(success=False, error=str(e))
    
    async def _fetch_weather(self, city: str, units: str) -> dict:
        """获取天气数据"""
        # 实际实现中调用天气 API
        pass

# 注册工具
agent = PiAgent(...)
agent.tools.register(WeatherTool(api_key="YOUR_KEY"))
```

---

## 四、对话管理系统

### 4.1 消息模型

```python
@dataclass
class Message:
    """消息模型"""
    id: str
    role: str  # "user" | "assistant" | "system" | "tool"
    content: str
    timestamp: datetime
    metadata: dict = field(default_factory=dict)

@dataclass
class Conversation:
    """对话模型"""
    id: str
    messages: List[Message] = field(default_factory=list)
    created_at: datetime
    updated_at: datetime
    metadata: dict = field(default_factory=dict)
    
    def add_message(self, message: Message) -> None:
        """添加消息"""
        self.messages.append(message)
        self.updated_at = datetime.now()
    
    def get_recent(self, n: int = 10) -> List[Message]:
        """获取最近 n 条消息"""
        return self.messages[-n:]
```

### 4.2 上下文管理

```python
class Context:
    """上下文管理器"""
    
    def __init__(
        self,
        max_tokens: int = 4096,
        system_prompt: str = ""
    ):
        self.max_tokens = max_tokens
        self.system_prompt = system_prompt
        self._messages: List[Message] = []
        self._memory: List[Message] = []
    
    def add(self, message: Message) -> None:
        """添加消息"""
        self._messages.append(message)
        self._trim()
    
    def inject_memory(self, memories: List[str]) -> None:
        """注入记忆到上下文"""
        memory_content = "\n\n".join(memories)
        system_with_memory = (
            f"{self.system_prompt}\n\n"
            f"## Relevant Memories:\n{memory_content}"
        )
        return system_with_memory
    
    def _trim(self) -> None:
        """裁剪过长的上下文"""
        while self._estimate_tokens() > self.max_tokens:
            if len(self._messages) > 2:
                self._messages.pop(0)
            else:
                break
    
    def _estimate_tokens(self) -> int:
        """估算 token 数量"""
        total = len(self.system_prompt.split())
        for msg in self._messages:
            total += len(msg.content.split())
        return total
    
    def to_llm_format(self) -> List[dict]:
        """转换为 LLM 格式"""
        messages = [{"role": "system", "content": self.system_prompt}]
        messages.extend([
            {"role": m.role, "content": m.content}
            for m in self._messages
        ])
        return messages
```

### 4.3 系统提示词优化

Pi Framework 追求 **系统提示词 < 1000 tokens**，这需要精心优化：

```python
class PromptOptimizer:
    """提示词优化器"""
    
    @staticmethod
    def compress(prompt: str, max_tokens: int = 1000) -> str:
        """压缩提示词"""
        
        # 1. 移除冗余空白
        prompt = re.sub(r'\s+', ' ', prompt).strip()
        
        # 2. 压缩示例
        prompt = PromptOptimizer._compress_examples(prompt)
        
        # 3. 提取关键指令
        if len(prompt.split()) > max_tokens:
            prompt = PromptOptimizer._extract_essentials(prompt)
        
        return prompt
    
    @staticmethod
    def _compress_examples(prompt: str) -> str:
        """压缩示例"""
        
        # 替换长示例为简短引用
        example_pattern = r'Example:.*?(?=\n\n|\n#|$)'
        
        def shorten_example(match):
            example = match.group(0)
            if len(example) > 200:
                return f"[Example: {len(example.split())} words, truncated]"
            return example
        
        return re.sub(example_pattern, shorten_example, prompt, flags=re.DOTALL)
    
    @staticmethod
    def _extract_essentials(prompt: str) -> str:
        """提取核心指令"""
        
        essentials = [
            "You are a helpful AI assistant.",
            "Use available tools when needed.",
            "Provide clear, concise responses.",
        ]
        
        # 尝试提取用户定义的核心部分
        sections = prompt.split("\n\n")
        core_sections = []
        
        for section in sections:
            if any(kw in section.lower() for kw in ["important", "must", "always", "critical"]):
                core_sections.append(section)
        
        if core_sections:
            return "\n\n".join(essentials + core_sections)
        
        return "\n\n".join(essentials)
```

---

## 五、Plugin 机制

### 5.1 插件接口定义

```python
class PiPlugin(ABC):
    """Pi 插件基类"""
    
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
    def description(self) -> str:
        """插件描述"""
        return ""
    
    @property
    def dependencies(self) -> List[str]:
        """依赖插件"""
        return []
    
    async def on_load(self, agent: PiAgent) -> None:
        """插件加载时调用"""
        pass
    
    async def on_unload(self) -> None:
        """插件卸载时调用"""
        pass
    
    def register_tools(self) -> List[Tool]:
        """注册工具"""
        return []
    
    def register_handlers(self) -> Dict[str, Callable]:
        """注册事件处理器"""
        return {}
```

### 5.2 插件生命周期

```
┌─────────────────────────────────────────┐
│           Plugin Lifecycle               │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│  load() → 检查依赖 → 解析配置            │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│  on_load() → 注册工具/处理器              │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│        [Plugin Active Period]           │
│    工具调用 / 事件处理 / 状态管理        │
└─────────────────────────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│  on_unload() → 清理资源 → 注销           │
└─────────────────────────────────────────┘
```

### 5.3 插件开发示例

```python
# 示例：创建 GitHub 集成插件
from pi.plugins import PiPlugin
from pi.core import Tool, ToolResult

class GitHubPlugin(PiPlugin):
    """GitHub 集成插件"""
    
    name = "github"
    version = "1.0.0"
    description = "GitHub 集成工具集"
    
    def __init__(self, token: str):
        self.token = token
    
    async def on_load(self, agent: PiAgent):
        """加载时注册工具"""
        # 注册 GitHub 相关工具
        agent.tools.register(GitHubReposTool(self.token))
        agent.tools.register(GitHubIssuesTool(self.token))
        agent.tools.register(GitHubPRsTool(self.token))
    
    def register_handlers(self) -> Dict[str, Callable]:
        """注册事件处理器"""
        return {
            "message.created": self._on_message_created,
            "tool.executed": self._on_tool_executed,
        }

class GitHubReposTool(Tool):
    """GitHub 仓库工具"""
    
    name = "github_repos"
    description = "列出 GitHub 仓库或获取仓库信息"
    params = ["operation", "repo"]
    
    async def execute(
        self,
        operation: str,  # "list" | "get" | "create"
        repo: str = None,
        **kwargs
    ) -> ToolResult:
        """执行 GitHub 操作"""
        
        if operation == "list":
            repos = await self._list_repos()
            return ToolResult(success=True, output=repos)
        
        elif operation == "get" and repo:
            info = await self._get_repo(repo)
            return ToolResult(success=True, output=info)
        
        return ToolResult(success=False, error="Invalid operation")
```

### 5.4 插件注册与加载

```python
class PluginLoader:
    """插件加载器"""
    
    def __init__(self, plugin_dir: str = "./plugins"):
        self.plugin_dir = Path(plugin_dir)
        self._registry: Dict[str, PiPlugin] = {}
        self._dependencies: Dict[str, List[str]] = {}
    
    async def load_plugin(
        self,
        plugin_class: Type[PiPlugin],
        config: dict = None
    ) -> PiPlugin:
        """加载插件"""
        
        plugin = plugin_class(**(config or {}))
        
        # 检查依赖
        for dep in plugin.dependencies:
            if dep not in self._registry:
                raise DependencyError(f"Missing dependency: {dep}")
        
        # 初始化插件
        await plugin.on_load(self._agent)
        
        # 注册
        self._registry[plugin.name] = plugin
        
        return plugin
    
    async def unload_plugin(self, name: str) -> None:
        """卸载插件"""
        
        if name not in self._registry:
            return
        
        plugin = self._registry[name]
        
        # 调用卸载钩子
        await plugin.on_unload()
        
        # 注销工具
        for tool in plugin.register_tools():
            self._agent.tools.unregister(tool.name)
        
        # 从注册表移除
        del self._registry[name]
    
    def list_plugins(self) -> List[dict]:
        """列出已加载插件"""
        return [
            {"name": p.name, "version": p.version}
            for p in self._registry.values()
        ]
```

---

## 六、LLM 适配器

### 6.1 适配器接口

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Optional, Dict, Any

@dataclass
class LLMResponse:
    """LLM 响应"""
    content: str
    tool_calls: List[Dict[str, Any]] = None
    usage: Dict[str, int] = None
    model: str = None
    metadata: Dict[str, Any] = None

class LLMAdapter(ABC):
    """LLM 适配器基类"""
    
    @property
    @abstractmethod
    def provider(self) -> str:
        """提供商名称"""
        pass
    
    @property
    @abstractmethod
    def default_model(self) -> str:
        """默认模型"""
        pass
    
    @abstractmethod
    async def complete(
        self,
        messages: List[dict],
        tools: List[Tool] = None,
        **kwargs
    ) -> LLMResponse:
        """完成对话"""
        pass
    
    @abstractmethod
    async def stream(
        self,
        messages: List[dict],
        **kwargs
    ) -> AsyncIterator[str]:
        """流式输出"""
        pass
```

### 6.2 Anthropic 适配器实现

```python
class AnthropicAdapter(LLMAdapter):
    """Anthropic Claude 适配器"""
    
    def __init__(
        self,
        api_key: str,
        model: str = "claude-sonnet-4-20250514"
    ):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.model = model
    
    @property
    def provider(self) -> str:
        return "anthropic"
    
    @property
    def default_model(self) -> str:
        return self.model
    
    async def complete(
        self,
        messages: List[dict],
        tools: List[Tool] = None,
        **kwargs
    ) -> LLMResponse:
        """调用 Claude"""
        
        # 分离 system 和 messages
        system = next(
            (m["content"] for m in messages if m["role"] == "system"),
            ""
        )
        conversation = [m for m in messages if m["role"] != "system"]
        
        # 构建请求参数
        params = {
            "model": kwargs.get("model", self.model),
            "max_tokens": kwargs.get("max_tokens", 4096),
            "system": system,
            "messages": conversation,
        }
        
        # 添加工具定义
        if tools:
            params["tools"] = [self._convert_tool(t) for t in tools]
        
        # 调用 API
        response = self.client.messages.create(**params)
        
        # 解析响应
        return self._parse_response(response)
    
    def _convert_tool(self, tool: Tool) -> dict:
        """转换工具定义"""
        return {
            "name": tool.name,
            "description": tool.description,
            "input_schema": {
                "type": "object",
                "properties": {
                    p: {"type": "string"}
                    for p in tool.params
                },
                "required": tool.params
            }
        }
    
    def _parse_response(self, response) -> LLMResponse:
        """解析 API 响应"""
        
        content = response.content[0].text
        tool_calls = []
        
        # 检查是否有工具调用
        for block in response.content[1:]:
            if block.type == "tool_use":
                tool_calls.append({
                    "name": block.name,
                    "input": block.input,
                    "id": block.id
                })
        
        return LLMResponse(
            content=content,
            tool_calls=tool_calls if tool_calls else None,
            usage={
                "input_tokens": response.usage.input_tokens,
                "output_tokens": response.usage.output_tokens
            },
            model=response.model
        )
```

### 6.3 多模型路由

```python
class ModelRouter:
    """多模型路由器"""
    
    def __init__(self):
        self.adapters: Dict[str, LLMAdapter] = {}
        self.default: str = None
    
    def register(self, name: str, adapter: LLMAdapter):
        """注册适配器"""
        self.adapters[name] = adapter
        if not self.default:
            self.default = name
    
    async def complete(
        self,
        messages: List[dict],
        provider: str = None,
        **kwargs
    ) -> LLMResponse:
        """路由到指定或最优模型"""
        
        provider = provider or self.default
        
        if provider not in self.adapters:
            raise ValueError(f"Unknown provider: {provider}")
        
        adapter = self.adapters[provider]
        return await adapter.complete(messages, **kwargs)
    
    def select_optimal(
        self,
        task_type: str,
        context_length: int = 0
    ) -> str:
        """根据任务类型选择最优模型"""
        
        strategies = {
            "code": ["openai", "anthropic"],
            "reasoning": ["anthropic", "openai"],
            "fast": ["groq", "openai"],
            "creative": ["openai", "anthropic"],
            "long_context": ["anthropic", "openai"]
        }
        
        candidates = strategies.get(task_type, [self.default])
        
        for candidate in candidates:
            if candidate in self.adapters:
                return candidate
        
        return self.default
```

---

## 七、错误处理与重试

### 7.1 重试策略

```python
from functools import wraps
import asyncio

def retry(
    max_attempts: int = 3,
    delay: float = 1.0,
    exponential_backoff: bool = True,
    exceptions: tuple = (Exception,)
):
    """重试装饰器"""
    
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    return await func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    
                    if attempt < max_attempts - 1:
                        wait_time = delay * (
                            2 ** attempt if exponential_backoff else 1
                        )
                        await asyncio.sleep(wait_time)
            
            raise last_exception
        
        return wrapper
    
    return decorator

# 使用示例
class ReliableAgent(PiAgent):
    """带重试的 Agent"""
    
    @retry(max_attempts=3, delay=1.0)
    async def process_with_retry(self, messages):
        return await self.process(messages)
```

### 7.2 降级策略

```python
class FallbackManager:
    """降级管理器"""
    
    def __init__(self):
        self.fallbacks: Dict[str, List[str]] = {}
    
    def register_fallback(
        self,
        primary: str,
        fallback: str
    ) -> None:
        """注册降级方案"""
        if primary not in self.fallbacks:
            self.fallbacks[primary] = []
        self.fallbacks[primary].append(fallback)
    
    async def execute_with_fallback(
        self,
        func: Callable,
        *args,
        **kwargs
    ):
        """执行带降级的函数"""
        
        try:
            return await func(*args, **kwargs)
        except Exception as e:
            primary = func.__name__
            
            if primary in self.fallbacks:
                for fallback_name in self.fallbacks[primary]:
                    fallback_func = getattr(self, fallback_name, None)
                    if fallback_func:
                        try:
                            return await fallback_func(*args, **kwargs)
                        except:
                            continue
            
            raise e
```

---

## 八、集成示例

### 8.1 快速创建 Agent

```python
from pi import PiAgent, AnthropicAdapter, ToolRegistry
from pi.tools import ReadTool, WriteTool, EditTool, BashTool

async def main():
    # 1. 创建 LLM 适配器
    llm = AnthropicAdapter(
        api_key="your-api-key",
        model="claude-sonnet-4-20250514"
    )
    
    # 2. 注册工具
    tools = ToolRegistry()
    tools.register(ReadTool())
    tools.register(WriteTool())
    tools.register(EditTool())
    tools.register(BashTool())
    
    # 3. 定义系统提示
    system_prompt = """You are a helpful coding assistant.
You have access to file operations and shell commands.
Always explain your actions and verify dangerous operations."""
    
    # 4. 创建 Agent
    agent = PiAgent(
        llm=llm,
        tools=tools,
        system_prompt=system_prompt
    )
    
    # 5. 处理对话
    response = await agent.process([
        {"role": "user", "content": "Hello, help me with coding."}
    ])
    
    print(response.content)

asyncio.run(main())
```

### 8.2 带插件的完整配置

```python
from pi import PiAgent, PluginLoader
from pi.adapters import AnthropicAdapter

async def main():
    # 1. 初始化
    llm = AnthropicAdapter(api_key="your-key")
    agent = PiAgent(llm=llm, ...)
    
    # 2. 加载插件
    loader = PluginLoader("./plugins")
    await loader.load_plugin(GitHubPlugin, {"token": "gh_token"})
    await loader.load_plugin(WeatherPlugin, {"api_key": "weather_key"})
    await loader.load_plugin(CalculatorPlugin)
    
    # 3. 使用
    response = await agent.process(messages)
    print(response.content)

asyncio.run(main())
```

---

## 九、相关文档

- [[OpenClaw完整指南]] - OpenClaw 完整指南
- [[OpenClaw架构解析]] - OpenClaw 架构
- [[OpenClaw插件开发]] - 插件开发指南
- [[OpenClaw记忆系统]] - 记忆系统
- [[Hermes_Agent详解]] - Hermes Agent

---

*文档更新于 2026年4月18日*
