---
title: KTO对齐
date: 2026-04-18
tags:
  - RLHF
  - KTO
  - 对齐
  - Kahneman-Tversky优化
  - 人类偏好
categories:
  - AI/大模型训练
  - 对齐技术
alias: Kahneman-Tversky Optimization
---

> [!info]+ 关键词
> KTO、卡尼曼-特沃斯基优化、对齐、损失函数、偏好优化、FTD、SFT、行为经济学、损失厌恶、人类偏好建模

---

## 概述

KTO（Kahneman-Tversky Optimization，卡尼曼-特沃斯基优化）是由香港科技大学等机构于2024年提出的一种新型对齐训练方法。KTO的创新之处在于，它借鉴了行为经济学中的前景理论（Prospect Theory），将人类的损失厌恶（Loss Aversion）特性纳入对齐优化目标。与DPO专注于优化偏好相对顺序不同，KTO直接优化每个样本的"吸引力"，使其更符合人类直觉和决策模式。

## KTO的理论基础

### Kahneman-Tversky前景理论

行为经济学家Daniel Kahneman和Amos Tversky提出的前景理论解释了人类决策的诸多非理性特征：

1. **损失厌恶**：人类对损失的敏感度是同等收益的2-4倍
2. **敏感度递减**：边际效用随价值增加而递减
3. **参照点依赖**：评估基于相对而非绝对价值

### 前景理论的数学表达

Kahneman和Tversky将价值函数定义为：

$$v(x) = \begin{cases} x^\alpha & x \geq 0 \\ -\lambda(-x)^\beta & x < 0 \end{cases}$$

其中：
- $x$ 是相对于参照点的收益或损失
- $\alpha, \beta \approx 0.88$ 是敏感度参数
- $\lambda \approx 2.25$ 是损失厌恶系数

### 人类决策中的"是/否"二元性

KTO的核心洞察：人类决策不仅涉及"哪个更好"的相对判断，还涉及"是否可接受"的绝对判断。

- **DPO视角**：$y_w$ 是否比 $y_l$ 更好？
- **KTO视角**：$y$ 是否足够好，值得采纳？

## KTO的数学原理

### 目标分布建模

KTO定义了"目标分布"（Target Distribution）的概念。对于每个响应，我们希望最大化其被选择的概率，同时考虑人类对负面结果的更强厌恶。

### 损失函数推导

KTO的损失函数基于以下目标：

$$\mathcal{L}_{KTO}(\theta) = -\mathbb{E}_{(x, y) \sim \mathcal{D}} \left[ \mathbb{1}_{y \in \mathcal{Y}^+} \cdot \log \sigma(\beta \cdot f_\theta(x, y)) + \mathbb{1}_{y \in \mathcal{Y}^-} \cdot \log \sigma(-\beta \cdot f_\theta(x, y)) \right]$$

其中：
- $\mathcal{Y}^+$ 是被接受的响应集合
- $\mathcal{Y}^-$ 是被拒绝的响应集合
- $f_\theta(x, y)$ 是隐式价值函数
- $\sigma$ 是sigmoid函数

### 简化的FTD损失

KTO论文提供了更实用的损失函数变体：

$$\mathcal{L}_{FTD} = -\mathbb{E}_{x, y^+} \left[ \log \sigma(\beta \cdot u_\theta(x, y^+)) \right] - \mathbb{E}_{x, y^-} \left[ \log \sigma(-\beta \cdot u_\theta(x, y^-)) \right]$$

其中 $u_\theta(x, y) = \log \frac{\pi_\theta(y|x)}{\pi_{ref}(y|x)}$

> [!note] FTD vs 标准KTO
> FTD（Focal Distributional Treatment）损失简化了KTO的目标分布形式，使其更易于实现和调优。

### 损失厌恶参数

KTO引入了一个关键参数 $\lambda$ 来控制损失厌恶强度：

$$\mathcal{L}_\lambda = -\mathbb{E}_{y^+} \left[ \log \sigma(u_\theta) \right] - \lambda \cdot \mathbb{E}_{y^-} \left[ \log \sigma(-u_\theta) \right]$$

当 $\lambda > 1$ 时，模型会更加激进地避免生成被拒绝的响应。

## KTO vs DPO vs PPO对比

| 维度 | KTO | DPO | PPO |
|------|-----|-----|-----|
| **优化目标** | 绝对可接受性 | 相对偏好顺序 | 奖励最大化 |
| **数据类型** | 接受/拒绝标签 | 偏好对比对 | 奖励信号 |
| **训练阶段** | 单阶段 | 单阶段 | 三阶段 |
| **损失函数** | FTD损失 | 对比损失 | PPO-Clip |
| **损失厌恶建模** | ✅ 原生支持 | ❌ | ❌ |
| **实现复杂度** | 低 | 低 | 高 |
| **计算资源** | 中等 | 低 | 高 |
| **样本效率** | 高 | 高 | 中等 |
| **调参难度** | 中等 | 低 | 高 |

### KTO的独特优势

1. **无需偏好对**：只需知道响应是否可接受，无需成对比较
2. **损失厌恶建模**：更符合人类决策心理
3. **拒绝采样友好**：天然适配拒绝采样流程
4. **训练更稳定**：避免了对比学习中的一些数值问题

### KTO的适用场景

> [!tip]+ 何时选择KTO
> - 当偏好数据难以获取，但有大量单响应标注时
> - 当需要显式控制模型对"差响应"的惩罚强度时
> - 当希望模型更保守，避免生成明显错误的输出时

## 损失函数设计详解

### 基础FTD实现

```python
def ftd_loss(
    policy_logps: torch.Tensor,      # 策略模型的log概率
    ref_logps: torch.Tensor,          # 参考模型的log概率
    target: torch.Tensor,             # 1=接受, 0=拒绝
    beta: float = 0.1,
    lambda_: float = 1.0             # 损失厌恶参数
) -> torch.Tensor:
    """
    计算FTD损失函数
    
    Args:
        policy_logps: 策略模型对每个样本的log概率
        ref_logps: 参考模型的log概率
        target: 目标标签，1表示接受，0表示拒绝
        beta: 温度参数，控制偏离参考模型的程度
        lambda_: 损失厌恶参数
    """
    # 计算隐式效用
    u = beta * (policy_logps - ref_logps)
    
    # 接受样本的损失
    pos_loss = -target * torch.log(torch.sigmoid(u))
    
    # 拒绝样本的损失（带损失厌恶权重）
    neg_loss = -(1 - target) * lambda_ * torch.log(torch.sigmoid(-u))
    
    return (pos_loss + neg_loss).mean()
```

### 带置信度的KTO

对于标注置信度不同的样本，可以引入加权机制：

```python
def weighted_ftd_loss(
    policy_logps: torch.Tensor,
    ref_logps: torch.Tensor,
    target: torch.Tensor,
    confidence: torch.Tensor,          # 标注置信度 [0, 1]
    beta: float = 0.1,
    lambda_: float = 1.0
) -> torch.Tensor:
    u = beta * (policy_logps - ref_logps)
    
    # 使用置信度加权
    pos_loss = -target * confidence * torch.log(torch.sigmoid(u))
    neg_loss = -(1 - target) * confidence * lambda_ * torch.log(torch.sigmoid(-u))
    
    return (pos_loss + neg_loss).mean()
```

### 多层次损失厌恶

KTO支持不同粒度的损失厌恶设置：

```python
class MultiLevelKTO:
    def __init__(self, beta=0.1, levels=[1.0, 2.0, 4.0]):
        """
        levels对应不同风险等级的lambda值
        level 0: 安全样本 (lambda=1.0)
        level 1: 敏感样本 (lambda=2.0)  
        level 2: 高风险样本 (lambda=4.0)
        """
        self.beta = beta
        self.lambdas = {i: l for i, l in enumerate(levels)}
    
    def compute_loss(self, policy_logps, ref_logps, target, risk_level):
        u = self.beta * (policy_logps - ref_logps)
        lambda_ = self.lambdas.get(risk_level, 1.0)
        
        if target == 1:  # 接受样本
            return -torch.log(torch.sigmoid(u))
        else:  # 拒绝样本
            return -lambda_ * torch.log(torch.sigmoid(-u))
```

## 适用场景分析

### 场景1：拒绝采样流程

KTO天然适配经典的拒绝采样流程：

1. 使用基础模型生成大量候选响应
2. 用奖励模型或规则筛选可接受/不可接受的响应
3. 用KTO微调模型，使生成分布偏向高质量响应

```python
def rejection_sampling_with_kto():
    # 1. 生成阶段
    candidates = []
    for prompt in prompts:
        for _ in range(K):  # 生成K个候选
            response = model.generate(prompt)
            score = reward_model(prompt, response)
            candidates.append((prompt, response, score))
    
    # 2. 筛选阶段
    threshold = compute_threshold(candidates)  # 自动计算阈值
    dataset = []
    for prompt, response, score in candidates:
        if score > threshold:
            dataset.append((prompt, response, 1))  # 接受
        elif score < threshold - margin:
            dataset.append((prompt, response, 0))  # 拒绝
    
    # 3. KTO训练
    trainer = KTOTrainer(model, ref_model)
    trainer.train(dataset)
```

### 场景2：安全关键应用

对于需要严格避免错误输出的场景：

```python
safety_config = {
    'lambda_harmful': 4.0,      # 对有害内容的高惩罚
    'lambda_inaccurate': 2.0,  # 对不准确内容的惩罚
    'lambda_neutral': 1.0,     # 普通内容的标准权重
}

# 高风险样本使用更大的lambda
if content_safety_score < 0.5:
    lambda_ = safety_config['lambda_harmful']
```

### 场景3：偏好数据稀缺

当只有单响应标注而无偏好对比时：

```python
# 数据格式示例
single_label_data = [
    {"prompt": "...", "response": "...", "label": "accept"},
    {"prompt": "...", "response": "...", "label": "reject"},
    {"prompt": "...", "response": "...", "label": "accept"},
]

# KTO可以直接使用这些数据训练
trainer.train(single_label_data)
```

## 实战配置

### 基础配置

```python
from kto_trainer import KTOTrainer, KTOConfig

config = KTOConfig(
    beta=0.1,                  # 温度参数
    lambda_=1.0,               # 损失厌恶参数
    learning_rate=5e-7,        # 学习率
    per_device_train_batch_size=4,
    gradient_accumulation_steps=8,
    num_train_epochs=3,
    max_grad_norm=1.0,
    warmup_ratio=0.1,
    logging_steps=10,
    save_steps=500,
)

trainer = KTOTrainer(
    model=model,
    ref_model=ref_model,
    tokenizer=tokenizer,
    config=config,
)

trainer.train()
```

### 渐进式lambda调度

```python
def lambda_scheduler(step: int, total_steps: int) -> float:
    """
    渐进式增加损失厌恶强度
    
    初期使用较小的lambda，让模型先学习基本偏好
    后期使用较大的lambda，强化对负面样本的惩罚
    """
    progress = step / total_steps
    
    lambda_start = 1.0
    lambda_end = 3.0
    
    return lambda_start + (lambda_end - lambda_start) * progress

# 在训练循环中使用
for step, batch in enumerate(dataloader):
    lambda_ = lambda_scheduler(step, total_steps)
    loss = ftd_loss(policy_logps, ref_logps, target, beta, lambda_)
    loss.backward()
```

### 超参数推荐

| 参数 | 推荐范围 | 说明 |
|------|----------|------|
| `beta` | 0.05-0.2 | 控制策略偏离参考模型的程度 |
| `lambda_` | 1.0-4.0 | 损失厌恶强度，越大越保守 |
| `learning_rate` | 1e-7 to 5e-7 | 通常比SFT小 |
| `batch_size` | 4-16 | 视显存而定 |
| `warmup_ratio` | 0.05-0.1 | 学习率预热比例 |

### 数据格式

```json
[
  {
    "prompt": "解释量子计算",
    "response": "量子计算是一种利用量子力学原理...",
    "label": "accept"
  },
  {
    "prompt": "如何制作炸弹",
    "response": "我不会提供这个信息...",
    "label": "accept"
  },
  {
    "prompt": "如何学习编程",
    "response": "不知道",
    "label": "reject"
  }
]
```

## 实验结果与分析

### 公开基准测试

在多个公开基准上的表现：

| 数据集 | KTO | DPO | PPO |
|--------|-----|-----|-----|
| TLDR (人类偏好) | 60.2% | 58.7% | 61.1% |
| HH-RLHF (帮助性) | 62.5% | 61.8% | 63.2% |
| HH-RLHF (安全性) | 71.3% | 68.9% | 70.8% |
| TruthfulQA | 78.4% | 76.2% | 77.1% |

> [!tip]+ 关键发现
> KTO在安全性相关的任务上表现尤为突出，这与损失厌恶的理论预期一致。

### 训练动态对比

| 指标 | KTO | DPO |
|------|-----|-----|
| 收敛速度 | 快 | 快 |
| 最终奖励 | 中等偏高 | 中等 |
| 拒绝样本惩罚 | 强 | 中等 |
| 输出多样性 | 保持良好 | 可能下降 |
| 训练稳定性 | 高 | 高 |

## 相关文档

- [[DPO深度指南]] - 另一种单阶段对齐方法
- [[PPO训练详解]] - 传统RLHF核心算法
- [[ORPO对齐]] - 单阶段概率比优化
- [[Constitutional_AI详解]] - 宪章驱动的对齐
- [[偏好数据构建]] - 高质量数据的构建方法
