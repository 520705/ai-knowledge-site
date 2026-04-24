---
title: SKILL.md语法详解
date: 2026-04-18
tags:
  - skills
  - SKILL.md
  - 语法规范
  - 元数据
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills基础理论
alias: [SKILL.md语法, SKILL.md格式, Skill File Format]
---

## 关键词

| 字段类别 | 核心字段 | 说明 |
|---------|----------|------|
| 元数据 | name, description, version | Skill身份标识 |
| 触发 | trigger, trigger_words | 激活条件 |
| 指令 | instructions, system_prompt | 行为定义 |
| 示例 | examples, few_shot | 学习样本 |
| 输出 | output_format, output_path | 结果规范 |
| 工具 | tools, api_calls | 能力扩展 |
| 依赖 | requires, imports | 关联关系 |

---

# SKILL.md标准格式概述

## 1.1 文件格式规范

SKILL.md是Skills系统的核心文件，它采用Markdown语法编写，同时包含结构化的元数据区块（YAML格式的frontmatter）和自然语言描述的行为指令。这种混合格式的优势在于：既保持了人类可读性，又为机器解析提供了明确的结构。

文件命名遵循以下规范：

- 文件名必须以`SKILL.`为前缀
- 使用小写字母和连字符
- 避免使用特殊字符和空格
- 建议包含版本号，如`SKILL.example-v1.0.md`

完整的SKILL.md文件结构如下：

```markdown
---
name: skill-name
description: 简短描述
trigger: [trigger_word_1, trigger_word_2]
version: 1.0.0
author: author-name
requires: [dependency-skill-1]
tools: [tool-1, tool-2]
---

# Skill Instructions

## 详细指令
...
```

## 1.2 frontmatter与主体的关系

frontmatter是SKILL.md文件开头的YAML区块，用三横线`---`包裹。它主要用于定义Skill的元数据，这些元数据供系统解析器使用，用于Skill的发现、路由和加载。

文件主体则是自然语言描述的指令内容，供AI模型理解并执行。两者的关系可以类比为"配置文件"与"源代码"的关系：frontmatter告诉系统"这个Skill是什么"，主体告诉AI"这个Skill要做什么"。

> [!tip]
> frontmatter中的字段应该是机器可解析的精确值，而主体中的描述可以更加灵活和自然。这种分工使得SKILL.md文件既能被系统处理，又保留了人类可读性。

---

# frontmatter规范详解

## 2.1 必须字段

### name字段

`name`是Skill的唯一标识符，在整个系统中必须唯一。它应该简洁、描述性且符合命名规范。

命名规则：

- 只能包含小写字母、数字和连字符
- 必须以字母开头
- 最大长度50字符
- 不能包含空格，使用连字符替代
- 建议使用功能描述性命名

```yaml
# 正确的命名
name: code-reviewer
name: data-analyst
name: email-composer

# 错误的命名
name: CodeReviewer  # 大写字母
name: code reviewer  # 空格
name: code.reviewer  # 特殊字符
```

### description字段

`description`是对Skill功能的简要描述，用于人类理解和Skill发现。描述应该清晰、准确，一句话概括核心功能。

```yaml
description: 自动审查代码并提供改进建议
```

### trigger字段

`trigger`定义了激活Skill的关键词或模式。可以是单个字符串，也可以是字符串数组。

```yaml
# 单个触发词
trigger: "代码审查"

# 多个触发词（任一匹配即激活）
trigger:
  - "代码审查"
  - "review code"
  - "检查代码"

# 带权重的触发词
trigger:
  - word: "审查代码"
    weight: 1.0
  - word: "看看代码"
    weight: 0.7
```

## 2.2 可选字段

### version字段

```yaml
version: 1.0.0  # 遵循语义化版本规范
```

### author字段

```yaml
author: "张三"
author: "团队名称 <team@example.com>"
```

### category字段

用于分类管理多个Skills：

```yaml
category:
  - 开发工具
  - 代码质量
  - 自动化测试
```

### tags字段

提供多维度的标签，便于搜索和筛选：

```yaml
tags:
  - javascript
  - typescript
  - 前端开发
  - 代码审查
```

### requires字段

声明该Skill依赖的其他Skills：

```yaml
requires:
  - SKILL.code-formatter
  - SKILL.git-helper
```

### tools字段

声明该Skill需要使用的工具：

```yaml
tools:
  - terminal
  - file_editor
  - web_search
```

### output字段

定义Skill输出的默认配置：

```yaml
output:
  format: markdown
  path: "./output"
  filename_pattern: "{date}-{skill-name}-{index}"
```

> [!warning]
> 避免在frontmatter中放置过多的配置信息。frontmatter应该保持精简，详细的配置应该放在文件主体的专门章节中。

---

# 核心字段详解：instructions

## 3.1 instructions的作用

`instructions`字段是SKILL.md的核心，它定义了Skill的主要行为指令。这部分内容会被作为系统提示（System Prompt）的一部分发送给AI模型，因此指令的质量直接影响Skill的执行效果。

instructions通常包含以下几个部分：

1. **角色定义**：明确AI在这个Skill中扮演的角色
2. **任务描述**：具体要完成什么任务
3. **行为约束**：完成任务时需要遵守的规则
4. **输出格式**：结果应该以什么形式呈现
5. **边界条件**：什么情况应该拒绝或报错

## 3.2 编写高质量instructions的原则

**清晰性原则**：指令应该明确、无歧义。避免使用"可能"、"也许"等模糊词汇。对于每个行为，应该给出明确的触发条件和执行方式。

```yaml
# 不好的示例
instructions: |
  你是一个助手，有时候会帮助用户处理一些任务。
  如果用户需要的话，你也可以做一些其他的事情。

# 好的示例
instructions: |
  你是一个代码审查助手，专门负责审查JavaScript代码。
  当用户发送包含.js后缀的文件内容时，你应该：
  1. 语法检查：识别潜在的语法错误
  2. 风格检查：检查是否符合ESLint规则
  3. 最佳实践：检查是否遵循React/Vue等框架的最佳实践
  4. 性能建议：识别可能的性能问题
```

**完整性原则**：指令应该覆盖所有可能的场景和边界情况。对于异常输入，应该给出明确的处理方式。

**简洁性原则**：在保持完整性的前提下，指令应该尽量简洁。避免冗长的解释和不必要的修饰词。

**可执行原则**：指令应该描述可执行的行为，而不是抽象的目标。

```yaml
# 不可执行的指令
instructions: |
  提高代码质量。

# 可执行的指令
instructions: |
  对代码进行以下六个维度的检查：
  1. 命名规范：变量和函数命名是否清晰、有意义
  2. 函数长度：每个函数不超过50行
  3. 圈复杂度：每个函数不超过10
  4. 重复代码：识别重复超过3次的代码块
  5. 注释质量：关键逻辑是否有必要的注释
  6. 错误处理：是否有适当的try-catch
```

## 3.3 instructions的结构化写法

对于复杂的Skill，建议采用结构化的指令写法：

```yaml
instructions: |
  ## 角色定义
  你是一个专业的技术文档撰写助手，专注于API文档的编写。
  
  ## 核心职责
  1. 解析代码中的函数、类、接口定义
  2. 提取参数、返回值、异常信息
  3. 生成符合OpenAPI规范的文档
  
  ## 输出格式
  生成的文档必须包含以下部分：
  - 概述：功能简述
  - 端点：API路径和方法
  - 参数：请求参数列表（含类型、必填、说明）
  - 响应：响应格式和状态码
  - 示例：请求和响应的JSON示例
  
  ## 约束条件
  - 使用中文编写
  - 术语保持一致性
  - 示例代码必须可运行
```

---

# 核心字段详解：examples

## 4.1 examples的作用

`examples`字段用于提供Few-shot学习样本。人类天生善于从例子中学习，AI模型同样如此。高质量的examples可以显著提升Skill的执行效果。

examples的作用包括：

- **明确输出格式**：通过示例让AI理解期望的输出结构
- **展示边界情况**：示例可以包含特殊情况如何处理
- **传递风格偏好**：示例可以体现语言风格和专业术语使用

## 4.2 编写examples的原则

**代表性原则**：examples应该覆盖主要的输入类型和场景。如果一个Skill需要处理多种情况，应该为每种情况提供示例。

**一致性原则**：examples中的输入输出对应该与实际使用场景一致。避免在examples中使用过于简化或人为构造的例子。

**可验证原则**：examples应该有明确的正确答案，用于验证Skill的输出质量。

```yaml
examples:
  - input: |
      ```javascript
      function add(a, b) {
        return a + b;
      }
      ```
    output: |
      ## 函数分析：add
      
      **功能**：对两个数求和
      
      **参数**：
      | 参数名 | 类型 | 必填 | 说明 |
      |-------|------|------|------|
      | a | number | 是 | 第一个加数 |
      | b | number | 是 | 第二个加数 |
      
      **返回值**：number，两数之和
      
      **示例**：
      ```javascript
      add(1, 2) // 返回 3
      ```
```

## 4.3 examples的数量控制

examples的数量并非越多越好。根据研究和实践经验：

- 1-3个examples通常足以覆盖主要场景
- 5个以上的examples可能导致过拟合
- 如果需要处理很多边界情况，考虑拆分成多个Skills

> [!note]
> 在某些情况下，0个examples反而更好，特别是当instructions已经足够清晰，或者期望AI自由发挥而非遵循固定格式时。

---

# 工具定义与参数

## 5.1 tools字段的格式

`tools`字段用于声明Skill可以使用的外部工具。这使得Skill具备了调用外部系统、执行代码、读写文件等能力。

```yaml
tools:
  - name: file_reader
    description: 读取本地文件
    parameters:
      path:
        type: string
        required: true
        description: 文件路径
      encoding:
        type: string
        default: utf-8
        
  - name: command_executor
    description: 执行shell命令
    parameters:
      command:
        type: string
        required: true
      timeout:
        type: integer
        default: 30000
```

## 5.2 工具定义模板

完整的工具定义应该包含以下字段：

```yaml
tools:
  - name: tool_name          # 工具名称
    description: 工具用途描述   # 简短说明
    category: 文件操作          # 工具分类
    parameters:               # 参数定义
      param_name:
        type: string|number|boolean|object|array
        required: true|false
        default: default_value
        description: 参数说明
        enum: [value1, value2]  # 枚举值限制
    returns:
      type: string
      description: 返回值说明
    examples:                 # 工具使用示例
      - input: "..."
        output: "..."
```

## 5.3 内置工具vs自定义工具

系统中可以预定义一些常用工具，供所有Skills直接引用：

**内置工具**：

- `file_reader`：读取文件
- `file_writer`：写入文件
- `terminal`：执行命令
- `web_search`：网络搜索
- `web_fetch`：获取网页内容
- `clipboard`：读写剪贴板

**自定义工具**：

自定义工具需要开发者自己实现，通常需要：

1. 在frontmatter的`tools`字段中声明
2. 在instructions中描述如何使用
3. 提供工具的代码实现

---

# 依赖与引用

## 6.1 requires字段

`requires`字段用于声明Skill的依赖关系。当一个Skill需要使用另一个Skill的能力时，应该在requires中声明。

```yaml
requires:
  - SKILL.code-formatter
  - SKILL.git-helper
```

系统加载Skill时，会自动先加载其依赖的Skills。

## 6.2 依赖冲突处理

当多个Skills依赖同一Skill的不同版本时，会产生版本冲突。处理策略包括：

**优先策略**：优先加载显式指定的版本
**最新策略**：使用最新的兼容版本
**错误策略**：检测到冲突时报错，要求手动解决

## 6.3 条件依赖

可以指定条件性的依赖：

```yaml
requires:
  - skill: SKILL.code-formatter
    when:
      language: javascript
  - skill: SKILL.code-formatter-python
    when:
      language: python
```

> [!example]
> 一个完整的SKILL.md文件示例：
> ```markdown
> ---
> name: api-doc-generator
> description: 根据代码注释自动生成API文档
> version: 1.2.0
> author: DevTeam
> trigger:
>   - "生成API文档"
>   - "生成接口文档"
>   - "api doc"
> category:
>   - 开发工具
>   - 文档生成
> tags:
>   - API
>   - 文档
>   - 自动化
> requires:
>   - SKILL.code-parser
> tools:
>   - file_reader
>   - file_writer
> output:
>   format: markdown
>   path: "./docs/api"
> ---
> 
> # API文档生成器
> 
> 你是一个专业的API文档生成助手...
> 
> [详细指令内容...]
> ```

---

# 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[Skills编写规范]] - 编码规范和最佳实践
- [[Skills设计模式]] - 设计模式应用
- [[Tool集成指南]] - 工具集成详解
- [[Skills测试与优化]] - 测试策略

---

*本文档详细介绍了SKILL.md的完整语法规范，是编写高质量Skills的基础参考资料。*
