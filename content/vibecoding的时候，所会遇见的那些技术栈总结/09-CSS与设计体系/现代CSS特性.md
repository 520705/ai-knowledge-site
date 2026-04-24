# 现代 CSS 特性完全指南

> [!NOTE]
> 本文档涵盖 Container Queries、Cascade Layers、:has() 选择器、色彩函数、CSS 作用域等现代 CSS 特性，以及浏览器兼容性策略和深度实战技巧。

---

## 技术概述与定位

### CSS 的演进历程

CSS（层叠样式表）自1996年诞生以来，经历了从简单的样式声明到现代复杂样式系统的漫长演进。理解现代 CSS 特性的定位，需要先回顾其发展脉络：

**CSS 1.0（1996年）**：定义了字体、颜色、文本属性等基础样式。

**CSS 2.0（1998年）**：引入了盒模型、定位、浮动等核心概念，奠定了现代布局的基础。

**CSS 2.1（2009-2011年）**：对 CSS 2.0 的修订，解决了实现不一致问题。

**CSS 3（2010年代至今）**：模块化发展，引入了选择器、背景边框、文本效果、2D/3D变换、动画、多列布局、Flexbox、Grid 等众多特性。

**现代 CSS（2020年代）**：进入了"声明式 UI"时代，Container Queries、Cascade Layers、:has()、CSS Houdini、OKLCH 色彩空间等特性让 CSS 具备了前所未有的能力。

### 现代 CSS 的核心定位

现代 CSS 的定位可以从以下几个维度理解：

**组件化时代**：传统 CSS 的全局作用域在大型应用中造成样式冲突和难以维护的问题。现代 CSS 通过 CSS Modules、CSS-in-JS、Cascade Layers 等技术实现了真正的组件级作用域。

**声明式交互**：过去需要 JavaScript 实现的复杂交互效果，现在可以通过纯 CSS 实现。例如 `:has()` 选择器让父元素能够感知子元素状态，Container Queries 让组件响应自身容器而非视口。

**性能优先**：GPU 加速动画、will-change 提示、content-visibility 等特性让 CSS 成为高性能 UI 的首选方案。

**设计系统友好**：Custom Properties（CSS 变量）与设计 token 系统天然契合，color-mix()、OKLCH 等色彩函数让动态主题成为标准配置。

### 现代 CSS 的浏览器支持现状

截至 2026 年，现代 CSS 特性的浏览器支持已达到生产就绪阶段：

| 特性 | Chrome | Firefox | Safari | Edge | 支持率 |
|------|--------|---------|--------|------|--------|
| Custom Properties | 49+ | 31+ | 9.1+ | 15+ | 97%+ |
| Flexbox | 21+ | 28+ | 9+ | 11+ | 99%+ |
| CSS Grid | 57+ | 52+ | 10.1+ | 16+ | 98%+ |
| :has() | 105+ | 121+ | 15.4+ | 105+ | 92%+ |
| Container Queries | 105+ | 110+ | 16+ | 105+ | 90%+ |
| Cascade Layers | 99+ | 97+ | 15.4+ | 99+ | 92%+ |
| color-mix() | 111+ | 113+ | 16.2+ | 111+ | 88%+ |
| OKLCH | 119+ | 113+ | 16.4+ | 119+ | 85%+ |
| Subgrid | 117+ | 71+ | 16+ | 117+ | 88%+ |
| View Transitions | 111+ | 老版本不支持 | 18+ | 111+ | 78% |

### 技术选型建议

**对于新项目**：可以直接采用所有现代 CSS 特性，因为目标用户的浏览器通常会自动更新。

**对于企业级项目**：建议采用渐进增强策略，核心样式使用广泛支持的基础特性，高级特性使用 @supports 条件加载。

**对于需要兼容旧浏览器的项目**：使用 PostCSS 的 autoprefixer、CSS Modules、或 CSS-in-JS 方案来处理兼容性问题。

### 为什么现代 CSS 值得关注

在 vibecoding 时代，CSS 的角色正在发生根本性转变。传统的 CSS 开发模式需要依赖大量的 JavaScript 来实现复杂的交互效果，但现代 CSS 让这种模式发生了革命性的变化：

**声明式优先**：通过 `:has()` 选择器，我们可以实现过去需要事件监听器才能完成的交互逻辑。例如，当表单包含无效输入时自动显示错误状态，当列表包含特定项目时改变样式等。

**组件自包含**：Container Queries 的出现让组件真正成为自包含的单元。一个卡片组件可以在侧边栏中显示为紧凑布局，在主内容区显示为宽版布局，而无需任何 JavaScript 逻辑。

**性能原生支持**：现代 CSS 内置了大量性能优化特性，如 content-visibility 可以自动跳过离屏内容的渲染，will-change 可以提示浏览器提前优化特定元素。

**设计系统集成**：CSS 变量与设计 token 系统的结合让主题切换、组件变体、品牌定制等需求变得前所未有的简单。

---

## 完整安装与配置

### 开发环境准备

现代 CSS 开发通常需要以下工具链：

#### Node.js 环境配置

```bash
# 检查 Node.js 版本（建议 Node.js 18+）
node --version
npm --version

# 创建项目目录
mkdir my-css-project && cd my-css-project

# 初始化 npm 项目
npm init -y
```

#### Vite 项目配置

Vite 是现代前端开发的首选构建工具，它对现代 CSS 特性有良好的支持：

```bash
# 创建 Vite 项目
npm create vite@latest my-app -- --template vanilla-ts

# 进入目录
cd my-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

```typescript
// vite.config.ts - Vite 配置文件
import { defineConfig } from 'vite';
import autoprefixer from 'autoprefixer';

export default defineConfig({
  css: {
    // PostCSS 插件配置
    postcss: {
      plugins: [
        autoprefixer(), // 自动添加浏览器前缀
      ],
    },
    // CSS Modules 配置
    modules: {
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
    // 开发者工具
    devSourcemap: true,
  },
  // 构建优化
  build: {
    cssCodeSplit: true,
    sourcemap: true,
    target: 'esnext',
  },
});
```

#### Tailwind CSS 配置

如果使用 Tailwind CSS 作为基础框架：

```bash
# 安装 Tailwind CSS
npm install -D tailwindcss postcss autoprefixer

# 初始化配置文件
npx tailwindcss init -p
```

```javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      // 自定义设计 token
      colors: {
        primary: {
          50: 'oklch(97% 0.02 250)',
          100: 'oklch(94% 0.04 250)',
          500: 'oklch(60% 0.2 250)',
          900: 'oklch(30% 0.15 250)',
        },
      },
      // 自定义断点
      screens: {
        '3xl': '1920px',
      },
    },
  },
  plugins: [],
};
```

```css
/* src/styles.css - Tailwind 入口文件 */
@tailwind base;
@tailwind components;
@tailwind utilities;

/* 自定义基础样式 */
@layer base {
  :root {
    --color-primary: oklch(60% 0.2 250);
  }
  
  body {
    @apply antialiased;
  }
}

/* 自定义组件样式 */
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-primary-500 text-white rounded-lg;
    @apply hover:bg-primary-600 transition-colors;
  }
}

/* 自定义工具类 */
@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

### PostCSS 配置

PostCSS 是现代 CSS 处理的核心工具：

```javascript
// postcss.config.js
export default {
  plugins: {
    // Tailwind CSS
    tailwindcss: {},
    
    // Autoprefixer - 自动添加浏览器前缀
    autoprefixer: {
      flexbox: 'no-2009',
      grid: true,
    },
    
    // CSS Nesting - CSS 嵌套支持
    'postcss-nesting': {},
    
    // 颜色函数 polyfill
    'postcss-color-functional-notation': {},
    
    // 自定义属性 polyfill（如果需要支持旧浏览器）
    'postcss-custom-properties': {
      preserve: true,
    },
  },
};
```

### IDE 配置

#### VS Code 设置

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.quickSuggestions": {
    "strings": true
  },
  "files.associations": {
    "*.css": "postcss"
  },
  "css.lint.compatibleVendorPrefixes": "warning",
  "css.lint.validProperties": [
    "container-type",
    "container-name",
    "view-transition-name",
    "scroll-timeline"
  ],
  "stylelint.validate": [
    "css",
    "postcss"
  ]
}
```

#### 推荐的 VS Code 扩展

- **PostCSS Language Support**：PostCSS 语法高亮
- **Stylelint**：CSS/SCSS Lint 工具
- **CSS Variables**：CSS 变量智能提示
- **Prettier - Code formatter**：代码格式化

### 浏览器开发工具配置

现代 Chrome DevTools 提供了强大的 CSS 调试能力：

1. **打开 CSS 概览面板**：通过 More Tools > CSS Overview 查看页面使用的颜色、字体、选择器统计
2. **Elements 面板增强**：直接编辑 CSS 变量、查看计算后的值
3. **Sources 面板**：设置断点调试 CSS 动画
4. **Rendering 面板**：模拟 prefers-color-scheme、prefers-reduced-motion 等媒体查询

### CSS 工程化工具链

现代 CSS 开发推荐使用完整的工程化工具链来确保代码质量和兼容性：

```bash
# 安装核心工具
npm install -D \
  postcss \
  autoprefixer \
  postcss-nesting \
  postcss-custom-properties \
  cssnano \
  stylelint \
  prettier

# 安装现代 CSS 插件
npm install -D \
  postcss-preset-env \
  postcss-color-functional-notation \
  stylelint-config-recommended \
  stylelint-config-prettier
```

```javascript
// postcss.config.js - 完整配置
export default {
  plugins: [
    // CSS Nesting 支持
    'postcss-nesting',
    
    // 现代 CSS 语法转换
    ['postcss-preset-env', {
      stage: 2,
      features: {
        'nesting-rules': true,
        'custom-media-queries': true,
        'color-functional-notation': true,
      },
    }],
    
    // 自动前缀
    ['autoprefixer', {
      flexbox: 'no-2009',
      grid: true,
    }],
    
    // 生产环境压缩
    ...(process.env.NODE_ENV === 'production' ? [
      ['cssnano', {
        preset: 'default',
      }],
    ] : []),
  ],
};
```

```javascript
// .stylelintrc.json - Stylelint 配置
{
  "extends": [
    "stylelint-config-recommended",
    "stylelint-config-prettier"
  ],
  "rules": {
    "at-rule-no-unknown": [
      true,
      {
        "ignoreAtRules": [
          "tailwind",
          "apply",
          "layer",
          "responsive",
          "screen"
        ]
      }
    ],
    "property-no-unknown": [
      true,
      {
        "ignoreProperties": [
          "container-type",
          "container-name",
          "view-transition-name"
        ]
      }
    ]
  }
}
```

---

## 核心概念详解

### Custom Properties（CSS 变量）

#### 深度原理解析

CSS Custom Properties 不仅是简单的变量替换，更是 CSS 架构的革命性变革。其核心机制涉及：

**继承与作用域**：CSS 变量遵循继承规则，可以在任何选择器中定义，子元素自动继承父元素的值：

```css
/* 全局变量 - :root 伪类 */
:root {
  --primary-color: #3b82f6;
  --spacing-unit: 8px;
  --border-radius: 4px;
}

/* 组件级变量 */
.card {
  --card-padding: 16px;
  --card-bg: white;
}

/* 主题变体 */
.card.dark {
  --card-bg: #1e293b;
  --card-text: #f1f5f9;
}

/* 响应式变量 */
@media (min-width: 768px) {
  :root {
    --spacing-unit: 10px;
    --font-size-base: 17px;
  }
}
```

**计算与组合**：CSS 变量可以在 calc() 函数中使用，实现动态计算：

```css
:root {
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;
  
  /* 组合计算 */
  --card-padding: calc(var(--spacing-md) + var(--spacing-sm));
  --card-gap: calc(var(--spacing-md) * 2);
  
  /* 复杂计算 */
  --container-width: calc(100% - var(--spacing-xl) * 2);
  --grid-columns: 12;
  --grid-gap: 24px;
  --column-width: calc((var(--container-width) - (var(--grid-gap) * (var(--grid-columns) - 1))) / var(--grid-columns));
}

/* 使用计算值 */
.grid {
  display: grid;
  grid-template-columns: repeat(var(--grid-columns), 1fr);
  gap: var(--grid-gap);
}
```

**备用值机制**：var() 函数支持备用值语法：

```css
/* 基础语法 */
.element {
  color: var(--theme-color);                    /* 无备用值 */
  color: var(--theme-color, #3b82f6);          /* 带备用值 */
  color: var(--theme-color, var(--fallback));   /* 备用值也可以是变量 */
  color: var(--missing, red, blue);            /* 多个备用值 */
}

/* 复杂备用值 */
.card {
  /* 备用值中可以使用任何有效 CSS 值 */
  background: var(--card-gradient, linear-gradient(to right, #667eea, #764ba2));
  font-size: var(--card-font-size, clamp(14px, 2vw, 18px));
  border-radius: var(--radius, var(--spacing-sm));
}
```

#### JavaScript 操作

CSS 变量可以通过 JavaScript 动态读取和修改：

```javascript
// 获取变量值
const element = document.querySelector('.card');
const styles = getComputedStyle(element);

// 获取具体值
const primaryColor = styles.getPropertyValue('--primary-color').trim();
console.log('Primary color:', primaryColor);

// 获取带单位的值
const spacing = styles.getPropertyValue('--spacing').trim();

// 直接在 style 对象上操作
element.style.setProperty('--primary-color', '#10b981');

// 使用 CSSUnitValue API（现代浏览器）
const value = styles.getPropertyValue('--spacing');
const parsed = CSS.parseComponentValue(value);
console.log(parsed.value, parsed.unit);

// 批量更新多个变量
element.style.cssText = `
  --primary-color: #10b981;
  --secondary-color: #6366f1;
  --spacing: 24px;
`;

// 或者使用 updateSettings
element.style.updateSettings?.({
  '--primary-color': '#10b981',
});

// 监听 CSS 变量变化（CSS Houdini API）
if (CSS.registerProperty) {
  CSS.registerProperty({
    name: '--my-color',
    syntax: '<color>',
    inherits: false,
    initialValue: '#3b82f6',
  });
}

// 观察属性变化（部分浏览器支持）
const observer = new CSSPropertyRuleObserver(element);
observer.observe('--primary-color', (oldValue, newValue) => {
  console.log('Color changed from', oldValue, 'to', newValue);
});
```

#### 设计 Token 系统

CSS 变量是构建设计 Token 系统的理想选择：

```css
:root {
  /* ===== 色彩系统 ===== */
  --color-brand-50: oklch(97% 0.02 250);
  --color-brand-100: oklch(94% 0.04 250);
  --color-brand-200: oklch(88% 0.08 250);
  --color-brand-300: oklch(76% 0.14 250);
  --color-brand-400: oklch(62% 0.20 250);
  --color-brand-500: oklch(52% 0.22 250);
  --color-brand-600: oklch(44% 0.20 250);
  --color-brand-700: oklch(36% 0.18 250);
  --color-brand-800: oklch(28% 0.14 250);
  --color-brand-900: oklch(22% 0.10 250);

  /* 语义化颜色映射 */
  --color-background: var(--color-brand-50);
  --color-foreground: var(--color-brand-900);
  --color-primary: var(--color-brand-500);
  --color-primary-hover: var(--color-brand-600);
  --color-accent: var(--color-brand-400);
  
  /* 暗色模式 */
  @media (prefers-color-scheme: dark) {
    --color-background: var(--color-brand-900);
    --color-foreground: var(--color-brand-50);
    --color-primary: var(--color-brand-400);
  }
  
  /* ===== 字体系统 ===== */
  --font-family-sans: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  --font-family-serif: 'Georgia', 'Times New Roman', serif;
  --font-family-mono: 'JetBrains Mono', 'Fira Code', monospace;
  
  --font-size-xs: 0.75rem;    /* 12px */
  --font-size-sm: 0.875rem;   /* 14px */
  --font-size-base: 1rem;     /* 16px */
  --font-size-lg: 1.125rem;  /* 18px */
  --font-size-xl: 1.25rem;   /* 20px */
  --font-size-2xl: 1.5rem;   /* 24px */
  --font-size-3xl: 1.875rem; /* 30px */
  --font-size-4xl: 2.25rem;  /* 36px */
  
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
  
  --line-height-tight: 1.25;
  --line-height-normal: 1.5;
  --line-height-relaxed: 1.75;
  
  /* ===== 间距系统 ===== */
  --spacing-0: 0;
  --spacing-1: 0.25rem;   /* 4px */
  --spacing-2: 0.5rem;    /* 8px */
  --spacing-3: 0.75rem;   /* 12px */
  --spacing-4: 1rem;      /* 16px */
  --spacing-5: 1.25rem;   /* 20px */
  --spacing-6: 1.5rem;    /* 24px */
  --spacing-8: 2rem;      /* 32px */
  --spacing-10: 2.5rem;   /* 40px */
  --spacing-12: 3rem;     /* 48px */
  --spacing-16: 4rem;     /* 64px */
  
  /* ===== 圆角系统 ===== */
  --radius-none: 0;
  --radius-sm: 0.25rem;   /* 4px */
  --radius-md: 0.5rem;    /* 8px */
  --radius-lg: 0.75rem;   /* 12px */
  --radius-xl: 1rem;      /* 16px */
  --radius-2xl: 1.5rem;  /* 24px */
  --radius-full: 9999px;
  
  /* ===== 阴影系统 ===== */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  
  /* ===== 动画系统 ===== */
  --duration-fast: 100ms;
  --duration-normal: 200ms;
  --duration-slow: 300ms;
  --duration-slower: 500ms;
  
  --ease-in: cubic-bezier(0.4, 0, 1, 1);
  --ease-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
  --ease-bounce: cubic-bezier(0.34, 1.56, 0.64, 1);
  
  /* ===== Z-Index 系统 ===== */
  --z-base: 0;
  --z-dropdown: 1000;
  --z-sticky: 1020;
  --z-fixed: 1030;
  --z-modal-backdrop: 1040;
  --z-modal: 1050;
  --z-popover: 1060;
  --z-tooltip: 1070;
  --z-toast: 1080;
}
```

#### 高级变量技巧

```css
/* ===== 条件变量 ===== */
.theme-button {
  --button-bg: var(--color-primary);
  --button-color: white;
}

.theme-button:hover {
  --button-bg: var(--color-primary-hover);
}

/* ===== 响应式变量覆盖 ===== */
.container {
  --columns: 1;
  --gap: 16px;
}

@media (min-width: 768px) {
  .container {
    --columns: 2;
    --gap: 24px;
  }
}

@media (min-width: 1024px) {
  .container {
    --columns: 3;
    --gap: 32px;
  }
}

/* ===== 组件变体系统 ===== */
.button {
  --btn-padding: 12px 24px;
  --btn-font-size: 14px;
  --btn-radius: 8px;
  --btn-shadow: none;
}

.button--small {
  --btn-padding: 8px 16px;
  --btn-font-size: 12px;
  --btn-radius: 6px;
}

.button--large {
  --btn-padding: 16px 32px;
  --btn-font-size: 16px;
  --btn-radius: 12px;
}

.button--primary {
  --btn-bg: var(--color-primary);
  --btn-color: white;
}

.button--secondary {
  --btn-bg: transparent;
  --btn-color: var(--color-primary);
  --btn-shadow: inset 0 0 0 2px var(--color-primary);
}

/* 应用变量 */
.button {
  padding: var(--btn-padding);
  font-size: var(--btn-font-size);
  border-radius: var(--btn-radius);
  box-shadow: var(--btn-shadow);
  background: var(--btn-bg);
  color: var(--btn-color);
}
```

### Container Queries（容器查询）

#### 深度原理解析

Container Queries 是 CSS 响应式设计的重大突破，它允许组件根据自身容器的尺寸来调整样式，而非仅仅依赖视口尺寸：

**核心概念**：
- **Container（容器）**：定义了查询上下文的父元素
- **Contained Element（被包含元素）**：响应容器尺寸变化的子元素
- **Containment（包含）**：通过 `container-type` 属性启用

```css
/* 定义容器 - 方式1：简写 */
.card-container {
  container: card / inline-size; /* container-name: card; container-type: inline-size */
}

/* 定义容器 - 方式2：分开写 */
.card-wrapper {
  container-type: inline-size;   /* 监听内联方向尺寸 */
  container-name: card;            /* 命名容器 */
}

/* 块方向容器 */
.sidebar {
  container-type: block-size;     /* 监听块方向尺寸 */
  container-name: sidebar;
}

/* 两个方向都监听 */
.panel {
  container-type: size;           /* 监听宽高 */
  container-name: panel;
}

/* 定义无尺寸约束的容器（仅命名空间）*/
.container {
  container-type: normal;         /* 不创建包含上下文 */
  container-name: my-container;
}
```

#### 容器查询语法

```css
/* 默认样式（无查询条件时） */
.product-card {
  display: flex;
  flex-direction: column;
}

/* 基础查询：容器宽度 >= 300px */
@container (min-width: 300px) {
  .product-card {
    flex-direction: row;
  }
}

/* 基础查询：容器宽度 >= 500px */
@container (min-width: 500px) {
  .product-card {
    gap: 32px;
    padding: 24px;
  }
  
  .product-card .title {
    font-size: 1.5rem;
  }
}

/* 命名容器查询 */
@container card (min-width: 400px) {
  .product-card {
    flex-direction: row;
  }
}

/* 范围查询 */
@container (300px <= width <= 600px) {
  .product-card {
    /* 宽度在 300-600px 之间时 */
  }
}

/* 组合查询 */
@container (min-width: 300px) and (max-width: 600px) {
  .product-card {
    /* 组合条件 */
  }
}

/* 否定查询 */
@container not (min-width: 300px) {
  .product-card {
    /* 宽度 < 300px */
  }
}

/* 复杂条件 */
@container (aspect-ratio > 1) {
  .product-card {
    /* 横向布局 */
  }
}
```

#### 实战：响应式卡片组件

```css
/* ===== 基础容器定义 ===== */
.media-card {
  container-type: inline-size;
  container-name: media-card;
}

/* ===== 默认样式（最小尺寸）====== */
.card-inner {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.card-image {
  width: 100%;
  aspect-ratio: 16 / 9;
  object-fit: cover;
  border-radius: var(--radius-lg);
}

.card-content {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
}

.card-title {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-semibold);
  line-height: var(--line-height-tight);
}

.card-description {
  font-size: var(--font-size-sm);
  color: var(--color-text-muted);
  line-height: var(--line-height-normal);
}

.card-actions {
  display: flex;
  gap: 8px;
  margin-top: 8px;
}

/* ===== 小容器（>= 320px）====== */
@container media-card (min-width: 320px) {
  .card-actions {
    flex-direction: row;
  }
}

/* ===== 中等容器（>= 400px）====== */
@container media-card (min-width: 400px) {
  .card-inner {
    flex-direction: row;
    align-items: flex-start;
  }
  
  .card-image {
    width: 45%;
    aspect-ratio: 1;
    border-radius: var(--radius-xl);
  }
  
  .card-content {
    flex: 1;
    padding: 20px;
  }
  
  .card-title {
    font-size: var(--font-size-xl);
  }
}

/* ===== 大容器（>= 600px）====== */
@container media-card (min-width: 600px) {
  .card-inner {
    gap: 32px;
  }
  
  .card-image {
    width: 50%;
  }
  
  .card-content {
    padding: 32px;
    justify-content: center;
  }
  
  .card-title {
    font-size: var(--font-size-2xl);
    margin-bottom: 12px;
  }
  
  .card-description {
    font-size: var(--font-size-base);
  }
  
  .card-actions {
    margin-top: 20px;
  }
}

/* ===== 超大容器（>= 800px）====== */
@container media-card (min-width: 800px) {
  .card-inner {
    gap: 48px;
  }
  
  .card-content {
    padding: 40px;
  }
  
  .card-title {
    font-size: var(--font-size-3xl);
  }
  
  .card-description {
    font-size: var(--font-size-lg);
    max-width: 600px;
  }
}
```

#### 容器查询与媒体查询的对比

| 维度 | 媒体查询 | 容器查询 |
|------|----------|----------|
| **查询基准** | 视口（viewport） | 父容器（container） |
| **适用场景** | 页面整体布局 | 可复用组件 |
| **组件独立性** | 依赖页面上下文 | 完全独立 |
| **响应式粒度** | 页面级 | 组件级 |
| **使用示例** | 页面三栏布局 | 卡片组件自适应 |

```css
/* 媒体查询方式 - 组件依赖于外部页面布局 */
@media (min-width: 768px) {
  .card {
    flex-direction: row; /* 假设卡片在侧边栏时会出错 */
  }
}

/* 容器查询方式 - 组件自主响应 */
.card-wrapper {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    flex-direction: row; /* 无论在哪里都正确 */
  }
}
```

#### 容器查询实战场景

```css
/* ===== 场景1：文章卡片 ===== */
.article-card-wrapper {
  container-type: inline-size;
  container-name: article;
}

.article-card {
  display: grid;
  gap: 16px;
}

.article-card__image {
  width: 100%;
  aspect-ratio: 16/9;
}

.article-card__content {
  padding: 16px;
}

/* 小容器：堆叠布局 */
@container article (max-width: 399px) {
  .article-card {
    grid-template-columns: 1fr;
  }
}

/* 中容器：图片在左 */
@container article (min-width: 400px) and (max-width: 599px) {
  .article-card {
    grid-template-columns: 120px 1fr;
  }
  
  .article-card__image {
    aspect-ratio: 1;
  }
}

/* 大容器：完整布局 */
@container article (min-width: 600px) {
  .article-card {
    grid-template-columns: 200px 1fr;
  }
  
  .article-card__content {
    padding: 24px;
  }
}

/* ===== 场景2：产品网格 ===== */
.product-grid-wrapper {
  container-type: inline-size;
  container-name: product-grid;
}

.product-grid {
  display: grid;
  gap: 16px;
}

/* 单列 */
@container product-grid (max-width: 299px) {
  .product-grid {
    grid-template-columns: 1fr;
  }
}

/* 两列 */
@container product-grid (min-width: 300px) and (max-width: 499px) {
  .product-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 三列 */
@container product-grid (min-width: 500px) and (max-width: 799px) {
  .product-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

/* 四列 */
@container product-grid (min-width: 800px) {
  .product-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

/* ===== 场景3：侧边栏组件 ===== */
.sidebar-widget-wrapper {
  container-type: inline-size;
  container-name: sidebar-widget;
}

.sidebar-widget {
  padding: 12px;
  background: white;
  border-radius: 8px;
}

/* 窄侧边栏 */
@container sidebar-widget (max-width: 199px) {
  .sidebar-widget {
    padding: 8px;
  }
  
  .sidebar-widget__title {
    font-size: 12px;
    text-align: center;
  }
  
  .sidebar-widget__list {
    display: none;
  }
}

/* 正常侧边栏 */
@container sidebar-widget (min-width: 200px) {
  .sidebar-widget__title {
    font-size: 14px;
  }
  
  .sidebar-widget__list {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
}
```

### Cascade Layers（级联层）

#### 深度原理解析

Cascade Layers 解决了 CSS 中一个长期困扰开发者的问题：**样式优先级冲突**。它允许开发者显式控制不同来源样式的优先级顺序：

**核心概念**：
- **层（Layer）**：样式的逻辑分组
- **声明顺序**：决定层的优先级
- **层内优先级**：仍然遵循普通 CSS 优先级规则

```css
/* 定义层顺序（从低到高）*/
@layer reset, base, components, patterns, utilities, overrides;

/* 或者先声明后定义 */
@layer utilities;
@layer components;
@layer base;
@layer reset;
```

#### 层的使用场景

```css
/* ===== 场景1：第三方库覆盖 ===== */
/* 问题：Bootstrap 的 specificity 很高，难以覆盖 */
.bootstrap-btn {
  background: blue;
}

.my-btn {
  background: red; /* 可能无效！ */
}

/* 解决方案：使用层 */
@layer vendors, overrides {
  @layer vendors {
    .btn {
      display: inline-block;
      padding: 8px 16px;
      border-radius: 4px;
      background: blue;
    }
  }
  
  @layer overrides {
    /* 在更高层中，无视 specificity！ */
    .btn {
      background: red !important;
    }
    
    /* 即使是低 specificity 选择器也能覆盖 */
    button {
      background: green !important;
    }
  }
}

/* ===== 场景2：样式分块管理 ===== */
@layer reset, base, layout, components, utilities;

/* reset.css - 浏览器默认样式重置 */
@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
  }
  
  * {
    margin: 0;
    padding: 0;
  }
  
  html {
    -webkit-text-size-adjust: 100%;
    tab-size: 4;
  }
  
  body {
    min-height: 100vh;
    line-height: 1.5;
  }
  
  img, picture, video, canvas, svg {
    display: block;
    max-width: 100%;
  }
  
  input, button, textarea, select {
    font: inherit;
  }
  
  p, h1, h2, h3, h4, h5, h6 {
    overflow-wrap: break-word;
  }
}

/* base.css - 基础样式 */
@layer base {
  body {
    font-family: system-ui, -apple-system, sans-serif;
    font-size: 16px;
    line-height: 1.5;
    background-color: white;
    color: black;
  }
  
  h1 {
    font-size: 2em;
    font-weight: 700;
    line-height: 1.2;
  }
  
  h2 {
    font-size: 1.5em;
    font-weight: 600;
    line-height: 1.3;
  }
  
  a {
    color: inherit;
    text-decoration: none;
  }
}

/* layout.css - 布局样式 */
@layer layout {
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 16px;
  }
  
  .grid {
    display: grid;
    gap: 24px;
  }
  
  .flex {
    display: flex;
  }
}

/* components.css - 组件样式 */
@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 8px 16px;
    border: none;
    border-radius: 6px;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  
  .button-primary {
    background-color: #3b82f6;
    color: white;
  }
  
  .button-primary:hover {
    background-color: #2563eb;
  }
  
  .card {
    background: white;
    border-radius: 8px;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
}

/* utilities.css - 工具类 */
@layer utilities {
  .hidden {
    display: none !important;
  }
  
  .sr-only {
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
  
  .text-center {
    text-align: center;
  }
  
  .mt-4 {
    margin-top: 16px;
  }
  
  .p-4 {
    padding: 16px;
  }
}
```

#### 条件性层

```css
/* 仅在特定条件下加载层 */
@layer utilities when media(screen and (width >= 768px)) {
  .hidden-mobile {
    display: none;
  }
}

/* 嵌套层结构 */
@layer framework {
  @layer reset {
    /* Framework reset styles */
    * {
      box-sizing: border-box;
    }
  }
  
  @layer base {
    /* Framework base styles */
  }
  
  @layer components {
    /* Framework component styles */
  }
  
  @layer utilities {
    /* Framework utilities */
  }
}

/* 引用嵌套层 */
@layer framework.reset;
@layer framework.base;
```

#### 层与 !important 的关系

```css
@layer utilities {
  .btn {
    display: inline-flex !important; /* 层内的 !important */
  }
}

/* 无层样式可以覆盖有层的 !important（层优先级更高）*/
.btn {
  display: block; /* 覆盖上面的 !important */
}
```

### :has() 选择器

#### 深度原理解析

:has() 是 CSS 选择器的一个革命性补充，它实现了**父选择器**和**条件选择器**的功能：

**核心概念**：
- **父选择器**：选择包含特定子元素的父元素
- **条件选择器**：基于兄弟元素状态选择元素
- **否定逻辑**：结合 :not() 实现复杂条件

```css
/* 基础语法 */
parent:has(child) { }     /* 选择包含 child 的 parent */
parent:has(> child) { }   /* 选择直接包含 child 的 parent */
parent:has(+ sibling) { } /* 选择紧邻 sibling 的 parent */
parent:has(~ sibling) { } /* 选择包含后续 sibling 的 parent */
```

#### 实战场景详解

```css
/* ===== 场景1：表单验证状态 ===== */
/* 显示验证错误的表单 */
form:has(:invalid) {
  border-color: #ef4444;
  background-color: #fef2f2;
}

form:has(:invalid):has(:focus) {
  border-color: #dc2626;
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.2);
}

/* 成功状态的表单 */
form:has(:valid:not(:placeholder-shown)):not(:has(:invalid)) {
  border-color: #22c55e;
  background-color: #f0fdf4;
}

/* ===== 场景2：选中状态指示 ===== */
/* 单选按钮组 */
.radio-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.radio-group:has(input:checked) {
  border: 2px solid #3b82f6;
  border-radius: 8px;
  padding: 16px;
}

.radio-option {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.radio-option:has(input:checked) {
  background-color: #dbeafe;
  font-weight: 500;
}

.radio-option:has(input:checked)::after {
  content: ' ✓';
  color: #3b82f6;
}

/* ===== 场景3：列表状态 ===== */
/* 空状态 */
.list:empty::after {
  content: '暂无数据';
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 48px;
  color: #6b7280;
  font-size: 14px;
}

/* 有内容时隐藏占位符 */
.list:not(:empty) .empty-placeholder {
  display: none;
}

/* 列表项数量提示 */
.grid:has(.card:nth-child(3)) {
  /* 至少3个卡片时触发 */
  position: relative;
}

.grid:has(.card:nth-child(3))::after {
  content: '更多内容...';
  position: absolute;
  bottom: -30px;
  left: 0;
  right: 0;
  text-align: center;
  color: #6b7280;
  font-size: 12px;
}

/* ===== 场景4：图片处理 ===== */
/* 无图片占位 */
figure:not(:has(img)) {
  background: linear-gradient(135deg, #f3f4f6 0%, #e5e7eb 100%);
  min-height: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}

figure:not(:has(img))::before {
  content: '📷';
  font-size: 48px;
  opacity: 0.3;
}

/* 加载中的图片 */
figure:has(img[loading="lazy"]) .image-overlay {
  opacity: 0;
}

figure:not(:has(img[complete])) {
  background: #f3f4f6;
}

/* ===== 场景5：导航状态 ===== */
/* 有下拉菜单的导航项 */
.nav-item:has(.dropdown) > .nav-link::after {
  content: ' ▼';
  font-size: 10px;
  opacity: 0.6;
}

/* 当前激活的菜单分支 */
.nav-branch:has(.nav-item.active) > .nav-branch-header {
  font-weight: 600;
  color: #3b82f6;
}

/* ===== 场景6：卡片变体 ===== */
/* 特色卡片 */
.card:has(.featured-badge) {
  border: 2px solid #f59e0b;
  position: relative;
}

.card:has(.featured-badge)::before {
  content: '特色';
  position: absolute;
  top: -10px;
  right: 16px;
  background: #f59e0b;
  color: white;
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
  font-weight: 600;
}

/* 有徽章的卡片 */
.card:has(.badge) {
  padding-top: 32px;
}

/* 有操作的卡片 */
.card:has(.card-actions) {
  display: flex;
  flex-direction: column;
}

.card:has(.card-actions) .card-content {
  flex: 1;
}

/* ===== 场景7：复杂条件组合 ===== */
/* 同时满足多个条件 */
.article:has(h1):has(p:first-of-type):has(img) {
  grid-column: span 2;
}

/* 否定组合 */
.card:not(:has(.skeleton)):not(:has(.loading)) {
  animation: fadeIn 0.3s ease;
}

/* 范围检测 */
.form:has(:focus):has(:invalid) {
  /* 聚焦且有无效输入 */
}

.form:not(:has(:invalid)):has(:valid) {
  /* 所有字段有效 */
}
```

### 色彩函数

#### color-mix() 函数

```css
:root {
  /* 基础品牌色 */
  --blue-500: #3b82f6;
  
  /* 使用 color-mix 生成色阶 */
  --blue-50: color-mix(in srgb, var(--blue-500) 10%, white);
  --blue-100: color-mix(in srgb, var(--blue-500) 20%, white);
  --blue-200: color-mix(in srgb, var(--blue-500) 40%, white);
  --blue-300: color-mix(in srgb, var(--blue-500) 60%, white);
  --blue-400: color-mix(in srgb, var(--blue-500) 80%, white);
  --blue-500: var(--blue-500);  /* 原色 */
  --blue-600: color-mix(in srgb, var(--blue-500) 90%, black);
  --blue-700: color-mix(in srgb, var(--blue-500) 80%, black);
  --blue-800: color-mix(in srgb, var(--blue-500) 70%, black);
  --blue-900: color-mix(in srgb, var(--blue-500) 50%, black);
  
  /* 色相旋转混合 */
  --purple-mix: color-mix(in srgb, var(--blue-500) 70%, hsl(300 100% 50%));
  --green-mix: color-mix(in srgb, var(--blue-500) 70%, hsl(150 100% 50%));
  --orange-mix: color-mix(in srgb, var(--blue-500) 70%, hsl(30 100% 50%));
  
  /* 不同色彩空间 */
  --blue-srgb: color-mix(in srgb, var(--blue-500) 50%, white);
  --blue-hsl: color-mix(in hsl, var(--blue-500) 50%, white);
  --blue-lab: color-mix(in lab, var(--blue-500) 50%, white);
  --blue-oklch: color-mix(in oklch, var(--blue-500) 50%, white);
}
```

#### OKLCH 色彩空间

OKLCH 是一种感知均匀的色彩空间，特别适合设计系统：

```css
:root {
  /* OKLCH 参数解释 */
  /* L: 亮度 (0-100%) - 感知亮度 */
  /* C: 色度 (0-0.4) - 饱和度/鲜艳度 */
  /* H: 色相 (0-360) - 颜色角度 */
  
  /* 蓝色系列 - 相同色相，不同亮度 */
  --blue-50: oklch(97% 0.02 250);
  --blue-100: oklch(94% 0.04 250);
  --blue-200: oklch(88% 0.08 250);
  --blue-300: oklch(76% 0.14 250);
  --blue-400: oklch(62% 0.20 250);
  --blue-500: oklch(52% 0.22 250);
  --blue-600: oklch(44% 0.20 250);
  --blue-700: oklch(36% 0.18 250);
  --blue-800: oklch(28% 0.14 250);
  --blue-900: oklch(22% 0.10 250);
  
  /* 彩虹色相 - 相同亮度和色度 */
  --red: oklch(52% 0.22 20);
  --orange: oklch(62% 0.22 50);
  --yellow: oklch(82% 0.18 90);
  --green: oklch(62% 0.20 145);
  --cyan: oklch(62% 0.20 200);
  --blue: oklch(52% 0.22 250);
  --purple: oklch(52% 0.22 300);
  --pink: oklch(62% 0.22 340);
  
  /* 主题色 - 高色度值 */
  --brand-primary: oklch(55% 0.25 250);
  --brand-secondary: oklch(65% 0.18 170);
  --brand-accent: oklch(70% 0.22 30);
}

/* relativeColor() - 相对颜色（CSS Color Level 5）*/
.button {
  --base: #3b82f6;
  background: var(--base);
}

.button:hover {
  /* 在原有颜色基础上增加亮度 */
  background: oklch(from var(--base) calc(l + 0.1) c h);
}

.button:active {
  /* 在原有颜色基础上降低亮度 */
  background: oklch(from var(--base) calc(l - 0.1) c h);
}

.button:disabled {
  /* 降低色度使其更灰 */
  background: oklch(from var(--base) l calc(c * 0.3) h);
  opacity: 0.6;
}

/* 动态主题色 */
.theme-primary {
  --primary: oklch(60% 0.24 250);
}

.theme-primary.dark {
  --primary: oklch(65% 0.18 250);
}
```

#### light-dark() 函数

```css
/* 主题配置 */
:root {
  color-scheme: light dark;
}

body {
  /* 自动响应系统主题 */
  background: light-dark(white, #0f172a);
  color: light-dark(#0f172a, white);
}

/* 组件级主题切换 */
.card {
  background: light-dark(
    hsl(0 0% 100%),      /* 浅色模式：白色 */
    hsl(220 20% 10%)     /* 深色模式：深灰蓝 */
  );
  
  color: light-dark(
    hsl(220 10% 10%),    /* 浅色模式：深灰 */
    hsl(0 0% 95%)        /* 深色模式：近白 */
  );
  
  border: 1px solid light-dark(
    hsl(220 10% 90%),    /* 浅色模式：浅灰边框 */
    hsl(220 10% 20%)     /* 深色模式：深灰边框 */
  );
}

/* 复杂场景 */
:root {
  --surface: light-dark(#ffffff, #0f172a);
  --surface-elevated: light-dark(#f8fafc, #1e293b);
  --text-primary: light-dark(#0f172a, #f8fafc);
  --text-secondary: light-dark(#475569, #94a3b8);
  --border: light-dark(#e2e8f0, #334155);
}
```

### CSS 作用域（@scope）

#### 基础用法

```css
/* 定义作用域 */
@scope (.card) {
  .title {
    font-size: 1.5rem;
    font-weight: 600;
  }
  
  .content {
    color: #666;
    line-height: 1.6;
  }
  
  .footer {
    border-top: 1px solid #e5e7eb;
    padding-top: 16px;
    margin-top: 16px;
  }
}
```

#### 作用域限制

```css
/* 作用域限制（to 语法）*/
/* .title 只在 .card 内但不匹配 .footer 时生效 */
@scope (.card) to (.footer) {
  .title {
    font-size: 1.5rem;
    color: #333;
  }
  
  .content {
    color: #666;
  }
  
  /* 这个会匹配 .footer 内的元素，不会被包含 */
  .actions {
    display: flex;
    gap: 8px;
  }
}

/* 排除特定区域 */
@scope (.article) to (.sidebar, .comments) {
  p {
    line-height: 1.8;
  }
  
  h2 {
    margin-top: 2em;
  }
}
```

#### 作用域变体

```css
/* 悬停变体 */
@scope (.card) {
  &:hover {
    /* 悬停时的样式 */
  }
  
  &:hover .title {
    color: var(--primary);
  }
  
  /* 焦点变体 */
  &:focus-within {
    outline: 2px solid var(--primary);
  }
}

/* 响应式作用域 */
@scope (.grid) {
  .item {
    padding: 12px;
  }
  
  @media (min-width: 768px) {
    .item {
      padding: 24px;
    }
  }
}
```

#### 结合层

```css
/* 层内作用域 */
@layer components {
  @scope (.card) {
    .title {
      font-size: 1.25rem;
    }
  }
  
  @scope (.button) {
    /* 按钮相关样式 */
  }
}

/* 作用域内嵌套层 */
@scope (.modal) {
  @layer header {
    .modal-header {
      padding: 16px 24px;
      border-bottom: 1px solid #e5e7eb;
    }
  }
  
  @layer body {
    .modal-body {
      padding: 24px;
    }
  }
  
  @layer footer {
    .modal-footer {
      padding: 16px 24px;
      border-top: 1px solid #e5e7eb;
      display: flex;
      justify-content: flex-end;
      gap: 12px;
    }
  }
}
```

### Subgrid（子网格）

#### 深度原理解析

Subgrid 允许嵌套网格继承父级网格的轨道，实现跨组件对齐：

```css
/* 父容器定义 grid */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

/* 子网格继承父轨道 */
.grid-item {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* 继承父级行轨道 */
}

.grid-item .card-header {
  /* 对齐到第一行 */
}

.grid-item .card-body {
  /* 对齐到第二行 */
}

.grid-item .card-footer {
  /* 对齐到第三行 */
}
```

#### 实战场景

```css
/* ===== 产品卡片网格 ===== */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  grid-template-rows: auto 1fr auto;
  gap: 24px;
}

/* 产品卡片使用子网格 */
.product-card {
  display: grid;
  grid-grid: span 3;
  grid-template-rows: subgrid;
  gap: 16px;
}

.product-card__image {
  aspect-ratio: 4/3;
}

.product-card__title {
  font-size: 1.125rem;
  font-weight: 600;
}

.product-card__description {
  color: #666;
  line-height: 1.6;
}

.product-card__price {
  font-size: 1.25rem;
  font-weight: 700;
  color: var(--color-primary);
}

/* ===== 仪表盘布局 ===== */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr 300px;
  grid-template-rows: auto 1fr auto;
  gap: 24px;
  min-height: 100vh;
}

.dashboard__header {
  grid-column: 1 / -1;
}

.dashboard__sidebar {
  grid-row: 2 / 3;
}

.dashboard__main {
  grid-row: 2 / 3;
}

.dashboard__aside {
  grid-row: 2 / 3;
}

.dashboard__footer {
  grid-column: 1 / -1;
}

/* 子区域使用子网格 */
.widget {
  display: grid;
  grid-row: span 2;
  grid-template-rows: subgrid;
  gap: 16px;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}

.widget__header {
  font-weight: 600;
  padding-bottom: 12px;
  border-bottom: 1px solid #e5e7eb;
}

.widget__content {
  flex: 1;
}

.widget__footer {
  padding-top: 12px;
  border-top: 1px solid #e5e7eb;
}
```

---

## 常用命令与操作

### CSS 变量操作

```bash
# 开发时监控 CSS 文件
npx chokidar "src/**/*.css" -c "npm run build:css"

# 生成 CSS 变量文档
npx css-tokens-generator src/styles/tokens.css --output docs/tokens.md
```

### 构建工具集成

```javascript
// vite.config.js - CSS 相关配置
import { defineConfig } from 'vite';
import autoprefixer from 'autoprefixer';
import cssnano from 'cssnano';

export default defineConfig({
  css: {
    // PostCSS 配置
    postcss: {
      plugins: [
        autoprefixer(),
        cssnano({
          preset: ['default', {
            discardComments: { removeAll: true },
            normalizeWhitespace: true,
            minifySelectors: true,
          }],
        }),
      ],
    },
    
    // CSS Modules
    modules: {
      localsConvention: 'camelCase',
      generateScopedName: '[name]__[local]___[hash:base64:5]',
    },
    
    // Devtools
    devSourcemap: true,
  },
});
```

### CSS 处理命令

```bash
# 运行 PostCSS 处理
npx postcss src/styles.css -o dist/styles.css

# 监听文件变化
npx postcss src/**/*.css --dir dist --watch

# 生成 sourcemap
npx postcss src/styles.css -o dist/styles.css --map
```

---

## 高级配置与技巧

### 性能优化

#### content-visibility

```css
/* 渲染优化 - 跳过离屏内容渲染 */
.article {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px; /* 估计高度 */
}

.off-screen-section {
  content-visibility: hidden; /* 完全跳过渲染 */
}

/* 配合 contain 使用 */
.optimized-container {
  contain: layout style paint;
  content-visibility: auto;
  contain-intrinsic-size: 1000px; /* 滚动条计算 */
}
```

#### will-change 优化

```css
/* 合理使用 will-change */
.animated-element {
  /* 动画开始前设置 */
  will-change: transform, opacity;
}

/* 动画结束后移除 */
.animated-element.end-animation {
  will-change: auto;
}

/* 不要在大量元素上使用 */
.bad-example .item {
  will-change: transform; /* 会创建大量合成层 */
}

/* 正确做法 */
.good-example {
  will-change: transform;
}

.good-example:hover .item {
  transform: translateY(-4px);
}
```

### 复杂布局技巧

#### Subgrid 布局

```css
/* 父容器定义 grid */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

/* 子网格继承父轨道 */
.grid-item {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* 继承父级行轨道 */
}

.grid-item .card-header {
  /* 对齐到第一行 */
}

.grid-item .card-body {
  /* 对齐到第二行 */
}

.grid-item .card-footer {
  /* 对齐到第三行 */
}
```

### 主题系统高级技巧

```css
/* ===== 多主题系统 ===== */
:root {
  /* 默认主题 */
  --color-bg: #ffffff;
  --color-text: #1a1a1a;
  --color-primary: #3b82f6;
}

[data-theme="dark"] {
  --color-bg: #0f172a;
  --color-text: #f8fafc;
  --color-primary: #60a5fa;
}

[data-theme="high-contrast"] {
  --color-bg: #000000;
  --color-text: #ffffff;
  --color-primary: #ffff00;
}

/* ===== 动态品牌色 ===== */
[data-brand="blue"] {
  --brand-hue: 220;
}

[data-brand="green"] {
  --brand-hue: 145;
}

[data-brand="purple"] {
  --brand-hue: 280;
}

:root {
  --color-primary: oklch(60% 0.2 var(--brand-hue, 220));
}

/* ===== 组件级主题覆盖 ===== */
.featured-section {
  --color-primary: #f59e0b; /* 覆盖品牌色 */
  --color-bg: #fef3c7; /* 自定义背景 */
}

/* ===== 渐变主题 ===== */
[data-gradient="true"] {
  --gradient-start: var(--color-primary);
  --gradient-end: oklch(from var(--color-primary) calc(l + 0.1) c h);
}

.gradient-button {
  background: linear-gradient(
    135deg,
    var(--gradient-start),
    var(--gradient-end)
  );
}
```

### CSS 嵌套技巧

```css
/* 现代 CSS 嵌套语法 */
.card {
  padding: 20px;
  
  /* 直接嵌套 */
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
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 4px;
    background: var(--color-primary);
  }
  
  /* 条件嵌套 */
  @media (min-width: 768px) {
    padding: 32px;
  }
  
  /* 复杂选择器 */
  & + & {
    margin-top: 24px;
  }
  
  &--featured {
    border: 2px solid var(--color-primary);
  }
}

/* 媒体查询内嵌套 */
.grid {
  display: grid;
  gap: 16px;
  
  @media (min-width: 768px) {
    grid-template-columns: repeat(2, 1fr);
  }
  
  @media (min-width: 1024px) {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

### contain 属性详解

```css
/* ===== contain 属性 ===== */
/* layout: 元素布局独立 */
.isolated {
  contain: layout;
}

/* style: 防止样式影响外部 */
.scoped {
  contain: style;
}

/* paint: 防止内容溢出可见 */
.clipped {
  contain: paint;
}

/* size: 元素大小不依赖内容 */
.fixed-size {
  contain: size;
  width: 200px;
  height: 200px;
}

/* 组合使用 */
.fully-contained {
  contain: layout style paint;
}

/* 配合 content-visibility 使用 */
.visible-content {
  content-visibility: auto;
  contain: layout style paint;
  contain-intrinsic-size: 0 500px;
}
```

---

## 与同类技术对比

### CSS-in-JS 对比

| 维度 | CSS Custom Properties | CSS-in-JS |
|------|----------------------|-----------|
| **运行时能力** | 受限 | 强大 |
| **性能** | 最优 | 略差 |
| **开发体验** | 好 | 非常好 |
| **包体积** | 无 | 依赖库大小 |
| **适用场景** | 设计系统 | 动态主题 |

### Tailwind CSS 对比

| 维度 | Tailwind | 原生 CSS |
|------|----------|----------|
| **类名数量** | 极多 | 无限制 |
| **定制成本** | 配置复杂 | 即插即用 |
| **产物大小** | 可优化 | 最优 |
| **学习曲线** | 陡峭 | 平缓 |

### CSS 架构方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **BEM** | 简单、无依赖 | 命名繁琐 | 小型项目 |
| **CSS Modules** | 原生支持、局部作用域 | 需要构建工具 | 中型项目 |
| **CSS-in-JS** | 组件化、动态样式 | 包体积、性能开销 | React 项目 |
| **Tailwind CSS** | 快速开发、原子化 |  HTML 混乱 | 快速迭代 |
| **原生 CSS + Layers** | 性能最优、无依赖 | 需要现代浏览器 | 现代项目 |

### 选择策略建议

```markdown
## CSS 架构选择指南

### 小型项目（< 10 个页面）
- 推荐：BEM 或原生 CSS
- 原因：简单直接，无需额外工具

### 中型项目（10-50 个页面）
- 推荐：CSS Modules + CSS Layers
- 原因：组件化、局部作用域、兼容性好

### 大型项目（> 50 个页面）
- 推荐：Tailwind CSS 或 CSS-in-JS
- 原因：开发效率高、样式一致性

### 设计系统
- 推荐：原生 CSS + CSS Variables + Layers
- 原因：性能最优、设计 token 友好
```

---

## 常见问题与解决方案

### 问题1：CSS 变量不生效

**原因**：
1. 变量未定义
2. 选择器优先级问题
3. 作用域问题

**解决**：

```css
/* 确保变量在正确作用域定义 */
:root {
  --color-primary: #3b82f6; /* 全局定义 */
}

.card {
  --color-primary: #10b981; /* 组件覆盖 */
}

/* 使用 calc() 时注意单位 */
:root {
  --spacing: 16; /* 无单位 */
}

.card {
  padding: calc(var(--spacing) * 1px); /* 需要乘以单位 */
}
```

### 问题2：Container Queries 不生效

**原因**：
1. 未定义 container-type
2. 容器有 overflow: hidden 限制
3. 父元素高度问题

**解决**：

```css
/* 正确设置容器 */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

/* 确保父元素不限制尺寸 */
.card-wrapper {
  position: relative;
  width: 100%;
  max-width: none;
}
```

### 问题3：:has() 选择器性能问题

**原因**：
1. 选择器过于复杂
2. 匹配元素过多

**解决**：

```css
/* 优化选择器 */
.card:has(.badge) { } /* 好 */

.card:has(.badge .icon .image) { } /* 差 */

/* 限制范围 */
.list:has(.item) { } /* 好 */

/* 全局优化 */
*:has(.heavy) { } /* 避免 */
```

### 问题4：Cascade Layers 优先级问题

**原因**：
层顺序声明不正确

**解决**：

```css
/* 正确声明层顺序 */
@layer reset, base, components, utilities;

/* 或者在使用前声明 */
@layer utilities; /* 声明但不定义 */
@layer base;
@layer components;

/* 然后定义 */
@layer base {
  /* base 样式 */
}

@layer components {
  /* components 样式 */
}

@layer utilities {
  /* utilities 样式 */
}
```

### 问题5：OKLCH 颜色不一致

**原因**：
不同浏览器对 OKLCH 支持不同

**解决**：

```css
/* 提供回退值 */
.button {
  background: #3b82f6; /* 回退 */
  background: oklch(60% 0.2 250); /* 现代浏览器 */
}

/* 使用 @supports 检测 */
@supports (color: oklch(60% 0.2 250)) {
  .button {
    background: oklch(60% 0.2 250);
  }
}
```

---

## 实战项目示例

### 示例1：响应式设计系统

```css
/* design-system.css - 完整设计系统实现 */

/* ===== 设计 Token ===== */
:root {
  /* 颜色系统 */
  --color-primary-50: oklch(97% 0.02 250);
  --color-primary-100: oklch(94% 0.04 250);
  --color-primary-200: oklch(88% 0.08 250);
  --color-primary-300: oklch(76% 0.14 250);
  --color-primary-400: oklch(62% 0.20 250);
  --color-primary-500: oklch(52% 0.22 250);
  --color-primary-600: oklch(44% 0.20 250);
  --color-primary-700: oklch(36% 0.18 250);
  --color-primary-800: oklch(28% 0.14 250);
  --color-primary-900: oklch(22% 0.10 250);
  
  /* 语义化颜色 */
  --color-bg: var(--color-primary-50);
  --color-text: var(--color-primary-900);
  --color-accent: var(--color-primary-500);
  
  /* 间距 */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  
  /* 圆角 */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;
  --radius-full: 9999px;
  
  /* 阴影 */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
  
  /* 过渡 */
  --transition-fast: 100ms ease;
  --transition-base: 200ms ease;
  --transition-slow: 300ms ease;
}

/* ===== 基础组件 ===== */
@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: var(--space-2);
    padding: var(--space-2) var(--space-4);
    border-radius: var(--radius-md);
    font-weight: 500;
    transition: all var(--transition-base);
    cursor: pointer;
    border: none;
  }
  
  .button-primary {
    background: var(--color-primary-500);
    color: white;
  }
  
  .button-primary:hover {
    background: var(--color-primary-600);
    transform: translateY(-1px);
    box-shadow: var(--shadow-md);
  }
  
  .button-primary:active {
    transform: translateY(0);
    box-shadow: var(--shadow-sm);
  }
  
  .card {
    background: white;
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-md);
    overflow: hidden;
  }
  
  .card-body {
    padding: var(--space-6);
  }
}

/* ===== 响应式组件 ===== */
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
  
  .card-body {
    padding: var(--space-8);
  }
}
```

### 示例2：主题切换系统

```css
/* theme-system.css */

:root {
  /* 浅色主题 */
  --theme-bg: white;
  --theme-surface: #f8fafc;
  --theme-text: #0f172a;
  --theme-text-muted: #64748b;
  --theme-border: #e2e8f0;
  --theme-primary: oklch(55% 0.22 250);
}

[data-theme="dark"] {
  /* 深色主题 */
  --theme-bg: #0f172a;
  --theme-surface: #1e293b;
  --theme-text: #f8fafc;
  --theme-text-muted: #94a3b8;
  --theme-border: #334155;
  --theme-primary: oklch(65% 0.18 250);
}

[data-theme="synthwave"] {
  /* 赛博主题 */
  --theme-bg: #0a0a1a;
  --theme-surface: #1a1a3a;
  --theme-text: #f0f0ff;
  --theme-text-muted: #8888aa;
  --theme-border: #3a3a6a;
  --theme-primary: oklch(70% 0.25 300);
}

/* 应用主题 */
body {
  background: var(--theme-bg);
  color: var(--theme-text);
}

.surface {
  background: var(--theme-surface);
  border: 1px solid var(--theme-border);
}

.text-muted {
  color: var(--theme-text-muted);
}
```

---

## 附录：浏览器兼容速查表

### CSS 新特性兼容性速查（2026年）

| 特性 | Chrome | Firefox | Safari | Edge | iOS | Android |
|------|--------|---------|--------|------|-----|---------|
| **布局相关** |
| CSS Grid | 57+ | 52+ | 10.1+ | 16+ | 10+ | 52+ |
| Flexbox | 21+ | 28+ | 9+ | 11+ | 9+ | 4.4+ |
| Subgrid | 117+ | 71+ | 16+ | 117+ | 16+ | 117+ |
| Container Queries | 105+ | 110+ | 16+ | 105+ | 16+ | 105+ |
| **选择器** |
| :has() | 105+ | 121+ | 15.4+ | 105+ | 15.4+ | 105+ |
| :is() | 88+ | 78+ | 14+ | 88+ | 14+ | 88+ |
| :where() | 88+ | 78+ | 14+ | 88+ | 14+ | 88+ |
| :focus-visible | 119+ | 103+ | 15.4+ | 119+ | 15.4+ | 119+ |
| **样式相关** |
| CSS Variables | 49+ | 31+ | 9.1+ | 15+ | 9.1+ | 49+ |
| Cascade Layers | 99+ | 97+ | 15.4+ | 99+ | 15.4+ | 99+ |
| @scope | 118+ | 老版本 | 17.2+ | 118+ | 17.2+ | 118+ |
| **颜色相关** |
| color-mix() | 111+ | 113+ | 16.2+ | 111+ | 16.2+ | 111+ |
| OKLCH | 119+ | 113+ | 16.4+ | 119+ | 16.4+ | 119+ |
| relativeColor() | 119+ | 老版本 | 17.4+ | 119+ | 17.4+ | 119+ |
| **动画相关** |
| CSS Animations | 43+ | 16+ | 9+ | 12+ | 9+ | 4+ |
| CSS Transitions | 26+ | 16+ | 9+ | 12+ | 9+ | 4+ |
| View Transitions | 111+ | 老版本 | 18+ | 111+ | 18+ | 111+ |
| scroll-timeline | 115+ | 老版本 | 15.4+ | 115+ | 15.4+ | 115+ |
| **渲染优化** |
| content-visibility | 85+ | 老版本 | 17.4+ | 85+ | 17.4+ | 85+ |
| will-change | 36+ | 36+ | 14+ | 79+ | 14+ | 36+ |
| contain | 52+ | 72+ | 15.4+ | 79+ | 15.4+ | 52+ |
| **其他** |
| aspect-ratio | 88+ | 82+ | 15+ | 88+ | 15+ | 88+ |
| object-fit | 32+ | 36+ | 10+ | 79+ | 10+ | 32+ |
| clip-path | 55+ | 54+ | 15.4+ | 79+ | 15.4+ | 55+ |
| backdrop-filter | 76+ | 103+ | 17+ | 76+ | 17+ | 76+ |

### Polyfill 推荐

```bash
# CSS Variables polyfill
npm install -D css-vars-ponyfill

# Container Queries polyfill
npm install -D cqfill

# :has() polyfill
npm install -D has-polyfill
```

### PostCSS 插件推荐

```javascript
// postcss.config.js
export default {
  plugins: {
    // 自动添加前缀
    'autoprefixer': { grid: true },
    
    // CSS Variables 支持
    'postcss-custom-properties': { preserve: true },
    
    // 嵌套语法
    'postcss-nesting': {},
    
    // 颜色函数
    'postcss-color-functional-notation': {},
    
    // :has() 选择器（实验性）
    'postcss-selector-has': {},
  }
};
```

---

## 附录：常见 CSS 框架对比

### Tailwind CSS vs UnoCSS vs Vanilla CSS

| 维度 | Tailwind CSS | UnoCSS | Vanilla CSS |
|------|--------------|--------|-------------|
| **理念** | 实用类优先 | 原子化但可定制 | 原生 CSS |
| **构建时** | 预构建 | 即时原子化 | 无 |
| **包大小** | ~300KB (JIT) | 极小 | 0KB |
| **学习曲线** | 陡峭 | 中等 | 平缓 |
| **灵活性** | 高 | 极高 | 最高 |
| **性能** | 优秀 | 最优 | 最优 |
| **适用场景** | 中大型项目 | 各类项目 | 小型项目 |

---

> [!TIP]
> **现代 CSS 策略**：建立以 Custom Properties 为基础的设计 token 系统，使用 Container Queries 实现真正的组件级响应式，通过 Cascade Layers 管理样式优先级，:has() 简化复杂选择器。这些特性组合起来，可以实现以往需要 JavaScript 才能完成的设计交互。
