---
title: Cursor实战技巧与Agent-Mode指南
date: 2026-04-24
tags:
  - AI编程
  - Cursor
  - Agent Mode
  - Cursor Rules
  - 高级技巧
categories:
  - AI编程助手
  - 工具指南
---

> [!NOTE]
> 本文档专注于 Cursor 的高级功能，包括 Agent Mode 深度使用、Cursor Rules 配置、自定义快捷键和进阶 Prompt 技巧。

---

## Cursor Rules 完全指南

Cursor Rules 是 Cursor 独有的配置系统，允许开发者定义项目级别的 AI 行为规范。通过 Rules，团队可以统一代码风格和命名规范、强制执行安全最佳实践、指导 AI 理解业务逻辑、保持跨团队的一致性。

### Rules 存储位置

```
项目根目录/
├── .cursor/
│   └── rules/
│       ├── default.mdc      # 全局默认规则
│       ├── code-style.mdc   # 代码风格规则
│       ├── security.mdc     # 安全规则
│       └── architecture.mdc # 架构规则
```

你可以在 `.cursor/rules/` 目录下创建多个规则文件，Cursor 会自动加载所有 `.mdc` 文件。你可以根据需要将这些规则文件分成不同的类别，比如代码风格、安全规范、架构要求等。

### 规则文件格式

```markdown
# cursorules
# 描述：React 项目代码风格规范

## 代码风格
- 使用 2 空格缩进
- 组件文件使用 PascalCase（如 UserProfile.tsx）
- 工具函数使用 camelCase（如 formatDate.ts）
- 始终使用 TypeScript 的严格模式
- 优先使用函数组件和 Hooks

## 命名规范
- 变量和函数：camelCase
- 常量：UPPER_SNAKE_CASE
- 类型和接口：PascalCase，前缀 I（如 IUserProps）
- CSS 类名：kebab-case

## React 最佳实践
- 组件最大行数：200 行，超过则拆分
- Hooks 依赖数组必须完整，避免 ESLint 警告
- 使用 React.memo 优化重渲染
- 状态提升到最小必要层级

## TypeScript 规范
- 禁用 any，必须指定具体类型
- 接口和类型别名根据场景选择
- 使用可选链和空值合并运算符
```

> [!TIP]
> 规则文件的第一行必须是 `# cursorules`，这是 Cursor 识别规则文件的标识符。

### 内置 Rules 模板

| 模板名称 | 适用场景 | 核心内容 |
|---------|---------|---------|
| **React/Next.js** | React 项目 | Hooks 规范、组件拆分、性能优化 |
| **Vue/Nuxt** | Vue 项目 | Composition API、Pinia 状态管理 |
| **Python** | Python 项目 | PEP8、类型注解、Docstring 规范 |
| **TypeScript** | TS 项目 | 严格类型、接口定义、泛型使用 |
| **Security** | 全栈项目 | OWASP Top 10、输入验证、安全存储 |
| **Accessibility** | Web 项目 | WCAG 2.1、语义化标签、键盘导航 |

### Rules 编写技巧

#### 1. 使用示例而非描述

规则的编写方式会显著影响 AI 的遵循程度。相比抽象的描述，具体示例往往更有效。

```markdown
# ❌ 效果差
- 使用有意义的变量名

# ✅ 效果好
- 变量名必须使用完整的英文单词或公认缩写
- ✅ good: userName, totalCount, isActive
- ❌ bad: n, x, tmp, data, val
```

当你能给出具体的"好"和"坏"示例时，AI 能更准确地理解你的要求。这种方式特别适合描述代码风格类的规则。

#### 2. 明确禁止行为

明确告诉 AI 什么是不能做的，往往比只描述期望的行为更有效。

```markdown
# 禁止的代码模式
- 禁止使用 var，必须使用 const 或 let
- 禁止内联样式（除非是动态值）
- 禁止使用 any 类型
- 禁止 console.log 用于生产代码
```

#### 3. 优先级标记

为规则添加优先级标记能帮助 AI 在规则冲突时做出正确的选择。

```markdown
# @priority high
本规则必须严格遵守，违反将导致安全漏洞或严重错误。

# @priority medium
本规则应尽量遵守，特殊情况可酌情处理。

# @priority low
本规则为推荐做法，AI 可根据上下文灵活处理。
```

### 高级 Rules 配置

#### 框架特定规则

```markdown
# cursorules
# 描述：Next.js 14 App Router 项目规范

## 目录结构
- 使用 App Router 结构
- 页面组件放在 app/ 目录
- 组件放在 components/ 目录
- 工具函数放在 lib/ 或 utils/ 目录
- 类型定义放在 types/ 目录

## 文件命名
- 页面文件：page.tsx, layout.tsx, loading.tsx, error.tsx
- 组件文件：PascalCase.tsx
- 工具文件：camelCase.ts
- 样式文件：ComponentName.module.css

## 服务器组件 vs 客户端组件
- 默认使用服务器组件
- 仅在需要 "use client" 特性时标记客户端组件
- 客户端组件需要是异步的或需要事件处理器
- 避免不必要的客户端组件转换

## 数据获取
- 使用 async/await 进行服务端数据获取
- 使用 fetch 的缓存选项管理缓存策略
- 避免在客户端使用 useEffect 获取初始数据

## API 路由
- 使用 route.ts 文件定义 API 路由
- 分离 GET/POST/PUT/DELETE 处理器
- 使用 Zod 进行请求验证
```

#### 多语言项目规则

```markdown
# cursorules
# 描述：全栈 TypeScript 项目规范

## 后端（Express/NestJS）
### 目录结构
src/
├── controllers/    # 控制器
├── services/      # 业务逻辑
├── models/        # 数据模型
├── middleware/    # 中间件
├── routes/        # 路由定义
└── utils/         # 工具函数

### 命名规范
- 控制器：XxxController.ts
- 服务：XxxService.ts
- 模型：Xxx.model.ts
- 路由：xxx.routes.ts

### 错误处理
- 使用统一的错误响应格式
- 所有异步操作使用 try-catch
- 使用自定义错误类区分错误类型

## 前端（React）
### 目录结构
src/
├── components/    # UI 组件
├── pages/         # 页面组件
├── hooks/         # 自定义 Hooks
├── contexts/      # React Context
├── services/      # API 调用
└── utils/         # 工具函数

### 组件规范
- 组件最大行数：200 行
- Props 接口命名：XxxProps
- 使用函数组件和 Hooks
- 状态管理：根据复杂度选择 Context/Redux/Zustand

## 数据库（PostgreSQL）
### 命名规范
- 表名：snake_case，复数形式（users, posts, comments）
- 列名：snake_case
- 主键：id（使用 UUID）
- 外键：xxx_id 格式
- 时间戳：created_at, updated_at
```

---

## Agent Mode 深度使用

### Agent Mode 原理

Agent Mode 基于大语言模型的推理能力，通过"思考-规划-执行"的循环完成任务。这种设计让 AI 不仅能被动地回答问题，还能主动地规划和执行复杂的编程任务。

```
┌─────────────────────────────────────────────────────────────┐
│                      Agent Mode 执行循环                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐                                            │
│  │   思考阶段    │   AI 分析任务，理解用户意图                  │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │   规划阶段    │   制定执行计划，拆解子任务                    │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │   执行阶段    │   调用工具执行操作                          │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────┐                                            │
│  │   验证阶段    │   检查执行结果                              │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│    ┌────┴────┐                                              │
│    │ 循环终止  │                                              │
│    │ 或继续执行 │                                              │
│    └───────────┘                                              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

在思考阶段，AI 会分析用户的任务描述，理解用户的意图和期望结果。在规划阶段，AI 会制定详细的执行计划，将大任务拆解为可执行的小任务。在执行阶段，AI 会调用各种工具来完成任务。在验证阶段，AI 会检查执行结果是否满足要求。

### Agent Mode 工具调用

Agent Mode 可以调用多种工具完成复杂任务：

| 工具类别 | 工具名称 | 功能描述 |
|---------|---------|---------|
| **文件操作** | Read | 读取文件内容 |
| | Write | 创建或覆盖文件 |
| | Edit | 精确编辑文件内容 |
| | Delete | 删除文件或目录 |
| | Move | 移动或重命名文件 |
| | Mkdir | 创建目录 |
| **终端操作** | Terminal | 执行 Shell 命令 |
| | Script | 执行脚本文件 |
| **搜索操作** | Grep | 全文搜索 |
| | Glob | 文件模式匹配 |
| **Web 操作** | WebSearch | 搜索网页 |
| | WebFetch | 获取网页内容 |
| **Git 操作** | GitBranch | 创建分支 |
| | GitCommit | 提交更改 |
| | GitPush | 推送到远程 |

### Agent Mode 使用场景

#### 场景 1：项目初始化

```markdown
# 任务：创建一个完整的 React + TypeScript 项目

用户输入：
"创建一个新的 React 项目，使用 TypeScript、Vite、Tailwind CSS。
包含用户认证模块（登录、注册、找回密码），
使用 React Router 管理路由，
使用 React Query 管理服务器状态，
使用 Zustand 管理客户端状态，
配置好 ESLint 和 Prettier。"

# Agent 执行过程：

## 步骤 1：创建项目结构
mkdir my-project && cd my-project
npm create vite@latest . -- --template react-ts
npm install
npm install tailwindcss postcss autoprefixer
npx tailwindcss init -p
npm install react-router-dom @tanstack/react-query zustand
npm install -D eslint prettier eslint-plugin-react-hooks

## 步骤 2：配置文件
✓ vite.config.ts
✓ tsconfig.json
✓ tailwind.config.js
✓ postcss.config.js
✓ .eslintrc.cjs
✓ .prettierrc

## 步骤 3：创建目录结构
src/
├── components/
├── pages/
├── hooks/
├── stores/
├── services/
├── types/
└── utils/

## 步骤 4：实现认证模块
✓ src/pages/Login.tsx
✓ src/pages/Register.tsx
✓ src/pages/ForgotPassword.tsx
✓ src/services/authService.ts
✓ src/stores/authStore.ts
✓ src/hooks/useAuth.ts

## 步骤 5：配置路由
✓ src/App.tsx
✓ src/routes/ProtectedRoute.tsx

## 步骤 6：测试验证
✓ 运行开发服务器
✓ 访问页面验证功能

✅ 项目创建完成！
```

#### 场景 2：代码迁移

```markdown
# 任务：将 JavaScript 项目迁移到 TypeScript

用户输入：
"将 src/utils 目录下的所有 JavaScript 文件迁移到 TypeScript，
添加完整的类型定义，
确保类型安全，
保持原有功能不变。"

# Agent 执行过程：

## 步骤 1：分析现有文件
src/utils/
├── formatDate.js
├── validation.js
├── apiClient.js
└── storage.js

## 步骤 2：创建类型定义
✓ src/types/utils.ts

## 步骤 3：逐个迁移文件
✓ src/utils/formatDate.ts
✓ src/utils/validation.ts
✓ src/utils/apiClient.ts
✓ src/utils/storage.ts

## 步骤 4：验证迁移
运行 TypeScript 编译器检查类型错误
运行测试确保功能正常

✅ 迁移完成！
```

#### 场景 3：Bug 修复

```markdown
# 任务：修复用户登录失败的问题

用户输入：
"用户在登录时出现 'Invalid token' 错误，
但服务器日志显示 token 生成正常。
请定位问题并修复。"

# Agent 执行过程：

## 步骤 1：分析相关代码
- src/services/authService.ts
- src/middleware/auth.ts
- src/pages/Login.tsx

## 步骤 2：运行测试重现问题
执行登录流程，检查 token 传递

## 步骤 3：定位根因
发现：客户端存储 token 时使用了 localStorage.setItem
但读取时使用了 sessionStorage.getItem
导致 token 不匹配

## 步骤 4：修复问题
将 auth.ts 中的存储方式统一为 localStorage

## 步骤 5：验证修复
重新执行登录流程，验证问题已解决

✅ Bug 修复完成！
```

### Agent Mode 最佳实践

#### 1. 任务描述要清晰

模糊的描述会导致 Agent 做出不符合预期的事情。清晰的任务描述能显著提高 Agent 的执行效果。

```markdown
# ❌ 模糊的描述
"修一下那个 bug"

# ✅ 清晰的描述
"修复 src/api/user.ts 中 getUserById 函数的问题：
- 传入有效 ID 时返回 undefined
- 错误日志：[Error] Cannot read property 'name' of undefined
- 相关代码：
  const user = await getUserById(id);
  console.log(user.name);  // 这里报错"
```

#### 2. 提供足够的上下文

上下文信息越丰富，Agent 的理解就越准确，生成的结果就越符合预期。

```markdown
# 包含上下文的描述
"在 src/features/user 模块中添加用户头像上传功能：
- 使用 AWS S3 存储图片
- 支持 JPG、PNG 格式
- 最大文件大小 5MB
- 需要压缩图片到合理大小再上传
- 参考现有的 imageService.ts 实现"
```

#### 3. 分步骤处理复杂任务

对于复杂的大任务，分步骤执行能让你更好地控制过程，也能及时发现和纠正问题。

```markdown
# 分步骤执行
"这个重构任务较大，我们分步骤进行：

步骤 1：重构用户模型
步骤 2：迁移用户服务层
步骤 3：更新 API 路由
步骤 4：修改前端组件
步骤 5：添加测试

先从步骤 1 开始，完成后我再指示你继续下一步。"
```

### Agent Mode 安全设置

> [!WARNING]
> Agent Mode 可以执行文件操作和终端命令，请谨慎使用。建议在处理不熟悉的代码时设置更严格的确认模式。

```json
// .cursor/settings.json
{
  "cursor.agentMode": {
    "fileOperations": "confirm",    // 文件操作需要确认
    "terminalCommands": "confirm",  // 终端命令需要确认
    "deleteOperations": "always", // 删除操作始终确认
    "networkRequests": "confirm"   // 网络请求需要确认
  }
}
```

---

## 自定义配置与快捷键

### 基础配置

Cursor 的核心配置文件位于 `~/.cursor/` 目录：

```json
{
  "cursor": {
    "model": "sonnet",
    "temperature": 0.7,
    "maxTokens": 4096,
    "preamble": "你是一个专业的 TypeScript 开发工程师..."
  }
}
```

### 完整配置示例

```json
{
  "cursor.chat": {
    "model": "claude-opus-4.6",
    "temperature": 0.7,
    "maxTokens": 8192,
    "contextWindow": "fullProject"
  },
  "cursor.completion": {
    "model": "claude-sonnet-4.6",
    "inline": true,
    "tabComplete": true,
    "ghostText": true
  },
  "cursor.agent": {
    "model": "claude-opus-4.6",
    "autoApprove": false,
    "maxIterations": 50
  },
  "cursor.editor": {
    "formatOnSave": true,
    "autoClosingBrackets": true,
    "autoClosingQuotes": true,
    "tabSize": 2,
    "wordWrap": "on"
  }
}
```

### 常用快捷键

#### macOS 快捷键

| 功能 | 快捷键 | 说明 |
|------|--------|------|
| AI 对话 | `Cmd + K` | 唤起 AI 对话窗口 |
| 接受建议 | `Tab` | 接受代码补全建议 |
| 拒绝建议 | `Esc` | 拒绝代码补全建议 |
| 代码补全 | `Cmd + L` | 触发代码补全 |
| 解释代码 | `Cmd + Shift + L` | 解释选中的代码 |
| 修复代码 | `Cmd + Shift + E` | 修复代码错误 |
| Agent 模式 | `Cmd + Shift + G` | 启动 Agent Mode |
| 打开设置 | `Cmd + ,` | 打开设置面板 |
| 查找文件 | `Cmd + P` | 快速打开文件 |
| 全局搜索 | `Cmd + Shift + F` | 搜索所有文件 |
| 命令面板 | `Cmd + Shift + P` | 执行命令 |

#### Windows/Linux 快捷键

| 功能 | 快捷键 | 说明 |
|------|--------|------|
| AI 对话 | `Ctrl + K` | 唤起 AI 对话窗口 |
| 接受建议 | `Tab` | 接受代码补全建议 |
| 拒绝建议 | `Esc` | 拒绝代码补全建议 |
| 代码补全 | `Ctrl + L` | 触发代码补全 |
| 解释代码 | `Ctrl + Shift + L` | 解释选中的代码 |
| 修复代码 | `Ctrl + Shift + E` | 修复代码错误 |
| Agent 模式 | `Ctrl + Shift + G` | 启动 Agent Mode |
| 打开设置 | `Ctrl + ,` | 打开设置面板 |
| 查找文件 | `Ctrl + P` | 快速打开文件 |
| 全局搜索 | `Ctrl + Shift + F` | 搜索所有文件 |
| 命令面板 | `Ctrl + Shift + P` | 执行命令 |

### 自定义快捷键

在 `keybindings.json` 中添加：

```json
[
  {
    "key": "cmd+shift+h",
    "command": "cursor.chat",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "cursor.agent",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+c",
    "command": "cursor.quickChat",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "cursor.refactor",
    "when": "editorTextFocus"
  }
]
```

### 主题配置

Cursor 支持自定义主题，包括明暗主题切换、语法高亮配色、AI 对话界面主题和终端配色方案。

```json
// 主题配置
{
  "workbench.colorTheme": "One Dark Pro",
  "editor.tokenColorCustomizations": {
    "[One Dark Pro]": {
      "textMateRules": [
        {
          "scope": "comment",
          "settings": {
            "foreground": "#5c6370",
            "fontStyle": "italic"
          }
        }
      ]
    }
  }
}
```

### AI 特定配置

```json
{
  "cursor.ai": {
    "syntaxHighlighting": true,
    "inlineCompletions": true,
    "ghostText": true,
    "contextMenu": true,
    "hoverDescriptions": true
  },
  "cursor.chat": {
    "codeBlocks": {
      "syntaxHighlighting": true,
      "lineNumbers": true,
      "copyButton": true
    },
    "markdown": {
      "mathSupport": true,
      "diagramSupport": true
    }
  }
}
```

---

## 实战 Prompt 技巧

### 1. 基础代码生成

#### 生成函数

```markdown
请用 TypeScript 写一个防抖函数 debounce，要求：
- 接受两个参数：要防抖的函数和延迟毫秒数
- 返回一个新函数
- 支持 this 上下文绑定
- 包含完整的类型声明

```typescript
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout> | null = null;
  
  return function (this: any, ...args: Parameters<T>) {
    const context = this;
    
    if (timeoutId !== null) {
      clearTimeout(timeoutId);
    }
    
    timeoutId = setTimeout(() => {
      func.apply(context, args);
      timeoutId = null;
    }, wait);
  };
}
```
```

#### 生成组件

```markdown
请生成一个 React 用户卡片组件 UserCard：
- 接收 user 对象作为 props
- 显示头像、用户名、邮箱
- 使用 TypeScript + Tailwind CSS
- 包含加载状态和错误状态处理
- 组件命名为 PascalCase，文件命名为 kebab-case

```tsx
import React from 'react';

interface User {
  id: string;
  avatar: string;
  username: string;
  email: string;
  isLoading?: boolean;
  error?: string | null;
}

interface UserCardProps {
  user: User;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

export const UserCard: React.FC<UserCardProps> = ({
  user,
  onEdit,
  onDelete
}) => {
  if (user.isLoading) {
    return (
      <div className="bg-white rounded-lg shadow-md p-4 animate-pulse">
        <div className="flex items-center space-x-4">
          <div className="w-12 h-12 bg-gray-200 rounded-full" />
          <div className="flex-1 space-y-2">
            <div className="h-4 bg-gray-200 rounded w-3/4" />
            <div className="h-3 bg-gray-200 rounded w-1/2" />
          </div>
        </div>
      </div>
    );
  }

  if (user.error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-lg p-4">
        <p className="text-red-600 text-sm">{user.error}</p>
      </div>
    );
  }

  return (
    <div className="bg-white rounded-lg shadow-md p-4 hover:shadow-lg transition-shadow">
      <div className="flex items-center space-x-4">
        <img
          src={user.avatar}
          alt={`${user.username}'s avatar`}
          className="w-12 h-12 rounded-full object-cover"
        />
        <div className="flex-1">
          <h3 className="font-semibold text-gray-900">{user.username}</h3>
          <p className="text-sm text-gray-500">{user.email}</p>
        </div>
        <div className="flex space-x-2">
          {onEdit && (
            <button
              onClick={() => onEdit(user.id)}
              className="px-3 py-1 text-sm text-blue-600 hover:bg-blue-50 rounded"
            >
              Edit
            </button>
          )}
          {onDelete && (
            <button
              onClick={() => onDelete(user.id)}
              className="px-3 py-1 text-sm text-red-600 hover:bg-red-50 rounded"
            >
              Delete
            </button>
          )}
        </div>
      </div>
    </div>
  );
};

export default UserCard;
```
```

### 2. 代码重构

#### 重构提示模板

```markdown
请帮我重构以下代码，目标是：
1. 提高可读性（重命名变量、添加注释）
2. 提取重复逻辑为公共函数
3. 优化性能（减少不必要的重渲染）
4. 符合本项目的 Cursor Rules 规范

[粘贴代码]

## 重构要求
- 保持原有功能不变
- 添加必要的类型定义
- 遵循单职责原则
- 添加单元测试覆盖重构后的代码
```

#### 迁移提示模板

```markdown
请将以下 Vue 2 Options API 代码迁移到 Vue 3 Composition API：
- 使用 <script setup> 语法
- 将 data 转为 ref/reactive
- 将 computed 保持或改为 computed
- 将 methods 保持
- 将 watch 改为 watchEffect 或 watch
- 添加适当的 TypeScript 类型

[粘贴代码]
```

### 3. Bug 修复

#### 错误分析提示

```markdown
我遇到了一个 [错误类型] 错误：
错误信息：[粘贴错误日志]
- 错误类型：TypeError
- 错误位置：src/utils/format.ts:15
- 错误信息：Cannot read property 'map' of undefined

及相关代码：[粘贴相关代码]

请分析可能的原因，并提供修复方案。如果需要修改多个文件，请列出所有需要修改的文件。

## 分析要求
1. 列出可能的原因（按概率排序）
2. 提供每种原因的验证方法
3. 给出推荐的修复方案
4. 防止类似问题再次发生的建议
```

#### 测试驱动修复

```markdown
请为这个函数编写单元测试，然后根据测试结果修复 bug：

[粘贴函数代码]

测试要求：
- 使用 Vitest 框架
- 覆盖正常路径和边界情况
- 包含边界值测试
- 测试用例命名清晰

```typescript
import { describe, it, expect } from 'vitest';

describe('functionName', () => {
  it('should handle normal case', () => {
    // test implementation
  });
  
  it('should handle edge cases', () => {
    // test implementation
  });
});
```
```

### 4. 代码审查

#### 审查提示模板

```markdown
请审查以下代码，重点关注：
1. 潜在的 bug 和安全问题
2. 性能优化机会
3. 代码可读性和可维护性
4. 是否符合 [React/Vue/Angular] 最佳实践
5. TypeScript 类型是否完善

[粘贴代码]

请用以下格式输出：

## 代码审查报告

### 🔴 高危问题
1. [问题描述] - 位置：[文件:行号]
   - 影响：[说明危害]
   - 建议：[修复方案]

### 🟡 中危问题
1. [问题描述] - 位置：[文件:行号]
   - 影响：[说明危害]
   - 建议：[修复方案]

### 🟢 建议改进
1. [改进建议] - 位置：[文件:行号]
   - 原因：[说明理由]
   - 建议：[具体方案]

### 总体评分
- 可读性：[1-10]
- 性能：[1-10]
- 安全性：[1-10]
- 可维护性：[1-10]
```

### 5. 项目初始化

#### 创建项目结构

```markdown
请帮我创建一个 [React/Vue/Next.js/Nuxt] 项目的初始结构：
- 使用 TypeScript
- 使用 [Tailwind CSS / CSS Modules / Styled Components]
- 集成 ESLint 和 Prettier
- 配置好路径别名（如 @/ 表示 src/）
- 包含基础组件和工具函数目录
- 遵循本项目的 Cursor Rules

## 项目要求
- 遵循 Clean Architecture 原则
- 配置好环境变量加载
- 设置好开发/生产环境区分
- 配置好日志系统
- 添加 README 文档
```

### 6. 高级 Prompt 技巧

#### 1. 使用任务分解

对于复杂任务，将其分解为多个步骤能获得更好的结果。

```markdown
# 复杂任务分解
"帮我重构用户认证模块。这个任务较大，我们分步骤进行：

第一步：分析现有代码结构，列出所有需要修改的文件
第二步：设计新的架构方案
第三步：实施后端重构
第四步：实施前端重构
第五步：添加测试
第六步：验证功能

请先执行第一步。"
```

#### 2. 设置约束条件

明确的约束条件能帮助 AI 在正确的方向上工作。

```markdown
# 带约束的请求
"请实现一个排序算法，要求：
1. 时间复杂度不超过 O(n log n)
2. 空间复杂度不超过 O(1)
3. 必须是稳定的排序
4. 使用 TypeScript 实现
5. 添加完整的类型注释

请比较快速排序、归并排序和堆排序，选择最适合的方案。"
```

#### 3. 要求解释推理过程

对于复杂问题，先要求 AI 解释推理过程可以确保理解正确。

```markdown
# 请求推理过程
"请帮我分析这段代码的性能问题：

[粘贴代码]

请：
1. 解释代码的执行流程
2. 识别性能瓶颈
3. 分析时间复杂度和空间复杂度
4. 提供优化方案及预期效果
5. 给出优化前后的对比数据"
```

#### 4. 多轮迭代优化

通过多轮对话逐步优化结果，比一次性生成最佳结果更可靠。

```markdown
# 第一轮：基础实现
"请实现一个 React 计数器组件"

# 第二轮：增强功能
"很好，现在添加重置功能，并支持自定义步长"

# 第三轮：优化体验
"现在添加动画效果，使用 Framer Motion"
```

---

## Cursor AI 提示词工程深度指南

### 提示词基础原则

#### 1. 清晰的结构

好的提示词应该有清晰的结构，包括角色定义（可选）、上下文、任务和约束条件。

```markdown
# 高效提示词结构

## 角色定义（可选）
你是一个资深的前端工程师，专注于 React 和 TypeScript。

## 上下文
当前项目是一个电商平台，使用 Next.js 14 App Router。

## 任务
创建一个商品卡片组件。

## 约束条件
- 使用 TypeScript
- 使用 Tailwind CSS
- 包含图片、标题、价格、评分
- 支持加载和错误状态

## 输出要求
- 完整的组件代码
- Props 类型定义
- 使用示例
```

#### 2. 提供足够的上下文

上下文越多，AI 的回答就越准确。

```markdown
# ❌ 效果差的提示词
"创建用户组件"

# ✅ 效果好的提示词
"创建 src/components/UserCard.tsx 组件，要求：
- 接收 User 类型作为 props
- 显示用户名、邮箱、头像
- 包含编辑和删除按钮
- 参考 src/components/Button.tsx 的样式风格
- 使用项目的 design tokens（在 src/styles/tokens.css 中定义）
- 遵循项目的 Cursor Rules（.cursor/rules/default.mdc）"
```

#### 3. 指定输出格式

明确指定输出格式能获得更符合预期的结果。

```markdown
"创建一个防抖函数，要求：
- 使用 TypeScript
- 返回类型：(...args: T[]) => void
- 包含 JSDoc 注释
- 提供使用示例

输出格式：
1. 完整代码
2. 类型说明
3. 使用示例
4. 性能注意事项"
```

### 高级提示词技巧

#### 1. 链式思维提示

让 AI 逐步思考问题，提高回答质量。

```markdown
"请一步步分析这个问题：

1. 首先分析代码的执行流程
2. 识别可能的性能瓶颈
3. 列出每种优化方案
4. 比较各方案的优缺点
5. 给出推荐方案及实现代码"
```

#### 2. 约束条件提示

通过明确的约束条件指导 AI 的输出。

```markdown
"实现一个排序算法，要求：
1. 时间复杂度 O(n log n)
2. 空间复杂度 O(1)
3. 稳定排序
4. 使用 TypeScript
5. 添加完整类型注解
6. 包含边界测试"
```

#### 3. 示例驱动提示

提供参考示例能帮助 AI 理解你的期望。

```markdown
"参考以下实现方式，在 src/services 中创建用户服务：

参考实现：
```typescript
// src/services/baseService.ts
export class BaseService<T> {
  protected baseUrl: string;
  
  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }
  
  async findAll(): Promise<T[]> {
    const response = await fetch(this.baseUrl);
    return response.json();
  }
  
  async findById(id: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}/${id}`);
    return response.json();
  }
}
```

要求：
- 继承 BaseService
- 添加 User 特有的方法（findByEmail）
- 实现 CRUD 操作"
```

#### 4. 迭代优化提示

通过多轮对话逐步完善结果。

```markdown
# 第一轮：基础实现
"实现一个计数器组件"

# 第二轮：添加状态
"基于上面的组件，添加：
- 最大/最小值限制
- 步长支持
- 格式化显示"

# 第三轮：优化体验
"为组件添加：
- 动画效果（使用 Framer Motion）
- 键盘快捷键（+/- 键）
- 触控滑动支持"
```

---

## 企业级配置与团队协作

### Business 套餐核心功能

#### 1. 团队 Rules 共享

```markdown
# 团队 Rules 配置

1. 创建团队规则库
   .cursor/rules/
   ├── team-default.mdc      # 团队默认规范
   ├── team-security.mdc   # 安全规范
   ├── team-code-style.mdc  # 代码风格
   └── team-architecture.mdc # 架构规范

2. 规则继承
   - 团队规则 → 项目规则 → 个人规则
   - 优先级：个人 > 项目 > 团队
```

#### 2. 使用分析仪表板

```markdown
# 团队使用分析

仪表板提供：
├─ 活跃用户数
├─ 平均每日请求量
├─ 模型使用分布
├─ 高频功能排行
├─ 代码产出统计
└─ 效率提升指标
```

#### 3. 策略管理

```markdown
# 企业策略配置

1. 功能限制
   - 禁用特定模型
   - 限制文件操作权限
   - 控制 Agent 模式使用

2. 合规配置
   - 启用审计日志
   - 配置数据保留策略
   - 设置敏感词过滤

3. 成本控制
   - 设置月度预算上限
   - 部门成本分配
   - 超额预警
```

### 企业部署指南

#### 1. SSO 配置

```yaml
# SAML 2.0 配置示例
sso:
  provider: okta  # 或 azure-ad, google-workspace
  entity_id: "cursor-enterprise"
  sso_url: "https://company.okta.com/app/..."
  certificate: "/path/to/cert.pem"
  
  # 用户属性映射
  attribute_mapping:
    email: user.email
    name: user.firstName
    groups: user.groups
```

#### 2. 数据合规

```markdown
# GDPR 合规配置

1. 数据保留
   - 对话历史：30 天
   - 代码片段：90 天
   - 使用日志：1 年

2. 数据处理
   - 不使用客户代码训练模型
   - 数据加密传输和存储
   - 完整的删除权

3. 审计
   - 所有 API 调用记录
   - 用户操作审计日志
   - 合规报告生成
```

---

## 相关资源

- [[Cursor完全指南.md]] - Cursor 核心功能和选型建议
- [[Cursor配置指南与故障排除.md]] - 故障排除和常见问题
- [[GitHub-Copilot.md]] - GitHub Copilot 完整指南
- [[Windsurf.md]] - Windsurf AI 编程助手

