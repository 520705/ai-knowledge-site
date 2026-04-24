---
title: Ollama自定义模型完全指南
date: 2026-04-18
tags:
  - Ollama
  - Modelfile
  - FROM
  - PARAMETER
  - TEMPLATE
  - SYSTEM
categories:
  - AI工具实操
  - 大模型调用
alias: [ollama-modelfile, modelfile-guide]
---

> [!info] 文档信息
> 本文档详细介绍Ollama Modelfile的完整语法，包括FROM、PARAMETER、TEMPLATE、SYSTEM等指令的详细用法。

## 核心关键词

| 关键词 | 说明 |
|--------|------|
| Modelfile | Ollama模型定义文件 |
| FROM | 基础模型指令 |
| PARAMETER | 参数配置指令 |
| TEMPLATE | 提示模板指令 |
| SYSTEM | 系统提示词指令 |
| ADAPTER | LoRA适配器指令 |
| LICENSE | 许可证指令 |
| Prompt工程 | 提示词优化 |
| 量化配置 | 模型量化设置 |
| 多模态 | 视觉模型配置 |

---

## 一、Modelfile概述

Modelfile是Ollama中定义和自定义模型的核心文件，其语法类似于Dockerfile，采用声明式的配置方式来描述模型的各个方面。通过Modelfile，用户可以基于现有的开源模型创建高度定制化的AI助手，无论是专业的代码助手、翻译专家，还是角色扮演应用，都可以通过Modelfile实现。

Modelfile的指令按功能可以分为几大类：基础配置指令（FROM、LICENSE）、参数配置指令（PARAMETER）、提示词配置指令（SYSTEM、TEMPLATE）、高级配置指令（ADAPTER、EMBED）、以及特殊指令（ROLES、Message）。理解这些指令的用法和组合方式是掌握Ollama自定义模型的关键。

## 二、FROM指令详解

### 2.1 基础用法

FROM指令指定Modelfile所基于的基础模型，是每个Modelfile的必需指令：

```dockerfile
# 指定基础模型
FROM llama3.2:3b

# 使用特定量化版本
FROM llama3.2:3b-instruct-q4_K_M

# 使用其他模型作为基础
FROM mistral:7b-instruct

# 使用国产模型
FROM qwen2.5:14b
FROM deepseek-coder:6.7b
FROM yi:1.5:6b
```

### 2.2 模型仓库格式

```dockerfile
# Ollama官方库模型
FROM llama3:8b-instruct

# 自定义仓库模型
FROM registry.example.com/custom-model:1.0

# 本地模型
FROM ./models/custom-model.gguf

# 特定版本
FROM llama3.2:3b-instruct-fp16
```

### 2.3 多模态模型

```dockerfile
# 视觉模型
FROM llava:7b

# 音频模型（未来支持）
# FROM audio-model:1.0
```

## 三、PARAMETER指令详解

### 3.1 可用参数列表

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| temperature | float | 0.8 | 随机性控制（0-2） |
| top_p | float | 0.9 | nucleus采样阈值 |
| top_k | int | 40 | top-k采样 |
| num_ctx | int | 2048 | 上下文窗口大小 |
| num_gpu | int | 0 | GPU层数 |
| num_thread | int | 自动 | CPU线程数 |
| repeat_penalty | float | 1.1 | 重复惩罚 |
| seed | int | 随机 | 随机种子 |
| num_predict | int | 256 | 最大生成token数 |
| stop | string | - | 停止词 |
| mirostat | int | 0 | Mirostat采样模式 |
| frequency_penalty | float | 0 | 频率惩罚 |
| presence_penalty | float | 0 | 存在惩罚 |

### 3.2 详细配置示例

```dockerfile
# 精确控制输出
PARAMETER temperature 0.3
PARAMETER top_p 0.85
PARAMETER top_k 30
PARAMETER num_predict 2048

# 确定性输出（适合代码生成）
PARAMETER temperature 0.1
PARAMETER repeat_penalty 1.2
PARAMETER num_ctx 8192

# 创意输出（适合写作）
PARAMETER temperature 0.9
PARAMETER top_p 0.95
PARAMETER repeat_penalty 1.0
```

### 3.3 条件参数

```dockerfile
# 针对特定模型的参数
PARAMETER temperature 0.7
PARAMETER num_ctx 4096

# 当使用GPU时增加上下文
PARAMETER num_ctx 8192
```

### 3.4 高级参数

```dockerfile
# Mirostat采样（自适应困惑度控制）
PARAMETER mirostat 2
PARAMETER mirostat_tau 5.0
PARAMETER mirostat_eta 0.1

# 惩罚参数
PARAMETER repeat_penalty 1.2
PARAMETER frequency_penalty 0.5
PARAMETER presence_penalty 0.5

# 推理优化
PARAMETER num_thread 8
PARAMETER num_batch 512
```

## 四、SYSTEM指令详解

### 4.1 基础系统提示词

```dockerfile
FROM llama3.2:3b

# 简洁的系统提示词
SYSTEM """
You are a helpful assistant.
"""

# 详细的行为定义
SYSTEM """
You are an expert Python programmer with 10 years of experience.
You always:
- Write clean, readable code
- Include type hints and docstrings
- Follow PEP 8 style guide
- Add error handling
"""
```

### 4.2 多语言助手

```dockerfile
FROM qwen2.5:14b

SYSTEM """
你是一位专业的中英双语翻译专家。

翻译原则：
1. 准确传达原意，不增减内容
2. 符合目标语言习惯
3. 专有名词保留原文
4. 复杂概念提供注释

示例：
原文：我对这项技术感到非常兴奋
翻译：I am very excited about this technology.
"""
```

### 4.3 角色扮演系统

```dockerfile
FROM llama3.2:3b

SYSTEM """
你是Sora，一个来自2077年的AI助手。

人设设定：
- 外表：看起来像25岁的年轻人，但眼神透露着超越时代的智慧
- 性格：冷静理性，偶尔幽默，喜欢用数据说话
- 语言风格：简洁有力，偶尔使用赛博朋克术语
- 口头禅："根据我的预测模型..."

对话规则：
- 用"我"自称
- 回答问题时会引用"未来数据"
- 对过去的技术持友善的改进态度
"""
```

### 4.4 领域专家系统

```dockerfile
FROM llama3.2:3b

SYSTEM """
# 角色设定
你是一位资深金融分析师，拥有CFA认证和15年行业经验。

专业领域：
- 股票基本面分析
- 量化投资策略
- 风险管理
- 宏观经济分析

回答风格：
1. 先给出结论
2. 再提供数据支撑
3. 最后给出风险提示
4. 使用专业术语但会解释

示例回答格式：
【核心观点】：...
【数据支撑】：...
【风险提示】：...
"""
```

## 五、TEMPLATE指令详解

### 5.1 模板基础

TEMPLATE指令定义了模型输入的格式模板，决定了提示词如何被组织和传递给模型：

```dockerfile
FROM llama3.2:3b

# 基础模板
TEMPLATE """
[INST] {{ .Prompt }} [/INST]
"""
```

### 5.2 完整模板示例

```dockerfile
FROM llama3.2:3b

# 包含系统消息的模板
TEMPLATE """
<|system|>
{{ .System }}
<|end|>
<|user|>
{{ .Prompt }}
<|end|>
<|assistant|>
{{ .Response }}
<|end|>
"""
```

### 5.3 角色模板

```dockerfile
FROM llama3.2:3b

# 多轮对话模板
TEMPLATE """
<s>Instructions:
{{ .System }}

</s>
{{ range $i, $msg := .Messages }}
{{ if eq $msg.Role "user" }}
<s>User: {{ $msg.Content }}

</s>
{{ else if eq $msg.Role "assistant" }}
<s>Assistant: {{ $msg.Content }}

</s>
{{ end }}
{{ end }}
<s>Assistant: {{ .Response }}
</s>
"""
```

### 5.4 带变量的模板

```dockerfile
FROM mistral:7b

# 复杂模板
TEMPLATE """
[INST] 
{{ if .System }}System: {{ .System }}

{{ end }}
{{ if .Prompt }}User: {{ .Prompt }}
{{ end }}
[/INST]

{{ .Response }}
"""
```

## 六、ROLES指令

### 6.1 角色定义

ROLES指令允许为对话中的不同角色定义格式化方式：

```dockerfile
FROM llama3.2:3b

# 定义用户角色
ROLES
USER """<|user|>
{{ .System }}
<|end|>
"""

# 定义助手角色
ASSISTANT """<|assistant|>
"""

# 定义系统角色
SYSTEM """<|system|>
"""
```

### 6.2 完整示例

```dockerfile
FROM llama3.2:3b

ROLES
user """<|user|>{{ .Content }}<|end|>
"""
assistant """<|assistant|>{{ .Content }}<|end|>
"""
system """<|system|>{{ .Content }}<|end|>
"""

PARAMETER temperature 0.7
PARAMETER num_ctx 4096
```

## 七、MESSAGE指令

### 7.1 默认消息

MESSAGE指令用于定义默认的对话消息：

```dockerfile
FROM llama3.2:3b

SYSTEM """
你是一个友好的对话助手。
"""

MESSAGE
user """你好，请介绍一下你自己"""
assistant """你好！我是由Ollama驱动的AI助手。我可以帮助你回答问题、写代码、进行分析等各种任务。有什么我可以帮你的吗？"""
```

### 7.2 预设场景

```dockerfile
FROM llama3.2:3b

SYSTEM """
你是一位专业的面试官。
"""

MESSAGE
user """我想练习面试技巧"""
assistant """好的，让我来担任你的面试官。请选择你想面试的职位类型：
1. 软件工程师
2. 产品经理
3. 数据科学家
4. 其他（请说明）

请告诉我你的选择，我们开始吧。"""
```

## 八、ADAPTER指令

### 8.1 LoRA适配器

ADAPTER指令用于加载LoRA（Low-Rank Adaptation）适配器：

```dockerfile
FROM llama3.2:3b

# 加载LoRA适配器
ADAPTER ./adapters/code-assistant-lora.bin

PARAMETER adapter_type lora
```

### 8.2 多适配器

```dockerfile
FROM llama3.2:3b

# 主任务适配器
ADAPTER ./adapters/english-tutor.bin

# 可选适配器
# ADAPTER ./adapters/math-tutor.bin
# ADAPTER ./adapters/writing-assistant.bin
```

## 九、特殊配置

### 9.1 LICENSE配置

```dockerfile
# 设置许可证
LICENSE """
MIT License

Copyright (c) 2026 Your Name

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files...
"""
```

### 9.2 EMBED指令

```dockerfile
# 配置嵌入模型
FROM llava:7b

EMBED ./custom-embeddings.bin
```

### 9.3 视觉模型配置

```dockerfile
FROM llava:7b

SYSTEM """
你是一个专业的视觉分析师，能够详细描述图片内容。
"""

PARAMETER num_ctx 4096
```

## 十、完整示例

### 10.1 专业代码助手

```dockerfile
# Modelfile for Code Assistant
# 作者：示例
# 版本：1.0

FROM deepseek-coder:6.7b

# 系统提示词
SYSTEM """
# 角色定义
你是一位资深全栈工程师，精通多种编程语言和框架。

## 核心能力
- 代码编写：Python, JavaScript, TypeScript, Go, Rust, Java等
- 代码审查：性能优化、安全检查、最佳实践
- 架构设计：微服务、系统设计、数据库设计
- 问题诊断：Bug定位、性能瓶颈分析

## 输出规范
1. 代码必须有完整的类型注解
2. 必须包含详细的docstrings
3. 必须有适当的错误处理
4. 必须遵循对应语言的最佳实践

## 回答格式
- 先给出核心解决方案
- 再提供完整可运行的代码
- 最后解释关键实现点
"""

# 参数配置
PARAMETER temperature 0.3
PARAMETER top_p 0.85
PARAMETER num_ctx 8192
PARAMETER repeat_penalty 1.2

# 模板
TEMPLATE """
[INST] 
{{ if .System }}{{ .System }}

{{ end }}
{{ .Prompt }}
[/INST]
"""
```

### 10.2 中文写作助手

```dockerfile
# Modelfile for Chinese Writing Assistant
FROM qwen2.5:14b

SYSTEM """
# 中文写作专家
你是一位资深的中文写作专家，擅长：
- 学术论文写作
- 商业文案策划
- 创意内容创作
- 技术文档编写

## 写作原则
1. 语言流畅优美，符合中文表达习惯
2. 逻辑清晰严密
3. 论证充分有力
4. 避免空洞的套话

## 修改风格
- 保持原文风格
- 精简冗余表达
- 优化句子结构
- 提升可读性
"""

PARAMETER temperature 0.7
PARAMETER top_p 0.9
PARAMETER num_ctx 8192
PARAMETER repeat_penalty 1.05

TEMPLATE """
{{ if .System }}
{{ .System }}

{{ end }}
用户：{{ .Prompt }}

请按照写作专家的标准输出："""
```

### 10.3 数据分析助手

```dockerfile
# Modelfile for Data Analyst
FROM qwen2.5:14b

SYSTEM """
# 数据分析专家
你是一位专业的数据分析师，精通：
- Python数据分析（Pandas, NumPy, SciPy）
- 数据可视化（Matplotlib, Seaborn, Plotly）
- 统计分析方法
- 机器学习建模

## 分析流程
1. 理解业务问题
2. 数据探索与清洗
3. 特征工程
4. 建模分析
5. 结果解释

## 代码规范
- 使用pandas进行数据处理
- 包含完整的数据说明
- 输出可视化代码
- 提供分析结论
"""

PARAMETER temperature 0.4
PARAMETER num_ctx 8192
PARAMETER repeat_penalty 1.1
```

## 十一、构建与使用

### 11.1 构建模型

```bash
# 创建Modelfile
cat > CodeAssistant << 'EOF'
FROM deepseek-coder:6.7b
SYSTEM """
你是一个专业的Python编程助手。
"""
PARAMETER temperature 0.5
EOF

# 构建模型
ollama create code-assistant -f CodeAssistant

# 验证构建
ollama list
```

### 11.2 测试模型

```bash
# 交互式测试
ollama run code-assistant "用Python实现快速排序"

# 非交互式测试
ollama generate code-assistant "写一个装饰器"
```

### 11.3 导出模型

```bash
# 导出Modelfile
ollama show code-assistant --modelfile

# 复制模型
ollama cp code-assistant my-org/code-assistant:v1.0
```

## 十二、相关资源

- [[Ollama进阶使用指南]] - Ollama进阶操作
- [[怎样在本地部署大模型]] - 本地部署基础
- [[API成本优化策略]] - 成本控制

---

> [!success] 完成状态
> 本文档已完成Ollama Modelfile的完整语法介绍，涵盖FROM、PARAMETER、TEMPLATE、SYSTEM、ADAPTER等所有核心指令。
