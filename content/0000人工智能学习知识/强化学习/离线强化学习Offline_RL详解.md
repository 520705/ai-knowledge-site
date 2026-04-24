---
title: 离线强化学习Offline RL详解
date: 2026-04-24
tags:
  - 强化学习
  - Offline RL
  - CQL
  - IQL
  - 离线策略学习
  - D4RL
aliases:
  - 离线强化学习完全指南
  - Offline RL算法详解
---

# 离线强化学习Offline RL详解

## 在线RL vs离线RL：为什么离线RL更难

先说个故事帮助理解。假设你想学游泳，有两种方式：

**在线学习**就像你直接跳进游泳池，扑腾几下呛几口水，然后从呛水的经验里学——每呛一口你就知道"这动作不对"，调整姿势再来。这种方式学得慢但真实。

**离线学习**就像你坐在岸上看别人游泳的视频录像，你只能从这些录像里学。你没法真的跳下去试试，看完录像你得自己琢磨："这个人手臂划水的角度看起来很舒服，我下次试试这个角度"。问题是你光看不动手，你根本不知道那个"舒服"到底是多舒服，你也可能在完全错误的理解下练习了100次。

这就是离线RL的核心困境：**数据是死的，环境是活的，智能体必须在静态数据中学会动态决策**。

用更技术的话说：

| 特性 | 在线RL | 离线RL |
|------|--------|--------|
| 数据收集 | 边学边采 | 提前采集好 |
| 探索策略 | 主动探索 | 无探索 |
| 分布偏移 | 轻微可控 | 严重 |
| Q值估计 | 可以bootstrapping | bootstrap会灾难性崩塌 |
| 核心挑战 | 方差大 | 分布偏移 |

在线RL里，智能体在当前策略和环境交互，如果Q值估计高了，下一步采样的数据会"纠正"这个高估——因为真实奖励会告诉智能体真实价值。但离线RL里，你用那些"别的策略"收集的数据来训练，你的高估得不到纠正，Q值会越堆越高，最后整个策略崩溃。

这个现象有个专门的名字：**分布偏移（Distributional Shift）**。

## 分布偏移：离线RL的核心挑战

分布偏移说的是：训练时用的数据分布，和智能体实际部署时遇到的分布，不一样。

具体来说，离线RL的数据集$D$是由某种行为策略$\beta(a|s)$生成的。但我们训练的目的是学到一个新策略$\pi(a|s)$，这个$\pi$和数据里的$\beta$大概率不一样。

问题出在Q值更新上。DQN类的更新公式：

$$Q(s,a) \leftarrow Q(s,a) + \alpha(r + \gamma \max_{a'} Q(s',a') - Q(s,a))$$

注意那个$\max_{a'} Q(s',a')$——这是"拔尖"操作，选择Q值最大的动作。在线学习中，下一步探索时你真的会去试那个动作，看看它是不是真的那么好。但在离线学习中，你永远用那些数据里的$(s', a')$对来更新，如果某个$a'$没在数据里出现过，它的Q值估计就可能偏离真实值很远。

想象一下：你的数据集里从来没有在状态$s'$下选择过动作"右转"，所以$Q(s', \text{右转})$在数据驱动下根本没被好好训练过，值可能是0或者负数。但实际上"右转"才是最优动作！

**三种主要的分布偏移问题**：

1. **状态偏移（State Distribution Shift）**：智能体的策略导致它访问的状态，在原始数据集里很少见
2. **动作偏移（Action Distribution Shift）**：智能体选择的动作，行为策略很少选
3. **复合偏移（Compounding Shift）**：上面两个问题叠加，越往后越偏移

## Offline RL方法分类

离线RL的研究这几年火得不行，方法多如牛毛。但剥开外表看内核，主要就两派：**约束类**和**正则化类**。

### 约束类方法

核心思想：**给策略划定边界，限制它不能跑太远**。既然数据是$\beta$生成的，那就要求学到的策略$\pi$和$\beta$别差太远。

代表方法：
- BCQ（Batch Constrained Q-learning）：只选择数据集中"接近"的动作
- CQL（Conservative Q-Learning）：给不在数据集中的动作惩罚
- RAMBO：对抗性正则化

### 正则化类方法

核心思想：**在损失函数里加一项，推动策略往数据分布的方向靠拢**。

代表方法：
- BRAC（Batch Regularized Actor-Critic）
- BEAR（Bootstrapped Error Accumulation Reduction）
- AWAC（Advantage Weighted Actor-Critic）

说白了，约束类是"硬限制"，正则化类是"软惩罚"。实际效果差不多，看具体问题哪个更好使。

## CQL：为什么要保守估计Q值

CQL是现在离线RL最火的方法之一，来自伯克利2020年的论文。它的核心洞察很有意思：

> **离线RL学到的Q值往往被高估，而高估会导致灾难性的决策失败。**

怎么解决这个问题？CQL说：那我就故意把Q值压低一点，越保守越好。

具体做法是在标准的SAC/Soft Actor-Critic损失上加一项：

$$\min_q \max_\pi \mathbb{E}_{s\sim D}[ \underbrace{D_{KL}(\pi(\cdot|s) \| \frac{\exp Q(s,\cdot)}{Z(s)})}_{\text{SAC策略损失}}] + \underbrace{\tau \cdot \mathbb{E}_{s,a\sim D}[\log \sum_{a'} \exp Q(s,a') - Q(s,a)]}_{\text{CQL正则项}}$$

最后那一项就是CQL的核心创新。它在做什么？

- $\log \sum_{a'} \exp Q(s,a')$：对所有动作的Q值做对数和（相当于softmax的log），这会拉高所有动作的Q值
- $-Q(s,a)$：然后减去当前数据中实际采取的动作的Q值

结果就是：**数据集中出现过的动作，Q值被相对压低了；没出现过的动作，Q值相对被拉高了**。但整个过程中所有Q值都被压低了一些，所以叫"保守"。

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CQL:
    def __init__(self, state_dim, action_dim, hidden=256, lr=3e-4, 
                 min_q_weight=1.0, temp=1.0):
        self.min_q_weight = min_q_weight
        self.temp = temp
        
        # 三个Q网络（CQL通常用更多Q网络来稳定）
        self.Q1 = QNetwork(state_dim, action_dim, hidden)
        self.Q2 = QNetwork(state_dim, action_dim, hidden)
        self.Q_target1 = QNetwork(state_dim, action_dim, hidden)
        self.Q_target2 = QNetwork(state_dim, action_dim, hidden)
        
        self.actor = Actor(state_dim, action_dim, hidden)
        
        # 同步目标网络
        self.hard_update()
        
        self.optim_Q = optim.Adam(
            list(self.Q1.parameters()) + list(self.Q2.parameters()), 
            lr=lr
        )
        self.optim_actor = optim.Adam(self.actor.parameters(), lr=lr)
    
    def update(self, batch):
        states, actions, rewards, next_states, dones = batch
        
        # 1. 标准SAC更新，计算当前Q值
        with torch.no_grad():
            next_actions = self.actor(next_states)
            next_q1 = self.Q_target1(next_states, next_actions)
            next_q2 = self.Q_target2(next_states, next_actions)
            next_q = torch.min(next_q1, next_q2)
            target_q = rewards + (1 - dones) * 0.99 * next_q
        
        # 2. Critic损失（标准MSE + CQL正则项）
        current_q1 = self.Q1(states, actions)
        current_q2 = self.Q2(states, actions)
        
        critic_loss = F.mse_loss(current_q1, target_q) + \
                      F.mse_loss(current_q2, target_q)
        
        # 3. CQL正则项（关键！）
        # 在所有动作上采样计算
        num_samples = 10
        random_actions = torch.rand(states.shape[0], num_samples, actions.shape[-1])
        random_actions = (random_actions * 2 - 1) * 2  # 缩放到[-2, 2]
        
        # 用当前策略采样的动作
        policy_actions, log_pi = self.actor.get_action(states, with_log_prob=True)
        
        # 在状态空间采样计算
        q1_random = self.Q1(states, random_actions)  # [batch, num_samples]
        q2_random = self.Q2(states, random_actions)
        q1_policy = self.Q1(states, policy_actions)
        q2_policy = self.Q2(states, policy_actions)
        
        # CQL(H) - 在所有动作上取log-sum-exp
        q1_logsumexp = torch.logsumexp(q1_random, dim=1)  # [batch]
        q2_logsumexp = torch.logsumexp(q2_random, dim=1)
        
        # 减去数据中动作的Q值
        cql_loss1 = q1_logsumexp.mean() - current_q1.mean()
        cql_loss2 = q2_logsumexp.mean() - current_q2.mean()
        cql_loss = (cql_loss1 + cql_loss2) * self.min_q_weight
        
        total_critic_loss = critic_loss + cql_loss
        
        # 4. 更新Q网络
        self.optim_Q.zero_grad()
        total_critic_loss.backward()
        self.optim_Q.step()
        
        # 5. 更新策略
        new_actions, log_pi = self.actor.get_action(states, with_log_prob=True)
        q1_new = self.Q1(states, new_actions)
        q2_new = self.Q2(states, new_actions)
        q_new = torch.min(q1_new, q2_new)
        
        actor_loss = -q_new.mean() + self.temp * log_pi.mean()
        
        self.optim_actor.zero_grad()
        actor_loss.backward()
        self.optim_actor.step()
        
        return critic_loss.item(), cql_loss.item(), actor_loss.item()
    
    def hard_update(self):
        self.Q_target1.load_state_dict(self.Q1.state_dict())
        self.Q_target2.load_state_dict(self.Q2.state_dict())
    
    def soft_update(self, tau=0.005):
        for target, source in zip(
            self.Q_target1.parameters(), self.Q1.parameters()
        ):
            target.data.copy_(tau * source.data + (1 - tau) * target.data)
        for target, source in zip(
            self.Q_target2.parameters(), self.Q2.parameters()
        ):
            target.data.copy_(tau * source.data + (1 - tau) * target.data)
```

CQL的效果总结：**它让Q函数变得更保守，策略不会轻易相信"没见过的动作是好的"。** 这在离线场景下至关重要。

## TD3+BC：大道至简

TD3+BC是TD3加上BC（Behavior Cloning）正则项的组合，出自2021年的一篇短论文。这篇文章很有意思，因为它的核心发现是：**大多数离线RL方法的复杂技巧可能是不必要的**。

TD3+BC的做法极其简单：

```python
class TD3BC:
    def __init__(self, state_dim, action_dim, hidden=256, lr=3e-4, 
                 policy_noise=0.2, noise_clip=0.5, policy_freq=2,
                 alpha=2.5):  # BC正则系数
        self.Q1 = QNetwork(state_dim, action_dim, hidden)
        self.Q2 = QNetwork(state_dim, action_dim, hidden)
        self.Q_target1 = QNetwork(state_dim, action_dim, hidden)
        self.Q_target2 = QNetwork(state_dim, action_dim, hidden)
        self.actor = Actor(state_dim, action_dim, hidden)
        
        self.policy_noise = policy_noise
        self.noise_clip = noise_clip
        self.policy_freq = policy_freq
        self.alpha = alpha
        self.training_iter = 0
        
        self.hard_update()
    
    def update(self, batch):
        states, actions, rewards, next_states, dones = batch
        self.training_iter += 1
        
        # 标准TD3更新
        with torch.no_grad():
            noise = (torch.randn_like(actions) * self.policy_noise).clamp(
                -self.noise_clip, self.noise_clip
            )
            next_actions = (self.actor(next_states) + noise).clamp(-1, 1)
            
            target_q = rewards + (1 - dones) * 0.99 * \
                torch.min(self.Q_target1(next_states, next_actions),
                         self.Q_target2(next_states, next_actions))
        
        current_q1 = self.Q1(states, actions)
        current_q2 = self.Q2(states, actions)
        critic_loss = F.mse_loss(current_q1, target_q) + \
                      F.mse_loss(current_q2, target_q)
        
        self.optim_Q.zero_grad()
        critic_loss.backward()
        self.optim_Q.step()
        
        # 延迟策略更新
        if self.training_iter % self.policy_freq == 0:
            policy_actions = self.actor(states)
            
            # Q值损失（最大化）
            q_value = self.Q1(states, policy_actions).mean()
            
            # BC正则项：鼓励策略靠近数据集中的动作
            bc_loss = ((policy_actions - actions) ** 2).mean()
            
            # 总损失：Q值最大化 + BC正则
            actor_loss = -q_value + self.alpha * bc_loss
            
            self.optim_actor.zero_grad()
            actor_loss.backward()
            self.optim_actor.step()
            
            self.soft_update()
            
            return critic_loss.item(), actor_loss.item()
        
        return critic_loss.item(), 0.0
```

TD3+BC的直觉是：**离线RL的主要问题不是算法不够 fancy，而是策略太容易跑偏。** 加一个简单的BC正则项，把策略"拽"回数据分布附近，就能解决大部分问题。

实验结果也印证了这一点——TD3+BC在很多任务上能和更复杂的CQL、IQL打得有来有回。

## IQL：从四分位数学习

IQL（Implicit Q-Learning）是2021年NeurIPS的论文，提出了一个很聪明的思路：**不直接估计Q值，而是估计期望回报的分位数**。

传统的Q-Learning要估计的是$Q(s,a) = \mathbb{E}[G]$，这很难。IQL的想法是：我不知道期望是多少，但我可以估计不同百分位的值，然后从中"推导"出期望。

具体来说，IQL学习一个函数$V(s)$表示"在状态$s$下，按照某种策略能获得的回报"。然后用Expectile Regression来估计：

```python
class IQL:
    def __init__(self, state_dim, action_dim, hidden=256, 
                 expectile=0.7, beta=3.0, lr=3e-4):
        self.expectile = expectile
        self.beta = beta
        
        self.V = ValueNetwork(state_dim, hidden)
        self.Q1 = QNetwork(state_dim, action_dim, hidden)
        self.Q2 = QNetwork(state_dim, action_dim, hidden)
        self.actor = Actor(state_dim, action_dim, hidden)
        
        self.optim = optim.Adam(
            list(self.V.parameters()) + 
            list(self.Q1.parameters()) + 
            list(self.Q2.parameters()),
            lr=lr
        )
        self.optim_actor = optim.Adam(self.actor.parameters(), lr=lr)
    
    def expectile_loss(self, diff, expectile=0.7):
        """
        期望损失：不对称MSE
        高于0的差异用expectile权重，低于0的用1-expectile权重
        """
        weight = torch.where(diff > 0, expectile, 1 - expectile)
        return weight * (diff ** 2)
    
    def update(self, batch):
        states, actions, rewards, next_states, dones = batch
        
        # 1. 计算target V
        with torch.no_grad():
            next_actions = self.actor(next_states)
            next_q = torch.min(
                self.Q1(next_states, next_actions),
                self.Q2(next_states, next_actions)
            )
            target_v = next_q
        
        # 2. 更新V：期望损失
        v_current = self.V(states)
        v_loss = self.expectile_loss(target_v - v_current, self.expectile).mean()
        
        # 3. 更新Q：标准MSE
        with torch.no_grad():
            q_target = rewards + (1 - dones) * 0.99 * v_current.detach()
        
        q1_loss = F.mse_loss(self.Q1(states, actions), q_target)
        q2_loss = F.mse_loss(self.Q2(states, actions), q_target)
        
        # 4. 计算优势，提取高回报的动作
        with torch.no_grad():
            v = self.V(states)
            q = torch.min(self.Q1(states, actions), self.Q2(states, actions))
            advantage = q - v
            
            # 只更新优势为正的动作
            exp_adv = torch.exp(advantage / self.beta)
            exp_adv = torch.clamp(exp_adv, 0, 100)  # 防止数值爆炸
        
        # 5. 策略更新（加权BC）
        policy_actions = self.actor(states)
        log_probs = -((policy_actions - actions) ** 2).sum(dim=-1)
        
        # 用指数优势加权
        actor_loss = -(exp_adv * log_probs).mean()
        
        self.optim.zero_grad()
        (v_loss + q1_loss + q2_loss).backward()
        self.optim.step()
        
        self.optim_actor.zero_grad()
        actor_loss.backward()
        self.optim_actor.step()
        
        return v_loss.item(), q1_loss.item() + q2_loss.item(), actor_loss.item()
```

IQL的核心洞察：**不追求精确的Q值估计，而是通过expectile regression隐式地学习。** 这样学出来的V函数更稳定，不容易过估计。

## D4RL：为什么这个benchmark如此重要

做离线RL研究，D4RL是绕不开的benchmark。它来自加州大学伯克利分校，提供了一系列标准化的离线RL数据集。

D4RL包含多种任务：

| 环境 | 数据类型 | 特点 |
|------|---------|------|
| MuJoCo (HalfCheetah, Hopper, Walker) | random, medium, medium-expert, expert | 机器人 locomotion |
| AntMaze | umaze, umaze-diverse, medium-diverse, large-diverse | 迷宫导航 |
| Adroit | random, Claw, Pen, Door | 精细操作任务 |
| Franka Kitchen | partial, mixed | 厨房任务 |

每个环境有不同质量的数据集：
- **random**：完全随机策略生成的数据
- **medium**：中等水平策略（50%最优）
- **medium-expert**：混合数据
- **expert**：专家策略生成的数据

为什么D4RL重要？因为它解决了离线RL研究中的一个关键问题：**没有标准benchmark，论文结果没法比较**。之前每个论文都用自己造的数据集，很难说清楚谁好谁坏。

```python
import d4rl

def load_d4rl_dataset(env_name):
    """
    加载D4RL数据集
    """
    env = gym.make(env_name)
    dataset = env.get_dataset()
    
    print(f"Dataset size: {len(dataset['observations'])}")
    print(f"Observation shape: {dataset['observations'].shape}")
    print(f"Action shape: {dataset['actions'].shape}")
    
    # 统计一下数据质量
    rewards = dataset['rewards']
    print(f"Mean reward: {rewards.mean():.3f}")
    print(f"Max reward: {rewards.max():.3f}")
    print(f"Reward distribution: min={rewards.min():.3f}, "
          f"median={np.median(rewards):.3f}, max={rewards.max():.3f}")
    
    return dataset

# 加载数据集
dataset = load_d4rl_dataset('halfcheetah-medium-v2')
```

## 代码实战：d3rlpy实现离线RL

d3rlpy是日本研究者开发的离线RL库，接口很干净，文档也全。下面用它来实现完整的离线RL训练流程。

```python
import d3rlpy
import gymnasium as gym
import numpy as np
from d3rlpy.datasets import get_d4rl

# ============== 准备数据 ==============
# 方法1：使用d3rlpy自带的d4rl加载
dataset, env = get_d4rl('hopper-medium-v2')

# 方法2：自己加载
# env = gym.make('Hopper-v4')
# dataset = d4rl.online_to_offline(env, policy=...)  # 把在线数据转成离线格式

print(f"Dataset size: {len(dataset)}")
print(f"State dim: {dataset.observation_shape}")
print(f"Action dim: {dataset.action_shape}")

# ============== 选择算法 ==============
# CQL
cql = d3rlpy.algos.CQL(
    actor_learning_rate=3e-4,
    critic_learning_rate=3e-4,
    batch_size=256,
    gamma=0.99,
    tau=0.005,
    min_q_weight=1.0,  # CQL正则项权重
    temperature=1.0,
)

# IQL（更简单，效果也经常差不多）
iql = d3rlpy.algos.IQL(
    actor_learning_rate=3e-4,
    critic_learning_rate=3e-4,
    batch_size=256,
    gamma=0.99,
    expectile=0.7,
    weight_type='softmax',
)

# TD3+BC（最简单）
td3_bc = d3rlpy.algos.TD3PlusBC(
    actor_learning_rate=3e-4,
    critic_learning_rate=3e-4,
    batch_size=256,
    gamma=0.99,
    alpha=2.5,
)

# ============== 训练 ==============
# 基础训练
td3_bc.fit(
    dataset,
    n_steps=100000,
    experiment_name="td3_bc_hopper",
    tensorboard_log_dir="./logs",
)

# 评估
eval_env = gym.make('Hopper-v4')
returns = td3_bc.evaluate(eval_env, n_trials=10)
print(f"Mean return: {np.mean(returns):.2f} +/- {np.std(returns):.2f}")

# 保存模型
td3_bc.save_policy("td3_bc_hopper.pt")

# 加载模型
loaded_policy = d3rlpy.algos.TD3PlusBC.load("td3_bc_hopper.pt")

# ============== 完整训练脚本 ==============
def train_offline_rl(env_name='hopper-medium-v2', algo_name='td3_bc', 
                    n_steps=100000):
    # 加载数据和环境
    dataset, env = get_d4rl(env_name)
    
    # 根据算法名选择
    algos = {
        'cql': d3rlpy.algos.CQL,
        'iql': d3rlpy.algos.IQL,
        'td3_bc': d3rlpy.algos.TD3PlusBC,
    }
    
    algo = algos[algo_name](
        actor_learning_rate=3e-4,
        critic_learning_rate=3e-4,
        batch_size=256,
        gamma=0.99,
    )
    
    # 训练
    algo.fit(
        dataset,
        n_steps=n_steps,
        experiment_name=f"{algo_name}_{env_name}",
        tensorboard_log_dir="./logs",
        save_interval=10000,
    )
    
    # 评估
    eval_env = gym.make(env_name.split('-')[0] + '-v4')
    returns = algo.evaluate(eval_env, n_trials=10)
    
    print(f"\nFinal Results for {algo_name} on {env_name}:")
    print(f"  Mean return: {np.mean(returns):.2f}")
    print(f"  Std return: {np.std(returns):.2f}")
    print(f"  Min return: {np.min(returns):.2f}")
    print(f"  Max return: {np.max(returns):.2f}")
    
    return algo, returns

if __name__ == '__main__':
    # 训练不同算法对比
    for algo in ['td3_bc', 'cql', 'iql']:
        train_offline_rl('hopper-medium-v2', algo, n_steps=50000)
```

## 离线RL的调参注意点

### 1. CQL的min_q_weight是关键

CQL论文里做了大量的消融实验，发现min_q_weight这个参数对结果影响最大：

- 太小（<0.5）：正则不够，离线RL的问题依然存在
- 太大（>10）：策略被压得太保守，学不到东西
- 推荐范围：1.0~5.0，具体看任务

调参技巧：**先从1.0开始，如果发现策略跑偏（训练曲线突然变差），加大到2.0-5.0。**

### 2. 数据质量比算法更重要

我做过很多实验，发现一个扎心的结论：**同一算法，数据质量好效果就好，数据质量差神仙也难救。**

| 数据类型 | 典型效果 |
|---------|---------|
| expert | 可以接近专家水平 |
| medium-expert | 通常能达到medium到expert之间 |
| medium | 效果有限，很难超过medium |
| random | 基本没戏 |

所以做离线RL项目，第一件事是检查数据质量。

### 3. 观察价值估计

离线RL训练时，密切关注Q值的变化：

```python
def monitor_training(algo, dataset):
    """训练过程中的监控"""
    states = dataset.observations[:1000]
    
    for step in range(0, 100000, 1000):
        # 计算当前Q值估计
        with torch.no_grad():
            actions = algo.actor(torch.FloatTensor(states))
            q_values = algo.Q1(states, actions).mean().item()
        
        # 记录
        print(f"Step {step}: Q-value estimate = {q_values:.2f}")
        
        # 检测异常
        if q_values > 1000 or q_values < -100:
            print("WARNING: Q-value exploded!")
            return False
    
    return True
```

### 4. 渐进式训练

对于特别难的任务，可以考虑渐进式策略：

```python
def progressive_cql_training(dataset, algo='cql'):
    """渐进式训练：先BC预热，再RL微调"""
    
    # 阶段1：BC预热
    print("Phase 1: BC warmup...")
    warmup_algo = d3rlpy.algos.BC(
        batch_size=256,
        learning_rate=3e-4,
    )
    warmup_algo.fit(dataset, n_steps=10000)
    
    # 阶段2：RL微调
    print("Phase 2: RL fine-tuning...")
    rl_algo = d3rlpy.algos.CQL(actor_learning_rate=1e-4)  # 学习率降低
    rl_algo.copy_policy_from(warmup_algo)  # 从BC策略初始化
    rl_algo.fit(dataset, n_steps=100000)
    
    return rl_algo
```

## 总结

离线RL是个很有意思的领域，因为它解决了实际应用中的核心问题：**数据是现成的，但没法在线探索。**

关键要点：

1. **分布偏移是核心挑战**：策略和数据分布不匹配会导致Q值崩塌
2. **CQL是目前的主流方法**：保守估计Q值，避免过度乐观
3. **TD3+BC简单有效**：大道至简，有时候简单就是好
4. **IQL是个不错的备选**：不用直接估计Q值，更稳定
5. **D4RL是标配benchmark**：做研究必须用它
6. **调参要耐心**：min_q_weight是关键，数据质量比算法更重要

实际项目中，我建议的策略是：**先用TD3+BC快速跑通baseline，效果不好再上CQL或IQL。** 大部分问题其实不是算法不够 fancy，而是数据质量或超参数的问题。
