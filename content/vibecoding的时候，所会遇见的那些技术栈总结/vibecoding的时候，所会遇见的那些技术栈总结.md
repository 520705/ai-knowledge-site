# Vibecoding 技术栈全景图

> [!NOTE]
> 本文档为 Vibecoding（全流程 AI 辅助编程）技术栈的完整索引，涵盖 **13 大技术领域、88 种技术工具**。每个子主题均有独立的详细文档——结构化、代码丰富、数据支撑、实战导向。

> [!TIP]
> **阅读路径**：建议按以下顺序浏览：AI 编程助手 → 前端框架 → 前端元框架 → 后端框架 → 数据库 → 云服务 → API 工具 → CSS → 构建工具 → 移动端 → JS 核心

---

## 目录导航

| 类别 | 技术数 | 定位 |
|------|--------|------|
| [一、AI 编程助手](#一ai-编程助手) | 7 种 | 编程入口，AI 辅助核心 |
| [二、前端框架](#二前端框架) | 5 种 | UI 组件化开发标准 |
| [三、前端元框架](#三前端元框架) | 8 种 | 路由、SSR/SSG、构建、样式 |
| [四、后端框架](#四后端框架) | 9 种 | 服务端开发，API 构建 |
| [五、编程语言](#五编程语言) | 5 种 | TypeScript、Python、Go、Rust、JavaScript |
| [六、数据库](#六数据库) | 6 种 | PostgreSQL、MySQL、SQLite、MongoDB、Redis、Supabase |
| [七、云服务与 DevOps](#七云服务与-devops) | 7 种 | 部署、容器化、CI/CD |
| [八、API 与全栈工具](#八api-与全栈工具) | 11 种 | REST/GraphQL、ORM、实时、认证 |
| [九、CSS 与设计体系](#九css-与设计体系) | 10 种 | 布局、样式框架、动画、设计系统 |
| [十、Markdown 与文档工具](#十markdown-与文档工具) | 7 种 | 文档写作、知识管理、API 文档 |
| [十一、构建工具与工程化](#十一构建工具与工程化) | 7 种 | 构建、包管理、代码质量、CI/CD |
| [十二、移动端与跨平台](#十二移动端与跨平台) | 4 种 | PWA、React Native、Flutter、响应式 |
| [十三、JavaScript 核心与 DOM](#十三javascript-核心与-dom) | 5 种 | 语言底层、浏览器 API、执行机制 |

---

## 一、AI 编程助手

> AI 编程助手是 Vibecoding 的核心入口，直接决定了开发体验的流畅度。AI 助手负责需求理解、代码生成、多文件协作、Bug 修复、代码审查全流程。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[Cursor]] | 7031 | AI 原生 IDE，Vibecoding 首选工具 |
| [[GitHub Copilot]] | 4686 | 老牌 AI 编程助手，生态成熟 |
| [[Windsurf]] | 5357 | 创新性 AI 编程工具，Cascade 智能体 |
| [[Cline]] | 5355 | 开源 AI 编程助手，VS Code 扩展 |
| [[Roo Code]] | 5675 | 开源 AI 编程助手，垂直替代方案 |
| [[Amazon Q Developer]] | 2627 | 企业级 AI 编程助手，AWS 深度集成 |
| [[Gemini Code Assist]] | 4602 | Google 生态 AI 编程助手 |

---

## 二、前端框架

> 前端框架是 UI 开发的根基。在 AI 辅助编程时代，React 凭借其生态统治力仍是最主流选择，但 Vue 的渐进式设计和 Svelte 的极致性能也各有适用场景。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[React]] | 5898 | 组件化 UI 开发标准，前端生态霸主 |
| [[Vue]] | 5778 | 渐进式框架，易学易用，中文社区活跃 |
| [[Angular]] | 4764 | 企业级框架，TypeScript 原生，全功能集成 |
| [[Svelte]] | 4097 | 编译时框架，零运行时开销 |
| [[Solid]] | 4472 | 响应式框架，性能极致 |

---

## 三、前端元框架

> 元框架在框架之上提供了路由、数据获取、SSR/SSG、部署等完整能力。Next.js 是当前最成熟的 React 元框架，配合 Vercel 可实现零配置部署。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[Next.js]] | 2492 | React 元框架，SSR/SSG/ISR 全能，Vibecoding 前端首选 |
| [[Nuxt]] | 2328 | Vue 元框架，SSR 优先，文件路由优雅 |
| [[Astro]] | 2258 | 内容优先框架，多岛屿架构，极致性能 |
| [[SvelteKit]] | 2261 | Svelte 全栈框架，端到端类型安全 |
| [[Remix]] | 2502 | Web 标准优先框架，渐进增强 |
| [[Vite]] | 2170 | 现代构建工具，HMR 极速 |
| [[shadcn/ui]] | 2369 | 可复制组件库，AI 友好设计 |
| [[Tailwind CSS]] | 3324 | 实用优先 CSS 框架，原子化样式 |

---

## 四、后端框架

> 后端框架的选择取决于团队技术栈偏好和项目需求。Node.js 生态以 Express/Fastify/NestJS 为代表，Python 生态以 FastAPI/Django/Flask 为代表，Bun 则以一体化运行时异军突起。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[Express]] | 3274 | Node.js Web 框架经典，简单灵活 |
| [[Fastify]] | 2581 | 高性能 Node.js 框架，插件系统 |
| [[NestJS]] | 3472 | 渐进式 Node.js 框架，Angular 风格 |
| [[FastAPI]] | 2117 | 现代 Python Web 框架，类型安全，AI 友好 |
| [[Django]] | 2118 | Python 全栈框架，大而全 |
| [[Flask]] | 2133 | Python 轻量框架，微服务首选 |
| [[Spring Boot]] | 3291 | Java 企业级框架，生态完整 |
| [[Bun]] | 6251 | JavaScript 运行时 + 工具链，一体化 |
| [[Hono]] | 3481 | 轻量极速 Web 框架，边缘计算友好 |

---

## 五、编程语言

> 编程语言是所有技术栈的根基。TypeScript 是前端和 AI 应用的首选，Python 在 AI/数据科学领域无可匹敌，Go 是云原生和高并发服务的首选。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[TypeScript]] | 4201 | JavaScript 超集，类型安全，前端标配 |
| [[Python]] | 2101 | AI/数据科学首选，后端快速开发 |
| [[Go]] | 2341 | 高并发服务器语言，K8s 生态 |
| [[Rust]] | 4405 | 系统级语言，性能与安全并重 |
| [[JavaScript]] | 2651 | Web 核心语言，Node.js 生态 |

---

## 六、数据库

> 数据库是数据的持久化层。关系型数据库（PostgreSQL/MySQL）擅长结构化数据，文档数据库（MongoDB）适合灵活 schema，内存数据库（Redis）用于高速缓存。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[PostgreSQL]] | 3183 | 功能最全的关系型数据库，开源领袖 |
| [[MySQL]] | 3231 | 最流行的开源关系型数据库 |
| [[SQLite]] | 2105 | 嵌入式数据库，单文件，极简部署 |
| [[MongoDB]] | 3109 | 文档数据库，JSON 原生，灵活 schema |
| [[Redis]] | 2882 | 内存数据库，高速缓存与消息队列 |
| [[Supabase]] | 4306 | Firebase 开源替代，PostgreSQL + 实时 |

---

## 七、云服务与 DevOps

> 云服务决定了应用的部署和运维方式。Vercel/Netlify 适合前端项目，Cloudflare 提供 CDN 和边缘计算，Docker 实现容器化，GitHub Actions 实现 CI/CD 自动化。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[Vercel]] | 2245 | 前端部署首选，Next.js 官方托管 |
| [[Netlify]] | 3811 | 前端部署与 JAMstack 托管 |
| [[Cloudflare]] | 3796 | CDN + 边缘计算 + Workers |
| [[Docker]] | 2788 | 容器化标准，隔离与一致 |
| [[GitHub Actions]] | 4099 | CI/CD 自动化，GitHub 原生 |
| [[Railway]] | 2107 | 现代应用托管，极简部署体验 |
| [[Fly.io]] | 2085 | 边缘计算应用托管，全球低延迟 |

---

## 八、API 与全栈工具

> API 是前后端分离的桥梁。REST API 是通用标准，GraphQL 提供灵活的查询能力，tRPC 实现端到端类型安全。Prisma/Drizzle 等 ORM 提供了类型安全的数据库操作，Convex/Liveblocks 提供了实时能力。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[REST API]] | 3643 | Web API 设计规范与最佳实践 |
| [[GraphQL]] | 3222 | 数据查询语言，按需获取 |
| [[tRPC]] | 2820 | 端到端类型安全 API，TypeScript 原生 |
| [[Prisma]] | 2390 | TypeScript ORM，类型安全数据库操作 |
| [[Drizzle ORM]] | 2141 | 轻量 TypeScript ORM，SQL-like 语法 |
| [[Convex]] | 2859 | 实时后端平台，替代 Firebase |
| [[Clerk]] | 2220 | 身份认证全栈方案 |
| [[Kinde]] | 2262 | 开发者友好身份认证 |
| [[Liveblocks]] | 2650 | 实时协作基础设施 |
| [[WebSocket]] | 2817 | 实时双向通信协议 |
| [[API 设计规范]] | 3944 | RESTful 设计规范与最佳实践 |

---

## 九、CSS 与设计体系

> CSS 是前端视觉表现的核心。现代 CSS 包含 Flexbox/Grid 布局系统、Tailwind CSS 原子化框架、设计 token 主题系统，以及 Container Queries、:has()、Cascade Layers 等最新特性。在 AI 编程时代，CSS 的重要性不降反升——AI 可以生成结构，但视觉细节和响应式行为仍需开发者掌控。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[CSS 总览]] | 3626 | CSS 生态全景图、设计系统架构 |
| [[Flexbox 与 Grid]] | 3393 | 一维/二维布局核心，经典模式与实战代码 |
| [[Tailwind CSS]] | 3324 | 原子化样式框架，Vibecoding 原生工具 |
| [[CSS 动画]] | 2376 | 过渡、关键帧、View Transitions、性能优化 |
| [[现代 CSS 特性]] | 2738 | Container Queries、Cascade Layers、:has()、色彩函数 |
| [[PostCSS]] | 3523 | CSS 转换、增强、自动化工具链 |
| [[设计系统]] | 3101 | Design Token、组件化样式隔离、主题切换 |
| [[CSS 性能优化]] | 2234 | 渲染流水线、重排重绘、合成层优化 |
| [[Tailwind CSS 深度指南]] | 557 | Tailwind 配置、主题、组件化最佳实践 |
| [[PostCSS 与预处理]] | 3523 | Sass/Less、PostCSS 插件生态 |

---

## 十、Markdown 与文档工具

> Markdown 是 Vibecoding 时代的文档标准。从知识管理（Obsidian）、技术博客（Astro/VitePress）、API 文档（Mintlify/Scalar）到幻灯演示（Slidev），Markdown 生态覆盖了技术写作的全场景。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[Markdown 总览]] | 2593 | Markdown 语法体系、工具链全景 |
| [[Obsidian]] | 3460 | 双向链接、知识图谱、OFM 语法 |
| [[VitePress]] | 2781 | 静态文档站点生成器 |
| [[API 文档工具]] | 2144 | OpenAPI、Mintlify、Scalar |
| [[MDX]] | 3158 | Markdown + JSX，可嵌入 React 组件 |

---

## 十一、构建工具与工程化

> 构建工具将源码转换为可运行的产物。Vite 是当前最快的开发服务器和构建工具，Webpack 仍是复杂项目的主流选择，Turbopack/Rspack 则代表了 Rust 原生打包的未来方向。工程化还包括包管理、Monorepo、Linting 等环节。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[构建工具总览]] | 2464 | 构建工具生态、Vite/Webpack/Rollup 详解 |
| [[Vite 深度指南]] | 2808 | 即时服务、插件生态、配置详解 |
| [[Turbopack 与 Rspack]] | 3887 | Rust 原生打包，10x 性能提升 |
| [[包管理器]] | 2236 | npm/pnpm/yarn、workspace、lockfile |
| [[Turborepo 与 Monorepo]] | 3803 | Monorepo 构建加速 |
| [[代码质量工具]] | 2768 | ESLint、Prettier、Git Hooks |

---

## 十二、移动端与跨平台

> 移动端覆盖能力是产品化的关键。PWA 以最低成本覆盖移动 Web，React Native/Flutter 实现原生应用，Capacitor 可将 Web 项目转化为原生 App。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[移动端总览]] | 2297 | 移动端开发全景图、技术路线图 |
| [[PWA 深度指南]] | 3738 | Service Worker、Manifest、离线能力 |
| [[React Native 与 Expo]] | 3450 | 跨平台移动开发，Expo 工具链 |

---

## 十三、JavaScript 核心与 DOM

> JavaScript 是前端开发的灵魂语言。理解 JS 执行上下文、事件循环、闭包、异步编程、DOM/BOM API 等底层机制，是从「会用框架」到「理解原理」的跃迁，也是解决疑难 Bug 和性能问题的前提。

| 技术 | 行数 | 核心定位 |
|------|------|---------|
| [[JavaScript 核心]] | 5393 | ES2024+ 新特性、执行上下文、异步编程 |
| [[事件循环]] | 5321 | Event Loop、Microtask、RAF、Worker |
| [[DOM 与 BOM]] | 2920 | 节点操作、BOM、Intersection Observer |
| [[模块系统]] | 4919 | ES Modules、CommonJS、动态导入 |

---

## Vibecoding 实践路线图

### L1: 快速原型（1-2 小时）

```
Cursor/AI Coding Assistant
  + Next.js (App Router)
  + Tailwind CSS + shadcn/ui
  + Supabase (Auth + Database)
  + Vercel (Deploy)
```

> 适用于：MVP、PoC、创意验证

### L2: 生产级应用（1-2 天）

```
GitHub Copilot / Windsurf
  + Next.js (全栈)
  + PostgreSQL / MongoDB
  + Prisma / Drizzle ORM
  + Docker
  + Cloudflare (CDN + Workers)
  + GitHub Actions (CI/CD)
```

> 适用于：初创产品、内部工具、SaaS

### L3: 企业级架构（1-2 周）

```
AI Coding (多工具协作)
  + 前端: Next.js / Nuxt + shadcn/ui + Tailwind CSS
  + 后端: FastAPI / NestJS / Spring Boot
  + 数据库: PostgreSQL + Redis
  + CSS: 设计系统 + Design Tokens
  + 文档: Obsidian + Quartz
  + DevOps: Docker + GitHub Actions + Kubernetes
  + 监控: Cloudflare Analytics
```

> 适用于：商业产品、企业系统、高并发应用

### L4: AI 原生应用（1-3 天）

```
Cursor + Claude/GPT-5
  + Next.js + Vercel AI SDK
  + TypeScript + Zod
  + OpenAI / Anthropic / Google AI API
  + Supabase (存储 + 实时)
  + Tailwind CSS + shadcn/ui
  + 部署: Vercel
```

> 适用于：AI 聊天机器人、智能助手、内容生成工具

---

## 技术选型决策树

```
是否有复杂 UI 交互？
  ├─ 是 → React (Next.js) 或 Vue (Nuxt)
  └─ 否 → Astro 或纯 HTML + Tailwind

是否需要实时功能？
  ├─ 是 → WebSocket / Convex / Liveblocks
  └─ 否 → REST API / tRPC

数据复杂度如何？
  ├─ 结构化 → PostgreSQL / MySQL
  ├─ 文档型 → MongoDB
  └─ 简单 KV → SQLite / Redis

团队技术栈偏好？
  ├─ JS/TS 全栈 → Next.js + Fastify/Express
  ├─ Python → FastAPI / Django
  └─ Java → Spring Boot

样式方案？
  ├─ AI 快速开发 → Tailwind CSS + shadcn/ui
  ├─ 设计系统 → 自定义 CSS + Design Tokens
  └─ 两者结合 → Tailwind + 自定义组件

部署需求？
  ├─ 快速上线 → Vercel / Netlify / Railway
  ├─ 边缘计算 → Cloudflare Workers / Fly.io
  └─ 自托管 → Docker + VPS
```

---

## 附录：生态工具推荐

### AI 工具链

| 用途 | 工具 | 说明 |
|------|------|------|
| 代码生成 | Claude / GPT-5 | 最强代码能力 |
| AI 搜索 | Perplexity / Phind | 技术问题检索 |
| 代码审查 | Cursor Agent | 多文件重构 |
| 文档生成 | Mintlify | AI 文档生成 |
| AI 对话 | Vercel AI SDK | 全栈 AI 应用框架 |

### CSS 与设计工具链

| 用途 | 工具 | 说明 |
|------|------|------|
| 样式框架 | Tailwind CSS | 原子化实用优先 |
| 组件库 | shadcn/ui | 可复制组件，AI 友好 |
| 设计系统 | Storybook | 组件可视化文档 |
| 动画 | Framer Motion | React 动画库 |
| 图标 | Lucide / Heroicons | 开源图标集 |
| 色彩 | OKLCH / color-mix | 现代色彩函数 |

### 协作与监控

| 用途 | 工具 | 说明 |
|------|------|------|
| 版本控制 | Git + GitHub | 代码协作 |
| 域名/DNS | Cloudflare | DNS + CDN |
| 监控告警 | Sentry / Grafana | 错误追踪 + 指标 |
| 性能分析 | Lighthouse / PageSpeed | 前端性能优化 |
| CI/CD | GitHub Actions | 自动化流水线 |
| 文档托管 | Vercel / Netlify | 零配置部署 |

---

## 统计总览

| 类别 | 技术数量 | 平均行数 | 行数范围 |
|------|---------|---------|----------|
| **一、AI 编程助手** | 7 种 | 4,833 | 2,627 - 7,031 |
| **二、前端框架** | 5 种 | 4,904 | 4,097 - 5,898 |
| **三、前端元框架** | 8 种 | 2,700 | 2,170 - 3,324 |
| **四、后端框架** | 9 种 | 3,137 | 2,117 - 6,251 |
| **五、编程语言** | 5 种 | 3,142 | 2,101 - 4,405 |
| **六、数据库** | 6 种 | 3,118 | 2,105 - 4,306 |
| **七、云服务与 DevOps** | 7 种 | 2,947 | 2,085 - 4,099 |
| **八、API 与全栈工具** | 11 种 | 2,808 | 2,141 - 3,944 |
| **九、CSS 与设计体系** | 10 种 | 2,844 | 557 - 3,626 |
| **十、Markdown 与文档工具** | 7 种 | 2,852 | 2,144 - 3,460 |
| **十一、构建工具与工程化** | 7 种 | 2,837 | 557 - 3,887 |
| **十二、移动端与跨平台** | 4 种 | 3,162 | 2,297 - 3,738 |
| **十三、JavaScript 核心与 DOM** | 5 种 | 4,538 | 2,920 - 5,393 |
| **合计** | **88 种技术** | **3,419** | **557 - 7,031** |

---

> [!TIP]
> **Vibecoding 推荐栈**：Cursor（IDE）+ Next.js（框架）+ Tailwind CSS（样式）+ shadcn/ui（组件）+ TypeScript（类型）+ Prisma（ORM）+ PostgreSQL（数据库）+ Supabase（认证/存储）+ Vercel（部署）。这套组合覆盖了从创意验证到生产部署的完整流程。

---

## 关联文档

- [[大模型调用]] — AI 编程助手的调用方式与提示词工程
- [[Cursor 使用指南]] — AI 原生 IDE 的进阶技巧
- [[TypeScript 实战]] — 类型安全的最佳实践
- [[Next.js 全栈开发]] — 从原型到生产的完整流程
- [[设计系统构建]] — 从 Design Token 到组件库
