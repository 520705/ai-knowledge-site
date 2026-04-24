---
title: GAN深度指南
date: 2026-04-18
tags:
  - 生成对抗网络
  - 深度学习
  - 生成模型
  - 对抗训练
  - 神经网络
categories:
  - 人工智能
  - 对抗算法
  - GAN及其变体
alias: GAN深度指南
---

## 关键词

| 术语 | 英文 | 核心概念 |
|------|------|----------|
| 生成对抗网络 | GAN | 生成器与判别器的对抗框架 |
| 生成器 | Generator | 从随机噪声生成假样本 |
| 判别器 | Discriminator | 区分真假样本 |
| 模式崩溃 | Mode Collapse | 生成样本多样性不足 |
| JS散度 | Jensen-Shannon Divergence | GAN损失函数的理论基础 |
| Wasserstein距离 | EM Distance | 改进GAN训练稳定性的度量 |
| 条件生成 | Conditional Generation | 基于条件信息的可控生成 |
| 渐进式生长 | Progressive Growing | 从低分辨率逐步训练到高分辨率 |
| 风格迁移 | Style Transfer | 控制生成图像的风格特征 |

---

## 1. 引言：对抗学习的诞生

生成对抗网络（Generative Adversarial Network, GAN）由 Ian Goodfellow 等人在 2014 年提出，是一种基于博弈论的生成式建模框架。GAN 的核心思想是通过让两个神经网络相互对抗来学习数据分布，这种"左右互搏"的训练范式深刻影响了深度学习的发展方向。

> [!note] 历史意义
> GAN 的提出标志着生成式 AI 进入了一个新纪元。在此之前，自编码器和变分自编码器是主流的生成模型，但 GAN 的出现使得生成图像的质量有了质的飞跃。

---

## 2. Goodfellow 2014 原始GAN

### 2.1 核心架构

原始 GAN 包含两个核心组件：生成器 $G$ 和判别器 $D$。两者通过对抗训练相互提升，最终达到纳什均衡状态。

```python
import torch
import torch.nn as nn
import torch.optim as optim

class Generator(nn.Module):
    """生成器网络：从 latent space 映射到数据空间"""
    def __init__(self, latent_dim, img_shape):
        super(Generator, self).__init__()
        self.img_shape = img_shape
        
        def block(in_feat, out_feat, normalize=True):
            layers = [nn.Linear(in_feat, out_feat)]
            if normalize:
                layers.append(nn.BatchNorm1d(out_feat, 0.8))
            layers.append(nn.LeakyReLU(0.2, inplace=True))
            return layers
        
        self.model = nn.Sequential(
            *block(latent_dim, 128, normalize=False),
            *block(128, 256),
            *block(256, 512),
            *block(512, 1024),
            nn.Linear(1024, int(torch.prod(torch.tensor(img_shape)))),
            nn.Tanh()
        )
    
    def forward(self, z):
        img = self.model(z)
        img = img.view(img.size(0), *self.img_shape)
        return img

class Discriminator(nn.Module):
    """判别器网络：判断样本为真或假的二分类器"""
    def __init__(self, img_shape):
        super(Discriminator, self).__init__()
        
        self.model = nn.Sequential(
            nn.Linear(int(torch.prod(torch.tensor(img_shape))), 512),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Linear(256, 1),
            nn.Sigmoid()
        )
    
    def forward(self, img):
        img_flat = img.view(img.size(0), -1)
        validity = self.model(img_flat)
        return validity
```

### 2.2 目标函数与博弈论本质

GAN 的训练过程可以形式化为一个二人零和博弈：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[\log D(\mathbf{x})] + \mathbb{E}_{\mathbf{z} \sim p_{\mathbf{z}}(\mathbf{z})}[\log(1 - D(G(\mathbf{z})))]
$$

这个目标函数体现了博弈论中的零和博弈思想：判别器 $D$ 试图最大化正确分类的概率，而生成器 $G$ 试图最小化判别器的准确率。

### 2.3 训练流程

```python
def train_gan(generator, discriminator, dataloader, latent_dim, epochs, device):
    """GAN 训练循环"""
    adversarial_loss = torch.nn.BCELoss()
    optimizer_G = optim.Adam(generator.parameters(), lr=0.0002, betas=(0.5, 0.999))
    optimizer_D = optim.Adam(discriminator.parameters(), lr=0.0002, betas=(0.5, 0.999))
    
    for epoch in range(epochs):
        for imgs, _ in dataloader:
            batch_size = imgs.size(0)
            imgs = imgs.to(device)
            
            # 真实标签与假标签
            valid = torch.ones(batch_size, 1).to(device)
            fake = torch.zeros(batch_size, 1).to(device)
            
            # 训练生成器
            optimizer_G.zero_grad()
            z = torch.randn(batch_size, latent_dim).to(device)
            gen_imgs = generator(z)
            g_loss = adversarial_loss(discriminator(gen_imgs), valid)
            g_loss.backward()
            optimizer_G.step()
            
            # 训练判别器
            optimizer_D.zero_grad()
            real_loss = adversarial_loss(discriminator(imgs), valid)
            fake_loss = adversarial_loss(discriminator(gen_imgs.detach()), fake)
            d_loss = (real_loss + fake_loss) / 2
            d_loss.backward()
            optimizer_D.step()
```

---

## 3. JS散度与训练困难

### 3.1 原始GAN的理论分析

从信息论角度，原始 GAN 的目标函数可以解释为最小化生成分布 $p_g$ 与真实分布 $p_{\text{data}}$ 之间的 JS 散度：

$$
\text{JS}(p_{\text{data}} \| p_g) = \frac{1}{2} \text{KL}(p_{\text{data}} \| p_m) + \frac{1}{2} \text{KL}(p_g \| p_m)
$$

其中 $p_m = \frac{p_{\text{data}} + p_g}{2}$ 是混合分布。

### 3.2 JS散度的致命缺陷

JS 散度存在一个根本性问题：当两个分布完全不重叠时，JS 散度趋近于常数 $\log 2$，梯度恒为 0。

```python
# 梯度消失的直观演示
import numpy as np

def js_divergence_gradients():
    """
    当生成分布与真实分布完全不重叠时，
    JS散度的梯度为0，导致训练停滞
    """
    # 假设真实分布为均值=0的高斯，生成分布为均值=10的高斯
    p_data = np.random.normal(0, 1, 10000)
    p_g = np.random.normal(10, 1, 10000)
    
    # 计算JS散度（理论上接近log2 ≈ 0.693）
    # 无论参数如何变化，梯度都接近0
    pass
```

> [!warning] 训练不稳定性
> 当生成分布 $p_g$ 与真实分布 $p_{\text{data}}$ 的支撑集（support）不重叠或重叠可忽略时，判别器最优时 $D^*(\mathbf{x}) = \frac{1}{2}$，此时生成器的梯度消失。

### 3.3 模式崩溃（Mode Collapse）

模式崩溃是 GAN 训练中的另一个核心问题：生成器学会"欺骗"判别器，但只生成少数几种样本，忽略了数据分布的多样性。

```
真实数据分布: 双峰分布
p_data(x) = 0.5 * N(μ=2, σ=1) + 0.5 * N(μ=8, σ=1)

理想生成: 应该生成两个峰的所有样本
模式崩溃: 只生成 μ=2 附近的样本（或只生成 μ=8 附近的）
```

---

## 4. Wasserstein GAN (WGAN)

### 4.1 理论突破

Wasserstein GAN 由 Martin Arjovsky 等人在 2017 年提出，用 Earth-Mover（EM）距离替代 JS 散度：

$$
W(p_{\text{data}}, p_g) = \inf_{\gamma \in \Pi(p_{\text{data}}, p_g)} \mathbb{E}_{(x, y) \sim \gamma} [\|x - y\|]
$$

EM 距离的核心优势：即使两个分布完全不重叠，EM 距离仍然能提供有意义的梯度信号。

### 4.2 WGAN实现

```python
class WGAN_Generator(nn.Module):
    """WGAN 生成器"""
    def __init__(self, latent_dim, img_shape):
        super().__init__()
        self.img_shape = img_shape
        self.model = nn.Sequential(
            nn.Linear(latent_dim, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 1024),
            nn.LeakyReLU(0.2),
            nn.Linear(1024, int(torch.prod(torch.tensor(img_shape)))),
            nn.Tanh()
        )
    
    def forward(self, z):
        img = self.model(z)
        img = img.view(img.size(0), *self.img_shape)
        return img

class WGAN_Discriminator(nn.Module):
    """WGAN 判别器（ critics，无sigmoid输出层）"""
    def __init__(self, img_shape):
        super().__init__()
        self.model = nn.Sequential(
            nn.Linear(int(torch.prod(torch.tensor(img_shape))), 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 1)  # 无激活函数，输出为任意实数
        )
    
    def forward(self, img):
        img_flat = img.view(img.size(0), -1)
        validity = self.model(img_flat)
        return validity

def train_wgan(generator, discriminator, dataloader, latent_dim, epochs, device, clip_value=0.01):
    """WGAN 训练：使用权重裁剪替代梯度惩罚"""
    optimizer_G = optim.RMSprop(generator.parameters(), lr=0.00005)
    optimizer_D = optim.RMSprop(discriminator.parameters(), lr=0.00005)
    
    for epoch in range(epochs):
        for imgs, _ in dataloader:
            batch_size = imgs.size(0)
            imgs = imgs.to(device)
            
            # 训练判别器（多个迭代）
            for _ in range(5):
                optimizer_D.zero_grad()
                z = torch.randn(batch_size, latent_dim).to(device)
                gen_imgs = generator(z)
                
                d_loss = -torch.mean(discriminator(imgs)) + torch.mean(discriminator(gen_imgs))
                d_loss.backward()
                optimizer_D.step()
                
                # 权重裁剪
                for p in discriminator.parameters():
                    p.data.clamp_(-clip_value, clip_value)
            
            # 训练生成器
            optimizer_G.zero_grad()
            z = torch.randn(batch_size, latent_dim).to(device)
            gen_imgs = generator(z)
            g_loss = -torch.mean(discriminator(gen_imgs))
            g_loss.backward()
            optimizer_G.step()
```

---

## 5. 条件GAN（cGAN）

### 5.1 条件生成的理论框架

条件生成对抗网络（Conditional GAN, cGAN）由 Mehdi Mirza 和 Simon Osindero 在 2014 年提出，通过将条件信息 $y$ 引入生成器和判别器：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[\log D(\mathbf{x}|y)] + \mathbb{E}_{\mathbf{z} \sim p_{\mathbf{z}}(\mathbf{z})}[\log(1 - D(G(\mathbf{z}|y)|y))]
$$

### 5.2 cGAN架构实现

```python
class ConditionalGenerator(nn.Module):
    """条件生成器：同时接受噪声和条件信息"""
    def __init__(self, latent_dim, num_classes, img_shape):
        super().__init__()
        self.img_shape = img_shape
        self.label_emb = nn.Embedding(num_classes, num_classes)
        
        # 将噪声和条件标签拼接
        self.model = nn.Sequential(
            nn.Linear(latent_dim + num_classes, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 256),
            nn.BatchNorm1d(256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 512),
            nn.BatchNorm1d(512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 1024),
            nn.BatchNorm1d(1024),
            nn.LeakyReLU(0.2),
            nn.Linear(1024, int(torch.prod(torch.tensor(img_shape)))),
            nn.Tanh()
        )
    
    def forward(self, z, labels):
        # 嵌入标签并与噪声拼接
        label_embedding = self.label_emb(labels)
        gen_input = torch.cat((z, label_embedding), dim=-1)
        img = self.model(gen_input)
        img = img.view(img.size(0), *self.img_shape)
        return img

class ConditionalDiscriminator(nn.Module):
    """条件判别器：同时接受图像和条件信息"""
    def __init__(self, num_classes, img_shape):
        super().__init__()
        self.img_shape = img_shape
        self.label_emb = nn.Embedding(num_classes, int(torch.prod(torch.tensor(img_shape))))
        
        self.model = nn.Sequential(
            nn.Linear(int(torch.prod(torch.tensor(img_shape))) + num_classes, 512),
            nn.LeakyReLU(0.2),
            nn.Dropout(0.4),
            nn.Linear(512, 512),
            nn.LeakyReLU(0.2),
            nn.Dropout(0.4),
            nn.Linear(512, 1),
            nn.Sigmoid()
        )
    
    def forward(self, img, labels):
        label_embedding = self.label_emb(labels)
        d_in = torch.cat((img.view(img.size(0), -1), label_embedding), dim=-1)
        validity = self.model(d_in)
        return validity
```

---

## 6. Progressive GAN 与 StyleGAN 系列

### 6.1 Progressive Growing GAN

Progressive GAN（ProGAN）由 Karras 等人在 2018 年提出，核心思想是从低分辨率开始逐步增加网络深度：

```
训练阶段:
Level 1: 4×4 分辨率 ─────────────────────────────────────────────> 稳定
Level 2: 8×8 分辨率 ─────────────────────────────────────────────> 稳定
Level 3: 16×16 分辨率 ─────────────────────────────────────────────> 稳定
Level 4: 32×32 分辨率 ─────────────────────────────────────────────> 稳定
Level 5: 64×64 分辨率 ─────────────────────────────────────────────> 稳定
...
Level N: 1024×1024 分辨率 ─────────────────────────────────────> 稳定
```

### 6.2 StyleGAN：风格控制的革命

StyleGAN 在 ProGAN 基础上引入自适应实例归一化（AdaIN），实现对生成图像风格的控制：

$$
\text{AdaIN}(x_i, y) = y_{s,i} \cdot \frac{x_i - \mu(x_i)}{\sigma(x_i)} + y_{b,i}
$$

```python
class StyleBlock(nn.Module):
    """StyleGAN 的核心风格控制模块"""
    def __init__(self, in_channels, style_dim):
        super().__init__()
        self.conv = nn.Conv2d(in_channels, in_channels, 3, padding=1)
        self.affine = nn.Linear(style_dim, in_channels * 2)
    
    def forward(self, x, style):
        """
        x: 特征图
        style: 风格向量（来自映射网络）
        """
        # 从风格向量预测缩放和偏移参数
        style_params = self.affine(style)
        scale, bias = style_params.chunk(2, dim=-1)
        
        # 自适应实例归一化
        x = (x - x.mean(dim=[2, 3], keepdim=True)) / (x.std(dim=[2, 3], keepdim=True) + 1e-5)
        x = x * (scale.view(scale.size(0), -1, 1, 1) + 1) + bias.view(bias.size(0), -1, 1, 1)
        return self.conv(x)
```

### 6.3 StyleGAN3：消除混叠伪影

StyleGAN3 在 2021 年提出，通过消除特征图的离散采样效应，实现了图像的连续平移和旋转不变性：

> [!tip] StyleGAN3 的核心改进
> - 使用低通滤波器确保信号在网络传播中保持连续性
> - 重新设计上采样和下采样操作
> - 实现了真正的等变性而非近似等变性

---

## 7. 学术引用与参考文献

1. Goodfellow, I., et al. (2014). "Generative Adversarial Networks." *NeurIPS*.
2. Arjovsky, M., Chintala, S., & Bottou, L. (2017). "Wasserstein GAN." *ICML*.
3. Radford, A., Metz, L., & Chintala, S. (2016). "Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks." *ICLR*.
4. Mirza, M., & Osindero, S. (2014). "Conditional Generative Adversarial Nets." *arXiv*.
5. Karras, T., et al. (2018). "Progressive Growing of GANs for Improved Quality, Stability, and Variation." *ICLR*.
6. Karras, T., et al. (2019). "A Style-Based Generator Architecture for Generative Adversarial Networks." *CVPR*.
7. Karras, T., et al. (2021). "Alias-Free Generative Adversarial Networks." *NeurIPS*.

---

## 8. 相关文档

- [[GAN变体详解]] - 深入了解 CGAN、Pix2Pix、CycleGAN、BigGAN 等变体
- [[对抗样本深度指南]] - 对抗样本的定义与攻击方法
- [[博弈论与AI]] - 理解 GAN 的博弈论基础
- [[对抗训练与鲁棒性]] - 对抗训练提升模型鲁棒性
- [[多智能体博弈详解]] - 多智能体系统中的对抗学习
