---
title: Adapter微调
date: 2026-04-18
tags:
  - LLM
  - Adapter
  - PEFT
  - Fine-tuning
  - Bottleneck
  - 参数高效微调
categories:
  - 人工智能
  - 大模型微调
alias: Adapter Fine-tuning
---

## 关键词

| 术语 | 英文 | 核心含义 |
|------|------|----------|
| Adapter | Adapter Layer | 适配器层结构 |
| 瓶颈层 | Bottleneck Layer | 降维-升维结构 |
| 残差连接 | Residual Connection | 跳跃连接机制 |
| 串行结构 | Series Adapter | 与原层串联 |
| 并行结构 | Parallel Adapter | 与原层并联 |
| 适配器融合 | AdapterFusion | 多任务适配器整合 |
| Houlsby Adapter | Houlsby Architecture | 标准适配器架构 |
| AdapterHub | AdapterHub Framework | 适配器框架 |
| 轻量化 | Lightweight | 参数压缩技术 |
| 任务迁移 | Task Transfer | 跨任务知识迁移 |

---

## 概述

[[Adapter（适配器）]] 微调是 2019 年由 Hugging Face 研究团队提出的一种参数高效微调技术。其核心思想是在 Transformer 架构的每层中插入小型可训练模块（Adapter），同时冻结预训练模型的原始参数，仅训练适配器层的参数。

Adapter 的设计哲学与 [[LoRA]] 有异曲同工之妙——都希望通过引入少量可训练参数来适应下游任务。但 Adapter 采用了不同于 LoRA 的架构设计：在维度上做"压缩-解压"处理，形成瓶颈结构，从而实现参数量的大幅削减。

---

## 1. Adapter机制原理

### 1.1 核心架构

Adapter 的基本结构包含两个关键组件：

1. **下投影层（Down-project）**：将高维特征压缩到低维瓶颈空间
2. **上投影层（Up-project）**：将低维特征恢复到原始高维空间

```
原始Transformer层：
    Input (d) → [Transformer Layer] → Output (d)

Adapter结构（插入位置）：
    Input (d) → [Transformer Layer] → Output (d)
                    ↓
              ┌─────────────────────┐
              │    Adapter Layer    │
              │  ┌───────────────┐  │
              │  │ Down-project  │  │
              │  │   d → r       │  │
              │  └───────┬───────┘  │
              │          ↓          │
              │  ┌───────────────┐  │
              │  │  Activation   │  │
              │  │  (非线性)      │  │
              │  └───────┬───────┘  │
              │          ↓          │
              │  ┌───────────────┐  │
              │  │ Up-project    │  │
              │  │   r → d       │  │
              │  └───────────────┘  │
              └─────────────────────┘
                    ↓ (残差相加)
              Output (d)
```

### 1.2 数学表达

对于输入 $x \in \mathbb{R}^d$，Adapter 的前向传播为：

$$Adapter(x) = W_{up} \cdot \sigma(W_{down} \cdot x) + x$$

其中：
- $W_{down} \in \mathbb{R}^{r \times d}$：下投影矩阵
- $W_{up} \in \mathbb{R}^{d \times r}$：上投影矩阵
- $\sigma$：非线性激活函数（通常为 ReLU）
- $r \ll d$：瓶颈维度

> [!note]
> 可训练参数量：$r \times d + d \times r = 2rd$，相比全参数 $d^2$，压缩比为 $\frac{2r}{d}$。

---

## 2. Houlsby Adapter详解

### 2.1 架构设计

Houlsby Adapter（原版 Adapter）是 Google 在 2019 年提出的经典架构：

```python
import torch
import torch.nn as nn

class HoulsbyAdapter(nn.Module):
    """
    Houlsby Adapter 实现
    串行结构，插入在Attention和FFN之后
    """
    
    def __init__(self, d_model, r=64, non_linearity="relu"):
        super().__init__()
        self.d_model = d_model
        self.bottleneck_dim = r
        
        # 激活函数
        if non_linearity == "relu":
            self.activation = nn.ReLU()
        elif non_linearity == "gelu":
            self.activation = nn.GELU()
        else:
            raise ValueError(f"Unknown activation: {non_linearity}")
        
        # 下投影
        self.down_proj = nn.Linear(d_model, r, bias=False)
        
        # 上投影
        self.up_proj = nn.Linear(r, d_model, bias=False)
        
        # 初始化：上投影为零矩阵（等价于恒等映射）
        nn.init.zeros_(self.up_proj.weight)
        if self.up_proj.bias is not None:
            nn.init.zeros_(self.up_proj.bias)
    
    def forward(self, x, residual=None):
        """
        Args:
            x: 输入张量 [batch, seq_len, d_model]
            residual: 残差连接（可选）
        """
        # 计算适配器输出
        bottleneck = self.down_proj(x)
        bottleneck = self.activation(bottleneck)
        adapter_output = self.up_proj(bottleneck)
        
        # 残差连接
        if residual is not None:
            return residual + adapter_output
        else:
            return x + adapter_output
```

### 2.2 参数计算

| 维度 | 参数计算 | 示例 (d=768, r=64) |
|------|----------|-------------------|
| 下投影 | $r \times d$ | 49,152 |
| 上投影 | $d \times r$ | 49,152 |
| 偏置（可选） | $r + d$ | 832 |
| 总计 | $2rd + (r+d)$ | 99,136 |
| 全连接对比 | $d^2$ | 589,824 |
| 压缩比 | $\frac{2r}{1}$ | 约 6% |

### 2.3 插入位置

Houlsby Adapter 可以插入在 Transformer 的多个位置：

```python
class TransformerWithAdapters(nn.Module):
    """
    Transformer层中Adapter的插入位置
    """
    
    def __init__(self, d_model, n_heads, d_ff, adapter_r=64):
        super().__init__()
        self.attention = MultiHeadAttention(d_model, n_heads)
        self.ffn = FeedForward(d_model, d_ff)
        
        # 位置1: Attention之后
        self.adapter_attn = HoulsbyAdapter(d_model, adapter_r)
        
        # 位置2: FFN之后
        self.adapter_ffn = HoulsbyAdapter(d_model, adapter_r)
        
        # Layer Norm
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
    
    def forward(self, x, mask=None):
        # Self-attention with residual
        attn_out, _ = self.attention(x, x, x, mask)
        x = self.norm1(x + attn_out)
        x = self.adapter_attn(x)  # Adapter after attention
        
        # FFN with residual
        ffn_out = self.ffn(x)
        x = self.norm2(x + ffn_out)
        x = self.adapter_ffn(x)  # Adapter after FFN
        
        return x
```

---

## 3. Adapter层结构设计

### 3.1 串行 Adapter（Series）

串行 Adapter 与原始层串联，计算顺序为：

```
Input → [Original Layer] → [Adapter] → Output
```

特点：
- 对原始特征进行"精炼"
- 增加的计算量较小
- 训练稳定

```python
class SeriesAdapter(nn.Module):
    """
    串行Adapter：插入在LayerNorm之后
    """
    
    def __init__(self, d_model, r=64):
        super().__init__()
        self.adapter = nn.Sequential(
            nn.Linear(d_model, r),
            nn.ReLU(),
            nn.Linear(r, d_model),
        )
        # 初始化
        nn.init.zeros_(self.adapter[2].weight)
        nn.init.zeros_(self.adapter[2].bias)
    
    def forward(self, x):
        return x + self.adapter(x)
```

### 3.2 并行 Adapter（Parallel）

并行 Adapter 与原始层并联，计算顺序为：

```
Input → [Original Layer] ─┬─→ [+] → Output
       → [Adapter Layer] ─┘
```

特点：
- 原始能力与适配能力分离
- 收敛可能更快
- 适合多任务场景

```python
class ParallelAdapter(nn.Module):
    """
    并行Adapter：与原始层并联
    """
    
    def __init__(self, d_model, r=64):
        super().__init__()
        self.adapter = nn.Sequential(
            nn.Linear(d_model, r),
            nn.ReLU(),
            nn.Linear(r, d_model),
        )
        # 初始化
        nn.init.zeros_(self.adapter[2].weight)
    
    def forward(self, original_output, input_tensor):
        """
        Args:
            original_output: 原始层的输出
            input_tensor: 原始输入（用于计算Adapter）
        """
        adapter_output = self.adapter(input_tensor)
        return original_output + adapter_output

# 使用示例
class ParallelTransformerLayer(nn.Module):
    def __init__(self, d_model, n_heads, r=64):
        super().__init__()
        self.attention = MultiHeadAttention(d_model, n_heads)
        self.ffn = FeedForward(d_model)
        
        # 并行Adapter
        self.adapter_attn = ParallelAdapter(d_model, r)
        self.adapter_ffn = ParallelAdapter(d_model, r)
    
    def forward(self, x, mask=None):
        # 计算原始注意力
        attn_out, _ = self.attention(x, x, x, mask)
        
        # 并行计算Adapter（使用原始输入）
        x = self.adapter_attn(attn_out, x)
        
        # FFN
        ffn_out = self.ffn(x)
        x = self.adapter_ffn(ffn_out, x)
        
        return x
```

### 3.3 串行 vs 并行 对比

| 维度 | 串行 Adapter | 并行 Adapter |
|------|-------------|-------------|
| 参数量 | 相同 | 相同 |
| 计算顺序 | 顺序执行 | 可并行执行 |
| 表达能力 | 较弱 | 较强 |
| 训练稳定性 | 较好 | 可能不稳定 |
| 适用场景 | 单任务 | 多任务 |

---

## 4. Adapter变体

### 4.1 轻量化 Adapter

**Compacter** 和 **AdapterDrop** 通过改进架构进一步减少参数量：

```python
class CompacterAdapter(nn.Module):
    """
    Compacter: 使用Kronecker分解进一步压缩
    """
    
    def __init__(self, d_model, r=64):
        super().__init__()
        # Kronecker分解：W = A ⊗ B
        # 参数量从 r×d 减少到 r×d1 + d1×d2 + ...
        self.phm_rule = nn.Linear(d_model, r, bias=False)
        
        # 或使用 AdapterDrop：随机丢弃部分 Adapter
        self.drop_rate = 0.1
    
    def forward(self, x):
        # 训练时随机丢弃
        if self.training and torch.rand(1).item() < self.drop_rate:
            return x
        return x + self.phm_rule(x)
```

### 4.2 AdapterFusion

AdapterFusion 解决多任务学习中的适配器融合问题：

```python
class AdapterFusion(nn.Module):
    """
    AdapterFusion: 融合多个Adapter的知识
    """
    
    def __init__(self, d_model, n_tasks):
        super().__init__()
        self.n_tasks = n_tasks
        self.query = nn.Linear(d_model, d_model)
        self.key = nn.Linear(d_model, d_model)
        self.value = nn.Linear(d_model, d_model)
        
        # 每个任务一个value投影
        self.value_proj = nn.ModuleList([
            nn.Linear(d_model, d_model) for _ in range(n_tasks)
        ])
    
    def forward(self, adapter_outputs, task_ids):
        """
        Args:
            adapter_outputs: [n_tasks, batch, seq, d_model]
            task_ids: 当前样本对应的任务ID
        """
        # Query: 使用当前任务Adapter的输出
        q = self.query(adapter_outputs[task_ids])
        
        # Key-Value: 所有Adapter
        k = self.key(adapter_outputs)  # [n_tasks, B, D]
        v = torch.stack([self.value_proj[i](adapter_outputs[i]) 
                        for i in range(self.n_tasks)], dim=0)
        
        # 注意力加权
        scores = torch.matmul(q, k.transpose(-2, -1)) / (d_model ** 0.5)
        weights = torch.softmax(scores, dim=-1)
        
        fused = torch.matmul(weights, v.transpose(0, 1))
        return fused.squeeze(0)
```

---

## 5. AdapterHub框架

### 5.1 框架概述

AdapterHub 是 Hugging Face 提供的 Adapter 统一框架：

```python
from transformers import AutoModel, AutoTokenizer
from adapter_transformers import AdapterType, HoulsbyConfig, PfeifferConfig

# 加载模型
model = AutoModel.from_pretrained("bert-base-uncased")

# 添加Adapter
model.add_adapter("sst-2", AdapterType("text_classification"))
model.add_adapter("qnli", AdapterType("question_answering"))

# 训练特定Adapter
model.set_active_adapters("sst-2")

# 或者使用预置配置
model.add_adapter("houlsby", config=HoulsbyConfig(r=64, reduction_factor=16))
model.add_adapter("pfeiffer", config=PfeifferConfig(r=64))
```

### 5.2 常用配置

| 配置类型 | 瓶颈维度 | 缩减因子 | 参数量（BERT-base） |
|----------|----------|----------|-------------------|
| Houlsby | 64 | 16 | ~3.5M |
| Pfeiffer | 64 | 16 | ~3.5M |
| Houlsby (small) | 8 | 16 | ~0.4M |
| Pfeiffer (large) | 128 | 16 | ~7M |

### 5.3 多任务训练

```python
from adapter_transformers import AdapterTrainer

# 配置训练参数
training_args = TrainingArguments(
    output_dir="./adapter_output",
    per_device_train_batch_size=32,
    learning_rate=1e-4,
    num_train_epochs=3,
)

# 创建Trainer
trainer = AdapterTrainer(
    model=model,
    args=training_args,
    train_dataset=train_dataset,
    adapter_name="sst-2",
)

# 训练
trainer.train()

# 切换到另一个任务
model.set_active_adapters("qnli")
trainer = AdapterTrainer(
    model=model,
    args=training_args,
    train_dataset=qnli_dataset,
    adapter_name="qnli",
)
trainer.train()
```

---

## 6. Adapter与LoRA对比

### 6.1 架构差异

| 维度 | Adapter | LoRA |
|------|---------|------|
| 架构 | 瓶颈结构 | 低秩分解 |
| 参数化方式 | 独立权重 | 分解形式 |
| 位置 | 插入层内部 | 并行于Attention |
| 非线性 | 有（ReLU/GELU） | 无（线性） |
| 推理开销 | 串行计算 | 可合并消除 |

### 6.2 代码结构对比

```python
# ============ Adapter ============
class Adapter(nn.Module):
    def __init__(self, d, r):
        self.down = nn.Linear(d, r)  # d → r
        self.activation = nn.ReLU()
        self.up = nn.Linear(r, d)    # r → d
    
    def forward(self, x):
        return x + self.up(self.activation(self.down(x)))

# ============ LoRA ============
class LoRA(nn.Module):
    def __init__(self, d_in, d_out, r):
        self.A = nn.Parameter(torch.randn(r, d_in) * 0.01)  # r × d_in
        self.B = nn.Parameter(torch.zeros(d_out, r))          # d_out × r
    
    def forward(self, x):
        return x + (x @ self.A.T @ self.B.T)  # 无非线性
```

### 6.3 效果对比

| 任务 | 全参数 | Adapter | LoRA |
|------|--------|---------|------|
| SST-2 | 94.0 | 93.8 | 94.1 |
| QNLI | 91.8 | 91.2 | 91.7 |
| MNLI | 85.8 | 85.3 | 85.6 |
| 问答 | 87.5 | 86.9 | 87.3 |

> [!tip]
> 实践中，LoRA 和 Adapter 的效果相近，但 LoRA 的实现更简洁，推理时可以通过权重合并消除额外开销。

---

## 7. Adapter微调完整示例

### 7.1 使用HuggingFace PEFT库

```python
#!/usr/bin/env python3
"""
Adapter微调完整示例
"""

import torch
from transformers import (
    AutoModel,
    AutoTokenizer,
    TrainingArguments,
    Trainer,
    DataCollatorForLanguageModeling,
)
from peft import AdapterConfig, TaskType, get_peft_model

def setup_adapter_model(model_name, adapter_r=64, reduction_factor=16):
    """配置Adapter微调模型"""
    
    # 加载预训练模型
    model = AutoModel.from_pretrained(model_name)
    
    # Adapter配置
    adapter_config = AdapterConfig(
        hidden_size=model.config.hidden_size,
        adapter_size=adapter_r,
        adapter_activation="relu",
        adapter_reduction_factor=reduction_factor,
        attn_dim=None,  # 用于MHA的Adapter
        attn_bn=False,
    )
    
    # 添加Adapter
    model.add_adapter("task_adapter", config=adapter_config)
    
    # 冻结其他参数
    for name, param in model.named_parameters():
        if "adapter" not in name:
            param.requires_grad = False
    
    # 统计可训练参数
    trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
    total_params = sum(p.numel() for p in model.parameters())
    print(f"Trainable: {trainable_params:,} / {total_params:,} ({trainable_params/total_params*100:.3f}%)")
    
    return model

def main():
    # ============ 配置 ============
    MODEL_NAME = "bert-base-uncased"
    OUTPUT_DIR = "./adapter_output"
    
    # ============ 加载模型 ============
    print("Loading model...")
    model = setup_adapter_model(MODEL_NAME, adapter_r=64, reduction_factor=16)
    
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
    
    # ============ 数据预处理 ============
    def tokenize_function(examples):
        return tokenizer(
            examples["text"],
            truncation=True,
            max_length=512,
            padding="max_length",
        )
    
    from datasets import load_dataset
    dataset = load_dataset("glue", "sst2")
    tokenized_dataset = dataset.map(tokenize_function, batched=True)
    
    # ============ 训练 ============
    training_args = TrainingArguments(
        output_dir=OUTPUT_DIR,
        per_device_train_batch_size=16,
        learning_rate=1e-4,
        num_train_epochs=3,
        warmup_ratio=0.1,
        logging_steps=100,
        save_steps=500,
        eval_strategy="steps",
        load_best_model_at_end=True,
    )
    
    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=tokenized_dataset["train"],
        eval_dataset=tokenized_dataset["validation"],
        data_collator=DataCollatorForLanguageModeling(tokenizer, mlm=True),
    )
    
    # 设置活动Adapter
    model.set_active_adapters("task_adapter")
    
    print("Starting training...")
    trainer.train()
    
    # ============ 保存 ============
    model.save_adapter("./saved_adapters/task_adapter", "task_adapter")
    print("Adapter saved!")

if __name__ == "__main__":
    main()
```

### 7.2 完整配置示例

```yaml
# adapter_config.yaml
model:
  name: "bert-base-uncased"
  adapter_r: 64
  reduction_factor: 16
  adapter_type: "houlsby"  # houlsby 或 pfeiffer
  non_linearity: "relu"

training:
  per_device_batch_size: 16
  gradient_accumulation_steps: 1
  learning_rate: 1e-4
  num_epochs: 3
  warmup_ratio: 0.1
  weight_decay: 0.01
  max_grad_norm: 1.0
  
  # 优化器
  optimizer: "adamw"
  adam_beta1: 0.9
  adam_beta2: 0.999
  adam_epsilon: 1e-8

# 参数量对比
# 全参数: 110M
# Adapter (r=64, reduction=16): ~3.5M (3.2%)
# LoRA (r=8): ~0.3M (0.3%)
```

---

## 8. Adapter的优缺点总结

### 8.1 优势

| 优势 | 说明 |
|------|------|
| 参数量小 | 通常只需训练全参数的 1-3% |
| 任务隔离 | 每个任务独立的Adapter，互不干扰 |
| 灵活扩展 | 可随时添加新任务的Adapter |
| 可联合训练 | 多任务Adapter可联合优化 |
| 推理时切换 | 可在推理时动态切换不同任务的Adapter |

### 8.2 局限性

| 局限 | 说明 |
|------|------|
| 推理开销 | Adapter计算串行，增加延迟 |
| 收敛较慢 | 瓶颈结构限制表达能力 |
| 超参敏感 | 瓶颈维度r和reduction_factor需要调优 |
| 不易合并 | 难以像LoRA一样合并权重 |

> [!warning]
> Adapter 的主要缺点是推理时会引入额外的计算延迟。如果对延迟敏感的应用场景，建议使用 LoRA 或在推理前合并权重。

---

## 相关文档

- [[LoRA微调深度指南]] - 低秩适配方法
- [[QLoRA微调详解]] - 量化+LoRA技术
- [[全参数微调]] - 传统微调方法
- [[P-Tuning微调]] - Prompt Tuning方法
- [[prefix微调]] - Prefix Tuning方法
- [[微调技术对比总结]] - 各技术综合对比

---

*本文档由 AI 知识库自动生成*
