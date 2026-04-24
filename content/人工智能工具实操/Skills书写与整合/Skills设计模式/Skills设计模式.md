---
title: Skills设计模式 - 如何组织多个Skill的艺术
date: 2026-04-18
tags:
  - skills
  - 设计模式
  - 软件架构
  - 模式识别
  - 进阶教程
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills设计模式
alias: [Skills设计模式, Skills Patterns, 技能设计模式, 如何组织多个Skill]
description: 详细讲解各种Skill设计模式，帮助你构建复杂的AI应用
---

# Skills设计模式：像搭积木一样构建AI能力

> 写一个Skill不难，但当你要管理10个、100个Skill的时候，问题就来了：
> - 这些Skill之间是什么关系？
> - 怎么让它们配合工作？
> - 怎么避免重复代码？
> - 怎么让系统更容易扩展？
>
> 这就是"设计模式"要解决的问题。

---

## 0. 先来理解：什么是设计模式

### 0.1 从生活理解设计模式

假设你在装修一套房子，需要买家具。你有两个选择：

**选择一：全部买成品家具**
- 优点：直接可用
- 缺点：尺寸可能不合适，功能固定

**选择二：买宜家家具，自己组装**
- 优点：灵活搭配，按需组合
- 缺点：需要了解组装规则

设计模式就像是"组装家具的说明书"——告诉你什么零件能拼在一起、怎么拼。

### 0.2 为什么Skills需要设计模式

单个Skill很简单，多个Skill放在一起就复杂了：

| 问题 | 没有设计模式 | 有设计模式 |
|------|-------------|-----------|
| Skill太多难管理 | 乱成一团 | 结构清晰 |
| 能力难以复用 | 每个Skill自己写一遍 | 复用基础能力 |
| 扩展困难 | 牵一发动全身 | 模块化，易扩展 |
| 新人上手慢 | 不知道从哪开始 | 有固定模式可循 |

---

## 1. 单例模式：一个系统只需要一个实例

### 1.1 什么时候用

当你的Skill需要：
- 维护全局唯一的状态
- 管理共享资源
- 提供全局访问点

**典型场景**：
- 日志记录器（全局统一记录日志）
- 配置管理器（全局统一读取配置）
- 缓存管理器（全局统一管理缓存）
- 认证服务（全局统一验证身份）

### 1.2 怎么实现

```yaml
# SKILL.global-logger.md
---
name: global-logger
description: 全局日志记录器
version: 1.0.0
singleton: true              # 标记为单例
instance_id: system-logger  # 实例ID
---

# 全局日志记录器

这是一个单例Skill，整个系统只有一个实例。

## 使用场景
- 记录系统运行日志
- 记录用户操作日志
- 记录错误日志

## 全局唯一性
- 状态在所有会话间共享
- 不允许创建多个实例
- 线程安全

## 日志级别
- DEBUG：调试信息
- INFO：一般信息
- WARN：警告信息
- ERROR：错误信息
```

### 1.3 变体：会话级单例

有时候你需要"每个用户会话一个实例，但会话内唯一"：

```yaml
# SKILL.user-context.md
---
name: user-context
description: 用户上下文管理器
version: 1.0.0
singleton_scope: session    # 会话级单例
---

# 用户上下文管理器

## 实例管理规则
- 每个用户会话创建一个实例
- 会话内所有请求共享同一实例
- 会话结束时销毁实例

## 存储内容
- 用户基本信息
- 当前对话上下文
- 用户偏好设置
```

### 1.4 什么时候不用单例

| 场景 | 不建议用单例 |
|------|-------------|
| Skill需要保存大量会话数据 | 数据会相互污染 |
| Skill之间有状态依赖 | 难以测试 |
| 需要并发执行 | 可能产生竞态条件 |

---

## 2. 组合模式：把多个Skill打包成一个

### 2.1 什么时候用

当你需要：
- 把多个相关能力打包给用户用
- 提供"一键完成"的体验
- 统一管理多个相关Skill

**典型场景**：
- 代码质量套件（语法检查+风格检查+安全检查）
- 周报生成器（收集数据+生成内容+格式化输出）
- 邮件助手（撰写+校对+发送）

### 2.2 怎么实现：树形组合

```yaml
# SKILL.code-quality-suite.md
---
name: code-quality-suite
description: 代码质量套件（组合多个检查Skill）
version: 1.0.0
type: composite
children:
  - SKILL.code-linter
  - SKILL.code-formatter
  - SKILL.security-checker
  - SKILL.performance-analyzer
---

# 代码质量套件

这是一个组合Skill，把多个单一职责的检查Skills组合在一起，
为用户提供一站式的代码质量检查服务。

## 执行流程
1. **并行执行**所有子Skills（同时检查语法、风格、安全、性能）
2. **收集结果**（汇总各子Skill的检查结果）
3. **去重合并**（消除重复的警告）
4. **生成报告**（输出综合报告）

## 配置选项

| 选项 | 说明 | 默认值 |
|------|------|--------|
| parallel | 是否并行执行 | true |
| fail_fast | 遇到错误是否立即停止 | false |
| report_format | 报告格式 | summary |

## 使用示例
```
用户：检查这段代码的质量
     ↓
系统：同时执行4个检查
     ↓
系统：收集结果，去重排序
     ↓
用户：收到综合报告
```

### 2.3 怎么实现：动态组合

根据输入类型动态选择要执行的子Skills：

```yaml
# SKILL.dynamic-code-analyzer.md
---
name: dynamic-code-analyzer
description: 动态代码分析器
version: 1.0.0
type: composite
dynamic_children: true
child_selector:
  language:
    javascript:
      - SKILL.eslint
      - SKILL.bundle-analyzer
    python:
      - SKILL.pylint
      - SKILL.type-checker
    rust:
      - SKILL.clippy
      - SKILL.cargo-check
---

# 动态代码分析器

根据输入代码的语言类型，动态选择相应的分析Skills。

## 选择规则

| 语言 | 执行的检查 |
|------|-----------|
| JavaScript/TypeScript | ESLint + Bundle分析 |
| Python | Pylint + 类型检查 |
| Rust | Clippy + Cargo检查 |
| 其他 | 通用代码分析 |

## 示例
```
用户：检查这个Python文件的代码质量
     ↓
系统：识别语言为Python
     ↓
系统：执行Pylint和类型检查
     ↓
用户：收到Python特有的分析报告
```
```

### 2.4 结果聚合逻辑

组合多个Skill时，需要定义聚合逻辑：

```yaml
aggregation:
  strategy: merge           # 合并策略
  conflict_resolution: last_write_wins  # 冲突解决：后写的覆盖先写的
  deduplication: true       # 去重
  result_priority:          # 优先级排序
    - security            # 安全问题优先
    - error              # 然后是错误
    - warning            # 然后是警告
    - info               # 最后是提示
```

---

## 3. 链式模式：一步步处理的艺术

### 3.1 什么时候用

当你需要：
- 数据处理流水线（一个步骤的输出是下一步的输入）
- 文本处理流程（分词→词性标注→实体识别）
- 代码处理流程（解析→分析→优化→生成）

**典型场景**：
- 文档处理（读取→解析→提取→格式化）
- 数据清洗（导入→清洗→转换→导出）
- 翻译流程（分句→翻译→校对→排版）

### 3.2 怎么实现：静态链式

```yaml
# SKILL.text-analysis-pipeline.md
---
name: text-analysis-pipeline
description: 文本分析处理流水线
version: 1.0.0
type: chain
chain:
  steps:
    - skill: SKILL.text-tokenizer
      name: 分词
      input: raw_text
      output: tokens
    - skill: SKILL.pos-tagger
      name: 词性标注
      input: tokens
      output: tagged_tokens
    - skill: SKILL.ner-tagger
      name: 命名实体识别
      input: tagged_tokens
      output: entities
    - skill: SKILL.sentiment-analyzer
      name: 情感分析
      input: entities
      output: sentiment
---

# 文本分析处理流水线

## 执行流程
```
原始文本
   ↓
┌─────────┐
│  分词   │  将文本切分成词语
└─────────┘
   ↓
┌─────────┐
│ 词性标注 │  标注每个词的词性
└─────────┘
   ↓
┌─────────┐
│ 命名实体 │  识别人名、地名等实体
└─────────┘
   ↓
┌─────────┐
│ 情感分析 │  判断文本的情感倾向
└─────────┘
   ↓
综合结果
```

## 中间结果
每个步骤的中间结果都会被保存：

| 步骤 | 输入 | 输出 | 用途 |
|------|------|------|------|
| 分词 | "今天天气很好" | ["今天", "天气", "很", "好"] | 后续步骤使用 |
| 词性标注 | tokens | [("今天", 时间), ("天气", 名词)] | NER使用 |
| NER | tagged_tokens | [实体1, 实体2] | 情感分析参考 |
| 情感分析 | entities | 正面/负面/中性 | 最终结果 |

## 问题排查
中间结果可用于：
- 定位是哪个环节出错
- 复用链中的中间结果
- 跳过已完成的部分
```

### 3.3 条件链式

根据中间结果选择不同的处理路径：

```yaml
# SKILL.conditional-document-processor.md
---
name: conditional-document-processor
description: 条件文档处理器
version: 1.0.0
type: conditional-chain
chain:
  - step: document-classifier
    branches:
      technical:
        - SKILL.tech-doc-processor
      business:
        - SKILL.business-doc-processor
      legal:
        - SKILL.legal-doc-processor
  - step: content-enhancer
    always: true    # 这个步骤总是执行
---

# 条件文档处理器

## 处理流程
```
文档输入
   ↓
┌──────────────────┐
│   文档分类器      │  判断文档类型
└────────┬─────────┘
         ↓
    ┌────┼────┐
    ↓    ↓    ↓
技术   业务   法律
文档   文档   文档
处理器 处理器 处理器
    ↓    ↓    ↓
    └────┼────┘
         ↓
┌──────────────────┐
│   内容增强器      │  总是执行
└──────────────────┘
         ↓
    最终输出
```

### 3.4 链式模式的错误处理

链式处理中，某一步的错误可能影响整个链：

```yaml
error_handling:
  strategy: fallback_with_skip  # 跳过+降级策略
  
  on_error:
    - log_error                # 记录错误
    - skip_step               # 跳过当前步骤
    - use_fallback_skill      # 使用备用Skill
    
  max_retries: 3              # 最多重试3次
  retry_delay: 1000           # 重试间隔1秒
```

| 错误处理策略 | 说明 | 适用场景 |
|-------------|------|----------|
| **Fail Fast** | 遇到错误立即停止 | 严格要求完整性的场景 |
| **Skip and Continue** | 跳过错误步骤继续 | 非关键步骤可跳过的场景 |
| **Fallback** | 使用备用Skill | 有降级方案的场景 |
| **Compensate** | 执行补偿操作 | 需要回滚的场景 |

---

## 4. 分支模式：根据条件走不同路径

### 4.1 什么时候用

当你需要：
- 根据输入类型选择处理方式
- 根据用户身份展示不同内容
- 根据条件执行不同逻辑

**典型场景**：
- 多格式文档处理（Word/PDF/HTML分别处理）
- 多语言内容处理
- 条件化业务逻辑

### 4.2 怎么实现：基于输入类型的分支

```yaml
# SKILL.document-processor.md
---
name: document-processor
description: 多格式文档处理器
version: 1.0.0
type: branch
branch_on: input.format
branches:
  pdf:
    skill: SKILL.pdf-processor
    condition:
      extension: [".pdf"]
  word:
    skill: SKILL.word-processor
    condition:
      extension: [".doc", ".docx"]
  html:
    skill: SKILL.html-processor
    condition:
      extension: [".html", ".htm"]
  markdown:
    skill: SKILL.markdown-processor
    condition:
      extension: [".md", ".markdown"]
  default:
    skill: SKILL.plaintext-processor
---

# 多格式文档处理器

## 支持的格式

| 格式 | 处理Skill | 处理方式 |
|------|-----------|----------|
| PDF | pdf-processor | 提取文本和图片 |
| Word | word-processor | 保留格式和样式 |
| HTML | html-processor | 提取结构化内容 |
| Markdown | markdown-processor | 解析元数据和目录 |
| 其他 | plaintext-processor | 纯文本处理 |

## 分支选择流程
```
输入文件 → 检测扩展名 → 匹配分支 → 执行对应Skill → 返回统一格式
```

## 输出统一化
无论输入什么格式，输出都是统一的：
```json
{
  "content": "文本内容",
  "metadata": {
    "title": "标题",
    "author": "作者",
    "length": "字数"
  },
  "format": "normalized"
}
```
```

### 4.3 怎么实现：基于规则的分支

用规则引擎实现更复杂的分支逻辑：

```yaml
# SKILL.intelligent-router.md
---
name: intelligent-router
description: 智能路由器
version: 1.0.0
type: rule-based-branch
rules:
  - name: 紧急问题优先
    condition: "issue.priority == 'urgent'"
    then:
      - SKILL.incident-manager
      - SKILL.auto-notification
      
  - name: 技术问题分类
    condition: "issue.category == 'technical'"
    then:
      - SKILL.tech-classifier
      - SKILL.tech-assignment
      
  - name: 业务问题升级
    condition: "issue.category == 'business'"
    then:
      - SKILL.business-escalation
      - SKILL.manager-notification
      
  - name: 默认处理
    condition: "true"
    then:
      - SKILL.general-processor
      - SKILL.wait-for-human
---

# 智能路由器

根据问题特征自动选择最合适的处理流程。

## 规则优先级
从上到下匹配，第一个匹配的规则生效。

## 规则示例

### 紧急问题规则
```
条件：priority == 'urgent'
动作：启动应急流程 + 自动通知负责人
```

### 技术问题规则
```
条件：category == 'technical'
动作：技术分类 + 分配给相应专家
```

### 业务问题规则
```
条件：category == 'business'
动作：升级给业务负责人 + 通知经理
```

### 默认规则
```
条件：true（始终匹配，作为兜底）
动作：通用处理 + 等待人工介入
```
```

---

## 5. 循环模式：重复执行直到完成

### 5.1 什么时候用

当你需要：
- 迭代优化（生成→评估→改进→再生成）
- 批量处理（处理一个又一个）
- 轮询等待（等待某个条件满足）

**典型场景**：
- AI写作（初稿→评审→修改→定稿）
- 代码优化（分析→优化→测试→验证）
- 数据同步（检查→同步→验证→重复）

### 5.2 怎么实现：固定次数循环

```yaml
# SKILL.iterative-code-optimizer.md
---
name: iterative-code-optimizer
description: 迭代式代码优化器
version: 1.0.0
type: loop
loop:
  max_iterations: 5           # 最多迭代5次
  iteration_skill: SKILL.code-improve
  exit_condition: |
    function check(result) {
      return result.quality_score >= 90 
          || result.improvement_delta < 1;
    }
---

# 迭代式代码优化器

## 工作流程
```
输入代码
    ↓
┌─────────────────┐
│  迭代 1          │ → 检查质量分数
│  代码改进        │ ↓
└────────┬────────┘
         ↓
    质量分数 ≥ 90？
         ↓
    ┌─是→ 输出结果 ✓
    ↓否
┌─────────────────┐
│  迭代 2          │ → 检查改进幅度
│  代码改进        │ ↓
└────────┬────────┘
         ↓
    改进幅度 < 1%？
         ↓
    ┌─是→ 输出结果 ✓
    ↓否
    ...（继续迭代，最多5次）
```

## 退出条件

| 条件 | 说明 |
|------|------|
| 质量分数 ≥ 90 | 代码质量已经很好 |
| 改进幅度 < 1% | 继续优化收益不大 |
| 达到最大迭代次数（5次） | 防止无限循环 |

## 迭代监控
每次迭代会记录：
- 迭代序号
- 当前质量分数
- 本次改进内容
- 距离目标的差距
```

### 5.3 怎么实现：条件循环

直到某个条件满足才停止：

```yaml
# SKILL.data-sync-loop.md
---
name: data-sync-loop
description: 数据同步循环
version: 1.0.0
type: conditional-loop
loop:
  condition_skill: SKILL.check-sync-status
  body_skill: SKILL.perform-sync
  max_iterations: 100
  sleep_between: 5000    # 每次循环间隔5秒
  exit_on:
    - sync_complete       # 同步完成
    - max_retries_exceeded  # 超过最大重试次数
    - critical_error      # 遇到严重错误
---

# 数据同步循环

## 执行流程
```
开始同步
    ↓
┌─────────────────────────┐
│  检查同步状态            │ ← 检查是否还有未同步的数据
└───────────┬─────────────┘
            ↓
       有待同步数据？
       ┌─否→ 同步完成 ✓
       ↓是
┌─────────────────────────┐
│  执行同步                │
│  （每次同步一批数据）     │
└───────────┬─────────────┘
            ↓
        等待5秒
            ↓
        回到检查状态步骤
```

---

## 6. 错误处理模式：让系统更健壮

### 6.1 重试模式

遇到临时性错误时自动重试：

```yaml
# SKILL.resilient-api-caller.md
---
name: resilient-api-caller
description: 弹性API调用器
version: 1.0.0
retry:
  max_attempts: 3           # 最多重试3次
  backoff:
    type: exponential       # 指数退避
    initial_delay: 1000     # 初始延迟1秒
    multiplier: 2           # 每次翻倍
    max_delay: 30000        # 最大延迟30秒
  retryable_errors:
    - network_timeout       # 网络超时
    - service_unavailable   # 服务不可用
    - rate_limit            # 限流
  non_retryable_errors:
    - auth_failure          # 认证失败
    - invalid_request       # 无效请求
---

# 弹性API调用器

## 重试策略

| 重试次数 | 延迟时间 | 累计等待 |
|---------|---------|---------|
| 第1次 | 1秒 | 1秒 |
| 第2次 | 2秒 | 3秒 |
| 第3次 | 4秒 | 7秒 |

## 错误分类

| 可重试 | 不可重试 |
|--------|----------|
| 网络超时 | 认证失败 |
| 服务不可用 | 无效请求 |
| 限流 | 参数错误 |

## 什么时候重试
- 网络问题 → 重试
- 服务暂时不可用 → 重试
- 请求太多被限流 → 重试

## 什么时候不重试
- 认证信息错误 → 不重试，修复认证
- 请求格式错误 → 不重试，修复请求
- 权限不足 → 不重试，获取权限
```

### 6.2 熔断模式

当错误率过高时暂停调用，保护系统：

```yaml
# SKILL.circuit-protected-service.md
---
name: circuit-protected-service
description: 带熔断保护的服务调用
version: 1.0.0
circuit_breaker:
  failure_threshold: 5      # 5次失败触发熔断
  success_threshold: 3      # 3次成功恢复
  timeout: 60000            # 熔断持续60秒
  half_open_max_calls: 1    # 半开状态最多1个请求
---

# 熔断保护模式

## 三种状态

### 关闭状态（正常）
```
所有请求正常通过
失败计数累计
失败达到阈值 → 进入打开状态
```

### 打开状态（熔断）
```
所有请求立即失败
等待超时时间
超时后 → 进入半开状态
```

### 半开状态（试探）
```
允许1个请求通过
成功 → 恢复关闭状态
失败 → 重新进入打开状态
```

## 配置说明

| 参数 | 说明 |
|------|------|
| failure_threshold | 触发熔断的失败次数 |
| success_threshold | 恢复所需的成功次数 |
| timeout | 熔断持续时间 |
| half_open_max_calls | 半开状态允许的请求数 |
```

### 6.3 降级模式

主方案失败时使用备选方案：

```yaml
# SKILL.fallback-enabled-skill.md
---
name: fallback-enabled-skill
description: 支持降级的Skill
version: 1.0.0
fallback:
  primary: SKILL.advanced-analyzer
  fallback_chain:
    - SKILL.basic-analyzer
    - SKILL.simple-heuristic
    - SKILL.default-response
  fallback_conditions:
    - primary_timeout        # 主方案超时
    - primary_error         # 主方案报错
    - low_confidence_score  # 置信度太低
---

# 支持降级的Skill

## 降级链路
```
请求 → 高级分析器
         ↓ 失败/超时/低置信度
      基础分析器
         ↓ 失败
      简单启发式
         ↓ 失败
      默认响应（永不失败）
```

## 降级策略

| 层级 | Skill | 说明 |
|------|-------|------|
| 第1层 | 高级分析器 | 最精确，但可能慢或不稳定 |
| 第2层 | 基础分析器 | 准确性一般，但稳定 |
| 第3层 | 简单启发式 | 快速但不精确 |
| 第4层 | 默认响应 | 永远成功，但只是"无法处理" |

## 降级通知
每次降级都会：
- 记录详细日志
- 发送监控告警
- 统计降级次数
```

---

## 7. 组合使用：实际案例

### 7.1 案例：智能客服系统

这是一个综合使用多种模式的案例：

```yaml
# SKILL.smart-customer-service.md
---
name: smart-customer-service
description: 智能客服系统
version: 1.0.0
type: composite

# 使用组合模式：整合多个能力
children:
  - SKILL.intent-classifier      # 意图分类（分支模式）
  - SKILL.response-generator     # 响应生成（链式模式）
  - SKILL.quality-checker        # 质量检查（单例模式）

# 使用循环模式：迭代优化响应
loop:
  max_iterations: 3
  iteration_skill: SKILL.response-refiner

# 使用降级模式：保证可用性
fallback:
  primary: SKILL.ai-response
  fallback_chain:
    - SKILL.template-response
    - SKILL.human-escalation
---

# 智能客服系统

## 系统架构

```
用户问题
    ↓
┌─────────────────────────┐
│   意图分类器（分支模式）  │ → 技术问题/业务问题/投诉/其他
└───────────┬─────────────┘
            ↓
    ┌───────┼───────┐
    ↓       ↓       ↓
  技术    业务    投诉
  路由    路由    路由
    ↓       ↓       ↓
┌─────────────────────────┐
│  响应生成器（链式模式）  │
│  理解 → 检索 → 生成     │
└───────────┬─────────────┘
            ↓
┌─────────────────────────┐
│   质量检查器（单例）     │ → 格式/安全/情感检查
└───────────┬─────────────┘
            ↓
┌─────────────────────────┐
│  响应优化器（循环模式）   │
│  迭代3次提升质量         │
└───────────┬─────────────┘
            ↓
用户回复
```

## 容错机制

| 场景 | 处理方式 |
|------|----------|
| 意图分类失败 | 使用通用路由 |
| 响应生成超时 | 降级到模板响应 |
| 质量检查不通过 | 迭代优化 |
| 3次迭代后仍不通过 | 转人工服务 |

---

## 8. 模式选择指南

### 8.1 根据场景选择

| 你的需求 | 推荐模式 |
|----------|----------|
| 只需要一个实例 | 单例模式 |
| 把多个能力打包 | 组合模式 |
| 一步步处理数据 | 链式模式 |
| 根据条件选路径 | 分支模式 |
| 重复执行直到完成 | 循环模式 |
| 处理可能的错误 | 重试/熔断/降级 |

### 8.2 模式的组合

实际应用中，模式通常是组合使用的：

```
┌─────────┐
│ 组合模式  │ ← 外层：用组合打包整个系统
│  ┌─────┐│
│  │分支 ││ ← 内层1：根据类型分支
│  └─────┘│
│  ┌─────┐│
│  │链式 ││ ← 内层2：顺序处理
│  └─────┘│
│  ┌─────┐│
│  │循环 ││ ← 内层3：迭代优化
│  └─────┘│
└─────────┘
```

### 8.3 过度设计的陷阱

**警告**：不是用越多模式越好！

| 简单场景 | 不需要 | 用太多会 |
|----------|--------|----------|
| 1个Skill | 任何模式 | 过度复杂 |
| 2-3个Skill | 简单组合 | 过度工程 |
| 5个以上 | 适当模式 | 恰到好处 |

**经验法则**：如果你的Skill少于5个，先别想什么设计模式，用简单的组合就够了。

---

## 9. 常见问题

### Q1: 什么时候用组合模式而不是链式模式？

**答**：
- 组合模式：子Skills相对独立，并行或无序执行
- 链式模式：子Skills有依赖，一个的输出是下一个的输入

### Q2: 循环模式会导致死循环吗？

**答**：有可能。所以必须设置：
1. `max_iterations` 最大迭代次数
2. `exit_condition` 退出条件
3. 超时机制

### Q3: 多个模式混用会不会很乱？

**答**：会。所以建议：
1. 在设计阶段就想清楚模式选择
2. 做好文档说明
3. 给每个Skill加上`type`标记

### Q4: 性能上有什么考虑？

**答**：
- 链式模式：关注每个步骤的延迟
- 组合模式：考虑并行执行的可能性
- 循环模式：注意迭代次数限制

---

## 总结

1. **单例模式**：全局唯一实例
2. **组合模式**：把多个Skill打包
3. **链式模式**：一步步顺序处理
4. **分支模式**：根据条件选路径
5. **循环模式**：重复执行直到完成
6. **错误处理**：重试、熔断、降级

记住：**模式是工具，不是目的**。选择合适的，而不是选择复杂的。

---

## 下一步

- 想学多Skill如何通信协作 → [[Skills角色分配]]
- 想学如何让Skill调用外部工具 → [[Tool集成指南]]
- 想学如何测试你的设计 → [[Skills测试与优化]]

---

## 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[Skills角色分配]] - 角色与协作模式
- [[Skills编写规范]] - 编码规范
- [[Tool集成指南]] - 工具集成
- [[Skills测试与优化]] - 测试策略

---

*本文档系统介绍了Skills设计中常用的设计模式，以及它们的具体实现方式。*
