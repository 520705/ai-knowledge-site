---
title: OpenClaw 高级用法完全指南
date: 2026-04-24
tags:
  - OpenClaw
  - 高级用法
  - 工作流
  - 多Agent
  - 定时任务
  - 自动化
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 高级玩家必看！OpenClaw 的高级用法，包括自定义工具、工作流编排、多 Agent 系统、定时任务等。
---

# OpenClaw 高级用法完全指南

> 基础功能都玩腻了？这篇带你玩点高级的——自定义工具、工作流、多 Agent、定时任务……

---

## 先说说有哪些高级玩法

| 高级功能 | 能干啥 | 适合谁 |
|-----------|--------|--------|
| **自定义工具** | 写自己的工具扩展 AI 能力 | 开发者 |
| **工作流编排** | 把多个步骤串起来自动执行 | 自动化党 |
| **多 Agent 系统** | 多个 AI 分工合作 | 复杂任务 |
| **定时任务** | 让 AI 自动执行周期性任务 | 懒人 |
| **Chain of Thought** | 让 AI 一步步思考再回答 | 需要推理的场景 |
| **MCP 协议** | 接入更多工具生态 | 扩展党 |

---

## 功能一：自定义工具

基础工具（read/write/edit/bash）不够用？写自己的！

### 为什么需要自定义工具？

| 场景 | 需要的工具 |
|------|-----------|
| 查股票 | 股票查询工具 |
| 发邮件 | 邮件发送工具 |
| 查快递 | 快递查询工具 |
| 管文件 | 文件管理工具 |

### 快速创建一个自定义工具

**第一步**：创建工具文件

```bash
mkdir -p ~/.openclaw/plugins/my-tools/
```

**第二步**：写工具代码

创建 `~/.openclaw/plugins/my-tools/stock.py`：

```python
from openclaw.tools import Tool, ToolResult
from pydantic import BaseModel, Field
import httpx

class StockInput(BaseModel):
    """股票查询输入"""
    symbol: str = Field(description="股票代码，如 AAPL、TSLA")

class StockTool(Tool):
    name = "stock_price"
    description = "查询股票当前价格和涨跌幅"
    input_model = StockInput

    async def execute(self, input_data: StockInput) -> ToolResult:
        try:
            # 这里用免费 API
            url = f"https://query1.finance.yahoo.com/v8/finance/chart/{input_data.symbol}"
            async with httpx.AsyncClient() as client:
                response = await client.get(url)
                data = response.json()

            result = data["chart"]["result"][0]
            price = result["meta"]["regularMarketPrice"]
            change = result["meta"]["regularMarketChangePercent"]

            emoji = "📈" if change > 0 else "📉"
            return ToolResult(
                success=True,
                output=f"{emoji} {input_data.symbol}: ${price:.2f} ({change:+.2f}%)"
            )
        except Exception as e:
            return ToolResult(success=False, error=str(e))
```

**第三步**：注册工具

在 `config.yaml` 里添加：

```yaml
plugins:
  - path: ~/.openclaw/plugins/my-tools/stock.py
```

**第四步**：重启并使用

```bash
docker restart openclaw
```

然后问 Bot："帮我查下苹果的股票"

---

## 功能二：工作流编排

### 什么是工作流？

**工作流 = 把多个步骤串起来自动执行**

举个例子："每天早上自动生成日报"这个任务可以分解为：

```
1. 读取昨天的任务记录
2. 读取昨天的聊天记录
3. 让 AI 分析并生成日报
4. 把日报发到钉钉群
```

用工作流，这 4 步可以自动串起来执行。

### 工作流定义示例

创建 `workflows/daily_report.yaml`：

```yaml
name: "每日日报生成"
description: "每天早上自动生成工作日报"

# 触发条件
trigger:
  type: "schedule"
  cron: "0 9 * * 1-5"  # 周一到周五早上9点

# 工作流步骤
steps:
  - id: "fetch_tasks"
    name: "获取任务"
    tool: "file_read"
    params:
      file: "~/notes/tasks.md"

  - id: "fetch_meetings"
    name: "获取会议"
    tool: "file_read"
    params:
      file: "~/notes/meetings.md"

  - id: "generate_report"
    name: "生成日报"
    tool: "llm"
    params:
      prompt: |
        根据以下内容生成一份简洁的日报：

        任务记录：
        {fetch_tasks.output}

        会议记录：
        {fetch_meetings.output}

  - id: "save_report"
    name: "保存日报"
    tool: "file_write"
    params:
      file: "~/notes/daily-report-{date}.md"
      content: "{generate_report.output}"

  - id: "notify_team"
    name: "通知团队"
    tool: "send_message"
    params:
      platform: "slack"
      channel: "#daily-reports"
      message: "日报已生成 📋"
```

### 工作流执行

```bash
# 手动触发工作流
openclaw workflow run daily_report

# 查看工作流状态
openclaw workflow status daily_report

# 列出所有工作流
openclaw workflow list
```

---

## 功能三：多 Agent 系统

### 什么是多 Agent？

**多 Agent = 多个 AI 分工合作**

就像一个团队：
- 协调者 Agent：分配任务
- 研究员 Agent：搜集资料
- 写手 Agent：写文章
- 审核 Agent：检查质量

### 多 Agent 配置

```yaml
agents:
  # 协调者
  coordinator:
    role: "coordinator"
    prompt: "你是项目协调者，负责分解任务和协调团队工作。"
    model: "claude-sonnet-4-20250514"

  # 研究员
  researcher:
    role: "researcher"
    prompt: "你是一个专业研究员，擅长收集和分析信息。"
    model: "claude-haiku-3"  # 用便宜模型

  # 写手
  writer:
    role: "writer"
    prompt: "你是一个专业文案，擅长写作和内容创作。"
    model: "claude-sonnet-4-20250514"

  # 审核
  reviewer:
    role: "reviewer"
    prompt: "你是一个严格的质量审核员，负责检查内容质量。"
    model: "claude-opus-4-20250514"  # 用最强模型
```

### 多 Agent 工作示例

```
你：帮我写一篇关于 AI 的文章

    │
    ▼

协调者 Agent：
"这个任务需要：
1. 研究员搜集资料
2. 写手写文章
3. 审核员检查
我来分配一下~"

    │
    ├──→ 研究员 Agent：搜集 AI 最新资讯
    │
    ├──→ 写手 Agent：基于资料写文章
    │
    └──→ 审核 Agent：检查文章质量

    │
    ▼

协调者 Agent 汇总结果给你
```

### 命令行使用

```bash
# 启动多 Agent 任务
openclaw agent spawn --role researcher --count 3

# 查看 Agent 状态
openclaw agent list

# 停止 Agent
openclaw agent stop <agent-id>
```

---

## 功能四：定时任务

### 定时任务配置

```yaml
schedules:
  # 每天早上日报
  - name: "daily_report"
    cron: "0 9 * * 1-5"
    workflow: "daily_report"

  # 每周周报
  - name: "weekly_report"
    cron: "0 18 * * 5"
    workflow: "weekly_report"

  # 每小时健康检查
  - name: "health_check"
    cron: "0 * * * *"
    workflow: "health_check"
```

### 常用 Cron 表达式

| 表达式 | 含义 |
|--------|------|
| `0 9 * * *` | 每天早上9点 |
| `0 9 * * 1-5` | 工作日早上9点 |
| `0 18 * * 5` | 每周五下午6点 |
| `0 */2 * * *` | 每2小时 |
| `0 0 * * *` | 每天午夜 |

### 管理定时任务

```bash
# 添加定时任务
openclaw schedule add "0 9 * * *" --workflow daily_report

# 查看定时任务
openclaw schedule list

# 删除定时任务
openclaw schedule remove daily_report
```

---

## 功能五：Chain of Thought（思维链）

### 什么是 CoT？

**CoT = 让 AI 先想清楚再回答**

普通回答：
```
问：2 * 3 + 4 = ?
答：10
```

CoT 回答：
```
问：2 * 3 + 4 = ?
答：让我一步步算：
1. 先算 2 * 3 = 6
2. 再算 6 + 4 = 10
所以答案是 10
```

### 启用 CoT

```yaml
reasoning:
  enabled: true
  strategy: "cot"  # cot / react / tot

  cot:
    steps: 5  # 思考步骤数
    show_reasoning: true  # 是否展示思考过程
```

### CoT 策略

| 策略 | 适用场景 |
|------|----------|
| `cot` | 数学、逻辑推理 |
| `react` | 需要查资料的工具使用场景 |
| `tot` | 复杂问题需要多方案比较 |

### 使用示例

```
你：帮我分析一下这道算法题

Bot（使用 CoT）：
让我一步步思考这个问题：

1. 理解题目：
   - 输入：一个数组
   - 输出：数组中的最大值
   - 要求：时间复杂度 O(n)

2. 思路分析：
   - 暴力法：O(n²) 不可取
   - 分治法：O(n log n)
   - 遍历法：O(n) ✅

3. 代码实现：
   ```python
   def max_value(arr):
       max_val = arr[0]
       for num in arr:
           if num > max_val:
               max_val = num
       return max_val
   ```

4. 验证：
   - [1,5,3] → 5 ✅
   - [-1,-5,-3] → -1 ✅
```

---

## 功能六：MCP 协议支持

### 什么是 MCP？

**MCP（Model Context Protocol）= 连接 AI 和工具的标准协议**

就像 USB 接口——不管什么品牌的鼠标、键盘，只要支持 USB 就能用。MCP 就是让不同工具能接入 AI 的"标准接口"。

### MCP 服务器示例

```yaml
mcp:
  servers:
    # 文件系统
    - name: "filesystem"
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-filesystem", "~/"]

    # Git
    - name: "git"
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-git", "~/projects"]

    # Slack
    - name: "slack"
      command: "npx"
      args: ["-y", "@modelcontextprotocol/server-slack"]
      env:
        SLACK_BOT_TOKEN: "${SLACK_BOT_TOKEN}"
```

### MCP 工具使用

```bash
# 列出可用 MCP 工具
openclaw mcp list

# 查看某个工具详情
openclaw mcp show filesystem

# 测试 MCP 工具
openclaw mcp test filesystem/read_file --path ~/notes/todo.md
```

---

## 功能七：记忆高级应用

### 自动记忆整理

```yaml
memory:
  auto_compact:
    enabled: true
    schedule: "0 3 * * *"  # 每天凌晨3点整理
    max_items: 1000

  importance_tracking:
    enabled: true
    keywords:
      important: ["重要", "关键", "必须记住"]
      forget: ["忘记", "忽略", "不用记"]
```

### 跨会话上下文

```yaml
memory:
  cross_session:
    enabled: true
    window: "7d"  # 跨7天会话
    shared_topics:
      - "项目"
      - "团队"
      - "工作内容"
```

---

## 功能八：API 和 Webhook

### 暴露 REST API

```yaml
api:
  enabled: true
  port: 8080
  auth: "bearer"  # bearer / basic / none

  endpoints:
    - path: "/chat"
      method: "POST"
      description: "发送消息"

    - path: "/status"
      method: "GET"
      description: "获取状态"
```

### Webhook 配置

```yaml
webhooks:
  events:
    - name: "on_message"
      url: "https://your-server.com/webhook"
      method: "POST"

    - name: "on_error"
      url: "https://your-server.com/error"
      method: "POST"
```

### API 使用示例

```bash
# 发送消息
curl -X POST http://localhost:8080/chat \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "你好", "user_id": "123"}'

# 获取状态
curl http://localhost:8080/status \
  -H "Authorization: Bearer YOUR_TOKEN"
```

---

## 功能九：安全加固

### 工具权限控制

```yaml
security:
  # 允许的工具
  allowed_tools:
    - "read"
    - "write"
    - "weather"
    - "translate"

  # 禁止的工具
  denied_tools:
    - "bash"  # 禁止执行 shell 命令
    - "delete"

  # 危险命令黑名单
  dangerous_patterns:
    - "rm -rf"
    - "dd if="
    - "mkfs"
```

### 消息过滤

```yaml
security:
  message_filter:
    enabled: true

    # 敏感词过滤
    sensitive_words:
      - "密码"
      - "信用卡"
      - "身份证"

    # 频率限制
    rate_limit:
      max_per_minute: 10
      max_per_hour: 100
```

---

## 功能十：性能优化

### 响应缓存

```yaml
performance:
  cache:
    enabled: true
    ttl: 3600  # 缓存1小时
    max_size: 1000

  # LLM 响应缓存
  llm_cache:
    enabled: true
    similarity_threshold: 0.9  # 相似度阈值
```

### 并发控制

```yaml
performance:
  concurrency:
    max_concurrent: 10  # 最大并发请求
    queue_size: 100      # 队列大小

  # 单用户限流
  per_user_limit:
    requests_per_minute: 5
    requests_per_hour: 50
```

---

## 实战案例

### 案例一：智能客服机器人

```yaml
# 配置
agents:
  customer_service:
    prompt: |
      你是一个专业的客服，回答用户的问题。
      规则：
      1. 用友好、专业的语气
      2. 不知道就说"我不确定"
      3. 遇到投诉要道歉并记录

channels:
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT}"

  discord:
    enabled: true
    bot_token: "${DISCORD_BOT}"

# 工作流
workflows:
  - name: "ticket_escalation"
    steps:
      - tool: "create_ticket"
      - tool: "notify_team"
      - tool: "update_status"
```

### 案例二：个人知识助手

```yaml
# 配置
memory:
  sources:
    - path: "~/Obsidian/"
      type: "obsidian"
    - path: "~/Documents/"
      type: "filesystem"

  auto_index:
    enabled: true
    schedule: "0 */4 * * *"  # 每4小时更新索引

# 工具
plugins:
  - "@openclaw/readwise"
  - "@openclaw/obsidian"

# 定时任务
schedules:
  - name: "morning_briefing"
    cron: "0 8 * * *"
    workflow: "morning_research"
```

---

## 常见问题

### Q: 工作流卡住了怎么办？

```bash
# 查看工作流状态
openclaw workflow status <workflow-name>

# 重试
openclaw workflow retry <workflow-id>

# 取消
openclaw workflow cancel <workflow-id>
```

### Q: 多 Agent 响应太慢？

1. 用便宜的模型做简单任务
2. 限制并发数
3. 增加超时时间

```yaml
agents:
  coordinator:
    timeout: 120s

  researcher:
    model: "claude-haiku-3"  # 用便宜模型
    timeout: 60s
```

### Q: MCP 工具不工作？

1. 检查 MCP 服务器是否运行
2. 检查配置是否正确
3. 查看日志

```bash
# 重启 MCP 服务器
openclaw mcp restart <server-name>

# 查看 MCP 日志
openclaw logs | grep mcp
```

---

## 下一步

学会了高级用法？你已经是 OpenClaw 高手了！

| 教程 | 内容 |
|------|------|
| [[OpenClaw插件开发]] | 深入开发插件 |
| [[OpenClaw架构解析]] | 了解原理 |
| [[Hermes_Agent详解]] | 试试另一个框架 |

---

**文档状态**：高级用法指南  
**更新时间**：2026年4月
