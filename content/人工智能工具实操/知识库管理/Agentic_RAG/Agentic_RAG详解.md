---
title: Agentic RAG详解
date: 2026-04-24
tags:
  - Agentic RAG
  - 智能RAG
  - 自适应RAG
  - ReAct
  - 迭代检索
categories:
  - 人工智能工具实操/知识库管理
aliases:
  - Agentic RAG
  - 智能RAG
---

# Agentic RAG详解：给RAG装上大脑

> [!tip] 这篇文章解决什么问题
> 传统RAG遇到复杂问题就卡壳，查不到正确答案怎么办？Agentic RAG就是来解决这个问题的——它能让RAG系统像真人一样思考，遇到问题会反思、会换策略、会多步推理。

## 前言：传统RAG的痛点

先说说我踩过的坑。

用传统RAG做企业知识库问答时，遇到这类问题就傻眼：

**问题1：多跳推理**
> 问："张三和李四在同一个项目组吗？他们合作过什么产品？"

传统RAG只能处理单跳问题，这种需要先查张三、再查李四、再查他们关系的复合问题，根本搞不定。

**问题2：假正例**
> 问："深度学习框架有哪些？"
> 
> 检索结果返回了一堆相关内容，但其中一条讲的是"深度学习在游戏中的应用"，另一条讲的是"深度学习的数学基础"——都是相关内容，但都不是你真正想要的"框架列表"。

**问题3：检索失败就摆烂**
> 
> 传统RAG检索到什么就返回什么，检索质量差就认命了，不会想办法补救。

Agentic RAG就是来解决这些问题的。

## 一、Agentic RAG是什么

### 1.1 一句话解释

Agentic RAG就是**给RAG系统装上一个AI大脑**，让它能自主决策、动态调整、反思纠错。

### 1.2 和传统RAG的区别

| 维度 | 传统RAG | Agentic RAG |
|------|---------|-------------|
| 检索策略 | 固定 | 动态选择 |
| 遇到问题 | 摆烂 | 反思重试 |
| 复杂问题 | 搞不定 | 分解处理 |
| 跨数据源 | 不支持 | 支持 |
| 结果质量 | 听天由命 | 主动优化 |

### 1.3 打个比方

**传统RAG**像一个只会机械执行的员工：
- 问什么就查什么
- 查不到就说"没有"
- 从不思考有没有其他办法

**Agentic RAG**像一个会思考的顾问：
- 先分析问题是什么类型
- 决定用什么策略去查
- 查不到会反思是不是策略有问题
- 会尝试不同的方法
- 最后整合信息给出答案

## 二、核心设计模式

### 2.1 ReAct模式

ReAct（Reasoning + Acting）是最经典的Agentic RAG模式。

**核心思想**：思考→行动→观察→再思考→再行动...

```python
# ReAct的思考流程示例
async def react_query(question):
    context = []
    
    # 第一轮思考
    thought = "这个问题需要先找到张三和李四的信息"
    action = "search_kg(query='张三 李四')"
    result = await execute(action)
    context.append(result)
    
    # 第二轮思考
    thought = "找到了他们的信息，现在需要找到合作过的产品"
    if "同一个项目组" in question:
        action = "search_kg(query='张三 李四 合作')"
        result = await execute(action)
        context.append(result)
    
    # 整合答案
    answer = await generate_answer(question, context)
    return answer
```

### 2.2 Self-RAG模式

Self-RAG让模型自己反思检索结果。

**核心思想**：生成答案前，先问自己"检索结果够不够用？"

```python
async def self_rag_query(question):
    # 第一次检索
    results = await search(question)
    
    # 反思检索质量
    reflection = await llm.evaluate("""
        用户问题：{question}
        检索结果：{results}
        
        检索结果是否足够回答这个问题？
        如果不够，还缺什么信息？
    """)
    
    # 根据反思决定下一步
    if reflection["is_sufficient"]:
        return await generate_answer(question, results)
    else:
        # 补充检索缺失的信息
        more_results = await search(reflection["missing_aspects"])
        all_results = merge(results, more_results)
        return await generate_answer(question, all_results)
```

### 2.3 CRAG模式

CRAG（Corrective RAG，纠错RAG）专注于检索质量控制。

**核心思想**：检索结果质量差就触发纠错。

```python
async def crag_query(question):
    results = await search(question)
    
    # 检查检索质量
    quality = evaluate_retrieval_quality(results, question)
    
    if quality > 0.8:
        # 质量好，直接生成
        return await generate_answer(question, results)
    
    elif quality > 0.5:
        # 质量一般，尝试优化
        improved_results = await improve_query(question, results)
        return await generate_answer(question, improved_results)
    
    else:
        # 质量太差，换策略重查
        alternative_results = await search_alternative(question)
        return await generate_answer(question, alternative_results)
```

## 三、子问题分解

### 3.1 为什么要分解

复杂问题不能直接查，需要拆成简单问题。

比如问："张三和李四在同一个项目组吗？他们合作过什么产品？"

拆成：
1. 张三在哪个项目组？
2. 李四在哪个项目组？
3. 张三和李四合作过什么产品？

### 3.2 分解示例

```python
async def decompose_question(question):
    """将复杂问题分解为简单问题"""
    
    prompt = f"""
请将以下问题分解为可以独立回答的简单问题：

问题：{question}

要求：
1. 每个子问题应该能单独检索和回答
2. 标注子问题之间的依赖关系
3. 按逻辑顺序排列

请以JSON格式返回：
{{
    "sub_questions": [
        {{"id": 1, "question": "...", "depends_on": []}},
        {{"id": 2, "question": "...", "depends_on": [1]}}
    ]
}}
"""
    
    result = await llm.generate(prompt)
    return json.loads(result)
```

### 3.3 按顺序执行

```python
async def answer_decomposed(question):
    """按依赖顺序回答子问题"""
    
    # 分解问题
    sub_questions = await decompose_question(question)
    
    answers = {}
    
    # 按依赖顺序执行
    for sq in sub_questions:
        # 先等依赖的子问题完成
        if sq["depends_on"]:
            await asyncio.gather(*[answers[d] for d in sq["depends_on"]])
        
        # 执行子问题检索
        results = await search(sq["question"])
        answer = await generate_answer(sq["question"], results)
        answers[sq["id"]] = answer
    
    # 整合最终答案
    return await synthesize(question, answers)
```

## 四、自适应检索策略

### 4.1 为什么要自适应

不同问题适合不同的检索策略：
- 具体概念问题 → 语义检索
- 精确术语问题 → 关键词检索
- 关系问题 → 知识图谱检索

Agentic RAG会根据问题类型自动选择策略。

### 4.2 策略选择

```python
def select_retrieval_strategy(question):
    """根据问题特征选择检索策略"""
    
    # 关键词检测
    semantic_keywords = ["是什么", "为什么", "如何", "原理"]
    keyword_keywords = ["定义", "术语", "API", "参数"]
    kg_keywords = ["关系", "区别", "对比", "谁"]
    
    text = question.lower()
    
    if any(kw in text for kw in semantic_keywords):
        return "semantic"  # 语义检索
    elif any(kw in text for kw in keyword_keywords):
        return "keyword"  # 关键词检索
    elif any(kw in text for kw in kg_keywords):
        return "knowledge_graph"  # 知识图谱检索
    else:
        return "hybrid"  # 混合检索
```

### 4.3 动态调整

```python
async def adaptive_retrieve(question):
    """自适应检索"""
    
    strategy = select_retrieval_strategy(question)
    
    if strategy == "semantic":
        results = await vector_search(question)
    elif strategy == "keyword":
        results = await bm25_search(question)
    elif strategy == "knowledge_graph":
        results = await kg_search(question)
    else:
        results = await hybrid_search(question)
    
    # 评估检索质量
    quality = evaluate(results, question)
    
    # 质量不够，尝试其他策略
    if quality < 0.6:
        alt_results = await search_with_alternative_strategy(question)
        results = merge_and_rerank(results, alt_results)
    
    return results
```

## 五、迭代优化

### 5.1 什么是迭代优化

简单说就是：一次检索不够，就多查几次。

```python
async def iterative_retrieve(question, max_iterations=3):
    """迭代检索"""
    
    all_results = []
    
    for i in range(max_iterations):
        # 选择策略
        strategy = select_retrieval_strategy(question)
        
        # 检索
        results = await search(question, strategy=strategy)
        all_results.extend(results)
        
        # 反思
        reflection = await reflect(question, results)
        
        if reflection["is_sufficient"]:
            break
        
        # 改进查询
        question = reflection["improved_query"]
    
    return merge_and_rerank(all_results)
```

### 5.2 反思机制

```python
async def reflect(question, results):
    """反思检索结果"""
    
    prompt = f"""
问题：{question}
检索结果：{results}

请反思：
1. 检索结果是否足够回答问题？
2. 是否有遗漏的重要信息？
3. 需要用什么关键词重新检索？

请以JSON格式返回：
{{
    "is_sufficient": true/false,
    "missing_aspects": ["..."],
    "improved_query": "..."
}}
"""
    
    result = await llm.generate(prompt)
    return json.loads(result)
```

## 六、实战代码

### 6.1 简单实现

```python
from enum import Enum
import asyncio

class RetrievalStrategy(Enum):
    SEMANTIC = "semantic"
    KEYWORD = "keyword"
    HYBRID = "hybrid"
    KNOWLEDGE_GRAPH = "knowledge_graph"

class AgenticRAG:
    """Agentic RAG简化实现"""
    
    def __init__(self, llm, retriever):
        self.llm = llm
        self.retriever = retriever
    
    async def query(self, question, mode="auto"):
        if mode == "quick":
            return await self._quick_query(question)
        else:
            return await self._deep_query(question)
    
    async def _quick_query(self, question):
        """快速查询：单次检索"""
        results = await self.retriever.search(question, top_k=5)
        answer = await self.llm.generate(question, results)
        return {"answer": answer, "mode": "quick"}
    
    async def _deep_query(self, question):
        """深度查询：多策略迭代"""
        # 1. 分析问题复杂度
        is_complex = self._is_complex(question)
        
        if is_complex:
            # 分解问题
            sub_questions = await self._decompose(question)
            
            # 并行检索子问题
            tasks = [self._quick_query(sq) for sq in sub_questions]
            sub_answers = await asyncio.gather(*tasks)
            
            # 整合答案
            answer = await self._synthesize(question, sub_answers)
        else:
            # 单次检索
            results = await self.retriever.search(question, top_k=10)
            
            # 检查质量
            quality = await self._check_quality(question, results)
            
            if quality < 0.6:
                # 重新检索
                results = await self._retry_with_improved_query(question)
            
            answer = await self.llm.generate(question, results)
        
        return {"answer": answer, "mode": "deep"}
    
    def _is_complex(self, question):
        """判断问题复杂度"""
        complexity_indicators = ["和", "与", "或者", "对比", "区别"]
        return sum(1 for ind in complexity_indicators if ind in question) >= 2
    
    async def _decompose(self, question):
        """分解问题"""
        prompt = f"将问题'{question}'分解为2-4个简单问题"
        result = await self.llm.generate(prompt)
        # 简化处理，实际应该用更复杂的解析
        return [question]  # 返回原问题作为占位
    
    async def _check_quality(self, question, results):
        """检查检索质量"""
        # 简化实现，实际应该用LLM评估
        return 0.8 if results else 0.0
    
    async def _retry_with_improved_query(self, question):
        """改进查询重新检索"""
        return await self.retriever.search(question, top_k=10)
    
    async def _synthesize(self, question, sub_answers):
        """整合子问题答案"""
        combined = "\n".join([a["answer"] for a in sub_answers])
        return await self.llm.generate(question, [combined])
```

### 6.2 使用示例

```python
# 初始化
rag = AgenticRAG(llm=your_llm, retriever=your_retriever)

# 快速查询
result = await rag.query("什么是深度学习", mode="quick")

# 深度查询
result = await rag.query(
    "张三和李四在同一个项目组吗？他们合作过什么产品？",
    mode="deep"
)

print(result["answer"])
```

## 七、什么时候用Agentic RAG

### 7.1 适合的场景

- **复杂多跳问题**：需要多个步骤推理
- **高精度要求**：医疗、法律、金融等领域
- **跨数据源**：需要整合多个知识库
- **用户期望高**：对回答质量要求严格

### 7.2 不适合的场景

- **简单问答**：问什么答什么的那种
- **延迟敏感**：每次查询都要几秒钟受不了
- **预算有限**：Agentic RAG调用LLM次数多，成本高
- **单跳问题**：传统RAG就够用了

### 7.3 升级建议

不要一上来就上Agentic RAG：

```
阶段1：先跑通传统RAG
   ↓ 验证需求
阶段2：加上重排
   ↓ 验证效果
阶段3：加上查询改写
   ↓ 验证召回
阶段4：最后才上Agentic RAG
```

## 八、常见问题

### 8.1 延迟太高怎么办

Agentic RAG因为要多次LLM调用，延迟可能比较高。

**解决方案**：
- 限制最大迭代次数
- 设置超时机制
- 快速模式跳过复杂推理
- 缓存常用问题的结果

### 8.2 成本太高怎么办

Agentic RAG的LLM调用次数可能是传统RAG的3-5倍。

**解决方案**：
- 用更便宜的模型处理简单任务
- 设置最大调用次数限制
- 缓存中间结果

### 8.3 效果不稳定怎么办

Agentic RAG的效果依赖LLM的推理质量，可能不稳定。

**解决方案**：
- 设置最低质量阈值
- 多策略并行取最优
- 人工审核高风险回答

## 九、总结

Agentic RAG是一个**强大的武器**，但不是所有场景都需要。

**核心价值**：
- 让RAG能处理复杂问题
- 自动选择最优检索策略
- 遇到问题会反思和重试

**使用建议**：
- 先用传统RAG验证需求
- 逐步升级到Agentic RAG
- 根据场景选择合适的复杂度

**记住**：杀鸡焉用牛刀，简单问题就别折腾Agentic RAG了。

## 相关主题

- [[知识库管理]] - RAG知识库整体架构
- [[GraphRAG]] - 知识图谱增强的RAG
- [[查询改写与扩展]] - Query处理优化
- [[知识库评估体系]] - RAG系统评估

---

> [!note] 更新记录
> - 2026-04-24：改写完成，语言风格优化
> - 简化代码，突出核心逻辑
