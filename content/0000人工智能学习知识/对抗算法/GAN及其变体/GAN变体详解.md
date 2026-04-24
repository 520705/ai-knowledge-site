---
title: GAN变体详解
date: 2026-04-18
tags:
  - GAN变体
  - 图像转换
  - 深度学习
  - 生成模型
  - 评估指标
categories:
  - 人工智能
  - 对抗算法
  - GAN及其变体
alias: GAN变体详解
---

## 关键词

| 术语 | 英文 | 核心概念 |
|------|------|----------|
| 条件生成对抗网络 | cGAN | 基于条件标签的引导生成 |
| 图像到图像转换 | Pix2Pix | 配对图像转换的通用框架 |
| 无配对转换 | CycleGAN | 无需配对数据的域转换 |
| 大尺度生成 | BigGAN | 高分辨率大规模图像生成 |
| 自注意力机制 | Self-Attention | 捕获长距离依赖关系 |
| 弗雷歇 inception 距离 | FID | 评估生成图像质量的核心指标 |
| Inception Score | IS | 衡量生成样本质量和多样性的指标 |
| 生成器-判别器一致性 | Cycle Consistency | 循环一致性损失函数 |
| 类别平衡采样 | Class Balancing | BigGAN的训练策略 |
| 谱归一化 | Spectral Normalization | 稳定GAN训练的归一化技术 |

---

## 1. 引言

自2014年 Goodfellow 提出原始 GAN 以来，研究者们开发了数百种 GAN 变体来解决特定问题或提升性能。本文档系统梳理最具影响力的 GAN 变体架构，涵盖条件生成、图像转换、大尺度训练和评估体系四个维度。

---

## 2. GAN变体全景图：从原始GAN到StyleGAN3

说起GAN的发展史，简直就是一部"打怪升级"的进化史。2014年Goodfellow发表那篇开山之作时，估计也没想到这个"两个网络互相对抗"的简单想法，会在接下来十年里催生出这么多厉害的变体。

### 2.1 原始GAN的问题

原始GAN听起来很美好——生成器和判别器互相博弈，最终达到平衡。但实际操作起来，问题一大堆：

**训练不稳定**是最头疼的。生成器和判别器的训练速度很难平衡，经常出现一方把另一方碾压的情况。一旦判别器太强，生成器就彻底学不动了；反过来的话，判别器就成了摆设。

**模式崩溃（Mode Collapse）**也是常见病。生成器学会了只生成那么几种"安全"的样本，虽然骗过了判别器，但实际上根本没学到真实数据的分布。比如说你想生成各种数字的手写图像，结果模型只会生成"1"，那可就尴尬了。

**梯度消失**问题在早期特别严重。当判别器接近完美时，生成器能获得的梯度几乎为零，根本不知道怎么改进。

### 2.2 GAN家族的进化路线

针对这些问题，后来的研究者们从多个方向开展了"救援行动"：

**架构改进派**：代表是DCGAN，直接把全连接网络换成卷积神经网络，让GAN终于能生成像样的图像了。

**损失函数改进派**：WGAN系列另辟蹊径，换掉了JS散度这个"罪魁祸首"，用Wasserstein距离从根本上解决了训练不稳定的问题。

**条件控制派**：cGAN、AC-GAN这些让生成变得可控，你想生成什么类别的图像都行。

**图像翻译派**：Pix2Pix、CycleGAN解决了一类特殊但超实用的任务——图像到图像的转换。

**高分辨率派**：Progressive GAN、StyleGAN系列告诉你怎么生成高清大图，而且还能控制生成图像的各种细节。

**大规模训练派**：BigGAN证明了"大力出奇迹"，只要硬件够强、数据够多、batch size够大，效果就是能上一个台阶。

接下来的章节，我们就按照这个脉络，一个一个详细聊聊这些变体到底是怎么回事。

---

## 3. DCGAN实战：深度卷积GAN的PyTorch代码

DCGAN（Deep Convolutional GAN）是第一个真正让GAN"看见图像"的架构。2016年Radford等人发表这篇论文时，首次证明了卷积神经网络配合GAN可以生成质量不错的图像。

### 3.1 DCGAN的核心设计原则

DCGAN之所以能成功，关键在于几个精心设计的设计决策：

**全卷积网络**：生成器和判别器都使用卷积层代替池化层，让网络自己学习"该看哪里"。

**批归一化（Batch Normalization）**：几乎所有层都加了BN，这让训练稳定多了。

**移除全连接层**：早期的GAN喜欢在中间加几层全连接，DCGAN直接把它们干掉了，减少了参数量。

**ReLU和LeakyReLU**：生成器用ReLU输出层用Tanh，判别器全部用LeakyReLU。

### 3.2 完整的PyTorch实现

```python
import torch
import torch.nn as nn

class Generator(nn.Module):
    """DCGAN生成器：从随机噪声生成图像"""
    def __init__(self, latent_dim=100, channels=3, features_g=64):
        super().__init__()
        self.latent_dim = latent_dim
        
        # 输入是 latent_dim x 1 x 1 的特征图
        self.net = nn.Sequential(
            # latent_dim -> features_g*8*4*4
            nn.ConvTranspose2d(latent_dim, features_g * 8, 4, 1, 0, bias=False),
            nn.BatchNorm2d(features_g * 8),
            nn.ReLU(True),
            # 4x4 -> 8x8
            nn.ConvTranspose2d(features_g * 8, features_g * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g * 4),
            nn.ReLU(True),
            # 8x8 -> 16x16
            nn.ConvTranspose2d(features_g * 4, features_g * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g * 2),
            nn.ReLU(True),
            # 16x16 -> 32x32
            nn.ConvTranspose2d(features_g * 2, features_g, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_g),
            nn.ReLU(True),
            # 32x32 -> 64x64
            nn.ConvTranspose2d(features_g, channels, 4, 2, 1, bias=False),
            nn.Tanh()  # 输出范围 [-1, 1]
        )
    
    def forward(self, z):
        z = z.view(z.size(0), self.latent_dim, 1, 1)
        return self.net(z)


class Discriminator(nn.Module):
    """DCGAN判别器：判断图像是真是假"""
    def __init__(self, channels=3, features_d=64):
        super().__init__()
        
        # 图像逐步变小，通道数逐步变大
        self.net = nn.Sequential(
            # 64x64 -> 32x32
            nn.Conv2d(channels, features_d, 4, 2, 1, bias=False),
            nn.LeakyReLU(0.2, inplace=True),
            # 32x32 -> 16x16
            nn.Conv2d(features_d, features_d * 2, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 2),
            nn.LeakyReLU(0.2, inplace=True),
            # 16x16 -> 8x8
            nn.Conv2d(features_d * 2, features_d * 4, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 4),
            nn.LeakyReLU(0.2, inplace=True),
            # 8x8 -> 4x4
            nn.Conv2d(features_d * 4, features_d * 8, 4, 2, 1, bias=False),
            nn.BatchNorm2d(features_d * 8),
            nn.LeakyReLU(0.2, inplace=True),
            # 4x4 -> 1x1
            nn.Conv2d(features_d * 8, 1, 4, 1, 0, bias=False),
            nn.Sigmoid()
        )
    
    def forward(self, x):
        return self.net(x).view(x.size(0), -1).squeeze(1)


def train_dcgan(generator, discriminator, dataloader, epochs=50, device='cuda'):
    """DCGAN训练循环示例"""
    criterion = nn.BCELoss()
    optimizer_g = torch.optim.Adam(generator.parameters(), lr=0.0002, betas=(0.5, 0.999))
    optimizer_d = torch.optim.Adam(discriminator.parameters(), lr=0.0002, betas=(0.5, 0.999))
    
    generator.to(device)
    discriminator.to(device)
    
    for epoch in range(epochs):
        for batch_idx, real_images in enumerate(dataloader):
            real_images = real_images.to(device)
            batch_size = real_images.size(0)
            
            # 训练判别器：让它分清明真假
            noise = torch.randn(batch_size, 100, device=device)
            fake_images = generator(noise)
            
            real_labels = torch.full((batch_size,), 1.0, device=device)
            fake_labels = torch.full((batch_size,), 0.0, device=device)
            
            optimizer_d.zero_grad()
            
            # 真图像判别
            real_output = discriminator(real_images)
            d_loss_real = criterion(real_output, real_labels)
            
            # 假图像判别
            fake_output = discriminator(fake_images.detach())
            d_loss_fake = criterion(fake_output, fake_labels)
            
            # 判别器总损失
            d_loss = (d_loss_real + d_loss_fake) / 2
            d_loss.backward()
            optimizer_d.step()
            
            # 训练生成器：让假图像骗过判别器
            optimizer_g.zero_grad()
            fake_output = discriminator(fake_images)
            g_loss = criterion(fake_output, real_labels)  # 生成器希望被判别器识别为真
            g_loss.backward()
            optimizer_g.step()
            
            if batch_idx % 100 == 0:
                print(f"Epoch [{epoch}/{epochs}] Batch [{batch_idx}/{len(dataloader)}] "
                      f"D_loss: {d_loss.item():.4f}, G_loss: {g_loss.item():.4f}")
```

### 3.3 训练DCGAN的小技巧

DCGAN虽然比原始GAN稳定多了，但实际操作中还是有几点要注意：

**学习率别设太高**：0.0002配合Adam的betas=(0.5, 0.999)是比较经典的选择。

**判别器和生成器的训练要平衡**：如果判别器Loss一直很低，说明它太强了，生成器学不动。

**观察生成的图像**：如果一直是一堆噪声或者模式崩溃，早点发现问题。

**批量大小**：256或512是比较常用的，太小了训练不稳定，太大了显存可能不够。

---

## 4. WGAN：Wasserstein距离如何解决训练不稳定问题？

### 4.1 原始GAN为什么训练困难？

在说WGAN之前，得先搞清楚原始GAN为什么这么难训练。

原始GAN用的是JS散度来衡量生成分布和真实分布的差异。问题在于，当这两个分布完全不重叠时（这种情况在训练初期非常常见），JS散度的梯度几乎是零。这就好比你让一个盲人去学射击，告诉他"打中了"或"没打中"——这种二元反馈太粗糙了，根本没法指导他微调方向。

WGAN的作者Arjovsky想了一个更聪明的办法：用Wasserstein距离（也叫Earth-Mover距离）代替JS散度。

### 4.2 Wasserstein距离的直观理解

Wasserstein距离说的是"把一堆土从分布P运到分布Q，最少需要搬多少"。

举个例子：假设P是[0, 1]区间上的均匀分布，Q是[2, 3]区间上的均匀分布。要把P变成Q，你需要把质量向右移动至少2个单位，这就是Wasserstein距离。

这个距离的好处是：即使两个分布完全不重叠，它依然能反映它们之间的"远近"。而JS散度在完全不重叠时，只能告诉你"它们完全不同"，但没法告诉你"有多不同"。

### 4.3 WGAN的实现

WGAN的核心改动有三个：

```python
class WGAN_Discriminator(nn.Module):
    """WGAN判别器（论文中称为Critic）"""
    def __init__(self, channels=3, features_d=64):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(channels, features_d, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d, features_d * 2, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 2, features_d * 4, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 4, features_d * 8, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 8, 1, 4, 1, 0)  # 最后一层没有Sigmoid！
        )
    
    def forward(self, x):
        return self.net(x).view(x.size(0), -1).squeeze(1)


def wasserstein_loss(discriminator, real_images, fake_images, optimizer_d, clip_value=0.01):
    """
    WGAN损失函数
    关键改动：
    1. 判别器最后一层没有Sigmoid
    2. 用实际输出的均值作为"真伪程度"打分
    3. 训练判别器后要对权重进行裁剪
    """
    # 真图像打分
    real_validity = discriminator(real_images)
    # 假图像打分
    fake_validity = discriminator(fake_images)
    
    # WGAN损失：真实样本得分高，假样本得分低
    wasserstein_d = -(real_validity.mean() - fake_validity.mean())
    
    wasserstein_d.backward()
    optimizer_d.step()
    
    # 权重裁剪：限制判别器权重在 [-clip_value, clip_value]
    for p in discriminator.parameters():
        p.data.clamp_(-clip_value, clip_value)
    
    return wasserstein_d.item()


def train_wgan(generator, discriminator, dataloader, epochs=100, device='cuda'):
    """WGAN训练循环"""
    optimizer_g = torch.optim.RMSprop(generator.parameters(), lr=0.00005)
    optimizer_d = torch.optim.RMSprop(discriminator.parameters(), lr=0.00005)
    
    for epoch in range(epochs):
        for batch_idx, real_images in enumerate(dataloader):
            real_images = real_images.to(device)
            batch_size = real_images.size(0)
            
            # 训练判别器（Critic）多次
            for _ in range(5):
                noise = torch.randn(batch_size, 100, device=device)
                fake_images = generator(noise)
                
                optimizer_d.zero_grad()
                d_loss = wasserstein_loss(discriminator, real_images, fake_images, optimizer_d)
            
            # 训练生成器一次
            noise = torch.randn(batch_size, 100, device=device)
            fake_images = generator(noise)
            
            optimizer_g.zero_grad()
            fake_validity = discriminator(fake_images)
            g_loss = -fake_validity.mean()  # 生成器希望假图像得分越高越好
            g_loss.backward()
            optimizer_g.step()
```

### 4.4 WGAN vs 原始GAN

WGAN相比原始GAN有几个显著优势：

**训练稳定多了**：不再需要小心翼翼地平衡判别器和生成器的训练速度。

**有有意义的损失函数**：你可以直接用判别器的输出来监控训练进度——数值越小说明分布越接近。

**不再需要批归一化**：虽然加上也行，但不加也能训练。

不过WGAN也有缺点：权重裁剪这个操作比较粗暴，可能会限制网络的表达能力，而且裁剪值的大小需要调参。

---

## 5. WGAN-GP：梯度惩罚的具体实现

### 5.1 权重裁剪的局限性

WGAN用权重裁剪来满足Lipschitz约束，但这个方法有点简单粗暴。裁剪值设大了，约束太松；设小了，网络容量受限。而且在深层网络中，权重裁剪可能导致梯度消失或爆炸。

WGAN-GP（WGAN with Gradient Penalty）用一种更优雅的方式来满足Lipschitz约束。

### 5.2 梯度惩罚的原理

梯度惩罚的思路是：不去硬裁剪权重，而是让判别器对输入的梯度范数接近1。

具体来说，WGAN-GP在损失函数中加入一个惩罚项：

$$
\mathcal{L}_{GP} = \lambda \mathbb{E}_{\hat{x} \sim p_{\hat{x}}} [(||\nabla_{\hat{x}} D(\hat{x})||_2 - 1)^2]
$$

其中 $\hat{x}$ 是在真实图像和生成图像之间随机插值得到的样本。

这个惩罚项的意思是：让判别器对任意输入的梯度范数都接近1，这样就满足了1-Lipschitz约束。

### 5.3 WGAN-GP的PyTorch实现

```python
class WGANGP_Discriminator(nn.Module):
    """WGAN-GP判别器"""
    def __init__(self, channels=3, features_d=64):
        super().__init__()
        self.net = nn.Sequential(
            nn.Conv2d(channels, features_d, 4, 2, 1),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d, features_d * 2, 4, 2, 1),
            nn.InstanceNorm2d(features_d * 2),  # WGAN-GP用InstanceNorm代替BatchNorm
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 2, features_d * 4, 4, 2, 1),
            nn.InstanceNorm2d(features_d * 4),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 4, features_d * 8, 4, 2, 1),
            nn.InstanceNorm2d(features_d * 8),
            nn.LeakyReLU(0.2, inplace=True),
            nn.Conv2d(features_d * 8, 1, 4, 1, 0)
        )
    
    def forward(self, x):
        return self.net(x).view(x.size(0), -1).squeeze(1)


def gradient_penalty(discriminator, real_images, fake_images, device):
    """
    计算梯度惩罚
    在真实图像和生成图像之间随机采样，然后计算判别器对这些样本的梯度
    """
    batch_size = real_images.size(0)
    epsilon = torch.rand(batch_size, 1, 1, 1, device=device)
    
    # 在真实图像和生成图像之间进行线性插值
    interpolated_images = epsilon * real_images + (1 - epsilon) * fake_images
    interpolated_images.requires_grad_(True)
    
    # 获取判别器对插值图像的输出
    mixed_scores = discriminator(interpolated_images)
    
    # 计算梯度
    gradients = torch.autograd.grad(
        outputs=mixed_scores,
        inputs=interpolated_images,
        grad_outputs=torch.ones_like(mixed_scores),
        create_graph=True,
        retain_graph=True,
        only_inputs=True
    )[0]
    
    # 计算梯度范数并惩罚与1的偏离
    gradients = gradients.view(batch_size, -1)
    gradient_norm = gradients.norm(2, dim=1)
    penalty = ((gradient_norm - 1) ** 2).mean()
    
    return penalty


def train_wgan_gp(generator, discriminator, dataloader, epochs=100, device='cuda'):
    """WGAN-GP训练循环"""
    optimizer_g = torch.optim.Adam(generator.parameters(), lr=0.0001, betas=(0, 0.9))
    optimizer_d = torch.optim.Adam(discriminator.parameters(), lr=0.0001, betas=(0, 0.9))
    
    lambda_gp = 10  # 梯度惩罚系数
    
    for epoch in range(epochs):
        for batch_idx, real_images in enumerate(dataloader):
            real_images = real_images.to(device)
            batch_size = real_images.size(0)
            
            # 训练判别器
            noise = torch.randn(batch_size, 100, device=device)
            fake_images = generator(noise)
            
            optimizer_d.zero_grad()
            
            real_validity = discriminator(real_images)
            fake_validity = discriminator(fake_images)
            
            # WGAN损失
            wasserstein_d = -(real_validity.mean() - fake_validity.mean())
            
            # 梯度惩罚
            gp = gradient_penalty(discriminator, real_images, fake_images, device)
            
            d_loss = wasserstein_d + lambda_gp * gp
            d_loss.backward()
            optimizer_d.step()
            
            # 训练生成器（每n步训练一次）
            if batch_idx % 5 == 0:
                optimizer_g.zero_grad()
                noise = torch.randn(batch_size, 100, device=device)
                fake_images = generator(noise)
                fake_validity = discriminator(fake_images)
                g_loss = -fake_validity.mean()
                g_loss.backward()
                optimizer_g.step()
```

### 5.4 WGAN-GP的实际效果

WGAN-GP在实际应用中比原始WGAN好用多了：

**训练更稳定**：不用再调权重裁剪的超参数。

**生成质量更好**：梯度惩罚让判别器有更好的梯度流。

**收敛更快**：判别器可以训练更多次而不担心过强。

WGAN-GP很长一段时间都是GAN训练的首选方案，直到后来又出现了Spectral Normalization等新方法。

---

## 6. 条件生成对抗网络（cGAN）

### 6.1 理论基础

条件生成对抗网络（Conditional GAN, cGAN）在原始 GAN 基础上引入条件信息 $y$，可以是类别标签、文本描述、图像等多种形式。目标函数修正为：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[\log D(\mathbf{x}|y)] + \mathbb{E}_{\mathbf{z} \sim p_{\mathbf{z}}(\mathbf{z})}[\log(1 - D(G(\mathbf{z}|y)|y))]
$$

### 6.2 AC-GAN：辅助分类器判别器

AC-GAN（Auxiliary Classifier GAN）同时优化分类精度和生成质量：

```python
class ACGAN_Discriminator(nn.Module):
    """AC-GAN 判别器：输出真假判断和类别预测"""
    def __init__(self, img_channels, num_classes, img_size):
        super().__init__()
        self.img_size = img_size
        
        # 特征提取
        self.conv = nn.Sequential(
            nn.Conv2d(img_channels, 32, 3, padding=1),
            nn.LeakyReLU(0.2),
            nn.Conv2d(32, 64, 3, stride=2, padding=1),
            nn.BatchNorm2d(64),
            nn.LeakyReLU(0.2),
            nn.Conv2d(64, 128, 3, stride=2, padding=1),
            nn.BatchNorm2d(128),
            nn.LeakyReLU(0.2),
            nn.Conv2d(128, 256, 3, stride=2, padding=1),
            nn.BatchNorm2d(256),
            nn.LeakyReLU(0.2),
        )
        
        # 真假判断输出
        self.discriminator = nn.Linear(256 * (img_size // 8) ** 2, 1)
        # 类别预测输出
        self.classifier = nn.Linear(256 * (img_size // 8) ** 2, num_classes)
    
    def forward(self, x):
        features = self.conv(x).view(x.size(0), -1)
        validity = self.discriminator(features)
        class_pred = self.classifier(features)
        return validity, class_pred

def acgan_loss(discriminator, real_images, fake_images, real_labels, generated_labels, device):
    """AC-GAN 损失函数"""
    real_validity, real_class = discriminator(real_images)
    fake_validity, fake_class = discriminator(fake_images.detach())
    
    # 真假损失
    real_loss = nn.BCEWithLogitsLoss()(real_validity, torch.ones_like(real_validity))
    fake_loss = nn.BCEWithLogitsLoss()(fake_validity, torch.zeros_like(fake_validity))
    d_loss = (real_loss + fake_loss) / 2
    
    # 分类损失（同时在真实和假图像上训练）
    class_loss = nn.CrossEntropyLoss()(torch.cat([real_class, fake_class], dim=0),
                                         torch.cat([real_labels, generated_labels], dim=0).to(device))
    
    return d_loss + class_loss
```

> [!note] AC-GAN 的优势
> AC-GAN 通过辅助分类器同时实现条件生成和分类任务，在 CIFAR-10 等数据集上取得了优异效果。但存在模式崩溃风险，因为判别器可能过度关注分类任务而忽视真假判断。

---

## 7. Pix2Pix：条件GAN在图像翻译中的应用

### 7.1 核心思想

Pix2Pix 由 Isola 等人在 2017 年提出，是首个通用的配对图像转换框架。其核心创新在于使用 U-Net 架构作为生成器，并引入 L1 损失而非 L2 损失：

$$
\mathcal{L}_{cGAN}(G, D) = \mathbb{E}_{\mathbf{x}, \mathbf{y}}[\log D(\mathbf{x}, \mathbf{y})] + \mathbb{E}_{\mathbf{x}, \mathbf{z}}[\log(1 - D(\mathbf{x}, G(\mathbf{x}, \mathbf{z})))]
$$

$$
\mathcal{L}_{L1}(G) = \mathbb{E}_{\mathbf{x}, \mathbf{y}, \mathbf{z}}[|\mathbf{y} - G(\mathbf{x}, \mathbf{z})|_1]
$$

总损失函数：$\mathcal{L} = \mathcal{L}_{cGAN} + \lambda \mathcal{L}_{L1}$

### 7.2 U-Net 生成器实现

```python
class UNetGenerator(nn.Module):
    """U-Net 生成器：编码器-解码器架构 + 跳跃连接"""
    def __init__(self, input_channels, num_filters=64):
        super().__init__()
        
        # 下采样（编码器）
        self.down1 = nn.Sequential(
            nn.Conv2d(input_channels, num_filters, 4, stride=2, padding=1),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down2 = nn.Sequential(
            nn.Conv2d(num_filters, num_filters * 2, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 2),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down3 = nn.Sequential(
            nn.Conv2d(num_filters * 2, num_filters * 4, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 4),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down4 = nn.Sequential(
            nn.Conv2d(num_filters * 4, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down5 = nn.Sequential(
            nn.Conv2d(num_filters * 8, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down6 = nn.Sequential(
            nn.Conv2d(num_filters * 8, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.down7 = nn.Sequential(
            nn.Conv2d(num_filters * 8, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.LeakyReLU(0.2, inplace=True)
        )
        self.bottleneck = nn.Sequential(
            nn.Conv2d(num_filters * 8, num_filters * 8, 4, stride=2, padding=1),
            nn.ReLU(inplace=True)
        )
        
        # 上采样（解码器）+ 跳跃连接
        self.up1 = nn.Sequential(
            nn.ConvTranspose2d(num_filters * 8, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.Dropout(0.5),
            nn.ReLU(inplace=True)
        )
        self.up2 = nn.Sequential(
            nn.ConvTranspose2d(num_filters * 16, num_filters * 8, 4, stride=2, padding=1),
            nn.BatchNorm2d(num_filters * 8),
            nn.Dropout(0.5),
            nn.ReLU(inplace=True)
        )
        # ... 省略部分上采样层
    
    def forward(self, x):
        # 编码
        d1 = self.down1(x)
        d2 = self.down2(d1)
        d3 = self.down3(d2)
        d4 = self.down4(d3)
        d5 = self.down5(d4)
        d6 = self.down6(d5)
        d7 = self.down7(d6)
        bottleneck = self.bottleneck(d7)
        
        # 解码 + 跳跃连接
        u1 = self.up1(bottleneck)
        u1 = torch.cat([u1, d7], dim=1)
        u2 = self.up2(u1)
        u2 = torch.cat([u2, d6], dim=1)
        # ... 继续上采样和跳跃连接
        
        return self.final(nn.Sequential(*[u1, u2, ...]))  # 简化表示
```

### 7.3 PatchGAN 判别器

Pix2Pix 使用 PatchGAN 判别器，只判断图像块的真假，而非整个图像：

```python
class PatchGANDiscriminator(nn.Module):
    """PatchGAN 判别器：输出 N×N 补丁级别的真假判断"""
    def __init__(self, input_channels, num_filters=64, n_layers=3):
        super().__init__()
        
        layers = []
        # 初始卷积
        layers.append(nn.Conv2d(input_channels * 2, num_filters, 4, stride=2, padding=1))
        layers.append(nn.LeakyReLU(0.2, inplace=True))
        
        # 中间层
        nf_mult = 1
        for i in range(1, n_layers):
            nf_mult_prev = nf_mult
            nf_mult = min(2 ** i, 8)
            layers.append(nn.Conv2d(num_filters * nf_mult_prev, num_filters * nf_mult,
                                    4, stride=2, padding=1))
            layers.append(nn.BatchNorm2d(num_filters * nf_mult))
            layers.append(nn.LeakyReLU(0.2, inplace=True))
        
        # 输出层：1×1 卷积输出概率图
        layers.append(nn.Conv2d(num_filters * nf_mult, 1, 4, stride=1, padding=1))
        
        self.model = nn.Sequential(*layers)
    
    def forward(self, x, y):
        # 将输入图像和目标图像在通道维度拼接
        return self.model(torch.cat([x, y], dim=1))
```

---

## 8. CycleGAN：无需配对数据的图像翻译

### 8.1 循环一致性原理

CycleGAN 的核心创新是循环一致性损失（Cycle Consistency Loss），允许在没有配对数据的情况下学习两个域之间的转换：

$$
\mathcal{L}_{\text{cyc}}(G, F) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[|F(G(\mathbf{x})) - \mathbf{x}|_1] + \mathbb{E}_{\mathbf{y} \sim p_{\text{data}}(\mathbf{y})}[|G(F(\mathbf{y})) - \mathbf{y}|_1]
$$

完整损失：
$$
\mathcal{L}(G, F, D_X, D_Y) = \mathcal{L}_{cGAN}(G, D_Y, X, Y) + \mathcal{L}_{cGAN}(F, D_X, Y, X) + \lambda \mathcal{L}_{\text{cyc}}(G, F)
$$

### 8.2 CycleGAN 架构实现

```python
class ResidualBlock(nn.Module):
    """残差块：用于 CycleGAN 生成器"""
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
    """CycleGAN 生成器：编码器-残差块-解码器架构"""
    def __init__(self, input_channels=3, num_residual_blocks=9):
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
        residual_blocks = [ResidualBlock(256) for _ in range(num_residual_blocks)]
        self.residuals = nn.Sequential(*residual_blocks)
        
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

class CycleGANLosses:
    """CycleGAN 损失计算"""
    def __init__(self, lambda_cycle=10.0):
        self.lambda_cycle = lambda_cycle
        self.mse_loss = nn.MSELoss()
        self.l1_loss = nn.L1Loss()
    
    def generator_loss(self, fake_output):
        """对抗损失：让假图像被判别器识别为真"""
        return self.mse_loss(fake_output, torch.ones_like(fake_output))
    
    def cycle_consistency_loss(self, real_image, reconstructed_image,
                               fake_image, reconstructed_back):
        """循环一致性损失"""
        forward_loss = self.l1_loss(reconstructed_image, real_image)
        backward_loss = self.l1_loss(reconstructed_back, fake_image)
        return forward_loss + backward_loss
    
    def identity_loss(self, real_image, same_image):
        """身份损失：G(y) 应接近 y"""
        return self.l1_loss(same_image, real_image)
```

> [!tip] CycleGAN 应用场景
> - 马到斑马的转换
> - 莫奈画作到照片的转换
> - 夏季风景到冬季风景的转换
> - 素描到彩色图像的转换

---

## 9. StyleGAN：从粗糙到精细的图像生成

### 9.1 StyleGAN的核心创新

StyleGAN是NVIDIA在2018年提出的惊人成果，它第一次让我们能够控制生成图像的"风格"——从粗略的轮廓到细微的纹理，都可以独立调节。

StyleGAN的核心创新是**自适应实例归一化（Adaptive Instance Normalization, AdaIN）**和**映射网络（Mapping Network）**。

**映射网络**的作用是把随机噪声z映射到一个中间潜空间W。这个W空间的每个维度可能有更明确的语义含义，不像原始z空间那样纠缠在一起。

**AdaIN**则负责把这个W空间的向量"注入"到生成器的每一层。具体来说：

$$
\text{AdaIN}(x, y) = y_s \cdot \frac{x - \mu(x)}{\sigma(x)} + y_b
$$

其中 $y_s$ 和 $y_b$ 是从W空间向量学到的缩放和偏置参数。这样每一层都可以接收来自W空间的控制信号，从而影响图像的不同细节层次。

### 9.2 StyleGAN的基本结构

```python
class StyleGANGenerator(nn.Module):
    """StyleGAN生成器简化版"""
    def __init__(self, latent_dim=512, n_mapping=8):
        super().__init__()
        self.latent_dim = latent_dim
        
        # 映射网络：把z映射到W空间
        self.mapping = nn.Sequential(
            nn.Linear(latent_dim, latent_dim),
            nn.LeakyReLU(0.2),
            *[self._make_mapping_block(latent_dim) for _ in range(n_mapping - 1)]
        )
        
        # 每一层的Style（AdaIN参数）
        self.style_layers = nn.ModuleList([
            nn.Linear(latent_dim, 2 * channels) 
            for channels in [512, 512, 512, 512, 256, 128, 64, 32, 16]
        ])
        
        # 初始卷积层
        self.initial_conv = nn.Conv2d(latent_dim, 512, 3, padding=1)
        self.initial_noise = nn.Parameter(torch.randn(1, 512, 4, 4))
        
        # 合成网络
        self.synthesis = self._build_synthesis_network()
    
    def _make_mapping_block(self, dim):
        return nn.Sequential(
            nn.Linear(dim, dim),
            nn.LeakyReLU(0.2)
        )
    
    def _build_synthesis_network(self):
        layers = []
        channels = [512, 512, 512, 512, 256, 128, 64, 32, 16]
        
        for i in range(len(channels) - 1):
            layers.append(nn.Sequential(
                nn.Conv2d(channels[i], channels[i + 1], 3, padding=1),
                nn.LeakyReLU(0.2)
            ))
        
        return nn.Sequential(*layers)
    
    def adain(self, x, style):
        """自适应实例归一化"""
        size = x.size()
        x = x.view(x.size(0), x.size(1), -1)
        mean = x.mean(dim=2, keepdim=True)
        std = x.std(dim=2, keepdim=True) + 1e-8
        x = (x - mean) / std
        
        scale, bias = style[:, :size[1]], style[:, size[1]:]
        x = x * scale.view(*size[:2], 1) + bias.view(*size[:2], 1)
        return x.view(*size)
    
    def forward(self, z, styles=None):
        # 映射到W空间
        w = self.mapping(z)
        
        # 如果没有指定styles，从w复制到每一层
        if styles is None:
            styles = [w for _ in self.style_layers]
        
        # 初始层
        x = self.initial_conv(self.initial_noise)
        
        # 逐步上采样，每层注入不同的style
        for i, (layer, style) in enumerate(zip(self.synthesis, styles)):
            x = layer(x)
            if i < len(self.style_layers):
                style_params = self.style_layers[i](style)
                x = self.adain(x, style_params)
        
        return x
```

### 9.3 StyleGAN的风格混合

StyleGAN的一个很酷的特性是**风格混合（Style Mixing）**：你可以用不同的W向量控制不同的层级。

比如：
- 用向量A的低层（粗粒度）控制轮廓和姿态
- 用向量B的高层（细粒度）控制颜色和纹理

这样就能生成"混合风格"的新图像，这在实际应用中非常有价值。

---

## 10. StyleGAN2/3：改进的架构与训练稳定性

### 10.1 StyleGAN2的改进

StyleGAN2在StyleGAN的基础上做了几项重要改进：

**移除渐进式增长**：StyleGAN的渐进式增长虽然能生成高分辨率图像，但会导致细节被"锁定"在特定分辨率的问题。StyleGAN2改用更直接的架构。

**权重解调（Weight Demodulation）**：用一种更简洁的方式实现类似AdaIN的效果，同时让训练更稳定。

**路径长度正则化（Path Length Regularization）**：让生成器学到的映射更平滑。

### 10.2 StyleGAN3的革命性改进

StyleGAN3更进一步，解决了StyleGAN2中图像"抖动"的问题——即纹理看起来像是粘在屏幕上而不是真正附着在物体表面。

StyleGAN3的核心改进是**等变卷积（Equivariant Convolution）**，通过仔细设计上采样和滤波器的结构，让特征图能够自然地随着底层图像变换而变换。

这对于需要精确控制的视频生成任务特别重要。

---

## 11. Progressive GAN：从小图逐步长大的训练策略

### 11.1 渐进式训练的思路

Progressive GAN（简称ProGAN）是Karras等人2018年提出的，它的想法很简单但很有效：**从小分辨率开始训练，逐步增大**。

训练过程是这样的：
1. 先训练4x4分辨率的生成器和判别器
2. 然后同时给两边各加一层，扩展到8x8
3. 继续训练到16x16，然后32x32，64x64...
4. 直到达到目标分辨率

这样做的好处是：
- 低分辨率阶段，模型学习的是整体的"构图"——物体大概长什么样
- 高分辨率阶段，只需要专注于添加细节，而不用从头学习整体结构
- 训练更稳定，因为低分辨率时模型收敛快，能快速进入良性循环

### 11.2 渐进式增长的实现细节

```python
class ProgressiveGAN:
    """渐进式GAN的简化实现"""
    def __init__(self, latent_dim=512, max_resolution=1024):
        self.latent_dim = latent_dim
        self.max_resolution = max_resolution
        self.current_depth = 3  # 从8x8开始
        
        # 初始化网络（只在需要时构建）
        self.generator = self._build_generator()
        self.discriminator = self._build_discriminator()
    
    def fade_in(self, model, new_layers, alpha):
        """
        平滑过渡：新加的层从0慢慢增长到完全接管
        alpha从0到1，表示新层的重要性逐渐增加
        """
        # 这个函数实现新层和旧层输出的加权平均
        return alpha * new_layers + (1 - alpha) * old_layers
    
    def grow_network(self):
        """网络增长：添加新的上采样/下采样层"""
        self.current_depth += 1
        self.generator.add_block()
        self.discriminator.add_block()
    
    def train_step(self, batch_size, alpha):
        """
        训练步骤
        alpha: 过渡阶段使用的混合系数
        """
        target_resolution = 2 ** self.current_depth
        
        # 生成当前分辨率的图像
        noise = torch.randn(batch_size, self.latent_dim)
        fake_images = self.generator(noise, alpha=alpha)
        
        # 训练判别器和生成器
        # ... 标准GAN训练代码 ...
```

### 11.3 ProGAN vs 直接训练

如果你直接训练一个1024x1024的GAN，问题会很多：
- 显存消耗巨大
- 训练时间长且不稳定
- 很容易模式崩溃

ProGAN的渐进式训练把这些问题都缓解了。这也是为什么后来的StyleGAN、BigGAN都借鉴了这种思路。

---

## 12. BigGAN：大尺度图像生成

### 12.1 核心技术创新

BigGAN 由 Brock 等人在 2019 年提出，实现了当时最高质量的 ImageNet 条件生成。其核心技术包括：

1. **大批次训练**：batch size 达到 2048
2. **类别平衡采样**：缓解类别不平衡问题
3. **截断技巧（Truncation Trick）**：控制生成样本的多样性-质量权衡
4. **谱归一化**：稳定训练过程

### 12.2 BigGAN 架构细节

```python
class SelfAttention(nn.Module):
    """自注意力模块：BigGAN 的关键组件"""
    def __init__(self, in_channels):
        super().__init__()
        self.in_channels = in_channels
        self.query = nn.Conv2d(in_channels, in_channels // 8, 1)
        self.key = nn.Conv2d(in_channels, in_channels // 8, 1)
        self.value = nn.Conv2d(in_channels, in_channels, 1)
        self.gamma = nn.Parameter(torch.zeros(1))
    
    def forward(self, x):
        batch_size, C, H, W = x.size()
        
        # 简化注意力计算
        f = self.query(x).view(batch_size, -1, H * W)  # B, C', HW
        g = self.key(x).view(batch_size, -1, H * W)    # B, C', HW
        h = self.value(x).view(batch_size, -1, H * W)   # B, C, HW
        
        # 计算注意力矩阵
        s = torch.bmm(f.permute(0, 2, 1), g)  # B, HW, HW
        beta = torch.softmax(s, dim=-1)
        
        # 加权聚合
        o = torch.bmm(h, beta.permute(0, 2, 1))  # B, C, HW
        o = o.view(batch_size, C, H, W)
        
        return self.gamma * o + x

class BigGANGenerator(nn.Module):
    """BigGAN 生成器架构"""
    def __init__(self, latent_dim, num_classes, channels=3, num_features=96):
        super().__init__()
        self.num_classes = num_classes
        
        # 共享嵌入层（类别嵌入与线性层权重共享）
        self.embed = nn.Embedding(num_classes, latent_dim)
        
        # 初始块
        self.initial = nn.Sequential(
            nn.Linear(latent_dim, 4 * 4 * 16 * num_features),
            nn.BatchNorm1d(4 * 4 * 16 * num_features)
        )
        
        # 中间层（渐进式上采样 + 自注意力）
        self.layers = nn.ModuleList([
            nn.Sequential(
                nn.ConvTranspose2d(16 * num_features, 16 * num_features, 4, stride=2, padding=1),
                nn.BatchNorm2d(16 * num_features),
                nn.ReLU(inplace=True)
            ),
            SelfAttention(16 * num_features),  # 插入自注意力
            nn.Sequential(
                nn.ConvTranspose2d(16 * num_features, 8 * num_features, 4, stride=2, padding=1),
                nn.BatchNorm2d(8 * num_features),
                nn.ReLU(inplace=True)
            ),
            SelfAttention(8 * num_features),
            # ... 更多层
        ])
        
        # 输出层
        self.output = nn.Sequential(
            nn.Conv2d(num_features, channels, 3, stride=1, padding=1),
            nn.Tanh()
        )
    
    def forward(self, z, class_labels):
        # 嵌入类别并与噪声拼接
        class_embed = self.embed(class_labels)
        input_vector = z + class_embed
        
        # 生成初始特征
        x = self.initial(input_vector)
        x = x.view(-1, 16 * 96, 4, 4)
        
        # 通过各层
        for layer in self.layers:
            x = layer(x)
        
        return self.output(x)

def biggan_training_techniques():
    """
    BigGAN 关键技术：
    1. 类别平衡采样：确保所有类别均匀采样
    2. 截断技巧：在推理时限制 z 的范围以提高质量
    3. 谱归一化：限制判别器的 Lipschitz 常数
    """
    pass

# 截断技巧示例
def truncate_z(z, threshold=2.0):
    """截断 z 向量以提高生成质量（但降低多样性）"""
    return np.clip(z, -threshold, threshold) * threshold / np.maximum(np.abs(z), threshold)
```

---

## 13. SAGAN：自注意力生成对抗网络

### 13.1 自注意力机制

SAGAN（Self-Attention GAN）在 GAN 中引入自注意力机制，使模型能够捕获图像中远距离区域之间的依赖关系：

```python
class SelfAttentionBlock(nn.Module):
    """SAGAN 自注意力块"""
    def __init__(self, in_channels):
        super().__init__()
        self.in_channels = in_channels
        self.query_conv = nn.Conv2d(in_channels, in_channels // 8, 1)
        self.key_conv = nn.Conv2d(in_channels, in_channels // 8, 1)
        self.value_conv = nn.Conv2d(in_channels, in_channels, 1)
        self.gamma = nn.Parameter(torch.zeros(1))
        
        # 初始化 gamma 为 0，使注意力机制从弱到强逐渐学习
        nn.init.zeros_(self.gamma)
    
    def forward(self, x):
        batch_size, C, H, W = x.size()
        
        # 线性变换
        query = self.query_conv(x).view(batch_size, -1, H * W).permute(0, 2, 1)  # B, HW, C'
        key = self.key_conv(x).view(batch_size, -1, H * W)  # B, C', HW
        value = self.value_conv(x).view(batch_size, -1, H * W)  # B, C, HW
        
        # 注意力权重
        attention = torch.bmm(query, key)  # B, HW, HW
        attention = attention / (self.in_channels ** 0.5)
        attention = torch.softmax(attention, dim=-1)
        
        # 加权求和
        out = torch.bmm(value, attention.permute(0, 2, 1))  # B, C, HW
        out = out.view(batch_size, C, H, W)
        
        return self.gamma * out + x
```

> [!note] SAGAN vs BigGAN
> SAGAN 的自注意力机制被 BigGAN 采用并改进。两者的核心区别在于：SAGAN 在单尺度上应用注意力，而 BigGAN 在多个尺度上集成注意力，并结合大批次训练和条件技术。

---

## 14. VAE-GAN：用VAE改进GAN的潜在空间

### 14.1 VAE和GAN的结合

VAE-GAN是Larsen等人2016年提出的，它把变分自编码器（VAE）和GAN的优点结合起来。

VAE的好处是有一个结构化的潜在空间，而且encoder可以直接把图像映射到潜在向量，这样就能做"图像重建"和"潜在空间插值"这类操作。GAN的好处是生成的图像质量高。

VAE-GAN的想法是：
- 用VAE的encoder把真实图像编码到潜在空间
- 用GAN的generator从潜在向量生成图像
- 让生成的图像和真实图像尽可能相似

### 14.2 VAE-GAN的损失函数

VAE-GAN的损失函数由三部分组成：

**VAE重建损失**：让decoder生成的图像接近encoder输入的真实图像。

**GAN对抗损失**：让decoder生成的图像骗过判别器。

**潜在空间正则化**：让encoder学到的潜在分布接近标准正态分布。

```python
class VAEGAN_Generator(nn.Module):
    """VAE-GAN生成器（Decoder）"""
    def __init__(self, latent_dim=100, channels=3, features=64):
        super().__init__()
        
        # 类似DCGAN的生成器
        self.net = nn.Sequential(
            nn.ConvTranspose2d(latent_dim, features * 8, 4, 1, 0),
            nn.BatchNorm2d(features * 8),
            nn.ReLU(True),
            nn.ConvTranspose2d(features * 8, features * 4, 4, 2, 1),
            nn.BatchNorm2d(features * 4),
            nn.ReLU(True),
            nn.ConvTranspose2d(features * 4, features * 2, 4, 2, 1),
            nn.BatchNorm2d(features * 2),
            nn.ReLU(True),
            nn.ConvTranspose2d(features * 2, features, 4, 2, 1),
            nn.BatchNorm2d(features),
            nn.ReLU(True),
            nn.ConvTranspose2d(features, channels, 4, 2, 1),
            nn.Tanh()
        )
    
    def forward(self, z):
        return self.net(z.view(z.size(0), -1, 1, 1))


class VAEGAN_Encoder(nn.Module):
    """VAE-GAN编码器"""
    def __init__(self, channels=3, features=64, latent_dim=100):
        super().__init__()
        self.latent_dim = latent_dim
        
        self.net = nn.Sequential(
            nn.Conv2d(channels, features, 4, 2, 1),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features, features * 2, 4, 2, 1),
            nn.BatchNorm2d(features * 2),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features * 2, features * 4, 4, 2, 1),
            nn.BatchNorm2d(features * 4),
            nn.LeakyReLU(0.2),
            nn.Conv2d(features * 4, features * 8, 4, 2, 1),
            nn.BatchNorm2d(features * 8),
            nn.LeakyReLU(0.2),
        )
        
        # 输出均值和方差
        self.fc_mean = nn.Linear(features * 8 * 4 * 4, latent_dim)
        self.fc_logvar = nn.Linear(features * 8 * 4 * 4, latent_dim)
    
    def forward(self, x):
        x = self.net(x).view(x.size(0), -1)
        mu = self.fc_mean(x)
        logvar = self.fc_logvar(x)
        
        # 重参数化技巧
        std = torch.exp(0.5 * logvar)
        eps = torch.randn_like(std)
        z = mu + eps * std
        
        return z, mu, logvar


def vae_gan_loss(encoder, decoder, discriminator, real_images, latent_dim):
    """
    VAE-GAN损失计算
    """
    # 编码
    z, mu, logvar = encoder(real_images)
    
    # 解码（重建）
    reconstructed = decoder(z)
    
    # VAE重建损失
    reconstruction_loss = nn.L1Loss()(reconstructed, real_images)
    
    # KL散度
    kl_loss = -0.5 * torch.mean(1 + logvar - mu.pow(2) - logvar.exp())
    
    # GAN损失（判别器）
    real_validity = discriminator(real_images)
    fake_validity = discriminator(reconstructed)
    
    # 生成器（解码器）希望重建图像被判别器识别为真
    g_loss = nn.BCEWithLogitsLoss()(fake_validity, torch.ones_like(fake_validity))
    
    # 判别器希望区分真实图像和重建图像
    d_loss_real = nn.BCEWithLogitsLoss()(real_validity, torch.ones_like(real_validity))
    d_loss_fake = nn.BCEWithLogitsLoss()(fake_validity, torch.zeros_like(fake_validity))
    d_loss = (d_loss_real + d_loss_fake) / 2
    
    # 总损失
    total_loss = reconstruction_loss + kl_loss + g_loss
    
    return total_loss, d_loss, reconstruction_loss, kl_loss
```

---

## 15. GAN 评估指标

### 15.1 Inception Score (IS)

Inception Score 通过预训练的 Inception 模型评估生成样本的质量和多样性：

$$
\text{IS} = \exp\left(\mathbb{E}_{\mathbf{x}} \left[ D_{\text{KL}}(p(y|\mathbf{x}) \| p(y) \right]\right)
$$

其中 $p(y|\mathbf{x})$ 是 Inception 模型对图像 $\mathbf{x}$ 的类别预测分布，$p(y) = \mathbb{E}_{\mathbf{x}}[p(y|\mathbf{x})]$ 是边缘分布。

```python
import torch
from torchvision.models import inception_v3
import torch.nn.functional as F

def calculate_inception_score(images, model, splits=10):
    """
    计算 Inception Score
    
    高分条件：
    - 每个生成图像有清晰的类别预测 (p(y|x) 接近 one-hot)
    - 生成图像涵盖多个类别 (p(y) 接近均匀分布)
    """
    model.eval()
    with torch.no_grad():
        # 获取 Inception 特征
        preds = model(images)  # B x 1000
        
        # 计算边缘分布 p(y)
        py = preds.mean(dim=0)
        
        # 计算每个样本的 KL 散度
        kl_divs = []
        for i in range(len(images)):
            pyx = F.softmax(preds[i], dim=-1)
            kl_div = F.kl_div(torch.log(pyx + 1e-10), py, reduction='batchmean')
            kl_divs.append(kl_div)
        
        # 计算 IS
        is_score = torch.exp(torch.stack(kl_divs).mean())
    
    return is_score.item()
```

### 15.2 Fréchet Inception Distance (FID)

FID 通过比较真实图像和生成图像在 Inception 特征空间中的分布来评估：

$$
\text{FID} = \|\mu_r - \mu_g\|^2 + \text{Tr}(\Sigma_r + \Sigma_g - 2\sqrt{\Sigma_r \Sigma_g})
$$

其中 $(\mu_r, \Sigma_r)$ 和 $(\mu_g, \Sigma_g)$ 分别是真实图像和生成图像特征的均值和协方差矩阵。

```python
import numpy as np
from scipy import linalg

def calculate_fid(real_features, fake_features):
    """
    计算 Fréchet Inception Distance
    
    FID 越低表示生成图像质量越高
    """
    # 计算均值
    mu_real = np.mean(real_features, axis=0)
    mu_fake = np.mean(fake_features, axis=0)
    
    # 计算协方差矩阵
    sigma_real = np.cov(real_features, rowvar=False)
    sigma_fake = np.cov(fake_features, rowvar=False)
    
    # 计算 FID
    diff = mu_real - mu_fake
    covmean = linalg.sqrtm(sigma_real @ sigma_fake)
    
    # 处理复数（数值误差）
    if np.iscomplexobj(covmean):
        covmean = covmean.real
    
    fid = diff @ diff + np.trace(sigma_real + sigma_fake - 2 * covmean)
    
    return fid

def extract_inception_features(dataloader, model, device):
    """提取 Inception 特征用于 FID 计算"""
    features = []
    model.eval()
    
    with torch.no_grad():
        for images, _ in dataloader:
            images = images.to(device)
            feat = model(images)
            # 使用池化层输出而非分类层
            feat = F.adaptive_avg_pool2d(feat, output_size=(1, 1))
            features.append(feat.view(feat.size(0), -1).cpu().numpy())
    
    return np.vstack(features)
```

> [!tip] IS vs FID
> - **Inception Score (IS)**：只衡量生成样本本身的质量和多样性，不与真实数据比较
> - **Fréchet Inception Distance (FID)**：直接比较生成分布与真实分布，衡量样本覆盖度和质量
> - FID 更常用，因为它能捕捉模式崩溃等全局问题

---

## 16. GAN变体对比表：按任务选择合适的GAN

| 任务场景 | 推荐GAN | 理由 | 替代方案 |
|----------|---------|------|----------|
| **基础图像生成** | DCGAN | 简单易实现，训练稳定 | WGAN-GP |
| **训练不稳定** | WGAN-GP | 损失函数设计合理，训练稳定 | WGAN, R1正则化 |
| **需要控制生成类别** | cGAN/AC-GAN | 条件信息控制能力强 | BigGAN |
| **配对图像翻译** | Pix2Pix | 配对数据下效果最佳 | CUT |
| **无配对图像翻译** | CycleGAN | 无需配对数据，应用广泛 | MUNIT, UNIT |
| **高分辨率图像** | StyleGAN2/3 | 生成质量最高，细节丰富 | Progressive GAN |
| **大规模ImageNet** | BigGAN | 高分辨率高质量，类别多样 | StyleGAN + 条件 |
| **长距离依赖** | SAGAN | 自注意力捕获全局信息 | BigGAN |
| **潜在空间操作** | VAE-GAN | 结构化潜在空间，支持插值 | BEGAN |
| **艺术风格转换** | CycleGAN | 无需成对数据，可跨域转换 | StarGAN v2 |

---

## 17. 实战：选择合适的GAN用于你的项目

### 17.1 先问自己几个问题

在选择GAN变体之前，先想清楚这些：

**你的数据是什么样的？**
- 有配对数据（输入-输出图像对）→ Pix2Pix
- 无配对数据，但有域的概念（马→斑马）→ CycleGAN
- 只有独立图像，没有对应关系 → DCGAN、WGAN-GP

**你需要控制生成结果吗？**
- 只需要生成随机样本 → DCGAN
- 需要指定类别 → cGAN
- 需要细粒度控制风格 → StyleGAN系列

**你对生成质量的要求有多高？**
- 随便玩玩 → DCGAN足够
- 追求SOTA → StyleGAN2/3或BigGAN
- 需要商用级质量 → StyleGAN3

**你的硬件条件如何？**
- 普通GPU（<16GB显存）→ DCGAN、WGAN-GP
- 强力GPU（>24GB）→ BigGAN、StyleGAN2
- 多卡集群 → BigGAN可以跑更大的batch

### 17.2 快速开始建议

**新手入门**：从DCGAN或WGAN-GP开始，理解GAN的基本训练流程。

**图像翻译任务**：
- 有配对数据 → 直接用Pix2Pix
- 无配对数据 → CycleGAN
- 需要多域转换 → StarGAN v2

**追求高质量**：直接用StyleGAN2/3，预训练模型众多，迁移学习方便。

**做研究**：BigGAN在ImageNet上的表现是baseline，很多新方法都拿它对比。

### 17.3 避坑指南

**别一上来就用StyleGAN**：它的代码复杂，调试困难，出了问题都不知道去哪找。

**别忽视判别器的训练**：很多GAN变种的问题根源都在判别器太强或太弱。

**保存中间结果**：GAN训练经常突然崩溃或者退化，定期保存生成样本很有必要。

**FID比IS更可靠**：IS容易被对抗样本欺骗，FID更全面。

---

## 18. 学术引用与参考文献

1. Mirza, M., & Osindero, S. (2014). "Conditional Generative Adversarial Nets." *arXiv:1411.1784*.
2. Odena, A., Olah, C., & Shlens, J. (2017). "Conditional Image Synthesis with Auxiliary Classifier GANs." *ICML*.
3. Isola, P., et al. (2017). "Image-to-Image Translation with Conditional Adversarial Networks." *CVPR*.
4. Zhu, J. Y., et al. (2017). "Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks." *ICCV*.
5. Brock, A., Donahue, J., & Simonyan, K. (2019). "Large Scale GAN Training for High Fidelity Natural Image Synthesis." *ICLR*.
6. Zhang, H., et al. (2019). "Self-Attention Generative Adversarial Networks." *ICML*.
7. Heusel, M., et al. (2017). "GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium." *NeurIPS*.
8. Salimans, T., et al. (2016). "Improved Techniques for Training GANs." *NeurIPS*.
9. Radford, A., Metz, L., & Chintala, S. (2016). "Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks." *ICLR*.
10. Arjovsky, M., Chintala, S., & Bottou, L. (2017). "Wasserstein GAN." *ICML*.
11. Gulrajani, I., et al. (2017). "Improved Training of Wasserstein GANs." *NeurIPS*.
12. Karras, T., et al. (2018). "Progressive Growing of GANs for Improved Quality, Stability, and Variation." *ICLR*.
13. Karras, T., et al. (2019). "A Style-Based Generator Architecture for Generative Adversarial Networks." *CVPR*.
14. Karras, T., et al. (2021). "Alias-Free Generative Adversarial Networks." *NeurIPS*.
15. Larsen, A. B. L., et al. (2016). "Autoencoding Beyond Pixels Using a Learned Similarity Metric." *ICML*.

---

## 19. 相关文档

- [[GAN深度指南]] - 原始 GAN、DCGAN、Wasserstein GAN 等基础架构
- [[对抗样本深度指南]] - 对抗攻击与防御的原理
- [[博弈论与AI]] - GAN 的博弈论理论基础
- [[对抗训练与鲁棒性]] - 对抗训练方法与鲁棒性提升
