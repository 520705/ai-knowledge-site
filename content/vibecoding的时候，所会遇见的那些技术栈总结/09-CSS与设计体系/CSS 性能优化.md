# CSS 性能优化

> 本文档系统性地剖析 CSS 性能优化的核心原理，从浏览器的渲染流水线入手，深入讲解重排重绘机制、动画优化策略、选择器性能、关键渲染路径等关键议题，并提供可操作的性能检测工具与兼容性处理方案。

## 技术概述与定位

### CSS 性能优化的重要性

在现代 Web 应用中，CSS 性能对用户体验有着直接影响。当用户与页面交互时，任何可感知的延迟或卡顿都会降低用户满意度。CSS 作为渲染流水线的关键环节，其编写质量直接影响页面的首次绘制时间、交互响应速度和滚动流畅度。掌握 CSS 性能优化的原理和技巧，是前端工程师提升用户体验的关键能力。

CSS 性能问题通常体现在几个方面：渲染阻塞导致页面白屏时间过长、频繁的重排重绘造成卡顿、不必要的样式计算消耗 CPU 资源、以及动画掉帧影响视觉体验。这些问题在大中型应用中尤为明显，因为代码量越大，维护者越多，样式代码的质量越容易参差不齐。

理解 CSS 性能优化的第一步是掌握浏览器的渲染流水线。当浏览器接收到 HTML 文档时，它会经历一个复杂的过程将文档转换为屏幕上的像素。

### 1.1 浏览器渲染流水线

浏览器渲染页面的完整流程如下：

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   HTML/CSS  │───▶│   DOM/CSSOM │───▶│  Render Tree│───▶│    Layout   │
│   下载解析   │    │    构建     │    │    构建     │    │   (重排)    │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
                                                               │
                                                               ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   合成图层   │◀───│   Paint    │◀───│   分层      │◀───│   更新布局   │
│   (Composite)│    │  (重绘)    │    │   (Layer)   │    │   信息      │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

| 阶段 | 触发条件 | 性能影响 |
|------|----------|----------|
| **DOM/CSSOM 构建** | HTML/CSS 解析完成 | 首次渲染关键路径 |
| **Render Tree 构建** | DOM 与 CSSOM 合并 | 决定可见元素 |
| **Layout（布局/重排）** | 几何属性变化 | **高成本**，触发后续所有阶段 |
| **Paint（绘制）** | 视觉属性变化 | 中等成本，可能不需要重排 |
| **Composite（合成）** | transform/opacity 变化 | **最低成本**，GPU 加速 |

### 1.2 重排（Reflow）与重绘（Repaint）

**重排**（也称回流）发生在元素的几何属性（位置、尺寸、边距等）发生变化时。浏览器需要重新计算元素在视口中的布局。

**重绘**发生在元素的外观（颜色、背景、边框等）发生变化但不影响布局时。浏览器只需要重新绘制受影响的像素。

```javascript
// 触发重排的操作（高成本）
element.style.width = '200px';           // 尺寸变化
element.style.padding = '20px';          // 内边距变化
element.style.marginLeft = '10px';       // 外边距变化
element.style.borderWidth = '2px';        // 边框变化
element.offsetTop;                        // 获取布局属性
element.scrollTop;                         // 触发强制同步布局
element.clientHeight;                     // 触发强制同步布局
element.getBoundingClientRect();           // 触发强制同步布局

// 仅触发重绘的操作（中等成本）
element.style.backgroundColor = 'red';    // 背景色变化
element.style.color = 'blue';             // 文字颜色变化
element.style.borderStyle = 'solid';      // 边框样式变化
element.style.boxShadow = '0 2px 4px rgba(0,0,0,0.1)'; // 阴影变化
```

> [!WARNING] 强制同步布局（Forced Synchronous Layout）
> 当 JavaScript 读取布局属性时会触发强制同步布局，这是性能杀手：
> 
> ```javascript
> // 反模式：强制同步布局在循环中
> function badExample() {
>   const items = document.querySelectorAll('.item');
>   for (let i = 0; i < items.length; i++) {
>     items[i].style.width = items[i].offsetWidth + 10 + 'px'; // 每次都触发重排！
>   }
> }
> 
> // 优化：批量读取，先统一读取再统一写入
> function goodExample() {
>   const items = document.querySelectorAll('.item');
>   const widths = Array.from(items).map(item => item.offsetWidth); // 批量读取
>   widths.forEach((width, i) => {
>     items[i].style.width = width + 10 + 'px'; // 批量写入
>   });
> }
> ```

### 1.3 浏览器优化机制

现代浏览器会自动批量处理多次 DOM 操作，但这不可靠。我们应该主动避免触发重排的代码模式：

```javascript
// 使用 CSS 类批量修改（推荐）
element.classList.add('active', 'expanded', 'highlighted');

// 使用 display: none 隐藏后修改
element.style.display = 'none';
// ... 大量 DOM 操作
element.style.display = 'block';

// 使用 documentFragment（批量 DOM 操作）
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}
container.appendChild(fragment); // 只触发一次重排

// 使用 requestAnimationFrame 同步到渲染帧
function animate() {
  element.style.transform = `translateX(${position}px)`;
  requestAnimationFrame(animate);
}
```

---

## 动画性能优化

CSS 动画是现代 UI 的核心，正确的动画实现能带来流畅的用户体验，错误的实现则会导致卡顿和电量消耗。

### 2.1 合成属性（Compositing Properties）

浏览器能够以极高效率处理两种 CSS 属性，因为它们可以在**独立的合成层**中运行，不需要触发重排或重绘：

| 属性 | 说明 | 性能等级 |
|------|------|----------|
| `transform` | 2D/3D 变换（translate、scale、rotate） | **最优** |
| `opacity` | 透明度 | **最优** |
| `filter` | 滤镜效果（部分） | 中等 |
| `clip-path` | 裁剪路径 | 中等 |

```css
/* 高性能动画：只改变 transform 和 opacity */
@keyframes slideIn {
  from {
    transform: translateX(-100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

@keyframes scaleIn {
  from {
    transform: scale(0.8);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* 使用 */
.performance-animation {
  animation: slideIn 0.3s ease-out;
}
```

> [!DANGER] 低性能动画模式
> 以下属性会触发重排或重绘，应避免用于动画：
> 
> ```css
> /* 差：触发重排 */
> @keyframes badAnimation {
>   from { width: 0; }
>   to { width: 100%; }
> }
> 
> /* 差：触发重绘 */
> @keyframes badAnimation2 {
>   from { background-color: red; }
>   to { background-color: blue; }
> }
> 
> /* 好：使用 transform 替代 */
> @keyframes goodAnimation {
>   from { transform: scaleX(0); }
>   to { transform: scaleX(1); }
> }
> ```

### 2.2 will-change 属性

`will-change` 通知浏览器元素即将发生变换，允许浏览器提前创建合成层。这是动画优化的关键工具，但过度使用会导致内存问题。

```css
/* 预通知浏览器即将变化 */
.slider {
  will-change: transform;
}

.animated-element {
  will-change: transform, opacity;
}

/* 动画结束后移除 will-change */
.animated-element.animating {
  will-change: transform;
}

.animated-element {
  transition: transform 0.3s ease;
}
.animated-element:hover {
  transform: scale(1.1);
}
```

> [!CAUTION] will-change 使用原则
> - **时机**：在动画开始前 100ms 添加，在动画结束后移除
> - **范围**：只对需要动画的元素使用，不要对父容器使用
> - **数量**：同一页面避免超过 20 个合成层
> - **内存**：每个合成层消耗额外内存，过多会导致崩溃

### 2.3 GPU 加速原理

现代浏览器的合成层运行在 GPU 上，GPU 加速的实现方式：

```css
/* 强制创建合成层的方法 */
.accelerated {
  /* 方法 1：3D 变换 */
  transform: translateZ(0);
  /* 或 */
  transform: translate3d(0, 0, 0);
  
  /* 方法 2：will-change */
  will-change: transform;
  
  /* 方法 3：position: fixed（特殊处理）*/
}

/* 应用示例：平滑滚动效果 */
.list-item {
  will-change: transform;
  transition: transform 0.2s ease;
}

.list-item:hover {
  transform: translateY(-2px);
}

/* 应用示例：卡片翻转 */
.flip-card {
  perspective: 1000px;
}

.flip-card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.6s;
  transform-style: preserve-3d;
}

.flip-card.flipped .flip-card-inner {
  transform: rotateY(180deg);
}
```

### 2.4 FLIP 动画技术

FLIP（First、Last、Invert、Play）是一种优化复杂动画的技术，通过将计算从 JavaScript 转移到 CSS 来提高性能：

```javascript
// FLIP 技术示例：列表项位置交换动画
function animateReorder(listElement) {
  // First: 记录初始位置
  const firstRect = listElement.getBoundingClientRect();
  
  // 执行 DOM 变更
  updateListOrder();
  
  // Last: 记录最终位置
  const lastRect = listElement.getBoundingClientRect();
  
  // Invert: 计算偏移并应用（CSS 处理）
  const deltaX = firstRect.left - lastRect.left;
  const deltaY = firstRect.top - lastRect.top;
  
  listElement.style.transform = `translate(${deltaX}px, ${deltaY}px)`;
  
  // 强制重绘
  listElement.offsetHeight; // 触发重绘以应用 transform
  
  // Play: 移除变换，让浏览器完成动画
  listElement.style.transition = 'transform 0.3s ease-out';
  listElement.style.transform = 'translate(0, 0)';
  
  // 清理
  listElement.addEventListener('transitionend', () => {
    listElement.style.transition = '';
    listElement.style.transform = '';
  }, { once: true });
}
```

```css
/* 配合的 CSS */
.animate-item {
  transition: transform 0.3s ease-out;
  will-change: transform;
}
```

---

## 3. CSS 选择器性能

选择器性能影响浏览器构建 Render Tree 的效率。虽然现代浏览器优化了选择器匹配，但不当的选择器仍可能成为性能瓶颈。

### 3.1 选择器匹配复杂度

选择器从右到左匹配，浏览器首先找到匹配的元素，然后向上遍历 DOM 验证父元素。

```css
/* 高效选择器：低特异性，匹配路径短 */
.button { }
#submit-btn { }
.header .nav-link { }

/* 低效选择器：过度嵌套 */
html body div.container section.article div.content p.intro { }

/* 低效选择器：通用选择器开销大 */
.container * { }
.header > * { }
```

### 3.2 选择器性能排名

| 排名 | 选择器类型 | 性能 | 示例 |
|------|-----------|------|------|
| 1 | ID 选择器 | 最快 | `#header` |
| 2 | 类选择器 | 快 | `.button` |
| 3 | 标签选择器 | 快 | `div` |
| 4 | 属性选择器 | 中等 | `[type="text"]` |
| 5 | 伪类选择器 | 中等 | `:hover` |
| 6 | 后代选择器 | 较慢 | `.parent .child` |
| 7 | 兄弟选择器 | 较慢 | `.sibling + .target` |
| 8 | 通用选择器 | 慢 | `*` |
| 9 | 复合选择器链 | 慢 | `.a.b.c` |

### 3.3 降低特异性策略

高特异性选择器会导致样式难以覆盖和维护：

```css
/* 问题：高特异性 */
#header .nav ul li a.link { }  /* 特异性: 0,1,4 */

/* 解决 1：使用 BEM 命名 */
.nav__link { }  /* 特异性: 0,1,0 */

/* 解决 2：提升选择器而非增加特异性 */
.nav a.link { }  /* 特异性: 0,2,0 */
[data-nav] a.link { }  /* 特异性: 0,2,1 */

/* 解决 3：使用 CSS 自定义属性覆盖 */
.nav-link {
  color: var(--link-color);  /* 低特异性，可轻松覆盖 */
}

/* 解决 4：组合类名 */
.link-primary { }  /* 可与 .nav-link 组合使用 */
.nav-link.link-primary { }  /* 特异性: 0,2,0 */
```

### 3.4 避免过深嵌套

```css
/* 不推荐：4 层以上嵌套 */
.article-body .content .paragraph .highlight {
  background-color: yellow;
}

/* 推荐：保持 3 层以内 */
.article-highlight {
  background-color: yellow;
}

/* 或使用单一类名配合 BEM */
.article__highlight {
  background-color: yellow;
}
```

> [!TIP] CSS 嵌套层数建议
> - CSS 预处理器的嵌套（`&`）不直接影响选择器特异性
> - 建议嵌套不超过 3 层，以提高可读性和性能
> - 使用类名替代嵌套来降低特异性

---

## 4. 关键渲染路径优化

关键渲染路径（Critical Rendering Path）是指浏览器将 HTML、CSS 和 JavaScript 转换为屏幕上像素所经历的关键步骤序列。

### 4.1 关键 CSS 识别与内联

首屏渲染需要的 CSS 应尽快加载：

```html
<!-- 内联关键 CSS -->
<style>
  /* 首屏必需的样式 */
  .header { height: 60px; background: #fff; }
  .hero { min-height: 400px; }
  .critical-path { font-size: 16px; }
</style>

<!-- 延迟加载非关键 CSS -->
<link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles.css"></noscript>
```

### 4.2 使用 `content-visibility` 优化长列表

`content-visibility` 是现代 CSS 的性能利器，允许浏览器跳过屏幕外内容的渲染：

```css
/* 跳过离屏内容的渲染 */
.list-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 100px; /* 估计高度 */
}

/* 可滚动容器优化 */
.scrollable-container {
  content-visibility: auto;
  contain-intrinsic-size: 0 800px;
  overflow-y: auto;
}

/* 保持可见但延迟渲染 */
.offscreen {
  content-visibility: hidden;
}
```

> [!NOTE] content-visibility 兼容性
> 现代浏览器（Chrome 85+、Firefox 121+、Safari 18+）已支持，但旧版浏览器会优雅降级。

### 4.3 CSS 加载策略

```html
<!-- 1. 使用 preload 预加载关键 CSS -->
<link rel="preload" href="critical.css" as="style" onload="this.rel='stylesheet'">

<!-- 2. 使用 media 属性条件加载 -->
<link rel="stylesheet" href="print.css" media="print">

<!-- 3. 避免阻塞渲染的样式表 -->
<link rel="stylesheet" href="async.css" media="print" onload="this.media='all'">

<!-- 4. 使用 fetchpriority 标记优先级 -->
<link rel="stylesheet" href="critical.css" fetchpriority="high">
```

### 4.4 渲染阻塞 JavaScript 避免

```html
<!-- 反模式：阻塞渲染 -->
<script src="app.js"></script>

<!-- 推荐：异步加载 -->
<script async src="app.js"></script>

<!-- 推荐：延迟执行 -->
<script defer src="app.js"></script>

<!-- 差异说明 -->
<!-- async: 下载不阻塞，执行会阻塞，顺序不保证 -->
<!-- defer: 下载不阻塞，执行在 DOM 解析后，顺序保证 -->
```

---

## 5. 浏览器兼容性策略

### 5.1 @supports 特性查询

使用 `@supports` 检测浏览器对 CSS 特性的支持：

```css
/* 基本用法 */
@supports (display: grid) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}

/* 带 fallback 的写法 */
.container {
  display: flex;
  flex-wrap: wrap;
}

@supports (display: grid) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
  }
}

/* 复合条件 */
@supports (display: grid) and (container-type: inline-size) {
  .card {
    container-type: inline-size;
  }
}

/* 或条件 */
@supports (backdrop-filter: blur(10px)) or (-webkit-backdrop-filter: blur(10px)) {
  .glass {
    backdrop-filter: blur(10px);
    -webkit-backdrop-filter: blur(10px);
  }
}

/* 非条件 */
@supports not (aspect-ratio: 1) {
  .video-wrapper {
    position: relative;
    padding-top: 56.25%; /* 16:9 比例 */
  }
  .video-wrapper iframe {
    position: absolute;
    top: 0;
    left: 0;
  }
}
```

### 5.2 JavaScript 特性检测

```javascript
// 检测 CSS 特性支持
function supportsCSS(property, value) {
  const support = CSS.supports(property, value);
  return support;
}

// 使用示例
const supportsGrid = CSS.supports('display', 'grid');
const supportsAspectRatio = CSS.supports('aspect-ratio', '1');
const supportsContentVisibility = CSS.supports('content-visibility', 'auto');

// 渐进增强的 JavaScript 实现
function initComponent() {
  if (CSS.supports('selector(:has(.child))')) {
    // 使用 :has() 选择器
    document.querySelector('.parent:has(.active)').classList.add('highlighted');
  } else {
    // Fallback 实现
    document.querySelectorAll('.parent').forEach(parent => {
      if (parent.querySelector('.active')) {
        parent.classList.add('highlighted');
      }
    });
  }
}
```

### 5.3 caniuse 使用指南

caniuse.com 是查询 CSS/JS 特性浏览器支持情况的核心工具：

```javascript
// caniuse-api (npm 包)
import { support } from 'caniuse-api';

const gridSupport = support('css-grid');
console.log(gridSupport); // 返回各浏览器支持情况

const features = ['css-grid', 'css-subgrid', 'css-container-queries'];
features.forEach(feat => {
  const info = support(feat);
  const globalUsage = info.global || 0;
  console.log(`${feat}: ${globalUsage}% 全球使用率`);
});
```

### 5.4 渐进增强与优雅降级

```css
/* 渐进增强：先实现基础功能，再增强 */
.card {
  /* 基础样式 - 所有浏览器 */
  padding: 1rem;
  background: white;
  border: 1px solid #ddd;
}

@supports (backdrop-filter: blur(10px)) {
  .card {
    background: rgba(255, 255, 255, 0.8);
    backdrop-filter: blur(10px);
  }
}

@supports (selector(:has(.badge))) {
  /* 更现代的选择器 */
  .card:has(.badge) {
    border-color: blue;
  }
}

/* 容器查询的渐进增强 */
.card-wrapper {
  /* 基础布局 */
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

@supports (container-type: inline-size) {
  .card-wrapper {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .card-wrapper {
      flex-direction: row;
    }
  }
}
```

---

## 6. 性能检测工具

### 6.1 Lighthouse

Lighthouse 是 Chrome DevTools 内置的性能审计工具：

```bash
# CLI 方式运行
npx lighthouse https://example.com \
  --output=html \
  --output-path=./lighthouse-report.html \
  --only-categories=performance

# 详细配置
npx lighthouse https://example.com \
  --preset=desktop \
  --throttling-method=simulate \
  --form-factor=desktop \
  --screenEmulation.disabled
```

Lighthouse 性能指标解读：

| 指标 | 含义 | 目标值 |
|------|------|--------|
| **FCP** (First Contentful Paint) | 首个内容绘制 | < 1.8s |
| **LCP** (Largest Contentful Paint) | 最大内容绘制 | < 2.5s |
| **TBT** (Total Blocking Time) | 总阻塞时间 | < 200ms |
| **CLS** (Cumulative Layout Shift) | 累计布局偏移 | < 0.1 |
| **SI** (Speed Index) | 速度指数 | < 3.4s |

### 6.2 Chrome DevTools Performance 面板

```javascript
// 使用 Performance API 手动标记
performance.mark('动画开始');
animateElement();
performance.mark('动画结束');
performance.measure('动画耗时', '动画开始', '动画结束');

// 使用 User Timing API 测量关键操作
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.name}: ${entry.duration}ms`);
  }
});
observer.observe({ entryTypes: ['measure', 'longtask'] });
```

DevTools Performance 面板分析要点：
1. 查找红色的 **Layout Shift** 事件
2. 检查 **Long Tasks**（超过 50ms 的任务）
3. 观察 **Frames** 图表，确保保持在 60fps 线上
4. 分析 **Layer** 图层数量，过多会消耗内存

### 6.3 CSS 性能调试清单

```css
/* 检查合成层数量 */
.layer {
  /* 在 DevTools 中查看层数 */
  will-change: transform;
}

/* 监控重排 */
.reflow-monitor {
  /* 使用 performance.getEntriesByType('layout-shift') */
  /* DevTools > Rendering > Paint flashing */
}

/* 动画性能检查 */
@keyframes anim {
  /* 使用 CSS Animations 替代 JavaScript 动画 */
  transform: translateX(100px);
}
```

### 6.4 常用性能测量 API

```javascript
// Element Timing API - 测量元素渲染时间
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`${entry.target.id} 渲染时间: ${entry.renderTime}`);
  }
});
observer.observe({ type: 'element', buffered: true });

// Paint Timing API - 测量绘制时间
const paintObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.name === 'first-contentful-paint') {
      console.log('FCP:', entry.startTime);
    }
  }
});
paintObserver.observe({ type: 'paint', buffered: true });

// Long Animation Frames API - 测量长动画帧
const frameObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(`长帧: ${entry.duration}ms`);
  }
});
frameObserver.observe({ type: 'long-animation-frame', buffered: true });

// Layout Instability API - 测量布局偏移
const layoutObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (!entry.hadRecentInput) {
      console.log(`布局偏移: ${entry.value}`);
    }
  }
});
layoutObserver.observe({ type: 'layout-shift', buffered: true });
```

---

## 高级动画模式

### 滚动驱动动画

现代 CSS 支持基于滚动位置的动画，无需 JavaScript：

```css
/* 滚动驱动动画 */
.scroll-animate {
  animation: fade-in linear;
  animation-timeline: scroll(root block);
  animation-range: 0% 50%;
}

@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 视差效果 */
.parallax-layer {
  transform: translateY(
    calc(scrollY * var(--parallax-speed, 0.5))
  );
}

/* 滚动进度指示器 */
.progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  height: 3px;
  background: var(--primary-color);
  transform-origin: left;
  scale: var(--scroll-progress, 0) 1;
}

/* 使用 view() 函数 */
@supports (view-transition-name: none) {
  .card {
    view-transition-name: card;
  }
}
```

### 复杂动画模式

```css
/* 序列动画 */
@keyframes sequence-animation {
  0% {
    opacity: 0;
    transform: translateX(-20px);
  }
  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

.sequence-item {
  animation: sequence-animation 0.5s ease-out forwards;
  opacity: 0;
}

.sequence-item:nth-child(1) { animation-delay: 0ms; }
.sequence-item:nth-child(2) { animation-delay: 100ms; }
.sequence-item:nth-child(3) { animation-delay: 200ms; }
.sequence-item:nth-child(4) { animation-delay: 300ms; }
.sequence-item:nth-child(5) { animation-delay: 400ms; }

/* 交错动画网格 */
.grid-animation {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.grid-animation > * {
  animation: fade-scale 0.6s ease-out forwards;
}

.grid-animation > *:nth-child(1) { animation-delay: 0ms; }
.grid-animation > *:nth-child(2) { animation-delay: 100ms; }
.grid-animation > *:nth-child(3) { animation-delay: 200ms; }
.grid-animation > *:nth-child(4) { animation-delay: 50ms; }
.grid-animation > *:nth-child(5) { animation-delay: 150ms; }
.grid-animation > *:nth-child(6) { animation-delay: 250ms; }

@keyframes fade-scale {
  from {
    opacity: 0;
    transform: scale(0.9);
  }
  to {
    opacity: 1;
    transform: scale(1);
  }
}
```

---

## 常见问题与解决方案

### 问题 1：首屏渲染缓慢

**症状**：页面加载后需要等待一段时间才开始渲染内容

**原因**：CSS 文件阻塞渲染

**解决方案**：

```html
<!-- 关键 CSS 内联 -->
<style>
  /* 首屏必需的关键样式 */
  .header { display: flex; align-items: center; }
  .hero { min-height: calc(100vh - 64px); }
  .critical-button { background: #3b82f6; color: white; }
</style>

<!-- 非关键 CSS 异步加载 -->
<link rel="preload" href="non-critical.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="non-critical.css"></noscript>
```

### 问题 2：动画掉帧

**症状**：动画不流畅，出现明显卡顿

**原因**：动画触发了重排或使用了低效的属性

**解决方案**：

```css
/* 差的做法 */
.bad-animation {
  animation: move 1s infinite;
}

@keyframes move {
  from { left: 0; }
  to { left: 100px; } /* 触发重排 */
}

/* 好的做法 */
.good-animation {
  will-change: transform;
  animation: move-transform 1s infinite;
}

@keyframes move-transform {
  from { transform: translateX(0); }
  to { transform: translateX(100px); } /* 仅触发合成 */
}
```

### 问题 3：长列表渲染慢

**症状**：包含大量列表项的页面滚动卡顿

**原因**：DOM 节点过多，样式计算开销大

**解决方案**：

```css
/* 使用 content-visibility */
.virtualized-list > * {
  content-visibility: auto;
  contain-intrinsic-size: 0 60px; /* 估计高度 */
}

/* 使用 contain 属性限制重排范围 */
.card {
  contain: layout style paint;
}

/* 懒加载区域 */
.lazy-section {
  content-visibility: hidden;
}

.lazy-section.visible {
  content-visibility: visible;
}
```

### 问题 4：样式冲突难以追踪

**症状**：样式覆盖问题难以定位，修改一个元素影响多个地方

**原因**：选择器特异性管理不当，缺乏样式隔离

**解决方案**：

```css
/* 使用 CSS Modules */
.component-name {
  /* 自动生成唯一类名 */
}

/* 使用 BEM 命名 */
.block__element--modifier {
  /* 清晰的层级结构 */
}

/* 使用 CSS 自定义属性作用域 */
.theme-container {
  --button-bg: blue;
  --button-color: white;
}

/* 特定区域覆盖 */
.special-section {
  --button-bg: green; /* 仅影响此区域 */
}
```

---

## 实战项目示例

### 项目 1：高性能轮播组件

**需求**：构建一个支持手势操作、动画流畅的轮播组件

```css
/* carousel.css */
.carousel {
  position: relative;
  overflow: hidden;
  width: 100%;
}

.carousel__track {
  display: flex;
  will-change: transform;
  transition: transform 0.5s cubic-bezier(0.25, 0.1, 0.25, 1);
  touch-action: pan-y;
}

.carousel__slide {
  flex: 0 0 100%;
  width: 100%;
  height: 400px;
  /* 懒加载优化 */
  content-visibility: auto;
  contain-intrinsic-size: 0 400px;
}

.carousel__slide img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* 指示器优化 */
.carousel__indicators {
  display: flex;
  gap: 8px;
  justify-content: center;
  padding: 16px 0;
}

.carousel__indicator {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--color-gray-300);
  transition: transform 0.2s ease;
}

.carousel__indicator--active {
  transform: scale(1.25);
  background: var(--color-primary);
}

/* 预加载相邻幻灯片 */
.carousel__slide--adjacent {
  content-visibility: visible;
}
```

### 项目 2：主题切换动画

**需求**：实现主题切换时的平滑过渡动画

```css
/* theme-transition.css */

/* 定义过渡变量 */
:root {
  --transition-colors: background-color 0.3s ease,
                      color 0.3s ease,
                      border-color 0.3s ease;
}

/* 主题切换基础样式 */
.theme-transition {
  transition: var(--transition-colors);
}

/* 复杂属性的平滑过渡 */
.card {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  box-shadow: var(--card-shadow);
  transition:
    background-color 0.3s ease,
    border-color 0.3s ease,
    box-shadow 0.3s ease;
}

/* 颜色变量自动插值 */
@property --gradient-angle {
  syntax: '<angle>';
  initial-value: 0deg;
  inherits: false;
}

.animated-gradient {
  background: conic-gradient(
    from var(--gradient-angle),
    var(--color-primary),
    var(--color-secondary),
    var(--color-primary)
  );
  animation: rotate-gradient 3s linear infinite;
}

@keyframes rotate-gradient {
  to {
    --gradient-angle: 360deg;
  }
}

/* 布局过渡 */
.expanding-panel {
  max-height: 0;
  overflow: hidden;
  transition: max-height 0.4s ease-out;
}

.expanding-panel--open {
  max-height: 1000px; /* 足够大的值 */
}

/* 使用 grid-template-rows 实现平滑展开 */
.grid-panel {
  display: grid;
  grid-template-rows: 0fr;
  transition: grid-template-rows 0.4s ease-out;
}

.grid-panel--open {
  grid-template-rows: 1fr;
}

.grid-panel__content {
  overflow: hidden;
}
```

---

## 性能检测工具

### Lighthouse CI 配置

```javascript
// lighthouserc.js
module.exports = {
  ci: {
    collect: {
      url: ['http://localhost:3000'],
      startServerCommand: 'npm run start',
      startServerReadyPattern: 'Server running',
      startServerReadyTimeout: 30000,
    },
    assert: {
      preset: 'lighthouse:recommended',
      assertions: {
        'first-contentful-paint': ['warn', { maxNumericValue: 2000 }],
        'largest-contentful-paint': ['error', { maxNumericValue: 4000 }],
        'cumulative-layout-shift': ['error', { maxNumericValue: 0.1 }],
        'total-blocking-time': ['error', { maxNumericValue: 500 }],
        'interactive': ['error', { maxNumericValue: 5000 }],
        'uses-optimized-css': 'warn',
        'unused-css-rules': 'warn',
      },
    },
    upload: {
      target: 'temporary-public-storage',
    },
  },
};
```

### 自动化性能监控

```javascript
// performance-monitor.js
class CSSPerformanceMonitor {
  constructor() {
    this.metrics = new Map();
    this.observer = null;
  }

  start() {
    // 监控长任务
    this.observer = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (entry.entryType === 'longtask') {
          console.warn('Long task detected:', entry.duration);
        }
      });
    });
    this.observer.observe({ entryTypes: ['longtask'] });

    // 监控布局偏移
    const layoutObserver = new PerformanceObserver((list) => {
      list.getEntries().forEach((entry) => {
        if (!entry.hadRecentInput) {
          console.warn('Layout shift:', entry.value);
        }
      });
    });
    layoutObserver.observe({ entryTypes: ['layout-shift'] });
  }

  stop() {
    this.observer?.disconnect();
  }

  getMetrics() {
    return {
      paint: this.getPaintMetrics(),
      layout: this.getLayoutMetrics(),
    };
  }

  getPaintMetrics() {
    const entries = performance.getEntriesByType('paint');
    return entries.reduce((acc, entry) => {
      acc[entry.name] = entry.startTime;
      return acc;
    }, {});
  }

  getLayoutMetrics() {
    const entries = performance.getEntriesByType('layout-shift');
    return {
      count: entries.length,
      total: entries.reduce((sum, e) => sum + e.value, 0),
    };
  }
}
```

---

## 总结

CSS 性能优化是一个系统性工程，需要从渲染原理出发，理解重排重绘机制，运用合成属性和 `will-change` 实现 GPU 加速，避免低效选择器，优化关键渲染路径，并合理使用兼容性策略。

核心优化原则：
- **优先使用 `transform` 和 `opacity`** 进行动画
- **避免触发布局变化** 的属性动画
- **合理使用 `will-change`** 但不过度
- **批量 DOM 操作** 减少重排次数
- **渐进增强** 确保兼容性
- **持续监控** 使用 DevTools 和 Lighthouse

### 性能优化清单

| 检查项 | 目标 | 检查方法 |
|--------|------|----------|
| FCP | < 1.8s | Lighthouse |
| LCP | < 2.5s | Lighthouse |
| CLS | < 0.1 | Lighthouse |
| TBT | < 200ms | Lighthouse |
| 动画帧率 | 60fps | DevTools |
| 选择器特异性 | < 0,2,0 | 人工审查 |
| CSS 体积 | < 50KB | 压缩后测量 |
| 关键 CSS | < 14KB | Lighthouse |

### 开发阶段检查

| 检查项 | 目标 | 工具 |
|--------|------|------|
| 选择器特异性 | < 0,2,0 | Stylelint |
| CSS 嵌套深度 | < 4 层 | Stylelint |
| 声明顺序 | 字母顺序 | Prettier |
| 注释完整性 | 公共组件有注释 | ESLint |
| 变量使用 | 使用 CSS 变量 | Stylelint |

### 构建阶段检查

| 检查项 | 目标 | 工具 |
|--------|------|------|
| 文件大小 | < 50KB gzip | Webpack |
| 选择器数量 | < 3000 | csstree-validator |
| 声明数量 | < 10000 | csstree-validator |
| 空行检查 | 无多余空行 | Prettier |

### 生产阶段检查

| 检查项 | 目标 | Lighthouse |
|--------|------|-----------|
| 未使用的 CSS | < 10% | Lighthouse |
| 关键 CSS | < 14KB | Lighthouse |
| 渲染阻塞 | 0 | Lighthouse |
| 样式计算 | < 50ms | DevTools |

---

## CSS 布局优化

### CSS Grid 性能优化

```css
/* 固定数量的列 */
.optimized-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 固定数量，更高效 */
  gap: 1rem;
}

/* 避免隐式网格创建 */
.explicit-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: repeat(3, auto); /* 显式定义 */
  grid-auto-flow: row; /* 明确流向 */
}

/* 使用 auto-fill vs auto-fit */
.auto-fill {
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  /* 保留空列 */
}

.auto-fit {
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  /* 空列塌陷 */
}

/* 减少重排的属性 */
.efficient-grid {
  display: grid;
  gap: 0; /* 减少 gap 变化 */
}

/* 内容感知布局 */
.content-aware {
  grid-template-columns: auto 1fr auto;
  /* 中间列填充剩余空间 */
}
```

### Flexbox 性能优化

```css
/* 固定大小而非内容感知 */
.optimized-flex {
  display: flex;
}

.flex-item {
  flex: 0 0 200px; /* 固定宽度 */
}

/* 避免 flex-basis 变化触发的重排 */
.fixed-basis {
  flex: 0 0 auto;
}

/* 使用 order 而非 DOM 重排 */
.reorder {
  display: flex;
}

.first { order: -1; }
.last { order: 1; }

/* 避免 flex-shrink 导致的性能问题 */
.stable {
  flex: 1 1 0%; /* 避免内容变化导致的重新计算 */
}
```

### 容器查询优化

```css
/* 容器查询基础 */
.card-container {
  container-type: inline-size;
  container-name: card;
}

.card {
  display: grid;
  grid-template-columns: 1fr;
}

@container card (min-width: 400px) {
  .card {
    grid-template-columns: auto 1fr;
  }
}

/* 容器查询组合 */
.complex-container {
  container-type: inline-size;
  container-name: layout;
}

@container layout (min-width: 600px) and (min-height: 400px) {
  .layout {
    display: grid;
    grid-template-columns: 200px 1fr;
  }
}
```

---

## 渲染优化技巧

### 减少样式计算

```css
/* 避免通配符选择器 */
.wildcard-slow * {
  color: red;
}

/* 使用具体选择器 */
.specific-fast {
  color: red;
}

/* 避免属性选择器在大量元素上 */
.attribute-slow [data-id="1"] {
  color: red;
}

/* 使用 class 选择器 */
.class-fast .data-item {
  color: red;
}

/* 减少 :not() 选择器使用 */
.not-slow:not(.hidden) {
  color: red;
}

/* 替代方案：显式设置 */
.visible {
  color: red;
}
```

### 减少绘制区域

```css
/* 使用 will-change 限制绘制区域 */
.paint-area {
  will-change: transform;
  contain: layout paint;
}

/* 分离高频更新元素 */
.animating-element {
  position: fixed; /* 独立于主文档流 */
  z-index: 1000;
}

/* 使用 transform 和 opacity */
.gpu-accelerated {
  transform: translateZ(0);
  will-change: opacity;
}

/* 避免 paint 的触发器 */
.no-paint {
  transform: translateX(0); /* 不触发重绘 */
}

.trigger-paint {
  left: 0; /* 触发重绘 */
}
```

### 层叠上下文优化

```css
/* 创建独立的合成层 */
.isolated-layer {
  will-change: transform;
  transform: translateZ(0);
}

/* 管理层叠顺序 */
.layer-1 { z-index: 1; }
.layer-2 { z-index: 2; }

/* 避免层叠上下文陷阱 */
.no-stacking-trap {
  isolation: isolate;
}

/* 透明的层叠上下文 */
.transparent-context {
  opacity: 0.99; /* 创建层叠上下文但不触发绘制 */
}
```

---

## 响应式设计性能

### 媒体查询优化

```css
/* 优先使用 Mobile First */
.mobile-style {
  /* 默认样式（移动端） */
}

@media (min-width: 768px) {
  .mobile-style {
    /* 平板样式 */
  }
}

@media (min-width: 1024px) {
  .mobile-style {
    /* 桌面样式 */
  }
}

/* 减少媒体查询数量 */
.bp-small { color: blue; }
.bp-medium { color: green; }
.bp-large { color: red; }

@media (min-width: 768px) {
  .bp-small { color: blue; } /* 保持不变 */
}

/* 使用容器查询替代媒体查询 */
.container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .content {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }
}
```

### 图片性能

```css
/* 响应式图片 */
.responsive-image {
  max-width: 100%;
  height: auto;
  content-visibility: auto;
}

/* 延迟加载图片 */
.lazy-image {
  content-visibility: hidden;
}

.lazy-image.loaded {
  content-visibility: visible;
}

/* 使用 srcset */
.picture-srcset {
  aspect-ratio: 16 / 9;
  background-color: #f0f0f0;
}

/* 预加载关键图片 */
.preloaded-image {
  content-visibility: hidden;
}
```

---

## 无障碍与性能平衡

### AOM 与性能平衡

```css
/* 无障碍焦点样式 */
.focus-visible:focus-visible {
  outline: 2px solid var(--focus-color);
  outline-offset: 2px;
}

/* 减少动画以满足 prefers-reduced-motion */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}

/* 高对比度模式优化 */
@media (prefers-contrast: high) {
  :root {
    --border-width: 2px;
    --outline-width: 3px;
  }

  .interactive {
    border: var(--border-width) solid currentColor;
  }
}
```

---

## 性能问题排查指南

### 问题诊断流程

```markdown
1. **识别问题类型**
   - 首次加载慢 → 关键渲染路径
   - 滚动卡顿 → 动画/合成层
   - 交互延迟 → 事件处理/重排

2. **使用 DevTools 分析**
   - Performance 面板：记录并分析帧率
   - Layers 面板：检查合成层数量
   - Rendering 面板：启用 FPS 计数器

3. **识别瓶颈**
   - 红色 Layout 事件 → 重排过多
   - 黄色 Paint 事件 → 绘制开销大
   - 紫色 Composite 事件 → 合成层问题

4. **实施优化**
   - 重排问题 → 批量操作、requestAnimationFrame
   - 重绘问题 → will-change、GPU 加速
   - 合成问题 → transform/opacity

5. **验证效果**
   - 再次记录 Performance
   - 对比优化前后的帧率
   - 检查 Lighthouse 评分
```

### 常见性能陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|---------|
| 大量合成层 | 内存占用高 | 减少 will-change 使用 |
| 频繁重排 | 滚动卡顿 | 批量读取、写入 |
| 复杂选择器 | 样式计算慢 | 简化选择器 |
| 大型样式表 | 加载慢 | 代码分割、压缩 |
| 阻塞脚本 | FCP 延迟 | async/defer |

---

> [!RELATED]
> - [[设计系统与组件化]] - 样式架构设计
> - [[现代CSS特性]] - CSS 新特性探索
> - [[Flexbox与Grid]] - 布局系统详解
> - [[CSS动画与运动设计]] - 动画设计专题

---

## CSS 预处理器性能优化

### SCSS/Less 编译优化

```scss
// 使用 @use 而非 @import 以提升编译速度
// @import 会导致全局作用域污染和重复编译
@use 'variables' as vars;
@use 'mixins' as mixins;
@use 'functions' as functions;

// 避免深层嵌套
// 差的实践：5 层以上嵌套
.nested {
  .level1 {
    .level2 {
      .level3 {
        .level4 {
          .level5 {
            color: red;
          }
        }
      }
    }
  }
}

// 好的实践：最多 3 层嵌套
.nested {
  .level1 {
    .content {
      .element {
        color: red;
      }
    }
  }
}

// 使用占位符选择器避免重复输出
%button-base {
  padding: 0.75rem 1.5rem;
  border-radius: 0.5rem;
  font-weight: 500;
  cursor: pointer;
}

.primary-button {
  @extend %button-base;
  background: blue;
  color: white;
}

.secondary-button {
  @extend %button-base;
  background: transparent;
  border: 1px solid blue;
  color: blue;
}

// 使用 @for 循环生成响应式类
@for $i from 1 through 12 {
  .col-#{$i} {
    width: calc(100% / 12 * #{$i});
  }
}

// 使用 @each 遍历列表生成图标类
$icons: search, home, user, settings, logout;

@each $icon in $icons {
  .icon-#{$icon} {
    background-image: url('/icons/#{$icon}.svg');
  }
}

// 避免使用 @mixin 大量参数化样式
// 好的实践：使用 CSS 自定义属性
:root {
  --button-padding-x: 1.5rem;
  --button-padding-y: 0.75rem;
}

.button {
  padding: var(--button-padding-y) var(--button-padding-x);
}
```

### PostCSS 优化配置

```javascript
// postcss.config.js
module.exports = {
  plugins: [
    // 1. CSS Nesting - 支持现代 CSS 嵌套
    'postcss-nesting',

    // 2. 自动前缀
    ['autoprefixer', {
      grid: true,
      flexbox: true,
      cascade: false
    }],

    // 3. 压缩和优化
    ['cssnano', {
      preset: ['default', {
        discardComments: { removeAll: true },
        normalizeWhitespace: true,
        minifySelectors: true,
        normalizeWhitespace: true,
        // 合并相同的规则
        mergeRules: true,
        // 合并媒体查询
        mediaQueries: true,
        // 优化 calc()
        calc: true,
        // 移除空规则
        discardEmpty: true
      }]
    }]
  ]
};
```

---

## 现代 CSS 性能特性

### CSS 包含属性（Containment）

```css
/* contain 属性隔离元素的渲染影响 */
.contained {
  contain: layout;
  /* layout: 元素的布局独立计算 */
}

.contained-paint {
  contain: paint;
  /* paint: 元素不会被绘制到视口外的内容影响 */
}

.contained-style {
  contain: style;
  /* style: 计数器等样式属性不会影响外部 */
}

.contained-content {
  contain: content;
  /* content: 相当于 layout paint，但不包含自身的后代 */
}

.contained-strict {
  contain: strict;
  /* strict: 相当于 content，但限制更严格 */
}

/* 实战应用：卡片列表 */
.card-list {
  contain: layout;
}

.card {
  contain: content;
  /* 卡片内容变化不会触发列表重排 */
}

/* 独立小部件 */
.widget {
  contain: layout style;
  /* 部件内部变化不会影响页面其他部分 */
}
```

### CSS 空间系统

```css
/* 使用逻辑属性实现更灵活的布局 */
.card {
  margin-block-start: 1rem;
  margin-block-end: 1rem;
  margin-inline-start: auto;
  margin-inline-end: auto;
  padding-block: 1.5rem;
  padding-inline: 1rem;
}

/* 逻辑边距值 */
.logical-margin {
  margin-inline: 2rem;  /* 等同于 margin-left + margin-right */
  margin-block: 1rem;    /* 等同于 margin-top + margin-bottom */
}

/* 逻辑边框 */
.logical-border {
  border-inline-start: 2px solid blue;
  border-inline-end: 2px solid blue;
  border-block-start: 1px solid gray;
  border-block-end: 1px solid gray;
}

/* 逻辑定位 */
.floating-element {
  inset-inline-end: 0;  /* 等同于 right: 0 */
  inset-block-start: 1rem;  /* 等同于 top: 1rem */
}
```

### 颜色空间优化

```css
/* OKLCH 色彩空间 - 更好的一致性和性能 */
:root {
  --primary: oklch(0.7 0.2 250);
  --secondary: oklch(0.6 0.15 180);
}

/* 使用 color-mix 实现颜色变体 */
.mixed {
  background: color-mix(in oklch, var(--primary) 80%, white);
}

.mixed-dark {
  background: color-mix(in oklch, var(--primary) 80%, black);
}

/* 相对颜色语法 */
.adaptive-button {
  background: var(--primary);
  color: oklch(from var(--primary) calc(l + 0.4) c h);
  /* 从 --primary 推导对比色 */
}

.hover-button {
  background: oklch(from var(--primary) calc(l + 0.1) c h);
  /* 悬停时增加亮度 */
}
```

---

## CSS 变量性能

### 变量的性能影响

```css
/* CSS 变量的性能特点 */

/* 读取 CSS 变量（性能友好） */
.fast-element {
  color: var(--text-color);
  /* 读取操作几乎无开销 */
}

/* 写入 CSS 变量（可能触发重排） */
:root {
  --theme-color: red;
}

.theme-change {
  --theme-color: blue;
  /* 只影响引用了该变量的元素，不触发全局重排 */
}

/* 变量的继承链 */
.parent {
  --local-var: 100px;
}

.child {
  width: var(--local-var);
  /* 子元素继承父元素的变量值 */
}

/* 使用 @layer 减少变量计算 */
@layer base, components;

@layer base {
  :root {
    --color: red;
  }
}

@layer components {
  .card {
    color: var(--color);
    /* 变量在 @layer 内定义，计算更高效 */
  }
}
```

### 变量与主题切换

```css
/* 高效的主题切换模式 */
:root {
  --bg-primary: #ffffff;
  --bg-secondary: #f8fafc;
  --text-primary: #0f172a;
  --text-secondary: #475569;
}

[data-theme="dark"] {
  --bg-primary: #0f172a;
  --bg-secondary: #1e293b;
  --text-primary: #f8fafc;
  --text-secondary: #cbd5e1;
}

.theme-element {
  background: var(--bg-primary);
  color: var(--text-primary);
  transition: background-color 0.3s, color 0.3s;
}

/* 使用自定义属性存储计算值 */
:root {
  --computed-spacing: calc(1rem * 2);
  --computed-size: min(100px, 50%);
}

/* 避免过度使用 CSS 变量 */
```

---

## 性能测试与监控

### 自动化性能测试

```javascript
// css-perf-test.js
import puppeteer from 'puppeteer';

class CSSPerformanceTest {
  constructor() {
    this.results = [];
  }

  async measureStyleCalculation(element) {
    const metrics = await element.evaluate(() => {
      const start = performance.now();

      // 强制样式计算
      getComputedStyle(element);

      const end = performance.now();
      return end - start;
    });

    return metrics;
  }

  async measureLayout(element) {
    const metrics = await element.evaluate(() => {
      const start = performance.now();

      // 强制布局
      element.getBoundingClientRect();

      const end = performance.now();
      return end - start;
    });

    return metrics;
  }

  async testSelectorPerformance(selectors) {
    const results = [];

    for (const selector of selectors) {
      const elements = document.querySelectorAll(selector);
      const count = elements.length;

      const start = performance.now();
      document.querySelectorAll(selector);
      const queryTime = performance.now() - start;

      results.push({
        selector,
        elementCount: count,
        queryTime
      });
    }

    return results;
  }

  async measureAnimation() {
    return new Promise(resolve => {
      const entries = [];

      const observer = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          entries.push({
            name: entry.name,
            duration: entry.duration,
            startTime: entry.startTime
          });
        });
      });

      observer.observe({ entryTypes: ['animation'] });

      // 等待动画完成
      setTimeout(() => {
        observer.disconnect();
        resolve(entries);
      }, 5000);
    });
  }
}

// 运行测试
const test = new CSSPerformanceTest();
const browser = await puppeteer.launch();
const page = await browser.newPage();

await page.goto('http://localhost:3000');
const element = await page.$('.test-element');

const styleTime = await test.measureStyleCalculation(element);
const layoutTime = await test.measureLayout(element);

console.log(`Style calculation: ${styleTime.toFixed(2)}ms`);
console.log(`Layout time: ${layoutTime.toFixed(2)}ms`);

await browser.close();
```

### 性能回归检测

```yaml
# .github/workflows/css-perf.yml
name: CSS Performance Regression

on:
  pull_request:
    paths:
      - 'src/**/*.css'
      - 'src/**/*.scss'

jobs:
  performance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4

      - name: Install dependencies
        run: npm ci

      - name: Build CSS
        run: npm run build:css

      - name: Measure bundle size
        run: |
          gzip -c dist/styles.css | wc -c > css-size.txt

      - name: Check CSS size
        run: |
          SIZE=$(cat css-size.txt)
          THRESHOLD=50000
          if [ $SIZE -gt $THRESHOLD ]; then
            echo "CSS bundle too large: $SIZE bytes"
            exit 1
          fi

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: http://localhost:3000
          budgetPath: ./lighthouse-budget.json
```

---

## 高级优化案例

### 虚拟列表优化

```css
/* 虚拟列表的 CSS 优化 */
.virtual-list {
  content-visibility: auto;
  contain-intrinsic-size: 0 50px; /* 估计每行高度 */
}

.virtual-list__item {
  /* 仅渲染可见区域的元素 */
  content-visibility: auto;
}

/* 分离滚动容器 */
.virtual-list__viewport {
  contain: strict;
  overflow-y: auto;
}

/* 列表项样式隔离 */
.virtual-list__item {
  contain: layout style;
}
```

### 复杂表单性能

```css
/* 表单输入优化 */
.form-input {
  /* 使用 contain 隔离重排 */
  contain: layout style;

  /* 避免触发布局的属性 */
  will-change: transform;
}

.form-input:focus {
  transform: scale(1.02);
  /* 使用 transform 而非 width/margin */
}

/* 表单验证样式 */
.form-input.invalid {
  border-color: var(--error-color);
  animation: shake 0.3s ease;
  /* 动画使用 GPU 加速 */
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-5px); }
  75% { transform: translateX(5px); }
}
```

### 图表渲染优化

```css
/* 图表容器优化 */
.chart-container {
  contain: strict;
  /* 图表区域完全隔离 */
}

.chart-svg {
  /* SVG 优化 */
  will-change: transform;
  contain: layout;
}

.chart-series {
  /* 数据系列 */
  transition: d 0.3s ease;
  /* 平滑过渡 */
}

/* Canvas 图表容器 */
.chart-canvas-wrapper {
  position: relative;
  contain: layout;
}

.chart-canvas {
  position: absolute;
  top: 0;
  left: 0;
  will-change: transform;
}
```

---

> [!SUCCESS]
> CSS 性能优化是一个持续的过程，需要在开发过程中不断关注和改进。通过遵循本文介绍的原则和技巧，结合自动化测试和监控，可以显著提升 Web 应用的性能和用户体验。
