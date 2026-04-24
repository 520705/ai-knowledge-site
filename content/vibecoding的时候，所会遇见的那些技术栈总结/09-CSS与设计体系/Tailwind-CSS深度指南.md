# Tailwind CSS 完全指南

> [!NOTE]
> 本文档深度解析 Tailwind CSS 3.x/4.x 版本的完整能力，涵盖配置体系、响应式设计、暗色模式、性能优化及与 shadcn/ui 的协同使用，是 AI 辅助编程的首选样式工具。

---

## Tailwind CSS 核心哲学

Tailwind CSS 遵循「实用优先」（Utility-First）原则，通过组合原子化类名构建界面，而非编写传统 CSS。选择它的核心理由：

1. **AI 原生友好**：AI 生成的类名组合更自然，减少 AI 理解上下文成本
2. **零 CSS 文件**：无需维护独立的 `.css` 文件
3. **设计一致性**：通过配置文件强制统一的设计语言
4. **极致性能**：生产构建自动清除未使用样式（Purging）

---

## 基础语法速查

### 空间（Spacing）

```html
<!-- 外边距 -->
<div class="m-4">        <!-- margin: 16px -->
<div class="mx-auto">     <!-- margin-left/right: auto，居中 -->
<div class="mt-4">        <!-- margin-top -->
<div class="mb-4">        <!-- margin-bottom -->
<div class="ml-4">        <!-- margin-left -->
<div class="mr-4">        <!-- margin-right -->
<div class="my-4">        <!-- margin-top/bottom -->
<div class="-mt-4">       <!-- 负值 -->

<!-- 内边距 -->
<div class="p-4">        <!-- padding: 16px -->
<div class="px-4">        <!-- padding-left/right -->
<div class="py-4">        <!-- padding-top/bottom -->

<!-- 间距刻度（0.25rem 为单位）-->
<div class="space-y-4">  <!-- 子元素间距（自动计算）-->
<!-- 0=0px, 1=4px, 2=8px, 3=12px, 4=16px, 5=20px, 6=24px...16=64px -->
```

### 颜色（Colors）

```html
<!-- 背景色 -->
<div class="bg-blue-500">        <!-- 蓝色 500 -->
<div class="bg-red-100">          <!-- 浅红 -->
<div class="bg-gray-900">         <!-- 深灰 -->
<div class="bg-transparent">      <!-- 透明 -->
<div class="bg-current">         <!-- 继承当前颜色 -->

<!-- 文字颜色 -->
<p class="text-gray-600">        <!-- 灰色文字 -->
<p class="text-white">            <!-- 白色 -->
<p class="text-opacity-50">       <!-- 透明度（v3）-->
<p class="text-gray-600/50">      <!-- 透明度简写（v3+）-->

<!-- 边框颜色 -->
<div class="border border-gray-200">
<div class="border-t-2 border-blue-500">

<!-- 渐变背景 -->
<div class="bg-gradient-to-r from-blue-500 to-purple-600">
<div class="bg-gradient-to-br from-green-400 via-blue-500 to-purple-600">
```

### 排版（Typography）

```html
<!-- 字号 -->
<p class="text-xs">      <!-- 12px -->
<p class="text-sm">      <!-- 14px -->
<p class="text-base">    <!-- 16px（默认）-->
<p class="text-lg">      <!-- 18px -->
<p class="text-xl">      <!-- 20px -->
<p class="text-2xl">     <!-- 24px -->
<p class="text-4xl">     <!-- 36px -->

<!-- 字重 -->
<p class="font-light">       <!-- 300 -->
<p class="font-normal">      <!-- 400 -->
<p class="font-medium">      <!-- 500 -->
<p class="font-semibold">    <!-- 600 -->
<p class="font-bold">        <!-- 700 -->

<!-- 行高 -->
<p class="leading-none">     <!-- 1 -->
<p class="leading-tight">    <!-- 1.25 -->
<p class="leading-normal">   <!-- 1.5（默认）-->
<p class="leading-relaxed">  <!-- 1.625 -->
<p class="leading-loose">    <!-- 2 -->

<!-- 字间距 -->
<p class="tracking-tight">   <!-- -0.025em -->
<p class="tracking-wide">     <!-- 0.025em -->

<!-- 对齐 -->
<p class="text-left">       <!-- 左对齐 -->
<p class="text-center">     <!-- 居中 -->
<p class="text-right">       <!-- 右对齐 -->

<!-- 文本装饰 -->
<p class="underline">        <!-- 下划线 -->
<p class="line-through">     <!-- 删除线 -->
<p class="no-underline">     <!-- 无装饰 -->
```

### 边框与圆角

```html
<!-- 边框宽度 -->
<div class="border">              <!-- 1px -->
<div class="border-2">             <!-- 2px -->
<div class="border-4">             <!-- 4px -->
<div class="border-t-2 border-b-0"> <!-- 仅顶部 -->

<!-- 圆角 -->
<div class="rounded">              <!-- 默认圆角（0.25rem）-->
<div class="rounded-sm">           <!-- 0.125rem -->
<div class="rounded-md">           <!-- 0.375rem -->
<div class="rounded-lg">           <!-- 0.5rem -->
<div class="rounded-xl">           <!-- 0.75rem -->
<div class="rounded-2xl">          <!-- 1rem -->
<div class="rounded-full">          <!-- 圆 -->
<div class="rounded-t-lg">           <!-- 仅顶部 -->
<div class="rounded-l-full">         <!-- 仅左侧 -->
```

### 阴影

```html
<div class="shadow-sm">    <!-- 小阴影 -->
<div class="shadow">       <!-- 默认阴影 -->
<div class="shadow-md">    <!-- 中阴影 -->
<div class="shadow-lg">    <!-- 大阴影 -->
<div class="shadow-xl">    <!-- 特大阴影 -->
<div class="shadow-2xl">   <!-- 超大阴影 -->
<div class="shadow-inner">  <!-- 内阴影 -->
<div class="shadow-none">   <!-- 无阴影 -->
```

### 定位

```html
<!-- Display -->
<div class="block">          <!-- 块级 -->
<div class="inline">          <!-- 行内 -->
<div class="inline-block">    <!-- 行内块 -->
<div class="flex">           <!-- Flex 容器 -->
<div class="inline-flex">     <!-- 行内 Flex -->
<div class="grid">            <!-- Grid 容器 -->
<div class="hidden">         <!-- 隐藏 -->
<div class="contents">       <!-- 透明容器 -->

<!-- Position -->
<div class="static">         <!-- 静态（默认）-->
<div class="relative">        <!-- 相对定位 -->
<div class="absolute">        <!-- 绝对定位 -->
<div class="fixed">           <!-- 固定定位 -->
<div class="sticky">          <!-- 粘性定位 -->

<!-- 位置坐标 -->
<div class="top-4">          <!-- top: 16px -->
<div class="right-0">         <!-- right: 0 -->
<div class="bottom-4">        <!-- bottom: 16px -->
<div class="left-1/2">        <!-- left: 50% -->
<div class="-top-4">          <!-- 负值 -->
<div class="inset-0">         <!-- top/right/bottom/left: 0 -->
<div class="inset-x-4">       <!-- left/right -->
<div class="inset-y-0">       <!-- top/bottom -->
```

### Flexbox

```html
<div class="flex">                               <!-- 激活 flex -->
<div class="inline-flex">                          <!-- 行内 flex -->
<div class="flex-row">                             <!-- 默认：水平 -->
<div class="flex-col">                             <!-- 垂直 -->
<div class="flex-wrap">                            <!-- 换行 -->
<div class="flex-nowrap">                          <!-- 不换行 -->

<!-- 主轴对齐 (justify-content) -->
<div class="justify-start">                        <!-- 默认 -->
<div class="justify-center">                      <!-- 居中 -->
<div class="justify-end">                          <!-- 末尾 -->
<div class="justify-between">                      <!-- 两端对齐 -->
<div class="justify-around">                       <!-- 等宽环绕 -->
<div class="justify-evenly">                       <!-- 完全等距 -->

<!-- 交叉轴对齐 (align-items) -->
<div class="items-start">                          <!-- 起点 -->
<div class="items-center">                        <!-- 居中 -->
<div class="items-end">                           <!-- 终点 -->
<div class="items-stretch">                        <!-- 拉伸（默认）-->
<div class="items-baseline">                       <!-- 基线 -->

<!-- 弹性增长 -->
<div class="flex-1">        <!-- flex: 1 1 0% -->
<div class="flex-auto">      <!-- flex: 1 1 auto -->
<div class="flex-none">      <!-- flex: 0 0 auto -->
<div class="grow">           <!-- flex-grow: 1 -->
<div class="shrink">         <!-- flex-shrink: 1 -->
<div class="shrink-0">       <!-- flex-shrink: 0 -->

<!-- 间隙 -->
<div class="gap-4">         <!-- 行列间隙 -->
<div class="gap-x-4 gap-y-2"> <!-- 行列分别设置 -->
<div class="space-x-4">      <!-- 子元素水平间距 -->
<div class="space-y-4">      <!-- 子元素垂直间距 -->
```

---

## 响应式设计

### 断点前缀

```html
<!-- 移动优先：基础样式作用于所有屏幕 -->
<div class="block">

<!-- sm: 640px 及以上 -->
<div class="block sm:flex">

<!-- md: 768px 及以上 -->
<div class="block md:grid md:grid-cols-2">

<!-- lg: 1024px 及以上 -->
<div class="text-sm md:text-base lg:text-lg">

<!-- xl: 1280px 及以上 -->
<div class="w-full xl:w-1/2">

<!-- 2xl: 1536px 及以上 -->
<div class="hidden 2xl:block">
```

### 典型响应式模式

```html
<!-- 响应式卡片网格 -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">

<!-- 响应式导航 -->
<nav class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">

<!-- 响应式排版 -->
<h1 class="text-2xl sm:text-3xl md:text-4xl lg:text-5xl font-bold">
  响应式标题
</h1>

<!-- 响应式间距 -->
<div class="p-4 sm:p-6 md:p-8 lg:p-12">
```

---

## 状态变体

### 交互状态

```html
<!-- 悬停状态 -->
<button class="bg-blue-500 hover:bg-blue-600">
  悬停变深
</button>
<button class="bg-blue-500 hover:scale-105">
  悬停放大
</button>

<!-- 聚焦状态（无障碍）-->
<input class="border-gray-300 focus:border-blue-500 focus:ring-2 focus:ring-blue-200">

<!-- 激活状态 -->
<button class="bg-blue-500 active:bg-blue-700">
  按下变深
</button>

<!-- 禁用状态 -->
<button class="bg-gray-300 cursor-not-allowed opacity-50" disabled>
  禁用按钮
</button>
```

### 伪类变体完整列表

```html
<!-- 所有可用状态前缀 -->
<div class="
  hover:           /* 悬停 */
  focus:           /* 聚焦 */
  focus-within:    /* 子元素聚焦时 */
  focus-visible:   /* 仅键盘聚焦时 */
  active:          /* 按下激活 */
  disabled:        /* 禁用状态 */
  required:        /* 必填状态 */
  invalid:         /* 无效状态 */
  valid:           /* 有效状态 */
  checked:         /* 选中状态 */
  default:         /* 默认状态 */
  placeholder:    /* 占位符 */
  first:           /* 第一个子元素 */
  last:            /* 最后一个子元素 */
  only:            /* 唯一子元素 */
  odd:             /* 奇数子元素 */
  even:           /* 偶数子元素 */
  empty:           /* 空元素 */
  sibling:         /* 后续兄弟元素（需配合）*/
  group-hover:     /* 父元素悬停时 */
  group-focus:     /* 父元素聚焦时 */
  group-active:    /* 父元素激活时 */
">
```

### Group 变体

```html
<div class="group hover:shadow-lg">
  <div class="group-hover:scale-105">
    <!-- 父 div 悬停时，此 div 缩放 -->
  </div>
  <p class="text-gray-500 group-hover:text-blue-500">
    <!-- 父悬停时文字变蓝 -->
  </p>
</div>
```

---

## 暗色模式

### 三种配置模式

```javascript
// tailwind.config.js
module.exports = {
  darkMode: 'class',  // 通过 CSS 类切换（推荐）
  // darkMode: 'media',  // 使用 prefers-color-scheme
  // darkMode: 'selector', // 通过选择器切换（v4）
}
```

### 使用方式

```html
<!-- class 模式 -->
<div class="bg-white dark:bg-gray-900 text-gray-900 dark:text-white">
  内容
</div>

<!-- 切换按钮 -->
<button onclick="document.documentElement.classList.toggle('dark')">
  切换暗色模式
</button>

<!-- 条件变体 -->
<p class="text-gray-500 dark:text-gray-400">
  次要文字
</p>
<button class="bg-blue-600 dark:bg-blue-500">
  主按钮
</button>
```

---

## 动画

### 内置动画

```html
<!-- 旋转 -->
<div class="animate-spin">

<!-- 脉冲 -->
<div class="animate-pulse">

<!-- 摇摆 -->
<div class="animate-ping">

<!-- 弹跳 -->
<div class="animate-bounce">

<!-- 渐入 -->
<div class="animate-fade-in">  <!-- 需自定义 -->
```

### 自定义动画

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      keyframes: {
        'fade-in': {
          '0%': { opacity: '0', transform: 'translateY(10px)' },
          '100%': { opacity: '1', transform: 'translateY(0)' },
        },
        'slide-in': {
          '0%': { transform: 'translateX(-100%)' },
          '100%': { transform: 'translateX(0)' },
        },
        'scale-in': {
          '0%': { opacity: '0', transform: 'scale(0.95)' },
          '100%': { opacity: '1', transform: 'scale(1)' },
        },
      },
      animation: {
        'fade-in': 'fade-in 0.4s ease-out',
        'slide-in': 'slide-in 0.3s ease-out',
        'scale-in': 'scale-in 0.2s ease-out',
      },
    },
  },
}
```

### 悬停动画组合

```html
<button class="
  bg-blue-500 
  hover:bg-blue-600 
  hover:scale-105 
  hover:shadow-lg 
  active:scale-95 
  transition-all 
  duration-200
">
  交互动画按钮
</button>
```

---

## 自定义配置

### 扩展主题

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#eff6ff',
          100: '#dbeafe',
          500: '#3b82f6',  // 主色
          900: '#1e3a8a',
        },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
      spacing: {
        '128': '32rem',
        '144': '36rem',
      },
      borderRadius: {
        '4xl': '2rem',
      },
      boxShadow: {
        'glow': '0 0 20px rgb(59 130 246 / 0.3)',
      },
    },
  },
}
```

### 使用配置的值

```html
<!-- 使用自定义颜色 -->
<div class="bg-brand-500 text-brand-50">

<!-- 使用自定义字体 -->
<p class="font-mono">

<!-- 使用自定义阴影 -->
<div class="shadow-glow">
```

---

## AI 辅助开发最佳实践

### AI 生成规范

```html
<!-- 告诉 AI：
  1. 使用 Tailwind 的原子类
  2. 避免生成 <style> 块
  3. 优先使用内置变体而非自定义
  4. 保持类名可读性
-->

<!-- 好案例：完整描述 + Tailwind 约束 -->
<div class="flex items-center justify-between p-6 bg-white rounded-xl shadow-md hover:shadow-lg transition-shadow duration-300">
  <!-- AI 可直接生成这类代码 -->
</div>

<!-- 不推荐的提示词 -->
<!-- ❌ "给我一个好看的卡片" -->
<!-- ✅ "用 Tailwind 的原子类，创建一个白底圆角卡片，带阴影和悬停动效" -->
```

### 常用模式模板

```html
<!-- 按钮 -->
<button class="
  inline-flex items-center justify-center
  px-4 py-2 
  text-sm font-medium 
  text-white 
  bg-blue-600 rounded-lg
  hover:bg-blue-700
  focus:outline-none focus:ring-2 focus:ring-blue-300
  disabled:opacity-50 disabled:cursor-not-allowed
  transition-colors duration-200
">
  按钮文字
</button>

<!-- 输入框 -->
<input class="
  w-full px-4 py-2 
  text-sm text-gray-900 
  bg-white border border-gray-300 rounded-lg
  focus:outline-none focus:ring-2 focus:ring-blue-200 focus:border-blue-500
  placeholder:text-gray-400
">

<!-- 卡片 -->
<div class="
  p-6 
  bg-white rounded-xl 
  border border-gray-200 
  shadow-sm
  hover:shadow-md hover:border-gray-300
  transition-all duration-200
">
  <h3 class="text-lg font-semibold text-gray-900">标题</h3>
  <p class="mt-2 text-sm text-gray-600">内容</p>
</div>
```

---

> [!TIP]
> **AI 编程核心策略**：始终要求 AI 使用 Tailwind 类名而非生成 CSS 文件。保持类名简洁、可读、可组合。复杂样式封装在 `components/ui/` 中，通过 shadcn/ui 等现成组件减少重复。
