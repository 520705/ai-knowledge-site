---
date: 2026-04-24
tags:
  - CSS
  - 现代CSS
  - CSS新特性
---

# 现代 CSS 特性完全指南

> [!NOTE]
> 本文档涵盖 Container Queries、Cascade Layers、:has() 选择器、色彩函数、CSS 作用域等现代 CSS 特性，以及浏览器兼容性策略和深度实战技巧。

---

## CSS 的演进与当代定位

### 从样式表到声明式 UI 引擎

CSS（层叠样式表）自1996年诞生以来，经历了从简单的样式声明到现代复杂样式系统的漫长演进。理解现代 CSS 特性的定位，需要先回顾其发展脉络。

在 CSS 1.0 时代，我们只能定义字体、颜色、文本属性等最基础的样式。那时候的网页设计极其简陋，CSS 主要是作为一种辅助工具，用来微调 HTML 默认的渲染效果。

CSS 2.0 引入了盒模型、定位、浮动等核心概念，奠定了现代布局的基础。这个版本的 CSS 允许开发者控制元素的大小、位置和层叠关系，使得复杂的页面布局成为可能。尽管当时浏览器的实现参差不齐，但 CSS 2.0 确立了许多至今仍在使用的核心概念。

CSS 3 开始的模块化发展彻底改变了 CSS 的面貌。从 2010 年代开始，CSS 被分解为多个独立模块并行发展，引入了选择器、背景边框、文本效果、2D/3D 变换、动画、多列布局、Flexbox、Grid 等众多特性。这种模块化方法允许浏览器厂商选择性地实现不同的特性，加速了新特性的普及。

今天的现代 CSS 已经进入了声明式 UI 的新时代。Container Queries 让组件能够响应自身容器而非视口的尺寸变化。`:has()` 选择器让父元素能够感知子元素的状态。CSS Houdini 提供了访问浏览器渲染引擎的接口。OKLCH 色彩空间让颜色定义更加直观和一致。这些特性让 CSS 不再只是一种样式声明语言，而是具备了构建复杂用户界面的能力。

### 为什么现代 CSS 值得关注

在 vibecoding 时代，CSS 的角色正在发生根本性转变。传统的 CSS 开发模式需要依赖大量的 JavaScript 来实现复杂的交互效果，但现代 CSS 让这种模式发生了革命性的变化。

首先是声明式优先的理念。通过 `:has()` 选择器，我们可以实现过去需要事件监听器才能完成的交互逻辑。例如，当表单包含无效输入时自动显示错误状态，当列表包含特定项目时改变样式，当卡片包含徽章时自动调整布局。这些逻辑过去需要在 JavaScript 中监听 DOM 变化，现在可以直接在 CSS 中声明。

其次是组件自包含的追求。Container Queries 的出现让组件真正成为自包含的单元。在过去，一个响应式卡片组件在侧边栏中可能显示为紧凑布局，在主内容区显示为宽版布局，但这种自适应能力需要依赖外部容器传递信息。Container Queries 让组件可以直接检测自身容器的尺寸，自主决定渲染方式，而不需要任何 JavaScript 逻辑或外部 CSS 类名。

第三是性能原生支持。现代 CSS 内置了大量性能优化特性。`content-visibility` 可以自动跳过离屏内容的渲染，显著提升长页面的滚动性能。`will-change` 可以提示浏览器提前优化特定元素。CSS 动画使用 GPU 加速，确保流畅的视觉效果。

第四是设计系统集成。CSS 变量与设计 token 系统的结合让主题切换、组件变体、品牌定制等需求变得前所未有的简单。你不需要重新编译 CSS，只需要修改变量值，整个界面的主题就会自动更新。

### 浏览器支持现状与选型策略

截至 2026 年，现代 CSS 特性的浏览器支持已达到生产就绪阶段。对于新项目，可以直接采用所有现代 CSS 特性，因为目标用户的浏览器通常会自动更新。对于企业级项目，建议采用渐进增强策略，核心样式使用广泛支持的基础特性，高级特性使用 `@supports` 条件加载。对于需要兼容旧浏览器的项目，使用 PostCSS 的 autoprefixer、CSS Modules、或 CSS-in-JS 方案来处理兼容性问题。

---

## Custom Properties 深度解析

### CSS 变量的核心机制

CSS Custom Properties（CSS 变量）是现代 CSS 最基础的特性之一，它不仅是简单的变量替换，更是 CSS 架构的革命性变革。

CSS 变量与其他 CSS 值的关键区别在于：CSS 变量的值可以由 JavaScript 动态修改，而静态 CSS 值一旦编译就固定了。这种动态性为样式系统带来了无限可能。

CSS 变量的继承规则是其核心机制之一。CSS 变量遵循正常的继承规则，可以在任何选择器中定义，子元素自动继承父元素的值。这种机制允许我们为不同的上下文设置不同的变量值：

```css
/* 全局变量 - 定义在 :root 伪类上，所有元素都可以继承 */
:root {
  --primary-color: #3b82f6;
  --spacing-unit: 8px;
  --border-radius: 4px;
  --font-family-base: 'Inter', system-ui, sans-serif;
}

/* 组件级变量 - 只有 .card 及其子元素可以使用 */
.card {
  --card-padding: 16px;
  --card-bg: white;
  --card-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 主题变体 - 覆盖组件变量 */
.card.dark {
  --card-bg: #1e293b;
  --card-text: #f1f5f9;
  --card-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}

/* 响应式变量 - 根据屏幕尺寸调整 */
@media (min-width: 768px) {
  :root {
    --spacing-unit: 10px;
    --font-size-base: 17px;
  }
}
```

这个继承机制的强大之处在于，我们可以为不同的 DOM 层级设置不同作用域的变量，实现真正的设计系统层级结构。设计系统的全局变量定义在根级别，组件变量定义在组件容器上，主题覆盖定义在特定类名上。

### 计算与组合的高级用法

CSS 变量可以在 `calc()` 函数中使用，实现动态计算。这种能力让 CSS 变量不仅仅是静态值的别名，而是真正的可计算表达式：

```css
:root {
  /* 基础间距值 */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;

  /* 组合计算 */
  --card-padding: calc(var(--spacing-md) + var(--spacing-sm));
  --card-gap: calc(var(--spacing-md) * 2);
  --modal-padding: calc(var(--spacing-lg) + var(--spacing-md));

  /* 复杂计算 */
  --container-width: calc(100% - var(--spacing-xl) * 2);
  --grid-columns: 12;
  --grid-gap: 24px;
  --column-width: calc(
    (var(--container-width) - (var(--grid-gap) * (var(--grid-columns) - 1))) / var(--grid-columns)
  );

  /* 响应式计算 */
  --fluid-font-size: clamp(1rem, 2vw, 1.5rem);
  --fluid-spacing: clamp(8px, 2vw, 24px);
}

/* 使用计算值 */
.grid {
  display: grid;
  grid-template-columns: repeat(var(--grid-columns), 1fr);
  gap: var(--grid-gap);
}

.card {
  padding: var(--card-padding);
  gap: var(--card-gap);
}

h1 {
  font-size: var(--fluid-font-size);
}
```

备用值机制是另一个重要特性。当变量未定义时，备用值会生效：

```css
.element {
  /* 无备用值 - 如果变量未定义则使用默认值（通常是 initial）*/
  color: var(--theme-color);

  /* 带备用值 */
  color: var(--theme-color, #3b82f6);

  /* 备用值也可以是变量 */
  color: var(--theme-color, var(--fallback-color));

  /* 多个备用值 */
  color: var(--missing, red, blue);

  /* 复杂备用值 */
  background: var(--card-gradient, linear-gradient(to right, #667eea, #764ba2));
  font-size: var(--card-font-size, clamp(14px, 2vw, 18px));
}
```

### JavaScript 操作 CSS 变量

CSS 变量可以通过 JavaScript 动态读取和修改，这是实现运行时主题切换的基础：

```javascript
// 获取元素上设置的变量值
const element = document.querySelector('.card');
const computedStyle = getComputedStyle(element);

// 获取具体变量值
const primaryColor = computedStyle.getPropertyValue('--primary-color').trim();
console.log('Primary color:', primaryColor);

// 获取带单位的值
const spacing = computedStyle.getPropertyValue('--spacing').trim();

// 动态设置变量值
element.style.setProperty('--primary-color', '#10b981');

// 批量更新多个变量
element.style.cssText = `
  --primary-color: #10b981;
  --secondary-color: #6366f1;
  --spacing: 24px;
`;

// 删除变量
element.style.removeProperty('--temporary-variable');

// 使用 CSSUnitValue API（现代浏览器）
const value = computedStyle.getPropertyValue('--spacing');
const parsed = CSS.parseComponentValue?.(value) || value;
if (typeof parsed === 'object' && parsed.value !== undefined) {
  console.log('Value:', parsed.value, 'Unit:', parsed.unit);
}
```

### 设计 Token 系统的完整实现

CSS 变量是构建设计 Token 系统的理想选择。一个完整的设计系统应该包含多个层级的变量：

```css
:root {
  /* ===== 色彩系统 ===== */

  /* 品牌色 - 使用 OKLCH 以获得更好的感知均匀性 */
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

  /* 功能色 */
  --color-success: oklch(62% 0.20 145);
  --color-warning: oklch(70% 0.18 80);
  --color-error: oklch(55% 0.22 25);
  --color-info: oklch(60% 0.20 220);

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
  --font-size-5xl: 3rem;     /* 48px */

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
  --spacing-3: 0.75rem;    /* 12px */
  --spacing-4: 1rem;       /* 16px */
  --spacing-5: 1.25rem;   /* 20px */
  --spacing-6: 1.5rem;    /* 24px */
  --spacing-8: 2rem;       /* 32px */
  --spacing-10: 2.5rem;    /* 40px */
  --spacing-12: 3rem;      /* 48px */
  --spacing-16: 4rem;      /* 64px */
  --spacing-20: 5rem;      /* 80px */
  --spacing-24: 6rem;     /* 96px */

  /* ===== 圆角系统 ===== */
  --radius-none: 0;
  --radius-sm: 0.25rem;   /* 4px */
  --radius-md: 0.5rem;    /* 8px */
  --radius-lg: 0.75rem;   /* 12px */
  --radius-xl: 1rem;      /* 16px */
  --radius-2xl: 1.5rem;  /* 24px */
  --radius-full: 9999px;

  /* ===== 阴影系统 ===== */
  --shadow-xs: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-sm: 0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1);
  --shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1);
  --shadow-2xl: 0 25px 50px -12px rgb(0 0 0 / 0.25);

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

---

## Container Queries 容器查询

### 核心概念与原理

Container Queries 是 CSS 响应式设计的重大突破，它允许组件根据自身容器的尺寸来调整样式，而非仅仅依赖视口尺寸。这个特性的出现，解决了响应式设计中长期存在的一个痛点：组件无法独立于页面布局进行响应式调整。

在 Container Queries 出现之前，如果你想让一个卡片组件在侧边栏中显示紧凑布局，在主内容区显示宽版布局，你需要知道这个卡片将被放在什么容器中，然后在媒体查询中针对不同的容器尺寸编写样式。这种方式的问题是：组件与页面布局强耦合，无法复用。

Container Queries 彻底解决了这个问题。你只需要在组件的容器上定义一个查询上下文，然后组件就可以直接响应这个容器的尺寸变化，而不需要知道这个容器在页面的什么位置。

```css
/* 定义容器 - 简写形式 */
.card-container {
  container: card / inline-size;
  /* 等价于:
     container-name: card;
     container-type: inline-size;
  */
}

/* 定义容器 - 详细形式 */
.card-wrapper {
  container-type: inline-size;   /* 监听内联方向尺寸（宽度）*/
  container-name: card;            /* 命名容器 */
}

/* 块方向容器 */
.sidebar {
  container-type: block-size;     /* 监听块方向尺寸（高度）*/
  container-name: sidebar;
}

/* 两个方向都监听 */
.panel {
  container-type: size;           /* 监听宽高 */
  container-name: panel;
}

/* 无尺寸约束的容器（仅命名空间）*/
.container {
  container-type: normal;         /* 不创建包含上下文 */
  container-name: my-container;
}
```

### 容器查询语法详解

```css
/* 默认样式 - 无查询条件时 */
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

/* 命名容器查询 - 只响应特定容器的变化 */
@container card (min-width: 400px) {
  .product-card {
    flex-direction: row;
  }
}

/* 范围查询 */
@container (300px <= width <= 600px) {
  .product-card {
    /* 宽度在 300-600px 之间时触发 */
    padding: 16px;
  }
}

/* 组合查询 */
@container (min-width: 300px) and (max-width: 600px) {
  .product-card {
    /* 同时满足两个条件 */
    font-size: 0.875rem;
  }
}

/* 否定查询 */
@container not (min-width: 300px) {
  .product-card {
    /* 宽度 < 300px */
    flex-direction: column;
  }
}

/* 纵横比查询 */
@container (aspect-ratio > 1) {
  .product-card {
    /* 横向布局 */
    flex-direction: row;
  }
}
```

### 实战：响应式卡片组件

这是一个完整的响应式卡片组件示例，展示了如何利用 Container Queries 实现真正的组件级响应式：

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
  color: var(--color-foreground);
}

.card-description {
  font-size: var(--font-size-sm);
  color: var(--color-text-secondary);
  line-height: var(--line-height-normal);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
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

  .card-description {
    -webkit-line-clamp: 3;
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
    -webkit-line-clamp: 4;
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

### Container Queries 与媒体查询的对比

理解这两种查询方式的区别和使用场景非常重要：

| 维度 | 媒体查询 | 容器查询 |
|------|----------|----------|
| 查询基准 | 视口（viewport） | 父容器（container） |
| 适用场景 | 页面整体布局 | 可复用组件 |
| 组件独立性 | 依赖页面上下文 | 完全独立 |
| 响应式粒度 | 页面级 | 组件级 |

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

---

## Cascade Layers 级联层

### 核心概念与使用场景

Cascade Layers 解决了 CSS 中一个长期困扰开发者的问题：样式优先级冲突。当我们使用第三方库或框架时，经常会遇到难以覆盖的样式。传统的解决方案是增加选择器的特异性或使用 !important，但这会导致代码混乱和难以维护。

Cascade Layers 允许开发者显式控制不同来源样式的优先级顺序。通过将样式分配到不同的层中，后声明的层自动获得更高的优先级，无论其选择器的特异性如何。

```css
/* 定义层顺序（从低到高）- 这个声明应该在所有 CSS 之前 */
@layer reset, base, components, patterns, utilities, overrides;

/* 或者先声明后定义 */
@layer utilities;
@layer components;
@layer base;
@layer reset;
```

### 实战：样式分块管理

```css
/* 定义层顺序 */
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
    font-family: var(--font-family-sans);
    font-size: var(--font-size-base);
    line-height: var(--line-height-normal);
    background-color: var(--color-background);
    color: var(--color-foreground);
  }

  h1 {
    font-size: var(--font-size-4xl);
    font-weight: var(--font-weight-bold);
    line-height: var(--line-height-tight);
  }

  h2 {
    font-size: var(--font-size-3xl);
    font-weight: var(--font-weight-semibold);
    line-height: var(--line-height-tight);
  }

  a {
    color: var(--color-primary);
    text-decoration: none;
    transition: color var(--duration-fast) var(--ease-out);
  }

  a:hover {
    color: var(--color-primary-hover);
  }
}

/* layout.css - 布局样式 */
@layer layout {
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 var(--spacing-4);
  }

  @media (min-width: 768px) {
    .container {
      padding: 0 var(--spacing-6);
    }
  }

  @media (min-width: 1024px) {
    .container {
      padding: 0 var(--spacing-8);
    }
  }

  .grid {
    display: grid;
    gap: var(--spacing-6);
  }

  .flex {
    display: flex;
  }

  .flex-center {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

/* components.css - 组件样式 */
@layer components {
  .button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: var(--spacing-2) var(--spacing-4);
    border: none;
    border-radius: var(--radius-md);
    font-weight: var(--font-weight-medium);
    cursor: pointer;
    transition: all var(--duration-normal) var(--ease-out);
  }

  .button-primary {
    background-color: var(--color-primary);
    color: white;
  }

  .button-primary:hover {
    background-color: var(--color-primary-hover);
    transform: translateY(-1px);
    box-shadow: var(--shadow-md);
  }

  .card {
    background: var(--color-surface);
    border-radius: var(--radius-lg);
    box-shadow: var(--shadow-sm);
    overflow: hidden;
    transition: all var(--duration-normal) var(--ease-out);
  }

  .card:hover {
    box-shadow: var(--shadow-md);
    transform: translateY(-2px);
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

  .text-balance {
    text-wrap: balance;
  }

  .mt-4 {
    margin-top: var(--spacing-4);
  }

  .p-4 {
    padding: var(--spacing-4);
  }

  .gap-4 {
    gap: var(--spacing-4);
  }
}
```

### 覆盖第三方库样式

这是 Cascade Layers 最实用的场景之一：

```css
/* 问题：Bootstrap 的 specificity 很高，难以覆盖 */
.bootstrap-btn {
  background: blue !important;
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
```

---

## :has() 选择器

### 革命性的父选择器

`:has()` 是 CSS 选择器的一个革命性补充，它实现了父选择器和条件选择器的能力。在过去，CSS 只能从父元素选择子元素或从前面选择器选择后面的兄弟元素，但无法根据子元素的状态来选择父元素。

```css
/* 基础语法 */
parent:has(child) { }     /* 选择包含 child 的 parent */
parent:has(> child) { }   /* 选择直接包含 child 的 parent */
parent:has(+ sibling) { } /* 选择紧邻 sibling 的 parent */
parent:has(~ sibling) { } /* 选择包含后续 sibling 的 parent */
```

### 实战场景详解

```css
/* ===== 场景1：表单验证状态 ===== */
/* 显示验证错误的表单 */
form:has(:invalid) {
  border-color: var(--color-error);
  background-color: var(--color-error-bg);
}

form:has(:invalid):has(:focus) {
  border-color: var(--color-error);
  box-shadow: 0 0 0 3px rgba(239, 68, 68, 0.2);
}

/* 成功状态的表单 */
form:has(:valid:not(:placeholder-shown)):not(:has(:invalid)) {
  border-color: var(--color-success);
  background-color: var(--color-success-bg);
}

/* ===== 场景2：选中状态指示 ===== */
.radio-group {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-2);
}

.radio-group:has(input:checked) {
  border: 2px solid var(--color-primary);
  border-radius: var(--radius-md);
  padding: var(--spacing-4);
}

.radio-option {
  display: flex;
  align-items: center;
  gap: var(--spacing-2);
  padding: var(--spacing-2) var(--spacing-3);
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: background-color var(--duration-fast) var(--ease-out);
}

.radio-option:has(input:checked) {
  background-color: var(--color-primary-bg);
  font-weight: var(--font-weight-medium);
}

.radio-option:has(input:checked)::after {
  content: ' ✓';
  color: var(--color-primary);
}

/* ===== 场景3：空状态处理 ===== */
/* 空列表显示占位符 */
.list:empty::after {
  content: '暂无数据';
  display: flex;
  align-items: center;
  justify-content: center;
  padding: var(--spacing-12);
  color: var(--color-text-tertiary);
  font-size: var(--font-size-sm);
}

/* ===== 场景4：图片处理 ===== */
/* 无图片占位 */
figure:not(:has(img)) {
  background: linear-gradient(135deg, var(--color-gray-100) 0%, var(--color-gray-200) 100%);
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

/* ===== 场景5：导航状态 ===== */
/* 有下拉菜单的导航项 */
.nav-item:has(.dropdown) > .nav-link::after {
  content: ' ▼';
  font-size: 10px;
  opacity: 0.6;
}

/* 当前激活的菜单分支 */
.nav-branch:has(.nav-item.active) > .nav-branch-header {
  font-weight: var(--font-weight-semibold);
  color: var(--color-primary);
}

/* ===== 场景6：卡片变体 ===== */
/* 特色卡片 */
.card:has(.featured-badge) {
  border: 2px solid var(--color-warning);
  position: relative;
}

.card:has(.featured-badge)::before {
  content: '特色';
  position: absolute;
  top: -10px;
  right: var(--spacing-4);
  background: var(--color-warning);
  color: white;
  padding: 2px var(--spacing-2);
  border-radius: var(--radius-sm);
  font-size: var(--font-size-xs);
  font-weight: var(--font-weight-semibold);
}

/* 有徽章的卡片 */
.card:has(.badge) {
  padding-top: var(--spacing-8);
}

/* ===== 场景7：复杂条件组合 ===== */
/* 同时满足多个条件 */
.article:has(h1):has(p:first-of-type):has(img) {
  grid-column: span 2;
}

/* 否定组合 */
.card:not(:has(.skeleton)):not(:has(.loading)) {
  animation: fadeIn var(--duration-normal) var(--ease-out);
}
```

---

## 色彩函数详解

### color-mix() 函数

`color-mix()` 函数允许你在 CSS 中混合两种颜色，这是创建色阶和变体的强大工具：

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
  /* 原色 */
  --blue-500: var(--blue-500);
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

### OKLCH 色彩空间

OKLCH 是一种感知均匀的色彩空间，特别适合设计系统。OKLCH 代表 Lightness（亮度）、Chroma（色度）、Hue（色相），这三个参数更符合人类对颜色的感知方式：

```css
:root {
  /* OKLCH 参数解释 */
  /* L: 亮度 (0-100%) - 感知亮度，越大越亮 */
  /* C: 色度 (0-0.4) - 饱和度/鲜艳度，越大越鲜艳 */
  /* H: 色相 (0-360) - 颜色角度，0/360 是红色，120 是绿色，240 是蓝色 */

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

/* relativeColor() - 相对颜色语法 */
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
```

### light-dark() 函数

`light-dark()` 函数提供了简洁的明暗主题颜色切换方式：

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
```

---

## CSS 作用域与嵌套

### @scope 作用域规则

CSS 作用域允许你限制选择器的应用范围，类似于 CSS Modules 的功能但不需要构建工具：

```css
/* 定义作用域 */
@scope (.card) {
  .title {
    font-size: 1.5rem;
    font-weight: 600;
  }

  .content {
    color: var(--color-text-secondary);
    line-height: 1.6;
  }

  .footer {
    border-top: 1px solid var(--color-border);
    padding-top: var(--spacing-4);
    margin-top: var(--spacing-4);
  }
}

/* 作用域限制（to 语法）*/
/* .title 只在 .card 内但不匹配 .footer 时生效 */
@scope (.card) to (.footer) {
  .title {
    font-size: 1.5rem;
    color: var(--color-foreground);
  }

  /* 这个不会匹配 .footer 内的元素 */
  .actions {
    display: flex;
    gap: var(--spacing-2);
  }
}
```

### CSS 嵌套语法

现代 CSS 支持原生的嵌套语法，不再需要 SCSS 或 PostCSS 插件：

```css
/* 现代 CSS 嵌套语法 */
.card {
  padding: var(--spacing-6);

  /* 直接嵌套 */
  & .card-header {
    font-size: var(--font-size-lg);
    font-weight: var(--font-weight-semibold);
  }

  /* 伪类嵌套 */
  &:hover {
    box-shadow: var(--shadow-lg);
    transform: translateY(-2px);
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

  /* 媒体查询嵌套 */
  @media (min-width: 768px) {
    padding: var(--spacing-8);
  }

  /* 复杂选择器 */
  & + & {
    margin-top: var(--spacing-6);
  }

  &--featured {
    border: 2px solid var(--color-primary);
  }
}
```

---

## Subgrid 子网格

### 核心概念

Subgrid 允许嵌套网格继承父级网格的轨道，实现跨组件对齐。这是 Grid 布局的终极能力：

```css
/* 父容器定义 grid */
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: var(--spacing-6);
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

### 实战：产品卡片网格

```css
/* ===== 产品卡片网格 ===== */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  grid-template-rows: auto 1fr auto;
  gap: var(--spacing-6);
}

/* 产品卡片使用子网格 */
.product-card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid;
  gap: var(--spacing-4);
  padding: var(--spacing-4);
  background: var(--color-surface);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-sm);
}

.product-card__image {
  aspect-ratio: 4/3;
  border-radius: var(--radius-md);
  overflow: hidden;
}

.product-card__image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.product-card__title {
  font-size: var(--font-size-lg);
  font-weight: var(--font-weight-semibold);
  color: var(--color-foreground);
}

.product-card__description {
  color: var(--color-text-secondary);
  line-height: var(--line-height-relaxed);
}

.product-card__price {
  font-size: var(--font-size-xl);
  font-weight: var(--font-weight-bold);
  color: var(--color-primary);
}
```

---

## 性能优化与最佳实践

### content-visibility 渲染优化

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

### will-change 优化

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

---

## 常见问题与解决方案

### CSS 变量不生效

**原因**：变量未定义、选择器优先级问题或作用域问题。

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

### Container Queries 不生效

**原因**：未定义 container-type、容器有 overflow 限制或父元素高度问题。

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

---

> [!TIP]
> **现代 CSS 策略**：建立以 Custom Properties 为基础的设计 token 系统，使用 Container Queries 实现真正的组件级响应式，通过 Cascade Layers 管理样式优先级，:has() 简化复杂选择器。这些特性组合起来，可以实现以往需要 JavaScript 才能完成的设计交互。

> [!RELATED]
> - [[Tailwind-CSS深度指南]] - Tailwind CSS 完整指南
> - [[设计系统与组件化]] - 组件化开发的完整指南
> - [[CSS动画与运动设计]] - 动画与交互设计
