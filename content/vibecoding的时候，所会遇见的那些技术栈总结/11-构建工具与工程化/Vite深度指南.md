# Vite 深度完全指南

> [!NOTE]
> 本文档是 Vite 5.x 的完整参考，涵盖配置详解、插件生态、开发服务器、性能优化及生产构建策略。Vite 以其极快的冷启动和热更新速度，正在成为现代前端开发的首选构建工具。

---

## 1. 技术概述与定位

### 1.1 Vite 的诞生背景

在前端开发工具的发展历程中，Webpack 长期占据着统治地位。然而，随着项目规模的增长，Webpack 的一些固有缺陷逐渐暴露出来。冷启动时间长是最大的痛点——一个包含数千个模块的中型项目，Webpack 的启动时间可能达到 30 秒甚至更久。热更新效率低下同样是困扰开发者的问题，每次代码修改后，需要等待数秒才能看到变更效果。这些问题严重影响了开发体验和团队效率。

Vite 的出现正是为了解决这些痛点。Vite 由 Evan You（Vue.js 的 creator）于 2020 年创建，它采用了一种全新的开发服务器架构，利用浏览器原生 ES Modules 的能力来实现极速的开发体验。Vite 不需要打包开发阶段的代码，而是直接在浏览器中运行原生 ES 模块，通过请求按需编译和提供服务。这种设计从根本上解决了冷启动和热更新的性能问题。

### 1.2 Vite 的核心设计理念

Vite 的设计哲学可以概括为「极速开发，优化生产」。在开发阶段，Vite 利用浏览器原生 ES Modules 的能力，实现了真正的即时启动和毫秒级热更新。在生产阶段，Vite 使用 Rollup 进行打包优化，生成高度优化的生产 bundle。

这种分而治之的策略是 Vite 区别于其他工具的关键。传统打包工具在开发阶段也要进行完整的打包工作，而 Vite 跳过了这一步骤，只在浏览器需要时才进行模块的编译和转换。这种懒编译（Lazy Compilation）的方式使得开发服务器可以在亚秒级时间内启动完成。

### 1.3 Vite 与其他构建工具的对比

| 特性 | Vite | Webpack | Parcel | esbuild |
|------|------|---------|--------|---------|
| **冷启动速度** | <500ms | 10-60s | 1-5s | <100ms |
| **热更新速度** | <50ms | 2-10s | <1s | <50ms |
| **配置复杂度** | 低 | 高 | 极低 | 无需配置 |
| **插件生态** | 丰富 | 极其丰富 | 一般 | 有限 |
| **Tree Shaking** | 优秀 | 良好 | 良好 | 优秀 |
| **生产打包** | Rollup | webpack | Parcel | 可选 |
| **TypeScript 支持** | 原生 | 需要配置 | 原生 | 原生 |
| **CSS Modules** | 原生支持 | 需要配置 | 支持 | 原生 |

### 1.4 Vite 的技术架构

Vite 的技术架构分为两部分：开发服务器和生产构建器。开发服务器基于原生 ES Modules 构建，利用浏览器的原生模块解析能力，避免了传统打包工具的全量打包过程。生产构建器基于 Rollup，继承了其成熟的打包优化能力。

```
Vite 技术架构:

┌─────────────────────────────────────────────────────────────────┐
│                      Vite 架构概览                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    开发阶段 (Dev)                        │   │
│   │                                                          │   │
│   │   ┌─────────────┐    ┌─────────────┐    ┌───────────┐  │   │
│   │   │   Browser   │───→│   Vite Dev  │───→│  esbuild  │  │   │
│   │   │   ESM      │    │   Server    │    │  Transform│  │   │
│   │   │   Request  │←───│   <500ms    │←───│  <50ms    │  │   │
│   │   └─────────────┘    └─────────────┘    └───────────┘  │   │
│   │         │                  │                   │        │   │
│   │         └──────────────────┴───────────────────┘        │   │
│   │                          │                              │   │
│   │                          ▼                              │   │
│   │                   ┌─────────────┐                       │   │
│   │                   │  File       │                       │   │
│   │                   │  System     │                       │   │
│   │                   └─────────────┘                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                    生产阶段 (Build)                      │   │
│   │                                                          │   │
│   │   ┌─────────────┐    ┌─────────────┐    ┌───────────┐  │   │
│   │   │  Source     │───→│   Rollup    │───→│  Optimized│  │   │
│   │   │  Code      │    │   Bundler   │    │  Output   │  │   │
│   │   └─────────────┘    └─────────────┘    └───────────┘  │   │
│   │                                                          │   │
│   │   特性:                                                   │   │
│   │   • Tree Shaking                                          │   │
│   │   • Code Splitting                                        │   │
│   │   • Chunk Splitting                                        │   │
│   │   • Asset Optimization                                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 1.5 Vite 的适用场景

Vite 特别适合以下场景：

- **新项目启动**：Vite 的极速启动特性使其成为新项目的理想选择
- **前端框架项目**：Vite 官方支持 Vue、React、Svelte、Preact、Lit 等框架
- **组件库开发**：Vite 的开发体验和 Rollup 的打包优化非常适合组件库
- **TypeScript 项目**：Vite 原生支持 TypeScript，无需额外配置
- **快速原型开发**：Vite 的低配置特性使其非常适合快速原型开发
- **静态网站生成**：配合 SSG 插件，Vite 可以用于生成静态网站

---

## 2. 完整安装与配置

### 2.1 项目初始化

#### 2.1.1 使用模板创建项目

Vite 提供了官方的项目模板，可以通过 npm、yarn 或 pnpm 快速创建：

```bash
# npm
npm create vite@latest my-vue-project -- --template vue
npm create vite@latest my-react-project -- --template react
npm create vite@latest my-vue-ts-project -- --template vue-ts
npm create vite@latest my-react-ts-project -- --template react-ts
npm create vite@latest my-svelte-project -- --template svelte
npm create vite@latest my-svelte-ts-project -- --template svelte-ts

# yarn
yarn create vite my-project -- --template vue-ts

# pnpm
pnpm create vite my-project -- --template vue-ts
```

#### 2.1.2 手动安装 Vite

对于已有项目，可以手动安装 Vite：

```bash
# 安装 Vite 和 TypeScript（如果使用 TypeScript）
npm install vite typescript -D

# 安装 Vue 插件（如果使用 Vue）
npm install @vitejs/plugin-vue -D

# 安装 React 插件（如果使用 React）
npm install @vitejs/plugin-react -D

# 安装 Node 类型定义
npm install @types/node -D
```

#### 2.1.3 初始化 package.json

```json
{
  "name": "my-vite-project",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx,.cts,.mts --fix"
  },
  "dependencies": {
    "vue": "^3.4.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.0.0",
    "vite": "^5.4.0",
    "typescript": "^5.5.0",
    "vue-tsc": "^2.0.0"
  }
}
```

### 2.2 配置文件详解

#### 2.2.1 vite.config.ts 基础结构

Vite 的配置文件支持多种格式：`vite.config.js`、`vite.config.ts`、`vite.config.mjs`、`vite.config.mts`。推荐使用 TypeScript 格式以获得完整的类型提示。

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  // 基础配置
  base: '/',
  mode: 'development',
  define: {
    'process.env': {},
  },
  publicDir: 'public',

  // 插件配置
  plugins: [
    vue({
      include: [/\.vue$/],
      template: {
        compilerOptions: {
          isCustomElement: (tag) => false,
        },
      },
    }),
    react({
      include: '**/*.{jsx,tsx}',
    }),
  ],

  // 路径解析配置
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@assets': path.resolve(__dirname, './src/assets'),
      '@views': path.resolve(__dirname, './src/views'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
      '@types': path.resolve(__dirname, './src/types'),
    },
    extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'],
  },

  // CSS 配置
  css: {
    modules: {
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
      less: {
        modifyVars: {
          'primary-color': '#1890ff',
        },
      },
    },
    devSourcemap: true,
  },

  // 构建配置
  build: {
    target: 'esnext',
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'esbuild',
    chunkSizeWarningLimit: 500,
    rollupOptions: {
      input: {
        main: path.resolve(__dirname, 'index.html'),
      },
      output: {
        entryFileNames: 'assets/[name]-[hash].js',
        chunkFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',
        manualChunks: {
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          'vendor-vendor': ['lodash', 'axios', 'dayjs'],
        },
      },
      external: [],
      plugins: [],
    },
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
    reportCompressedSize: true,
    rollupOptions: {},
  },

  // 开发服务器配置
  server: {
    port: 3000,
    host: '0.0.0.0',
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
    hmr: {
      overlay: true,
      port: 3001,
    },
    cors: true,
    force: true,
    watch: {
      ignored: ['**/node_modules/**', 'dist/**'],
    },
  },

  // 依赖优化配置
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
    exclude: [],
    esbuildOptions: {
      target: 'esnext',
    },
  },

  // 预构建配置
  preview: {
    port: 4173,
    host: '0.0.0.0',
    cors: true,
    proxy: {},
  },
});
```

#### 2.2.2 TypeScript 配置

Vite 项目需要配置 `tsconfig.json` 以支持 TypeScript：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
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

    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@utils/*": ["./src/utils/*"],
      "@hooks/*": ["./src/hooks/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

```json
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

#### 2.2.3 环境变量文件

Vite 支持多种环境变量文件，用于不同场景的配置：

```bash
# .env                  # 所有环境都会加载
# .env.local            # 所有环境，会被 git 忽略
# .env.[mode]           # 特定模式
# .env.[mode].local     # 特定模式，会被 git 忽略

# 示例文件内容
VITE_APP_TITLE=我的应用
VITE_API_BASE_URL=https://api.example.com
VITE_UPLOAD_MAX_SIZE=5242880
VITE_ENABLE_ANALYTICS=false
VITE_MAPBOX_TOKEN=pk.xxxxxx
```

```typescript
// src/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_BASE_URL: string;
  readonly VITE_UPLOAD_MAX_SIZE: string;
  readonly VITE_ENABLE_ANALYTICS: string;
  readonly VITE_MAPBOX_TOKEN: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

declare module '*.vue' {
  import type { DefineComponent } from 'vue';
  const component: DefineComponent<object, object, unknown>;
  export default component;
}
```

### 2.3 框架特定配置

#### 2.3.1 Vue 项目配置

```typescript
// vite.config.ts - Vue 项目
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import path from 'path';

export default defineConfig({
  plugins: [
    vue({
      // Vue 编译器选项
      template: {
        compilerOptions: {
          // 自定义元素处理
          isCustomElement: (tag) => {
            return tag.startsWith('ion-') || tag === 'micro-app';
          },
          // 指令转换
          directiveTransforms: {
            on: (dir, node, context) => {
              // 自定义指令转换逻辑
            },
          },
        },
        // 转换函数
        transformAssetUrls: {
          video: ['src', 'href'],
          audio: ['src', 'href'],
          source: ['src', 'href'],
          img: ['src'],
          image: ['xlink:href'],
          'use': ['xlink:href'],
        },
      },
      // Reactivity transform
      reactivityTransform: true,
    }),
    vueJsx({
      // JSX 配置
      transformOn: true,
      mergeProps: true,
      enableObjectSlots: true,
    }),
  ],

  // CSS 配置
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss"; @import "@/styles/mixins.scss";`,
      },
    },
  },
});
```

#### 2.3.2 React 项目配置

```typescript
// vite.config.ts - React 项目
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [
    react({
      // 使用官方 React JSX 运行时
      jsxImportSource: 'react',
      // 自定义 Babel 插件
      babel: {
        plugins: [
          ['@babel/plugin-proposal-decorators', { legacy: true }],
          ['@babel/plugin-transform-class-properties', { loose: true }],
        ],
        babelrc: false,
        configFile: false,
      },
      // Fast Refresh 配置
      fastRefresh: true,
      // 启用 React Refresh 入口
      refreshLogic: true,
    }),
  ],

  // 优化配置
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'zustand',
      'axios',
    ],
    esbuildOptions: {
      target: 'esnext',
    },
  },
});
```

---

## 3. 核心概念详解

### 3.1 开发服务器原理

#### 3.1.1 原生 ES Modules 架构

Vite 开发服务器的核心是利用浏览器原生 ES Modules 的能力。当浏览器请求一个模块时，Vite 服务器拦截请求，按需编译模块并返回给浏览器。这种方式避免了传统打包工具的全量打包过程。

```
原生 ES Modules 请求流程:

┌─────────────────────────────────────────────────────────────────┐
│                        Browser                                   │
│                                                                  │
│   1. 请求 main.ts                                                 │
│   └──────────────→ ┌──────────────┐                              │
│                   │  Vite Dev    │                              │
│                   │  Server      │                              │
│   2. 返回转换后的 │  localhost    │                              │
│   ES Module       │  :3000        │                              │
│   ←────────────── └──────────────┘                              │
│                                                                  │
│   3. 浏览器解析依赖                                                │
│   └──────────────→ ┌──────────────┐                              │
│                   │  按需编译      │                              │
│   4. 返回模块A    │  每个模块      │                              │
│   ←────────────── └──────────────┘                              │
│                                                                  │
│   5. 浏览器解析模块A的依赖                                         │
│   └──────────────→ ┌──────────────┐                              │
│                   │  按需编译      │                              │
│   6. 返回模块B    │  模块B         │                              │
│   ←────────────── └──────────────┘                              │
│                                                                  │
│   ... 重复直到所有依赖加载完成                                      │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 依赖预构建

Vite 在首次启动时会进行依赖预构建（Dependency Pre-bundling）。这个过程使用 esbuild 将 CommonJS 格式的依赖转换为 ESM 格式，并合并小文件以减少请求数量。

```typescript
// 预构建配置
export default defineConfig({
  optimizeDeps: {
    // 预构建的依赖
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'lodash',
      'lodash-es',
      'axios',
      'dayjs',
    ],
    // 排除的依赖
    exclude: ['@vite/client'],
    // esbuild 选项
    esbuildOptions: {
      target: 'esnext',
      treeShaking: true,
    },
    // 缓存目录
    cacheDir: 'node_modules/.vite',
  },
});
```

#### 3.1.3 热模块替换（HMR）

Vite 的 HMR 系统非常高效，只更新实际修改的模块，无需重新加载整个页面。

```typescript
// HMR API 示例
// src/hmr.ts
import { createHotContext } from '@vite/client';

if (import.meta.hot) {
  // 创建 HMR 上下文
  createHotContext(import.meta.url);

  // 监听模块更新
  import.meta.hot.accept(() => {
    console.log('模块已更新');
  });

  // 接受自身更新
  import.meta.hot.accept((newModule) => {
    console.log('新模块内容:', newModule);
  });

  // 监听更新失败
  import.meta.hot.on('vite:beforeUpdate', (data) => {
    console.log('即将更新:', data);
  });

  import.meta.hot.on('vite:afterUpdate', (data) => {
    console.log('更新完成:', data);
  });

  // 错误处理
  import.meta.hot.on('vite:error', (data) => {
    console.error('更新错误:', data);
  });

  // 手动使模块失效
  import.meta.hot.decline();

  // 延迟更新
  import.meta.hot.dispose((data) => {
    // 清理工作
    data.cachedData = moduleCache;
  });
}
```

### 3.2 Rollup 构建系统

#### 3.2.1 构建流程

Vite 的生产构建使用 Rollup。Rollup 是专为 JavaScript 库设计的打包工具，以高效的 Tree Shaking 能力著称。

```
Rollup 构建流程:

┌─────────────────────────────────────────────────────────────────┐
│                    Rollup 构建流程                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Source Files                                                    │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│   │ main.js │  │ util.js │  │helper.js│  │ lib.js │            │
│   └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │
│        │            │            │            │                 │
│        └────────────┴─────┬──────┴────────────┘                 │
│                           │                                      │
│                           ▼                                      │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                 Module Graph Builder                     │   │
│   │                                                          │   │
│   │  ┌────────────┐    ┌────────────┐    ┌────────────┐      │   │
│   │  │  分析导入   │───→│  构建依赖  │───→│  循环依赖  │      │   │
│   │  │  导出声明   │    │   图谱    │    │   处理    │      │   │
│   │  └────────────┘    └────────────┘    └────────────┘      │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                  Tree Shaking                            │   │
│   │                                                          │   │
│   │  检测未使用的导出                                          │   │
│   │  移除死代码                                               │   │
│   │  分析副作用                                               │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │                  Bundle Generation                      │   │
│   │                                                          │   │
│   │  Chunk Splitting                                         │   │
│   │  Code Splitting                                          │   │
│   │  Asset Processing                                        │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                      │
│                           ▼                                      │
│   Output Files                                                  │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│   │ main.js │  │vendor.js│  │style.css│                         │
│   └─────────┘  └─────────┘  └─────────┘                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.2.2 代码分割策略

```typescript
// 代码分割配置
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // 手动分包
        manualChunks: (id) => {
          // React 生态系统
          if (id.includes('node_modules/react')) {
            return 'react-vendor';
          }
          if (id.includes('node_modules/react-dom')) {
            return 'react-vendor';
          }
          if (id.includes('node_modules/react-router')) {
            return 'react-vendor';
          }

          // UI 组件库
          if (id.includes('node_modules/antd')) {
            return 'antd-vendor';
          }
          if (id.includes('node_modules/@mui')) {
            return 'mui-vendor';
          }

          // 工具库
          if (id.includes('node_modules/lodash')) {
            return 'lodash-vendor';
          }
          if (id.includes('node_modules/axios')) {
            return 'axios-vendor';
          }
          if (id.includes('node_modules/dayjs')) {
            return 'dayjs-vendor';
          }

          // 图表库
          if (id.includes('node_modules/echarts')) {
            return 'echarts-vendor';
          }
          if (id.includes('node_modules/recharts')) {
            return 'recharts-vendor';
          }

          // 其他 node_modules
          if (id.includes('node_modules')) {
            return 'vendor';
          }

          // 源代码按目录分割
          if (id.includes('/views/')) {
            return 'views';
          }
          if (id.includes('/components/')) {
            return 'components';
          }
          if (id.includes('/hooks/')) {
            return 'hooks';
          }
        },

        // 分隔符
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',

        // 格式化
        compact: true,
        hoistTransitiveImports: false,
        inlineDynamicImports: false,
      },

      // 外部依赖
      external: (id) => {
        // 外部化 CDN 依赖
        if (id.startsWith('https://')) {
          return true;
        }
        // 外部化特定模块
        if (id.includes('node_modules/external-lib')) {
          return true;
        }
        return false;
      },

      // 输出格式
      output: {
        format: 'es',
        entryFileNames: '[name].js',
        // 兼容旧版浏览器
        legacyDynamicImport: true,
        generatedCode: {
          preset: 'es2015',
          constBindings: true,
          objectShorthand: true,
        },
      },
    },
  },
});
```

### 3.4 插件系统（续）

#### 3.4.1 插件开发进阶

自定义插件开发是 Vite 使用中的高级技巧，下面详细介绍插件开发的各个层面：

**虚拟模块插件开发**：
虚拟模块是一种不依赖于实际文件系统的模块，常用于生成动态配置、类型定义等：

```typescript
// vite.config.ts - 虚拟模块完整示例
import { defineConfig, Plugin, ResolvedConfig } from 'vite';
import fs from 'fs';
import path from 'path';

interface VirtualModuleOptions {
  modules: Record<string, string>;
  cache?: boolean;
}

function virtualModulesPlugin(options: VirtualModuleOptions): Plugin {
  const virtualModuleIdPrefix = 'virtual:';
  const resolvedVirtualModuleIdPrefix = '\0virtual:';
  
  // 存储所有虚拟模块的代码
  const modules = new Map<string, string>();
  
  // 初始化模块
  for (const [id, code] of Object.entries(options.modules)) {
    modules.set(id, code);
  }
  
  return {
    name: 'vite-plugin-virtual-modules',
    
    // 确保在其他插件之前解析
    enforce: 'pre',
    
    // 配置解析完成时的钩子
    configResolved(config: ResolvedConfig) {
      // 可以在这里访问最终解析的配置
      console.log('Virtual modules plugin initialized');
    },
    
    // 解析模块 ID
    resolveId(id: string, importer: string | undefined, options: { ssr?: boolean }) {
      // 处理虚拟模块请求
      if (id.startsWith(virtualModuleIdPrefix)) {
        const moduleId = id.slice(virtualModuleIdPrefix.length);
        
        // 检查模块是否存在
        if (modules.has(moduleId)) {
          return resolvedVirtualModuleIdPrefix + moduleId;
        }
      }
      
      // 标记外部模块
      if (id === 'virtual-external') {
        return { id: '\0virtual-external', external: true };
      }
      
      return null;
    },
    
    // 加载模块内容
    load(id: string) {
      // 处理虚拟模块加载
      if (id.startsWith(resolvedVirtualModuleIdPrefix)) {
        const moduleId = id.slice(resolvedVirtualModuleIdPrefix.length);
        
        if (modules.has(moduleId)) {
          let code = modules.get(moduleId)!;
          
          // 处理动态内容替换
          code = code.replace(/\$\{(\w+)\}/g, (_, key) => {
            return process.env[key] || '';
          });
          
          return {
            code,
            // 可以在此处返回 source map
            map: null,
          };
        }
      }
      
      // 处理外部虚拟模块
      if (id === '\0virtual-external') {
        return 'export const external = "external value";';
      }
      
      return null;
    },
    
    // 转换代码
    transform(code: string, id: string) {
      if (!id.startsWith(resolvedVirtualModuleIdPrefix)) {
        return null;
      }
      
      const moduleId = id.slice(resolvedVirtualModuleIdPrefix.length);
      
      // 可以在这里对虚拟模块进行额外的转换
      // 例如添加调试信息
      if (process.env.NODE_ENV === 'development') {
        return {
          code: `/* Virtual module: ${moduleId} */\n${code}`,
          map: null,
        };
      }
      
      return null;
    },
    
    // 构建开始钩子
    buildStart() {
      // 可以在构建开始时执行一些初始化工作
      console.log('Build started with virtual modules');
    },
    
    // 构建结束钩子
    buildEnd() {
      // 清理工作
      if (options.cache === false) {
        modules.clear();
      }
    },
  };
}

// 实际使用的例子
function createConfigPlugin(): Plugin {
  const modules = {
    'app-config': `
      export const config = {
        appName: '${process.env.VITE_APP_NAME || 'My App'}',
        apiUrl: '${process.env.VITE_API_URL || 'http://localhost:3000'}',
        version: '${process.env.npm_package_version || '1.0.0'}',
        buildTime: new Date('${new Date().toISOString()}').toISOString(),
        features: {
          darkMode: ${process.env.VITE_FEATURE_DARK_MODE !== 'false'},
          analytics: ${process.env.VITE_FEATURE_ANALYTICS === 'true'},
          debug: ${process.env.NODE_ENV === 'development'},
        },
      };
      
      export type Config = typeof config;
    `,
    'feature-flags': `
      export const flags = {
        ENABLE_NEW_UI: ${process.env.VITE_FLAG_NEW_UI === 'true'},
        BETA_FEATURES: ${process.env.VITE_FLAG_BETA === 'true'},
        EXPERIMENTAL: ${process.env.VITE_FLAG_EXPERIMENTAL === 'true'},
      };
      
      export type FeatureFlag = keyof typeof flags;
    `,
    'routes': `
      const routes = [
        { path: '/', name: 'Home', component: () => import('/src/pages/Home.vue') },
        { path: '/about', name: 'About', component: () => import('/src/pages/About.vue') },
        { path: '/dashboard', name: 'Dashboard', component: () => import('/src/pages/Dashboard.vue'), auth: true },
        { path: '/settings', name: 'Settings', component: () => import('/src/pages/Settings.vue'), auth: true },
      ];
      
      export default routes;
      
      export function getRouteByName(name: string) {
        return routes.find(r => r.name === name);
      }
      
      export function getAuthRoutes() {
        return routes.filter(r => r.auth);
      }
    `,
  };
  
  return virtualModulesPlugin({ modules, cache: true });
}

export default defineConfig({
  plugins: [createConfigPlugin()],
});
```

**Server 中间件插件开发**：
Vite 的开发服务器基于 Connect，可以添加自定义中间件：

```typescript
// vite.config.ts - 中间件插件示例
import { defineConfig, Plugin, ViteDevServer } from 'vite';
import express, { Request, Response, NextFunction } from 'express';

function apiProxyPlugin(): Plugin {
  return {
    name: 'vite-plugin-api-proxy',
    configureServer(server: ViteDevServer) {
      // 创建 Express 应用
      const app = express();
      
      // CORS 中间件
      app.use((req: Request, res: Response, next: NextFunction) => {
        res.header('Access-Control-Allow-Origin', '*');
        res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
        res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
        
        if (req.method === 'OPTIONS') {
          res.sendStatus(200);
          return;
        }
        
        next();
      });
      
      // JSON body 解析
      app.use(express.json());
      
      // API 路由
      app.post('/api/auth/login', async (req: Request, res: Response) => {
        const { username, password } = req.body;
        
        try {
          // 模拟登录验证
          const token = await mockLogin(username, password);
          res.json({ success: true, token });
        } catch (error) {
          res.status(401).json({ success: false, error: 'Invalid credentials' });
        }
      });
      
      app.get('/api/users', async (req: Request, res: Response) => {
        // 模拟获取用户列表
        const users = await mockGetUsers();
        res.json({ success: true, users });
      });
      
      app.get('/api/users/:id', async (req: Request, res: Response) => {
        const { id } = req.params;
        const user = await mockGetUserById(id);
        
        if (user) {
          res.json({ success: true, user });
        } else {
          res.status(404).json({ success: false, error: 'User not found' });
        }
      });
      
      app.post('/api/data', async (req: Request, res: Response) => {
        console.log('Received data:', req.body);
        res.json({ success: true, message: 'Data received' });
      });
      
      // 错误处理中间件
      app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
        console.error('API Error:', err);
        res.status(500).json({ 
          success: false, 
          error: 'Internal server error',
          details: process.env.NODE_ENV === 'development' ? err.message : undefined,
        });
      });
      
      // 将 Express 应用挂载到 Vite 服务器
      server.middlewares.use('/api', app);
      
      // 添加 WebSocket 支持（如果需要）
      server.httpServer?.on('upgrade', (request, socket, head) => {
        if (request.url === '/ws') {
          // 处理 WebSocket 连接
          console.log('WebSocket connection established');
        }
      });
    },
  };
}

// 模拟函数
async function mockLogin(username: string, password: string): Promise<string> {
  // 模拟 API 延迟
  await new Promise(resolve => setTimeout(resolve, 500));
  
  if (username === 'admin' && password === 'admin') {
    return 'mock-jwt-token-' + Date.now();
  }
  
  throw new Error('Invalid credentials');
}

async function mockGetUsers() {
  await new Promise(resolve => setTimeout(resolve, 200));
  
  return [
    { id: '1', name: 'Alice', email: 'alice@example.com', role: 'admin' },
    { id: '2', name: 'Bob', email: 'bob@example.com', role: 'user' },
    { id: '3', name: 'Charlie', email: 'charlie@example.com', role: 'user' },
  ];
}

async function mockGetUserById(id: string) {
  const users = await mockGetUsers();
  return users.find(u => u.id === id);
}

export default defineConfig({
  plugins: [apiProxyPlugin()],
  server: {
    port: 3000,
    host: true,
  },
});
```

#### 3.4.2 插件间的交互

Vite 插件可以相互通信，这在大型项目中非常有用：

```typescript
// vite.config.ts - 插件间通信示例

// 定义模块类型
interface ModuleInfo {
  id: string;
  path: string;
  size: number;
  dependencies: string[];
}

// 共享上下文
const pluginContext = {
  modules: new Map<string, ModuleInfo>(),
  buildStartTime: 0,
};

// 插件 A：收集模块信息
function moduleCollectorPlugin(): Plugin {
  return {
    name: 'module-collector',
    
    buildStart() {
      pluginContext.buildStartTime = Date.now();
    },
    
    // 收集模块信息
    transform(code: string, id: string) {
      if (!id.includes('node_modules')) {
        // 计算模块大小
        const size = Buffer.byteLength(code, 'utf8');
        
        // 提取依赖（简化版本）
        const deps = [];
        const importRegex = /import\s+.*?from\s+['"](.+?)['"]/g;
        let match;
        while ((match = importRegex.exec(code)) !== null) {
          deps.push(match[1]);
        }
        
        pluginContext.modules.set(id, {
          id,
          path: id,
          size,
          dependencies: deps,
        });
      }
      
      return null;
    },
    
    // 将模块信息写入全局
    generateBundle(options, bundle) {
      // 可以在此处访问所有打包的资源
      // 用于生成构建报告
    },
  };
}

// 插件 B：使用收集的信息
function moduleReporterPlugin(): Plugin {
  let config: ResolvedConfig;
  
  return {
    name: 'module-reporter',
    
    configResolved(resolvedConfig) {
      config = resolvedConfig;
    },
    
    buildEnd() {
      if (config.command === 'build') {
        // 生成构建报告
        const totalSize = Array.from(pluginContext.modules.values())
          .reduce((sum, mod) => sum + mod.size, 0);
        
        const buildTime = Date.now() - pluginContext.buildStartTime;
        
        console.log('\n📊 Build Report:');
        console.log('─'.repeat(50));
        console.log(`Total Modules: ${pluginContext.modules.size}`);
        console.log(`Total Size: ${(totalSize / 1024).toFixed(2)} KB`);
        console.log(`Build Time: ${buildTime}ms`);
        
        // 找出最大的模块
        const sortedModules = Array.from(pluginContext.modules.values())
          .sort((a, b) => b.size - a.size);
        
        console.log('\n🔍 Largest Modules:');
        sortedModules.slice(0, 5).forEach((mod, i) => {
          console.log(`  ${i + 1}. ${mod.path.split('/').pop()}: ${(mod.size / 1024).toFixed(2)} KB`);
        });
        
        // 生成 JSON 报告
        const report = {
          timestamp: new Date().toISOString(),
          totalModules: pluginContext.modules.size,
          totalSize,
          buildTime,
          modules: Array.from(pluginContext.modules.values()),
        };
        
        fs.writeFileSync(
          './build-report.json',
          JSON.stringify(report, null, 2)
        );
      }
    },
  };
}

// 插件 C：暴露 API 给其他插件
function apiProviderPlugin(): Plugin {
  const api = {
    getModules: () => pluginContext.modules,
    registerModule: (id: string, info: ModuleInfo) => {
      pluginContext.modules.set(id, info);
    },
    getConfig: () => config,
  };
  
  return {
    name: 'api-provider',
    
    // 将 API 暴露到全局
    apply: 'serve',
    
    configureServer(server) {
      // 在开发服务器上暴露 API
      server.config.logger.info('API Provider initialized');
      
      // 可以通过中间件访问 API
      server.middlewares.use('/__api', (req, res) => {
        if (req.url === '/modules') {
          res.setHeader('Content-Type', 'application/json');
          res.end(JSON.stringify(Array.from(api.getModules().values())));
        }
      });
    },
  };
}

export default defineConfig({
  plugins: [
    moduleCollectorPlugin(),
    moduleReporterPlugin(),
    apiProviderPlugin(),
  ],
});
```

### 3.5 环境变量与模式

#### 3.5.1 环境变量深度解析

Vite 的环境变量系统基于 `dotenv` 库，但提供了更丰富的功能：

```typescript
// 环境变量类型定义
// src/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_BASE_URL: string;
  readonly VITE_API_TIMEOUT: string;
  readonly VITE_UPLOAD_MAX_SIZE: string;
  readonly VITE_ENABLE_ANALYTICS: string;
  readonly VITE_MAPBOX_TOKEN: string;
  readonly VITE_GA_TRACKING_ID: string;
  readonly VITE_STRIPE_PUBLIC_KEY: string;
  readonly VITE_SENTRY_DSN: string;
  readonly VITE_BUILD_TIMESTAMP: string;
  readonly VITE_GIT_COMMIT_SHA: string;
  readonly VITE_APP_VERSION: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

declare module '*.vue' {
  import type { DefineComponent } from 'vue';
  const component: DefineComponent<object, object, unknown>;
  export default component;
}
```

**环境变量文件优先级**：
```bash
# 文件加载优先级（从高到低）
.env                          # 始终加载
.env.local                    # 始终加载，会被 git 忽略
.env.[mode]                   # 根据模式加载
.env.[mode].local            # 根据模式加载，会被 git 忽略

# 实际例子：
# 运行 VITE_ENV=staging npx vite build
# 加载顺序：
# 1. .env
# 2. .env.local
# 3. .env.staging
# 4. .env.staging.local
```

**环境变量使用示例**：
```typescript
// src/config/environment.ts
export const environment = {
  // 从环境变量读取配置
  production: import.meta.env.PROD,
  development: import.meta.env.DEV,
  
  // API 配置
  apiBaseUrl: import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000',
  apiTimeout: parseInt(import.meta.env.VITE_API_TIMEOUT || '30000', 10),
  
  // 功能开关
  features: {
    analytics: import.meta.env.VITE_ENABLE_ANALYTICS === 'true',
    darkMode: true,
    betaFeatures: import.meta.env.VITE_FLAG_BETA === 'true',
  },
  
  // 第三方服务
  services: {
    mapbox: {
      token: import.meta.env.VITE_MAPBOX_TOKEN,
    },
    googleAnalytics: {
      trackingId: import.meta.env.VITE_GA_TRACKING_ID,
    },
    stripe: {
      publicKey: import.meta.env.VITE_STRIPE_PUBLIC_KEY,
    },
    sentry: {
      dsn: import.meta.env.VITE_SENTRY_DSN,
    },
  },
  
  // 构建信息
  build: {
    timestamp: import.meta.env.VITE_BUILD_TIMESTAMP || new Date().toISOString(),
    gitCommit: import.meta.env.VITE_GIT_COMMIT_SHA || 'unknown',
    version: import.meta.env.VITE_APP_VERSION || '1.0.0',
  },
};

// 环境检测
export function isProduction(): boolean {
  return import.meta.env.PROD;
}

export function isDevelopment(): boolean {
  return import.meta.env.DEV;
}

export function isTest(): boolean {
  return import.meta.env.MODE === 'test';
}

export function getMode(): string {
  return import.meta.env.MODE;
}
```

#### 3.5.2 模式与环境组合

Vite 支持多种构建模式和环境的组合：

```bash
# .env 文件结构

# .env - 基础配置（所有模式共享）
VITE_APP_NAME=My Application
VITE_DEFAULT_LANGUAGE=zh-CN
VITE_TIMEZONE=Asia/Shanghai

# .env.development - 开发模式
VITE_API_BASE_URL=http://localhost:8000
VITE_DEBUG=true
VITE_LOG_LEVEL=debug

# .env.staging - 预发布环境
VITE_API_BASE_URL=https://staging-api.example.com
VITE_DEBUG=false
VITE_LOG_LEVEL=info

# .env.production - 生产环境
VITE_API_BASE_URL=https://api.example.com
VITE_DEBUG=false
VITE_LOG_LEVEL=warn

# .env.test - 测试环境
VITE_API_BASE_URL=http://localhost:8888
VITE_DEBUG=true
VITE_LOG_LEVEL=debug
VITE_MOCK_API=true
```

```typescript
// vite.config.ts - 多环境配置
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
  // 加载指定模式的环境变量
  const env = loadEnv(mode, process.cwd(), '');
  
  // 使用环境变量配置
  return {
    define: {
      // 全局变量定义
      __APP_VERSION__: JSON.stringify(env.VITE_APP_VERSION || '1.0.0'),
      __API_BASE_URL__: JSON.stringify(env.VITE_API_BASE_URL),
      __DEBUG__: JSON.stringify(env.VITE_DEBUG === 'true'),
    },
    
    build: {
      // 根据环境调整构建配置
      minify: mode === 'production' ? 'terser' : 'esbuild',
      sourcemap: mode !== 'production',
      
      // Rollup 选项
      rollupOptions: {
        output: {
          // 根据环境调整输出
          manualChunks: mode === 'production' ? {
            'vendor-react': ['react', 'react-dom'],
            'vendor-utils': ['lodash-es', 'axios'],
          } : undefined,
        },
      },
    },
    
    server: {
      port: parseInt(env.VITE_DEV_PORT || '3000', 10),
      proxy: mode === 'development' ? {
        '/api': {
          target: env.VITE_API_BASE_URL,
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ''),
        },
      } : undefined,
    },
    
    plugins: [
      // 条件插件
      mode === 'production' && compressionPlugin(),
      mode === 'development' && mockPlugin(),
    ].filter(Boolean),
  };
});
```

### 3.6 性能优化策略

#### 3.6.1 构建性能优化

```typescript
// vite.config.ts - 构建性能优化完整配置
import { defineConfig } from 'vite';
import viteCompression from 'vite-plugin-compression';
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  build: {
    // 目标环境 - 使用更高版本可以生成更小的代码
    target: 'esnext',
    
    // 输出目录
    outDir: 'dist',
    
    // 资源目录
    assetsDir: 'assets',
    
    // 静态资源内联阈值
    assetsInlineLimit: 4096, // 4kb
    
    // 启用 CSS 代码分割
    cssCodeSplit: true,
    
    // CSS Target
    cssTarget: 'chrome80',
    
    // 启用 Source Map（生产环境建议关闭）
    sourcemap: false,
    
    // 压缩器选择
    minify: 'esbuild',
    
    // Terser 配置（如果使用）
    terserOptions: {
      compress: {
        // 移除 console.log
        drop_console: true,
        // 移除 debugger
        drop_debugger: true,
        // 移除特定函数
        pure_funcs: ['console.log', 'console.info', 'console.debug'],
        // 压缩级别
        passes: 2,
        // 移除未使用的代码
        unused: true,
      },
      format: {
        // 移除注释
        comments: false,
      },
      mangle: {
        // Safari 10+ 兼容
        safari10: true,
      },
    },
    
    // Chunk 大小警告限制
    chunkSizeWarningLimit: 600,
    
    // 手动分包配置
    rollupOptions: {
      output: {
        // 入口文件命名
        entryFileNames: 'assets/js/[name]-[hash].js',
        
        // Chunk 命名
        chunkFileNames: 'assets/js/[name]-[hash].js',
        
        // 资源命名
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',
        
        // 手动分包策略
        manualChunks: (id) => {
          // Node_modules 分包
          if (id.includes('node_modules')) {
            // React 生态
            if (id.includes('react')) {
              return 'vendor-react';
            }
            
            // UI 组件库
            if (id.includes('@ant-design') || id.includes('antd')) {
              return 'vendor-antd';
            }
            if (id.includes('@mui') || id.includes('material-ui')) {
              return 'vendor-mui';
            }
            
            // 工具库
            if (id.includes('lodash')) {
              return 'vendor-lodash';
            }
            if (id.includes('axios')) {
              return 'vendor-axios';
            }
            
            // 大型库单独分包
            if (id.includes('echarts')) {
              return 'vendor-echarts';
            }
            if (id.includes('three')) {
              return 'vendor-three';
            }
            if (id.includes('pdfjs')) {
              return 'vendor-pdf';
            }
            
            // 其他 node_modules
            return 'vendor';
          }
          
          // 源代码分包策略
          if (id.includes('/views/')) {
            return 'views';
          }
          if (id.includes('/components/')) {
            return 'components';
          }
          if (id.includes('/composables/')) {
            return 'composables';
          }
          if (id.includes('/stores/')) {
            return 'stores';
          }
        },
        
        // 分隔符配置
        compact: true,
        
        // 是否内联动态导入
        inlineDynamicImports: false,
      },
      
      // 外部依赖
      external: [
        'electron',
        'fsevents',
      ],
      
      // 缓存
      cache: true,
    },
    
    // 报告压缩大小
    reportCompressedSize: true,
    
    // chunk 大小限制
    chunkSizeLimit: 1000000,
    
    // 清理输出目录
    emptyOutDir: true,
  },
  
  // 依赖预构建优化
  optimizeDeps: {
    // 强制预构建的依赖
    include: [
      'vue',
      'vue-router',
      'pinia',
      'axios',
      'dayjs',
      'lodash-es',
      'react',
      'react-dom',
      'react-router-dom',
    ],
    
    // 排除的依赖
    exclude: [],
    
    // esbuild 选项
    esbuildOptions: {
      target: 'esnext',
      keepNames: true,
    },
    
    // 缓存目录
    cacheDir: 'node_modules/.vite',
  },
  
  // 实验性功能
  experimental: {
    // 顶层 await 支持
    renderBuiltUrl(filename, type) {
      if (type === 'asset' && !filename.startsWith('http')) {
        // 自定义资源 URL 生成
        return { relative: true };
      }
      return { relative: true };
    },
  },
});
```

#### 3.6.2 开发服务器性能优化

```typescript
// vite.config.ts - 开发服务器优化
export default defineConfig({
  server: {
    // 端口
    port: 3000,
    
    // 主机
    host: '0.0.0.0',
    
    // 自动打开浏览器
    open: true,
    
    // HTTPS 配置
    https: false,
    // https: {
    //   key: './cert/localhost-key.pem',
    //   cert: './cert/localhost.pem',
    // },
    
    // 代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      },
    },
    
    // HMR 配置
    hmr: {
      // 禁用 HMR 覆盖层
      overlay: true,
      // HMR 连接端口
      port: 3001,
    },
    
    // CORS 配置
    cors: true,
    
    // 强制预构建
    force: false,
    
    // 监听选项
    watch: {
      ignored: ['**/node_modules/**', 'dist/**'],
      usePolling: false, // Windows 上可设为 true
      interval: 100,
    },
    
    // 静默模式
    silent: false,
    
    // 显示构建时间
    showBuildTime: true,
    
    // 显示文件变化
    showFileWatch: false,
  },
  
  // 预览服务器配置
  preview: {
    port: 4173,
    host: '0.0.0.0',
    strictPort: false,
    open: false,
    proxy: {},
  },
});
```

---

## 4. 常用命令与操作

### 4.1 命令行基础

#### 4.1.1 开发命令

```bash
# 启动开发服务器（默认端口 3000）
npm run dev

# 指定端口
npm run dev -- --port 5173

# 指定主机（允许外部访问）
npm run dev -- --host

# 指定配置文件
npm run dev -- --config vite.config.ts

# 打开浏览器
npm run dev -- --open

# 清除缓存后启动
npm run dev -- --force

# 自定义缓存目录
npm run dev -- --cachingDir .vite-cache
```

#### 4.1.2 构建命令

```bash
# 生产构建
npm run build

# 构建并预览
npm run build && npm run preview

# 构建但不压缩
npm run build -- --mode development

# 构建并分析
ANALYZE=true npm run build

# 自定义输出目录
npm run build -- --outDir dist-custom

# 生成 sourcemap
npm run build -- --sourcemap
```

#### 4.1.3 预览命令

```bash
# 预览生产构建
npm run preview

# 指定端口
npm run preview -- --port 4173

# 预览并开启 sourcemap
npm run preview -- --sourcemap
```

### 4.2 脚本配置

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "build:dev": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "preview": "vite preview",
    "preview:prod": "vite preview --outDir dist",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs,.ts,.tsx --fix",
    "typecheck": "vue-tsc --noEmit",
    "typecheck:watch": "vue-tsc --noEmit --watch",
    "format": "prettier --write src/**/*.{vue,ts,tsx,js,jsx,json,css,scss}",
    "clean": "rm -rf dist node_modules/.vite"
  }
}
```

### 4.3 环境变量操作

```bash
# 使用特定模式
npm run build -- --mode staging

# 使用 .env.staging 文件
# VITE_API_URL=https://staging.api.example.com

# 查看加载的环境变量
DEBUG=vite:config npm run dev
```

---

## 5. 高级配置与技巧

### 5.1 性能优化配置

#### 5.1.1 构建优化

```typescript
// vite.config.ts - 构建优化
export default defineConfig({
  build: {
    // 目标浏览器
    target: 'es2015',

    // 输出目录
    outDir: 'dist',

    // 资源目录
    assetsDir: 'assets',

    // 启用 sourcemap
    sourcemap: false,

    // 压缩器
    minify: 'esbuild',

    // Chunk 大小警告限制
    chunkSizeWarningLimit: 600,

    // CSS 代码分割
    cssCodeSplit: true,

    // 压缩选项
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
        pure_funcs: ['console.log', 'console.info'],
        passes: 2,
      },
      format: {
        comments: false,
      },
      mangle: {
        safari10: true,
      },
    },

    // Rollup 选项
    rollupOptions: {
      output: {
        // 手动分包
        manualChunks: (id) => {
          if (id.includes('node_modules')) {
            if (id.includes('react')) return 'react-vendor';
            if (id.includes('antd')) return 'antd-vendor';
            return 'vendor';
          }
        },

        // 分隔符
        chunkFileNames: 'assets/[name]-[hash].js',
        entryFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',

        // 横幅
        banner: {
          js: '/* My App v1.0.0 */',
        },
      },

      // 外部依赖
      external: ['electron'],
    },

    // 报告压缩大小
    reportCompressedSize: true,

    // 块最大大小
    chunkSizeLimit: 1000000,
  },
});
```

#### 5.1.2 依赖优化

```typescript
// 依赖预构建优化
export default defineConfig({
  optimizeDeps: {
    // 强制预构建的依赖
    include: [
      'vue',
      'vue-router',
      'pinia',
      'axios',
      'dayjs',
    ],

    // 排除的依赖
    exclude: [
      '@vite/client',
    ],

    // 构建选项
    esbuildOptions: {
      target: 'esnext',
      treeShaking: true,
      keepNames: true,
      logOverride: { 'this-is-undefined-in-esm': 'silent' },
    },

    // 缓存目录
    cacheDir: 'node_modules/.vite',

    // 入口点
    entries: ['index.html'],
  },
});
```

### 5.2 开发服务器高级配置

#### 5.2.1 代理配置

```typescript
export default defineConfig({
  server: {
    // 端口
    port: 3000,

    // 主机
    host: '0.0.0.0',

    // HTTPS
    https: {
      key: './certificates/localhost-key.pem',
      cert: './certificates/localhost.pem',
    },

    // 代理配置
    proxy: {
      // 简单代理
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },

      // 带重写的代理
      '/v2/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/v2\/api/, '/api/v2'),
      },

      // WebSocket 代理
      '/ws': {
        target: 'ws://localhost:8080',
        ws: true,
      },

      // 带配置的代理
      '/custom-api': {
        target: 'http://localhost:9000',
        changeOrigin: true,
        configure: (proxy, options) => {
          proxy.on('proxyReq', (proxyReq, req, res) => {
            // 添加自定义头
            proxyReq.setHeader('X-Custom-Header', 'value');
          });
        },
      },
    },

    // HMR 配置
    hmr: {
      overlay: true,
      port: 3001,
      server: undefined,
    },

    // 静态文件服务
    static: {
      directory: path.join(__dirname, 'public'),
      prefix: '/static',
    },

    // CORS 配置
    cors: true,

    // 强制开发模式
    force: true,

    // 监视选项
    watch: {
      ignored: ['**/node_modules/**', 'dist/**'],
      usePolling: false,
      interval: 100,
    },

    // 中间件
    middlewareMode: false,

    // 响应头
    headers: {
      'X-Custom-Header': 'value',
    },
  },
});
```

#### 5.2.2 中间件配置

```typescript
// vite.config.ts - 自定义中间件
import { defineConfig } from 'vite';
import express from 'express';

export default defineConfig({
  server: {
    middlewareMode: false,

    configureServer(server) {
      // 添加 Express 中间件
      const app = server.middlewares;
      const router = express.Router();

      router.get('/api/config', (req, res) => {
        res.json({
          apiUrl: process.env.API_URL,
          version: '1.0.0',
        });
      });

      app.use('/custom', router);
    },

    configurePreviewServer(server) {
      // 预览服务器中间件
      const app = server.middlewares;

      app.use((req, res, next) => {
        console.log(`Preview: ${req.method} ${req.url}`);
        next();
      });
    },
  },
});
```

### 5.3 高级插件开发

#### 5.3.1 自定义插件示例

```typescript
// vite.config.ts - 自定义插件
import { defineConfig, Plugin } from 'vite';
import fs from 'fs';
import path from 'path';

// 自动导入版本号插件
function versionPlugin(): Plugin {
  const virtualModuleId = 'virtual:version';
  const resolvedVirtualModuleId = '\0' + virtualModuleId;

  return {
    name: 'version-plugin',

    resolveId(id) {
      if (id === virtualModuleId) {
        return resolvedVirtualModuleId;
      }
    },

    load(id) {
      if (id === resolvedVirtualModuleId) {
        const packageJson = fs.readFileSync(
          path.resolve(process.cwd(), 'package.json'),
          'utf-8'
        );
        const { version } = JSON.parse(packageJson);
        const buildTime = new Date().toISOString();

        return `export const version = '${version}'; export const buildTime = '${buildTime}';`;
      }
    },
  };
}

// 构建信息插件
function buildInfoPlugin(): Plugin {
  return {
    name: 'build-info-plugin',
    buildStart() {
      console.log('Build started at:', new Date().toISOString());
    },
    buildEnd() {
      console.log('Build ended at:', new Date().toISOString());
    },
    closeBundle() {
      console.log('Bundle closed');
    },
  };
}

// CSS 变量注入插件
function cssVarsPlugin(): Plugin {
  return {
    name: 'css-vars-plugin',
    enforce: 'pre',
    transform(code, id) {
      if (id.endsWith('.css')) {
        // 注入 CSS 变量
        const vars = `
          :root {
            --primary-color: #1890ff;
            --secondary-color: #52c41a;
            --text-color: #333;
            --bg-color: #fff;
          }
        `;
        return vars + code;
      }
    },
  };
}

export default defineConfig({
  plugins: [
    versionPlugin(),
    buildInfoPlugin(),
    cssVarsPlugin(),
  ],
});
```

#### 5.3.2 虚拟模块插件

```typescript
// 虚拟模块示例
function virtualModulesPlugin(): Plugin {
  const virtualModules = {
    'virtual:config': `
      export const config = {
        apiUrl: import.meta.env.VITE_API_URL,
        enableDebug: import.meta.env.VITE_ENABLE_DEBUG === 'true',
        features: {
          darkMode: true,
          analytics: false,
        },
      };
    `,
    'virtual:constants': `
      export const APP_NAME = 'My App';
      export const APP_VERSION = '1.0.0';
      export const SUPPORT_EMAIL = 'support@example.com';
    `,
  };

  return {
    name: 'virtual-modules',
    resolveId(id) {
      if (id in virtualModules) {
        return id;
      }
    },
    load(id) {
      if (id in virtualModules) {
        return virtualModules[id];
      }
    },
  };
}
```

### 5.4 特殊场景配置

#### 5.4.1 微前端配置

```typescript
// vite.config.ts - 微前端配置
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        format: 'system',
        systemDontBundle: true,
      },
    },
  },
});
```

#### 5.4.2 Electron 配置

```typescript
// vite.config.ts - Electron 配置
export default defineConfig({
  base: './',
  build: {
    outDir: 'dist-electron',
    rollupOptions: {
      external: ['electron'],
      output: {
        format: 'cjs',
      },
    },
  },
});
```

#### 5.4.3 库模式配置

```typescript
// vite.config.ts - 库模式配置
export default defineConfig({
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.ts'),
      name: 'MyLib',
      formats: ['es', 'umd', 'iife'],
      fileName: (format) => `my-lib.${format}.js`,
    },
    rollupOptions: {
      external: ['vue', 'react'],
      output: {
        globals: {
          vue: 'Vue',
          react: 'React',
        },
        exports: 'named',
      },
    },
  },
});
```

---

## 6. 与同类技术对比

### 6.1 Vite vs Webpack

#### 6.1.1 核心差异

| 维度 | Vite | Webpack |
|------|------|---------|
| **架构** | 开发基于 ESM，生产基于 Rollup | 全流程基于自身打包引擎 |
| **启动速度** | <500ms（无需打包） | 10-60s（全量打包） |
| **热更新** | 精确更新单个模块 | 重新打包依赖树 |
| **配置复杂度** | 低 | 高 |
| **插件生态** | 丰富（兼容 Rollup 插件） | 极其丰富 |
| **学习曲线** | 平缓 | 陡峭 |
| **调试体验** | 接近原生 | 需要配置 source-map |
| **兼容性** | 现代浏览器优先 | 可配置旧版浏览器 |
| **适用场景** | 新项目、小中大型项目 | 复杂配置需求、超大型项目 |

#### 6.1.2 迁移指南

从 Webpack 迁移到 Vite 的关键步骤：

```typescript
// Webpack 配置 → Vite 配置对照

// Webpack
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js',
    publicPath: '/',
  },
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'babel-loader',
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
      },
    ],
  },
  plugins: [
    new HtmlWebpackPlugin({ template: './public/index.html' }),
  ],
};

// Vite (vite.config.ts)
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    rollupOptions: {
      input: './src/index.html',
    },
  },
  plugins: [vue()],
});
```

### 6.2 Vite vs esbuild

| 维度 | Vite | esbuild |
|------|------|---------|
| **定位** | 完整构建工具 | 高性能打包库 |
| **开发服务器** | 内置 | 需要手动实现 |
| **插件系统** | 完善 | 有限 |
| **Tree Shaking** | 优秀 | 优秀 |
| **代码分割** | 支持 | 有限支持 |
| **生态系统** | 成熟 | 新兴 |
| **使用场景** | 项目构建 | 工具开发、脚本 |

### 6.3 Vite vs Parcel

| 维度 | Vite | Parcel |
|------|------|--------|
| **配置需求** | 少量配置 | 零配置 |
| **插件系统** | Rollup 风格 | 类 Webpack 风格 |
| **构建速度** | 快 | 快 |
| **生态成熟度** | 成熟 | 一般 |
| **自定义程度** | 高 | 低 |

### 6.4 选型建议

```
Vite 选型决策树:

项目类型是库/框架吗？
│
├─ 是 → 需要完全自定义构建流程吗？
│        ├─ 是 → 使用 esbuild 或自定义 Rollup
│        └─ 否 → Vite (lib 模式)
│
└─ 否 → 项目规模？
         │
         ├─ 小型（<100 模块）→ Vite
         │
         ├─ 中型（100-1000 模块）→ Vite
         │
         └─ 大型（>1000 模块）→ 评估复杂需求
                  │
                  ├─ 需要大量自定义配置 → Webpack 5
                  ├─ 需要微前端 → Vite + Module Federation
                  └─ 无特殊需求 → Vite
```

---

## 7. 常见问题与解决方案

### 7.1 开发环境问题

#### 7.1.1 端口被占用

```bash
# 方法 1：使用不同端口
npm run dev -- --port 5174

# 方法 2：查找并终止占用进程
lsof -i :3000
kill -9 <PID>
```

#### 7.1.2 缓存问题

```bash
# 清除缓存
rm -rf node_modules/.vite

# 强制重新构建
npm run dev -- --force
```

#### 7.1.3 Node 版本不兼容

```bash
# 检查 Node 版本
node --version

# 建议使用 Node 18+ 或 20+
# 可使用 nvm 切换版本
nvm use 20
```

### 7.2 构建问题

#### 7.2.1 Chunk 大小超限

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    chunkSizeWarningLimit: 1000, // 增加限制
    rollupOptions: {
      output: {
        manualChunks: {
          // 分包处理大模块
          'vendor-lodash': ['lodash'],
          'vendor-charts': ['echarts', 'recharts'],
        },
      },
    },
  },
});
```

#### 7.2.2 构建失败

```bash
# 完整重建
rm -rf node_modules dist node_modules/.vite
npm install
npm run build
```

#### 7.2.3 TypeScript 错误

```bash
# 同步 TypeScript 配置
npx vue-tsc --noEmit

# 检查 tsconfig
cat tsconfig.json | grep -A 20 "compilerOptions"
```

### 7.3 性能问题

#### 7.3.1 依赖预构建问题

```typescript
// vite.config.ts - 强制预构建
export default defineConfig({
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'large-library',
    ],
    force: true, // 强制重新预构建
  },
});
```

#### 7.3.2 大项目优化

```typescript
// vite.config.ts - 大项目优化
export default defineConfig({
  server: {
    // 使用较新浏览器
    hmr: {
      overlay: false, // 禁用错误遮罩
    },
    watch: {
      usePolling: false,
    },
  },
  build: {
    target: 'es2015',
    minify: 'terser',
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom'],
          'vendor-utils': ['lodash', 'axios', 'dayjs'],
        },
      },
    },
  },
});
```

### 7.4 插件兼容问题

#### 7.4.1 插件顺序问题

```typescript
// vite.config.ts
export default defineConfig({
  plugins: [
    // enforce: 'pre' 在前面
    vue(),
    // enforce: 'post' 在后面
  ],
});
```

#### 7.4.2 插件不生效

```typescript
// 检查插件配置
// 确保插件正确安装
npm list <plugin-name>

// 确保插件在 plugins 数组中
export default defineConfig({
  plugins: [
    vue(),
    // 添加缺失的插件
    VitePlugins.autoImport(),
  ],
});
```

---

## 8. 实战项目示例

### 8.1 Vue 3 + Vite 企业级项目

#### 8.1.1 项目结构

```
enterprise-vue/
├── src/
│   ├── App.vue
│   ├── main.ts
│   ├── api/
│   │   ├── index.ts
│   │   ├── user.ts
│   │   └── product.ts
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button.vue
│   │   │   ├── Input.vue
│   │   │   └── Modal.vue
│   │   └── business/
│   │       ├── ProductCard.vue
│   │       └── UserAvatar.vue
│   ├── composables/
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   └── usePagination.ts
│   ├── layouts/
│   │   ├── Default.vue
│   │   └── Blank.vue
│   ├── pages/
│   │   ├── Home.vue
│   │   ├── About.vue
│   │   └── Login.vue
│   ├── router/
│   │   └── index.ts
│   ├── stores/
│   │   ├── user.ts
│   │   └── app.ts
│   ├── styles/
│   │   ├── variables.scss
│   │   ├── mixins.scss
│   │   └── global.scss
│   ├── types/
│   │   └── index.ts
│   └── utils/
│       ├── request.ts
│       └── storage.ts
├── public/
│   ├── favicon.ico
│   └── images/
├── index.html
├── package.json
├── tsconfig.json
├── vite.config.ts
└── .env
```

#### 8.1.2 完整配置示例

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import path from 'path';
import { viteStaticCopy } from 'vite-plugin-static-copy';

export default defineConfig({
  base: '/',
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@composables': path.resolve(__dirname, './src/composables'),
      '@layouts': path.resolve(__dirname, './src/layouts'),
      '@pages': path.resolve(__dirname, './src/pages'),
      '@router': path.resolve(__dirname, './src/router'),
      '@stores': path.resolve(__dirname, './src/stores'),
      '@styles': path.resolve(__dirname, './src/styles'),
      '@types': path.resolve(__dirname, './src/types'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@api': path.resolve(__dirname, './src/api'),
    },
    extensions: ['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json'],
  },
  plugins: [
    vue(),
    vueJsx(),
    viteStaticCopy({
      targets: [
        {
          src: 'public/*',
          dest: '',
        },
      ],
    }),
  ],
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss"; @import "@/styles/mixins.scss";`,
      },
    },
  },
  build: {
    target: 'es2015',
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'esbuild',
    chunkSizeWarningLimit: 500,
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-vue': ['vue', 'vue-router', 'pinia'],
          'vendor-utils': ['axios', 'dayjs', 'lodash-es'],
        },
        chunkFileNames: 'assets/[name]-[hash].js',
        entryFileNames: 'assets/[name]-[hash].js',
        assetFileNames: 'assets/[name]-[hash].[ext]',
      },
    },
  },
  server: {
    port: 3000,
    host: '0.0.0.0',
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
  optimizeDeps: {
    include: ['vue', 'vue-router', 'pinia', 'axios', 'dayjs'],
  },
});
```

#### 8.1.3 主入口文件

```typescript
// src/main.ts
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import router from './router';
import App from './App.vue';

import './styles/global.scss';

const app = createApp(App);
const pinia = createPinia();

app.use(pinia);
app.use(router);

app.mount('#app');
```

#### 8.1.4 路由配置

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router';
import type { RouteRecordRaw } from 'vue-router';

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/layouts/Default.vue'),
    children: [
      {
        path: '',
        name: 'Home',
        component: () => import('@/pages/Home.vue'),
      },
      {
        path: 'about',
        name: 'About',
        component: () => import('@/pages/About.vue'),
      },
    ],
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/pages/Login.vue'),
  },
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
});

router.beforeEach((to, from, next) => {
  const token = localStorage.getItem('token');
  if (to.meta.requiresAuth && !token) {
    next('/login');
  } else {
    next();
  }
});

export default router;
```

### 8.2 React + Vite 项目

```typescript
// vite.config.ts - React 项目
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [
    react({
      jsxImportSource: 'react',
      babel: {
        plugins: [
          ['@babel/plugin-proposal-decorators', { legacy: true }],
        ],
      },
      fastRefresh: true,
    }),
  ],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: false,
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          'vendor-antd': ['antd', '@ant-design/icons'],
        },
      },
    },
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
      },
    },
  },
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom', 'antd'],
  },
});
```

---

> [!TIP]
> **Vite 最佳实践**：始终使用 TypeScript 配置文件；生产构建使用 manualChunks 进行细粒度分包；结合 unplugin-auto-import 和 unplugin-vue-components 减少 import 语句；使用 vite-plugin-pwa 提供离线能力。对于大型项目，考虑使用 Vite 的依赖预构建和分包策略优化构建性能。
