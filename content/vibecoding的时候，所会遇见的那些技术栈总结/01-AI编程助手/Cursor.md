# Cursor - AI 原生 IDE 权威指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Cursor 的核心功能、价格体系、与竞品对比及实战技巧。

---

## 目录

1. [[#Cursor 概述与市场定位]]
2. [[#核心功能详解]]
3. [[#价格体系与订阅计划]]
4. [[#Cursor Rules 完全指南]]
5. [[#Agent Mode 深度使用]]
6. [[#自定义配置与快捷键]]
7. [[#实战 Prompt 技巧]]
8. [[#与其他工具对比]]
9. [[#局限性分析]]
10. [[#选型建议]]
11. [[#进阶技巧与最佳实践]]
12. [[#企业级配置与团队协作]]
13. [[#常见问题与故障排除]]
14. [[#参考资料]]

---

## Cursor 概述与市场定位

### 什么是 Cursor

Cursor 是由 **Anysphere** 公司开发的 **AI 原生集成开发环境（IDE）**，专为 vibecoding（AI 驱动编程）场景设计。与传统 IDE 的"AI 辅助"理念不同，Cursor 将 AI 能力深度融入编辑器的每个角落，实现了"对话即编程"的全新范式。

Cursor 于 2023 年正式发布，凭借其创新的 AI 集成方式和出色的用户体验，迅速成为 AI 编程工具领域的标杆产品。截至 2026 年初，Cursor 的估值已达 **25 亿美元**，年化收入突破 1 亿美元，用户数超过数百万。

### 核心定位

Cursor 的目标用户群体：

- **独立开发者**：追求极致开发效率的个人开发者
- **小型团队**：需要快速迭代的初创公司和远程团队
- **AI 先行者**：愿意采用新技术提升生产力的技术团队
- **学习者**：正在学习编程，希望获得 AI 辅助指导的新手

> [!IMPORTANT]
> Cursor 并不是要替代 VS Code，而是为 AI 时代重新定义 IDE 的可能性。如果你习惯 VS Code，Cursor 提供了完整的兼容性导入。

### 技术架构

```
┌─────────────────────────────────────────────────────────┐
│                    Cursor IDE                            │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │   Editor    │  │  AI Chat    │  │  Terminal   │     │
│  │   (VS Code) │  │   (Claude)  │  │   (Agent)   │     │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘     │
│         │                │                │              │
│  ┌──────┴────────────────┴────────────────┴──────┐     │
│  │              AI Engine Layer                   │     │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐      │     │
│  │  │  Claude  │ │   GPT    │ │  Gemini  │ ...  │     │
│  │  └──────────┘ └──────────┘ └──────────┘      │     │
│  └──────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────┘
```

### 版本演进历程

| 时间 | 版本 | 重大更新 |
|------|------|----------|
| 2023.01 | Cursor v0.1 | 首次发布，基础 AI 对话 |
| 2023.06 | Cursor v0.2 | 增加代码补全功能 |
| 2023.09 | Cursor v0.3 | Composer 功能上线 |
| 2024.02 | Cursor v0.4 | Agent Mode 正式发布 |
| 2024.06 | Cursor 1.0 | 正式版本，品牌重塑 |
| 2024.10 | Cursor 2.0 | 多模型支持增强 |
| 2025.01 | Cursor 2.5 | Claude Opus 集成 |
| 2025.06 | Cursor 3.0 | Cursor Rules 2.0 |
| 2026.01 | Cursor 3.5 | Cascade 架构引入 |

### 为什么选择 Cursor

#### 1. AI 深度集成

Cursor 的最大特点是 AI 的深度集成。与其他工具的"插件式"AI 辅助不同，Cursor 从底层架构就将 AI 作为核心组件：

```python
# 其他工具的工作流
用户 → 写代码 → 调用 AI → 获取建议 → 手动应用建议

# Cursor 的工作流
用户 → 与 AI 对话 → AI 直接修改代码 → 立即生效
```

#### 2. 上下文感知能力

Cursor 能够理解整个项目的上下文：

```markdown
上下文理解层级：
├─ 当前文件（自动）
├─ 打开的标签页（自动）
├─ 项目目录结构（可配置）
├─ 依赖配置文件（自动）
├─ Git 历史（可配置）
└─ 文档和注释（可配置）
```

#### 3. 多模型支持

Cursor 支持多种 AI 模型，用户可以根据任务需求选择：

| 模型 | 特点 | 适用场景 |
|------|------|----------|
| Claude Opus 4.6 | 最强推理能力 | 复杂架构设计 |
| Claude Sonnet 4.6 | 均衡性价比 | 日常开发（推荐） |
| GPT-4.5 | 创意能力强 | 需要创意的任务 |
| GPT-4o | 响应速度快 | 快速修复 |
| Gemini 2.0 | 多模态能力 | 图像相关任务 |

### 与其他 IDE 的关系

> [!TIP]
> Cursor 基于 VS Code 构建，兼容大部分 VS Code 扩展和配置。用户可以无缝迁移现有的 VS Code 工作流。

#### 兼容性说明

```
VS Code 兼容性矩阵：

✅ 完全兼容：
├─ 扩展插件（大部分）
├─ 快捷键配置
├─ 主题设置
├─ 工作区配置
└─ 用户设置

⚠️ 部分兼容：
├─ 终端集成（有自己的 AI 终端）
├─ 调试配置（需要适配）
└─ 远程开发（功能受限）

❌ 不兼容：
├─ VS Code 特定的 Live Share
└─ VS Code 的某些实验性功能
```

---

## 核心功能详解

### 1. AI Chat（智能对话）

Cursor 的 AI Chat 功能是其最核心的特性之一，提供了三种交互模式：

#### Cmd+K 快捷对话

在编辑器任意位置唤起 AI 对话窗口，支持：
- 选中代码后自动作为上下文
- 支持多轮对话和代码追问
- 可以直接应用 AI 的修改建议

```typescript
// 选中这段代码后按 Cmd+K
function calculateTotal(items: CartItem[]): number {
  return items.reduce((sum, item) => sum + item.price * item.quantity, 0);
}

// AI 会自动理解这段代码并提供相关建议
```

#### 独立 Chat 面板

专门的对话界面，支持：
- 文件级别的上下文理解
- 拖拽文件到对话中
- 代码高亮和格式化显示
- 会话历史保存

#### 跨文件对话

可以询问涉及多个文件的问题，Cursor 会自动：
- 解析项目结构和依赖关系
- 理解模块间的调用关系
- 提供跨文件的修改建议

#### Chat 功能高级技巧

##### 1. 上下文注入

```markdown
# 使用 @ 符号注入特定上下文
@file:src/users/model.ts    # 注入特定文件
@folder:src/api            # 注入整个目录
@doc:README.md             # 注入文档内容
@search:auth middleware     # 注入搜索结果
@git:last commit           # 注入 Git 信息
```

##### 2. 对话预设模板

```markdown
# 重构模板
请帮我重构 [选中的代码]，目标是：
1. 提高可读性（重命名变量、添加注释）
2. 提取重复逻辑为公共函数
3. 优化性能（减少不必要的重渲染）
4. 符合本项目的 Cursor Rules 规范

# Bug 修复模板
我遇到了一个 [错误类型] 错误：
错误信息：[粘贴错误日志]
相关代码：[粘贴相关代码]

请分析可能的原因，并提供修复方案。

# 测试生成模板
请为 [选中的函数/模块] 生成单元测试：
- 使用 [测试框架]
- 覆盖正常路径和边界情况
- 包含边界值测试
```

### 2. Agent Mode（智能体模式）

Agent Mode 是 Cursor 2.0 引入的重磅功能，允许 AI 执行更复杂的任务：

| 能力 | 说明 |
|------|------|
| **文件操作** | 创建、重命名、移动、删除文件 |
| **代码修改** | 自动修改多个文件中的代码 |
| **终端执行** | 运行命令行工具和脚本 |
| **浏览器操作** | 搜索网页、验证修复 |
| **Git 操作** | 自动提交代码变更 |

#### Agent Mode 使用流程

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent Mode 工作流                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  1. 任务输入                                                  │
│     用户描述任务 → "重构整个认证模块"                           │
│                    ↓                                         │
│  2. 任务分析                                                  │
│     AI 分析需求 → 识别涉及的文件 → 制定执行计划                 │
│                    ↓                                         │
│  3. 执行确认                                                  │
│     展示执行计划 → 用户确认 → 开始执行                          │
│                    ↓                                         │
│  4. 分步执行                                                  │
│     读取文件 → 分析代码 → 应用修改 → 验证结果                   │
│                    ↓                                         │
│  5. 结果汇报                                                  │
│     完成总结 → 修改清单 → 潜在问题提醒                          │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### Agent Mode 代码示例

```typescript
// 场景：使用 Agent Mode 重构用户认证模块

// 用户输入：
"重构 src/auth/ 目录下的所有认证相关代码，
使用策略模式重写认证逻辑，
保持 API 兼容性"

/*
 * Agent 执行过程：
 * 
 * 1. 分析现有代码结构
 *    - src/auth/login.ts
 *    - src/auth/register.ts
 *    - src/auth/passwordReset.ts
 *    - src/auth/session.ts
 * 
 * 2. 制定重构计划
 *    - 创建 AuthenticationStrategy 接口
 *    - 实现 LocalStrategy、OAuthStrategy 等
 *    - 创建 StrategyFactory
 *    - 迁移现有逻辑到新架构
 *    - 更新测试用例
 * 
 * 3. 执行修改
 *    [修改过程...]
 * 
 * 4. 验证结果
 *    - 运行测试
 *    - 检查 API 兼容性
 *    - 生成变更报告
 */
```

> [!TIP]
> Agent Mode 非常适合处理重复性高的任务，如批量重命名变量、统一代码风格、添加测试用例等。

### 3. Composer（代码作曲家）

Composer 是 Cursor 的高级功能，支持多文件编辑和项目级修改：

#### 常规模式

- 选中多个文件同时编辑
- AI 理解文件间的依赖关系
- 保持修改的一致性

#### Agent 模式

- 接收复杂任务描述
- 自动拆解为多个子任务
- 按步骤执行并汇报进度

#### Composer 工作示例

```markdown
# 场景：创建完整的用户模块

任务描述：
"创建一个用户管理模块，包含：
- 用户实体（用户名、邮箱、密码、角色）
- CRUD API 端点
- JWT 认证中间件
- 单元测试
- API 文档"

# Composer 执行：

## 步骤 1：创建数据模型
src/models/User.ts       [✓]
src/types/User.ts        [✓]

## 步骤 2：创建服务层
src/services/userService.ts  [✓]

## 步骤 3：创建控制器
src/controllers/userController.ts  [✓]

## 步骤 4：创建路由
src/routes/userRoutes.ts    [✓]

## 步骤 5：创建中间件
src/middleware/auth.ts     [✓]
src/middleware/validate.ts [✓]

## 步骤 6：编写测试
src/tests/user.test.ts      [✓]

## 步骤 7：生成文档
docs/api/users.md           [✓]
```

### 4. 代码补全

#### Tab 补全

- 智能代码片段推荐
- 学习个人编码习惯
- 支持多行补全

```typescript
// 示例 1：函数补全
const fetchUser = async (id: string) => {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
// 按 Tab 键自动补全：
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  const data = await response.json();
  return {
    ...data,
    fetchedAt: new Date().toISOString()
  };
};
```

```python
# 示例 2：Python 类补全
class DataProcessor:
    def __init__(self, config: dict):
        self.config = config
        self.data = None
        
    def load_data(self, path: str):
        with open(path, 'r') as f:
            self.data = json.load(f)
    
    def process(self) -> pd.DataFrame:
        # 按 Tab 补全处理逻辑
        df = pd.DataFrame(self.data)
        return df.dropna().reset_index(drop=True)
```

#### 注释生成代码

- 写下注释描述功能
- 按 Tab 生成对应代码
- 支持中文注释

```typescript
// 输入注释并按 Tab 生成代码：

/**
 * 计算两个日期之间的天数差异
 * @param startDate 开始日期
 * @param endDate 结束日期
 * @returns 天数
 */
function calculateDaysBetween(startDate: Date, endDate: Date): number {
  const oneDay = 24 * 60 * 60 * 1000;
  const diffDays = Math.round(Math.abs((endDate.getTime() - startDate.getTime()) / oneDay));
  return diffDays;
}

// 中文注释同样支持：

/* 用户注册函数
   参数：email-邮箱地址，password-密码
   返回：用户对象或错误信息 */
```

### 5. 代码解释与调试

| 功能 | 快捷键 | 说明 |
|------|--------|------|
| **悬停解释** | 鼠标悬停 | 显示变量/函数 AI 解释 |
| **错误诊断** | `Cmd+Shift+E` | AI 自动分析错误 |
| **性能分析** | `Cmd+K` 询问 | 识别潜在性能问题 |

```typescript
// 示例：悬停解释
const userList = users.filter(u => u.isActive);
// 悬停显示：过滤活跃用户列表，返回新数组（不修改原数组）
// 时间复杂度：O(n)，空间复杂度：O(n)
```

### 6. 代码审查功能

Cursor 提供内置的代码审查功能：

```markdown
# 代码审查流程

1. 选中需要审查的代码
2. 按 Cmd+K 唤起 AI
3. 输入："审查这段代码，重点关注安全问题"

# AI 输出格式：

## 代码审查报告

### 安全性问题
🔴 [高危] 第 15 行：发现硬编码密码
   建议：使用环境变量或密钥管理服务

🟡 [中危] 第 23 行：SQL 拼接存在注入风险
   建议：使用参数化查询

🟡 [中危] 第 45 行：缺少输入验证
   建议：添加 Zod 或 Joi 验证

### 性能问题
🟢 [优化] 第 30 行：可使用 useMemo 缓存计算结果

### 代码质量
🟢 [建议] 第 18 行：变量命名不够清晰
   建议：rename `data` → `userData`
```

### 7. 终端集成

Cursor 的终端与 AI 深度集成：

```bash
# 在终端中使用 AI 辅助
# 按 Cmd+K 在终端唤起 AI 对话

# 示例命令：
"帮我创建一个 PostgreSQL Docker 容器"
"解释这个错误日志"
"优化这个 SQL 查询"
"运行测试并分析失败原因"
```

### 8. Git 集成

| 功能 | 说明 |
|------|------|
| **Commit 消息生成** | AI 自动生成规范的提交信息 |
| **Branch 命名建议** | 根据变更内容建议分支名 |
| **PR 描述生成** | 自动分析变更生成 PR 描述 |
| **冲突解决** | AI 辅助解决 Git 冲突 |

```bash
# Commit 消息生成示例

# 变更内容：
- 添加用户注册功能
- 添加邮箱验证
- 添加密码强度检测

# AI 生成的 Commit 消息：
feat(auth): implement user registration with email verification

- Add user registration endpoint with email/password
- Implement email format validation
- Add password strength validation (min 8 chars, requires uppercase, number)
- Hash passwords using bcrypt before storage
- Add unit tests for validation logic
```

---

## 价格体系与订阅计划

### 价格对比表

| 套餐 | 价格 | 功能 | 适用场景 |
|------|------|------|----------|
| **Free** | $0 | 100 次 Premium 请求/月 | 轻度体验 |
| **Pro** | $20/月 或 $16/月（年付） | 无限 Premium 请求，Claude 4/5、GPT-4.5 | 个人开发者 |
| **Business** | $40/用户/月 | 团队协作、Cursor Rules 共享、企业安全 | 小型团队 |
| **Enterprise** | 定制报价 | SSO、SAML、高级合规、SLA 保障 | 中大型企业 |
| **Higher Education** | $16/月 | 学生/教师专属，需 .edu 邮箱验证 | 教育场景 |

### 各套餐详细功能

#### Free 套餐

- 每月 100 次 Premium AI 请求
- 基础代码补全
- 只能使用 Claude 3.5 Sonnet
- 适合尝鲜体验

> [!NOTE]
> Free 套餐的 100 次限制是针对 Premium 请求（使用 Claude Opus/GPT-4 的复杂请求），普通代码补全不计入限制。

#### Pro 套餐（推荐个人开发者）

- **无限** Premium AI 请求
- 模型选择：Claude Opus 4.6、Claude Sonnet 4.6、GPT-4.5、GPT-4o
- 优先使用最新模型
- 高级代码补全
- 每日 500 次 Cursor Small（快速模式）

```markdown
# Pro 套餐模型配额说明

| 模型 | 每日限制 | 说明 |
|------|---------|------|
| Claude Sonnet 4.6 | 无限 | 日常使用推荐 |
| Claude Opus 4.6 | 100次/天 | 复杂任务 |
| GPT-4.5 | 50次/天 | 备用选择 |
| GPT-4o | 50次/天 | 快速响应 |
```

#### Business 套餐（推荐团队）

- 所有 Pro 功能
- 团队共享 Cursor Rules
- 使用情况分析仪表板
- 团队使用策略管理
- SSO 支持（即将推出）

#### Enterprise 套餐（推荐企业）

- 所有 Business 功能
- 自定义数据保留策略
- 高级安全合规（SOC2、GDPR）
- 专属客户成功经理
- 99.9% SLA 保障

> [!TIP]
> 对于个人开发者，Pro 套餐的性价比最高。如果你是学生，可以申请 Higher Education 套餐享受 Pro 功能但只需 $16/月。

### Token 消耗说明

| 模型 | 每千请求消耗 | 月均消耗估算 |
|------|-------------|-------------|
| Claude Sonnet 4.6 | 1 token | 普通使用约 50-200 请求/天 |
| Claude Opus 4.6 | 3 token | 复杂任务使用 |
| GPT-4.5 | 2 token | 备用选择 |

### 计费常见问题

> [!IMPORTANT]
> **Q: 如果我用完了每日的快速请求配额怎么办？**
> A: 可以继续使用，但会降级到基础模型（Claude Sonnet），速度可能较慢。

> **Q: Pro 套餐可以多人共享吗？**
> A: 不可以，Pro 套餐是按用户计费的。每个用户需要单独订阅。

> **Q: 年付套餐可以退款吗？**
> A: Cursor 提供 7 天退款保证，但需要联系客服申请。

---

## Cursor Rules 完全指南

### 什么是 Cursor Rules

Cursor Rules 是 Cursor 独有的配置系统，允许开发者定义项目级别的 AI 行为规范。通过 Rules，团队可以：

- 统一代码风格和命名规范
- 强制执行安全最佳实践
- 指导 AI 理解业务逻辑
- 保持跨团队的一致性

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

```markdown
# ❌ 效果差
- 使用有意义的变量名

# ✅ 效果好
- 变量名必须使用完整的英文单词或公认缩写
- ✅ good: userName, totalCount, isActive
- ❌ bad: n, x, tmp, data, val
```

#### 2. 明确禁止行为

```markdown
# 禁止的代码模式
- 禁止使用 var，必须使用 const 或 let
- 禁止内联样式（除非是动态值）
- 禁止使用 any 类型
- 禁止 console.log 用于生产代码
```

#### 3. 优先级标记

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

Agent Mode 基于大语言模型的推理能力，通过"思考-规划-执行"的循环完成任务：

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

Cursor 支持自定义主题，包括：
- 明暗主题切换
- 语法高亮配色
- AI 对话界面主题
- 终端配色方案

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

```markdown
# 第一轮：基础实现
"请实现一个 React 计数器组件"

# 第二轮：增强功能
"很好，现在添加重置功能，并支持自定义步长"

# 第三轮：优化体验
"现在添加动画效果，使用 Framer Motion"
```

---

## 与其他工具对比

### 功能对比表

| 特性 | Cursor | VS Code + Copilot |
|------|--------|-------------------|
| **AI 集成深度** | 深度原生集成 | 插件扩展 |
| **代码修改方式** | 直接编辑文件 | 复制粘贴建议 |
| **多文件修改** | Agent 自动处理 | 手动逐文件修改 |
| **上下文理解** | 全项目理解 | 单文件或选中代码 |
| **价格** | $20/月（Pro） | $19/月（Copilot Pro） |
| **模型选择** | 多种可选 | 主要 GPT-4 |
| **自定义规则** | Cursor Rules | 无原生支持 |
| **学习曲线** | 较低（继承 VS Code） | 依赖 Copilot 体验 |
| **生态扩展** | VS Code 扩展兼容 | 原生 VS Code |

### 性能对比

| 场景 | Cursor | VS Code + Copilot |
|------|--------|-------------------|
| 简单补全 | 响应快 | 响应快 |
| 复杂重构 | AI 直接修改 | 提供建议，需手动应用 |
| 多文件修改 | Agent 自动完成 | 无法直接完成 |
| 首次加载 | 较慢（需加载模型） | 较快 |

> [!IMPORTANT]
> Cursor 的优势在于"AI 执行"而非"AI 建议"。Copilot 告诉你怎么做，Cursor 直接帮你做完。

### 使用场景对比

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 快速单行补全 | 均可 | 两者响应速度相近 |
| 大规模重构 | **Cursor** | Agent 可自动处理多文件 |
| 新项目初始化 | **Cursor** | 一次对话完成全部结构 |
| 学习编程 | VS Code + Copilot | 更偏向引导学习 |
| 代码审查 | **Cursor** | 更深入的上下文分析 |
| 预算有限 | VS Code + Copilot | Free 用户限制更宽松 |

### 技术架构对比

```
┌─────────────────────────────────────────────────────────────┐
│                      Cursor 架构                             │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Cursor IDE (自研 UI 层)                  │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │              AI Integration Layer                    │   │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐   │   │
│  │  │ Claude  │ │   GPT   │ │ Gemini  │ │  其他   │   │   │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘   │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │              Context Management                     │   │
│  │  项目索引 | 文件解析 | 依赖分析 | 历史追踪            │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  VS Code + Copilot 架构                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              VS Code (微软官方 IDE)                   │   │
│  ├─────────────────────────────────────────────────────┤   │
│  │              Copilot Extension                        │   │
│  │  ┌─────────────────────────────────────────────┐    │   │
│  │  │           GitHub Copilot API                  │    │   │
│  │  │           (OpenAI GPT-4)                      │    │   │
│  │  └─────────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 成本效益分析

| 维度 | Cursor | VS Code + Copilot |
|------|--------|-------------------|
| **初始成本** | $0（Free） | $0（Free） |
| **Pro 月费** | $20 | $19（Copilot Pro） |
| **年费（个人）** | $192 | $228（Copilot 年付） |
| **企业成本** | $40/用户/月 | $19-39/用户/月 |
| **学习成本** | 低 | 中 |
| **效率提升** | 高 | 中 |

---

## 局限性分析

### 1. 技术局限性

| 局限点 | 说明 | 缓解方案 |
|-------|------|---------|
| **上下文窗口** | 不同模型有不同的上下文限制 | 分段处理大文件 |
| **代码理解深度** | 对复杂架构的理解可能不完整 | 配合 Cursor Rules 提供更多上下文 |
| **模型幻觉** | AI 可能生成不存在的 API | 交叉验证重要代码 |
| **长任务处理** | 复杂任务可能耗时较长 | 拆分为多个子任务 |
| **实时性** | 无法实时监听代码变化 | 需要手动触发 |

### 2. 产品局限性

| 局限点 | 说明 | 缓解方案 |
|-------|------|---------|
| **价格** | Pro 套餐 $20/月 | 学生优惠可用 $16/月 |
| **离线不可用** | 完全依赖云端 AI | 暂无本地方案 |
| **学习成本** | Agent 模式需要适应 | 从基础功能开始使用 |
| **企业功能** | Business/Enterprise 较贵 | 可申请团队优惠 |
| **内存占用** | IDE 本身占用较大内存 | 使用轻量级配置 |

### 3. 安全性考量

- **数据隐私**：代码会发送到 AI 服务商处理
- **敏感代码**：不建议处理高度敏感的代码
- **API 成本**：使用自己的 API Key 可控制成本

> [!NOTE]
> 企业用户应考虑使用 Cursor Enterprise 版本，其提供更严格的数据安全策略。

### 4. 使用体验局限

| 局限点 | 说明 | 用户反馈 |
|-------|------|----------|
| **首次响应慢** | 需要加载模型 | 可以接受 |
| **网络依赖** | 断网完全不可用 | 严重痛点 |
| **自动补全干扰** | 有时建议不相关 | 可以按 Esc 拒绝 |
| **长对话失忆** | 上下文超出限制 | 需要开启新对话 |

---

## 选型建议

### 何时选择 Cursor

| 场景 | 推荐程度 | 原因 |
|------|---------|------|
| vibecoding 开发 | ⭐⭐⭐⭐⭐ | AI 集成最佳 |
| 快速原型开发 | ⭐⭐⭐⭐⭐ | Agent 模式大幅提升效率 |
| 全栈项目开发 | ⭐⭐⭐⭐ | 多文件处理能力强 |
| 学习编程 | ⭐⭐⭐ | 可能过于依赖 AI |
| 企业级开发 | ⭐⭐⭐⭐ | Business 套餐提供团队协作 |
| 预算有限 | ⭐⭐ | 免费额度有限 |

### 替代方案对比

| 场景 | Cursor | Windsurf | Cline |
|------|--------|---------|-------|
| AI 集成深度 | 最佳 | 较好 | 较好（开源） |
| 价格 | $20/月 | $10/月或免费 | 免费（开源） |
| VS Code 兼容 | ✅ | ✅ | ✅ |
| 多模型支持 | 多种 | Codeium 为主 | 多种（可配置） |
| 自主控制 | 依赖云端 | 依赖云端 | 可完全本地 |

### 新手入门建议

1. **第一周**：体验免费额度，熟悉基础功能（Cmd+K、Tab 补全）
2. **第二周**：尝试 Composer 的多文件编辑功能
3. **第三周**：配置项目 Cursor Rules，定制 AI 行为
4. **第四周**：根据需求决定是否升级 Pro 套餐

### 团队选型建议

| 团队规模 | 推荐方案 | 理由 |
|---------|---------|------|
| 个人开发者 | Cursor Pro | 功能完整，性价比高 |
| 2-5 人团队 | Cursor Business | 共享 Rules，团队协作 |
| 5-20 人团队 | Cursor Enterprise | SSO，安全合规 |
| 20+ 人企业 | Cursor Enterprise + 自定义 | 按需定制 |

---

## 进阶技巧与最佳实践

### 1. 效率提升技巧

#### 快捷键组合

```markdown
# 高效工作流组合

1. 快速修复循环
Cmd+K → "修复这个错误" → Tab（应用）→ Tab（下一个）→ ...

2. 批量重命名
Cmd+Shift+F → 搜索 → Cmd+K → "重命名为 Xxx" → 应用所有

3. 快速生成测试
选中函数 → Cmd+K → "为这个函数生成测试" → Tab（应用）

4. 代码片段循环
Tab（接受） → Tab（下一个建议） → Tab（接受） → ...
```

#### AI 对话优化

```markdown
# 高效对话技巧

1. 使用 @ 引用特定上下文
   @file:src/users/model.ts
   @folder:src/api
   @search:auth middleware

2. 限制回答范围
   "仅修改 src/auth/ 目录下的文件"
   "使用不超过 50 行代码"

3. 要求逐步解释
   "请先解释你的方案，确认后再实施"

4. 使用示例引导
   "参考 Xxx.ts 的实现方式"
```

### 2. 代码质量保证

#### 代码审查流程

```markdown
# AI 辅助代码审查流程

1. 提交前审查
   Cmd+K → "审查我选择的代码"

2. 批量审查
   Cmd+Shift+K → "审查 src/features/user 模块"

3. 重点审查
   "重点审查安全性，不要关注格式问题"

4. 修复验证
   "应用修复建议后，重新审查确保问题已解决"
```

#### 自动化测试增强

```markdown
# AI 生成测试的技巧

1. 指定测试框架
   "使用 Vitest 生成测试"

2. 指定覆盖率目标
   "确保分支覆盖率 80% 以上"

3. 包含边界情况
   "包含空值、极端值、错误情况测试"

4. 生成描述性测试名
   "测试命名要清晰描述测试场景"
```

### 3. 项目管理技巧

#### 大型项目管理

```markdown
# 处理大型项目的策略

1. 分模块处理
   "仅处理 src/features/auth 模块"

2. 使用索引
   "先分析项目结构，建立索引"

3. 增量修改
   "分批次修改，每次不超过 10 个文件"

4. 版本控制
   "每次大规模修改前创建分支"
```

#### 团队协作

```markdown
# 团队协作最佳实践

1. 共享 Cursor Rules
   - 将 .cursor/rules 纳入版本控制
   - 使用 default.mdc 作为团队规范

2. 代码审查集成
   - 使用 Cursor 进行 PR 前审查
   - 将审查结果同步到 GitHub

3. 知识共享
   - 将好的 Prompt 保存为模板
   - 团队内部分享有效的提示词
```

### 4. 性能优化

#### 响应速度优化

```markdown
# 提升响应速度的技巧

1. 选择合适的模型
   - 简单任务用 Claude Sonnet
   - 复杂任务用 Claude Opus

2. 限制上下文范围
   "仅分析当前文件，不要扫描整个项目"

3. 开启快速模式
   - 使用 Cursor Small 进行简单查询
   - 只在必要时使用 Premium 模型

4. 预热索引
   - 打开项目后等待索引建立
   - 避免在索引未完成时进行复杂查询
```

#### 成本控制

```markdown
# 控制 API 成本的策略

1. 使用缓存
   - 开启 Cursor 的上下文缓存
   - 复用对话历史

2. 选择性价比模型
   - 日常使用：Claude Sonnet
   - 复杂推理：Claude Opus

3. 优化 Prompt
   - 使用简洁明确的 Prompt
   - 避免不必要的上下文

4. 监控使用量
   - 定期检查使用统计
   - 设置使用警报
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
   ├── team-security.mdc     # 安全规范
   ├── team-code-style.mdc   # 代码风格
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

## 常见问题与故障排除

### 安装问题

> [!NOTE]
> 以下是常见安装问题的解决方案。

#### 问题 1：Cursor 无法启动

**症状**：点击 Cursor 图标无反应

**解决方案**：
1. 检查系统要求（macOS 10.15+ / Windows 10+ / Linux）
2. 重新安装 Cursor
3. 清除缓存：`rm -rf ~/.cursor/Cache`
4. 检查日志：`~/.cursor/logs/`

#### 问题 2：扩展不兼容

**症状**：VS Code 扩展在 Cursor 中无法使用

**解决方案**：
1. 确认扩展支持 Cursor
2. 查找 Cursor 专用版本
3. 使用替代扩展

### 使用问题

#### 问题 3：AI 响应缓慢

**原因**：
- 网络连接问题
- 服务器负载高
- 请求超出上下文限制

**解决方案**：
1. 检查网络连接
2. 减少请求上下文
3. 使用快速模式
4. 稍后重试

#### 问题 4：代码建议不准确

**原因**：
- 上下文不足
- 项目结构复杂
- 模型能力限制

**解决方案**：
1. 提供更多上下文
2. 使用 @ 引用相关文件
3. 优化 Cursor Rules
4. 尝试其他模型

### 账户问题

#### 问题 5：订阅显示未激活

**解决方案**：
1. 退出并重新登录
2. 检查付款状态
3. 联系客服：support@cursor.sh

#### 问题 6：免费额度用完

**解决方案**：
1. 等待下月刷新
2. 升级到 Pro 套餐
3. 使用基础补全功能

### 技术问题

#### 问题 7：文件无法保存

**症状**：Cursor 提示无法保存文件

**解决方案**：
1. 检查文件权限
2. 确认磁盘空间充足
3. 禁用冲突扩展

#### 问题 8：Git 操作失败

**症状**：Git 命令执行失败

**解决方案**：
1. 检查 Git 配置
2. 确认 SSH 密钥配置
3. 使用终端手动执行 Git 命令

---

## Cursor 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 10.15+ / Windows 10+ / Linux (Ubuntu 20.04+) |
| 内存 | 8GB RAM |
| 磁盘空间 | 1GB 可用空间 |
| 网络 | 稳定的互联网连接（用于 AI 服务） |

#### 推荐配置

| 配置项 | 推荐 |
|--------|------|
| 操作系统 | macOS 13+ / Windows 11 / Ubuntu 22.04+ |
| 内存 | 16GB+ RAM |
| 处理器 | 多核处理器 |
| 网络 | 高速宽带连接 |

### 安装步骤详解

#### macOS 安装

```bash
# 方法一：官网下载
# 1. 访问 https://cursor.sh/
# 2. 点击 Download 按钮
# 3. 下载 .dmg 文件
# 4. 打开 .dmg 文件
# 5. 将 Cursor.app 拖入 Applications 文件夹

# 方法二：Homebrew 安装
brew install --cask cursor

# 方法三：命令行安装
curl -fsSL https://cursor.sh/install.sh | sh
```

#### Windows 安装

```powershell
# 方法一：官网下载
# 1. 访问 https://cursor.sh/
# 2. 下载 .exe 安装包
# 3. 运行安装程序

# 方法二：Windows Package Manager (winget)
winget install Cursor.Cursor

# 方法三：Chocolatey
choco install cursor
```

#### Linux 安装

```bash
# Debian/Ubuntu
# 1. 下载 .deb 文件
wget https://cursor.sh/download/linux?version=latest -O cursor.deb

# 2. 安装
sudo dpkg -i cursor.deb

# 3. 修复依赖（如需要）
sudo apt-get install -f

# Fedora/RHEL
sudo rpm -i cursor.rpm

# Arch Linux (使用 yay)
yay -S cursor
```

### 首次启动配置

#### 1. 账户登录

```markdown
首次启动 Cursor 时：
1. 启动画面显示后，点击 "Sign In" 按钮
2. 可选择以下登录方式：
   - GitHub 账户登录（推荐）
   - Google 账户登录
   - 邮箱密码登录
3. 完成身份验证后，Cursor 会自动同步设置
```

#### 2. 主题设置

```json
// .vscode/settings.json 中配置
{
  "workbench.colorTheme": "One Dark Pro",
  "editor.fontSize": 14,
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontLigatures": true,
  "editor.cursorStyle": "line",
  "editor.cursorBlinking": "smooth"
}
```

#### 3. AI 模型配置

```json
// .vscode/settings.json
{
  "cursor.ai": {
    "model": "claude-sonnet-4.6",
    "chatModel": "claude-opus-4.6",
    "temperature": 0.7,
    "maxTokens": 8192
  },
  "cursor.completion": {
    "inline": true,
    "ghostText": true,
    "tabComplete": true
  }
}
```

#### 4. 代理设置（如需要）

```json
{
  "cursor.proxy": {
    "enabled": true,
    "url": "http://proxy.example.com:8080",
    "bypass": [
      "localhost",
      "127.0.0.1"
    ]
  }
}
```

### Cursor Rules 进阶配置

#### 全局规则模板

```markdown
# .cursor/rules/default.mdc
# cursorules
# 全局代码规范 - 适用于所有项目

## 代码风格
- 缩进：2 空格（TypeScript/JavaScript）或 4 空格（Python）
- 行宽：100 字符
- 文件末尾：单行空行
- 字符串引号：单引号优先（JS/TS），双引号（Python）

## TypeScript 规范
- 严格模式：true
- 禁用 any：所有类型必须明确
- 接口 vs 类型别名：
  - 使用 interface 定义对象结构
  - 使用 type 定义联合类型、交叉类型
- 导出：使用命名导出优于默认导出

## 命名规范
- 变量/函数：camelCase
- 类/接口/类型：PascalCase
- 常量：UPPER_SNAKE_CASE
- 文件名：kebab-case.ts

## 安全规范
- 禁止硬编码敏感信息
- 所有 API 密钥使用环境变量
- 用户输入必须验证
- 使用参数化查询防止 SQL 注入

## 性能规范
- 避免不必要的重渲染
- 使用 useMemo/useCallback 优化
- 大数据列表使用虚拟滚动
- 图片懒加载
```

#### Next.js 项目规则

```markdown
# .cursor/rules/nextjs.mdc
# cursorules
# Next.js 14 App Router 项目规范

## 目录结构
```
src/
├── app/                    # App Router 页面
│   ├── page.tsx          # 首页
│   ├── layout.tsx        # 根布局
│   ├── globals.css       # 全局样式
│   └── [slug]/          # 动态路由
├── components/            # React 组件
│   ├── ui/              # 基础 UI 组件
│   ├── features/        # 功能组件
│   └── layouts/        # 布局组件
├── lib/                  # 工具函数
├── hooks/                # 自定义 Hooks
├── types/               # TypeScript 类型
├── services/            # API 服务层
└── stores/              # 状态管理
```

## 服务器组件 vs 客户端组件
- 默认使用服务器组件
- 仅在需要以下特性时使用 "use client"：
  - useState/useReducer
  - useEffect
  - 浏览器 API
  - 自定义事件处理

## 数据获取
- 服务端：使用 async/await 直接查询数据库
- 客户端：使用 React Query / SWR
- 缓存：使用 Next.js 的 fetch 缓存选项

## API 路由
- 使用 route.ts 文件
- 分离 HTTP 方法处理器
- 使用 Zod 进行输入验证
- 统一错误响应格式
```

### 快捷键大全

#### macOS 系统快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **AI 基础** | 唤起 AI 对话 | `Cmd + K` | 打开 AI 聊天窗口 |
| | 接受补全建议 | `Tab` | 接受代码补全 |
| | 拒绝补全建议 | `Esc` | 拒绝当前建议 |
| | 触发补全 | `Cmd + L` | 手动触发代码补全 |
| | AI 解释 | `Cmd + Shift + L` | 解释选中代码 |
| | AI 修复 | `Cmd + Shift + E` | 修复代码错误 |
| **Agent 模式** | 启动 Agent | `Cmd + Shift + G` | 启动 Agent Mode |
| | 确认执行 | `Enter` | 确认 Agent 操作 |
| | 取消执行 | `Esc` | 取消 Agent 操作 |
| **编辑器** | 快速打开文件 | `Cmd + P` | 模糊搜索文件 |
| | 命令面板 | `Cmd + Shift + P` | 搜索命令 |
| | 全局搜索 | `Cmd + Shift + F` | 搜索所有文件 |
| | 多光标选择 | `Cmd + D` | 选择下一个匹配 |
| | 列选择 | `Option + 拖动` | 列模式选择 |
| **导航** | 转到定义 | `Cmd + 点击` | 跳转到定义 |
| | 查找引用 | `Shift + F12` | 查找所有引用 |
| | 返回 | `Cmd + U` | 返回上一个位置 |
| **终端** | 打开终端 | `` Cmd + ` `` | 切换到终端 |
| | 新终端 | `Cmd + Shift + `` ` `` | 新建终端标签 |

#### Windows/Linux 快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **AI 基础** | 唤起 AI 对话 | `Ctrl + K` | 打开 AI 聊天窗口 |
| | 接受补全建议 | `Tab` | 接受代码补全 |
| | 拒绝补全建议 | `Esc` | 拒绝当前建议 |
| | 触发补全 | `Ctrl + L` | 手动触发代码补全 |
| | AI 解释 | `Ctrl + Shift + L` | 解释选中代码 |
| | AI 修复 | `Ctrl + Shift + E` | 修复代码错误 |
| **Agent 模式** | 启动 Agent | `Ctrl + Shift + G` | 启动 Agent Mode |
| | 确认执行 | `Enter` | 确认 Agent 操作 |
| | 取消执行 | `Esc` | 取消 Agent 操作 |
| **编辑器** | 快速打开文件 | `Ctrl + P` | 模糊搜索文件 |
| | 命令面板 | `Ctrl + Shift + P` | 搜索命令 |
| | 全局搜索 | `Ctrl + Shift + F` | 搜索所有文件 |
| | 多光标选择 | `Ctrl + D` | 选择下一个匹配 |
| **终端** | 打开终端 | `` Ctrl + ` `` | 切换到终端 |
| | 新终端 | `Ctrl + Shift + `` ` `` | 新建终端标签 |

#### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+h",
    "command": "cursor.chat",
    "args": { "mode": "inline" },
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "cmd+shift+a",
    "command": "cursor.agent",
    "args": { "mode": "ask" },
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "cursor.refactor",
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "cmd+shift+b",
    "command": "cursor.build",
    "when": "editorTextFocus && editorLangId == 'typescript'"
  },
  {
    "key": "cmd+shift+t",
    "command": "cursor.test",
    "when": "editorTextFocus"
  }
]
```

---

## Cursor AI 提示词工程深度指南

### 提示词基础原则

#### 1. 清晰的结构

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

```markdown
"请一步步分析这个问题：

1. 首先分析代码的执行流程
2. 识别可能的性能瓶颈
3. 列出每种优化方案
4. 比较各方案的优缺点
5. 给出推荐方案及实现代码"
```

#### 2. 约束条件提示

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

### 技术栈特定提示词

#### React + TypeScript

```markdown
"创建一个 React 函数组件 UserProfile，要求：

技术栈：
- React 18
- TypeScript 5.x
- Tailwind CSS
- React Query

功能需求：
1. 通过 userId 获取用户数据
2. 显示用户头像、姓名、邮箱、简介
3. 支持编辑模式
4. 显示用户最近活动列表

代码规范：
- 使用 useQuery 获取数据
- 使用 useMutation 处理更新
- 组件最大 150 行
- 提取自定义 Hooks

错误处理：
- 加载状态：骨架屏
- 错误状态：错误提示 + 重试按钮
- 空状态：友好提示"
```

#### Next.js App Router

```markdown
"在 src/app/users/[id] 目录下创建用户详情页，要求：

技术栈：
- Next.js 14 App Router
- TypeScript
- Prisma ORM
- Zod 验证

页面结构：
1. generateMetadata 函数获取元数据
2. 页面组件为异步服务器组件
3. 包含用户信息卡片
4. 用户帖子列表（分页）
5. 用户设置面板

数据获取：
- 直接使用 Prisma 查询（服务端）
- 使用 revalidate 选项设置缓存策略
- 错误处理使用 error.tsx

API 路由：
- GET /api/users/[id] - 获取用户详情
- PATCH /api/users/[id] - 更新用户信息
- 使用 Zod 验证请求数据"
```

#### Node.js + Express

```markdown
"创建 src/routes/userRoutes.ts，包含完整的用户 CRUD 路由，要求：

技术栈：
- Express.js
- TypeScript
- Prisma
- Zod

路由端点：
- GET /users - 列表（分页、筛选）
- GET /users/:id - 详情
- POST /users - 创建
- PUT /users/:id - 更新
- DELETE /users/:id - 删除

功能要求：
1. 输入验证（Zod Schema）
2. 错误处理中间件
3. 认证中间件（JWT）
4. 权限控制（admin/user）
5. 分页支持

响应格式：
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
  meta?: {
    page: number;
    limit: number;
    total: number;
  };
}
```"
```

---

## Cursor 与现代技术栈深度集成

### Next.js 15 + React 19 集成

#### 项目初始化

```markdown
# Cursor AI 辅助初始化 Next.js 项目

提示词：
"创建一个 Next.js 15 项目，要求：
- 使用 App Router
- TypeScript
- Tailwind CSS
- Prisma ORM + PostgreSQL
- NextAuth.js v5 认证
- React Query 管理客户端状态
- Zod 表单验证

配置：
- ESLint + Prettier
- Husky + lint-staged
- Jest + React Testing Library

目录结构：
- 使用 feature-based 结构
- API 路由在 app/api/
- 组件在 components/
- 类型在 types/"
```

#### Cursor 生成的代码示例

**src/app/users/page.tsx**

```typescript
import { Suspense } from 'react';
import { UserList } from '@/components/features/UserList';
import { UserListSkeleton } from '@/components/ui/skeletons/UserListSkeleton';
import { getUsers } from '@/services/userService';

interface UsersPageProps {
  searchParams: Promise<{ page?: string; limit?: string }>;
}

export default async function UsersPage({ searchParams }: UsersPageProps) {
  const params = await searchParams;
  const page = parseInt(params.page || '1', 10);
  const limit = parseInt(params.limit || '20', 10);

  const { users, total } = await getUsers({ page, limit });

  return (
    <div className="container mx-auto py-8">
      <div className="mb-8">
        <h1 className="text-3xl font-bold">用户列表</h1>
        <p className="text-muted-foreground mt-2">
          共 {total} 位用户
        </p>
      </div>

      <Suspense fallback={<UserListSkeleton count={limit} />}>
        <UserList users={users} total={total} page={page} limit={limit} />
      </Suspense>
    </div>
  );
}

export const metadata = {
  title: '用户列表 - My App',
  description: '浏览和管理所有用户',
};
```

**src/services/userService.ts**

```typescript
import { prisma } from '@/lib/prisma';
import { cache } from 'react';

export interface GetUsersParams {
  page?: number;
  limit?: number;
  search?: string;
  role?: 'USER' | 'ADMIN';
}

export interface GetUsersResult {
  users: User[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export const getUsers = cache(async (params: GetUsersParams): Promise<GetUsersResult> => {
  const { page = 1, limit = 20, search, role } = params;
  const skip = (page - 1) * limit;

  const where = {
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' as const } },
        { email: { contains: search, mode: 'insensitive' as const } },
      ],
    }),
    ...(role && { role }),
  };

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
        role: true,
        createdAt: true,
        _count: { select: { posts: true } },
      },
    }),
    prisma.user.count({ where }),
  ]);

  return {
    users: users as User[],
    total,
    page,
    limit,
    totalPages: Math.ceil(total / limit),
  };
});
```

### React Native + Expo 集成

#### Cursor 辅助开发

```markdown
# Cursor AI 辅助 React Native 开发

提示词：
"使用 Expo + React Native 创建一个待办事项应用，要求：
- Expo SDK 52
- TypeScript
- Expo Router 文件路由
- Zustand 状态管理
- AsyncStorage 持久化
- 支持深色模式
- 支持 iOS/Android 双平台

功能：
1. 创建/编辑/删除待办
2. 待办分组
3. 截止日期提醒
4. 本地推送通知
5. 数据导出为 JSON"
```

#### 生成的代码示例

**app/_layout.tsx**

```typescript
import { DarkTheme, LightTheme } from '@/theme';
import { useColorScheme } from '@/hooks/useColorScheme';
import { StatusBar } from 'expo-status-bar';
import { Stack } from 'expo-router';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  const colorScheme = useColorScheme();

  return (
    <SafeAreaProvider>
      <GestureHandlerRootView style={{ flex: 1 }}>
        <Stack
          screenOptions={{
            headerStyle: {
              backgroundColor: colorScheme === 'dark' 
                ? DarkTheme.colors.background 
                : LightTheme.colors.background,
            },
            headerTintColor: colorScheme === 'dark'
              ? DarkTheme.colors.text
              : LightTheme.colors.text,
          }}
        >
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="+not-found" options={{ title: 'Oops!' }} />
        </Stack>
        <StatusBar style={colorScheme === 'dark' ? 'light' : 'dark'} />
      </GestureHandlerRootView>
    </SafeAreaProvider>
  );
}
```

**app/(tabs)/index.tsx**

```typescript
import { useState } from 'react';
import { View, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';
import { TextInput } from '@/components/ui/TextInput';
import { TodoItem } from '@/components/features/TodoItem';
import { useTodos } from '@/stores/todoStore';
import { useTheme } from '@/hooks/useTheme';
import { Todo } from '@/types';

export default function TodosScreen() {
  const router = useRouter();
  const { theme } = useTheme();
  const [newTodoTitle, setNewTodoTitle] = useState('');
  const { todos, addTodo, toggleTodo, deleteTodo } = useTodos();

  const handleAddTodo = async () => {
    if (!newTodoTitle.trim()) return;
    
    const newTodo: Todo = {
      id: Date.now().toString(),
      title: newTodoTitle.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
      priority: 'medium',
    };
    
    addTodo(newTodo);
    setNewTodoTitle('');
  };

  const renderItem = ({ item }: { item: Todo }) => (
    <TodoItem
      todo={item}
      onToggle={() => toggleTodo(item.id)}
      onDelete={() => deleteTodo(item.id)}
      onPress={() => router.push(`/todo/${item.id}`)}
    />
  );

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <View style={styles.inputContainer}>
        <TextInput
          value={newTodoTitle}
          onChangeText={setNewTodoTitle}
          placeholder="添加新待办..."
          onSubmitEditing={handleAddTodo}
          returnKeyType="done"
        />
        <TouchableOpacity
          style={[styles.addButton, { backgroundColor: theme.colors.primary }]}
          onPress={handleAddTodo}
        >
          <Text style={styles.addButtonText}>添加</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={todos}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.list}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
}
```

### 数据库设计集成

#### Prisma Schema 最佳实践

```prisma
// prisma/schema.prisma
// Cursor AI 辅助生成的完整数据模型

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 用户模型
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  avatar        String?
  passwordHash   String
  role          Role      @default(USER)
  emailVerified DateTime?
  
  // 关系
  accounts      Account[]
  sessions      Session[]
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  
  // 时间戳
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?

  @@index([email])
  @@index([role])
  @@map("users")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

// OAuth 账户
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refreshToken      String? @db.Text
  accessToken      String? @db.Text
  expiresAt        Int?
  tokenType        String?
  scope            String?
  idToken          String? @db.Text
  sessionState     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

// 会话
model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

// 帖子
model Post {
  id         String   @id @default(cuid())
  title      String   @db.VarChar(255)
  slug       String   @unique
  content    String   @db.Text
  excerpt    String?  @db.VarChar(500)
  coverImage String?
  published  Boolean  @default(false)
  publishedAt DateTime?
  
  // 关系
  author     User     @relation(fields: [authorId], references: [id])
  authorId   String
  comments   Comment[]
  tags       Tag[]
  likes      Like[]

  // SEO
  metaTitle       String? @db.VarChar(70)
  metaDescription String? @db.VarChar(160)

  // 时间戳
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([slug])
  @@index([published])
  @@map("posts")
}

// 评论
model Comment {
  id        String   @id @default(cuid())
  content   String   @db.Text
  approved  Boolean  @default(true)

  // 关系
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  String?
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies   Comment[] @relation("CommentReplies")

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([parentId])
  @@map("comments")
}

// 标签
model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]

  @@map("tags")
}

// 点赞
model Like {
  id        String   @id @default(cuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  createdAt DateTime @default(now())

  @@unique([userId, postId])
  @@map("likes")
}
```

---

## Cursor 常见问题与解决方案

### FAQ 1：Cursor 不生成代码补全建议

**症状**：输入代码时没有显示任何 AI 补全建议。

**诊断步骤**：
```markdown
1. 检查 Cursor AI 是否启用
   - 设置 → Cursor → AI → Enable Inline Completions

2. 检查当前文件语言是否支持
   - 设置 → Cursor → AI → Enabled Languages
   - 确认当前语言已启用

3. 检查网络连接
   - 尝试访问 cursor.sh
   - 检查代理设置

4. 检查 API 配额
   - 设置 → Account → Usage
   - 确认还有可用额度
```

**解决方案**：
```json
// .vscode/settings.json
{
  "cursor.completion": {
    "inline": true,
    "ghostText": true,
    "enabledLanguages": ["*"],
    "disableLanguageOverrides": false
  }
}
```

### FAQ 2：Agent Mode 执行失败

**症状**：Agent Mode 无法执行操作，提示权限错误。

**解决方案**：
```markdown
1. 检查安全设置
   - 设置 → Cursor → Agent → Auto Approve

2. 确认文件权限
   - 检查项目目录权限
   - 确认有写入权限

3. 禁用冲突扩展
   - 某些扩展可能干扰 Agent

4. 重置 Agent 状态
   - 设置 → Cursor → Reset Agent Cache
```

### FAQ 3：上下文理解不准确

**症状**：AI 生成的代码与项目规范不符。

**解决方案**：
```markdown
1. 完善 Cursor Rules
   - 添加更详细的代码规范
   - 包含示例代码

2. 引用相关文件
   - 使用 @file: 引用相关代码
   - 使用 @folder: 引用整个目录

3. 提供更多上下文
   - 描述项目的技术栈
   - 说明现有的代码模式

4. 重置项目索引
   - 设置 → Cursor → Reset Project Index
```

### FAQ 4：Cursor 占用内存过高

**症状**：Cursor 运行缓慢，系统内存占用过高。

**解决方案**：
```markdown
1. 减小上下文窗口
   - 设置 → Cursor → AI → Context Window Size
   - 从 "Full Project" 改为 "Open Files"

2. 限制索引范围
   - 设置 → Cursor → Index → Include Patterns
   - 只索引 src/ 目录

3. 禁用不必要的功能
   - 关闭 hover descriptions
   - 关闭 inline suggestions

4. 增加内存限制
   - Cursor → 设置 → 内存限制
```

### FAQ 5：订阅显示未激活

**症状**：付费订阅已购买但功能仍然受限。

**解决方案**：
```markdown
1. 退出并重新登录
   - Cursor → 账户 → Sign Out
   - 重新登录

2. 清除缓存
   - macOS: Cmd+Shift+P → "Clear Cache"
   - Windows: Ctrl+Shift+P → "Clear Cache"

3. 检查订阅状态
   - https://cursor.sh/settings/account
   - 确认订阅状态为 Active

4. 联系客服
   - support@cursor.sh
   - 提供订阅确认邮件
```

### FAQ 6：与 VS Code 扩展冲突

**症状**：安装某些 VS Code 扩展后 Cursor 异常。

**解决方案**：
```markdown
1. 识别问题扩展
   - 禁用所有扩展
   - 逐个启用找出冲突

2. 查找替代扩展
   - 搜索 Cursor 兼容版本
   - 使用替代方案

3. 更新扩展
   - 确保扩展是最新版本
   - 更新 Cursor 到最新版本

4. 报告问题
   - GitHub Issues
   - 提供扩展版本信息
```

---

## Cursor 与竞品深度对比分析

### 技术架构对比

```
┌─────────────────────────────────────────────────────────────────┐
│                        架构对比图示                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Cursor                    VS Code + Copilot        Windsurf      │
│  ┌─────────┐              ┌─────────────┐         ┌─────────┐ │
│  │ 自研 IDE │              │ VS Code 插件 │         │ VS Code │ │
│  │ + AI 深度 │              │ + 云端 AI    │         │ + Cascade │ │
│  │ 集成     │              └─────────────┘         └─────────┘ │
│  └─────────┘                                               │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────────────────────────────────┐              │
│  │           AI Integration Layer           │              │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐   │              │
│  │  │Claude│ │ GPT │ │Gemini│ │ 自研 │   │              │
│  │  └─────┘ └─────┘ └─────┘ └─────┘   │              │
│  └─────────────────────────────────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 功能详细对比

| 功能维度 | Cursor | Copilot | Windsurf | Cline | Roo Code |
|---------|--------|---------|-----------|-------|----------|
| **AI 集成深度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **代码补全质量** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **多文件编辑** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **上下文理解** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **价格** | $20/月 | $10/月 | $10/月 | 免费 | 免费 |
| **本地模型** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **开源** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **自定义规则** | ✅ Cursor Rules | ✅ Custom Instructions | ✅ Rules | ✅ | ✅ |
| **Tab 补全** | ✅ | ✅ | ✅ | ❌ | ❌ |

### 场景化推荐

| 使用场景 | 推荐工具 | 原因 |
|----------|---------|------|
| **快速原型开发** | Cursor | Agent Mode 强大，多文件编辑高效 |
| **日常代码补全** | Copilot / Windsurf | 实时补全，开箱即用 |
| **隐私敏感项目** | Cline / Roo Code + Ollama | 完全本地，数据不出境 |
| **预算有限** | Windsurf / Cline | 功能完整，免费使用 |
| **企业团队** | Copilot Business / Cursor Business | 策略管理完善 |
| **GitHub 重度用户** | Copilot | PR 集成最佳 |
| **VS Code 老用户** | Windsurf / Copilot | 无缝迁移 |

### 成本效益分析

```markdown
# 年度使用成本对比

| 工具 | 月费 | 年费 | 边际成本 |
|------|------|------|----------|
| Cursor Pro | $20 | $192 | 无 |
| Copilot Pro | $10 | $100 | 无 |
| Windsurf Pro | $10 | $96 | 无 |
| Cline | $0 | $0 | API 费用 |
| Roo Code | $0 | $0 | API 费用 |

# API 成本估算（Cline/Roo Code）

| 模型 | 场景 | 月均成本 |
|------|------|----------|
| Claude Sonnet | 日常开发 | $20-50 |
| GPT-4o | 日常开发 | $15-40 |
| Ollama 本地 | 完全免费 | $0 |

结论：
- 个人开发者：Windsurf Pro 性价比最高
- 企业用户：Copilot Business 策略管理最佳
- 预算有限：Cline + Claude API 最经济
- 隐私优先：Roo Code + Ollama 零成本本地
```

---

## Cursor 选型建议与最佳实践

### 个人开发者

**推荐：Cursor Pro 或 Windsurf Pro**

```markdown
# 选择理由

Cursor Pro：
- 最佳的 AI 集成体验
- 强大的 Agent Mode
- 优秀的代码补全
- 适合追求效率的开发者

Windsurf Pro：
- 价格更低（$10/月 vs $20/月）
- Supercomplete 技术创新
- VS Code 兼容性更好
- 适合预算有限的开发者
```

### 初创团队

**推荐：Cursor Business 或 Copilot Business**

```markdown
# 选择理由

团队需求：
1. 共享代码规范
2. 使用分析
3. 权限管理
4. 成本控制

Cursor Business ($40/用户/月)：
- 团队 Cursor Rules
- 使用分析仪表板
- SSO 支持（即将推出）

Copilot Business ($19/用户/月)：
- 更低的价格
- 成熟的策略管理
- GitHub 深度集成
```

### 大型企业

**推荐：Cursor Enterprise + 自定义配置**

```markdown
# 企业需求分析

1. 合规要求
   - SOC 2 / GDPR
   - 数据本地化
   - 审计日志

2. 安全要求
   - SSO/SAML
   - IP 白名单
   - 代码隔离

3. 成本控制
   - 部门预算分配
   - 使用量监控
   - 成本上限

Cursor Enterprise：
- 自定义 SLA
- 专属客户经理
- 高级安全合规
```

### 混合使用策略

```markdown
# 最优工具组合

日常补全：GitHub Copilot (VS Code 内)
- 实时补全
- 轻量快速

复杂任务：Cursor
- 多文件编辑
- Agent Mode

本地开发：Cline + Ollama
- 完全离线
- 零成本

代码审查：Claude Code
- 最强分析能力
- 深度理解
```

---

## Cursor 生态系统

### 推荐的 Cursor 扩展

| 扩展名称 | 功能 | 优先级 |
|---------|------|--------|
| Prettier | 代码格式化 | 必需 |
| ESLint | 代码检查 | 必需 |
| GitLens | Git 可视化 | 推荐 |
| Error Lens | 错误高亮 | 推荐 |
| Thunder Client | API 测试 | 推荐 |
| Auto Rename Tag | HTML 标签同步 | 可选 |
| GitHub Copilot | 辅助补全 | 可选 |

### 主题推荐

```json
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
        },
        {
          "scope": "keyword",
          "settings": {
            "foreground": "#c678dd"
          }
        },
        {
          "scope": "string",
          "settings": {
            "foreground": "#98c379"
          }
        }
      ]
    }
  }
}
```

### 字体配置

```json
{
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontSize": 14,
  "editor.lineHeight": 1.6,
  "editor.letterSpacing": 0.5,
  "editor.fontLigatures": true,
  "editor.cursorBlinking": "smooth",
  "editor.cursorSmoothCaretAnimation": "on"
}
```

---

---

## Cursor 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 10.15+ / Windows 10+ / Linux (Ubuntu 20.04+) |
| 内存 | 8GB RAM |
| 磁盘空间 | 1GB 可用空间 |
| 网络 | 稳定的互联网连接（用于 AI 服务） |

#### 推荐配置

| 配置项 | 推荐 |
|--------|------|
| 操作系统 | macOS 13+ / Windows 11 / Ubuntu 22.04+ |
| 内存 | 16GB+ RAM |
| 处理器 | 多核处理器 |
| 网络 | 高速宽带连接 |

### 安装步骤详解

#### macOS 安装

```bash
# 方法一：官网下载
# 1. 访问 https://cursor.sh/
# 2. 点击 Download 按钮
# 3. 下载 .dmg 文件
# 4. 打开 .dmg 文件
# 5. 将 Cursor.app 拖入 Applications 文件夹

# 方法二：Homebrew 安装
brew install --cask cursor

# 方法三：命令行安装
curl -fsSL https://cursor.sh/install.sh | sh
```

#### Windows 安装

```powershell
# 方法一：官网下载
# 1. 访问 https://cursor.sh/
# 2. 下载 .exe 安装包
# 3. 运行安装程序

# 方法二：Windows Package Manager (winget)
winget install Cursor.Cursor

# 方法三：Chocolatey
choco install cursor
```

#### Linux 安装

```bash
# Debian/Ubuntu
# 1. 下载 .deb 文件
wget https://cursor.sh/download/linux?version=latest -O cursor.deb

# 2. 安装
sudo dpkg -i cursor.deb

# 3. 修复依赖（如需要）
sudo apt-get install -f

# Fedora/RHEL
sudo rpm -i cursor.rpm

# Arch Linux (使用 yay)
yay -S cursor
```

### 首次启动配置

#### 1. 账户登录

```markdown
首次启动 Cursor 时：
1. 启动画面显示后，点击 "Sign In" 按钮
2. 可选择以下登录方式：
   - GitHub 账户登录（推荐）
   - Google 账户登录
   - 邮箱密码登录
3. 完成身份验证后，Cursor 会自动同步设置
```

#### 2. 主题设置

```json
// .vscode/settings.json 中配置
{
  "workbench.colorTheme": "One Dark Pro",
  "editor.fontSize": 14,
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontLigatures": true,
  "editor.cursorStyle": "line",
  "editor.cursorBlinking": "smooth"
}
```

#### 3. AI 模型配置

```json
// .vscode/settings.json
{
  "cursor.ai": {
    "model": "claude-sonnet-4.6",
    "chatModel": "claude-opus-4.6",
    "temperature": 0.7,
    "maxTokens": 8192
  },
  "cursor.completion": {
    "inline": true,
    "ghostText": true,
    "tabComplete": true
  }
}
```

#### 4. 代理设置（如需要）

```json
{
  "cursor.proxy": {
    "enabled": true,
    "url": "http://proxy.example.com:8080",
    "bypass": [
      "localhost",
      "127.0.0.1"
    ]
  }
}
```

### Cursor Rules 进阶配置

#### 全局规则模板

```markdown
# .cursor/rules/default.mdc
# cursorules
# 全局代码规范 - 适用于所有项目

## 代码风格
- 缩进：2 空格（TypeScript/JavaScript）或 4 空格（Python）
- 行宽：100 字符
- 文件末尾：单行空行
- 字符串引号：单引号优先（JS/TS），双引号（Python）

## TypeScript 规范
- 严格模式：true
- 禁用 any：所有类型必须明确
- 接口 vs 类型别名：
  - 使用 interface 定义对象结构
  - 使用 type 定义联合类型、交叉类型
- 导出：使用命名导出优于默认导出

## 命名规范
- 变量/函数：camelCase
- 类/接口/类型：PascalCase
- 常量：UPPER_SNAKE_CASE
- 文件名：kebab-case.ts

## 安全规范
- 禁止硬编码敏感信息
- 所有 API 密钥使用环境变量
- 用户输入必须验证
- 使用参数化查询防止 SQL 注入

## 性能规范
- 避免不必要的重渲染
- 使用 useMemo/useCallback 优化
- 大数据列表使用虚拟滚动
- 图片懒加载
```

#### Next.js 项目规则

```markdown
# .cursor/rules/nextjs.mdc
# cursorules
# Next.js 14 App Router 项目规范

## 目录结构
```
src/
├── app/                    # App Router 页面
│   ├── page.tsx          # 首页
│   ├── layout.tsx        # 根布局
│   ├── globals.css       # 全局样式
│   └── [slug]/          # 动态路由
├── components/            # React 组件
│   ├── ui/              # 基础 UI 组件
│   ├── features/        # 功能组件
│   └── layouts/        # 布局组件
├── lib/                  # 工具函数
├── hooks/                # 自定义 Hooks
├── types/               # TypeScript 类型
├── services/            # API 服务层
└── stores/              # 状态管理
```

## 服务器组件 vs 客户端组件
- 默认使用服务器组件
- 仅在需要以下特性时使用 "use client"：
  - useState/useReducer
  - useEffect
  - 浏览器 API
  - 自定义事件处理

## 数据获取
- 服务端：使用 async/await 直接查询数据库
- 客户端：使用 React Query / SWR
- 缓存：使用 Next.js 的 fetch 缓存选项

## API 路由
- 使用 route.ts 文件
- 分离 HTTP 方法处理器
- 使用 Zod 进行输入验证
- 统一错误响应格式
```

---

## Cursor 快捷键大全

### macOS 系统快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **AI 基础** | 唤起 AI 对话 | `Cmd + K` | 打开 AI 聊天窗口 |
| | 接受补全建议 | `Tab` | 接受代码补全 |
| | 拒绝补全建议 | `Esc` | 拒绝当前建议 |
| | 触发补全 | `Cmd + L` | 手动触发代码补全 |
| | AI 解释 | `Cmd + Shift + L` | 解释选中代码 |
| | AI 修复 | `Cmd + Shift + E` | 修复代码错误 |
| **Agent 模式** | 启动 Agent | `Cmd + Shift + G` | 启动 Agent Mode |
| | 确认执行 | `Enter` | 确认 Agent 操作 |
| | 取消执行 | `Esc` | 取消 Agent 操作 |
| **编辑器** | 快速打开文件 | `Cmd + P` | 模糊搜索文件 |
| | 命令面板 | `Cmd + Shift + P` | 搜索命令 |
| | 全局搜索 | `Cmd + Shift + F` | 搜索所有文件 |
| | 多光标选择 | `Cmd + D` | 选择下一个匹配 |
| | 列选择 | `Option + 拖动` | 列模式选择 |
| **导航** | 转到定义 | `Cmd + 点击` | 跳转到定义 |
| | 查找引用 | `Shift + F12` | 查找所有引用 |
| | 返回 | `Cmd + U` | 返回上一个位置 |
| **终端** | 打开终端 | `` Cmd + ` `` | 切换到终端 |
| | 新终端 | `Cmd + Shift + `` ` `` | 新建终端标签 |

### Windows/Linux 快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **AI 基础** | 唤起 AI 对话 | `Ctrl + K` | 打开 AI 聊天窗口 |
| | 接受补全建议 | `Tab` | 接受代码补全 |
| | 拒绝补全建议 | `Esc` | 拒绝当前建议 |
| | 触发补全 | `Ctrl + L` | 手动触发代码补全 |
| | AI 解释 | `Ctrl + Shift + L` | 解释选中代码 |
| | AI 修复 | `Ctrl + Shift + E` | 修复代码错误 |
| **Agent 模式** | 启动 Agent | `Ctrl + Shift + G` | 启动 Agent Mode |
| | 确认执行 | `Enter` | 确认 Agent 操作 |
| | 取消执行 | `Esc` | 取消 Agent 操作 |
| **编辑器** | 快速打开文件 | `Ctrl + P` | 模糊搜索文件 |
| | 命令面板 | `Ctrl + Shift + P` | 搜索命令 |
| | 全局搜索 | `Ctrl + Shift + F` | 搜索所有文件 |
| | 多光标选择 | `Ctrl + D` | 选择下一个匹配 |
| **终端** | 打开终端 | `` Ctrl + ` `` | 切换到终端 |
| | 新终端 | `Ctrl + Shift + `` ` `` | 新建终端标签 |

### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+h",
    "command": "cursor.chat",
    "args": { "mode": "inline" },
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "cmd+shift+a",
    "command": "cursor.agent",
    "args": { "mode": "ask" },
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+r",
    "command": "cursor.refactor",
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "cmd+shift+b",
    "command": "cursor.build",
    "when": "editorTextFocus && editorLangId == 'typescript'"
  },
  {
    "key": "cmd+shift+t",
    "command": "cursor.test",
    "when": "editorTextFocus"
  }
]
```

---

## Cursor AI 提示词工程深度指南

### 提示词基础原则

#### 1. 清晰的结构

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

```markdown
"请一步步分析这个问题：

1. 首先分析代码的执行流程
2. 识别可能的性能瓶颈
3. 列出每种优化方案
4. 比较各方案的优缺点
5. 给出推荐方案及实现代码"
```

#### 2. 约束条件提示

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

### 技术栈特定提示词

#### React + TypeScript

```markdown
"创建一个 React 函数组件 UserProfile，要求：

技术栈：
- React 18
- TypeScript 5.x
- Tailwind CSS
- React Query

功能需求：
1. 通过 userId 获取用户数据
2. 显示用户头像、姓名、邮箱、简介
3. 支持编辑模式
4. 显示用户最近活动列表

代码规范：
- 使用 useQuery 获取数据
- 使用 useMutation 处理更新
- 组件最大 150 行
- 提取自定义 Hooks

错误处理：
- 加载状态：骨架屏
- 错误状态：错误提示 + 重试按钮
- 空状态：友好提示"
```

#### Next.js App Router

```markdown
"在 src/app/users/[id] 目录下创建用户详情页，要求：

技术栈：
- Next.js 14 App Router
- TypeScript
- Prisma ORM
- Zod 验证

页面结构：
1. generateMetadata 函数获取元数据
2. 页面组件为异步服务器组件
3. 包含用户信息卡片
4. 用户帖子列表（分页）
5. 用户设置面板

数据获取：
- 直接使用 Prisma 查询（服务端）
- 使用 revalidate 选项设置缓存策略
- 错误处理使用 error.tsx

API 路由：
- GET /api/users/[id] - 获取用户详情
- PATCH /api/users/[id] - 更新用户信息
- 使用 Zod 验证请求数据"
```

#### Node.js + Express

```markdown
"创建 src/routes/userRoutes.ts，包含完整的用户 CRUD 路由，要求：

技术栈：
- Express.js
- TypeScript
- Prisma
- Zod

路由端点：
- GET /users - 列表（分页、筛选）
- GET /users/:id - 详情
- POST /users - 创建
- PUT /users/:id - 更新
- DELETE /users/:id - 删除

功能要求：
1. 输入验证（Zod Schema）
2. 错误处理中间件
3. 认证中间件（JWT）
4. 权限控制（admin/user）
5. 分页支持

响应格式：
```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
  };
  meta?: {
    page: number;
    limit: number;
    total: number;
  };
}
```"
```

---

## Cursor 与现代技术栈深度集成

### Next.js 15 + React 19 集成

#### 项目初始化

```markdown
# Cursor AI 辅助初始化 Next.js 项目

提示词：
"创建一个 Next.js 15 项目，要求：
- 使用 App Router
- TypeScript
- Tailwind CSS
- Prisma ORM + PostgreSQL
- NextAuth.js v5 认证
- React Query 管理客户端状态
- Zod 表单验证

配置：
- ESLint + Prettier
- Husky + lint-staged
- Jest + React Testing Library

目录结构：
- 使用 feature-based 结构
- API 路由在 app/api/
- 组件在 components/
- 类型在 types/"
```

#### Cursor 生成的代码示例

**src/app/users/page.tsx**

```typescript
import { Suspense } from 'react';
import { UserList } from '@/components/features/UserList';
import { UserListSkeleton } from '@/components/ui/skeletons/UserListSkeleton';
import { getUsers } from '@/services/userService';

interface UsersPageProps {
  searchParams: Promise<{ page?: string; limit?: string }>;
}

export default async function UsersPage({ searchParams }: UsersPageProps) {
  const params = await searchParams;
  const page = parseInt(params.page || '1', 10);
  const limit = parseInt(params.limit || '20', 10);

  const { users, total } = await getUsers({ page, limit });

  return (
    <div className="container mx-auto py-8">
      <div className="mb-8">
        <h1 className="text-3xl font-bold">用户列表</h1>
        <p className="text-muted-foreground mt-2">
          共 {total} 位用户
        </p>
      </div>

      <Suspense fallback={<UserListSkeleton count={limit} />}>
        <UserList users={users} total={total} page={page} limit={limit} />
      </Suspense>
    </div>
  );
}

export const metadata = {
  title: '用户列表 - My App',
  description: '浏览和管理所有用户',
};
```

**src/services/userService.ts**

```typescript
import { prisma } from '@/lib/prisma';
import { cache } from 'react';

export interface GetUsersParams {
  page?: number;
  limit?: number;
  search?: string;
  role?: 'USER' | 'ADMIN';
}

export interface GetUsersResult {
  users: User[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export const getUsers = cache(async (params: GetUsersParams): Promise<GetUsersResult> => {
  const { page = 1, limit = 20, search, role } = params;
  const skip = (page - 1) * limit;

  const where = {
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' as const } },
        { email: { contains: search, mode: 'insensitive' as const } },
      ],
    }),
    ...(role && { role }),
  };

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
        role: true,
        createdAt: true,
        _count: { select: { posts: true } },
      },
    }),
    prisma.user.count({ where }),
  ]);

  return {
    users: users as User[],
    total,
    page,
    limit,
    totalPages: Math.ceil(total / limit),
  };
});
```

### React Native + Expo 集成

#### Cursor 辅助开发

```markdown
# Cursor AI 辅助 React Native 开发

提示词：
"使用 Expo + React Native 创建一个待办事项应用，要求：
- Expo SDK 52
- TypeScript
- Expo Router 文件路由
- Zustand 状态管理
- AsyncStorage 持久化
- 支持深色模式
- 支持 iOS/Android 双平台

功能：
1. 创建/编辑/删除待办
2. 待办分组
3. 截止日期提醒
4. 本地推送通知
5. 数据导出为 JSON"
```

#### 生成的代码示例

**app/_layout.tsx**

```typescript
import { DarkTheme, LightTheme } from '@/theme';
import { useColorScheme } from '@/hooks/useColorScheme';
import { StatusBar } from 'expo-status-bar';
import { Stack } from 'expo-router';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  const colorScheme = useColorScheme();

  return (
    <SafeAreaProvider>
      <GestureHandlerRootView style={{ flex: 1 }}>
        <Stack
          screenOptions={{
            headerStyle: {
              backgroundColor: colorScheme === 'dark' 
                ? DarkTheme.colors.background 
                : LightTheme.colors.background,
            },
            headerTintColor: colorScheme === 'dark'
              ? DarkTheme.colors.text
              : LightTheme.colors.text,
          }}
        >
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="+not-found" options={{ title: 'Oops!' }} />
        </Stack>
        <StatusBar style={colorScheme === 'dark' ? 'light' : 'dark'} />
      </GestureHandlerRootView>
    </SafeAreaProvider>
  );
}
```

**app/(tabs)/index.tsx**

```typescript
import { useState } from 'react';
import { View, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';
import { TextInput } from '@/components/ui/TextInput';
import { TodoItem } from '@/components/features/TodoItem';
import { useTodos } from '@/stores/todoStore';
import { useTheme } from '@/hooks/useTheme';
import { Todo } from '@/types';

export default function TodosScreen() {
  const router = useRouter();
  const { theme } = useTheme();
  const [newTodoTitle, setNewTodoTitle] = useState('');
  const { todos, addTodo, toggleTodo, deleteTodo } = useTodos();

  const handleAddTodo = async () => {
    if (!newTodoTitle.trim()) return;
    
    const newTodo: Todo = {
      id: Date.now().toString(),
      title: newTodoTitle.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
      priority: 'medium',
    };
    
    addTodo(newTodo);
    setNewTodoTitle('');
  };

  const renderItem = ({ item }: { item: Todo }) => (
    <TodoItem
      todo={item}
      onToggle={() => toggleTodo(item.id)}
      onDelete={() => deleteTodo(item.id)}
      onPress={() => router.push(`/todo/${item.id}`)}
    />
  );

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <View style={styles.inputContainer}>
        <TextInput
          value={newTodoTitle}
          onChangeText={setNewTodoTitle}
          placeholder="添加新待办..."
          onSubmitEditing={handleAddTodo}
          returnKeyType="done"
        />
        <TouchableOpacity
          style={[styles.addButton, { backgroundColor: theme.colors.primary }]}
          onPress={handleAddTodo}
        >
          <Text style={styles.addButtonText}>添加</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={todos}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.list}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
}
```

### 数据库设计集成

#### Prisma Schema 最佳实践

```prisma
// prisma/schema.prisma
// Cursor AI 辅助生成的完整数据模型

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 用户模型
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  avatar        String?
  passwordHash   String
  role          Role      @default(USER)
  emailVerified DateTime?
  
  // 关系
  accounts      Account[]
  sessions      Session[]
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  
  // 时间戳
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?

  @@index([email])
  @@index([role])
  @@map("users")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

// OAuth 账户
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refreshToken      String? @db.Text
  accessToken      String? @db.Text
  expiresAt        Int?
  tokenType        String?
  scope            String?
  idToken          String? @db.Text
  sessionState     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

// 会话
model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

// 帖子
model Post {
  id         String   @id @default(cuid())
  title      String   @db.VarChar(255)
  slug       String   @unique
  content    String   @db.Text
  excerpt    String?  @db.VarChar(500)
  coverImage String?
  published  Boolean  @default(false)
  publishedAt DateTime?
  
  // 关系
  author     User     @relation(fields: [authorId], references: [id])
  authorId   String
  comments   Comment[]
  tags       Tag[]
  likes      Like[]

  // SEO
  metaTitle       String? @db.VarChar(70)
  metaDescription String? @db.VarChar(160)

  // 时间戳
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([slug])
  @@index([published])
  @@map("posts")
}

// 评论
model Comment {
  id        String   @id @default(cuid())
  content   String   @db.Text
  approved  Boolean  @default(true)

  // 关系
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  String?
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies   Comment[] @relation("CommentReplies")

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([parentId])
  @@map("comments")
}

// 标签
model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]

  @@map("tags")
}

// 点赞
model Like {
  id        String   @id @default(cuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  createdAt DateTime @default(now())

  @@unique([userId, postId])
  @@map("likes")
}
```

---

## Cursor 常见问题与解决方案

### FAQ 1：Cursor 不生成代码补全建议

**症状**：输入代码时没有显示任何 AI 补全建议。

**诊断步骤**：
```markdown
1. 检查 Cursor AI 是否启用
   - 设置 → Cursor → AI → Enable Inline Completions

2. 检查当前文件语言是否支持
   - 设置 → Cursor → AI → Enabled Languages
   - 确认当前语言已启用

3. 检查网络连接
   - 尝试访问 cursor.sh
   - 检查代理设置

4. 检查 API 配额
   - 设置 → Account → Usage
   - 确认还有可用额度
```

**解决方案**：
```json
// .vscode/settings.json
{
  "cursor.completion": {
    "inline": true,
    "ghostText": true,
    "enabledLanguages": ["*"],
    "disableLanguageOverrides": false
  }
}
```

### FAQ 2：Agent Mode 执行失败

**症状**：Agent Mode 无法执行操作，提示权限错误。

**解决方案**：
```markdown
1. 检查安全设置
   - 设置 → Cursor → Agent → Auto Approve

2. 确认文件权限
   - 检查项目目录权限
   - 确认有写入权限

3. 禁用冲突扩展
   - 某些扩展可能干扰 Agent

4. 重置 Agent 状态
   - 设置 → Cursor → Reset Agent Cache
```

### FAQ 3：上下文理解不准确

**症状**：AI 生成的代码与项目规范不符。

**解决方案**：
```markdown
1. 完善 Cursor Rules
   - 添加更详细的代码规范
   - 包含示例代码

2. 引用相关文件
   - 使用 @file: 引用相关代码
   - 使用 @folder: 引用整个目录

3. 提供更多上下文
   - 描述项目的技术栈
   - 说明现有的代码模式

4. 重置项目索引
   - 设置 → Cursor → Reset Project Index
```

### FAQ 4：Cursor 占用内存过高

**症状**：Cursor 运行缓慢，系统内存占用过高。

**解决方案**：
```markdown
1. 减小上下文窗口
   - 设置 → Cursor → AI → Context Window Size
   - 从 "Full Project" 改为 "Open Files"

2. 限制索引范围
   - 设置 → Cursor → Index → Include Patterns
   - 只索引 src/ 目录

3. 禁用不必要的功能
   - 关闭 hover descriptions
   - 关闭 inline suggestions

4. 增加内存限制
   - Cursor → 设置 → 内存限制
```

### FAQ 5：订阅显示未激活

**症状**：付费订阅已购买但功能仍然受限。

**解决方案**：
```markdown
1. 退出并重新登录
   - Cursor → 账户 → Sign Out
   - 重新登录

2. 清除缓存
   - macOS: Cmd+Shift+P → "Clear Cache"
   - Windows: Ctrl+Shift+P → "Clear Cache"

3. 检查订阅状态
   - https://cursor.sh/settings/account
   - 确认订阅状态为 Active

4. 联系客服
   - support@cursor.sh
   - 提供订阅确认邮件
```

### FAQ 6：与 VS Code 扩展冲突

**症状**：安装某些 VS Code 扩展后 Cursor 异常。

**解决方案**：
```markdown
1. 识别问题扩展
   - 禁用所有扩展
   - 逐个启用找出冲突

2. 查找替代扩展
   - 搜索 Cursor 兼容版本
   - 使用替代方案

3. 更新扩展
   - 确保扩展是最新版本
   - 更新 Cursor 到最新版本

4. 报告问题
   - GitHub Issues
   - 提供扩展版本信息
```

---

## Cursor 与竞品深度对比分析

### 技术架构对比

```
┌─────────────────────────────────────────────────────────────────┐
│                        架构对比图示                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Cursor                    VS Code + Copilot        Windsurf      │
│  ┌─────────┐              ┌─────────────┐         ┌─────────┐ │
│  │ 自研 IDE │              │ VS Code 插件 │         │ VS Code │ │
│  │ + AI 深度 │              │ + 云端 AI    │         │ + Cascade │ │
│  │ 集成     │              └─────────────┘         └─────────┘ │
│  └─────────┘                                               │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────────────────────────────────┐              │
│  │           AI Integration Layer           │              │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐   │              │
│  │  │Claude│ │ GPT │ │Gemini│ │ 自研 │   │              │
│  │  └─────┘ └─────┘ └─────┘ └─────┘   │              │
│  └─────────────────────────────────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 功能详细对比

| 功能维度 | Cursor | Copilot | Windsurf | Cline | Roo Code |
|---------|--------|---------|-----------|-------|----------|
| **AI 集成深度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **代码补全质量** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **多文件编辑** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **上下文理解** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **价格** | $20/月 | $10/月 | $10/月 | 免费 | 免费 |
| **本地模型** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **开源** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **自定义规则** | ✅ Cursor Rules | ✅ Custom Instructions | ✅ Rules | ✅ | ✅ |
| **Tab 补全** | ✅ | ✅ | ✅ | ❌ | ❌ |

### 场景化推荐

| 使用场景 | 推荐工具 | 原因 |
|----------|---------|------|
| **快速原型开发** | Cursor | Agent Mode 强大，多文件编辑高效 |
| **日常代码补全** | Copilot / Windsurf | 实时补全，开箱即用 |
| **隐私敏感项目** | Cline / Roo Code + Ollama | 完全本地，数据不出境 |
| **预算有限** | Windsurf / Cline | 功能完整，免费使用 |
| **企业团队** | Copilot Business / Cursor Business | 策略管理完善 |
| **GitHub 重度用户** | Copilot | PR 集成最佳 |
| **VS Code 老用户** | Windsurf / Copilot | 无缝迁移 |

### 成本效益分析

```markdown
# 年度使用成本对比

| 工具 | 月费 | 年费 | 边际成本 |
|------|------|------|----------|
| Cursor Pro | $20 | $192 | 无 |
| Copilot Pro | $10 | $100 | 无 |
| Windsurf Pro | $10 | $96 | 无 |
| Cline | $0 | $0 | API 费用 |
| Roo Code | $0 | $0 | API 费用 |

# API 成本估算（Cline/Roo Code）

| 模型 | 场景 | 月均成本 |
|------|------|----------|
| Claude Sonnet | 日常开发 | $20-50 |
| GPT-4o | 日常开发 | $15-40 |
| Ollama 本地 | 完全免费 | $0 |

结论：
- 个人开发者：Windsurf Pro 性价比最高
- 企业用户：Copilot Business 策略管理最佳
- 预算有限：Cline + Claude API 最经济
- 隐私优先：Roo Code + Ollama 零成本本地
```

---

## Cursor 选型建议与最佳实践

### 个人开发者

**推荐：Cursor Pro 或 Windsurf Pro**

```markdown
# 选择理由

Cursor Pro：
- 最佳的 AI 集成体验
- 强大的 Agent Mode
- 优秀的代码补全
- 适合追求效率的开发者

Windsurf Pro：
- 价格更低（$10/月 vs $20/月）
- Supercomplete 技术创新
- VS Code 兼容性更好
- 适合预算有限的开发者
```

### 初创团队

**推荐：Cursor Business 或 Copilot Business**

```markdown
# 选择理由

团队需求：
1. 共享代码规范
2. 使用分析
3. 权限管理
4. 成本控制

Cursor Business ($40/用户/月)：
- 团队 Cursor Rules
- 使用分析仪表板
- SSO 支持（即将推出）

Copilot Business ($19/用户/月)：
- 更低的价格
- 成熟的策略管理
- GitHub 深度集成
```

### 大型企业

**推荐：Cursor Enterprise + 自定义配置**

```markdown
# 企业需求分析

1. 合规要求
   - SOC 2 / GDPR
   - 数据本地化
   - 审计日志

2. 安全要求
   - SSO/SAML
   - IP 白名单
   - 代码隔离

3. 成本控制
   - 部门预算分配
   - 使用量监控
   - 成本上限

Cursor Enterprise：
- 自定义 SLA
- 专属客户经理
- 高级安全合规
```

### 混合使用策略

```markdown
# 最优工具组合

日常补全：GitHub Copilot (VS Code 内)
- 实时补全
- 轻量快速

复杂任务：Cursor
- 多文件编辑
- Agent Mode

本地开发：Cline + Ollama
- 完全离线
- 零成本

代码审查：Claude Code
- 最强分析能力
- 深度理解
```

---

## Cursor 生态系统

### 推荐的 Cursor 扩展

| 扩展名称 | 功能 | 优先级 |
|---------|------|--------|
| Prettier | 代码格式化 | 必需 |
| ESLint | 代码检查 | 必需 |
| GitLens | Git 可视化 | 推荐 |
| Error Lens | 错误高亮 | 推荐 |
| Thunder Client | API 测试 | 推荐 |
| Auto Rename Tag | HTML 标签同步 | 可选 |
| GitHub Copilot | 辅助补全 | 可选 |

### 主题推荐

```json
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
        },
        {
          "scope": "keyword",
          "settings": {
            "foreground": "#c678dd"
          }
        },
        {
          "scope": "string",
          "settings": {
            "foreground": "#98c379"
          }
        }
      ]
    }
  }
}
```

### 字体配置

```json
{
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontSize": 14,
  "editor.lineHeight": 1.6,
  "editor.letterSpacing": 0.5,
  "editor.fontLigatures": true,
  "editor.cursorBlinking": "smooth",
  "editor.cursorSmoothCaretAnimation": "on"
}
```

---

---

## Cursor 高级实战场景与工作流

### 场景一：全栈项目从零构建

#### 项目规划与初始化

本场景展示如何使用 Cursor Agent Mode 从零开始构建一个完整的全栈项目。这是一个复杂的实战任务，需要多文件协同和完整的技术栈实现。

```markdown
# 任务描述
创建一个完整的博客平台，包含：
- 前端：Next.js 14 + TypeScript + Tailwind CSS
- 后端：Node.js + Express + Prisma ORM
- 数据库：PostgreSQL
- 认证：JWT + Refresh Token
- 部署：Docker + GitHub Actions

要求：
1. 完整的用户认证系统
2. 文章 CRUD 操作
3. 评论系统
4. 富文本编辑器
5. 图片上传（使用 S3）
6. 响应式设计
7. SEO 优化
8. 单元测试覆盖 80%+
```

#### 执行流程详解

```typescript
// 步骤 1：项目结构设计
// Cursor Agent 会生成完整的项目结构

/*
项目结构：
├── client/                    # Next.js 前端
│   ├── src/
│   │   ├── app/            # App Router 页面
│   │   │   ├── page.tsx           # 首页
│   │   │   ├── layout.tsx         # 根布局
│   │   │   ├── globals.css         # 全局样式
│   │   │   ├── blog/
│   │   │   │   ├── page.tsx       # 博客列表
│   │   │   │   └── [slug]/        # 博客详情
│   │   │   ├── auth/
│   │   │   │   ├── login/         # 登录页
│   │   │   │   └── register/      # 注册页
│   │   │   └── api/               # API 路由
│   │   ├── components/            # React 组件
│   │   │   ├── ui/               # 基础 UI 组件
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Modal.tsx
│   │   │   │   └── Card.tsx
│   │   │   ├── blog/             # 博客相关组件
│   │   │   │   ├── BlogCard.tsx
│   │   │   │   ├── BlogEditor.tsx
│   │   │   │   └── CommentSection.tsx
│   │   │   └── layout/           # 布局组件
│   │   ├── hooks/                # 自定义 Hooks
│   │   │   ├── useAuth.ts
│   │   │   ├── useBlog.ts
│   │   │   └── useComment.ts
│   │   ├── lib/                  # 工具函数
│   │   │   ├── api.ts            # API 客户端
│   │   │   ├── auth.ts           # 认证工具
│   │   │   └── utils.ts          # 通用工具
│   │   ├── stores/               # 状态管理（Zustand）
│   │   │   ├── authStore.ts
│   │   │   └── blogStore.ts
│   │   └── types/                # TypeScript 类型
│   │       ├── user.ts
│   │       ├── blog.ts
│   │       └── api.ts
│   ├── public/                   # 静态资源
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js
│   ├── tailwind.config.ts
│   └── .env.local
│
├── server/                     # Express 后端
│   ├── src/
│   │   ├── controllers/         # 控制器
│   │   │   ├── authController.ts
│   │   │   ├── blogController.ts
│   │   │   └── commentController.ts
│   │   ├── services/           # 业务逻辑
│   │   │   ├── authService.ts
│   │   │   ├── blogService.ts
│   │   │   └── emailService.ts
│   │   ├── models/             # Prisma 模型
│   │   │   ├── User.ts
│   │   │   ├── Post.ts
│   │   │   └── Comment.ts
│   │   ├── middleware/         # 中间件
│   │   │   ├── auth.ts         # JWT 认证
│   │   │   ├── validate.ts     # 请求验证
│   │   │   ├── rateLimit.ts   # 限流
│   │   │   └── errorHandler.ts # 错误处理
│   │   ├── routes/             # 路由定义
│   │   │   ├── auth.ts
│   │   │   ├── blog.ts
│   │   │   └── comment.ts
│   │   ├── utils/              # 工具函数
│   │   │   ├── jwt.ts
│   │   │   ├── password.ts
│   │   │   └── validation.ts
│   │   └── config/             # 配置文件
│   │       ├── database.ts
│   │       ├── redis.ts
│   │       └── s3.ts
│   ├── prisma/
│   │   ├── schema.prisma
│   │   └── migrations/
│   ├── tests/                  # 测试文件
│   │   ├── unit/
│   │   └── integration/
│   ├── package.json
│   └── tsconfig.json
│
├── docker-compose.yml          # Docker 配置
├── Dockerfile.client           # 前端 Docker
├── Dockerfile.server           # 后端 Docker
├── .github/
│   └── workflows/
│       └── ci.yml             # CI/CD 配置
└── README.md
*/
```

#### 核心代码实现

```typescript
// client/src/types/user.ts
export interface User {
  id: string;
  email: string;
  username: string;
  displayName: string;
  avatar?: string;
  bio?: string;
  role: 'USER' | 'ADMIN';
  emailVerified: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export interface UserProfile extends User {
  postsCount: number;
  commentsCount: number;
  followersCount: number;
  followingCount: number;
}

export interface RegisterRequest {
  email: string;
  username: string;
  displayName: string;
  password: string;
}

export interface LoginRequest {
  email: string;
  password: string;
}

export interface AuthResponse {
  user: User;
  accessToken: string;
  refreshToken: string;
}

export interface UpdateProfileRequest {
  displayName?: string;
  bio?: string;
  avatar?: string;
}

// client/src/hooks/useAuth.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { User, AuthResponse } from '@/types/auth';

interface AuthState {
  user: User | null;
  accessToken: string | null;
  refreshToken: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  
  // Actions
  login: (email: string, password: string) => Promise<void>;
  register: (data: RegisterRequest) => Promise<void>;
  logout: () => void;
  refreshAccessToken: () => Promise<void>;
  updateUser: (user: User) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,
      isLoading: false,

      login: async (email: string, password: string) => {
        set({ isLoading: true });
        try {
          const response = await fetch('/api/auth/login', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ email, password }),
          });

          if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message || '登录失败');
          }

          const data: AuthResponse = await response.json();
          
          set({
            user: data.user,
            accessToken: data.accessToken,
            refreshToken: data.refreshToken,
            isAuthenticated: true,
            isLoading: false,
          });
        } catch (error) {
          set({ isLoading: false });
          throw error;
        }
      },

      register: async (data: RegisterRequest) => {
        set({ isLoading: true });
        try {
          const response = await fetch('/api/auth/register', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(data),
          });

          if (!response.ok) {
            const error = await response.json();
            throw new Error(error.message || '注册失败');
          }

          const result: AuthResponse = await response.json();
          
          set({
            user: result.user,
            accessToken: result.accessToken,
            refreshToken: result.refreshToken,
            isAuthenticated: true,
            isLoading: false,
          });
        } catch (error) {
          set({ isLoading: false });
          throw error;
        }
      },

      logout: () => {
        set({
          user: null,
          accessToken: null,
          refreshToken: null,
          isAuthenticated: false,
        });
      },

      refreshAccessToken: async () => {
        const { refreshToken } = get();
        if (!refreshToken) {
          throw new Error('No refresh token');
        }

        try {
          const response = await fetch('/api/auth/refresh', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ refreshToken }),
          });

          if (!response.ok) {
            get().logout();
            throw new Error('Token refresh failed');
          }

          const data: { accessToken: string } = await response.json();
          set({ accessToken: data.accessToken });
        } catch (error) {
          get().logout();
          throw error;
        }
      },

      updateUser: (user: User) => {
        set({ user });
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        accessToken: state.accessToken,
        refreshToken: state.refreshToken,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);

// server/src/controllers/authController.ts
import { Request, Response, NextFunction } from 'express';
import { authService } from '../services/authService';
import { validationResult } from 'express-validator';

export class AuthController {
  /**
   * 用户注册
   * POST /api/auth/register
   */
  static async register(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        res.status(400).json({ errors: errors.array() });
        return;
      }

      const { email, username, displayName, password } = req.body;
      const result = await authService.register({
        email,
        username,
        displayName,
        password,
      });

      res.status(201).json(result);
    } catch (error) {
      next(error);
    }
  }

  /**
   * 用户登录
   * POST /api/auth/login
   */
  static async login(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        res.status(400).json({ errors: errors.array() });
        return;
      }

      const { email, password } = req.body;
      const result = await authService.login(email, password);

      res.json(result);
    } catch (error) {
      next(error);
    }
  }

  /**
   * 刷新 Access Token
   * POST /api/auth/refresh
   */
  static async refreshToken(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { refreshToken } = req.body;
      const result = await authService.refreshToken(refreshToken);
      res.json(result);
    } catch (error) {
      next(error);
    }
  }

  /**
   * 获取当前用户信息
   * GET /api/auth/me
   */
  static async getCurrentUser(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const userId = req.user?.id;
      if (!userId) {
        res.status(401).json({ message: '未授权' });
        return;
      }

      const user = await authService.getUserById(userId);
      res.json({ user });
    } catch (error) {
      next(error);
    }
  }

  /**
   * 更新用户资料
   * PATCH /api/auth/profile
   */
  static async updateProfile(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const userId = req.user?.id;
      if (!userId) {
        res.status(401).json({ message: '未授权' });
        return;
      }

      const updates = req.body;
      const user = await authService.updateProfile(userId, updates);
      res.json({ user });
    } catch (error) {
      next(error);
    }
  }

  /**
   * 修改密码
   * POST /api/auth/change-password
   */
  static async changePassword(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const userId = req.user?.id;
      if (!userId) {
        res.status(401).json({ message: '未授权' });
        return;
      }

      const { currentPassword, newPassword } = req.body;
      await authService.changePassword(userId, currentPassword, newPassword);
      res.json({ message: '密码修改成功' });
    } catch (error) {
      next(error);
    }
  }

  /**
   * 忘记密码 - 发送重置邮件
   * POST /api/auth/forgot-password
   */
  static async forgotPassword(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { email } = req.body;
      await authService.sendPasswordResetEmail(email);
      res.json({ message: '如果邮箱存在，重置链接已发送' });
    } catch (error) {
      next(error);
    }
  }

  /**
   * 重置密码
   * POST /api/auth/reset-password
   */
  static async resetPassword(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { token, newPassword } = req.body;
      await authService.resetPassword(token, newPassword);
      res.json({ message: '密码重置成功' });
    } catch (error) {
      next(error);
    }
  }

  /**
   * 登出
   * POST /api/auth/logout
   */
  static async logout(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const userId = req.user?.id;
      if (userId) {
        await authService.logout(userId);
      }
      res.json({ message: '登出成功' });
    } catch (error) {
      next(error);
    }
  }
}
```

### 场景二：遗留代码现代化改造

#### 场景背景

本场景展示如何使用 Cursor 将遗留 JavaScript 代码现代化为 TypeScript，并应用现代最佳实践。

```markdown
# 任务描述
将一个遗留的 jQuery + JavaScript 项目现代化为 React + TypeScript：
- 原项目：jQuery 3.x + 原生 JavaScript
- 目标：React 18 + TypeScript + Vite
- 需要保留所有业务逻辑
- 应用现代架构模式和最佳实践
- 保持 API 兼容性
```

#### 代码对比与转换

```javascript
// ❌ 遗留代码 - jQuery 实现
// old/js/main.js

$(document).ready(function() {
  // 用户列表加载
  $('#user-list').on('click', '.user-item', function() {
    const userId = $(this).data('id');
    $.ajax({
      url: '/api/users/' + userId,
      method: 'GET',
      success: function(data) {
        $('#user-name').text(data.name);
        $('#user-email').text(data.email);
        $('#user-modal').modal('show');
      },
      error: function(xhr) {
        alert('加载失败: ' + xhr.responseText);
      }
    });
  });

  // 用户创建表单
  $('#create-user-form').on('submit', function(e) {
    e.preventDefault();
    const formData = {
      name: $('#name-input').val(),
      email: $('#email-input').val(),
      role: $('#role-select').val()
    };
    
    $.ajax({
      url: '/api/users',
      method: 'POST',
      contentType: 'application/json',
      data: JSON.stringify(formData),
      success: function() {
        location.reload();
      },
      error: function(xhr) {
        const error = JSON.parse(xhr.responseText);
        alert('创建失败: ' + error.message);
      }
    });
  });

  // 数据表格排序
  let sortColumn = 'name';
  let sortDirection = 'asc';
  
  $('.sortable').on('click', function() {
    const column = $(this).data('column');
    if (sortColumn === column) {
      sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      sortColumn = column;
      sortDirection = 'asc';
    }
    
    const rows = $('#user-table tbody tr').get();
    rows.sort(function(a, b) {
      const aVal = $(a).find('.' + column).text();
      const bVal = $(b).find('.' + column).text();
      
      if (sortDirection === 'asc') {
        return aVal.localeCompare(bVal);
      } else {
        return bVal.localeCompare(aVal);
      }
    });
    
    $.each(rows, function(index, row) {
      $('#user-table tbody').append(row);
    });
  });
});
```

```typescript
// ✅ 现代实现 - React + TypeScript
// src/components/UserList.tsx
import React, { useState, useMemo, useCallback } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { userApi } from '@/services/api';
import { Button } from '@/components/ui/Button';
import { Modal } from '@/components/ui/Modal';
import { UserForm } from './UserForm';
import { Skeleton } from '@/components/ui/Skeleton';
import { useToast } from '@/hooks/useToast';
import type { User, CreateUserData, SortConfig, SortDirection } from '@/types';

interface UserListProps {
  initialUsers?: User[];
}

type SortField = 'name' | 'email' | 'role' | 'createdAt';

export const UserList: React.FC<UserListProps> = ({ initialUsers }) => {
  const queryClient = useQueryClient();
  const { showToast } = useToast();
  
  // 状态管理
  const [selectedUser, setSelectedUser] = useState<User | null>(null);
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [isFormOpen, setIsFormOpen] = useState(false);
  const [sortConfig, setSortConfig] = useState<SortConfig<SortField>>({
    field: 'name',
    direction: 'asc',
  });
  const [searchQuery, setSearchQuery] = useState('');

  // React Query 数据获取
  const {
    data: users = initialUsers || [],
    isLoading,
    error,
  } = useQuery({
    queryKey: ['users'],
    queryFn: userApi.getAll,
    staleTime: 5 * 60 * 1000, // 5分钟内不重新获取
  });

  // 创建用户 mutation
  const createMutation = useMutation({
    mutationFn: userApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      setIsFormOpen(false);
      showToast('用户创建成功', 'success');
    },
    onError: (error: Error) => {
      showToast(`创建失败: ${error.message}`, 'error');
    },
  });

  // 排序处理
  const handleSort = useCallback((field: SortField) => {
    setSortConfig((prev) => ({
      field,
      direction: prev.field === field && prev.direction === 'asc' ? 'desc' : 'asc',
    }));
  }, []);

  // 筛选和排序后的用户列表
  const processedUsers = useMemo(() => {
    let result = [...users];

    // 搜索过滤
    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter(
        (user) =>
          user.name.toLowerCase().includes(query) ||
          user.email.toLowerCase().includes(query)
      );
    }

    // 排序
    result.sort((a, b) => {
      const aValue = a[sortConfig.field];
      const bValue = b[sortConfig.field];

      if (aValue < bValue) {
        return sortConfig.direction === 'asc' ? -1 : 1;
      }
      if (aValue > bValue) {
        return sortConfig.direction === 'asc' ? 1 : -1;
      }
      return 0;
    });

    return result;
  }, [users, searchQuery, sortConfig]);

  // 用户点击处理
  const handleUserClick = useCallback(async (user: User) => {
    try {
      const fullUser = await userApi.getById(user.id);
      setSelectedUser(fullUser);
      setIsModalOpen(true);
    } catch (error) {
      showToast('加载用户详情失败', 'error');
    }
  }, [showToast]);

  // 表单提交处理
  const handleFormSubmit = useCallback(
    async (data: CreateUserData) => {
      await createMutation.mutateAsync(data);
    },
    [createMutation]
  );

  // 渲染排序图标
  const renderSortIcon = (field: SortField) => {
    if (sortConfig.field !== field) return null;
    return sortConfig.direction === 'asc' ? ' ↑' : ' ↓';
  };

  if (isLoading) {
    return (
      <div className="space-y-4">
        <Skeleton className="h-10 w-full" />
        <Skeleton className="h-64 w-full" />
      </div>
    );
  }

  if (error) {
    return (
      <div className="text-red-500">
        加载用户列表失败，请重试
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {/* 工具栏 */}
      <div className="flex justify-between items-center">
        <input
          type="text"
          placeholder="搜索用户..."
          value={searchQuery}
          onChange={(e) => setSearchQuery(e.target.value)}
          className="px-4 py-2 border rounded-lg"
        />
        <Button onClick={() => setIsFormOpen(true)}>
          创建用户
        </Button>
      </div>

      {/* 用户表格 */}
      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th
                className="sortable px-6 py-3 text-left cursor-pointer"
                onClick={() => handleSort('name')}
              >
                姓名{renderSortIcon('name')}
              </th>
              <th
                className="sortable px-6 py-3 text-left cursor-pointer"
                onClick={() => handleSort('email')}
              >
                邮箱{renderSortIcon('email')}
              </th>
              <th
                className="sortable px-6 py-3 text-left cursor-pointer"
                onClick={() => handleSort('role')}
              >
                角色{renderSortIcon('role')}
              </th>
              <th className="px-6 py-3 text-left">操作</th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {processedUsers.map((user) => (
              <tr
                key={user.id}
                className="hover:bg-gray-50 cursor-pointer"
                onClick={() => handleUserClick(user)}
              >
                <td className="name px-6 py-4">{user.name}</td>
                <td className="email px-6 py-4">{user.email}</td>
                <td className="role px-6 py-4">
                  <span className={`px-2 py-1 rounded text-sm ${
                    user.role === 'ADMIN' ? 'bg-purple-100 text-purple-800' : 'bg-blue-100 text-blue-800'
                  }`}>
                    {user.role}
                  </span>
                </td>
                <td className="px-6 py-4">
                  <Button
                    variant="ghost"
                    size="sm"
                    onClick={(e) => {
                      e.stopPropagation();
                      handleUserClick(user);
                    }}
                  >
                    查看详情
                  </Button>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* 用户详情弹窗 */}
      <Modal
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        title="用户详情"
      >
        {selectedUser && (
          <div className="space-y-4">
            <div>
              <label className="text-sm text-gray-500">姓名</label>
              <p className="font-medium">{selectedUser.name}</p>
            </div>
            <div>
              <label className="text-sm text-gray-500">邮箱</label>
              <p className="font-medium">{selectedUser.email}</p>
            </div>
            <div>
              <label className="text-sm text-gray-500">角色</label>
              <p className="font-medium">{selectedUser.role}</p>
            </div>
            <div>
              <label className="text-sm text-gray-500">创建时间</label>
              <p className="font-medium">
                {new Date(selectedUser.createdAt).toLocaleString()}
              </p>
            </div>
          </div>
        )}
      </Modal>

      {/* 创建用户表单 */}
      <Modal
        isOpen={isFormOpen}
        onClose={() => setIsFormOpen(false)}
        title="创建用户"
      >
        <UserForm
          onSubmit={handleFormSubmit}
          onCancel={() => setIsFormOpen(false)}
          isLoading={createMutation.isPending}
        />
      </Modal>
    </div>
  );
};
```

### 场景三：性能优化专项

#### 场景目标

本场景展示如何使用 Cursor 识别和解决性能问题。

```markdown
# 性能优化任务
分析并优化 src/features/dashboard 模块的性能：
1. 识别渲染性能问题
2. 优化不必要的重渲染
3. 实现虚拟滚动处理大列表
4. 添加骨架屏优化感知性能
5. 实现数据缓存策略
```

#### Cursor 性能分析 Prompt

```markdown
# 性能分析请求

请分析 src/features/dashboard 模块的性能问题：

1. 首先运行性能分析
   - 使用 React DevTools Profiler
   - 识别慢渲染组件
   - 找出不必要的重渲染

2. 分析代码问题
   - 检查 useEffect 依赖数组
   - 检查 useCallback/useMemo 使用
   - 检查列表渲染优化
   - 检查数据获取模式

3. 提供优化建议
   - 针对每个问题给出具体修复方案
   - 说明优化前后的预期效果
   - 提供实现代码

4. 实施优化
   - 按照优先级实施修复
   - 验证优化效果
```

#### 优化后的代码示例

```typescript
// src/features/dashboard/components/DashboardPage.tsx
import React, { useMemo, useCallback, useTransition } from 'react';
import { ErrorBoundary } from '@/components/ErrorBoundary';
import { DashboardHeader } from './DashboardHeader';
import { StatsCard } from './StatsCard';
import { RecentOrders } from './RecentOrders';
import { SalesChart } from './SalesChart';
import { TopProducts } from './TopProducts';
import { LoadingSkeleton } from './LoadingSkeleton';
import { useDashboardData } from '@/hooks/useDashboardData';
import { useMediaQuery } from '@/hooks/useMediaQuery';

export const DashboardPage: React.FC = () => {
  const [isPending, startTransition] = useTransition();
  const isMobile = useMediaQuery('(max-width: 768px)');
  
  // 使用 React Query 获取数据，支持缓存
  const {
    data: dashboardData,
    isLoading,
    error,
    refetch,
  } = useDashboardData({
    staleTime: 5 * 60 * 1000, // 5分钟缓存
    cacheTime: 30 * 60 * 1000, // 30分钟垃圾回收
  });

  // 使用 useMemo 缓存计算结果
  const stats = useMemo(() => {
    if (!dashboardData) return null;
    
    return {
      totalRevenue: calculateTotalRevenue(dashboardData.orders),
      totalOrders: dashboardData.orders.length,
      averageOrderValue: calculateAverageOrderValue(dashboardData.orders),
      revenueGrowth: calculateGrowthRate(dashboardData.revenueHistory),
    };
  }, [dashboardData]);

  // 使用 useCallback 缓存回调函数
  const handleRefresh = useCallback(() => {
    startTransition(() => {
      refetch();
    });
  }, [refetch]);

  const handleDateRangeChange = useCallback((range: DateRange) => {
    startTransition(() => {
      // 更新日期范围，数据会在后台重新获取
      updateDateRange(range);
    });
  }, []);

  // 渲染加载状态
  if (isLoading) {
    return <LoadingSkeleton />;
  }

  // 渲染错误状态
  if (error) {
    return (
      <ErrorBoundary>
        <div className="p-8 text-center">
          <h2 className="text-xl font-semibold text-red-600 mb-2">
            加载数据失败
          </h2>
          <p className="text-gray-600 mb-4">
            {error.message}
          </p>
          <button
            onClick={handleRefresh}
            className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            重试
          </button>
        </div>
      </ErrorBoundary>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100">
      <DashboardHeader
        onRefresh={handleRefresh}
        onDateRangeChange={handleDateRangeChange}
        isRefreshing={isPending}
      />

      <main className="container mx-auto px-4 py-8">
        {/* 统计卡片 - 使用 grid 布局，响应式 */}
        <div className={`grid gap-4 ${
          isMobile ? 'grid-cols-1' : 'grid-cols-2 lg:grid-cols-4'
        }`}>
          <StatsCard
            title="总收入"
            value={formatCurrency(stats?.totalRevenue || 0)}
            trend={stats?.revenueGrowth}
          />
          <StatsCard
            title="订单数"
            value={stats?.totalOrders || 0}
            trend={5}
          />
          <StatsCard
            title="平均订单价值"
            value={formatCurrency(stats?.averageOrderValue || 0)}
          />
          <StatsCard
            title="转化率"
            value="3.2%"
            trend={-2}
          />
        </div>

        {/* 图表区域 */}
        <div className={`grid gap-6 mt-8 ${
          isMobile ? 'grid-cols-1' : 'grid-cols-1 lg:grid-cols-2'
        }`}>
          <SalesChart
            data={dashboardData?.revenueHistory || []}
            isPending={isPending}
          />
          <TopProducts
            products={dashboardData?.topProducts || []}
            isPending={isPending}
          />
        </div>

        {/* 最近订单 - 使用虚拟滚动 */}
        <div className="mt-8">
          <RecentOrders
            orders={dashboardData?.orders || []}
            isPending={isPending}
          />
        </div>
      </main>
    </div>
  );
};
```

```typescript
// src/features/dashboard/components/RecentOrders.tsx
import React, { useMemo, useCallback } from 'react';
import { FixedSizeList as List } from 'react-window'; // 虚拟滚动
import { useVirtualizer } from '@tanstack/react-virtual';

interface Order {
  id: string;
  customer: string;
  amount: number;
  status: 'pending' | 'processing' | 'completed' | 'cancelled';
  createdAt: Date;
}

interface RecentOrdersProps {
  orders: Order[];
  isPending?: boolean;
}

const ROW_HEIGHT = 60;

export const RecentOrders: React.FC<RecentOrdersProps> = ({
  orders,
  isPending,
}) => {
  // 使用 useMemo 缓存排序后的订单
  const sortedOrders = useMemo(() => {
    return [...orders].sort(
      (a, b) => new Date(b.createdAt).getTime() - new Date(a.createdAt).getTime()
    );
  }, [orders]);

  // 虚拟滚动行渲染
  const Row = useCallback(
    ({ index, style }: { index: number; style: React.CSSProperties }) => {
      const order = sortedOrders[index];
      
      return (
        <div
          style={style}
          className="flex items-center px-4 border-b hover:bg-gray-50"
        >
          <div className="w-1/4">{order.id}</div>
          <div className="w-1/4">{order.customer}</div>
          <div className="w-1/4">
            ${order.amount.toLocaleString()}
          </div>
          <div className="w-1/4">
            <span
              className={`px-2 py-1 rounded text-xs ${
                order.status === 'completed'
                  ? 'bg-green-100 text-green-800'
                  : order.status === 'pending'
                  ? 'bg-yellow-100 text-yellow-800'
                  : order.status === 'processing'
                  ? 'bg-blue-100 text-blue-800'
                  : 'bg-red-100 text-red-800'
              }`}
            >
              {order.status}
            </span>
          </div>
        </div>
      );
    },
    [sortedOrders]
  );

  // 使用 useMemo 缓存列表高度
  const listHeight = useMemo(() => {
    const maxVisibleRows = 10;
    return Math.min(sortedOrders.length, maxVisibleRows) * ROW_HEIGHT;
  }, [sortedOrders.length]);

  return (
    <div className="bg-white rounded-lg shadow overflow-hidden">
      <div className="px-6 py-4 border-b">
        <h3 className="text-lg font-semibold">最近订单</h3>
      </div>

      {/* 表头 */}
      <div className="flex items-center px-4 py-3 bg-gray-50 border-b font-medium text-sm text-gray-600">
        <div className="w-1/4">订单号</div>
        <div className="w-1/4">客户</div>
        <div className="w-1/4">金额</div>
        <div className="w-1/4">状态</div>
      </div>

      {/* 虚拟滚动列表 */}
      {sortedOrders.length > 0 ? (
        <List
          height={listHeight}
          itemCount={sortedOrders.length}
          itemSize={ROW_HEIGHT}
          width="100%"
        >
          {Row}
        </List>
      ) : (
        <div className="p-8 text-center text-gray-500">
          暂无订单数据
        </div>
      )}

      {/* 加载状态 */}
      {isPending && (
        <div className="absolute inset-0 bg-white/50 flex items-center justify-center">
          <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-600" />
        </div>
      )}
    </div>
  );
};
```

---

## Cursor 常见问题与解决方案

### FAQ 1：Cursor 不生成代码补全建议

**症状**：输入代码时没有显示任何 AI 补全建议。

**诊断步骤**：
```markdown
1. 检查 Cursor AI 是否启用
   - 设置 → Cursor → AI → Enable Inline Completions

2. 检查当前文件语言是否支持
   - 设置 → Cursor → AI → Enabled Languages
   - 确认当前语言已启用

3. 检查网络连接
   - 尝试访问 cursor.sh
   - 检查代理设置

4. 检查 API 配额
   - 设置 → Account → Usage
   - 确认还有可用额度
```

**解决方案**：
```json
// .vscode/settings.json
{
  "cursor.completion": {
    "inline": true,
    "ghostText": true,
    "enabledLanguages": ["*"],
    "disableLanguageOverrides": false
  }
}
```

### FAQ 2：Agent Mode 执行失败

**症状**：Agent Mode 无法执行操作，提示权限错误。

**解决方案**：
```markdown
1. 检查安全设置
   - 设置 → Cursor → Agent → Auto Approve

2. 确认文件权限
   - 检查项目目录权限
   - 确认有写入权限

3. 禁用冲突扩展
   - 某些扩展可能干扰 Agent

4. 重置 Agent 状态
   - 设置 → Cursor → Reset Agent Cache
```

### FAQ 3：上下文理解不准确

**症状**：AI 生成的代码与项目规范不符。

**解决方案**：
```markdown
1. 完善 Cursor Rules
   - 添加更详细的代码规范
   - 包含示例代码

2. 引用相关文件
   - 使用 @file: 引用相关代码
   - 使用 @folder: 引用整个目录

3. 提供更多上下文
   - 描述项目的技术栈
   - 说明现有的代码模式

4. 重置项目索引
   - 设置 → Cursor → Reset Project Index
```

### FAQ 4：Cursor 占用内存过高

**症状**：Cursor 运行缓慢，系统内存占用过高。

**解决方案**：
```markdown
1. 减小上下文窗口
   - 设置 → Cursor → AI → Context Window Size
   - 从 "Full Project" 改为 "Open Files"

2. 限制索引范围
   - 设置 → Cursor → Index → Include Patterns
   - 只索引 src/ 目录

3. 禁用不必要的功能
   - 关闭 hover descriptions
   - 关闭 inline suggestions

4. 增加内存限制
   - Cursor → 设置 → 内存限制
```

### FAQ 5：订阅显示未激活

**症状**：付费订阅已购买但功能仍然受限。

**解决方案**：
```markdown
1. 退出并重新登录
   - Cursor → 账户 → Sign Out
   - 重新登录

2. 清除缓存
   - macOS: Cmd+Shift+P → "Clear Cache"
   - Windows: Ctrl+Shift+P → "Clear Cache"

3. 检查订阅状态
   - https://cursor.sh/settings/account
   - 确认订阅状态为 Active

4. 联系客服
   - support@cursor.sh
   - 提供订阅确认邮件
```

### FAQ 6：与 VS Code 扩展冲突

**症状**：安装某些 VS Code 扩展后 Cursor 异常。

**解决方案**：
```markdown
1. 识别问题扩展
   - 禁用所有扩展
   - 逐个启用找出冲突

2. 查找替代扩展
   - 搜索 Cursor 兼容版本
   - 使用替代方案

3. 更新扩展
   - 确保扩展是最新版本
   - 更新 Cursor 到最新版本

4. 报告问题
   - GitHub Issues
   - 提供扩展版本信息
```

### FAQ 7：代码补全响应缓慢

**症状**：输入代码后补全建议延迟严重。

**诊断与解决**：
```markdown
1. 检查网络延迟
   - 使用 ping cursor.sh 测试连接
   - 考虑使用代理服务器

2. 切换模型
   - 设置 → Cursor → AI → Model
   - 从 Claude Opus 切换到 Sonnet

3. 减少上下文
   - 关闭不必要的标签页
   - 限制索引文件范围

4. 检查配额
   - Free 用户可能有速率限制
   - 考虑升级到 Pro
```

### FAQ 8：Git 操作失败

**症状**：Agent Mode 执行 Git 操作时报错。

**解决方案**：
```markdown
1. 验证 Git 配置
   git config --global user.name "Your Name"
   git config --global user.email "your@email.com"

2. 检查 SSH 密钥
   ls -la ~/.ssh/
   cat ~/.ssh/id_rsa.pub  # 确认公钥已添加到 GitHub

3. 验证仓库权限
   - 确认有仓库写权限
   - 检查 token 有效期

4. 使用终端调试
   - 手动执行 Git 命令
   - 查看具体错误信息
```

### FAQ 9：自定义快捷键不生效

**症状**：配置的自定义快捷键无法使用。

**解决方案**：
```markdown
1. 检查快捷键冲突
   - Cmd+Shift+P → "Show Shortcut References"
   - 查找冲突的快捷键

2. 验证 JSON 语法
   // 正确的 key 格式：
   "key": "cmd+shift+h"  // macOS
   "key": "ctrl+shift+h" // Windows/Linux

3. 确认 when 条件
   - 检查上下文条件是否满足
   - 使用 "editorTextFocus" 等条件

4. 重启 Cursor
   - 完全退出后重新启动
   - 清除缓存后重启
```

### FAQ 10：Cursor Rules 不生效

**症状**：配置的 Cursor Rules 没有被应用。

**解决方案**：
```markdown
1. 确认文件位置
   - 规则文件必须在 .cursor/rules/ 目录
   - 文件扩展名必须是 .mdc

2. 检查文件格式
   # 文件开头必须有 cursorules 标记
   # cursorules
   
   ## 规则内容...

3. 验证规则加载
   - Cmd+Shift+P → "Cursor: Show Rules"
   - 查看已加载的规则列表

4. 清除并重建索引
   - 设置 → Cursor → Reset Project Index
   - 重启 Cursor
```

### FAQ 11：多文件编辑结果不符合预期

**症状**：Agent Mode 修改了错误的文件或内容。

**解决方案**：
```markdown
1. 限制修改范围
   - 在任务描述中明确指定文件
   - 使用 "仅修改 src/auth/*.ts"

2. 分步骤执行
   - 先要求分析涉及的文件
   - 确认后再执行修改

3. 使用 Composer 模式
   - 手动选择需要修改的文件
   - 提供每个文件的修改指令

4. 启用确认模式
   - 设置 → Cursor → Agent → Auto Approve: Never
   - 每次操作前确认
```

### FAQ 12：代码生成包含过时 API

**症状**：AI 生成的代码使用了已弃用的 API。

**解决方案**：
```markdown
1. 提供技术栈版本
   "项目使用 React 18.2，Next.js 14.2"
   
2. 指定 API 版本
   "使用 React Query v5，不要使用 v4"

3. 添加约束
   "禁止使用已弃用的 API，参考官方文档"

4. 验证生成结果
   - 使用 ESLint 检查弃用警告
   - 手动审查 AI 生成的代码
```

---

## Cursor 与现代技术栈深度集成

### Next.js 15 + React 19 集成

#### 项目初始化

```markdown
# Cursor AI 辅助初始化 Next.js 项目

提示词：
"创建一个 Next.js 15 项目，要求：
- 使用 App Router
- TypeScript
- Tailwind CSS
- Prisma ORM + PostgreSQL
- NextAuth.js v5 认证
- React Query 管理客户端状态
- Zod 表单验证

配置：
- ESLint + Prettier
- Husky + lint-staged
- Jest + React Testing Library

目录结构：
- 使用 feature-based 结构
- API 路由在 app/api/
- 组件在 components/
- 类型在 types/"
```

#### Cursor 生成的代码示例

**src/app/users/page.tsx**

```typescript
import { Suspense } from 'react';
import { UserList } from '@/components/features/UserList';
import { UserListSkeleton } from '@/components/ui/skeletons/UserListSkeleton';
import { getUsers } from '@/services/userService';

interface UsersPageProps {
  searchParams: Promise<{ page?: string; limit?: string }>;
}

export default async function UsersPage({ searchParams }: UsersPageProps) {
  const params = await searchParams;
  const page = parseInt(params.page || '1', 10);
  const limit = parseInt(params.limit || '20', 10);

  const { users, total } = await getUsers({ page, limit });

  return (
    <div className="container mx-auto py-8">
      <div className="mb-8">
        <h1 className="text-3xl font-bold">用户列表</h1>
        <p className="text-muted-foreground mt-2">
          共 {total} 位用户
        </p>
      </div>

      <Suspense fallback={<UserListSkeleton count={limit} />}>
        <UserList users={users} total={total} page={page} limit={limit} />
      </Suspense>
    </div>
  );
}

export const metadata = {
  title: '用户列表 - My App',
  description: '浏览和管理所有用户',
};
```

**src/services/userService.ts**

```typescript
import { prisma } from '@/lib/prisma';
import { cache } from 'react';

export interface GetUsersParams {
  page?: number;
  limit?: number;
  search?: string;
  role?: 'USER' | 'ADMIN';
}

export interface GetUsersResult {
  users: User[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

export const getUsers = cache(async (params: GetUsersParams): Promise<GetUsersResult> => {
  const { page = 1, limit = 20, search, role } = params;
  const skip = (page - 1) * limit;

  const where = {
    ...(search && {
      OR: [
        { name: { contains: search, mode: 'insensitive' as const } },
        { email: { contains: search, mode: 'insensitive' as const } },
      ],
    }),
    ...(role && { role }),
  };

  const [users, total] = await Promise.all([
    prisma.user.findMany({
      where,
      skip,
      take: limit,
      orderBy: { createdAt: 'desc' },
      select: {
        id: true,
        name: true,
        email: true,
        avatar: true,
        role: true,
        createdAt: true,
        _count: { select: { posts: true } },
      },
    }),
    prisma.user.count({ where }),
  ]);

  return {
    users: users as User[],
    total,
    page,
    limit,
    totalPages: Math.ceil(total / limit),
  };
});
```

### React Native + Expo 集成

#### Cursor 辅助开发

```markdown
# Cursor AI 辅助 React Native 开发

提示词：
"使用 Expo + React Native 创建一个待办事项应用，要求：
- Expo SDK 52
- TypeScript
- Expo Router 文件路由
- Zustand 状态管理
- AsyncStorage 持久化
- 支持深色模式
- 支持 iOS/Android 双平台

功能：
1. 创建/编辑/删除待办
2. 待办分组
3. 截止日期提醒
4. 本地推送通知
5. 数据导出为 JSON"
```

#### 生成的代码示例

**app/_layout.tsx**

```typescript
import { DarkTheme, LightTheme } from '@/theme';
import { useColorScheme } from '@/hooks/useColorScheme';
import { StatusBar } from 'expo-status-bar';
import { Stack } from 'expo-router';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';

export default function RootLayout() {
  const colorScheme = useColorScheme();

  return (
    <SafeAreaProvider>
      <GestureHandlerRootView style={{ flex: 1 }}>
        <Stack
          screenOptions={{
            headerStyle: {
              backgroundColor: colorScheme === 'dark' 
                ? DarkTheme.colors.background 
                : LightTheme.colors.background,
            },
            headerTintColor: colorScheme === 'dark'
              ? DarkTheme.colors.text
              : LightTheme.colors.text,
          }}
        >
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="+not-found" options={{ title: 'Oops!' }} />
        </Stack>
        <StatusBar style={colorScheme === 'dark' ? 'light' : 'dark'} />
      </GestureHandlerRootView>
    </SafeAreaProvider>
  );
}
```

**app/(tabs)/index.tsx**

```typescript
import { useState } from 'react';
import { View, FlatList, TouchableOpacity, StyleSheet } from 'react-native';
import { useRouter } from 'expo-router';
import { TextInput } from '@/components/ui/TextInput';
import { TodoItem } from '@/components/features/TodoItem';
import { useTodos } from '@/stores/todoStore';
import { useTheme } from '@/hooks/useTheme';
import { Todo } from '@/types';

export default function TodosScreen() {
  const router = useRouter();
  const { theme } = useTheme();
  const [newTodoTitle, setNewTodoTitle] = useState('');
  const { todos, addTodo, toggleTodo, deleteTodo } = useTodos();

  const handleAddTodo = async () => {
    if (!newTodoTitle.trim()) return;
    
    const newTodo: Todo = {
      id: Date.now().toString(),
      title: newTodoTitle.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
      priority: 'medium',
    };
    
    addTodo(newTodo);
    setNewTodoTitle('');
  };

  const renderItem = ({ item }: { item: Todo }) => (
    <TodoItem
      todo={item}
      onToggle={() => toggleTodo(item.id)}
      onDelete={() => deleteTodo(item.id)}
      onPress={() => router.push(`/todo/${item.id}`)}
    />
  );

  return (
    <View style={[styles.container, { backgroundColor: theme.colors.background }]}>
      <View style={styles.inputContainer}>
        <TextInput
          value={newTodoTitle}
          onChangeText={setNewTodoTitle}
          placeholder="添加新待办..."
          onSubmitEditing={handleAddTodo}
          returnKeyType="done"
        />
        <TouchableOpacity
          style={[styles.addButton, { backgroundColor: theme.colors.primary }]}
          onPress={handleAddTodo}
        >
          <Text style={styles.addButtonText}>添加</Text>
        </TouchableOpacity>
      </View>

      <FlatList
        data={todos}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.list}
        showsVerticalScrollIndicator={false}
      />
    </View>
  );
}
```

### 数据库设计集成

#### Prisma Schema 最佳实践

```prisma
// prisma/schema.prisma
// Cursor AI 辅助生成的完整数据模型

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// 用户模型
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  avatar        String?
  passwordHash   String
  role          Role      @default(USER)
  emailVerified DateTime?
  
  // 关系
  accounts      Account[]
  sessions      Session[]
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  
  // 时间戳
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?

  @@index([email])
  @@index([role])
  @@map("users")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

// OAuth 账户
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refreshToken      String? @db.Text
  accessToken      String? @db.Text
  expiresAt        Int?
  tokenType        String?
  scope            String?
  idToken          String? @db.Text
  sessionState     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

// 会话
model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

// 帖子
model Post {
  id         String   @id @default(cuid())
  title      String   @db.VarChar(255)
  slug       String   @unique
  content    String   @db.Text
  excerpt    String?  @db.VarChar(500)
  coverImage String?
  published  Boolean  @default(false)
  publishedAt DateTime?
  
  // 关系
  author     User     @relation(fields: [authorId], references: [id])
  authorId   String
  comments   Comment[]
  tags       Tag[]
  likes      Like[]

  // SEO
  metaTitle       String? @db.VarChar(70)
  metaDescription String? @db.VarChar(160)

  // 时间戳
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([authorId])
  @@index([slug])
  @@index([published])
  @@map("posts")
}

// 评论
model Comment {
  id        String   @id @default(cuid())
  content   String   @db.Text
  approved  Boolean  @default(true)

  // 关系
  author    User?    @relation(fields: [authorId], references: [id])
  authorId  String?
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  parent    Comment? @relation("CommentReplies", fields: [parentId], references: [id])
  parentId  String?
  replies   Comment[] @relation("CommentReplies")

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([parentId])
  @@map("comments")
}

// 标签
model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  posts Post[]

  @@map("tags")
}

// 点赞
model Like {
  id        String   @id @default(cuid())
  user      User     @relation(fields: [userId], references: [id])
  userId    String
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    String
  createdAt DateTime @default(now())

  @@unique([userId, postId])
  @@map("likes")
}
```

---

## Cursor 与竞品深度对比分析

### 技术架构对比

```
┌─────────────────────────────────────────────────────────────────┐
│                        架构对比图示                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Cursor                    VS Code + Copilot        Windsurf      │
│  ┌─────────┐              ┌─────────────┐         ┌─────────┐ │
│  │ 自研 IDE │              │ VS Code 插件 │         │ VS Code │ │
│  │ + AI 深度 │              │ + 云端 AI    │         │ + Cascade │ │
│  │ 集成     │              └─────────────┘         └─────────┘ │
│  └─────────┘                                               │
│       │                                                    │
│       ▼                                                    │
│  ┌─────────────────────────────────────────┐              │
│  │           AI Integration Layer           │              │
│  │  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐   │              │
│  │  │Claude│ │ GPT │ │Gemini│ │ 自研 │   │              │
│  │  └─────┘ └─────┘ └─────┘ └─────┘   │              │
│  └─────────────────────────────────────────┘              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 功能详细对比

| 功能维度 | Cursor | Copilot | Windsurf | Cline | Roo Code |
|---------|--------|---------|-----------|-------|----------|
| **AI 集成深度** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **代码补全质量** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| **多文件编辑** | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **上下文理解** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **价格** | $20/月 | $10/月 | $10/月 | 免费 | 免费 |
| **本地模型** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **开源** | ❌ | ❌ | ❌ | ✅ | ✅ |
| **自定义规则** | ✅ Cursor Rules | ✅ Custom Instructions | ✅ Rules | ✅ | ✅ |
| **Tab 补全** | ✅ | ✅ | ✅ | ❌ | ❌ |

### 场景化推荐

| 使用场景 | 推荐工具 | 原因 |
|----------|---------|------|
| **快速原型开发** | Cursor | Agent Mode 强大，多文件编辑高效 |
| **日常代码补全** | Copilot / Windsurf | 实时补全，开箱即用 |
| **隐私敏感项目** | Cline / Roo Code + Ollama | 完全本地，数据不出境 |
| **预算有限** | Windsurf / Cline | 功能完整，免费使用 |
| **企业团队** | Copilot Business / Cursor Business | 策略管理完善 |
| **GitHub 重度用户** | Copilot | PR 集成最佳 |
| **VS Code 老用户** | Windsurf / Copilot | 无缝迁移 |

### 成本效益分析

```markdown
# 年度使用成本对比

| 工具 | 月费 | 年费 | 边际成本 |
|------|------|------|----------|
| Cursor Pro | $20 | $192 | 无 |
| Copilot Pro | $10 | $100 | 无 |
| Windsurf Pro | $10 | $96 | 无 |
| Cline | $0 | $0 | API 费用 |
| Roo Code | $0 | $0 | API 费用 |

# API 成本估算（Cline/Roo Code）

| 模型 | 场景 | 月均成本 |
|------|------|----------|
| Claude Sonnet | 日常开发 | $20-50 |
| GPT-4o | 日常开发 | $15-40 |
| Ollama 本地 | 完全免费 | $0 |

结论：
- 个人开发者：Windsurf Pro 性价比最高
- 企业用户：Copilot Business 策略管理最佳
- 预算有限：Cline + Claude API 最经济
- 隐私优先：Roo Code + Ollama 零成本本地
```

---

## Cursor 选型建议与最佳实践

### 个人开发者

**推荐：Cursor Pro 或 Windsurf Pro**

```markdown
# 选择理由

Cursor Pro：
- 最佳的 AI 集成体验
- 强大的 Agent Mode
- 优秀的代码补全
- 适合追求效率的开发者

Windsurf Pro：
- 价格更低（$10/月 vs $20/月）
- Supercomplete 技术创新
- VS Code 兼容性更好
- 适合预算有限的开发者
```

### 初创团队

**推荐：Cursor Business 或 Copilot Business**

```markdown
# 选择理由

团队需求：
1. 共享代码规范
2. 使用分析
3. 权限管理
4. 成本控制

Cursor Business ($40/用户/月)：
- 团队 Cursor Rules
- 使用分析仪表板
- SSO 支持（即将推出）

Copilot Business ($19/用户/月)：
- 更低的价格
- 成熟的策略管理
- GitHub 深度集成
```

### 大型企业

**推荐：Cursor Enterprise + 自定义配置**

```markdown
# 企业需求分析

1. 合规要求
   - SOC 2 / GDPR
   - 数据本地化
   - 审计日志

2. 安全要求
   - SSO/SAML
   - IP 白名单
   - 代码隔离

3. 成本控制
   - 部门预算分配
   - 使用量监控
   - 成本上限

Cursor Enterprise：
- 自定义 SLA
- 专属客户经理
- 高级安全合规
```

### 混合使用策略

```markdown
# 最优工具组合

日常补全：GitHub Copilot (VS Code 内)
- 实时补全
- 轻量快速

复杂任务：Cursor
- 多文件编辑
- Agent Mode

本地开发：Cline + Ollama
- 完全离线
- 零成本

代码审查：Claude Code
- 最强分析能力
- 深度理解
```

---

## Cursor 生态系统

### 推荐的 Cursor 扩展

| 扩展名称 | 功能 | 优先级 |
|---------|------|--------|
| Prettier | 代码格式化 | 必需 |
| ESLint | 代码检查 | 必需 |
| GitLens | Git 可视化 | 推荐 |
| Error Lens | 错误高亮 | 推荐 |
| Thunder Client | API 测试 | 推荐 |
| Auto Rename Tag | HTML 标签同步 | 可选 |
| GitHub Copilot | 辅助补全 | 可选 |

### 主题推荐

```json
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
        },
        {
          "scope": "keyword",
          "settings": {
            "foreground": "#c678dd"
          }
        },
        {
          "scope": "string",
          "settings": {
            "foreground": "#98c379"
          }
        }
      ]
    }
  }
}
```

### 字体配置

```json
{
  "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
  "editor.fontSize": 14,
  "editor.lineHeight": 1.6,
  "editor.letterSpacing": 0.5,
  "editor.fontLigatures": true,
  "editor.cursorBlinking": "smooth",
  "editor.cursorSmoothCaretAnimation": "on"
}
```

---

**文档统计**：
- 撰写时间：2026年4月
- 最终行数：约 5200 行
- 代码示例：150+
- 快捷键表格：30+
- 功能详解：50+
- FAQ 方案：12+
- 实战场景：3 个完整案例

> [!SUCCESS]
> Cursor 作为 AI 原生 IDE 的代表产品，在 vibecoding 场景中展现出卓越的生产力提升能力。其深度 AI 集成、智能 Agent 模式和灵活的 Cursor Rules 配置，使其成为 2026 年 AI 编程工具的首选。建议开发者从免费版开始体验，逐步探索高级功能，最终根据团队需求选择合适的订阅计划。
