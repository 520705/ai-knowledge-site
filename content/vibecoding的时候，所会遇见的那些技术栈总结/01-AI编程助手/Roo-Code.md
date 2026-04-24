# Roo Code：开源 AI 编程助手的深度解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 Roo Code 的技术架构、与 Cline 的渊源、核心功能配置及与主流 AI 编程工具的对比分析。

---

## 目录

1. [[#概述与历史渊源]]
2. [[#与 Cline 的关系解析]]
3. [[#核心功能详解]]
4. [[#安装与配置指南]]
5. [[#高级配置与自定义]]
6. [[#MCP 协议集成]]
7. [[#自定义 Prompt 工程]]
8. [[#协作模式与团队使用]]
9. [[#与主流工具的对比分析]]
10. [[#开源优势与局限性]]
11. [[#实战场景与最佳实践]]
12. [[#常见问题与故障排除]]
13. [[#选型建议]]
14. [[#参考资料]]

---

## 概述与历史渊源

### 什么是 Roo Code

Roo Code（前身为 "Cline"）是一款运行在 VS Code 和 JetBrains IDE 中的开源 AI 编程助手。它允许开发者通过自然语言描述直接生成代码、修改文件、执行终端命令、操作浏览器等操作。与传统的代码补全工具不同，Roo Code 以**自主代理（Autonomous Agent）** 的形式工作，能够理解项目上下文、规划任务步骤、调用多种工具来完成复杂编程任务。

Roo Code 的核心设计理念是**让 AI 成为真正的编程搭档**，而非仅仅是一个代码片段生成器。它支持多文件编辑、Git 操作、文件系统操作，并能访问 Web 通过 `browser` 工具搜索信息或验证事实。

### 项目背景

Roo Code 由独立开发者 **saoud**（网名）于 2023 年创建，最初命名为 Cline（Claude Line 的缩写）。2024 年底，由于与 Anthropic 公司就商标问题进行协商，Cline 正式更名为 **Roo Code**（Roo 取自袋鼠，呼应 Anthropic 的 Claude 命名体系）。

更名后项目继续保持高速发展，截至 2026 年初：
- **GitHub Stars**: 55,000+
- **周下载量**: 200,000+
- **活跃贡献者**: 100+
- **支持的模型**: 15+

### 核心理念

```markdown
# Roo Code 设计哲学

┌─────────────────────────────────────────────────────────────┐
│                      核心理念                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 开发者优先                                               │
│     ├─ 开发者保持对代码的完全控制                               │
│     ├─ AI 是协作者而非替代者                                   │
│     └─ 决策权始终在开发者手中                                  │
│                                                              │
│  2. 透明可控                                                  │
│     ├─ 开源代码，完全可审计                                    │
│     ├─ 可配置的权限和安全设置                                  │
│     └─ 清晰的操作日志                                         │
│                                                              │
│  3. 开放生态                                                  │
│     ├─ 支持多种 AI 模型                                       │
│     ├─ MCP 协议扩展                                           │
│     └─ 完全可定制的工作流                                      │
│                                                              │
│  4. 隐私保护                                                  │
│     ├─ 支持本地模型运行                                       │
│     ├─ 代码不强制上传到云端                                    │
│     └─ 敏感项目可完全离线                                     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 与传统工具的区别

| 维度 | 传统代码补全 | Roo Code |
|------|-------------|----------|
| **交互方式** | 自动补全 | 对话式交互 |
| **上下文** | 当前文件 | 整个项目 |
| **操作能力** | 仅生成代码 | 执行文件操作 |
| **任务处理** | 单次请求 | 多步骤任务 |
| **自主性** | 无 | 可配置自主级别 |
| **学习方式** | 无 | 理解项目结构 |

---

## 与 Cline 的关系解析

### 命名变更始末

2024 年 11 月，Anthropic 向 Cline 项目发送了商标侵权通知，要求项目更名。理由是 "Cline" 与 "Claude" 在视觉和听觉上存在混淆可能。项目维护者经过慎重考虑后决定配合更名，将项目改名为 **Roo Code**。

> [!IMPORTANT]
> 更名仅涉及品牌层面，核心代码保持不变。用户无需担心功能缺失或数据迁移问题，所有 Cline 的配置和历史记录均可无缝迁移至 Roo Code。

### 变更时间线

| 时间 | 事件 | 影响 |
|------|------|------|
| 2023.12 | Cline v1 发布 | 初始版本 |
| 2024.03 | Cline v2 | MCP 支持 |
| 2024.07 | Cline v3 | JetBrains 支持 |
| 2024.11 | 收到商标通知 | 开始更名讨论 |
| 2024.12 | Roo Code v3.5 | 正式更名 |
| 2025.01 | Roo Code v4.0 | 全新品牌 |
| 2026.01 | Roo Code v5.0 | 多 Agent 支持 |

### Fork 分支生态

Cline 更名后，社区出现了多个 Fork 分支，形成了一个有趣的开源生态：

| 分支名称 | 维护者 | 特点 | 状态 |
|---------|--------|------|------|
| **Roo Code** | سعود | 官方主分支，持续更新 | 活跃维护 |
| **Cline Fork** | community | 保持原名，部分功能增强 | 社区维护 |
| **Roo Code Pro** | community | 添加高级功能 | 独立分支 |
| **Roo Code Plus** | community | 性能优化 | 独立分支 |

> [!TIP]
> 用户应优先选择 Roo Code 官方版本，以获得最稳定的功能更新和安全补丁。

---

## 核心功能详解

### 1. 多步骤任务执行

Roo Code 的核心能力在于能够将复杂任务分解为多个步骤，并自主执行：

```
用户请求 → 任务规划 → 工具调用 → 结果验证 → 迭代优化
```

#### 任务循环机制

```markdown
# 任务执行流程

1. 理解阶段
   ├─ 分析用户输入
   ├─ 提取关键信息
   └─ 识别约束条件

2. 规划阶段
   ├─ 拆解子任务
   ├─ 确定执行顺序
   └─ 预估风险

3. 执行阶段
   ├─ 调用适当工具
   ├─ 处理中间结果
   └─ 错误处理

4. 验证阶段
   ├─ 检查执行结果
   ├─ 确认任务完成
   └─ 生成报告
```

#### 示例工作流

```
用户：创建一个用户注册功能，包含邮箱验证和密码加密

Roo Code 分析：
1. 检查现有数据库 Schema
   └─ Read: prisma/schema.prisma

2. 创建用户表迁移文件
   └─ Write: prisma/migrations/xxx_create_user.sql

3. 实现注册 API 路由
   └─ Write: src/routes/auth/register.ts

4. 添加邮箱格式验证
   └─ Edit: src/validators/email.ts

5. 集成 bcrypt 密码加密
   └─ Edit: src/services/authService.ts

6. 编写单元测试
   └─ Write: src/__tests__/auth.test.ts

7. 更新 API 文档
   └─ Edit: docs/api/auth.md

✅ 任务完成！
```

### 2. 工具生态系统

Roo Code 内置了丰富的工具，能够完成多种开发任务：

| 工具名称 | 功能描述 | 使用场景 |
|---------|---------|---------|
| `read` | 读取文件内容 | 理解现有代码 |
| `write` | 写入或创建文件 | 生成新代码 |
| `edit` | 精确修改文件 | 局部代码调整 |
| `delete` | 删除文件或目录 | 清理冗余代码 |
| `bash` | 执行 Shell 命令 | 运行测试、构建 |
| `browser` | 访问网页 | 搜索文档、验证 API |
| `grep` | 全文搜索 | 定位代码位置 |
| `glob` | 文件模式匹配 | 批量文件操作 |

#### 工具使用示例

```typescript
// 读取文件
Read: src/utils/format.ts
// 返回文件内容供分析

// 写入文件
Write: src/components/UserCard.tsx
---
import React from 'react';

interface UserProps {
  name: string;
  email: string;
  avatar?: string;
}

export const UserCard: React.FC<UserProps> = ({ name, email, avatar }) => {
  return (
    <div className="user-card">
      {avatar && <img src={avatar} alt={name} />}
      <h3>{name}</h3>
      <p>{email}</p>
    </div>
  );
};
---

// 编辑文件
Edit:
File: src/utils/format.ts
Old: return value.toLowerCase();
New: return value.trim().toLowerCase();

// 执行命令
Terminal: npm test -- user.test.ts

// 搜索代码
Grep: "useAuth"
In: src/

// 文件模式匹配
Glob: src/**/*.test.ts
```

### 3. MCP 协议支持

Roo Code 原生支持 Model Context Protocol（MCP），可以连接外部工具和服务：

```json
// .roo/mcp.json 配置示例
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "./src"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your-token-here"
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
        "SLACK_BOT_TOKEN": "xoxb-..."
      }
    }
  }
}
```

### 4. 多模型支持

Roo Code 不绑定特定 AI 提供商，支持多种模型：

| 提供商 | 支持模型 | API 格式 |
|-------|---------|---------|
| **Anthropic** | Claude 3.5/3.7 Sonnet, Opus, Haiku | Anthropic API |
| **OpenAI** | GPT-4o, o1, o3 | OpenAI Compatible |
| **Google** | Gemini 2.0 Pro/Flash | Google AI API |
| **Ollama** | Llama, Qwen, Mistral 等 | Ollama Local |
| **LM Studio** | 本地量化模型 | OpenAI Compatible |
| **OpenRouter** | 聚合多模型 | OpenAI Compatible |
| **DeepSeek** | DeepSeek V3/R1 | DeepSeek API |

> [!TIP]
> 使用 Ollama 或 LM Studio 运行本地模型，数据完全保留在本地，适合处理敏感代码或离线开发场景。

### 5. 成本控制机制

Roo Code 内置详细的 Token 计数和成本追踪功能：

```
会话统计：
├─ 输入 Token：12,450
├─ 输出 Token：3,280
├─ 预估成本：$0.08
└─ 速率限制：安全（低于阈值）
```

用户可以设置每会话或每日成本上限，防止意外超支：

```json
{
  "roo-code": {
    "costControl": {
      "enabled": true,
      "maxCostPerSession": 5.0,
      "maxCostPerDay": 50.0,
      "alertThreshold": 0.8
    }
  }
}
```

### 6. 对话管理

```markdown
# 对话功能

## 基本操作
├─ 新建会话：/new
├─ 继续对话：继续输入即可
├─ 清空对话：/clear
└─ 导出对话：/export

## 上下文管理
├─ @file: 添加文件到上下文
├─ @folder: 添加文件夹到上下文
├─ @search: 添加搜索结果到上下文
└─ @git: 添加 Git 信息到上下文

## 对话技巧
├─ 使用 "继续" 执行上一个命令的后续步骤
├─ 使用 "撤销" 回退上一步操作
└─ 使用 "解释" 请求解释生成的代码
```

---

## 安装与配置指南

### 环境要求

| 要求 | 最低配置 | 推荐配置 |
|------|---------|---------|
| IDE | VS Code 1.75+ / WebStorm 2024+ | VS Code 最新版 |
| 网络 | 能访问 AI API | 稳定快速连接 |
| 内存 | 4GB 可用 | 8GB+ 可用 |
| 磁盘 | 200MB 可用 | 无特殊要求 |

### 安装步骤

#### VS Code 安装

1. 打开 VS Code，进入 Extensions（扩展）面板
2. 搜索 "Roo Code" 或 "Roo Code"
3. 点击安装（注意识别官方发布者）
4. 安装完成后，按 `Cmd/Ctrl + Shift + P` 打开命令面板
5. 输入 "Roo Code: Open Settings" 打开配置

```bash
# 或者使用命令行安装
code --install-extension saoudrizwan.roo-code

# VSIX 文件安装
code --install-extension roo-code-*.vsix
```

#### JetBrains 安装

1. 打开 JetBrains IDE，进入 Settings → Plugins
2. 搜索 "Roo Code" 或 "Roo Code"
3. 安装插件并重启 IDE

> [!NOTE]
> JetBrains 版本功能可能略有不同，部分功能在 JetBrains 上可能不可用。

### API Key 配置

Roo Code 需要配置 AI API Key 才能工作：

#### 1. Anthropic API Key（推荐）

```json
// 设置页面配置
{
  "apiProvider": "anthropic",
  "apiKey": "sk-ant-xxxxx",
  "model": "claude-sonnet-4-20250514"
}
```

获取方式：
1. 访问 https://console.anthropic.com/
2. 注册账户并登录
3. 进入 API Keys 页面
4. 点击 "Create Key" 生成新的 API Key

#### 2. OpenAI API Key

```json
{
  "apiProvider": "openai",
  "apiKey": "sk-xxxxx",
  "model": "gpt-4o"
}
```

获取方式：
1. 访问 https://platform.openai.com/
2. 注册账户并登录
3. 进入 API Keys 页面
4. 创建新的 Secret Key

#### 3. OpenRouter（聚合方案）

```json
{
  "apiProvider": "openrouter",
  "apiKey": "sk-or-xxxxx",
  "model": "anthropic/claude-sonnet-4"
}
```

> [!IMPORTANT]
> API Key 应妥善保管，切勿提交至代码仓库。建议使用环境变量或 IDE 的 Secrets Storage。

---

## 高级配置与自定义

### 系统提示词定制

Roo Code 允许通过自定义系统提示词来调整 AI 行为：

```markdown
# .roo/system_prompt.md
# cursorules

# 角色定义
你是一个经验丰富的前端架构师，专注于 React 和 TypeScript。

# 约束条件
- 优先使用函数式组件和 Hooks
- 所有组件必须编写 PropTypes 或 TypeScript 类型
- 遵循 Airbnb JavaScript Style Guide
- 组件文件不超过 200 行，超出则必须拆分
- 使用 Tailwind CSS 进行样式设计
- 遵循原子化 CSS 原则

# 代码规范
- 变量命名：camelCase
- 组件命名：PascalCase
- 常量命名：UPPER_SNAKE_CASE
- 文件命名：kebab-case.tsx

# 输出格式
- 代码块必须标注语言类型
- 复杂逻辑需附加注释
- 最后给出时间复杂度分析

# 错误处理
- 所有异步操作必须使用 try-catch
- 统一使用自定义 Error 类
- 添加适当的日志记录

# 性能优化
- 优先使用 React.memo
- 使用 useMemo 和 useCallback
- 实现虚拟滚动优化长列表
```

### 任务批准模式

Roo Code 提供三种任务批准模式，平衡自动化与安全性：

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **Auto Approve** | 所有操作自动执行 | 简单任务、熟悉代码库 |
| **Ask After Every Tool Call** | 每个工具调用前询问 | 谨慎开发、数据敏感 |
| **Never Approve** | 完全手动模式 | 审计需求、完全控制 |

#### 配置方式

```json
{
  "roo-code": {
    "autoApprove": "ask",
    "autoApproveExportFunctions": true,
    "autoApproveIntelligent": false,
    "askForDrafts": true,
    "maxAutoApproveEdits": 5
  }
}
```

### 任务模板

定义可复用的任务模板，加速常见开发流程：

```json
// .roo/templates/react-component.json
{
  "name": "创建 React 组件",
  "description": "生成标准化的 React 函数组件",
  "prompt": "创建一个新的 React 函数组件，名为 {componentName}，包含：
1. TypeScript Props 接口
2. 使用 useState 和 useEffect
3. 导出默认组件和命名组件
4. 包含 Jest 测试文件
5. 使用 CSS Modules 样式",
  "variables": [
    {
      "name": "componentName",
      "type": "string",
      "required": true,
      "description": "组件名称（PascalCase）"
    }
  ]
}
```

### 快捷键配置

优化工作流程的快捷键设置：

```json
// keybindings.json
{
  "roo-code.keybindings": [
    {
      "key": "cmd+shift+r",
      "command": "roo-code.new-task"
    },
    {
      "key": "cmd+shift+a",
      "command": "roo-code.accept-suggestion"
    },
    {
      "key": "cmd+shift+d",
      "command": "roo-code.discard"
    },
    {
      "key": "cmd+shift+c",
      "command": "roo-code.toggle-panel"
    }
  ]
}
```

### 高级设置

```json
{
  "roo-code": {
    "general": {
      "model": "claude-sonnet-4-20250514",
      "temperature": 0.7,
      "maxTokens": 8192,
      "stream": true
    },
    "sandbox": {
      "dangerouslySkipParse": false,
      "maxWorkspaceSize": "500MB",
      "allowedPaths": ["${workspaceFolder}"]
    },
    "tools": {
      "bash": {
        "timeout": 60,
        "cwd": "${workspaceFolder}"
      },
      "browser": {
        "headless": true,
        "viewport": { "width": 1280, "height": 720 }
      }
    },
    "ui": {
      "showWelcomeMessage": true,
      "showTokenCount": true,
      "showCostEstimate": true
    }
  }
}
```

---

## MCP 协议集成

### MCP 简介

Model Context Protocol (MCP) 是一个开放标准，允许 AI 系统与外部工具进行标准化交互。Roo Code 原生支持 MCP，可以连接各种服务扩展功能。

### 常用 MCP 服务器

#### 1. 文件系统服务器

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "allowedDirectories": ["/path/to/project"]
    }
  }
}
```

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

#### 3. Memory 服务器

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

### MCP 工具使用示例

```markdown
# GitHub 集成示例

用户: 创建 GitHub issue 报告这个 bug

Roo Code:
Tool: use_mcp_tool
Server: github
Tool: create_issue
Arguments:
{
  "owner": "username",
  "repo": "my-project",
  "title": "Bug: Login fails with special characters",
  "body": "## Description\n\nLogin fails when username contains... ",
  "labels": ["bug", "high-priority"]
}

# Slack 通知示例

用户: 通知团队新版本已部署

Roo Code:
Tool: use_mcp_tool
Server: slack
Tool: send_message
Arguments:
{
  "channel": "#engineering",
  "text": "🚀 Version 2.0.0 deployed to production!"
}
```

---

## 自定义 Prompt 工程

### Prompt 设计原则

```markdown
# 有效的 Prompt 设计

## 1. 清晰的结构
├─ 角色定义（可选）
├─ 任务描述
├─ 约束条件
├─ 输出格式
└─ 示例（可选）

## 2. 具体的要求
├─ 使用具体而非抽象
├─ 提供足够的上下文
├─ 说明预期结果
└─ 指定限制条件

## 3. 迭代优化
├─ 从简单开始
├─ 根据结果调整
├─ 添加更多细节
└─ 持续改进
```

### 常用 Prompt 模板

#### 1. 代码生成模板

```markdown
# 代码生成 Prompt

请生成 [编程语言] 代码，实现 [功能描述]。

## 技术要求
- 语言：[具体语言和版本]
- 框架：[使用的框架]
- 类型系统：[是否需要类型]

## 功能要求
[详细描述需要实现的功能]

## 约束条件
- 性能要求：[如有]
- 安全性：[如有]
- 兼容性：[如有]

## 输出格式
- 完整可运行的代码
- 包含必要的注释
- 附带使用示例
```

#### 2. 代码重构模板

```markdown
# 代码重构 Prompt

请重构以下代码，目标：
1. 提高可读性
2. 优化性能
3. 提升可维护性
4. 保持功能不变

## 当前代码
[粘贴代码]

## 重构要求
- 保持 API 兼容性
- 不改变外部行为
- 添加适当的测试

## 关注点
[指定需要特别关注的方面]
```

#### 3. Bug 修复模板

```markdown
# Bug 修复 Prompt

我遇到了一个问题，请帮忙分析和修复。

## 错误信息
[粘贴错误日志或描述]

## 相关代码
[粘贴相关代码片段]

## 复现步骤
[描述如何复现这个问题]

## 环境信息
- 语言/框架版本：
- 操作系统：
- 其他相关依赖：
```

### 项目级 Prompt

```markdown
# .roo/project-prompt.md

## 项目信息
- 项目名称：[项目名称]
- 技术栈：[技术栈描述]
- 代码风格：[风格指南]

## 架构规范
- 目录结构：[描述]
- 模块划分：[描述]
- 设计模式：[描述]

## 代码规范
[详细的代码规范]

## 安全要求
[安全相关的约束]
```

---

## 协作模式与团队使用

### 个人使用模式

```markdown
# 个人使用配置

## 基本设置
- 批准模式：Ask After Every Tool Call
- 日志记录：全部记录
- 成本追踪：启用

## 常用任务
1. 代码生成
2. Bug 修复
3. 重构
4. 测试生成
5. 文档编写
```

### 团队协作配置

```json
{
  "roo-code": {
    "team": {
      "sharedRules": true,
      "rulesPath": ".roo/team-rules",
      "approvalWorkflow": {
        "enabled": true,
        "requireApprovalFor": [
          "delete",
          "bash"
        ],
        "approvers": ["team-lead", "senior-dev"]
      }
    }
  }
}
```

### 团队规则示例

```markdown
# .roo/team-rules/default.mdc

## 团队代码规范

### TypeScript
- 严格模式
- 禁止 any
- 必须有类型注解

### React
- 函数组件
- Hooks 优先
- 组件拆分原则

### Git
- 提交信息格式
- 分支命名规范
- Code Review 要求
```

---

## 与主流工具的对比分析

### 功能矩阵对比

| 功能 | Roo Code | Cursor | GitHub Copilot | Claude Code |
|------|----------|--------|---------------|------------|
| **代码生成** | ✅ | ✅ | ✅ | ✅ |
| **多文件编辑** | ✅ | ✅ | ✅ | ✅ |
| **终端命令执行** | ✅ | ❌ | ❌ | ✅ |
| **Web 搜索** | ✅ | ❌ | ❌ | ✅ |
| **Git 操作** | ✅ | 部分 | ❌ | ✅ |
| **MCP 扩展** | ✅ | ✅ | ❌ | ❌ |
| **本地模型** | ✅ | ❌ | ❌ | ❌ |
| **价格** | 免费 | $20/月 | $10/月 | $25/月 |
| **开源** | ✅ | ❌ | ❌ | ❌ |

### 成本对比

| 工具 | 订阅费用 | API 成本 | 总成本 |
|------|---------|---------|--------|
| **Roo Code** | 免费 | 按量付费（自选模型） | 极低 |
| **Cursor Pro** | $20/月 | 包含在订阅内 | $20/月 |
| **Copilot** | $10/月（个人）/ $19/月（企业） | 包含在订阅内 | $10-19/月 |
| **Claude Code** | $25/月 | 包含在订阅内 | $25/月 |

> [!TIP]
> 使用 Roo Code + Ollama 本地模型，总成本趋近于零，且数据完全隐私。

### 适用场景推荐

| 场景 | 推荐工具 | 理由 |
|------|---------|------|
| 个人开发者、低预算 | **Roo Code** | 完全免费，灵活配置 |
| 企业团队、安全合规 | **Copilot Business** | SOC 认证、权限管理 |
| 快速原型、创意工作 | **Cursor** | 优秀 UX、Tab 补全 |
| 深度代码分析、审计 | **Claude Code** | 最强代码理解能力 |
| 隐私敏感、离线开发 | **Roo Code + Ollama** | 完全本地、数据不出境 |

### 详细对比

#### Roo Code vs Cline

| 维度 | Roo Code | Cline |
|------|----------|-------|
| **品牌** | Roo Code | Cline |
| **核心功能** | 相同 | 相同 |
| **维护者** | سعود | سعود |
| **社区** | 活跃 | 活跃 |
| **更新频率** | 每周 | 每周 |
| **建议选择** | 新用户 | 老用户可继续使用 |

#### Roo Code vs Cursor

| 维度 | Roo Code | Cursor |
|------|----------|--------|
| **许可证** | MIT | 专有 |
| **价格** | 免费 | $20/月 |
| **IDE** | VS Code/JetBrains 扩展 | 自有 IDE |
| **Tab 补全** | ❌ | ✅ |
| **实时补全** | ❌ | ✅ |
| **多模型支持** | ✅ | ✅ |
| **本地模型** | ✅ | ❌ |
| **学习曲线** | 中等 | 低 |

---

## 开源优势与局限性

### 开源优势

#### 1. 透明性与可审计性

开源意味着任何人都可以审查代码逻辑：

```
代码审计路径：
1. GitHub 仓库 → 完整源代码
2. 提交历史 → 变更追踪
3. Issue 区 → 已知问题
4. PR 区 → 社区审核
```

> [!NOTE]
> 对于企业用户，可自行审计代码确保无后门或数据泄露风险，这是闭源工具无法提供的保障。

#### 2. 社区驱动创新

开源社区持续贡献新功能和改进：

- **第三方 MCP 服务器**：社区开发的 Salesforce、Notion 等集成
- **主题和 UI 改进**：用户贡献的界面优化
- **功能增强分支**：实验性功能的试验田

#### 3. 数据隐私

完全本地运行的能力：

```bash
# 使用 Ollama 完全离线
ollama pull llama3.2

# Roo Code 配置
{
  "apiProvider": "ollama",
  "apiBase": "http://localhost:11434"
}
```

所有代码和对话都在本地处理，不经过任何第三方服务器。

#### 4. 社区支持

```markdown
# 社区资源

## 官方渠道
├─ GitHub Issues：Bug 报告和功能请求
├─ GitHub Discussions：使用讨论
└─ Discord：实时交流

## 社区资源
├─ Awesome Roo Code：精选资源列表
├─ Roo Code Templates：模板分享
└─ 博客教程：用户分享经验
```

### 局限性

#### 1. 用户体验

| 局限 | 说明 | 影响 |
|------|------|------|
| **无 Tab 补全** | 缺乏实时代码补全 | 需要主动触发对话 |
| **界面简陋** | 功能优先于界面 | 学习成本略高 |
| **需要配置** | 非开箱即用 | 初始设置时间 |

#### 2. 功能深度

| 局限 | 说明 | 影响 |
|------|------|------|
| **Agent 稳定性** | 不如 Claude Code | 复杂任务可能失败 |
| **多模态** | 图片理解较弱 | 视觉相关任务受限 |
| **代码索引** | 无内置语义搜索 | 大项目查询受限 |

#### 3. 生态整合

| 局限 | 说明 | 影响 |
|------|------|------|
| **IDE 深度** | 无原生 IDE 集成 | 某些功能受限 |
| **协作功能** | 无内置协作 | 依赖外部工具 |
| **企业支持** | 无官方企业版 | 大企业采购受限 |

---

## 实战场景与最佳实践

### 场景一：快速构建 CRUD API

**Prompt 示例：**

```markdown
使用 Express + TypeScript 创建一个用户管理 API，包含：
1. 用户注册（POST /users/register）
2. 用户登录（POST /users/login）
3. 获取用户信息（GET /users/:id）
4. 更新用户信息（PUT /users/:id）
5. 删除用户（DELETE /users/:id）

要求：
- 使用 Prisma ORM
- JWT 认证
- 输入验证（Zod）
- 统一错误处理
- Swagger 文档
```

**执行流程：**

1. Roo Code 分析项目结构
2. 创建 `src/routes/users.ts`
3. 创建 `src/controllers/userController.ts`
4. 创建 `src/services/userService.ts`
5. 添加 `src/middleware/auth.ts`
6. 更新 `app.ts` 注册路由
7. 创建 `prisma/schema.prisma`
8. 编写单元测试

### 场景二：代码重构与迁移

**Prompt 示例：**

```markdown
将项目中的 JavaScript 代码迁移到 TypeScript，具体要求：
1. 为所有函数添加 TypeScript 类型
2. 使用 interface 替代 PropTypes
3. 将 .js 文件重命名为 .ts
4. 更新 import 语句
5. 修复类型错误
6. 运行 tsc --noEmit 验证

注意保持原有功能和测试通过。
```

### 场景三：Bug 定位与修复

**Prompt 示例：**

```markdown
用户在注册时遇到 "Email already exists" 错误，但数据库中并无该邮箱记录。

请：
1. 检查注册 API 的邮箱验证逻辑
2. 查看数据库查询语句
3. 检查是否存在大小写敏感问题
4. 验证请求参数是否正确处理
5. 定位问题并提供修复方案
```

### 场景四：性能优化

**Prompt 示例：**

```markdown
分析 src/services/userService.ts 中 getUserById 函数的性能问题：

当前实现：
[粘贴代码]

已知问题：
- 数据库查询 N+1 问题
- 缺少缓存
- 响应时间 > 500ms

请：
1. 识别性能瓶颈
2. 优化查询逻辑
3. 添加缓存机制
4. 提供优化前后的性能对比
```

### 场景五：测试生成

**Prompt 示例：**

```markdown
为 src/utils/validator.ts 中的 validateEmail 函数生成完整的测试用例：

要求：
- 使用 Vitest 框架
- 测试用例命名清晰
- 包含正常情况和边界情况
- 包含错误情况
- 测试覆盖率 > 90%
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

#### 问题 2：扩展不显示

**解决方案**：
1. 检查扩展是否启用
2. 尝试重新加载窗口
3. 检查控制台错误

### 配置问题

#### 问题 3：API 密钥无效

**解决方案**：
1. 确认 API 密钥正确
2. 检查 API 额度
3. 确认 API 权限
4. 尝试重新生成密钥

#### 问题 4：模型连接失败

**解决方案**：
1. 确认模型服务运行
2. 检查端口配置
3. 确认防火墙设置
4. 查看日志

### 使用问题

#### 问题 5：任务执行失败

**解决方案**：
1. 简化任务描述
2. 提供更多上下文
3. 分解为多个小任务
4. 开始新的对话

#### 问题 6：文件操作失败

**解决方案**：
1. 确认文件路径正确
2. 检查文件权限
3. 确认目录存在
4. 手动创建目录

---

## 选型建议

### 何时选择 Roo Code

- ✅ 预算有限，无法承担每月 $10-25 的订阅费用
- ✅ 数据隐私要求严格，代码不能上传至第三方
- ✅ 需要高度自定义的 AI 行为
- ✅ 希望使用本地开源模型（如 Llama、Qwen）
- ✅ 欣赏开源文化，愿意参与社区建设
- ✅ 技术能力强，愿意配置和维护

### 何时选择其他工具

- ❌ 希望获得开箱即用的优秀体验 → 选择 Cursor
- ❌ 企业采购、需要 SLA 支持 → 选择 Copilot Business
- ❌ 主要进行深度代码分析和重构 → 选择 Claude Code
- ❌ 缺乏配置意愿，需要"傻瓜式"工具 → 选择 Copilot

### 混合使用策略

许多开发者采用组合策略：

```
日常简单任务：GitHub Copilot（实时补全）
复杂任务：Roo Code（自主代理）
深度分析：Claude Code（代码理解）
原型开发：Cursor（快速迭代）
隐私项目：Roo Code + Ollama（完全本地）
```

### 快速入门路径

1. **第一天**：安装 Roo Code，配置 API 密钥
2. **第一周**：完成几个小型任务，熟悉工具调用
3. **第二周**：尝试本地模型（Ollama）
4. **第三周**：自定义系统提示和工作流
5. **第四周**：探索 MCP 扩展

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| 官方文档 | https://roocode.com/docs |
| GitHub 仓库 | https://github.com/saoudrizwan/roo-code |
| VS Code Market | https://marketplace.visualstudio.com/items?itemName=saoudrizwan.roo-code |
| JetBrains Plugin | https://plugins.jetbrains.com/plugin/roo-code |
| MCP 协议 | https://modelcontextprotocol.io |

### 社区资源

| 资源 | 说明 |
|------|------|
| Discord | 官方社区讨论 |
| GitHub Issues | Bug 反馈和功能请求 |
| Reddit | r/roocode |
| Twitter | 官方更新 |

### 学习资源

```markdown
# 推荐学习路径

## 入门
1. 官方文档快速入门
2. 基础配置教程
3. 第一个任务实战

## 进阶
1. 自定义 Prompt 工程
2. MCP 服务器配置
3. 工作流自动化

## 高级
1. 本地模型部署
2. 企业级配置
3. 自定义 MCP 开发
```

### API 提供商

| 提供商 | 定价页面 |
|------|---------|
| Anthropic | https://www.anthropic.com/pricing |
| OpenAI | https://openai.com/api/pricing |
| Google | https://ai.google.dev/pricing |
| DeepSeek | https://api-docs.deepseek.com |
| Ollama | https://ollama.com |
| LM Studio | https://lmstudio.ai |

---

> [!SUCCESS]
> Roo Code 作为开源 AI 编程助手的代表，以其高度可定制性、本地运行能力和零成本优势，成为 Copilot 和 Cursor 的有力替代方案。对于注重隐私、预算有限或热爱开源文化的开发者，Roo Code 是 vibecoding 工作流中不可或缺的工具。

---

## Roo Code 高级配置与优化

### 1. 性能优化配置

```json
// 性能优化配置
{
  "roo-code": {
    "performance": {
      "maxConcurrentTools": 3,
      "toolTimeout": 60,
      "responseTimeout": 120,
      "cacheEnabled": true,
      "cacheSize": "100MB",
      "streamResponses": true
    },
    "context": {
      "maxFiles": 50,
      "maxTokens": 100000,
      "priorityFiles": ["*.ts", "*.tsx", "*.js"],
      "excludePatterns": ["node_modules/**", "dist/**", "*.test.ts"]
    }
  }
}
```

### 2. 安全强化配置

```json
// 安全配置
{
  "roo-code": {
    "security": {
      "allowedDirectories": ["${workspaceFolder}"],
      "blockedCommands": [
        "rm -rf /*",
        "format C:",
        "del /f /s /q"
      ],
      "requireConfirmation": {
        "delete": true,
        "bash": true,
        "network": true
      },
      "auditLogging": {
        "enabled": true,
        "path": ".roo/audit.log",
        "includeContext": true
      }
    }
  }
}
```

### 3. 高级 MCP 配置

```json
// MCP 服务器高级配置
{
  "roo-code": {
    "mcpServers": {
      "filesystem": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-filesystem"],
        "allowedDirectories": [
          "${workspaceFolder}/src",
          "${workspaceFolder}/tests"
        ],
        "maxFileSize": "10MB"
      },
      "github": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-github"],
        "env": {
          "GITHUB_TOKEN": "${GITHUB_TOKEN}"
        },
        "scopes": ["repo", "read:user"]
      },
      "slack": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-slack"],
        "env": {
          "SLACK_BOT_TOKEN": "${SLACK_BOT_TOKEN}"
        }
      },
      "database": {
        "command": "node",
        "args": ["./mcp-servers/database-server.js"],
        "env": {
          "DATABASE_URL": "${DATABASE_URL}"
        }
      }
    }
  }
}
```

### 4. 自定义 MCP 服务器

```typescript
// mcp-servers/database-server.ts
import { McpServer } from '@modelcontextprotocol/sdk/server';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio';
import { z } from 'zod';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

const server = new McpServer({
  name: 'Database Server',
  version: '1.0.0',
});

// 查询工具
server.tool(
  'db-query',
  'Execute a database query',
  {
    query: z.string().describe('SQL query to execute'),
    params: z.array(z.any()).optional(),
  },
  async ({ query, params = [] }) => {
    try {
      const result = await prisma.$queryRawUnsafe(query, ...params);
      return {
        content: [{ type: 'text', text: JSON.stringify(result, null, 2) }],
      };
    } catch (error) {
      return {
        content: [{ type: 'text', text: `Error: ${error.message}` }],
        isError: true,
      };
    }
  }
);

// 查询用户
server.tool(
  'db-get-user',
  'Get a user by ID or email',
  {
    identifier: z.string().describe('User ID or email'),
  },
  async ({ identifier }) => {
    const user = identifier.includes('@')
      ? await prisma.user.findUnique({ where: { email: identifier } })
      : await prisma.user.findUnique({ where: { id: identifier } });
    
    if (!user) {
      return { content: [{ type: 'text', text: 'User not found' }], isError: true };
    }
    
    return {
      content: [{ type: 'text', text: JSON.stringify(user, null, 2) }],
    };
  }
);

// 列出表
server.tool('db-list-tables', 'List all database tables', {}, async () => {
  const tables = await prisma.$queryRaw`
    SELECT table_name 
    FROM information_schema.tables 
    WHERE table_schema = 'public'
  `;
  return {
    content: [{ type: 'text', text: JSON.stringify(tables, null, 2) }],
  };
});

const transport = new StdioServerTransport();
server.run(transport);
```

---

## Roo Code 企业级部署

### 1. 企业配置示例

```yaml
# .roo/enterprise-config.yaml

# 组织配置
organization:
  name: "Acme Corporation"
  domain: "acme.com"
  sso:
    enabled: true
    provider: "okta"
    clientId: "${OKTA_CLIENT_ID}"
    clientSecret: "${OKTA_CLIENT_SECRET}"

# 用户管理
users:
  defaultRole: "developer"
  roles:
    - name: "admin"
      permissions: ["*"]
    - name: "developer"
      permissions:
        - "read"
        - "write"
        - "execute"
        - "test"
    - name: "viewer"
      permissions:
        - "read"

# 安全策略
security:
  audit:
    enabled: true
    retentionDays: 90
    exportFormat: "json"
  dataProtection:
    encryptionEnabled: true
    maskSensitiveData: true
    sensitivePatterns:
      - "password"
      - "secret"
      - "token"
      - "apiKey"

# 成本控制
costControl:
  enabled: true
  budgetPerUser: 100.00
  budgetPerMonth: 5000.00
  alertThresholds:
    - 50%
    - 75%
    - 90%

# 集成
integrations:
  github:
    enabled: true
    autoSyncRules: true
  slack:
    enabled: true
    channel: "#devops"
  jira:
    enabled: true
    projectKey: "DEV"
```

### 2. 团队规则模板

```markdown
# .roo/team-rules/frontend.md

## React 组件规范

### 组件结构
```
ComponentName/
├── ComponentName.tsx        # 主组件
├── ComponentName.module.css  # CSS 模块
├── ComponentName.test.tsx   # 测试
├── index.ts                 # 导出
└── types.ts                 # 类型定义
```

### 命名规范
- 组件：PascalCase
- 文件：kebab-case
- 测试：ComponentName.test.tsx
- CSS 类：kebab-case

### 代码规范
- 使用函数组件
- Props 接口命名：ComponentNameProps
- 使用 React.FC 或直接函数声明
- 组件最大行数：200 行
- 提取自定义 Hooks

### 性能要求
- 大列表使用虚拟滚动
- 使用 React.memo 优化
- useCallback 和 useMemo 适当使用
- 懒加载组件

### 测试要求
- 覆盖率 > 80%
- 测试用户交互
- 测试边界情况
```

### 3. 审计日志配置

```typescript
// .roo/audit-logger.ts

interface AuditEntry {
  timestamp: string;
  userId: string;
  action: string;
  target: string;
  details?: Record<string, unknown>;
  duration: number;
  success: boolean;
  error?: string;
}

class AuditLogger {
  private entries: AuditEntry[] = [];
  private flushInterval: number;
  
  constructor(flushInterval = 60000) {
    this.flushInterval = flushInterval;
    setInterval(() => this.flush(), this.flushInterval);
  }
  
  log(entry: Omit<AuditEntry, 'timestamp'>) {
    this.entries.push({
      ...entry,
      timestamp: new Date().toISOString(),
    });
  }
  
  async flush() {
    if (this.entries.length === 0) return;
    
    const entries = [...this.entries];
    this.entries = [];
    
    await fetch('/api/audit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(entries),
    });
  }
}

export const auditLogger = new AuditLogger();
```

---

## Roo Code 最佳实践

### 1. 项目结构最佳实践

```
最佳项目结构示例：
project/
├── .roo/
│   ├── rules/                    # 项目规则
│   │   ├── default.mdc
│   │   ├── typescript.mdc
│   │   └── react.mdc
│   ├── templates/                # 任务模板
│   │   ├── component.json
│   │   └── api.json
│   ├── workflows/                # 工作流
│   │   └── ci.yml
│   └── mcp.json                 # MCP 配置
├── src/
│   ├── components/
│   ├── hooks/
│   ├── services/
│   └── utils/
├── tests/
├── docs/
└── package.json
```

### 2. Prompt 工程最佳实践

```markdown
# 高效 Prompt 设计原则

## 1. 结构化 Prompt

```markdown
## 角色
你是一个 [角色]，专注于 [领域]。

## 上下文
[提供项目背景和相关信息]

## 任务
[明确描述需要完成的任务]

## 约束
- [约束条件 1]
- [约束条件 2]

## 输出
[期望的输出格式]

## 示例
[提供参考示例]
```

## 2. 渐进式 Prompt

```
阶段 1：基础需求
"创建一个 React 组件显示用户信息"

阶段 2：详细需求
"基于刚才的组件，添加：
- 头像显示
- 加载状态
- 错误处理"

阶段 3：优化需求
"现在优化这个组件：
- 使用 React.memo
- 添加 TypeScript 类型
- 包含测试"
```

## 3. 上下文管理

```
好实践：
- 使用 @ 引用文件
- 提供相关的代码片段
- 说明项目的技术栈

差实践：
- 不提供任何上下文
- 引用太多无关文件
- 描述过于笼统
```
```

### 3. 安全最佳实践

```markdown
# 使用 Roo Code 的安全准则

## 敏感操作
- 所有删除操作需要确认
- 网络请求需要确认
- Bash 命令需要确认

## 敏感信息
- 永远不在对话中分享密钥
- 使用环境变量存储敏感信息
- 定期轮换 API 密钥

## 代码审查
- AI 生成的代码必须审查
- 特别关注安全相关代码
- 运行安全扫描工具

## 本地模型
- 处理敏感代码时使用本地模型
- 确保 Ollama 正确配置
- 验证数据不出本地
```

### 4. 性能优化技巧

```markdown
# 提升 Roo Code 性能

## 1. 减少上下文
- 只引用相关文件
- 避免加载整个目录
- 及时清空对话

## 2. 选择合适的模型
- 简单任务用轻量模型
- 复杂任务用强大模型
- 平衡速度和效果

## 3. 优化请求
- 简洁的 Prompt
- 明确的指令
- 避免重复

## 4. 本地缓存
- 使用 Ollama 缓存
- 减少 API 调用
- 提升响应速度
```
```

---

## Roo Code 与 DevOps 集成

### 1. Docker 集成

```dockerfile
# Dockerfile 示例
FROM node:20-alpine AS base

# Install dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci --only=production

# Development
FROM base AS development
WORKDIR /app
COPY package.json package-lock.json* ./
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]

# Build
FROM development AS builder
WORKDIR /app
RUN npm run build

# Production
FROM base AS production
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: development
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:secret@db:5432/myapp
    depends_on:
      - db
      - redis

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

### 2. Kubernetes 集成

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
      - name: myapp
        image: myregistry/myapp:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: myapp-secrets
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### 3. CI/CD 流水线

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v4
        
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
        
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}
        
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  test:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        
      - name: Run tests
        run: npm test -- --coverage
        
      - name: Upload coverage
        uses: codecov/codecov-action@v4

  deploy-staging:
    needs: [build, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - name: Deploy to staging
        run: |
          kubectl config use-context staging
          kubectl set image deployment/myapp app=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/myapp

  deploy-production:
    needs: [build, test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Deploy to production
        run: |
          kubectl config use-context production
          kubectl set image deployment/myapp app=${{ needs.build.outputs.image-tag }}
          kubectl rollout status deployment/myapp
          kubectl rollout status deployment/myapp --timeout=5m
```

### 4. 监控与日志

```typescript
// src/monitoring/index.ts
import { metrics, Counter, Histogram, Logger } from '@opentelemetry/api';
import { NodeSDK } from '@opentelemetry/sdk-node';
import { JaegerExporter } from '@opentelemetry/exporter-jaeger';
import { HttpInstrumentation } from '@opentelemetry/instrumentation-http';
import { ExpressInstrumentation } from '@opentelemetry/instrumentation-express';

const logger = new Logger({ name: 'monitoring' });

// 初始化 OpenTelemetry
const sdk = new NodeSDK({
  traceExporter: new JaegerExporter({
    endpoint: process.env.JAEGER_ENDPOINT,
  }),
  instrumentations: [
    new HttpInstrumentation(),
    new ExpressInstrumentation(),
  ],
});

sdk.start();

// 自定义指标
const httpRequestsTotal = new Counter('http_requests_total', {
  description: 'Total number of HTTP requests',
});

const httpRequestDuration = new Histogram('http_request_duration_seconds', {
  description: 'HTTP request duration in seconds',
  boundaries: [0.01, 0.05, 0.1, 0.5, 1, 5],
});

export { httpRequestsTotal, httpRequestDuration, logger };
```

```typescript
// src/logging/index.ts
import { createLogger, transports, format } from 'winston';

export const logger = createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json()
  ),
  transports: [
    new transports.Console({
      format: format.combine(
        format.colorize(),
        format.simple()
      ),
    }),
    new transports.File({
      filename: 'logs/error.log',
      level: 'error',
    }),
    new transports.File({
      filename: 'logs/combined.log',
    }),
  ],
});

// 结构化日志
export function logRequest(req: Request, res: Response, duration: number) {
  logger.info('HTTP Request', {
    method: req.method,
    url: req.url,
    status: res.statusCode,
    duration,
    userAgent: req.headers['user-agent'],
    ip: req.ip,
  });
}
```

---

## Roo Code 与云服务集成

### 1. AWS 集成

```typescript
// src/services/aws.ts
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { LambdaClient, InvokeCommand } from '@aws-sdk/client-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { SQSClient, SendMessageCommand } from '@aws-sdk/client-sqs';

const s3Client = new S3Client({ region: process.env.AWS_REGION });
const lambdaClient = new LambdaClient({ region: process.env.AWS_REGION });
const dynamoClient = new DynamoDBClient({ region: process.env.AWS_REGION });
const sqsClient = new SQSClient({ region: process.env.AWS_REGION });

export const s3Service = {
  async upload(key: string, body: Buffer, contentType: string) {
    const command = new PutObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key,
      Body: body,
      ContentType: contentType,
    });
    await s3Client.send(command);
    return `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`;
  },
  
  async download(key: string) {
    const command = new GetObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key,
    });
    const response = await s3Client.send(command);
    return response.Body;
  },
};

export const lambdaService = {
  async invoke(functionName: string, payload: object) {
    const command = new InvokeCommand({
      FunctionName: functionName,
      Payload: JSON.stringify(payload),
    });
    const response = await lambdaClient.send(command);
    return JSON.parse(new TextDecoder().decode(response.Payload));
  },
};

export const sqsService = {
  async sendMessage(queueUrl: string, message: object) {
    const command = new SendMessageCommand({
      QueueUrl: queueUrl,
      MessageBody: JSON.stringify(message),
    });
    return sqsClient.send(command);
  },
};
```

### 2. Azure 集成

```typescript
// src/services/azure.ts
import { BlobServiceClient } from '@azure/storage-blob';
import { CosmosClient } from '@azure/cosmos';
import { QueueServiceClient } from '@azure/storage-queue';

const blobServiceClient = new BlobServiceClient(
  process.env.AZURE_STORAGE_CONNECTION_STRING!
);

export const azureBlobService = {
  async upload(containerName: string, blobName: string, data: Buffer) {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlockBlobClient(blobName);
    await blobClient.uploadData(data);
    return blobClient.url;
  },
  
  async download(containerName: string, blobName: string) {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlobClient(blobName);
    const downloadResponse = await blobClient.download();
    return downloadResponse.blobBody;
  },
};

// Cosmos DB
const cosmosClient = new CosmosClient(process.env.AZURE_COSMOS_CONNECTION!);
const database = cosmosClient.database(process.env.AZURE_COSMOS_DATABASE!);
const container = database.container(process.env.AZURE_COSMOS_CONTAINER!);

export const cosmosService = {
  async create(item: object) {
    const { resource } = await container.items.create(item);
    return resource;
  },
  
  async read(id: string, partitionKey: string) {
    const { resource } = await container.item(id, partitionKey).read();
    return resource;
  },
  
  async query(query: string) {
    const { resources } = await container.items
      .query(query)
      .fetchAll();
    return resources;
  },
};
```

### 3. Google Cloud 集成

```typescript
// src/services/gcp.ts
import { Storage } from '@google-cloud/storage';
import { BigQuery } from '@google-cloud/bigquery';
import { PubSub } from '@google-cloud/pubsub';

const storage = new Storage();
const bigquery = new BigQuery();
const pubsub = new PubSub();

export const gcsService = {
  async upload(bucketName: string, fileName: string, data: Buffer) {
    const bucket = storage.bucket(bucketName);
    const file = bucket.file(fileName);
    await file.save(data);
    return `gs://${bucketName}/${fileName}`;
  },
  
  async download(bucketName: string, fileName: string) {
    const bucket = storage.bucket(bucketName);
    const file = bucket.file(fileName);
    const [exists] = await file.exists();
    if (!exists) throw new Error('File not found');
    return file.download();
  },
};

export const bigqueryService = {
  async query(sql: string, params?: object[]) {
    const options = {
      query: sql,
      location: 'US',
      params: params,
    };
    const [job] = await bigquery.createJob(options);
    const [rows] = await job.getQueryResults();
    return rows;
  },
  
  async insert(datasetId: string, tableId: string, rows: object[]) {
    const dataset = bigquery.dataset(datasetId);
    const table = dataset.table(tableId);
    await table.insert(rows);
  },
};

export const pubsubService = {
  async publish(topicName: string, message: object) {
    const topic = pubsub.topic(topicName);
    await topic.publishMessage({
      data: Buffer.from(JSON.stringify(message)),
    });
  },
  
  async subscribe(subscriptionName: string, callback: (message: object) => Promise<void>) {
    const subscription = pubsub.subscription(subscriptionName);
    subscription.on('message', async (message) => {
      try {
        await callback(JSON.parse(message.data.toString()));
        message.ack();
      } catch (error) {
        console.error('Error processing message:', error);
        message.nack();
      }
    });
  },
};
```

---

## Roo Code 高级使用案例

### 1. 微服务架构开发

```markdown
# 使用 Roo Code 开发微服务架构

## 项目结构

```
microservices/
├── user-service/
│   ├── src/
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── models/
│   │   ├── routes/
│   │   └── index.ts
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
├── order-service/
├── payment-service/
└── notification-service/
```

## Roo Code 辅助开发

用户: 为这个微服务项目创建统一的错误处理中间件

Roo Code 输出：

```typescript
// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  statusCode: number;
  isOperational: boolean;
  
  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super(message, 400);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401);
  }
}

export class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403);
  }
}

export function errorHandler(
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: {
        message: err.message,
        statusCode: err.statusCode,
      },
    });
  }
  
  // 未知错误
  console.error('Unexpected error:', err);
  
  return res.status(500).json({
    success: false,
    error: {
      message: 'Internal server error',
      statusCode: 500,
    },
  });
}
```

## 服务间通信

```typescript
// src/services/grpc-client.ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';

const PROTO_PATH = './proto/user.proto';

const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
});

const userProto = grpc.loadPackageDefinition(packageDefinition) as any;

export class UserServiceClient {
  private client: any;
  
  constructor(address: string) {
    this.client = new userProto.UserService(
      address,
      grpc.credentials.createInsecure()
    );
  }
  
  async getUser(id: string): Promise<User> {
    return new Promise((resolve, reject) => {
      this.client.GetUser({ id }, (err: Error, response: User) => {
        if (err) reject(err);
        else resolve(response);
      });
    });
  }
  
  async createUser(data: CreateUserDto): Promise<User> {
    return new Promise((resolve, reject) => {
      this.client.CreateUser(data, (err: Error, response: User) => {
        if (err) reject(err);
        else resolve(response);
      });
    });
  }
}
```
```

### 2. 事件驱动架构

```typescript
// src/events/index.ts
import { EventEmitter } from 'events';

export const eventBus = new EventEmitter();

export const Events = {
  USER_CREATED: 'user:created',
  USER_UPDATED: 'user:updated',
  USER_DELETED: 'user:deleted',
  ORDER_CREATED: 'order:created',
  ORDER_COMPLETED: 'order:completed',
  PAYMENT_PROCESSED: 'payment:processed',
} as const;

export type EventPayload = {
  [Events.USER_CREATED]: { id: string; email: string; name: string };
  [Events.USER_UPDATED]: { id: string; updates: Partial<User> };
  [Events.USER_DELETED]: { id: string };
  [Events.ORDER_CREATED]: { id: string; userId: string; items: OrderItem[] };
  [Events.ORDER_COMPLETED]: { id: string; total: number };
  [Events.PAYMENT_PROCESSED]: { orderId: string; success: boolean };
};

// 事件处理器
export const handlers = {
  async onUserCreated(payload: EventPayload[typeof Events.USER_CREATED]) {
    // 发送欢迎邮件
    await emailService.sendWelcomeEmail(payload.email);
    
    // 创建默认设置
    await settingsService.createDefault(payload.id);
    
    // 记录审计日志
    await auditLogService.log('user_created', payload);
  },
  
  async onOrderCompleted(payload: EventPayload[typeof Events.ORDER_COMPLETED]) {
    // 更新统计数据
    await analyticsService.trackOrder(payload);
    
    // 发送通知
    await notificationService.sendOrderConfirmation(payload.id);
    
    // 触发后续流程
    await fulfillmentService.processOrder(payload.id);
  },
};

// 注册处理器
Object.entries(handlers).forEach(([event, handler]) => {
  eventBus.on(event, handler);
});

// 发布事件
export function publishEvent<K extends keyof typeof Events>(
  event: K,
  payload: EventPayload[typeof Events[K]]
) {
  eventBus.emit(event, payload);
}
```

### 3. GraphQL API 开发

```typescript
// src/graphql/schema.ts
import { gql } from 'apollo-server-express';

export const typeDefs = gql`
  type User {
    id: ID!
    email: String!
    name: String!
    avatar: String
    role: UserRole!
    createdAt: DateTime!
    updatedAt: DateTime!
    posts: [Post!]!
    comments: [Comment!]!
  }
  
  enum UserRole {
    USER
    ADMIN
    MODERATOR
  }
  
  type Post {
    id: ID!
    title: String!
    content: String!
    published: Boolean!
    author: User!
    comments: [Comment!]!
    createdAt: DateTime!
    updatedAt: DateTime!
  }
  
  type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
    createdAt: DateTime!
  }
  
  input CreateUserInput {
    email: String!
    password: String!
    name: String!
  }
  
  input UpdateUserInput {
    name: String
    avatar: String
  }
  
  type Query {
    users: [User!]!
    user(id: ID!): User
    me: User
    posts: [Post!]!
    post(id: ID!): Post
  }
  
  type Mutation {
    createUser(input: CreateUserInput!): User!
    updateUser(id: ID!, input: UpdateUserInput!): User!
    deleteUser(id: ID!): Boolean!
    publishPost(id: ID!): Post!
  }
  
  type Subscription {
    userCreated: User!
    postPublished: Post!
  }
`;

export const resolvers = {
  Query: {
    users: () => userService.findAll(),
    user: (_, { id }) => userService.findById(id),
    me: (_, __, { user }) => user,
    posts: () => postService.findAll(),
    post: (_, { id }) => postService.findById(id),
  },
  
  Mutation: {
    createUser: (_, { input }) => userService.create(input),
    updateUser: (_, { id, input }) => userService.update(id, input),
    deleteUser: (_, { id }) => userService.delete(id),
    publishPost: (_, { id }) => postService.publish(id),
  },
  
  User: {
    posts: (user) => postService.findByAuthor(user.id),
    comments: (user) => commentService.findByAuthor(user.id),
  },
  
  Post: {
    author: (post) => userService.findById(post.authorId),
    comments: (post) => commentService.findByPost(post.id),
  },
  
  Subscription: {
    userCreated: {
      subscribe: (_, __, { pubsub }) =>
        pubsub.asyncIterableIterator(['USER_CREATED']),
    },
    postPublished: {
      subscribe: (_, __, { pubsub }) =>
        pubsub.asyncIterableIterator(['POST_PUBLISHED']),
    },
  },
};
```

---

## 附录：Roo Code 命令参考

### CLI 命令

```bash
# 常用 CLI 命令
roo-code --help                    # 显示帮助
roo-code init                      # 初始化项目配置
roo-code config                    # 打开配置编辑器
roo-code models                   # 列出可用模型
roo-code models set <name>         # 设置默认模型
roo-code history                  # 查看对话历史
roo-code history clear             # 清除历史
roo-code template list            # 列出模板
roo-code template create <name>    # 创建模板
roo-code workflow list            # 列出工作流
roo-code workflow run <name>       # 运行工作流
roo-code mcp list                 # 列出 MCP 服务器
roo-code mcp add <name>           # 添加 MCP 服务器
roo-code mcp remove <name>        # 移除 MCP 服务器
roo-code stats                    # 显示使用统计
roo-code export <file>            # 导出对话
roo-code import <file>           # 导入对话
```

### 配置命令

```bash
# 配置管理
roo-code config get <key>         # 获取配置值
roo-code config set <key> <value> # 设置配置值
roo-code config reset              # 重置配置
roo-code config export <file>     # 导出配置
roo-code config import <file>     # 导入配置
```

### 调试命令

```bash
# 调试命令
roo-code debug on                  # 开启调试模式
roo-code debug off                 # 关闭调试模式
roo-code debug log <file>         # 导出日志
roo-code debug inspect            # 检查状态
```

---

## Roo Code 完整安装与配置指南

### 系统要求

#### 环境要求

| 配置项 | 最低要求 | 推荐配置 |
|--------|----------|----------|
| 操作系统 | macOS 10.15 / Windows 10 / Linux Ubuntu 18.04 | macOS 12+ / Windows 11 |
| VS Code | 1.75.0+ | 最新版本 |
| JetBrains | 2023.1+ | 最新版本 |
| 内存 | 4GB RAM | 8GB+ RAM |
| 网络 | 能访问 AI API | 稳定快速连接 |

### VS Code 安装

#### 方法一：扩展市场

```markdown
# 安装步骤

1. 打开 VS Code
2. 按 Cmd/Ctrl + P 打开命令面板
3. 输入以下命令：
   ext install saoudrizwan.roo-code

4. 等待安装完成
5. 重启 VS Code
6. 按 Cmd/Ctrl + Shift + I 打开 Roo Code
```

#### 方法二：VSIX 文件

```bash
# 1. 下载最新版本
# 访问 https://github.com/saoudrizwan/roo-code/releases

# 2. 安装 VSIX
code --install-extension roo-code-*.vsix

# 3. 重启 VS Code
```

#### 方法三：命令行安装

```bash
# macOS
brew install --cask roo-code

# Linux
curl -L https://github.com/saoudrizwan/roo-code/releases/latest/download/roo-code-linux-x64.vsix -o roo-code.vsix
code --install-extension roo-code.vsix
```

### JetBrains 安装

```markdown
# JetBrains IDE 中安装 Roo Code

1. 打开 JetBrains IDE
2. 进入 Settings → Plugins
3. 搜索 "Roo Code" 或 "Roo Code"
4. 点击 Install
5. 重启 IDE
6. Tools → Roo Code → Open Settings
```

### API Key 配置

#### Anthropic Claude API

```json
// .vscode/settings.json
{
  "roo-code": {
    "apiProvider": "anthropic",
    "apiKey": "sk-ant-xxxxxxxxxxxxxxxxxxxx",
    "model": "claude-sonnet-4-20250514"
  }
}
```

获取方式：
1. 访问 https://console.anthropic.com/
2. 注册并登录
3. 进入 API Keys 页面
4. 创建新的 API Key

#### OpenAI API

```json
{
  "roo-code": {
    "apiProvider": "openai",
    "apiKey": "sk-xxxxxxxxxxxxxxxxxxxxxxxx",
    "model": "gpt-4o"
  }
}
```

#### OpenRouter（聚合方案）

```json
{
  "roo-code": {
    "apiProvider": "openrouter",
    "apiKey": "sk-or-xxxxx",
    "model": "anthropic/claude-sonnet-4"
  }
}
```

### 本地模型配置

#### Ollama 配置

```json
{
  "roo-code": {
    "apiProvider": "ollama",
    "apiBase": "http://localhost:11434/v1",
    "model": "llama3.2"
  }
}
```

Ollama 常用命令：
```bash
# 拉取模型
ollama pull llama3.2          # 通用
ollama pull qwen2.5:14b         # 中文支持好
ollama pull codellama:34b       # 代码能力强
ollama pull mistral:7b           # 通用能力强

# 查看已安装模型
ollama list

# 启动服务
ollama serve

# 运行模型
ollama run llama3.2 "Hello"
```

#### LM Studio 配置

```json
{
  "roo-code": {
    "apiProvider": "openai-compatible",
    "apiBase": "http://localhost:1234/v1",
    "model": "mistral-7b-instruct-v0.3"
  }
}
```

### 代理配置

```json
{
  "roo-code": {
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

## Roo Code 快捷键大全

### VS Code 默认快捷键

| 功能 | macOS | Windows/Linux | 说明 |
|------|--------|----------------|------|
| 打开 Roo Code | `Cmd + Shift + I` | `Ctrl + Shift + I` | 打开对话面板 |
| 新建任务 | `Cmd + Shift + R` | `Ctrl + Shift + R` | 开始新任务 |
| 接受建议 | `Tab` | `Tab` | 接受 AI 建议 |
| 拒绝建议 | `Esc` | `Esc` | 拒绝建议 |
| 终端执行 | `Cmd + Shift + T` | `Ctrl + Shift + T` | 执行终端命令 |

### JetBrains 快捷键

| 功能 | 默认快捷键 | 说明 |
|------|------------|------|
| 打开 Roo Code | `Shift + Ctrl + I` | 打开对话面板 |
| 新建任务 | `Shift + Ctrl + R` | 开始新任务 |
| 接受建议 | `Tab` | 接受建议 |

### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+i",
    "command": "roo-code.open",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "roo-code.newTask",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "roo-code.acceptSuggestion",
    "when": "suggestionVisible"
  },
  {
    "key": "cmd+shift+d",
    "command": "roo-code.rejectSuggestion",
    "when": "suggestionVisible"
  }
]
```

---

## Roo Code 高级配置

### 任务批准模式

Roo Code 提供三种任务批准模式：

```json
{
  "roo-code": {
    "autoApprove": "ask",
    "autoApproveExportFunctions": true,
    "autoApproveIntelligent": false,
    "askForDrafts": true,
    "maxAutoApproveEdits": 5
  }
}
```

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **Auto Approve** | 所有操作自动执行 | 简单任务、熟悉代码库 |
| **Ask After Every Tool Call** | 每个工具调用前询问 | 谨慎开发、数据敏感 |
| **Never Approve** | 完全手动模式 | 审计需求、完全控制 |

### 成本控制配置

```json
{
  "roo-code": {
    "costControl": {
      "enabled": true,
      "maxCostPerSession": 5.0,
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

### 上下文配置

```json
{
  "roo-code": {
    "context": {
      "maxFiles": 50,
      "maxTokens": 100000,
      "priorityFiles": [
        "*.ts",
        "*.tsx",
        "*.js",
        "*.jsx"
      ],
      "excludePatterns": [
        "node_modules/**",
        "dist/**",
        "*.test.ts",
        "coverage/**"
      ]
    }
  }
}
```

### 性能配置

```json
{
  "roo-code": {
    "performance": {
      "maxConcurrentTools": 3,
      "toolTimeout": 60,
      "responseTimeout": 120,
      "cacheEnabled": true,
      "cacheSize": "100MB",
      "streamResponses": true
    }
  }
}
```

---

## Roo Code 云服务集成

### AWS 集成

#### S3 服务

```typescript
// src/services/aws/s3.ts
import { S3Client, PutObjectCommand, GetObjectCommand, DeleteObjectCommand } from '@aws-sdk/client-s3';

const s3Client = new S3Client({ region: process.env.AWS_REGION });

export const s3Service = {
  async upload(key: string, body: Buffer, contentType: string): Promise<string> {
    const command = new PutObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key,
      Body: body,
      ContentType: contentType,
    });
    await s3Client.send(command);
    return `https://${process.env.S3_BUCKET}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`;
  },

  async download(key: string): Promise<Buffer> {
    const command = new GetObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key,
    });
    const response = await s3Client.send(command);
    const chunks: Buffer[] = [];
    for await (const chunk of response.Body as any) {
      chunks.push(chunk);
    }
    return Buffer.concat(chunks);
  },

  async delete(key: string): Promise<void> {
    const command = new DeleteObjectCommand({
      Bucket: process.env.S3_BUCKET,
      Key: key,
    });
    await s3Client.send(command);
  },

  async list(prefix: string): Promise<string[]> {
    const command = new ListObjectsV2Command({
      Bucket: process.env.S3_BUCKET,
      Prefix: prefix,
    });
    const response = await s3Client.send(command);
    return response.Contents?.map(obj => obj.Key || '') || [];
  }
};
```

#### Lambda 服务

```typescript
// src/services/aws/lambda.ts
import { LambdaClient, InvokeCommand } from '@aws-sdk/client-lambda';

const lambdaClient = new LambdaClient({ region: process.env.AWS_REGION });

export const lambdaService = {
  async invoke<T = any>(functionName: string, payload: object): Promise<T> {
    const command = new InvokeCommand({
      FunctionName: functionName,
      Payload: JSON.stringify(payload),
      InvocationType: 'RequestResponse',
    });
    const response = await lambdaClient.send(command);
    const decoder = new TextDecoder();
    return JSON.parse(decoder.decode(response.Payload));
  },

  async invokeAsync(functionName: string, payload: object): Promise<void> {
    const command = new InvokeCommand({
      FunctionName: functionName,
      Payload: JSON.stringify(payload),
      InvocationType: 'Event',
    });
    await lambdaClient.send(command);
  }
};
```

#### DynamoDB 服务

```typescript
// src/services/aws/dynamodb.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand, DeleteCommand } from '@aws-sdk/lib-dynamodb';

const ddbClient = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(ddbClient);

export const dynamoService = {
  async put<T>(tableName: string, item: T): Promise<void> {
    await docClient.send(new PutCommand({
      TableName: tableName,
      Item: item,
    }));
  },

  async get<T>(tableName: string, key: Record<string, any>): Promise<T | null> {
    const result = await docClient.send(new GetCommand({
      TableName: tableName,
      Key: key,
    }));
    return result.Item as T || null;
  },

  async query<T>(
    tableName: string,
    params: {
      indexName?: string;
      keyCondition: string;
      expressionValues: Record<string, any>;
      expressionNames?: Record<string, string>;
      filter?: string;
    }
  ): Promise<T[]> {
    const result = await docClient.send(new QueryCommand({
      TableName: tableName,
      IndexName: params.indexName,
      KeyConditionExpression: params.keyCondition,
      ExpressionAttributeValues: params.expressionValues,
      ExpressionAttributeNames: params.expressionNames,
      FilterExpression: params.filter,
    }));
    return result.Items as T[] || [];
  },

  async delete(tableName: string, key: Record<string, any>): Promise<void> {
    await docClient.send(new DeleteCommand({
      TableName: tableName,
      Key: key,
    }));
  }
};
```

### Azure 集成

#### Blob Storage 服务

```typescript
// src/services/azure/blob.ts
import { BlobServiceClient, StorageSharedKeyCredential } from '@azure/storage-blob';

const blobServiceClient = new BlobServiceClient(
  process.env.AZURE_STORAGE_CONNECTION_STRING!
);

export const azureBlobService = {
  async upload(containerName: string, blobName: string, data: Buffer, contentType: string): Promise<string> {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlockBlobClient(blobName);
    await blobClient.uploadData(data, { blobHTTPHeaders: { blobContentType: contentType } });
    return blobClient.url;
  },

  async download(containerName: string, blobName: string): Promise<Buffer> {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlobClient(blobName);
    const downloadResponse = await blobClient.download();
    const downloaded = await streamToBuffer(downloadResponse.readableStreamBody);
    return downloaded;
  },

  async delete(containerName: string, blobName: string): Promise<void> {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobClient = containerClient.getBlobClient(blobName);
    await blobClient.delete();
  },

  async list(containerName: string, prefix?: string): Promise<string[]> {
    const containerClient = blobServiceClient.getContainerClient(containerName);
    const blobs: string[] = [];
    for await (const blob of containerClient.listBlobsFlat({ prefix })) {
      blobs.push(blob.name);
    }
    return blobs;
  }
};

async function streamToBuffer(stream: NodeJS.ReadableStream): Promise<Buffer> {
  const chunks: Buffer[] = [];
  for await (const chunk of stream as any) {
    chunks.push(Buffer.from(chunk));
  }
  return Buffer.concat(chunks);
}
```

#### Cosmos DB 服务

```typescript
// src/services/azure/cosmos.ts
import { CosmosClient, Container } from '@azure/cosmos';

const cosmosClient = new CosmosClient(process.env.AZURE_COSMOS_CONNECTION!);
const container: Container = cosmosClient
  .database(process.env.AZURE_COSMOS_DATABASE!)
  .container(process.env.AZURE_COSMOS_CONTAINER!);

export const cosmosService = {
  async create<T extends object>(item: T): Promise<T> {
    const { resource } = await container.items.create(item);
    return resource as T;
  },

  async read<T>(id: string, partitionKey: string): Promise<T | null> {
    try {
      const { resource } = await container.item(id, partitionKey).read();
      return resource as T || null;
    } catch {
      return null;
    }
  },

  async update<T extends { id: string }>(item: T): Promise<T> {
    const { resource } = await container.item(item.id, item.id).replace(item);
    return resource as T;
  },

  async delete(id: string, partitionKey: string): Promise<void> {
    await container.item(id, partitionKey).delete();
  },

  async query<T>(
    query: string,
    parameters?: { name: string; value: any }[]
  ): Promise<T[]> {
    const { resources } = await container.items.query({
      query,
      parameters: parameters?.map(p => ({ ...p, value: p.value }))
    }).fetchAll();
    return resources as T[];
  }
};
```

### Google Cloud 集成

#### Cloud Storage 服务

```typescript
// src/services/gcp/storage.ts
import { Storage } from '@google-cloud/storage';

const storage = new Storage({
  projectId: process.env.GCP_PROJECT_ID,
  keyFilename: process.env.GCP_KEY_FILE,
});

export const gcsService = {
  async upload(bucketName: string, fileName: string, data: Buffer, options?: {
    contentType?: string;
    metadata?: Record<string, string>;
  }): Promise<string> {
    const bucket = storage.bucket(bucketName);
    const file = bucket.file(fileName);
    await file.save(data, {
      contentType: options?.contentType,
      metadata: options?.metadata,
    });
    return `gs://${bucketName}/${fileName}`;
  },

  async download(bucketName: string, fileName: string): Promise<Buffer> {
    const bucket = storage.bucket(bucketName);
    const file = bucket.file(fileName);
    const [exists] = await file.exists();
    if (!exists) throw new Error('File not found');
    const [data] = await file.download();
    return data;
  },

  async delete(bucketName: string, fileName: string): Promise<void> {
    const bucket = storage.bucket(bucketName);
    await bucket.file(fileName).delete();
  },

  async list(bucketName: string, prefix?: string): Promise<string[]> {
    const bucket = storage.bucket(bucketName);
    const [files] = await bucket.getFiles({ prefix });
    return files.map(file => file.name);
  }
};
```

#### BigQuery 服务

```typescript
// src/services/gcp/bigquery.ts
import { BigQuery } from '@google-cloud/bigquery';

const bigquery = new BigQuery({
  projectId: process.env.GCP_PROJECT_ID,
  keyFilename: process.env.GCP_KEY_FILE,
});

export const bigqueryService = {
  async query<T = any>(
    sql: string,
    params?: Record<string, any>,
    options?: { location?: string; timeout?: number }
  ): Promise<T[]> {
    const [rows] = await bigquery.query({
      query: sql,
      params: params,
      location: options?.location,
      timeout: options?.timeout,
    });
    return rows as T[];
  },

  async insert(datasetId: string, tableId: string, rows: object[]): Promise<void> {
    await bigquery
      .dataset(datasetId)
      .table(tableId)
      .insert(rows);
  },

  async createDataset(datasetId: string): Promise<void> {
    await bigquery.createDataset(datasetId);
  },

  async createTable(
    datasetId: string,
    tableId: string,
    schema: { name: string; type: string; mode?: string }[]
  ): Promise<void> {
    await bigquery
      .dataset(datasetId)
      .createTable(tableId, {
        schema: schema.reduce((acc, col) => {
          acc[col.name] = { type: col.type, mode: col.mode as any };
          return acc;
        }, {} as any),
      });
  }
};
```

---

## Roo Code 企业级部署

### 企业配置示例

```yaml
# .roo/enterprise/enterprise.yaml

organization:
  name: "Acme Corporation"
  domain: "acme.com"
  
  sso:
    enabled: true
    provider: "okta"
    client_id: "${OKTA_CLIENT_ID}"
    client_secret: "${OKTA_CLIENT_SECRET}"
    
users:
  default_role: "developer"
  
  roles:
    - name: "admin"
      permissions:
        - "*"
    - name: "developer"
      permissions:
        - "read"
        - "write"
        - "execute"
        - "test"
    - name: "viewer"
      permissions:
        - "read"

security:
  audit:
    enabled: true
    retention_days: 90
    export_format: "json"
    
  data_protection:
    encryption_enabled: true
    mask_sensitive_data: true
    sensitive_patterns:
      - "password"
      - "secret"
      - "token"
      - "api_key"
      - "credit_card"

cost_control:
  enabled: true
  budget_per_user: 100.00
  budget_per_month: 5000.00
  alert_thresholds:
    - 0.5
    - 0.75
    - 0.9

integrations:
  github:
    enabled: true
    auto_sync_rules: true
  slack:
    enabled: true
    channel: "#devops"
  jira:
    enabled: true
    project_key: "DEV"
```

### 团队规则示例

#### TypeScript 规范

```markdown
# .roo/team-rules/typescript.md

## TypeScript 规范

### 基本规则
- 严格模式：true
- 禁用 any，使用 unknown 代替
- 所有函数必须有返回类型
- 优先使用 interface

### 命名规范
- 变量/函数：camelCase
- 类型/接口/类：PascalCase
- 常量：UPPER_SNAKE_CASE
- 文件：kebab-case.ts

### 类型定义
```typescript
// ✅ 好的示例
interface User {
  id: string;
  name: string;
  email: Email;
}

type Email = string & { __brand: 'email' };

// ❌ 差的示例
interface User {
  id: any;
  name: any;
}
```

### 错误处理
```typescript
// ✅ 好的示例
async function fetchUser(id: string): Promise<Result<User, Error>> {
  try {
    const user = await db.users.findById(id);
    return { ok: true, value: user };
  } catch (error) {
    return { ok: false, error: error as Error };
  }
}
```
```

#### React 规范

```markdown
# .roo/team-rules/react.md

## React 规范

### 组件结构
```
ComponentName/
├── ComponentName.tsx        # 主组件
├── ComponentName.module.css  # CSS 模块
├── ComponentName.test.tsx   # 测试
├── index.ts                 # 导出
└── types.ts                 # 类型定义
```

### 命名规范
- 组件文件：PascalCase.tsx
- Hooks：useCamelCase.ts
- 工具函数：camelCase.ts
- 常量：UPPER_SNAKE_CASE

### 代码规范
- 使用函数组件
- 使用 Hooks 而非类组件
- Props 接口命名：ComponentNameProps
- 组件最大行数：200 行

### 性能优化
- 列表渲染使用 key
- 使用 React.memo 优化重渲染
- 合理使用 useMemo/useCallback
- 大列表使用虚拟滚动
```

### Git 规范

```markdown
# .roo/team-rules/git.md

## Git 提交规范

### 提交信息格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type 类型
- feat：新功能
- fix：Bug 修复
- docs：文档更新
- style：格式调整
- refactor：代码重构
- test：测试相关
- chore：构建/工具

### 示例
```
feat(auth): 添加双因素认证

- 添加 TOTP 支持
- 集成 Google Authenticator
- 添加 QR 码生成

Closes #123
```

### 分支命名
- feature/xxx-功能描述
- fix/xxx-问题描述
- refactor/xxx-重构内容
- hotfix/xxx-紧急修复
```

---

## Roo Code 与 DevOps 工具链集成

### Docker 集成

```dockerfile
# Dockerfile 示例
FROM node:20-alpine AS base

WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production

# Development
FROM base AS development
COPY package*.json ./
RUN npm ci
COPY . .
CMD ["npm", "run", "dev"]

# Builder
FROM development AS builder
RUN npm run build

# Production
FROM base AS production
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY package*.json ./

USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Kubernetes 配置

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - name: myapp
          image: myregistry/myapp:latest
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: myapp-secrets
                  key: database-url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
```

### CI/CD 流水线

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Test
        run: npm test -- --coverage

      - name: Build
        run: npm run build

      - name: Build Docker image
        run: |
          docker build -t ${{ env.IMAGE_NAME }}:${{ github.sha }} .
          docker tag ${{ env.IMAGE_NAME }}:${{ github.sha }} ${{ env.IMAGE_NAME }}:latest

      - name: Push to Registry
        if: github.event_name == 'push'
        run: |
          echo ${{ secrets.GITHUB_TOKEN }} | docker login ${{ env.REGISTRY }} -u ${{ github.actor }} --password-stdin
          docker push ${{ env.IMAGE_NAME }}:${{ github.sha }}
          docker push ${{ env.IMAGE_NAME }}:latest

  deploy:
    needs: build
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl config use-context production
          kubectl set image deployment/myapp app=${{ env.IMAGE_NAME }}:${{ github.sha }}
          kubectl rollout status deployment/myapp
```

---

## Roo Code 常见问题与解决方案

### FAQ 1：VS Code 扩展冲突

**症状**：Roo Code 与其他扩展冲突。

**解决方案**：
```markdown
1. 禁用冲突扩展
   - 逐一禁用可能冲突的扩展
   - 找出具体是哪个扩展冲突

2. 更新扩展
   - 确保所有扩展都是最新版本
   - 更新 VS Code 本身

3. 重置扩展
   - Cmd/Ctrl + Shift + P
   - 输入 "Reload Window"
```

### FAQ 2：API 连接失败

**症状**：无法连接到 AI API。

**解决方案**：
```markdown
1. 检查 API Key
   - 确认 API Key 正确
   - 检查 API Key 权限

2. 检查网络
   - 确认网络连接正常
   - 检查代理设置

3. 测试 API
   - 使用 curl 测试 API
   - 确认 API 服务正常

4. 查看日志
   - Roo Code 输出面板
   - 详细错误信息
```

### FAQ 3：工具执行失败

**症状**：工具（如 Terminal）无法执行。

**解决方案**：
```markdown
1. 检查权限
   - Roo Code → 设置 → 工具权限
   - 启用所需工具

2. 检查路径
   - 确认工具路径正确
   - 确认在 PATH 中

3. 手动测试
   - 在终端中手动执行命令
   - 确认命令本身可用
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
```

### FAQ 5：Ollama 模型加载慢

**症状**：Ollama 模型加载很慢。

**解决方案**：
```markdown
1. 选择更小的模型
   - llama3.2 > codellama
   - mistral > llama3

2. 优化 Ollama
   ```bash
   # 设置并发
   OLLAMA_NUM_PARALLEL=4 ollama serve
   
   # 限制内存使用
   OLLAMA_MAX_LOADED_MODELS=1 ollama serve
   ```

3. 使用 GPU 加速
   - NVIDIA GPU 配置 CUDA
   - 大幅提升推理速度
```

---

## Roo Code 选型建议

### 个人开发者

**推荐：Roo Code + Claude Sonnet**

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

**推荐：Roo Code + 团队规范**

```markdown
# 团队配置

1. 统一 API 密钥
   - 使用团队 API Key
   - 控制成本分摊

2. 共享配置
   - 统一的 .roo 目录
   - 共享系统提示

3. 自定义工具
   - 团队专用工具
   - 标准化工作流
```

### 隐私敏感场景

**推荐：Roo Code + Ollama**

```markdown
# 完全本地方案

1. Ollama 部署
   ```bash
   ollama pull llama3.2
   ollama pull codellama
   ```

2. Roo Code 配置
   ```json
   {
     "apiProvider": "ollama",
     "apiBase": "http://localhost:11434/v1",
     "model": "llama3.2"
   }
   ```

3. 优势
   - 数据完全本地
   - 零 API 成本
   - 完全离线可用
```

### 企业用户

**推荐：Roo Code Enterprise 方案**

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

## Roo Code 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 10.15+ / Windows 10+ / Linux |
| 内存 | 4GB RAM |
| 磁盘空间 | 200MB 可用空间 |
| 网络 | 稳定连接（使用云端模型）或离线（使用本地模型） |
| 基础 IDE | VS Code 1.75+ 或 JetBrains IDE 2023.2+ |

#### 使用本地模型的额外要求

| 配置项 | 推荐 |
|--------|------|
| 内存 | 16GB+ RAM |
| 磁盘 | 10GB+（模型文件） |
| GPU | NVIDIA（可选，推荐） |

### 安装步骤详解

#### VS Code 安装

```bash
# 方法一：VS Code Marketplace
# 1. 打开 VS Code
# 2. 按 Cmd/Ctrl + P
# 3. 输入：ext install rooveterinaryinc.roo-cline
# 4. 点击安装

# 方法二：命令行安装
code --install-extension rooveterinaryinc.roo-cline

# 方法三：下载 VSIX
# 访问 https://marketplace.visualstudio.com/
# 下载 .vsix 文件后安装
```

#### JetBrains IDE 安装

```markdown
# 1. 打开 JetBrains IDE
# 2. Settings → Plugins → Marketplace
# 3. 搜索 "Roo Code"
# 4. 点击 Install
# 5. 重启 IDE
```

#### CLI 安装

```bash
# Homebrew (macOS/Linux)
brew install roo-code

# npm 全局安装
npm install -g roo-code-cli

# 直接下载二进制
wget https://github.com/RooVetGit/roo-code/releases/latest/roo-code-linux
chmod +x roo-code-linux
./roo-code-linux
```

### 首次配置

#### 1. API Key 配置

```markdown
# Roo Code 支持多种 API 提供商

## Anthropic (Claude)
1. 访问 https://console.anthropic.com/
2. 获取 API Key
3. 设置 → Roo Code → API Provider → Anthropic
4. 填入 API Key

## OpenAI (GPT-4)
1. 访问 https://platform.openai.com/
2. 创建 API Key
3. 设置 → Provider → OpenAI

## OpenRouter
1. 访问 https://openrouter.ai/
2. 获取 API Key
3. 支持多种模型聚合

## 本地模型 (Ollama/LM Studio)
1. 运行 Ollama/LM Studio
2. 设置 → Provider → Local
3. 配置端点和模型
```

#### 2. 基础设置

```json
// .vscode/settings.json
{
  "roo.code": {
    "apiProvider": "anthropic",
    "model": "claude-sonnet-4-20250514",
    "maxTokens": 8192,
    "temperature": 0.7,
    
    // 自动批准设置
    "autoApproval": {
      "enabled": true,
      "excludePatterns": ["rm -rf", "DROP TABLE"]
    },
    
    // 本地模型配置
    "localModels": {
      "enabled": true,
      "provider": "ollama",
      "endpoint": "http://localhost:11434/v1/chat/completions",
      "model": "llama3"
    }
  }
}
```

#### 3. MCP 配置

```json
{
  "roo.code.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git"]
    },
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

### Roo Code 快捷键大全

#### 核心快捷键

| 功能分类 | 功能 | macOS | Windows/Linux | 说明 |
|---------|------|-------|---------------|------|
| **Roo Code 面板** | 打开面板 | `Cmd/Ctrl + Shift + V` | 同 | 打开 Roo Code 侧边栏 |
| | 发送消息 | `Enter` | `Enter` | 发送消息 |
| | 多行输入 | `Shift + Enter` | `Shift + Enter` | 换行 |
| | 清空输入 | `Cmd/Ctrl + K` | `Ctrl + K` | 清空输入框 |
| **工具操作** | 批准工具 | `Enter` | `Enter` | 批准执行 |
| | 拒绝工具 | `Cmd/Ctrl + D` | `Ctrl + D` | 拒绝当前 |
| | 全部批准 | `Cmd/Ctrl + Shift + A` | `Ctrl + Shift + A` | 批准所有 |
| | 停止执行 | `Esc` | `Esc` | 停止生成 |
| **任务管理** | 新任务 | `Cmd/Ctrl + N` | `Ctrl + N` | 新建任务 |
| | 继续任务 | `Cmd/Ctrl + R` | `Ctrl + R` | 继续上次 |
| | 任务历史 | `Cmd/Ctrl + H` | `Ctrl + H` | 查看历史 |

#### 特殊命令

| 命令 | 功能 |
|------|------|
| `/clear` | 清空对话上下文 |
| `/model <name>` | 切换模型 |
| `/cost` | 显示当前消耗 |
| `/token` | 显示 token 统计 |
| `/export` | 导出对话记录 |

---

## Roo Code AI 提示词工程深度指南

### 任务描述最佳实践

#### 1. 清晰的任务定义

```markdown
# ❌ 不明确的描述
"修复 bug"

# ✅ 明确的描述
"修复用户登录后无法访问 /dashboard 页面的问题
- 问题文件：src/pages/Dashboard.tsx
- 错误信息：Uncaught TypeError: Cannot read properties of undefined
- 预期行为：登录后自动跳转到 /dashboard"
```

#### 2. 分层式任务结构

```markdown
# 复杂任务分层

## 目标
重构 src/services/userService.ts，添加缓存支持

## 背景
当前服务每次请求都查询数据库，高并发下性能差

## 约束
- 使用 Redis 作为缓存
- 缓存过期时间 5 分钟
- 保持现有接口不变

## 验证
1. 运行现有测试
2. 性能测试：响应时间 < 100ms
3. 压力测试：1000 并发
```

#### 3. 代码示例引导

```markdown
# 提供参考代码

参考以下实现模式：

```typescript
// 现有的错误处理模式
async function handleError(error: Error) {
  logger.error(error.message, { stack: error.stack });
  return { success: false, error: error.message };
}

// 需要的实现
// 参考上面的模式，实现一个日志服务
```
```

### 迭代式开发

```markdown
# 第一轮：基础功能
"实现一个 Todo 列表组件，包含：
- 添加任务
- 删除任务
- 标记完成
- localStorage 持久化"

# 第二轮：增强功能
"基于上面的实现，添加：
- 任务分类（工作/个人）
- 截止日期
- 优先级
- drag & drop 排序"

# 第三轮：优化体验
"添加动画效果（Framer Motion）：
- 任务添加滑入
- 删除滑出
- 拖拽反馈"
```

### 技术栈提示词模板

#### React + TypeScript

```markdown
"创建一个 React 函数组件 DataTable，要求：

组件属性：
```typescript
interface Props<T> {
  data: T[];
  columns: Column<T>[];
  onRowClick?: (row: T) => void;
  loading?: boolean;
  emptyText?: string;
}

interface Column<T> {
  key: keyof T;
  header: string;
  render?: (value: T[keyof T], row: T) => ReactNode;
}
```

功能：
- 列排序
- 行选择
- 分页
- 加载状态
- 空状态

技术栈：
- React 18
- TypeScript
- Tailwind CSS"
```

#### Node.js + Express

```markdown
"创建 RESTful API 路由 /api/products

端点：
- GET / - 列表（分页、筛选）
- GET /:id - 详情
- POST / - 创建
- PUT /:id - 更新
- DELETE /:id - 删除

要求：
- 异步处理
- 输入验证（Zod）
- 错误中间件
- 统一的响应格式"
```

---

## Roo Code 与现代技术栈集成

### Next.js 14 App Router

**app/products/page.tsx**

```typescript
import { Suspense } from 'react';
import { ProductList } from '@/components/ProductList';
import { getProducts } from '@/lib/products';

interface PageProps {
  searchParams: Promise<{ page?: string; category?: string }>;
}

export default async function ProductsPage({ searchParams }: PageProps) {
  const params = await searchParams;
  const page = parseInt(params.page || '1', 10);
  const category = params.category || 'all';
  
  const { products, total, pages } = await getProducts({ page, category });
  
  return (
    <main className="container mx-auto py-8">
      <h1 className="text-3xl font-bold mb-8">产品列表</h1>
      
      <Suspense fallback={<ProductListSkeleton />}>
        <ProductList 
          products={products} 
          total={total}
          currentPage={page}
          totalPages={pages}
        />
      </Suspense>
    </main>
  );
}
```

### Prisma + PostgreSQL

**lib/prisma.ts**

```typescript
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma = global.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' 
    ? ['query', 'error', 'warn'] 
    : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  global.prisma = prisma;
}
```

**lib/products.ts**

```typescript
import { prisma } from './prisma';
import { cache } from 'react';

export interface GetProductsParams {
  page?: number;
  limit?: number;
  category?: string;
  search?: string;
}

export const getProducts = cache(async (params: GetProductsParams) => {
  const { page = 1, limit = 12, category, search } = params;
  const skip = (page - 1) * limit;
  
  const where = {
    ...(category && category !== 'all' && { category }),
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' as const } },
        { description: { contains: search, mode: 'insensitive' as const } },
      ],
    }),
  };
  
  const [products, total] = await Promise.all([
    prisma.product.findMany({
      where,
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      include: { category: true },
    }),
    prisma.product.count({ where }),
  ]);
  
  return {
    products,
    total,
    page,
    pages: Math.ceil(total / limit),
  };
});
```

### 状态管理 - Zustand

**stores/useProductStore.ts**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface Product {
  id: string;
  name: string;
  price: number;
  stock: number;
}

interface ProductState {
  products: Product[];
  selectedId: string | null;
  filter: string;
  
  setProducts: (products: Product[]) => void;
  addProduct: (product: Product) => void;
  updateProduct: (id: string, updates: Partial<Product>) => void;
  removeProduct: (id: string) => void;
  setSelectedId: (id: string | null) => void;
  setFilter: (filter: string) => void;
  
  filteredProducts: () => Product[];
  selectedProduct: () => Product | undefined;
}

export const useProductStore = create<ProductState>()(
  persist(
    (set, get) => ({
      products: [],
      selectedId: null,
      filter: '',
      
      setProducts: (products) => set({ products }),
      
      addProduct: (product) => 
        set((state) => ({ products: [...state.products, product] })),
      
      updateProduct: (id, updates) =>
        set((state) => ({
          products: state.products.map((p) =>
            p.id === id ? { ...p, ...updates } : p
          ),
        })),
      
      removeProduct: (id) =>
        set((state) => ({
          products: state.products.filter((p) => p.id !== id),
          selectedId: state.selectedId === id ? null : state.selectedId,
        })),
      
      setSelectedId: (selectedId) => set({ selectedId }),
      setFilter: (filter) => set({ filter }),
      
      filteredProducts: () => {
        const { products, filter } = get();
        if (!filter) return products;
        return products.filter((p) =>
          p.name.toLowerCase().includes(filter.toLowerCase())
        );
      },
      
      selectedProduct: () => {
        const { products, selectedId } = get();
        return products.find((p) => p.id === selectedId);
      },
    }),
    {
      name: 'product-storage',
    }
  )
);
```

---

## Roo Code 常见问题与解决方案

### FAQ 1：API Key 配置问题

**症状**：无法连接到 AI 服务。

**解决方案**：
```markdown
1. 验证 API Key
   - 检查 Key 是否正确
   - 确认没有多余空格

2. 检查网络
   - ping api.anthropic.com
   - 检查代理设置

3. 查看日志
   - 输出面板有详细错误
   - 查找具体错误码

4. 重置配置
   - 删除 .vscode/settings.json 中的 roo 配置
   - 重新配置
```

### FAQ 2：工具执行无响应

**症状**：工具调用后没有结果。

**解决方案**：
```markdown
1. 检查自动批准
   - 设置 → Roo Code → Auto Approve
   - 尝试启用自动批准

2. 手动批准
   - 按 Enter 批准每个工具

3. 查看工具日志
   - 输出面板显示执行详情

4. 检查文件权限
   - 确保有写入权限
```

### FAQ 3：本地模型连接失败

**症状**：Ollama 连接失败或响应慢。

**解决方案**：
```markdown
1. 确认 Ollama 运行
   Bash: curl http://localhost:11434/api/tags

2. 检查模型
   Bash: ollama list

3. 配置端点
   ```json
   {
     "roo.code.localModels": {
       "provider": "ollama",
       "endpoint": "http://localhost:11434/v1/chat/completions",
       "model": "llama3"
     }
   }
   ```

4. LM Studio 特殊配置
   - 启用 "Local Server"
   - 设置 CORS
```

### FAQ 4：上下文过长

**症状**：响应变慢或失败。

**解决方案**：
```markdown
1. 减少上下文
   - 使用 /clear 重置
   - 减少引用文件

2. 使用压缩模型
   - Claude Haiku
   - GPT-4o-mini

3. 分段处理
   - 将大任务拆分
```

### FAQ 5：MCP 服务器不工作

**症状**：MCP 工具不可用。

**解决方案**：
```markdown
1. 检查配置
   ```json
   {
     "roo.code.mcpServers": {
       "server-name": {
         "command": "npx",
         "args": ["-y", "@scope/package"]
       }
     }
   }
   ```

2. 安装依赖
   Bash: npx -y @anthropic/mcp-server-xxx

3. 重启
   - 保存配置后重启 VS Code
```

---

## Roo Code 与竞品深度对比

### 功能矩阵

| 功能 | Roo Code | Cline | Copilot | Cursor |
|------|----------|-------|----------|---------|
| **开源** | ✅ | ✅ | ❌ | ❌ |
| **本地模型** | ✅ | ✅ | ❌ | ❌ |
| **MCP 协议** | ✅ | ✅ | ❌ | 部分 |
| **多模型支持** | ✅ | ✅ | ❌ | ✅ |
| **JetBrains 支持** | ✅ | ✅ | ✅ | ❌ |
| **自动工具** | ✅ | ✅ | ❌ | ✅ |
| **成本控制** | ✅ | ✅ | ❌ | ❌ |

### 成本效益分析

| 方案 | 月成本 | 优势 | 劣势 |
|------|--------|------|------|
| **Roo + Ollama** | $0 | 零成本、隐私 | 需本地资源 |
| **Roo + Claude** | $15-50 | 高质量 | API 费用 |
| **Copilot Pro** | $10 | 集成好 | 订阅制 |
| **Cursor Pro** | $20 | 功能全 | 订阅制 |

### 场景化选择

| 场景 | 推荐 | 理由 |
|------|------|------|
| 完全离线 | Roo + Ollama | 零网络依赖 |
| 隐私敏感 | Roo + Ollama | 代码不出本机 |
| 成本敏感 | Roo + DeepSeek | API 极便宜 |
| 高质量需求 | Roo + Claude | 顶级模型 |
| JetBrains 用户 | Roo Code | 原生支持 |
| 快速上手 | Copilot | 开箱即用 |

---

## Roo Code 企业级部署指南

### 企业架构设计

```yaml
# 企业部署架构

┌─────────────────────────────────────────────────┐
│              Enterprise Architecture               │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────────────────────────────────────┐│
│  │            SSO / SAML Gateway                ││
│  └─────────────────────────────────────────────┘│
│                        │                         │
│                        ▼                         │
│  ┌─────────────────────────────────────────────┐│
│  │          API Gateway (成本控制)               ││
│  │  - 速率限制                                   ││
│  │  - 使用统计                                   ││
│  │  - 模型路由                                   ││
│  └─────────────────────────────────────────────┘│
│                        │                         │
│          ┌─────────────┼─────────────┐          │
│          ▼             ▼             ▼          │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐│
│  │  Claude API │ │   DeepSeek │ │   Ollama   ││
│  │  (关键代码)  │ │  (常规代码) │ │  (实验)    ││
│  └─────────────┘ └─────────────┘ └─────────────┘│
│                                                  │
└─────────────────────────────────────────────────┘
```

### 团队配置管理

```json
// .vscode/team-settings.json
{
  "roo.code": {
    "apiProvider": "custom",
    "endpoint": "https://api.company.com/ai",
    "apiKey": "${ROO_API_KEY}",
    
    "teamRules": {
      "enabled": true,
      "rulesFile": ".roo/team-rules.md",
      "autoEnforce": true
    },
    
    "costControl": {
      "monthlyLimit": 1000,
      "alertThreshold": 0.8,
      "modelLimits": {
        "claude-opus": 100,
        "claude-sonnet": 500,
        "deepseek-coder": 1000
      }
    },
    
    "audit": {
      "enabled": true,
      "logFile": ".roo/audit.log",
      "includeContent": false
    }
  }
}
```

### 安全配置

```markdown
# 企业安全配置

## 1. 数据隔离
- 使用私有 API 端点
- 配置白名单 IP
- 启用审计日志

## 2. 访问控制
- SSO 集成
- 基于角色的权限
- API Key 管理

## 3. 合规
- SOC 2 Type II
- GDPR 合规
- 数据保留策略

## 4. 监控
- 使用量仪表板
- 异常检测
- 成本告警
```

### 团队工作流

```markdown
# 团队最佳实践

## 代码审查
1. 使用 Roo Code 生成代码
2. 团队代码规范检查
3. 人工审查
4. 合并前测试

## 知识共享
1. 创建 .roo/team-rules.md
2. 共享常用提示词模板
3. 记录最佳实践

## 培训
1. 定期分享会
2. 最佳案例库
3. 内部文档
```

---

## Roo Code AI 提示词深度工程

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

### 输出要求
- 完整可运行的代码
- 类型定义
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
[粘贴错误堆栈跟踪]

## 相关代码
```[语言]
[相关代码片段]
```

## 环境信息
- 操作系统：[OS 版本]
- 运行时：[版本]
- 依赖版本：[关键依赖版本]

## 复现步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 预期行为
[描述期望的正确行为]

## 修复要求
- 不能破坏现有功能
- 添加边界检查
- 包含回归测试
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
1. [问题1]
2. [问题2]
3. [问题3]

## 重构约束
- 保持 API 兼容性
- 不改变外部行为
- 测试覆盖率 ≥ 80%
- 遵循 [代码规范]

## 验证要求
- 运行现有测试
- 性能基准测试
```

### 2. 提示词优化技巧

#### 上下文构建

```markdown
# 技巧1：精确的文件引用

## ❌ 模糊引用
"在用户模块中添加删除功能"

## ✅ 精确引用
"在 src/features/user/services/UserService.ts 的 UserService 类中，
添加 deleteUser(id: string): Promise<void> 方法。
参考同文件中的 getUserById 方法的实现模式。"

# 技巧2：提供完整的上下文

## ❌ 缺少上下文
"添加表单验证"

## ✅ 完整的上下文
"为 src/components/UserForm.tsx 添加表单验证。
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

## ❌ 缺少约束
"优化查询性能"

## ✅ 明确的约束
"优化 src/services/UserService.ts 中的 findAll 方法性能。

约束条件：
1. 响应时间必须 < 100ms
2. 不能改变现有 API 接口
3. 必须使用 Prisma ORM
4. 考虑添加 Redis 缓存

当前问题：
- N+1 查询问题
- 缺少索引
- 没有缓存"
```

#### 迭代开发

```markdown
# 技巧4：分步骤迭代

## 第一轮：基础实现
"创建一个基础的 Todo 组件，包含：
- 添加任务
- 删除任务
- 标记完成"

## 第二轮：添加功能
"基于上面的 Todo 组件，添加：
- 任务分类
- 优先级
- 截止日期"

## 第三轮：持久化
"将 Todo 组件改为使用 localStorage 持久化"

## 第四轮：优化
"优化 Todo 组件：
- React.memo
- useCallback
- 虚拟滚动"
```

### 3. 高级使用场景

#### 大型项目迁移

```markdown
# 场景：React 项目升级

## 项目背景
- 项目规模：500+ 组件
- 技术栈：React 15 → React 18
- 迁移周期：3 个月

## 协作流程

### 阶段 1：评估
"分析当前项目：
1. 统计组件数量和复杂度
2. 识别关键依赖
3. 评估迁移风险
4. 制定迁移计划"

### 阶段 2：基础设施
"准备基础设施：
1. 配置 React 18 开发环境
2. 设置双版本运行模式
3. 创建自动化测试套件"

### 阶段 3：组件迁移
"按优先级迁移组件：
1. 核心组件
2. 业务逻辑组件
3. 页面级组件
4. 工具类组件

每次迁移：
- 分析组件依赖
- 迁移到新语法
- 更新导入路径
- 运行测试验证"

### 阶段 4：清理
"迁移完成后清理：
1. 移除遗留代码
2. 清理双重兼容代码
3. 优化性能"
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
   
2. 错误处理
   - 异常处理
   - 日志记录

3. 代码质量
   - 代码重复
   - 复杂度
   - 可维护性

## 输出格式
- 发现的问题列表
- 严重程度评级
- 修复建议
- 代码评分（1-10）
```

### 4. 专业领域模板

#### React 开发

```markdown
# React 开发提示词

## 组件开发
"创建一个 React 函数组件，要求：
- 使用 TypeScript
- 完整的 Props 接口定义
- 遵循项目规范
- 包含状态管理
- 添加 JSDoc 注释"

## Hooks 开发
"创建一个自定义 React Hook，要求：
- 提取可复用逻辑
- 适当的依赖管理
- 清理副作用
- TypeScript 类型支持"

## 状态管理
"使用 Zustand 创建一个 store，要求：
- 完整的类型定义
- 持久化支持
- 适当的 actions 组织"
```

#### Node.js 后端

```markdown
# Node.js 后端开发提示词

## API 开发
"创建一个 RESTful API 模块，要求：
- 使用 Express + TypeScript
- 完整的类型定义
- 输入验证（Zod）
- 错误处理中间件
- 统一响应格式"

## 数据库操作
"使用 Prisma 创建一个数据服务，要求：
- 完整的类型定义
- 错误处理
- 事务支持"

## 认证授权
"实现 JWT 认证中间件，要求：
- Token 验证
- 过期处理
- 权限检查"
```

#### DevOps 配置

```markdown
# DevOps 配置提示词

## Docker 配置
"为项目创建 Dockerfile，要求：
- 多阶段构建
- 生产优化
- 最小化镜像
- 健康检查"

## CI/CD 配置
"创建 GitHub Actions 流水线，要求：
- 代码检查
- 测试
- 构建
- 部署"

## Kubernetes 配置
"创建 K8s 部署配置，要求：
- 副本管理
- 资源限制
- 健康检查"
```

---

## Roo Code 与现代技术栈集成

### 1. 前端技术栈

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
    <div className="user-card">
      {avatar ? (
        <img src={avatar} alt={name} />
      ) : (
        <div className="avatar-placeholder">
          {name.charAt(0).toUpperCase()}
        </div>
      )}
      
      <h3>{name}</h3>
      <p>{email}</p>
      
      <div className="actions">
        <button onClick={handleEdit}>Edit</button>
        <button onClick={handleDelete} disabled={isLoading}>
          {isLoading ? 'Deleting...' : 'Delete'}
        </button>
      </div>
    </div>
  );
};
```

#### Vue 3 + TypeScript

```typescript
// Vue 3 组合式 API 开发模板
<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue';

interface Props {
  userId: string;
  editable?: boolean;
}

const props = withDefaults(defineProps<Props>(), {
  editable: true,
});

const emit = defineEmits<{
  (e: 'update', data: UserData): void;
  (e: 'delete', id: string): void;
}>();

const user = ref<UserData | null>(null);
const isLoading = ref(false);
const error = ref<string | null>(null);

const displayName = computed(() => {
  return user.value?.name ?? 'Unknown User';
});

const canEdit = computed(() => {
  return props.editable && user.value !== null;
});

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

onMounted(() => {
  fetchUser();
});

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
        <button @click="handleDelete">Delete</button>
      </div>
    </template>
  </div>
</template>
```

### 2. 后端技术栈

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

class UserRole(str, Enum):
    USER = "user"
    ADMIN = "admin"
    MODERATOR = "moderator"

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=2, max_length=100)
    role: UserRole = UserRole.USER

class UserCreate(UserBase):
    password: str = Field(..., min_length=8, max_length=100)
    
    @validator('password')
    def validate_password(cls, v):
        if not any(c.isupper() for c in v):
            raise ValueError('Password must contain uppercase')
        if not any(c.islower() for c in v):
            raise ValueError('Password must contain lowercase')
        if not any(c.isdigit() for c in v):
            raise ValueError('Password must contain digit')
        return v

class UserResponse(UserBase):
    id: str
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True

@app.get("/users", response_model=List[UserResponse])
async def list_users():
    # Implementation here
    pass

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: str):
    # Implementation here
    pass

@app.post("/users", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate):
    # Implementation here
    pass
```

#### Django + DRF

```python
# Django REST Framework 开发模板
from rest_framework import serializers, viewsets, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from django.contrib.auth.models import User
from django.contrib.auth.hashers import make_password

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 
                  'last_name', 'is_active', 'date_joined']
        read_only_fields = ['id', 'date_joined']
    
    def validate_email(self, value):
        email = value.lower()
        user = User.objects.filter(email__iexact=email)
        if self.instance:
            user = user.exclude(pk=self.instance.pk)
        if user.exists():
            raise serializers.ValidationError(
                "A user with this email already exists."
            )
        return value
    
    def create(self, validated_data):
        validated_data['password'] = make_password(
            validated_data.get('password', ''))
        return super().create(validated_data)

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]
    
    @action(detail=False, methods=['get'])
    def me(self, request):
        serializer = self.get_serializer(request.user)
        return Response(serializer.data)
```

### 3. 数据库集成

#### Prisma Schema

```prisma
// Prisma Schema 模板
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

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
  
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  
  @@index([email])
  @@index([username])
  @@index([role])
}

model Post {
  id          String     @id @default(cuid())
  title       String
  slug        String     @unique
  content     String
  excerpt     String?
  status      PostStatus @default(DRAFT)
  viewCount   Int        @default(0)
  
  author      User       @relation(fields: [authorId], references: [id])
  authorId    String
  category    Category?  @relation(fields: [categoryId], references: [id])
  categoryId  String?
  tags        Tag[]
  comments    Comment[]
  likes       Like[]
  
  publishedAt DateTime?
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  
  @@index([authorId])
  @@index([categoryId])
  @@index([status])
}

model Category {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]
  
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
  
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  post      Post    @relation(fields: [postId], references: [id])
  postId   String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies  Comment[] @relation("CommentReplies")
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([authorId])
  @@index([postId])
  @@index([parentId])
}

model Like {
  id      String @id @default(cuid())
  
  user    User   @relation(fields: [userId], references: [id])
  userId  String
  post    Post  @relation(fields: [postId], references: [id])
  postId  String
  
  @@unique([userId, postId])
  @@index([userId])
  @@index([postId])
}
```

---

## Roo Code 快捷键完全手册

### 默认快捷键

| 功能分类 | 功能 | macOS | Windows/Linux | 说明 |
|---------|------|-------|---------------|------|
| **Roo Code 面板** | 打开面板 | `Cmd/Ctrl + Shift + V` | 同 | 打开侧边栏 |
| | 发送消息 | `Enter` | `Enter` | 发送消息 |
| | 多行输入 | `Shift + Enter` | `Shift + Enter` | 换行 |
| | 清空输入 | `Cmd/Ctrl + K` | `Ctrl + K` | 清空输入框 |
| **工具操作** | 批准工具 | `Enter` | `Enter` | 批准执行 |
| | 拒绝工具 | `Cmd/Ctrl + D` | `Ctrl + D` | 拒绝当前 |
| | 全部批准 | `Cmd/Ctrl + Shift + A` | `Ctrl + Shift + A` | 批准所有 |
| | 停止执行 | `Esc` | `Esc` | 停止生成 |
| **任务管理** | 新任务 | `Cmd/Ctrl + N` | `Ctrl + N` | 新建任务 |
| | 继续任务 | `Cmd/Ctrl + R` | `Ctrl + R` | 继续上次 |
| | 任务历史 | `Cmd/Ctrl + H` | `Ctrl + H` | 查看历史 |

### 特殊命令

| 命令 | 功能 | 用法 |
|------|------|------|
| `/clear` | 清空对话上下文 | `/clear` |
| `/model <name>` | 切换模型 | `/model claude-sonnet` |
| `/cost` | 显示当前消耗 | `/cost` |
| `/token` | 显示 token 统计 | `/token` |
| `/export` | 导出对话记录 | `/export filename` |
| `/mcp list` | 列出 MCP 服务器 | `/mcp list` |
| `/mcp add <name>` | 添加 MCP 服务器 | `/mcp add github` |
| `/mcp remove <name>` | 移除 MCP 服务器 | `/mcp remove github` |

### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+v",
    "command": "roo-code.open",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+n",
    "command": "roo-code.newTask",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+h",
    "command": "roo-code.history",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "roo-code.acceptAll",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+d",
    "command": "roo-code.dismissAll",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+g",
    "command": "roo-code.generate",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "roo-code.refactor",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+t",
    "command": "roo-code.test",
    "when": "editorTextFocus"
  }
]
```

---

## Roo Code 完整配置参考

### 基础配置

```json
{
  "roo.code": {
    // API 配置
    "apiProvider": "anthropic",
    "apiKey": "${ANTHROPIC_API_KEY}",
    "model": "claude-sonnet-4-20250514",
    
    // 模型参数
    "maxTokens": 8192,
    "temperature": 0.7,
    "topP": 0.9,
    
    // 自动批准
    "autoApproval": {
      "enabled": true,
      "excludePatterns": ["rm -rf", "DROP TABLE"]
    },
    
    // 本地模型
    "localModels": {
      "enabled": false,
      "provider": "ollama",
      "endpoint": "http://localhost:11434/v1/chat/completions",
      "model": "llama3"
    }
  }
}
```

### MCP 配置

```json
{
  "roo.code.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-git"]
    },
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

### 安全配置

```json
{
  "roo.code.security": {
    "allowedDirectories": ["${workspaceFolder}"],
    "blockedCommands": [
      "rm -rf /*",
      "format C:",
      "del /f /s /q"
    ],
    "requireConfirmation": {
      "delete": true,
      "bash": true,
      "network": true
    },
    "auditLogging": {
      "enabled": true,
      "path": ".roo/audit.log",
      "includeContext": true
    }
  }
}
```

### 成本控制

```json
{
  "roo.code.costControl": {
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
  "roo.code.performance": {
    "maxConcurrentTools": 3,
    "toolTimeout": 60,
    "responseTimeout": 120,
    "cacheEnabled": true,
    "cacheSize": "100MB",
    "streamResponses": true
  }
}
```

### 上下文配置

```json
{
  "roo.code.context": {
    "maxFiles": 50,
    "maxTokens": 100000,
    "priorityFiles": [
      "*.ts",
      "*.tsx",
      "*.js",
      "*.jsx"
    ],
    "excludePatterns": [
      "node_modules/**",
      "dist/**",
      "*.test.ts",
      "coverage/**"
    ]
  }
}
```

---

**Roo Code 文档最终统计**：
- 撰写时间：2026年4月
- 最终行数：约 **5600+ 行**
- 代码示例：110+
- 配置示例：40+
- 快捷键表格：35+
- 提示词模板：25+
- 技术栈集成：30+

> [!SUCCESS]
> Roo Code 作为 Cline 的重要分支，在保持开源优势的同时持续创新。其对 JetBrains IDE 的原生支持使其成为跨平台开发者的首选工具。结合 Ollama 等本地模型实现完全隐私保护，DeepSeek 等低成本 API 实现成本优化，Roo Code 成为 2026 年最具性价比的 AI 编程助手。
