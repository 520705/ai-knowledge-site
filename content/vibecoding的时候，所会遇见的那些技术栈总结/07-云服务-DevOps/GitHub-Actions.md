---
date: 2026-04-24
tags:
  - GitHub Actions
  - CI/CD
  - DevOps
  - 自动化
description: GitHub Actions CI/CD 自动化完整指南，涵盖 workflow 语法、YAML 深度解析、环境变量、缓存策略、Docker 集成、安全实践。
---

# GitHub Actions：CI/CD 自动化与 GitHub 原生集成

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 GitHub Actions 核心概念、YAML 语法深度解析、矩阵构建、Secret 管理、Docker 镜像构建，以及 CI/CD 最佳实践。面向没有 CI/CD 经验的读者，从零开始理解自动化部署。

---

## GitHub Actions 是什么

### 从手动部署说起

假设你开发了一个 Web 应用，每次更新代码后都要手动登录服务器、pull 最新代码、重启服务。如果是个人项目还好，但如果是一个团队在协作，每个人都可能随时发布代码，手动部署很快就会变成一场灾难——有人覆盖了别人的改动，有人忘了重启服务，有人部署了没经过测试的代码。

GitHub Actions 就是来解决这个问题的。它是 GitHub 内置的自动化工具，可以在你 push 代码、创建 PR、合并分支、甚至定时触发时，自动执行一系列预先定义好的任务：运行测试、构建镜像、部署到服务器、发通知。

整个流程不需要你登录任何服务器，所有操作都在云端自动完成。

### 核心概念：Workflow、Job、Step

理解 GitHub Actions 只要记住三个层级：

**Workflow（工作流）** 是最顶层的概念，定义了整个自动化流程。每个 GitHub 仓库可以有一个或多个 Workflow，配置文件放在 `.github/workflows/` 目录下，以 `.yml` 或 `.yaml` 为扩展名。

**Job（作业）** 是一个 Workflow 中的独立任务单元。一个 Workflow 可以包含多个 Job，这些 Job 默认并行执行，也可以配置为串行依赖。Job 之间可以通过 artifacts 传递数据。

**Step（步骤）** 是 Job 中的具体操作步骤。每个 Step 可以是 Shell 命令，也可以是一个可复用的 Action。

```
Workflow（工作流）
├── Job 1: 测试
│   ├── Step 1: 检出代码
│   ├── Step 2: 安装依赖
│   └── Step 3: 运行测试
│
├── Job 2: 构建镜像（并行）
│   ├── Step 1: 检出代码
│   ├── Step 2: 构建 Docker 镜像
│   └── Step 3: 推送到仓库
│
└── Job 3: 部署（依赖 Job 1 和 2 完成）
    ├── Step 1: 拉取最新镜像
    └── Step 2: 重启服务
```

---

## YAML 语法深度解析

### 基础语法要点

GitHub Actions 的配置文件用 YAML 格式编写。YAML 对缩进非常敏感，通常用两个空格缩进。下面是一个最简单的 Workflow 示例：

```yaml
# workflow 文件存放在 .github/workflows/ 目录下
name: CI Pipeline

# on 定义触发条件
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# jobs 定义所有作业
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

### 触发条件（on）

触发条件决定了 Workflow 何时执行，可以非常灵活：

```yaml
on:
  # 在 main 分支 push 时触发
  push:
    branches:
      - main
      - develop
    # 只监听特定目录的变化（节省 CI 时间）
    paths:
      - "src/**"
      - "package.json"

  # 在 PR 创建或更新时触发
  pull_request:
    branches: [main]
    types: [opened, synchronize, reopened]

  # 定时触发（cron 格式，UTC 时间）
  schedule:
    - cron: "0 2 * * *"  # 每天凌晨 2 点

  # 手动触发（在 GitHub 网页上手动运行）
  workflow_dispatch:
    inputs:
      environment:
        description: "部署环境"
        required: true
        default: staging
        type: choice
        options:
          - staging
          - production

  # 标签创建时触发
  create:

  # 发布时触发
  release:
    types: [published]

  # 外部事件触发（repository_dispatch）
  repository_dispatch:
    types: [deploy]
```

### 环境变量与 Secret

环境变量在 CI/CD 中非常常用，可以用来存储配置信息、API 密钥、数据库连接字符串等。

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      # 全局环境变量，所有 steps 都可用
      NODE_ENV: test
      DATABASE_NAME: test_db
    steps:
      - name: Print environment
        run: echo ${{ env.DATABASE_NAME }}

      - name: Build with env
        run: npm run build
        env:
          # 这个变量只在当前 step 可见
          API_URL: https://api.test.example.com
```

Secret 是用来存储敏感信息的安全方式，不会出现在日志输出中：

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # 需要在 GitHub 仓库设置中配置环境
    steps:
      - name: Deploy to server
        env:
          # 从 GitHub Secrets 读取，不会在日志中暴露
          SSH_KEY: ${{ secrets.SERVER_SSH_KEY }}
          DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}
        run: |
          echo "$SSH_KEY" > deploy_key
          chmod 600 deploy_key
          ssh -i deploy_key user@server "./deploy.sh"
```

### 条件执行与矩阵构建

可以用 `if` 表达式控制 step 是否执行：

```yaml
steps:
  - name: Build and push
    if: github.ref == 'refs/heads/main'
    run: |
      docker build -t myapp:latest .
      docker push myapp:latest

  - name: Notify on failure
    if: failure()
    run: echo "Pipeline failed!"
```

矩阵构建（Matrix）是 GitHub Actions 最强大的特性之一，允许你用一套配置并行测试多个环境组合：

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [18, 20, 22]
        operating-system: [ubuntu-latest, windows-latest]
        # 也可以用 include 添加特定组合
        # exclude 排除特定组合
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}

      - name: Install dependencies
        run: npm ci

      - name: Run tests on ${{ matrix.operating-system }}
        run: npm test
```

---

## 完整 CI/CD Pipeline 示例

### Node.js 项目的完整流水线

下面是一个真实项目中常见的完整 CI/CD 流水线，包括代码检出、依赖安装、类型检查、代码检查、测试、构建 Docker 镜像、安全扫描、推送到镜像仓库：

```yaml
name: Node.js CI/CD Pipeline

on:
  push:
    branches: [main]
    tags: ["v*"]  # 打标签时触发
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─────────────────────────────────────────────────────────
  # Job 1: 代码质量检查（并行执行）
  # ─────────────────────────────────────────────────────────
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Check types
        run: npm run typecheck

  # ─────────────────────────────────────────────────────────
  # Job 2: 单元测试与集成测试
  # ─────────────────────────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    services:
      # 测试时启动数据库服务
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

      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run tests with coverage
        run: npm test -- --coverage
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
          REDIS_URL: redis://localhost:6379/0

      - name: Upload coverage report
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: false

  # ─────────────────────────────────────────────────────────
  # Job 3: 构建并推送 Docker 镜像
  # ─────────────────────────────────────────────────────────
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]  # 依赖前面两个 job
    permissions:
      contents: read
      packages: write  # 写入 Packages

    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,prefix=

      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # ─────────────────────────────────────────────────────────
  # Job 4: 安全扫描
  # ─────────────────────────────────────────────────────────
  security:
    runs-on: ubuntu-latest
    needs: build
    permissions:
      security-events: write
      contents: read

    steps:
      - uses: actions/checkout@v4

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.ref == 'refs/heads/main' && 'latest' || github.sha }}
          format: "sarif"
          output: "trivy-results.sarif"
          severity: "CRITICAL,HIGH"
          exit-code: "1"

      - name: Upload Trivy results to GitHub Security
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: "trivy-results.sarif"

  # ─────────────────────────────────────────────────────────
  # Job 5: 部署到服务器
  # ─────────────────────────────────────────────────────────
  deploy:
    runs-on: ubuntu-latest
    needs: [build, security]
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Deploy to server via SSH
        uses: appleboy/ssh-action@v1
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

### 缓存策略

依赖安装是 CI 中最耗时的步骤之一，合理的缓存策略可以大幅加速构建。GitHub Actions 提供了专门的缓存 action：

```yaml
steps:
  # npm 缓存
  - name: Setup Node.js
    uses: actions/setup-node@v4
    with:
      node-version: "20"
      cache: "npm"  # 自动缓存 node_modules

  # pip 缓存
  - name: Setup Python
    uses: actions/setup-python@v5
    with:
      python-version: "3.12"
      cache: "pip"

  # 使用 actions/cache 手动缓存
  - name: Cache Docker layers
    uses: actions/cache@v4
    with:
      path: /tmp/.buildx-cache
      key: ${{ runner.os }}-buildx-${{ github.sha }}
      restore-keys: |
        ${{ runner.os }}-buildx-

  # Cargo/Rust 缓存
  - name: Cache Rust dependencies
    uses: Swatinem/rust-cache@v2
    with:
      workspaces: "./src -> target"
```

### 矩阵构建的高级用法

矩阵构建不只是简单的版本组合，还可以做更复杂的配置：

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false  # 一个失败不影响其他的
      matrix:
        include:
          # 添加自定义配置的组合
          - node-version: "20"
            coverage: true
            elasticsearch: "7.17.0"
        exclude:
          # 排除不兼容的组合
          - node-version: "18"
            elasticsearch: "8.0.0"

    steps:
      - uses: actions/checkout@v4

      - name: Setup Elasticsearch
        if: matrix.elasticsearch
        uses: elastic/elastic-github-actions/elasticsearch@main
        with:
          stack-version: ${{ matrix.elasticsearch }}
```

---

## GitHub Actions 与 Docker 集成

### 构建并推送到多种镜像仓库

现代项目通常需要将镜像推送到多个仓库以适应不同环境：

```yaml
jobs:
  build-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata for Docker Hub
        id: meta-dockerhub
        uses: docker/metadata-action@v5
        with:
          images: ${{ secrets.DOCKERHUB_USERNAME }}/myapp
          tags: |
            type=semver,pattern={{version}}
            type=sha
            type=raw,value=latest,enable=${{ github.ref == 'refs/heads/main' }}

      - name: Build and push to Docker Hub
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta-dockerhub.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

      - name: Build and push to GHCR
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ghcr.io/${{ github.repository }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### 多架构镜像构建

为了兼容不同架构的设备（x86 和 ARM），可以构建多架构镜像：

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build and push multi-platform image
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64,linux/arm/v7
          push: true
          tags: |
            myusername/myapp:latest
            myusername/myapp:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## 复用与模块化

### 使用市场 Action

GitHub Actions 最大的优势之一是有丰富的社区生态，大量常用操作已经被封装成了可复用的 Action。以下是一些最常用的：

| Action | 用途 | 来源 |
|--------|------|------|
| `actions/checkout@v4` | 检出代码 | 官方 |
| `actions/setup-node@v4` | 安装 Node.js | 官方 |
| `actions/setup-python@v5` | 安装 Python | 官方 |
| `docker/login-action@v3` | Docker 登录 | Docker 官方 |
| `docker/build-push-action@v5` | 构建并推送镜像 | Docker 官方 |
| `appleboy/ssh-action@v1` | SSH 远程执行 | 社区 |
| `peaceiris/actions-gh-pages@v4` | 部署静态页面 | 社区 |

### 创建可复用 Workflow

如果多个项目需要相同的 CI/CD 流程，可以创建可复用 Workflow：

```yaml
# .github/workflows/reusable-workflow.yml
name: Reusable Docker Deploy

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
      image-tag:
        required: true
        type: string
    secrets:
      SSH_HOST:
        required: true
      SSH_KEY:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SSH_HOST }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull myapp:${{ inputs.image-tag }}
            docker stop myapp || true
            docker run -d --name myapp myapp:${{ inputs.image-tag }}
```

然后在其他 Workflow 中调用：

```yaml
# .github/workflows/release.yml
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      tag: ${{ steps.meta.outputs.tags }}
    steps:
      # ... 构建步骤 ...

  deploy:
    needs: build
    uses: ./.github/workflows/reusable-workflow.yml
    with:
      environment: production
      image-tag: latest
    secrets:
      SSH_HOST: ${{ secrets.SSH_HOST }}
      SSH_KEY: ${{ secrets.SSH_KEY }}
```

---

## 安全最佳实践

### Secret 管理

敏感信息绝对不能硬编码在 Workflow 文件里，必须使用 GitHub Secrets。以下是配置和管理 Secret 的方法：

1. 在 GitHub 仓库页面，进入 Settings → Secrets and variables → Actions
2. 点击 "New repository secret" 添加新的密钥
3. 在 Workflow 中通过 `${{ secrets.SECRET_NAME }}` 引用

```yaml
steps:
  - name: Deploy
    env:
      # 使用 Secret，值不会出现在日志中
      API_KEY: ${{ secrets.API_KEY }}
      DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
    run: ./deploy.sh
```

### 最小权限原则

每个 Job 应该有最小必要的权限，避免过度授权：

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read    # 只需要读取代码
      packages: write   # 需要写入镜像

  deploy:
    runs-on: ubuntu-latest
    environment: production
    permissions: {}     # 继承仓库级别权限

  security:
    runs-on: ubuntu-latest
    permissions:
      security-events: write  # 只写安全事件
```

### 环境保护规则

为生产环境设置保护规则，防止未经审批的部署：

```yaml
# 在 workflow 中指定环境
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
```

然后在仓库 Settings → Environments 中配置：需要审批人、必须等待一定时间后才能部署、禁止手动删除等。

### 避免注入攻击

谨慎处理来自 GitHub 上下文的变量，防止注入攻击：

```yaml
# 不安全的做法：直接使用变量作为命令的一部分
- name: Run command
  run: echo ${{ github.event.inputs.name }} | bash

# 更安全的做法：使用环境变量传递参数
- name: Run command
  env:
    INPUT_NAME: ${{ github.event.inputs.name }}
  run: echo "$INPUT_NAME" | bash
```

---

## GitHub Actions vs 其他 CI/CD 平台

| 维度 | GitHub Actions | GitLab CI | Jenkins |
|------|---------------|-----------|---------|
| 集成度 | 与 GitHub 无缝集成 | 与 GitLab 无缝集成 | 需要独立部署 |
| 配置文件 | YAML，在仓库内管理 | YAML，在仓库内管理 | Jenkinsfile 或 Web UI |
| 托管方式 | 全托管，无需服务器 | 可自建或用 GitLab.com | 完全自建 |
| 免费额度 | 2000 分钟/月（免费账户） | 2000 分钟/月 | 无限制（自建） |
| 市场生态 | 丰富的 Action 市场 | 丰富的 CI 模板 | 丰富的插件生态 |
| 学习曲线 | 低，概念简单 | 中等 | 高，配置复杂 |

对于已经在使用 GitHub 的团队来说，GitHub Actions 是最自然的选择，因为它不需要额外的账户和基础设施，直接在仓库设置里就能完成所有配置。

---

> [!SUCCESS]
> GitHub Actions 是现代软件开发中不可或缺的一环。从自动运行测试到 Docker 镜像构建和部署，它把原本需要人工操作的重复工作全部自动化了。掌握了 Workflow 编写、矩阵构建、Secret 管理和复用 Workflow 这些核心技能，你就拥有了快速、可靠、可重复的软件交付能力。
