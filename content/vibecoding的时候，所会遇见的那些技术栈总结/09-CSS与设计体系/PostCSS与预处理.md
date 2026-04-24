# PostCSS 与 CSS 预处理完全指南

> [!NOTE]
> 本文档涵盖 PostCSS 插件生态、Sass/SCSS 预处理器、CSS Modules、Stylelint 及 CSS 工程化最佳实践，是构建现代化 CSS 工作流的完整指南。PostCSS 不仅是 CSS 的转换工具，更是现代 CSS 工程的基石。

---

## 1. 技术概述与定位

### 1.1 CSS 工程化的演进历程

在前端开发的历史长河中，CSS 的工程化经历了漫长的演进过程。早期的 CSS 几乎是纯手写的简单样式表，开发者直接在 HTML 文件中嵌入 `<style>` 标签，或者创建几个简单的 `.css` 文件。随着 Web 应用复杂度的提升，这种简单模式很快暴露出了问题：代码复用困难、命名冲突严重、维护成本高昂、缺乏现代编程语言的表达能力。

CSS 预处理器的出现标志着 CSS 工程化的第一次重大飞跃。Sass（最初称为 Syntactically Awesome StyleSheets）于 2006 年诞生，它引入了变量、嵌套、混合器（Mixin）、继承等编程概念，极大地提升了 CSS 的可维护性。随后 LESS、Stylus 等预处理器相继问世，形成了 CSS 预处理器的繁荣生态。这些工具将类似 CSS 的语法编译成标准的 CSS 代码，让开发者能够用更高级的语法编写样式。

PostCSS 的诞生则代表了 CSS 工程化的第二次重大突破。与预处理器不同，PostCSS 并不是一种新的语言或语法，而是一个用 JavaScript 插件转换 CSS 的工具链。PostCSS 的核心是一个解析器，它能够将 CSS 代码解析成抽象语法树（AST），然后允许开发者编写插件来转换这个语法树。这种设计使得 PostCSS 具有极高的灵活性和扩展性，可以实现从自动添加浏览器前缀到完整的 CSS 新语法转换。

### 1.2 PostCSS 的技术定位

PostCSS 的定位非常明确：成为 CSS 转换的「瑞士军刀」。它的设计哲学是「做一件事并做到极致」，通过组合不同的插件来完成各种 CSS 处理任务。这种模块化的设计理念使得 PostCSS 能够：

1. **保持 CSS 标准兼容性**：PostCSS 不会引入新的语法糖，而是专注于转换现有的或即将成为标准的 CSS 语法
2. **插件生态丰富**：从 autoprefixer 到 cssnext，从 cssnano 到 postcss-modules，插件生态几乎涵盖了所有 CSS 处理需求
3. **性能优异**：PostCSS 使用 JavaScript 编写，在 Node.js 环境下运行效率很高
4. **易于集成**：可以无缝集成到 Webpack、Rspack、Vite、Rollup 等主流构建工具中

```
PostCSS 技术定位图:

┌─────────────────────────────────────────────────────────────────┐
│                       PostCSS 生态系统                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────────────────────────────────────────┐    │
│   │                    输入层                               │    │
│   │  Standard CSS | Sass(通过预处理器) | Less | Stylus      │    │
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
│   │  │Autoprefixer│ │ cssnext │ │ cssnano │ │Modules  │   │    │
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

### 1.3 预处理器与 PostCSS 的关系

很多开发者会产生一个疑问：既然有了 Sass/LESS 等预处理器，为什么还需要 PostCSS？实际上，这两者并不是互斥的，而是可以协同工作的。

预处理器（如 Sass）解决的是「编写体验」的问题，它们让开发者能够用更方便、更强大的语法来编写样式。而 PostCSS 解决的是「转换输出」的问题，它在预处理器输出标准 CSS 之后，对 CSS 进行进一步的处理和优化。

一个典型的现代化 CSS 工作流是这样的：

1. **编写阶段**：开发者使用 Sass/LESS 等预处理器语法编写样式，享受变量、嵌套、混合器等便利特性
2. **编译阶段**：预处理器将 SCSS/LESS 编译成标准 CSS
3. **转换阶段**：PostCSS 接收标准 CSS，通过各种插件进行转换，如添加前缀、优化代码、转换新语法等
4. **输出阶段**：最终生成浏览器可识别的 CSS 文件

这种分工明确的工作流让每种工具专注于自己的强项，既保证了开发体验，又保证了最终输出的质量。

### 1.4 CSS 预处理器对比

| 特性 | Sass (SCSS) | LESS | Stylus |
|------|-------------|------|--------|
| **语法风格** | 类 CSS（SCSS）/缩进（Sass） | 类 CSS | 灵活（可省略冒号/大括号/分号） |
| **变量符号** | `$variable` | `@variable` | `variable` |
| **混合器语法** | `@mixin` / `@include` | `.class()` / `.mixin()` | `mixin()` |
| **继承** | `@extend` | 无原生支持 | `extends` |
| **条件语句** | `@if` / `@else` | JavaScript 评估 | `@if` |
| **循环** | `@for` / `@each` / `@while` | JavaScript 评估 | `@for` |
| **内置函数** | 丰富 | 一般 | 丰富 |
| **社区生态** | 非常大 | 一般 | 较小 |
| **学习曲线** | 低 | 低 | 中等 |
| **编译速度** | 快（dart-sass） | 较快 | 快 |
| **框架集成** | Bootstrap 4+、Foundation | Bootstrap 3、Many | 较少 |

---

## 2. 完整安装与配置

### 2.1 PostCSS 安装与配置

#### 2.1.1 基础安装

PostCSS 可以作为独立工具使用，也可以集成到各种构建工具中。首先介绍独立使用方式：

```bash
# 安装 PostCSS CLI
npm install postcss postcss-cli -D

# 安装常用插件
npm install autoprefixer postcss-preset-env cssnano -D
```

#### 2.1.2 配置文件创建

PostCSS 的配置文件可以是 `postcss.config.js`、`postcss.config.cjs`、`.postcssrc` 或 `.postcssrc.json`：

```javascript
// postcss.config.js
module.exports = {
  syntax: 'postcss-scss',  // 使用 SCSS 语法
  parser: 'postcss-scss',  // 指定解析器
  plugins: [
    // 1. Tailwind CSS
    require('tailwindcss'),

    // 2. Autoprefixer - 自动添加浏览器前缀
    require('autoprefixer')({
      // 目标浏览器
      overrideBrowserslist: [
        '> 1%',
        'last 2 versions',
        'not dead',
        'not IE 11',
      ],
      // Grid 组件前缀
      grid: 'autoplay',
      // Flexbox 前缀版本
      flexbox: 'no-2009',
      // 是否添加 remove 属性
      remove: true,
    }),

    // 3. cssnext - 使用未来的 CSS 特性
    require('postcss-preset-env')({
      stage: 2,  // CSS 提案阶段
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'custom-selectors': true,
        'color-functional-notation': true,
      },
    }),

    // 4. cssnano - 压缩优化
    require('cssnano')({
      preset: 'default',  // 'default' | 'advanced' | 'lite'
    }),
  ],
};
```

#### 2.1.3 与构建工具集成

**Vite 集成**：

```typescript
// vite.config.ts
import postcss from 'rollup-plugin-postcss';

export default defineConfig({
  css: {
    // Vite 内置的 PostCSS 支持
    postcss: {
      plugins: [
        require('tailwindcss'),
        require('autoprefixer'),
      ],
    },
    // SCSS 预处理器
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
      },
    },
  },
});
```

**Webpack/Rspack 集成**：

```javascript
// rspack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // 提取 CSS 到单独文件
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
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
          // 如果需要预处理
          'sass-loader',
        ],
      },
      {
        test: /\.scss$/,
        use: [
          'css-loader',
          'postcss-loader',
          'sass-loader',
        ],
      },
    ],
  },
};
```

### 2.2 Sass/SCSS 安装与配置

#### 2.2.1 安装 Sass 编译器

Sass 有两个主要的编译器实现：Dart Sass（官方推荐）和 LibSass（已废弃）。我们推荐使用 Dart Sass：

```bash
# 使用 npm 安装 Dart Sass
npm install sass -D

# 或者安装 sass-embedded（更快）
npm install sass-embedded -D
```

#### 2.2.2 项目配置

```javascript
// sass.config.js
module.exports = {
  // 加载器配置
  loaders: {
    // 全局变量注入
    additionalData: `
      @import "@/styles/_variables.scss";
      @import "@/styles/_mixins.scss";
      @import "@/styles/_functions.scss";
    `,
  },

  // 编译器选项
  sassOptions: {
    // 输出风格
    // 'expanded' | 'compressed' | 'compact' | 'nested'
    outputStyle: 'expanded',

    // 源映射
    sourceMap: true,
    sourceMapContents: true,

    // 精度
    precision: 5,

    // 警告视为错误
    silenceDeprecations: ['import', 'color-alpha'],

    // 添加路径用于@import
    includePaths: [
      './node_modules',
      './src/styles',
    ],

    // 字符编码
    charset: true,
  },
};
```

### 2.3 CSS Modules 配置

CSS Modules 是将 CSS 类名局部化的一种解决方案，避免命名冲突：

#### 2.3.1 Webpack/Rspack 配置

```javascript
// rspack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.module\.css$/,
        use: [
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                plugins: [
                  require('autoprefixer'),
                ],
              },
            },
          },
        ],
        options: {
          modules: {
            // 命名模式
            localIdentName: '[name]__[local]--[hash:base64:5]',

            // 命名规则
            namingPattern: 'module',

            // 导出选项
            exportLocalsConvention: 'camelCase',

            // 是否启用 CSS Modules
            mode: 'local',

            // 自定义哈希
            hashPrefix: 'hash',

            // 驼峰命名
            camelCase: true,
          },
        },
      },
    ],
  },
};
```

#### 2.3.2 Vite 配置

Vite 原生支持 CSS Modules：

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
      localsConvention: 'camelCaseOnly',
    },
  },
});
```

### 2.4 Stylelint 配置

Stylelint 是 CSS 的代码质量检查工具：

#### 2.4.1 安装与配置

```bash
npm install stylelint stylelint-config-standard stylelint-config-recess-order -D
```

```json
// .stylelintrc.json
{
  "extends": [
    "stylelint-config-standard",
    "stylelint-config-recess-order",
    "stylelint-config-prettier"
  ],
  "rules": {
    // 颜色相关
    "color-no-hex": true,
    "color-named": "never",

    // 选择器相关
    "selector-class-pattern": "^[a-z][a-z0-9]*(-[a-z0-9]+)*$",
    "selector-id-pattern": "^[a-z][a-z0-9]*(-[a-z0-9]+)*$",
    "selector-max-compound-selectors": 4,
    "selector-max-specificity": "0,4,0",
    "selector-no-vendor-prefix": true,

    // 属性相关
    "property-no-vendor-prefix": true,
    "property-no-unknown": true,
    "declaration-block-no-redundant-longhand-properties": true,
    "declaration-no-important": null,

    // 值相关
    "value-no-vendor-prefix": true,
    "value-keyword-case": "lower",

    // 单位相关
    "unit-case": "lower",
    "number-max-precision": 4,

    // 函数相关
    "function-url-quotes": "always",

    // 字符串相关
    "string-quotes": "double",

    // 长度相关
    "length-zero-no-unit": true,

    // 格式相关
    "indentation": 2,
    "max-nesting-depth": 4,
    "no-duplicate-selectors": true,
    "no-empty-source": true,

    // 注释
    "comment-empty-line-before": [
      "always",
      {
        "except": ["first-nested"],
        "ignore": ["after-comment", "inside-block"]
      }
    ],

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

#### 2.4.2 package.json 脚本配置

```json
{
  "scripts": {
    "lint:css": "stylelint \"src/**/*.{css,scss,vue}\" --fix",
    "lint:style": "stylelint \"src/**/*.{css,scss}\" --fix"
  }
}
```

---

## 3. 核心概念详解

### 3.1 PostCSS 核心概念

#### 3.1.1 工作原理

PostCSS 的核心是一个解析-转换-生成的管道。输入是 CSS 字符串，解析器将其转换成抽象语法树（AST），然后插件对 AST 进行各种转换操作，最后生成器将 AST 重新转换成 CSS 字符串。

```
PostCSS 工作流程:

┌─────────────────────────────────────────────────────────────────┐
│                      PostCSS 处理流程                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   Input CSS                                                       │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │ .container { display: flex; justify-content: center; }   │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  1. Parser (解析)                                         │  │
│   │                                                          │  │
│   │  Root {                                                    │  │
│   │    type: 'root',                                          │  │
│   │    nodes: [                                               │  │
│   │      Rule {                                               │  │
│   │        selector: '.container',                            │  │
│   │        nodes: [                                           │  │
│   │          Declaration { prop: 'display', value: 'flex' },  │  │
│   │          Declaration { prop: 'justify-content', value:   │  │
│   │            'center' }                                     │  │
│   │        ]                                                  │  │
│   │      }                                                    │  │
│   │    ]                                                      │  │
│   │  }                                                        │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  2. Plugin Transformation (插件转换)                        │  │
│   │                                                          │  │
│   │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │  │
│   │  │ autoprefixer│→ │ cssnext    │→ │ cssnano     │      │  │
│   │  └─────────────┘  └─────────────┘  └─────────────┘      │  │
│   └───────────────────────────────────────────────────────────┘  │
│                              │                                   │
│                              ▼                                   │
│   ┌───────────────────────────────────────────────────────────┐  │
│   │  3. Stringifier (生成)                                    │  │
│   │                                                          │  │
│   │  Output CSS                                              │  │
│   │  ┌─────────────────────────────────────────────────────┐│  │
│   │  │ .container {                                         ││  │
│   │  │   display: -webkit-box;                              ││  │
│   │  │   display: -ms-flexbox;                             ││  │
│   │  │   display: flex;                                     ││  │
│   │  │   -webkit-box-pack: center;                          ││  │
│   │  │   -ms-flex-pack: center;                             ││  │
│   │  │   justify-content: center;                           ││  │
│   │  │ }                                                    ││  │
│   │  └─────────────────────────────────────────────────────┘│  │
│   └───────────────────────────────────────────────────────────┘  │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### 3.1.2 常用插件详解

**autoprefixer**：自动添加浏览器前缀

```javascript
// autoprefixer 配置详解
module.exports = {
  plugins: [
    require('autoprefixer')({
      // 目标浏览器列表
      // 可以是数组、字符串或浏览器查询字符串
      overrideBrowserslist: [
        // 使用率 > 1% 且最新两个版本的浏览器
        '> 1%',
        'last 2 versions',
        'not dead',

        // 或者指定具体浏览器
        'Chrome >= 60',
        'Firefox >= 55',
        'Safari >= 10',
        'iOS >= 10',
        'Edge >= 12',

        // 排除特定浏览器
        'not IE 11',
        'not op_mini all',
      ],

      // Grid 组件前缀
      // 'autoplay' | 'autofill' | 'no-autoplay'
      grid: 'autoplay',

      // Flexbox 前缀版本
      // 'no-2009' | 'initial'
      flexbox: 'no-2009',

      // 是否移除无用的前缀
      remove: true,

      // 自定义前缀地图
      flexbox?: 'no-2009' | 'initial',
    }),
  ],
};
```

**cssnext / postcss-preset-env**：使用未来的 CSS 特性

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
        // 嵌套规则
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

        // 媒体查询范围
        'media-query-ranges': true,

        // 颜色相关
        'color-functional-notation': {
          preserve: true,
        },
      },

      // 是否保留原始值
      preserve: true,

      // 导入支持
      importFrom: {
        customMedia: {
          '--viewport-xs': '320px',
          '--viewport-sm': '640px',
          '--viewport-md': '768px',
          '--viewport-lg': '1024px',
          '--viewport-xl': '1280px',
        },
      },
    }),
  ],
};
```

**cssnano**：CSS 压缩与优化

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
        // 移除注释
        'postcss-discard-comments',

        // 移除重复规则
        'postcss-discard-duplicates',

        // 合并规则
        'postcss-merge-rules',

        // 合并 longhand 属性
        'postcss-merge-longhand',

        // 移除空选择器
        'postcss-discard-empty',

        // 最小化颜色值
        'postcss-minify-colors',

        // 最小化选择器
        'postcss-minify-selectors',

        // 最小化值
        'postcss-minify-params',

        // 最小化字体权重
        'postcss-minify-font-values',

        // 转换 color 函数
        'postcss-normalize-chars',

        // 转换长度单位
        'postcss-convert-values',

        // 压缩 id 选择器
        'postcss-reduce-idents',

        // 移除无用规则
        'postcss-reduce-rules',

        // 转换 CSS
        'postcss-svgo',

        // 最小化 unicode 范围
        'postcss-unique-selectors',

        // 安全优化
        'postcss-normalize-url',
        'postcss-ordered-values',
        'postcss-reduce-transforms',
      ],

      // 是否保留 sourcemap
      sourcemap: false,
    }),
  ],
};
```

#### 3.1.3 自定义 PostCSS 插件

```javascript
// 自定义 PostCSS 插件示例
const postcss = require('postcss');
const syntax = require('postcss-scss');

// 插件函数
const myPlugin = postcss.plugin('my-postcss-plugin', (options) => {
  return (root, result) => {
    // options 是插件配置
    const prefix = options?.prefix || 'custom';

    // 遍历所有规则
    root.walkRules((rule) => {
      // 在选择器前添加前缀
      rule.selector = `.${prefix} ${rule.selector}`;

      // 遍历所有声明
      rule.walkDecls((decl) => {
        // 处理特定属性
        if (decl.prop === 'color') {
          // 可以在这里转换颜色值
          console.log(`Found color: ${decl.value}`);
        }
      });
    });
  };
});

// 使用插件
module.exports = {
  plugins: [
    myPlugin({ prefix: 'my-prefix' }),
    require('autoprefixer'),
  ],
};
```

### 3.2 Sass/SCSS 核心概念

#### 3.2.1 变量系统

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
$image-extension: '.png';

// 4. 列表变量
$sizes: (sm, md, lg, xl);
$breakpoints: (320px, 768px, 1024px, 1280px);

// 5. Map 变量（键值对）
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

// 6. 布尔变量
$enable-flex: true;
$enable-grid: true;
$enable-transitions: true;

// 7. 空值和 null
$empty-value: null;
```

#### 3.2.2 嵌套规则

```scss
// 嵌套基础

// 1. 基础嵌套
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

  .footer {
    margin-top: 16px;
    padding-top: 16px;
    border-top: 1px solid #eee;
  }
}

// 2. 父选择器引用
.button {
  display: inline-flex;
  align-items: center;
  padding: 8px 16px;

  // & 替换父选择器
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

// 3. 属性嵌套
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

  margin: {
    top: 10px;
    right: 20px;
    bottom: 10px;
    left: 20px;
  }

  padding: {
    top: 10px;
    right: 15px;
    bottom: 10px;
    left: 15px;
  }
}
```

#### 3.2.3 Mixin 和 Include

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

@mixin flex-column {
  display: flex;
  flex-direction: column;
}

// 使用 Mixin
.container {
  @include flex-center;
}

.header {
  @include flex-between;
}

.sidebar {
  @include flex-column;
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

.btn-primary {
  @include button-variant(#3b82f6, white);
}

.btn-danger {
  @include button-variant(#ef4444, white);
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

// 使用 @content
.card {
  padding: 16px;

  @include respond-to(mobile) {
    padding: 8px;
  }

  @include respond-to(tablet) {
    padding: 12px;
  }
}

// 4. 可变参数 Mixin
@mixin box-shadow($shadows...) {
  box-shadow: $shadows;
}

.shadowed {
  @include box-shadow(0 2px 4px rgba(0,0,0,0.1), 0 4px 8px rgba(0,0,0,0.1));
}

// 5. Mixin 中的 Mixin
@mixin flex-base {
  display: flex;
}

@mixin button-base {
  @include flex-base;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}
```

#### 3.2.4 继承 (Extend)

```scss
// Placeholder 选择器
// 不会被编译到最终 CSS 中

// 1. 基础样式占位符
%button-base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border-radius: 4px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.2s ease;
}

%text-overflow {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

%visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

// 使用 Extend
.primary-button {
  @extend %button-base;
  background-color: blue;
  color: white;
}

.secondary-button {
  @extend %button-base;
  background-color: gray;
  color: black;
}

.truncated-text {
  @extend %text-overflow;
}

.hidden-label {
  @extend %visually-hidden;
}

// 2. 链式继承
.base-card {
  padding: 16px;
  border-radius: 8px;
  background: white;
}

.featured-card {
  @extend .base-card;
  border: 2px solid gold;
}

.highlighted-card {
  @extend .featured-card;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
```

#### 3.2.5 函数

```scss
// Sass 内置函数

// 1. 颜色函数
$color: #3b82f6;

// 颜色变暗/变亮
lighten($color, 10%);  // 变亮
darken($color, 10%);   // 变暗

// 调整饱和度
saturate($color, 20%);   // 增加饱和度
desaturate($color, 20%); // 降低饱和度

// 调整透明度
rgba($color, 0.5);       // 带透明度
opacify($color, 0.3);    // 增加不透明度
transparentize($color, 0.3); // 降低不透明度

// 灰度
grayscale($color);

// 反转颜色
invert($color);

// 混合颜色
mix($color, #ff0000, 50%); // 混合两种颜色

// 2. 字符串函数
unquote("hello");           // 移除引号
quote(hello);               // 添加引号
to-upper-case("hello");     // 转大写
to-lower-case("HELLO");     // 转小写
str-length("hello");        // 字符串长度
str-insert("hello", " world", 6); // 插入字符串

// 3. 数值函数
abs(-10px);                 // 绝对值
round(3.5px);              // 四舍五入
ceil(3.2px);               // 向上取整
floor(3.8px);              // 向下取整
percentage(0.5);           // 转换为百分比
min(1px, 2px, 3px);       // 最小值
max(1px, 2px, 3px);       // 最大值
clamp(100px, 50%, 200px); // 限制范围

// 4. 列表函数
length(10px 20px 30px);   // 列表长度
nth(10px 20px 30px, 2);   // 获取第 n 个元素
index(10px 20px 30px, 20px); // 查找元素索引
append(10px 20px, 30px);  // 添加元素
join(10px, 20px);         // 合并列表

// 5. Map 函数
map-get($colors, primary);           // 获取值
map-keys($colors);                    // 获取所有键
map-values($colors);                  // 获取所有值
map-has-key($colors, primary);       // 检查键是否存在
map-merge($colors, (info: #00bcd4)); // 合并 Map

// 自定义函数
@function rem($px) {
  @return calc($px / 16) * 1rem;
}

@function em($px, $base: 16px) {
  @return calc($px / $base) * 1em;
}

@function fluid-clamp($min-px, $max-px, $min-viewport: 320px, $max-viewport: 1280px) {
  $slope: calc(($max-px - $min-px) / ($max-viewport - $min-viewport));
  $slope-vw: calc($slope * 100vw);
  $intercept: calc($min-px - $slope * $min-viewport);
  @return clamp(#{$min-px}, #{$slope-vw} + #{$intercept}, #{$max-px});
}

.title {
  font-size: rem(32);
  line-height: em(40, 32);
  padding: fluid-clamp(16px, 32px);
}
```

#### 3.2.6 条件语句和循环

```scss
// 1. @if/@else 语句
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

// 2. @for 循环
@for $i from 1 through 12 {
  .col-#{$i} {
    width: calc($i / 12 * 100%);
  }
}

// 3. @each 循环 - 遍历列表
$colors: (primary: blue, secondary: green, danger: red);

@each $name, $color in $colors {
  .btn-#{$name} {
    background-color: $color;
    color: white;
  }
}

// 4. @each 循环 - 遍历 Map
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

// 5. @while 循环
$i: 1;
@while $i <= 4 {
  .level-#{$i} {
    font-size: 10px + ($i * 2px);
  }
  $i: $i + 1;
}

// 6. 嵌套循环
@each $breakpoint in (mobile, tablet, desktop) {
  @each $size in (sm, md, lg, xl) {
    .icon-#{$breakpoint}-#{$size} {
      width: #{$size}px;
    }
  }
}
```

### 3.4 Tailwind CSS 深度集成

#### 3.4.1 Tailwind CSS 工作原理

Tailwind CSS 是一个实用优先（utility-first）的 CSS 框架，它的工作原理与传统的 CSS 框架有本质区别。

**核心原理**：
Tailwind CSS 不会生成传统意义上的组件样式，而是提供大量低级工具类，开发者通过组合这些类来构建自定义设计。

```
Tailwind CSS 处理流程:

Source Code
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  HTML/Vue/React 组件中的 Tailwind 类名                   │
│  <div class="flex items-center justify-between p-4">   │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  Tailwind CSS 构建时扫描所有类名                        │
│  ┌─────────────────────────────────────────────────┐  │
│  │  1. 解析 HTML/JSX/Vue 文件                      │  │
│  │  2. 提取 class="" 和 className="" 属性         │  │
│  │  3. 匹配 Tailwind 配置中的工具类                │  │
│  │  4. 生成对应的 CSS                             │  │
│  └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  最终生成的 CSS                                       │
│  .flex { display: flex; }                            │
│  .items-center { align-items: center; }              │
│  .justify-between { justify-content: space-between; } │
│  .p-4 { padding: 1rem; }                            │
└─────────────────────────────────────────────────────────┘
```

#### 3.4.2 Tailwind CSS 配置详解

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  // 内容配置 - 告诉 Tailwind 需要扫描哪些文件
  content: [
    './index.html',
    './src/**/*.{vue,js,ts,jsx,tsx,vue}',
    './components/**/*.{vue,js,ts,jsx,tsx}',
    './pages/**/*.{vue,js,ts,jsx,tsx}',
    './layouts/**/*.{vue,js,ts,jsx,tsx}',
    './plugins/**/*.{js,ts}',
    './app.vue',
    './error.vue',
  ],
  
  // 预设配置
  presets: [
    // 使用自定义预设
    require('./tailwind.preset.js'),
  ],
  
  // 主题配置 - 自定义设计系统
  theme: {
    // 扩展默认主题
    extend: {
      // 颜色系统
      colors: {
        // 品牌色
        brand: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
          950: '#082f49',
        },
        
        // 语义颜色
        success: {
          light: '#dcfce7',
          DEFAULT: '#22c55e',
          dark: '#15803d',
        },
        warning: {
          light: '#fef9c3',
          DEFAULT: '#eab308',
          dark: '#a16207',
        },
        danger: {
          light: '#fee2e2',
          DEFAULT: '#ef4444',
          dark: '#dc2626',
        },
      },
      
      // 字体系统
      fontFamily: {
        sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
        serif: ['Merriweather', 'Georgia', 'serif'],
        mono: ['JetBrains Mono', 'Fira Code', 'monospace'],
        display: ['Poppins', 'sans-serif'],
      },
      
      // 字体大小
      fontSize: {
        'xs': ['0.75rem', { lineHeight: '1rem' }],
        'sm': ['0.875rem', { lineHeight: '1.25rem' }],
        'base': ['1rem', { lineHeight: '1.5rem' }],
        'lg': ['1.125rem', { lineHeight: '1.75rem' }],
        'xl': ['1.25rem', { lineHeight: '1.75rem' }],
        '2xl': ['1.5rem', { lineHeight: '2rem' }],
        '3xl': ['1.875rem', { lineHeight: '2.25rem' }],
        '4xl': ['2.25rem', { lineHeight: '2.5rem' }],
        '5xl': ['3rem', { lineHeight: '1.1' }],
        '6xl': ['3.75rem', { lineHeight: '1' }],
        '7xl': ['4.5rem', { lineHeight: '1' }],
        '8xl': ['6rem', { lineHeight: '1' }],
        '9xl': ['8rem', { lineHeight: '1' }],
      },
      
      // 间距系统
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
        '128': '32rem',
        '144': '36rem',
      },
      
      // 边框半径
      borderRadius: {
        'none': '0',
        'sm': '0.125rem',
        'DEFAULT': '0.25rem',
        'md': '0.375rem',
        'lg': '0.5rem',
        'xl': '0.75rem',
        '2xl': '1rem',
        '3xl': '1.5rem',
        'full': '9999px',
      },
      
      // 阴影系统
      boxShadow: {
        'inner-lg': 'inset 0 2px 4px 0 rgb(0 0 0 / 0.05)',
        'glow': '0 0 20px rgb(59 130 246 / 0.5)',
        'glow-lg': '0 0 40px rgb(59 130 246 / 0.5)',
      },
      
      // 动画
      animation: {
        'spin-slow': 'spin 3s linear infinite',
        'bounce-slow': 'bounce 3s infinite',
        'fade-in': 'fadeIn 0.3s ease-out',
        'slide-up': 'slideUp 0.3s ease-out',
      },
      
      // 关键帧
      keyframes: {
        fadeIn: {
          '0%': { opacity: '0' },
          '100%': { opacity: '1' },
        },
        slideUp: {
          '0%': { transform: 'translateY(10px)', opacity: '0' },
          '100%': { transform: 'translateY(0)', opacity: '1' },
        },
      },
      
      // 容器
      container: {
        center: true,
        padding: '1rem',
        screens: {
          sm: '640px',
          md: '768px',
          lg: '1024px',
          xl: '1280px',
          '2xl': '1400px',
        },
      },
    },
  },
  
  // 变体配置
  variants: {
    extend: {
      // 启用 hover, focus, active 等变体
      backgroundColor: ['hover', 'focus', 'active', 'disabled'],
      textColor: ['hover', 'focus', 'active', 'disabled'],
      borderColor: ['hover', 'focus'],
      opacity: ['disabled'],
      scale: ['hover', 'focus', 'active'],
      translate: ['hover', 'focus'],
      animation: ['motion-safe', 'motion-reduce'],
    },
  },
  
  // 插件
  plugins: [
    // 表单样式重置
    require('@tailwindcss/forms'),
    
    // 排版插件
    require('@tailwindcss/typography'),
    
    // 容器查询
    require('@tailwindcss/container-queries'),
    
    // 自定义插件
    function({ addUtilities, addComponents, addBase, theme }) {
      // 添加工具类
      addUtilities({
        '.text-balance': {
          'text-wrap': 'balance',
        },
        '.text-pretty': {
          'text-wrap': 'pretty',
        },
      });
      
      // 添加组件样式
      addComponents({
        '.btn': {
          '@apply inline-flex items-center justify-center rounded-lg font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2': {},
        },
      });
      
      // 添加基础样式
      addBase({
        'h1': { '@apply text-4xl font-bold': {} },
        'h2': { '@apply text-3xl font-bold': {} },
      });
    },
  ],
};
```

#### 3.4.3 Tailwind CSS 与 PostCSS 集成

```javascript
// postcss.config.js
module.exports = {
  plugins: {
    // 1. Tailwind CSS（必须最先执行）
    require('tailwindcss'),
    
    // 2. Autoprefixer（自动添加前缀）
    require('autoprefixer')({
      overrideBrowserslist: [
        '> 1%',
        'last 2 versions',
        'not dead',
      ],
    }),
    
    // 3. CSSNext / PostCSS Preset Env（可选）
    // 启用现代 CSS 特性
    require('postcss-preset-env')({
      stage: 2,
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'custom-selectors': true,
      },
    }),
    
    // 4. CSSnano（仅在生产环境）
    ...(process.env.NODE_ENV === 'production' 
      ? [require('cssnano')({ preset: 'default' })] 
      : []),
  },
};
```

```scss
// src/styles/main.scss

// 1. Tailwind 基础指令
@tailwind base;
@tailwind components;
@tailwind utilities;

// 2. Tailwind Layer 覆盖
@layer base {
  // 自定义基础样式
  body {
    @apply bg-gray-50 text-gray-900 antialiased;
    font-family: 'Inter', system-ui, sans-serif;
  }
  
  // 自定义标题样式
  h1, h2, h3, h4, h5, h6 {
    @apply font-display font-semibold text-gray-900;
  }
  
  // 自定义链接样式
  a {
    @apply text-brand-600 hover:text-brand-700 transition-colors;
  }
  
  // 自定义输入框样式
  input[type="text"],
  input[type="email"],
  input[type="password"],
  textarea,
  select {
    @apply border-gray-300 rounded-md shadow-sm focus:border-brand-500 focus:ring-brand-500;
  }
}

@layer components {
  // 自定义组件类
  .btn {
    @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg transition-all;
    
    // 状态变体
    &-primary {
      @apply bg-brand-600 text-white hover:bg-brand-700;
      @apply focus:ring-brand-500 focus:ring-offset-2;
      @apply disabled:opacity-50 disabled:cursor-not-allowed;
    }
    
    &-secondary {
      @apply bg-white text-gray-700 border border-gray-300;
      @apply hover:bg-gray-50;
      @apply focus:ring-brand-500 focus:ring-offset-2;
    }
    
    &-danger {
      @apply bg-danger text-white hover:bg-danger-dark;
      @apply focus:ring-danger focus:ring-offset-2;
    }
    
    // 尺寸变体
    &-sm {
      @apply px-3 py-1.5 text-xs;
    }
    
    &-lg {
      @apply px-6 py-3 text-base;
    }
  }
  
  // 卡片组件
  .card {
    @apply bg-white rounded-xl shadow-sm border border-gray-100;
    @apply transition-shadow hover:shadow-md;
    
    &-header {
      @apply px-6 py-4 border-b border-gray-100;
    }
    
    &-body {
      @apply px-6 py-4;
    }
    
    &-footer {
      @apply px-6 py-4 border-t border-gray-100;
      @apply bg-gray-50 rounded-b-xl;
    }
  }
  
  // 输入组
  .input-group {
    @apply relative;
    
    &-icon {
      @apply absolute inset-y-0 left-0 flex items-center pl-3 pointer-events-none;
      @apply text-gray-400;
    }
    
    &-input {
      @apply block w-full pl-10 pr-3 py-2;
    }
  }
}

@layer utilities {
  // 自定义工具类
  .text-balance {
    text-wrap: balance;
  }
  
  .scrollbar-hide {
    -ms-overflow-style: none;
    scrollbar-width: none;
    
    &::-webkit-scrollbar {
      display: none;
    }
  }
  
  .gradient-text {
    @apply bg-clip-text text-transparent bg-gradient-to-r;
    
    &-brand {
      background-image: linear-gradient(to right, theme('colors.brand.500'), theme('colors.brand.700'));
    }
  }
}
```

### 3.5 CSS 架构模式

#### 3.5.1 ITCSS 架构

ITCSS（Inverted Triangle CSS）是一种 CSS 架构方法论，旨在更好地组织和管理大型项目的样式。

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
    ╲    ╱
     ╲  ╱
      ╲╱
```

```scss
// ITCSS 文件结构
// styles/
// ├── 1-settings/         # 配置
// │   ├── _variables.scss
// │   ├── _colors.scss
// │   ├── _typography.scss
// │   └── _config.scss
// │
// ├── 2-tools/           # 工具
// │   ├── _mixins.scss
// │   ├── _functions.scss
// │   └── _helpers.scss
// │
// ├── 3-generic/         # 通用
// │   ├── _reset.scss
// │   ├── _normalize.scss
// │   └── _box-sizing.scss
// │
// ├── 4-base/           # 基础元素
// │   ├── _html.scss
// │   ├── _body.scss
// │   ├── _headings.scss
// │   ├── _links.scss
// │   └── _images.scss
// │
// ├── 5-objects/        # 对象（布局）
// │   ├── _containers.scss
// │   ├── _grid.scss
// │   ├── _media.scss
// │   └── _lists.scss
// │
// ├── 6-components/     # 组件
// │   ├── _buttons.scss
// │   ├── _cards.scss
// │   ├── _forms.scss
// │   ├── _navigation.scss
// │   └── _modals.scss
// │
// └── 7-utilities/     # 工具类（最高优先级）
//     ├── _utilities.scss
//     ├── _helpers.scss
//     └── _trumps.scss

// 1-settings/_variables.scss
// ==========================================

// 颜色系统
$color-brand: (
  50: #f0f9ff,
  100: #e0f2fe,
  200: #bae6fd,
  300: #7dd3fc,
  400: #38bdf8,
  500: #0ea5e9,
  600: #0284c7,
  700: #0369a1,
  800: #075985,
  900: #0c4a6e,
);

$color-gray: (
  50: #f9fafb,
  100: #f3f4f6,
  200: #e5e7eb,
  300: #d1d5db,
  400: #9ca3af,
  500: #6b7280,
  600: #4b5563,
  700: #374151,
  800: #1f2937,
  900: #111827,
);

$color-semantic: (
  success: #22c55e,
  warning: #eab308,
  danger: #ef4444,
  info: #3b82f6,
);

// 排版系统
$font-family-sans: 'Inter', system-ui, -apple-system, sans-serif;
$font-family-serif: 'Merriweather', Georgia, serif;
$font-family-mono: 'JetBrains Mono', 'Fira Code', monospace;

$font-size-base: 1rem;
$line-height-tight: 1.25;
$line-height-normal: 1.5;
$line-height-relaxed: 1.75;

// 间距系统
$spacing-unit: 0.25rem;
$spacing-scale: (
  0: 0,
  1: $spacing-unit,
  2: $spacing-unit * 2,
  3: $spacing-unit * 3,
  4: $spacing-unit * 4,
  5: $spacing-unit * 5,
  6: $spacing-unit * 6,
  8: $spacing-unit * 8,
  10: $spacing-unit * 10,
  12: $spacing-unit * 12,
  16: $spacing-unit * 16,
  20: $spacing-unit * 20,
  24: $spacing-unit * 24,
);

// 断点
$breakpoints: (
  sm: 640px,
  md: 768px,
  lg: 1024px,
  xl: 1280px,
  2xl: 1536px,
);

// 2-tools/_mixins.scss
// ==========================================

// 响应式断点 mixin
@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  }
}

// 响应式断点（向下）
@mixin respond-below($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (max-width: #{map-get($breakpoints, $breakpoint) - 1px}) {
      @content;
    }
  }
}

// 响应式断点（范围）
@mixin respond-between($lower, $upper) {
  @if map-has-key($breakpoints, $lower) and map-has-key($breakpoints, $upper) {
    @media (min-width: #{map-get($breakpoints, $lower)}) and (max-width: #{map-get($breakpoints, $upper) - 1px}) {
      @content;
    }
  }
}

// Flexbox 工具 mixin
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin flex-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

@mixin flex-column {
  display: flex;
  flex-direction: column;
}

// 文字截断 mixin
@mixin text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@mixin text-clamp($lines: 2) {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: $lines;
  overflow: hidden;
}

// 4-base/_body.scss
// ==========================================

html {
  font-size: 16px;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  scroll-behavior: smooth;
}

body {
  font-family: $font-family-sans;
  font-size: $font-size-base;
  line-height: $line-height-normal;
  color: map-get($color-gray, 900);
  background-color: #ffffff;
}

// 5-objects/_containers.scss
// ==========================================

.o-container {
  width: 100%;
  max-width: 1280px;
  margin-inline: auto;
  padding-inline: map-get($spacing-scale, 4);
  
  @include respond-to(md) {
    padding-inline: map-get($spacing-scale, 6);
  }
  
  @include respond-to(lg) {
    padding-inline: map-get($spacing-scale, 8);
  }
}

.o-container--narrow {
  max-width: 768px;
}

.o-container--wide {
  max-width: 1536px;
}

// 6-components/_buttons.scss
// ==========================================

.c-btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: map-get($spacing-scale, 2) map-get($spacing-scale, 4);
  font-weight: 500;
  border-radius: 0.375rem;
  transition: all 0.2s ease;
  cursor: pointer;
  
  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
  
  // Primary variant
  &--primary {
    background-color: map-get($color-brand, 600);
    color: white;
    
    &:hover:not(:disabled) {
      background-color: map-get($color-brand, 700);
    }
    
    &:focus {
      outline: none;
      box-shadow: 0 0 0 3px rgba(map-get($color-brand, 500), 0.5);
    }
  }
  
  // Secondary variant
  &--secondary {
    background-color: white;
    color: map-get($color-gray, 700);
    border: 1px solid map-get($color-gray, 300);
    
    &:hover:not(:disabled) {
      background-color: map-get($color-gray, 50);
    }
  }
  
  // Sizes
  &--sm {
    padding: map-get($spacing-scale, 1) map-get($spacing-scale, 3);
    font-size: 0.875rem;
  }
  
  &--lg {
    padding: map-get($spacing-scale, 3) map-get($spacing-scale, 6);
    font-size: 1.125rem;
  }
}

// 7-utilities/_utilities.scss
// ==========================================

.u-text-center { text-align: center; }
.u-text-left { text-align: left; }
.u-text-right { text-align: right; }

.u-font-bold { font-weight: 700; }
.u-font-semibold { font-weight: 600; }
.u-font-medium { font-weight: 500; }

.u-hidden { display: none; }
.u-visible { visibility: visible; }
.u-invisible { visibility: hidden; }

.u-sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

#### 3.5.2 SMACSS 架构

SMACSS（Scalable and Modular Architecture for CSS）是另一种流行的 CSS 架构方法。

```scss
// SMACSS 文件结构
// styles/
// ├── base/               # 基础样式（reset/normalize）
// │   ├── _reset.scss
// │   ├── _base.scss
// │   └── _typography.scss
// │
// ├── layout/            # 布局样式
// │   ├── _header.scss
// │   ├── _footer.scss
// │   ├── _sidebar.scss
// │   └── _grid.scss
// │
// ├── module/           # 模块样式
// │   ├── _button.scss
// │   ├── _card.scss
// │   ├── _navigation.scss
// │   └── _form.scss
// │
// ├── state/           # 状态样式
// │   ├── _is-active.scss
// │   ├── _is-hidden.scss
// │   └── _is-expanded.scss
// │
// └── theme/           # 主题样式
//     ├── _light.scss
//     ├── _dark.scss
//     └── _high-contrast.scss

// SMACSS 命名约定
// ==========================================

// Layout: l- 前缀
.l-header {}
.l-sidebar {}
.l-main-content {}
.l-grid {}

// Module: 无前缀，使用语义化名称
.button {}
.card {}
.navigation {}

// Sub-module: -- 变体
.button--primary {}
.button--secondary {}

// State: is- 前缀
.is-active {}
.is-hidden {}
.is-expanded {}
.is-disabled {}

// Theme: theme- 前缀或 data-theme 属性
[data-theme="dark"] {}
.theme-high-contrast {}

// 实际示例
// ==========================================

// Layout
.l-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.l-sidebar {
  position: fixed;
  top: 0;
  left: 0;
  width: 250px;
  height: 100vh;
  background: #f5f5f5;
  border-right: 1px solid #ddd;
}

// Module
.card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  
  &__header {
    padding: 16px;
    border-bottom: 1px solid #eee;
  }
  
  &__body {
    padding: 16px;
  }
  
  &__footer {
    padding: 16px;
    background: #f9f9f9;
    border-top: 1px solid #eee;
  }
  
  // Sub-modules
  &--featured {
    border: 2px solid #3b82f6;
  }
  
  &--compact {
    .card__body {
      padding: 8px;
    }
  }
}

// State
.button {
  &.is-loading {
    opacity: 0.7;
    pointer-events: none;
    
    &::after {
      content: '';
      display: inline-block;
      width: 16px;
      height: 16px;
      border: 2px solid currentColor;
      border-right-color: transparent;
      border-radius: 50%;
      animation: spin 0.6s linear infinite;
    }
  }
  
  &.is-disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}

.navigation {
  &__item {
    display: block;
    padding: 8px 16px;
    
    &.is-active {
      background: #3b82f6;
      color: white;
    }
  }
}

// Theme
[data-theme="dark"] {
  .card {
    background: #1f2937;
    color: #f9fafb;
  }
  
  .button {
    background: #374151;
    color: #f9fafb;
    border-color: #4b5563;
  }
}
```

### 3.6 CSS 新特性与未来

#### 3.6.1 CSS 容器查询

容器查询是 CSS 的重要新特性，允许基于父容器尺寸而非视口尺寸进行样式响应。

```css
/* 容器查询基本用法 */
.card-container {
  /* 定义容器 */
  container-type: inline-size;
  container-name: card;
}

.card {
  /* 使用容器查询 */
  @container card (min-width: 400px) {
    display: flex;
    flex-direction: row;
  }
  
  @container card (max-width: 399px) {
    display: flex;
    flex-direction: column;
  }
}

/* 嵌套容器查询 */
.outer-container {
  container-type: inline-size;
  
  .inner-container {
    container-type: inline-size;
    
    @container (min-width: 300px) {
      /* 样式 */
    }
  }
}

/* 容器查询单位 */
.card {
  /* cqw: 容器宽度的 1% */
  /* cqh: 容器高度的 1% */
  /* cqi: 容器内联尺寸的 1% */
  /* cqb: 容器块级尺寸的 1% */
  width: 50cqi;
  font-size: 2cqb;
  padding: 2cqw;
}

/* 容器查询样式传播 */
.card-container {
  container-type: inline-size;
  container-style: safe;
}

.card {
  /* 使用 container-name 指定查询哪个容器 */
  @container card (min-width: 300px) {
    font-size: 1.25rem;
  }
  
  /* 不指定则查询最近的容器 */
  @container (min-width: 200px) {
    border-radius: 0.5rem;
  }
}
```

#### 3.6.2 CSS 嵌套

现代 CSS 原生支持嵌套语法，无需预处理器。

```css
/* CSS 原生嵌套 */
.card {
  padding: 1rem;
  background: white;
  
  /* 嵌套选择器 */
  & .card-header {
    font-size: 1.25rem;
    font-weight: 600;
  }
  
  /* 伪类嵌套 */
  &:hover {
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
  
  /* 伪元素嵌套 */
  &::before {
    content: '';
    display: block;
    height: 4px;
    background: linear-gradient(to right, #3b82f6, #8b5cf6);
  }
  
  /* 媒体查询嵌套 */
  @media (max-width: 768px) {
    padding: 0.75rem;
  }
  
  /* 条件嵌套 */
  @supports (display: grid) {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 嵌套 at-rule */
.card {
  color: blue;
  
  @media (width >= 600px) {
    color: red;
    
    &:hover {
      color: green;
    }
  }
}

/* 复杂的嵌套示例 */
.navigation {
  &__list {
    display: flex;
    list-style: none;
    
    &-item {
      position: relative;
      
      &:hover {
        .navigation__submenu {
          display: block;
        }
      }
    }
  }
  
  &__link {
    display: block;
    padding: 0.5rem 1rem;
    color: inherit;
    
    &:hover {
      background: rgba(0, 0, 0, 0.05);
    }
    
    &--active {
      color: blue;
      font-weight: 600;
    }
  }
  
  &__submenu {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    min-width: 200px;
    background: white;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }
}
```

#### 3.6.3 CSS 颜色函数

```css
/* 颜色空间 */
.element {
  /* OKLCH - 更自然的颜色过渡 */
  background: oklch(65% 0.2 240);
  
  /* OKLAB - 感知均匀的颜色空间 */
  color: oklab(59% 0.1 -0.1);
  
  /* Display P3 - 更广的色域 */
  background: color(display-p3 0.8 0.2 0.4);
  
  /* LCH - 亮度和色度 */
  background: lch(65% 60 240);
}

/* 颜色混合 */
.blend {
  /* 基础混合 */
  background: color-mix(in srgb, red 40%, blue);
  
  /* 指定色彩空间 */
  background: color-mix(in oklch, red 40%, blue);
  
  /* 透明度混合 */
  background: color-mix(in srgb, red 40%, blue 60%);
}

/* 相对颜色语法 */
:root {
  --brand-hue: 240;
  --brand-chroma: 0.3;
  --brand-l: 0.5;
  
  --brand-base: oklch(var(--brand-l) var(--brand-chroma) var(--brand-hue));
  --brand-light: oklch(
    calc(var(--brand-l) + 0.2) 
    var(--brand-chroma) 
    var(--brand-hue)
  );
  --brand-dark: oklch(
    calc(var(--brand-l) - 0.2) 
    var(--brand-chroma) 
    var(--brand-hue)
  );
}

/* 渐变中使用新颜色空间 */
.gradient {
  /* OKLCH 渐变 - 更平滑的过渡 */
  background: linear-gradient(
    to right in oklch,
    oklch(70% 0.2 240),
    oklch(70% 0.2 0)
  );
  
  /* 传统的 sRGB 渐变 */
  background: linear-gradient(to right, #ff0000, #0000ff);
}

/* 颜色对比度 */
.text {
  /* 自动选择可访问的对比色 */
  color: color-contrast(white vs red, blue, green);
  
  /* 手动设置对比度 */
  background: oklch(95% 0.02 240);
  color: oklch(15% 0.1 240); /* 高对比度 */
}
```

---

## 4. 常用命令与操作

### 4.1 命令行工具

#### 4.1.1 PostCSS CLI

```bash
# 编译单个文件
postcss src/styles/main.css -o dist/styles/main.css

# 监视文件变化
postcss src/styles/*.css --watch -o dist/styles/

# 使用配置文件
postcss src/styles/main.css --config postcss.config.js -o dist/styles/main.css

# 自动添加前缀
postcss src/styles/main.css --use autoprefixer -o dist/styles/main.css

# 使用多个插件
postcss src/styles/main.css \
  --use postcss-preset-env \
  --use autoprefixer \
  --use cssnano \
  -o dist/styles/main.min.css

# 压缩输出
postcss src/styles/main.css --env production -o dist/styles/main.min.css

# 显示详细日志
postcss src/styles/main.css --verbose -o dist/styles/main.css

# 输出 Source Map
postcss src/styles/main.css --map -o dist/styles/main.css
```

#### 4.1.2 Sass CLI

```bash
# 编译 SCSS 到 CSS
sass src/styles/main.scss dist/styles/main.css

# 监视文件变化
sass --watch src/styles/main.scss:dist/styles/main.css

# 监视整个目录
sass --watch src/styles:dist/styles

# 压缩输出
sass --style=compressed src/styles/main.scss dist/styles/main.min.css

# 输出 Source Map
sass --sourcemap=auto src/styles/main.scss dist/styles/main.css

# 指定加载路径
sass --load-path=node_modules/bootstrap/scss src/styles/main.scss dist/styles/main.css

# 编译所有文件
sass src/styles/:dist/styles/

# 显示帮助
sass --help
```

### 4.2 package.json 脚本配置

```json
{
  "scripts": {
    "css:dev": "postcss src/styles/main.css --watch -o dist/styles/",
    "css:build": "postcss src/styles/main.css --env production -o dist/styles/main.min.css",
    "css:lint": "stylelint \"src/**/*.{css,scss}\" --fix",
    "sass:dev": "sass --watch src/styles:dist/styles --style=expanded --sourcemap=auto",
    "sass:build": "sass src/styles:dist/styles --style=compressed --no-source-map",
    "sass:lint": "sass --no-source-map --no-emoji src/styles",
    "dev:css": "concurrently \"npm run css:dev\" \"npm run sass:dev\"",
    "build:css": "npm run css:build && npm run sass:build"
  }
}
```

---

## 5. 高级配置与技巧

### 5.1 CSS 新特性应用

#### 5.1.1 自定义媒体查询

```css
/* 定义自定义媒体查询 */
@custom-media --viewport-xs (width <= 320px);
@custom-media --viewport-sm (width <= 640px);
@custom-media --viewport-md (width <= 768px);
@custom-media --viewport-lg (width <= 1024px);
@custom-media --viewport-xl (width <= 1280px);

@custom-media --motion-ok (prefers-reduced-motion: no-preference);
@custom-media --motion (prefers-reduced-motion: reduce);
@custom-media --dark (prefers-color-scheme: dark);
@custom-media --light (prefers-color-scheme: light);

/* 使用自定义媒体查询 */
.container {
  padding: 16px;
}

@media (--viewport-md) {
  .container {
    padding: 24px;
  }
}

@media (--dark) {
  .container {
    background: #1a1a1a;
    color: #fff;
  }
}
```

#### 5.1.2 自定义选择器

```css
/* 定义自定义选择器 */
@custom-selector :--heading h1, h2, h3, h4, h5, h6;
@custom-selector :--btn button, .button;
@custom-selector :--input input, textarea, select;
@custom-selector :--card .card, [role="card"];

/* 使用自定义选择器 */
:--heading {
  font-weight: bold;
  line-height: 1.2;
}

:--btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 8px 16px;
  border-radius: 4px;
}

:--input {
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

:--card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
```

#### 5.1.3 嵌套规则

```css
/* 嵌套规则 */
.card {
  padding: 16px;

  & .header {
    font-size: 20px;
  }

  & .content {
    color: #333;

    & p {
      margin-bottom: 8px;
    }

    &:hover {
      background: #f5f5f5;
    }
  }

  &--featured {
    border: 2px solid gold;

    & .header {
      background: gold;
    }
  }

  @media (max-width: 768px) {
    padding: 8px;
  }
}
```

#### 5.1.4 颜色函数

```css
/* 颜色函数 */
:root {
  --base-color: #3b82f6;
  --light-color: color-mix(in srgb, var(--base-color) 20%, white);
  --dark-color: color-mix(in srgb, var(--base-color) 20%, black);
  --transparent-color: color-mix(in srgb, var(--base-color) 50%, transparent);
}

/* CSS 颜色空间 */
.element {
  /* OKLCH - 更自然的渐变 */
  background: oklch(65% 0.2 240);

  /* OKLCH 渐变 */
  background: linear-gradient(
    to right in oklch,
    oklch(70% 0.2 240),
    oklch(70% 0.2 120)
  );
}
```

### 5.2 响应式设计模式

```scss
// 响应式设计模式

// 1. 断点 Mixin
$breakpoints: (
  'xs': 0,
  'sm': 640px,
  'md': 768px,
  'lg': 1024px,
  'xl': 1280px,
  '2xl': 1536px,
);

@mixin breakpoint-up($breakpoint) {
  @media (min-width: map-get($breakpoints, $breakpoint)) {
    @content;
  }
}

@mixin breakpoint-down($breakpoint) {
  @media (max-width: map-get($breakpoints, $breakpoint) - 1px) {
    @content;
  }
}

// 2. 容器查询 Mixin
@mixin container-query($min, $max: null) {
  @if $max {
    @container (min-width: #{$min}) and (max-width: #{$max}) {
      @content;
    }
  } @else {
    @container (min-width: #{$min}) {
      @content;
    }
  }
}

// 3. 流体 typography
@function fluid-type($min-vw, $max-vw, $min-size, $max-size) {
  $slope: calc(($max-size - $min-size) / ($max-vw - $min-vw));
  $slope-vw: calc($slope * 100vw);
  $intercept: calc($min-size - $slope * $min-vw);
  @return clamp(#{min-size}, #{$slope-vw} + #{$intercept}, #{$max-size});
}

// 使用示例
.typography {
  font-size: fluid-type(320px, 1280px, 14px, 18px);

  h1 {
    font-size: fluid-type(320px, 1280px, 28px, 48px);
  }

  h2 {
    font-size: fluid-type(320px, 1280px, 22px, 36px);
  }
}

// 4. 网格系统
@mixin grid($columns: 12, $gap: 16px) {
  display: grid;
  grid-template-columns: repeat($columns, 1fr);
  gap: $gap;
}

@mixin col-span($span, $columns: 12) {
  grid-column: span $span;
}

// 5. Flex 工具类
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin flex-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

@mixin flex-column {
  display: flex;
  flex-direction: column;
}

// 6. 间距工具
$spacing-scale: (
  0: 0,
  1: 0.25rem,
  2: 0.5rem,
  3: 1rem,
  4: 1.5rem,
  5: 2rem,
  6: 3rem,
  8: 4rem,
);

@each $name, $value in $spacing-scale {
  .m-#{$name} { margin: $value; }
  .mt-#{$name} { margin-top: $value; }
  .mb-#{$name} { margin-bottom: $value; }
  .ml-#{$name} { margin-left: $value; }
  .mr-#{$name} { margin-right: $value; }
  .p-#{$name} { padding: $value; }
  .pt-#{$name} { padding-top: $value; }
  .pb-#{$name} { padding-bottom: $value; }
  .pl-#{$name} { padding-left: $value; }
  .pr-#{$name} { padding-right: $value; }
}
```

### 5.3 动画与过渡

```scss
// 动画与过渡工具

// 1. 过渡 Mixin
@mixin transition($properties: all, $duration: 0.2s, $easing: ease) {
  transition: $properties $duration $easing;
}

@mixin transition-multiple($transitions...) {
  transition: $transitions;
}

// 2. 动画 Mixin
@mixin animation($name, $duration: 0.3s, $timing: ease, $delay: 0s, $count: 1) {
  animation: {
    name: $name;
    duration: $duration;
    timing-function: $timing;
    delay: $delay;
    iteration-count: $count;
    fill-mode: forwards;
  }
}

// 3. 关键帧生成
@mixin keyframes($name) {
  @keyframes #{$name} {
    @content;
  }
}

// 4. 使用示例
.fade-in {
  @include animation(fadeIn, 0.3s, ease-out);
}

@include keyframes(fadeIn) {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

// 5. 悬停效果
@mixin hover-lift {
  transition: transform 0.2s ease, box-shadow 0.2s ease;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  }
}

@mixin hover-scale($scale: 1.05) {
  transition: transform 0.2s ease;

  &:hover {
    transform: scale($scale);
  }
}

// 6. 首屏动画延迟
@for $i from 1 through 10 {
  .stagger-#{$i} {
    animation-delay: #{$i * 0.1}s;
  }
}
```

---

## 6. 与同类技术对比

### 6.1 预处理器对比

| 特性 | Sass (SCSS) | LESS | Stylus |
|------|-------------|------|--------|
| **语法灵活性** | 严格 | 灵活 | 最灵活 |
| **变量符号** | `$` | `@` | 可省略 |
| **社区规模** | 最大 | 中等 | 较小 |
| **框架支持** | Bootstrap, many | Bootstrap 3 | Few |
| **编译速度** | 快 | 快 | 非常快 |
| **学习曲线** | 低 | 低 | 中 |
| **CSS 输出** | 清晰 | 清晰 | 可配置 |
| **循环支持** | 原生 | 需 JS | 原生 |
| **条件语句** | 原生 | 需 JS | 原生 |

### 6.2 PostCSS 与预处理器

```
┌─────────────────────────────────────────────────────────────────┐
│                   CSS 处理工具对比                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌───────────────────────┐     ┌───────────────────────┐      │
│   │      预处理器         │     │       PostCSS         │      │
│   │   Sass/LESS/Stylus    │     │                       │      │
│   ├───────────────────────┤     ├───────────────────────┤      │
│   │  • 新语法扩展         │     │  • CSS 标准转换       │      │
│   │  • 变量/混合器        │     │  • 自动前缀           │      │
│   │  • 嵌套规则           │     │  • 代码优化           │      │
│   │  • 继承机制           │     │  • 新语法支持         │      │
│   │  • 函数库             │     │  • 模块化             │      │
│   └───────────────────────┘     └───────────────────────┘      │
│            │                              │                     │
│            │                              │                     │
│            ▼                              ▼                     │
│   ┌─────────────────────────────────────────────────────┐      │
│   │                    输出标准 CSS                      │      │
│   └─────────────────────────────────────────────────────┘      │
│                                                                  │
│   两者可以协同使用：预处理器 → PostCSS → 最终 CSS                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 6.3 CSS Modules vs BEM

| 特性 | CSS Modules | BEM |
|------|-----------|-----|
| **实现方式** | 编译时转换 | 命名约定 |
| **作用域** | 自动局部化 | 依赖命名规范 |
| **学习成本** | 低 | 中 |
| **迁移成本** | 中等 | 低 |
| **调试** | 需要 sourcemap | 直接可读 |
| **与现有代码** | 需要改造 | 可以渐进采用 |
| **框架支持** | React/Vue 生态 | 通用 |

### 6.4 选型建议

```
CSS 技术选型决策树:

需要使用新的 CSS 语法吗？
│
├─ 是 → 使用 PostCSS-preset-env
│        │
│        └─ 需要兼容旧浏览器吗？
│                 │
│                 ├─ 是 → 添加 autoprefixer
│                 └─ 否 → 单独使用 preset-env
│
└─ 否 → 需要高级编程能力吗？
         │
         ├─ 是 → 使用 Sass/SCSS + PostCSS
         │        │
         │        └─ 需要模块化吗？
         │                 │
         │                 ├─ 是 → 使用 CSS Modules
         │                 └─ 否 → 使用 BEM 命名约定
         │
         └─ 否 → 使用原生 CSS + PostCSS
```

---

## 7. 常见问题与解决方案

### 7.1 PostCSS 问题

#### 7.1.1 插件执行顺序

**问题**：插件执行顺序不正确导致问题。

**解决方案**：PostCSS 插件按数组顺序执行，后面的插件处理前面插件的输出。

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    // 1. 先处理其他语法（如 SCSS）
    require('postcss-scss'),

    // 2. 然后处理新语法
    require('postcss-preset-env')({ stage: 2 }),

    // 3. 添加前缀
    require('autoprefixer'),

    // 4. 最后压缩
    require('cssnano'),
  ],
};
```

#### 7.1.2 Source Map 问题

**问题**：Source Map 与实际代码不匹配。

**解决方案**：确保所有 loader 和插件的 Source Map 配置一致。

```javascript
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: 'css-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'postcss-loader',
            options: {
              sourceMap: true,
              postcssOptions: {
                plugins: [/* ... */],
              },
            },
          },
        ],
      },
    ],
  },
};
```

### 7.2 Sass 问题

#### 7.2.1 @import vs @use

**问题**：@import 已被弃用。

**解决方案**：使用 @use 和 @forward。

```scss
// _variables.scss
$primary-color: #3b82f6;
$font-stack: 'Inter', sans-serif;

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
  font-family: variables.$font-stack;
}

.container {
  @include mixins.flex-center;
}
```

#### 7.2.2 路径问题

**问题**：Sass 找不到导入的文件。

**解决方案**：配置 includePaths。

```javascript
// vite.config.ts
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/_variables.scss";`,
        includePaths: [
          path.resolve(__dirname, 'src/styles'),
          'node_modules',
        ],
      },
    },
  },
});
```

### 7.3 CSS Modules 问题

#### 7.3.1 类名组合

**问题**：如何组合多个 CSS Modules 类名。

**解决方案**：使用 classnames 库或模板字符串。

```tsx
// React 示例
import styles from './Button.module.css';
import cn from 'classnames';

// 方法 1：模板字符串
<button className={`${styles.button} ${styles.primary}`}>

// 方法 2：数组 + filter
<button className={[styles.button, styles.primary].filter(Boolean).join(' ')}>

// 方法 3：classnames 库
<button className={cn(styles.button, styles.primary)}>

// 方法 4：条件类名
<button className={cn(styles.button, { [styles.primary]: isPrimary })}>
```

#### 7.3.2 全局样式

**问题**：CSS Modules 中需要使用全局样式。

**解决方案**：使用 :global 选择器。

```css
/* Button.module.css */

/* 整个文件是局部的 */
.button {
  composes: global(btn) from "./global.css";
}

/* 或者使用 :global */
:global(.external-style) {
  color: blue;
}

/* 混合使用 */
:local(.localClass) {
  color: red;
}

:global(.external) {
  color: blue;
}
```

---

## 8. 实战项目示例

### 8.1 现代 CSS 工作流

#### 8.1.1 项目结构

```
modern-css-project/
├── src/
│   ├── styles/
│   │   ├── _variables.scss
│   │   ├── _mixins.scss
│   │   ├── _functions.scss
│   │   ├── _base.scss
│   │   ├── _components.scss
│   │   ├── _utilities.scss
│   │   └── main.scss
│   ├── components/
│   └── pages/
├── public/
├── package.json
├── postcss.config.js
├── stylelint.config.js
└── vite.config.ts
```

#### 8.1.2 完整配置

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    require('tailwindcss'),
    require('postcss-preset-env')({
      stage: 2,
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'custom-selectors': true,
        'color-functional-notation': true,
      },
    }),
    require('autoprefixer'),
    require('cssnano'),
  ],
};
```

```scss
// src/styles/_variables.scss

// 颜色系统
$colors: (
  primary: (
    50: #eff6ff,
    100: #dbeafe,
    200: #bfdbfe,
    300: #93c5fd,
    400: #60a5fa,
    500: #3b82f6,
    600: #2563eb,
    700: #1d4ed8,
    800: #1e40af,
    900: #1e3a8a,
  ),
  neutral: (
    50: #fafafa,
    100: #f4f4f5,
    200: #e4e4e7,
    300: #d4d4d8,
    400: #a1a1aa,
    500: #71717a,
    600: #52525b,
    700: #3f3f46,
    800: #27272a,
    900: #18181b,
  ),
  success: #10b981,
  warning: #f59e0b,
  danger: #ef4444,
);

// 断点
$breakpoints: (
  xs: 0,
  sm: 640px,
  md: 768px,
  lg: 1024px,
  xl: 1280px,
  2xl: 1536px,
);

// 间距
$spacing: (
  0: 0,
  px: 1px,
  0.5: 0.125rem,
  1: 0.25rem,
  1.5: 0.375rem,
  2: 0.5rem,
  2.5: 0.625rem,
  3: 0.75rem,
  3.5: 0.875rem,
  4: 1rem,
  5: 1.25rem,
  6: 1.5rem,
  7: 1.75rem,
  8: 2rem,
  9: 2.25rem,
  10: 2.5rem,
  12: 3rem,
  14: 3.5rem,
  16: 4rem,
  20: 5rem,
  24: 6rem,
  28: 7rem,
  32: 8rem,
);

// 字体
$font-sizes: (
  xs: 0.75rem,
  sm: 0.875rem,
  base: 1rem,
  lg: 1.125rem,
  xl: 1.25rem,
  2xl: 1.5rem,
  3xl: 1.875rem,
  4xl: 2.25rem,
  5xl: 3rem,
  6xl: 3.75rem,
  7xl: 4.5rem,
);

// 阴影
$shadows: (
  sm: 0 1px 2px 0 rgb(0 0 0 / 0.05),
  DEFAULT: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1),
  md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1),
  lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1),
  xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1),
  2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25),
  inner: inset 0 2px 4px 0 rgb(0 0 0 / 0.05),
);

// 圆角
$radii: (
  none: 0,
  sm: 0.125rem,
  DEFAULT: 0.25rem,
  md: 0.375rem,
  lg: 0.5rem,
  xl: 0.75rem,
  2xl: 1rem,
  3xl: 1.5rem,
  full: 9999px,
);

// 过渡
$transitions: (
  fast: 150ms,
  DEFAULT: 200ms,
  slow: 300ms,
  slower: 500ms,
);

// z-index
$z-index: (
  0: 0,
  10: 10,
  20: 20,
  30: 30,
  40: 40,
  50: 50,
  auto: auto,
  dropdown: 1000,
  sticky: 1020,
  fixed: 1030,
  modal-backdrop: 1040,
  modal: 1050,
  popover: 1060,
  tooltip: 1070,
);
```

```scss
// src/styles/_mixins.scss
@use 'variables' as var;

// 响应式断点
@mixin respond-to($breakpoint) {
  @if $breakpoint == xs {
    @media (min-width: var.$breakpoints) { @content; }
  }
}

// Flexbox 工具
@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin flex-between {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

@mixin flex-column {
  display: flex;
  flex-direction: column;
}

// Grid 工具
@mixin grid($columns: 12, $gap: 16px) {
  display: grid;
  grid-template-columns: repeat($columns, 1fr);
  gap: $gap;
}

// 文字截断
@mixin text-truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

@mixin text-clamp($lines: 2) {
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: $lines;
  overflow: hidden;
}

// 阴影
@mixin shadow($size: 'DEFAULT') {
  box-shadow: map-get(var.$shadows, $size);
}

// 按钮基础
@mixin button-base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 0.25rem;
  cursor: pointer;
  transition: all 200ms ease;
  border: none;
  outline: none;

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
```

```scss
// src/styles/main.scss
@use 'variables' as var;
@use 'mixins' as mix;

// Tailwind CSS
@tailwind base;
@tailwind components;
@tailwind utilities;

// 自定义样式
body {
  font-family: 'Inter', system-ui, -apple-system, sans-serif;
  color: map-get(map-get(var.$colors, neutral), 900);
  background-color: map-get(map-get(var.$colors, neutral), 50);
}

h1, h2, h3, h4, h5, h6 {
  font-weight: 600;
  line-height: 1.2;
}

a {
  color: map-get(map-get(var.$colors, primary), 600);
  text-decoration: none;

  &:hover {
    color: map-get(map-get(var.$colors, primary), 700);
  }
}

img, video {
  max-width: 100%;
  height: auto;
}
```

### 8.2 组件示例

```vue
<!-- Button.vue -->
<template>
  <button
    :class="[
      'btn',
      `btn--${variant}`,
      `btn--${size}`,
      { 'btn--loading': loading, 'btn--disabled': disabled }
    ]"
    :disabled="disabled || loading"
    @click="handleClick"
  >
    <span v-if="loading" class="btn__spinner"></span>
    <span :class="{ 'btn__content--hidden': loading }">
      <slot>Button</slot>
    </span>
  </button>
</template>

<script setup lang="ts">
import { defineProps, defineEmits } from 'vue';

const props = defineProps<{
  variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
}>();

const emit = defineEmits<{
  (e: 'click', event: MouseEvent): void;
}>();

const handleClick = (event: MouseEvent) => {
  if (!props.disabled && !props.loading) {
    emit('click', event);
  }
};
</script>

<style scoped lang="scss">
@use '@/styles/variables' as var;
@use '@/styles/mixins' as mix;

.btn {
  @include mix.button-base;

  // Variants
  &--primary {
    background-color: map-get(map-get(var.$colors, primary), 600);
    color: white;

    &:hover:not(:disabled) {
      background-color: map-get(map-get(var.$colors, primary), 700);
    }
  }

  &--secondary {
    background-color: map-get(map-get(var.$colors, neutral), 200);
    color: map-get(map-get(var.$colors, neutral), 900);

    &:hover:not(:disabled) {
      background-color: map-get(map-get(var.$colors, neutral), 300);
    }
  }

  &--danger {
    background-color: var.$danger;
    color: white;

    &:hover:not(:disabled) {
      background-color: darken(var.$danger, 10%);
    }
  }

  &--ghost {
    background-color: transparent;
    color: map-get(map-get(var.$colors, primary), 600);

    &:hover:not(:disabled) {
      background-color: map-get(map-get(var.$colors, primary), 50);
    }
  }

  // Sizes
  &--sm {
    padding: 0.25rem 0.75rem;
    font-size: 0.75rem;
  }

  &--md {
    padding: 0.5rem 1rem;
    font-size: 0.875rem;
  }

  &--lg {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
  }

  // Loading state
  &--loading {
    position: relative;
  }

  &__spinner {
    position: absolute;
    width: 1rem;
    height: 1rem;
    border: 2px solid currentColor;
    border-top-color: transparent;
    border-radius: 50%;
    animation: spin 0.6s linear infinite;
  }

  &__content--hidden {
    visibility: hidden;
  }
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}
</style>
```

---

> [!TIP]
> **CSS 工程化推荐栈**：PostCSS + Tailwind CSS + Autoprefixer + CSSnano + Stylelint。这套组合覆盖了样式转换、浏览器兼容、自动优化和代码质量检查。对于需要更强大编程能力的项目，可以添加 Sass/SCSS 作为预处理器层。
