---
title: 数据格式化与Tokenization
date: 2026-04-18
tags:
  - 数据格式化
  - Tokenization
  - ChatML
  - ShareGPT
  - 分词器配置
  - 特殊Token
  - 上下文管理
  - 数据集打包
categories:
  - 人工智能
  - 大模型调用
  - 数据处理
alias: [大模型数据格式化, Token化处理, 数据格式转换]
---

## 关键词

| 数据格式化 | Tokenization | ChatML | ShareGPT | 分词器 | 特殊Token | 上下文长度 | 数据集打包 | HF数据集 | 序列长度 |

---

## 一、训练数据格式化

### 1.1 主流数据格式概述

在大模型微调训练中，数据的格式化方式直接影响模型对对话结构的理解和生成能力。不同的训练框架和模型架构可能采用不同的数据格式，但核心目标一致：将多轮对话转换为模型可以学习的序列形式。

#### 主要格式对比

| 格式 | 特点 | 适用场景 | 代表框架 |
|-----|-----|---------|---------|
| ChatML | 显式标记角色，使用特殊Token包裹消息 | 通用对话训练 | LLaMA-Factory, Axolotl |
| ShareGPT | 简洁格式，from/to字段标识角色 | 对话数据共享 | 多数开源项目 |
| OpenAI | 系统/用户/助手分离 | API格式兼容 | OpenAI微调 |
| Anthropic | 详细的人类/助手标注格式 | RLHF训练 | Claude相关 |
| Alpaca | 简单指令格式 | 指令微调 | Stanford Alpaca |

### 1.2 ChatML格式详解

ChatML（Chat Markup Language）是一种广泛使用的对话格式，其核心思想是为每个角色和消息边界添加明确的特殊Token标记，使模型能够清晰识别对话结构。

#### ChatML格式规范

```python
class ChatMLFormatter:
    """ChatML格式转换器"""
    
    SYSTEM_TOKEN = "<|im_start|>"
    SYSTEM_END_TOKEN = "<|im_end|>"
    USER_TOKEN = "<|im_start|>user"
    ASSISTANT_TOKEN = "<|im_start|>assistant"
    END_TOKEN = "<|im_end|>"
    
    def format_single_turn(self, instruction, response):
        """
        格式化单轮对话
        
        格式：
        <|im_start|>user
        {instruction}<|im_end|>
        <|im_start|>assistant
        {response}<|im_end|>
        """
        return (
            f"{self.USER_TOKEN}\n"
            f"{instruction}"
            f"{self.END_TOKEN}\n"
            f"{self.ASSISTANT_TOKEN}\n"
            f"{response}"
            f"{self.END_TOKEN}"
        )
        
    def format_multi_turn(self, conversation):
        """
        格式化多轮对话
        
        Args:
            conversation: [{"role": "user/assistant/system", "content": "..."}]
        """
        formatted = []
        
        for turn in conversation:
            role = turn["role"]
            content = turn["content"]
            
            if role == "system":
                formatted.append(
                    f"{self.SYSTEM_TOKEN}\n{content}{self.END_TOKEN}"
                )
            elif role == "user":
                formatted.append(
                    f"{self.USER_TOKEN}\n{content}{self.END_TOKEN}"
                )
            elif role == "assistant":
                formatted.append(
                    f"{self.ASSISTANT_TOKEN}\n{content}{self.END_TOKEN}"
                )
                
        return "\n".join(formatted)
        
    def format_for_training(self, conversation, include_labels=True):
        """
        格式化用于训练的数据
        
        训练时需要计算loss的部分是assistant的回复
        """
        formatted = self.format_multi_turn(conversation)
        
        # 计算prompt长度（不计算loss的部分）
        prompt_length = len(self.tokenize(formatted.split(
            f"{self.ASSISTANT_TOKEN}\n"
        )[0] + f"{self.ASSISTANT_TOKEN}\n"))
        
        return {
            "text": formatted,
            "prompt_length": prompt_length,
            "labels": self._create_labels(formatted, prompt_length)
        }
```

#### ChatML完整示例

```
<|im_start|>system
你是一个有帮助的AI助手。请用简洁、专业的方式回答用户的问题。<|im_end|>
<|im_start|>user
请解释什么是机器学习中的过拟合现象？<|im_end|>
<|im_start|>assistant
过拟合（Overfitting）是指机器学习模型在训练数据上表现很好，但在测试数据或新数据上表现较差的现象。

产生过拟合的主要原因：
1. **模型过于复杂**：模型的参数量远大于训练数据量
2. **训练时间过长**：模型过度学习了训练数据中的噪声和细节
3. **训练数据不足**：数据量太小，无法代表真实分布

常见的解决策略：
- 增加训练数据量
- 使用正则化技术（L1/L2）
- 采用Dropout机制
- 交叉验证选择最佳模型
- 简化模型结构<|im_end|>
<|im_start|>user
那欠拟合呢？<|im_end|>
<|im_start|>assistant
欠拟合（Underfitting）与过拟合相反，是指模型在训练数据和测试数据上都表现不佳，无法捕捉数据中的基本规律。

产生欠拟合的原因：
1. **模型过于简单**：模型容量不足
2. **特征不足**：输入特征没有包含足够信息
3. **训练不充分**：训练轮数不够

解决欠拟合的方法：
- 增加模型复杂度
- 添加更多特征
- 增加训练时间
- 减少正则化强度<|im_end|>
```

### 1.3 ShareGPT格式

ShareGPT格式是另一种常用的对话格式，以其简洁性著称：

```python
class ShareGPTFormatter:
    """ShareGPT格式转换器"""
    
    def format_conversation(self, conversation):
        """
        转换为ShareGPT格式
        
        格式示例：
        {
            "conversations": [
                {"from": "human", "value": "..."},
                {"from": "gpt", "value": "..."}
            ]
        }
        """
        # 角色映射
        role_map = {
            "user": "human",
            "assistant": "gpt",
            "system": "human"  # system消息合并到首条user消息
        }
        
        conversations = []
        
        for turn in conversation:
            role = role_map.get(turn["role"], turn["role"])
            conversations.append({
                "from": role,
                "value": turn["content"]
            })
            
        return {
            "conversations": conversations
        }
        
    def parse_sharegpt(self, data):
        """解析ShareGPT格式数据"""
        if "conversations" not in data:
            return []
            
        result = []
        for conv in data["conversations"]:
            role = "user" if conv["from"] == "human" else "assistant"
            result.append({
                "role": role,
                "content": conv["value"]
            })
            
        return result
```

### 1.4 多格式转换工具

```python
class DataFormatConverter:
    """多格式数据转换器"""
    
    def __init__(self):
        self.formatters = {
            "chatml": ChatMLFormatter(),
            "sharegpt": ShareGPTFormatter(),
            "openai": OpenAIFormatter(),
            "alpaca": AlpacaFormatter()
        }
        
    def convert(self, data, from_format, to_format):
        """
        格式转换
        
        Args:
            data: 输入数据
            from_format: 源格式
            to_format: 目标格式
        """
        if from_format not in self.formatters:
            raise ValueError(f"不支持的源格式: {from_format}")
        if to_format not in self.formatters:
            raise ValueError(f"不支持的目标格式: {to_format}")
            
        # 转换为中间格式（统一对话列表）
        conversation = self._to_universal(data, from_format)
        
        # 从中间格式转换到目标格式
        return self._from_universal(conversation, to_format)
        
    def _to_universal(self, data, format_name):
        """转换为统一中间格式"""
        if format_name == "chatml":
            return self._parse_chatml(data)
        elif format_name == "sharegpt":
            return self.formatters["sharegpt"].parse_sharegpt(data)
        elif format_name == "alpaca":
            return [{"role": "user", "content": data["instruction"]},
                   {"role": "assistant", "content": data["output"]}]
        else:
            return data
            
    def _from_universal(self, conversation, format_name):
        """从统一格式转换"""
        formatter = self.formatters.get(format_name)
        
        if format_name == "chatml":
            return formatter.format_multi_turn(conversation)
        elif format_name == "sharegpt":
            return formatter.format_conversation(conversation)
        elif format_name == "alpaca":
            return {
                "instruction": conversation[0]["content"],
                "output": conversation[1]["content"]
            }
        else:
            return conversation
```

---

## 二、分词器（Tokenizer）配置

### 2.1 分词器选择原则

分词器是连接原始文本与模型输入的关键组件，其选择直接影响：

- **词汇表大小**：影响模型参数量和嵌入层大小
- **Token效率**：决定上下文长度的有效利用率
- **多语言支持**：不同分词器对各语言的处理效率差异显著
- **特殊Token处理**：影响格式标记和系统提示的编码

#### 主流分词器对比

| 分词器 | 模型 | 词汇表大小 | 中文Token数/字 | 特点 |
|-------|-----|-----------|---------------|------|
| GPT-4 Tokenizer | GPT-4 | ~100k | ~1.3-2 | 优化英文，中文效率较低 |
| Claude Tokenizer | Claude | ~100k | ~1.5 | 均衡设计 |
| TikToken | GPT系列 | 100k | ~2 | 高效，OpenAI官方使用 |
| SentencePiece | LLaMA/BLOOM | 32k-200k | ~1.5 | 开源模型常用 |
| Jieba+自定义 | 中文专用 | 可变 | 1 | 最优中文效率 |

### 2.2 分词器配置与使用

```python
from transformers import AutoTokenizer, PreTrainedTokenizer
import tiktoken

class TokenizerConfigurator:
    """分词器配置管理器"""
    
    def __init__(self):
        self.tokenizers = {}
        
    def load_tokenizer(self, model_name, trust_remote_code=True):
        """加载分词器"""
        if model_name not in self.tokenizers:
            self.tokenizers[model_name] = AutoTokenizer.from_pretrained(
                model_name,
                trust_remote_code=trust_remote_code
            )
        return self.tokenizers[model_name]
        
    def count_tokens(self, text, model_name="gpt-4"):
        """计算Token数量"""
        if model_name.startswith("gpt"):
            return self._count_tiktoken(text, model_name)
        else:
            tokenizer = self.load_tokenizer(model_name)
            return len(tokenizer.encode(text))
            
    def _count_tiktoken(self, text, model_name):
        """使用TikToken计算Token数"""
        encoding = tiktoken.encoding_for_model(model_name)
        return len(encoding.encode(text))
        
    def truncate_sequence(self, text, max_tokens, model_name, 
                        truncation_side="right"):
        """
        截断序列到指定Token数
        """
        tokenizer = self.load_tokenizer(model_name)
        
        encoded = tokenizer.encode(
            text,
            truncation=True,
            max_length=max_tokens,
            truncation_side=truncation_side
        )
        
        return tokenizer.decode(encoded)
        
    def batch_encode(self, texts, model_name, max_length=2048):
        """批量编码"""
        tokenizer = self.load_tokenizer(model_name)
        
        return tokenizer(
            texts,
            padding=True,
            truncation=True,
            max_length=max_length,
            return_tensors="pt"
        )
```

### 2.3 词汇表扩展

```python
class VocabularyExtender:
    """词汇表扩展器"""
    
    def __init__(self, base_tokenizer):
        self.tokenizer = base_tokenizer
        self.original_vocab_size = len(self.tokenizer)
        
    def add_special_tokens(self, special_tokens):
        """
        添加特殊Token
        
        Args:
            special_tokens: [{"id": 1, "content": "[TOOL_CALL]"}, ...]
        """
        # 定义新的特殊Token
        new_special_tokens = {
            "additional_special_tokens": [t["content"] for t in special_tokens]
        }
        
        # 添加到词汇表
        self.tokenizer.add_special_tokens(new_special_tokens)
        
        # 更新token到ID的映射
        for st in special_tokens:
            st["token_id"] = self.tokenizer.convert_tokens_to_ids(st["content"])
            
        return self.tokenizer
        
    def add_domain_tokens(self, domain_words):
        """
        添加领域特定词汇
        
        使用BPE分词器的词缀功能
        """
        # 对领域词汇进行预-tokenize
        new_tokens = []
        
        for word in domain_words:
            # 检查是否已经是独立token
            if word not in self.tokenizer vocab:
                new_tokens.append(word)
                
        # 添加新token
        num_added = self.tokenizer.add_tokens(new_tokens)
        
        return {
            "tokens_added": num_added,
            "new_vocab_size": len(self.tokenizer),
            "original_vocab_size": self.original_vocab_size
        }
        
    def merge_tokens(self, phrase, target_token):
        """
        合并高频短语为单一Token
        
        适用于固定的专有名词或短语
        """
        # 添加目标token
        self.tokenizer.add_tokens([target_token])
        
        # 定义合并规则
        new_token_id = self.tokenizer.convert_tokens_to_ids(target_token)
        
        # 重新训练tokenizer以合并短语
        # 注意：这需要重新训练BPE模型
        return new_token_id
```

---

## 三、特殊Token使用

### 3.1 特殊Token分类体系

在大模型训练中，特殊Token承担着结构标记、角色标识、格式控制等多重功能。

#### 功能分类

| Token类型 | 示例 | 功能 | 训练策略 |
|----------|------|------|---------|
| 角色Token | `<|im_start|>user` | 标识发言者 | 学习 |
| 边界Token | `<|im_end|>` | 标识消息边界 | 学习 |
| 系统Token | `<|system|>` | 系统级指令 | 重点学习 |
| 填充Token | `<|pad|>` | 序列填充 | 不计算loss |
| 未知Token | `<|unk|>` | 未登录词 | - |
| 序列Token | `<|endoftext|>` | 序列结束 | 学习 |

### 3.2 特殊Token配置

```python
class SpecialTokenManager:
    """特殊Token管理器"""
    
    DEFAULT_SYSTEM_TOKENS = {
        "chatml": {
            "system_start": "<|im_start|>system",
            "system_end": "<|im_end|>",
            "user_start": "<|im_start|>user",
            "assistant_start": "<|im_start|>assistant",
            "eos": "<|im_end|>"
        },
        "qwen": {
            "system_start": "<|im_start|>system",
            "user_start": "<|im_start|>user",
            "assistant_start": "<|im_start|>assistant",
            "eos": "<|im_end|>"
        },
        "baichuan": {
            "system_start": "<reserved_106>",
            "user_start": "<reserved_107>",
            "assistant_start": "<reserved_108>",
            "eos": "</s>"
        }
    }
    
    def __init__(self, model_type="chatml"):
        self.tokens = self.DEFAULT_SYSTEM_TOKENS.get(model_type, {})
        
    def configure_tokenizer(self, tokenizer):
        """配置分词器的特殊Token"""
        
        special_tokens = {
            "pad_token": "<|pad|>",
            "unk_token": "<|unk|>",
            "bos_token": "<|beginofseries|>",
            "eos_token": self.tokens.get("eos", "</s>")
        }
        
        # 添加额外特殊Token
        additional_special_tokens = [
            self.tokens.get("system_start", "<|system|>"),
            self.tokens.get("user_start", "<|user|>"),
            self.tokens.get("assistant_start", "<|assistant|>")
        ]
        
        special_tokens["additional_special_tokens"] = additional_special_tokens
        
        tokenizer.add_special_tokens(special_tokens)
        
        return tokenizer
        
    def create_mask_pattern(self, text, tokenizer):
        """
        创建训练掩码模式
        
        确定哪些Token在训练时计算loss
        """
        # 解析对话结构
        system_pattern = self.tokens.get("system_start", "")
        user_pattern = self.tokens.get("user_start", "")
        assistant_pattern = self.tokens.get("assistant_start", "")
        eos_pattern = self.tokens.get("eos", "")
        
        tokens = tokenizer.encode(text, add_special_tokens=False)
        labels = [-100] * len(tokens)  # -100表示不计算loss
        
        current_role = None
        
        for i, token_id in enumerate(tokens):
            token = tokenizer.decode([token_id])
            
            # 检测角色切换
            if user_pattern and token == user_pattern.replace("<|im_start|>user", "user"):
                current_role = "user"
            elif assistant_pattern and token == assistant_pattern.replace("<|im_start|>assistant", "assistant"):
                current_role = "assistant"
                
            # 只对assistant回复计算loss
            if current_role == "assistant":
                labels[i] = token_id
            else:
                labels[i] = -100
                
        return labels
```

### 3.3 特殊Token的训练策略

```python
class SpecialTokenTrainingStrategy:
    """特殊Token训练策略"""
    
    def __init__(self):
        self.token_weights = {
            "role_token": 1.0,
            "content_token": 1.0,
            "system_token": 2.0,  # 系统Token重点学习
            "eos_token": 1.5
        }
        
    def create_loss_weights(self, token_ids, token_type_map):
        """
        创建损失权重
        
        某些Token需要更高的学习权重
        """
        weights = []
        
        for token_id in token_ids:
            token_type = token_type_map.get(token_id, "content_token")
            weight = self.token_weights.get(token_type, 1.0)
            weights.append(weight)
            
        return weights
        
    def apply_label_smoothing(self, labels, smoothing_factor=0.1):
        """
        对特殊Token应用标签平滑
        
        减少对精确Token位置的过度自信
        """
        num_classes = self.tokenizer.vocab_size
        
        # 创建平滑后的标签分布
        smoothed = np.full(num_classes, smoothing_factor / (num_classes - 1))
        
        for i, label in enumerate(labels):
            if label >= 0:
                smoothed[label] = 1 - smoothing_factor
            else:
                smoothed = -100  # 保持忽略
                
        return smoothed
```

---

## 四、上下文长度管理

### 4.1 序列长度分析

```python
class SequenceLengthAnalyzer:
    """序列长度分析器"""
    
    def __init__(self, tokenizer):
        self.tokenizer = tokenizer
        
    def analyze_dataset(self, dataset, max_length=4096):
        """
        分析数据集的序列长度分布
        """
        lengths = []
        truncated_count = 0
        
        for item in dataset:
            text = self._extract_text(item)
            tokens = self.tokenizer.encode(text)
            length = len(tokens)
            
            lengths.append(length)
            
            if length > max_length:
                truncated_count += 1
                
        return {
            "stats": {
                "mean": np.mean(lengths),
                "median": np.median(lengths),
                "std": np.std(lengths),
                "min": np.min(lengths),
                "max": np.max(lengths),
                "p25": np.percentile(lengths, 25),
                "p75": np.percentile(lengths, 75),
                "p95": np.percentile(lengths, 95),
                "p99": np.percentile(lengths, 99)
            },
            "truncation_rate": truncated_count / len(dataset),
            "length_distribution": self._create_histogram(lengths)
        }
        
    def estimate_context_efficiency(self, dataset):
        """
        估算上下文利用效率
        """
        total_tokens = 0
        wasted_tokens = 0
        
        for item in dataset:
            text = self._extract_text(item)
            tokens = self.tokenizer.encode(text)
            length = len(tokens)
            
            # 计算浪费的Token（接近最大长度但未达到）
            optimal_length = min(length, 2048)
            total_tokens += optimal_length
            
            # 低于最优长度的数据
            if length < 512:
                wasted_tokens += 512 - length
                
        efficiency = 1 - (wasted_tokens / total_tokens)
        
        return {
            "overall_efficiency": efficiency,
            "recommendation": self._suggest_optimization(efficiency)
        }
```

### 4.2 上下文窗口策略

```python
class ContextWindowManager:
    """上下文窗口管理器"""
    
    def __init__(self, max_length, tokenizer):
        self.max_length = max_length
        self.tokenizer = tokenizer
        
    def truncate_conversation(self, conversation, strategy="rolling"):
        """
        截断对话以适应上下文窗口
        
        Args:
            strategy: 截断策略
                - "rolling": 从最早的消息开始截断
                - "balanced": 平衡保留系统和最近对话
                - "recent": 只保留最近的对话
                - "head_tail": 保留系统提示和最近对话，中间截断
        """
        if strategy == "rolling":
            return self._rolling_truncate(conversation)
        elif strategy == "balanced":
            return self._balanced_truncate(conversation)
        elif strategy == "recent":
            return self._recent_truncate(conversation)
        elif strategy == "head_tail":
            return self._head_tail_truncate(conversation)
            
    def _rolling_truncate(self, conversation):
        """滚动截断：保留最新的消息"""
        result = []
        total_length = 0
        
        # 从后向前添加消息
        for turn in reversed(conversation):
            turn_length = self._estimate_turn_length(turn)
            
            if total_length + turn_length <= self.max_length:
                result.insert(0, turn)
                total_length += turn_length
            else:
                break
                
        return result
        
    def _head_tail_truncate(self, conversation):
        """
        头尾截断：保留系统提示和最近对话
        
        格式：[系统] + [最近N条对话]
        """
        system_msg = None
        other_msgs = []
        
        # 分离系统消息
        for turn in conversation:
            if turn.get("role") == "system":
                system_msg = turn
            else:
                other_msgs.append(turn)
                
        # 编码系统消息
        system_length = self._estimate_turn_length(system_msg) if system_msg else 0
        
        # 从后向前选择其他消息
        budget = self.max_length - system_length
        selected = []
        total = 0
        
        for turn in reversed(other_msgs):
            turn_length = self._estimate_turn_length(turn)
            
            if total + turn_length <= budget:
                selected.insert(0, turn)
                total += turn_length
            else:
                break
                
        # 组合结果
        result = []
        if system_msg:
            result.append(system_msg)
        result.extend(selected)
        
        return result
        
    def _estimate_turn_length(self, turn):
        """估算单条消息的Token长度"""
        if not turn:
            return 0
            
        text = f"{turn['role']}: {turn['content']}"
        return len(self.tokenizer.encode(text))
        
    def pack_multiple_conversations(self, conversations):
        """
        将多个短对话打包成一个序列
        
        使用特殊Token分隔
        """
        packed_tokens = []
        packed_labels = []
        
        separator = self.tokenizer.encode("[SEP]")
        
        for conv in conversations:
            for turn in conv:
                tokens = self.tokenizer.encode(turn["content"])
                packed_tokens.extend(tokens)
                
                # 根据角色决定是否计算loss
                if turn["role"] == "assistant":
                    packed_labels.extend(tokens)
                else:
                    packed_labels.extend([-100] * len(tokens))
                    
            packed_tokens.extend(separator)
            packed_labels.extend([-100] * len(separator))
            
        # 截断到最大长度
        if len(packed_tokens) > self.max_length:
            packed_tokens = packed_tokens[:self.max_length]
            packed_labels = packed_labels[:self.max_length]
            
        return {
            "input_ids": packed_tokens,
            "labels": packed_labels
        }
```

---

## 五、数据集打包格式

### 5.1 HuggingFace数据集格式

```python
class HuggingFaceDatasetCreator:
    """HuggingFace数据集创建器"""
    
    def __init__(self):
        self.dataset_info = {
            "description": "",
            "citation": "",
            "homepage": "",
            "license": ""
        }
        
    def create_dataset(self, data_records, output_dir, dataset_name):
        """
        创建HuggingFace格式数据集
        """
        from datasets import Dataset, DatasetDict, load_dataset, Audio, Image
        import pandas as pd
        
        # 转换为DataFrame
        df = pd.DataFrame(data_records)
        
        # 创建Dataset
        dataset = Dataset.from_pandas(df)
        
        # 添加元数据
        dataset.info.description = self.dataset_info["description"]
        dataset.info.citation = self.dataset_info["citation"]
        dataset.info.homepage = self.dataset_info["homepage"]
        dataset.info.license = self.dataset_info["license"]
        
        # 保存数据集
        dataset.save_to_disk(f"{output_dir}/{dataset_name}")
        
        # 同时保存为Arrow格式
        dataset.to_arrow(f"{output_dir}/{dataset_name}_arrow")
        
        # 保存为JSONL
        dataset.to_json(f"{output_dir}/{dataset_name}.jsonl")
        
        return dataset
        
    def create_splits(self, data_records, train_ratio=0.9, 
                     val_ratio=0.05, test_ratio=0.05):
        """
        创建训练/验证/测试集分割
        """
        from datasets import DatasetDict
        
        import random
        random.shuffle(data_records)
        
        n = len(data_records)
        train_end = int(n * train_ratio)
        val_end = train_end + int(n * val_ratio)
        
        splits = {
            "train": data_records[:train_end],
            "validation": data_records[train_end:val_end],
            "test": data_records[val_end:]
        }
        
        dataset_dict = DatasetDict({
            split: self._create_single_dataset(records)
            for split, records in splits.items()
        })
        
        return dataset_dict
        
    def _create_single_dataset(self, records):
        """创建单个数据集"""
        from datasets import Dataset
        return Dataset.from_list(records)
```

### 5.2 Parquet格式优化

```python
class ParquetDatasetOptimizer:
    """Parquet数据集优化器"""
    
    def __init__(self):
        self.compression = "zstd"
        
    def convert_to_parquet(self, jsonl_path, output_path,
                          chunk_size=10000):
        """
        将JSONL转换为优化的Parquet格式
        """
        import pandas as pd
        import pyarrow as pa
        import pyarrow.parquet as pq
        
        # 分块读取
        chunks = pd.read_json(jsonl_path, lines=True, chunksize=chunk_size)
        
        writer = None
        
        for i, chunk in enumerate(chunks):
            table = pa.Table.from_pandas(chunk)
            
            if writer is None:
                writer = pq.ParquetWriter(
                    output_path,
                    table.schema,
                    compression=self.compression
                )
                
            writer.write_table(table)
            
        writer.close()
        
        return output_path
        
    def optimize_for_training(self, parquet_path, 
                             max_token_length=2048):
        """
        针对训练进行优化
        """
        # 读取并添加索引列
        df = pd.read_parquet(parquet_path)
        
        # 添加token长度列
        df["token_length"] = df["text"].apply(lambda x: len(x.split()))
        
        # 添加质量分数列（如果存在）
        if "quality_score" not in df.columns:
            df["quality_score"] = 1.0
            
        # 按质量分数排序（高质量数据在前）
        df = df.sort_values("quality_score", ascending=False)
        
        # 过滤过长样本
        df = df[df["token_length"] <= max_token_length]
        
        # 保存优化后的数据集
        optimized_path = parquet_path.replace(".parquet", "_optimized.parquet")
        df.to_parquet(optimized_path, compression=self.compression)
        
        return {
            "output_path": optimized_path,
            "num_samples": len(df),
            "avg_quality": df["quality_score"].mean()
        }
```

### 5.3 数据集版本管理

```python
class DatasetVersionManager:
    """数据集版本管理器"""
    
    def __init__(self, base_path):
        self.base_path = base_path
        self.versions_file = f"{base_path}/versions.json"
        
    def create_version(self, dataset_path, version_tag, metadata=None):
        """
        创建数据集版本快照
        """
        import hashlib
        import json
        from datetime import datetime
        
        # 计算数据集指纹
        fingerprint = self._compute_fingerprint(dataset_path)
        
        # 读取当前版本记录
        versions = self._load_versions()
        
        # 创建新版本
        new_version = {
            "version": version_tag,
            "created_at": datetime.now().isoformat(),
            "path": dataset_path,
            "fingerprint": fingerprint,
            "metadata": metadata or {},
            "stats": self._compute_stats(dataset_path)
        }
        
        versions[version_tag] = new_version
        
        # 保存版本记录
        self._save_versions(versions)
        
        return new_version
        
    def _compute_fingerprint(self, path):
        """计算数据集指纹"""
        import os
        
        hash_md5 = hashlib.md5()
        
        for root, dirs, files in os.walk(path):
            for file in sorted(files):
                filepath = os.path.join(root, file)
                with open(filepath, "rb") as f:
                    for chunk in iter(lambda: f.read(4096), b""):
                        hash_md5.update(chunk)
                        
        return hash_md5.hexdigest()
        
    def compare_versions(self, version_a, version_b):
        """比较两个版本的差异"""
        versions = self._load_versions()
        
        if version_a not in versions or version_b not in versions:
            raise ValueError("版本不存在")
            
        v_a = versions[version_a]
        v_b = versions[version_b]
        
        return {
            "added_samples": v_b["stats"]["num_samples"] - v_a["stats"]["num_samples"],
            "changed": v_a["fingerprint"] != v_b["fingerprint"],
            "metadata_changes": self._diff_metadata(
                v_a["metadata"], v_b["metadata"]
            )
        }
```

---

## 六、实战：完整数据处理流程

### 6.1 端到端数据处理脚本

```python
class CompleteDataPipeline:
    """完整数据处理流水线"""
    
    def __init__(self, config):
        self.config = config
        self.tokenizer = self._load_tokenizer()
        
    def process(self, input_path, output_path):
        """
        执行完整的数据处理流程
        """
        # 步骤1：加载原始数据
        raw_data = self._load_raw_data(input_path)
        print(f"加载数据: {len(raw_data)} 条")
        
        # 步骤2：数据清洗
        cleaned_data = self._clean_data(raw_data)
        print(f"清洗后: {len(cleaned_data)} 条")
        
        # 步骤3：数据增强
        if self.config.get("enable_augmentation"):
            augmented_data = self._augment_data(cleaned_data)
            print(f"增强后: {len(augmented_data)} 条")
        else:
            augmented_data = cleaned_data
            
        # 步骤4：格式转换
        formatted_data = self._format_data(augmented_data)
        print(f"格式化完成")
        
        # 步骤5：Tokenization
        tokenized_data = self._tokenize_data(formatted_data)
        print(f"Token化完成")
        
        # 步骤6：序列长度分析
        length_analysis = self._analyze_lengths(tokenized_data)
        print(f"长度分析: 平均 {length_analysis['mean']:.0f} tokens")
        
        # 步骤7：数据打包
        self._pack_dataset(tokenized_data, output_path)
        print(f"保存到: {output_path}")
        
        return {
            "num_samples": len(tokenized_data),
            "length_stats": length_analysis
        }
        
    def _load_raw_data(self, path):
        """加载原始数据"""
        if path.endswith(".jsonl"):
            return self._load_jsonl(path)
        elif path.endswith(".json"):
            return self._load_json(path)
        elif path.endswith(".csv"):
            return self._load_csv(path)
        else:
            raise ValueError(f"不支持的文件格式: {path}")
            
    def _load_jsonl(self, path):
        """加载JSONL文件"""
        data = []
        with open(path, "r", encoding="utf-8") as f:
            for line in f:
                if line.strip():
                    data.append(json.loads(line))
        return data
        
    def _clean_data(self, data):
        """数据清洗"""
        from dataclasses import dataclass, field
        
        @dataclass
        class CleaningRules:
            min_length: int = 10
            max_length: int = 50000
            max_repetition_ratio: float = 0.3
            required_fields: list = field(default_factory=lambda: ["instruction", "response"])
            
        rules = CleaningRules(**self.config.get("cleaning_rules", {}))
        
        cleaned = []
        for item in data:
            # 检查必填字段
            if not all(item.get(f) for f in rules.required_fields):
                continue
                
            # 检查长度
            text = f"{item['instruction']} {item['response']}"
            if not (rules.min_length <= len(text) <= rules.max_length):
                continue
                
            # 检查重复率
            if self._has_high_repetition(text, rules.max_repetition_ratio):
                continue
                
            cleaned.append(item)
            
        return cleaned
        
    def _has_high_repetition(self, text, threshold):
        """检查重复率"""
        words = text.split()
        if len(words) < 10:
            return False
            
        unique_words = len(set(words))
        repetition_ratio = 1 - unique_words / len(words)
        
        return repetition_ratio > threshold
        
    def _format_data(self, data):
        """数据格式转换"""
        formatter = DataFormatConverter()
        target_format = self.config.get("output_format", "chatml")
        
        formatted = []
        for item in data:
            # 转换为对话格式
            conversation = [
                {"role": "user", "content": item["instruction"]},
                {"role": "assistant", "content": item["response"]}
            ]
            
            # 添加系统消息（如果配置）
            if self.config.get("system_prompt"):
                conversation.insert(0, {
                    "role": "system",
                    "content": self.config["system_prompt"]
                })
                
            # 格式转换
            formatted_item = formatter.convert(
                conversation,
                from_format="universal",
                to_format=target_format
            )
            
            formatted.append(formatted_item)
            
        return formatted
        
    def _tokenize_data(self, data):
        """Token化处理"""
        max_length = self.config.get("max_length", 2048)
        
        tokenized = []
        for item in data:
            tokens = self.tokenizer.encode(
                item,
                truncation=True,
                max_length=max_length
            )
            
            tokenized.append({
                "input_ids": tokens,
                "labels": tokens.copy()  # 用于SFT
            })
            
        return tokenized
        
    def _analyze_lengths(self, data):
        """分析序列长度"""
        lengths = [len(item["input_ids"]) for item in data]
        
        return {
            "mean": np.mean(lengths),
            "median": np.median(lengths),
            "std": np.std(lengths),
            "p95": np.percentile(lengths, 95),
            "p99": np.percentile(lengths, 99)
        }
        
    def _pack_dataset(self, data, output_path):
        """打包数据集"""
        from datasets import Dataset
        
        # 转换为HuggingFace格式
        hf_dataset = Dataset.from_list(data)
        
        # 保存
        hf_dataset.save_to_disk(output_path)
        
        # 同时保存为Arrow
        hf_dataset.to_arrow(output_path + "_arrow")
```

### 6.2 配置示例

```yaml
# data_processing_config.yaml
pipeline:
  name: "llm_finetuning_data_processing"
  version: "1.0.0"

input:
  path: "./raw_data/conversations.jsonl"
  format: "jsonl"

cleaning:
  min_length: 20
  max_length: 50000
  max_repetition_ratio: 0.3
  remove_duplicates: true
  toxicity_filter: true
  toxicity_threshold: 0.5

augmentation:
  enable: true
  methods:
    - paraphrase
    - back_translation
  target_variants_per_sample: 2

format:
  output_format: "chatml"
  system_prompt: "你是一个有帮助的AI助手。"
  include_metadata: true

tokenization:
  tokenizer: "Qwen/Qwen2-7B"
  max_length: 4096
  padding: "max_length"
  truncation: true

output:
  path: "./processed_data/train_dataset"
  format: "huggingface"
  save_arrow: true
  save_jsonl: true

stats:
  compute_length_distribution: true
  compute_topic_distribution: true
  sample_quality_check: true
```

---

## 相关文档

- [[数据收集指南]] - 数据采集策略
- [[数据标注最佳实践]] - 数据标注流程
- [[数据清洗技术]] - 数据清洗与过滤
- [[数据增强方法]] - 数据增强技术
- [[开源微调框架]] - LLaMA-Factory等框架
- [[微调技术]] - 各类微调技术详解
