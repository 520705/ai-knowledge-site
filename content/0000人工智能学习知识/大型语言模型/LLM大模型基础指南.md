---
title: LLM大模型基础指南
date: 2026-04-24
tags:
  - 大语言模型
  - LLM
  - GPT
  - Transformer
aliases: [LLM基础, 大模型基础]
---

# LLM大模型基础指南

大型语言模型（Large Language Model，LLM）是这两年最火的技术方向，从ChatGPT爆火开始，几乎各行各业都在讨论它到底能做什么、会不会取代某些工作。但很多人在学习LLM的时候，容易陷入两个极端：要么只懂调API调用，不知其所以然；要么被论文和公式吓退，不知道从哪里入门。这篇指南的目标，就是让你在理论和实战之间找到一条清晰的学习路径——既理解LLM背后的核心原理，也能动手跑起来。

## GPT系列演进：从量变到质变

要理解今天的大语言模型，必须从GPT系列说起。GPT的全称是Generative Pre-trained Transformer，翻译过来就是"生成式预训练Transformer"。这个命名本身就揭示了它的核心技术路线：用Transformer架构做预训练，然后用它来生成文本。

### GPT-1：一切的开始

2018年，OpenAI发布了GPT-1，这篇论文叫《Improving Language Understanding by Generative Pre-Training》。那时候GPT-1只有1.17亿参数，说实话规模不大，但它提出的预训练+微调范式影响深远。GPT-1的核心思路是：先在大规模无标注文本上做无监督预训练，学习语言的通用表示；然后在特定任务上做有监督微调。这个两阶段范式，后来成了几乎所有大模型的标配。

GPT-1用的是标准的Transformer decoder架构，堆了12层transformer块，768维隐藏状态，12个注意力头。它在各种自然语言理解任务上取得了不错的效果，但参数量限制了它的能力上限。

### GPT-2：走向生成

2019年GPT-2发布，参数量飙到15亿，一口气涨了十几倍。更关键的是，GPT-2展示了大规模语言模型的一个惊人能力：**无需特定任务微调，仅凭Prompt就能完成多种任务**。OpenAI当时甚至因为担心这个模型太强被用来生成假新闻，只发布了缩小版模型和部分论文。

GPT-2的核心洞察是，语言模型本质上是一个通用的任务求解器。当你训练一个足够强大的语言模型时，它能够根据指令和上下文推断出你想让它做什么，而不是只能做某一种特定任务。这种"涌现"的泛化能力，让研究者看到了Scaling的曙光。

### GPT-3：大力出奇迹

2020年的GPT-3是一个标志性节点。1750亿参数，45TB训练数据，耗电量据说相当于一个小型发电站几个月的产出。GPT-3不再需要微调——给它几个示例（Few-shot Learning）或者干脆只给指令（Zero-shot Learning），它就能完成翻译、写作、代码生成、数学推理等各种任务。

GPT-3的论文标题叫《Language Models are Few-Shot Learners》，它证明了模型的规模和训练数据量到了一定程度，泛化能力会产生质的飞跃。但GPT-3也有明显局限：推理速度慢、成本高、幻觉问题严重，而且它本质上还是基于概率的"文字接龙"，并没有真正的"理解"能力。

### GPT-4：多模态与安全性

GPT-4在2023年3月发布，具体参数量OpenAI没有公布，但从各项能力推断应该是万亿级别。GPT-4最大的进步有两点：一是引入了多模态能力，可以处理图像输入；二是大幅提升了推理能力和安全性。GPT-4在各种专业考试（律师考试、医学考试、编程竞赛）上的表现已经达到或超过人类平均水平。

不过GPT-4的训练成本据说高达数亿美元，这让绝大多数研究机构望而却步，也直接推动了开源社区寻找更高效训练方法的热情。

### GPT-4o与最新进展

2024年5月，OpenAI发布了GPT-4o（"o"代表Omni），实现了真正的原生多模态——不再需要单独的视觉编码器，文本、音频、图像统一在一个模型里处理。GPT-4o的响应延迟大幅降低，语音对话的体验接近人类自然交流。

更重要的是，OpenAI开始推出更小但能力依然强大的模型如GPT-4o mini，以及开源了一些技术论文和模型权重（虽然不是完全开源）。整个领域正在从"唯参数论"转向效率与能力并重的阶段。

## LLaMA系列：开源阵营的崛起

如果说GPT系列是闭源模型的代表，那LLaMA就是开源阵营的旗手。LLaMA由Meta（原Facebook）在2023年2月发布，迅速成为开源大模型的事实标准。

### LLaMA-1：参数效率的突破

LLaMA-1最惊人的地方在于**参数效率**。它最小的版本只有70亿参数，但在很多任务上的表现居然可以和1750亿参数的GPT-3掰手腕。Meta是怎么做到的？答案是在**训练数据质量**和**训练时长**上做文章。

GPT-3的训练数据来源比较杂，质量参差不齐。LLaMA-1只用了高质量的文本数据，包括Wikipedia、Books、CommonCrawl的一个精选子集，以及GitHub和ArXiv的学术论文。虽然数据量少，但质量高，训练步数足够长，最终效果反而更好。

LLaMA-1开源了7B、13B、33B、65B四个版本（1B是"十亿参数"）。65B版本在多数基准测试上可以媲美PaLM（Google的5400亿参数模型）。这个"小模型打败大模型"的结果，给整个社区注入了一针强心剂——训练大模型不一定要堆参数，数据质量和方法同样重要。

### LLaMA-2：开源许可的妥协与进步

2023年7月发布的LLaMA-2在几个方面做了升级。模型规模扩展到了70B，引入了RLHF（基于人类反馈的强化学习）微调，发布了专门的对话模型LLaMA-2 Chat。

不过LLaMA-2的开源许可引发了一些争议。它允许免费商用，但有附加条款：如果月活跃用户超过7亿，需要向Meta申请许可。这实际上是在保护Meta的商业利益——毕竟如果有个超级App用LLaMA-2做到7亿用户，Meta肯定不想白白错过这个流量入口。

### LLaMA-3：开源模型的性能飞跃

2024年4月发布的LLaMA-3将开源模型的性能推到了新高度。8B和70B的版本在MMLU（大规模多任务语言理解）等基准上大幅领先上代产品，405B的巨无霸版本更是直接对标GPT-4。Meta还训练了一个专门用于工具使用和代码生成的变体LLaMA-3-Instruct。

LLaMA-3的成功秘诀之一是扩展了训练数据量，据说训练语料达到了15T tokens（15万亿token），是LLaMA-2的数倍。同时Meta也在训练方法、RLHF流程上做了大量优化。

围绕LLaMA生态，涌现出了Alpaca、Vicuna、Orca等微调版本，以及llama.cpp、Ollama、LM Studio等推理框架。你可以在自己的电脑上跑起一个70B的LLaMA模型（虽然需要足够的显存），这在GPT-4时代是不可想象的。

## 国产大模型：百花齐放

2023年开始，国内大模型呈现爆发式增长。百度文心、阿里通义、讯飞星火、智谱ChatGLM、百川智能、DeepSeek等纷纷登场，竞争激烈。

**ChatGLM**由智谱AI研发，基于GLM（General Language Model）架构，中文能力出色。ChatGLM3引入了多模态能力，开源的6B版本可以在消费级显卡上运行，是很多开发者的首选。

**Qwen（通义千问）**来自阿里云，开源了Qwen-1.5、Qwen-2系列，其中Qwen2-72B在多项国际基准上表现优异。阿里采取了相对开放的开源策略，商用友好，生态建设也比较完善。

**DeepSeek**是最近两年崛起的新势力，由幻方量化孵化。DeepSeek-V2提出了MLA（Multi-head Latent Attention）注意力机制，在保持高性能的同时大幅降低显存占用。DeepSeek-Math在数学推理上表现尤为突出。他们的开源策略也比较激进，引起了社区的广泛关注。

国产大模型的共同特点是**中文优化做得好**，在中文理解、文化背景、知识表达上有明显优势。对国内开发者来说，选择国产模型意味着更好的中文交互体验和更低的合规风险。

## Scaling Law：规模的神话与边界

Scaling Law（缩放定律）是理解大模型最重要的理论框架之一。它描述的是：**模型的性能如何随着参数量、训练数据量、计算量的增加而变化**。

2020年OpenAI在论文《Scaling Laws for Neural Language Models》中给出了一个核心发现：模型性能（用困惑度衡量）和模型参数量、数据集大小、计算量之间存在幂律关系——即双对数坐标下是一条直线。这意味着，只要按比例增加这三个要素，模型性能就能可预测地提升。

具体来说，如果把计算预算增加10倍，最优策略是把模型参数量增加约5倍，训练token数增加约2倍。也就是说，不能只增大模型，数据量也要同步跟上。

Google DeepMind在2022年的论文《Training Compute-Optimal Large Language Models》（Chinchilla论文）中进一步指出，之前的大模型普遍**训得不够**——模型参数量增大的速度远快于数据量。Chinchilla（70B）的提出者认为，如果计算预算固定，模型参数量和数据量的最优比例大约是1:20，而不是GPT-3那样的1:15左右。

后来Meta的LLaMA系列就践行了Chinchilla的思路：用相对小但高质量的数据集训练更长时间，取得了优异效果。

不过Scaling Law也告诉我们，**Scaling的收益在递减**——从100B到200B参数带来的提升，不如从1B到10B那么显著。到了万亿参数级别，继续Scaling的边际收益越来越小，这也是为什么研究者开始转向效率优化（更少的参数、更低的计算量达到同样效果）。

## 涌现能力：超越训练的智慧

涌现能力（Emergent Abilities）是LLM最神奇也最令人困惑的特性。简单来说，涌现能力指的是：**某些能力在小模型上完全不存在，但当模型规模超过某个临界点后，突然就出现了**。这种"突然涌现"的现象，无法通过小模型的Scaling直接预测。

### 思维链（Chain-of-Thought）

思维链是最著名的涌现能力之一。2022年Google的论文《Chain-of-Thought Prompting Elicits Reasoning in Large Language Models》发现，当给大模型提供一些"展示推理过程"的示例时，它的数学和推理能力会大幅提升。

有趣的是，这种能力**不会在小模型上出现**。一个7B的模型，你给它展示思维链，它可能学不会推理步骤，反而被干扰；但一个100B以上的模型，思维链Prompt能显著提升它的准确率。这个临界点大约在100B参数左右——当然，随着技术进步，临界点也在逐渐下降。

涌现能力的机制至今没有完美的理论解释。一种解释是：大模型在训练过程中见过足够多的推理范例，从而学会了"推理的模式"。另一种解释是：大模型有足够大的参数空间，可以隐式模拟复杂的计算图，思维链只是激活了这些潜在的计算能力。

### 代码能力

代码生成也是涌现能力的一个典型案例。小模型的代码输出往往语法混乱、逻辑错误；大模型则可以写出结构清晰、逻辑正确的代码，甚至能根据注释生成完整的程序。

GPT-4在HumanEval基准（手写的164道编程题）上的通过率达到67%，远超GPT-3的5%左右。这个飞跃发生在参数量跨越某个阈值之后，不是线性渐进的。

涌现能力给我们的启示是：**在某个规模之前，不要轻易判断一个大模型"没有某种能力"，它可能只是还没到临界点**。当然，这也意味着Scaling和算法改进需要同时进行——单纯增大模型不是万能的。

## Flash Attention：让注意力不再拖后腿

Transformer的核心问题是**注意力计算的复杂度是O(n²)**——序列长度翻倍，计算量要翻四倍。这在长上下文场景下简直是灾难。Flash Attention就是为了解决这个问题。

### 标准Attention的瓶颈

回顾一下标准Self-Attention的计算过程：

1. 输入序列X（长度n，维度d）经过三个线性变换，得到Q、K、V
2. 计算注意力分数：S = Q × K^T
3. 对S做Softmax归一化
4. 用归一化后的权重乘以V得到输出

问题出在第2、3步。Q × K^T是一个n×n的矩阵，当n=8192时，这个矩阵有6700万个元素。如果用FP16（半精度浮点）存储，就需要134MB显存——光这一个矩阵就够呛了。更别说后面还要存完整的注意力矩阵用于反向传播。

### 分块计算与重计算

Flash Attention的核心思想是**分块计算（tiling）**：不再一次性计算完整的注意力矩阵，而是把Q、K、V分成多个小块（tiles），一块一块地处理，每次只需要把一小块加载到显存中。

具体来说，Flash Attention把K和V分成T个块，每次取一个Q块，去和K块做点积、Softmax，然后和对应的V块加权求和。由于每次只加载需要的块，显存占用从O(n²)降低到了O(n)。

代价是计算量略有增加（因为要多次读写显存），但显存节省是革命性的——这也是为什么今天能在有限的GPU显存上运行长上下文模型。

另一个关键技术是**重计算（Recomputation）**。标准Attention在反向传播时需要保存完整的注意力矩阵用于梯度计算。Flash Attention不保存它，而是在反向传播时用保存好的统计量（Softmax的分子分母）重新计算出梯度，这样把显存占用又降了一个量级。

Flash Attention由Tri Dao（现在是Princeton的助理教授）等人提出，2022年的Flash Attention论文和2023年的Flash Attention-2论文逐步优化，目前已经成了训练和推理大模型的标配。Hugging Face、vLLM、DeepSpeed等主流框架都集成了Flash Attention。

## Rotary Position Embedding：位置编码的进化

位置编码（Position Embedding）是Transformer处理序列顺序的关键。标准Transformer用的是**绝对位置编码**（Absolute Position Embedding），给每个位置分配一个固定的向量。但这种方法有两个问题：无法处理超过训练时见过的最长序列，对相对位置关系的表达能力有限。

Rotary Position Embedding（RoPE）是2021年由苏剑林（Su Jianlin）提出的一种新方案，现在被LLaMA、GLM、Qwen等主流开源模型广泛采用。

### 旋转的智慧

RoPE的核心思想是：**用旋转矩阵来表示位置信息**。对于一个d维的词向量，RoPE把它每两个维度配成一对，然后对每一对施加不同频率的旋转。

具体来说，位置m的旋转矩阵是一个分块对角矩阵，每2×2的块对应一对维度，旋转角度是m×θ，其中θ是预设的角度参数：

```
cos(mθ)  -sin(mθ)
sin(mθ)   cos(mθ)
```

当我们要计算位置m和位置n两个词的注意力时，只需要把Query和Key都旋转到对应位置，然后做点积。由于旋转矩阵的正交性，点积结果只依赖于位置的**相对距离m-n**，这正是我们想要的！

### 远程衰减特性

RoPE还有一个优雅的特性：**远程衰减**。随着相对距离增大，两个位置之间的点积会逐渐减小，这意味着模型天然会更多关注近距离的上下文，不需要额外的衰减函数。这在处理长文档时特别有价值。

相比之下，标准的位置编码对于所有位置一视同仁，需要额外的ALiBi（Attention with Linear Biases）技术来添加位置偏好。RoPE把位置信息编码在了向量本身的几何结构中，更紧凑、更自然。

## GQA与MQA：注意力机制的效率革命

标准的Multi-Head Attention（MHA）中，每个注意力头都有独立的Q、K、V三个投影矩阵。假设模型有h个注意力头，那么就有h组K和h组V。对于一个L层的模型，K和V的参数量是MHA的h倍——这在h=32、64时是巨大的显存负担。

### Multi-Query Attention（MQA）

MQA的思路很简单：**所有注意力头共享同一组K和V**，只有Q是每头独立的。这样K和V的参数量从h组变成1组，Key-Value缓存的显存占用也大幅降低。

MQA最早在2019年的论文《Fast Transformer Decoding: One Write-Head is All You Need》中提出，最初用于加速Transformer推理。研究发现，MQA在大幅加速推理的同时，模型质量只有轻微下降（多数任务<0.5%的性能损失）。这个trade-off对于实际应用来说非常划算。

### Group Query Attention（GQA）

MQA的问题是所有注意力头共享一组K、V，这可能限制了模型的多样性表达能力。GQA是MQA和MHA的折中：**把注意力头分成G个组，每组共享一组K和V**。

当G等于h时，GQA退化为MHA；当G等于1时，GQA退化为MQA。通过调整G，可以在**推理效率和模型质量**之间找到平衡点。

LLaMA-2采用了GQA，DeepSeek-V2提出了更激进的MLA（Multi-head Latent Attention）通过低秩分解进一步压缩KV缓存。Google的Gemma-2也采用了GQA。这个趋势很明确：**未来大模型都会采用某种形式的分组合并注意力**，标准MHA会逐渐成为历史。

## 代码实战：用transformers库玩转LLM

终于到了实战环节。我们用Hugging Face的transformers库来加载和使用大语言模型。

### 环境准备

```bash
pip install transformers torch accelerate sentencepiece
```

如果你的GPU显存有限（<16GB），建议使用量化版本，需要额外安装：

```bash
pip install bitsandbytes
```

### 基础加载与推理

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

# 选择一个中等规模的模型，8B参数
model_name = "meta-llama/Llama-3.2-3B-Instruct"  # 3B参数版本，显存友好

# 加载tokenizer
tokenizer = AutoTokenizer.from_pretrained(model_name)
# 设置pad token（某些模型需要）
if tokenizer.pad_token is None:
    tokenizer.pad_token = tokenizer.eos_token

# 加载模型到GPU
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.float16,  # 半精度节省显存
    device_map="auto",          # 自动分配到可用GPU
)

# 构造对话
messages = [
    {"role": "system", "content": "你是一个乐于助人的AI助手。"},
    {"role": "user", "content": "解释一下什么是大语言模型，用简单的语言。"}
]

# 使用chat template格式化输入
input_text = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True
)

# 分词
inputs = tokenizer(input_text, return_tensors="pt").to(model.device)

# 生成
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=512,
        temperature=0.7,
        top_p=0.9,
        do_sample=True,
    )

# 解码输出
response = tokenizer.decode(outputs[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)
print(response)
```

### 使用量化版本加载超大模型

如果你的显存只有8GB甚至更少，可以用量化技术压缩模型：

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
import torch

# 4-bit量化配置
quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_compute_dtype=torch.float16,
    bnb_4bit_quant_type="nf4",      # NF4格式，量化精度更高
    bnb_4bit_use_double_quant=True,  # 双重量化，进一步压缩
)

# 加载70B参数模型（需要~14GB显存，量化后）
model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3.1-8B-Instruct",
    quantization_config=quantization_config,
    device_map="auto",
)
```

### 自定义生成策略

```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen2.5-7B-Instruct")
model = AutoModelForCausalLM.from_pretrained(
    "Qwen/Qwen2.5-7B-Instruct",
    torch_dtype=torch.float16,
    device_map="auto",
)

# 构造带few-shot示例的prompt
prompt = """请根据以下示例，判断句子的情感是正面还是负面。

示例1：
文本：这家餐厅的服务太差了，等了半小时才上菜。
情感：负面

示例2：
文本：这部电影太精彩了，全程无尿点！
情感：正面

请判断：
文本：产品中规中矩，没有特别亮眼的地方。
情感："""

inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

# 使用贪心解码（适合确定性输出）
with torch.no_grad():
    outputs = model.generate(
        **inputs,
        max_new_tokens=20,
        do_sample=False,  # 关闭随机采样
    )

result = tokenizer.decode(outputs[0][inputs["input_ids"].shape[1]:], skip_special_tokens=True)
print(f"情感判断：{result}")
```

### 使用vLLM加速推理

对于生产环境，transformers的原生推理速度往往不够快。vLLM是目前最流行的推理加速框架：

```python
from vllm import LLM, SamplingParams

# 初始化vLLM引擎
llm = LLM(
    model="deepseek-ai/DeepSeek-V2.5",
    tensor_parallel_size=2,  # 多卡并行
    trust_remote_code=True,
)

# 设置采样参数
sampling_params = SamplingParams(
    temperature=0.7,
    top_p=0.9,
    max_tokens=1024,
)

# 批量推理
prompts = [
    "用Python写一个快速排序",
    "解释牛顿力学三大定律",
    "写一首关于秋天的诗",
]

outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    print(f"Prompt: {output.prompt}")
    print(f"Generated: {output.outputs[0].text}")
    print("---")
```

vLLM通过PagedAttention技术管理KV缓存，支持连续批处理（Continuous Batching），推理速度比Hugging Face原生实现快数倍到数十倍，是生产环境部署的首选。

### 本地运行：Ollama的极简体验

如果你连配置Python环境都觉得麻烦，可以用Ollama一行命令跑起大模型：

```bash
# 安装Ollama后
ollama pull llama3.2        # 下载模型
ollama run llama3.2         # 启动对话
```

```python
import ollama

response = ollama.chat(
    model="llama3.2",
    messages=[
        {"role": "user", "content": "你好，请介绍一下你自己"}
    ]
)
print(response["message"]["content"])
```

Ollama支持Mac本地运行（利用Metal加速），Windows用户可以用WSL。它把模型管理、显存分配、推理优化都封装好了，适合快速原型验证。

## 结语

大语言模型的技术栈很深，从Transformer架构到Scaling Law，从注意力机制优化到推理加速，每一块都有大量值得深挖的内容。这篇指南只是一个起点，帮助你搭建起基本框架。

下一步，你可以选择几个方向深入：想理解原理就去读论文原始推导；想提升工程能力就去研究分布式训练和推理优化；想做应用就去探索Prompt工程和Agent开发。LLM领域发展极快，保持学习的同时也要有耐心——没有人能跟上所有进展，选择自己最感兴趣的方向深耕就够了。

祝你在LLM的学习旅程中有所收获。
