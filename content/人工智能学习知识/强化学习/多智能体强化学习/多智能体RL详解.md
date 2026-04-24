---
title: 多智能体RL详解
date: 2026-04-18
tags:
  - 强化学习
  - 多智能体
  - 博弈论
  - 协作
  - 竞争
  - QMIX
  - COMA
categories:
  - 强化学习
  - 多智能体系统
alias:
  - Multi-Agent RL
  - Cooperative RL
  - MARL
  - Nash Q-Learning
---

# 多智能体强化学习详解

## 关键词速览

| 核心概念 | 博弈论 | Nash均衡 | 信用分配 | 联合动作值函数 |
|:---------|:-------|:---------|:---------|:--------------|
| 合作 | 竞争 | 混合场景 | 值分解 | 通信协议 |

> [!abstract]+ 核心关键词表
> | 术语 | 英文 | 符号/技术 | 说明 |
> |:-----|:-----|:----------|:-----|
> | 多智能体RL | MARL | $n$ 智能体 | 多个智能体同时学习 |
> | 博弈论 | Game Theory | $\Gamma$ | 策略交互的数学框架 |
> | 零和博弈 | Zero-Sum Game | $\max_i \min_j$ | 完全对抗关系 |
> | 协调博弈 | Coordination Game | - | 需要协调一致行动 |
> | Nash均衡 | Nash Equilibrium | $s^*$ | 无人能单边改进的策略组合 |
> | 联合动作价值 | Joint Q | $Q_{tot}$ | 所有智能体的联合Q值 |
> | 值分解 | Value Decomposition | QMIX/VDN | 分解联合Q函数 |
> | 信用分配 | Credit Assignment | - | 识别个体贡献 |
> | COMA | Counterfactual Multi-Agent | COMA | 使用反事实基线 |
> | 中心化训练 | Centralized Training | CTDE | 训练时使用全局信息 |
> | 去中心化执行 | Decentralized Execution | - | 执行时只用局部观测 |

---

## 一、引言：从单智能体到多智能体

现实世界中的决策问题往往是多智能体参与的：多机器人协同、多人游戏、交通调度、金融市场。多智能体强化学习（Multi-Agent Reinforcement Learning, MARL）研究多个智能体在共享环境中同时学习和决策的问题，面临独特挑战同时也展现出巨大潜力。

> [!note]+ 单智能体 vs 多智能体
> **核心差异：**
> 1. **环境非平稳性**：其他智能体的策略变化导致环境动态变化
> 2. **信用分配问题**：团队奖励如何归因于个体贡献
> 3. **收敛困难**：Nash均衡的寻找比单智能体最优困难得多
> 4. **计算复杂度**：状态-动作空间指数爆炸

---

## 二、博弈论基础

### 2.1 博弈论框架

多智能体交互可以用博弈论框架描述。定义一个**博弈**（Game）$\Gamma$：

$$
\Gamma = (\mathcal{N}, \mathcal{S}, \{\mathcal{A}_i\}, \{\mathcal{R}_i\}, \mathcal{P}, \gamma)
$$

- $\mathcal{N} = \{1, 2, ..., n\}$：智能体集合
- $\mathcal{S}$：联合状态空间
- $\mathcal{A}_i$：智能体 $i$ 的动作空间
- $\mathcal{R}_i$：智能体 $i$ 的奖励函数
- $\mathcal{P}$：状态转移函数
- $\gamma$：折扣因子

### 2.2 博弈类型

| 博弈类型 | 特点 | 示例 |
|:---------|:-----|:-----|
| **合作博弈** | 智能体共享同一目标 | 机器人协作搬运 |
| **竞争博弈** | 零和或常和 | 棋类游戏、二人博弈 |
| **混合博弈** | 合作与竞争并存 | 多方商业竞争 |
| **协调博弈** | 需要选择一致行动 | 路口同时通行 |

> [!info]+ 协调博弈示例
> 两个智能体需要同时选择行动 $(A, A)$ 或 $(B, B)$：
> - $(A,A)$：奖励 (10, 10)
> - $(B,B)$：奖励 (8, 8)
> - $(A,B)$ 或 $(B,A)$：奖励 (0, 0)
> 
> 问题：智能体如何协调选择？纯纳什均衡有两个：$(A,A)$ 和 $(B,B)$。

### 2.3 Nash均衡

**Nash均衡**是博弈论的核心概念：对于策略组合 $(\pi_1^*, \pi_2^*, ..., \pi_n^*)$，如果没有任何智能体能通过单边改变策略获得更高回报，则称该策略组合为Nash均衡。

形式化定义：对于智能体 $i$ 的策略 $\pi_i^*$：
$$
\mathbb{E}[r_i | \pi_i^*, \pi_{-i}^*] \geq \mathbb{E}[r_i | \pi_i, \pi_{-i}^*], \quad \forall \pi_i
$$

**纳什均衡的类型：**
1. **纯策略Nash均衡**：确定性策略的均衡
2. **混合策略Nash均衡**：随机策略的均衡
3. **相关均衡（Correlated Equilibrium）**：允许策略相关性的均衡

---

## 三、Nash Q-Learning

### 3.1 算法原理

Nash Q-Learning（Littman, 2001）是最早的多智能体Q学习算法之一，专为零和博弈设计。核心思想是在每个状态计算Nash均衡Q值：

对于零和博弈，每个状态 $s$ 的Q值构成一个双人矩阵博弈。定义：

$$
Q(s) = \begin{bmatrix} Q_{11}(s) & Q_{12}(s) \\ Q_{21}(s) & Q_{22}(s) \end{bmatrix}
$$

其中 $Q_{ij}(s)$ 是智能体1选动作 $i$，智能体2选动作 $j$ 时的Q值。

Nash Q-Learning通过求解该博弈的Nash均衡策略 $(\pi_1^*, \pi_2^*)$，然后计算均衡Q值：

$$
NashQ(s) = \pi_1^T \cdot Q(s) \cdot \pi_2
$$

### 3.2 实现框架

```python
import numpy as np
from typing import Tuple, List

class NashQLearning:
    """
    Nash Q-Learning for two-player zero-sum games.
    """
    def __init__(self, n_states, n_actions_1, n_actions_2, 
                 alpha=0.1, gamma=0.99):
        self.n_states = n_states
        self.n_actions_1 = n_actions_1
        self.n_actions_2 = n_actions_2
        self.alpha = alpha
        self.gamma = gamma
        
        # Q表: [state, action1, action2]
        self.Q = np.zeros((n_states, n_actions_1, n_actions_2))
    
    def compute_nash_equilibrium(self, Q_matrix):
        """
        Compute Nash equilibrium for zero-sum game.
        Uses linear programming or fictitious play.
        """
        # 简化为贪婪响应（实际应使用更复杂的均衡求解器）
        # 对于零和博弈，可以求解minimax
        best_action_1 = np.argmax(np.max(Q_matrix, axis=2))
        best_action_2 = np.argmax(np.max(Q_matrix, axis=1))
        
        # 返回策略和均衡值
        pi_1 = np.zeros(self.n_actions_1)
        pi_1[best_action_1] = 1.0
        
        pi_2 = np.zeros(self.n_actions_2)
        pi_2[best_action_2] = 1.0
        
        return pi_1, pi_2, Q_matrix[best_action_1, best_action_2]
    
    def update(self, state, action1, action2, reward, next_state):
        """Update Q values."""
        # 下一状态的Nash均衡
        _, _, next_nash_q = self.compute_nash_equilibrium(self.Q[next_state])
        
        # TD目标
        td_target = reward + self.gamma * next_nash_q
        
        # 更新当前状态的Q值
        self.Q[state, action1, action2] += self.alpha * (
            td_target - self.Q[state, action1, action2]
        )
        
        return self.Q[state, action1, action2]
    
    def select_action(self, state, epsilon=0.1):
        """Epsilon-greedy action selection using Nash equilibrium."""
        if np.random.random() < epsilon:
            return np.random.randint(self.n_actions_1)
        
        # 计算当前状态的Nash均衡
        pi_1, _, _ = self.compute_nash_equilibrium(self.Q[state])
        return np.random.choice(self.n_actions_1, p=pi_1)
```

### 3.3 Nash Q-Learning的局限性

> [!warning]+ Nash Q-Learning的局限
> 1. **零和假设**：仅适用于完全对抗场景
> 2. **计算复杂度**：Nash均衡求解是PPAD-complete
> 3. **扩展性差**：难以扩展到多个智能体
> 4. **一般和博弈**：不适用

---

## 四、MARL的核心挑战

### 4.1 非平稳性问题

在多智能体环境中，环境对于单个智能体而言是**非平稳的**——即使智能体不采取任何行动，环境动态也会因为其他智能体的策略变化而改变。这使得基于单智能体假设的强化学习算法失效。

**问题表现：**
- Q值可能持续振荡
- 策略可能无法收敛
- 智能体可能"过度适应"其他智能体的当前策略

**解决方案：**
1. **中心化训练去中心化执行（CTDE）**
2. **智能体特定的状态表示**
3. **考虑其他智能体意图的建模**

### 4.2 信用分配问题

在合作博弈中，团队奖励如何分配给各个智能体是一个核心问题。

> [!example]+ 信用分配问题
> 两个智能体协作完成任务：
> - 智能体A执行了99%的关键工作
> - 智能体B只在最后阶段轻拍一下
> - 两者获得相同的团队奖励
> 
> 问题：智能体A无法学习到自己的贡献价值，可能降低工作积极性。

### 4.3 维度灾难

联合状态-动作空间随智能体数量指数增长：

$$
|\mathcal{S}_{joint}| = |\mathcal{S}_1| \times |\mathcal{S}_2| \times \cdots \times |\mathcal{S}_n|
$$

---

## 五、值分解方法

### 5.1 VDN（Value Decomposition Networks）

VDN（Sunehag et al., 2018）假设联合Q值可以分解为个体Q值的和：

$$
Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a}) = \sum_{i=1}^{n} Q_i(\tau_i, a_i)
$$

其中 $\boldsymbol{\tau} = (\tau_1, ..., \tau_n)$ 是联合历史观测，$\boldsymbol{a} = (a_1, ..., a_n)$ 是联合动作。

```python
class VDN:
    """
    Value Decomposition Networks for cooperative MARL.
    """
    def __init__(self, obs_dims, n_actions, hidden_dim=64):
        self.n_agents = len(obs_dims)
        
        # 每个智能体独立的Q网络
        self.q_networks = nn.ModuleList([
            QNetwork(obs_dims[i], n_actions, hidden_dim)
            for i in range(self.n_agents)
        ])
        
        # 目标网络
        self.target_networks = nn.ModuleList([
            QNetwork(obs_dims[i], n_actions, hidden_dim)
            for i in range(self.n_agents)
        ])
        self.hard_update()
    
    def forward(self, observations):
        """
        Forward pass returning individual Q values.
        observations: list of [batch, obs_dim] tensors
        """
        individual_qs = []
        for i, (obs, net) in enumerate(zip(observations, self.q_networks)):
            q = net(obs)
            individual_qs.append(q)
        return individual_qs
    
    def get_q_tot(self, observations, actions):
        """
        Compute total Q value by summing individual Q values.
        """
        individual_qs = self.forward(observations)
        
        # 收集每个智能体选择动作的Q值
        q_selected = []
        for i, (q, action) in enumerate(zip(individual_qs, actions)):
            q_selected.append(q.gather(1, action.long()))
        
        # 求和得到联合Q值
        q_tot = torch.sum(torch.cat(q_selected, dim=1), dim=1, keepdim=True)
        return q_tot
    
    def update(self, batch):
        """VDN update rule."""
        observations, actions, rewards, next_obs, dones = batch
        
        # 目标Q值（使用目标网络和VDN求和）
        with torch.no_grad():
            next_q_tot = self.target_networks.get_q_tot(next_obs, actions)
            target_q = rewards + self.gamma * next_q_tot * (1 - dones)
        
        # 当前Q值
        current_q = self.q_networks.get_q_tot(observations, actions)
        
        # 损失
        loss = nn.MSELoss()(current_q, target_q)
        
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
        
        return loss.item()
    
    def hard_update(self):
        for target, source in zip(self.target_networks, self.q_networks):
            target.load_state_dict(source.state_dict())
```

### 5.2 QMIX

QMIX（Rashid et al., 2018）使用**单调混合网络**实现值分解，允许更复杂的非线性分解：

$$
Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a}) = f_{mix}\left(Q_1(\tau_1, a_1), ..., Q_n(\tau_n, a_n)\right)
$$

约束是单调性：$\frac{\partial Q_{tot}}{\partial Q_i} \geq 0, \forall i$

```python
class QMixNet(nn.Module):
    """
    QMIX mixing network for value decomposition.
    """
    def __init__(self, n_agents, state_dim, hidden_dim=32):
        super().__init__()
        self.n_agents = n_agents
        self.state_dim = state_dim
        
        # 第一层权重的状态依赖生成
        self.hyper_w1 = nn.Sequential(
            nn.Linear(state_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, n_agents * hidden_dim)
        )
        
        # 第一层偏置的状态依赖生成
        self.hyper_b1 = nn.Linear(state_dim, hidden_dim)
        
        # 第二层权重
        self.hyper_w2 = nn.Sequential(
            nn.Linear(state_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim)
        )
        
        # 第二层偏置
        self.hyper_b2 = nn.Sequential(
            nn.Linear(state_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, 1)
        )
    
    def forward(self, agent_qs, states):
        """
        Mix individual Q values into total Q.
        
        Args:
            agent_qs: [batch, n_agents] individual Q values
            states: [batch, state_dim] state embeddings
        """
        batch_size = agent_qs.size(0)
        
        # 代理Q值变为 [batch, n_agents, 1]
        agent_qs = agent_qs.unsqueeze(-1)
        
        # 计算第一层权重（状态依赖）
        w1 = torch.abs(self.hyper_w1(states))  # 绝对值保证单调性
        w1 = w1.view(batch_size, self.n_agents, -1)  # [batch, n_agents, hidden]
        
        b1 = self.hyper_b1(states).view(batch_size, 1, -1)  # [batch, 1, hidden]
        
        # 第一层：agent_qs -> hidden
        hidden = torch.bmm(agent_qs, w1) + b1
        hidden = torch.relu(hidden)
        
        # 第二层权重
        w2 = torch.abs(self.hyper_w2(states)).view(batch_size, -1, 1)
        b2 = self.hyper_b2(states).view(batch_size, 1, 1)
        
        # 第二层：hidden -> 1
        q_tot = torch.bmm(hidden, w2) + b2
        q_tot = q_tot.view(batch_size, 1)
        
        return q_tot

class QMIX:
    """
    QMIX: Monotonic Value Factorisation for Multi-Agent Cooperation.
    """
    def __init__(self, obs_dims, n_actions, state_dim, 
                 lr=0.0005, gamma=0.99):
        self.n_agents = len(obs_dims)
        
        # Agent networks
        self.agent_networks = nn.ModuleList([
            RNNAgent(obs_dims[i], n_actions, hidden_dim=64)
            for i in range(self.n_agents)
        ])
        
        # Mixing network
        self.mixer = QMixNet(self.n_agents, state_dim, hidden_dim=32)
        
        # Target networks
        self.target_agent_networks = nn.ModuleList([
            RNNAgent(obs_dims[i], n_actions, hidden_dim=64)
            for i in range(self.n_agents)
        ])
        self.target_mixer = QMixNet(self.n_agents, state_dim, hidden_dim=32)
        
        self.hard_update()
    
    def forward(self, observations, states, hidden_states=None, target=False):
        """Forward pass."""
        if target:
            agent_networks = self.target_agent_networks
            mixer = self.target_mixer
        else:
            agent_networks = self.agent_networks
            mixer = self.mixer
        
        # 获取每个智能体的Q值
        agent_qs = []
        new_hidden_states = []
        
        for i, (obs, net) in enumerate(zip(observations, agent_networks)):
            if hidden_states is not None:
                h = hidden_states[i]
            else:
                h = None
            
            q, new_h = net(obs, h)
            agent_qs.append(q)
            new_hidden_states.append(new_h)
        
        # Stack: [batch, n_agents, n_actions]
        agent_qs = torch.stack(agent_qs, dim=1)
        
        # Mix to get total Q
        q_tot = mixer(agent_qs, states)
        
        return q_tot, agent_qs, new_hidden_states
```

### 5.3 VDN vs QMIX对比

| 特性 | VDN | QMIX |
|:-----|:----|:-----|
| **分解形式** | 线性求和 | 非线性单调混合 |
| **表达能力** | 有限 | 更强 |
| **学习复杂度** | 低 | 中 |
| **收敛保证** | 理论保证 | 经验更好 |
| **适用场景** | 简单合作 | 复杂合作 |

---

## 六、COMA算法

### 6.1 反事实基线

COMA（Counterfactual Multi-Agent, Foerster et al., 2018）使用**反事实基线**解决信用分配问题。

核心思想：对于智能体 $i$，计算如果只有它改变动作（其他智能体不动）时的Q值差异：

$$
c_i(a_i, a_{-i}) = Q_{tot}(\boldsymbol{a}) - \sum_{a_i'} \pi_i(a_i'|o_i) \cdot Q_{tot}(a_i', a_{-i})
$$

```python
class COMA:
    """
    Counterfactual Multi-Agent (COMA) algorithm.
    """
    def __init__(self, obs_dims, n_actions, state_dim):
        self.n_agents = len(obs_dims)
        self.n_actions = n_actions
        
        # Critic网络（中心化）
        self.critic = CentralizedCritic(state_dim, self.n_agents * n_actions)
        
        # Actor网络（去中心化）
        self.actors = nn.ModuleList([
            DecentralizedActor(obs_dims[i], n_actions)
            for i in range(self.n_agents)
        ])
        
        # 目标网络
        self.target_critic = CentralizedCritic(state_dim, self.n_agents * n_actions)
    
    def compute_critic(self, state, actions):
        """计算联合Q值."""
        # actions: [batch, n_agents]
        action_indices = actions.sum(dim=1).long()  # 简化为标量
        return self.critic(state, action_indices)
    
    def compute_advantage(self, state, actions, obs):
        """
        计算每个智能体的优势函数（使用反事实基线）.
        """
        batch_size = state.size(0)
        advantages = []
        
        for i in range(self.n_agents):
            # 联合Q值
            q_joint = self.critic(state, self._actions_to_joint(actions))
            
            # 反事实基线：边缘化智能体i的动作
            q_counterfactual = 0
            for a_i in range(self.n_actions):
                counter_actions = actions.clone()
                counter_actions[:, i] = a_i
                q_a_i = self.critic(state, self._actions_to_joint(counter_actions))
                q_counterfactual += self.actors[i].get_prob(obs[i])[:, a_i] * q_a_i
            
            # 优势 = 联合Q - 反事实基线
            advantage = q_joint - q_counterfactual
            advantages.append(advantage)
        
        return advantages
    
    def update(self, batch):
        """COMA更新."""
        states, obs_list, actions, rewards, next_states, next_obs_list = batch
        
        # 计算优势
        advantages = self.compute_advantage(states, actions, obs_list)
        
        # 更新每个智能体的Actor
        for i in range(self.n_agents):
            log_probs = self.actors[i].get_log_prob(obs_list[i], actions[:, i])
            policy_loss = -(log_probs * advantages[i].detach()).mean()
            
            self.actor_optimizers[i].zero_grad()
            policy_loss.backward()
            self.actor_optimizers[i].step()
```

---

## 七、通信与协作机制

### 7.1 通信学习

当智能体可以通过通信通道交换信息时，可以学习更复杂的协作策略。

```python
class CommNet:
    """
    CommNet: Learning to communicate.
    """
    def __init__(self, obs_dim, hidden_dim, n_agents):
        self.n_agents = n_agents
        self.hidden_dim = hidden_dim
        
        # 编码器
        self.encoder = nn.Linear(obs_dim, hidden_dim)
        
        # 通信层
        self.comm_layer = nn.ModuleList([
            CommCell(hidden_dim) for _ in range(3)  # 3层通信
        ])
        
        # 解码器（策略和价值）
        self.decoder = nn.Linear(hidden_dim, hidden_dim)
        self.policy_head = nn.Linear(hidden_dim, n_actions)
        self.value_head = nn.Linear(hidden_dim, 1)
    
    def forward(self, observations, steps=3):
        """
        前向传播，包含通信步骤.
        
        Args:
            observations: [batch, n_agents, obs_dim]
            steps: 通信步数
        """
        batch_size = observations.size(0)
        
        # 初始化隐藏状态
        h = self.encoder(observations)  # [batch, n_agents, hidden]
        
        # 通信步骤
        for step in range(steps):
            # 收集所有智能体的消息
            h_sum = h.sum(dim=1, keepdim=True)  # [batch, 1, hidden]
            
            # 广播给所有智能体（使用均值）
            messages = h_sum.expand(-1, self.n_agents, -1) / self.n_agents
            
            # 更新隐藏状态
            h_new = self.comm_layer[step](h, messages)
            h = h_new
        
        # 解码
        h_decoded = self.decoder(h)
        policy_logits = self.policy_head(h_decoded)
        values = self.value_head(h_decoded)
        
        return policy_logits, values
```

### 7.2 硬编码通信协议

除了学习通信协议，有时使用硬编码的通信规则更稳定：

| 协议类型 | 描述 | 适用场景 |
|:---------|:-----|:---------|
| **立即通信** | 每步发送当前观测 | 低带宽需求 |
| **意图通信** | 发送未来行动计划 | 需要协调的场景 |
| **重要性加权** | 根据观测重要性加权通信 | 信息不对称环境 |
| **延迟通信** | 异步通信，更新率低 | 通信受限场景 |

---

## 八、数学形式化总结

### MARL问题定义

$$
\max_{\theta_1,...,\theta_n} \mathbb{E}_{\boldsymbol{\tau} \sim \pi_{\boldsymbol{\theta}}} \left[ \sum_{t=0}^{T} \gamma^t r_t \right]
$$

### VDN值分解

$$
\boxed{Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a}) = \sum_{i=1}^{n} Q_i(\tau_i, a_i; \theta_i)}
$$

### QMIX单调约束

$$
\boxed{\frac{\partial Q_{tot}}{\partial Q_i} \geq 0, \quad \forall i \in \{1, ..., n\}}
$$

### COMA反事实优势

$$
\boxed{c_i(a_i, a_{-i}) = Q_{tot}(\boldsymbol{a}) - \sum_{a_i'} \pi_i(a_i'|o_i) \cdot Q_{tot}(a_i', a_{-i})}
$$

---

## 九、相关文档

- [[../MDP与Bellman方程/MDP与Bellman方程详解|MDP与Bellman方程]] — 单智能体理论基础
- [[../Q学习/Q学习深度指南|Q学习]] — 单智能体值函数方法
- [[../DQN与变体/DQN深度指南|DQN]] — 深度值函数方法
- [[../策略梯度/策略梯度方法详解|策略梯度方法]] — 策略优化方法
- [[../PPO与TRPO/PPO深度指南|PPO]] — 单智能体策略优化
- [[../强化学习应用/RL应用场景|RL应用场景]] — 应用案例

---

## 参考文献

1. Littman, M. L. (2001). Value-function reinforcement learning in Markov games. *Journal of Cognitive Systems Research*, 2(1), 55-66.
2. Sunehag, P., et al. (2018). Value-decomposition networks for cooperative multi-agent learning. *AAMAS*.
3. Rashid, T., et al. (2018). QMIX: Monotonic value function factorisation for deep multi-agent reinforcement learning. *ICML*.
4. Foerster, J. N., et al. (2018). Counterfactual multi-agent policy gradients. *AAAI*.
5. Lowe, R., et al. (2017). Multi-agent actor-critic for mixed cooperative-competitive environments. *NeurIPS*.
6. Foerster, J. N., et al. (2017). Learning to communicate with deep multi-agent reinforcement learning. *NeurIPS*.

---
*多智能体强化学习是迈向通用人工智能的重要一步，涉及合作、竞争和复杂交互的协调问题。*
