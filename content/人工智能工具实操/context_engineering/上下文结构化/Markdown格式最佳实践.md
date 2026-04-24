---
title: Markdown格式最佳实践
date: 2026-04-24
tags:
  - markdown
  - formatting
  - context-structure
  - llm-optimization
  - prompt-engineering
categories:
  - context-engineering
  - structured-context
---

> [!abstract] 摘要
> Markdown是给AI写提示词最常用的格式，但用不好反而会降低效果。这篇为零基础读者讲解：Markdown的基础语法、给LLM用的Markdown最佳实践、如何用Markdown组织复杂上下文、以及常见的Markdown"坑"和解决方案。看完你就能写出结构清晰、AI爱看的Markdown了。

## 先理解一个问题：为什么AI需要Markdown？

### AI看文字的方式和人不一样

```
人眼阅读：
小明去学校，他背着书包，书包里有很多书。
↓ 轻松理解

AI处理：
小明去学校，他背着书包，书包里有很多书。
↓ 机器会"数"token，分析结构，找关键词
```

**AI的特点**：
- 能识别结构标记（如标题、列表）
- 对格式敏感：同样的内容，格式不同效果不同
- 擅长处理有明确边界的区块

### Markdown能让AI更聪明

```
没有Markdown（混乱）：
今天我们讨论了三个问题第一是预算第二是时间第三是人员请分别说明

有Markdown（清晰）：
## 会议讨论的问题

1. **预算**：xxx
2. **时间**：xxx
3. **人员**：xxx
```

---

## 一、Markdown基础回顾

### 常用语法一览

```
┌─────────────────────────────────────────────────────────────┐
│ Markdown基础语法速查                                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  # 标题        ## 二级标题        ### 三级标题               │
│  **粗体**      *斜体*            ~~删除线~~                │
│  `行内代码`    ```代码块```                                 │
│  - 无序列表    1. 有序列表    - [ ] 待办事项              │
│  > 引用        | 表格 |                                      │
│  --- 分隔线    [链接](url)   ![图片](url)                  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 基础代码实现

```python
class MarkdownBasic:
    """Markdown基础生成"""

    @staticmethod
    def headings():
        """标题"""
        return """# 一级标题（文章标题）
## 二级标题（大章节）
### 三级标题（小节）
#### 四级标题（子节）
"""

    @staticmethod
    def emphasis():
        """强调"""
        return """**这是粗体** - 用于强调重要内容
*这是斜体* - 用于轻微强调
***粗斜体*** - 用于特别强调
~~删除线~~ - 用于表示过时内容
"""

    @staticmethod
    def inline_code():
        """行内代码"""
        return """Python中用 `print()` 输出
函数调用 `process_data(data)`
变量 `max_tokens = 8000`
"""

    @staticmethod
    def code_block():
        """代码块"""
        return """```python
def hello():
    print("Hello, World!")
```
"""

    @staticmethod
    def lists():
        """列表"""
        return """无序列表：
- 第一项
- 第二项
  - 嵌套的子项
  - 另一个子项
- 第三项

有序列表：
1. 步骤一
2. 步骤二
3. 步骤三

待办事项：
- [x] 已完成的任务
- [ ] 待办的任务
- [ ] 另一个待办
"""

    @staticmethod
    def blockquote():
        """引用"""
        return """> 这是一段引用
> 可以引用名人名言
> 或者重要文档内容
>
> 引用可以多行
"""

    @staticmethod
    def table():
        """表格"""
        return """| 姓名 | 年龄 | 职业 |
|------|------|------|
| 小明 | 25 | 工程师 |
| 小红 | 23 | 设计师 |
"""

    @staticmethod
    def horizontal_rule():
        """分隔线"""
        return """
---
或者
***
或者
___
"""
```

---

## 二、给LLM用的Markdown最佳实践

### 原则1：结构要清晰

**❌ 错误示范：混乱的结构**

```markdown
小明今天去了商店他买了苹果香蕉和橘子然后回家了
```

**✅ 正确示范：清晰的结构**

```markdown
# 小明的一天

## 早上活动
- 起床
- 吃早餐
- 出门

## 购物行程
去了**社区商店**，买了以下物品：
1. 苹果（3个）
2. 香蕉（5根）
3. 橘子（2个）

## 回家
购物完成后，小明直接回家了。
```

### 原则2：用标记突出关键信息

```python
class KeyInformationMarker:
    """关键信息标记"""

    @staticmethod
    def mark_important():
        """标记重要信息"""
        return """## 关键数据

**年增长率**：30%
**核心指标**：DAU > 1000万
**截止日期**：2024-12-31

> [!important]
> 这些数据是决策的重要依据！
"""

    @staticmethod
    def mark_action():
        """标记需要执行的操作"""
        return """## 待执行任务

- [ ] 检查服务器状态
- [ ] 更新数据库
- [ ] 发送通知邮件

> [!action]
> ⚠️ 请在完成任务后更新状态
"""

    @staticmethod
    def mark_warning():
        """标记警告信息"""
        return """> [!warning]
> 性能问题：当前响应时间 > 2秒，需要优化
"""

    @staticmethod
    def mark_tip():
        """标记提示信息"""
        return """> [!tip]
> 💡 提示：使用索引可以提升查询速度10倍
"""
```

### 原则3：分区块组织内容

```python
class ContentBlockOrganizer:
    """内容区块组织器"""

    @staticmethod
    def role_definition():
        """角色定义区块"""
        return """---
角色：你是一个资深Python后端工程师

背景：
- 10年开发经验
- 擅长Django、FastAPI
- 熟悉微服务架构

目标：
- 提供高质量代码
- 解释技术细节
- 给出最佳实践

约束：
- 代码必须可运行
- 必须有注释
- 必须考虑性能
---
"""

    @staticmethod
    def context_block():
        """上下文区块"""
        return """---
[上下文信息]

用户信息：
- 用户名：小明
- 会员等级：VIP
- 偏好：简洁风格

对话历史：
- 之前询问了登录功能
- 需要实现找回密码

当前任务：
实现邮箱找回密码功能
---
"""

    @staticmethod
    def instruction_block():
        """指令区块"""
        return """---
[指令]

请完成以下任务：
1. 创建密码重置API
2. 发送重置邮件
3. 实现密码更新逻辑

输出要求：
- 提供完整代码
- 包含错误处理
- 添加单元测试

格式要求：
- 使用FastAPI
- 返回JSON格式
- 添加文档字符串
---
"""

    @staticmethod
    def example_block():
        """示例区块"""
        return """---
[示例]

输入：
```json
{
  "email": "user@example.com"
}
```

期望输出：
```json
{
  "success": true,
  "message": "重置邮件已发送"
}
```

错误示例：
❌ 返回明文密码
❌ 不验证邮箱存在性
✓ 返回统一格式的错误信息
---
"""
```

### 原则4：代码块要完整

**❌ 错误：残缺代码**

```markdown
def process(data):
    # 省略...
    return result
```

**✅ 正确：完整可运行的代码**

```markdown
```python
from typing import Dict, Any
from dataclasses import dataclass

@dataclass
class ProcessResult:
    """处理结果"""
    success: bool
    data: Any
    error: str = ""

def process(data: Dict[str, Any]) -> ProcessResult:
    """
    处理数据

    Args:
        data: 输入数据字典

    Returns:
        ProcessResult: 处理结果

    Raises:
        ValueError: 数据格式错误
    """
    try:
        # 验证数据
        if not data:
            raise ValueError("数据不能为空")

        # 处理逻辑
        result = data.get("value", 0) * 2

        return ProcessResult(
            success=True,
            data=result
        )
    except ValueError as e:
        return ProcessResult(
            success=False,
            data=None,
            error=str(e)
        )

# 测试
if __name__ == "__main__":
    test_data = {"value": 10}
    result = process(test_data)
    print(f"处理结果：{result}")
```
```

---

## 三、复杂上下文的Markdown组织

### 场景1：多文档汇总

```python
class MultiDocumentOrganizer:
    """多文档组织器"""

    @staticmethod
    def merge_documents(docs: list) -> str:
        """
        合并多个文档

        Args:
            docs: 文档列表，每个元素是 {'title': str, 'content': str, 'source': str}
        """
        output = ["# 文档汇总\n"]

        for i, doc in enumerate(docs, 1):
            output.append(f"## 文档{i}：{doc['title']}")
            output.append(f"**来源**：{doc['source']}")
            output.append("")
            output.append(doc['content'])
            output.append("\n---\n")

        return "\n".join(output)

    @staticmethod
    def structured_merge(docs: list) -> str:
        """结构化合并"""
        output = ["# 资料汇总\n"]

        # 按主题分组
        by_theme = {}
        for doc in docs:
            theme = doc.get('theme', '其他')
            if theme not in by_theme:
                by_theme[theme] = []
            by_theme[theme].append(doc)

        # 输出
        for theme, theme_docs in by_theme.items():
            output.append(f"## {theme}")
            output.append("")

            for doc in theme_docs:
                output.append(f"### {doc['title']}")
                output.append(f"来源：{doc['source']} | 日期：{doc.get('date', '未知')}")
                output.append("")
                output.append(doc['content'])
                output.append("")

        return "\n".join(output)


# 使用示例
def demo_multi_document():
    docs = [
        {
            'title': '产品需求文档',
            'content': '产品需求详细内容...',
            'source': 'PRD_v2.pdf',
            'theme': '产品'
        },
        {
            'title': '技术设计文档',
            'content': '技术架构设计...',
            'source': 'design_doc.md',
            'theme': '技术'
        },
        {
            'title': '用户研究报告',
            'content': '用户调研结果...',
            'source': 'research_2024.pdf',
            'theme': '产品'
        }
    ]

    organizer = MultiDocumentOrganizer()
    merged = organizer.structured_merge(docs)
    print(merged)
```

### 场景2：对话历史格式化

```python
class ConversationFormatter:
    """对话格式化器"""

    @staticmethod
    def format_conversation(messages: list, max_turns: int = 10) -> str:
        """
        格式化对话历史

        Args:
            messages: 消息列表
            max_turns: 最多保留的对话轮数
        """
        output = ["## 对话历史\n"]

        # 保留最近的消息
        recent = messages[-max_turns:] if len(messages) > max_turns else messages

        for i, msg in enumerate(recent, 1):
            role_emoji = {
                'user': '👤',
                'assistant': '🤖',
                'system': '⚙️'
            }.get(msg['role'], '💬')

            output.append(f"### {role_emoji} {msg['role'].upper()} - 轮次{i}")

            # 如果是代码内容，用代码块
            if '```' in msg['content']:
                output.append(msg['content'])
            else:
                output.append(msg['content'])

            output.append("")

        return "\n".join(output)

    @staticmethod
    def format_with_summary(messages: list) -> str:
        """带摘要的对话格式化"""
        if len(messages) <= 10:
            return ConversationFormatter.format_conversation(messages)

        output = ["## 对话摘要\n"]

        # 历史摘要
        output.append("### 早期对话摘要")
        output.append("[由LLM生成的早期对话摘要]")
        output.append("")

        # 最近的对话
        output.append("### 最近对话")
        output.append(ConversationFormatter.format_conversation(
            messages[-5:],  # 只保留最近5轮
            max_turns=5
        ))

        return "\n".join(output)
```

### 场景3：数据表格化

```python
class TableFormatter:
    """表格格式化器"""

    @staticmethod
    def create_comparison_table(items: list, headers: list) -> str:
        """创建对比表格"""
        lines = []

        # 表头
        header_line = "| " + " | ".join(headers) + " |"
        separator = "|" + "|".join(["---" for _ in headers]) + "|"

        lines.append(header_line)
        lines.append(separator)

        # 数据行
        for item in items:
            row = "| " + " | ".join(str(item.get(h, "")) for h in headers) + " |"
            lines.append(row)

        return "\n".join(lines)

    @staticmethod
    def create_feature_matrix(features: dict) -> str:
        """创建特性矩阵"""
        lines = ["## 特性对比\n"]

        # 表头
        headers = ["特性", "实现状态", "优先级", "备注"]
        lines.append(TableFormatter.create_comparison_table([], headers))

        # 数据
        items = []
        for name, info in features.items():
            items.append({
                "特性": name,
                "实现状态": "✅ 已完成" if info.get('done') else "⏳ 进行中",
                "优先级": info.get('priority', '中'),
                "备注": info.get('note', '-')
            })

        if items:
            lines.append(TableFormatter.create_comparison_table(items, headers))

        return "\n".join(lines)

    @staticmethod
    def create_progress_table(tasks: list) -> str:
        """创建进度表格"""
        lines = ["## 任务进度\n"]

        headers = ["任务", "负责人", "状态", "完成度"]
        items = []

        for task in tasks:
            status_map = {
                'todo': '📋 待开始',
                'in_progress': '🔄 进行中',
                'review': '👀 审核中',
                'done': '✅ 已完成',
                'blocked': '🚫 阻塞'
            }

            items.append({
                "任务": task['name'],
                "负责人": task.get('owner', '-'),
                "状态": status_map.get(task.get('status', 'todo'), '📋 未知'),
                "完成度": f"{task.get('progress', 0)}%"
            })

        if items:
            lines.append(TableFormatter.create_comparison_table(items, headers))

        return "\n".join(lines)
```

---

## 四、Markdown在Prompt中的高级用法

### 技巧1：Chain-of-Thought分步

```python
class CoTPromptFormatter:
    """思维链提示格式化"""

    @staticmethod
    def step_by_step() -> str:
        """分步思考模板"""
        return """## 问题分析

请按以下步骤分析并回答问题：

### 步骤1：理解问题
- 识别问题的核心是什么
- 确定需要的信息

### 步骤2：分析信息
- 整理已知条件
- 找出关键数据

### 步骤3：推理过程
请展示你的推理步骤：

```
步骤3.1：...
步骤3.2：...
步骤3.3：...
```

### 步骤4：得出结论
基于以上分析，结论是：...

### 步骤5：验证
- 检查逻辑是否正确
- 确认结论是否合理

---
问题：{question}

你的分析：
"""

    @staticmethod
    def with_examples() -> str:
        """带示例的思考模板"""
        return """## 任务：{task_name}

### 示例

**示例输入**：{example_input}

**推理过程**：
```
1. 识别输入类型：...
2. 应用规则：...
3. 生成输出：...
```

**示例输出**：
```json
{expected_output}
```

### 实际任务

**输入**：{actual_input}

请按照示例的格式完成推理并输出结果：
"""
```

### 技巧2：角色扮演模板

```python
class RolePlayFormatter:
    """角色扮演格式化器"""

    @staticmethod
    def define_role(
        name: str,
        background: str,
        expertise: list,
        constraints: list
    ) -> str:
        """定义角色"""
        output = [f"## 角色定义：{name}\n"]

        output.append("### 背景")
        output.append(background)
        output.append("")

        output.append("### 专业领域")
        for exp in expertise:
            output.append(f"- {exp}")
        output.append("")

        output.append("### 行为约束")
        for constraint in constraints:
            output.append(f"- {constraint}")
        output.append("")

        output.append("---\n")
        output.append("现在，请以这个角色的身份回答问题。")

        return "\n".join(output)

    @staticmethod
    def multi_agent() -> str:
        """多智能体模板"""
        return """# 团队讨论

## 参与者

### 🎯 产品经理
- 关注：用户需求、产品价值、商业目标
- 发言风格：简洁、直接、关注ROI

### 💻 技术专家
- 关注：可行性、技术架构、性能
- 发言风格：详细、专业、数据驱动

### 📊 数据分析师
- 关注：数据指标、趋势分析、洞察
- 发言风格：客观、谨慎、引用数据

### 🎨 设计师
- 关注：用户体验、界面美观、一致性
- 发言风格：创意、以用户为中心

---
## 议题

{topic}

---
## 讨论规则

1. 每位参与者先陈述自己的观点
2. 然后回应其他人的观点
3. 最后达成共识或保留分歧

开始讨论：
"""
```

### 技巧3：输出格式强制

```python
class OutputFormatFormatter:
    """输出格式格式化器"""

    @staticmethod
    def json_format(schema: dict) -> str:
        """JSON格式模板"""
        return f"""## 输出要求

请以JSON格式输出，结构如下：

```json
{schema}
```

### 示例

```json
{json.dumps({
    "status": "success",
    "data": {{"key": "value"}},
    "message": "操作完成"
}, ensure_ascii=False, indent=2)}
```

### 注意事项
- 所有字段必须存在
- 字段类型必须正确
- 不要添加额外字段

---
你的回答（JSON格式）：
"""

    @staticmethod
    def structured_report() -> str:
        """结构化报告模板"""
        return """## 分析报告

### 1. 执行摘要
（用2-3句话概括核心发现）

### 2. 详细分析

#### 2.1 关键发现
- 发现1：[描述] | 影响：[评估]
- 发现2：[描述] | 影响：[评估]

#### 2.2 数据支持
| 指标 | 数值 | 趋势 |
|------|------|------|
| ... | ... | ... |

### 3. 结论
（基于分析得出的结论）

### 4. 建议
基于以上分析，建议：
1. ...
2. ...
3. ...

### 5. 风险提示
- 风险1：[描述] | 概率：[评估] | 影响：[评估]
- 风险2：[描述] | 概率：[评估] | 影响：[评估]

---
"""
```

---

## 五、常见"坑"与解决方案

### 坑1：Markdown渲染不一致

```python
class MarkdownCompatibility:
    """Markdown兼容性处理"""

    # 不同平台支持的语法
    SUPPORTED_FEATURES = {
        'chatgpt': ['headings', 'bold', 'italic', 'code', 'lists', 'blockquote'],
        'claude': ['headings', 'bold', 'italic', 'code', 'lists', 'blockquote', 'callouts'],
        'gemini': ['headings', 'bold', 'italic', 'code', 'lists'],
        'kimi': ['headings', 'bold', 'italic', 'code', 'lists', 'blockquote']
    }

    @staticmethod
    def make_compatible(text: str, platform: str) -> str:
        """转换为平台兼容的Markdown"""
        supported = MarkdownCompatibility.SUPPORTED_FEATURES.get(
            platform,
            MarkdownCompatibility.SUPPORTED_FEATURES['chatgpt']
        )

        # 移除不支持的特性
        if 'callouts' not in supported:
            # 移除callout语法
            text = MarkdownCompatibility._remove_callouts(text)

        if 'blockquote' not in supported:
            # 替换blockquote为普通文本
            text = MarkdownCompatibility._replace_blockquote(text)

        return text

    @staticmethod
    def _remove_callouts(text: str) -> str:
        """移除callout"""
        import re
        # 移除 > [!type] 这样的标记
        return re.sub(r'\[![\w]+\]', '', text)

    @staticmethod
    def _replace_blockquote(text: str) -> str:
        """替换blockquote"""
        lines = text.split('\n')
        result = []

        for line in lines:
            if line.startswith('>'):
                line = '【提示】' + line.lstrip('>').strip()
            result.append(line)

        return '\n'.join(result)
```

### 坑2：表格太复杂AI解析错

```python
class TableSimplifier:
    """表格简化器"""

    @staticmethod
    def simplify_table(table_text: str) -> str:
        """简化复杂表格"""
        lines = table_text.strip().split('\n')

        # 解析表头和数据
        header_line = None
        data_lines = []

        for line in lines:
            if '|' in line and not line.strip().startswith('|---'):
                if header_line is None:
                    header_line = line
                else:
                    data_lines.append(line)

        if not header_line:
            return table_text

        # 简化：减少列数，保留关键列
        headers = [h.strip() for h in header_line.split('|') if h.strip()]

        # 如果列太多，转换为列表形式
        if len(headers) > 5:
            return TableSimplifier._to_list_format(headers, data_lines)

        return table_text

    @staticmethod
    def _to_list_format(headers: list, data_lines: list) -> str:
        """转换为列表格式"""
        output = []

        for i, line in enumerate(data_lines):
            cells = [c.strip() for c in line.split('|') if c.strip()]
            output.append(f"### 项目{i+1}")

            for j, header in enumerate(headers):
                if j < len(cells):
                    output.append(f"- **{header}**：{cells[j]}")
            output.append("")

        return '\n'.join(output)
```

### 坑3：代码块嵌套混乱

```python
class CodeBlockFixer:
    """代码块修复器"""

    @staticmethod
    def fix_nested_code(text: str) -> str:
        """修复嵌套的代码块"""
        import re

        # 方案1：使用不同语言标记
        # 方案2：转义内层代码块

        lines = text.split('\n')
        result = []
        in_code_block = False
        code_block_start = None

        for line in lines:
            # 检测代码块开始
            if line.strip().startswith('```'):
                if not in_code_block:
                    in_code_block = True
                    code_block_start = line
                    result.append(line)
                else:
                    # 内层代码块 - 需要转义
                    # 检查语言是否不同
                    if line != code_block_start:
                        # 暂时结束外层，插入转义的代码
                        result.append(code_block_start.replace('```', '~~~'))
                        result.append(line.replace('```', '~~~'))
                        result.append(code_block_start.replace('```', '~~~'))
                        result.append(line)
                    else:
                        result.append(line)
                        in_code_block = False
                        code_block_start = None
            else:
                result.append(line)

        return '\n'.join(result)

    @staticmethod
    def avoid_nesting(text: str) -> str:
        """避免嵌套：改用其他方式"""
        # 方案1：用行内代码代替
        # 方案2：用引用代替
        # 方案3：拆分成多个独立块

        lines = text.split('\n')
        result = []

        in_code_block = False
        buffer = []

        for line in lines:
            if line.strip().startswith('```'):
                if not in_code_block:
                    in_code_block = True
                    result.extend(buffer)
                    buffer = []
                    result.append(line)
                else:
                    # 代码块结束
                    result.append(line)
                    in_code_block = False
            elif '`' in line and not in_code_block:
                # 包含行内代码，整行保留
                buffer.append(line)
            else:
                buffer.append(line)

        result.extend(buffer)
        return '\n'.join(result)
```

---

## 六、生产级Markdown生成器

```python
class ProductionMarkdownGenerator:
    """生产级Markdown生成器"""

    def __init__(self):
        self.sections = []

    def add_heading(self, text: str, level: int = 1):
        """添加标题"""
        prefix = '#' * level
        self.sections.append(f"{prefix} {text}\n")
        return self

    def add_paragraph(self, text: str):
        """添加段落"""
        self.sections.append(f"{text}\n")
        return self

    def add_bullet_list(self, items: list, ordered: bool = False):
        """添加列表"""
        for i, item in enumerate(items, 1):
            prefix = f"{i}." if ordered else "-"
            self.sections.append(f"{prefix} {item}\n")
        return self

    def add_code_block(self, code: str, language: str = ""):
        """添加代码块"""
        self.sections.append(f"```{language}\n{code}\n```\n")
        return self

    def add_callout(self, text: str, callout_type: str = "note"):
        """添加标注"""
        self.sections.append(f"> [!{callout_type}]\n> {text}\n")
        return self

    def add_table(self, headers: list, rows: list):
        """添加表格"""
        # 表头
        self.sections.append("| " + " | ".join(headers) + " |\n")
        # 分隔线
        self.sections.append("|" + "|".join(["---"] * len(headers)) + "|\n")
        # 数据行
        for row in rows:
            self.sections.append("| " + " | ".join(str(c) for c in row) + " |\n")
        return self

    def add_divider(self):
        """添加分隔线"""
        self.sections.append("---\n")
        return self

    def add_section(self, title: str, content: str):
        """添加区块"""
        self.sections.append(f"## {title}\n")
        self.sections.append(f"{content}\n")
        return self

    def build(self) -> str:
        """生成最终Markdown"""
        return ''.join(self.sections)


# 使用示例
def demo_markdown_generator():
    """演示Markdown生成器"""
    md = ProductionMarkdownGenerator()

    result = md.add_heading("项目报告", 1) \
               .add_divider() \
               .add_section("项目概述", "这是一个示例项目") \
               .add_heading("关键指标", 2) \
               .add_table(
                   ["指标", "数值", "状态"],
                   [
                       ["用户数", "10,000", "✅ 达标"],
                       ["转化率", "5.2%", "⚠️ 略低"],
                       ["响应时间", "120ms", "✅ 优秀"]
                   ]
               ) \
               .add_heading("待办事项", 2) \
               .add_bullet_list([
                   "优化数据库查询",
                   "增加缓存层",
                   "完善单元测试"
               ]) \
               .add_heading("代码示例", 2) \
               .add_code_block("print('Hello, World!')", "python") \
               .add_callout("注意：以上代码仅供参考", "warning") \
               .build()

    print(result)


# 直接使用示例
def demo_direct_usage():
    """直接使用Markdown"""

    # 生成提示词
    prompt = """# 技术方案评审

## 方案概述
请评审以下技术方案：

## 方案详情

### 架构设计
```
┌─────────────┐     ┌─────────────┐
│   前端      │────▶│   API网关   │
└─────────────┘     └─────────────┘
                          │
                          ▼
                   ┌─────────────┐
                   │   服务层    │
                   └─────────────┘
```

### 技术栈
| 组件 | 技术选型 | 理由 |
|------|----------|------|
| 前端 | React | 生态完善 |
| 后端 | Go | 性能优秀 |
| 数据库 | PostgreSQL | 可靠 |

### 性能指标
- **QPS**: 10,000+
- **延迟**: < 50ms
- **可用性**: 99.9%

## 评审要点

请重点关注：
1. 架构是否合理？
2. 技术选型是否合适？
3. 性能指标能否达标？

> [!question]
> 需要特别关注数据库扩展性问题
"""

    print(prompt)
```

---

## 七、总结

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Markdown最佳实践速查表                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  🎯 核心原则                                                         │
│  ├─ 结构清晰：善用标题、列表、表格                                      │
│  ├─ 突出重点：用粗体、标注、颜色                                       │
│  ├─ 区块分明：用分隔线区分不同内容                                     │
│  └─ 代码完整：确保代码块可运行                                         │
│                                                                      │
│  📊 常用模板                                                         │
│  ├─ 角色定义：背景 + 专业领域 + 约束                                  │
│  ├─ 任务说明：任务 + 要求 + 示例                                      │
│  ├─ 输出格式：格式 + 示例 + 注意事项                                  │
│  └─ 分步思考：步骤 + 推理 + 结论                                      │
│                                                                      │
│  ⚠️ 常见问题                                                         │
│  ├─ 平台兼容性：不同LLM支持不同语法                                   │
│  ├─ 表格复杂：超过5列改用列表                                          │
│  ├─ 代码嵌套：用语言区分或拆分                                        │
│  └─ 过度格式：简洁比花哨更重要                                        │
│                                                                      │
│  💡 进阶技巧                                                         │
│  ├─ Obsidian Callout：平台特有的标注语法                             │
│  ├─ Mermaid图：用文字画图                                            │
│  └─ 变量模板：用{placeholder}标记待填内容                             │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 相关主题

- [[分隔符与标记系统]] - Markdown中的分隔符使用
- [[上下文结构化]] - 整体上下文组织策略
- [[Prompt工程基础]] - Markdown在Prompt中的应用

---

## 参考文献

1. Gruber, J. (2004). **Markdown Syntax Documentation.**
2. CommonMark. **A Common Markdown Specification.**
3. Obsidian. **Obsidian Formatting Syntax.**
