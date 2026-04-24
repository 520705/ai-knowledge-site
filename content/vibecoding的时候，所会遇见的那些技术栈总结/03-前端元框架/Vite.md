# Vite 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Vite 6 核心特性、Dev Server 原理、Plugin 系统、Module Federation 及在 AI 应用中的战略定位。

---

## 目录

1. [[#Vite 概述与核心优势]]
2. [[#Vite 6 新特性]]
3. [[#Dev Server 原理详解]]
4. [[#Vite vs Webpack vs Parcel vs esbuild 对比]]
5. [[#Plugin 系统]]
6. [[#常用 Vite Plugin 列表]]
7. [[#Module Federation]]
8. [[#兼容 Vite 的框架]]
9. [[#Vite 配置详解]]
10. [[#AI 应用实战]]
11. [[#选型建议]]

---

## Vite 概述与核心优势

### 什么是 Vite

Vite 是由 Vue 作者尤雨溪主导开发的**下一代前端构建工具**，其核心理念是利用现代浏览器的原生 ES Module（ESM）能力，实现极致的开发体验。Vite 在开发阶段跳过打包过程，直接以原生 ESM 方式提供服务；仅在生产构建时使用 Rollup 进行高效打包。

**Vite 核心特点：**

| 特性 | 说明 |
|------|------|
| **极速冷启动** | 无需打包，直接 serve 原生 ESM |
| **极速 HMR** | 模块级热更新，毫秒级响应 |
| **按需编译** | 只编译当前访问的模块 |
| **生产优化** | 使用 Rollup 进行高效 tree-shaking |
| **原生 TypeScript** | 原生支持，无需额外转译 |
| **多框架支持** | Vue/React/Svelte/Solid 等 |

### Vite 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| Vite 1.0 | 2020 | 发布基础版，ESM Dev Server |
| Vite 2.0 | 2021 | 插件系统重写，CSS 支持增强 |
| Vite 3.0 | 2022 | 改进冷启动，更好的依赖优化 |
| Vite 4.0 | 2022 | Rollup 3 升级，兼容性改进 |
| Vite 5.0 | 2023 | Rollup 4，性能优化 |
| **Vite 6.0** | **2024** | **环境 API、Module Federation、稳定化** |

### 核心理念

**Vite 的设计哲学：**

1. **开发时无打包**：利用浏览器原生 ESM，避免开发时的完整打包
2. **按需编译**：只编译实际使用的代码，减少等待时间
3. **依赖预构建**：使用 esbuild 预构建依赖，提升加载速度
4. **HMR 精确化**：只更新变化的模块，而非重新渲染整个应用

> [!IMPORTANT]
> Vite 的"无打包"理念是革命性的。在 Webpack 时代，开发启动需要等待完整打包；Vite 将这个时间从分钟级降低到秒级。

---

## Vite 6 新特性

### 环境 API（Environment API）

Vite 6 引入了全新的 Environment API，提供了更灵活的多环境配置能力：

```typescript
// vite.config.ts
import { defineConfig, loadEnv } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig(({ command, mode }) => {
  // 加载环境变量
  const env = loadEnv(mode, process.cwd(), '');

  return {
    plugins: [react()],
    environments: {
      // 客户端环境
      client: {
        resolve: {
          alias: {
            '@': '/src'
          }
        }
      },
      // SSR 环境
      ssr: {
        resolve: {
          alias: {
            '@': '/src'
          }
        }
      },
      // 边缘环境
      edge: {
        resolve: {
          alias: {
            '@': '/src'
          }
        }
      }
    }
  };
});
```

**使用环境变量：**

```typescript
// 在应用中使用
import { environments } from 'vite';

const isServer = environments.client;
const isEdge = environments.edge;
```

### Module Federation 稳定化

Vite 6 将 Module Federation 从实验状态提升为稳定功能：

```typescript
// host/vite.config.ts
import { defineConfig } from 'vite';
import { VitePluginFederation } from 'vite-plugin-federation';

export default defineConfig({
  plugins: [
    VitePluginFederation({
      name: 'host',
      shared: ['react', 'react-dom']
    })
  ]
});
```

### 更快的依赖预构建

Vite 6 改进了依赖预构建算法：

| 优化项 | Vite 5 | Vite 6 |
|--------|--------|--------|
| **首次预构建** | ~2-3 秒 | ~1 秒 |
| **增量更新** | 需完全重建 | 仅更新变化文件 |
| **缓存策略** | 简单哈希 | 智能依赖图 |

### 环境变量类型增强

```typescript
// env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_WS_URL: string;
  readonly VITE_PUBLIC_KEY: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### CSS 改进

**CSS @layer 支持：**

```css
@layer base {
  * {
    margin: 0;
    padding: 0;
  }
}

@layer components {
  .button {
    padding: 8px 16px;
  }
}

@layer utilities {
  .hidden {
    display: none;
  }
}
```

---

## Dev Server 原理详解

### 传统构建工具 vs Vite

**传统构建工具（Webpack）：**

```
开发流程：
1. 扫描所有文件依赖关系
2. 构建完整依赖图
3. 将所有模块打包为 bundle
4. 启动 Dev Server
5. 浏览器加载 bundle

问题：文件越多，启动越慢
典型时间：30秒-2分钟
```

**Vite：**

```
开发流程：
1. 启动原生 ESM Dev Server
2. 浏览器请求模块 → 按需编译
3. 首次访问时编译当前路由
4. HMR 时只更新变化的模块

优势：启动即服务，文件多少不影响
典型时间：<1 秒
```

### Dev Server 工作流程

```
┌─────────────────────────────────────────────────────────┐
│                    Browser (ESM)                        │
│                                                         │
│  <script type="module" src="/src/main.tsx"></script>    │
└─────────────────────┬───────────────────────────────────┘
                      │
                      │ HTTP Request
                      ▼
┌─────────────────────────────────────────────────────────┐
│                  Vite Dev Server                        │
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │           HTTP Request Transform                 │   │
│  │  - 解析请求 URL                                  │   │
│  │  - 处理特殊协议 (@id, /@fs, /@modules)          │   │
│  └──────────────────────┬──────────────────────────┘   │
│                         │                               │
│                         ▼                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Module Resolution                   │   │
│  │  - 路径别名解析 (@/)                            │   │
│  │  - npm 包定位                                   │   │
│  │  - 特殊文件处理 (.tsx, .jsx, .vue)             │   │
│  └──────────────────────┬──────────────────────────┘   │
│                         │                               │
│                         ▼                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │           Transform (esbuild/swc)              │   │
│  │  - TypeScript → JavaScript                      │   │
│  │  - JSX → 函数调用                               │   │
│  │  - CSS Modules → 标准 CSS                       │   │
│  │  - 内联资源处理                                 │   │
│  └──────────────────────┬──────────────────────────┘   │
│                         │                               │
│                         ▼                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │             Cache & Hot Module                   │   │
│  │  - 结果缓存                                     │   │
│  │  - HMR WebSocket 通知                           │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                      │
                      │ Transformed Response
                      ▼
┌─────────────────────────────────────────────────────────┐
│                   Browser (ESM)                         │
│                                                         │
│  接收模块 → 解析依赖 → 递归请求依赖 → 执行模块            │
└─────────────────────────────────────────────────────────┘
```

### 依赖预构建（Dependency Pre-bundling）

Vite 使用 esbuild 对依赖进行预构建，解决以下问题：

1. **CJS/ESM 混合问题**：将 CommonJS 模块转换为 ESM
2. **网络请求优化**：将多个 import 合并，减少请求数
3. **处理内部 import**：解析复杂的依赖关系

```javascript
// 预构建配置（可选）
export default defineConfig({
  optimizeDeps: {
    include: ['react', 'react-dom', 'lodash'],
    exclude: ['your-local-package']
  }
});
```

### HMR 原理

```
┌─────────────────────────────────────────────────────────┐
│                     Vite Dev Server                     │
│                                                         │
│  ┌────────────────┐     ┌────────────────────────┐    │
│  │  File Watcher   │────▶│   HMR Engine           │    │
│  │  (chokidar)     │     │                       │    │
│  └────────────────┘     │  1. 接收文件变化通知   │    │
│                         │  2. 确定影响范围        │    │
│                         │  3. 发送更新信号        │    │
│                         └───────────┬────────────┘    │
│                                     │                 │
│                                     ▼                 │
│                         ┌────────────────────────────┐ │
│                         │   WebSocket / HMR Client    │ │
│                         └────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
                      │
                      │ WebSocket Connection
                      ▼
┌─────────────────────────────────────────────────────────┐
│                     Browser                             │
│                                                         │
│  1. 接收 HMR 信号                                       │
│  2. 加载更新的模块（如果有）                             │
│  3. 调用 HMR 接受函数（通常是框架处理）                   │
│  4. 更新组件状态，重新渲染                              │
└─────────────────────────────────────────────────────────┘
```

---

## Vite vs Webpack vs Parcel vs esbuild 对比

### 构建工具对比表

| 特性 | Vite 6 | Webpack 5 | Parcel 2 | esbuild |
|------|--------|-----------|----------|---------|
| **冷启动时间** | <1s | 30s-2min | 2-5s | <100ms |
| **HMR 速度** | 毫秒级 | 秒级 | 秒级 | 无 HMR |
| **包体积** | ~500KB | ~2MB | ~1.5MB | ~5MB |
| **配置复杂度** | 低 | 高 | 零配置 | 极简 |
| **插件生态** | 丰富 | 最丰富 | 丰富 | 较少 |
| **tree-shaking** | ✅ 完整 | ✅ 完整 | ✅ 完整 | ✅ 完整 |
| **Code Splitting** | ✅ | ✅ | ✅ | ❌ |
| **CSS Modules** | ✅ | ✅ | ✅ | ✅ |
| **TypeScript** | ✅ 原生 | ✅ 需配置 | ✅ 原生 | ✅ |
| **多框架支持** | ✅ | ✅ | ✅ | ✅ |
| **生产构建速度** | 快 | 慢 | 中等 | 最快 |
| **生产包大小** | 优化良好 | 需优化 | 优化良好 | 需处理 |

### 性能对比

| 测试场景 | Vite | Webpack | esbuild | 说明 |
|---------|------|---------|---------|------|
| **冷启动（1000 模块）** | 0.8s | 45s | 0.05s | Vite 基于 ESM |
| **HMR（单文件）** | 50ms | 500ms | N/A | Vite 按需更新 |
| **生产构建** | 10s | 60s | 2s | Rollup vs Go |
| **TypeScript 编译** | 3s | 15s | 0.5s | esbuild 更快 |
| **内存占用** | 中等 | 高 | 低 | 架构差异 |

### 适用场景对比

| 场景 | 首选 | 备选 |
|------|------|------|
| **Vue/React 开发** | Vite | Webpack |
| **Svelte 开发** | Vite | Rollup |
| **巨型应用** | Webpack | Vite |
| **简单静态页面** | Parcel | Vite |
| **库开发** | Rollup | esbuild |
| **Node.js SSR** | Vite | Webpack |
| **边缘函数** | esbuild | Vite |

---

## Plugin 系统

### Vite 插件结构

```typescript
import type { Plugin } from 'vite';

export default function myPlugin(): Plugin {
  return {
    name: 'vite-plugin-example',

    // 钩子执行顺序
    enforce: 'pre', // 或 'post'

    // 钩子函数
    buildStart() {
      // 构建开始时调用
    },

    resolveId(source, importer) {
      // 解析模块 ID 时调用
      // 返回 null 表示使用默认解析
      // 返回 { id: resolvedId } 表示重定向
    },

    load(id) {
      // 加载模块时调用
      // 返回 null 表示使用默认加载
      // 返回代码字符串表示自定义内容
    },

    transform(code, id) {
      // 转换模块内容时调用
      // 返回 { code, map } 或 null
    },

    configureServer(server) {
      // 配置 Dev Server
      // 添加中间件、自定义路由等
    },

    buildEnd() {
      // 构建结束时调用
    }
  };
}
```

### 完整插件示例

```typescript
import type { Plugin, ViteDevServer } from 'vite';
import { createFilter } from 'vite';

interface Options {
  include?: string[];
  exclude?: string[];
  verbose?: boolean;
}

export default function htmlPlugin(options: Options = {}): Plugin {
  const filter = createFilter(
    options.include,
    options.exclude
  );

  return {
    name: 'vite-plugin-custom-transform',

    enforce: 'pre',

    transform(code, id) {
      if (!filter(id)) return null;

      if (options.verbose) {
        console.log(`[vite-plugin-custom] Transforming ${id}`);
      }

      // 示例：替换特定字符串
      const transformed = code.replace(
        /__VERSION__/g,
        process.env.npm_package_version || '1.0.0'
      );

      return {
        code: transformed,
        map: null
      };
    },

    configureServer(server: ViteDevServer) {
      // 开发服务器配置
      server.middlewares.use('/api', (req, res) => {
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify({ message: 'Dev API Response' }));
      });
    }
  };
}
```

### 常用插件钩子

| 钩子 | 阶段 | 说明 |
|------|------|------|
| `buildStart` | 构建开始 | 初始化，构建选项验证 |
| `configResolved` | 配置解析后 | 读取最终配置 |
| `configureServer` | 配置 Dev Server | 添加中间件等 |
| `transformIndexHtml` | 转换 HTML | 自定义 index.html |
| `resolveId` | 解析模块 ID | 处理别名、虚拟模块 |
| `load` | 加载模块 | 自定义模块加载 |
| `transform` | 转换代码 | Babel/ts/SWC 转换 |
| `shouldTransformCachedModule` | 缓存判断 | 控制缓存行为 |
| `handleHotUpdate` | HMR 更新 | 自定义 HMR 处理 |
| `buildEnd` | 构建结束 | 最终清理 |

---

## 常用 Vite Plugin 列表

### 框架集成

| 插件 | 功能 | GitHub Stars |
|------|------|-------------|
| `@vitejs/plugin-react` | React 支持（SWC） | 官方 |
| `@vitejs/plugin-vue` | Vue 3 支持 | 官方 |
| `@vitejs/plugin-vue-jsx` | Vue JSX 支持 | 官方 |
| `@sveltejs/vite-plugin-svelte` | Svelte 支持 | 官方 |
| `vite-plugin-solid` | Solid.js 支持 | ~1K |

### 构建优化

| 插件 | 功能 | GitHub Stars |
|------|------|-------------|
| `vite-plugin-compression` | Gzip/Brotli 压缩 | ~1K |
| `vite-plugin-visualizer` | Bundle 可视化 | ~3K |
| `vite-plugin-checker` | TypeScript 检查 | ~2K |
| `vite-plugin-inspect` | 中间件检查 | ~1K |

### 开发工具

| 插件 | 功能 | GitHub Stars |
|------|------|-------------|
| `vite-plugin-mock` | Mock 数据 | ~3K |
| `vite-plugin-pwa` | PWA 支持 | ~6K |
| `vite-plugin-md` | Markdown 支持 | ~1K |
| `vite-plugin-pages` | 文件路由生成 | ~3K |

### 特殊功能

| 插件 | 功能 | GitHub Stars |
|------|------|-------------|
| `vite-plugin-federation` | Module Federation | ~3K |
| `vite-plugin-svg-icons` | SVG 图标 | ~1K |
| `vite-plugin-importer` | 自动导入 | ~500 |
| `unplugin-vue-components` | 组件自动导入 | ~4K |

### AI 相关插件

| 插件 | 功能 |
|------|------|
| `vite-plugin-markdown` | 支持 Markdown 文件 |
| `vite-plugin-ai-components` | AI 组件集成 |
| `vite-plugin-stream` | 流式响应支持 |

---

## Module Federation

### 什么是 Module Federation

Module Federation（模块联邦）允许独立构建的应用共享模块，实现微前端的模块复用。

### 主机应用配置

```typescript
// host/vite.config.ts
import { defineConfig } from 'vite';
import { VitePluginFederation } from 'vite-plugin-federation';

export default defineConfig({
  plugins: [
    VitePluginFederation({
      // 主机名称
      name: 'host',
      // 共享的依赖
      shared: ['react', 'react-dom', 'zustand']
    })
  ]
});
```

### 远程应用配置

```typescript
// remote/vite.config.ts
import { defineConfig } from 'vite';
import { VitePluginFederation } from 'vite-plugin-federation';

export default defineConfig({
  plugins: [
    VitePluginFederation({
      name: 'remote',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button.tsx',
        './Header': './src/components/Header.tsx',
        './store': './src/store/index.ts'
      },
      shared: ['react', 'react-dom', 'zustand']
    })
  ]
});
```

### 消费远程模块

```typescript
// 在主机应用中使用
import { lazy } from 'react';

const RemoteButton = lazy(() => import('remote/Button'));
const RemoteHeader = lazy(() => import('remote/Header'));
const remoteStore = await import('remote/store');

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <RemoteHeader />
        <RemoteButton onClick={() => remoteStore.increment()}>
          Click me
        </RemoteButton>
      </Suspense>
    </div>
  );
}
```

---

## 兼容 Vite 的框架

### 框架列表

| 框架 | 配置方式 | 说明 |
|------|---------|------|
| **Vue** | 官方插件 | Vue 3 + Vite 官方推荐 |
| **Nuxt** | Nuxt 3 内置 | Vue 3 全栈框架 |
| **Svelte** | 官方插件 | Svelte + Vite 官方集成 |
| **SvelteKit** | 官方支持 | Svelte 全栈框架 |
| **React** | @vitejs/plugin-react | React + Vite 官方插件 |
| **Next.js** | 实验性支持 | Next.js 14 实验性 Vite 支持 |
| **Solid** | 社区插件 | SolidStart 使用 Vite |
| **Astro** | 内置支持 | Astro 原生 Vite |
| **RedwoodJS** | 内置支持 | 全栈 React 框架 |
| **Vitesse** | 模板 | Vue 3 + Vite 启动模板 |
| **Tauri** | 集成 | 桌面应用开发 |

### Nuxt 3 配置

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  // Vite 配置直接写在 vite 字段
  vite: {
    optimizeDeps: {
      include: ['vue', 'vue-router']
    },
    server: {
      port: 3000
    }
  },

  modules: [
    '@pinia/nuxt',
    '@nuxtjs/tailwindcss'
  ]
});
```

### Astro 配置

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [react(), tailwind()],
  vite: {
    optimizeDeps: {
      exclude: ['@astrojs/react']
    }
  }
});
```

---

## Vite 配置详解

### 完整配置示例

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

export default defineConfig({
  // 基础路径
  base: '/',

  // 环境变量
  envDir: './env',
  envPrefix: 'VITE_',

  // 服务器配置
  server: {
    port: 3000,
    host: true,
    https: false,
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    },
    cors: true
  },

  // 预览配置
  preview: {
    port: 4173,
    proxy: {
      '/api': 'http://localhost:8080'
    }
  },

  // 构建配置
  build: {
    target: 'esnext',
    outDir: 'dist',
    assetsDir: 'assets',
    assetsInlineLimit: 4096,
    cssCodeSplit: true,
    cssMinify: true,
    sourcemap: false,
    minify: 'esbuild',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor': ['react', 'react-dom'],
          'utils': ['lodash', 'dayjs']
        },
        chunkFileNames: 'assets/[name]-[hash].js',
        entryFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]'
      }
    }
  },

  // 依赖优化
  optimizeDeps: {
    include: ['react', 'react-dom'],
    exclude: ['@internal/pkg']
  },

  // 路径别名
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils')
    },
    extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json']
  },

  // 插件
  plugins: [react()]
});
```

### TypeScript 配置

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

---

## AI 应用实战

### Vite + React + AI SDK

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true
      }
    }
  }
});
```

### 流式响应配置

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      '/api/stream': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        configure: (proxy) => {
          // 确保流式响应正常工作
          proxy.on('proxyRes', (proxyRes) => {
            proxyRes.headers['x-accel-buffering'] = 'no';
          });
        }
      }
    }
  }
});
```

### AI 应用启动模板

```bash
# 创建项目
npm create vite@latest my-ai-app -- --template react-ts

# 安装依赖
cd my-ai-app
npm install
npm install ai @ai-sdk/openai zustand @tanstack/react-query

# 启动开发服务器
npm run dev
```

---

## 选型建议

### 何时选择 Vite

| 场景 | 推荐理由 |
|------|---------|
| **新项目启动** | 开发体验最佳，启动速度快 |
| **Vue 3 项目** | Vue 官方推荐构建工具 |
| **追求 HMR 体验** | 毫秒级热更新 |
| **库开发** | 与 Rollup 配合良好 |
| **TypeScript 项目** | 原生支持，无需额外配置 |
| **微前端架构** | Module Federation 支持 |

### 何时考虑 Webpack

| 场景 | 推荐原因 |
|------|---------|
| **巨型企业应用** | 成熟的代码分割和缓存策略 |
| **复杂工作流** | 需要精细的构建控制 |
| **遗留项目** | 已有 Webpack 配置 |
| **特定插件依赖** | 依赖 Webpack 特有插件 |

> [!TIP]
> Vite 在 2026 年已成为新项目的首选构建工具。其开发体验优势明显，生产构建质量与 Webpack 持平。对于 AI 应用，Vite 的快速迭代能力尤为重要。

---

> [!SUCCESS]
> Vite 以其革命性的开发体验重新定义了前端构建工具的标准。极速冷启动、毫秒级 HMR、原生 TypeScript 支持，使其成为 2026 年前端开发的首选构建工具。配合 @vitejs/plugin-react、@vitejs/plugin-vue 等官方插件，Vite 可满足绝大多数项目的构建需求。

---

## 完整安装指南

### 环境要求与前置准备

**Node.js 版本要求：**
- Vite 6: Node.js 18.12 或更高版本
- 推荐使用 Node.js 20 LTS 或 22 LTS

```bash
# 使用 fnm 安装 Node.js 20
curl -fsSL https://fnm.vercel.app/install | bash
fnm install 20
fnm use 20

# 验证安装
node --version  # 应显示 v20.x.x
npm --version   # 应显示 10.x.x
```

### 项目创建详细流程

**方式一：使用 create-vite（推荐）**

```bash
# 交互式创建
npm create vite@latest

# 非交互式创建
npm create vite@latest my-app -- --template vanilla
npm create vite@latest my-app -- --template vanilla-ts
npm create vite@latest my-app -- --template vue
npm create vite@latest my-app -- --template vue-ts
npm create vite@latest my-app -- --template react
npm create vite@latest my-app -- --template react-ts
npm create vite@latest my-app -- --template react-swc
npm create vite@latest my-app -- --template react-swc-ts
npm create vite@latest my-app -- --template preact
npm create vite@latest my-app -- --template preact-ts
npm create vite@latest my-app -- --template lit
npm create vite@latest my-app -- --template lit-ts
npm create vite@latest my-app -- --template svelte
npm create vite@latest my-app -- --template svelte-ts
npm create vite@latest my-app -- --template qwik
npm create vite@latest my-app -- --template qwik-ts
```

**create-vite 模板说明：**

| 模板 | 说明 | 适用场景 |
|------|------|---------|
| `vanilla` | 纯 JavaScript | 简单页面、原型开发 |
| `vanilla-ts` | 纯 TypeScript | 简单页面 + 类型安全 |
| `vue` | Vue 3 | Vue 项目 |
| `vue-ts` | Vue 3 + TypeScript | Vue 项目（推荐） |
| `react` | React | React 项目 |
| `react-ts` | React + TypeScript | React 项目（推荐） |
| `react-swc` | React + SWC | 更快编译 |
| `preact` | Preact | 轻量 React 替代 |
| `lit` | Lit Web Components | Web Components |
| `svelte` | Svelte | Svelte 项目 |
| `qwik` | Qwik | Resumable 框架 |

**方式二：手动创建 Vite 项目**

```bash
# 创建项目目录
mkdir my-app && cd my-app

# 初始化 npm 项目
npm init -y

# 安装 Vite 核心依赖
npm install vite

# 安装 React 插件（根据需要）
npm install -D @vitejs/plugin-react

# 安装 React
npm install react react-dom

# 创建基础文件
mkdir -p src public
touch index.html src/main.js vite.config.js
```

**package.json 配置：**

```json
{
  "name": "my-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@vitejs/plugin-react": "^4.3.0",
    "vite": "^6.0.0"
  }
}
```

### Vite 配置详解

**vite.config.ts 完整配置：**

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import vue from '@vitejs/plugin-vue';
import svelte from '@sveltejs/vite-plugin-svelte';
import { resolve } from 'path';

export default defineConfig({
  // === 基础配置 ===
  base: '/',  // 部署基础路径

  // === 模式配置 ===
  // 可通过 --mode production 开发环境覆盖
  // 会自动加载 .env.[mode] 文件
  mode: 'development',

  // === 环境变量 ===
  envDir: './env',  // 环境变量文件目录
  envPrefix: ['VITE_', 'MY_'],  // 环境变量前缀

  // === CSS 配置 ===
  css: {
    // CSS 预处理器
    preprocessorOptions: {
      css: {
        charset: false,
      },
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
      less: {
        math: 'always',
      },
    },
    // CSS Modules
    modules: {
      generateScopedName: '[name]__[local]___[hash:base64:5]',
      localsConvention: 'camelCase',
    },
    // DevTools sourcemap
    devSourcemap: true,
  },

  // === JSON ===
  json: {
    stringify: true,  // 使用 JSON.stringify
  },

  // === 构建选项 ===
  build: {
    target: 'esnext',  // 浏览器目标
    modulePreload: {
      polyfill: false,  // 不包含 polyfill
    },
    outDir: 'dist',  // 输出目录
    assetsDir: 'assets',  // 资源目录
    assetsInlineLimit: 4096,  // 小于此大小转为 base64
    cssCodeSplit: true,  // CSS 代码分割
    cssMinify: true,  // CSS 压缩
    sourcemap: false,  // 不生成 sourcemap
    minify: 'esbuild',  // 压缩器
    chunkSizeWarningLimit: 500,  // 警告阈值 (KB)
    rollupOptions: {
      output: {
        // Manual chunks
        manualChunks: {
          'vendor': ['react', 'react-dom'],
          'utils': ['lodash-es', 'dayjs'],
        },
        // 文件名格式化
        entryFileNames: 'assets/[name]-[hash].js',
        chunkFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',
        // 静态资源内联
        inlineDynamicImports: false,
      },
      // 外部依赖
      external: ['electron'],
      // 插件
      plugins: [],
    },
    // 临时文件目录
    tmpDir: '.vite',
    // 压缩选项
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },

  // === 依赖优化 ===
  optimizeDeps: {
    include: ['react', 'react-dom', 'lodash-es'],
    exclude: ['@internal/pkg'],
    entries: ['./index.html'],
    force: false,  // 强制重新预构建
  },

  // === 路径解析 ===
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),
      '@components': resolve(__dirname, 'src/components'),
      '@utils': resolve(__dirname, 'src/utils'),
      '@hooks': resolve(__dirname, 'src/hooks'),
    },
    extensions: ['.mjs', '.js', '.mts', '.ts', '.jsx', '.tsx', '.json'],
    // 条件导出
    conditions: [],
    // 解析 mainFields
    mainFields: ['module', 'jsnext:main', 'jsnext'],
  },

  // === 服务端选项 ===
  server: {
    port: 5173,
    host: true,  // 监听所有地址
    strictPort: false,  // 端口被占用时尝试下一个
    https: false,  // 或传入 https 配置对象
    open: false,  // 开发时打开浏览器
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        configure: (proxy) => {
          // 配置代理
        },
      },
    },
    cors: true,  // 启用 CORS
    headers: {
      // 自定义响应头
    },
    // 服务端渲染
    ssr: {
      external: ['node-fetch'],
      noExternal: [],
    },
    // 监听文件变化
    watch: {
      usePolling: false,  // 某些环境下需要启用
      ignored: ['**/node_modules/**', '**/dist/**'],
    },
  },

  // === 预览选项 ===
  preview: {
    port: 4173,
    host: true,
    strictPort: false,
    proxy: {
      '/api': 'http://localhost:3000',
    },
  },

  // === 环境 API ===
  environments: {
    client: {},
    ssr: {},
    edge: {},
  },

  // === 插件 ===
  plugins: [
    react({
      // React 插件配置
      include: '**/*.{jsx,tsx}',
      exclude: '**/node_modules/**',
      babel: {
        plugins: [],
        presets: [],
      },
      // SWC 替代 Babel
      parser: 'babel',  // 'babel' | 'sucrase' | 'swc'
    }),
    // Vue 插件
    vue({
      template: {
        compilerOptions: {
          // 自定义指令
        },
      },
      script: {
        defineModel: true,
        propsDestructure: true,
      },
    }),
  ],
});
```

### 目录结构最佳实践

**推荐的 Vite 项目结构：**

```
my-app/
├── src/                       # 源码
│   ├── main.jsx              # 入口文件
│   ├── App.jsx               # 根组件
│   ├── app.css               # 全局样式
│   ├── components/           # 组件
│   │   ├── ui/             # UI 组件
│   │   │   ├── Button.jsx
│   │   │   ├── Input.jsx
│   │   │   └── Modal.jsx
│   │   ├── features/       # 功能组件
│   │   │   ├── Auth/
│   │   │   └── Dashboard/
│   │   └── layout/        # 布局组件
│   │       ├── Header.jsx
│   │       └── Footer.jsx
│   ├── pages/               # 页面（如果使用 React Router）
│   │   ├── Home.jsx
│   │   ├── About.jsx
│   │   └── NotFound.jsx
│   ├── hooks/                # 自定义 Hooks
│   │   ├── useLocalStorage.js
│   │   └── useFetch.js
│   ├── utils/               # 工具函数
│   │   ├── format.js
│   │   └── validation.js
│   ├── lib/                 # 第三方库封装
│   │   ├── api.js
│   │   └── storage.js
│   ├── stores/              # 状态管理（Zustand/Jotai）
│   │   ├── userStore.js
│   │   └── cartStore.js
│   ├── assets/              # 资源
│   │   ├── images/
│   │   ├── icons/
│   │   └── fonts/
│   └── styles/              # 样式
│       ├── variables.css
│       └── global.css
├── public/                   # 静态资源（直接复制到输出）
│   ├── favicon.ico
│   └── robots.txt
├── index.html                # HTML 入口
├── vite.config.js           # Vite 配置
├── tsconfig.json            # TypeScript 配置
├── jsconfig.json            # JavaScript 配置
├── .env                    # 环境变量
├── .env.local              # 本地环境变量
├── .env.production         # 生产环境变量
├── .eslintrc.cjs           # ESLint 配置
├── .prettierrc.json        # Prettier 配置
└── package.json
```

---

## Plugin 系统深入

### 插件生命周期

Vite 插件在开发和构建阶段有不同的生命周期钩子：

```typescript
import type { Plugin } from 'vite';

export default function myPlugin(options = {}): Plugin {
  return {
    name: 'vite-plugin-example',  // 必须唯一
    enforce: 'pre',  // 'pre' | 'post' | undefined

    // === 构建阶段钩子 ===
    buildStart() {
      // 构建开始时调用
      // 适合初始化资源
    },

    configResolved(config) {
      // 配置解析完成后调用
      // 可读取最终配置
    },

    configureServer(server) {
      // 配置开发服务器
      // 可添加中间件、路由等
    },

    transformIndexHtml(html) {
      // 转换 index.html
      // 可添加脚本、样式等
    },

    // === 模块处理钩子 ===
    resolveId(source, importer, options) {
      // 解析模块 ID
      // 返回 null 使用默认解析
      // 返回 { id } 表示重定向
    },

    load(id) {
      // 加载模块
      // 返回 null 使用默认加载
      // 返回 { code } 表示自定义内容
    },

    transform(code, id) {
      // 转换模块内容
      // 返回 { code, map } 或 null
    },

    // === 热更新钩子 ===
    handleHotUpdate(ctx) {
      // 文件变化时调用
      // 可自定义热更新行为
    },

    // === 构建阶段钩子 ===
    renderChunk(code, chunk, options) {
      // 渲染 chunk
      // 可用于代码优化
    },

    generateBundle(options, bundle, isWrite) {
      // 生成最终产物前调用
      // 可修改产物
    },

    writeBundle(options, bundle) {
      // 写入产物后调用
    },

    buildEnd() {
      // 构建结束时调用
    },
  };
}
```

### 常用插件实战

#### 1. 环境变量注入插件

```typescript
// plugins/inject-env.ts
import type { Plugin } from 'vite';
import { loadEnv } from 'vite';

export default function injectEnvPlugin(mode: string): Plugin {
  return {
    name: 'vite-plugin-inject-env',
    config(config, env) {
      const envDir = config.envDir || process.cwd();
      const viteEnv = loadEnv(mode, envDir, '');

      return {
        define: {
          // 注入环境变量到代码
          'import.meta.env.VITE_APP_TITLE': JSON.stringify(viteEnv.VITE_APP_TITLE || 'My App'),
          'import.meta.env.VITE_API_URL': JSON.stringify(viteEnv.VITE_API_URL || ''),
        },
      };
    },
  };
}
```

#### 2. SVG 组件化插件

```typescript
// plugins/svg-component.ts
import type { Plugin } from 'vite';
import { readFileSync } from 'fs';
import { resolve, parse } from 'path';

export default function svgComponentPlugin(): Plugin {
  return {
    name: 'vite-plugin-svg-component',
    enforce: 'pre',

    resolveId(source, importer) {
      if (source.endsWith('.svg?component')) {
        const id = source.replace('?component', '');
        return `\0svg:${id}`;
      }
    },

    load(id) {
      if (id.startsWith('\0svg:')) {
        const filePath = id.replace('\0svg:', '');
        const svgContent = readFileSync(filePath, 'utf-8');
        const { name } = parse(filePath);

        // 转换为 React 组件
        const component = `
import React from 'react';

const ${name}Icon = (props) => (
  ${svgContent
    .replace('<svg', '<svg {...props}')
    .replace('class=', 'className=')
    .replace(/fill="[^"]*"/g, (match) => {
      // 移除硬编码的 fill
      return match.includes('currentColor') ? match : '';
    })
  }
);

${name}Icon.displayName = '${name}Icon';

export default ${name}Icon;
        `;

        return component;
      }
    },
  };
}
```

#### 3. Mock 数据插件

```typescript
// plugins/mock.ts
import type { Plugin, ViteDevServer } from 'vite';
import type { Connect } from 'express';

interface Mock {
  method: 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';
  url: string;
  response: any;
  delay?: number;
}

export default function mockPlugin(mocks: Mock[]): Plugin {
  return {
    name: 'vite-plugin-mock',
    configureServer(server: ViteDevServer) {
      server.middlewares.use('/mock' as any, (req: Connect.IncomingMessage, res: any, next: any) => {
        const mock = mocks.find(
          (m) => m.method === req.method && m.url === req.url
        );

        if (!mock) {
          return next();
        }

        setTimeout(() => {
          res.setHeader('Content-Type', 'application/json');
          res.end(JSON.stringify(mock.response));
        }, mock.delay || 0);
      });
    },
  };
}

// 使用
// vite.config.ts
import mockPlugin from './plugins/mock';

export default defineConfig({
  plugins: [
    mockPlugin([
      {
        method: 'GET',
        url: '/api/users',
        response: [
          { id: 1, name: 'Alice' },
          { id: 2, name: 'Bob' },
        ],
      },
      {
        method: 'POST',
        url: '/api/users',
        response: { id: 3, name: 'Charlie' },
        delay: 1000,
      },
    ]),
  ],
});
```

### 插件开发最佳实践

#### 1. 类型安全的插件选项

```typescript
import type { Plugin } from 'vite';

interface MyPluginOptions {
  include?: string | string[];
  exclude?: string | string[];
  transform?: (code: string, id: string) => string;
  verbose?: boolean;
}

export default function myPlugin(options: MyPluginOptions = {}): Plugin {
  const {
    include = ['**/*.js', '**/*.ts'],
    exclude = ['**/node_modules/**'],
    transform = (code) => code,
    verbose = false,
  } = options;

  return {
    name: 'vite-plugin-my-plugin',

    transform(code, id) {
      // 检查是否应该处理
      if (!shouldTransform(id, include, exclude)) {
        return null;
      }

      if (verbose) {
        console.log(`[vite-plugin-my-plugin] Transforming ${id}`);
      }

      return {
        code: transform(code, id),
        map: null,
      };
    },
  };
}

function shouldTransform(
  id: string,
  include: string | string[],
  exclude: string | string[]
): boolean {
  const includes = Array.isArray(include) ? include : [include];
  const excludes = Array.isArray(exclude) ? exclude : [exclude];

  return matches(id, includes) && !matches(id, excludes);
}

function matches(id: string, patterns: string[]): boolean {
  return patterns.some((pattern) => {
    if (pattern.includes('*')) {
      const regex = new RegExp(
        '^' + pattern.replace(/\*/g, '.*').replace(/\?/g, '.') + '$'
      );
      return regex.test(id);
    }
    return id.includes(pattern);
  });
}
```

#### 2. 异步插件

```typescript
import type { Plugin } from 'vite';

export default function asyncPlugin(): Plugin {
  return {
    name: 'vite-plugin-async',

    async transform(code, id) {
      // 异步处理
      const result = await someAsyncOperation(code);

      return {
        code: result,
        map: null,
      };
    },

    async buildStart() {
      // 异步初始化
      await initializePlugin();
    },

    async configResolved(config) {
      // 异步配置处理
      const customConfig = await fetchConfig();
      Object.assign(config, customConfig);
    },
  };
}
```

---

## 高级构建配置

### Code Splitting 配置

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // Manual chunks - 手动分割
        manualChunks: (id) => {
          // 第三方库
          if (id.includes('node_modules')) {
            if (id.includes('react')) {
              return 'react-vendor';
            }
            if (id.includes('lodash')) {
              return 'lodash-vendor';
            }
            return 'vendor';
          }

          // 功能模块
          if (id.includes('/features/auth/')) {
            return 'auth';
          }
          if (id.includes('/features/dashboard/')) {
            return 'dashboard';
          }
        },

        // 动态导入分割
        manualChunks: (id, { getModuleInfo }) => {
          const moduleInfo = getModuleInfo(id);
          if (moduleInfo?.isDynamicEntry) {
            return `chunk-${id.split('/').pop()}`;
          }
        },
      },
    },
  },
});
```

### 静态资源处理

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    assetsInlineLimit: 4096,  // 4KB 以下的资源内联

    rollupOptions: {
      output: {
        assetFileNames: (assetInfo) => {
          const info = assetInfo.name.split('.');
          const ext = info[info.length - 1];

          // 按类型分组
          if (/png|jpe?g|svg|gif|webp/.test(ext)) {
            return 'assets/images/[name]-[hash][extname]';
          }
          if (/ttf|woff2?|eot/.test(ext)) {
            return 'assets/fonts/[name]-[hash][extname]';
          }
          return 'assets/[name]-[hash][extname]';
        },
      },
    },
  },
});
```

### CSS 代码分割

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    cssCodeSplit: true,  // 默认 true

    // 如果想合并所有 CSS
    // cssCodeSplit: false,

    rollupOptions: {
      output: {
        // 为每个 entry 生成单独的 CSS
        // 或使用插件手动处理
      },
    },
  },
});
```

### Sourcemap 配置

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    // 构建时 sourcemap
    sourcemap: false,  // 不生成
    // sourcemap: true,  // 生成完整 sourcemap
    // sourcemap: 'inline',  // 内联 sourcemap
    // sourcemap: 'hidden',  // 生成但不引用

    rollupOptions: {
      output: {
        // Chunk sourcemap
        sourcemap: true,
      },
    },
  },

  css: {
    // CSS sourcemap
    devSourcemap: true,
  },
});
```

---

## 性能优化实战

### 开发环境优化

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    // 预加载
    warmup: {
      // 预热常用模块
      clientFiles: ['./src/App.tsx', './src/pages/Home.tsx'],
    },
  },

  optimizeDeps: {
    // 预构建依赖
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      '@tanstack/react-query',
      'zustand',
      'lodash-es',
      'dayjs',
    ],
    // 排除不应预构建的包
    exclude: ['@vite/client'],
  },
});
```

### 生产构建优化

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    target: 'esnext',

    // 代码压缩
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,      // 移除 console.log
        drop_debugger: true,     // 移除 debugger
        pure_funcs: ['console.log', 'console.info'],
        passes: 2,               // 多轮压缩
        unsafe_arrows: true,      // 转换箭头函数
        unsafe_methods: true,     // 优化不安全方法
      },
      mangle: {
        safari10: true,         // Safari 10 兼容性
      },
      format: {
        comments: false,          // 移除注释
      },
    },

    // CSS 压缩
    cssCodeSplit: true,
    cssMinify: true,

    // Chunk 分割
    rollupOptions: {
      output: {
        // 块名哈希
        entryFileNames: 'assets/[name]-[hash].js',
        chunkFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',

        // 手动分割
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            const name = id.split('node_modules/')[1].split('/')[0];
            if (name.startsWith('@')) {
              return `vendor-${name.split('/')[0].slice(1)}`;
            }
            return `vendor-${name}`;
          }
        },

        // 分隔符
        chunkFileNames: (chunkInfo) => {
          const facadeModuleId = chunkInfo.facadeModuleId
            ? chunkInfo.facadeModuleId.split('/').slice(-2, -1)[0]
            : 'shared';
          return `assets/[name]-${facadeModuleId}-[hash].js`;
        },
      },
    },

    // 报告压缩大小
    chunkSizeWarningLimit: 500,

    // 目标浏览器
    target: ['es2020', 'edge88', 'firefox78', 'chrome87', 'safari14'],

    // 临时代码分割
    rollupOptions: {
      output: {
        // 实验性：内联动态导入
        inlineDynamicImports: false,
      },
    },
  },
});
```

### 分析构建产物

```typescript
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'stats.html',        // 报告文件
      open: true,                    // 构建后自动打开
      gzipSize: true,                // 显示 gzip 大小
      brotliSize: true,              // 显示 brotli 大小
      template: 'treemap',            // 展示模板
    }),
  ],
});
```

---

## TypeScript 集成

### TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "lib": ["ESNext", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path Mapping */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@hooks/*": ["src/hooks/*"],
      "@utils/*": ["src/utils/*"],
      "@stores/*": ["src/stores/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```json
// tsconfig.node.json
{
  "compilerOptions": {
    "composite": true,
    "skipLibCheck": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowSyntheticDefaultImports": true
  },
  "include": ["vite.config.ts"]
}
```

### 类型检查

```typescript
// vite.config.ts
export default defineConfig({
  // 构建时类型检查
  esbuild: {
    // 排除类型检查
    exclude: [],
    // 自定义 JSX
    jsxFactory: 'React.createElement',
    jsxFragment: 'React.Fragment',
  },
});
```

---

## 环境变量深入

### 环境变量文件

```
.env                  # 所有环境加载
.env.local           # 所有环境，本地覆盖
.env.[mode]          # 特定模式
.env.[mode].local    # 特定模式，本地覆盖
```

**加载优先级：**
1. `envDir` 指定的目录
2. `process.cwd()`
3. 优先级：`mode.local` > `mode` > `local` > ``

### 安全使用环境变量

```typescript
// 定义接口
// src/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_URL: string;
  readonly VITE_WS_URL: string;
  readonly VITE_PUBLIC_KEY: string;
  readonly VITE_ANALYTICS_ID: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 运行时环境变量

```typescript
// 方式一：构建时注入
// vite.config.ts
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString()),
  },
});

// 方式二：服务端注入
// 在服务端设置 cookie 或 header
// vite.config.ts
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:3000',
        changeOrigin: true,
        configure: (proxy) => {
          proxy.on('proxyReq', (proxyReq, req) => {
            // 注入服务端数据
            proxyReq.setHeader('x-app-version', process.env.APP_VERSION);
          });
        },
      },
    },
  },
});
```

---

## 测试配置

### Vitest 集成

```bash
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react @testing-library/jest-dom
```

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.ts'],
    include: ['tests/**/*.{test,spec}.{js,mjs,cjs,ts,mts,cts,jsx,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/virtual:*',
      ],
    },
  },
});
```

```typescript
// tests/setup.ts
import '@testing-library/jest-dom';

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation((query) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(),
    removeListener: jest.fn(),
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

```typescript
// tests/example.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import App from '../src/App';

describe('App', () => {
  it('renders without crashing', () => {
    render(<App />);
    expect(screen.getByText(/learn/i)).toBeInTheDocument();
  });

  it('handles user interaction', async () => {
    const user = userEvent.setup();
    render(<App />);

    const button = screen.getByRole('button');
    await user.click(button);

    expect(button).toHaveAttribute('aria-pressed', 'true');
  });
});
```

---

## 常见陷阱与解决方案

### 构建问题

**1. 动态导入失败**

**问题：** 生产环境动态导入失败。

**原因：** 通常是路径别名或资源路径配置问题。

```typescript
// ❌ 问题配置
resolve: {
  alias: {
    '@': '/src',  // 绝对路径在某些情况下有问题
  },
}

// ✅ 正确配置
resolve: {
  alias: {
    '@': resolve(__dirname, 'src'),
  },
}
```

**2. 第三方库构建失败**

**问题：** 某些 npm 包在构建时报错。

**解决方案：** 使用 `noExternal` 或排除。

```typescript
// vite.config.ts
export default defineConfig({
  ssr: {
    noExternal: ['some-package'],
    external: ['another-package'],
  },
});
```

### 开发问题

**1. HMR 不工作**

**问题：** 文件修改后页面不更新。

**解决方案：** 检查文件是否在 `server.watch.ignored` 中。

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    watch: {
      ignored: ['!**/src/**'],  // 排除某些文件
    },
  },
});
```

**2. 内存溢出**

**问题：** 大项目开发时内存溢出。

**解决方案：** 限制预构建和优化缓存。

```typescript
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    maxPlayers: 20,
    minify: false,  // 开发环境跳过压缩
  },
  build: {
    target: 'esnext',
    minify: false,
  },
});
```

---

## 附录：资源链接

### 官方资源

- [Vite 官方文档](https://vitejs.dev)
- [Vite GitHub](https://github.com/vitejs/vite)
- [Vite Discord](https://discord.gg/vite)

### 学习资源

- [Vite 指南](https://vitejs.dev/guide)
- [Vite 插件列表](https://github.com/vitejs/awesome-vite)
- [Vite Example](https://github.com/vitejs/vite/tree/main/packages/create-vite)

### 相关工具

- [Vitest](https://vitest.dev) - Vite 原生测试框架
- [Playwright](https://playwright.dev) - E2E 测试
- [Rollup](https://rollupjs.org) - Vite 底层打包器
- [esbuild](https://esbuild.github.io) - Vite 使用的编译器
