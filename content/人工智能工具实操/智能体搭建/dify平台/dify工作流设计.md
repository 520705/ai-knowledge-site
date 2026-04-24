---
title: Dify工作流设计：从基础到进阶
date: 2026-04-24
tags:
  - Dify
  - 工作流
  - DAG编排
  - 节点配置
  - 工作流设计
  - 实战案例
categories:
  - AI工具实操
  - 智能体搭建
keywords:
  - Dify工作流
  - LLM节点
  - 知识库节点
  - 条件分支
  - 变量管理
  - DAG编排
  - 错误处理
  - 重试机制
  - 循环节点
  - 工作流调试
description: 深入讲解Dify工作流的节点类型、编排方式、错误处理机制和调试技巧，通过大量实战案例帮助开发者掌握复杂AI工作流的构建方法。
---

# Dify工作流设计：从基础到进阶

> [!NOTE] 这篇指南讲什么
> 这是Dify工作流的进阶教程，重点讲解各种节点的配置方法、编排技巧和实战案例。适合有Dify基础，想深入学习工作流设计的读者。

## 工作流核心概念

### 什么是DAG工作流？

Dify的工作流引擎基于DAG（有向无环图）模型。DAG是一种特殊的数据结构：
- **有向**：节点之间有明确的数据流向
- **无环**：不能形成循环，确保流程有始有终

用大白话讲：**工作流就是用节点和线画出来的流程图**，每个节点做一件事，线表示数据的流向。

### 为什么需要工作流？

| 不用工作流 | 用工作流 |
|-----------|----------|
| AI直接回答 | 先理解意图，再处理 |
| 简单问答 | 复杂业务流程 |
| 单轮响应 | 多轮交互 |
| 难以调试 | 每步都可观测 |
| 难以复用 | 模块化组件 |

### 工作流的核心优势

- **模块化**：每个节点只做一件事，可单独测试和复用
- **可视化**：业务流程一目了然
- **可观测**：每个节点的输入输出都可见
- **可扩展**：新功能只加新节点，不影响现有逻辑

## 节点类型详解

### LLM节点

LLM节点是工作流中最核心的节点，负责调用大语言模型。

**基础配置：**

```yaml
节点配置:
  model: gpt-4o
  temperature: 0.7
  max_tokens: 2000
  
  prompt: |
    你是一个{{role}}，请根据以下信息回答用户问题。
    
    用户问题：{{query}}
    
    参考资料：
    {{context}}
```

**变量引用：**

```yaml
prompt: |
  用户输入：{{ variable_name }}
  
  上一个节点的输出：{{ node_name.output }}
  
  嵌套属性：{{ node_name.output.field }}
```

**多种模型支持：**

```yaml
# 动态选择模型
model: "{{ model_selector.output.model }}"

# 或者直接指定
model: gpt-4o-mini  # 简单任务
model: gpt-4o       # 复杂任务
model: claude-3-5-sonnet  # 长文本
```

### 知识库检索节点

知识库检索是RAG系统的核心，让AI能基于文档回答问题。

**基础配置：**

```yaml
节点配置:
  knowledge_base_id: kb_xxxxx
  
  retrieval_model:
    top_k: 5           # 召回数量
    score_threshold: 0.7  # 相似度阈值
    mode: hybrid       # 混合检索
```

**检索模式对比：**

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| 语义检索 | 基于向量相似度 | 理解意图的查询 |
| 全文检索 | 关键词匹配 | 精确术语查询 |
| 混合检索 | 两者结合 | **推荐使用** |

**输出格式：**

```yaml
输出变量:
  - documents: 检索到的文档列表
  - documents[0].content: 文档内容
  - documents[0].metadata: 文档元信息
  - documents[0].score: 相似度分数
```

### 条件分支节点

条件分支实现工作流中的决策逻辑，根据不同条件走向不同分支。

**基础条件：**

```yaml
conditions:
  - if: "{{ score }} > 0.8"
    then: high_quality_flow
  - if: "{{ score }} > 0.5"
    then: medium_quality_flow
  - else: low_quality_flow
```

**复合条件：**

```yaml
conditions:
  - if: "{{ status }} == 'vip' AND {{ score }} > 0.6"
    then: vip_flow
  - if: "{{ type }} == 'complaint' OR {{ type }} == 'refund'"
    then: sensitive_flow
```

**运算符支持：**

| 运算符 | 说明 | 示例 |
|--------|------|------|
| == | 等于 | `{{ status }} == "active"` |
| != | 不等于 | `{{ type }} != "spam"` |
| > | 大于 | `{{ score }} > 0.8` |
| < | 小于 | `{{ amount }} < 1000` |
| >= | 大于等于 | `{{ level }} >= 5` |
| <= | 小于等于 | `{{ level }} <= 10` |
| AND | 且 | `A AND B` |
| OR | 或 | `A OR B` |
| contains | 包含 | `{{ tags }} contains "important"` |

### 变量节点

变量节点用于声明、管理和转换变量。

**变量类型：**

| 类型 | 说明 | 示例 |
|------|------|------|
| 字符串 | 文本 | `"Hello"` |
| 数值 | 数字 | `123`、`3.14` |
| 布尔 | true/false | `true`、`false` |
| 对象 | 复合数据 | `{"name": "张三"}` |
| 数组 | 列表 | `["a", "b", "c"]` |

**变量赋值：**

```yaml
variables:
  - name: greeting
    type: string
    value: "您好，{{ user_name }}！"
  
  - name: is_vip
    type: boolean
    value: "{{ user_level }} >= 5"
  
  - name: user_info
    type: object
    value:
      name: "{{ user_name }}"
      level: "{{ user_level }}"
      risk: "{{ calculate_risk() }}"
```

**变量引用：**

```yaml
# 字符串变量
{{ greeting }}

# 对象属性
{{ user_info.name }}

# 数组元素
{{ items[0] }}

# 数组长度
{{ items | length }}
```

### 模板转换节点

模板节点用于格式化输出，生成结构化的文本内容。

**JSON输出模板：**

```yaml
template: |
  {
    "query": "{{ query }}",
    "answer": "{{ llm_output }}",
    "sources": [
      {% for doc in knowledge_retrieval.documents %}
      {
        "title": "{{ doc.metadata.title }}",
        "content": "{{ doc.content }}",
        "relevance": {{ doc.score }}
      }{% if not loop.last %},{% endif %}
      {% endfor %}
    ],
    "metadata": {
      "model": "{{ model_name }}",
      "tokens_used": {{ token_count }},
      "processing_time_ms": {{ elapsed_ms }}
    }
  }
```

**Markdown输出模板：**

```yaml
template: |
  # 分析报告：{{ topic }}
  
  ## 摘要
  {{ summary }}
  
  ## 详细内容
  {{ content }}
  
  ## 参考资料
  {% for ref in references %}
  {{ loop.index }}. [{{ ref.title }}]({{ ref.url }})
  {% endfor %}
  
  ---
  生成时间：{{ timestamp }}
```

### HTTP请求节点

HTTP节点用于调用外部API。

**GET请求：**

```yaml
method: GET
url: "https://api.example.com/users/{{ user_id }}"
headers:
  Authorization: "Bearer {{ api_key }}"
timeout: 30
```

**POST请求：**

```yaml
method: POST
url: "https://api.example.com/analyze"
headers:
  Authorization: "Bearer {{ api_key }}"
  Content-Type: "application/json"
body:
  type: json
  data:
    text: "{{ user_input }}"
    language: "zh"
timeout: 30
```

**响应处理：**

```yaml
输出变量:
  - response: 响应体
  - response.data: 响应数据
  - response.status_code: 状态码
  - response.headers: 响应头
```

### 代码执行节点

代码节点让你能用Python或JavaScript写自定义逻辑。

**Python示例——数据清洗：**

```python
import json
import re

def main(params: dict) -> dict:
    # 获取输入
    text = params.get("text", "")
    user_id = params.get("user_id", "")
    
    # 数据清洗
    cleaned = text.strip()
    cleaned = re.sub(r'\s+', ' ', cleaned)  # 合并多余空格
    
    # 提取关键信息
    order_match = re.search(r'订单[号]?[:：]?\s*([A-Z0-9]{10,})', cleaned)
    order_id = order_match.group(1) if order_match else None
    
    # 关键词提取
    keywords = re.findall(r'\b\w{2,}\b', cleaned)
    
    return {
        "cleaned_text": cleaned,
        "order_id": order_id,
        "keywords": list(set(keywords))[:10],
        "has_order": bool(order_id),
        "word_count": len(cleaned)
    }
```

**Python示例——数据转换：**

```python
def main(params: dict) -> dict:
    # 获取数据
    raw_data = params.get("data", [])
    
    # 转换格式
    transformed = []
    for item in raw_data:
        transformed.append({
            "id": str(item.get("id", "")),
            "name": item.get("name", "").strip(),
            "value": float(item.get("value", 0)),
            "status": "active" if item.get("value", 0) > 0 else "inactive",
            "created_at": item.get("created_at", "")
        })
    
    # 统计分析
    total = sum(i["value"] for i in transformed)
    avg = total / len(transformed) if transformed else 0
    
    return {
        "items": transformed,
        "total": total,
        "average": round(avg, 2),
        "count": len(transformed)
    }
```

**JavaScript示例：**

```javascript
function main({params}) {
    const { text, threshold = 0.5 } = params;
    
    // 文本处理
    const cleaned = text.trim().replace(/\s+/g, ' ');
    
    // 分数计算
    const words = cleaned.split(' ');
    const scores = words.map(w => w.length / 10);
    const avgScore = scores.reduce((a, b) => a + b, 0) / scores.length;
    
    // 判断结果
    const passed = avgScore >= threshold;
    
    return {
        original: text,
        cleaned: cleaned,
        wordCount: words.length,
        averageScore: avgScore.toFixed(2),
        passed: passed,
        grade: avgScore >= 0.8 ? 'A' : avgScore >= 0.6 ? 'B' : 'C'
    };
}
```

### 迭代节点

迭代节点用于遍历数组，对每个元素执行相同操作。

**基础配置：**

```yaml
iterator: "{{ items }}"  # 要遍历的数组
cursor: "current_item"   # 当前元素变量名

# 迭代体
body:
  - llm: 处理当前元素
    input: "{{ current_item }}"
```

**实战——批量生成内容：**

```yaml
iterator: "{{ product_list }}"
cursor: "product"

body:
  - llm: 生成描述
    prompt: |
      为以下产品生成简短描述：
      产品名称：{{ product.name }}
      产品特点：{{ product.features }}
      
      要求：50字以内，突出卖点。
    output: description
  
  - variable: 收集结果
    operation: append
    value:
      product_name: "{{ product.name }}"
      description: "{{ description }}"
```

### 变量聚合节点

聚合节点用于合并多个分支的输出。

**基础用法：**

```yaml
sources:
  - node: knowledge_retrieval
    selector:
      - documents
  - node: web_search
    selector:
      - results

merge_mode: concat  # concat/union
```

### 结束节点

结束节点定义工作流的终止。

**配置：**

```yaml
type: common  # 普通结束

outputs:
  - name: result
    value: "{{ final_output }}"
  
  - name: status
    value: "success"
```

## DAG编排与控制流

### 顺序执行

最简单的模式，所有节点按顺序线性执行。

```
开始 → 数据预处理 → LLM生成 → 格式化输出 → 结束
```

**适用场景：**
- 数据处理流水线
- 简单的问答流程
- 单轮响应的工作流

### 并行执行

多个分支同时执行，最后合并。

```
                    → 分支A：知识库检索
                   /
开始 → 问题理解 ───┼──→ 分支B：意图分类
                   \
                    → 分支C：用户画像获取
                    
合并 → 整合输出 → 结束
```

**实战示例：**

```
开始
  ↓
┌──────────────────────┐
│     并行执行           │
│  ┌────────┬───────┐ │
│  ↓        ↓       ↓ │
│知识检索  用户画像  历史分析│
│  ↓        ↓       ↓ │
│  └────────┴───────┘ │
└──────────────────────┘
  ↓
整合输出 → 结束
```

### 条件分支

根据中间结果动态选择执行路径。

```
开始 → 问题分类
         │
    ┌────┴────┐
    ↓         ↓
技术问题    投诉问题
    ↓         ↓
技术专家   安抚策略
    ↓         ↓
解决方案   生成回复
    └────┬────┘
         ↓
       结束
```

**实战——智能客服分流：**

```yaml
条件分支配置:
  conditions:
    - if: "{{ intent }}" == "product_inquiry"
      then: product_flow
    - if: "{{ intent }}" == "after_sales"
      then: aftersales_flow
    - if: "{{ intent }}" == "complaint"
      then: complaint_flow
    - else: general_flow
```

### 循环执行

循环节点让工作流能够重复执行某个过程。

**循环结构：**

```
开始 → 初始化
         ↓
    ┌────┴────┐
    ↓         ↓
条件判断  继续执行
    ↓yes     ↓no
  执行逻辑   ↓
    ↓      结束
    ↓
  更新状态
    ↓
    ↓
    └──→ (回到条件判断)
```

**实战——质量检查循环：**

```
开始 → 生成内容
         ↓
    质量检查
         │
    ┌────┴────┐
    ↓         ↓
   通过      不通过
    ↓         ↓
   结束    调整策略 → 重新生成
                     ↓
                   质量检查 (最多5次)
```

### 容错机制

工作流中可能出现各种异常，需要合理的容错设计。

```
正常流程：
开始 → A → B → C → 结束

有错误处理：
开始 → A → B → C → 结束
           ↓错误
        错误处理 → 降级输出
```

## 错误处理与重试机制

### 节点级错误处理

每个节点都可以配置错误处理策略：

```yaml
error_handling:
  # 策略选择
  strategy: retry_then_continue
  
  # retry: 失败重试
  # skip: 跳过节点
  # continue: 继续执行
  # fallback: 使用默认值
  # fail: 直接失败
  
  # 重试配置
  retry:
    max_attempts: 3
    initial_delay: 1    # 秒
    max_delay: 30
    backoff: exponential  # 指数退避
  
  # 降级配置
  fallback:
    enabled: true
    use_default_value: true
    default_output: "抱歉，服务暂时不可用，请稍后再试。"
```

### 常见错误类型及处理

| 错误类型 | 典型原因 | 推荐策略 |
|---------|---------|----------|
| LLM超时 | 模型响应慢 | 重试3次，超时后降级 |
| API限流 | 请求频率超限 | 指数退避，加入队列 |
| 知识库无结果 | 检索失败 | 放宽条件或通用回答 |
| 参数验证失败 | 格式错误 | 跳过，使用空值 |
| 外部服务异常 | 服务不可用 | 降级到备用服务 |

### 全局错误处理

```yaml
工作流级别配置:
  on_error:
    # 记录错误日志
    log_error: true
    
    # 发送告警
    notify:
      channels:
        - email
        - webhook
      threshold: 3  # 连续3次才通知
    
    # 自动恢复
    recovery:
      enabled: true
      cleanup_on_error: true
```

## 调试技巧与最佳实践

### 单节点调试

1. 选择要调试的节点
2. 点击"测试此节点"
3. 提供测试输入数据
4. 查看输出和日志
5. 调整配置

### 断点与步骤执行

```
调试模式:
  - 设置断点在关键节点
  - 单步执行，观察每一步的变量状态
  - 在任意节点注入测试数据
  - 回放执行历史
```

### 性能分析

```json
{
  "execution_summary": {
    "total_duration_ms": 3245,
    "node_executions": [
      {"node": "user_input", "duration_ms": 12, "status": "success"},
      {"node": "intent_classify", "duration_ms": 156, "status": "success"},
      {"node": "knowledge_retrieval", "duration_ms": 892, "status": "success"},
      {"node": "llm_generate", "duration_ms": 2185, "status": "success"}
    ]
  }
}
```

**性能瓶颈识别：**
- 知识库检索慢 → 优化索引、增加缓存
- LLM响应慢 → 选更快的模型、减少输入token
- 代码节点慢 → 优化算法、减少计算

## 高级应用案例

### 案例一：智能客服工作流

**需求：** 自动处理客户咨询，分流到不同处理流程。

```
开始
  │
  ↓
┌─────────────────────────────────────────┐
│ 1. 用户意图识别                          │
│    - 分析问题类型（咨询/投诉/售后/退款）     │
│    - 提取关键实体（订单号、产品名等）        │
└─────────────────────────────────────────┘
  │
  ↓
条件分流
  │
  ├─ 咨询 → 知识库检索 → 生成回答
  ├─ 投诉 → 情绪分析 → 安抚策略 → 生成回复
  └─ 退款 → 订单验证 → 退款评估 → 审批通知
  │
  ↓
答案质量检查
  │
  ├─ 通过 → 输出答案
  └─ 不通过 → 重新生成（最多2次）
  │
  ↓
保存会话记录 → 结束
```

**节点详细配置：**

**1. 意图识别LLM**
```yaml
model: gpt-4o-mini
prompt: |
  分析用户消息的意图：
  - inquiry: 售前咨询
  - complaint: 投诉建议
  - refund: 退款申请
  - aftersales: 售后问题
  - other: 其他
  
  用户消息：{{ user_message }}
  
  只返回意图类别名称。
```

**2. 知识库检索**
```yaml
knowledge_base_id: product_kb
retrieval_model:
  top_k: 3
  score_threshold: 0.7
  mode: hybrid
```

**3. 情绪分析LLM**
```yaml
model: gpt-4o
prompt: |
  分析用户消息的情绪强度：
  
  用户消息：{{ user_message }}
  
  返回JSON：
  {
    "emotion": "angry/frustrated/neutral/positive",
    "intensity": 1-5,
    "reason": "简短原因说明"
  }
```

**4. 答案质量检查**
```yaml
conditions:
  - if: "{{ confidence }}" >= 0.8
    then: pass
  - if: "{{ confidence }}" >= 0.5
    then: regenerate
  - else: escalate
```

### 案例二：多模态内容审核

**需求：** 对用户上传的内容进行多维度审核。

```
开始 → 内容解析 → 文本审核 → 图像审核 → 综合判定 → 输出结果
                 ↓
           ├─ 敏感词检测
           ├─ 违规内容识别
           └─ 质量评分
```

**节点配置：**

**1. 内容解析**
```yaml
type: code
python: |
  import json
  
  content = params.get("content", {})
  
  return {
      "text": content.get("text", ""),
      "images": content.get("images", []),
      "has_text": bool(content.get("text")),
      "has_images": bool(content.get("images")),
      "content_type": "mixed" if content.get("text") and content.get("images") else ("text" if content.get("text") else "image")
  }
```

**2. 文本审核LLM**
```yaml
model: gpt-4o
prompt: |
  判断以下文本是否包含违规内容：
  
  文本：{{ text_analysis.text }}
  
  违规类型：
  - 政治敏感
  - 色情低俗
  - 暴力血腥
  - 虚假信息
  - 侵权内容
  
  返回JSON：
  {
    "is_safe": true/false,
    "violations": ["违规类型列表"],
    "confidence": 0-1,
    "suggestion": "通过/需人工审核/拒绝"
  }
```

**3. 综合判定**
```yaml
conditions:
  - if: "{{ text_result.is_safe }} == true AND {{ image_result.is_safe }} == true"
    then: approve
  - if: "{{ text_result.suggestion }} == '需人工审核' OR {{ image_result.suggestion }} == '需人工审核'"
    then: manual_review
  - else: reject
```

### 案例三：自动化报告生成

**需求：** 根据数据自动生成分析报告。

```
开始 → 数据获取 → 数据分析 → 报告生成 → 格式转换 → 通知发送
         ↓
    ┌─────┴─────┐
    ↓           ↓
数据库查询   API获取
    ↓           ↓
    └─────┬─────┘
          ↓
      数据清洗
```

**节点配置：**

**1. 数据库查询**
```yaml
type: http_request
method: POST
url: "{{ database_api_url }}"
body:
  query: |
    SELECT 
      date,
      SUM(sales) as total_sales,
      COUNT(DISTINCT customer_id) as customers,
      AVG(order_value) as avg_order
    FROM orders
    WHERE date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY)
    GROUP BY date
    ORDER BY date
```

**2. 数据分析LLM**
```yaml
model: gpt-4o
prompt: |
  分析以下销售数据，找出关键洞察：
  
  数据：
  {{ database_query.output }}
  
  请分析：
  1. 销售趋势（上升/下降/平稳）
  2. 主要增长点
  3. 潜在问题
  4. 建议措施
  
  返回JSON格式，包含分析结论和建议。
```

**3. 报告生成**
```yaml
model: gpt-4o
prompt: |
  基于以下分析，生成完整的日报：
  
  分析结果：
  {{ data_analysis.output }}
  
  要求：
  1. 结构清晰，包含摘要、正文、建议
  2. 使用Markdown格式
  3. 数据可视化用表格展示
  4. 篇幅500-1000字
```

### 案例四：RAG增强问答

**需求：** 结合知识库和实时信息进行智能问答。

```
开始 → 查询理解 → ┬→ 知识库检索
                ├→ 网络搜索
                └→ 历史对话
                      ↓
                 结果融合
                      ↓
                 生成回答
                      ↓
                 质量验证 → 输出
                      ↓
                 不通过 → 重新生成
```

**节点配置：**

**1. 查询理解LLM**
```yaml
model: gpt-4o
prompt: |
  分析用户查询，提取关键信息：
  
  用户查询：{{ user_query }}
  
  返回JSON：
  {
    "keywords": ["关键词列表"],
    "intent": "简单查询/复杂分析/对比分析",
    "requires_realtime": true/false,
    "requires_knowledge": true/false
  }
```

**2. 知识库检索**
```yaml
conditions:
  - if: "{{ query_understanding.requires_knowledge }}"
    then: knowledge_retrieval
  - else: skip_knowledge
```

**3. 网络搜索**
```yaml
conditions:
  - if: "{{ query_understanding.requires_realtime }}"
    then: web_search
  - else: skip_web
```

**4. 结果融合**
```yaml
type: code
python: |
  knowledge = params.get("knowledge_results", [])
  web_results = params.get("web_results", [])
  
  # 合并去重
  all_results = knowledge + web_results
  
  # 按相关性排序
  sorted_results = sorted(
      all_results, 
      key=lambda x: x.get("score", 0), 
      reverse=True
  )
  
  return {
      "merged_results": sorted_results[:10],
      "has_knowledge": bool(knowledge),
      "has_web_info": bool(web_results)
  }
```

## 工作流设计模式

### 模式一：ETL流水线

Extract（提取）→ Transform（转换）→ Load（加载）

```
数据源 → 提取 → 清洗 → 转换 → 加载 → 目标
```

### 模式二：Fan-out/Fan-in

分散处理 → 聚合结果

```
                → 分支1处理
               /
输入 → 分散 ──→ 分支2处理 → 聚合 → 输出
               \
                → 分支3处理
```

### 模式三：Saga模式

分布式事务处理

```
开始 → 步骤1 → 步骤2 → 步骤3 → 完成
   ↓错误      ↓错误      ↓错误
补偿1   补偿2    补偿3
   ↓         ↓         ↓
  回滚 ←──────┼──────→ 回滚
```

### 模式四：管道过滤

数据流式处理

```
输入 → 过滤1 → 过滤2 → 转换 → 格式化 → 输出
```

## 总结与进阶路径

### 学习路径

**入门阶段：**
- 熟悉各类节点的功能和基本配置
- 理解工作流的执行逻辑
- 能构建简单的线性工作流

**进阶阶段：**
- 掌握条件分支、循环、并行执行
- 学会设计复杂的多路径工作流
- 能处理各种异常情况

**精通阶段：**
- 进行工作流性能优化
- 设计可复用的工作流模板
- 建立完善的设计规范和最佳实践

### 资源推荐

- Dify官方示例工作流
- Dify社区分享
- GitHub上的开源工作流

---

## 相关文档

- [[dify平台深度指南]] - Dify平台整体介绍
- [[智能体搭建]] - AI Agent核心概念
- [[工作流设计模式]] - 通用工作流设计原则
- [[多Agent系统设计]] - 多Agent协作的工作流设计

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
