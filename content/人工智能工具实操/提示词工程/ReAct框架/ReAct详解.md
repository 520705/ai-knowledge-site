---
title: ReAct详解
date: 2026-04-18
tags:
  - 提示词工程
  - ReAct
  - 推理行动
  - 工具使用
  - Agent
  - 自主智能体
  - 交互式推理
categories:
  - 人工智能/提示词工程/推理技术
keywords:
  - ReAct
  - Synergizing Reasoning
  - Acting
  - 推理行动
  - 工具调用
  - LLM Agent
  - 自主智能体
  - 交互式推理
  - 外部工具
  - 动态规划
---

# ReAct详解

> **关键词**：ReAct、Synergizing Reasoning、Acting、推理行动、工具调用、LLM Agent、自主智能体、交互式推理、外部工具、动态规划

## 一、ReAct框架概述

ReAct（**Re**asoning + **Act**ing，推理与行动协同）是由Google研究院和普林斯顿大学的研究者于2023年提出的一个创新性提示词框架。与传统的纯推理（Reasoning Only）或纯行动（Acting Only）方法不同，ReAct通过将语言模型的推理能力和行动能力进行有机整合，实现了"边想边做，边做边想"的交互式问题解决范式。

ReAct的核心理念源自一个深刻的观察：人类的智能行为往往是推理和行动的交织过程。当我们解决复杂问题时，会先进行推理以理解问题和制定计划，然后采取行动来验证假设或获取信息，再基于行动结果进行下一轮推理。这种推理-行动-推理的循环模式比单纯的思考或行动都更加高效和可靠。

> [!important] ReAct的核心创新
> ReAct不仅仅是"先推理再行动"，而是让推理和行动在一个循环中相互增强：推理指导行动，行动结果反馈给推理，形成一个不断迭代优化的动态过程。

## 二、核心原理与工作机制

### 2.1 推理-行动循环

ReAct的基本工作流程可以描述为一个持续迭代的循环：

```
用户问题 → 推理(Thought) → 行动(Action) → 观察(Observation) → 推理(Thought) → ...
```

每个循环包含三个核心环节：

| 环节 | 含义 | 作用 |
|------|------|------|
| Thought（思考） | 对当前状态的分析和推理 | 理解问题、制定下一步计划 |
| Action（行动） | 执行的具体操作 | 获取信息、操作外部系统 |
| Observation（观察） | 行动的结果反馈 | 验证假设、更新状态 |

### 2.2 Thought的设计

Thought是ReAct中的推理环节，它需要完成以下任务：

1. **问题理解**：分析当前状态和目标
2. **知识检索**：决定需要什么信息
3. **计划制定**：决定下一步应该采取什么行动
4. **假设验证**：评估已有信息是否足够

```markdown
# Thought示例

## Thought 1
问题分析：
- 用户询问的是特斯拉股票的目标价
- 我需要获取最新的分析师预测数据
- 我需要了解影响股价的关键因素

计划：
- 使用搜索工具查询最新分析师报告
- 使用计算工具进行合理估值
```

### 2.3 Action的设计

Action是ReAct中的行动环节，它包括但不限于：

| 行动类型 | 描述 | 示例 |
|----------|------|------|
| Search | 搜索外部信息 | 搜索"特斯拉分析师评级" |
| Retrieve | 从知识库获取信息 | 检索"财务分析知识库" |
| Calculate | 执行计算 | 计算"净现值"或"增长率" |
| Read | 读取文件或网页 | 读取PDF报告或网页内容 |
| Query | 查询数据库 | 执行SQL查询 |
| Generate | 生成内容 | 生成报告或代码 |

```python
# Action定义示例

actions = {
    "search": {
        "description": "搜索网络获取实时信息",
        "parameters": {"query": "搜索关键词"}
    },
    "calculate": {
        "description": "执行数学计算",
        "parameters": {"expression": "计算表达式"}
    },
    "retrieve": {
        "description": "从知识库检索信息",
        "parameters": {"topic": "检索主题"}
    },
    "finish": {
        "description": "完成任务，输出最终答案",
        "parameters": {"answer": "最终答案"}
    }
}
```

### 2.4 Observation的作用

Observation是行动结果的反馈，它：

1. 提供行动执行的状态信息
2. 返回查询或计算的结果
3. 指示是否有错误或异常
4. 更新Agent的内部状态

```markdown
## Observation
搜索结果返回：
- 高盛分析师：目标价$280，维持买入评级
- 摩根士丹利分析师：目标价$260，维持增持评级
- 15位分析师平均目标价：$275

这些数据与当前股价$210相比，上涨空间约31%。
```

## 三、ReAct提示词模板

### 3.1 基础模板

```markdown
# ReAct提示词基础模板

你是一个智能助手，可以推理问题和执行行动来完成任务。

## 可用工具
- search(query): 搜索网络信息
- retrieve(topic): 从知识库检索信息
- calculate(expression): 执行数学计算
- finish(answer): 完成回答

## 工作流程
对于每个问题，你将按照以下格式进行推理和行动：

问题：{用户问题}

Thought 1: [你的第一个思考，分析问题和制定计划]
Action 1: [你选择执行的行动]
Observation 1: [行动的结果]
Thought 2: [基于观察结果的思考]
Action 2: [你的下一个行动]
...

请用中文回答。

问题：{用户问题}
```

### 3.2 完整示例

```markdown
# ReAct完整示例

## 系统提示
你是一个财务分析助手，使用ReAct框架进行系统性分析。

## 可用工具
- search(query): 搜索最新财务新闻和分析师报告
- retrieve(topic): 检索财务分析方法
- calculate(expression): 执行财务计算
- retrieve_financials(symbol): 获取公司财务数据
- finish(answer): 完成分析并给出建议

## 行为准则
1. 每一步行动都要有清晰的推理
2. 优先使用搜索和检索获取一手信息
3. 计算时确保数据准确
4. 最终建议要有充分依据

## 示例格式
问题：{问题描述}

Thought: [思考过程]
Action: [行动名称(参数)]
Observation: [行动结果]
...

---

问题：分析苹果公司(AAPL)的投资价值，给出目标价预测。
```

### 3.3 增强模板

```markdown
# ReAct增强模板

## 角色定义
你是一位资深投资分析师，擅长系统性研究和量化分析。

## 能力边界
- ✅ 可以使用外部工具获取实时信息
- ✅ 可以执行复杂的财务计算
- ✅ 可以进行多维度投资分析
- ❌ 不能提供确定的投资建议
- ❌ 不能预测股价短期波动

## 推理框架
每次推理必须包含：
1. **问题分解**：将复杂问题分解为可执行的子问题
2. **信息缺口**：明确还需要什么信息
3. **行动计划**：制定获取信息的步骤
4. **风险提示**：识别分析中的不确定性

## 输出规范
最终输出必须包含：
- 分析摘要（200字以内）
- 关键发现（3-5点）
- 估值区间
- 主要风险
- 免责声明
```

## 四、ReAct与工具集成

### 4.1 工具类型与集成

ReAct框架可以集成的工具类型：

#### 信息获取工具

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| Web Search | 搜索网络信息 | 实时新闻、研究报告 |
| Knowledge Base | 检索结构化知识 | 专业知识问答 |
| Database | 查询数据库 | 结构化数据分析 |
| API | 调用外部服务 | 实时数据、业务集成 |

#### 信息处理工具

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| Calculator | 数学计算 | 财务分析、统计分析 |
| Code Executor | 执行代码 | 数据处理、模型运行 |
| Parser | 解析文档 | 提取非结构化数据 |
| Formatter | 格式化输出 | 生成报告、图表 |

#### 行动执行工具

| 工具 | 功能 | 适用场景 |
|------|------|----------|
| Email | 发送邮件 | 通知、报告发送 |
| Calendar | 管理日程 | 会议安排、提醒 |
| File System | 文件操作 | 文档管理、存储 |

### 4.2 工具调用格式

```markdown
# 工具调用格式

## 格式说明
Action: tool_name
Action Input: {"param1": "value1", "param2": "value2"}

## 示例
Action: search
Action Input: {"query": "特斯拉2024年Q4财报要点"}

Action: calculate
Action Input: {"expression": "(85000000000 - 78000000000) / 78000000000 * 100"}

Action: retrieve_financials
Action Input: {"symbol": "AAPL", "period": "annual"}
```

### 4.3 多工具协同

```markdown
# 多工具协同示例

Thought: 用户需要全面的市场分析报告。我需要：
1. 搜索最新的市场数据
2. 获取公司财务数据
3. 执行财务分析计算
4. 整合信息生成报告

Action: search
Action Input: {"query": "2024年新能源汽车市场分析报告"}

Observation: 搜索返回5篇相关报告，主要数据：
- 全球新能源汽车销量同比增长35%
- 中国市场占全球60%份额
- 预计2025年渗透率将达到30%

Thought: 获取了市场概览，现在需要获取具体公司的财务数据。

Action: retrieve_financials
Action Input: {"symbol": "TSLA", "period": "quarterly", "metrics": ["revenue", "margin", "guidance"]}

Observation: 特斯拉2024年Q3财务数据：
- 营收：251亿美元（YoY +8%）
- 毛利率：19.2%
- 指引：预计全年交付量增长20%

Thought: 现在我有了市场数据和公司财务数据，可以进行综合分析了。

Action: calculate
Action Input: {"expression": "calculate_valuation(revenue=251e8, growth=0.08, pe=45)"}

Observation: 基于DCF模型的估值结果：目标价区间$280-$320

...
```

## 五、ReAct高级应用

### 5.1 复杂推理链设计

```markdown
# 复杂推理链示例

## 任务：竞争对手分析

Thought 1: 我需要进行全面的竞争对手分析。首先需要确定主要竞争对手。
Action 1: search(query="电动汽车市场主要竞争对手 2024")
Observation 1: 主要竞争对手包括：
- 比亚迪（全球销量第一）
- 大众ID系列
- 现代起亚Ioniq系列
- Rivian、Lucid等新势力

Thought 2: 好的，已识别主要竞争对手。现在需要获取各公司的市场份额数据。
Action 2: retrieve(topic="全球电动汽车市场份额 2024")
Observation 2: 2024年Q3全球市场份额：
- 特斯拉：16%
- 比亚迪：22%
- 大众：8%
- 其他：54%

Thought 3: 数据表明比亚迪已超越特斯拉成为全球第一。现在分析各公司的核心竞争力。
Action 3: [继续深入分析]

...

Thought N: 综合以上分析，生成最终报告。
Action N: finish(answer=final_report)
```

### 5.2 自我纠错机制

```markdown
# 自我纠错示例

Thought 1: 用户问的是特斯拉的竞争优势。我将搜索相关信息。
Action 1: search(query="特斯拉竞争优势 2024")
Observation 1: 返回结果包含一些过时信息（2022年数据）。

Thought 2: 搜索结果中包含了过时数据，需要更精确的时间筛选。
Action 2: search(query="特斯拉竞争优势 最新 2024 2025")
Observation 2: 获取到最新的竞争优势分析。

Thought 3: 现在有了最新数据，可以进行分析了。
[继续分析流程]

---

# 错误处理示例

Thought: 我需要计算增长率，但提供的数据似乎不完整。
Action: retrieve_financials
Action Input: {"symbol": "AAPL", "period": "annual", "metrics": ["revenue"]}

Observation: 返回数据缺少2020年和2021年的数据。

Thought: 数据存在缺口，这会影响计算的准确性。我需要：
1. 标记这个数据缺口
2. 使用可用的年份数据计算
3. 在报告中说明限制

Action: calculate
Action Input: {"expression": "calculate_growth(available_data)", "note": "基于2022-2024数据计算"}
...
```

### 5.3 多轮对话管理

```markdown
# 多轮对话ReAct模板

## 对话历史管理
保留对话历史用于上下文理解：
- 用户的原始问题
- 每轮的Thought、Action、Observation
- 已获取的关键信息摘要

## 状态跟踪
维护当前分析状态：
- 已完成的信息收集
- 正在进行中的分析
- 待完成的任务

## 增量更新
当用户提供额外信息时：
1. 评估新信息的相关性
2. 判断是否需要调整分析方向
3. 决定是否重新执行某些步骤

## 格式
---
[用户]: 追加问题/补充信息

[Agent]:
Thought: [如何处理新信息]
Action: [需要的行动]
Observation: [结果]
...
```

## 六、ReAct变体与扩展

### 6.1 ReAct-Perceiver

ReAct-Perceiver扩展了原始ReAct，增加了感知能力：

```markdown
# ReAct-Perceiver 扩展

## 新增组件
- **Perceive**：感知环境变化
- **Update**：更新内部状态
- **Reflect**：反思已执行的操作

## 流程
Thought → Action → Perceive → Observation → Reflect → Update → Next Thought
```

### 6.2 ReAct-Self-Correct

ReAct-Self-Correct增加了自我纠错能力：

```markdown
# ReAct-Self-Correct 流程

## 检查点机制
在每个重要步骤后进行自我检查：
1. 逻辑一致性检查
2. 数据准确性检查
3. 假设合理性检查

## 纠错触发条件
- 发现逻辑矛盾
- 数据与预期不符
- 收到外部反馈

## 纠错流程
发现问题 → 回溯分析 → 修正行动 → 继续执行
```

### 6.3 ReAct-Plan

ReAct-Plan增强了规划能力：

```markdown
# ReAct-Plan 增强

## 预规划阶段
在执行行动前，先生成完整计划：

Thought: 用户需要完整的投资分析报告。
Plan: 
1. 市场调研 → 2. 财务分析 → 3. 估值计算 → 4. 风险评估 → 5. 报告生成

## 执行监控
在执行过程中监控计划执行情况：
- 任务完成度
- 时间消耗
- 偏差修正
```

## 七、ReAct与CoT/ToT对比

### 7.1 核心差异

| 维度 | CoT | ToT | ReAct |
|------|-----|-----|-------|
| 推理方式 | 线性 | 树状 | 循环 |
| 行动能力 | 无 | 无 | 有 |
| 外部交互 | 无 | 无 | 有 |
| 问题类型 | 封闭问题 | 探索问题 | 交互问题 |
| 信息获取 | 内部知识 | 内部知识 | 外部工具 |
| 适用场景 | 数学推理 | 创意生成 | 动态任务 |

### 7.2 选择决策树

```
任务类型 → 推荐方法
│
├─ 需要精确推理 → CoT
│   └─ 简单数学、逻辑题
│
├─ 需要探索多种可能 → ToT
│   └─ 创意写作、战略规划
│
├─ 需要外部信息/工具 → ReAct
│   └─ 研究分析、实时问答
│
├─ 需要动态交互 → ReAct
│   └─ 客服、数据查询
│
└─ 复杂综合任务 → ReAct + CoT/ToT
    └─ 先用ToT规划，再用ReAct执行
```

### 7.3 组合策略

#### ReAct + CoT

```markdown
# 组合模式：ReAct + CoT

Thought: [使用CoT进行深度推理]
Action: [执行获取信息]
Observation: [CoT验证推理结果]
...
```

#### ReAct + ToT

```markdown
# 组合模式：ReAct + ToT

阶段1（ToT规划）：
- 生成多个可能的分析路径
- 评估并选择最佳路径

阶段2（ReAct执行）：
- 按照选定路径执行
- 动态调整计划
```

## 八、实战案例

### 案例一：研究性问答

```markdown
问题：特斯拉的自动驾驶技术领先竞争对手多长时间？

Thought 1: 用户想知道特斯拉自动驾驶技术的相对优势。这需要：
1. 了解特斯拉FSD的技术进展
2. 了解主要竞争对手的技术状态
3. 进行对比分析

Action 1: search(query="特斯拉FSD最新技术进展 2025")
Observation 1: 特斯拉FSD V13版本已实现端到端神经网络架构，
在城市道路驾驶中表现显著提升。

Thought 2: 了解了特斯拉的技术进展，现在需要了解竞争对手的情况。

Action 2: search(query="Waymo通用自动驾驶技术进展 2025")
Observation 2: Waymo在特定区域表现优秀，但依赖高清地图，
扩展性受限。

Action 3: search(query="华为自动驾驶技术进展 2025")
Observation 3: 华为ADS 3.0已在中国市场部署，
在某些场景表现接近特斯拉。

Thought 3: 现在有了足够的信息进行综合分析。

Action 4: calculate
Action Input: {"analysis": "技术对比分析"}
Observation 4: 分析结果：
- 特斯拉：领先约1-2年（端到端架构）
- 华为：快速追赶，差距缩小
- Waymo：特定场景领先但扩展性差

Thought 4: 可以给出综合分析了。

Action 5: finish
Action Input: {"answer": "特斯拉自动驾驶技术领先竞争对手约1-2年..."}
```

### 案例二：数据分析任务

```markdown
问题：分析苹果公司过去5年的营收增长趋势，并预测未来3年走势。

Thought 1: 这是一个财务分析任务，需要：
1. 获取苹果5年财务数据
2. 分析增长趋势
3. 建立预测模型
4. 给出预测结果

Action 1: retrieve_financials
Action Input: {"symbol": "AAPL", "period": "annual", "years": 5, "metrics": ["revenue", "growth"]}
Observation 1: 苹果2019-2024营收数据：
- 2019: $2602亿
- 2020: $2745亿 (+5.5%)
- 2021: $3658亿 (+33.2%)
- 2022: $3943亿 (+7.8%)
- 2023: $3833亿 (-2.8%)
- 2024: $3910亿 (+2.0%)

Thought 2: 数据获取成功。现在分析增长模式并建立预测模型。

Action 2: calculate
Action Input: {"expression": "calculate_cagr(data=[2602,2745,3658,3943,3833,3910], years=5)"}
Observation 2: 5年复合增长率(CAGR): 8.5%

Action 3: calculate
Action Input: {"expression": "predict_revenue(historical=data, years=3, model='linear')"}
Observation 3: 未来3年预测：
- 2025: $4100亿 (+4.9%)
- 2026: $4300亿 (+4.9%)
- 2027: $4500亿 (+4.7%)

...

Action N: finish
Action Input: {"answer": "苹果营收分析及预测报告..."}
```

## 九、最佳实践与注意事项

### 9.1 设计原则

> [!tip] ReAct设计最佳实践
> 1. **清晰的Thought**：每一步推理都要有明确的逻辑
> 2. **合理的Action**：行动要与推理一致
> 3. **有效的Observation**：结果要能被正确解读
> 4. **明确的终止**：定义何时应该结束循环
> 5. **错误处理**：考虑失败情况的处理方式

### 9.2 常见问题与解决

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 循环过长 | 推理不充分 | 增加推理深度，减少行动次数 |
| 行动失败 | 工具调用错误 | 添加错误处理和重试机制 |
| 信息不足 | 搜索不准确 | 优化搜索策略，增加信息源 |
| 逻辑矛盾 | 推理跳跃 | 增加中间推理步骤 |

### 9.3 性能优化

1. **减少不必要的行动**：在Thought中充分分析
2. **批量获取信息**：尽量一次获取完整信息
3. **缓存中间结果**：避免重复获取相同信息
4. **设置超时机制**：防止无限循环

## 相关主题

- [[链式思考CoT]] - 纯推理方法
- [[思维树ToT]] - 树状探索方法
- [[CoT进阶技术]] - Self-Ask等进阶技术
- [[自洽性Self-Consistency]] - 多数投票机制
