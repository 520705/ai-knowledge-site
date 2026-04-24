# PostCSS 与 CSS 预处理完全指南

date: 2026-04-24

> [!NOTE]
> 本文档涵盖 PostCSS 插件生态、Sass/SCSS 预处理器、CSS Modules、Stylelint 及 CSS 工程化最佳实践，是构建现代化 CSS 工作流的完整指南。PostCSS 不仅是 CSS 的转换工具，更是现代 CSS 工程的基石。

---

## 技术概述与定位

### CSS 工程化的演进历程

在前端开发的历史长河中，CSS 的工程化经历了漫长的演进过程。早期的 CSS 几乎是纯手写的简单样式表，开发者直接在 HTML 文件中嵌入 `<style>` 标签，或者创建几个简单的 `.css` 文件。随着 Web 应用复杂度的提升，这种简单模式很快暴露出了问题：代码复用困难、命名冲突严重、维护成本高昂、缺乏现代编程语言的表达能力。

CSS 预处理器（如 Sass、LESS、Stylus）的出现标志着 CSS 工程化的第一次重大飞跃。Sass（最初称为 Syntactically Awesome StyleSheets）于 2006 年诞生，它引入了变量、嵌套、混合器（Mixin）、继承等编程概念，极大地提升了 CSS 的可维护性。这些工具将类似 CSS 的语法编译成标准的 CSS 代码，让开发者能够用更高级的语法编写样式。

PostCSS 的诞生则代表了 CSS 工程化的第二次重大突破。与预处理器不同，PostCSS 并不是一种新的语言或语法，而是一个用 JavaScript 插件转换 CSS 的工具链。PostCSS 的核心是一个解析器，它能够将 CSS 代码解析成抽象语法树（AST），然后允许开发者编写插件来转换这个语法树。这种设计使得 PostCSS 具有极高的灵活性和扩展性，可以实现从自动添加浏览器前缀到完整的 CSS 新语法转换。

现代 CSS 工程的典型工作流是：开发者使用 Sass/SCSS 编写样式（享受变量、嵌套、混合器等便利），预处理器将 SCSS 编译成标准 CSS，PostCSS 接收标准 CSS 并通过各种插件进行转换（如添加前缀、优化代码、转换新语法），最终生成浏览器可识别的优化 CSS。这种分工明确的工作流让每种工具专注于自己的强项。

### PostCSS 的技术定位

PostCSS 的定位非常明确：成为 CSS 转换的「瑞士军刀」。它的设计哲学是「做一件事并做到极致」，通过组合不同的插件来完成各种 CSS 处理任务。

```
PostCSS 技术定位图:

┌─────────────────────────────────────────────────────────────────┐
│                       PostCSS 生态系统                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                    输入层                               │    │
│   │  Standard CSS | Sass(通过预处理器) | Less | Stylus   │    │
│   └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                   PostCSS 核心                           │    │
│   │  ┌─────────┐  ┌─────────┐  ┌─────────┐                │    │
│   │  │ Parser  │→ │ AST     │→ │ Stringifier│               │    │
│   │  └─────────┘  └─────────┘  └─────────┘                │    │
│   └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                    插件层                               │    │
│   │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐   │    │
│   │  │Autoprefixer│ │ postcss-preset-env │ cssnano │Modules│   │    │
│   │  └──────────┘ └──────────┘ └──────────┘ └──────────┘   │    │
│   └─────────────────────────────────────────────────────────┘    │
│                              │                                   │
│                              ▼                                   │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                    输出层                               │    │
│   │              Standard CSS (优化后)                      │    │
│   └─────────────────────────────────────────────────────────┘    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

PostCSS 插件可以分为几大类。第一类是「前缀与兼容性」类，如 autoprefixer 自动添加浏览器私有前缀，确保 CSS 特性在各种浏览器中正常工作。第二类是「新语法转换」类，如 postcss-preset-env 将尚未成为标准的 CSS 语法转换为当前可用的语法，让你提前使用未来的 CSS 特性。第三类是「优化压缩」类，如 cssnano 通过多种优化策略减小 CSS 文件体积。第四类是「模块化」类，如 postcss-modules 实现 CSS 类名的局部作用域，避免命名冲突。

### 预处理器与 PostCSS 的关系

很多开发者会产生一个疑问：既然有了 Sass/LESS 等预处理器，为什么还需要 PostCSS？实际上，这两者并不是互斥的，而是可以协同工作的。

预处理器（如 Sass）解决的是「编写体验」的问题，它们让开发者能够用更方便、更强大的语法来编写样式。而 PostCSS 解决的是「转换输出」的问题，它在预处理器输出标准 CSS 之后，对 CSS 进行进一步的处理和优化。

一个典型的现代化 CSS 工作流是这样的。首先是编写阶段，开发者使用 Sass/LESS 等预处理器语法编写样式，享受变量、嵌套、混合器等便利特性。然后是编译阶段，预处理器将 SCSS/LESS 编译成标准 CSS。接着是转换阶段，PostCSS 接收标准 CSS，通过各种插件进行转换，如添加前缀、优化代码、转换新语法等。最后是输出阶段，生成浏览器可识别的 CSS 文件。

```
CSS 处理工具协作流程:

  编写阶段              编译阶段              转换阶段              输出阶段
  
┌──────────┐        ┌──────────┐        ┌──────────┐        ┌──────────┐
│ Sass/SCSS│   →   │  Sass    │   →   │ PostCSS  │   →   │ 标准 CSS │
│  源代码   │        │  编译    │        │  插件链   │        │  最终文件 │
└──────────┘        └──────────┘        └──────────┘        └──────────┘
                                                    │
                          ┌─────────────────────────┼─────────────────────────┐
                          │                         │                         │
                          ▼                         ▼                         ▼
                    ┌──────────┐           ┌──────────┐           ┌──────────┐
                    │Autoprefixer│         │preset-env│          │  cssnano │
                    │  前缀    │           │  新语法   │          │   压缩   │
                    └──────────┘           └──────────┘           └──────────┘
```

### CSS 预处理器对比

| 特性 | Sass (SCSS) | LESS | Stylus |
|------|-------------|------|--------|
| **语法风格** | 类 CSS（SCSS）/缩进（Sass） | 类 CSS | 灵活（可省略冒号/大括号/分号） |
| **变量符号** | `$variable` | `@variable` | `variable` |
| **混合器语法** | `@mixin` / `@include` | `.class()` / `.mixin()` | `mixin()` |
| **继承** | `@extend` | 无原生支持 | `extends` |
| **条件语句** | `@if` / `@else` | JavaScript 评估 | `@if` |
| **循环** | `@for` / `@each` / `@while` | JavaScript 评估 | `@for` |
| **社区生态** | 非常大 | 一般 | 较小 |
| **学习曲线** | 低 | 低 | 中等 |
| **编译速度** | 快（dart-sass） | 较快 | 快 |

Sass 是目前最流行的预处理器，它有两大语法变体：SCSS（Sassy CSS）使用类似 CSS 的语法，带有大括号和分号，学习成本最低；Sass（缩进语法）使用缩进表示嵌套，不需要大括号，代码更简洁。两种语法可以混用，编译器会自动识别。Dart Sass 是官方推荐的实现，使用 Dart 语言编写，编译速度快且持续维护中。LibSass 是 Sass 的 C/C++ 实现，曾经因为速度优势流行，但目前已停止维护，新项目不推荐使用。

---

## 完整安装与配置

### PostCSS 安装与配置

#### 基础安装

PostCSS 可以作为独立工具使用，也可以集成到各种构建工具中。独立使用方式适合简单的样式处理或学习研究，生产项目通常集成到 Vite、Webpack 等构建工具中。

```bash
# 安装 PostCSS CLI（命令行工具）
npm install postcss postcss-cli -D

# 安装常用插件
npm install autoprefixer postcss-preset-env cssnano -D
```

PostCSS 的配置文件可以是 `postcss.config.js`、`postcss.config.cjs`、`.postcssrc` 或 `.postcssrc.json`。推荐使用 `postcss.config.js`（或 `.cjs` 如果使用 CommonJS 模块）。

```javascript
// postcss.config.js
module.exports = {
  syntax: 'postcss-scss',  // 使用 SCSS 语法解析器
  parser: 'postcss-scss',
  plugins: [
    // 1. Tailwind CSS（如果使用）
    require('tailwindcss'),

    // 2. Autoprefixer - 自动添加浏览器前缀
    require('autoprefixer')({
      // 目标浏览器配置
      overrideBrowserslist: [
        '> 1%',               // 全球使用率 > 1% 的浏览器
        'last 2 versions',    // 最新两个版本
        'not dead',           // 仍在维护的浏览器
        'not IE 11',          // 排除 IE 11
      ],
      // Grid 组件前缀
      grid: 'autoplay',
      // Flexbox 前缀版本
      flexbox: 'no-2009',
      // 是否移除无用的前缀
      remove: true,
    }),

    // 3. postcss-preset-env - 使用未来的 CSS 特性
    require('postcss-preset-env')({
      stage: 2,  // CSS 提案阶段（0-4，越小越实验性）
      features: {
        'nesting-rules': true,           // 原生嵌套
        'custom-media-queries': true,   // 自定义媒体查询
        'custom-selectors': true,         // 自定义选择器
        'color-functional-notation': true,  // 现代颜色函数
      },
    }),

    // 4. cssnano - 压缩优化（仅生产环境）
    ...(process.env.NODE_ENV === 'production'
      ? [require('cssnano')({ preset: 'default' })]
      : []),
  ],
};
```

#### 与构建工具集成

**Vite 集成**是现代项目最常见的方式。Vite 原生支持 PostCSS，只需在项目根目录创建 `postcss.config.js` 文件即可自动应用。

```typescript
// vite.config.ts
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    // Vite 内置的 PostCSS 支持（自动读取 postcss.config.js）
    postcss: {
      plugins: [
        require('tailwindcss'),
        require('autoprefixer'),
      ],
    },
    // SCSS 预处理器配置
    preprocessorOptions: {
      scss: {
        additionalData: `
          @import "@/styles/variables.scss";
          @import "@/styles/mixins.scss";
        `,
      },
    },
  },
});
```

**Webpack/Rspack 集成**需要显式配置 loader：

```javascript
// rspack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 提取 CSS 到单独文件
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  require('tailwindcss'),
                  require('autoprefixer'),
                  ...(process.env.NODE_ENV === 'production'
                    ? [require('cssnano')]
                    : []),
                ],
              },
            },
          },
        ],
      },
      {
        test: /\.scss$/,
        use: [
          'css-loader',
          'postcss-loader',
          'sass-loader',  // SCSS 编译
        ],
      },
    ],
  },
};
```

### Sass/SCSS 安装与配置

#### 安装 Sass 编译器

Sass 有两个主要的编译器实现：Dart Sass（官方推荐）和 LibSass（已废弃）。强烈推荐使用 Dart Sass。

```bash
# 使用 npm 安装 Dart Sass
npm install sass -D

# 或者安装 sass-embedded（更快，Node 原生绑定）
npm install sass-embedded -D
```

#### 项目配置

```javascript
// sass.config.js
module.exports = {
  // 加载器配置
  loaders: {
    // 全局变量注入（每个 SCSS 文件自动引入）
    additionalData: `
      @import "@/styles/_variables.scss";
      @import "@/styles/_mixins.scss";
    `,
  },

  // 编译器选项
  sassOptions: {
    // 输出风格：'expanded' | 'compressed' | 'compact' | 'nested'
    outputStyle: 'expanded',

    // 源映射
    sourceMap: true,
    sourceMapContents: true,

    // 精度
    precision: 5,

    // 忽略废弃警告
    silenceDeprecations: ['import', 'color-alpha'],

    // 导入路径
    includePaths: [
      './node_modules',
      './src/styles',
    ],

    // 字符编码
    charset: true,
  },
};
```

### CSS Modules 配置

CSS Modules 是将 CSS 类名局部化的一种解决方案，避免命名冲突。在 Vite 中原生支持，无需额外安装。

```typescript
// vite.config.ts
export default defineConfig({
  css: {
    modules: {
      // 类名生成规则
      generateScopedName: '[name]__[local]___[hash:base64:5]',

      // 驼峰命名
      localsConvention: 'camelCase',

      // 是否启用
      // true | false | 'local' | 'global'
      mode: 'local',
    },
  },
});
```

### Stylelint 配置

Stylelint 是 CSS 的代码质量检查工具，帮助维持代码风格一致性和发现潜在问题。

```bash
npm install stylelint stylelint-config-standard stylelint-config-recess-order -D
```

```json
// .stylelintrc.json
{
  "extends": [
    "stylelint-config-standard",
    "stylelint-config-recess-order"
  ],
  "rules": {
    // 颜色相关
    "color-no-hex": true,
    "color-named": "never",

    // 选择器相关
    "selector-class-pattern": "^[a-z][a-z0-9]*(-[a-z0-9]+)*$",
    "selector-max-compound-selectors": 4,
    "selector-max-specificity": "0,4,0",

    // 属性相关
    "property-no-vendor-prefix": true,
    "declaration-block-no-redundant-longhand-properties": true,

    // 值相关
    "value-keyword-case": "lower",

    // 单位相关
    "unit-case": "lower",
    "number-max-precision": 4,

    // 格式相关
    "indentation": 2,
    "max-nesting-depth": 4,
    "no-duplicate-selectors": true,

    // 忽略文件
    "ignoreFiles": [
      "node_modules/**",
      "dist/**",
      "build/**",
      "coverage/**"
    ]
  }
}
```

---

## 核心概念详解

### PostCSS 核心概念

#### 工作原理

PostCSS 的核心是一个解析-转换-生成的管道。输入是 CSS 字符串，解析器将其转换成抽象语法树（AST），然后插件对 AST 进行各种转换操作，最后生成器将 AST 重新转换成 CSS 字符串。

```
PostCSS 工作流程:

┌─────────────────────────────────────────────────────────────────┐
│                      PostCSS 处理流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Input CSS                                                       │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │ .container { display: flex; justify-content: center; }    │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  1. Parser (解析)                                        │  │
│   │  Root { type: 'root', nodes: [Rule { selector: '.container',│ │
│   │    nodes: [Declaration { prop: 'display', value: 'flex' }] }│ │
│   │  ] }                                                      │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  2. Plugin Transformation (插件转换)                      │  │
│   │  autoprefixer → postcss-preset-env → cssnano              │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  3. Stringifier (生成)                                   │  │
│   │  .container {                                            │  │
│   │    display: -webkit-box;                                  │  │
│   │    display: -ms-flexbox;                                   │  │
│   │    display: flex;                                         │  │
│   │    -webkit-box-pack: center;                              │  │
│   │    -ms-flex-pack: center;                                 │  │
│   │    justify-content: center;                                │  │
│   │  }                                                        │  │
│   └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 常用插件详解

**autoprefixer** 是最流行的 PostCSS 插件之一，它根据目标浏览器列表自动添加必要的 CSS 前缀。例如，`display: flex` 会自动转换为包含 `-webkit-box`、`-ms-flexbox` 等前缀的版本，确保在旧版 Safari 和 IE 中正常工作。

```javascript
// autoprefixer 配置详解
module.exports = {
  plugins: [
    require('autoprefixer')({
      // 目标浏览器列表
      overrideBrowserslist: [
        '> 1%',              // 使用率 > 1% 的浏览器
        'last 2 versions',   // 最新两个版本
        'not dead',           // 仍在维护的浏览器
        'Chrome >= 60',      // 具体版本要求
        'Firefox >= 55',
        'Safari >= 10',
        'iOS >= 10',
        'Edge >= 12',
        'not IE 11',         // 排除特定浏览器
      ],

      // Grid 组件前缀
      // 'autoplay' - 自动添加 Grid 相关前缀
      // 'autofill' - 自动填充模式
      // 'no-autoplay' - 不处理 Grid
      grid: 'autoplay',

      // Flexbox 前缀版本
      // 'no-2009' - 不添加 2009 语法（-webkit-box）
      // 'initial' - 添加所有版本
      flexbox: 'no-2009',

      // 是否移除无用的前缀
      remove: true,
    }),
  ],
};
```

**postcss-preset-env** 是一个综合性的插件包，将未来的 CSS 语法转换为当前可用的语法。它基于 CSS 提案阶段（Stage 0-4）提供支持，数字越小表示越实验性。

```javascript
// postcss-preset-env 配置详解
module.exports = {
  plugins: [
    require('postcss-preset-env')({
      // CSS 提案阶段
      // 0 - 第0阶段（任何提案）
      // 1 - 第1阶段（探索性）
      // 2 - 第2阶段（征求意见）← 推荐
      // 3 - 第3阶段（候选推荐）
      // 4 - 第4阶段（已标准化）
      stage: 2,

      // 启用哪些特性
      features: {
        // 嵌套规则（现在浏览器已原生支持）
        'nesting-rules': true,

        // 自定义媒体查询
        'custom-media-queries': true,

        // 自定义选择器
        'custom-selectors': true,

        // 颜色函数
        'color-functional-notation': true,

        // 伪类选择器
        'focus-visible-pseudo-class': true,
        'focus-within-pseudo-class': true,
      },

      // 是否保留原始值
      preserve: true,
    }),
  ],
};
```

**cssnano** 是 CSS 的优化和压缩工具，通过多种策略减小 CSS 文件体积，同时保持功能完全一致。

```javascript
// cssnano 配置详解
module.exports = {
  plugins: [
    require('cssnano')({
      // 预设
      // 'default' - 默认预设，安全的优化
      // 'advanced' - 高级预设，可能改变选择器结构
      // 'lite' - 轻量预设，只做基础优化
      // 'short' - 最短属性值
      preset: 'default',

      // 优化项配置
      plugins: [
        'postcss-discard-comments',      // 移除注释
        'postcss-discard-duplicates',    // 移除重复规则
        'postcss-merge-rules',           // 合并相同规则
        'postcss-merge-longhand',        // 合并简写属性
        'postcss-discard-empty',         // 移除空选择器
        'postcss-minify-colors',         // 最小化颜色值
        'postcss-normalize-url',        // 标准化 URL
        'postcss-reduce-transforms',    // 最小化 transform
      ],
    }),
  ],
};
```

### Sass/SCSS 核心概念

#### 变量系统

Sass 的变量系统是其最强大的特性之一，允许你定义可重用的值，避免硬编码。

```scss
// 变量类型

// 1. 数值变量
$base-font-size: 16px;
$spacing-unit: 8px;
$border-radius: 4px;

// 2. 颜色变量
$primary-color: #3b82f6;
$secondary-color: #8b5cf6;
$success-color: #10b981;
$warning-color: #f59e0b;
$danger-color: #ef4444;

// 3. 字符串变量
$font-stack: 'Inter', system-ui, -apple-system, sans-serif;
$icon-path: '/assets/icons/';

// 4. Map 变量（键值对）
$colors: (
  primary: #3b82f6,
  secondary: #8b5cf6,
  success: #10b981,
  warning: #f59e0b,
  danger: #ef4444,
);

$theme: (
  dark: (
    bg: #0f172a,
    text: #f1f5f9,
    border: #334155,
  ),
  light: (
    bg: #ffffff,
    text: #0f172a,
    border: #e2e8f0,
  ),
);

// 5. 使用 Map
.alert {
  color: map.get($colors, danger);
  background: map.get(map.get($theme, light), bg);
}
```

#### 嵌套规则

嵌套是 Sass 最常用的特性之一，允许在选择器内部声明子选择器，生成嵌套的 CSS 规则。

```scss
// 嵌套基础
.card {
  padding: 16px;

  .header {
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 12px;
  }

  .content {
    color: #333;

    p {
      margin-bottom: 8px;
      line-height: 1.6;
    }
  }
}

// 父选择器引用 &
.button {
  display: inline-flex;
  align-items: center;
  padding: 8px 16px;

  &:hover {
    background-color: #f5f5f5;
  }

  &:active {
    transform: scale(0.98);
  }

  &:focus {
    outline: 2px solid blue;
    outline-offset: 2px;
  }

  // 修改器
  &--primary {
    background-color: blue;
    color: white;

    &:hover {
      background-color: darken(blue, 10%);
    }
  }

  &--secondary {
    background-color: gray;
    color: black;
  }

  // 组合选择器
  & + & {
    margin-left: 8px;
  }

  &__icon {
    margin-right: 8px;
  }
}

// 属性嵌套
.element {
  font: {
    family: 'Arial', sans-serif;
    size: 16px;
    weight: bold;
  }

  border: {
    width: 1px;
    style: solid;
    color: #ccc;
  }
}
```

#### Mixin 和 Include

Mixin（混合器）是 Sass 中最强大的复用机制，允许定义可重用的样式块，甚至可以接受参数。

```scss
// Mixin 定义和使用

// 1. 基础 Mixin
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

@mixin flex-between {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

// 2. 带参数的 Mixin
@mixin button-variant($background, $color) {
  background-color: $background;
  color: $color;

  &:hover {
    background-color: darken($background, 10%);
  }

  &:active {
    background-color: darken($background, 15%);
  }
}

// 3. 带默认参数的 Mixin
@mixin respond-to($breakpoint, $direction: min) {
  @if $breakpoint == mobile {
    @media (max-width: 640px) { @content; }
  } @else if $breakpoint == tablet {
    @media (max-width: 1024px) { @content; }
  } @else if $breakpoint == desktop {
    @media (max-width: 1280px) { @content; }
  }
}

// 使用 @content 传递样式块
.card {
  padding: 16px;

  @include respond-to(mobile) {
    padding: 8px;
  }
}

// 4. 可变参数 Mixin
@mixin box-shadow($shadows...) {
  box-shadow: $shadows;
}

.shadowed {
  @include box-shadow(0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1));
}
```

#### 函数

Sass 内置了大量实用函数，可以进行颜色操作、数学计算、字符串处理等。

```scss
// Sass 内置函数

// 1. 颜色函数
$color: #3b82f6;

lighten($color, 10%);   // 变亮
darken($color, 10%);    // 变暗
saturate($color, 20%);  // 增加饱和度
desaturate($color, 20%); // 降低饱和度
mix($color, #ff0000, 50%); // 混合两种颜色

// 2. 数值函数
abs(-10px);           // 绝对值
round(3.5px);         // 四舍五入
ceil(3.2px);          // 向上取整
floor(3.8px);         // 向下取整
percentage(0.5);       // 转换为百分比
clamp(100px, 50%, 200px); // 限制范围

// 3. 自定义函数
@function rem($px) {
  @return calc($px / 16) * 1rem;
}

@function fluid-clamp($min-px, $max-px, $min-viewport: 320px, $max-viewport: 1280px) {
  $slope: calc(($max-px - $min-px) / ($max-viewport - $min-viewport));
  $slope-vw: calc($slope * 100vw);
  $intercept: calc($min-px - $slope * $min-viewport);
  @return clamp(#{$min-px}, #{$slope-vw} + #{$intercept}, #{$max-px});
}

.title {
  font-size: rem(32);
  padding: fluid-clamp(16px, 32px);
}
```

#### 条件语句和循环

```scss
// @if/@else 语句
$theme: dark;

.element {
  @if $theme == dark {
    background-color: #1a1a1a;
    color: #ffffff;
  } @else if $theme == light {
    background-color: #ffffff;
    color: #1a1a1a;
  } @else {
    background-color: #f5f5f5;
    color: #333333;
  }
}

// @for 循环
@for $i from 1 through 12 {
  .col-#{$i} {
    width: calc($i / 12 * 100%);
  }
}

// @each 循环 - 遍历 Map
$theme-colors: (
  "success": #10b981,
  "info": #3b82f6,
  "warning": #f59e0b,
  "danger": #ef4444,
);

@each $key, $value in $theme-colors {
  .text-#{$key} {
    color: $value;
  }
  .bg-#{$key} {
    background-color: $value;
  }
}
```

---

## CSS 架构模式

### ITCSS 架构

ITCSS（Inverted Triangle CSS）是一种 CSS 架构方法论，通过将样式按特异性（Specificity）分层来更好地组织和管理大型项目的样式。

```
ITCSS 架构层次:

        △
       ╱  ╲
      ╱ 内层 ╲          ← Less Specific (低特异性)
     ╱  通用   ╲
    ╱ 样式    ╲
   ╱──────────╲
  ╱  对象     ╲
 ╱  样式     ╲
╱─────────────╲
╱   组件     ╲          ← More Specific (高特异性)
╱   样式    ╲
╱────────────╲
╱   工具类   ╲
╲   最高优先级╱          ← Most Specific (最高特异性)
 ╲──────────╱
  ╲ Trumps ╱
   ╲──────╱
```

```scss
// ITCSS 文件结构与示例

// 1-settings/_variables.scss
// ==========================================
$color-brand: #3b82f6;
$color-gray: (
  50: #f9fafb,
  100: #f3f4f6,
  900: #111827,
);

$font-family-sans: 'Inter', system-ui, sans-serif;
$font-size-base: 1rem;

$breakpoints: (
  sm: 640px,
  md: 768px,
  lg: 1024px,
  xl: 1280px,
);

// 2-tools/_mixins.scss
// ==========================================
@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  }
}

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin text-clamp($lines: 2) {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: $lines;
  overflow: hidden;
}

// 5-objects/_containers.scss
// ==========================================
.o-container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: 1rem;
  
  @include respond-to(md) {
    padding-inline: 1.5rem;
  }
}

// 6-components/_buttons.scss
// ==========================================
.c-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  font-weight: 500;
  border-radius: 0.375rem;
  transition: all 0.2s ease;
  cursor: pointer;
  
  &--primary {
    background-color: $color-brand;
    color: white;
  }
}

// 7-utilities/_utilities.scss
// ==========================================
.u-text-center { text-align: center; }
.u-hidden { display: none; }
.u-sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  border: 0;
}
```

### Tailwind CSS 集成

Tailwind CSS 是一个实用优先（utility-first）的 CSS 框架，与 PostCSS 集成非常紧密。

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    require('tailwindcss'),
    require('autoprefixer'),
    ...(process.env.NODE_ENV === 'production'
      ? [require('cssnano')]
      : []),
  },
};
```

```scss
// src/styles/main.scss

// Tailwind 指令
@tailwind base;
@tailwind components;
@tailwind utilities;

// 自定义组件类
@layer components {
  .btn {
    @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg;
    
    &-primary {
      @apply bg-blue-600 text-white hover:bg-blue-700;
    }
    
    &-secondary {
      @apply bg-white text-gray-700 border border-gray-300 hover:bg-gray-50;
    }
  }
  
  .card {
    @apply bg-white rounded-xl shadow-sm border border-gray-100;
    
    &-header {
      @apply px-6 py-4 border-b border-gray-100;
    }
  }
}
```

---

## 常见问题与解决方案

### PostCSS 问题

**插件执行顺序**是常见的问题来源。PostCSS 插件按数组顺序执行，后面的插件处理前面插件的输出，所以顺序很重要。

```javascript
// 正确的顺序
module.exports = {
  plugins: [
    // 1. 先处理新语法
    require('postcss-preset-env')({ stage: 2 }),

    // 2. 添加前缀（在新语法转换之后）
    require('autoprefixer'),

    // 3. 最后压缩
    require('cssnano'),
  ],
};
```

### Sass 问题

**@import vs @use**：`@import` 已被废弃，应使用 `@use` 和 `@forward`。

```scss
// 正确：使用 @use
// _variables.scss
$primary-color: #3b82f6;

// _mixins.scss
@use 'variables';

@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

// main.scss
@use 'variables';
@use 'mixins';

body {
  color: variables.$primary-color;
}

.container {
  @include mixins.flex-center;
}
```

---

> [!TIP]
> **CSS 工程化推荐栈**：PostCSS + Tailwind CSS + Autoprefixer + CSSnano + Stylelint。这套组合覆盖了样式转换、浏览器兼容、自动优化和代码质量检查。对于需要更强大编程能力的项目，可以添加 Sass/SCSS 作为预处理器层。

---

*本文档由 [[归愚知识系统]] 自动生成*
