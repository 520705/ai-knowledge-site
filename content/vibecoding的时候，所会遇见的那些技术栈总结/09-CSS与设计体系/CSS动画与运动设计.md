# CSS 动画与运动设计完全指南

> [!NOTE]
> 本文档涵盖 CSS 过渡、CSS 动画、View Transitions API、动画性能优化及 Motion 原则，是构建流畅 UI 动效的完整参考。

---

## 技术概述与定位

### 动画在现代 UI 设计中的角色

在现代 Web 应用中，动画不仅是视觉装饰，更是用户体验的核心组成部分。恰当的动画能够：

**提供视觉反馈**：让用户知道操作已被系统接收和响应。

**引导用户注意力**：通过运动轨迹引导用户关注重要信息。

**建立空间关系**：帮助用户理解界面元素的层次和位置关系。

**传达品牌个性**：通过独特的运动风格传达产品调性。

### CSS 动画技术栈

CSS 提供了完整的动画技术栈，从简单到复杂包括：

| 技术 | 适用场景 | 复杂度 |
|------|----------|--------|
| **Transitions** | 状态变化的平滑过渡 | 简单 |
| **Animations** | 复杂关键帧动画 | 中等 |
| **View Transitions** | 页面级过渡效果 | 中等 |
| **Scroll-driven** | 滚动触发动画 | 高级 |
| **Motion Path** | 路径动画 | 高级 |

### 浏览器支持现状（2026年）

| 特性 | Chrome | Firefox | Safari | Edge |
|------|--------|---------|--------|------|
| CSS Transitions | 26+ | 16+ | 9+ | 12+ |
| CSS Animations | 43+ | 16+ | 9+ | 12+ |
| View Transitions | 111+ | 老版本 | 18+ | 111+ |
| Scroll-driven | 115+ | 老版本 | 16+ | 115+ |
| Motion Path | 55+ | 老版本 | 15.4+ | 79+ |
| @keyframes | 43+ | 16+ | 9+ | 12+ |

### 为什么运动设计如此重要

在数字化产品中，运动设计已经成为了用户体验不可或缺的一部分。根据最新的用户体验研究数据：

**认知科学视角**：人类视觉系统对运动变化极为敏感。当界面元素发生动画变化时，用户的大脑会自动将其识别为重要的信息信号。这种本能反应使得动画成为引导用户注意力的最有效工具之一。

**情感连接**：精心设计的动画能够唤起用户的情感反应。例如，弹性动画给人以活力和现代感，而柔和的渐变动画则传达稳定和可信赖。不同的动画风格可以强化产品的品牌形象。

**空间感知**：在二维屏幕上展示三维空间感是 UI 设计的核心挑战。恰当的运动设计通过模拟真实世界的物理规律，帮助用户理解界面层次、元素关系和操作结果。

**状态反馈**：动画是最自然的交互反馈形式。当用户点击按钮、移动滑块或提交表单时，适当的动画变化能够立即确认操作已被系统接收和响应。

### 动画的性能影响

动画对性能的影响是设计时必须考虑的关键因素：

**帧率要求**：人类视觉系统能够感知 60fps（每秒 60 帧）以下的帧率变化。为了保证流畅的视觉体验，动画必须保持至少 30fps，而 60fps 是理想目标。任何低于这个标准的动画都会被视为卡顿。

**渲染管线**：浏览器渲染页面时需要经过布局（Layout）、绘制（Paint）和合成（Composite）三个阶段。动画触发的阶段越靠后，性能开销越小。最佳的动画属性是 `transform` 和 `opacity`，因为它们不会触发布局和重绘。

**GPU 加速**：现代浏览器能够将某些 CSS 属性（主要是 transform 和 opacity）的动画交给 GPU 处理，从而获得更好的性能。合理利用这一特性可以显著提升动画的流畅度。

---

## 完整安装与配置

### 开发环境准备

现代 CSS 动画开发需要良好的开发工具支持：

#### 基础项目结构

```bash
# 创建动画演示项目
mkdir css-animation-demo && cd css-animation-demo
npm init -y

# 安装开发依赖
npm install -D vite autoprefixer postcss
```

#### Vite 配置

```javascript
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    postcss: {
      plugins: [
        // 自动添加浏览器前缀
        autoprefixer(),
      ],
    },
  },
  // 开发服务器配置
  server: {
    port: 3000,
    host: true,
  },
  // 构建配置
  build: {
    target: 'esnext',
    cssCodeSplit: false, // 动画性能优化
  },
});
```

#### CSS 文件组织

```css
/* styles/animation/base.css - 动画基础 */
@layer animation.base {
  /* 关键帧定义 */
  @keyframes fadeIn {
    from { opacity: 0; }
    to { opacity: 1; }
  }
  
  @keyframes slideUp {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  @keyframes scaleIn {
    from {
      opacity: 0;
      transform: scale(0.95);
    }
    to {
      opacity: 1;
      transform: scale(1);
    }
  }
}

/* styles/animation/utilities.css - 动画工具类 */
@layer animation.utilities {
  .animate-fade-in {
    animation: fadeIn 400ms ease forwards;
  }
  
  .animate-slide-up {
    animation: slideUp 400ms cubic-bezier(0.16, 1, 0.3, 1) forwards;
  }
  
  .animate-scale-in {
    animation: scaleIn 300ms ease forwards;
  }
  
  .animate-pulse {
    animation: pulse 2s ease-in-out infinite;
  }
  
  .animate-spin {
    animation: spin 1s linear infinite;
  }
}

/* styles/animation/components.css - 组件动画 */
@layer animation.components {
  .button {
    transition: all 200ms ease;
  }
  
  .modal-backdrop {
    animation: fadeIn 200ms ease forwards;
  }
  
  .dropdown-menu {
    animation: slideDown 200ms ease forwards;
  }
}
```

### 动画库选择

对于复杂动画需求，可以考虑使用动画库：

```bash
# Motion（Framer Motion 的 CSS 版本）
npm install motion

# Anime.js
npm install animejs

# GSAP (GreenSock)
npm install gsap
```

### 动画开发工具配置

```javascript
// vite.config.js - 完整的动画开发配置
import { defineConfig } from 'vite';
import autoprefixer from 'autoprefixer';

export default defineConfig({
  css: {
    postcss: {
      plugins: [
        autoprefixer({
          // 支持的浏览器
          flexbox: 'no-2009',
          grid: true,
        }),
      ],
    },
  },
  server: {
    port: 3000,
    host: true,
    // 允许动画热更新
    hmr: {
      overlay: true,
    },
  },
  build: {
    target: 'esnext',
    cssCodeSplit: true,
    // 分离关键 CSS
    cssTarget: 'chrome80',
  },
});
```

---

## 核心概念详解

### CSS Transitions（过渡）

#### 深度原理解析

CSS Transitions 是最简单的动画形式，用于在元素状态变化时创建平滑过渡效果。

**核心属性**：
- `transition-property`：指定要过渡的 CSS 属性
- `transition-duration`：过渡持续时间
- `transition-timing-function`：缓动函数
- `transition-delay`：延迟时间

```css
/* 完整语法 */
.element {
  transition-property: all;                  /* 属性名 */
  transition-duration: 300ms;                /* 持续时间 */
  transition-timing-function: ease;          /* 缓动函数 */
  transition-delay: 0s;                      /* 延迟 */
  
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

#### 缓动函数详解

缓动函数决定了动画的速度曲线，是动画质量的关键：

```css
/* ===== 内置缓动函数 ===== */
.ease { transition-timing-function: ease; }           /* 慢-快-慢 */
.linear { transition-timing-function: linear; }       /* 匀速 */
.ease-in { transition-timing-function: ease-in; }     /* 慢-快 */
.ease-out { transition-timing-function: ease-out; }  /* 快-慢 */
.ease-in-out { transition-timing-function: ease-in-out; } /* 慢-快-慢 */

/* ===== steps() 分步函数 ===== */
.steps-start { transition-timing-function: steps(4, start); }    /* 4步，起点跳变 */
.steps-end { transition-timing-function: steps(4, end); }        /* 4步，终点跳变 */
.steps-both { transition-timing-function: steps(4, start end); }  /* 首尾跳变 */

/* ===== cubic-bezier() 自定义曲线 ===== */
/* 标准曲线 */
.bezier-ease { transition-timing-function: cubic-bezier(0.25, 0.1, 0.25, 1); }
.bezier-ease-in { transition-timing-function: cubic-bezier(0.42, 0, 1, 1); }
.bezier-ease-out { transition-timing-function: cubic-bezier(0, 0, 0.58, 1); }
.bezier-ease-in-out { transition-timing-function: cubic-bezier(0.42, 0, 0.58, 1); }

/* 弹性效果 */
.bezier-bounce { transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1); }
.bezier-elastic { transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55); }
.bezier-back { transition-timing-function: cubic-bezier(0.34, 1.56, 0.64, 1.2); }

/* 特殊曲线 */
.bezier-smooth { transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1); }
.bezier-snappy { transition-timing-function: cubic-bezier(0.2, 0, 0, 1); }
```

#### 常用缓动曲线参考

```css
:root {
  /* Material Design 曲线 */
  --ease-standard: cubic-bezier(0.4, 0.0, 0.2, 1);         /* 标准 */
  --ease-decelerate: cubic-bezier(0.0, 0.0, 0.2, 1);        /* 入场 */
  --ease-accelerate: cubic-bezier(0.4, 0.0, 1, 1);           /* 退场 */
  --ease-sharp: cubic-bezier(0.4, 0.0, 0.6, 1);             /* 锐利 */
  
  /* 弹簧效果 */
  --ease-spring: cubic-bezier(0.175, 0.885, 0.32, 1.275);    /* 强回弹 */
  --ease-spring-smooth: cubic-bezier(0.23, 1, 0.32, 1);      /* 平滑回弹 */
  
  /* 自定义曲线 */
  --ease-smooth-out: cubic-bezier(0, 0, 0.2, 1);
  --ease-quick-out: cubic-bezier(0.2, 0, 0, 1);
  --ease-moderate: cubic-bezier(0.4, 0, 0.2, 1);
}

/* 应用示例 */
.entrance {
  transition: all 300ms var(--ease-decelerate);
}

.exit {
  transition: all 200ms var(--ease-accelerate);
}

.emphasis {
  transition: all 400ms var(--ease-spring);
}
```

#### 实战：按钮状态动画

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

#### 过渡性能优化

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
  transition: top 200ms ease;       /* 触发重排 */
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
.will-change-demo {
  will-change: transform, opacity;
  transition: transform 200ms ease, opacity 200ms ease;
}

/* 动画结束后移除 will-change */
.will-change-demo.animated {
  will-change: auto; /* 释放资源 */
}
```

### CSS Animations（关键帧动画）

#### 深度原理解析

CSS Animations 与 Transitions 的核心区别在于：Animations 可以定义多个关键帧，实现更复杂的动画效果。

**核心属性**：
- `animation-name`：关键帧名称
- `animation-duration`：动画持续时间
- `animation-timing-function`：缓动函数
- `animation-iteration-count`：迭代次数
- `animation-direction`：播放方向
- `animation-fill-mode`：填充模式
- `animation-play-state`：播放状态

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

#### 关键帧定义详解

```css
/* ===== 基础关键帧 ===== */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
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

/* ===== 多步骤关键帧 ===== */
@keyframes progressBar {
  0% { width: 0%; }
  20% { width: 20%; background-color: #ef4444; }
  40% { width: 45%; background-color: #f59e0b; }
  60% { width: 70%; background-color: #eab308; }
  80% { width: 85%; background-color: #22c55e; }
  100% { width: 100%; background-color: #10b981; }
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

/* ===== 闪烁效果 ===== */
@keyframes blink {
  0%, 50%, 100% { opacity: 1; }
  25%, 75% { opacity: 0; }
}

/* ===== 渐变动画 ===== */
@keyframes gradientShift {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

/* ===== 旋转效果 ===== */
@keyframes rotate360 {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes rotateFromOrigin {
  0% { transform: rotate(-180deg); }
  100% { transform: rotate(0deg); }
}
```

#### 动画属性详解

```css
/* ===== 迭代次数 ===== */
.iteration-once { animation-iteration-count: 1; }           /* 播放1次 */
.iteration-twice { animation-iteration-count: 2; }           /* 播放2次 */
.iteration-three { animation-iteration-count: 3; }           /* 播放3次 */
.iteration-infinite { animation-iteration-count: infinite; } /* 无限循环 */

/* ===== 播放方向 ===== */
.direction-normal { animation-direction: normal; }                 /* 正向 */
.direction-reverse { animation-direction: reverse; }              /* 反向 */
.direction-alternate { animation-direction: alternate; }        /* 正反交替 */
.direction-alternate-reverse { animation-direction: alternate-reverse; } /* 先反后正 */

/* ===== 填充模式 ===== */
.fill-none { animation-fill-mode: none; }      /* 动画前后保持原样 */
.fill-forwards { animation-fill-mode: forwards; }  /* 动画结束后保持最终状态 */
.fill-backwards { animation-fill-mode: backwards; } /* 动画开始前显示第一帧 */
.fill-both { animation-fill-mode: both; }           /* 向前向后结合 */

/* ===== 播放状态 ===== */
.playing { animation-play-state: running; }  /* 播放中 */
.paused { animation-play-state: paused; }   /* 暂停 */

/* 悬停暂停示例 */
.hoverable-animation:hover {
  animation-play-state: paused;
}

/* 点击暂停/播放 */
.pause-on-click:active {
  animation-play-state: paused;
}

/* 组合使用 */
.complex-animation {
  animation: 
    fadeIn 400ms ease forwards,
    slideUp 400ms ease 100ms forwards,
    scaleIn 300ms ease 200ms forwards;
}
```

#### 实战：页面加载动画

```css
/* ===== 骨架屏闪烁 ===== */
@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

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

/* 骨架屏内容 */
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

/* ===== 内容淡入 ===== */
.page-content {
  animation: fadeIn 600ms ease forwards;
}

/* ===== 加载完成移除动画 ===== */
.loader-hidden {
  animation: fadeOut 300ms ease forwards;
}

@keyframes fadeOut {
  to { opacity: 0; visibility: hidden; }
}
```

#### 实战：复杂交互动画

```css
/* ===== 卡片悬停效果 ===== */
.card {
  position: relative;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  transition: transform 300ms ease, box-shadow 300ms ease;
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

/* ===== 卡片内容过渡 ===== */
.card-content {
  transition: opacity 300ms ease, transform 300ms ease;
}

.card:hover .card-content {
  opacity: 1;
  transform: translateY(0);
}

/* 初始隐藏状态 */
.card-content-hidden {
  opacity: 0;
  transform: translateY(10px);
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

/* ===== 标签切换动画 ===== */
.tab-content {
  animation: fadeSlideIn 300ms ease forwards;
}

@keyframes fadeSlideIn {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
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

### View Transitions API

#### 深度原理解析

View Transitions API 是现代 CSS 的革命性特性，它允许开发者创建页面级和元素级的平滑过渡效果。

**核心概念**：
- **View Transition**：两个视图状态之间的平滑过渡
- **View Transition Name**：元素的过渡标识
- **默认过渡**：整个页面的淡入淡出
- **自定义过渡**：元素级别的动画效果

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

#### 元素级过渡

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

#### JavaScript 控制

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

// 过渡失败
transition.skipTransition();

// ===== 监听过渡事件 =====
document.addEventListener('viewtransitionstart', (e) => {
  console.log('过渡开始', e.viewTransitionName);
});

document.addEventListener('viewtransitionend', (e) => {
  console.log('过渡结束', e.viewTransitionName);
});

document.addEventListener('viewtransitioncancel', (e) => {
  console.log('过渡取消');
});

// ===== 条件性过渡 =====
if (!document.startViewTransition) {
  // 降级处理
  updateDOM();
} else {
  document.startViewTransition(() => updateDOM());
}
```

### Scroll-driven Animations（滚动驱动动画）

#### 深度原理解析

Scroll-driven Animations 是 CSS 的最新特性，允许动画直接与滚动位置关联，无需 JavaScript：

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

### Motion Path（路径动画）

```css
/* ===== 路径定义 ===== */
.motion-path {
  offset-path: path('M 0 0 L 100 100 Q 200 0 300 100');
  offset-rotate: auto 90deg; /* 自动旋转跟随路径 */
  animation: moveAlongPath 2s ease-in-out infinite;
}

@keyframes moveAlongPath {
  0% { offset-distance: 0%; }
  100% { offset-distance: 100%; }
}

/* ===== 圆形路径 ===== */
.circular-path {
  offset-path: circle(100px at 50% 50%);
  animation: moveCircular 4s linear infinite;
}

@keyframes moveCircular {
  from { offset-distance: 0%; }
  to { offset-distance: 100%; }
}
```

---

## 动画性能优化

### 触发重排/重绘的属性

```css
/* ===== 避免动画的属性 ===== */
/* 触发 Layout（重排）- 最慢 */
.bad-animations {
  animation: badWidth 1s ease;      /* width */
  animation: badHeight 1s ease;     /* height */
  animation: badMargin 1s ease;     /* margin */
  animation: badPadding 1s ease;    /* padding */
  animation: badTop 1s ease;         /* top */
  animation: badLeft 1s ease;         /* left */
  animation: badFontSize 1s ease;    /* font-size */
}

/* 触发 Paint（重绘）- 中等 */
/* color, background, border-color, visibility, box-shadow, text-shadow */

/* ===== 推荐动画的属性 ===== */
/* 可合成动画 - GPU 加速 */
.good-animations {
  animation: goodTransform 1s ease;   /* transform */
  animation: goodOpacity 1s ease;     /* opacity */
  animation: goodFilter 1s ease;      /* filter */
  animation: goodClipPath 1s ease;     /* clip-path */
  animation: goodMask 1s ease;         /* mask-image */
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
// ===== Chrome DevTools Performance 面板 =====
// 1. 打开 DevTools (F12)
// 2. 选择 Performance 标签
// 3. 点击录制按钮
// 4. 执行动画操作
// 5. 停止录制，分析火焰图

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
    
    if (delta > 16.67) { // 超过 60fps 帧时间
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

## Motion 原则

### Material Design 运动原则

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

/* ===== 协调一致 ===== */
:root {
  /* 时间系统 */
  --duration-instant: 50ms;     /* 极快：交互反馈 */
  --duration-fast: 100ms;       /* 快：状态切换 */
  --duration-quick: 150ms;      /* 较快：小型组件 */
  --duration-normal: 200ms;     /* 标准：大多数动画 */
  --duration-moderate: 300ms;   /* 中等：较大组件 */
  --duration-slow: 400ms;       /* 慢：页面元素 */
  --duration-slower: 500ms;      /* 较慢：复杂动画 */
  --duration-slowest: 800ms;    /* 极慢：整体过渡 */
}

/* ===== 协调原则 ===== */
.coordinated-animation {
  /* 组件内部过渡协调 */
  transition: 
    transform var(--duration-fast) var(--ease-out),
    background-color var(--duration-normal) var(--ease-standard),
    box-shadow var(--duration-normal) var(--ease-standard);
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
    
    /* 保留功能性过渡（无运动）*/
    /* 移除位置变化和缩放动画 */
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

/* ===== 条件性动画 ===== */
.optional-animation {
  /* 默认动画 */
  transition: transform 300ms ease;
}

@media (prefers-reduced-motion: reduce) {
  .optional-animation {
    /* 简化动画或禁用 */
    transition: none;
  }
}
```

### 动画时间指南

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

## 常用命令与操作

### 动画调试命令

```bash
# 浏览器控制台调试
# 查看动画性能
performance.getEntriesByType('animation')[0]

# 获取动画元素
document.getAnimations()

# 暂停所有动画
document.getAnimations().forEach(a => a.pause())

# 恢复所有动画
document.getAnimations().forEach(a => a.play())
```

### 常用工具

```bash
# CSS 动画验证
npm install -D stylelint-plugin-motion

# 性能监控
npm install -D web-vitals
```

---

## 高级配置与技巧

### 组合动画

```css
/* ===== 多动画并行 ===== */
.multi-animation {
  animation: 
    fadeIn 400ms ease forwards,
    slideUp 400ms ease 100ms forwards,
    scaleIn 300ms ease 200ms forwards;
}

/* ===== 动画延迟序列 ===== */
.staggered-list .item:nth-child(1) { animation-delay: 0ms; }
.staggered-list .item:nth-child(2) { animation-delay: 100ms; }
.staggered-list .item:nth-child(3) { animation-delay: 200ms; }
.staggered-list .item:nth-child(4) { animation-delay: 300ms; }
.staggered-list .item:nth-child(5) { animation-delay: 400ms; }

/* 使用 CSS 变量自动计算 */
.staggered-item {
  --delay: 100ms;
  animation: fadeIn 400ms ease calc(var(--index) * var(--delay)) forwards;
}
```

### 动画状态管理

```css
/* ===== 动画状态类 ===== */
.animating { /* 动画进行中 */ }
.animated { /* 动画已完成 */ }
.paused { animation-play-state: paused; }
.running { animation-play-state: running; }

/* ===== 条件动画 ===== */
.visible .animate-on-visible {
  animation: fadeIn 400ms ease forwards;
}

[data-animate="true"] .animate {
  animation: slideUp 500ms ease forwards;
}
```

### 复杂动画模式

```css
/* ===== 打字机效果 ===== */
@keyframes typewriter {
  from { width: 0; }
  to { width: 100%; }
}

.typewriter {
  overflow: hidden;
  white-space: nowrap;
  animation: typewriter 3s steps(30) forwards;
}

/* ===== 光标闪烁 ===== */
.typewriter::after {
  content: '|';
  animation: cursorBlink 1s step-end infinite;
}

@keyframes cursorBlink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

/* ===== 渐变文字 ===== */
.gradient-text {
  background: linear-gradient(90deg, #667eea, #764ba2, #f093fb);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 3s linear infinite;
}

@keyframes gradientShift {
  to { background-position: 200% center; }
}

/* ===== 霓虹灯效果 ===== */
.neon {
  text-shadow: 
    0 0 5px #fff,
    0 0 10px #fff,
    0 0 20px #228dff,
    0 0 30px #228dff,
    0 0 40px #228dff;
  animation: neonPulse 1.5s ease-in-out infinite alternate;
}

@keyframes neonPulse {
  from {
    text-shadow: 
      0 0 5px #fff,
      0 0 10px #fff,
      0 0 20px #228dff,
      0 0 30px #228dff;
  }
  to {
    text-shadow: 
      0 0 10px #fff,
      0 0 20px #fff,
      0 0 30px #228dff,
      0 0 40px #228dff,
      0 0 50px #228dff,
      0 0 60px #228dff;
  }
}

/* ===== 浮雕效果 ===== */
.embossed {
  color: #888;
  text-shadow: 
    -1px -1px 1px #fff,
    1px 1px 1px #000;
}

/* ===== 描边动画 ===== */
.stroke-animation {
  -webkit-text-stroke: 2px #333;
  color: transparent;
  animation: strokeReveal 2s ease forwards;
}

@keyframes strokeReveal {
  to {
    color: #333;
  }
}
```

### 3D 变换动画

```css
/* ===== 3D 透视容器 ===== */
.perspective-container {
  perspective: 1000px;
  perspective-origin: center center;
}

/* ===== 3D 翻转卡片 ===== */
.flip-card {
  width: 300px;
  height: 400px;
  perspective: 1000px;
}

.flip-card-inner {
  width: 100%;
  height: 100%;
  position: relative;
  transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  transform-style: preserve-3d;
}

.flip-card:hover .flip-card-inner {
  transform: rotateY(180deg);
}

.flip-card-front,
.flip-card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
}

.flip-card-front {
  background: #3b82f6;
}

.flip-card-back {
  background: #8b5cf6;
  transform: rotateY(180deg);
}

/* ===== 3D 旋转立方体 ===== */
.cube-container {
  width: 200px;
  height: 200px;
  perspective: 500px;
}

.cube {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  animation: cubeRotate 10s linear infinite;
}

@keyframes cubeRotate {
  from { transform: rotateX(0) rotateY(0); }
  to { transform: rotateX(360deg) rotateY(360deg); }
}

.cube-face {
  position: absolute;
  width: 200px;
  height: 200px;
  opacity: 0.8;
}

.cube-face-front { transform: rotateY(0deg) translateZ(100px); background: #ef4444; }
.cube-face-back { transform: rotateY(180deg) translateZ(100px); background: #f59e0b; }
.cube-face-right { transform: rotateY(90deg) translateZ(100px); background: #22c55e; }
.cube-face-left { transform: rotateY(-90deg) translateZ(100px); background: #3b82f6; }
.cube-face-top { transform: rotateX(90deg) translateZ(100px); background: #8b5cf6; }
.cube-face-bottom { transform: rotateX(-90deg) translateZ(100px); background: #ec4899; }

/* ===== 3D 倾斜效果 ===== */
.tilt-card {
  transform-style: preserve-3d;
  transition: transform 0.3s ease;
}

.tilt-card:hover {
  transform: perspective(1000px) rotateX(5deg) rotateY(-5deg);
}
```

### SVG 动画技巧

```css
/* ===== SVG 路径绘制动画 ===== */
.svg-draw {
  stroke-dasharray: 1000;
  stroke-dashoffset: 1000;
  animation: drawPath 3s ease forwards;
}

@keyframes drawPath {
  to { stroke-dashoffset: 0; }
}

/* ===== SVG 填充动画 ===== */
.svg-fill {
  animation: fillSvg 1s ease forwards;
}

@keyframes fillSvg {
  from { fill: transparent; }
  to { fill: #3b82f6; }
}

/* ===== SVG 模糊动画 ===== */
.svg-blur {
  filter: blur(0);
  animation: blurSvg 0.5s ease forwards;
}

@keyframes blurSvg {
  to { filter: blur(10px); }
}

/* ===== SVG 缩放动画 ===== */
.svg-scale {
  transform-origin: center;
  animation: scaleSvg 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

@keyframes scaleSvg {
  from { transform: scale(0); }
  to { transform: scale(1); }
}
```

---

## 与同类技术对比

### CSS 动画 vs JavaScript 动画

| 维度 | CSS 动画 | JavaScript 动画 |
|------|----------|-----------------|
| **性能** | GPU 加速，优秀 | 依赖实现方式 |
| **控制力** | 有限 | 精确控制 |
| **复杂度** | 简单场景优秀 | 复杂场景灵活 |
| **交互响应** | 原生支持 | 需要手动实现 |
| **适用场景** | 状态变化、过渡 | 物理模拟、游戏 |

### 动画库对比

| 库 | 特点 | 适用场景 |
|----|------|----------|
| **Framer Motion** | React 专用，功能强大 | React 应用 |
| **GSAP** | 工业级性能，插件丰富 | 复杂动画 |
| **Anime.js** | 轻量，API 友好 | 简单到中等复杂度 |
| **Motion** | 声明式，Vue 支持 | Vue 应用 |

### 选择指南

```markdown
## 动画技术选择指南

### 简单状态变化
- 推荐：CSS Transitions
- 示例：按钮悬停、颜色变化

### 复杂关键帧动画
- 推荐：CSS Animations
- 示例：加载动画、入场动画

### 页面级过渡
- 推荐：View Transitions API
- 示例：页面切换、视图转换

### 滚动关联动画
- 推荐：Scroll-driven Animations
- 示例：滚动进度条、视差效果

### 物理模拟/游戏
- 推荐：JavaScript 动画库（GSAP）
- 示例：拖拽、物理碰撞

### 复杂交互
- 推荐：Framer Motion / GSAP
- 示例：拖拽排序、手势操作
```

---

## 常见问题与解决方案

### 问题1：动画卡顿

**原因**：触发了 Layout 或 Paint

**解决**：

```css
/* 使用 transform 和 opacity */
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

**解决**：

```css
.gpu-accelerated {
  transform: translateZ(0);
  will-change: transform, opacity;
}
```

### 问题3：动画闪烁

**原因**：backface-visibility 问题

**解决**：

```css
.no-flicker {
  backface-visibility: hidden;
  perspective: 1000px;
}
```

### 问题4：动画不同步

**原因**：缓动函数不匹配

**解决**：

```css
/* 统一使用相同的缓动函数 */
.element {
  transition-timing-function: cubic-bezier(0.16, 1, 0.3, 1);
}

@keyframes slideIn {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}

.element {
  animation: slideIn 300ms cubic-bezier(0.16, 1, 0.3, 1);
}
```

### 问题5：动画不触发

**原因**：伪元素动画需要特殊处理

**解决**：

```css
/* 伪元素动画 */
.element::after {
  content: '';
  position: absolute;
  inset: 0;
  opacity: 0;
  transition: opacity 200ms ease;
}

.element:hover::after {
  opacity: 1;
}

/* 确保伪元素有内容 */
.element::before {
  content: '';
  /* 其他样式 */
}
```

---

## 实战项目示例

### 示例1：页面加载动画

```css
/* page-loader.css */

/* ===== 加载器容器 ===== */
.page-loader {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: white;
  transition: opacity 400ms ease, visibility 400ms ease;
}

.page-loader.loaded {
  opacity: 0;
  visibility: hidden;
  pointer-events: none;
}

/* ===== Logo 动画 ===== */
.loader-logo {
  width: 80px;
  height: 80px;
  margin-bottom: 32px;
  animation: logoPulse 1.5s ease-in-out infinite;
}

@keyframes logoPulse {
  0%, 100% {
    transform: scale(1);
    opacity: 1;
  }
  50% {
    transform: scale(1.05);
    opacity: 0.8;
  }
}

/* ===== 加载指示器 ===== */
.loader-dots {
  display: flex;
  gap: 8px;
}

.loader-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #3b82f6;
  animation: dotBounce 1.4s ease-in-out infinite;
}

.loader-dot:nth-child(1) { animation-delay: 0ms; }
.loader-dot:nth-child(2) { animation-delay: 100ms; }
.loader-dot:nth-child(3) { animation-delay: 200ms; }

@keyframes dotBounce {
  0%, 80%, 100% {
    transform: scale(0.6);
    opacity: 0.5;
  }
  40% {
    transform: scale(1);
    opacity: 1;
  }
}

/* ===== 进度条加载器 ===== */
.loader-progress {
  width: 200px;
  height: 4px;
  background: #e5e7eb;
  border-radius: 2px;
  overflow: hidden;
  margin-top: 24px;
}

.loader-progress-bar {
  height: 100%;
  background: linear-gradient(90deg, #3b82f6, #8b5cf6);
  animation: progressLoad 2s ease-in-out forwards;
}

@keyframes progressLoad {
  from { width: 0%; }
  to { width: 100%; }
}

/* ===== 文字动画 ===== */
.loader-text {
  margin-top: 16px;
  color: #6b7280;
  font-size: 14px;
  animation: textFade 300ms ease 200ms forwards;
  opacity: 0;
}

@keyframes textFade {
  to { opacity: 1; }
}
```

### 示例2：卡片交互动画

```css
/* card-interaction.css */

/* ===== 卡片基础 ===== */
.interactive-card {
  position: relative;
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  transition: 
    transform 300ms cubic-bezier(0.16, 1, 0.3, 1),
    box-shadow 300ms ease;
}

.interactive-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.15);
}

/* ===== 顶部装饰条 ===== */
.interactive-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #3b82f6, #8b5cf6, #ec4899);
  transform: scaleX(0);
  transform-origin: left;
  transition: transform 300ms cubic-bezier(0.16, 1, 0.3, 1);
}

.interactive-card:hover::before {
  transform: scaleX(1);
}

/* ===== 图片缩放 ===== */
.card-media {
  position: relative;
  overflow: hidden;
}

.card-media img {
  width: 100%;
  aspect-ratio: 16/10;
  object-fit: cover;
  transition: transform 500ms cubic-bezier(0.16, 1, 0.3, 1);
}

.interactive-card:hover .card-media img {
  transform: scale(1.1);
}

/* ===== 图片遮罩 ===== */
.card-media::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(
    to top,
    rgba(0, 0, 0, 0.4) 0%,
    transparent 50%
  );
  opacity: 0;
  transition: opacity 300ms ease;
}

.interactive-card:hover .card-media::after {
  opacity: 1;
}

/* ===== 内容区域 ===== */
.card-body {
  padding: 24px;
  transition: transform 300ms ease;
}

.interactive-card:hover .card-body {
  transform: translateY(-4px);
}

/* ===== 标题动画 ===== */
.card-title {
  font-size: 1.25rem;
  font-weight: 600;
  margin-bottom: 8px;
  transition: color 200ms ease;
}

.interactive-card:hover .card-title {
  color: #3b82f6;
}

/* ===== 描述文字 ===== */
.card-description {
  color: #6b7280;
  font-size: 0.875rem;
  line-height: 1.6;
  margin-bottom: 16px;
}

/* ===== 操作按钮 ===== */
.card-actions {
  display: flex;
  gap: 12px;
  opacity: 0;
  transform: translateY(10px);
  transition: 
    opacity 300ms ease,
    transform 300ms ease;
}

.interactive-card:hover .card-actions {
  opacity: 1;
  transform: translateY(0);
}

.card-btn {
  flex: 1;
  padding: 10px 16px;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  text-align: center;
  cursor: pointer;
  transition: all 200ms ease;
}

.card-btn-primary {
  background: #3b82f6;
  color: white;
  border: none;
}

.card-btn-primary:hover {
  background: #2563eb;
  transform: translateY(-1px);
}

.card-btn-secondary {
  background: white;
  color: #3b82f6;
  border: 1px solid #3b82f6;
}

.card-btn-secondary:hover {
  background: #eff6ff;
}
```

---

## 附录：动画性能测试工具

### Chrome DevTools Animation 检查

```javascript
// 打开 DevTools → More Tools → Animations
// 可以查看：
// 1. 动画时间线
// 2. 每帧的 CSS 属性变化
// 3. 性能影响评估
```

### FPS 计数器

```javascript
// 创建 FPS 计数器
(function() {
  const frameTimes = [];
  let lastTime = performance.now();
  
  function updateFPS() {
    const now = performance.now();
    const delta = now - lastTime;
    frameTimes.push(delta);
    
    if (frameTimes.length > 60) {
      frameTimes.shift();
    }
    
    const avgFrameTime = frameTimes.reduce((a, b) => a + b, 0) / frameTimes.length;
    const fps = Math.round(1000 / avgFrameTime);
    
    // 更新显示
    const fpsDisplay = document.getElementById('fps-counter');
    if (fpsDisplay) {
      fpsDisplay.textContent = `${fps} FPS`;
      fpsDisplay.style.color = fps >= 50 ? '#22c55e' : fps >= 30 ? '#f59e0b' : '#ef4444';
    }
    
    lastTime = now;
    requestAnimationFrame(updateFPS);
  }
  
  // 启动
  if (document.readyState === 'complete') {
    updateFPS();
  } else {
    window.addEventListener('load', updateFPS);
  }
})();
```

### 性能分析清单

```markdown
## 动画性能检查清单

- [ ] 仅使用 transform 和 opacity 动画
- [ ] 使用 will-change 但不过度使用
- [ ] 动画元素使用 contain 属性
- [ ] 避免在动画期间触发布局
- [ ] 使用 requestAnimationFrame 同步 JavaScript 动画
- [ ] 测试 prefers-reduced-motion 支持
- [ ] 使用 Chrome DevTools Performance 面板分析
- [ ] 确保 60 FPS 运行
```

---

## 附录：动画资源推荐

### 在线缓动曲线编辑器

- [cubic-bezier.com](https://cubic-bezier.com) - 可视化贝塞尔曲线
- [easings.net](https://easings.net) - 常见缓动函数库
- [theappdesigner.io](http://theappdesigner.io/frontend/easing-gallery) - 缓动曲线库

### 动画库对比

| 库 | 大小 | 特点 | 适用场景 |
|----|------|------|----------|
| **Framer Motion** | ~50KB | React 专用，功能全面 | React 应用 |
| **GSAP** | ~30KB (核心) | 工业级性能 | 复杂动画 |
| **Anime.js** | ~20KB | 轻量简洁 | 简单到中等 |
| **Motion** | ~10KB | Vue 3 友好 | Vue 应用 |
| **Lottie** | 依赖库 | AE 导出 | 复杂矢量动画 |
| **CSS** | 0KB | 原生，性能最优 | 简单动画 |

---

## 附录：常见动画效果代码片段

### 打字机效果

```css
@keyframes typewriter {
  from { width: 0; }
  to { width: 100%; }
}

.typewriter {
  overflow: hidden;
  white-space: nowrap;
  animation: typewriter 3s steps(30) forwards;
}
```

### 渐变文字

```css
.gradient-text {
  background: linear-gradient(90deg, #667eea, #764ba2, #f093fb);
  background-size: 200% auto;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: gradientShift 3s linear infinite;
}

@keyframes gradientShift {
  to { background-position: 200% center; }
}
```

### 霓虹灯效果

```css
.neon {
  text-shadow: 
    0 0 5px #fff,
    0 0 10px #fff,
    0 0 20px #228dff,
    0 0 30px #228dff,
    0 0 40px #228dff;
  animation: neonPulse 1.5s ease-in-out infinite alternate;
}

@keyframes neonPulse {
  from {
    text-shadow: 
      0 0 5px #fff,
      0 0 10px #fff,
      0 0 20px #228dff,
      0 0 30px #228dff;
  }
  to {
    text-shadow: 
      0 0 10px #fff,
      0 0 20px #fff,
      0 0 30px #228dff,
      0 0 40px #228dff,
      0 0 50px #228dff,
      0 0 60px #228dff;
  }
}
```

---

> [!TIP]
> **动画最佳实践**：始终使用 `transform` 和 `opacity` 做动画；使用 `will-change` 提示但不要滥用；在 `prefers-reduced-motion` 时优雅降级；保持动画时间协调一致。
