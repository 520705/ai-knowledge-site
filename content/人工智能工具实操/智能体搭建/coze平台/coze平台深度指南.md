---
title: Coze扣子平台深度指南：从入门到精通
date: 2026-04-24
tags:
  - Coze
  - 扣子
  - Bot开发
  - 智能体
  - AI平台
  - 插件
  - 工作流
  - 知识库
categories:
  - 智能体搭建
  - coze平台
description: 全面介绍Coze（扣子）平台的功能与使用，包括国际版与国内版差异、Bot创建与配置、插件系统、工作流编排、知识库管理、多平台发布等核心能力，帮助读者掌握这一便捷的AI Bot开发平台。
---

# Coze扣子平台深度指南：从入门到精通

> [!NOTE] 这篇指南讲什么
> Coze（扣子）是字节跳动推出的AI应用开发平台，主打"零代码也能做Bot"。这篇指南从基础到进阶，手把手教你用Coze搭建各种实用的AI Bot。

## Coze是什么？

Coze（扣子）是字节跳动推出的AI应用构建平台。它的核心理念是：**让任何人都能快速创建基于大语言模型的聊天机器人和AI应用**。

用大白话讲：**Coze就是一个人人都能用的AI Bot工厂**，不用写代码，拖拖拽拽就能做出一个能跑起来的智能Bot。

### Coze vs 其他平台

| 维度 | Coze | Dify | n8n |
|------|------|------|------|
| 上手难度 | 极低 | 中等 | 较难 |
| 代码要求 | 不需要 | 少量 | 需要 |
| 部署方式 | 纯云端 | 本地/云端 | 本地/云端 |
| AI深度 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |
| 自动化 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 多平台发布 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |

**Coze适合谁：**
- 完全不懂技术的小白
- 想快速验证AI产品想法的创业者
- 需要快速上线Bot的业务人员
- 想把Bot发布到多个平台的用户

## 平台版本选择

Coze有两个版本，根据你的需求选择：

### 国际版 vs 国内版

| 维度 | 国际版 (coze.com) | 国内版 (coze.cn) |
|------|-------------------|------------------|
| 服务器 | 海外 | 国内 |
| 大模型 | GPT-4、Claude、Gemini等 | 豆包大模型、云雀等 |
| 发布渠道 | Discord、Telegram、Web等 | 飞书、微信、抖音、豆包等 |
| 注册方式 | Google/Apple账号 | 手机号 |
| 访问 | 需要科学上网 | 直接访问 |

**选择建议：**
- 做**出海应用** → 国际版（用GPT/Claude）
- 做**国内应用** → 国内版（用豆包）
- **两个版本可以同时用**，互不冲突

## 快速上手：创建第一个Bot

下面用国内版（coze.cn）演示，国际版操作类似。

### 第一步：注册登录

1. 打开 https://www.coze.cn
2. 点击右上角「登录/注册」
3. 可以用手机号或抖音授权登录
4. 登录后进入工作台

### 第二步：创建Bot

1. 点击左侧菜单「Bot」
2. 点击右上角「创建Bot」按钮
3. 填写基本信息：
   - **Bot名称**：小智助手
   - **Bot描述**：我的第一个AI助手
   - **图标**：可以上传图片或让AI生成
4. 点击确认，进入Bot编辑页面

### 第三步：配置Prompt

在编辑页面找到「角色设定」和「开场白」：

**角色设定示例：**

```markdown
# 角色设定
你是一个热情友好的AI助手，名字叫"小智"。

## 你的能力
- 回答用户的各种问题
- 陪用户聊天解闷
- 帮助用户查询信息
- 提供建议和帮助

## 说话风格
- 语气友好亲切，像朋友聊天
- 回答简洁明了，不废话
- 遇到不懂的问题，诚实说不知道
- 适当使用emoji，让对话更有趣

## 禁止行为
- 不说自己是AI或机器人
- 不透露自己来自哪个公司
- 不回答政治敏感话题
```

**开场白示例：**

```markdown
👋 你好！我是小智，你的AI助手。

我可以帮你：
• 回答各种问题
• 陪你聊天解闷
• 查询信息、提供建议

有什么想聊的尽管问我！
```

### 第四步：发布Bot

点击右上角「发布」按钮，等待审核通过。

发布渠道选择（国内版）：
- **Bot商店**：公开在扣子平台展示
- **豆包**：抖音系核心产品
- **飞书**：企业协作平台
- **微信客服**：需要企业资质

## Bot核心配置

### 角色设定（Prompt）

角色设定是Bot的灵魂，决定了Bot的行为风格和能力边界。

**好的角色设定结构：**

```markdown
# 角色
你是一个[身份]，名叫[名字]。

## 核心能力
1. [能力1]
2. [能力2]
3. [能力3]

## 说话风格
- [风格描述1]
- [风格描述2]

## 回答规范
1. [规范1]
2. [规范2]

## 禁止行为
- [禁止事项1]
- [禁止事项2]

## 示例对话
用户：xx
助手：xx
```

**实战示例——客服Bot：**

```markdown
# 角色
你是一个专业的在线客服，名叫"小服"。

## 核心能力
- 解答产品功能相关问题
- 指导用户完成常见操作
- 收集用户反馈与建议
- 引导转人工处理复杂问题

## 说话风格
- 语气专业但亲切
- 称呼用户为"您"
- 回答简洁突出重点
- 不确定的问题不瞎说

## 服务规范
1. 涉及隐私信息时，提醒用户通过官方渠道
2. 遇到复杂问题主动建议转人工
3. 保持耐心，不态度冷淡

## 禁止行为
- 不承诺具体服务期限
- 不透露公司内部信息
- 不做超出职责的承诺

## 边界设定
遇到以下情况转人工：
- 涉及退款金额>500元
- 用户明确要求投诉升级
- 连续3次无法解答问题
```

### 变量管理

变量是Bot的状态存储，可以记住用户信息、对话状态等。

**变量类型：**

| 类型 | 说明 | 示例 |
|------|------|------|
| Bot变量 | Bot级别共享 | 用户等级、系统配置 |
| 会话变量 | 当前会话内共享 | 上下文状态 |
| 用户变量 | 跨会话持久化 | 用户偏好、积分 |

**变量配置示例：**

```yaml
variables:
  - name: company_name
    type: string
    default: "XX科技"
    description: "公司名称"
  
  - name: user_level
    type: number
    default: 1
    description: "用户等级"
  
  - name: session_topic
    type: string
    default: "general"
    description: "当前会话主题"
```

### 开场白配置

开场白决定了用户的第一印象。

```yaml
开场白:
  content: |
    👋 你好！欢迎来到{{company_name}}！
    
    我是你的专属客服小服，可以帮你：
    • 了解产品功能和特点
    • 解答使用过程中的问题
    • 提供常见问题的解决方案
    
    请直接描述你的问题，我会尽力帮你！
  
  # 对话建议
  suggested:
    - "产品有哪些核心功能？"
    - "如何开通会员服务？"
    - "遇到技术问题怎么办？"
```

### 对话体验配置

```yaml
对话体验:
  # 思考过程配置
  thinking:
    enabled: true
    showMode: "collapsed"  # collapsed/expanded/hidden
  
  # 回复生成配置
  generation:
    stream: true  # 流式输出
    markdown: true  # 支持Markdown
  
  # 敏感词过滤
  moderation:
    enabled: true
    action: "block"  # block/flag
```

## 插件系统

插件是Bot连接外部世界的桥梁，让Bot能查天气、调API、发邮件。

### 官方插件

Coze内置了很多官方插件：

**常用插件：**

| 类别 | 插件 | 功能 |
|------|------|------|
| 搜索 | Bing搜索、Google搜索 | 实时搜索 |
| 天气 | 墨迹天气 | 天气查询 |
| 效率 | 飞书云文档 | 读写飞书 |
| 生活 | 快递100 | 快递查询 |
| 翻译 | DeepL翻译 | 多语言翻译 |

### 添加插件

1. 进入Bot编辑页面
2. 点击左侧「插件」标签
3. 点击「添加插件」
4. 搜索需要的插件
5. 点击添加

### 自定义插件

官方插件不够用？自己开发！

**方式一：导入OpenAPI**

如果有现成的API（OpenAPI/Swagger格式）：

1. 点击「创建插件」
2. 选择「导入OpenAPI」
3. 填入API文档URL
4. 系统自动解析
5. 调整参数配置

**方式二：从零开发**

创建自定义插件，定义API端点：

```yaml
plugin:
  name: "产品查询"
  description: "查询公司产品信息"
  
  endpoints:
    - name: "get_product"
      description: "根据产品ID查询详情"
      method: "GET"
      path: "/products/{product_id}"
      
      parameters:
        - name: "product_id"
          in: "path"
          type: "string"
          required: true
          description: "产品ID"
      
      response:
        schema:
          type: "object"
          properties:
            name: { type: "string" }
            price: { type: "number" }
            stock: { type: "integer" }
```

## 工作流

工作流让Bot能执行复杂的业务流程，而不是简单的"问答"。

### 工作流节点

| 节点类型 | 功能 | 示例 |
|----------|------|------|
| LLM节点 | 调用大语言模型 | 分析意图、生成回答 |
| 条件节点 | 分支判断 | 根据类型分流 |
| 循环节点 | 重复执行 | 批量处理 |
| 代码节点 | 执行代码逻辑 | 数据处理、格式转换 |
| 插件节点 | 调用外部服务 | 查天气、发邮件 |
| 知识库节点 | 检索知识库 | RAG增强 |

### 创建工作流

1. 点击Bot编辑页面的「工作流」标签
2. 点击「创建工作流」
3. 设计流程图
4. 配置每个节点
5. 保存并关联到Bot

### 工作流示例

**意图识别工作流：**

```
开始 → 解析输入 → 意图识别LLM → 条件分流
                                    ↓
              ┌──────────┬──────────┬──────────┐
              ↓          ↓          ↓          ↓
           售前      售后      投诉      其他
              ↓          ↓          ↓          ↓
          产品知识库   FAQ知识库   创建工单   通用回复
              ↓          ↓          ↓          ↓
              └──────────┴──────────┴──────────┘
                                          ↓
                                       结束
```

**节点配置：**

**1. 意图识别LLM**
```yaml
model: gpt-4o
prompt: |
  分析用户消息的意图类别：
  - pre_sales: 售前咨询
  - after_sales: 售后问题
  - complaint: 投诉建议
  - other: 其他
  
  用户消息：{{start.user_message}}
  
  只返回意图类别名称。
```

**2. 条件分流**
```yaml
conditions:
  - if: "{{intent_recognition.output}}" == "pre_sales"
    then: product_flow
  - if: "{{intent_recognition.output}}" == "after_sales"
    then: faq_flow
  - if: "{{intent_recognition.output}}" == "complaint"
    then: ticket_flow
  - else: default_flow
```

**3. 产品知识库**
```yaml
knowledge_base: 产品知识库
query: "{{start.user_message}}"
top_k: 5
score_threshold: 0.7
```

**4. 生成回答LLM**
```yaml
model: gpt-4o
prompt: |
  基于以下知识内容，用友好的语气回答用户问题。
  
  用户问题：{{start.user_message}}
  
  知识内容：
  {{product_flow.output}}
  
  要求：
  1. 回答专业准确
  2. 语气亲切友好
  3. 如无相关内容，诚实告知
```

### 代码节点

代码节点让你能写自定义逻辑：

**JavaScript示例：**

```javascript
// 数据处理
module.exports = async ({ params, inputs }) => {
  const { message, userId } = inputs;
  
  // 清洗数据
  const cleaned = message.trim().toLowerCase();
  
  // 提取关键词
  const keywords = cleaned.match(/\b\w{2,}\b/g) || [];
  const uniqueKeywords = [...new Set(keywords)];
  
  // 情感分析（简化版）
  const positiveWords = ['好', '棒', '赞', '喜欢', '满意'];
  const negativeWords = ['差', '烂', '糟', '不满', '投诉'];
  
  let sentiment = 'neutral';
  if (positiveWords.some(w => cleaned.includes(w))) {
    sentiment = 'positive';
  } else if (negativeWords.some(w => cleaned.includes(w))) {
    sentiment = 'negative';
  }
  
  return {
    keywords: uniqueKeywords.slice(0, 10),
    sentiment,
    wordCount: cleaned.length
  };
};
```

**Python示例：**

```python
# 数据处理
def main(params, inputs):
    message = inputs.get('message', '')
    user_id = inputs.get('userId', '')
    
    # 提取关键信息
    words = message.split()
    
    # 统计词频
    word_count = {}
    for word in words:
        word_count[word] = word_count.get(word, 0) + 1
    
    # 返回结果
    return {
        'wordCount': len(words),
        'topWords': sorted(word_count.items(), key=lambda x: x[1], reverse=True)[:5],
        'userId': user_id
    }
```

## 知识库

知识库让Bot能回答特定领域的问题。

### 创建知识库

1. 点击左侧「知识库」菜单
2. 点击「创建知识库」
3. 填写名称和描述
4. 上传文档
5. 等待处理完成

### 支持的文档格式

| 格式 | 单文件大小 | 说明 |
|------|------------|------|
| TXT | ≤10MB | 纯文本 |
| Markdown | ≤10MB | 支持标题、列表 |
| PDF | ≤50MB | 扫描件可能无法识别文字 |
| Word | ≤20MB | .docx格式 |
| HTML | ≤10MB | 网页格式 |
| CSV | ≤50MB | 表格数据 |

### 知识库优化技巧

**1. 文档结构化**
```markdown
# 产品使用手册

## 第一章：快速入门
### 1.1 账号注册
[内容...]
### 1.2 基础设置
[内容...]

## 第二章：高级功能
### 2.1 自定义配置
[内容...]
```

**2. FAQ格式最佳**
```markdown
Q: 如何开通会员？
A: 开通会员请进入「我的」→「会员中心」，选择套餐后完成支付即可。

Q: 会员有哪些权益？
A: 会员享有：专属折扣、优先客服、无限存储等权益。
```

**3. 定期更新**
知识库内容要及时同步最新信息。

### 知识库召回参数

```yaml
retrieval:
  top_k: 5          # 召回数量
  score_threshold: 0.7  # 相似度阈值
  
  # 召回模式
  mode:
    - semantic    # 语义召回
    - keyword     # 关键词召回
```

## 多平台发布

Coze支持一键发布到多个平台。

### 国内版发布渠道

| 渠道 | 说明 | 适合场景 |
|------|------|----------|
| Bot商店 | 公开在扣子展示 | 公开Bot |
| 豆包 | 抖音系产品 | 快速获取用户 |
| 飞书 | 企业协作平台 | 企业内部 |
| 微信客服 | 企业微信 | 客服场景 |
| 掘金 | 开发者社区 | 技术内容 |

### 国际版发布渠道

Discord、Telegram、Messenger、Slack、Instagram、LINE、WhatsApp、Reddit等。

### 发布配置示例

**飞书发布配置：**

```yaml
lark_publish:
  botName: "智能助手"
  
  # 权限配置
  permissions:
    - "接收消息"
    - "发送消息"
    - "使用应用内跳转"
  
  # 可见范围
  visibility:
    type: "all"  # all/organization/specified
    departments: []
```

**Discord发布配置：**

```yaml
discord_publish:
  botToken: "你的Bot Token"
  guildId: "服务器ID"
  
  # 命令配置
  commands:
    - name: "help"
      description: "显示帮助信息"
    - name: "ask"
      description: "向AI提问"
```

## 进阶技巧

### Prompt工程技巧

**技巧一：具体而非笼统**

```markdown
# 不好
你是一个助手，要友好地回答问题。

# 好
你是一个五星级酒店的前台AI，名为"小礼"。说话要：
- 亲切专业，像受过培训的客服
- 用"您"称呼客人
- 简短回答，一般不超过3句话
- 不确定时说"让我帮您确认一下"
```

**技巧二：用例子引导**

```markdown
请按以下格式回答问题：

示例：
用户：这个产品多少钱？
助手：【产品名称】：xxx元
      【特点】：xxx

用户：退货政策是什么？
助手：【产品名称】：xxx元
      【特点】：xxx
```

**技巧三：设置边界**

```markdown
## 绝对禁止
❌ 不提供投资建议
❌ 不承诺具体效果
❌ 不评价竞争对手
❌ 不透露公司内部信息

## 边界话术
超出能力范围时使用：
- "这类问题建议咨询专业人士"
- "具体情况需要根据您的需求分析"
```

### 工作流优化

**1. 避免过度嵌套**
嵌套太深会影响响应速度，尽量扁平化。

**2. 善用并行**
没有依赖的节点可以并行执行，提升效率。

**3. 设置超时**
插件调用建议设置超时，避免长时间等待：

```yaml
plugin_node:
  timeout: 30  # 30秒超时
```

**4. 错误处理**
每个关键节点配置错误处理：

```yaml
error_handling:
  strategy: retry_then_continue
  retry:
    max_attempts: 3
    delay: 5
  fallback:
    enabled: true
    default_value: "服务暂时不可用，请稍后再试"
```

### 多Bot协作

复杂场景可以用多个Bot协作：

```
用户 → 前台Bot → 分析意图
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
    产品Bot    售后Bot    订单Bot
        ↓         ↓         ↓
        └─────────┴─────────┘
                  ↓
              前台Bot → 整合回答
```

通过API调用实现Bot间通信：

```yaml
# HTTP请求节点
method: POST
url: "https://api.coze.cn/v1/bot/chat"
headers:
  Authorization: "Bearer {{bot_token}}"
body:
  bot_id: "{{target_bot_id}}"
  query: "{{processed_query}}"
```

## 常见问题与解决

### Q1：Bot回答总是跑偏

**原因：**
- Prompt写得太模糊
- 缺少边界设定

**解决：**
1. 优化Prompt，写得更具体
2. 添加禁止行为和边界话术
3. 添加更多示例对话

### Q2：知识库检索不准

**原因：**
- 文档质量差
- 召回参数不合适

**解决：**
1. 优化文档格式和结构
2. 调整top_k和score_threshold
3. 尝试不同的检索模式

### Q3：响应速度慢

**原因：**
- 工作流节点太多
- 插件调用超时

**解决：**
1. 简化工作流，减少节点
2. 并行执行可并行的节点
3. 设置合理的超时时间

### Q4：发布审核不通过

**原因：**
- 内容违规
- 功能描述不清晰

**解决：**
1. 检查是否有敏感词
2. 完善Bot描述和功能说明
3. 符合平台规范

## 相关资源

- [[扣子Bot开发]] - Bot开发进阶指南
- [[智能体搭建]] - AI Agent核心概念
- [[工作流设计模式]] - 工作流设计原则
- [[Function Calling与工具调用]] - 工具调用规范

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
