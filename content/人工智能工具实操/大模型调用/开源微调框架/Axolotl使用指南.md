---
title: Axolotl使用指南
date: 2026-04-18
tags:
  - Axolotl
  - YAML配置
  - 多节点训练
  - DeepSpeed
  - 训练恢复
  - 故障排查
  - 分布式训练
  - 高效微调
  - 模型训练
  - SFT/DPO
categories:
  - AI工具实操
  - 大模型调用
  - 开源微调框架
alias: Axolotl Guide
---

# Axolotl使用指南

## 关键词速览

> [!note]+ 关键词
> | Axolotl | YAML配置 | DeepSpeed | 多节点 | FSDP |
> |----------|---------|-----------|--------|------|
> | 训练恢复 | Slurm | Flash Attention | 梯度检查点 | 量化训练 |
> | RLHF/DPO | 故障排查 | GPU集群 | 模型并行 | 断点续训 |

---

## 1. Axolotl设计理念

Axolotl（蝾螈）是由OpenAccess AI Collective维护的开源微调框架，其设计哲学强调**配置即代码**、**灵活性优先**、**生产就绪**。与Unsloth的极致优化、LLaMA Factory的开箱即用不同，Axolotl面向的是需要深度定制和大规模训练的 advanced 用户。

### 1.1 核心理念

#### 配置驱动（Configuration as Code）

Axolotl采用声明式YAML配置文件来定义训练流程，这种设计带来以下优势：

- **版本控制友好**：所有训练配置可以纳入Git管理
- **可复现性**：同一配置文件可在不同环境中重现训练
- **社区共享**：YAML文件易于分享和讨论
- **CI/CD集成**：便于自动化训练流水线

#### 极简抽象层

Axolotl在底层库（Hugging Face Transformers、DeepSpeed、FSDP等）之上构建了统一的抽象层，用户无需深入了解底层细节即可使用高级特性：

```yaml
# Axolotl隐藏了底层复杂性
# 只需声明式配置
base_model: meta-llama/Llama-2-7b-hf
learning_rate: 3e-4
epochs: 3
```

#### 插件式架构

通过YAML配置可以灵活组合不同的组件：

- 多种模型加载器（本地、HF Hub、S3）
- 多种优化器（AdamW、AdamW8bit、bitsandbytes）
- 多种训练策略（LoRA、全参数、QLoRA）
- 多种加速框架（DeepSpeed、FSDP）

### 1.2 适用人群

| 用户类型 | 适合度 | 原因 |
|---------|--------|------|
| 追求极致性能优化 | ⭐⭐⭐ | 支持DeepSpeed/FSDP高级特性 |
| 需要大规模集群训练 | ⭐⭐⭐⭐⭐ | 原生支持多节点/Slurm |
| 习惯图形界面操作 | ⭐⭐ | 纯命令行，无WebUI |
| 快速原型验证 | ⭐⭐⭐ | YAML配置简洁明了 |
| 生产环境部署 | ⭐⭐⭐⭐ | 稳定可靠，社区活跃 |

---

## 2. YAML配置语法

### 2.1 配置结构概览

完整的Axolotl配置通常包含以下部分：

```yaml
# 1. 模型配置
base_model: meta-llama/Llama-2-7b-hf
model_type: LlamaForCausalLM
tokenizer_type: LlamaTokenizer

# 2. 数据配置
datasets:
  - path: abisee/荔/荔
    type: alpaca
dataset_prepared_path: ./data/prepared
val_size: 0.1

# 3. 训练配置
epochs: 3
learning_rate: 3e-4
batch_size: 4
gradient_accumulation_steps: 4

# 4. 优化配置
optimizer:
  type: adamw_torch
  weight_decay: 0.01

# 5. 输出配置
output_dir: ./output/llama-2-7b
```

### 2.2 模型配置详解

```yaml
# 基本模型配置
base_model: Qwen/Qwen2.5-7B-Instruct
base_model_config: Qwen/Qwen2.5-7B-Instruct

# 模型类型（通常自动检测）
model_type: qwen2
tokenizer_type: Qwen2Tokenizer

# 加载选项
load_in_8bit: false
load_in_4bit: true                # QLoRA模式
gptq_groupsize: 128              # GPTQ量化
gptq_bits: 4

# 序列长度
sequence_len: 8192               # 最大序列长度

# 专家混合模型（MoE）
triton: true
special_tokens:
  eot_token: <|im_end|>
  pad_token: <|im_end|>
```

### 2.3 数据集配置

#### 2.3.1 单数据集

```yaml
datasets:
  - path:yahma/alpaca-cleaned
    type: alpaca                # 数据集格式类型
```

#### 2.3.2 多数据集混合

```yaml
datasets:
  - path:yahma/alpaca-cleaned
    type: alpaca
    ds_type: json              # 数据集文件类型
  - path: /path/to/local/data.jsonl
    type: sharegpt
    ds_type: jsonl
  - path: meta-math/MetaMath-Qwen2-7B
    type: chatml
```

#### 2.3.3 数据集格式类型

| 格式 | 用途 | 字段要求 |
|------|------|---------|
| alpaca | 指令微调 | instruction, input, output |
| sharegpt | 对话数据 | messages (角色-内容对) |
| oa_completion | OpenAI格式 | prompt, completion |
| conftest | 对话完成 | conversations |
| mpw | 多提示词 | input, prompts, chosen |

#### 2.3.4 数据预处理

```yaml
# 数据集预处理选项
dataset_prepared_path: ./data/prepared  # 预处理缓存路径
val_size: 0.05                         # 验证集比例
max_packed_sequences: 4                 # 序列打包数量
```

### 2.4 训练配置

```yaml
# 基础训练参数
num_epochs: 3
micro_batch_size: 2                    # per_device_batch_size
gradient_accumulation_steps: 8        # 等效batch_size = 16
learning_rate: 3e-4
optimizer:
  type: adamw_torch
  lr_decay: cosine
  warmup: 0.1                          # 预热比例
  cosine_min_lr_ratio: 0.1

# 高级训练配置
max_steps: -1                         # -1表示按epoch训练
max_seq_len: 4096                     # 最大序列长度
cutoff_len: 2048                      # 截断长度
sample_packing: true                   # 序列打包（提升效率）
```

### 2.5 LoRA/QLora配置

```yaml
# LoRA配置
lora_model_dir: null                   # 加载已有LoRA
lora_alpha: 16
lora_dropout: 0.05
lora_r: 8                             # rank
lora_target_modules:                  # 目标模块
  - q_proj
  - k_proj
  - v_proj
  - o_proj
lora_target_modules_append:           # 追加模块
  - gate_proj
  - up_proj
  - down_proj
lora_fan_in_fan_out: false
lora_linear_bias: false

# QLoRA专用
load_in_4bit: true
bnb_4bit_compute_dtype: bfloat16
bnb_4bit_quant_type: nf4
bnb_4bit_use_double_quant: true
```

> [!tip]+ LoRA模块选择建议
> - **仅注意力层**（q/k/v_proj）：适合轻量微调
> - **注意力+MLP**（all-linear）：最高表达能力，推荐
> - **自定义模块**：根据模型架构调整

---

## 3. 多节点训练

### 3.1 Slurm集群配置

Axolotl原生支持Slurm调度系统：

```yaml
# slurm_config.yml
base_model: meta-llama/Llama-2-70b-hf
cluster_config:
  slurm:
    partition: compute
    account: ml_training
    nodes: 4
    tasks_per_node: 8
    cpus_per_task: 4
    mem: 128G
    time: "72:00:00"
    job_name: llama-70b-finetune
    comment: "QLoRA fine-tuning"
```

```bash
# 使用Slurm提交训练任务
cd /path/to/axolotl
sbatch --nodes=4 --ntasks-per-node=8 ./submit_slurm.sh
```

### 3.2 Slurm启动脚本

```bash
#!/bin/bash
# submit_slarm.sh

#SBATCH --nodes=4
#SBATCH --ntasks-per-node=8
#SBATCH --gres=gpu:8
#SBATCH --time=72:00:00
#SBATCH --partition=compute
#SBATCH --job-name=axolotl-llama

# 加载模块
module load cuda/12.1
module load cudnn/v8.9.6.50-cuda-12.1
module load conda
conda activate axolotl

# 启动训练
srun python -m axolotl.main \
    --config ./examples/llama-2/qlora.yml
```

### 3.3 非Slurm多节点

对于没有Slurm的集群，可以使用SSH或PyTorch Launch：

```bash
# 使用torchrun多节点
torchrun \
    --nnodes=2 \
    --node_rank=0 \
    --nproc_per_node=8 \
    --master_addr=192.168.1.10 \
    --master_port=29500 \
    -m axolotl.main \
    --config ./examples/llama-2/qlora.yml

# 节点1
torchrun \
    --nnodes=2 \
    --node_rank=1 \
    --nproc_per_node=8 \
    --master_addr=192.168.1.10 \
    --master_port=29500 \
    -m axolotl.main \
    --config ./examples/llama-2/qlora.yml
```

### 3.4 多节点网络配置

```yaml
# 网络优化配置
nccl_config:
  - NCCL_IB_DISABLE: 0
  - NCCL_NET_GDR_LEVEL: P2P
  - NCCL_GRAPH_MAPPING_ENABLE: 1
  - NCCL_GRAPH_ENABLE: 1
  - NCCL_TOPOLOGY_FILE: /path/to/topology.file
```

> [!warning]+ 多节点注意事项
> - 确保所有节点间网络畅通（SSH无密码访问）
> - 共享存储路径在各节点保持一致
> - 检查NCCL拓扑配置以优化通信效率

---

## 4. DeepSpeed配置

### 4.1 DeepSpeed ZeRO阶段

```yaml
# ZeRO Stage 1
# 仅分片优化器状态
deepspeed:
  stage: 1
  offload_optimizer_dev: cpu
  offload_optimizer_cpu: nvme

# ZeRO Stage 2（推荐用于70B以下模型）
deepspeed:
  stage: 2
  offload_optimizer:
    device: cpu
    pin_memory: true
  offload_param:
    device: none
  allgather_bucket_size: 5e7
  reduce_bucket_size: 5e7
  overlap_comm: true

# ZeRO Stage 3（用于超大模型）
deepspeed:
  stage: 3
  offload_optimizer:
    device: cpu
  offload_param:
    device: cpu
  overlap_comm: true
  reduce_scatter: true
 contiguous_gradients: true
```

### 4.2 完整DeepSpeed配置

```yaml
# 完整DeepSpeed配置
deepspeed:
  stage: 2
  offload_optimizer:
    device: cpu
    pin_memory: true
  offload_param:
    device: none
  zero_optimization:
    stage: 2
    allgather_partitions: true
    allgather_bucket_size: 5e7
    overlap_comm: true
    reduce_scatter: true
    reduce_bucket_size: 5e7
    contiguous_gradients: true
  fp16:
    enabled: false
  bf16:
    enabled: true                      # 推荐使用BF16
  gradient_clipping: 1.0
  gradient_accumulation_steps: 4
  steps_per_print: 10
  train_batch_size: auto
  train_micro_batch_size_per_gpu: auto
```

### 4.3 启动DeepSpeed训练

```bash
# Axolotl会自动检测并启动DeepSpeed
python -m axolotl.main \
    --config ./examples/llama-2/qlora-deepspeed.yml

# 或显式指定
CUDA_VISIBLE_DEVICES=0,1,2,3,4,5,6,7 \
    deepspeed train.py \
    --num_gpus 8 \
    --config ./examples/llama-2/qlora-deepspeed.yml
```

### 4.4 DeepSpeed通信优化

```yaml
deepspeed:
  # 通信优化
  communication_data_type: fp16
  gradient_predivide_factor: 8.0
  
  # 内存优化
  round_robin_gradients: true
  hbm_estimate_model_size: auto
```

---

## 5. 训练监控与恢复

### 5.1 WandB集成

```yaml
# Weights & Biases配置
wandb_mode: online                  # online/offrun/offline
wandb_project: llama-finetune
wandb_watch: false
wandb_run_name: llama2-7b-qlora-exp1
wandb_entity: my_team               # 组织
wandb_dir: ./logs/wandb
```

```bash
# 或通过环境变量配置
export WANDB_API_KEY=your_api_key
export WANDB_MODE=online
```

### 5.2 TensorBoard集成

```yaml
# TensorBoard配置
tensorboard:
  enabled: true
  log_dir: ./logs/tensorboard
```

```bash
# 启动TensorBoard
tensorboard --logdir ./logs/tensorboard --port 6006

# 或使用Axolotl内置
python -m axolotl.main \
    --config ./examples/llama-2/qlora.yml \
    --tensorboard_dir ./logs/tensorboard
```

### 5.3 断点恢复训练

```bash
# Axolotl会自动保存检查点
# 使用--resume_from_checkpoint恢复
python -m axolotl.main \
    --config ./examples/llama-2/qlora.yml \
    --resume_from_checkpoint ./output/checkpoints/global_step_500

# 从最新检查点恢复
python -m axolotl.main \
    --config ./examples/llama-2/qlora.yml \
    --resume_from_checkpoint true

# 从HF Hub恢复
python -m axolotl.main \
    --config ./examples/llama-2/qlora.yml \
    --resume_from_checkpoint hf://username/checkpoint_name
```

### 5.4 自动检查点配置

```yaml
# 检查点保存配置
save_strategy: steps               # steps/epochs
save_steps: 100
save_total_limit: 5               # 保留的最大检查点数量
save_safetensors: true             # 使用safetensors格式
```

---

## 6. 常见问题排查

### 6.1 CUDA OOM（显存不足）

> [!danger]+ OOM问题排查
> 显存溢出是最常见问题，排查顺序：

1. **减少batch size**
   ```yaml
   micro_batch_size: 1             # 从小值开始
   gradient_accumulation_steps: 16  # 用梯度累积补偿
   ```

2. **启用梯度检查点**
   ```yaml
   gradient_checkpointing: true
   ```

3. **使用QLoRA量化**
   ```yaml
   load_in_4bit: true
   bnb_4bit_compute_dtype: bfloat16
   ```

4. **降低序列长度**
   ```yaml
   max_seq_len: 2048              # 适当降低
   ```

5. **启用DeepSpeed ZeRO**
   ```yaml
   deepspeed:
     stage: 2
   ```

### 6.2 数据集加载错误

> [!danger]+ 常见数据集问题

1. **格式不匹配**
   ```bash
   # 错误：类型指定错误
   datasets:
     - path: my_data.jsonl
       type: alpaca  # 应该是 sharegpt
   
   # 检查数据集实际格式
   head -n 5 my_data.jsonl
   ```

2. **字段名称错误**
   ```yaml
   # Alpaca格式字段
   - path: my_alpaca_data.json
     type: alpaca
     field_mapping:                 # 自定义字段映射
       instruction: prompt
       output: response
   ```

3. **编码问题**
   ```bash
   # 检查文件编码
   file my_data.jsonl
   # 转换为UTF-8
   iconv -f GBK -t UTF-8 input.json > output.json
   ```

### 6.3 训练loss不下降

> [!warning]+ Loss异常排查

1. **学习率问题**
   ```yaml
   learning_rate: 3e-4             # 典型值范围
   warmup_steps: 100               # 确保预热足够
   ```

2. **数据质量问题**
   ```bash
   # 检查数据是否有重复
   python -c "import json; data = [json.loads(l) for l in open('data.jsonl')]; print(f'Total: {len(data)}, Unique: {len(set(data))}')"
   ```

3. **标签混淆**
   ```yaml
   # 检查标签是否正确
   train_on_inputs: false           # 不在输入上训练
   ```

### 6.4 多节点通信问题

> [!danger]+ 分布式训练问题

1. **NCCL超时**
   ```bash
   # 设置NCCL超时
   export NCCL_TIMEOUT=1800
   export NCCL_DEBUG=INFO
   ```

2. **网络连通性测试**
   ```bash
   # 在所有节点测试
   NCCL_IB_DISABLE=1 python -c "import torch; torch.distributed.is_available()"
   ```

3. **共享存储路径**
   ```bash
   # 确保所有节点可访问相同路径
   ls -la /shared/storage/path
   ```

### 6.5 模型加载问题

> [!danger]+ 模型相关问题

1. **HuggingFace Token问题**
   ```bash
   # 登录HuggingFace
   huggingface-cli login
   
   # 或设置token
   export HF_TOKEN=your_token_here
   ```

2. **模型格式不兼容**
   ```yaml
   # 强制指定模型类型
   base_model: ./local/llama-2-7b
   model_type: llama
   tokenizer_type: LlamaTokenizer
   ```

3. **安全文件缺失**
   ```bash
   # 重新下载config.json
   python -c "
   from transformers import AutoConfig
   cfg = AutoConfig.from_pretrained('model_name')
   cfg.save_pretrained('./local_model')
   "
   ```

---

## 相关文档

- [[Unsloth使用指南]] - 快速微调入门
- [[LLaMA_Factory完整指南]] - WebUI友好的框架
- [[DeepSpeed训练优化]] - 分布式训练核心技术
- [[框架对比与选择]] - 选择合适的微调工具
