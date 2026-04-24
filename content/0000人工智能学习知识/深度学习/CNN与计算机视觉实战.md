---
title: CNN与计算机视觉实战
date: 2026-04-24
tags:
  - 深度学习
  - CNN
  - 计算机视觉
  - PyTorch
  - 迁移学习
aliases:
  - cnn-computer-vision-guide
---

# CNN与计算机视觉实战

计算机视觉是深度学习最成功的应用领域之一。从图像分类到目标检测，从语义分割到人脸识别，卷积神经网络（CNN）几乎统治了整个视觉领域。这篇文章会带你从卷积操作的基本原理讲起，遍历经典CNN架构的演进，最后用PyTorch实现ResNet并训练在CIFAR-10上。

## 卷积层：理解卷积操作

卷积层是CNN的核心。不同于全连接层每个神经元与所有输入相连，卷积层只和局部区域连接。这种局部连接和权重共享的设计，让CNN能够高效处理图像这种高维数据。

### 卷积操作到底在做什么

卷积操作本质上是一个滑动窗口滤波器。假设你有一个5×5的输入图像和一个3×3的卷积核（也叫滤波器）。卷积核会在图像上滑动，每到一个位置就做一次"局部区域与卷积核的对应元素相乘再求和"的操作。

你可能会问：为什么要这样做？答案是特征提取。不同的卷积核能检测不同的视觉特征：边缘、纹理、角点、形状等。比如一个检测水平边缘的卷积核，在遇到水平边缘时会产生高响应，遇到垂直边缘则响应低。

卷积核的值是模型要学习的参数。通过反向传播，CNN自动学习出对当前任务最有用的特征检测器。这就是深度学习"端到端"学习魅力的体现——你不需要手动设计特征，模型自己学出来。

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import matplotlib.pyplot as plt
import numpy as np

# 手动实现卷积操作，理解其原理
def manual_conv2d(input_tensor, kernel, stride=1, padding=0):
    """
    手动实现2D卷积
    input_tensor: (batch, channels, height, width)
    kernel: (out_channels, in_channels, kH, kW)
    """
    if padding > 0:
        input_tensor = F.pad(input_tensor, (padding,)*4)
    
    batch, in_channels, H, W = input_tensor.shape
    out_channels, _, kH, kW = kernel.shape
    
    out_H = (H - kH) // stride + 1
    out_W = (W - kW) // stride + 1
    
    output = torch.zeros(batch, out_channels, out_H, out_W)
    
    for b in range(batch):
        for oc in range(out_channels):
            for i in range(out_H):
                for j in range(out_W):
                    h_start = i * stride
                    w_start = j * stride
                    # 提取局部区域
                    region = input_tensor[b, :, h_start:h_start+kH, w_start:w_start+kW]
                    # 逐通道卷积并求和
                    output[b, oc, i, j] = (region * kernel[oc]).sum()
    
    return output

# 测试卷积操作
input_tensor = torch.randn(1, 1, 5, 5)
kernel = torch.randn(1, 1, 3, 3)

print("输入:")
print(input_tensor.squeeze().numpy())
print("\n卷积核:")
print(kernel.squeeze().numpy())

# PyTorch的卷积验证
conv = nn.Conv2d(1, 1, kernel_size=3, bias=False)
conv.weight.data = kernel
output = conv(input_tensor)
print("\n输出 (PyTorch Conv2d):")
print(output.squeeze().detach().numpy())
```

### 步长、填充与感受野

三个关键参数控制卷积的行为：**步长（stride）**、**填充（padding）**和**感受野（receptive field）**。

步长控制卷积核滑动的步进大小。stride=1时每步移动1个像素，stride=2时跳过一个像素。步长大于1可以降低特征图尺寸，实现下采样。

填充是在输入边界周围添加额外的像素。常用的"same"填充保持输出尺寸与输入相同。无填充（valid）会使输出变小。padding=1常用来补偿3×3卷积导致的尺寸缩小（每层减2）。

感受野是指输出特征图上一个像素对应输入图像的区域大小。浅层的卷积核感受野小，只能看到局部细节；深层的卷积核感受野大，能看到更大的区域。堆叠多层小卷积核可以达到和大卷积核相同的感受野，但参数量更少、非线性更多。

```python
# 对比不同步长和填充的效果
print("=" * 60)
print("步长和填充对特征图尺寸的影响")
print("=" * 60)

# 输入7x7图像，3x3卷积
input_size = 7
kernel_size = 3

for stride in [1, 2]:
    for padding in [0, 1]:
        output_size = (input_size + 2*padding - kernel_size) // stride + 1
        print(f"输入={input_size}×{input_size}, stride={stride}, padding={padding} → 输出={output_size}×{output_size}")

# 感受野计算
print("\n" + "=" * 60)
print("感受野计算")
print("=" * 60)

def calc_receptive_field(layers, kernel_size=3):
    """计算多层卷积后的感受野"""
    rf = 1
    for i, layer in enumerate(layers):
        rf = rf + (kernel_size - 1) * layer
        print(f"Layer {i+1}: RF = {rf}")
    return rf

# 3层3x3卷积的感受野
print("\n3层3x3卷积的感受野:")
calc_receptive_field([1, 1, 1])

# 等效的单层7x7卷积
print("\n等效单层7x7卷积:")
print("Layer 1: RF = 7")
```

### 多通道卷积与分组卷积

真实世界的图像通常是彩色的，有RGB三个通道。每个卷积核也要有3个通道，分别和RGB做卷积，然后相加得到一个输出通道。

一个卷积层可以有多个卷积核，每个卷积核产生一个输出通道。所以卷积层的参数形状是 (out_channels, in_channels, kH, kW)。

分组卷积（Grouped Convolution）是一种节省计算量的技巧。它把输入通道分成G组，每组独立做卷积，最后再合并。这最早在AlexNet中用来解决显存问题，后来ShuffleNet将其发扬光大。

深度可分离卷积（Depthwise Separable Convolution）是分组卷积的极端情况：每组只有一个通道。它先做逐通道卷积（Depthwise），再做一个1×1卷积融合通道信息（Pointwise）。MobileNet系列就是靠这个实现了极轻量的模型。

```python
# 多通道卷积演示
batch, in_channels, height, width = 2, 3, 32, 32
out_channels = 16
kernel_size = 3

# 标准卷积
conv_standard = nn.Conv2d(in_channels, out_channels, kernel_size)
print(f"标准卷积参数: {sum(p.numel() for p in conv_standard.parameters())}")

# 分组卷积 (groups=2)
conv_grouped = nn.Conv2d(in_channels, out_channels, kernel_size, groups=2)
print(f"分组卷积 (g=2) 参数: {sum(p.numel() for p in conv_grouped.parameters())}")

# 深度可分离卷积
conv_dw = nn.Conv2d(in_channels, in_channels, kernel_size, groups=in_channels)
conv_pw = nn.Conv2d(in_channels, out_channels, 1)
depthwise_separable = nn.Sequential(conv_dw, conv_pw)
total_params = sum(p.numel() for p in conv_dw.parameters()) + sum(p.numel() for p in conv_pw.parameters())
print(f"深度可分离卷积参数: {total_params}")

print(f"\n参数节省比例: {(1 - total_params/sum(p.numel() for p in conv_standard.parameters()))*100:.1f}%")
```

## 经典CNN架构演进

理解CNN架构的演进历史，不仅能帮助你更好地设计模型，还能从前辈的创新中汲取灵感。

### LeNet到AlexNet：深度学习的觉醒

1998年LeCun提出的LeNet是CNN的开山之作，但受限于当时的计算能力和数据量，它一直默默无闻。直到2012年AlexNet在ImageNet竞赛中以压倒性优势夺冠，深度学习才开始席卷计算机视觉领域。

AlexNet相比LeNet的主要改进：
- 网络更深（8层 vs 5层）
- 使用ReLU激活函数（比tanh快很多）
- Dropout正则化（防止过拟合）
- GPU并行训练
- 数据增强（随机裁剪、水平翻转）

```python
# LeNet-5 实现
class LeNet(nn.Module):
    def __init__(self, num_classes=10):
        super().__init__()
        self.conv1 = nn.Conv2d(3, 6, kernel_size=5)      # 32x32 → 28x28
        self.pool1 = nn.AvgPool2d(kernel_size=2, stride=2)  # 28x28 → 14x14
        self.conv2 = nn.Conv2d(6, 16, kernel_size=5)      # 14x14 → 10x10
        self.pool2 = nn.AvgPool2d(kernel_size=2, stride=2)  # 10x10 → 5x5
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, num_classes)
    
    def forward(self, x):
        x = self.pool1(torch.relu(self.conv1(x)))
        x = self.pool2(torch.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = torch.relu(self.fc1(x))
        x = torch.relu(self.fc2(x))
        x = self.fc3(x)
        return x

# AlexNet 实现
class AlexNet(nn.Module):
    def __init__(self, num_classes=1000):
        super().__init__()
        self.features = nn.Sequential(
            # 224x224x3 → 55x55x96
            nn.Conv2d(3, 96, kernel_size=11, stride=4, padding=2),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),  # 55x55 → 27x27
            
            # 27x27x256 → 13x13x256
            nn.Conv2d(96, 256, kernel_size=5, padding=2),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),  # 27x27 → 13x13
            
            # 13x13x384 → 13x13x384
            nn.Conv2d(256, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            
            # 13x13x384 → 13x13x384
            nn.Conv2d(384, 384, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            
            # 13x13x384 → 13x13x256
            nn.Conv2d(384, 256, kernel_size=3, padding=1),
            nn.ReLU(inplace=True),
            nn.MaxPool2d(kernel_size=3, stride=2),  # 13x13 → 6x6
        )
        
        self.avgpool = nn.AdaptiveAvgPool2d((6, 6))
        self.classifier = nn.Sequential(
            nn.Dropout(0.5),
            nn.Linear(256 * 6 * 6, 4096),
            nn.ReLU(inplace=True),
            nn.Dropout(0.5),
            nn.Linear(4096, 4096),
            nn.ReLU(inplace=True),
            nn.Linear(4096, num_classes),
        )
    
    def forward(self, x):
        x = self.features(x)
        x = self.avgpool(x)
        x = x.view(x.size(0), -1)
        x = self.classifier(x)
        return x

print("LeNet-5 参数量:", sum(p.numel() for p in LeNet().parameters()))
print("AlexNet 参数量:", sum(p.numel() for p in AlexNet().parameters()))
```

### VGG：更深的网络，更小的卷积核

2014年VGGNet提出了一个简单但深刻的观点：用多层小卷积核代替大卷积核。两个3×3卷积堆叠的有效感受野是5×5，但参数量只有25 vs 18（假设单通道）。三个3×3堆叠相当于7×7感受野。

VGG-16和VGG-19成为后续很多工作的backbone。但VGG的缺点也很明显：参数量大（140M+）、训练慢、 inference慢。

### 残差连接：越过了深度的墙

ResNet（Residual Network）是2015年何恺明团队的杰作，解决了"网络越深效果越差"的问题。他们提出了残差连接（skip connection）：不是学习x到H(x)的映射，而是学习残差F(x) = H(x) - x。

为什么残差学习更容易？因为如果 identity mapping 已经是最优解，把F(x)push到0比学习一个新的映射容易得多。残差连接让信息可以直接流动，缓解了梯度消失问题，使得训练1000+层的网络成为可能。

```python
# ResNet的核心：残差块
class ResidualBlock(nn.Module):
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super().__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, 
                              stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.relu = nn.ReLU(inplace=True)
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3,
                              stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.downsample = downsample
        
    def forward(self, x):
        identity = x
        
        out = self.conv1(x)
        out = self.bn1(out)
        out = self.relu(out)
        
        out = self.conv2(out)
        out = self.bn2(out)
        
        # 残差连接
        if self.downsample is not None:
            identity = self.downsample(x)
        out += identity
        out = self.relu(out)
        
        return out

# 完整的ResNet-18实现
class ResNet18(nn.Module):
    def __init__(self, num_classes=1000):
        super().__init__()
        self.in_channels = 64
        
        self.conv1 = nn.Conv2d(3, 64, kernel_size=7, stride=2, padding=3, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        self.relu = nn.ReLU(inplace=True)
        self.maxpool = nn.MaxPool2d(kernel_size=3, stride=2, padding=1)
        
        self.layer1 = self._make_layer(64, 2, stride=1)
        self.layer2 = self._make_layer(128, 2, stride=2)
        self.layer3 = self._make_layer(256, 2, stride=2)
        self.layer4 = self._make_layer(512, 2, stride=2)
        
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512, num_classes)
        
        # 权重初始化
        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
            elif isinstance(m, nn.BatchNorm2d):
                nn.init.constant_(m.weight, 1)
                nn.init.constant_(m.bias, 0)
    
    def _make_layer(self, out_channels, num_blocks, stride):
        downsample = None
        if stride != 1 or self.in_channels != out_channels:
            downsample = nn.Sequential(
                nn.Conv2d(self.in_channels, out_channels, kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels)
            )
        
        layers = []
        layers.append(ResidualBlock(self.in_channels, out_channels, stride, downsample))
        self.in_channels = out_channels
        
        for _ in range(1, num_blocks):
            layers.append(ResidualBlock(out_channels, out_channels))
        
        return nn.Sequential(*layers)
    
    def forward(self, x):
        x = self.conv1(x)
        x = self.bn1(x)
        x = self.relu(x)
        x = self.maxpool(x)
        
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
        
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.fc(x)
        
        return x

print("ResNet-18 参数量:", sum(p.numel() for p in ResNet18().parameters()))
```

## 批量归一化BatchNorm

BatchNorm是深度学习中最重要的技术之一。它在每个batch上对神经元输出做归一化，使得每层的输入保持稳定的分布，大大加速了训练。

### BatchNorm的工作原理

BatchNorm在训练时的行为：
1. 计算当前batch的均值μ_B和方差σ²_B
2. 归一化：(x - μ_B) / √(σ²_B + ε)
3. 线性变换：γ * x_norm + β

γ和β是可学习的参数，让模型自己决定是否需要归一化。

BatchNorm在推理时使用全局统计量（移动平均的均值和方差），而不是当前batch的。这保证了inference的确定性。

### BatchNorm在网络中的位置

关于BatchNorm的位置（BN before or after activation），有两种主流观点：

**原论文方案（BN before ReLU）**：归一化 → 线性变换（含γβ）→ ReLU。这是原始ResNet的做法。

**实践方案（BN after ReLU）**：卷积 → ReLU → BN。这种做法在很多现代架构中使用。

两种方案各有道理，实际效果差别不大，按照主流实现来用就好。

```python
# BatchNorm演示
bn = nn.BatchNorm2d(num_features=64)

# 模拟训练模式
bn.train()
print("训练模式下的BatchNorm:")
for i in range(3):
    x = torch.randn(32, 64, 32, 32)  # batch=32, channels=64
    y = bn(x)
    print(f"  Batch {i+1}: mean={y.mean().item():.4f}, std={y.std().item():.4f}")

# 模拟推理模式
bn.eval()
print("\n推理模式下的BatchNorm:")
with torch.no_grad():
    for i in range(3):
        x = torch.randn(32, 64, 32, 32)
        y = bn(x)
        print(f"  Batch {i+1}: mean={y.mean().item():.4f}, std={y.std().item():.4f}")

print(f"\n全局统计量 - running_mean前4个: {bn.running_mean[:4]}")
print(f"全局统计量 - running_var前4个: {bn.running_var[:4]}")
```

## 数据增强：扩充你的数据集

数据增强是防止过拟合、提升模型泛化能力的关键技术。它通过对训练数据做随机变换，人为增加数据的多样性。

### 经典数据增强

翻转（水平翻转为主，垂直翻转要谨慎）、旋转（一般±15度）、裁剪（随机裁剪到原图的0.8-1.0）、缩放、色彩抖动（亮度、对比度、饱和度、色调）、添加噪声。

```python
from torchvision import transforms
from PIL import Image
import torchvision.datasets as datasets

# 定义训练和测试的数据增强
train_transform = transforms.Compose([
    transforms.RandomResizedCrop(224, scale=(0.08, 1.0)),  # 随机裁剪并缩放到224
    transforms.RandomHorizontalFlip(p=0.5),                  # 50%概率水平翻转
    transforms.RandomRotation(15),                           # ±15度随机旋转
    transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1),  # 色彩抖动
    transforms.RandomAffine(degrees=0, translate=(0.1, 0.1)),  # 随机仿射变换
    transforms.ToTensor(),                                    # 转换为张量
    transforms.Normalize(mean=[0.485, 0.456, 0.406],          # ImageNet标准化
                        std=[0.229, 0.224, 0.225]),
    transforms.RandomErasing(p=0.1, scale=(0.02, 0.2)),     # 随机擦除（SimCLR使用）
])

test_transform = transforms.Compose([
    transforms.Resize(256),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.485, 0.456, 0.406],
                        std=[0.229, 0.224, 0.225])
])

# 演示数据增强效果
def visualize_augmentations(image_path=None):
    """可视化数据增强效果"""
    # 加载一张示例图片
    if image_path:
        img = Image.open(image_path).convert('RGB')
    else:
        # 创建一张测试图片
        img = Image.new('RGB', (224, 224), color=(100, 150, 200))
        from PIL import ImageDraw, ImageFont
        draw = ImageDraw.Draw(img)
        draw.text((80, 90), "Test Image", fill='white')
    
    fig, axes = plt.subplots(2, 5, figsize=(15, 6))
    axes = axes.flatten()
    
    for i in range(10):
        if i == 0:
            axes[i].imshow(img)
            axes[i].set_title("Original")
        else:
            # 应用随机变换
            img_aug = train_transform(img)
            # 反标准化用于显示
            mean = torch.tensor([0.485, 0.456, 0.406]).view(3, 1, 1)
            std = torch.tensor([0.229, 0.224, 0.225]).view(3, 1, 1)
            img_display = img_aug * std + mean
            img_display = torch.clamp(img_display, 0, 1)
            axes[i].imshow(img_display.permute(1, 2, 0).numpy())
            axes[i].set_title(f"Augmented {i}")
        axes[i].axis('off')
    
    plt.tight_layout()
    plt.savefig('data_augmentation_examples.png', dpi=150)

# visualize_augmentations()
```

### MixUp与CutMix

MixUp和CutMix是更高级的数据增强方法，在分类和检测任务上效果显著。

MixUp把两张图片按比例混合，同时混合它们的标签。比如两张图片按0.7:0.3混合，标签也按同样比例混合。这强迫模型学习更平滑的决策边界。

CutMix把图片的一块区域裁剪下来贴到另一张图片上，标签按面积比例混合。好处是模型能看到完整的物体，而不是被混合后的模糊图片。

```python
def mixup_data(x, y, alpha=0.2):
    """MixUp数据增强"""
    if alpha > 0:
        lam = np.random.beta(alpha, alpha)
    else:
        lam = 1
    
    batch_size = x.size(0)
    index = torch.randperm(batch_size).to(x.device)
    
    mixed_x = lam * x + (1 - lam) * x[index, :]
    y_a, y_b = y, y[index]
    
    return mixed_x, y_a, y_b, lam

def cutmix_data(x, y, alpha=1.0):
    """CutMix数据增强"""
    lam = np.random.beta(alpha, alpha)
    batch_size = x.size(0)
    index = torch.randperm(batch_size).to(x.device)
    
    W = x.size(2)
    H = x.size(3)
    cut_rat = np.sqrt(1. - lam)
    cut_w = int(W * cut_rat)
    cut_h = int(H * cut_rat)
    
    cx = np.random.randint(W)
    cy = np.random.randint(H)
    
    bbx1 = np.clip(cx - cut_w // 2, 0, W)
    bby1 = np.clip(cy - cut_h // 2, 0, H)
    bbx2 = np.clip(cx + cut_w // 2, 0, W)
    bby2 = np.clip(cy + cut_h // 2, 0, H)
    
    x[:, :, bbx1:bbx2, bby1:bby2] = x[index, :, bbx1:bbx2, bby1:bby2]
    
    lam = 1 - ((bbx2 - bbx1) * (bby2 - bby1) / (W * H))
    y_a, y_b = y, y[index]
    
    return x, y_a, y_b, lam

def mixup_criterion(criterion, pred, y_a, y_b, lam):
    """MixUp/CutMix的损失函数"""
    return lam * criterion(pred, y_a) + (1 - lam) * criterion(pred, y_b)

# 可视化MixUp和CutMix
def visualize_mixup_cutmix():
    """可视化MixUp和CutMix效果"""
    # 创建两张测试图片
    img1 = Image.new('RGB', (224, 224), color=(255, 100, 100))  # 红色
    img2 = Image.new('RGB', (224, 224), color=(100, 100, 255))  # 蓝色
    
    # 转换为张量
    to_tensor = transforms.ToTensor()
    img1_tensor = to_tensor(img1).unsqueeze(0)
    img2_tensor = to_tensor(img2).unsqueeze(0)
    
    fig, axes = plt.subplots(2, 3, figsize=(12, 8))
    
    # 原始图片
    axes[0, 0].imshow(img1)
    axes[0, 0].set_title('Image 1')
    axes[0, 0].axis('off')
    
    axes[0, 1].imshow(img2)
    axes[0, 1].set_title('Image 2')
    axes[0, 1].axis('off')
    
    # MixUp效果 (lambda=0.5)
    lam = 0.5
    mixed = lam * img1_tensor + (1 - lam) * img2_tensor
    mixed_img = transforms.ToPILImage()(mixed.squeeze().clamp(0, 1))
    axes[0, 2].imshow(mixed_img)
    axes[0, 2].set_title(f'MixUp (λ={lam})')
    axes[0, 2].axis('off')
    
    # CutMix效果
    W, H = 224, 224
    cut_w, cut_h = int(W * np.sqrt(0.6)), int(H * np.sqrt(0.6))
    cx, cy = 112, 112
    bbx1, bby1 = cx - cut_w // 2, cy - cut_h // 2
    bbx2, bby2 = cx + cut_w // 2, cy + cut_h // 2
    
    cutmix = img1_tensor.clone()
    cutmix[:, :, bbx1:bbx2, bby1:bby2] = img2_tensor[:, :, bbx1:bbx2, bby1:bby2]
    cutmix_img = transforms.ToPILImage()(cutmix.squeeze().clamp(0, 1))
    axes[1, 0].imshow(cutmix_img)
    axes[1, 0].set_title(f'CutMix (λ={1-(bbx2-bbx1)*(bby2-bby1)/(W*H):.2f})')
    axes[1, 0].axis('off')
    
    # 更多的MixUp例子
    for i, lam in enumerate([0.3, 0.7]):
        mixed = lam * img1_tensor + (1 - lam) * img2_tensor
        mixed_img = transforms.ToPILImage()(mixed.squeeze().clamp(0, 1))
        axes[1, i+1].imshow(mixed_img)
        axes[1, i+1].set_title(f'MixUp (λ={lam})')
        axes[1, i+1].axis('off')
    
    plt.tight_layout()
    plt.savefig('mixup_cutmix_examples.png', dpi=150)

visualize_mixup_cutmix()
```

## 实战：ResNet在CIFAR-10上训练

现在来一个完整的实战：用PyTorch实现ResNet并在CIFAR-10上训练。

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.nn.functional as F
import torchvision
import torchvision.transforms as transforms
from torch.utils.data import DataLoader
import torchvision.models as models
import time
import copy

# 设置随机种子保证可复现性
def set_seed(seed=42):
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    np.random.seed(seed)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False

set_seed(42)

# 检查GPU
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"使用设备: {device}")

# CIFAR-10数据集准备
# CIFAR-10: 32x32 RGB图像，10个类别
transform_train = transforms.Compose([
    transforms.RandomCrop(32, padding=4),
    transforms.RandomHorizontalFlip(),
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010))
])

transform_test = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.4914, 0.4822, 0.4465), (0.2023, 0.1994, 0.2010))
])

trainset = torchvision.datasets.CIFAR10(root='./data', train=True, download=True, transform=transform_train)
testset = torchvision.datasets.CIFAR10(root='./data', train=False, download=True, transform=transform_test)

trainloader = DataLoader(trainset, batch_size=128, shuffle=True, num_workers=4, pin_memory=True)
testloader = DataLoader(testset, batch_size=128, shuffle=False, num_workers=4, pin_memory=True)

classes = ('plane', 'car', 'bird', 'cat', 'deer', 'dog', 'frog', 'horse', 'ship', 'truck')

print(f"训练集大小: {len(trainset)}")
print(f"测试集大小: {len(testset)}")

# 针对CIFAR-10优化的ResNet (ResNet-18变体，输入32x32)
class BasicBlock(nn.Module):
    expansion = 1
    
    def __init__(self, in_channels, out_channels, stride=1, downsample=None):
        super().__init__()
        self.conv1 = nn.Conv2d(in_channels, out_channels, kernel_size=3, 
                               stride=stride, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(out_channels)
        self.conv2 = nn.Conv2d(out_channels, out_channels, kernel_size=3,
                               stride=1, padding=1, bias=False)
        self.bn2 = nn.BatchNorm2d(out_channels)
        self.downsample = downsample
        
    def forward(self, x):
        identity = x
        
        out = F.relu(self.bn1(self.conv1(x)))
        out = self.bn2(self.conv2(out))
        
        if self.downsample is not None:
            identity = self.downsample(x)
        
        out += identity
        out = F.relu(out)
        
        return out

class ResNetCIFAR(nn.Module):
    """适用于CIFAR-32x32输入的ResNet"""
    def __init__(self, block, num_blocks, num_classes=10):
        super().__init__()
        self.in_channels = 64
        
        # CIFAR-10使用的初始化：3个卷积层而不是ResNet原版的7x7
        self.conv1 = nn.Conv2d(3, 64, kernel_size=3, stride=1, padding=1, bias=False)
        self.bn1 = nn.BatchNorm2d(64)
        # 不用maxpool，直接跟上residual blocks
        
        self.layer1 = self._make_layer(block, 64, num_blocks[0], stride=1)
        self.layer2 = self._make_layer(block, 128, num_blocks[1], stride=2)
        self.layer3 = self._make_layer(block, 256, num_blocks[2], stride=2)
        self.layer4 = self._make_layer(block, 512, num_blocks[3], stride=2)
        
        self.avgpool = nn.AdaptiveAvgPool2d((1, 1))
        self.fc = nn.Linear(512 * block.expansion, num_classes)
        
        # 权重初始化
        for m in self.modules():
            if isinstance(m, nn.Conv2d):
                nn.init.kaiming_normal_(m.weight, mode='fan_out', nonlinearity='relu')
            elif isinstance(m, nn.BatchNorm2d):
                nn.init.constant_(m.weight, 1)
                nn.init.constant_(m.bias, 0)
    
    def _make_layer(self, block, out_channels, num_blocks, stride):
        downsample = None
        if stride != 1 or self.in_channels != out_channels * block.expansion:
            downsample = nn.Sequential(
                nn.Conv2d(self.in_channels, out_channels * block.expansion,
                         kernel_size=1, stride=stride, bias=False),
                nn.BatchNorm2d(out_channels * block.expansion)
            )
        
        layers = []
        layers.append(block(self.in_channels, out_channels, stride, downsample))
        self.in_channels = out_channels * block.expansion
        
        for _ in range(1, num_blocks):
            layers.append(block(self.in_channels, out_channels))
        
        return nn.Sequential(*layers)
    
    def forward(self, x):
        x = F.relu(self.bn1(self.conv1(x)))
        x = self.layer1(x)
        x = self.layer2(x)
        x = self.layer3(x)
        x = self.layer4(x)
        x = self.avgpool(x)
        x = torch.flatten(x, 1)
        x = self.fc(x)
        return x

def ResNet18CIFAR():
    return ResNetCIFAR(BasicBlock, [2, 2, 2, 2])

# 创建模型
model = ResNet18CIFAR().to(device)
print(f"\nResNet-18 for CIFAR-10 参数量: {sum(p.numel() for p in model.parameters()):,}")

# 训练函数
def train_epoch(model, dataloader, criterion, optimizer, device):
    model.train()
    running_loss = 0.0
    correct = 0
    total = 0
    
    for inputs, targets in dataloader:
        inputs, targets = inputs.to(device), targets.to(device)
        
        optimizer.zero_grad()
        outputs = model(inputs)
        loss = criterion(outputs, targets)
        loss.backward()
        optimizer.step()
        
        running_loss += loss.item() * inputs.size(0)
        _, predicted = outputs.max(1)
        total += targets.size(0)
        correct += predicted.eq(targets).sum().item()
    
    return running_loss / total, correct / total

def evaluate(model, dataloader, device):
    model.eval()
    correct = 0
    total = 0
    
    with torch.no_grad():
        for inputs, targets in dataloader:
            inputs, targets = inputs.to(device), targets.to(device)
            outputs = model(inputs)
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
    
    return correct / total

# 训练配置
criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(model.parameters(), lr=0.1, momentum=0.9, weight_decay=5e-4)
scheduler = optim.lr_scheduler.MultiStepLR(optimizer, milestones=[82, 123], gamma=0.1)

# 训练循环
num_epochs = 150
best_acc = 0.0
history = {'train_loss': [], 'train_acc': [], 'test_acc': []}

print("\n开始训练...")
start_time = time.time()

for epoch in range(num_epochs):
    train_loss, train_acc = train_epoch(model, trainloader, criterion, optimizer, device)
    test_acc = evaluate(model, testloader, device)
    
    history['train_loss'].append(train_loss)
    history['train_acc'].append(train_acc)
    history['test_acc'].append(test_acc)
    
    scheduler.step()
    
    if test_acc > best_acc:
        best_acc = test_acc
        best_model_state = copy.deepcopy(model.state_dict())
    
    if (epoch + 1) % 10 == 0 or epoch == 0:
        print(f"Epoch [{epoch+1:3d}/{num_epochs}] "
              f"Loss: {train_loss:.4f} "
              f"Train Acc: {train_acc:.4f} "
              f"Test Acc: {test_acc:.4f} "
              f"LR: {scheduler.get_last_lr()[0]:.6f}")

total_time = time.time() - start_time
print(f"\n训练完成! 耗时: {total_time/60:.1f}分钟")
print(f"最佳测试准确率: {best_acc:.4f}")

# 保存最佳模型
torch.save(best_model_state, 'resnet18_cifar10.pth')
print("模型已保存到 resnet18_cifar10.pth")

# 绘制训练曲线
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

axes[0].plot(history['train_loss'], label='Train Loss')
axes[0].set_xlabel('Epoch')
axes[0].set_ylabel('Loss')
axes[0].set_title('Training Loss')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

axes[1].plot(history['train_acc'], label='Train Acc')
axes[1].plot(history['test_acc'], label='Test Acc')
axes[1].set_xlabel('Epoch')
axes[1].set_ylabel('Accuracy')
axes[1].set_title('Training vs Test Accuracy')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('training_curves.png', dpi=150)
```

## 迁移学习：从预训练模型出发

从头训练大型CNN需要海量数据和计算资源。迁移学习让你站在巨人的肩膀上——使用在ImageNet等大数据集上预训练的模型，只需微调就能适应新任务。

### 两种迁移策略

**特征提取（Feature Extraction）**：冻结预训练模型的大部分层，只训练最后几层（通常是分类头）。适合小数据集、计算资源有限的场景。预训练模型充当强大的特征提取器。

**微调（Fine-tuning）**：解冻部分或全部层，用较小的学习率训练整个网络。适合有一定数据量的场景，让模型学习新任务的特定特征。

```python
# 迁移学习实战：使用预训练的ResNet-18
print("=" * 60)
print("迁移学习实战：ImageNet预训练模型")
print("=" * 60)

# 加载预训练的ResNet-18
model_pretrained = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)

# 特征提取模式：冻结所有层
for param in model_pretrained.parameters():
    param.requires_grad = False

# 替换最后的分类层
num_features = model_pretrained.fc.in_features
model_pretrained.fc = nn.Linear(num_features, 10)  # CIFAR-10有10类

# 只有fc层的参数是可训练的
trainable_params = sum(p.numel() for p in model_pretrained.parameters() if p.requires_grad)
total_params = sum(p.numel() for p in model_pretrained.parameters())
print(f"特征提取模式 - 可训练参数: {trainable_params:,} / {total_params:,} ({trainable_params/total_params*100:.2f}%)")

# 微调模式：解冻最后几层
for param in model_pretrained.layer4.parameters():
    param.requires_grad = True

# 使用更小的学习率
optimizer_ft = optim.SGD([
    {'params': model_pretrained.fc.parameters(), 'lr': 0.1},
    {'params': model_pretrained.layer4.parameters(), 'lr': 0.01},
], momentum=0.9, weight_decay=5e-4)

# 学习率调度器 - 恢复预训练层用更小的学习率
def train_model(model, trainloader, testloader, optimizer, scheduler, num_epochs=20):
    best_acc = 0
    
    for epoch in range(num_epochs):
        # 训练
        model.train()
        running_loss = 0.0
        correct = 0
        total = 0
        
        for inputs, targets in trainloader:
            inputs, targets = inputs.to(device), targets.to(device)
            
            optimizer.zero_grad()
            outputs = model(inputs)
            loss = F.cross_entropy(outputs, targets)
            loss.backward()
            optimizer.step()
            
            running_loss += loss.item() * inputs.size(0)
            _, predicted = outputs.max(1)
            total += targets.size(0)
            correct += predicted.eq(targets).sum().item()
        
        scheduler.step()
        
        # 评估
        model.eval()
        test_correct = 0
        test_total = 0
        with torch.no_grad():
            for inputs, targets in testloader:
                inputs, targets = inputs.to(device), targets.to(device)
                outputs = model(inputs)
                _, predicted = outputs.max(1)
                test_total += targets.size(0)
                test_correct += predicted.eq(targets).sum().item()
        
        train_acc = correct / total
        test_acc = test_correct / test_total
        
        if test_acc > best_acc:
            best_acc = test_acc
        
        if (epoch + 1) % 5 == 0:
            print(f"Epoch {epoch+1}: Train Acc={train_acc:.4f}, Test Acc={test_acc:.4f}")
    
    return best_acc

# 训练迁移学习模型
model_pretrained = model_pretrained.to(device)
optimizer_ft = optim.SGD(filter(lambda p: p.requires_grad, model_pretrained.parameters()),
                        lr=0.01, momentum=0.9, weight_decay=5e-4)
scheduler_ft = optim.lr_scheduler.StepLR(optimizer_ft, step_size=5, gamma=0.1)

print("\n训练特征提取模型...")
best_acc_ft = train_model(model_pretrained, trainloader, testloader, optimizer_ft, scheduler_ft, num_epochs=30)
print(f"特征提取最佳准确率: {best_acc_ft:.4f}")

# 对比从头训练 vs 迁移学习
print("\n" + "=" * 60)
print("对比：从头训练 vs 迁移学习")
print("=" * 60)
print(f"从头训练 (150 epochs): 最佳准确率 ~{best_acc:.4f}")
print(f"迁移学习 (30 epochs):  最佳准确率 {best_acc_ft:.4f}")
print(f"迁移学习只用了1/5的时间就达到了相近甚至更好的效果！")
```

## 总结

这篇文章从卷积操作的基本原理讲起，涵盖了CNN的核心知识点。

卷积操作是CNN的基础，理解步长、填充、感受野这些概念对设计和调试模型至关重要。经典架构的演进展示了深度学习研究的脉络：从LeNet到AlexNet唤醒了这个领域，VGG证明了深度的力量，ResNet用残差连接突破了深度的瓶颈。

BatchNorm、Dropout、数据增强这些技术是训练稳定模型的必备。BatchNorm通过归一化稳定了每层的输入分布，Dropout防止过拟合，数据增强扩充了训练数据的多样性。MixUp和CutMix是更高级的增强方法，能显著提升模型泛化能力。

迁移学习是现代计算机视觉的基石。站在ImageNet预训练模型的肩膀上，你只需要很少的数据和计算资源就能获得很好的效果。特征提取适合小数据场景，微调适合有一定数据量的场景。

最后提醒一句：深度学习是实践性很强的领域，光看不动手是不行的。找一些开源数据集，自己跑一跑代码，改一改参数，感受一下不同选择带来的效果差异。祝你玩得开心！

---

*本文为深度学习实战指南系列文章，主要涵盖CNN与计算机视觉的核心知识点和实践技巧。*
