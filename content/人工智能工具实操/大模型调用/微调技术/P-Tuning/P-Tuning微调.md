---
title: P-Tuning微调
date: 2026-04-18
tags:
  - LLM
  - P-Tuning
  - Prompt-Tuning
  - PEFT
  - Fine-tuning
  - Soft-Prompt
  - 提示微调
categories:
  - 人工智能
  - 大模型微调
alias: P-Tuning Fine-tuning
---

## 关键词

| 术语 | 英文 | 核心含义 |
|------|------|----------|
| P-Tuning | Prompt Tuning | 提示微调方法 |
| 软提示 | Soft Prompt | 可学习的连续提示 |
| 硬提示 | Hard Prompt | 离散文本提示 |
| 连续提示 | Continuous Prompt | 嵌入空间的提示 |
| 离散提示 | Discrete Prompt | 文本形式的提示 |
| 前缀提示 | Prefix Prompt | 添加在输入前的提示 |
| Prompt Encoder | Prompt Encoder | 提示编码器 |
| 伪令牌 | Pseudo Token | 虚拟可学习令牌 |
| 多层提示 | Multi-Layer Prompt | 多层分布的提示 |
| 知识蒸馏 | Knowledge Distillation | 知识迁移技术 |

---

## 概述

[[P-Tuning（Prompt Tuning）]] 是 2021 年由清华大学和华为诺亚方舟实验室提出的参数高效微调技术。与 [[LoRA]]、[[Adapter微调]] 等通过修改模型权重不同，P-Tuning 的核心思路是：**不改变预训练模型的任何参数，而是通过学习一组"软提示"（Soft Prompts）来引导模型行为**。

P-Tuning 的灵感来源于自然语言处理中"提示工程"（Prompt Engineering）的成功实践。但与传统的手工设计离散提示不同，P-Tuning 将提示本身也作为可学习的参数，在连续空间中优化。

---

## 1. Prompt Tuning原理

### 1.1 传统Prompt vs P-Tuning

**传统Prompt（硬提示）**：
```
输入: "The movie was [MASK]"
      ↓
手动设计的离散文本
      ↓
模型预测: "great"
```

**P-Tuning（软提示）**：
```
输入: [LEARNABLE_1] [LEARNABLE_2] ... [LEARNABLE_k] "The movie was [MASK]"
      ↓
可学习的连续嵌入向量
      ↓
模型预测: "great"
```

### 1.2 核心思想

P-Tuning 的关键洞察是：**提示不一定要是人类可解释的文本**，只要在模型的嵌入空间中有效即可。

```python
import torch
import torch.nn as nn
from transformers import AutoModel, AutoTokenizer, PretrainedConfig

class PromptTuning(nn.Module):
    """
    P-Tuning 核心实现
    学习连续的prompt embedding，而非离散的token
    """
    
    def __init__(self, model_name, n_prompt_tokens=20, prompt_dim=None):
        super().__init__()
        
        # 加载预训练模型（冻结）
        self.model = AutoModel.from_pretrained(model_name)
        self.model.eval()
        for param in self.model.parameters():
            param.requires_grad = False
        
        # 获取嵌入维度
        if prompt_dim is None:
            prompt_dim = self.model.config.hidden_size
        
        self.n_prompt_tokens = n_prompt_tokens
        self.prompt_dim = prompt_dim
        
        # 可学习的软提示
        # 初始化为随机向量（可改进初始化策略）
        self.prompt_embeddings = nn.Parameter(
            torch.randn(n_prompt_tokens, prompt_dim) * 0.01
        )
        
        print(f"P-Tuning: {n_prompt_tokens} prompts, {prompt_dim} dim")
        print(f"Trainable params: {n_prompt_tokens * prompt_dim:,}")
    
    def forward(self, input_ids, attention_mask=None):
        # 获取文本嵌入
        text_embeddings = self.model.get_input_embeddings()(input_ids)
        
        # 拼接软提示和文本嵌入
        # [batch, n_prompt, dim] + [batch, seq_len, dim]
        prompt_embeds = self.prompt_embeddings.unsqueeze(0).expand(
            text_embeddings.size(0), -1, -1
        )
        
        # 在序列前面插入prompt
        combined_embeddings = torch.cat([prompt_embeds, text_embeddings], dim=1)
        
        # 调整attention mask
        if attention_mask is not None:
            prompt_mask = torch.ones(
                attention_mask.size(0), self.n_prompt_tokens,
                device=attention_mask.device
            )
            combined_mask = torch.cat([prompt_mask, attention_mask], dim=1)
        else:
            combined_mask = None
        
        # 前向传播
        outputs = self.model(
            inputs_embeds=combined_embeddings,
            attention_mask=combined_mask,
        )
        
        return outputs
```

---

## 2. 连续Prompt vs 离散Prompt

### 2.1 离散Prompt的局限性

传统Prompt Engineering面临以下挑战：

| 挑战 | 说明 |
|------|------|
| 人工设计耗时 | 需要反复尝试不同措辞 |
| 任务迁移性差 | 一个任务的有效prompt不一定适合另一个 |
| 优化困难 | 离散空间的梯度优化不可行 |
| 依赖模型规模 | 小模型对prompt敏感度低 |

> [!note]
> GPT-3 的论文表明，离散Prompt需要精心设计，且不同prompt之间的性能差异可能超过5个百分点。这促使研究者探索可学习的软提示。

### 2.2 连续Prompt的优势

```python
# 连续Prompt vs 离散Prompt 对比

class ContinuousPromptConfig:
    """连续提示配置"""
    n_tokens: 20          # 可学习的软token数量
    initialization: "uniform"  # 初始化方式
    optimizer: "Adam"    # 使用梯度优化
    learning_rate: 0.1   # 通常使用较大学习率

class DiscretePromptConfig:
    """离散提示配置"""
    template: "This is a {task}. {input}"  # 固定模板
    verbalizer: ["positive", "negative"]    # 标签映射
    search_space: "discrete"  # 手动搜索
```

### 2.3 初始化策略

P-Tuning 的一个关键设计是软提示的初始化。常见策略：

```python
class PromptInitialization:
    """
    软提示初始化策略
    """
    
    @staticmethod
    def random_uniform(shape, dim):
        """策略1：随机均匀初始化"""
        return torch.randn(shape, dim) * 0.01
    
    @staticmethod
    def from_vocab(tokens, embedding_layer):
        """策略2：从真实词汇初始化"""
        # 选择有意义的中文词汇
        words = ["这是", "关于", "问题", "分析", "讨论"]
        indices = [tokenizer.convert_tokens_to_ids(w) for w in words]
        return embedding_layer(torch.tensor(indices)).mean(dim=0)
    
    @staticmethod
    def from_distribution(vocab_size, dim, n_samples=100):
        """策略3：从正态分布采样"""
        # 使用预训练嵌入的统计分布
        mean = torch.zeros(dim)
        std = 0.07  # 经验值
        return torch.randn(shape) * std
    
    @staticmethod
    def kaiming(shape, dim):
        """策略4：Kaiming初始化"""
        # 适合ReLU激活
        return torch.randn(shape, dim) * (2.0 / dim) ** 0.5
```

---

## 3. P-Tuning v1 vs v2

### 3.1 P-Tuning v1

v1 版本的核心特点：

```python
class P-tuning_v1(nn.Module):
    """
    P-Tuning v1: 仅在输入层添加prompt
    """
    
    def __init__(self, model, n_tokens=20):
        super().__init__()
        self.model = model
        self.n_tokens = n_tokens
        
        # 嵌入维度
        self.embedding_dim = model.config.hidden_size
        
        # 可学习的prompt tokens
        self.prompt_embeddings = nn.Parameter(
            torch.randn(n_tokens, self.embedding_dim) * 0.01
        )
        
        # 可选的LSTM/BiLSTM编码器
        self.lstm = nn.LSTM(
            input_size=self.embedding_dim,
            hidden_size=self.embedding_dim // 2,
            num_layers=2,
            bidirectional=True,
            batch_first=True,
        )
    
    def forward(self, input_ids, labels=None):
        # 获取文本嵌入
        text_embeddings = self.model.get_input_embeddings()(input_ids)
        
        # 复制prompt到batch维度
        batch_size = input_ids.size(0)
        prompt = self.prompt_embeddings.unsqueeze(0).expand(batch_size, -1, -1)
        
        # 可选：使用LSTM编码prompt，增强表达
        if hasattr(self, 'lstm'):
            prompt = self.lstm(prompt)[0]
        
        # 拼接
        combined = torch.cat([prompt, text_embeddings], dim=1)
        
        # 前向传播
        outputs = self.model(inputs_embeds=combined)
        return outputs
```

**v1的局限性**：
- 仅在输入层添加prompt，对深层网络的影响有限
- 大模型中，浅层的prompt信息可能在深层被稀释

### 3.2 P-Tuning v2

v2 版本的关键改进：**在每一层都添加可学习的prompt**

```python
class P-tuning_v2(nn.Module):
    """
    P-Tuning v2: 多层prompt（Deep Prompt Tuning）
    """
    
    def __init__(self, model, n_layers, n_tokens=20):
        super().__init__()
        self.model = model
        self.n_layers = n_layers
        self.n_tokens = n_tokens
        
        embedding_dim = model.config.hidden_size
        
        # 为每一层创建独立的prompt
        self.layer_prompts = nn.ParameterList([
            nn.Parameter(torch.randn(n_tokens, embedding_dim) * 0.01)
            for _ in range(n_layers)
        ])
    
    def forward(self, input_ids, attention_mask=None):
        # 获取嵌入
        text_embeddings = self.model.get_input_embeddings()(input_ids)
        
        # 初始hidden states
        hidden_states = text_embeddings.transpose(0, 1)  # [seq, batch, dim]
        
        # 逐层处理
        for layer_idx, layer in enumerate(self.model.transformer.layer):
            # 插入layer prompt
            if layer_idx < self.n_layers:
                layer_prompt = self.layer_prompts[layer_idx].unsqueeze(1)  # [n_token, 1, dim]
                hidden_states = torch.cat([
                    layer_prompt.expand(-1, hidden_states.size(1), -1),
                    hidden_states
                ], dim=0)
            
            # 应用attention
            layer_output = layer(hidden_states, attention_mask=None)
            hidden_states = layer_output[0]
            
            # 如果添加了prompt，需要调整hidden states
            if layer_idx < self.n_layers:
                hidden_states = hidden_states[self.n_tokens:]
        
        return hidden_states.transpose(0, 1)  # [batch, seq, dim]
```

### 3.3 v1 vs v2 对比

| 维度 | P-Tuning v1 | P-Tuning v2 |
|------|-------------|-------------|
| Prompt位置 | 仅输入层 | 所有Transformer层 |
| 参数位置 | 嵌入空间 | 注意力层输入 |
| 参数量 | 约0.1% | 约0.1-1%（取决于层数） |
| 大模型效果 | 一般 | 显著提升 |
| 适用规模 | <10B | >10B |
| 训练稳定性 | 较好 | 需要更多技巧 |

---

## 4. P-Tuning v2详解

### 4.1 架构设计

```
P-Tuning v2 多层提示示意图

Input: [LEARNABLE_1] ... [LEARNABLE_k] The movie was [MASK]

Layer 0: [LP_1] ... [LP_k] + Attention computations
Layer 1: [LP_1] ... [LP_k] + Attention computations
...
Layer N: [LP_1] ... [LP_k] + Attention computations
         ↓
最终输出
```

### 4.2 关键超参数

```python
class P-tuning_v2_config:
    """P-Tuning v2 配置"""
    
    # Prompt相关
    num_prompt_tokens = 20        # 每个prefix的token数
    num_prefix_layers = 40        # 添加prefix的层数（通常=模型层数）
    prefix_projection = False     # 是否使用MLP投影
    prefix_hidden_dim = 512       # 投影维度（如果启用）
    
    # 优化相关
    learning_rate = 0.1           # 较大学习率
    warmup_steps = 100
    weight_decay = 0.01
    
    # 其他
    use_dropout = 0.1             # prompt的dropout
    gradient_checkpointing = True # 节省显存
```

### 4.3 使用PEFT库实现

```python
from peft import PromptTuningConfig, get_peft_model, TaskType

# P-Tuning v2 配置
peft_config = PromptTuningConfig(
    task_type=TaskType.SEQ_CLS,  # 或 CAUSAL_LM
    prompt_tuning_init="TEXT",    # 初始化方式
    num_virtual_tokens=20,         # 虚拟token数量
    num_layers=12,                # 添加prefix的层数
    num_attention_heads=12,       # attention heads
    hidden_size=768,              # 嵌入维度
    prompt_tuning_init_text="Classify the sentiment: ",  # 初始化文本
    tokenizer_name_or_path="bert-base-uncased",
)

# 创建PEFT模型
model = AutoModelForSequenceClassification.from_pretrained("bert-base-uncased")
model = get_peft_model(model, peft_config)

# 统计参数
model.print_trainable_parameters()
# Output: 
# trainable params: 20 || all params: 109,483,020 || trainable%: 0.000018%
```

---

## 5. 可学习参数初始化策略

### 5.1 策略对比

| 策略 | 方法 | 优缺点 |
|------|------|--------|
| 随机初始化 | N(0, 0.01) | 简单，可能收敛慢 |
| 词汇初始化 | 使用真实词嵌入 | 利用预训练知识 |
| 任务相关初始化 | 任务标签词嵌入 | 任务相关性更强 |
| 对比初始化 | 相似任务Prompt平均 | 迁移性更好 |

### 5.2 任务相关初始化

```python
class TaskAwarePromptInit:
    """
    任务感知的Prompt初始化
    """
    
    def __init__(self, tokenizer, embedding_layer):
        self.tokenizer = tokenizer
        self.embedding = embedding_layer
    
    def get_init_prompts(self, task_description, n_tokens=20):
        """
        根据任务描述生成初始化prompt
        """
        # 获取任务关键词的嵌入
        keywords = self.extract_keywords(task_description)
        
        # 获取关键词的token ids
        token_ids = self.tokenizer.convert_tokens_to_ids(keywords)
        
        # 获取嵌入
        keyword_embeds = self.embedding(torch.tensor(token_ids))
        
        # 填充到n_tokens长度
        if len(keywords) < n_tokens:
            # 重复关键词嵌入
            repeats = (n_tokens // len(keywords)) + 1
            init_embed = keyword_embeds.repeat(repeats, 1)[:n_tokens]
        else:
            init_embed = keyword_embeds[:n_tokens]
        
        return init_embed
    
    def extract_keywords(self, description):
        """提取任务关键词"""
        # 简化版：手动指定关键词
        keyword_map = {
            "sentiment": ["positive", "negative", "review", "feeling"],
            "question_answer": ["answer", "question", "context"],
            "summarization": ["summarize", "summary", "main"],
        }
        
        # 匹配任务类型
        for key, words in keyword_map.items():
            if key in description.lower():
                return words
        
        return ["the", "this", "that"]  # 默认
```

---

## 6. 适用场景分析

### 6.1 P-Tuning的优势场景

| 场景 | 适用性 | 说明 |
|------|--------|------|
| 超大规模模型 | ★★★★★ | 仅需训练极少量参数 |
| 多任务切换 | ★★★★☆ | 每个任务独立prompt |
| 快速实验 | ★★★★☆ | 无需改变模型权重 |
| 小样本学习 | ★★★★☆ | prompt本身就是few-shot |
| 模型压缩 | ★★★★☆ | 极低存储开销 |

### 6.2 P-Tuning的局限性

| 局限 | 说明 |
|------|------|
| 收敛较慢 | 需要训练才能获得有效prompt |
| Prompt可解释性 | 软提示无法解释含义 |
| 任务复杂时效果下降 | 简单任务效果可能更好 |
| 与模型架构耦合 | 需要了解模型结构 |

> [!warning]
> P-Tuning 在 **小模型（<10B）** 上的效果不如大模型。对于小模型，建议使用 [[LoRA]] 或 [[全参数微调]]。

### 6.3 选型建议

```
选型决策：
├── 模型规模 < 10B
│   └── 推荐: LoRA > Adapter > 全参数 > P-Tuning
│
├── 模型规模 10B - 100B
│   ├── 任务简单: P-Tuning
│   ├── 任务复杂: LoRA / QLoRA
│   └── 效果优先: 全参数 (如有资源)
│
└── 模型规模 > 100B
    └── 推荐: P-Tuning / LoRA (QLoRA)
```

---

## 7. P-Tuning完整示例

### 7.1 序列分类任务

```python
#!/usr/bin/env python3
"""
P-Tuning v2 序列分类示例
"""

import torch
from transformers import (
    AutoModel,
    AutoTokenizer,
    AutoModelForSequenceClassification,
    TrainingArguments,
    Trainer,
)
from peft import PromptTuningConfig, get_peft_model, TaskType

def main():
    # ============ 配置 ============
    MODEL_NAME = "bert-base-uncased"
    OUTPUT_DIR = "./ptuning_output"
    N_PROMPT_TOKENS = 20
    NUM_LAYERS = 12  # BERT-base有12层
    
    # ============ 加载模型 ============
    print("Loading model...")
    model = AutoModelForSequenceClassification.from_pretrained(MODEL_NAME)
    
    # P-Tuning配置
    peft_config = PromptTuningConfig(
        task_type=TaskType.SEQ_CLS,
        prompt_tuning_init="RANDOM",
        num_virtual_tokens=N_PROMPT_TOKENS,
        num_layers=NUM_LAYERS,
        num_attention_heads=12,
        hidden_size=768,
    )
    
    # 包装为PEFT模型
    model = get_peft_model(model, peft_config)
    
    # 冻结原始参数
    for name, param in model.named_parameters():
        if "prompt" not in name:
            param.requires_grad = False
    
    model.print_trainable_parameters()
    
    tokenizer = AutoTokenizer.from_pretrained(MODEL_NAME)
    
    # ============ 数据预处理 ============
    from datasets import load_dataset
    
    def tokenize_function(examples):
        return tokenizer(
            examples["text"],
            truncation=True,
            max_length=512,
        )
    
    dataset = load_dataset("glue", "sst2")
    tokenized_dataset = dataset.map(tokenize_function, batched=True)
    
    # ============ 训练 ============
    training_args = TrainingArguments(
        output_dir=OUTPUT_DIR,
        per_device_train_batch_size=16,
        learning_rate=0.3,  # P-Tuning通常用较大学习率
        num_train_epochs=10,
        warmup_steps=100,
        logging_steps=50,
        save_steps=500,
        eval_strategy="steps",
        load_best_model_at_end=True,
    )
    
    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=tokenized_dataset["train"],
        eval_dataset=tokenized_dataset["validation"],
    )
    
    print("Starting training...")
    trainer.train()
    
    # ============ 保存 ============
    model.save_pretrained(f"{OUTPUT_DIR}/final_model")
    print("Model saved!")

if __name__ == "__main__":
    main()
```

### 7.2 因果语言建模

```python
"""
P-Tuning 用于因果语言建模 (Causal LM)
适用于LLaMA等自回归模型
"""

from peft import PromptTuningConfig, get_peft_model, TaskType

# 配置
peft_config = PromptTuningConfig(
    task_type=TaskType.CAUSAL_LM,
    prompt_tuning_init="TEXT",
    num_virtual_tokens=50,
    prompt_tuning_init_text="Complete the following: ",
    tokenizer_name_or_path="meta-llama/Llama-2-7b-hf",
)

# 加载模型
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    torch_dtype=torch.bfloat16,
    device_map="auto",
)

# 包装
model = get_peft_model(model, peft_config)

# 训练
# ... (同上的训练流程)
```

### 7.3 训练配置建议

```python
# P-Tuning训练配置
training_args = TrainingArguments(
    # 学习率：P-Tuning通常用较大学习率
    learning_rate=0.3,  # 可尝试0.1-1.0
    warmup_ratio=0.1,
    
    # 优化器
    optim="adamw_torch",
    weight_decay=0.01,
    
    # 训练策略
    num_train_epochs=10,  # P-Tuning需要更多epoch
    per_device_train_batch_size=16,
    gradient_accumulation_steps=2,
    
    # 评估
    eval_strategy="steps",
    eval_steps=200,
    
    # 保存
    save_strategy="steps",
    save_steps=500,
    save_total_limit=1,
)
```

---

## 8. P-Tuning vs 其他方法

### 8.1 参数量对比

| 方法 | 参数量 | 占比 | 适用规模 |
|------|--------|------|----------|
| 全参数 | 100% | - | - |
| LoRA | 1-5% | 取决于rank | 任意 |
| Adapter | 1-3% | 取决于bottleneck | 任意 |
| P-Tuning | <0.1% | 固定 | >10B最佳 |
| Prefix-Tuning | ~0.1% | 固定 | 任意 |

### 8.2 性能对比

| 任务 | 全参数 | LoRA | P-Tuning v2 |
|------|--------|------|-------------|
| 文本分类 | 94.0 | 93.8 | 93.2 |
| 问答 | 91.8 | 91.5 | 90.8 |
| 文本生成 | 优秀 | 优秀 | 良好 |
| 少样本 | - | - | 优秀 |

### 8.3 优缺点总结

| P-Tuning 优势 | P-Tuning 局限 |
|---------------|---------------|
| 参数量最少 | 小模型效果一般 |
| 可解释性（文本初始化） | 收敛较慢 |
| 适合超大规模模型 | prompt可能不稳定 |
| 便于多任务切换 | 需要较多样本学习 |

---

## 相关文档

- [[LoRA微调深度指南]] - 低秩适配方法
- [[QLoRA微调详解]] - 量化+LoRA技术
- [[Adapter微调]] - Adapter机制对比
- [[prefix微调]] - Prefix Tuning方法
- [[全参数微调]] - 传统微调方法
- [[微调技术对比总结]] - 各技术综合对比

---

*本文档由 AI 知识库自动生成*
