# GitHub Actions：CI/CD 自动化与 GitHub 原生集成

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 GitHub Actions 核心概念、YAML 语法、常用 Action、Matrix 构建、Secret 管理，以及 AI 应用 CI/CD 实战指南。

---

## 目录

1. [[#GitHub Actions 核心概念]]
2. [[#Workflow 语法详解]]
3. [[#常用 Action 列表]]
4. [[#Matrix 构建]]
5. [[#Secret 管理与环境变量]]
6. [[#CI/CD 最佳实践]]
7. [[#GitHub Actions vs GitLab CI vs Jenkins 对比]]
8. [[#实战：Node.js + Docker + Vercel CI/CD]]

---

## GitHub Actions 核心概念

### 核心概念层级

GitHub Actions 是 GitHub 提供的持续集成和持续部署（CI/CD）平台，通过 YAML 配置文件定义自动化工作流。

```
┌─────────────────────────────────────────────────────────────────────┐
│                    GitHub Actions 概念层级                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Workflow（工作流）                                                  │
│  ├── 最顶层定义，整个自动化流程                                        │
│  └── .github/workflows/*.yml                                        │
│         │                                                           │
│         └── Job（作业）                                              │
│             ├── 一个 Workflow 包含多个 Job                            │
│             └── Job 之间可并行或串行执行                               │
│                   │                                                 │
│                   └── Step（步骤）                                   │
│                       ├── 每个 Job 包含多个 Step                     │
│                       └── Step 是最小执行单元                        │
│                             │                                        │
│                             └── Action（动作）                        │
│                                 ├── 复用的工作单元                    │
│                                 └── marketplace 上的可复用组件         │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 触发条件

| 触发器 | 说明 | 示例 |
|--------|------|------|
| `push` | 推送到仓库 | 分支、标签、路径过滤 |
| `pull_request` | PR 活动 | opened, closed, synchronize |
| `schedule` | 定时任务 | cron 表达式 |
| `workflow_dispatch` | 手动触发 | inputs 参数 |
| `repository_dispatch` | API 触发 | 自定义事件类型 |
| `release` | 发布活动 | published, created |
| `issue_comment` | Issue 评论 | created, edited |
| `deployment` | 部署活动 | 环境信息 |

---

## Workflow 语法详解

### 完整 Workflow 示例

```yaml
# .github/workflows/ci.yml
# ─────────────────────────────────────────────────────────────
# 完整的 CI/CD Workflow
# ─────────────────────────────────────────────────────────────

name: CI/CD Pipeline

# 触发条件
on:
    # 主分支推送和 PR
    push:
        branches: [main, develop]
        paths-ignore:
            - '*.md'
            - 'docs/**'
    pull_request:
        branches: [main]
        types: [opened, synchronize, reopened]
    # 定时任务（每周一凌晨）
    schedule:
        - cron: '0 2 * * 1'
    # 手动触发
    workflow_dispatch:
        inputs:
            environment:
                description: 'Deploy environment'
                required: true
                default: 'staging'
                type: choice
                options:
                    - staging
                    - production

# 环境变量
env:
    NODE_VERSION: '20'
    REGISTRY: ghcr.io
    IMAGE_NAME: ${{ github.repository }}

# 作业定义
jobs:
    # ─────────────────────────────────────────────────────
    # 作业 1：代码检查
    # ─────────────────────────────────────────────────────
    lint:
        name: Lint & Type Check
        runs-on: ubuntu-latest
        
        steps:
            - name: Checkout code
              uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ env.NODE_VERSION }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Run ESLint
              run: npm run lint
            
            - name: Type check
              run: npm run typecheck
            
            - name: Lint with GitHub Actions annotations
              uses: reviewdog/action-eslint@v1
              env:
                  REVIEWDOG_GITHUB_API_TOKEN: ${{ secrets.GITHUB_TOKEN }}

    # ─────────────────────────────────────────────────────
    # 作业 2：单元测试
    # ─────────────────────────────────────────────────────
    test:
        name: Unit Tests
        runs-on: ubuntu-latest
        needs: lint  # 依赖 lint 作业
        
        services:
            # PostgreSQL 服务容器
            postgres:
                image: postgres:16-alpine
                env:
                    POSTGRES_USER: test
                    POSTGRES_PASSWORD: test
                    POSTGRES_DB: test
                ports:
                    - 5432:5432
                options: >-
                    --health-cmd pg_isready
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5
            
            # Redis 服务容器
            redis:
                image: redis:7-alpine
                ports:
                    - 6379:6379
                options: >-
                    --health-cmd "redis-cli ping"
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5
        
        steps:
            - name: Checkout code
              uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ env.NODE_VERSION }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Generate Prisma Client
              run: npx prisma generate
            
            - name: Run tests
              run: npm run test:coverage
              env:
                  DATABASE_URL: postgresql://test:test@localhost:5432/test
                  REDIS_URL: redis://localhost:6379
            
            - name: Upload coverage
              uses: actions/upload-artifact@v4
              with:
                  name: coverage-report
                  path: coverage/
                  retention-days: 7
            
            - name: Comment coverage
              uses: romeovs/lcov-reporter@v0.3.0
              if: github.event_name == 'pull_request'
              with:
                  lcov-file: coverage/lcov.info
                  github-token: ${{ secrets.GITHUB_TOKEN }}

    # ─────────────────────────────────────────────────────
    # 作业 3：构建 Docker 镜像
    # ─────────────────────────────────────────────────────
    build:
        name: Build Docker Image
        runs-on: ubuntu-latest
        needs: test  # 依赖 test 作业
        if: github.event_name == 'push'
        
        outputs:
            image-tag: ${{ steps.meta.outputs.tags }}
            sha-tag: ${{ env.IMAGE_TAG }}
        
        steps:
            - name: Checkout code
              uses: actions/checkout@v4
            
            - name: Setup Docker Buildx
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
              uses: docker/build-push-action@v6
              with:
                  context: .
                  push: ${{ github.ref == 'refs/heads/main' }}
                  tags: ${{ steps.meta.outputs.tags }}
                  cache-from: type=gha
                  cache-to: type=gha,mode=max

    # ─────────────────────────────────────────────────────
    # 作业 4：部署到 Staging
    # ─────────────────────────────────────────────────────
    deploy-staging:
        name: Deploy to Staging
        runs-on: ubuntu-latest
        needs: build
        if: github.ref == 'refs/heads/develop' || github.event.inputs.environment == 'staging'
        
        environment:
            name: staging
            url: https://staging.example.com
        
        steps:
            - name: Deploy to server
              uses: appleboy/ssh-action@v1
              with:
                  host: ${{ secrets.STAGING_HOST }}
                  username: ${{ secrets.STAGING_USER }}
                  key: ${{ secrets.STAGING_SSH_KEY }}
                  script: |
                      cd /app
                      docker pull ${{ needs.build.outputs.image-tag }}
                      docker-compose -f docker-compose.staging.yml up -d
                      docker image prune -f

    # ─────────────────────────────────────────────────────
    # 作业 5：部署到 Production
    # ─────────────────────────────────────────────────────
    deploy-production:
        name: Deploy to Production
        runs-on: ubuntu-latest
        needs: [build, deploy-staging]
        if: github.ref == 'refs/heads/main' && github.event.inputs.environment != 'staging'
        
        environment:
            name: production
            url: https://example.com
        
        steps:
            - name: Deploy to production
              uses: appleboy/ssh-action@v1
              with:
                  host: ${{ secrets.PROD_HOST }}
                  username: ${{ secrets.PROD_USER }}
                  key: ${{ secrets.PROD_SSH_KEY }}
                  script: |
                      cd /app
                      docker pull ${{ needs.build.outputs.image-tag }}
                      docker-compose -f docker-compose.prod.yml up -d
                      ./scripts/health-check.sh
```

---

## 常用 Action 列表

### 核心 Action

| Action | 版本 | 说明 |
|--------|------|------|
| `actions/checkout` | v4 | 检出代码仓库 |
| `actions/setup-node` | v4 | 配置 Node.js 环境 |
| `actions/setup-python` | v5 | 配置 Python 环境 |
| `actions/setup-java` | v4 | 配置 Java 环境 |
| `actions/setup-go` | v5 | 配置 Go 环境 |
| `actions/cache` | v4 | 依赖缓存 |
| `actions/upload-artifact` | v4 | 上传构建产物 |
| `actions/download-artifact` | v4 | 下载构建产物 |

### Docker Action

| Action | 版本 | 说明 |
|--------|------|------|
| `docker/login-action` | v3 | 登录 Docker Registry |
| `docker/setup-buildx-action` | v3 | 设置 Docker Buildx |
| `docker/build-push-action` | v6 | 构建并推送镜像 |
| `docker/metadata-action` | v5 | 提取镜像元数据 |

### 部署 Action

| Action | 版本 | 说明 |
|--------|------|------|
| `appleboy/ssh-action` | v1 | SSH 远程部署 |
| `JamesIves/github-pages-deploy-action` | v4 | 部署到 GitHub Pages |
| `nwtgck/actions-netlify` | v3 | 部署到 Netlify |
| `nicksemionenko/cdeploy` | v1 | 部署到 Vercel |

### 代码质量 Action

| Action | 版本 | 说明 |
|--------|------|------|
| `reviewdog/action-eslint` | v1 | ESLint 检查 |
| `reviewdog/action-golangci-lint` | v3 | Go Lint 检查 |
| `reviewdog/action-tsc` | v1 | TypeScript 检查 |
| `codecov/codecov-action` | v4 | 上传测试覆盖率 |

---

## Matrix 构建

### Matrix 基础用法

```yaml
# Matrix 构建示例
jobs:
    test:
        runs-on: ubuntu-latest
        
        strategy:
            fail-fast: false  # 矩阵中一个失败不影响其他
            matrix:
                node-version: [18, 20, 22]
                # 也可以组合多个维度
                include:
                    - node-version: 22
                      operating-system: ubuntu-latest
                      feature-flags: 'new-engine'
```

### 完整 Matrix 示例

```yaml
# .github/workflows/matrix-test.yml
jobs:
    # ─────────────────────────────────────────────────────
    # 跨版本测试矩阵
    # ─────────────────────────────────────────────────────
    test-node:
        name: Test on Node.js ${{ matrix.node-version }}
        runs-on: ubuntu-latest
        
        strategy:
            fail-fast: false
            matrix:
                node-version: [18, 20, 22]
                database: [postgres:15, postgres:16]
        
        services:
            postgres:
                image: ${{ matrix.database }}
                env:
                    POSTGRES_USER: test
                    POSTGRES_PASSWORD: test
                    POSTGRES_DB: test
                ports:
                    - 5432:5432
                options: >-
                    --health-cmd pg_isready
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Node.js ${{ matrix.node-version }}
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ matrix.node-version }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Run tests
              run: npm test
              env:
                  DATABASE_URL: postgresql://test:test@localhost:5432/test

    # ─────────────────────────────────────────────────────
    # 跨平台构建矩阵
    # ─────────────────────────────────────────────────────
    build-multiplatform:
        name: Build for ${{ matrix.os }}
        runs-on: ${{ matrix.os }}
        strategy:
            fail-fast: false
            matrix:
                os: [ubuntu-latest, windows-latest, macos-latest]
                node-version: [18, 20]
                exclude:
                    # 排除无效组合
                    - os: windows-latest
                      node-version: 18
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ matrix.node-version }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Build
              run: npm run build
            
            - name: Upload artifact
              uses: actions/upload-artifact@v4
              with:
                  name: build-${{ matrix.os }}-node${{ matrix.node-version }}
                  path: dist/
```

---

## Secret 管理和环境变量

### Secret 类型

| 类型 | 作用域 | 说明 |
|------|--------|------|
| `secrets` | 仓库 | 仓库级别的密钥 |
| `env` | Workflow | Workflow 环境变量 |
| `vars` | 仓库 | 非敏感配置变量 |
| `secrets` | 组织 | 组织级别的密钥 |
| `env` | 组织 | 组织环境变量 |
| `secrets` | 环境 | 特定环境的密钥 |

### Secret 使用

```yaml
# .github/workflows/secrets-demo.yml
name: Secrets Demo

on:
    workflow_dispatch:

jobs:
    demo:
        runs-on: ubuntu-latest
        environment: production  # 指定环境
        
        env:
            # 非敏感环境变量
            APP_NAME: my-app
            LOG_LEVEL: info
        
        steps:
            - name: Use secrets
              run: |
                  echo "Using DATABASE_URL: ${{ secrets.DATABASE_URL }}"
                  echo "Using API_KEY: ${{ secrets.API_KEY }}"
            
            - name: Multi-line secret
              run: |
                  echo "${{ secrets.SSH_PRIVATE_KEY }}" > key.pem
                  chmod 600 key.pem
            
            - name: Organization secret
              run: |
                  echo "Using org secret: ${{ secrets.ORG_SECRET }}"
```

### 环境保护规则

```yaml
# 在仓库 Settings → Environments 配置

# workflow 中引用
jobs:
    deploy:
        runs-on: ubuntu-latest
        environment:
            name: production
            url: https://example.com
        # 或者带规则的环境
        environment:
            name: staging
            protection_rules:
                - id: require-reviewers
                  required_reviewers: 1
                - id: wait-timer
                  wait_timer: 30
```

### 变量（Vars）

```yaml
# vars 与 secrets 的区别
# vars 适合非敏感配置

# 仓库 Settings → Variables 中配置
# COMMON_NODE_VERSION = 20
# DEPLOY_REGION = us-east-1

jobs:
    demo:
        runs-on: ubuntu-latest
        steps:
            - name: Use variables
              run: |
                  echo "Node version: ${{ vars.COMMON_NODE_VERSION }}"
                  echo "Region: ${{ vars.DEPLOY_REGION }}"
```

---

## CI/CD 最佳实践

### 依赖缓存

```yaml
# ─────────────────────────────────────────────────────────────
# 依赖缓存最佳实践
# ─────────────────────────────────────────────────────────────

jobs:
    cached-build:
        runs-on: ubuntu-latest
        
        steps:
            - name: Checkout
              uses: actions/checkout@v4
            
            # Node.js 缓存
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: '20'
                  cache: 'npm'  # 自动缓存 node_modules
            
            # Python 缓存
            - name: Setup Python
              uses: actions/setup-python@v5
              with:
                  python-version: '3.12'
                  cache: 'pip'
            
            # Go 缓存
            - name: Setup Go
              uses: actions/setup-go@v5
              with:
                  go-version: '1.23'
                  cache: true
            
            # Rust 缓存
            - name: Cache Rust
              uses: Swatinem/rust-cache@v2
              with:
                  workspaces: './src -> target'
            
            # Maven 缓存
            - name: Cache Maven
              uses: actions/cache@v4
              with:
                  path: ~/.m2/repository
                  key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
                  restore-keys: |
                      ${{ runner.os }}-maven-
            
            - name: Install
              run: npm ci
```

### 分段构建

```yaml
# ─────────────────────────────────────────────────────────────
# 分段构建策略
# ─────────────────────────────────────────────────────────────

jobs:
    # 阶段 1：代码质量（最快）
    quality:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: Lint
              run: npm run lint
            - name: Type check
              run: npm run typecheck
    
    # 阶段 2：测试（中等速度）
    test:
        runs-on: ubuntu-latest
        needs: quality  # 依赖 quality
        services:
            postgres:
                image: postgres:16
                # ...
        steps:
            - uses: actions/checkout@v4
            - name: Test
              run: npm test
              env:
                  DATABASE_URL: postgresql://...
    
    # 阶段 3：构建（较慢）
    build:
        runs-on: ubuntu-latest
        needs: test  # 依赖 test
        steps:
            - uses: actions/checkout@v4
            - name: Build
              run: npm run build
            
            - name: Upload artifact
              uses: actions/upload-artifact@v4
              with:
                  name: build-output
                  path: dist/
    
    # 阶段 4：部署（最慢，可能失败）
    deploy:
        runs-on: ubuntu-latest
        needs: build  # 依赖 build
        if: github.ref == 'refs/heads/main'
        steps:
            - uses: actions/checkout@v4
            - name: Download artifact
              uses: actions/download-artifact@v4
              with:
                  name: build-output
            - name: Deploy
              run: npm run deploy
```

### 并行执行优化

```yaml
# ─────────────────────────────────────────────────────────────
# 并行执行独立作业
# ─────────────────────────────────────────────────────────────

jobs:
    # 三个独立测试可以并行执行
    unit-tests:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: Unit tests
              run: npm run test:unit
    
    integration-tests:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: Integration tests
              run: npm run test:integration
    
    e2e-tests:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            - name: E2E tests
              run: npm run test:e2e
    
    # 汇总报告
    report:
        runs-on: ubuntu-latest
        needs: [unit-tests, integration-tests, e2e-tests]
        steps:
            - name: Generate report
              run: echo "All tests completed"
```

---

## GitHub Actions vs GitLab CI vs Jenkins 对比

### 核心对比表

| 维度 | GitHub Actions | GitLab CI/CD | Jenkins |
|------|----------------|--------------|---------|
| **集成度** | GitHub 原生 | GitLab 原生 | 独立部署 |
| **配置格式** | YAML | YAML | Declarative/脚本 |
| **托管方式** | 云端/自托管 | 云端/自托管 | 仅自托管 |
| **免费额度** | 2000 分钟/月（免费版） | 400 分钟/月（免费版） | 无限（自托管） |
| **并行执行** | ✓（最多 256 个 job） | ✓（付费版） | ✓ |
| **矩阵构建** | ✓ | ✓ | ✓ |
| **自托管运行器** | ✓ | ✓ | ✓ |
| **容器支持** | ✓ | ✓ | ✓ |
| **Secret 管理** | 内置 | 内置 | 插件 |
| **市场 Action** | 15000+ | 700+ | 1800+ |
| **学习曲线** | 低 | 中 | 高 |

### GitLab CI 示例对比

```yaml
# .gitlab-ci.yml - GitLab CI 配置
stages:
    - lint
    - test
    - build
    - deploy

lint:
    stage: lint
    image: node:20-alpine
    script:
        - npm ci
        - npm run lint

test:
    stage: test
    image: node:20-alpine
    services:
        - postgres:16-alpine
    script:
        - npm ci
        - npm test
    variables:
        DATABASE_URL: postgresql://postgres:postgres@postgres:5432/test

build:
    stage: build
    image: docker:latest
    services:
        - docker:dind
    script:
        - docker build -t my-app:$CI_COMMIT_SHA .
        - docker push my-app:$CI_COMMIT_SHA

deploy:
    stage: deploy
    script:
        - echo "Deploying..."
    only:
        - main
```

### Jenkins Declarative Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        NODE_VERSION = '20'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Lint') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './scripts/deploy.sh'
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

## 实战：Node.js + Docker + Vercel CI/CD

### 完整 Workflow

```yaml
# .github/workflows/full-ci-cd.yml
name: Full CI/CD Pipeline

on:
    push:
        branches: [main, develop]
    pull_request:
        branches: [main]
    workflow_dispatch:

env:
    NODE_VERSION: '20'
    REGISTRY: ghcr.io
    IMAGE_NAME: ${{ github.repository }}

jobs:
    # ─────────────────────────────────────────────────────
    # 阶段 1：质量检查
    # ─────────────────────────────────────────────────────
    quality:
        name: Code Quality
        runs-on: ubuntu-latest
        timeout-minutes: 15
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ env.NODE_VERSION }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Lint
              run: npm run lint
            
            - name: Type check
              run: npm run typecheck
            
            - name: Check formatting
              run: npm run format:check

    # ─────────────────────────────────────────────────────
    # 阶段 2：测试
    # ─────────────────────────────────────────────────────
    test:
        name: Test Suite
        runs-on: ubuntu-latest
        needs: quality
        timeout-minutes: 30
        
        services:
            postgres:
                image: postgres:16-alpine
                env:
                    POSTGRES_USER: test
                    POSTGRES_PASSWORD: test
                    POSTGRES_DB: test
                ports:
                    - 5432:5432
                options: >-
                    --health-cmd pg_isready
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5
            
            redis:
                image: redis:7-alpine
                ports:
                    - 6379:6379
                options: >-
                    --health-cmd "redis-cli ping"
                    --health-interval 10s
                    --health-timeout 5s
                    --health-retries 5
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: ${{ env.NODE_VERSION }}
                  cache: 'npm'
            
            - name: Install dependencies
              run: npm ci
            
            - name: Generate Prisma Client
              run: npx prisma generate
            
            - name: Run tests
              run: npm run test:coverage
              env:
                  DATABASE_URL: postgresql://test:test@localhost:5432/test
                  REDIS_URL: redis://localhost:6379/0
            
            - name: Upload coverage
              uses: actions/upload-artifact@v4
              with:
                  name: coverage
                  path: coverage/
                  retention-days: 14

    # ─────────────────────────────────────────────────────
    # 阶段 3：构建 Docker 镜像
    # ─────────────────────────────────────────────────────
    docker:
        name: Build Docker Image
        runs-on: ubuntu-latest
        needs: test
        if: github.event_name == 'push'
        timeout-minutes: 20
        
        outputs:
            tags: ${{ steps.meta.outputs.tags }}
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Docker Buildx
              uses: docker/setup-buildx-action@v3
            
            - name: Login to Container Registry
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
                      type=sha
            
            - name: Build and push
              uses: docker/build-push-action@v6
              with:
                  context: .
                  push: ${{ github.ref == 'refs/heads/main' }}
                  tags: ${{ steps.meta.outputs.tags }}
                  cache-from: type=gha
                  cache-to: type=gha,mode=max

    # ─────────────────────────────────────────────────────
    # 阶段 4：部署到 Vercel
    # ─────────────────────────────────────────────────────
    deploy-vercel:
        name: Deploy to Vercel
        runs-on: ubuntu-latest
        needs: test
        if: github.ref == 'refs/heads/main'
        timeout-minutes: 10
        
        steps:
            - uses: actions/checkout@v4
            
            - name: Deploy to Vercel
              uses: nicksemionenko/cdeploy@v1
              with:
                  token: ${{ secrets.VERCEL_TOKEN }}
                  org-id: ${{ secrets.VERCEL_ORG_ID }}
                  project-id: ${{ secrets.VERCEL_PROJECT_ID }}
                  prod: true  # 部署到生产环境

    # ─────────────────────────────────────────────────────
    # 阶段 5：通知
    # ─────────────────────────────────────────────────────
    notify:
        name: Notify
        runs-on: ubuntu-latest
        needs: [deploy-vercel]
        if: always()
        steps:
            - name: Send Slack notification
              if: needs.deploy-vercel.result == 'success'
              uses: slackapi/slack-github-action@v1
              with:
                  payload: |
                      {
                        "text": "✅ Deployment successful!",
                        "blocks": [
                          {
                            "type": "section",
                            "text": {
                              "type": "mrkdwn",
                              "text": "*Deployment Successful* :white_check_mark:"
                            }
                          },
                          {
                            "type": "context",
                            "elements": [
                              {
                                "type": "mrkdwn",
                                "text": "Repository: ${{ github.repository }}"
                              },
                              {
                                "type": "mrkdwn",
                                "text": "Commit: ${{ github.sha }}"
                              }
                            ]
                          }
                        ]
                      }
              env:
                  SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
                  SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
            
            - name: Send failure notification
              if: needs.deploy-vercel.result == 'failure'
              uses: slackapi/slack-github-action@v1
              with:
                  payload: |
                      {
                        "text": "❌ Deployment failed!",
                        "blocks": [
                          {
                            "type": "section",
                            "text": {
                              "type": "mrkdwn",
                              "text": "*Deployment Failed* :x:"
                            }
                          }
                        ]
                      }
              env:
                  SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
                  SLACK_WEBHOOK_TYPE: INCOMING_WEBHOOK
```

### AI 代码审查 Action

```yaml
# .github/workflows/ai-review.yml
name: AI Code Review

on:
    pull_request:
        types: [opened, synchronize]
    issue_comment:
        types: [created]

jobs:
    ai-review:
        name: AI Code Review
        runs-on: ubuntu-latest
        if: github.event_name == 'pull_request' || 
            (github.event_name == 'issue_comment' && 
             contains(github.event.comment.body, '/review'))
        timeout-minutes: 10
        
        steps:
            - uses: actions/checkout@v4
              with:
                  fetch-depth: 0  # 获取完整提交历史
            
            - name: Get PR diff
              id: pr
              uses: actions/github-script@v7
              with:
                  script: |
                      const { data: pr } = await github.rest.pulls.get({
                        owner: context.repo.owner,
                        repo: context.repo.repo,
                        pull_number: context.payload.pull_request.number,
                      });
                      return pr;
            
            - name: AI Review
              run: |
                  # 使用 Claude API 进行代码审查
                  curl -X POST "https://api.anthropic.com/v1/messages" \
                    -H "x-api-key: ${{ secrets.ANTHROPIC_API_KEY }}" \
                    -H "anthropic-version: 2023-06-01" \
                    -H "content-type: application/json" \
                    -d '{
                      "model": "claude-sonnet-4-20250514",
                      "max_tokens": 1024,
                      "messages": [
                        {
                          "role": "user",
                          "content": "你是一个代码审查专家。请审查以下 PR 的变更：\n\n'${PR_BODY}'\n\n请提供建设性的反馈，关注：\n1. 代码质量和最佳实践\n2. 潜在 bug 或安全问题\n3. 性能考虑\n4. 可维护性问题"
                        }
                      ]
                    }'
              env:
                  PR_BODY: ${{ steps.pr.outputs.body }}
```

---

# GitHub Actions：CI/CD 自动化与 GitHub 原生集成

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 GitHub Actions 核心概念、YAML 语法、常用 Action、Matrix 构建、Secret 管理，以及 AI 应用 CI/CD 实战指南。

---

## 服务概述与定位

### GitHub Actions 产品定位

GitHub Actions 是 GitHub 于 2019 年推出的自动化工作流平台，经过多年的迭代发展，已成为全球最受欢迎的 CI/CD 解决方案之一。截至 2026 年，GitHub 平台托管了超过 5 亿个仓库，其中超过 60% 的仓库使用 GitHub Actions 进行自动化构建和部署。

GitHub Actions 的核心价值主张在于其与 GitHub 生态系统的深度整合。开发者无需离开 GitHub 界面即可完成从代码提交、测试、构建到部署的完整流程。同时，GitHub Marketplace 提供了超过 15,000 个社区贡献的 Actions，覆盖了从代码检查、云服务部署到安全扫描的方方面面。

#### 与竞品的核心差异

| 维度 | GitHub Actions | GitLab CI/CD | Jenkins | Bitrise |
|------|---------------|--------------|---------|---------|
| **平台定位** | 开发者工具链集成 | DevOps 全平台 | 通用 CI 引擎 | 移动端专项 |
| **学习曲线** | 低（YAML 语法） | 中（YAML + 概念） | 高（Groovy DSL） | 低 |
| **自托管成本** | $15/runner/月 | $15/500mins | 基础设施成本 | 不支持 |
| **并行能力** | 256 jobs/workflow | 付费版限制 | 无限制 | 5 并行 |
| **Action 市场** | 15000+ | 700+ | 1800+ | 有限 |
| **容器支持** | Docker 原生 | Docker | 需要配置 | Docker |

#### 适用场景分析

```mermaid
┌─────────────────────────────────────────────────────────────────────────┐
│                      GitHub Actions 适用场景矩阵                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   推荐使用 GitHub Actions 的场景：                                        │
│   ✓ 开源项目（免费额度慷慨）                                             │
│   ✓ 与 GitHub 代码仓库紧密集成的项目                                      │
│   ✓ 需要快速上手 CI/CD 的团队                                           │
│   ✓ 需要与 GitHub Marketplace Actions 集成的项目                          │
│   ✓ 容器化应用的构建和部署                                               │
│   ✓ 多平台测试（Windows/macOS/Linux）                                    │
│   ✓ Serverless 函数部署（Vercel/Netlify/Cloudflare）                    │
│   ✓ AI/ML 模型的自动化训练和部署                                         │
│                                                                         │
│   建议考虑其他方案的场景：                                                │
│   ✗ 需要深度 Kubernetes 集成的企业项目                                   │
│   ✗ 复杂的多阶段部署流程（考虑 Tekton/Argo CD）                          │
│   ✗ 移动端专项构建（考虑 Bitrise/CodeMagic）                            │
│   ✗ 已有 Jenkins 基础设施的大型组织                                      │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 完整配置教程

### 环境准备与基础配置

在开始使用 GitHub Actions 之前，需要完成以下环境准备工作。

#### 仓库配置清单

```yaml
# .github/ISSUE_TEMPLATE/
# ─────────────────────────────────────────────────────────────────────────────
# 问题模板配置
# ─────────────────────────────────────────────────────────────────────────────

# bug_report.yml
name: Bug Report
description: 报告一个 Bug 帮助我们改进
title: '[Bug]: '
labels: [bug]
body:
  - type: markdown
    attributes:
      value: |
        ## Bug 描述
        请清楚描述你遇到的问题。
        
        ## 复现步骤
        1. Go to '...'
        2. Click on '....'
        3. Scroll down to '....'
        4. See error
        
        ## 预期行为
        描述你期望发生的事情。
        
        ## 屏幕截图
        如果适用，添加截图帮助说明问题。
        
        ## 环境信息
        - OS: [e.g. macOS, Windows]
        - Browser: [e.g. chrome, safari]
        - Version: [e.g. 22.0]
        
        ## 附加上下文
        添加关于该 Bug 的其他信息。
        
  - type: textarea
    id: reproduction
    attributes:
      label: 复现步骤
      placeholder: 详细描述复现步骤...
    validations:
      required: true
      
  - type: dropdown
    id: severity
    attributes:
      label: 严重程度
      options:
        - Critical
        - High
        - Medium
        - Low
        - Trivial
    validations:
      required: true

# feature_request.yml
name: Feature Request
description: 为项目提出新功能建议
title: '[Feature]: '
labels: [enhancement]
body:
  - type: markdown
    attributes:
      value: |
        ## 功能描述
        清楚描述你希望添加的功能。
        
        ## 使用场景
        描述该功能将解决的用例。
        
        ## 替代方案
        描述你考虑过的替代方案或变通方法。
        
        ## 附加上下文
        添加关于该功能请求的其他信息。
```

#### 分支保护规则配置

```yaml
# .github/workflows/branch-protection.yml
# ─────────────────────────────────────────────────────────────────────────────
# 分支保护工作流
# ─────────────────────────────────────────────────────────────────────────────

name: Branch Protection Monitor

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main
  # 定期检查分支保护规则
  schedule:
    - cron: '0 0 * * 0'  # 每周检查

jobs:
  check-branch-protection:
    name: Check Branch Protection Rules
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Get branch protection rules
        id: protection
        uses: octokit/request-action@v2
        with:
          route: GET /repos/{owner}/{repo}/branches/{branch}/protection
          owner: ${{ github.repository_owner }}
          repo: ${{ github.event.repository.name }}
          branch: main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Validate protection rules
        run: |
          echo "Branch protection rules for main:"
          echo "Require pull request reviews: ${{ steps.protection.outputs.required_pull_request_reviews }}"
          echo "Required status checks: ${{ steps.protection.outputs.required_status_checks_contexts }}"
          echo "Enforce admin: ${{ steps.protection.outputs.enforce_admins }}"
      
      - name: Report missing protections
        if: steps.protection.outputs.required_pull_request_reviews == 'false'
        run: |
          echo "::error::Main branch is missing pull request review requirements!"
          exit 1
```

### 首个 Workflow 实战

以下是创建一个完整 CI Workflow 的详细步骤。

#### 步骤 1：创建目录结构

```bash
# 创建工作流目录
mkdir -p .github/workflows

# 目录结构
.github/
├── workflows/
│   ├── ci.yml           # 主 CI 工作流
│   ├── cd.yml           # CD 部署工作流
│   ├── pr-review.yml    # PR 审查自动化
│   └── scheduled.yml    # 定时任务
├── ISSUE_TEMPLATE/
│   ├── bug_report.yml
│   └── feature_request.yml
├── PULL_REQUEST_TEMPLATE.md
└── CODEOWNERS
```

#### 步骤 2：创建首个 CI 工作流

```yaml
# .github/workflows/first-ci.yml
# ─────────────────────────────────────────────────────────────────────────────
# 首个 CI 工作流 - 完整注释版
# ─────────────────────────────────────────────────────────────────────────────

name: My First CI Pipeline

# ─────────────────────────────────────────────────────────────────────────────
# 触发条件
# ─────────────────────────────────────────────────────────────────────────────
on:
  # 在 push 到 main 和 develop 分支时触发
  push:
    branches:
      - main
      - develop
    # 只在特定文件改变时触发（节省资源）
    paths:
      - 'src/**'
      - 'tests/**'
      - 'package.json'
      - '*.lock'
      - '.github/workflows/ci.yml'
  
  # 在 PR 打开、同步、更新时触发
  pull_request:
    branches:
      - main
    types:
      - opened
      - synchronize
      - reopened
  
  # 允许手动触发（workflow_dispatch）
  workflow_dispatch:
    inputs:
      logLevel:
        description: '日志级别'
        required: false
        default: 'warning'
        type: choice
        options:
          - info
          - warning
          - debug

# ─────────────────────────────────────────────────────────────────────────────
# 环境变量
# ─────────────────────────────────────────────────────────────────────────────
env:
  NODE_VERSION: '20'
  NPM_VERSION: '10'
  COVERAGE_THRESHOLD: 80

# ─────────────────────────────────────────────────────────────────────────────
# 作业定义
# ─────────────────────────────────────────────────────────────────────────────
jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 作业 1：Lint - 代码风格检查
  # ─────────────────────────────────────────────────────────────────────────────
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    
    # 步骤定义
    steps:
      # 1. 检出代码
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 获取完整历史，用于语义化版本判断
      
      # 2. 设置 Node.js 环境
      - name: Setup Node.js ${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          # 启用依赖缓存
          cache: 'npm'
      
      # 3. 安装依赖
      - name: Install dependencies
        run: npm ci
      
      # 4. 运行 Lint
      - name: Run ESLint
        run: npm run lint
      
      # 5. 运行 Prettier 检查
      - name: Check code formatting
        run: npm run format:check

  # ─────────────────────────────────────────────────────────────────────────────
  # 作业 2：Test - 测试套件
  # ─────────────────────────────────────────────────────────────────────────────
  test:
    name: Run Tests
    runs-on: ubuntu-latest
    # 依赖 lint 作业
    needs: lint
    
    # 服务容器（数据库等）
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
      
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      # 生成 Prisma Client（如果使用）
      - name: Generate Prisma Client
        run: npx prisma generate
        continue-on-error: true
      
      # 运行测试
      - name: Run unit tests
        run: npm run test:unit -- --coverage
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          DATABASE_URL: postgresql://test_user:test_password@localhost:5432/test_db
          REDIS_URL: redis://localhost:6379/0
      
      # 上传测试覆盖率
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          flags: unittests
          name: codecov-umbrella
          fail_ci_if_error: true
          verbose: true

  # ─────────────────────────────────────────────────────────────────────────────
  # 作业 3：Build - 构建
  # ─────────────────────────────────────────────────────────────────────────────
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: test
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ env.NODE_VERSION }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build
        env:
          NODE_ENV: production
      
      # 上传构建产物
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: |
            dist/
            build/
          retention-days: 30
          compression-level: 9

  # ─────────────────────────────────────────────────────────────────────────────
  # 作业 4：Security Scan - 安全扫描
  # ─────────────────────────────────────────────────────────────────────────────
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    needs: lint
    # 设置安全扫描的超时时间
    timeout-minutes: 15
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          # 深度扫描需要完整历史
          fetch-depth: 0
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: Run npm audit
        run: npm audit --audit-level=${{ env.AUDIT_LEVEL || 'moderate' }}
        continue-on-error: true
```

---

## 核心功能详解

### 工作流表达式与条件逻辑

GitHub Actions 提供了强大的表达式语言，可以动态控制工作流的执行行为。

#### 表达式基础

```yaml
# .github/workflows/expressions.yml
# ─────────────────────────────────────────────────────────────────────────────
# 表达式与条件逻辑完整指南
# ─────────────────────────────────────────────────────────────────────────────

name: Expression Examples

on:
  workflow_dispatch:
    inputs:
      environment:
        description: '部署环境'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 基础表达式语法
  # ─────────────────────────────────────────────────────────────────────────────
  expression-basics:
    name: Expression Basics
    runs-on: ubuntu-latest
    steps:
      - name: String literal
        run: echo "Hello World"
      
      - name: Variables
        run: echo "Ref: ${{ github.ref }}"
      
      - name: Object access
        run: echo "Repo: ${{ github.repository }}"
      
      - name: Function call
        run: echo "Now: ${{ steps.timestamp.outputs.time }}"
      
      - name: Multi-line expression
        run: |
          echo "Context: ${{ toJSON(github) }}"
    
    outputs:
      timestamp:
        value: ${{ steps.timer.outputs.now }}
        # ─────────────────────────────────────────────────────────────────────────────
        # 条件输出
        # ─────────────────────────────────────────────────────────────────────────────
        conditional: ${{ needs.build.outputs.version != '' && needs.build.outputs.version || '0.0.0' }}

  # ─────────────────────────────────────────────────────────────────────────────
  # 条件执行
  # ─────────────────────────────────────────────────────────────────────────────
  conditional-jobs:
    name: Conditional Execution
    runs-on: ubuntu-latest
    # 作业级别条件
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    
    steps:
      - name: Conditional step
        if: ${{ github.event_name == 'push' }}
        run: echo "This runs on push"
      
      - name: Environment check
        if: ${{ contains(github.event.head_commit.message, '[skip ci]') }}
        run: echo "Skipping CI"
        # 使用 always() 即使前面失败也继续
        if: ${{ always() }}

  # ─────────────────────────────────────────────────────────────────────────────
  # 矩阵组合条件
  # ─────────────────────────────────────────────────────────────────────────────
  matrix-conditions:
    name: Test on ${{ matrix.node }}-${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    if: ${{ matrix.node == 'lts' || matrix.os != 'windows-latest' }}
    
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [18, 20, 'lts']
        # 排除特定组合
        exclude:
          - os: windows-latest
            node: 18
        # 包含特殊组合
        include:
          - os: ubuntu-latest
            node: 22
            experimental: true

  # ─────────────────────────────────────────────────────────────────────────────
  # 复杂条件逻辑
  # ─────────────────────────────────────────────────────────────────────────────
  complex-conditions:
    name: Complex Logic
    runs-on: ubuntu-latest
    # 复杂的布尔逻辑
    if: >-
      github.event_name == 'push' 
      && (
        github.ref == 'refs/heads/main' 
        || contains(github.event.head_commit.message, '[ci run]')
      )
      && !contains(github.event.head_commit.message, '[skip ci]')
    
    steps:
      - name: Check if PR is ready
        if: >-
          github.event_name == 'pull_request'
          && (
            github.event.pull_request.draft == false
            || contains(github.event.pull_request.labels.*.name, 'ready-for-review')
          )
        run: echo "PR is ready for CI"
```

#### 常用函数参考

```yaml
# .github/workflows/functions.yml
# ─────────────────────────────────────────────────────────────────────────────
# GitHub Actions 表达式函数完整参考
# ─────────────────────────────────────────────────────────────────────────────

name: Functions Reference

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  functions-demo:
    name: Functions Demo
    runs-on: ubuntu-latest
    
    steps:
      # ─────────────────────────────────────────────────────────────────────────────
      # 字符串函数
      # ─────────────────────────────────────────────────────────────────────────────
      - name: String functions
        run: |
          # startsWith - 检查字符串开头
          echo "Starts with refs/heads/: ${{ startsWith(github.ref, 'refs/heads/') }}"
          
          # endsWith - 检查字符串结尾
          echo "Ends with .md: ${{ endsWith(github.event.pull_request.title, '.md') }}"
          
          # contains - 检查包含关系
          echo "Contains 'fix': ${{ contains(github.event.pull_request.body, 'fix') }}"
          
          # contains - 数组包含
          echo "Has bug label: ${{ contains(github.event.pull_request.labels.*.name, 'bug') }}"
          
          # join - 合并数组
          echo "Labels: ${{ join(github.event.pull_request.labels.*.name, ', ') }}"
          
          # toJSON - 转换为 JSON
          echo "PR JSON: ${{ toJSON(github.event.pull_request) }}"
          
          # fromJSON - 解析 JSON
          echo "PR Number: ${{ fromJSON(github.event.pull_request.number) }}"
          
          # format - 字符串格式化
          echo "Formatted: ${{ format('Hello {0}!', 'World') }}"
          
          # replace - 替换
          echo "Replaced: ${{ replace('hello-world', '-', '_') }}"
          
          # toLowerCase / toUpperCase
          echo "Lower: ${{ toLowerCase('HELLO') }}"
          echo "Upper: ${{ toUpperCase('hello') }}"
      
      # ─────────────────────────────────────────────────────────────────────────────
      # 数组函数
      # ─────────────────────────────────────────────────────────────────────────────
      - name: Array functions
        run: |
          # matrix.* - 访问矩阵变量
          echo "Matrix value: ${{ matrix.value }}"
          
          # steps.*.outputs.* - 访问步骤输出
          echo "Previous step output: ${{ steps.previous.outputs.result }}"
          
          # needs.*.outputs.* - 访问依赖作业输出
          echo "Build output: ${{ needs.build.outputs.image }}"
      
      # ─────────────────────────────────────────────────────────────────────────────
      # 验证函数
      # ─────────────────────────────────────────────────────────────────────────────
      - name: Validation functions
        run: |
          # success - 检查前一步是否成功
          echo "Previous step success: ${{ success() }}"
          
          # failure - 检查前一步是否失败
          echo "Any failure: ${{ failure() }}"
          
          # cancelled - 检查是否被取消
          echo "Was cancelled: ${{ cancelled() }}"
          
          # always - 总是返回 true
          echo "Always: ${{ always() }}"
          
          # null - 检查是否为空
          echo "Is null: ${{ github.event.pull_request.draft == null }}"
      
      # ─────────────────────────────────────────────────────────────────────────────
      # 类型转换函数
      # ─────────────────────────────────────────────────────────────────────────────
      - name: Type casting
        run: |
          # toJSON - 转换为 JSON 字符串
          echo "${{ toJSON(steps) }}"
          
          # fromJSON - 从 JSON 字符串解析
          MY_OBJECT='{"key": "value"}'
          echo "Parsed: ${{ fromJSON(MY_OBJECT).key }}"
          
          # 类型转换示例
          # String()
          # toString(123) => "123"
          # Number()
          # number('123') => 123
          # Boolean()
          # boolean('true') => true
      
      # ─────────────────────────────────────────────────────────────────────────────
      # 操作符
      # ─────────────────────────────────────────────────────────────────────────────
      - name: Operators
        run: |
          # 比较操作符
          # ==, !=, <, <=, >, >=, &&, ||, !
          
          # 示例
          echo "Equality: ${{ 1 == 1 }}"
          echo "Greater: ${{ 5 > 3 }}"
          echo "And: ${{ true && true }}"
          echo "Or: ${{ true || false }}"
          echo "Not: ${{ !false }}"
```

### 自定义 Action 开发

GitHub Actions 允许开发者创建可复用的自定义 Action，这是构建团队标准化工作流的基础。

#### JavaScript Action

```javascript
// action.yml
// name: 'Custom GitHub Action'
// description: 'A custom action for demonstration'
// author: 'Your Name'
// inputs:
//   name:
//     description: 'Name to greet'
//     required: true
//     default: 'World'
//   greeting:
//     description: 'Greeting prefix'
//     required: false
//     default: 'Hello'
// outputs:
//   greeting:
//     description: 'The generated greeting'
// runs:
//   using: 'node20'
//   main: 'index.js'

// index.js
// ─────────────────────────────────────────────────────────────────────────────
// 自定义 JavaScript Action
// ─────────────────────────────────────────────────────────────────────────────

const core = require('@actions/core');
const github = require('@actions/github');

async function run() {
  try {
    // ─────────────────────────────────────────────────────────────────────────────
    // 获取输入
    // ─────────────────────────────────────────────────────────────────────────────
    const name = core.getInput('name', { required: true });
    const greeting = core.getInput('greeting', { required: false }) || 'Hello';
    const token = core.getInput('token', { required: false }) || process.env.GITHUB_TOKEN;
    
    // ─────────────────────────────────────────────────────────────────────────────
    // 处理逻辑
    // ─────────────────────────────────────────────────────────────────────────────
    const fullGreeting = `${greeting}, ${name}!`;
    
    // 获取 GitHub 上下文
    const context = github.context;
    const prNumber = context.payload.pull_request?.number;
    
    // 设置输出
    core.setOutput('greeting', fullGreeting);
    core.setOutput('timestamp', new Date().toISOString());
    core.setOutput('pr_number', prNumber || 'none');
    
    // 设置环境变量
    core.exportVariable('GREETING_MESSAGE', fullGreeting);
    
    // 添加路径
    core.addPath('/usr/local/bin');
    
    // 记录日志
    core.info(`Generated greeting: ${fullGreeting}`);
    core.debug(`GitHub context: ${JSON.stringify(context)}`);
    
    // ─────────────────────────────────────────────────────────────────────────────
    // GitHub API 调用
    // ─────────────────────────────────────────────────────────────────────────────
    if (token) {
      const octokit = github.getOctokit(token);
      
      // 添加 PR 注释
      if (prNumber) {
        await octokit.rest.issues.createComment({
          owner: context.repo.owner,
          repo: context.repo.repo,
          issue_number: prNumber,
          body: `🤖 ${fullGreeting}\n\nThis comment was created by a custom GitHub Action.`
        });
      }
    }
    
    // ─────────────────────────────────────────────────────────────────────────────
    // 错误处理
    // ─────────────────────────────────────────────────────────────────────────────
    // 设置失败
    // core.setFailed('Something went wrong');
    
    // 警告
    core.warning('This is a warning message');
    
    // 错误（但不失败）
    core.error('This is an error message', {
      title: 'Custom Error Title',
      file: __filename,
      line: 42,
      column: 10
    });
    
    // 开始组日志
    core.startGroup('Custom Group');
    console.log('Inside custom group');
    core.endGroup();
    
  } catch (error) {
    core.setFailed(error.message);
  }
}

run();
```

#### Docker Action

```dockerfile
# Dockerfile
# ─────────────────────────────────────────────────────────────────────────────
# 自定义 Docker Action
# ─────────────────────────────────────────────────────────────────────────────

FROM ubuntu:24.04

# 设置元数据
LABEL "com.github.actions.name"="docker-action"
LABEL "com.github.actions.description"="A custom Docker-based GitHub Action"
LABEL "com.github.actions.icon"="package"
LABEL "com.github.actions.color"="purple"

# 防止交互提示
ENV DEBIAN_FRONTEND=noninteractive

# 安装依赖
RUN apt-get update && apt-get install -y \
    curl \
    jq \
    git \
    && rm -rf /var/lib/apt/lists/*

# 设置工作目录
WORKDIR /action

# 复制入口脚本
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

# 设置入口点
ENTRYPOINT ["/entrypoint.sh"]
```

```bash
#!/bin/bash
# entrypoint.sh
# ─────────────────────────────────────────────────────────────────────────────
# Docker Action 入口脚本
# ─────────────────────────────────────────────────────────────────────────────

set -e

# ─────────────────────────────────────────────────────────────────────────────
# 读取输入
# ─────────────────────────────────────────────────────────────────────────────
# GitHub 提供环境变量格式的输入
INPUT_NAME="${INPUT_NAME:-World}"
INPUT_GREETING="${INPUT_GREETING:-Hello}"

# ─────────────────────────────────────────────────────────────────────────────
# 处理逻辑
# ─────────────────────────────────────────────────────────────────────────────
echo "GitHub Event: ${GITHUB_EVENT_NAME}"
echo "GitHub Ref: ${GITHUB_REF}"
echo "GitHub Run ID: ${GITHUB_RUN_ID}"

# 生成问候语
GREETING="${INPUT_GREETING}, ${INPUT_NAME}!"
echo "::set-output name=greeting::${GREETING}"
echo "::set-output name=timestamp::$(date -Iseconds)"

# 显示上下文信息
echo "Event path: ${GITHUB_EVENT_PATH}"
if [ -f "${GITHUB_EVENT_PATH}" ]; then
    echo "Event payload:"
    cat "${GITHUB_EVENT_PATH}" | jq '.' | head -20
fi

# ─────────────────────────────────────────────────────────────────────────────
# 模拟一些处理逻辑
# ─────────────────────────────────────────────────────────────────────────────
echo "Processing..."
sleep 1

# 设置输出（新版格式）
echo "greeting=${GREETING}" >> "${GITHUB_OUTPUT}"
echo "timestamp=$(date -Iseconds)" >> "${GITHITHUB_OUTPUT}"

echo "Done!"
```

```yaml
# action.yml for Docker Action
name: 'Docker Custom Action'
description: 'A custom action using Docker'
author: 'Your Name'
inputs:
  name:
    description: 'Name to greet'
    required: false
    default: 'World'
  greeting:
    description: 'Greeting prefix'
    required: false
    default: 'Hello'
outputs:
  greeting:
    description: 'The generated greeting'
  timestamp:
    description: 'Current timestamp'
runs:
  using: 'docker'
  image: 'Dockerfile'
  env:
    SOME_ENV_VAR: 'value'
branding:
  icon: 'heart'
  color: 'purple'
```

#### Composite Action（组合 Action）

```yaml
# action.yml for Composite Action
# ─────────────────────────────────────────────────────────────────────────────
# 组合 Action 示例
# ─────────────────────────────────────────────────────────────────────────────

name: 'Setup Node.js Environment'
description: 'Sets up Node.js with caching and tools'
author: 'Your Team'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '20'
  cache:
    description: 'Package manager to cache'
    required: false
    default: 'npm'
  cache-dependency-path:
    description: 'Path to dependency files for caching'
    required: false
    default: '**/package-lock.json'

outputs:
  node-version:
    description: 'The version of Node.js that was installed'
  cache-hit:
    description: 'Whether cache was hit'

runs:
  using: 'composite'
  steps:
    # ─────────────────────────────────────────────────────────────────────────────
    # 步骤 1：安装 Node.js
    # ─────────────────────────────────────────────────────────────────────────────
    - name: Setup Node.js ${{ inputs.node-version }}
      id: setup-node
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: ${{ inputs.cache }}
        cache-dependency-path: ${{ inputs.cache-dependency-path }}
    
    # ─────────────────────────────────────────────────────────────────────────────
    # 步骤 2：安装依赖
    # ─────────────────────────────────────────────────────────────────────────────
    - name: Install dependencies
      shell: bash
      run: |
        if [ "${{ inputs.cache }}" == "npm" ]; then
          npm ci
        elif [ "${{ inputs.cache }}" == "yarn" ]; then
          yarn install --frozen-lockfile
        elif [ "${{ inputs.cache }}" == "pnpm" ]; then
          pnpm install --frozen-lockfile
        fi
    
    # ─────────────────────────────────────────────────────────────────────────────
    # 步骤 3：生成 Prisma Client
    # ─────────────────────────────────────────────────────────────────────────────
    - name: Generate Prisma Client
      if: hashFiles('**/prisma/schema.prisma')
      shell: bash
      run: npx prisma generate
    
    # ─────────────────────────────────────────────────────────────────────────────
    # 设置输出
    # ─────────────────────────────────────────────────────────────────────────────
    - name: Set outputs
      id: outputs
      shell: bash
      run: |
        echo "node-version=${{ inputs.node-version }}" >> $GITHUB_OUTPUT
        echo "cache-hit=${{ steps.setup-node.outputs.cache-hit }}" >> $GITHUB_OUTPUT

branding:
  icon: 'package'
  color: 'green'
```

### 缓存策略深度配置

```yaml
# .github/workflows/caching.yml
# ─────────────────────────────────────────────────────────────────────────────
# 高级缓存配置
# ─────────────────────────────────────────────────────────────────────────────

name: Advanced Caching

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 基础依赖缓存
  # ─────────────────────────────────────────────────────────────────────────────
  with-built-in-cache:
    name: With Built-in Cache
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js with npm cache
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Cache node_modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      
      - name: Cache pip (Python)
        uses: actions/cache@v4
        with:
          path: ~/.cache/pip
          key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
          restore-keys: |
            ${{ runner.os }}-pip-
      
      - name: Cache Gradle (Java)
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

  # ─────────────────────────────────────────────────────────────────────────────
  # 跨平台缓存
  # ─────────────────────────────────────────────────────────────────────────────
  cross-platform-cache:
    name: Cross-platform Cache (${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        include:
          - os: ubuntu-latest
            cache_dir: ~/.npm
          - os: windows-latest
            cache_dir: ~\AppData\Local\npm
          - os: macos-latest
            cache_dir: ~/Library/Caches/npm
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache based on OS
        uses: actions/cache@v4
        with:
          path: ${{ matrix.cache_dir }}
          key: ${{ matrix.os }}-npm-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ matrix.os }}-npm-

  # ─────────────────────────────────────────────────────────────────────────────
  # 分布式缓存（使用 cache-server）
  # ─────────────────────────────────────────────────────────────────────────────
  distributed-cache:
    name: Distributed Cache
    runs-on: ubuntu-latest
    
    services:
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup cache server
        run: |
          # 配置自定义缓存服务器
          echo "CACHE_SERVER=redis://localhost:6379" >> $GITHUB_ENV
      
      - name: Build with cache
        run: |
          # 使用自定义缓存
          npm ci --cache=$CACHE_SERVER

  # ─────────────────────────────────────────────────────────────────────────────
  # 构建缓存
  # ─────────────────────────────────────────────────────────────────────────────
  build-cache:
    name: Build Cache
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Restore build cache
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
            .nuxt/cache
            dist/.turbo
          key: ${{ runner.os }}-build-${{ github.sha }}
          restore-keys: |
            ${{ runner.os }}-build-
      
      - name: Build
        run: npm run build
      
      - name: Save build cache
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
            .nuxt/cache
          key: ${{ runner.os }}-build-${{ github.sha }}
```

---

## 部署配置

### 多环境部署策略

```yaml
# .github/workflows/multi-environment-deploy.yml
# ─────────────────────────────────────────────────────────────────────────────
# 多环境部署工作流
# ─────────────────────────────────────────────────────────────────────────────

name: Multi-Environment Deployment

on:
  push:
    branches:
      - develop
      - main
  workflow_dispatch:
    inputs:
      environment:
        description: 'Manual deployment environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production
      skip_tests:
        description: 'Skip tests (not recommended)'
        required: false
        type: boolean
        default: false

# 环境变量
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 预检查
  # ─────────────────────────────────────────────────────────────────────────────
  preflight:
    name: Preflight Checks
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.determine-env.outputs.environment }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Determine environment
        id: determine-env
        run: |
          if [ "${{ github.event.inputs.environment }}" != "" ]; then
            echo "environment=${{ github.event.inputs.environment }}" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref }}" == "refs/heads/main" ]; then
            echo "environment=production" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref }}" == "refs/heads/develop" ]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi
      
      - name: Check deployment permissions
        run: |
          echo "Deploying to: ${{ needs.preflight.outputs.environment }}"
          echo "Actor: ${{ github.actor }}"
          echo "Event: ${{ github.event_name }}"

  # ─────────────────────────────────────────────────────────────────────────────
  # CI 阶段
  # ─────────────────────────────────────────────────────────────────────────────
  ci:
    name: CI Pipeline
    needs: preflight
    runs-on: ubuntu-latest
    if: ${{ !inputs.skip_tests }}
    
    outputs:
      test-passed: ${{ steps.test.outputs.passed }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Test
        id: test
        run: |
          npm test
          echo "passed=true" >> $GITHUB_OUTPUT

  # ─────────────────────────────────────────────────────────────────────────────
  # 构建 Docker 镜像
  # ─────────────────────────────────────────────────────────────────────────────
  build:
    name: Build Docker Image
    needs: [preflight, ci]
    runs-on: ubuntu-latest
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Container Registry
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
        uses: docker/build-push-action@v6
        with:
          context: .
          push: ${{ needs.preflight.outputs.environment == 'production' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            BUILD_VERSION=${{ github.sha }}
            ENVIRONMENT=${{ needs.preflight.outputs.environment }}

  # ─────────────────────────────────────────────────────────────────────────────
  # 部署到 Development
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-development:
    name: Deploy to Development
    needs: [preflight, build]
    if: needs.preflight.outputs.environment == 'development'
    runs-on: ubuntu-latest
    
    environment:
      name: development
      url: https://dev.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to development
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.DEV_HOST }}
          username: ${{ secrets.DEV_USER }}
          key: ${{ secrets.DEV_SSH_KEY }}
          script: |
            cd /app/development
            docker pull ${{ needs.build.outputs.image-tag }}
            docker-compose -f docker-compose.dev.yml up -d
            ./scripts/health-check.sh http://localhost:3000/health

  # ─────────────────────────────────────────────────────────────────────────────
  # 部署到 Staging
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-staging:
    name: Deploy to Staging
    needs: [preflight, build]
    if: needs.preflight.outputs.environment == 'staging'
    runs-on: ubuntu-latest
    
    environment:
      name: staging
      url: https://staging.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run smoke tests before deploy
        run: |
          echo "Running pre-deployment smoke tests..."
      
      - name: Deploy to staging
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.STAGING_HOST }}
          username: ${{ secrets.STAGING_USER }}
          key: ${{ secrets.STAGING_SSH_KEY }}
          script: |
            cd /app/staging
            docker pull ${{ needs.build.outputs.image-tag }}
            docker-compose -f docker-compose.staging.yml up -d --no-deps
            ./scripts/health-check.sh https://staging.example.com/health
            ./scripts/notify.sh "Staging deployed: ${{ github.sha }}"

  # ─────────────────────────────────────────────────────────────────────────────
  # 部署到 Production（需要审批）
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-production:
    name: Deploy to Production
    needs: [preflight, build]
    if: needs.preflight.outputs.environment == 'production'
    runs-on: ubuntu-latest
    
    environment:
      name: production
      url: https://example.com
      # 生产环境需要人工审批
      deployment_reviewers:
        - team-leads
        - platform-team
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Run production smoke tests
        run: |
          echo "Running production smoke tests..."
          curl -f https://example.com/health || exit 1
      
      - name: Blue-Green Deployment
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          envs: IMAGE_TAG
          script: |
            cd /app/production
            
            # Blue-Green 部署
            CURRENT=$(docker compose ps -q blue)
            NEW=green
            
            if [ "$CURRENT" == "$(docker compose ps -q green)" ]; then
              NEW=blue
            fi
            
            echo "Deploying to $NEW..."
            
            # 启动新版本
            IMAGE_TAG=${{ needs.build.outputs.image-tag }} docker-compose -f docker-compose.prod.yml up -d $NEW
            
            # 等待健康检查
            ./scripts/wait-for-health.sh http://localhost:3000/health 60
            
            # 切换流量
            docker-compose -f docker-compose.prod.yml exec haproxy reload
            
            # 停止旧版本
            docker-compose -f docker-compose.prod.yml stop $CURRENT || true
            
            # 健康检查确认
            ./scripts/health-check.sh https://example.com/health
            
            # 通知
            ./scripts/notify.sh "Production deployed to $NEW: ${{ github.sha }}" \
              --slack ${{ secrets.SLACK_WEBHOOK_URL }} \
              --discord ${{ secrets.DISCORD_WEBHOOK_URL }}
      
      - name: Create deployment summary
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.repos.createCommitStatus({
              owner: context.repo.owner,
              repo: context.repo.repo,
              sha: context.sha,
              state: 'success',
              target_url: 'https://example.com',
              description: 'Deployment completed successfully',
              context: 'deployment/production'
            });

  # ─────────────────────────────────────────────────────────────────────────────
  # 回滚机制
  # ─────────────────────────────────────────────────────────────────────────────
  rollback:
    name: Rollback
    runs-on: ubuntu-latest
    if: ${{ github.event.inputs.rollback == 'true' }}
    
    environment:
      name: production
      url: https://example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Get previous successful deployment
        id: previous
        uses: actions/github-script@v7
        with:
          result-encoding: string
          script: |
            const { data: deployments } = await github.rest.repos.listDeployments({
              owner: context.repo.owner,
              repo: context.repo.repo,
              environment: 'production',
              per_page: 10
            });
            
            const successful = deployments.find(d => d.payload?.status === 'success');
            return successful?.sha || '';
      
      - name: Rollback
        if: steps.previous.outputs.result != ''
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          username: ${{ secrets.PROD_USER }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /app/production
            docker-compose -f docker-compose.prod.yml pull
            docker-compose -f docker-compose.yml up -d
            ./scripts/health-check.sh https://example.com/health
            ./scripts/notify.sh "Rolled back to: ${{ steps.previous.outputs.result }}"
```

### 蓝绿部署与金丝雀发布

```yaml
# .github/workflows/blue-green-canary.yml
# ─────────────────────────────────────────────────────────────────────────────
# 蓝绿部署与金丝雀发布
# ─────────────────────────────────────────────────────────────────────────────

name: Blue-Green & Canary Deployment

on:
  push:
    branches: [main]

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 蓝绿部署
  # ─────────────────────────────────────────────────────────────────────────────
  blue-green:
    name: Blue-Green Deployment
    runs-on: ubuntu-latest
    environment: production
    
    outputs:
      active-slot: ${{ steps.deploy.outputs.active-slot }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Determine active slot
        id: slots
        run: |
          # 从配置读取当前活跃槽位
          CURRENT=$(curl -s ${{ secrets.CONFIG_API }}/active-slot)
          if [ "$CURRENT" == "blue" ]; then
            echo "inactive-slot=green" >> $GITHUB_OUTPUT
            echo "active-slot=blue" >> $GITHUB_OUTPUT
          else
            echo "inactive-slot=blue" >> $GITHUB_OUTPUT
            echo "active-slot=green" >> $GITHUB_OUTPUT
          fi
      
      - name: Deploy to inactive slot
        id: deploy
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            cd /app
            
            # 部署到非活跃槽位
            INACTIVE=${{ steps.slots.outputs.inactive-slot }}
            docker-compose -f docker-compose.$INACTIVE.yml up -d
            
            # 健康检查
            ./scripts/health-check.sh http://$INACTIVE.$DOMAIN/health 30
      
      - name: Switch traffic
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.LB_HOST }}
          key: ${{ secrets.LB_SSH_KEY }}
          script: |
            # 更新负载均衡器配置
            ./scripts/switch-slot.sh ${{ steps.slots.outputs.inactive-slot }}
            nginx -s reload
      
      - name: Verify deployment
        run: |
          sleep 30
          curl -f https://example.com/health || exit 1
      
      - name: Update active slot config
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.PROD_HOST }}
          key: ${{ secrets.PROD_SSH_KEY }}
          script: |
            curl -X PUT ${{ secrets.CONFIG_API }}/active-slot \
              -d '${{ steps.slots.outputs.inactive-slot }}'

  # ─────────────────────────────────────────────────────────────────────────────
  # 金丝雀发布
  # ─────────────────────────────────────────────────────────────────────────────
  canary:
    name: Canary Release
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy canary (10% traffic)
        run: |
          # 部署 10% 流量的金丝雀版本
          kubectl set image deployment/app canary=app:${{ github.sha }}
          kubectl patch deployment app -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":"25%","maxUnavailable":0}}}}'
        
      - name: Wait for canary stabilization
        run: |
          # 等待 5 分钟让流量稳定
          sleep 300
      
      - name: Monitor metrics
        run: |
          # 检查错误率
          ERROR_RATE=$(curl -s ${{ secrets.METRICS_API }}/error-rate)
          if (( $(echo "$ERROR_RATE > 0.01" | bc -l) )); then
            echo "::error::Error rate too high: $ERROR_RATE"
            exit 1
          fi
          
          # 检查延迟
          P99_LATENCY=$(curl -s ${{ secrets.METRICS_API }}/p99-latency)
          if (( $(echo "$P99_LATENCY > 500" | bc -l) )); then
            echo "::error::Latency too high: $P99_LATENCY ms"
            exit 1
          fi
      
      - name: Promote canary (100% traffic)
        if: success()
        run: |
          kubectl rollout undo deployment/app  # 回滚到稳定版本
          kubectl set image deployment/app app=${{ github.sha }}
          kubectl rollout status deployment/app
      
      - name: Rollback canary
        if: failure()
        run: |
          kubectl rollout undo deployment/app
          echo "::error::Canary deployment failed, rolled back"
```

---

## 环境变量与密钥管理

### 密钥管理最佳实践

```yaml
# .github/workflows/secrets-best-practices.yml
# ─────────────────────────────────────────────────────────────────────────────
# 密钥管理最佳实践
# ─────────────────────────────────────────────────────────────────────────────

name: Secrets Management

on:
  workflow_dispatch:

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 基础密钥使用
  # ─────────────────────────────────────────────────────────────────────────────
  basic-secrets:
    name: Basic Secret Usage
    runs-on: ubuntu-latest
    
    env:
      # 非敏感环境变量
      APP_NAME: my-app
      LOG_LEVEL: info
      API_URL: https://api.example.com
    
    steps:
      # 在 step 中引用 secrets
      - name: Use secrets
        run: |
          # 方式 1：通过上下文引用
          echo "DB URL: ${{ secrets.DATABASE_URL }}"
          
          # 方式 2：通过环境变量传递
          echo "API Key is set: ${#SECRET_API_KEY//?/X}"
      
      # 使用多行密钥
      - name: Write SSH key
        run: |
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > key.pem
          chmod 600 key.pem
          ssh-keyscan -H github.com >> ~/.ssh/known_hosts
      
      # 使用 Organization secrets
      - name: Use org secrets
        run: |
          echo "Using org secret: ${{ secrets.ORG_SHARED_TOKEN }}"
    
    # 通过 env 传递给所有 steps
    env:
      SECRET_API_KEY: ${{ secrets.API_KEY }}

  # ─────────────────────────────────────────────────────────────────────────────
  # 密钥加密与验证
  # ─────────────────────────────────────────────────────────────────────────────
  secret-encryption:
    name: Secret Encryption
    runs-on: ubuntu-latest
    
    steps:
      - name: Encrypt sensitive data
        run: |
          # 使用 GPG 加密敏感配置
          echo "${{ secrets.ENCRYPTED_CONFIG }}" | gpg --decrypt > config.enc
          
          # 验证解密后的内容
          if [ -z "$(cat config.enc)" ]; then
            echo "::error::Decryption failed"
            exit 1
          fi
      
      - name: Validate secrets format
        run: |
          # 验证数据库 URL 格式
          if [[ "${{ secrets.DATABASE_URL }}" =~ ^postgresql:// ]]; then
            echo "Valid PostgreSQL URL"
          else
            echo "::error::Invalid DATABASE_URL format"
            exit 1
          fi

  # ─────────────────────────────────────────────────────────────────────────────
  # 外部密钥管理集成
  # ─────────────────────────────────────────────────────────────────────────────
  external-secrets:
    name: External Secrets Manager
    runs-on: ubuntu-latest
    
    steps:
      # AWS Secrets Manager
      - name: Fetch from AWS Secrets Manager
        uses: aws-actions/aws-secrets-manager-get-secrets@v1
        with:
          secret-ids: |
            prod/github-actions-secret
          parse-json-secrets: 'true'
      
      # HashiCorp Vault
      - name: Fetch from HashiCorp Vault
        uses: hashicorp/vault-action@v3
        with:
          method: aws
          url: https://vault.example.com
          role: github-actions
          secrets: |
            secret/data/production/api-key API_KEY | api_key
            secret/data/production/db-credentials DB_CREDS | db_creds
      
      # Google Cloud Secret Manager
      - name: Fetch from GCP Secret Manager
        uses: google-github-actions/auth@v2
        with:
          credentials_json: ${{ secrets.GCP_SA_KEY }}
      
      - name: Get GCP Secret
        run: |
          gcloud secrets versions access latest --secret=api-key --format=json

  # ─────────────────────────────────────────────────────────────────────────────
  # 临时密钥生成
  # ─────────────────────────────────────────────────────────────────────────────
  temporary-secrets:
    name: Temporary Secrets
    runs-on: ubuntu-latest
    
    steps:
      - name: Generate temporary token
        id: generate
        run: |
          # 生成临时访问令牌
          TEMP_TOKEN=$(openssl rand -hex 32)
          echo "temp_token=$TEMP_TOKEN" >> $GITHUB_OUTPUT
      
      - name: Use temporary token
        run: |
          echo "Using temp token: ${{ steps.generate.outputs.temp_token }}"
      
      - name: Cleanup
        run: |
          # 清理敏感数据
          echo "##vso[task.setvariable type=readonly;]temp_token="
          rm -f sensitive-file.txt
```

### 环境特定配置

```yaml
# .github/workflows/environment-config.yml
# ─────────────────────────────────────────────────────────────────────────────
# 环境特定配置
# ─────────────────────────────────────────────────────────────────────────────

name: Environment Configuration

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment'
        required: true
        type: choice
        options:
          - development
          - staging
          - production

jobs:
  configure:
    name: Configure for ${{ inputs.environment || 'development' }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment || 'development' }}
    
    steps:
      - uses: actions/checkout@v4
      
      # 加载环境特定配置
      - name: Load environment config
        id: config
        run: |
          ENV=${{ inputs.environment || 'development' }}
          
          case $ENV in
            development)
              echo "API_URL=https://dev-api.example.com" >> $GITHUB_ENV
              echo "LOG_LEVEL=debug" >> $GITHUB_ENV
              echo "ENABLE_DEBUG=true" >> $GITHUB_ENV
              ;;
            staging)
              echo "API_URL=https://staging-api.example.com" >> $GITHUB_ENV
              echo "LOG_LEVEL=info" >> $GITHUB_ENV
              echo "ENABLE_DEBUG=false" >> $GITHUB_ENV
              ;;
            production)
              echo "API_URL=https://api.example.com" >> $GITHUB_ENV
              echo "LOG_LEVEL=warn" >> $GITHUB_ENV
              echo "ENABLE_DEBUG=false" >> $GITHUB_ENV
              ;;
          esac
          
          echo "environment=$ENV" >> $GITHUB_OUTPUT
      
      - name: Create .env file
        run: |
          cat > .env << EOF
          NODE_ENV=${{ env.environment }}
          API_URL=${{ env.API_URL }}
          LOG_LEVEL=${{ env.LOG_LEVEL }}
          DATABASE_URL=${{ secrets.DATABASE_URL }}
          REDIS_URL=${{ secrets.REDIS_URL }}
          API_KEY=${{ secrets.API_KEY }}
          SENTRY_DSN=${{ secrets.SENTRY_DSN }}
          EOF
      
      - name: Verify configuration
        run: |
          echo "Environment: ${{ steps.config.outputs.environment }}"
          echo "API URL: ${{ env.API_URL }}"
          echo "Debug enabled: ${{ env.ENABLE_DEBUG }}"
```

---

## CI/CD 集成

### 与主流服务的集成

```yaml
# .github/workflows/integrations.yml
# ─────────────────────────────────────────────────────────────────────────────
# 主流服务集成
# ─────────────────────────────────────────────────────────────────────────────

name: Service Integrations

on:
  push:
    branches: [main]

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # Vercel 集成
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-vercel:
    name: Deploy to Vercel
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Vercel
        uses: nicksemionenko/cdeploy@v1
        with:
          token: ${{ secrets.VERCEL_TOKEN }}
          org-id: ${{ secrets.VERCEL_ORG_ID }}
          project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          prod: true
      
      - name: Comment on PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `🚀 Vercel deployment ready! [Preview URL](https://example.vercel.app)`
            })

  # ─────────────────────────────────────────────────────────────────────────────
  # Netlify 集成
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-netlify:
    name: Deploy to Netlify
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./dist
          production-deploy: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: true
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}

  # ─────────────────────────────────────────────────────────────────────────────
  # Supabase 集成
  # ─────────────────────────────────────────────────────────────────────────────
  supabase-deploy:
    name: Deploy to Supabase
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install Supabase CLI
        run: |
          npm install -g supabase
          supabase --version
      
      - name: Link to Supabase project
        run: supabase link --project-ref ${{ secrets.SUPABASE_PROJECT_REF }}
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      
      - name: Push database changes
        run: supabase db push
        env:
          SUPABASE_DB_PASSWORD: ${{ secrets.SUPABASE_DB_PASSWORD }}
      
      - name: Deploy Edge Functions
        run: supabase functions deploy
        env:
          SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}

  # ─────────────────────────────────────────────────────────────────────────────
  # Fly.io 集成
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-fly:
    name: Deploy to Fly.io
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Fly.io
        uses: superfly/flyctl-actions/setup-flyctl@master
      
      - name: Deploy to Fly.io
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}

  # ─────────────────────────────────────────────────────────────────────────────
  # Railway 集成
  # ─────────────────────────────────────────────────────────────────────────────
  deploy-railway:
    name: Deploy to Railway
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Create Railway deployment
        uses: heroku-deploy@v1
        with:
          api_key: ${{ secrets.RAILWAY_TOKEN }}
          app_name: my-app-production
          environment: production
          dockerfile_path: ./Dockerfile
```

### Kubernetes 集成

```yaml
# .github/workflows/kubernetes.yml
# ─────────────────────────────────────────────────────────────────────────────
# Kubernetes 部署
# ─────────────────────────────────────────────────────────────────────────────

name: Kubernetes Deployment

on:
  push:
    branches: [main]

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 设置 kubectl
  # ─────────────────────────────────────────────────────────────────────────────
  setup:
    name: Setup
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v4
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v2
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
          context: production
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ github.repository }}
          tags: type=sha
      
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

  # ─────────────────────────────────────────────────────────────────────────────
  # 构建并推送到 registry
  # ─────────────────────────────────────────────────────────────────────────────
  build:
    name: Build & Push
    needs: setup
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build and push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ needs.setup.outputs.image-tag }}
          cache-from: type=gha

  # ─────────────────────────────────────────────────────────────────────────────
  # 部署到 Kubernetes
  # ─────────────────────────────────────────────────────────────────────────────
  deploy:
    name: Deploy to Kubernetes
    needs: [setup, build]
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v2
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Update image
        run: |
          kubectl set image deployment/app \
            app=${{ needs.setup.outputs.image-tag }} \
            -n production
      
      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/app \
            -n production \
            --timeout=300s
      
      - name: Verify deployment
        run: |
          kubectl get pods -n production -l app=app
          kubectl get svc -n production -l app=app

  # ─────────────────────────────────────────────────────────────────────────────
  # Helm 部署
  # ─────────────────────────────────────────────────────────────────────────────
  helm-deploy:
    name: Helm Deployment
    needs: build
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v2
        with:
          kubeconfig: ${{ secrets.KUBE_CONFIG }}
      
      - name: Setup Helm
        uses: azure/setup-helm@v4
      
      - name: Install chart
        run: |
          helm upgrade --install app ./charts/app \
            --namespace production \
            --create-namespace \
            --set image.tag=${{ needs.build.outputs.image-tag }} \
            --set secrets.apiKey=${{ secrets.API_KEY }} \
            --wait \
            --timeout 5m
```

---

## 性能优化与缓存策略

### Workflow 执行优化

```yaml
# .github/workflows/optimization.yml
# ─────────────────────────────────────────────────────────────────────────────
# Workflow 性能优化
# ─────────────────────────────────────────────────────────────────────────────

name: Optimized Workflow

on:
  push:
    branches: [main]

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 优化 1：增量构建
  # ─────────────────────────────────────────────────────────────────────────────
  optimized-build:
    name: Optimized Build
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 1  # 只获取最新提交
      
      - name: Get changed files
        id: changed
        uses: tj-actions/changed-files@v41
        with:
          files: |
            src/**
            package.json
            package-lock.json
      
      - name: Conditional build
        if: steps.changed.outputs.any_changed == 'true'
        run: |
          echo "Building due to changes..."
          npm ci
          npm run build
      
      - name: Skip build
        if: steps.changed.outputs.any_changed == 'false'
        run: |
          echo "No relevant changes, skipping build"

  # ─────────────────────────────────────────────────────────────────────────────
  # 优化 2：依赖缓存
  # ─────────────────────────────────────────────────────────────────────────────
  cached-dependencies:
    name: Cached Dependencies
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # 使用 setup-node 的内置缓存
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      # 额外缓存 layer（用于多模块项目）
      - name: Cache .nuxt
        uses: actions/cache@v4
        with:
          path: .nuxt
          key: ${{ runner.os }}-nuxt-${{ hashFiles('nuxt.config.ts') }}
      
      - name: Install dependencies
        run: npm ci

  # ─────────────────────────────────────────────────────────────────────────────
  # 优化 3：并行执行
  # ─────────────────────────────────────────────────────────────────────────────
  parallel-tests:
    name: Test Suite
    runs-on: ubuntu-latest
    
    # 将测试分成多个并行任务
    strategy:
      fail-fast: false
      matrix:
        # 分片测试
        shard: [1, 2, 3, 4]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests (shard ${{ matrix.shard }}/4)
        run: |
          npx playwright test \
            --shard=${{ matrix.shard }}/4 \
            --reporter=line

  # ─────────────────────────────────────────────────────────────────────────────
  # 优化 4：构建缓存层
  # ─────────────────────────────────────────────────────────────────────────────
  docker-layer-cache:
    name: Docker with Cache
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      # 使用 GitHub Actions 缓存作为 Docker 层缓存
      - name: Build image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: false
          tags: app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          # 使用 registry 缓存
          cache-from: |
            type=gha
            type=registry,ref=${{ env.REGISTRY }}/${{ github.repository }}:latest
          cache-to: type=gha,mode=max

  # ─────────────────────────────────────────────────────────────────────────────
  # 优化 5：跳过不必要的运行
  # ─────────────────────────────────────────────────────────────────────────────
  skip-unnecessary:
    name: Smart CI
    runs-on: ubuntu-latest
    # 在根级别过滤，避免启动 runner
    if: >-
      !contains(github.event.head_commit.message, '[skip ci]') &&
      !contains(github.event.head_commit.message, '[ci skip]') &&
      !contains(github.event.pull_request.title, '[skip ci]')
    
    steps:
      - uses: actions/checkout@v4
        with:
          # 对于文档更新，跳过构建
          filter: blob:none
          path: src
      
      - name: Check if build needed
        run: |
          # 只在代码文件改变时构建
          if git diff --name-only HEAD~1 | grep -qE '\.(js|ts|jsx|tsx|vue)$'; then
            echo "code_changed=true" >> $GITHUB_OUTPUT
          fi
      
      - name: Build
        if: steps.check.outputs.code_changed == 'true'
        run: npm run build
```

### 自托管 Runner 配置

```yaml
# .github/workflows/self-hosted.yml
# ─────────────────────────────────────────────────────────────────────────────
# 自托管 Runner 配置
# ─────────────────────────────────────────────────────────────────────────────

name: Self-Hosted Runner Workflow

on:
  workflow_dispatch:

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 在自托管 runner 上运行
  # ─────────────────────────────────────────────────────────────────────────────
  run-on-self-hosted:
    name: Self-Hosted Job
    # 指定标签选择 runner
    runs-on: [self-hosted, linux, x64, production]
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Run build
        run: |
          echo "Running on: $(hostname)"
          cat /etc/os-release
          node --version
          npm --version
      
      - name: Cache using local path
        uses: actions/cache@v4
        with:
          path: ~/.npm
          key: ${{ runner.name }}-npm-${{ hashFiles('**/package-lock.json') }}
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test

  # ─────────────────────────────────────────────────────────────────────────────
  # Windows 自托管 runner
  # ─────────────────────────────────────────────────────────────────────────────
  run-on-windows:
    name: Windows Self-Hosted
    runs-on: [self-hosted, windows, x64]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      
      - name: Build
        run: dotnet build -c Release
      
      - name: Test
        run: dotnet test -c Release --no-build

  # ─────────────────────────────────────────────────────────────────────────────
  # macOS 自托管 runner
  # ─────────────────────────────────────────────────────────────────────────────
  run-on-macos:
    name: macOS Self-Hosted
    runs-on: [self-hosted, macos, x64]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Xcode
        run: |
          xcode-select -p
          xcodebuild -version
      
      - name: Build
        run: xcodebuild -project App.xcodeproj -scheme App -configuration Release build

  # ─────────────────────────────────────────────────────────────────────────────
  # Docker 容器中的作业
  # ─────────────────────────────────────────────────────────────────────────────
  run-in-container:
    name: Container Job
    runs-on: ubuntu-latest
    
    container:
      image: node:20-alpine
      options: --cpus 2 --memory 4g
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test

  # ─────────────────────────────────────────────────────────────────────────────
  # 多平台构建矩阵
  # ─────────────────────────────────────────────────────────────────────────────
  multi-platform:
    name: Build ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false
      matrix:
        include:
          - os: [self-hosted, linux, arm64]
            platform: linux/arm64
          - os: [self-hosted, linux, x64]
            platform: linux/amd64
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build for ${{ matrix.platform }}
        run: |
          echo "Building for ${{ matrix.platform }}"
          npm run build -- --platform ${{ matrix.platform }}
```

---

## 成本估算

### GitHub Actions 定价模型

GitHub Actions 的计费基于存储分钟数（用于 macOS 和 Windows）和执行分钟数（用于所有操作系统）。

```yaml
# .github/workflows/cost-estimation.yml
# ─────────────────────────────────────────────────────────────────────────────
# 成本估算与优化
# ─────────────────────────────────────────────────────────────────────────────

name: Cost Estimation

on:
  workflow_dispatch:
    inputs:
      monthly_builds:
        description: '每月构建次数'
        required: true
        default: '100'
      avg_duration_minutes:
        description: '平均构建时长（分钟）'
        required: true
        default: '5'

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 成本分析
  # ─────────────────────────────────────────────────────────────────────────────
  cost-analysis:
    name: Cost Analysis
    runs-on: ubuntu-latest
    
    steps:
      - name: Calculate costs
        run: |
          MONTHLY_BUILDS=${{ inputs.monthly_builds }}
          AVG_DURATION=${{ inputs.avg_duration_minutes }}
          TOTAL_MINUTES=$((MONTHLY_BUILDS * AVG_DURATION))
          
          # Linux 免费额度：2000 分钟/月
          LINUX_FREE=2000
          LINUX_USED=$((TOTAL_MINUTES))
          LINUX_PAID=$((LINUX_USED > LINUX_FREE ? LINUX_USED - LINUX_FREE : 0))
          LINUX_COST=$(echo "scale=2; $LINUX_PAID * 0.008" | bc)  # $0.008/分钟
          
          echo "## 成本估算" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 使用量" >> $GITHUB_STEP_SUMMARY
          echo "| 指标 | 值 |" >> $GITHUB_STEP_SUMMARY
          echo "|------|-----|" >> $GITHUB_STEP_SUMMARY
          echo "| 每月构建次数 | $MONTHLY_BUILDS |" >> $GITHUB_STEP_SUMMARY
          echo "| 平均构建时长 | $AVG_DURATION 分钟 |" >> $GITHUB_STEP_SUMMARY
          echo "| 总分钟数 | $TOTAL_MINUTES |" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### Linux 成本" >> $GITHUB_STEP_SUMMARY
          echo "| 项目 | 值 |" >> $GITHUB_STEP_SUMMARY
          echo "|------|-----|" >> $GITHUB_STEP_SUMMARY
          echo "| 免费额度 | $LINUX_FREE 分钟 |" >> $GITHUB_STEP_SUMMARY
          echo "| 付费分钟数 | $LINUX_PAID |" >> $GITHUB_STEP_SUMMARY
          echo "| 预计月费 | \$$LINUX_COST |" >> $GITHUB_STEP_SUMMARY
          echo "| 预计年费 | \$$(echo "scale=2; $LINUX_COST * 12" | bc) |" >> $GITHUB_STEP_SUMMARY

  # ─────────────────────────────────────────────────────────────────────────────
  # 成本优化建议
  # ─────────────────────────────────────────────────────────────────────────────
  optimization-tips:
    name: Optimization Tips
    runs-on: ubuntu-latest
    
    steps:
      - name: Generate optimization report
        run: |
          echo "## 成本优化建议" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 1. 使用缓存" >> $GITHUB_STEP_SUMMARY
          echo "```yaml" >> $GITHUB_STEP_SUMMARY
          echo "- uses: actions/setup-node@v4" >> $GITHUB_STEP_SUMMARY
          echo "  with:" >> $GITHUB_STEP_SUMMARY
          echo "    cache: 'npm'  # 自动缓存 node_modules" >> $GITHUB_STEP_SUMMARY
          echo "```" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 2. 跳过不必要的运行" >> $GITHUB_STEP_SUMMARY
          echo "- 使用 `paths-ignore` 跳过文档更新时的构建" >> $GITHUB_STEP_SUMMARY
          echo "- 在 commit message 中添加 `[skip ci]`" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 3. 并行化测试" >> $GITHUB_STEP_SUMMARY
          echo "- 使用矩阵策略并行运行测试" >> $GITHUB_STEP_SUMMARY
          echo "- 使用 Playwright 的 `--shard` 功能" >> $GITHUB_STEP_SUMMARY
          echo "" >> $GITHUB_STEP_SUMMARY
          echo "### 4. 选择合适的 runner" >> $GITHUB_STEP_SUMMARY
          echo "| OS | 免费额度 | 超出后价格 |" >> $GITHUB_STEP_SUMMARY
          echo "|----|----------|------------|" >> $GITHUB_STEP_SUMMARY
          echo "| Linux | 2000 分钟/月 | $0.008/分钟 |" >> $GITHUB_STEP_SUMMARY
          echo "| Windows | 2000 分钟/月 | $0.016/分钟 |" >> $GITHUB_STEP_SUMMARY
          echo "| macOS | 200 分钟/月 | $0.08/分钟 |" >> $GITHUB_STEP_SUMMARY
```

### 定价对比表

| 套餐 | 价格 | Linux 分钟 | macOS 分钟 | Windows 分钟 | 存储 |
|------|------|-----------|-----------|-------------|------|
| **免费版** | $0 | 2,000/月 | 200/月 | 2,000/月 | 500MB |
| **Team** | $4/用户/月 | 3,000/月 | 250/月 | 3,000/月 | 2GB |
| **Enterprise** | $21/用户/月 | 50,000/月 | 10,000/月 | 30,000/月 | 50GB |

> [!TIP]
> - 使用自托管 Runner 可以节省 Linux 付费分钟数
> - GitHub Enterprise Cloud 提供更优惠的分钟数套餐
> - Actions 缓存不占用存储配额

---

## 常见问题与解决方案

### 调试与故障排除

```yaml
# .github/workflows/debugging.yml
# ─────────────────────────────────────────────────────────────────────────────
# 调试与故障排除
# ─────────────────────────────────────────────────────────────────────────────

name: Debug Workflow

on:
  workflow_dispatch:
    inputs:
      debug_mode:
        description: 'Enable debug mode'
        required: false
        type: boolean
        default: false

env:
  # 启用详细日志
  ACTIONS_STEP_DEBUG: ${{ inputs.debug_mode && 'true' || 'false' }}

jobs:
  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 1：Secrets 无法访问
  # ─────────────────────────────────────────────────────────────────────────────
  debug-secrets:
    name: Debug Secrets
    runs-on: ubuntu-latest
    
    steps:
      - name: Check secret availability
        run: |
          # 检查 secrets 是否正确设置
          if [ -z "${{ secrets.SOME_SECRET }}" ]; then
            echo "::error::Secret SOME_SECRET is not set"
            echo "::warning::Please go to Settings > Secrets and add SOME_SECRET"
          else
            echo "Secret SOME_SECRET is available"
            # 安全地输出（只显示长度）
            echo "Secret length: ${#SECRET_VALUE}"
          fi
      
      - name: Verify environment
        run: |
          echo "Repository: ${{ github.repository }}"
          echo "Ref: ${{ github.ref }}"
          echo "Event: ${{ github.event_name }}"
          echo "Actor: ${{ github.actor }}"

  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 2：缓存未命中
  # ─────────────────────────────────────────────────────────────────────────────
  debug-cache:
    name: Debug Cache
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          # 清理工作目录以测试缓存
          clean: true
      
      - name: Setup Node.js with cache
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Check cache key
        id: cache-key
        run: |
          # 手动计算缓存键
          echo "Commit: ${{ github.sha }}"
          echo "Hash of package-lock.json:"
          sha256sum package-lock.json || echo "No package-lock.json"
          
          # 模拟缓存查找
          CACHE_KEY="node-${{ runner.os }}-$(sha256sum package-lock.json | cut -d' ' -f1)"
          echo "Expected cache key: $CACHE_KEY"
          echo "cache_key=$CACHE_KEY" >> $GITHUB_OUTPUT
      
      - name: Restore cache
        uses: actions/cache@v4
        id: cache-restore
        with:
          path: node_modules
          key: ${{ steps.cache-key.outputs.cache_key }}
          restore-keys: |
            node-${{ runner.os }}-

  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 3：服务容器连接失败
  # ─────────────────────────────────────────────────────────────────────────────
  debug-services:
    name: Debug Services
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - name: Debug service connection
        run: |
          # 等待服务启动
          echo "Waiting for PostgreSQL..."
          sleep 10
          
          # 检查服务状态
          echo "Checking service logs..."
          
          # 尝试连接
          PGPASSWORD=test psql -h localhost -U test -d test -c "SELECT version();"
          
          # 验证环境变量
          echo "DB Host: $POSTGRES_HOST"
          echo "DB Port: $POSTGRES_PORT"

  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 4：权限不足
  # ─────────────────────────────────────────────────────────────────────────────
  debug-permissions:
    name: Debug Permissions
    runs-on: ubuntu-latest
    
    permissions:
      # 显式声明所需权限
      contents: read
      packages: write
      deployments: write
      id-token: write  # OIDC 权限
    
    steps:
      - name: Check token permissions
        run: |
          echo "Token permissions:"
          cat <<'EOF'
          - contents: read (for checkout)
          - packages: write (for pushing images)
          - deployments: write (for deployment status)
          - id-token: write (for OIDC)
          EOF
      
      - name: Request permissions
        uses: actions/github@v8
        with:
          retrun-token: true
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # 使用 GITHUB_TOKEN 请求额外权限
          echo "Current permissions:"
          gh api /repos/${{ github.repository }}/actions/permissions

  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 5：超时问题
  # ─────────────────────────────────────────────────────────────────────────────
  debug-timeout:
    name: Debug Timeout
    runs-on: ubuntu-latest
    timeout-minutes: 5  # 设置较短超时用于测试
    
    steps:
      - name: Long running task
        run: |
          echo "Starting task at $(date)"
          
          # 模拟长时间任务
          for i in {1..10}; do
            echo "Step $i..."
            sleep 1
          done
          
          echo "Completed at $(date)"

  # ─────────────────────────────────────────────────────────────────────────────
  # 问题 6：网络问题
  # ─────────────────────────────────────────────────────────────────────────────
  debug-network:
    name: Debug Network
    runs-on: ubuntu-latest
    
    steps:
      - name: Network diagnostics
        run: |
          echo "=== Network Diagnostics ==="
          echo ""
          echo "Hostname: $(hostname)"
          echo "DNS servers:"
          cat /etc/resolv.conf
          echo ""
          echo "=== Testing connectivity ==="
          curl -I https://github.com || echo "Cannot reach GitHub"
          curl -I https://api.github.com || echo "Cannot reach GitHub API"
          echo ""
          echo "=== Environment proxy ==="
          echo "HTTP_PROXY: ${HTTP_PROXY:-not set}"
          echo "HTTPS_PROXY: ${HTTPS_PROXY:-not set}"
          echo "NO_PROXY: ${NO_PROXY:-not set}"
```

### 常见错误速查表

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| `Resource not accessible` | 权限不足 | 在 workflow 中添加 `permissions` 或使用 `GITHUB_TOKEN` |
| `Secrets is not set` | Secrets 未配置 | 在仓库 Settings > Secrets 中添加 |
| `Cache not found` | 缓存键不匹配 | 使用 `hashFiles()` 确保键正确 |
| `Service not ready` | 服务启动延迟 | 添加健康检查或 `sleep` 等待 |
| `GraphQL timeout` | API 限流 | 使用 `retry` 或减少并发 |
| `Docker build failed` | 构建上下文错误 | 检查 Dockerfile 路径 |
| `Runner not found` | 标签不匹配 | 检查 self-hosted runner 标签 |
| `Concurrency conflict` | 并发取消 | 使用 `concurrency` 组控制 |
| `Artifact expired` | 默认 90 天过期 | 使用 `retention-days` 调整 |
| `Matrix combination invalid` | 排除规则冲突 | 检查 matrix 配置 |

### 工作流调试技巧

```bash
# ─────────────────────────────────────────────────────────────────────────────
# 调试技巧 1：本地运行 Actions（使用 act）
# ─────────────────────────────────────────────────────────────────────────────

# 安装 act
brew install act

# 运行特定 workflow
act -W .github/workflows/ci.yml

# 使用特定事件触发
act workflow_dispatch -W .github/workflows/deploy.yml

# ─────────────────────────────────────────────────────────────────────────────
# 调试技巧 2：查看详细日志
# ─────────────────────────────────────────────────────────────────────────────

# 启用调试日志
# 1. 在 workflow 中设置环境变量
env:
  ACTIONS_STEP_DEBUG: true

# 2. 或在仓库设置中启用
# Settings > Actions > General > Debug logging

# ─────────────────────────────────────────────────────────────────────────────
# 调试技巧 3：模拟工作流环境
# ─────────────────────────────────────────────────────────────────────────────

# 创建本地开发 workflow
# .github/workflows/local-dev.yml
name: Local Dev Workflow

on:
  push:
    branches-ignore: [main, develop]  # 只在本地分支运行

jobs:
  local-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Simulate CI
        run: |
          echo "Local development simulation"
          # 你的测试步骤
```

---

## 服务概述与定位

### GitHub Actions 产品定位与市场地位

GitHub Actions 作为全球最大的代码托管平台 GitHub 的原生 CI/CD 解决方案，在 2026 年已服务超过 5000 万个活跃仓库，涵盖从个人开源项目到企业级复杂系统的广泛应用场景。其成功背后的核心逻辑在于"深度集成"与"生态丰富"的双轮驱动——开发者无需离开 GitHub 即可完成代码审查、自动测试、安全扫描、生产部署的完整闭环，同时 Marketplace 上超过 15000 个社区贡献的 Actions 大幅降低了自动化实现的门槛。

从市场定位角度分析，GitHub Actions 与其竞品形成了清晰的差异化竞争格局。GitLab CI/CD 作为 GitLab 平台的原生集成方案，在 DevOps 全流程覆盖方面具有优势，但其平台绑定特性限制了在多平台协作场景中的灵活性。Jenkins 作为历史最悠久的 CI 工具，凭借其高度可定制性仍在大型企业市场占有一席之地，但 Groovy DSL 的陡峭学习曲线和自托管的基础设施成本使其在中小型团队中逐渐失势。CircleCI 和 Travis CI 则在特定垂直场景（容器化、并行构建）保持竞争力，但综合功能覆盖度不及 GitHub Actions。Bitrise 作为移动端专项 CI 平台，在 iOS/Android 原生开发场景仍具优势，但跨平台和多语言支持能力有限。

#### 核心竞争优势分析

```mermaid
┌─────────────────────────────────────────────────────────────────────────┐
│                    GitHub Actions 核心竞争优势矩阵                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │    平台集成      │    │    生态丰富      │    │    学习曲线      │    │
│  ├─────────────────┤    ├─────────────────┤    ├─────────────────┤    │
│  │ • PR 评论集成    │    │ • 15000+ Actions │    │ • YAML 语法简洁  │    │
│  │ • 状态检查保护   │    │ • 模板市场       │    │ • 丰富的文档    │    │
│  │ • Release 追踪  │    │ • 社区活跃       │    │ • 入门教程完善  │    │
│  │ • Security 扫描  │    │ • 版本持续更新   │    │ • 模板市场     │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│                                                                         │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐    │
│  │    成本优势      │    │    扩展能力      │    │    托管服务      │    │
│  ├─────────────────┤    ├─────────────────┤    ├─────────────────┤    │
│  │ • 慷慨免费额度   │    │ • 自托管 Runner │    │ • 免运维        │    │
│  │ • 按分钟计费     │    │ • 容器支持      │    │ • 全球分布节点  │    │
│  │ • 免费版 2000分  │    │ • API 集成      │    │ • 自动弹性伸缩  │    │
│  │ • 开源项目特惠   │    │ • 矩阵构建      │    │ • 高可用架构    │    │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘    │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

#### 定价模型详解

GitHub Actions 采用基于使用量的透明定价模式，主要考量维度包括执行时间（分钟数）和存储空间：

| 计划层级 | 免费额度 | 超出计费（Linux） | 超出计费（macOS/Windows） | 适用场景 |
|---------|---------|-------------------|------------------------|---------|
| Free | 2000 分钟/月 | $0.008/分钟 | $0.08-0.16/分钟 | 开源项目、小型团队 |
| Team | 3000 分钟/月 | $0.008/分钟 | $0.08-0.16/分钟 | 成长型团队 |
| Enterprise | 50000 分钟/月 | $0.008/分钟 | $0.08-0.16/分钟 | 大型企业 |

**成本优化核心策略**：充分利用缓存机制减少重复构建时间、使用 `paths-ignore` 跳过非代码变更触发、合理设计并行矩阵避免资源浪费、自托管 Runner 在大规模场景下更具成本效益。

#### 技术架构与执行模型

GitHub Actions 的底层架构构建于高度分布式的容器化基础设施之上。每一个 Job 默认运行在独立的虚拟机或容器中，通过命名空间隔离确保安全性。Runner 作为工作流执行的载体，分为 GitHub 托管（无需运维、自动弹性）和自托管（完全控制、自定义环境）两种模式。Job 之间的依赖关系通过有向无环图（DAG）进行调度优化，无依赖的 Job 自动并行执行以缩短整体耗时。

#### 适用场景决策树

```mermaid
graph TD
    A[开始选择 CI/CD 工具] --> B{项目托管在哪里?}
    B -->|GitHub| C[强烈推荐 GitHub Actions]
    B -->|GitLab| D[推荐 GitLab CI/CD]
    B -->|多平台| E{规模如何?}
    E -->|小型| C
    E -->|大型| F[考虑 Jenkins + GitHub Actions]
    
    C --> G{需要移动端构建?}
    G -->|是| H[考虑 Bitrise/Codemagic]
    G -->|否| I{预算限制?}
    I -->|是| J[GitHub Actions Free 足够]
    I -->|否| K[考虑 Team/Enterprise]
    
    H --> L{是否需要深度 K8s 集成?}
    L -->|是| M[考虑 Tekton/Argo CD]
    L -->|否| N[GitHub Actions + 自定义 Runner]
```

---

## 相关资源

- [[CI/CD 流水线设计]]
- [[Docker 容器化最佳实践]]
- [[Kubernetes 部署配置]]
- [[Terraform 基础设施即代码]]
- [[GitHub Marketplace Actions]]
- [[Vercel 部署配置]]
- [[Netlify 自动化部署]]
- [[Cloudflare Workers 边缘计算]]

---

> [!SUCCESS]
> GitHub Actions 是 vibecoding 时代 CI/CD 的核心工具，通过与 GitHub 的深度集成，实现了从代码提交到生产部署的完整自动化。掌握 Matrix 构建、多环境部署、Secret 管理，以及 Docker/Vercel/Netlify 的集成，就掌握了现代 DevOps 的核心技能。
