---
title: MDP与Bellman方程详解
date: 2026-04-18
tags:
  - 强化学习
  - 马尔可夫决策过程
  - Bellman方程
  - 动态规划
  - 理论基础
categories:
  - 强化学习
  - 人工智能
alias:
  - Markov Decision Process
  - Bellman Equation
---

# MDP与Bellman方程详解

## 关键词速览

| 核心概念 | 状态空间 | 动作空间 | 转移概率 | 即时奖励 |
|:---------|:---------|:---------|:---------|:---------|
| 策略函数 | 价值函数 | Q函数 | 最优策略 | 累积折扣回报 |

> [!abstract]+ 核心关键词表
> | 术语 | 英文 | 符号 | 说明 |
> |:-----|:-----|:-----|:-----|
> | 马尔可夫决策过程 | MDP | $\mathcal{M}$ | 序列决策问题的数学框架 |
> | 状态 | State | $s \in \mathcal{S}$ | 智能体对环境的感知 |
> | 动作 | Action | $a \in \mathcal{A}$ | 智能体的决策行为 |
> | 奖励 | Reward | $r_t$ | 环境对动作的即时反馈 |
> | 转移概率 | Transition | $P(s'|s,a)$ | 执行动作后的状态转移概率 |
> | 策略 | Policy | $\pi(a\|s)$ | 状态到动作的映射 |
> | 价值函数 | Value Function | $V^\pi(s)$ | 状态的价值评估 |
> | Q函数 | Action-Value | $Q^\pi(s,a)$ | 状态-动作对的价值评估 |
| 折扣因子 | Discount Factor | $\gamma$ | 未来奖励的重要性衰减系数 |
| 最优价值函数 | Optimal Value | $V^*(s)$ | 所有策略中的最大价值函数 |
| 最优Q函数 | Optimal Q | $Q^*(s,a)$ | 最优策略下的Q值 |

---

## 一、马尔可夫决策过程（MDP）定义

马尔可夫决策过程（Markov Decision Process, MDP）是强化学习的理论基础，为序贯决策问题提供了优雅的数学描述框架。MDP的核心假设是**马尔可夫性质**：给定当前状态，未来状态的条件概率分布仅依赖于当前状态，而与历史状态序列无关。这一特性使得MDP能够捕捉环境动态的精髓，同时保持数学处理的简洁性。

MDP由五元组 $\mathcal{M} = (\mathcal{S}, \mathcal{A}, \mathcal{P}, \mathcal{R}, \gamma)$ 完整定义，其中 $\mathcal{S}$ 表示状态空间，$\mathcal{A}$ 表示动作空间，$\mathcal{P}: \mathcal{S} \times \mathcal{A} \times \mathcal{S} \rightarrow [0,1]$ 定义了状态转移概率函数，$\mathcal{R}: \mathcal{S} \times \mathcal{A} \times \mathcal{S} \rightarrow \mathbb{R}$ 是奖励函数，$\gamma \in [0,1]$ 为折扣因子。状态空间可以是离散的有限集合，也可以是连续的无限空间，如机器人的位置坐标。动作空间同样可以是离散的（上下左右移动）或连续的（关节角度、力矩大小）。

### 1.1 状态与状态空间

状态 $s_t$ 是智能体对环境当前情况的完整描述。在理想情况下，状态应包含做出最优决策所需的全部信息，即满足充分性原则。实际应用中，状态可能是原始感知数据（如像素值），也可能是经过特征工程提取的高级特征。状态空间 $\mathcal{S}$ 的选择直接影响学习算法的难度——过于简化的状态空间可能丢失关键信息，而过于复杂的状态空间则会导致维度灾难。

> [!note]+ 状态设计的艺术
> 在[[Q学习]]和[[DQN]]等算法中，状态表示的质量直接决定了智能体的学习效果。良好的状态表示应当满足：
> - **完备性**：包含影响奖励的所有相关信息
> - **紧凑性**：去除冗余和噪声信息
> - **可区分性**：不同状态应对应不同的最优动作

### 1.2 动作与动作空间

动作 $a_t$ 是智能体在给定状态下可以采取的行为。动作空间同样分为离散型和连续型。离散动作空间如游戏中的上下左右移动、围棋中的落子位置；连续动作空间如机械臂关节的角度控制、汽车的转向角和加速度。动作空间的选择涉及控制粒度与计算复杂度的权衡。

### 1.3 奖励机制

奖励函数 $\mathcal{R}$ 是智能体与环境交互的信号，定义了"什么是好的行为"。奖励可以是即时的（每一步获得 $r_t$）或延迟的（仅在任务结束时获得）。奖励设计是强化学习中的关键环节，错误的奖励设计可能导致智能体发现意想不到的"作弊"行为。Shane Legg等人的研究表明，奖励函数设计需要考虑激励相容性（Incentive Compatibility）问题。

---

## 二、策略（Policy）

策略 $\pi$ 是智能体的核心，决定了在每个状态下应采取何种动作。策略可以是确定性的 $\pi: \mathcal{S} \rightarrow \mathcal{A}$，也可以是随机性的 $\pi: \mathcal{S} \times \mathcal{A} \rightarrow [0,1]$。确定性策略给出的是具体的动作选择，而随机策略输出的是各动作的概率分布。

### 2.1 确定性策略

确定性策略 $\pi(s) = a$ 直接指定每个状态对应的最优动作。这种策略形式简单，在实际应用中计算效率高，特别适用于离散动作空间的确定性问题。

### 2.2 随机性策略

随机策略 $\pi(a|s) = \mathbb{P}(a|s)$ 提供了动作选择的概率分布。随机策略在以下场景中尤为重要：

1. **部分可观测环境（POMDP）**：当智能体无法完全观测环境状态时
2. **探索-利用平衡**[[Q学习]]中的ε-greedy策略
3. **连续动作空间**：无法枚举所有动作，必须用概率分布表示

> [!tip] 策略选择原则
> 确定性策略适合确定性环境和需要精确控制的应用；随机策略适合需要探索的复杂环境和连续动作空间问题。

---

## 三、价值函数与Q函数

价值函数是评估策略优劣的核心工具，定义了从长期角度看"什么是好的状态"或"什么是好的状态-动作对"。

### 3.1 累积折扣回报

智能体的目标是最大化累积折扣回报（Cumulative Discounted Return）：

$$
G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
$$

折扣因子 $\gamma \in [0,1]$ 体现了未来奖励的重要性。$\gamma = 0$ 时，智能体只关注即时奖励；$\gamma \rightarrow 1$ 时，智能体更重视长期回报。折扣因子还保证了无限时间 horizon 下回报的有限性。

### 3.2 状态价值函数

状态价值函数 $V^\pi(s)$ 定义为从状态 $s$ 开始，遵循策略 $\pi$ 获得的期望累积折扣回报：

$$
V^\pi(s) = \mathbb{E}_\pi [G_t | S_t = s] = \mathbb{E}_\pi \left[ \sum_{k=0}^{\infty} \gamma^k R_{t+k+1} | S_t = s \right]
$$

$V^\pi(s)$ 回答了"如果我处于状态 $s$，平均而言我能获得多少回报"这一根本问题。

### 3.3 动作价值函数（Q函数）

动作价值函数 $Q^\pi(s,a)$ 定义为从状态 $s$ 执行动作 $a$ 后，遵循策略 $\pi$ 获得的期望累积折扣回报：

$$
Q^\pi(s,a) = \mathbb{E}_\pi [G_t | S_t = s, A_t = a] = \mathbb{E}_\pi \left[ R_{t+1} + \gamma V^\pi(S_{t+1}) | S_t = s, A_t = a \right]
$$

$Q^\pi(s,a)$ 回答了"在状态 $s$ 下采取动作 $a$，之后遵循策略 $\pi$，平均而言能获得多少回报"。

### 3.4 价值函数与Q函数的转换

两种价值函数之间存在明确的数学关系：

$$
V^\pi(s) = \sum_{a \in \mathcal{A}} \pi(a|s) Q^\pi(s,a)
$$

这表明状态价值是所有可能动作价值的加权平均，权重为策略 $\pi$ 选择的概率。

---

## 四、Bellman方程推导

Bellman方程是动态规划的数学基础，揭示了价值函数内部的递归结构。这一方程由Richard Bellman在1950年代提出，是强化学习理论的基石。

### 4.1 Bellman期望方程

将累积回报的定义展开，可以得到价值函数的递归形式——Bellman期望方程：

$$
\begin{aligned}
V^\pi(s) &= \mathbb{E}_\pi [R_{t+1} + \gamma V^\pi(S_{t+1}) | S_t = s] \\
&= \sum_{a \in \mathcal{A}} \pi(a|s) \sum_{s' \in \mathcal{S}} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]
\end{aligned}
$$

类似地，Q函数的Bellman期望方程为：

$$
\begin{aligned}
Q^\pi(s,a) &= \mathbb{E}_\pi [R_{t+1} + \gamma Q^\pi(S_{t+1}, A_{t+1}) | S_t = s, A_t = a] \\
&= \sum_{s' \in \mathcal{S}} P(s'|s,a) \left[ R(s,a,s') + \gamma \sum_{a' \in \mathcal{A}} \pi(a'|s') Q^\pi(s',a') \right]
\end{aligned}
$$

> [!info] Bellman方程的直观理解
> Bellman方程的核心思想是**最优性原理**：整体最优等价于局部最优。具体而言，一个策略在状态 $s$ 下的价值，等于立即获得的奖励加上折扣后的未来价值。这一递归结构使得我们可以用迭代方法求解价值函数。

### 4.2 Bellman最优方程

当目标是寻找最优策略时，价值函数满足Bellman最优方程。对于最优价值函数 $V^*(s)$：

$$
V^*(s) = \max_{a \in \mathcal{A}} \sum_{s' \in \mathcal{S}} P(s'|s,a) \left[ R(s,a,s') + \gamma V^*(s') \right]
$$

对于最优Q函数 $Q^*(s,a)$：

$$
Q^*(s,a) = \sum_{s' \in \mathcal{S}} P(s'|s,a) \left[ R(s,a,s') + \gamma \max_{a' \in \mathcal{A}} Q^*(s',a') \right]
$$

最优价值函数 $V^*$ 和最优Q函数 $Q^*$ 满足以下关系：

$$
V^*(s) = \max_a Q^*(s,a)
$$

---

## 五、最优性原理与最优策略

### 5.1 最优性原理

最优性原理（Principle of Optimality）由Bellman提出，其核心断言是：一个最优策略具有这样的性质——无论初始状态和初始动作如何，接下来的决策必定构成针对新状态的最优策略。这为逆推法（Backward Induction）提供了理论基础。

### 5.2 最优策略的存在性

对于有限状态空间的MDP，最优策略必定存在。对于连续空间，最优策略的存在性需要额外的技术条件。在无限horizon折扣 MDP中，如果状态空间和动作空间都是有限的，则必定存在确定性的最优策略。

### 5.3 最优价值函数的唯一性

虽然最优策略可能不唯一，但最优价值函数 $V^*$ 和最优Q函数 $Q^*$ 是唯一的。任何达到最优价值函数的策略都是最优策略。

---

## 六、MDP求解方法概述

### 6.1 动态规划方法

当环境模型（$P$ 和 $R$）已知时，可以使用经典的动态规划方法：

- **策略迭代（Policy Iteration）**：交替进行策略评估和策略改进
- **价值迭代（Value Iteration）**：直接迭代求解Bellman最优方程

### 6.2 无模型方法

当环境模型未知时，需要使用无模型方法，包括：

- [[Q学习]]：离策略TD学习算法
- [[策略梯度方法]]: 直接优化策略参数
- [[DQN与变体|DQN]]: 深度Q网络

这些方法将在后续章节详细讨论。

> [!example]- 冰湖环境示例
> 在经典的FrozenLake-v0环境中：
> - $\mathcal{S} = \{S, F, H, G\}$（起点、冰面、洞、目标）
> - $\mathcal{A} = \{左, 右, 上, 下\}$
> - $P$ 和 $R$ 定义了状态转移和奖励规则
> - 智能体需要学习到达目标的最优策略

---

## 七、数学形式化总结

### MDP核心公式

$$
\boxed{V^\pi(s) = \sum_{a} \pi(a|s) \sum_{s'} P(s'|s,a)[R(s,a,s') + \gamma V^\pi(s')]}
$$

$$
\boxed{Q^\pi(s,a) = \sum_{s'} P(s'|s,a)[R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s',a')]}
$$

### Bellman最优方程

$$
\boxed{V^*(s) = \max_a \sum_{s'} P(s'|s,a)[R(s,a,s') + \gamma V^*(s')]}
$$

$$
\boxed{Q^*(s,a) = \sum_{s'} P(s'|s,a)[R(s,a,s') + \gamma \max_{a'} Q^*(s',a')]}
$$

---

## 八、相关文档

- [[../Q学习/Q学习深度指南|Q学习算法]] — 无模型强化学习的经典方法
- [[../DQN与变体/DQN深度指南|深度Q网络]] — 结合深度学习的Q学习扩展
- [[../策略梯度/策略梯度方法详解|策略梯度方法]] — 直接优化策略的强化学习方法
- [[../PPO与TRPO/PPO深度指南|PPO算法]] — 主流的策略优化算法
- [[../多智能体强化学习/多智能体RL详解|多智能体RL]] — 多个智能体协同决策
- [[../强化学习应用/RL应用场景|RL应用场景]] — 强化学习的实际应用

---

## 参考文献

1. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
2. Bellman, R. (1957). *Dynamic Programming*. Princeton University Press.
3. Puterman, M. L. (1994). *Markov Decision Processes: Discrete Stochastic Dynamic Programming*. Wiley.
4. Bertsekas, D. P. (2017). *Dynamic Programming and Optimal Control* (Vol. I-II). Athena Scientific.
5. Mnih, V., et al. (2015). Human-level control through deep reinforcement learning. *Nature*, 518(7540), 529-533.

---
*本文档为强化学习系列的核心理论基础，后续所有算法均建立在MDP与Bellman方程框架之上。*
