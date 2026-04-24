---
title: Context Engineering 上下文工程
date: 2026-04-24
tags:
  - context-engineering
  - prompt-engineering
  - llm
  - information-architecture
  - prompt-optimization
categories:
  - 人工智能
  - LLM应用
---

> [!abstract] 摘要
> Context Engineering（上下文工程）是2025-2026年AI开发领域最重要的技术之一。这篇文章为零基础读者详细讲解：什么是Context Engineering、它和Prompt Engineering有什么区别、为什么上下文质量决定了AI输出的质量、以及如何系统性地设计和优化AI的上下文环境。文章包含大量代码示例和实战技巧，看完你就能开始在实际项目中应用Context Engineering了。

## 先理解一个场景：你和AI的对话出了什么问题？

### 想象一个对话

你打开ChatGPT，问了一个问题：

```
你：帮我写一个用户登录功能
AI：好的，以下是代码...

你：不对，要用JWT
AI：好的，以下是使用JWT的代码...

你：还要支持微信登录
AI：好的，以下是支持微信登录的代码...

你：我之前说的那个电商项目，不是新项目
AI：抱歉，我不知道你的电商项目...
```

**问题出在哪？**

- AI"忘记"了你之前说的"电商项目"
- AI不知道你的技术栈偏好
- AI不清楚你的代码规范

**这就是Context Engineering要解决的问题！**

---

## 一、什么是Context Engineering？

### 用搬家的比喻理解

```
Prompt Engineering（提示工程）：
像是在搬家时告诉搬家工人："小心点搬"

Context Engineering（上下文工程）：
像是给搬家工人准备好：
- 哪件东西放哪个房间（信息结构）
- 哪些是易碎品（重要标记）
- 物品的价值（优先级）
```

### 正式定义

**Context Engineering** 是系统性地设计、构建和优化大型语言模型在推理时接收的完整信息环境的学科。

```
关键点：
1. "系统性地" - 不是随意塞内容，是有方法论的
2. "完整信息环境" - 包括所有输入，不只是提示词
3. "学科" - 这是一套需要学习的技能
```

### 上下文里有什么？

当你和AI对话时，进入AI"眼睛"里的内容包括：

```
┌─────────────────────────────────────────────────────────────┐
│                     完整上下文                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1️⃣ System Prompt（系统提示）                                 │
│     你给AI的永久指令，比如"你是一个Python专家"                   │
│                                                              │
│  2️⃣ 对话历史（Chat History）                                  │
│     你和AI之前的对话内容                                       │
│                                                              │
│  3️⃣ 用户输入（User Input）                                    │
│     你当前问的问题                                            │
│                                                              │
│  4️⃣ 外部知识（RAG检索结果）                                   │
│     从文档/数据库查到的相关信息                                │
│                                                              │
│  5️⃣ 工具输出（Tool Results）                                  │
│     搜索引擎、计算器等工具返回的结果                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 二、Context Engineering vs Prompt Engineering

### 核心区别

| 维度 | Prompt Engineering | Context Engineering |
|------|-------------------|---------------------|
| **焦点** | 优化单个提示词 | 设计整个信息环境 |
| **范围** | 一次对话 | 跨会话持久系统 |
| **控制方式** | 每次手动调整 | 自动化维护 |
| **时间成本** | 每次都要想 | 一次性设计，长期受益 |
| **类比** | 选词措辞 | 决定桌上放什么资料 |

### 一个形象的比喻

```
Prompt Engineering：选词
"这个问题怎么问比较好？"
↓
"请分析数据" vs "帮我分析一下这份销售报表"
↓

Context Engineering：准备资料
"回答这个问题需要哪些背景知识？"
↓
在提问前，先准备好：
- 销售报表数据
- 往期对比数据
- 分析模板
```

### 为什么Context Engineering更重要？

研究数据告诉我们：

```
在生产级AI系统中：

├── 提示词本身（Prompt）
│   └── 只占 10-20% 的内容
│
└── 上下文内容（Context）
    ├── 检索文档：50-70%
    ├── 对话历史：10-20%
    └── 其他：5-10%

结论：优化上下文往往比优化提示词效果更明显！
```

---

## 三、Context Engineering的五大支柱

### 支柱1：检索（Retrieval）

从外部知识库找到相关信息：

```python
class SimpleRetriever:
    """
    简单的检索器示例
    从文档库中找到相关内容
    """

    def __init__(self, documents: list):
        """
        Args:
            documents: 文档列表，每个文档是 {'content': str, 'source': str}
        """
        self.documents = documents

    def retrieve(self, query: str, top_k: int = 3) -> list:
        """
        检索相关文档

        Args:
            query: 用户问题
            top_k: 返回几个结果

        Returns:
            相关文档列表
        """
        # 简单实现：用关键词匹配
        query_words = set(query.lower().split())

        scored = []
        for doc in self.documents:
            content_words = set(doc['content'].lower().split())
            # 计算重叠的词
            overlap = query_words & content_words
            score = len(overlap) / len(query_words) if query_words else 0

            scored.append({
                'doc': doc,
                'score': score
            })

        # 按分数排序，返回top_k
        scored.sort(key=lambda x: x['score'], reverse=True)
        return [s['doc'] for s in scored[:top_k] if s['score'] > 0]


# 使用示例
docs = [
    {'content': 'Python是一门高级编程语言', 'source': 'python.txt'},
    {'content': 'JavaScript用于Web开发', 'source': 'js.txt'},
    {'content': '机器学习是AI的分支', 'source': 'ml.txt'}
]

retriever = SimpleRetriever(docs)
results = retriever.retrieve('Python编程语言')
print(results)  # [{'content': 'Python是一门高级编程语言', 'source': 'python.txt'}]
```

### 支柱2：记忆管理（Memory Management）

管理对话历史和持久化信息：

```python
class SimpleMemoryManager:
    """
    简单的记忆管理器
    管理对话历史和关键信息
    """

    def __init__(self, max_history: int = 10):
        """
        Args:
            max_history: 最多保存多少轮对话
        """
        self.max_history = max_history
        self.messages = []        # 对话历史
        self.facts = {}           # 记住的关键事实

    def add_message(self, role: str, content: str):
        """添加消息"""
        self.messages.append({
            'role': role,
            'content': content
        })

        # 检查是否有关键事实需要记住
        self._extract_facts(content)

        # 清理过长的历史
        if len(self.messages) > self.max_history:
            self.messages = self.messages[-self.max_history:]

    def _extract_facts(self, content: str):
        """提取关键事实（简化版）"""
        # 检测"我叫"、"我叫小明"等模式
        if '我叫' in content or '我的名字是' in content:
            # 简单提取（实际应该用正则）
            self.facts['user_name'] = content
        if '用Python' in content or '用python' in content.lower():
            self.facts['preferred_language'] = 'Python'

    def get_context(self) -> str:
        """获取完整上下文"""
        parts = []

        # 添加记忆的事实
        if self.facts:
            parts.append("[记住的信息]")
            for key, value in self.facts.items():
                parts.append(f"- {key}: {value}")
            parts.append("")

        # 添加对话历史
        for msg in self.messages:
            parts.append(f"{msg['role']}: {msg['content']}")

        return '\n'.join(parts)


# 使用示例
memory = SimpleMemoryManager(max_history=5)

memory.add_message('user', '我叫小明')
memory.add_message('assistant', '你好小明！')
memory.add_message('user', '帮我写一个登录功能，用Python')
memory.add_message('assistant', '好的，这是Python登录功能...')

context = memory.get_context()
print(context)
```

### 支柱3：状态管理（State Management）

跟踪当前任务状态：

```python
class TaskStateManager:
    """
    任务状态管理器
    跟踪当前任务的进度和上下文
    """

    def __init__(self):
        self.current_task = None
        self.task_steps = []      # 任务步骤
        self.completed_steps = [] # 已完成步骤
        self.pending_data = {}     # 待处理的数据

    def start_task(self, task_name: str, description: str):
        """开始新任务"""
        self.current_task = {
            'name': task_name,
            'description': description,
            'status': 'in_progress'
        }
        self.task_steps = []
        self.completed_steps = []

    def add_step(self, step: str):
        """添加任务步骤"""
        self.task_steps.append(step)

    def complete_step(self, step: str):
        """标记步骤完成"""
        if step in self.task_steps and step not in self.completed_steps:
            self.completed_steps.append(step)

    def get_status(self) -> dict:
        """获取当前状态"""
        progress = len(self.completed_steps) / len(self.task_steps) if self.task_steps else 0

        return {
            'task': self.current_task,
            'progress': f"{progress:.0%}",
            'completed': self.completed_steps,
            'pending': [s for s in self.task_steps if s not in self.completed_steps],
            'data': self.pending_data
        }


# 使用示例
state = TaskStateManager()
state.start_task('用户登录功能', '实现完整的用户认证系统')
state.add_step('设计数据库表')
state.add_step('实现注册API')
state.add_step('实现登录API')
state.add_step('添加JWT支持')
state.complete_step('设计数据库表')
state.pending_data['user_table'] = 'CREATE TABLE users...'

print(state.get_status())
```

### 支柱4：上下文压缩（Context Compression）

在有限token内塞入更多信息：

```python
class SimpleContextCompressor:
    """
    简单的上下文压缩器
    把长文本压缩成短摘要
    """

    def __init__(self, llm_client):
        self.llm = llm_client

    def compress(self, text: str, target_length: int = 500) -> str:
        """
        压缩文本

        Args:
            text: 要压缩的文本
            target_length: 目标长度（字符数）

        Returns:
            压缩后的文本
        """
        # 简化实现：直接截断 + 摘要提示
        # 实际应该用LLM生成摘要

        if len(text) <= target_length:
            return text

        # 简单截断到目标长度
        compressed = text[:target_length] + "..."

        return compressed

    def smart_compress(self, text: str, max_tokens: int = 500) -> str:
        """
        智能压缩（需要LLM）
        """
        prompt = f"""请将以下文本压缩到大约{max_tokens}字，保留核心信息：

{text}

压缩要求：
1. 删除次要细节
2. 保留关键数据和结论
3. 保持原文的逻辑结构

压缩后："""

        return self.llm.generate(prompt)


# 使用示例
class ConversationCompressor:
    """
    对话历史压缩器
    定期把旧对话压缩成摘要
    """

    def __init__(self, llm_client, compress_after: int = 10):
        self.llm = llm_client
        self.compress_after = compress_after  # 多少轮后压缩
        self.history = []
        self.summary = None

    def add(self, role: str, content: str):
        """添加消息"""
        self.history.append({'role': role, 'content': content})

        # 检查是否需要压缩
        if len(self.history) >= self.compress_after:
            self._compress_old()

    def _compress_old(self):
        """压缩旧对话"""
        if len(self.history) < self.compress_after:
            return

        # 取前半部分压缩
        old_messages = self.history[:len(self.history)//2]

        prompt = f"""总结以下对话的核心内容，保留关键信息：

{self._format_messages(old_messages)}

总结要求：
1. 保留主要话题和决定
2. 保留用户的核心需求
3. 删除闲聊和细节
4. 控制在100字以内

总结："""

        self.summary = self.llm.generate(prompt)

        # 保留后半部分
        self.history = self.history[len(old_messages):]

    def get_context(self) -> str:
        """获取完整上下文"""
        parts = []

        if self.summary:
            parts.append(f"[早期对话摘要] {self.summary}\n")

        for msg in self.history:
            parts.append(f"{msg['role']}: {msg['content']}")

        return '\n'.join(parts)

    def _format_messages(self, messages: list) -> str:
        return '\n'.join([f"{m['role']}: {m['content']}" for m in messages])
```

### 支柱5：信息路由（Information Routing）

决定什么信息在什么时候用：

```python
class ContextRouter:
    """
    上下文路由器
    根据任务类型决定使用哪些信息
    """

    def __init__(self):
        # 不同类型的任务需要不同的上下文
        self.routing_rules = {
            'code_review': ['code_standards', 'recent_commits'],
            'data_analysis': ['data_schema', 'business_rules'],
            'customer_service': ['user_history', 'product_info'],
            'general': []
        }

    def route(self, task_type: str, available_contexts: dict) -> list:
        """
        根据任务类型路由上下文

        Args:
            task_type: 任务类型
            available_contexts: 可用的上下文 {'context_name': content}

        Returns:
            需要使用的上下文列表
        """
        # 获取该任务类型需要的上下文类型
        needed_types = self.routing_rules.get(task_type, self.routing_rules['general'])

        # 筛选可用的上下文
        selected = []
        for ctx_type in needed_types:
            if ctx_type in available_contexts:
                selected.append({
                    'type': ctx_type,
                    'content': available_contexts[ctx_type]
                })

        return selected


# 使用示例
router = ContextRouter()

available = {
    'code_standards': '编码规范：使用PEP8...',
    'recent_commits': '最近提交：修复了登录bug...',
    'data_schema': '数据库结构...',
    'user_history': '用户历史...'
}

contexts = router.route('code_review', available)
for ctx in contexts:
    print(f"[{ctx['type']}]\n{ctx['content']}\n")
```

---

## 四、五类上下文详解

### 对AI编程助手，五类上下文特别重要

```
┌─────────────────────────────────────────────────────────────┐
│                 AI编程助手的五类上下文                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📐 架构上下文                                               │
│  ├─ 系统由哪些组件构成                                       │
│  ├─ 组件之间怎么通信                                        │
│  └─ 数据怎么流转                                            │
│                                                              │
│  💻 代码库上下文                                             │
│  ├─ 有哪些工具函数可用                                      │
│  ├─ 现有代码的风格                                          │
│  └─ 依赖关系                                                │
│                                                              │
│  📋 业务领域上下文                                           │
│  ├─ 这个领域的专业术语                                      │
│  ├─ 业务规则和限制                                          │
│  └─ 合规要求                                                │
│                                                              │
│  🔧 开发流程上下文                                           │
│  ├─ Git提交规范                                             │
│  ├─ 代码审查要求                                            │
│  └─ 测试覆盖率要求                                          │
│                                                              │
│  📜 历史执行上下文                                           │
│  ├─ 之前尝试过什么                                          │
│  ├─ 遇到过什么问题                                          │
│  └─ 性能数据                                                │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 五、上下文窗口管理

### 核心挑战

```
┌─────────────────────────────────────────────────────────────┐
│                  上下文窗口的三大挑战                            │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1️⃣ Token限制                                               │
│     模型一次能看的token数有限                                  │
│     - GPT-4 Turbo: 128K tokens                              │
│     - Claude 3: 200K tokens                                 │
│     - Gemini 1.5: 1M tokens                                 │
│                                                              │
│  2️⃣ 信号稀释                                                │
│     信息太多，重要的被淹没                                     │
│     研究发现：超过100K tokens后，有效利用率只有60%            │
│                                                              │
│  3️⃣ 中间丢失                                                │
│     "Lost in the Middle" - 模型记不住中间的内容               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 滑动窗口策略

```python
class SlidingWindowManager:
    """
    滑动窗口管理器
    用一个"移动的窗口"管理对话历史
    """

    def __init__(self, window_size: int = 10, overlap: int = 2):
        """
        Args:
            window_size: 窗口大小（多少条消息）
            overlap: 重叠数量（窗口之间重叠几条）
        """
        self.window_size = window_size
        self.overlap = overlap
        self.messages = []

    def add(self, role: str, content: str):
        """添加消息"""
        self.messages.append({'role': role, 'content': content})

    def get_window(self, query: str = None) -> list:
        """
        获取当前窗口内的消息

        Args:
            query: 可选，当前的问题（用于决定窗口位置）

        Returns:
            窗口内的消息列表
        """
        if len(self.messages) <= self.window_size:
            return self.messages

        # 返回最近的窗口
        start = max(0, len(self.messages) - self.window_size)
        return self.messages[start:]

    def get_context(self) -> str:
        """获取窗口内的上下文"""
        window = self.get_window()
        return self._format_messages(window)

    def _format_messages(self, messages: list) -> str:
        return '\n'.join([f"{m['role']}: {m['content']}" for m in messages])
```

### 渐进式披露

```
┌─────────────────────────────────────────────────────────────┐
│                   渐进式披露策略                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  第1层：发现层（始终可见）                                     │
│  ├─ 约80 tokens                                             │
│  ├─ 名称、简单描述                                           │
│  └─ 用于快速判断是否相关                                      │
│                                                              │
│  第2层：激活层（相关时加载）                                   │
│  ├─ 275-2000 tokens                                         │
│  ├─ 完整指令、使用说明                                        │
│  └─ 触发时才加载                                             │
│                                                              │
│  第3层：执行层（任务期间）                                     │
│  ├─ 2000-8000 tokens                                        │
│  ├─ 脚本、详细材料                                           │
│  └─ 任务真正开始时加载                                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 六、上下文结构化最佳实践

### 为什么结构重要？

```
无结构的上下文（AI头疼）：
小明今天去商店买了苹果香蕉和橘子然后回家然后洗了苹果然后吃然后看书然后睡觉

有结构的上下文（AI清爽）：
# 小明的一天

## 早上
- 去了商店
- 买了苹果、香蕉、橘子

## 中午
- 回家
- 洗苹果
- 吃苹果

## 晚上
- 看书
- 睡觉
```

### 推荐格式：XML vs Markdown

**研究数据**：
- Claude在XML格式上得分：87分
- Claude在Markdown格式上得分：71分

```xml
<!-- XML格式示例 -->
<role>Python后端开发专家</role>
<task>优化数据库查询性能</task>
<constraints>
    <limit>响应时间 < 100ms</limit>
    <limit>使用现有ORM框架</limit>
</constraints>
<input>
    <query>当前查询: {{query}}</query>
    <schema>数据库schema: {{schema}}</schema>
</input>
```

```markdown
<!-- Markdown格式示例 -->
## 角色
Python后端开发专家

## 任务
优化数据库查询性能

## 限制
- 响应时间 < 100ms
- 使用现有ORM框架

## 输入
- 当前查询：{{query}}
- 数据库schema：{{schema}}
```

### 结构化提示词模板

```python
class StructuredPromptBuilder:
    """
    结构化提示词构建器
    """

    @staticmethod
    def build_task_prompt(
        role: str,
        task: str,
        context: str,
        constraints: list,
        output_format: str
    ) -> str:
        """
        构建结构化提示词

        Args:
            role: 角色定义
            task: 具体任务
            context: 背景上下文
            constraints: 约束条件
            output_format: 输出格式要求
        """
        prompt = f"""<role>{role}</role>
<task>{task}</task>

<context>
{context}
</context>

<constraints>
{chr(10).join([f"<constraint>{c}</constraint>" for c in constraints])}
</constraints>

<output_format>{output_format}</output_format>"""

        return prompt

    @staticmethod
    def build_code_review_prompt(
        code: str,
        language: str,
        focus_areas: list
    ) -> str:
        """代码审查提示词"""
        return f"""你是{language}代码审查专家。

请审查以下代码，关注以下方面：
{chr(10).join([f"- {area}" for area in focus_areas])}

代码：
```{language}
{code}
```

审查结果（JSON格式）：
{{
    "issues": [...],
    "suggestions": [...],
    "rating": "1-10"
}}
"""


# 使用示例
prompt = StructuredPromptBuilder.build_task_prompt(
    role="资深Python后端工程师",
    task="实现用户认证API",
    context="项目使用FastAPI框架，需要支持JWT认证",
    constraints=[
        "使用现有的User模型",
        "返回标准HTTP状态码",
        "包含适当的错误处理"
    ],
    output_format="提供完整可运行的代码"
)

print(prompt)
```

---

## 七、Context Caching（上下文缓存）

### 缓存能省多少钱？

```
┌─────────────────────────────────────────────────────────────┐
│                     主流平台缓存折扣                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Gemini 2.5+/3:    90% 折扣                                 │
│  Gemini 2.0:       75% 折扣                                 │
│  Claude:            90% 折扣 (0.1x 基础价格)                │
│  GPT-5.4:           90% 折扣                                 │
│  GPT-4:             50% 折扣                                 │
│                                                              │
│  例子：原来$1的请求，缓存后只需$0.1！                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 隐式缓存 vs 显式缓存

```
隐式缓存（自动）：
├── 默认启用，不用配置
├── 自动检测重复内容
├── 没有保证节省
└── 最低限制：1024 tokens

显式缓存（手动）：
├── 手动创建缓存
├── 设置TTL（过期时间）
├── 保证折扣
└── 成本可预测
```

### 缓存使用示例

```python
# 隐式缓存示例
messages = [
    {"role": "system", "content": "你是一个法律顾问..."},  # 重复内容
    {"role": "user", "content": "第一个问题"}
]
response1 = client.chat.completions.create(
    model="claude-3",
    messages=messages
)
# 第二次请求相同系统提示，会自动命中缓存

# 显式缓存示例（伪代码）
cache = client.caches.create(
    instructions="你是一个法律顾问，熟悉中国合同法...",
    ttl_seconds=3600  # 1小时过期
)

# 使用缓存
messages = [
    {"role": "system", "content": cache.id, "cache_control": {"type": "ephemeral"}},
    {"role": "user", "content": "问题1"}
]
response = client.chat.completions.create(
    model="claude-3",
    messages=messages
)
```

---

## 八、上下文质量评估

### 评估什么？

```
┌─────────────────────────────────────────────────────────────┐
│                  上下文质量四维度                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  🎯 Faithfulness（忠实度）                                   │
│     输出和上下文一致吗？有没有编造？                            │
│                                                              │
│  🔗 Relevance（相关性）                                      │
│     上下文和查询相关吗？                                       │
│                                                              │
│  📊 Coverage（覆盖度）                                      │
│     上下文覆盖了答案需要的全部信息吗？                          │
│                                                              │
│  💡 Utilization（利用率）                                    │
│     模型有没有好好用上下文？                                   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 简单评估器

```python
class SimpleContextEvaluator:
    """
    简单的上下文评估器
    """

    def __init__(self, llm_client):
        self.llm = llm_client

    def evaluate(self, context: str, query: str, answer: str) -> dict:
        """
        评估上下文质量

        Returns:
            {'faithfulness': float, 'relevance': float, 'coverage': float}
        """
        # 评估忠实度
        faithfulness = self._check_faithfulness(context, answer)

        # 评估相关性
        relevance = self._check_relevance(context, query)

        # 评估覆盖度
        coverage = self._check_coverage(context, query, answer)

        return {
            'faithfulness': faithfulness,
            'relevance': relevance,
            'coverage': coverage,
            'overall': (faithfulness + relevance + coverage) / 3
        }

    def _check_faithfulness(self, context: str, answer: str) -> float:
        """检查忠实度"""
        prompt = f"""评估回答是否忠实于上下文。

上下文：
{context[:1000]}

回答：
{answer}

回答是否在上下文中能找到支持？
- 如果是：输出1
- 如果不是：输出0
- 如果部分是：输出0.5

只输出数字："""

        result = self.llm.generate(prompt).strip()
        try:
            return float(result)
        except:
            return 0.5

    def _check_relevance(self, context: str, query: str) -> float:
        """检查相关性"""
        # 简单实现：关键词重叠
        query_words = set(query.lower().split())
        context_words = set(context.lower().split())
        overlap = len(query_words & context_words)
        return overlap / len(query_words) if query_words else 0

    def _check_coverage(self, context: str, query: str, answer: str) -> float:
        """检查覆盖度"""
        # 简化：检查回答长度是否合理
        if len(answer) < 50:
            return 0.3  # 太短，可能没回答完整
        elif len(answer) > 2000:
            return 0.7  # 太长，可能有废话
        else:
            return 0.8  # 适中
```

---

## 九、实战：构建完整的Context Engineering系统

```python
class ContextEngineeringSystem:
    """
    完整的Context Engineering系统
    整合所有组件
    """

    def __init__(
        self,
        llm_client,
        vector_store=None,
        memory_manager=None
    ):
        self.llm = llm_client
        self.vector_store = vector_store
        self.memory = memory_manager or SimpleMemoryManager()
        self.state = TaskStateManager()
        self.router = ContextRouter()
        self.compressor = None  # 可选

        # 配置
        self.config = {
            'max_context_tokens': 8000,
            'enable_retrieval': True,
            'enable_memory': True,
            'enable_state': True
        }

    def query(
        self,
        user_input: str,
        task_type: str = 'general',
        retrieve_top_k: int = 3
    ) -> str:
        """
        处理用户查询

        Args:
            user_input: 用户输入
            task_type: 任务类型
            retrieve_top_k: 检索返回数量

        Returns:
            AI的回答
        """
        # 1. 添加到记忆
        self.memory.add_message('user', user_input)

        # 2. 检索相关文档（如果有）
        retrieved_docs = []
        if self.config['enable_retrieval'] and self.vector_store:
            retrieved_docs = self.vector_store.search(
                query=user_input,
                top_k=retrieve_top_k
            )

        # 3. 路由上下文
        context_sources = {
            'memory': self.memory.get_context(),
        }
        if retrieved_docs:
            context_sources['retrieved'] = '\n\n'.join([
                doc['content'] for doc in retrieved_docs
            ])

        routed = self.router.route(task_type, context_sources)

        # 4. 构建完整上下文
        full_context = self._build_full_context(routed, user_input)

        # 5. 生成回答
        answer = self.llm.generate(full_context)

        # 6. 添加到记忆
        self.memory.add_message('assistant', answer)

        return answer

    def _build_full_context(self, routed: list, user_input: str) -> str:
        """构建完整上下文"""
        parts = []

        # 添加路由来的上下文
        for ctx in routed:
            parts.append(f"[{ctx['type']}]\n{ctx['content']}\n")

        # 添加用户输入
        parts.append(f"[当前问题]\n{user_input}\n")

        return '\n'.join(parts)


# 使用示例
def demo_ce_system():
    """演示完整的Context Engineering系统"""

    # 初始化
    # ce = ContextEngineeringSystem(
    #     llm_client=llm,
    #     vector_store=vector_store
    # )

    # 第一轮对话
    # answer1 = ce.query(
    #     "我叫小明，要做一个电商网站",
    #     task_type='general'
    # )
    # print(f"AI: {answer1}")

    # 第二轮对话
    # answer2 = ce.query(
    #     "帮我设计数据库",
    #     task_type='data_modeling'
    # )
    # print(f"AI: {answer2}")
    # AI应该记得小明要做电商网站

    print("Context Engineering系统演示完成！")
```

---

## 十、常见问题与解决

```
┌─────────────────────────────────────────────────────────────┐
│                  Context Engineering 常见问题                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ❓ 问题1：AI忘记重要信息                                      │
│     ├─ 原因：历史太长被截断                                    │
│     └─ 解决：使用记忆管理器，把重要事实单独存储                 │
│                                                              │
│  ❓ 问题2：AI输出和上下文矛盾                                  │
│     ├─ 原因：上下文不够清晰或模型幻觉                           │
│     └─ 解决：结构化上下文，添加忠实度检查                       │
│                                                              │
│  ❓ 问题3：上下文太长超出限制                                   │
│     ├─ 原因：塞了太多无关信息                                   │
│     └─ 解决：使用路由器，只选相关内容                           │
│                                                              │
│  ❓ 问题4：不同任务需要不同上下文                               │
│     ├─ 原因：通用上下文不够精准                                 │
│     └─ 解决：任务分类 + 动态路由                               │
│                                                              │
│  ❓ 问题5：成本太高                                            │
│     ├─ 原因：每次都传大量上下文                                 │
│     └─ 解决：使用缓存、压缩、渐进式披露                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 十一、工具推荐

```
┌─────────────────────────────────────────────────────────────┐
│                     主流框架对比                                  │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  🛠️ LangChain / LangGraph                                   │
│     ├─ 优点：通用编排框架，适合复杂工作流                      │
│     ├─ 适用：多步骤Agent、条件逻辑                            │
│     └─ 学习曲线：较陡                                         │
│                                                              │
│  🛠️ LlamaIndex                                              │
│     ├─ 优点：数据检索为中心，高级分块                          │
│     ├─ 适用：RAG、知识图谱                                    │
│     └─ 学习曲线：较缓                                         │
│                                                              │
│  🛠️ 两者结合                                                 │
│     ├─ LlamaIndex处理数据                                    │
│     └─ LangGraph处理控制流                                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 十二、总结

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Context Engineering 速查表                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  🎯 核心概念                                                         │
│  ├─ Context Engineering = 系统性地设计AI的完整信息环境               │
│  ├─ 比Prompt Engineering更重要（上下文占80-90%的内容）              │
│  └─ 是2025-2026年AI开发的核心技术                                   │
│                                                                      │
│  🏛️ 五大支柱                                                         │
│  ├─ 检索：从知识库找到相关信息                                      │
│  ├─ 记忆管理：管理对话历史和持久信息                                 │
│  ├─ 状态管理：跟踪任务进度                                          │
│  ├─ 上下文压缩：在有限空间内塞入更多信息                            │
│  └─ 信息路由：决定什么信息在什么时候用                               │
│                                                                      │
│  📊 五类上下文                                                       │
│  ├─ 架构上下文、业务领域上下文、开发流程上下文                       │
│  └─ 代码库上下文、历史执行上下文                                    │
│                                                                      │
│  🔧 关键技术                                                         │
│  ├─ 滑动窗口、渐进式披露、上下文缓存                                │
│  ├─ 混合检索、重排序、压缩                                          │
│  └─ 结构化提示词、评估框架                                          │
│                                                                      │
│  💡 最佳实践                                                         │
│  ├─ 用XML或Markdown结构化内容                                      │
│  ├─ 重要信息放开头或结尾                                            │
│  ├─ 使用缓存节省成本                                                │
│  └─ 定期评估上下文质量                                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 相关主题

- [[上下文窗口深度解析]] - 深入理解窗口限制
- [[滑动窗口技术]] - 滑动窗口的具体实现
- [[上下文压缩技术]] - 压缩方法详解
- [[RAG上下文优化指南]] - RAG中的上下文优化
- [[对话历史管理]] - 对话历史的管理策略
- [[FewShot示例选择]] - 如何选择好的示例
- [[上下文质量评估]] - 评估上下文质量

---

## 参考文献

1. **State of Context Engineering in 2026** - Towards AI, 2026
2. **What is Context Engineering?** - Packmind, 2026
3. **Context Engineering: The Complete Guide** - TECHSY, 2026
4. **Anthropic Claude API Documentation** - Anthropic, 2026
5. **LangChain Context Engineering** - LangChain Docs, 2025
6. **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks** - NeurIPS 2020

---

*本文档为零基础读者深入讲解Context Engineering，包含了大量代码示例和实战技巧。Context Engineering是构建高质量AI应用的核心技能，值得深入学习和实践。*
