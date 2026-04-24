---
title: OpenClaw 完整指南
date: 2026-04-24
tags:
  - OpenClaw
  - 完整指南
  - 功能介绍
  - Pi-Framework
  - 记忆系统
  - 插件
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 全面介绍 OpenClaw 的所有功能，从 Pi Framework 到多平台集成，带你彻底玩转这个框架。
---

# OpenClaw 完整指南

> 这篇是 OpenClaw 的全面介绍。安装好了？或者只是想了解能干什么？看这篇就够了。

---

## 先说最重要的几个概念

### OpenClaw 是什么？

OpenClaw 是一个**个人 AI 助手的运行环境**。你可以把它理解成一个"智能插座"，接上各种聊天平台（微信、Telegram、Discord），再接上 AI 大脑（Claude、GPT），就能让它们对话了。

它的口号是：

> **"Any OS. Any Platform. The lobster way."**
> 翻译：什么系统都能跑，什么平台都能接，小龙虾的方式。

### 几个核心概念

| 概念 | 人话解释 | 类比 |
|------|----------|------|
| **Pi Framework** | OpenClaw 的"大脑"，负责思考和决策 | 人的大脑 |
| **Gateway** | "路由器"，把各平台的消息转发给大脑 | 人的耳朵和嘴 |
| **Channel** | 聊天平台适配器（微信/Telegram/Discord） | 各种通讯设备 |
| **Memory** | 记忆系统，让 AI 记住之前聊过什么 | 人的记忆 |
| **Skill/Plugin** | 技能/插件，给 AI 增加新能力 | 人的新技能 |

---

## Pi Framework：小而美的 AI 大脑

### 为什么叫"Pi"？

"Pi"（π）是数学里的圆周率——无限不循环小数。开发者用它命名，大概是想说：这个框架简单到不能再简单，但又功能无限。

### Pi 的设计哲学

Pi Framework 的核心理念是：**少即是多**。

其他 AI Agent 框架可能给你 100 个工具，但 Pi 只给你 **4 个核心工具**：

| 工具 | 能干啥 | 举个例子 |
|------|--------|----------|
| **read** | 读取文件 | 读你的笔记、配置文件 |
| **write** | 写入文件 | 把结果保存到文件 |
| **edit** | 编辑文件 | 修改某个文件的内容 |
| **bash** | 执行命令 | 运行终端命令 |

就这 4 个？但这就够了。

> 想象一下，给你 100 把不同的瑞士军刀，你可能记不住每个怎么用。但如果只有一把小刀，你反而能用到极致。

### 为什么系统提示词要 < 1000 tokens？

Token 可以理解为 AI 处理文字的"计量单位"。提示词越短，消耗越少，响应越快，还能留更多空间给对话内容。

Pi 的目标是把系统提示词压到 **1000 tokens 以内**，这需要精心设计，就像写高考作文——每个字都得有用。

---

## 多平台接入：一条消息，多端同步

### 支持哪些平台？

OpenClaw 像个"万能插头"，支持超多聊天平台：

| 平台 | 状态 | 说明 |
|------|------|------|
| **Telegram** | ✅ 官方支持 | 最推荐，Bot 开发门槛低 |
| **Discord** | ✅ 官方支持 | 适合社区、服务器 |
| **WhatsApp** | ✅ 官方支持 | 适合跨境沟通 |
| **Slack** | ✅ 官方支持 | 企业用得多 |
| **Signal** | ✅ 官方支持 | 注重隐私 |
| **Microsoft Teams** | ✅ 官方支持 | 企业办公 |
| **微信** | ⚠️ 企业版 | 个人用有门槛 |
| **飞书** | ⚠️ 企业版 | 同上 |
| **Matrix** | ✅ 社区支持 | 去中心化协议 |
| **IRC** | ✅ 社区支持 | 老牌聊天协议 |
| **Email** | ✅ 社区支持 | 邮件也能接 |

### 怎么接 Telegram？（最简单）

1. 去 @BotFather 创建一个 Bot，获得 Token
2. 去 @userinfobot 获取你的用户 ID
3. 编辑配置：

```yaml
channels:
  telegram:
    enabled: true
    bot_token: "123456:ABC-xxxxx"   # BotFather 给的
    allowed_users:
      - 123456789                      # 你的用户 ID
```

4. 重启服务，Bot 就能用了。

### 怎么接 Discord？

Discord 比 Telegram 稍微复杂一点，但功能也更强大。

**第一步**：去 Discord Developer Portal 创建 Application

1. 访问 https://discord.com/developers/applications
2. 点 "New Application" 创建应用
3. 点 "Bot" 页面
4. 点 "Reset Token" 获取 Bot Token

**第二步**：配置权限

在 Bot 页面，开启这些 Intent：
- SERVER MEMBERS INTENT
- MESSAGE CONTENT INTENT

**第三步**：邀请 Bot 到服务器

1. 去 OAuth2 → URL Generator
2. Scopes 勾选 `bot`
3. Bot Permissions 勾选需要的权限
4. 复制生成的 URL，浏览器打开，添加到你的服务器

**第四步**：配置 OpenClaw

```yaml
channels:
  discord:
    enabled: true
    bot_token: "MTEyyyy.BBBBB.xxxxx"   # 你的 Bot Token
    allowed_guilds:
      - 123456789                       # 服务器 ID
```

---

## 记忆系统：让 AI 记住你是谁

### 为什么需要记忆？

普通 AI 每次对话都是"失忆"的——你说"帮我查下上次那个文件"，它一脸懵逼。

OpenClaw 的记忆系统让 AI 能跨会话记住：
- 你是谁
- 你喜欢什么
- 之前聊过什么
- 你的工作习惯

### 记忆是怎么存的？

OpenClaw 用 **Markdown 文件**存记忆。打开你的 `~/.openclaw/` 目录，会看到这样的结构：

```
memory/
├── long_term/           # 长期记忆
│   ├── facts.md         # 事实：比如"我叫张三"
│   ├── preferences.md    # 偏好：比如"喜欢用中文回复"
│   └── knowledge.md     # 知识：比如"我是个程序员"
├── short_term/          # 短期记忆（当前会话）
│   └── session_xxx.md
└── working/            # 工作上下文
    └── context.md
```

### 怎么手动添加记忆？

创建或编辑 `memory/long_term/facts.md`：

```markdown
# 关于用户的事实

- 名字叫小明
- 是一名全栈工程师
- 主要使用 JavaScript 和 Python
- 在北京工作
- 喜欢喝咖啡
```

AI 下次对话时，会自动读取这些内容。

### 记忆是怎么工作的？

每次你发消息，OpenClaw 会：

1. **检索**：在记忆文件里搜索相关内容
2. **注入**：把最相关的记忆塞到 AI 的上下文里
3. **思考**：AI 根据记忆理解你的问题
4. **回复**：给出个性化的回答

---

## 技能系统：给 AI 装插件

### 什么是技能？

技能（Skill）就像给 AI 装一个 App。比如：
- 装个"天气技能" → AI 就能查天气
- 装个"翻译技能" → AI 就能翻译
- 装个"GitHub 技能" → AI 就能帮你管代码

### ClawHub：技能市场

ClawHub 是 OpenClaw 的官方技能市场，有 **3200+ 个技能**可选。

访问地址：https://clawhub.openclaw.ai

### 怎么装技能？

**命令行安装**：

```bash
# 搜索技能
openclaw skill search "天气"

# 安装
openclaw skill install @openclaw/weather

# 查看已安装
openclaw skill list

# 卸载
openclaw skill uninstall @openclaw/weather
```

**或者用配置文件**：

```yaml
skills:
  install:
    - name: "@openclaw/weather"
    - name: "@openclaw/github"
    - name: "@openclaw/notion"
```

### 热门技能推荐

| 技能 | 能干啥 | 适合谁 |
|------|--------|--------|
| `@openclaw/weather` | 查天气 | 所有人 |
| `@openclaw/github` | 管 GitHub | 开发者 |
| `@openclaw/notion` | 读写 Notion | 知识管理党 |
| `@openclaw/calendar` | 管日程 | 日理万机党 |
| `@openclaw/translate` | 翻译 | 多语言党 |
| `@openclaw/image-gen` | AI 画图 | 创作者 |

---

## 配置详解

### 完整配置示例

```yaml
# config.yaml

# ===== AI 模型配置 =====
llm:
  provider: "anthropic"                        #anthropic/openai/google/groq
  model: "claude-sonnet-4-20250514"          # 具体用哪个模型
  api_key: "${ANTHROPIC_API_KEY}"             # 用环境变量更安全
  max_tokens: 4096                            # 最大输出长度
  temperature: 0.7                             # 创造性，0-1，越高越随机

# ===== 聊天平台配置 =====
channels:
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT_TOKEN}"
    allowed_users:
      - 123456789

  discord:
    enabled: false
    bot_token: "${DISCORD_BOT_TOKEN}"

# ===== 记忆配置 =====
memory:
  type: "markdown"                           # markdown/sqlite/vector
  long_term_dir: "./memory/long_term"
  max_context: 10                            # 最多注入多少条记忆

# ===== 系统提示词 =====
system:
  prompt: |
    你是一个友好、helpful 的中文 AI 助手。
    请用简洁、口语化的方式回答问题。
    如果不确定就说"我不知道"，别瞎编。

# ===== 技能配置 =====
skills:
  install:
    - "@openclaw/weather"
    - "@openclaw/translate"

# ===== 安全设置 =====
security:
  allowed_tools: ["read", "write"]          # 只允许特定工具
  denied_tools: ["bash"]                       # 禁用危险工具
```

### 配置项说明

| 配置项 | 说明 | 推荐值 |
|--------|------|--------|
| `provider` | AI 服务商 | `anthropic`（性价比高） |
| `temperature` | 创造性 | 0.7（平衡） |
| `max_tokens` | 最大输出 | 4096（够用） |
| `allowed_users` | 白名单 | 填你的用户 ID |
| `allowed_tools` | 允许的工具 | 按需配置 |

---

## 工作区：多个 AI 分身

### 什么是工作区？

工作区（Workspace）就像给 AI 开多个"房间"。每个房间的 AI 记忆、配置、对话都是独立的。

**使用场景**：
- 工作用 AI 和生活用 AI 分开
- 给不同的项目配不同的 AI 角色
- 测试不同的 AI 配置

### 配置多工作区

```yaml
workspaces:
  default:
    name: "默认助手"
    prompt: "你是一个通用助手"

  developer:
    name: "开发者助手"
    prompt: "你是一个资深程序员，擅长代码审查和架构设计"
    llm:
      model: "gpt-4-turbo"                    # 这个工作区用 GPT

  writer:
    name: "写作助手"
    prompt: "你是一个专业文案，擅长写作和内容创作"
    llm:
      model: "claude-sonnet-4-20250514"
```

### 工作区切换

在 Telegram 里：
```
/workspace developer    # 切换到开发者助手
/workspace writer      # 切换到写作助手
```

---

## 常用命令

### OpenClaw CLI 命令

```bash
# 启动
openclaw gateway

# 引导配置
openclaw onboard

# 打开控制面板
openclaw dashboard

# 技能管理
openclaw skill list
openclaw skill install <name>
openclaw skill uninstall <name>

# 更新
openclaw update

# 查看状态
openclaw status

# 日志
openclaw logs

# 诊断问题
openclaw doctor
```

---

## 常见问题

### Q: 怎么让 AI 用中文回答？

在 `system.prompt` 里明确说：

```yaml
system:
  prompt: "请始终用中文回答问题。"
```

### Q: AI 回复太慢了怎么办？

1. 用 `groq` provider（免费，速度快）：
```yaml
llm:
  provider: "groq"
  model: "llama-3.3-70b-versatile"
```

2. 减少 `max_tokens`

3. 用更小的模型，比如 `claude-3-haiku`

### Q: 怎么让 AI 记住更多东西？

1. 多往 `memory/long_term/` 里写
2. 定期整理记忆文件
3. 安装 `@openclaw/active-memory` 插件（自动记住重要对话）

### Q: 能同时接多个平台吗？

完全可以！只要在 `channels` 里启用多个就行。

```yaml
channels:
  telegram:
    enabled: true
  discord:
    enabled: true
  slack:
    enabled: true
```

### Q: 数据存在哪？

默认在 `~/.openclaw/` 目录：
- `~/.openclaw/config/` - 配置文件
- `~/.openclaw/data/` - 运行数据
- `~/.openclaw/memory/` - 记忆文件
- `~/.openclaw/plugins/` - 插件

---

## 下一步

学会了基础？继续探索：

| 教程 | 内容 | 难度 |
|------|------|------|
| [[OpenClaw架构解析]] | 技术内幕，怎么工作的 | ⭐⭐ |
| [[OpenClaw多平台集成]] | 接更多平台 | ⭐ |
| [[OpenClaw记忆系统]] | 记忆系统详解 | ⭐⭐ |
| [[OpenClaw插件开发]] | 自己写插件 | ⭐⭐⭐ |
| [[Pi_Framework深度解析]] | Pi 框架设计哲学 | ⭐⭐⭐ |
| [[Hermes_Agent详解]] | 另一个框架 | ⭐⭐ |
| [[Hermes_vs_OpenClaw对比]] | 选哪个更好 | ⭐ |

---

**文档状态**：完整指南  
**更新时间**：2026年4月
