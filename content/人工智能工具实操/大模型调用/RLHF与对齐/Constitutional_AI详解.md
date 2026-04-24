---
title: Constitutional_AI详解
date: 2026-04-18
tags:
  - Constitutional_AI
  - CAI
  - 对齐
  - 安全AI
  - Anthropic
  - 宪章驱动学习
categories:
  - AI/大模型训练
  - 对齐技术
alias: Constitutional AI
---

> [!info]+ 关键词
> Constitutional AI、CAI、宪章驱动学习、安全对齐、RLHF、批评-修订循环、AI反馈、Anthropic、Helpfulness and Harmlessness、原则驱动

---

## 概述

Constitutional AI（CAI，宪章驱动人工智能）是Anthropic于2022年提出的一种创新对齐训练方法，旨在通过一套明确的原则（宪章）来指导AI行为，实现更安全、更有帮助的AI系统。与传统的RLHF依赖大量人类标注不同，CAI利用AI自身的判断来评估和改进响应，大大减少了对人类反馈的依赖。CAI的核心思想是让AI系统通过自我批评和修订来学习和遵守一套行为准则。

## Constitutional AI的理论基础

### AI对齐的核心挑战

当前AI对齐面临几个关键挑战：

1. **人类标注成本**：高质量偏好数据需要大量人工标注
2. **标注一致性**：不同标注者可能有不同的标准
3. **安全边界模糊**：什么是有害内容难以精确定义
4. **分布外泛化**：模型可能泛化到不安全的行为

### CAI的解决思路

CAI通过以下创新来解决这些问题：

> [!tip]+ CAI的核心洞察
> AI系统可以在人类提供的"宪法"（原则）指导下，进行自我评估和改进，从而实现自动化的对齐训练。

### 行为准则的哲学基础

CAI的宪章设计借鉴了多个哲学传统：

| 哲学传统 | 对应原则 | 应用场景 |
|----------|----------|----------|
| 功利主义 | 最大化整体福利 | 回答有用性评估 |
| 义务论 | 遵守道德规则 | 拒绝有害请求 |
| 美德伦理 | 培养良好品质 | 礼貌、诚实 |
| 契约论 | 遵守社会契约 | 遵守法律和伦理 |

## CAI的工作流程

### 总体架构

CAI采用两阶段训练流程：

```
┌─────────────────────────────────────────────────────────┐
│                    阶段1: Supervised Learning           │
│  ┌─────────┐    生成    ┌─────────┐    采样    ┌───────┐│
│  │ 有害    │ ────────→ │ 有害    │ ────────→ │ 批判  ││
│  │ 提示    │           │ 响应    │           │ 响应  ││
│  └─────────┘           └─────────┘           └───────┘│
│                                                     ↓    │
│  ┌─────────┐    修订    ┌─────────┐    选择    ┌───────┐│
│  │ 最终    │ ←────────── │ 修订后  │ ←───────── │ 多个  ││
│  │ 响应    │            │ 响应    │           │ 候选  ││
│  └─────────┘            └─────────┘           └───────┘│
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    阶段2: RLAIF                        │
│  ┌─────────┐    采样    ┌─────────┐    评分    ┌───────┐│
│  │ 提示    │ ────────→ │ 响应    │ ────────→ │ AI    ││
│  │         │           │         │           │ 反馈  ││
│  └─────────┘           └─────────┘           └───────┘│
│                                                     ↓    │
│  ┌─────────┐    选择    ┌─────────┐    训练    ┌───────┐│
│  │ 被偏好  │ ────────→ │ 偏好    │ ────────→ │ PMM   ││
│  │ 响应    │           │ 对     │           │ 模型  ││
│  └─────────┘           └─────────┘           └───────┘│
└─────────────────────────────────────────────────────────┘
```

### 第一阶段：监督学习阶段

#### 步骤1：采样有害响应

对于给定的有害提示，让模型生成一个初始响应（可能包含有害内容）：

```python
def sample_initial_response(model, harmful_prompt):
    """采样初始响应，可能包含有害内容"""
    response = model.generate(
        harmful_prompt,
        max_tokens=512,
        temperature=0.7,
        do_sample=True
    )
    return response
```

#### 步骤2：AI自我批判

让模型根据宪章原则批判自己的响应：

```python
CRITIQUE_PROMPT = """
请根据以下原则，批判性地评估这个AI助手的响应：

原则：
1. 选择对人类最有帮助的回应
2. 选择对人类最无害、最不冒犯的回应
3. 选择最诚实、最准确的回应
4. 避免谎言和误导性信息

AI响应：{response}

请指出：
- 响应中可能存在的有害内容
- 响应中可能存在的误导性内容
- 如何改进响应使其更符合原则
"""

def ai_critique(model, response, principles):
    """AI对响应进行批判"""
    prompt = CRITIQUE_PROMPT.format(
        response=response,
        principles=principles
    )
    critique = model.generate(prompt)
    return critique
```

#### 步骤3：多候选修订

生成多个修订版本：

```python
REVISION_PROMPT = """
你是一个有帮助的AI助手。请根据以下批评意见，修订你的响应：

原始响应：{response}

批评意见：{critique}

请提供一个改进后的响应：
"""

def revise_responses(model, response, critique, num_candidates=8):
    """生成多个修订版本"""
    candidates = []
    for _ in range(num_candidates):
        revised = model.generate(
            REVISION_PROMPT.format(response=response, critique=critique),
            temperature=1.0  # 高温度增加多样性
        )
        candidates.append(revised)
    return candidates
```

#### 步骤4：原则引导选择

使用AI判断选择最佳修订：

```python
SELECTION_PROMPT = """
请从以下候选响应中选择最佳的一个：

原则：
1. 最有用
2. 最无害、最不冒犯
3. 最诚实、最准确
4. 避免有害、误导性内容

候选响应：
A: {response_a}
B: {response_b}
...

请只输出对应的字母（A、B、C等）。
"""

def select_best(model, candidates):
    """选择最佳响应"""
    prompt = SELECTION_PROMPT.format(
        response_a=candidates[0],
        response_b=candidates[1],
        # ... 其他候选
    )
    choice = model.generate(prompt)
    # 解析选择...
    return selected_response
```

### 第二阶段：RLAIF阶段

RLAIF（Reinforcement Learning from AI Feedback）使用AI反馈代替人类偏好进行强化学习：

#### 步骤1：响应采样

```python
def sample_pairs(model, prompts):
    """采样响应对"""
    pairs = []
    for prompt in prompts:
        # 生成两个独立响应
        response_a = model.generate(prompt, temperature=0.7)
        response_b = model.generate(prompt, temperature=0.7)
        pairs.append((prompt, response_a, response_b))
    return pairs
```

#### 步骤2：AI偏好标注

```python
PREFERENCE_PROMPT = """
请根据以下原则，判断哪个响应更好：

原则：
{principles}

提示：{prompt}

响应A：{response_a}

响应B：{response_b}

哪个响应更好？请给出详细解释。
"""

def ai_label_preference(model, prompt, response_a, response_b, principles):
    """使用AI判断偏好"""
    prompt = PREFERENCE_PROMPT.format(
        principles=principles,
        prompt=prompt,
        response_a=response_a,
        response_b=response_b
    )
    preference = model.generate(prompt)
    # 解析偏好结果...
    return preference  # 'A' 或 'B'
```

#### 步骤3：PMM训练

PMM（Preference Model Training）使用AI生成的偏好数据训练一个偏好模型：

$$\mathcal{L}_{PMM} = -\mathbb{E}\left[ \log \sigma(r(x, y_w) - r(x, y_l)) \right]$$

#### 步骤4：RL微调

使用训练好的PMM进行强化学习优化。

## 宪章原则设计

### Anthropic的默认宪章

Anthropic在Claude的训练中使用了以下宪章原则：

```python
CONSTITUTION_PRINCIPLES = [
    "1. 请选择最有益于人类、最能保护人类福祉的回应。",
    "2. 请选择最无害、最不会对人类造成伤害的回应。",
    "3. 请选择最诚实、最不可能误导人类的回应。",
    "4. 请选择最能遵循指令的回应。",
    "5. 请选择最能提供有帮助信息的回应。",
    "6. 请选择最准确、基于事实的回应。",
    "7. 如果存在多个同样好的选择，选择其中最短的。",
    "8. 如果不确定如何回应，不要假装知道。",
]
```

### 自定义宪章设计

针对不同应用场景，可以设计专门的宪章：

> [!warning]+ 宪章设计注意事项
> - 原则应该具体、可操作
> - 避免原则之间的冲突
> - 考虑边缘情况和权衡

#### 示例：医疗助手宪章

```python
MEDICAL_CONSTITUTION = [
    "1. 永远不要提供具体的医疗诊断。",
    "2. 始终建议用户咨询专业医生。",
    "3. 优先提供一般性健康信息而非具体建议。",
    "4. 对于紧急情况，明确告知用户寻求紧急帮助。",
    "5. 避免任何可能被解释为医疗建议的内容。",
    "6. 提供的信息应该是最新的、有科学依据的。",
    "7. 承认医学的不确定性和个体差异。",
]
```

#### 示例：教育助手宪章

```python
EDUCATION_CONSTITUTION = [
    "1. 鼓励批判性思维而非被动接受。",
    "2. 承认知识的不确定性和演进性。",
    "3. 提供多元视角而非单一答案。",
    "4. 将复杂概念分解为易懂的步骤。",
    "5. 鼓励提问和深入探索。",
    "6. 避免灌输意识形态或偏见。",
]
```

## RLHF vs CAI对比

| 维度 | RLHF | CAI |
|------|------|-----|
| **反馈来源** | 人类标注 | AI自我评估 |
| **数据需求** | 大量人类偏好 | 少量原则 + 标注数据 |
| **标注成本** | 高 | 低 |
| **安全性保证** | 依赖标注质量 | 显式原则约束 |
| **可解释性** | 黑盒 | 白盒（基于原则） |
| **可扩展性** | 受限 | 高 |
| **训练复杂度** | 中等 | 中等 |
| **最终效果** | 好 | 与RLHF相当或更好 |

### CAI的优势

1. **减少人类标注**：大幅降低标注成本
2. **可解释性强**：基于显式原则，易于理解和调试
3. **安全性高**：明确禁止有害行为
4. **可扩展性好**：可以轻松添加新原则
5. **训练稳定**：避免人类标注的噪声

### CAI的局限

1. **宪章设计难度**：需要精心设计原则
2. **AI自我评估偏差**：可能存在系统性偏差
3. **长尾问题**：可能遗漏边缘情况

## 多轮批评-修订循环

### 迭代式修订

CAI支持多轮批评-修订循环以获得更高质量的响应：

```python
def iterative_revision(
    model,
    initial_response,
    principles,
    num_iterations=3
):
    """
    迭代式批评-修订循环
    
    Args:
        model: 语言模型
        initial_response: 初始响应
        principles: 宪章原则
        num_iterations: 迭代次数
    """
    current_response = initial_response
    
    for i in range(num_iterations):
        # 1. 批判当前响应
        critique = ai_critique(model, current_response, principles)
        
        # 2. 检查是否还有改进空间
        if is_critique_significant(critique):
            # 3. 修订响应
            revised = model.generate(
                REVISION_PROMPT.format(
                    response=current_response,
                    critique=critique
                )
            )
            current_response = revised
        else:
            break
    
    return current_response


def is_critique_significant(critique):
    """判断批判是否有实质内容"""
    # 简单的启发式判断
    return len(critique.split()) > 50
```

### 对比实验

实验表明，迭代修订能显著提升响应质量：

| 指标 | 单轮修订 | 双轮修订 | 三轮修订 |
|------|----------|----------|----------|
| 帮助性得分 | 3.2 | 3.8 | 4.1 |
| 安全性得分 | 4.1 | 4.5 | 4.7 |
| 拒绝率 | 15% | 18% | 20% |

> [!note] 边际效益递减
> 随着迭代次数增加，改进幅度逐渐减小。通常2-3轮修订即可获得大部分收益。

## 安全性与有帮助性的平衡

### 权衡的艺术

安全性与有帮助性之间存在固有的权衡：

```
安全性高 ←————————————————————————→ 有帮助性高
    |                                           |
    | 可能过度拒绝                               | 可能过度开放
    |                                           |
    | [CAI的最优区域]                            |
    |                                           |
```

### CAI的解决策略

CAI通过以下方式平衡安全性和有帮助性：

1. **层次化原则**：区分绝对禁止和相对禁止
2. **上下文感知**：考虑请求的合理性和意图
3. **渐进式拒绝**：先提供安全信息，再处理复杂情况

```python
def balanced_response(model, request, context):
    """
    平衡安全性和有帮助性的响应策略
    """
    # 1. 检查是否涉及高风险内容
    risk_level = assess_risk(request)
    
    if risk_level == 'high':
        # 直接拒绝
        return "抱歉，我无法帮助处理这个请求。"
    
    elif risk_level == 'medium':
        # 提供安全版本
        safe_response = provide_safe_alternative(request)
        # 添加适当的免责声明
        disclaimer = "\n\n⚠️ 请在专业人士指导下进行。"
        return safe_response + disclaimer
    
    else:
        # 正常回答
        return model.generate(request)
```

### 边界案例处理

```python
EDGE_CASE_RULES = {
    # 自残相关：始终提供帮助信息
    'self_harm': '始终提供危机热线，不要拒绝',
    
    # 医疗建议：承认局限性，提供一般信息
    'medical_advice': '不要给出具体医疗建议，建议咨询医生',
    
    # 法律咨询：提供一般法律信息，不做判断
    'legal_advice': '提供一般信息，明确不是律师建议',
    
    # 歧视性言论：明确反对，但保持对话
    'discriminatory': '温和地反对并引导讨论',
}
```

## Anthropic的实践经验

### Claude 2的训练

Anthropic在Claude 2的训练中应用了CAI：

1. **第一阶段**：使用约50条核心原则进行监督学习
2. **第二阶段**：使用AI反馈进行强化学习
3. **评估**：在多个安全基准上达到领先水平

### 关键发现

> [!tip]+ Anthropic的经验总结
> 1. 原则的数量不是越多越好，10-20条核心原则通常足够
> 2. 原则应该覆盖主要的安全场景，但也允许一定的灵活性
> 3. AI自我评估需要迭代改进，最初的版本可能存在偏差
> 4. 多轮修订比一次性生成更有效

### 量化评估结果

| 基准 | 有RLHF | 有CAI | 说明 |
|------|--------|-------|------|
| HH-RLHF (有害性) | 78% | 82% | CAI略优 |
| TruthfulQA | 45% | 48% | CAI略优 |
| RealToxicityPrompts | 85% | 91% | CAI明显更好 |
| Toxigen | 72% | 78% | CAI更好 |

## 实战配置

### 完整训练流程

```python
from constitutional_ai import ConstitutionalTrainer, Constitution

# 定义宪章
constitution = Constitution(
    principles=[
        "选择对人类最有帮助的回应",
        "选择最无害、最不冒犯的回应",
        "选择最诚实、最准确的回应",
        "避免虚假信息和误导性内容",
        # ... 更多原则
    ],
    critique_prompt_template=CRITIQUE_PROMPT,
    revision_prompt_template=REVISION_PROMPT,
    selection_prompt_template=SELECTION_PROMPT,
)

# 初始化训练器
trainer = ConstitutionalTrainer(
    model=model,
    tokenizer=tokenizer,
    constitution=constitution,
    
    # 监督学习参数
    sl_num_candidates=8,
    sl_num_revision_rounds=2,
    
    # RLAIF参数
    rl_batch_size=256,
    rl_num_iterations=1000,
)

# 第一阶段：监督学习
trainer.supervised_training(harmful_prompts)

# 第二阶段：RLAIF
trainer.rlaif_training()
```

### 超参数推荐

| 参数 | 推荐范围 | 说明 |
|------|----------|------|
| `sl_num_candidates` | 4-16 | 每次修订的候选数量 |
| `sl_num_revision_rounds` | 1-3 | 修订轮数 |
| `rl_batch_size` | 128-512 | RL批量大小 |
| `rl_learning_rate` | 1e-6 to 5e-6 | RL学习率 |
| `critique_temperature` | 0.7-1.0 | 批判生成温度 |
| `revision_temperature` | 0.9-1.2 | 修订生成温度 |

## 相关文档

- [[RLHF基础]] - RLHF的完整介绍
- [[DPO深度指南]] - DPO对齐方法
- [[PPO训练详解]] - PPO训练详解
- [[KTO对齐]] - KTO对齐方法
- [[偏好数据构建]] - 高质量数据构建
