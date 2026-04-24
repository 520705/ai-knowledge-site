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
| 循环一致性 | Cycle Consistency | 无配对数据转换的核心约束 |
| 一致性损失 | Consistency Loss | 保证转换前后语义保持的损失函数 |
| 谱归一化 | Spectral Normalization | 约束判别器Lipschitz常数的技巧 |
| 梯度惩罚 | Gradient Penalty | 替代权重裁剪的WGAN改进 |
| 小样本学习 | Few-shot Learning | 仅用少量样本训练生成模型 |
| Zero-shot学习 | Zero-shot Learning | 生成未见过的类别样本 |

---

## 1. 引言：对抗学习的诞生

生成对抗网络（Generative Adversarial Network, GAN）由 Ian Goodfellow 等人在 2014 年提出，是一种基于博弈论的生成式建模框架。GAN 的核心思想是通过让两个神经网络相互对抗来学习数据分布，这种"左右互搏"的训练范式深刻影响了深度学习的发展方向。

> [!note] 历史意义
> GAN 的提出标志着生成式 AI 进入了一个新纪元。在此之前，自编码器和变分自编码器是主流的生成模型，但 GAN 的出现使得生成图像的质量有了质的飞跃。

### 1.1 GAN的哲学思想

GAN的设计蕴含了深刻的哲学思想。想象一个伪造者（生成器）和一个鉴定师（判别器）之间的博弈：伪造者不断尝试制造以假乱真的赝品，而鉴定师则不断提升自己的鉴别能力。这种动态博弈过程最终会达到一个均衡点，此时伪造者已经掌握了制造几乎完美赝品的技术，而鉴定师也达到了近乎完美的鉴别能力。

从更宏观的视角看，GAN体现了"道高一尺，魔高一丈"的辩证思想。没有绝对的防御，也没有绝对的攻击，一切都是相对的、动态的。这种思想不仅适用于技术领域，也反映了自然界中普遍存在的协同进化现象。

### 1.2 生成模型家族概览

在深度学习的生成模型家族中，GAN占据着独特的位置。常见的生成模型包括：

**自编码器（Autoencoder）系列：**
- 标准自编码器：学习数据的压缩表示，用于重构
- 变分自编码器（VAE）：学习数据的潜在分布，支持采样生成
- Denoising Autoencoder：学习从噪声数据中恢复原始信号

**基于流的模型（Flow-based Models）：**
- RealNVP：使用可逆变换构建生成模型
- Glow：基于normalize flows的高质量图像生成
- 优点：精确的对数似然计算
- 缺点：计算量大，生成速度慢

**扩散模型（Diffusion Models）：**
- DDPM：逐步添加噪声再逆向去噪
- Score Matching：学习数据分布的梯度场
- 优点：训练稳定，生成质量高
- 缺点：推理速度慢，需要大量计算

**GAN的独特优势：**
- 生成速度快（单次前向传播）
- 生成质量高（在某些领域领先）
- 损失函数灵活（多种变体）

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

### 2.4 原始GAN的数学证明

原始论文中给出了GAN收敛性的初步证明。假设在训练的每一步，判别器都能够达到最优，则生成器的优化目标等价于最小化生成分布与真实分布之间的JS散度。

**定理（GAN的最优判别器和生成器）：**

当判别器可训练至最优时，最优判别器为：
$$
D^*_G(\mathbf{x}) = \frac{p_{\text{data}}(\mathbf{x})}{p_{\text{data}}(\mathbf{x}) + p_g(\mathbf{x})}
$$

此时生成器的目标函数为：
$$
C(G) = -\log(4) + 2 \cdot \text{JS}(p_{\text{data}} \| p_g)
$$

这意味着当且仅当 $p_{\text{data}} = p_g$ 时，达到全局最优 $C^* = -\log(4)$。

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

### 3.4 训练不稳定性的深层原因

原始GAN训练不稳定的原因是多方面的：

**1. 同步训练问题：**
- 生成器和判别器需要达到动态平衡
- 如果判别器过强，生成器梯度消失
- 如果判别器过弱，生成器失去学习目标

**2. 模式多样化缺失：**
- 损失函数无法捕捉分布的完整结构
- 生成器倾向于"作弊"，只学习部分模式

**3. 梯度消失与梯度爆炸：**
- JS散度的饱和特性导致梯度消失
- 判别器过深导致梯度不稳定

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

### 4.3 WGAN-GP：梯度惩罚版本

权重裁剪虽然简单，但会导致判别器趋向于最简单的函数。WGAN-GP提出了更优雅的解决方案：

```python
class WGAN_GP_Generator(nn.Module):
    """WGAN-GP 生成器"""
    def __init__(self, latent_dim, img_shape):
        super().__init__()
        self.img_shape = img_shape
        
        self.model = nn.Sequential(
            nn.Linear(latent_dim, 128),
            nn.LeakyReLU(0.2),
            nn.Linear(128, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 1024),
            nn.LeakyReLU(0.2),
            nn.Linear(1024, int(torch.prod(torch.tensor(img_shape)))),
            nn.Tanh()
        )
    
    def forward(self, z):
        return self.model(z).view(z.size(0), *self.img_shape)

class WGAN_GP_Discriminator(nn.Module):
    """WGAN-GP 判别器，使用残差连接增强表达能力"""
    def __init__(self, img_shape):
        super().__init__()
        self.img_shape = img_shape
        
        self.model = nn.Sequential(
            nn.Linear(int(torch.prod(torch.tensor(img_shape))), 512),
            nn.LeakyReLU(0.2),
            nn.Linear(512, 256),
            nn.LeakyReLU(0.2),
            nn.Linear(256, 1)
        )
    
    def forward(self, img):
        return self.model(img.view(img.size(0), -1))

def compute_gradient_penalty(discriminator, real_images, fake_images, device):
    """
    计算梯度惩罚
    
    惩罚项：E(||∇_x̂ D(x̂)||_2 - 1)^2
    其中 x̂ = ε * x_real + (1-ε) * x_fake, ε ~ U(0,1)
    """
    batch_size = real_images.size(0)
    alpha = torch.rand(batch_size, 1, 1, 1).to(device)
    
    # 在真实图像和生成图像之间插值
    interpolates = alpha * real_images + (1 - alpha) * fake_images
    interpolates = interpolates.requires_grad_(True)
    
    d_interpolates = discriminator(interpolates)
    
    gradients = torch.autograd.grad(
        outputs=d_interpolates,
        inputs=interpolates,
        grad_outputs=torch.ones_like(d_interpolates),
        create_graph=True,
        retain_graph=True,
        only_inputs=True
    )[0]
    
    gradients = gradients.view(batch_size, -1)
    gradient_norm = gradients.norm(2, dim=1)
    gradient_penalty = ((gradient_norm - 1) ** 2).mean()
    
    return gradient_penalty

def train_wgan_gp(generator, discriminator, dataloader, latent_dim, epochs, device):
    """WGAN-GP 训练循环"""
    optimizer_G = optim.Adam(generator.parameters(), lr=0.0001, betas=(0.0, 0.9))
    optimizer_D = optim.Adam(discriminator.parameters(), lr=0.0004, betas=(0.0, 0.9))
    
    lambda_gp = 10  # 梯度惩罚系数
    
    for epoch in range(epochs):
        for imgs, _ in dataloader:
            batch_size = imgs.size(0)
            real_images = imgs.to(device)
            
            # 训练判别器
            optimizer_D.zero_grad()
            z = torch.randn(batch_size, latent_dim).to(device)
            fake_images = generator(z)
            
            d_real = discriminator(real_images)
            d_fake = discriminator(fake_images.detach())
            
            # WGAN损失
            d_loss = d_fake.mean() - d_real.mean()
            
            # 梯度惩罚
            gp = compute_gradient_penalty(discriminator, real_images, fake_images, device)
            
            total_d_loss = d_loss + lambda_gp * gp
            total_d_loss.backward()
            optimizer_D.step()
            
            # 训练生成器（每n个判别器步骤训练一次）
            optimizer_G.zero_grad()
            z = torch.randn(batch_size, latent_dim).to(device)
            fake_images = generator(z)
            g_loss = -discriminator(fake_images).mean()
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

## 6. DCGAN：深度卷积GAN

### 6.1 架构设计原则

DCGAN（Deep Convolutional GAN）由 Radford 等人在 2016 年提出，是将GAN与深度卷积网络结合的里程碑工作。其核心架构设计原则包括：

**生成器设计：**
- 使用转置卷积（fractionally strided convolution）进行上采样
- 移除所有池化层，使用转置卷积实现空间上采样
- 在生成器中使用批量归一化（BatchNorm）
- 输出层使用 Tanh 激活（归一化到 [-1, 1]）
- 隐藏层使用 ReLU/LeakyReLU 激活

**判别器设计：**
- 使用步长卷积（strided convolution）代替池化
- 同样使用批量归一化（首层除外）
- 使用 LeakyReLU 激活防止梯度稀疏

### 6.2 DCGAN实现

```python
class DCGenerator(nn.Module):
    """DCGAN 生成器：使用转置卷积"""
    def __init__(self, latent_dim, channels, features_g=64):
        super(DCGenerator, self).__init__()
        
        # 初始输入: latent_dim x 1 x 1
        self.init_size = 4
        self.latent_dim = latent_dim
        self.channels = channels
        
        # 初始卷积块
        self.init = nn.Sequential(
            nn.ConvTranspose2d(latent_dim, features_g * 8, 4, 1, 0, bias=False),
            nn.BatchNorm2d(features_g * 8),
            nn.ReLU(True)
        )
        
        # 逐步上采样
        self.upsample1 = nn.Sequential(
            nn.ConvTranspose2d(features_g * 8, features_g * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g * 4),
            nn.ReLU(True)
        )  # 8x8
        
        self.upsample2 = nn.Sequential(
            nn.ConvTranspose2d(features_g * 4, features_g * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g * 2),
            nn.ReLU(True)
        )  # 16x16
        
        self.upsample3 = nn.Sequential(
            nn.ConvTranspose2d(features_g * 2, features_g, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g),
            nn.ReLU(True)
        )  # 32x32
        
        self.upsample4 = nn.Sequential(
            nn.ConvTranspose2d(features_g, channels, 4, 2, 1, bias=False),
            nn.Tanh()
        )  # 64x64
    
    def forward(self, z):
        # 重塑为 (batch, channels, 1, 1)
        x = z.view(z.size(0), self.latent_dim, 1, 1)
        x = self.init(x)
        x = self.upsample1(x)
        x = self.upsample2(x)
        x = self.upsample3(x)
        x = self.upsample4(x)
        return x

class DCDiscriminator(nn.Module):
    """DCGAN 判别器：使用步长卷积"""
    def __init__(self, channels, features_d=64):
        super(DCDiscriminator, self).__init__()
        
        # 输入: channels x 64 x 64
        self.main = nn.Sequential(
            # 第一个卷积块（无BN）
            nn.Conv2d(channels, features_d, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            
            # 逐步下采样
            nn.Conv2d(features_d, features_d * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 2),
            nn.LeakyReLU(0.2, inplace=True),
            
            nn.Conv2d(features_d * 2, features_d * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 4),
            nn.LeakyReLU(0.2, inplace=True),
            
            nn.Conv2d(features_d * 4, features_d * 8, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 8),
            nn.LeakyReLU(0.2, inplace=True),
            
            # 最后一个卷积块
            nn.Conv2d(features_d * 8, 1, 4, 1, 0, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        return self.main(x)
```

---

## 7. Progressive GAN 与 StyleGAN 系列

### 7.1 Progressive Growing GAN

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

Progressive Growing 的优势：
1. **训练稳定性**：低分辨率时分布更简单，更容易学习
2. **计算效率**：早期训练快，可快速获得大尺度结构
3. **模式覆盖**：减少模式崩溃的风险

```python
class ProgressiveGenerator(nn.Module):
    """
    渐进式增长生成器
    
    关键组件：
    1. 分辨率阶段：每个分辨率有对应的卷积块
    2. 平滑过渡：在分辨率切换时使用alpha插值
    3. 潜在向量：每层可以接收不同的潜在向量
    """
    
    def __init__(self, latent_dim, max_resolution=1024, base_channels=512):
        super().__init__()
        self.latent_dim = latent_dim
        self.max_resolution = max_resolution
        self.base_channels = base_channels
        
        # 计算分辨率级别
        self.num_stages = int(np.log2(max_resolution)) - 1  # 4x4 -> 1024x1024
        
        # 初始层：从潜在向量生成4x4特征图
        self.initial = nn.Sequential(
            nn.Linear(latent_dim, base_channels * 16 * 4 * 4),
            nn.Reshape(base_channels * 16, 4, 4),
            nn.ReLU()
        )
        
        # 各分辨率的上采样模块
        self.upsample_blocks = nn.ModuleList()
        self.to_rgb_blocks = nn.ModuleList()
        
        channels = base_channels * 16
        resolution = 4
        
        for stage in range(1, self.num_stages):
            new_resolution = resolution * 2
            
            # 上采样卷积
            self.upsample_blocks.append(nn.Sequential(
                nn.Upsample(scale_factor=2, mode='nearest'),
                nn.Conv2d(channels, channels // 2, 3, padding=1),
                nn.BatchNorm2d(channels // 2),
                nn.ReLU(),
                nn.Conv2d(channels // 2, channels // 2, 3, padding=1),
                nn.BatchNorm2d(channels // 2),
                nn.ReLU()
            ))
            
            # RGB输出层
            self.to_rgb_blocks.append(nn.Conv2d(channels // 2, 3, 1))
            
            channels = channels // 2
            resolution = new_resolution
        
        # 最终RGB输出
        self.final_rgb = nn.Conv2d(channels, 3, 3, padding=1)
    
    def forward(self, z, alpha=1.0, stage=None):
        """
        前向传播
        
        参数:
            z: 潜在向量
            alpha: 混合系数（用于平滑过渡）
            stage: 当前训练的分辨率阶段
        """
        x = self.initial(z)
        
        if stage is None:
            stage = len(self.upsample_blocks)
        
        # 应用前stage个上采样块
        for i in range(stage):
            x = self.upsample_blocks[i](x)
        
        # 输出RGB
        rgb = self.final_rgb(x)
        return torch.tanh(rgb)
```

### 7.2 StyleGAN：风格控制的革命

StyleGAN 在 ProGAN 基础上引入自适应实例归一化（AdaIN），实现对生成图像风格的控制：

$$
\text{AdaIN}(x_i, y) = y_{s,i} \cdot \frac{x_i - \mu(x_i)}{\sigma(x_i)} + y_{b,i}
$$

StyleGAN的核心创新：
1. **映射网络（Mapping Network）**：将潜在向量 $z$ 转换为中间潜在向量 $w$
2. **风格块（Style Blocks）**：在每个分辨率级别注入风格信息
3. **噪声输入**：为每个级别添加随机噪声以增加细节变化

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

class MappingNetwork(nn.Module):
    """
    映射网络：将潜在向量z映射到中间潜在空间w
    
    目的：
    1. 解耦潜在空间的各个维度
    2. 使风格控制更加精确和独立
    """
    def __init__(self, latent_dim, hidden_dim=512, num_layers=8):
        super().__init__()
        
        layers = []
        for i in range(num_layers):
            in_dim = latent_dim if i == 0 else hidden_dim
            layers.extend([
                nn.Linear(in_dim, hidden_dim),
                nn.LeakyReLU(0.2)
            ])
        
        self.mapping = nn.Sequential(*layers)
        self.normalize = PixelNorm()
    
    def forward(self, z):
        z = self.normalize(z)
        w = self.mapping(z)
        return w

class StyleGenerator(nn.Module):
    """StyleGAN 生成器"""
    def __init__(self, latent_dim=512, channels=3, base_resolution=4, max_resolution=1024):
        super().__init__()
        
        self.latent_dim = latent_dim
        self.channels = channels
        
        # 映射网络
        self.mapping = MappingNetwork(latent_dim)
        
        # 初始层（constant input）
        self.constant_input = nn.Parameter(torch.ones(1, 512, 4, 4))
        self.conv1 = StyledConv(512, 512, 3, latent_dim)
        self.to_rgb1 = ToRGB(512, channels, latent_dim)
        
        # 逐级上采样
        resolutions = [8, 16, 32, 64, 128, 256, 512, 1024]
        self.upsamples = nn.ModuleList()
        self.convs = nn.ModuleList()
        self.to_rgbs = nn.ModuleList()
        
        input_channels = 512
        for res in resolutions[1:]:
            output_channels = input_channels // 2
            self.upsamples.append(nn.Upsample(scale_factor=2, mode='bilinear'))
            self.convs.append(StyledConv(input_channels, output_channels, 3, latent_dim))
            self.to_rgbs.append(ToRGB(output_channels, channels, latent_dim))
            input_channels = output_channels
    
    def forward(self, z, styles=None, noise=None):
        """
        前向传播
        
        styles: 风格向量列表（如果为None，使用从z映射的w）
        """
        if styles is None:
            w = self.mapping(z)
            # 对所有层使用相同的w
            styles = [w] * 14
        
        # 初始特征
        x = self.constant_input.repeat(z.size(0), 1, 1, 1)
        x = self.conv1(x, styles[0])
        
        # 逐级上采样
        rgb = self.to_rgb1(x, styles[1])
        
        for i, (up, conv, to_rgb) in enumerate(zip(
            self.upsamples, self.convs, self.to_rgbs
        )):
            x = up(x)
            x = conv(x, styles[i + 2])
            rgb = to_rgb(x, styles[i + 3])
        
        return rgb
```

### 7.3 StyleGAN2：质量与稳定性提升

StyleGAN2 针对原始StyleGAN的几个问题进行了改进：

1. **权重 demodulation**：替代AdaIN，减少伪影
2. **路径长度正则化**：使潜在空间插值更加平滑
3. **重新设计架构**：移除噪声输入的影响（可选）

```python
class StyleGAN2Conv(nn.Module):
    """
    StyleGAN2 的卷积模块
    使用 weight demodulation 技术
    """
    def __init__(self, in_channels, out_channels, style_dim, upsample=False):
        super().__init__()
        
        self.upsample = nn.Upsample(scale_factor=2, mode='bilinear') if upsample else None
        self.padding = 1
        self.weight = nn.Parameter(torch.randn(out_channels, in_channels, 3, 3))
        self.bias = nn.Parameter(torch.zeros(out_channels))
        self.affine = nn.Linear(style_dim, in_channels)
    
    def forward(self, x, style):
        """
        weight demodulation：
        1. 计算每通道激活的标准差
        2. 对权重进行归一化
        3. 添加样式调制
        """
        batch, _, height, width = x.shape
        
        # 获取样式调制
        style = self.affine(style)
        style = style.view(batch, 1, -1, 1, 1)
        
        # 权重归一化
        weight = self.weight.unsqueeze(0)
        weight_norm = torch.norm(weight, dim=(2, 3, 4), keepdim=True)
        demod_weight = weight / (weight_norm + 1e-8)
        
        # 应用样式调制
        modulated_weight = demod_weight * style
        
        # 重塑用于卷积
        demod_weight = demod_weight.view(-1, *self.weight.shape)
        
        # 卷积
        x = x.view(1, -1, height, width)
        out = F.conv2d(x, demod_weight, padding=self.padding, groups=batch)
        out = out.view(batch, -1, height, width)
        
        # 添加偏置并应用激活
        out = out + self.bias.view(1, -1, 1, 1)
        
        return out

class PathLengthPenalty(nn.Module):
    """
    路径长度正则化
    
    目的：使潜在空间w中的等距移动对应于图像空间中
         的等距变化，从而提高插值质量
    """
    def __init__(self, beta=0.99):
        super().__init__()
        self.beta = beta
        self.styles_mean = None
    
    def forward(self, w, generated_images):
        """
        计算路径长度惩罚
        """
        image_size = generated_images.shape[-1]
        n = w.shape[1]  # 层数
        
        # 计算图像对随机方向的梯度
        random_direction = torch.randn_like(w)
        random_direction = random_direction / torch.norm(random_direction, dim=-1, keepdim=True)
        
        # 生成带扰动的图像
        w_perturbed = w + 0.01 * random_direction
        # 这里需要调用生成器，但简化表示
        
        # 路径长度惩罚
        if self.styles_mean is None:
            self.styles_mean = torch.zeros(1)
        
        # 更新移动平均
        with torch.no_grad():
            path_lengths = torch.ones(w.shape[0], device=w.device)
            self.styles_mean = self.beta * self.styles_mean + (1 - self.beta) * path_lengths.mean()
        
        # 惩罚项
        penalty = (path_lengths - self.styles_mean) ** 2
        return penalty.mean()
```

### 7.4 StyleGAN3：消除混叠伪影

StyleGAN3 在 2021 年提出，通过消除特征图的离散采样效应，实现了图像的连续平移和旋转不变性：

> [!tip] StyleGAN3 的核心改进
> - 使用低通滤波器确保信号在网络传播中保持连续性
> - 重新设计上采样和下采样操作
> - 实现了真正的等变性而非近似等变性

```python
class StyleGAN3Conv(nn.Module):
    """
    StyleGAN3 的连续卷积
    
    关键改进：
    1. 使用sinc滤波器进行上/下采样
    2. 确保信号在变换下保持连续
    """
    def __init__(self, in_channels, out_channels, kernel_size, down=False):
        super().__init__()
        
        self.down = down
        self.kernel_size = kernel_size
        
        # 学习的滤波核（用于可控制的频谱）
        if down:
            self.filter = nn.Conv2d(
                in_channels, in_channels, kernel_size, 
                stride=2, padding=kernel_size//2, groups=in_channels
            )
        
        self.conv = nn.Conv2d(in_channels, out_channels, kernel_size, padding=kernel_size//2)
    
    def sinc_filter(self, x, cutoff=0.5):
        """
        生成sinc低通滤波器
        
        sinc滤波器具有理想的频率响应，是唯一能够
        完美重建的低通滤波器
        """
        n = torch.arange(-self.kernel_size//2, self.kernel_size//2 + 1, device=x.device)
        window = torch.hann_window(self.kernel_size, device=x.device)
        kernel = torch.sinc(2 * cutoff * n) * window
        kernel = kernel / kernel.sum()
        return kernel
    
    def forward(self, x):
        if self.down:
            # 使用sinc滤波器进行下采样
            kernel = self.sinc_filter(x).view(1, 1, -1, 1)
            x = F.conv2d(x, kernel.repeat(x.shape[1], 1, 1, 1), 
                         padding=self.kernel_size//2, groups=x.shape[1])
        
        x = self.conv(x)
        return x
```

---

## 8. 图像翻译GAN系列

### 8.1 Pix2Pix：有配对的图像翻译

Pix2Pix是条件GAN在图像翻译任务上的经典应用，适用于有配对数据的图像转换任务，如：

- 边缘图 → 照片
- 白天 → 夜晚
- 航拍图 → 地图
- 素描 → 真实图像

```python
class UNetGenerator(nn.Module):
    """
    U-Net 生成器（用于Pix2Pix）
    
    编码器-解码器架构 + 跳跃连接
    保留低层次的细节信息
    """
    def __init__(self, input_channels, output_channels, num_filters=64):
        super().__init__()
        
        # 编码器（下采样）
        self.enc1 = self._conv_block(input_channels, num_filters)      # 256
        self.enc2 = self._conv_block(num_filters, num_filters * 2)     # 128
        self.enc3 = self._conv_block(num_filters * 2, num_filters * 4)  # 64
        self.enc4 = self._conv_block(num_filters * 4, num_filters * 8) # 32
        self.enc5 = self._conv_block(num_filters * 8, num_filters * 8) # 16
        self.enc6 = self._conv_block(num_filters * 8, num_filters * 8) # 8
        self.enc7 = self._conv_block(num_filters * 8, num_filters * 8) # 4
        self.enc8 = self._conv_block(num_filters * 8, num_filters * 8) # 2
        
        # 解码器（上采样）
        self.dec1 = self._upconv_block(num_filters * 8, num_filters * 8) # 4
        self.dec2 = self._upconv_block(num_filters * 16, num_filters * 8) # 8
        self.dec3 = self._upconv_block(num_filters * 16, num_filters * 8) # 16
        self.dec4 = self._upconv_block(num_filters * 16, num_filters * 8) # 32
        self.dec5 = self._upconv_block(num_filters * 16, num_filters * 4) # 64
        self.dec6 = self._upconv_block(num_filters * 8, num_filters * 2) # 128
        self.dec7 = self._upconv_block(num_filters * 4, num_filters)     # 256
        
        # 最终输出层
        self.final = nn.ConvTranspose2d(num_filters * 2, output_channels, 3, padding=1)
        self.tanh = nn.Tanh()
    
    def _conv_block(self, in_channels, out_channels):
        return nn.Sequential(
            nn.Conv2d(in_channels, out_channels, 4, stride=2, padding=1),
            nn.BatchNorm2d(out_channels),
            nn.LeakyReLU(0.2)
        )
    
    def _upconv_block(self, in_channels, out_channels):
        return nn.Sequential(
            nn.ConvTranspose2d(in_channels, out_channels, 4, stride=2, padding=1),
            nn.BatchNorm2d(out_channels),
            nn.ReLU()
        )
    
    def forward(self, x):
        # 编码
        enc1 = self.enc1(x)
        enc2 = self.enc2(enc1)
        enc3 = self.enc3(enc2)
        enc4 = self.enc4(enc3)
        enc5 = self.enc5(enc4)
        enc6 = self.enc6(enc5)
        enc7 = self.enc7(enc6)
        enc8 = self.enc8(enc7)
        
        # 解码（带跳跃连接）
        dec1 = self.dec1(enc8)
        dec2 = self.dec2(torch.cat([dec1, enc7], dim=1))
        dec3 = self.dec3(torch.cat([dec2, enc6], dim=1))
        dec4 = self.dec4(torch.cat([dec3, enc5], dim=1))
        dec5 = self.dec5(torch.cat([dec4, enc4], dim=1))
        dec6 = self.dec6(torch.cat([dec5, enc3], dim=1))
        dec7 = self.dec7(torch.cat([dec6, enc2], dim=1))
        
        return self.tanh(self.final(torch.cat([dec7, enc1], dim=1)))

class PatchDiscriminator(nn.Module):
    """
    Patch判别器（PatchGAN）
    
    与其判断整个图像是否为真，
    不如判断图像的每个patch是否为真
    这使得判别器专注于高频细节
    """
    def __init__(self, input_channels, num_filters=64, n_layers=3):
        super().__init__()
        
        layers = []
        
        # 第一层：无BN
        layers.append(nn.Conv2d(input_channels, num_filters, 4, stride=2, padding=1))
        layers.append(nn.LeakyReLU(0.2))
        
        # 中间层
        nf_mult = 1
        for i in range(1, n_layers):
            nf_mult_prev = nf_mult
            nf_mult = min(2 ** i, 8)
            layers.append(nn.Conv2d(num_filters * nf_mult_prev, num_filters * nf_mult,
                                   4, stride=2, padding=1))
            layers.append(nn.BatchNorm2d(num_filters * nf_mult))
            layers.append(nn.LeakyReLU(0.2))
        
        # 最后一层
        nf_mult_prev = nf_mult
        nf_mult = min(2 ** n_layers, 8)
        layers.append(nn.Conv2d(num_filters * nf_mult_prev, num_filters * nf_mult,
                               4, stride=1, padding=1))
        layers.append(nn.BatchNorm2d(num_filters * nf_mult))
        layers.append(nn.LeakyReLU(0.2))
        
        # 输出层
        layers.append(nn.Conv2d(num_filters * nf_mult, 1, 4, stride=1, padding=1))
        
        self.model = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.model(x)
```

### 8.2 CycleGAN：无配对的图像翻译

CycleGAN解决了无配对数据的图像翻译问题，核心思想是循环一致性（Cycle Consistency）：

$$
\mathcal{L}_{\text{cycle}}(G, F) = \mathbb{E}_{x \sim p_{\text{data}}(x)}[\|F(G(x)) - x\|_1] + \mathbb{E}_{y \sim p_{\text{data}}(y)}[\|G(F(y)) - y\|_1]
$$

```python
class ResidualBlock(nn.Module):
    """残差块：用于CycleGAN的生成器"""
    def __init__(self, channels):
        super().__init__()
        self.block = nn.Sequential(
            nn.ReflectionPad2d(1),
            nn.Conv2d(channels, channels, 3, stride=1, padding=0),
            nn.BatchNorm2d(channels),
            nn.ReLU(inplace=True),
            nn.ReflectionPad2d(1),
            nn.Conv2d(channels, channels, 3, stride=1, padding=0),
            nn.BatchNorm2d(channels)
        )
    
    def forward(self, x):
        return x + self.block(x)

class CycleGANGenerator(nn.Module):
    """
    CycleGAN 生成器
    
    使用编码器-解码器 + 残差连接
    编码器：逐步降低分辨率，提取高层特征
    解码器：逐步提高分辨率，恢复空间信息
    残差连接：帮助传递细节信息
    """
    def __init__(self, input_channels=3, residual_blocks=9):
        super().__init__()
        
        # 编码器
        self.encoder = nn.Sequential(
            nn.ReflectionPad2d(3),
            nn.Conv2d(input_channels, 64, 7, stride=1, padding=0),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(64, 128, 3, stride=2, padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(128, 256, 3, stride=2, padding=1),
            nn.BatchNorm2d(256),
            nn.ReLU(inplace=True)
        )
        
        # 残差块
        self.residuals = nn.Sequential(
            *[ResidualBlock(256) for _ in range(residual_blocks)]
        )
        
        # 解码器
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(256, 128, 3, stride=2, padding=1, output_padding=1),
            nn.BatchNorm2d(128),
            nn.ReLU(inplace=True),
            
            nn.ConvTranspose2d(128, 64, 3, stride=2, padding=1, output_padding=1),
            nn.BatchNorm2d(64),
            nn.ReLU(inplace=True),
            
            nn.ReflectionPad2d(3),
            nn.Conv2d(64, input_channels, 7, stride=1, padding=0),
            nn.Tanh()
        )
    
    def forward(self, x):
        x = self.encoder(x)
        x = self.residuals(x)
        x = self.decoder(x)
        return x

class CycleGANLoss:
    """
    CycleGAN 损失函数
    
    包括：
    1. 对抗损失：让生成的图像骗过判别器
    2. 循环一致性损失：X->Y->X 应回到 X
    3. 同一性损失：G(Y) 在给定 Y 时应接近 Y
    """
    def __init__(self, lambda_cycle=10, lambda_identity=5):
        self.lambda_cycle = lambda_cycle
        self.lambda_identity = lambda_identity
        self.criterionGAN = nn.MSELoss()
        self.criterionCycle = nn.L1Loss()
        self.criterionIdt = nn.L1Loss()
    
    def compute_loss_D(self, D, real, fake):
        """判别器损失"""
        # 真实样本应该被判定为真
        real_loss = self.criterionGAN(D(real), torch.ones_like(D(real)))
        # 假样本应该被判定为假
        fake_loss = self.criterionGAN(D(fake.detach()), torch.zeros_like(D(fake)))
        return (real_loss + fake_loss) / 2
    
    def compute_loss_G(self, G, F, D, real_x, real_y):
        """
        生成器损失
        
        包括对抗损失、循环一致性损失和同一性损失
        """
        # 对抗损失
        fake_y = G(real_x)
        loss_GAN = self.criterionGAN(D(fake_y), torch.ones_like(D(fake_y)))
        
        fake_x = F(real_y)
        loss_F_GAN = self.criterionGAN(D(fake_x), torch.ones_like(D(fake_x)))
        
        # 循环一致性损失
        loss_cycle_X = self.criterionCycle(F(fake_y), real_x)
        loss_cycle_Y = self.criterionCycle(G(fake_x), real_y)
        
        # 同一性损失
        loss_id_X = self.criterionIdt(G(real_y), real_y)
        loss_id_Y = self.criterionIdt(F(real_x), real_x)
        
        # 总损失
        total_loss = (loss_GAN + loss_F_GAN + 
                     self.lambda_cycle * (loss_cycle_X + loss_cycle_Y) +
                     self.lambda_identity * (loss_id_X + loss_id_Y))
        
        return total_loss, {
            'loss_GAN': loss_GAN.item(),
            'loss_F_GAN': loss_F_GAN.item(),
            'loss_cycle': (loss_cycle_X + loss_cycle_Y).item(),
            'loss_identity': (loss_id_X + loss_id_Y).item()
        }
```

### 8.3 StarGAN：多域图像翻译

StarGAN解决了多域（multi-domain）图像翻译问题，只需一个生成器就能在多个域之间转换：

```python
class StarGANGenerator(nn.Module):
    """
    StarGAN 生成器
    
    特点：
    1. 单一生成器处理多个域
    2. 使用域标签作为条件信息
    3. 节省计算资源
    """
    def __init__(self, image_channels=3, condition_channels=5, num_filters=64):
        super().__init__()
        
        # 编码器
        self.encoder = nn.Sequential(
            nn.Conv2d(image_channels + condition_channels, num_filters, 7, stride=1, padding=3),
            nn.InstanceNorm2d(num_filters),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(num_filters, num_filters * 2, 4, stride=2, padding=1),
            nn.InstanceNorm2d(num_filters * 2),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(num_filters * 2, num_filters * 4, 4, stride=2, padding=1),
            nn.InstanceNorm2d(num_filters * 4),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(num_filters * 4, num_filters * 8, 4, stride=2, padding=1),
            nn.InstanceNorm2d(num_filters * 8),
            nn.ReLU(inplace=True)
        )
        
        # 残差块
        self.residual_blocks = nn.Sequential(
            *[ResidualBlock(num_filters * 8) for _ in range(6)]
        )
        
        # 解码器
        self.decoder = nn.Sequential(
            nn.ConvTranspose2d(num_filters * 8, num_filters * 4, 4, stride=2, padding=1, output_padding=1),
            nn.InstanceNorm2d(num_filters * 4),
            nn.ReLU(inplace=True),
            
            nn.ConvTranspose2d(num_filters * 4, num_filters * 2, 4, stride=2, padding=1, output_padding=1),
            nn.InstanceNorm2d(num_filters * 2),
            nn.ReLU(inplace=True),
            
            nn.ConvTranspose2d(num_filters * 2, num_filters, 4, stride=2, padding=1, output_padding=1),
            nn.InstanceNorm2d(num_filters),
            nn.ReLU(inplace=True),
            
            nn.Conv2d(num_filters, image_channels, 7, stride=1, padding=3),
            nn.Tanh()
        )
    
    def forward(self, x, domain_labels):
        """
        前向传播
        
        x: 输入图像
        domain_labels: 目标域的one-hot编码
        """
        # 将域标签扩展到与图像相同的空间尺寸
        batch, _, h, w = x.shape
        domain_map = domain_labels.view(batch, -1, 1, 1).expand(batch, -1, h, w)
        
        # 拼接图像和域标签
        combined = torch.cat([x, domain_map], dim=1)
        
        # 编码
        encoded = self.encoder(combined)
        
        # 残差块
        residual_out = self.residual_blocks(encoded)
        
        # 解码
        output = self.decoder(residual_out)
        
        return output

class StarGANDiscriminator(nn.Module):
    """
    StarGAN 判别器
    
    采用分类器结构：
    1. 判断图像真假
    2. 判断图像属于哪个域
    """
    def __init__(self, image_channels=3, num_domains=5, num_filters=64):
        super().__init__()
        
        self.num_domains = num_domains
        
        # 共享特征提取
        self.feature_extractor = nn.Sequential(
            nn.Conv2d(image_channels, num_filters, 4, stride=2, padding=1),
            nn.LeakyReLU(0.2, inplace=True),
            
            nn.Conv2d(num_filters, num_filters * 2, 4, stride=2, padding=1),
            nn.LeakyReLU(0.2, inplace=True),
            
            nn.Conv2d(num_filters * 2, num_filters * 4, 4, stride=2, padding=1),
            nn.LeakyReLU(0.2, inplace=True),
            
            nn.Conv2d(num_filters * 4, num_filters * 8, 4, stride=2, padding=1),
            nn.LeakyReLU(0.2, inplace=True)
        )
        
        # 真假判断头
        self.adversarial_head = nn.Conv2d(num_filters * 8, 1, 3, stride=1, padding=1)
        
        # 域分类头
        self.domain_head = nn.Sequential(
            nn.AdaptiveAvgPool2d(1),
            nn.Flatten(),
            nn.Linear(num_filters * 8, num_domains)
        )
    
    def forward(self, x):
        features = self.feature_extractor(x)
        real_fake = self.adversarial_head(features)
        domain = self.domain_head(features)
        return real_fake, domain
```

---

## 9. 大规模GAN

### 9.1 BigGAN：大规模高保真生成

BigGAN通过大规模训练实现了前所未有的生成质量，核心创新包括：

1. **大batch训练**：使用2048的batch size
2. **类别条件BatchNorm**：根据类别动态调整归一化参数
3. **截断技巧（Truncation Trick）**：控制潜在向量的分布
4. **正交正则化**：稳定训练

```python
class BigGANConfig:
    """BigGAN 配置"""
    batch_size = 2048
    latent_dim = 120
    embedding_dim = 128
    channels = {
        'G': [16, 16, 16, 8, 8, 8, 4, 4, 2, 2, 1],
        'D': [1, 2, 2, 4, 4, 8, 8, 8, 16, 16, 16]
    }

class SelfAttention(nn.Module):
    """
    自注意力模块
    
    使生成器能够捕获图像中的长距离依赖关系
    对于生成复杂场景特别重要
    """
    def __init__(self, channels):
        super().__init__()
        self.channels = channels
        
        # 查询、键、值投影
        self.query = nn.Conv2d(channels, channels // 8, 1)
        self.key = nn.Conv2d(channels, channels // 8, 1)
        self.value = nn.Conv2d(channels, channels, 1)
        
        # 输出投影
        self.out = nn.Conv2d(channels, channels, 1)
        
        # 缩放因子
        self.gamma = nn.Parameter(torch.zeros(1))
    
    def forward(self, x):
        batch_size, C, H, W = x.shape
        
        # 计算 Q, K, V
        q = self.query(x).view(batch_size, -1, H * W).permute(0, 2, 1)  # (B, HW, C')
        k = self.key(x).view(batch_size, -1, H * W)  # (B, C', HW)
        v = self.value(x).view(batch_size, -1, H * W).permute(0, 2, 1)  # (B, HW, C)
        
        # 注意力权重
        attention = torch.bmm(q, k) / (self.channels ** 0.5)
        attention = F.softmax(attention, dim=-1)
        
        # 应用注意力
        out = torch.bmm(attention, v)
        out = out.permute(0, 2, 1).view(batch_size, -1, H, W)
        out = self.out(out)
        
        # 残差连接
        return self.gamma * out + x

class ConditionalBatchNorm2d(nn.Module):
    """
    条件BatchNorm
    
    根据类别标签动态调整归一化参数
    """
    def __init__(self, num_features, num_classes):
        super().__init__()
        self.num_features = num_features
        self.bn = nn.BatchNorm2d(num_features, affine=False)
        self.embed = nn.Embedding(num_classes, num_features * 2)
        
        # 初始化嵌入权重
        nn.init.zeros_(self.embed.weight[:, :num_features])
        nn.init.zeros_(self.embed.weight[:, num_features:])
    
    def forward(self, x, class_labels):
        out = self.bn(x)
        gamma, beta = self.embed(class_labels).chunk(2, dim=-1)
        gamma = gamma.unsqueeze(-1).unsqueeze(-1)
        beta = beta.unsqueeze(-1).unsqueeze(-1)
        return gamma * out + beta

class BigGANGeneratorBlock(nn.Module):
    """
    BigGAN 生成器模块
    
    包含：
    1. 上采样
    2. 条件BatchNorm
    3. 非线性激活
    4. 卷积
    5. 可选的自注意力
    """
    def __init__(self, in_channels, out_channels, latent_dim, num_classes,
                 upsample=True, attention=False):
        super().__init__()
        
        self.attention = SelfAttention(out_channels) if attention else None
        
        # 条件BatchNorm
        self.cbn1 = ConditionalBatchNorm2d(in_channels, num_classes)
        self.cbn2 = ConditionalBatchNorm2d(out_channels, num_classes)
        
        # 卷积层
        self.conv = nn.Conv2d(in_channels, out_channels, 3, padding=1)
        self.conv_out = nn.Conv2d(out_channels, out_channels, 3, padding=1)
        
        self.upsample = nn.Upsample(scale_factor=2, mode='bilinear') if upsample else None
    
    def forward(self, x, latent, class_labels):
        # 第一层
        x = self.cbn1(x, class_labels)
        x = F.relu(x)
        if self.upsample:
            x = self.upsample(x)
        x = self.conv(x)
        
        # 第二层
        x = self.cbn2(x, class_labels)
        x = F.relu(x)
        x = self.conv_out(x)
        
        # 自注意力
        if self.attention:
            x = self.attention(x)
        
        return x
```

### 9.2 SAGAN：自注意力GAN

SAGAN将自注意力机制引入GAN，使生成器和判别器都能捕获全局依赖：

```python
class SelfAttentionGAN:
    """
    SAGAN (Self-Attention GAN) 完整训练框架
    """
    
    def __init__(self, num_classes=1000, latent_dim=128, image_size=128):
        self.num_classes = num_classes
        self.latent_dim = latent_dim
        
        self.generator = SAGANGenerator(latent_dim, num_classes, image_size)
        self.discriminator = SAGANDiscriminator(num_classes, image_size)
        
        # 谱归一化（用于判别器）
        self.apply_spectral_norm(self.discriminator)
    
    @staticmethod
    def apply_spectral_norm(model):
        """应用谱归一化到所有卷积层"""
        for module in model.modules():
            if isinstance(module, (nn.Conv2d, nn.ConvTranspose2d)):
                nn.utils.spectral_norm(module)
    
    def generate_condition_vector(self, z, class_labels):
        """生成条件向量：潜在向量 + 类别嵌入"""
        label_embedding = nn.Embedding(self.num_classes, self.latent_dim)(class_labels)
        return z + 0.1 * label_embedding  # 残差连接

class SAGANLoss:
    """
    SAGAN 的 hinge loss
    
    比原始GAN的logistic loss更稳定
    """
    def __init__(self, lambda_gp=10):
        self.lambda_gp = lambda_gp
    
    def d_loss(self, real_logit, fake_logit):
        """判别器损失：hinge loss"""
        real_loss = F.relu(1 - real_logit).mean()
        fake_loss = F.relu(1 + fake_logit).mean()
        return (real_loss + fake_loss) / 2
    
    def g_loss(self, fake_logit):
        """生成器损失"""
        return -fake_logit.mean()
    
    def gradient_penalty(self, discriminator, real, fake):
        """WGAN-GP风格梯度惩罚"""
        alpha = torch.rand(real.size(0), 1, 1, 1)
        interpolated = alpha * real + (1 - alpha) * fake
        
        interpolated.requires_grad_(True)
        d_interp = discriminator(interpolated)
        
        gradients = torch.autograd.grad(
            outputs=d_interp,
            inputs=interpolated,
            grad_outputs=torch.ones_like(d_interp),
            create_graph=True,
            retain_graph=True
        )[0]
        
        gradients = gradients.view(real.size(0), -1)
        grad_norm = gradients.norm(2, dim=1)
        penalty = ((grad_norm - 1) ** 2).mean()
        
        return self.lambda_gp * penalty
```

---

## 10. GAN的训练技巧与稳定性

### 10.1 训练不稳定性的根源与解决方案

GAN训练的不稳定性源于多个方面，以下是系统性的解决方案：

**问题1：模式崩溃（Mode Collapse）**

解决方案：
- 使用Unrolled GAN：让生成器看到判别器的未来更新
- 使用Minibatch Discrimination：在判别器中添加批次判别
- 使用多个判别器或生成器

```python
class MinibatchDiscrimination(nn.Module):
    """
    小批次判别
    
    目的：防止生成器只产生相似的样本
    
    方法：在判别器中计算样本之间的相似度
    """
    def __init__(self, input_channels, num_kernels, kernel_dim=5):
        super().__init__()
        self.num_kernels = num_kernels
        self.kernel_dim = kernel_dim
        
        # 将输入映射到潜在空间
        self.mapping = nn.Linear(input_channels, num_kernels * kernel_dim)
    
    def forward(self, x):
        batch_size = x.size(0)
        
        # 映射并重塑
        mapped = self.mapping(x)
        mapped = mapped.view(batch_size, self.num_kernels, self.kernel_dim)
        
        # 计算每个样本与其他样本的L1距离
        x1 = mapped.unsqueeze(2)  # (B, num_kernels, 1, kernel_dim)
        x2 = mapped.unsqueeze(1)  # (B, 1, num_kernels, kernel_dim)
        
        # L1距离
        diff = torch.abs(x1 - x2)
        distances = torch.sum(diff, dim=3)  # (B, num_kernels, num_kernels)
        
        # 取负指数得到相似度
        similarities = torch.exp(-distances)
        
        # 对每个样本，计算与其他所有样本的相似度之和（减去自身）
        minibatch_features = torch.sum(similarities, dim=2) - 1
        
        return minibatch_features
```

**问题2：梯度消失**

解决方案：
- 使用Wasserstein距离替代JS散度
- 使用带有动量的优化器
- 避免使用sigmoid交叉熵损失

**问题3：平衡困难**

解决方案：
- 使用TTUR（Two Timescale Update Rule）
- 判别器多更新几次
- 自适应学习率

```python
class TTUROptimizer:
    """
    TTUR：双时间尺度更新规则
    
    生成器和判别器使用不同的学习率
    通常判别器用更大的学习率
    """
    def __init__(self, generator, discriminator, 
                 g_lr=0.0001, d_lr=0.0004,
                 b1=0.0, b2=0.999):
        self.optimizer_G = optim.Adam(generator.parameters(), lr=g_lr, betas=(b1, b2))
        self.optimizer_D = optim.Adam(discriminator.parameters(), lr=d_lr, betas=(b1, b2))
    
    def step(self, generator_loss, discriminator_loss):
        # 先更新判别器
        self.optimizer_D.zero_grad()
        discriminator_loss.backward()
        self.optimizer_D.step()
        
        # 再更新生成器
        self.optimizer_G.zero_grad()
        generator_loss.backward()
        self.optimizer_G.step()
```

### 10.2 谱归一化（Spectral Normalization）

谱归一化是稳定GAN训练的重要技术，它约束判别器的Lipschitz常数：

```python
def apply_spectral_norm_to_conv(conv_layer):
    """
    对卷积层应用谱归一化
    
    核心思想：使每一层的Lipschitz常数 ≤ 1
    
    实现：使用权重的最大奇异值进行归一化
    """
    return nn.utils.spectral_norm(conv_layer)

class SNDense(nn.Module):
    """谱归一化的全连接层"""
    def __init__(self, in_features, out_features):
        super().__init__()
        self.linear = nn.Linear(in_features, out_features)
        nn.utils.spectral_norm(self.linear)
    
    def forward(self, x):
        return self.linear(x)

class SNConv2d(nn.Module):
    """谱归一化的卷积层"""
    def __init__(self, in_channels, out_channels, kernel_size, stride=1, padding=0):
        super().__init__()
        self.conv = nn.Conv2d(in_channels, out_channels, kernel_size, 
                              stride=stride, padding=padding)
        nn.utils.spectral_norm(self.conv)
    
    def forward(self, x):
        return self.conv(x)

def create_sn_discriminator(base_discriminator):
    """
    将任意判别器转换为谱归一化版本
    
    递归遍历所有子模块，将卷积和线性层替换为谱归一化版本
    """
    def replace_with_sn(module):
        for name, child in module.named_children():
            if isinstance(child, nn.Conv2d):
                setattr(module, name, SNConv2d(
                    child.in_channels, child.out_channels,
                    child.kernel_size, child.stride, child.padding
                ))
            elif isinstance(child, nn.Linear):
                setattr(module, name, SNDense(child.in_features, child.out_features))
            else:
                replace_with_sn(child)
    
    sn_discriminator = copy.deepcopy(base_discriminator)
    replace_with_sn(sn_discriminator)
    return sn_discriminator
```

### 10.3 实例归一化与自适应归一化

```python
class AdaptiveInstanceNorm(nn.Module):
    """
    自适应实例归一化（AdaIN）
    
    风格迁移的核心操作
    """
    def __init__(self, eps=1e-5):
        super().__init__()
        self.eps = eps
    
    def forward(self, content, style):
        """
        content: 内容特征
        style: 风格特征
        """
        size = content.size()
        
        # 计算content的均值和方差
        content_mean = content.view(size[0], size[1], -1).mean(dim=2).view(size[0], size[1], 1, 1)
        content_var = content.view(size[0], size[1], -1).var(dim=2, unbiased=False).view(size[0], size[1], 1, 1)
        
        # 计算style的均值和方差
        style_mean = style.view(size[0], size[1], -1).mean(dim=2).view(size[0], size[1], 1, 1)
        style_var = style.view(size[0], size[1], -1).var(dim=2, unbiased=False).view(size[0], size[1], 1, 1)
        
        # 归一化content，然后用style的统计量进行缩放
        content_norm = (content - content_mean) / torch.sqrt(content_var + self.eps)
        output = content_norm * torch.sqrt(style_var + self.eps) + style_mean
        
        return output

class LayerNorm2d(nn.Module):
    """
    层归一化（针对2D特征图）
    
    与Instance Norm的区别：
    - Instance Norm: 对每个样本、每个通道独立归一化
    - Layer Norm: 对每个样本、每个位置的所有通道归一化
    """
    def __init__(self, eps=1e-5):
        super().__init__()
        self.eps = eps
    
    def forward(self, x):
        # x: (B, C, H, W)
        mean = x.mean(dim=(1, 2, 3), keepdim=True)
        var = x.var(dim=(1, 2, 3), unbiased=False, keepdim=True)
        return (x - mean) / torch.sqrt(var + self.eps)

class GroupNorm(nn.Module):
    """
    组归一化（Group Normalization）
    
    不依赖于batch size，适合batch很小时使用
    """
    def __init__(self, num_groups, num_channels, eps=1e-5):
        super().__init__()
        self.num_groups = num_groups
        self.num_channels = num_channels
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(num_channels))
        self.bias = nn.Parameter(torch.zeros(num_channels))
    
    def forward(self, x):
        # x: (B, C, H, W)
        B, C, H, W = x.shape
        G = self.num_groups
        
        # 将通道分成G组
        x = x.view(B, G, C // G, H, W)
        
        # 在每组内计算均值和方差
        mean = x.mean(dim=(2, 3, 4), keepdim=True)
        var = x.var(dim=(2, 3, 4), unbiased=False, keepdim=True)
        
        # 归一化
        x = (x - mean) / torch.sqrt(var + self.eps)
        x = x.view(B, C, H, W)
        
        # 仿射变换
        return x * self.weight.view(1, C, 1, 1) + self.bias.view(1, C, 1, 1)
```

---

## 11. GAN的评估指标

### 11.1 Inception Score (IS)

Inception Score是评估GAN生成质量的经典指标：

$$
\text{IS}(G) = \exp\left(\mathbb{E}_{x \sim p_g} \left[ D_{\text{KL}}(p(y|x) \| p(y)) \right]\right)
$$

其中 $p(y|x)$ 是Inception网络的类别输出，$p(y) = \mathbb{E}_{x \sim p_g}[p(y|x)]$。

```python
def calculate_inception_score(images, model, num_splits=10):
    """
    计算Inception Score
    
    原理：
    - 如果生成图像质量高，Inception网络对每张图的预测应该是确定的（低熵）
    - 如果生成图像多样性高，各类别的边缘分布应该均匀（高熵）
    
    KL散度 = 条件熵的减少量 = 模型对图像的确定性程度
    """
    import torch.nn.functional as F
    
    model.eval()
    preds = []
    
    with torch.no_grad():
        for i in range(0, len(images), batch_size):
            batch = images[i:i+batch_size]
            batch = F.interpolate(batch, size=(299, 299), mode='bilinear')
            pred = model(batch)
            preds.append(F.softmax(pred, dim=-1).cpu().numpy())
    
    preds = np.concatenate(preds, axis=0)
    
    # 计算每张图的KL散度
    split_scores = []
    split_size = preds.shape[0] // num_splits
    
    for k in range(num_splits):
        part = preds[k * split_size:(k + 1) * split_size, :]
        # 计算 p(y)
        py = np.mean(part, axis=0)
        # 计算每个样本的KL散度
        kl = part * (np.log(part) - np.log(py + 1e-16))
        kl = np.sum(kl, axis=1)
        split_scores.append(np.exp(np.mean(kl)))
    
    return np.mean(split_scores), np.std(split_scores)
```

### 11.2 Fréchet Inception Distance (FID)

FID通过比较真实图像和生成图像在特征空间中的分布距离：

$$
\text{FID} = \|\mu_1 - \mu_2\|^2 + \text{Tr}(\Sigma_1 + \Sigma_2 - 2\sqrt{\Sigma_1 \Sigma_2})
$$

```python
def calculate_fid(real_images, fake_images, model):
    """
    计算Fréchet Inception Distance
    
    步骤：
    1. 用Inception网络提取真实图像和生成图像的特征
    2. 假设特征服从多元高斯分布
    3. 计算两个高斯分布之间的Fréchet距离
    """
    model.eval()
    
    def get_activations(images):
        activations = []
        with torch.no_grad():
            for i in range(0, len(images), batch_size):
                batch = images[i:i+batch_size]
                batch = F.interpolate(batch, size=(299, 299), mode='bilinear')
                act = model(batch)
                activations.append(act.cpu().numpy())
        return np.concatenate(activations, axis=0)
    
    # 获取特征
    real_acts = get_activations(real_images)
    fake_acts = get_activations(fake_images)
    
    # 计算均值和协方差
    mu_real = np.mean(real_acts, axis=0)
    mu_fake = np.mean(fake_acts, axis=0)
    
    sigma_real = np.cov(real_acts, rowvar=False)
    sigma_fake = np.cov(fake_acts, rowvar=False)
    
    # 计算FID
    diff = mu_real - mu_fake
    covmean = sqrtm(sigma_real @ sigma_fake)
    
    if np.iscomplexobj(covmean):
        covmean = covmean.real
    
    fid = diff @ diff + np.trace(sigma_real + sigma_fake - 2 * covmean)
    
    return fid

def sqrtm(matrix):
    """矩阵平方根的数值稳定计算"""
    eigenvalues, eigenvectors = np.linalg.eigh(matrix)
    return eigenvectors @ np.diag(np.sqrt(np.maximum(eigenvalues, 0))) @ eigenvectors.T
```

### 11.3 Precision, Recall和F1

```python
def calculate_precision_recall(real_features, fake_features, k=3):
    """
    计算生成模型的Precision和Recall
    
    Precision：生成样本中有多少可以被视为真实样本的近邻
    Recall：真实样本中有多少可以被生成样本覆盖
    """
    from sklearn.neighbors import NearestNeighbors
    
    # 使用k近邻
    nn_real = NearestNeighbors(n_neighbors=k + 1).fit(real_features)
    nn_fake = NearestNeighbors(n_neighbors=k + 1).fit(fake_features)
    
    # 计算fake样本的precision
    _, indices = nn_real.kneighbors(fake_features)
    precision = np.mean([1 for idx in indices if idx[0] in idx[1:]])  # 简化
    
    # 计算recall
    _, indices = nn_fake.kneighbors(real_features)
    recall = np.mean([1 for idx in indices if idx[0] in idx[1:]])
    
    # F1分数
    f1 = 2 * precision * recall / (precision + recall + 1e-8)
    
    return precision, recall, f1

def manifold_kneighbors(X_train, X_test, n_neighbors=5):
    """
    流形k近邻分析
    
    用于评估生成样本是否位于真实数据流形上
    """
    from sklearn.neighbors import NearestNeighbors
    
    nn = NearestNeighbors(n_neighbors=n_neighbors)
    nn.fit(X_train)
    
    distances, indices = nn.kneighbors(X_test)
    
    return distances, indices
```

---

## 12. GAN的应用场景

### 12.1 图像生成与增强

GAN在图像生成领域的应用包括：

- **人脸合成**：StyleGAN系列生成逼真的人脸
- **艺术创作**：GAN辅助艺术设计，如DeepDream风格迁移
- **数据增强**：生成稀有类别的样本以改善分类器训练
- **超分辨率**：ESRGAN等用于图像放大

```python
class DataAugmentationGAN:
    """
    用于数据增强的GAN
    
    特别适用于：
    1. 医疗影像（数据稀缺）
    2. 稀有事件检测
    3. 小样本学习
    """
    def __init__(self, generator, num_classes):
        self.generator = generator
        self.num_classes = num_classes
    
    def generate_for_class(self, target_class, num_samples):
        """为特定类别生成样本"""
        z = torch.randn(num_samples, self.generator.latent_dim)
        labels = torch.full((num_samples,), target_class)
        
        with torch.no_grad():
            generated = self.generator(z, labels)
        
        return generated
    
    def augment_dataset(self, dataset, samples_per_class=100):
        """增强整个数据集"""
        augmented_images = []
        augmented_labels = []
        
        for class_idx in range(self.num_classes):
            gen_samples = self.generate_for_class(class_idx, samples_per_class)
            augmented_images.append(gen_samples.cpu().numpy())
            augmented_labels.extend([class_idx] * samples_per_class)
        
        augmented_images = np.concatenate(augmented_images, axis=0)
        augmented_labels = np.array(augmented_labels)
        
        return augmented_images, augmented_labels
```

### 12.2 图像编辑与操作

GAN使精细的图像编辑成为可能：

- **语义编辑**：如GANSpace、InterfaceGAN等
- **局部修改**：如Inpainting网络
- **风格混合**：不同风格图像的融合

```python
class GANBasedImageEditor:
    """
    基于GAN的图像编辑
    
    技术：
    1. 潜在空间插值
    2. 方向向量操纵
    3. 层叠编辑
    """
    
    def __init__(self, generator):
        self.G = generator
    
    def interpolate(self, z1, z2, num_steps=10):
        """在两个潜在向量之间插值"""
        alphas = np.linspace(0, 1, num_steps)
        interpolated = []
        
        for alpha in alphas:
            z = alpha * z1 + (1 - alpha) * z2
            with torch.no_grad():
                img = self.G(z)
            interpolated.append(img)
        
        return torch.stack(interpolated)
    
    def find_edit_direction(self, attribute1_images, attribute2_images):
        """
        学习属性编辑方向
        
        方法：在两个属性类别的中心点之间建立方向向量
        """
        with torch.no_grad():
            feats1 = torch.stack([self.extract_feature(img) for img in attribute1_images])
            feats2 = torch.stack([self.extract_feature(img) for img in attribute2_images])
        
        center1 = feats1.mean(dim=0)
        center2 = feats2.mean(dim=0)
        
        direction = center2 - center1
        return direction / (torch.norm(direction) + 1e-8)
    
    def edit_image(self, z, direction, magnitude):
        """
        沿方向向量编辑图像
        
        z: 原始潜在向量
        direction: 编辑方向
        magnitude: 编辑强度
        """
        z_edited = z + magnitude * direction
        
        with torch.no_grad():
            edited_image = self.G(z_edited)
        
        return edited_image
    
    def layer_wise_edit(self, z, layer_idx, direction):
        """
        分层编辑
        
        不同层控制不同级别的特征：
        - 低层：颜色、纹理
        - 中层：局部结构
        - 高层：整体布局、语义
        """
        # 实现细节：修改指定层的激活
        pass
```

### 12.3 视频生成

GAN也被扩展到视频生成领域：

- **视频预测**：预测未来帧
- **视频风格化**：如Vid2Vid
- **舞蹈生成**：根据音乐生成舞蹈动作

```python
class VideoGenerator:
    """
    视频生成GAN
    
    关键组件：
    1. 时序建模：LSTM/Transformer
    2. 运动一致性：光流约束
    3. 帧间平滑：时序判别器
    """
    
    def __init__(self, image_generator, hidden_dim=256):
        self.image_gen = image_generator
        self.temporal_model = nn.LSTM(hidden_dim, hidden_dim, batch_first=True)
    
    def generate_video(self, num_frames, latent_dim):
        """生成长度为num_frames的视频"""
        batch_size = 1
        
        # 初始化隐藏状态
        h = torch.zeros(1, batch_size, hidden_dim)
        c = torch.zeros(1, batch_size, hidden_dim)
        
        frames = []
        z_prev = torch.randn(batch_size, latent_dim)
        
        for t in range(num_frames):
            # 时序建模
            output, (h, c) = self.temporal_model(z_prev.unsqueeze(1), (h, c))
            z_curr = output.squeeze(1) + torch.randn(batch_size, latent_dim) * 0.1
            
            # 生成帧
            with torch.no_grad():
                frame = self.image_gen(z_curr)
            frames.append(frame)
            
            z_prev = z_curr
        
        return torch.stack(frames, dim=1)  # (B, T, C, H, W)
```

### 12.4 文本生成与多模态

GAN与语言模型的结合：

- **文本到图像**：如StackGAN、DALL-E系列思想
- **图像到文本**：生成描述
- **跨模态转换**：如CLIP引导的编辑

```python
class TextToImageGAN:
    """
    文本到图像生成
    
    架构：
    1. 文本编码器：LSTM/Transformer
    2. 层级生成器：从粗到细
    3. 匹配判别器：文本-图像匹配判断
    """
    
    def __init__(self, vocab_size, embed_dim, latent_dim):
        self.text_encoder = nn.LSTM(
            vocab_size, embed_dim, 
            batch_first=True, bidirectional=True
        )
        
        self.stage1_generator = Stage1Generator(embed_dim * 2, latent_dim)
        self.stage2_generator = Stage2Generator(embed_dim * 2, latent_dim)
        
        self.matching_discriminator = MatchingDiscriminator()
    
    def encode_text(self, text_tokens):
        """编码文本描述"""
        output, (hidden, _) = self.text_encoder(text_tokens)
        # 拼接双向隐藏状态
        hidden = torch.cat([hidden[-2], hidden[-1]], dim=-1)
        return hidden
    
    def generate(self, text, stage=2):
        """从文本生成图像"""
        text_features = self.encode_text(text)
        
        if stage == 1:
            return self.stage1_generator(text_features)
        else:
            # 两阶段：从粗到细
            coarse = self.stage1_generator(text_features)
            refined = self.stage2_generator(text_features, coarse)
            return refined
```

---

## 13. GAN的最新研究方向

### 13.1 Diffusion GAN：融合扩散模型

Diffusion Models的兴起促使研究者探索GAN与扩散模型的结合：

```python
class DiffusionGAN:
    """
    Diffusion GAN
    
    思想：用GAN加速扩散模型的采样过程
    
    原理：
    - 扩散模型需要多步迭代去噪
    - GAN可以学习一步到位的去噪
    - 两者结合：GAN作为扩散模型的加速器
    """
    
    def __init__(self, latent_dim, diffusion_steps=1000):
        self.denoiser = ConditionalGenerator(latent_dim)
        self.discriminator = Discriminator()
        self.diffusion_steps = diffusion_steps
    
    def forward_diffusion(self, x0, t):
        """前向扩散：添加噪声"""
        noise = torch.randn_like(x0)
        alpha_bar = self.get_noise_schedule(t)
        return torch.sqrt(alpha_bar) * x0 + torch.sqrt(1 - alpha_bar) * noise, noise
    
    def gan_denoise(self, xt, timestep):
        """
        用GAN进行去噪
        
        GAN学习从xt到x_{t-1}的映射
        比传统去噪网络更高效
        """
        t_embedding = self.get_timestep_embedding(timestep)
        return self.denoiser(xt, t_embedding)
    
    def train_step(self, real_images, text_features=None):
        """训练Diffusion GAN"""
        # 采样时间步
        t = torch.randint(0, self.diffusion_steps, (real_images.size(0),))
        
        # 前向扩散
        noisy_images, noise = self.forward_diffusion(real_images, t)
        
        # GAN去噪
        denoised = self.gan_denoise(noisy_images, t)
        
        # 训练判别器
        d_loss = self.compute_discriminator_loss(denoised.detach(), real_images)
        
        # 训练生成器（去噪器）
        g_loss = self.compute_generator_loss(denoised, real_images)
        
        return d_loss, g_loss
```

### 13.2 Transformer GAN

Transformer架构也开始应用于GAN：

```python
class TransformerDiscriminator(nn.Module):
    """
    基于Transformer的判别器
    
    使用自注意力捕获图像中的长距离依赖
    """
    def __init__(self, patch_size=16, embed_dim=768, num_heads=12, num_layers=12):
        super().__init__()
        
        self.patch_embed = nn.Conv2d(3, embed_dim, patch_size, stride=patch_size)
        
        self.pos_embed = nn.Parameter(torch.zeros(1, (256 // patch_size) ** 2, embed_dim))
        
        encoder_layer = nn.TransformerEncoderLayer(
            d_model=embed_dim,
            nhead=num_heads,
            dim_feedforward=embed_dim * 4,
            dropout=0.1,
            activation='gelu',
            batch_first=True
        )
        self.transformer = nn.TransformerEncoder(encoder_layer, num_layers=num_layers)
        
        self.norm = nn.LayerNorm(embed_dim)
        self.head = nn.Linear(embed_dim, 1)
    
    def forward(self, x):
        # 分patch
        x = self.patch_embed(x)  # (B, embed_dim, H/P, W/P)
        x = x.flatten(2).transpose(1, 2)  # (B, num_patches, embed_dim)
        
        # 添加位置编码
        x = x + self.pos_embed
        
        # Transformer编码
        x = self.transformer(x)
        x = self.norm(x)
        
        # 全局池化 + 分类
        x = x.mean(dim=1)
        return self.head(x)
```

### 13.3 NeRF-GAN：三维感知生成

```python
class NeRFGAN:
    """
    NeRF-GAN：三维感知生成
    
    结合神经辐射场（NeRF）和GAN
    支持从任意视角渲染生成的三维内容
    """
    
    def __init__(self):
        self.nerf = NeuralRadianceField()
        self.rendering_layer = VolumetricRenderer()
    
    def generate_3d_scene(self, z):
        """从潜在向量生成完整的三维场景"""
        # 生成场景参数
        scene_params = self.decode_latent(z)
        
        # 体积渲染
        rgb, depth = self.rendering_layer(
            self.nerf, 
            scene_params,
            camera_poses=self.sample_camera_poses()
        )
        
        return rgb, depth
    
    def train(self, real_images, camera_poses):
        """训练NeRF-GAN"""
        # 渲染伪影
        fake_images = self.generate_3d_scene(self.get_latent(real_images))
        
        # GAN损失
        d_loss = self.compute_gan_loss(real_images, fake_images)
        g_loss = self.compute_generator_loss(fake_images)
        
        # 渲染一致性损失
        render_loss = self.compute_render_consistency_loss(real_images, fake_images)
        
        return d_loss, g_loss + render_loss
```

---

## 14. 学术引用与参考文献

1. Goodfellow, I., et al. (2014). "Generative Adversarial Networks." *NeurIPS*.
2. Arjovsky, M., Chintala, S., & Bottou, L. (2017). "Wasserstein GAN." *ICML*.
3. Radford, A., Metz, L., & Chintala, S. (2016). "Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks." *ICLR*.
4. Mirza, M., & Osindero, S. (2014). "Conditional Generative Adversarial Nets." *arXiv*.
5. Karras, T., et al. (2018). "Progressive Growing of GANs for Improved Quality, Stability, and Variation." *ICLR*.
6. Karras, T., et al. (2019). "A Style-Based Generator Architecture for Generative Adversarial Networks." *CVPR*.
7. Karras, T., et al. (2021). "Alias-Free Generative Adversarial Networks." *NeurIPS*.
8. Isola, P., et al. (2017). "Image-to-Image Translation with Conditional Adversarial Networks." *CVPR*.
9. Zhu, J.-Y., et al. (2017). "Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks." *ICCV*.
10. Choi, Y., et al. (2018). "StarGAN: Unified Generative Adversarial Networks for Multi-Domain Image-to-Image Translation." *CVPR*.
11. Brock, A., Donahue, J., & Simonyan, K. (2019). "Large Scale GAN Training for High Fidelity Natural Image Synthesis." *ICLR*.
12. Zhang, H., et al. (2019). "Self-Attention Generative Adversarial Networks." *ICML*.
13. Gulrajani, I., et al. (2017). "Improved Training of Wasserstein GANs." *NeurIPS*.
14. Miyato, T., et al. (2018). "Spectral Normalization for Generative Adversarial Networks." *ICLR*.
15. Mao, X., et al. (2017). "Least Squares Generative Adversarial Networks." *ICCV*.

---

## 15. 相关文档

- [[GAN变体详解]] - 深入了解 CGAN、Pix2Pix、CycleGAN、BigGAN 等变体
- [[对抗样本深度指南]] - 对抗样本的定义与攻击方法
- [[博弈论与AI]] - 理解 GAN 的博弈论基础
- [[对抗训练与鲁棒性]] - 对抗训练提升模型鲁棒性
- [[多智能体博弈详解]] - 多智能体系统中的对抗学习
