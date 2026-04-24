---
title: Transformer算法完整指南
date: 2026-04-18
tags:
  - Transformer
  - 注意力机制
  - 自注意力
  - BERT
  - GPT
  - 大语言模型
categories:
  - 人工智能学习知识
  - 机器学习
alias: Transformer算法指南/Attention Mechanism
---

# Transformer算法完整指南

> [!note]
> Transformer是当前大语言模型和多模态AI的核心架构，本文深入解析其从注意力机制到完整系统的所有核心组件。

## 关键词速览

| 术语 | 英文 | 核心概念 |
|------|------|----------|
| 自注意力 | Self-Attention | Query-Key-Value注意力机制 |
| 多头注意力 | Multi-Head Attention | 多组注意力并行计算 |
| 位置编码 | Positional Encoding | 注入序列位置信息 |
| 旋转位置编码 | RoPE | 旋转矩阵实现相对位置编码 |
| 层归一化 | Layer Normalization | 稳定训练的关键技术 |
| 前馈网络 | FFN | Feed-Forward Network |
| 残差连接 | Residual Connection | 梯度流动的关键设计 |
| Flash Attention | Flash Attention | 高效注意力计算 |

---

# 一、Transformer架构概述

## 1.1 从RNN到Transformer的演进

传统的[[深度学习|RNN]]存在两个根本性问题：

1. **顺序计算**：无法并行处理序列数据
2. **长距离依赖**：梯度消失/爆炸导致难以学习远距离依赖

Transformer通过**完全并行化**的注意力机制彻底解决了这些问题，成为现代AI的基石。

## 1.2 经典Transformer架构

Transformer采用**编码器-解码器（Encoder-Decoder）**结构：

```
输入 → [编码器 × N] → 编码表示 → [解码器 × N] → 输出
```

- **编码器**：理解输入，提取特征
- **解码器**：生成输出，自回归预测

> [!tip] Encoder-only vs Decoder-only
> - **BERT**：仅编码器，用于理解任务
> - **GPT**：仅解码器，用于生成任务
> - **T5**：完整编码器-解码器，用于seq2seq任务

---

# 二、注意力机制详解

## 2.1 Scaled Dot-Product Attention

注意力机制的核心思想是"Query向Key-Value查询"：

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

### 2.1.1 数学推导

1. **计算相似度**：$QK^T$ 计算Query与Key的点积相似度
2. **缩放**：除以 $\sqrt{d_k}$ 防止点积过大导致梯度消失
3. **Softmax归一化**：得到注意力权重
4. **加权求和**：权重乘以Value得到输出

```python
import numpy as np

def scaled_dot_product_attention(Q, K, V, mask=None):
    """
    Q: (batch, seq_q, d_k)
    K: (batch, seq_k, d_k)
    V: (batch, seq_v, d_v)
    """
    d_k = Q.shape[-1]
    
    # 1. 计算点积相似度
    scores = np.matmul(Q, K.transpose(0, 2, 1))  # (batch, seq_q, seq_k)
    
    # 2. 缩放
    scores = scores / np.sqrt(d_k)
    
    # 3. 应用mask（可选）
    if mask is not None:
        scores = scores + mask
    
    # 4. Softmax归一化
    exp_scores = np.exp(scores - np.max(scores, axis=-1, keepdims=True))
    attn_weights = exp_scores / np.sum(exp_scores, axis=-1, keepdims=True)
    
    # 5. 加权求和
    output = np.matmul(attn_weights, V)
    
    return output, attn_weights
```

### 2.1.2 为什么需要缩放？

> [!warning] 维度缩放的重要性
> 当 $d_k$ 很大时，$QK^T$ 的点积值方差会变得很大，导致Softmax输出趋向于one-hot（梯度极小）。除以 $\sqrt{d_k}$ 可以保持方差稳定。

## 2.2 Multi-Head Attention（MHA）

单头注意力的局限：只能捕获一种类型的相关性。

### 2.2.1 多头注意力机制

$$
\begin{aligned}
\text{MultiHead}(Q, K, V) &= \text{Concat}(\text{head}_1, \ldots, \text{head}_h)W^O \\
\text{head}_i &= \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
\end{aligned}
$$

```python
class MultiHeadAttention:
    def __init__(self, d_model, num_heads):
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        # 投影矩阵
        self.W_q = np.random.randn(d_model, d_model) * 0.02
        self.W_k = np.random.randn(d_model, d_model) * 0.02
        self.W_v = np.random.randn(d_model, d_model) * 0.02
        self.W_o = np.random.randn(d_model, d_model) * 0.02
    
    def split_heads(self, x, batch_size):
        """将最后一个维度分割成num_heads个小维度"""
        x = x.reshape(batch_size, -1, self.num_heads, self.d_k)
        return x.transpose(0, 2, 1, 3)  # (batch, heads, seq, d_k)
    
    def forward(self, Q, K, V, mask=None):
        batch_size = Q.shape[0]
        
        # 线性投影
        Q = np.dot(Q, self.W_q)  # (batch, seq, d_model)
        K = np.dot(K, self.W_k)
        V = np.dot(V, self.W_v)
        
        # 分头
        Q = self.split_heads(Q, batch_size)
        K = self.split_heads(K, batch_size)
        V = self.split_heads(V, batch_size)
        
        # 注意力计算
        attn_output, _ = scaled_dot_product_attention(Q, K, V, mask)
        
        # 合并多头
        attn_output = attn_output.transpose(0, 2, 1, 3).reshape(batch_size, -1, self.d_model)
        
        # 最终投影
        output = np.dot(attn_output, self.W_o)
        
        return output
```

### 2.2.2 多头注意力的优势

| 优势 | 说明 |
|------|------|
| 捕获多维关系 | 不同头学习不同类型的依赖（如语法、语义、位置） |
| 增强表达能力 | 组合多个子空间的表示 |
| 提升鲁棒性 | 某个头失效不影响整体 |

> [!info] 注意力头设计
> - 实践中8-16个头是常见配置
> - 可以使用注意力模式可视化分析不同头的作用
> - 某些头可能学到"停止词"或"句法依赖"等特定模式

---

# 三、位置编码

## 3.1 为什么需要位置编码？

Transformer的注意力机制是**置换不变**的——对输入序列的任意排列，输出相同。位置编码为模型注入序列顺序信息。

## 3.2 绝对位置编码

### 3.2.1 正弦/余弦位置编码

原始Transformer使用的方案：

$$
\begin{aligned}
PE_{(pos, 2i)} &= \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right) \\
PE_{(pos, 2i+1)} &= \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)
\end{aligned}
$$

```python
def positional_encoding(seq_len, d_model):
    """生成正弦/余弦位置编码"""
    PE = np.zeros((seq_len, d_model))
    
    position = np.arange(seq_len)[:, np.newaxis]
    div_term = np.exp(np.arange(0, d_model, 2) * (-np.log(10000.0) / d_model))
    
    PE[:, 0::2] = np.sin(position * div_term)
    PE[:, 1::2] = np.cos(position * div_term)
    
    return PE
```

> [!note] 正弦位置编码的特性
> - 不同位置有不同的编码
> - 相邻位置的编码有规律性差异
> - 可以表示相对位置（通过线性组合）

### 3.2.2 可学习位置编码

现代模型常使用可学习的位置嵌入：

```python
class LearnablePositionalEncoding:
    def __init__(self, max_len, d_model):
        self.pe = np.random.randn(max_len, d_model) * 0.02
    
    def forward(self, x):
        return x + self.pe[:x.shape[1]]
```

## 3.3 旋转位置编码（RoPE）

RoPE是目前LLM（如LLaMA、GLM）广泛使用的位置编码方案。

### 3.3.1 核心思想

RoPE通过**旋转矩阵**实现位置编码，将绝对位置信息融入Query和Key：

对于2维子空间，RoPE定义为旋转矩阵：
$$
R_{\theta,m} = \begin{bmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{bmatrix}
$$

### 3.3.2 实现

```python
def precompute_freqs_cis(dim, seq_len, theta=10000.0):
    """预计算频率用于RoPE"""
    freqs = 1.0 / (theta ** (np.arange(0, dim, 2) / dim))
    t = np.arange(seq_len)
    freqs = np.outer(t, freqs)
    freqs_cis = np.exp(1j * freqs)
    return freqs_cis

def apply_rotary_pos_emb(q, k, freqs_cis):
    """应用旋转位置编码"""
    q_complex = q.astype(np.complex64)
    k_complex = k.astype(np.complex64)
    
    # 复数乘法实现旋转
    q_rotated = np.stack([
        q_complex[..., 0::2] * freqs_cis.real - q_complex[..., 1::2] * freqs_cis.imag,
        q_complex[..., 0::2] * freqs_cis.imag + q_complex[..., 1::2] * freqs_cis.real
    ], axis=-1).reshape(q.shape)
    
    k_rotated = np.stack([
        k_complex[..., 0::2] * freqs_cis.real - k_complex[..., 1::2] * freqs_cis.imag,
        k_complex[..., 0::2] * freqs_cis.imag + k_complex[..., 1::2] * freqs_cis.real
    ], axis=-1).reshape(k.shape)
    
    return q_rotated.real, k_rotated.real
```

### 3.3.3 RoPE的优势

- **解耦位置与语义**：不干扰token的语义表示
- **支持长上下文**：无需显式存储位置编码
- **可扩展序列长度**：通过插值技术（如NTK-aware scaling）

## 3.4 ALiBi（Attention with Linear Biases）

ALiBi通过在注意力分数上添加线性偏置来编码位置信息，无需位置嵌入。

```python
def alibi_bias(seq_len, num_heads):
    """生成ALiBi偏置矩阵"""
    # 创建从1开始的距离矩阵
    distances = np.arange(1, seq_len + 1)
    distances = distances[np.newaxis, :] - distances[:, np.newaxis]
    distances = distances // 2  # 整数除法
    
    # 基础斜率
    slopes = 2 ** (-8 / num_heads * np.arange(1, num_heads + 1))
    
    # 计算偏置
    bias = -slopes[:, np.newaxis, np.newaxis] * distances[np.newaxis, :, :]
    
    return bias
```

---

# 四、层归一化（Layer Normalization）

## 4.1 归一化方法对比

| 归一化 | 计算维度 | 应用场景 |
|--------|----------|----------|
| BatchNorm | batch维度 | CV，稳定但需大batch |
| LayerNorm | 特征维度 | NLP，序列模型 |
| InstanceNorm | 单样本特征 | 风格迁移 |
| GroupNorm | 特征分组 | 小batch的CV任务 |

## 4.2 Layer Normalization公式

$$
\mu = \frac{1}{H}\sum_{i=1}^{H} x_i, \quad \sigma = \sqrt{\frac{1}{H}\sum_{i=1}^{H}(x_i - \mu)^2}
$$

$$
\hat{x}_i = \frac{x_i - \mu}{\sigma + \epsilon}
$$

$$
y_i = \gamma \hat{x}_i + \beta
$$

```python
class LayerNorm:
    def __init__(self, d_model, eps=1e-6):
        self.gamma = np.ones(d_model)
        self.beta = np.zeros(d_model)
        self.eps = eps
    
    def forward(self, x):
        # x shape: (batch, seq_len, d_model)
        mean = np.mean(x, axis=-1, keepdims=True)
        var = np.var(x, axis=-1, keepdims=True)
        
        x_norm = (x - mean) / np.sqrt(var + self.eps)
        return self.gamma * x_norm + self.beta
```

## 4.3 Pre-LN vs Post-LN

> [!tip] Pre-LN vs Post-LN
> - **Post-LN**：LN在残差连接之后（原始Transformer）
> - **Pre-LN**：LN在注意力/FFN之前（现代LLM常用）
> - Pre-LN训练更稳定，梯度分布更均匀

---

# 五、编码器与解码器架构

## 5.1 编码器层

```python
class EncoderLayer:
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.norm1 = LayerNorm(d_model)
        self.norm2 = LayerNorm(d_model)
        
        # Feed-Forward Network
        self.ffn = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Linear(d_ff, d_model)
        )
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, mask=None):
        # Pre-LN结构
        x_norm = self.norm1(x)
        attn_output = self.self_attn(x_norm, x_norm, x_norm, mask)
        x = x + self.dropout(attn_output)
        
        x_norm = self.norm2(x)
        ffn_output = self.ffn(x_norm)
        x = x + self.dropout(ffn_output)
        
        return x
```

## 5.2 解码器层

```python
class DecoderLayer:
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.cross_attn = MultiHeadAttention(d_model, num_heads)
        self.norm1 = LayerNorm(d_model)
        self.norm2 = LayerNorm(d_model)
        self.norm3 = LayerNorm(d_model)
        
        self.ffn = nn.Sequential(
            nn.Linear(d_model, d_ff),
            nn.GELU(),
            nn.Linear(d_ff, d_model)
        )
    
    def forward(self, x, enc_output, src_mask=None, tgt_mask=None):
        # 自注意力（masked）
        x_norm = self.norm1(x)
        self_attn_output = self.self_attn(x_norm, x_norm, x_norm, tgt_mask)
        x = x + self_attn_output
        
        # 交叉注意力（看向编码器）
        x_norm = self.norm2(x)
        cross_attn_output = self.cross_attn(x_norm, enc_output, enc_output, src_mask)
        x = x + cross_attn_output
        
        # FFN
        x_norm = self.norm3(x)
        ffn_output = self.ffn(x_norm)
        x = x + ffn_output
        
        return x
```

---

# 六、经典模型解析

## 6.1 BERT（Bidirectional Encoder Representations from Transformers）

### 6.1.1 核心创新

- **双向上下文**：同时考虑左右上下文
- **MLM预训练**：Masked Language Modeling
- **NSP预训练**：Next Sentence Prediction

### 6.1.2 预训练任务

```python
# Masked Language Modeling
def mlm_masking(tokens, mask_prob=0.15):
    """BERT风格的MLM掩码"""
    labels = tokens.copy()
    
    mask_indices = np.random.random(len(tokens)) < mask_prob
    
    # 80%替换为[MASK]
    tokens[mask_indices & (np.random.random(len(tokens)) < 0.8)] = MASK_TOKEN
    
    # 10%替换为随机token
    tokens[mask_indices & (np.random.random(len(tokens)) < 0.5)] = np.random.randint(VOCAB_SIZE)
    
    # 10%保持不变
    
    return tokens, labels
```

## 6.2 GPT系列（Generative Pre-trained Transformer）

### 6.2.1 架构特点

- **仅解码器**：单向自回归模型
- **下一个Token预测**：最大化似然
- **零样本能力**：涌现于大规模预训练

### 6.2.2 GPT-3关键参数

| 模型 | 参数量 | 上下文长度 |
|------|--------|------------|
| GPT-3 | 175B | 2048 |
| GPT-3.5 | - | 16K |
| GPT-4 | - | 128K |
| GPT-4o | - | 128K |

## 6.3 T5（Text-to-Text Transfer Transformer）

### 6.3.1 统一框架

T5将所有NLP任务统一为text-to-text格式：

| 任务 | 输入 | 输出 |
|------|------|------|
| 翻译 | translate English to German: Hello | Hallo |
| 摘要 | summarize: [长文本] | [摘要] |
| 问答 | question: ... context: ... | answer |

---

# 七、Flash Attention

## 7.1 背景：注意力计算的痛点

标准注意力计算需要 $O(N^2)$ 的显存存储注意力矩阵，对于长序列（64K+）几乎不可行。

## 7.2 核心思想：分块计算

Flash Attention通过**分块（Tile）**和**重计算**实现 $O(N)$ 显存：

1. **分块**：将Q、K、V分成小块处理
2. **在线计算**：流式更新注意力统计量
3. **重计算**：反向传播时重新计算注意力而非存储

## 7.3 实现原理

```python
def flash_attention(Q, K, V, block_size=128):
    """
    Flash Attention的简化实现
    核心思想：分块计算 + 在线更新
    """
    batch_size, seq_len, d_k = Q.shape
    d_k = d_k  # 避免与block_size混淆
    
    # 初始化
    m = np.full(batch_size, -np.inf)  # 行最大值
    l = np.zeros((batch_size, seq_len))  # 行计数
    P = np.zeros((batch_size, seq_len, seq_len))  # 最终注意力矩阵（不显式存储）
    
    O = np.zeros((batch_size, seq_len, d_k))  # 输出
    
    # 分块遍历
    for j in range(0, seq_len, block_size):
        # 计算块内的注意力
        K_block = K[:, j:j+block_size]
        V_block = V[:, j:j+block_size]
        
        # 计算S = QK^T
        S_block = np.einsum('bsd,btd->bst', Q, K_block)
        
        # 安全的指数函数
        m_block = np.max(S_block, axis=-1, keepdims=True)
        S_block_minus_m = np.exp(S_block - m_block)
        
        # 更新统计量
        O = np.einsum('bsd,bst,btf->bdf', O, S_block_minus_m / l[:, :, np.newaxis], V_block)
        
        # 更新最大值和计数
        m_new = np.maximum.outer(m, m_block.squeeze(-1))
        l = l * np.exp(m - m_new) + np.sum(S_block_minus_m, axis=-1)
        m = m_new
    
    # 最终归一化
    O = O / l[:, :, np.newaxis]
    
    return O
```

> [!warning] 实际实现注意事项
> 真正的Flash Attention使用CUDA kernel实现，需要：
> - Tiling配置（BMM, BMM1等）
> - SRAM/DRAM数据移动优化
> - 反向传播的重计算

## 7.4 Flash Attention的变体

| 变体 | 特点 | 改进 |
|------|------|------|
| Flash Attention-2 | 更好的并行和循环优化 | 2-4x加速 |
| Flash Attention-3 | FP8量化支持 | 更高效 |
| Flash Decoding | 解码阶段优化 | 生成加速 |
| Paged Attention | KV Cache分页管理 | 内存效率 |

---

# 八、Transformer变体

## 8.1 高效Transformer

| 架构 | 复杂度 | 核心思想 |
|------|--------|----------|
| Linear Attention | $O(N)$ | 核函数近似 |
| ReZero | - | 可学习的残差权重 |
| RT-DETR | - | Real-Time DETR |

## 8.2 混合架构

| 架构 | 组合方式 |
|------|----------|
| Hyena | 注意力 + MLP |
| Mamba | SSM + 注意力 |
| RWKV | Linear Attention + RNN |

---

# 九、相关文档

- [[深度学习]] - 深度学习完整指南
- [[机器学习]] - 机器学习基础
- [[万字长文-走进因果推断]] - 因果推断与Transformer

---

# 参考资料

- Vaswani, A., et al. (2017). Attention is All You Need. *NeurIPS*.
- Devlin, J., et al. (2018). BERT: Pre-training of Deep Bidirectional Transformers. *NAACL*.
- Radford, A., et al. (2019). Language Models are Unsupervised Multitask Learners. *OpenAI Technical Report*.
- Dao, T., et al. (2022). FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness. *NeurIPS*.
- Su, J., et al. (2022). RoFormer: Enhanced Transformer with Rotary Position Embedding. *arXiv*.
- Press, O., et al. (2021). ALiBi: Train Short, Test Long: Attention with Linear Biases Enable Input Length Extrapolation. *ICLR*.
