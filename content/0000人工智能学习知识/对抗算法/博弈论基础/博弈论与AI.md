---
title: 博弈论与AI
date: 2026-04-18
tags:
  - 博弈论
  - 人工智能
  - Nash均衡
  - 零和博弈
  - 机制设计
categories:
  - 人工智能
  - 对抗算法
  - 博弈论基础
alias: 博弈论与AI
---

## 关键词

| 术语 | 英文 | 核心概念 |
|------|------|----------|
| 博弈论 | Game Theory | 研究策略交互的数学框架 |
| 参与人 | Player | 博弈中的决策主体 |
| 策略 | Strategy | 参与人可选择的行动集合 |
| 收益函数 | Payoff Function | 策略组合对应的收益数值 |
| 占优策略 | Dominant Strategy | 无论对手如何选择都是最优的策略 |
| Nash均衡 | Nash Equilibrium | 所有参与人都无法单方面获利的状态 |
| 零和博弈 | Zero-Sum Game | 一方所得必为另一方所失的博弈 |
| 混合策略 | Mixed Strategy | 以概率分布选择纯策略 |
| 纳什求解器 | Nash Solver | 计算博弈均衡的算法 |
| 机制设计 | Mechanism Design | 设计激励相容的规则 |

---

## 1. 博弈论概述：策略交互的数学语言

博弈论（Game Theory）是研究多个决策主体之间策略交互的数学框架，其起源可追溯到冯·诺依曼和奥斯卡·摩根斯特恩 1944 年的开创性著作《博弈论与经济行为》。在人工智能领域，博弈论不仅是理解对抗学习（如 GAN）的理论基础，更是设计多智能体系统、强化学习算法和算法机制的核心工具。

> [!note] 博弈论在AI中的地位
> 从 AlphaGo 的蒙特卡洛树搜索到 GAN 的对抗训练，从自动驾驶的决策规划到推荐系统的策略博弈，博弈论提供了一种统一的语言来描述和解决 AI 中的策略交互问题。

---

## 2. 博弈论基础概念

### 2.1 博弈的构成要素

一个标准博弈由以下要素构成：

$$
\Gamma = (N, (A_i)_{i \in N}, (u_i)_{i \in N})
$$

其中：
- $N = \{1, 2, \ldots, n\}$：参与人集合
- $A_i$：参与人 $i$ 的策略空间（可能的行动集合）
- $u_i: A_1 \times A_2 \times \cdots \times A_n \rightarrow \mathbb{R}$：参与人 $i$ 的收益函数

```python
import numpy as np
from itertools import product

class NormalFormGame:
    """
    标准式博弈（Normal Form Game）表示
    
    博弈要素：
    - 参与人集合
    - 每个参与人的策略空间
    - 每个参与人的收益矩阵
    """
    
    def __init__(self, players, strategy_spaces, payoff_matrices):
        """
        参数:
            players: 参与人列表
            strategy_spaces: 各参与人的策略空间
            payoff_matrices: 各参与人的收益矩阵（numpy数组）
        """
        self.players = players
        self.strategy_spaces = strategy_spaces
        self.payoff_matrices = payoff_matrices
    
    def get_payoff(self, action_profile):
        """给定行动剖面，获取各参与人的收益"""
        rewards = []
        for i, player in enumerate(self.players):
            # 根据行动剖面索引收益矩阵
            idx = tuple(action_profile[j] if j != i else slice(None) 
                       for j in range(len(self.players)))
            reward = self.payoff_matrices[i]
            for j, a in enumerate(action_profile):
                if j != i:
                    reward = reward[a]
                else:
                    reward = reward[slice(None)]
            rewards.append(self._get_nested_value(self.payoff_matrices[i], action_profile, i))
        return rewards
    
    def _get_nested_value(self, matrix, action_profile, player_idx):
        """从嵌套矩阵中获取值"""
        idx = [action_profile[j] for j in range(len(action_profile)) if j != player_idx]
        return matrix[tuple(idx)]
    
    def best_response(self, player_idx, opponent_actions):
        """
        计算参与人的最优响应
        
        给定其他参与人的行动，选择能够最大化自身收益的行动
        """
        my_strategy_space = self.strategy_spaces[player_idx]
        best_action = 0
        best_payoff = float('-inf')
        
        for my_action in range(len(my_strategy_space)):
            action_profile = list(opponent_actions) + [my_action]
            payoff = self.get_payoff(action_profile)[player_idx]
            if payoff > best_payoff:
                best_payoff = payoff
                best_action = my_action
        
        return best_action, best_payoff

# 囚徒困境博弈示例
def create_prisoners_dilemma():
    """创建囚徒困境博弈"""
    players = ["囚徒A", "囚徒B"]
    # 0 = 沉默, 1 = 背叛
    strategy_spaces = [["沉默", "背叛"], ["沉默", "背叛"]]
    # 收益矩阵：[坦白者收益, 沉默者收益]
    # 囚徒A的收益矩阵
    payoff_A = np.array([[-1, -4], [0, -3]])  # 行=囚徒A的选择, 列=囚徒B的选择
    # 囚徒B的收益矩阵（对称）
    payoff_B = np.array([[-1, 0], [-4, -3]])
    
    return NormalFormGame(players, strategy_spaces, [payoff_A, payoff_B])
```

### 2.2 博弈的分类

```
博弈分类体系
├── 参与人数量
│   ├── 两人博弈
│   └── n人博弈（n > 2）
├── 收益结构
│   ├── 零和博弈（一方所得为另一方所失）
│   ├── 常和博弈（总收益恒定）
│   └── 变和博弈（总收益可变）
├── 时间维度
│   ├── 静态博弈（同时行动）
│   └── 动态博弈（序贯行动）
├── 信息结构
│   ├── 完全信息博弈
│   └── 不完全信息博弈（贝叶斯博弈）
└── 策略类型
    ├── 纯策略（确定性选择）
    └── 混合策略（概率性选择）
```

---

## 3. 占优策略与Nash均衡

### 3.1 占优策略

占优策略（Dominant Strategy）是指无论其他参与人如何选择，都是最优的策略。

**严格占优策略定义**：对于参与人 $i$，策略 $s_i^* \in S_i$ 是严格占优策略，当且仅当对于任意 $s_i' \in S_i, s_{-i} \in S_{-i}$：

$$
u_i(s_i^*, s_{-i}) > u_i(s_i', s_{-i})
$$

```python
def find_dominant_strategy(game, player_idx):
    """
    查找参与人的占优策略
    
    如果一个策略在所有情况下都优于其他策略，则为占优策略
    """
    num_strategies = len(game.strategy_spaces[player_idx])
    
    for my_action in range(num_strategies):
        is_dominant = True
        
        # 检查该策略是否在所有对手策略下都是最优的
        opponent_strategies = [len(game.strategy_spaces[j]) for j in range(len(game.players)) if j != player_idx]
        
        for opponent_action in range(np.prod(opponent_strategies)):
            opponent_profile = []
            temp = opponent_action
            for num_s in opponent_strategies:
                opponent_profile.append(temp % num_s)
                temp //= num_s
            
            action_profile = opponent_profile.copy()
            action_profile.insert(player_idx, my_action)
            
            my_payoff = game.get_payoff(action_profile)[player_idx]
            
            # 比较与其他策略的收益
            for other_action in range(num_strategies):
                if other_action == my_action:
                    continue
                
                other_profile = opponent_profile.copy()
                other_profile.insert(player_idx, other_action)
                other_payoff = game.get_payoff(other_profile)[player_idx]
                
                if my_payoff <= other_payoff:
                    is_dominant = False
                    break
            
            if not is_dominant:
                break
        
        if is_dominant:
            return my_action, game.strategy_spaces[player_idx][my_action]
    
    return None, None

# 囚徒困境分析
def analyze_prisoners_dilemma():
    """
    囚徒困境分析：
    
    收益矩阵（双方收益）:
                 囚徒B
              沉默    背叛
    囚徒A 沉默  (-1,-1) (-4, 0)
         背叛  ( 0,-4) (-3,-3)
    
    分析：
    - 对囚徒A：背叛总是优于沉默（无论B如何选择）
      - 若B沉默：背叛得0 > 沉默得-1 ✓
      - 若B背叛：背叛得-3 > 沉默得-4 ✓
    - 因此"背叛"是囚徒A的占优策略
    - 同理，"背叛"也是囚徒B的占优策略
    - Nash均衡：(背叛, 背叛) = (-3, -3)
    - 但帕累托最优：(沉默, 沉默) = (-1, -1)
    """
    game = create_prisoners_dilemma()
    
    for i, player in enumerate(game.players):
        action, strategy = find_dominant_strategy(game, i)
        if strategy:
            print(f"{player} 的占优策略: {strategy}")
    
    print("\nNash均衡: (背叛, 背叛) = (-3, -3)")
    print("帕累托最优: (沉默, 沉默) = (-1, -1)")
```

### 3.2 Nash均衡

Nash均衡是博弈论中最重要的均衡概念之一：在这个状态下，每个参与人都无法通过单方面改变策略来提高自己的收益。

**Nash均衡定义**：策略剖面 $(s_1^*, s_2^*, \ldots, s_n^*)$ 是Nash均衡，当且仅当对于每个参与人 $i$ 和任意 $s_i' \in S_i$：

$$
u_i(s_i^*, s_{-i}^*) \geq u_i(s_i', s_{-i}^*)
$$

```python
def find_nash_equilibrium(game):
    """
    暴力搜索法找Nash均衡（适用于小规模博弈）
    
    枚举所有策略组合，检查每个参与人是否无法单方面改进
    """
    strategy_counts = [len(space) for space in game.strategy_spaces]
    equilibria = []
    
    # 枚举所有策略组合
    for action_profile in product(*[range(c) for c in strategy_counts]):
        is_equilibrium = True
        
        # 检查每个参与人是否无法单方面改进
        for player_idx in range(len(game.players)):
            current_payoff = game.get_payoff(action_profile)[player_idx]
            
            # 检查该参与人所有其他策略
            for alternative_action in range(strategy_counts[player_idx]):
                if alternative_action == action_profile[player_idx]:
                    continue
                
                alternative_profile = list(action_profile)
                alternative_profile[player_idx] = alternative_action
                
                alternative_payoff = game.get_payoff(alternative_profile)[player_idx]
                
                # 如果存在更好的单方面偏离，则不是均衡
                if alternative_payoff > current_payoff:
                    is_equilibrium = False
                    break
            
            if not is_equilibrium:
                break
        
        if is_equilibrium:
            equilibria.append(action_profile)
    
    return equilibria

# 石头剪刀布博弈
def create_rock_paper_scissors():
    """创建石头剪刀布博弈"""
    players = ["玩家1", "玩家2"]
    # 0=石头, 1=布, 2=剪刀
    strategy_spaces = [["石头", "布", "剪刀"], ["石头", "布", "剪刀"]]
    
    # 玩家1的收益：赢=1, 平=0, 输=-1
    payoff_1 = np.array([
        [0, -1, 1],   # 玩家1出石头
        [1, 0, -1],   # 玩家1出布
        [-1, 1, 0]    # 玩家1出剪刀
    ])
    
    # 玩家2的收益（相反）
    payoff_2 = np.array([
        [0, 1, -1],
        [-1, 0, 1],
        [1, -1, 0]
    ])
    
    return NormalFormGame(players, strategy_spaces, [payoff_1, payoff_2])

def analyze_rock_paper_scissors():
    """
    石头剪刀布分析：
    
    纯策略Nash均衡：不存在
    - 任何纯策略组合都存在至少一方可以改进
    
    混合策略Nash均衡：存在
    - 三个策略等概率 (1/3, 1/3, 1/3)
    - 这是对称Nash均衡
    """
    game = create_rock_paper_scissors()
    equilibria = find_nash_equilibrium(game)
    print(f"纯策略Nash均衡数量: {len(equilibria)}")
    print("（石头剪刀布没有纯策略均衡）")
```

> [!note] Nash均衡的存在性
> 纳什定理（Nash, 1950）：每个有限博弈至少存在一个Nash均衡（可能是混合策略均衡）。

---

## 4. 零和博弈

### 4.1 零和博弈的定义与性质

零和博弈（Zero-Sum Game）是一类特殊的博弈，其中一方的收益恰好是另一方的损失：

$$
u_1(s_1, s_2) + u_2(s_1, s_2) = 0, \quad \forall s_1 \in S_1, s_2 \in S_2
$$

这意味着在零和博弈中，参与人之间是完全对立的，没有任何合作的可能。

```python
class ZeroSumGame:
    """
    零和博弈特殊处理
    
    零和博弈性质：
    1. 一方的最优策略直接由对方的最优策略决定
    2. 存在鞍点（如果纯策略均衡存在）
    3. minimax定理保证混合策略均衡存在
    """
    
    def __init__(self, payoff_matrix):
        """
        参数:
            payoff_matrix: 行玩家（玩家1）的收益矩阵
                          列玩家（玩家2）的收益为 -payoff_matrix
        """
        self.payoff_matrix = np.array(payoff_matrix)
        self.player1_payoffs = self.payoff_matrix
        self.player2_payoffs = -self.payoff_matrix
    
    def row_maximin(self):
        """
        行玩家（最大化最小收益）的最优响应
        最大化最小值策略
        """
        # 每行的最小值
        row_mins = self.payoff_matrix.min(axis=1)
        # 选择最小值最大的行
        best_row = np.argmax(row_mins)
        return best_row, row_mins[best_row]
    
    def column_minimax(self):
        """
        列玩家（最小化最大损失）的最优响应
        """
        # 每列的最大值
        col_maxs = self.payoff_matrix.max(axis=0)
        # 选择最大值最小的列
        best_col = np.argmin(col_maxs)
        return best_col, col_maxs[best_col]
    
    def find_saddle_point(self):
        """
        查找鞍点（纯策略均衡）
        
        鞍点条件：
        - 行最小值中的最大值 = 列最大值中的最小值
        """
        row_mins = self.payoff_matrix.min(axis=1)
        col_maxs = self.payoff_matrix.max(axis=0)
        
        maximin = row_mins.max()
        minimax = col_maxs.min()
        
        if maximin == minimax:
            # 找到鞍点
            row_idx = np.argmax(row_mins)
            col_idx = np.argmin(col_maxs)
            return True, (row_idx, col_idx), maximin
        else:
            return False, None, None

# 剪刀石头布零和博弈分析
def analyze_rps_zero_sum():
    game = ZeroSumGame(np.array([
        [0, -1, 1],
        [1, 0, -1],
        [-1, 1, 0]
    ]))
    
    has_saddle, point, value = game.find_saddle_point()
    print(f"鞍点存在: {has_saddle}")
    if has_saddle:
        print(f"鞍点位置: {point}")
        print(f"博弈值: {value}")
    else:
        print("无纯策略均衡，需要混合策略")
    
    # 行玩家最大化最小收益
    best_row, maxmin = game.row_maximin()
    # 列玩家最小化最大收益
    best_col, minmax = game.column_minimax()
    
    print(f"\n行玩家maximin: {maxmin}")
    print(f"列玩家minimax: {minmax}")
    print(f"maximin != minimax，证实无纯策略均衡")
```

### 4.2 Minimax定理

冯·诺依曼的Minimax定理是博弈论的基石之一：

> **定理（Minimax Theorem）**：对于任意零和博弈 $\Gamma$，有：
> $$
> \max_{s_1 \in \Delta(S_1)} \min_{s_2 \in \Delta(S_2)} u_1(s_1, s_2) = \min_{s_2 \in \Delta(S_2)} \max_{s_1 \in \Delta(S_1)} u_1(s_1, s_2)
> $$

这一定理保证零和博弈总是存在混合策略均衡。

```python
def fictitious_play(game, num_iterations=10000):
    """
    虚拟对局算法（Fictitious Play）
    
    迭代学习过程：
    1. 每个参与人根据对方的历史平均策略选择最优响应
    2. 更新历史平均
    3. 重复直到收敛
    
    收敛性：在两人零和博弈中，虚拟对局收敛到Nash均衡
    """
    n_rows, n_cols = game.payoff_matrix.shape
    
    # 初始化历史计数
    row_counts = np.ones(n_rows)  # 每个策略被选中的次数
    col_counts = np.ones(n_cols)
    
    row_distribution = row_counts / row_counts.sum()
    col_distribution = col_counts / col_counts.sum()
    
    for iteration in range(num_iterations):
        # 行玩家对列玩家历史策略的最优响应
        expected_payoffs = game.payoff_matrix @ col_distribution
        best_row = np.argmax(expected_payoffs)
        
        # 列玩家对行玩家历史策略的最优响应
        expected_payoffs_col = game.payoff_matrix.T @ row_distribution
        best_col = np.argmin(expected_payoffs_col)
        
        # 更新计数
        row_counts[best_row] += 1
        col_counts[best_col] += 1
        
        # 更新分布
        row_distribution = row_counts / row_counts.sum()
        col_distribution = col_counts / col_counts.sum()
        
        if iteration % 1000 == 0:
            value = row_distribution @ game.payoff_matrix @ col_distribution
            print(f"Iter {iteration}: 博弈值估计 = {value:.4f}")
            print(f"  行玩家策略: {row_distribution}")
            print(f"  列玩家策略: {col_distribution}")
    
    return row_distribution, col_distribution
```

---

## 5. 混合策略纳什均衡

### 5.1 混合策略的定义

混合策略（Mixed Strategy）是指参与人以一定概率分布在多个纯策略之间随机选择：

$$
\sigma_i \in \Delta(S_i) = \left\{ (p_1, p_2, \ldots, p_{|S_i|}) : \sum_{j=1}^{|S_i|} p_j = 1, p_j \geq 0 \right\}
$$

在混合策略均衡中，每个参与人选择各个纯策略的概率使得对方对每个纯策略都无差异（除非某个策略概率为0）。

```python
def find_mixed_nash_equilibrium(game):
    """
    寻找两人博弈的混合策略Nash均衡
    
    对于2x2博弈，可以通过解析方法求解
    """
    payoff_matrix = game.payoff_matrix
    n_rows, n_cols = payoff_matrix.shape
    
    equilibria = []
    
    if n_rows == 2 and n_cols == 2:
        # 2x2博弈混合均衡求解
        # 设行玩家以(p, 1-p)选择策略1和2
        # 设列玩家以(q, 1-q)选择策略1和2
        
        # 行玩家无差异条件
        # payoff[0,q] = payoff[1,q]
        # a*q + b*(1-q) = c*q + d*(1-q)
        # q = (d-b)/(a-b-c+d)
        
        a = payoff_matrix[0, 0]  # (R1, C1)
        b = payoff_matrix[0, 1]  # (R1, C2)
        c = payoff_matrix[1, 0]  # (R2, C1)
        d = payoff_matrix[1, 1]  # (R2, C2)
        
        denom = a - b - c + d
        if abs(denom) > 1e-10:
            q = (d - b) / denom  # 列玩家选择策略1的概率
            if 0 <= q <= 1:
                # 计算行玩家策略
                p = (d - c) / (a - b - c + d) if abs(denom) > 1e-10 else 0.5
                if 0 <= p <= 1:
                    equilibria.append({
                        'player1': [p, 1-p],
                        'player2': [q, 1-q],
                        'value': p * (a * q + b * (1-q)) + (1-p) * (c * q + d * (1-q))
                    })
    
    return equilibria

def analyze_matching_pennies():
    """
    硬币配对博弈（Matching Pennies）
    
    收益矩阵:
                 列玩家
              Head    Tail
    行玩家 Head  1,-1   -1,1
         Tail  -1,1    1,-1
    
    分析：
    - 无纯策略均衡
    - 混合策略均衡：各以0.5概率选择
    """
    game = ZeroSumGame(np.array([
        [1, -1],
        [-1, 1]
    ]))
    
    equilibria = find_mixed_nash_equilibrium(game)
    print("硬币配对博弈混合均衡:")
    for eq in equilibria:
        print(f"  行玩家: Head={eq['player1'][0]:.2f}, Tail={eq['player1'][1]:.2f}")
        print(f"  列玩家: Head={eq['player2'][0]:.2f}, Tail={eq['player2'][1]:.2f}")
        print(f"  博弈值: {eq['value']:.2f}")
```

### 5.2 支持选择与 indifference

在混合策略Nash均衡中，每个被赋予正概率的纯策略（构成支持集）必须给参与人带来相同的期望收益：

$$
u_i(s_i^k, \sigma_{-i}^*) = u_i(s_i^l, \sigma_{-i}^*), \quad \forall s_i^k, s_i^l \in \text{supp}(\sigma_i^*)
$$

```python
def solve_mixed_strategy_lp(game, player_idx):
    """
    使用线性规划求解混合策略Nash均衡
    
    以玩家1为例：
    - 最大化 v（博弈值）
    - 约束：每个策略的期望收益 >= v
    """
    from scipy.optimize import linprog
    import numpy as np
    
    if len(game.players) != 2:
        raise NotImplementedError("仅支持两人博弈")
    
    payoff_matrix = game.payoff_matrix
    n_strategies = payoff_matrix.shape[0] if player_idx == 0 else payoff_matrix.shape[1]
    
    # 线性规划：最大化 v
    # 约束：Ax <= b, x >= 0
    
    if player_idx == 0:
        # 行玩家：minimize -v
        # 约束：p^T * payoff >= v for all columns
        A_ub = -payoff_matrix.T  # 不等式左侧系数
        b_ub = -np.ones(payoff_matrix.shape[1])  # v 的系数（设为1）
        
        # 约束：p.sum() = 1
        A_eq = np.ones((1, n_strategies + 1))
        b_eq = np.array([1.0])
        
        # 变量：[p1, p2, ..., pn, v]
        c = np.zeros(n_strategies + 1)
        c[-1] = -1  # 最大化 v = 最小化 -v
    else:
        # 列玩家：maximize v
        A_ub = payoff_matrix  # 不等式左侧
        b_ub = np.ones(payoff_matrix.shape[0])
        
        A_eq = np.ones((1, n_strategies + 1))
        b_eq = np.array([1.0])
        
        c = np.zeros(n_strategies + 1)
        c[-1] = 1  # 最大化 v
    
    result = linprog(c, A_ub=A_ub, b_ub=b_ub, A_eq=A_eq, b_eq=b_eq,
                     bounds=[(0, 1)] * n_strategies + [(-None, None)])
    
    if result.success:
        strategy = result.x[:-1]
        value = result.x[-1]
        return strategy, value
    else:
        return None, None
```

---

## 6. 博弈论在AI中的应用

### 6.1 GAN的博弈论解释

生成对抗网络（GAN）本质上是一个两人零和博弈：

$$
\min_G \max_D V(D, G) = \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}}[\log D(\mathbf{x})] + \mathbb{E}_{\mathbf{z} \sim p_\mathbf{z}}[\log(1 - D(G(\mathbf{z})))]
$$

```python
class GANGame:
    """
    GAN的博弈论建模
    
    博弈要素：
    - 参与人1：生成器G（最小化）
    - 参与人2：判别器D（最大化）
    - 零和博弈：G的损失 = -D的损失
    """
    
    def __init__(self):
        self.generator_loss_history = []
        self.discriminator_loss_history = []
    
    def compute_gan_objective(self, real_samples, fake_samples, discriminator, generator):
        """
        计算GAN的博弈目标值
        """
        # 判别器试图最大化
        d_real = discriminator(real_samples)
        d_fake = discriminator(fake_samples.detach())
        
        # 二分类交叉熵损失
        d_loss_real = -torch.log(d_real + 1e-8).mean()
        d_loss_fake = -torch.log(1 - d_fake + 1e-8).mean()
        d_loss = (d_loss_real + d_loss_fake) / 2
        
        # 生成器试图最小化（但通过判别器的反向信号）
        g_loss = -torch.log(d_fake + 1e-8).mean()
        
        return g_loss, d_loss
    
    def check_nash_equilibrium(self, generator, discriminator, real_data, latent_dim):
        """
        检查GAN是否接近Nash均衡
        """
        generator.eval()
        discriminator.eval()
        
        with torch.no_grad():
            # 生成样本
            z = torch.randn(100, latent_dim).to(next(generator.parameters()).device)
            fake_samples = generator(z)
            
            # 判别器输出
            d_real = discriminator(real_data[:100]).mean().item()
            d_fake = discriminator(fake_samples).mean().item()
        
        # Nash均衡条件：
        # - D(G(z)) ≈ 0.5（无法区分真假）
        # - G无法进一步改进
        
        equilibrium_gap = abs(d_fake - 0.5)
        
        return {
            'D(real)': d_real,
            'D(fake)': d_fake,
            'equilibrium_gap': equilibrium_gap,
            'is_near_equilibrium': equilibrium_gap < 0.1
        }
```

### 6.2 多智能体强化学习中的博弈论

```python
class MultiAgentGame:
    """
    多智能体博弈框架
    
    应用场景：
    - 多智能体强化学习
    - 分布式优化
    - 联邦学习
    """
    
    def __init__(self, agents, payoff_functions):
        self.agents = agents
        self.payoff_functions = payoff_functions
    
    def best_response_iteration(self, agent_idx, others_strategies):
        """
        最佳响应迭代
        
        给定其他智能体的策略，计算当前智能体的最优策略
        """
        agent = self.agents[agent_idx]
        payoff_fn = self.payoff_functions[agent_idx]
        
        # 离散化策略空间搜索
        best_strategy = None
        best_payoff = float('-inf')
        
        for strategy in agent.strategy_space:
            payoff = payoff_fn(strategy, others_strategies)
            if payoff > best_payoff:
                best_payoff = payoff
                best_strategy = strategy
        
        return best_strategy, best_payoff
    
    def fictitious_play_iteration(self, history):
        """
        虚拟对局迭代
        """
        new_strategies = []
        
        for i, agent in enumerate(self.agents):
            # 计算其他智能体的历史平均策略
            others_histories = [history[j] for j in range(len(self.agents)) if j != i]
            others_avg_strategies = [np.mean(h, axis=0) for h in others_histories]
            
            # 计算最优响应
            best_response, _ = self.best_response_iteration(i, others_avg_strategies)
            new_strategies.append(best_response)
        
        return new_strategies
```

---

## 7. 机制设计基础

### 7.1 机制设计的核心问题

机制设计（Mechanism Design）是博弈论的"反向工程"：不是分析给定规则下的结果，而是设计规则以实现期望的结果。

核心概念：
- **激励相容（Incentive Compatibility）**：说实话是对每个参与人的占优策略
- **社会福利最大化**：$\max_{a \in A} \sum_{i} v_i(a)$
- **个体理性（Individual Rationality）**：参与人有参与的动机

```python
class VickreyAuction:
    """
     Vickrey拍卖（第二价格密封拍卖）
    
    机制设计特点：
    - 激励相容：说实话是占优策略
    - 社会福利最大化
    - 个体理性
    """
    
    def __init__(self, reserve_price=0):
        self.reserve_price = reserve_price
        self.bids = []
    
    def submit_bid(self, bidder_id, bid):
        """提交投标"""
        if bid >= self.reserve_price:
            self.bids.append((bidder_id, bid))
    
    def determine_winner(self):
        """确定胜者和最终价格"""
        if not self.bids:
            return None, None, None
        
        # 按投标排序
        sorted_bids = sorted(self.bids, key=lambda x: x[1], reverse=True)
        
        winner_id = sorted_bids[0][0]
        winner_bid = sorted_bids[0][1]
        
        # 第二高价格（如果有多个投标人）
        if len(sorted_bids) > 1:
            price = sorted_bids[1][1]
        else:
            price = self.reserve_price
        
        return winner_id, price, winner_bid

class DoubleAuction:
    """
    双向拍卖
    
    买卖双方同时报价，机制设计决定交易价格
    """
    
    def __init__(self):
        self.buyer_bids = []  # (buyer_id, bid)
        self.seller_asks = []  # (seller_id, ask)
    
    def submit_buyer_bid(self, buyer_id, bid):
        self.buyer_bids.append((buyer_id, bid))
    
    def submit_seller_ask(self, seller_id, ask):
        self.seller_asks.append((seller_id, ask))
    
    def clear_market(self):
        """清空市场，找到交易对"""
        buyers = sorted(self.buyer_bids, key=lambda x: x[1], reverse=True)
        sellers = sorted(self.seller_asks, key=lambda x: x[1])
        
        trades = []
        i, j = 0, 0
        
        while i < len(buyers) and j < len(sellers):
            buyer_id, buyer_bid = buyers[i]
            seller_id, seller_ask = sellers[j]
            
            if buyer_bid >= seller_ask:
                # 交易可以发生
                price = (buyer_bid + seller_ask) / 2  # 中点价格
                trades.append({
                    'buyer': buyer_id,
                    'seller': seller_id,
                    'price': price
                })
                i += 1
                j += 1
            else:
                break
        
        return trades
```

> [!tip] 激励相容的重要性
> 在机制设计中，激励相容确保参与人有动机按照设计的规则行事。Vickrey拍卖之所以有效，是因为它将"说实话"变成每个投标人的占优策略。

---

## 8. 学术引用与参考文献

1. von Neumann, J., & Morgenstern, O. (1944). "Theory of Games and Economic Behavior." *Princeton University Press*.
2. Nash, J. (1950). "Equilibrium Points in N-Person Games." *PNAS*, 36(1), 48-49.
3. von Neumann, J. (1928). "Zur Theorie der Gesellschaftsspiele." *Mathematische Annalen*, 100(1), 295-320.
4. Goodfellow, I., et al. (2014). "Generative Adversarial Networks." *NeurIPS*.
5. Shoham, Y., & Leyton-Brown, K. (2008). "Multiagent Systems: Algorithmic, Game-Theoretic, and Logical Foundations." *Cambridge University Press*.
6. Nisan, N., et al. (2007). "Algorithmic Game Theory." *Cambridge University Press*.
7. Myerson, R. B. (1991). "Game Theory: Analysis of Conflict." *Harvard University Press*.
8. Hart, S., & Mas-Colell, A. (2000). "A Simple Adaptive Procedure Leading to Correlated Equilibrium." *Econometrica*, 68(5), 1127-1150.

---

## 9. 相关文档

- [[GAN深度指南]] - GAN的博弈论基础与训练动态
- [[多智能体博弈详解]] - 多智能体系统中的博弈论应用
- [[对抗训练与鲁棒性]] - 对抗训练中的min-max优化
- [[对抗样本深度指南]] - 对抗攻击的博弈论视角
