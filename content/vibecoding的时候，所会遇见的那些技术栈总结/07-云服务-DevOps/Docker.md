# Docker：容器化技术的工业标准

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Docker 27 新特性、BuildKit、多阶段构建、Docker Compose 最佳实践，以及 AI 应用容器化实战指南。

---

## 目录

1. [[#Docker 核心概念与架构]]
2. [[#Docker 27 新特性]]
3. [[#Dockerfile 最佳实践]]
4. [[#多阶段构建（Multi-stage）]]
5. [[#Docker Compose 完整指南]]
6. [[#Volume 与 Bind Mount]]
7. [[#网络模式详解]]
8. [[#Docker vs Podman vs containerd 对比]]
9. [[#Docker Desktop vs Orbstack]]
10. [[#AI 应用容器化实战]]

---

## Docker 核心概念与架构

### 容器化技术概述

容器化是一种轻量级操作系统级虚拟化技术，允许多个隔离的用户空间实例在同一台主机上运行。Docker 由 Solomon Hykes 于 2013 年创建，通过将应用程序及其依赖打包为标准化单元，实现了「Build Once, Run Anywhere」的核心理念。

与虚拟机相比，容器共享宿主机的内核，启动时间以秒计（而非分钟），资源开销极低：

| 维度 | 虚拟机 | 容器 |
|------|--------|------|
| **启动时间** | 1-5 分钟 | 毫秒-秒级 |
| **资源占用** | 完整操作系统（GB 级） | 应用+依赖（MB 级） |
| **隔离性** | 完全隔离 | 进程级隔离 |
| **性能损耗** | 15-30% | 1-5% |
| **密度** | 低（每主机 10+） | 高（每主机 100+） |

### Docker 核心组件

```
┌─────────────────────────────────────────────────────────────┐
│                        Docker 主机                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────────┐   │
│  │   容器 1     │    │   容器 2     │    │   容器 3     │   │
│  │  ┌───────┐  │    │  ┌───────┐  │    │  ┌───────┐   │   │
│  │  │ APP A │  │    │  │ APP B │  │    │  │ APP C │   │   │
│  │  └───────┘  │    │  └───────┘  │    │  └───────┘   │   │
│  └─────────────┘    └─────────────┘    └──────────────┘   │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                    Docker Daemon                      │  │
│  │   (containerd + runc + 镜像管理 + 网络 + 存储)        │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                     OS Kernel                        │  │
│  │           (Namespaces + Cgroups + Seccomp)           │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 核心概念澄清

| 概念 | 说明 | 类比 |
|------|------|------|
| **Image（镜像）** | 只读模板，包含应用及依赖 | 面向对象中的类 |
| **Container（容器）** | 镜像的运行实例 | 面向对象中的对象 |
| **Layer（层）** | 镜像的增量修改 | Git commit |
| **Registry（仓库）** | 存储和分发镜像 | GitHub |
| **Dockerfile** | 构建镜像的脚本 | Makefile |

---

## Docker 27 新特性

### Docker 27 主要更新（2026年）

Docker 27 是截至 2026年的最新稳定版本，引入了多项生产就绪特性：

| 特性 | 说明 | 受益场景 |
|------|------|---------|
| **Docker Compose v3** | 服务网格、健康检查、扩展增强 | 微服务编排 |
| **BuildKit v0.16** | 并行构建、改进缓存、更好的错误信息 | CI/CD |
| **Docker Scout** | 镜像安全分析 | 生产环境 |
| **Container File System** | 改进的层合并算法 | 构建性能 |
| **WSL2 集成优化** | 更快的 macOS 虚拟机性能 | Windows 开发 |
| **镜像签名验证** | Cosign 集成 | 安全合规 |

### BuildKit 高级特性

BuildKit 是 Docker 的下一代构建引擎，提供更高效的镜像构建能力：

```bash
# 启用 BuildKit（Docker 27 默认启用）
export DOCKER_BUILDKIT=1

# 并行构建（自动）
# BuildKit 自动分析依赖关系并行构建

# 更好的缓存
# ─── building stage 1/5 ──────────────────────
# CACHED 表示使用缓存

# 改进的错误信息
# 显示失败步骤的完整上下文
```

### Docker Scout 安全分析

```bash
# 安装 Docker Scout CLI
docker scout install

# 分析镜像漏洞
docker scout cves myapp:latest

# 生成 SBOM（软件物料清单）
docker scout sbom myapp:latest

# 主动建议
docker scout recommend myapp:latest
```

### Compose v3 新特性

```yaml
# docker-compose.yml
services:
  web:
    image: nginx:alpine
    # 新增：健康检查
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    
    # 新增：依赖等待
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    # 新增：扩展配置
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  db:
    image: postgres:16-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

## Dockerfile 最佳实践

### Dockerfile 最佳实践清单

#### 基础镜像选择

```dockerfile
# ✓ 推荐：使用特定版本镜像
FROM node:20-alpine

# ✗ 避免：latest 标签可能导致构建不稳定
FROM node:latest

# ✓ 推荐：Alpine 镜像体积更小（~5MB vs ~900MB）
FROM python:3.12-slim

# ✗ 避免：debian 基础镜像体积过大
FROM python:3.12
```

#### 层缓存优化

```dockerfile
# ─────────────────────────────────────────────────────────────
# 优化策略：按变更频率排序
# ─────────────────────────────────────────────────────────────

# 1. 静态内容（最少变更）→ 最先 COPY
COPY package.json package-lock.json* ./

# 2. 依赖安装（变更较少）
RUN npm ci --only=production

# 3. 应用代码（频繁变更）→ 最后 COPY
COPY src/ ./src/

# 4. 配置和静态资源（中等变更）
COPY public/ ./public/
```

#### 多行指令提高可读性

```dockerfile
# ✓ 推荐：使用多行格式便于注释和维护
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        curl \
        wget \
        ca-certificates \
        gnupg \
        lsb-release && \
    rm -rf /var/lib/apt/lists/*

# ✗ 避免：单行过长难以阅读
RUN apt-get update && apt-get install -y git curl wget ca-certificates gnupg lsb-release && rm -rf /var/lib/apt/lists/*
```

#### 最小化层数

```dockerfile
# ✓ 推荐：合并 RUN 指令
RUN apt-get update && \
    apt-get install -y nginx && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ✗ 避免：多个 RUN 指令产生多个层
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get clean
```

#### 非 root 用户运行

```dockerfile
# 创建应用用户
RUN groupadd --gid 1000 appgroup && \
    useradd --uid 1000 --gid appgroup --shell /bin/bash appuser

# 切换到非 root 用户
USER appuser

# 注意：在 USER 之前确保所有需要的权限已设置
```

#### 清理不必要的文件

```dockerfile
# 清理缓存和临时文件
RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 清理 npm 缓存
RUN npm ci --only=production && \
    npm cache clean --force
```

#### 使用 .dockerignore

```dockerignore
# .dockerignore
# 版本控制
.git
.gitignore

# 依赖
node_modules
__pycache__
*.pyc

# 开发文件
.env
.env.*
*.log

# IDE
.vscode
.idea
*.swp

# 测试
coverage
*.test.js
*.spec.js
__tests__

# 构建产物
dist
build
.next
.nuxt
```

### 完整的 Node.js Dockerfile

```dockerfile
# Dockerfile
# ─────────────────────────────────────────────────────────────
# 多阶段构建：构建阶段
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装所有依赖（包含 devDependencies 用于构建）
RUN npm ci

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# ─────────────────────────────────────────────────────────────
# 多阶段构建：生产阶段
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS production

WORKDIR /app

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# 复制 package.json（用于依赖安装）
COPY package*.json ./

# 安装生产依赖
RUN npm ci --only=production && npm cache clean --force

# 复制构建产物（从 builder 阶段）
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000

# 切换用户
USER nodejs

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# 启动命令
CMD ["node", "dist/server.js"]
```

---

## 多阶段构建（Multi-stage）

### 多阶段构建原理

多阶段构建允许在单个 Dockerfile 中使用多个 FROM 指令，每个阶段可以有不同的基础镜像，最终只保留需要的产物：

```
┌─────────────────────────────────────────────────────────────┐
│  Stage 1: builder                                           │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ node:20-alpine                                       │   │
│  │   ├── npm ci                                         │   │
│  │   ├── 编译 TypeScript                                │   │
│  │   ├── 安装所有 devDependencies                       │   │
│  │   └── 输出：dist/, node_modules/（含构建工具）        │   │
│  └─────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│  Stage 2: production                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ node:20-alpine-slim                                  │   │
│  │   └── 仅复制最终需要的文件                           │   │
│  │       ├── dist/（编译产物）                          │   │
│  │       ├── node_modules/（仅生产依赖）                │   │
│  │       └── 丢弃：源代码、构建工具、构建缓存            │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Python 多阶段构建

```dockerfile
# ─────────────────────────────────────────────────────────────
# Stage 1: Build
# ─────────────────────────────────────────────────────────────
FROM python:3.12-slim AS builder

WORKDIR /app

# 安装构建依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        libpq-dev \
        && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装所有依赖
RUN pip install --user -r requirements.txt

# 复制源代码
COPY . .

# ─────────────────────────────────────────────────────────────
# Stage 2: Production
# ─────────────────────────────────────────────────────────────
FROM python:3.12-slim AS production

# 安装运行时依赖（不含编译工具）
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libpq5 \
        && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 复制依赖（从 builder）
COPY --from=builder /root/.local /root/.local
ENV PATH=/root/.local/bin:$PATH

# 复制应用代码
COPY --chown=1000:1000 . .

# 非 root 用户
USER 1000

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Go 多阶段构建

```dockerfile
# ─────────────────────────────────────────────────────────────
# Stage 1: Build
# ─────────────────────────────────────────────────────────────
FROM golang:1.23-alpine AS builder

WORKDIR /app

# 安装构建依赖
RUN apk add --no-cache git

# 复制 go mod
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码
COPY . .

# 编译二进制（静态链接）
RUN CGO_ENABLED=0 GOOS=linux go build \
    -ldflags '-w -s' \
    -o /app/server \
    ./cmd/server

# ─────────────────────────────────────────────────────────────
# Stage 2: Production
# ─────────────────────────────────────────────────────────────
FROM alpine:3.19 AS production

RUN apk add --no-cache ca-certificates tzdata

WORKDIR /app

# 复制二进制
COPY --from=builder /app/server .

# 创建非 root 用户
RUN adduser -D -u 1000 appuser
USER appuser

EXPOSE 8080

CMD ["./server"]
```

---

## Docker Compose 完整指南

### Docker Compose 核心概念

Docker Compose 是一个用于定义和运行多容器应用的工具，通过 YAML 文件描述服务、网络和卷，使用单个命令启动整个应用栈。

### Next.js + PostgreSQL + Redis 完整示例

```yaml
# docker-compose.yml
# ─────────────────────────────────────────────────────────────
# Next.js + PostgreSQL + Redis 完整架构
# ─────────────────────────────────────────────────────────────

services:
  # ───────────────────────────────────────────────────────
  # Web 应用
  # ───────────────────────────────────────────────────────
  web:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: myapp-web
    ports:
      - "3000:3000"
    environment:
      # 数据库连接
      DATABASE_URL: postgresql://appuser:apppassword@db:5432/myapp
      # Redis 连接
      REDIS_URL: redis://cache:6379/0
      # Node 环境
      NODE_ENV: production
      # Next.js 配置
      NEXT_PUBLIC_API_URL: http://localhost:3000
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    restart: unless-stopped
    networks:
      - backend
    volumes:
      # 绑定挂载：开发时热重载
      - ./src:/app/src:ro
      # 命名卷：生产数据持久化
      - app_uploads:/app/uploads

  # ───────────────────────────────────────────────────────
  # PostgreSQL 数据库
  # ───────────────────────────────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: myapp-db
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
      POSTGRES_DB: myapp
      # 性能优化
      POSTGRES_MAX_CONNECTIONS: 100
      POSTGRES_SHARED_BUFFERS: 256MB
      POSTGRES_EFFECTIVE_CACHE_SIZE: 1GB
    ports:
      - "5432:5432"
    volumes:
      # 命名卷：数据持久化
      - postgres_data:/var/lib/postgresql/data
      # 初始化脚本
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U appuser -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: unless-stopped
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G

  # ───────────────────────────────────────────────────────
  # Redis 缓存
  # ───────────────────────────────────────────────────────
  cache:
    image: redis:7-alpine
    container_name: myapp-redis
    command: >
      redis-server
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --save 60 1
      --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: unless-stopped
    networks:
      - backend
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  # ───────────────────────────────────────────────────────
  # Nginx 反向代理（可选）
  # ───────────────────────────────────────────────────────
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./docker/nginx/conf.d:/etc/nginx/conf.d:ro
      - nginx_certs:/etc/nginx/certs:ro
    depends_on:
      - web
    restart: unless-stopped
    networks:
      - backend

# ─────────────────────────────────────────────────────────────
# 网络配置
# ─────────────────────────────────────────────────────────────
networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16

# ─────────────────────────────────────────────────────────────
# 卷配置
# ─────────────────────────────────────────────────────────────
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  app_uploads:
    driver: local
  nginx_certs:
    driver: local
```

### 常用 Docker Compose 命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `docker compose up -d` | 后台启动所有服务 | `docker compose up -d --build` |
| `docker compose down` | 停止并删除容器 | `docker compose down -v`（删除卷）|
| `docker compose ps` | 查看服务状态 | `docker compose ps` |
| `docker compose logs -f` | 查看日志 | `docker compose logs -f web` |
| `docker compose exec` | 在容器中执行命令 | `docker compose exec db psql` |
| `docker compose restart` | 重启服务 | `docker compose restart web` |
| `docker compose build` | 重新构建镜像 | `docker compose build --no-cache` |
| `docker compose scale` | 扩展服务实例 | `docker compose up -d --scale web=3` |

---

## Volume 与 Bind Mount

### 卷类型对比

| 特性 | 匿名卷（Anonymous） | 命名卷（Named Volume） | 绑定挂载（Bind Mount） |
|------|---------------------|----------------------|-----------------------|
| **语法** | `-v /path` | `-v name:/path` | `-v /host:/container` |
| **数据持久化** | 容器删除后丢失 | 容器删除后保留 | 取决于主机 |
| **用途** | 临时存储 | 生产数据 | 开发/配置 |
| **性能** | 优 | 优 | 取决于文件系统 |
| **跨平台** | ✓ | ✓ | 需注意路径格式 |

### 命名卷实战

```yaml
# docker-compose.yml
services:
  db:
    image: postgres:16-alpine
    volumes:
      # 命名卷：持久化数据
      - postgres_data:/var/lib/postgresql/data
      # 绑定挂载：配置文件（开发用）
      - ./config/postgresql.conf:/etc/postgresql/postgresql.conf

volumes:
  postgres_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /mnt/data/postgres
```

### 权限问题处理

```dockerfile
# 在 Dockerfile 中设置正确权限
RUN mkdir -p /app/uploads && \
    chown -R nodejs:nodejs /app/uploads

# 或在运行时处理
chmod 777 ./uploads  # 仅开发环境

# 使用 nfs volume（生产环境）
volumes:
  data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=192.168.1.100,rw
      device: ":/exports/data"
```

---

## 网络模式详解

### Docker 网络驱动

| 驱动 | 用途 | 特点 |
|------|------|------|
| **bridge** | 默认驱动，容器间通信 | 单主机，NAT 通信 |
| **host** | 移除网络隔离 | 最高性能，无端口映射 |
| **overlay** | 跨主机容器通信 | Docker Swarm |
| **macvlan** | 直接分配 MAC 地址 | 需网络支持 |
| **none** | 禁用网络 | 隔离容器 |

### 自定义网络实现服务发现

```yaml
# docker-compose.yml
services:
  web:
    networks:
      - frontend
      - backend
    # 服务名即为 DNS 名称
    environment:
      DATABASE_URL: postgresql://db:5432/myapp
      REDIS_URL: redis://cache:6379

  api:
    networks:
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # 禁用外部访问
```

---

## Docker vs Podman vs containerd 对比

### 核心对比

| 维度 | Docker | Podman | containerd |
|------|--------|--------|------------|
| **架构** | daemon 模式 | 无守护进程 | 仅运行时 |
| **权限要求** | root 或 docker 组 | 普通用户 | 需配合其他工具 |
| **镜像兼容** | OCI 标准（兼容） | OCI 标准（兼容） | OCI 标准 |
| **rootless** | ✗ | ✓ | ✗ |
| **构建能力** | ✓ 内置 | ✗（需 Buildah） | ✗ |
| **K8s 兼容性** | ✓ | ✓ | ✓（K8s 默认） |
| **学习曲线** | 低 | 中 | 高 |
| **企业采用** | ★★★★★ | ★★★☆☆ | ★★★★☆ |

### Podman + Buildah 替代方案

```bash
# Podman 使用（与 Docker 语法兼容）
alias docker=podman
podman build -t myapp .
podman run -d myapp

# Buildah 单独构建
buildah bud -t myapp .
```

> [!IMPORTANT]
> Podman 4.0+ 提供了与 Docker 完整的命令行兼容性，大多数场景可直接替换使用。

---

## Docker Desktop vs Orbstack

### macOS 容器运行时对比

| 维度 | Docker Desktop | Orbstack |
|------|----------------|----------|
| **性能** | 较慢（虚拟机） | 极快（macOS 原生） |
| **资源占用** | 高（完整 Linux VM） | 低 |
| **Rosetta 2** | ✗ | ✓（ARM/x86 转换） |
| **启动时间** | 10-30 秒 | < 1 秒 |
| **价格** | $0-$21/月 | $0-$8/月 |
| **Kubernetes** | 内置 | 需要手动配置 |
| **与 macOS 集成** | 一般 | 优秀 |

### Orbstack 配置示例

```bash
# 安装 Orbstack
brew install --cask orbstack

# 验证安装
orb version

# Docker 兼容模式
orb docker --version

# 迁移 Docker Desktop
orb migrate
```

---

## AI 应用容器化实战

### LangChain + PostgreSQL + Redis 架构

```yaml
# docker-compose.yml
# AI 应用完整架构
# ─────────────────────────────────────────────────────────────

services:
  # ───────────────────────────────────────────────────────
  # AI 应用核心
  # ───────────────────────────────────────────────────────
  app:
    build:
      context: .
      dockerfile: Dockerfile.ai
    container_name: ai-app
    environment:
      # LLM 配置
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      # 数据库
      DATABASE_URL: postgresql://aiuser:aipassword@postgres:5432/ai_db
      # 向量数据库
      PGVECTOR_CONNECTION_STRING: postgresql://aiuser:aipassword@postgres:5432/ai_db
      # 缓存
      REDIS_URL: redis://redis:6379/0
      # API 配置
      API_BASE_URL: http://app:8000
      CORS_ORIGINS: "http://localhost:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    networks:
      - ai-network

  # ───────────────────────────────────────────────────────
  # PostgreSQL（支持 pgvector）
  # ───────────────────────────────────────────────────────
  postgres:
    image: pgvector/pgvector:pg16
    container_name: ai-postgres
    environment:
      POSTGRES_USER: aiuser
      POSTGRES_PASSWORD: aipassword
      POSTGRES_DB: ai_db
    ports:
      - "5432:5432"
    volumes:
      - pgvector_data:/var/lib/postgresql/data
      - ./docker/postgres/postgresql.conf:/etc/postgresql/postgresql.conf
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U aiuser -d ai_db"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - ai-network
    deploy:
      resources:
        limits:
          memory: 2G

  # ───────────────────────────────────────────────────────
  # Redis（会话缓存 + 向量缓存）
  # ───────────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: ai-redis
    command: >
      redis-server
      --maxmemory 512mb
      --maxmemory-policy allkeys-lru
      --save 900 1
      --save 300 10
      --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: unless-stopped
    networks:
      - ai-network
    deploy:
      resources:
        limits:
          memory: 512M

networks:
  ai-network:
    driver: bridge

volumes:
  pgvector_data:
  redis_data:
```

### Dockerfile.ai

```dockerfile
# Dockerfile.ai - AI 应用专用
FROM python:3.12-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        build-essential \
        libpq-dev \
        && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

# 非 root 用户
RUN useradd -m -u 1000 appuser && \
    chown -R appuser:appuser /app
USER appuser

EXPOSE 8000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### requirements.txt 示例

```txt
# requirements.txt
# Web 框架
fastapi==0.115.0
uvicorn[standard]==0.32.0
pydantic==2.9.0

# AI/LLM
langchain==0.3.7
langchain-openai==0.2.6
langchain-anthropic==0.2.5
langchain-community==0.3.5
langchain-pgvector==0.2.0
pgvector==0.3.1

# 数据库
asyncpg==0.30.0
sqlalchemy[asyncio]==2.0.35
alembic==1.14.0

# Redis
redis==5.2.0
aioredis==2.0.2

# 向量嵌入
sentence-transformers==3.2.0
numpy==1.26.4

# 工具
python-dotenv==1.0.1
httpx==0.27.2
```

---

### 完整的 Node.js Dockerfile

```dockerfile
# Dockerfile
# ─────────────────────────────────────────────────────────────
# 多阶段构建：构建阶段
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装所有依赖（包含 devDependencies 用于构建）
RUN npm ci

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# ─────────────────────────────────────────────────────────────
# 多阶段构建：生产阶段
# ─────────────────────────────────────────────────────────────
FROM node:20-alpine AS production

WORKDIR /app

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# 复制 package.json（用于依赖安装）
COPY package*.json ./

# 安装生产依赖
RUN npm ci --only=production && npm cache clean --force

# 复制构建产物（从 builder 阶段）
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules

# 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000

# 切换用户
USER nodejs

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# 启动命令
CMD ["node", "dist/server.js"]
```

---

## 服务概述与定位

### Docker 在现代开发中的角色

Docker 不仅是容器化技术，更是现代软件交付的核心基础设施。在 vibecoding 工作流中，Docker 扮演着以下关键角色：

#### 本地开发环境标准化

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Docker 开发环境架构                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     开发者工作站                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │ │
│  │  │ Docker      │  │ Docker      │  │ Docker      │          │ │
│  │  │ Container   │  │ Container   │  │ Container   │          │ │
│  │  │ (Frontend)  │  │ (Backend)   │  │ (Database)  │          │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │ │
│  │        │                │                │                    │ │
│  │        └────────────────┼────────────────┘                    │ │
│  │                         │                                      │ │
│  │                  ┌─────┴─────┐                               │ │
│  │                  │  网络桥接   │                               │ │
│  │                  └───────────┘                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 生产环境一致性保证

Docker 确保开发、测试、生产环境完全一致，消除「在我机器上能运行」的经典问题。

| 环境 | 基础镜像 | 依赖版本 | 配置 |
|------|---------|----------|------|
| 开发 | `node:20-alpine` | package.json | 开发模式 |
| 测试 | `node:20-alpine` | package-lock.json | 测试配置 |
| 生产 | `node:20-alpine` | package-lock.json | 生产优化 |

#### 团队协作效率提升

通过 Dockerfile 和 docker-compose.yml，团队成员可以在几秒钟内获得完全一致的开发环境，无需手动安装依赖或配置系统。

### Docker 与 DevOps 文化

Docker 是 DevOps 实践的关键推动者：

```yaml
# 完整的 CI/CD 流水线
stages:
  - build    # 构建 Docker 镜像
  - test     # 在容器中运行测试
  - push     # 推送到镜像仓库
  - deploy   # 部署到各环境

# 每个阶段都使用相同的容器环境
.job_template: &job_template
  image: myapp:build-$CI_COMMIT_SHA
  services:
    - docker:dind  # Docker-in-Docker
```

### 容器化成熟度模型

| 级别 | 描述 | 特征 |
|------|------|------|
| **L1: 基础** | 使用已有镜像 | 仅运行单个容器 |
| **L2: 标准化** | 自定义 Dockerfile | 标准化构建流程 |
| **L3: 编排** | 使用 Docker Compose | 多容器协作 |
| **L4: 集群** | 使用 Swarm/K8s | 服务发现、负载均衡 |
| **L5: 平台** | 云原生架构 | 自动扩缩容、自愈 |

---

## 完整配置教程

### 开发环境配置

#### macOS 开发环境

```bash
# 1. 安装 Orbstack（推荐，比 Docker Desktop 更快）
brew install --cask orbstack

# 2. 验证安装
orb version

# 3. 启用 Docker 兼容模式
orb docker --version

# 4. 配置 Docker CLI
mkdir -p ~/.docker
cat > ~/.docker/config.json << 'EOF'
{
    "auths": {},
    "credsStore": "osxkeychain",
    "currentContext": "orbstack"
}
EOF

# 5. 验证 Docker 运行
docker info
docker run --rm hello-world
```

#### Linux 开发环境

```bash
# Ubuntu/Debian 安装
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release

# 添加 Docker GPG 密钥
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加 Docker 仓库
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 安装 Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 添加当前用户到 docker 组
sudo usermod -aG docker $USER
newgrp docker

# 启用 Docker 服务
sudo systemctl enable docker
sudo systemctl start docker
```

#### Windows WSL2 环境

```powershell
# 1. 安装 WSL2
wsl --install -d Ubuntu-22.04

# 2. 在 WSL2 中安装 Docker
sudo apt-get update
sudo apt-get install -y docker.io

# 3. 配置 Docker daemon
sudo nano /etc/docker/daemon.json
# {
#     "storage-driver": "overlay2",
#     "dns": ["8.8.8.8", "8.8.4.4"]
# }

# 4. 启动 Docker
sudo service docker start

# 5. 验证
docker run --rm hello-world
```

### Docker 配置深度优化

#### daemon.json 配置

```json
{
    "builder": {
        "gc": {
            "enabled": true,
            "defaultKeepStorage": "20GB"
        }
    },
    "experimental": false,
    "features": {
        "buildkit": true
    },
    "storage-driver": "overlay2",
    "storage-opts": [
        "overlay2.override_kernel_check=true"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "10m",
        "max-file": "3"
    },
    "metrics-addr": "127.0.0.1:9323",
    "registry-mirrors": [
        "https://mirror.ccs.tencentyun.com",
        "https://docker.mirrors.ustc.edu.cn"
    ],
    "insecure-registries": [],
    "live-restore": true,
    "userland-proxy": false,
    "default-ulimits": {
        "nofile": {
            "Name": "nofile",
            "Hard": 64000,
            "Soft": 64000
        }
    }
}
```

#### 网络配置

```bash
# 创建自定义 bridge 网络
docker network create \
    --driver bridge \
    --subnet 172.20.0.0/16 \
    --gateway 172.20.0.1 \
    --ip-range 172.20.5.0/24 \
    my-network

# 创建 macvlan 网络（需要网络支持）
docker network create \
    --driver macvlan \
    --subnet 192.168.1.0/24 \
    --gateway 192.168.1.1 \
    -o parent=eth0 \
    my-macvlan

# 查看网络详情
docker network inspect my-network
```

#### 资源限制配置

```bash
# 全局资源限制（在 daemon.json 中配置）
# {
#     "default-ulimits": {
#         "memory": {
#             "Name": "memory",
#             "Hard": 1073741824,  # 1GB
#             "Soft": 536870912     # 512MB
#         },
#         "pids": {
#             "Name": "pids",
#             "Hard": 512,
#             "Soft": 256
#         }
#     }
# }

# 为特定容器设置资源限制
docker run -d \
    --name myapp \
    --memory 512m \
    --memory-swap 1g \
    --cpus 1.5 \
    --cpuset-cpus "0,1" \
    --ulimit nofile=65536:65536 \
    myapp:latest

# 查看资源使用
docker stats
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

## 核心功能详解

### 镜像管理深度解析

#### 镜像层级与存储原理

Docker 镜像采用分层存储（Layered Filesystem）机制：

```
┌─────────────────────────────────────────────────────────────┐
│                    Docker 镜像层级结构                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  容器层 (Container Layer) - 可写                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │                                                     │   │
│  │  /app/src (应用代码)                                │   │
│  │  /app/uploads (上传文件)                             │   │
│  │                                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                           ↑                                  │
│                           │ COPY --from=builder            │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer N: 运行指令层 (run)                            │  │
│  │  /usr/local/bin (可执行文件)                          │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ COPY                            │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer N-1: 依赖安装层 (run npm ci)                   │  │
│  │  /app/node_modules                                    │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ COPY                            │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer N-2: package.json 层 (run)                      │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ COPY                            │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer N-3: 源代码层                                   │  │
│  │  /app/src                                             │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ WORKDIR                         │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer 2: 基础配置层 (环境变量、端口)                   │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ USER                           │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer 1: 用户创建层                                   │  │
│  └───────────────────────────────────────────────────────┘  │
│                           ↑                                  │
│                           │ FROM                            │
│  ┌─────────────────────────┴─────────────────────────────┐  │
│  │  Layer 0: 基础镜像层 (node:20-alpine)                  │  │
│  │  /bin, /usr, /lib, /etc ...                           │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

#### 镜像构建优化策略

```dockerfile
# 优化策略 1: 利用 BuildKit 缓存
# 构建时会自动并行处理没有依赖关系的步骤

# 优化策略 2: 精确控制 COPY 顺序
# 频繁变更的文件应该放在后面

# 优化策略 3: 使用 .dockerignore
# .dockerignore
node_modules
.git
.env*
*.log
npm-debug.log*
.DS_Store
coverage
.nyc_output
dist
build

# 优化策略 4: 合并 RUN 指令减少层数
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        package1 \
        package2 \
        package3 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# 优化策略 5: 使用多阶段构建减小镜像体积
FROM golang:1.23 AS builder
WORKDIR /app
COPY . .
RUN go build -o server .

FROM alpine:3.19
COPY --from=builder /app/server /server
CMD ["/server"]

# 优化策略 6: 标签使用特定版本
# ✓ 推荐
FROM node:20.11.0-alpine3.19
# ✗ 避免
FROM node:latest
FROM node:20
```

#### 镜像仓库操作

```bash
# 登录到私有仓库
docker login registry.example.com -u username

# 推送镜像
docker tag myapp:latest registry.example.com/myapp:v1.0.0
docker push registry.example.com/myapp:v1.0.0

# 拉取镜像
docker pull registry.example.com/myapp:v1.0.0

# 镜像构建缓存
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --tag myapp:latest \
    --cache-from type=registry,ref=myapp:buildcache \
    --cache-to type=registry,ref=myapp:buildcache,mode=max \
    --push \
    .

# 导出和导入镜像
docker save -o myapp.tar myapp:latest
docker load -i myapp.tar

# 压缩镜像（用于传输）
docker save myapp:latest | gzip > myapp.tar.gz
docker load -i <(gunzip -c myapp.tar.gz)
```

### 容器生命周期管理

#### 容器状态机

```
                    ┌──────────────┐
                    │   CREATED    │
                    └──────┬───────┘
                           │ docker run / docker create
                           ▼
                    ┌──────────────┐
         ┌──────────│   RUNNING    │──────────┐
         │          └──────┬───────┘          │
         │                 │                  │
         │    docker stop  │ docker kill      │ docker restart
         │                 │                  │
         ▼                 ▼                  │
    ┌─────────┐      ┌─────────┐            │
    │  EXITED │      │  PAUSED │            │
    └────┬────┘      └────┬────┘            │
         │                 │                  │
         │   docker unpause                  │
         │                 │                  │
         │    docker rm    │                 │
         │                 │                 │
         ▼                 ▼                 ▼
    ┌─────────────────────────────────────────┐
    │              DELETED                    │
    └─────────────────────────────────────────┘
```

#### 容器操作命令

```bash
# 创建容器（不启动）
docker create --name myapp \
    -e NODE_ENV=production \
    -p 3000:3000 \
    -v $(pwd)/data:/data \
    myapp:latest

# 启动容器
docker start myapp

# 停止容器
docker stop myapp
docker stop myapp --time 30  # 30秒后强制停止

# 重启容器
docker restart myapp
docker restart -t 60 myapp  # 60秒超时

# 暂停/恢复容器
docker pause myapp
docker unpause myapp

# 进入运行中的容器
docker exec -it myapp /bin/sh
docker exec -it myapp bash
docker exec -u root myapp bash  # 以 root 身份

# 查看容器进程
docker top myapp

# 查看容器详细信息
docker inspect myapp
docker inspect --format '{{.NetworkSettings.IPAddress}}' myapp

# 查看端口映射
docker port myapp

# 查看容器日志
docker logs myapp
docker logs -f myapp
docker logs --tail 100 myapp
docker logs --since 2024-01-01 myapp
docker logs --since 10m myapp

# 复制文件
docker cp myapp:/app/config.json ./config.json
docker cp ./config.json myapp:/app/config.json

# 查看容器文件系统变化
docker diff myapp
# A: 添加  C: 改变  D: 删除
```

### 网络深度配置

#### Docker 网络驱动详解

```bash
# 1. Bridge 网络（默认）
docker network create my-bridge
docker run --network my-bridge --name app1 myapp
docker run --network my-bridge --name app2 myapp
# app1 和 app2 可以通过容器名互相通信

# 2. Host 网络（直接使用主机网络）
docker run --network host --name myapp myapp
# 容器直接使用主机网络栈，适合高性能场景

# 3. Overlay 网络（Swarm 集群）
docker network create \
    --driver overlay \
    --attachable \
    my-overlay
docker service create --network my-overlay myapp

# 4. Macvlan 网络（直接分配 MAC）
docker network create \
    --driver macvlan \
    --subnet 192.168.1.0/24 \
    --gateway 192.168.1.1 \
    -o parent=eth0 \
    my-macvlan

# 5. None 网络（完全隔离）
docker run --network none myapp
```

#### 服务发现机制

```yaml
# docker-compose.yml - 服务发现示例
services:
  web:
    networks:
      - frontend
      - backend
    environment:
      # 使用服务名作为主机名
      DATABASE_URL: postgresql://db:5432/myapp
      REDIS_URL: redis://cache:6379/0
      API_URL: http://api:8080/api

  api:
    networks:
      - backend
    environment:
      DATABASE_URL: postgresql://db:5432/myapp
      CACHE_URL: redis://cache:6379/1

  db:
    networks:
      - backend
    # 数据库不需要暴露端口

  cache:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # 禁止外部访问
```

#### DNS 配置

```bash
# 查看容器 DNS 配置
docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' myapp
docker exec myapp cat /etc/resolv.conf

# 自定义 DNS 服务器
docker run --dns 8.8.8.8 --dns 8.8.4.4 myapp

# 添加 hosts 条目
docker run --add-host myhost:192.168.1.100 myapp

# 容器内访问主机
# macOS: 使用 host.docker.internal
# Linux: 需要额外配置
docker run --add-host=host.docker.internal:host-gateway myapp

# Windows
docker run --add-host=host.docker.internal:host-gateway myapp
```

### 存储卷高级操作

#### 卷类型与选择

| 卷类型 | 使用场景 | 性能 | 跨容器共享 | 主机访问 |
|--------|----------|------|-----------|----------|
| **匿名卷** | 临时数据 | 高 | 否 | 否 |
| **命名卷** | 持久数据 | 高 | 是 | 有限 |
| **绑定挂载** | 配置文件/开发 | 最高 | 是 | 是 |
| **tmpfs** | 敏感数据 | 最高 | 否 | 否 |
| **nfs** | 分布式存储 | 中 | 是 | 是 |

#### tmpfs（内存存储）

```bash
# 将敏感数据存储在内存中
docker run -d \
    --name myapp \
    --tmpfs /run/secrets:rw,noexec,nosuid,size=1024m \
    myapp:latest

# tmpfs 选项
# rw: 读写
# ro: 只读
# noexec: 禁止执行
# nosuid: 禁止设置 SUID
# nodev: 禁止设备文件
# size: 最大大小
# mode: 权限模式
```

#### 备份与恢复

```bash
# 备份命名卷
docker run --rm \
    -v myvolume:/source \
    -v $(pwd):/backup \
    alpine \
    tar czf /backup/myvolume-backup.tar.gz -C /source .

# 恢复备份
docker volume create myvolume
docker run --rm \
    -v myvolume:/target \
    -v $(pwd):/backup \
    alpine \
    sh -c "tar xzf /backup/myvolume-backup.tar.gz -C /target"

# 查看卷信息
docker volume inspect myvolume
docker volume ls
docker volume ls -f dangling=true  # 未使用的卷
```

---

## 部署配置（Dockerfile 等）

### Java 应用 Dockerfile

```dockerfile
# multi-stage Dockerfile for Java/Spring Boot
# ─────────────────────────────────────────────────────────────
# Stage 1: Build
# ─────────────────────────────────────────────────────────────
FROM maven:3.9-eclipse-temurin-21 AS builder

WORKDIR /app

# 复制 pom.xml（利用缓存）
COPY pom.xml .
RUN mvn dependency:go-offline -B

# 复制源代码
COPY src ./src
COPY config ./config

# 构建应用
RUN mvn package -DskipTests -B

# ─────────────────────────────────────────────────────────────
# Stage 2: Runtime
# ─────────────────────────────────────────────────────────────
FROM eclipse-temurin:21-jre-alpine

# 创建非 root 用户
RUN groupadd -g 1000 appgroup && \
    useradd -r -u 1000 -g appgroup -s /bin/false appuser

WORKDIR /app

# 复制构建产物
COPY --from=builder /app/target/*.jar app.jar
COPY --from=builder /app/config ./config

# 创建日志目录
RUN mkdir -p /app/logs && chown -R appuser:appgroup /app

# 切换用户
USER appuser

# 暴露端口
EXPOSE 8080

# 健康检查
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
    CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

# JVM 优化配置
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0 -XX:+UseG1GC"

# 启动命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### .NET 应用 Dockerfile

```dockerfile
# multi-stage Dockerfile for .NET
# ─────────────────────────────────────────────────────────────
# Stage 1: Build
# ─────────────────────────────────────────────────────────────
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS builder

WORKDIR /src

# 复制 csproj（利用缓存）
COPY *.csproj ./
RUN dotnet restore

# 复制源代码
COPY . .

# 构建应用
RUN dotnet publish -c Release -o /app/publish --no-restore

# ─────────────────────────────────────────────────────────────
# Stage 2: Runtime
# ─────────────────────────────────────────────────────────────
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS production

WORKDIR /app

# 创建非 root 用户
RUN groupadd -g 1000 appgroup && \
    useradd -r -u 1000 -g appgroup -s /bin/false appuser

# 复制构建产物
COPY --from=builder /app/publish .

# 设置权限
RUN chown -R appuser:appgroup /app

# 切换用户
USER appuser

# 暴露端口
EXPOSE 8080

# 环境变量
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

# 启动
ENTRYPOINT ["dotnet", "app.dll"]
```

### Ruby on Rails Dockerfile

```dockerfile
# multi-stage Dockerfile for Rails
# ─────────────────────────────────────────────────────────────
# Stage 1: Assets Build
# ─────────────────────────────────────────────────────────────
FROM ruby:3.3-slim AS builder

# 安装系统依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    libvips \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 安装 bundler
RUN gem install bundler:2.5.0

# 复制依赖文件
COPY Gemfile Gemfile.lock ./
RUN bundle config set --local deployment 'true' && \
    bundle config set --local without 'development test' && \
    bundle install --jobs 4 --retry 3

# 复制源代码
COPY . .

# 安装 nodejs（用于资产编译）
RUN apt-get update && apt-get install -y curl \
    && curl -fsSL https://deb.nodesource.com/setup_20.x | bash - \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# 预编译资产
RUN bundle exec rails assets:precompile

# ─────────────────────────────────────────────────────────────
# Stage 2: Production
# ─────────────────────────────────────────────────────────────
FROM ruby:3.3-slim AS production

# 安装运行时依赖
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    libvips \
    curl \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# 创建非 root 用户
RUN groupadd -g 1000 rails && \
    useradd -r -u 1000 -g rails -s /bin/false rails

# 复制依赖和源代码
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder --chown=rails:rails /app .

# 创建必要目录
RUN mkdir -p tmp/pids log && \
    chown -R rails:rails log tmp

# 切换用户
USER rails

# 环境变量
ENV RAILS_ENV=production
ENV SECRET_KEY_BASE=dummy_for_precompile

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:3000/health || exit 1

# 启动
CMD ["bin/rails", "server", "-b", "0.0.0.0"]
```

---

## 环境变量与密钥管理

### 环境变量配置

#### 构建时 vs 运行时变量

| 类型 | 说明 | 示例 | 安全性 |
|------|------|------|--------|
| **ARG** | 构建时变量 | `ARG VERSION` | ✗ 可见于镜像历史 |
| **ENV** | 运行时变量 | `ENV NODE_ENV=production` | ✓ 可被覆盖 |
| **SECRET** | 敏感信息 | `ARG GITHUB_TOKEN` | ✗ 需谨慎 |
| **.env** | 本地配置 | `DATABASE_URL=...` | ✓ 不提交到 Git |

#### 安全的环境变量策略

```bash
# 1. 使用 .env 文件（不提交到版本控制）
cat > .env << 'EOF'
# 数据库
DATABASE_URL=postgresql://user:password@host:5432/db
DATABASE_PASSWORD=secret123

# API 密钥
OPENAI_API_KEY=sk-xxxxx
STRIPE_SECRET_KEY=sk_test_xxxxx

# JWT 密钥
JWT_SECRET=your-super-secret-jwt-key-here
EOF

# 2. .dockerignore 排除
cat >> .dockerignore << 'EOF'
.env
.env.*
!.env.example
EOF

# 3. 运行时注入敏感变量
docker run -d \
    --name myapp \
    --env-file .env \
    --env DATABASE_PASSWORD=runtime_secret \
    -e API_KEY=non_secret_key \
    myapp:latest

# 4. 从 Docker Secrets 读取（Swarm 模式）
docker secret create db_password password.txt
docker service create \
    --secret db_password \
    --env FILE="/run/secrets/db_password" \
    myapp
```

### 密钥管理最佳实践

#### 使用 Docker Secrets（Swarm 模式）

```bash
# 创建 secrets
echo "my-super-secret-password" | docker secret create db_password -
docker secret create ssl_cert ./cert.pem
docker secret create ssl_key ./key.pem

# 在服务中使用
docker service create \
    --name myapp \
    --secret source=db_password,target=DATABASE_PASSWORD \
    --secret source=ssl_cert \
    --secret source=ssl_key \
    myapp:latest

# 服务内访问
# 文件路径: /run/secrets/<target>
```

#### 外部密钥管理集成

```yaml
# docker-compose.yml - 使用 Vault
services:
  app:
    image: myapp:latest
    environment:
      # Vault Agent 自动注入
      DATABASE_PASSWORD_FILE: /vault/secrets/db_password
    volumes:
      - vault-secrets:/vault/secrets:ro

volumes:
  vault-secrets:
    driver: local
```

#### Kubernetes Secret 集成

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secrets
type: Opaque
stringData:
  DATABASE_URL: postgresql://user:password@db:5432/db
  API_KEY: sk-xxxxx
---
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
    - name: myapp
      image: myapp:latest
      envFrom:
        - secretRef:
            name: myapp-secrets
      env:
        - name: NODE_ENV
          value: "production"
```

---

## CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/docker.yml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─────────────────────────────────────────────────────────
  # Job 1: 构建和测试
  # ─────────────────────────────────────────────────────────
  build-test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Cache Docker layers
        uses: actions/cache@v4
        with:
          path: /tmp/.buildx-cache
          key: ${{ runner.os }}-buildx-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-buildx-

      - name: Build image
        uses: docker/build-push-action@v5
        with:
          context: .
          load: true
          tags: myapp:test
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max
          env:
            DATABASE_URL: postgresql://test:test@postgres:5432/test

      - name: Run tests
        run: |
          docker run --rm \
            --network host \
            -e DATABASE_URL=postgresql://test:test@localhost:5432/test \
            myapp:test \
            npm test

      - name: Image metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,prefix=

      - name: Push to Registry
        if: github.event_name != 'pull_request'
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=local,src=/tmp/.buildx-cache
          cache-to: type=local,dest=/tmp/.buildx-cache-new,mode=max

      # Fix cache directory
      - name: Move cache
        run: |
          rm -rf /tmp/.buildx-cache
          mv /tmp/.buildx-cache-new /tmp/.buildx-cache

  # ─────────────────────────────────────────────────────────
  # Job 2: 安全扫描
  # ─────────────────────────────────────────────────────────
  security:
    runs-on: ubuntu-latest
    needs: build-test
    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # ─────────────────────────────────────────────────────────
  # Job 3: 部署到服务器
  # ─────────────────────────────────────────────────────────
  deploy:
    runs-on: ubuntu-latest
    needs: security
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to Server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USER }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            docker stop myapp || true
            docker rm myapp || true
            docker run -d \
              --name myapp \
              --restart unless-stopped \
              -p 80:8080 \
              -p 443:8443 \
              --env-file /opt/myapp/.env \
              ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            docker image prune -f
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - scan
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"

services:
  - docker:dind

# ─────────────────────────────────────────────────────────
# Build stage
# ─────────────────────────────────────────────────────────
build:
  stage: build
  image: docker:latest
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker build -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  artifacts:
    paths:
      - build/

# ─────────────────────────────────────────────────────────
# Test stage
# ─────────────────────────────────────────────────────────
test:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  services:
    - postgres:16-alpine
  variables:
    POSTGRES_DB: test
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
    DATABASE_URL: postgresql://test:test@postgres:5432/test
  script:
    - npm ci
    - npm test

# ─────────────────────────────────────────────────────────
# Security scan
# ─────────────────────────────────────────────────────────
security:
  stage: scan
  image: aquasec/trivy:latest
  script:
    - trivy image --exit-code 0 --severity HIGH,CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  allow_failure: true

# ─────────────────────────────────────────────────────────
# Deploy to staging
# ─────────────────────────────────────────────────────────
deploy-staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $STAGING_HOST >> ~/.ssh/known_hosts
    - scp -r ./deploy staging-user@$STAGING_HOST:/opt/
    - ssh staging-user@$STAGING_HOST "cd /opt/deploy && docker-compose -f docker-compose.staging.yml up -d"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main

# ─────────────────────────────────────────────────────────
# Deploy to production
# ─────────────────────────────────────────────────────────
deploy-production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - eval $(ssh-agent -s)
    - echo "$SSH_PRIVATE_KEY" | tr -d '\r' | ssh-add -
    - mkdir -p ~/.ssh
    - chmod 700 ~/.ssh
    - ssh-keyscan $PROD_HOST >> ~/.ssh/known_hosts
    - scp -r ./deploy prod-user@$PROD_HOST:/opt/
    - ssh prod-user@$PROD_HOST "cd /opt/deploy && docker-compose -f docker-compose.production.yml up -d"
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - tags
```

### Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent {
        docker {
            image 'docker:latest'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'myapp'
        DOCKER_CREDS = credentials('docker-registry')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    def imageTag = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag

                    sh """
                        docker build \
                            --tag ${REGISTRY}/${IMAGE_NAME}:${imageTag} \
                            --tag ${REGISTRY}/${IMAGE_NAME}:latest \
                            .
                    """
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    sh """
                        docker run --rm \
                            --network=mynetwork \
                            --env-file .env.test \
                            ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                            npm test
                    """
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    sh """
                        docker run --rm \
                            -v /var/run/docker.sock:/var/run/docker.sock \
                            aquasec/trivy:latest \
                            image --exit-code 1 \
                            --severity HIGH,CRITICAL \
                            ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Push') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh """
                        echo ${DOCKER_CREDS} | docker login -u _json_key --password-stdin ${REGISTRY}
                        docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${REGISTRY}/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                script {
                    def remote = [:]
                    remote.name = 'prod-server'
                    remote.host = 'prod.example.com'
                    remote.user = 'deploy'
                    remote.privateKey = '/path/to/key'
                    remote.knownHosts = '/path/to/known_hosts'

                    sshCommand remote: remote, command: """
                        cd /opt/myapp
                        docker-compose pull
                        docker-compose up -d
                        docker image prune -f
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

---

## 性能优化与缓存策略

### 镜像构建优化

#### BuildKit 缓存配置

```bash
# 启用 BuildKit
export DOCKER_BUILDKIT=1

# 或在 daemon.json 中启用
# {
#     "features": {
#         "buildkit": true
#     }
# }

# 使用 GHA 缓存
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --tag myapp:latest \
    --cache-from type=gha,scope=myapp \
    --cache-to type=gha,mode=max,scope=myapp \
    --push \
    .

# 使用本地缓存
docker buildx build \
    --tag myapp:latest \
    --cache-from type=local,src=/tmp/cache \
    --cache-to type=local,dest=/tmp/cache-new,mode=max \
    --push \
    .

# 使用 registry 缓存
docker buildx build \
    --tag myapp:latest \
    --cache-from type=registry,ref=myapp:buildcache \
    --cache-to type=registry,ref=myapp:buildcache,mode=max \
    --push \
    .
```

#### 并行构建优化

```bash
# BuildKit 自动并行化
# Dockerfile
# 所有 FROM 指令并行构建
# 所有 RUN 指令按依赖关系并行执行

# 自定义并行度
DOCKER_BUILDKIT=1 COMPOSE_DOCKER_CLI_BUILD=1 docker compose build --parallel
```

### 运行时性能优化

#### 容器资源配置

```bash
# 内存优化
docker run -d \
    --name myapp \
    --memory 512m \
    --memory-swap 1g \
    --memory-reservation 256m \
    --memory-swappiness 60 \
    myapp:latest

# CPU 优化
docker run -d \
    --name myapp \
    --cpus 2 \
    --cpuset-cpus "0,1" \
    --cpu-period 100000 \
    --cpu-quota 50000 \
    myapp:latest

# IO 优化
docker run -d \
    --name myapp \
    --blkio-weight 500 \
    --device-read-bps /dev/sda:10mb \
    --device-write-bps /dev/sda:10mb \
    --device-read-iops /dev/sda:1000 \
    --device-write-iops /dev/sda:1000 \
    myapp:latest
```

#### JVM 容器感知配置

```dockerfile
# JVM 自动检测容器资源
ENV JAVA_OPTS="\
    -XX:+UseContainerSupport \
    -XX:MaxRAMPercentage=75.0 \
    -XX:InitialRAMPercentage=50.0 \
    -XX:+UseG1GC \
    -XX:+ExitOnOutOfMemoryError"
```

### 缓存策略

#### HTTP 缓存头配置

```nginx
# nginx.conf
server {
    # 静态资源长期缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # HTML 不缓存
    location ~* \.html$ {
        expires -1;
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }

    # API 响应
    location /api/ {
        expires -1;
        add_header Cache-Control "no-cache";
    }
}
```

#### Redis 缓存策略

```conf
# redis.conf
# 内存策略
maxmemory 512mb
maxmemory-policy allkeys-lru

# 持久化
save 900 1
save 300 10
save 60 10000

# AOF 持久化
appendonly yes
appendfsync everysec

# 压缩
rdbcompression yes
rdbchecksum yes
```

---

## 成本估算与选型建议

### Docker 成本因素

| 成本项 | 说明 | 优化建议 |
|--------|------|----------|
| **存储** | 镜像大小 | 使用 Alpine 基础镜像，多阶段构建 |
| **带宽** | 镜像拉取 | 使用私有仓库，配置镜像缓存 |
| **计算** | 构建时间 | 并行构建，利用缓存 |
| **运维** | 维护成本 | 自动化 CI/CD，选择托管服务 |

### 选型建议矩阵

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| **个人开发** | Docker Desktop/Orbstack | 简单易用 |
| **小型团队** | 托管镜像仓库 | 减少运维负担 |
| **中大型团队** | 自建镜像仓库 | 成本控制，安全性 |
| **CI/CD** | BuildKit 缓存 | 加速构建 |
| **生产环境** | Kubernetes + 私有仓库 | 规模化运维 |

---

## 常见问题与解决方案

### 构建问题

#### 问题：构建缓存失效

```bash
# 症状：每次构建都重新下载依赖
# 原因：package.json 变更导致缓存失效

# 解决方案：优化 Dockerfile 层顺序
# Dockerfile
COPY package*.json ./
RUN npm ci  # 先安装依赖（缓存）
COPY . .    # 再复制代码
```

#### 问题：多平台构建失败

```bash
# 症状：ARM 镜像在 x86 上构建失败
# 解决方案：使用 QEMU 或交叉编译

# 启用 experimental 构建器
docker buildx create --use --name mybuilder
docker buildx inspect mybuilder --bootstrap

# 构建多平台镜像
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --tag myapp:latest \
    --push \
    .
```

### 运行问题

#### 问题：容器无法启动

```bash
# 检查日志
docker logs myapp

# 检查容器状态
docker inspect myapp

# 常见原因：
# 1. 端口冲突
docker run -d -p 8080:8080 myapp  # 8080 已被占用
docker run -d -p 3000:8080 myapp   # 使用不同的主机端口

# 2. 权限问题
docker run -d --user 1000 myapp   # 以非 root 运行
chmod 755 ./entrypoint.sh          # 修复脚本权限

# 3. 依赖服务未就绪
docker-compose up -d db            # 先启动依赖
docker-compose up -d app           # 再启动应用
```

#### 问题：容器频繁重启

```bash
# 检查重启次数
docker ps --format "table {{.Names}}\t{{.Status}}"

# 检查 OOM
docker inspect myapp | grep -i oom

# 解决方案：增加内存限制
docker run -d --memory 1g myapp

# 检查健康检查
docker inspect myapp | grep -A 20 Health
```

### 网络问题

#### 问题：容器间无法通信

```bash
# 检查网络
docker network inspect bridge
docker network ls

# 创建网络并连接
docker network create mynet
docker network connect mynet container1
docker network connect mynet container2

# 使用 docker-compose（推荐）
# docker-compose.yml
services:
  app:
    networks:
      - mynet
networks:
  mynet:
    driver: bridge
```

#### 问题：无法从容器访问外网

```bash
# 检查 DNS
docker exec myapp ping google.com
docker exec myapp cat /etc/resolv.conf

# 添加 DNS
docker run --dns 8.8.8.8 myapp

# 检查 iptables
sudo iptables -L -n
sudo sysctl net.ipv4.ip_forward=1
```

### 存储问题

#### 问题：数据丢失

```bash
# 使用命名卷持久化数据
docker run -d \
    -v mydata:/data \
    myapp

# 检查卷
docker volume inspect mydata

# 备份数据
docker run --rm \
    -v mydata:/data \
    -v $(pwd):/backup \
    alpine \
    tar czf /backup/backup.tar.gz -C /data .

# 恢复数据
docker run --rm \
    -v mydata:/data \
    -v $(pwd):/backup \
    alpine \
    tar xzf /backup/backup.tar.gz -C /data
```

#### 问题：磁盘空间不足

```bash
# 清理未使用的资源
docker system prune -a
docker system prune --volumes

# 清理特定资源
docker image prune -a
docker container prune
docker volume prune
docker network prune

# 查看磁盘使用
docker system df
```

### 安全问题

#### 问题：镜像漏洞

```bash
# 扫描镜像
docker scout cves myapp:latest
trivy image myapp:latest
grype myapp:latest

# 最小化基础镜像
FROM alpine:3.19
FROM node:20-alpine
FROM python:3.12-slim

# 定期更新镜像
docker pull node:20-alpine
docker build --no-cache -t myapp .
```

---

> [!SUCCESS]
> Docker 是 vibecoding 时代的基础设施核心：本地开发用它构建一致环境，CI/CD 用它自动化测试和构建，生产环境用它部署服务。掌握 Dockerfile 最佳实践、多阶段构建、Docker Compose 编排，就掌握了云原生应用开发的核心技能。
