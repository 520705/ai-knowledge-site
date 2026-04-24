# shadcn/ui 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 shadcn/ui 设计哲学、核心组件、变体系统、与传统 UI 库对比及 AI 应用集成。

---

## 目录

1. [[#shadcn/ui 概述]]
2. [[#设计哲学：Copy vs Install]]
3. [[#shadcn/ui vs MUI vs Ant Design vs Chakra UI 对比]]
4. [[#核心组件列表]]
5. [[#Radix UI 原语]]
6. [[#变体系统（Variants）]]
7. [[#CLI 工具使用]]
8. [[#Tailwind CSS 集成]]
9. [[#组件开发规范]]
10. [[#AI 应用实战]]
11. [[#选型建议]]

---

## shadcn/ui 概述

### 什么是 shadcn/ui

shadcn/ui 并不是一个传统的 UI 组件库，而是一个**组件集合**和**最佳实践集**。其核心理念是"Copy to your project"（复制到你的项目），组件源码直接复制到你的代码库中，而非通过 npm 安装包。这意味着你拥有完整的组件代码，可以自由修改、定制和扩展。

**shadcn/ui 核心特点：**

| 特性 | 说明 |
|------|------|
| **Copy to your project** | 组件复制到本地，完全控制 |
| **无 npm 包依赖** | 组件代码在项目中，可随意修改 |
| **基于 Radix UI** | 使用 Radix UI 原语实现无障碍 |
| **Tailwind CSS 样式** | 使用 Tailwind CSS 进行样式控制 |
| **可定制变体** | 通过 CSS 变量和类名系统定制外观 |
| **TypeScript 优先** | 完全类型安全 |

### shadcn/ui 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| shadcn/ui 0.1 | 2022 | 发布基础版，Button/Dialog/Input |
| shadcn/ui 0.3-0.5 | 2023 | 添加 DataTable/Form/Sheet |
| **shadcn/ui v1** | **2024** | **全新变体系统，改进 CLI，文档重构** |

### 核心理念

**shadcn/ui 的设计哲学：**

1. **所有权**：组件代码在项目中，你拥有完全控制权
2. **简单性**：避免过度抽象，保持代码清晰
3. **可访问性**：基于 Radix UI，确保无障碍支持
4. **可定制**：通过 Tailwind CSS 变量轻松定制外观
5. **AI 友好**：组件代码可被 AI 理解和修改

> [!IMPORTANT]
> shadcn/ui 不是"安装即用"的组件库，而是需要你"复制代码，理解代码，然后定制代码"。这种模式特别适合 AI 编程，因为 AI 可以直接理解和修改组件源码。

---

## 设计哲学：Copy vs Install

### 传统 UI 库：Install

```
npm install @mui/material

// 使用
import { Button } from '@mui/material';
<Button>Click me</Button>

// 问题：
// 1. 组件黑盒，难以定制
// 2. 样式被库锁定
// 3. 升级可能破坏定制
// 4. AI 难以理解内部实现
```

### shadcn/ui：Copy

```
npx shadcn@latest add button

// 生成文件：
// components/ui/button.tsx

// 使用
import { Button } from '@/components/ui/button';
<Button>Click me</Button>

// 优势：
// 1. 组件代码在项目中，完全透明
// 2. 可直接修改源码
// 3. 不受库升级影响
// 4. AI 可直接理解和修改
```

### 代码所有权对比

**MUI Button 源码（黑盒）：**

```tsx
// 你只能使用，无法修改内部实现
import { Button as MuiButton } from '@mui/material';

// 定制只能通过 props，能力有限
<MuiButton
  variant="contained"
  color="primary"
  size="large"
  disabled={false}
  startIcon={<Icon />}
>
  Click me
</MuiButton>
```

**shadcn/ui Button 源码（完全透明）：**

```tsx
// components/ui/button.tsx
// 你可以完全看到和修改这个组件

import * as React from "react"
import { Slot } from "@radix-ui/react-slot"
import { cva, type VariantProps } from "class-variance-authority"

import { cn } from "@/lib/utils"

const buttonVariants = cva(
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50 [&_svg]:pointer-events-none [&_svg]:size-4 [&_svg]:shrink-0",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-8",
        icon: "h-9 w-9",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)

// 你可以完全修改这个组件
export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : "button"
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    )
  }
)
Button.displayName = "Button"

export { Button, buttonVariants }
```

---

## shadcn/ui vs MUI vs Ant Design vs Chakra UI 对比

### 框架特性对比

| 特性 | shadcn/ui | MUI (Material UI) | Ant Design | Chakra UI |
|------|-----------|------------------|------------|-----------|
| **安装方式** | Copy | npm install | npm install | npm install |
| **代码位置** | 项目内 | node_modules | node_modules | node_modules |
| **样式方案** | Tailwind CSS | Emotion/Styled | Less | Emotion/Styled |
| **TypeScript** | 原生 | 原生 | 原生 | 原生 |
| **无障碍** | 优秀（Radix） | 良好 | 良好 | 良好 |
| **定制难度** | 简单（源码可见） | 复杂 | 复杂 | 中等 |
| **包体积影响** | 零（按需复制） | ~200KB | ~300KB | ~150KB |
| **主题系统** | CSS 变量 | Theme Provider | ConfigProvider | Theme Provider |
| **组件数量** | ~50 个 | ~50 个 | ~80 个 | ~50 个 |
| **AI 可读性** | 优秀 | 较差 | 较差 | 中等 |

### 开发体验对比

| 维度 | shadcn/ui | MUI | Ant Design | Chakra UI |
|------|-----------|-----|------------|-----------|
| **初始设置** | 简单 | 复杂 | 复杂 | 中等 |
| **组件导入** | 直接引用 | npm 包 | npm 包 | npm 包 |
| **样式覆盖** | 源码修改/类名 | sx prop/主题 | less 变量 | sx prop/主题 |
| **状态管理** | React 状态 | React 状态 | React 状态 | 内部状态 |
| **文档质量** | 优秀 | 优秀 | 良好 | 良好 |
| **社区支持** | 增长中 | 庞大 | 庞大 | 中等 |

### 适用场景对比

| 场景 | 首选 | 原因 |
|------|------|------|
| **AI 辅助开发** | shadcn/ui | 代码透明，AI 可理解修改 |
| **快速原型** | shadcn/ui / Chakra | 配置简单 |
| **企业后台** | Ant Design / MUI | 组件丰富 |
| **设计系统** | shadcn/ui | 完全可控 |
| **Material Design** | MUI | 官方实现 |
| **国产项目** | Ant Design | 中文文档，本地化 |

---

## 核心组件列表

### 基础组件

| 组件 | 说明 | 文件位置 |
|------|------|---------|
| **Button** | 按钮，支持多种变体和尺寸 | `components/ui/button.tsx` |
| **Input** | 文本输入框 | `components/ui/input.tsx` |
| **Label** | 表单标签 | `components/ui/label.tsx` |
| **Form** | 表单组件 | `components/ui/form.tsx` |
| **Textarea** | 多行文本输入 | `components/ui/textarea.tsx` |
| **Select** | 下拉选择 | `components/ui/select.tsx` |
| **Checkbox** | 复选框 | `components/ui/checkbox.tsx` |
| **RadioGroup** | 单选组 | `components/ui/radio-group.tsx` |
| **Switch** | 开关 | `components/ui/switch.tsx` |
| **Slider** | 滑块 | `components/ui/slider.tsx` |

### 反馈组件

| 组件 | 说明 | 文件位置 |
|------|------|---------|
| **Alert** | 警告提示 | `components/ui/alert.tsx` |
| **AlertDialog** | 警告对话框 | `components/ui/alert-dialog.tsx` |
| **Toast** | 轻提示 | `components/ui/toast.tsx` |
| **Tooltip** | 工具提示 | `components/ui/tooltip.tsx` |
| **Progress** | 进度条 | `components/ui/progress.tsx` |
| **Skeleton** | 骨架屏 | `components/ui/skeleton.tsx` |
| **Spinner** | 加载动画 | `components/ui/spinner.tsx` |
| **Badge** | 徽章 | `components/ui/badge.tsx` |

### 布局组件

| 组件 | 说明 | 文件位置 |
|------|------|---------|
| **Card** | 卡片 | `components/ui/card.tsx` |
| **Sheet** | 侧边抽屉 | `components/ui/sheet.tsx` |
| **Dialog** | 对话框 | `components/ui/dialog.tsx` |
| **Drawer** | 抽屉（移动端） | `components/ui/drawer.tsx` |
| **Tabs** | 标签页 | `components/ui/tabs.tsx` |
| **Accordion** | 手风琴 | `components/ui/accordion.tsx` |
| **Separator** | 分隔线 | `components/ui/separator.tsx` |
| **AspectRatio** | 宽高比 | `components/ui/aspect-ratio.tsx` |

### 数据展示组件

| 组件 | 说明 | 文件位置 |
|------|------|---------|
| **Table** | 表格 | `components/ui/table.tsx` |
| **DataTable** | 数据表格（高级） | `components/ui/data-table.tsx` |
| **Avatar** | 头像 | `components/ui/avatar.tsx` |
| **Calendar** | 日历 | `components/ui/calendar.tsx` |
| **Chart** | 图表 | `components/ui/chart.tsx` |
| **HoverCard** | 悬停卡片 | `components/ui/hover-card.tsx` |
| **Popover** | 弹出框 | `components/ui/popover.tsx` |

### 导航组件

| 组件 | 说明 | 文件位置 |
|------|------|---------|
| **NavigationMenu** | 导航菜单 | `components/ui/navigation-menu.tsx` |
| **Breadcrumb** | 面包屑 | `components/ui/breadcrumb.tsx` |
| **DropdownMenu** | 下拉菜单 | `components/ui/dropdown-menu.tsx` |
| **ContextMenu** | 右键菜单 | `components/ui/context-menu.tsx` |
| **Menubar** | 菜单栏 | `components/ui/menubar.tsx` |
| **Tabs** | 标签导航 | `components/ui/tabs.tsx` |
| **ScrollArea** | 滚动区域 | `components/ui/scroll-area.tsx` |

---

## Radix UI 原语

### 什么是 Radix UI

Radix UI 是一组无样式、可访问的 UI 原语（Primitives）。shadcn/ui 使用 Radix UI 作为底层实现，确保组件的无障碍支持。

### 常用 Radix 原语

| Radix 包 | 功能 | shadcn 组件 |
|---------|------|------------|
| `@radix-ui/react-dialog` | 对话框 | Dialog, AlertDialog |
| `@radix-ui/react-dropdown-menu` | 下拉菜单 | DropdownMenu |
| `@radix-ui/react-select` | 选择器 | Select |
| `@radix-ui/react-tabs` | 标签页 | Tabs |
| `@radix-ui/react-tooltip` | 工具提示 | Tooltip |
| `@radix-ui/react-accordion` | 手风琴 | Accordion |
| `@radix-ui/react-checkbox` | 复选框 | Checkbox |
| `@radix-ui/react-progress` | 进度条 | Progress |
| `@radix-ui/react-slider` | 滑块 | Slider |
| `@radix-ui/react-switch` | 开关 | Switch |
| `@radix-ui/react-popover` | 弹出框 | Popover |
| `@radix-ui/react-scroll-area` | 滚动区域 | ScrollArea |

### Radix Dialog 示例

```tsx
import * as React from "react"
import * as DialogPrimitive from "@radix-ui/react-dialog"
import { X } from "lucide-react"

import { cn } from "@/lib/utils"

const Dialog = DialogPrimitive.Root

const DialogTrigger = DialogPrimitive.Trigger

const DialogPortal = DialogPrimitive.Portal

const DialogClose = DialogPrimitive.Close

const DialogOverlay = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Overlay>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Overlay>
>(({ className, ...props }, ref) => (
  <DialogPrimitive.Overlay
    ref={ref}
    className={cn(
      "fixed inset-0 z-50 bg-black/80 data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0",
      className
    )}
    {...props}
  />
))
DialogOverlay.displayName = DialogPrimitive.Overlay.displayName

const DialogContent = React.forwardRef<
  React.ElementRef<typeof DialogPrimitive.Content>,
  React.ComponentPropsWithoutRef<typeof DialogPrimitive.Content>
>(({ className, children, ...props }, ref) => (
  <DialogPortal>
    <DialogOverlay />
    <DialogPrimitive.Content
      ref={ref}
      className={cn(
        "fixed left-[50%] top-[50%] z-50 grid w-full max-w-lg translate-x-[-50%] translate-y-[-50%] gap-4 border bg-background p-6 shadow-lg duration-200 data-[state=open]:animate-in data-[state=closed]:animate-out data-[state=closed]:fade-out-0 data-[state=open]:fade-in-0 data-[state=closed]:zoom-out-95 data-[state=open]:zoom-in-95 data-[state=closed]:slide-out-to-left-1/2 data-[state=closed]:slide-out-to-top-[48%] data-[state=open]:slide-in-from-left-1/2 data-[state=open]:slide-in-from-top-[48%] sm:rounded-lg",
        className
      )}
      {...props}
    >
      {children}
      <DialogPrimitive.Close className="absolute right-4 top-4 rounded-sm opacity-70 ring-offset-background transition-opacity hover:opacity-100 focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 disabled:pointer-events-none data-[state=open]:bg-accent data-[state=open]:text-muted-foreground">
        <X className="h-4 w-4" />
        <span className="sr-only">Close</span>
      </DialogPrimitive.Close>
    </DialogPrimitive.Content>
  </DialogPortal>
))
DialogContent.displayName = DialogPrimitive.Content.displayName

export {
  Dialog,
  DialogPortal,
  DialogOverlay,
  DialogTrigger,
  DialogClose,
  DialogContent,
}
```

---

## 变体系统（Variants）

### CVA（Class Variance Authority）

shadcn/ui 使用 `class-variance-authority`（CVA）库来管理组件变体：

```tsx
import { cva, type VariantProps } from "class-variance-authority"

const buttonVariants = cva(
  // 基础样式
  "inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-1 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      // 变体选项
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
        outline: "border border-input bg-background hover:bg-accent hover:text-accent-foreground",
        secondary: "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        ghost: "hover:bg-accent hover:text-accent-foreground",
        link: "text-primary underline-offset-4 hover:underline",
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3 text-xs",
        lg: "h-10 rounded-md px-8",
        icon: "h-9 w-9",
      },
    },
    // 默认变体
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
)
```

### 使用变体

```tsx
import { Button } from "@/components/ui/button"
import { buttonVariants } from "@/components/ui/button"

// 使用组件
<Button variant="destructive" size="sm">
  Delete
</Button>

// 单独使用变体（不需要 Button 组件）
<button className={buttonVariants({ variant: "outline", size: "lg" })}>
  Custom Button
</button>
```

### 自定义变体

```tsx
// 创建带新变体的按钮
const customButtonVariants = cva(
  buttonVariants(),
  {
    variants: {
      // 扩展原有变体
      variant: {
        ...buttonVariants().variants?.variant,
        ai: "bg-gradient-to-r from-purple-500 to-pink-500 text-white hover:from-purple-600 hover:to-pink-600",
      },
    },
  }
)

// 使用新变体
<button className={customButtonVariants({ variant: "ai" })}>
  AI Powered
</button>
```

---

## CLI 工具使用

### 初始化

```bash
# 创建新项目（Next.js + shadcn/ui）
npx create-next-app@latest my-app --typescript --tailwind --eslint

# 进入项目目录
cd my-app

# 初始化 shadcn/ui
npx shadcn@latest init

# 选择配置选项：
# - Style: Default
# - Base Color: Slate
# - CSS file: globals.css
# - CSS variables: Yes
# - Customize: Yes
```

### 添加组件

```bash
# 添加单个组件
npx shadcn@latest add button

# 添加多个组件
npx shadcn@latest add button dialog input label

# 添加所有组件
npx shadcn@latest add --all
```

### CLI 命令参考

| 命令 | 功能 |
|------|------|
| `init` | 初始化 shadcn/ui 配置 |
| `add <component>` | 添加组件 |
| `add --all` | 添加所有组件 |
| `add --yes` | 无需确认添加 |
| `upgrade` | 升级组件到最新版本 |
| `diff` | 查看组件差异 |
| `Doctor` | 检查配置问题 |

### 升级组件

```bash
# 升级单个组件
npx shadcn@latest upgrade button

# 升级所有组件
npx shadcn@latest upgrade

# 预览升级差异
npx shadcn@latest diff button
```

---

## Tailwind CSS 集成

### CSS 变量系统

shadcn/ui 使用 CSS 变量定义主题：

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ... 更多暗色模式变量 */
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### 自定义主题

```css
/* 自定义主题变量 */
@layer base {
  :root {
    /* 品牌颜色 */
    --brand: 262 83% 58%;
    --brand-foreground: 0 0% 100%;
  }
}
```

```tsx
// 使用品牌颜色
<div className="bg-[hsl(var(--brand))] text-[hsl(var(--brand-foreground))]">
  Brand Color
</div>
```

---

## 组件开发规范

### 项目结构

```
src/
├── components/
│   ├── ui/              # shadcn/ui 组件
│   │   ├── button.tsx
│   │   ├── dialog.tsx
│   │   └── ...
│   └── custom/          # 自定义组件
│       ├── chat-message.tsx
│       └── ai-response.tsx
├── lib/
│   └── utils.ts         # cn() 工具函数
└── app/
    └── globals.css      # 全局样式
```

### utils.ts 工具

```typescript
// lib/utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

### 开发新组件模板

```tsx
import * as React from "react"
import { cn } from "@/lib/utils"

export interface CustomComponentProps
  extends React.HTMLAttributes<HTMLDivElement> {
  variant?: "default" | "outline"
}

const CustomComponent = React.forwardRef<
  HTMLDivElement,
  CustomComponentProps
>(({ className, variant = "default", ...props }, ref) => {
  return (
    <div
      ref={ref}
      className={cn(
        // 基础样式
        "rounded-lg border p-4",
        // 变体样式
        variant === "default" && "bg-white",
        variant === "outline" && "bg-transparent border-gray-300",
        // 自定义类名
        className
      )}
      {...props}
    />
  )
})
CustomComponent.displayName = "CustomComponent"

export { CustomComponent }
```

---

## AI 应用实战

### AI 聊天消息组件

```tsx
import * as React from "react"
import { cn } from "@/lib/utils"
import { Avatar, AvatarFallback, AvatarImage } from "@/components/ui/avatar"
import { Badge } from "@/components/ui/badge"

interface ChatMessageProps extends React.HTMLAttributes<HTMLDivElement> {
  role: "user" | "assistant"
  content: string
  timestamp?: Date
  isStreaming?: boolean
}

const ChatMessage = React.forwardRef<HTMLDivElement, ChatMessageProps>(
  ({ className, role, content, timestamp, isStreaming, ...props }, ref) => {
    return (
      <div
        ref={ref}
        className={cn(
          "flex gap-3 p-4",
          role === "user" && "flex-row-reverse",
          className
        )}
        {...props}
      >
        <Avatar className="h-8 w-8">
          <AvatarFallback>
            {role === "user" ? "U" : "AI"}
          </AvatarFallback>
          <AvatarImage
            src={role === "assistant" ? "/ai-avatar.png" : undefined}
          />
        </Avatar>

        <div
          className={cn(
            "flex flex-col gap-1 max-w-[80%]",
            role === "user" && "items-end"
          )}
        >
          <div
            className={cn(
              "rounded-2xl px-4 py-2",
              role === "user"
                ? "bg-primary text-primary-foreground"
                : "bg-muted"
            )}
          >
            <p className="whitespace-pre-wrap">{content}</p>
            {isStreaming && (
              <span className="inline-block w-2 h-4 ml-1 bg-current animate-pulse" />
            )}
          </div>

          {timestamp && (
            <span className="text-xs text-muted-foreground">
              {timestamp.toLocaleTimeString()}
            </span>
          )}
        </div>
      </div>
    )
  }
)
ChatMessage.displayName = "ChatMessage"

export { ChatMessage }
```

### 使用示例

```tsx
"use client"

import { useState, useRef, useEffect } from "react"
import { ChatMessage } from "@/components/custom/chat-message"
import { Button } from "@/components/ui/button"
import { Textarea } from "@/components/ui/textarea"
import { ScrollArea } from "@/components/ui/scroll-area"

export function AIChat() {
  const [messages, setMessages] = useState<Array<{
    id: string
    role: "user" | "assistant"
    content: string
    timestamp: Date
  }>>([])
  const [input, setInput] = useState("")
  const [isLoading, setIsLoading] = useState(false)
  const scrollRef = useRef<HTMLDivElement>(null)

  const sendMessage = async () => {
    if (!input.trim() || isLoading) return

    const userMessage = {
      id: crypto.randomUUID(),
      role: "user" as const,
      content: input,
      timestamp: new Date()
    }

    setMessages(prev => [...prev, userMessage])
    setInput("")
    setIsLoading(true)

    try {
      const response = await fetch("/api/chat", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ messages: [...messages, userMessage] })
      })

      const data = await response.json()

      setMessages(prev => [...prev, {
        id: crypto.randomUUID(),
        role: "assistant",
        content: data.content,
        timestamp: new Date()
      }])
    } finally {
      setIsLoading(false)
    }
  }

  return (
    <div className="flex flex-col h-screen">
      <ScrollArea ref={scrollRef} className="flex-1 p-4">
        {messages.map(message => (
          <ChatMessage
            key={message.id}
            role={message.role}
            content={message.content}
            timestamp={message.timestamp}
          />
        ))}
      </ScrollArea>

      <div className="p-4 border-t">
        <div className="flex gap-2">
          <Textarea
            value={input}
            onChange={(e) => setInput(e.target.value)}
            placeholder="Ask AI..."
            className="min-h-[44px] max-h-[200px]"
            onKeyDown={(e) => {
              if (e.key === "Enter" && !e.shiftKey) {
                e.preventDefault()
                sendMessage()
              }
            }}
          />
          <Button onClick={sendMessage} disabled={isLoading || !input.trim()}>
            Send
          </Button>
        </div>
      </div>
    </div>
  )
}
```

---

## 选型建议

### 何时选择 shadcn/ui

| 场景 | 推荐理由 |
|------|---------|
| **AI 辅助开发** | 代码透明，AI 可直接理解和修改 |
| **需要深度定制** | 组件源码在项目中，完全可控 |
| **设计系统开发** | 轻松定制设计令牌 |
| **Tailwind CSS 项目** | 与 Tailwind 完美集成 |
| **TypeScript 项目** | 原生类型安全 |
| **追求包体积** | 按需复制，无 npm 包体积影响 |

### 何时考虑传统 UI 库

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| **快速 MVP** | MUI / Ant Design | 组件丰富，开箱即用 |
| **Material Design** | MUI | 官方实现 |
| **企业后台** | Ant Design | 组件齐全 |
| **Vue 项目** | Vuetify / Naive UI | shadcn/ui 仅支持 React |

> [!TIP]
> shadcn/ui 在 AI 编程时代具有独特优势。组件代码的透明性和可修改性使其成为 AI 辅助开发的理想选择。

---

> [!SUCCESS]
> shadcn/ui 以"Copy to your project"的理念重新定义了前端组件库。它不是又一个 npm 包，而是组件代码的最佳实践集。配合 Radix UI 的无障碍支持、Tailwind CSS 的样式控制和 TypeScript 的类型安全，shadcn/ui 已成为 2026 年 React 项目 UI 开发的首选方案。

---

## 完整安装指南

### 环境要求与前置准备

**技术栈要求：**
- React 18.0+ 或 React 19.0+
- Next.js 14+ (App Router) 或 Vite
- TypeScript 5.0+
- Tailwind CSS 3.4+
- Node.js 18.0+

### 项目创建详细流程

**方式一：使用 create-next-app + shadcn/ui**

```bash
# 创建 Next.js 项目
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*" \
  --use-npm

# 进入项目目录
cd my-app

# 初始化 shadcn/ui
npx shadcn@latest init

# 选择配置选项（交互式）：
# ? Would you like to use default (1) or minimal setup?
#   > 1 (Default)
# ? Which style would you like to use?
#   > New York
# ? Which color would you like to use as base color?
#   > Slate
# ? Would you like to use CSS variables for theming?
#   > Yes
```

**方式二：非交互式初始化**

```bash
# 一键创建完整项目
npx create-next-app@latest my-app --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

cd my-app

# 使用选项初始化
npx shadcn@latest init -d

# 或者指定配置
npx shadcn@latest init \
  --style new-york \
  --base-color slate \
  --css-variables true \
  --tailwind-config tailwind.config.ts \
  --components-dir @/components \
  --lib-dir @/lib \
  --utils alias utils
```

**方式三：添加到现有项目**

```bash
# 进入现有项目
cd my-existing-app

# 初始化 shadcn/ui
npx shadcn@latest init

# 添加组件
npx shadcn@latest add button card dialog
```

### 组件安装

```bash
# 添加单个组件
npx shadcn@latest add button
npx shadcn@latest add dialog
npx shadcn@latest add form

# 批量添加
npx shadcn@latest add button card dialog input label textarea select checkbox radio-group switch slider

# 添加所有组件
npx shadcn@latest add --all

# 添加带配置的组件
npx shadcn@latest add button --yes  # 跳过确认
```

### 完整组件列表

| 组件 | 说明 | 安装命令 |
|------|------|---------|
| **基础** | | |
| `button` | 按钮 | `npx shadcn@latest add button` |
| `input` | 文本输入框 | `npx shadcn@latest add input` |
| `label` | 表单标签 | `npx shadcn@latest add label` |
| `textarea` | 多行文本 | `npx shadcn@latest add textarea` |
| `select` | 下拉选择 | `npx shadcn@latest add select` |
| `checkbox` | 复选框 | `npx shadcn@latest add checkbox` |
| `radio-group` | 单选组 | `npx shadcn@latest add radio-group` |
| `switch` | 开关 | `npx shadcn@latest add switch` |
| `slider` | 滑块 | `npx shadcn@latest add slider` |
| `toggle` | 切换 | `npx shadcn@latest add toggle` |
| **布局** | | |
| `card` | 卡片 | `npx shadcn@latest add card` |
| `sheet` | 侧边抽屉 | `npx shadcn@latest add sheet` |
| `dialog` | 对话框 | `npx shadcn@latest add dialog` |
| `drawer` | 移动端抽屉 | `npx shadcn@latest add drawer` |
| `popover` | 弹出框 | `npx shadcn@latest add popover` |
| `tooltip` | 工具提示 | `npx shadcn@latest add tooltip` |
| `tabs` | 标签页 | `npx shadcn@latest add tabs` |
| `accordion` | 手风琴 | `npx shadcn@latest add accordion` |
| `separator` | 分隔线 | `npx shadcn@latest add separator` |
| **反馈** | | |
| `alert` | 警告提示 | `npx shadcn@latest add alert` |
| `alert-dialog` | 警告对话框 | `npx shadcn@latest add alert-dialog` |
| `toast` | 轻提示 | `npx shadcn@latest add toast` |
| `progress` | 进度条 | `npx shadcn@latest add progress` |
| `skeleton` | 骨架屏 | `npx shadcn@latest add skeleton` |
| `spinner` | 加载动画 | `npx shadcn@latest add spinner` |
| `badge` | 徽章 | `npx shadcn@latest add badge` |
| **数据** | | |
| `table` | 表格 | `npx shadcn@latest add table` |
| `data-table` | 数据表格 | `npx shadcn@latest add data-table` |
| `avatar` | 头像 | `npx shadcn@latest add avatar` |
| `calendar` | 日历 | `npx shadcn@latest add calendar` |
| `chart` | 图表 | `npx shadcn@latest add chart` |
| **导航** | | |
| `navigation-menu` | 导航菜单 | `npx shadcn@latest add navigation-menu` |
| `breadcrumb` | 面包屑 | `npx shadcn@latest add breadcrumb` |
| `dropdown-menu` | 下拉菜单 | `npx shadcn@latest add dropdown-menu` |
| `context-menu` | 右键菜单 | `npx shadcn@latest add context-menu` |
| `menubar` | 菜单栏 | `npx shadcn@latest add menubar` |
| `scroll-area` | 滚动区域 | `npx shadcn@latest add scroll-area` |
| **表单** | | |
| `form` | 表单 | `npx shadcn@latest add form` |
| `input-otp` | OTP 输入 | `npx shadcn@latest add input-otp` |
| `combobox` | 组合框 | `npx shadcn@latest add combobox` |

### 配置详解

**components.json 配置：**

```json
{
  "$schema": "https://ui.shadcn.com/schema.json",
  "style": "new-york",
  "rsc": true,
  "tsx": true,
  "tailwind": {
    "config": "tailwind.config.ts",
    "css": "src/app/globals.css",
    "baseColor": "slate",
    "cssVariables": true,
    "prefix": ""
  },
  "aliases": {
    "components": "@/components",
    "utils": "@/lib/utils",
    "ui": "@/components/ui",
    "lib": "@/lib",
    "hooks": "@/hooks"
  },
  "iconLibrary": "lucide"
}
```

---

## 主题定制系统

### CSS 变量深入

**globals.css 完整配置：**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    /* === 颜色系统 === */
    /* 背景色 */
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;

    /* 卡片 */
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;

    /* 弹出层 */
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;

    /* 主色 */
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;

    /* 次要色 */
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;

    /* 强调色 */
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;

    /* 强调 */
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;

    /* 破坏性操作 */
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;

    /* 边框和输入 */
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;

    /* 圆角 */
    --radius: 0.5rem;

    /* 图表颜色 */
    --chart-1: 12 76% 61%;
    --chart-2: 173 58% 39%;
    --chart-3: 197 37% 24%;
    --chart-4: 43 74% 66%;
    --chart-5: 27 87% 67%;
  }

  .dark {
    /* 暗色模式变量 */
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;

    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;

    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;

    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;

    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;

    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;

    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;

    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;

    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### 自定义主题

**品牌主题示例：**

```css
@layer base {
  :root {
    /* 科技蓝主题 */
    --primary: 210 100% 50%;
    --primary-foreground: 0 0% 100%;
    --secondary: 210 40% 96%;
    --accent: 210 40% 96%;
    --destructive: 0 84% 60%;
    --border: 210 30% 90%;
    --ring: 210 100% 50%;
  }

  .dark {
    /* 暗色科技蓝 */
    --primary: 210 100% 65%;
    --primary-foreground: 0 0% 100%;
    --secondary: 215 28% 17%;
    --accent: 215 28% 17%;
    --destructive: 0 62% 40%;
    --border: 215 28% 25%;
    --ring: 210 100% 65%;
  }

  /* 渐变主题 */
  .gradient-theme {
    --primary: 260 80% 60%;
    --background: 260 60% 98%;
  }

  /* 高对比度主题 */
  .high-contrast {
    --foreground: 0 0% 0%;
    --background: 0 0% 100%;
    --border: 0 0% 0%;
  }
}
```

### 动态主题切换

```tsx
// components/theme-provider.tsx
'use client';

import * as React from 'react';
import { ThemeProvider as NextThemesProvider } from 'next-themes';

export function ThemeProvider({
  children,
  ...props
}: React.ComponentProps<typeof NextThemesProvider>) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}
```

```tsx
// app/layout.tsx
import { ThemeProvider } from '@/components/theme-provider';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="zh-CN" suppressHydrationWarning>
      <head />
      <body>
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
        </ThemeProvider>
      </body>
    </html>
  );
}
```

---

## 表单系统深入

### Zod 验证集成

```typescript
// lib/validations.ts
import { z } from 'zod';

export const LoginSchema = z.object({
  email: z
    .string()
    .min(1, { message: '邮箱不能为空' })
    .email({ message: '请输入有效的邮箱地址' }),
  password: z
    .string()
    .min(6, { message: '密码至少 6 个字符' })
    .max(100, { message: '密码过长' }),
});

export const RegisterSchema = z.object({
  name: z
    .string()
    .min(2, { message: '姓名至少 2 个字符' })
    .max(50, { message: '姓名过长' }),
  email: z
    .string()
    .min(1, { message: '邮箱不能为空' })
    .email({ message: '请输入有效的邮箱地址' }),
  password: z
    .string()
    .min(8, { message: '密码至少 8 个字符' })
    .regex(/[A-Z]/, { message: '密码必须包含大写字母' })
    .regex(/[a-z]/, { message: '密码必须包含小写字母' })
    .regex(/[0-9]/, { message: '密码必须包含数字' }),
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: '两次密码不一致',
  path: ['confirmPassword'],
});

export const PostSchema = z.object({
  title: z
    .string()
    .min(1, { message: '标题不能为空' })
    .max(200, { message: '标题过长' }),
  content: z
    .string()
    .min(10, { message: '内容至少 10 个字符' }),
  tags: z.array(z.string()).optional(),
  published: z.boolean().default(false),
});
```

### 完整表单示例

```tsx
// components/forms/register-form.tsx
'use client';

import * as React from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { useFormStatus } from 'react-dom';
import { useActionState } from 'react';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Checkbox } from '@/components/ui/checkbox';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { register, type ActionState } from '@/actions/register';
import { RegisterSchema } from '@/lib/validations';

export function RegisterForm() {
  const [state, action] = useActionState<ActionState, FormData>(register, null);

  const form = useForm<z.infer<typeof RegisterSchema>>({
    resolver: zodResolver(RegisterSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
      confirmPassword: '',
    },
  });

  async function onSubmit(values: z.infer<typeof RegisterSchema>) {
    const formData = new FormData();
    Object.entries(values).forEach(([key, value]) => {
      formData.append(key, value);
    });

    await action(formData);
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
        {state?.error && (
          <Alert variant="destructive">
            <AlertDescription>{state.error}</AlertDescription>
          </Alert>
        )}

        {state?.success && (
          <Alert className="border-green-500 text-green-600">
            <AlertDescription>注册成功！</AlertDescription>
          </Alert>
        )}

        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>姓名</FormLabel>
              <FormControl>
                <Input placeholder="张三" {...field} />
              </FormControl>
              <FormDescription>您的公开显示名称</FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>邮箱</FormLabel>
              <FormControl>
                <Input type="email" placeholder="zhangsan@example.com" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>密码</FormLabel>
              <FormControl>
                <Input type="password" placeholder="********" {...field} />
              </FormControl>
              <FormDescription>
                至少 8 个字符，包含大小写字母和数字
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="confirmPassword"
          render={({ field }) => (
            <FormItem>
              <FormLabel>确认密码</FormLabel>
              <FormControl>
                <Input type="password" placeholder="********" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <SubmitButton />
      </form>
    </Form>
  );
}

function SubmitButton() {
  const { pending } = useFormStatus();

  return (
    <Button type="submit" className="w-full" disabled={pending}>
      {pending ? '注册中...' : '注册'}
    </Button>
  );
}
```

### Server Action 表单验证

```typescript
// actions/register.ts
'use server';

import { z } from 'zod';
import { RegisterSchema } from '@/lib/validations';
import { db } from '@/lib/db';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export type ActionState = {
  errors?: {
    name?: string[];
    email?: string[];
    password?: string[];
    confirmPassword?: string[];
    _form?: string[];
  };
  error?: string;
  success?: boolean;
};

export async function register(
  prevState: ActionState,
  formData: FormData
): Promise<ActionState> {
  // 模拟网络延迟
  await new Promise((resolve) => setTimeout(resolve, 1000));

  const data = {
    name: formData.get('name'),
    email: formData.get('email'),
    password: formData.get('password'),
    confirmPassword: formData.get('confirmPassword'),
  };

  // 验证数据
  const validated = RegisterSchema.safeParse(data);

  if (!validated.success) {
    return {
      errors: validated.error.flatten().fieldErrors,
    };
  }

  // 检查邮箱是否已存在
  const existingUser = await db.user.findUnique({
    where: { email: validated.data.email },
  });

  if (existingUser) {
    return {
      errors: {
        email: ['该邮箱已被注册'],
      },
    };
  }

  // 创建用户
  const user = await db.user.create({
    data: {
      name: validated.data.name,
      email: validated.data.email,
      passwordHash: await hashPassword(validated.data.password),
    },
  });

  // 更新路径
  revalidatePath('/login');

  return {
    success: true,
  };
}
```

---

## 数据表格 (Data Table)

### 基础配置

```tsx
// components/data-table.tsx
'use client';

import * as React from 'react';
import {
  ColumnDef,
  ColumnFiltersState,
  SortingState,
  VisibilityState,
  flexRender,
  getCoreRowModel,
  getFilteredRowModel,
  getPaginationRowModel,
  getSortedRowModel,
  useReactTable,
} from '@tanstack/react-table';
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import {
  DropdownMenu,
  DropdownMenuCheckboxItem,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { ArrowUpDown, MoreHorizontal, Plus } from 'lucide-react';

interface DataTableProps<TData, TValue> {
  columns: ColumnDef<TData, TValue>[];
  data: TData[];
}

export function DataTable<TData, TValue>({
  columns,
  data,
}: DataTableProps<TData, TValue>) {
  const [sorting, setSorting] = React.useState<SortingState>([]);
  const [columnFilters, setColumnFilters] = React.useState<ColumnFiltersState>([]);
  const [columnVisibility, setColumnVisibility] = React.useState<VisibilityState>({});

  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getPaginationRowModel: getPaginationRowModel(),
    getSortedRowModel: getSortedRowModel(),
    getFilteredRowModel: getFilteredRowModel(),
    onSortingChange: setSorting,
    onColumnFiltersChange: setColumnFilters,
    onColumnVisibilityChange: setColumnVisibility,
    state: {
      sorting,
      columnFilters,
      columnVisibility,
    },
  });

  return (
    <div className="w-full">
      {/* 工具栏 */}
      <div className="flex items-center py-4 gap-4">
        <Input
          placeholder="搜索..."
          value={(table.getColumn('title')?.getFilterValue() as string) ?? ''}
          onChange={(event) =>
            table.getColumn('title')?.setFilterValue(event.target.value)
          }
          className="max-w-sm"
        />

        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="outline" className="ml-auto">
              列
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            {table
              .getAllColumns()
              .filter((column) => column.getCanHide())
              .map((column) => {
                return (
                  <DropdownMenuCheckboxItem
                    key={column.id}
                    className="capitalize"
                    checked={column.getIsVisible()}
                    onCheckedChange={(value) => column.toggleVisibility(!!value)}
                  >
                    {column.id}
                  </DropdownMenuCheckboxItem>
                );
              })}
          </DropdownMenuContent>
        </DropdownMenu>
      </div>

      {/* 表格 */}
      <div className="rounded-md border">
        <Table>
          <TableHeader>
            {table.getHeaderGroups().map((headerGroup) => (
              <TableRow key={headerGroup.id}>
                {headerGroup.headers.map((header) => {
                  return (
                    <TableHead key={header.id}>
                      {header.isPlaceholder
                        ? null
                        : flexRender(
                            header.column.columnDef.header,
                            header.getContext()
                          )}
                    </TableHead>
                  );
                })}
              </TableRow>
            ))}
          </TableHeader>
          <TableBody>
            {table.getRowModel().rows?.length ? (
              table.getRowModel().rows.map((row) => (
                <TableRow key={row.id} data-state={row.getIsSelected() && 'selected'}>
                  {row.getVisibleCells().map((cell) => (
                    <TableCell key={cell.id}>
                      {flexRender(cell.column.columnDef.cell, cell.getContext())}
                    </TableCell>
                  ))}
                </TableRow>
              ))
            ) : (
              <TableRow>
                <TableCell colSpan={columns.length} className="h-24 text-center">
                  无数据
                </TableCell>
              </TableRow>
            )}
          </TableBody>
        </Table>
      </div>

      {/* 分页 */}
      <div className="flex items-center justify-end space-x-2 py-4">
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.previousPage()}
          disabled={!table.getCanPreviousPage()}
        >
          上一页
        </Button>
        <Button
          variant="outline"
          size="sm"
          onClick={() => table.nextPage()}
          disabled={!table.getCanNextPage()}
        >
          下一页
        </Button>
      </div>
    </div>
  );
}
```

### 列定义

```tsx
// components/posts-columns.tsx
'use client';

import { ColumnDef } from '@tanstack/react-table';
import { MoreHorizontal, ArrowUpDown } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Checkbox } from '@/components/ui/checkbox';
import { Badge } from '@/components/ui/badge';

export type Post = {
  id: string;
  title: string;
  status: 'draft' | 'published' | 'archived';
  author: {
    name: string;
  };
  createdAt: string;
  updatedAt: string;
};

export const columns: ColumnDef<Post>[] = [
  {
    id: 'select',
    header: ({ table }) => (
      <Checkbox
        checked={table.getIsAllPageRowsSelected()}
        onCheckedChange={(value) => table.toggleAllPageRowsSelected(!!value)}
        aria-label="Select all"
      />
    ),
    cell: ({ row }) => (
      <Checkbox
        checked={row.getIsSelected()}
        onCheckedChange={(value) => row.toggleSelected(!!value)}
        aria-label="Select row"
      />
    ),
    enableSorting: false,
    enableHiding: false,
  },
  {
    accessorKey: 'title',
    header: ({ column }) => {
      return (
        <Button
          variant="ghost"
          onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}
        >
          标题
          <ArrowUpDown className="ml-2 h-4 w-4" />
        </Button>
      );
    },
    cell: ({ row }) => (
      <div className="font-medium">{row.getValue('title')}</div>
    ),
  },
  {
    accessorKey: 'status',
    header: '状态',
    cell: ({ row }) => {
      const status = row.getValue('status') as string;
      const variants: Record<string, 'default' | 'secondary' | 'destructive' | 'outline'> = {
        published: 'default',
        draft: 'secondary',
        archived: 'outline',
      };

      return (
        <Badge variant={variants[status] || 'secondary'}>
          {status === 'published' ? '已发布' : status === 'draft' ? '草稿' : '已归档'}
        </Badge>
      );
    },
  },
  {
    accessorKey: 'author.name',
    header: '作者',
    cell: ({ row }) => (
      <div className="text-muted-foreground">{row.original.author.name}</div>
    ),
  },
  {
    accessorKey: 'createdAt',
    header: ({ column }) => {
      return (
        <Button
          variant="ghost"
          onClick={() => column.toggleSorting(column.getIsSorted() === 'asc')}
        >
          创建时间
          <ArrowUpDown className="ml-2 h-4 w-4" />
        </Button>
      );
    },
    cell: ({ row }) => {
      const date = new Date(row.getValue('createdAt'));
      return (
        <div className="text-muted-foreground">
          {date.toLocaleDateString('zh-CN')}
        </div>
      );
    },
  },
  {
    id: 'actions',
    cell: ({ row }) => {
      const post = row.original;

      return (
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" className="h-8 w-8 p-0">
              <span className="sr-only">打开菜单</span>
              <MoreHorizontal className="h-4 w-4" />
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            <DropdownMenuLabel>操作</DropdownMenuLabel>
            <DropdownMenuItem
              onClick={() => navigator.clipboard.writeText(post.id)}
            >
              复制 ID
            </DropdownMenuItem>
            <DropdownMenuSeparator />
            <DropdownMenuItem asChild>
              <a href={`/posts/${post.id}/edit`}>编辑</a>
            </DropdownMenuItem>
            <DropdownMenuItem
              className="text-red-600"
              onClick={() => {
                if (confirm('确定要删除这篇文章吗？')) {
                  deletePost(post.id);
                }
              }}
            >
              删除
            </DropdownMenuItem>
          </DropdownMenuContent>
        </DropdownMenu>
      );
    },
  },
];
```

### 完整使用示例

```tsx
// app/posts/page.tsx
'use client';

import { useState, useEffect } from 'react';
import { DataTable } from '@/components/data-table';
import { columns } from './posts-columns';
import type { Post } from './posts-columns';

export default function PostsPage() {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchPosts() {
      try {
        const res = await fetch('/api/posts');
        const data = await res.json();
        setPosts(data);
      } catch (error) {
        console.error('Failed to fetch posts:', error);
      } finally {
        setLoading(false);
      }
    }

    fetchPosts();
  }, []);

  if (loading) {
    return <div className="flex h-screen items-center justify-center">加载中...</div>;
  }

  return (
    <div className="container mx-auto py-10">
      <div className="mb-6 flex items-center justify-between">
        <h1 className="text-3xl font-bold">文章管理</h1>
        <Button asChild>
          <a href="/posts/new">
            <Plus className="mr-2 h-4 w-4" />
            新建文章
          </a>
        </Button>
      </div>

      <DataTable columns={columns} data={posts} />
    </div>
  );
}
```

---

## 动画系统

### Tailwind Animate 集成

```bash
npm install tailwindcss-animate class-variance-authority clsx tailwind-merge
```

**tailwind.config.ts 配置：**

```typescript
import type { Config } from 'tailwindcss';

export default {
  darkMode: ['class'],
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: '0' },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: '0' },
        },
        'animate-in': {
          from: { opacity: '0' },
          to: { opacity: '1' },
        },
        'animate-out': {
          from: { opacity: '1' },
          to: { opacity: '0' },
        },
        'fade-in': {
          from: { opacity: '0' },
          to: { opacity: '1' },
        },
        'fade-out': {
          from: { opacity: '1' },
          to: { opacity: '0' },
        },
        'slide-in-from-top': {
          from: { transform: 'translateY(-100%)' },
          to: { transform: 'translateY(0)' },
        },
        'slide-in-from-bottom': {
          from: { transform: 'translateY(100%)' },
          to: { transform: 'translateY(0)' },
        },
        'slide-in-from-left': {
          from: { transform: 'translateX(-100%)' },
          to: { transform: 'translateX(0)' },
        },
        'slide-in-from-right': {
          from: { transform: 'translateX(100%)' },
          to: { transform: 'translateX(0)' },
        },
        'zoom-in': {
          from: { transform: 'scale(0.95)', opacity: '0' },
          to: { transform: 'scale(1)', opacity: '1' },
        },
        'zoom-out': {
          from: { transform: 'scale(1)', opacity: '1' },
          to: { transform: 'scale(0.95)', opacity: '0' },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
        'animate-in': 'animate-in 0.3s ease-out',
        'animate-out': 'animate-out 0.2s ease-in',
        'fade-in': 'fade-in 0.2s ease-out',
        'fade-out': 'fade-out 0.2s ease-in',
        'slide-in-from-top': 'slide-in-from-top 0.3s ease-out',
        'slide-in-from-bottom': 'slide-in-from-bottom 0.3s ease-out',
        'slide-in-from-left': 'slide-in-from-left 0.3s ease-out',
        'slide-in-from-right': 'slide-in-from-right 0.3s ease-out',
        'zoom-in': 'zoom-in 0.2s ease-out',
        'zoom-out': 'zoom-out 0.2s ease-in',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
} satisfies Config;
```

### 动画组件示例

```tsx
// components/ui/card.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';

const Card = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement>
>(({ className, ...props }, ref) => (
  <div
    ref={ref}
    className={cn(
      'rounded-lg border bg-card text-card-foreground shadow-sm',
      'transition-all duration-200',
      'hover:shadow-md hover:scale-[1.01]',
      className
    )}
    {...props}
  />
));
Card.displayName = 'Card';

const CardHeader = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement>
>(({ className, ...props }, ref) => (
  <div
    ref={ref}
    className={cn('flex flex-col space-y-1.5 p-6', className)}
    {...props}
  />
));
CardHeader.displayName = 'CardHeader';

const CardTitle = React.forwardRef<
  HTMLParagraphElement,
  React.HTMLAttributes<HTMLHeadingElement>
>(({ className, ...props }, ref) => (
  <h3
    ref={ref}
    className={cn(
      'text-2xl font-semibold leading-none tracking-tight',
      className
    )}
    {...props}
  />
));
CardTitle.displayName = 'CardTitle';

const CardContent = React.forwardRef<
  HTMLDivElement,
  React.HTMLAttributes<HTMLDivElement>
>(({ className, ...props }, ref) => (
  <div ref={ref} className={cn('p-6 pt-0', className)} {...props} />
));
CardContent.displayName = 'CardContent';

export { Card, CardHeader, CardTitle, CardContent };
```

### 页面转场动画

```tsx
// components/page-transition.tsx
'use client';

import { motion } from 'framer-motion';

interface PageTransitionProps {
  children: React.ReactNode;
}

export function PageTransition({ children }: PageTransitionProps) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -20 }}
      transition={{ duration: 0.2 }}
    >
      {children}
    </motion.div>
  );
}

// 使用
// app/about/page.tsx
import { PageTransition } from '@/components/page-transition';

export default function AboutPage() {
  return (
    <PageTransition>
      <div className="container">
        <h1>关于页面</h1>
        <p>带动画的内容</p>
      </div>
    </PageTransition>
  );
}
```

---

## 常见陷阱与解决方案

### 组件使用问题

**1. Dialog/Sheet 状态管理**

**问题：** Dialog 无法正确打开/关闭。

**解决方案：** 使用 controlled 模式或正确的 ref 管理。

```tsx
// ❌ 问题示例
function BadDialog() {
  return (
    <Dialog>
      <DialogTrigger>打开</DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>标题</DialogTitle>
        </DialogHeader>
        内容
      </DialogContent>
    </Dialog>
  );
}

// ✅ 正确示例
function GoodDialog() {
  const [open, setOpen] = useState(false);

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button onClick={() => setOpen(true)}>打开</Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>标题</DialogTitle>
        </DialogHeader>
        内容
        <DialogClose onClick={() => setOpen(false)} />
      </DialogContent>
    </Dialog>
  );
}
```

**2. Form 表单验证触发时机**

**问题：** 表单验证在提交时没有正确触发。

**解决方案：** 确保正确配置 `react-hook-form` 和 `zod`。

```tsx
// ✅ 正确配置
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
});

export function MyForm() {
  const form = useForm<z.infer<typeof schema>>({
    resolver: zodResolver(schema),
    mode: 'onBlur', // 或 'onChange'
  });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
      </form>
    </Form>
  );
}
```

### 样式问题

**1. 自定义样式优先级**

**问题：** 自定义样式被覆盖。

**解决方案：** 使用 `!` 或更具体的选择器。

```tsx
// ❌ 样式被覆盖
<div className="bg-red-500">
  <p className="bg-blue-500">文字</p> {/* 被外层影响 */}
</div>

// ✅ 使用 important 或 cn 合并
<div className="bg-red-500">
  <p className={cn('bg-blue-500', 'bg-blue-500')}>文字</p>
</div>

// ✅ 使用 !important（不推荐）
<p className="!bg-blue-500">文字</p>
```

**2. 暗色模式样式不一致**

**问题：** 暗色模式下颜色不正确。

**解决方案：** 使用 CSS 变量和 dark: 变体。

```css
/* globals.css */
@layer base {
  :root {
    --primary: 221.2 83.2% 53.3%;
  }

  .dark {
    --primary: 217.2 91.2% 59.8%;
  }
}

/* 组件中使用 */
.button {
  background-color: hsl(var(--primary));
}

.button:hover {
  background-color: hsl(var(--primary) / 0.9);
}
```

---

## 性能优化

### 组件懒加载

```tsx
// components/dynamic-dialog.tsx
'use client';

import dynamic from 'next/dynamic';

export const Dialog = dynamic(
  () => import('@/components/ui/dialog').then((mod) => mod.Dialog),
  {
    loading: () => <div className="h-10 w-10 animate-pulse rounded-md bg-muted" />,
    ssr: false,
  }
);

export const DialogTrigger = dynamic(
  () => import('@/components/ui/dialog').then((mod) => mod.DialogTrigger),
  { ssr: false }
);

export const DialogContent = dynamic(
  () => import('@/components/ui/dialog').then((mod) => mod.DialogContent),
  { ssr: false }
);
```

### 样式压缩

**使用 Tailwind CLI 生产构建：**

```bash
npm install -D @tailwindcss/forms autoprefixer postcss

# 添加到 postcss.config.js
export default {
  plugins: {
    '@tailwindcss/postcss': {},
    autoprefixer: {},
  },
};
```

```bash
# 生产构建优化 CSS
npx tailwindcss -o dist/styles.css --minify
```

---

## 附录：资源链接

### 官方资源

- [shadcn/ui 官网](https://ui.shadcn.com)
- [shadcn/ui GitHub](https://github.com/shadcn-ui/ui)
- [Radix UI](https://www.radix-ui.com)
- [Tailwind CSS](https://tailwindcss.com)

### 学习资源

- [shadcn/ui 文档](https://ui.shadcn.com/docs)
- [TanStack Table](https://tanstack.com/table)
- [React Hook Form](https://react-hook-form.com)
- [Zod](https://zod.dev)

### 相关工具

- [Lucide React](https://lucide.dev) - 图标库
- [class-variance-authority](https://cva.style) - 变体系统
- [clsx](https://github.com/lukeed/clsx) - 条件类名
- [tailwind-merge](https://github.com/dcastil/tailwind-merge) - Tailwind 类合并
