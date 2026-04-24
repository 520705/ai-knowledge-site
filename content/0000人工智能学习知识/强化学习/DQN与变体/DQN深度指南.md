---
title: DQN深度指南
date: 2026-04-18
tags:
  - 强化学习
  - 深度强化学习
  - DQN
  - 深度Q网络
  - 经验回放
  - 目标网络
categories:
  - 强化学习
  - 深度强化学习
  - 值函数方法
alias:
  - Deep Q-Network
  - DQN Variants
  - Experience Replay
---

# DQN深度指南

> [!abstract]+ 一句话理解DQN
> 用神经网络当"老司机"，记住每种局势下该走哪步棋，然后通过反复复盘自己的棋谱来变得越来越厉害。

## 关键词速览

| 核心概念 | 经验回放 | 目标网络 | 深度神经网络 | 过估计 |
|:---------|:---------|:---------|:-------------|:-------|
| Double DQN | Dueling DQN | PER | Rainbow | 梯度裁剪 |

> [!abstract]+ 核心关键词表
> | 术语 | 英文 | 符号/技术 | 说明 |
> |:-----|:-----|:----------|:-----|
> | 深度Q网络 | Deep Q-Network | DQN | 深度学习与Q学习的结合 |
> | 经验回放 | Experience Replay | Replay Buffer | 存储和重放转移的机制 |
> | 目标网络 | Target Network | $Q_{target}$ | 延迟更新的目标网络 |
> | 双重DQN | Double DQN | DDQN | 解决Q值过估计问题 |
> | 竞争DQN | Dueling DQN | $V(s) + A(s,a)$ | 分离状态价值和优势 |
> | 优先级回放 | PER | SumTree | 基于TD误差的采样优先级 |
> | Rainbow | Rainbow DQN | 组合方法 | 多种DQN变体的集成 |
> | 梯度裁剪 | Gradient Clipping | $\nabla$ | 防止梯度爆炸 |
> | 目标Q值 | Target Q-value | $y_j$ | TD学习的目标值 |
> | 贪心策略 | Greedy Policy | $\arg\max$ | 选择最优动作 |

---

## 一、为什么需要DQN：从Q表说起

### 1.1 Q学习的老毛病——表格装不下

在说DQN之前，咱们先聊聊它的老祖宗Q学习。Q学习的核心是维护一张表格，每个状态-动作对都对应一个Q值，记录着"在这个状态下选这个动作，将来能拿多少分"。这就像考试前背题库，把每道题和每个答案的组合都背下来。

问题来了：想象你玩《超级马里奥》，画面是256×240的像素，每个像素有256种颜色可能，状态数比宇宙里的原子还多！这张表格根本存不下，更别说填满了。

现实项目里问题更明显：

- **自动驾驶**：真实路况的状态空间是连续的、光子级细腻的，你没法穷举
- **机器人控制**：关节角度、速度、力矩全连着，状态数爆炸
- **游戏AI**：围棋有$10^{170}$种局面，Atari游戏每帧都是全新画面

所以Q表这条路，走到高维问题就堵死了。

### 1.2 神经网络登场：让机器自己"总结规律"

这时候科学家们想了个办法：与其死记硬背所有状态，不如让神经网络学会"规律"。

打个比方：你不需要记住"北京今天PM2.5是68，空气质量指数是89"，你只需要知道"PM2.5高、天气闷"意味着空气质量不好。神经网络就是干这个的——它能从海量数据里抽象出规律，然后**泛化**到没见过的情况。

所以DQN干的事情就是：

$$
Q(s,a|\theta) \approx Q^*(s,a)
$$

用参数为$\theta$的神经网络来**逼近**最优动作价值函数。输入一个状态，输出每个动作的Q值，神经网络自动学会哪些特征重要、怎么组合。

这就好比你不用背所有棋谱，而是学会了下棋的"棋感"——看到棋盘就知道哪步好。

### 1.3 DQN的核心挑战：三个坑和三个对策

深度学习和强化学习一结合，麻烦就来了。DQN的论文（Mnih et al., 2015）专门讲了三个大坑：

**第一个坑：数据不独立**

你在游戏里连着走了三步，这三步的画面肯定高度相关——场景没变，只是小人挪了一点。如果神经网络看到这种连续样本，会误以为它们是一类数据，结果学歪了。

**第二个坑：数据分布飘移**

强化学习里，你一边学一边玩，策略在变，数据来源也在变。神经网络最怕这种情况——老师出题风格一直在变，学生根本没法复习。

**第三个坑：目标值乱晃**

Q学习的更新公式是：

$$
Q(s,a) \leftarrow r + \gamma \max_{a'} Q(s',a')
$$

问题在于，等号右边的$\max_{a'} Q(s',a')$本身就是神经网络的预测。你每更新一次网络，这个"目标值"就跟着变一次。这就像你在射箭，每一箭的靶子都在移动，怎么练都练不准。

这三个坑，DQN分别用**经验回放**、**目标网络**和**固定目标**来解决。下面咱们一个个细说。

---

## 二、经验回放：为什么随机打散能打破相关性

### 2.1 直观理解：为什么不能按顺序学

先想象你在学游泳。正确的做法是什么？先练划水、再练换气、再练腿部动作，交叉练习、反复巩固。

但如果你按顺序学：今天只练划水，连着练1000次；明天只练换气，连着练1000次——效果肯定差。为什么？因为大脑需要**多样化的刺激**来建立鲁棒的特征。

强化学习也是这个道理。

智能体在环境里跑，会产生一连串经验：`(状态1, 动作1, 奖励1, 状态2)`, `(状态2, 动作2, 奖励2, 状态3)`... 相邻的经验高度相关——状态2既是第一条的"下一状态"，又是第二条的"当前状态"。

如果神经网络按顺序看到这些样本，它会发现"状态1和状态2长得很像，标签却完全不同"，这会让网络困惑，梯度估计的方差变大。

### 2.2 经验回放机制：把记忆存起来，随机抽样

DQN的做法是建一个**回放缓冲区**（Replay Buffer），把每条转移`(s, a, r, s', done)`都存进去。存够一批后，每次更新随机抽一批来训练。

```python
class ReplayBuffer:
    """经验回放缓冲区 - 简单实现"""
    
    def __init__(self, capacity=100000):
        # 用 deque 方便自动淘汰旧数据
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        """存入一条经验"""
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        """随机抽取一批经验"""
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (
            np.array(states),
            np.array(actions),
            np.array(rewards),
            np.array(next_states),
            np.array(dones)
        )
    
    def __len__(self):
        return len(self.buffer)
```

这样做有三个好处：

1. **打破时间相关性**：随机抽样让前后脚的经验被分开送进网络，网络不会"惯性思维"
2. **样本复用**：每条经验可以被训练很多次，数据效率暴增——要知道在Atari游戏里采集50M帧数据要花几十个小时
3. **稳定分布**：旧策略产生的经验被新策略稀释，数据分布变化是平滑的而非突变

### 2.3 经验回放调优：大小和预热都有讲究

实际使用时有几个经验值可以参考：

- **缓冲区大小**：100K到1M不等。太小会导致数据被快速覆盖（尤其是复杂任务）；太大会占用大量内存。Atari游戏一般用100万
- **预热期（warm-up）**：开始训练前先让智能体随机跑一段时间（比如50000步），填满缓冲区再开始学习。这样保证初始训练有足够多样的数据
- **批量大小**：32到128常见。显存够的话，偏大一点（64、128）训练更稳定，但收敛速度不一定更快

```python
# 典型的预热逻辑
if len(agent.memory) < 50000:  # 预热期，只收集数据不训练
    action = env.action_space.sample()  # 随机动作
else:
    action = agent.select_action(state)  # 开始用策略选择
```

---

## 三、目标网络：为什么固定目标能让训练稳下来

### 3.1 "移动靶"问题：最难缠的训练困境

想象你在学投篮，教练每次投完篮都把篮筐挪个位置。篮筐一会儿在左边、一会儿在右边，你永远投不准——因为你每次调整姿势，评判标准也在变。

神经网络训练也有这个问题。看这个损失函数：

$$
L(\theta) = \left(r + \gamma \max_{a'} Q(s',a'|\theta) - Q(s,a|\theta)\right)^2
$$

Loss里的目标值`r + γ * max Q(s',a')`本身依赖网络参数$\theta$。每更新一次$\theta$，目标值就跟着变，网络在追一个不断移动的靶子。

这在强化学习里叫"非平稳性问题"（non-stationarity），是训练不稳定的根源之一。

### 3.2 目标网络：引入一个"假想敌"

DQN的解法很巧妙：**再训练一个网络，专门用来生成目标值，这个网络的更新频率更低**。

这个目标网络的参数叫$\theta^-$，每隔$C$步才从主网络（也叫策略网络，参数$\theta$）复制一次。更新时：

$$
y_j = r_j + \gamma \max_{a'} Q_{target}(s'_j, a'|\theta^-)
$$

注意：这里的目标$Q_{target}$用的是**旧的参数**$\theta^-$，而$Q(s,a|\theta)$是主网络在用**当前参数**$\theta$。这样目标值在$C$步之内是固定的，网络的训练就稳了。

```python
class DQNAgent:
    def __init__(self, input_shape, n_actions):
        # 主网络 - 每次训练都更新
        self.policy_net = DQN(input_shape, n_actions)
        # 目标网络 - 隔C步同步一次
        self.target_net = DQN(input_shape, n_actions)
        self.target_net.load_state_dict(self.policy_net.state_dict())
        self.target_net.eval()  # 目标网络不参与训练
    
    def update(self, batch):
        # 计算当前Q值 - 用主网络
        q_values = self.policy_net(states).gather(1, actions.unsqueeze(1))
        
        # 计算目标Q值 - 用目标网络（detach()断开梯度）
        with torch.no_grad():
            target_q = rewards + self.gamma * self.target_net(next_states).max(1)[0]
        
        # 损失 + 反向传播 只更新主网络
        loss = nn.MSELoss()(q_values.squeeze(), target_q)
        loss.backward()
```

### 3.3 硬更新 vs 软更新：哪种更好

目标网络的更新有两种方式：

**硬更新**（Hard Update）：每隔$C$步直接复制参数。这是DQN原始论文的做法，$C$通常取10000。

```python
# 硬更新
if step % target_update_freq == 0:
    self.target_net.load_state_dict(self.policy_net.state_dict())
```

**软更新**（Soft Update）：每步都挪一点点，更平滑。公式是：

$$
\theta^- \leftarrow \tau \theta + (1-\tau)\theta^-
$$

$\tau$通常取0.005或0.001。

```python
# 软更新
tau = 0.005
for target_param, policy_param in zip(
    self.target_net.parameters(), 
    self.policy_net.parameters()
):
    target_param.data.copy_(
        tau * policy_param.data + (1 - tau) * target_param.data
    )
```

硬更新简单粗暴但有效；软更新更平滑，适合需要持续学习的场景，但引入了额外超参数。实战中硬更新更常用。

### 3.4 为什么不能每步都同步

你可能会问：每步都同步不行吗？目标值不是更准吗？

不行。想象每步都同步：主网络更新一点点，目标网络跟着变一点点，损失函数的目标值几乎每步都在晃，训练还是稳不住。核心在于**需要让目标值稳定足够久**，让网络有机会"消化"当前的误差。太频繁的同步等于没同步，太久不同步会导致目标值严重过时。10000步是一个经验值，效果好所以一直用。

---

## 四、ε-greedy探索：既不傻随机也不死脑筋

### 4.1 探索与利用的经典困境

强化学习里有个根本矛盾：**探索（Exploration）** 和 **利用（Exploitation）**。

- **利用**：用当前学到的知识选最优动作
- **探索**：尝试没见过的动作，可能发现更好的策略

只利用会让智能体困在局部最优；只探索会让智能体像没头苍蝇一样乱撞。ε-greedy是最简单的平衡方法：

- 以概率ε选随机动作（探索）
- 以概率1-ε选当前最优动作（利用）

```python
def select_action(self, state, epsilon):
    if random.random() < epsilon:
        return random.randrange(self.n_actions)  # 随机探索
    else:
        with torch.no_grad():
            state_tensor = torch.FloatTensor(state).unsqueeze(0)
            q_values = self.policy_net(state_tensor)
            return q_values.argmax().item()  # 利用已知最优
```

### 4.2 ε的衰减策略

ε从1.0（完全随机）开始，随着训练逐渐衰减到一个小值（比如0.01或0.1），常用的衰减方式：

**线性衰减**：简单直接

```python
epsilon = max(epsilon_min, epsilon_start - step * epsilon_decay)
```

**指数衰减**：前期衰减快，后期衰减慢

```python
epsilon = max(epsilon_min, epsilon_start * epsilon_decay ** step)
```

**分段衰减**：在某些里程碑（比如10%、50%训练进度）切换衰减率

Atari游戏的常用配置：ε从1.0起步，在100万帧内线性衰减到0.1，之后保持不变。

---

## 五、Double DQN：解决Q值过估计问题

### 5.1 过估计：max操作偏爱"吹牛"

标准DQN有个系统性的bug：**Q值会被高估**。

问题出在`max`操作上。看这个公式：

$$
\max_{a'} Q(s',a') \approx \max_{a'} (\text{真实值} + \text{噪声})
$$

假设每个Q值的估计都有误差$\epsilon$，有正有负。取max的时候，正误差更容易被选中——因为`max(真实+正误差)` > `max(真实)`。这就像在一群说自己"能跑100米"的人里选最快的，正吹牛的人更容易被选中。

长期累积下来，Q值会系统性地偏高，导致策略崩坏。

### 5.2 Double DQN的解法：解耦选择和评估

Double DQN（Van Hasselt et al., 2016）的思路很巧妙：**用两个网络分别做动作选择和价值评估**。

标准DQN的目标：
$$
y_j = r_j + \gamma \max_{a'} Q_{target}(s'_j, a'|\theta^-)
$$

Double DQN的目标：
$$
y_j = r_j + \gamma Q_{target}\left(s'_j, \arg\max_{a'} Q_{policy}(s'_j, a'|\theta)\right)
$$

区别在于：动作选择用**策略网络**（当前参数），价值评估用**目标网络**（旧参数）。这样选择和评估分开了，噪声不会被放大。

```python
def double_dqn_update(self, batch):
    """Double DQN的更新逻辑"""
    states, actions, rewards, next_states, dones = batch
    
    # 用策略网络选择下一个动作（更"当前"的判断）
    with torch.no_grad():
        next_actions = self.policy_net(next_states).argmax(1)
        # 用目标网络评估价值（避免自己夸自己）
        next_q_values = self.target_net(next_states).gather(
            1, next_actions.unsqueeze(1)
        ).squeeze(1)
    
    # 目标Q值
    target_q = rewards + self.gamma * next_q_values * (1 - dones)
    
    # 计算损失，只更新策略网络
    current_q = self.policy_net(states).gather(1, actions.unsqueeze(1)).squeeze(1)
    loss = nn.SmoothL1Loss()(current_q, target_q)  # 用Huber loss更鲁棒
    
    self.optimizer.zero_grad()
    loss.backward()
    torch.nn.utils.clip_grad_norm_(self.policy_net.parameters(), 10)
    self.optimizer.step()
```

### 5.3 Double DQN效果显著

实验表明，Double DQN在多个Atari游戏上带来了显著提升。以游戏《被困》为例：

- 标准DQN的Q值会持续高估，最终导致策略崩溃
- Double DQN的Q值估计更准确，训练更稳定
- 平均性能提升约11%

这个改进很简单，效果很显著，所以后来几乎所有DQN变体都用上了Double DQN。

---

## 六、Dueling DQN：分离"状态好不好"和"动作好不好"

### 6.1 一个直观的例子

想象你在看电影。有两种信息很有价值：

1. **这部电影整体好不好**（跟看哪段无关）——这就是状态价值V(s)
2. **在好电影里，哭戏比笑戏更催泪**——这就是优势函数A(s,a)

把它们分开学，然后再组合，效果更好。

### 6.2 Q值的分解

Dueling DQN（Wang et al., 2016）把Q值分解成两部分：

$$
Q(s,a) = V(s) + A(s,a)
$$

- $V(s)$：在状态$s$下，不考虑具体动作，平均能拿多少分
- $A(s,a)$：选了动作$a$比"平均水平"好多少

实际实现时有个小技巧，防止歧义：

$$
Q(s,a) = V(s) + \left(A(s,a) - \frac{1}{|\mathcal{A}|}\sum_{a'} A(s,a')\right)
$$

减去均值后，$V(s)$就能直接读了——因为当$Q = V + (A - mean(A))$且$A = mean(A)$时，$Q = V$。

### 6.3 Dueling网络的结构

```python
class DuelingDQN(nn.Module):
    """Dueling DQN - 分离价值流和优势流"""
    
    def __init__(self, input_shape, n_actions):
        super(DuelingDQN, self).__init__()
        
        # 共享的特征提取层
        self.conv = nn.Sequential(
            nn.Conv2d(input_shape[0], 32, kernel_size=8, stride=4),
            nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=4, stride=2),
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, stride=1),
            nn.ReLU(),
            nn.Flatten()
        )
        
        conv_out = self._get_conv_out(input_shape)
        
        # 价值流 - 输出单个标量：V(s)
        self.value_stream = nn.Sequential(
            nn.Linear(conv_out, 512),
            nn.ReLU(),
            nn.Linear(512, 1)
        )
        
        # 优势流 - 输出每个动作的优势：A(s,a)
        self.advantage_stream = nn.Sequential(
            nn.Linear(conv_out, 512),
            nn.ReLU(),
            nn.Linear(512, n_actions)
        )
    
    def forward(self, x):
        features = self.conv(x)
        
        # 分别计算
        value = self.value_stream(features)           # [batch, 1]
        advantage = self.advantage_stream(features)    # [batch, n_actions]
        
        # Q = V + (A - mean(A))
        q_values = value + (advantage - advantage.mean(dim=1, keepdim=True))
        return q_values
```

### 6.4 为什么Dueling结构有效

Dueling DQN的优势在于**学习效率更高**。

在很多情况下，$V(s)$变化不大（比如游戏里大部分时间在跑路，状态价值比较稳定），而$A(s,a)$只在关键决策点变化大（比如遇到敌人、发现宝藏）。用Dueling结构，网络可以**专门更新变化的部分**。

打个比方：你评价一家餐厅，菜好吃程度（价值流）和服务水平（优势流）是可以独立评估的。你不会因为这次服务员态度差，就把整家餐厅否定掉。

实验表明，Dueling DQN在多个Atari游戏上比标准DQN提升约9%。

---

## 七、优先级经验回放（PER）：不是所有经验都同等重要

### 7.1 随机采样的浪费

标准经验回放是均匀随机采样。但直觉告诉我们：**有些经验比别的经验更重要**。

比如：

- 你刚学到一个"绝杀技"，TD误差很大，这条经验信息量足
- 你已经掌握得很好的一些常规操作，TD误差很小，回放它进步有限

PER（Schaul et al., 2016）的核心思想：**按TD误差的大小分配优先级，误差大的经验被采样的概率更高**。

### 7.2 优先级的数学表达

样本$i$被采样的概率：

$$
P(i) = \frac{p_i^\alpha}{\sum_k p_k^\alpha}
$$

其中$p_i$是优先级权重，通常取TD误差绝对值加一个小常数防止零概率：

$$
p_i = |\delta_i| + \epsilon
$$

$\alpha$控制优先级的激进程度：$\alpha=0$时退化为均匀采样，$\alpha=1$时完全按优先级采样。

### 7.3 重要性采样加权

问题来了：优先采样会改变数据分布，导致梯度估计有偏。解决方案是用**重要性采样权重**（Importance Sampling Weight）修正：

$$
w_i = \left(\frac{1}{P(i)} \cdot \frac{1}{\max_j w_j}\right)^\beta
$$

$\beta$控制补偿程度。训练初期$\beta$接近1（全量补偿），后期慢慢减小到0（不用补偿），因为网络已经足够稳定了。

### 7.4 PER的完整实现

```python
class PrioritizedReplayBuffer:
    """优先级经验回放缓冲区 - 基于SumTree实现"""
    
    def __init__(self, capacity=100000, alpha=0.6, beta=0.4, beta_inc=0.0001):
        self.capacity = capacity
        self.alpha = alpha
        self.beta = beta
        self.beta_inc = beta_inc
        
        # SumTree高效实现优先级查询
        self.tree = SumTree(capacity)
        self.max_priority = 1.0
    
    def push(self, state, action, reward, next_state, done):
        """存入经验，优先级为当前最大值"""
        transition = (state, action, reward, next_state, done)
        self.tree.add(self.max_priority, transition)
    
    def sample(self, batch_size):
        """分层采样 + IS权重"""
        batch = []
        indices = []
        weights = []
        
        for _ in range(batch_size):
            # 从SumTree采样
            priority, index, data = self.tree.sample()
            batch.append(data)
            indices.append(index)
            
            # 计算IS权重
            prob = priority / self.tree.total_priority()
            weight = (len(self.tree) * prob) ** (-self.beta)
            weights.append(weight)
        
        # 归一化权重
        weights = np.array(weights) / max(weights)
        
        states, actions, rewards, next_states, dones = zip(*batch)
        
        # 更新beta
        self.beta = min(1.0, self.beta + self.beta_inc)
        
        return (
            np.array(states), np.array(actions), np.array(rewards),
            np.array(next_states), np.array(dones), 
            indices, weights
        )
    
    def update_priorities(self, indices, td_errors):
        """训练后更新优先级"""
        for idx, error in zip(indices, td_errors):
            priority = abs(error) + 0.01  # 加小常数防止零
            self.tree.update(idx, priority)
            self.max_priority = max(self.max_priority, priority)
```

### 7.5 PER的效果

实验表明，PER能带来约28%的样本效率提升。但也引入了一些复杂度：

- 需要维护SumTree等数据结构
- IS权重会改变梯度，需要相应调整
- $\alpha$和$\beta$需要调参

实践中，如果你的任务样本很贵（采集成本高）、TD误差分布不均，PER是值得尝试的。

---

## 八、NoisyNet：不走寻常路的探索策略

### 8.1 ε-greedy的局限

ε-greedy虽然简单，但有个问题：**它对所有状态一视同仁**。

想象你在迷宫里，某个分支你已经探索过很多次了、确定是死路，按ε-greedy你还是有概率随机走那边。而另一个分支你没去过，却被漏掉了。

更优雅的做法是：**让探索本身适应状态**。

### 8.2 NoisyNet的思路

NoisyNet在网络的权重里加入可学习的噪声参数：

$$
\mathbf{w} = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon}
$$

- $\boldsymbol{\mu}$：均值参数（可学习）
- $\boldsymbol{\sigma}$：标准差参数（可学习）
- $\boldsymbol{\epsilon}$：随机噪声（每次前向传播重新采样）

这样每次选动作时，网络的输出会因噪声而随机变化，实现**隐式探索**——不需要显式的ε参数。

```python
class NoisyLinear(nn.Module):
    """Noisy线性层"""
    
    def __init__(self, in_features, out_features, sigma_init=0.5):
        super().__init__()
        self.in_features = in_features
        self.out_features = out_features
        
        # 可学习的均值和标准差
        self.mu_weight = nn.Parameter(torch.Tensor(out_features, in_features))
        self.sigma_weight = nn.Parameter(torch.Tensor(out_features, in_features))
        self.mu_bias = nn.Parameter(torch.Tensor(out_features))
        self.sigma_bias = nn.Parameter(torch.Tensor(out_features))
        
        self.register_buffer('epsilon_weight', torch.zeros(out_features, in_features))
        self.register_buffer('epsilon_bias', torch.zeros(out_features))
        
        self.reset_parameters(sigma_init)
        self.reset_noise()
    
    def reset_parameters(self, sigma_init):
        mu_range = 1 / math.sqrt(self.in_features)
        self.mu_weight.data.uniform_(-mu_range, mu_range)
        self.mu_bias.data.uniform_(-mu_range, mu_range)
        self.sigma_weight.data.fill_(sigma_init / math.sqrt(self.in_features))
        self.sigma_bias.data.fill_(sigma_init / math.sqrt(self.out_features))
    
    def reset_noise(self):
        """重采样噪声"""
        self.epsilon_weight.data = torch.randn_like(self.epsilon_weight)
        self.epsilon_bias.data = torch.randn_like(self.epsilon_bias)
    
    def forward(self, x):
        if self.training:
            weight = self.mu_weight + self.sigma_weight * self.epsilon_weight
            bias = self.mu_bias + self.sigma_bias * self.epsilon_bias
        else:
            weight = self.mu_weight
            bias = self.mu_bias
        return nn.functional.linear(x, weight, bias)
```

### 8.3 NoisyNet vs ε-greedy

| 特性 | ε-greedy | NoisyNet |
|:-----|:---------|:---------|
| 探索方式 | 显式随机 | 隐式随机 |
| 状态适应性 | 无 | 有 |
| 参数数量 | 1个ε | 与网络参数相同 |
| 收敛速度 | 快但粗糙 | 慢但精细 |

实验表明，NoisyNet在某些任务上能带来8%的提升，尤其适合探索空间复杂、状态依赖性强的任务。

---

## 九、Rainbow：集大成者

### 9.1 为什么组合有效

Rainbow（DeepMind, 2017）把七种技术打包在一起：

1. **Double DQN**：解决过估计
2. **Dueling DQN**：高效分解价值
3. **Prioritized Replay**：智能采样
4. **NoisyNet**：自适应探索
5. **Distributional RL**：分布化价值估计
6. **N-step Returns**：多步TD
7. **Experience Replay**：基础数据管理

这些技术并非简单叠加，而是**互补**。比如PER需要Double DQN来校正优先级带来的偏差；Dueling结构让N-step Returns的方差更可控。

### 9.2 Rainbow vs 单项技术

在Atari 57个游戏上的实验结果：

- Rainbow的中位得分是标准DQN的**200%+**
- 每个单独的技术都有提升，但组合后效果远超任何单项
- 最强的单项是Distributional RL（+33%），但配合其他技术后整体更强

### 9.3 实战建议：不用全上

Rainbow很强大，但实操中建议：

- **初学者**：从标准DQN开始，理解核心机制
- **进阶**：加Double DQN（几乎必加）
- **性能敏感**：加Dueling或PER
- **复杂探索任务**：考虑NoisyNet
- **追求极限**：Rainbow，但调参复杂度暴增

---

## 十、调参指南：经验大于理论

### 10.1 网络结构

**对于图像输入（Atari风格）**：

```python
self.conv = nn.Sequential(
    nn.Conv2d(4, 32, 8, stride=4),   # 输入4帧84x84灰度图
    nn.ReLU(),
    nn.Conv2d(32, 64, 4, stride=2),
    nn.ReLU(),
    nn.Conv2d(64, 64, 3, stride=1),
    nn.ReLU(),
    nn.Flatten(),
    nn.Linear(3136, 512),  # 3136 = 64 * 7 * 7
    nn.ReLU(),
    nn.Linear(512, n_actions)
)
```

**对于低维状态输入**：

```python
self.fc = nn.Sequential(
    nn.Linear(input_dim, 128),
    nn.ReLU(),
    nn.Linear(128, 128),
    nn.ReLU(),
    nn.Linear(128, n_actions)
)
```

### 10.2 关键超参数

| 参数 | 推荐值 | 说明 |
|:-----|:------|:-----|
| 学习率 | 0.00025 | Adam优化器，DQN论文原值 |
| 折扣因子γ | 0.99 | 高值利于长期回报 |
| 批量大小 | 32或64 | 显存允许可增大 |
| 目标网络更新间隔 | 10000步 | 硬更新 |
| 软更新τ | 0.005 | 如用软更新 |
| ε初始值 | 1.0 | 完全随机开始 |
| ε最小值 | 0.01或0.1 | 留少量探索 |
| ε衰减帧数 | 1M | 线性衰减 |
| 经验回放大小 | 1M | Atari标准配置 |
| 梯度裁剪 | 10.0 | Huber loss时用1.0 |
| 预热帧数 | 50000 | 先填满缓冲区 |

### 10.3 训练不稳定时的排查清单

| 症状 | 可能原因 | 解决方案 |
|:-----|:---------|:---------|
| Q值NaN | 梯度爆炸、奖励无界 | 梯度裁剪到1.0、奖励归一化 |
| 性能突然退化 | 目标网络更新过快 | 增大更新间隔 |
| 训练停滞 | ε衰减太快、探索不足 | 延长衰减、增大ε_min |
| 收敛太慢 | 学习率不合适 | 试试0.0001或0.0005 |
| 显存爆炸 | batch过大、缓冲区太大 | 减小batch，限制replay size |

### 10.4 奖励归一化

奖励尺度不一致会导致训练困难。常用方法：

```python
def normalize_reward(reward, mean, std):
    return (reward - mean) / (std + 1e-8)
```

或者用更鲁棒的方法——对奖励做clip：

```python
reward = np.clip(reward, -1, 1)  # Atari论文的做法
```

---

## 十一、实战：用DQN玩CartPole

### 11.1 任务简介

CartPole是OpenAI Gym的经典环境：

- 小车上放着一根杆子，要保持杆子不倒
- 状态：位置、速度、角度、角速度（4维连续）
- 动作：向左或向右（2离散）
- 目标：坚持200步不倒

这比Atari简单得多，适合入门练手。

### 11.2 完整代码

```python
"""
DQN玩CartPole - 完整可运行代码
运行方式：pip install torch gym numpy
         python dqn_cartpole.py
"""

import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
import gym
from collections import deque
import random
import matplotlib.pyplot as plt

# ============ 1. 网络定义 ============

class DQN(nn.Module):
    """简单的全连接网络"""
    def __init__(self, state_dim, action_dim):
        super(DQN, self).__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, action_dim)
        )
    
    def forward(self, x):
        return self.net(x)

# ============ 2. 经验回放缓冲区 ============

class ReplayBuffer:
    def __init__(self, capacity=10000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (
            np.array(states, dtype=np.float32),
            np.array(actions, dtype=np.int64),
            np.array(rewards, dtype=np.float32),
            np.array(next_states, dtype=np.float32),
            np.array(dones, dtype=np.float32)
        )
    
    def __len__(self):
        return len(self.buffer)

# ============ 3. DQN智能体 ============

class DQNAgent:
    def __init__(
        self, 
        state_dim, 
        action_dim,
        learning_rate=0.001,
        gamma=0.99,
        epsilon=1.0,
        epsilon_min=0.01,
        epsilon_decay=0.995,
        batch_size=64,
        replay_capacity=10000,
        target_update_freq=10
    ):
        self.state_dim = state_dim
        self.action_dim = action_dim
        self.gamma = gamma
        self.epsilon = epsilon
        self.epsilon_min = epsilon_min
        self.epsilon_decay = epsilon_decay
        self.batch_size = batch_size
        self.target_update_freq = target_update_freq
        self.train_step = 0
        
        # 策略网络和目标网络
        self.policy_net = DQN(state_dim, action_dim)
        self.target_net = DQN(state_dim, action_dim)
        self.target_net.load_state_dict(self.policy_net.state_dict())
        self.target_net.eval()
        
        self.optimizer = optim.Adam(self.policy_net.parameters(), lr=learning_rate)
        self.memory = ReplayBuffer(replay_capacity)
        
        # 用Huber loss比MSE更鲁棒
        self.loss_fn = nn.SmoothL1Loss()
    
    def select_action(self, state, training=True):
        """ε-greedy动作选择"""
        if training and random.random() < self.epsilon:
            return random.randrange(self.action_dim)
        
        with torch.no_grad():
            state_tensor = torch.FloatTensor(state).unsqueeze(0)
            q_values = self.policy_net(state_tensor)
            return q_values.argmax().item()
    
    def store(self, state, action, reward, next_state, done):
        self.memory.push(state, action, reward, next_state, done)
    
    def update(self):
        """训练一步"""
        if len(self.memory) < self.batch_size:
            return None
        
        # 采样
        states, actions, rewards, next_states, dones = self.memory.sample(self.batch_size)
        
        # 转为张量
        states = torch.FloatTensor(states)
        actions = torch.LongTensor(actions)
        rewards = torch.FloatTensor(rewards)
        next_states = torch.FloatTensor(next_states)
        dones = torch.FloatTensor(dones)
        
        # 当前Q值
        current_q = self.policy_net(states).gather(1, actions.unsqueeze(1)).squeeze(1)
        
        # 目标Q值（Double DQN）
        with torch.no_grad():
            next_actions = self.policy_net(next_states).argmax(1)
            next_q = self.target_net(next_states).gather(1, next_actions.unsqueeze(1)).squeeze(1)
            target_q = rewards + self.gamma * next_q * (1 - dones)
        
        # 损失和反向传播
        loss = self.loss_fn(current_q, target_q)
        self.optimizer.zero_grad()
        loss.backward()
        
        # 梯度裁剪
        torch.nn.utils.clip_grad_norm_(self.policy_net.parameters(), 100)
        
        self.optimizer.step()
        self.train_step += 1
        
        # 更新目标网络
        if self.train_step % self.target_update_freq == 0:
            self.target_net.load_state_dict(self.policy_net.state_dict())
        
        # ε衰减
        self.epsilon = max(self.epsilon_min, self.epsilon * self.epsilon_decay)
        
        return loss.item()
    
    def save(self, path='dqn_cartpole.pth'):
        torch.save(self.policy_net.state_dict(), path)
        print(f"模型已保存到 {path}")
    
    def load(self, path='dqn_cartpole.pth'):
        self.policy_net.load_state_dict(torch.load(path))
        self.target_net.load_state_dict(torch.load(path))
        print(f"模型已加载")

# ============ 4. 训练主循环 ============

def train(env_name='CartPole-v1', num_episodes=500, render=False):
    """训练DQN智能体"""
    env = gym.make(env_name)
    
    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.n
    
    agent = DQNAgent(
        state_dim=state_dim,
        action_dim=action_dim,
        learning_rate=0.001,
        gamma=0.99,
        batch_size=64,
        target_update_freq=10
    )
    
    episode_rewards = []
    losses = []
    
    for episode in range(num_episodes):
        state, _ = env.reset()
        total_reward = 0
        total_loss = 0
        loss_count = 0
        
        while True:
            if render:
                env.render()
            
            # 选择动作
            action = agent.select_action(state)
            
            # 执行
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            # 存储
            agent.store(state, action, reward, next_state, done)
            
            # 更新
            loss = agent.update()
            if loss is not None:
                total_loss += loss
                loss_count += 1
            
            state = next_state
            total_reward += reward
            
            if done:
                break
        
        episode_rewards.append(total_reward)
        avg_loss = total_loss / loss_count if loss_count > 0 else 0
        losses.append(avg_loss)
        
        # 打印进度
        if (episode + 1) % 10 == 0:
            avg_reward = np.mean(episode_rewards[-10:])
            print(f"Episode {episode+1}/{num_episodes} | "
                  f"奖励: {total_reward:.1f} | "
                  f"10局平均: {avg_reward:.1f} | "
                  f"ε: {agent.epsilon:.3f} | "
                  f"Loss: {avg_loss:.4f}")
        
        # 检查是否 solved
        if np.mean(episode_rewards[-100:]) >= 195:
            print(f"\n训练完成！最近100局平均奖励达到 {np.mean(episode_rewards[-100:]):.1f}")
            break
    
    env.close()
    
    # 绘制训练曲线
    plt.figure(figsize=(12, 4))
    
    plt.subplot(1, 2, 1)
    plt.plot(episode_rewards)
    plt.xlabel('Episode')
    plt.ylabel('Reward')
    plt.title('Training Rewards')
    plt.axhline(y=195, color='r', linestyle='--', label='Solved threshold')
    plt.legend()
    
    plt.subplot(1, 2, 2)
    plt.plot(losses)
    plt.xlabel('Episode')
    plt.ylabel('Loss')
    plt.title('Training Loss')
    
    plt.tight_layout()
    plt.savefig('training_curve.png', dpi=150)
    plt.show()
    
    agent.save()
    return agent, episode_rewards

# ============ 5. 评估与可视化 ============

def evaluate(agent, env_name='CartPole-v1', num_episodes=10, render=True):
    """评估训练好的智能体"""
    env = gym.make(env_name)
    
    total_rewards = []
    
    for episode in range(num_episodes):
        state, _ = env.reset()
        total_reward = 0
        
        while True:
            if render:
                env.render()
            
            action = agent.select_action(state, training=False)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            state = next_state
            total_reward += reward
            
            if done:
                break
        
        total_rewards.append(total_reward)
        print(f"Episode {episode+1}: {total_reward}")
    
    env.close()
    
    avg_reward = np.mean(total_rewards)
    print(f"\n平均奖励: {avg_reward:.1f}")
    return avg_reward

if __name__ == '__main__':
    # 训练
    agent, rewards = train(num_episodes=300)
    
    # 评估
    print("\n========== 评估 ==========")
    evaluate(agent, num_episodes=5, render=True)
```

### 11.3 预期结果

运行后，你应该看到：

- **Episode 50左右**：奖励开始上升，从10左右增长到50+
- **Episode 100左右**：能达到100+步
- **Episode 200左右**：基本能稳定在195（solved threshold）以上
- **Loss曲线**：前期波动大，后期逐渐收敛

如果一直不收敛，检查：学习率是否太大/太小、ε是否衰减太快、gamma是否合适（CartPole用0.99没问题）。

---

## 十二、DQN实战：用DQN玩Atari游戏（进阶）

### 12.1 Atari环境准备

Atari游戏比CartPole复杂得多，需要：

1. **帧堆叠**：单帧信息不足，把连续4帧堆在一起作为状态
2. **图像预处理**：RGB转灰度、缩放到84x84
3. **跳帧（Frame Skip）**：每隔几帧才执行动作，减少计算

```python
import gym
import numpy as np
from gym.wrappers import FrameStack, ResizeObservation, GrayScaleObservation

def make_atari_env(env_name):
    """构建Atari预处理环境"""
    env = gym.make(env_name)
    
    # 灰度化 + 缩放
    env = GrayScaleObservation(env)
    env = ResizeObservation(env, (84, 84))
    
    # 帧堆叠（4帧）
    env = FrameStack(env, num_stack=4)
    
    return env
```

### 12.2 Atari版DQN网络

Atari用卷积网络处理图像：

```python
class AtariDQN(nn.Module):
    """Atari游戏专用DQN网络"""
    def __init__(self, input_shape=(4, 84, 84), n_actions=4):
        super(AtariDQN, self).__init__()
        
        self.conv = nn.Sequential(
            # 输入: 4 x 84 x 84
            nn.Conv2d(4, 32, kernel_size=8, stride=4),
            nn.ReLU(),
            # 32 x 20 x 20
            nn.Conv2d(32, 64, kernel_size=4, stride=2),
            nn.ReLU(),
            # 64 x 9 x 9
            nn.Conv2d(64, 64, kernel_size=3, stride=1),
            nn.ReLU(),
            # 64 x 7 x 7
            nn.Flatten(),
            # 3136
            nn.Linear(64 * 7 * 7, 512),
            nn.ReLU(),
            nn.Linear(512, n_actions)
        )
    
    def forward(self, x):
        return self.conv(x)
```

### 12.3 完整训练脚本

```python
"""
DQN玩Atari Breakout - 完整训练脚本
需要：pip install torch gymnasium[accept-rom-license] ale-py
"""

import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
import gymnasium as gym
from collections import deque
import random
import time
from gymnasium.wrappers import FrameStack, ResizeObservation, GrayScaleObservation

# ============ 环境预处理 ============

def make_env(env_name):
    env = gym.make(env_name, render_mode='rgb_array')
    env = GrayScaleObservation(env)
    env = ResizeObservation(env, (84, 84))
    env = FrameStack(env, num_stack=4)
    return env

# ============ 网络定义 ============

class AtariDQN(nn.Module):
    def __init__(self, n_actions):
        super().__init__()
        
        self.conv = nn.Sequential(
            nn.Conv2d(4, 32, 8, stride=4),
            nn.ReLU(),
            nn.Conv2d(32, 64, 4, stride=2),
            nn.ReLU(),
            nn.Conv2d(64, 64, 3, stride=1),
            nn.ReLU(),
            nn.Flatten(),
        )
        
        self.fc = nn.Sequential(
            nn.Linear(64 * 7 * 7, 512),
            nn.ReLU(),
            nn.Linear(512, n_actions)
        )
    
    def forward(self, x):
        x = x / 255.0  # 归一化
        conv_out = self.conv(x)
        return self.fc(conv_out)

# ============ 经验回放 ============

class ReplayBuffer:
    def __init__(self, capacity=1000000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, *args):
        self.buffer.append(args)
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        return zip(*batch)
    
    def __len__(self):
        return len(self.buffer)

# ============ 训练 ============

def train_atari(
    env_name='BreakoutNoFrameskip-v4',
    num_frames=50000000,
    batch_size=32,
    learning_rate=0.00025,
    gamma=0.99,
    target_update_freq=10000,
    eval_freq=250000
):
    env = make_env(env_name)
    
    n_actions = env.action_space.n
    agent_net = AtariDQN(n_actions)
    target_net = AtariDQN(n_actions)
    target_net.load_state_dict(agent_net.state_dict())
    
    optimizer = optim.Adam(agent_net.parameters(), lr=learning_rate)
    replay_buffer = ReplayBuffer()
    
    state, _ = env.reset()
    episode_reward = 0
    episode_count = 0
    episode_rewards = []
    
    start_time = time.time()
    
    for frame in range(num_frames):
        # ε-greedy（线性衰减到10万帧）
        epsilon = max(0.1, 1.0 - frame / 100000)
        
        # 选择动作
        if random.random() < epsilon:
            action = env.action_space.sample()
        else:
            with torch.no_grad():
                state_tensor = torch.from_numpy(state).float().unsqueeze(0)
                action = agent_net(state_tensor).argmax().item()
        
        # 执行
        next_state, reward, terminated, truncated, _ = env.step(action)
        done = terminated or truncated
        
        # 存储（奖励clip）
        replay_buffer.push(
            state, action, np.clip(reward, -1, 1), 
            next_state, float(terminated)
        )
        state = next_state
        episode_reward += reward
        
        if done:
            episode_rewards.append(episode_reward)
            episode_reward = 0
            state, _ = env.reset()
            episode_count += 1
        
        # 训练
        if len(replay_buffer) > 50000:
            # 采样
            states, actions, rewards, next_states, dones = replay_buffer.sample(batch_size)
            
            states = torch.from_numpy(np.array(states)).float()
            actions = torch.LongTensor(actions)
            rewards = torch.FloatTensor(rewards)
            next_states = torch.from_numpy(np.array(next_states)).float()
            dones = torch.FloatTensor(dones)
            
            # Double DQN
            with torch.no_grad():
                next_actions = agent_net(next_states).argmax(1)
                next_q = target_net(next_states).gather(1, next_actions.unsqueeze(1)).squeeze(1)
                target_q = rewards + gamma * next_q * (1 - dones)
            
            current_q = agent_net(states).gather(1, actions.unsqueeze(1)).squeeze(1)
            loss = nn.SmoothL1Loss()(current_q, target_q)
            
            optimizer.zero_grad()
            loss.backward()
            torch.nn.utils.clip_grad_norm_(agent_net.parameters(), 10)
            optimizer.step()
            
            # 更新目标网络
            if frame % target_update_freq == 0:
                target_net.load_state_dict(agent_net.state_dict())
        
        # 日志
        if frame % 10000 == 0:
            fps = frame / (time.time() - start_time)
            avg_reward = np.mean(episode_rewards[-10:]) if episode_rewards else 0
            print(f"Frame {frame:,} | FPS: {fps:.0f} | "
                  f"Episodes: {episode_count} | "
                  f"Avg Reward: {avg_reward:.1f} | "
                  f"ε: {epsilon:.3f}")
        
        # 保存
        if frame % 500000 == 0 and frame > 0:
            torch.save(agent_net.state_dict(), f'checkpoint_{frame}.pth')
    
    env.close()
    return agent_net

if __name__ == '__main__':
    train_atari()
```

### 12.4 训练技巧

Atari游戏训练50M帧需要很长时间（GPU上可能几十小时），以下是一些加速技巧：

1. **使用多个环境并行**：用`SubprocVecEnv`并行跑多个游戏实例，采集速度翻倍
2. **梯度累积**：如果显存不够，把batch分成多个小batch累积梯度再更新
3. **混合精度训练**：用`torch.cuda.amp`加速
4. **断点续训**：保存checkpoint，支持从某个frame继续训练

---

## 十三、DQN的局限与后续发展

### 13.1 DQN不是银弹

DQN有几个固有限制：

1. **离散动作空间**：DQN只能输出有限个动作的Q值，连续动作没法用（虽然可以用连续动作空间的Q学习变体）
2. **过估计问题**：即使加了Double DQN，也只是缓解，不能完全消除
3. **训练不稳定**：需要很多技巧（目标网络、梯度裁剪等）才能稳住
4. **样本效率低**：虽然比纯在线学习好，但比PPO等方法还是慢

### 13.2 后继者们

- **Actor-Critic类方法**（PPO、SAC）：直接优化策略，更适合连续动作
- **Rainbow**：在DQN基础上堆技术，性能更强但更复杂
- **Q-Transformer**：用Transformer处理时序，样本效率更高
- **DreamerV3**：世界模型 + 强化学习，纯离线训练

如果你的任务能用DQN解决（比如离散动作空间的游戏），DQN是好选择。如果需要连续控制或更高样本效率，考虑PPO/SAC。

---

## 十四、数学形式化总结

### DQN损失函数

$$
\boxed{L(\theta) = \mathbb{E}_{(s,a,r,s')\sim \mathcal{D}} \left[ \left( y - Q(s,a|\theta) \right)^2 \right]}
$$

其中目标 $y = r + \gamma \max_{a'} Q_{target}(s',a'|\theta^-)$

### Double DQN目标

$$
\boxed{y_j = r_j + \gamma Q_{target}\left(s'_j, \arg\max_{a'} Q_{policy}(s'_j, a')\right)}
$$

### Dueling Q值分解

$$
\boxed{Q(s,a) = V(s) + A(s,a) - \frac{1}{|\mathcal{A}|}\sum_{a'} A(s,a')}
$$

### PER采样概率

$$
\boxed{P(i) = \frac{p_i^\alpha}{\sum_k p_k^\alpha}}, \quad p_i = |\delta_i| + \epsilon
$$

### NoisyNet权重

$$
\boxed{\mathbf{w} = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon}}
$$

---

## 十五、相关文档

- [[MDP与Bellman方程详解|MDP与Bellman方程]] — 理论基础
- [[Q学习深度指南|Q学习深度指南]] — 表格型Q学习
- [[策略梯度方法详解|策略梯度方法]] — 直接策略优化
- [[PPO深度指南|PPO算法]] — 现代策略优化
- [[多智能体RL详解|多智能体RL]] — 多智能体扩展
- [[RL应用场景|RL应用场景]] — 实际应用案例

---

## 十六、参考文献

1. Mnih, V., et al. (2015). Human-level control through deep reinforcement learning. *Nature*, 518(7540), 529-533.
2. Mnih, V., et al. (2013). Playing Atari with deep reinforcement learning. *arXiv:1312.5602*.
3. Van Hasselt, H., Guez, A., & Silver, D. (2016). Deep reinforcement learning with double Q-learning. *AAAI*, 30(1).
4. Wang, Z., et al. (2016). Dueling network architectures for deep reinforcement learning. *ICML*, 1995-2003.
5. Schaul, T., et al. (2016). Prioritized experience replay. *ICLR*.
6. Hessel, M., et al. (2018). Rainbow: Combining improvements in deep reinforcement learning. *AAAI*, 2018.
7. Fortunato, M., et al. (2018). Noisy networks for exploration. *ICLR*.

---

*DQN开创了深度强化学习的新时代，后续的许多算法都建立在其核心思想之上。希望这篇指南能帮助你从理论到实践全面掌握DQN。*
