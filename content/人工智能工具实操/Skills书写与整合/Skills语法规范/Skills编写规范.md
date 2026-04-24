---
title: Skills编写规范
date: 2026-04-18
tags:
  - skills
  - 编码规范
  - 命名规范
  - 文档规范
  - 版本管理
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills语法规范
alias: [Skills编写规范, Skills编码规范, Skill Writing Standards]
---

## 关键词

| 规范类别 | 核心内容 | 重要性 |
|---------|----------|-------|
| 命名规范 | 文件名、变量名、触发词 | 高 |
| 文档结构 | frontmatter、章节、格式 | 高 |
| 示例规范 | 输入输出、边界情况 | 高 |
| 注释规范 | 必要性、格式、避免冗余 | 中 |
| 版本管理 | 语义化版本、变更记录 | 高 |
| 质量检查 | 自检清单、审核流程 | 高 |

---

# 命名规范

## 1.1 文件命名规范

SKILL.md文件的命名应遵循统一规范，确保文件在文件系统和代码库中易于识别和管理。

### 命名格式

```
SKILL.<功能描述>[-<变体>].md
```

**规则说明**：

- 必须以`SKILL.`开头
- 功能描述使用小写字母和连字符
- 变体部分可选，用于区分同一功能的多个版本
- 扩展名必须是`.md`

### 正确命名示例

```bash
# 基础功能
SKILL.code-reviewer.md
SKILL.data-analyzer.md
SKILL.email-composer.md

# 带版本标识
SKILL.code-reviewer-v2.md
SKILL.legacy.pdf-parser.md

# 带变体标识
SKILL.data-analyzer-fast.md
SKILL.data-analyzer-detailed.md
```

### 错误命名示例

```bash
# 错误：大写字母
SKILL.CodeReviewer.md

# 错误：下划线
SKILL/code_reviewer.md

# 错误：空格
SKILL/code reviewer.md

# 错误：缺少前缀
code-reviewer.md

# 错误：特殊字符
SKILL.code@reviewer.md
```

## 1.2 变量和参数命名

在SKILL.md的frontmatter和配置中，变量命名应遵循以下规范：

```yaml
# 使用小写字母和连字符
good_variable_name: value

# 避免使用下划线或驼峰
bad_variable_name: value    # 下划线不推荐
badVariableName: value     # 驼峰不推荐

# 布尔变量使用is/has/can前缀
is_active: true
has_permission: true
can_execute: true

# 列表变量使用复数形式
trigger_words:
  - word_1
  - word_2
```

## 1.3 触发词命名

触发词作为用户与Skills交互的入口，其命名应清晰、直观：

```yaml
# 清晰的触发词
trigger:
  - "代码审查"
  - "检查代码"
  - "review code"

# 模糊的触发词
trigger:
  - "处理"       # 太模糊
  - "帮忙"       # 太模糊
  - "做一下"     # 不规范
```

---

# 文档结构规范

## 2.1 frontmatter结构

SKILL.md文件必须包含完整的frontmatter，且字段顺序应保持一致：

```yaml
---
# 第一部分：身份标识（必须）
name: skill-name
description: 简短描述（一句话）
version: 1.0.0

# 第二部分：关联信息（推荐）
author: 作者名
category: [分类1, 分类2]
tags: [标签1, 标签2]

# 第三部分：行为配置（可选）
trigger: [触发词1, 触发词2]
requires: [依赖1, 依赖2]
tools: [工具1, 工具2]

# 第四部分：输出配置（可选）
output:
  format: markdown
  path: ./output
---

# 正文内容
```

## 2.2 正文结构规范

正文部分应包含以下标准章节：

```markdown
# [Skill名称]

## 概述
简要介绍Skill的功能和使用场景。

## 角色定义
明确Skill扮演的角色和其核心能力。

## 使用限制
列出Skill的适用边界和禁忌事项。

## 使用示例
提供实际使用示例，展示输入输出。

## 配置选项
详细说明可配置的参数。

## 常见问题
FAQ和已知限制。
```

## 2.3 内容格式规范

### 标题层级

```markdown
# 一级标题：文档标题
## 二级标题：主要章节
### 三级标题：子章节
#### 四级标题：具体条目（谨慎使用）
```

### 列表格式

```markdown
## 有序列表
1. 第一步
2. 第二步
3. 第三步

## 无序列表
- 项目A
- 项目B
  - 子项目B1
  - 子项目B2
- 项目C
```

### 表格格式

```markdown
| 列1 | 列2 | 列3 |
|-----|-----|-----|
| A1  | B1  | C1  |
| A2  | B2  | C2  |
```

> [!tip]
> 表格应该用于展示需要对比或查找的信息，避免使用表格来展示大段文本。

---

# 示例编写规范

## 3.1 示例的质量标准

高质量的示例应满足以下标准：

**真实性**：示例应该基于真实使用场景，避免过于简化或人为构造。

**完整性**：示例应该包含完整的输入和期望的输出。

**可验证性**：示例应该有明确的评判标准，便于验证Skill的正确性。

**可执行性**：示例中的代码和命令应该是可运行的。

## 3.2 示例结构

```yaml
examples:
  - name: 基本使用示例
    description: 展示最常见的使用场景
    input: |
      # 输入内容
      ```language
      code_here
      ```
    output: |
      # 期望输出
      ```output
      expected_result
      ```
    expected_behavior: |
      描述这个示例中期望的行为
      
  - name: 边界情况示例
    description: 展示处理边界情况的能力
    input: |
      # 边界输入
    output: |
      # 边界输出
    expected_behavior: |
      如何处理这个边界情况
```

## 3.3 示例数量指南

| Skill类型 | 建议示例数量 | 说明 |
|-----------|-------------|------|
| 简单转换 | 2-3个 | 基本输入、典型输出、错误处理 |
| 复杂处理 | 5-8个 | 覆盖多种输入类型和边界情况 |
| 专业领域 | 10个以上 | 覆盖专业场景的各种变化 |
| 通用助手 | 3-5个 | 不同领域的典型示例 |

> [!warning]
> 过多的示例可能导致过拟合，降低Skill的泛化能力。如果需要处理很多边界情况，考虑拆分成多个更专注的Skills。

---

# 注释规范

## 4.1 何时需要注释

注释应该用于解释"为什么"，而不是"是什么"。以下是需要注释的场景：

**业务规则**：解释复杂的业务逻辑或决策依据。

**非显而易见的实现**：解释不直观的实现选择。

**外部依赖**：说明对外部系统的假设和依赖。

**已知的限制**：明确标注已知的限制和边界。

```yaml
# 好的注释：解释原因
# 根据业务规则，金额需要四舍五入到分
# 参考：财务系统规范 v2.3 第5.2节
rounding: half_up

# 不好的注释：重复代码内容
# 设置超时时间为30秒
timeout: 30000  # 30秒
```

## 4.2 注释的格式

```yaml
# 单行注释
# 这是一个简短说明

# 多行注释使用YAML的多行语法
description: |
  这是一个多行描述。
  可以包含多段落。
  用于解释复杂的内容。

# 重要提示使用callout格式
# > [!note]
# > 这是需要注意的重要信息
```

## 4.3 避免的注释模式

```yaml
# 避免：无意义的注释
good: true  # 设置good为true

# 避免：过时的注释
# TODO: 修复这个问题 (但TODO已经不存在)

# 避免：过于详细的注释（代码本身应该自解释）
# 定义一个字符串变量
name: "张三"  # 姓名字符串
```

---

# 版本管理规范

## 5.1 语义化版本

Skills应遵循语义化版本规范（SemVer）：

```
主版本.次版本.修订版本
MAJOR.MINOR.PATCH
```

**MAJOR（主版本）**：不兼容的API变更

**MINOR（次版本）**：向后兼容的功能新增

**PATCH（修订版本）**：向后兼容的问题修复

```yaml
version: 1.2.3
# 1 = 主版本
# 2 = 次版本
# 3 = 修订版本
```

## 5.2 版本变更记录

每次版本变更应在文档中记录：

```yaml
changelog:
  - version: 1.2.0
    date: 2026-04-18
    type: minor
    changes:
      added:
        - 新增Python代码审查能力
        - 新增安全漏洞扫描功能
      changed:
        - 优化了输出格式的一致性
      deprecated:
        - 旧版输出格式（将在2.0版本移除）
        
  - version: 1.1.0
    date: 2026-03-15
    type: minor
    changes:
      added:
        - 新增TypeScript支持
```

## 5.3 版本兼容性

```yaml
compatibility:
  requires: ">=1.0.0,<2.0.0"
  conflicts_with:
    - SKILL.old-version
  migration_guide: |
    从1.x迁移到2.x的指南...
```

---

# 质量检查清单

## 6.1 编写自检清单

在提交Skill之前，应进行以下检查：

### 元数据检查

- [ ] `name`字段符合命名规范
- [ ] `description`简洁明了，不超过50字
- [ ] `version`符合语义化版本规范
- [ ] `trigger`触发词清晰、无歧义
- [ ] `category`和`tags`分类合理

### 内容检查

- [ ] frontmatter格式正确、字段完整
- [ ] 正文结构清晰、层级合理
- [ ] 所有代码示例语法正确
- [ ] 所有配置示例格式正确
- [ ] 没有未解决的TODO或占位符

### 一致性检查

- [ ] 示例输入与描述一致
- [ ] 示例输出与实际行为一致
- [ ] 配置选项与代码实现一致
- [ ] 版本号与changelog一致

### 可维护性检查

- [ ] 注释说明了"为什么"而非"是什么"
- [ ] 复杂的逻辑有充分的解释
- [ ] 已知的限制和边界有明确标注
- [ ] 变更历史记录完整

## 6.2 审核流程

```yaml
review_process:
  stages:
    - name: self_review
      required: true
      checklist: above
      
    - name: peer_review
      required: true
      reviewers: 1
      focus:
        - 逻辑正确性
        - 用例覆盖度
        - 命名规范性
        
    - name: integration_test
      required: true
      tests:
        - basic_functionality
        - error_handling
        - performance
        
    - name: approval
      required: true
      approvers:
        - tech_lead
        - domain_expert
```

> [!note]
> 审核流程的严格程度应根据项目的规模和重要性调整。对于小型项目，可以简化审核流程；对于关键业务系统，应严格执行完整流程。

---

# 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[SKILL_md语法详解]] - SKILL.md格式详解
- [[Skills设计模式]] - 设计模式应用
- [[Tool集成指南]] - 工具集成
- [[Skills测试与优化]] - 测试策略

---

*本文档定义了Skills编写的各项规范，是保证Skills质量的指导标准。*
