---
title: OpenClaw安装部署
date: 2026-04-18
tags:
  - 安装部署
  - Docker
  - 源码
  - Python-3.11
  - GPU配置
  - 多平台
  - Linux
  - macOS
  - Windows
  - 硬件要求
categories:
  - 小龙虾生态
  - OpenClaw部署
alias: OpenClaw Installation and Deployment
---

# OpenClaw 安装部署

> [!note] 文档信息
> 本文详细介绍 OpenClaw 的各种部署方式，包括 Docker 部署、源码安装、多平台支持、GPU 配置等完整指南。

---

## 关键词速览

| 关键词 | 说明 |
|--------|------|
| Docker | 容器化部署方案 |
| 源码安装 | 从源码编译安装 |
| Python 3.11+ | Python 版本要求 |
| GPU配置 | CUDA 加速支持 |
| Linux | Linux 系统部署 |
| macOS | macOS 系统部署 |
| Windows | Windows 系统部署 |
| 环境变量 | 配置管理 |
| systemd | 服务管理 |
| 硬件要求 | 系统资源需求 |

---

## 一、系统要求

### 1.1 硬件要求

| 组件 | 最低配置 | 推荐配置 |
|------|----------|----------|
| **CPU** | 2 核心 | 4+ 核心 |
| **内存** | 4 GB | 8+ GB |
| **磁盘** | 10 GB | 50+ GB SSD |
| **网络** | 稳定互联网 | 高速连接 |

### 1.2 软件要求

| 软件 | 版本要求 | 说明 |
|------|----------|------|
| **Python** | ≥ 3.11 | 必须 |
| **Docker** | ≥ 20.10 | 推荐 |
| **Git** | 最新版 | 源码安装需要 |
| **pip** | 最新版 | 包管理 |

### 1.3 LLM API 要求

> [!tip] 注意
> OpenClaw 本身不运行 AI 模型，需要配置 LLM API（如 OpenAI、Anthropic 等）才能正常工作。

---

## 二、Docker 部署（推荐）

### 2.1 快速部署

```bash
# 1. 拉取官方镜像
docker pull openclaw/openclaw:latest

# 2. 创建配置目录
mkdir -p ~/openclaw/{config,data,plugins,memory}

# 3. 创建配置文件
cat > ~/openclaw/config/config.yaml << 'EOF'
openclaw:
  version: "2.x"
  
llm:
  provider: "anthropic"
  model: "claude-sonnet-4-20250514"
  api_key: "${ANTHROPIC_API_KEY}"

channels:
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT_TOKEN}"
EOF

# 4. 启动容器
docker run -d \
  --name openclaw \
  --restart unless-stopped \
  -p 18789:18789 \
  -p 8080:8080 \
  -v ~/openclaw/config:/app/config \
  -v ~/openclaw/data:/app/data \
  -v ~/openclaw/plugins:/app/plugins \
  -v ~/openclaw/memory:/app/memory \
  -e ANTHROPIC_API_KEY="your-api-key" \
  -e TELEGRAM_BOT_TOKEN="your-bot-token" \
  openclaw/openclaw:latest
```

### 2.2 Docker Compose 部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    container_name: openclaw
    restart: unless-stopped
    ports:
      - "18789:18789"  # WebSocket 控制端口
      - "8080:8080"    # HTTP API 端口
    volumes:
      - ./config:/app/config
      - ./data:/app/data
      - ./plugins:/app/plugins
      - ./memory:/app/memory
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
      - LOG_LEVEL=info
    networks:
      - openclaw-network

  # 可选：Redis 缓存
  redis:
    image: redis:7-alpine
    container_name: openclaw-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - openclaw-network

volumes:
  redis-data:

networks:
  openclaw-network:
    driver: bridge
```

```bash
# 启动服务
docker-compose up -d

# 查看日志
docker-compose logs -f openclaw

# 停止服务
docker-compose down
```

### 2.3 GPU 支持

```bash
# NVIDIA GPU 支持
docker run -d \
  --name openclaw-gpu \
  --gpus all \
  -p 18789:18789 \
  -v ~/openclaw:/app \
  openclaw/openclaw:latest
```

### 2.4 远程 Docker 部署

```bash
# 连接到远程 Docker daemon
export DOCKER_HOST=ssh://user@remote-server

# 或者使用 Docker Context
docker context create remote --docker "host=ssh://user@remote-server"
docker context use remote

# 部署
docker run -d \
  --name openclaw \
  -p 18789:18789 \
  -v ~/openclaw/config:/app/config \
  openclaw/openclaw:latest
```

---

## 三、源码安装

### 3.1 前提条件

```bash
# macOS
brew install python@3.11 git

# Ubuntu/Debian
sudo apt update
sudo apt install python3.11 python3.11-venv git

# Fedora/RHEL
sudo dnf install python3.11 git
```

### 3.2 创建虚拟环境

```bash
# 创建项目目录
mkdir -p ~/openclaw && cd ~/openclaw

# 创建虚拟环境
python3.11 -m venv venv

# 激活虚拟环境
source venv/bin/activate  # Linux/macOS
# 或
.\venv\Scripts\activate   # Windows

# 升级 pip
pip install --upgrade pip
```

### 3.3 克隆与安装

```bash
# 克隆仓库
git clone https://github.com/openclaw-project/openclaw.git
cd openclaw

# 安装依赖
pip install -e .

# 安装可选依赖
pip install -e ".[dev]"        # 开发依赖
pip install -e ".[telegram]"   # Telegram 支持
pip install -e ".[discord]"    # Discord 支持
pip install -e ".[whatsapp]"   # WhatsApp 支持
pip install -e ".[all]"        # 所有可选依赖
```

### 3.4 配置与运行

```bash
# 1. 创建配置目录
mkdir -p ~/.openclaw

# 2. 复制配置模板
cp config.example.yaml ~/.openclaw/config.yaml

# 3. 编辑配置
nano ~/.openclaw/config.yaml

# 4. 运行 OpenClaw
openclaw run

# 或者开发模式（热重载）
openclaw run --dev
```

### 3.5 Windows 特殊说明

```powershell
# PowerShell 安装步骤

# 1. 安装 Python 3.11+
winget install Python.Python.3.11

# 2. 克隆仓库
git clone https://github.com/openclaw-project/openclaw.git
cd openclaw

# 3. 创建虚拟环境
python -m venv venv
.\venv\Scripts\Activate.ps1

# 4. 安装
pip install -e .

# 5. 运行
openclaw run
```

---

## 四、多平台部署

### 4.1 Linux 系统服务

```bash
# 创建 systemd 服务文件
sudo nano /etc/systemd/system/openclaw.service

# 内容：
[Unit]
Description=OpenClaw AI Assistant
After=network.target

[Service]
Type=simple
User=openclaw
Group=openclaw
WorkingDirectory=/home/openclaw
Environment="PATH=/home/openclaw/venv/bin"
EnvironmentFile=/home/openclaw/.env
ExecStart=/home/openclaw/venv/bin/openclaw run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
# 创建用户
sudo useradd -m -s /bin/bash openclaw
sudo mkdir -p /home/openclaw
sudo chown openclaw:openclaw /home/openclaw

# 设置权限
sudo systemctl daemon-reload
sudo systemctl enable openclaw
sudo systemctl start openclaw

# 查看状态
sudo systemctl status openclaw

# 查看日志
sudo journalctl -u openclaw -f
```

### 4.2 macOS LaunchD

```xml
<!-- ~/Library/LaunchAgents/com.openclaw.plist -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "...">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.openclaw</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/username/openclaw/venv/bin/openclaw</string>
        <string>run</string>
    </array>
    <key>WorkingDirectory</key>
    <string>/Users/username/openclaw</string>
    <key>EnvironmentVariables</key>
    <dict>
        <key>PATH</key>
        <string>/Users/username/openclaw/venv/bin</string>
    </dict>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/username/openclaw/openclaw.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/username/openclaw/openclaw.error.log</string>
</dict>
</plist>
```

```bash
# 加载服务
launchctl load ~/Library/LaunchAgents/com.openclaw.plist

# 卸载服务
launchctl unload ~/Library/LaunchAgents/com.openclaw.plist
```

### 4.3 Raspberry Pi 部署

```bash
# 1. 安装依赖
sudo apt update
sudo apt install -y python3.11-venv git libffi-dev libssl-dev

# 2. 创建虚拟环境
python3.11 -m venv openclaw
source openclaw/bin/activate

# 3. 安装（使用轻量依赖）
pip install openclaw[minimal]

# 4. 创建服务
sudo nano /etc/systemd/system/openclaw.service

# 5. 启动
sudo systemctl enable openclaw
sudo systemctl start openclaw
```

### 4.4 NAS 部署（群晖/威联通）

```bash
# 群晖 DSM 7.x

# 1. 安装 Python 3.11 套件（从套件中心）
# 2. 通过 SSH 连接到 NAS

# 3. 创建虚拟环境
python3.11 -m venv /volume1/openclaw/venv

# 4. 安装
/volume1/openclaw/venv/bin/pip install openclaw[all]

# 5. 配置（使用 File Station 编辑）
# 在 /volume1/openclaw/ 创建 config.yaml

# 6. 创建任务计划（控制面板 > 任务计划）
# 启动命令: /volume1/openclaw/venv/bin/openclaw run
```

---

## 五、高级配置

### 5.1 反向代理配置

#### Nginx

```nginx
# /etc/nginx/sites-available/openclaw
upstream openclaw_backend {
    server 127.0.0.1:18789;
}

server {
    listen 80;
    server_name openclaw.example.com;
    
    # SSL 配置
    listen 443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/openclaw.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/openclaw.example.com/privkey.pem;
    
    # WebSocket 支持
    location /ws {
        proxy_pass http://openclaw_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # HTTP API
    location / {
        proxy_pass http://openclaw_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

#### Caddy

```caddy
# Caddyfile
openclaw.example.com {
    reverse_proxy /ws/* 127.0.0.1:18789
    reverse_proxy /* 127.0.0.1:18789
    
    encode gzip
    
    tls {
        protocols tls1.2 tls1.3
    }
}
```

### 5.2 HTTPS 配置

```bash
# 使用 Let's Encrypt

# 安装 certbot
sudo apt install certbot python3-certbot-nginx

# 获取证书
sudo certbot --nginx -d openclaw.example.com

# 自动续期测试
sudo certbot renew --dry-run
```

### 5.3 负载均衡

```yaml
# docker-compose.yml - 多实例
version: '3.8'

services:
  openclaw-1:
    image: openclaw/openclaw:latest
    environment:
      - INSTANCE_ID=1
    networks:
      - openclaw-net

  openclaw-2:
    image: openclaw/openclaw:latest
    environment:
      - INSTANCE_ID=2
    networks:
      - openclaw-net

  nginx:
    image: nginx:alpine
    ports:
      - "18789:18789"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - openclaw-1
      - openclaw-2
    networks:
      - openclaw-net

networks:
  openclaw-net:
```

### 5.4 监控配置

```yaml
# docker-compose.yml - 添加监控
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    ports:
      - "18789:18789"
      - "9090:9090"  # Prometheus metrics
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9091:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - prometheus

volumes:
  grafana-data:
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'openclaw'
    static_configs:
      - targets: ['openclaw:9090']
```

---

## 六、故障排除

### 6.1 常见问题

> [!faq] Q1: 启动失败，提示 "ModuleNotFoundError"
> 
> **解决**：确保已正确安装依赖
> ```bash
> pip install -e .
> ```

> [!faq] Q2: Docker 容器不断重启
> 
> **解决**：检查日志和配置
> ```bash
> docker logs openclaw
> docker exec openclaw openclaw doctor
> ```

> [!faq] Q3: WebSocket 连接失败
> 
> **解决**：检查端口和防火墙
> ```bash
> # 检查端口
> netstat -tlnp | grep 18789
> 
> # 测试连接
> curl http://localhost:18789/health
> ```

> [!faq] Q4: 内存占用过高
> 
> **解决**：限制容器资源
> ```yaml
> # docker-compose.yml
> services:
>   openclaw:
>     deploy:
>       resources:
>         limits:
>           memory: 4G
> ```

### 6.2 诊断命令

```bash
# OpenClaw 诊断
openclaw doctor

# 检查配置
openclaw config validate

# 检查连接
openclaw check --llm
openclaw check --channels

# 查看日志
openclaw logs --level DEBUG
```

### 6.3 日志管理

```bash
# Docker 日志
docker logs -f openclaw --tail 100

# 日志轮转
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

---

## 七、安全加固

### 7.1 最佳实践

| 安全措施 | 说明 |
|----------|------|
| **环境变量存储密钥** | 避免在配置文件中明文存储 |
| **最小权限原则** | 容器使用非 root 用户 |
| **网络隔离** | 使用 Docker 网络 |
| **定期更新** | 及时更新镜像和依赖 |
| **备份配置** | 定期备份配置文件 |

### 7.2 安全配置示例

```yaml
# docker-compose.yml - 安全加固
version: '3.8'

services:
  openclaw:
    image: openclaw/openclaw:latest
    user: "1000:1000"  # 非 root 用户
    read_only: true    # 只读文件系统
    security_opt:
      - no-new-privileges:true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=100m
    networks:
      - internal
  
  nginx:
    image: nginx:alpine
    networks:
      - web
      - internal

networks:
  web:
    internal: false
  internal:
    internal: true
```

---

## 八、相关文档

- [[OpenClaw完整指南]] - OpenClaw 完整指南
- [[OpenClaw配置详解]] - 详细配置说明
- [[OpenClaw多平台集成]] - 多平台集成
- [[OpenClaw架构解析]] - 技术架构
- [[OpenClaw高级用法]] - 高级技巧

---

*文档更新于 2026年4月18日*
