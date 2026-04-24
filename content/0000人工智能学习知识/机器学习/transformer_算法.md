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
> Transformer是当前大语言模型和多模态AI的核心架构，本文深入解析其从注意力机制到完整系统的所有核心组件。学完本文后，你应该能够：从零开始用PyTorch实现一个简易Transformer，理解GPT/BERT等大模型的训练原理，能够在实际项目中应用和微调Transformer。

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

# 一、从人类注意力理解Transformer

## 1.1 注意力机制的灵魂拷问

在深入技术细节之前，让我们先回答一个根本问题：为什么叫"注意力"？它到底在"注意"什么？

想象你走进一家咖啡馆，环顾四周。你的眼睛会同时接收整个场景的信息——吧台、桌椅、其他顾客、窗外风景——但你的大脑并不会平等处理所有信息。如果你在等人，你会不自觉地关注门口；如果肚子饿了，你会更关注菜单或食物；如果你在偷听邻桌的对话，你会把注意力集中在他们身上。

这个过程有三个关键要素：

第一，你有一个**目标**（query）："我想找到我等的那个人"。

第二，周围的一切都是**候选信息**（keys）：门口的人、吧台的服务员、窗边看报纸的人。

第三，你能获取的**具体内容**（values）：每个人的长相、穿着、正在做什么。

你的大脑会计算目标与每个候选的"相关度"，然后把注意力集中到最匹配的那个人身上。这就是注意力机制的核心思想——用Query去Query所有Key，找到最相关的Value。

## 1.2 从RNN到Transformer的演进

理解了注意力之后，我们来看看为什么Transformer能取代RNN成为NLP的主流。

### RNN为什么会被淘汰？

传统的RNN（循环神经网络）处理序列数据的方式有点像读书：逐字逐句地读，读完当前字才能读下一个。这导致两个致命问题：

第一个问题是**无法并行**。如果要计算第100个词的隐藏状态，必须先算完前99个。这意味着GPU强大的并行计算能力被浪费了，训练速度极慢。

第二个问题是**长距离依赖困难**。假设你在读一篇侦探小说，看到最后揭示凶手是谁时，模型需要"记住"开头埋下的线索。但在RNN中，信息经过多次传递会逐渐稀释或爆炸（梯度消失/爆炸），早期的重要信息往往被后来的信息覆盖。

举个具体的例子：

```
输入：我来自北京，在上海工作了五年，今天终于回到了_____
```

要预测"北京"，模型需要跨越很长的距离找到"北京"这个信息。RNN在处理这个长距离依赖时往往表现很差。

### Transformer的破局之道

Transformer用注意力机制彻底解决了这两个问题：

并行化：所有位置的隐藏状态可以同时计算，因为注意力机制本质上就是矩阵运算。128个词的句子，处理第1个词和第128个词的计算量是一样的。

长距离依赖：Query和Key的点积可以一步到位建立任意两个位置之间的关联，路径长度为1，不再有信息衰减的问题。

这就是为什么2017年Google发表《Attention is All You Need》后，整个NLP领域在几年内完成了从RNN到Transformer的全面迁移。

---

# 二、注意力机制详解

## 2.1 Scaled Dot-Product Attention

现在我们深入技术细节。注意力机制的核心公式看起来很简单：

$$
\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V
$$

但这个公式背后有深刻的直觉。

### 2.1.1 直觉解释

**第一步：Query和Key的匹配。** 假设Query是"今天天气怎么样"，它会被转换成一个向量表示。Key就是仓库里所有可能问题的"标签"。通过点积 $QK^T$，我们计算Query与每个Key的相似度。相似度越高，说明这个问题越匹配。

**第二步：缩放防止梯度消失。** 当维度 $d_k$ 很大时，点积的方差会变得很大。比如两个均值为0、方差为1的向量，点积的方差大约是 $d_k$。这会导致softmax的输出趋向于one-hot（一个位置接近1，其他接近0），梯度几乎为0，模型难以学习。除以 $\sqrt{d_k}$ 可以保持方差稳定。

**第三步：加权求和。** softmax输出的权重表示对各个Value的"关注程度"。把这些权重乘以对应的Value加权求和，就得到了最终的输出——一个融合了所有相关信息的新表示。

### 2.1.2 代码实现

```python
import numpy as np

def scaled_dot_product_attention(Q, K, V, mask=None):
    """
    Scaled Dot-Product Attention 实现
    
    参数：
        Q: (batch, seq_q, d_k) - 查询向量
        K: (batch, seq_k, d_k) - 键向量
        V: (batch, seq_v, d_v) - 值向量
        mask: 可选的mask矩阵，用于遮挡未来位置
    
    返回：
        output: 注意力加权后的输出
        attn_weights: 注意力权重矩阵
    """
    d_k = Q.shape[-1]
    
    # 第一步：计算Query和Key的点积，得到相似度矩阵
    # scores[i,j] 表示第i个query对第j个key的注意力分数
    scores = np.matmul(Q, K.transpose(0, 2, 1))  # (batch, seq_q, seq_k)
    
    # 第二步：缩放，防止softmax梯度消失
    # 为什么用 d_k 的平方根？因为点积的方差是 d_k
    scores = scores / np.sqrt(d_k)
    
    # 第三步：应用mask（用于解码器中遮挡未来token）
    if mask is not None:
        scores = scores + mask
    
    # 第四步：softmax归一化，得到注意力权重
    # 减去最大值是为了数值稳定性，防止指数爆炸
    exp_scores = np.exp(scores - np.max(scores, axis=-1, keepdims=True))
    attn_weights = exp_scores / np.sum(exp_scores, axis=-1, keepdims=True)
    
    # 第五步：注意力权重乘以Value，加权求和
    output = np.matmul(attn_weights, V)
    
    return output, attn_weights


# 一个具体的例子
if __name__ == "__main__":
    # 假设batch=1, seq_len=3, d_k=4
    Q = np.random.randn(1, 3, 4)
    K = np.random.randn(1, 3, 4)
    V = np.random.randn(1, 3, 4)
    
    output, weights = scaled_dot_product_attention(Q, K, V)
    
    print(f"Q shape: {Q.shape}")
    print(f"注意力权重 shape: {weights.shape}")
    print(f"注意力权重的每一行加起来应该等于1: {np.sum(weights, axis=-1)}")
    print(f"\n示例：第一个query对所有位置的注意力权重: {weights[0, 0]}")
```

### 2.1.3 实际例子：翻译任务中的注意力

假设我们要翻译一句英文到中文：

```
输入: "The cat sat on the mat"
输出: "猫坐在垫子上"
```

当解码器生成"坐"这个字时，它的Query可能是"坐"。通过注意力机制，它会Query编码器中所有英文单词的Key。

解码器发现：
- "sat"（坐）对应分数很高 → 这个词最相关
- "cat"（猫）对应分数中等 → 需要关联主语
- "on"（在...上）对应分数中等 → 提示了介词关系
- 其他词分数较低 → 关联度低

这就是注意力机制可视化中经常看到的"热力图"——它告诉我们模型在做决策时"看"了输入的哪些部分。

## 2.2 Multi-Head Attention（多头注意力）

### 2.2.1 为什么需要多头？

想象你要理解一句话"她看到猫坐在垫子上，然后笑了"。

这个句子涉及多种关系：
- **指代消解**：她是谁？需要关联"她"和句子中其他人物
- **语法结构**：谁做了什么动作？"猫"是"坐"的主语
- **空间关系**：什么东西在什么上面？"猫"在"垫子"上
- **因果关系**：为什么笑？因为看到了有趣的事情

单头注意力只能捕捉一种类型的关系。多头注意力就像是有多双眼睛，每双眼睛关注不同类型的关联。把多个头的输出拼接起来，模型就能同时捕捉各种复杂的关系。

### 2.2.2 数学公式

$$
\begin{aligned}
\text{MultiHead}(Q, K, V) &= \text{Concat}(\text{head}_1, \ldots, \text{head}_h)W^O \\
\text{head}_i &= \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
\end{aligned}
$$

每个头有自己独立的 $W_Q, W_K, W_V$ 投影矩阵。计算完所有头的注意力后，把结果拼接起来，再用一个 $W_O$ 投影矩阵融合。

### 2.2.3 PyTorch实现

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import math

class MultiHeadAttention(nn.Module):
    """
    多头注意力机制的完整实现
    
    和NumPy版本相比，这里展示的是工业级实现：
    - 支持dropout正则化
    - 完善的mask支持
    - 残差连接和层归一化的接口预留
    """
    
    def __init__(self, d_model, num_heads, dropout=0.1):
        super().__init__()
        assert d_model % num_heads == 0, "d_model 必须能被 num_heads 整除"
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads  # 每个头的维度
        
        # 四个投影矩阵：用nn.Linear比手动定义权重更规范
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, query, key, value, mask=None):
        """
        前向传播
        
        参数：
            query: (batch, seq_len, d_model)
            key, value: 通常和query相同（自注意力）或来自编码器（交叉注意力）
            mask: 用于遮挡的mask矩阵
        
        返回：
            output: 注意力输出
            attention_weights: 注意力权重（用于可视化）
        """
        batch_size = query.size(0)
        
        # 1. 线性投影 + 分头
        # 原始实现是先投影再分头，现代实现通常一起做（效率更高）
        Q = self.W_q(query).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(key).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(value).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        
        # 2. 计算注意力
        # 注意这里直接用 PyTorch 的 F.scaled_dot_product_attention
        # 这是官方优化过的实现，等价于我们手动写的版本
        if mask is not None:
            # 处理4D mask的情况（head维度扩展）
            if len(mask.shape) == 3:
                mask = mask.unsqueeze(1)  # (batch, 1, seq_q, seq_k)
            attn_output = F.scaled_dot_product_attention(Q, K, V, attn_mask=mask, dropout=self.dropout if self.training else 0)
        else:
            attn_output = F.scaled_dot_product_attention(Q, K, V, dropout=self.dropout if self.training else 0)
        
        # 保存注意力权重（用于分析或可视化）
        attention_weights = attn_output.transpose(1, 2)  # 便于查看
        
        # 3. 合并多头：(batch, num_heads, seq, d_k) -> (batch, seq, d_model)
        attn_output = attn_output.transpose(1, 2).contiguous().view(batch_size, -1, self.d_model)
        
        # 4. 最终投影
        output = self.W_o(attn_output)
        
        return output, attention_weights


# 测试代码
if __name__ == "__main__":
    # 创建一个多头注意力层
    mha = MultiHeadAttention(d_model=512, num_heads=8)
    
    # 模拟输入
    batch_size = 2
    seq_len = 10
    d_model = 512
    
    x = torch.randn(batch_size, seq_len, d_model)
    
    # 前向传播
    output, weights = mha(x, x, x)  # 自注意力：Q=K=V=x
    
    print(f"输入形状: {x.shape}")
    print(f"输出形状: {output.shape}")
    print(f"注意力权重形状: {weights.shape}")  # (batch, seq, seq)
    print(f"注意力权重对角线之和（自身注意程度）: {torch.diagonal(weights, dim1=2, dim2=3).mean()}")
```

### 2.2.4 不同头的可视化分析

训练好的Transformer中，不同的注意力头往往学会捕捉不同类型的依赖：

**头1（位置敏感型）**：可能关注相邻词，如形容词修饰名词

**头2（句法依赖型）**：可能关注主语-谓语关系

**头3（指代消解型）**：可能关注代词和其指代的名词

**头4（语义关联型）**：可能关注同义词或语义相关的词

这就是为什么多头注意力如此强大——它允许模型自动学习多种类型的关联，而不需要人工设计特征。

---

# 三、位置编码：让序列有顺序

## 3.1 为什么必须位置编码

Transformer的核心是注意力机制，而注意力本质上是"无序"的。什么意思？

假设输入是三个词："我爱你"。如果把顺序打乱变成"你爱我"，对注意力机制来说是一样的——它只关心Query和Key的匹配，不关心位置。

但语言是有顺序的！"我爱你"和"爱你我"表达的意思完全不同。所以我们必须把位置信息"注入"给模型。

## 3.2 正弦/余弦位置编码

原始Transformer使用了一种巧妙的固定位置编码：

$$
\begin{aligned}
PE_{(pos, 2i)} &= \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right) \\
PE_{(pos, 2i+1)} &= \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)
\end{aligned}
$$

### 3.2.1 直观理解

这个公式看起来复杂，但直觉很简单：

- **奇偶位置交替**：偶数位置用sin，奇数位置用cos
- **频率递减**：随着维度i增加，分母变大，周期变长，低维捕捉高频位置变化，高维捕捉低频位置变化
- **每个位置唯一**：不同位置的编码向量不同
- **相对位置可线性表示**：$PE_{pos+k}$ 可以表示为 $PE_{pos}$ 的线性组合

### 3.2.2 PyTorch实现

```python
import torch
import torch.nn as nn
import math

class PositionalEncoding(nn.Module):
    """
    原始Transformer的正弦/余弦位置编码
    
    优点：
    - 可以处理任意长度的序列（理论上）
    - 不需要学习，参数少
    - 有一定外推能力
    
    缺点：
    - 被发现不是最优方案
    - 外推到训练长度外的效果下降
    """
    
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)
        
        # 创建位置编码矩阵
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        
        # 偶数维度用sin
        pe[:, 0::2] = torch.sin(position * div_term)
        # 奇数维度用cos
        pe[:, 1::2] = torch.cos(position * div_term)
        
        # 添加batch维度：(1, max_len, d_model)
        pe = pe.unsqueeze(0)
        
        # 注册为buffer：不是模型参数，但会随模型保存/加载
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        """
        x: (batch, seq_len, d_model)
        """
        # 把位置编码加到输入上
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)


# 验证：不同位置编码的内积
def analyze_positional_encoding():
    """分析位置编码的特性"""
    d_model = 64
    max_len = 100
    
    pe = torch.zeros(max_len, d_model)
    position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
    div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
    pe[:, 0::2] = torch.sin(position * div_term)
    pe[:, 1::2] = torch.cos(position * div_term)
    
    # 计算位置i和位置j的编码的余弦相似度
    similarities = torch.nn.functional.cosine_similarity(pe.unsqueeze(1), pe.unsqueeze(0), dim=2)
    
    print("位置编码相似度矩阵（相邻位置相似度高，随距离增加递减）:")
    print(f"位置0和位置1的相似度: {similarities[0, 1]:.3f}")
    print(f"位置0和位置10的相似度: {similarities[0, 10]:.3f}")
    print(f"位置0和位置50的相似度: {similarities[0, 50]:.3f}")


if __name__ == "__main__":
    analyze_positional_encoding()
```

## 3.3 可学习的位置编码

现代模型（尤其是BERT之后）更常用可学习的位置编码：

```python
class LearnedPositionalEncoding(nn.Module):
    """
    可学习的位置编码
    
    直接把位置编码当作可学习的参数：
    - 初始化为全0或小的随机值
    - 通过反向传播学习
    - 通常比正弦编码效果更好
    
    缺点：
    - 需要事先指定最大长度
    - 参数更多
    """
    
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        self.pe = nn.Embedding(max_len, d_model)
    
    def forward(self, x):
        """
        x: (batch, seq_len, d_model)
        """
        batch_size, seq_len, _ = x.shape
        # 生成位置索引 [0, 1, 2, ..., seq_len-1]
        position = torch.arange(seq_len, device=x.device).unsqueeze(0).expand(batch_size, seq_len)
        # 获取位置嵌入并加到输入上
        x = x + self.pe(position)
        return x
```

## 3.4 旋转位置编码（RoPE）

RoPE是LLaMA、GLM等大语言模型广泛使用的位置编码方案，核心思想是用旋转矩阵编码位置信息。

### 3.4.1 为什么RoPE更优秀？

传统的加性位置编码（正弦/余弦或可学习）有一个问题：它改变了Query和Key的语义方向。RoPE通过旋转Query和Key向量来编码位置，保持了语义完整性。

从数学上看，如果两个token的相对位置是k，它们的旋转角度差也是k，所以RoPE天然支持相对位置注意力。

### 3.4.2 RoPE实现

```python
import torch
import math

def precompute_freqs_cis(dim, seq_len, theta=10000.0):
    """
    预计算复数频率，用于RoPE
    
    参数：
        dim: 嵌入维度（必须是偶数）
        seq_len: 序列长度
        theta: 基础频率参数
    
    返回：
        freqs_cis: 复数形式的频率 (seq_len, dim//2)
    """
    freqs = 1.0 / (theta ** (torch.arange(0, dim, 2, device='cuda' if torch.cuda.is_available() else 'cpu').float() / dim))
    t = torch.arange(seq_len, device=freqs.device)
    freqs = torch.outer(t, freqs)
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)  # 转换为复数：cos + i*sin
    return freqs_cis


def apply_rotary_pos_emb(q, k, freqs_cis):
    """
    应用旋转位置编码到Query和Key
    
    参数：
        q: Query张量 (batch, num_heads, seq_len, dim)
        k: Key张量
        freqs_cis: 预计算的复数频率 (seq_len, dim//2)
    
    返回：
        旋转后的q和k
    """
    def rotate_half(x):
        """将向量后半部分取负，实现旋转"""
        x1, x2 = x[..., : x.shape[-1] // 2], x[..., x.shape[-1] // 2 :]
        return torch.cat([-x2, x1], dim=-1)
    
    # 复数乘法：q * freq
    # 复数乘法规则：(a+bi)*(c+di) = (ac-bd) + (ad+bc)i
    q_float = q.float()
    k_float = k.float()
    
    # 将freqs_cis扩展到匹配q的形状
    freqs_cis = freqs_cis.unsqueeze(0).unsqueeze(0)  # (1, 1, seq, dim//2)
    
    # 复数乘法
    q_complex = torch.view_as_complex(q_float.reshape(*q_float.shape[:-1], -1, 2))
    k_complex = torch.view_as_complex(k_float.reshape(*k_float.shape[:-1], -1, 2))
    
    q_rotated = torch.view_as_real(q_complex * freqs_cis).flatten(-2)
    k_rotated = torch.view_as_real(k_complex * freqs_cis).flatten(-2)
    
    return q_rotated.type_as(q), k_rotated.type_as(k)


# 简化版本：只展示核心思想
class SimpleRoPE:
    """简化版RoPE，用于理解原理"""
    
    @staticmethod
    def rotate_half(x, dim):
        """旋转操作的核心"""
        x1 = x[..., :x.shape[dim] // 2]
        x2 = x[..., x.shape[dim] // 2:]
        return torch.cat([-x2, x1], dim=dim)
    
    @staticmethod
    def apply(x, freqs):
        """
        x: (batch, seq, dim)
        freqs: (seq, dim//2) - 预计算的旋转角度
        """
        x1, x2 = x[..., :x.shape[-1] // 2], x[..., x.shape[-1] // 2:]
        
        # 旋转
        a = x1 * freqs[..., 0:1] - x2 * freqs[..., 1:2]
        b = x1 * freqs[..., 1:2] + x2 * freqs[..., 0:1]
        
        return torch.cat([a, b], dim=-1)
```

## 3.5 ALiBi：不需要位置编码的位置编码

ALiBi（Attention with Linear Biases）是一种完全不同的思路：不需要显式的位置编码，而是在注意力分数上添加线性偏置。

```python
def alibi_bias(seq_len, num_heads, device='cpu'):
    """
    生成ALiBi偏置矩阵
    
    核心思想：
    - 不在embedding上添加位置信息
    - 而是在注意力分数上添加与距离相关的线性偏置
    - 距离越远，偏置越大（越不被注意）
    """
    # 创建距离矩阵
    distances = torch.arange(1, seq_len + 1, device=device)[:, None] - torch.arange(1, seq_len + 1, device=device)[None, :]
    distances = distances.float() // 2  # 整数除法，距离取半
    
    # 不同的头使用不同的斜率
    # 头i的斜率 = 2^(-8/num_heads * i)
    slopes = 2 ** (-8 / num_heads * torch.arange(1, num_heads + 1, device=device).float())
    
    # 计算偏置：(num_heads, seq, seq)
    bias = -slopes[:, None, None] * torch.abs(distances)[None, :, :]
    
    return bias  # 形状：(num_heads, seq_len, seq_len)


# 验证ALiBi的特性
def test_alibi():
    bias = alibi_bias(seq_len=8, num_heads=4)
    print("ALiBi偏置矩阵（对角线为0，越远越小）:")
    print(bias[0])  # 第一个头的偏置
```

### 3.6 位置编码方案对比

| 方案 | 代表模型 | 优点 | 缺点 |
|------|----------|------|------|
| Sinusoidal | 原始Transformer | 可外推、无需学习 | 效果一般 |
| Learned PE | BERT | 效果好 | 不能外推到训练长度外 |
| RoPE | LLaMA, GLM | 支持长上下文、外推好 | 实现较复杂 |
| ALiBi | MPT, Bloom | 优秀的外推能力 | 不支持相对位置计算 |

---

# 四、从零实现一个完整的Transformer

现在我们来实现一个完整的mini-Transformer。这个实现麻雀虽小但五脏俱全，包括编码器、解码器、掩码、训练循环等。

## 4.1 完整代码

```python
"""
Mini Transformer: 从零实现一个完整的Transformer
用于机器翻译任务作为示例
"""

import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torch.utils.data import Dataset, DataLoader
import math
import random
import numpy as np

# ==================== 1. 位置编码 ====================

class PositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000, dropout=0.1):
        super().__init__()
        self.dropout = nn.Dropout(p=dropout)
        
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len, dtype=torch.float).unsqueeze(1)
        div_term = torch.exp(torch.arange(0, d_model, 2).float() * (-math.log(10000.0) / d_model))
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        pe = pe.unsqueeze(0)
        self.register_buffer('pe', pe)
    
    def forward(self, x):
        x = x + self.pe[:, :x.size(1)]
        return self.dropout(x)


# ==================== 2. 注意力层 ====================

class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads, dropout=0.1):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
        
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, query, key, value, mask=None):
        batch_size = query.size(0)
        
        # 线性投影并分头
        Q = self.W_q(query).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(key).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(value).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        
        # 计算注意力
        if mask is not None:
            if len(mask.shape) == 3:
                mask = mask.unsqueeze(1)
            attn_output = F.scaled_dot_product_attention(Q, K, V, attn_mask=mask, dropout=self.dropout if self.training else 0)
        else:
            attn_output = F.scaled_dot_product_attention(Q, K, V, dropout=self.dropout if self.training else 0)
        
        # 合并多头
        attn_output = attn_output.transpose(1, 2).contiguous().view(batch_size, -1, self.d_model)
        
        return self.W_o(attn_output)


# ==================== 3. 前馈网络 ====================

class FeedForward(nn.Module):
    """
    Position-wise Feed-Forward Network
    
    两个线性层，中间有ReLU/GELU激活
    FFN(x) = W2 * activation(W1 * x + b1) + b2
    
    这里用GELU代替ReLU，因为现代大模型普遍用GELU
    """
    def __init__(self, d_model, d_ff, dropout=0.1):
        super().__init__()
        self.linear1 = nn.Linear(d_model, d_ff)
        self.linear2 = nn.Linear(d_ff, d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x):
        return self.linear2(self.dropout(F.gelu(self.linear1(x))))


# ==================== 4. 编码器层 ====================

class EncoderLayer(nn.Module):
    """
    编码器层 = 自注意力 + 残差 + 层归一化 + FFN + 残差 + 层归一化
    
    Pre-LN结构：现代Transformer更常用Pre-LN，训练更稳定
    """
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        self.self_attn = MultiHeadAttention(d_model, num_heads, dropout)
        self.feed_forward = FeedForward(d_model, d_ff, dropout)
        
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, mask=None):
        # Pre-LN 结构
        # 自注意力 + 残差连接
        x = x + self.dropout(self.self_attn(self.norm1(x), self.norm1(x), self.norm1(x), mask))
        
        # FFN + 残差连接
        x = x + self.dropout(self.feed_forward(self.norm2(x)))
        
        return x


# ==================== 5. 解码器层 ====================

class DecoderLayer(nn.Module):
    """
    解码器层包含三个子层：
    1. Masked Self-Attention（不能看到未来）
    2. Cross-Attention（看向编码器）
    3. FFN
    """
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        super().__init__()
        
        self.self_attn = MultiHeadAttention(d_model, num_heads, dropout)
        self.cross_attn = MultiHeadAttention(d_model, num_heads, dropout)
        self.feed_forward = FeedForward(d_model, d_ff, dropout)
        
        self.norm1 = nn.LayerNorm(d_model)
        self.norm2 = nn.LayerNorm(d_model)
        self.norm3 = nn.LayerNorm(d_model)
        self.dropout = nn.Dropout(dropout)
    
    def forward(self, x, enc_output, tgt_mask=None, src_mask=None):
        # Masked Self-Attention
        x = x + self.dropout(self.self_attn(self.norm1(x), self.norm1(x), self.norm1(x), tgt_mask))
        
        # Cross-Attention（解码器Query，编码器Key/Value）
        x = x + self.dropout(self.cross_attn(self.norm2(x), enc_output, enc_output, src_mask))
        
        # FFN
        x = x + self.dropout(self.feed_forward(self.norm3(x)))
        
        return x


# ==================== 6. 完整Transformer ====================

class MiniTransformer(nn.Module):
    """
    完整的Transformer模型
    
    组件：
    - 词嵌入层
    - 位置编码
    - N层编码器
    - N层解码器
    - 输出投影层（vocab projection）
    """
    
    def __init__(self, vocab_size, d_model=512, num_heads=8, num_layers=6, d_ff=2048, dropout=0.1, max_len=5000):
        super().__init__()
        
        self.d_model = d_model
        self.vocab_size = vocab_size
        
        # 词嵌入
        self.token_embedding = nn.Embedding(vocab_size, d_model)
        self.position_encoding = PositionalEncoding(d_model, max_len, dropout)
        
        # 编码器和解码器
        self.encoder_layers = nn.ModuleList([
            EncoderLayer(d_model, num_heads, d_ff, dropout) 
            for _ in range(num_layers)
        ])
        self.decoder_layers = nn.ModuleList([
            DecoderLayer(d_model, num_heads, d_ff, dropout) 
            for _ in range(num_layers)
        ])
        
        # 输出层
        self.projection = nn.Linear(d_model, vocab_size)
        
        self.dropout = nn.Dropout(dropout)
        self._init_weights()
    
    def _init_weights(self):
        """权重初始化"""
        for p in self.parameters():
            if p.dim() > 1:
                nn.init.xavier_uniform_(p)
    
    def encode(self, src, src_mask=None):
        """
        编码器前向传播
        src: (batch, src_len)
        """
        # 词嵌入 + 位置编码
        x = self.dropout(self.token_embedding(src)) * math.sqrt(self.d_model)
        x = self.position_encoding(x)
        
        # 通过所有编码器层
        for layer in self.encoder_layers:
            x = layer(x, src_mask)
        
        return x
    
    def decode(self, tgt, enc_output, tgt_mask=None, src_mask=None):
        """
        解码器前向传播
        tgt: (batch, tgt_len)
        """
        x = self.dropout(self.token_embedding(tgt)) * math.sqrt(self.d_model)
        x = self.position_encoding(x)
        
        for layer in self.decoder_layers:
            x = layer(x, enc_output, tgt_mask, src_mask)
        
        return self.projection(x)
    
    def forward(self, src, tgt, src_mask=None, tgt_mask=None):
        """
        完整前向传播
        返回: (batch, tgt_len, vocab_size)
        """
        enc_output = self.encode(src, src_mask)
        dec_output = self.decode(tgt, enc_output, tgt_mask, src_mask)
        return dec_output


# ==================== 7. 掩码生成 ====================

def create_padding_mask(seq, pad_idx=0):
    """
    生成padding mask
    标记哪些位置是padding（在注意力中应该被mask掉）
    """
    return (seq == pad_idx).unsqueeze(1).unsqueeze(2)


def create_causal_mask(size):
    """
    生成因果mask（解码器中不能看到未来的token）
    
    上三角为True（要mask掉），下三角为False（可以看到）
    """
    return torch.triu(torch.ones(size, size), diagonal=1).bool().unsqueeze(0).unsqueeze(0)


# ==================== 8. 训练函数 ====================

def train_step(model, optimizer, src, tgt, tgt_input, tgt_output, criterion, device):
    """
    单步训练
    """
    model.train()
    
    src = src.to(device)
    tgt_input = tgt_input.to(device)
    tgt_output = tgt_output.to(device)
    
    # 创建掩码
    tgt_mask = create_causal_mask(tgt_input.size(1)).to(device)
    src_mask = None  # 简化起见，假设没有padding
    
    # 前向传播
    logits = model(src, tgt_input, src_mask, tgt_mask)
    
    # 计算损失（忽略padding位置的loss）
    loss = criterion(logits.view(-1, logits.size(-1)), tgt_output.view(-1))
    
    # 反向传播
    optimizer.zero_grad()
    loss.backward()
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)  # 梯度裁剪
    optimizer.step()
    
    return loss.item()


# ==================== 9. 简化数据集示例 ====================

class SimpleTranslationDataset(Dataset):
    """
    简化版翻译数据集
    实际使用时应该用真实的数据集
    """
    def __init__(self, src_texts, tgt_texts, src_tokenizer, tgt_tokenizer, max_len=100):
        self.src_texts = src_texts
        self.tgt_texts = tgt_texts
        self.src_tokenizer = src_tokenizer
        self.tgt_tokenizer = tgt_tokenizer
        self.max_len = max_len
    
    def __len__(self):
        return len(self.src_texts)
    
    def __getitem__(self, idx):
        src = self.src_tokenizer(self.src_texts[idx], self.max_len)
        tgt = self.tgt_tokenizer(self.tgt_texts[idx], self.max_len)
        
        # 解码器输入是 <sos> token开始的
        tgt_input = tgt[:-1]
        # 解码器输出是 </sos> token结束的
        tgt_output = tgt[1:]
        
        return torch.tensor(src), torch.tensor(tgt_input), torch.tensor(tgt_output)


if __name__ == "__main__":
    # 快速测试
    print("=== Mini Transformer 测试 ===")
    
    # 创建模型
    vocab_size = 10000
    model = MiniTransformer(vocab_size, d_model=256, num_heads=8, num_layers=4)
    
    # 测试前向传播
    batch_size = 2
    src_len = 10
    tgt_len = 8
    
    src = torch.randint(0, vocab_size, (batch_size, src_len))
    tgt_input = torch.randint(0, vocab_size, (batch_size, tgt_len))
    
    logits = model(src, tgt_input)
    print(f"输入形状: src={src.shape}, tgt_input={tgt_input.shape}")
    print(f"输出形状: {logits.shape}")  # (batch, tgt_len, vocab_size)
    
    # 计算损失
    tgt_output = torch.randint(0, vocab_size, (batch_size, tgt_len))
    criterion = nn.CrossEntropyLoss()
    loss = criterion(logits.view(-1, vocab_size), tgt_output.view(-1))
    print(f"初始损失: {loss.item():.4f}")
    
    # 测试掩码
    causal_mask = create_causal_mask(5)
    print(f"\n因果mask（True表示要mask掉）:\n{causal_mask[0, 0]}")
```

## 4.2 代码结构总结

整个Transformer由以下几个核心组件构成：

**编码器**：词嵌入 → 位置编码 → N个EncoderLayer → 输出

**解码器**：词嵌入 → 位置编码 → N个DecoderLayer → 词汇投影

**关键技巧**：
- 残差连接：让梯度直接流过，解决深层网络的训练问题
- 层归一化：稳定训练，加速收敛
- Dropout：防止过拟合
- 掩码：因果mask防止看到未来，padding mask忽略无用位置

---

# 五、大语言模型的训练秘密

## 5.1 GPT是怎么训练的？

GPT（Generative Pre-trained Transformer）的训练分为三个阶段：

### 5.1.1 阶段一：预训练（Pretraining）

预训练的目标是让模型学会"续写"文本。大规模语料（比如整个互联网）被切分成句子，模型学习预测下一个词。

这个任务看似简单，但当数据量足够大时，模型需要：
- 理解语法结构
- 掌握世界知识
- 学习推理能力
- 理解上下文关系

损失函数就是标准的语言模型交叉熵：

$$
\mathcal{L} = -\sum_{t=1}^{T} \log P(x_t | x_{<t})
$$

这叫**下一个Token预测**（Next Token Prediction），是所有自回归语言模型的核心训练目标。

### 5.1.2 阶段二：监督微调（SFT）

预训练后，模型已经是一个强大的"猜下一个词"的机器，但它不太会"听话"。SFT阶段用人工标注的问答对来教模型"怎么回答问题"。

格式通常是：

```
[用户问题] -> [人工回答]
```

SFT的数据量相对较小（几万到几十万条），但质量很高。模型在这里学会：
- 遵循指令
- 对话格式
- 专业领域知识

### 5.1.3 阶段三：人类反馈强化学习（RLHF）

RLHF是大语言模型最重要的创新之一。它解决了一个核心问题：模型生成的回答可能有毒、有害、不符合人类价值观。

RLHF的步骤：

**第一步：训练奖励模型（Reward Model）**

收集人类对同一问题的多个回答的比较数据，训练一个奖励模型 $r(x, y)$ 来预测人类偏好。

**第二步：用强化学习微调**

用PPO算法（Proximal Policy Optimization）来优化语言模型：

$$
\max_\theta \mathbb{E}_{x \sim D, y \sim \pi_\theta(y|x)} [r(x, y)] - \beta \cdot D_{KL}(\pi_\theta || \pi_{ref})
$$

第二项是KL散度惩罚，防止新策略偏离SFT模型太远，保持模型不要"走偏"。

### 5.1.4 完整的训练代码框架

```python
"""
LLM训练的简化框架
展示三个阶段的训练流程
"""

import torch
import torch.nn as nn
from torch.utils.data import DataLoader
from transformers import GPT2LMHeadModel, GPT2Tokenizer, AutoTokenizer
import json

class LLMTrainingPipeline:
    """
    大语言模型训练的完整流程
    """
    
    def __init__(self, model_name="gpt2", device="cuda"):
        self.device = device
        self.tokenizer = GPT2Tokenizer.from_pretrained(model_name)
        self.model = GPT2LMHeadModel.from_pretrained(model_name).to(device)
        self.ref_model = None  # 用于RLHF
        
    # ==================== 预训练 ====================
    def pretrain(self, dataloader, epochs=1):
        """
        预训练阶段
        大规模无标注语料，学习语言建模
        """
        optimizer = torch.optim.AdamW(self.model.parameters(), lr=1e-4)
        self.model.train()
        
        for epoch in range(epochs):
            total_loss = 0
            for batch in dataloader:
                input_ids = batch['input_ids'].to(self.device)
                
                # 语言模型只需要一个输入
                # 标签就是 shifted 的 input_ids
                outputs = self.model(input_ids, labels=input_ids)
                loss = outputs.loss
                
                optimizer.zero_grad()
                loss.backward()
                optimizer.step()
                
                total_loss += loss.item()
            
            print(f"预训练 Epoch {epoch+1}, Loss: {total_loss/len(dataloader):.4f}")
    
    # ==================== SFT微调 ====================
    def sft_train(self, dataset_path, epochs=3):
        """
        SFT阶段
        使用指令数据微调
        """
        # 加载SFT数据
        with open(dataset_path, 'r') as f:
            sft_data = json.load(f)
        
        optimizer = torch.optim.AdamW(self.model.parameters(), lr=5e-6)
        
        for epoch in range(epochs):
            self.model.train()
            total_loss = 0
            
            for item in sft_data:
                # 格式：[INST] 用户问题 [/INST] 回答
                prompt = f"[INST] {item['instruction']} [/INST] {item['response']}"
                input_ids = self.tokenizer(prompt, return_tensors="pt")['input_ids'].to(self.device)
                
                outputs = self.model(input_ids, labels=input_ids)
                loss = outputs.loss
                
                optimizer.zero_grad()
                loss.backward()
                optimizer.step()
                
                total_loss += loss.item()
            
            print(f"SFT Epoch {epoch+1}, Loss: {total_loss/len(sft_data):.4f}")
    
    # ==================== RLHF训练 ====================
    def setup_rlhf(self):
        """
        设置RLHF
        保存参考模型，计算KL散度
        """
        self.ref_model = GPT2LMHeadModel.from_pretrained("gpt2").to(self.device)
        self.ref_model.eval()  # 参考模型不更新
    
    def rlhf_step(self, prompt, responses, rewards):
        """
        RLHF的单步更新（简化版）
        
        参数：
            prompt: 用户问题
            responses: 模型生成的多个回答
            rewards: 人类/奖励模型给出的评分
        """
        optimizer = torch.optim.AdamW(self.model.parameters(), lr=1e-6)
        
        for response, reward in zip(responses, rewards):
            # 构建输入
            input_text = f"[INST] {prompt} [/INST] {response}"
            input_ids = self.tokenizer(input_text, return_tensors="pt")['input_ids'].to(self.device)
            
            # 前向传播
            outputs = self.model(input_ids)
            logits = outputs.logits
            
            # 简化版PPO：直接用reward作为loss
            # 实际实现需要用完整的PPO算法
            log_probs = torch.log_softmax(logits[:, :-1], dim=-1)
            
            # 计算reward加权损失
            loss = -reward * log_probs.mean()
            
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
        
        # KL惩罚（保持新策略不偏离参考模型太远）
        if self.ref_model:
            kl_penalty = self.compute_kl_penalty()
            return kl_penalty
    
    def compute_kl_penalty(self):
        """计算KL散度惩罚"""
        with torch.no_grad():
            ref_logits = self.ref_model(input_ids).logits
        current_logits = self.model(input_ids).logits
        
        # KL(current || ref)
        kl = torch.nn.functional.kl_div(
            torch.log_softmax(current_logits, dim=-1),
            torch.softmax(ref_logits, dim=-1),
            reduction='batchmean'
        )
        return kl
```

## 5.2 为什么LLM能涌现出能力？

GPT-3之后，研究者发现一个神奇的现象：当模型规模超过某个阈值后，会突然涌现出一些在小模型上没有的能力，比如：

- **思维链（Chain of Thought）**：能进行多步推理
- **零样本学习（Zero-shot）**：能完成从未见过的任务
- **代码生成**：能写程序

这背后的原因仍然是研究热点。一个可能的解释是：

当模型足够大时，它在预训练数据中见过足够多的"例子"。比如"解数学题"这个任务，虽然没有专门的数学训练数据，但在网页、教材、论坛上散落着大量的解题过程。大模型通过记忆+泛化，学会了解题的模式。

这让我们重新思考什么是"理解"——也许对语言模型来说，"能预测下一个词"和"真正理解"之间的界限并没有那么清晰。

---

# 六、Transformer在视觉领域的应用

## 6.1 ViT：Vision Transformer

Transformer最初是为NLP设计的，但研究者很快发现它也能用于图像——只要把图像切分成小块（patches）。

### 6.1.1 核心思想

ViT的做法很简单：
1. 把图像切成16x16的小块
2. 每个小块flatten成一个向量
3. 这些向量就像句子中的词，通过Transformer处理

```
图像 → 切分patches → Linear Projection → Transformer编码器 → 分类头
```

### 6.1.2 ViT代码实现

```python
import torch
import torch.nn as nn
from einops import rearrange

class PatchEmbedding(nn.Module):
    """
    图像转Patch嵌入
    把H x W x C的图像切成N个P x P x C的小块
    """
    def __init__(self, img_size=224, patch_size=16, in_channels=3, embed_dim=768):
        super().__init__()
        self.img_size = img_size
        self.patch_size = patch_size
        self.num_patches = (img_size // patch_size) ** 2
        
        # 线性投影层
        self.proj = nn.Conv2d(in_channels, embed_dim, kernel_size=patch_size, stride=patch_size)
    
    def forward(self, x):
        # x: (batch, channels, height, width)
        x = self.proj(x)  # (batch, embed_dim, num_patches_h, num_patches_w)
        x = rearrange(x, 'b c h w -> b (h w) c')  # (batch, num_patches, embed_dim)
        return x


class VisionTransformer(nn.Module):
    """
    Vision Transformer (ViT)
    
    核心组件：
    1. Patch Embedding
    2. 类别token [CLS] 和位置编码
    3. 标准Transformer编码器
    4. 分类头
    """
    def __init__(self, img_size=224, patch_size=16, in_channels=3, 
                 embed_dim=768, depth=12, num_heads=12, num_classes=1000):
        super().__init__()
        
        self.patch_embed = PatchEmbedding(img_size, patch_size, in_channels, embed_dim)
        self.num_patches = self.patch_embed.num_patches
        
        # 可学习的类别token
        self.cls_token = nn.Parameter(torch.zeros(1, 1, embed_dim))
        
        # 位置编码
        self.pos_embed = nn.Parameter(torch.zeros(1, self.num_patches + 1, embed_dim))
        
        # Transformer编码器（和NLP版本的Encoder完全一样）
        self.encoder_layers = nn.ModuleList([
            EncoderLayer(embed_dim, num_heads) for _ in range(depth)
        ])
        self.norm = nn.LayerNorm(embed_dim)
        
        # 分类头
        self.head = nn.Linear(embed_dim, num_classes)
        
        self._init_weights()
    
    def _init_weights(self):
        nn.init.trunc_normal_(self.pos_embed, std=0.02)
        nn.init.trunc_normal_(self.cls_token, std=0.02)
    
    def forward(self, x):
        batch_size = x.shape[0]
        
        # Patch嵌入
        x = self.patch_embed(x)  # (batch, num_patches, embed_dim)
        
        # 添加类别token
        cls_tokens = self.cls_token.expand(batch_size, -1, -1)
        x = torch.cat([cls_tokens, x], dim=1)  # (batch, num_patches + 1, embed_dim)
        
        # 添加位置编码
        x = x + self.pos_embed
        
        # Transformer编码
        for layer in self.encoder_layers:
            x = layer(x)
        x = self.norm(x)
        
        # 取[CLS] token的输出作为图像表示
        cls_output = x[:, 0]
        
        return self.head(cls_output)
```

## 6.2 SAM：Segment Anything Model

SAM是Meta推出的"通用分割模型"，能在给定提示（point、box、text等）的情况下分割图像中的任意物体。

SAM的核心创新：
- **Prompt-based**：分割变成了一种"问答"
- **IoU head**：预测分割质量的置信度
- **海量数据**：用数据引擎生成了1100万张训练图像

SAM的架构：
1. 图像编码器：ViT或ResNet，提取图像特征
2. 提示编码器：编码点、框、掩码等提示
3. 解码器：融合图像特征和提示，生成掩码

## 6.3 更多视觉Transformer变体

| 模型 | 年份 | 核心创新 |
|------|------|----------|
| ViT | 2020 | 将图像转patch，用Transformer处理 |
| DeiT | 2021 | 数据高效训练，学生蒸馏 |
| Swin Transformer | 2021 | 层级结构，移位窗口注意力 |
| MAE | 2022 | 掩码自编码器，ImageNet预训练新范式 |
| SAM | 2023 | 通用分割，提示驱动 |
| DINOv2 | 2024 | 自监督视觉特征，性能超越ImageNet |

---

# 七、高效微调技术

## 7.1 为什么需要高效微调？

训练一个完整的大模型需要海量的算力和数据：
- LLaMA-7B：需要约8张A100（80GB）GPU
- LLaMA-70B：需要约16张A100（80GB）GPU
- GPT-4：传闻需要数万张GPU

这对普通研究者和中小企业来说完全不可承受。高效微调技术让我们只需要更新模型的一小部分参数，就能达到接近全量微调的效果。

## 7.2 LoRA：Low-Rank Adaptation

### 7.2.1 核心思想

LoRA的灵感来自一个大发现：大模型虽然参数多，但实际有效的参数空间是低秩的。

LoRA不直接更新原始权重 $W$，而是添加两个小的矩阵 $A$ 和 $B$：

$$
W' = W + \Delta W = W + BA
$$

其中 $W \in \mathbb{R}^{d \times k}$，$A \in \mathbb{R}^{r \times k}$，$B \in \mathbb{R}^{d \times r}$，$r \ll \min(d, k)$。

### 7.2.2 代码实现

```python
import torch
import torch.nn as nn
import math

class LoRALinear(nn.Module):
    """
    LoRA线性层
    
    将原始线性层分解为：
    - 冻结的原始权重 W
    - 可训练的低秩矩阵 A 和 B
    
    参数量对比：
    - 原始: d * k
    - LoRA: r * (d + k), r << min(d, k)
    
    例如：d=4096, k=4096, r=8
    - 原始: 16,777,216 参数
    - LoRA: 8 * (4096 + 4096) = 65,536 参数
    - 压缩比：256倍！
    """
    
    def __init__(self, in_features, out_features, rank=8, alpha=16, dropout=0.1):
        super().__init__()
        self.in_features = in_features
        self.out_features = out_features
        self.rank = rank
        self.alpha = alpha
        
        # 冻结原始权重
        self.weight = nn.Parameter(torch.empty(out_features, in_features), requires_grad=False)
        
        # LoRA参数
        self.lora_A = nn.Parameter(torch.empty(rank, in_features))
        self.lora_B = nn.Parameter(torch.empty(out_features, rank))
        self.lora_dropout = nn.Dropout(dropout)
        
        # 缩放因子
        self.scaling = alpha / rank
        
        self._init_lora_weights()
    
    def _init_lora_weights(self):
        """初始化LoRA的A和B"""
        # A用随机正态分布初始化
        nn.init.normal_(self.lora_A, std=1 / self.rank)
        # B初始化为0，这样开始时LoRA不起作用
        nn.init.zeros_(self.lora_B)
    
    def forward(self, x):
        """
        前向传播
        原权重冻结，只计算LoRA部分的贡献
        """
        # 原始输出（需要手动实现，因为原始weight被冻结）
        original_output = F.linear(x, self.weight.t())
        
        # LoRA部分的输出
        # x -> lora_A -> lora_dropout -> lora_B -> scale
        lora_output = F.linear(self.lora_dropout(x), self.lora_B @ self.lora_A) * self.scaling
        
        return original_output + lora_output


class LoRAViTAttention(nn.Module):
    """
    为ViT的注意力层添加LoRA
    
    通常LoRA应用于：
    - Q投影：lora_A_q, lora_B_q
    - V投影：lora_A_v, lora_B_v
    """
    
    def __init__(self, original_attn, rank=8, alpha=16):
        super().__init__()
        self.original_attn = original_attn
        d_model = original_attn.in_features
        
        self.rank = rank
        self.alpha = alpha
        
        # LoRA for Query
        self.lora_A_q = nn.Parameter(torch.zeros(rank, d_model))
        self.lora_B_q = nn.Parameter(torch.zeros(d_model, rank))
        
        # LoRA for Value
        self.lora_A_v = nn.Parameter(torch.zeros(rank, d_model))
        self.lora_B_v = nn.Parameter(torch.zeros(d_model, rank))
        
        # 冻结原始注意力层
        for p in original_attn.parameters():
            p.requires_grad = False
        
        self._init_lora_weights()
    
    def _init_lora_weights(self):
        nn.init.normal_(self.lora_A_q, std=1 / self.rank)
        nn.init.normal_(self.lora_A_v, std=1 / self.rank)
        nn.init.zeros_(self.lora_B_q)
        nn.init.zeros_(self.lora_B_v)
    
    def forward(self, *args, **kwargs):
        # 获取原始注意力输出
        output = self.original_attn(*args, **kwargs)
        
        # 添加LoRA的贡献（简化实现）
        # 实际需要访问Q, K, V分别添加LoRA
        return output


def apply_lora_to_model(model, rank=8, alpha=16, target_modules=['q_proj', 'v_proj']):
    """
    给模型应用LoRA
    
    参数：
        model: 原始模型
        rank: LoRA秩
        alpha: 缩放因子
        target_modules: 要应用LoRA的模块名
    """
    for name, module in model.named_modules():
        if isinstance(module, nn.Linear):
            # 检查是否是目标模块
            if any(target in name for target in target_modules):
                # 替换为LoRA版本
                parent_name = name.rsplit('.', 1)[0]
                parent = model.get_submodule(parent_name)
                child_name = name.rsplit('.', 1)[1]
                
                lora_module = LoRALinear(
                    module.in_features, 
                    module.out_features, 
                    rank=rank, 
                    alpha=alpha
                )
                # 复制原始权重
                lora_module.weight.data = module.weight.data.clone()
                if module.bias is not None:
                    lora_module.bias = module.bias.clone()
                
                setattr(parent, child_name, lora_module)
                print(f"Applied LoRA to {name}")


# ==================== 训练脚本示例 ====================

def train_with_lora():
    """
    LoRA训练示例
    """
    from transformers import AutoModelForCausalLM, AutoTokenizer
    
    # 加载原始模型
    model_name = "gpt2"
    model = AutoModelForCausalLM.from_pretrained(model_name)
    
    # 应用LoRA
    apply_lora_to_model(
        model, 
        rank=8, 
        alpha=16, 
        target_modules=['c_attn', 'c_proj']  # GPT-2的注意力层
    )
    
    # 统计可训练参数
    trainable_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
    total_params = sum(p.numel() for p in model.parameters())
    print(f"可训练参数: {trainable_params:,} / {total_params:,} ({100*trainable_params/total_params:.2f}%)")
    
    # 开始训练...
    # optimizer只优化LoRA参数
    optimizer = torch.optim.AdamW(
        filter(lambda p: p.requires_grad, model.parameters()),
        lr=1e-4
    )
```

## 7.3 Adapter：适配器微调

Adapter是另一种高效微调方法，在Transformer层之间插入小的"适配器"模块。

```python
class Adapter(nn.Module):
    """
    Adapter模块
    
    核心思想：在原始Transformer层中插入Adapter
    Adapter通常包含：下投影 -> 非线性 -> 上投影
    
    前向传播：
    x -> LayerNorm -> Adapter -> + x -> 输出
    """
    
    def __init__(self, d_model, adapter_dim, bottleneck_dim):
        super().__init__()
        
        # Adapter通常包含一个下投影和上投影
        self.down_project = nn.Linear(d_model, adapter_dim)
        self.up_project = nn.Linear(adapter_dim, d_model)
        
        # 非线性激活
        self.activation = nn.GELU()
        
        # 初始化为接近恒等映射
        nn.init.zeros_(self.down_project.weight)
        nn.init.zeros_(self.down_project.bias)
        nn.init.zeros_(self.up_project.weight)
        nn.init.zeros_(self.up_project.bias)
    
    def forward(self, x):
        # Adapter的前向传播
        adapter_output = self.up_project(self.activation(self.down_project(x)))
        return x + adapter_output  # 残差连接


class TransformerLayerWithAdapter(nn.Module):
    """
    带Adapter的Transformer层
    """
    
    def __init__(self, d_model, num_heads, d_ff, adapter_dim=64):
        super().__init__()
        
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.norm1 = nn.LayerNorm(d_model)
        
        self.feed_forward = FeedForward(d_model, d_ff)
        self.norm2 = nn.LayerNorm(d_model)
        
        # 添加Adapter
        self.adapter = Adapter(d_model, adapter_dim, d_ff)
        self.norm3 = nn.LayerNorm(d_model)
    
    def forward(self, x, mask=None):
        # 自注意力 + 残差
        x = x + self.self_attn(self.norm1(x), self.norm1(x), self.norm1(x), mask)
        
        # FFN + 残差
        x = x + self.feed_forward(self.norm2(x))
        
        # Adapter + 残差
        x = x + self.adapter(self.norm3(x))
        
        return x
```

## 7.4 高效微调方法对比

| 方法 | 核心思想 | 参数量 | 效果 |
|------|----------|--------|------|
| LoRA | 低秩矩阵分解 | 极低（0.1%-1%） | 接近全量微调 |
| Adapter | 插入适配器模块 | 低（1%-5%） | 效果好 |
| Prefix-Tuning | 可学习的前缀 | 极低 | 效果一般 |
| Prompt-Tuning | 可学习的prompt | 最低 | 依赖基座模型 |
| BitFit | 只训练bias | 最低 | 效果一般 |

---

# 八、工程实践：让Transformer跑得更快

## 8.1 显存优化

大模型训练最头疼的问题是显存不够。以下是几个关键的优化技术：

### 8.1.1 混合精度训练

用FP16/BF16代替FP32，可以节省一半显存，同时现代GPU的BF16计算单元更快。

```python
from torch.cuda.amp import autocast, GradScaler

# 训练循环
scaler = GradScaler()

for batch in dataloader:
    optimizer.zero_grad()
    
    # 自动混合精度
    with autocast(dtype=torch.bfloat16):
        outputs = model(input_ids, labels=labels)
        loss = outputs.loss
    
    # 梯度缩放，防止下溢
    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

### 8.1.2 梯度检查点（Gradient Checkpointing）

用时间换空间：前向传播时不保存所有中间激活值，而是在反向传播时重新计算。

```python
# PyTorch中启用梯度检查点
model.train()

# 方式1：模型级别
model = gradient_checkpointing_enable(model)

# 方式2：手动指定需要检查点的层
torch.utils.checkpoint.checkpoint(forward_fn, *args)
```

### 8.1.3 Flash Attention

Flash Attention是革命性的注意力实现，将显存复杂度从 $O(N^2)$ 降到 $O(N)$。

原理：
- 分块处理：将Q、K、V分成小块
- 在线计算：流式更新注意力统计量
- 重计算：反向传播时重新计算注意力，而非存储

```python
# PyTorch 2.0+ 直接支持Flash Attention
from torch.nn.functional import scaled_dot_product_attention

# 自动选择最优实现（Flash Attention/CUDA实现/朴素实现）
output = scaled_dot_product_attention(
    q, k, v,
    attn_mask=None,
    dropout_p=0.0 if not training else 0.1,
    is_causal=True  # 自动生成causal mask
)
```

## 8.2 推理加速

### 8.2.1 KV Cache

自回归生成时，解码器每次只生成一个新token，但需要重新计算所有历史token的注意力。KV Cache缓存已计算的Key和Value，避免重复计算。

```python
def generate_with_kv_cache(model, prompt, max_length=100):
    """
    使用KV Cache加速生成
    """
    input_ids = tokenizer.encode(prompt, return_tensors="pt").to(device)
    
    # 首次前向传播
    outputs = model(input_ids, use_cache=True)
    past_key_values = outputs.past_key_values  # 缓存的KV
    
    next_token = outputs.logits[:, -1, :].argmax(dim=-1, keepdim=True)
    generated = [next_token.item()]
    
    # 后续token只需要计算新token的注意力
    for _ in range(max_length - 1):
        outputs = model(
            next_token, 
            past_key_values=past_key_values,
            use_cache=True
        )
        past_key_values = outputs.past_key_values
        next_token = outputs.logits[:, -1, :].argmax(dim=-1, keepdim=True)
        generated.append(next_token.item())
        
        if next_token == tokenizer.eos_token_id:
            break
    
    return tokenizer.decode(generated)
```

### 8.2.2 批量推理

多个请求一起推理，共享计算。

```python
# 批量推理示例
prompts = ["今天天气", "你好", "讲个笑话"]
input_ids_list = [tokenizer.encode(p, return_tensors="pt") for p in prompts]

# 填充到相同长度
max_len = max(ids.shape[1] for ids in input_ids_list)
input_ids_padded = torch.zeros(len(prompts), max_len, dtype=torch.long)
attention_mask = torch.zeros_like(input_ids_padded)

for i, ids in enumerate(input_ids_list):
    input_ids_padded[i, :ids.shape[1]] = ids
    attention_mask[i, :ids.shape[1]] = 1

# 批量前向传播
outputs = model(input_ids_padded.to(device), attention_mask=attention_mask.to(device))
```

## 8.3 模型量化

量化通过减少权重精度来节省显存和加速推理。

| 量化方式 | 精度 | 显存节省 | 速度提升 | 精度损失 |
|----------|------|----------|----------|----------|
| FP16 | 16位 | 50% | 1.2x | 几乎无 |
| BF16 | 16位 | 50% | 1.2x | 几乎无 |
| INT8 | 8位 | 75% | 2-4x | 低 |
| INT4 | 4位 | 87.5% | 4-8x | 中等 |
| GPTQ | 4/8位 | 87.5% | 4-8x | 可控 |

```python
# 使用bitsandbytes进行量化
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

quantization_config = BitsAndBytesConfig(
    load_in_8bit=True,  # INT8量化
    llm_int8_threshold=6.0,  # outlier阈值
)

model = AutoModelForCausalLM.from_pretrained(
    "gpt2",
    quantization_config=quantization_config,
    device_map="auto"
)
```

---

# 九、新手常见困惑解答

## 9.1 为什么叫"注意力"？它到底在"注意"什么？

这是一个非常好的问题。很多初学者被"注意力"这个名字误导了。

其实这里没有真正的"注意力"，只是一种类比：

当你读"那只猫很可爱，它在沙发上睡觉"时，你的大脑会"注意"到"它"指的是"猫"。注意力机制做的事类似——它计算句子中每个词对其他词的"相关程度"。

更准确的名字可能是"加权聚合"或"上下文融合"，但"注意力"这个名字太有画面感了，所以沿用至今。

## 9.2 多头注意力为什么要"多头"？

可以理解为"多角度理解"：

想象你在分析一句话。你的大脑可能同时关注：
- 语法关系（主谓宾）
- 指代关系（代词指代什么）
- 情感倾向（积极还是消极）
- 实体识别（人名、地名）

每个"头"就像一个"观察角度"，不同的头学习不同类型的关联。

## 9.3 位置编码是必须的吗？

不是绝对的，但几乎是必须的。

理论上，如果只用纯注意力（无位置编码），模型可以通过学习来隐式捕获位置信息。但实验证明，显式添加位置编码让训练更高效、效果更好。

## 9.4 Decoder的Masked Self-Attention是怎么工作的？

"Masked"的意思是"遮住"未来位置。

在训练时，解码器知道完整的正确答案（tgt），但不能"作弊"看到当前位置之后的内容。所以要mask掉：

```
位置0: 可以看[0]
位置1: 可以看[0, 1]
位置2: 可以看[0, 1, 2]
...
```

实现上，在softmax之前把要mask的位置设为一个很大的负数，这样softmax后这些位置的权重接近0。

## 9.5 Transformer训练为什么不稳定？

深层Transformer训练不稳定的原因：
1. **残差累积**：每层输出和输入相加，深层时数值可能爆炸
2. **注意力权重不稳定**：某些head可能学到极端权重
3. **梯度消失/爆炸**：深层网络的经典问题

解决方案：
- **层归一化**：放在残差连接内部（Post-LN）或之前（Pre-LN）
- **Xavier初始化**：让梯度方差稳定
- **梯度裁剪**：防止梯度爆炸
- **学习率预热**：开始时用小学习率，逐步增加

## 9.6 BERT和GPT的区别是什么？

| 维度 | BERT | GPT |
|------|------|-----|
| 架构 | Encoder-only | Decoder-only |
| 注意力 | 双向（能看到全部上下文） | 单向（只能看到之前） |
| 预训练任务 | MLM + NSP | Next Token Prediction |
| 擅长任务 | 理解任务（分类、NER） | 生成任务（续写、对话） |
| 代表模型 | BERT, RoBERTa, DeBERTa | GPT-2/3/4, LLaMA |

简单说：**BERT是完形填空**，**GPT是续写高手**。

---

# 十、参考资料

- Vaswani, A., et al. (2017). Attention is All You Need. *NeurIPS*.
- Devlin, J., et al. (2018). BERT: Pre-training of Deep Bidirectional Transformers. *NAACL*.
- Radford, A., et al. (2019). Language Models are Unsupervised Multitask Learners. *OpenAI Technical Report*.
- Dao, T., et al. (2022). FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness. *NeurIPS*.
- Su, J., et al. (2022). RoFormer: Enhanced Transformer with Rotary Position Embedding. *arXiv*.
- Press, O., et al. (2021). ALiBi: Train Short, Test Long: Attention with Linear Biases Enable Input Length Extrapolation. *ICLR*.
- Hu, E. J., et al. (2021). LoRA: Low-Rank Adaptation of Large Language Models. *ICLR*.
- Dosovitskiy, A., et al. (2020). An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale. *ICLR*.
- Ouyang, L., et al. (2022). Training language models to follow instructions with human feedback. *NeurIPS*.
