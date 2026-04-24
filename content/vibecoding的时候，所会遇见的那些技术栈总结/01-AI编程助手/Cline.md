# Cline - 开源 AI 编程助手权威指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Cline 的核心功能、开源特性、与竞品对比及配置指南。

---

## 目录

1. [[#Cline 概述与开源价值]]
2. [[#核心功能详解]]
3. [[#支持的模型]]
4. [[#安装与配置]]
5. [[#工具调用能力]]
6. [[#MCP 协议深度集成]]
7. [[#任务分解与执行]]
8. [[#自定义工具与工作流]]
9. [[#进阶配置与优化]]
10. [[#与 Cursor/Copilot 对比]]
11. [[#局限性分析]]
12. [[#选型建议]]
13. [[#实战技巧与最佳实践]]
14. [[#常见问题与故障排除]]
15. [[#参考资料]]

---

## Cline 概述与开源价值

### 产品背景

Cline（原名 Claude Dev，前身为 Claude CLI）是一款**完全开源**的 AI 编程助手，以 VS Code 扩展的形式提供。与商业产品不同，Cline 的代码完全开源，用户可以自由查看、修改和分发。

Cline 由独立开发者 **Sierra Huang**（网名 سعود）创建并维护，于 2023 年底首次发布。经过两年多的快速发展，Cline 已成为最受开发者欢迎的开源 AI 编程工具之一，在 GitHub 上获得了超过 **50,000 颗星**，成为开源 AI 编程工具领域的标杆项目。

### 开源核心价值

> [!IMPORTANT]
> Cline 的开源特性使其成为追求自主控制和安全性的开发者的首选。

| 价值维度 | 说明 |
|---------|------|
| **代码透明** | 100% 开源代码，可审计 |
| **数据自主** | 不强制云端处理，可完全本地运行 |
| **社区驱动** | 活跃的社区贡献，快速迭代 |
| **定制自由** | 可根据需求修改和扩展 |
| **无锁定** | 不依赖特定服务商 |
| **免费使用** | 无订阅费用，仅需 API 成本 |

### 开源生态优势

```markdown
# Cline 开源优势详解

┌─────────────────────────────────────────────────────────────┐
│                      开源透明度                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  代码可审计                                                   │
│  ├─ 100% 源代码可见                                         │
│  ├─ 无隐藏行为或后门                                        │
│  ├─ 安全漏洞可快速发现                                      │
│  └─ 社区安全审查                                            │
│                                                              │
│  自主控制                                                    │
│  ├─ 可修改源代码                                            │
│  ├─ 可自托管关键组件                                        │
│  ├─ 可禁用遥测和跟踪                                        │
│  └─ 完全控制数据流                                          │
│                                                              │
│  社区力量                                                    │
│  ├─ 快速的功能迭代                                          │
│  ├─ 丰富的社区插件                                          │
│  ├─ 详尽的文档和教程                                        │
│  └─ 积极的问题响应                                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 与开源竞品对比

| 特性 | Cline | Codeium | Continue |
|------|-------|---------|----------|
| **许可证** | MIT | 专有+免费 | Apache 2.0 |
| **模型支持** | 多种 | Codeium 专属 | 多种 |
| **活跃度** | 非常活跃 | 活跃 | 活跃 |
| **VS Code** | ✅ | ✅ | ✅ |
| **JetBrains** | ❌ | ✅ | ✅ |
| **Vim/Emacs** | ❌ | ✅ | ✅ |
| **Star 数** | 50,000+ | 100,000+ | 15,000+ |
| **社区贡献** | 非常活跃 | 官方维护 | 活跃 |

### 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    VS Code                                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Cline Extension                             │   │
│  │                                                              │   │
│  │  ┌──────────────┐  ┌──────────────────────┐            │   │
│  │  │  Task Planner │  │   Tool Executor      │            │   │
│  │  │  (任务规划)   │  │   (工具执行)         │            │   │
│  │  └──────────────┘  └──────────────────────┘            │   │
│  │                                                              │   │
│  │  ┌──────────────┐  ┌──────────────────────┐            │   │
│  │  │  MCP Client  │  │   File System        │            │   │
│  │  │  (MCP客户端)  │  │   (文件系统)         │            │   │
│  │  └──────────────┘  └──────────────────────┘            │   │
│  │                                                              │   │
│  │  ┌──────────────┐  ┌──────────────────────┐            │   │
│  │  │  API Router  │  │   Context Manager    │            │   │
│  │  │  (API路由)   │  │   (上下文管理)        │            │   │
│  │  └──────────────┘  └──────────────────────┘            │   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              API Layer                                    │   │
│  │                                                              │   │
│  │  Anthropic / OpenAI / Google / DeepSeek / Ollama / LM Studio│   │
│  │                                                              │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心功能详解

### 1. 智能任务执行

Cline 的核心理念是"思考-规划-执行"的循环：

#### 任务循环

```
用户输入任务
    ↓
Cline 分析任务
    ↓
生成执行计划
    ↓
执行计划步骤
    ├── 创建/编辑文件
    ├── 运行终端命令
    ├── 浏览器操作
    └── 其他工具
    ↓
验证结果
    ↓
报告完成/请求确认
```

#### 任务类型支持

| 任务类型 | 示例 | 执行方式 |
|---------|------|---------|
| **代码生成** | "创建用户认证模块" | 自动生成文件 |
| **代码修改** | "重构 calculate 函数" | 编辑现有文件 |
| **Bug 修复** | "修复登录报错" | 分析+修复 |
| **测试生成** | "为 UserService 写测试" | 生成测试文件 |
| **重构** | "迁移到 TypeScript" | 批量文件操作 |
| **文档** | "生成 API 文档" | 输出 Markdown |
| **调试** | "分析这个错误" | 错误诊断 |
| **优化** | "优化查询性能" | 性能分析 |

### 2. 多轮对话

Cline 支持复杂的多轮对话：

```markdown
用户: 帮我创建一个 React 组件
Cline: 已创建 src/components/UserCard.tsx

用户: 添加 loading 状态
Cline: 已在组件中添加 loading 状态

用户: 现在添加错误处理
Cline: 已添加错误边界和错误状态

用户: 能否使用 React Query 来管理数据获取？
Cline: 了解，将重构为使用 React Query
```

### 3. 项目级理解

Cline 能够理解整个项目结构：

| 理解能力 | 说明 |
|---------|------|
| **目录结构** | 理解项目的文件和目录组织 |
| **依赖关系** | 解析 package.json、requirements.txt 等 |
| **导入关系** | 追踪模块间的 import/export |
| **框架规范** | 理解 React/Vue/Angular 等框架约定 |
| **类型系统** | TypeScript 类型和接口 |
| **测试框架** | 理解项目使用的测试工具 |

### 4. 自动文件操作

Cline 可以自动执行以下文件操作：

| 操作 | 命令示例 | 说明 |
|------|---------|------|
| 创建文件 | `Write: src/utils/format.ts` | 创建或覆盖文件 |
| 编辑文件 | `Edit: src/utils/format.ts` | 精确编辑部分内容 |
| 删除文件 | `Delete: src/temp.ts` | 删除文件或目录 |
| 重命名 | `Rename: old.ts → new.ts` | 移动或重命名 |
| 移动文件 | `Move: src/a.ts → lib/a.ts` | 改变文件位置 |
| 目录操作 | `Mkdir: src/components` | 创建目录 |

### 5. 终端集成

Cline 深度集成 VS Code 终端：

```bash
# Cline 可以执行任意终端命令
# 示例：安装依赖
> npm install

# 运行测试
> npm test

# 构建项目
> npm run build

# 运行脚本
> ./scripts/deploy.sh

# Git 操作
> git add . && git commit -m "feat: add user authentication"

# Docker 操作
> docker build -t myapp .
> docker run -p 3000:3000 myapp
```

### 6. Web 能力

| 工具 | 功能 | 示例 |
|------|------|------|
| **WebSearch** | 搜索网页获取信息 | `Search: "React best practices 2026"` |
| **WebFetch** | 获取网页内容 | `Fetch: https://docs.example.com` |
| **Browser** | 浏览器自动化 | 打开网页、点击、填写表单 |

```markdown
# Web 搜索示例
用户: 查找 React 18 的新特性
Cline: Searching the web for "React 18 new features 2026"...
Cline: 找到了以下 React 18 新特性：

1. Concurrent Features
   - useTransition
   - useDeferredValue
   - Suspense 改进

2. Automatic Batching
   - 自动批处理所有更新

3. New APIs
   - startTransition
   - useId
   - useSyncExternalStore
```

---

## 支持的模型

### 模型支持总览

| 提供商 | 模型 | API 方式 | 本地支持 | 推荐度 |
|--------|------|---------|---------|--------|
| **Anthropic** | Claude Opus/Sonnet/Haiku | 云端 API | ❌ | ⭐⭐⭐⭐⭐ |
| **OpenAI** | GPT-4.5/GPT-4o/GPT-3.5 | 云端 API | ❌ | ⭐⭐⭐⭐ |
| **Google** | Gemini 2.5 Pro/Flash | 云端 API | ❌ | ⭐⭐⭐ |
| **DeepSeek** | DeepSeek V3/R1 | 云端 API | ❌ | ⭐⭐⭐ |
| **Ollama** | Llama/Qwen/Mistral | 本地 | ✅ | ⭐⭐⭐⭐ |
| **LM Studio** | 多种 GGUF 模型 | 本地 | ✅ | ⭐⭐⭐⭐ |
| **SambaNova** | Llama/Gemma | 云端 | ❌ | ⭐⭐ |
| **AWS Bedrock** | Claude/GPT | AWS | ❌ | ⭐⭐⭐ |
| **Azure OpenAI** | GPT-4 | Azure | ❌ | ⭐⭐⭐ |

### Anthropic Claude 系列

| 模型 | 推荐场景 | API 成本 | 特点 |
|------|---------|---------|------|
| Claude Opus 4.6 | 复杂推理、架构设计 | 高 | 最强推理能力 |
| Claude Sonnet 4.6 | 日常代码生成（推荐） | 中 | 性价比最高 |
| Claude Haiku 4.5 | 快速任务、成本敏感 | 低 | 响应最快 |

```json
// Cline 配置示例 - Anthropic
{
  "cline": {
    "apiProvider": "anthropic",
    "anthropicApiKey": "sk-ant-xxxxxxxxxxxxxxxxxxxx",
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 8192,
    "temperature": 0.7
  }
}
```

### OpenAI GPT 系列

| 模型 | 推荐场景 | API 成本 | 特点 |
|------|---------|---------|------|
| GPT-4.5 | 复杂任务 | 高 | 最强通用能力 |
| GPT-4o | 均衡选择 | 中 | 性价比高 |
| GPT-4o-mini | 快速、成本敏感 | 低 | 轻量快速 |

```json
// Cline 配置示例 - OpenAI
{
  "cline": {
    "apiProvider": "openai",
    "openAiApiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx",
    "model": "gpt-4o",
    "maxTokens": 4096,
    "temperature": 0.7
  }
}
```

### 本地模型支持

> [!TIP]
> 使用 Ollama 或 LM Studio 运行本地模型，可以实现完全离线使用，且无 API 费用限制。

#### Ollama 配置

```json
// .vscode/settings.json
{
  "cline": {
    "apiProvider": "ollama",
    "ollamaApiBase": "http://localhost:11434/v1",
    "model": "llama3.2",
    "maxTokens": 4096,
    "temperature": 0.7
  }
}
```

```bash
# Ollama 常用命令

# 安装 Ollama
# macOS/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# Windows: 从 https://ollama.ai 下载安装包

# 拉取模型
ollama pull llama3.2        # 推荐：轻量高效
ollama pull qwen2.5:14b      # 中文支持好
ollama pull codellama:34b    # 代码能力强
ollama pull mistral:7b       # 通用能力强

# 查看已安装模型
ollama list

# 创建自定义模型
ollama create my-custom-model -f Modelfile

# 运行模型
ollama run llama3.2 "Hello, how are you?"

# API 服务
ollama serve  # 启动 API 服务器
```

#### LM Studio 配置

```json
// .vscode/settings.json
{
  "cline": {
    "apiProvider": "openai-compatible",
    "openAiApiBase": "http://localhost:1234/v1",
    "model": "lmstudio-community/Mistral-7B-Instruct-v0.3-GGUF",
    "maxTokens": 4096,
    "temperature": 0.7
  }
}
```

```bash
# LM Studio 常用操作

# 1. 从 https://lmstudio.ai 下载 LM Studio

# 2. 下载模型
# - 打开 LM Studio
# - 在搜索框中搜索模型
# - 下载喜欢的模型（如 Mistral-7B）

# 3. 启动本地服务器
# - 点击 "Local Server" 标签
# - 点击 "Start Server" 按钮
# - 默认地址：http://localhost:1234/v1

# 4. 在 Cline 中配置
{
  "apiProvider": "openai-compatible",
  "openAiApiBase": "http://localhost:1234/v1",
  "model": "你的模型名称"
}
```

### 模型选择建议

| 场景 | 推荐模型 | 原因 |
|------|---------|------|
| 日常代码生成 | Claude Sonnet 4.6 | 性价比最高 |
| 复杂架构设计 | Claude Opus 4.6 | 最强推理能力 |
| 快速简单任务 | Claude Haiku 4.5 | 响应最快 |
| 预算极其有限 | 本地 Ollama | 零 API 成本 |
| 隐私敏感 | 本地模型 | 数据不离开本地 |
| 中文项目 | Qwen 2.5 | 中文支持优秀 |

---

## 安装与配置

### 安装步骤

#### 1. VS Code 内安装

```bash
# 方法一：VS Code 扩展市场
1. 打开 VS Code
2. 按 Cmd/Ctrl + P 打开命令面板
3. 输入 "ext install saoudrizwan.claude-dev"
4. 点击安装
5. 重启 VS Code

# 方法二：VSIX 文件安装
# 下载最新 .vsix 文件
# https://github.com/cline/cline/releases

# 安装
code --install-extension cline-*.vsix

# 方法三：命令行安装
# macOS
brew install cline

# Linux
curl -L https://github.com/cline/cline/releases/latest/download/cline-linux-x64.vsix -o cline.vsix
code --install-extension cline.vsix
```

### 配置 API 密钥

#### Anthropic Claude

```json
// .vscode/settings.json
{
  "cline": {
    "apiProvider": "anthropic",
    "anthropicApiKey": "sk-ant-xxxxxxxxxxxxxxxxxxxx"
  }
}
```

#### OpenAI

```json
{
  "cline": {
    "apiProvider": "openai",
    "openAiApiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

#### Google Gemini

```json
{
  "cline": {
    "apiProvider": "gemini",
    "geminiApiKey": "AIzaSyxxxxxxxxxxxxxxxxx"
  }
}
```

#### DeepSeek

```json
{
  "cline": {
    "apiProvider": "deepseek",
    "deepseekApiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

> [!IMPORTANT]
> API Key 应妥善保管，切勿提交至代码仓库。建议：
> 1. 使用环境变量存储
> 2. 添加到 .gitignore
> 3. 使用 VS Code Secrets Storage

### 高级配置

#### 代理设置

```json
{
  "cline": {
    "proxy": {
      "enabled": true,
      "url": "http://proxy.example.com:8080",
      "auth": {
        "username": "user",
        "password": "pass"
      }
    }
  }
}
```

#### 请求限制

```json
{
  "cline": {
    "maxTokens": 8192,
    "temperature": 0.7,
    "requestTimeout": 120,
    "maxRetries": 3
  }
}
```

#### 系统提示自定义

```json
{
  "cline": {
    "systemPrompt": "你是一个专业的 React 开发工程师，擅长使用 TypeScript 和 Tailwind CSS。",
    "customInstructions": {
      "codeStyle": "遵循 Airbnb JavaScript Style Guide",
      "typescript": "使用严格模式，禁止 any 类型",
      "react": "优先使用函数组件和 Hooks"
    }
  }
}
```

---

## 工具调用能力

### 可用工具列表

| 工具 | 功能 | 权限要求 |
|------|------|---------|
| **Read** | 读取文件内容 | 自动 |
| **Write** | 创建/覆盖文件 | 自动 |
| **Edit** | 编辑文件内容 | 自动 |
| **Delete** | 删除文件 | 需要确认 |
| **Read Multiple** | 批量读取文件 | 自动 |
| **Glob** | 文件模式匹配 | 自动 |
| **Grep** | 全文搜索 | 自动 |
| **Terminal** | 执行终端命令 | 需要确认 |
| **WebSearch** | 搜索网页 | 需要确认 |
| **WebFetch** | 获取网页内容 | 需要确认 |
| **Browser** | 浏览器自动化 | 需要确认 |
| **UseMcpTool** | 调用 MCP 工具 | 需要确认 |

### 文件操作工具

#### Read 工具

```markdown
# 读取单个文件
Read: /path/to/file.txt

# 读取多个文件
Read Multiple:
- /path/to/file1.ts
- /path/to/file2.ts
- /path/to/file3.ts

# 读取文件特定行
Read: /path/to/file.ts
From line: 10
To line: 50
```

#### Write 工具

```markdown
Write: src/components/Button.tsx
---
import React from 'react';

interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  className?: string;
}

export const Button: React.FC<ButtonProps> = ({
  children,
  onClick,
  variant = 'primary',
  size = 'md',
  disabled = false,
  className = ''
}) => {
  const baseStyles = 'rounded font-medium transition-colors';
  
  const variantStyles = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700'
  };
  
  const sizeStyles = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };
  
  return (
    <button
      type="button"
      onClick={onClick}
      disabled={disabled}
      className={`
        ${baseStyles}
        ${variantStyles[variant]}
        ${sizeStyles[size]}
        ${disabled ? 'opacity-50 cursor-not-allowed' : ''}
        ${className}
      `}
    >
      {children}
    </button>
  );
};

export default Button;
---
```

#### Edit 工具

```markdown
Edit:
File: src/utils/format.ts
Old: return value.toLowerCase();
New: return value.trim().toLowerCase();

# 或使用更精确的编辑
Edit:
File: src/components/UserCard.tsx
Old:
function UserCard({ user }) {
  return (
    <div>
      <span>{user.name}</span>
    </div>
  );
}

New:
function UserCard({ user }) {
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

#### Delete 工具

```markdown
# 删除单个文件
Delete: src/temp/old-file.ts

# 删除目录
Delete: src/deprecated/

# 批量删除
Delete:
- src/temp/file1.ts
- src/temp/file2.ts
- src/temp/file3.ts
```

### 终端工具

#### 执行命令

```bash
# 单条命令
Terminal: npm install

# 多条命令（使用 && 连接）
Terminal: 
cd /path/to/project && npm install && npm run build

# 或使用换行
Terminal: 
cd /path/to/project
npm install
npm run build
npm test
```

> [!WARNING]
> 终端命令可能包含风险操作，Cline 会根据命令类型决定是否自动执行或请求确认。

#### 常用终端命令示例

```bash
# 包管理
Terminal: npm install react react-dom
Terminal: yarn add axios
Terminal: pnpm add lodash

# Git 操作
Terminal: git add . && git commit -m "feat: add new feature"
Terminal: git push origin main
Terminal: git checkout -b feature/new-feature

# 构建和测试
Terminal: npm run build
Terminal: npm test
Terminal: npm run lint

# Docker
Terminal: docker build -t myapp .
Terminal: docker-compose up -d
```

### 搜索工具

#### Glob 工具

```markdown
# 查找特定模式的文件
Glob: src/**/*.ts

# 查找测试文件
Glob: src/**/*.test.ts

# 查找所有 React 组件
Glob: src/**/*.tsx

# 查找配置文件
Glob: **/package.json
Glob: **/*.config.js
```

#### Grep 工具

```markdown
# 搜索关键词
Grep: "TODO"
In: src/

# 正则表达式搜索
Grep: "function \w+\("
In: src/
Regex: true

# 搜索并显示上下文
Grep: "useAuth"
In: src/
Context: 3

# 搜索多个目录
Grep: "authenticate"
In:
- src/
- tests/
- docs/
```

### 浏览器工具

```javascript
// 打开网页
Open: https://docs.example.com

// 搜索网页
Search: "React useEffect best practices 2026"

// 获取页面内容
Fetch: https://api.example.com/data

// 浏览器自动化
Open: https://github.com
Click: "Sign in" button
Fill: "username" with "myusername"
Fill: "password" with "mypassword"
Click: "Sign in" button
```

---

## MCP 协议深度集成

### 什么是 MCP

**Model Context Protocol (MCP)** 是一个开放协议，允许 AI 模型与外部工具和服务进行标准化交互。Cline 原生支持 MCP，可以连接各种外部工具扩展其能力。

### MCP 服务器配置

```json
// .vscode/settings.json
{
  "cline": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src"]
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      },
      "memory": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-memory"]
      }
    }
  }
}
```

### 常用 MCP 服务器

| 服务器 | 功能 | 安装命令 |
|--------|------|---------|
| **filesystem** | 文件系统操作 | `npx -y @modelcontextprotocol/server-filesystem <path>` |
| **github** | GitHub API 操作 | `npx -y @modelcontextprotocol/server-github` |
| **memory** | 持久化记忆 | `npx -y @modelcontextprotocol/server-memory` |
| **slack** | Slack 消息发送 | `npx -y @modelcontextprotocol/server-slack` |
| **puppeteer** | 浏览器自动化 | `npx -y @modelcontextprotocol/server-puppeteer` |

### 自定义 MCP 服务器

```typescript
// my-mcp-server/index.ts
import { McpServer } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';
import { z } from 'zod';

const server = new McpServer({
  name: 'my-custom-server',
  version: '1.0.0'
});

server.tool(
  'get-weather',
  'Get weather for a location',
  {
    location: z.string().describe('City name or coordinates')
  },
  async ({ location }) => {
    // 调用天气 API
    const weather = await fetchWeather(location);
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(weather)
      }]
    };
  }
);

const transport = new StdioServerTransport();
server.run(transport);
```

```json
// 配置自定义服务器
{
  "cline": {
    "mcpServers": {
      "my-weather": {
        "command": "node",
        "args": ["/path/to/my-mcp-server/dist/index.js"],
        "env": {
          "WEATHER_API_KEY": "your-api-key"
        }
      }
    }
  }
}
```

### MCP 工具使用示例

```markdown
# 使用 MCP GitHub 服务器

用户: 创建 GitHub issue

Cline:
UseMcpTool:
Server: github
Tool: create_issue
Arguments:
{
  "owner": "username",
  "repo": "my-project",
  "title": "Bug: Login not working",
  "body": "## Description\n\nLogin fails with error...",
  "labels": ["bug", "high-priority"]
}

# 使用 MCP Slack 服务器

用户: 发送 Slack 消息通知团队

Cline:
UseMcpTool:
Server: slack
Tool: send_message
Arguments:
{
  "channel": "#engineering",
  "text": "🚀 New version deployed to production!"
}
```

---

## 任务分解与执行

### 任务分解原理

Cline 使用"思考-规划-执行"循环处理复杂任务：

```
┌─────────────────────────────────────────────────────────────┐
│                    任务处理流程                                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 任务理解                                                  │
│     ↓                                                        │
│     AI 分析用户输入                                           │
│     提取关键信息和约束                                        │
│                                                              │
│  2. 计划生成                                                  │
│     ↓                                                        │
│     将任务分解为可执行步骤                                     │
│     确定工具调用顺序                                           │
│     预估风险和依赖                                             │
│                                                              │
│  3. 执行监控                                                  │
│     ↓                                                        │
│     按计划执行每个步骤                                         │
│     监控执行结果                                               │
│     处理错误和异常                                             │
│                                                              │
│  4. 结果验证                                                  │
│     ↓                                                        │
│     验证最终结果                                               │
│     提供完成报告                                               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 任务执行示例

#### 示例 1：创建完整模块

```markdown
用户: 使用 Express + TypeScript 创建一个用户管理模块，包含：
- 用户 CRUD API
- JWT 认证
- 输入验证
- 单元测试

Cline 分析任务...

## 执行计划

1. 创建项目结构和配置文件
   ├─ package.json
   ├─ tsconfig.json
   ├─ src/index.ts
   └─ src/config/

2. 实现数据模型
   ├─ src/models/User.ts
   └─ src/types/

3. 实现服务层
   ├─ src/services/userService.ts
   └─ src/services/authService.ts

4. 实现控制器
   ├─ src/controllers/userController.ts
   └─ src/controllers/authController.ts

5. 实现路由
   └─ src/routes/

6. 实现中间件
   ├─ src/middleware/auth.ts
   └─ src/middleware/validation.ts

7. 编写测试
   └─ src/__tests__/

开始执行...

## 步骤 1：创建项目结构

Write: package.json
---
{
  "name": "user-management-api",
  "version": "1.0.0",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.2",
    "bcrypt": "^5.1.1",
    "zod": "^3.22.4"
  }
}
---

Write: tsconfig.json
---
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true
  }
}
---

## 步骤 2：实现数据模型

Write: src/types/user.ts
---
export interface User {
  id: string;
  email: string;
  name: string;
  passwordHash: string;
  role: 'user' | 'admin';
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateUserDto {
  email: string;
  name: string;
  password: string;
}

export interface UpdateUserDto {
  name?: string;
  email?: string;
}
---

... (继续执行剩余步骤)

✅ 任务完成！
已创建 15 个文件
代码行数：850+
测试覆盖：75%
```

#### 示例 2：Bug 修复

```markdown
用户: 修复 src/utils/validator.ts 中的 email 验证问题

Cline 分析代码...

Read: src/utils/validator.ts
---
export function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}
---

Cline 分析问题：
├─ 当前正则无法处理 + 符号
├─ 无法处理国际化域名
└─ 缺少长度检查

## 修复方案

Edit:
File: src/utils/validator.ts
Old:
export function validateEmail(email: string): boolean {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

New:
export function validateEmail(email: string): boolean {
  // RFC 5322 简化正则
  const regex = /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/;
  
  // 基本长度检查
  if (email.length > 254 || email.length < 3) {
    return false;
  }
  
  return regex.test(email);
}

## 验证修复

Terminal: npm test -- validator.test.ts
✅ 所有测试通过

✅ Bug 修复完成！
```

### 任务执行模式

| 模式 | 说明 | 适用场景 |
|------|------|---------|
| **自动执行** | AI 自动执行所有操作 | 简单明确的任务 |
| **逐项确认** | 每个操作前请求确认 | 中等复杂度 |
| **完全手动** | AI 只提供建议 | 复杂或敏感任务 |

---

## 自定义工具与工作流

### 自定义提示模板

创建可复用的任务模板：

```markdown
# .cline/templates/react-component.md

## 模板名称
React 函数组件

## 描述
生成标准化的 React 函数组件，包含完整的类型定义和最佳实践

## 模板内容
请创建一个 React 函数组件，名为 {componentName}，满足以下要求：

### 文件结构
- 组件文件：src/components/{ComponentName}.tsx
- 测试文件：src/components/{ComponentName}.test.tsx
- 样式文件：src/components/{ComponentName}.module.css

### 代码要求
1. 使用 TypeScript，启用严格模式
2. 定义完整的 Props 接口
3. 使用 FC 或直接函数声明
4. 添加适当的 JSDoc 注释
5. 包含必要的状态管理（useState）
6. 添加副作用处理（useEffect）

### 样式要求
- 使用 CSS Modules
- 遵循 BEM 命名规范
- 支持主题变量

### 测试要求
- 使用 Vitest + React Testing Library
- 测试渲染和交互
- 覆盖率 > 80%
```

### 自定义系统提示

```markdown
# .cline/system-prompt.md

## 角色定义
你是一个经验丰富的前端架构师，专注于 React 和 TypeScript生态系统。

## 技术栈
- React 18+ (Hooks, Suspense)
- TypeScript 5.x (严格模式)
- 状态管理：Zustand / Redux Toolkit
- 样式方案：Tailwind CSS / CSS Modules
- 测试：Vitest + Testing Library
- 构建：Vite

## 代码规范
- 遵循 Airbnb JavaScript Style Guide
- 使用函数组件和 Hooks
- 所有组件必须有类型定义
- 禁止使用 any 类型
- 使用 ESLint + Prettier

## 组件规范
- 单一职责原则
- 组件最大行数：200 行
- 提取可复用逻辑到 hooks
- 使用 Composition 模式

## 性能优化
- 使用 React.memo 优化重渲染
- 使用 useMemo/useCallback
- 实现虚拟滚动优化长列表
- 代码分割和懒加载
```

### 工作流自动化

```json
// .cline/workflows/refactor-component.json
{
  "name": "组件重构工作流",
  "description": "自动化重构 React 组件的完整流程",
  "steps": [
    {
      "name": "分析组件",
      "action": "read",
      "target": "${componentPath}"
    },
    {
      "name": "识别问题",
      "action": "analyze",
      "focus": ["complexity", "props", "state"]
    },
    {
      "name": "生成重构计划",
      "action": "plan",
      "output": "重构计划"
    },
    {
      "name": "执行重构",
      "action": "implement",
      "autoConfirm": false
    },
    {
      "name": "生成测试",
      "action": "test",
      "framework": "vitest"
    },
    {
      "name": "验证结果",
      "action": "verify",
      "checks": ["lint", "typecheck", "test"]
    }
  ]
}
```

---

## 进阶配置与优化

### 性能优化配置

```json
{
  "cline": {
    "performance": {
      "maxConcurrentRequests": 3,
      "requestTimeout": 120,
      "cacheEnabled": true,
      "cacheSize": "100MB"
    }
  }
}
```

### 成本控制

```json
{
  "cline": {
    "costControl": {
      "enabled": true,
      "maxCostPerSession": 10.0,
      "maxCostPerDay": 50.0,
      "alertThreshold": 0.8,
      "modelFallback": {
        "enabled": true,
        "fallbackModel": "claude-haiku-4-20250514",
        "fallbackThreshold": 0.5
      }
    }
  }
}
```

### 安全配置

```json
{
  "cline": {
    "security": {
      "allowTerminalCommands": "ask",
      "allowFileOperations": "auto",
      "allowNetworkRequests": "ask",
      "allowDestructiveActions": "always",
      "allowedDirectories": ["/path/to/project"],
      "blockedPatterns": [
        "rm -rf /",
        "curl .* | sh",
        "wget .* | sh"
      ]
    }
  }
}
```

### 快捷键配置

```json
{
  "keybindings": [
    {
      "key": "cmd+shift+i",
      "command": "cline.start",
      "when": "editorTextFocus"
    },
    {
      "key": "cmd+shift+r",
      "command": "cline.refactor",
      "when": "editorTextFocus"
    },
    {
      "key": "cmd+shift+t",
      "command": "cline.test",
      "when": "editorTextFocus"
    }
  ]
}
```

---

## 与 Cursor/Copilot 对比

### 功能对比表

| 特性 | Cline | Cursor | GitHub Copilot |
|------|-------|--------|----------------|
| **许可证** | MIT（开源） | 专有 | 专有 |
| **价格** | $0 | $20/月 | $10/月 |
| **模型选择** | 多种可选 | 多种可选 | OpenAI 为主 |
| **本地部署** | ✅ 完全支持 | ❌ | ❌ |
| **VS Code 扩展** | ✅ | ❌ | ✅ |
| **Agent 模式** | ✅ | ✅ | ✅ |
| **多文件编辑** | ✅ | ✅ | ❌ |
| **终端集成** | ✅ | ✅ | CLI 单独 |
| **浏览器自动化** | ✅ | ❌ | ❌ |
| **MCP 支持** | ✅ | ✅ | ❌ |
| **代码审查** | ✅ | ✅ | ✅ |

### 成本对比

| 工具 | 直接成本 | 隐性成本 |
|------|---------|---------|
| **Cline** | $0（使用自己的 API） | 需要配置和维护 |
| **Cursor** | $20/月 | 较低 |
| **Copilot** | $10/月 | 较低 |

> [!TIP]
> Cline 本身免费，但需要自备 API 密钥。如果已有 API 密钥，Cline 的边际成本为零。

### 适用场景对比

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 完全离线使用 | **Cline** | 支持 Ollama 本地运行 |
| 隐私敏感项目 | **Cline** | 支持本地模型 |
| 预算有限 | **Cline** | 开源免费 |
| 即开即用 | **Cursor/Copilot** | 无需配置 |
| JetBrains IDE | **Copilot** | 原生 JetBrains 支持 |
| 复杂多文件任务 | **Cline/Cursor** | 两者都支持 |
| GitHub 集成 | **Copilot** | 深度集成 |
| 中文项目 | **Cline/Copilot** | Claude 中文支持好 |

### 优劣势对比

```
┌─────────────────────────────────────────────────────────────┐
│                    Cline vs Cursor vs Copilot                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Cline                                                        │
│  ├─ ✅ 完全开源，可审计                                       │
│  ├─ ✅ 支持本地模型，完全离线                                 │
│  ├─ ✅ 零订阅费用                                            │
│  ├─ ✅ 高度可定制                                            │
│  ├─ ❌ 需要手动配置                                          │
│  ├─ ❌ 无内置 UI 编辑器                                      │
│  └─ ❌ 社区支持相对有限                                      │
│                                                              │
│  Cursor                                                       │
│  ├─ ✅ 优秀的用户体验                                        │
│  ├─ ✅ 深度 AI 集成                                          │
│  ├─ ✅ Tab 补全功能                                          │
│  ├─ ❌ 价格较高 ($20/月)                                     │
│  ├─ ❌ 依赖云端                                              │
│  └─ ❌ 不支持本地模型                                        │
│                                                              │
│  Copilot                                                      │
│  ├─ ✅ 微软生态深度集成                                      │
│  ├─ ✅ 完善的团队功能                                        │
│  ├─ ✅ 多 IDE 支持                                          │
│  ├─ ❌ 主要依赖 OpenAI                                       │
│  ├─ ❌ 多文件编辑能力弱                                      │
│  └─ ❌ 上下文窗口有限                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 局限性分析

### 1. 使用门槛

| 局限点 | 说明 | 缓解方案 |
|-------|------|---------|
| **需要配置** | 需要手动配置 API | 参考官方文档 |
| **无 UI 引导** | 主要通过命令交互 | 熟悉命令语法 |
| **模型成本** | 需自备 API 密钥 | 使用免费额度或本地模型 |

### 2. 功能局限性

| 局限点 | 说明 |
|-------|------|
| **无原生 IDE** | 依赖 VS Code 扩展 |
| **VS Code 独占** | 不支持其他 IDE |
| **GUI 相对简单** | 功能优先于界面 |
| **社区支持** | 主要依赖 GitHub Issues |
| **无实时补全** | 需要主动对话触发 |

### 3. 安全考量

> [!NOTE]
> 使用 Cline 时，代码仍会发送到配置的 API 服务商处理。请确保使用可信的 API 服务商。

| 考量点 | 说明 |
|-------|------|
| **API 密钥安全** | 存储在 VS Code 配置中 |
| **代码隐私** | 取决于 API 服务商 |
| **本地模型** | 完全隐私保护 |
| **网络请求** | 需谨慎授权 |

---

## 选型建议

### 何时选择 Cline

| 场景 | 推荐程度 | 原因 |
|------|---------|------|
| 追求零成本 | ⭐⭐⭐⭐⭐ | 完全免费 |
| 隐私敏感项目 | ⭐⭐⭐⭐⭐ | 支持本地模型 |
| 完全离线使用 | ⭐⭐⭐⭐⭐ | Ollama 完美支持 |
| 开发者/极客 | ⭐⭐⭐⭐ | 可深度定制 |
| 技术能力强 | ⭐⭐⭐⭐ | 需要一定配置能力 |
| 喜欢开源文化 | ⭐⭐⭐⭐⭐ | MIT 许可证 |

### 何时选择其他工具

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 希望开箱即用 | Cursor | 无需配置 |
| 企业采购 | Copilot Business | SLA 支持 |
| 深度代码分析 | Claude Code | 最强代码理解 |
| 缺乏配置意愿 | Copilot | 傻瓜式 |

### 迁移指南

#### 从 Copilot 迁移

```bash
# 1. 安装 Cline
# VS Code 扩展市场搜索 "cline"

# 2. 配置 API 密钥
# 设置 → Extensions → Cline → API Provider

# 3. 禁用 Copilot（可选）
# 设置 → Extensions → Copilot → 禁用

# 4. 开始使用
# Cmd/Ctrl + Shift + I 打开 Cline
```

#### 从 Cursor 迁移

| 功能映射 | Cursor | Cline |
|---------|--------|-------|
| AI Chat | Cmd+K | Cmd+Shift+I |
| Agent Mode | Cmd+Shift+G | 直接输入任务 |
| Composer | 多文件编辑 | 任务规划 |

### 快速入门路径

1. **第一天**：安装 Cline，配置 API 密钥
2. **第一周**：完成几个小型任务，熟悉工具调用
3. **第二周**：尝试本地模型（Ollama）
4. **第三周**：自定义系统提示和工作流
5. **第四周**：配置 MCP 服务器扩展功能

---

## 实战技巧与最佳实践

### 1. 效率提升技巧

#### 任务描述技巧

```markdown
# ❌ 效果差的描述
"修 bug"

# ✅ 效果好的描述
"修复 src/api/user.ts 中 getUserById 函数的问题：
- 症状：传入有效 ID 返回 undefined
- 错误日志：TypeError: Cannot read property 'name' of undefined
- 相关代码：
  const user = await getUserById(id);
  console.log(user.name);
请定位问题并修复，同时更新相关测试。"

# ✅ 更详细的描述
"在 src/features/auth 模块中添加双因素认证（2FA）功能：
- 使用 TOTP 算法
- 支持 Google Authenticator
- 提供 QR 码生成
- 包含验证和禁用功能
参考现有 auth.service.ts 的实现风格。"
```

#### 上下文利用

```markdown
# 充分利用上下文

1. 引用相关文件
   "查看 src/users/model.ts 中的 User 类型定义，
    然后在 src/users/service.ts 中实现类似的 CRUD 操作"

2. 指定项目规范
   "在 src/components/ 目录下创建新组件，
    遵循本项目的组件规范（在 .cline/templates 中定义）"

3. 说明技术栈
   "使用 Prisma ORM 和 PostgreSQL 实现数据持久化"
```

### 2. 安全最佳实践

```markdown
# 使用 Cline 的安全准则

1. 敏感操作确认
   - 删除文件前确认
   - 运行危险命令前确认
   - 网络请求前确认

2. 敏感信息处理
   - 永远不要让 AI 处理密钥和密码
   - 使用环境变量而非硬编码
   - 敏感代码手动编写

3. 代码审查
   - AI 生成的代码需要人工审查
   - 重点检查安全相关代码
   - 运行安全扫描工具

4. 本地模型
   - 处理敏感代码时使用本地模型
   - 确保数据不离开本地
```

### 3. 调试技巧

```markdown
# 高效调试工作流

1. 错误分析
   "分析这个错误并定位问题代码：
   [粘贴错误日志]"

2. 修复验证
   "修复后运行测试验证：
   npm test -- user.test.ts"

3. 回归测试
   "确保修复不会影响其他功能：
   npm test"

4. 性能分析
   "分析这个函数的性能问题：
   [粘贴代码]"
```

---

## 常见问题与故障排除

### 安装问题

#### 问题 1：扩展安装失败

**解决方案**：
1. 确认 VS Code 版本 ≥ 1.75
2. 检查网络连接
3. 清除扩展缓存
4. 重启 VS Code

#### 问题 2：无法加载扩展

**解决方案**：
1. 检查 VS Code 控制台错误
2. 更新 VS Code 到最新版本
3. 禁用冲突扩展
4. 重置扩展状态

### 配置问题

#### 问题 3：API 密钥无效

**解决方案**：
1. 确认 API 密钥正确
2. 检查 API 额度是否充足
3. 确认 API 密钥权限
4. 尝试重新生成密钥

#### 问题 4：模型连接失败

**解决方案**：
1. 确认模型服务运行中
2. 检查端口配置
3. 确认防火墙设置
4. 查看日志排查问题

### 使用问题

#### 问题 5：任务执行卡住

**解决方案**：
1. 按 Esc 取消当前任务
2. 开始新的对话
3. 简化任务描述
4. 检查 API 连接

#### 问题 6：文件操作失败

**解决方案**：
1. 确认文件路径正确
2. 检查文件权限
3. 确认目录存在
4. 手动创建目录

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| GitHub | https://github.com/cline/cline |
| VS Code Market | https://marketplace.visualstudio.com/items?itemName=saoudrizwan.claude-dev |
| 文档 | https://github.com/cline/cline/blob/main/README.md |
| Changelog | https://github.com/cline/cline/releases |

### 社区资源

| 资源 | 说明 |
|------|------|
| Discord | 官方社区讨论 |
| GitHub Issues | Bug 反馈和功能请求 |
| Reddit | r/ClaudeDev |
| Twitter | 官方更新 |

### API 提供商

| 提供商 | 定价页面 |
|------|---------|
| Anthropic | https://www.anthropic.com/pricing |
| OpenAI | https://openai.com/api/pricing |
| Google | https://ai.google.dev/pricing |
| DeepSeek | https://api-docs.deepseek.com |

### 本地模型

| 工具 | 链接 |
|------|------|
| Ollama | https://ollama.com |
| LM Studio | https://lmstudio.ai |
| Ollama Library | https://ollama.com/library |

---

> [!SUCCESS]
> Cline 作为完全开源的 AI 编程助手，为追求自主控制和零成本的开发者提供了出色的选择。其对多种模型的支持（包括本地 Ollama）和丰富的工具调用能力，使其成为 2026 年最具性价比的 AI 编程工具。对于技术能力较强、注重隐私或有预算限制的开发者，Cline 是不可替代的首选方案。

---

## Cline 高级配置与自定义

### 1. 自定义工具开发

```typescript
// 自定义工具示例
// src/custom-tools/hello-world.ts

import { Tool } from '@cline/sdk';

export const helloWorldTool: Tool = {
  name: 'hello_world',
  description: 'Prints a greeting message',
  inputSchema: {
    type: 'object',
    properties: {
      name: {
        type: 'string',
        description: 'Name to greet',
      },
      language: {
        type: 'string',
        enum: ['en', 'zh', 'ja'],
        default: 'en',
        description: 'Language for greeting',
      },
    },
    required: ['name'],
  },
  
  async execute({ name, language = 'en' }) {
    const greetings: Record<string, string> = {
      en: `Hello, ${name}!`,
      zh: `你好，${name}！`,
      ja: `こんにちは、${name}さん！`,
    };
    
    return {
      content: [
        {
          type: 'text',
          text: greetings[language] || greetings.en,
        },
      ],
    };
  },
};

// 使用自定义工具
// {
  // "tools": {
    // "hello-world": "node ./dist/custom-tools/hello-world.js"
  // }
// }
```

### 2. 工作流自动化

```json
// .cline/workflows/feature-development.json
{
  "name": "Feature Development",
  "description": "Complete workflow for developing a new feature",
  "steps": [
    {
      "name": "create-branch",
      "type": "bash",
      "command": "git checkout -b feature/{feature-name}",
      "skip": {
        "condition": "branch-exists",
        "value": "feature/{feature-name}"
      }
    },
    {
      "name": "create-files",
      "type": "file",
      "files": [
        {
          "path": "src/features/{feature-name}/index.ts",
          "content": "// Feature exports\n"
        },
        {
          "path": "src/features/{feature-name}/types.ts",
          "content": "// Feature types\n"
        },
        {
          "path": "src/features/{feature-name}/service.ts",
          "content": "// Feature service\n"
        },
        {
          "path": "src/features/{feature-name}/__tests__/index.test.ts",
          "content": "// Feature tests\n"
        }
      ]
    },
    {
      "name": "implement-feature",
      "type": "prompt",
      "template": "Implement the {feature-name} feature with the following requirements:\n{requirements}\n\nFollow the project conventions and best practices."
    },
    {
      "name": "write-tests",
      "type": "prompt",
      "template": "Write comprehensive tests for the {feature-name} feature covering:\n- Happy path\n- Edge cases\n- Error handling"
    },
    {
      "name": "run-tests",
      "type": "bash",
      "command": "npm test -- --coverage src/features/{feature-name}"
    },
    {
      "name": "commit-changes",
      "type": "bash",
      "command": "git add . && git commit -m 'feat({feature-name}): {commit-message}'",
      "condition": "tests-pass"
    },
    {
      "name": "push-branch",
      "type": "bash",
      "command": "git push -u origin feature/{feature-name}"
    }
  ]
}
```

### 3. 代码模板系统

```typescript
// .cline/templates/component.ts
import { ComponentTemplate, FileTemplate } from '@cline/sdk';

export const reactComponentTemplate: ComponentTemplate = {
  name: 'React Component',
  description: 'Creates a new React functional component with TypeScript',
  prompts: [
    {
      name: 'componentName',
      type: 'input',
      message: 'Component name (PascalCase)',
      validate: (value: string) => /^[A-Z][a-zA-Z0-9]*$/.test(value),
    },
    {
      name: 'withStyles',
      type: 'confirm',
      message: 'Include CSS module?',
      default: true,
    },
    {
      name: 'withTest',
      type: 'confirm',
      message: 'Include test file?',
      default: true,
    },
  ],
  
  files: (answers) => [
    {
      path: `src/components/{componentName}.tsx`,
      content: `import React from 'react';
${answers.withStyles ? `import styles from './{componentName}.module.css';` : ''}

interface {componentName}Props {
  children?: React.ReactNode;
  className?: string;
}

export const {componentName}: React.FC<{componentName}Props> = ({
  children,
  className = '',
}) => {
  return (
    <div className={${ answers.withStyles ? `styles.{componentName.toLowerCase()} ${className}` : `className` }}>
      {children}
    </div>
  );
};

export default {componentName};
`,
    },
    ...(answers.withStyles ? [
      {
        path: `src/components/{componentName}.module.css`,
        content: `.{componentName.toLowerCase()} {\n  /* Component styles */\n}`,
      },
    ] : []),
    ...(answers.withTest ? [
      {
        path: `src/components/{componentName}.test.tsx`,
        content: `import { render, screen } from '@testing-library/react';
import {{componentName}} from './{componentName}';

describe('{componentName}', () => {
  it('renders children', () => {
    render(<{componentName}>Test Content</{componentName}>);
    expect(screen.getByText('Test Content')).toBeInTheDocument();
  });
});
`,
      },
    ] : []),
  ],
};
```

### 4. 高级调试配置

```json
// .cline/debug-config.json
{
  "debugging": {
    "enabled": true,
    "logLevel": "debug",
    "logFile": ".cline/logs/debug.log",
    "inspectTools": true,
    "inspectContext": true
  },
  "breakpoints": {
    "onError": true,
    "onWarning": false,
    "onToolCall": false,
    "onContextChange": true
  },
  "profiling": {
    "enabled": true,
    "reportPath": ".cline/logs/profile.json",
    "trackMemory": true,
    "trackTiming": true
  }
}
```

### 5. CI/CD 集成

```yaml
# .github/workflows/cline.yml
name: Cline Integration

on:
  push:
    branches: [main]
  pull_request:
    types: [opened, synchronize]

jobs:
  cline-review:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Cline
        run: npm install -g @cline/cli
      
      - name: Run Cline Code Review
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          cline review \
            --model claude-sonnet-4 \
            --output cline-review.md
      
      - name: Post Review Comment
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const review = fs.readFileSync('cline-review.md', 'utf-8');
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '## Cline AI Code Review\n\n' + review
            });
```

---

## Cline 高级使用案例

### 1. 大型代码库分析

```markdown
# 大型代码库分析工作流

用户: 分析整个项目的代码质量

Cline 分析：

## 项目概览
- 总文件数：1,234
- 代码行数：89,456
- 语言分布：
  - TypeScript: 67%
  - JavaScript: 18%
  - Python: 12%
  - Other: 3%

## 代码质量分析

### 复杂度评分
- 平均圈复杂度：4.2 (良好)
- 最大圈复杂度：23 (需要重构)
- 重复代码块：12 处

### 问题发现

#### 高优先级
1. src/services/auth.ts:234
   - 函数过长 (150 行)
   - 建议：拆分为多个函数

2. src/utils/parser.ts:56
   - 硬编码敏感信息
   - 建议：使用环境变量

#### 中优先级
1. src/components/*.tsx (多处)
   - 缺少 React.memo 优化
   - 建议：优化重渲染

2. src/hooks/*.ts (多处)
   - 缺少依赖数组检查
   - 建议：添加 eslint-plugin-react-hooks

## 重构建议

### 短期（1-2 周）
1. 修复安全漏洞
2. 优化高复杂度函数
3. 添加单元测试

### 长期（1 个月）
1. 架构重构
2. 性能优化
3. 文档完善
```

### 2. 渐进式重构

```markdown
# 重构大型模块

用户: 重构整个 src/features/user 模块

Cline 分析并执行：

## 重构计划

### 阶段 1：基础设施
[1/5] 创建类型定义
[2/5] 创建 DTO 验证
[3/5] 创建数据库 schema

### 阶段 2：服务层
[1/3] 重写 UserService
[2/3] 重写 AuthService
[3/3] 添加缓存层

### 阶段 3：控制器
[1/2] 重写 UserController
[2/2] 重写 AuthController

### 阶段 4：测试
[1/2] 编写集成测试
[2/2] 编写单元测试

### 阶段 5：迁移
[1/1] 迁移旧代码

## 执行进度

✅ 阶段 1 完成
✅ 阶段 2 完成
⏳ 阶段 3 进行中...
```

### 3. 自动化文档生成

```markdown
# 生成 API 文档

用户: 为整个 API 生成 Swagger 文档

Cline 执行：

## 生成文档

### src/routes/user.ts
```typescript
/**
 * @route GET /api/users
 * @description 获取用户列表
 * @access 管理员
 * @queryParam {number} page - 页码 (默认: 1)
 * @queryParam {number} limit - 每页数量 (默认: 20)
 * @response 200 - 用户列表
 * @response 401 - 未授权
 * @response 403 - 权限不足
 */
router.get('/', authMiddleware, adminOnly, async (req, res) => {
  // ...
});
```

### src/routes/auth.ts
```typescript
/**
 * @route POST /api/auth/register
 * @description 用户注册
 * @access 公开
 * @body {string} email - 邮箱
 * @body {string} password - 密码
 * @body {string} name - 用户名
 * @response 201 - 注册成功
 * @response 400 - 参数错误
 * @response 409 - 邮箱已存在
 */
router.post('/register', validate(registerSchema), async (req, res) => {
  // ...
});
```

✅ API 文档生成完成！
```

---

## Cline 完整安装与配置指南

### 系统要求

#### 环境要求

| 配置项 | 最低要求 | 推荐配置 |
|--------|----------|----------|
| 操作系统 | macOS 10.15 / Windows 10 / Linux Ubuntu 18.04 | macOS 12+ / Windows 11 |
| VS Code | 1.75.0+ | 最新版本 |
| 内存 | 4GB RAM | 8GB+ RAM |
| 网络 | 能访问 AI API | 稳定快速连接 |
| API Key | Anthropic/OpenAI 等 | 具备有效配额 |

### VS Code 安装

#### 方法一：扩展市场

```markdown
# 安装步骤

1. 打开 VS Code
2. 按 Cmd/Ctrl + P 打开命令面板
3. 输入以下命令：
   ext install saoudrizwan.claude-dev

4. 等待安装完成
5. 重启 VS Code
6. 按 Cmd/Ctrl + Shift + I 打开 Cline
```

#### 方法二：VSIX 文件

```bash
# 1. 下载最新版本
# 访问 https://github.com/cline/cline/releases

# 2. 安装 VSIX
code --install-extension cline-*.vsix

# 3. 重启 VS Code
```

#### 方法三：命令行安装

```bash
# macOS
brew install --cask cline

# Linux
curl -L https://github.com/cline/cline/releases/latest/download/cline-linux-x64.vsix -o cline.vsix
code --install-extension cline.vsix
```

### API Key 配置

#### Anthropic Claude API

```json
// .vscode/settings.json
{
  "cline": {
    "apiProvider": "anthropic",
    "anthropicApiKey": "sk-ant-xxxxxxxxxxxxxxxxxxxx"
  }
}
```

获取方式：
1. 访问 https://console.anthropic.com/
2. 注册账户并登录
3. 进入 API Keys 页面
4. 点击 "Create Key" 生成新的 API Key

#### OpenAI API

```json
{
  "cline": {
    "apiProvider": "openai",
    "openAiApiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

获取方式：
1. 访问 https://platform.openai.com/
2. 注册账户并登录
3. 进入 API Keys 页面
4. 创建新的 Secret Key

#### Google Gemini API

```json
{
  "cline": {
    "apiProvider": "gemini",
    "geminiApiKey": "AIzaSyxxxxxxxxxxxxxxxxx"
  }
}
```

#### DeepSeek API

```json
{
  "cline": {
    "apiProvider": "deepseek",
    "deepseekApiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

### Ollama 本地模型配置

```json
{
  "cline": {
    "apiProvider": "ollama",
    "ollamaApiBase": "http://localhost:11434/v1",
    "model": "llama3.2"
  }
}
```

Ollama 安装：
```bash
# macOS/Linux
curl -fsSL https://ollama.ai/install.sh | sh

# 拉取模型
ollama pull llama3.2

# 启动服务
ollama serve
```

### LM Studio 配置

```json
{
  "cline": {
    "apiProvider": "openai-compatible",
    "openAiApiBase": "http://localhost:1234/v1",
    "model": "mistral-7b-instruct"
  }
}
```

### 代理配置

```json
{
  "cline": {
    "proxy": {
      "enabled": true,
      "url": "http://proxy.example.com:8080",
      "bypass": [
        "localhost",
        "127.0.0.1"
      ]
    }
  }
}
```

---

## Cline 快捷键大全

### 默认快捷键

| 功能 | macOS | Windows/Linux | 说明 |
|------|--------|----------------|------|
| 打开 Cline | `Cmd + Shift + I` | `Ctrl + Shift + I` | 打开对话面板 |
| 新建任务 | `Cmd + Shift + R` | `Ctrl + Shift + R` | 开始新任务 |
| 接受建议 | `Tab` | `Tab` | 接受 AI 建议 |
| 拒绝建议 | `Esc` | `Esc` | 拒绝建议 |

### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+i",
    "command": "cline.start",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "cline.newTask",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+c",
    "command": "cline.togglePanel",
    "when": "editorTextFocus"
  }
]
```

---

## Cline MCP 协议深度集成

### MCP 协议概述

Model Context Protocol (MCP) 是一个开放标准，允许 AI 系统与外部工具进行标准化交互。

### MCP 服务器配置

```json
{
  "cline": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src"]
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        }
      },
      "memory": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-memory"]
      },
      "slack": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-slack"],
        "env": {
          "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
        }
      }
    }
  }
}
```

### 常用 MCP 服务器

#### 1. 文件系统服务器

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "allowedDirectories": ["${workspaceFolder}"]
    }
  }
}
```

功能：
- 读取指定目录中的文件
- 列出目录内容
- 创建新文件
- 修改现有文件

#### 2. GitHub 服务器

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

功能：
- 创建 Issue
- 管理 PR
- 查看仓库内容
- 提交代码

#### 3. Slack 服务器

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "xoxb-..."
      }
    }
  }
}
```

功能：
- 发送消息
- 创建频道
- 管理成员

#### 4. 数据库服务器

```json
{
  "mcpServers": {
    "database": {
      "command": "node",
      "args": ["./mcp-servers/database/index.js"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  }
}
```

### 自定义 MCP 服务器开发

#### 服务器模板

```typescript
// mcp-servers/custom-server/index.ts
import { McpServer } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';
import { z } from 'zod';

const server = new McpServer({
  name: 'Custom Server',
  version: '1.0.0',
});

// 添加工具
server.tool(
  'get-weather',
  '获取指定城市的天气信息',
  {
    location: z.string().describe('城市名称'),
  },
  async ({ location }) => {
    // 调用天气 API
    const weather = await fetch(
      `https://api.weather.example.com?city=${location}`
    );
    const data = await weather.json();
    
    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(data, null, 2),
        },
      ],
    };
  }
);

server.tool(
  'send-email',
  '发送邮件',
  {
    to: z.string().email(),
    subject: z.string(),
    body: z.string(),
  },
  async ({ to, subject, body }) => {
    // 发送邮件逻辑
    await emailService.send({ to, subject, body });
    
    return {
      content: [{ type: 'text', text: '邮件发送成功' }],
    };
  }
);

const transport = new StdioServerTransport();
server.run(transport);
```

#### 配置自定义服务器

```json
{
  "cline": {
    "mcpServers": {
      "custom": {
        "command": "node",
        "args": ["./mcp-servers/custom-server/dist/index.js"],
        "env": {
          "API_KEY": "your-api-key",
          "DATABASE_URL": "postgresql://..."
        }
      }
    }
  }
}
```

### MCP 工具使用示例

#### GitHub 集成

```markdown
# GitHub 操作示例

用户: 创建一个 GitHub issue

Cline:
Tool: use_mcp_tool
Server: github
Tool: create_issue
Arguments:
{
  "owner": "username",
  "repo": "my-project",
  "title": "Bug: 用户登录失败",
  "body": "## 问题描述\n\n用户在使用某些邮箱登录时失败...\n\n## 复现步骤\n\n1. 打开登录页面\n2. 输入邮箱\n3. 点击登录\n\n## 预期行为\n\n应该成功登录",
  "labels": ["bug", "high-priority"]
}

✅ Issue 创建成功！
```

#### Slack 集成

```markdown
# Slack 通知示例

用户: 通知团队新版本已部署

Cline:
Tool: use_mcp_tool
Server: slack
Tool: send_message
Arguments:
{
  "channel": "#engineering",
  "text": "🚀 新版本 v2.0.0 已部署到生产环境！\n\n变更内容：\n- 性能优化\n- Bug 修复\n- 新功能"
}

✅ 消息发送成功！
```

---

## Cline 自定义工具开发

### 工具开发基础

#### 工具定义结构

```typescript
interface Tool {
  name: string;          // 工具名称
  description: string;     // 工具描述
  inputSchema: Schema;      // 输入参数模式
  execute: Function;       // 执行函数
}
```

#### 工具执行流程

```
用户请求
    ↓
解析参数
    ↓
执行工具
    ↓
返回结果
    ↓
格式化输出
```

### 工具开发示例

#### 1. 数据库工具

```typescript
// tools/database.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export const databaseTool = {
  name: 'db-query',
  description: '执行数据库查询',
  inputSchema: {
    type: 'object',
    properties: {
      query: {
        type: 'string',
        description: 'SQL 查询语句'
      }
    },
    required: ['query']
  },
  
  async execute({ query }: { query: string }) {
    try {
      const result = await prisma.$queryRawUnsafe(query);
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(result, null, 2)
        }]
      };
    } catch (error) {
      return {
        content: [{
          type: 'text',
          text: `错误: ${error.message}`
        }],
        isError: true
      };
    }
  }
};
```

#### 2. API 请求工具

```typescript
// tools/api-request.ts
export const apiRequestTool = {
  name: 'http-request',
  description: '发送 HTTP 请求',
  inputSchema: {
    type: 'object',
    properties: {
      method: {
        type: 'string',
        enum: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
        description: 'HTTP 方法'
      },
      url: {
        type: 'string',
        description: '请求 URL'
      },
      headers: {
        type: 'object',
        description: '请求头',
        additionalProperties: { type: 'string' }
      },
      body: {
        type: 'object',
        description: '请求体'
      }
    },
    required: ['method', 'url']
  },
  
  async execute({ method, url, headers = {}, body }: any) {
    try {
      const response = await fetch(url, {
        method,
        headers: {
          'Content-Type': 'application/json',
          ...headers
        },
        body: body ? JSON.stringify(body) : undefined
      });
      
      const data = await response.json();
      
      return {
        content: [{
          type: 'text',
          text: JSON.stringify({
            status: response.status,
            headers: Object.fromEntries(response.headers.entries()),
            body: data
          }, null, 2)
        }]
      };
    } catch (error) {
      return {
        content: [{ type: 'text', text: `请求失败: ${error.message}` }],
        isError: true
      };
    }
  }
};
```

#### 3. 文件处理工具

```typescript
// tools/file-processor.ts
import * as fs from 'fs/promises';
import * as path from 'path';

export const fileProcessorTool = {
  name: 'process-files',
  description: '批量处理文件',
  inputSchema: {
    type: 'object',
    properties: {
      operation: {
        type: 'string',
        enum: ['copy', 'move', 'delete', 'rename'],
        description: '操作类型'
      },
      source: {
        type: 'string',
        description: '源文件路径'
      },
      destination: {
        type: 'string',
        description: '目标路径（用于 copy/move/rename）'
      }
    },
    required: ['operation', 'source']
  },
  
  async execute({ operation, source, destination }: any) {
    try {
      switch (operation) {
        case 'copy':
          await fs.copyFile(source, destination);
          return {
            content: [{ type: 'text', text: `文件已复制: ${source} → ${destination}` }]
          };
          
        case 'move':
          await fs.rename(source, destination);
          return {
            content: [{ type: 'text', text: `文件已移动: ${source} → ${destination}` }]
          };
          
        case 'delete':
          await fs.unlink(source);
          return {
            content: [{ type: 'text', text: `文件已删除: ${source}` }]
          };
          
        case 'rename':
          await fs.rename(source, destination);
          return {
            content: [{ type: 'text', text: `文件已重命名: ${source} → ${destination}` }]
          };
          
        default:
          return {
            content: [{ type: 'text', text: `未知操作: ${operation}` }],
            isError: true
          };
      }
    } catch (error) {
      return {
        content: [{ type: 'text', text: `操作失败: ${error.message}` }],
        isError: true
      };
    }
  }
};
```

### 工具注册

```typescript
// tools/index.ts
import { databaseTool } from './database';
import { apiRequestTool } from './api-request';
import { fileProcessorTool } from './file-processor';

export const tools = [
  databaseTool,
  apiRequestTool,
  fileProcessorTool
];

// 在 Cline 中注册工具
// settings.json
{
  "cline": {
    "customTools": "./tools/index.ts"
  }
}
```

---

## Cline 与主流工具深度对比

### 功能对比矩阵

| 功能 | Cline | Cursor | GitHub Copilot | Windsurf | Claude Code |
|------|--------|---------|----------------|----------|---------------|
| **许可证** | MIT | 专有 | 专有 | 专有 | 专有 |
| **价格** | 免费 | $20/月 | $10/月 | $10/月 | $25/月 |
| **代码生成** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **多文件编辑** | ✅ | ✅ | 有限 | ✅ | ✅ |
| **终端命令** | ✅ | ✅ | ❌ | ✅ | ✅ |
| **Web 搜索** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **本地模型** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **MCP 支持** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **Tab 补全** | ❌ | ✅ | ✅ | ✅ | ❌ |
| **VS Code** | ✅ | ❌ | ✅ | ✅ | ❌ |
| **JetBrains** | ✅ | ❌ | ✅ | ❌ | ❌ |
| **自有 IDE** | ❌ | ✅ | ❌ | ✅ | ✅ |

### 成本对比

```markdown
| 工具 | 订阅费 | API 成本 | 总成本 | 适合场景 |
|------|---------|---------|--------|----------|
| Cline | $0 | 自选 | 极低 | 预算有限开发者 |
| Cursor | $20/月 | 包含 | $20/月 | 追求效率 |
| Copilot | $10/月 | 包含 | $10/月 | GitHub 用户 |
| Windsurf | $10/月 | 包含 | $10/月 | VS Code 迁移 |
| Claude Code | $25/月 | 包含 | $25/月 | 深度分析 |

# API 成本参考（Cline 适用）

| 模型 | 场景 | 月均成本 |
|------|------|----------|
| Claude Sonnet 4.6 | 日常开发 | $20-50 |
| GPT-4o | 日常开发 | $15-40 |
| Ollama 本地 | 完全离线 | $0 |
```

### 场景化推荐

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 预算有限 | Cline | 完全免费，API 成本可控 |
| 追求零成本 | Cline + Ollama | 完全本地，零成本 |
| 隐私敏感 | Cline + Ollama | 数据不离开本地 |
| 开箱即用 | Cursor / Copilot | 无需配置 |
| VS Code 老用户 | Windsurf / Copilot | 无缝迁移 |
| JetBrains 用户 | Copilot | 原生支持 |

### 优劣势分析

#### Cline 优势

```markdown
✅ 完全开源，可审计代码
✅ 支持多种 AI 模型
✅ 本地模型，零 API 成本
✅ MCP 协议扩展性强
✅ 高度可定制
✅ VS Code + JetBrains 双支持
```

#### Cline 劣势

```markdown
❌ 需要手动配置 API
❌ 无 Tab 补全功能
❌ 社区支持相对有限
❌ 界面相对简陋
❌ 学习曲线略高
```

---

## Cline 常见问题与解决方案

### FAQ 1：API Key 无效

**症状**：提示 API Key 无效或已过期。

**解决方案**：
```markdown
1. 确认 API Key 正确
   - 检查是否有遗漏字符
   - 确认没有多余空格

2. 检查 API Key 状态
   - 登录 AI 服务商控制台
   - 确认 Key 状态为 Active

3. 检查 API 配额
   - 确认还有可用配额
   - 考虑升级套餐

4. 尝试重新生成 Key
   - 旧的 Key 可能已失效
   - 生成新的 Key 并更新配置
```

### FAQ 2：Ollama 连接失败

**症状**：无法连接到本地 Ollama 模型。

**解决方案**：
```markdown
1. 确认 Ollama 正在运行
   ```bash
   ollama serve
   ```

2. 检查端口
   - 默认端口：11434
   - 确认防火墙允许

3. 测试连接
   ```bash
   curl http://localhost:11434/api/tags
   ```

4. 更新配置
   ```json
   {
     "apiProvider": "ollama",
     "ollamaApiBase": "http://localhost:11434/v1",
     "model": "llama3.2"
   }
   ```
```

### FAQ 3：工具执行失败

**症状**：工具（如 Terminal、WebSearch）无法执行。

**解决方案**：
```markdown
1. 检查工具权限
   - Cline → 设置 → 工具权限
   - 启用所需工具

2. 确认目录访问
   - 检查工作目录权限
   - 确认文件路径正确

3. 查看错误日志
   - Cline 输出面板
   - 详细错误信息

4. 重新加载
   - Cmd/Ctrl + Shift + P
   - 重新加载窗口
```

### FAQ 4：上下文耗尽

**症状**：对话过程中上下文被重置。

**解决方案**：
```markdown
1. 开启新对话
   - 新对话有完整上下文

2. 精简任务
   - 将大任务拆分为小任务
   - 每次只处理一个文件

3. 优化提示词
   - 简洁明确的指令
   - 避免重复描述

4. 引用关键文件
   - 使用 @file: 引用
   - 只引用相关文件
```

### FAQ 5：响应速度慢

**症状**：Cline 回复很慢。

**解决方案**：
```markdown
1. 检查网络
   - 确认网络连接稳定
   - 检查代理设置

2. 选择更快的模型
   - Claude Haiku > Sonnet > Opus
   - GPT-4o-mini > GPT-4o

3. 减少请求量
   - 合并多个小请求
   - 使用批量操作

4. 使用本地模型
   - Ollama 模型响应更快
   - 无网络延迟
```

---

## Cline 选型建议

### 个人开发者

**推荐：Cline + Claude Sonnet**

```markdown
# 选择理由

1. 零订阅成本
   - 只付 API 费用
   - 按需使用

2. 灵活性高
   - 支持多种模型
   - 可切换不同场景

3. 完全可控
   - 开源代码
   - 自定义工具
```

### 技术团队

**推荐：Cline + 团队规范**

```markdown
# 团队配置

1. 统一 API 密钥
   - 使用团队 API Key
   - 控制成本分摊

2. 共享配置
   - 统一的 .cline 目录
   - 共享系统提示

3. 自定义工具
   - 团队专用工具
   - 标准化工作流
```

### 隐私敏感场景

**推荐：Cline + Ollama**

```markdown
# 完全本地方案

1. Ollama 部署
   ```bash
   ollama pull llama3.2
   ollama pull codellama
   ```

2. Cline 配置
   ```json
   {
     "apiProvider": "ollama",
     "ollamaApiBase": "http://localhost:11434/v1",
     "model": "llama3.2"
   }
   ```

3. 优势
   - 数据完全本地
   - 零 API 成本
   - 完全离线可用
```

### 企业用户

**推荐：Cline Enterprise 方案**

```markdown
# 企业考虑因素

1. 安全合规
   - 自托管方案
   - 代码不外传

2. 成本控制
   - API 成本可控
   - 无订阅费用

3. 定制开发
   - 根据需求定制
   - 集成内部工具
```

---

## Cline 高级应用案例

### 案例 1：自动化代码审查

```markdown
# Cline 代码审查流程

用户: 审查 src/auth 目录的代码

Cline 分析：

## 审查范围
- src/auth/login.ts
- src/auth/register.ts
- src/auth/password.ts
- src/auth/session.ts

## 审查结果

### 🔴 高危问题

1. 密码明文存储（auth/password.ts:45）
   - 直接存储用户密码
   - 建议：使用 bcrypt 加密

2. SQL 注入风险（auth/login.ts:23）
   - 使用字符串拼接
   - 建议：使用参数化查询

### 🟡 中危问题

1. 缺少速率限制（auth/login.ts）
   - 可能被暴力破解
   - 建议：添加限流中间件

2. Token 永不过期（auth/session.ts）
   - 安全风险
   - 建议：设置过期时间

### 🟢 建议改进

1. 统一错误处理
2. 添加审计日志
3. 完善单元测试

是否需要修复这些问题？
```

### 案例 2：自动化测试生成

```markdown
# Cline 测试生成

用户: 为 src/utils/validator.ts 生成单元测试

Cline 分析并生成测试：

```typescript
// src/utils/validator.test.ts

import { describe, it, expect } from 'vitest';
import {
  validateEmail,
  validatePassword,
  validateUsername
} from './validator';

describe('validator', () => {
  describe('validateEmail', () => {
    it('should validate correct email', () => {
      expect(validateEmail('test@example.com')).toBe(true);
    });

    it('should reject email without @', () => {
      expect(validateEmail('testexample.com')).toBe(false);
    });

    it('should reject email without domain', () => {
      expect(validateEmail('test@')).toBe(false);
    });

    it('should reject email with spaces', () => {
      expect(validateEmail('test @example.com')).toBe(false);
    });
  });

  describe('validatePassword', () => {
    it('should validate strong password', () => {
      expect(validatePassword('Str0ng@Pass')).toBe(true);
    });

    it('should reject short password', () => {
      expect(validatePassword('Ab1@')).toBe(false);
    });

    it('should reject password without uppercase', () => {
      expect(validatePassword('strong@pass1')).toBe(false);
    });
  });
});
```

✅ 测试文件已生成！
运行 `npm test` 验证？
```

### 案例 3：数据库迁移

```markdown
# Cline 数据库迁移

用户: 将 MySQL 迁移到 PostgreSQL

Cline 分析并执行：

## 迁移计划

### 阶段 1：数据库 Schema 转换

1. 分析 MySQL Schema
   - users 表
   - posts 表
   - comments 表

2. 转换为 PostgreSQL
   - SERIAL → BIGSERIAL
   - TINYINT → SMALLINT
   - DATETIME → TIMESTAMP

### 阶段 2：SQL 语句转换

需要修改的语句：
- AUTO_INCREMENT → SERIAL
- LIMIT offset, count → LIMIT count OFFSET offset
- IFNULL → COALESCE

### 阶段 3：数据迁移

1. 导出 MySQL 数据
2. 转换数据格式
3. 导入 PostgreSQL

### 阶段 4：应用适配

1. 更新 ORM 配置
2. 修改连接字符串
3. 测试所有查询

是否开始执行迁移？
```

---

## Cline 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 10.15+ / Windows 10+ / Linux |
| 内存 | 4GB RAM |
| 磁盘空间 | 200MB 可用空间 |
| 网络 | 稳定连接（使用云端模型）或离线（使用本地模型） |
| 基础 IDE | VS Code 1.70+ 或 JetBrains IDE 2023.1+ |

#### 使用本地模型的额外要求

| 配置项 | 推荐 |
|--------|------|
| 内存 | 16GB+ RAM（用于运行大模型） |
| 磁盘 | 10GB+（模型文件存储） |
| GPU | NVIDIA GPU（可选，但推荐） |

### 安装步骤详解

#### VS Code 安装

```bash
# 方法一：VS Code Marketplace
# 1. 打开 VS Code
# 2. 按 Cmd/Ctrl + P
# 3. 输入：ext install cabor.vscode-cline
# 4. 点击安装

# 方法二：命令行安装
code --install-extension cabor.vscode-cline

# 方法三：VSIX 文件安装
# 1. 下载 .vsix 文件
# 2. VS Code → 扩展 → ... → 从 VSIX 安装
```

#### JetBrains IDE 安装

```markdown
# 1. 打开 JetBrains IDE
# 2. Settings → Plugins → Marketplace
# 3. 搜索 "Cline"
# 4. 点击 Install
# 5. 重启 IDE
```

#### CLI 安装（独立版本）

```bash
# 使用 npm 全局安装
npm install -g @anthropic-ai/cline

# 或使用 npx 直接运行
npx @anthropic-ai/cline

# Homebrew 安装（macOS）
brew install cline
```

### 首次配置

#### 1. API Key 设置

```markdown
# Cline 支持多种 AI 提供商

## Anthropic (推荐 Claude)
1. 访问 https://console.anthropic.com/
2. 创建 API Key
3. 在 Cline 设置中填入

## OpenAI (GPT-4)
1. 访问 https://platform.openai.com/
2. 创建 API Key
3. 在 Cline 设置中配置

## Google (Gemini)
1. 访问 https://aistudio.google.com/
2. 获取 API Key
3. 配置到 Cline

## DeepSeek
1. 访问 https://platform.deepseek.com/
2. 创建 API Key
3. 用于成本优化场景
```

#### 2. 基本设置

```json
// .vscode/settings.json
{
  "cline": {
    "apiProvider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 8192,
    "temperature": 0.7,
    "autoApproval": "ask",
    "localModel": {
      "enabled": false,
      "provider": "ollama",
      "endpoint": "http://localhost:11434"
    }
  }
}
```

#### 3. MCP 服务器配置

```json
{
  "cline.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-fs"]
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-git"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"]
    }
  }
}
```

### Cline 快捷键大全

#### 核心快捷键

| 功能分类 | 功能 | macOS | Windows/Linux | 说明 |
|---------|------|-------|---------------|------|
| **Cline 面板** | 打开面板 | `Cmd/Ctrl + Shift + V` | 同 | 打开 Cline 侧边栏 |
| | 发送消息 | `Enter` | `Enter` | 发送当前输入 |
| | 新任务 | `Cmd/Ctrl + N` | `Ctrl + N` | 开始新任务 |
| | 停止生成 | `Esc` | `Esc` | 停止当前生成 |
| **工具调用** | 批准工具 | `Enter` | `Enter` | 批准工具执行 |
| | 拒绝工具 | `Cmd/Ctrl + D` | `Ctrl + D` | 拒绝工具执行 |
| | 全部批准 | `Cmd/Ctrl + Shift + A` | `Ctrl + Shift + A` | 批准所有待执行 |
| **任务管理** | 查看任务 | `Cmd/Ctrl + Shift + T` | `Ctrl + Shift + T` | 打开任务列表 |
| | 继续任务 | `Cmd/Ctrl + Shift + R` | `Ctrl + Shift + R` | 继续上次任务 |

#### 工具快捷操作

| 操作 | 快捷键 |
|------|--------|
| 读取文件 | `R` 或 `Read` |
| 写入文件 | `W` 或 `Write` |
| 编辑文件 | `E` 或 `Edit` |
| 删除文件 | `D` 或 `Delete` |
| 执行终端 | `B` 或 `Bash` |
| 搜索文件 | `G` 或 `Glob` |
| 搜索内容 | `Grep` |
| 浏览器 | `Browser` |

---

## Cline AI 提示词工程深度指南

### 任务描述格式

#### 1. 基础任务格式

```markdown
# 基础格式

任务：[具体要做什么]
文件：[相关文件路径]
约束：[额外约束条件]

示例：
任务：创建一个用户登录函数
文件：src/auth/login.ts
约束：使用 JWT，返回 token 和用户信息
```

#### 2. 高级任务格式

```markdown
# 高级格式

## 目标
[要实现的功能]

## 背景
[项目上下文]

## 文件结构
[相关文件]

## 约束
- [约束1]
- [约束2]

## 验证
[如何验证结果]

## 执行
[是否自动执行]
```

#### 3. 多步骤任务

```markdown
# 多步骤任务

## 步骤 1：分析
分析当前代码结构

## 步骤 2：设计
设计新的架构

## 步骤 3：实现
按模块实现功能

## 步骤 4：测试
编写并运行测试

## 步骤 5：文档
更新相关文档
```

### 工具使用技巧

#### 1. 精确文件操作

```markdown
# 使用 Read 工具
读取文件内容：Read /path/to/file

# 使用 Grep 搜索
搜索文本：Grep "pattern" /path/to/dir

# 使用 Glob 查找
查找文件：Glob "*.ts" /path/to/dir
```

#### 2. 批量操作

```markdown
# 批量创建文件
1. Write src/components/Button.tsx
2. Write src/components/Input.tsx
3. Write src/components/Form.tsx

# 批量修改
1. Grep "oldText" src/
2. Edit 所有匹配的文件
```

#### 3. 自定义工作流

```markdown
# 定义工作流
工作流 = [
  "读取 package.json 了解项目结构",
  "读取 src/index.tsx 了解入口",
  "创建 src/App.tsx",
  "创建 src/styles/App.css"
]
```

---

## Cline 与现代技术栈集成

### Next.js + TypeScript 集成

#### 项目初始化工作流

```markdown
# 初始化 Next.js 项目

任务：使用 Cline 初始化一个新的 Next.js 14 项目

步骤：
1. Bash: npx create-next-app@latest my-app --typescript --tailwind --eslint
2. Bash: cd my-app && npm install @prisma/client prisma
3. Bash: npx prisma init
4. Write prisma/schema.prisma（数据模型）
5. Write src/lib/prisma.ts（Prisma 客户端）
6. Edit .env 添加数据库 URL

执行此工作流
```

#### 生成的数据模型

**prisma/schema.prisma**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  passwordHash  String
  role          Role      @default(USER)
  emailVerified DateTime?
  image         String?
  
  accounts Account[]
  sessions Session[]
  posts    Post[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([email])
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String   @db.Text
  published Boolean  @default(false)
  authorId  String
  
  author   User     @relation(fields: [authorId], references: [id])
  categories Category[]
  tags      Tag[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([authorId])
}

model Category {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]
}

enum Role {
  USER
  ADMIN
}
```

### React + 状态管理集成

**stores/useAuthStore.ts**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  name: string;
  image?: string;
  role: 'USER' | 'ADMIN';
}

interface AuthState {
  user: User | null;
  isLoading: boolean;
  setUser: (user: User | null) => void;
  setLoading: (loading: boolean) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      isLoading: true,
      
      setUser: (user) => set({ user, isLoading: false }),
      setLoading: (isLoading) => set({ isLoading }),
      logout: () => set({ user: null }),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ user: state.user }),
    }
  )
);
```

---

## Cline 常见问题与解决方案

### FAQ 1：API Key 配置失败

**症状**：Cline 无法连接到 AI 服务。

**解决方案**：
```markdown
1. 检查 API Key 格式
   - 确保没有多余空格
   - 检查 Key 前缀（sk- 或 claude-）

2. 验证 API Key
   - 访问提供商的 API 测试页面
   - 确认 Key 有效

3. 检查网络
   - 确保能访问 API 端点
   - 检查代理设置

4. 查看日志
   - Cline 输出面板有详细日志
   - 查找错误信息
```

### FAQ 2：工具执行被阻止

**症状**：工具调用被安全提示阻止。

**解决方案**：
```markdown
1. 理解安全模式
   - Cline 默认需要确认工具执行
   - 这是为了防止意外操作

2. 配置自动批准
   - 设置 → Cline → Auto-Approve
   - 可选：ask / browser-asking / never

3. 逐个批准
   - 按 Enter 批准当前工具
   - 按 Esc 拒绝

4. 批量批准
   - Cmd/Ctrl + Shift + A 批准所有
```

### FAQ 3：本地模型无法连接

**症状**：Ollama 或 LM Studio 连接失败。

**解决方案**：
```markdown
1. 检查 Ollama 服务
   Bash: curl http://localhost:11434/api/tags

2. 确认模型已下载
   Bash: ollama list

3. 配置端点
   {
     "cline.localModel": {
       "provider": "ollama",
       "endpoint": "http://localhost:11434",
       "model": "llama3"
     }
   }

4. LM Studio 特殊配置
   - 启用 CORS
   - 启用 local server
```

### FAQ 4：上下文窗口耗尽

**症状**：长对话后响应变慢或失败。

**解决方案**：
```markdown
1. 清理上下文
   - 使用 /clear 或新对话

2. 减小请求
   - 只 @ 需要文件
   - 避免引用大文件

3. 使用压缩
   - 某些提供商支持上下文压缩

4. 分段处理
   - 将大任务拆分成小任务
```

### FAQ 5：MCP 服务器配置问题

**症状**：MCP 工具不可用。

**解决方案**：
```markdown
1. 检查 MCP 配置
   {
     "cline.mcpServers": {
       "server-name": {
         "command": "npx",
         "args": ["-y", "@scope/package"]
       }
     }
   }

2. 安装依赖
   Bash: npm install -g @anthropic/mcp-server-xxx

3. 重启 Cline
   - 保存配置后需要重启

4. 查看 MCP 日志
   - 输出面板有 MCP 相关日志
```

---

## Cline 与竞品深度对比

### 开源工具对比

| 功能 | Cline | Roo Code | Copilot | Cursor |
|------|-------|----------|----------|---------|
| **开源** | ✅ | ✅ | ❌ | ❌ |
| **本地模型** | ✅ | ✅ | ❌ | ❌ |
| **MCP 支持** | ✅ | ✅ | ❌ | 部分 |
| **多提供商** | ✅ | ✅ | ❌ | ❌ |
| **自动工具执行** | ✅ | ✅ | ❌ | ✅ |
| **VS Code 集成** | ✅ | ✅ | ✅ | ✅ |
| **活跃开发** | ✅ | ✅ | ✅ | ✅ |

### 成本对比

| 工具 | 基础设施成本 | API 成本 | 总成本 |
|------|-------------|---------|--------|
| **Cline + Claude** | $0 | 按量计费 | 可控 |
| **Cline + Ollama** | 本地 | $0 | $0 |
| **Copilot Pro** | $0 | $10/月 | $10/月 |
| **Cursor Pro** | $0 | $20/月 | $20/月 |

### 场景化推荐

| 场景 | 推荐工具 | 理由 |
|------|---------|------|
| 隐私敏感开发 | Cline + Ollama | 完全离线、零成本 |
| 预算有限 | Cline + DeepSeek | 成本极低 |
| 高质量代码 | Cline + Claude | 最强模型能力 |
| 快速原型 | Cline + GPT-4o | 速度最快 |
| VS Code 用户 | Cline/Roo Code | 原生支持 |
| JetBrains 用户 | Roo Code | JetBrains 原生 |

---

## Cline MCP 协议深度指南

### MCP 架构

```
┌─────────────────────────────────────────────────┐
│                  Cline Client                    │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌──────────────┐      ┌──────────────────┐    │
│  │   Task       │─────▶│  Tool Executor   │    │
│  │   Planner    │      └────────┬─────────┘    │
│  └──────────────┘               │              │
│         │                       │              │
│         ▼                       ▼              │
│  ┌──────────────┐      ┌──────────────────┐    │
│  │   Context    │      │   MCP Client     │    │
│  │   Manager    │      │   (Protocol)     │    │
│  └──────────────┘      └────────┬─────────┘    │
│                                  │              │
└──────────────────────────────────┼──────────────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼              ▼              ▼
             ┌───────────┐ ┌───────────┐ ┌───────────┐
             │  Filesystem │ │    Git    │ │  GitHub   │
             │   Server    │ │  Server   │ │  Server   │
             └───────────┘ └───────────┘ └───────────┘
```

### 常用 MCP 服务器

#### 1. 文件系统服务器

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-fs"],
      "env": {
        "allowedDirectory": "/path/to/project"
      }
    }
  }
}
```

#### 2. Git 服务器

```json
{
  "mcpServers": {
    "git": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-git"],
      "env": {
        "gitPath": "/usr/bin/git"
      }
    }
  }
}
```

#### 3. GitHub 服务器

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

### 自定义 MCP 服务器开发

**server/index.ts**

```typescript
import { McpServer } from '@anthropic-ai/mcp-server';
import { z } from 'zod';

const server = new McpServer({
  name: 'my-custom-server',
  version: '1.0.0',
});

server.tool(
  'get_weather',
  'Get current weather for a location',
  {
    location: z.string().describe('City name'),
    units: z.enum(['celsius', 'fahrenheit']).default('celsius'),
  },
  async ({ location, units }) => {
    const response = await fetch(
      `https://api.weather.com/v3/wx/conditions/current?location=${encodeURIComponent(location)}&units=${units}`
    );
    const data = await response.json();
    
    return {
      content: [
        {
          type: 'text',
          text: `Current weather in ${location}: ${data.temp}°${units === 'celsius' ? 'C' : 'F'}, ${data.condition}`,
        },
      ],
    };
  }
);

server.start();
```

---

**文档统计**：
- 撰写时间：2026年4月
- 最终行数：约 4000 行
- 代码示例：70+
- 配置示例：30+
- MCP 服务器：10+
- 自定义工具：8+
- FAQ 方案：8+

> [!SUCCESS]
> Cline 作为开源 AI 编程助手的代表，凭借其 MCP 协议支持、多模型灵活切换和完全本地运行的能力，为注重隐私和成本的开发者提供了优秀的解决方案。建议注重隐私的用户结合 Ollama 使用完全本地方案，注重成本的用户可选择 DeepSeek 等低成本 API。

---

## Cline 高级 AI 提示词工程

### 1. 提示词模板库

#### 代码生成模板

```markdown
# 代码生成提示词模板

## 基础模板
请用 [编程语言] 实现 [功能描述]。

## 详细模板

### 角色设定
你是一个 [技术领域] 专家，专注于 [具体技术]。

### 技术要求
- 语言：[语言版本]
- 框架：[框架名称和版本]
- 类型系统：[类型要求]
- 代码规范：[规范名称]

### 功能需求
1. [功能点1]
2. [功能点2]
3. [功能点3]

### 约束条件
- 性能要求：[如有]
- 安全性：[如有]
- 兼容性：[如有]
- 依赖限制：[如有]

### 输出要求
- 完整可运行的代码
- 类型定义（如适用）
- 单元测试
- 使用示例
- 错误处理
```

#### Bug 修复模板

```markdown
# Bug 修复提示词模板

## 问题描述
[详细描述问题现象]

## 错误信息
```
[错误堆栈跟踪]
```

## 相关代码
```[语言]
[相关代码片段]
```

## 环境信息
- 操作系统：[OS 版本]
- 运行时：[运行时版本]
- 依赖版本：[关键依赖版本]

## 复现步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 预期行为
[描述期望的正确行为]

## 已尝试的解决
1. [方案1] - 结果
2. [方案2] - 结果

## 修复要求
- 修复后不能破坏现有功能
- 添加适当的边界检查
- 包含回归测试
- 保持代码风格一致
```

#### 重构模板

```markdown
# 重构提示词模板

## 重构目标
[描述需要达成的重构目标]

## 当前代码
```[语言]
[待重构的代码]
```

## 问题分析
1. [问题1：代码过长/重复/难以理解等]
2. [问题2]
3. [问题3]

## 重构约束
- 保持 API 兼容性
- 不改变外部行为
- 测试覆盖率 ≥ 80%
- 遵循 [代码规范]

## 重构步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 验证要求
- 运行现有测试
- 性能基准测试
- 代码覆盖率报告
```

### 2. 提示词优化技巧

#### 上下文优化

```markdown
# 技巧1：精确的文件引用

## 模糊引用 ❌
"在用户模块中添加删除功能"

## 精确引用 ✅
"在 src/features/user/services/UserService.ts 的 UserService 类中，
添加 deleteUser(id: string): Promise<void> 方法。
参考同文件中的 getUserById 方法的实现模式。"

# 技巧2：提供完整的上下文

## 缺少上下文 ❌
"添加表单验证"

## 完整的上下文 ✅
"为 src/components/UserForm.tsx 中的 UserForm 组件添加表单验证。
当前字段：email, password, confirmPassword, name
验证规则：
- email: 有效邮箱格式
- password: 最少8字符，包含大小写和数字
- confirmPassword: 必须与 password 相同
- name: 2-50个字符
参考项目中的 validate.ts 工具函数。"
```

#### 约束条件技巧

```markdown
# 技巧3：明确的约束条件

## 缺少约束 ❌
"优化查询性能"

## 明确的约束 ✅
"优化 src/services/UserService.ts 中的 findAll 方法性能。

约束条件：
1. 响应时间必须 < 100ms
2. 不能改变现有 API 接口
3. 必须使用 Prisma ORM
4. 考虑添加 Redis 缓存
5. 保持与现有代码风格一致

当前问题：
- N+1 查询问题
- 缺少索引
- 没有缓存"
```

#### 迭代开发技巧

```markdown
# 技巧4：分步骤迭代

## 第一轮：基础实现
"创建一个基础的 Todo 组件，包含：
- 添加任务
- 删除任务
- 标记完成
不需要持久化，状态存储在组件内。"

## 第二轮：添加功能
"基于上面的 Todo 组件，添加：
- 任务分类（工作/个人）
- 优先级（高/中/低）
- 截止日期"

## 第三轮：持久化
"将 Todo 组件改为使用 localStorage 持久化：
- 初始化时从 localStorage 读取
- 状态变化时自动保存
- 添加保存状态指示器"

## 第四轮：优化
"优化 Todo 组件：
- 使用 React.memo 减少重渲染
- 使用 useCallback 缓存回调
- 添加虚拟滚动优化长列表"
```

### 3. 高级使用场景

#### 大型项目迁移

```markdown
# 场景：MySQL 到 PostgreSQL 迁移

## 迁移任务
将 MySQL 数据库迁移到 PostgreSQL。

## 前置分析
"分析 MySQL 数据库结构：
1. 列出所有表
2. 识别数据类型差异
3. 识别语法差异
4. 评估存储过程和函数

生成迁移报告。"

## Schema 转换
"转换数据库 Schema：
1. SERIAL → BIGSERIAL
2. TINY INT/SMALLINT → SMALLINT
3. DATETIME → TIMESTAMP
4. VARCHAR(n) → VARCHAR(n)
5. TEXT → TEXT
6. ENUM → 自定义类型

生成 Prisma schema 文件。"

## 数据迁移
"执行数据迁移：
1. 导出 MySQL 数据（CSV）
2. 转换数据格式
3. 导入 PostgreSQL
4. 验证数据完整性
5. 重建索引和外键"

## 应用适配
"更新应用以适配 PostgreSQL：
1. 更新 Prisma 配置
2. 修改 SQL 查询（如有）
3. 更新连接字符串
4. 测试所有 CRUD 操作
5. 性能测试和优化"
```

#### 代码审查

```markdown
# 场景：自动化代码审查

## 审查任务
"审查 src/auth 目录的代码，重点关注安全性。"

## 审查标准
1. 安全性
   - 密码存储安全
   - SQL 注入防护
   - XSS 防护
   - CSRF 防护
   
2. 错误处理
   - 异常处理
   - 日志记录
   - 用户反馈

3. 代码质量
   - 代码重复
   - 复杂度
   - 可维护性

## 输出格式
- 发现的问题列表
- 严重程度评级（高/中/低）
- 修复建议
- 代码评分（1-10）"
```

### 4. 专业领域提示词

#### React 开发

```markdown
# React 开发提示词

## 组件开发
"创建一个 React 函数组件，要求：
- 使用 TypeScript
- 完整的 Props 接口定义
- 遵循项目的组件规范
- 包含必要的状态管理
- 添加 JSDoc 注释
- 包含 PropTypes 或 TypeScript 类型"

## Hooks 开发
"创建一个自定义 React Hook，要求：
- 提取可复用的逻辑
- 适当的依赖管理
- 清理副作用
- TypeScript 类型支持
- 完整的 JSDoc

参考项目中现有的 hooks 实现模式。"

## 状态管理
"使用 Zustand 创建一个 store，要求：
- 完整的类型定义
- 持久化支持
- 适当的 actions 组织
- 性能优化考虑

参考项目中的 store 实现模式。"
```

#### Node.js 后端开发

```markdown
# Node.js 后端开发提示词

## API 开发
"创建一个 RESTful API 模块，要求：
- 使用 Express + TypeScript
- 完整的类型定义
- 输入验证（Zod）
- 错误处理中间件
- 统一响应格式
- API 文档注释

遵循项目的 API 规范和目录结构。"

## 数据库操作
"使用 Prisma 创建一个数据服务，要求：
- 完整的类型定义
- 错误处理
- 事务支持
- 性能考虑

遵循项目的数据库规范。"

## 认证授权
"实现 JWT 认证中间件，要求：
- Token 验证
- 过期处理
- 权限检查
- 安全最佳实践

遵循项目的认证规范。"
```

#### DevOps 配置

```markdown
# DevOps 配置提示词

## Docker 配置
"为 [项目名称] 创建 Dockerfile，要求：
- 多阶段构建
- 生产优化
- 最小化镜像
- 安全配置
- 健康检查

参考 Docker 最佳实践。"

## CI/CD 配置
"创建 GitHub Actions 流水线，要求：
- 代码检查
- 测试
- 构建
- 部署
- 通知

遵循项目的 CI/CD 规范。"

## Kubernetes 配置
"为服务创建 Kubernetes 部署配置，要求：
- 副本管理
- 资源限制
- 健康检查
- 配置管理
- 持久化存储

参考项目的 K8s 规范。"
```

### 5. 提示词性能优化

#### Token 优化

```markdown
# Token 优化技巧

## 精简描述
"请帮我创建一个用户管理模块，这个模块需要包含用户的增删改查功能，
包括创建用户、获取用户列表、获取单个用户详情、更新用户信息和删除用户。
同时还需要包含用户验证功能，比如邮箱验证和密码重置等功能。
需要使用 TypeScript 编写，遵循严格的类型检查..."

优化为：

"创建用户管理模块：
- CRUD API（/api/users）
- Prisma model: User
- JWT 认证
- Zod 验证
- TypeScript 严格模式"
```

#### 上下文管理

```markdown
# 选择性引用

## 引用过多 ❌
"参考整个 src/components 目录和 src/hooks 目录中的所有文件，
以及 src/types 目录和 src/utils 目录..."

## 精确引用 ✅
"参考：
- src/components/UserCard.tsx（组件结构）
- src/hooks/useUser.ts（数据获取）
- src/types/user.ts（类型定义）"
```

---

## Cline 与现代技术栈集成

### 1. 前端技术栈集成

#### React + TypeScript

```typescript
// React 组件开发模板
interface UserCardProps {
  id: string;
  name: string;
  email: string;
  avatar?: string;
  onEdit?: (id: string) => void;
  onDelete?: (id: string) => void;
}

export const UserCard: React.FC<UserCardProps> = ({
  id,
  name,
  email,
  avatar,
  onEdit,
  onDelete,
}) => {
  const [isEditing, setIsEditing] = useState(false);
  const [isLoading, setIsLoading] = useState(false);

  const handleEdit = () => {
    if (onEdit) {
      onEdit(id);
    }
  };

  const handleDelete = async () => {
    if (!confirm('Are you sure you want to delete this user?')) {
      return;
    }
    
    setIsLoading(true);
    try {
      if (onDelete) {
        await onDelete(id);
      }
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="user-card" data-testid={`user-card-${id}`}>
      {avatar ? (
        <img src={avatar} alt={name} className="user-card__avatar" />
      ) : (
        <div className="user-card__avatar-placeholder">
          {name.charAt(0).toUpperCase()}
        </div>
      )}
      
      <div className="user-card__info">
        <h3 className="user-card__name">{name}</h3>
        <p className="user-card__email">{email}</p>
      </div>
      
      <div className="user-card__actions">
        <button
          type="button"
          onClick={handleEdit}
          disabled={isLoading}
          className="btn btn--secondary"
        >
          Edit
        </button>
        <button
          type="button"
          onClick={handleDelete}
          disabled={isLoading}
          className="btn btn--danger"
        >
          {isLoading ? 'Deleting...' : 'Delete'}
        </button>
      </div>
    </div>
  );
};

// 测试模板
import { render, screen, fireEvent } from '@testing-library/react';
import { UserCard } from './UserCard';

describe('UserCard', () => {
  const defaultProps = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
  };

  it('renders user information', () => {
    render(<UserCard {...defaultProps} />);
    
    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', async () => {
    const onEdit = jest.fn();
    render(<UserCard {...defaultProps} onEdit={onEdit} />);
    
    fireEvent.click(screen.getByText('Edit'));
    
    expect(onEdit).toHaveBeenCalledWith('1');
  });

  it('calls onDelete with confirmation', async () => {
    const onDelete = jest.fn();
    global.confirm = jest.fn(() => true);
    
    render(<UserCard {...defaultProps} onDelete={onDelete} />);
    
    fireEvent.click(screen.getByText('Delete'));
    
    expect(global.confirm).toHaveBeenCalled();
    expect(onDelete).toHaveBeenCalledWith('1');
  });
});
```

#### Vue 3 + TypeScript

```typescript
// Vue 3 组合式 API 开发模板
<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';

// Props 定义
interface Props {
  userId: string;
  editable?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  editable: true,
});

// Emits 定义
const emit = defineEmits<{
  (e: 'update', data: UserData): void;
  (e: 'delete', id: string): void;
}>();

// 响应式数据
const user = ref<UserData | null>(null);
const isLoading = ref(false);
const error = ref<string | null>(null);

// 计算属性
const displayName = computed(() => {
  return user.value?.name ?? 'Unknown User';
});

const canEdit = computed(() => {
  return props.editable && user.value !== null;
});

// 方法
async function fetchUser() {
  isLoading.value = true;
  error.value = null;
  
  try {
    const response = await fetch(`/api/users/${props.userId}`);
    if (!response.ok) {
      throw new Error('Failed to fetch user');
    }
    user.value = await response.json();
  } catch (e) {
    error.value = e instanceof Error ? e.message : 'Unknown error';
  } finally {
    isLoading.value = false;
  }
}

function handleUpdate(data: UserData) {
  emit('update', data);
}

function handleDelete() {
  if (confirm('Are you sure?')) {
    emit('delete', props.userId);
  }
}

// 生命周期
onMounted(() => {
  fetchUser();
});

// 监听器
watch(() => props.userId, () => {
  fetchUser();
});
</script>

<template>
  <div class="user-profile">
    <div v-if="isLoading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <template v-else-if="user">
      <h2>{{ displayName }}</h2>
      <p>{{ user.email }}</p>
      <div v-if="canEdit" class="actions">
        <button @click="handleUpdate">Edit</button>
        <button @click="handleDelete" class="danger">Delete</button>
      </div>
    </template>
  </div>
</template>
```

### 2. 后端技术栈集成

#### FastAPI + Pydantic

```python
# FastAPI 开发模板
from fastapi import FastAPI, HTTPException, Depends, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, EmailStr, Field, validator
from typing import Optional, List
from datetime import datetime
from enum import Enum

app = FastAPI(title="User API", version="1.0.0")
security = HTTPBearer()

# Enums
class UserRole(str, Enum):
    USER = "user"
    ADMIN = "admin"
    MODERATOR = "moderator"

# Pydantic Models
class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    role: UserRole = UserRole.USER

class UserCreate(UserBase):
    password: str = Field(..., min_length=8, max_length=100)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase letter')
        if not any(c.islower() for c in v):
            raise ValueError('Password must contain lowercase letter')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v

class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    name: Optional[str] = Field(None, min_length=2, max_length=100)
    password: Optional[str] = Field(None, min_length=8, max_length=100)

class UserResponse(UserBase):
    id: str
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True

class PaginatedResponse(BaseModel):
    items: List[UserResponse]
    total: int
    page: int
    page_size: int
    pages: int

class ErrorResponse(BaseModel):
    detail: str
    error_code: Optional[str] = None

# Database Model (example)
class User:
    def __init__(self, id: str, email: str, name: str, 
                 password_hash: str, role: UserRole,
                 created_at: datetime, updated_at: datetime):
        self.id = id
        self.email = email
        self.name = name
        self.password_hash = password_hash
        self.role = role
        self.created_at = created_at
        self.updated_at = updated_at

# API Endpoints
@app.get("/users", response_model=PaginatedResponse, tags=["users"])
async def list_users(
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    search: Optional[str] = None
):
    """List all users with pagination."""
    # Implementation here
    pass

@app.get("/users/{user_id}", response_model=UserResponse, tags=["users"])
async def get_user(user_id: str):
    """Get a single user by ID."""
    # Implementation here
    pass

@app.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED, tags=["users"])
async def create_user(user: UserCreate):
    """Create a new user."""
    # Implementation here
    pass

@app.put("/users/{user_id}", response_model=UserResponse, tags=["users"])
async def update_user(user_id: str, user: UserUpdate):
    """Update an existing user."""
    # Implementation here
    pass

@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT, tags=["users"])
async def delete_user(user_id: str):
    """Delete a user."""
    # Implementation here
    pass
```

#### Django + DRF

```python
# Django REST Framework 开发模板
from rest_framework import serializers, viewsets, permissions, filters
from rest_framework.decorators import action
from rest_framework.response import Response
from django.db.models import Q
from django.contrib.auth.models import User, Group
from django.contrib.auth.hashers import make_password, check_password
from django_filters.rest_framework import DjangoFilterBackend
from datetime import datetime

# Serializers
class UserSerializer(serializers.ModelSerializer):
    """User model serializer."""
    
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 
                  'last_name', 'is_active', 'date_joined', 'groups']
        read_only_fields = ['id', 'date_joined']
    
    def validate_email(self, value):
        """Validate email uniqueness."""
        email = value.lower()
        user = User.objects.filter(email__iexact=email)
        
        # Exclude current instance on update
        if self.instance:
            user = user.exclude(pk=self.instance.pk)
        
        if user.exists():
            raise serializers.ValidationError(
                "A user with this email already exists."
            )
        return value
    
    def create(self, validated_data):
        """Create a new user."""
        validated_data['password'] = make_password(
            validated_data.get('password', '')
        )
        return super().create(validated_data)
    
    def update(self, instance, validated_data):
        """Update a user."""
        password = validated_data.pop('password', None)
        if password:
            instance.password = make_password(password)
        return super().update(instance, validated_data)


class GroupSerializer(serializers.ModelSerializer):
    """Group model serializer."""
    
    class Meta:
        model = Group
        fields = ['id', 'name', 'permissions']
        read_only_fields = ['id']


# ViewSets
class UserViewSet(viewsets.ModelViewSet):
    """
    User API endpoint.
    
    Provides CRUD operations and custom actions:
    - GET /users/ - List all users
    - GET /users/{id}/ - Get single user
    - POST /users/ - Create user
    - PUT /users/{id}/ - Update user
    - PATCH /users/{id}/ - Partial update
    - DELETE /users/{id}/ - Delete user
    - POST /users/{id}/activate/ - Activate user
    - POST /users/{id}/deactivate/ - Deactivate user
    """
    
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, 
                       filters.OrderingFilter]
    filterset_fields = ['is_active', 'is_staff', 'groups']
    search_fields = ['username', 'email', 'first_name', 'last_name']
    ordering_fields = ['username', 'email', 'date_joined']
    ordering = ['-date_joined']
    
    def get_queryset(self):
        """Filter queryset based on user permissions."""
        user = self.request.user
        if user.is_superuser:
            return User.objects.all()
        return User.objects.filter(pk=user.pk)
    
    @action(detail=True, methods=['post'])
    def activate(self, request, pk=None):
        """Activate a user account."""
        user = self.get_object()
        user.is_active = True
        user.save()
        serializer = self.get_serializer(user)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def deactivate(self, request, pk=None):
        """Deactivate a user account."""
        user = self.get_object()
        if user.is_superuser:
            raise serializers.ValidationError(
                "Cannot deactivate superuser account."
            )
        user.is_active = False
        user.save()
        serializer = self.get_serializer(user)
        return Response(serializer.data)
    
    @action(detail=False, methods=['get'])
    def me(self, request):
        """Get current user profile."""
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)
```

### 3. 数据库集成

#### PostgreSQL + Prisma

```prisma
// Prisma Schema 模板
generator client {
  provider = "prisma-client-js"
  output   = "../node_modules/.prisma/client"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Enums
enum UserRole {
  USER
  ADMIN
  MODERATOR
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

// Models
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  username      String    @unique
  password      String
  firstName     String?
  lastName      String?
  role          UserRole  @default(USER)
  isActive      Boolean   @default(true)
  emailVerified DateTime?
  image         String?
  
  // Relations
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  following     Follow[]  @relation("Following")
  followers     Follow[]  @relation("Followers")
  
  // Timestamps
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Indexes
  @@index([email])
  @@index([username])
  @@index([role])
  @@index([createdAt])
}

model Post {
  id          String     @id @default(cuid())
  title       String
  slug        String     @unique
  content     String
  excerpt     String?
  status      PostStatus @default(DRAFT)
  viewCount   Int        @default(0)
  
  // Relations
  author      User       @relation(fields: [authorId], references: [id])
  authorId    String
  category    Category?  @relation(fields: [categoryId], references: [id])
  categoryId  String?
  tags        Tag[]
  comments    Comment[]
  likes       Like[]
  
  // Timestamps
  publishedAt DateTime?
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  
  // Indexes
  @@index([authorId])
  @@index([categoryId])
  @@index([status])
  @@index([publishedAt])
}

model Category {
  id       String @id @default(cuid())
  name     String @unique
  slug     String @unique
  posts    Post[]
  
  @@index([slug])
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]
  
  @@index([slug])
}

model Comment {
  id        String   @id @default(cuid())
  content   String
  isApproved Boolean @default(true)
  
  // Relations
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  post     Post     @relation(fields: [postId], references: [id])
  postId   String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies  Comment[] @relation("CommentReplies")
  
  // Timestamps
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([authorId])
  @@index([postId])
  @@index([parentId])
}

model Like {
  id        String   @id @default(cuid())
  
  // Relations
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  post      Post    @relation(fields: [postId], references: [id])
  postId    String
  
  // Constraints
  @@unique([userId, postId])
  @@index([userId])
  @@index([postId])
}

model Follow {
  id          String @id @default(cuid())
  
  // Relations
  follower    User   @relation("Following", fields: [followerId], references: [id])
  followerId  String
  following  User   @relation("Followers", fields: [followingId], references: [id])
  followingId String
  
  // Constraints
  @@unique([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}
```

---

## Cline 快捷键完全手册

### 默认快捷键

| 功能分类 | 功能 | macOS | Windows/Linux | 说明 |
|---------|------|-------|---------------|------|
| **Cline 面板** | 打开面板 | `Cmd + Shift + I` | `Ctrl + Shift + I` | 打开对话面板 |
| | 发送消息 | `Enter` | `Enter` | 发送当前输入 |
| | 多行输入 | `Shift + Enter` | `Shift + Enter` | 换行 |
| | 清空输入 | `Cmd + K` | `Ctrl + K` | 清空输入框 |
| | 停止生成 | `Esc` | `Esc` | 停止当前生成 |
| **任务管理** | 新任务 | `Cmd + Shift + N` | `Ctrl + Shift + N` | 开始新任务 |
| | 继续任务 | `Cmd + Shift + R` | `Ctrl + Shift + R` | 继续上次任务 |
| | 任务历史 | `Cmd + Shift + H` | `Ctrl + Shift + H` | 查看历史 |
| **工具操作** | 批准工具 | `Enter` | `Enter` | 批准执行 |
| | 拒绝工具 | `Cmd + D` | `Ctrl + D` | 拒绝当前 |
| | 全部批准 | `Cmd + Shift + A` | `Ctrl + Shift + A` | 批准所有 |

### 自定义快捷键

```json
// keybindings.json
[
  // Cline 快捷键
  {
    "key": "cmd+shift+i",
    "command": "cline.open",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+n",
    "command": "cline.newTask",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+h",
    "command": "cline.history",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "cline.acceptAll",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+d",
    "command": "cline.dismissAll",
    "when": "editorTextFocus"
  },
  
  // 快速操作
  {
    "key": "cmd+shift+g",
    "command": "cline.generate",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "cline.refactor",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+t",
    "command": "cline.test",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+b",
    "command": "cline.explain",
    "when": "editorTextFocus"
  }
]
```

---

## Cline 命令参考

### 对话命令

| 命令 | 功能 | 用法 |
|------|------|------|
| `/new` | 开始新对话 | `/new` |
| `/clear` | 清空上下文 | `/clear` |
| `/model <name>` | 切换模型 | `/model claude-sonnet` |
| `/cost` | 查看成本 | `/cost` |
| `/token` | 查看 Token 统计 | `/token` |
| `/export` | 导出对话 | `/export filename.md` |
| `/help` | 获取帮助 | `/help` |
| `/stop` | 停止生成 | `/stop` |

### 工具命令

| 命令 | 功能 | 用法 |
|------|------|------|
| `@file:` | 引用文件 | `@file:src/app.ts` |
| `@folder:` | 引用目录 | `@folder:src/components` |
| `@symbol:` | 引用符号 | `@symbol:UserService` |
| `@git:` | Git 信息 | `@git:HEAD` |
| `@search:` | 搜索结果 | `@search:auth` |
| `@mcp:` | MCP 工具 | `@mcp:github/issue` |

### MCP 服务器命令

| 命令 | 功能 | 用法 |
|------|------|------|
| `/mcp list` | 列出 MCP 服务器 | `/mcp list` |
| `/mcp add <name>` | 添加 MCP 服务器 | `/mcp add github` |
| `/mcp remove <name>` | 移除 MCP 服务器 | `/mcp remove github` |
| `/mcp restart` | 重启 MCP 服务器 | `/mcp restart` |

---

## Cline 完整配置参考

### 基础配置

```json
{
  "cline": {
    // API 配置
    "apiProvider": "anthropic",
    "apiKey": "${ANTHROPIC_API_KEY}",
    "model": "claude-sonnet-4-20250514",
    
    // 模型参数
    "maxTokens": 8192,
    "temperature": 0.7,
    "topP": 0.9,
    "topK": 40,
    
    // 自动批准
    "autoApprove": "ask",
    "autoApproveExportFunctions": true,
    "autoApproveIntelligently": false,
    
    // 上下文
    "context": {
      "maxFiles": 50,
      "maxTokens": 100000,
      "priorityFiles": ["*.ts", "*.tsx", "*.js", "*.py"],
      "excludePatterns": ["node_modules/**", "dist/**", "*.test.ts"]
    }
  }
}
```

### MCP 配置

```json
{
  "cline.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "allowedDirectories": ["${workspaceFolder}"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "slack": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-slack"],
      "env": {
        "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
      }
    }
  }
}
```

### 安全配置

```json
{
  "cline.security": {
    "allowedDirectories": ["${workspaceFolder}"],
    "blockedCommands": [
      "rm -rf /*",
      "format C:",
      "del /f /s /q"
    ],
    "requireConfirmation": {
      "delete": true,
      "bash": true,
      "network": true,
      "destructive": true
    },
    "auditLogging": {
      "enabled": true,
      "path": ".cline/audit.log",
      "includeContext": true
    }
  }
}
```

### 成本控制配置

```json
{
  "cline.costControl": {
    "enabled": true,
    "maxCostPerSession": 5.0,
    "maxCostPerDay": 50.0,
    "alertThreshold": 0.8,
    "modelFallback": {
      "enabled": true,
      "fallbackModel": "claude-haiku-3-20250514",
      "fallbackThreshold": 0.5
    }
  }
}
```

### 性能配置

```json
{
  "cline.performance": {
    "maxConcurrentTools": 3,
    "toolTimeout": 60,
    "responseTimeout": 120,
    "cacheEnabled": true,
    "cacheSize": "100MB",
    "streamResponses": true
  }
}
```

### UI 配置

```json
{
  "cline.ui": {
    "showWelcomeMessage": true,
    "showTokenCount": true,
    "showCostEstimate": true,
    "showModelIndicator": true,
    "position": "right",
    "width": 450
  }
}
```

---

**Cline 文档最终统计**：
- 撰写时间：2026年4月
- 最终行数：约 **5000+ 行**
- 代码示例：90+
- 配置示例：35+
- 快捷键表格：30+
- 提示词模板：20+
- 技术栈集成：25+

> [!SUCCESS]
> Cline 作为完全开源的 AI 编程助手，凭借其 MCP 协议支持、多模型灵活切换和完全本地运行的能力，为注重隐私和成本的开发者提供了优秀的解决方案。结合 Claude、GPT 等大模型的强大能力，以及 Ollama 等本地模型的隐私保护，Cline 成为 2026 年最具性价比的 AI 编程工具。
