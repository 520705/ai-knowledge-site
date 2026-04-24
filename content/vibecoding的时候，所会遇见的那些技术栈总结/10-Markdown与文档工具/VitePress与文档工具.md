# VitePress 与文档工具深度指南

> VitePress 是 Vue 团队打造的静态站点生成器，专为技术文档设计。本文深入对比 VitePress、Docusaurus、GitBook、Slidev 等主流文档工具，提供完整配置指南和选型建议。

## 目录

- [[#VitePress 核心特性]]
- [[#VitePress 完整配置]]
- [[#Docusaurus 对比分析]]
- [[#GitBook 与 Slidev]]
- [[#工具选型决策树]]
- [[#迁移与最佳实践]]
- [[#相关资源]]

---

## VitePress 核心特性

### VitePress 架构概览

VitePress 基于以下技术栈构建：

| 层级 | 技术选型 | 作用 |
|------|---------|------|
| 构建工具 | Vite | 极速开发服务器和构建 |
| 框架 | Vue 3 | 组件化和响应式能力 |
| Markdown | markdown-it | Markdown 解析 |
| 样式 | CSS Variables | 主题定制 |

```bash
# 创建 VitePress 项目
npm create vitepress@latest my-docs
cd my-docs
npm install
```

```typescript
// .vitepress/config.ts
import { defineConfig } from 'vitepress'

export default defineConfig({
  title: '我的文档',
  description: '使用 VitePress 构建的精美文档',
  
  themeConfig: {
    logo: '/logo.svg',
    nav: [
      { text: '指南', link: '/guide/' },
      { text: 'API', link: '/api/' },
    ],
  },
})
```

### VitePress vs VuePress

VitePress 和 VuePress 虽然名称相似，但定位不同：

| 特性 | VitePress | VuePress |
|------|-----------|----------|
| 目标场景 | 技术文档（轻量级） | 复杂文档系统 |
| 默认主题 | 更现代、性能更好 | 功能丰富 |
| 主题定制 | 需从头开始 | 继承或扩展 |
| Vue 依赖 | 可选 | 必须 |
| 构建速度 | 极快（基于 Vite） | 较慢 |
| Markdown 扩展 | 内置 | 需插件 |

---

## VitePress 完整配置

### 基础配置

```typescript
// .vitepress/config.ts
import { defineConfig } from 'vitepress'

export default defineConfig({
  // 站点元数据
  title: 'AI Engineer Handbook',
  description: 'AI 工程帅实践指南与工具手册',
  lang: 'zh-CN',

  // 基础路径配置
  base: '/',  // 部署到域名的根路径
  
  // Markdown 配置
  markdown: {
    lineNumbers: true,
    theme: {
      light: 'github-light',
      dark: 'github-dark',
    },
    container: {
      tipLabel: '提示',
      warningLabel: '警告',
      dangerLabel: '危险',
      infoLabel: '信息',
      detailsLabel: '详情',
    },
  },

  // Head 标签配置
  head: [
    ['link', { rel: 'icon', href: '/favicon.ico' }],
    ['meta', { name: 'theme-color', content: '#3c8772' }],
    ['meta', { property: 'og:type', content: 'website' }],
  ],

  // 国际化配置
  locales: {
    root: {
      label: '简体中文',
      lang: 'zh-CN',
    },
    en: {
      label: 'English',
      lang: 'en',
      link: '/en/',
    },
  },
})
```

### 导航栏配置

```typescript
// .vitepress/config.ts
export default defineConfig({
  themeConfig: {
    // 顶部导航栏
    nav: [
      // 简单文本链接
      { text: '指南', link: '/guide/introduction' },
      { text: '变更日志', link: '/changelog' },
      
      // 下拉菜单
      {
        text: '版本',
        items: [
          { text: 'v2.x (最新)', link: '/guide/' },
          { text: 'v1.x', link: '/v1/' },
          { text: 'v0.x', link: '/v0/' },
        ],
      },
      
      // 带图标的链接
      {
        text: 'GitHub',
        link: 'https://github.com/vuejs/vitepress',
        icon: {
          svg: '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/></svg>',
        },
      },
    ],
  },
})
```

### 侧边栏配置

```typescript
// .vitepress/config.ts
export default defineConfig({
  themeConfig: {
    // 多层级侧边栏
    sidebar: {
      '/guide/': [
        {
          text: '入门',
          collapsed: false,
          items: [
            { text: '简介', link: '/guide/introduction' },
            { text: '快速开始', link: '/guide/getting-started' },
            { text: '目录结构', link: '/guide/directory' },
          ],
        },
        {
          text: '核心概念',
          collapsed: true,
          items: [
            { text: '配置系统', link: '/guide/configuration' },
            { text: '主题开发', link: '/guide/theming' },
            { text: '插件系统', link: '/guide/plugins' },
          ],
        },
        {
          text: '进阶指南',
          collapsed: true,
          items: [
            { text: '静态资源', link: '/guide/assets' },
            { text: '国际化', link: '/guide/i18n' },
            { text: '部署', link: '/guide/deployment' },
          ],
        },
      ],
      
      // API 文档侧边栏
      '/api/': [
        {
          text: 'API 参考',
          items: [
            { text: 'CLI', link: '/api/cli' },
            { text: '配置', link: '/api/configuration' },
            { text: 'Frontmatter', link: '/api/frontmatter' },
          ],
        },
      ],
    },
    
    // 其他配置
    outline: {
      level: [2, 3],
      label: '目录',
    },
    
    lastUpdated: {
      text: '最后更新',
      formatOptions: {
        dateStyle: 'short',
        timeStyle: 'short',
      },
    },
    
    editLink: {
      pattern: 'https://github.com/org/repo/edit/main/docs/:path',
      text: '在 GitHub 上编辑此页',
    },
  },
})
```

### 主题定制

```typescript
// .vitepress/theme/index.ts
import { h } from 'vue'
import type { Theme } from 'vitepress'
import DefaultTheme from 'vitepress/theme'
import './custom.css'

// 自定义组件
import Announcement from './components/Announcement.vue'
import CodeGroup from './components/CodeGroup.vue'
import Badge from './components/Badge.vue'

export default {
  extends: DefaultTheme,
  
  Layout: () => {
    return h(DefaultTheme.Layout, null, {
      'home-hero-image': () => h('img', {
        src: '/hero.svg',
        alt: 'Hero Image',
        class: 'custom-hero',
      }),
    })
  },
  
  enhanceApp({ app, router, siteData }) {
    // 注册全局组件
    app.component('Announcement', Announcement)
    app.component('CodeGroup', CodeGroup)
    app.component('Badge', Badge)
    
    // 注册自定义指令
    app.directive('loading', {
      mounted(el, binding) {
        if (binding.value) {
          el.classList.add('is-loading')
        } else {
          el.classList.remove('is-loading')
        }
      },
    })
  },
} satisfies Theme
```

```css
/* .vitepress/theme/custom.css */
:root {
  --vp-c-brand-1: #3c8772;
  --vp-c-brand-2: #2d6a56;
  --vp-c-brand-3: #1e4d3c;
  
  --vp-c-bg: #ffffff;
  --vp-c-bg-soft: #f6f6f6;
  
  --vp-font-family-base: 'Inter', system-ui, sans-serif;
  --vp-font-family-mono: 'JetBrains Mono', monospace;
}

/* 自定义 Hero 区域 */
.VPHome .VPHero .name {
  font-size: 3.5rem;
  background: linear-gradient(135deg, #3c8772, #2d6a56);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* 代码块样式 */
.vp-code-group .tabs input:checked + label {
  border-color: var(--vp-c-brand-1);
  color: var(--vp-c-brand-1);
}

/* 自定义容器 */
.custom-tip {
  border-left: 4px solid var(--vp-c-brand-1);
  background: var(--vp-c-bg-soft);
}
```

### Frontmatter 配置

```yaml
---
title: 快速开始
description: 学习如何快速配置 VitePress
editLink: true
lastUpdated: true

# 页面 Meta
meta:
  - name: description
    content: VitePress 快速开始教程
  - name: keywords
    content: vitepress, documentation, vue

# 上次更新时间
lastUpdated:
  text: 最后更新于
  formatOptions:
    dateStyle: long

# 上一页/下一页
prev: 
  text: 简介
  link: /guide/introduction
next:
  text: 配置
  link: /guide/configuration

# 大纲配置
outline: [2, 3]

# 目录深度
toc: true
---
```

---

## Docusaurus 对比分析

### Docusaurus 核心特性

Docusaurus 是 Meta 维护的文档框架：

```bash
# 创建 Docusaurus 项目
npx create-docusaurus@latest my-website classic

# 启动开发服务器
npm start
```

```javascript
// docusaurus.config.js
export default {
  title: 'My Site',
  tagline: 'The tagline for my site',
  url: 'https://your-docusaurus-site.example.com',
  baseUrl: '/',
  
  // i18n 配置
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'zh-CN'],
  },
  
  // 预设和插件
  presets: [
    [
      'classic',
      {
        docs: {
          sidebarPath: './sidebars.js',
          editUrl: 'https://github.com/facebook/docusaurus/edit/main/',
        },
        blog: {
          showReadingTime: true,
          feedOptions: {
            type: ['rss', 'atom'],
          },
        },
        theme: {
          customCss: './src/css/custom.css',
        },
      },
    ],
  ],
  
  // 插件配置
  plugins: [
    [
      '@docusaurus/plugin-ideal-image',
      {
        quality: 85,
        max: 1030,
        min: 640,
        steps: 3,
      },
    ],
  ],
  
  // 主题配置
  themeConfig: {
    image: 'img/social-card.jpg',
    navbar: {
      title: 'My Site',
      logo: {
        alt: 'My Site Logo',
        src: 'img/logo.svg',
      },
      items: [
        {
          type: 'docSidebar',
          sidebarId: 'tutorialSidebar',
          position: 'left',
          label: 'Tutorial',
        },
        { to: '/blog', label: 'Blog', position: 'left' },
        {
          href: 'https://github.com/facebook/docusaurus',
          label: 'GitHub',
          position: 'right',
        },
      ],
    },
    footer: {
      style: 'dark',
      links: [
        {
          title: 'Docs',
          items: [
            { label: 'Tutorial', to: '/docs/intro' },
          ],
        },
      ],
      copyright: `Copyright © ${new Date().getFullYear()} My Project.`,
    },
    prism: {
      theme: prismThemes.github,
      darkTheme: prismThemes.dracula,
    },
  },
};
```

### 功能对比矩阵

| 功能 | VitePress | Docusaurus |
|------|-----------|------------|
| 框架基础 | Vue 3 | React |
| 构建速度 | 极快 | 中等 |
| 包体积 | 轻量 (~100KB) | 较重 (~500KB) |
| 默认主题美观度 | 现代简洁 | 功能丰富 |
| 博客支持 | 需手动配置 | 内置 |
| 版本管理 | 无内置 | 内置版本切换 |
| i18n | 社区插件 | 内置 |
| MDX 支持 | 基础 | 完整 |
| 搜索 | 本地搜索/Algolia | Algolia/Docusaurus Search |
| React 集成 | 需额外配置 | 原生支持 |
| 插件系统 | 有限的 Vite 插件 | 丰富的插件生态 |

### 性能对比

| 指标 | VitePress | Docusaurus |
|------|-----------|------------|
| 首次内容绘制 (FCP) | ~0.8s | ~1.2s |
| 最大内容绘制 (LCP) | ~1.2s | ~1.8s |
| JavaScript 体积 | ~150KB | ~400KB |
| 构建时间 (100 页) | ~5s | ~15s |
| 开发服务器启动 | <1s | ~3s |

---

## GitBook 与 Slidev

### GitBook

GitBook 提供云端托管的文档服务：

```yaml
# gitbook.yaml
root: ./docs

structure:
  readme: 01-introduction/README.md
  summary: SUMMARY.md
  globs: "**/*.md"

html:
  theme: default
  variables:
    themeColor: '#3c8772'

plugins:
  - mathjax
  -ermaid
  - github-buttons
  - collapsible-chapters
  
  config:
    mathjax:
      autoNumber: true
    mermaid:
      theme: default
```

```markdown
# 文档结构示例

## 第一章：入门
- [[安装与配置|01-installation/README.md]]
- [[快速开始|02-quickstart/README.md]]

## 第二章：进阶
- [[主题定制|03-theming/README.md]]
- [[插件开发|04-plugins/README.md]]
```

### Slidev

Slidev 是为开发者打造的演示工具：

```bash
# 创建 Slidev 项目
npm create slidev@latest my-presentation

# 启动编辑模式
npm run dev

# 构建静态文件
npm run build
```

```markdown
---
theme: seriph
background: https://picsum.photos/1920/1080
class: text-center
---

# 欢迎使用 Slidev

使用 Markdown 编写幻灯片，嵌入 Vue 组件。

<!-- v-click -->
点击后显示的内容

---
layout: two-cols
---

# 左侧内容

- 支持 Markdown
- 支持 Vue 组件
- 支持代码高亮

::right::

# 右侧内容

```python
def hello():
    print("Hello, Slidev!")
```
```

```vue
<!-- components/InteractiveCounter.vue -->
<template>
  <div class="counter">
    <button @click="count--">-</button>
    <span>{{ count }}</span>
    <button @click="count++">+</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>
```

---

## 工具选型决策树

```
文档工具选型决策树
│
├─ 是否需要 React 生态？
│   ├─ 是 → Docusaurus
│   └─ 否 ↓
│
├─ 是否需要博客功能？
│   ├─ 是 → VitePress + 插件 或 Docusaurus
│   └─ 否 ↓
│
├─ 是否优先考虑性能？
│   ├─ 是 → VitePress
│   └─ 否 ↓
│
├─ 是否需要云端托管/协作？
│   ├─ 是 → GitBook
│   └─ 否 ↓
│
└─ 需要幻灯片演示？
    ├─ 是 → Slidev
    └─ 最终选择 → VitePress
```

| 场景 | 推荐工具 |
|------|---------|
| 库/框架文档 | VitePress / Docusaurus |
| 产品文档 | GitBook / VitePress |
| 技术博客 | Docusaurus / VitePress |
| 演示幻灯片 | Slidev |
| 开源项目文档 | VitePress / Docusaurus |
| 内部文档 | VitePress / GitBook |

---

## 迁移与最佳实践

### 从 VuePress 迁移到 VitePress

```typescript
// vitepress.config.ts (新)
// 替代 vuepress/config.js

import { defineConfig } from 'vitepress'

export default defineConfig({
  // base 配置保持一致
  base: '/',
  
  // dest 改为 outDir
  outDir: './dist',
  
  // themeConfig 替代原有的 themeConfig
  themeConfig: {
    nav: [...],
    sidebar: [...],
  },
  
  // markdown 配置
  markdown: {
    lineNumbers: true,
  },
})
```

### 性能优化建议

> [!TIP] VitePress 性能优化
> - 使用 `vitepress/dist/client/theme-default/style/vars.css` 覆盖变量
> - 对大图片使用 WebP 格式
> - 启用 Gzip/Brotli 压缩
> - 利用预加载优化关键资源

---

## 相关资源

- [[VitePress 官方文档]]
- [[Docusaurus 官方文档]]
- [[Slidev 官方文档]]
- [[GitBook 官方文档]]
- [[VuePress vs VitePress]]
- [[静态站点生成器对比]]

---

*本文档由 [[归愚知识系统]] 自动生成*

---

## VitePress 高级配置

### 搜索功能配置

VitePress 支持多种搜索方案，从内置的简单搜索到专业的 Algolia DocSearch：

```typescript
// .vitepress/config.ts
export default defineConfig({
  // 方案1: MiniSearch 本地搜索（默认，免费）
  themeConfig: {
    search: {
      provider: 'local', // 或 'local'
      
      options: {
        // 搜索结果数量
        maxResults: 15,
        
        // 搜索结果描述长度
        maxSuggestions: 10,
        
        // 搜索归类
        locales: {
          zh: {
            placeholder: '搜索文档',
            translations: {
              button: {
                buttonText: '搜索',
                buttonAriaLabel: '搜索文档',
              },
            },
          },
        },
        
        // 排除页面
        excludeCollections: ['api'],
      },
    },
  },
  
  // 方案2: Algolia DocSearch（专业，功能强大）
  themeConfig: {
    search: {
      provider: 'algolia',
      
      options: {
        // Algolia 配置
        appId: 'YOUR_APP_ID',
        apiKey: 'YOUR_SEARCH_API_KEY',
        indexName: 'YOUR_INDEX_NAME',
        
        // 搜索选项
        placeholder: '搜索文档',
        translations: {
          modal: {
            searchBox: {
              resetButtonTitle: '清除搜索',
              cancelButtonText: '取消',
            },
            startScreen: {
              recentSearchesTitle: '最近搜索',
              noRecentSearchesText: '没有最近的搜索',
              saveRecentSearchButtonTitle: '保存此搜索',
              removeRecentSearchButtonTitle: '删除此搜索',
              favoriteSearchesTitle: '收藏',
              removeFavoriteSearchButtonTitle: '移除收藏',
            },
            errorScreen: {
              titleText: '无法获取结果',
              helpText: '你可能需要检查网络连接',
            },
            footer: {
              selectText: '选择',
              navigateText: '导航',
              closeText: '关闭',
              searchByText: '搜索提供者',
            },
            noResultsScreen: {
              noResultsText: '没有找到相关结果',
              suggestedQueryText: '尝试搜索',
              reportMissingResultsText: '搜索结果缺失？',
              reportMissingResultsLinkText: '反馈',
            },
          },
        },
      },
    },
  },
})
```

### 社交链接与贡献者

```typescript
// .vitepress/config.ts
export default defineConfig({
  themeConfig: {
    // 社交链接
    socialLinks: [
      { icon: 'github', link: 'https://github.com/vuejs/vitepress' },
      { icon: 'twitter', link: 'https://twitter.com/vuejs' },
      { icon: 'discord', link: 'https://discord.gg/vue' },
      { 
        icon: 'slack', 
        link: 'https://join.slack.com/t/vitepress/shared_invite/xxx',
        ariaLabel: 'Slack',
      },
    ],
    
    // 文档贡献者
    editLink: {
      pattern: 'https://github.com/vuejs/vitepress/edit/main/docs/:path',
      text: '在 GitHub 上编辑此页',
    },
    
    lastUpdated: {
      enabled: true,
      text: '最后更新',
      formatOptions: {
        dateStyle: 'medium',
        timeStyle: 'short',
      },
    },
    
    // 文档历史
    docFooter: {
      prev: '上一页',
      next: '下一页',
    },
    
    // 语料库链接
    openInNewBrowser: true,
  },
})
```

### 代码块高级功能

```typescript
// .vitepress/config.ts
export default defineConfig({
  markdown: {
    // 代码行高亮
    lineNumbers: true,
    
    // 代码块配置
    codeCopyButtonTitle: '复制代码',
    
    // 代码主题
    theme: {
      light: 'github-light',
      dark: 'github-dark',
    },
    
    // 容器类型
    container: {
      type: ['tip', 'warning', 'danger', 'info', 'details'],
      
      // 自定义容器
      customContainers: {
        callout: {
          collapsed: false,
          defaultTitle: '',
        },
        video: true,
        codeGroup: true,
      },
    },
    
    // 代码块语言别名
    languageAliases: {
      js: 'javascript',
      ts: 'typescript',
      vue: 'html',
      sh: 'bash',
      shell: 'bash',
    },
  },
})

// Frontmatter 中配置代码块
---
title: 示例代码
highlights: [3, 5, 7]  // 高亮特定行
lineNumbers: true
---

```typescript
const a = 1;    // 不会被高亮
const b = 2;    // 高亮（第3行）
const c = 3;
const d = 4;    // 高亮（第5行）
const e = 5;
const f = 6;    // 高亮（第7行）
const g = 7;
```
```

### MDX 增强支持

虽然 VitePress 原生支持 MDX，但可以通过插件获得更好的体验：

```typescript
// .vitepress/config.ts
import { defineConfig } from 'vitepress'
import mdx from 'vite-plugin-mdx'

export default defineConfig({
  vite: {
    plugins: [
      mdx({
        MDXProvider: require.resolve('@mdx-js/react'),
      }),
    ],
  },
  
  markdown: {
    // 启用更多 MDX 特性
    remarkPlugins: [
      require('remark-frontmatter'),
      require('remark-mdx-frontmatter'),
      [require('remark-gfm'), { singleLineCodeFrames: true }],
    ],
    
    rehypePlugins: [
      require('rehype-slug'),
      require('rehype-autolink-headings'),
      [require('rehype-prism-plus'), { ignoreMissing: true }],
    ],
  },
})

// .vitepress/theme/index.ts
import DefaultTheme from 'vitepress/theme'
import MDXProvider from '@mdx-js/react'
import './mdx.css'

export default {
  extends: DefaultTheme,
  
  enhanceApp({ app }) {
    // 注册 MDX 组件
    app.use(MDXProvider)
  },
}
```

### 多语言与国际化

```typescript
// .vitepress/config.ts
export default defineConfig({
  // 基础语言配置
  lang: 'zh-CN',
  title: 'VitePress',
  description: '基于 Vite 的下一代文档框架',
  
  // 多语言配置
  locales: {
    // 根语言
    root: {
      label: '简体中文',
      lang: 'zh-CN',
      link: '/',
      
      themeConfig: {
        // 中文专属导航
        nav: [
          { text: '指南', link: '/zh/guide/' },
          { text: 'API', link: '/zh/api/' },
        ],
        
        sidebar: {
          '/zh/guide/': [
            /* ... */
          ],
        },
      },
    },
    
    // 英文
    en: {
      label: 'English',
      lang: 'en',
      link: '/en/',
      
      themeConfig: {
        outlineTitle: 'On this page',
        lastUpdated: 'Last Updated',
        editLink: {
          pattern: 'https://github.com/vuejs/vitepress/edit/main/docs/:path',
          text: 'Edit this page on GitHub',
        },
      },
    },
    
    // 日文
    ja: {
      label: '日本語',
      lang: 'ja',
      link: '/ja/',
      
      themeConfig: {
        outlineTitle: 'このページの内容',
      },
    },
  },
})

// 目录结构
docs/
├── index.md              # 中文首页
├── guide/
│   └── getting-started.md
├── en/
│   ├── index.md         # 英文首页
│   ├── guide/
│   │   └── getting-started.md
│   └── api/
└── ja/
    ├── index.md         # 日文首页
    └── guide/
        └── getting-started.md
```

### 部署配置

```typescript
// .vitepress/config.ts
export default defineConfig({
  // 部署基础路径
  // GitHub Pages: '/repo-name/'
  // Netlify: '/' (自动检测)
  base: process.env.NODE_ENV === 'production' 
    ? '/vitepress/' 
    : '/',
  
  // 构建输出目录
  outDir: '.vitepress/dist',
  
  // 构建配置
  build: {
    // 目标浏览器
    target: 'es2015',
    
    // 滚动覆盖
    scrollOffset: 80,
    
    // 静态资源处理
    assetsInlineLimit: 4096, // 4kb 以下的资源内联
    
    // CSS 代码分割
    cssCodeSplit: true,
    
    // 压缩
    minify: 'terser',
    
    // 打包分析
    rollupOptions: {
      output: {
        manualChunks: {
          'vitepress': ['vitepress'],
          'shiki': ['shiki'],
        },
      },
    },
  },
  
  // 清理控制台警告
  ignoreDeadLinks: [
    // 忽略特定链接
    '/legacy',
    // 忽略外部链接检查
    (url) => url.startsWith('http'),
  ],
})
```

### 静态资源与图片

```typescript
// .vitepress/config.ts
export default defineConfig({
  srcDir: 'docs',
  
  // 图片配置
  images: {
    domains: [
      'picsum.photos',
      'images.unsplash.com',
      'cdn.your-domain.com',
    ],
    
    // 本地图片目录
    localDir: 'docs/.vitepress/public',
  },
  
  // 公共文件目录
  // 放在 .vitepress/public 下的文件会直接复制到输出目录
  // public/
  //   favicon.svg
  //   logo.png
  //   og-image.jpg
})

// 使用图片
// 文档中使用
// ![图片描述](/logo.png)
// ![图片描述](./logo.png)  // 相对路径

// Frontmatter 中使用
---
hero:
  image: /logo.png
  alt: Logo
---
```

---

## Docusaurus 深度配置

### 内容管理

Docusaurus 提供了强大的内容管理系统：

```javascript
// docs/tutorials/tutorial-basics/
├── _category_.json        # 分类配置
├── congratulations.md     # 文档页面
├── create-a-blog-post.md
├── create-a-document.md
├── create-a-page.md
├── deploy-your-site.md
├── markdown-features/
│   ├── _category_.json
│   ├── index.mdx
│   └── math-equations.png
└── sidebar-items.json    # 侧边栏配置
```

```json
// _category_.json
{
  "label": "Getting Started",
  "position": 2,
  "collapsible": true,
  "collapsed": false,
  "link": {
    "type": "generated-index",
    "description": "Learn the most important Docusaurus concepts..."
  },
  "className": "category-getting-started"
}
```

### 版本管理

Docusaurus 内置了完整的版本控制功能：

```javascript
// docusaurus.config.js
module.exports = {
  presets: [
    [
      'classic',
      {
        docs: {
          // 版本配置
          lastVersion: 'current',
          versions: {
            current: {
              label: '2.0 (Next)',
              path: 'next',
              banner: 'unmaintained',
            },
            '1.0.0': {
              label: '1.0.0',
              path: '1.0.0',
              badge: true,
              banner: 'none',
            },
          },
          
          // 版本降级提示
          showLastUpdateTime: true,
          showLastUpdateAuthor: true,
        },
      },
    ],
  ],
}
```

### 插件系统

```javascript
// 常用插件配置
plugins: [
  // SEO 优化
  [
    '@docusaurus/plugin-ideal-image',
    {
      quality: 85,
      max: 1920,
      min: 640,
      steps: 3,
      disableInDev: false,
    },
  ],
  
  // Google Analytics
  [
    '@docusaurus/plugin-google-gtag',
    {
      trackingID: 'G-XXXXXXXXXX',
      anonymizeIP: true,
    },
  ],
  
  // Sitemap
  [
    '@docusaurus/plugin-sitemap',
    {
      changefreq: 'weekly',
      priority: 0.5,
      ignorePatterns: ['/tags/**'],
      filename: 'sitemap.xml',
    },
  ],
  
  // PWA
  [
    '@docusaurus/plugin-pwa',
    {
      debug: true,
      offlineModeActivationStrategies: [
        'appInstalled',
        'standalone',
        'saveToOrigin',
      ],
      pwaHead: [
        { tagName: 'link', rel: 'icon', href: '/img/docusaurus.png' },
        { tagName: 'link', rel: 'manifest', href: '/manifest.json' },
        { tagName: 'meta', name: 'theme-color', content: '#3cddc9' },
      ],
    },
  ],
]
```

### Swizzling 组件

Docusaurus 允许通过 Swizzling 来自定义组件：

```bash
# 危险 Swizzle（会 fork 组件）
npm run swizzle DocVersionBanner -- --danger

# 安全 Swizzle（仅包装组件）
npm run swizzle SearchBar -- --wrap

# 查看可 Swizzle 的组件
npm run swizzle -- --list
```

```tsx
// src/theme/SearchBar/index.tsx
// 安全 Swizzle 示例
import React from 'react';
import SearchBar from '@theme-original/SearchBar';

export default function SearchBarWrapper(props) {
  // 添加自定义逻辑
  const handleSearch = (query) => {
    console.log('Searching for:', query);
    return originalSearch(query);
  };
  
  return (
    <div className="custom-search">
      <SearchBar {...props} />
      <div className="search-hints">
        提示：使用 "site:" 限定搜索范围
      </div>
    </div>
  );
}
```

---

## GitBook 高级配置

### GitBook CLI 配置

```yaml
# .gitbook.yaml
root: ./docs

# 目录结构
structure:
  readme: 01-introduction/README.md
  summary: SUMMARY.md
  globs:
    docs: "**/*.md"
    api: "docs/api/**/*.md"

# PDF 输出
pdf:
  enabled: true
  pageNumbers: true
  fontFamily: -apple-system, BlinkMacSystemFont
  fontSize: 12
  margin:
    top: 14
    bottom: 14
    left: 14
    right: 14

# 电子书输出
ebook:
  enabled: true
  tocLevel: 3

# 插件配置
plugins:
  - search-pro
  - advanced-emoji
  - github-buttons
  - anchors
  - codeblock-filename
  - toggle-sidebar
  - copy-code-button
  
  # 主题
  - theme-default

# 插件配置
pluginsConfig:
  github-buttons:
    buttons:
      - user: your-username
        repo: your-repo
        type: star
        size: small
  
  codeblock-filename:
    css: |
      .filename {
        background-color: #e6e6e6;
        padding: 5px 10px;
        border-radius: 3px;
      }
```

### GitBook 插件开发

```javascript
// plugins/my-plugin/index.js
module.exports = {
  // 书籍初始化时调用
  init: function(book) {
    console.log('Book initialized:', book.title);
  },
  
  // 页面渲染前调用
  beforePageRead: function(page) {
    // 可以修改页面内容
    page.content = preprocess(page.content);
    return page;
  },
  
  // 页面渲染后调用
  afterPageRead: function(page) {
    // 可以添加额外资源
    return page;
  },
  
  // 生成产物前调用
  beforeGenerate: function(book) {
    return book;
  },
  
  // 自定义钩子
  hooks: {
    'page:before.generate': function(page) {
      // 处理页面生成前
      return page;
    },
    'page:generate': function(page, block) {
      // 处理页面生成
      return page;
    },
  },
  
  // PDF 生成
  pdf: {
    embedFonts: true,
    pageCallback: function(pageNumber, totalPages) {
      return {
        text: `Page ${pageNumber} of ${totalPages}`,
        alignment: 'center',
      };
    },
  },
}
```

---

## Slidev 进阶教程

### 幻灯片主题开发

```typescript
// themes/my-theme/index.ts
export default {
  // 基础颜色
  colors: {
    primary: '#3c8772',
    secondary: '#2d6a56',
    accent: '#f59e0b',
    background: '#ffffff',
    text: '#1f2937',
  },
  
  // 字体
  fonts: {
    sans: 'Inter, sans-serif',
    mono: 'Fira Code, monospace',
    serif: 'Merriweather, serif',
  },
  
  // 动画
  animations: {
    enter: {
      fade: true,
      scale: true,
    },
    leave: {
      fade: true,
      scale: false,
    },
  },
  
  // 布局
  layouts: {
    // 自定义布局
    myLayout: {
      slots: ['title', 'content', 'footer'],
      // 布局样式
      css: `
        .layout-myLayout {
          @apply flex items-center justify-center h-full;
        }
      `,
    },
  },
  
  // 幻灯片默认样式
  slides: {
    background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
    backgroundFit: 'cover',
  },
}

// 使用自定义主题
---
theme: ./themes/my-theme
---
```

### Slidev 交互组件

```vue
<!-- components/InteractiveQuiz.vue -->
<template>
  <div class="quiz-container">
    <h3>{{ question }}</h3>
    
    <div class="options">
      <button
        v-for="(option, index) in options"
        :key="index"
        :class="[
          'option',
          {
            'selected': selected === index,
            'correct': showResult && index === correct,
            'incorrect': showResult && selected === index && index !== correct,
          }
        ]"
        @click="select(index)"
      >
        {{ option }}
      </button>
    </div>
    
    <div v-if="showResult" class="result">
      <p v-if="selected === correct">正确！</p>
      <p v-else>错误，正确答案是：{{ options[correct] }}</p>
      <button @click="$emit('next')">下一题</button>
    </div>
  </div>
</template>

<script setup>
defineProps({
  question: String,
  options: Array,
  correct: Number,
})

const selected = ref(null)
const showResult = ref(false)

function select(index) {
  selected.value = index
  showResult.value = true
}
</script>
```

```markdown
---
theme: seriph
---

# 测验时间

<v-quiz 
  question="VitePress 是由哪个团队开发的？"
  :options="['Vue团队', 'React团队', 'Angular团队', 'Svelte团队']"
  :correct="0"
/>

<!-- 使用自定义组件 -->
<InteractiveDemo />
```

### Slidev 录制与导出

```bash
# 录制幻灯片
npx slidev --record

# 导出为 PDF
npx slidev export

# 导出为图片
npx slidev export --format png

# 导出为 PPT
npx slidev export --format pptx

# 导出特定页面
npx slidev export --range 1,3,5-8
```

---

## 文档工具对比深度分析

### 详细功能矩阵

| 特性 | VitePress | Docusaurus | GitBook | Slidev |
|------|-----------|------------|---------|--------|
| **基础框架** | Vue 3 | React | 自研 | Vue 3 |
| **构建速度** | ⚡⚡⚡⚡⚡ | ⚡⚡⚡ | ⚡⚡ | ⚡⚡⚡⚡ |
| **默认包体积** | ~90KB | ~400KB | 云端 | ~120KB |
| **SSR 支持** | ✅ | ✅ | ❌ | ❌ |
| **静态生成** | ✅ | ✅ | ✅ | ✅ |
| **MDX 支持** | 基础 | 完整 | ❌ | ✅ |
| **主题系统** | CSS 变量 | React 组件 | YAML | Vue 组件 |
| **国际化** | 插件 | 内置 | 内置 | ❌ |
| **版本管理** | ❌ | ✅ | ✅ | ❌ |
| **搜索** | 本地/Algolia | Algolia/本地 | 内置 | ❌ |
| **博客系统** | 需插件 | 内置 | 内置 | ❌ |
| **评论系统** | 需集成 | 需集成 | 内置 | ❌ |
| **分析统计** | 需集成 | 需集成 | 内置 | ❌ |
| **协作编辑** | ❌ | ❌ | ✅ | ❌ |
| **私有部署** | ✅ | ✅ | ❌ | ✅ |
| **社区生态** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

### 性能对比数据

基于 100 页文档的测试数据：

| 指标 | VitePress | Docusaurus | GitBook CLI |
|------|-----------|------------|-------------|
| **开发服务器启动** | 0.8s | 3.2s | N/A |
| **热更新速度** | <50ms | ~200ms | N/A |
| **生产构建** | 4.5s | 18s | 6s |
| **首次内容绘制** | 0.6s | 1.4s | 0.8s |
| **JS Bundle** | 85KB | 380KB | 0KB (云端) |
| **CSS Bundle** | 35KB | 120KB | 0KB |
| ** Lighthouse 分数** | 98 | 92 | 95 |

### 成本对比

| 方案 | 基础设施成本 | 维护成本 | 适合场景 |
|------|-------------|---------|---------|
| **VitePress** | $5-20/月 (VPS) | 低 | 重视性能的团队 |
| **Docusaurus** | $10-50/月 | 中 | 需要复杂功能的团队 |
| **GitBook** | $0-15/月 | 极低 | 快速启动，无需运维 |
| **Slidev** | $0 (本地) | 低 | 演示文稿 |

---

## 迁移指南

### 从 VuePress 迁移到 VitePress

```typescript
// 1. 依赖更新
// package.json
{
  "devDependencies": {
    // 删除
    // "vuepress": "^1.9.0"
    
    // 添加
    "vitepress": "^1.0.0"
  }
}

// 2. 配置文件重命名
// .vuepress/config.js → .vitepress/config.ts
// .vuepress/theme/index.js → .vitepress/theme/index.ts

// 3. 配置迁移
// 旧版 (VuePress)
module.exports = {
  base: '/',
  title: 'My Site',
  description: 'Description',
  themeConfig: {
    nav: [...],
    sidebar: [...],
  },
}

// 新版 (VitePress)
import { defineConfig } from 'vitepress'

export default defineConfig({
  base: '/',
  title: 'My Site',
  description: 'Description',
  themeConfig: {
    nav: [...],
    sidebar: [...],
  },
})

// 4. Frontmatter 迁移
// 旧版
---
title: Title
navbar: true
sidebar: auto
---

// 新版
---
title: Title
---
// navbar 和 sidebar 现在通过 config 配置

// 5. 主题迁移
// 旧版
module.exports = {
  extend: '@vuepress/theme-default'
}

// 新版
import DefaultTheme from 'vitepress/theme'

export default {
  extends: DefaultTheme,
}
```

### 从 Jekyll/Hugo 迁移

```markdown
<!-- Jekyll 到 VitePress -->
<!-- _posts/2024-01-01-hello.md -->

---
layout: post
title: Hello World
date: 2024-01-01
categories: [tutorial]
---

# Hello World

Welcome to my blog!
```

转换为 VitePress 格式：

```markdown
<!-- docs/posts/hello-world.md -->

---
title: Hello World
date: 2024-01-01
categories:
  - tutorial
---

# Hello World

Welcome to my blog!
```

### 从 Notion 迁移

```typescript
// 使用 Notion to MDX 工具
import { Client } from '@notionhq/client';
import { toMarkdown } from '@notionapi/to-markdown';

async function migrateNotion() {
  const notion = new Client({ auth: process.env.NOTION_TOKEN });
  
  // 获取页面
  const page = await notion.pages.retrieve({ page_id: 'xxx' });
  
  // 获取块内容
  const blocks = await notion.blocks.children.list({ 
    block_id: 'xxx' 
  });
  
  // 转换为 Markdown
  let markdown = '';
  for (const block of blocks.results) {
    markdown += toMarkdown(block);
  }
  
  // 保存为 MDX
  await writeFile('docs/notion-export.mdx', markdown);
}
```

---

## 实战项目模板

### 开源项目文档模板

```typescript
// .vitepress/config.ts
export default defineConfig({
  title: 'My Awesome Project',
  description: 'The most awesome project ever',
  
  themeConfig: {
    // Logo 和 Hero
    logo: '/logo.svg',
    hero: {
      name: 'My Project',
      text: 'Build amazing things',
      tagline: 'Fast, reliable, and delightful',
      actions: [
        { theme: 'brand', text: '快速开始', link: '/guide/' },
        { theme: 'alt', text: 'GitHub', link: 'https://github.com' },
      ],
    },
    
    // 功能特性展示
    features: [
      {
        icon: '🚀',
        title: '极致性能',
        details: '基于 Vite，构建速度毫秒级',
      },
      {
        icon: '🎨',
        title: '精美主题',
        details: '开箱即用的现代设计',
      },
      {
        icon: '🔌',
        title: '插件系统',
        details: '丰富的插件生态',
      },
    ],
    
    // 导航
    nav: [
      { text: '指南', link: '/guide/' },
      { text: 'API', link: '/api/' },
      { text: '博客', link: '/blog/' },
      {
        text: '资源',
        items: [
          { text: 'Changelog', link: '/changelog' },
          { text: '贡献指南', link: '/contributing' },
        ],
      },
    ],
    
    // 社交链接
    socialLinks: [
      { icon: 'github', link: 'https://github.com' },
      { icon: 'twitter', link: 'https://twitter.com' },
    ],
    
    // 页脚
    footer: {
      message: '基于 MIT 协议发布',
      copyright: 'Copyright © 2024 My Project',
    },
  },
})
```

### 公司内部文档模板

```typescript
// .vitepress/config.ts
export default defineConfig({
  title: '内部文档中心',
  description: '公司内部技术文档平台',
  
  head: [
    ['link', { rel: 'icon', href: '/favicon.ico' }],
    ['meta', { name: 'robots', content: 'noindex,nofollow' }],
  ],
  
  themeConfig: {
    // 搜索配置
    search: {
      provider: 'local',
      options: {
        excludeCollections: ['api'],
      },
    },
    
    // 内部导航
    nav: [
      { text: '首页', link: '/' },
      { text: '技术文档', link: '/tech/' },
      { text: '产品手册', link: '/product/' },
      { text: '流程规范', link: '/process/' },
      {
        text: '工具',
        items: [
          { text: '代码规范', link: '/tools/linting' },
          { text: '部署指南', link: '/tools/deployment' },
        ],
      },
    ],
    
    // 侧边栏
    sidebar: {
      '/tech/': [
        {
          text: '后端开发',
          collapsed: false,
          items: [
            { text: '编码规范', link: '/tech/backend/coding-style' },
            { text: 'API 设计', link: '/tech/backend/api-design' },
            { text: '数据库规范', link: '/tech/backend/db-rules' },
          ],
        },
        {
          text: '前端开发',
          collapsed: false,
          items: [
            { text: '组件规范', link: '/tech/frontend/components' },
            { text: '样式指南', link: '/tech/frontend/styles' },
          ],
        },
      ],
    },
    
    // 编辑链接
    editLink: {
      pattern: 'https://git.company.com/docs/:path',
      text: '在 Git 中编辑',
    },
    
    // 贡献者
    lastUpdated: {
      enabled: true,
      formatOptions: {
        dateStyle: 'short',
      },
    },
  },
})
```

---

## 文档工具安装与配置完整指南

### VitePress 完整安装流程

```bash
# 方式一：使用 npm create
npm create vitepress@latest my-docs
cd my-docs
npm install

# 方式二：手动安装
mkdir my-docs && cd my-docs
npm init -y
npm install vitepress vue
npm install -D vite

# 目录结构
my-docs/
├── .vitepress/
│   ├── config.ts          # 配置文件
│   ├── theme/             # 主题目录
│   │   ├── index.ts       # 主题入口
│   │   └── custom.css     # 自定义样式
│   └── public/            # 静态资源
│       ├── favicon.svg
│       └── logo.png
├── index.md              # 首页
├── guide/
│   └── getting-started.md
├── api/
│   └── reference.md
└── package.json
```

```typescript
// .vitepress/config.ts
import { defineConfig } from 'vitepress';

export default defineConfig({
  // 基础配置
  title: '我的文档',
  description: '使用 VitePress 构建的精美文档',
  
  // 头部配置
  head: [
    // Favicon
    ['link', { rel: 'icon', href: '/favicon.svg' }],
    // 主题色
    ['meta', { name: 'theme-color', content: '#3c8772' }],
    // Open Graph
    ['meta', { property: 'og:type', content: 'website' }],
    ['meta', { property: 'og:title', content: '我的文档' }],
    ['meta', { property: 'og:description', content: '文档描述' }],
  ],
  
  // Markdown 配置
  markdown: {
    lineNumbers: true,
    theme: {
      light: 'github-light',
      dark: 'github-dark',
    },
    config: (md) => {
      // 添加更多插件
      md.use(/* 插件 */);
    },
  },
  
  // 主题配置
  themeConfig: {
    // Logo
    logo: '/logo.png',
    siteTitle: '我的文档',
    
    // 导航栏
    nav: [
      { text: '指南', link: '/guide/' },
      { text: 'API', link: '/api/' },
      {
        text: '资源',
        items: [
          { text: '博客', link: '/blog/' },
          { text: 'Changelog', link: '/changelog' },
        ],
      },
    ],
    
    // 侧边栏
    sidebar: {
      '/guide/': [
        {
          text: '入门',
          items: [
            { text: '简介', link: '/guide/' },
            { text: '安装', link: '/guide/installation' },
          ],
        },
      ],
    },
    
    // 社交链接
    socialLinks: [
      { icon: 'github', link: 'https://github.com/vuejs/vitepress' },
    ],
    
    // 搜索配置
    search: {
      provider: 'local',
      options: {
        detailedView: true,
      },
    },
    
    // 页脚
    footer: {
      message: '基于 MIT 许可证发布',
      copyright: '版权所有 © 2024',
    },
    
    // 编辑链接
    editLink: {
      pattern: 'https://github.com/org/repo/edit/main/docs/:path',
      text: '在 GitHub 上编辑此页',
    },
    
    // 文档元信息
    lastUpdated: {
      text: '最后更新',
      formatOptions: {
        dateStyle: 'short',
        timeStyle: 'short',
      },
    },
  },
});
```

### Docusaurus 完整安装流程

```bash
# 创建项目
npx create-docusaurus@latest my-website classic

# 目录结构
my-website/
├── blog/                    # 博客文章
│   ├── 2019-05-28-hola.md
│   ├── 2019-05-29-hello-world.md
│   └── 2021-08-01-mdx-blog-post.mdx
├── docs/                    # 文档
│   ├── intro.md
│   ├── tutorial-basics/
│   │   ├── _category_.json
│   │   ├── congratulations.md
│   │   ├── create-a-blog-post.md
│   │   └── create-a-document.md
│   └── tutorial-extras/
├── src/                     # 源代码
│   ├── css/
│   │   └── custom.css
│   ├── pages/
│   │   ├── index.tsx
│   │   └── typescript.ts
│   └── components/
├── static/                  # 静态资源
│   └── img/
├── docs.json               # 侧边栏配置
├── sidebars.js            # 侧边栏配置
├── babel.config.js
├── package.json
└── docusaurus.config.js
```

```javascript
// docusaurus.config.js
export default {
  // 项目元信息
  title: 'My Site',
  tagline: 'Dinosaurs are cool',
  url: 'https://your-docusaurus-site.example.com',
  baseUrl: '/',
  
  // GitHub pages 部署
  organizationName: 'facebook',
  projectName: 'docusaurus',
  
  // i18n
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'zh-CN'],
    localeConfigs: {
      en: {
        label: 'English',
        direction: 'ltr',
      },
      'zh-CN': {
        label: '简体中文',
        direction: 'ltr',
      },
    },
  },
  
  // 预设和插件
  presets: [
    [
      'classic',
      {
        docs: {
          sidebarPath: './sidebars.js',
          editUrl: 'https://github.com/facebook/docusaurus/edit/main/',
          showLastUpdateAuthor: true,
          showLastUpdateTime: true,
        },
        blog: {
          showReadingTime: true,
          feedOptions: {
            type: ['rss', 'atom'],
            xslt: true,
          },
          editUrl: 'https://github.com/facebook/docusaurus/edit/main/',
          blogTitle: 'My Blog',
          blogDescription: 'A Docusaurus-powered blog',
          postsPerPage: 10,
          blogSidebarTitle: 'Recent posts',
          blogSidebarCount: 'ALL',
        },
        theme: {
          customCss: './src/css/custom.css',
        },
      },
    ],
  ],
  
  // 主题配置
  themeConfig: {
    // 图片
    image: 'img/social-card.jpg',
    
    // 导航栏
    navbar: {
      title: 'My Site',
      logo: {
        alt: 'My Site Logo',
        src: 'img/logo.svg',
      },
      items: [
        {
          type: 'docSidebar',
          sidebarId: 'tutorialSidebar',
          position: 'left',
          label: 'Tutorial',
        },
        { to: '/blog', label: 'Blog', position: 'left' },
        {
          type: 'docsVersionDropdown',
          position: 'right',
        },
        {
          type: 'localeDropdown',
          position: 'right',
        },
        {
          href: 'https://github.com/facebook/docusaurus',
          position: 'right',
          className: 'header-github-link',
          'aria-label': 'GitHub repository',
        },
      ],
    },
    
    // 页脚
    footer: {
      style: 'dark',
      links: [
        {
          title: 'Docs',
          items: [
            { label: 'Tutorial', to: '/docs/intro' },
            { label: 'API Reference', to: '/docs/api' },
          ],
        },
        {
          title: 'Community',
          items: [
            {
              label: 'Stack Overflow',
              href: 'https://stackoverflow.com/questions/tagged/docusaurus',
            },
            {
              label: 'Discord',
              href: 'https://discordapp.com/invite/docusaurus',
            },
          ],
        },
      ],
      copyright: `Copyright © ${new Date().getFullYear()} My Project, Inc. Built with Docusaurus.`,
    },
    
    // Prism 代码高亮
    prism: {
      theme: lightCodeTheme,
      darkTheme: darkCodeTheme,
      additionalLanguages: ['bash', 'diff', 'json', 'yaml', 'java', 'ruby'],
    },
    
    // Algolia 搜索
    algolia: {
      appId: 'YOUR_APP_ID',
      apiKey: 'YOUR_SEARCH_API_KEY',
      indexName: 'YOUR_INDEX_NAME',
      contextualSearch: true,
      searchParameters: {},
      searchPagePath: 'search',
    },
  },
};
```

---

## VitePress 常用命令与操作

### 开发命令

```bash
# 开发服务器
npm run docs:dev        # 启动开发服务器
npm run docs:dev -- --port 3000  # 指定端口

# 生产构建
npm run docs:build      # 构建静态文件
npm run docs:preview    # 预览生产构建

# 常用 npm scripts
"scripts": {
  "docs:dev": "vitepress dev docs",
  "docs:build": "vitepress build docs",
  "docs:preview": "vitepress preview docs",
  "docs:serve": "vitepress serve docs"
}
```

### GitHub Actions 部署

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build
        run: npm run docs:build
        env:
          NODE_OPTIONS: --max_old_space_size=4096
          
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: docs/.vitepress/dist
          
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Netlify 部署

```bash
# netlify.toml
[build]
  command = "npm run docs:build"
  publish = "docs/.vitepress/dist"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# 或者使用 CLI
npm install -g netlify-cli
netlify deploy --prod --dir=docs/.vitepress/dist
```

### VPS 部署 (Nginx)

```nginx
# /etc/nginx/sites-available/docs
server {
    listen 80;
    server_name docs.example.com;
    
    root /var/www/docs;
    index index.html;
    
    # Gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    
    # 缓存配置
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # SPA 回退
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

---

## VitePress 与同类技术深度对比

### VitePress vs Next.js Docs

```yaml
# 对比矩阵
特性:
  框架:
    VitePress: Vue 3 (可选)
    Next.js: React (必须)
  
  内容格式:
    VitePress: Markdown + Vue 组件
    Next.js: MDX + React 组件
  
  静态生成:
    VitePress: SSG，输出纯静态文件
    Next.js: SSG + SSR + ISR
  
  页面路由:
    VitePress: 基于文件系统的 Markdown 路由
    Next.js: 基于文件系统的 React 路由
  
  部署:
    VitePress: 任何静态托管
    Next.js: 需要 Node.js 环境或导出为静态
  
  学习曲线:
    VitePress: 低，适合非前端开发者
    Next.js: 中，需要 React 知识
  
  适用场景:
    VitePress: 纯文档站点
    Next.js: 文档 + 复杂应用混合
```

### VitePress vs MkDocs (Python)

```yaml
# 对比矩阵
特性:
  语言:
    VitePress: JavaScript/TypeScript
    MkDocs: Python
  
  主题:
    VitePress: Vue 组件系统
    MkDocs: Jinja2 模板
  
  插件:
    VitePress: Vite 插件 + Remark/Rehype
    MkDocs: Python 插件
  
  Markdown 扩展:
    VitePress: JSX 组件
    MkDocs: Python 宏
  
  搜索:
    VitePress: 内置本地搜索 / Algolia
    MkDocs: MkDocs 内置搜索 / Algolia
  
  性能:
    VitePress: 极快 (Vite)
    MkDocs: 较快 (Python)
  
  适用场景:
    VitePress: 前端/全栈团队
    MkDocs: Python 团队 / 数据文档
```

### VitePress vs Docsify (运行时渲染)

```yaml
# 对比矩阵
特性:
  渲染方式:
    VitePress: 编译时 SSG
    Docsify: 运行时浏览器渲染
  
  性能:
    VitePress: 首屏更快，SEO 友好
    Docsify: 首屏需要加载 JS
  
  内容更新:
    VitePress: 需要重新构建
    Docsify: 直接更新 Markdown
  
  离线支持:
    VitePress: 需要 Service Worker
    Docsify: 原生支持
  
  部署:
    VitePress: 需要构建步骤
    Docsify: 直接部署 Markdown
  
  适用场景:
    VitePress: 生产文档 / SEO 重要
    Docsify: 快速原型 / 小团队协作
```

---

## VitePress 高级配置与技巧

### 插件开发

```typescript
// .vitepress/plugins/my-plugin.ts
import type { Plugin } from 'vite';

export function myPlugin(): Plugin {
  return {
    name: 'vitepress-my-plugin',
    
    // 虚拟模块
    resolveId(id) {
      if (id === 'virtual:my-data') {
        return id;
      }
    },
    
    load(id) {
      if (id === 'virtual:my-data') {
        return `export const myData = ${JSON.stringify({ foo: 'bar' })};`;
      }
    },
    
    // Markdown 转换
    enforce: 'pre',
    transform(code, id) {
      if (!id.endsWith('.md')) return;
      // 转换 Markdown
      return code;
    },
  };
}

// 使用插件
// .vitepress/config.ts
import { defineConfig } from 'vitepress';
import { myPlugin } from './plugins/my-plugin';

export default defineConfig({
  vite: {
    plugins: [myPlugin()],
  },
});
```

### 自定义 Markdown 指令

```typescript
// .vitepress/plugins/directives.ts
import type { Plugin } from 'vite';
import { visit } from 'unist-util-visit';

export function directivesPlugin(): Plugin {
  return {
    name: 'vitepress-directives',
    enforce: 'pre',
    transform(code, id) {
      if (!id.endsWith('.md')) return;
      
      // 处理 :::tip 语法
      code = code.replace(
        /:::(\w+)\s*\n([\s\S]*?):::/g,
        (match, type, content) => {
          return `<${type.toLowerCase()}>${content.trim()}</${type.toLowerCase()}>`;
        }
      );
      
      // 处理 :: component :: 语法
      code = code.replace(
        /:::\s*(\w+)\s*\n([\s\S]*?)\n:::/g,
        (match, name, props) => {
          return `<${name} ${props.trim()} />`;
        }
      );
      
      return code;
    },
  };
}
```

### SEO 优化

```typescript
// .vitepress/config.ts
export default defineConfig({
  head: [
    // 基础 SEO
    ['meta', { name: 'description', content: '站点描述' }],
    ['meta', { name: 'keywords', content: '关键词1, 关键词2' }],
    ['meta', { name: 'author', content: '作者名' }],
    
    // Open Graph
    ['meta', { property: 'og:type', content: 'website' }],
    ['meta', { property: 'og:title', content: '站点标题' }],
    ['meta', { property: 'og:description', content: '站点描述' }],
    ['meta', { property: 'og:image', content: '/og-image.jpg' }],
    ['meta', { property: 'og:url', content: 'https://example.com' }],
    
    // Twitter Card
    ['meta', { name: 'twitter:card', content: 'summary_large_image' }],
    ['meta', { name: 'twitter:title', content: '站点标题' }],
    ['meta', { name: 'twitter:description', content: '站点描述' }],
    ['meta', { name: 'twitter:image', content: '/og-image.jpg' }],
    
    // 规范链接
    ['link', { rel: 'canonical', href: 'https://example.com' }],
    
    // RSS
    ['link', { rel: 'alternate', type: 'application/rss+xml', title: 'RSS Feed', href: '/feed.xml' }],
  ],
});
```

---

## VitePress 实战项目示例

### 示例 1：API 参考文档

```typescript
// .vitepress/config.ts
export default defineConfig({
  title: 'API Reference',
  
  themeConfig: {
    logo: '/api-logo.svg',
    
    nav: [
      { text: '指南', link: '/guide/' },
      { text: 'API', link: '/api/' },
      { text: 'SDKs', link: '/sdks/' },
      { text: 'Changelog', link: '/changelog' },
    ],
    
    sidebar: {
      '/api/': [
        {
          text: '认证',
          items: [
            { text: 'API Keys', link: '/api/auth/api-keys' },
            { text: 'OAuth 2.0', link: '/api/auth/oauth' },
            { text: 'JWT Tokens', link: '/api/auth/jwt' },
          ],
        },
        {
          text: '用户',
          items: [
            { text: '获取用户', link: '/api/users/get' },
            { text: '创建用户', link: '/api/users/create' },
            { text: '更新用户', link: '/api/users/update' },
            { text: '删除用户', link: '/api/users/delete' },
          ],
        },
      ],
    },
  },
});
```

```markdown
---
title: 获取用户
description: 获取指定用户的详细信息
---

# 获取用户

获取指定用户的详细信息。

## 请求

```
GET /api/users/{id}
```

### 路径参数

| 参数 | 类型 | 描述 |
|------|------|------|
| `id` | string | 用户 ID |

### 请求头

| 头 | 必填 | 描述 |
|-----|------|------|
| `Authorization` | 是 | Bearer Token |

## 响应

```json
{
  "id": "usr_123456",
  "name": "张三",
  "email": "zhangsan@example.com",
  "role": "admin",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

## SDK 示例

```javascript
import { Client } from '@example/sdk';

const client = new Client({ apiKey: 'YOUR_API_KEY' });
const user = await client.users.get('usr_123456');
console.log(user.name);
```

:::tip
确保妥善保管您的 API Key，不要在客户端代码中暴露。
:::
```

### 示例 2：开源项目文档

```typescript
// .vitepress/config.ts
export default defineConfig({
  title: 'Awesome CLI',
  description: '一个高效的命令行工具',
  
  themeConfig: {
    hero: {
      name: 'Awesome CLI',
      text: '让命令行更高效',
      tagline: '快速、简单、可扩展的命令行工具',
      actions: [
        { theme: 'brand', text: '快速开始', link: '/guide/' },
        { theme: 'alt', text: 'GitHub', link: 'https://github.com/example/awesome-cli' },
      ],
    },
    
    features: [
      {
        icon: '🚀',
        title: '极速启动',
        details: '基于 Rust 构建，启动时间 < 50ms',
      },
      {
        icon: '🔌',
        title: '插件系统',
        details: '强大的插件 API，轻松扩展功能',
      },
      {
        icon: '🎨',
        title: '主题支持',
        details: '内置多种主题，支持自定义配色',
      },
      {
        icon: '📦',
        title: '零依赖',
        details: '单文件部署，无需额外运行环境',
      },
    ],
    
    socialLinks: [
      { icon: 'github', link: 'https://github.com/example/awesome-cli' },
      { icon: 'twitter', link: 'https://twitter.com/example' },
      { icon: 'discord', link: 'https://discord.gg/example' },
    ],
  },
});
```

---

## 文档工具常见问题与解决方案

### 问题 1：VitePress 构建失败

```bash
# 问题：构建时出现内存溢出
# 解决方案 1：增加 Node.js 内存
NODE_OPTIONS=--max_old_space_size=4096 npm run docs:build

# 解决方案 2：优化图片
# 使用 WebP 格式
```

### 问题 2：Markdown 中的 Vue 组件不渲染

```markdown
---
title: 使用组件
---

# 我的页面

<script setup>
import MyComponent from './MyComponent.vue'
</script>

<MyComponent />
```

### 问题 3：代码块高亮不工作

```typescript
// 确保语言正确标注
// ```typescript  (不是 ```ts)
// ```javascript (不是 ```js)
```

### 问题 4：搜索不返回结果

```typescript
// 确保 Markdown 文件在正确的位置
// 默认在项目根目录
srcDir: 'docs',
```

---

## 相关资源

- [[VitePress 官方文档]]
- [[Docusaurus 官方文档]]
- [[Slidev 官方文档]]
- [[GitBook 官方文档]]
- [[VuePress vs VitePress]]
- [[静态站点生成器对比]]
- [[MDX 深度指南]]
- [[OpenAPI 文档工具]]

---

*本文档由 [[归愚知识系统]] 自动生成*
