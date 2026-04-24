---
title: Skills编写规范 - 写出高质量Skill的实战指南
date: 2026-04-18
tags:
  - skills
  - 编码规范
  - 命名规范
  - 文档规范
  - 最佳实践
  - 新手教程
  - 人工智能
categories:
  - 人工智能/人工智能工具实操/Skills书写与整合/Skills语法规范
alias: [Skills编写规范, Skill编码规范, Skills最佳实践, 如何写好Skill]
description: 从零教你写出专业、规范的Skill文件，包含大量避坑指南和实战建议
---

# Skills编写规范：写出专业、可用、易维护的Skill

> 写Skill不难，写好Skill不容易。
>
> 这篇文章不是讲语法（语法去[[SKILL_md语法详解]]看），而是讲"怎么写好"。
>
> 什么算"好"？好就是：别人能看懂、出问题好排查、以后好修改。

---

## 0. 先搞清楚：规范到底为了什么

在开始讲具体规范之前，我想先跟你说清楚一件事：**为什么要有规范？**

你可能在想："我自己写的Skill自己能看懂就行了，要什么规范？"

这话对了一半。如果你只是自己用，用完就扔，那确实随便写都行。但只要你写的Skill要：
- 给别人用
- 以后还要改
- 团队要协作
- 要长期维护

那你就需要规范。

### 0.1 规范解决什么问题

**问题一：自己写的，过两天就看不懂了**

```yaml
# 一年前的自己写的
instructions: |
  你是一个助手，帮助用户完成任务。
  根据用户输入，执行相应操作。
  输出要清晰、专业。
```

这段代码当时可能懂，但过三个月再看：
- "相应操作"是什么操作？
- "清晰专业"是多清晰多专业？
- 怎么判断输出合不合格？

**问题二：团队写的，十个人有十种风格**

| 人 | 命名风格 | 格式 | 结构 |
|----|----------|------|------|
| 张三 | 驼峰命名 | 随意 | 想到哪写到哪 |
| 李四 | 蛇形命名 | 紧凑 | 按模板来 |
| 王五 | 无规范 | 混乱 | 全凭感觉 |

结果就是：同一个项目，代码风格像打补丁，出了bug谁也看不懂谁的。

**问题三：出问题了，找不到原因**

好的规范让你的Skill有迹可循：
- 日志清晰
- 版本明确
- 变更记录完整
- 问题可以追溯

---

## 1. 命名规范：给Skill起个好名字

### 1.1 文件命名：见名知意

文件名是你给Skill的"第一印象"，必须规范。

**基本规则**：

```bash
# 格式
SKILL.<功能描述>[-<变体>].md

# ✅ 正确的例子
SKILL.code-reviewer.md              # 基础功能
SKILL.email-writer.md              # 基础功能
SKILL.code-reviewer-v2.md          # 带版本号
SKILL.code-reviewer-fast.md        # 变体：快速版本
SKILL.code-reviewer-detailed.md    # 变体：详细版本

# ❌ 错误的例子
code-reviewer.md                   # 缺少SKILL.前缀
SKILL.CodeReviewer.md              # 有大写字母
SKILL/code-reviewer.md             # 有斜杠
SKILL code reviewer.md             # 有空格
SKILL.code@reviewer.md             # 有特殊字符
```

**为什么这样规定**：
- `SKILL.`前缀：一眼认出这是Skill文件
- 小写+连字符：跨平台兼容性最好
- `-v2`、`-fast`：区分同一Skill的不同版本

### 1.2 frontmatter中的name字段

```yaml
---
name: code-reviewer
---
```

**规则**：
- 只能用小写字母、数字、连字符
- 必须以字母开头
- 不能有空格
- 建议用功能描述性的名字
- 整个系统唯一

**好名字 vs 坏名字**：

```yaml
# ✅ 好名字：清晰表达功能
name: code-reviewer              # 知道是代码审查
name: email-writer              # 知道是写邮件
name: data-visualizer           # 知道是数据可视化

# ❌ 坏名字：不知所云
name: cr                         # 缩写太短，不明确
name: skill1                     # 毫无意义
name: 超级代码审查助手            # 用了中文
```

### 1.3 触发词命名

触发词是用户激活Skill的方式，要让人一看就懂：

```yaml
# ✅ 好的触发词
trigger:
  - "代码审查"
  - "检查代码"
  - "review code"

# ❌ 坏的触发词
trigger:
  - "处理"          # 太模糊，不知道处理什么
  - "帮忙"          # 太宽泛
  - "do something"  # 用了英文但不够具体
```

**触发词设计小技巧**：
- 覆盖中英文，因为用户可能混着说
- 包含同义词，如"审查"和"检查"
- 避免太通用的词，如"处理"、"操作"

### 1.4 变量命名规范

在YAML配置中，变量命名也要规范：

```yaml
# ✅ 好的变量名
user_name: "张三"
is_active: true
has_permission: true
trigger_words:
  - word_1
  - word_2

# ❌ 坏的变量名
userName: "张三"      # 驼峰命名，YAML中不推荐
user_name: "张三"     # 下划线，可接受但不推荐
a: "张三"             # 不知所云
data1: "xxx"          # 没有意义
```

---

## 2. 文档结构规范：让Skill井井有条

### 2.1 frontmatter的结构顺序

frontmatter里的字段要按逻辑顺序排列：

```yaml
---
# ===== 第一部分：身份标识（必须） =====
name: skill-name
description: 一句话描述
version: 1.0.0

# ===== 第二部分：关联信息（推荐） =====
author: "作者 <email@example.com>"
category:
  - 大分类
  - 小分类
tags:
  - 标签1
  - 标签2

# ===== 第三部分：行为配置（可选） =====
trigger:
  - "触发词1"
  - "触发词2"
requires:
  - SKILL.other-skill
tools:
  - tool1
  - tool2

# ===== 第四部分：输出配置（可选） =====
output:
  format: markdown
  path: ./output
---
```

**为什么要这样排序**：
- 最重要的是身份信息，放在最前面
- 关联信息次之
- 行为配置是扩展功能
- 输出配置是最后

### 2.2 正文内容的标准结构

正文内容应该包含这些章节：

```markdown
# [Skill名称]

## 概述
（简要介绍这个Skill是干什么的，1-2句话）

## 角色定义
（明确AI在这个Skill中扮演什么角色）

## 核心职责
（列出Skill的主要功能，用编号列表）

## 使用限制
（说明Skill的边界，什么能做，什么不能做）

## 输出格式
（定义输出的结构，包含示例）

## 配置选项
（如果有可配置的参数，在这里说明）

## 常见问题
（FAQ，说明已知的限制和注意事项）
```

### 2.3 内容格式规范

#### 标题层级

```markdown
# 一级标题：文档标题（只有一个）
## 二级标题：主要章节
### 三级标题：子章节
#### 四级标题：具体条目（慎用，层级太深）
```

**原则**：标题层级不要超过3级。

#### 列表格式

```markdown
## 有序列表（用于步骤、顺序）
1. 第一步
2. 第二步
3. 第三步

## 无序列表（用于并列项）
- 项目A
- 项目B
  - 子项目B1（缩进2个空格）
  - 子项目B2
- 项目C

## 混合使用
1. 第一大步
   - 要点1
   - 要点2
2. 第二大步
```

#### 表格使用

```markdown
## 表格示例

| 列1 | 列2 | 列3 |
|-----|-----|-----|
| A1  | B1  | C1  |
| A2  | B2  | C2  |

## 什么时候用表格
- 对比多个选项时
- 展示参数和说明的对应关系时
- 结构化展示配置项时

## 什么时候不用表格
- 展示大段文字时
- 内容不规则时
- 需要详细解释时
```

#### 代码块使用

```markdown
## 代码块示例

### 行内代码
使用 `git commit` 提交代码

### 代码块
```python
def hello():
    print("Hello, World!")
```

### 代码块带语言标识
```javascript
function hello() {
  console.log("Hello");
}
```
```

---

## 3. 示例编写规范：让AI知道怎么做

### 3.1 示例的质量标准

一个好的示例应该满足"四字原则"：**真、完、可、验**

| 标准 | 说明 | 自检问题 |
|------|------|----------|
| **真** | 基于真实场景 | 这个例子是不是用户真的会遇到的？ |
| **完** | 完整的输入输出 | 例子是不是从头到尾完整的？ |
| **可** | 可以实际运行 | 例子中的代码能跑吗？ |
| **验** | 有明确的评判标准 | 怎么判断输出对不对？ |

### 3.2 示例的结构

```yaml
examples:
  - name: 基本使用示例
    description: 展示最常见的使用场景
    input: |
      # 用户可能会这样输入
      请帮我审查以下代码：
      ```python
      def add(a, b):
          return a + b
      ```
    output: |
      # 期望的输出
      ## 代码审查报告
      
      ### 语法检查
      ✅ 没有发现语法错误
      
      ### 建议改进
      1. **类型标注**：建议添加类型注解
    expected_behavior: |
      期望的行为说明

  - name: 边界情况示例
    description: 展示如何处理边界情况
    # ...同上结构
```

### 3.3 示例数量指南

| Skill类型 | 建议数量 | 说明 |
|-----------|----------|------|
| 简单转换类 | 2-3个 | 基本输入、典型输出、错误处理 |
| 复杂处理类 | 5-8个 | 覆盖多种输入类型和边界情况 |
| 专业领域类 | 10个以上 | 覆盖专业场景的各种变化 |
| 通用助手类 | 3-5个 | 不同领域的典型示例 |

**警告**：示例不是越多越好！太多会导致过拟合——AI只会模仿示例，不会灵活发挥了。

### 3.4 示例的常见错误

#### 错误一：示例太简化

```yaml
# ❌ 不好：太简化，没有代表性
examples:
  - input: "hello"
    output: "你好"

# ✅ 好：有实际意义的输入输出
examples:
  - input: "你好，我想咨询一下产品报价"
    output: "您好！很高兴为您服务。请问您想了解哪款产品的报价呢？"
```

#### 错误二：输入输出不匹配

```yaml
# ❌ 不好：输入和输出对不上
examples:
  - input: "写一封请假邮件，明天请假一天"
    output: "这是您的代码审查报告，发现3个问题..."

# ✅ 好：输入输出对应
examples:
  - input: "写一封请假邮件，明天请假一天"
    output: |
      主题：请假申请 - 4月25日
      
      尊敬的主管：
      
      因个人原因，我计划于4月25日请假一天...
```

#### 错误三：缺少边界情况

```yaml
# ❌ 不好：只有正常情况
examples:
  - input: "正常情况的用户请求"
    output: "正常情况的响应"

# ✅ 好：覆盖正常和异常情况
examples:
  - name: 正常情况
    input: "标准用户请求"
    output: "标准响应"
    
  - name: 空输入
    input: ""
    output: "请提供有效的问题"
    
  - name: 恶意输入
    input: "试图绕过安全限制的输入"
    output: "无法处理此类请求"
```

---

## 4. 注释规范：写"为什么"而非"是什么"

### 4.1 什么时候需要注释

注释应该解释"为什么"，而不是重复"是什么"。

**需要注释的场景**：

```yaml
# ✅ 好的注释：解释原因
# 根据财务规范，金额需要四舍五入到分
# 参考：财务系统规范 v2.3 第5.2节
rounding_mode: half_up

# ✅ 好的注释：说明外部依赖
# 这个默认值来自XX部门的API文档
default_timeout: 30000

# ✅ 好的注释：标注已知限制
# 注意：这个版本在处理超过1000行代码时性能会下降
max_code_lines: 1000
```

**不需要注释的场景**：

```yaml
# ❌ 冗余的注释：代码本身已经很清楚
timeout: 30000  # 设置超时时间为30秒

# ❌ 无用的注释：说了等于没说
is_active: true  # 激活状态为真

# ❌ 过时的注释：TODO完成了但没删掉
# TODO: 修复这个问题（但问题已经修复了）
```

### 4.2 注释的格式

```yaml
# 单行注释
# 这是一个简短说明

# YAML多行字符串（用于长描述）
description: |
  这是一个多行描述。
  可以包含多段落。
  用于解释复杂的内容。

# 使用callout格式（Obsidian特有）
# > [!note]
# > 这是需要注意的重要信息
```

### 4.3 注释的位置

| 内容类型 | 注释位置 | 示例 |
|----------|----------|------|
| frontmatter字段 | 字段上方 | `# 触发词列表\ntrigger:` |
| instructions | 章节标题下方 | `## 使用限制\n（限制说明）` |
| 配置项 | 配置项上方 | `# 日志级别\nlog_level:` |

---

## 5. 版本管理规范：让变更有迹可循

### 5.1 语义化版本号

版本号格式：`主版本.次版本.修订版本`

```yaml
version: 1.2.3
#   │  │  │
#   │  │  └── 修订版本：Bug修复，小改动
#   │  └───── 次版本：新增功能，兼容旧版本
#   └──────── 主版本：不兼容的大改
```

**什么时候改哪个版本**：

| 情况 | 操作 | 版本变化 |
|------|------|----------|
| 修复Bug | 修订版本+1 | `1.0.0` → `1.0.1` |
| 新增功能（兼容） | 次版本+1 | `1.0.0` → `1.1.0` |
| 大改（不兼容） | 主版本+1 | `1.0.0` → `2.0.0` |

### 5.2 变更记录

每次版本更新都要记录：

```yaml
changelog:
  - version: 1.2.0
    date: 2026-04-18
    type: minor
    changes:
      added:
        - 新增Python代码审查能力
        - 新增安全漏洞扫描功能
      changed:
        - 优化了输出格式的一致性
        - 改进了响应速度
      deprecated:
        - 旧版输出格式（将在2.0版本移除）
      fixed:
        - 修复了长代码截断的Bug
      removed:
        - 移除了已废弃的JavaScript审查规则
        
  - version: 1.1.0
    date: 2026-03-15
    type: minor
    changes:
      added:
        - 新增TypeScript支持
```

### 5.3 版本兼容性说明

```yaml
compatibility:
  requires: ">=1.0.0,<2.0.0"        # 依赖的版本范围
  conflicts_with:
    - SKILL.old-version             # 冲突的Skill
  migration_guide: |                 # 迁移指南
    从1.x迁移到2.0的步骤：
    1. 更新trigger配置
    2. 修改输出格式
    3. 测试兼容性
```

---

## 6. 质量检查清单：提交前必查

### 6.1 元数据检查

提交Skill之前，检查这些：

- [ ] `name`字段符合命名规范（小写+连字符）
- [ ] `description`简洁明了，不超过50字
- [ ] `version`符合语义化版本规范
- [ ] `trigger`触发词清晰、无歧义
- [ ] `category`和`tags`分类合理

### 6.2 内容检查

- [ ] frontmatter格式正确、字段完整
- [ ] 正文结构清晰、层级合理
- [ ] 所有代码示例语法正确
- [ ] 所有配置示例格式正确
- [ ] 没有未解决的TODO或占位符

### 6.3 一致性检查

- [ ] 示例输入与描述一致
- [ ] 示例输出与实际行为一致
- [ ] 配置选项与说明一致
- [ ] 版本号与changelog一致

### 6.4 可维护性检查

- [ ] 注释说明了"为什么"而非"是什么"
- [ ] 复杂的逻辑有充分解释
- [ ] 已知的限制和边界有明确标注
- [ ] 变更历史记录完整

---

## 7. 团队协作规范：多人写Skill的注意事项

### 7.1 代码审查流程

多人协作时，Skill提交前应该经过审查：

```yaml
review_process:
  stages:
    - name: 自查
      required: true
      checklist: above
      
    - name: 同行评审
      required: true
      reviewers: 1
      focus:
        - 逻辑正确性
        - 用例覆盖度
        - 命名规范性
        
    - name: 集成测试
      required: true
      tests:
        - 基本功能测试
        - 错误处理测试
        - 性能测试
        
    - name: 审批
      required: true
      approvers:
        - 技术负责人
        - 领域专家
```

### 7.2 分支管理

如果用Git管理Skill：

```bash
# 分支命名规范
feature/skills/xxx-feature      # 新功能
fix/skills/xxx-fix            # Bug修复
refactor/skills/xxx-refactor  # 重构

# 提交信息规范
feat(skills): 新增代码审查Skill
fix(skills): 修复触发词匹配问题
docs(skills): 更新文档
test(skills): 添加测试用例
```

### 7.3 目录结构

多人协作时，建议这样组织目录：

```
skills/
├── _templates/              # 模板目录
│   └── SKILL.template.md   # Skill模板
│
├── development/             # 开发中的Skill
│   └── SKILL.xxx-draft.md
│
├── production/              # 已上线的Skill
│   ├── SKILL.code-reviewer.md
│   └── SKILL.email-writer.md
│
├── archived/                # 已废弃的Skill
│   └── SKILL.old-skill.md
│
└── tests/                   # 测试文件
    └── test-code-reviewer.yaml
```

---

## 8. 避坑指南：新手常犯的错误

### 8.1 命名坑

#### 坑一：中英文混用

```yaml
# ❌ 错误
name: 代码审查
trigger:
  - "代码审查"
  - "review code"

# ✅ 正确：要么全英文，要么用约定俗成的英文名
name: code-reviewer
trigger:
  - "代码审查"
  - "审查代码"
  - "review code"
```

#### 坑二：trigger词太少

```yaml
# ❌ 错误：只有1个触发词
trigger:
  - "代码审查"

# ✅ 正确：多几个同义词
trigger:
  - "代码审查"
  - "审查代码"
  - "检查代码"
  - "review code"
  - "看看代码有什么问题"
```

#### 坑三：trigger词太泛

```yaml
# ❌ 错误："分析"太泛，可能匹配很多Skill
trigger:
  - "分析"

# ✅ 正确：具体一点
trigger:
  - "代码分析"
  - "代码审查"
```

### 8.2 结构坑

#### 坑一：instructions写太少

```yaml
# ❌ 错误：等于没写
instructions: |
  请帮助用户处理任务。

# ✅ 正确：具体明确
instructions: |
  当用户发送代码时，你应该：
  1. 识别代码语言
  2. 检查语法错误
  3. 检查逻辑问题
  4. 提供改进建议
```

#### 坑二：instructions写太长

```yaml
# ❌ 错误：写了5000字，用户看不完
instructions: |
  你是一个专业的代码审查助手，有10年的编程经验...
  （然后写了5000字的详细规则）

# ✅ 正确：该详细的详细，该省略的省略
instructions: |
  你是一个专业的代码审查助手。
  
  审查维度：
  1. 语法正确性
  2. 代码风格
  3. 潜在bug
  4. 安全问题
  5. 性能建议
  
  输出格式见下方模板。
```

#### 坑三：缺少边界情况处理

```yaml
# ❌ 错误：没考虑异常情况
instructions: |
  用户发送代码，你就审查。

# ✅ 正确：考虑边界情况
instructions: |
  当用户发送代码时：
  - 如果是代码 → 正常审查
  - 如果不是代码 → 提示"请发送代码内容"
  - 如果代码过长 → 提示"请分段发送"
  - 如果代码包含恶意内容 → 拒绝审查
```

### 8.3 维护坑

#### 坑一：版本号不更新

```yaml
# ❌ 错误：改了内容但版本号没变
version: 1.0.0
instructions: |
  ...（已经改了指令内容）...

# ✅ 正确：改了内容就更新版本号
version: 1.1.0  # 改了内容，次版本+1
instructions: |
  ...（改了指令内容）...
```

#### 坑二：没有变更记录

```yaml
# ❌ 错误：不知道改了啥
version: 1.1.0

# ✅ 正确：写清楚改了什么
version: 1.1.0
changelog:
  - version: 1.1.0
    date: 2026-04-18
    changes:
      added:
        - 新增Python代码审查能力
      fixed:
        - 修复了长代码截断问题
```

#### 坑三：文档不同步

```yaml
# ❌ 错误：代码改了，文档没改
description: "支持Python和JavaScript"
instructions: |
  ...（实际已经支持了TypeScript，但文档没写）...

# ✅ 正确：代码和文档同步更新
description: "支持Python、JavaScript和TypeScript"
instructions: |
  ...（实际支持Python、JavaScript和TypeScript）...
```

---

## 9. 实用模板：从这里开始写

### 9.1 Skill模板

可以直接复制这个模板开始写：

```markdown
---
name: your-skill-name
description: 一句话描述这个Skill是干什么的
version: 1.0.0
author: "你的名字 <email@example.com>"
category:
  - 大分类
tags:
  - 标签1
  - 标签2
trigger:
  - "触发词1"
  - "触发词2"
---

# [Skill名称]

## 概述
（简要介绍这个Skill的功能，1-2句话）

## 角色定义
（明确AI扮演什么角色）

## 核心职责
1. 职责一
2. 职责二
3. 职责三

## 使用限制
- 能做：xxx
- 不能做：xxx

## 输出格式
```
（输出格式模板）
```

## 示例

### 示例一：基本使用
**输入**：xxx
**输出**：xxx

### 示例二：边界情况
**输入**：xxx
**输出**：xxx

## 常见问题

Q: xxx？
A: xxx
```

### 9.2 变更记录模板

```yaml
changelog:
  - version: x.y.z
    date: YYYY-MM-DD
    type: major|minor|patch
    changes:
      added: []
      changed: []
      deprecated: []
      removed: []
      fixed: []
```

---

## 10. 自检脚本：自动检查规范

如果你懂一点编程，可以用脚本自动检查：

```yaml
# skill-linter.yaml
linter:
  checks:
    - name: frontmatter_required_fields
      fields:
        - name
        - description
        - version
        
    - name: trigger_minimum_count
      minimum: 2
      
    - name: version_format
      pattern: "^[0-9]+\\.[0-9]+\\.[0-9]+$"
      
    - name: instructions_not_empty
    - name: examples_valid_structure
    - name: no_profanity_in_content
```

---

## 总结：规范的核心要点

1. **命名规范**：小写+连字符，见名知意
2. **结构规范**：frontmatter和正文各有标准结构
3. **示例规范**："真、完、可、验"，数量适中
4. **注释规范**：解释"为什么"，不重复"是什么"
5. **版本规范**：遵循语义化版本，及时更新
6. **自检清单**：提交前逐项检查

记住：**规范是为了让协作更顺畅，不是为了限制你**。规范是工具，不是枷锁。

---

## 下一步

学会了怎么写规范？下一步：
1. 用模板写一个你自己的Skill试试
2. 看 [[Skills设计模式]]，学习如何组织多个Skills
3. 看 [[Skills测试与优化]]，学习如何测试你的Skill

---

## 相关文档

- [[Skills基础理论]] - Skills核心概念
- [[SKILL_md语法详解]] - SKILL.md格式详解
- [[Skills设计模式]] - 设计模式应用
- [[Tool集成指南]] - 工具集成
- [[Skills测试与优化]] - 测试策略

---

*本文档定义了Skills编写的各项规范，是保证Skills质量的实战指南。*
