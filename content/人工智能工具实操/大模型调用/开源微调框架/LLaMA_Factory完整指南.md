---
title: LLaMA Factory完整指南
date: 2026-04-18
tags:
  - LLaMA Factory
  - 微调框架
  - WebUI
  - 命令行
  - 多模态
  - 分布式训练
  - 数据集配置
  - 高效微调
  - PEFT
  - DPO/RLHF
categories:
  - AI工具实操
  - 大模型调用
  - 开源微调框架
alias: LLaMA Factory Complete Guide
---

# LLaMA Factory完整指南

## 关键词速览

> [!note]+ 关键词
> | LLaMA Factory | WebUI | CLI | 数据集管理 | 模态支持 |
> |---------------|-------|-----|------------|---------|
> | 单步启动 | 多模态微调 | 分布式训练 | 断点续训 | 国产模型 |
> | Qwen/ChatGLM | DeepSpeed | Flash Attention | 多卡并行 | 模型注册 |

---

## 1. LLaMA Factory功能概览

LLaMA Factory是由HIASUNA团队开发的开源大模型微调平台，其核心设计理念是**降低微调门槛、提升训练效率、支持多元化需求**。与Unsloth专注于极致优化不同，LLaMA Factory提供了更全面的功能和更友好的交互方式。

### 1.1 核心功能矩阵

| 功能模块 | 支持内容 |
|---------|---------|
| 微调方法 | LoRA、QLoRA、全参数微调、Free Fine-tuning |
| 模型支持 | 100+开源模型，包括LLaMA、Qwen、ChatGLM、Mistral等 |
| 训练方式 | 单GPU、多GPU、分布式训练 |
| 数据集 | 内置丰富数据集、支持自定义上传、格式自动转换 |
| 交互界面 | WebUI图形界面、命令行、Python API |
| 多模态 | 图像描述、视觉问答等多模态任务支持 |

### 1.2 架构设计

LLaMA Factory采用模块化设计，核心组件包括：

- **Trainer模块**：封装Hugging Face Trainer，支持多种训练策略
- **Model Loader**：统一的模型加载接口，支持量化加载
- **Dataset Manager**：数据集预处理、格式转换、数据增强
- **WebUI Server**：基于Gradio的图形化界面
- **CLI Parser**：命令行参数解析与执行
- **Monitor**：训练过程监控与日志记录

### 1.3 与竞品对比

| 特性 | LLaMA Factory | Unsloth | Axolotl |
|------|--------------|---------|---------|
| 界面友好度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| 训练速度 | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| 灵活性 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| 多模态支持 | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| 学习曲线 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |

---

## 2. 支持的模型与微调方法

### 2.1 支持的模型列表（部分）

#### LLaMA系列
- LLaMA（7B/13B/30B/65B）
- LLaMA-2（7B/13B/70B）
- LLaMA-3/3.1/3.2（8B/70B/405B）
- CodeLLaMA（7B/13B/34B/70B）

#### 国内开源模型
| 模型 | 开发商 | 特点 |
|------|--------|------|
| Qwen系列 | 阿里云 | 强大中文能力 |
| ChatGLM系列 | 智谱AI | 对话优化 |
| Yi系列 | 零一万物 | 中英双语 |
| DeepSeek系列 | 深度求索 | 代码能力突出 |
| Baichuan系列 | 百川智能 | 通用对话 |
|InternLM系列 | 上海AI Lab | 开源标杆 |

#### 其他主流模型
- Mistral 7B / Mixtral 8x7B
- Gemma 2B / 7B / 9B
- Phi-3 Mini / Small / Medium
- StarCoder / StarCoder2
- BLOOM / BLOOMZ

### 2.2 微调方法支持

LLaMA Factory支持多种高效微调方法：

#### LoRA系列
```yaml
# LoRA配置示例
finetuning_type: lora
lora_alpha: 16
lora_dropout: 0.05
lora_rank: 8
lora_target: all
```

#### QLoRA（量化+LoRA）
```yaml
# QLoRA配置
finetuning_type: lora
quantization_bit: 4
bnb_kwargs:
  llm_int8_threshold: 6.0
  llm_int8_has_fp16_weight: false
```

#### 全参数微调
```yaml
# 全参数微调配置
finetuning_type: full
```

#### Freeze微调（冻结部分参数）
```yaml
# Freeze微调
finetuning_type: freeze
freeze_alpha: 32
freeze_trainable_layers: 4
```

### 2.3 高级训练方法

LLaMA Factory还支持多种进阶训练范式：

- **DPO（Direct Preference Optimization）**：直接偏好优化
- **ORPO（Odds Ratio Preference Optimization）**：赔率比偏好优化
- **PreDPO**：DPO的预训练版本
- **KTO（Kullback-Leibler Optimal Transport）**：KL散度最优传输

```yaml
# DPO训练配置示例
stage: dpo
pretainer_model_or_path: /path/to/sft_model
finetuning_type: lora
lora_rank: 8
lora_alpha: 16
```

---

## 3. WebUI使用

### 3.1 启动WebUI

LLaMA Factory提供开箱即用的Web界面：

```bash
# 方式一：直接启动（自动检测配置）
llamafactory-cli webui

# 方式二：指定参数启动
llamafactory-cli webui \
    --stage sft \
    --model_name_or_path Qwen/Qwen2.5-7B-Instruct \
    --template chatml \
    --finetuning_type lora \
    --lora_rank 8 \
    --lora_alpha 16

# 方式三：使用配置文件启动
llamafactory-cli webui --config examples/train_full/qwen2_full.yaml
```

### 3.2 WebUI界面说明

启动后访问 `http://localhost:7860`，界面主要包含以下模块：

#### 3.2.1 模型选择区
- **模型名称/路径**：支持Hugging Face模型名或本地路径
- **模型类型**：自动检测或手动指定
- **量化等级**：FP16/BF16/INT8/INT4

#### 3.2.2 数据集配置
- **预置数据集**：从下拉菜单选择内置数据集
- **自定义数据集**：上传JSONL文件
- **数据集预览**：查看数据样例

#### 3.2.3 训练参数区
- **LoRA配置**：rank、alpha、dropout
- **训练参数**：batch size、learning rate、epoch
- **优化器配置**：AdamW/AdamW8bit/SGD
- **学习率调度**：cosine/linear/constant

#### 3.2.4 输出配置
- **输出目录**：训练结果保存路径
- **日志级别**：DEBUG/INFO/WARNING
- **定期保存**：保存间隔设置

### 3.3 WebUI训练流程

1. **选择模型**：在左侧面板选择或上传模型
2. **配置数据集**：添加训练数据集和验证数据集
3. **设置训练参数**：根据硬件配置调整参数
4. **启动训练**：点击"Start Training"按钮
5. **监控进度**：实时查看loss曲线和GPU占用
6. **导出模型**：训练完成后导出为LoRA权重或完整模型

> [!tip]+ WebUI使用技巧
> - 使用"Preview Dataset"功能在训练前检查数据格式
> - 启用"Resume Training"可以从断点恢复训练
> - "Export Model"支持直接导出为Ollama兼容格式

---

## 4. 命令行训练

### 4.1 基本命令格式

```bash
llamafactory-cli train \
    --stage sft \
    --model_name_or_path Qwen/Qwen2.5-7B-Instruct \
    --dataset alpaca_en,alpaca_zh \
    --template chatml \
    --finetuning_type lora \
    --lora_rank 8 \
    --lora_alpha 16 \
    --output_dir ./output/qwen2.5-7b-lora \
    --overwrite_cache \
    --do_train \
    --fp16
```

### 4.2 YAML配置文件

推荐使用YAML文件管理复杂配置：

```yaml
# examples/train_lora/qwen2_lora.yaml
### 模型配置
model_name_or_path: Qwen/Qwen2.5-7B-Instruct
quantization_bit: 4                    # 4bit量化
bnb_4bit_compute_dtype: bf16          # 计算精度

### 数据集配置
dataset: alpaca_en,alpaca_zh           # 多个数据集用逗号分隔
dataset_dir: data
template: chatml                       # 模板格式
cutoff_len: 2048                       # 最大序列长度
max_samples: 100000                    # 最大样本数
overwrite_cache: true

### LoRA配置
finetuning_type: lora
lora_rank: 8
lora_alpha: 16
lora_dropout: 0.05
lora_target: all                       # 对所有层应用LoRA

### 训练参数
stage: sft
per_device_train_batch_size: 2
gradient_accumulation_steps: 8
learning_rate: 5.0e-5
num_train_epochs: 3.0
lr_scheduler_type: cosine
warmup_ratio: 0.1
bf16: true
optim: adamw_torch
max_grad_norm: 1.0
logging_steps: 10
save_steps: 500
val_size: 0.01
val_size_ratio_to_sample: true

### 输出配置
output_dir: ./output/qwen2.5-7b-lora
logging_dir: ./output/logs
```

### 4.3 使用配置文件训练

```bash
# 使用YAML配置文件启动训练
llamafactory-cli train examples/train_lora/qwen2_lora.yaml

# 覆盖特定参数
llamafactory-cli train examples/train_lora/qwen2_lora.yaml \
    --output_dir ./output/my_model \
    --num_train_epochs 5
```

---

## 5. 数据集配置

### 5.1 数据集格式

LLaMA Factory支持多种数据集格式：

#### 5.1.1 Alpaca格式
```json
{
    "instruction": "给出三个健康饮食的建议",
    "input": "",
    "output": "1. 多吃蔬菜水果\n2. 控制糖分摄入\n3. 保持规律饮食习惯"
}
```

#### 5.1.2 对话格式（ChatML）
```json
{
    "messages": [
        {"role": "user", "content": "你好"},
        {"role": "assistant", "content": "你好！有什么可以帮助你的吗？"}
    ]
}
```

#### 5.1.3 多轮对话格式
```json
{
    "messages": [
        {"role": "system", "content": "你是一个有帮助的助手"},
        {"role": "user", "content": "什么是机器学习？"},
        {"role": "assistant", "content": "机器学习是..."},
        {"role": "user", "content": "能举个例子吗？"},
        {"role": "assistant", "content": "当然可以..."}
    ]
}
```

### 5.2 dataset_info.json配置

在`data`目录下创建`dataset_info.json`注册数据集：

```json
{
    "my_dataset": {
        "file_name": "my_data.jsonl",
        "formatting": "alpaca",
        "columns": {
            "prompt": "instruction",
            "response": "output"
        }
    },
    "chat_dataset": {
        "file_name": "chat_data.jsonl",
        "formatting": "sharegpt",
        "columns": {
            "messages": "messages"
        }
    }
}
```

### 5.3 内置数据集

LLaMA Factory内置了丰富的数据集：

| 数据集名称 | 用途 | 语言 |
|-----------|------|------|
| alpaca_en | 指令微调 | 英文 |
| alpaca_zh | 指令微调 | 中文 |
| belle_multiturn | 多轮对话 | 中文 |
| codealpaca | 代码生成 | 英文 |
| orca_math | 数学推理 | 英文 |
| openassistant | 通用对话 | 多语言 |

### 5.4 自定义数据集使用

```yaml
# 使用自定义数据集
dataset: my_custom_dataset,alpaca_zh
dataset_dir: ./my_data        # 自定义数据目录

# 数据集混合比例
sample_packing: false          # 禁用样本打包
max_length: 2048               # 最大长度
```

> [!warning]+ 数据集注意事项
> - 确保JSONL文件每行都是有效的JSON
> - 中文字符需要正确UTF-8编码
> - 超长样本会被截断，建议预处理时控制长度

---

## 6. 多模态支持（Vision）

### 6.1 支持的视觉语言模型

LLaMA Factory支持多种VLM微调：

| 模型 | 描述 |
|------|------|
| LLaVA系列 | LLaMA + CLIP视觉编码器 |
| Qwen-VL | 阿里云视觉语言模型 |
| InternVL | 上海AI Lab视觉模型 |
| Phi-3.5-Vision | 微软多模态模型 |

### 6.2 多模态配置

```yaml
# 多模态微调配置
### 模型配置
model_name_or_path: lmms-lab/llava-1.5-7b-hf
quantization_bit: 4

### 数据集配置（图像描述）
dataset: llava_en_sft,llava_zh_sft
dataset_dir: ./data/vlm
template: llava
image_folder: ./data/vlm/images

### 视觉LoRA配置
finetuning_type: lora
lora_rank: 16
lora_alpha: 32
lora_target: vision_modules      # 只对视觉模块应用LoRA

### 训练参数
per_device_train_batch_size: 1
gradient_accumulation_steps: 8
learning_rate: 1.0e-4
num_train_epochs: 1.0
bf16: true
```

### 6.3 多模态数据集格式

```json
{
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "image", "image": "path/to/image.jpg"},
                {"type": "text", "text": "描述这张图片"}
            ]
        },
        {
            "role": "assistant",
            "content": "这张图片展示..."
        }
    ]
}
```

---

## 7. 分布式训练配置

### 7.1 单机多卡

#### 7.1.1 DeepSpeed ZeRO配置

```yaml
# ds_config.json
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
            "device": "cpu",
            "pin_memory": true
        },
        "allgather_partitions": true,
        "allgather_bucket_size": 5e7,
        "overlap_comm": true,
        "reduce_scatter": true,
        "reduce_bucket_size": 5e7,
        "contiguous_gradients": true,
        "round_robin_gradients": true
    },
    "gradient_accumulation_steps": "auto",
    "gradient_clipping": "auto",
    "train_batch_size": "auto",
    "train_micro_batch_size_per_gpu": "auto"
}
```

#### 7.1.2 启动分布式训练

```bash
# 使用DeepSpeed单卡
llamafactory-cli train examples/train_lora/qwen2_lora.yaml \
    --deepspeed ds_config.json

# 使用DeepSpeed多卡
torchrun --nproc_per_node=2 \
    -m llmtuner.hparams.deepspeed \
    examples/train_lora/qwen2_lora.yaml \
    --deepspeed ds_config.json

# 使用FSDP
llamafactory-cli train examples/train_lora/qwen2_lora.yaml \
    --fsdp full \
    --fsdp_config fsdp_config.json
```

### 7.2 多节点训练

```bash
# 多节点配置
# 节点0执行
torchrun \
    --nnodes=2 \
    --node_rank=0 \
    --nproc_per_node=8 \
    --master_addr=192.168.1.1 \
    --master_port=29500 \
    -m llmtuner.hparams.deepspeed \
    examples/train_lora/qwen2_lora.yaml \
    --deepspeed ds_config.json

# 节点1执行（只需修改node_rank）
torchrun \
    --nnodes=2 \
    --node_rank=1 \
    --nproc_per_node=8 \
    --master_addr=192.168.1.1 \
    --master_port=29500 \
    -m llmtuner.hparams.deepspeed \
    examples/train_lora/qwen2_lora.yaml \
    --deepspeed ds_config.json
```

### 7.3 梯度检查点配置

```yaml
# 启用梯度检查点以节省显存
gradient_checkpointing: true
gradient_checkpointing_ratio: 1.0
```

### 7.4 混合精度与优化器

```yaml
# 混合精度训练
fp16: false
bf16: true                        # 推荐使用BF16

# 优化器配置
optim: adamw_torch                # 或 adamw_8bit省显存
optim_kwargs:
  betas: [0.9, 0.999]
  weight_decay: 0.01
lr_scheduler_type: cosine
warmup_ratio: 0.1
```

---

## 相关文档

- [[Unsloth使用指南]] - 高效快速的微调方案
- [[Axolotl使用指南]] - YAML配置驱动的灵活框架
- [[DeepSpeed训练优化]] - 分布式训练核心技术
- [[框架对比与选择]] - 根据场景选择合适工具
