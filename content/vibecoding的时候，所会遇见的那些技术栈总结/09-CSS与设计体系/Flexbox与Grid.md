# Flexbox 与 CSS Grid 布局完全指南

> [!NOTE]
> 本文档是 CSS 布局体系的核心专题，深入讲解 Flexbox 一维布局和 CSS Grid 二维布局的完整能力，涵盖实战模式、常见陷阱及性能考量。

---

## Flexbox 核心概念

Flexbox（弹性盒子布局）是目前最广泛使用的 CSS 布局模型，特别适合以下场景：
- 水平/垂直居中
- 导航栏分配
- 卡片网格自动换行
- 表单对齐

### Flex 容器属性

```css
.flex-container {
  /* 激活 flex 上下文 */
  display: flex;
  display: inline-flex;  /* 行内元素 flex */
  
  /* 主轴方向（flex-direction）*/
  flex-direction: row;             /* 水平，从左到右（默认）*/
  flex-direction: row-reverse;    /* 水平，从右到左 */
  flex-direction: column;         /* 垂直，从上到下 */
  flex-direction: column-reverse;  /* 垂直，从下到上 */
  
  /* 是否换行（flex-wrap）*/
  flex-wrap: nowrap;       /* 不换行，强制一行（默认）*/
  flex-wrap: wrap;         /* 换行，优先缩小 */
  flex-wrap: wrap-reverse; /* 换行，反向排列 */
  
  /* 主轴对齐（justify-content）*/
  justify-content: flex-start;     /* 左对齐（默认）*/
  justify-content: flex-end;       /* 右对齐 */
  justify-content: center;         /* 居中 */
  justify-content: space-between;   /* 两端对齐，中间均分 */
  justify-content: space-around;   /* 等宽环绕（左右margin相等）*/
  justify-content: space-evenly;   /* 完全等距（含两端）*/
  
  /* 交叉轴对齐（align-items）*/
  align-items: stretch;      /* 拉伸填满交叉轴（默认）*/
  align-items: flex-start;  /* 交叉轴起点对齐 */
  align-items: flex-end;    /* 交叉轴终点对齐 */
  align-items: center;      /* 交叉轴居中 */
  align-items: baseline;    /* 按基线对齐（文字对齐）*/
  
  /* 多行对齐（align-content，仅 wrap 模式生效）*/
  align-content: flex-start;    /* 顶部对齐 */
  align-content: flex-end;      /* 底部对齐 */
  align-content: center;        /* 居中对齐 */
  align-content: space-between;  /* 两端对齐 */
  align-content: space-around;   /* 等宽环绕 */
  align-content: stretch;        /* 拉伸填满（默认）*/
}
```

### Flex 子项属性

```css
.flex-item {
  /* 增长因子（flex-grow）*/
  flex-grow: 0;   /* 不增长（默认）*/
  flex-grow: 1;   /* 等比增长 */
  flex-grow: 2;   /* 增长量是其他项的2倍 */
  
  /* 收缩因子（flex-shrink）*/
  flex-shrink: 1;   /* 允许收缩（默认）*/
  flex-shrink: 0;   /* 不收缩，保持原尺寸 */
  
  /* 基准尺寸（flex-basis）*/
  flex-basis: auto;     /* 依据自身尺寸（默认）*/
  flex-basis: 200px;    /* 固定基准 */
  flex-basis: 50%;     /* 百分比基准 */
  
  /* 简写：flex: grow shrink basis */
  flex: 0 1 auto;    /* 默认值 */
  flex: 1;           /* flex: 1 1 0%，填满剩余空间 */
  flex: 0 0 200px;   /* 固定宽度，不伸缩 */
  flex: auto;        /* flex: 1 1 auto，响应式伸缩 */
  flex: none;        /* flex: 0 0 auto，不伸缩 */
  
  /* 单独控制交叉轴对齐（覆盖 align-items）*/
  align-self: auto;      /* 继承 align-items（默认）*/
  align-self: flex-start;
  align-self: center;
  align-self: flex-end;
  align-self: stretch;
  
  /* 改变顺序（order）*/
  order: 0;    /* 默认顺序 */
  order: -1;   /* 移到最前 */
  order: 1;    /* 移到最后 */
}
```

---

## Flexbox 实战布局模式

### 1. 水平垂直居中

```css
/* 方案1：Flexbox 居中（最常用）*/
.center-v1 {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 方案2：margin auto 在 flex 中居中 */
.center-v2 {
  display: flex;
  /* 子元素 */
}
.center-v2 .child {
  margin: auto;
}

/* 方案3：绝对定位 + transform */
.center-v3 {
  position: relative;
}
.center-v3 .child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

/* 方案4：grid 居中 */
.center-v4 {
  display: grid;
  place-items: center;
}
```

### 2. 导航栏

```css
/* 经典导航栏 */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 24px;
  height: 64px;
  background: white;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.nav-links {
  display: flex;
  gap: 32px;
}

.nav-links a {
  text-decoration: none;
  color: #475569;
  transition: color 200ms ease;
}

.nav-links a:hover {
  color: #0f172a;
}

/* 自适应导航栏 */
.auto-nav {
  display: flex;
  gap: 24px;
}

.nav-item {
  flex: 1;  /* 平分可用空间 */
  text-align: center;
}
```

### 3. 卡片网格

```css
.card-container {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
}

.card {
  /* 每行恰好3个，不够时自动换行 */
  flex: 0 1 calc((100% - 2 * 24px) / 3);
  min-width: 280px;  /* 防止过度缩小 */
}

/* 响应式 */
@media (max-width: 1024px) {
  .card {
    flex: 0 1 calc((100% - 24px) / 2);
  }
}

@media (max-width: 640px) {
  .card {
    flex: 1 1 100%;  /* 移动端单列 */
  }
}
```

### 4. 圣杯布局

```css
.holy-grail {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.holy-grail header,
.holy-grail footer {
  flex: 0 0 auto;  /* 固定高度 */
}

.holy-grail .main {
  display: flex;
  flex: 1;
  min-height: 0;
}

.holy-grail .sidebar-left,
.holy-grail .sidebar-right {
  flex: 0 0 200px;
}

.holy-grail .content {
  flex: 1;
  min-width: 0;  /* 重要：防止内容溢出 */
}

@media (max-width: 768px) {
  .holy-grail .main {
    flex-direction: column;
  }
  .holy-grail .sidebar-left,
  .holy-grail .sidebar-right {
    flex: 0 0 auto;
  }
}
```

### 5. 粘性 Footer

```css
/* 传统方案 */
.sticky-footer-v1 {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.sticky-footer-v1 main {
  flex: 1;
}

/* 现代方案：更简洁 */
.sticky-footer-v2 {
  display: flex;
  flex-direction: column;
  min-height: 100dvh;  /* 动态视口高度，移动端兼容 */
}
```

---

## CSS Grid 深入指南

### Grid 基础配置

```css
.grid-container {
  display: grid;
  
  /* 定义列 */
  grid-template-columns: 200px 1fr 200px;
  grid-template-columns: 100px 2fr 1fr;  /* 比例定义 */
  
  /* 使用 repeat() 简化 */
  grid-template-columns: repeat(4, 1fr);     /* 4等列 */
  grid-template-columns: repeat(3, minmax(200px, 1fr));
  
  /* auto-fill vs auto-fit */
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));  /* 填满换行 */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));    /* 自适应收缩 */
  
  /* 定义行 */
  grid-template-rows: auto 1fr auto;
  
  /* 间距 */
  gap: 24px;                /* 行列统一 */
  row-gap: 16px;            /* 单独设置行间距 */
  column-gap: 32px;         /* 单独设置列间距 */
}
```

### 网格定位

```css
.grid-item {
  /* 按网格线定位 */
  grid-column: 1 / 3;        /* 从第1线到第3线 */
  grid-column: 1 / span 2;   /* 从第1线跨2列 */
  grid-column: 1 / -1;       /* 跨越到最后 */
  
  grid-row: 2 / 4;            /* 从第2行线到第4行线 */
  
  /* 简写 */
  grid-area: 1 / 1 / 3 / 3;  /* row-start / col-start / row-end / col-end */
}

/* 命名区域 */
.page-layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
}

.header  { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main    { grid-area: main; }
.aside   { grid-area: aside; }
.footer  { grid-area: footer; }
```

### 命名网格线

```css
.named-lines {
  display: grid;
  grid-template-columns: 
    [full-start sidebar-start] 200px
    [sidebar-end content-start] 1fr
    [content-end] 200px [full-end];
}

/* 多命名 */
.multi-names {
  display: grid;
  grid-template-columns: 
    [col-start] 1fr 
    [col-end sidebar-start] 200px 
    [sidebar-end content-start] 1fr 
    [content-end];
}

.item {
  grid-column: col-start / col-end;  /* 使用命名线 */
}
```

### Subgrid（子网格对齐）

Subgrid 是 Grid 的关键能力，允许嵌套元素对齐父网格：

```css
.parent-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 24px;
}

.card {
  display: grid;
  grid-template-columns: subgrid;    /* 继承父列轨道 */
  grid-template-rows: subgrid;     /* 继承父行轨道 */
  grid-row: span 3;               /* 跨越父行 */
}

/* 实战场景：卡片列表内对齐 */
.card-header,
.card-body,
.card-footer {
  /* 自动对齐到同一列 */
}
```

### 隐式网格

```css
/* 超出定义范围的元素自动创建行/列 */
.implicit-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: 200px 200px;  /* 只定义前两行 */
  gap: 16px;
}

/* 设置隐式行高度 */
.implicit-grid {
  grid-auto-rows: minmax(100px, auto);  /* 隐式行最小100px */
  grid-auto-flow: row;                   /* 优先填充行 */
  grid-auto-flow: column;                 /* 优先填充列 */
}
```

---

## Grid 实战布局模式

### 1. 页面整体布局

```css
.page-layout {
  display: grid;
  grid-template-columns: 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header"
    "main"
    "footer";
  min-height: 100vh;
}

.page-layout > header { grid-area: header; }
.page-layout > main   { grid-area: main; }
.page-layout > footer { grid-area: footer; }

/* 带侧边栏的变体 */
.page-with-sidebar {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header  header"
    "sidebar main"
    "footer  footer";
  min-height: 100vh;
}

@media (max-width: 768px) {
  .page-with-sidebar {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "main"
      "footer";
  }
}
```

### 2. 栅格响应式卡片

```css
.auto-grid {
  display: grid;
  /* 自动填充，最小 280px */
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

.grid-item {
  /* 每个卡片内部也可以是 grid */
  display: grid;
  grid-template-rows: auto 1fr auto;
}
```

### 3. 内容宽度控制（写作排版）

```css
.reading-layout {
  display: grid;
  grid-template-columns: 
    [full-start] 1fr 
    [wide-start] 
    minmax(0, 3fr)
    [content-start] 
    min(65ch, 100%) 
    [content-end]
    minmax(0, 3fr)
    [wide-end] 
    1fr 
    [full-end];
  gap: 0;
}

/* 全宽图片 */
.full-bleed {
  grid-column: full-start / full-end;
}

/* 标准正文 */
.prose {
  grid-column: content-start / content-end;
}

/* 宽图表（超出正文但不越界）*/
.wide-figure {
  grid-column: wide-start / wide-end;
}
```

### 4. 瀑布流布局（Masonry）

```css
/* 原生 CSS Masonry（FireFox 支持，2026年覆盖率 ~75%）*/
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  grid-template-rows: masonry;  /* FireFox 专用 */
  gap: 16px;
}

/* 兼容方案：CSS Columns */
.masonry-columns {
  columns: 3 250px;  /* 最少3列，每列最小250px */
  column-gap: 16px;
}

.masonry-columns > .item {
  break-inside: avoid;
  margin-bottom: 16px;
}
```

---

## Flexbox vs Grid 决策树

```
需要布局什么？
│
├─ 单行/单列元素排列
│   └─ Flexbox ✓
│       - 导航栏
│       - 按钮组
│       - 列表项对齐
│       - 垂直居中
│
├─ 页面级整体布局
│   └─ CSS Grid ✓
│       - 经典两栏/三栏布局
│       - 命名区域布局
│       - 精确行列控制
│
├─ 卡片/组件网格
│   ├─ 固定列数 → CSS Grid ✓
│   └─ 自动换行且等宽 → Flexbox ✓
│
├─ 组件内部布局
│   └─ Flexbox ✓（更轻量）
│
└─ 复杂报告/仪表盘
    └─ CSS Grid ✓
```

---

## 常见陷阱与解决方案

### Flexbox 陷阱

```css
/* 陷阱1：flex 子项不收缩导致溢出 */
.bad-flex {
  display: flex;
}
.bad-flex .long-text {
  flex-shrink: 1;  /* 文字不会自动换行 */
  min-width: 0;    /* 解决方案：添加 */
}

/* 陷阱2：flex 容器高度不均 */
.bad-flex {
  align-items: stretch;  /* 默认拉伸 */
}
.bad-flex > * {
  /* 所有子项高度相同 */
}

/* 陷阱3：gap 与 margin 混用 */
.dont-mix {
  display: flex;
  gap: 16px;
  margin: 16px;  /* 不要混用 */
}

/* 陷阱4：flex-basis 与 width 冲突 */
.conflict {
  flex-basis: 200px;
  width: 50%;  /* basis 优先级更高 */
}
```

### Grid 陷阱

```css
/* 陷阱1：忘记 min-width: 0 */
.content-overflow {
  /* Grid 子项默认 min-width: auto，不会收缩 */
  min-width: 0;  /* 解决方案 */
}

/* 陷阱2：隐式网格无法控制 */
.implicit-rows {
  /* 未定义的行高度 */
  grid-auto-rows: minmax(100px, auto);  /* 解决方案 */
}

/* 陷阱3：grid-area 拼写错误 */
.wrong-area {
  grid-area: mainn;  /* 拼写错误不会报错 */
  /* 检查 template-areas 是否匹配 */
}

/* 陷阱4：z-index 上下文 */
.grid-item {
  position: relative;  /* 定位后 z-index 才生效 */
  z-index: 1;
}
```

---

> [!TIP]
> **黄金法则**：用 Flexbox 布局「流」（一维），用 Grid 布局「结构」（二维）。大多数 UI 组件用 Flexbox 就能解决；页面级布局用 Grid 更清晰。

---

## CSS Grid 高级特性

### Grid 模板区域与布局

```css
/* 模板区域布局 */
.page-layout {
  display: grid;
  grid-template-columns: 250px 1fr 250px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
  gap: 1px;
  background: #e5e7eb; /* gap 的颜色 */
}

.page-layout > * {
  background: white;
}

.header {
  grid-area: header;
  padding: 1rem;
}

.sidebar {
  grid-area: sidebar;
  padding: 1rem;
}

.main {
  grid-area: main;
  padding: 1rem;
}

.aside {
  grid-area: aside;
  padding: 1rem;
}

.footer {
  grid-area: footer;
  padding: 1rem;
}

/* 响应式变化 */
@media (max-width: 1024px) {
  .page-layout {
    grid-template-columns: 200px 1fr;
    grid-template-areas:
      "header header"
      "sidebar main"
      "footer footer";
  }
  
  .aside {
    display: none;
  }
}

@media (max-width: 768px) {
  .page-layout {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "main"
      "footer";
  }
  
  .sidebar {
    display: none;
  }
}
```

### Grid 间隙系统

```css
/* 基本间隙 */
.grid {
  display: grid;
  gap: 1rem;              /* 行列统一 */
  row-gap: 2rem;          /* 行间距 */
  column-gap: 3rem;        /* 列间距 */
}

/* 使用 CSS 变量创建间距系统 */
:root {
  --grid-gap-xs: 0.25rem;
  --grid-gap-sm: 0.5rem;
  --grid-gap-md: 1rem;
  --grid-gap-lg: 1.5rem;
  --grid-gap-xl: 2rem;
}

.grid-responsive {
  display: grid;
  gap: var(--grid-gap-md);
}

/* 响应式间隙 */
@media (min-width: 768px) {
  .grid-responsive {
    gap: var(--grid-gap-lg);
  }
}

@media (min-width: 1024px) {
  .grid-responsive {
    gap: var(--grid-gap-xl);
  }
}
```

### Grid 对齐详解

```css
/* 容器对齐 */
.grid-container {
  display: grid;
  
  /* 整个网格在容器中的对齐 */
  justify-content: start;      /* 左对齐 */
  justify-content: center;      /* 居中 */
  justify-content: end;         /* 右对齐 */
  justify-content: stretch;      /* 拉伸（默认）*/
  justify-content: space-between;
  justify-content: space-around;
  justify-content: space-evenly;
  
  align-content: start;         /* 顶部对齐 */
  align-content: center;        /* 居中 */
  align-content: end;           /* 底部对齐 */
  align-content: stretch;        /* 拉伸（默认）*/
  align-content: space-between;
  align-content: space-around;
  align-content: space-evenly;
  
  /* 简写 */
  place-content: center;        /* align-content justify-content */
  place-content: start end;
}

/* 项目对齐 */
.grid-item {
  /* 单个项目在单元格中的对齐 */
  justify-self: start;          /* 左对齐 */
  justify-self: center;         /* 居中 */
  justify-self: end;            /* 右对齐 */
  justify-self: stretch;         /* 拉伸（默认）*/
  
  align-self: start;             /* 顶部对齐 */
  align-self: center;           /* 居中 */
  align-self: end;              /* 底部对齐 */
  align-self: stretch;          /* 拉伸（默认）*/
  
  /* 简写 */
  place-self: center;           /* align-self justify-self */
  place-self: start end;
}

/* align-items vs justify-items */
/* align-items: 交叉轴对齐所有项目 */
/* justify-items: 主轴对齐所有项目 */
/* place-items: 两者简写 */
.grid-container {
  align-items: center;           /* 垂直居中 */
  justify-items: stretch;        /* 水平拉伸 */
  place-items: center stretch;   /* 简写 */
}
```

---

## Flexbox 高级模式

### Flex 容器与方向

```css
/* Flex 容器 */
.flex {
  display: flex;
  display: inline-flex;         /* 行内-flex */
}

/* Flex 方向 */
.flex-row {
  flex-direction: row;               /* 左到右（默认）*/
  flex-direction: row-reverse;       /* 右到左 */
  flex-direction: column;             /* 上到下 */
  flex-direction: column-reverse;   /* 下到上 */
}

/* Flex 换行 */
.flex-wrap {
  flex-wrap: nowrap;              /* 不换行（默认）*/
  flex-wrap: wrap;                /* 换行 */
  flex-wrap: wrap-reverse;       /* 反向换行 */
}

/* Flex 流动（方向 + 换行）*/
.flex-flow {
  flex-flow: row nowrap;          /* 默认 */
  flex-flow: column wrap;
  flex-flow: row-reverse wrap-reverse;
}
```

### Flex 对齐详解

```css
/* 主轴对齐 (justify-content) */
.main-axis {
  justify-content: flex-start;      /* 起点对齐（默认）*/
  justify-content: flex-end;          /* 终点对齐 */
  justify-content: center;           /* 居中 */
  justify-content: space-between;   /* 首尾贴边，中间等分 */
  justify-content: space-around;     /* 两端间距为中间一半 */
  justify-content: space-evenly;     /* 完全等分 */
}

/* 交叉轴对齐 (align-items) */
.cross-axis {
  align-items: stretch;              /* 拉伸填满（默认）*/
  align-items: flex-start;           /* 起点对齐 */
  align-items: flex-end;             /* 终点对齐 */
  align-items: center;               /* 居中 */
  align-items: baseline;             /* 按文字基线对齐 */
}

/* 多行对齐 (align-content) */
.multi-line {
  /* 仅在 flex-wrap: wrap 时生效 */
  align-content: flex-start;         /* 行组贴近起点 */
  align-content: flex-end;           /* 行组贴近终点 */
  align-content: center;             /* 行组居中 */
  align-content: space-between;      /* 首尾行贴近边，中间行等分 */
  align-content: space-around;       /* 类比 justify-content */
  align-content: space-evenly;       /* 完全等分 */
  align-content: stretch;            /* 拉伸填满（默认）*/
}

/* place-items 和 place-content */
.place-align {
  /* place-items: align-items justify-items */
  place-items: center;               /* 双向居中 */
  place-items: start stretch;
  
  /* place-content: align-content justify-content */
  place-content: center space-between;
  
  /* place-self: align-self justify-self */
  place-self: center;
}
```

### Flex 项目属性详解

```css
/* flex-grow: 增长因子 */
.grow {
  flex-grow: 0;                     /* 不增长（默认）*/
  flex-grow: 1;                     /* 等比增长 */
  flex-grow: 2;                     /* 增长量是其他项的2倍 */
}

/* flex-shrink: 收缩因子 */
.shrink {
  flex-shrink: 1;                   /* 允许收缩（默认）*/
  flex-shrink: 0;                   /* 不收缩 */
  flex-shrink: 0.5;                 /* 收缩能力减半 */
}

/* flex-basis: 基准尺寸 */
.basis {
  flex-basis: auto;                  /* 依据自身尺寸（默认）*/
  flex-basis: 100px;                /* 固定宽度 */
  flex-basis: 50%;                  /* 容器的一半 */
  flex-basis: min-content;          /* 最小内容宽度 */
  flex-basis: max-content;          /* 最大内容宽度 */
  flex-basis: fit-content(200px);  /* 适应内容，最多200px */
}

/* flex 简写 */
/* flex: flex-grow flex-shrink flex-basis */
.shorthand {
  flex: 1;                          /* flex: 1 1 0% */
  flex: 1 0;                        /* flex: 1 0 0% */
  flex: 1 200px;                   /* flex: 1 1 200px */
  flex: auto;                       /* flex: 1 1 auto */
  flex: none;                       /* flex: 0 0 auto */
  flex: initial;                    /* flex: 0 1 auto */
}

/* align-self: 单独覆盖 align-items */
.override {
  align-self: auto;                 /* 继承 align-items（默认）*/
  align-self: flex-start;
  align-self: flex-end;
  align-self: center;
  align-self: stretch;
  align-self: baseline;
}

/* order: 排列顺序 */
.order {
  order: 0;                        /* 默认顺序 */
  order: -1;                        /* 移到最前 */
  order: 1;                         /* 移到最后 */
  order: 999;                       /* 最最后 */
}
```

---

## 布局实战案例

### 案例 1：仪表盘布局

```css
/* 仪表盘主布局 */
.dashboard {
  display: grid;
  grid-template-columns: 260px 1fr;
  grid-template-rows: 64px 1fr;
  grid-template-areas:
    "header header"
    "sidebar main";
  height: 100vh;
  background: #f8fafc;
}

.dashboard-header {
  grid-area: header;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  background: white;
  border-bottom: 1px solid #e2e8f0;
}

.dashboard-sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  padding: 16px;
  background: white;
  border-right: 1px solid #e2e8f0;
  overflow-y: auto;
}

.dashboard-main {
  grid-area: main;
  padding: 24px;
  overflow-y: auto;
}

/* 指标卡片网格 */
.metrics-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  margin-bottom: 24px;
}

.metric-card {
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.metric-card__label {
  font-size: 14px;
  color: #64748b;
  margin-bottom: 8px;
}

.metric-card__value {
  font-size: 28px;
  font-weight: 700;
  color: #0f172a;
}

.metric-card__trend {
  display: flex;
  align-items: center;
  gap: 4px;
  margin-top: 8px;
  font-size: 13px;
}

.metric-card__trend--up {
  color: #22c55e;
}

.metric-card__trend--down {
  color: #ef4444;
}

/* 图表区域 */
.chart-section {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
  margin-bottom: 24px;
}

.chart-card {
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.chart-card__title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 16px;
}

/* 响应式 */
@media (max-width: 1024px) {
  .dashboard {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "main";
  }
  
  .dashboard-sidebar {
    display: none;
  }
  
  .chart-section {
    grid-template-columns: 1fr;
  }
}
```

### 案例 2：社交卡片布局

```css
/* 卡片网格 */
.card-feed {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 20px;
  padding: 20px;
}

/* 单张卡片 */
.social-card {
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgb(0 0 0 / 0.08);
  transition: transform 200ms ease, box-shadow 200ms ease;
}

.social-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgb(0 0 0 / 0.12);
}

.social-card__image {
  position: relative;
  aspect-ratio: 16 / 9;
  overflow: hidden;
}

.social-card__image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.social-card__badge {
  position: absolute;
  top: 12px;
  left: 12px;
  padding: 4px 12px;
  background: rgb(0 0 0 / 0.6);
  color: white;
  font-size: 12px;
  font-weight: 500;
  border-radius: 20px;
}

.social-card__content {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 16px;
}

.social-card__title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 8px;
  color: #0f172a;
}

.social-card__description {
  font-size: 14px;
  color: #64748b;
  line-height: 1.5;
  margin-bottom: 16px;
  flex: 1;
}

.social-card__meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 16px;
  border-top: 1px solid #e2e8f0;
}

.social-card__author {
  display: flex;
  align-items: center;
  gap: 8px;
}

.social-card__avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  object-fit: cover;
}

.social-card__name {
  font-size: 14px;
  font-weight: 500;
  color: #334155;
}

.social-card__date {
  font-size: 12px;
  color: #94a3b8;
}

.social-card__stats {
  display: flex;
  align-items: center;
  gap: 12px;
}

.social-card__stat {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 13px;
  color: #64748b;
}
```

### 案例 3：表单布局

```css
/* 表单容器 */
.form-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 32px;
}

/* 垂直表单 */
.form-vertical {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.form-field {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.form-label {
  font-size: 14px;
  font-weight: 500;
  color: #334155;
}

.form-input {
  padding: 12px 16px;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 200ms ease, box-shadow 200ms ease;
}

.form-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgb(59 130 246 / 0.2);
}

.form-hint {
  font-size: 13px;
  color: #64748b;
}

.form-error {
  font-size: 13px;
  color: #ef4444;
}

/* 内联表单 */
.form-inline {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  align-items: flex-end;
}

.form-inline .form-field {
  flex: 1 1 200px;
}

/* 双列表单 */
.form-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 16px;
}

@media (max-width: 640px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}

/* 按钮组 */
.form-actions {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
  margin-top: 24px;
}
```

### 案例 4：电商产品列表

```css
/* 产品网格 */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

/* 产品卡片 */
.product-card {
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  transition: box-shadow 300ms ease;
}

.product-card:hover {
  box-shadow: 0 12px 40px rgb(0 0 0 / 0.12);
}

.product-card__image {
  position: relative;
  aspect-ratio: 1;
  overflow: hidden;
}

.product-card__image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 500ms ease;
}

.product-card:hover .product-card__image img {
  transform: scale(1.05);
}

.product-card__badge {
  position: absolute;
  top: 12px;
  left: 12px;
  padding: 6px 12px;
  font-size: 12px;
  font-weight: 600;
  border-radius: 4px;
}

.product-card__badge--sale {
  background: #ef4444;
  color: white;
}

.product-card__badge--new {
  background: #3b82f6;
  color: white;
}

.product-card__badge--hot {
  background: #f59e0b;
  color: white;
}

.product-card__wishlist {
  position: absolute;
  top: 12px;
  right: 12px;
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
  border-radius: 50%;
  box-shadow: 0 2px 8px rgb(0 0 0 / 0.1);
  cursor: pointer;
  transition: transform 200ms ease;
}

.product-card__wishlist:hover {
  transform: scale(1.1);
}

.product-card__content {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 16px;
}

.product-card__category {
  font-size: 12px;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 4px;
}

.product-card__title {
  font-size: 16px;
  font-weight: 600;
  color: #0f172a;
  margin-bottom: 8px;
}

.product-card__rating {
  display: flex;
  align-items: center;
  gap: 4px;
  margin-bottom: 8px;
}

.product-card__rating-stars {
  display: flex;
  color: #f59e0b;
}

.product-card__rating-count {
  font-size: 13px;
  color: #64748b;
  margin-left: 4px;
}

.product-card__price {
  display: flex;
  align-items: baseline;
  gap: 8px;
  margin-top: auto;
}

.product-card__price-current {
  font-size: 20px;
  font-weight: 700;
  color: #0f172a;
}

.product-card__price-original {
  font-size: 14px;
  color: #94a3b8;
  text-decoration: line-through;
}
```

---

## Flexbox 与 Grid 性能优化

### CSS 动画性能

```css
/* 触发 GPU 加速的属性 */
.gpu-accelerated {
  /* 变换和透明度动画性能最好 */
  transform: translateX(0);
  opacity: 1;
  will-change: transform, opacity;
}

/* 动画示例 */
@keyframes slideIn {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.animate-slide {
  animation: slideIn 300ms ease-out;
}

/* 列表项交错动画 */
.list-item {
  animation: slideIn 300ms ease-out both;
}

.list-item:nth-child(1) { animation-delay: 0ms; }
.list-item:nth-child(2) { animation-delay: 50ms; }
.list-item:nth-child(3) { animation-delay: 100ms; }
.list-item:nth-child(4) { animation-delay: 150ms; }
.list-item:nth-child(5) { animation-delay: 200ms; }

/* 减少动画（无障碍）*/
@media (prefers-reduced-motion: reduce) {
  .animate-slide {
    animation: none;
  }
  
  .list-item {
    animation: none;
  }
}
```

### 布局性能技巧

```css
/* 避免布局抖动 */
.stable {
  /* 为动态内容预留空间 */
  min-height: 100px;
  min-width: 200px;
}

/* 使用 contain 隔离重排 */
.contained {
  contain: layout style;
  /* layout: 内部布局不影响外部 */
  /* style: 计数等样式计算隔离 */
}

/* 懒加载区域的占位符 */
.skeleton {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 16px;
}

.skeleton-item {
  aspect-ratio: 16 / 9;
  background: linear-gradient(
    90deg,
    #f0f0f0 0%,
    #e0e0e0 50%,
    #f0f0f0 100%
  );
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: 8px;
}

@keyframes shimmer {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

---

## 常见问题与解决方案

### Flexbox 常见问题

```css
/* 问题 1：flex 子项内容不换行 */
.no-wrap {
  display: flex;
  flex-wrap: nowrap;  /* 默认不换行 */
}

.no-wrap .item {
  flex-shrink: 1;
  min-width: 0;  /* 关键：允许收缩 */
  /* 或 */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 问题 2：flex 容器有意外高度 */
.unexpected-height {
  display: flex;
  /* 检查是否有默认的外边距 */
  /* 确保子元素没有不必要的 margin */
}

.unexpected-height > * {
  margin: 0;  /* 重置 */
}

/* 问题 3：justify-content 不生效 */
.check-wrap {
  display: flex;
  flex-wrap: wrap;
  /* justify-content 在不换行时效果明显 */
  /* 换行时需要 align-content */
  align-content: flex-start;
}

/* 问题 4：flex 子项不等高 */
.equal-height {
  display: flex;
  align-items: stretch;  /* 默认值，确保不变 */
}

.equal-height > * {
  /* 确保内容撑满 */
  display: flex;
  flex-direction: column;
}
```

### Grid 常见问题

```css
/* 问题 1：网格线编号混乱 */
.grid-lines {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* 
   * 列线编号：1 2 3 4
   * 行线编号：1 2 3 4
   */
}

.grid-item {
  grid-column: 1 / 3;  /* 从第1线到第3线，跨2列 */
  grid-row: 1 / 2;    /* 从第1线到第2线，跨1行 */
}

/* 问题 2：项目重叠 */
.overlap {
  display: grid;
}

.overlapping-item {
  grid-area: 1 / 1 / 3 / 3;  /* 跨越多个单元格 */
  z-index: 1;  /* 需要定位或 z-index */
}

/* 问题 3：隐式网格高度不确定 */
.implicit-rows {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: minmax(100px, auto);  /* 最小100px，最大自适应 */
}

/* 问题 4：fr 单位计算不准确 */
.fr-accuracy {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr;
  /* 确保容器有明确宽度 */
  width: 100%;
  /* 或 */
  max-width: 1200px;
  /* fr 是基于可用空间计算，不是容器宽度 */
}
```

---

## 参考资源

| 资源 | 链接 |
|------|------|
| CSS Grid 规范 | https://www.w3.org/TR/css-grid-1/ |
| Flexbox 规范 | https://www.w3.org/TR/css-flexbox-1/ |
| Grid Garden（游戏）| https://cssgridgarden.com/ |
| Flexbox Froggy（游戏）| https://flexboxfroggy.com/ |
| CSS-Tricks Grid 指南 | https://css-tricks.com/snippets/css/complete-guide-grid/ |
| CSS-Tricks Flexbox 指南 | https://css-tricks.com/snippets/css/a-guide-to-flexbox/ |

---

*本文档由 [[归愚知识系统]] 自动生成*

---

## Flexbox 高级应用

### Flexbox 底层原理

理解 Flexbox 的底层算法对于解决复杂布局问题至关重要：

```css
/* Flexbox 布局算法详解 */

/* 
 * Flex 布局分为两个阶段：
 * 1. 测量阶段（Measuring Phase）
 *    - 确定每个 flex item 的基础尺寸
 *    - 处理 flex-basis
 * 
 * 2. 分配阶段（分配剩余空间）
 *    - 处理 flex-grow（增长）
 *    - 处理 flex-shrink（收缩）
 */

/* 
 * 主轴排列算法：
 * 1. 计算所有 flex item 的 flex-basis
 * 2. 计算所有 item 的初始尺寸
 * 3. 计算主轴上的剩余空间
 * 4. 如果有 flex-grow > 0 的 item，按比例分配剩余空间
 * 5. 如果空间不足，按比例收缩 flex-shrink > 0 的 item
 */

/* 
 * 交叉轴排列算法：
 * 1. 计算每行的 cross size
 * 2. 确定 cross-axis 的 align-content（多行）或 align-items（单行）
 * 3. 对齐每个 item
 */

/* 关键属性详解 */

/* flex-basis 的优先级 */
.flex-item {
  /* 当同时指定 width 和 flex-basis 时 */
  width: 200px;          /* 值 A */
  flex-basis: 150px;     /* 值 B */
  
  /* 
   * 结果：flex-basis 生效（150px）
   * 因为 flex-basis 的优先级高于 width/height
   */
}

/* flex-shrink 的收缩算法 */
.flex-container {
  display: flex;
  width: 500px; /* 容器宽度 */
}

.flex-item {
  flex-shrink: 1; /* 收缩因子 */
  flex-basis: 200px;
}

/* 
 * 假设有 3 个 item，每个 200px
 * 总宽度 = 600px，容器 = 500px
 * 溢出 = 100px
 * 
 * 收缩量 = 溢出空间 × (item.shrink × item.base) / Σ(all items shrink × base)
 * 
 * item1: 100 × (1 × 200) / (200 + 200 + 200) = 33.33px
 * item2: 100 × (1 × 200) / 600 = 33.33px
 * item3: 100 × (1 × 200) / 600 = 33.33px
 */

/* flex-grow 的增长算法 */
.flex-container {
  display: flex;
  width: 600px;
}

.flex-item {
  flex-grow: 1;
  flex-basis: 100px;
}

/* 
 * 假设有 3 个 item，每个基础 100px
 * 总基础宽度 = 300px
 * 剩余空间 = 600 - 300 = 300px
 * 
 * 每个 item 增长量 = 剩余空间 × (item.grow / Σ all grow)
 * 每个 item 增长量 = 300 × (1 / 3) = 100px
 * 
 * 最终每个 item = 100 + 100 = 200px
 */
```

### Flexbox 高级布局模式

#### 1. 响应式导航栏

```css
/* 响应式导航栏 */
.navbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 16px;
  padding: 16px 24px;
  background: white;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.logo {
  flex-shrink: 0;
  font-weight: 700;
  font-size: 1.25rem;
}

.nav-links {
  display: flex;
  flex-wrap: wrap;
  gap: 8px 24px;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-link {
  padding: 8px 16px;
  color: #475569;
  text-decoration: none;
  border-radius: 6px;
  transition: all 200ms ease;
}

.nav-link:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.nav-spacer {
  /* 自动占据剩余空间 */
  flex: 1;
}

.actions {
  display: flex;
  gap: 8px;
}

/* 移动端适配 */
@media (max-width: 768px) {
  .navbar {
    flex-direction: column;
    align-items: stretch;
  }
  
  .nav-links {
    flex-direction: column;
    gap: 4px;
  }
  
  .nav-spacer {
    display: none;
  }
  
  .actions {
    justify-content: stretch;
  }
  
  .actions > * {
    flex: 1;
  }
}
```

#### 2. 卡片系统

```css
/* 响应式卡片容器 */
.card-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
}

/* 自适应卡片 */
.card {
  display: flex;
  flex-direction: column;
  
  /* 最小宽度，确保卡片不会太窄 */
  flex: 1 1 300px;
  
  /* 最大宽度，防止卡片过宽 */
  max-width: 400px;
  
  /* 卡片内容 */
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
  overflow: hidden;
}

.card-image {
  aspect-ratio: 16 / 9;
  object-fit: cover;
}

.card-content {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 20px;
}

.card-title {
  margin: 0 0 8px;
  font-size: 1.125rem;
  font-weight: 600;
}

.card-description {
  flex: 1;
  margin: 0 0 16px;
  color: #64748b;
  font-size: 0.875rem;
  line-height: 1.6;
}

.card-actions {
  display: flex;
  gap: 8px;
  margin-top: auto;
}

/* 固定宽度列 */
.card-grid-fixed {
  display: flex;
  gap: 16px;
  overflow-x: auto;
  padding-bottom: 16px;
  -webkit-overflow-scrolling: touch;
}

.card-fixed {
  flex: 0 0 280px; /* 固定宽度 */
  height: 360px;
}

/* 瀑布流卡片 */
.card-masonry {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.masonry-column {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.card-masonry .card {
  max-width: none; /* 移除最大宽度限制 */
}
```

#### 3. 表单布局

```css
/* 经典表单布局 */
.form {
  display: flex;
  flex-direction: column;
  gap: 20px;
  max-width: 500px;
}

.form-row {
  display: flex;
  gap: 16px;
}

.form-group {
  display: flex;
  flex-direction: column;
  flex: 1;
  gap: 6px;
}

.form-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: #374151;
}

.form-input {
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
  font-size: 1rem;
  transition: border-color 200ms ease;
}

.form-input:focus {
  outline: none;
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgb(59 130 246 / 0.1);
}

.form-error {
  font-size: 0.75rem;
  color: #dc2626;
}

.form-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  margin-top: 8px;
}

/* 响应式表单 */
@media (max-width: 480px) {
  .form-row {
    flex-direction: column;
    gap: 16px;
  }
  
  .form-actions {
    flex-direction: column-reverse;
  }
  
  .form-actions > * {
    width: 100%;
  }
}
```

#### 4. 聊天界面布局

```css
/* 聊天界面 */
.chat {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-height: 100vh;
}

.chat-header {
  flex: 0 0 auto;
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px 20px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
}

.chat-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

.chat-info {
  flex: 1;
}

.chat-name {
  margin: 0;
  font-weight: 600;
}

.chat-status {
  font-size: 0.75rem;
  color: #6b7280;
}

.chat-messages {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 20px;
  overflow-y: auto;
}

.message {
  display: flex;
  flex-direction: column;
  max-width: 70%;
}

.message.sent {
  align-self: flex-end;
}

.message.received {
  align-self: flex-start;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 16px;
  line-height: 1.5;
}

.message.sent .message-bubble {
  background: #3b82f6;
  color: white;
  border-bottom-right-radius: 4px;
}

.message.received .message-bubble {
  background: #f3f4f6;
  color: #1f2937;
  border-bottom-left-radius: 4px;
}

.message-time {
  font-size: 0.625rem;
  color: #9ca3af;
  margin-top: 4px;
}

.chat-input-area {
  flex: 0 0 auto;
  display: flex;
  gap: 12px;
  padding: 16px 20px;
  background: white;
  border-top: 1px solid #e5e7eb;
}

.chat-input {
  flex: 1;
  padding: 12px 16px;
  border: 1px solid #d1d5db;
  border-radius: 24px;
  resize: none;
  font-family: inherit;
}

.chat-send {
  flex: 0 0 auto;
  padding: 12px 24px;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 24px;
  font-weight: 600;
  cursor: pointer;
  transition: background 200ms ease;
}

.chat-send:hover {
  background: #2563eb;
}
```

### Flexbox 动画与过渡

```css
/* Flexbox 动画 */

/* 1. 展开/收起动画 */
.accordion {
  display: flex;
  flex-direction: column;
}

.accordion-item {
  overflow: hidden;
  transition: max-height 300ms ease;
}

.accordion-content {
  max-height: 0;
  overflow: hidden;
  transition: max-height 300ms ease;
}

.accordion-item.active .accordion-content {
  max-height: 500px; /* 或使用 JS 计算实际高度 */
}

/* 2. 排序动画 */
.sortable-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.sortable-item {
  display: flex;
  align-items: center;
  padding: 12px 16px;
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  cursor: grab;
  transition: transform 200ms ease, box-shadow 200ms ease;
}

.sortable-item:hover {
  box-shadow: 0 4px 12px rgb(0 0 0 / 0.1);
}

.sortable-item:active {
  cursor: grabbing;
  transform: scale(1.02);
}

.sortable-item.dragging {
  opacity: 0.5;
  transform: scale(1.05);
}

/* 3. 加载动画 */
.loading-flex {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  min-height: 200px;
}

.loading-dot {
  width: 10px;
  height: 10px;
  background: #3b82f6;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out both;
}

.loading-dot:nth-child(1) { animation-delay: -0.32s; }
.loading-dot:nth-child(2) { animation-delay: -0.16s; }

@keyframes bounce {
  0%, 80%, 100% {
    transform: scale(0);
  }
  40% {
    transform: scale(1);
  }
}

/* 4. 标签切换动画 */
.tab-list {
  display: flex;
  gap: 4px;
  border-bottom: 1px solid #e5e7eb;
}

.tab-item {
  padding: 12px 20px;
  background: transparent;
  border: none;
  cursor: pointer;
  position: relative;
  transition: color 200ms ease;
}

.tab-item::after {
  content: '';
  position: absolute;
  bottom: -1px;
  left: 0;
  width: 100%;
  height: 2px;
  background: #3b82f6;
  transform: scaleX(0);
  transition: transform 200ms ease;
}

.tab-item:hover {
  color: #3b82f6;
}

.tab-item.active {
  color: #3b82f6;
}

.tab-item.active::after {
  transform: scaleX(1);
}
```

---

## CSS Grid 高级应用

### Grid 底层算法

```css
/* Grid 布局算法详解 */

/*
 * Grid 布局分为以下阶段：
 * 1. 轨道尺寸定义阶段
 *    - 解析 grid-template-columns/rows
 *    - 计算轨道尺寸
 * 
 * 2. 项目定位阶段
 *    - 根据 grid-area/grid-column/grid-row 定位
 *    - 处理跨轨道项目
 * 
 * 3. 内容包裹阶段
 *    - 确定隐式网格轨道尺寸
 *    - auto-fill vs auto-fit 的区别
 */

/* fr 单位详解 */
.grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  /* 等价于 */
  grid-template-columns: repeat(3, 1fr);
  
  /* 可用空间 = 容器宽度 - 固定轨道 - gap */
  /* 每个 1fr = 可用空间 / 总 fr 数 */
}

/* min-content 和 max-content */
.content-grid {
  display: grid;
  grid-template-columns: min-content max-content auto;
  gap: 16px;
}

/* minmax() 函数 */
.minmax-grid {
  display: grid;
  grid-template-columns: minmax(200px, 1fr) minmax(150px, 300px);
  /* 第一列：最小 200px，最大 1fr */
  /* 第二列：最小 150px，最大 300px */
}

/* auto-fill vs auto-fit */
.auto-fill-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  /* 
   * auto-fill: 尽可能多地创建轨道，即使为空也保留空间
   * 适合：需要保持对齐的卡片网格
   */
}

.auto-fit-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  /*
   * auto-fit: 轨道会被压缩，没有内容的轨道宽度为 0
   * 适合：响应式内容区域
   */
}

/* 实际对比 */
.grid-container {
  width: 700px;
  gap: 16px;
}

.grid-container > * {
  height: 100px;
  background: #e5e7eb;
  border-radius: 4px;
}

/* 当只有 2 个 item 时：
 * auto-fill: 3 列（2 列有内容，1 列为空）
 * auto-fit: 2 列（没有空列）
 */
```

### Grid 高级布局模式

#### 1. 仪表盘布局

```css
/* 仪表盘/管理后台布局 */
.dashboard {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 60px 1fr 50px;
  grid-template-areas:
    "header header"
    "sidebar main"
    "sidebar footer";
  height: 100vh;
}

.dashboard-header {
  grid-area: header;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  background: white;
  border-bottom: 1px solid #e5e7eb;
}

.dashboard-sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
  background: #1f2937;
  color: white;
  overflow-y: auto;
}

.dashboard-main {
  grid-area: main;
  display: grid;
  grid-template-rows: auto 1fr;
  padding: 24px;
  background: #f9fafb;
  overflow-y: auto;
}

.dashboard-footer {
  grid-area: footer;
  display: flex;
  align-items: center;
  justify-content: center;
  background: white;
  border-top: 1px solid #e5e7eb;
  font-size: 0.875rem;
  color: #6b7280;
}

/* 仪表盘内容区 */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 24px;
  margin-bottom: 24px;
}

.stat-card {
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.stat-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 12px;
}

.stat-title {
  font-size: 0.875rem;
  font-weight: 500;
  color: #6b7280;
}

.stat-icon {
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-size: 1.25rem;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: #111827;
}

.stat-change {
  font-size: 0.875rem;
  margin-top: 8px;
}

.stat-change.positive {
  color: #059669;
}

.stat-change.negative {
  color: #dc2626;
}

/* 图表区域 */
.charts-grid {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 24px;
  flex: 1;
}

.chart-card {
  display: flex;
  flex-direction: column;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.chart-title {
  margin: 0 0 16px;
  font-size: 1rem;
  font-weight: 600;
}

.chart-content {
  flex: 1;
  min-height: 200px;
}

/* 移动端响应式 */
@media (max-width: 1024px) {
  .dashboard {
    grid-template-columns: 1fr;
    grid-template-rows: 60px 1fr 50px;
    grid-template-areas:
      "header"
      "main"
      "footer";
  }
  
  .dashboard-sidebar {
    display: none; /* 或使用汉堡菜单 */
  }
  
  .charts-grid {
    grid-template-columns: 1fr;
  }
}
```

#### 2. 照片墙布局

```css
/* 照片墙/画廊布局 */
.photo-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-auto-rows: 200px;
  gap: 8px;
}

/* 不同尺寸的照片 */
.photo {
  position: relative;
  overflow: hidden;
  border-radius: 8px;
}

.photo img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 300ms ease;
}

.photo:hover img {
  transform: scale(1.05);
}

/* 跨列跨行 */
.photo.tall {
  grid-row: span 2;
}

.photo.wide {
  grid-column: span 2;
}

.photo.large {
  grid-row: span 2;
  grid-column: span 2;
}

/* 悬停效果 */
.photo-overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background: rgb(0 0 0 / 0.5);
  color: white;
  opacity: 0;
  transition: opacity 200ms ease;
}

.photo:hover .photo-overlay {
  opacity: 1;
}

/* Masonry 布局（传统方案）*/
.masonry-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

.masonry-column {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.masonry-item {
  break-inside: avoid;
  border-radius: 8px;
  overflow: hidden;
}

.masonry-item img {
  width: 100%;
  height: auto;
  display: block;
}
```

#### 3. 栅格系统

```css
/* 基于 Grid 的栅格系统 */
.grid-system {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 24px;
}

/* 列宽类 */
.col-1  { grid-column: span 1; }
.col-2  { grid-column: span 2; }
.col-3  { grid-column: span 3; }
.col-4  { grid-column: span 4; }
.col-5  { grid-column: span 5; }
.col-6  { grid-column: span 6; }
.col-7  { grid-column: span 7; }
.col-8  { grid-column: span 8; }
.col-9  { grid-column: span 9; }
.col-10 { grid-column: span 10; }
.col-11 { grid-column: span 11; }
.col-12 { grid-column: span 12; }

/* 偏移类 */
.offset-1  { grid-column-start: 2; }
.offset-2  { grid-column-start: 3; }
.offset-3  { grid-column-start: 4; }
.offset-4  { grid-column-start: 5; }
.offset-6  { grid-column-start: 7; }
.offset-8  { grid-column-start: 9; }

/* 响应式断点 */
@media (max-width: 1024px) {
  .col-lg-6 { grid-column: span 6; }
  .col-lg-12 { grid-column: span 12; }
}

@media (max-width: 768px) {
  .col-md-6 { grid-column: span 6; }
  .col-md-12 { grid-column: span 12; }
}

@media (max-width: 480px) {
  .col-sm-12 { grid-column: span 12; }
}
```

#### 4. 响应式表格

```css
/* 响应式表格 */
.table-container {
  display: grid;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
}

.table-header,
.table-row {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr 1fr auto;
  gap: 16px;
  padding: 12px 16px;
  align-items: center;
}

.table-header {
  background: #f9fafb;
  font-weight: 600;
  font-size: 0.875rem;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.table-row {
  border-top: 1px solid #e5e7eb;
  background: white;
  transition: background 150ms ease;
}

.table-row:hover {
  background: #f9fafb;
}

.table-cell {
  display: flex;
  align-items: center;
  min-width: 0;
}

.table-cell.truncate {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 移动端卡片视图 */
@media (max-width: 768px) {
  .table-header {
    display: none;
  }
  
  .table-row {
    display: flex;
    flex-direction: column;
    gap: 8px;
    padding: 16px;
  }
  
  .table-cell {
    display: flex;
    justify-content: space-between;
  }
  
  .table-cell::before {
    content: attr(data-label);
    font-weight: 500;
    color: #6b7280;
  }
}
```

### Grid 与 Subgrid 高级应用

```css
/* Subgrid 深入应用 */

/* 1. 卡片网格对齐 */
.card-collection {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 24px;
}

.card {
  display: grid;
  /* 子网格继承父网格 */
  grid-template-columns: subgrid;
  grid-template-rows: subgrid;
  
  /* 卡片占据 3 行 */
  grid-row: span 3;
  
  /* 卡片内部自动对齐到父网格 */
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
  overflow: hidden;
}

.card-image {
  grid-column: 1 / -1;
  aspect-ratio: 16 / 9;
}

.card-content {
  grid-column: 1 / -1;
  padding: 20px;
}

.card-title {
  /* 与其他卡片标题自动对齐 */
}

.card-description {
  /* 与其他卡片描述自动对齐 */
}

.card-footer {
  grid-column: 1 / -1;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 20px;
  border-top: 1px solid #e5e7eb;
}

/* 2. 表单对齐 */
.form-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
}

.form-row {
  display: grid;
  grid-column: 1 / -1;
  grid-template-columns: subgrid;
  gap: 24px;
}

.form-field {
  display: grid;
  grid-template-columns: auto 1fr;
  gap: 8px;
  align-items: start;
}

.form-label {
  padding-top: 10px;
  font-weight: 500;
}

.form-input {
  padding: 10px 14px;
  border: 1px solid #d1d5db;
  border-radius: 6px;
}

/* 3. 列表项对齐 */
.list-grid {
  display: grid;
  grid-template-columns: auto 1fr auto auto;
  gap: 16px;
  align-items: center;
}

.list-item {
  display: grid;
  grid-column: 1 / -1;
  grid-template-columns: subgrid;
  padding: 16px;
  background: white;
  border-radius: 8px;
}

.list-avatar {
  grid-column: 1;
}

.list-content {
  grid-column: 2;
}

.list-name {
  font-weight: 600;
}

.list-email {
  color: #6b7280;
  font-size: 0.875rem;
}

.list-actions {
  grid-column: 3;
  display: flex;
  gap: 8px;
}

.list-status {
  grid-column: 4;
}
```

### Grid 动画效果

```css
/* Grid 动画 */

/* 1. 项目入场动画 */
.grid-with-animation {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.animated-item {
  opacity: 0;
  transform: translateY(20px);
  animation: fadeInUp 400ms ease forwards;
}

.animated-item:nth-child(1) { animation-delay: 0ms; }
.animated-item:nth-child(2) { animation-delay: 100ms; }
.animated-item:nth-child(3) { animation-delay: 200ms; }
.animated-item:nth-child(4) { animation-delay: 300ms; }
.animated-item:nth-child(5) { animation-delay: 400ms; }
.animated-item:nth-child(6) { animation-delay: 500ms; }

@keyframes fadeInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 2. 布局切换动画 */
.grid-layout-switcher {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  transition: grid-template-columns 300ms ease;
}

.grid-layout-switcher.list-view {
  grid-template-columns: 1fr;
}

.grid-layout-switcher.list-view .item {
  display: flex;
  align-items: center;
  gap: 16px;
}

/* 3. 排序动画 */
.sortable-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 16px;
}

.sortable-item {
  transition: transform 300ms ease, opacity 300ms ease;
}

.sortable-item.moving {
  opacity: 0.5;
  transform: scale(0.95);
}

.sortable-item.dropping {
  transform: scale(1.02);
}

/* 4. 交错动画 */
.staggered-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 24px;
}

.staggered-item {
  --delay: 0;
  animation: staggerFade 500ms ease forwards;
  animation-delay: calc(var(--index) * 80ms);
}

@keyframes staggerFade {
  from {
    opacity: 0;
    transform: translateY(30px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

/* 5. 网格扩展动画 */
.expanding-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 0;
  transition: gap 300ms ease;
}

.expanding-grid.expanded {
  gap: 24px;
}
```

---

## Flexbox 与 Grid 混合使用

### 最佳实践组合

```css
/* Flexbox 和 Grid 的黄金组合 */

/* 
 * 原则：
 * 1. 页面级布局用 Grid
 * 2. 组件内部布局用 Flexbox
 * 3. 需要精确控制的二维布局用 Grid
 * 4. 需要灵活流动的一维布局用 Flexbox
 */

/* 页面布局示例 */
.page {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 80px 1fr auto;
  min-height: 100vh;
}

/* Header 内部用 Flexbox */
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
}

.header-logo {
  flex-shrink: 0;
}

.header-nav {
  display: flex;
  gap: 8px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* Sidebar 内部用 Flexbox */
.sidebar {
  display: flex;
  flex-direction: column;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid #e5e7eb;
}

/* Main 区域用 Grid */
.main-content {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
  padding: 24px;
}

/* 卡片组件内部用 Flexbox */
.card {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.card-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.card-body {
  flex: 1;
}

.card-footer {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: auto;
}
```

### 常见布局模式

```css
/* 1. 三明治布局 */
.sandwich {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
}

.sandwich > header,
.sandwich > footer {
  flex: 0 0 auto;
}

.sandwich > main {
  flex: 1;
  display: grid; /* 内部再次使用 Grid */
  grid-template-columns: 1fr;
}

/* 2. holy-grail 布局（完整版）*/
.holy-grail {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: 60px 1fr 50px;
  grid-template-areas:
    "header header header"
    "nav main aside"
    "footer footer footer";
  min-height: 100vh;
}

.holy-grail header {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.holy-grail nav,
.holy-grail aside {
  display: flex;
  flex-direction: column;
}

/* 3. 双栏布局（简单版）*/
.two-columns {
  display: flex;
  gap: 32px;
}

.sidebar {
  width: 280px;
  flex-shrink: 0;
}

.content {
  flex: 1;
  min-width: 0;
}

/* 响应式双栏 */
@media (max-width: 768px) {
  .two-columns {
    flex-direction: column;
  }
  
  .sidebar {
    width: 100%;
  }
}

/* 4. 多栏内容布局 */
.multi-column-content {
  display: grid;
  grid-template-columns: 
    [full-start] 1fr 
    [wide-start] minmax(0, 3fr) 
    [content-start] min(65ch, 100%) 
    [content-end] minmax(0, 3fr) 
    [wide-end] 1fr 
    [full-end];
}

.full-bleed {
  grid-column: full-start / full-end;
}

.wide-content {
  grid-column: wide-start / wide-end;
}

.prose {
  grid-column: content-start / content-end;
}
```

### 响应式布局策略

```css
/* 响应式布局策略 */

/* 1. 移动优先断点 */
.container {
  /* 基础样式（移动端）*/
  display: flex;
  flex-direction: column;
  padding: 16px;
}

/* 平板 */
@media (min-width: 640px) {
  .container {
    flex-direction: row;
    flex-wrap: wrap;
    padding: 24px;
  }
  
  .container > * {
    flex: 1 1 300px;
  }
}

/* 桌面 */
@media (min-width: 1024px) {
  .container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    max-width: 1200px;
    margin: 0 auto;
  }
}

/* 2. CSS 容器查询（现代方案）*/
.card-wrapper {
  container-type: inline-size;
  container-name: card;
}

.card {
  display: flex;
  flex-direction: column;
}

@container card (min-width: 400px) {
  .card {
    flex-direction: row;
  }
}

@container card (min-width: 600px) {
  .card-image {
    width: 40%;
  }
}

/* 3. Grid 自动响应式 */
.auto-grid {
  display: grid;
  /* 智能自动填充 */
  grid-template-columns: repeat(auto-fit, minmax(min(280px, 100%), 1fr));
  gap: 24px;
}

/* 4. 弹性布局 */
.elastic-grid {
  display: grid;
  /* 使用 clamp() 控制列数 */
  grid-template-columns: repeat(
    clamp(1, 
      calc((100% - 2 * 24px) / 3), 
      4
    ), 
    1fr
  );
  gap: 24px;
}
```

---

## 调试与开发工具

### Chrome DevTools Grid/Flexbox 调试

```css
/* Chrome DevTools 调试技巧 */

/* 1. 启用 Grid 覆盖层 */
.grid-container {
  outline: 2px dashed red;
  outline-offset: 4px;
}

/* 2. 显示网格线 */
.grid-container::before {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  background-image: 
    repeating-linear-gradient(
      90deg,
      transparent,
      transparent calc(200px - 1px),
      #f00 calc(200px - 1px),
      #f00 200px
    );
}

/* 3. 显示 flex 容器 */
.flex-container::before {
  content: 'flex';
  position: absolute;
  top: 0;
  left: 0;
  padding: 4px 8px;
  background: #3b82f6;
  color: white;
  font-size: 12px;
  border-radius: 0 0 4px 0;
}

/* 4. 可视化间距 */
.gap-debug > * {
  outline: 1px solid rgba(255, 0, 0, 0.2);
}
```

### 常见问题诊断

```css
/* 问题诊断 */

/* 1. 元素不显示 */
.debug-hidden {
  display: none; /* 被隐藏了 */
  visibility: hidden; /* 可见性隐藏 */
  opacity: 0; /* 透明度为0 */
  width: 0; /* 宽度为0 */
  height: 0; /* 高度为0 */
  overflow: hidden; /* 溢出隐藏 */
}

/* 2. 溢出问题 */
.debug-overflow {
  overflow-x: auto; /* 水平溢出 */
  overflow-y: hidden; /* 垂直溢出 */
  white-space: nowrap; /* 不换行 */
}

/* 3. 对齐问题 */
.debug-alignment {
  /* 交叉轴对齐检查 */
  align-items: baseline; /* 是否因 baseline 对齐导致错位 */
  
  /* 使用 outline 可视化 */
  outline: 1px solid red;
}

/* 4. 间距问题 */
.debug-spacing {
  /* margin 折叠 */
  margin-top: 24px; /* 可能与下方元素的 margin 折叠 */
  
  /* padding vs margin */
  padding: 24px; /* 内边距在边框内 */
  margin: 24px; /* 外边距在边框外 */
}
```

### 浏览器兼容性处理

```css
/* 浏览器兼容策略 */

/* 1. 使用 @supports 检测 */
@supports (display: grid) {
  .container {
    display: grid;
  }
}

@supports not (display: grid) {
  .container {
    display: flex;
    flex-wrap: wrap;
  }
}

/* 2. Subgrid 兼容性 */
@supports (grid-template-columns: subgrid) {
  .card {
    grid-template-columns: subgrid;
  }
}

/* 3. Gap 兼容性 */
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

/* Safari 旧版本 */
_::-webkit-full-page-media, _:future, .grid {
  display: flex;
  flex-wrap: wrap;
}

_::-webkit-full-page-media, _:future, .grid > * {
  width: calc((100% - 2 * 24px) / 3);
  margin-right: 24px;
  margin-bottom: 24px;
}

_::-webkit-full-page-media, _:future, .grid > *:nth-child(3n) {
  margin-right: 0;
}

/* 4. 容器查询兼容性 */
@supports (container-type: inline-size) {
  .card-wrapper {
    container-type: inline-size;
  }
}

/* 回退方案 */
.card-wrapper {
  /* 基础样式 */
}

@supports (container-type: inline-size) {
  .card-wrapper {
    container-type: inline-size;
  }
  
  /* 增强样式 */
}
```

---

> [!SUMMARY]
> Flexbox 和 CSS Grid 是现代 CSS 布局的两大支柱。Flexbox 擅长处理一维布局和组件内部排列，而 Grid 则是二维布局和复杂页面结构的最佳选择。熟练掌握两者的组合使用，结合容器查询和现代 CSS 特性，可以应对任何复杂的布局需求。

---

*本文档由 [[归愚知识系统]] 自动生成*
