---
title: ClawHub 技能市场完全指南
date: 2026-04-24
tags:
  - ClawHub
  - 插件市场
  - 技能
  - OpenClaw
  - 安装教程
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 学会在 ClawHub 上找插件、装插件、管理插件，让你的 AI 助手功能翻倍。
---

# ClawHub 技能市场完全指南

> ClawHub 是 OpenClaw 的"应用商店"，有 3200+ 个插件可以装。这篇告诉你怎么逛这个市场、怎么挑插件、怎么装上它。

---

## ClawHub 是什么？

### 一句话解释

**ClawHub = OpenClaw 的"应用商店"**

就像手机有应用商店（App Store/应用宝），OpenClaw 也有自己的应用商店——ClawHub。

在这里，你可以找到各种各样的"技能包"：
- 查天气的
- 管 GitHub 的
- 读写 Notion 的
- AI 画图的
- 翻译的
- ……

装上就能用，不用自己写代码。

### 数据概览

| 指标 | 数据 |
|------|------|
| 插件总数 | **3200+** |
| 活跃开发者 | 500+ |
| 总下载量 | 500万+ |
| 周更新量 | 100+ |
| 分类数量 | 15 大类 |

---

## 访问 ClawHub

### 网页版（推荐）

访问地址：https://clawhub.openclaw.ai

在这里你可以：
- 按分类浏览插件
- 搜索插件
- 看插件评分和评价
- 找相似插件

### 命令行版

```bash
# 搜索插件
openclaw skill search "天气"

# 列出热门插件
openclaw skill list --sort downloads

# 查看插件详情
openclaw skill info @openclaw/weather
```

---

## 插件分类一览

### 15 大类

| 分类 | 数量 | 代表插件 |
|------|------|----------|
| **生产力** | 500+ | 邮件、日历、任务管理 |
| **开发工具** | 400+ | GitHub、Docker、API 测试 |
| **信息检索** | 600+ | 搜索、新闻、股票 |
| **数据分析** | 300+ | CSV、JSON、可视化 |
| **媒体处理** | 250+ | 图片、音频、视频 |
| **通信** | 200+ | 邮件、短信、通知 |
| **系统工具** | 200+ | 文件、进程、网络 |
| **AI/ML** | 400+ | 画图、翻译、OCR |
| **娱乐** | 300+ | 游戏、音乐、段子 |
| **社交** | 150+ | Twitter、Reddit |
| **电商** | 100+ | 价格追踪、订单管理 |
| **金融** | 150+ | 投资、预算、发票 |
| **健康** | 80+ | 运动、饮食、冥想 |
| **教育** | 120+ | 学习、记忆、测验 |
| **其他** | 200+ | 未分类 |

---

## 热门插件推荐

### 必备插件

这些插件装机量最高，建议先装：

| 插件 | 功能 | 装机量 |
|------|------|--------|
| `@openclaw/weather` | 查天气 | 80万+ |
| `@openclaw/github` | 管 GitHub | 60万+ |
| `@openclaw/notion` | 读写 Notion | 50万+ |
| `@openclaw/web-search` | 网页搜索 | 80万+ |
| `@openclaw/translate` | 翻译 | 45万+ |

### 开发者必备

| 插件 | 功能 |
|------|------|
| `@openclaw/docker-manager` | Docker 容器管理 |
| `@openclaw/code-reviewer` | 代码审查 |
| `@openclaw/api-tester` | API 测试 |
| `@openclaw/deploy-bot` | 一键部署 |

### 效率工具

| 插件 | 功能 |
|------|------|
| `@openclaw/calendar` | 日历管理 |
| `@openclaw/email-pro` | 邮件处理 |
| `@openclaw/reminder` | 智能提醒 |
| `@openclaw/pomodoro` | 番茄钟 |

### 知识管理

| 插件 | 功能 |
|------|------|
| `@openclaw/obsidian` | Obsidian 集成 |
| `@openclaw/readwise` | 读书笔记 |
| `@openclaw/wiki-search` | 维基百科搜索 |

---

## 怎么装插件？

### 方式一：命令行安装

最简单的方式：

```bash
# 安装单个插件
openclaw skill install @openclaw/weather

# 安装多个插件
openclaw skill install @openclaw/weather @openclaw/translate @openclaw/github

# 安装指定版本
openclaw skill install @openclaw/weather==2.3.0

# 从 GitHub 安装（开发版）
openclaw skill install --source github username/repo
```

### 方式二：配置文件

编辑 `config.yaml`：

```yaml
skills:
  install:
    - "@openclaw/weather"      # 天气
    - "@openclaw/translate"    # 翻译
    - "@openclaw/github"       # GitHub
    - "@openclaw/notion"       # Notion
```

### 方式三：交互式安装

```bash
# 启动交互式安装向导
openclaw skill install --interactive

# 流程：
# 1. 选择分类
# 2. 浏览插件列表
# 3. 查看插件详情
# 4. 确认安装
# 5. 配置参数
```

---

## 怎么找合适的插件？

### 按需求找

| 需求 | 搜索关键词 | 推荐插件 |
|------|-----------|----------|
| 查天气 | weather | `@openclaw/weather` |
| 查天气（中文） | weather cn | `@openclaw/weather-cn` |
| 翻译 | translate | `@openclaw/translate` |
| GitHub | github | `@openclaw/github` |
| 笔记 | note, obsidian | `@openclaw/obsidian` |
| 日历 | calendar | `@openclaw/calendar` |
| 画图 | image, draw | `@openclaw/image-gen` |

### 按评分找

```bash
# 按评分排序
openclaw skill list --sort rating

# 找评分 4.5 以上的
openclaw skill search "github" --min-rating 4.5
```

### 按下载量找

```bash
# 找最热门的
openclaw skill list --sort downloads --limit 20

# 找新发布的
openclaw skill list --sort newest --limit 10
```

---

## 插件使用示例

### 天气插件

**安装**：
```bash
openclaw skill install @openclaw/weather
```

**使用**（在 Telegram/Discord 里）：
```
用户: 北京今天天气怎么样？
Bot: 🌤️ 北京今天天气：
- 温度：22°C
- 天气：晴
- 空气：良
- 湿度：45%

适合外出，记得防晒！
```

### 翻译插件

**安装**：
```bash
openclaw skill install @openclaw/translate
```

**使用**：
```
用户: 帮我翻译 "Hello, how are you?" 成中文
Bot: 中文翻译：「你好，你怎么样？」

用户: 把这段日语翻译成英文
Bot: 日语原文：今日は良い天気ですね
英文翻译：The weather is nice today, isn't it?
```

### GitHub 插件

**安装**：
```bash
openclaw skill install @openclaw/github
```

**使用**：
```
用户: 帮我看看我的仓库有什么新 issue
Bot: 📋 你的仓库最新 Issue：

🔴 [Bug] 登录页面打不开
   5 分钟前 · @username1
   
⚠️ [问题] API 响应太慢
   2 小时前 · @username2

✅ [已完成] 更新文档
   1 天前 · @username3
```

### Notion 插件

**安装**：
```bash
openclaw skill install @openclaw/notion
```

**配置**（在 `config.yaml` 里）：
```yaml
skills:
  config:
    notion:
      api_key: "${NOTION_API_KEY}"
      database_id: "你的Notion数据库ID"
```

**使用**：
```
用户: 把我今天的学习笔记保存到 Notion
Bot: ✅ 已保存！

📝 标题：2026-04-24 学习笔记
📁 位置：Notion 数据库 > Daily Notes
🔗 链接：https://notion.so/xxx
```

---

## 插件管理

### 查看已安装

```bash
# 列出所有已安装插件
openclaw skill list

# 查看某个插件详情
openclaw skill info @openclaw/weather

# 查看插件配置
openclaw skill config @openclaw/weather
```

### 更新插件

```bash
# 更新单个插件
openclaw skill update @openclaw/weather

# 更新所有插件
openclaw skill update --all

# 查看可更新列表
openclaw skill outdated
```

### 卸载插件

```bash
# 卸载单个插件
openclaw skill uninstall @openclaw/weather

# 卸载多个
openclaw skill uninstall @openclaw/weather @openclaw/translate
```

### 启用/禁用插件

```bash
# 禁用（不删除）
openclaw skill disable @openclaw/weather

# 启用
openclaw skill enable @openclaw/weather
```

---

## 开发自己的插件

有编程基础？想贡献社区？可以开发自己的插件。

### 插件结构

```
my-plugin/
├── plugin.json          # 插件信息
├── __init__.py         # 入口
├── tools/              # 工具目录
│   └── my_tool.py
└── requirements.txt    # 依赖
```

### 最小插件示例

**plugin.json**：
```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "description": "我的第一个插件",
  "author": "你的名字"
}
```

**__init__.py**：
```python
from openclaw.plugins import Plugin

class Plugin(Plugin):
    name = "my-plugin"

    async def on_load(self):
        print("我的插件加载了！")

    def get_tools(self):
        # 返回你的工具列表
        return []
```

### 发布到 ClawHub

```bash
# 本地测试
openclaw skill install ./my-plugin

# 发布
openclaw skill publish --path ./my-plugin
```

详细开发教程见：[[OpenClaw插件开发]]

---

## 常见问题

### Q: 装插件报错？

**可能原因**：

1. **版本不兼容**
```bash
# 查看 OpenClaw 版本
openclaw --version

# 检查插件兼容性
openclaw skill info @openclaw/weather
```

2. **缺少依赖**
```bash
# 查看缺失的依赖
docker logs openclaw 2>&1 | grep "ImportError"
```

3. **权限问题**
```bash
# 重新安装
openclaw skill uninstall <plugin>
openclaw skill install <plugin>
```

### Q: 插件装了但不管用？

1. **重启服务**：
```bash
docker restart openclaw
```

2. **检查配置**：有些插件需要配置 API Key
```bash
openclaw skill config <plugin>
```

3. **看日志**：
```bash
docker logs openclaw 2>&1 | grep -i error
```

### Q: 怎么找中文插件？

```bash
# 搜索中文相关的
openclaw skill search "中文"
openclaw skill search "chinese"

# 或者在网页版筛选
# https://clawhub.openclaw.ai/search?q=中文
```

### Q: 能自己写插件吗？

当然可以！见：[[OpenClaw插件开发]]

---

## 安全提示

### 风险提示

ClawHub 的插件质量参差不齐，大约 **7.1%** 可能有安全风险。

**建议**：
1. 优先装官方插件（`@openclaw/*`）
2. 装机量大的插件（经过社区验证）
3. 查看插件评分和评价
4. 不要装来路不明的插件

### 安全检查

```bash
# 安装前扫描
openclaw skill scan @openclaw/weather

# 检查已安装插件
openclaw skill audit
```

---

## 下一步

学会了用插件？继续探索：

| 教程 | 内容 |
|------|------|
| [[OpenClaw多平台集成]] | 接更多平台 |
| [[OpenClaw记忆系统]] | 让 AI 记住更多 |
| [[OpenClaw插件开发]] | 自己写插件 |
| [[OpenClaw高级用法]] | 高级技巧 |

---

**文档状态**：ClawHub 完全指南  
**更新时间**：2026年4月
