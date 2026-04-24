---
title: Unsloth使用指南
date: 2026-04-18
tags:
  - LLM微调
  - Unsloth
  - LoRA
  - QLoRA
  - 高效训练
  - PyTorch
  - 量化训练
  - GPU优化
  - 模型训练
  - 4-bit量化
categories:
  - AI工具实操
  - 大模型调用
  - 开源微调框架
alias: Unsloth Guide
---

# Unsloth使用指南

## 关键词速览

> [!note]+ 关键词
> | Unsloth | LoRA微调 | QLoRA | 4-bit量化 | 2x加速 |
> |---------|---------|-------|----------|--------|
> | 显存减半 | GGUF导出 | 多模态 | FlashAttention | DyT |
> | 免费可用 | Llama/Mistral/Gemma | TRL集成 | 模型动物园 | 分布式训练 |

---

## 1. Unsloth核心优势

Unsloth是由Daniel Han等人维护的开源项目，其核心理念是**让大模型微调变得更快、更省显存、更易于使用**。与原生Hugging Face Trainer相比，Unsloth在多个维度上实现了突破性优化。

### 1.1 速度优势

Unsloth实现了**2倍以上的训练速度提升**，这一优势主要来源于以下几个技术层面：

- **Flash Attention 2集成**：Flash Attention是一种注意力机制的高效实现，通过IO感知算法显著减少GPU显存访问次数，在长序列场景下效果尤为明显。Unsloth默认启用Flash Attention 2，可将注意力计算速度提升2-4倍。

- **动态Token Rescaling（DyT）**：Unsloth团队创新性地提出了DyT初始化方法，替代传统LayerNorm。DyT通过动态缩放机制，在保持模型表达能力的同时显著加速收敛。

- **算子融合优化**：将多个连续的计算操作融合为单一Kernel，减少中间结果的显存占用和通信开销。例如将QKV投影、注意力计算中的多个操作合并执行。

### 1.2 显存优势

Unsloth实现了**50%以上的显存占用降低**，这使得普通消费级GPU（如RTX 3090/4090）也能训练数十亿参数的大模型：

- **4-bit量化骨干网络**：利用bitsandbytes的NF4量化格式，将模型权重从FP16/BF16压缩到4-bit，显存占用减半而精度损失极小。

- **优化器状态分页**：通过CPU内存卸载优化器状态，避免GPU显存溢出。

- **梯度累积优化**：支持更大的batch size配置，在有限显存下实现等效的大batch训练效果。

### 1.3 兼容性优势

- 100%兼容Hugging Face生态，支持与TRL（Transformer Reinforcement Learning）、peft等主流库无缝集成
- 支持导出为GGUF格式，可被llama.cpp、Ollama等推理引擎直接加载
- 提供预测试的模型列表，确保稳定性和性能表现

---

## 2. 支持的模型列表

Unsloth维护了一个经过测试和优化的模型列表，涵盖主流开源大模型：

### 2.1 LLaMA系列

| 模型 | 参数量 | 是否默认支持 |
|------|--------|-------------|
| LLaMA 2 | 7B/13B/70B | ✅ |
| LLaMA 3 | 8B/70B | ✅ |
| LLaMA 3.1 | 8B/70B/405B | ✅ |
| CodeLLaMA | 7B/13B/34B | ✅ |

### 2.2 Mistral系列

| 模型 | 参数量 | 特点 |
|------|--------|------|
| Mistral 7B | 7B | 旗舰开源模型 |
| Mixtral 8x7B | 46.7B | 专家混合架构 |
| Mistral-Nemo | 12B | 多语言优化 |

### 2.3 Gemma系列

| 模型 | 参数量 | 开发商 |
|------|--------|--------|
| Gemma 2B | 2B | Google |
| Gemma 7B | 7B | Google |
| Gemma 2 9B | 9B | Google (最新) |

### 2.4 Qwen系列

| 模型 | 参数量 | 特点 |
|------|--------|------|
| Qwen 2 0.5B-72B | 全系 | 阿里云开源 |
| Qwen 2.5 0.5B-72B | 全系 | 代码能力增强 |

### 2.5 Phi系列

| 模型 | 参数量 | 开发商 |
|------|--------|--------|
| Phi-3 Mini | 3.8B | Microsoft |
| Phi-3 Small | 7B | Microsoft |

### 2.6 其他支持模型

- **Yi系列**：Yi-6B/9B/34B（01.AI）
- **DeepSeek系列**：DeepSeek LLM 7B/33B/V2
- **DynaBert**：自定义动态架构
- **StarCoder系列**：代码专用模型

> [!warning]+ 注意
> 部分新型号模型（如Llama 4、Gemma 3）可能需要从源码安装Unsloth以获得最新支持。建议在安装前查阅[官方文档](https://unsloth.ai/docs)确认兼容性。

---

## 3. 安装与快速开始

### 3.1 系统要求

- **操作系统**：Linux（Ubuntu 20.04+）、macOS（Apple Silicon M系列）、Windows（WSL2）
- **Python**：3.8 - 3.11
- **GPU**：NVIDIA GPU（CUDA 11.8+）或Apple Silicon（M1/M2/M3）
- **显存**：最低8GB（建议16GB以上以获得流畅体验）

### 3.2 安装方式

Unsloth提供两种安装方式，推荐使用conda或pip快速安装：

```bash
# 方式一：使用pip安装（推荐）
pip install unsloth unsloth_zoo

# 方式二：使用conda安装（解决依赖问题）
conda install -c conda-forge pytorch-cuda=12.1 pytorch torchvision torchaudio
pip install unsloth

# 方式三：从源码安装（获取最新功能）
pip install git+https://github.com/unslothai/unsloth.git
```

### 3.3 验证安装

```python
import torch
from unsloth import FastLanguageModel

# 检查CUDA可用性
print(f"CUDA available: {torch.cuda.is_available()}")
print(f"CUDA device: {torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'N/A'}")

# 测试模型加载
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/mistral-7b-bnb-4bit",
    max_seq_length = 2048,
    dtype = None,
    load_in_4bit = True,
)
print("模型加载成功！")
```

### 3.4 快速训练示例

以下是一个完整的LoRA微调流程，从加载模型到启动训练：

```python
from unsloth import FastLanguageModel
from unsloth.trainers import TrainingArguments
from datasets import load_dataset
from trl import SFTTrainer

# 1. 加载量化模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/mistral-7b-bnb-4bit",
    max_seq_length = 2048,
    dtype = None,
    load_in_4bit = True,
)

# 2. 配置LoRA适配器
model = FastLanguageModel.get_peft_model(
    model,
    r = 16,                    # LoRA rank
    target_modules = [         # 目标模块
        "q_proj", "k_proj", "v_proj", "o_proj",
        "gate_proj", "up_proj", "down_proj"
    ],
    lora_alpha = 16,
    lora_dropout = 0.05,
    bias = "none",
    use_gradient_checkpointing = "unsloth",  # Unsloth优化的梯度检查点
)

# 3. 加载数据集（示例为对话数据集）
dataset = load_dataset("yahma/alpaca-cleaned", split = "train")

# 4. 配置训练参数
trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = 2048,
    dataset_num_proc = 4,
    training_arguments = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        warmup_steps = 10,
        max_steps = 60,
        learning_rate = 2e-4,
        fp16 = not torch.cuda.is_bf16_compatible(),
        bf16 = torch.cuda.is_bf16_compatible(),
        logging_steps = 10,
        optim = "adamw_8bit",
        weight_decay = 0.01,
        lr_scheduler_type = "linear",
        output_dir = "outputs",
    ),
)

# 5. 启动训练
trainer.train()
```

---

## 4. LoRA/QLoRA训练配置

### 4.1 LoRA配置详解

LoRA（Low-Rank Adaptation）通过在预训练模型旁添加低秩矩阵来模拟参数更新，显著减少可训练参数数量：

```python
# LoRA配置参数详解
lora_config = {
    "r": 16,                    # 秩，越大表达能力越强但显存占用越高
    "lora_alpha": 16,           # 缩放因子，通常设为rank的1-2倍
    "lora_dropout": 0.05,       # Dropout概率，防止过拟合
    "target_modules": [         # 应用LoRA的模块
        "q_proj", "k_proj", "v_proj", "o_proj",  # 注意力层
        "gate_proj", "up_proj", "down_proj",      # FFN层
    ],
    "bias": "none",             # 是否训练偏置项
    "task_type": "CAUSAL_LM",   # 任务类型
}
```

> [!tip]+ 秩（r）的选择建议
> - **r=8**：轻量级微调，显存占用最小，适合快速实验
> - **r=16**：平衡配置，默认推荐值
> - **r=32**：高表达能力，适合复杂任务
> - **r=64**：最高精度，适合最终部署前的大模型微调

### 4.2 QLoRA配置详解

QLoRA（Quantized LoRA）在量化模型基础上进行LoRA微调，是目前最低显存占用的方案：

```python
from unsloth import FastLanguageModel

# QLoRA完整配置
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/llama-3-8b-bnb-4bit",
    max_seq_length = 4096,           # 可扩展到更长序列
    dtype = None,                     # 自动选择精度
    load_in_4bit = True,              # 启用4bit量化
)

# 梯度检查点配置（大幅降低显存）
model = FastLanguageModel.get_peft_model(
    model,
    r = 32,
    lora_alpha = 64,
    lora_dropout = 0.1,
    target_modules = "all-linear",    # 对所有线性层应用LoRA
    use_gradient_checkpointing = "unsloth",
    max_seq_length = 4096,
)
```

### 4.3 多模态配置

Unsloth也开始支持视觉语言模型（VLM）的微调：

```python
from unsloth import FastVisionModel

# 加载视觉语言模型
model, tokenizer = FastVisionModel.from_pretrained(
    model_name = "unsloth/llava-1.5-7b-hf-bnb-4bit",
)

# 配置视觉模块的LoRA
model = FastVisionModel.get_peft_model(
    model,
    r = 16,
    lora_alpha = 32,
    vision_lora = True,     # 对视觉编码器也应用LoRA
    llm_lora = True,        # 对语言模型应用LoRA
)
```

---

## 5. 训练监控与保存

### 5.1 训练过程监控

```python
from unsloth.chat_templates import get_chat_template

# 设置聊天模板（用于对话数据）
tokenizer = get_chat_template(
    tokenizer,
    chat_template = "llama-3.1",
)

# 自定义回调监控训练过程
from transformers import TrainerCallback

class LossMonitorCallback(TrainerCallback):
    def on_log(self, args, state, control, logs=None, **kwargs):
        if logs:
            print(f"Step {state.global_step} | "
                  f"Loss: {logs.get('loss', 0):.4f} | "
                  f"Learning Rate: {logs.get('learning_rate', 0):.2e}")

# 在训练器中注册回调
trainer = SFTTrainer(
    model = model,
    args = training_arguments,
    train_dataset = dataset,
    callbacks = [LossMonitorCallback()],
)
```

### 5.2 使用WandB/TensorBoard监控

```python
# WandB集成
training_arguments = TrainingArguments(
    output_dir = "outputs",
    report_to = "wandb",        # 启用WandB监控
    run_name = "llama3-finetune",
    
    # TensorBoard配置（本地）
    # report_to = "tensorboard",
)

# 启动训练后，在终端运行：
# wandb login
# tensorboard --logdir outputs/runs
```

### 5.3 模型保存与导出

```python
# 保存LoRA适配器（推荐，用于后续推理）
model.save_pretrained("lora_model")
tokenizer.save_pretrained("lora_model")

# 导出为GGUF格式（用于llama.cpp、Ollama）
from unsloth import save_model_to_gguf

save_model_to_gguf(
    "lora_model",                  # 输入路径
    tokenizer,
    quantization = "Q4_K_M",       # 量化方法
    save_directory = "gguf_model" # 输出路径
)

# 合并到原模型并保存完整模型
model.save_pretrained_merged(
    "merged_model",
    tokenizer,
    save_method = "merged_16bit",  # 或 "merged_4bit"
)
```

---

## 6. 多GPU训练

### 6.1 单机多卡配置

```python
from unsloth import FastLanguageModel
import torch
from transformers import DataParallel

# 检测可用GPU数量
n_gpus = torch.cuda.device_count()
print(f"可用GPU数量: {n_gpus}")

# 加载模型
model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/llama-3-8b-bnb-4bit",
    load_in_4bit = True,
)

# 包装为DataParallel
if n_gpus > 1:
    model = torch.nn.DataParallel(model, device_ids=list(range(n_gpus)))
```

### 6.2 DeepSpeed ZeRO配置

对于更大规模训练，使用DeepSpeed：

```bash
# 创建deepspeed配置文件 ds_config.json
{
    "fp16": {
        "enabled": "auto"
    },
    "bf16": {
        "enabled": "auto"
    },
    "zero_optimization": {
        "stage": 2,
        "offload_optimizer": {
            "device": "cpu"
        },
        "allgather_partitions": true,
        "allgather_bucket_size": 2e8,
        "overlap_comm": true,
        "reduce_scatter": true,
        "reduce_bucket_size": 2e8,
        "contiguous_gradients": true
    },
    "gradient_accumulation_steps": "auto",
    "gradient_clipping": "auto",
    "train_batch_size": "auto",
    "train_micro_batch_size_per_gpu": "auto"
}
```

### 6.3 启动多GPU训练

```bash
# 使用DeepSpeed启动
deepspeed --num_gpus=2 train.py \
    --deepspeed ds_config.json \
    --per_device_train_batch_size 4

# 或使用torchrun
torchrun --nproc_per_node=2 train.py
```

> [!warning]+ 多GPU注意事项
> - 量化模型的多卡训练需要确保各卡显存均衡
> - DeepSpeed配置需根据实际显存调整stage和offload策略
> - 建议先在单卡验证后再扩展到多卡

---

## 7. 实战案例

### 7.1 案例一：对话助手微调

```python
# 对话数据集格式化
def format_conversation(example):
    return {
        "text": f"### Human: {example['instruction']}\n\n### Assistant: {example['output']}"
    }

dataset = dataset.map(format_conversation)

# 使用ChatML模板
tokenizer = get_chat_template(tokenizer, chat_template = "chatml")

# 完整训练脚本
from unsloth import FastLanguageModel
from unsloth.chat_templates import train_on_responses_only
from unsloth.trainers import GenEvalConfig

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/Qwen2.5-7B-bnb-4bit",
    max_seq_length = 4096,
    load_in_4bit = True,
)

model = FastLanguageModel.get_peft_model(
    model,
    r = 32,
    target_modules = "all-linear",
    lora_alpha = 64,
    use_gradient_checkpointing = "unsloth",
)

trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = 4096,
    train_on_responses_only = True,  # 只在响应部分计算损失
    args = TrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 4,
        warmup_ratio = 0.1,
        max_steps = 500,
        learning_rate = 2e-4,
        fp16 = not torch.cuda.is_bf16_compatible(),
        bf16 = torch.cuda.is_bf16_compatible(),
        logging_steps = 10,
        optim = "adamw_8bit",
        output_dir = "chat_assistant",
    ),
)

trainer.train()
```

### 7.2 案例二：代码模型微调

```python
# 代码特定数据集
dataset = load_dataset("smangrul/openhermes-llama2-code", split="train")

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = "unsloth/codellama-7b-bnb-4bit",
    max_seq_length = 4096,
    load_in_4bit = True,
)

model = FastLanguageModel.get_peft_model(
    model,
    r = 16,
    lora_alpha = 32,
    target_modules = ["q_proj", "v_proj", "k_proj", "o_proj",
                      "gate_proj", "up_proj", "down_proj"],
    use_gradient_checkpointing = "unsloth",
)

# 训练配置优化
trainer = SFTTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = 4096,
    args = TrainingArguments(
        per_device_train_batch_size = 4,
        gradient_accumulation_steps = 2,
        max_steps = 1000,
        learning_rate = 3e-4,      # 代码任务可用更高学习率
        lr_scheduler_type = "cosine",
        output_dir = "code_model",
    ),
)

trainer.train()
```

---

## 相关文档

- [[LLaMA_Factory完整指南]] - 另一个主流微调框架
- [[Axolotl使用指南]] - YAML配置的灵活微调工具
- [[DeepSpeed训练优化]] - 分布式训练与显存优化
- [[框架对比与选择]] - 根据场景选择合适的微调工具
