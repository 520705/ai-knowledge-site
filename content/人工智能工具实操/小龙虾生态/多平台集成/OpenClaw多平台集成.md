---
title: OpenClaw 多平台集成完全指南
date: 2026-04-24
tags:
  - OpenClaw
  - Telegram
  - Discord
  - WhatsApp
  - Slack
  - 集成
  - 机器人
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 手把手教你在 Telegram、Discord、Slack、WhatsApp 等平台接入 OpenClaw，让 AI 助手无处不在。
---

# OpenClaw 多平台集成完全指南

> 想让 AI 助手在微信、Telegram、Discord 上都能用？这篇就是讲这个的。

---

## 先搞清楚：OpenClaw 支持哪些平台？

OpenClaw 支持超多平台，可以用"万能插头"来形容：

| 平台 | 推荐度 | 难度 | 说明 |
|------|--------|------|------|
| **Telegram** | ⭐⭐⭐⭐⭐ | 简单 | 最推荐，门槛最低 |
| **Discord** | ⭐⭐⭐⭐ | 中等 | 适合社区/游戏服务器 |
| **Slack** | ⭐⭐⭐⭐ | 中等 | 企业办公场景 |
| **WhatsApp** | ⭐⭐⭐ | 中等 | 跨境沟通 |
| **Signal** | ⭐⭐⭐ | 较难 | 注重隐私 |
| **Microsoft Teams** | ⭐⭐⭐ | 中等 | 企业场景 |
| **Matrix** | ⭐⭐ | 较难 | 去中心化协议 |
| **Email** | ⭐⭐ | 简单 | 邮件也能接 |
| **微信** | ⭐ | 难 | 企业版才有 |

**我的建议**：先玩 Telegram，最简单。

---

## 平台一：Telegram（最简单！）

### 为什么先推荐 Telegram？

1. **门槛低**：创建一个 Bot 只需 2 分钟
2. **免费**：Bot 完全免费用
3. **功能全**：支持文字、图片、语音、文件
4. **API 稳定**：很少出问题

### 第一步：创建 Bot

1. 打开 Telegram，搜索 **@BotFather**
2. 点击 "Start"
3. 发送 `/newbot`
4. 给 Bot 取个名字（比如"小明助手"）

```
You: /newbot
BotFather: Alright, a new bot. How are we going to call it? Please choose a name for your bot.
```

5. 给 Bot 取个用户名（必须以 bot 结尾，比如 `xiaoming_helper_bot`）

```
You: xiaoming_helper_bot
BotFather: Good! Now let's give your bot a description...
```

6. 完成！BotFather 会给你一串 Token，**复制保存好**

```
Done! Congratulations on your new bot. You will find it at t.me/xiaoming_helper_bot.
You can now add a description, about section and profile picture for your bot, see /help for a collection of commands.

Use this token to access the HTTP API:
1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ123456789

Keep your token secure and store it safely, it can be used by anyone to control your bot.
```

### 第二步：获取你的用户 ID

1. Telegram 搜索 **@userinfobot**
2. 点击 Start
3. 它会回复你的用户 ID（一串数字），**复制保存**

```
Your user ID: 123456789
```

### 第三步：配置 OpenClaw

编辑 `config.yaml`：

```yaml
channels:
  telegram:
    enabled: true
    bot_token: "1234567890:ABCdefGHIjklMNOpqrSTUvwxYZ123456789"  # BotFather 给的 Token
    allowed_users:
      - 123456789                                        # userinfobot 给的用户 ID
```

### 第四步：重启服务

```bash
docker restart openclaw
```

### 第五步：测试！

1. Telegram 搜索你的 Bot 名字
2. 点击 Start
3. 发一条消息：`你好`

如果 Bot 回复了，恭喜！如果没反应，看日志：

```bash
docker logs openclaw 2>&1 | grep -i telegram
```

### 进阶配置

```yaml
channels:
  telegram:
    enabled: true
    bot_token: "xxx"
    allowed_users:
      - 123456789

    # 消息设置
    message:
      max_length: 4096              # 最大消息长度
      parse_mode: "HTML"          # HTML 或 Markdown
      reply_to_original: false     # 回复原消息还是新消息

    # 群组设置
    group:
      require_mention: true        # 必须 @ 机器人才能回复
      admin_only: false            # 只有管理员能用
```

---

## 平台二：Discord

### 为什么用 Discord？

- **社区氛围好**：适合做客服、社区机器人
- **功能强大**：支持 Embed、Slash Commands、按钮
- **频道管理**：可以做权限控制

### 第一步：创建 Discord Application

1. 访问 https://discord.com/developers/applications
2. 点击 "New Application"
3. 填个名字，点 Create

### 第二步：创建 Bot

1. 左侧菜单点 "Bot"
2. 点 "Add Bot"
3. 点 "Reset Token"，**复制 Token 保存**

**重要**：开启这些权限！
- SERVER MEMBERS INTENT
- MESSAGE CONTENT INTENT

### 第三步：邀请 Bot 到服务器

1. 左侧菜单点 "OAuth2" → "URL Generator"
2. Scopes 勾选 `bot`
3. Bot Permissions 勾选：
   - Send Messages
   - Read Message History
   - Manage Messages（可选）
4. 复制生成的 URL
5. 浏览器打开这个 URL，选择你的服务器

### 第四步：配置 OpenClaw

```yaml
channels:
  discord:
    enabled: true
    bot_token: "MTEyyyy.BBBBB.ccccccc"  # Bot Token
    allowed_guilds:
      - 123456789                       # 服务器 ID（在 Discord 开开发者模式能看到）
    channels:
      - 987654321                       # 频道 ID
```

### 第五步：测试！

在你的频道发消息 `@你的Bot 你好`，看它回不回应。

### 进阶：Slash Commands

Discord 支持斜杠命令：

```yaml
channels:
  discord:
    enabled: true
    bot_token: "xxx"
    commands:
      - name: "ask"
        description: "问 AI 问题"
        options:
          - name: "question"
            type: 3  # STRING
            required: true
            description: "你想问什么"
```

---

## 平台三：Slack

### 为什么用 Slack？

- **企业场景**：团队协作、办公自动化
- **功能丰富**：Modal、按钮、选择器
- **集成好**：和其他企业工具（Notion、Jira）集成方便

### 第一步：创建 Slack App

1. 访问 https://api.slack.com/apps
2. 点 "Create New App"
3. 选择 "From scratch"
4. 填名字，选工作区

### 第二步：配置权限

1. 左侧菜单 "OAuth & Permissions"
2. Scopes 里添加：
   - `chat:write`
   - `commands`
   - `app_mentions:read`
   - `im:history`
3. 点 "Install to Workspace"
4. 复制 Bot User OAuth Token（以 `xoxb-` 开头）

### 第三步：开启 Socket Mode

1. 左侧菜单 "Socket Mode"
2. 点 Enable Socket Mode
3. 创建 App-Level Token，添加 `connections:write` scope

### 第四步：配置 OpenClaw

```yaml
channels:
  slack:
    enabled: true
    app_token: "xapp-xxx"    # App-Level Token
    bot_token: "xoxb-xxx"    # Bot User OAuth Token
    signing_secret: "xxx"     # Basic Information 里能找到
```

### 第五步：测试！

在 Slack 发消息 `@你的Bot 你好`

---

## 平台四：WhatsApp

### 为什么用 WhatsApp？

- **用户多**：全球用户超 20 亿
- **跨境方便**：适合国际化产品
- **商业版稳定**：WhatsApp Business API

### 第一步：获取 WhatsApp Business API

这个比 Telegram 复杂，需要：
1. 注册 WhatsApp Business API
2. 创建 Business Account
3. 获取 Phone Number ID 和 Access Token

### 第二步：配置 OpenClaw

```yaml
channels:
  whatsapp:
    enabled: true
    phone_number_id: "xxx"      # Phone Number ID
    access_token: "xxx"        # Access Token
    business_account_id: "xxx" # Business Account ID
    webhook_verify_token: "xxx" # 你自己设置的验证 Token
```

### 第三步：配置 Webhook

在 WhatsApp Business 后台配置 Webhook URL：
```
https://你的域名/webhooks/whatsapp
```

### 进阶配置

```yaml
channels:
  whatsapp:
    enabled: true
    phone_number_id: "xxx"
    access_token: "xxx"

    # 安全设置
    security:
      verify_token: "你的验证Token"
      allow_from:                    # 白名单
        - "+86138xxxxxxx"

    # 消息格式
    message:
      template_enabled: true         # 使用模板消息
      max_length: 4096
```

---

## 平台五：Email

### 为什么用 Email？

- **通用**：人人都有邮箱
- **异步**：不需要实时回复
- **正式**：适合重要通知

### 配置 SMTP/IMAP

```yaml
channels:
  email:
    enabled: true

    smtp:
      host: "smtp.gmail.com"
      port: 587
      username: "${SMTP_USERNAME}"
      password: "${SMTP_PASSWORD}"
      starttls: true

    imap:
      host: "imap.gmail.com"
      port: 993
      username: "${IMAP_USERNAME}"
      password: "${IMAP_PASSWORD}"

    # 收到邮件的处理规则
    rules:
      - name: "ai-commands"
        subject_contains: "[AI]"
        action: "process"
```

### 使用示例

```
发送邮件到：your-email@gmail.com
主题：[AI] 帮我查下北京天气

---

Bot 回复：
🌤️ 北京今天天气晴，22°C，适合外出。
```

---

## 同时接多个平台

OpenClaw 可以同时接多个平台！

```yaml
channels:
  # Telegram
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT_TOKEN}"
    allowed_users:
      - 123456789

  # Discord
  discord:
    enabled: true
    bot_token: "${DISCORD_BOT_TOKEN}"

  # Slack
  slack:
    enabled: false        # 暂时不启用
    bot_token: "${SLACK_BOT_TOKEN}"

  # Email
  email:
    enabled: true
    smtp:
      host: "smtp.gmail.com"
      port: 587
      username: "${SMTP_USERNAME}"
      password: "${SMTP_PASSWORD}"
```

**注意**：
- 每个平台需要独立的 Token/配置
- 同时启用的平台越多，资源消耗越大
- 建议先接一个平台，调试好了再加第二个

---

## 消息路由配置

### 多工作区消息隔离

```yaml
workspaces:
  personal:
    channels:
      telegram:
        allowed_users:
          - 123456789

  team:
    channels:
      discord:
        allowed_guilds:
          - 999888777

      slack:
        team_id: "T0123456789"
```

### 跨平台消息同步

想把 Telegram 的消息同步到 Discord？

```yaml
channels:
  telegram:
    enabled: true
    bot_token: "xxx"

  discord:
    enabled: true
    bot_token: "xxx"

# 通过插件实现跨平台同步
plugins:
  - name: "cross-platform-sync"
    config:
      rules:
        - source: "telegram"
          target: "discord"
          channel_map:
            "personal_chat": "announcements"
```

---

## 常见问题

### Q: Bot 不回复消息？

排查顺序：

1. **检查 enabled 配置**
```yaml
channels:
  telegram:
    enabled: true  # 必须是 true
```

2. **检查 Token**
```bash
# 检查日志有没有 Token 错误
docker logs openclaw 2>&1 | grep -i "token"
```

3. **检查用户白名单**
```yaml
channels:
  telegram:
    allowed_users:
      - 123456789   # 你在 userinfobot 看到的 ID
```

### Q: Discord Bot 显示"无法响应"？

1. 检查 Bot 权限是否足够
2. 检查 MESSAGE CONTENT INTENT 是否开启
3. 检查 Bot 是否在频道里

### Q: Slack Bot 不在线？

1. 检查 Socket Mode 是否开启
2. 检查 Event Subscriptions 是否开启
3. 检查 Bot Token 是否正确

### Q: 想限制 Bot 只在某些群/频道响应？

```yaml
channels:
  telegram:
    allowed_users:
      - 123456789
      - 987654321

  discord:
    allowed_guilds:
      - 123456789
    allowed_channels:
      - 987654321
```

### Q: 如何处理私聊和群聊不同行为？

```yaml
channels:
  telegram:
    private:
      enabled: true
      auto_reply: true

    group:
      enabled: true
      require_mention: true    # 必须 @ 机器人才回复
```

---

## 安全建议

### 1. 设置白名单

```yaml
channels:
  telegram:
    allowed_users:
      - 你的ID          # 只有你能用

  discord:
    allowed_guilds:
      - 你的服务器ID    # 只有你的服务器能用
```

### 2. 敏感操作二次确认

```yaml
security:
  dangerous_tools:
    require_confirmation:
      - "bash"
      - "delete_file"
      - "send_external_message"
```

### 3. 消息审计

```yaml
security:
  audit:
    enabled: true
    log_channel: "audit-channel-id"
```

---

## 下一步

多平台都接好了？继续学习：

| 教程 | 内容 |
|------|------|
| [[OpenClaw记忆系统]] | 让 AI 跨平台记住你是谁 |
| [[ClawHub技能市场]] | 给各平台装插件 |
| [[OpenClaw高级用法]] | 多 Agent、工作流 |
| [[OpenClaw插件开发]] | 自己写平台适配器 |

---

**文档状态**：多平台集成指南  
**更新时间**：2026年4月
