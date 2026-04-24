---
title: OpenClaw 记忆系统完全指南
date: 2026-04-24
tags:
  - OpenClaw
  - 记忆系统
  - Memory
  - 上下文
  - 知识管理
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 深入了解 OpenClaw 的记忆系统，怎么让 AI 记住你是谁，怎么管理记忆。
---

# OpenClaw 记忆系统完全指南

> 普通 AI 每次对话都是"新面孔"，你不认识它，它也不认识你。记忆系统就是让 AI 变成"老朋友"的关键。

---

## 先搞清楚：什么是记忆系统？

### 简单解释

**记忆系统 = AI 的"笔记本"**

想象一下：
- 普通 AI 就像鱼，只有 7 秒记忆
- 有记忆系统的 AI 就像老朋友，记得你叫什么、喜欢什么、之前聊过什么

### 没有记忆 vs 有记忆

**没有记忆**：

```
你：帮我看看上次那个项目怎么样了
Bot：抱歉，我不知道你在说什么项目
```

**有记忆**：

```
你：帮我看看上次那个项目怎么样了
Bot：你是说"智慧城市"项目吗？目前进度如下：
- 第一阶段：已完成 ✅
- 第二阶段：进行中 🔄
- 第三阶段：待开始 ⏳
```

### 记忆系统的价值

| 场景 | 没有记忆 | 有记忆 |
|------|----------|--------|
| "上次聊的那个事" | 不记得 | 记得 |
| "我是谁" | 不认识 | 认识 |
| "我喜欢的风格" | 不知道 | 知道 |
| "之前给的数据" | 不记得 | 记得 |

---

## OpenClaw 记忆系统是怎么工作的？

### 三层记忆结构

OpenClaw 的记忆分为三层：

| 层次 | 生命周期 | 容量 | 用途 |
|------|----------|------|------|
| **工作记忆** | 当前会话 | ~8000 tokens | 正在处理的信息 |
| **短期记忆** | 几天 | ~100 条 | 最近的对话 |
| **长期记忆** | 永久 | ~1000 条 | 重要的事实和偏好 |

### 打个比方

```
工作记忆 = 桌上的便签（用完就扔）
短期记忆 = 抽屉里的文件（保留一周）
长期记忆 = 保险柜里的重要文件（永久保存）
```

---

## 记忆是怎么存的？

### 文件结构

OpenClaw 用 **Markdown 文件**存记忆：

```
~/.openclaw/memory/
├── long_term/                    # 长期记忆
│   ├── facts.md                 # 事实：你是谁、做什么
│   ├── preferences.md           # 偏好：你喜欢什么
│   ├── knowledge.md             # 知识：你的专业领域
│   └── projects/                # 项目记忆
│       └── smart-city.md        # "智慧城市"项目详情
├── short_term/                  # 短期记忆
│   └── session_20260424_abc.md # 今天会话
└── working/                    # 工作记忆
    └── context.md              # 当前工作上下文
```

### 记忆文件长什么样？

**facts.md**：
```markdown
# 关于用户的事实

- 名字叫张三
- 在北京工作，是一名产品经理
- 创业中，项目是智慧城市解决方案
- 2025 年开始做 AI 相关产品
- 技术背景：Python、JavaScript
```

**preferences.md**：
```markdown
# 用户偏好

- 喜欢简洁的回复，不要太啰嗦
- 喜欢用中文
- 喜欢在早上处理复杂问题
- 咖啡爱好者
- 不喜欢被打扰，除非紧急
```

**projects/smart-city.md**：
```markdown
# 智慧城市项目

## 基本信息
- 启动时间：2025年3月
- 团队规模：5人
- 当前阶段：第二阶段开发

## 技术栈
- 前端：React + TypeScript
- 后端：Python FastAPI
- 数据库：PostgreSQL
- AI：Claude API

## 当前进展
- [x] 需求调研
- [x] 架构设计
- [ ] 第一阶段开发
- [ ] 测试
```

---

## 怎么管理记忆？

### 方式一：手动编辑文件

最直接的方式——直接改 Markdown 文件。

```bash
# 打开记忆文件
nano ~/.openclaw/memory/long_term/facts.md

# 或者用 VSCode
code ~/.openclaw/memory/long_term/facts.md
```

### 方式二：告诉 AI 帮忙记录

```
你：记住我叫小明，在上海工作，是个前端开发
Bot：好的，已记住！
- 名字：小明
- 城市：上海
- 职业：前端开发
```

### 方式三：安装记忆管理插件

```bash
# 安装记忆插件
openclaw skill install @openclaw/memory-manager

# 常用命令
openclaw memory list        # 列出所有记忆
openclaw memory add "..."   # 添加记忆
openclaw memory delete <id> # 删除记忆
openclaw memory search "..." # 搜索记忆
```

---

## 怎么让 AI 自动记住重要对话？

### 触发词设置

在 `config.yaml` 里设置：

```yaml
memory:
  auto_remember:
    enabled: true
    keywords:
      - "记住"
      - "帮我记一下"
      - "这个很重要"
      - "以后要用"
    importance_threshold: 0.7    # 重要性阈值
```

### 工作原理

```
你说："帮我记住，我喜欢用中文回复"
    │
    ▼
OpenClaw 检测到触发词
    │
    ▼
提取关键信息
    │
    ▼
保存到 facts.md 或 preferences.md
    │
    ▼
下次对话时，AI 读取这些记忆
    │
    ▼
AI 用中文回复 ✅
```

### 手动标记重要性

```
你：这是一个非常重要的项目，帮我记住
Bot：好的，已标记为重要信息！
```

---

## 记忆检索是怎么工作的？

### 检索流程

```
你问问题
    │
    ▼
OpenClaw 提取关键词
    │
    ▼
搜索记忆文件
    │
    ├─── long_term/ 里搜
    ├─── short_term/ 里搜
    └─── working/ 里搜
    │
    ▼
按相关性排序
    │
    ▼
取最相关的几条记忆
    │
    ▼
塞到 AI 的上下文里
    │
    ▼
AI 有了"背景知识"，回复更准确
```

### 检索算法

OpenClaw 用**关键词匹配**来检索记忆：

```python
# 伪代码示例
def search_memory(query):
    # 1. 把问题拆成关键词
    keywords = extract_keywords(query)

    # 2. 搜索所有记忆文件
    results = []
    for file in all_memory_files:
        score = calculate_match_score(keywords, file.content)
        if score > 0.3:
            results.append((score, file))

    # 3. 按匹配度排序
    results.sort(reverse=True)

    # 4. 返回前 5 条
    return results[:5]
```

**局限性**：目前是简单的关键词匹配，不如 Hermes 的语义搜索智能。

---

## 进阶：向量数据库（可选）

如果觉得简单关键词搜索不够用，可以升级到**向量数据库**。

### 什么是向量搜索？

普通搜索找"完全匹配的词"，向量搜索找"意思相近的内容"。

| 搜索方式 | 你搜 | 找到 |
|----------|------|------|
| **关键词搜索** | "咖啡" | 包含"咖啡"的文章 |
| **向量搜索** | "早上喝什么提神" | 包含"咖啡"的文章 ✅ |

### 怎么启用向量搜索？

```yaml
memory:
  type: "vector"
  vector:
    provider: "chroma"      # chroma/qdrant/pinecone
    embedding_model: "text-embedding-3-small"

    # Chroma 配置（本地）
    chroma:
      persist_directory: "~/.openclaw/vector_db"
```

### 安装向量数据库

```bash
# Chroma（推荐，最简单）
pip install chromadb

# 或者用 Docker
docker run -d -p 8000:8000 chromadb/chroma
```

---

## 记忆安全和管理

### 备份记忆

```bash
# 备份
tar -czf memory-backup-$(date +%Y%m%d).tar.gz ~/.openclaw/memory/

# 恢复
tar -xzf memory-backup-20260424.tar.gz
```

### 导出记忆

```bash
# 导出为 JSON
openclaw memory export --format json --output memory.json

# 导出为 Markdown
openclaw memory export --format markdown --output memory/
```

### 清理记忆

```bash
# 清理旧的短期记忆（30天前）
openclaw memory prune --older-than 30d

# 清理不重要记忆
openclaw memory prune --importance low

# 查看记忆统计
openclaw memory stats
```

### 记忆加密（敏感信息）

对于敏感记忆，可以加密存储：

```yaml
memory:
  encryption:
    enabled: true
    key: "${MEMORY_ENCRYPTION_KEY}"    # 16/24/32 字节密钥
```

---

## 常见问题

### Q: AI 还是不记得之前的事？

排查步骤：

1. **检查记忆文件是否存在**
```bash
ls -la ~/.openclaw/memory/long_term/
```

2. **检查内容是否正确**
```bash
cat ~/.openclaw/memory/long_term/facts.md
```

3. **检查配置文件**
```yaml
memory:
  enabled: true
  long_term_dir: "./memory/long_term"
```

4. **重启服务**
```bash
docker restart openclaw
```

### Q: 记忆太多了，AI 加载很慢？

解决方案：

1. **减少加载数量**
```yaml
memory:
  max_context: 5    # 最多加载 5 条记忆
```

2. **定期清理不重要记忆**
```bash
openclaw memory prune --importance low
```

3. **压缩记忆**
```bash
openclaw memory compact
```

### Q: 不想让 AI 记住某些事？

**方式一**：清除特定记忆
```bash
openclaw memory delete "某条记忆"
```

**方式二**：忽略重要对话
```
你：这是私人对话，不要记住
```

**方式三**：禁用自动记忆
```yaml
memory:
  auto_remember:
    enabled: false
```

### Q: 切换用户时记忆串了？

确保每个用户有独立的记忆文件：

```yaml
memory:
  per_user_memory: true    # 开启用户隔离
```

---

## 记忆系统最佳实践

### 1. 定期整理

每周花 5 分钟整理记忆：
- 删除过时信息
- 更新变化的信息
- 补充新信息

### 2. 保持简洁

记忆不是写日记，要简洁：

```markdown
# ✅ 好例子
- 叫张三
- 创业中，项目：智慧城市
- 技术栈：React + Python

# ❌ 坏例子
- 2023年3月15日在咖啡馆遇到了一个叫张三的创业者
- 他看起来很有精神，穿着蓝色衬衫
- 我们聊了很久关于AI的话题
```

### 3. 结构化存储

按主题分类存储：

```
memory/long_term/
├── facts.md           # 基本事实
├── preferences.md     # 偏好
├── skills.md         # 技能/专业
├── projects/        # 项目
├── contacts/         # 联系人
└── notes/            # 杂项
```

### 4. 标记重要程度

```markdown
<!-- importance: high -->
- 这个项目非常重要，是公司的核心产品

<!-- importance: medium -->
- 喜欢用简洁的回复

<!-- importance: low -->
- 今天下雨了，心情不太好
```

### 5. 与笔记软件同步

```bash
# 同步 Obsidian
openclaw skill install @openclaw/obsidian

# 配置同步
openclaw memory sync --source obsidian --target openclaw
```

---

## 和 Hermes 的记忆系统对比

| 维度 | OpenClaw | Hermes |
|------|----------|--------|
| **存储方式** | Markdown 文件 | SQLite + FTS5 |
| **检索方式** | 关键词匹配 | 语义搜索 |
| **速度** | 较慢 | 快（毫秒级） |
| **容量** | 万级 | 百万级 |
| **智能程度** | 基础 | 更智能 |

**简单说**：
- OpenClaw 的记忆简单够用
- Hermes 的记忆更强大但配置复杂

---

## 下一步

学会了记忆系统？继续学习：

| 教程 | 内容 |
|------|------|
| [[OpenClaw高级用法]] | 记忆的高级应用 |
| [[OpenClaw插件开发]] | 开发记忆增强插件 |
| [[Hermes_Agent详解]] | Hermes 的智能记忆 |

---

**文档状态**：记忆系统指南  
**更新时间**：2026年4月
