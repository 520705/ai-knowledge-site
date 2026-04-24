# Turbopack 与 Rspack：新一代 Rust 原生构建工具深度指南

> [!NOTE]
> Turbopack（Vercel 出品）和 Rspack（ByteDance 出品）是新一代 Rust 原生打包工具，提供 10 倍于传统 Webpack 的构建速度。本文档将从技术定位、核心原理、实战配置等多个维度进行深度剖析。

---

## 1. 技术概述与定位

### 1.1 传统构建工具的困境

在深入了解 Turbopack 和 Rspack 之前，我们首先需要理解为什么现代前端开发迫切需要新一代构建工具。JavaScript 生态系统的蓬勃发展带来了前所未有的复杂性。前端项目的依赖数量从十年前的几十个跃升到如今的数千个，一个中等规模的企业级应用可能包含超过 500 个 npm 包。Webpack 作为 2012 年发布的工具，其基于 JavaScript 的架构在面对这种规模时显得力不从心。冷启动时间从几秒延伸到几分钟，开发者的等待时间成为了最大的效率瓶颈。

Webpack 的核心问题在于其架构设计。它使用 `enhanced-resolve` 进行模块解析，通过 `acorn` 进行代码解析，依赖 `loader` 链式调用处理各类资源，最后通过 `webpack-bundle-analyzer` 等插件进行优化。这种设计虽然灵活，但在处理大型项目时，每个模块都需要经历完整的解析-转换-打包流程。增量构建虽然存在，但由于 JavaScript 单线程的限制和复杂的缓存机制，效果往往不如预期。

更糟糕的是，Webpack 的配置复杂度随着项目规模增长而指数级上升。一个典型的 React 企业项目可能需要数百行配置，涉及 `module.rules`、`plugins`、`optimization`、`resolve` 等多个顶级配置项，每个配置项又有数十个子选项。这种配置地狱不仅增加了维护成本，也使得新人上手变得极其困难。

### 1.2 Rust 语言的降维打击

Rust 语言的出现为构建工具带来了新的可能性。Rust 是一种系统级编程语言，具有以下特性：

- **零成本抽象**：高层抽象不引入运行时开销
- **内存安全**：通过所有权系统和借用检查器避免内存泄漏
- **并发安全**：无需垃圾回收即可实现安全并发
- **接近 C++ 的性能**：编译后的代码执行效率极高

将这些特性应用于前端构建工具开发，意味着：

1. **启动速度**：Rust 的快速编译和高效执行使得工具可以在毫秒级启动
2. **增量构建**：Rust 的增量编译器（MIR）天然支持细粒度的缓存
3. **多线程并行**：无需 JavaScript 的事件循环限制，可以充分利用多核 CPU
4. **内存效率**：避免 V8 引擎的内存开销和垃圾回收停顿

### 1.3 Turbopack 的技术定位

Turbopack 是 Vercel 公司于 2022 年 10 月发布的全新构建工具，定位为「专为 JavaScript 和 TypeScript 优化的极速打包器」。它的核心技术特性包括：

**增量计算引擎**：Turbopack 使用自研的增量计算引擎，允许只重新计算发生变更的部分。当一个文件被修改时，只有依赖该文件的模块需要重新处理，其他模块直接复用缓存。这与 Webpack 的粗粒度缓存机制形成鲜明对比。

**智能代码分割**：Turbopack 的代码分割策略更加智能。它会根据模块间的引用关系、动态导入声明和运行时使用情况自动决定最优分割方案，而不是依赖开发者手动配置。

**深度 TypeScript 支持**：TypeScript 的类型检查和语法转换是构建过程中的重要耗时点。Turbopack 内置了针对 TypeScript 的优化处理，能够直接处理 TypeScript 语法而无需额外的转译步骤。

**与 Next.js 的深度集成**：目前 Turbopack 主要通过 Next.js 15 的 `--turbopack` 标志启用。Vercel 计划在未来版本中提供独立的 Turbopack CLI，使其可以在 Next.js 之外的项目中使用。

### 1.4 Rspack 的技术定位

Rspack 是字节跳动 Web Infra 团队于 2023 年 3 月开源的构建工具，定位为「高性能 Webpack 替代方案」。与 Turbopack 不同，Rspack 的设计理念是：

**完全兼容 Webpack 配置**：Rspack 提供了与 Webpack 高度兼容的 API，现有 Webpack 项目可以零成本迁移。团队对 `webpack.config.js` 中的常用配置项进行了逐一实现，确保现有配置可以直接复用。

**基于 SWC 的编译器**：Rspack 使用 SWC（Speedy Web Compiler）作为代码转译引擎。SWC 是用 Rust 编写的 JavaScript 编译器，性能是 Babel 的 20-70 倍。Rspack 内置了 `builtin:swc-loader`，无需额外安装 Babel 即可处理 TypeScript 和 JSX。

**社区驱动的插件生态**：Rspack 积极与社区合作，目前已经支持大部分常用的 Webpack 插件，如 `html-webpack-plugin`、`mini-css-extract-plugin`、`compression-webpack-plugin` 等。团队还提供了 `compat` 包来模拟 Webpack 的某些行为。

**生产级别的稳定性**：Rspack 已经在字节跳动内部经过大规模验证，被用于生产环境的数百个项目中。这确保了其在实际场景中的稳定性和可靠性。

### 1.5 架构对比分析

```javascript
// Turbopack 架构示意
Turbopack Architecture:
┌─────────────────────────────────────────────────────────────┐
│                    Turbopack Core Engine                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Module     │  │  Transform  │  │  Incremental       │  │
│  │  Resolver   │  │  Pipeline   │  │  Compute Engine    │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Bundler    │  │  Code       │  │  Asset             │  │
│  │  Graph      │  │  Splitter   │  │  Emitter           │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

```javascript
// Rspack 架构示意
Rspack Architecture:
┌─────────────────────────────────────────────────────────────┐
│                      Rspack Core                            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Webpack    │  │  SWC        │  │  Plugin             │  │
│  │  Compat     │  │  Compiler   │  │  System             │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Module     │  │  Chunk      │  │  Output             │  │
│  │  Graph      │  │  Generation │  │  Generator          │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. 完整安装与配置

### 2.1 Turbopack 安装与配置

#### 2.1.1 环境要求

Turbopack 对运行环境有特定要求：

- **Node.js 版本**：要求 Node.js 18.17.0 或更高版本。Turbopack 使用了一些较新的 Node.js API，如模块同步导入和原生 Promise 并发控制。
- **操作系统**：支持 macOS、Linux 和 Windows。在 Windows 上建议使用 WSL2 以获得最佳性能。
- **内存要求**：建议至少 8GB RAM，更复杂的项目可能需要 16GB 以上。

```bash
# 检查 Node.js 版本
node --version
# v18.17.0 或更高

# 检查 npm 版本
npm --version
# 9.x 或更高
```

#### 2.1.2 Next.js 集成安装

Turbopack 目前主要通过 Next.js 提供支持。最简单的安装方式是创建一个新的 Next.js 项目并启用 Turbopack：

```bash
# 创建新的 Next.js 项目
npx create-next-app@latest my-app --typescript

# 进入项目目录
cd my-app

# 在开发模式下使用 Turbopack
npm run dev --turbopack

# 或者使用 next dev 命令（Next.js 15+ 默认使用 Turbopack）
npx next dev --turbopack
```

对于已有的 Next.js 项目，只需升级到 Next.js 15 并使用相应的命令：

```bash
# 升级 Next.js 到最新版本
npm install next@latest react@latest react-dom@latest

# 验证安装
npx next --version
# 15.0.0 或更高
```

#### 2.1.3 Turbopack 独立安装

虽然 Turbopack 主要与 Next.js 集成，但 Vercel 也提供了独立包的预览版本：

```bash
# 安装 Turbopack CLI
npm install @vercel/turbopack -D

# 在 package.json 中添加脚本
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build"
  }
}
```

#### 2.1.4 Turbopack 配置文件

Turbopack 的配置文件是 `turbopack.config.js`，位于项目根目录：

```javascript
// turbopack.config.js
/** @type {import('@vercel/turbopack').TurbopackConfig} */
module.exports = {
  // 目标平台
  targets: [
    {
      // 浏览器端构建
      platform: 'browser',
      // 输出目录
      output: {
        path: './dist/browser',
        filename: '[name].[hash].js',
      },
    },
    {
      // 服务端构建（用于 SSR）
      platform: 'server',
      output: {
        path: './dist/server',
        filename: '[name].[hash].js',
      },
    },
  ],

  // 是否启用代码压缩
  bundle: true,

  // 是否启用代码分割
  splitChunks: true,

  // 目标环境
  env: {
    // 生产环境
    production: {
      minify: true,
      treeShaking: true,
    },
    // 开发环境
    development: {
      minify: false,
      sourceMaps: true,
    },
  },

  // 加载器配置
  loaders: {
    // TypeScript 配置
    typescript: {
      tsconfig: './tsconfig.json',
      // 是否启用严格模式
      strict: true,
    },
    // 图片加载配置
    images: {
      limit: 8192,
      formats: ['webp', 'avif'],
    },
  },

  // 实验性功能
  experimental: {
    // 启用 Rust 原生模块解析
    nativeModuleResolution: true,
    // 启用增量编译
    incrementalCompilation: true,
  },
};
```

#### 2.1.5 TypeScript 配置集成

Turbopack 可以读取项目的 `tsconfig.json` 并自动配置 TypeScript 支持：

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "checkJs": false,
    "jsx": "react-jsx",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "removeComments": true,
    "noEmit": true,
    "isolatedModules": true,
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 2.2 Rspack 安装与配置

#### 2.2.1 项目初始化

Rspack 的安装非常直接，支持从零创建项目或迁移现有 Webpack 项目：

```bash
# 创建新项目目录
mkdir my-rspack-app && cd my-rspack-app

# 初始化 npm 项目
npm init -y

# 安装 Rspack CLI 和核心包
npm install @rspack/cli @rspack/core -D

# 安装运行时依赖
npm install react react-dom
```

#### 2.2.2 创建 Rspack 配置文件

Rspack 的配置文件与 Webpack 高度兼容，可以命名为 `rspack.config.js` 或 `webpack.config.js`：

```javascript
// rspack.config.js
const rspack = require('@rspack/core');
const path = require('path');

module.exports = {
  // 入口文件
  entry: {
    main: './src/index.tsx',
  },

  // 输出配置
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    clean: true,
    publicPath: '/',
  },

  // 解析配置
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@utils': path.resolve(__dirname, 'src/utils'),
    },
    fallback: {
      path: false,
      fs: false,
    },
  },

  // 模块规则
  module: {
    rules: [
      // TypeScript 和 JSX 处理
      {
        test: /\.tsx?$/,
        use: 'builtin:swc-loader',
        exclude: /node_modules/,
      },
      // CSS 处理
      {
        test: /\.css$/,
        use: ['postcss-loader'],
        type: 'css',
      },
      // SCSS 处理
      {
        test: /\.scss$/,
        use: ['postcss-loader', 'sass-loader'],
        type: 'css',
      },
      // 图片处理
      {
        test: /\.(png|jpe?g|gif|svg|webp|avif)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'images/[name].[hash][ext]',
        },
      },
      // 字体处理
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash][ext]',
        },
      },
      // JSON 处理
      {
        test: /\.json$/,
        type: 'json',
      },
    ],
  },

  // 插件配置
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
      inject: 'body',
      scriptLoading: 'defer',
    }),
    new rspack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
    }),
  ],

  // 优化配置
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244800,
      cacheGroups: {
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name(module) {
            const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1];
            return `vendor.${packageName.replace('@', '')}`;
          },
        },
        default: {
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
      },
    },
    minimizer: [
      new rspack.SwcJsMinimizerRspackPlugin({
        format: {
          comments: false,
        },
      }),
    ],
  },

  // 开发服务器
  devServer: {
    port: 3000,
    hot: true,
    historyApiFallback: true,
    static: {
      directory: path.join(__dirname, 'public'),
    },
    client: {
      overlay: {
        errors: true,
        warnings: false,
      },
    },
  },

  // 开发模式配置
  mode: 'development',
  devtool: 'source-map',

  // 性能提示
  performance: {
    hints: false,
    maxEntrypointSize: 512000,
    maxAssetSize: 512000,
  },

  // 缓存配置
  cache: true,

  // 统计信息
  stats: 'errors-warnings',
};
```

#### 2.2.3 从 Webpack 迁移的配置映射

以下是从 Webpack 到 Rspack 的配置项映射参考：

| Webpack 配置 | Rspack 配置 | 说明 |
|-------------|-------------|------|
| `resolve.extensions` | `resolve.extensions` | 相同 |
| `resolve.alias` | `resolve.alias` | 相同 |
| `module.rules` | `module.rules` | 相同 |
| `plugins` | `plugins` | 大部分兼容 |
| `optimization.splitChunks` | `optimization.splitChunks` | 相同 |
| `plugins[HtmlWebpackPlugin]` | `plugins[HtmlRspackPlugin]` | 替换包名 |
| `builtin:swc-loader` | `builtin:swc-loader` | Rspack 内置 |

#### 2.2.4 Rspack 与其他工具集成

**与 TypeScript 集成**：

```bash
npm install typescript @types/react @types/react-dom -D
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "noEmit": true,
    "isolatedModules": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**与 React 集成**：

```javascript
// rspack.config.js - React 配置
const rspack = require('@rspack/core');

module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                  development: process.env.NODE_ENV === 'development',
                  refresh: process.env.NODE_ENV === 'development',
                },
              },
              externalHelpers: false,
            },
          },
        },
        exclude: /node_modules/,
      },
    ],
  },
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
    }),
    // React Fast Refresh 插件
    new rspack.container.ModuleFederationPlugin({
      name: 'app',
      filename: 'remoteEntry.js',
      exposes: {
        './App': './src/App',
      },
      shared: {
        react: { singleton: true, eager: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true, eager: true, requiredVersion: '^18.0.0' },
      },
    }),
  ],
};
```

**与 Vue 3 集成**：

```bash
npm install vue @vitejs/plugin-vue -D
```

```javascript
// rspack.config.js - Vue 配置
const rspack = require('@rspack/core');
const path = require('path');

module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'builtin:swc-loader',
        options: {
          jsc: {
            parser: require('vue-tsc'),
            transform: {
              vue: {
                // Vue 3 的编译器选项
              },
            },
          },
        },
      },
      // 使用 vue-loader
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          experimentalInlineMatchResource: true,
        },
      },
    ],
  },
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
    }),
  ],
};
```

---

## 3. 核心概念详解

### 3.1 Turbopack 核心概念

#### 3.1.1 增量计算引擎

Turbopack 的核心创新在于其增量计算引擎（Incremental Computation Engine）。传统的构建工具在处理文件变更时，往往需要重新扫描整个依赖图并重新处理所有受影响的模块。而 Turbopack 的增量计算引擎能够精确追踪每个模块的计算结果，并在文件变更时只重新计算必要的部分。

```javascript
// 增量计算引擎工作原理示意
Incremental Computation Flow:

┌─────────────────────────────────────────────────────────────────┐
│                        文件系统变更检测                           │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    模块依赖图（Dependency Graph）                  │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐                 │
│  │  App.tsx │──────│ Button.tsx│──────│ Icon.tsx│                │
│  └─────────┘      └─────────┘      └─────────┘                 │
│        │                                                    │
│        ▼                                                    │
│  ┌─────────┐      ┌─────────┐      ┌─────────┐                 │
│  │ Header.tsx│──────│ Nav.tsx │──────│ Menu.tsx│                │
│  └─────────┘      └─────────┘      └─────────┘                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    缓存层级（Cache Hierarchy）                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Memory Cache │  │  Disk Cache  │  │  Remote Cache│          │
│  │  (进程内)      │  │  (本地文件)   │  │  (远程共享)   │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    只重新计算受影响部分                            │
│  假设 Icon.tsx 变更：                                            │
│  - Icon.tsx → 重新计算                                          │
│  - Button.tsx → 重新计算（依赖 Icon）                            │
│  - App.tsx → 重新计算（依赖 Button）                             │
│  - Nav.tsx → 跳过（不依赖 Icon）                                  │
│  - Menu.tsx → 跳过（不依赖 Icon）                                 │
└─────────────────────────────────────────────────────────────────┘
```

增量计算引擎的三个核心组件：

**变更检测器（Change Detection）**：使用操作系统级文件监控（如 inotify、FSevents、FSEvents）追踪文件变更，支持增量扫描以检测新增和删除的文件。

**依赖追踪器（Dependency Tracker）**：维护完整的模块依赖图，记录每个模块依赖哪些模块以及被哪些模块依赖。当某个模块变更时，可以快速确定所有受影响的上游和下游模块。

**结果缓存器（Result Cache）**：采用内容寻址存储（Content-Addressed Storage），根据文件内容和环境配置生成唯一哈希作为缓存键。即使删除后重建，只要内容相同即可命中缓存。

#### 3.1.2 模块解析（Module Resolution）

Turbopack 的模块解析系统支持多种协议和查找策略：

```javascript
// Turbopack 支持的协议类型
Module Resolution Protocols:

1. 文件系统协议 (file://)
   import "./utils/helper";
   import "/Users/project/src/utils/helper";

2. npm 协议 (node_modules/)
   import "react";
   import "@scope/package";
   import "package/subpath";

3. URL 协议
   import "data:text/javascript,export const x = 1;";
   import "https://cdn.example.com/lib.js";

4. 别名协议
   import "@/components/Button";
   import "~assets/logo.png";
```

解析过程遵循以下优先级：

1. **绝对路径**：如果路径以 `/` 开头，直接解析为文件系统绝对路径
2. **相对路径**：如果路径以 `./` 或 `../` 开头，相对于当前文件位置解析
3. **模块路径**：在 `node_modules` 目录中查找，或使用别名映射
4. **自定义协议**：检查是否匹配自定义解析器（如 `virtual:`、`raw:` 等）

#### 3.1.3 代码转换管道（Transform Pipeline）

Turbopack 的代码转换管道采用流水线架构，每个转换步骤（transform）都是独立的，可以并行处理：

```javascript
// 代码转换管道流程
Transform Pipeline:

Source Code (.ts/.tsx/.js/.jsx)
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 1: 解析（Parsing）                                 │
│  - 词法分析（Tokenization）                                │
│  - 语法分析（AST Generation）                              │
│  - 类型检查（Type Checking）← Turbopack 跳过此步          │
└─────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 2: 转换（Transformation）                          │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐                 │
│  │ SWC     │  │ Babel   │  │ Custom  │                 │
│  │ Plugins │  │ Plugins │  │ Plugins │                 │
│  └─────────┘  └─────────┘  └─────────┘                 │
└─────────────────────────────────────────────────────────┘
           │
           ▼
┌─────────────────────────────────────────────────────────┐
│  Phase 3: 生成（Generation）                               │
│  - AST → Code                                             │
│  - Source Maps Generation                                 │
│  - Code Splitting                                         │
└─────────────────────────────────────────────────────────┘
           │
           ▼
        Output
```

转换管道支持以下常见转换：

- **TypeScript**：语法转换（保留类型注解）和移除（通过 `tsc` 单独处理）
- **JSX/TSX**：转换为 `React.createElement` 调用或使用新的 JSX 运行时
- **Decorator**：转换为符合 ECMAScript 标准的格式
- **Import/Export**：转换为目标模块系统（ESM/CJS/UMD）
- **Flow**：语法转换

#### 3.1.4 树摇优化（Tree Shaking）

Turbopack 内置了强大的树摇（Tree Shaking）优化能力，能够消除未使用的代码（dead code）：

```javascript
// 树摇的工作原理
Tree Shaking Analysis:

// 源文件：math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export function multiply(a, b) {
  return a * b;
}

export function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

// 使用文件：index.js
import { add } from './math.js';

console.log(add(1, 2));

// 树摇后的输出（理想情况）
function add(a, b) {
  return a + b;
}

console.log(add(1, 2));

// 以下函数被完全移除：
// - subtract
// - multiply
// - divide
```

树摇的三个前提条件：

1. **静态分析**：代码必须是静态可分析的，不能使用动态 `import()` 以外的条件导入
2. **ES Modules**：源代码必须使用 ES Module 语法（`import`/`export`），而非 CommonJS
3. **副作用标记**：需要正确标记有副作用的模块（side effects），以便构建工具判断是否可以安全移除

```javascript
// package.json 中的副作用配置
{
  "name": "my-package",
  "sideEffects": false  // 所有文件都没有副作用
}

// 或者指定有副作用的文件
{
  "sideEffects": [
    "./src/polyfills.js",
    "./src/analytics.js"
  ]
}
```

#### 3.1.5 Turbopack 缓存机制详解

Turbopack 的缓存系统是其高性能的关键组成部分，它采用了多级缓存架构来最大化构建效率。

**内存缓存层（In-Memory Cache）**：
内存缓存是最快的缓存层，存储在进程内存中。当 Turbopack 处理文件时，首先检查内存缓存是否已有该文件的处理结果。内存缓存的特点是读写速度极快（纳秒级），但容量有限，且在进程重启后会丢失。Turbopack 的内存缓存采用内容寻址存储（Content-Addressed Storage），每个缓存项都有一个基于文件内容和环境配置的哈希键。

```javascript
// Turbopack 内存缓存工作流程
Memory Cache Flow:

┌─────────────────────────────────────────────────────────────────┐
│                     文件变更检测                                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     内容哈希计算                                    │
│  hash = SHA256(fileContent + dependencies + envConfig)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   内存缓存查询 (L1 Cache)                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Key: hash_value                                        │   │
│  │  Value: {                                                │   │
│  │    transformedCode: "..."                               │   │
│  │    dependencies: ["dep1", "dep2"],                      │   │
│  │    metadata: { size, lineCount }                         │   │
│  │  }                                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │ 命中                        │ 未命中
              ▼                              ▼
     返回缓存结果                    继续处理文件
```

**磁盘缓存层（Disk Cache）**：
磁盘缓存是第二级缓存，使用文件系统存储处理结果。与内存缓存相比，磁盘缓存容量更大（可以存储数 GB 的数据），并且在进程重启后仍然保留。Turbopack 的磁盘缓存默认存储在 `.turbo/cache` 目录下。

```javascript
// 磁盘缓存配置
{
  cache: {
    // 缓存类型
    type: 'filesystem',
    
    // 缓存目录
    directory: '.turbo/cache',
    
    // 缓存版本（改变会强制重建）
    version: 'v1',
    
    // 压缩设置
    compress: true,
    
    // 最大缓存大小（字节）
    maxCacheSize: 1024 * 1024 * 1024 * 10, // 10GB
    
    // 单个文件最大缓存大小
    maxFileCacheSize: 1024 * 1024 * 100, // 100MB
    
    // 缓存保留时间（秒）
    maxAge: 7 * 24 * 60 * 60, // 7天
  }
}
```

**远程缓存层（Remote Cache）**：
对于团队协作场景，Turbopack 支持远程缓存。远程缓存允许多台机器共享构建结果，当团队成员构建相同的代码时，可以直接下载远程缓存的构建结果，而无需重新构建。

```javascript
// 远程缓存配置
{
  remoteCache: {
    // 远程缓存服务 URL
    url: 'https://turbo-cache.example.com',
    
    // 团队标识
    team: 'my-team',
    
    // API Token（用于认证）
    token: process.env.TURBO_TOKEN,
    
    // 是否启用远程缓存
    enabled: process.env.CI === 'true',
    
    // 缓存策略
    strategy: 'remote-first', // 'remote-first' | 'local-only' | 'remote-only'
    
    // 离线模式
    offline: false,
  }
}
```

**缓存失效策略**：
Turbopack 使用精细化的缓存失效策略来确定何时需要重新构建：

1. **文件内容哈希**：如果文件的 SHA-256 哈希发生变化，缓存失效
2. **依赖图哈希**：如果文件的依赖项发生变化，缓存失效
3. **环境配置哈希**：如果环境变量或构建配置发生变化，相关缓存失效
4. **工具链哈希**：如果 Turbopack 版本或编译器版本发生变化，所有缓存失效

```javascript
// 缓存键生成逻辑
function generateCacheKey(options) {
  const { fileContent, dependencies, envConfig, toolchainVersion } = options;
  
  const contentHash = crypto.createHash('sha256')
    .update(fileContent)
    .digest('hex');
  
  const depsHash = crypto.createHash('sha256')
    .update(JSON.stringify(dependencies.sort()))
    .digest('hex');
  
  const envHash = crypto.createHash('sha256')
    .update(JSON.stringify(envConfig))
    .digest('hex');
  
  return crypto.createHash('sha256')
    .update(`${contentHash}:${depsHash}:${envHash}:${toolchainVersion}`)
    .digest('hex');
}
```

#### 3.1.6 Turbopack 与 Next.js 深度集成

Turbopack 与 Next.js 15 的集成是其最重要的应用场景，这种集成带来了显著的性能提升。

**App Router 支持**：
Next.js 15 的 App Router 是现代 React 应用的标准路由方案，Turbopack 对其提供了完整的支持：

```javascript
// next.config.js - App Router 配置
const nextConfig = {
  // 启用 Turbopack
  experimental: {
    turbo: {
      // App Router 配置
      appDir: true,
      
      // 服务器组件支持
      serverComponents: true,
      
      // 流式 SSR 支持
      serverActions: {
        allowedOrigins: ['localhost:3000'],
      },
    },
  },
  
  // 图片优化配置
  images: {
    formats: ['image/avif', 'image/webp'],
    remotePatterns: [
      { protocol: 'https', hostname: '**.amazonaws.com' },
    ],
  },
};
```

**动态导入与代码分割**：
Turbopack 自动处理 Next.js 的动态导入，优化 bundle 分割：

```javascript
// app/page.tsx
import dynamic from 'next/dynamic';

// 动态导入重型组件
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <Skeleton />,
  ssr: true,
});

export default function Page() {
  return (
    <div>
      <h1>Dashboard</h1>
      <HeavyChart data={chartData} />
    </div>
  );
}
```

**路由预取优化**：
Turbopack 与 Next.js 的 Link 组件集成，自动进行路由预取：

```javascript
// app/components/Navigation.tsx
import Link from 'next/link';

// 默认行为：鼠标悬停时预取
export function Navigation() {
  return (
    <nav>
      <Link href="/dashboard">Dashboard</Link>
      <Link href="/settings">Settings</Link>
      <Link href="/profile">Profile</Link>
    </nav>
  );
}

// 手动控制预取策略
import { useRouter } from 'next/router';

function PrefetchLink({ href, children }) {
  const router = useRouter();
  
  return (
    <Link 
      href={href}
      prefetch={false}  // 禁用自动预取
      onMouseEnter={() => {
        router.prefetch(href);  // 手动预取
      }}
    >
      {children}
    </Link>
  );
}
```

**增量静态再生成（ISR）**：
Turbopack 支持 Next.js 的增量静态再生成功能：

```javascript
// app/blog/[slug]/page.tsx
export const revalidate = 60; // 每 60 秒重新生成

async function getPost(slug) {
  const res = await fetch(`https://api.example.com/posts/${slug}`, {
    next: { revalidate: 60 },  // ISR 配置
  });
  return res.json();
}

export default async function BlogPost({ params }) {
  const post = await getPost(params.slug);
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  );
}
```

### 3.2 Rspack 核心概念

#### 3.2.1 Webpack 兼容层深度解析

Rspack 的 Webpack 兼容层是其最核心的设计特性之一，这个兼容层确保了大量现有的 Webpack 配置和插件可以在 Rspack 中无缝使用。

**配置兼容的实现原理**：
Rspack 的配置兼容不是简单的语法转换，而是深度模拟了 Webpack 的配置解析和处理逻辑。

```javascript
// Rspack 配置加载流程
Configuration Loading Flow:

┌─────────────────────────────────────────────────────────────────┐
│                 配置文件读取                                     │
│  rspack.config.js / webpack.config.js                           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              Webpack 配置标准化                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  1. 解析 webpack.config.js 导出                          │   │
│  │  2. 处理环境变量（process.env.NODE_ENV）                  │   │
│  │  3. 应用 webpack-merge 配置合并                           │   │
│  │  4. 验证配置 Schema                                     │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              Webpack 配置到 Rspack 配置转换                       │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  resolve.extensions → resolve.extensions                │   │
│  │  module.rules → module.rules                            │   │
│  │  plugins → plugins (部分需要适配)                       │   │
│  │  optimization → optimization                           │   │
│  │  output → output                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 Rspack 内部处理                                   │
└─────────────────────────────────────────────────────────────────┘
```

**兼容的 Webpack 配置项**：
Rspack 高度兼容以下 Webpack 配置项：

```javascript
// 完整兼容的配置项
const compatibleConfig = {
  // 入口配置
  entry: './src/index.js',
  
  // 输出配置
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
    publicPath: '/',
    clean: true,
  },
  
  // 解析配置
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
    mainFields: ['browser', 'module', 'main'],
    symlinks: false,
  },
  
  // 模块配置
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: 'builtin:swc-loader',
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        type: 'css',
      },
    ],
  },
  
  // 插件配置
  plugins: [
    new rspack.HtmlRspackPlugin({ template: './public/index.html' }),
  ],
  
  // 优化配置
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
    minimizer: [
      new rspack.SwcJsMinimizerRspackPlugin(),
    ],
  },
  
  // 开发服务器
  devServer: {
    port: 3000,
    hot: true,
  },
  
  // 性能提示
  performance: {
    hints: 'warning',
    maxEntrypointSize: 512000,
    maxAssetSize: 512000,
  },
  
  // 缓存配置
  cache: true,
  
  // 目标平台
  target: 'web',
  
  // 模式
  mode: 'production',
  
  // 上下文
  context: __dirname,
  
  // 记录
  record: false,
  
  // 快照
  snapshot: {
    managedPaths: [/node_modules/],
  },
};
```

**需要适配的配置项**：
某些 Webpack 配置项需要特别处理或替代方案：

```javascript
// 需要适配的配置项
const adaptedConfig = {
  // HtmlWebpackPlugin → HtmlRspackPlugin
  // plugins: [
  //   new HtmlWebpackPlugin({ template: './src/index.html' })
  //   ↓
  //   new rspack.HtmlRspackPlugin({ template: './src/index.html' })
  // ]
  
  // MiniCssExtractPlugin → rspack.CssMinimizerRspackPlugin
  // new MiniCssExtractPlugin({ filename: '[name].css' })
  // ↓
  // new rspack.CssMinimizerRspackPlugin()
  
  // babel-loader → builtin:swc-loader
  // {
  //   test: /\.tsx?$/,
  //   use: 'babel-loader'
  // }
  // ↓
  // {
  //   test: /\.tsx?$/,
  //   use: 'builtin:swc-loader'
  // }
  
  // DefinePlugin → rspack.DefinePlugin (完全兼容)
  // new webpack.DefinePlugin({ ... })
  // ↓
  // new rspack.DefinePlugin({ ... })
};
```

**Rspack 特有的配置项**：
除了 Webpack 兼容的配置项，Rspack 还提供了一些特有的配置选项：

```javascript
// Rspack 特有配置
const rspackSpecificConfig = {
  // 实验性功能
  experiments: {
    // 启用 Rust 解析器（更快）
    rspackResolver: true,
    
    // 启用 CSS Modules
    css: true,
    
    // 顶层 await 支持
    topLevelAwait: true,
  },
  
  // 内置 SWC 编译器选项
  builtins: {
    // React 配置
    react: {
      runtime: 'automatic',  // 'automatic' | 'classic'
      development: process.env.NODE_ENV === 'development',
      refresh: process.env.NODE_ENV === 'development',
    },
    
    // 定义全局变量
    define: {
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV),
    },
    
    // 按需引入 polyfill
    polyfills: {
      iterator: true,
      generator: true,
    },
  },
  
  // 源码映射类型
  devtool: 'source-map',
  
  // 统计信息
  stats: {
    colors: true,
    modules: true,
    children: true,
  },
};
```

#### 3.2.2 SWC 编译器深度解析（续）

SWC 编译器是 Rspack 的核心动力，理解它的工作原理对于深度使用 Rspack 至关重要。

**SWC 的解析器实现**：
SWC 的解析器是用 Rust 编写的高性能 JavaScript/TypeScript 解析器。它采用了手写的解析器（Hand-written Parser）而非生成的解析器，以获得更好的性能和更精确的错误报告。

```rust
// SWC 解析器架构（概念性 Rust 代码）
pub struct Parser {
    // 输入源
    input: Input,
    
    // token 流
    tokens: Vec<Token>,
    
    // 当前解析位置
    pos: BytePos,
}

impl Parser {
    // 创建解析器实例
    pub fn new(source: &str) -> Self {
        Parser {
            input: Input::new(source),
            tokens: Vec::new(),
            pos: BytePos(0),
        }
    }
    
    // 解析文件
    pub fn parse_file(&mut self) -> Result<File, Error> {
        let mut module = Module {
            span: DUMMY_SP,
            body: vec![],
            shebang: None,
        };
        
        // 循环解析直到文件结束
        while !self.is_eof() {
            // 跳过空白符
            self.skip_whitespace();
            
            if self.is_eof() {
                break;
            }
            
            // 解析语句
            let stmt = self.parse_stmt()?;
            module.body.push(ModuleItem::Stmt(stmt));
        }
        
        Ok(File { module })
    }
    
    // 解析语句
    fn parse_stmt(&mut self) -> Result<Stmt, Error> {
        match self.curr_token_kind() {
            TokenKind::Let | TokenKind::Const => self.parse_var_decl(),
            TokenKind::Function => self.parse_fn_decl(),
            TokenKind::If => self.parse_if_stmt(),
            TokenKind::Return => self.parse_return_stmt(),
            // ... 更多语句类型
            _ => self.parse_expr_stmt(),
        }
    }
}
```

**SWC 的 AST 设计**：
SWC 使用了独特的 AST 设计来优化内存使用和解析速度：

```rust
// SWC AST 节点设计
#[derive(Debug, Clone, Fold, Visit, Walk)]
pub struct Program {
    pub span: Span,
    pub body: Vec<ModuleItem>,
    pub shebang: Option<String>,
}

#[derive(Debug, Clone, Fold, Visit, Walk)]
pub enum ModuleItem {
    ModuleDecl(ModuleDecl),
    Stmt(Stmt),
}

#[derive(Debug, Clone, Fold, Visit, Walk)]
pub enum ModuleDecl {
    Import(ImportDecl),
    Export(ExportDecl),
    ExportNamed(NamedExport),
    ExportDefault(ExportDefault),
    ExportAll(ExportAll),
}

#[derive(Debug, Clone, Fold, Visit, Walk)]
pub enum Stmt {
    Block(BlockStmt),
    Empty(EmptyStmt),
    Debugger(DebuggerStmt),
    With(WithStmt),
    Return(ReturnStmt),
    Labeled(LabeledStmt),
    Break(BreakStmt),
    Continue(ContinueStmt),
    If(IfStmt),
    Switch(SwitchStmt),
    Throw(ThrowStmt),
    Try(TryStmt),
    While(WhileStmt),
    DoWhile(DoWhileStmt),
    For(ForStmt),
    ForIn(ForInStmt),
    ForOf(ForOfStmt),
    Func(FnDecl),
    Class(ClassDecl),
    Decl(Decl),
    Expr(ExprStmt),
}
```

**SWC 的转换管道**：
SWC 的转换管道采用了流水线架构，每个转换步骤都可以独立执行和并行处理：

```
SWC Transform Pipeline:

Source Code
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                   Phase 1: Parse                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Lexer → Tokens → AST (Concrete)                │   │
│  │  并行处理多文件                                   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                   Phase 2: Visit                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │  AST (Concrete) → AST (Abstract)               │   │
│  │  深度遍历 + 节点转换                             │   │
│  │  自定义 Visitor 实现                             │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                   Phase 3: Fold                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │  语义分析 + 优化                                  │   │
│  │  Tree Shaking 分析                               │   │
│  │  常量折叠                                        │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                   Phase 4: Output                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │  AST → Code                                      │   │
│  │  Source Map 生成                                 │   │
│  │  Minification                                   │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
     │
     ▼
Transformed Code
```

**SWC 插件系统**：
SWC 支持自定义插件来扩展其转换能力：

```javascript
// 自定义 SWC 插件示例
// 插件必须是编译成 .wasm 或 .d.ts 的 Rust 代码

// my-plugin/src/lib.rs
use swc_core::{
    ecma::ast::*,
    ecma::visit::{as_folder, FoldWith, VisitMut},
    plugin::plugin_transform,
};

pub struct TransformPlugin;

impl VisitMut for TransformPlugin {
    // 处理函数声明
    fn visit_mut_fn_decl(&mut self, fn_decl: &mut FnDecl) {
        fn_decl.visit_mut_children_with(self);
        
        // 在函数名后添加 _transformed 后缀
        if let Some(ident) = &mut fn_decl.ident {
            ident.sym = format!("{}_transformed", ident.sym).into();
        }
    }
    
    // 处理变量声明
    fn visit_mut_var_decl(&mut self, var_decl: &mut VarDecl) {
        var_decl.visit_mut_children_with(self);
        
        // 将所有 const 转换为 let（示例）
        if var_decl.kind == VarDeclKind::Const {
            var_decl.kind = VarDeclKind::Let;
        }
    }
}

#[plugin_transform]
pub fn process_transform(program: Program, _metadata: TransformPluginConfig) -> Program {
    program.fold_with(&mut as_folder(TransformPlugin))
}
```

#### 3.2.3 模块系统与 Chunk 生成（续）

**高级代码分割策略**：
Rspack 提供了细粒度的代码分割控制，允许开发者根据业务需求定制分割策略：

```javascript
// rspack.config.js - 高级代码分割
module.exports = {
  optimization: {
    splitChunks: {
      // 全局配置
      chunks: 'all',
      minSize: 20000,
      maxSize: 250000,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,
      
      // 缓存组配置
      cacheGroups: {
        // React 生态系统 - 最高优先级
        reactVendor: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router|scheduler)[\\/]/,
          name: 'vendor-react',
          chunks: 'all',
          priority: 40,
          enforce: true,
        },
        
        // UI 组件库
        uiVendor: {
          test: /[\\/]node_modules[\\/](@mui|@ant-design|antd|chakra-ui|radix|headlessui)[\\/]/,
          name: 'vendor-ui',
          chunks: 'all',
          priority: 35,
          enforce: true,
        },
        
        // 状态管理
        stateVendor: {
          test: /[\\/]node_modules[\\/](@reduxjs|redux|zustand|jotai|mobx|mobx-react)[\\/]/,
          name: 'vendor-state',
          chunks: 'all',
          priority: 30,
        },
        
        // 数据处理库
        dataVendor: {
          test: /[\\/]node_modules[\\/](lodash|lodash-es|axios|ky|swr|react-query|@tanstack)[\\/]/,
          name: 'vendor-data',
          chunks: 'all',
          priority: 25,
        },
        
        // 工具函数库
        utilVendor: {
          test: /[\\/]node_modules[\\/](date-fns|moment|dayjs|clsx|classnames|object-hash|uuid)[\\/]/,
          name: 'vendor-util',
          chunks: 'all',
          priority: 20,
        },
        
        // 低频大型库
        heavyVendor: {
          test: /[\\/]node_modules[\\/](echarts|d3|three|@apollo|graphql|monaco|pdfjs)[\\/]/,
          name: 'vendor-heavy',
          chunks: 'async',
          priority: 15,
          enforce: true,
        },
        
        // 第三方 SDK
        sdkVendor: {
          test: /[\\/]node_modules[\\/](@sentry|@analytics|segment|firebase|amplitude)[\\/]/,
          name: 'vendor-sdk',
          chunks: 'all',
          priority: 10,
        },
        
        // 公共模块 - 至少被两个入口使用
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: -10,
          reuseExistingChunk: true,
          enforce: true,
        },
      },
    },
    
    // 运行时 chunk 单独提取
    runtimeChunk: 'single',
    
    // 模块 ID 策略
    moduleIds: 'deterministic',
    chunkIds: 'deterministic',
    
    // 标识可用导出
    usedExports: true,
    
    // 侧 Effects 标记
    sideEffects: true,
    
    // 提供导出
    providedExports: true,
  },
};
```

**动态导入与预取策略**：
合理的动态导入和预取可以显著提升应用性能：

```typescript
// src/routes.tsx
import { lazy, Suspense } from 'react';
import { Routes, Route, Navigate } from 'react-router-dom';

// 基础路由 - 使用静态导入
import Dashboard from './pages/Dashboard';
import Settings from './pages/Settings';
import Profile from './pages/Profile';

// 重型页面 - 使用动态导入
const Analytics = lazy(() => 
  import(/* webpackChunkName: "analytics", webpackPrefetch: true */ './pages/Analytics')
);

const Reports = lazy(() => 
  import(/* webpackChunkName: "reports", webpackPrefetch: true */ './pages/Reports')
);

const Admin = lazy(() => 
  import(/* webpackChunkName: "admin", webpackPrefetch: true */ './pages/Admin')
);

// 配置页面 - 使用按需加载
const SystemSettings = lazy(() => 
  import(/* webpackChunkName: "system-settings" */ './pages/SystemSettings')
);

const UserManagement = lazy(() => 
  import(/* webpackChunkName: "user-management" */ './pages/UserManagement')
);

// 报告页面 - 延迟预取
const PerformanceReport = lazy(() => {
  const promise = import(/* webpackChunkName: "performance-report" */ './pages/PerformanceReport');
  // 鼠标悬停时预取
  window?.requestIdleCallback?.(() => promise);
  return promise;
});

function LoadingFallback() {
  return (
    <div className="loading-skeleton">
      <div className="skeleton-header" />
      <div className="skeleton-content" />
    </div>
  );
}

export function AppRoutes() {
  return (
    <Suspense fallback={<LoadingFallback />}>
      <Routes>
        <Route path="/" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
        <Route path="/analytics" element={<Analytics />} />
        <Route path="/reports/*" element={<Reports />} />
        <Route path="/admin/*" element={<Admin />} />
        <Route path="/admin/settings" element={<SystemSettings />} />
        <Route path="/admin/users" element={<UserManagement />} />
        <Route path="/reports/performance" element={<PerformanceReport />} />
        <Route path="*" element={<Navigate to="/" replace />} />
      </Routes>
    </Suspense>
  );
}
```

**模块 Federation 配置**：
Rspack 支持 Module Federation，用于微前端架构：

```javascript
// host/rspack.config.js - 主机应用
const rspack = require('@rspack/core');
const { container } = require('@module-federation/rspack');

module.exports = {
  plugins: [
    new container.ModuleFederationPlugin({
      // 应用名称
      name: 'host_app',
      
      // 共享依赖
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
          eager: true,
          strictVersion: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
          eager: true,
          strictVersion: true,
        },
        'react-router-dom': {
          singleton: true,
          requiredVersion: '^6.0.0',
        },
      },
      
      // 远程模块
      remotes: {
        remote_app: 'remote_app@https://remote.example.com/remoteEntry.js',
        design_system: 'design_system@https://cdn.example.com/design-system/remoteEntry.js',
      },
      
      // 暴露的模块
      exposes: {
        './App': './src/App',
        './Store': './src/store',
        './Config': './src/config',
      },
      
      // 文件名
      filename: 'remoteEntry.js',
      
      // 共享范围
      shareScope: 'default',
      
      // 库目标
      library: {
        type: 'module',
      },
    }),
  ],
};

// remote/rspack.config.js - 远程应用
module.exports = {
  plugins: [
    new container.ModuleFederationPlugin({
      name: 'remote_app',
      
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
      },
      
      // 暴露的组件
      exposes: {
        './Button': './src/components/Button',
        './Modal': './src/components/Modal',
        './Dashboard': './src/pages/Dashboard',
      },
      
      filename: 'remoteEntry.js',
    }),
  ],
};
```

#### 3.2.4 插件系统架构（续）

**Rspack 插件生命周期详解**：
Rspack 的插件系统基于 Tapable，提供了丰富的生命周期钩子：

```javascript
// 插件生命周期完整图示
Plugin Lifecycle:

┌─────────────────────────────────────────────────────────────────┐
│                    Compiler 生命周期                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │ beforeRun   │ ← 编译开始前，清理输出                          │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │    run      │ ← 编译正式开始                                  │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │ beforeCompile│ ← 编译前准备                                    │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │   compile   │ ← 创建新 Compilation                             │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │    make     │ ← 构建模块                                      │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │ afterCompile│ ← 编译完成                                      │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │    emit     │ ← 输出文件到目录                                │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │    done     │ ← 编译完全结束                                  │
│  └─────────────┘                                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Compilation 生命周期                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌─────────────┐                                                │
│  │ addEntry    │ ← 添加入口点                                     │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │  finish     │ ← 模块构建完成                                  │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ▼                                                        │
│  ┌─────────────┐                                                │
│  │    seal    │ ← 开始优化和生成                                 │
│  └──────┬──────┘                                                │
│         │                                                        │
│         ├──────────────────────────────────────────────┐        │
│         │                                                      │        │
│         ▼                                                      ▼        │
│  ┌─────────────┐                        ┌─────────────┐            │
│  │ optimize   │ ←─────────────→ │   seal  │            │
│  └──────┬─────┘                        └──────┬─────┘            │
│         │                                      │                   │
│         ▼                                      │                   │
│  ┌─────────────┐                               │                   │
│  │  moduleIds  │                               │                   │
│  └──────┬─────┘                               │                   │
│         │                                      │                   │
│         ▼                                      │                   │
│  ┌─────────────┐                               │                   │
│  │  chunkIds  │                               │                   │
│  └──────┬─────┘                               │                   │
│         │                                      │                   │
│         ▼                                      ▼                   │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐            │
│  │  optimize   │ → │   module    │ → │    chunk    │            │
│  │   chunks   │   │ generation │   │ generation │            │
│  └─────────────┘   └──────┬─────┘   └─────────────┘            │
│                            │                                      │
│                            ▼                                      │
│                    ┌─────────────┐                               │
│                    │chunkAssets │                               │
│                    └──────┬─────┘                               │
│                           │                                       │
│                           ▼                                       │
│                    ┌─────────────┐                               │
│                    │  process    │ ← 处理资源                      │
│                    │   Assets   │                               │
│                    └─────────────┘                               │
│                           │                                       │
│                           ▼                                       │
│                    ┌─────────────┐                               │
│                    │   complete  │ ← 完成                          │
│                    └─────────────┘                               │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**复杂插件开发示例**：
```javascript
// rspack.config.js - 自定义构建分析插件
const rspack = require('@rspack/core');

class BundleAnalyzerPlugin {
  constructor(options = {}) {
    this.options = {
      analyzerMode: 'server', // 'server' | 'static' | 'json' | 'disabled'
      reportFilename: 'bundle-report.html',
      openAnalyzer: true,
      generateStatsFile: false,
      statsFilename: 'bundle-stats.json',
      logLevel: 'info', // 'info' | 'warn' | 'error' | 'silent'
      ...options,
    };
    
    this.analyzerPort = 8888;
    this.server = null;
  }
  
  apply(compiler) {
    // 钩子注册
    this.registerHooks(compiler);
    
    // 插件名称
    compiler.options.plugins.push(this);
  }
  
  registerHooks(compiler) {
    // 编译开始钩子
    compiler.hooks.beforeCompile.tap('BundleAnalyzer', (compilationParams) => {
      this.log('info', 'Starting bundle analysis...');
      this.buildStartTime = Date.now();
    });
    
    // 编译完成钩子
    compiler.hooks.done.tap('BundleAnalyzer', (stats) => {
      const analysis = this.analyzeBundle(stats);
      
      // 生成报告
      this.generateReport(analysis);
      
      // 输出统计信息
      this.outputStats(analysis);
      
      this.log('info', `Analysis completed in ${Date.now() - this.buildStartTime}ms`);
    });
    
    // 发出资源钩子
    compiler.hooks.emit.tapAsync('BundleAnalyzer', (compilation, callback) => {
      // 收集资源信息
      this.assetStats = {};
      
      for (const [filename, asset] of Object.entries(compilation.assets)) {
        this.assetStats[filename] = {
          size: asset.size(),
          sizeFormatted: this.formatSize(asset.size()),
          chunkNames: asset.getChunkNames?.() || [],
        };
      }
      
      callback();
    });
    
    // 优化模块钩子
    compilation.hooks.optimizeModules.tap('BundleAnalyzer', (modules) => {
      this.moduleStats = {
        total: modules.length,
        byType: this.categorizeModules(modules),
      };
    });
    
    // 优化 chunk 钩子
    compilation.hooks.optimizeChunks.tap('BundleAnalyzer', (chunks) => {
      this.chunkStats = chunks.map(chunk => ({
        id: chunk.id,
        name: chunk.name,
        size: chunk.size(),
        modules: chunk.modules.size,
        parents: Array.from(chunk.parents).map(p => p.id),
        children: Array.from(chunk.children).map(c => c.id),
      }));
    });
  }
  
  analyzeBundle(stats) {
    const json = stats.toJson({
      all: false,
      modules: true,
      chunks: true,
      assets: true,
      entrypoints: true,
      outputPath: true,
    });
    
    return {
      // 构建信息
      buildInfo: {
        hash: stats.hash,
        time: stats.endTime - stats.startTime,
        builtAt: stats.builtAt,
      },
      
      // 模块信息
      modules: this.processModules(json.modules || []),
      
      // Chunk 信息
      chunks: this.processChunks(json.chunks || []),
      
      // 资源信息
      assets: this.assetStats,
      
      // 包大小统计
      sizeStats: this.calculateSizeStats(json.assets || []),
      
      // 入口点信息
      entrypoints: this.processEntrypoints(json.entrypoints || {}),
    };
  }
  
  processModules(modules) {
    return modules
      .filter(m => m.type !== 'concatenated')
      .map(m => ({
        name: m.name,
        size: m.size,
        id: m.identifier,
        path: m.modulePath,
        categories: this.categorizeModule(m),
        chunks: m.chunks,
      }))
      .sort((a, b) => b.size - a.size);
  }
  
  processChunks(chunks) {
    return chunks.map(chunk => ({
      id: chunk.id,
      names: chunk.names,
      size: chunk.size,
      files: chunk.files,
      modules: chunk.modules?.length || 0,
    }));
  }
  
  categorizeModules(modules) {
    const categories = {
      vendor: { size: 0, count: 0 },
      app: { size: 0, count: 0 },
      other: { size: 0, count: 0 },
    };
    
    for (const mod of modules) {
      if (mod.name?.includes('node_modules')) {
        categories.vendor.size += mod.size;
        categories.vendor.count++;
      } else if (mod.name?.includes('src/')) {
        categories.app.size += mod.size;
        categories.app.count++;
      } else {
        categories.other.size += mod.size;
        categories.other.count++;
      }
    }
    
    return categories;
  }
  
  calculateSizeStats(assets) {
    const stats = {
      total: 0,
      byType: {},
      largest: [],
    };
    
    for (const asset of assets) {
      stats.total += asset.size;
      
      const ext = asset.name.split('.').pop();
      if (!stats.byType[ext]) {
        stats.byType[ext] = { size: 0, count: 0 };
      }
      stats.byType[ext].size += asset.size;
      stats.byType[ext].count++;
      
      stats.largest.push({
        name: asset.name,
        size: asset.size,
      });
    }
    
    stats.largest.sort((a, b) => b.size - a.size);
    stats.largest = stats.largest.slice(0, 10);
    
    return stats;
  }
  
  processEntrypoints(entrypoints) {
    const result = {};
    
    for (const [name, entry] of Object.entries(entrypoints)) {
      result[name] = {
        chunks: entry.chunks.map(id => {
          const chunk = this.chunkStats?.find(c => c.id === id);
          return chunk || { id };
        }),
        size: entry.chunks.reduce((sum, id) => {
          const chunk = this.chunkStats?.find(c => c.id === id);
          return sum + (chunk?.size || 0);
        }, 0),
      };
    }
    
    return result;
  }
  
  generateReport(analysis) {
    const reportData = JSON.stringify(analysis, null, 2);
    const reportPath = path.join(process.cwd(), this.options.reportFilename);
    
    // 写入报告文件
    fs.writeFileSync(reportPath, reportData);
    
    this.log('info', `Report saved to: ${reportPath}`);
    
    // 如果需要启动服务器
    if (this.options.analyzerMode === 'server') {
      this.startAnalyzerServer(analysis);
    }
  }
  
  outputStats(analysis) {
    console.log('\n📊 Bundle Analysis Summary:');
    console.log('─'.repeat(50));
    console.log(`Total Size: ${this.formatSize(analysis.sizeStats.total)}`);
    console.log(`Build Time: ${analysis.buildInfo.time}ms`);
    console.log(`Modules: ${analysis.modules.length}`);
    console.log(`Chunks: ${analysis.chunks.length}`);
    console.log('\n📦 Size by Type:');
    
    for (const [type, stats] of Object.entries(analysis.sizeStats.byType)) {
      const percentage = ((stats.size / analysis.sizeStats.total) * 100).toFixed(1);
      console.log(`  .${type}: ${this.formatSize(stats.size)} (${percentage}%)`);
    }
    
    console.log('\n🔍 Largest Assets:');
    for (const asset of analysis.sizeStats.largest.slice(0, 5)) {
      console.log(`  ${asset.name}: ${this.formatSize(asset.size)}`);
    }
    
    console.log('─'.repeat(50) + '\n');
  }
  
  formatSize(bytes) {
    if (bytes < 1024) return `${bytes} B`;
    if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(2)} KB`;
    if (bytes < 1024 * 1024 * 1024) return `${(bytes / (1024 * 1024)).toFixed(2)} MB`;
    return `${(bytes / (1024 * 1024 * 1024)).toFixed(2)} GB`;
  }
  
  log(level, message) {
    if (this.options.logLevel === 'silent') return;
    
    const prefix = level === 'error' ? '❌' : level === 'warn' ? '⚠️' : 'ℹ️';
    console.log(`${prefix} [BundleAnalyzer] ${message}`);
  }
  
  startAnalyzerServer(analysis) {
    // 简化的分析服务器实现
    const express = require('express');
    const app = express();
    
    app.get('/analysis', (req, res) => {
      res.json(analysis);
    });
    
    app.get('/bundle-stats.json', (req, res) => {
      res.json(analysis);
    });
    
    this.server = app.listen(this.analyzerPort, () => {
      this.log('info', `Analyzer server running at http://localhost:${this.analyzerPort}`);
      
      if (this.options.openAnalyzer) {
        const { exec } = require('child_process');
        exec(`open http://localhost:${this.analyzerPort}`);
      }
    });
  }
}

module.exports = BundleAnalyzerPlugin;
```

---

## 4. 常用命令与操作

### 4.1 Turbopack 命令行操作

#### 4.1.1 开发模式

Turbopack 在开发模式下提供极速的启动速度和热更新：

```bash
# Next.js 项目启动开发服务器
npx next dev --turbopack

# 或简写
npx next dev -t

# 指定端口
npx next dev --turbopack --port 3001

# 指定主机（允许外部访问）
npx next dev --turbopack --hostname 0.0.0.0

# 启用 Turbopack 日志
TURBOPACK_LOG_LEVEL=verbose npx next dev --turbopack

# 增量构建日志
TURBOPACK_DEBUG=1 npx next dev --turbopack
```

#### 4.1.2 生产构建

```bash
# 生产环境构建
npx next build

# 构建并分析输出
ANALYZE=true npx next build

# 静态导出（SSG）
npx next build && npx next export

# 增量构建（只构建变更部分）
npx next build --no-lint
```

#### 4.1.3 Turbopack 独立 CLI 命令

```bash
# 初始化项目
npx @vercel/turbopack init

# 开发服务器
npx turbopack dev

# 生产构建
npx turbopack build

# 类型检查
npx turbopack typecheck

# Lint 检查
npx turbopack lint
```

### 4.2 Rspack 命令行操作

#### 4.2.1 Rspack CLI 基本命令

```bash
# 初始化项目
npx @rspack/cli init

# 开发服务器
npx rspack serve

# 简写
npx rspack dev

# 生产构建
npx rspack build

# 简写
npx rspack
```

#### 4.2.2 命令行选项

```bash
# 指定配置文件
npx rspack build --config rspack.config.js

# 指定入口文件
npx rspack build --entry ./src/index.tsx

# 指定输出目录
npx rspack build --output-path ./dist

# 指定环境模式
npx rspack build --mode production
npx rspack build --mode development

# 启用 Source Map
npx rspack build --devtool source-map

# 监听模式（自动重新构建）
npx rspack build --watch

# 分析构建产物
npx rspack build --analyze

# 启用内联运行时（禁用文件分离）
npx rspack build --no-split-chunks

# 指定目标平台
npx rspack build --target node
npx rspack build --target web

# 统计输出
npx rspack build --stats verbose

# 帮助信息
npx rspack --help
```

#### 4.2.3 package.json 脚本配置

```json
{
  "scripts": {
    "dev": "rspack serve",
    "build": "rspack build",
    "build:analyze": "cross-env ANALYZE=true rspack build",
    "build:staging": "cross-env NODE_ENV=staging rspack build",
    "build:prod": "cross-env NODE_ENV=production rspack build",
    "preview": "serve dist -p 3000",
    "typecheck": "tsc --noEmit",
    "lint": "eslint src --ext .ts,.tsx",
    "format": "prettier --write \"src/**/*.{ts,tsx}\""
  }
}
```

#### 4.2.4 开发服务器配置

```bash
# 指定端口
npx rspack serve --port 3001

# 启用 HTTPS
npx rspack serve --https

# 指定主机
npx rspack serve --host 0.0.0.0

# 启用热模块替换
npx rspack serve --hot

# 打开浏览器
npx rspack serve --open

# 指定配置文件
npx rspack serve --config rspack.config.js
```

---

## 5. 高级配置与技巧

### 5.1 Turbopack 高级配置

#### 5.1.1 环境隔离配置

Turbopack 支持为不同目标平台生成隔离的 bundle：

```javascript
// turbopack.config.js
module.exports = {
  targets: [
    {
      platform: 'browser',
      entry: './src/browser-entry.ts',
      output: {
        path: './dist/browser',
      },
      define: {
        'process.env.PLATFORM': JSON.stringify('browser'),
      },
    },
    {
      platform: 'server',
      entry: './src/server-entry.ts',
      output: {
        path: './dist/server',
      },
      define: {
        'process.env.PLATFORM': JSON.stringify('server'),
      },
    },
    {
      platform: 'edge',
      entry: './src/edge-entry.ts',
      output: {
        path: './dist/edge',
      },
      define: {
        'process.env.PLATFORM': JSON.stringify('edge'),
      },
    },
  ],
};
```

#### 5.1.2 实验性功能

```javascript
// turbopack.config.js
module.exports = {
  experimental: {
    // Rust 原生模块解析
    nativeModuleResolution: true,
    
    // 增量编译
    incrementalCompilation: true,
    
    // 模块联邦
    moduleFederation: {
      tsHostConfig: {
        resolveRemote: true,
      },
    },
    
    // 持久化缓存
    persistentCaching: {
      backend: 'filesystem',
      cacheDir: '.turbo/cache',
    },
    
    // 打包分析
    bundleAnalysis: {
      enabled: process.env.ANALYZE === 'true',
      outputPath: './.turbo/analysis',
    },
    
    // CSS Modules
    cssModules: {
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
  },
};
```

#### 5.1.3 自定义 Loader

```javascript
// turbopack.config.js
module.exports = {
  rules: [
    {
      test: /\.custom$/,
      loaders: [
        {
          loader: 'raw-loader',
        },
        {
          loader: 'custom-transform-loader',
          options: {
            transform: 'base64',
          },
        },
      ],
    },
  ],
};
```

### 5.2 Rspack 高级配置

#### 5.2.1 代码分割高级配置

```javascript
// rspack.config.js - 高级代码分割
module.exports = {
  optimization: {
    splitChunks: {
      // 全局配置
      chunks: 'all',
      minSize: 20000,
      maxSize: 250000,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      enforceSizeThreshold: 50000,
      
      // 缓存组配置
      cacheGroups: {
        // 默认 vendors 组
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10,
          reuseExistingChunk: true,
          name(module) {
            const packageName = module.context.match(
              /[\\/]node_modules[\\/](.*?)([\\/]|$)/
            )?.[1];
            if (!packageName) return false;
            
            // 按包名分割
            if (['react', 'react-dom', 'react-router-dom'].includes(packageName)) {
              return 'react-vendor';
            }
            if (['lodash', 'lodash-es'].some(p => packageName.includes(p))) {
              return 'lodash-vendor';
            }
            if (packageName.startsWith('@mui') || packageName.startsWith('@material')) {
              return 'mui-vendor';
            }
            
            // 其他 node_modules 包
            return `vendor-${packageName.slice(0, 3)}`;
          },
        },
        
        // React 生态系统
        reactEcosystem: {
          test: /[\\/]node_modules[\\/](react|react-dom|react-router|scheduler)[\\/]/,
          name: 'react-vendor',
          chunks: 'all',
          priority: 20,
        },
        
        // UI 组件库
        uiLibrary: {
          test: /[\\/]node_modules[\\/](@mui|@ant-design|antd|chakra-ui)[\\/]/,
          name: 'ui-vendor',
          chunks: 'all',
          priority: 15,
        },
        
        // 工具函数库
        utilities: {
          test: /[\\/]node_modules[\\/](lodash|axios|dayjs|moment)[\\/]/,
          name: 'util-vendor',
          chunks: 'all',
          priority: 10,
        },
        
        // 公共模块
        common: {
          name: 'common',
          minChunks: 2,
          priority: -20,
          reuseExistingChunk: true,
        },
        
        // 低频代码
        lowFrequency: {
          test: /[\\/]node_modules[\\/](echarts|d3|three|@apollo)/,
          name: 'low-freq',
          chunks: 'async',
          priority: 5,
          enforce: true,
        },
      },
    },
    
    // 运行时 chunk
    runtimeChunk: {
      name: 'runtime',
    },
    
    // 模块 ID 策略
    moduleIds: 'deterministic',
    chunkIds: 'deterministic',
    
    // 压缩配置
    minimize: true,
    minimizer: [
      new rspack.SwcJsMinimizerRspackPlugin({
        format: {
          comments: false,
        },
        extractComments: false,
      }),
      new rspack.CssMinimizerRspackPlugin({
        minimizerOptions: {
          preset: [
            'default',
            {
              discardComments: { removeAll: true },
              normalizeWhitespace: true,
            },
          ],
        },
      }),
    ],
  },
};
```

#### 5.2.2 Module Federation 配置

```javascript
// rspack.config.js - Module Federation
const rspack = require('@rspack/core');
const { container } = require('@module-federation/rspack');

module.exports = {
  plugins: [
    new container.ModuleFederationPlugin({
      // 应用名称
      name: 'host_app',
      
      // 共享依赖
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
          eager: true,
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
          eager: true,
        },
        'react-router-dom': {
          singleton: true,
        },
      },
      
      // 远程模块配置
      remotes: {
        remote_app: 'remote_app@http://localhost:3001/remoteEntry.js',
        design_system: 'design_system@http://localhost:3002/remoteEntry.js',
      },
      
      // 暴露的模块
      exposes: {
        './App': './src/App',
        './utils': './src/utils',
      },
      
      // 文件名
      filename: 'remoteEntry.js',
      shareScope: 'default',
    }),
  ],
};
```

#### 5.2.3 环境变量注入

```javascript
// rspack.config.js - 环境变量配置
const rspack = require('@rspack/core');

module.exports = {
  plugins: [
    // 基础环境变量
    new rspack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(
        process.env.NODE_ENV || 'development'
      ),
      
      // 运行时变量
      'process.env.API_URL': JSON.stringify(
        process.env.API_URL || 'http://localhost:8000'
      ),
      
      // 构建时变量
      __VERSION__: JSON.stringify(process.env.npm_package_version),
      __BUILD_TIME__: JSON.stringify(new Date().toISOString()),
      
      // 特性开关
      'process.env.ENABLE_ANALYTICS': JSON.stringify(
        process.env.ENABLE_ANALYTICS === 'true'
      ),
      'process.env.ENABLE_DEBUG': JSON.stringify(
        process.env.ENABLE_DEBUG === 'true'
      ),
      
      // 条件编译
      __DEV__: process.env.NODE_ENV === 'development',
      __PROD__: process.env.NODE_ENV === 'production',
    }),
    
    // 构建时环境变量
    new rspack.EnvironmentPlugin({
      NODE_ENV: 'development',
      API_URL: '',
      DEBUG: false,
    }),
  ],
};
```

#### 5.2.4 性能优化配置

```javascript
// rspack.config.js - 性能优化
module.exports = {
  // 性能提示
  performance: {
    hints: 'warning',  // 'warning' | 'error' | false
    maxEntrypointSize: 400000,
    maxAssetSize: 300000,
    assetFilter: (assetFilename) => {
      return !assetFilename.endsWith('.map');
    },
  },
  
  // 缓存配置
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename],
    },
    cacheDirectory: '.rspack_cache',
    name: 'production',
    version: process.env.npm_package_version,
  },
  
  // 优化配置
  optimization: {
    // 移除空模块
    removeEmptyChunks: true,
    
    // 合并重复模块
    mergeDuplicateChunks: true,
    
    // 标识可处理的 chunk
    flagIncludedChunks: true,
    
    // 节点模块内联
    innerGraph: true,
    
    // 使用可复用 chunk
    usedExports: true,
    
    // 阶段标记
    providedExports: true,
    
    // 依赖项优化
    dependencyCollection: true,
  },
  
  // 快照配置
  snapshot: {
    mountPaths: {
      // 监视 node_modules 变化
      'node_modules': 'mtime',
    },
    modulePaths: {
      // 监视模块路径变化
      'src': 'hash',
    },
  },
};
```

#### 5.2.5 TypeScript 严格配置

```javascript
// rspack.config.js - TypeScript 配置
const rspack = require('@rspack/core');

module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
                decorators: true,
                dynamicImport: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                  development: process.env.NODE_ENV === 'development',
                  refresh: true,
                },
              },
              target: 'es2015',
              loose: false,
              externalHelpers: false,
              keepClassMembers: false,
            },
            env: {
              targets: 'defaults',
              mode: 'usage',
              coreJs: '3.30',
            },
          },
        },
        exclude: /node_modules/,
      },
    ],
  },
  
  // 类型声明文件
  typedAvail: true,
};
```

---

## 6. 与同类技术对比

### 6.1 Turbopack vs Rspack vs Webpack vs Vite

#### 6.1.1 核心指标对比

| 特性 | Turbopack | Rspack | Webpack 5 | Vite |
|------|-----------|--------|-----------|------|
| **编程语言** | Rust | Rust | JavaScript | Go (开发) + Rollup (生产) |
| **启动速度** | ~200ms | ~500ms | 10-60s | ~300ms |
| **热更新速度** | <100ms | ~200ms | 2-10s | ~50ms |
| **配置兼容性** | 独立生态 | 高度兼容 Webpack | 原有配置 | Vite 特定配置 |
| **插件生态** | 新兴 | 快速成长 | 成熟完善 | 丰富 |
| **生产构建** | 优化中 | 成熟稳定 | 成熟稳定 | 成熟稳定 |
| **学习曲线** | 低（新生态） | 中（兼容 Webpack） | 高（配置复杂） | 低（配置简洁） |
| **TypeScript 支持** | 内置 | SWC | 需额外配置 | esbuild/SWC |
| **社区活跃度** | 快速增长 | 活跃 | 非常活跃 | 活跃 |
| **生产验证** | 进行中 | 大规模验证 | 成熟 | 成熟 |

#### 6.1.2 启动速度对比

```
启动速度对比（中等规模项目 ~1000 模块）:

Turbopack:  ████ 200ms
Rspack:     ████████ 500ms
Vite:       ██████ 300ms (依赖预构建)
Webpack 5:  ████████████████████████████████ 45s
```

#### 6.1.3 热更新速度对比

```
热更新速度对比（单文件修改）:

Turbopack:  ██ <100ms
Rspack:     ████ ~200ms
Vite:       █ <50ms (HMR)
Webpack 5:  ██████████ 3s
```

#### 6.1.4 适用场景分析

**Turbopack 适用场景**：

- 使用 Next.js 15+ 的新项目
- 对启动速度有极致要求的场景
- 愿意拥抱新生态的团队
- 与 Vercel 平台深度集成的项目

**Rspack 适用场景**：

- 已有 Webpack 配置需要迁移的项目
- 需要生产级稳定性的企业项目
- 使用字节跳动内部组件库的项目
- 需要复用现有 Webpack 插件的项目

**Vite 适用场景**：

- Vue 3 + Vite 生态项目
- 需要极佳开发体验的项目
- 追求配置简洁性的团队
- 小型到中型项目

**Webpack 5 适用场景**：

- 依赖复杂 Webpack 插件生态的项目
- 需要精细控制构建过程的场景
- 有大量 Webpack 配置积累的团队
- 超大型复杂项目（需要时间优化）

### 6.2 技术选型决策树

```
构建工具选型决策树:

项目是否使用 Next.js?
│
├─ 是 → Next.js 15+?
│        │
│        ├─ 是 → 使用 Turbopack（--turbopack）
│        └─ 否 → 使用 Rspack 迁移
│
└─ 否 → 是否有现有 Webpack 配置?
         │
         ├─ 是 → 是否需要复用现有插件?
                  │
                  ├─ 是 → 评估 Rspack 兼容性 → 可行则用 Rspack
                  └─ 否 → 评估迁移成本 → 高则保留 Webpack，低则用 Rspack
                  │
         └─ 否 → 项目规模?
                  │
                  ├─ 小型 → Vite
                  ├─ 中型 → Vite 或 Rspack
                  └─ 大型 → Rspack 或 Webpack 5
```

---

## 7. 常见问题与解决方案

### 7.1 Turbopack 常见问题

#### 7.1.1 不支持的特性

**问题**：某些 Webpack 特性在 Turbopack 中尚未支持。

**解决方案**：

```javascript
// 检查不支持的特性
// Turbopack 会输出警告
// 例如：某些 loader 和 plugin

// 替代方案
// 1. 使用 Next.js 提供的内置支持
// 2. 等待 Turbopack 迭代
// 3. 使用自定义 loader 包装

// 示例：使用 SWC 替代 Babel
// turbopack.config.js
module.exports = {
  experimental: {
    resolveAlias: {
      // 替代 webpack-alias
    },
  },
};
```

#### 7.1.2 Node.js 兼容性问题

**问题**：某些 Node.js API 在 Turbopack 中不可用。

**解决方案**：

```javascript
// 使用 Node.js polyfill 配置
module.exports = {
  experimental: {
    nodeProtocolInTurbopack: true,
  },
  resolve: {
    fallback: {
      // Webpack 风格的 fallback
      fs: false,
      path: false,
      crypto: false,
    },
  },
};
```

#### 7.1.3 TypeScript 类型检查

**问题**：Turbopack 不执行 TypeScript 类型检查。

**解决方案**：

```bash
# 单独运行 TypeScript 检查
npx tsc --noEmit

# 或在 package.json 中添加
{
  "scripts": {
    "typecheck": "tsc --noEmit && next build"
  }
}
```

### 7.2 Rspack 常见问题

#### 7.2.1 Webpack 插件兼容性问题

**问题**：某些 Webpack 插件在 Rspack 中不工作。

**解决方案**：

```javascript
// 检查 Rspack 兼容性列表
// https://rspress.dev/guide/extra/compatible-plugins

// 使用 Rspack 替代插件
// webpack.config.js → rspack.config.js
// HtmlWebpackPlugin → rspack.HtmlRspackPlugin
// MiniCssExtractPlugin → rspack.MinimizerRspackPlugin

// 示例配置
const rspack = require('@rspack/core');

module.exports = {
  plugins: [
    // 使用 Rspack 版本
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
    }),
    
    // CSS 提取
    new rspack.MinimizerRspackPlugin({
      minimizerOptions: {
        cssMinimizerOptions: {
          minimizerOptions: {
            preset: ['default', { discardComments: { removeAll: true } }],
          },
        },
      },
    }),
  ],
};
```

#### 7.2.2 SWC 配置问题

**问题**：SWC 转换结果与 Babel 不一致。

**解决方案**：

```javascript
// rspack.config.js - 调整 SWC 配置
module.exports = {
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                  development: false,
                  refresh: false,
                },
              },
              // 确保与 Babel 一致的输出
              loose: false,
              externalHelpers: false,
              // 目标环境
              target: 'es2015',
            },
            env: {
              targets: {
                chrome: '80',
                firefox: '75',
                safari: '13',
                edge: '80',
                ie: '11',
              },
              mode: 'entry',
              coreJs: '3.30',
            },
          },
        },
      },
    ],
  },
};
```

#### 7.2.3 性能问题排查

**问题**：Rspack 构建速度变慢。

**解决方案**：

```javascript
// rspack.config.js - 性能优化
module.exports = {
  // 启用缓存
  cache: {
    type: 'filesystem',
    cacheDirectory: '.rspack_cache',
  },
  
  // 优化 resolve
  resolve: {
    extensions: ['.tsx', '.ts', '.jsx', '.js', '.json'],  // 常用放前
    modules: ['node_modules'],
    symlinks: false,  // 如果没有符号链接
  },
  
  // 排除不需要处理的模块
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        exclude: /node_modules/,
        use: 'builtin:swc-loader',
      },
    ],
  },
  
  // 优化输出
  output: {
    pathinfo: false,  // 生产环境关闭
  },
  
  optimization: {
    removeAvailableModules: false,  // 如果不需要
    removeEmptyChunks: true,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          chunks: 'all',
        },
      },
    },
  },
};
```

#### 7.2.4 构建产物问题

**问题**：生产构建产物大小异常。

**解决方案**：

```bash
# 分析构建产物
npx rspack build --analyze

# 或使用 Bundle Analyzer
npm install @rspack/bundle-analyzer -D
```

```javascript
// rspack.config.js
const { BundleAnalyzerPlugin } = require('@rspack/bundle-analyzer');

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      reportFilename: 'bundle-report.html',
      openAnalyzer: false,
    }),
  ],
};
```

### 7.3 迁移常见问题

#### 7.3.1 Webpack 到 Rspack 迁移

**常见问题清单**：

| 问题类型 | 常见问题 | 解决方案 |
|---------|---------|---------|
| 插件不兼容 | 某些 webpack plugin 无法使用 | 使用 Rspack 对应插件或寻找替代 |
| Loader 配置 | babel-loader 配置迁移 | 改用 `builtin:swc-loader` |
| 模块解析 | resolve 配置差异 | 调整 resolve 配置 |
| 优化配置 | optimization 不生效 | 确认 Rspack 支持的选项 |
| CSS 处理 | CSS Modules 配置 | 使用 Rspack 的 CSS Modules 支持 |

**迁移检查清单**：

```bash
# 1. 备份现有配置
cp webpack.config.js webpack.config.backup.js

# 2. 创建 Rspack 配置
cp webpack.config.js rspack.config.js

# 3. 修改入口
# webpack.config.js
# entry: './src/index.js'
# rspack.config.js
const rspack = require('@rspack/core');
# entry: './src/index.js'

# 4. 替换插件
# webpack.config.js
# plugins: [new HtmlWebpackPlugin({ ... })]
# rspack.config.js
plugins: [new rspack.HtmlRspackPlugin({ ... })]

# 5. 替换 Loader
# webpack.config.js
# rules: [{ test: /\.tsx?$/, use: 'babel-loader' }]
# rspack.config.js
rules: [{ test: /\.tsx?$/, use: 'builtin:swc-loader' }]

# 6. 测试运行
npx rspack build

# 7. 验证输出
# 检查 dist 目录和 bundle 大小
```

---

## 8. 实战项目示例

### 8.1 Turbopack 实战：Next.js 15 企业级项目

#### 8.1.1 项目结构

```
enterprise-app/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   ├── dashboard/
│   │   └── page.tsx
│   └── settings/
│       └── page.tsx
├── components/
│   ├── ui/
│   │   ├── Button.tsx
│   │   ├── Input.tsx
│   │   └── Card.tsx
│   └── charts/
│       └── LineChart.tsx
├── lib/
│   ├── api.ts
│   ├── auth.ts
│   └── utils.ts
├── hooks/
│   ├── useAuth.ts
│   └── useData.ts
├── styles/
│   └── globals.css
├── public/
│   └── images/
├── package.json
├── tsconfig.json
└── next.config.js
```

#### 8.1.2 配置文件

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Turbopack 配置
  experimental: {
    // Turbopack 特定配置
    turbo: {
      // 静态资源路径
      staticPageExtensions: ['js', 'jsx', 'ts', 'tsx', 'md', 'mdx'],
      
      // 打包分析
      bundleAnalysis: {
        enabled: process.env.ANALYZE === 'true',
      },
      
      // 图片优化
      images: {
        formats: ['image/avif', 'image/webp'],
      },
    },
  },
  
  // 图片配置
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
    ],
  },
  
  // 压缩配置
  compress: true,
  
  // 头部配置
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
        ],
      },
    ];
  },
};

module.exports = nextConfig;
```

#### 8.1.3 package.json

```json
{
  "name": "enterprise-app",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "typecheck": "tsc --noEmit",
    "analyze": "ANALYZE=true npm run build"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "lucide-react": "^0.400.0",
    "recharts": "^2.12.0",
    "date-fns": "^3.6.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "typescript": "^5.5.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@types/node": "^20.14.0",
    "eslint": "^8.57.0",
    "eslint-config-next": "^15.0.0"
  }
}
```

#### 8.1.4 组件示例

```typescript
// components/ui/Button.tsx
'use client';

import { forwardRef, ButtonHTMLAttributes } from 'react';
import styles from './Button.module.css';

type ButtonVariant = 'primary' | 'secondary' | 'outline' | 'ghost';
type ButtonSize = 'sm' | 'md' | 'lg';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: ButtonVariant;
  size?: ButtonSize;
  loading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  (
    {
      children,
      variant = 'primary',
      size = 'md',
      loading = false,
      disabled,
      className,
      ...props
    },
    ref
  ) => {
    return (
      <button
        ref={ref}
        className={`${styles.button} ${styles[variant]} ${styles[size]} ${className || ''}`}
        disabled={disabled || loading}
        {...props}
      >
        {loading && <span className={styles.spinner} />}
        <span className={loading ? styles.hiddenText : ''}>{children}</span>
      </button>
    );
  }
);

Button.displayName = 'Button';
```

### 8.2 Rspack 实战：React 企业级项目

#### 8.2.1 项目结构

```
rspack-enterprise/
├── src/
│   ├── index.tsx
│   ├── App.tsx
│   ├── components/
│   │   ├── common/
│   │   │   ├── Button/
│   │   │   ├── Modal/
│   │   │   └── Table/
│   │   └── business/
│   │       ├── Dashboard/
│   │       └── UserList/
│   ├── pages/
│   │   ├── Home.tsx
│   │   ├── About.tsx
│   │   └── NotFound.tsx
│   ├── hooks/
│   │   └── useAsync.ts
│   ├── utils/
│   │   └── format.ts
│   └── styles/
│       ├── global.css
│       └── variables.scss
├── public/
│   └── index.html
├── package.json
├── tsconfig.json
├── rspack.config.js
└── postcss.config.js
```

#### 8.2.2 完整 Rspack 配置

```javascript
// rspack.config.js
const rspack = require('@rspack/core');
const path = require('path');
const { VueLoaderPlugin } = require('vue-loader');

const isDev = process.env.NODE_ENV === 'development';
const isProd = process.env.NODE_ENV === 'production';

/** @type {import('@rspack/cli').Configuration} */
module.exports = {
  // 入口配置
  entry: {
    main: './src/index.tsx',
    // 可添加多个入口
    // admin: './src/admin.tsx',
  },

  // 输出配置
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: isProd ? '[name].[contenthash].js' : '[name].js',
    chunkFilename: isProd ? '[name].[contenthash].chunk.js' : '[name].chunk.js',
    assetModuleFilename: 'assets/[name].[hash][ext]',
    clean: isProd,
    publicPath: '/',
    pathinfo: isDev,
  },

  // 解析配置
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx', '.json', '.vue'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@components': path.resolve(__dirname, 'src/components'),
      '@pages': path.resolve(__dirname, 'src/pages'),
      '@hooks': path.resolve(__dirname, 'src/hooks'),
      '@utils': path.resolve(__dirname, 'src/utils'),
      '@styles': path.resolve(__dirname, 'src/styles'),
    },
    symlinks: false,
  },

  // 模块配置
  module: {
    rules: [
      // TypeScript 和 JSX
      {
        test: /\.tsx?$/,
        use: {
          loader: 'builtin:swc-loader',
          options: {
            jsc: {
              parser: {
                syntax: 'typescript',
                tsx: true,
                decorators: true,
                dynamicImport: true,
              },
              transform: {
                react: {
                  runtime: 'automatic',
                  development: isDev,
                  refresh: isDev,
                },
              },
              externalHelpers: false,
              target: 'es2015',
            },
            env: {
              targets: {
                chrome: '90',
                firefox: '90',
                safari: '14',
                edge: '90',
              },
              mode: 'usage',
              coreJs: '3.30',
            },
          },
        },
        exclude: /node_modules/,
      },

      // Vue
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          experimentalInlineMatchResource: true,
        },
      },

      // CSS
      {
        test: /\.css$/,
        type: 'css',
        use: [
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  require('tailwindcss'),
                  require('autoprefixer'),
                  ...(isProd ? [require('cssnano')] : []),
                ],
              },
            },
          },
        ],
      },

      // SCSS
      {
        test: /\.scss$/,
        type: 'css',
        use: [
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [require('autoprefixer')],
              },
            },
          },
          {
            loader: 'sass-loader',
            options: {
              additionalData: `@import "@/styles/variables.scss";`,
            },
          },
        ],
      },

      // 图片
      {
        test: /\.(png|jpe?g|gif|webp|avif)$/i,
        type: 'asset',
        parser: {
          dataUrlCondition: {
            maxSize: 8 * 1024, // 8kb
          },
        },
        generator: {
          filename: 'images/[name].[hash][ext]',
        },
      },

      // SVG
      {
        test: /\.svg$/i,
        type: 'asset',
        resourceQuery: { not: [/component/] },
        generator: {
          filename: 'icons/[name].[hash][ext]',
        },
      },
      {
        test: /\.svg$/i,
        type: 'asset/resource',
        resourceQuery: /component/,
        generator: {
          filename: 'icons/[name].[hash][ext]',
        },
      },

      // 字体
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'fonts/[name].[hash][ext]',
        },
      },

      // JSON
      {
        test: /\.json$/i,
        type: 'json',
      },

      // CSV/TSV
      {
        test: /\.(csv|tsv)$/i,
        type: 'asset/source',
      },

      // XML
      {
        test: /\.xml$/i,
        type: 'asset/source',
      },

      // Markdown
      {
        test: /\.md$/i,
        type: 'asset/source',
      },

      // 其他资源
      {
        test: /\.(pdf|doc|docx|xls|xlsx|ppt|pptx)$/i,
        type: 'asset/resource',
        generator: {
          filename: 'documents/[name].[hash][ext]',
        },
      },
    ],
  },

  // 插件配置
  plugins: [
    new rspack.HtmlRspackPlugin({
      template: './public/index.html',
      filename: 'index.html',
      inject: 'body',
      scriptLoading: 'defer',
      minify: isProd ? {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true,
        minifyCSS: true,
        minifyJS: true,
      } : false,
    }),

    new rspack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
      'process.env.API_URL': JSON.stringify(process.env.API_URL || 'http://localhost:8000'),
      'process.env.APP_VERSION': JSON.stringify(process.env.npm_package_version || '1.0.0'),
    }),

    // Vue Loader 插件
    new VueLoaderPlugin(),

    // 进度插件
    isDev && new rspack.ProgressPlugin({ handler: (percentage, message) => {
      console.log(`${(percentage * 100).toFixed(0)}% ${message}`);
    }}),

    // 清理输出目录
    isProd && new rspack.CleanPlugin({
      dry: false,
      cleanStaleWebpackAssets: true,
      protectWebpackAssets: false,
    }),
  ].filter(Boolean),

  // 优化配置
  optimization: {
    minimize: isProd,
    minimizer: [
      new rspack.SwcJsMinimizerRspackPlugin({
        format: {
          comments: false,
        },
        extractComments: false,
        minimizerOptions: {
          compress: {
            passes: 2,
            dropConsole: isProd && process.env.DROP_CONSOLE === 'true',
            pure_funcs: ['console.log', 'console.debug'],
          },
          mangle: {
            safari10: true,
          },
        },
      }),
      new rspack.CssMinimizerRspackPlugin({
        minimizerOptions: {
          preset: ['default', {
            discardComments: { removeAll: true },
            minifyFontValues: { removeQuotes: false },
          }],
        },
      }),
    ],
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 250000,
      minChunks: 1,
      maxAsyncRequests: 30,
      maxInitialRequests: 30,
      cacheGroups: {
        reactVendor: {
          test: /[\\/]node_modules[\\/](react|react-dom|scheduler)[\\/]/,
          name: 'react-vendor',
          chunks: 'all',
          priority: 30,
        },
        utilsVendor: {
          test: /[\\/]node_modules[\\/](lodash|axios|dayjs|query-string)[\\/]/,
          name: 'utils-vendor',
          chunks: 'all',
          priority: 20,
        },
        uiVendor: {
          test: /[\\/]node_modules[\\/]@(antd|ant-design|mui|@mui)[\\/]/,
          name: 'ui-vendor',
          chunks: 'all',
          priority: 25,
        },
        chartsVendor: {
          test: /[\\/]node_modules[\\/](recharts|echarts|d3|chart\.js)[\\/]/,
          name: 'charts-vendor',
          chunks: 'all',
          priority: 15,
          enforce: true,
        },
        common: {
          name: 'common',
          minChunks: 2,
          chunks: 'all',
          priority: 10,
          reuseExistingChunk: true,
        },
      },
    },
    runtimeChunk: 'single',
    moduleIds: 'deterministic',
    chunkIds: 'deterministic',
    usedExports: true,
    sideEffects: true,
    providedExports: true,
  },

  // 开发服务器
  devServer: {
    port: 3000,
    host: '0.0.0.0',
    hot: true,
    compress: true,
    historyApiFallback: true,
    static: {
      directory: path.join(__dirname, 'public'),
      watch: true,
    },
    client: {
      overlay: {
        errors: true,
        warnings: false,
        runtimeErrors: (error) => {
          if (error.message.includes('ResizeObserver')) {
            return false;
          }
          return true;
        },
      },
      progress: true,
    },
    open: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
        pathRewrite: { '^/api': '' },
      },
    },
  },

  // 性能提示
  performance: {
    hints: isProd ? 'warning' : false,
    maxEntrypointSize: 400000,
    maxAssetSize: 300000,
  },

  // 缓存
  cache: {
    type: 'filesystem',
    buildDependencies: {
      config: [__filename],
    },
    cacheDirectory: path.resolve(__dirname, '.rspack_cache'),
    compression: 'gzip',
  },

  // 快照
  snapshot: {
    managedPaths: [path.resolve(__dirname, 'node_modules')],
    immutablePaths: [],
  },

  // Devtool
  devtool: isDev ? 'eval-source-map' : 'source-map',

  // 统计信息
  stats: {
    assets: true,
    colors: true,
    errors: true,
    errorDetails: true,
    hash: false,
    modules: false,
    performance: true,
    timings: true,
    warnings: true,
  },
};
```

#### 8.2.3 package.json

```json
{
  "name": "rspack-enterprise",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "rspack serve",
    "build": "rspack build --mode production",
    "build:dev": "rspack build --mode development",
    "preview": "npx serve dist -p 3000",
    "typecheck": "tsc --noEmit",
    "lint": "eslint src --ext .ts,.tsx --fix",
    "analyze": "cross-env ANALYZE=true rspack build"
  },
  "dependencies": {
    "react": "^18.3.0",
    "react-dom": "^18.3.0",
    "react-router-dom": "^6.23.0",
    "lodash-es": "^4.17.21",
    "axios": "^1.7.0",
    "dayjs": "^1.11.11",
    "recharts": "^2.12.7",
    "antd": "^5.17.0"
  },
  "devDependencies": {
    "@rspack/cli": "^0.6.0",
    "@rspack/core": "^0.6.0",
    "@rspack/bundle-analyzer": "^0.6.0",
    "@types/react": "^18.3.0",
    "@types/react-dom": "^18.3.0",
    "@types/lodash-es": "^4.17.12",
    "typescript": "^5.5.0",
    "postcss": "^8.4.38",
    "postcss-loader": "^8.1.1",
    "sass": "^1.77.0",
    "sass-loader": "^14.2.0",
    "tailwindcss": "^3.4.3",
    "autoprefixer": "^10.4.19",
    "cssnano": "^7.0.0",
    "vue": "^3.4.27",
    "vue-loader": "^17.4.2",
    "eslint": "^8.57.0",
    "cross-env": "^7.0.3",
    "serve": "^14.2.3"
  }
}
```

---

> [!TIP]
> **选型建议**：新项目使用 Next.js 15 + Turbopack；需要迁移 Webpack 项目时选择 Rspack 作为过渡方案。两者都是 Rust 原生构建工具，能提供 10 倍于传统 Webpack 的构建速度。
