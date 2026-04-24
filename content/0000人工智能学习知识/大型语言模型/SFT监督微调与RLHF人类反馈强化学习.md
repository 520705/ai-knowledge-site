---
title: SFT监督微调与RLHF人类反馈强化学习
date: 2026-04-24
tags:
  - SFT
  - RLHF
  - LoRA
  - DPO
  - LLM微调
aliases: [RLHF, 监督微调, 大模型微调]
---

# SFT监督微调与RLHF人类反馈强化学习

如果你已经训练好了一个通用的大语言模型，想让它变成某个领域的专家——比如专门写代码的模型、专门做客服对话的模型，或者能遵循特定指令的助手——那就需要用到微调技术。这篇文章就来聊聊微调的两大主流范式：SFT（监督微调）和RLHF（人类反馈强化学习），从原理到代码实战，带你彻底搞懂这些技术。

## 1. 为什么需要微调：通用模型vs专用模型

在聊微调之前，我们先搞清楚一个问题：既然GPT-4、Claude这些通用模型已经很强大了，为什么还要微调？

**通用模型的特点**是"什么都会一点，但什么都不精"。它能写文章、做翻译、答问题，但如果你想让模型在某个特定任务上表现更好，比如让你的AI助手总是用特定格式回复、或者让代码模型熟悉你们公司的代码规范，通用模型往往就差那么一口气。这不是因为模型本身不够好，而是因为它没有针对你的具体场景学习过。

**微调的核心理念**就是：站在巨人肩膀上，用相对少量的数据，让通用模型学会特定技能。相比从头训练一个专用模型，微调的成本要低得多——通常只需要几千到几万条高质量样本，就能让模型的行为发生显著改变。

微调主要解决两类问题：一是**任务适配**，让模型学会执行特定类型的任务，比如情感分析、实体识别；二是**行为对齐**，让模型的输出风格、价值观符合预期，比如更乐于助人、更诚实、更符合人类偏好。

这里有个重要的概念需要区分：**能力获取（capability acquisition）**和**行为对齐（alignment）**。能力获取是让模型"知道怎么做"，比如学会某种语言、某种技能；行为对齐是让模型"愿意按照人类期望的方式做"。SFT主要解决能力获取问题，而RLHF则主要用于行为对齐。

## 2. 全参数微调SFT：流程与技巧

SFT（Supervised Fine-Tuning，监督微调）是最直观的微调方式。它的思想很简单：既然预训练是让模型在大规模无标签数据上学习语言能力，那SFT就是在特定任务数据上继续训练，让模型学会执行具体任务。

### 2.1 SFT的基本流程

SFT的流程可以分为以下几个步骤：

**第一步，准备数据**。你需要准备一批有标签的训练数据，格式通常是"输入-输出"对。比如对于对话任务，就是"用户消息-助手回复"对；对于代码任务，就是"自然语言描述-代码实现"对。数据质量比数量更重要——几千条精心标注的数据，效果往往好过几十万条粗制滥造的数据。

**第二步，构造训练样本**。把数据转换成模型可以学习的格式。对于自回归语言模型，训练时通常把整个对话拼成一个序列，用特殊token标记不同部分（比如`<user>`、`<assistant>`），然后让模型预测下一个token。

**第三步，开始训练**。加载预训练好的模型，用你的数据继续训练。这一步和预训练类似，但有两个关键区别：一是数据规模小得多，通常几GB就够了；二是学习率要调低一些，因为模型本来就已经很强了，大学习率容易把已有的能力破坏掉。

**第四步，评估与迭代**。用保留的测试集评估模型效果，根据需要调整数据或超参数。

### 2.2 SFT的关键技巧

在实际操作中，有几个技巧能显著提升SFT的效果：

**学习率调度**非常重要。常用的策略是先用较小的学习率预热（warmup），让模型慢慢适应新任务，然后再逐步降低学习率。一个典型的设置是：warmup步数占总步数的5%-10%，然后余弦衰减到初始学习率的十分之一。

**梯度累积**可以在显存有限的情况下模拟大批量训练。比如你的显卡只能放下8个样本的batch，但想要等效32个样本的batch size，就可以用4个step累积梯度后再更新参数。

**早停（Early Stopping）**能防止过拟合。监控验证集上的loss，当连续若干个epoch loss不再下降时就停止训练，避免模型在训练集上"背答案"。

**数据格式一致性**也很关键。训练时的数据格式要和推理时保持一致。比如你在训练时用`<|user|>`标记用户消息，推理时也要用同样的格式，否则模型会困惑。

## 3. LoRA：低秩适应原理

全参数微调虽然效果好，但有个致命问题：太吃显存。一个7B参数的模型，光是模型权重就要14GB+，加上梯度、优化器状态，训练时可能需要几十GB的显存，大多数消费级显卡根本跑不动。

LoRA（Low-Rank Adaptation，低秩适应）就是为了解决这个问题而生的。

### 3.1 核心思想

LoRA的灵感来自一个关键发现：**神经网络中有很多冗余的参数，我们可以把参数更新用低秩矩阵来近似**。

具体来说，假设原模型的某个权重矩阵是 $W_0 \in \mathbb{R}^{d \times k}$。全参数微调要训练的是 $\Delta W$，最终权重变成 $W_0 + \Delta W$。

LoRA的做法是：不直接训练 $\Delta W$，而是训练两个小矩阵 $A$ 和 $B$，其中 $B \in \mathbb{R}^{d \times r}$，$A \in \mathbb{R}^{r \times k}$，$r$ 是远小于 $d$ 和 $k$ 的秩。这样：

$$W_0 + \Delta W = W_0 + BA$$

训练时，**$W_0$ 保持冻结不更新，只训练 $A$ 和 $B$**。推理时，把 $BA$ 加到 $W_0$ 上，就得到了微调后的权重。

为什么这样有效？因为研究人员发现，**微调过程中的参数变化往往具有很低的内在秩**——也就是说，$\Delta W$ 可以用一个低秩矩阵很好地近似。这意味着我们不需要学习那么多参数，只要用低秩分解就能捕捉到足够的信息。

### 3.2 关键参数详解

使用LoRA时，有几个核心参数需要调：

**rank（r）**：这是最关键的参数，决定了低秩矩阵的秩。r越大，模型的表达能力越强，可训练的参数越多，效果通常更好，但显存占用也更高。常用的取值是4、8、16、32、64。对于小数据集或简单任务，r=4或r=8就够了；对于复杂任务或大数据集，可能需要r=32或更高。

**alpha**：这是一个缩放因子，最终的权重更新是 $\frac{\alpha}{r} BA$。通常把alpha设为rank的倍数（比如r=8时alpha=16），这样可以在不增加参数量的情况下调整更新的幅度。

**dropout**：LoRA权重引入的dropout正则化，可以防止过拟合。如果数据量小或任务简单，dropout可以设高一些（如0.1）；如果数据充足，可以设为0。

**target_modules**：指定哪些模块应用LoRA。常见的选择是`q_proj`和`v_proj`（注意力机制的Query和Value投影），这两个通常效果最好。如果显存允许，也可以加上`k_proj`、`o_proj`，甚至所有线性层。哪些模块值得加LoRA，取决于具体任务——对于代码任务，可能需要关注所有与知识相关的模块；对于对话任务，主要关注语言生成相关的模块。

**lora_alpha**和**lora_dropout**：上面已经说过了。

**bias**：可以选择对bias也应用LoRA。设为`"none"`表示不对bias做处理，这是最省显存的选择；`"all"`表示所有bias都用LoRA；`"lora_only"`表示只有LoRA引入的bias需要训练。

### 3.3 LoRA的变体

LoRA生态很丰富，有很多改进版本：

**QLoRA**（Quantized LoRA）：结合了量化和LoRA，可以在极低显存下微调大模型，后面会详细介绍。

**AdaLoRA**：自适应调整不同层的rank，重要的层用高rank，不重要的层用低rank。

**LoRA+**：为A和B矩阵使用不同的学习率，据说能提升训练稳定性和效果。

**LoRA fine-tuning in qlora**：就是QLoRA，结合了量化和LoRA。

## 4. QLoRA：4-bit NormalFloat量化+LoRA

QLoRA（Quantized Low-Rank Adaptation）是LoRA的增强版，由Tim Dettmers等人在2023年提出。它的核心创新是：把量化技术和LoRA结合起来，使得在消费级显卡上微调几十B参数的大模型成为可能。

### 4.1 量化基础

在说QLoRA之前，先简单介绍一下量化（Quantization）。

量化本质上是把高精度数值用低精度表示。比如原本用32位浮点数（FP32）存储的权重，可以压缩成8位整数（INT8）甚至4位整数（INT4）。这能大幅减少显存占用——INT8只需要FP32四分之一的显存，INT4只需要八分之一。

但量化会带来精度损失。简单的量化（比如四舍五入）会导致模型性能明显下降。好的量化方法需要尽量保持模型的表达能力。

### 4.2 4-bit NormalFloat量化

QLoRA用了一种特殊的4位量化方法：**4-bit NormalFloat（NF4）**。

为什么叫"NormalFloat"？因为这种量化方法针对的是服从正态分布的权重值设计的。它的核心思想是：把数值空间划分成2^4=16个区间，这些区间的大小不是均匀的，而是根据正态分布的概率密度来设计的——靠近0的区域区间更小（分辨率更高），远离0的区域区间更大。

这样做的效果是：对于大多数神经网络权重（集中在0附近），量化误差更小；对于少数极端值，误差大一点也能接受。

### 4.3 分页优化器

QLoRA还引入了一个实用的技术：**分页优化器（Paged Optimizers）**。

训练大模型时，优化器状态（如Adam的动量）会占用大量显存。分页优化器的思想是：当显存不足时，把优化器状态临时卸载到CPU内存（就像操作系统的内存分页一样），等需要时再加载回来。这可以有效防止显存溢出，让训练更加稳定。

### 4.4 QLoRA的训练配置

一个典型的QLoRA配置大概是这样的：

```python
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training

# 4-bit量化配置
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16
)

# 加载量化模型
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto"
)

# 准备模型进行k-bit训练
model = prepare_model_for_kbit_training(model)

# LoRA配置
lora_config = LoraConfig(
    r=64,
    lora_alpha=16,
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"],
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# 应用LoRA
model = get_peft_model(model, lora_config)
```

这段代码展示了QLoRA的典型用法：首先用`BitsAndBytesConfig`配置4位量化加载模型，然后用`prepare_model_for_kbit_training`做一些必要的模型准备，最后用`get_peft_model`应用LoRA适配器。

## 5. RLHF三步骤：SFT→奖励模型→PPO/DPO

RLHF（Reinforcement Learning from Human Feedback，人类反馈强化学习）是一个三阶段的训练框架，用于让模型的输出更符合人类偏好。OpenAI在InstructGPT论文中首次系统性地提出并应用了这套方法。

### 5.1 RLHF的三阶段概述

**第一阶段：SFT（监督微调）**

这是RLHF的起点。我们需要先有一个能遵循指令的基础模型。具体做法是用高质量的"指令-回答"对数据进行SFT，让模型学会"当人类给出这样的指令时，应该这样回答"。

这一阶段的数据质量至关重要。OpenAI的做法是让标注员写出他们认为的理想回答，然后用来微调GPT-3。这些回答通常比较长、结构清晰、解释充分。

**第二阶段：训练奖励模型（Reward Model）**

SFT模型虽然能生成不错的回答，但它不知道自己生成的回答质量如何。奖励模型的作用就是给模型生成的回答打分——告诉模型什么样的回答是好的，什么样的是不好的。

训练奖励模型的方法是：让SFT模型对同一个问题生成多个回答，然后让人类标注员比较这些回答，给出偏好排序。接下来训练一个奖励模型，让它学会预测"人类会给这个回答打多少分"。

这个阶段的难点在于**数据标注成本很高**——需要大量人工比较，而且标注者的主观偏好可能有差异。为了提高效率，有时会用LLM来辅助标注（比如用GPT-4来判断回答质量），但这会引入新的偏差。

**第三阶段：用强化学习优化策略**

有了奖励模型，就可以用强化学习来优化SFT模型了。目标是让模型学会生成能获得高奖励分数的回答。

这一阶段通常用PPO（Proximal Policy Optimization）算法。但PPO在LLM场景下有一些特殊的设计，后面会详细介绍。

除了PPO，近年来也出现了DPO（Direct Preference Optimization）等简化方法，不需要单独训练奖励模型，直接用偏好数据优化策略。

### 5.2 为什么需要RLHF而不是直接SFT？

你可能会问：既然SFT能让模型学会回答问题，为什么还需要RLHF？

原因是：**SFT只能模仿训练数据，而RLHF可以学习隐含的偏好**。

想象一下这个场景：你有10个标注员对同一问题写了10个回答，每个回答风格都不一样。如果用SFT，模型只能学会"平均地模仿"这10个回答。但如果用RLHF，模型可以学习"人类觉得哪个回答更好"，从而学会识别和生成更符合偏好的内容。

RLHF还有一个优势：**可以处理开放式生成任务**。对于代码生成、创意写作这类任务，很难定义什么是"正确答案"，但可以用偏好学习来引导模型生成更优质的输出。

## 6. PPO在LLM中的应用：KL散度约束、clip机制

PPO（Proximal Policy Optimization，近端策略优化）是强化学习中非常流行的算法，被广泛应用于RLHF中。OpenAI在InstructGPT中使用的就是PPO。

### 6.1 PPO的基本原理

PPO是一种策略梯度算法，它的核心思想是：**不要一次更新太多**。每次更新时，新策略和旧策略的差异要控制在一定范围内，防止一次更新幅度过大导致性能崩溃。

PPO用了一个巧妙的目标函数：

$$L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min \left( r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \hat{A}_t \right) \right]$$

其中 $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$ 是新旧策略的概率比，$\hat{A}_t$ 是优势函数的估计，$\epsilon$ 是一个小常数（通常0.1-0.2），$\text{clip}$ 函数把概率比限制在 $[1-\epsilon, 1+\epsilon]$ 范围内。

这个公式的直观理解是：如果优势为正（这个动作是好的），就尽量增大选择这个动作的概率，但不要增大的太多（被clip限制住）；如果优势为负（这个动作是不好的），就尽量减小选择这个动作的概率，但也不要减小的太多。

### 6.2 KL散度约束

在LLM场景下应用PPO，有一个关键问题需要解决：**KL散度约束**。

没有约束的情况下，强化学习可能会"走偏"——模型可能会找到一些"取巧"的方式获得高奖励，但这些方式实际上是错误的。比如一个对话模型可能会学到"只要在回答中多夸奖用户，就能获得高奖励"，但这不是我们真正想要的。

为了防止这种情况，PPO在LLM中会加入KL散度约束：

$$\text{KL penalty} = \beta \cdot \mathbb{E}_t \left[ D_{KL}(\pi_{RL} \| \pi_{SFT}) \right]$$

这个KL项衡量的是强化学习策略和SFT策略的差异。通过惩罚过大的KL散度，可以让模型不会偏离SFT学到的基本行为太远。

### 6.3 Reward Model的价值函数

在LLM的PPO中，还有一个特殊的组件：**奖励模型的价值函数**（Value Function）。

价值函数 $V(s)$ 估计的是"从当前状态开始，能获得的总未来奖励"。它和奖励模型的分工不同：奖励模型给单个回答打分，价值函数帮助估计多步生成过程中的累积回报。

训练时，价值函数用均方误差来拟合奖励模型的输出：

$$L_V = \mathbb{E}_t \left[ (V(s_t) - \hat{R}_t)^2 \right]$$

其中 $\hat{R}_t$ 是从时间步t开始的后续奖励之和。

### 6.4 PPO的训练流程

在实际训练中，LLM的PPO通常这样进行：

1. **采样**：用当前的策略模型生成一批回答
2. **计算奖励**：用奖励模型给每个回答打分
3. **计算优势**：用价值函数和奖励计算优势估计
4. **更新策略**：用PPO算法更新策略模型
5. **更新价值函数**：用新的采样数据更新价值函数

这个过程会迭代进行，直到策略模型的性能达到预期。

### 6.5 PPO的挑战

PPO在LLM中的应用面临一些挑战：

**训练不稳定**：LLM的输出空间非常大，策略模型很容易在某些方面过拟合，导致训练不稳定。

**超参数敏感**：KL散度系数$\beta$、clip范围$\epsilon$、学习率等超参数对训练效果影响很大，需要仔细调优。

**计算开销大**：PPO需要同时维护策略模型、价值函数、参考模型（SFT模型）和奖励模型，显存占用很高。

正是这些挑战推动了DPO等更简单方法的发展。

## 7. DPO（直接偏好优化）：无需奖励模型的替代方案

DPO（Direct Preference Optimization，直接偏好优化）是由Berkeley的Rafael Rafailov等人于2023年提出的新方法。它最大的亮点是：**不需要单独训练奖励模型，也不需要复杂的强化学习过程，直接用偏好数据优化策略**。

### 7.1 DPO的理论基础

DPO的核心洞察是：可以通过数学推导，把RLHF中的奖励优化问题转化成一个等价的分类问题。

在标准RLHF中，优化目标是最大化期望奖励，同时保持与参考策略的KL散度约束：

$$\max_\pi \mathbb{E}_{x\sim D, y\sim \pi(\cdot|x)} [r(x,y)] - \beta \cdot D_{KL}(\pi\| \pi_{ref})$$

DPO的推导表明，这个优化问题等价于直接优化以下目标：

$$L_{DPO} = - \mathbb{E}_{(x, y_w, y_l) \sim D} \left[ \log \sigma \left( \beta \log \frac{\pi_\theta(y_w|x)}{\pi_{ref}(y_w|x)} - \beta \log \frac{\pi_\theta(y_l|x)}{\pi_{ref}(y_l|x)} \right) \right]$$

其中 $y_w$ 是偏好回答（chosen），$y_l$ 是不偏好回答（rejected），$\sigma$ 是sigmoid函数。

这个公式的直观理解是：让模型给偏好回答分配更高的概率（相对于参考模型），给不偏好回答分配更低的概率。

### 7.2 DPO的优势

DPO相比PPO有以下几个显著优势：

**简化流程**：不需要训练单独的奖励模型，不需要进行强化学习采样，也不需要价值函数。整个流程就是一个简单的对比学习/分类任务。

**训练稳定**：PPO需要处理 Advantage Estimation 等复杂步骤，DPO直接优化目标函数，训练过程更加稳定。

**计算高效**：PPO需要同时加载策略模型、价值函数、参考模型等多个模型，DPO只需要策略模型和参考模型，显存占用大幅降低。

**调参简单**：PPO有很多超参数需要调（KL系数、clip范围、学习率调度等），DPO的超参数相对较少。

### 7.3 DPO的注意事项

DPO虽然简单，但也有一些需要注意的地方：

**数据质量很重要**：DPO的效果很大程度上取决于偏好数据的质量。如果偏好标注不一致或噪声很大，DPO的效果也会受影响。

**KL系数$\beta$的选择**：$\beta$ 控制了优化过程中对参考策略的偏离程度。$\beta$ 太小，模型可能学不到足够的信息；$\beta$ 太大，模型可能难以偏离参考策略。需要根据具体任务调整。

**训练不当时可能过度优化**：如果训练数据中偏好不够一致，DPO可能会过度拟合某些偏好模式。

## 8. 数据构建：如何构造高质量的SFT/偏好数据

无论使用哪种微调方法，数据质量都是决定成败的关键因素。这一部分专门聊聊如何构建高质量的训练数据。

### 8.1 SFT数据构建

SFT需要的是"指令-回答"对。构建这类数据有几种常见方法：

**人工标注**：最可靠但成本最高。找来专业的标注员，让他们根据给定的指令写出高质量的回答。人工标注的优势是可以严格控制回答的质量和风格，劣势是成本高、速度慢、难以大规模生产。

**LLM生成**：用强模型（如GPT-4）来生成训练数据。做法是先设计一批高质量的种子指令，然后用LLM生成对应的回答。生成的回答需要人工审核或用自动方法筛选。这种方法成本低、扩展性强，但质量依赖于生成模型的能力。

**数据增强**：通过对现有数据进行改写、扩写等操作，生成更多训练样本。比如把一个简短的回答改写成详细版本，或者把正式的回答改写成口语版本。

构建SFT数据时需要注意几个原则：

**多样性**：指令要覆盖不同的任务类型、难度级别、领域知识。单一类型的指令会导致模型过拟合到特定模式。

**高质量回答**：回答要准确、完整、有帮助。避免有事实错误的回答、过于冗长或过于简略的回答、不相关或有害的回答。

**格式一致性**：如果模型需要学会特定输出格式，整个数据集的格式要保持一致。

### 8.2 偏好数据构建

偏好数据用于训练奖励模型或DPO，通常是三元组形式：(问题, 回答A, 回答B)，加上一个偏好标签（哪个回答更好，或同样好）。

**成对比较标注**：这是最常见的偏好数据格式。让标注员比较同一问题的两个回答，选出更好的那个（或标记同样好、同样差）。

标注时需要给标注员明确的评判标准，比如：哪个回答更准确？哪个回答更有帮助？哪个回答更符合特定风格？不同维度的权重如何分配？

**排序标注**：有时候不只是两个回答比较，而是对多个回答进行排序。这可以获得更丰富的偏好信息，但标注成本也更高。

**LLM辅助标注**：用强模型（如GPT-4）来辅助标注或生成偏好数据。比如让模型生成多个候选回答，然后用另一个模型来比较排序。这种方法可以大幅降低标注成本，但可能引入模型偏见。

### 8.3 数据质量控制

无论用什么方法构建数据，质量控制都是必不可少的环节：

**一致性检查**：检查数据是否有格式错误、缺失字段、重复内容等明显问题。

**抽样审核**：定期从数据中抽样，人工审核样本质量。建立清晰的质量评分标准，确保标注员之间的一致性。

**分布分析**：分析数据的分布，确保覆盖了目标场景的各种情况。避免在某些类型上过拟合，在另一些类型上欠拟合。

**去偏处理**：检查数据中是否存在系统性偏见，比如某些群体总是得到负面描述，某些观点被过度代表等。

## 9. 代码实战：用PEFT库（LoRA/QLoRA）微调

现在进入实战环节。先介绍用PEFT库进行LoRA/QLoRA微调的具体代码。

### 9.1 环境准备

首先安装必要的库：

```bash
pip install transformers peft accelerate bitsandbytes
pip install datasets trl
```

### 9.2 加载模型和配置LoRA

```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model, prepare_model_for_kbit_training

# 模型名称（以Mistral为例）
model_name = "mistralai/Mistral-7B-v0.1"

# QLoRA配置 - 4位量化
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_use_double_quant=True,  # 双重量化，进一步节省显存
    bnb_4bit_quant_type="nf4",       # 4-bit NormalFloat量化
    bnb_4bit_compute_dtype=torch.bfloat16
)

# 加载分词器
tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token  # 设置padding token

# 加载量化模型
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto",
    trust_remote_code=True
)

# 准备模型进行k-bit训练
model = prepare_model_for_kbit_training(model)
```

### 9.3 配置LoRA参数

```python
# LoRA配置
lora_config = LoraConfig(
    r=64,                           # 秩，越大表达能力越强
    lora_alpha=16,                  # 缩放因子，通常设为rank的倍数
    target_modules=[                 # 应用LoRA的模块
        "q_proj",                   # Query投影
        "v_proj",                   # Value投影
        "k_proj",                   # Key投影
        "o_proj",                   # 输出投影
    ],
    lora_dropout=0.05,              # dropout概率
    bias="none",                    # bias处理方式
    task_type="CAUSAL_LM"           # 任务类型：因果语言模型
)

# 应用LoRA到模型
model = get_peft_model(model, lora_config)

# 打印可训练参数数量
model.print_trainable_parameters()
# 输出类似: "trainable params: 83,886,080 || all params: 3,530,138,576 || trainable%: 2.375"
```

### 9.4 准备训练数据

```python
from datasets import load_dataset

# 加载数据集（以OpenAssistant的对话数据为例）
dataset = load_dataset("OpenAssistant/oasst_top1")

# 定义预处理函数
def format_example(example):
    # 构建对话格式
    messages = example["messages"]
    text = ""
    for msg in messages:
        if msg["role"] == "user":
            text += f"<|user|>{msg['content']}<|endoftext|>"
        elif msg["role"] == "assistant":
            text += f"<|assistant|>{msg['content']}<|endoftext|>"
    return {"text": text}

# 应用预处理
dataset = dataset.map(format_example)

# 分词
def tokenize(example):
    result = tokenizer(
        example["text"],
        truncation=True,
        max_length=512,
        padding="max_length"
    )
    result["labels"] = result["input_ids"].copy()
    return result

dataset = dataset.map(tokenize, remove_columns=["text"])
```

### 9.5 配置训练参数并开始训练

```python
from transformers import TrainingArguments
from trl import SFTTrainer

# 训练参数
training_args = TrainingArguments(
    output_dir="./lora-mistral",           # 输出目录
    num_train_epochs=3,                     # 训练轮数
    per_device_train_batch_size=4,         # 每个设备的batch size
    gradient_accumulation_steps=4,         # 梯度累积步数
    learning_rate=2e-4,                    # 学习率
    warmup_ratio=0.03,                     # warmup比例
    lr_scheduler_type="cosine",            # 学习率调度器
    logging_steps=10,                       # 日志记录间隔
    save_steps=100,                         # 保存检查点间隔
    fp16=True,                              # 使用半精度
    report_to="wandb",                      # 报告到wandb
)

# 创建SFTTrainer
trainer = SFTTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset["train"],
    dataset_text_field="text",
    max_seq_length=512,
)

# 开始训练
trainer.train()
```

### 9.6 模型推理

训练完成后，加载模型进行推理：

```python
from peft import PeftModel

# 加载基础模型
base_model = AutoModelForCausalLM.from_pretrained(
    model_name,
    quantization_config=bnb_config,
    device_map="auto",
    trust_remote_code=True
)

# 加载LoRA权重
model = PeftModel.from_pretrained(base_model, "./lora-mistral/checkpoint-100")

# 合并权重（可选，合并后推理更快）
model = model.merge_and_unload()

# 生成示例
prompt = "<|user|>给我写一个Python快速排序函数<|endoftext|><|assistant|>"
inputs = tokenizer(prompt, return_tensors="pt").to("cuda")

outputs = model.generate(
    **inputs,
    max_new_tokens=256,
    temperature=0.7,
    top_p=0.9,
    do_sample=True
)

print(tokenizer.decode(outputs[0], skip_special_tokens=False))
```

## 10. 代码实战：用TRL库实现DPO训练

TRL（Transformer Reinforcement Learning）是Hugging Face开发的专门用于RLHF训练的库。这一部分演示如何用TRL进行DPO训练。

### 10.1 准备偏好数据

DPO需要偏好数据，格式是包含"prompt"、"chosen"（偏好回答）、"rejected"（不偏好回答）的数据集：

```python
from datasets import Dataset
import pandas as pd

# 示例偏好数据
data = {
    "prompt": [
        "解释一下什么是量子纠缠",
        "给我写一个Python的快速排序",
        "为什么天空是蓝色的"
    ],
    "chosen": [
        "量子纠缠是量子力学中一种非常神奇的现象。当两个粒子处于纠缠态时，无论它们相距多远，对其中一个粒子的测量会立即影响另一个粒子的状态。这意味着测量一个粒子的某个属性（比如自旋），会瞬间决定另一个粒子的对应属性，即使它们相隔数万光年。爱因斯坦曾称之为"鬼魅般的超距作用"，这也是量子力学与经典物理最大的区别之一。",
        "def quicksort(arr):\n    if len(arr) <= 1:\n        return arr\n    pivot = arr[len(arr) // 2]\n    left = [x for x in arr if x < pivot]\n    middle = [x for x in arr if x == pivot]\n    right = [x for x in arr if x > pivot]\n    return quicksort(left) + middle + quicksort(right)\n\n# 测试\nprint(quicksort([3,6,8,10,1,1,2]))",
        "天空呈现蓝色是因为瑞利散射效应。当阳光进入大气层时，会与空气分子发生散射。阳光中波长较短的蓝光比波长较长的红光更容易被散射，而且散射强度与波长的四次方成反比。因此蓝色光被散射到各个方向，使得整个天空看起来是蓝色的。日落时天空呈红色是因为阳光需要穿过更厚的大气层，蓝光几乎被完全散射掉，只剩下红橙色的光。"
    ],
    "rejected": [
        "量子纠缠就是两个粒子连在一起。",
        "用sort函数不就行了？arr.sort()",
        "因为光的折射。"
    ]
}

# 创建数据集
df = pd.DataFrame(data)
dataset = Dataset.from_pandas(df)
print(dataset)
```

### 10.2 加载模型

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from trl import DPOConfig, DPOTrainer

# 加载预训练模型作为参考
model_name = "gpt2"  # 用小模型gpt2做演示，生产环境用更大的模型

model = AutoModelForCausalLM.from_pretrained(model_name)
tokenizer = AutoTokenizer.from_pretrained(model_name)
tokenizer.pad_token = tokenizer.eos_token
```

### 10.3 配置DPO训练

```python
# DPO训练参数
training_args = DPOConfig(
    output_dir="./dpo-gpt2",
    num_train_epochs=3,
    per_device_train_batch_size=2,
    gradient_accumulation_steps=4,
    learning_rate=1e-5,
    warmup_ratio=0.1,
    logging_steps=10,
    save_steps=100,
    beta=0.1,                        # KL散度惩罚系数
    report_to="wandb",
)

# 创建DPO Trainer
dpo_trainer = DPOTrainer(
    model=model,
    args=training_args,
    train_dataset=dataset,
    tokenizer=tokenizer,
    max_prompt_length=128,
    max_length=512,
)
```

### 10.4 开始DPO训练

```python
# 开始训练
dpo_trainer.train()
```

### 10.5 用训练好的模型生成

```python
# 生成示例
prompt = "解释一下什么是量子纠缠"

# Tokenize
inputs = tokenizer(prompt, return_tensors="pt")

# 生成
outputs = model.generate(
    **inputs,
    max_new_tokens=200,
    temperature=0.7,
    do_sample=True
)

result = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(result)
```

### 10.6 DPO训练的技巧

在实际使用DPO时，有几个值得注意的点：

**beta参数的选择**：beta控制着参考模型的影响程度。beta太小，模型可能偏离参考模型太远；beta太大，训练效果不明显。通常从0.1开始尝试，根据效果调整。

**数据格式一致性**：确保"chosen"和"rejected"回答格式一致，比如都用相同的特殊token包裹。

**训练长度控制**：设置合理的max_prompt_length和max_length，既要容纳输入，又要留足够空间给输出。

**评估指标**：DPO训练时可以监控的是DPO loss、KL散度、reward margin等指标。loss下降不代表效果一定好，最终还是要看实际生成质量。

## 总结

这篇文章我们系统地介绍了SFT和RLHF两大LLM微调范式：

**SFT（监督微调）**是最基础的微调方式，用高质量的"指令-回答"数据让模型学会执行特定任务。它的优点是简单直接，缺点是需要大量精心标注的数据。

**LoRA/QLoRA**是参数高效微调的代表技术，通过低秩分解和量化，大大降低了微调的显存门槛。LoRA的rank参数控制表达能力，QLoRA在此基础上加入了NF4量化，使得在消费级显卡上微调大模型成为可能。

**RLHF三阶段**（SFT→奖励模型→PPO）是一个完整的对齐框架，先让模型具备基本能力，再用偏好数据训练奖励模型，最后用强化学习优化策略。PPO中的KL散度约束和clip机制是保证训练稳定性的关键。

**DPO（直接偏好优化）**是近年来兴起的新方法，通过数学变换把RLHF的强化学习问题转化成一个简单的对比学习问题，大大简化了训练流程，降低了计算开销。

**数据构建**是所有微调方法的基础。高质量的SFT数据和偏好数据往往比算法改进更能提升模型效果。

在实际应用中，选择哪种方法取决于你的具体需求、可用资源和数据情况。如果你有充足的标注数据，SFT可能就足够了；如果你想让模型更好地符合人类偏好，可以尝试DPO；如果你有更复杂的对齐需求，完整的RLHF流程可能更合适。

微调是一个不断迭代的过程：准备好数据，选择合适的方法，训练模型，评估效果，根据反馈调整数据和算法，循环往复。没有一劳永逸的解决方案，但有了这篇文章的基础知识，你应该能够更自信地开始自己的微调之旅了。
