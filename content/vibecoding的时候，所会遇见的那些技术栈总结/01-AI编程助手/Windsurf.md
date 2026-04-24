# Windsurf - Codeium 出品 AI 编程助手

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Windsurf 的核心功能、Cascade 架构、与竞品对比及实战技巧。

---

## 目录

1. [[#Windsurf 概述与市场定位]]
2. [[#Cascade 智能体架构]]
3. [[#Supercomplete 技术]]
4. [[#核心功能详解]]
5. [[#价格体系]]
6. [[#Windsurf Rules 与配置]]
7. [[#Agentic Workflow 深度使用]]
8. [[#多文件协作模式]]
9. [[#与其他工具对比]]
10. [[#局限性分析]]
11. [[#选型建议]]
12. [[#进阶技巧与最佳实践]]
13. [[#企业级配置]]
14. [[#常见问题与故障排除]]
15. [[#参考资料]]

---

## Windsurf 概述与市场定位

### 产品背景

Windsurf 是由 **Codeium** 公司于 2024 年推出的 AI 编程助手。Codeium 成立于 2022 年，是一家专注于 AI 代码生成的初创公司，其核心产品 Codeium 是一个免费的 AI 代码补全工具，已被超过 100 万开发者使用。

Windsurf 的推出标志着 Codeium 从单一的代码补全工具向完整的 AI 编程助手转型。与 Cursor 类似，Windsurf 也采用了自研 IDE 的路线，但更加强调**工作流自动化**和**智能体协作**。

### 核心定位

Windsurf 的差异化定位是**"AI 协作而非 AI 主导"**，强调：

- **人类在环**：AI 是协作者而非替代者
- **工作流优化**：自动化重复性高的开发任务
- **渐进式 AI**：根据任务复杂度选择合适的 AI 介入程度
- **开放生态**：不锁定用户，支持多种模型和工具

> [!TIP]
> Windsurf 的设计理念是"AI 增强开发者"而非"AI 替代开发者"，更适合希望保持对代码控制权的开发者。

### 技术架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Windsurf IDE                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │   Editor    │  │  Cascade    │  │   Terminal   │       │
│  │  (Monaco)   │  │   Agent     │  │   (集成)    │       │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘       │
│         │                │                │                  │
│  ┌──────┴────────────────┴────────────────┴──────┐       │
│  │              Cascade Engine                           │       │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐           │       │
│  │  │ 上下文   │ │  任务    │ │  执行    │           │       │
│  │  │ 管理器   │ │  规划器  │ │  引擎    │           │       │
│  │  └──────────┘ └──────────┘ └──────────┘           │       │
│  └──────────────────────────────────────────────────┘       │
│  ┌──────────────────────────────────────────────────┐       │
│  │           Multi-Model Layer                          │       │
│  │  Codeium (自有模型) + Claude + GPT + Gemini        │       │
│  └──────────────────────────────────────────────────┘       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 版本演进

| 时间 | 版本 | 重大更新 |
|------|------|----------|
| 2024.03 | Windsurf v1 | 首次发布，基于 VS Code 分支 |
| 2024.06 | Cascade 发布 | 引入 Cascade AI Agent |
| 2024.09 | Supercomplete | 推出超级代码补全功能 |
| 2025.02 | Cascade v2 | 增强的上下文理解能力 |
| 2025.08 | Flow Architect | 项目级任务规划 |
| 2026.01 | Cascade v3 | 支持多 Agent 协作 |
| 2026.03 | Flow Library | 社区 Flow 分享平台 |

### 为什么选择 Windsurf

#### 1. 独特的 AI 协作理念

Windsurf 强调"人类在环"的设计哲学：

```markdown
# 人类在环设计

┌─────────────────────────────────────────────────────────────┐
│                    任务复杂度 → AI 介入程度                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  简单任务          中等任务          复杂任务          极复杂任务  │
│  ┌──────┐        ┌──────┐        ┌──────┐        ┌──────┐  │
│  │ AI   │        │ AI   │        │ 协作  │        │ 监督  │  │
│  │ 主导 │        │ 主导 │        │ 模式  │        │ 模式  │  │
│  │      │        │ 用户 │        │ 用户  │        │ 用户  │  │
│  │ 建议  │        │ 确认 │        │ 指导  │        │ 审核  │  │
│  └──────┘        └──────┘        └──────┘        └──────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### 2. VS Code 兼容优势

```markdown
# 迁移优势

从 VS Code 迁移到 Windsurf：

✅ 完全兼容：
├─ VS Code 扩展（95%+）
├─ 快捷键配置
├─ 主题设置
├─ 工作区配置
└─ 用户设置（JSON 格式）

🔄 需要适应：
├─ AI 相关快捷键
├─ Cascade 对话界面
└─ Flow 功能

❌ 不支持：
├─ VS Code Live Share
└─ 某些实验性功能
```

---

## Cascade 智能体架构

### 概述

Cascade 是 Windsurf 的核心 AI 引擎，采用**混合智能体架构**，结合了规则引擎、机器学习和用户意图理解的多种技术。

### 核心组件

#### 1. 上下文管理器（Context Manager）

负责收集和维护任务相关的上下文信息：

| 上下文类型 | 来源 | 优先级 |
|-----------|------|-------|
| 当前文件 | 编辑器光标位置 | 高 |
| 打开的文件 | IDE 文件标签 | 高 |
| 项目结构 | 目录树和导入关系 | 中 |
| Git 历史 | 最近修改的文件 | 中 |
| 相关文档 | README、API 文档 | 低 |
| 测试文件 | 对应的测试文件 | 中 |

#### 2. 任务规划器（Task Planner）

将复杂任务拆解为可执行的子任务：

```yaml
# 任务规划示例
input: "重构用户认证模块"
planning:
  - step: 1
    action: analyze
    target: "理解当前认证实现"
    tools:
      - read: "src/auth/*.ts"
      - grep: "认证相关函数"
    
  - step: 2
    action: identify
    target: "识别依赖关系"
    tools:
      - grep: "import.*auth"
      - read: "package.json"
    
  - step: 3
    action: design
    target: "设计新的架构"
    output: "architecture.md"
    
  - step: 4
    action: implement
    subtasks:
      - "创建新的认证服务"
      - "更新用户模型"
      - "修改路由保护"
      - "更新测试"
      
  - step: 5
    action: test
    target: "验证重构结果"
    tools:
      - terminal: "npm test"
      - terminal: "npm run build"
```

#### 3. 执行引擎（Execution Engine）

负责任务的实际执行：

| 执行模式 | 说明 | 适用场景 |
|---------|------|---------|
| **立即执行** | AI 立即执行操作 | 简单明确的任务 |
| **协作模式** | AI 提供建议，用户确认 | 中等复杂度 |
| **审查模式** | AI 分析，用户执行 | 复杂或敏感任务 |

### Cascade 的独特能力

#### 1. 上下文窗口扩展

| 能力 | 说明 |
|------|------|
| 全项目索引 | 建立完整的代码库索引 |
| 语义搜索 | 理解代码语义而非关键词 |
| 跨文件推理 | 追踪文件间的调用关系 |
| 历史上下文 | 理解 Git 变更历史 |

```typescript
// 上下文理解示例
// 当用户询问 "如何添加新的认证提供者" 时
// Cascade 会理解：

1. 现有认证结构
   ├─ src/auth/
   │  ├─ providers/
   │  │  ├─ local.ts
   │  │  ├─ google.ts
   │  │  └─ github.ts
   │  └─ index.ts

2. 现有提供者模式
   └─ interface AuthProvider {
        authenticate(): Promise<User>;
        getUserInfo(): Promise<UserInfo>;
     }

3. 集成点
   └─ src/auth/index.ts 中的工厂函数

4. 测试模式
   └─ tests/auth/*.test.ts
```

#### 2. 多轮对话

Cascade 支持复杂的多轮对话场景：

```markdown
用户: 帮我重构这个函数
Cascade: 已分析函数 [func_name]，识别到以下问题：
         1. 函数过长 (150行)
         2. 嵌套过深 (5层)
         3. 缺少类型注解
         
         是否继续重构？

用户: 是，但保持向后兼容
Cascade: 了解，将在重构时：
         1. 创建新函数 [new_func_name]
         2. 保留原函数作为包装器
         3. 添加弃用警告
         
         准备开始，请确认...

用户: 开始执行
Cascade: ## 执行进度
         [1/5] 创建新函数...
         [2/5] 提取辅助函数...
         [3/5] 更新类型定义...
         [4/5] 创建包装器...
         [5/5] 验证兼容性...
         
         ✅ 重构完成！
```

#### 3. 主动建议

Cascade 不仅响应用户请求，还会主动提供建议：

| 建议类型 | 触发条件 |
|---------|---------|
| 代码改进 | 检测到代码异味 |
| 安全警告 | 发现潜在安全漏洞 |
| 性能优化 | 识别性能瓶颈 |
| 测试覆盖 | 函数缺少测试 |
| 文档缺失 | 重要函数无文档 |
| 依赖更新 | 检测到过时依赖 |

---

## Supercomplete 技术

### 概述

Supercomplete 是 Windsurf 独有的代码补全技术，区别于传统的单行或片段补全，Supercomplete 尝试一次性生成完整的代码块。

### 技术原理

```
传统补全:
┌────────────────────┐
│ function add(a, b) │ ← 用户输入
│ █                  │ ← 补全 1 行
└────────────────────┘

Supercomplete:
┌────────────────────────────────────┐
│ function add(a, b) {               │ ← 用户输入
│   if (typeof a !== 'number'        │
│       || typeof b !== 'number') {   │
│     throw new TypeError(           │ ← Supercomplete
│       'Arguments must be           │   补全整个
│        numbers'                     │   函数
│     );                             │
│   }                                │
│   return a + b;                   │
│ }                                  │
└────────────────────────────────────┘
```

### 技术对比

```markdown
# 补全技术演进

1. 单行补全（传统）
   ├─ 每次补全一行
   ├─ 需要多次 Tab
   └─ 上下文有限

2. 多行补全（Copilot）
   ├─ 一次补全多行
   ├─ 仍受限于单次调用
   └─ 需人工判断截断点

3. Supercomplete（Windsurf）
   ├─ 整函数一次性生成
   ├─ 智能边界检测
   └─ 语义级理解
```

### Supercomplete 的特点

| 特性 | 说明 |
|------|------|
| **整函数补全** | 一次性生成完整函数 |
| **类型感知** | 根据类型注解生成代码 |
| **模式学习** | 学习项目中的代码模式 |
| **智能截断** | 过长时智能选择最佳截断点 |
| **多补全选择** | 提供多个补全版本供选择 |

### 使用示例

```typescript
// 示例 1：TypeScript 接口实现补全

// 用户输入
interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(user: CreateUserDto): Promise<User>;
  update(id: string, data: UpdateUserDto): Promise<User>;
  delete(id: string): Promise<void>;
}

// 按 Tab 触发 Supercomplete
// 生成完整实现：
class PostgresUserRepository implements UserRepository {
  constructor(private db: Database) {}
  
  async findById(id: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    const result = await this.db.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
    return result.rows[0] || null;
  }
  
  async create(user: CreateUserDto): Promise<User> {
    const result = await this.db.query(
      `INSERT INTO users (name, email, password_hash) 
       VALUES ($1, $2, $3) RETURNING *`,
      [user.name, user.email, user.passwordHash]
    );
    return result.rows[0];
  }
  
  async update(id: string, data: UpdateUserDto): Promise<User> {
    const updates = Object.entries(data)
      .filter(([, v]) => v !== undefined)
      .map(([k], i) => `${k} = $${i + 2}`)
      .join(', ');
    const result = await this.db.query(
      `UPDATE users SET ${updates} WHERE id = $1 RETURNING *`,
      [id, ...Object.values(data)]
    );
    return result.rows[0];
  }
  
  async delete(id: string): Promise<void> {
    await this.db.query('DELETE FROM users WHERE id = $1', [id]);
  }
}
```

```python
# 示例 2：Python 类补全

# 用户输入
class DataValidator:
    """数据验证器，支持多种验证规则"""
    
    def __init__(self):
        self.rules = []
    
    def add_rule(self, field: str, rule: callable) -> 'DataValidator':
        """添加验证规则"""
        pass
    
    def validate(self, data: dict) -> tuple[bool, list[str]]:
        """验证数据，返回(是否通过, 错误列表)"""
        pass

# 按 Tab 触发 Supercomplete
# 生成完整实现：
    def add_rule(self, field: str, rule: callable) -> 'DataValidator':
        """添加验证规则
        
        Args:
            field: 字段名
            rule: 验证函数，应返回 (bool, str) 元组
            
        Returns:
            self，支持链式调用
        """
        self.rules.append((field, rule))
        return self
    
    def validate(self, data: dict) -> tuple[bool, list[str]]:
        """验证数据
        
        Args:
            data: 待验证的数据字典
            
        Returns:
            (is_valid, errors) - 是否通过验证和错误列表
        """
        errors = []
        
        for field, rule in self.rules:
            if field not in data:
                errors.append(f"Missing required field: {field}")
                continue
                
            is_valid, error_msg = rule(data[field])
            if not is_valid:
                errors.append(f"{field}: {error_msg}")
        
        return len(errors) == 0, errors
    
    def validate_or_raise(self, data: dict) -> None:
        """验证数据，失败则抛出异常"""
        is_valid, errors = self.validate(data)
        if not is_valid:
            raise ValidationError("; ".join(errors))
```

### 与传统补全的对比

| 维度 | 传统补全 | Supercomplete |
|------|---------|---------------|
| 补全粒度 | 单行/片段 | 整函数/模块 |
| 上下文需求 | 当前行 | 完整函数签名 |
| 生成速度 | 快 | 稍慢 |
| 准确性 | 高 | 依赖上下文质量 |
| 适用场景 | 简单代码 | 完整实现 |

---

## 核心功能详解

### 1. AI Flow（智能工作流）

Windsurf 的 AI Flow 功能将常见的开发工作流自动化：

#### Onboarding Flow

```markdown
# 功能说明
├─ 分析新项目结构
├─ 生成项目地图
└─ 推荐关键文件阅读顺序

# 示例：运行 Onboarding Flow
/wflow onboarding

# 输出：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎯 项目分析完成
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 技术栈识别：
├─ 框架：React 18 + Vite
├─ 语言：TypeScript
├─ 样式：Tailwind CSS
├─ 状态：Zustand
├─ API：React Query
└─ 测试：Vitest

📁 项目结构：
├─ src/
│  ├─ components/     # UI 组件
│  ├─ pages/          # 页面组件
│  ├─ hooks/          # 自定义 Hooks
│  ├─ stores/         # 状态管理
│  ├─ api/            # API 请求
│  └─ utils/          # 工具函数
├─ tests/
└─ docs/

📖 推荐阅读顺序：
1. README.md          # 项目概述
2. src/main.tsx       # 入口文件
3. src/App.tsx        # 根组件
4. src/api/           # API 层
5. src/stores/        # 状态管理
6. src/pages/         # 页面组件

💡 关键提示：
├─ 使用 pnpm 作为包管理器
├─ 遵循原子化 CSS 规范
└─ API 请求统一通过 useApi hook
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### Refactor Flow

```markdown
# 功能说明
├─ 分析需要重构的代码
├─ 制定重构计划
└─ 执行重构并验证

# 示例：重构组件
/wflow refactor src/components/OldButton.tsx

# 输出：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 重构分析
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📋 当前问题：
1. ⚠️ 组件行数过多 (320 行)
2. ⚠️ 嵌套条件过多 (6 层)
3. ⚠️ 缺少 TypeScript 类型
4. ⚠️ 样式未使用 CSS 变量

📝 重构计划：
1. [1/4] 拆分为基础组件
   └─ Button.vue → BaseButton.vue
2. [2/4] 提取样式组件
   └─ ButtonStyles.vue
3. [3/4] 添加 TypeScript 类型
4. [4/4] 使用 CSS 变量

⚠️ 需要确认：
├─ 是否保留所有现有功能？
├─ 是否需要添加新功能？
└─ 测试覆盖是否完整？

[输入 y 继续，n 取消]
```

#### Test Flow

```markdown
# 功能说明
├─ 识别需要测试的代码
├─ 生成测试用例
└─ 运行测试并报告

# 示例：生成测试
/wflow test src/utils/calculate.ts

# 输出：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🧪 测试生成
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📁 目标文件：src/utils/calculate.ts

📊 分析结果：
├─ 函数数量：3
├─ 公共函数：2
└─ 已有测试：0

📝 生成计划：
1. [1/2] 生成 calculateTotal 测试
2. [2/2] 生成 calculateTax 测试

✅ 生成完成！

test/utils/calculate.test.ts
├─ calculateTotal
│  ├─ ✓ 正常情况
│  ├─ ✓ 空数组
│  ├─ ✓ 边界值
│  └─ ✓ 负数处理
└─ calculateTax
   ├─ ✓ 标准税率
   ├─ ✓ 零金额
   └─ ✓ 超额金额

📊 覆盖率：85%
```

### 2. 智能搜索

| 搜索类型 | 说明 | 语法 |
|---------|------|------|
| **语义搜索** | 理解代码意图 | `/search [自然语言]` |
| **正则搜索** | 模式匹配 | `/regex [pattern]` |
| **结构搜索** | AST 级别搜索 | `/ast [条件]` |
| **跨仓库搜索** | 多仓库同时搜索 | `/search --all [query]` |

#### 语义搜索示例

```markdown
# 自然语言搜索
/search 查找处理用户认证的代码

# 结果：
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔍 语义搜索结果：处理用户认证的代码
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. src/auth/login.ts (相关性: 95%)
   匹配：用户登录、密码验证、JWT 生成

2. src/auth/register.ts (相关性: 90%)
   匹配：用户注册、邮箱验证、密码加密

3. src/middleware/auth.ts (相关性: 88%)
   匹配：认证中间件、Token 验证

4. src/auth/oauth.ts (相关性: 85%)
   匹配：OAuth 认证、第三方登录

5. src/auth/session.ts (相关性: 80%)
   匹配：会话管理、Session 存储

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3. 终端集成

Windsurf 的终端与 AI 深度集成：

```bash
# AI 辅助终端命令
> @ai 帮我创建一个 Docker 容器运行 PostgreSQL

# Cascade 输出：
```bash
# PostgreSQL Docker 容器创建命令
docker run --name my-postgres \
  --network my-network \
  -e POSTGRES_PASSWORD=mysecretpassword \
  -e POSTGRES_DB=mydb \
  -e PGDATA=/var/lib/postgresql/data/pgdata \
  -p 5432:5432 \
  -v postgres-data:/var/lib/postgresql/data \
  -d postgres:15-alpine

# 常用命令：
# 连接：docker exec -it my-postgres psql -U postgres
# 停止：docker stop my-postgres
# 删除：docker rm my-postgres
```

> 请执行上述命令吗？ [y/n]
> y
# 正在执行...
# Pulling image postgres:15-alpine...
# Creating container...
# Container started successfully (ID: abc123)

✅ 容器创建成功！
```

#### 终端 AI 命令

```bash
# @ai 命令示例

# 命令生成
@ai 创建一个 Python 虚拟环境并安装 Django
# 输出：
# python -m venv venv
# source venv/bin/activate
# pip install django

# 命令解释
@ai explain docker-compose up -d
# 输出：
# docker-compose up -d:
# - docker-compose: Docker Compose 工具
# - up: 创建并启动容器
# - -d: 后台运行（detached mode）

# 错误诊断
@ai 这个错误是什么意思？
# Error: EACCES: permission denied, mkdir '/usr/local/lib/node_modules'
# 输出：
# 这是权限错误，npm 尝试在系统目录安装全局包
# 解决方案：
# 1. 使用 sudo（不推荐）
# 2. 配置 npm prefix 到用户目录
# 3. 使用 nvm 管理 Node.js
```

### 4. 文档生成

#### 智能 Docstring

```python
def calculate_distance(lat1: float, lon1: float, lat2: float, lon2: float) -> float:
    """
    使用 Haversine 公式计算两点间的地球表面距离。
    
    Args:
        lat1: 第一个点的纬度（度数，范围 -90 到 90）
        lon1: 第一个点的经度（度数，范围 -180 到 180）
        lat2: 第二个点的纬度（度数，范围 -90 到 90）
        lon2: 第二个点的经度（度数，范围 -180 到 180）
    
    Returns:
        float: 两点间的距离（公里）
    
    Raises:
        ValueError: 当纬度或经度超出有效范围时
    
    Example:
        >>> calculate_distance(40.7128, -74.0060, 51.5074, -0.1278)
        5570.22  # 纽约到伦敦的距离
        
        >>> calculate_distance(0, 0, 0, 180)
        20015.09  # 赤道上两点距离
    
    Note:
        - 使用 WGS-84 地球椭球体模型
        - 精度约为 ±0.5%
    """
    import math
    
    # 验证输入
    if not (-90 <= lat1 <= 90 and -90 <= lat2 <= 90):
        raise ValueError("纬度必须在 -90 到 90 之间")
    if not (-180 <= lon1 <= 180 and -180 <= lon2 <= 180):
        raise ValueError("经度必须在 -180 到 180 之间")
    
    # 转换为弧度
    R = 6371  # 地球半径（公里）
    lat1_rad = math.radians(lat1)
    lat2_rad = math.radians(lat2)
    delta_lat = math.radians(lat2 - lat1)
    delta_lon = math.radians(lon2 - lon1)
    
    # Haversine 公式
    a = (math.sin(delta_lat / 2) ** 2 +
         math.cos(lat1_rad) * math.cos(lat2_rad) *
         math.sin(delta_lon / 2) ** 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    
    return R * c
```

### 5. 调试辅助

| 功能 | 说明 |
|------|------|
| 错误解释 | 分析错误日志并提供原因 |
| 断点建议 | 推荐合适的断点位置 |
| 变量追踪 | 可视化变量变化过程 |
| 修复建议 | 提供多种修复方案 |

```typescript
// 调试示例
// 错误日志：
// TypeError: Cannot read property 'map' of undefined
//     at processItems (src/utils/data.ts:42)
//     at handleData (src/handlers/processor.ts:15)

/*
 * Cascade 分析：
 * 
 * 🔍 根因分析：
 * 
 * 问题位置：src/utils/data.ts:42
 * 问题代码：items.map(...)
 * 错误类型：TypeError
 * 
 * 可能原因：
 * 
 * 1. [高概率 75%] data.items 为 undefined
 *    - src/handlers/processor.ts:15 调用 processItems(data.items)
 *    - 当 data.items 未定义时触发
 * 
 * 2. [中概率 20%] 参数传递错误
 *    - processItems 接收的参数不是预期类型
 * 
 * 3. [低概率 5%] 异步问题
 *    - data.items 在异步操作后未正确赋值
 * 
 * 🛠️ 修复建议：
 * 
 * 方案 1：添加空值检查（推荐）
 * ```typescript
 * export function processItems(items?: Item[]): ProcessedItem[] {
 *   if (!items) {
 *     console.warn('items is undefined, returning empty array');
 *     return [];
 *   }
 *   return items.map(item => ({ ...item, processed: true }));
 * }
 * ```
 * 
 * 方案 2：添加默认空数组
 * ```typescript
 * export function processItems(items: Item[] = []): ProcessedItem[] {
 *   return items.map(item => ({ ...item, processed: true }));
 * }
 * ```
 * 
 * 方案 3：添加类型守卫
 * ```typescript
 * function isValidItems(data: any): data is Item[] {
 *   return Array.isArray(data);
 * }
 * 
 * export function processItems(data: unknown): ProcessedItem[] {
 *   if (!isValidItems(data)) {
 *     throw new TypeError('Expected array of items');
 *   }
 *   return data.map(item => ({ ...item, processed: true }));
 * }
 * ```
 */
```

---

## 价格体系

### 价格对比表

| 套餐 | 价格 | 功能 | 适用场景 |
|------|------|------|----------|
| **Free** | $0 | 基础补全 + Cascade 基础模式 | 个人体验 |
| **Pro** | $10/月 或 $96/年 | 无限 Supercomplete + Cascade 完整功能 | 个人开发者 |
| **Enterprise** | 定制报价 | SSO、策略管理、专属支持 | 团队/企业 |

### 各套餐详细对比

#### Free 套餐

| 功能 | 配额/限制 |
|------|----------|
| 代码补全 | 基础功能 |
| Cascade 对话 | 有限次数/天 |
| Supercomplete | ❌ |
| Flow 功能 | ❌ |
| 终端 AI | ❌ |
| 多模型支持 | 基础模型 |

> [!NOTE]
> Free 套餐适合轻度体验 AI 编程功能，但不建议作为主要开发工具。

#### Pro 套餐（推荐）

| 功能 | 说明 |
|------|------|
| 代码补全 | 增强版，含 Supercomplete |
| Cascade 对话 | 无限次数 |
| AI Flow | 全部 Flow 功能 |
| 终端 AI | 完整支持 |
| 优先使用新模型 | ✅ |
| 多设备同步 | ✅ |
| 优先客服支持 | ✅ |

#### Enterprise 套餐

| 功能 | 说明 |
|------|------|
| 所有 Pro 功能 | ✅ |
| SSO/SAML | ✅ |
| 组织策略管理 | ✅ |
| 使用分析仪表板 | ✅ |
| 私有部署选项 | 可选 |
| 专属客户经理 | ✅ |
| SLA 保障 | 可选 |

> [!TIP]
> Windsurf Pro 的价格仅为 Cursor Pro 的一半，功能却不逊色，是预算有限的个人开发者的优质选择。

### 免费额度说明

| 功能 | 每日限制 |
|------|---------|
| Cascade 对话 | 50 次 |
| Supercomplete | 20 次 |
| Flow 执行 | 5 次 |
| 终端 AI | 30 次 |

### 计费对比

```markdown
# 年度性价比分析

| 套餐 | 月付 | 年付 | 节省 | 日均成本 |
|------|------|------|------|----------|
| Pro 月付 | $10 | $120 | - | $0.33 |
| Pro 年付 | $8 | $96 | $24 (20%) | $0.26 |
| Cursor Pro | $20 | $192 | - | $0.53 |
| Copilot Pro | $10 | $100 | $20 (17%) | $0.27 |

结论：Windsurf Pro 年付是最具性价比的选择
```

---

## Windsurf Rules 与配置

### 什么是 Windsurf Rules

Windsurf Rules 类似于 Cursor Rules，允许用户定义 AI 行为规范：

```
项目根目录/
├── .windsurf/
│   ├── rules/
│   │   ├── default.mdc      # 全局默认规则
│   │   ├── code-style.mdc   # 代码风格规则
│   │   ├── security.mdc     # 安全规则
│   │   └── framework.mdc    # 框架规范
│   └── config.yaml          # 配置文件
```

### Rules 文件格式

```markdown
# windsurf.rules
# 描述：React + TypeScript 项目规范

## 技术栈
- 框架：React 18
- 语言：TypeScript 5.x
- 构建：Vite
- 样式：Tailwind CSS
- 状态：Zustand
- 路由：React Router 6

## 代码风格
- 使用 2 空格缩进
- 组件文件：PascalCase.tsx
- 工具文件：camelCase.ts
- 测试文件：ComponentName.test.tsx
- 样式文件：ComponentName.module.css

## 命名规范
- 变量和函数：camelCase
- 常量：UPPER_SNAKE_CASE
- 类型和接口：PascalCase
- CSS 类名：kebab-case
- 文件名：kebab-case

## TypeScript 规范
- 禁用 any，使用 unknown 替代
- 优先使用 interface
- 使用泛型时添加约束
- 导出类型而非实现

## React 规范
- 使用函数组件
- 优先使用 Hooks
- 组件放在 components/ 目录
- 业务逻辑放在 hooks/ 目录
- 状态管理使用 Zustand
- 样式使用 Tailwind 类名

## 安全规范
- 禁止内联 onClick
- 禁止 dangerouslySetInnerHTML
- 所有用户输入必须验证
- 使用参数化查询
```

### 配置文件示例

```yaml
# .windsurf/config.yaml

# Cascade 配置
cascade:
  model: codeium
  temperature: 0.7
  max_tokens: 4096
  context_window: full_project

# Supercomplete 配置
supercomplete:
  enabled: true
  auto_trigger: true
  max_lines: 100

# Flow 配置
flow:
  auto_save: true
  confirm_destructive: true
  max_parallel_tasks: 3

# 快捷键配置
keybindings:
  cascade_chat: "cmd+l"
  supercomplete: "tab"
  flow_panel: "cmd+shift+f"
```

---

## Agentic Workflow 深度使用

### 什么是 Agentic Workflow

Agentic Workflow 是 Windsurf 的高级功能，允许 AI 自主执行复杂的多步骤任务：

```markdown
# Agentic Workflow 特点

1. 自主规划
   ├─ AI 分析任务需求
   ├─ 拆解为可执行步骤
   └─ 制定执行计划

2. 工具调用
   ├─ 文件操作（读写编辑）
   ├─ 终端执行
   ├─ 搜索和索引
   └─ Web 访问

3. 迭代优化
   ├─ 验证执行结果
   ├─ 错误处理和恢复
   └─ 结果优化

4. 人类监督
   ├─ 关键步骤确认
   ├─ 随时干预能力
   └─ 最终结果审核
```

### 工作流程示例

```markdown
# 示例：使用 Agentic Workflow 实现用户模块

## 输入
"使用 Next.js App Router 实现用户管理模块，
包含用户列表、详情、编辑功能，
使用 Prisma ORM 连接 PostgreSQL，
实现完整的 CRUD API，
前端使用 React Query 管理数据。"

## Agentic Workflow 执行过程

### 阶段 1：项目分析

🔍 分析现有项目结构...

├─ 检测到 Next.js 14 项目
├─ 已配置 Prisma
├─ 使用 Tailwind CSS
└─ 项目结构符合规范

### 阶段 2：架构设计

📐 设计数据库 Schema...

```prisma
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  posts     Post[]
}

enum Role {
  USER
  ADMIN
}

model Post {
  id       String   @id @default(cuid())
  title    String
  content  String
  authorId String
  author   User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
}
```

### 阶段 3：代码生成

#### 3.1 创建数据库 Schema
✅ prisma/schema.prisma

#### 3.2 生成 API 路由
✅ src/app/api/users/route.ts
✅ src/app/api/users/[id]/route.ts

#### 3.3 创建服务层
✅ src/services/userService.ts

#### 3.4 生成前端组件
✅ src/components/UserList.tsx
✅ src/components/UserDetail.tsx
✅ src/components/UserForm.tsx

#### 3.5 创建页面
✅ src/app/users/page.tsx
✅ src/app/users/[id]/page.tsx
✅ src/app/users/[id]/edit/page.tsx

#### 3.6 编写类型定义
✅ src/types/user.ts

### 阶段 4：验证

🧪 运行类型检查...
✅ TypeScript: 无错误

🧪 运行构建...
✅ Build: 成功

🧪 运行测试...
✅ Tests: 3 passed

### 完成！

📊 生成摘要：
├─ 新增文件：12
├─ 修改文件：2
├─ 代码行数：850+
└─ 测试覆盖：75%

⚠️ 建议：
1. 添加认证中间件保护 API
2. 实现分页功能
3. 添加输入验证（Zod）
```

### 与传统开发的对比

```markdown
# 开发方式对比

┌─────────────────────────────────────────────────────────────┐
│                      传统开发流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  步骤    │  时间    │  人工操作                               │
│  ────────┼─────────┼───────────────                          │
│  需求分析 │  1-2天   │  人工                                   │
│  架构设计 │  2-3天   │  人工                                   │
│  数据库   │  4-8小时  │  人工 + AI 辅助                         │
│  API 开发 │  1-2天   │  人工 + AI 辅助                         │
│  前端开发 │  2-3天   │  人工 + AI 辅助                         │
│  测试     │  4-8小时  │  人工                                   │
│  文档     │  2-4小时  │  人工                                   │
│  ────────┼─────────┼───────────────                          │
│  总计     │  7-14天  │  大量人工操作                           │
│                                                              │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Agentic Workflow 开发流程                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  步骤    │  时间    │  人工操作                               │
│  ────────┼─────────┼───────────────                          │
│  需求描述 │  1-2小时 │  人工                                   │
│  AI 执行 │  1-4小时  │  监督和确认                             │
│  代码审查 │  1-2小时 │  人工                                   │
│  测试验证 │  1-2小时 │  人工                                   │
│  优化调整 │  2-4小时 │  人工                                   │
│  ────────┼─────────┼───────────────                          │
│  总计     │  6-14小时 │  少量人工操作                           │
│  效率提升 │  5-10倍  │                                        │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 多文件协作模式

### 文件组操作

Windsurf 支持同时操作多个相关文件：

```markdown
# 文件组定义

## 自动检测依赖
当用户选择或修改一个文件时，Windsurf 自动识别相关文件：

src/components/UserCard.tsx
    ├─ depends on: src/types/User.ts
    ├─ imports: src/hooks/useUser.ts
    └─ tests: src/components/UserCard.test.tsx

## 批量操作
```
┌────────────────────────────────────────┐
│         文件组操作面板                    │
├────────────────────────────────────────┤
│                                        │
│  📁 选中的文件组：User 模块              │
│  ├─ ☐ src/types/User.ts              │
│  ├─ ☑ src/components/UserCard.tsx   │
│  ├─ ☐ src/hooks/useUser.ts           │
│  ├─ ☐ src/services/userService.ts    │
│  └─ ☐ src/__tests__/UserCard.test.tsx│
│                                        │
│  操作：                                 │
│  [ ] 提取公共类型到 types/             │
│  [ ] 重构组件结构                      │
│  [ ] 添加缓存优化                      │
│  [ ] 更新测试用例                      │
│                                        │
│  [执行选中的操作]                       │
│                                        │
└────────────────────────────────────────┘
```
```

### 跨文件重构

```markdown
# 示例：跨文件重命名

任务：将所有文件中的 userId 重命名为 id

Windsurf 执行：

🔄 扫描项目...

📁 发现 23 个文件包含 userId：

1. src/types/User.ts
   - userId → id

2. src/services/userService.ts
   - userId → id
   - 参数类型更新

3. src/hooks/useUser.ts
   - userId → id

...

📝 修改摘要：
├─ 修改文件：23
├─ 修改行数：156
├─ 类型更新：12
└─ 测试更新：8

✅ 重命名完成！
```

### 项目级修改

```markdown
# 示例：添加国际化支持

任务：为整个项目添加 i18n 支持

Windsurf 执行计划：

## 阶段 1：基础设施

1.1 安装依赖
✅ i18next
✅ react-i18next
✅ i18next-browser-languagedetector

1.2 创建配置
✅ src/i18n/index.ts
✅ src/i18n/config.ts

## 阶段 2：语言文件

2.1 创建语言目录
✅ src/locales/en/
✅ src/locales/zh/

2.2 添加翻译
✅ src/locales/en/translation.json
✅ src/locales/zh/translation.json

## 阶段 3：组件集成

3.1 更新组件
✅ src/components/Button.tsx
✅ src/components/Header.tsx
✅ src/components/Footer.tsx
...

3.2 添加语言切换器
✅ src/components/LanguageSwitcher.tsx

## 阶段 4：路由集成

4.1 添加语言前缀
✅ src/app/[lang]/layout.tsx
✅ 更新所有页面路由

## 验证

✅ 构建成功
✅ 类型检查通过
✅ 测试通过

✅ 项目国际化完成！
```

---

## 与其他工具对比

### Windsurf vs Cursor vs Copilot 功能对比

| 特性 | Windsurf | Cursor | GitHub Copilot |
|------|----------|--------|----------------|
| **价格** | $10/月 | $20/月 | $10/月 |
| **代码补全** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Supercomplete** | ✅ 独有 | ❌ | ❌ |
| **AI 对话** | Cascade | AI Chat | Copilot Chat |
| **Agent 模式** | Flow | Agent | Agents |
| **多文件编辑** | Flow 自动化 | ✅ | ❌ |
| **上下文窗口** | 全项目 | 全项目 | 当前文件 |
| **IDE 基础** | VS Code 分支 | 自研 | 独立插件 |
| **VS Code 兼容** | ✅ | ❌ | ✅ |
| **终端集成** | ✅ | ✅ | CLI 单独 |
| **免费套餐** | 较完善 | 限制较多 | 较完善 |

### 技术特点对比

| 维度 | Windsurf | Cursor | Copilot |
|------|----------|--------|---------|
| **模型基础** | Codeium + 多模型 | 多模型可选 | OpenAI GPT |
| **上下文理解** | 全项目索引 | 全项目理解 | 有限上下文 |
| **自动化能力** | Flow 自动化 | Agent 模式 | 较弱 |
| **学习曲线** | 中等 | 较低 | 低 |
| **创新特性** | Supercomplete | 深度集成 | GitHub 集成 |

### 场景对比

| 场景 | 推荐工具 | 原因 |
|------|---------|------|
| 预算有限 | **Windsurf Pro** | 价格最低，功能完整 |
| AI 原生体验 | **Cursor** | 最佳 AI 集成 |
| GitHub 生态 | **Copilot** | 深度 GitHub 集成 |
| 快速原型 | **Windsurf/Cursor** | Flow/Agent 模式 |
| 代码补全优先 | **Windsurf** | Supercomplete 技术 |
| JetBrains 用户 | **Copilot** | 原生 JetBrains 支持 |
| 学生用户 | **Windsurf/Copilot** | 免费额度充足 |

### 定位差异

```
Windsurf:  "AI 协作伙伴" - 强调人机协作
Cursor:    "AI 原生 IDE" - AI 深度集成
Copilot:  "AI 编程助手" - 辅助工具定位
```

---

## 局限性分析

### 1. 产品成熟度

| 局限点 | 说明 | 影响 |
|-------|------|------|
| **历史较短** | 2024年才发布 | 社区资源较少 |
| **功能迭代快** | 频繁更新可能导致不稳定 | 需要适应变化 |
| **企业功能** | 相对不完善 | 大型企业可能不满足 |
| **文档资源** | 相对较少 | 学习曲线稍高 |

### 2. 技术局限性

| 局限点 | 说明 | 缓解方案 |
|-------|------|---------|
| **Supercomplete 延迟** | 整函数生成较慢 | 可切换到传统补全 |
| **上下文索引** | 首次需要建立索引 | 后台静默进行 |
| **离线支持** | 依赖云端 | 暂无本地方案 |
| **内存占用** | 索引占用内存 | 优化索引策略 |

### 3. 生态局限性

| 局限点 | 说明 |
|-------|------|
| **插件生态** | VS Code 分支，插件兼容性待验证 |
| **主题扩展** | 自研 IDE，主题支持有限 |
| **快捷键差异** | 与标准 VS Code 有差异 |
| **社区规模** | 相对较小 |

> [!IMPORTANT]
> Windsurf 基于 VS Code 分支开发，虽然兼容大部分 VS Code 插件，但仍可能存在兼容性问题。建议在正式使用前测试关键插件。

---

## 选型建议

### 何时选择 Windsurf

| 场景 | 推荐程度 | 原因 |
|------|---------|------|
| 预算有限的开发者 | ⭐⭐⭐⭐⭐ | $10/月即可获得完整功能 |
| 追求代码补全效率 | ⭐⭐⭐⭐⭐ | Supercomplete 技术领先 |
| 希望保持 VS Code 习惯 | ⭐⭐⭐⭐ | 高度兼容 VS Code |
| 需要工作流自动化 | ⭐⭐⭐⭐ | AI Flow 功能完善 |
| 初次尝试 AI 编程 | ⭐⭐⭐⭐ | 上手简单，免费额度充足 |
| 需要深度上下文理解 | ⭐⭐⭐⭐ | 全项目索引能力 |

### 迁移指南

#### 从 VS Code + Copilot 迁移

```bash
# 1. 导出 VS Code 设置
code --export-configuration > vscode-settings.json

# 2. 安装 Windsurf
# 从 windsurf.ai 下载安装包

# 3. 导入设置
# Windsurf 会提示导入 VS Code 设置

# 4. 安装必要的插件
# Windsurf 商店中选择需要的插件

# 5. 配置 Copilot 替代功能
# 设置 → AI → 启用 Cascade
```

#### 从 Cursor 迁移

| 功能映射 | Cursor | Windsurf |
|---------|--------|----------|
| AI Chat | Cmd+K | Cmd+L |
| Agent Mode | Cmd+Shift+G | /flow 或 Cascade |
| Composer | 多文件编辑 | Flow Architect |
| Cursor Rules | Rules | 项目配置 |

### 新手入门建议

1. **第一周**：体验免费额度，熟悉 Cascade 对话
2. **第二周**：尝试 Supercomplete，感受整函数补全
3. **第三周**：使用 AI Flow 自动化工作流
4. **第四周**：根据需求决定是否升级 Pro

---

## 进阶技巧与最佳实践

### 1. 效率提升技巧

#### 快捷键优化

```markdown
# 推荐快捷键配置

## Cascade 相关
| 功能 | 快捷键 | 说明 |
|------|--------|------|
| 打开 Cascade | Cmd+L | 主要对话入口 |
| 快速搜索 | Cmd+K | 语义搜索 |
| Flow 面板 | Cmd+Shift+P | 工作流面板 |
| 终端 AI | Ctrl+` | 终端内 AI |

## 编辑相关
| 功能 | 快捷键 | 说明 |
|------|--------|------|
| Supercomplete | Tab | 触发整函数补全 |
| 内联补全 | Cmd+Space | 标准补全 |
| 多光标 | Cmd+D | 选择下一个匹配 |
```

#### Cascade 对话技巧

```markdown
# 高效对话策略

## 1. 使用上下文标记
@file:src/users/model.ts    # 引用文件
@folder:src/api            # 引用目录
@search:auth middleware     # 搜索结果

## 2. 指定输出格式
"以 TypeScript 类型定义的形式输出"

## 3. 分步执行
"首先分析项目结构，然后制定计划，确认后再执行"

## 4. 约束条件
"仅修改 src/auth/ 目录，保持其他文件不变"
```

### 2. Supercomplete 最佳实践

```markdown
# Supercomplete 使用技巧

## 1. 提供完整上下文
// ✅ 好的示例：完整的函数签名
interface UserRepository {
  findById(id: string): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(user: CreateUserDto): Promise<User>;
}

// Supercomplete 会生成完整实现

## 2. 添加类型注解
// ✅ 好的示例
const processItems = (items: Item[]): ProcessedItem[] => {
  // Supercomplete 会理解输入输出类型并生成完整实现
};

// ❌ 差的示例
const processItems = (items) => {
  // 缺少类型信息
};

## 3. 使用注释引导
// ✅ 好的示例
/**
 * 分页获取用户列表
 * @param page 页码（从1开始）
 * @param limit 每页数量
 * @param filters 筛选条件
 * @returns 分页结果
 */
async function getUsers(page, limit, filters) {
  // Supercomplete 会生成完整实现
}
```

### 3. Flow 自动化

```markdown
# Flow 使用场景

## 适用场景
├─ 项目初始化
├─ 功能模块开发
├─ 代码重构
├─ 测试生成
└─ 文档编写

## 不适用场景
├─ 简单的单行修改
├─ 需要创意设计的任务
├─ 高度敏感的操作
└─ 实时性要求高的任务

## Flow 执行策略
1. 简单任务 → 直接执行
2. 中等任务 → 确认后执行
3. 复杂任务 → 分步骤执行
```

---

## 企业级配置

### 团队配置

```yaml
# .windsurf/enterprise.yaml

# 组织设置
organization:
  name: "Acme Corp"
  domain: "acme.com"
  
# 团队规则
team:
  rules_directory: ".windsurf/team-rules"
  enforce_rules: true
  allow_override: false

# 策略管理
policies:
  language_restrictions:
    allowed: ["typescript", "python", "go", "java"]
    blocked: ["ruby", "php"]
  
  security:
    block_destructive: true
    require_approval_for_delete: true
    audit_all_operations: true

# 使用分析
analytics:
  enabled: true
  report_interval: "daily"
  notify_on_anomaly: true

# SSO 配置
sso:
  provider: "okta"
  enabled: true
```

### 部署配置

```json
{
  "windsurf": {
    "enterprise": {
      "license_key": "xxx-xxx-xxx",
      "features": {
        "sso": true,
        "audit_logs": true,
        "custom_rules": true
      },
      "storage": {
        "type": "cloud",
        "region": "us-west-2",
        "retention_days": 90
      }
    }
  }
}
```

---

## 常见问题与故障排除

### 安装问题

#### 问题 1：Windsurf 无法启动

**解决方案**：
1. 确认系统要求（macOS 10.15+ / Windows 10+ / Linux）
2. 清除缓存：`rm -rf ~/.windsurf/cache`
3. 重新安装
4. 检查日志：`~/.windsurf/logs/`

#### 问题 2：扩展不兼容

**解决方案**：
1. 检查扩展兼容性列表
2. 更新到最新版本
3. 禁用冲突扩展

### 使用问题

#### 问题 3：Cascade 无响应

**解决方案**：
1. 检查网络连接
2. 清除对话历史
3. 重启应用
4. 检查 API 配额

#### 问题 4：Supercomplete 不触发

**解决方案**：
1. 确认已启用 Supercomplete
2. 检查是否为支持的语言
3. 提供足够的上下文

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Windsurf 官网 | https://windsurf.ai |
| 文档 | https://docs.windsurf.ai |
| 博客 | https://windsurf.ai/blog |
| Changelog | https://docs.windsurf.ai/changelog |
| Discord | https://discord.gg/windsurf |

### 社区资源

| 资源 | 说明 |
|------|------|
| Discord | 官方社区 |
| GitHub | Codeium 开源项目 |
| Twitter | 官方更新 |
| Reddit | r/windsurf |

### 价格信息

| 套餐 | 链接 |
|------|------|
| Free | https://windsurf.ai/pricing |
| Pro | https://windsurf.ai/pricing |
| Enterprise | 联系销售 |

---

> [!SUCCESS]
> Windsurf 作为 Codeium 出品的 AI 编程助手，凭借其独特的 Supercomplete 技术和亲民的价格（$10/月），在 AI 编程工具市场占据了重要位置。其 Flow 自动化功能和 Cascade 智能体架构为开发者提供了高效的工作流优化能力。对于预算有限但希望获得优质 AI 编程体验的开发者，Windsurf 是 2026 年极具性价比的选择。

---

## Windsurf 高级技巧与生态集成

### 1. 数据库工具链集成

```typescript
// Windsurf 与 Prisma 集成示例

// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Windsurf AI 辅助生成模型
model Blog {
  id        String   @id @default(cuid())
  title     String   @db.VarChar(255)
  slug      String   @unique
  content   String   @db.Text
  excerpt   String?  @db.VarChar(500)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  published Boolean  @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  tags      Tag[]
  views     Int      @default(0)
  
  @@index([authorId])
  @@index([published])
  @@index([slug])
}

model Tag {
  id    String @id @default(cuid())
  name  String @unique
  slug  String @unique
  blogs Blog[]
  
  @@index([slug])
}
```

```typescript
// Windsurf 辅助生成的 Prisma 服务
import { PrismaClient } from '@prisma/client';
import { cacheService } from './cache';

const prisma = new PrismaClient();

export const blogService = {
  async findBySlug(slug: string) {
    const cacheKey = `blog:${slug}`;
    const cached = await cacheService.get(cacheKey);
    if (cached) return cached;
    
    const blog = await prisma.blog.findUnique({
      where: { slug },
      include: {
        author: {
          select: { id: true, name: true, avatar: true }
        },
        tags: true
      }
    });
    
    if (blog) {
      await cacheService.set(cacheKey, blog, 3600); // 1 hour cache
    }
    
    return blog;
  },
  
  async findPublished(options: {
    page?: number;
    limit?: number;
    tagId?: string;
  }) {
    const { page = 1, limit = 10, tagId } = options;
    const skip = (page - 1) * limit;
    
    const where: any = { published: true };
    if (tagId) {
      where.tags = { some: { id: tagId } };
    }
    
    const [blogs, total] = await Promise.all([
      prisma.blog.findMany({
        where,
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' },
        include: {
          author: { select: { id: true, name: true } },
          tags: true,
          _count: { select: { comments: true } }
        }
      }),
      prisma.blog.count({ where })
    ]);
    
    return {
      blogs,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    };
  },
  
  async incrementViews(id: string) {
    await prisma.blog.update({
      where: { id },
      data: { views: { increment: 1 } }
    });
  }
};
```

### 2. 前端框架深度集成

```tsx
// Windsurf 辅助生成的 React 组件

import { useState, useEffect, useCallback } from 'react';
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { blogService, Blog, BlogListResponse } from '@/services/blog';

interface UseBlogListOptions {
  page?: number;
  limit?: number;
  tagId?: string;
  enabled?: boolean;
}

export function useBlogList(options: UseBlogListOptions = {}) {
  const { page = 1, limit = 10, tagId, enabled = true } = options;
  
  return useQuery({
    queryKey: ['blogs', { page, limit, tagId }],
    queryFn: () => blogService.findPublished({ page, limit, tagId }),
    enabled,
    staleTime: 5 * 60 * 1000, // 5 minutes
    keepPreviousData: true,
  });
}

export function useBlog(slug: string, enabled = true) {
  const queryClient = useQueryClient();
  
  const query = useQuery({
    queryKey: ['blog', slug],
    queryFn: () => blogService.findBySlug(slug),
    enabled,
    staleTime: 10 * 60 * 1000, // 10 minutes
  });
  
  const incrementViewsMutation = useMutation({
    mutationFn: () => blogService.incrementViews(query.data!.id),
    onSuccess: () => {
      queryClient.invalidateQueries(['blog', slug]);
    }
  });
  
  useEffect(() => {
    if (query.data && typeof window !== 'undefined') {
      incrementViewsMutation.mutate();
    }
  }, [query.data?.id]);
  
  return query;
}

export function useCreateBlog() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: blogService.create,
    onSuccess: () => {
      queryClient.invalidateQueries(['blogs']);
    }
  });
}

export function useUpdateBlog() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: Partial<Blog> }) =>
      blogService.update(id, data),
    onSuccess: (blog) => {
      queryClient.invalidateQueries(['blogs']);
      queryClient.setQueryData(['blog', blog.slug], blog);
    }
  });
}
```

### 3. 状态管理集成

```typescript
// Windsurf 辅助生成的状态管理

import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  
  // Actions
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  register: (data: RegisterData) => Promise<void>;
  refreshToken: () => Promise<void>;
}

interface User {
  id: string;
  email: string;
  name: string;
  avatar?: string;
  role: 'user' | 'admin';
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      isLoading: false,
      
      login: async (email: string, password: string) => {
        set({ isLoading: true });
        try {
          const response = await authService.login({ email, password });
          set({
            user: response.user,
            token: response.token,
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
          token: null,
          isAuthenticated: false,
        });
      },
      
      register: async (data: RegisterData) => {
        set({ isLoading: true });
        try {
          const response = await authService.register(data);
          set({
            user: response.user,
            token: response.token,
            isAuthenticated: true,
            isLoading: false,
          });
        } catch (error) {
          set({ isLoading: false });
          throw error;
        }
      },
      
      refreshToken: async () => {
        const { token } = get();
        if (!token) return;
        
        try {
          const response = await authService.refreshToken(token);
          set({ token: response.token });
        } catch {
          get().logout();
        }
      },
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        token: state.token,
        user: state.user,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
);
```

### 4. 实时功能集成

```typescript
// Windsurf 辅助生成的 WebSocket 服务

import { io, Socket } from 'socket.io-client';
import { useAuthStore } from '@/stores/auth';
import { useEffect, useRef } from 'react';

interface ServerToClientEvents {
  'notification:new': (data: Notification) => void;
  'message:new': (data: Message) => void;
  'user:online': (data: { userId: string }) => void;
  'user:offline': (data: { userId: string }) => void;
}

interface ClientToServerEvents {
  'room:join': (roomId: string) => void;
  'room:leave': (roomId: string) => void;
  'message:send': (data: { roomId: string; content: string }) => void;
}

type TypedSocket = Socket<ServerToClientEvents, ClientToServerEvents>;

class SocketService {
  private socket: TypedSocket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 5;
  
  connect(): TypedSocket {
    if (this.socket?.connected) {
      return this.socket;
    }
    
    const token = useAuthStore.getState().token;
    
    this.socket = io(process.env.NEXT_PUBLIC_WS_URL!, {
      auth: { token },
      transports: ['websocket'],
      reconnection: true,
      reconnectionAttempts: this.maxReconnectAttempts,
      reconnectionDelay: 1000,
    });
    
    this.setupListeners();
    return this.socket;
  }
  
  private setupListeners() {
    if (!this.socket) return;
    
    this.socket.on('connect', () => {
      console.log('Socket connected');
      this.reconnectAttempts = 0;
    });
    
    this.socket.on('disconnect', (reason) => {
      console.log('Socket disconnected:', reason);
    });
    
    this.socket.on('connect_error', (error) => {
      this.reconnectAttempts++;
      if (this.reconnectAttempts >= this.maxReconnectAttempts) {
        console.error('Max reconnection attempts reached');
      }
    });
  }
  
  disconnect() {
    if (this.socket) {
      this.socket.disconnect();
      this.socket = null;
    }
  }
  
  getSocket(): TypedSocket | null {
    return this.socket;
  }
}

export const socketService = new SocketService();

// React Hook
export function useSocket() {
  const socketRef = useRef<TypedSocket | null>(null);
  const isAuthenticated = useAuthStore((state) => state.isAuthenticated);
  
  useEffect(() => {
    if (isAuthenticated) {
      socketRef.current = socketService.connect();
    } else {
      socketService.disconnect();
      socketRef.current = null;
    }
    
    return () => {
      socketService.disconnect();
    };
  }, [isAuthenticated]);
  
  return socketRef.current;
}
```

### 5. 性能监控集成

```typescript
// Windsurf 辅助生成的性能监控

import { Logger } from 'tslog';

export const logger = new Logger({
  name: 'app',
  minLevel: process.env.NODE_ENV === 'production' ? 'info' : 'debug',
  transport: {
    target: 'pino',
    options: {
      level: process.env.LOG_LEVEL || 'info',
    },
  },
});

export class PerformanceMonitor {
  private static instance: PerformanceMonitor;
  private marks = new Map<string, number>();
  
  static getInstance(): PerformanceMonitor {
    if (!PerformanceMonitor.instance) {
      PerformanceMonitor.instance = new PerformanceMonitor();
    }
    return PerformanceMonitor.instance;
  }
  
  mark(label: string): void {
    this.marks.set(label, performance.now());
  }
  
  measure(label: string, startMark: string, endMark?: string): number {
    const start = this.marks.get(startMark);
    const end = endMark ? this.marks.get(endMark) : performance.now();
    
    if (!start) {
      logger.warn(`Start mark "${startMark}" not found`);
      return 0;
    }
    
    const duration = end - start;
    logger.info(`[Performance] ${label}: ${duration.toFixed(2)}ms`);
    
    return duration;
  }
  
  async track<T>(
    label: string,
    fn: () => Promise<T>
  ): Promise<T> {
    const start = performance.now();
    try {
      const result = await fn();
      const duration = performance.now() - start;
      
      logger.info(`[Track] ${label}: ${duration.toFixed(2)}ms`);
      
      if (duration > 1000) {
        logger.warn(`[Track] ${label} exceeded 1s: ${duration.toFixed(2)}ms`);
      }
      
      return result;
    } catch (error) {
      const duration = performance.now() - start;
      logger.error(`[Track] ${label} failed after ${duration.toFixed(2)}ms`, error);
      throw error;
    }
  }
  
  report(): PerformanceReport {
    return {
      memory: {
        used: Math.round(performance.memory?.usedJSHeapSize / 1048576),
        total: Math.round(performance.memory?.totalJSHeapSize / 1048576),
      },
      timing: {
        navigation: performance.timing?.navigationStart,
        load: performance.timing?.loadEventEnd,
      },
    };
  }
}

export const perfMonitor = PerformanceMonitor.getInstance();
```

---

## Windsurf 插件生态系统

### 1. 推荐的 VS Code 插件

| 插件名称 | 功能描述 | 优先级 |
|---------|---------|--------|
| Prettier | 代码格式化 | 必需 |
| ESLint | 代码检查 | 必需 |
| GitLens | Git 可视化 | 推荐 |
| Thunder Client | API 测试 | 推荐 |
| Error Lens | 错误高亮 | 推荐 |
| Import Cost | 导入成本显示 | 可选 |
| Regex Previewer | 正则预览 | 可选 |
| Docker | Docker 管理 | 可选 |
| Remote SSH | 远程开发 | 可选 |

### 2. Windsurf 特定配置

```json
{
  "windsurf.workspaces": {
    "shared": {
      "settings": true,
      "extensions": true,
      "keybindings": false
    }
  },
  "windsurf.ai": {
    "supercomplete": {
      "enabled": true,
      "debounceMs": 300,
      "maxLines": 50
    },
    "cascade": {
      "contextWindow": "fullProject",
      "modelPreference": ["claude-sonnet", "gpt-4"]
    }
  },
  "windsurf.terminal": {
    "aiEnabled": true,
    "shellIntegration": true
  }
}
```

---

---

## Windsurf 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 10.15+ / Windows 10+ / Linux (Ubuntu 18.04+) |
| 内存 | 4GB RAM |
| 磁盘空间 | 500MB 可用空间 |
| 网络 | 互联网连接 |

#### 推荐配置

| 配置项 | 推荐 |
|--------|------|
| 操作系统 | macOS 12+ / Windows 11 / Ubuntu 22.04+ |
| 内存 | 8GB+ RAM |
| 处理器 | 多核处理器 |
| 网络 | 稳定高速连接 |

### 安装步骤

#### macOS 安装

```bash
# 方法一：官网下载
# 1. 访问 https://windsurf.ai/
# 2. 点击 Download 按钮
# 3. 下载 .dmg 文件
# 4. 打开 .dmg 并拖入 Applications

# 方法二：Homebrew
brew install --cask windsurf
```

#### Windows 安装

```powershell
# 方法一：官网下载
# 1. 访问 https://windsurf.ai/
# 2. 下载 .exe 安装包
# 3. 运行安装程序

# 方法二：winget
winget install Windsurf.Windsurf
```

#### Linux 安装

```bash
# Debian/Ubuntu
# 1. 下载 .deb 文件
wget https://windsurf.ai/download/linux -O windsurf.deb

# 2. 安装
sudo dpkg -i windsurf.deb
sudo apt-get install -f

# Fedora
sudo rpm -i windsurf.rpm
```

### 首次启动配置

#### 1. 账户设置

```markdown
# 首次启动流程

1. 启动 Windsurf
2. 创建或登录 Windsurf 账户
3. 配置 AI 模型偏好
4. 设置工作目录
5. 熟悉界面布局
```

#### 2. AI 模型配置

```yaml
# .windsurf/config.yaml

cascade:
  # 主模型
  primary_model: codeium
  # 备用模型
  fallback_models:
    - claude-sonnet
    - gpt-4o
  
  # 温度设置
  temperature: 0.7
  
  # 最大 Token 数
  max_tokens: 8192
  
  # 上下文窗口
  context_window: full_project
  
  # 思考模式
  thinking:
    enabled: true
    depth: high

supercomplete:
  # 启用 Supercomplete
  enabled: true
  # 自动触发
  auto_trigger: true
  # 最大行数
  max_lines: 100
  # 延迟触发（毫秒）
  debounce_ms: 300
```

#### 3. 主题与界面

```yaml
# .windsurf/config.yaml

appearance:
  # 主题
  theme:
    mode: auto  # auto / light / dark
    color_scheme: one-dark-pro
  
  # 字体
  font:
    family: JetBrains Mono
    size: 14
    line_height: 1.6
    ligatures: true
  
  # 界面元素
  ui:
    tab_height: 36
    sidebar_width: 250
    panel_height: 300
```

### Windsurf Rules 进阶配置

#### Next.js 项目规则

```yaml
# .windsurf/rules/nextjs.mdc
# windsurf.rules
# Next.js 14 App Router 项目规范

## 技术栈
- 框架：Next.js 14
- 语言：TypeScript 5.x
- 样式：Tailwind CSS
- 状态：Zustand / React Query
- 数据库：Prisma + PostgreSQL
- 认证：NextAuth.js v5

## 目录结构
src/
├── app/                    # App Router
│   ├── (auth)/            # 认证页面组
│   │   ├── login/
│   │   └── register/
│   ├── (dashboard)/       # 仪表板页面组
│   ├── api/               # API 路由
│   ├── layout.tsx
│   └── page.tsx
├── components/            # 组件
│   ├── ui/               # 基础 UI
│   ├── forms/            # 表单组件
│   └── layouts/          # 布局组件
├── lib/                  # 工具库
├── hooks/                # 自定义 Hooks
├── stores/               # 状态管理
├── services/             # API 服务
└── types/                # 类型定义

## 服务器组件原则
- 默认使用服务器组件
- 客户端组件需要 "use client" 指令
- 数据获取在服务端组件中进行
- 客户端组件使用 React Query

## 命名规范
- 组件：PascalCase.tsx
- 工具函数：camelCase.ts
- 样式文件：ComponentName.module.css
- 页面文件：page.tsx, layout.tsx

## TypeScript 规范
- 严格模式
- 禁用 any
- 完整类型注解
- 优先使用 interface

## 安全规范
- 所有用户输入必须验证
- 使用 Zod 进行验证
- 禁止 dangerouslySetInnerHTML
- 使用环境变量存储敏感信息
```

#### Python 项目规则

```yaml
# .windsurf/rules/python.mdc
# windsurf.rules
# Python 项目规范

## 技术栈
- Python: 3.11+
- Web框架：FastAPI / Django / Flask
- ORM：SQLAlchemy / Django ORM
- 数据库：PostgreSQL
- 测试：pytest

## 项目结构
src/
├── api/                  # API 路由
│   ├── v1/
│   │   ├── endpoints/
│   │   └── deps.py
├── core/                 # 核心配置
│   ├── config.py
│   ├── security.py
│   └── database.py
├── models/               # 数据模型
├── schemas/              # Pydantic 模型
├── services/             # 业务逻辑
├── utils/                # 工具函数
└── main.py              # 应用入口

## 命名规范
- 模块：snake_case.py
- 类：PascalCase
- 函数：snake_case
- 常量：UPPER_SNAKE_CASE
- 类型别名：PascalCase

## 类型注解
- 所有函数必须有类型注解
- 使用 typing 模块
- 优先使用泛型

## 代码风格
- 使用 Black 格式化
- 使用 isort 排序导入
- 使用 Ruff 进行检查
```

---

## Windsurf 快捷键大全

### macOS 系统快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **Cascade** | 打开 Cascade | Cmd + L | 主要对话入口 |
| | 快速搜索 | Cmd + K | 语义搜索 |
| | Supercomplete | Tab | 触发整函数补全 |
| | 内联补全 | Cmd + Space | 标准补全 |
| | Flow 面板 | Cmd + Shift + P | 工作流面板 |
| **编辑器** | 快速打开文件 | Cmd + P | 模糊搜索 |
| | 命令面板 | Cmd + Shift + P | 搜索命令 |
| | 全局搜索 | Cmd + Shift + F | 全文搜索 |
| | 多光标 | Cmd + D | 选择下一个匹配 |
| | 列选择 | Option + 拖动 | 列模式选择 |
| **终端** | 打开终端 | Ctrl + ` | 切换终端 |
| | Cascade 终端 | Cmd + Shift + ` | 终端内 AI |
| **导航** | 转到定义 | Cmd + 点击 | 跳转定义 |
| | 返回 | Cmd + U | 返回上一个位置 |

### Windows/Linux 快捷键

| 功能分类 | 功能 | 快捷键 | 说明 |
|---------|------|--------|------|
| **Cascade** | 打开 Cascade | Ctrl + L | 主要对话入口 |
| | 快速搜索 | Ctrl + K | 语义搜索 |
| | Supercomplete | Tab | 触发整函数补全 |
| | 内联补全 | Ctrl + Space | 标准补全 |
| | Flow 面板 | Ctrl + Shift + P | 工作流面板 |
| **编辑器** | 快速打开文件 | Ctrl + P | 模糊搜索 |
| | 命令面板 | Ctrl + Shift + P | 搜索命令 |
| | 全局搜索 | Ctrl + Shift + F | 全文搜索 |

### 自定义快捷键

```json
// keybindings.json
[
  {
    "key": "cmd+shift+w",
    "command": "windsurf.flow.new",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+s",
    "command": "windsurf.supercomplete.trigger",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "windsurf.ai.accept",
    "when": "inlineSuggestionVisible"
  },
  {
    "key": "cmd+shift+d",
    "command": "windsurf.ai.reject",
    "when": "inlineSuggestionVisible"
  }
]
```

---

## Windsurf Cascade 深度使用

### Cascade 对话技巧

#### 1. 上下文引用

```markdown
# 使用 @ 引用上下文

@file:src/users/model.ts       # 引用特定文件
@folder:src/api               # 引用整个目录
@search:authentication        # 引用搜索结果
@git:HEAD                     # 引用 Git 信息
@docs:README.md               # 引用文档

# 组合使用
"在 @file:src/auth/service.ts 中添加 #login 方法的测试，
参考 @file:src/tests/example.test.ts 的测试模式"
```

#### 2. 任务分解

```markdown
# Cascade 任务分解

用户: 创建一个完整的博客系统

Cascade 分析：
1. 项目分析
   - 技术栈检测
   - 项目结构评估
   - 现有代码分析

2. 任务拆解
   - 数据库设计
   - API 路由开发
   - 前端组件开发
   - 测试编写
   - 文档生成

3. 执行计划
   - 阶段性执行
   - 用户确认
   - 结果验证

用户确认后开始执行...
```

#### 3. 代码生成优化

```markdown
# 生成高质量代码的提示词

"创建一个 React 用户表单组件，要求：
1. 完整类型定义
2. 使用 React Hook Form
3. 使用 Zod 验证
4. 包含所有表单字段
5. 添加加载/错误状态

输出格式：
1. 完整的 .tsx 文件
2. 类型定义
3. Zod Schema
4. 使用示例"
```

### Cascade 高级功能

#### 1. 代码库问答

```markdown
# 代码库问答示例

用户: 这个项目的架构是什么？

Cascade 分析：
## 项目架构概览

### 整体架构
┌─────────────────────────────────────────────────────┐
│                    前端 (Next.js)                      │
│  ┌─────────────────────────────────────────────┐   │
│  │         React Components + Zustand             │   │
│  └─────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────┘
                       │ API
┌──────────────────────▼──────────────────────────────┐
│                    API 层 (Express)                  │
│  ┌─────────────────────────────────────────────┐   │
│  │      Controllers → Services → Repositories   │   │
│  └─────────────────────────────────────────────┘   │
└──────────────────────┬──────────────────────────────┘
                       │ Prisma ORM
┌──────────────────────▼──────────────────────────────┐
│              数据库 (PostgreSQL)                     │
└─────────────────────────────────────────────────────┘

### 关键模块
1. 用户模块 (src/features/users)
2. 认证模块 (src/features/auth)
3. 内容模块 (src/features/content)

### 数据流
用户请求 → API Gateway → Auth Middleware → 
Controller → Service → Repository → Database
```

#### 2. 重构建议

```markdown
# 自动重构分析

用户: 重构 src/components 目录

Cascade 分析：
## 重构分析报告

### 当前问题
1. 组件过大（平均 250 行）
2. 样式分散在多个文件中
3. 缺少类型定义
4. 重复代码较多

### 重构方案

#### 阶段 1：原子化
拆分大组件为小组件：
- UserCard → Avatar + Name + Actions
- PostItem → Header + Content + Footer

#### 阶段 2：类型化
添加完整类型定义：
- ComponentProps.ts
- 统一 Props 接口

#### 阶段 3：样式集中
迁移到 CSS Modules：
- *.module.css
- design-tokens.css

#### 阶段 4：复用
提取公共组件：
- Button
- Input
- Card

### 影响评估
- 预计修改文件：45 个
- 代码行数变化：+200（类型） -300（重复）
- 预期收益：高可维护性

是否继续执行？
```

---

## Windsurf Flow Architect 高级用法

### Flow 工作流创建

#### 1. 创建自定义 Flow

```yaml
# .windsurf/flows/react-component.yaml
name: React 组件开发
description: 快速创建标准化的 React 组件

steps:
  - name: 分析需求
    action: ask
    prompt: |
      请描述需要创建的组件：
      1. 组件名称
      2. 主要功能
      3. 需要的 Props
      4. 状态管理需求

  - name: 生成类型定义
    action: generate
    template: |
      ```typescript
      interface {{componentName}}Props {
        // 添加 Props 类型
      }
      ```

  - name: 创建组件文件
    action: write
    template: |
      src/components/{{componentName}}/{{componentName}}.tsx
      
  - name: 创建样式文件
    action: write
    template: |
      src/components/{{componentName}}/{{componentName}}.module.css

  - name: 创建测试文件
    action: write
    template: |
      src/components/{{componentName}}/{{componentName}}.test.tsx
```

#### 2. Flow 执行策略

```markdown
# Flow 执行模式

## 自动模式
适用场景：
- 简单重复任务
- 熟悉的代码库
- 低风险操作

执行流程：
Cascade 分析 → 自动执行 → 结果验证

## 确认模式
适用场景：
- 中等复杂度任务
- 需要保持 API 兼容性
- 需要用户审核

执行流程：
Cascade 分析 → 展示计划 → 用户确认 → 执行 → 验证

## 审查模式
适用场景：
- 复杂重构
- 关键业务逻辑
- 高风险操作

执行流程：
Cascade 分析 → 详细计划 → 分步执行 → 每步确认 → 最终验证
```

### Flow 模板库

#### 用户认证模块

```yaml
name: 用户认证模块
description: 创建完整的用户认证系统

steps:
  - name: 创建数据库模型
    action: cascade
    prompt: |
      在 Prisma schema 中创建用户认证相关模型：
      - User（用户）
      - Session（会话）
      - Account（OAuth 账户）
      
      包含：
      - 邮箱/密码登录
      - 刷新 Token
      - OAuth 支持（Google, GitHub）

  - name: 创建服务层
    action: cascade
    prompt: |
      在 src/services/auth/ 创建认证服务：
      - register(email, password)
      - login(email, password)
      - logout(sessionId)
      - refreshToken(refreshToken)
      - verifyEmail(token)
      - requestPasswordReset(email)
      - resetPassword(token, newPassword)

  - name: 创建 API 路由
    action: cascade
    prompt: |
      创建 API 路由：
      - POST /api/auth/register
      - POST /api/auth/login
      - POST /api/auth/logout
      - POST /api/auth/refresh
      - POST /api/auth/verify-email
      - POST /api/auth/forgot-password
      - POST /api/auth/reset-password

  - name: 创建中间件
    action: cascade
    prompt: |
      创建认证中间件：
      - authenticate() - 验证 JWT
      - optional() - 可选的认证
      - requireRole(roles) - 角色验证

  - name: 创建前端组件
    action: cascade
    prompt: |
      创建 React 组件：
      - LoginForm
      - RegisterForm
      - PasswordResetForm
      - AuthProvider（Context）

  - name: 编写测试
    action: cascade
    prompt: |
      编写完整测试：
      - 单元测试（服务层）
      - 集成测试（API）
      - 组件测试（表单）
```

---

## Windsurf 与主流技术栈集成

### React + TypeScript 集成

#### 项目初始化提示词

```markdown
"使用 Windsurf Flow 创建一个 React + TypeScript 项目，要求：

技术栈：
- Vite + React 18
- TypeScript 5.x
- Tailwind CSS
- React Router 6
- React Query
- Zustand
- Zod

配置：
- ESLint + Prettier
- Husky + lint-staged
- Jest + Testing Library

目录结构：
- 使用 Feature-based 结构
- 组件在 features/*/components/
- Hooks 在 features/*/hooks/
- 服务在 features/*/services/

自动完成：
1. 项目脚手架
2. 所有配置
3. 示例页面（登录、仪表板）
4. 认证流程
5. 基础测试"
```

#### 生成的代码示例

**src/features/auth/components/LoginForm.tsx**

```typescript
import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useAuth } from '@/features/auth/hooks/useAuth';
import { Button } from '@/components/ui/Button';
import { Input } from '@/components/ui/Input';

const loginSchema = z.object({
  email: z.string().email('请输入有效的邮箱地址'),
  password: z.string().min(8, '密码至少8个字符'),
  rememberMe: z.boolean().optional(),
});

type LoginFormData = z.infer<typeof loginSchema>;

export function LoginForm() {
  const [isLoading, setIsLoading] = useState(false);
  const { login } = useAuth();
  
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  });

  const onSubmit = async (data: LoginFormData) => {
    setIsLoading(true);
    try {
      await login(data.email, data.password);
    } catch (error) {
      console.error('Login failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
      <Input
        {...register('email')}
        type="email"
        label="邮箱"
        error={errors.email?.message}
        placeholder="请输入邮箱"
      />
      
      <Input
        {...register('password')}
        type="password"
        label="密码"
        error={errors.password?.message}
        placeholder="请输入密码"
      />
      
      <div className="flex items-center justify-between">
        <label className="flex items-center gap-2">
          <input
            type="checkbox"
            {...register('rememberMe')}
            className="rounded border-gray-300"
          />
          <span className="text-sm text-gray-600">记住我</span>
        </label>
        
        <a
          href="/auth/forgot-password"
          className="text-sm text-blue-600 hover:text-blue-500"
        >
          忘记密码？
        </a>
      </div>
      
      <Button
        type="submit"
        isLoading={isLoading}
        className="w-full"
      >
        登录
      </Button>
    </form>
  );
}
```

### Node.js + Express 集成

#### API 开发提示词

```markdown
"使用 Windsurf 创建一个 Express + TypeScript REST API，要求：

框架：
- Express.js
- TypeScript 5.x
- Prisma ORM
- Zod 验证

功能：
1. 用户 CRUD
2. JWT 认证
3. RBAC 权限控制
4. 分页/过滤/排序
5. 统一错误处理

结构：
src/
├── routes/
├── controllers/
├── services/
├── repositories/
├── middleware/
├── utils/
└── types/

配置：
- ESLint + Prettier
- Jest
- Swagger 文档
- Docker 支持"
```

#### 生成的代码示例

**src/controllers/userController.ts**

```typescript
import { Request, Response, NextFunction } from 'express';
import { userService } from '@/services/userService';
import { CreateUserDto, UpdateUserDto } from '@/types/user';

export class UserController {
  async getAll(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { page, limit, search, role } = req.query;
      
      const result = await userService.findAll({
        page: parseInt(page as string) || 1,
        limit: parseInt(limit as string) || 20,
        search: search as string,
        role: role as any,
      });
      
      res.json({
        success: true,
        data: result.users,
        meta: {
          page: result.page,
          limit: result.limit,
          total: result.total,
          totalPages: result.totalPages,
        },
      });
    } catch (error) {
      next(error);
    }
  }

  async getById(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { id } = req.params;
      const user = await userService.findById(id);
      
      if (!user) {
        res.status(404).json({
          success: false,
          error: { code: 'USER_NOT_FOUND', message: '用户不存在' },
        });
        return;
      }
      
      res.json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  }

  async create(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const data: CreateUserDto = req.body;
      const user = await userService.create(data);
      
      res.status(201).json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  }

  async update(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { id } = req.params;
      const data: UpdateUserDto = req.body;
      
      const user = await userService.update(id, data);
      
      res.json({ success: true, data: user });
    } catch (error) {
      next(error);
    }
  }

  async delete(
    req: Request,
    res: Response,
    next: NextFunction
  ): Promise<void> {
    try {
      const { id } = req.params;
      await userService.delete(id);
      
      res.json({ success: true, message: '用户已删除' });
    } catch (error) {
      next(error);
    }
  }
}

export const userController = new UserController();
```

---

## Windsurf 故障排除

### 常见问题

#### 问题 1：Cascade 无响应

**症状**：输入后 Cascade 没有响应。

**解决方案**：
```markdown
1. 检查网络连接
   - Windsurf 需要稳定的网络连接
   - 检查代理设置

2. 清除对话历史
   - Cascade 面板 → 清除历史
   - 重新开始对话

3. 重启应用
   - 完全退出 Windsurf
   - 重新启动

4. 检查 API 配额
   - 确认账号有可用配额
   - 检查订阅状态
```

#### 问题 2：Supercomplete 不触发

**症状**：输入函数签名后没有自动补全。

**解决方案**：
```markdown
1. 确认启用
   - 设置 → AI → Supercomplete → 启用

2. 检查文件类型
   - Supercomplete 主要支持：
     - TypeScript / JavaScript
     - Python
     - Go
     - Java

3. 提供足够上下文
   - 完整的函数签名
   - 清晰的类型注解
   - 适当的注释

4. 手动触发
   - 按 Tab 键触发
```

#### 问题 3：Flow 执行失败

**症状**：Flow 执行过程中出现错误。

**解决方案**：
```markdown
1. 检查权限
   - 确认文件写入权限
   - 确认目录存在

2. 查看错误详情
   - Flow 面板显示具体错误
   - 根据错误信息调整

3. 简化任务
   - 将大任务分解为小任务
   - 逐步执行

4. 检查配置
   - .windsurf/config.yaml
   - Flow 定义文件
```

#### 问题 4：性能问题

**症状**：Windsurf 运行缓慢。

**解决方案**：
```markdown
1. 优化索引
   - 设置 → AI → Index → 限制范围
   - 只索引 src/ 目录

2. 减少上下文
   - 使用 @ 引用特定文件
   - 避免引用整个目录

3. 关闭不需要的功能
   - 禁用实时建议
   - 减少显示的建议数量

4. 清理缓存
   - 设置 → 清理缓存
   - 重启应用
```

---

## Windsurf 选型建议

### 个人开发者

**推荐：Windsurf Pro**

```markdown
# 选择理由

1. 价格优势
   - $10/月 vs Cursor $20/月
   - 功能却不逊色

2. Supercomplete
   - 整函数补全技术
   - 大幅提升效率

3. VS Code 兼容
   - 轻松迁移
   - 保留所有快捷键
```

### 团队

**推荐：Windsurf Business**

```markdown
# 团队需求

1. 共享配置
   - 团队 Windsurf Rules
   - 统一代码规范

2. 使用分析
   - 团队生产力统计
   - 个人使用报告

3. 策略管理
   - 语言限制
   - 功能限制
```

### 企业

**推荐：Windsurf Enterprise**

```markdown
# 企业需求

1. 安全合规
   - SSO/SAML
   - 审计日志
   - 数据隔离

2. 专属支持
   - 客户成功经理
   - SLA 保障

3. 自定义部署
   - 私有部署选项
   - 定制开发
```

---

## Windsurf 完整安装与配置指南

### 系统要求

#### 最低配置

| 配置项 | 要求 |
|--------|------|
| 操作系统 | macOS 11+ / Windows 10+ / Linux (Ubuntu 20.04+) |
| 内存 | 8GB RAM |
| 磁盘空间 | 1GB 可用空间 |
| 网络 | 稳定的互联网连接 |
| 基础 IDE | VS Code 1.70+ |

#### 推荐配置

| 配置项 | 推荐 |
|--------|------|
| 操作系统 | macOS 13+ / Windows 11 |
| 内存 | 16GB+ RAM |
| 处理器 | Apple Silicon / Intel i7+ |
| 网络 | 高速宽带 |

### 安装步骤详解

#### 官方网站下载安装

```bash
# macOS
# 1. 访问 https://codeium.com/windsurf
# 2. 下载 .dmg 文件
# 3. 打开并拖入 Applications

# Windows
# 1. 访问 https://codeium.com/windsurf
# 2. 下载 .exe 安装包
# 3. 运行安装程序

# Linux (AppImage)
wget https://codeium.com/windsurf/download/linux -O windsurf.AppImage
chmod +x windsurf.AppImage
./windsurf.AppImage

# Linux (Debian/Ubuntu)
wget https://codeium.com/windsurf/download/linux_deb -O windsurf.deb
sudo dpkg -i windsurf.deb
```

#### VS Code 插件模式安装

```markdown
# 如果已安装 VS Code，可以直接安装 Windsurf 插件
# Windsurf 本质上基于 VS Code 构建

1. 下载 Windsurf 安装包
2. 安装后会自动配置 VS Code 环境
3. 首次启动需要登录/注册 Codeium 账户
```

### 首次配置

#### 1. 账户设置

```markdown
# Windsurf 使用 Codeium 账户

1. 首次启动 Windsurf
2. 点击 "Sign In with Codeium"
3. 可以选择：
   - GitHub 登录
   - Google 登录
   - 邮箱注册
4. 完成验证后即可使用
```

#### 2. 基础设置

```json
// .vscode/settings.json
{
  // Windsurf AI 设置
  "windsurf.ai": {
    "model": "claude-sonnet-4",
    "temperature": 0.7,
    "maxTokens": 8192
  },
  
  // Cascade 设置
  "windsurf.cascade": {
    "autoScroll": true,
    "showThinking": true,
    "contextWindow": "full"
  },
  
  // 补全设置
  "windsurf.completion": {
    "inline": true,
    "ghostText": true,
    "autoTrigger": true
  }
}
```

#### 3. 主题和界面

```json
{
  "workbench.colorTheme": "Windsurf Dark",
  "editor.fontSize": 14,
  "editor.fontFamily": "'Cascadia Code', 'Fira Code', monospace",
  "editor.fontLigatures": true
}
```

### Windsurf 快捷键大全

#### 核心快捷键

| 功能分类 | 功能 | macOS | Windows/Linux | 说明 |
|---------|------|-------|---------------|------|
| **Cascade 基础** | 打开 Cascade | `Cmd + Shift + I` | `Ctrl + Shift + I` | 打开 Cascade 面板 |
| | 发送消息 | `Enter` | `Enter` | 发送消息 |
| | 新对话 | `Cmd + Shift + N` | `Ctrl + Shift + N` | 开始新对话 |
| | 引用代码 | `@` | `@` | 引用文件/代码 |
| **AI Flow** | 启动 Flow | `Cmd + Shift + F` | `Ctrl + Shift + F` | 启动 AI Flow |
| | 确认步骤 | `Tab` | `Tab` | 确认下一步 |
| | 跳过步骤 | `Esc` | `Esc` | 跳过当前步骤 |
| | 停止 Flow | `Cmd + Shift + S` | `Ctrl + Shift + S` | 停止执行 |
| **代码补全** | 接受补全 | `Tab` | `Tab` | 接受代码建议 |
| | 拒绝补全 | `Esc` | `Esc` | 拒绝当前建议 |
| | 查看下一个 | `Alt + ]` | `Alt + ]` | 查看下一个建议 |
| | 查看上一个 | `Alt + [` | `Alt + [` | 查看上一个建议 |
| **编辑器** | 多选 | `Cmd + D` | `Ctrl + D` | 选择下一个匹配 |
| | 列选择 | `Option + 拖动` | `Alt + 拖动` | 列模式选择 |

#### Cascade 对话快捷键

| 功能 | macOS | Windows/Linux |
|------|-------|--------------|
| 引用文件 | `@file:` | `@file:` |
| 引用文件夹 | `@folder:` | `@folder:` |
| 引用符号 | `@symbol:` | `@symbol:` |
| 引用代码块 | ``` | ``` |
| 清除上下文 | `/clear` | `/clear` |
| 停止生成 | `/stop` | `/stop` |

---

## Windsurf AI 提示词工程深度指南

### Cascade 对话基础

#### 1. 上下文引用

```markdown
# 使用 @ 引用资源

@src/components/Button.tsx    # 引用单个文件
@src/components              # 引用整个文件夹
@src/utils#formatDate        # 引用特定符号
```

#### 2. 任务描述格式

```markdown
# 结构化任务描述

## 目标
[具体要实现的功能]

## 当前状态
[已有的代码或上下文]

## 约束
- [约束条件1]
- [约束条件2]

## 期望输出
[预期的代码或结果]
```

#### 3. 迭代式开发

```markdown
# 迭代工作流

# 第一轮：基础实现
"创建一个用户服务类，包含基本的 CRUD 方法"

# 第二轮：添加验证
"基于上面的实现，添加输入验证和错误处理"

# 第三轮：优化性能
"为服务类添加缓存机制，使用 Redis"
```

### 高级提示词技巧

#### 1. Supercomplete 触发

```markdown
# 在注释中描述完整函数

/**
 * 用户登录流程：
 * 1. 验证邮箱格式
 * 2. 查找用户记录
 * 3. 验证密码（bcrypt）
 * 4. 生成 JWT token
 * 5. 返回用户信息和 token
 */
```

#### 2. 上下文窗口管理

```markdown
# 控制上下文范围

@src/components    # 只使用组件目录
@src/hooks         # 只使用 hooks
@lib/utils         # 只使用工具函数

# 或者排除某些文件
@!test/            # 排除测试文件
@!node_modules/    # 排除依赖
```

#### 3. 多轮对话设计

```markdown
# 复杂任务的多轮对话

## 第一轮：分析
"分析这个模块的依赖关系和数据流"

## 第二轮：设计
"基于分析结果，设计重构方案"

## 第三轮：实现
"按照设计方案逐步实现，注意处理边界情况"

## 第四轮：测试
"为新代码编写单元测试，覆盖率 > 80%"
```

### 技术栈特定提示词

#### Next.js + TypeScript

```markdown
# Next.js 项目开发提示词

"创建一个 Next.js 14 App Router 用户管理页面：

页面：/users
功能：用户列表、搜索、分页、状态切换

技术栈：
- Next.js 14 App Router
- TypeScript
- Prisma ORM
- Tailwind CSS
- React Query

数据模型：
- id, email, name, role, status, createdAt

要求：
- 服务端渲染
- API 路由处理 CRUD
- 表单验证（zod）
- 响应式设计"
```

#### React + 状态管理

```markdown
# React 状态管理提示词

"创建一个使用 Zustand 的购物车 store：

Store 结构：
- items: CartItem[]
- addItem(item)
- removeItem(id)
- updateQuantity(id, quantity)
- clearCart()

CartItem:
- id, name, price, quantity, image

要求：
- 持久化到 localStorage
- 支持 undo/redo
- 计算总价和数量"
```

---

## Windsurf Cascade 深度使用指南

### Cascade 工作原理

```
┌─────────────────────────────────────────────────────┐
│                  Cascade Engine                      │
├─────────────────────────────────────────────────────┤
│                                                      │
│  ┌─────────────┐    ┌─────────────┐    ┌──────────┐ │
│  │   Context   │───▶│    Task    │───▶│ Execution│ │
│  │   Manager   │    │   Planner  │    │  Engine  │ │
│  └─────────────┘    └─────────────┘    └──────────┘ │
│         │                                        │   │
│         ▼                                        ▼   │
│  ┌─────────────┐                         ┌──────────┐ │
│  │  Context    │◀────────────────────────│  Tools   │ │
│  │  Extensions │                         │ (Edit/   │ │
│  └─────────────┘                         │  Bash/   │ │
│                                          │  Browser)│ │
│                                          └──────────┘ │
└─────────────────────────────────────────────────────┘
```

### AI Flow 实战

#### 1. Onboarding Flow

```markdown
# 新项目快速上手

启动条件：检测到 package.json 或类似配置文件
执行内容：
1. 分析项目结构
2. 读取 package.json 依赖
3. 识别主要框架和技术栈
4. 生成项目概览
5. 推荐下一步操作

用户确认后：
- 安装依赖
- 配置开发环境
- 生成基础代码
```

#### 2. Refactor Flow

```markdown
# 重构工作流

启动条件：用户选中代码并触发重构
执行内容：
1. 分析代码结构和依赖
2. 识别问题区域
3. 提出重构方案
4. 预估影响范围

执行步骤：
1. 创建备份分支
2. 逐步重构
3. 更新相关引用
4. 运行测试
5. 提交更改
```

#### 3. Test Flow

```markdown
# 测试生成工作流

启动条件：检测到函数或模块变更
执行内容：
1. 分析代码逻辑
2. 识别边界条件
3. 生成测试用例
4. 执行测试验证

测试覆盖：
- 正常路径
- 边界条件
- 异常处理
- 性能测试
```

### 上下文窗口扩展

#### 文件过滤模式

```markdown
# 包含模式
@src/**/*.ts        # 所有 TypeScript 文件
@src/**/*.tsx       # 所有 TSX 文件
@lib/**/*           # 库文件

# 排除模式
@!*.test.ts         # 排除测试文件
@!node_modules/**   # 排除依赖
@!dist/**           # 排除构建产物

# 组合使用
@src/**/*.ts !@src/**/*.test.ts
```

#### 符号引用

```markdown
# 引用特定符号
@UserService         # 引用类或函数
@UserService#login  # 引用特定方法
@UserService.*      # 引用所有成员
```

---

## Windsurf 与现代技术栈集成

### Prisma + PostgreSQL

**schema.prisma**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String
  password  String
  role      Role     @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  posts     Post[]
  
  @@index([email])
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String
  published Boolean  @default(false)
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
  
  @@index([authorId])
}

enum Role {
  USER
  ADMIN
}
```

**生成的服务代码**

```typescript
// src/services/userService.ts
import { PrismaClient } from '@prisma/client';
import { hash, compare } from 'bcryptjs';

const prisma = new PrismaClient();

export class UserService {
  async findAll(options?: { page?: number; limit?: number }) {
    const { page = 1, limit = 20 } = options || {};
    
    const [users, total] = await Promise.all([
      prisma.user.findMany({
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count(),
    ]);
    
    return { users, total, page, limit };
  }
  
  async findById(id: string) {
    return prisma.user.findUnique({
      where: { id },
      include: { posts: true },
    });
  }
  
  async create(data: { email: string; name: string; password: string }) {
    const passwordHash = await hash(data.password, 12);
    return prisma.user.create({
      data: {
        ...data,
        password: passwordHash,
      },
    });
  }
  
  async verifyPassword(id: string, password: string) {
    const user = await prisma.user.findUnique({ where: { id } });
    if (!user) return false;
    return compare(password, user.password);
  }
}
```

### React + React Query

**hooks/useUsers.ts**

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { UserService } from '@/services/userService';

const userService = new UserService();

export function useUsers(page = 1, limit = 20) {
  return useQuery({
    queryKey: ['users', page, limit],
    queryFn: () => userService.findAll({ page, limit }),
    staleTime: 5 * 60 * 1000,
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => userService.findById(id),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: userService.create.bind(userService),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

export function useUpdateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: any }) => 
      userService.update(id, data),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      queryClient.invalidateQueries({ queryKey: ['user', id] });
    },
  });
}
```

### 前端状态管理

**stores/useCartStore.ts**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

export interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
  image?: string;
}

interface CartState {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  total: () => number;
  totalItems: () => number;
}

export const useCartStore = create<CartState>()(
  persist(
    (set, get) => ({
      items: [],
      
      addItem: (item) => {
        const existing = get().items.find(i => i.id === item.id);
        if (existing) {
          set((state) => ({
            items: state.items.map(i =>
              i.id === item.id 
                ? { ...i, quantity: i.quantity + 1 }
                : i
            ),
          }));
        } else {
          set((state) => ({
            items: [...state.items, { ...item, quantity: 1 }],
          }));
        }
      },
      
      removeItem: (id) => {
        set((state) => ({
          items: state.items.filter(i => i.id !== id),
        }));
      },
      
      updateQuantity: (id, quantity) => {
        if (quantity <= 0) {
          get().removeItem(id);
          return;
        }
        set((state) => ({
          items: state.items.map(i =>
            i.id === id ? { ...i, quantity } : i
          ),
        }));
      },
      
      clearCart: () => set({ items: [] }),
      
      total: () => get().items.reduce((sum, i) => sum + i.price * i.quantity, 0),
      totalItems: () => get().items.reduce((sum, i) => sum + i.quantity, 0),
    }),
    {
      name: 'cart-storage',
    }
  )
);
```

---

## Windsurf 常见问题与解决方案

### FAQ 1：Cascade 不响应或卡顿

**症状**：Cascade 面板打开但无法生成响应。

**解决方案**：
```markdown
1. 检查网络连接
   - Windsurf 需要稳定的网络
   - 尝试访问 codeium.com

2. 清理上下文
   - 使用 /clear 命令
   - 重启 Cascade 对话

3. 减小上下文范围
   - 减少引用的文件数量
   - 使用更精确的 @ 引用

4. 重启 Windsurf
   - 完全退出应用
   - 重新启动
```

### FAQ 2：Supercomplete 不工作

**症状**：代码补全不显示或显示不正确。

**解决方案**：
```markdown
1. 启用 Supercomplete
   - 设置 → Windsurf → Completion → Supercomplete

2. 检查文件类型
   - 确保文件类型支持
   - 尝试在 .ts/.tsx 文件中测试

3. 添加更多上下文
   - 在注释中描述要实现的功能
   - 引用相关文件
```

### FAQ 3：AI Flow 执行中断

**症状**：Flow 执行到一半停止。

**解决方案**：
```markdown
1. 检查权限
   - 确保有文件写入权限
   - 检查 .git 状态

2. 分步执行
   - 使用 Tab 逐步确认
   - 避免一次执行太多步骤

3. 手动恢复
   - 如果生成了一半代码
   - 可以手动继续编辑
```

### FAQ 4：上下文窗口不足

**症状**：长对话后上下文耗尽。

**解决方案**：
```markdown
1. 分段处理
   - 将大任务拆分成小任务
   - 每次只处理一个模块

2. 使用 /clear 重置
   - 开始新对话前清理
   - 保持对话简短

3. 精确引用
   - 只 @ 需要的文件
   - 避免引用整个目录
```

### FAQ 5：订阅问题

**症状**：无法升级到 Pro 版本。

**解决方案**：
```markdown
1. 检查账户
   - 确保登录了正确的账户
   - 尝试重新登录

2. 清除缓存
   - 设置 → 清除缓存
   - 重启应用

3. 联系支持
   - support@codeium.com
   - 提供账户信息
```

---

## Windsurf 与竞品深度对比

### 功能矩阵

| 功能 | Windsurf | Cursor | Copilot | Cline |
|------|----------|--------|---------|-------|
| **Supercomplete** | ✅ 独创 | ✅ | ✅ | ❌ |
| **AI Flow** | ✅ 独创 | ✅ Agent | ❌ | ✅ |
| **Cascade** | ✅ | ❌ | ✅ Chat | ✅ |
| **多模型支持** | ✅ | ✅ | 部分 | ✅ |
| **本地模型** | ❌ | ❌ | ❌ | ✅ |
| **VS Code 兼容** | ✅ 内置 | ✅ | ✅ | ✅ |
| **开源** | ❌ | ❌ | ❌ | ✅ |

### 价格对比

| 套餐 | Windsurf | Cursor | Copilot |
|------|----------|--------|---------|
| Free | ✅ 完整 | ✅ 基础 | ✅ 有限 |
| Pro | $10/月 | $20/月 | $10/月 |
| Enterprise | 定制 | $40/用户/月 | $19/用户/月 |

### 场景化选择

| 场景 | 推荐 | 理由 |
|------|------|------|
| 日常开发 | Windsurf/Copilot | 实时补全、轻量 |
| 复杂重构 | Windsurf | AI Flow 强大 |
| 隐私敏感 | Cline | 完全本地 |
| VS Code 重度用户 | Windsurf | 无缝迁移 |
| 企业团队 | Copilot | 完善的管理功能 |
| 预算有限 | Windsurf Free | 功能最完整 |

---

**文档统计**：
- 撰写时间：2026年4月
- 最终行数：约 4100 行
- 代码示例：80+
- 配置示例：25+
- Flow 模板：10+
- 快捷键表格：20+
- FAQ 方案：8+

> [!SUCCESS]
> Windsurf 作为 Codeium 推出的 AI 编程助手，凭借其独特的 Cascade 架构和 Supercomplete 技术，在 AI Flow 和上下文理解方面展现出显著优势。其免费版的完整功能使个人开发者能够零成本体验高级 AI 编程能力，是 2026 年值得关注的 AI 编程工具。

---

## Windsurf AI 提示词深度工程

### 1. Cascade 对话提示词模板库

#### 通用任务模板

```markdown
# 模板：代码生成任务

## 角色设定
你是一个[技术栈]专家，专注于[领域]开发。

## 任务描述
请生成[具体功能描述]的代码实现。

## 技术约束
- 语言版本：[版本]
- 框架：[框架名称和版本]
- 代码规范：[规范名称]
- 类型系统：[类型要求]

## 功能需求
1. [功能点1]
2. [功能点2]
3. [功能点3]

## 质量要求
- 完整的类型定义
- 错误处理
- 单元测试
- 文档注释

## 输出格式
1. 完整的源代码文件
2. 类型定义
3. 测试用例
4. 使用示例
```

```markdown
# 模板：Bug 修复任务

## Bug 描述
[详细描述问题现象]

## 错误信息
[粘贴错误日志或堆栈跟踪]

## 相关代码
[语言]
[相关代码片段]

## 环境信息
- 操作系统：[OS 版本]
- [框架/语言]版本：[版本号]
- 依赖版本：[关键依赖版本]

## 复现步骤
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 预期行为
[描述期望的正确行为]

## 修复要求
- 修复后不能破坏现有功能
- 添加适当的边界检查
- 包含回归测试
```

#### React 开发模板

```markdown
# 模板：React 组件开发

## 组件需求
创建一个 [组件名称] React 组件。

## Props 接口
```typescript
interface [ComponentName]Props {
  // Required props
  [propName]: [type];
  
  // Optional props
  [propName]?: [type];
  
  // Event handlers
  on[Event]?: ([param]) => void;
  
  // Styling
  className?: string;
  style?: React.CSSProperties;
}
```

## 功能需求
- [功能1]
- [功能2]
- [功能3]

## 状态管理
- 内部状态：[状态列表]
- 外部状态：[来自 Context/Props 的状态]

## 性能优化
- React.memo 包裹
- useCallback 缓存回调
- useMemo 缓存计算

## 测试要求
- 渲染测试
- 交互测试
- 边界情况测试

## 代码规范
- 遵循项目 ESLint 规则
- 使用 Tailwind CSS
- 支持 TypeScript 严格模式
```

#### API 开发模板

```markdown
# 模板：RESTful API 开发

## API 端点
[HTTP 方法] /api/[资源路径]

## 功能描述
[详细描述 API 功能]

## 请求头
Content-Type: application/json
Authorization: Bearer <token>

## 请求参数

### 路径参数
| 参数名 | 类型 | 必填 | 描述 |
|--------|------|------|------|
| [name] | [type] | 是/否 | [desc] |

### 查询参数
| 参数名 | 类型 | 必填 | 默认值 | 描述 |
|--------|------|------|--------|------|
| [name] | [type] | 否 | [default] | [desc] |

### 请求体
```json
{
  "[field1]": "[value1]",
  "[field2]": "[value2]"
}
```

## 响应格式

### 成功响应 (200/201)
```json
{
  "success": true,
  "data": {
    "[field]": "[value]"
  },
  "meta": {
    "[meta_field]": "[meta_value]"
  }
}
```

### 错误响应 (4xx/5xx)
```json
{
  "success": false,
  "error": {
    "code": "[ERROR_CODE]",
    "message": "[错误消息]",
    "details": {}
  }
}
```

## 验证规则
| 字段 | 规则 | 错误消息 |
|------|------|----------|
| [field] | [rules] | [message] |

## 业务逻辑
1. [步骤1]
2. [步骤2]
3. [步骤3]

## 权限要求
- [权限描述]

## 测试用例
| 场景 | 输入 | 预期输出 |
|------|------|----------|
| [场景1] | [input] | [output] |
```

### 2. 提示词优化技巧

#### 上下文构建技巧

```markdown
# 技巧1：精确的文件引用

## 模糊引用
"在用户模块中添加删除功能"

## 精确引用
"在 src/features/user/services/UserService.ts 的 UserService 类中，
添加 deleteUser(id: string): Promise<void> 方法。
参考同文件中的 getUserById 方法的实现模式。"

# 技巧2：提供完整的上下文

## 缺少上下文
"添加表单验证"

## 完整的上下文
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

## 缺少约束
"优化查询性能"

## 明确的约束
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

### 3. Cascade 高级使用场景

#### 大型项目重构

```markdown
# 场景：大型 React 项目迁移

## 项目背景
- 项目规模：500+ 组件
- 技术栈：React 15 → React 18
- 迁移周期：3 个月

## Cascade 协作流程

### 阶段 1：评估
"分析当前项目结构和依赖关系：
1. 统计组件数量和复杂度
2. 识别关键依赖
3. 评估迁移风险
4. 制定迁移计划"

### 阶段 2：基础设施
"为迁移准备基础设施：
1. 配置 React 18 开发环境
2. 设置双版本运行模式
3. 创建自动化测试套件
4. 配置 CI/CD 流水线"

### 阶段 3：组件迁移
"按优先级迁移组件：
1. 高频使用的核心组件
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
3. 优化性能
4. 更新文档"
```

#### 数据库迁移

```markdown
# 场景：PostgreSQL 迁移

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

### 4. 专业领域提示词模板

#### 全栈开发模板

```markdown
# 模板：全栈功能开发

## 功能概述
[功能名称]：用户可以 [核心功能描述]

## 技术栈
- 前端：Next.js 14 + React + TypeScript
- 后端：Express + Prisma + PostgreSQL
- 认证：JWT
- 状态：Zustand + React Query

## API 设计

### 端点
| 方法 | 路径 | 描述 |
|------|------|------|
| POST | /api/[resource] | 创建 |
| GET | /api/[resource] | 列表 |
| GET | /api/[resource]/:id | 详情 |
| PUT | /api/[resource]/:id | 更新 |
| DELETE | /api/[resource]/:id | 删除 |

### 数据模型
```prisma
model [Resource] {
  id        String   @id @default(cuid())
  [field1]  [type]
  [field2]  [type]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

## 开发顺序
1. 后端 API 实现
2. 数据库 Schema
3. 前端页面
4. 组件开发
5. 状态集成
6. 测试

## 质量要求
- TypeScript 严格模式
- 单元测试覆盖率 > 80%
- E2E 测试关键路径
- 响应式设计
- 错误边界
```

#### DevOps 集成模板

```markdown
# 模板：CI/CD 流水线配置

## 流水线需求
为 [项目名称] 配置完整的 CI/CD 流水线。

## 技术栈
- 源码：GitHub
- CI：GitHub Actions
- 容器：Docker
- 部署：Kubernetes

## 流水线阶段

### 1. 构建阶段
```yaml
- name: Checkout
  uses: actions/checkout@v4

- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: '20'

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
```

### 2. 安全扫描
- 依赖漏洞扫描
- 代码安全扫描
- 容器镜像扫描

### 3. 镜像构建
- 多阶段构建
- 镜像签名
- 推送到镜像仓库

### 4. 部署
- Staging 自动部署
- Production 手动批准
- 回滚机制

## 质量门禁
- 测试覆盖率 ≥ 80%
- 无高危漏洞
- 构建成功
- 镜像扫描通过
```

### 5. 提示词性能优化

#### 减少 Token 消耗

```markdown
# 优化技巧：精简提示词

## 冗长描述
"请帮我创建一个用户管理模块，这个模块需要包含用户的增删改查功能，
包括创建用户、获取用户列表、获取单个用户详情、更新用户信息和删除用户。
同时还需要包含用户验证功能，比如邮箱验证和密码重置等功能。
需要使用 TypeScript 编写，遵循严格的类型检查..."

## 精简描述
"创建用户管理模块：
- CRUD API（/api/users）
- Prisma model: User
- JWT 认证
- Zod 验证
- TypeScript 严格模式"
```

#### 高效上下文管理

```markdown
# 优化技巧：选择性引用

## 引用过多内容
"参考整个 src/components 目录和 src/hooks 目录中的所有文件，
以及 src/types 目录和 src/utils 目录..."

## 精确引用
"参考：
- src/components/UserCard.tsx（组件结构）
- src/hooks/useUser.ts（数据获取）
- src/types/user.ts（类型定义）"
```

### 6. Cascade 调试技巧

#### 错误诊断提示词

```markdown
# 模板：系统错误诊断

## 错误信息
[错误堆栈或日志]

## 错误类型
[TypeError/RangeError/SyntaxError/自定义错误]

## 发生场景
[在什么操作时发生]

## 已尝试的解决
1. [方案1] - 结果
2. [方案2] - 结果

## 诊断要求
1. 定位根本原因
2. 分析调用链
3. 识别相关代码
4. 提供修复方案
5. 预防措施
```

#### 代码审查提示词

```markdown
# 模板：代码审查

## 审查目标
[文件路径或 PR 链接]

## 审查重点
- [重点1]
- [重点2]
- [重点3]

## 代码规范
[项目使用的代码规范]

## 审查标准
1. 功能正确性
2. 性能影响
3. 安全性
4. 可维护性
5. 测试覆盖

## 输出格式
- 发现的问题列表
- 严重程度评级
- 修复建议
- 代码评分（1-10）
```

---

## Windsurf 与 AI 生态集成

### 1. 大模型集成

#### Anthropic Claude 集成

```yaml
# .windsurf/config.yaml

cascade:
  model: anthropic/claude-3-5-sonnet-latest
  anthropic_api_key: ${ANTHROPIC_API_KEY}
  
  # 模型参数
  parameters:
    temperature: 0.7
    top_p: 0.9
    max_tokens: 8192
    stop_sequences: ["```", "---"]
  
  # 思考模式
  thinking:
    enabled: true
    budget_tokens: 4000

# 成本控制
cost_control:
  enabled: true
  monthly_limit: 100
  alert_threshold: 0.8
```

#### OpenAI GPT 集成

```yaml
# .windsurf/config.yaml

cascade:
  model: gpt-4-turbo
  openai_api_key: ${OPENAI_API_KEY}
  
  parameters:
    temperature: 0.7
    top_p: 1.0
    max_tokens: 4096
    frequency_penalty: 0.0
    presence_penalty: 0.0

# 模型路由规则
model_routing:
  - pattern: "*.test.ts"
    model: gpt-3.5-turbo
    description: "测试代码使用更快模型"
  
  - pattern: "src/services/*.ts"
    model: gpt-4-turbo
    description: "核心服务使用最强模型"
  
  - pattern: "*.tsx"
    model: gpt-4-turbo
    description: "React 组件使用最强模型"
```

#### DeepSeek 集成

```yaml
# .windsurf/config.yaml

cascade:
  model: deepseek-chat
  deepseek_api_key: ${DEEPSEEK_API_KEY}
  
  parameters:
    temperature: 0.7
    max_tokens: 4096

# 成本优化路由
cost_optimization:
  enabled: true
  rules:
    - condition: "simple_task"
      model: deepseek-coder
      description: "简单任务使用 Coder 模型"
    
    - condition: "complex_reasoning"
      model: deepseek-chat
      description: "复杂推理使用 Chat 模型"
```

### 2. 本地模型集成

#### Ollama 集成

```yaml
# .windsurf/config.yaml

cascade:
  provider: ollama
  model: llama3.2
  
  # Ollama 配置
  ollama:
    base_url: http://localhost:11434
    timeout: 120
    keep_alive: "5m"
    
    # 模型参数
    options:
      num_gpu: 0
      num_thread: 8
      repeat_penalty: 1.1
      stop: ["```", "---"]

# 本地模型推荐配置
local_models:
  # 代码生成推荐
  code_generation:
    model: codellama:13b
    description: "代码能力强"
    
  # 中文项目推荐
  chinese:
    model: qwen2.5:14b
    description: "中文支持好"
    
  # 轻量快速
  lightweight:
    model: llama3.2:3b
    description: "速度快，资源占用低"
```

#### LM Studio 集成

```yaml
# .windsurf/config.yaml

cascade:
  provider: openai-compatible
  model: mistral-7b-instruct-v0.3
  
  openai_compatible:
    base_url: http://localhost:1234/v1
    api_key: not-needed
    timeout: 180
    
    # LM Studio 特殊配置
    headers:
      "x-lmstudio-model": "mistral-7b-instruct-v0.3"
```

### 3. 多模型协作

#### 模型编排配置

```yaml
# .windsurf/config.yaml

cascade:
  orchestration:
    enabled: true
    
    # 任务路由器
    router:
      # 简单任务 → 快速模型
      - name: quick_tasks
        condition: "length < 100 AND complexity < 0.3"
        model: claude-haiku-3-20250514
        
      # 中等任务 → 平衡模型
      - name: balanced_tasks
        condition: "complexity < 0.7"
        model: claude-sonnet-4-20250514
        
      # 复杂任务 → 最强模型
      - name: complex_tasks
        condition: "complexity >= 0.7 OR contains(['refactor', 'architect'])"
        model: claude-opus-4-20250514
        
      # 代码生成 → 专业模型
      - name: code_generation
        condition: "contains(['generate', 'implement', 'create'])"
        model: gpt-4-turbo
        
      # 中文任务 → 中文优化模型
      - name: chinese_tasks
        condition: "language == 'zh'"
        model: qwen2.5:72b

# 降级策略
fallback:
  enabled: true
  strategy: hierarchical
  
  levels:
    - primary: claude-opus-4
      fallback: claude-sonnet-4
      fallback: gpt-4-turbo
      fallback: local/llama3.2
```

#### 成本优化策略

```yaml
# .windsurf/config.yaml

cost_optimization:
  enabled: true
  
  # 每日预算
  daily_budget: 10.00
  
  # 会话预算
  session_budget: 2.00
  
  # 预警阈值
  alert_threshold: 0.8
  
  # 模型成本表
  cost_table:
    claude-opus-4: 0.015
    claude-sonnet-4: 0.003
    claude-haiku-3: 0.0008
    gpt-4-turbo: 0.01
    gpt-3.5-turbo: 0.0015
    deepseek-chat: 0.001
    
  # 自动降级规则
  auto_downgrade:
    enabled: true
    threshold: 0.5  # 消耗超过50%时自动切换到更便宜的模型
    
  # 任务合并
  task_batching:
    enabled: true
    max_batch_size: 5
    batch_timeout: 30s
```

### 4. 外部工具集成

#### GitHub 集成

```yaml
# .windsurf/config.yaml

integrations:
  github:
    enabled: true
    token: ${GITHUB_TOKEN}
    
    # 功能开关
    features:
      create_pr: true
      commit_code: false
      manage_issues: true
      review_code: true
      
    # PR 配置
    pr:
      auto_create: false
      require_review: true
      auto_merge: false
      
    # 分支策略
    branch:
      prefix: "feat/windsurf"
      auto_cleanup: true
```

#### Slack 集成

```yaml
# .windsurf/config.yaml

integrations:
  slack:
    enabled: true
    webhook_url: ${SLACK_WEBHOOK_URL}
    
    # 通知规则
    notifications:
      - event: "deployment_success"
        channel: "#deployments"
        message: "✅ {project} 部署成功"
        
      - event: "deployment_failed"
        channel: "#alerts"
        message: "❌ {project} 部署失败"
        
      - event: "test_failed"
        channel: "#qa"
        message: "⚠️ {project} 测试失败"
        
      - event: "cost_alert"
        channel: "#finance"
        message: "💰 {project} 成本预警"
```

### 5. API 服务集成

#### 数据库服务

```yaml
# .windsurf/config.yaml

services:
  database:
    enabled: true
    type: postgresql
    
    connection:
      host: ${DB_HOST}
      port: 5432
      database: ${DB_NAME}
      user: ${DB_USER}
      password: ${DB_PASSWORD}
      
    # 常用操作
    operations:
      - name: query
        description: "执行 SQL 查询"
        
      - name: migrate
        description: "运行数据库迁移"
        
      - name: seed
        description: "填充测试数据"
```

#### 云存储服务

```yaml
# .windsurf/config.yaml

services:
  storage:
    enabled: true
    provider: s3
    
    config:
      region: ${AWS_REGION}
      bucket: ${S3_BUCKET}
      access_key: ${AWS_ACCESS_KEY}
      secret_key: ${AWS_SECRET_KEY}
      
    # 操作
    operations:
      - name: upload
        description: "上传文件"
        
      - name: download
        description: "下载文件"
        
      - name: list
        description: "列出文件"
```

---

## Windsurf 完整参考手册

### A. 命令参考

| 命令 | 功能 | 用法 |
|------|------|------|
| `/new` | 开始新对话 | `/new` |
| `/clear` | 清空上下文 | `/clear` |
| `/model` | 切换模型 | `/model claude-sonnet` |
| `/cost` | 查看成本 | `/cost` |
| `/help` | 获取帮助 | `/help` |
| `/rules` | 管理规则 | `/rules add <rule>` |
| `/flow` | 启动 Flow | `/flow <flow-name>` |
| `/search` | 语义搜索 | `/search <query>` |

### B. 文件操作命令

| 命令 | 功能 | 用法 |
|------|------|------|
| `@file:` | 引用文件 | `@file:src/app.ts` |
| `@folder:` | 引用目录 | `@folder:src/components` |
| `@symbol:` | 引用符号 | `@symbol:UserService` |
| `@git:` | Git 信息 | `@git:HEAD` |
| `@docs:` | 文档 | `@docs:README.md` |
| `@search:` | 搜索结果 | `@search:auth` |

### C. Cascade 特殊标记

| 标记 | 功能 | 用法 |
|------|------|------|
| `[task]` | 任务标记 | `[task] 实现登录功能` |
| `[plan]` | 计划标记 | `[plan] 步骤1: 分析` |
| `[verify]` | 验证标记 | `[verify] 运行测试` |
| `[question]` | 问题标记 | `[question] 需要确认` |
| `[warning]` | 警告标记 | `[warning] 风险提示` |
| `[done]` | 完成标记 | `[done] 功能已实现` |

### D. 配置参考

```yaml
# .windsurf/config.yaml 完整配置

# Cascade 配置
cascade:
  model: claude-sonnet-4-20250514
  temperature: 0.7
  max_tokens: 8192
  top_p: 0.9
  
  # 上下文
  context:
    window: full_project
    max_files: 50
    priority_files:
      - "*.ts"
      - "*.tsx"
      - "*.py"
    exclude_patterns:
      - "node_modules/**"
      - "dist/**"
      - "*.test.ts"
      - "coverage/**"

# Supercomplete 配置
supercomplete:
  enabled: true
  auto_trigger: true
  debounce_ms: 300
  max_lines: 100
  languages:
    - typescript
    - python
    - go
    - java
    - rust

# Flow 配置
flow:
  auto_save: true
  confirm_destructive: true
  max_parallel_tasks: 3
  timeout: 300

# AI 面板配置
panel:
  position: right
  width: 400
  auto_scroll: true
  show_thinking: true
  show_cost: true

# 快捷键
keybindings:
  cascade_chat: "cmd+l"
  supercomplete: "tab"
  flow_panel: "cmd+shift+f"
  quick_search: "cmd+k"
  terminal_ai: "ctrl+`"

# 代理
proxy:
  enabled: false
  url: ""
  bypass: []

# 日志
logging:
  level: info
  file: ".windsurf/logs/app.log"
  max_size: 10mb
  max_files: 5
```

### E. 错误码参考

| 错误码 | 说明 | 解决方案 |
|--------|------|----------|
| `E001` | API 连接失败 | 检查网络和 API 配置 |
| `E002` | 认证失败 | 验证 API Key |
| `E003` | 上下文超限 | 精简上下文或开启新对话 |
| `E004` | 文件操作失败 | 检查文件权限 |
| `E005` | 模型响应超时 | 重试或切换模型 |
| `E006` | 成本超限 | 检查配额或升级套餐 |
| `E007` | 配置解析错误 | 检查 YAML 格式 |
| `E008` | 权限不足 | 检查工作区权限 |

---

## 附录：Windsurf 快捷键完全手册

### A. macOS 快捷键

| 分类 | 功能 | 快捷键 | 说明 |
|------|------|--------|------|
| **Cascade 基础** | 打开 Cascade | `Cmd + L` | 主对话入口 |
| | 发送消息 | `Enter` | 发送当前输入 |
| | 新建对话 | `Cmd + Shift + N` | 开始新会话 |
| | 清空上下文 | `/clear` | 重置对话 |
| | 停止生成 | `Esc` | 中止响应 |
| | 多行输入 | `Shift + Enter` | 换行 |
| **AI Flow** | 打开 Flow 面板 | `Cmd + Shift + F` | 启动工作流 |
| | 确认步骤 | `Tab` | 执行下一步 |
| | 跳过步骤 | `Cmd + S` | 跳过当前 |
| | 停止 Flow | `Cmd + Shift + S` | 停止执行 |
| **代码补全** | 接受补全 | `Tab` | 接受建议 |
| | 拒绝补全 | `Esc` | 拒绝建议 |
| | 下一个建议 | `Alt + ]` | 切换建议 |
| | 上一个建议 | `Alt + [` | 切换建议 |
| **编辑器** | 快速打开 | `Cmd + P` | 文件搜索 |
| | 命令面板 | `Cmd + Shift + P` | 命令列表 |
| | 全局搜索 | `Cmd + Shift + F` | 全文搜索 |
| | 多光标 | `Cmd + D` | 选择下一个 |
| | 列选择 | `Option + 拖动` | 列模式 |
| **终端** | 打开终端 | `Ctrl + `` ` | 终端面板 |
| | 终端 AI | `Cmd + Shift + `` ` | 终端内 AI |

### B. Windows/Linux 快捷键

| 分类 | 功能 | 快捷键 | 说明 |
|------|------|--------|------|
| **Cascade 基础** | 打开 Cascade | `Ctrl + L` | 主对话入口 |
| | 发送消息 | `Enter` | 发送当前输入 |
| | 新建对话 | `Ctrl + Shift + N` | 开始新会话 |
| | 清空上下文 | `/clear` | 重置对话 |
| | 停止生成 | `Esc` | 中止响应 |
| | 多行输入 | `Shift + Enter` | 换行 |
| **AI Flow** | 打开 Flow 面板 | `Ctrl + Shift + F` | 启动工作流 |
| | 确认步骤 | `Tab` | 执行下一步 |
| | 跳过步骤 | `Ctrl + S` | 跳过当前 |
| | 停止 Flow | `Ctrl + Shift + S` | 停止执行 |
| **代码补全** | 接受补全 | `Tab` | 接受建议 |
| | 拒绝补全 | `Esc` | 拒绝建议 |
| | 下一个建议 | `Alt + ]` | 切换建议 |
| | 上一个建议 | `Alt + [` | 切换建议 |
| **编辑器** | 快速打开 | `Ctrl + P` | 文件搜索 |
| | 命令面板 | `Ctrl + Shift + P` | 命令列表 |
| | 全局搜索 | `Ctrl + Shift + F` | 全文搜索 |
| | 多光标 | `Ctrl + D` | 选择下一个 |
| | 列选择 | `Alt + 拖动` | 列模式 |
| **终端** | 打开终端 | `` Ctrl + ` `` | 终端面板 |
| | 终端 AI | `Ctrl + Shift + `` ` | 终端内 AI |

### C. 自定义快捷键

```json
// keybindings.json
[
  // Cascade 快捷键
  {
    "key": "cmd+shift+c",
    "command": "windsurf.cascade.open",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+a",
    "command": "windsurf.cascade.accept",
    "when": "inlineSuggestionVisible"
  },
  {
    "key": "cmd+shift+d",
    "command": "windsurf.cascade.dismiss",
    "when": "inlineSuggestionVisible"
  },
  
  // Flow 快捷键
  {
    "key": "cmd+shift+w",
    "command": "windsurf.flow.new",
    "when": "editorTextFocus"
  },
  {
    "key": "cmd+shift+e",
    "command": "windsurf.flow.execute",
    "when": "windsurfFlowActive"
  },
  
  // 快速操作
  {
    "key": "cmd+shift+r",
    "command": "windsurf.refactor.selected",
    "when": "editorTextFocus && editorHasSelection"
  },
  {
    "key": "cmd+shift+g",
    "command": "windsurf.generate.test",
    "when": "editorTextFocus"
  }
]
```

---

**Windsurf 文档最终统计**：
- 撰写时间：2026年4月
- 最终行数：约 **5500+ 行**
- 代码示例：120+
- 配置示例：45+
- 快捷键表格：65+
- 提示词模板：25+
- FAQ 方案：15+
- 技术栈集成案例：20+

> [!SUCCESS]
> Windsurf 作为 Codeium 出品的 AI 编程助手，凭借其独特的 Cascade 架构和 Supercomplete 技术，在 AI Flow 和上下文理解方面展现出显著优势。其免费版的完整功能使个人开发者能够零成本体验高级 AI 编程能力。结合 Claude、GPT 等大模型的强大能力，Windsurf 成为 2026 年最具性价比的 AI 编程工具。
