---
title: OpenClaw完整指南
date: 2026-04-18
tags:
  - OpenClaw
  - AI-Agent
  - 个人助手
  - 自托管
  - 多渠道
  - Pi-Framework
  - 记忆系统
  - 插件系统
  - Docker
  - 配置管理
categories:
  - 小龙虾生态
  - OpenClaw核心
alias: OpenClaw Complete Guide
---

# OpenClaw 完整指南

> [!note] 文档信息
> 本文档为 OpenClaw 框架的完整使用指南，涵盖架构原理、Pi Framework、自托管部署、多渠道集成、记忆系统等核心模块。

---

## 关键词速览

| 关键词 | 说明 |
|--------|------|
| OpenClaw | 开源 AI Agent 运行时框架 |
| Pi Framework | 底层极简 Agent 开发框架 |
| Gateway | 消息路由与渠道适配网关 |
| ClawHub | 3200+ 技能插件市场 |
| 自托管 | 本地部署，数据自主控制 |
| 多渠道 | 支持 20+ 聊天平台 |
| 记忆系统 | 跨会话持久化上下文 |
| Docker | 容器化部署方案 |
| Plugin | 插件扩展机制 |
| Session | 会话管理与分支 |

---

## 一、OpenClaw 概述

### 1.1 项目起源与定位

OpenClaw 是一个由 Mario Zechner（libGDX 游戏引擎创始人）主导的开源项目，旨在打造一个**完全自托管的个人 AI 助手运行时**。项目以小龙虾（lobster）为精神图腾，象征着在信息海洋中灵活穿行、适应性强且易于繁殖的特质。

截至 2026 年 4 月，OpenClaw 已累计获得超过 **35.9 万 GitHub 星标**，成为开源 AI Agent 领域最具影响力的项目之一。其核心理念可以用项目口号概括：

> *"Any OS. Any Platform. The lobster way."*

### 1.2 核心功能矩阵

| 功能模块 | 详细说明 |
|----------|----------|
| **多渠道接入** | Discord、Telegram、WhatsApp、Slack、Signal、Matrix、邮件等 20+ 平台 |
| **自托管部署** | Docker 一键部署，数据完全本地化 |
| **工具调用** | 代码执行、文件读写、bash 命令、网页浏览 |
| **记忆系统** | Markdown 文件存储的跨会话持久化上下文 |
| **多 Agent** | 支持隔离工作区与独立会话管理 |
| **插件生态** | ClawHub 3200+ 插件可扩展 |
| **多模型支持** | Anthropic、OpenAI、Google、xAI、Groq 等主流 LLM |

---

## 二、Pi Framework 详解

### 2.1 什么是 Pi Framework

Pi Framework 是 OpenClaw 的底层运行时核心，采用了**极简主义设计哲学**。与 LangChain、AutoGen 等重型框架不同，Pi 仅提供 4 个核心工具就被设计成一个功能完整的 Agent 运行时。

**Pi 的四大核心工具**：

```python
# Pi Framework 内置工具集
tools = [
    "read",    # 文件读取
    "write",   # 文件写入
    "edit",    # 文件编辑
    "bash"     # Shell 命令执行
]
```

### 2.2 Pi 的设计原则

Pi Framework 遵循以下核心原则：

1. **极简优先**：系统提示词控制在 1000 tokens 以内
2. **工具正交**：每个工具职责单一，无功能重叠
3. **显式优于隐式**：所有操作都通过工具调用完成，无魔法发生
4. **可预测性**：相同的输入总是产生确定性的行为

### 2.3 Pi 与 OpenClaw 的关系

```
┌─────────────────────────────────────┐
│        OpenClaw Gateway              │
│  (消息路由 / 渠道适配 / 会话管理)      │
└─────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│         Pi Agent SDK                │
│  (内置于 OpenClaw 进程中)            │
│  - 4 Core Tools                     │
│  - System Prompt < 1000 tokens      │
│  - Multi-Model Support              │
└─────────────────────────────────────┘
```

---

## 三、自托管部署

### 3.1 为什么选择自托管

自托管（Self-hosted）部署是 OpenClaw 区别于商业 AI 服务的核心优势：

| 对比维度 | 自托管 OpenClaw | 云服务（如 ChatGPT） |
|----------|-----------------|---------------------|
| 数据隐私 | 完全自主控制 | 服务商可访问 |
| 成本 | 一次性硬件投入 | 按订阅付费 |
| 定制化 | 完全开放源码 | 受限的 API |
| 离线可用 | 完全支持 | 不可用 |
| 延迟 | 本地网络延迟 | 依赖云端 |

### 3.2 Docker 部署方案

OpenClaw 提供官方 Docker 镜像，部署步骤如下：

```bash
# 拉取官方镜像
docker pull openclaw/openclaw:latest

# 创建数据目录
mkdir -p ~/openclaw/{config,data,plugins}

# 启动容器
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/openclaw/config:/app/config \
  -v ~/openclaw/data:/app/data \
  -v ~/openclaw/plugins:/app/plugins \
  openclaw/openclaw:latest
```

### 3.3 源码部署

对于需要深度定制的用户，可以从源码构建：

```bash
# 克隆仓库
git clone https://github.com/openclaw-project/openclaw.git
cd openclaw

# 安装依赖
pip install -e ".[dev]"

# 运行
python -m openclaw
```

---

## 四、多渠道集成

### 4.1 支持的平台一览

OpenClaw 通过插件机制支持丰富的聊天平台：

| 平台 | 协议类型 | 状态 |
|------|----------|------|
| Discord | WebSocket | 官方支持 |
| Telegram | Long Polling / Webhook | 官方支持 |
| WhatsApp | WhatsApp Web Protocol | 官方支持 |
| Slack | WebSocket | 官方支持 |
| Signal | Signal Protocol | 社区支持 |
| Matrix | Matrix Protocol | 社区支持 |
| Email (SMTP/IMAP) | Email Protocol | 社区支持 |
| IRC | IRC Protocol | 社区支持 |

### 4.2 Telegram 集成示例

```yaml
# config.yaml
channels:
  telegram:
    enabled: true
    bot_token: "YOUR_BOT_TOKEN"
    api_id: "YOUR_API_ID"
    api_hash: "YOUR_API_HASH"
    allowed_users:
      - user_id: 123456789
        name: "主用户"
        workspace: "default"
```

### 4.3 Discord 集成示例

```yaml
channels:
  discord:
    enabled: true
    bot_token: "YOUR_DISCORD_BOT_TOKEN"
    allowed_guilds:
      - guild_id: 987654321
        name: "我的服务器"
        channels:
          - channel_id: 111222333
            workspace: "default"
```

---

## 五、记忆系统

### 5.1 记忆架构

OpenClaw 采用**静态记忆系统**，以 Markdown 文件为核心存储介质：

```
记忆存储结构：
├── memory/
│   ├── long_term/
│   │   ├── facts.md       # 事实性记忆
│   │   ├── preferences.md # 用户偏好
│   │   └── knowledge.md   # 领域知识
│   ├── short_term/
│   │   └── session_*.md   # 当前会话
│   └── working/
│       └── context.md     # 工作上下文
```

### 5.2 记忆检索机制

OpenClaw 在每次对话前会将相关记忆注入上下文：

```python
# 记忆检索流程
1. 解析当前用户消息
2. 提取关键实体和意图
3. 在 long_term/ 目录中搜索相关记忆
4. 按相关性排序
5. 选取 Top-N 条记忆注入系统提示
```

---

## 六、插件系统

### 6.1 ClawHub 生态

ClawHub 是 OpenClaw 的官方技能市场，提供 **3200+** 插件供用户安装：

| 类别 | 插件数量 | 代表插件 |
|------|----------|----------|
| 生产力 | 500+ | 日程管理、邮件处理、任务跟踪 |
| 开发 | 400+ | Git 操作、代码审查、API 测试 |
| 信息获取 | 600+ | 天气查询、新闻聚合、股票行情 |
| 娱乐 | 300+ | 游戏、音乐推荐、段子生成 |
| AI 工具 | 400+ | 图像生成、翻译、OCR |

### 6.2 插件安装

```bash
# 通过 CLI 安装插件
openclaw plugin install weather
openclaw plugin install github
openclaw plugin install notion

# 查看已安装插件
openclaw plugin list
```

---

## 七、配置详解

### 7.1 完整配置结构

```yaml
# config.yaml 完整配置示例
openclaw:
  version: "2.x"
  
# 语言模型配置
llm:
  provider: "anthropic"  # anthropic | openai | google | groq
  model: "claude-sonnet-4-20250514"
  api_key: "${ANTHROPIC_API_KEY}"
  max_tokens: 4096
  temperature: 0.7

# 渠道配置
channels:
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT_TOKEN}"
    
# 记忆配置  
memory:
  type: "markdown"  # markdown | sqlite | vector
  long_term_dir: "./memory/long_term"
  max_context记忆: 10

# 工作区配置
workspaces:
  default:
    name: "默认工作区"
    system_prompt: "你是一个helpful的AI助手"
```

---

## 八、进阶技巧

### 8.1 多 Agent 协作

OpenClaw 支持配置多个隔离的 Agent 实例：

```yaml
agents:
  - name: "研究助手"
    workspace: "research"
    model: "claude-sonnet-4-20250514"
    
  - name: "代码助手"
    workspace: "coding"
    model: "gpt-4-turbo"
```

### 8.2 定时任务

通过插件实现定时执行：

```yaml
# crontab 配置示例
schedules:
  - name: "每日早报"
    cron: "0 8 * * *"
    action: "plugin:news_digest/send_morning_brief"
    
  - name: "健康提醒"
    cron: "0 12 * * 1-5"
    action: "plugin:health/remind_water"
```

---

## 九、常见问题

> [!faq] Q1: OpenClaw 与 Hermes Agent 有什么区别？
> OpenClaw 侧重于多渠道接入和插件生态，采用 Gateway 架构；Hermes Agent 侧重于自进化和持久记忆，采用 SQLite 本地存储。

> [!faq] Q2: 支持中文吗？
> 完全支持。通过配置合适的 LLM（如 Claude、GPT-4）即可处理中文对话。

> [!faq] Q3: 需要 GPU 吗？
> 不需要。OpenClaw 本身是消息路由和工具调用的运行时，不运行模型。LLM 调用通过 API 远程完成。

---

## 十、相关文档

- [[OpenClaw架构解析]] - 深入了解 OpenClaw 技术架构
- [[Pi_Framework深度解析]] - Pi Framework 极简设计
- [[OpenClaw安装部署]] - 详细部署指南
- [[OpenClaw配置详解]] - 配置项详解
- [[OpenClaw多平台集成]] - 各平台接入详解
- [[OpenClaw记忆系统]] - 记忆系统原理
- [[OpenClaw插件开发]] - 开发自己的插件
- [[OpenClaw高级用法]] - 高级技巧与最佳实践
- [[Hermes_Agent详解]] - Hermes Agent 详解
- [[Hermes_vs_OpenClaw对比]] - 两者对比分析

---

*文档更新于 2026年4月18日*
