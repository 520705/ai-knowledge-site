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

现实世界中的决策问题往往不是一个人单打独斗能搞定的。想象一下：仓库里的多个机器人在协同搬运货物，城市里的自动驾驶汽车需要互相避让，足球场上两个队伍在激烈对抗，甚至是金融市场里无数交易者在博弈。这些场景都有一个共同点——多个决策者同时存在，每个人的行为都会影响其他人，同时也被其他人影响。

这就是多智能体强化学习（Multi-Agent Reinforcement Learning，简称MARL）要研究的问题。

说起来，多智能体RL其实没有那么神秘。咱们可以把单智能体RL看成一个人独自在迷宫里找出口，而多智能体RL就像是多个人一起在迷宫里摸索，但问题在于——你不仅要知道怎么走出去，还得猜到别人会怎么走，同时还得提防别人故意给你使绊子。

这个难度，可不是简单的人数相加那么简单。

> [!note]+ 单智能体 vs 多智能体
> **核心差异：**
> 1. **环境非平稳性**：其他智能体的策略变化导致环境动态变化
> 2. **信用分配问题**：团队奖励如何归因于个体贡献
> 3. **收敛困难**：Nash均衡的寻找比单智能体最优困难得多
> 4. **计算复杂度**：状态-动作空间指数爆炸

### 为什么多个智能体比一个更难？

这个问题值得好好唠唠。在单智能体RL里，环境虽然可能是随机的，但它至少是"稳定"的——环境的转移概率只跟你自己的动作有关，不会因为你改变策略就发生变化。

但多智能体环境下，这事儿就变了。

举个例子。假设你在学下棋，单智能体RL就像是你一个人在那复盘，棋局规则是固定的，不会因为你今天想换个走法，棋盘就突然变了规则。但多智能体RL呢？就像是你的对手也在学，而且他学得比你快——你今天刚研究出一个新招，明天对手可能就破解了，后天你又得想新办法。这就好比你在一艘船上划船，结果这船不只是你在划，旁边还有好几个人也在划，而且大家还在互相较劲。

这就是所谓的**非平稳性问题**——从单个智能体的视角看，环境变得不可预测了，因为其他智能体的策略在不断变化。

还有一个头疼的问题是**信用分配**。在很多合作场景里，大家一起干活，但最后拿到的奖励是所有人的。这时候怎么知道谁贡献大谁贡献小？干多干少一个样，时间长了谁还愿意多干？

这些问题加起来，让多智能体RL成了一个相当有挑战性的研究领域。

---

## 二、博弈论视角：理解智能体之间的交互

### 2.1 博弈论框架

既然多个智能体之间存在复杂的交互，那就得有个数学框架来描述这种交互。博弈论就是干这个的。

一个博弈可以用下面的公式来描述：

$$
\Gamma = (\mathcal{N}, \mathcal{S}, \{\mathcal{A}_i\}, \{\mathcal{R}_i\}, \mathcal{P}, \gamma)
$$

别被公式吓到，咱们逐个拆解：

- $\mathcal{N} = \{1, 2, ..., n\}$：智能体集合，就是有几个玩家
- $\mathcal{S}$：联合状态空间，表示所有可能的游戏状态
- $\mathcal{A}_i$：智能体 $i$ 的动作空间，它能做什么
- $\mathcal{R}_i$：智能体 $i$ 的奖励函数，做得好有糖吃
- $\mathcal{P}$：状态转移函数，环境怎么变化
- $\gamma$：折扣因子，看重眼前还是未来

这个框架看起来跟单智能体RL差不多，但关键区别在于每个智能体都有自己的奖励函数，而且大家的奖励可能不一样。

### 2.2 博弈类型

根据智能体之间的关系，博弈可以分为几种类型：

| 博弈类型 | 特点 | 示例 |
|:---------|:-----|:-----|
| **合作博弈** | 智能体共享同一目标 | 机器人协作搬运 |
| **竞争博弈** | 零和或常和 | 棋类游戏、二人博弈 |
| **混合博弈** | 合作与竞争并存 | 多方商业竞争 |
| **协调博弈** | 需要选择一致行动 | 路口同时通行 |

#### 合作博弈

合作博弈最好理解，就是大家一条心，朝着同一个目标使劲。比如两个机器人一起搬一个大箱子，两个人一起划船，都属于这种。大家的奖励函数是一样的，或者至少是高度相关的——你赢我也赢，你输我也输。

#### 竞争博弈

竞争博弈就刺激了，最典型的就是零和博弈。什么是零和？简单说就是蛋糕就这么大，你吃多了我就得少吃。最典型的例子就是下棋——你赢了对面就是输了，你输了对面就是赢了。这种场景下，智能体的利益是直接对立的。

#### 混合博弈

混合博弈更贴近现实。你跟同事既有合作关系（大家要一起完成项目），又有竞争关系（年终奖就那么多，谁表现好谁拿得多）。这种合作与竞争并存的场景，是最难处理的。

#### 协调博弈

协调博弈听起来简单，但门道不少。咱们来具体看看：

> [!info]+ 协调博弈示例
> 两个智能体需要同时选择行动 $(A, A)$ 或 $(B, B)$：
> - $(A,A)$：奖励 (10, 10)
> - $(B,B)$：奖励 (8, 8)
> - $(A,B)$ 或 $(B,A)$：奖励 (0, 0)
> 
> 问题：智能体如何协调选择？纯纳什均衡有两个：$(A,A)$ 和 $(B,B)$。

这个例子有意思的地方在于，两个智能体其实都更想选A（10 > 8），但问题是——大家怎么协调到A上去？如果两个智能体是分开训练的，谁都不知道对方会选什么，那就很尴尬了。选A风险很大，万一对方选B，你就什么都得不到；选B虽然奖励少点，但至少保底。

### 2.3 Nash均衡：多智能体的"共识"

说到多智能体RL，就不得不提Nash均衡。这个概念由数学家约翰·纳什提出，是博弈论的核心。

**Nash均衡**的定义其实挺直白的：对于策略组合 $(\pi_1^*, \pi_2^*, ..., \pi_n^*)$，如果没有任何一个智能体能通过单方面改变自己的策略来获得更高回报，那这个策略组合就是Nash均衡。

用人话说就是：谁都没有动力单飞，因为单飞了反而会更糟。

形式化一点，对于智能体 $i$ 的策略 $\pi_i^*$：

$$
\mathbb{E}[r_i | \pi_i^*, \pi_{-i}^*] \geq \mathbb{E}[r_i | \pi_i, \pi_{-i}^*], \quad \forall \pi_i
$$

这个公式的意思是：在其他人都保持策略 $\pi_{-i}^*$ 不变的情况下，智能体 $i$ 换成任何其他策略 $\pi_i$，期望回报都不会更高。

Nash均衡有几种类型：

1. **纯策略Nash均衡**：每个智能体都选一个确定的动作，不会随机
2. **混合策略Nash均衡**：每个智能体以一定概率选择不同的动作
3. **相关均衡（Correlated Equilibrium）**：允许策略之间有关联，比如我可以相信你会选择A，所以我选B

Nash均衡在MARL中扮演什么角色呢？它是多智能体学习的一个重要目标——找到一个大家都满意的稳定状态，谁都没有动力去打破它。

---

## 三、Nash Q-Learning：如何在零和博弈中找到均衡

### 3.1 算法原理

Nash Q-Learning（Littman, 2001）是最早的多智能体Q学习算法之一，专门为零和博弈设计。核心思想是在每个状态计算Nash均衡Q值。

对于零和博弈，每个状态 $s$ 的Q值可以构成一个双人矩阵博弈：

$$
Q(s) = \begin{bmatrix} Q_{11}(s) & Q_{12}(s) \\ Q_{21}(s) & Q_{22}(s) \end{bmatrix}
$$

这里的 $Q_{ij}(s)$ 是智能体1选动作 $i$，智能体2选动作 $j$ 时的Q值。零和博弈意味着智能体1的收益就是智能体2的损失，所以整个博弈可以用一个支付矩阵来描述。

Nash Q-Learning的做法是：对于每个状态，先求出该博弈的Nash均衡策略 $(\pi_1^*, \pi_2^*)$，然后计算均衡Q值：

$$
NashQ(s) = \pi_1^T \cdot Q(s) \cdot \pi_2
$$

然后用这个NashQ值来更新Q表，就像标准的Q学习一样。

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

Nash Q-Learning虽然开创性地将博弈论引入多智能体学习，但它有不少局限：

> [!warning]+ Nash Q-Learning的局限
> 1. **零和假设**：仅适用于完全对抗场景，大多数实际问题不是严格的零和博弈
> 2. **计算复杂度**：Nash均衡求解是PPAD-complete问题，计算量很大
> 3. **扩展性差**：主要针对两个智能体设计，难以扩展到多个智能体
> 4. **一般和博弈**：不适用，现实中的博弈往往涉及合作与竞争的混合

---

## 四、竞争场景：Minimax-Q学习

说到竞争场景，就不能不提Minimax-Q学习。它是Nash Q-Learning在零和博弈中的一个特例，思路更直接——每个智能体都想最大化自己的收益，同时假设对手会最小化自己的收益。

Minimax-Q的核心就是求解一个minimax问题：

$$
\pi_i^*(s) = \arg\max_{\pi_i} \min_{\pi_{-i}} \mathbb{E}[Q(s, a_i, a_{-i})]
$$

简单说就是：我选一个策略，让最坏情况下的收益最大化。这是一种保守但稳健的策略，特别适合对抗性场景。

---

## 五、合作场景：Team-Q和协调Q学习

### 5.1 Team-Q学习

在纯合作场景下，所有智能体共享同一个目标函数。Team-Q学习假设所有智能体是一个"团队"，大家的利益完全一致。

这时候，每个智能体可以直接学习联合动作值函数 $Q(\boldsymbol{\tau}, \boldsymbol{a})$，其中 $\boldsymbol{\tau}$ 是所有智能体的观测历史，$\boldsymbol{a}$ 是联合动作。但问题是联合动作空间随智能体数量指数增长，所以Team-Q通常只适用于少量智能体。

### 5.2 协调Q学习

协调Q学习是专门为协调博弈设计的。它的核心挑战是：智能体之间需要达成一致的行动，但可能没有通信手段。

一个简单的办法是使用"协调技巧"——比如在某个状态约定大家都选同一个动作（因为协调博弈往往有多个均衡，但选择相同的动作是共同知识）。

---

## 六、独立Q学习IQL：最简单但最常用的方法

### 6.1 基本思想

独立Q学习（Independent Q-Learning，IQL）是最直接的多智能体RL方法——每个智能体独立运行标准的Q学习算法，假装其他智能体不存在。

听起来很不靠谱对吧？但实际上IQL是应用最广泛的方法之一。为什么？因为它足够简单，而且在很多场景下效果还不错。

```python
class IndependentQLearning:
    """
    Independent Q-Learning: each agent learns independently.
    """
    def __init__(self, obs_dims, n_actions, lr=0.1, gamma=0.99):
        self.n_agents = len(obs_dims)
        
        # 每个智能体独立的Q网络
        self.q_networks = nn.ModuleList([
            QNetwork(obs_dim, n_actions, hidden_dim=64)
            for obs_dim in obs_dims
        ])
    
    def update(self, agent_id, obs, action, reward, next_obs, done):
        """
        Update a single agent's Q network.
        """
        q_current = self.q_networks[agent_id](obs)
        q_next = self.q_networks[agent_id](next_obs).max()
        
        # 标准TD更新
        target = reward + self.gamma * q_next * (1 - done)
        loss = (q_current[action] - target) ** 2
        
        loss.backward()
        self.optimizers[agent_id].step()
```

### 6.2 为什么IQL还能work？

你可能会问：IQL完全忽视了其他智能体的存在，这不是睁眼瞎吗？

关键在于：虽然其他智能体的策略在变化，但从单个智能体的视角看，这些变化可以被纳入"环境噪声"。如果环境变化得不太快，Q学习还是能学到一些有用的东西。

而且IQL有一个隐含的假设：**其他智能体的策略相对稳定**。在很多实际场景中，虽然不是完全稳定，但也不会变化得太剧烈。

### 6.3 IQL的问题

当然，IQL的问题也很明显：

1. **非平稳性**：其他智能体的策略变化导致Q值不再可靠
2. **不收敛**：理论上，IQL在多智能体环境下几乎不会收敛
3. **自私行为**：智能体可能学会"剥削"其他智能体的当前策略，而不是达到真正的均衡

尽管如此，由于其简单性，IQL经常被用作baseline，跟其他更复杂的方法做对比。

---

## 七、值函数分解：VDN、QMIX

### 7.1 为什么需要值分解？

在合作场景下，所有智能体共享一个团队奖励 $r_{team}$。这时候的问题在于：每个智能体只知道团队干得好不好，但不知道自己的贡献有多大。

一个 naive 的做法是直接学习联合Q值 $Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a})$，但这面临两个问题：

1. **维度灾难**：联合状态-动作空间太大了
2. **泛化问题**：学到的联合Q值难以泛化到新的智能体组合

值分解（Value Decomposition）的基本思想是：**联合Q值虽然不能直接分解，但可以用一种可分解的形式来近似**。

### 7.2 VDN：简单但有效的线性分解

VDN（Value Decomposition Networks，Sunehag et al., 2018）假设联合Q值可以分解为个体Q值的和：

$$
Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a}) = \sum_{i=1}^{n} Q_i(\tau_i, a_i)
$$

这个假设的好处是：
- 每个智能体只需要维护自己的Q网络 $Q_i$
- 执行时只需要每个智能体根据自己的Q值选动作
- 训练时通过求和得到联合Q值

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

VDN的优点是简单、理论上有保证。但它的表达能力有限——联合Q值必须是简单的加和，无法捕捉智能体之间复杂的交互。

### 7.3 QMIX：非线性分解的威力

QMIX（Rashid et al., 2018）采用更复杂的分解方式：

$$
Q_{tot}(\boldsymbol{\tau}, \boldsymbol{a}) = f_{mix}\left(Q_1(\tau_1, a_1), ..., Q_n(\tau_n, a_n)\right)
$$

关键在于对混合网络 $f_{mix}$ 加了一个约束：**单调性**。

$$
\frac{\partial Q_{tot}}{\partial Q_i} \geq 0, \forall i
$$

这个约束的意义是：保证个体Q值的增大总是对应着联合Q值的增大。这允许更复杂的非线性分解，同时保持了执行时的去中心化特性——每个智能体还是只需要看自己的Q值。

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

### 7.4 VDN vs QMIX对比

| 特性 | VDN | QMIX |
|:-----|:----|:-----|
| **分解形式** | 线性求和 | 非线性单调混合 |
| **表达能力** | 有限 | 更强 |
| **学习复杂度** | 低 | 中 |
| **收敛保证** | 理论保证 | 经验更好 |
| **适用场景** | 简单合作 | 复杂合作 |

简单来说，VDN是"够用就好"的方案，QMIX是"性能更强"的方案。具体选哪个，要看任务的复杂度。

---

## 八、COMA算法：反事实基线解决信用分配

### 8.1 信用分配问题的本质

在合作场景里，信用分配是个老大难问题。大家一起干活，拿到的奖励是团队的，但谁干得多谁干得少？

举个例子：两个机器人一起完成一个任务，任务完成后得到奖励+10。但机器人A可能做了99%的工作，机器人B只是在旁边看了一眼。最后奖励平分，每人+5。

这对机器人A公平吗？显然不公平。但问题是：从全局奖励来看，你怎么知道谁贡献大？

### 8.2 COMA的反事实基线

COMA（Counterfactual Multi-Agent，Foerster等，2018）用了一个很聪明的思路：**反事实**。

反事实的意思是：假设其他智能体不动，只有我换一个动作，结果会怎样？

对于智能体 $i$，COMA计算：

$$
c_i(a_i, a_{-i}) = Q_{tot}(\boldsymbol{a}) - \sum_{a_i'} \pi_i(a_i'|o_i) \cdot Q_{tot}(a_i', a_{-i})
$$

这个公式的物理意义是：
- $Q_{tot}(\boldsymbol{a})$：当前联合动作的Q值
- $\sum_{a_i'} \pi_i(a_i'|o_i) \cdot Q_{tot}(a_i', a_{-i})$：如果智能体$i$换成其他动作，用当前策略加权的期望Q值

两者相减，就是"如果智能体$i$换动作，能多得多少"。这就是智能体$i$的**优势函数**。

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

COMA的核心思想是：**通过反事实来估算个体贡献**。这比简单的值分解更精细，但计算代价也更高。

---

## 九、通信与协作：智能体之间如何传递信息

### 9.1 为什么需要通信？

在很多场景下，智能体之间可以通信——通过某种通道交换信息。这能大大提升协作能力。

比如在《狼人杀》里，玩家之间需要互相交流信息；在编队飞行中，飞机之间需要共享位置信息；在机器人协作中，机器人可能需要告知彼此自己的状态。

通信学习（Learning to Communicate）是MARL的一个重要研究方向，核心问题是：**智能体应该通信什么？以什么格式通信？如何学习最优的通信协议？**

### 9.2 RIAL和DIAL

RIAL（Reinforced Inter-Agent Learning）和DIAL（Differentiable Inter-Agent Learning）是两个经典的通信学习方法。

RIAL的做法是：
- 把通信动作当成普通动作来学习
- 每个智能体有一个独立的通信Q网络
- 通信是离散的，有限带宽

DIAL的创新在于：
- 通信梯度可以通过通道传递
- 这允许端到端地学习通信协议
- 通信动作可以被差异化，允许更精细的控制

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

### 9.3 硬编码通信协议

除了学习通信协议，有时候用硬编码的通信规则反而更稳定：

| 协议类型 | 描述 | 适用场景 |
|:---------|:-----|:---------|
| **立即通信** | 每步发送当前观测 | 低带宽需求 |
| **意图通信** | 发送未来行动计划 | 需要协调的场景 |
| **重要性加权** | 根据观测重要性加权通信 | 信息不对称环境 |
| **延迟通信** | 异步通信，更新率低 | 通信受限场景 |

---

## 十、自我博弈：AlphaGo-style的自我对弈学习

### 10.1 自我博弈的思想

自我博弈（Self-Play）是让智能体跟"自己"对弈。这听起来有点奇怪——自己打自己能学到什么？

关键在于：**随着学习进行，智能体要跟自己越来越强的版本对弈**。

打个比方：你在练习下棋，最好的方法不是找一本棋谱死记硬背，而是不断跟更强的对手下。问题是，如果你是初学者，你去哪找愿意跟你下的强手？自我博弈解决了这个问题——让AI成为自己的陪练。

### 10.2 FCP：循环策略训练

最简单的自我博弈是FCP（Fictitious Co-Play）——维护两个策略，一个是当前正在训练的策略，另一个是对手策略。随着训练进行，对手策略也会更新。

```python
class SelfPlay:
    """
    Self-play training for competitive scenarios.
    """
    def __init__(self, agent_class, *args):
        # 当前策略
        self.current = agent_class(*args)
        # 对手策略池
        self.opponent_pool = [agent_class(*args)]
        
    def select_opponent(self):
        """从对手池中随机选一个."""
        return random.choice(self.opponent_pool)
    
    def update_opponent_pool(self):
        """定期更新对手池."""
        self.opponent_pool.append(copy.deepcopy(self.current))
        # 可以选择淘汰差的对手
        if len(self.opponent_pool) > self.max_pool_size:
            self.opponent_pool.pop(0)
    
    def train_step(self):
        """一次训练迭代."""
        opponent = self.select_opponent()
        
        # 当前策略 vs 对手
        episode = self.collect_episode(self.current, opponent)
        self.current.update(episode)
        
        # 定期更新对手池
        if self.should_update_pool():
            self.update_opponent_pool()
```

### 10.3 NFSP：神经虚拟自我博弈

NFSP（Neural Fictitious Self-Play）结合了强化学习和博弈论的思想：
- **平均策略**：基于历史策略的经验分布
- **当前策略**：正在学习的策略
- 每场对局中，以一定概率选择当前策略或平均策略

这模拟了人类棋手的思考方式——既在创新（当前策略），也在总结经验（平均策略）。

---

## 十一、MADDPG：多智能体Actor-Critic

### 11.1 MADDPG的基本思想

MADDPG（Multi-Agent DDPG，Lowe等，2017）将DDPG扩展到多智能体场景，核心思路是**中心化训练去中心化执行（CTDE）**。

- **训练时**：使用全局信息（所有智能体的观测和动作）
- **执行时**：每个智能体只用自己的局部观测

这样既能利用丰富的训练信息，又能在执行时保持去中心化的灵活性。

```python
class MADDPG:
    """
    Multi-Agent DDPG for mixed cooperative-competitive environments.
    """
    def __init__(self, obs_dims, action_dims, n_agents):
        self.n_agents = n_agents
        
        # Critic网络（中心化）
        self.critics = nn.ModuleList([
            CentralizedCritic(obs_dims, action_dims, hidden_dim=64)
            for _ in range(n_agents)
        ])
        
        # Actor网络（去中心化）
        self.actors = nn.ModuleList([
            DecentralizedActor(obs_dims[i], action_dims[i])
            for i in range(n_agents)
        ])
        
        # 目标网络
        self.target_critics = nn.ModuleList([...])
        self.target_actors = nn.ModuleList([...])
    
    def update(self, replay_buffer):
        """MADDPG更新."""
        batch = replay_buffer.sample()
        
        for i in range(self.n_agents):
            # 收集所有智能体的动作
            all_actions = [self.actors[j](batch['obs'][j]) for j in range(self.n_agents)]
            
            # Critic更新
            q_values = self.critics[i](batch['state'], all_actions)
            next_actions = [self.target_actors[j](batch['next_obs'][j]) for j in range(self.n_agents)]
            next_q = self.target_critics[i](batch['next_state'], next_actions)
            
            # ... 标准DDPG更新
```

### 11.2 为什么要中心化Critic？

MADDPG的关键洞察是：**Critic可以中心化，但Actor必须去中心化**。

Critic的作用是估计值函数，评估当前策略的好坏。在训练时，拥有更多信息（全局状态、所有智能体的动作）当然能估计得更准。

Actor负责实际决策，执行时只能看到局部观测。如果Actor依赖全局信息，那就没法在实际环境中用了——因为你不可能在执行时实时获取所有信息。

---

## 十二、MARL的核心挑战

### 12.1 非平稳性问题

在多智能体环境中，环境对于单个智能体而言是**非平稳的**——即使智能体不采取任何行动，环境动态也会因为其他智能体的策略变化而改变。

**问题表现：**
- Q值可能持续振荡
- 策略可能无法收敛
- 智能体可能"过度适应"其他智能体的当前策略

**解决方案：**
1. **CTDE范式**：中心化训练去中心化执行
2. **智能体特定的状态表示**：考虑其他智能体存在的影响
3. **考虑其他智能体意图的建模**：通过建模对手来预测环境变化

### 12.2 信用分配问题

在合作博弈中，团队奖励如何分配给各个智能体是一个核心问题。

> [!example]+ 信用分配问题
> 两个智能体协作完成任务：
> - 智能体A执行了99%的关键工作
> - 智能体B只在最后阶段轻拍一下
> - 两者获得相同的团队奖励
> 
> 问题：智能体A无法学习到自己的贡献价值，可能降低工作积极性。

解决方案包括COMA的反事实基线、值分解方法等。

### 12.3 探索与利用

在单智能体RL中，探索是为了发现更好的策略。但在多智能体环境中，探索还有另一个目的：**探测其他智能体的策略**。

你不仅要探索"什么动作对我有利"，还要探索"对手会对我的动作做出什么反应"。这让探索问题变得更加复杂。

---

## 十三、MARL应用场景

### 13.1 机器人协作

多个机器人协同完成任务，比如：
- 仓库机器人协同搬运
- 无人机编队飞行
- 多机器人地图构建

这类场景通常是合作博弈，挑战在于信用分配和协调。

### 13.2 自动驾驶车队

自动驾驶汽车需要互相协调：
- 车队编队控制
- 交叉路口通行
- 动态路径规划

这类场景涉及合作与竞争的混合——大家要协作避免碰撞，但也有竞争（都想先走）。

### 13.3 游戏AI

游戏是多智能体RL的重要测试平台：
- 星际争霸微操
- Dota 2团战
- 足球游戏
- 多人博弈游戏

OpenAI的Dota 5v5 AI（Five）就是MARL的典型应用，击败了世界冠军队伍。

### 13.4 金融市场

多个交易 agent 在市场中交互：
- 高频交易策略
- 投资组合优化
- 市场操纵检测

这类场景往往是混合博弈，既有合作（套利）也有竞争（抢单）。

---

## 十四、动手实验：PyTorch实现多智能体粒子环境

### 14.1 环境设置

Multi-Agent Particle Environment（MPE）是一个经典的测试平台，由OpenAI开发。我们用它来演示协作任务。

```python
import numpy as np
import torch
import torch.nn as nn
from pettingzoo.mpe import simple_spread_v3

# 创建环境
env = simple_spread_v3.parallel_env(N=3, local_ratio=0, max_cycles=25, seed=42)
observations = env.reset()

# 环境结构
print(f"智能体数量: {len(env.agents)}")
print(f"观测空间: {env.observation_space}")
print(f"动作空间: {env.action_space}")
```

### 14.2 简单的协作策略

实现一个简单的协作策略：每个智能体朝着自己的目标点移动，同时避开其他智能体。

```python
class SimpleCollaborativeAgent:
    """简单的协作智能体"""
    def __init__(self, obs_dim, action_dim):
        self.action_dim = action_dim
        # 简单的策略网络
        self.network = nn.Sequential(
            nn.Linear(obs_dim, 128),
            nn.ReLU(),
            nn.Linear(128, action_dim)
        )
    
    def forward(self, obs):
        """前向传播"""
        return self.network(obs)
    
    def act(self, obs, epsilon=0.1):
        """选择动作"""
        if np.random.random() < epsilon:
            return np.random.randint(self.action_dim)
        
        obs_tensor = torch.FloatTensor(obs).unsqueeze(0)
        with torch.no_grad():
            action = self.network(obs_tensor).argmax().item()
        return action

def run_episode(env, agents, epsilon=0.1):
    """运行一个episode"""
    observations = env.reset()
    total_rewards = {agent: 0 for agent in env.agents}
    
    for step in range(100):  # max_steps
        actions = {}
        for agent_id, obs in observations.items():
            agent = agents[agent_id]
            actions[agent_id] = agent.act(obs, epsilon)
        
        observations, rewards, dones, _ = env.step(actions)
        
        for agent_id, reward in rewards.items():
            total_rewards[agent_id] += reward
        
        if all(dones.values()):
            break
    
    return total_rewards

# 训练循环
agents = {
    agent_id: SimpleCollaborativeAgent(obs_dim, action_dim)
    for agent_id in env.agents
}

for episode in range(1000):
    rewards = run_episode(env, agents, epsilon=0.1)
    avg_reward = np.mean(list(rewards.values()))
    
    if episode % 100 == 0:
        print(f"Episode {episode}: Avg Reward = {avg_reward:.2f}")
```

### 14.3 加入值分解

现在用VDN来改进训练：

```python
class VDNAgent(nn.Module):
    """带VDN的协作智能体"""
    def __init__(self, obs_dim, action_dim, hidden_dim=64):
        super().__init__()
        self.q_network = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, action_dim)
        )
    
    def forward(self, obs):
        return self.q_network(obs)

class VDNAlgorithm:
    """VDN算法实现"""
    def __init__(self, obs_dims, action_dim):
        self.agents = nn.ModuleList([
            VDNAgent(obs_dim, action_dim)
            for obs_dim in obs_dims
        ])
        self.target_agents = nn.ModuleList([
            VDNAgent(obs_dim, action_dim)
            for obs_dim in obs_dims
        ])
        self.optimizer = torch.optim.Adam(self.agents.parameters(), lr=0.001)
        self.gamma = 0.99
    
    def get_q_tot(self, observations):
        """计算联合Q值（VDN求和）"""
        q_values = []
        for i, obs in enumerate(observations):
            q = self.agents[i](obs)
            q_values.append(q)
        # 求和
        q_tot = torch.stack(q_values).sum(dim=0)
        return q_tot
    
    def update(self, batch):
        """VDN更新"""
        observations, actions, rewards, next_obs, dones = batch
        
        # 目标Q值
        with torch.no_grad():
            next_q_tot = self.get_q_tot(next_obs)
            target_q = rewards + self.gamma * next_q_tot * (1 - dones)
        
        # 当前Q值
        current_q = self.get_q_tot(observations)
        current_q = current_q.gather(1, actions.long())
        
        # 损失
        loss = nn.MSELoss()(current_q, target_q)
        
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
        
        return loss.item()
    
    def soft_update(self, tau=0.01):
        """软更新目标网络"""
        for target, source in zip(self.target_agents, self.agents):
            for tp, sp in zip(target.parameters(), source.parameters()):
                tp.data.copy_(tau * sp.data + (1 - tau) * tp.data)
```

### 14.4 运行和改进

运行上述代码，观察协作行为。可以尝试的改进方向：

1. **加入通信**：让智能体学习交换位置信息
2. **调整网络结构**：增加RNN处理历史信息
3. **使用QMIX**：处理更复杂的协调任务
4. **加入注意力机制**：让智能体关注重要的队友

---

## 十五、数学形式化总结

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

## 十六、相关文档

- [[MDP与Bellman方程详解|MDP与Bellman方程]] — 单智能体理论基础
- [[Q学习深度指南|Q学习]] — 单智能体值函数方法
- [[DQN深度指南|DQN]] — 深度值函数方法
- [[策略梯度方法详解|策略梯度方法]] — 策略优化方法
- [[PPO深度指南|PPO]] — 单智能体策略优化
- [[RL应用场景|RL应用场景]] — 应用案例

---

## 参考文献

1. Littman, M. L. (2001). Value-function reinforcement learning in Markov games. *Journal of Cognitive Systems Research*, 2(1), 55-66.
2. Sunehag, P., et al. (2018). Value-decomposition networks for cooperative multi-agent learning. *AAMAS*.
3. Rashid, T., et al. (2018). QMIX: Monotonic value function factorisation for deep multi-agent reinforcement learning. *ICML*.
4. Foerster, J. N., et al. (2018). Counterfactual multi-agent policy gradients. *AAAI*.
5. Lowe, R., et al. (2017). Multi-agent actor-critic for mixed cooperative-competitive environments. *NeurIPS*.
6. Foerster, J. N., et al. (2017). Learning to communicate with deep multi-agent reinforcement learning. *NeurIPS*.

---

## 写在最后

多智能体强化学习是强化学习领域最激动人心的方向之一。它不仅仅是"多个智能体一起学习"那么简单，而是涉及到博弈论、通信理论、协同决策等领域的深度交叉。

从理论上讲，多智能体RL让我们更接近通用人工智能——真实世界中的智能体从来不是孤立的，它们需要跟其他智能体交互、协作、竞争。

从实践上讲，多智能体RL已经应用在游戏AI、机器人协作、自动驾驶、金融市场等众多领域，而且还在不断拓展。

当然，这个领域还有很多未解决的问题：如何在一般博弈中找到均衡？如何更好地进行信用分配？如何处理大规模智能体系统？这些问题的答案，可能会塑造人工智能的未来。

---

*多智能体强化学习是迈向通用人工智能的重要一步，涉及合作、竞争和复杂交互的协调问题。*
