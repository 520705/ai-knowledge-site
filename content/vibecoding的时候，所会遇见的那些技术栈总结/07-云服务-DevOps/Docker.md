---
date: 2026-04-24
tags:
  - Docker
  - 容器化
  - DevOps
  - 云原生
description: Docker 容器化技术完整指南，涵盖镜像构建、最佳实践、Compose 编排、AI 应用容器化实战。
---

# Docker：容器化技术的工业标准

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Docker 27 新特性、多阶段构建、Docker Compose 完整指南，以及 AI 应用容器化实战技巧。面向零基础读者，用最通俗的语言解释容器的来龙去脉。

---

## Docker 是什么？为什么要用它？

### 从搬家说起理解容器

想象你要搬到一个新城市住。你有两个选择：第一，带着你的旧房子一起搬走（这就是虚拟机，把整个操作系统都打包了）；第二，只打包你的个人物品，让新城市的物业提供标准化的房间（这就是容器，只打包应用本身和它真正需要的东西，其他基础设施由宿主机提供）。

Docker 就是帮你做第二种选择的工具。它把应用程序和它依赖的所有东西——代码、运行时、系统工具、库文件——全部打包进一个标准化的"盒子"里，然后可以在任何有 Docker 的机器上原封不动地运行。这个"盒子"我们叫它**镜像（Image）**，运行起来的实例叫**容器（Container）**。

### 容器 vs 虚拟机

很多人会把容器和虚拟机搞混，其实它们有本质区别。虚拟机模拟的是一整台计算机，需要运行完整的操作系统，消耗大量内存和磁盘空间，启动也慢。而容器只是进程级别的隔离，共享宿主机的内核，轻量得多。以下是具体对比：

| 对比维度 | 虚拟机 | Docker 容器 |
|---------|--------|-----------|
| 启动时间 | 1-5 分钟 | 毫秒到几秒 |
| 磁盘占用 | 几个 GB | 几十到几百 MB |
| 内存开销 | 完整系统，几 GB | 仅应用消耗，MB 级 |
| 隔离程度 | 完全隔离 | 进程级隔离 |
| 性能损耗 | 15-30% | 1-5% |
| 密度（每台机器能跑多少） | 10 个左右 | 100 个以上 |

对于个人开发者和小型团队来说，Docker 的性价比极高。一台普通的开发笔记本就能同时运行几十个不同项目的容器，每个项目之间互不干扰。

### Docker 解决了什么问题

在 vibecoding 模式下快速开发时，最让人头疼的事情之一就是"在我机器上能跑，在你机器上就跑不通"。团队成员的系统环境各不相同，Mac 和 Windows 的差异、系统版本的差异、依赖版本的差异，都会导致这种问题。

Docker 的核心价值就是**环境一致性**——开发环境、测试环境、生产环境用的是同一个镜像，任何人在任何地方启动容器，看到的行为都是一模一样的。代码写完 push 到 Git，CI/CD 自动构建镜像并部署到服务器，整个流程不需要开发者在服务器上手动配置环境。

---

## Docker 的核心概念

### 镜像（Image）

镜像可以理解为一个只读的模板，它包含了运行应用程序所需的所有内容：代码、运行时、系统依赖、环境变量、配置文件。你可以把它想象成一个菜谱，上面写着做某道菜需要哪些原料、按照什么步骤来做。

Docker Hub 是官方维护的镜像仓库，里面有各种现成的镜像，比如 `nginx`（网页服务器）、`postgres`（数据库）、`node`（Node.js 运行环境）。你可以直接用这些现成镜像来构建自己的应用，比如：

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["node", "server.js"]
```

这个 Dockerfile 定义了一个 Node.js 应用的镜像构建步骤。每一条指令（FROM、COPY、RUN、CMD）都会在镜像中产生一个只读的层（Layer）。层的存在让镜像可以被复用和共享——如果多个镜像都基于同一个 Node.js 基础镜像，它们会共享这个基础层，节省磁盘空间。

### 容器（Container）

容器是镜像的运行实例。你可以基于同一个镜像启动多个容器，它们彼此独立、互不干扰。就像一个班级里有多个学生，他们都在用同一本教材（镜像），但每个人在里面做的笔记（容器层的改动）只属于自己。

```bash
# 从镜像启动一个容器
docker run -d --name my-app -p 3000:3000 myapp:latest

# 查看正在运行的容器
docker ps

# 查看所有容器（包括已停止的）
docker ps -a

# 停止容器
docker stop my-app

# 删除容器
docker rm my-app
```

### Dockerfile 是什么

Dockerfile 是一个文本文件，里面写了一系列指令，Docker 根据这些指令来构建镜像。Dockerfile 是可版本控制的，你可以把它和代码一起提交到 Git 仓库里，团队成员拿到代码的同时也拿到了构建镜像的完整说明。

### 镜像仓库（Registry）

镜像仓库用来存储和分发镜像。最常用的是官方的 Docker Hub，就像代码托管在 GitHub 一样，镜像也可以托管在 Docker Hub 上。你也可以搭建自己的私有仓库，或者使用 GitHub Container Registry（ghcr.io）、Google Container Registry 等服务。

```bash
# 登录到 Docker Hub
docker login

# 给镜像打标签
docker tag myapp:latest myusername/myapp:v1.0.0

# 推送到远程仓库
docker push myusername/myapp:v1.0.0

# 从远程仓库拉取
docker pull myusername/myapp:v1.0.0
```

---

## Dockerfile 最佳实践

### 为什么 Dockerfile 很重要

一个写得好不好的 Dockerfile，直接决定了镜像的大小、构建的速度、安全性，以及最终容器的运行表现。很多初学者写的 Dockerfile 能工作，但镜像体积庞大（可能几个 GB），每次代码改动都导致整层依赖重新构建，安全性也有隐患。

下面介绍一些经过大量实践验证的最佳实践。

### 选择合适的基础镜像

基础镜像是你的镜像的第一层地基，选得好不好直接影响镜像的最终大小和安全系数。

```dockerfile
# ✓ 推荐：使用 Alpine 变体，体积极小
FROM node:20-alpine

# ✓ 推荐：slim 变体，不含文档和示例
FROM python:3.12-slim

# ✗ 避免：使用 latest 标签，版本不确定
FROM node:latest

# ✗ 避免：使用完整版镜像，体积过大
FROM python:3.12
```

Alpine 是一个专为容器设计的轻量级 Linux 发行版，基础镜像只有约 5MB，相比之下完整的 Debian 基础镜像可能超过 100MB。使用 Alpine 或 slim 变体可以让你的镜像体积缩小 90% 以上。

### 利用层缓存加速构建

Docker 在构建镜像时会按照 Dockerfile 中的指令逐层执行，每一层都会生成一个缓存。当代码发生改动时，只有从改动那一行往后的指令需要重新执行，前面的层可以复用缓存。

为了让缓存最大化利用，我们应该把**不常变动的内容放在前面，频繁变动的内容放在后面**。

```dockerfile
# 好的做法：先复制依赖文件并安装（不常变动）
COPY package*.json ./
RUN npm ci --only=production

# 后复制源代码（频繁变动）
COPY src/ ./src/
COPY public/ ./public/

# 最后才是启动命令（几乎不变）
CMD ["node", "server.js"]
```

这样的好处是：当你只改了几行代码时，依赖安装的层会被缓存，不需要每次都重新下载 npm 包。

### 多阶段构建：缩小体积的利器

多阶段构建允许在一个 Dockerfile 中使用多个 FROM 指令，每个阶段可以有不同的基础镜像和工具，最终只把需要的产物复制到最终镜像中。这对于编译型语言尤其有用。

以一个 Go 应用为例：编译需要完整的 Go 工具链，生成的二进制文件却只需要一个轻量的 Linux 环境就可以运行。使用多阶段构建可以让最终镜像从 800MB 缩小到 10MB 以内。

```dockerfile
# 第一阶段：编译
FROM golang:1.23-alpine AS builder

WORKDIR /app

# 安装构建依赖
RUN apk add --no-cache git

COPY go.mod go.sum ./
RUN go mod download

COPY . .
# CGO_ENABLED=0 禁用 C 绑定，生成纯静态二进制
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags '-w -s' -o server ./cmd/server

# 第二阶段：生产运行
FROM alpine:3.19

# 只复制运行时需要的证书和时区数据
RUN apk add --no-cache ca-certificates tzdata

WORKDIR /app

# 只复制编译好的二进制文件，不包含任何构建工具
COPY --from=builder /app/server .

# 创建非 root 用户
RUN adduser -D -u 1000 appuser
USER appuser

EXPOSE 8080
CMD ["./server"]
```

这段多阶段构建的精髓在于：第一阶段用了完整的 Go 编译器来编译代码，第二阶段却只用了 Alpine 基础镜像（~5MB），只复制了编译好的二进制文件。最终镜像的体积取决于你的应用大小，通常只有十几到几十 MB。

### 非 root 用户运行

从安全角度考虑，容器内的进程不应该以 root 用户身份运行。Docker 默认容器内是 root 用户，虽然有 Namespace 隔离，但以非 root 用户运行可以进一步减小攻击面。

```dockerfile
# 创建用户组和应用用户
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001

# 切换到非 root 用户
USER nodejs

# 此后的所有操作都在 nodejs 用户下执行
```

### 清理不必要的文件

在构建过程中会产生各种临时文件和缓存，这些不需要进入最终的镜像。养成在 RUN 指令末尾清理的好习惯：

```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        git && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

### .dockerignore 文件

`.dockerignore` 的作用和 `.gitignore` 类似，告诉 Docker 在构建镜像时忽略哪些文件。不把 node_modules、.git、测试文件、日志等不需要的东西 COPY 进镜像，可以减小镜像体积，也能避免暴露敏感信息。

```dockerignore
# 版本控制
.git
.gitignore

# 依赖（应该通过 RUN npm ci 安装，而不是 COPY）
node_modules

# 开发文件
.env
.env.*
*.log
npm-debug.log*

# IDE 配置
.vscode
.idea
*.swp

# 测试和覆盖率
coverage
*.test.js
*.spec.js
__tests__

# 构建产物（会被重新构建）
dist
build
.next
.nuxt

# 其他
README.md
docker-compose.yml
Dockerfile
```

---

## Docker Compose：多容器编排

### 为什么需要 Docker Compose

实际项目很少只用一个容器。拿一个典型的 Web 应用来说，可能有：Node.js 应用容器、PostgreSQL 数据库容器、Redis 缓存容器、Nginx 反向代理容器。手动管理这些容器的启动顺序、端口映射、网络配置会非常繁琐。

Docker Compose 就是来解决这个问题的——你写一个 YAML 配置文件，描述所有服务及其关系，然后用一条命令启动整个应用栈。

### docker-compose.yml 完整示例

下面是一个真实项目中常见的配置：Next.js 前端 + PostgreSQL 数据库 + Redis 缓存 + Nginx 反向代理，通过 Docker Compose 统一管理。

```yaml
# docker-compose.yml
version: "3.9"

services:
  # ─────────────────────────────────────────────────────────
  # Next.js Web 应用
  # ─────────────────────────────────────────────────────────
  web:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    container_name: myapp-web
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgresql://appuser:apppassword@db:5432/myapp
      REDIS_URL: redis://cache:6379/0
      NODE_ENV: production
    # 等待数据库就绪后再启动
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
      # 命名卷：上传文件持久化
      - app_uploads:/app/uploads

  # ─────────────────────────────────────────────────────────
  # PostgreSQL 数据库
  # ─────────────────────────────────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: myapp-db
    environment:
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppassword
      POSTGRES_DB: myapp
      # 性能参数
      POSTGRES_MAX_CONNECTIONS: 100
      POSTGRES_SHARED_BUFFERS: 256MB
    ports:
      - "5432:5432"
    volumes:
      # 命名卷：数据持久化到宿主机
      - postgres_data:/var/lib/postgresql/data
      # 初始化脚本
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    healthcheck:
      # 健康检查确保数据库完全就绪
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
          cpus: "1.0"
          memory: 1G

  # ─────────────────────────────────────────────────────────
  # Redis 缓存
  # ─────────────────────────────────────────────────────────
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
          cpus: "0.5"
          memory: 512M

  # ─────────────────────────────────────────────────────────
  # Nginx 反向代理
  # ─────────────────────────────────────────────────────────
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

networks:
  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16

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

### 常用命令一览

学会了配置文件之后，剩下的就是记住几条常用命令：

| 命令 | 作用 | 常用参数 |
|------|------|---------|
| `docker compose up -d` | 后台启动所有服务 | `--build` 强制重新构建 |
| `docker compose down` | 停止并删除所有容器 | `-v` 同时删除卷 |
| `docker compose ps` | 查看服务状态 | |
| `docker compose logs -f` | 实时查看日志 | `web` 只看 web 服务日志 |
| `docker compose exec` | 在容器内执行命令 | `db psql -U appuser` |
| `docker compose restart` | 重启服务 | `web` 只重启 web |
| `docker compose build` | 重新构建镜像 | `--no-cache` 不用缓存 |
| `docker compose top` | 查看各容器内进程 | |

---

## 存储与网络

### 三种存储方式的选择

Docker 容器有三种存储数据的方式，各有各的适用场景：

**命名卷（Named Volume）** 是最推荐的方式。数据存在宿主机上，由 Docker 管理，和容器本身解耦。容器被删除后，数据依然保留。适合数据库文件、上传文件等需要持久化的数据。

**绑定挂载（Bind Mount）** 把宿主机的某个目录直接映射到容器内。适合开发时需要实时看到代码改动的场景（热重载），也适合配置文件。

**tmpfs** 把数据存在内存中，容器重启后数据丢失。适合存储密码、密钥等敏感信息，追求最高性能的场景。

```yaml
services:
  app:
    volumes:
      # 命名卷：持久化存储
      - postgres_data:/var/lib/postgresql/data
      # 绑定挂载：配置文件（只读）
      - ./config/nginx.conf:/etc/nginx/nginx.conf:ro
      # tmpfs：敏感数据存内存
      - type: tmpfs
        target: /run/secrets
        tmpfs:
          size: 1024
```

### 网络模式

Docker 默认会创建一个 bridge 网络，所有容器可以通过容器名互相访问。比如在 docker-compose.yml 里，web 服务可以通过 `postgresql://db:5432` 访问数据库服务，其中的 `db` 就是数据库服务的容器名，Docker 内置的 DNS 会自动解析到对应容器的 IP。

如果需要更精细的网络控制，可以创建多个网络来实现隔离：

```yaml
services:
  frontend:
    networks:
      - frontend  # 前端能访问外网和负载均衡器
  backend:
    networks:
      - backend  # 后端只能被前端访问
  database:
    networks:
      - backend  # 数据库只能被后端访问
    internal: true  # 完全禁止外部访问

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

---

## Docker + AI 应用

### 在容器中运行 Ollama

Ollama 是运行本地大模型的工具，可以轻松容器化部署：

```yaml
# docker-compose.yml 中加入 Ollama 服务
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_models:/root/.ollama
    restart: unless-stopped
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]

volumes:
  ollama_models:
```

启动后，通过简单的 HTTP 请求就能调用本地模型：

```bash
# 拉取模型
docker compose exec ollama ollama pull llama3.2

# 调用 API
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2",
  "prompt": "用一句话解释量子计算"
}'
```

### LangChain 应用容器化

AI 应用通常需要多个组件配合工作：Python/FastAPI 后端、向量数据库（pgvector）、缓存（Redis）。以下是一个完整的 Docker Compose 配置：

```yaml
services:
  app:
    build: .
    container_name: ai-app
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      DATABASE_URL: postgresql://aiuser:aipassword@postgres:5432/ai_db
      PGVECTOR_CONNECTION_STRING: postgresql://aiuser:aipassword@postgres:5432/ai_db
      REDIS_URL: redis://redis:6379/0
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped

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
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U aiuser -d ai_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    container_name: ai-redis
    command: redis-server --maxmemory 512mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data

volumes:
  pgvector_data:
  redis_data:
```

---

## 常见问题与排查

### 容器启动失败怎么办

这是最常见的问题，通常用三个步骤排查：

```bash
# 第一步：看日志，这是最直接的信息来源
docker logs my-container
docker logs --tail 100 my-container  # 只看最后 100 行

# 第二步：检查容器状态和配置
docker inspect my-container

# 第三步：进入容器内部手动调试
docker exec -it my-container /bin/sh
```

### 端口冲突

如果启动时报 `port is already allocated`，说明宿主机端口被占用了。可以用 `docker compose ps` 查看哪个服务的端口冲突，或者改用不冲突的宿主机端口：

```yaml
# 原来
ports:
  - "3000:3000"

# 改为（容器端口不变，宿主机用其他端口）
ports:
  - "3001:3000"
```

### 磁盘空间不足

长时间运行 Docker 后，镜像、容器、卷会积累大量未使用的资源。定期清理是个好习惯：

```bash
# 清理所有未使用的资源（镜像、容器、网络，不影响正在运行的）
docker system prune

# 更彻底地清理，包括未使用的卷
docker system prune --volumes

# 清理悬空镜像（没有标签的旧镜像）
docker image prune -a
```

### 权限问题

在 macOS 或 Linux 上，从容器内创建的文件默认属于 root 用户，可能导致宿主机普通用户无法修改。解决方案是创建非 root 用户（参考前文的 Dockerfile 示例），或者在运行时指定用户：

```bash
docker run -d --user 1000:1000 myapp:latest
```

---

> [!SUCCESS]
> Docker 是现代云原生开发的基础设施核心。掌握了镜像构建、Compose 编排、多阶段构建这些核心技能，你就能在本地快速搭建复杂的开发环境，也能将应用可靠地部署到生产服务器。容器化不仅是技术选择，更是一种让开发和运维团队高效协作的工作方式。
