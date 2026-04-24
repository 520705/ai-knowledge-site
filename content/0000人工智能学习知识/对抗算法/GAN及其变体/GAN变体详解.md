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

## 2. 条件生成对抗网络（cGAN）

### 2.1 理论基础

条件生成对抗网络（Conditional GAN, cGAN）在原始 GAN 基础上引入条件信息 $y$，可以是类别标签、文本描述、图像等多种形式。目标函数修正为：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[\log D(\mathbf{x}|y)] + \mathbb{E}_{\mathbf{z} \sim p_{\mathbf{z}}(\mathbf{z})}[\log(1 - D(G(\mathbf{z}|y)|y))]
$$

### 2.2 AC-GAN：辅助分类器判别器

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

## 3. Pix2Pix：图像到图像转换

### 3.1 核心思想

Pix2Pix 由 Isola 等人在 2017 年提出，是首个通用的配对图像转换框架。其核心创新在于使用 U-Net 架构作为生成器，并引入 L1 损失而非 L2 损失：

$$
\mathcal{L}_{cGAN}(G, D) = \mathbb{E}_{\mathbf{x}, \mathbf{y}}[\log D(\mathbf{x}, \mathbf{y})] + \mathbb{E}_{\mathbf{x}, \mathbf{z}}[\log(1 - D(\mathbf{x}, G(\mathbf{x}, \mathbf{z})))]
$$

$$
\mathcal{L}_{L1}(G) = \mathbb{E}_{\mathbf{x}, \mathbf{y}, \mathbf{z}}[|\mathbf{y} - G(\mathbf{x}, \mathbf{z})|_1]
$$

总损失函数：$\mathcal{L} = \mathcal{L}_{cGAN} + \lambda \mathcal{L}_{L1}$

### 3.2 U-Net 生成器实现

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

### 3.3 PatchGAN 判别器

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

## 4. CycleGAN：无配对图像转换

### 4.1 循环一致性原理

CycleGAN 的核心创新是循环一致性损失（Cycle Consistency Loss），允许在没有配对数据的情况下学习两个域之间的转换：

$$
\mathcal{L}_{\text{cyc}}(G, F) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}(\mathbf{x})}[|F(G(\mathbf{x})) - \mathbf{x}|_1] + \mathbb{E}_{\mathbf{y} \sim p_{\text{data}}(\mathbf{y})}[|G(F(\mathbf{y})) - \mathbf{y}|_1]
$$

完整损失：
$$
\mathcal{L}(G, F, D_X, D_Y) = \mathcal{L}_{cGAN}(G, D_Y, X, Y) + \mathcal{L}_{cGAN}(F, D_X, Y, X) + \lambda \mathcal{L}_{\text{cyc}}(G, F)
$$

### 4.2 CycleGAN 架构实现

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

## 5. BigGAN：大尺度图像生成

### 5.1 核心技术创新

BigGAN 由 Brock 等人在 2019 年提出，实现了当时最高质量的 ImageNet 条件生成。其核心技术包括：

1. **大批次训练**：batch size 达到 2048
2. **类别平衡采样**：缓解类别不平衡问题
3. **截断技巧（Truncation Trick）**：控制生成样本的多样性-质量权衡
4. **谱归一化**：稳定训练过程

### 5.2 BigGAN 架构细节

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

## 6. SAGAN：自注意力生成对抗网络

### 6.1 自注意力机制

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

## 7. GAN 评估指标

### 7.1 Inception Score (IS)

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

### 7.2 Fréchet Inception Distance (FID)

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

## 8. 学术引用与参考文献

1. Mirza, M., & Osindero, S. (2014). "Conditional Generative Adversarial Nets." *arXiv:1411.1784*.
2. Odena, A., Olah, C., & Shlens, J. (2017). "Conditional Image Synthesis with Auxiliary Classifier GANs." *ICML*.
3. Isola, P., et al. (2017). "Image-to-Image Translation with Conditional Adversarial Networks." *CVPR*.
4. Zhu, J. Y., et al. (2017). "Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks." *ICCV*.
5. Brock, A., Donahue, J., & Simonyan, K. (2019). "Large Scale GAN Training for High Fidelity Natural Image Synthesis." *ICLR*.
6. Zhang, H., et al. (2019). "Self-Attention Generative Adversarial Networks." *ICML*.
7. Heusel, M., et al. (2017). "GANs Trained by a Two Time-Scale Update Rule Converge to a Local Nash Equilibrium." *NeurIPS*.
8. Salimans, T., et al. (2016). "Improved Techniques for Training GANs." *NeurIPS*.

---

## 9. 相关文档

- [[GAN深度指南]] - 原始 GAN、DCGAN、Wasserstein GAN 等基础架构
- [[对抗样本深度指南]] - 对抗攻击与防御的原理
- [[博弈论与AI]] - GAN 的博弈论理论基础
- [[对抗训练与鲁棒性]] - 对抗训练方法与鲁棒性提升
