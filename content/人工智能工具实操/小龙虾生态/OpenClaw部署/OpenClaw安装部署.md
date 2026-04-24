---
title: OpenClaw安装部署完全指南
date: 2026-04-24
tags:
  - OpenClaw
  - 安装教程
  - Docker
  - 部署
  - npm
  - 小白教程
categories:
  - 人工智能工具实操
  - 小龙虾生态
  - 安装部署
description: 零基础安装OpenClaw，从环境准备到运行起来，每一步都有截图级别的详细说明。
---

# OpenClaw 安装部署完全指南

> 这篇教程专门写给完全没有技术背景的同学。手把手，每一步都讲清楚，遇到问题照着排查就行。

---

## 先搞清楚：安装方式有哪些？

OpenClaw 支持三种安装方式，选一个适合你的：

| 方式 | 适合人群 | 难度 | 备注 |
|------|----------|------|------|
| **Docker（推荐）** | 99% 的用户 | ⭐ | 装一次，以后升级方便 |
| **npm** | 不想装 Docker | ⭐⭐ | 需要懂一点命令行 |
| **源码** | 开发者/想改代码 | ⭐⭐⭐ | 需要 Node.js 基础 |

**强烈建议用 Docker**，除非你知道自己在干什么。

---

## 方式一：Docker 安装（手把手版）

### 第一步：安装 Docker

Docker 是什么？你可以把它理解为一个"轻量级虚拟机"，软件装在里面不会影响你的电脑。

**macOS 用户：**

1. 打开 App Store，搜索 "Docker Desktop"
2. 点击安装（安装包大约 600MB）
3. 安装完成后打开 Docker Desktop
4. 等待看到 Docker 图标栏显示 "Docker Desktop is running"

**Windows 用户：**

1. 去 https://www.docker.com/products/docker-desktop/ 下载
2. 双击安装包，一路 Next
3. **注意**：需要开启 WSL2 或 Hyper-V（安装程序会提示你）

**Linux 用户（Ubuntu 为例）：**

```bash
# 一键安装脚本
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# 把当前用户加入 docker 组（免 sudo）
sudo usermod -aG docker $USER
# 重新登录后生效
```

### 第二步：验证 Docker 装好了

打开终端（macOS 按 Cmd+空格，搜 "Terminal"），输入：

```bash
docker --version
```

如果看到类似 `Docker version 24.x.x` 这样的输出，恭喜你，装好了！

### 第三步：创建配置文件夹

```bash
# 创建目录
mkdir -p ~/openclaw/config
mkdir -p ~/openclaw/data
mkdir -p ~/openclaw/plugins

# 进入目录
cd ~/openclaw
```

> `~` 代表你的用户主目录，比如 `/Users/你的名字/`

### 第四步：创建配置文件

**新建一个 `config.yaml` 文件**：

```yaml
# llm = 大语言模型，就是 GPT、Claude 这些
llm:
  provider: "anthropic"                    # 用哪家？anthropic = Claude
  model: "claude-sonnet-4-20250514"         # 具体用哪个模型
  api_key: "sk-ant-api03-xxxxxxxxxxxxx"    # ⚠️ 改成你自己的 API Key

# channels = 消息渠道，就是你要接哪些聊天软件
channels:
  telegram:
    enabled: false                          # 暂时不接，先跑起来再说
  
# system = 系统设置
system:
  prompt: "你是一个友好的中文AI助手，请用中文回答。"
```

> **关于 API Key**：去 [Anthropic 官网](https://console.anthropic.com/) 注册账号，然后在 API Keys 页面创建一个。

### 第五步：拉取镜像并启动

```bash
# 拉取 OpenClaw 镜像（第一次会下载大约 1GB）
docker pull openclaw/openclaw:latest

# 启动！
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/openclaw/config:/app/config \
  -v ~/openclaw/data:/app/data \
  -v ~/openclaw/plugins:/app/plugins \
  --restart unless-stopped \
  openclaw/openclaw:latest
```

### 第六步：验证成功了没？

```bash
# 查看容器状态
docker ps

# 应该看到类似这样的输出：
# CONTAINER ID   IMAGE   COMMAND   STATUS   PORTS
# abc123def456   openclaw  ...      Up      0.0.0.0:18789->18789/tcp

# 查看启动日志
docker logs openclaw
```

如果最后几行显示类似这样的内容：

```
✅ Gateway listening on port 18789
✅ OpenClaw started successfully
```

**恭喜！安装成功！**

### 第七步：打开控制面板看看

打开浏览器，访问：

```
http://127.0.0.1:18789
```

你应该能看到 OpenClaw 的控制面板了。

---

## 方式二：npm 安装（不需要 Docker）

如果你实在不想装 Docker，可以试试 npm 方式。

### 第一步：安装 Node.js

Node.js 是 JavaScript 的运行环境，npm 是它的包管理器。

**macOS：**

```bash
# 用 Homebrew 安装（推荐）
brew install node@22

# 验证
node --version   # 应该显示 v22.x.x
npm --version    # 应该显示 10.x.x
```

**Windows：**

去 https://nodejs.org/ 下载 LTS 版本（推荐 22.x），安装包会自动处理一切。

**Linux：**

```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# 验证
node --version
npm --version
```

### 第二步：安装 OpenClaw

```bash
# 全局安装（-g 表示装到系统目录）
npm install -g openclaw@latest

# 验证
openclaw --version
```

### 第三步：引导配置

```bash
# 启动引导程序
openclaw onboard
```

这个命令会问你一些问题：

```
? 选择安装方式 (Use arrow keys)
❯ Docker
  npm
  源码
```

用键盘上下箭头选一项，回车确认。

接下来的问题：
- API Provider：选 `anthropic`
- API Key：粘贴你的密钥
- 默认模型：直接回车用默认值

### 第四步：启动！

```bash
# 启动服务
openclaw gateway

# 看到类似输出就成功了：
# Gateway listening on http://0.0.0.0:18789
```

### 后台运行（可选）

如果你不想一直开着终端窗口：

```bash
# macOS/Linux：用 nohup
nohup openclaw gateway > openclaw.log 2>&1 &

# 或者用 pm2（需要先安装）
npm install -g pm2
pm2 start openclaw --name openclaw
pm2 save

# 查看状态
pm2 status
pm2 logs openclaw
```

---

## 安装后必做：接 Telegram

> 如果你不知道什么是 Telegram，可以跳过这节，以后想接再说。

### 第一步：创建 Telegram Bot

1. 在 Telegram 里搜索 **@BotFather**
2. 点击 Start
3. 发送 `/newbot`
4. 给 Bot 取个名字（比如"我的 AI 助手"）
5. 给 Bot 取个用户名（必须以 bot 结尾，比如 `my_ai_helper_bot`）
6. BotFather 会给你一串 Token，**复制下来保存好**

### 第二步：获取你的用户 ID

1. 在 Telegram 里搜索 **@userinfobot**
2. 点击 Start
3. 它会回复你的用户 ID（是一串数字），**复制下来**

### 第三步：修改配置

编辑 `~/openclaw/config/config.yaml`：

```yaml
llm:
  provider: "anthropic"
  model: "claude-sonnet-4-20250514"
  api_key: "你的API密钥"

channels:
  telegram:
    enabled: true                          # 改成 true
    bot_token: "123456789:ABCdefGHIjkl"   # BotFather 给的 Token
    allowed_users:
      - 123456789                          # userinfobot 给的用户 ID
```

### 第四步：重启服务

```bash
# Docker 方式
docker restart openclaw

# npm 方式
# Ctrl+C 停止，然后重新运行 openclaw gateway
```

### 第五步：测试！

去 Telegram 找你的 Bot，点 Start，然后随便发条消息。

> 如果 Bot 不理你，先看日志：`docker logs openclaw`

---

## 常见问题排查

### 问题一：docker run 报 "docker: command not found"

**原因**：Docker 没装或没打开。

**解决**：
- macOS/Windows：去应用商店下载 Docker Desktop，然后打开它
- Linux：运行 `sudo apt install docker.io`

---

### 问题二：docker run 报 "permission denied"

**原因**：没有权限运行 Docker。

**解决**：
```bash
# 把当前用户加入 docker 组
sudo usermod -aG docker $USER

# 然后**退出终端，重新打开**
```

---

### 问题三：访问 127.0.0.1:18789 显示 "连接被拒绝"

**排查步骤**：

1. 检查容器是否在运行：
```bash
docker ps
```

2. 如果没有运行，看错误日志：
```bash
docker logs openclaw
```

3. 常见错误：
   - "API key is invalid" → 检查 API Key 是否正确
   - "Connection refused" → API 服务商那边可能挂了
   - "Port already in use" → 18789 端口被占用了

---

### 问题四：API Key 不工作

**可能原因**：

1. Key 写错了（最常见）
2. Key 过期了
3. 账号没钱了（Claude 按量付费）
4. Key 类型不对（比如用了错误的 API）

**诊断命令**：

```bash
# 测试 Claude API
curl -X POST https://api.anthropic.com/v1/messages \
  -H "x-api-key: 你的KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model":"claude-sonnet-4-20250514","max_tokens":10,"messages":[{"role":"user","content":"hi"}]}'
```

如果返回 JSON 而不是报错，说明 Key 是好用的。

---

### 问题五：Bot 不回复消息

**排查顺序**：

1. 检查 Telegram Bot Token 是否正确
2. 检查用户 ID 是否在 `allowed_users` 里
3. 检查 API Key 是否有余额
4. 看日志：

```bash
docker logs openclaw 2>&1 | grep -i "error"
```

---

## 进阶：Docker Compose 方式（更专业）

如果你想更方便地管理（开机自启、多个服务一起跑），可以用 Docker Compose。

### 创建 docker-compose.yml

在 `~/openclaw/` 目录下新建文件：

```yaml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "18789:18789"
    volumes:
      - ./config:/app/config
      - ./data:/app/data
      - ./plugins:/app/plugins
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:18789/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### 创建 .env 文件

在同一目录下创建 `.env` 文件（注意前面有个点）：

```
ANTHROPIC_API_KEY=sk-ant-api03-xxxxxxxxxxxxx
```

> **安全提醒**：`.env` 文件包含密钥，记得加到 `.gitignore` 里。

### 启动

```bash
# 启动所有服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs -f openclaw

# 停止
docker-compose down
```

---

## 进阶：服务器部署（7×24 小时运行）

如果你想把它跑在服务器上，让 Bot 24 小时在线：

### 推荐服务器

| 提供商 | 最低配置 | 价格 | 特点 |
|--------|----------|------|------|
| **Hetzner** | 2核4G | €3.5/月 | 欧洲性价比最高 |
| **Vultr** | 1核1G | $6/月 | 全球节点 |
| **DigitalOcean** | 1核1G | $6/月 | 稳定好 |

### SSH 连接服务器

```bash
ssh root@你的服务器IP
```

### 在服务器上安装

```bash
# 安装 Docker
curl -fsSL https://get.docker.com | sh

# 创建目录
mkdir -p ~/openclaw/config

# 上传配置文件（在你电脑上运行）
scp ~/openclaw/config/config.yaml root@服务器IP:~/openclaw/config/

# 启动
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/openclaw/config:/app/config \
  --restart unless-stopped \
  openclaw/openclaw:latest
```

### 用 PM2 保持后台运行

```bash
# 安装 Node.js 和 PM2
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs
npm install -g pm2

# 用 PM2 启动
pm2 start --name openclaw "docker run --rm -p 18789:18789 -v ~/openclaw/config:/app/config openclaw/openclaw:latest"

# 设置开机自启
pm2 startup
pm2 save
```

---

## 卸载

如果哪天不想用了：

```bash
# Docker 方式
docker stop openclaw
docker rm openclaw
docker rmi openclaw/openclaw:latest

# npm 方式
npm uninstall -g openclaw

# 删除配置文件（可选）
rm -rf ~/openclaw
```

---

## 下一步

安装成功啦！接下来可以学习：

- [[OpenClaw完整指南]] - 了解所有功能
- [[OpenClaw多平台集成]] - 接 Discord/微信等
- [[OpenClaw记忆系统]] - 让 AI 记住更多
- [[ClawHub技能市场]] - 装各种插件

---

**有问题？**

1. 先看日志：`docker logs openclaw`
2. 搜一下 GitHub Issues
3. 去 Discord 社区问

祝安装顺利！
