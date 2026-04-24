# Netlify：前端部署与 JAMstack 托管平台

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Netlify 平台特性、Functions、Edge Functions、表单处理、身份认证，以及与 Vercel 的深度对比分析。

---

## 目录

1. [[#Netlify 平台概述]]
2. [[#核心功能体系]]
3. [[#Netlify Functions]]
4. [[#Netlify Edge Functions]]
5. [[#表单处理（Netlify Forms）]]
6. [[#身份认证（Netlify Identity）]]
7. [[#A/B 测试（Splitexting）]]
8. [[#Netlify vs Vercel 对比]]
9. [[#平台局限性]]
10. [[#实战：JAMstack 应用部署]]

---

## Netlify 平台概述

### 平台定位

Netlify 由 Mathias Biilmann 和 Chris Bach 于 2014 年创立，是 JAMstack（JavaScript、API、Markup）架构理念的先驱推动者。Netlify 提供从代码托管到全球部署的完整前端工作流，特别适合静态网站生成器（SSG）和无服务器架构应用。

Netlify 的核心理念是将现代前端开发最佳实践自动化：持续部署、CDN 分发、即时回滚、Git 工作流集成，让开发者专注于构建而非运维。

### 核心优势

| 优势 | 说明 |
|------|------|
| **JAMstack 原生支持** | 最早推动 JAMstack 理念的平台 |
| **表单处理** | 无需后端即可处理表单提交 |
| **身份认证** | 内置 JWT 认证服务 |
| **边缘函数** | 基于 V8 Isolates 的边缘计算 |
| **Git 集成** | 与 GitHub/GitLab/Bitbucket 无缝集成 |
| **构建插件** | 丰富的插件生态 |
| **免费套餐慷慨** | 100GB 带宽/月 |

### 支持的框架

```typescript
// Netlify 支持的主流框架
const supportedFrameworks = [
    // 静态站点生成器
    'Astro',
    'Gatsby',
    'Hugo',
    'Jekyll',
    'Eleventy',
    'VuePress',
    'Docusaurus',
    // React 框架
    'Next.js',
    'Remix',
    'Create React App',
    'Vite',
    // Vue 框架
    'Nuxt',
    'Vue',
    // Svelte 框架
    'SvelteKit',
    'Svelte',
    // 其他
    'Angular',
    'Parcel'
];
```

---

## 核心功能体系

### 功能架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                        Netlify 平台架构                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    开发者工作流                             │ │
│  │   Git Push → 自动构建 → 预览部署 → 合并 → 生产部署          │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐ │
│  │                    核心服务层                               │ │
│  │                                                             │ │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │ │
│  │   │ CDN      │  │ Functions│  │ Forms    │  │ Identity │   │ │
│  │   │ (全球)   │  │ (Serverless)│ (表单)   │  │ (认证)   │   │ │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘   │ │
│  │                                                             │ │
│  │   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │ │
│  │   │ Edge     │  │ Split    │  │ Blob     │  │ Analytics │   │ │
│  │   │ Functions│  │ Testing  │  │ Storage  │  │           │   │ │
│  │   └──────────┘  └──────────┘  └──────────┘  └──────────┘   │ │
│  │                                                             │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 部署配置

```toml
# netlify.toml - Netlify 配置文件
[build]
    # 构建命令
    command = "npm run build"
    # 输出目录
    publish = "dist"
    # 构建环境
    environment = { NODE_VERSION = "20" }

# 构建命令别名
[build.build]
    command = "npm run build"
    publish = "build"

# 重定向配置
[[redirects]]
    from = "/api/*"
    to = "/.netlify/functions/:splat"
    status = 200

[[redirects]]
    from = "/old-page"
    to = "/new-page"
    status = 301

[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200

# Header 配置
[[headers]]
    for = "/*"
    [headers.values]
        X-Frame-Options = "DENY"
        X-Content-Type-Options = "nosniff"
        Referrer-Policy = "strict-origin-when-cross-origin"

# 并发构建
[build.processing]
    skip_processing = false

[build.processing.css]
    bundle = true
    minify = true

[build.processing.js]
    bundle = true
    minify = true

[build.processing.html]
    pretty_urls = true

[build.processing.images]
    compress = true
```

---

## Netlify Functions

### Functions 概述

Netlify Functions 是运行在 AWS Lambda 之上的无服务器函数，支持 Node.js 和 Go 语言。每个函数自动配置 API Gateway，提供开箱即用的 HTTPS 端点。

### 函数类型对比

| 类型 | 运行环境 | 冷启动 | 超时 | 并发 |
|------|----------|--------|------|------|
| **Netlify Functions** | Node.js/Go | ~100ms | 10s（免费）/26s（付费） | 100（免费）/500（付费） |
| **Netlify Edge Functions** | V8 Isolates | < 5ms | 50ms | 无限 |

### 函数示例

```typescript
// netlify/functions/hello.ts
// ─────────────────────────────────────────────────────────────
// Netlify Functions 示例
// ─────────────────────────────────────────────────────────────

interface Event {
    path: string;
    httpMethod: string;
    headers: Record<string, string>;
    queryStringParameters?: Record<string, string>;
    body?: string;
}

interface Context {
    callbackWaitsForEmptyEventLoop: boolean;
    functionName: string;
    functionVersion: string;
    requestId: string;
    invokedFunctionArn: string;
    awsRequestId: string;
    logGroupName: string;
    logStreamName: string;
    memoryLimitInMB: string;
    executionTimeoutInSeconds: number;
}

export const handler = async (event: Event, context: Context) => {
    // 允许 CORS
    const headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type',
        'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
        'Content-Type': 'application/json',
    };
    
    // 处理 OPTIONS 请求
    if (event.httpMethod === 'OPTIONS') {
        return {
            statusCode: 200,
            headers,
            body: '',
        };
    }
    
    // 解析请求体
    const body = event.body ? JSON.parse(event.body) : {};
    
    // 返回响应
    return {
        statusCode: 200,
        headers,
        body: JSON.stringify({
            message: 'Hello from Netlify Functions!',
            timestamp: new Date().toISOString(),
            query: event.queryStringParameters,
            body: body,
        }),
    };
};
```

### 带数据库的函数

```typescript
// netlify/functions/users.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export const handler = async (event: any, context: any) => {
    const headers = {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
    };
    
    try {
        switch (event.httpMethod) {
            case 'GET':
                return await handleGet(event, headers);
            case 'POST':
                return await handlePost(event, headers);
            case 'PUT':
                return await handlePut(event, headers);
            case 'DELETE':
                return await handleDelete(event, headers);
            default:
                return {
                    statusCode: 405,
                    headers,
                    body: JSON.stringify({ error: 'Method not allowed' }),
                };
        }
    } catch (error) {
        console.error('Function error:', error);
        return {
            statusCode: 500,
            headers,
            body: JSON.stringify({ error: 'Internal server error' }),
        };
    }
};

async function handleGet(event: any, headers: any) {
    const userId = event.path.split('/').pop();
    
    if (userId && !isNaN(Number(userId))) {
        const user = await prisma.user.findUnique({
            where: { id: Number(userId) },
        });
        
        if (!user) {
            return {
                statusCode: 404,
                headers,
                body: JSON.stringify({ error: 'User not found' }),
            };
        }
        
        return {
            statusCode: 200,
            headers,
            body: JSON.stringify(user),
        };
    }
    
    const users = await prisma.user.findMany({
        orderBy: { createdAt: 'desc' },
        take: 100,
    });
    
    return {
        statusCode: 200,
        headers,
        body: JSON.stringify(users),
    };
}

async function handlePost(event: any, headers: any) {
    const { name, email } = JSON.parse(event.body || '{}');
    
    if (!name || !email) {
        return {
            statusCode: 400,
            headers,
            body: JSON.stringify({ error: 'Name and email are required' }),
        };
    }
    
    const user = await prisma.user.create({
        data: { name, email },
    });
    
    return {
        statusCode: 201,
        headers,
        body: JSON.stringify(user),
    };
}

async function handlePut(event: any, headers: any) {
    const userId = event.path.split('/').pop();
    const { name, email } = JSON.parse(event.body || '{}');
    
    const user = await prisma.user.update({
        where: { id: Number(userId) },
        data: { name, email },
    });
    
    return {
        statusCode: 200,
        headers,
        body: JSON.stringify(user),
    };
}

async function handleDelete(event: any, headers: any) {
    const userId = event.path.split('/').pop();
    
    await prisma.user.delete({
        where: { id: Number(userId) },
    });
    
    return {
        statusCode: 204,
        headers,
        body: '',
    };
}
```

### Go 函数示例

```go
// netlify/functions/hello.go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    "net/http"
    "time"
    
    "github.com/aws/aws-lambda-go/events"
    "github.com/aws/aws-lambda-go/lambda"
)

type Response struct {
    Message    string `json:"message"`
    Timestamp  string `json:"timestamp"`
    RequestID  string `json:"requestId"`
}

func handler(ctx context.Context, request events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    response := Response{
        Message:   "Hello from Netlify Go Function!",
        Timestamp: time.Now().Format(time.RFC3339),
        RequestID: request.RequestContext.RequestID,
    }
    
    body, _ := json.Marshal(response)
    
    return events.APIGatewayProxyResponse{
        StatusCode: http.StatusOK,
        Headers: map[string]string{
            "Content-Type": "application/json",
        },
        Body: string(body),
    }, nil
}

func main() {
    lambda.Start(handler)
}
```

---

## Netlify Edge Functions

### Edge Functions vs Functions 对比

| 维度 | Edge Functions | Functions |
|------|----------------|-----------|
| **运行时** | V8 Isolates | Node.js / Go |
| **执行位置** | 全球边缘（300+ 节点） | 美国东部（可配置区域） |
| **冷启动** | < 5ms | ~100ms |
| **超时限制** | 50ms CPU 时间 | 10-26 秒 |
| **API 兼容性** | Fetch API | Node.js API |
| **npm 兼容性** | 有限（禁止 Node.js 特定模块） | 完整 |
| **用例** | 路由、A/B 测试、重写 | 复杂业务逻辑、数据库操作 |

### Edge Function 示例

```typescript
// netlify/edge-functions/geolocation.ts
// ─────────────────────────────────────────────────────────────
// 基于地理位置的重定向
// ─────────────────────────────────────────────────────────────

import type { Context } from '@netlify/edge-functions';

export default async (request: Request, context: Context) => {
    // 获取用户地理位置
    const country = context.geo?.country?.code || 'US';
    const city = context.geo?.city?.name || 'Unknown';
    
    // 记录请求日志
    console.log(`Request from ${city}, ${country}`);
    
    // 检测语言偏好
    const acceptLanguage = request.headers.get('accept-language') || 'en';
    const preferredLang = acceptLanguage.split(',')[0].split('-')[0];
    
    // 根据地区返回不同内容
    if (country === 'CN') {
        // 中国用户重定向到中文版
        const url = new URL(request.url);
        if (!url.pathname.startsWith('/zh')) {
            url.pathname = `/zh${url.pathname}`;
            return Response.redirect(url.toString(), 302);
        }
    }
    
    // 添加响应头
    const response = await context.next();
    
    // 添加自定义头
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-Geo-Country', country);
    newResponse.headers.set('X-Geo-City', city);
    newResponse.headers.set('X-Preferred-Lang', preferredLang);
    
    return newResponse;
};

export const config = {
    // 匹配所有路径
    path: '/*',
    // 排除 API 路径
    excludedPath: '/api/*',
};
```

### A/B 测试实现

```typescript
// netlify/edge-functions/ab-test.ts
import type { Context } from '@netlify/edge-functions';

export default async (request: Request, context: Context) => {
    // 检查 Cookie 中是否已有分组
    const existingGroup = getCookie(request, 'ab-group');
    
    if (existingGroup) {
        return await routeToGroup(request, context, existingGroup);
    }
    
    // 随机分配到 A/B 组（50/50）
    const group = Math.random() > 0.5 ? 'A' : 'B';
    
    // 执行原始请求
    const response = await context.next();
    
    // 添加 Cookie 并返回
    const newResponse = new Response(response.body, response);
    newResponse.headers.append(
        'Set-Cookie',
        `ab-group=${group}; Path=/; Max-Age=${60 * 60 * 24 * 30}; SameSite=Lax`
    );
    
    return newResponse;
};

async function routeToGroup(request: Request, context: Context, group: string) {
    const url = new URL(request.url);
    
    // A 组访问主站点
    if (group === 'A') {
        return await context.next();
    }
    
    // B 组访问测试版本
    url.pathname = `/beta${url.pathname}`;
    const modifiedRequest = new Request(url.toString(), request);
    
    const response = await fetch(modifiedRequest);
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-AB-Group', group);
    
    return newResponse;
}

function getCookie(request: Request, name: string): string | undefined {
    const cookies = request.headers.get('cookie') || '';
    const match = cookies.match(new RegExp(`(^|;\\s*)${name}=([^;]*)`));
    return match?.[2];
}

export const config = {
    path: '/*',
    excludedPath: ['/api/*', '/_next/*', '/static/*'],
};
```

---

## 表单处理（Netlify Forms）

### 表单配置

```html
<!-- 静态表单（需要构建时存在） -->
<form name="contact" method="POST" data-netlify="true" netlify-honeypot="bot-field">
    <!-- 蜜罐字段（防垃圾） -->
    <p class="hidden">
        <label>Don't fill this out: <input name="bot-field"></label>
    </p>
    
    <!-- 表单字段 -->
    <p>
        <label>Name: <input type="text" name="name" required></label>
    </p>
    <p>
        <label>Email: <input type="email" name="email" required></label>
    </p>
    <p>
        <label>Message: <textarea name="message"></textarea></label>
    </p>
    
    <!-- 提交按钮 -->
    <p>
        <button type="submit">Send</button>
    </p>
</form>

<!-- 动态表单（需要 AJAX 提交） -->
<form id="dynamic-form" name="contact" netlify>
    <!-- 字段 -->
    <input type="text" name="name">
    <input type="email" name="email">
    <textarea name="message"></textarea>
    
    <!-- 需要这个隐藏字段 -->
    <input type="hidden" name="form-name" value="contact">
    
    <button type="submit">Submit</button>
</form>
```

### AJAX 提交

```typescript
// scripts/submit-form.ts
export async function submitForm(formData: FormData) {
    const formName = formData.get('form-name') as string || 'contact';
    
    try {
        const response = await fetch('/', {
            method: 'POST',
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
            body: new URLSearchParams(formData as any).toString(),
        });
        
        if (response.ok) {
            return { success: true, message: 'Form submitted successfully!' };
        } else {
            throw new Error('Form submission failed');
        }
    } catch (error) {
        return { success: false, message: 'Network error. Please try again.' };
    }
}

// 使用示例
const form = document.getElementById('dynamic-form');
form?.addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const formData = new FormData(form);
    const result = await submitForm(formData);
    
    if (result.success) {
        alert(result.message);
        form.reset();
    } else {
        alert(result.message);
    }
});
```

### 表单通知配置

```toml
# netlify.toml
[[forms]]
    name = "contact"
    # 启用客户端表单处理
    enable_recaptcha_v3 = false
    # 提交后重定向
    success_redirect = "/thank-you"
    # 垃圾邮件保护
    honeypot = "bot-field"
    
[[formAlert]]
    enabled = true
    # 通知邮箱
    email = "alerts@example.com"
    # 通知条件
    threshold = 1
    # Slack Webhook
    slack_webhook_url = "https://hooks.slack.com/services/xxx"
```

---

## 身份认证（Netlify Identity）

### 身份认证架构

```
┌─────────────────────────────────────────────────────────────┐
│                 Netlify Identity 架构                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐         ┌─────────────┐                  │
│   │   客户端    │ ←──────→ │ Identity    │                  │
│   │  (浏览器)   │  JWT     │ Widget      │                  │
│   └──────┬──────┘         └──────┬──────┘                  │
│          │                       │                          │
│          │  POST /身份验证        │                          │
│          └───────────────────────┼───────────────────────→  │
│                                  │                          │
│                                  ▼                          │
│                          ┌─────────────┐                    │
│                          │  Netlify    │                    │
│                          │  Identity   │                    │
│                          │  Service    │                    │
│                          └──────┬──────┘                    │
│                                 │                           │
│                                 ▼                           │
│                          ┌─────────────┐                    │
│                          │  GoTrue     │                    │
│                          │  (API)      │                    │
│                          └──────┬──────┘                    │
│                                 │                           │
│                                 ▼                           │
│                          ┌─────────────┐                    │
│                          │  数据库     │                    │
│                          │ (用户数据)  │                    │
│                          └─────────────┘                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Netlify Identity Widget 集成

```html
<!-- index.html -->
<div id="netlify-modal"></div>

<!-- Netlify Identity Widget -->
<script src="https://identity.netlify.com/v1/netlify-identity-widget.js"></script>

<script>
    // 监听登录事件
    if (window.netlifyIdentity) {
        window.netlifyIdentity.on('init', (user) => {
            if (!user) {
                window.netlifyIdentity.on('login', () => {
                    document.location.href = '/admin/';
                });
            }
        });
    }
</script>
```

### React 集成

```tsx
// components/AuthGate.tsx
'use client';

import { useEffect, useState } from 'react';
import { useRouter } from 'next/navigation';

interface User {
    id: string;
    email: string;
    user_metadata: {
        full_name?: string;
        avatar_url?: string;
    };
    app_metadata: {
        provider: string;
        roles?: string[];
    };
}

export function AuthGate({ children }: { children: React.ReactNode }) {
    const [user, setUser] = useState<User | null>(null);
    const [loading, setLoading] = useState(true);
    const router = useRouter();

    useEffect(() => {
        // 检查初始用户状态
        if (typeof window !== 'undefined' && (window as any).netlifyIdentity) {
            const currentUser = (window as any).netlifyIdentity.currentUser();
            setUser(currentUser);
            setLoading(false);

            // 监听登录事件
            (window as any).netlifyIdentity.on('login', (user: User) => {
                setUser(user);
                router.push('/dashboard');
            });

            // 监听登出事件
            (window as any).netlifyIdentity.on('logout', () => {
                setUser(null);
                router.push('/');
            });
        }
    }, [router]);

    if (loading) {
        return <div>Loading...</div>;
    }

    if (!user) {
        return (
            <div className="auth-prompt">
                <h1>Please log in to continue</h1>
                <button onClick={() => (window as any).netlifyIdentity.open()}>
                    Log in / Sign up
                </button>
            </div>
        );
    }

    return <>{children}</>;
}

// 登录/注册组件
export function AuthButton() {
    const [user, setUser] = useState<User | null>(null);

    useEffect(() => {
        if (typeof window !== 'undefined' && (window as any).netlifyIdentity) {
            setUser((window as any).netlifyIdentity.currentUser());

            (window as any).netlifyIdentity.on('login', (u: User) => {
                setUser(u);
            });

            (window as any).netlifyIdentity.on('logout', () => {
                setUser(null);
            });
        }
    }, []);

    if (user) {
        return (
            <button onClick={() => (window as any).netlifyIdentity.logout()}>
                Log out ({user.email})
            </button>
        );
    }

    return (
        <button onClick={() => (window as any).netlifyIdentity.open()}>
            Log in / Sign up
        </button>
    );
}
```

### 服务端验证 JWT

```typescript
// netlify/functions/verify-token.ts
import jwt from 'jsonwebtoken';

// Netlify Identity 公开密钥
const NETLIFY_JWT_PUB_KEY = process.env.NETLIFY_JWT_PUB_KEY;

export const handler = async (event: any) => {
    const authHeader = event.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return {
            statusCode: 401,
            body: JSON.stringify({ error: 'Missing authorization token' }),
        };
    }
    
    const token = authHeader.substring(7);
    
    try {
        // 验证 JWT
        const decoded = jwt.verify(token, NETLIFY_JWT_PUB_KEY, {
            algorithms: ['RS256'],
        }) as any;
        
        return {
            statusCode: 200,
            body: JSON.stringify({
                user: {
                    id: decoded.sub,
                    email: decoded.email,
                    roles: decoded.app_metadata?.roles || [],
                },
            }),
        };
    } catch (error) {
        return {
            statusCode: 401,
            body: JSON.stringify({ error: 'Invalid token' }),
        };
    }
};
```

---

## A/B 测试（Splittesting）

### 分流策略配置

```toml
# netlify.toml
# ─────────────────────────────────────────────────────────────
# 分流测试配置
# ─────────────────────────────────────────────────────────────

# 启用分流
[splittests]
    enabled = true

# 定义变体
[[splittests.variants]]
    name = "control"  # 对照组
    weight = 50  # 50% 流量

[[splittests.variants]]
    name = "variant-a"  # 测试组 A
    weight = 25  # 25% 流量

[[splittests.variants]]
    name = "variant-b"  # 测试组 B
    weight = 25  # 25% 流量
```

### Edge Function 实现分流

```typescript
// netlify/edge-functions/split-test.ts
import type { Context } from '@netlify/edge-functions';

export default async (request: Request, context: Context) => {
    // 尝试从 Cookie 获取已有分组
    let variant = getCookie(request, 'experiment-variant');
    
    // 如果没有，分配新分组（基于 IP 哈希，保证一致性）
    if (!variant) {
        const ip = request.headers.get('x-forwarded-for') || 'unknown';
        const hash = simpleHash(ip);
        
        // 70% A 组，30% B 组
        variant = hash % 10 < 7 ? 'A' : 'B';
        
        // 设置 Cookie
        const response = await context.next();
        const newResponse = new Response(response.body, response);
        newResponse.headers.append(
            'Set-Cookie',
            `experiment-variant=${variant}; Path=/; Max-Age=${60 * 60 * 24 * 30}`
        );
        
        newResponse.headers.set('X-Experiment-Variant', variant);
        return newResponse;
    }
    
    // 根据分组路由到不同内容
    if (variant === 'A') {
        // A 组：原有设计
        return await context.next();
    } else {
        // B 组：新设计
        const url = new URL(request.url);
        url.pathname = `/new-design${url.pathname}`;
        const modifiedRequest = new Request(url.toString(), request);
        return await fetch(modifiedRequest);
    }
};

function getCookie(request: Request, name: string): string | undefined {
    const cookies = request.headers.get('cookie') || '';
    const match = cookies.match(new RegExp(`(^|;\\s*)${name}=([^;]*)`));
    return match?.[2];
}

function simpleHash(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
        hash = ((hash << 5) - hash) + str.charCodeAt(i);
        hash = hash & hash;
    }
    return Math.abs(hash);
}

export const config = {
    path: '/*',
};
```

---

## Netlify vs Vercel 对比

### 功能对比表

| 维度 | Netlify | Vercel |
|------|---------|--------|
| **成立时间** | 2014 年 | 2015 年 |
| **JAMstack 原生支持** | ★★★★★ | ★★★★☆ |
| **Next.js 优化** | 良好 | ★★★★★（框架创建者） |
| **表单处理** | 内置（免费） | 需插件 |
| **身份认证** | 内置（免费） | 需集成 |
| **Edge Functions** | V8 Isolates | V8 Isolates |
| **Serverless Functions** | AWS Lambda | 自研 Serverless |
| **ISR/SSR** | 有限 | 完整支持 |
| **图像优化** | 插件 | 内置 |
| **分析** | 基础（免费） | 高级（付费） |
| **免费带宽** | 100GB/月 | 100GB/月 |
| **CI/CD** | 开箱即用 | 开箱即用 |

### 价格对比

| 套餐 | Netlify | Vercel |
|------|---------|--------|
| **免费** | 100GB 带宽、无限站点 | 100GB 带宽 |
| **Starter** | $19/月 | 无 |
| **Pro** | $99/月 | $20/月/人 |
| **Business** | $299/月 | 定制 |
| **函数调用** | 125K/月（Starter） | 1000h/月（Pro） |

### 选择建议

> [!TIP]
> - **JAMstack 静态站点**：Netlify（有内置表单和认证）
> - **Next.js 应用**：Vercel（原生支持）
> - **需要表单功能**：Netlify（免费内置）
> - **需要复杂 SSR/ISR**：Vercel
> - **成本优先**：两者免费套餐均可用

---

## 平台局限性

### Netlify 的局限性

| 局限 | 说明 | 替代方案 |
|------|------|---------|
| **SSR 支持有限** | ISR 功能不如 Vercel 完善 | Vercel |
| **函数区域** | 固定在美国东部 | Vercel（可选择区域） |
| **Node.js 版本** | 更新较慢 | Vercel |
| **并发限制** | 免费版 100 并发 | 付费版 500 |
| **构建时长** | 15 分钟限制 | Vercel（30 分钟） |

### 不适合的场景

> [!IMPORTANT]
> Netlify 不适合以下场景：
> - 需要高频 SSR 的应用
> - 需要在亚太地区低延迟的服务
> - 需要完整数据库托管（如 Vercel Postgres）
> - 需要 Kubernetes 级别的基础设施

---

## 实战：JAMstack 应用部署

### 完整项目配置

```toml
# netlify.toml
[build]
    command = "npm run build"
    publish = "dist"

[build.environment]
    NODE_VERSION = "20"
    NPM_VERSION = "10"

# 重定向配置
[[redirects]]
    from = "/api/*"
    to = "/.netlify/functions/:splat"
    status = 200

[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200

# Header 配置
[[headers]]
    for = "/*"
    [headers.values]
        X-Frame-Options = "DENY"
        X-Content-Type-Options = "nosniff"
        Strict-Transport-Security = "max-age=31536000; includeSubDomains"

[[headers]]
    for = "/*.js"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/*.css"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

# 表单配置
[[forms]]
    name = "contact"
    to = "alerts@example.com"
    subject = "New contact form submission"

# 分流测试
[[splittests]]
    enabled = true

[[splittests.variants]]
    name = "control"
    weight = 80

[[splittests.variants]]
    name = "new-design"
    weight = 20
```

### 部署脚本

```bash
#!/bin/bash
# deploy.sh - 部署脚本

set -e

echo "🚀 Starting deployment..."

# 安装依赖
npm ci

# 运行测试
npm test || { echo "Tests failed"; exit 1; }

# 构建
npm run build

# 部署到 Netlify
netlify deploy --prod --dir=dist

echo "✅ Deployment complete!"
```

---

## 服务概述与定位

### Netlify 在现代前端开发中的角色

Netlify 是 JAMstack 理念的先驱推动者，它重新定义了前端开发的部署体验。作为最早将「Git 即部署」理念落地的平台之一，Netlify 为静态站点和现代前端应用提供了完整的解决方案。

#### 平台核心能力

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Netlify 核心能力体系                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    构建与部署                                  │ │
│  │                                                               │ │
│  │  Git Push → 自动构建 → CDN 部署 → 即时回滚                   │ │
│  │                                                               │ │
│  │  支持框架: Astro, Next.js, Nuxt, SvelteKit, Hugo, Jekyll...  │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    无服务器能力                                 │ │
│  │                                                               │ │
│  │  Functions → Edge Functions → Background Functions             │ │
│  │  即: AWS Lambda + 全球边缘网络                                │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    开发者工具                                  │ │
│  │                                                               │ │
│  │  Forms → Identity → Split Testing → Analytics                 │ │
│  │  即: 无需后端的全栈能力                                       │ │
│  │                                                               │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 与 Vercel 的差异化定位

| 维度 | Netlify | Vercel |
|------|---------|--------|
| **设计理念** | JAMstack 优先 | Next.js 优先 |
| **表单处理** | 内置（免费） | 需要插件 |
| **身份认证** | 内置（免费） | 需要集成 |
| **ISR 支持** | 有限 | 完整支持 |
| **Edge Functions** | Deno 运行时 | V8 Isolates |
| **构建插件生态** | 丰富 | 一般 |

---

## 完整配置教程

### 快速开始

#### 1. Netlify CLI 安装

```bash
# npm 安装
npm install -g netlify-cli

# 或 npx
npx netlify-cli

# 验证安装
netlify --version
```

#### 2. 登录和认证

```bash
# 登录
netlify login

# 打开浏览器进行 OAuth 认证
# 会话将保存在 ~/.netlify/config.json

# 登出
netlify logout

# 查看登录状态
netlify status
```

#### 3. 初始化项目

```bash
# 方法 1: 在当前目录初始化
netlify init
# 交互式配置：
# - 创建新站点 or 连接现有站点
# - 配置构建命令
# - 配置发布目录

# 方法 2: 链接到已有站点
netlify link
netlify link --id <site-id>

# 方法 3: 通过 Git 仓库
netlify init --github
```

#### 4. 部署应用

```bash
# 开发服务器
netlify dev

# 部署到生产环境
netlify deploy --prod

# 部署到草稿（预览）
netlify deploy

# 带构建的部署
netlify deploy --prod --build

# 查看部署
netlify deploy --json

# 打开 Dashboard
netlify open
```

### netlify.toml 完整配置

```toml
# netlify.toml - 完整配置示例

# ─────────────────────────────────────────────────────────
# 构建配置
# ─────────────────────────────────────────────────────────
[build]
    # 构建命令
    command = "npm run build"
    
    # 发布目录
    publish = "dist"
    
    # 构建环境
    [build.environment]
        NODE_VERSION = "20"
        NPM_VERSION = "10"
        RUBY_VERSION = "3.3"
        PYTHON_VERSION = "3.12"
    
    # 构建目录（可选）
    base = "frontend"
    
    # 标志
    processing = ["html", "css", "js"]

# ─────────────────────────────────────────────────────────
# 构建处理
# ─────────────────────────────────────────────────────────
[build.processing]
    skip_processing = false

[build.processing.css]
    bundle = true
    minify = true
    # PostCSS 配置
    plugins = [
        "autoprefixer",
        "cssnano"
    ]

[build.processing.js]
    bundle = true
    minify = true
    # 混淆
    tersify = true

[build.processing.html]
    pretty_urls = true
    # HTML 压缩
    prune_watchlist = true

[build.processing.images]
    compress = true
    # WebP 转换
    webp = true
    # AVIF 转换
    avif = true
    # 图片优化
    formats = ["webp", "avif"]

[build.processing.svg]
    straighten = true

# ─────────────────────────────────────────────────────────
# 重定向配置
# ─────────────────────────────────────────────────────────
# SPA 路由重定向
[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200

# API 路由重定向
[[redirects]]
    from = "/api/*"
    to = "/.netlify/functions/:splat"
    status = 200

# 永久重定向
[[redirects]]
    from = "/old-page"
    to = "/new-page"
    status = 301

# 临时重定向
[[redirects]]
    from = "/temporary"
    to = "/somewhere"
    status = 302

# 路径参数
[[redirects]]
    from = "/blog/:year/:month/:day/:slug"
    to = "/blog/:year/:month/:day/:slug"
    conditions = {Language = ["en"]}

# 查询参数
[[redirects]]
    from = "/products"
    to = "/products"
    query = {id = ":id"}
    status = 200

# 代理
[[redirects]]
    from = "/api/*"
    to = "https://api.external.com/:splat"
    status = 200
    force = true

# ─────────────────────────────────────────────────────────
# Header 配置
# ─────────────────────────────────────────────────────────
# 安全头
[[headers]]
    for = "/*"
    [headers.values]
        X-Frame-Options = "DENY"
        X-Content-Type-Options = "nosniff"
        X-XSS-Protection = "1; mode=block"
        Referrer-Policy = "strict-origin-when-cross-origin"
        Content-Security-Policy = "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
        Permissions-Policy = "camera=(), microphone=(), geolocation=()"

# 静态资源缓存
[[headers]]
    for = "/*.js"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/*.css"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/*.woff2"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

# 图片缓存
[[headers]]
    for = "/*.png"
    [headers.values]
        Cache-Control = "public, max-age=31536000"

[[headers]]
    for = "/*.jpg"
    [headers.values]
        Cache-Control = "public, max-age=31536000"

# HTML 不缓存
[[headers]]
    for = "/*.html"
    [headers.values]
        Cache-Control = "no-cache, no-store, must-revalidate"

# CORS 配置
[[headers]]
    for = "/api/*"
    [headers.values]
        Access-Control-Allow-Origin = "*"
        Access-Control-Allow-Methods = "GET, POST, PUT, DELETE, OPTIONS"
        Access-Control-Allow-Headers = "Content-Type, Authorization"

# ─────────────────────────────────────────────────────────
# 函数配置
# ─────────────────────────────────────────────────────────
[functions]
    # 函数目录
    directory = "netlify/functions"
    
    # Node.js 版本
    node_bundler = "esbuild"
    
    # 外部模块
    external_node_modules = ["sharp", "aws-sdk"]
    
    # 忽略规则
    included_files = [".env.*"]

# 函数级配置
[functions."*.js"]
    name = "custom-name"
    runtime = "nodejs18.x"
    memory = 256
    timeout = 10

# Edge Functions
[edge_functions]
    directory = "netlify/edge-functions"

# ─────────────────────────────────────────────────────────
# 表单配置
# ─────────────────────────────────────────────────────────
[[forms]]
    name = "contact"
    to = "alerts@example.com"
    subject = "New Contact Form Submission"
    success_redirect = "/thank-you"
    honeypot = "bot-field"

# Slack 通知
[[formAlert]]
    enabled = true
    email = "alerts@example.com"
    threshold = 5
    slack_webhook_url = "https://hooks.slack.com/services/xxx"

# ─────────────────────────────────────────────────────────
# 分流测试
# ─────────────────────────────────────────────────────────
[[splittests]]
    enabled = true

[[splittests.variants]]
    name = "control"
    weight = 80

[[splittests.variants]]
    name = "variant-a"
    weight = 20

# ─────────────────────────────────────────────────────────
# 环境配置
# ─────────────────────────────────────────────────────────
[context.production]
    command = "npm run build:production"
    [context.production.environment]
        NODE_ENV = "production"

[context.branch-deploy]
    command = "npm run build"
    [context.branch-deploy.environment]
        NODE_ENV = "staging"

[context.deploy-preview]
    command = "npm run build"
    [context.deploy-preview.environment]
        NODE_ENV = "preview"

[context.staging]
    command = "npm run build:staging"
    [context.staging.environment]
        NODE_ENV = "staging"

# ─────────────────────────────────────────────────────────
# 插件
# ─────────────────────────────────────────────────────────
[plugins]
    # 构建优化
    package = "@netlify/plugin-sitemap"
    package = "@netlify/plugin-robots-txt"
    package = "@netlify/plugin-nextjs"
```

---

## 核心功能详解

### Netlify Functions 深度解析

#### 函数类型对比

| 类型 | 运行时 | 冷启动 | 超时 | 并发 | 成本 |
|------|--------|--------|------|------|------|
| **Functions** | Node.js/Go | ~100ms | 10-26s | 100-500 | 按调用计费 |
| **Edge Functions** | Deno | < 5ms | 50ms | 无限 | 按请求计费 |
| **Background Functions** | Node.js | N/A | 15min | 有限 | 额外计费 |

#### 函数高级配置

```typescript
// netlify/functions/advanced.ts
import { Handler, HandlerEvent, HandlerContext } from '@netlify/functions';

// 函数级配置
export const config = {
    path: '/api/advanced',
    method: ['GET', 'POST'],
    memory: 1024,
    timeoutSec: 30,
    runtime: 'nodejs18.x',
};

// 预热函数
export const handler: Handler = async (
    event: HandlerEvent,
    context: HandlerContext
) => {
    // 记录调用
    console.log('Function called:', {
        path: event.path,
        httpMethod: event.httpMethod,
        queryStringParameters: event.queryStringParameters,
    });
    
    // CORS 头
    const headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Headers': 'Content-Type, Authorization',
        'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
        'Content-Type': 'application/json',
    };
    
    // 处理 OPTIONS
    if (event.httpMethod === 'OPTIONS') {
        return {
            statusCode: 200,
            headers,
            body: '',
        };
    }
    
    // 处理请求
    try {
        const body = event.body ? JSON.parse(event.body) : {};
        
        return {
            statusCode: 200,
            headers,
            body: JSON.stringify({
                message: 'Success',
                data: body,
                timestamp: new Date().toISOString(),
            }),
        };
    } catch (error) {
        return {
            statusCode: 500,
            headers,
            body: JSON.stringify({
                error: 'Internal server error',
                message: error instanceof Error ? error.message : 'Unknown error',
            }),
        };
    }
};
```

#### 数据库集成

```typescript
// netlify/functions/users.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

interface CreateUserInput {
    name: string;
    email: string;
    age?: number;
}

interface UpdateUserInput {
    name?: string;
    email?: string;
    age?: number;
}

export const handler = async (event: any) => {
    const headers = {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*',
    };
    
    const pathParts = event.path.split('/');
    const userId = pathParts[pathParts.length - 1];
    const isIdEndpoint = !isNaN(parseInt(userId));
    
    try {
        switch (event.httpMethod) {
            case 'GET':
                if (isIdEndpoint) {
                    const user = await prisma.user.findUnique({
                        where: { id: parseInt(userId) },
                        include: { posts: true },
                    });
                    
                    if (!user) {
                        return {
                            statusCode: 404,
                            headers,
                            body: JSON.stringify({ error: 'User not found' }),
                        };
                    }
                    
                    return {
                        statusCode: 200,
                        headers,
                        body: JSON.stringify(user),
                    };
                }
                
                const users = await prisma.user.findMany({
                    orderBy: { createdAt: 'desc' },
                    take: 100,
                    include: { _count: { select: { posts: true } } },
                });
                
                return {
                    statusCode: 200,
                    headers,
                    body: JSON.stringify(users),
                };
                
            case 'POST':
                const createInput: CreateUserInput = JSON.parse(event.body);
                
                if (!createInput.name || !createInput.email) {
                    return {
                        statusCode: 400,
                        headers,
                        body: JSON.stringify({ error: 'Name and email are required' }),
                    };
                }
                
                const newUser = await prisma.user.create({
                    data: createInput,
                });
                
                return {
                    statusCode: 201,
                    headers,
                    body: JSON.stringify(newUser),
                };
                
            case 'PUT':
                if (!isIdEndpoint) {
                    return {
                        statusCode: 400,
                        headers,
                        body: JSON.stringify({ error: 'User ID is required' }),
                    };
                }
                
                const updateInput: UpdateUserInput = JSON.parse(event.body);
                const updatedUser = await prisma.user.update({
                    where: { id: parseInt(userId) },
                    data: updateInput,
                });
                
                return {
                    statusCode: 200,
                    headers,
                    body: JSON.stringify(updatedUser),
                };
                
            case 'DELETE':
                if (!isIdEndpoint) {
                    return {
                        statusCode: 400,
                        headers,
                        body: JSON.stringify({ error: 'User ID is required' }),
                    };
                }
                
                await prisma.user.delete({
                    where: { id: parseInt(userId) },
                });
                
                return {
                    statusCode: 204,
                    headers,
                    body: '',
                };
                
            default:
                return {
                    statusCode: 405,
                    headers,
                    body: JSON.stringify({ error: 'Method not allowed' }),
                };
        }
    } catch (error) {
        console.error('Function error:', error);
        
        if (error.code === 'P2002') {
            return {
                statusCode: 409,
                headers,
                body: JSON.stringify({ error: 'User with this email already exists' }),
            };
        }
        
        return {
            statusCode: 500,
            headers,
            body: JSON.stringify({ error: 'Internal server error' }),
        };
    }
};
```

### Netlify Edge Functions 深度解析

#### Deno 运行时特性

```typescript
// netlify/edge-functions/deno-example.ts
// Netlify Edge Functions 使用 Deno 运行时

import { serve } from "https://deno.land/std@0.177.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

serve(async (req) => {
    const url = new URL(req.url);
    
    // 获取地理位置
    const country = req.headers.get("cf-ipcountry") || "US";
    const city = req.headers.get("cf-city") || "Unknown";
    
    // 获取语言偏好
    const acceptLanguage = req.headers.get("accept-language") || "en";
    const preferredLang = acceptLanguage.split(',')[0].split('-')[0];
    
    // 获取用户 ID（从 cookie）
    const cookies = req.headers.get("cookie") || "";
    const userId = cookies.match(/user_id=([^;]+)/)?.[1];
    
    // 创建响应
    const response = new Response(JSON.stringify({
        country,
        city,
        language: preferredLang,
        userId,
        timestamp: new Date().toISOString(),
    }), {
        headers: {
            "Content-Type": "application/json",
            "X-Edge-Location": `${city}, ${country}`,
            "Cache-Control": "no-store",
        },
    });
    
    // 添加响应头
    response.headers.set("X-Edge-Runtime", "Netlify");
    
    return response;
});

export const config = {
    path: "/api/edge/*",
    excludedPath: ["/api/internal/*"],
};
```

### Netlify Identity 深度配置

#### 多因素认证

```typescript
// netlify/functions/auth/mfa.ts
import jwt from 'jsonwebtoken';
import { NetlifyJwtVerifier } from '@netlify/jwt-verifier';

// MFA 验证
export const handler = async (event: any) => {
    const headers = {
        'Content-Type': 'application/json',
    };
    
    try {
        const { userId, code, action } = JSON.parse(event.body);
        
        switch (action) {
            case 'verify-mfa':
                // 验证 MFA 代码
                const isValid = await verifyMFACode(userId, code);
                
                if (!isValid) {
                    return {
                        statusCode: 401,
                        headers,
                        body: JSON.stringify({ error: 'Invalid MFA code' }),
                    };
                }
                
                // 生成 MFA 验证通过的 token
                const token = jwt.sign(
                    { userId, mfaVerified: true },
                    process.env.JWT_SECRET,
                    { expiresIn: '24h' }
                );
                
                return {
                    statusCode: 200,
                    headers,
                    body: JSON.stringify({
                        token,
                        message: 'MFA verified successfully',
                    }),
                };
                
            case 'enable-mfa':
                // 启用 MFA
                const secret = await enableMFA(userId);
                
                return {
                    statusCode: 200,
                    headers,
                    body: JSON.stringify({
                        secret,
                        qrCode: await generateQRCode(secret),
                    }),
                };
                
            default:
                return {
                    statusCode: 400,
                    headers,
                    body: JSON.stringify({ error: 'Invalid action' }),
                };
        }
    } catch (error) {
        return {
            statusCode: 500,
            headers,
            body: JSON.stringify({ error: 'Internal server error' }),
        };
    }
};

async function verifyMFACode(userId: string, code: string): Promise<boolean> {
    // 使用 TOTP 验证
    const speakeasy = require('speakeasy');
    const userSecret = await getUserMFASecret(userId);
    
    return speakeasy.totp.verify({
        secret: userSecret,
        encoding: 'base32',
        token: code,
        window: 1,
    });
}
```

---

## 部署配置

### 多环境部署

#### 环境分支配置

```toml
# netlify.toml
[context.production]
    command = "npm run build"
    [context.production.environment]
        NODE_ENV = "production"
        API_URL = "https://api.example.com"

[context.staging]
    command = "npm run build:staging"
    [context.staging.environment]
        NODE_ENV = "staging"
        API_URL = "https://api-staging.example.com"

[context.branch-deploy]
    command = "npm run build"
    [context.branch-deploy.environment]
        NODE_ENV = "preview"
        API_URL = "https://api-staging.example.com"
```

### 构建钩子

```bash
# 创建构建钩子
netlify buildhooks:add myhook
# 输出: https://api.netlify.com/build_hooks/xxx

# 触发构建
curl -X POST https://api.netlify.com/build_hooks/xxx

# Webhook 签名验证
# netlify/functions/verify-webhook.ts
import crypto from 'crypto';

export const handler = async (event: any) => {
    const signature = event.headers['x-netlify-hook-signature'];
    const body = event.body;
    
    const expectedSignature = crypto
        .createHmac('sha256', process.env.WEBHOOK_SECRET)
        .update(body)
        .digest('hex');
    
    if (signature !== expectedSignature) {
        return {
            statusCode: 401,
            body: JSON.stringify({ error: 'Invalid signature' }),
        };
    }
    
    // 处理 webhook
    const payload = JSON.parse(body);
    
    return {
        statusCode: 200,
        body: JSON.stringify({ message: 'Webhook received' }),
    };
};
```

---

## 环境变量与密钥管理

### 环境变量配置

```bash
# Netlify CLI 设置变量
netlify env:set DATABASE_URL "postgresql://..."
netlify env:set API_KEY "xxx"
netlify env:set --scope build,runtime NODE_ENV "production"

# 范围说明
# --scope build     - 仅构建时
# --scope runtime   - 仅运行时（函数）
# --scope all       - 两者都（默认）

# 批量设置
netlify env:import .env.production

# 列出变量
netlify env:list

# 移除变量
netlify env:unset API_KEY

# 开发时使用本地变量
netlify dev --env .env.local
```

### 敏感数据处理

```typescript
// netlify/functions/secure-handler.ts
export const handler = async (event: any) => {
    // 从环境变量获取敏感信息
    const dbUrl = process.env.DATABASE_URL;
    const apiKey = process.env.SECURE_API_KEY;
    
    // 永远不要在响应中返回敏感信息
    return {
        statusCode: 200,
        body: JSON.stringify({
            // ✓ 正确：只返回非敏感信息
            message: 'Success',
            userId: user.id,
            
            // ✗ 错误：会泄露敏感信息
            // apiKey: apiKey,
        }),
    };
};

// 审计日志
export const handler = async (event: any) => {
    // 记录操作但不记录敏感值
    console.log('API call', {
        path: event.path,
        method: event.httpMethod,
        userAgent: event.headers['user-agent'],
        // 不记录: event.headers['authorization']
    });
};
```

---

## CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/netlify.yml
name: Netlify Deploy

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

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './dist'
          production-deploy: ${{ github.ref == 'refs/heads/main' }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: true
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### 环境特定部署

```yaml
# .github/workflows/netlify-environments.yml
name: Netlify Multi-Environment Deploy

on:
  push:
    branches:
      - develop
      - staging
      - main

jobs:
  deploy-develop:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Development
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './dist'
          deploy-message: "Deploy from GitHub Actions - Develop"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_DEV }}

  deploy-staging:
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Staging
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './dist'
          production-deploy: false
          deploy-message: "Deploy from GitHub Actions - Staging"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_STAGING }}

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Production
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './dist'
          production-deploy: true
          deploy-message: "Deploy from GitHub Actions - Production"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID_PROD }}
```

---

## 性能优化与缓存策略

### 构建优化

```toml
# netlify.toml - 构建优化
[build]
    command = "npm run build"
    publish = "dist"

[build.processing]
    skip_processing = false

[build.processing.css]
    bundle = true
    minify = true

[build.processing.js]
    bundle = true
    minify = true
    # 代码分割
   分流 = true

[build.processing.images]
    compress = true
    webp = true
    formats = ["webp", "avif"]

# 缓存控制
[[headers]]
    for = "/assets/*"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/_next/static/*"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"
```

### Edge 缓存策略

```typescript
// netlify/edge-functions/cache-control.ts
export default async (request: Request, context: Context) => {
    const url = new URL(request.url);
    const pathname = url.pathname;
    
    // 静态资源 - 长期缓存
    if (pathname.match(/\.(js|css|woff2|png|jpg|webp|avif)$/)) {
        context.cache({
            maxAge: 31536000,
            staleWhileRevalidate: 86400,
        });
    }
    
    // API 响应 - 不缓存
    if (pathname.startsWith('/api/')) {
        context.cache({
            maxAge: 0,
            noStore: true,
        });
    }
    
    // 页面 - 短期缓存
    if (pathname.match(/^\/blog\//)) {
        context.cache({
            maxAge: 3600,
            staleWhileRevalidate: 86400,
        });
    }
    
    return context.next();
};

export const config = {
    path: "/*",
};
```

---

## 成本估算与选型建议

### 成本计算器

```typescript
// Netlify 成本计算
const pricingCalculator = {
    // 场景 1: 个人项目
    personal: {
        bandwidth: 100, // GB/月
        functionsCalls: 125000, // 免费额度
        buildMinutes: 300, // 免费额度
        
        calculate: () => {
            // 免费版完全够用
            return {
                bandwidth: 0,
                functions: 0,
                builds: 0,
                total: 0,
            };
        },
    },
    
    // 场景 2: 小型团队
    smallTeam: {
        bandwidth: 400, // GB/月
        functionsCalls: 500000,
        buildMinutes: 1000,
        teamMembers: 5,
        
        calculate: () => {
            const costs = {
                // Starter: $19/月
                // 100GB 带宽, 125K 函数调用, 300 构建分钟
                // 额外带宽: $0.4/GB
                bandwidth: Math.max(0, 400 - 100) * 0.4, // $120
                
                // 额外函数调用: $0.0004/调用
                functions: Math.max(0, 500000 - 125000) * 0.0004, // $150
                
                // 额外构建分钟: $0.036/分钟
                builds: Math.max(0, 1000 - 300) * 0.036, // $25
                
                // Starter 基础费用
                base: 19,
            };
            
            costs.total = Object.values(costs).reduce((a, b) => a + b, 0);
            return costs;
        },
    },
    
    // 场景 3: 商业项目
    business: {
        bandwidth: 1000, // GB/月
        functionsCalls: 2000000,
        buildMinutes: 3000,
        teamMembers: 15,
        
        calculate: () => {
            // Pro: $99/月
            // 400GB 带宽, 500K 函数调用, 1000 构建分钟
            const costs = {
                bandwidth: Math.max(0, 1000 - 400) * 0.35, // $210
                functions: Math.max(0, 2000000 - 500000) * 0.00035, // $525
                builds: Math.max(0, 3000 - 1000) * 0.03, // $60
                base: 99,
            };
            
            costs.total = Object.values(costs).reduce((a, b) => a + b, 0);
            return costs;
        },
    },
};
```

### 选型建议矩阵

| 场景 | 推荐方案 | 理由 |
|------|----------|------|
| **个人项目/博客** | Hobby 免费版 | 100GB 带宽足够 |
| **小型网站** | Starter $19/月 | 包含表单和身份认证 |
| **中型应用** | Pro $99/月 | 更多额度和功能 |
| **企业应用** | Business $299/月 | SSO、ACL、优先支持 |
| **电商平台** | Business + 额外带宽 | 需要更多带宽 |

---

## 常见问题与解决方案

### 构建问题

#### 问题：构建失败

```bash
# 检查构建日志
netlify deploy --debug

# 常见原因：

# 1. 依赖安装失败
# 检查 package.json
# netlify.toml
[build]
    command = "npm ci"

# 2. 构建命令错误
# 验证命令本地运行
npm run build

# 3. 环境变量缺失
netlify env:list
netlify env:set MISSING_VAR "value"
```

#### 问题：构建超时

```bash
# 优化构建命令
# package.json
{
    "scripts": {
        "build": "npm run lint && npm run test && npm run build:prod"
    }
}

# 分离 lint/test
# netlify.toml
[context.production]
    command = "npm run build"
```

### 部署问题

#### 问题：部署后显示旧内容

```bash
# 清除缓存并重新部署
netlify deploy --prod --build --message "Clear cache and redeploy"

# 或在 Dashboard 中
# Deploys → Clear cache → Deploy site
```

#### 问题：重定向不工作

```toml
# netlify.toml
# 确保重定向配置正确

# 常用重定向
[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200

# SPA 路由
# 确保发布目录有 index.html
```

### 函数问题

#### 问题：函数超时

```typescript
// netlify/functions/timeout-handler.ts
// 增加超时时间（需要付费版）

export const config = {
    timeoutSec: 26, // 最大 26 秒
};

export const handler = async (event: any) => {
    // 长时间任务使用异步
    const result = await processLongTask(event);
    
    return {
        statusCode: 200,
        body: JSON.stringify(result),
    };
};

// 或使用 Background Functions
```

#### 问题：函数内存不足

```typescript
// 优化内存使用
export const config = {
    memory: 1024, // 增加内存（需要付费版）
};

export const handler = async (event: any) => {
    // 处理大文件时使用流
    const stream = await fetchLargeData();
    
    return new Response(stream, {
        headers: { 'Content-Type': 'application/json' },
    });
};
```

---

## 服务概述与定位

### Netlify 在现代开发中的角色

Netlify 不仅仅是一个静态站点托管平台，更是一个完整的前端工作流解决方案。Netlify 的核心理念是**将现代前端开发的最佳实践自动化**，让开发者能够专注于构建卓越的用户体验。

#### 平台核心价值

```
┌─────────────────────────────────────────────────────────────────────┐
│                     Netlify 平台价值体系                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    JAMstack 架构                                │ │
│  │  JavaScript + APIs + Markup = 极快、安全、易扩展                │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    GitOps 工作流                                │ │
│  │  Push → Build → Preview → Merge → Deploy                      │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    边缘计算                                    │ │
│  │  Edge Functions → 全球分布 → 毫秒响应                          │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                              ↓                                      │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │                    内置能力                                    │ │
│  │  表单 + 认证 + 分析 + 分流 = 开箱即用                          │ │
│  └───────────────────────────────────────────────────────────────┘ │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

#### 与传统 Web 托管的对比

| 维度 | Netlify | 传统托管 | VPS |
|------|---------|----------|-----|
| **部署方式** | Git 推送 | FTP/SSH | SSH |
| **构建时间** | 优化构建 | 无构建 | 无构建 |
| **CDN** | 全球自动 | 有限 | 无 |
| **SSL** | 自动 | 手动 | 手动 |
| **CI/CD** | 内置 | 需配置 | 需配置 |
| **表单处理** | 内置 | 需后端 | 需后端 |
| **预览部署** | 每个 PR | 无 | 无 |

### 技术生态全景

#### 核心组件

```typescript
// Netlify 技术栈
const netlifyStack = {
    // 计算层
    compute: {
        'netlify_functions': 'AWS Lambda',
        'edge_functions': 'V8 Isolates',
        'buildbot': '自定义构建系统',
    },
    
    // 存储层
    storage: {
        'blob': 'Netlify Blob Storage',
        'forms': '表单数据存储',
        'identity': '用户数据',
    },
    
    // 网络层
    network: {
        'cdn': '全球 300+ PoPs',
        'proxy': '智能路由',
        'tls': '自动 HTTPS',
    },
    
    // 开发者工具
    devtools: {
        'cli': 'Netlify CLI',
        'git': 'GitHub/GitLab/Bitbucket',
        'plugins': '构建插件生态',
    },
};
```

---

## 完整配置教程

### Netlify CLI 深度使用

#### 安装与配置

```bash
# npm 安装
npm install -g netlify-cli

# Homebrew 安装 (macOS)
brew install netlify-cli

# 验证安装
netlify --version

# 登录
netlify login

# 链接到已有站点
netlify link

# 初始化新站点
netlify init
```

#### 常用命令

```bash
# 部署
netlify deploy                    # 部署到临时 URL
netlify deploy --prod           # 部署到生产
netlify deploy --dir=dist       # 指定目录
netlify deploy --message "Update" # 部署信息

# 站点管理
netlify sites:list              # 列出所有站点
netlify sites:create            # 创建新站点
netlify sites:delete            # 删除站点

# 环境变量
netlify env:list                # 列出变量
netlify env:set KEY=value       # 设置变量
netlify env:unset KEY           # 删除变量
netlify env:import --file .env # 批量导入

# 函数
netlify functions:create        # 创建函数
netlify functions:build         # 构建函数
netlify functions:invoke        # 调用函数
netlify functions:log          # 查看日志

# 获取远程环境
netlify env:pull               # 下载生产环境变量
```

### netlify.toml 完整配置

```toml
# netlify.toml - 完整配置示例

# ─────────────────────────────────────────────────────────
# 构建配置
# ─────────────────────────────────────────────────────────
[build]
    # 构建命令
    command = "npm run build"
    
    # 输出目录
    publish = "dist"
    
    # 基础目录（相对于仓库根目录）
    base = "/"
    
    # 构建环境变量
    [build.environment]
        NODE_VERSION = "20"
        NPM_VERSION = "10"
        PYTHON_VERSION = "3.12"
        HUGO_VERSION = "0.123.0"

# 构建命令别名
[build.build]
    command = "npm run build:prod"
    publish = "build"

# 开发设置
[build.settings]
    node_bundler = "esbuild"
    rust_version = "stable"

# ─────────────────────────────────────────────────────────
# 重定向配置
# ─────────────────────────────────────────────────────────
# SPA 重定向
[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200

# API 重定向
[[redirects]]
    from = "/api/*"
    to = "/.netlify/functions/:splat"
    status = 200

# 旧链接重定向
[[redirects]]
    from = "/old-page"
    to = "/new-page"
    status = 301

[[redirects]]
    from = "/blog/:year/:month/:slug"
    to = "/blog/:slug"
    status = 301

# 条件重定向
[[redirects]]
    from = "/preview/*"
    to = "/preview.html"
    status = 200
    conditions = {Language = ["en"]}

[[redirects]]
    from = "/preview/*"
    to = "/preview-es.html"
    status = 200
    conditions = {Language = ["es"]}

# 参数传递
[[redirects]]
    from = "/products/:id"
    to = "/product.html?id=:id"
    status = 200

# 代理
[[redirects]]
    from = "/api/*"
    to = "https://api.external.com/*"
    status = 200
    force = true

# ─────────────────────────────────────────────────────────
# Header 配置
# ─────────────────────────────────────────────────────────
# 安全头
[[headers]]
    for = "/*"
    [headers.values]
        X-Frame-Options = "DENY"
        X-Content-Type-Options = "nosniff"
        X-XSS-Protection = "1; mode=block"
        Referrer-Policy = "strict-origin-when-cross-origin"
        Permissions-Policy = "camera=(), microphone=(), geolocation=()"

# 缓存头
[[headers]]
    for = "/static/*"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/*.js"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/*.css"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

[[headers]]
    for = "/sw.js"
    [headers.values]
        Cache-Control = "no-cache"

# CORS 头
[[headers]]
    for = "/api/*"
    [headers.values]
        Access-Control-Allow-Origin = "*"
        Access-Control-Allow-Methods = "GET, POST, PUT, DELETE, OPTIONS"
        Access-Control-Allow-Headers = "Content-Type, Authorization"

# ─────────────────────────────────────────────────────────
# 表单配置
# ─────────────────────────────────────────────────────────
[[forms]]
    name = "contact"
    to = "alerts@example.com"
    subject = "New contact from {{name}}"
    success_redirect = "/thank-you"

[[forms]]
    name = "newsletter"
    to = "newsletter@example.com"
    captcha = true

# ─────────────────────────────────────────────────────────
# 分流测试
# ─────────────────────────────────────────────────────────
[[splittests]]
    enabled = true

[[splittests.variants]]
    name = "control"
    weight = 80

[[splittests.variants]]
    name = "variant-a"
    weight = 20

# ─────────────────────────────────────────────────────────
# 插件配置
# ─────────────────────────────────────────────────────────
[plugins]
    # 内置插件
    # 无需安装，直接使用

# 第三方插件（通过 npm 安装）
# [plugins.package]
#     enabled = true
#     [plugins.package.config]
#         option = "value"
```

### 多环境配置

#### 环境分支映射

```toml
# netlify.toml
[build]
    command = "npm run build"
    publish = "dist"

# 生产环境分支
[context.production]
    command = "npm run build:prod"
    [context.production.environment]
        NODE_ENV = "production"

# 预览环境
[context.deploy-preview]
    command = "npm run build:preview"
    [context.deploy-preview.environment]
        NODE_ENV = "staging"
        NEXT_PUBLIC_API_URL = "https://api-staging.example.com"

# 分支部署
[context.branch-deploy]
    command = "npm run build"
    [context.branch-deploy.environment]
        NODE_ENV = "staging"

# 特定分支
[context.branch-prod]
    branch = "production"
    command = "npm run build:prod"
```

---

## 核心功能详解

### Netlify Functions 深度解析

#### 函数配置

```typescript
// netlify/functions/hello.ts
export const handler = async (event, context) => {
    // event: AWS Lambda 事件对象
    // context: AWS Lambda 上下文对象
    
    return {
        statusCode: 200,
        body: JSON.stringify({ message: 'Hello!' }),
    };
};

// 导出配置
export const config = {
    path: '/hello',  // 函数路径
    memory: 1024,     // 内存 MB
    timeout: 10,      // 超时秒
};
```

#### 异步函数

```typescript
// netlify/functions/async-handler.ts
export const handler = async (event, context) => {
    // 异步操作
    const data = await fetchData();
    
    return {
        statusCode: 200,
        body: JSON.stringify(data),
    };
};

// 带错误处理
export const handler = async (event, context) => {
    try {
        const result = await riskyOperation();
        return {
            statusCode: 200,
            body: JSON.stringify(result),
        };
    } catch (error) {
        console.error('Error:', error);
        return {
            statusCode: 500,
            body: JSON.stringify({ error: 'Internal server error' }),
        };
    }
};
```

#### Stripe 集成

```typescript
// netlify/functions/create-checkout-session.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

export const handler = async (event) => {
    if (event.httpMethod !== 'POST') {
        return {
            statusCode: 405,
            body: JSON.stringify({ error: 'Method not allowed' }),
        };
    }

    try {
        const { priceId, successUrl, cancelUrl } = JSON.parse(event.body);

        const session = await stripe.checkout.sessions.create({
            payment_method_types: ['card'],
            line_items: [
                {
                    price: priceId,
                    quantity: 1,
                },
            ],
            mode: 'subscription',
            success_url: successUrl,
            cancel_url: cancelUrl,
        });

        return {
            statusCode: 200,
            body: JSON.stringify({ sessionId: session.id }),
        };
    } catch (error) {
        console.error('Stripe error:', error);
        return {
            statusCode: 500,
            body: JSON.stringify({ error: error.message }),
        };
    }
};
```

#### GraphQL 函数

```typescript
// netlify/functions/graphql.ts
import { graphql, buildSchema } from 'graphql';

const schema = buildSchema(`
    type Query {
        hello: String
        user(id: ID!): User
        users: [User]
    }
    
    type User {
        id: ID
        name: String
        email: String
    }
`);

const root = {
    hello: () => 'Hello, World!',
    user: ({ id }) => ({ id, name: 'User ' + id, email: `user${id}@example.com` }),
    users: () => [
        { id: '1', name: 'Alice', email: 'alice@example.com' },
        { id: '2', name: 'Bob', email: 'bob@example.com' },
    ],
};

export const handler = async (event) => {
    const { query, variables } = JSON.parse(event.body);

    try {
        const result = await graphql({
            schema,
            source: query,
            rootValue: root,
            variableValues: variables,
        });

        return {
            statusCode: 200,
            body: JSON.stringify(result),
        };
    } catch (error) {
        return {
            statusCode: 400,
            body: JSON.stringify({ errors: [{ message: error.message }] }),
        };
    }
};
```

### Netlify Edge Functions 高级用法

#### 请求改写

```typescript
// netlify/edge-functions/rewrite.ts
export default async (request, context) => {
    const url = new URL(request.url);
    
    // 检测 UA
    const ua = request.headers.get('user-agent');
    
    if (ua.includes('Mobile') || ua.includes('Android')) {
        // 移动设备重写
        url.pathname = `/mobile${url.pathname}`;
        const modifiedRequest = new Request(url.toString(), request);
        return await context.next({ request: modifiedRequest });
    }
    
    // 默认处理
    return await context.next();
};
```

#### 认证中间件

```typescript
// netlify/edge-functions/auth.ts
export default async (request, context) => {
    // 公开路径
    const publicPaths = ['/login', '/register', '/forgot-password'];
    if (publicPaths.some(p => request.url.includes(p))) {
        return await context.next();
    }
    
    // 检查认证
    const token = getToken(request);
    
    if (!token) {
        // 未认证，重定向到登录
        const loginUrl = new URL('/login', request.url);
        loginUrl.searchParams.set('redirect', request.url);
        return Response.redirect(loginUrl.toString(), 302);
    }
    
    // 验证 token
    const user = await verifyToken(token);
    
    if (!user) {
        // token 无效
        return new Response('Unauthorized', { status: 401 });
    }
    
    // 添加用户信息到上下文
    context.user = user;
    
    return await context.next();
};

function getToken(request) {
    const cookieHeader = request.headers.get('cookie');
    const cookies = Object.fromEntries(
        cookieHeader.split(';').map(c => c.trim().split('='))
    );
    return cookies['auth-token'];
}

async function verifyToken(token) {
    // 实现 token 验证逻辑
    // 可以调用其他 API 或使用 JWT 验证
    return null;
}
```

#### 实时数据分析

```typescript
// netlify/edge-functions/analytics.ts
export default async (request, context) => {
    const startTime = Date.now();
    
    // 处理请求
    const response = await context.next();
    
    // 计算响应时间
    const duration = Date.now() - startTime;
    
    // 记录分析数据
    const analytics = {
        url: request.url,
        method: request.method,
        status: response.status,
        duration,
        timestamp: new Date().toISOString(),
        country: context.geo?.country?.code,
        city: context.geo?.city?.name,
    };
    
    // 发送到分析服务
    await sendToAnalytics(analytics);
    
    return response;
};

async function sendToAnalytics(data) {
    // 可以发送到日志服务、数据库或分析平台
    console.log('Analytics:', JSON.stringify(data));
}
```

### Netlify Identity 高级配置

#### 自定义注册流程

```typescript
// netlify/functions/register.ts
import { GotrueError } from '@netlify/gotrue-js';

export const handler = async (event) => {
    if (event.httpMethod !== 'POST') {
        return {
            statusCode: 405,
            body: JSON.stringify({ error: 'Method not allowed' }),
        };
    }

    const { email, password, name } = JSON.parse(event.body);

    // 验证输入
    if (!email || !password) {
        return {
            statusCode: 400,
            body: JSON.stringify({ error: 'Email and password required' }),
        };
    }

    // 密码强度检查
    if (password.length < 8) {
        return {
            statusCode: 400,
            body: JSON.stringify({ error: 'Password must be at least 8 characters' }),
        };
    }

    try {
        // 创建用户
        const user = await createUser(email, password, {
            name,
            role: 'user',
        });

        // 发送验证邮件
        await sendVerificationEmail(email);

        return {
            statusCode: 201,
            body: JSON.stringify({
                message: 'User created. Please verify your email.',
                user: { id: user.id, email: user.email },
            }),
        };
    } catch (error) {
        if (error instanceof GotrueError) {
            return {
                statusCode: 400,
                body: JSON.stringify({ error: error.message }),
            };
        }

        return {
            statusCode: 500,
            body: JSON.stringify({ error: 'Internal server error' }),
        };
    }
};
```

#### 角色权限管理

```typescript
// netlify/functions/admin-only.ts
export const handler = async (event) => {
    // 验证认证
    const user = await verifyAuth(event);
    
    if (!user) {
        return {
            statusCode: 401,
            body: JSON.stringify({ error: 'Unauthorized' }),
        };
    }
    
    // 检查角色
    const roles = user.app_metadata?.roles || [];
    
    if (!roles.includes('admin') && !roles.includes('moderator')) {
        return {
            statusCode: 403,
            body: JSON.stringify({ error: 'Forbidden: Admin access required' }),
        };
    }
    
    // 处理请求
    return {
        statusCode: 200,
        body: JSON.stringify({ message: 'Welcome, Admin!' }),
    };
};

async function verifyAuth(event) {
    const authHeader = event.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return null;
    }
    
    const token = authHeader.substring(7);
    
    // 验证 JWT
    try {
        const decoded = verifyJWT(token);
        return decoded;
    } catch {
        return null;
    }
}
```

---

## 部署配置

### Next.js 应用部署

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
    // 输出目录
    output: 'standalone',
    
    // 图片优化
    images: {
        domains: ['images.unsplash.com', 'picsum.photos'],
        formats: ['image/avif', 'image/webp'],
    },
    
    // 实验性功能
    experimental: {
        serverActions: true,
        serverComponentsExternalPackages: ['@prisma/client'],
    },
};
```

```toml
# netlify.toml - Next.js 配置
[build]
    command = "npm run build"
    publish = ".next"

# Next.js 需要函数处理 SSR
[[plugins]]
    package = "@netlify/plugin-nextjs"

[build.environment]
    NEXT_TELEMETRY_DISABLED = "1"
```

### Astro 应用部署

```toml
# netlify.toml - Astro 配置
[build]
    command = "npm run build"
    publish = "dist"

# SSR 配置（如果使用 SSR 模式）
[[redirects]]
    from = "/*"
    to = "/.netlify/functions/entry"
    status = 200
```

### Hugo 站点部署

```toml
# netlify.toml - Hugo 配置
[build]
    command = "hugo --gc --minify"
    publish = "public"

[build.environment]
    HUGO_VERSION = "0.123.0"
```

### Nuxt 应用部署

```javascript
// nuxt.config.ts
export default defineNuxtConfig({
    // Netlify 函数
    nitro: {
        preset: 'aws-lambda',
    },
    
    // 预渲染
    routeRules: {
        '/': { prerender: true },
        '/blog/**': { prerender: true },
        '/api/**': { cache: { maxAge: 60 } },
    },
});
```

```toml
# netlify.toml - Nuxt 配置
[build]
    command = "npm run build"
    publish = ".output/public"

[[plugins]]
    package = "@netlify/plugin-nextjs"
```

---

## 环境变量与密钥管理

### 变量配置

```bash
# Netlify CLI 设置变量
netlify env:set DATABASE_URL "postgresql://..."
netlify env:set API_KEY "xxx"

# 设置多个
netlify env:set KEY1=value1 KEY2=value2

# 从文件导入
netlify env:import --file .env.production

# 导出变量
netlify env:pull --file .env.local

# 上下文变量
netlify env:set DATABASE_URL "staging-url" --context deploy-preview
netlify env:set DATABASE_URL "prod-url" --context production
```

### 密钥管理最佳实践

```bash
# 1. 使用 .env 文件（不提交到 Git）
# .env.example - 模板（不含敏感值）
DATABASE_URL=
STRIPE_SECRET_KEY=
JWT_SECRET=

# 2. 本地开发使用 .env.local
# 提交到 Netlify 的只是 .env.example

# 3. Netlify Dashboard 中设置敏感值
# Settings → Environment Variables

# 4. 定期轮换密钥
# netlify env:unset OLD_KEY
# netlify env:set NEW_KEY "xxx"
```

---

## CI/CD 集成

### GitHub Actions

```yaml
# .github/workflows/netlify-deploy.yml
name: Netlify Deployment

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

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './dist'
          production-branch: main
          production-deploy: ${{ github.ref == 'refs/heads/main' }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: true
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### GitLab CI

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

variables:
  NETLIFY_SITE_ID: ${NETLIFY_SITE_ID}

test:
  stage: test
  image: node:20-alpine
  script:
    - npm ci
    - npm test

build:
  stage: build
  image: node:20-alpine
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/

deploy:
  stage: deploy
  image: node:20-alpine
  before_script:
    - npm install -g netlify-cli
  script:
    - netlify deploy --dir=dist --prod
  only:
    - main
  environment:
    name: production
    url: ${NETLIFY_URL}
```

### CI/CD 最佳实践

```yaml
# 完整的 CI/CD 流水线
name: Full CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  # ─────────────────────────────────────────────────────────
  # 代码检查
  # ─────────────────────────────────────────────────────────
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint

  # ─────────────────────────────────────────────────────────
  # 测试
  # ─────────────────────────────────────────────────────────
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  # ─────────────────────────────────────────────────────────
  # 构建
  # ─────────────────────────────────────────────────────────
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # ─────────────────────────────────────────────────────────
  # 部署到 Staging
  # ─────────────────────────────────────────────────────────
  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/develop'
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: ./dist
          deploy-message: "Staging deploy from GitHub Actions"
          alias: staging
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_STAGING_SITE_ID }}

  # ─────────────────────────────────────────────────────────
  # 部署到 Production
  # ─────────────────────────────────────────────────────────
  deploy-production:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: ./dist
          production-deploy: true
          deploy-message: "Production deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_PRODUCTION_SITE_ID }}
```

---

## 性能优化与缓存策略

### 构建优化

```toml
# netlify.toml - 构建优化
[build]
    command = "npm run build"
    publish = "dist"

# 禁用不必要的处理
[build.processing]
    skip_processing = false

[build.processing.css]
    bundle = true
    minify = true

[build.processing.js]
    bundle = true
    minify = true

[build.processing.html]
    pretty_urls = true

[build.processing.images]
    compress = true
```

### 缓存策略

```toml
# netlify.toml - 缓存配置
# 静态资源长期缓存
[[headers]]
    for = "/static/*"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

# JavaScript 资源
[[headers]]
    for = "/*.js"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

# CSS 资源
[[headers]]
    for = "/*.css"
    [headers.values]
        Cache-Control = "public, max-age=31536000, immutable"

# 图片资源
[[headers]]
    for = "/images/*"
    [headers.values]
        Cache-Control = "public, max-age=2592000, immutable"

# API 响应不缓存
[[headers]]
    for = "/api/*"
    [headers.values]
        Cache-Control = "no-cache, no-store, must-revalidate"

# HTML 页面短缓存
[[headers]]
    for = "/*.html"
    [headers.values]
        Cache-Control = "public, max-age=0, must-revalidate"
```

### Edge Function 缓存

```typescript
// netlify/edge-functions/cached-api.ts
export default async (request, context) => {
    const cacheKey = request.url;
    
    // 检查缓存
    const cached = await context.cache.get(cacheKey);
    
    if (cached) {
        return new Response(cached, {
            headers: {
                'Content-Type': 'application/json',
                'X-Cache': 'HIT',
            },
        });
    }
    
    // 获取数据
    const response = await fetch('https://api.example.com/data');
    const data = await response.text();
    
    // 缓存 1 小时
    await context.cache.put(
        cacheKey,
        data,
        60 * 60 // seconds
    );
    
    return new Response(data, {
        headers: {
            'Content-Type': 'application/json',
            'X-Cache': 'MISS',
        },
    });
};

export const config = {
    path: '/api/data',
    cache: 'manual',  // 启用手动缓存
};
```

---

## 成本估算与选型建议

### 成本计算器

```python
# Netlify 成本计算
def calculate_cost(
    team_members: int,
    bandwidth_gb: float,
    build_minutes: float,
    function_requests: int,
    forms_submissions: int,
) -> dict:
    """计算 Netlify 月度成本"""
    
    # 免费额度
    free_bandwidth = 100  # GB
    free_build_minutes = 300  # 分钟
    free_functions = 125000  # 请求
    free_forms = 100  # 提交
    
    # 套餐价格
    plans = {
        'starter': {'price': 19, 'bandwidth': 400, 'members': 1},
        'pro': {'price': 99, 'bandwidth': 1000, 'members': 5},
        'business': {'price': 299, 'bandwidth': 2000, 'members': 15},
    }
    
    # 计算成本
    base_cost = 0
    extra_bandwidth_cost = 0
    extra_build_cost = 0
    extra_functions_cost = 0
    
    if team_members <= 1:
        plan = plans['starter']
    elif team_members <= 5:
        plan = plans['pro']
    else:
        plan = plans['business']
    
    base_cost = plan['price']
    
    # 超出带宽
    extra_bandwidth = max(0, bandwidth_gb - plan['bandwidth'])
    if extra_bandwidth > 0:
        extra_bandwidth_cost = extra_bandwidth * 0.20  # $0.20/GB
    
    # 超出构建时间
    if build_minutes > plan.get('build_minutes', 3000):
        extra_build = build_minutes - plan.get('build_minutes', 3000)
        extra_build_cost = (extra_build / 1000) * 3  # $3/1000 分钟
    
    # 超出函数请求
    extra_functions = max(0, function_requests - free_functions)
    extra_functions_cost = (extra_functions / 10000) * 0.20
    
    return {
        'plan': plan,
        'base_cost': base_cost,
        'extra_bandwidth_cost': round(extra_bandwidth_cost, 2),
        'extra_build_cost': round(extra_build_cost, 2),
        'extra_functions_cost': round(extra_functions_cost, 2),
        'total': round(base_cost + extra_bandwidth_cost + extra_build_cost + extra_functions_cost, 2),
    }
```

### 选型建议矩阵

| 场景 | 推荐套餐 | 估计成本 | 理由 |
|------|----------|-----------|------|
| **个人项目** | Starter ($19/月) | $19/月 | 足够小型站点 |
| **小型团队** | Pro ($99/月) | $99/月 | 5成员，高带宽 |
| **中型企业** | Business ($299/月) | $299/月 | 15成员，更多功能 |
| **高流量站点** | Business + 带宽包 | $400+/月 | 按需扩展 |
| **电商站点** | Pro + Forms | $150+/月 | 表单处理能力 |
| **大型电商** | Business | $500+/月 | 高带宽，多成员 |

---

## 常见问题与解决方案

### 部署问题

#### 问题：构建失败

```bash
# 常见原因和解决方案

# 1. 构建命令错误
# 检查 netlify.toml 中的 command
# 本地测试
npm run build

# 2. 依赖安装失败
# 检查 package.json
# 清除缓存
npm ci --legacy-peer-deps

# 3. 环境变量缺失
# 在 Netlify Dashboard 中添加
# Settings → Environment Variables

# 4. Node 版本不兼容
# 指定版本
[build.environment]
    NODE_VERSION = "20"
```

#### 问题：部署超时

```bash
# 解决方案

# 1. 优化构建命令
npm run build -- --no-lint

# 2. 增加构建超时
# Netlify 默认 15 分钟，可申请延长

# 3. 分离构建
# 使用独立构建服务
```

### 函数问题

#### 问题：函数执行超时

```typescript
// 解决方案

# 1. 优化函数逻辑
# 减少同步操作
# 使用异步处理

# 2. 增加超时配置
export const config = {
    timeout: 26,  // 最大 26 秒
};

# 3. 拆分函数
# 将长时间操作移到后台任务
```

#### 问题：函数内存不足

```typescript
// 解决方案

# 1. 增加内存配置
export const config = {
    memory: 3008,  // 最大 3008 MB
};

# 2. 优化内存使用
# 避免加载大文件
# 及时释放资源
```

### 重定向问题

#### 问题：重定向不生效

```toml
# 解决方案

# 1. 检查语法
[[redirects]]
    from = "/old"
    to = "/new"
    status = 301

# 2. 顺序很重要
# 更具体的规则放前面

# 3. 使用 force
[[redirects]]
    from = "/*"
    to = "/index.html"
    status = 200
    force = false  # 不强制重写已有文件
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://docs.netlify.com/ |
| CLI 文档 | https://cli.netlify.com/ |
| Functions 文档 | https://docs.netlify.com/functions/overview/ |
| Edge Functions 文档 | https://docs.netlify.com/edge-functions/ |
| Identity 文档 | https://docs.netlify.com/visitor-access/identity/ |
| Netlify Blog | https://www.netlify.com/blog/ |
| GitHub | https://github.com/netlify/ |
| Discord | https://discord.gg/netlify |

---

> [!SUCCESS]
> Netlify 是 JAMstack 架构的首选平台，提供开箱即用的表单处理、身份认证和边缘计算能力。对于不需要复杂 SSR 的静态站点和混合应用，Netlify 提供了比 Vercel 更丰富的内置功能，特别适合需要快速构建 MVP 的团队。
