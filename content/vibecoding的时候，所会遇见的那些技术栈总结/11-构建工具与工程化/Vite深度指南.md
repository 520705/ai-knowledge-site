# Vite 深度完全指南

date: 2026-04-24

> [!NOTE]
> 本文档是 Vite 6.x 的完整参考，涵盖配置详解、插件生态、开发服务器原理、热模块替换、生产构建策略及企业级项目实践。Vite 以其极快的冷启动和毫秒级热更新速度，正在成为现代前端开发的首选构建工具。

---

## 技术概述与定位

### Vite 的诞生背景与核心价值

在前端开发工具的发展历程中，Webpack 长期占据着统治地位。然而，随着项目规模的增长，Webpack 的一些固有缺陷逐渐暴露出来。冷启动时间长是最大的痛点，一个包含数千个模块的中型项目，Webpack 的启动时间可能达到 30 秒甚至更久。热更新效率低下同样是困扰开发者的问题，每次代码修改后，需要等待数秒才能看到变更效果。这些问题严重影响了开发体验和团队效率。

Vite 的出现正是为了解决这些痛点。Vite 由 Evan You（Vue.js 的创建者）于 2020 年创建，它采用了一种全新的开发服务器架构，利用浏览器原生 ES Modules 的能力来实现极速的开发体验。Vite 不需要打包开发阶段的代码，而是直接在浏览器中运行原生 ES 模块，通过请求按需编译和服务。这种设计从根本上解决了冷启动和热更新的性能问题。

Vite 的设计哲学可以概括为「极速开发，优化生产」。在开发阶段，Vite 利用浏览器原生 ES Modules 的能力，实现了真正的即时启动和毫秒级热更新。在生产阶段，Vite 使用 Rollup 进行打包优化，生成高度优化的生产 bundle。这种分而治之的策略是 Vite 区别于其他工具的关键。传统打包工具在开发阶段也要进行完整的打包工作，而 Vite 跳过了这一步骤，只在浏览器需要时才进行模块的编译和转换。

### Vite vs 其他构建工具

当我们将 Vite 与其他主流构建工具进行对比时，可以更清楚地看到它的优势与适用场景。Webpack 作为最成熟的构建工具，拥有最丰富的插件生态和最广泛的社区支持，但在开发体验上确实存在明显短板。esbuild 是一个用 Go 语言编写的高性能打包工具，它的构建速度极快，但作为通用构建工具的生态还不够丰富。Parcel 以零配置著称，适合快速原型开发，但在细粒度控制方面不如 Vite。

| 特性 | Vite | Webpack | esbuild | Parcel |
|------|------|---------|---------|--------|
| **冷启动速度** | <500ms | 10-60s | <100ms | 1-5s |
| **热更新速度** | <50ms | 2-10s | <50ms | <1s |
| **配置复杂度** | 低 | 高 | 无需配置 | 极低 |
| **插件生态** | 丰富（兼容 Rollup） | 极其丰富 | 有限 | 一般 |
| **Tree Shaking** | 优秀 | 良好 | 优秀 | 良好 |
| **生产打包** | Rollup | webpack | 可选 | Parcel |
| **TypeScript 支持** | 原生 | 需要配置 | 原生 | 原生 |

### Vite 的适用场景

Vite 特别适合以下场景：新项目启动时，Vite 的极速启动特性使其成为理想选择；前端框架项目中，官方支持 Vue、React、Svelte、Preact、Lit 等多种框架；组件库开发时，开发体验和打包优化都非常适合；TypeScript 项目中原生支持，无需额外配置；快速原型开发时低配置特性让开发者可以快速验证想法；静态网站生成场景下可以配合 SSG 插件使用。对于超大型项目（模块数超过 5000），如果需要完全自定义的构建流程，Webpack 5 仍然是更稳妥的选择。

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
         ├─ 中型（100-1000 模块）→ Vite
         └─ 大型（>1000 模块）→ 评估复杂需求
                  │
                  ├─ 需要大量自定义配置 → Webpack 5
                  ├─ 需要微前端 → Vite + Module Federation
                  └─ 无特殊需求 → Vite
```

---

## 项目初始化与基础配置

### 项目脚手架

Vite 提供了官方的项目模板，可以通过 npm、yarn 或 pnpm 快速创建项目。这种脚手架方式特别适合初次接触 Vite 的开发者，或者需要快速启动新项目的场景。官方模板经过精心设计，包含了最佳实践的配置，是学习的良好起点。

```bash
# npm 创建模板项目
npm create vite@latest my-vue-project -- --template vue
npm create vite@latest my-react-project -- --template react
npm create vite@latest my-vue-ts-project -- --template vue-ts
npm create vite@latest my-react-ts-project -- --template react-ts
npm create vite@latest my-svelte-project -- --template svelte
npm create vite@latest my-svelte-ts-project -- --template svelte-ts

# 使用 create-vite（更灵活）
npm create vite@latest my-app
# 选择框架和变体
```

对于已有项目，可以手动安装 Vite 和相关依赖。这种方式给予开发者更多的控制权，适合从现有项目迁移的场景。

```bash
# 安装 Vite 和 TypeScript
npm install vite typescript -D

# 安装框架插件
npm install @vitejs/plugin-vue -D     # Vue
npm install @vitejs/plugin-react -D   # React
npm install @vitejs/plugin-vue-jsx -D # Vue JSX

# 安装 Node 类型定义
npm install @types/node -D
```

### vite.config.ts 完整配置

Vite 的配置文件支持多种格式，推荐使用 TypeScript 格式以获得完整的类型提示和更好的开发体验。一个完善的 Vite 配置文件通常包含服务器配置、构建配置、插件配置、路径别名等多个方面。

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import react from '@vitejs/plugin-react';
import path from 'path';
import { viteStaticCopy } from 'vite-plugin-static-copy';

export default defineConfig({
  // 基础配置
  base: '/',                    // 部署基础路径
  mode: 'development',
  publicDir: 'public',          // 静态资源目录

  // 插件配置
  plugins: [
    vue({
      include: [/\.vue$/],
      template: {
        compilerOptions: {
          isCustomElement: (tag) => tag.startsWith('ion-'),
        },
      },
    }),
    react({
      include: '**/*.{jsx,tsx}',
    }),
  ],

  // 路径别名 - 这是非常重要的配置
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@assets': path.resolve(__dirname, './src/assets'),
      '@views': path.resolve(__dirname, './src/views'),
      '@hooks': path.resolve(__dirname, './src/hooks'),
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
        api: 'modern-compiler',
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
        // 分包策略
        manualChunks: {
          'vendor-react': ['react', 'react-dom', 'react-router-dom'],
          'vendor-utils': ['lodash-es', 'axios', 'dayjs'],
        },
      },
    },
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
    reportCompressedSize: true,
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
    },
  },

  // 依赖预构建
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
    exclude: [],
    esbuildOptions: {
      target: 'esnext',
    },
  },
});
```

### 环境变量配置

Vite 的环境变量系统设计得非常清晰，它通过不同的 `.env` 文件来管理不同环境的配置。这种设计让开发者可以轻松地在开发、预发布、生产等不同环境间切换，同时保持配置的安全性和便捷性。

```bash
# .env                  # 所有环境都会加载
# .env.local            # 所有环境，会被 git 忽略
# .env.[mode]           # 特定模式，如 .env.development
# .env.[mode].local     # 特定模式，会被 git 忽略

# 示例文件内容
VITE_APP_TITLE=我的应用
VITE_API_BASE_URL=https://api.example.com
VITE_UPLOAD_MAX_SIZE=5242880
```

> [!IMPORTANT]
> Vite 中只有以 `VITE_` 开头的变量才会暴露给客户端代码。以其他前缀或下划线开头的变量不会被包含在客户端代码中，这是出于安全考虑的设计。

```typescript
// src/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string;
  readonly VITE_API_BASE_URL: string;
  readonly VITE_UPLOAD_MAX_SIZE: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}

// 使用环境变量
console.log(import.meta.env.VITE_APP_TITLE);
```

### TypeScript 集成

Vite 对 TypeScript 的支持是开箱即用的，但为了获得最佳的开发和类型检查体验，建议配置 `tsconfig` 和类型定义文件。Vite 使用 esbuild 进行 TypeScript 的转译，这意味着它只处理语法转换，不进行类型检查。类型检查应该由独立的 `tsc --noEmit` 命令或 IDE 来完成。

```json
// tsconfig.json
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
    "jsx": "preserve",
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

## 开发服务器原理

### 原生 ES Modules 架构

Vite 开发服务器的核心是利用浏览器原生 ES Modules 的能力。当浏览器请求一个模块时，Vite 服务器拦截请求，按需编译模块并返回给浏览器。这种架构的革命性在于它完全抛弃了传统的打包思维。

```
原生 ES Modules 请求流程:

┌─────────────────────────────────────────────────────────────────┐
│                        Browser                                   │
│                                                                  │
│   1. 请求 main.ts                                                 │
│   └──────────────→ ┌──────────────┐                              │
│                   │  Vite Dev    │                              │
│                   │  Server      │                              │
│   2. 返回 ESM     │  localhost   │                              │
│   ←────────────── └──────────────┘                              │
│                                                                  │
│   3. 浏览器解析依赖，按需请求模块A                                  │
│   └──────────────→ ┌──────────────┐                              │
│   4. 返回模块A     │  按需编译    │                              │
│   ←────────────── └──────────────┘                              │
│                                                                  │
│   ... 重复直到所有依赖加载完成                                      │
└─────────────────────────────────────────────────────────────────┘
```

这种架构的优势在于：不需要打包，启动即服务——Vite 服务器可以在毫秒级启动，因为根本不需要打包任何东西；只编译当前需要的模块——即使项目包含数千个模块，初始只需要编译入口文件和立即需要的模块；模块缓存高效——重启后浏览器仍然可以利用缓存，只请求变更的模块。

### 依赖预构建

Vite 在首次启动时会进行依赖预构建（Dependency Pre-bundling）。这个过程使用 esbuild 将 CommonJS 格式的依赖转换为 ESM 格式，并合并小文件以减少请求数量。预构建是 Vite 性能优化的重要一环，它解决了以下几个关键问题。

第一个问题是 CommonJS 依赖的转换。许多 npm 包仍然使用 CommonJS 格式发布，而浏览器原生 ES Modules 不支持 CommonJS。预构建会将这些依赖转换为 ESM 格式。第二个问题是请求数量优化。许多 npm 包包含分散的小文件，如果每个文件单独请求会产生大量网络请求。预构建会将这些相关模块合并，减少请求数量。第三个问题是依赖解析优化。预构建会处理依赖之间的循环引用和复杂的导入关系，加快模块解析速度。

```typescript
// 预构建配置
export default defineConfig({
  optimizeDeps: {
    // 强制预构建的依赖
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'lodash',
      'lodash-es',
      'axios',
      'dayjs',
    ],
    // 排除的依赖（不预构建）
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

### 热模块替换（HMR）深度解析

Vite 的 HMR 系统非常高效，只更新实际修改的模块，无需重新加载整个页面。HMR 的工作原理涉及几个关键步骤：文件变更检测、模块图更新、边界确定和边界替换。

当文件发生变更时，Vite 的监听系统会检测到变更并确定需要更新的模块。然后，它会分析模块依赖图，确定所有受影响的边界。最后，只替换这些边界的模块，而不是重新加载整个应用。这种精细化的更新策略是 Vite 热更新如此快速的原因。

```typescript
// HMR API 使用示例
if (import.meta.hot) {
  // 接受模块更新
  import.meta.hot.accept(() => {
    console.log('模块已更新');
  });

  // 接受自身更新并获取新模块
  import.meta.hot.accept((newModule) => {
    console.log('新模块内容:', newModule);
  });

  // 监听更新事件
  import.meta.hot.on('vite:beforeUpdate', (data) => {
    console.log('即将更新:', data);
  });

  import.meta.hot.on('vite:afterUpdate', (data) => {
    console.log('更新完成:', data);
  });

  // 清理工作（模块卸载时）
  import.meta.hot.dispose((data) => {
    data.cachedData = moduleCache;
  });

  // 标记模块不可热更新
  import.meta.hot.decline();
}
```

框架插件（如 `@vitejs/plugin-vue`）已经内置了 HMR 支持，你不需要手动处理大多数场景。Vue 组件修改会自动触发热更新，Svelte 组件同样如此。

---

## Rollup 生产构建

### 构建流程

Vite 的生产构建使用 Rollup。Rollup 是专为 JavaScript 库设计的打包工具，以高效的 Tree Shaking 能力著称。在生产环境中，Vite 会使用 Rollup 进行完整的打包，生成高度优化的静态资源。

```
Rollup 构建流程:

Source Files → Module Graph → Tree Shaking → Bundle Generation → Output Files

┌─────────────────────────────────────────────────────────────────┐
│                    Rollup 构建流程                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Source Files                                                    │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐            │
│   │ main.js │  │ util.js │  │ lib.js │  │app.js  │            │
│   └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘            │
│        │            │            │            │                     │
│        └────────────┴─────┬──────┴────────────┘                   │
│                           │                                        │
│                           ▼                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Tree Shaking：移除未使用的代码                          │   │
│   │  - 分析 import/export                                   │   │
│   │  - 标记并移除死代码                                      │   │
│   │  - 保留实际使用的导出                                     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                        │
│                           ▼                                        │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Code Splitting：按需分割代码                            │   │
│   │  - 动态 import() 分割                                    │   │
│   │  - 手动分包配置                                          │   │
│   └─────────────────────────────────────────────────────────┘   │
│                           │                                        │
│                           ▼                                        │
│   Output Files                                                │
│   ┌─────────┐  ┌─────────┐  ┌─────────┐                         │
│   │ main.js │  │vendors.js│ │style.css│                        │
│   └─────────┘  └─────────┘  └─────────┘                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 代码分割策略

合理的代码分割可以优化首屏加载性能，让用户只下载当前页面需要的代码。代码分割的策略通常包括 vendor 分割、动态导入和路由级分割。

Vendor 分割是将第三方库分离到单独的 chunk 中。这样做的好处是第三方库的缓存可以更持久，因为它们的变更频率远低于业务代码。动态导入是使用 JavaScript 的动态 import 语法来实现按需加载，这是实现代码分割最常用的方式。路由级分割是在 SPA 应用中，每个路由对应的页面单独打包，用户访问时才加载对应代码。

```typescript
// 代码分割配置
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // 手动分包
        manualChunks: (id) => {
          // React 生态系统
          if (id.includes('node_modules/react')) return 'react-vendor';
          if (id.includes('node_modules/react-dom')) return 'react-vendor';
          if (id.includes('node_modules/react-router')) return 'react-vendor';

          // UI 组件库
          if (id.includes('node_modules/antd')) return 'antd-vendor';
          if (id.includes('node_modules/@mui')) return 'mui-vendor';

          // 工具库
          if (id.includes('node_modules/lodash')) return 'lodash-vendor';
          if (id.includes('node_modules/axios')) return 'axios-vendor';

          // 图表库（通常较大，单独分包）
          if (id.includes('node_modules/echarts')) return 'echarts-vendor';

          // 源代码按目录分割
          if (id.includes('/views/')) return 'views';
          if (id.includes('/components/')) return 'components';
        },

        // 文件命名模板
        chunkFileNames: 'assets/js/[name]-[hash].js',
        entryFileNames: 'assets/js/[name]-[hash].js',
        assetFileNames: 'assets/[ext]/[name]-[hash].[ext]',

        // 格式化选项
        compact: true,
        hoistTransitiveImports: false,
      },
    },
  },
});
```

### 生产构建优化

生产构建的优化是一个持续的过程，需要根据实际项目情况和性能测试结果进行调整。以下是一些常见的优化策略。

目标浏览器选择是一个重要的优化点。更现代的目标意味着更小的输出代码，因为转译需要处理的兼容性代码更少。压缩器选择方面，esbuild 速度更快但压缩率略低，terser 速度较慢但压缩率更高。代码移除策略可以移除调试代码和不必要的注释，进一步减小包体积。

```typescript
// 生产构建优化配置
export default defineConfig({
  build: {
    // 目标浏览器（更现代的目标 = 更小的输出）
    target: 'esnext',

    // 输出目录
    outDir: 'dist',

    // 静态资源目录
    assetsDir: 'assets',

    // 启用 sourcemap（生产环境可选关闭）
    sourcemap: false,

    // 压缩器
    minify: 'esbuild',  // 或 'terser'（更小但更慢）

    // CSS 代码分割
    cssCodeSplit: true,

    // Terser 选项
    terserOptions: {
      compress: {
        drop_console: true,    // 移除 console.log
        drop_debugger: true,    // 移除 debugger
        pure_funcs: ['console.log', 'console.info'],
        passes: 2,              // 多轮压缩
      },
      mangle: {
        safari10: true,         // Safari 10+ 兼容
      },
    },

    // 报告压缩大小
    reportCompressedSize: true,

    // Chunk 大小警告
    chunkSizeWarningLimit: 600,
  },
});
```

---

## 插件系统

### 官方插件

Vite 官方维护了一组插件，覆盖了常用框架和功能。这些插件经过精心设计和测试，与 Vite 的核心功能高度集成，是构建生产级应用的基础。

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import react from '@vitejs/plugin-react';
import svelte from '@sveltejs/vite-plugin-svelte';
import vueJsx from '@vitejs/plugin-vue-jsx';
import legacy from '@vitejs/plugin-legacy';

export default defineConfig({
  plugins: [
    // Vue 3
    vue(),
    vueJsx(),

    // 或者 React
    react({
      // React Fast Refresh
      fastRefresh: true,
      // 自定义 Babel 插件
      babel: {
        plugins: [],
      },
    }),

    // 或者 Svelte
    svelte(),

    // 兼容旧版浏览器
    legacy({
      targets: ['defaults', 'not IE 11'],
      polyfills: true,
    }),
  ],
});
```

### 常用社区插件

Vite 的插件生态非常丰富，社区插件覆盖了从 HTML 模板处理到 PWA 支持的各个方面。以下是一些最常用的社区插件及其配置方法。

```bash
# 安装常用插件
npm install -D vite-plugin-html        # HTML 模板处理
npm install -D vite-plugin-compression # gzip/brotli 压缩
npm install -D vite-plugin-visualizer  # Bundle 可视化分析
npm install -D vite-plugin-pwa         # PWA 支持
npm install -D unplugin-auto-import    # 自动导入（API、自动组件等）
npm install -D unplugin-vue-components  # 自动导入 Vue 组件
npm install -D @originjs/vite-plugin-global-style # 全局样式
```

```typescript
// 常用社区插件配置
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import vueJsx from '@vitejs/plugin-vue-jsx';
import path from 'path';

// 自动导入 API
import AutoImport from 'unplugin-auto-import/vite';
// 自动导入组件
import Components from 'unplugin-vue-components/vite';
import { NaiveUiResolver } from 'unplugin-vue-components/resolvers';

// PWA
import { VitePWA } from 'vite-plugin-pwa';

// Bundle 分析
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),

    // 自动导入 API（ref, computed, nextTick 等）
    AutoImport({
      imports: ['vue', 'vue-router', '@vueuse/core'],
      dts: 'src/auto-imports.d.ts',
      eslintrc: {
        enabled: true,
      },
    }),

    // 自动导入组件
    Components({
      dts: 'src/components.d.ts',
      resolvers: [
        // Naive UI 组件自动导入
        NaiveUiResolver(),
        // 自定义组件目录
        (component) => {
          const { name } = component;
          if (name.startsWith('Icon')) {
            return { name, from: '@iconify/vue' };
          }
        },
      ],
    }),

    // PWA
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt'],
      manifest: {
        name: 'My App',
        short_name: 'MyApp',
        theme_color: '#ffffff',
        icons: [
          { src: 'pwa-192x192.png', sizes: '192x192', type: 'image/png' },
          { src: 'pwa-512x512.png', sizes: '512x512', type: 'image/png' },
        ],
      },
      workbox: {
        globPatterns: ['**/*.{js,css,html,ico,png,svg,woff2}'],
        runtimeCaching: [
          {
            urlPattern: /^https:\/\/fonts\.googleapis\.com/,
            handler: 'CacheFirst',
            options: {
              cacheName: 'google-fonts-cache',
              expiration: { maxEntries: 10, maxAgeSeconds: 60 * 60 * 24 * 365 },
            },
          },
        ],
      },
    }),

    // Bundle 可视化
    visualizer({
      filename: 'bundle-stats.html',
      open: true,
      gzipSize: true,
    }),
  ],
});
```

### 自定义插件开发

Vite 插件基于 Rollup 插件接口，同时提供了开发服务器特有的钩子。开发自定义插件可以满足项目中特殊的需求，也是理解 Vite 工作原理的好方法。

一个典型的 Vite 插件包含几个关键部分：名称用于标识插件；resolveId 钩子用于解析模块路径；load 钩子用于加载模块内容；transform 钩子用于转换代码；configureServer 钩子用于配置开发服务器。

```typescript
// 自定义插件示例
import { defineConfig, Plugin } from 'vite';
import fs from 'fs';
import path from 'path';

function virtualModulePlugin(): Plugin {
  const virtualModuleId = 'virtual:my-module';
  const resolvedVirtualModuleId = '\0' + virtualModuleId;

  return {
    name: 'vite-plugin-virtual-module',
    
    // 解析模块 ID
    resolveId(id: string) {
      if (id === virtualModuleId) {
        return resolvedVirtualModuleId;
      }
    },

    // 加载模块
    load(id: string) {
      if (id === resolvedVirtualModuleId) {
        // 返回虚拟模块内容
        return `
          export const version = '1.0.0';
          export const buildTime = '${new Date().toISOString()}';
          export const config = ${JSON.stringify(loadConfig())};
        `;
      }
    },

    // 转换代码
    transform(code: string, id: string) {
      if (!id.includes('virtual')) return null;
      return code;
    },

    // 配置解析完成
    configResolved(config) {
      console.log('Final config:', config);
    },

    // 开发服务器配置完成
    configureServer(server) {
      // 添加自定义中间件
      server.middlewares.use('/custom-api', (req, res) => {
        res.end(JSON.stringify({ status: 'ok' }));
      });
    },
  };
}

function loadConfig() {
  const configPath = path.resolve(__dirname, 'config.json');
  try {
    return JSON.parse(fs.readFileSync(configPath, 'utf-8'));
  } catch {
    return { default: true };
  }
}

export default defineConfig({
  plugins: [virtualModulePlugin()],
});
```

---

## Docker 部署

### 多阶段构建

将 Vite 应用部署到 Docker 容器中，最佳实践是使用多阶段构建。这种方式可以显著减小最终镜像体积，同时保持构建过程的灵活性。

多阶段构建的第一阶段是构建阶段，使用包含 Node.js 的镜像进行构建。第二阶段是运行阶段，使用轻量的 Nginx 镜像来服务静态文件。这种方式的好处是最终镜像只包含运行时需要的文件，不需要 Node.js 等构建工具。

```dockerfile
# Dockerfile
# 构建阶段
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./
RUN npm ci

# 复制源代码
COPY . .

# 构建生产版本
RUN npm run build

# 运行阶段
FROM nginx:alpine AS runner

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 Nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

Nginx 配置是 Docker 部署的重要组成部分。一个完善的 Nginx 配置应该包含静态资源缓存配置、gzip 压缩配置和 SPA 路由支持。

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # 启用 gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
    gzip_min_length 1024;

    # SPA 路由支持
    location / {
        try_files $uri $uri/ /index.html;

        # 缓存静态资源
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # API 代理
    location /api/ {
        proxy_pass http://backend:8000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### Docker Compose

对于需要同时运行前端和后端的应用，Docker Compose 是很好的编排工具。它可以定义和管理多个容器，以及它们之间的依赖关系和网络配置。

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:80"
    environment:
      - NODE_ENV=production
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    image: node:20-alpine
    working_dir: /app
    command: node server.js
    ports:
      - "8000:8000"
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

---

## Vitest 测试集成

### Vitest 配置

Vitest 是 Vite 原生的单元测试框架，它与 Vite 共享配置，提供极快的测试体验。Vitest 的设计理念与 Vite 一脉相承：极速启动、即时热更新、支持 TypeScript、开箱即用。

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';
import path from 'path';

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    include: ['src/**/*.{test,spec}.{js,ts}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/tests/',
        '*.config.*',
      ],
    },
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### 测试编写示例

```typescript
// src/utils/__tests__/format.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate, formatCurrency } from '../format';

describe('formatDate', () => {
  it('应该正确格式化日期', () => {
    const date = new Date('2024-01-15');
    expect(formatDate(date)).toBe('2024-01-15');
  });

  it('应该处理无效日期', () => {
    expect(formatDate(new Date('invalid'))).toBe('Invalid Date');
  });
});

describe('formatCurrency', () => {
  it('应该正确格式化货币', () => {
    expect(formatCurrency(1234.56)).toBe('¥1,234.56');
  });
});
```

---

## 迁移指南

### 从 Webpack 迁移

从 Webpack 迁移到 Vite 的关键在于理解两者的设计差异，并针对性地调整配置。迁移通常可以分为入口配置、别名配置、CSS 处理、资源导入和代码分割等几个方面。

**1. 调整入口文件和输出配置：**

```javascript
// Webpack: webpack.config.js
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js',
    publicPath: '/',
  },
};

// Vite: vite.config.ts
export default defineConfig({
  build: {
    outDir: 'dist',
    rollupOptions: {
      input: './src/index.html',
      output: {},
    },
  },
});
```

**2. 调整别名配置：**

```javascript
// Webpack
resolve: {
  alias: {
    '@': path.resolve(__dirname, 'src'),
  },
}

// Vite
resolve: {
  alias: {
    '@': path.resolve(__dirname, './src'),
  },
}
```

**3. 调整 CSS 处理：**

```javascript
// Webpack: 各种 loader
module: {
  rules: [
    {
      test: /\.css$/,
      use: ['style-loader', 'css-loader', 'postcss-loader'],
    },
    {
      test: /\.scss$/,
      use: ['style-loader', 'css-loader', 'sass-loader'],
    },
  ],
}

// Vite: 内置支持，只需安装预处理器
// npm install -D sass
```

**4. 调整图片等资源导入：**

```javascript
// Webpack: 可能需要 url-loader 或 file-loader
{
  test: /\.(png|jpg|gif|svg)$/,
  type: 'asset/resource',
}

// Vite: 内置支持，无需额外配置
import logo from './logo.png';
```

---

## Vite 与前端元框架的关系

### 互补而非重复

值得注意的是，Vite 与前端元框架（如 Nuxt、Next.js）定位不同，文档内容互补而非重复。元框架在 Vite 之上封装了大量框架级别的功能，如服务端渲染、路由系统、页面级优化等。本文档专注于 Vite 本身的能力和配置，而元框架文档则关注如何在其基础上构建完整的应用。

```
层次结构:

┌─────────────────────────────────────────┐
│         前端元框架 (Nuxt, Next.js)      │
│         提供 SSR、路由、数据获取等       │
├─────────────────────────────────────────┤
│            Vite (核心构建工具)           │
│            专注于开发服务器和构建优化    │
├─────────────────────────────────────────┤
│        Rollup/esbuild (底层打包器)        │
│        处理实际的模块打包和转换          │
└─────────────────────────────────────────┘
```

---

> [!TIP]
> **Vite 最佳实践**：始终使用 TypeScript 配置文件以获得类型提示；生产构建使用 `manualChunks` 进行细粒度分包以优化加载性能；结合 `unplugin-auto-import` 和 `unplugin-vue-components` 减少 import 语句；使用 `vite-plugin-pwa` 提供离线能力。对于大型项目，考虑使用 Vite 的依赖预构建和分包策略优化构建性能。

---

> [!SUMMARY]
> Vite 通过其独特的原生 ESM 开发服务器和 Rollup 生产构建的组合，实现了开发体验和生产性能的完美平衡。掌握 Vite 的配置选项、插件生态和环境变量系统，能够帮助你构建高效、可靠的前端项目。从 Webpack 迁移到 Vite 通常是值得的，特别是对于新项目和中等规模的项目。Vite 的生态系统正在快速发展，掌握其核心概念和配置技巧，将为你的前端开发工作带来显著的效率提升。

---

*本文档由 [[归愚知识系统]] 自动生成*
