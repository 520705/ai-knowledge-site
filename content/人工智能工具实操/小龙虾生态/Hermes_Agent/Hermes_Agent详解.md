---
title: Hermes Agent 详解
date: 2026-04-24
tags:
  - Hermes-Agent
  - Nous-Research
  - 自进化
  - AI-Agent
  - SQLite
  - 技能生成
  - RLHF
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 深入了解 Hermes Agent，这是另一个主打"越用越聪明"的 AI Agent 框架，和 OpenClaw 风格不同。
---

# Hermes Agent 详解

> Hermes Agent 是一个和 OpenClaw 定位不同的 AI Agent 框架。它的核心理念是"越用越聪明"——随着使用，它会自动学习和进化。

---

## Hermes Agent 是什么？

### 一句话解释

**Hermes Agent = 会自己学习的 AI 助手**

和 OpenClaw 不同，Hermes 的最大特点是**自进化**——它会从你跟它的对话中学习，逐渐变得更懂你。

### 名字的由来

Hermes（赫尔墨斯）是希腊神话里的信使之神：
- 跑得最快，传递消息
- 聪明伶俐，灵活机智
- 沟通人神，无所不能

这和 AI Agent 作为"智能信使"的定位很搭。

### 谁做的？

**Nous Research**，一家专注于 AI 自进化研究的创业公司。

他们觉得现在的 AI Agent 太"傻"了——每次对话都是独立的，学不到东西。所以他们做了 Hermes，让 AI 真正能从交互中学习。

---

## 核心特点

### 和 OpenClaw 的主要区别

| 特性 | Hermes Agent | OpenClaw |
|------|---------------|-----------|
| **核心理念** | 越用越聪明 | 多平台接入 |
| **记忆系统** | SQLite + 智能检索 | Markdown 文件 |
| **自进化** | ✅ AI 自动生成技能 | ❌ 手动写插件 |
| **工具数量** | 40+ 内置 | 4 核心 + 插件 |
| **多渠道** | 5 个平台 | 20+ 平台 |
| **MLOps** | ✅ 支持 RLHF | ❌ 不支持 |

### Hermes 的三大杀手锏

#### 1. 持久记忆（比 OpenClaw 更强）

OpenClaw 用 Markdown 文件存记忆，Hermes 用 **SQLite 数据库 + FTS5 全文检索**。

**好处**：
- 检索速度更快（毫秒级）
- 支持语义搜索（找相关的不只是关键词）
- 数据规模更大（能存几十万条记忆）

#### 2. 自进化技能（独门绝技）

当 Hermes 帮你解决了一个复杂问题，它会：
1. 分析这个问题是怎么解决的
2. 把解决方法"提炼"成一个技能
3. 下次遇到类似问题，自动调用这个技能

```python
# 例子：你问了个复杂问题
"帮我分析一下这个 Python 代码的性能问题"

# Hermes 解决了，然后...
# 自动生成一个技能：
{
    "name": "python_performance_analysis",
    "trigger": "代码性能分析",
    "implementation": "...",
    "confidence": 0.8
}

# 下次你再说"帮我看看这段代码慢不慢"
# Hermes 会自动调用这个技能
```

#### 3. MLOps 能力（适合研究者）

Hermes 可以收集你和它的对话，导出成 RLHF 训练数据。

**有什么用？**
- 用这些数据训练你自己的模型
- 让模型更懂你的偏好
- 做 AI 研究

---

## 技术架构

### 整体结构

```
┌─────────────────────────────────────────────────────────┐
│                     用户界面层                            │
│          CLI / Web / Telegram / Discord / Slack         │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                   Hermes 核心运行时                        │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │ 技能引擎 (Skill Engine)                          │  │
│  │ - 问题分析 → 技能生成 → 技能验证 → 技能入库       │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │ 记忆管理器 (Memory Manager)                      │  │
│  │ - SQLite + FTS5 存储                            │  │
│  │ - 语义检索 / 重要性排序 / 自动压缩               │  │
│  └─────────────────────────────────────────────────┘  │
│                                                         │
│  ┌─────────────────────────────────────────────────┐  │
│  │ MLOps 流水线                                    │  │
│  │ - 轨迹收集 / 奖励计算 / 分布式训练               │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────┬───────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────┐
│                      数据层                              │
│      SQLite 数据库 │ 向量存储 │ 文件系统 (技能/配置)      │
└─────────────────────────────────────────────────────────┘
```

### 记忆系统详解

#### 为什么用 SQLite？

**SQLite 的优势**：
- 单文件数据库，不用单独装服务
- 查询速度快
- 支持全文搜索（FTS5）
- 数据可靠（ACID）

**对比 OpenClaw 的 Markdown**：

| 维度 | Hermes (SQLite) | OpenClaw (Markdown) |
|------|-----------------|---------------------|
| 检索速度 | 毫秒级 | 秒级 |
| 检索方式 | 语义相似度 | 关键词匹配 |
| 数据规模 | 百万级 | 万级 |
| 精确匹配 | 支持 | 困难 |

#### FTS5 全文搜索

FTS5 是 SQLite 的全文搜索扩展，让 Hermes 能快速找到相关记忆：

```sql
-- 创建全文索引表
CREATE VIRTUAL TABLE memories USING fts5(
    content,
    context,
    importance,
    tokenize='porter unicode61'
);

-- 搜索"用户喜欢喝咖啡"
SELECT * FROM memories
WHERE memories MATCH 'coffee OR 咖啡'
ORDER BY bm25(memories)
LIMIT 10;
```

#### 记忆分类

| 类型 | 说明 | 存储位置 |
|------|------|----------|
| **事实记忆** | "我叫张三" | 长期存储 |
| **偏好记忆** | "我喜欢喝美式" | 长期存储 |
| **上下文记忆** | 当前会话 | 短期存储 |
| **技能记忆** | 学会的技能 | 永久存储 |

### 技能生成流程

```
用户提问
    │
    ▼
问题分析
    │
    ├─── 这是什么问题类型？
    ├─── 需要哪些步骤解决？
    └─── 有什么可复用的模式？
    │
    ▼
解决方案执行
    │
    ▼
技能生成（如果问题值得学习）
    │
    ├─── 提取问题模式
    ├─── 泛化解决步骤
    ├─── 定义触发条件
    └─── 验证有效性
    │
    ▼
技能入库
    │
    ▼
下次遇到类似问题
    │
    ▼
自动调用已学到的技能 ⚡
```

---

## 内置工具（40+）

Hermes 开箱即用包含 40+ 个工具，分成这些类别：

### 信息检索类

| 工具 | 功能 | 例子 |
|------|------|------|
| `web_search` | 搜索网页 | 搜"今天天气" |
| `wiki_lookup` | 查维基百科 | 查"什么是 AI" |
| `code_search` | 代码搜索 | 找某个函数的实现 |
| `academic_search` | 学术论文搜索 | 找相关论文 |

### 文件操作类

| 工具 | 功能 |
|------|------|
| `file_read` | 读文件 |
| `file_write` | 写文件 |
| `file_search` | 搜索文件 |
| `file_convert` | 文件格式转换 |

### 代码执行类

| 工具 | 功能 |
|------|------|
| `python_exec` | 执行 Python 代码 |
| `bash` | 执行 shell 命令 |
| `git_operations` | Git 操作 |
| `docker_exec` | Docker 操作 |

### 通信类

| 工具 | 功能 |
|------|------|
| `send_email` | 发邮件 |
| `send_message` | 发消息 |
| `calendar` | 日历管理 |
| `notifications` | 发送通知 |

### 数据处理类

| 工具 | 功能 |
|------|------|
| `data_analysis` | 数据分析 |
| `csv_process` | CSV 处理 |
| `json_transform` | JSON 转换 |
| `sql_query` | SQL 查询 |

### 媒体处理类

| 工具 | 功能 |
|------|------|
| `image_process` | 图片处理 |
| `audio_transcribe` | 语音转文字 |
| `video_extract` | 视频提取 |
| `ocr` | 文字识别 |

---

## 安装和配置

### 安装 Hermes

```bash
# 官方安装脚本（自动安装依赖）
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash

# 安装完成后配置
hermes setup

# 选择 AI 提供商
hermes model

# 启动！
hermes
```

### 配置文件

编辑 `~/.hermes/config.yaml`：

```yaml
# AI 模型配置
model:
  provider: "anthropic"
  model: "claude-sonnet-4-20250514"
  api_key: "${ANTHROPIC_API_KEY}"

# 记忆配置
memory:
  type: "sqlite_fts5"
  db_path: "~/.hermes/memory.db"
  max_items: 10000

# 技能配置
skills:
  auto_generate: true      # 自动生成技能
  validation: true         # 验证技能有效性
  confidence_threshold: 0.7  # 置信度阈值

# 渠道配置
channels:
  telegram:
    enabled: false
  discord:
    enabled: false

# MLOps 配置
mlops:
  trajectory_collection: true  # 收集训练轨迹
  batch_processing: true        # 批量处理
  rlhf_export: true              # 导出 RLHF 数据
```

### Docker 部署

```bash
# 拉取镜像
docker pull nousresearch/hermes-agent:latest

# 运行
docker run -d \
  --name hermes \
  -p 18792:18792 \
  -v ~/.hermes:/root/.hermes \
  -e ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}" \
  nousresearch/hermes-agent:latest
```

---

## 常用命令

```bash
# 启动 CLI
hermes

# 技能管理
hermes skills list          # 列出所有技能
hermes skills show <name>   # 查看技能详情
hermes skills export <name> # 导出技能

# 记忆管理
hermes memory search "关键词"   # 搜索记忆
hermes memory list             # 列出记忆
hermes memory delete <id>      # 删除记忆

# 定时任务
hermes schedule add "0 8 * * *" --task daily-brief  # 添加定时任务
hermes schedule list           # 列出定时任务

# 多 Agent
hermes spawn --count 3 --task research  # 启动多个子 Agent

# 更新
hermes update
```

---

## 和 OpenClaw 怎么选？

### 选 Hermes 如果...

- ✅ 想让 AI 越用越懂你
- ✅ 需要强大的记忆和检索
- ✅ 在做 AI 研究，需要 RLHF 数据
- ✅ 需要 AI 自动生成技能

### 选 OpenClaw 如果...

- ✅ 需要接多个聊天平台
- ✅ 想要丰富的插件生态（3200+）
- ✅ 想要稳定的多渠道支持
- ✅ 社区更大，教程更多

### 我的建议

**新手先用 OpenClaw**，因为：
- 安装更简单
- 社区更大，遇到问题好找人问
- 插件生态丰富，装上就能用

**有经验了再试 Hermes**，因为：
- 功能更强大
- 自进化能力独特
- 适合深度定制

---

## 适用场景

### Hermes 最擅长的

| 场景 | 为什么适合 |
|------|------------|
| **个人知识助手** | 记忆强大，越用越懂你 |
| **研究伴侣** | 能自动整理研究资料 |
| **代码学习** | 能记住你学过的代码模式 |
| **长期项目** | 跨会话积累知识 |

### Hermes 不擅长的

| 场景 | 为什么不适合 |
|------|------------|
| **快速多渠道部署** | 渠道比 OpenClaw 少 |
| **需要丰富插件** | 生态还没 OpenClaw 大 |
| **简单客服机器人** | 有点大材小用 |

---

## 常见问题

### Q: Hermes 和 OpenClaw 能一起用吗？

技术上可以，但没必要。它们功能有重叠，同时跑会浪费资源。

**建议**：选择一个，用熟。

### Q: 自进化会把我的数据用于训练吗？

不会。Hermes 的自进化只在你本地发生，数据不会上传到服务器。

MLOps 功能需要你主动导出数据，用不用、怎么用，都是你自己决定。

### Q: Hermes 的记忆会无限增长吗？

不会。Hermes 有遗忘机制：

1. **重要性评分**：不常用的记忆会降权
2. **自动压缩**：当记忆太多，会自动总结压缩
3. **手动清理**：你可以随时删除不需要的记忆

```bash
# 手动清理记忆
hermes memory prune --before 30d  # 删除30天前的
hermes memory prune --importance low  # 删除低重要性的
```

### Q: 技能生成会把我的对话暴露吗？

不会。技能生成是在本地进行的，没有数据上传。

而且生成的是"模式"而不是原始对话，不会泄露隐私。

---

## 下一步

想深入了解？

| 教程 | 内容 |
|------|------|
| [[Hermes_vs_OpenClaw对比]] | 详细对比，帮你选择 |
| [[OpenClaw完整指南]] | OpenClaw 完整指南 |
| [[OpenClaw安装部署]] | 怎么安装 OpenClaw |
| [[小龙虾生态]] | 小龙虾生态整体介绍 |

---

**文档状态**：Hermes Agent 详解  
**更新时间**：2026年4月
