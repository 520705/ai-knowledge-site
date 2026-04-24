---
date: 2026-04-24
tags:
  - CSS
  - 动画
  - Motion Design
  - 交互动画
---

# CSS 动画与运动设计完全指南

> [!NOTE]
> 本文档涵盖 CSS 过渡、CSS 动画、View Transitions API、Scroll-driven Animations、动画性能优化及 Motion 原则，是构建流畅 UI 动效的完整参考。

---

## 动画在现代 UI 设计中的角色

### 运动设计的重要性

在现代 Web 应用中，动画不仅是视觉装饰，更是用户体验的核心组成部分。当我们谈论 UI 设计时，常常会忽略运动这个维度，但实际上，人类视觉系统对运动变化极为敏感，恰当的动画能够显著提升用户体验。

首先，动画提供即时视觉反馈。当用户点击一个按钮、滑动一个控件或提交一个表单时，适当的动画变化能够立即确认操作已被系统接收和响应。这种反馈让用户确信他们的操作正在被处理，而不是输入被忽略或丢失。在没有动画的应用中，用户常常不确定自己的操作是否生效，导致重复点击或焦虑等待。

其次，动画引导用户注意力。在界面上发生变化的元素会自动吸引用户的目光。通过精心设计的动画轨迹，我们可以将用户的注意力引导到重要的信息上，比如新收到的通知、需要确认的操作或需要关注的变化。这种引导比静态的视觉提示更加有效，因为它利用了人类视觉系统的本能反应。

第三，动画建立空间关系。在二维屏幕上展示界面元素的层次和位置关系是一个持续的设计挑战。通过运动设计，我们可以模拟真实世界的物理规律，帮助用户理解界面层次、元素之间的关系和操作结果。比如，当一个新的面板从侧面滑入时，用户会自然地理解这个面板与当前内容的关系。

第四，动画传达品牌个性。精心设计的动画风格可以强化产品的品牌形象。弹性动画给人以活力和现代感，柔和的渐变传达稳定和可信赖，而精心设计的微交互则让产品显得更加精致和用心。

### CSS 动画技术栈概览

CSS 提供了完整的动画技术栈，从简单到复杂包括多个层次。第一层是 Transitions，这是最简单的动画形式，用于在元素状态变化时创建平滑过渡效果。Transitions 非常适合处理 hover、focus、active 等交互状态的变化，只需要声明开始状态和结束状态，浏览器会自动计算中间帧。

第二层是 Animations，当需要更复杂的动画序列时，比如需要多个关键帧、循环播放或精确的时间控制，就需要使用 CSS Animations 和 @keyframes。这个层级的动画可以定义任意的关键帧序列，支持重复播放、方向控制和填充模式。

第三层是 View Transitions API，这是现代 CSS 的革命性特性，允许开发者创建页面级和元素级的平滑过渡效果。View Transitions API 可以让页面切换、模态框显示等场景的过渡动画变得非常简单。

第四层是 Scroll-driven Animations，这是 CSS 的最新特性，允许动画直接与滚动位置关联，无需任何 JavaScript。这个特性让实现滚动触发动画、阅读进度条、视差效果变得前所未有的简单。

### 动画的性能影响

动画对性能的影响是设计时必须考虑的关键因素。理解渲染管线是优化动画性能的基础。

浏览器渲染页面时需要经过几个阶段。首先是 JavaScript 执行，如果动画逻辑在 JavaScript 中，则需要先执行脚本。然后是样式计算（Style），浏览器计算每个元素应用的 CSS 规则。接着是布局（Layout），浏览器计算每个元素的几何属性，如位置和大小。然后是绘制（Paint），浏览器绘制每个元素的像素。最后是合成（Composite），浏览器将各层合成为最终画面显示在屏幕上。

动画触发的阶段越靠后，性能开销越小。最差的动画会触发布局和绘制重新执行，比如动画元素的 width、height、margin、padding、top、left 等属性。较好的动画只触发绘制，比如动画背景色、文字颜色等。最优的动画只触发合成层，比如 transform 和 opacity，因为这两个属性的变化可以在 GPU 中直接处理，不需要 CPU 参与计算。

为了保证流畅的动画体验，目标帧率应该维持在 60fps 左右，这意味着每帧的预算只有约 16.67 毫秒。任何低于这个标准的动画都会被用户感知为卡顿。

---

## CSS Transitions 过渡效果

### 核心属性详解

CSS Transitions 是最简单的动画形式，用于在元素状态变化时创建平滑过渡效果。它的核心是声明从「起点」状态到「终点」状态的变化，浏览器会自动计算中间帧。

transition-property 指定要过渡的 CSS 属性。可以指定单个属性如 `transition-property: background-color`，也可以使用 `all` 来过渡所有可过渡的属性，或者使用 `none` 来禁用过渡。在实际使用中，指定具体属性通常比 `all` 更好，因为可以精确控制哪些属性需要动画。

transition-duration 指定过渡持续时间，值可以是秒（s）或毫秒（ms）。建议使用 100-300ms 的过渡时间：太快会让用户无法感知，太慢会让界面显得迟钝。

transition-timing-function 指定缓动函数，控制动画的速度曲线。缓动函数是动画质量的关键，不同的缓动函数会给人完全不同的感觉。

```css
/* 完整语法 */
.element {
  /* 分开写 */
  transition-property: all;
  transition-duration: 300ms;
  transition-timing-function: ease;
  transition-delay: 0s;

  /* 简写形式 */
  transition: all 300ms ease 0s;

  /* 多属性分别过渡 */
  transition:
    background-color 200ms ease,
    transform 150ms ease-out,
    opacity 200ms ease,
    box-shadow 300ms ease;
}
```

### 缓动函数深度解析

缓动函数决定了动画的速度曲线，是动画质量的关键。理解缓动函数对于创建高质量的动画至关重要。

内置缓动函数提供了几种基本的速度曲线。`ease` 是慢-快-慢的效果，是最自然的曲线。`linear` 是匀速运动，适合需要恒定速度的场景，比如加载动画。`ease-in` 是慢-快，适合元素进入的场景。`ease-out` 是快-慢，适合元素退出的场景。`ease-in-out` 是慢-快-慢，适合元素在界面中移动的场景。

```css
/* 内置缓动函数示例 */
.ease { transition-timing-function: ease; }
.linear { transition-timing-function: linear; }
.ease-in { transition-timing-function: ease-in; }
.ease-out { transition-timing-function: ease-out; }
.ease-in-out { transition-timing-function: ease-in-out; }
```

`cubic-bezier()` 函数允许你自定义缓动曲线，通过指定两个控制点的坐标来定义贝塞尔曲线。这个函数的四个参数分别是 P1x、P1y、P2x、P2y，取值范围通常是 0-1，但 y 轴的值可以超出这个范围来实现弹性效果。

```css
/* 常用 cubic-bezier 曲线 */

/* 标准曲线 - 柔和的减速 */
.ease-standard {
  transition-timing-function: cubic-bezier(0.4, 0.0, 0.2, 1);
}

/* 减速曲线 - 快速进入，缓慢停止 */
.ease-decelerate {
  transition-timing-function: cubic-bezier(0.0, 0.0, 0.2, 1);
}

/* 加速曲线 - 缓慢进入，快速离开 */
.ease-accelerate {
  transition-timing-function: cubic-bezier(0.4, 0.0, 1, 1);
}

/* 锐利曲线 - 快速响应 */
.ease-sharp {
  transition-timing-function: cubic-bezier(0.4, 0.0, 0.6, 1);
}

/* 弹性效果 - 回弹效果 */
.ease-bounce {
  transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1);
}

/* 强弹性效果 */
.ease-elastic {
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}

/* 过度回弹 */
.ease-back {
  transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1.2);
}

/* Material Design 标准曲线 */
.material-standard {
  transition-timing-function: cubic-bezier(0.4, 0.0, 0.2, 1);
}

/* Material Design 减速曲线 */
.material-decelerate {
  transition-timing-function: cubic-bezier(0.0, 0.0, 0.2, 1);
}

/* Material Design 加速曲线 */
.material-accelerate {
  transition-timing-function: cubic-bezier(0.4, 0.0, 1, 1);
}
```

`steps()` 函数创建分步动画，将动画分成相等的步骤。适合创建机械或数字风格的动画效果。

```css
/* 分步函数示例 */
.steps-start {
  /* 4步，起点跳变 - 在每步开始时跳变 */
  transition-timing-function: steps(4, start);
}

.steps-end {
  /* 4步，终点跳变 - 在每步结束时跳变 */
  transition-timing-function: steps(4, end);
}

.steps-both {
  /* 首尾跳变 - 在开始和结束都跳变 */
  transition-timing-function: steps(4, start end);
}
```

### 实战：按钮状态动画

按钮是 UI 中最常见的交互元素，按钮动画的设计直接影响用户的交互体验。一个精心设计的按钮动画应该提供清晰的反馈，同时不会让用户感到等待或迟钝。

```css
/* ===== 按钮基础样式 ===== */
.button {
  /* 基础样式 */
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 12px 24px;
  min-height: 44px; /* 触摸友好 */
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  line-height: 1;
  cursor: pointer;
  user-select: none; /* 防止文字选择干扰 */

  /* 过渡配置 - 核心！ */
  transition:
    background-color 150ms ease,
    color 150ms ease,
    transform 100ms ease,
    box-shadow 150ms ease,
    border-color 150ms ease,
    opacity 150ms ease;
}

/* ===== 主要按钮变体 ===== */
.button-primary {
  background-color: #3b82f6;
  color: white;
}

.button-primary:hover {
  background-color: #2563eb;
  transform: translateY(-1px);
  box-shadow:
    0 4px 12px rgba(59, 130, 246, 0.3),
    0 2px 4px rgba(59, 130, 246, 0.2);
}

.button-primary:active {
  background-color: #1d4ed8;
  transform: translateY(0);
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.2);
}

/* ===== 次要按钮变体 ===== */
.button-secondary {
  background-color: white;
  color: #3b82f6;
  border: 2px solid #3b82f6;
}

.button-secondary:hover {
  background-color: #eff6ff;
  transform: translateY(-1px);
}

.button-secondary:active {
  background-color: #dbeafe;
  transform: translateY(0);
}

/* ===== 幽灵按钮变体 ===== */
.button-ghost {
  background-color: transparent;
  color: #3b82f6;
}

.button-ghost:hover {
  background-color: rgba(59, 130, 246, 0.1);
}

.button-ghost:active {
  background-color: rgba(59, 130, 246, 0.2);
}

/* ===== 危险按钮变体 ===== */
.button-danger {
  background-color: #ef4444;
  color: white;
}

.button-danger:hover {
  background-color: #dc2626;
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(239, 68, 68, 0.3);
}

/* ===== 禁用状态 ===== */
.button:disabled,
.button.disabled {
  opacity: 0.5;
  cursor: not-allowed;
  transform: none !important;
  box-shadow: none !important;
  pointer-events: none;
}

/* ===== 加载状态 ===== */
.button-loading {
  position: relative;
  color: transparent !important;
  pointer-events: none;
}

.button-loading::after {
  content: '';
  position: absolute;
  width: 16px;
  height: 16px;
  border: 2px solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spin 0.6s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

/* ===== 聚焦状态 ===== */
.button:focus-visible {
  outline: 2px solid #60a5fa;
  outline-offset: 2px;
}

/* ===== 图标按钮 ===== */
.button-icon {
  padding: 12px;
  min-width: 44px;
}

.button-icon .icon {
  width: 20px;
  height: 20px;
}
```

### 过渡性能优化原则

动画性能优化的核心原则是只动画 `transform` 和 `opacity` 这两个属性，因为它们的变化可以在 GPU 中直接处理，不会触发布局重排和重绘。

```css
/* ===== 性能友好的过渡属性 ===== */
/* 始终使用 transform 和 opacity */
.good-transition {
  transition:
    transform 200ms ease,
    opacity 200ms ease;
}

/* 避免触发布局重排的属性 */
.bad-transition {
  transition: width 200ms ease;      /* 触发重排 */
  transition: margin 200ms ease;     /* 触发重排 */
  transition: padding 200ms ease;     /* 触发重排 */
  transition: top 200ms ease;        /* 触发重排 */
  transition: left 200ms ease;       /* 触发重排 */
  transition: font-size 200ms ease;   /* 触发重排 */
}

/* ===== 复合变换优化 ===== */
.transform-demo {
  transition: transform 200ms var(--ease-spring);
}

.transform-demo:hover {
  transform:
    translateY(-4px)   /* 上浮 */
    scale(1.02)        /* 轻微放大 */
    rotate(0deg);      /* 可选：轻微旋转 */
}

/* ===== 使用 will-change 提示 ===== */
/* 注意：will-change 应该谨慎使用 */
.will-change-demo {
  will-change: transform, opacity;
  transition: transform 200ms ease, opacity 200ms ease;
}

/* 动画结束后移除 will-change */
.will-change-demo.animated {
  will-change: auto; /* 释放资源 */
}
```

---

## CSS Animations 关键帧动画

### 核心属性详解

CSS Animations 与 Transitions 的核心区别在于：Animations 可以定义多个关键帧，实现更复杂的动画效果，而 Transitions 只能处理两个状态之间的过渡。

animation-name 指定关键帧的名称，可以是任何自定义名称。animation-duration 指定动画持续时间，与 Transition 的 duration 类似。animation-timing-function 指定缓动函数。animation-iteration-count 指定动画播放次数，可以是数字或 `infinite`。animation-direction 指定播放方向：`normal` 正向播放，`reverse` 反向播放，`alternate` 正反交替，`alternate-reverse` 先反后正交替。animation-fill-mode 指定动画播放前后的状态：`none` 保持原样，`forwards` 停在最后一帧，`backwards` 在动画开始前显示第一帧，`both` 结合 forward 和 backward。animation-play-state 控制播放状态：`running` 播放中，`paused` 暂停。

```css
/* 完整语法 */
.animated-element {
  animation-name: fadeInSlideUp;
  animation-duration: 600ms;
  animation-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
  animation-delay: 0ms;
  animation-iteration-count: 1;
  animation-direction: normal;
  animation-fill-mode: both;
  animation-play-state: running;

  /* 简写 */
  animation: fadeInSlideUp 600ms cubic-bezier(0.16, 1, 0.3, 1) 0ms 1 normal both;
}
```

### 关键帧定义详解

`@keyframes` 规则定义动画的关键帧序列。每个关键帧用百分比或 from/to 关键字指定，浏览器会在这些关键帧之间插值生成中间帧。

```css
/* ===== 基础关键帧 ===== */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes fadeOut {
  from { opacity: 1; }
  to { opacity: 0; }
}

@keyframes slideInFromLeft {
  0% {
    transform: translateX(-100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInFromRight {
  0% {
    transform: translateX(100%);
    opacity: 0;
  }
  100% {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes slideInFromTop {
  0% {
    transform: translateY(-100%);
    opacity: 0;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes slideInFromBottom {
  0% {
    transform: translateY(100%);
    opacity: 0;
  }
  100% {
    transform: translateY(0);
    opacity: 1;
  }
}

/* ===== 缩放动画 ===== */
@keyframes scaleIn {
  0% {
    opacity: 0;
    transform: scale(0.8);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes scaleOut {
  0% {
    opacity: 1;
    transform: scale(1);
  }
  100% {
    opacity: 0;
    transform: scale(0.8);
  }
}

/* ===== 旋转动画 ===== */
@keyframes rotate360 {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes rotateFromOrigin {
  0% { transform: rotate(-180deg); }
  100% { transform: rotate(0deg); }
}

/* ===== 弹性动画 ===== */
@keyframes bounce {
  0%, 100% {
    transform: translateY(0);
    animation-timing-function: cubic-bezier(0.8, 0, 1, 1);
  }
  50% {
    transform: translateY(-24px);
    animation-timing-function: cubic-bezier(0, 0, 0.2, 1);
  }
}

/* ===== 脉冲效果 ===== */
@keyframes pulse {
  0%, 100% {
    opacity: 1;
    transform: scale(1);
  }
  50% {
    opacity: 0.7;
    transform: scale(1.05);
  }
}

/* ===== 呼吸效果 ===== */
@keyframes breathe {
  0%, 100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.4);
  }
  50% {
    transform: scale(1.02);
    box-shadow: 0 0 0 20px rgba(59, 130, 246, 0);
  }
}

/* ===== 闪光效果 ===== */
@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

/* ===== 摇摆效果 ===== */
@keyframes wiggle {
  0%, 100% { transform: rotate(-3deg); }
  25% { transform: rotate(3deg); }
  50% { transform: rotate(-3deg); }
  75% { transform: rotate(3deg); }
}

/* ===== 打字机光标 ===== */
@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

/* ===== 渐变背景动画 ===== */
@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}
```

### 实战：页面加载动画

页面加载动画是用户体验的重要组成部分，它可以缓解用户的等待焦虑，同时为应用增添专业感。

```css
/* ===== 骨架屏闪烁 ===== */
.skeleton {
  background: linear-gradient(
    90deg,
    #f0f0f0 25%,
    #e0e0e0 50%,
    #f0f0f0 75%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s ease-in-out infinite;
}

@keyframes shimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

/* 骨架屏内容块 */
.skeleton-text {
  height: 1em;
  border-radius: 4px;
  margin-bottom: 8px;
}

.skeleton-text.short { width: 60%; }
.skeleton-text.medium { width: 80%; }
.skeleton-text.full { width: 100%; }

.skeleton-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
}

.skeleton-image {
  width: 100%;
  aspect-ratio: 16/9;
  border-radius: 8px;
}

/* ===== 列表项依次入场 ===== */
@keyframes itemEnter {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.list-item {
  opacity: 0;
  animation: itemEnter 400ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 100ms; }
.list-item:nth-child(3) { animation-delay: 200ms; }
.list-item:nth-child(4) { animation-delay: 300ms; }
.list-item:nth-child(5) { animation-delay: 400ms; }
.list-item:nth-child(6) { animation-delay: 500ms; }
.list-item:nth-child(7) { animation-delay: 600ms; }
.list-item:nth-child(8) { animation-delay: 700ms; }

/* 批量处理更多项 */
.list-item:nth-child(n+9) { animation-delay: 800ms; }

/* ===== 交错网格动画 ===== */
@keyframes gridItemEnter {
  from {
    opacity: 0;
    transform: scale(0.9) translateY(10px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.grid-item {
  opacity: 0;
  animation: gridItemEnter 500ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

/* 交错延迟计算：delay = (row + col) * 100ms */
.grid-item:nth-child(3n+1) { animation-delay: 0ms; }
.grid-item:nth-child(3n+2) { animation-delay: 100ms; }
.grid-item:nth-child(3n) { animation-delay: 200ms; }

/* ===== 页面加载动画序列 ===== */
.page-loader {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
}

.loader-logo {
  width: 64px;
  height: 64px;
  margin-bottom: 24px;
  animation: pulse 1.5s ease-in-out infinite;
}

.loader-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #e5e7eb;
  border-top-color: #3b82f6;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

.loader-text {
  margin-top: 16px;
  color: #6b7280;
  font-size: 14px;
  animation: fadeIn 300ms ease 200ms forwards;
  opacity: 0;
}

/* ===== 加载完成淡出 ===== */
.loader-hidden {
  animation: fadeOut 300ms ease forwards;
}

@keyframes fadeOut {
  to { opacity: 0; visibility: hidden; }
}
```

### 实战：复杂交互动画

```css
/* ===== 卡片悬停效果 ===== */
.card {
  position: relative;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  transition:
    transform 300ms ease,
    box-shadow 300ms ease;
}

.card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.15);
}

.card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #3b82f6, #8b5cf6);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 300ms ease;
}

.card:hover::before {
  transform: scaleX(1);
}

/* ===== 卡片图片缩放 ===== */
.card-image {
  overflow: hidden;
}

.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 500ms cubic-bezier(0.16, 1, 0.3, 1);
}

.card:hover .card-image img {
  transform: scale(1.1);
}

/* ===== 按钮涟漪效果 ===== */
.ripple-container {
  position: relative;
  overflow: hidden;
}

.ripple {
  position: absolute;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.4);
  transform: scale(0);
  animation: ripple 600ms ease-out forwards;
  pointer-events: none;
}

@keyframes ripple {
  to {
    transform: scale(4);
    opacity: 0;
  }
}

/* ===== 下拉菜单动画 ===== */
.dropdown {
  transform-origin: top center;
  animation: dropdownOpen 200ms ease forwards;
}

@keyframes dropdownOpen {
  from {
    opacity: 0;
    transform: scaleY(0.8) translateY(-10px);
  }
  to {
    opacity: 1;
    transform: scaleY(1) translateY(0);
  }
}

/* ===== 模态框动画 ===== */
.modal-backdrop {
  animation: backdropFadeIn 200ms ease forwards;
}

@keyframes backdropFadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.modal-content {
  animation: modalSlideIn 300ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
}

@keyframes modalSlideIn {
  from {
    opacity: 0;
    transform: scale(0.95) translateY(20px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}

.modal-closing {
  animation: modalSlideOut 200ms ease forwards;
}

@keyframes modalSlideOut {
  to {
    opacity: 0;
    transform: scale(0.95) translateY(20px);
  }
}
```

---

## View Transitions API 视图过渡

### 核心概念与原理

View Transitions API 是现代 CSS 的革命性特性，它允许开发者创建页面级和元素级的平滑过渡效果。这个 API 最初由 Chrome 团队提出并实现，现在已被其他主流浏览器逐步支持。

View Transitions API 的核心思想是：当页面的 DOM 发生变化时，自动捕获新旧状态的「快照」，然后通过 CSS 动画在这两个状态之间平滑过渡。这种方式让页面切换、元素增删等场景的过渡变得极其简单。

```css
/* ===== 启用全局视图过渡 ===== */
@view-transition {
  navigation: auto;
}

/* ===== 自定义过渡效果 ===== */
::view-transition-old(root) {
  animation: 300ms ease-out both fade-out;
}

::view-transition-new(root) {
  animation: 300ms ease-out both fade-in;
}

@keyframes fade-out {
  to { opacity: 0; }
}

@keyframes fade-in {
  from { opacity: 0; }
}
```

### 元素级过渡

通过为元素指定 `view-transition-name`，可以实现元素级的新旧状态之间的平滑过渡，比如产品图片从列表页到详情页的连续动画。

```css
/* ===== 命名过渡元素 ===== */
.hero-image {
  view-transition-name: hero-image;
}

.product-card {
  view-transition-name: product-card;
}

/* ===== 自定义元素过渡 ===== */
::view-transition-old(hero-image),
::view-transition-new(hero-image) {
  animation-duration: 500ms;
  animation-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
}

/* ===== 分组过渡 ===== */
.product-page .product-image {
  view-transition-name: product-image;
}

.product-page .product-title {
  view-transition-name: product-title;
}

.product-page .product-price {
  view-transition-name: product-price;
}

/* ===== 过渡效果组合 ===== */
::view-transition-old(product-card) {
  animation: 400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-out;
}

::view-transition-new(product-card) {
  animation: 400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-in;
}

@keyframes slide-out {
  to {
    transform: translateX(-100%);
    opacity: 0;
  }
}

@keyframes slide-in {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
}
```

### JavaScript 控制

```javascript
// ===== 手动触发过渡 =====
document.startViewTransition(async () => {
  // 更新 DOM
  await updateDOM();

  // 等待 DOM 更新完成
  await document.domUpdated;
});

// ===== 过渡状态回调 =====
const transition = document.startViewTransition(() => {
  updateDOM();
});

// 过渡开始
transition.ready.then(() => {
  console.log('过渡开始');
});

// 过渡结束
transition.finished.then(() => {
  console.log('过渡结束');
});

// 跳过过渡
transition.skipTransition();

// ===== 条件性过渡 =====
if (!document.startViewTransition) {
  // 降级处理
  updateDOM();
} else {
  document.startViewTransition(() => updateDOM());
}
```

---

## Scroll-driven Animations 滚动驱动动画

### 核心概念与原理

Scroll-driven Animations 是 CSS 的最新特性，允许动画直接与滚动位置关联，无需 JavaScript。这个特性让实现滚动触发动画、阅读进度条、视差效果变得前所未有的简单。

```css
/* ===== 基础滚动动画 ===== */
@keyframes fadeInOnScroll {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}

.scroll-element {
  animation: fadeInOnScroll linear;
  animation-timeline: view();
}

/* ===== 进度条动画 ===== */
@keyframes progressScroll {
  from { transform: scaleX(0); }
  to { transform: scaleX(1); }
}

.progress-bar {
  animation: progressScroll linear;
  animation-timeline: scroll(root block);
  transform-origin: left center;
}

/* ===== 视差效果 ===== */
.parallax-layer {
  animation: parallaxMove linear;
  animation-timeline: scroll();
}

@keyframes parallaxMove {
  from { transform: translateY(0); }
  to { transform: translateY(-100px); }
}

/* ===== 固定效果 ===== */
.sticky-element {
  animation: fixedOnScroll linear;
  animation-timeline: view();
  animation-range: entry 0% cover 50%;
}

@keyframes fixedOnScroll {
  from { position: relative; }
  to { position: fixed; top: 0; }
}
```

---

## 动画性能优化

### 渲染管线与动画优化原则

理解浏览器渲染管线是优化动画性能的基础。浏览器渲染页面时需要经过以下阶段：JavaScript 执行、样式计算、布局、绘制和合成。

动画触发的阶段越靠后，性能开销越小。最优的动画只使用 `transform` 和 `opacity`，因为这两个属性的变化可以在 GPU 的合成阶段处理，不需要触发布局和绘制。

```css
/* ===== 避免动画的属性 ===== */
/* 触发 Layout（重排）- 最慢 */
.bad-animations {
  animation: badWidth 1s ease;
  animation: badHeight 1s ease;
  animation: badMargin 1s ease;
  animation: badPadding 1s ease;
  animation: badTop 1s ease;
  animation: badLeft 1s ease;
  animation: badFontSize 1s ease;
}

/* ===== 推荐动画的属性 ===== */
/* 可合成动画 - GPU 加速 */
.good-animations {
  animation: goodTransform 1s ease;
  animation: goodOpacity 1s ease;
  animation: goodFilter 1s ease;
  animation: goodClipPath 1s ease;
  animation: goodMask 1s ease;
}
```

### GPU 加速技巧

```css
/* ===== 强制创建合成层 ===== */
.gpu-layer {
  /* 方式1：3D 变换 */
  transform: translateZ(0);
  transform: translate3d(0, 0, 0);

  /* 方式2：will-change 提示 */
  will-change: transform, opacity;

  /* 方式3：opacity（最轻量）*/
  opacity: 0.99;
}

/* ===== will-change 使用指南 ===== */
/* 动画开始前 */
.prepare {
  will-change: transform;
}

/* 动画进行中 */
.animating {
  animation: move 1s ease forwards;
}

/* 动画结束后 */
.animated {
  will-change: auto; /* 释放资源 */
}

/* ===== contain 属性 ===== */
.contained {
  contain: layout style paint;
}

/* ===== 分离动画层 ===== */
.layer-separation {
  isolation: isolate;
}
```

### 性能检测工具

```javascript
// ===== JavaScript 性能监控 =====
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log('Frame time:', entry.duration.toFixed(2), 'ms');
    if (entry.duration > 16.67) {
      console.warn('Frame dropped!');
    }
  });
});

observer.observe({ type: 'frame', buffered: true });

// ===== 动画帧率检测 =====
let lastTime = performance.now();
let frameCount = 0;
let fps = 0;

function checkFPS() {
  const now = performance.now();
  frameCount++;

  if (now - lastTime >= 1000) {
    fps = frameCount;
    console.log(`FPS: ${fps}`);
    frameCount = 0;
    lastTime = now;
  }

  requestAnimationFrame(checkFPS);
}

checkFPS();

// ===== 卡顿检测 =====
function detectJank() {
  let lastFrameTime = performance.now();

  function checkFrame(currentTime) {
    const delta = currentTime - lastFrameTime;

    if (delta > 16.67) {
      console.warn(`Jank detected: ${delta.toFixed(2)}ms (${(1000 / delta).toFixed(1)} FPS)`);
    }

    lastFrameTime = currentTime;
    requestAnimationFrame(checkFrame);
  }

  requestAnimationFrame(checkFrame);
}

detectJank();
```

---

## Motion 原则与最佳实践

### Material Design 运动原则

Material Design 定义了一套运动原则，帮助创建自然、和谐的动画体验。

```css
/* ===== 物理真实感 ===== */
/* 入场：快出慢 */
.entrance-animation {
  animation: slideIn 250ms cubic-bezier(0, 0, 0.2, 1);
}

/* 退场：快出快 */
.exit-animation {
  animation: slideOut 200ms cubic-bezier(0.4, 0, 1, 1);
}

/* ===== 响应及时 ===== */
.immediate-feedback {
  transition: transform 100ms ease-out;
}

/* ===== 时间层级系统 ===== */
:root {
  /* 极快：即时反馈 */
  --motion-instant: 50ms;

  /* 快：交互反馈 */
  --motion-fast: 100ms;
  --motion-quick: 150ms;

  /* 标准：UI 状态变化 */
  --motion-normal: 200ms;
  --motion-moderate: 300ms;

  /* 慢：页面过渡 */
  --motion-slow: 400ms;
  --motion-slower: 500ms;

  /* 极慢：复杂动画 */
  --motion-slowest: 800ms;
}
```

### 尊重用户偏好

```css
/* ===== 减少动画偏好 ===== */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    /* 禁用大多数动画 */
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}

/* ===== JavaScript 检测 ===== */
if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) {
  document.documentElement.classList.add('reduced-motion');
}

// 监听偏好变化
window.matchMedia('(prefers-reduced-motion: reduce)').addEventListener('change', (e) => {
  if (e.matches) {
    document.documentElement.classList.add('reduced-motion');
  } else {
    document.documentElement.classList.remove('reduced-motion');
  }
});
```

---

## 常见问题与解决方案

### 问题1：动画卡顿

**原因**：触发了 Layout 或 Paint

**解决**：使用 transform 和 opacity

```css
/* 使用 transform 而非 width */
.bad {
  width: 100%;
  animation: expand 1s ease;
}

.good {
  transform: scaleX(1);
  animation: expand 1s ease;
}

@keyframes expand {
  to { transform: scaleX(1); }
}
```

### 问题2：动画不流畅

**原因**：缺少 GPU 加速

**解决**：添加 will-change 或 transform

```css
.gpu-accelerated {
  transform: translateZ(0);
  will-change: transform, opacity;
}
```

### 问题3：动画闪烁

**原因**：backface-visibility 问题

**解决**：添加 backface-visibility 和 perspective

```css
.no-flicker {
  backface-visibility: hidden;
  perspective: 1000px;
}
```

---

## 动画时间指南

```css
:root {
  /* ===== 时间层级系统 ===== */

  /* 极快：即时反馈 */
  --motion-instant: 50ms;

  /* 快：交互反馈 */
  --motion-fast: 100ms;
  --motion-quick: 150ms;

  /* 标准：UI 状态变化 */
  --motion-normal: 200ms;
  --motion-moderate: 300ms;

  /* 慢：页面过渡 */
  --motion-slow: 400ms;
  --motion-slower: 500ms;

  /* 极慢：复杂动画 */
  --motion-slowest: 800ms;

  /* ===== 缓动曲线 ===== */
  --ease-standard: cubic-bezier(0.4, 0.0, 0.2, 1);
  --ease-emphasized: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-decelerated: cubic-bezier(0.0, 0.0, 0.2, 1);
  --ease-accelerated: cubic-bezier(0.4, 0.0, 1, 1);
}
```

---

> [!TIP]
> **动画最佳实践**：始终使用 `transform` 和 `opacity` 做动画；使用 `will-change` 提示但不要滥用；在 `prefers-reduced-motion` 时优雅降级；保持动画时间协调一致；使用 Material Design 的运动原则作为参考。

> [!RELATED]
> - [[现代CSS特性]] - CSS 新特性探索
> - [[设计系统与组件化]] - 组件化开发的完整指南
> - [[Tailwind-CSS深度指南]] - Tailwind CSS 完整指南
