# Vercel：前端部署的首选平台

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Vercel 平台架构、Edge Network、Serverless Functions、Next.js 部署配置，以及与 Netlify/Cloudflare Pages 的对比分析。

---

## 目录

1. [[#Vercel 平台概述]]
2. [[#Vercel 技术架构]]
3. [[#Edge Network 与边缘计算]]
4. [[#Vercel Functions（Serverless）]]
5. [[#Next.js 部署与配置]]
6. [[#构建配置详解]]
7. [[#域名与 DNS 配置]]
8. [[#预览部署（Preview Deployments）]]
9. [[#Analytics 与 Speed Insights]]
10. [[#价格体系]]
11. [[#Vercel vs Netlify vs Cloudflare Pages 对比]]
12. [[#实战：Next.js 应用部署]]

---

## Vercel 平台概述

### 平台定位

Vercel 由 Guillermo Rauch 于 2015 年创立，是 Next.js 框架的创建者和主要维护者，也是现代前端部署领域的领导者。Vercel 提供开箱即用的部署体验，支持从静态网站到全栈应用的各类前端项目。

Vercel 的核心理念是**开发者体验优先**（Developer Experience First）：通过 Git 集成实现零配置部署，通过 Edge Network 实现全球低延迟访问，通过 Serverless Functions 实现后端能力扩展。

### 核心优势

| 优势 | 说明 |
|------|------|
| **零配置部署** | Git push 即可部署，无需编写 CI/CD 配置 |
| **Next.js 原生支持** | 框架创建者，享有最新特性优先支持 |
| **Edge Network** | 全球 70+ 个边缘节点，就近访问 |
| **预览部署** | 每个 PR 自动生成预览 URL |
| **即时回滚** | 一键回滚到任意历史版本 |
| **内置 Analytics** | 无需额外集成即可获得性能数据 |

### 支持的框架

```typescript
// Vercel 支持的所有主流框架
const supportedFrameworks = [
    // React 框架
    'Next.js',
    'Create React App',
    'Vite',
    'Remix',
    // Vue 框架
    'Nuxt',
    'Vue',
    // Svelte 框架
    'Svelte',
    'SvelteKit',
    // 静态站点生成器
    'Astro',
    'Gatsby',
    'Hugo',
    'Eleventy',
    // 全栈框架
    'Nest.js',
    'RedwoodJS',
    'Blitz',
    // 其他
    'Angular',
    'SvelteKit'
];
```

---

## Vercel 技术架构

### 全球基础设施

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Vercel Edge Network                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐             │
│  │  Asia   │  │ Europe  │  │ America │  │ Oceania │             │
│  │ Pacific │  │         │  │         │  │         │             │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘             │
│       │             │             │             │                   │
│  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐  ┌────┴────┐             │
│  │ Tokyo   │  │ Frankfurt│ │ New York │ │ Sydney  │             │
│  │ Seoul   │  │ London   │ │ San Jose │ │ Melbourne│            │
│  │ Singapore│ │ Paris   │  │ Toronto  │ │         │             │
│  │ Hong Kong│ │ Amsterdam│ │ Miami    │ │         │             │
│  │ Mumbai  │  │ Stockholm│ │ Bogota  │ │         │             │
│  └─────────┘  └─────────┘  └─────────┘  └─────────┘             │
│                                                                     │
│  ──────────────────────────────────────────────────────────────    │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐ │
│  │                   Regional Compute Layer                       │ │
│  │   每个区域：Lambda + S3 + KV + Postgres + Redis              │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 请求处理流程

```
用户请求
    │
    ▼
DNS 解析（vercel.app / 自定义域名）
    │
    ▼
全球 CDN 边缘节点（最近的 PoP）
    │
    ├── 静态资源 → 直接返回（缓存命中）
    │
    └── 动态请求 → Regional Edge
                      │
                      ▼
                ┌───────────┐
                │ SSR/SSG   │
                │ Functions │
                └───────────┘
                      │
                      ▼
                响应返回
```

---

## Edge Network 与边缘计算

### Edge Functions 特性

Vercel Edge Functions 基于 V8 隔离区（V8 Isolates）运行，而非传统 Node.js 环境，具有以下特性：

| 特性 | Edge Functions | Serverless Functions |
|------|----------------|---------------------|
| **运行时** | V8 Isolates | Node.js |
| **冷启动** | < 5ms | 100-500ms |
| **执行位置** | 边缘节点 | 区域数据中心 |
| **支持 API** | Fetch API | Node.js API |
| **npm 包** | 有限（轻量） | 全部 |
| **适用场景** | 路由重写、A/B 测试 | 复杂业务逻辑 |

### Edge Function 示例

```typescript
// middleware.ts - Edge Middleware
// 位置：项目根目录

import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
    // 记录请求
    console.log('Request IP:', request.ip);
    console.log('Request URL:', request.url);
    
    // A/B 测试：根据 Cookie 分组
    const abGroup = request.cookies.get('ab-group')?.value || 
        (Math.random() > 0.5 ? 'A' : 'B');
    
    // 重写请求到对应版本
    if (abGroup === 'B') {
        return NextResponse.rewrite(
            new URL('/beta' + request.nextUrl.pathname, request.url)
        );
    }
    
    // 添加响应头
    const response = NextResponse.next();
    response.headers.set('X-Custom-Header', 'value');
    response.headers.set('X-AB-Group', abGroup);
    
    // 设置 Cookie（如果不存在）
    if (!request.cookies.get('ab-group')) {
        response.cookies.set('ab-group', abGroup, {
            maxAge: 60 * 60 * 24 * 30, // 30 天
            path: '/',
            sameSite: 'lax'
        });
    }
    
    return response;
}

// 配置匹配的路径
export const config = {
    matcher: [
        /*
         * 匹配所有路径除了：
         * - api (API 路由)
         * - _next/static (静态文件)
         * - _next/image (图像优化)
         * - favicon.ico
         */
        '/((?!api|_next/static|_next/image|favicon.ico).*)'
    ]
};
```

### 中间件用例

```typescript
// geolocation.ts - 基于地理位置的重定向
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
    const country = request.geo?.country;
    const city = request.geo?.city;
    
    // 检测到欧盟用户，添加 GDPR Cookie 提示
    if (country === 'EU' || country === 'DE' || country === 'FR') {
        const response = NextResponse.next();
        response.headers.set('X-GDPR-Region', 'true');
        return response;
    }
    
    return NextResponse.next();
}
```

---

## Vercel Functions（Serverless）

### 函数类型对比

| 类型 | 冷启动 | 执行时间限制 | 运行时 |
|------|--------|-------------|--------|
| **Edge Function** | < 5ms | 50ms | V8 |
| **Serverless Function** | ~100ms | 10s（免费）/60s（付费） | Node.js |
| **ISR** | N/A | N/A | 缓存页面 |

### Serverless Function 示例

```typescript
// pages/api/users.ts - 用户 API
// 或 app/api/users/route.ts (App Router)

import type { NextApiRequest, NextApiResponse } from 'next';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export default async function handler(
    req: NextApiRequest,
    res: NextApiResponse
) {
    switch (req.method) {
        case 'GET':
            return handleGetUsers(req, res);
        case 'POST':
            return handleCreateUser(req, res);
        default:
            res.setHeader('Allow', ['GET', 'POST']);
            return res.status(405).json({ error: 'Method not allowed' });
    }
}

async function handleGetUsers(req: NextApiRequest, res: NextApiResponse) {
    const { page = '1', limit = '10' } = req.query;
    
    const users = await prisma.user.findMany({
        skip: (Number(page) - 1) * Number(limit),
        take: Number(limit),
        orderBy: { createdAt: 'desc' },
        select: {
            id: true,
            name: true,
            email: true,
            createdAt: true
        }
    });
    
    const total = await prisma.user.count();
    
    return res.status(200).json({
        users,
        pagination: {
            page: Number(page),
            limit: Number(limit),
            total,
            pages: Math.ceil(total / Number(limit))
        }
    });
}

async function handleCreateUser(req: NextApiRequest, res: NextApiResponse) {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({ error: 'Name and email are required' });
    }
    
    const user = await prisma.user.create({
        data: { name, email }
    });
    
    return res.status(201).json(user);
}
```

### App Router API Route

```typescript
// app/api/users/route.ts - App Router 风格
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET(request: NextRequest) {
    const { searchParams } = new URL(request.url);
    const page = searchParams.get('page') || '1';
    const limit = searchParams.get('limit') || '10';
    
    const users = await prisma.user.findMany({
        skip: (Number(page) - 1) * Number(limit),
        take: Number(limit)
    });
    
    return NextResponse.json({ users });
}

export async function POST(request: NextRequest) {
    const body = await request.json();
    const { name, email } = body;
    
    if (!name || !email) {
        return NextResponse.json(
            { error: 'Missing required fields' },
            { status: 400 }
        );
    }
    
    const user = await prisma.user.create({
        data: { name, email }
    });
    
    return NextResponse.json(user, { status: 201 });
}
```

---

## Next.js 部署与配置

### vercel.json 配置

```json
{
    "framework": "nextjs",
    "buildCommand": "npm run build",
    "devCommand": "npm run dev",
    "installCommand": "npm install",
    "outputDirectory": ".next",
    "regions": ["iad1", "sfo1", "hnd1"],
    "rewrites": [
        {
            "source": "/apiOLD/:path*",
            "destination": "https://old-site.vercel.app/:path*"
        }
    ],
    "headers": [
        {
            "source": "/(.*)",
            "headers": [
                {
                    "key": "X-Frame-Options",
                    "value": "DENY"
                },
                {
                    "key": "X-Content-Type-Options",
                    "value": "nosniff"
                },
                {
                    "key": "Referrer-Policy",
                    "value": "strict-origin-when-cross-origin"
                }
            ]
        }
    ],
    "redirects": [
        {
            "source": "/old-page",
            "destination": "/new-page",
            "permanent": true
        }
    ],
    "trailingSlash": true
}
```

### 环境变量配置

```bash
# .env.example - 环境变量示例
# ─────────────────────────────────────────────────────────────

# 数据库
DATABASE_URL=

# 认证
NEXTAUTH_SECRET=
NEXTAUTH_URL=

# 第三方 API
OPENAI_API_KEY=
STRIPE_SECRET_KEY=

# 应用配置
NEXT_PUBLIC_APP_URL=
```

```typescript
// lib/config.ts - 类型安全的环境变量
import { z } from 'zod';

const envSchema = z.object({
    DATABASE_URL: z.string().url(),
    NEXTAUTH_SECRET: z.string().min(32),
    NEXTAUTH_URL: z.string().url().optional(),
    OPENAI_API_KEY: z.string().startsWith('sk-'),
    NEXT_PUBLIC_APP_URL: z.string().url(),
});

export const env = envSchema.parse(process.env);
```

---

## 构建配置详解

### next.config.js

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
    // ─────────────────────────────────────────────────────
    // 图片优化
    // ─────────────────────────────────────────────────────
    images: {
        remotePatterns: [
            {
                protocol: 'https',
                hostname: 'images.unsplash.com',
            },
            {
                protocol: 'https',
                hostname: '*.amazonaws.com',
            },
            {
                protocol: 'https',
                hostname: 'picsum.photos',
            }
        ],
        formats: ['image/avif', 'image/webp'],
        deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048],
        imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
    },
    
    // ─────────────────────────────────────────────────────
    // 编译优化
    // ─────────────────────────────────────────────────────
    compiler: {
        // 移除 console.log（生产环境）
        removeConsole: process.env.NODE_ENV === 'production',
        // SWC 压缩
        swcMinify: true,
    },
    
    // ─────────────────────────────────────────────────────
    // 实验性特性
    // ─────────────────────────────────────────────────────
    experimental: {
        // 优化包导入
        optimizePackageImports: ['lucide-react', '@mui/material', 'recharts'],
        // 服务端组件严格模式
        serverActions: {
            bodySizeLimit: '2mb',
        },
        // PPR (Partial Prerendering)
        ppr: process.env.NODE_ENV === 'development',
    },
    
    // ─────────────────────────────────────────────────────
    // 重定向规则
    // ─────────────────────────────────────────────────────
    async redirects() {
        return [
            {
                source: '/old-blog/:slug',
                destination: '/blog/:slug',
                permanent: true, // 301 重定向
            },
            {
                source: '/legacy',
                destination: '/new-landing',
                permanent: false, // 302 重定向
            },
        ];
    },
    
    // ─────────────────────────────────────────────────────
    // Header 配置
    // ─────────────────────────────────────────────────────
    async headers() {
        return [
            {
                source: '/(.*)',
                headers: [
                    {
                        key: 'X-DNS-Prefetch-Control',
                        value: 'on',
                    },
                    {
                        key: 'Strict-Transport-Security',
                        value: 'max-age=63072000; includeSubDomains; preload',
                    },
                ],
            },
        ];
    },
    
    // ─────────────────────────────────────────────────────
    // webpack 配置
    // ─────────────────────────────────────────────────────
    webpack: (config, { isServer }) => {
        // 优化图片加载
        config.module.rules.push({
            test: /\.(png|jpg|jpeg|gif|svg)$/i,
            type: 'asset',
        });
        
        // Node.js 外部化
        if (isServer) {
            config.externals = [...(config.externals || []), '@node-rs/argon2'];
        }
        
        return config;
    },
};

module.exports = nextConfig;
```

---

## 域名与 DNS 配置

### 自定义域名配置

```bash
# Vercel CLI 配置域名
vercel domains add example.com
vercel certs add example.com

# 或通过 Dashboard 配置
# Settings → Domains → Add Domain
```

### DNS 配置示例

```typescript
// DNS 配置指南（针对不同域名服务商）

const dnsConfig = {
    // A 记录（根域名）
    '@': {
        type: 'A',
        value: '76.76.21.21',  // Vercel IP
        ttl: 300
    },
    
    // CNAME 记录（www 子域名）
    'www': {
        type: 'CNAME',
        value: 'cname.vercel-dns.com',
        ttl: 300
    },
    
    // CNAME 记录（API 子域名）
    'api': {
        type: 'CNAME',
        value: 'cname.vercel-dns.com',
        ttl: 300
    },
    
    // TXT 记录（验证）
    '@': {
        type: 'TXT',
        name: 'vercel-domain-verification',
        value: 'xxxxx-xxxx-xxxx',
        ttl: 300
    }
};
```

### SSL 证书

Vercel 自动为所有域名提供 Let's Encrypt SSL 证书，支持：

- 自动续期
- 自定义证书上传
- 支持通配符证书
- 支持 ALPN 和 SNI

---

## 预览部署（Preview Deployments）

### 预览部署工作流

```
开发者 → Push PR/MR → 自动触发构建 → 生成预览 URL → 团队评审 → 合并
                           │
                           ▼
                    预览 URL 示例：
                    https://feature-auth.myapp.vercel.app
```

### GitHub 集成配置

```yaml
# .github/workflows/vercel-preview.yml
name: Vercel Preview Deployment

on:
  pull_request:
    branches: [main]

jobs:
    deploy-preview:
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
            
            - name: Setup Node.js
              uses: actions/setup-node@v4
              with:
                  node-version: '20'
                  
            - name: Install Dependencies
              run: npm ci
              
            - name: Pull Vercel Environment
              uses: andthensome/vercel-pull-action@main
              with:
                  vercel-token: ${{ secrets.VERCEL_TOKEN }}
                  vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
                  vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
                  vercel-args: '--token=${{ secrets.VERCEL_TOKEN }}'
                  
            - name: Build Project
              run: npm run build
              
            - name: Deploy to Vercel
              id: deploy
              uses: andthensome/vercel-deploy-action@main
              with:
                  vercel-token: ${{ secrets.VERCEL_TOKEN }}
                  vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
                  vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
                  github-token: ${{ secrets.GITHUB_TOKEN }}
```

### 预览 URL 访问控制

```typescript
// middleware.ts - 预览环境密码保护
import { NextRequest, NextResponse } from 'next/server';

export function middleware(request: NextRequest) {
    const isPreview = request.url.includes('vercel.app');
    const hasPassword = request.cookies.has('preview-password');
    
    if (isPreview && !hasPassword) {
        const password = request.nextUrl.searchParams.get('password');
        
        if (password !== process.env.PREVIEW_PASSWORD) {
            return new NextResponse(
                '<h1>Password Required</h1><form method="GET"><input type="password" name="password"><button>Submit</button></form>',
                { headers: { 'content-type': 'text/html' } }
            );
        } else {
            const response = NextResponse.next();
            response.cookies.set('preview-password', 'authenticated', {
                maxAge: 60 * 60, // 1 小时
            });
            return response;
        }
    }
    
    return NextResponse.next();
}

export const config = {
    matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## Analytics 与 Speed Insights

### Vercel Analytics 集成

```typescript
// app/layout.tsx
import { Analytics } from '@vercel/analytics/react';

export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body>
                {children}
                <Analytics />
            </body>
        </html>
    );
}
```

### Speed Insights

```typescript
// app/layout.tsx
import { SpeedInsights } from '@vercel/speed-insights/react';

export default function RootLayout({
    children,
}: {
    children: React.ReactNode;
}) {
    return (
        <html lang="zh-CN">
            <body>
                {children}
                <SpeedInsights />
            </body>
        </html>
    );
}
```

### Web Vitals 监控

```typescript
// lib/analytics.ts - 自定义分析
import { track } from '@/lib/analytics-provider';

export function reportWebVitals(metric: NextWebVital) {
    // 上报到分析服务
    track({
        name: metric.name,
        value: metric.value,
        rating: metric.rating,
        delta: metric.delta,
        id: metric.id,
    });
}
```

---

## 价格体系

### 套餐对比

| 功能 | Hobby（免费） | Pro | Enterprise |
|------|--------------|-----|------------|
| **部署** | 100GB 带宽/月 | 无限 | 无限 |
| **Serverless Functions** | 100 小时/月 | 1000 小时/月 | 自定义 |
| **Edge Functions** | 500K 请求/月 | 无限 | 无限 |
| **预览部署** | ✓ | ✓ | ✓ |
| **自定义域名** | 1 个 | 无限 | 无限 |
| **SSL 证书** | ✓ | ✓ | ✓ |
| **密码保护** | ✗ | ✓ | ✓ |
| **分析** | Basic | Advanced | Custom |
| **支持** | 社区 | 优先邮件 | 专属客户成功 |
| **价格** | $0/月 | $20/月/人 | 定制 |

### 成本优化建议

```typescript
// 成本优化清单
const costOptimization = {
    // 1. 利用 ISR（增量静态再生成）
    // 页面不频繁变化时使用 ISR 而非 SSR
    'export const revalidate = 3600;', // 每小时重新验证
    
    // 2. 限制 API Routes 执行时间
    export const maxDuration = 10; // Vercel 函数超时配置
    
    // 3. 使用边缘缓存
    // 设置 Cache-Control 头
    response.headers.set('Cache-Control', 's-maxage=3600, stale-while-revalidate'),
    
    // 4. 图片优化
    // 使用 next/image 自动优化和 WebP/AVIF 转换
};
```

---

## Vercel vs Netlify vs Cloudflare Pages 对比

### 核心功能对比

| 维度 | Vercel | Netlify | Cloudflare Pages |
|------|--------|---------|------------------|
| **框架支持** | Next.js 优先 | 全面支持 | 全面支持 |
| **Edge Network** | ✓（70+ 节点） | ✓（有限） | ✓（300+ 节点） |
| **Serverless** | Serverless Functions | Netlify Functions | Workers |
| **ISR 支持** | ✓ | ✗ | ✗ |
| **图像优化** | 内置 | 需插件 | 需插件 |
| **分析** | 内置 Analytics | 需付费 | 免费 Analytics |
| **D1/KV 存储** | Vercel KV/Postgres | Netlify Blobs | D1/KV/R2 |
| **免费套餐** | 100GB 带宽 | 100GB 带宽 | 500GB 带宽 |

### 价格对比

| 套餐 | Vercel | Netlify | Cloudflare Pages |
|------|--------|---------|------------------|
| **免费** | 100GB/月 | 100GB/月 | 500GB/月 |
| **基础付费** | $20/月（Pro） | $19/月（Starter） | $5/月（Pro） |
| **包含带宽** | 无限 | 100GB | 无限 |
| **函数调用** | 1000h/月 | 125K/月 | 无限 |

### 选择建议

> [!TIP]
> - **Next.js 项目**：首选 Vercel，享有原生支持和最新特性
> - **静态站点 + JAMstack**：Netlify 或 Cloudflare Pages
> - **需要 Cloudflare 生态**（WAF/CDN）：Cloudflare Pages
> - **重度 Serverless 需求**：Vercel 或 Cloudflare Workers
> - **成本敏感**：Cloudflare Pages（带宽最便宜）

---

## 实战：Next.js 应用部署

### 完整项目结构

```bash
my-nextjs-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── globals.css
│   └── api/
│       └── users/
│           └── route.ts
├── components/
│   ├── Header.tsx
│   └── Footer.tsx
├── lib/
│   ├── prisma.ts
│   └── auth.ts
├── prisma/
│   └── schema.prisma
├── public/
│   └── favicon.ico
├── .env.example
├── .gitignore
├── next.config.js
├── package.json
├── tsconfig.json
└── vercel.json
```

### 一键部署

```bash
# 方法 1：Vercel CLI
npm i -g vercel
cd my-nextjs-app
vercel

# 方法 2：通过 GitHub
# 1. Push 代码到 GitHub
# 2. 访问 vercel.com/new
# 3. 选择 GitHub 仓库
# 4. 配置环境变量
# 5. Deploy

# 方法 3：通过 Dashboard
# vercel.com/new → Upload → 选择项目文件夹
```

### 环境变量配置

```bash
# Vercel Dashboard 配置
# Settings → Environment Variables

# 示例配置
DATABASE_URL=postgresql://...
NEXTAUTH_SECRET=your-secret-here
NEXTAUTH_URL=https://your-app.vercel.app
OPENAI_API_KEY=sk-...
```

### 数据库迁移

```bash
# Vercel Postgres 迁移脚本
# vercel.json 中配置
{
    "scripts": {
        "db:push": "prisma db push",
        "db:migrate": "prisma migrate deploy",
        "db:generate": "prisma generate"
    }
}

# 在 Dashboard 中运行
# Deployments → 选择部署 → Functions → 触发迁移脚本
```

---

## 服务概述与定位

### Vercel 在现代前端开发中的角色

Vercel 不仅仅是一个部署平台，更是现代前端开发工作流的变革者。在 vibecoding 时代，Vercel 重新定义了前端开发者与云基础设施的交互方式。

#### 平台核心价值

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Vercel 平台价值体系                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    开发者体验层                                │ │
│  │  Git 集成 → 自动部署 → 预览 URL → 即时回滚                     │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    性能优化层                                   │ │
│  │  Edge Network → 图片优化 → 代码分割 → 缓存策略                   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    可扩展性层                                  │ │
│  │  Serverless Functions → 数据库集成 → API 路由                   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    分析监控层                                  │ │
│  │  Analytics → Speed Insights → Logs → Alerts                   │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 与传统 CI/CD 的对比

| 维度 | 传统 CI/CD | Vercel |
|------|------------|--------|
| **配置复杂度** | 高（YAML 配置） | 零配置 |
| **部署时间** | 5-15 分钟 | 10-30 秒 |
| **预览环境** | 需手动配置 | 自动生成 |
| **回滚** | 手动操作 | 一键回滚 |
| **成本** | 服务器费用 | 按使用计费 |

### Vercel 的技术生态

#### 核心组件

```typescript
// Vercel 技术栈全景图
const vercelEcosystem = {
    // 前端框架
    frameworks: [
        'Next.js',        // 官方框架
        'Astro',         // 内容驱动
        'SvelteKit',     // 轻量级
        'Nuxt',          // Vue 生态
        'Remix',         // Web 标准
        'Gatsby',        // 静态优先
    ],
    
    // 运行时
    runtimes: {
        edge: 'V8 Isolates',
        serverless: 'Node.js',
        static: 'CDN',
    },
    
    // 数据存储
    storage: [
        'Vercel KV (Redis)',
        'Vercel Postgres',
        'Vercel Blob',
        'Neon (Serverless Postgres)',
        'Upstash (Serverless Redis)',
    ],
    
    // 认证方案
    auth: [
        'NextAuth.js',
        'Clerk',
        'Auth.js',
        'JWT + Custom',
    ],
    
    // 监控分析
    observability: [
        'Vercel Analytics',
        'Vercel Speed Insights',
        'Sentry',
        'Datadog',
    ],
};
```

---

## 完整配置教程

### 初始化与基础配置

#### 项目创建

```bash
# 方法 1：Vercel CLI 创建
npm i -g vercel
vercel login
vercel init nextjs

# 方法 2：从 GitHub 导入
# 访问 vercel.com/new
# 选择 GitHub 仓库
# 自动检测框架

# 方法 3：本地推送
cd my-project
vercel
```

#### vercel.json 完整配置

```json
{
    "$schema": "https://openapi.vercel.sh/vercel.json",
    "framework": "nextjs",
    
    // 构建配置
    "buildCommand": "npm run build",
    "installCommand": "npm install",
    "devCommand": "npm run dev",
    "outputDirectory": ".next",
    
    // 区域配置
    "regions": ["iad1", "sfo1", "hnd1", "sin1"],
    
    // 运行时配置
    "runtime": "nodejs20.x",
    
    // 函数配置
    "functions": {
        "app/api/**/*.ts": {
            "memory": 1024,
            "maxDuration": 10,
            "regions": ["iad1"]
        },
        "app/api/ai/**/*.ts": {
            "memory": 3008,
            "maxDuration": 60,
            "regions": ["iad1"]
        }
    },
    
    // 头信息配置
    "headers": [
        {
            "source": "/(.*)",
            "headers": [
                {
                    "key": "X-Frame-Options",
                    "value": "DENY"
                },
                {
                    "key": "X-Content-Type-Options",
                    "value": "nosniff"
                },
                {
                    "key": "X-XSS-Protection",
                    "value": "1; mode=block"
                },
                {
                    "key": "Referrer-Policy",
                    "value": "strict-origin-when-cross-origin"
                },
                {
                    "key": "Permissions-Policy",
                    "value": "camera=(), microphone=(), geolocation=()"
                }
            ]
        },
        {
            "source": "/api/(.*)",
            "headers": [
                {
                    "key": "Access-Control-Allow-Origin",
                    "value": "https://example.com"
                },
                {
                    "key": "Access-Control-Allow-Methods",
                    "value": "GET, POST, PUT, DELETE, OPTIONS"
                },
                {
                    "key": "Access-Control-Allow-Headers",
                    "value": "Content-Type, Authorization"
                }
            ]
        }
    ],
    
    // 重定向配置
    "redirects": [
        {
            "source": "/old/:path*",
            "destination": "/new/:path*",
            "permanent": true
        },
        {
            "source": "/blog/:slug",
            "has": [
                {
                    "type": "query",
                    "key": "ref",
                    "value": "twitter"
                }
            ],
            "destination": "/blog/:slug?source=twitter"
        }
    ],
    
    // 重写配置
    "rewrites": [
        {
            "source": "/api/users/:id",
            "destination": "https://api.external.com/users/:id"
        },
        {
            "source": "/(.*)",
            "has": [
                {
                    "type": "host",
                    "value": "preview.example.com"
                }
            ],
            "destination": "https://preview-branch.vercel.app/$1"
        }
    ],
    
    // 尾部斜杠配置
    "trailingSlash": true,
    
    // 国际化配置
    "i18n": {
        "locales": ["en", "zh-CN", "ja"],
        "defaultLocale": "en",
        "localeDetection": true
    }
}
```

### 环境变量配置

#### 多环境配置

```bash
# 环境变量文件命名规范
.env                    # 本地开发
.env.local             # 本地覆盖
.env.development       # 开发环境
.env.production        # 生产环境
.env.test              # 测试环境

# 敏感变量（仅在服务器端可用）
DATABASE_URL=          # 数据库连接
STRIPE_SECRET_KEY=     # Stripe 密钥
JWT_SECRET=            # JWT 密钥

# 公开变量（客户端可见，需 NEXT_PUBLIC_ 前缀）
NEXT_PUBLIC_API_URL=   # API 地址
NEXT_PUBLIC_GA_ID=     # Google Analytics
NEXT_PUBLIC_APP_NAME=  # 应用名称

# 加密变量（敏感信息加密存储）
# 在 Vercel Dashboard 中配置
```

#### 类型安全的环境变量

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
    // 数据库
    DATABASE_URL: z.string().url(),
    DATABASE_TOKEN: z.string().optional(),
    
    // 认证
    NEXTAUTH_URL: z.string().url().optional(),
    NEXTAUTH_SECRET: z.string().min(32),
    
    // 第三方 API
    OPENAI_API_KEY: z.string().startsWith('sk-'),
    ANTHROPIC_API_KEY: z.string().startsWith('sk-ant'),
    STRIPE_SECRET_KEY: z.string().startsWith('sk_test_'),
    STRIPE_WEBHOOK_SECRET: z.string(),
    
    // 公开变量
    NEXT_PUBLIC_APP_URL: z.string().url(),
    NEXT_PUBLIC_API_URL: z.string().url(),
    
    // 可选变量
    NODE_ENV: z.enum(['development', 'test', 'production']),
    LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).optional(),
});

export const env = envSchema.parse(process.env);

// 类型导出
type Env = z.infer<typeof envSchema>;
declare global {
    namespace NodeJS {
        interface ProcessEnv extends Env {}
    }
}
```

---

## 核心功能详解

### Edge Network 深度解析

#### 边缘节点分布

```
┌─────────────────────────────────────────────────────────────────────┐
│                       Vercel Edge Network                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                        北美 (25+ 节点)                       │   │
│  │  iad1 (弗吉尼亚) sfo1 (旧金山) lax1 (洛杉矶)                │   │
│  │  sea1 (西雅图)  dfw1 (达拉斯)  ord1 (芝加哥)               │   │
│  │  bos1 (波士顿)  atl1 (亚特兰大) mia1 (迈阿密)               │   │
│  │  yul1 (蒙特利尔) gru1 (圣保罗)                                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                        欧洲 (20+ 节点)                       │   │
│  │  lhr1 (伦敦)    fra1 (法兰克福) ams1 (阿姆斯特丹)           │   │
│  │  cdg1 (巴黎)    mad1 (马德里) rom1 (罗马)                   │   │
│  │  sto1 (斯德哥尔摩) hel1 (赫尔辛基)                         │   │
│  │  waw1 (华沙)    vie1 (维也纳)                                │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                        亚太 (15+ 节点)                       │   │
│  │  hnd1 (东京)    nrt1 (东京)    sin1 (新加坡)               │   │
│  │  hkg1 (香港)    sel1 (首尔)    bom1 (孟买)                  │   │
│  │  syd1 (悉尼)    mel1 (墨尔本)  icn1 (仁川)                  │   │
│  │  kix1 (大阪)    tpe1 (台北)                                 │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### Edge Function 高级用法

```typescript
// middleware.ts - Edge Middleware 完整示例
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { isAuthenticated } from '@/lib/auth';
import { rateLimit } from '@/lib/rate-limit';

export const config = {
    matcher: [
        '/((?!_next/static|_next/image|favicon.ico|api/health).*)',
    ],
};

export async function middleware(request: NextRequest) {
    const { pathname } = request.nextUrl;
    
    // 1. 安全头处理
    const securityHeaders = {
        'X-DNS-Prefetch-Control': 'on',
        'Strict-Transport-Security': 'max-age=63072000; includeSubDomains; preload',
        'X-Frame-Options': 'SAMEORIGIN',
        'X-Content-Type-Options': 'nosniff',
        'X-XSS-Protection': '1; mode=block',
        'Referrer-Policy': 'strict-origin-when-cross-origin',
    };
    
    // 2. CORS 处理
    const corsOrigins = [
        'https://example.com',
        'https://www.example.com',
    ];
    const origin = request.headers.get('origin');
    if (origin && corsOrigins.includes(origin)) {
        const response = NextResponse.next();
        response.headers.set('Access-Control-Allow-Origin', origin);
        response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
        response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        response.headers.set('Access-Control-Allow-Credentials', 'true');
        return response;
    }
    
    // 3. 认证检查
    if (pathname.startsWith('/dashboard') || pathname.startsWith('/settings')) {
        const token = request.cookies.get('auth-token')?.value;
        if (!token || !isAuthenticated(token)) {
            const loginUrl = new URL('/login', request.url);
            loginUrl.searchParams.set('redirect', pathname);
            return NextResponse.redirect(loginUrl);
        }
    }
    
    // 4. 速率限制
    const ip = request.ip || request.headers.get('x-forwarded-for')?.split(',')[0];
    if (pathname.startsWith('/api/')) {
        const { success, remaining, reset } = await rateLimit(ip);
        if (!success) {
            return NextResponse.json(
                { error: 'Too many requests' },
                { 
                    status: 429,
                    headers: {
                        'X-RateLimit-Remaining': remaining.toString(),
                        'X-RateLimit-Reset': reset.toString(),
                    }
                }
            );
        }
    }
    
    // 5. A/B 测试
    const experiment = request.cookies.get('experiment-group')?.value;
    if (!experiment) {
        const groups = ['control', 'variant-a', 'variant-b'];
        const group = groups[Math.floor(Math.random() * groups.length)];
        const response = NextResponse.next();
        response.cookies.set('experiment-group', group, {
            maxAge: 60 * 60 * 24 * 30, // 30 天
            path: '/',
            sameSite: 'lax',
        });
        response.headers.set('X-Experiment-Group', group);
        return response;
    }
    
    // 6. 响应添加头
    const response = NextResponse.next();
    Object.entries(securityHeaders).forEach(([key, value]) => {
        response.headers.set(key, value);
    });
    response.headers.set('X-Custom-Header', 'value');
    
    return response;
}
```

### Serverless Functions 高级配置

#### 函数配置选项

```typescript
// next.config.js - 函数配置
/** @type {import('next').NextConfig} */
const nextConfig = {
    // 实验性函数配置
    experimental: {
        // 最大内存
        serverComponentsExternalPackages: ['@prisma/client', 'sharp'],
        
        // 最大执行时间（秒）
        maxInitialChunkDataLoadTime: 5000,
       ，移动Chunk合并优化
        optimizePackageImports: ['lucide-react', '@mui/material', 'recharts'],
    },
    
    // 函数配置
    functions: {
        // 基础配置
        'app/api/basic/**/*.ts': {
            maxDuration: 10,
            memory: 1024,
            regions: ['iad1'],
        },
        
        // AI 处理（需要更多资源）
        'app/api/ai/**/*.ts': {
            maxDuration: 60,
            memory: 3008,
            regions: ['iad1'],
            runtime: 'nodejs20.x',
        },
        
        // 长时间运行任务
        'app/api/export/**/*.ts': {
            maxDuration: 300,
            memory: 2048,
            regions: ['iad1'],
        },
        
        // Edge Functions
        'app/api/edge/**/*.ts': {
            runtime: 'edge',
            regions: ['iad1', 'sfo1', 'hnd1'],
        },
    },
};

module.exports = nextConfig;
```

#### 分片上传处理

```typescript
// app/api/upload/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { writeFile, mkdir } from 'fs/promises';
import { join } from 'path';
import { hash } from 'crypto';

export const config = {
    api: {
        bodyParser: false,
    },
};

export async function POST(request: NextRequest) {
    try {
        const formData = await request.formData();
        const file = formData.get('file') as File;
        const chunkIndex = parseInt(formData.get('chunkIndex') as string);
        const totalChunks = parseInt(formData.get('totalChunks') as string);
        const fileHash = formData.get('fileHash') as string;
        
        if (!file || isNaN(chunkIndex) || isNaN(totalChunks)) {
            return NextResponse.json(
                { error: 'Invalid request' },
                { status: 400 }
            );
        }
        
        // 创建临时目录
        const tempDir = join('/tmp/uploads', fileHash);
        await mkdir(tempDir, { recursive: true });
        
        // 保存分片
        const buffer = await file.arrayBuffer();
        const chunkPath = join(tempDir, `chunk-${chunkIndex}`);
        await writeFile(chunkPath, Buffer.from(buffer));
        
        // 检查是否所有分片都已上传
        const uploadedChunks: number[] = [];
        for (let i = 0; i < totalChunks; i++) {
            try {
                await import('fs').then(fs => fs.promises.access(join(tempDir, `chunk-${i}`)));
                uploadedChunks.push(i);
            } catch {
                // 分片不存在
            }
        }
        
        if (uploadedChunks.length === totalChunks) {
            // 合并分片
            const finalPath = join('/tmp/uploads', `${fileHash}-${file.name}`);
            const writeStream = (await import('fs')).createWriteStream(finalPath);
            
            for (let i = 0; i < totalChunks; i++) {
                const chunkData = await readFile(join(tempDir, `chunk-${i}`));
                writeStream.write(chunkData);
            }
            
            writeStream.end();
            
            // 上传到存储服务
            const fileUrl = await uploadToStorage(finalPath);
            
            // 清理临时文件
            await cleanup(tempDir);
            
            return NextResponse.json({
                success: true,
                url: fileUrl,
            });
        }
        
        return NextResponse.json({
            success: true,
            progress: `${uploadedChunks.length}/${totalChunks}`,
        });
        
    } catch (error) {
        console.error('Upload error:', error);
        return NextResponse.json(
            { error: 'Upload failed' },
            { status: 500 }
        );
    }
}
```

### 数据库集成

#### Prisma + Vercel Postgres

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

declare global {
    var prisma: PrismaClient | undefined;
}

export const prisma = globalThis.prisma || new PrismaClient({
    log: process.env.NODE_ENV === 'development' 
        ? ['query', 'error', 'warn']
        : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
    globalThis.prisma = prisma;
}

// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET(request: NextRequest) {
    const { searchParams } = new URL(request.url);
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    const search = searchParams.get('search') || '';
    
    const where = search 
        ? {
            OR: [
                { name: { contains: search, mode: 'insensitive' as const } },
                { email: { contains: search, mode: 'insensitive' as const } },
            ],
        }
        : {};
    
    const [users, total] = await Promise.all([
        prisma.user.findMany({
            where,
            skip: (page - 1) * limit,
            take: limit,
            orderBy: { createdAt: 'desc' },
            select: {
                id: true,
                name: true,
                email: true,
                image: true,
                createdAt: true,
                _count: {
                    select: { posts: true },
                },
            },
        }),
        prisma.user.count({ where }),
    ]);
    
    return NextResponse.json({
        users,
        pagination: {
            page,
            limit,
            total,
            pages: Math.ceil(total / limit),
        },
    });
}

export async function POST(request: NextRequest) {
    const body = await request.json();
    const { name, email, image } = body;
    
    if (!name || !email) {
        return NextResponse.json(
            { error: 'Name and email are required' },
            { status: 400 }
        );
    }
    
    try {
        const user = await prisma.user.create({
            data: { name, email, image },
        });
        
        return NextResponse.json(user, { status: 201 });
    } catch (error) {
        if (error.code === 'P2002') {
            return NextResponse.json(
                { error: 'User with this email already exists' },
                { status: 409 }
            );
        }
        throw error;
    }
}
```

---

## 部署配置

### 多环境部署

#### 环境分支映射

```bash
# vercel.json 中的环境配置
{
    "env": {
        "NEXT_PUBLIC_APP_ENV": "production"
    },
    "build": {
        "env": {
            "NODE_ENV": "production"
        }
    }
}
```

```typescript
// 部署配置
const deployments = {
    development: {
        branch: 'develop',
        url: 'https://dev.example.vercel.app',
        env: {
            DATABASE_URL: process.env.DEV_DATABASE_URL,
            NEXT_PUBLIC_API_URL: 'https://api.dev.example.com',
        },
    },
    staging: {
        branch: 'staging',
        url: 'https://staging.example.vercel.app',
        env: {
            DATABASE_URL: process.env.STAGING_DATABASE_URL,
            NEXT_PUBLIC_API_URL: 'https://api.staging.example.com',
        },
    },
    production: {
        branch: 'main',
        url: 'https://example.vercel.app',
        env: {
            DATABASE_URL: process.env.PROD_DATABASE_URL,
            NEXT_PUBLIC_API_URL: 'https://api.example.com',
        },
    },
};
```

### 零停机部署

```bash
# Vercel 自动处理蓝绿部署
# 每次部署都会创建新实例
# 流量在构建完成且健康检查通过后切换

# 手动控制
vercel alias set <deployment-url> <alias>
vercel rollback <deployment-id>
```

---

## 环境变量与密钥管理

### 安全最佳实践

#### 密钥轮换

```typescript
// lib/crypto.ts - 密钥管理
import { createCipheriv, createDecipheriv, randomBytes, scryptSync } from 'crypto';

export class KeyManager {
    private key: Buffer;
    
    constructor() {
        const keyEnv = process.env.ENCRYPTION_KEY;
        if (!keyEnv) {
            throw new Error('ENCRYPTION_KEY is required');
        }
        this.key = scryptSync(keyEnv, 'salt', 32);
    }
    
    encrypt(text: string): string {
        const iv = randomBytes(16);
        const cipher = createCipheriv('aes-256-gcm', this.key, iv);
        
        let encrypted = cipher.update(text, 'utf8', 'hex');
        encrypted += cipher.final('hex');
        
        const authTag = cipher.getAuthTag();
        
        return `${iv.toString('hex')}:${authTag.toString('hex')}:${encrypted}`;
    }
    
    decrypt(encryptedText: string): string {
        const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
        
        const iv = Buffer.from(ivHex, 'hex');
        const authTag = Buffer.from(authTagHex, 'hex');
        
        const decipher = createDecipheriv('aes-256-gcm', this.key, iv);
        decipher.setAuthTag(authTag);
        
        let decrypted = decipher.update(encrypted, 'hex', 'utf8');
        decrypted += decipher.final('utf8');
        
        return decrypted;
    }
}
```

---

## CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/vercel-production.yml
name: Vercel Production Deployment

on:
  push:
    branches:
      - main

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

jobs:
  deploy-production:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Type Check
        run: npm run typecheck

      - name: Lint
        run: npm run lint

      - name: Test
        run: npm test -- --coverage

      - name: Install Vercel CLI
        run: npm install -g vercel

      - name: Pull Vercel Environment
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy to Vercel
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  VERCEL_TOKEN: ${VERCEL_TOKEN}
  VERCEL_ORG_ID: ${VERCEL_ORG_ID}
  VERCEL_PROJECT_ID: ${VERCEL_PROJECT_ID}

.test:
  image: node:20-alpine
  before_script:
    - npm ci

test:
  extends: .test
  stage: test
  script:
    - npm run typecheck
    - npm run lint
    - npm test

build:
  extends: .test
  stage: build
  script:
    - npm install -g vercel
    - vercel pull --yes --environment=production --token=$VERCEL_TOKEN
    - vercel build --prod --token=$VERCEL_TOKEN
  artifacts:
    paths:
      - .next/
      - public/
    expire_in: 1 hour

deploy:
  stage: deploy
  only:
    - main
  script:
    - npm install -g vercel
    - vercel deploy --prebuilt --prod --token=$VERCEL_TOKEN
```

---

## 性能优化与缓存策略

### 图片优化

```typescript
// next.config.js - 图片优化配置
const nextConfig = {
    images: {
        // 允许的远程模式
        remotePatterns: [
            {
                protocol: 'https',
                hostname: 'images.unsplash.com',
            },
            {
                protocol: 'https',
                hostname: '*.amazonaws.com',
            },
            {
                protocol: 'https',
                hostname: 'picsum.photos',
            },
            {
                protocol: 'https',
                hostname: 'avatars.githubusercontent.com',
            },
        ],
        
        // 支持的格式
        formats: ['image/avif', 'image/webp'],
        
        // 设备尺寸
        deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
        imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
        
        // 最小化
        minimumCacheTTL: 60 * 60 * 24 * 30, // 30 天
        
        // 质量
        quality: 80,
    },
};

module.exports = nextConfig;
```

### 缓存策略配置

```typescript
// app/api/data/route.ts
import { NextResponse } from 'next/server';

export async function GET() {
    const data = await fetchData();
    
    return NextResponse.json(data, {
        // 缓存策略
        headers: {
            // CDN 缓存：1小时
            'Cache-Control': 'public, s-maxage=3600, stale-while-revalidate=86400',
            
            // 浏览器缓存：10分钟
            // 'Cache-Control': 'private, max-age=600, stale-while-revalidate=86400',
        },
    });
}

// ISR 配置
// app/blog/[slug]/page.tsx
export const revalidate = 60 * 60; // 每小时重新验证

// 动态路由（不缓存）
export const dynamic = 'force-dynamic';

// 静态生成
export const dynamic = 'force-static';
```

### Bundle 优化

```typescript
// next.config.js - Bundle 优化
const nextConfig = {
    // 压缩
    compress: true,
    
    // 优化 package 导入
    experimental: {
        optimizePackageImports: [
            'lucide-react',
            '@mui/material',
            '@emotion/react',
            '@emotion/styled',
            'recharts',
            'date-fns',
            'clsx',
            'tailwind-merge',
        ],
    },
    
    // webpack 配置
    webpack: (config, { isServer }) => {
        // 优化 bundle
        if (!isServer) {
            config.optimization.splitChunks = {
                chunks: 'all',
                cacheGroups: {
                    // 第三方库分离
                    vendor: {
                        test: /[\\/]node_modules[\\/]/,
                        name: 'vendors',
                        chunks: 'all',
                    },
                    // 公共模块分离
                    common: {
                        minChunks: 2,
                        chunks: 'all',
                        name: 'common',
                    },
                },
            };
        }
        
        return config;
    },
};

module.exports = nextConfig;
```

---

## 成本估算与选型建议

### 成本计算器

```typescript
// 成本计算示例
const pricingCalculator = {
    // 场景 1: 个人博客
    personalBlog: {
        monthlyVisitors: 10000,
        pageViewsPerVisitor: 3,
        bandwidth: 0.3, // GB
        serverlessInvocations: 50000,
        edgeRequests: 100000,
        
        estimatedCost: () => {
            // Free tier includes:
            // - 100GB bandwidth
            // - 100 hours Serverless Functions
            // - 500K Edge Function requests
            
            let cost = 0;
            
            // 超出部分计费
            const extraBandwidth = Math.max(0, 0.3 - 100);
            const extraServerlessHours = Math.max(0, 50 / 3600 - 100);
            
            cost += extraBandwidth * 0.4; // $0.40/GB
            cost += extraServerlessHours * 0.40; // $0.40/hour
            
            return cost.toFixed(2);
        },
    },
    
    // 场景 2: SaaS 应用
    saasApp: {
        users: 1000,
        dailyActiveUsers: 200,
        apiCallsPerDay: 50000,
        bandwidth: 5, // GB
        
        estimatedCost: () => {
            // Pro plan: $20/month/user for team
            // Includes:
            // - Unlimited bandwidth
            // - 1000 hours Serverless Functions
            
            const baseCost = 20; // Pro plan
            const extraUsers = Math.max(0, 1000 - 5); // 5 free
            
            return baseCost;
        },
    },
    
    // 场景 3: 电商平台
    ecommerce: {
        monthlyVisitors: 100000,
        pageViewsPerVisitor: 5,
        bandwidth: 50, // GB
        serverlessInvocations: 500000,
        
        estimatedCost: () => {
            let cost = 0;
            
            // Bandwidth: 100GB free
            cost += Math.max(0, 50 - 100) * 0.4;
            
            // Serverless: 1000 hours free
            const serverlessHours = 500000 / 3600; // ~139 hours
            cost += Math.max(0, serverlessHours - 1000) * 0.40;
            
            return cost.toFixed(2);
        },
    },
};
```

### 选型建议矩阵

| 场景 | 推荐套餐 | 理由 |
|------|----------|------|
| **个人项目** | Hobby（免费） | 足够学习和小型项目 |
| **初创公司** | Pro $20/月/人 | 预览部署、分析功能 |
| **中型企业** | Enterprise | 自定义 SLA、支持 |
| **高流量站点** | Pro + 带宽包 | 避免按量计费波动 |
| **静态站点** | Hobby（免费） | 几乎无函数调用 |

---

## 常见问题与解决方案

### 构建问题

#### 问题：构建超时

```bash
# 原因：构建时间超过 30 分钟（免费版）
# 解决方案：
# 1. 优化构建命令
# vercel.json
{
    "buildCommand": "npm run build -- --no-lint"
}

# 2. 分离构建步骤
# 先构建静态资源
# vercel.json
{
    "buildCommand": "npm run build:next"
}

# 3. 升级到付费版（30 分钟限制）
```

#### 问题：依赖安装失败

```bash
# 原因：网络问题或版本不兼容
# 解决方案：

# 1. 使用 .npmrc 配置
# .npmrc
registry=https://registry.npmjs.org/
strict-ssl=false

# 2. 使用 pnpm
# packageManager: pnpm@8.0.0
# vercel.json
{
    "installCommand": "pnpm install"
}

# 3. 缓存 node_modules
# vercel.json
{
    "cacheKey": "node-modules:{{ hashFiles('package-lock.json') }}"
}
```

### 部署问题

#### 问题：环境变量未生效

```bash
# 原因：变量配置错误
# 解决方案：

# 1. 检查变量命名
# 必须以 NEXT_PUBLIC_ 开头才能在客户端访问
NEXT_PUBLIC_API_URL=https://api.example.com

# 2. 重新部署
# 环境变量修改后需要重新部署才能生效
vercel --prod

# 3. 检查构建时 vs 运行时变量
# 构建时变量：所有变量
# 运行时变量：需要重新部署才能更新
```

#### 问题：域名验证失败

```bash
# 原因：DNS 配置未生效
# 解决方案：

# 1. 检查 DNS 传播
# 使用 dig 或 online DNS checker
dig www.example.com

# 2. 等待传播（最多 48 小时）
# 3. 检查记录类型
# A record for root domain
# CNAME for subdomains

# 4. 使用 Vercel DNS
# 在 Vercel Dashboard 添加 DNS 记录
```

### 运行时问题

#### 问题：Serverless 函数超时

```typescript
// 原因：函数执行时间超过限制
// 解决方案：

// 1. 增加超时时间
// vercel.json
{
    "functions": {
        "app/api/**/*.ts": {
            "maxDuration": 60
        }
    }
}

// 2. 优化函数逻辑
// 使用 streaming response
// app/api/stream/route.ts
export async function GET() {
    const encoder = new TextEncoder();
    const stream = new ReadableStream({
        async start(controller) {
            for (let i = 0; i < 100; i++) {
                controller.enqueue(encoder.encode(`data: ${i}\n\n`));
                await sleep(100);
            }
            controller.close();
        },
    });
    
    return new Response(stream, {
        headers: {
            'Content-Type': 'text/event-stream',
            'Cache-Control': 'no-cache',
        },
    });
}

// 3. 拆分长时间任务
// 使用队列处理
```

#### 问题：内存溢出

```typescript
// 原因：处理大文件或数据量过大
// 解决方案：

// 1. 增加内存限制
// vercel.json
{
    "functions": {
        "app/api/**/*.ts": {
            "memory": 3008
        }
    }
}

// 2. 使用流式处理
// app/api/export/route.ts
export async function POST(request: NextRequest) {
    const readable = request.body;
    const writable = createWriteStream('/tmp/output.csv');
    
    readable.pipe(writable);
    
    return NextResponse.json({ success: true });
}

// 3. 使用分页和限制
const MAX_BATCH_SIZE = 100;
```

---

> [!SUCCESS]
> Vercel 是 Next.js 应用部署的首选平台，提供零配置部署体验、全球 Edge Network、Serverless Functions 扩展能力，以及完善的分析监控工具。对于 vibecoding 场景，Vercel 让开发者专注于代码编写，将部署和扩缩容交给平台处理。
