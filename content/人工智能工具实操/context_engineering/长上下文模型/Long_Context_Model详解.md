---
title: Long Context Model详解
date: 2026-04-18
tags:
  - long-context
  - sparse-attention
  - ring-attention
  - RoPE
  - ALiBi
  - million-token
categories:
  - context-engineering
  - long-context-models
---

> [!abstract] 摘要
> 长上下文模型是LLM发展的重要里程碑，本文深入解析Gemini 1.5/2.0的稀疏注意力机制、Claude 3/4的百万token上下文实现、RoPE与ALiBi位置编码原理，以及Ring Attention和LongChat等长上下文技术的核心原理与实现。

## 关键词速览

| 术语 | 英文 | 说明 |
|------|------|------|
| 稀疏注意力 | Sparse Attention | 只计算部分位置对 |
| Ring Attention | Ring Attention | 分布式长上下文 |
| RoPE | Rotary Position Embedding | 旋转位置编码 |
| ALiBi | Attention with Linear Biases | 线性偏置注意力 |
| Flash Attention | Flash Attention | 高效注意力计算 |
| 上下文窗口 | Context Window | 可处理的序列长度 |
| KV Cache | KV Cache | 键值缓存 |
| Memory Efficiency | Memory Efficiency | 内存使用效率 |
| Streaming | Streaming | 流式处理 |
| Context Extrapolation | Context Extrapolation | 上下文外推 |

## 一、长上下文模型概述

### 1.1 上下文长度演进

| 模型 | 发布时间 | 上下文窗口 | 关键技术 |
|------|---------|-----------|----------|
| GPT-2 | 2019 | 1K | 标准Transformer |
| LLaMA 1 | 2023 | 2K-4K | 改进位置编码 |
| LLaMA 2 | 2023 | 4K | 位置编码优化 |
| Claude 2 | 2023 | 100K | 注意力优化 |
| Gemini 1.5 | 2024 | 1M | 稀疏注意力 |
| Claude 3.5 | 2024 | 200K | 改进注意力 |
| Gemini 2.0 | 2024 | 1M+ | 原生稀疏 |
| Claude 4 | 2025 | 1M | 增强稀疏 |

### 1.2 为什么要长上下文

长上下文的价值场景：

1. **长文档处理**：整本书籍、法律文档、代码库
2. **视频理解**：多帧视频的字幕和描述
3. **代理应用**：多步骤任务的全流程追踪
4. **多文档问答**：跨多个文档的关联分析
5. **少样本学习**：大量示例的上下文学习

## 二、稀疏注意力机制

### 2.1 标准注意力的瓶颈

标准自注意力的计算复杂度为 $O(n^2)$：

```python
def standard_attention(Q, K, V):
    """
    标准注意力计算
    Q, K, V: [batch, seq_len, hidden_dim]
    """
    # 计算注意力分数: O(n^2 * d)
    scores = torch.matmul(Q, K.transpose(-2, -1))  # [batch, n, n]
    scores = scores / math.sqrt(Q.size(-1))  # 缩放
    
    # Softmax: O(n^2)
    attn_weights = F.softmax(scores, dim=-1)
    
    # 加权求和: O(n^2 * d)
    output = torch.matmul(attn_weights, V)
    
    return output, attn_weights
```

问题：
- 序列长度翻倍，计算量增加4倍
- 内存占用也增加4倍
- 无法处理超长序列

### 2.2 稀疏注意力模式

```python
class SparseAttention:
    """稀疏注意力实现"""
    
    @staticmethod
    def sliding_window_attention(
        Q, K, V,
        window_size: int = 512
    ):
        """
        滑动窗口注意力
        每个位置只关注窗口内的token
        """
        seq_len = Q.size(1)
        dim = Q.size(-1)
        
        output = torch.zeros_like(Q)
        
        for i in range(seq_len):
            # 窗口范围
            start = max(0, i - window_size // 2)
            end = min(seq_len, i + window_size // 2 + 1)
            
            # 局部注意力计算
            q_i = Q[:, i:i+1, :]  # [batch, 1, dim]
            k_local = K[:, start:end, :]  # [batch, window, dim]
            v_local = V[:, start:end, :]  # [batch, window, dim]
            
            # 计算局部注意力
            scores = torch.matmul(q_i, k_local.transpose(-2, -1)) / math.sqrt(dim)
            attn = F.softmax(scores, dim=-1)
            output[:, i:i+1, :] = torch.matmul(attn, v_local)
        
        return output
    
    @staticmethod
    def local_plus_global_attention(
        Q, K, V,
        window_size: int = 512,
        global_heads: int = 8
    ):
        """
        局部+全局注意力
        部分注意力头处理全序列
        """
        batch, seq_len, num_heads, dim = Q.shape
        
        # 分出头
        Q_local, Q_global = Q.split(num_heads - global_heads, dim=2)
        K_local, K_global = K.split(num_heads - global_heads, dim=2)
        V_local, V_global = V.split(num_heads - global_heads, dim=2)
        
        # 局部注意力
        local_out = SparseAttention.sliding_window_attention(
            Q_local, K_local, V_local, window_size
        )
        
        # 全局注意力
        scores_global = torch.matmul(Q_global, K.transpose(-2, -1)) / math.sqrt(dim)
        attn_global = F.softmax(scores_global, dim=-1)
        global_out = torch.matmul(attn_global, V)
        
        # 合并
        output = torch.cat([local_out, global_out], dim=2)
        
        return output
```

### 2.3 Longformer的稀疏模式

Longformer使用的注意力模式：

```python
class LongformerAttention:
    """
    Longformer注意力模式
    
    - sliding window: 局部上下文
    - global attention: 特殊token（如[CLS]）
    - dilated sliding window: 扩大感受野
    """
    
    def __init__(
        self,
        num_heads: int,
        head_dim: int,
        window_size: int = 512,
        global_tokens: list = None,
        dilation: int = 1
    ):
        self.num_heads = num_heads
        self.head_dim = head_dim
        self.window_size = window_size
        self.global_tokens = global_tokens or []
        self.dilation = dilation
    
    def forward(self, Q, K, V, attention_mask=None):
        batch_size, seq_len, _ = Q.shape
        
        # 构建稀疏注意力掩码
        attn_mask = self._build_attention_mask(seq_len)
        
        # 分块计算
        output = self._chunked_attention(Q, K, V, attn_mask)
        
        return output
    
    def _build_attention_mask(self, seq_len):
        """构建稀疏注意力掩码"""
        mask = torch.zeros(seq_len, seq_len)
        
        for i in range(seq_len):
            # 滑动窗口
            start = max(0, i - self.window_size // 2)
            end = min(seq_len, i + self.window_size // 2 + 1)
            
            # 步长采样（膨胀）
            if self.dilation > 1:
                window_indices = range(start, end, self.dilation)
            else:
                window_indices = range(start, end)
            
            mask[i, list(window_indices)] = 1
            
            # 全局注意力
            if i in self.global_tokens:
                mask[i, :] = 1  # 关注所有位置
        
        return mask.bool()
```

## 三、旋转位置编码（RoPE）

### 3.1 RoPE原理

RoPE通过旋转矩阵编码位置信息：

```python
import torch
import math

class RotaryPositionalEmbedding:
    """
    旋转位置编码 (Rotary Position Embedding)
    
    核心思想：将绝对位置编码为旋转，将相对位置编码为旋转角度差
    """
    
    def __init__(self, dim, max_seq_len=2048, base=10000):
        self.dim = dim
        self.base = base
        self.max_seq_len = max_seq_len
        
        # 预计算频率
        self.inv_freq = 1.0 / (base ** (torch.arange(0, dim, 2).float() / dim))
        
        # 缓存
        self._cache = {}
    
    def get_rotary_matrix(self, seq_len):
        """获取旋转矩阵"""
        if seq_len in self._cache:
            return self._cache[seq_len]
        
        # 位置
        positions = torch.arange(seq_len).float()
        
        # 计算角度
        angles = positions.unsqueeze(1) * self.inv_freq.unsqueeze(0)
        
        # 复数形式
        emb = torch.cat([angles, angles], dim=-1)
        
        # 转换为极坐标
        cos_emb = emb.cos()
        sin_emb = emb.sin()
        
        self._cache[seq_len] = (cos_emb, sin_emb)
        return cos_emb, sin_emb
    
    def rotate_query_key(self, q, k):
        """
        对Q和K应用旋转
        
        这样Q和K的内积就编码了相对位置信息
        """
        seq_len = q.size(1)
        cos_emb, sin_emb = self.get_rotary_matrix(seq_len)
        
        # 分割维度
        q1, q2 = q[..., ::2], q[..., 1::2]
        k1, k2 = k[..., ::2], k[..., 1::2]
        
        # 旋转
        q_rotated = torch.cat([
            q1 * cos_emb - q2 * sin_emb,
            q1 * sin_emb + q2 * cos_emb
        ], dim=-1)
        
        k_rotated = torch.cat([
            k1 * cos_emb - k2 * sin_emb,
            k1 * sin_emb + k2 * cos_emb
        ], dim=-1)
        
        return q_rotated, k_rotated
```

### 3.2 RoPE的数学推导

RoPE的核心性质：

对于位置 $m$ 和 $n$ 的token，其旋转后的内积只与相对位置 $m-n$ 有关：

$$\langle R_{\Theta,m} q_m, R_{\Theta,n} k_n \rangle = \langle q_m, k_n \rangle_{Rotary}$$

其中 $R_{\Theta,m}$ 是旋转矩阵：

$$R_{\Theta,m} = \begin{pmatrix} \cos(m\theta_1) & -\sin(m\theta_1) & 0 & 0 \\ \sin(m\theta_1) & \cos(m\theta_1) & 0 & 0 \\ 0 & 0 & \cos(m\theta_2) & -\sin(m\theta_2) \\ 0 & 0 & \sin(m\theta_2) & \cos(m\theta_2) \end{pmatrix}$$

**优势**：
- 自然编码相对位置
- 支持上下文外推（通过微调或扩展）
- 计算效率高

### 3.3 RoPE上下文外推

```python
class RoPELengthExtrapolation:
    """
    RoPE长度外推
    
    问题：训练时最大序列长度为2048，推理时需要更长
    解决：位置编码的插值和外推
    """
    
    @staticmethod
    def linear_position_interpolation(rope, new_len):
        """
        线性位置插值
        
        将[0, new_len]映射到[0, base_len]
        """
        old_len = rope.max_seq_len
        
        if new_len <= old_len:
            return rope  # 不需要外推
        
        # 缩放因子
        scale = old_len / new_len
        
        # 重新计算频率
        new_inv_freq = 1.0 / (rope.base ** (torch.arange(0, rope.dim, 2).float() / rope.dim))
        
        # 调整频率以实现插值
        rope.inv_freq = new_inv_freq * scale
        rope.max_seq_len = new_len
        rope._cache = {}  # 清除缓存
        
        return rope
    
    @staticmethod
    def NTK_aware_interpolation(rope, new_len, alpha=8):
        """
        NTK-aware 插值
        
        结合插值和外推的优点
        """
        old_len = rope.max_seq_len
        
        if new_len <= old_len:
            return rope
        
        # 计算缩放因子
        scale = (alpha * old_len) / (alpha * old_len + new_len - old_len)
        
        # 部分维度使用外推，部分使用插值
        dim = rope.dim
        extrapolate_dim = int(dim * (1 - scale)) + (dim % 2)
        interpolate_dim = dim - extrapolate_dim
        
        # 创建新的inv_freq
        new_inv_freq = torch.zeros(dim // 2)
        
        # 外推维度：保持原始频率
        extrapolate_freq = 1.0 / (rope.base ** (
            torch.arange(0, extrapolate_dim).float() / dim
        ))
        
        # 插值维度：压缩频率
        interpolate_freq = 1.0 / (
            (rope.base ** scale) ** (torch.arange(extrapolate_dim, dim // 2).float() / dim)
        )
        
        new_inv_freq[:extrapolate_dim // 2] = extrapolate_freq[:extrapolate_dim // 2]
        new_inv_freq[extrapolate_dim // 2:] = interpolate_freq
        
        rope.inv_freq = new_inv_freq
        rope.max_seq_len = new_len
        rope._cache = {}
        
        return rope
```

## 四、ALiBi位置编码

### 4.1 ALiBi原理

ALiBi（Attention with Linear Biases）不添加显式位置编码，而是通过注意力分数的线性偏置实现位置感知：

```python
class ALiBiAttention:
    """
    ALiBi注意力
    
    特点：
    - 不使用位置嵌入
    - 通过注意力分数的线性偏置编码位置
    - 天然支持超长序列
    """
    
    def __init__(self, num_heads, slope_range=(1/8, 1/2)):
        """
        Args:
            num_heads: 注意力头数量
            slope_range: 斜率范围，头之间均匀分布
        """
        self.num_heads = num_heads
        
        # 计算每个头的斜率
        start, end = slope_range
        slopes = torch.linspace(start, end, num_heads)
        self.slopes = slopes
    
    def get_attention_bias(self, seq_len):
        """
        生成注意力偏置矩阵
        
        偏置值 = -|distance| * slope
        """
        # 创建距离矩阵
        positions = torch.arange(seq_len)
        distance_matrix = positions.unsqueeze(0) - positions.unsqueeze(1)
        distance_matrix = distance_matrix.abs()
        
        # 应用斜率
        # [num_heads, seq_len, seq_len]
        bias = -distance_matrix.unsqueeze(0) * self.slopes.view(-1, 1, 1)
        
        # 下三角矩阵掩码（只看左边）
        mask = torch.tril(torch.ones_like(bias) * float('-inf'), diagonal=-1)
        
        return bias + mask
    
    def forward(self, Q, K, V, attention_mask=None):
        """
        带ALiBi的注意力计算
        """
        batch_size, num_heads, seq_len, head_dim = Q.shape
        
        # 标准注意力分数
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(head_dim)
        
        # 获取ALiBi偏置
        alibi_bias = self.get_attention_bias(seq_len).to(Q.device)
        
        # 应用偏置
        scores = scores + alibi_bias.unsqueeze(0)
        
        # 应用注意力掩码
        if attention_mask is not None:
            scores = scores.masked_fill(attention_mask == 0, float('-inf'))
        
        # Softmax和加权求和
        attn_weights = F.softmax(scores, dim=-1)
        output = torch.matmul(attn_weights, V)
        
        return output, attn_weights
```

### 4.2 ALiBi vs RoPE对比

| 特性 | ALiBi | RoPE |
|------|-------|------|
| 位置编码方式 | 无显式位置编码 | 旋转矩阵 |
| 相对位置 | 线性偏置 | 自然编码 |
| 外推能力 | 天然支持 | 需要特殊处理 |
| 计算开销 | 几乎无额外开销 | 矩阵乘法 |
| 硬件友好度 | 较高 | 中等 |
| 理论优雅性 | 简单直接 | 数学优美 |

## 五、Ring Attention

### 5.1 Ring Attention原理

Ring Attention将长序列分布到多个设备上计算：

```
设备0: 块0 ──────► 块1
  │                 │
  ▼                 ▼
设备3: 块3 ◄────── 块2
```

```python
class RingAttention:
    """
    Ring Attention 实现
    
    将序列分块，每个设备处理一块
    通过环形通信传递KV
    """
    
    def __init__(self, num_devices, chunk_size):
        self.num_devices = num_devices
        self.chunk_size = chunk_size
    
    def forward_distributed(self, Q_chunks, K, V, device_rank):
        """
        分布式注意力计算
        
        Args:
            Q_chunks: 当前设备的Q块
            K, V: 完整的K, V矩阵
            device_rank: 当前设备编号
        """
        seq_len = K.size(1)
        num_chunks = seq_len // self.chunk_size
        
        # 初始化输出
        output = torch.zeros_like(Q_chunks)
        
        # 环形通信
        K_recv = K.chunk(num_chunks, dim=1)[device_rank]
        V_recv = V.chunk(num_chunks, dim=1)[device_rank]
        
        for step in range(self.num_devices):
            # 计算当前块的注意力
            current_rank = (device_rank - step) % self.num_devices
            
            # 本地K, V块
            K_local = K.chunk(num_chunks, dim=1)[current_rank]
            V_local = V.chunk(num_chunks, dim=1)[current_rank]
            
            # 计算注意力
            scores = torch.matmul(Q_chunks, K_local.transpose(-2, -1))
            attn = F.softmax(scores, dim=-1)
            output += torch.matmul(attn, V_local)
            
            # 接收下一个设备的KV
            next_rank = (device_rank + 1) % self.num_devices
            K_recv = self._recv_from_device(K_recv, next_rank)
            V_recv = self._recv_from_device(V_recv, next_rank)
        
        return output
    
    def _recv_from_device(self, tensor, source_rank):
        """从其他设备接收数据"""
        # 简化实现，实际需要通信原语
        return tensor
```

### 5.2 Flash Attention与Ring的结合

```python
class FlashRingAttention:
    """
    Flash Attention + Ring Attention
    
    结合两者的优势：
    - Flash Attention: 高效的注意力计算
    - Ring Attention: 分布式处理长序列
    """
    
    @staticmethod
    def blockwise_flash_attention(
        Q, K, V,
        block_size: int = 128,
        device: str = 'cuda'
    ):
        """
        分块Flash Attention
        
        核心思想：分块计算，避免显存O(n^2)
        """
        batch_size, num_heads, seq_len, head_dim = Q.shape
        
        # 计算块数
        num_blocks = (seq_len + block_size - 1) // block_size
        
        # 初始化输出和缩放因子
        output = torch.zeros_like(Q)
        l = torch.zeros(batch_size, num_heads, seq_len, device=device)
        m = torch.full((batch_size, num_heads, seq_len), float('-inf'), device=device)
        
        # 分块处理
        for i in range(num_blocks):
            # 当前块
            j_end = min((i + 1) * block_size, seq_len)
            
            # 计算块内注意力
            Q_i = Q[:, :, i*block_size:j_end, :]
            K_j = K[:, :, i*block_size:j_end, :]
            V_j = V[:, :, i*block_size:j_end, :]
            
            # Flash Attention核心步骤
            output[:, :, i*block_size:j_end, :], l_i, m_i = \
                flash_attn_core(Q_i, K_j, V_j, l[:, :, i*block_size:j_end], m[:, :, i*block_size:j_end])
        
        return output
```

## 六、LongChat实现

### 6.1 LongChat架构

LongChat通过以下技术实现长上下文：

```python
class LongChatConfig:
    """LongChat配置"""
    
    def __init__(
        self,
        max_length: int = 32768,
        original_length: int = 4096,
        rope_type: str = "yarn",  # Yet Another RoPE extENsion
        rope_factor: float = 1.0,
        original_factor: float = 32.0
    ):
        self.max_length = max_length
        self.original_length = original_length
        self.rope_type = rope_type
        self.rope_factor = rope_factor
        self.original_factor = original_factor


class LongChatModel:
    """
    LongChat模型
    
    使用YaRN（Yet Another RoPE extENsion）进行长度外推
    """
    
    def __init__(self, config: LongChatConfig):
        self.config = config
        self.rope = self._build_yarn_rope()
    
    def _build_yarn_rope(self):
        """构建YaRN旋转位置编码"""
        dim = 128  # 假设每个头的维度
        
        # YaRN的修改：位置缩放 + 注意力缩放
        rope = RotaryPositionalEmbedding(dim)
        
        # 位置缩放
        original_len = self.config.original_length
        max_len = self.config.max_length
        
        # 计算缩放因子
        factor = self.config.original_factor
        
        # 调整inv_freq
        new_inv_freq = rope.inv_freq / factor
        
        # 部分外推
        extrapolate_dim = int(dim * (1 - 1/factor))
        new_inv_freq[extrapolate_dim:] *= factor
        
        rope.inv_freq = new_inv_freq
        rope.max_seq_len = max_len
        
        return rope
```

### 6.2 长度外推技术总结

```python
class LengthExtrapolationMethods:
    """长度外推方法总结"""
    
    @staticmethod
    def pos_interpolation(rope, new_len):
        """位置插值"""
        scale = rope.max_seq_len / new_len
        rope.inv_freq = rope.inv_freq * scale
        rope.max_seq_len = new_len
        return rope
    
    @staticmethod
    def huang2023_extrapolation(rope, new_len):
        """Huang 2023 外推方法"""
        # 位置乘以缩放因子
        return rope
    
    @staticmethod
    def yarn_rope(rope, new_len, factor=32, original_len=2048):
        """YaRN方法"""
        # 结合插值和外推
        scale = original_len / new_len
        
        dim = rope.dim
        extrapolate_portion = 1 - scale
        
        extrapolate_dims = int(dim * extrapolate_portion)
        
        # 外推部分保持原样
        # 插值部分使用压缩频率
        rope.inv_freq[:extrapolate_dims] = (
            rope.inv_freq[:extrapolate_dims]
        )
        rope.inv_freq[extrapolate_dims:] = (
            rope.inv_freq[extrapolate_dims:] * scale
        )
        
        rope.max_seq_len = new_len
        return rope
```

## 七、主流模型技术对比

### 7.1 技术选型对比

| 模型 | 位置编码 | 注意力机制 | 关键技术 |
|------|---------|-----------|----------|
| GPT-4 | 自定义 | Sparse | 未公开 |
| Claude 3/4 | ALiBi变体 | 稀疏+全局 | 改进的注意力 |
| Gemini 1.5 | RoPE变体 | 稀疏 | Transformer-XL |
| Gemini 2.0 | RoPE | 原生稀疏 | 更高效 |
| LLaMA 3 | RoPE | Full | 上下文外推 |
| Mistral | RoPE | Sliding Window | 滑动窗口 |
| LongChat | YaRN RoPE | Full | 长度外推 |

### 7.2 上下文长度选择建议

| 应用场景 | 推荐上下文 | 技术要点 |
|---------|-----------|----------|
| 对话助手 | 8K-32K | 摘要+滑动窗口 |
| 文档分析 | 128K-200K | 重排+压缩 |
| 代码分析 | 32K-128K | 语义分块 |
| 多文档问答 | 200K-1M | 分层检索 |
| Agent任务 | 128K-1M | 记忆整合 |

## 八、相关主题

- [[上下文窗口深度解析]]
- [[滑动窗口技术]]
- [[Context Caching详解]]
- [[上下文压缩技术]]
- [[上下文结构化]]

## 九、参考文献

1. Beltagy, I., et al. (2020). Longformer: The Long-Document Transformer. *arXiv:2004.05150*.
2. Child, R., et al. (2019). Generating Long Sequences with Sparse Transformers. *arXiv:1904.10509*.
3. Dao, T., et al. (2022). FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness. *NeurIPS*.
4. Sun, S., et al. (2023). YaRN: Efficient Context Window Extension of LLMs. *arXiv:2309.00071*.
5. Liu, F., et al. (2023). Distributed Attention for Long Context. *ICML*.
