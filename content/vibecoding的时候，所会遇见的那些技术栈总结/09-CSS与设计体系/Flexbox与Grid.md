# Flexbox 与 CSS Grid 完全指南

> [!NOTE]
> 本文档深入讲解 Flexbox 一维布局和 CSS Grid 二维布局的完整能力，涵盖实战模式、底层算法、常见陷阱与性能考量。阅读本文档后，你将对现代 CSS 布局有系统性的理解，能够在实战中灵活选择和组合这两种布局工具。

---

## Flexbox 核心概念

### Flexbox 的定位与适用场景

Flexbox（弹性盒子布局）是 CSS3 引入的一种一维布局模型，它的核心思想是通过「弹性容器」和「弹性项目」之间的关系来实现灵活的元素排列。Flexbox 特别擅长处理那些「沿着主轴或交叉轴排列元素」的场景，比如水平导航栏、垂直居中、卡片列表自动换行等。与传统的浮动布局相比，Flexbox 提供了更精细的对齐控制和更流畅的响应式行为。

在理解 Flexbox 之前，我们需要先明确几个基本概念。Flexbox 布局中存在两个轴：主轴（Main Axis）和交叉轴（Cross Axis）。主轴的方向由 `flex-direction` 属性决定，它可以是水平或垂直的。交叉轴始终垂直于主轴。当我们说「沿着主轴排列」时，指的就是元素在 flex 容器中主要流动方向上的布局；而「沿着交叉轴排列」则是指在垂直于主轴方向上的对齐方式。

Flexbox 特别适合以下几类场景：水平或垂直居中元素；导航栏中 Logo、链接、按钮的灵活分配；卡片网格中自动换行且等高排列；表单控件的对齐；以及任何需要根据可用空间动态分配尺寸的布局。这些场景的共同特点是「一维排列」——即只需要关注单个方向上的布局控制。

### Flex 容器属性详解

激活 Flexbox 布局非常简单，只需要在父元素上设置 `display: flex` 或 `display: inline-flex` 即可。前者创建一个块级 flex 容器，后者创建一个行内 flex 容器。这个简单的声明会立即改变容器的直接子元素的布局行为，它们会自动变成 flex 项目（Flex Item）。

```css
/* 基础容器设置 */
.flex-container {
  display: flex;           /* 块级 flex */
  display: inline-flex;    /* 行内 flex */
  
  /* 主轴方向控制 */
  flex-direction: row;              /* 水平，从左到右（默认）*/
  flex-direction: row-reverse;      /* 水平，从右到左 */
  flex-direction: column;          /* 垂直，从上到下 */
  flex-direction: column-reverse;  /* 垂直，从下到上 */
  
  /* 换行控制 */
  flex-wrap: nowrap;         /* 不换行，强制一行（默认，项目会压缩）*/
  flex-wrap: wrap;          /* 换行，优先保持项目尺寸 */
  flex-wrap: wrap-reverse; /* 换行，但反向排列 */
  
  /* 主轴对齐（沿主轴方向分配空间）*/
  justify-content: flex-start;     /* 向主轴起点对齐（默认）*/
  justify-content: flex-end;        /* 向主轴终点对齐 */
  justify-content: center;          /* 居中对齐 */
  justify-content: space-between;   /* 首尾贴边，中间项目均分空间 */
  justify-content: space-around;    /* 每个项目两侧空间相等（首尾项目空间减半）*/
  justify-content: space-evenly;    /* 所有项目间距完全相等（含首尾）*/
  
  /* 交叉轴对齐（垂直于主轴方向）*/
  align-items: stretch;      /* 拉伸填满交叉轴（默认）*/
  align-items: flex-start;  /* 向交叉轴起点对齐 */
  align-items: flex-end;     /* 向交叉轴终点对齐 */
  align-items: center;       /* 居中对齐 */
  align-items: baseline;    /* 按文字基线对齐 */
  
  /* 多行对齐（仅在 flex-wrap: wrap 时生效）*/
  align-content: flex-start;    /* 行组贴近交叉轴起点 */
  align-content: flex-end;       /* 行组贴近交叉轴终点 */
  align-content: center;         /* 行组居中对齐 */
  align-content: space-between;  /* 首尾行贴近边，中间行均分 */
  align-content: space-around;   /* 类比 justify-content: space-around */
  align-content: space-evenly;   /* 所有行间距完全相等 */
  align-content: stretch;         /* 拉伸填满（默认）*/
}
```

理解 `justify-content` 和 `align-items` 的区别是掌握 Flexbox 的关键。简单来说，`justify-content` 控制主轴方向上的对齐方式，`align-items` 控制交叉轴方向上的对齐方式。当 `flex-direction` 为 `row` 时，主轴是水平方向，`justify-content` 控制水平对齐，`align-items` 控制垂直对齐。当 `flex-direction` 改为 `column` 时，主轴变成垂直方向，此时 `justify-content` 控制垂直对齐，`align-items` 控制水平对齐。

`align-content` 是一个常被忽视但非常强大的属性。它的作用是在多行 flex 容器中（也就是 `flex-wrap: wrap` 时有多行项目的情况）对整个行组进行对齐。注意，只有当 flex 容器中有多行项目时，`align-content` 才会生效。如果只有一行，`align-items` 会作用于这一行中的所有项目，而 `align-content` 则不起作用。

### Flex 项目属性详解

Flex 项目是 flex 容器的直接子元素，它们继承了容器的弹性布局上下文，可以通过自己的属性来控制尺寸、顺序和行为。

```css
/* Flex 项目属性 */
.flex-item {
  /* 增长因子 - 决定项目如何分配剩余空间 */
  flex-grow: 0;   /* 不增长，保持原有尺寸（默认）*/
  flex-grow: 1;   /* 等比增长，有剩余空间时按比例分配 */
  flex-grow: 2;   /* 增长量是其他项目的两倍 */
  
  /* 收缩因子 - 决定空间不足时项目如何收缩 */
  flex-shrink: 1;   /* 允许收缩（默认）*/
  flex-shrink: 0;   /* 不收缩，保持原尺寸 */
  flex-shrink: 0.5; /* 收缩能力减半 */
  
  /* 基准尺寸 - 项目的理想尺寸 */
  flex-basis: auto;       /* 依据自身尺寸（width/height），若无则为内容尺寸（默认）*/
  flex-basis: 200px;      /* 固定基准尺寸 */
  flex-basis: 50%;       /* 相对于容器的百分比 */
  flex-basis: min-content; /* 最小内容宽度 */
  flex-basis: max-content; /* 最大内容宽度 */
  
  /* flex 简写 - 推荐使用简写形式 */
  flex: 0 1 auto;    /* 默认值：不增长、可收缩、按自身尺寸 */
  flex: 1;           /* flex: 1 1 0%，填满剩余空间，常用于等宽分布 */
  flex: 0 0 200px;   /* 不伸缩，固定宽度 200px */
  flex: auto;        /* flex: 1 1 auto，响应式伸缩（增长也收缩）*/
  flex: none;        /* flex: 0 0 auto，完全不伸缩 */
  
  /* 单独控制交叉轴对齐（覆盖容器的 align-items）*/
  align-self: auto;      /* 继承 align-items（默认）*/
  align-self: flex-start;
  align-self: flex-end;
  align-self: center;
  align-self: stretch;
  align-self: baseline;
  
  /* 排列顺序（数值越小越靠前）*/
  order: 0;    /* 默认顺序 */
  order: -1;   /* 移到最前面 */
  order: 1;    /* 移到最后面 */
  order: 999;  /* 排在最后 */
}
```

`flex` 简写属性是日常开发中使用频率最高的。它接受 1 到 3 个值，分别对应 `flex-grow`、`flex-shrink` 和 `flex-basis`。理解这三个值的相互作用对于写出健壮的 Flexbox 代码至关重要。

当只提供一个值时，`flex: 1` 表示 `flex: 1 1 0%`，这意味着项目可以增长（`flex-grow: 1`），可以收缩（`flex-shrink: 1`），基准尺寸为 0。这种配置常用于创建等宽分布的卡片或按钮组。当只提供两个值时，第一个是 `flex-grow`，第二个是 `flex-shrink`，`flex-basis` 默认为 0。三个值都提供时，按照 grow、shrink、basis 的顺序解析。

### Flexbox 底层算法：增长与收缩

理解 Flexbox 的底层算法对于解决复杂布局问题非常重要。Flex 布局分为两个主要阶段：主轴尺寸解析阶段和交叉轴尺寸解析阶段。在主轴尺寸解析阶段，浏览器首先根据 `flex-basis` 确定每个项目的理想尺寸，然后如果有剩余空间，根据 `flex-grow` 值按比例分配；如果空间不足，则根据 `flex-shrink` 值按比例收缩。

假设一个 flex 容器的宽度为 600px，内部有三个 flex 项目，`flex-basis` 都是 200px。计算过程如下：三个项目的初始总宽度为 600px，正好填满容器，没有剩余空间，所以 `flex-grow` 不起作用。如果容器宽度变为 400px，则溢出 200px，系统会按照各项目的 `flex-shrink` 值进行收缩。如果三个项目的 `flex-shrink` 都是 1，则每个项目收缩 200/3 ≈ 66.67px，最终每个项目宽度约为 133.33px。

收缩的精确算法是：计算每个项目的「收缩权重」（`flex-shrink × flex-basis`），然后按照权重比例分配溢出的空间。这种算法确保了大项目比小项目收缩更多，保持了相对比例的协调性。

---

## CSS Grid 深入指南

### Grid 的定位与适用场景

CSS Grid 是 CSS3 引入的二维布局系统，它的出现填补了 Flexbox 的不足。如果说 Flexbox 是为「一维流」设计的，那么 Grid 就是为「二维结构」设计的。Grid 允许我们同时控制行和列的布局，可以精确地指定任何元素在任何网格区域中的位置，这使得它特别适合页面级布局、仪表盘、数据表格以及任何需要精确行列对齐的设计。

Grid 的核心概念包括网格线（Grid Lines）、网格轨道（Grid Tracks）和网格区域（Grid Areas）。网格线是划分网格的线条，有行网格线和列网格线之分，通过编号（从 1 开始）或命名可以精确定位。网格轨道是相邻两条网格线之间的区域，即我们通常说的「行」和「列」。网格区域是由四条网格线围成的矩形区域，可以命名并赋给任意元素。

Grid 特别适合以下场景：页面整体布局（如经典的头部-侧边栏-主内容-底部布局）；需要精确行列对齐的数据展示；杂志式排版中图片与文字的复杂穿插；响应式卡片网格；以及任何需要同时控制水平和垂直布局的场景。值得注意的是，Grid 和 Flexbox 并不是互斥的，而是互补的。一个常见的最佳实践是：页面级布局用 Grid，组件内部布局用 Flexbox。

### Grid 基础配置

```css
/* Grid 基础配置 */
.grid-container {
  display: grid;   /* 或 display: inline-grid */
  
  /* 定义列轨道 */
  grid-template-columns: 200px 1fr 200px;  /* 三列：固定200px + 自适应 + 固定200px */
  grid-template-columns: 100px 2fr 1fr;     /* 比例定义：100px + 2份 + 1份 */
  
  /* 使用 repeat() 简化重复定义 */
  grid-template-columns: repeat(4, 1fr);      /* 4等列 */
  grid-template-columns: repeat(3, minmax(200px, 1fr));
  
  /* minmax() 函数：定义轨道尺寸范围 */
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));  /* 自动填充，最小250px */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));   /* 自动适应，空白列被压缩 */
  
  /* 定义行轨道 */
  grid-template-rows: auto 1fr auto;  /* 三行：自适应 + 占据剩余 + 自适应 */
  
  /* 间距（gutter）*/
  gap: 24px;           /* 行列统一间距 */
  row-gap: 16px;       /* 单独行间距 */
  column-gap: 32px;    /* 单独列间距 */
}
```

`auto-fill` 和 `auto-fit` 是两个非常有用但容易混淆的关键字。它们都用于自动生成足够数量的轨道来填满容器，区别在于：当容器有剩余空间时，`auto-fill` 会保留空轨道的位置，而 `auto-fit` 会将空轨道压缩为 0。简单来说，`auto-fit` 更适合内容驱动的响应式布局，`auto-fill` 更适合需要保持固定列数的场景。

### 网格定位与命名

Grid 提供了多种定位方式，可以根据项目需求选择最合适的方法。

```css
/* 按网格线编号定位 */
.grid-item {
  grid-column: 1 / 3;       /* 从第1条线到第3条线，跨2列 */
  grid-column: 1 / span 2;  /* 从第1条线开始，跨2列 */
  grid-column: 1 / -1;      /* 从第1条线到倒数第1条线，即跨越所有列 */
  grid-column: 2;           /* 简写：放置在第2条线上 */
  
  grid-row: 2 / 4;          /* 从第2条线到第4条线，跨2行 */
  grid-row: span 2;         /* 跨越2行 */
  
  /* 简写：row-start / col-start / row-end / col-end */
  grid-area: 1 / 1 / 3 / 3;  /* 占据左上角2x2区域 */
}

/* 使用命名区域（最直观的布局方式）*/
.page-layout {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
}

.page-layout > header { grid-area: header; }
.page-layout > nav    { grid-area: sidebar; }
.page-layout > main   { grid-area: main; }
.page-layout > aside  { grid-area: aside; }
.page-layout > footer { grid-area: footer; }

/* 命名网格线（适用于复杂布局）*/
.named-grid {
  display: grid;
  grid-template-columns: 
    [full-start sidebar-start] 200px
    [sidebar-end content-start] 1fr
    [content-end aside-start] 200px
    [aside-end full-end];
}

.full-bleed {
  grid-column: full-start / full-end;  /* 跨越整个网格 */
}

.sidebar {
  grid-column: sidebar-start / sidebar-end;  /* 左侧边栏 */
}

.content {
  grid-column: content-start / content-end;  /* 主内容区 */
}
```

命名区域是 Grid 布局中最直观的定位方式。通过 `grid-template-areas` 定义可视化的区域图，然后通过 `grid-area` 将元素关联到相应区域。这种方式的优势是布局意图一目了然，代码可读性极高。在响应式设计中，可以通过媒体查询改变 `grid-template-areas` 的定义来调整布局，而无需修改具体元素的定位代码。

### Subgrid（子网格对齐）

Subgrid 是 Grid 的高级特性，允许嵌套元素继承父网格的轨道定义，实现跨层级的精确对齐。这解决了长期以来「卡片内部元素与外部元素对齐」的难题。

```css
/* Subgrid 基本用法 */
.card-collection {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;  /* 定义3行轨道 */
  gap: 24px;
}

.card {
  display: grid;
  /* 继承父网格的列轨道 */
  grid-template-columns: subgrid;
  /* 继承父网格的行轨道，并跨越3行 */
  grid-template-rows: subgrid;
  grid-row: span 3;
  
  background: white;
  border-radius: 12px;
  overflow: hidden;
}

/* 卡片内部元素自动对齐到父网格 */
.card-image {
  grid-column: 1 / -1;  /* 跨越所有列 */
}

.card-title {
  /* 标题与其他卡片的标题自动对齐在同一水平线上 */
}

.card-description {
  /* 描述区域与其他卡片等高对齐 */
}

.card-footer {
  grid-column: 1 / -1;
  /* 底部按钮与其他卡片的底部对齐 */
}
```

Subgrid 的典型应用场景是卡片网格的对齐。想象一个产品列表，每个产品的名称长度不同，如果不用 Subgrid，很难让「查看详情」按钮在所有卡片底部对齐。使用 Subgrid 后，所有卡片共享父容器的行轨道定义，无论卡片内容多少，按钮都会对齐在同一水平线上。这个特性在 2024 年已获得主流浏览器全面支持，可以放心使用。

### 隐式网格与自动定位

当网格项目的数量超过明确定义的轨道数时，Grid 会自动创建隐式轨道来容纳多余的项目。

```css
/* 隐式网格配置 */
.implicit-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);  /* 只定义3列 */
  grid-template-rows: 200px 200px;        /* 只定义2行 */
  gap: 16px;
}

/* 未明确定义的行会自动创建 */
.implicit-grid > .item:nth-child(7) {
  /* 第7个项目会触发创建第3行隐式行 */
}

/* 控制隐式轨道尺寸 */
.implicit-grid {
  grid-auto-rows: minmax(100px, auto);  /* 隐式行最小100px，最大自适应内容 */
  grid-auto-flow: row;    /* 优先沿行方向填充（默认）*/
  grid-auto-flow: column; /* 优先沿列方向填充 */
  grid-auto-flow: dense;  /* 启用密集打包，自动填充空白位置 */
}
```

隐式网格使得 Grid 能够优雅地处理动态内容。你不需要预先知道会有多少个项目，Grid 会自动创建足够的轨道来容纳所有内容。通过 `grid-auto-rows` 可以控制这些自动创建的行的高度，而 `grid-auto-flow` 则控制项目的填充方向。`dense` 值是一个强大的特性，它会尝试用后面的项目填补前面留下的空白，适用于瀑布流式布局，但要注意这可能导致项目的视觉顺序与 DOM 顺序不一致，影响可访问性。

---

## Flexbox vs Grid：选择与组合

### 决策指南

选择 Flexbox 还是 Grid 是前端开发中的常见问题。理解两者的本质区别是做出正确选择的前提。

Flexbox 是内容驱动的布局模型，它的核心是「让内容自己决定如何排列」。当我们给一个 flex 容器设置 `flex: 1` 时，实际上是说「这些项目自己去分配剩余空间吧」。Flexbox 不知道也不关心下一个项目是什么，它只负责协调当前项目之间的关系。

Grid 是结构驱动的布局模型，它的核心是「先定义结构，内容填充进去」。在 Grid 中，你需要预先规划好有多少行、多少列，然后决定每个元素占据哪个区域。Grid 知道你有多少空间，然后根据你的定义分配给每个元素。

一个实用的判断方法是：如果你在写 HTML 的时候就能确定布局结构，用 Grid；如果你需要内容自己决定布局方式，用 Flexbox。例如，导航栏的链接数量在 HTML 编写时是知道的，可以固定分配，但链接的具体长度由内容决定，用 Flexbox 更合适。页面整体布局在设计阶段就已确定，用 Grid 更清晰。

```
┌────────────────────────────────────────────────────────────────┐
│                    Flexbox vs Grid 决策树                         │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  需要布局什么？                                                   │
│  │                                                              │
│  ├─ 单行/单列元素，方向一致                                       │
│  │   └─ Flexbox ✓                                               │
│  │       - 水平/垂直导航栏                                        │
│  │       - 按钮组                                                │
│  │       - 列表项对齐                                            │
│  │       - 单行/单列居中                                          │
│  │                                                              │
│  ├─ 需要精确的行列结构                                            │
│  │   └─ CSS Grid ✓                                               │
│  │       - 页面整体布局                                          │
│  │       - 仪表盘格子                                            │
│  │       - 栅格卡片系统                                          │
│  │       - 数据表格                                              │
│  │                                                              │
│  ├─ 组件内部小范围布局                                           │
│  │   ├─ 内容数量固定 → Grid ✓                                   │
│  │   └─ 内容数量不定 → Flexbox ✓                                │
│  │                                                              │
│  └─ 复杂混合场景                                                │
│      └─ Grid + Flexbox ✓                                        │
│          - Grid 定义页面骨架                                     │
│          - Flexbox 处理组件内部                                  │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### 混合使用实战

在真实项目中，Grid 和 Flexbox 往往需要配合使用才能达到最佳效果。常见的模式是使用 Grid 定义页面级布局，在具体的组件内部使用 Flexbox 处理组件自身的排列。

```css
/* 页面布局：Grid */
.page-layout {
  display: grid;
  grid-template-columns: 250px 1fr;
  grid-template-rows: 80px 1fr auto;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
  min-height: 100vh;
}

/* Header 内部：Flexbox */
.header {
  grid-area: header;
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
  gap: 24px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 12px;
}

/* Sidebar 内部：Flexbox 纵向排列 */
.sidebar {
  grid-area: sidebar;
  display: flex;
  flex-direction: column;
}

.sidebar-nav {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

/* 卡片网格：Grid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 24px;
}

/* 单个卡片内部：Flexbox */
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
  flex: 1;  /* 内容区占据剩余空间 */
}

.card-footer {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-top: auto;  /* 始终贴底 */
}
```

---

## 实战布局模式

### 响应式卡片网格

响应式卡片网格是现代网页中最常见的布局模式之一。下面的实现利用 `auto-fill` 和 `minmax()` 实现了无需媒体查询就能自动响应的卡片布局。

```css
/* 自动响应式卡片网格 */
.responsive-card-grid {
  display: grid;
  /* 关键：自动填充，每列最小280px，不超过1fr */
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

.card {
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 12px;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
  overflow: hidden;
  /* 可选：悬停效果 */
  transition: transform 200ms ease, box-shadow 200ms ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 12px 24px rgb(0 0 0 / 0.12);
}

.card-image {
  aspect-ratio: 16 / 9;
  overflow: hidden;
}

.card-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 500ms ease;
}

.card:hover .card-image img {
  transform: scale(1.05);
}

.card-content {
  display: flex;
  flex-direction: column;
  flex: 1;
  padding: 20px;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 600;
  margin-bottom: 8px;
  color: #0f172a;
}

.card-description {
  font-size: 0.875rem;
  color: #64748b;
  line-height: 1.6;
  margin-bottom: 16px;
  flex: 1;
}

.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-top: 16px;
  border-top: 1px solid #e2e8f0;
}
```

这个实现的核心在于 `grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`。当容器宽度大于 280px 时，Grid 会创建尽可能多的 280px 列；当容器宽度不足时，每列自动扩展到填满可用空间。整个过程是自动的，无需编写任何媒体查询。

### 圣杯布局

圣杯布局（Holy Grail Layout）是经典的三栏式页面布局，两侧是固定宽度的边栏，中间是自适应的内容区，底部是 footer。下面的实现同时展示了 Flexbox 和 Grid 两种方式。

```css
/* Grid 版本的圣杯布局 */
.holy-grail-grid {
  display: grid;
  grid-template-columns: 200px 1fr 200px;  /* 固定侧栏 + 自适应内容 */
  grid-template-rows: 64px 1fr auto;       /* 头部 + 主内容区 + footer */
  grid-template-areas:
    "header header header"
    "nav main aside"
    "footer footer footer";
  min-height: 100vh;
}

.holy-grail-grid > header {
  grid-area: header;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  background: white;
  border-bottom: 1px solid #e2e8f0;
}

.holy-grail-grid > nav { grid-area: nav; }
.holy-grail-grid > main { 
  grid-area: main; 
  padding: 24px;
  overflow-y: auto;
}
.holy-grail-grid > aside { grid-area: aside; }
.holy-grail-grid > footer { 
  grid-area: footer; 
  padding: 16px 24px;
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
}

/* 响应式：平板以下变为单列 */
@media (max-width: 768px) {
  .holy-grail-grid {
    grid-template-columns: 1fr;
    grid-template-areas:
      "header"
      "main"
      "footer";
  }
  
  .holy-grail-grid > nav,
  .holy-grail-grid > aside {
    display: none;  /* 隐藏侧栏，或改为折叠面板 */
  }
}
```

### 内容宽度控制（写作排版）

对于内容型网站，合理的文字宽度和图片展示是良好阅读体验的关键。下面的布局实现了正文宽度受限、全宽图片、超宽图表的多层次内容控制。

```css
/* 内容宽度控制布局 */
.reading-layout {
  display: grid;
  /* 使用命名线定义5个区域：全宽-宽内容-正文-宽内容-全宽 */
  grid-template-columns: 
    [full-start] 1fr 
    [wide-start content-start] minmax(0, 3fr)
    [content-start] min(65ch, 100%) 
    [content-end wide-end] minmax(0, 3fr)
    [wide-end] 1fr 
    [full-end];
  gap: 0;
}

/* 全宽内容（如大图、背景色块）*/
.full-bleed {
  grid-column: full-start / full-end;
}

/* 正常正文内容 */
.prose {
  grid-column: content-start / content-end;
  font-size: 1rem;
  line-height: 1.75;
  color: #334155;
}

/* 宽图表（如代码块、表格）*/
.wide-figure {
  grid-column: wide-start / wide-end;
}

/* 超宽媒体 */
.super-wide {
  grid-column: full-start / full-end;
}
```

这种布局技术常被知名内容网站采用。65ch 的正文宽度被认为是最舒适的阅读宽度，因为每行字符数大约在 65-75 之间时，眼睛不需要过度移动就能完成阅读行的扫描。超出正文的内容（如代码块、引用块）可以延伸到「宽区域」，而全宽内容（如 Hero 区域的大图）则可以跨越整个页面。

---

## 常见陷阱与解决方案

### Flexbox 陷阱

**陷阱一：flex 子项不换行导致溢出**

这是最常见的 Flexbox 问题。当容器空间不足时，flex 项目默认会收缩以适应容器，但如果项目内容不可压缩（如长英文单词或 URL），就会导致溢出。

```css
/* 问题代码 */
.bad-flex {
  display: flex;
}

.bad-item {
  flex-shrink: 1;  /* 默认允许收缩，但文字不会换行 */
  /* 结果：容器被撑破 */
}

/* 解决方案 */
.good-flex {
  display: flex;
  flex-wrap: wrap;  /* 允许换行 */
}

.good-item {
  flex-shrink: 1;
  min-width: 0;  /* 关键！允许收缩到0宽度 */
  /* 或者 */
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```

**陷阱二：flex-basis 与 width/height 冲突**

当同时指定 `flex-basis` 和 `width` 时，`flex-basis` 的优先级更高，因为它处于简写属性的「更高位置」。这可能导致意外的行为。

```css
/* 问题代码 */
.conflicting {
  flex-basis: 200px;
  width: 50%;  /* 会被忽略！flex-basis 优先级更高 */
}

/* 解决方案 */
.correct {
  flex: 0 1 200px;  /* 显式设置 shrink=1 */
  /* 或者如果需要百分比 */
  flex: 1 1 50%;    /* 使用 flex 的百分比 */
}
```

**陷阱三：gap 与 margin 混用**

`gap` 属性是专门为 Flexbox 和 Grid 设计的间距属性，它不会产生双倍间距的问题。但如果同时使用 `gap` 和 `margin`，就会产生意外的间距叠加。

```css
/* 问题代码 */
.bad-mix {
  display: flex;
  gap: 16px;
}

.bad-mix > * {
  margin: 16px;  /* 结果是 32px 的间距 */
}

/* 解决方案 */
.good-flex {
  display: flex;
  gap: 16px;  /* 只用 gap */
  /* 如果需要边距，使用 padding 而不是 margin */
}
```

### Grid 陷阱

**陷阱一：忘记 min-width: 0**

在 Grid 中，flex 项目的默认 `min-width` 是 `auto`，这意味着它们不会收缩到小于内容尺寸。当项目包含不可换行的内容（如长文本或固定宽度图片）时，就会导致整行或整列溢出。

```css
/* 问题：内容溢出 */
.overflow-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.overflow-item {
  /* 长文本不会自动换行，导致列宽被撑大 */
}

/* 解决方案 */
.fixed-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
}

.fixed-item {
  min-width: 0;  /* 允许收缩 */
  overflow: hidden;
  text-overflow: ellipsis;
}
```

**陷阱二：grid-area 名称不匹配**

`grid-area` 的值必须与 `grid-template-areas` 中定义的名称完全匹配，包括大小写。拼写错误不会产生任何错误提示，只会导致元素无法放置。

```css
/* 问题：拼写错误 */
.layout {
  display: grid;
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}

.header { grid-area: header; }
.sidebar { grid-area: siderbar; }  /* 拼写错误！应该是 sidebar */
.main { grid-area: main; }

/* 解决方案：仔细检查名称一致性 */
.sidebar { grid-area: sidebar; }  /* 正确 */
```

**陷阱三：隐式网格尺寸不可控**

当项目数量超过定义的轨道数时，隐式创建的轨道尺寸默认为 `auto`，可能导致高度不一致。

```css
/* 问题：隐式行高度不确定 */
.uncontrolled {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  /* 超过3行的项目会创建隐式行，高度不确定 */
}

/* 解决方案：设置 grid-auto-rows */
.controlled {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: minmax(100px, auto);  /* 最小100px，最大自适应 */
}
```

---

> [!TIP]
> **黄金法则**：用 Flexbox 布局「流」（一维），用 Grid 布局「结构」（二维）。大多数 UI 组件用 Flexbox 就能解决；页面级布局用 Grid 更清晰。两者配合使用，取长补短。

---

> [!SUMMARY]
> Flexbox 和 CSS Grid 是现代 CSS 布局的两大支柱。Flexbox 以其灵活性和对内容自适应的能力见长，适合处理一维排列和组件内部布局。Grid 以其精确的结构控制和二维布局能力见长，适合页面级布局和需要行列对齐的场景。熟练掌握两者的组合使用，结合 Subgrid 和容器查询等现代特性，可以应对任何复杂的布局需求。

---

*本文档由 [[归愚知识系统]] 自动生成*
