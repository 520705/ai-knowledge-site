---
title: Pi Framework 深度解析
date: 2026-04-24
tags:
  - Pi-Framework
  - 极简设计
  - 工具注册
  - Mario-Zechner
  - AI-Agent
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 深入了解 Pi Framework 的设计哲学，为什么只有4个工具却能做很多事。
---

# Pi Framework 深度解析

> 这篇是关于 Pi Framework 的详细介绍。如果你想知道 OpenClaw 的"大脑"是怎么设计的，为什么它这么"简"，这篇适合你看。

---

## Pi 是什么？

### 一句话解释

**Pi Framework = 一个极简的 AI Agent 开发框架**

它只给你 4 个核心工具（read/write/edit/bash），但通过组合，这 4 个工具可以完成几乎所有任务。

### 名字的由来

Pi（π）是：
- 数学里的圆周率
- 无限不循环
- 看似简单，实则无穷

这正是 Pi Framework 的设计哲学：**工具极少，但能力无限**。

### 谁写的？

Mario Zechner，libGDX 游戏引擎的创始人。

这位老哥做了十几年游戏引擎，深谙一个道理：

> **最好的 API 是最少的 API。**

于是他做了 Pi——一个"极简到极致"的 AI Agent 框架。

---

## 设计哲学：为什么要这么"简"？

### 对比一下

| 框架 | 工具数量 | 系统提示词 | 学习曲线 |
|------|----------|------------|----------|
| **Pi** | 4 | < 1000 tokens | ⭐ 低 |
| LangChain | 100+ | 数千 tokens | ⭐⭐⭐⭐ 高 |
| AutoGen | 50+ | 数千 tokens | ⭐⭐⭐ 高 |
| LlamaIndex | 30+ | 中等 | ⭐⭐⭐ 中 |

### 复杂框架的问题

用 LangChain 的时候，你可能会：

```
"我需要用哪个工具？"
"这个工具怎么调用？"
"为什么报错了？"
"这参数是啥意思？"
```

一个框架给你 100 个工具，意味着你要学习 100 个 API。

### Pi 的答案

**4 个工具，4 个概念：**

| 工具 | 做什么 | 类比 |
|------|--------|------|
| **read** | 读文件 | 眼睛 |
| **write** | 写文件 | 手（写） |
| **edit** | 改文件 | 手（改） |
| **bash** | 执行命令 | 手（操作） |

> 想象你有一双万能的手，可以做任何事。Pi 就是给 AI 装上这双万能的手。

### 组合的力量

**场景：用 Pi 做一个天气查询功能**

```python
# Pi 方式
# 1. 读一个配置文件
content = read("config.json")

# 2. 解析配置
config = json.loads(content)

# 3. 调用天气 API
result = bash(f"curl 'https://api.weather.com?city={config['city']}'")

# 4. 写日志
write("weather.log", result)
```

**LangChain 方式**：
```python
# 找 WeatherTool
# 初始化它
# 配置参数
# 调用
# 解析结果
```

虽然 Pi 看起来"手写"，但其实更灵活——你完全控制每一步。

---

## 核心概念

### 1. 工具（Tool）

Pi 的工具和其他框架不太一样：

```python
class Tool:
    name: str              # 工具名
    description: str        # 描述（给 AI 看的）
    params: List[str]      # 参数列表

    async def execute(**kwargs) -> ToolResult:
        # 执行逻辑
        pass
```

**工具的设计原则**：

| 原则 | 解释 | 例子 |
|------|------|------|
| **正交性** | 每个工具只做一件事 | read 只读，不写 |
| **显式性** | 操作结果明确返回 | 成功/失败+结果 |
| **可组合** | 工具可以串联 | 先 read 再 edit |

### 2. 上下文（Context）

Context 是 AI "思考"的原材料：

```python
class Context:
    system_prompt: str       # 系统提示词
    messages: List[Message]  # 对话历史
    memories: List[str]       # 记忆内容

    def add_message(role, content):
        # 添加对话

    def inject_memory(memories):
        # 注入记忆
```

**上下文是怎么构建的？**

```python
# 1. 系统提示词（固定不变）
system = "你是一个 helpful 的 AI 助手。"

# 2. 记忆（检索相关记忆）
relevant_memories = memory_manager.retrieve(query)
context = "\n".join(relevant_memories)

# 3. 对话历史（最近 N 条）
recent_messages = session.messages[-20:]

# 4. 组装
full_context = f"""
{system}

## 相关记忆
{context}

## 对话历史
{recent_messages}
"""
```

### 3. LLM 适配器（Adapter）

Pi 支持多个 AI 提供商：

```python
# Anthropic (Claude)
adapter = AnthropicAdapter(api_key="sk-ant-xxx")

# OpenAI (GPT)
adapter = OpenAIAdapter(api_key="sk-xxx")

# Google (Gemini)
adapter = GoogleAdapter(api_key="xxx")

# Groq (免费快)
adapter = GroqAdapter(api_key="xxx")
```

**为什么需要适配器？**

因为每个 AI 提供商的 API 都不一样：
- 参数名不同
- 返回格式不同
- token 计算方式不同

适配器把这些差异封装起来，让 Pi 的核心代码不用关心这些。

---

## 核心代码解析

### Pi Agent 主循环

```python
class PiAgent:
    def __init__(self, llm, tools, system_prompt):
        self.llm = llm              # AI 适配器
        self.tools = tools           # 工具列表
        self.system_prompt = system_prompt  # 系统提示

    async def process(self, messages):
        # 1. 构建上下文
        context = self.build_context(messages)

        # 2. 调用 LLM
        response = await self.llm.complete(
            prompt=context,
            tools=self.tools
        )

        # 3. 处理响应
        if response.tool_calls:
            # 需要调用工具
            return await self.execute_tools(response.tool_calls)
        else:
            # 直接返回文字
            return response.content

    def build_context(self, messages):
        # 组合系统提示 + 对话历史
        pass
```

### 工具注册

```python
# 注册工具
registry = ToolRegistry()

registry.register(ReadTool())
registry.register(WriteTool())
registry.register(EditTool())
registry.register(BashTool())

# 或者用装饰器
@registry.register
class MyTool(Tool):
    name = "my_tool"
    description = "我的自定义工具"

    async def execute(self, param1, param2):
        # 工具逻辑
        return ToolResult(success=True, output="结果")
```

### 自定义工具示例

**做一个"翻译工具"**：

```python
class TranslateTool(Tool):
    name = "translate"
    description = "将文本从一种语言翻译到另一种语言"
    params = ["text", "from_lang", "to_lang"]

    async def execute(self, text, from_lang, to_lang):
        # 调用翻译 API
        result = await translation_service.translate(
            text=text,
            source=from_lang,
            target=to_lang
        )

        return ToolResult(
            success=True,
            output=result.translated_text,
            metadata={
                "from": from_lang,
                "to": to_lang
            }
        )
```

**做一个"文件搜索工具"**：

```python
class FileSearchTool(Tool):
    name = "file_search"
    description = "在指定目录下搜索包含关键词的文件"
    params = ["directory", "keyword"]

    async def execute(self, directory, keyword):
        # 用 bash 命令搜索
        result = await bash(f'grep -r "{keyword}" {directory}')

        # 解析结果
        files = result.stdout.strip().split('\n')

        return ToolResult(
            success=True,
            output=f"找到 {len(files)} 个匹配",
            data=files
        )
```

---

## 工具详解

### read（读文件）

**用途**：读取文件内容

**参数**：
- `file_path`: 文件路径（必填）

**示例**：

```
Tool: read
Params: {"file_path": "/home/user/notes/todo.md"}
```

**返回**：
```json
{
  "success": true,
  "output": "# TODO\n\n- [x] 完成报告\n- [ ] 买牛奶\n- [ ] 给妈妈打电话"
}
```

**常见用途**：
- 读笔记
- 读配置
- 读代码
- 读文档

---

### write（写文件）

**用途**：创建或覆盖文件

**参数**：
- `file_path`: 文件路径（必填）
- `content`: 文件内容（必填）

**示例**：

```
Tool: write
Params: {
  "file_path": "/home/user/notes/new.md",
  "content": "# 新笔记\n\n这是新建的笔记内容。"
}
```

**注意**：
- 如果文件已存在，会**覆盖**
- 不会创建目录（需要先 mkdir）

---

### edit（编辑文件）

**用途**：修改文件的部分内容（比 write 更精准）

**参数**：
- `file_path`: 文件路径
- `old_string`: 要替换的原文（必须精确匹配）
- `new_string`: 替换后的内容

**示例**：

```python
# 原始文件
# # TODO
# - 完成报告
# - 买牛奶

# 调用 edit
edit(
    file_path="todo.md",
    old_string="- 完成报告",
    new_string="- [x] 完成报告"
)

# 修改后
# # TODO
# - [x] 完成报告
# - 买牛奶
```

**技巧**：
- `old_string` 要精确匹配，包括空格和换行
- 建议先 `read`，再看准了再 `edit`

---

### bash（执行命令）

**用途**：执行任何 shell 命令

**参数**：
- `command`: 要执行的命令

**示例**：

```
Tool: bash
Params: {"command": "ls -la /home/user"}
```

**返回值**：
```json
{
  "success": true,
  "stdout": "total 32\ndrwxr-xr-x  4 user user 4096 Apr 24 10:30 .",
  "stderr": ""
}
```

**常用命令**：
```bash
# 列出文件
ls, ll, ls -la

# 切换目录
cd /path/to/dir

# 创建目录
mkdir -p path/to/dir

# 复制文件
cp source dest

# 删除文件
rm file

# 读取文件内容
cat file

# 搜索
grep "pattern" file
find . -name "*.py"

# Git 操作
git status, git commit, git push

# 网络请求
curl https://api.example.com

# Docker
docker ps, docker logs, docker exec
```

**危险命令**（慎用！）：
```bash
rm -rf /      # 删除根目录，千万别跑！
dd if=/dev/zero of=/dev/sda  # 格式化硬盘！
```

> Pi 有安全检查，但最好还是别让 AI 执行危险的命令。

---

## 对话管理

### 消息历史

```python
class Conversation:
    def __init__(self):
        self.messages = []

    def add(self, role, content):
        """添加消息
        role: "user" | "assistant" | "system" | "tool"
        """
        self.messages.append(Message(role, content))

    def get_recent(self, n=10):
        """获取最近 N 条消息"""
        return self.messages[-n:]

    def trim(self, max_tokens=4000):
        """裁剪过长的对话"""
        while self.estimate_tokens() > max_tokens:
            # 删除最早的用户消息（保留系统消息）
            for i, msg in enumerate(self.messages):
                if msg.role == "user":
                    del self.messages[i]
                    break
```

### 上下文压缩

当对话太长时，Pi 会自动压缩：

```python
async def compress_context(messages):
    """压缩上下文，保留关键信息"""

    # 1. 提取关键信息
    summary_prompt = """
    请总结以下对话的关键信息，保留：
    - 用户的重要偏好
    - 讨论的结论
    - 未完成的任务

    对话：
    {messages}
    """

    summary = await llm.complete(summary_prompt)

    # 2. 重建上下文
    return f"""
    ## 对话摘要
    {summary}

    ## 最近对话
    {messages[-10:]}
    """
```

---

## 系统提示词优化

Pi 的目标是把系统提示词压到 **1000 tokens 以内**。

### 为什么重要？

| 问题 | 影响 |
|------|------|
| 提示词太长 | 消耗更多 token，花更多钱 |
| 响应空间小 | AI 回复被截断 |
| 处理速度慢 | 每次都要处理大量文本 |

### 优化技巧

**1. 删除废话**

```python
# ❌ 啰嗦版
system = """
你是一个极其聪明、超级智能、无所不能的 AI 助手。
你由世界上最优秀的工程师团队使用最新最先进的技术开发。
你的目标是为用户提供最优质、最有帮助、最专业的服务。
请用友好的语气和用户交流。
"""

# ✅ 简洁版
system = "你是一个友好、helpful 的中文 AI 助手。"
```

**2. 用例子代替说明**

```python
# ❌ 说明版
system = """
你应该用中文回复。
如果用户说"你好"，回复"你好！有什么可以帮你的？"
如果用户问天气，用"🌤️"开头。
"""

# ✅ 示例版
system = """
示例对话：
用户: 你好
助手: 你好！有什么可以帮你的？

用户: 北京天气
助手: 🌤️ 北京今天天气晴，22°C
"""
```

**3. 提取关键约束**

```python
# 只保留最重要的规则
system = """
你是一个中文 AI 助手。
规则：
1. 始终用中文回复
2. 不知道就说"我不确定"
3. 有危险的操作要警告用户
"""
```

---

## 与 OpenClaw 的关系

### Pi 是 OpenClaw 的"大脑"

```
OpenClaw
├── Gateway（消息路由）
│   ├── Channel 适配器
│   ├── Session 管理
│   └── Memory 管理
│
└── Pi Agent（AI 核心）
    ├── 提示词构建
    ├── LLM 调用
    └── 工具执行
```

### Pi 可以单独用吗？

**可以！**

Pi 是一个独立的框架，不依赖 OpenClaw：

```python
from pi import PiAgent, AnthropicAdapter
from pi.tools import ReadTool, WriteTool, BashTool

# 1. 创建适配器
llm = AnthropicAdapter(api_key="sk-ant-xxx")

# 2. 创建 Agent
agent = PiAgent(
    llm=llm,
    tools=[ReadTool(), WriteTool(), BashTool()],
    system_prompt="你是一个有帮助的助手。"
)

# 3. 对话
response = await agent.process("你好")
print(response)
```

### 什么时候用 Pi？什么时候用 OpenClaw？

| 场景 | 推荐 |
|------|------|
| 快速搭一个聊天机器人 | OpenClaw（开箱即用） |
| 深度定制 Agent 行为 | Pi（更灵活） |
| 接多个聊天平台 | OpenClaw |
| 只需要 AI 调用工具 | Pi |
| 想要丰富的插件生态 | OpenClaw |

---

## 性能对比

| 指标 | Pi | LangChain |
|------|-----|-----------|
| 响应速度 | 快 | 较慢 |
| 内存占用 | 低 | 高 |
| 代码量 | 少 | 多 |
| 学习成本 | 低 | 高 |
| 灵活性 | 高 | 中 |

---

## 下一步

学会了 Pi？继续学习：

| 教程 | 内容 |
|------|------|
| [[OpenClaw架构解析]] | Pi 在 OpenClaw 中怎么配合 |
| [[OpenClaw插件开发]] | 用 Pi 做插件开发 |
| [[OpenClaw高级用法]] | Pi 的高级用法 |
| [[Hermes_Agent详解]] | 另一个 Agent 框架 |

---

**文档状态**：Pi Framework 深度解析  
**更新时间**：2026年4月
