---
title: Actor-Critic算法深度指南
date: 2026-04-24
tags:
  - 强化学习
  - Actor-Critic
  - A2C
  - A3C
  - GAE
  - 策略梯度
aliases:
  - Actor-Critic深度指南
  - A2C与A3C算法详解
---

# Actor-Critic算法深度指南

## 为什么需要Actor-Critic

先说说强化学习里两个重要流派各自的苦衷。纯策略梯度方法（比如REINFORCE）有个致命问题——方差太大。你想象一下，智能体在环境中探索，有时候运气好拿到正向奖励，有时候运气差拿到负向奖励，这些随机性会导致策略更新的梯度估计晃得厉害。训练起来就是一会儿好一会儿差，曲线看着跟心电图似的，贼不稳定。

纯值函数方法（Q-Learning、DQN）呢，问题是它只能处理离散动作空间，而且学出来的是确定性策略，遇到需要随机性的场景就抓瞎了。更麻烦的是值函数的估计本身也有误差，这个误差会传播，导致Q值过估计的问题。

Actor-Critic的思想就是：能不能把这两个家伙的优点捏在一起？Actor负责输出策略（策略梯度），Critic负责估计值函数（减少方差）。Critic告诉Actor"你这步走得怎么样"，Actor根据Critic的反馈来调整自己的策略。这样既有策略梯度的灵活性（能处理连续动作），又有值函数的低方差优势。

说白了，Actor-Critic就是一个"理论指导+实践反馈"的组合。Actor是干活的，Critic是出主意的。干活的听主意的反馈调整动作，主意的评估干活的质量。

## Advantage Actor-Critic (A2C)

A2C是Actor-Critic家族里最基础的成员，理解了A2C就等于拿到了入场券。它的核心创新是用**优势函数**（Advantage Function）来替代原始的回报。

回顾一下原始策略梯度：$\nabla J(\theta) = \mathbb{E}[G_t \nabla \log \pi_\theta(a_t|s_t)]$，这里$G_t$是时序差分回报，方差贼大。A2C改成用优势函数：

$$A(s_t, a_t) = Q(s_t, a_t) - V(s_t)$$

优势函数的物理意义很直观：你选择动作$a_t$比按照当前策略选择动作好多少。如果$A>0$，说明这个动作比平均好，值得加强；$A<0$说明这个动作拖后腿了，应该削弱。

实际落地的时候，我们通常用TD误差来近似优势函数：$\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$。这个近似有两个好处：
1. 计算简单，不需要额外学习Q函数
2. 在线就能算，不需要等整个episode结束

A2C的伪代码大概是这个样子：

```python
for episode in range(num_episodes):
    states, actions, rewards = [], [], []
    state = env.reset()
    done = False
    
    while not done:
        action = actor.select_action(state)
        next_state, reward, done, _ = env.step(action)
        
        states.append(state)
        actions.append(action)
        rewards.append(reward)
        state = next_state
    
    # 计算优势并更新
    returns = compute_gae(rewards, values, gamma, lam)
    actor.update(states, actions, returns)
    critic.update(states, returns)
```

注意A2C的训练是同步的——等所有worker把数据收集齐了一起更新。这就是它和A3C的主要区别。

## A3C：异步并行的力量

A3C（Asynchronous Advantage Actor-Critic）是DeepMind在2016年放出来的大招。它的核心思想是：**独乐乐不如众乐乐，让多个worker同时在不同的环境中探索**。

传统的RL训练有个蛋疼的地方：样本效率太低。智能体在环境里吭哧吭哧探索，收集一条trajectory可能要好几十步，但每次更新只用这一条。异步并行的意思就是：开多个进程，每个进程有自己的智能体副本，在各自的环境中独立探索，然后异步地把自己的梯度上传到全局网络。

A3C的结构大概是这样的：

```python
import multiprocessing as mp
import numpy as np

class A3CWorker(mp.Process):
    def __init__(self, global_network, worker_id, env_id):
        super().__init__()
        self.global_network = global_network
        self.worker_id = worker_id
        self.env_id = env_id
        self.local_network = ActorCritic()  # 本地副本
    
    def run(self):
        while global_step < max_steps:
            # 1. 同步本地网络参数
            self.local_network.sync_from(self.global_network)
            
            # 2. 收集若干步经验
            states, actions, rewards = [], [], []
            state = self.env.reset()
            done = False
            step = 0
            
            while not done and step < 20:
                action = self.local_network.act(state)
                next_state, reward, done, _ = self.env.step(action)
                
                states.append(state)
                actions.append(action)
                rewards.append(reward)
                
                state = next_state
                step += 1
            
            # 3. 计算优势
            returns = self.compute_returns(rewards, done)
            
            # 4. 计算梯度并异步更新全局网络
            grads = self.local_network.compute_gradients(states, actions, returns)
            self.global_network.apply_gradients(grads)
```

为什么A3C有效？主要有三点原因：
1. **探索多样性**：多个worker在不同环境状态下探索，收集的样本多样性更高
2. **减少相关性**：如果用单个agent连续收集经验，相邻步骤高度相关，用它们更新会导致梯度估计有偏。多worker自然打破了这种相关性
3. **计算效率**：环境交互和梯度计算可以并行执行

不过A3C有个坑需要注意：异步更新会导致"stale gradients"问题。假设worker A算好了梯度准备上传，但worker B已经更新了全局网络，这时候A的梯度可能是基于旧参数算的，用它更新会拉偏网络。实践中这个问题影响没那么大，但确实存在。

## 策略与值函数网络设计

在实际实现中，Actor和Critic的网络怎么设计是个值得权衡的问题。

**共享编码器方案**：

```python
class ActorCriticShared(nn.Module):
    def __init__(self, state_dim, action_dim, hidden=64):
        super().__init__()
        # 共享的特征提取层
        self.shared = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.Tanh(),
            nn.Linear(hidden, hidden),
            nn.Tanh()
        )
        
        # Actor头：输出动作分布
        self.actor = nn.Sequential(
            nn.Linear(hidden, action_dim),
            # 连续动作用tanh激活
            nn.Tanh()
        )
        self.log_std = nn.Parameter(torch.zeros(action_dim))
        
        # Critic头：输出状态价值
        self.critic = nn.Sequential(
            nn.Linear(hidden, 1)
        )
    
    def forward(self, x):
        features = self.shared(x)
        return features
    
    def get_action(self, state):
        features = self.forward(state)
        mean = self.actor(features)
        std = torch.exp(self.log_std)
        dist = torch.distributions.Normal(mean, std)
        action = dist.sample()
        return action
    
    def get_value(self, state):
        features = self.forward(state)
        return self.critic(features)
```

**为什么共享设计更好？**
1. 节省计算：特征提取只需要跑一遍
2. 表征学习：价值函数的监督信号能帮助学到更好的状态特征，对策略也有帮助
3. 收敛更稳定：两个目标一起优化，学到的特征"性价比"更高

**分离设计什么时候用？**
当Actor和Critic的需求差异很大时。比如Actor需要精细的策略分布，Critic需要精确的价值估计，它们的最优特征可能不太一样。这种情况在复杂的视觉输入任务中更常见。

## 熵正则化：鼓励探索的利器

RL训练有个经典问题：策略容易过早收敛到局部最优。具体表现就是智能体找到一个"凑合能用"的策略后就躺平了，不再探索其他可能性。

熵正则化就是来解决这个问题的。它的思想很简单：在优化目标里加一项策略的熵：

$$J(\theta) = \mathbb{E}[r(\tau)] + \beta \cdot H(\pi_\theta)$$

其中$H(\pi_\theta) = \mathbb{E}[-\log \pi_\theta(a|s)]$是策略的熵，$\beta$是控制探索强度的系数。

熵越大意味着策略越随机（在每个状态下选择动作的概率分布越均匀）。把这个目标加进去，优化过程就会倾向于保持一定程度的随机性，不至于太早就锁定在确定性策略上。

代码实现上，需要在损失函数里加一项：

```python
def compute_loss(self, states, actions, returns):
    # 策略损失（加上熵正则化）
    dist = self.actor(states)
    log_probs = dist.log_prob(actions).sum(dim=-1)
    actor_loss = -(log_probs * advantages.detach()).mean()
    
    # 熵正则项
    entropy = dist.entropy().mean()
    actor_loss -= self.entropy_coef * entropy
    
    # 价值损失
    values = self.critic(states).squeeze()
    critic_loss = F.mse_loss(values, returns)
    
    return actor_loss + 0.5 * critic_loss
```

熵系数$\beta$的选取有讲究：
- 太大：策略太随机，学不到有效行为
- 太小：探索不够，容易陷入局部最优

常见做法是动态调整：训练初期设大一些让智能体充分探索，后期慢慢减小让它收敛到好策略。

## GAE：减少方差的艺术

GAE（Generalized Advantage Estimation）是John Schulman等人提出的方差缩减技术，现在已经是Proximal Policy Optimization (PPO)的标配。

回顾一下优势函数：$A(s_t, a_t) = \sum_{l=0}^{\infty} \gamma^l \delta_{t+l}$

其中$\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$是TD误差。

这就是TD($\lambda$)在策略梯度里的应用。GAE的定义是：

$$A_t^{GAE}(\gamma, \lambda) = \sum_{l=0}^{\infty} (\gamma\lambda)^l \delta_{t+l}$$

这个公式的美妙之处在于$\lambda$参数：

- 当$\lambda = 0$时：$A_t = \delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$，就是一步TD误差，方差最低但偏差较大
- 当$\lambda = 1$时：$A_t = \sum_{l=0}^{\infty} \gamma^l r_{t+l}$，就是完整的Monte Carlo回报，无偏但方差最大

$\lambda$在0和1之间连续变化，实现了偏差-方差的平滑 tradeoff。实践中通常取$\lambda = 0.95$或$\lambda = 0.99$。

GAE的代码实现：

```python
def compute_gae(rewards, values, gamma=0.99, lam=0.95):
    """
    rewards: list of rewards
    values: list of value estimates (including bootstrap)
    """
    advantages = []
    gae = 0
    
    # 从后往前算
    for t in reversed(range(len(rewards))):
        if t == len(rewards) - 1:
            next_value = 0  # terminal state
        else:
            next_value = values[t + 1]
        
        delta = rewards[t] + gamma * next_value - values[t]
        gae = delta + gamma * lam * gae
        advantages.insert(0, gae)
    
    return advantages
```

关键点是从后往前递推，因为后面的优势依赖于前面的TD误差。

## 代码实战：PyTorch实现A2C

下面用一个完整的A2C实现来串联所有概念。训练环境用Gymnasium的CartPole和Pendulum。

```python
import gymnasium as gym
import numpy as np
import torch
import torch.nn as nn
import torch.optim as optim
from collections import deque
import random

# ============== 网络定义 ==============
class ActorCritic(nn.Module):
    def __init__(self, state_dim, action_dim, hidden=64):
        super().__init__()
        self.shared = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.Tanh(),
            nn.Linear(hidden, hidden),
            nn.Tanh()
        )
        
        # Actor输出均值，log_std作为可学习参数
        self.actor_mean = nn.Linear(hidden, action_dim)
        self.log_std = nn.Parameter(torch.zeros(action_dim))
        
        # Critic输出状态价值
        self.critic = nn.Sequential(
            nn.Linear(hidden, hidden),
            nn.Tanh(),
            nn.Linear(hidden, 1)
        )
    
    def forward(self, x):
        features = self.shared(x)
        return features
    
    def get_action(self, state, deterministic=False):
        features = self.forward(state)
        mean = self.actor_mean(features)
        std = torch.exp(self.log_std)
        
        if deterministic:
            action = torch.tanh(mean)  # 直接返回均值
            return action, 0.0  # 熵为0
        
        dist = torch.distributions.Normal(mean, std)
        action = dist.sample()
        log_prob = dist.log_prob(action).sum(dim=-1)
        entropy = dist.entropy().sum(dim=-1).mean()
        return action, log_prob, entropy
    
    def get_value(self, state):
        features = self.forward(state)
        return self.critic(features)

# ============== GAE计算 ==============
def compute_gae(rewards, values, gamma=0.99, lam=0.95):
    advantages = []
    gae = 0
    
    for t in reversed(range(len(rewards))):
        if t == len(rewards) - 1:
            next_value = 0
        else:
            next_value = values[t + 1]
        
        delta = rewards[t] + gamma * next_value - values[t]
        gae = delta + gamma * lam * gae
        advantages.insert(0, gae)
    
    return advantages

# ============== A2C训练器 ==============
class A2CTrainer:
    def __init__(self, env_name, lr=3e-4, gamma=0.99, lam=0.95, 
                 entropy_coef=0.01, value_coef=0.5, max_grad_norm=0.5):
        self.env = gym.make(env_name)
        self.gamma = gamma
        self.lam = lam
        self.entropy_coef = entropy_coef
        self.value_coef = value_coef
        self.max_grad_norm = max_grad_norm
        
        state_dim = self.env.observation_space.shape[0]
        action_dim = self.env.action_space.shape[0]
        
        self.policy = ActorCritic(state_dim, action_dim)
        self.optimizer = optim.Adam(self.policy.parameters(), lr=lr)
        
    def collect_trajectory(self):
        """收集一条完整的trajectory"""
        states, actions, rewards, values, log_probs = [], [], [], [], []
        state, _ = self.env.reset()
        done = False
        
        while not done:
            state_t = torch.FloatTensor(state).unsqueeze(0)
            
            with torch.no_grad():
                value = self.policy.get_value(state_t).item()
            
            action, log_prob, entropy = self.policy.get_action(state_t)
            action_np = action.squeeze().numpy()
            
            next_state, reward, terminated, truncated, _ = self.env.step(action_np)
            done = terminated or truncated
            
            states.append(state)
            actions.append(action)
            rewards.append(reward)
            values.append(value)
            log_probs.append(log_prob)
            
            state = next_state
        
        # 计算回报
        returns = compute_gae(rewards, values, self.gamma, self.lam)
        
        return states, actions, rewards, log_probs, returns
    
    def update(self, states, actions, log_probs_old, returns):
        """执行一次策略更新"""
        # 转换为tensor
        states = torch.FloatTensor(np.array(states))
        actions = torch.cat(actions)
        returns = torch.FloatTensor(returns)
        log_probs_old = torch.cat(log_probs_old)
        
        # 重新计算log_prob和熵
        dist = torch.distributions.Normal(
            self.policy.actor_mean(self.policy.shared(states)),
            torch.exp(self.policy.log_std)
        )
        log_probs = dist.log_prob(actions).sum(dim=-1)
        entropy = dist.entropy().sum(dim=-1).mean()
        
        # 策略损失
        advantages = returns - self.policy.get_value(states).squeeze().detach()
        policy_loss = -(log_probs * advantages).mean()
        
        # 加熵正则化
        policy_loss -= self.entropy_coef * entropy
        
        # 价值损失
        values = self.policy.get_value(states).squeeze()
        value_loss = self.value_coef * F.mse_loss(values, returns)
        
        # 总损失
        loss = policy_loss + value_loss
        
        # 反向传播
        self.optimizer.zero_grad()
        loss.backward()
        nn.utils.clip_grad_norm_(self.policy.parameters(), self.max_grad_norm)
        self.optimizer.step()
        
        return loss.item(), entropy.item()

# ============== 训练循环 ==============
def train_a2c(env_name='CartPole-v1', num_episodes=500):
    trainer = A2CTrainer(env_name)
    
    reward_history = deque(maxlen=10)
    
    for episode in range(num_episodes):
        states, actions, rewards, log_probs, returns = trainer.collect_trajectory()
        loss, entropy = trainer.update(states, actions, log_probs, returns)
        
        total_reward = sum(rewards)
        reward_history.append(total_reward)
        
        if episode % 10 == 0:
            avg_reward = np.mean(reward_history)
            print(f"Episode {episode}: Avg Reward = {avg_reward:.1f}, "
                  f"Loss = {loss:.3f}, Entropy = {entropy:.3f}")
        
        if np.mean(reward_history) > 450:
            print(f"Solved at episode {episode}!")
            break
    
    return trainer.policy

if __name__ == '__main__':
    # 训练CartPole
    print("Training on CartPole...")
    train_a2c('CartPole-v1')
    
    # 训练Pendulum（连续动作空间）
    print("\nTraining on Pendulum...")
    train_a2c('Pendulum-v1')
```

**CartPole vs Pendulum的区别**：

| 特性 | CartPole | Pendulum |
|------|---------|----------|
| 动作空间 | 离散（2个动作） | 连续（力矩） |
| 状态维度 | 4 | 3 |
| 奖励设计 | 步数越多越好 | 靠近竖直且省力好 |
| 训练难度 | 相对简单 | 需要更细致的调参 |

## 调试技巧

### 策略崩溃

策略崩溃的表现是：训练过程中策略突然变得很差，熵急剧下降，然后一直保持很差的状态。

**诊断步骤**：
```python
# 监控这些指标
def diagnose_policy_collapse(policy, states_history, rewards_history):
    if len(states_history) < 100:
        return
    
    # 1. 检查熵是否骤降
    recent_entropy = states_history[-100:]
    if np.std(recent_entropy) > 0.5:  # 熵的波动太大
        print("WARNING: Entropy collapse detected!")
    
    # 2. 检查价值损失是否爆炸
    if rewards_history[-1] < np.mean(rewards_history[-50:]) - 2 * np.std(rewards_history[-50:]):
        print("WARNING: Reward collapse detected!")
```

**常见原因和解决方案**：
1. 学习率太大：试试1e-4或更小
2. 梯度爆炸：确保用了梯度裁剪
3. 奖励尺度不一致：做奖励归一化

### 价值过估计

在Actor-Critic框架里，Critic估计的价值可能系统性地高于真实值，这会导致Actor被误导。

**诊断方法**：
```python
# 定期检查价值估计的准确性
def check_value_accuracy(policy, env, num_episodes=10):
    overestimates = []
    
    for _ in range(num_episodes):
        state, _ = env.reset()
        episode_returns = 0
        
        while True:
            state_t = torch.FloatTensor(state).unsqueeze(0)
            with torch.no_grad():
                predicted_value = policy.get_value(state_t).item()
            
            action, _, _ = policy.get_action(state_t)
            next_state, reward, done, _, _ = env.step(action.squeeze().numpy())
            episode_returns += reward
            
            if done:
                # 比较预测值和实际回报
                overestimates.append(predicted_value - episode_returns)
                break
            state = next_state
    
    avg_overestimate = np.mean(overestimates)
    print(f"Average value overestimate: {avg_overestimate:.2f}")
    
    if avg_overestimate > 50:
        print("WARNING: Significant value overestimation!")
```

**解决方案**：
1. 用双Q网络（Double Q-learning）的思想，让Actor和Critic用不同的网络
2. 减小Critic的学习率
3. 在价值损失上加L2正则化

### 其他常见问题

**问题1：训练曲线剧烈震荡**
- 原因：批量大小太小，更新太频繁
- 解决：增大批量大小（128或256），或者用更大的GAE的$\lambda$

**问题2：智能体行为很"机械"，熵太小**
- 原因：熵系数太小
- 解决：增大熵系数到0.05-0.1

**问题3：看起来收敛了但实际效果很差**
- 原因：价值函数还没学准，策略在错误的价值信号下优化
- 解决：先单独训练Critic几个epoch，再一起训练

**问题4：多环境并行时有的环境特别慢**
- 原因：环境实现有问题或者环境之间状态差异太大
- 解决：用向量化环境统一管理，或者加超时机制

## 总结

Actor-Critic是强化学习的核心范式，理解了它就理解了现代RL的一半。A2C和A3C是入门的好起点，GAE是减少方差的标准武器，熵正则化是防止策略崩溃的秘密武器。

调试RL代码的黄金法则：**监控一切，保存checkpoint，从简单基线开始**。很多看起来玄学的问题，最后都是超参数或者bug导致的。

如果你是刚入门，建议先跑通A2C的代码，再尝试A3C的并行版本。等这些都玩转了，可以去试试PPO——它其实是A2C的升级版，更稳定也更常用。
