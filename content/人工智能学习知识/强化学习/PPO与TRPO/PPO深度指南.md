---
title: PPO深度指南
date: 2026-04-18
tags:
  - 强化学习
  - PPO
  - TRPO
  - 策略优化
  - 信任域
  - 剪裁
categories:
  - 强化学习
  - 策略优化方法
alias:
  - Proximal Policy Optimization
  - Trust Region Policy Optimization
  - Clipped Surrogate Objective
---

# PPO深度指南

## 关键词速览

| 核心概念 | 信任域 | 剪裁目标 | KL散度 | 自适应惩罚 |
|:---------|:-------|:---------|:-------|:-----------|
| 自然梯度 | 价值裁剪 | 广义优势估计 | 策略更新 | 目标函数 |

> [!abstract]+ 核心关键词表
> | 术语 | 英文 | 符号/技术 | 说明 |
> |:-----|:-----|:----------|:-----|
> | 近端策略优化 | PPO | PPO | 稳定策略更新的算法 |
> | 信任域策略优化 | TRPO | TRPO | 基于KL散度约束的策略优化 |
> | 剪裁目标函数 | Clipped Objective | $\min(r_t, \text{clip})$ | 防止策略过大更新 |
> | 替代优势函数 | Surrogate Advantage | $r_t(\theta) \cdot \hat{A}_t$ | 策略比值的加权 |
> | KL散度 | KL Divergence | $D_{KL}(\pi_{\theta_{old}} || \pi_\theta)$ | 策略分布差异度量 |
> | 自适应KL惩罚 | Adaptive KL Penalty | $\beta$ | 自动调整的KL惩罚系数 |
> | 价值函数裁剪 | Value Clipping | $\text{clip}(V, V_{old} - \epsilon, V_{old} + \epsilon)$ | 防止价值函数剧变 |
> | 广义优势估计 | GAE | $\hat{A}_t^{GAE(\lambda)}$ | 偏差-方差平衡的优势估计 |
> | 目标函数 | Objective | $L^{CLIP+VF+S}$ | PPO综合目标 |
> | 策略比值 | Probability Ratio | $r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$ | 新旧策略的概率比 |

---

## 一、从TRPO到PPO

### 1.1 TRPO原理

信任域策略优化（Trust Region Policy Optimization, TRPO）由Schulman等人于2015年提出，是一种保证策略单调改进的策略优化方法。TRPO的核心约束是限制相邻策略之间的KL散度：

$$
\max_\theta \quad \mathbb{E}_t \left[ \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)} \hat{A}_t \right]
$$

$$
\text{s.t.} \quad \mathbb{E}_t \left[ D_{KL}(\pi_{\theta_{old}}(\cdot|s_t) || \pi_\theta(\cdot|s_t)) \right] \leq \delta
$$

> [!info]+ TRPO的核心思想
> "信任域"隐喻：想象你在黑暗中山徒步，只能信任手电筒照亮的区域。TRPO同样限制策略更新步长在"可信任"的范围内，确保每次更新都是安全的改进。

### 1.2 TRPO的计算挑战

TRPO需要求解约束优化问题，通常使用共轭梯度法近似求解自然梯度。这种方法存在以下问题：

1. **计算复杂度高**：需要二阶优化，计算量大
2. **内存开销大**：需要存储Fisher信息矩阵或使用K-FAC近似
3. **与噪声抑制不兼容**：无法轻易与GAE等噪声技术结合
4. **实现困难**：自然梯度实现复杂

### 1.3 PPO的诞生

PPO（Proximal Policy Optimization）由Schulman等人于2017年提出，通过**一阶优化**和**剪裁机制**简化了TRPO，同时保持甚至超越了TRPO的性能。PPO的核心洞察是：**不需要精确的约束，只需要防止策略更新过大**。

---

## 二、PPO算法详解

### 2.1 剪裁替代目标函数

PPO使用剪裁机制限制策略更新的幅度：

$$
L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta) \cdot \hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \cdot \hat{A}_t \right) \right]
$$

其中策略比值：

$$
r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}
$$

### 2.2 剪裁机制的几何解释

```python
def compute_ppo_loss(log_probs_old, log_probs_new, advantages, epsilon=0.2):
    """
    PPO clip loss computation.
    
    epsilon: 超参数，通常为0.1或0.2
    """
    # 策略比值
    ratio = torch.exp(log_probs_new - log_probs_old)
    
    # 未剪裁的替代损失
    surr1 = ratio * advantages
    
    # 剪裁后的替代损失
    surr2 = torch.clamp(ratio, 1 - epsilon, 1 + epsilon) * advantages
    
    # 取最小值（当A>0时剪裁下界，当A<0时剪裁上界）
    policy_loss = -torch.min(surr1, surr2).mean()
    
    return policy_loss
```

> [!note]+ 剪裁机制详解
> 考虑两种情况：
> 
> **优势函数 $A > 0$（动作好于平均）：**
> - 如果 $r_t > 1 + \epsilon$：比值被剪裁到 $1+\epsilon$，阻止过度增加该动作概率
> - $\min$ 操作确保我们不会过度乐观地估计收益
> 
> **优势函数 $A < 0$（动作差于平均）：**
> - 如果 $r_t < 1 - \epsilon$：比值被剪裁到 $1-\epsilon$，阻止过度降低该动作概率
> - 这提供了一种"保护"，即使策略很差也能稳定更新

### 2.3 完整PPO实现

```python
import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
import gym

class PPOPolicy(nn.Module):
    """
    PPO policy network with Gaussian policy for continuous control.
    """
    def __init__(self, obs_dim, act_dim, hidden_dim=64):
        super().__init__()
        
        # Actor (policy)
        self.actor = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.Tanh(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.Tanh(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.Tanh()
        )
        self.log_std = nn.Parameter(torch.zeros(act_dim))
        
        # Critic (value function)
        self.critic = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.Tanh(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.Tanh(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.Tanh(),
            nn.Linear(hidden_dim, 1)
        )
    
    def forward(self, x):
        return self.actor(x), self.critic(x)
    
    def get_action(self, x, deterministic=False):
        """Sample action from policy."""
        mean = self.actor(x)
        std = self.log_std.exp()
        
        if deterministic:
            action = mean
        else:
            dist = torch.distributions.Normal(mean, std)
            action = dist.sample()
        
        action = torch.tanh(action)
        return action, dist.log_prob(action)
    
    def evaluate_actions(self, x, actions):
        """Compute log probs for actions (for PPO update)."""
        mean = self.actor(x)
        std = self.log_std.exp()
        dist = torch.distributions.Normal(mean, std)
        
        # Actions are already tanh-squashed
        log_probs = dist.log_prob(actions)
        
        # Add tanh correction for log_prob
        action_tanh = torch.tanh(actions)
        log_probs -= torch.log(1 - action_tanh.pow(2) + 1e-6)
        
        return log_probs.sum(-1, keepdim=True)

class PPOAgent:
    """
    Proximal Policy Optimization (PPO) agent.
    """
    def __init__(self, obs_dim, act_dim, hidden_dim=64, 
                 lr=3e-4, gamma=0.99, lam=0.95, 
                 clip_epsilon=0.2, value_coef=0.5, entropy_coef=0.0,
                 ppo_epochs=10, batch_size=64,
                 max_grad_norm=0.5):
        
        self.clip_epsilon = clip_epsilon
        self.value_coef = value_coef
        self.entropy_coef = entropy_coef
        self.ppo_epochs = ppo_epochs
        self.batch_size = batch_size
        self.max_grad_norm = max_grad_norm
        self.gamma = gamma
        self.lam = lam
        
        # Network
        self.policy = PPOPolicy(obs_dim, act_dim, hidden_dim)
        self.optimizer = optim.Adam(self.policy.parameters(), lr=lr)
        
        # Memory buffer
        self.buffer = []
    
    def select_action(self, obs, deterministic=False):
        """Select action given observation."""
        obs_tensor = torch.FloatTensor(obs).unsqueeze(0)
        
        with torch.no_grad():
            action, log_prob = self.policy.get_action(obs_tensor, deterministic)
            value = self.policy.critic(obs_tensor)
        
        return action.numpy()[0], log_prob.item(), value.item()
    
    def store_transition(self, obs, action, log_prob, value, reward, done):
        """Store transition in buffer."""
        self.buffer.append({
            'obs': obs,
            'action': action,
            'log_prob': log_prob,
            'value': value,
            'reward': reward,
            'done': done
        })
    
    def compute_gae(self, rewards, values, dones):
        """Compute Generalized Advantage Estimation."""
        advantages = []
        gae = 0
        
        values = [0.0] + values + [0.0]  # Pad values
        
        for t in reversed(range(len(rewards))):
            delta = rewards[t] + self.gamma * values[t + 1] * (1 - dones[t]) - values[t]
            gae = delta + self.gamma * self.lam * (1 - dones[t]) * gae
            advantages.insert(0, gae)
        
        return np.array(advantages)
    
    def compute_returns(self, rewards, dones, gamma=0.99):
        """Compute discounted returns."""
        returns = []
        discounted = 0
        
        for reward, done in zip(reversed(rewards), reversed(dones)):
            discounted = reward + gamma * discounted * (1 - done)
            returns.insert(0, discounted)
        
        return np.array(returns)
    
    def update(self):
        """Perform PPO update from collected buffer."""
        if len(self.buffer) < self.batch_size:
            return {}
        
        # Extract from buffer
        obs = np.array([t['obs'] for t in self.buffer])
        actions = np.array([t['action'] for t in self.buffer])
        old_log_probs = np.array([t['log_prob'] for t in self.buffer])
        values = np.array([t['value'] for t in self.buffer])
        rewards = np.array([t['reward'] for t in self.buffer])
        dones = np.array([t['done'] for t in self.buffer])
        
        # Compute advantages and returns
        advantages = self.compute_gae(rewards, values, dones)
        returns = advantages + np.array(values)
        
        # Normalize advantages
        advantages = (advantages - advantages.mean()) / (advantages.std() + 1e-8)
        
        # Convert to tensors
        obs_tensor = torch.FloatTensor(obs)
        actions_tensor = torch.FloatTensor(actions)
        old_log_probs_tensor = torch.FloatTensor(old_log_probs)
        advantages_tensor = torch.FloatTensor(advantages)
        returns_tensor = torch.FloatTensor(returns)
        
        # PPO update for multiple epochs
        policy_losses = []
        value_losses = []
        entropy_losses = []
        
        dataset_size = len(obs)
        indices = np.arange(dataset_size)
        
        for epoch in range(self.ppo_epochs):
            np.random.shuffle(indices)
            
            for start in range(0, dataset_size, self.batch_size):
                end = start + self.batch_size
                batch_idx = indices[start:end]
                
                batch_obs = obs_tensor[batch_idx]
                batch_actions = actions_tensor[batch_idx]
                batch_old_log_probs = old_log_probs_tensor[batch_idx]
                batch_advantages = advantages_tensor[batch_idx]
                batch_returns = returns_tensor[batch_idx]
                
                # Evaluate actions
                log_probs = self.policy.evaluate_actions(batch_obs, batch_actions)
                
                # PPO policy loss (clipped)
                ratio = torch.exp(log_probs - batch_old_log_probs)
                
                surr1 = ratio * batch_advantages
                surr2 = torch.clamp(ratio, 1 - self.clip_epsilon, 1 + self.clip_epsilon) * batch_advantages
                policy_loss = -torch.min(surr1, surr2).mean()
                
                # Value loss (optional clipping)
                values_pred = self.policy.critic(batch_obs).squeeze()
                value_loss = nn.MSELoss()(values_pred, batch_returns)
                
                # Entropy bonus (if applicable)
                entropy_loss = 0  # Can add entropy regularization here
                
                # Total loss
                loss = policy_loss + self.value_coef * value_loss - self.entropy_coef * entropy_loss
                
                # Gradient step
                self.optimizer.zero_grad()
                loss.backward()
                torch.nn.utils.clip_grad_norm_(self.policy.parameters(), self.max_grad_norm)
                self.optimizer.step()
                
                policy_losses.append(policy_loss.item())
                value_losses.append(value_loss.item())
        
        # Clear buffer
        self.buffer.clear()
        
        return {
            'policy_loss': np.mean(policy_losses),
            'value_loss': np.mean(value_losses)
        }

def train_ppo(env_name='HalfCheetah-v2', num_steps=1000000, 
              num_env_steps=2048, update_freq=2048):
    """Train PPO agent."""
    env = gym.make(env_name)
    
    obs_dim = env.observation_space.shape[0]
    act_dim = env.action_space.shape[0]
    
    agent = PPOAgent(
        obs_dim=obs_dim,
        act_dim=act_dim,
        hidden_dim=64,
        lr=3e-4,
        gamma=0.99,
        lam=0.95,
        clip_epsilon=0.2,
        value_coef=0.5,
        ppo_epochs=10,
        batch_size=64
    )
    
    obs = env.reset()
    episode_rewards = []
    total_steps = 0
    
    while total_steps < num_steps:
        episode_reward = 0
        episode_steps = 0
        
        for _ in range(num_env_steps):
            action, log_prob, value = agent.select_action(obs)
            next_obs, reward, done, _ = env.step(action)
            
            agent.store_transition(obs, action, log_prob, value, reward, done)
            
            episode_reward += reward
            episode_steps += 1
            total_steps += 1
            
            obs = next_obs
            
            if done:
                episode_rewards.append(episode_reward)
                obs = env.reset()
                episode_reward = 0
        
        # Update
        losses = agent.update()
        
        if total_steps % 10000 == 0:
            avg_reward = np.mean(episode_rewards[-10:]) if episode_rewards else 0
            print(f"Steps: {total_steps}, Avg Reward (last 10): {avg_reward:.2f}")
            print(f"  Policy Loss: {losses.get('policy_loss', 0):.4f}, "
                  f"Value Loss: {losses.get('value_loss', 0):.4f}")
```

---

## 三、自适应KL散度惩罚

### 3.1 自适应KL机制

除了剪裁目标，PPO还支持自适应KL散度惩罚版本：

$$
L^{KLPEN}(\theta) = \mathbb{E}_t \left[ \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)} \hat{A}_t - \beta D_{KL}(\pi_{\theta_{old}} || \pi_\theta) \right]
$$

其中惩罚系数 $\beta$ 自适应调整：

```python
class AdaptiveKLPPO:
    """PPO with adaptive KL penalty."""
    
    def __init__(self, target_kl=0.01, kl_penalty_coef=0.5):
        self.target_kl = target_kl
        self.kl_penalty_coef = kl_penalty_coef
        self.kl_history = []
    
    def update_beta(self, kl_divergence):
        """Adaptively adjust KL penalty coefficient."""
        self.kl_history.append(kl_divergence)
        
        if len(self.kl_history) < 10:
            return self.kl_penalty_coef
        
        recent_kl = np.mean(self.kl_history[-10:])
        
        if recent_kl < self.target_kl * 0.5:
            # KL too small, increase penalty to encourage more change
            self.kl_penalty_coef *= 2
        elif recent_kl > self.target_kl * 2:
            # KL too large, decrease penalty to slow down
            self.kl_penalty_coef /= 2
        
        # Clamp to reasonable range
        self.kl_penalty_coef = np.clip(self.kl_penalty_coef, 1e-4, 1e3)
        
        return self.kl_penalty_coef
    
    def compute_kl_divergence(self, old_policy, new_policy, states):
        """Compute average KL divergence between policies."""
        with torch.no_grad():
            old_log_probs = old_policy.evaluate_actions(states, states)
            new_log_probs = new_policy.evaluate_actions(states, states)
            
            # Approximate KL: D_KL(P||Q) ≈ exp(log(P) - log(Q)) - 1 - (log(P) - log(Q))
            kl = torch.exp(old_log_probs - new_log_probs) - 1 - (old_log_probs - new_log_probs)
        
        return kl.mean().item()
```

### 3.2 两种PPO变体对比

| 特性 | PPO-Clip | PPO-Penalty |
|:-----|:---------|:------------|
| 约束方式 | 硬剪裁 | 软惩罚 |
| 超参数 | $\epsilon$ | $\beta, \delta_{KL}$ |
| 稳定性 | 更稳定 | 对 $\beta$ 敏感 |
| 适用场景 | 通用 | 需要细粒度控制 |

---

## 四、PPO vs TRPO vs A2C对比

### 4.1 算法对比表

| 特性 | A2C | TRPO | PPO |
|:-----|:----|:-----|:----|
| **优化方式** | 异步/同步梯度 | 约束优化 | 一阶+剪裁 |
| **策略约束** | 无（熵正则化） | KL散度约束 | 概率比剪裁 |
| **计算效率** | 高 | 中 | 高 |
| **内存开销** | 低 | 中（二阶信息） | 低 |
| **实现难度** | 低 | 高 | 中 |
| **样本效率** | 中 | 高 | 高 |
| **收敛稳定性** | 中 | 高 | 高 |
| **大规模应用** | 一般 | 一般 | 首选 |

### 4.2 为什么PPO成为主流？

> [!tip]+ PPO的成功因素
> 1. **实现简单**：只需一阶优化，无需复杂二阶计算
> 2. **超参数鲁棒**：$\epsilon$ 不需要精细调整
> 3. **通用性好**：在离散和连续动作空间都表现优异
> 4. **与现有框架兼容**：易于与深度学习框架集成
> 5. **样本效率**：PPO 2在样本效率上有显著提升

### 4.3 PPO2与向量化环境

PPO2利用向量化环境提高样本效率：

```python
import gym
from stable_baselines3.common.vec_env import DummyVecEnv, SubprocVecEnv

def make_env(env_id):
    """Create environment factory."""
    def _init():
        env = gym.make(env_id)
        return env
    return _init

# 创建向量化环境
def create_vectorized_env(env_id, n_envs=8):
    """
    Create vectorized environments for parallel data collection.
    PPO2 style.
    """
    env = SubprocVecEnv([make_env(env_id) for _ in range(n_envs)])
    return env

class PPO2Agent:
    """
    PPO2 with vectorized environments.
    """
    def __init__(self, env_fn, n_steps=128, n_epochs=10, nminibatches=4):
        self.env = env_fn()
        self.n_steps = n_steps  # Steps per update
        self.n_epochs = n_epochs
        self.nminibatches = nminibatches
        
        # Calculate batch/minibatch sizes
        self.n_envs = self.env.num_envs
        self.batch_size = self.n_steps * self.n_envs
        self.minibatch_size = self.batch_size // nminibatches
    
    def collect_rollouts(self):
        """Collect n_steps of data from all environments."""
        observations = []
        actions = []
        rewards = []
        dones = []
        values = []
        log_probs = []
        
        obs = self.env.reset()
        
        for _ in range(self.n_steps):
            obs_tensor = torch.FloatTensor(obs)
            
            with torch.no_grad():
                action, log_prob = self.policy.get_action(obs_tensor)
                value = self.policy.critic(obs_tensor).squeeze()
            
            action_np = action.numpy()
            
            # Step all environments
            next_obs, reward, done, info = self.env.step(action_np)
            
            observations.append(obs)
            actions.append(action_np)
            rewards.append(reward)
            dones.append(done)
            values.append(value.numpy())
            log_probs.append(log_prob.numpy())
            
            obs = next_obs
        
        return (np.array(observations), np.array(actions), 
                np.array(rewards), np.array(dones),
                np.array(values), np.array(log_probs))
```

---

## 五、实战调参经验

### 5.1 超参数推荐值

> [!tip]+ PPO超参数调优指南
>
> | 参数 | 推荐范围 | 调整建议 |
> |:-----|:---------|:---------|
> | 学习率 | 3e-4 | 可使用线性衰减 |
> | Clip epsilon | 0.1 - 0.3 | 默认0.2效果良好 |
> | GAE lambda | 0.9 - 0.99 | 高值更平滑 |
> | PPO epochs | 10 - 30 | 10通常足够 |
> | Mini-batch size | 32 - 256 | 64/128常用 |
> | 价值系数 | 0.5 - 1.0 | 0.5是标准 |
> | 熵系数 | 0 - 0.01 | 鼓励探索时加 |
> | 梯度裁剪 | 0.5 - 1.0 | 默认0.5 |
> | 环境数量 | 1 - 16 | 根据CPU调整 |

### 5.2 调试技巧

```python
class PPODebugger:
    """Utilities for debugging PPO training."""
    
    @staticmethod
    def check_ratio_distribution(ratios):
        """检查策略比值分布."""
        ratios = np.array(ratios)
        
        print(f"Ratio Stats:")
        print(f"  Mean: {ratios.mean():.3f}")
        print(f"  Std: {ratios.std():.3f}")
        print(f"  Min: {ratios.min():.3f}")
        print(f"  Max: {ratios.max():.3f}")
        print(f"  Clipped (>1+ε): {(ratios > 1.2).mean()*100:.1f}%")
        print(f"  Clipped (<1-ε): {(ratios < 0.8).mean()*100:.1f}%")
        
        return ratios
    
    @staticmethod
    def check_gradient_norm(model):
        """检查梯度范数."""
        total_norm = 0
        for p in model.parameters():
            if p.grad is not None:
                param_norm = p.grad.data.norm(2)
                total_norm += param_norm.item() ** 2
        total_norm = total_norm ** 0.5
        return total_norm
    
    @staticmethod
    def check_value_predictions(values, returns):
        """检查价值预测质量."""
        values = np.array(values)
        returns = np.array(returns)
        
        explained_var = 1 - np.var(returns - values) / np.var(returns)
        
        print(f"Value Function Stats:")
        print(f"  Explained Variance: {explained_var:.3f}")
        print(f"  Value Mean: {values.mean():.3f}")
        print(f"  Returns Mean: {returns.mean():.3f}")
        
        return explained_var
```

### 5.3 常见问题与解决

| 问题 | 原因 | 解决方案 |
|:-----|:-----|:---------|
| 策略崩溃 | 更新过大 | 减小学习率或 $\epsilon$ |
| 价值函数不稳定 | 价值预测噪声大 | 增加价值系数，减小batch size |
| 探索不足 | 熵系数太小 | 增加熵系数 |
| 训练发散 | 梯度爆炸 | 增加梯度裁剪，检查reward缩放 |
| 回报不提升 | 环境或reward设计问题 | 简化环境，调试reward |

---

## 六、数学形式化总结

### PPO剪裁目标函数

$$
\boxed{L^{CLIP}(\theta) = \mathbb{E}_t \left[ \min\left( r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_t \right) \right]}
$$

### 自适应KL惩罚目标

$$
\boxed{L^{KLPEN}(\theta) = \mathbb{E}_t \left[ r_t(\theta)\hat{A}_t - \beta D_{KL}(\pi_{\theta_{old}} || \pi_\theta) \right]}
$$

### PPO完整目标

$$
\boxed{L^{PPO}(\theta) = L^{CLIP}(\theta) - c_1 L^{VF}(\theta) + c_2 S[\pi_\theta](s)}
$$

其中 $c_1$ 是价值系数，$c_2$ 是熵系数，$S$ 是策略熵。

---

## 七、相关文档

- [[../MDP与Bellman方程/MDP与Bellman方程详解|MDP与Bellman方程]] — 理论基础
- [[../Q学习/Q学习深度指南|Q学习]] — 值函数方法
- [[../DQN与变体/DQN深度指南|DQN]] — 深度值函数方法
- [[../策略梯度/策略梯度方法详解|策略梯度方法]] — 策略梯度基础
- [[../多智能体强化学习/多智能体RL详解|多智能体RL]] — 多智能体扩展
- [[../强化学习应用/RL应用场景|RL应用场景]] — 实际应用案例

---

## 参考文献

1. Schulman, J., Levine, S., Abbeel, P., Jordan, M., & Moritz, P. (2015). Trust region policy optimization. *ICML*, 1889-1897.
2. Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal policy optimization algorithms. *arXiv:1707.06347*.
3. Engstrom, L., et al. (2020). Implementation matters in RL: A case study on PPO. *ICLR 2020*.
4. Andrychowicz, M., et al. (2020). What matters in on-policy reinforcement learning? A large-scale empirical study. *arXiv:2006.05990*.
5. Raffin, A., et al. (2021). SB3: Stable Baselines3. *JMLR*.

---
*PPO以其简洁的实现和稳定的性能，成为当前强化学习研究和应用的首选算法。*
