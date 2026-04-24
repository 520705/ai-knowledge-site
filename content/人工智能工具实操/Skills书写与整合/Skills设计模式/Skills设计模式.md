---
title: Skills设计模式
date: 2026-04-18
tags:
  - skills
  - 设计模式
  - 软件架构
  - 模式识别
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills设计模式
alias: [Skills设计模式, Skills Patterns, 技能设计模式]
---

## 关键词

| 模式类别 | 核心模式 | 应用场景 |
|---------|----------|---------|
| 结构型 | 单例模式、组合模式 | 能力组织 |
| 行为型 | 链式模式、分支模式 | 流程控制 |
| 复合型 | 循环模式、错误处理模式 | 复杂逻辑 |
| 架构型 | 分层模式、管道模式 | 系统构建 |
| 协作型 | 代理模式、观察者模式 | 多技能协作 |

---

# 设计模式在Skills系统中的价值

## 1.1 为什么Skills需要设计模式

设计模式是软件工程领域经过大量实践验证的通用解决方案。将设计模式引入Skills系统，可以带来以下价值：

**结构化思维**：设计模式为Skills的组织和交互提供了成熟的思维框架，避免"重新发明轮子"。

**降低复杂度**：通过将复杂问题分解为标准化的模式组合，使得Skills系统的设计和维护更加可控。

**促进复用**：良好的设计模式使得Skills更容易被理解和复用，降低学习成本。

**提升质量**：设计模式经过大量项目验证，使用它们可以减少犯错的概率。

> [!tip]
> 设计模式不是银弹。对于简单的Skills，直接实现可能比套用模式更高效。应该根据实际需求选择是否使用模式，以及使用哪种模式。

## 1.2 设计模式的分类

在Skills系统中，设计模式可以分为三大类：

**结构型模式**：关注Skills之间的组织关系，如如何组合多个Skills、如何组织继承结构。

**行为型模式**：关注Skills的执行流程，如如何处理条件分支、如何组织顺序执行。

**架构型模式**：关注Skills系统的整体架构，如如何实现分层、如何管理依赖。

---

# 单例模式（Singleton）

## 2.1 模式定义与意图

单例模式确保一个Skill在整个系统中只有一个实例，并提供一个全局访问点。这在以下场景中特别有用：

- 全局配置管理Skill
- 认证授权Skill
- 日志记录Skill
- 缓存管理Skill

单例模式的核心价值在于：

- **避免重复初始化**：减少资源消耗
- **保证状态一致性**：全局状态统一管理
- **简化调用接口**：全局单点访问

## 2.2 实现方式

在Skills系统中实现单例模式，主要有三种方式：

**全局注册表方式**：

```yaml
---
name: global-logger
description: 全局日志记录器
version: 1.0.0
singleton: true
instance_id: system-logger
---

# 全局日志记录器

这是一个单例Skill，用于系统级日志记录。

## 使用限制
- 全局只有一个实例
- 状态在所有会话间共享
- 不允许创建多个实例
```

**类单例模式（通过状态隔离）**：

某些场景下需要"类单例"——每个用户会话一个实例，但会话内唯一：

```yaml
---
name: user-context-manager
description: 用户上下文管理器
version: 1.0.0
singleton_scope: session
---

# 用户上下文管理器

## 实例管理
- 每个用户会话创建一个实例
- 会话内所有请求共享同一实例
- 会话结束时销毁实例
```

## 2.3 适用场景与注意事项

**适用场景**：

- 需要维护全局唯一状态的Skill
- 资源密集型Skill，希望避免重复初始化
- 需要全局访问点的基础服务

**注意事项**：

- 避免在单例中存储过多会话特定的数据
- 注意线程安全问题（如果是并发执行）
- 单例的测试可能需要特殊处理（Mock/Stub）

> [!warning]
> 过度使用单例可能导致系统耦合度增加。在考虑使用单例之前，请确认是否有其他更合适的设计方案。

---

# 组合模式（Composite）

## 3.1 模式定义与意图

组合模式允许将多个Skills组合成一个树形结构，对单个Skill和Skill组合提供统一的接口。这使得用户可以以相同的方式使用简单Skills和复杂Skills组合。

组合模式的核心价值在于：

- **统一接口**：用户无需关心Skill是简单还是复杂
- **递归组合**：可以组合任意深度的Skill树
- **透明性**：使用方无需知道组合的内部结构

## 3.2 实现方式

**树形组合结构**：

```yaml
---
name: code-quality-suite
description: 代码质量套件（组合多个质量检查Skills）
version: 1.0.0
type: composite
children:
  - SKILL.code-linter
  - SKILL.code-formatter
  - SKILL.security-checker
  - SKILL.performance-analyzer
---

# 代码质量套件

这是一个组合Skill，它将多个单一职责的检查Skills组合在一起，
为用户提供一站式的代码质量检查服务。

## 执行流程
1. 并行执行所有子Skills
2. 收集各子Skill的检查结果
3. 合并结果，消除重复警告
4. 生成综合报告

## 配置选项
- `parallel`: 是否并行执行（默认true）
- `fail_fast`: 遇到错误是否立即停止（默认false）
- `report_format`: 报告格式（summary/detail）
```

**动态组合**：

```yaml
---
name: dynamic-code-analyzer
description: 动态代码分析器
version: 1.0.0
type: composite
dynamic_children: true
child_selector:
  language:
    javascript: [SKILL.eslint, SKILL.bundle-analyzer]
    python: [SKILL.pylint, SKILL.type-checker]
    rust: [SKILL.clippy, SKILL.cargo-check]
---

# 动态代码分析器

根据输入代码的语言类型，动态选择并组合相应的分析Skills。

## 动态选择规则
- JavaScript/TypeScript → ESLint + Bundle分析
- Python → Pylint + 类型检查
- Rust → Clippy + Cargo检查
- 其他语言 → 通用代码分析
```

## 3.3 组合的聚合逻辑

组合模式需要定义明确的聚合逻辑来处理子Skill的结果：

```yaml
aggregation:
  strategy: merge
  conflict_resolution: last_write_wins
  deduplication: true
  result_priority:
    - security
    - error
    - warning
    - info
```

---

# 链式模式（Chain）

## 4.1 模式定义与意图

链式模式将多个Skills串联成一条处理链，每个Skill处理完成后将结果传递给下一个Skill。这适用于管道式的处理流程，如：

- 数据处理流水线：验证 → 转换 → 增强 → 输出
- 文本处理流水线：分词 → 词性标注 → 实体识别 → 关系抽取
- 代码处理流水线：解析 → 分析 → 优化 → 生成

链式模式的核心价值在于：

- **关注点分离**：每个Skill只负责一个处理步骤
- **可插拔**：可以灵活替换链中的某个环节
- **可观测**：可以追踪数据在每个环节的处理情况

## 4.2 实现方式

**静态链式定义**：

```yaml
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
1. 接收原始文本
2. 分词处理
3. 词性标注
4. 命名实体识别
5. 情感分析
6. 输出综合结果

## 中间结果
每个步骤的中间结果会被保存，便于：
- 问题排查：定位是哪个环节出错
- 结果复用：链中的中间结果可以被单独使用
- 性能优化：跳过已完成的部分
```

**条件链式**：

```yaml
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
    always: true
```

## 4.3 链式模式的错误处理

链式模式中，某一步的错误可能影响整个链的执行。常见的错误处理策略：

- **Fail Fast**：遇到错误立即停止，返回错误信息
- **Skip and Continue**：遇到错误跳过该步骤，继续执行后续步骤
- **Fallback**：使用备选Skill替代出错的Skill
- **Compensate**：执行补偿操作，撤销已完成步骤的影响

```yaml
error_handling:
  strategy: fallback_with_skip
  on_error:
    - log_error
    - skip_step
    - use_fallback_skill
  max_retries: 3
  retry_delay: 1000
```

> [!note]
> 选择哪种错误处理策略，取决于业务场景的容错要求和数据一致性要求。对于金融、医疗等高可靠性场景，建议使用Fail Fast或Compensate策略。

---

# 分支模式（Branch）

## 5.1 模式定义与意图

分支模式根据输入或中间结果，选择执行不同的Skill或Skill分支。这适用于：

- 多格式文档处理（Word/PDF/HTML分别处理）
- 多语言内容处理
- 条件化业务逻辑

分支模式的核心价值在于：

- **灵活性**：同一输入可触发不同处理路径
- **可扩展性**：添加新分支不影响现有逻辑
- **清晰性**：每个分支的职责明确

## 5.2 实现方式

**基于输入类型的分支**：

```yaml
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
| 格式 | 处理Skill | 说明 |
|-----|-----------|------|
| PDF | pdf-processor | 提取文本和图片 |
| Word | word-processor | 保留格式和样式 |
| HTML | html-processor | 提取结构化内容 |
| Markdown | markdown-processor | 解析元数据和目录 |

## 分支选择逻辑
1. 根据文件扩展名确定类型
2. 调用对应类型的处理Skill
3. 返回统一格式的处理结果
```

**基于规则引擎的分支**：

```yaml
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
```

---

# 循环模式（Loop）

## 6.1 模式定义与意图

循环模式允许重复执行一个或一组Skills，直到满足退出条件。这适用于：

- 迭代优化任务（如生成-评估-改进循环）
- 批量处理任务
- 轮询等待任务

循环模式的核心价值在于：

- **自动化重试**：无需人工干预即可处理不稳定任务
- **迭代改进**：通过多次迭代逐步提升结果质量
- **批量处理**：自动处理大量相似任务

## 6.2 实现方式

**固定次数循环**：

```yaml
---
name: iterative-code-optimizer
description: 迭代式代码优化器
version: 1.0.0
type: loop
loop:
  max_iterations: 5
  iteration_skill: SKILL.code-improve
  exit_condition: |
    function check(result) {
      return result.quality_score >= 90 
          || result.improvement_delta < 1;
    }
---

# 迭代式代码优化器

## 工作原理
1. 接收待优化的代码
2. 调用代码改进Skill
3. 评估改进效果
4. 如果未达标，重复步骤2-3
5. 达到退出条件时停止

## 退出条件
- 质量分数达到90分以上
- 连续两次改进幅度小于1%
- 达到最大迭代次数（5次）

## 迭代监控
每次迭代会记录：
- 迭代序号
- 当前质量分数
- 本次改进内容
- 距离目标的差距
```

**条件循环**：

```yaml
---
name: data-sync-loop
description: 数据同步循环
version: 1.0.0
type: conditional-loop
loop:
  condition_skill: SKILL.check-sync-status
  body_skill: SKILL.perform-sync
  max_iterations: 100
  sleep_between: 5000  # 毫秒
  exit_on:
    - sync_complete
    - max_retries_exceeded
    - critical_error
```

---

# 错误处理模式（Error Handling）

## 7.1 模式定义与意图

错误处理模式定义了当Skill执行过程中遇到错误时的处理策略。良好的错误处理是保障系统健壮性的关键。

常见的错误处理模式包括：

- **重试模式（Retry）**：遇到临时性错误时自动重试
- **熔断模式（Circuit Breaker）**：当错误率过高时暂停调用
- **降级模式（Fallback）**：主Skill失败时使用备选方案
- **隔离模式（Isolation）**：某个Skill的错误不影响其他Skills

## 7.2 实现方式

**重试模式**：

```yaml
---
name: resilient-api-caller
description: 弹性API调用器
version: 1.0.0
retry:
  max_attempts: 3
  backoff:
    type: exponential
    initial_delay: 1000
    multiplier: 2
    max_delay: 30000
  retryable_errors:
    - network_timeout
    - service_unavailable
    - rate_limit
  non_retryable_errors:
    - auth_failure
    - invalid_request
---

# 弹性API调用器

## 重试策略
| 重试次数 | 延迟时间 | 累计等待 |
|---------|---------|---------|
| 第1次 | 1秒 | 1秒 |
| 第2次 | 2秒 | 3秒 |
| 第3次 | 4秒 | 7秒 |

## 错误分类
- **可重试错误**：网络超时、服务不可用、限流
- **不可重试错误**：认证失败、无效请求
```

**熔断模式**：

```yaml
---
name: circuit-protected-service
description: 带熔断保护的服务调用
version: 1.0.0
circuit_breaker:
  failure_threshold: 5      # 5次失败触发熔断
  success_threshold: 3      # 3次成功恢复
  timeout: 60000            # 熔断持续60秒
  half_open_max_calls: 1    # 半开状态最多1个请求
```

**降级模式**：

```yaml
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
    - primary_timeout
    - primary_error
    - low_confidence_score
---

# 支持降级的Skill

## 降级策略
1. 首先尝试高级分析器
2. 如果超时或失败，降级到基础分析器
3. 如果仍然失败，使用简单启发式方法
4. 最后返回默认响应（避免空结果）

## 降级通知
每次降级都会记录日志，并可选择通知监控系统
```

> [!example]
> 一个综合使用多种错误处理模式的示例：
> ```yaml
> ---
> name: robust-data-processor
> description: 健壮的数据处理器
> version: 1.0.0
> error_handling:
>   retry:
>     max_attempts: 3
>     backoff: exponential
>   circuit_breaker:
>     enabled: true
>     threshold: 5
>   fallback:
>     enabled: true
>     primary: SKILL.premium-processor
>     backup: SKILL.basic-processor
>   isolation:
>     enabled: true
>     timeout: 30000
> ```

---

# 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[Skills角色分配]] - 角色与协作模式
- [[Skills编写规范]] - 编码规范
- [[Skills测试与优化]] - 测试策略
- [[Tool集成指南]] - 工具集成

---

*本文档系统介绍了Skills设计中常用的设计模式，以及它们的具体实现方式。*
