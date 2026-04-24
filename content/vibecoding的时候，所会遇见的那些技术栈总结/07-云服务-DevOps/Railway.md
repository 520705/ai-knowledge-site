# Railway：现代应用托管与极简部署体验

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Railway 平台特性、支持的数据库、Nixpacks 构建系统、CLI 使用，以及与 Vercel/Render/Fly.io 的对比分析。

---

## 目录

1. [[#Railway 平台概述]]
2. [[#支持的数据库]]
3. [[#Railway CLI]]
4. [[#Nixpacks 构建系统]]
5. [[#典型使用场景]]
6. [[#价格模型]]
7. [[#Railway vs Vercel vs Render vs Fly.io 对比]]
8. [[#平台局限性]]
9. [[#实战：Next.js + PostgreSQL 部署]]

---

## Railway 平台概述

### 平台定位

Railway 成立于 2021 年，由 Prem Saggar 和 Jake Runburg 创立，定位为「开发者友好的现代云平台」。Railway 的核心理念是**降低云基础设施的复杂度**，让开发者能够在几分钟内部署生产级应用，而无需深入了解 Kubernetes、Docker 或复杂的云配置。

Railway 提供开箱即用的数据库支持、一键部署能力和透明合理的定价，是 vibecoding 场景下快速构建 MVP 和小型生产应用的理想选择。

### 核心优势

| 优势 | 说明 |
|------|------|
| **零配置部署** | Git 推送即可部署，自动检测框架 |
| **一键数据库** | PostgreSQL、MySQL、Redis、MongoDB 一键创建 |
| **透明定价** | 按实际使用计费，无需预付 |
| **全球边缘** | 14 个区域可选 |
| **Railway CLI** | 强大的命令行工具 |
| **模板市场** | 丰富的启动模板 |
| **团队协作** | 团队项目支持 |

### 平台架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                      Railway 平台架构                                │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     开发者界面                                  │ │
│  │   Railway Dashboard (Web) | Railway CLI | Railway API           │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     构建层（Nixpacks）                          │ │
│  │   自动检测框架 → 生成 Nix 表达式 → 构建镜像                      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     运行时层                                    │ │
│  │   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │ │
│  │   │ Web     │  │Worker   │  │Cron     │  │Plugins  │         │ │
│  │   │ Services│  │Services │  │Jobs     │  │         │         │ │
│  │   └─────────┘  └─────────┘  └─────────┘  └─────────┘         │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     数据层                                      │ │
│  │   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐         │ │
│  │   │Postgres │  │MySQL    │  │Redis    │  │MongoDB  │         │ │
│  │   └─────────┘  └─────────┘  └─────────┘  └─────────┘         │ │
│  │                                                             │ │
│  │   ┌─────────┐  ┌─────────┐  ┌─────────┐                     │ │
│  │   │Upstash  │  │Neon     │  │Turso    │                     │ │
│  │   │(Kafka)  │  │(Serverless Postgres)│                     │ │
│  │   └─────────┘  └─────────┘  └─────────┘                     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     基础设施层                                  │ │
│  │   AWS / GCP / Azure（多云支持）                                 │ │
│  │   14 个全球区域                                                  │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 支持的数据库

### 数据库矩阵

Railway 提供一键部署的托管数据库，所有数据库均提供自动备份、SSL 连接和连接池支持。

| 数据库 | 版本 | 存储限制 | 备份 | 连接池 |
|--------|------|----------|------|--------|
| **PostgreSQL** | 14-16 | 1GB（免费）| 自动 | 内置 |
| **MySQL** | 8 | 1GB（免费）| 自动 | 内置 |
| **Redis** | 6-7 | 256MB（免费）| RDB | N/A |
| **MongoDB** | 5-6 | 512MB（免费）| 自动 | 内置 |
| **ClickHouse** | 最新 | 按量计费 | 自动 | N/A |

### PostgreSQL 配置

```typescript
// railway.json - Railway 配置文件
{
    "$schema": "https://railway.app/schema.json",
    "build": {
        "builder": "NIXPACKS",
        "nixpacksPkgs": "# nixpacks pkgs"
    },
    "deploy": {
        "numReplicas": 2,
        "restartPolicyType": "ON_FAILURE",
        "restartPolicyAmount": 10
    }
}

// 或使用 TOML 格式
// railway.toml
[build]
    builder = "NIXPACKS"

[deploy]
    numReplicas = 2
    restartPolicyType = "ON_FAILURE"
    restartPolicyAmount = 10

[environment]
    NODE_ENV = "production"
```

### 环境变量

Railway 自动注入以下环境变量：

```bash
# Railway 自动注入的变量
DATABASE_URL=postgresql://user:password@host:port/db
POSTGRES_DB=railway
POSTGRES_USER=postgres
POSTGRES_PASSWORD=xxx
POSTGRES_HOST=xxx
POSTGRES_PORT=5432

REDIS_URL=redis://host:port
REDIS_PASSWORD=xxx

MONGO_URL=mongodb://user:password@host:port/db

# 应用信息
RAILWAY_PROJECT_ID=xxx
RAILWAY_DEPLOYMENT_ID=xxx
RAILWAY_GIT_COMMIT_SHA=xxx
PORT=8080  # Railway 自动设置
```

---

## Railway CLI

### 安装和配置

```bash
# 安装 Railway CLI
# macOS
brew install railway

# 或使用 npm
npm install -g @railway/cli

# 登录
railway login

# 初始化项目
railway init

# 链接到现有项目
railway link

# 查看当前项目
railway status

# 打开 Railway Dashboard
railway open
```

### 常用命令

| 命令 | 功能 | 示例 |
|------|------|------|
| `railway login` | 登录 | `railway login` |
| `railway init` | 初始化项目 | `railway init` |
| `railway link` | 链接项目 | `railway link` |
| `railway status` | 查看状态 | `railway status` |
| `railway up` | 部署 | `railway up` |
| `railway logs` | 查看日志 | `railway logs` |
| `railway run` | 本地运行 | `railway run npm dev` |
| `railway add` | 添加插件 | `railway add postgresql` |
| `railway environment` | 环境管理 | `railway environment dev` |
| `railway variables` | 变量管理 | `railway variables` |
| `railway domains` | 域名管理 | `railway domains` |
| `railway delete` | 删除部署 | `railway delete` |

### 部署命令

```bash
# 部署当前目录
railway up

# 部署指定目录
railway up ./backend

# 带环境部署
railway up --environment production

# 部署并设置变量
railway up -v NODE_ENV=production -v API_KEY=xxx

# 查看实时日志
railway logs -f

# 查看最近日志
railway logs --tail 100

# 本地运行（使用远程变量）
railway run npm dev

# 本地运行带数据库
railway run npm run dev:local
```

### 变量管理

```bash
# 设置变量
railway variable set DATABASE_URL=xxx
railway variable set API_KEY=xxx

# 批量设置
railway variable set KEY1=value1 KEY2=value2

# 从 .env 文件导入
railway variable set --file .env.production

# 查看变量
railway variable list

# 删除变量
railway variable unset KEY_NAME

# 设置私密变量（加密存储）
railway variable set --encrypt SECRET_KEY=xxx
```

### 环境管理

```bash
# 列出环境
railway environment list

# 创建环境
railway environment add staging

# 切换环境
railway environment development

# 删除环境
railway environment remove staging

# 部署到指定环境
railway up --environment production
```

---

## Nixpacks 构建系统

### Nixpacks 概述

Railway 使用 Nixpacks 作为默认构建系统。Nixpacks 是一种现代化的应用打包工具，能够自动检测项目类型并生成可重现的构建。

### 支持的语言和框架

```typescript
// Nixpacks 支持的语言和框架
const supportedLanguages = [
    // Node.js 生态
    'node',
    'nodejs',
    'node-server',
    'next',
    'remix',
    'nuxt',
    'sveltekit',
    'static',
    
    // Python 生态
    'python',
    'python-django',
    'python-fastapi',
    'python-flask',
    'python-poetry',
    
    // Go
    'go',
    'go-gin',
    'go-fiber',
    
    // Rust
    'rust',
    'actix-web',
    'axum',
    
    // Ruby
    'ruby',
    'ruby-on-rails',
    
    // PHP
    'php',
    'laravel',
    
    // Java
    'java-gradle',
    'java-maven',
    
    // 其他
    'dockerfile',  // 自定义 Dockerfile
    'elixir',
    'dart',
    'kotlin',
];
```

### 自定义构建

```dockerfile
# Dockerfile - Railway 支持自定义 Dockerfile
FROM node:20-alpine

WORKDIR /app

# 复制 package.json
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建
RUN npm run build

# 非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

```toml
# railway.toml - 使用 Dockerfile 构建
[build]
    builder = "dockerfile"
    dockerfilePath = "Dockerfile"
```

### 构建缓存

```toml
# railway.toml - 优化构建缓存
[build]
    builder = "NIXPACKS"
    
    # 缓存关键依赖
    [build.nixpacks]
        phases = ["setup", "install"]
        cache = ["node_modules", ".npm"]
```

---

## 典型使用场景

### Next.js 应用

```typescript
// railway.toml - Next.js 配置
{
    "$schema": "https://railway.app/schema.json",
    "build": {
        "builder": "NIXPACKS"
    },
    "deploy": {
        "numReplicas": 2,
        "healthcheckPath": "/api/health",
        "restartPolicyType": "ON_FAILURE"
    }
}
```

### API 服务 + 数据库

```yaml
# 典型架构
# ─────────────────────────────────────────────────────────────
# Web Service (Node.js/FastAPI)
#     │
#     └─── 连接 ───→ PostgreSQL Database
#     │
#     └─── 连接 ───→ Redis Cache
# ─────────────────────────────────────────────────────────────
```

```bash
# 部署步骤
# 1. 创建项目
railway init

# 2. 添加 PostgreSQL
railway add postgresql

# 3. 添加 Redis（可选）
railway add redis

# 4. 部署
railway up

# 5. 设置环境变量
railway variable set DATABASE_URL=${{Postgres.DATABASE_URL}}
railway variable set REDIS_URL=${{Redis.REDIS_URL}}
```

### Worker 服务

```toml
# railway.toml - Worker 配置
[build]
    builder = "NIXPACKS"

[[deploy]]
    command = "npm run worker"
    numReplicas = 1
    restartPolicyType = "always"

# 或使用分离的 worker 部署
# railway.toml - 分离 worker
[[deploy]]
    command = "npm run web"
    name = "web"
    port = 3000

[[deploy]]
    command = "npm run worker"
    name = "worker"
    healthcheckPath = "/health"
```

### Cron Jobs

```typescript
// railway.toml - Cron Job 配置
[build]
    builder = "NIXPACKS"

# Railway Plus 功能：定时任务
# 在 Dashboard 中配置 cron 表达式
# * * * * *  # 每分钟
# 0 * * * *  # 每小时
# 0 0 * * *  # 每天
```

---

## 价格模型

### 套餐对比

| 功能 | Starter（免费） | Pro | Team |
|------|----------------|-----|------|
| **项目数** | 3 | 无限 | 无限 |
| **并发构建** | 1 | 3 | 5 |
| **数据库存储** | 1GB | 10GB | 100GB |
| **Redis 存储** | 256MB | 1GB | 10GB |
| **带宽** | 100GB/月 | 1TB/月 | 无限 |
| **自定义域名** | ✓ | ✓ | ✓ |
| **SSL** | ✓ | ✓ | ✓ |
| **日志保留** | 1 天 | 7 天 | 30 天 |
| **团队成员** | 1 | 5 | 无限 |
| **支持** | 社区 | 邮件 | 优先 |

### 按使用计费

Railway 采用按实际使用计费的透明定价：

| 资源 | 价格 |
|------|------|
| **CPU** | $0.15/vCPU/小时 |
| **内存** | $0.10/GB/小时 |
| **Disk** | $0.15/GB/月 |
| **带宽** | $0.10/GB |
| **PostgreSQL** | $0.15/vCPU/小时 |
| **Redis** | $0.10/GB/小时 |
| **MongoDB** | $0.15/vCPU/小时 |

> [!TIP]
> 免费版 Starter 套餐对于个人项目和小型 MVP 完全够用，提供了足够的计算资源和数据库存储。

---

## Railway vs Vercel vs Render vs Fly.io 对比

### 核心对比表

| 维度 | Railway | Vercel | Render | Fly.io |
|------|---------|--------|--------|--------|
| **定位** | 全栈应用 | 前端+Serverless | Web 服务 | 容器应用 |
| **免费套餐** | 500 小时/月 | 100GB 带宽 | 750 小时/月 | 3 个共享 VMs |
| **数据库** | 原生支持 | 需集成 | 原生支持 | 需配置 |
| **容器支持** | 有限 | ✗ | ✗ | ✓ |
| **构建系统** | Nixpacks | 自研 | Buildpacks | Dockerfile |
| **全球边缘** | 14 区域 | 70+ 区域 | 有限 | 全球 |
| **Serverless** | ✗ | ✓ | ✗ | ✗ |
| **最低价格** | $0（免费版） | $20/月 | $7/月 | $0（免费版） |
| **学习曲线** | 低 | 低 | 低 | 中 |

### 功能矩阵

| 功能 | Railway | Vercel | Render | Fly.io |
|------|---------|--------|--------|--------|
| **Next.js** | ✓ | ✓✓✓ | ✓ | ✓ |
| **Nuxt/SvelteKit** | ✓ | ✓✓ | ✓ | ✓ |
| **FastAPI/Django** | ✓ | ✓ | ✓✓ | ✓ |
| **Node.js API** | ✓✓ | ✓✓ | ✓✓ | ✓✓ |
| **Docker 部署** | 有限 | ✗ | ✓ | ✓✓ |
| **gRPC** | ✗ | ✗ | ✗ | ✓ |
| **PostgreSQL** | ✓✓✓ | 需集成 | ✓✓ | 需配置 |
| **Redis** | ✓✓ | 需集成 | ✓✓ | 需配置 |
| **自动 HTTPS** | ✓ | ✓ | ✓ | ✓ |
| **预览部署** | ✓ | ✓ | ✓ | ✓ |
| **回滚** | ✓ | ✓ | ✓ | ✓ |

### 选择建议

> [!TIP]
> - **快速 MVP**：Railway（数据库内置）
> - **Next.js 应用**：Vercel（原生支持）
> - **简单 Web 服务**：Render（便宜）
> - **容器化应用**：Fly.io（Docker 原生）
> - **需要数据库**：Railway 或 Render
> - **成本敏感**：Railway 或 Fly.io（免费版更慷慨）

---

## 平台局限性

### Railway 的局限性

| 局限 | 说明 | 替代方案 |
|------|------|---------|
| **无 Serverless** | 仅支持常驻进程 | Vercel Functions |
| **容器支持有限** | 不支持 K8s 特性 | Fly.io |
| **区域有限** | 14 个区域 | Cloudflare/Vercel |
| **免费版限制** | 500 小时/月 | Fly.io（更慷慨） |
| **无内置 CDN** | 需配合 Cloudflare | Vercel |
| **gRPC 支持** | 有限 | Fly.io |

### 不适合的场景

> [!IMPORTANT]
> Railway 不适合以下场景：
> - 需要 Serverless 能力（函数级扩缩容）
> - 容器化微服务架构（需要 Kubernetes）
> - 超低延迟边缘计算（选择 Cloudflare Workers）
> - 超大规模应用（需要 Kubernetes + 自建）

---

## 实战：Next.js + PostgreSQL 部署

### 项目结构

```bash
my-nextjs-app/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   └── api/
│   │       └── users/
│   │           └── route.ts
│   └── lib/
│       └── prisma.ts
├── prisma/
│   └── schema.prisma
├── railway.toml
├── package.json
└── .env.example
```

### railway.toml 配置

```toml
# railway.toml
[build]
    builder = "NIXPACKS"

[deploy]
    numReplicas = 2
    restartPolicyType = "ON_FAILURE"
    restartPolicyAmount = 10
    healthcheckPath = "/api/health"

# 环境变量（示例，需要在 Dashboard 中设置敏感值）
[environment]
    NODE_ENV = "production"
    PORT = "8080"
```

### Prisma 配置

```prisma
// prisma/schema.prisma
generator client {
    provider = "prisma-client-js"
}

datasource db {
    provider = "postgresql"
    url      = env("DATABASE_URL")
}

model User {
    id        Int      @id @default(autoincrement())
    email     String   @unique
    name      String?
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt

    posts Post[]
}

model Post {
    id        Int      @id @default(autoincrement())
    title     String
    content   String?
    published Boolean  @default(false)
    authorId  Int
    author    User     @relation(fields: [authorId], references: [id])
    createdAt DateTime @default(now())
    updatedAt DateTime @updatedAt
}
```

### 部署步骤

```bash
# ─────────────────────────────────────────────────────────────
# 1. 初始化 Railway 项目
# ─────────────────────────────────────────────────────────────
railway login
railway init
railway link  # 如果已有项目

# ─────────────────────────────────────────────────────────────
# 2. 添加 PostgreSQL 数据库
# ─────────────────────────────────────────────────────────────
railway add postgresql

# Railway 会自动设置 DATABASE_URL 环境变量

# ─────────────────────────────────────────────────────────────
# 3. 设置环境变量
# ─────────────────────────────────────────────────────────────
# 在 .env.local 中设置
cat .env.example
# DATABASE_URL=
# NEXTAUTH_SECRET=
# NEXTAUTH_URL=

# 上传到 Railway
railway variables set NEXTAUTH_SECRET=your-secret-here
railway variables set NEXTAUTH_URL=https://your-app.railway.app

# ─────────────────────────────────────────────────────────────
# 4. 推送数据库 schema
# ─────────────────────────────────────────────────────────────
railway run npx prisma migrate deploy

# 或使用 Railway 的构建钩子
# railway.toml
[build]
    builder = "NIXPACKS"
    [build.nixpacks]
        env = {
            DATABASE_URL = "@variable DATABASE_URL"
        }

# ─────────────────────────────────────────────────────────────
# 5. 部署应用
# ─────────────────────────────────────────────────────────────
railway up

# ─────────────────────────────────────────────────────────────
# 6. 查看日志
# ─────────────────────────────────────────────────────────────
railway logs -f

# ─────────────────────────────────────────────────────────────
# 7. 配置域名
# ─────────────────────────────────────────────────────────────
railway domains add your-domain.com

# 或在 Dashboard 中配置 SSL
```

### 本地开发

```bash
# ─────────────────────────────────────────────────────────────
# 本地开发（使用远程数据库）
# ─────────────────────────────────────────────────────────────

# 1. 下载环境变量
railway run --print-env > .env

# 2. 启动本地开发服务器
railway run npm run dev

# ─────────────────────────────────────────────────────────────
# 本地开发（使用本地数据库）
# ─────────────────────────────────────────────────────────────

# 1. 启动本地 PostgreSQL
docker run -d \
    -e POSTGRES_USER=dev \
    -e POSTGRES_PASSWORD=dev \
    -e POSTGRES_DB=myapp \
    -p 5432:5432 \
    postgres:16-alpine

# 2. 设置本地 DATABASE_URL
export DATABASE_URL="postgresql://dev:dev@localhost:5432/myapp"

# 3. 运行迁移
npx prisma migrate dev

# 4. 启动开发服务器
npm run dev
```

### 健康检查 API

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET() {
    try {
        // 检查数据库连接
        await prisma.$queryRaw`SELECT 1`;
        
        return NextResponse.json({
            status: 'healthy',
            timestamp: new Date().toISOString(),
            version: process.env.APP_VERSION || '1.0.0',
        });
    } catch (error) {
        return NextResponse.json(
            {
                status: 'unhealthy',
                timestamp: new Date().toISOString(),
                error: 'Database connection failed',
            },
            { status: 503 }
        );
    }
}
```

### 多环境配置

```bash
# ─────────────────────────────────────────────────────────────
# 开发环境
# ─────────────────────────────────────────────────────────────
railway environment development
railway up

# ─────────────────────────────────────────────────────────────
# Staging 环境
# ─────────────────────────────────────────────────────────────
railway environment add staging
railway add postgresql  # Staging 专用数据库
railway up

# ─────────────────────────────────────────────────────────────
# 生产环境
# ─────────────────────────────────────────────────────────────
railway environment production
railway up
```

---

## 服务概述与定位

### Railway 在现代开发中的角色

Railway 是 2020 年代云平台的后起之秀，它重新定义了「开发者友好」的含义。与传统云服务相比，Railway 的核心理念是将开发者从繁琐的基础设施配置中解放出来，让「一键部署」成为现实。

#### 平台定位

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Railway 平台定位                                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  传统云服务:                                                         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 配置 VPC → 创建子网 → 配置安全组 → 创建实例 → 配置负载均衡     │   │
│  │ → 配置数据库 → 配置 DNS → 部署应用                          │   │
│  │ 总计: 数小时到数天的配置时间                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  Railway:                                                           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 连接 GitHub → Railway 自动检测 → 一键部署                    │   │
│  │ 总计: 60 秒完成部署                                         │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 与竞品的功能对比

| 功能 | Railway | Render | Fly.io | Vercel |
|------|---------|--------|--------|--------|
| **一键数据库** | ✅ | ✅ | ❌ | ❌ |
| **零配置部署** | ✅ | ✅ | ❌ | ✅ |
| **Git 集成** | ✅ | ✅ | ✅ | ✅ |
| **容器支持** | 有限 | ✅ | ✅ | ❌ |
| **边缘网络** | 14 区域 | 有限 | 全球 | 70+ 区域 |
| **按使用计费** | ✅ | ✅ | ✅ | 部分 |
| **免费套餐** | ✅ | ✅ | ✅ | ✅ |
| **团队协作** | ✅ | ✅ | ✅ | ✅ |

### Railway 平台架构详解

#### 整体架构

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Railway 整体架构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                     开发者界面层                               │ │
│  │                                                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │ │
│  │  │ Web UI      │  │ CLI         │  │ API         │          │ │
│  │  │ (Dashboard) │  │ (railway)  │  │ (REST/GraphQL)│       │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                      平台服务层                                │ │
│  │                                                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │ │
│  │  │ 构建引擎    │  │ 调度系统    │  │ 监控服务    │          │ │
│  │  │ (Nixpacks) │  │ (Nomad)     │  │ (Prometheus)│          │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │ │
│  │                                                               │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │ │
│  │  │ 网络代理    │  │ 证书管理    │  │ 存储服务    │          │ │
│  │  │ (Envoy)    │  │ (Let's Encrypt)│            │          │ │
│  │  └─────────────┘  └─────────────┘  └─────────────┘          │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                      基础设施层                                │ │
│  │                                                               │ │
│  │  ┌───────────────────────────────────────────────────────┐   │ │
│  │  │                    AWS / GCP / Azure                   │   │ │
│  │  │                                                       │   │ │
│  │  │  计算: EC2 / GCP Compute / Azure VMs                 │   │ │
│  │  │  存储: EBS / Persistent Disk / Managed Disks          │   │ │
│  │  │  网络: VPC / VPC / VNet                              │   │ │
│  │  │                                                       │   │ │
│  │  └───────────────────────────────────────────────────────┘   │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 请求处理流程

```
用户请求
    │
    ▼
Railway Proxy（负载均衡）
    │
    ├── 健康检查
    │   │
    │   └── 移除不健康实例
    │
    ├── 流量分发
    │   │
    │   └── 轮询/最少连接
    │
    └── 响应缓存
        │
        └── Cache-Control 头控制
```

---

## 完整配置教程

### 快速开始

#### 1. 安装 Railway CLI

```bash
# macOS (Homebrew)
brew install railway

# npm
npm install -g @railway/cli

# npx (无需安装)
npx @railway/cli login

# 验证安装
railway version
```

#### 2. 登录和认证

```bash
# 登录（打开浏览器进行 OAuth 认证）
railway login

# 登录团队账户
railway login --team

# 登出
railway logout

# 查看当前登录状态
railway whoami
```

#### 3. 初始化项目

```bash
# 方法 1: 在当前目录初始化新项目
railway init
# 会提示输入项目名称和团队

# 方法 2: 链接到已有项目
railway link [project-id]
# 从 Dashboard 获取 project-id

# 方法 3: 从 GitHub 导入
# 访问 railway.app/new
# 选择 GitHub 仓库
# Railway 自动检测项目类型
```

#### 4. 部署应用

```bash
# 部署当前目录
railway up

# 部署到指定环境
railway up --environment production

# 部署并设置变量
railway up -v KEY=value

# 查看部署状态
railway status

# 查看日志
railway logs -f

# 打开 Dashboard
railway open
```

### 高级配置

#### railway.toml 完整配置

```toml
# railway.toml - Railway 配置文件
# 放在项目根目录

# ─────────────────────────────────────────────────────────
# 构建配置
# ─────────────────────────────────────────────────────────
[build]
    # 构建方式: NIXPACKS 或 Dockerfile
    builder = "NIXPACKS"
    
    # Nixpacks 配置
    [build.nixpacks]
        # 基础镜像
        image = "nixpacks/dependencies"
        
        # 额外的 Nix 包
        pkgs = ["ffmpeg", "exiftool"]
        
        # 环境变量
        env = {
            NPM_CONFIG_PRODUCTION = "false"
        }
        
        # 缓存目录
        cache_dirs = ["/root/.cache", "/app/node_modules"]

# ─────────────────────────────────────────────────────────
# 部署配置
# ─────────────────────────────────────────────────────────
[deploy]
    # 副本数
    numReplicas = 2
    
    # 重启策略
    restartPolicyType = "ON_FAILURE"
    restartPolicyAmount = 10
    
    # 健康检查路径
    healthcheckPath = "/health"
    
    # 启动命令（覆盖自动检测）
    # command = "npm start"
    
    # 停止命令
    # stopCommand = "npm stop"
    
    # 预启动命令（迁移等）
    # prestartCommand = "npm run migrate"

# ─────────────────────────────────────────────────────────
# 环境变量（静态）
# ─────────────────────────────────────────────────────────
[environment.production]
    NODE_ENV = "production"
    PORT = "8080"
    
[environment.staging]
    NODE_ENV = "staging"
    PORT = "3000"

# ─────────────────────────────────────────────────────────
# 资源限制
# ─────────────────────────────────────────────────────────
[deploy.resources]
    # CPU: 0.5 - 8 vCPU
    cpu = "1"
    
    # 内存: 256MB - 32GB
    memory = "512Mi"

# ─────────────────────────────────────────────────────────
# 区域配置
# ─────────────────────────────────────────────────────────
[deploy.regions]
    # 主区域
    primary = "us-east"
    
    # 额外区域（需要付费计划）
    secondary = ["eu-west", "ap-southeast"]
```

#### JSON 格式配置

```json
// railway.json
{
    "$schema": "https://railway.app/schema.json",
    "build": {
        "builder": "NIXPACKS",
        "nixpacks": {
            "image": "nixpacks/dependencies",
            "pkgs": ["ffmpeg"],
            "env": {
                "NPM_CONFIG_PRODUCTION": "false"
            },
            "cacheDirs": ["/root/.cache", "/app/node_modules"]
        }
    },
    "deploy": {
        "numReplicas": 2,
        "restartPolicyType": "ON_FAILURE",
        "restartPolicyAmount": 10,
        "healthcheckPath": "/health",
        "resources": {
            "cpu": "1",
            "memory": "512Mi"
        }
    },
    "environment": {
        "production": {
            "NODE_ENV": "production",
            "PORT": "8080"
        }
    }
}
```

---

## 核心功能详解

### Nixpacks 深度解析

#### 支持的语言和框架

```python
# Nixpacks 支持完整列表
nixpacks_supported = {
    # Node.js 生态
    "node": "Node.js (默认版本)",
    "nodejs": "Node.js (LTS)",
    "node-server": "Node.js 服务器",
    "node-vercel": "Vercel 兼容",
    
    # 框架
    "next": "Next.js",
    "nextjs": "Next.js (显式)",
    "remix": "Remix",
    "nuxt": "Nuxt.js",
    "nuxt2": "Nuxt 2",
    "sveltekit": "SvelteKit",
    "svelte": "Svelte",
    "astro": "Astro",
    
    # Python 生态
    "python": "Python (默认)",
    "python2": "Python 2",
    "python3": "Python 3",
    "python-django": "Django",
    "python-fastapi": "FastAPI",
    "python-flask": "Flask",
    "python-pyramid": "Pyramid",
    "python-poetry": "Poetry",
    "pip": "Pip",
    
    # Go
    "go": "Go",
    "go-mod": "Go Modules",
    "go-gin": "Gin Web 框架",
    "go-fiber": "Fiber Web 框架",
    "go-echo": "Echo Web 框架",
    
    # Rust
    "rust": "Rust",
    "actix-web": "Actix Web",
    "axum": "Axum",
    "rocket": "Rocket",
    
    # Ruby
    "ruby": "Ruby",
    "ruby-on-rails": "Ruby on Rails",
    "ruby-sinatra": "Sinatra",
    
    # PHP
    "php": "PHP",
    "laravel": "Laravel",
    "composer": "Composer",
    
    # Java
    "java": "Java (默认)",
    "java-gradle": "Gradle",
    "java-maven": "Maven",
    "spring": "Spring Boot",
    
    # 其他
    "dockerfile": "Dockerfile",
    "elixir": "Elixir",
    "dart": "Dart",
    "kotlin": "Kotlin",
    "swift": "Swift",
    "zig": "Zig",
}

# 自动检测顺序
detection_order = [
    "Dockerfile",       # 优先使用 Dockerfile
    "package.json",     # Node.js
    "requirements.txt", # Python
    "go.mod",          # Go
    "Cargo.toml",       # Rust
    "Gemfile",         # Ruby
    "composer.json",    # PHP
    "pom.xml",         # Java Maven
    "build.gradle",    # Java Gradle
]
```

#### 自定义 Nixpacks 构建

```toml
# railway.toml - 自定义 Nixpacks 构建
[build]
    builder = "NIXPACKS"
    
    [build.nixpacks]
        # 基础镜像
        image = "nixpacks/dependencies"
        
        # 自定义 APT 包
        aptPkgs = ["ffmpeg", "exiftool", "libgl1"]
        
        # 自定义 npm/pip 包
        env = {
            NPM_CONFIG_PRODUCTION = "false",
            # Python 配置
            PIP_DISABLE_PIP_VERSION_CHECK = "1",
            PIP_NO_CACHE_DIR = "1",
        }
        
        # 缓存目录
        cacheDirs = [
            "/app/node_modules",
            "/root/.cache/pip",
            "/root/.cargo/registry",
        ]
        
        # 自定义构建命令
        # setupCommand = "npm run setup"
        # buildCommand = "npm run build"
        # startCommand = "npm start"
```

### 数据库深度配置

#### PostgreSQL 高级配置

```yaml
# Railway Dashboard 配置或 railway.toml
# PostgreSQL 16 with PostGIS

[build]
    builder = "NIXPACKS"

# 环境变量由 Railway 自动注入
# DATABASE_URL, POSTGRES_USER, POSTGRES_PASSWORD, etc.

# PostgreSQL 配置（通过 Railway 变量）
[environment]
    POSTGRES_HOST = "auto"
    POSTGRES_DB = "myapp"
    POSTGRES_MAX_CONNECTIONS = "100"
    POSTGRES_SHARED_BUFFERS = "256MB"
    POSTGRES_EFFECTIVE_CACHE_SIZE = "1GB"
    POSTGRES_MAINTENANCE_WORK_MEM = "128MB"
    POSTGRES_CHECKPOINT_COMPLETION_TARGET = "0.9"
    POSTGRES_WAL_BUFFERS = "16MB"
    POSTGRES_DEFAULT_STATISTICS_TARGET = "100"
    POSTGRES_RANDOM_PAGE_COST = "1.1"
    POSTGRES_EFFECTIVE_IO_CONCURRENCY = "200"
    POSTGRES_WORK_MEM = "8MB"
    POSTGRES_MIN_WAL_SIZE = "1GB"
    POSTGRES_MAX_WAL_SIZE = "4GB"
    POSTGRES_MAX_WORKER_PROCESSES = "4"
    POSTGRES_MAX_PARALLEL_WORKERS_PER_GATHERER = "2"
    POSTGRES_MAX_PARALLEL_WORKERS = "4"
    POSTGRES_MAX_PARALLEL_MAINTENANCE_WORKERS = "2"
```

#### Redis 配置

```toml
# railway.toml - Redis 配置
[environment]
    # Redis 连接
    REDIS_HOST = "auto"
    REDIS_PORT = "6379"
    REDIS_PASSWORD = "auto"
    
    # Redis 配置
    REDIS_MAX_MEMORY = "256mb"
    REDIS_MAX_MEMORY_POLICY = "allkeys-lru"
    REDIS_SAVE_INTERVAL = "900 1 300 100 60 10000"
    REDIS_APPEND_ONLY = "yes"
    REDIS_APPEND_FSYNC = "everysec"
    REDIS_TCP_KEEPALIVE = "60"
    REDIS_TIMEOUT = "0"
    REDIS_TCP_BACKLOG = "511"
```

### 部署配置

#### Docker 部署

```dockerfile
# Dockerfile - 完整示例
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装依赖
RUN npm ci

# 复制源代码
COPY . .

# 构建应用
RUN npm run build

# ─────────────────────────────────────────────────────────
# 生产阶段
# ─────────────────────────────────────────────────────────
FROM node:20-alpine AS production

WORKDIR /app

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# 复制 package.json
COPY package*.json ./

# 安装生产依赖
RUN npm ci --only=production && \
    npm cache clean --force

# 复制构建产物
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist

# 设置权限
RUN chown -R nodejs:nodejs /app

# 切换用户
USER nodejs

# 暴露端口
EXPOSE 3000

# 健康检查
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# 启动命令
CMD ["node", "dist/server.js"]
```

```toml
# railway.toml - 使用 Dockerfile
[build]
    builder = "dockerfile"
    dockerfilePath = "Dockerfile"

[deploy]
    numReplicas = 2
    healthcheckPath = "/health"
```

---

## 环境变量与密钥管理

### 变量配置

#### Railway 环境变量

```bash
# 设置环境变量
railway variables set KEY=value
railway variables set DATABASE_URL=postgresql://...

# 设置私密变量（加密存储）
railway variables set --encrypt SECRET_KEY=xxx

# 批量设置
railway variables set KEY1=value1 KEY2=value2

# 从文件导入
railway variables set --file .env.production

# 查看变量
railway variables list

# 删除变量
railway variables unset KEY

# 复制变量到其他环境
railway variables copy --from staging --to production
```

#### 多环境配置

```toml
# railway.toml - 环境特定配置
[environment.development]
    NODE_ENV = "development"
    LOG_LEVEL = "debug"
    API_URL = "http://localhost:3001"
    
[environment.staging]
    NODE_ENV = "staging"
    LOG_LEVEL = "info"
    API_URL = "https://api.staging.example.com"
    
[environment.production]
    NODE_ENV = "production"
    LOG_LEVEL = "warn"
    API_URL = "https://api.example.com"
```

### 密钥管理最佳实践

#### 敏感变量处理

```bash
# 1. 使用加密变量
railway variables set --encrypt DATABASE_PASSWORD=xxx
railway variables set --encrypt API_SECRET=xxx

# 2. 使用 Railway Secrets
railway variables set --secret JWT_SECRET

# 3. 引用其他插件的变量
railway variables set DATABASE_URL=${{Postgres.DATABASE_URL}}
railway variables set REDIS_URL=${{Redis.REDIS_URL}}

# 4. 变量引用链
railway variables set FULL_DATABASE_URL="postgresql://user:${{Postgres.POSTGRES_PASSWORD}}@${{Postgres.HOSTNAME}}:${{Postgres.PORT}}/${{Postgres.DATABASE}}"
```

---

## CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/railway-deploy.yml
name: Railway Deployment

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install Railway CLI
        run: npm install -g @railway/cli

      - name: Railway Login
        run: railway login --token ${{ secrets.RAILWAY_TOKEN }}

      - name: Railway Link
        run: railway link --project ${{ secrets.RAILWAY_PROJECT_ID }}

      - name: Set Variables
        run: |
          railway variables set NODE_ENV=production
          railway variables set --token=${{ secrets.RAILWAY_TOKEN }}

      - name: Deploy to Railway
        if: github.ref == 'refs/heads/main'
        run: railway up --prod

      - name: Deploy Preview
        if: github.event_name == 'pull_request'
        run: railway up

      - name: Health Check
        if: github.ref == 'refs/heads/main'
        run: |
          sleep 10
          curl -f https://${{ secrets.RAILWAY_PROJECT_ID }}.up.railway.app/health || exit 1
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  RAILWAY_TOKEN: ${RAILWAY_TOKEN}
  RAILWAY_PROJECT_ID: ${RAILWAY_PROJECT_ID}

.install_railway: &install_railway
  before_script:
    - npm install -g @railway/cli

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - npm run typecheck
    - npm test

deploy-staging:
  stage: deploy
  <<: *install_railway
  environment:
    name: staging
    url: https://staging-${{ secrets.RAILWAY_PROJECT_ID }}.up.railway.app
  script:
    - railway login --token $RAILWAY_TOKEN
    - railway link --project $RAILWAY_PROJECT_ID --environment staging
    - railway up
  only:
    - develop

deploy-production:
  stage: deploy
  <<: *install_railway
  environment:
    name: production
    url: https://production-${{ secrets.RAILWAY_PROJECT_ID }}.up.railway.app
  script:
    - railway login --token $RAILWAY_TOKEN
    - railway link --project $RAILWAY_PROJECT_ID --environment production
    - railway up --prod
  when: manual
  only:
    - main
```

---

## 性能优化与缓存策略

### 构建缓存

```toml
# railway.toml - 构建缓存配置
[build]
    builder = "NIXPACKS"
    
    [build.nixpacks]
        # 缓存依赖目录
        cache_dirs = [
            # Node.js
            "/app/node_modules",
            "/root/.npm",
            # Python
            "/root/.cache/pip",
            # Go
            "/root/go/pkg/mod",
            # Rust
            "/root/.cargo/registry",
            "/root/.cargo/git",
        ]
```

### 运行时优化

```toml
# railway.toml - 运行时优化
[deploy]
    # 副本数
    numReplicas = 3
    
    # 健康检查
    healthcheckPath = "/health"
    healthcheckTimeout = 5
    healthcheckInterval = 10
    healthcheckRetries = 3
    healthcheckStartPeriod = 30

[deploy.resources]
    # CPU 限制
    cpu = "2"
    
    # 内存限制
    memory = "1Gi"

# 自动扩缩容（Railway Plus）
[deploy.autoscaling]
    enabled = true
    minReplicas = 1
    maxReplicas = 10
    targetCPUUtilizationPercentage = 70
    targetMemoryUtilizationPercentage = 80
```

### 缓存策略

```typescript
// 应用级缓存配置
const cacheConfig = {
    // Redis 缓存
    redis: {
        ttl: 3600, // 1小时
        prefix: 'app:',
        compression: true,
    },
    
    // CDN 缓存（如果有）
    cdn: {
        staticAssets: '1y', // 1年
        apiResponses: '1m', // 1分钟
        htmlPages: '5m',    // 5分钟
    },
    
    // 数据库查询缓存
    database: {
        queryCacheSize: 100,
        statementTimeout: 30000,
    },
};
```

---

## 成本估算与选型建议

### 成本计算器

```typescript
// Railway 成本计算
const pricingCalculator = {
    // 场景 1: 个人项目
    personal: {
        resources: {
            cpu: 0.5,
            memory: 256, // MB
        },
        replicas: 1,
        database: 'starter', // 1GB PostgreSQL
        redis: 'starter', // 256MB
        
        calculateMonthly: () => {
            // Free tier: 500 hours
            // 实际成本取决于使用量
            
            const hourlyRate = {
                cpu: 0.15, // $0.15/vCPU/hour
                memory: 0.10, // $0.10/GB/hour
            };
            
            const hoursUsed = 500; // Free tier
            const cost = (
                (resources.cpu * hourlyRate.cpu * hoursUsed) +
                (resources.memory / 1024 * hourlyRate.memory * hoursUsed)
            );
            
            return cost.toFixed(2);
        },
    },
    
    // 场景 2: 小型团队
    smallTeam: {
        resources: {
            cpu: 1,
            memory: 512,
        },
        replicas: 2,
        database: 'pro', // 10GB PostgreSQL
        redis: 'pro', // 1GB
        
        calculateMonthly: () => {
            const hourlyRate = {
                cpu: 0.15,
                memory: 0.10,
            };
            
            const hoursPerMonth = 730; // ~30 days
            
            const computeCost = (
                resources.cpu * 
                hourlyRate.cpu * 
                hoursPerMonth * 
                resources.replicas
            );
            
            const memoryCost = (
                (resources.memory / 1024) * 
                hourlyRate.memory * 
                hoursPerMonth * 
                resources.replicas
            );
            
            return {
                compute: computeCost.toFixed(2),
                memory: memoryCost.toFixed(2),
                total: (computeCost + memoryCost).toFixed(2),
            };
        },
    },
    
    // 场景 3: 生产环境
    production: {
        resources: {
            cpu: 2,
            memory: 2048,
        },
        replicas: 3,
        database: 'pro',
        redis: 'pro',
        bandwidth: 100, // GB
        
        calculateMonthly: () => {
            const costs = {
                compute: 0,
                memory: 0,
                storage: 0,
                bandwidth: 0,
            };
            
            // 计算成本
            const hoursPerMonth = 730;
            costs.compute = 2 * 0.15 * hoursPerMonth * 3; // $657
            costs.memory = 2 * 0.10 * hoursPerMonth * 3; // $438
            costs.bandwidth = Math.max(0, 100 - 100) * 0.10; // $0 (free tier)
            
            return costs;
        },
    },
};
```

### 选型建议矩阵

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| **学习/个人项目** | Starter 免费版 | 500小时足够，数据库免费 |
| **MVP 开发** | Starter + Pro 数据库 | 快速验证想法 |
| **小团队产品** | Pro 计划 | 更好的资源和团队协作 |
| **生产环境** | Pro + 额外计算 | 高可用性和性能 |
| **高流量应用** | Pro + 预留实例 | 成本可预测 |

---

## 常见问题与解决方案

### 构建问题

#### 问题：构建失败

```bash
# 常见原因和解决方案

# 1. 依赖安装失败
# 解决方案：检查 package.json 或 requirements.txt
railway logs --tail 200

# 2. Nixpacks 无法检测项目类型
# 解决方案：使用 Dockerfile
[build]
    builder = "dockerfile"
    dockerfilePath = "Dockerfile"

# 3. 构建超时
# 解决方案：优化构建命令或使用付费计划
npm run build -- --no-lint
```

#### 问题：缓存未生效

```bash
# 解决方案：清除缓存
railway variables set NIXPACKS_NO_CACHE=1
railway up --force

# 或者在 railway.toml 中配置
[build.nixpacks]
    cache_dirs = ["/app/node_modules"]
```

### 部署问题

#### 问题：启动失败

```bash
# 检查日志
railway logs -f

# 常见原因：

# 1. 端口配置错误
# Railway 需要 PORT 环境变量
# Railway 会自动设置 PORT，无需手动配置
# 但确保应用监听 PORT

# 2. 健康检查失败
# 确保有 /health 端点
# app.get('/health', (req, res) => res.json({ status: 'ok' }))

# 3. 依赖服务未就绪
# 使用 depends_on 等待依赖
# Railway 自动处理数据库依赖
```

#### 问题：数据库连接失败

```bash
# 确保使用正确的连接字符串
railway variables list

# 检查 DATABASE_URL 格式
# postgresql://user:password@host:port/database

# 确保使用 Railway 提供的变量
railway variables set DATABASE_URL=${{Postgres.DATABASE_URL}}

# 本地开发时使用
railway run npm run dev
```

### 运行时问题

#### 问题：内存溢出

```bash
# 解决方案：增加内存限制
# railway.toml
[deploy.resources]
    memory = "1Gi"  # 增加内存

# 或者在 Dashboard 中调整
```

#### 问题：响应缓慢

```bash
# 解决方案：增加副本和优化配置
# railway.toml
[deploy]
    numReplicas = 2  # 增加副本

[deploy.resources]
    cpu = "2"  # 增加 CPU
```

### 存储问题

#### 问题：卷空间不足

```bash
# 检查存储使用
railway status

# 创建新卷（更大的）
railway volumes create new_data --region hkg --size 50

# 迁移数据
# 1. 停止应用
railway run stop

# 2. 创建快照
railway volumes create backup --region hkg --size 10

# 3. 迁移数据
railway run bash -c "cp -r /data/* /backup/"
```

### 数据库问题

#### 问题：连接失败

```bash
# 检查连接字符串
railway variables list | grep DATABASE

# 确保使用正确的格式
# postgresql://user:password@host:port/database

# 本地连接
railway run psql

# 远程连接
railway connect postgres
```

#### 问题：数据库迁移失败

```bash
# 使用 Railway CLI 运行迁移
railway run npx prisma migrate deploy

# 或者使用自定义命令
railway run npm run db:migrate

# 查看迁移日志
railway logs | grep migration
```

---

## 高级功能

### 团队协作

#### 团队管理

```bash
# 创建团队
railway team create my-team

# 邀请成员
railway team invitations send user@example.com --team my-team

# 列出团队成员
railway team members list --team my-team

# 移除成员
railway team members remove user@example.com --team my-team

# 设置成员角色
railway team members update user@example.com --role developer --team my-team
# 角色: owner, developer, viewer
```

#### 项目权限

```bash
# 转移项目所有权
railway projects transfer project-id --team new-team

# 复制项目
railway projects clone source-project-id

# 归档项目
railway projects archive project-id

# 恢复归档项目
railway projects unarchive project-id
```

### 插件系统

#### 可用插件

```bash
# 添加插件
railway add postgresql
railway add redis
railway add mongodb
railway add mysql

# 查看可用插件
railway plugins list

# 插件配置
railway plugins configure postgresql --version 16
```

#### 自定义插件

```bash
# railway.json
{
    "plugins": [
        {
            "id": "custom-plugin",
            "name": "My Custom Plugin",
            "version": "1.0.0",
            "environment": {
                "CUSTOM_VAR": "value"
            }
        }
    ]
}
```

### 监控与日志

#### 日志管理

```bash
# 查看实时日志
railway logs -f

# 查看最近日志
railway logs --tail 100

# 查看特定时间范围
railway logs --since "2024-01-01T00:00:00Z"

# 搜索日志
railway logs | grep error

# 导出日志
railway logs > app.log

# 日志保留策略
# Starter: 1 天
# Pro: 7 天
# Team: 30 天
```

#### 性能监控

```bash
# 查看资源使用
railway status

# 查看部署指标
railway metrics

# 集成外部监控
# railway.json
{
    "integrations": {
        "datadog": {
            "apiKey": "xxx",
            "enabled": true
        },
        "sentry": {
            "dsn": "xxx",
            "enabled": true
        }
    }
}
```

### 备份与恢复

#### 数据库备份

```bash
# 创建数据库备份
railway run pg_dump -Fc mydb > backup.dump

# 列出备份
railway backups list

# 恢复备份
railway backups restore backup-id

# 自动备份配置
# railway.json
{
    "backup": {
        "enabled": true,
        "schedule": "0 2 * * *",  # 每天凌晨2点
        "retention": 7  # 保留7天
    }
}
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.railway.app |
| CLI 文档 | https://docs.railway.app/reference/cli |
| API 文档 | https://docs.railway.app/reference/api |
| 模板市场 | https://railway.app/templates |
| 社区 | https://community.railway.app |
| GitHub | https://github.com/railwayapp |
| Discord | https://discord.gg/railway |

### 相关工具

| 工具 | 说明 |
|------|------|
| Railway CLI | 命令行工具 |
| Railway VS Code | VS Code 扩展 |
| Railway GitHub App | GitHub 集成 |
| Railway GitLab Plugin | GitLab 集成 |
| Nixpacks | 构建系统文档 |
| Doppler | 环境变量管理 |
| Neon's Postgres | Serverless Postgres |

---

> [!SUCCESS]
> Railway 是 vibecoding 场景下快速构建生产级应用的理想平台，其一键数据库、内置 PostgreSQL/MySQL/Redis 支持，以及透明的按使用计费模式，使开发者能够专注于代码而非基础设施。对于需要快速交付 MVP 的团队，Railway 提供了 Vercel 和传统云服务之间的最佳平衡点。
