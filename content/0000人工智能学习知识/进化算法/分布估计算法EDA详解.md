---
title: 分布估计算法EDA详解
date: 2026-04-24
tags:
  - 进化算法
  - 分布估计算法
  - EDA
  - PBIL
  - UMDA
  - 贝叶斯优化
  - 概率模型
aliases:
  - Estimation of Distribution Algorithm
  - EDA
  - Probabilistic Evolutionary Algorithm
---

# 分布估计算法 EDA 详解

你有没有想过：如果不维护一个种群，而是维护一个**概率分布**，会怎样？

传统遗传算法通过选择、交叉、变异来产生新个体，这些操作本质上是在"猜测"好的解应该长什么样。但如果我们能直接学习"好解的概率分布"，然后从这个分布中采样，不就省去了那些花里胡哨的遗传算子吗？

这正是**分布估计算法（Estimation of Distribution Algorithm，EDA）**的核心思想。EDA是进化算法领域的一个分支，它用概率模型来替代传统的遗传算子，被认为是"进化算法的概率模型新范式"。

## EDA概述：为什么需要概率模型

先回顾一下传统GA的工作方式：

1. 维护一个种群，每个个体代表一个候选解
2. 通过选择让好的个体有更多机会繁殖
3. 通过交叉组合不同个体的"基因片段"
4. 通过变异引入新基因

问题在于：**交叉和变异是盲目的**。我们根本不知道"好解"到底有什么特征，只是凭借直觉设计了一些"看起来合理"的算子。

EDA换了个思路：与其瞎猜，不如直接学习。每一代，EDA会：
1. 从当前种群中**估计**一个概率分布
2. 从这个分布中**采样**产生新个体
3. 用新个体替换部分旧种群

这样，EDA自动学会了"好解长什么样"——如果某个基因位大多数是好解都有值1，那下一代的概率分布就会倾向于在这个位置生成1。

### EDA vs GA的核心区别

| 维度 | 传统GA | EDA |
|------|--------|-----|
| 信息表示 | 种群（显式） | 概率分布（隐式） |
| 搜索机制 | 选择+交叉+变异 | 概率估计+采样 |
| 基因依赖 | 难以处理 | 显式建模 |
| 计算成本 | 交叉/变异便宜 | 概率估计可能昂贵 |
| 适用场景 | 一般优化 | 有结构的问题 |

EDA的"基因依赖建模"能力特别重要。举个例子：在旅行商问题中，城市1和城市3经常一起访问，这意味着它们之间可能有某种相关性。传统GA的交叉操作可能很难保留这种相关性，但EDA的概率模型可以轻松捕获。

## PBIL：基于人群的信息学习

PBIL（Population-Based Incremental Learning）是最简单的EDA之一，由Baluja和Caruana在1995年提出。它的核心是维护一个**概率向量**，代表每个基因位的"好值概率"。

### PBIL的原理

PBIL维护一个概率向量 $P = (p_1, p_2, ..., p_n)$，其中 $p_i$ 表示第 $i$ 个基因位为1的概率。

每一步迭代：
1. 从 $P$ 采样 $N$ 个个体
2. 评估这 $N$ 个个体的适应度
3. 从中选择最优的 $M$ 个个体
4. **更新 $P$**：让 $p_i$ 向"好解的第 $i$ 位值"靠近

更新公式：

$$p_i^{(new)} = (1 - \alpha) \cdot p_i^{(old)} + \alpha \cdot \bar{x}_i$$

其中 $\alpha$ 是学习率，$\bar{x}_i$ 是最优个体的第 $i$ 位平均值。

```python
import random
import numpy as np
from typing import List, Callable

class PBIL:
    """基于人群的增量学习"""
    
    def __init__(self, n_variables: int, 
                 pop_size: int = 100,
                 n_elites: int = 1,
                 learning_rate: float = 0.1,
                 mutation_prob: float = 0.02,
                 mutation_shift: float = 0.05):
        self.n = n_variables
        self.pop_size = pop_size
        self.n_elites = n_elites
        self.alpha = learning_rate
        self.mutation_prob = mutation_prob
        self.mutation_shift = mutation_shift
        
        # 初始化概率向量为0.5（无偏）
        self.prob_vector = np.ones(self.n) * 0.5
    
    def sample(self) -> np.ndarray:
        """从概率向量采样"""
        return (np.random.random(self.n) < self.prob_vector).astype(int)
    
    def update(self, elite_solutions: List[np.ndarray]):
        """更新概率向量"""
        # 计算精英个体的均值
        elite_mean = np.mean(elite_solutions, axis=0)
        
        # 概率向量向精英均值移动
        self.prob_vector = (1 - self.alpha) * self.prob_vector + self.alpha * elite_mean
    
    def mutate_probabilities(self):
        """变异概率向量"""
        for i in range(self.n):
            if random.random() < self.mutation_prob:
                # 概率向随机方向移动
                shift = random.uniform(-self.mutation_shift, self.mutation_shift)
                self.prob_vector[i] = np.clip(
                    self.prob_vector[i] + shift, 0.01, 0.99
                )
    
    def evolve(self, fitness_func: Callable[[np.ndarray], float], 
               generations: int = 100) -> tuple:
        """进化主循环"""
        best_solution = None
        best_fitness = float('-inf')
        
        for gen in range(generations):
            # 采样
            population = [self.sample() for _ in range(self.pop_size)]
            
            # 评估
            fitnesses = [(i, fitness_func(ind)) for i, ind in enumerate(population)]
            
            # 选择精英
            fitnesses.sort(key=lambda x: x[1], reverse=True)
            elites = [population[i] for i, _ in fitnesses[:self.n_elites]]
            
            # 更新概率向量
            self.update(elites)
            
            # 变异
            self.mutate_probabilities()
            
            # 记录最佳
            if fitnesses[0][1] > best_fitness:
                best_fitness = fitnesses[0][1]
                best_solution = population[fitnesses[0][0]].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best fitness = {best_fitness:.4f}")
        
        return best_solution, best_fitness


# 测试：用PBIL解决OneMax问题
def one_max(x: np.ndarray) -> float:
    """OneMax函数：最大化1的数量"""
    return np.sum(x)

# 测试：LeadindOnes问题
def leading_ones(x: np.ndarray) -> float:
    """LeadingOnes：计算前导1的数量"""
    for i, val in enumerate(x):
        if val == 0:
            return i
    return len(x)

if __name__ == "__main__":
    print("Testing PBIL on OneMax problem")
    
    pbil = PBIL(
        n_variables=50,
        pop_size=100,
        n_elites=5,
        learning_rate=0.1
    )
    
    best, fitness = pbil.evolve(one_max, generations=100)
    print(f"\nBest solution: {best[:10]}... (showing first 10 bits)")
    print(f"Best fitness: {fitness}")
    print(f"Number of ones: {np.sum(best)}")
```

### PBIL的变体

PBIL有几种变体：

**PBIL-a**：只学习精英个体的平均值，不考虑其他个体。

**PBIL-b**：学习精英个体的线性组合：

$$p_i^{(new)} = (1 - \alpha) \cdot p_i^{(old)} + \alpha \cdot \beta \cdot x_i^{best}$$

其中 $x_i^{best}$ 是最优个体的第 $i$ 位。

**PBIL-c**：学习所有精英个体的概率，不只是最优：

```python
def update_pbila(self, elite_solutions: List[np.ndarray]):
    """PBIL-a: 只学习最优解"""
    best_solution = max(elite_solutions, key=lambda x: self.fitness(x))
    best_vec = best_solution.astype(float)
    self.prob_vector = (1 - self.alpha) * self.prob_vector + self.alpha * best_vec

def update_pbilb(self, elite_solutions: List[np.ndarray]):
    """PBIL-b: 按适应度加权的平均"""
    # 计算权重
    fitnesses = [self.fitness(sol) for sol in elite_solutions]
    total_fitness = sum(fitnesses)
    weights = [f / total_fitness for f in fitnesses]
    
    # 加权平均
    weighted_sum = np.zeros(self.n)
    for sol, w in zip(elite_solutions, weights):
        weighted_sum += w * sol.astype(float)
    
    self.prob_vector = (1 - self.alpha) * self.prob_vector + self.alpha * weighted_sum
```

## UMDA：无分布估计算法

UMDA（Univariate Marginal Distribution Algorithm）是另一个经典的EDA算法。与PBIL不同，UMDA假设变量之间是**独立的**，因此它估计的是每个变量的边缘分布，而不是联合分布。

### UMDA的原理

UMDA的核心假设是：**好的解是由于某些变量有更高的概率取特定值，而这些变量之间没有显著的相关性**。

这个假设在很多问题上成立。比如在组合优化中，如果解的质量主要由各个变量的取值决定，而变量之间没有强耦合关系，UMDA就能工作得很好。

```python
class UMDA:
    """无分布估计算法"""
    
    def __init__(self, n_variables: int,
                 pop_size: int = 100,
                 selected_size: int = 50):
        self.n = n_variables
        self.pop_size = pop_size
        self.selected_size = selected_size
        
        # 概率模型：每个变量取1的概率
        self.probabilities = np.ones(self.n) * 0.5
    
    def sample(self) -> np.ndarray:
        """从概率模型采样"""
        return (np.random.random(self.n) < self.probabilities).astype(int)
    
    def estimate(self, selected_individuals: List[np.ndarray]):
        """从选中的个体估计概率分布"""
        selected_matrix = np.array(selected_individuals)
        
        # 估计每个变量取1的概率（频率）
        self.probabilities = np.mean(selected_matrix, axis=0)
    
    def select(self, population: List[np.ndarray], 
               fitnesses: List[float]) -> List[np.ndarray]:
        """基于适应度的选择"""
        # 按适应度排序，选择前selected_size个
        sorted_pairs = sorted(zip(population, fitnesses), 
                             key=lambda x: x[1], reverse=True)
        return [ind for ind, _ in sorted_pairs[:self.selected_size]]
    
    def evolve(self, fitness_func: Callable, generations: int = 100):
        """进化"""
        # 初始化
        population = [self.sample() for _ in range(self.pop_size)]
        
        best_solution = None
        best_fitness = float('-inf')
        
        for gen in range(generations):
            # 评估
            fitnesses = [fitness_func(ind) for ind in population]
            
            # 选择
            selected = self.select(population, fitnesses)
            
            # 估计
            self.estimate(selected)
            
            # 采样新个体
            new_population = [self.sample() for _ in range(self.pop_size)]
            
            # 合并（也可以完全替换）
            population = selected + new_population[:len(population) - len(selected)]
            
            # 更新最优
            current_best_idx = np.argmax(fitnesses)
            if fitnesses[current_best_idx] > best_fitness:
                best_fitness = fitnesses[current_best_idx]
                best_solution = population[current_best_idx].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best = {best_fitness:.4f}")
        
        return best_solution, best_fitness
```

### UMDA vs PBIL

| 方面 | UMDA | PBIL |
|------|------|------|
| 选择策略 | 确定性选择top-k | 采样后选择精英 |
| 概率更新 | 基于选中个体的频率 | 基于精英的移动方向 |
| 探索能力 | 较弱 | 较强 |
| 收敛速度 | 快 | 较慢但更稳定 |

UMDA收敛快，但容易陷入局部最优；PBIL有更多随机性，探索能力更强。

## MIMIC：利用互信息

MIMIC（Mutual Information Maximizing Input Clustering）是更高级的EDA，它考虑了**变量之间的依赖关系**。

### MIMIC的核心思想

MIMIC假设解空间有一个**隐含的最优分布**，这个分布可以用一个链式结构来近似：

$$P(X_1, X_2, ..., X_n) = P(X_{i_1}) \cdot P(X_{i_2} | X_{i_1}) \cdot ... \cdot P(X_{i_n} | X_{i_{n-1}})$$

这个链式结构是通过**互信息**来确定的：互信息越大的两个变量，应该离得越近。

互信息的定义：

$$I(X; Y) = \sum_{x,y} P(x,y) \log \frac{P(x,y)}{P(x)P(y)}$$

```python
from itertools import combinations

class MIMIC:
    """MIMIC: 利用互信息的EDA"""
    
    def __init__(self, n_variables: int,
                 pop_size: int = 100,
                 select_size: int = 50):
        self.n = n_variables
        self.pop_size = pop_size
        self.select_size = select_size
        
        # 链式顺序（待确定）
        self.chain_order = list(range(n_variables))
        
        # 边缘概率和条件概率
        self.marginal_probs = np.ones(self.n) * 0.5
        self.conditional_probs = {}  # (i, parent) -> P(X_i=1 | X_parent=0 or 1)
    
    def compute_mutual_information(self, X: np.ndarray, Y: np.ndarray):
        """计算两个变量的互信息"""
        n = len(X)
        
        # 统计频率
        p_x0 = np.sum(X == 0) / n
        p_x1 = np.sum(X == 1) / n
        p_y0 = np.sum(Y == 0) / n
        p_y1 = np.sum(Y == 1) / n
        
        p_x0y0 = np.sum((X == 0) & (Y == 0)) / n
        p_x0y1 = np.sum((X == 0) & (Y == 1)) / n
        p_x1y0 = np.sum((X == 1) & (Y == 0)) / n
        p_x1y1 = np.sum((X == 1) & (Y == 1)) / n
        
        mi = 0
        for px, py, pxy in [(p_x0, p_y0, p_x0y0), (p_x0, p_y1, p_x0y1),
                           (p_x1, p_y0, p_x1y0), (p_x1, p_y1, p_x1y1)]:
            if px > 0 and py > 0 and pxy > 0:
                mi += pxy * np.log(pxy / (px * py))
        
        return mi
    
    def learn_chain_structure(self, selected: List[np.ndarray]):
        """学习链式结构"""
        data = np.array(selected)
        n_vars = data.shape[1]
        
        # 计算所有变量对的互信息
        mi_matrix = np.zeros((n_vars, n_vars))
        for i in range(n_vars):
            for j in range(n_vars):
                if i != j:
                    mi_matrix[i, j] = self.compute_mutual_information(
                        data[:, i], data[:, j]
                    )
        
        # 贪婪构建链：从第一个变量开始，每次添加与之互信息最大的未加入变量
        unvisited = set(range(n_vars))
        
        # 选择边缘熵最大的作为起点
        entropies = -np.sum(data * np.log(data + 1e-10) + 
                           (1-data) * np.log(1-data + 1e-10), axis=0)
        first_var = np.argmax(entropies)
        
        chain = [first_var]
        unvisited.remove(first_var)
        
        while unvisited:
            last_var = chain[-1]
            best_next = max(unvisited, 
                          key=lambda x: mi_matrix[last_var, x])
            chain.append(best_next)
            unvisited.remove(best_next)
        
        self.chain_order = chain
    
    def estimate_probabilities(self, selected: List[np.ndarray]):
        """估计边缘和条件概率"""
        data = np.array(selected)
        
        # 重新排列数据
        ordered_data = data[:, self.chain_order]
        
        # 边缘概率
        self.marginal_probs[self.chain_order[0]] = np.mean(
            ordered_data[:, 0]
        )
        
        # 条件概率
        for i, var_idx in enumerate(self.chain_order[1:], 1):
            parent_idx = self.chain_order[i - 1]
            
            # P(X=1 | Parent=0)
            mask_parent_0 = ordered_data[:, i-1] == 0
            mask_parent_1 = ordered_data[:, i-1] == 1
            
            p_x1_given_parent0 = (np.sum(ordered_data[mask_parent_0, i]) / 
                                (np.sum(mask_parent_0) + 1e-10))
            p_x1_given_parent1 = (np.sum(ordered_data[mask_parent_1, i]) / 
                                (np.sum(mask_parent_1) + 1e-10))
            
            self.conditional_probs[(var_idx, parent_idx, 0)] = p_x1_given_parent0
            self.conditional_probs[(var_idx, parent_idx, 1)] = p_x1_given_parent1
    
    def sample(self) -> np.ndarray:
        """从链式分布采样"""
        solution = np.zeros(self.n, dtype=int)
        
        # 第一个变量
        first_var = self.chain_order[0]
        solution[first_var] = int(np.random.random() < self.marginal_probs[first_var])
        
        # 链式采样
        for i in range(1, len(self.chain_order)):
            var_idx = self.chain_order[i]
            parent_idx = self.chain_order[i - 1]
            parent_val = solution[parent_idx]
            
            p = self.conditional_probs.get(
                (var_idx, parent_idx, parent_val), 0.5
            )
            solution[var_idx] = int(np.random.random() < p)
        
        return solution
    
    def evolve(self, fitness_func: Callable, generations: int = 100):
        """进化"""
        population = [self.sample() for _ in range(self.pop_size)]
        
        best_solution = None
        best_fitness = float('-inf')
        
        for gen in range(generations):
            # 评估
            fitnesses = [fitness_func(ind) for ind in population]
            
            # 选择
            sorted_pairs = sorted(zip(population, fitnesses),
                                key=lambda x: x[1], reverse=True)
            selected = [ind for ind, _ in sorted_pairs[:self.select_size]]
            
            # 学习结构
            self.learn_chain_structure(selected)
            
            # 估计概率
            self.estimate_probabilities(selected)
            
            # 采样
            new_population = [self.sample() for _ in range(self.pop_size)]
            
            # 更新
            population = selected + new_population[:self.pop_size - len(selected)]
            
            # 最优
            current_best_idx = np.argmax(fitnesses)
            if fitnesses[current_best_idx] > best_fitness:
                best_fitness = fitnesses[current_best_idx]
                best_solution = population[current_best_idx].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best = {best_fitness:.4f}")
        
        return best_solution, best_fitness
```

MIMIC比UMDA更复杂，但对于有明显依赖结构的问题，MIMIC表现更好。

## 贝叶斯优化与EDA的联系与区别

贝叶斯优化（Bayesian Optimization）和EDA都是基于概率模型的优化方法，但它们的设计目标和使用场景不同。

### 共同点

1. 都维护一个概率模型来指导搜索
2. 都利用历史信息来更新模型
3. 都有"探索 vs 利用"的权衡

### 区别

| 方面 | EDA | 贝叶斯优化 |
|------|-----|-----------|
| 采样方式 | 从概率模型采样 | 采集函数（EI, UCB） |
| 模型复杂度 | 简单到复杂都有 | 通常用高斯过程 |
| 目标函数 | 评估所有候选 | 评估选中的候选 |
| 适用维度 | 低维到高维 | 低维（<20） |
| 计算成本 | 概率估计可控 | GP训练 O(n³) |

EDA和贝叶斯优化可以结合：EDA用于离散优化，贝叶斯优化用于连续优化。

## 贝叶斯网络EDA

贝叶斯网络EDA是更通用的概率建模方法。它用贝叶斯网络来表示变量之间的依赖关系，可以捕获任意复杂的依赖结构。

### 贝叶斯网络基础

贝叶斯网络是一个有向无环图（DAG），节点表示变量，边表示依赖关系。每个节点的概率取决于它的父节点。

学习贝叶斯网络包括两个任务：
1. **结构学习**：找到最优的网络结构
2. **参数学习**：给定结构，估计条件概率表（CPT）

```python
import random
from collections import defaultdict
from itertools import product

class BayesianNetworkEDA:
    """基于贝叶斯网络的EDA"""
    
    def __init__(self, n_variables: int,
                 pop_size: int = 100,
                 select_size: int = 50):
        self.n = n_variables
        self.pop_size = pop_size
        self.select_size = select_size
        
        # 网络结构：parent[child] = list of parents
        self.parents = defaultdict(list)
        
        # 条件概率表：cpt[(child, parent_values)] = P(child=1 | parents)
        self.cpt = {}
    
    def learn_structure(self, selected: List[np.ndarray], 
                       max_parents: int = 3):
        """学习贝叶斯网络结构"""
        data = np.array(selected)
        n = len(data)
        
        # 初始化：每个变量没有父节点
        self.parents = defaultdict(list)
        
        # 贪婪添加边
        for _ in range(self.n * max_parents):
            best_improvement = 0
            best_edge = None
            
            for child in range(self.n):
                for parent in range(self.n):
                    if parent == child:
                        continue
                    if parent in self.parents[child]:
                        continue
                    
                    # 添加这条边是否改善BIC评分
                    current_score = self._bic_score(data, child)
                    self.parents[child].append(parent)
                    new_score = self._bic_score(data, child)
                    self.parents[child].remove(parent)
                    
                    improvement = new_score - current_score
                    
                    if improvement > best_improvement:
                        best_improvement = improvement
                        best_edge = (child, parent)
            
            if best_edge and best_improvement > 0:
                self.parents[best_edge[0]].append(best_edge[1])
    
    def _bic_score(self, data: np.ndarray, child: int) -> float:
        """BIC评分：衡量结构对数据的拟合度"""
        n = len(data)
        child_data = data[:, child]
        
        # 如果没有父节点
        if not self.parents[child]:
            p = np.mean(child_data)
            if p == 0 or p == 1:
                return 0
            return n * (p * np.log(p) + (1-p) * np.log(1-p)) - 0.5 * np.log(n)
        
        # 有父节点：计算条件频率
        parents = self.parents[child]
        parent_data = data[:, parents]
        
        log_likelihood = 0
        for parent_vals in product([0, 1], repeat=len(parents)):
            mask = np.all(parent_data == parent_vals, axis=1)
            if np.sum(mask) == 0:
                continue
            
            n_j = np.sum(mask)
            n_1j = np.sum(child_data[mask])
            
            if n_1j == 0:
                continue
            p = n_1j / n_j
            log_likelihood += n_j * (p * np.log(p + 1e-10) + 
                                      (1-p) * np.log(1-p + 1e-10))
        
        # BIC惩罚
        k = 2 ** len(parents)
        bic = log_likelihood - 0.5 * np.log(n) * k
        
        return bic
    
    def estimate_parameters(self, selected: List[np.ndarray]):
        """估计条件概率"""
        data = np.array(selected)
        
        for child, parents in self.parents.items():
            if not parents:
                # 边缘概率
                self.cpt[(child,)] = np.mean(data[:, child])
            else:
                # 条件概率
                parent_data = data[:, parents]
                child_data = data[:, child]
                
                for parent_vals in product([0, 1], repeat=len(parents)):
                    mask = np.all(parent_data == parent_vals, axis=1)
                    if np.sum(mask) == 0:
                        self.cpt[(child,) + parent_vals] = 0.5
                    else:
                        self.cpt[(child,) + parent_vals] = np.mean(child_data[mask])
    
    def sample(self) -> np.ndarray:
        """从贝叶斯网络采样"""
        solution = np.zeros(self.n, dtype=int)
        
        # 拓扑排序（简单版：按父节点数量）
        nodes_sorted = sorted(range(self.n), 
                            key=lambda x: len(self.parents[x]))
        
        for node in nodes_sorted:
            parents = self.parents[node]
            
            if not parents:
                prob = self.cpt.get((node,), 0.5)
            else:
                parent_vals = tuple(solution[p] for p in parents)
                prob = self.cpt.get((node,) + parent_vals, 0.5)
            
            solution[node] = int(np.random.random() < prob)
        
        return solution
    
    def evolve(self, fitness_func: Callable, generations: int = 100):
        """进化"""
        population = [self.sample() for _ in range(self.pop_size)]
        
        best_solution = None
        best_fitness = float('-inf')
        
        for gen in range(generations):
            # 评估
            fitnesses = [fitness_func(ind) for ind in population]
            
            # 选择
            sorted_pairs = sorted(zip(population, fitnesses),
                                key=lambda x: x[1], reverse=True)
            selected = [ind for ind, _ in sorted_pairs[:self.select_size]]
            
            # 学习结构和参数
            self.learn_structure(selected)
            self.estimate_parameters(selected)
            
            # 采样
            new_population = [self.sample() for _ in range(self.pop_size)]
            
            population = selected + new_population[:self.pop_size - len(selected)]
            
            # 最优
            current_best_idx = np.argmax(fitnesses)
            if fitnesses[current_best_idx] > best_fitness:
                best_fitness = fitnesses[current_best_idx]
                best_solution = population[current_best_idx].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best = {best_fitness:.4f}")
        
        return best_solution, best_fitness
```

## 高斯过程EDA：连续空间的概率建模

对于连续优化问题，需要用高斯过程（GP）或其他连续概率模型来建模。

```python
from scipy.stats import norm
from scipy.optimize import minimize

class GaussianProcessEDA:
    """基于高斯过程的EDA（用于连续优化）"""
    
    def __init__(self, n_variables: int,
                 pop_size: int = 100,
                 select_size: int = 50,
                 bounds: tuple = (-5, 5)):
        self.n = n_variables
        self.pop_size = pop_size
        self.select_size = select_size
        self.bounds = bounds
        
        # 历史数据
        self.X_history = []
        self.y_history = []
        
        # GP超参数
        self.length_scale = 1.0
        self.noise_var = 0.01
        self.output_scale = 1.0
    
    def gaussian_kernel(self, X1, X2):
        """RBF核函数"""
        X1 = np.atleast_2d(X1)
        X2 = np.atleast_2d(X2)
        
        dists = np.sum((X1[:, np.newaxis, :] - X2[np.newaxis, :, :]) ** 2, axis=2)
        return self.output_scale * np.exp(-0.5 * dists / self.length_scale ** 2)
    
    def fit_gp(self):
        """拟合高斯过程"""
        if len(self.X_history) < 2:
            return
        
        X = np.array(self.X_history)
        y = np.array(self.y_history)
        
        # 简单的超参数估计（简化版）
        y_var = np.var(y)
        self.output_scale = y_var if y_var > 0 else 1.0
    
    def predict(self, X):
        """GP预测"""
        if not self.X_history:
            return np.zeros(len(X))
        
        X_train = np.array(self.X_history)
        y_train = np.array(self.y_history)
        
        K = self.gaussian_kernel(X_train, X_train) + self.noise_var * np.eye(len(X_train))
        K_star = self.gaussian_kernel(X_train, X)
        
        try:
            K_inv = np.linalg.inv(K + 1e-6 * np.eye(len(K)))
            mu = K_star.T @ K_inv @ y_train
            
            # 计算方差
            K_star_star = self.gaussian_kernel(X, X)
            var = np.diag(K_star_star - K_star.T @ K_inv @ K_star)
            var = np.maximum(var, 1e-6)
            
            return mu, var
        except:
            return np.zeros(len(X)), np.ones(len(X))
    
    def acquisition_function(self, X, acq='ei'):
        """采集函数：Expected Improvement"""
        mu, var = self.predict(X)
        
        if isinstance(mu, float):
            mu = np.array([mu])
            var = np.array([var])
        
        # 最好值
        y_best = max(self.y_history) if self.y_history else 0
        
        # 标准差
        std = np.sqrt(var)
        
        if acq == 'ei':
            # Expected Improvement
            with np.errstate(divide='ignore', invalid='ignore'):
                z = (mu - y_best) / std
                ei = (mu - y_best) * norm.cdf(z) + std * norm.pdf(z)
                ei[std < 1e-6] = 0
            return -ei  # 负号因为我们要最小化
    
    def sample_from_gp(self, n_samples: int = 1):
        """从GP后验采样"""
        if not self.X_history:
            # 无数据时从先验采样
            return np.random.uniform(self.bounds[0], self.bounds[1], 
                                    (n_samples, self.n))
        
        mu, var = self.predict(np.zeros((1, self.n)))  # dummy
        
        # 生成候选点
        candidates = np.random.uniform(self.bounds[0], self.bounds[1],
                                     (1000, self.n))
        
        # 用采集函数排序
        acq_values = -self.acquisition_function(candidates)
        
        # 选择最优的n_samples个
        indices = np.argsort(acq_values)[:n_samples]
        return candidates[indices]
    
    def evolve(self, fitness_func: Callable, generations: int = 100):
        """进化"""
        best_solution = None
        best_fitness = float('-inf')
        
        for gen in range(generations):
            # 从GP采样
            if gen == 0:
                # 第一代：随机采样
                population = np.random.uniform(
                    self.bounds[0], self.bounds[1], (self.pop_size, self.n)
                )
            else:
                # 从GP采样 + 一些随机扰动
                gp_samples = self.sample_from_gp(self.pop_size // 2)
                random_samples = np.random.uniform(
                    self.bounds[0], self.bounds[1], 
                    (self.pop_size - len(gp_samples), self.n)
                )
                population = np.vstack([gp_samples, random_samples])
            
            # 评估
            fitnesses = [fitness_func(ind) for ind in population]
            
            # 更新历史
            self.X_history.extend(population.tolist())
            self.y_history.extend(fitnesses)
            
            # 限制历史大小
            if len(self.X_history) > 500:
                self.X_history = self.X_history[-500:]
                self.y_history = self.y_history[-500:]
            
            # 拟合GP
            self.fit_gp()
            
            # 选择精英
            sorted_indices = np.argsort(fitnesses)[::-1]
            selected = [population[i] for i in sorted_indices[:self.select_size]]
            
            # 最优
            if fitnesses[sorted_indices[0]] > best_fitness:
                best_fitness = fitnesses[sorted_indices[0]]
                best_solution = population[sorted_indices[0]].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best = {best_fitness:.4f}")
        
        return best_solution, best_fitness


# 测试连续优化
def rastrigin(x: np.ndarray) -> float:
    A = 10
    return A * len(x) + np.sum(x**2 - A * np.cos(2 * np.pi * x))

if __name__ == "__main__":
    gp_eda = GaussianProcessEDA(n_variables=5, pop_size=50, 
                                bounds=(-5, 5))
    best, fitness = gp_eda.evolve(rastrigin, generations=50)
    print(f"Best solution: {best}")
    print(f"Best fitness: {fitness:.4f}")
```

## 代码实战：UMDA解决组合优化

下面是一个完整的UMDA实现，用于解决旅行商问题（TSP）。

```python
import random
import numpy as np
from typing import List, Tuple

class UMDATSP:
    """用UMDA解决旅行商问题"""
    
    def __init__(self, n_cities: int, distance_matrix: np.ndarray,
                 pop_size: int = 200, select_size: int = 100):
        self.n = n_cities
        self.distance_matrix = distance_matrix
        self.pop_size = pop_size
        self.select_size = select_size
        
        # 概率模型：p[i][j] = P(city j will be selected at position i)
        self.probabilities = np.ones((n_cities, n_cities)) / n_cities
    
    def sample(self) -> List[int]:
        """从概率模型采样一个解"""
        tour = []
        remaining = set(range(self.n))
        
        for pos in range(self.n):
            # 按概率选择下一个城市
            probs = [self.probabilities[pos][city] if city in remaining else 0 
                    for city in range(self.n)]
            probs = np.array(probs)
            probs /= probs.sum()  # 归一化
            
            city = np.random.choice(self.n, p=probs)
            tour.append(city)
            remaining.remove(city)
        
        return tour
    
    def evaluate(self, tour: List[int]) -> float:
        """计算路径长度"""
        total = 0
        for i in range(len(tour) - 1):
            total += self.distance_matrix[tour[i]][tour[i + 1]]
        total += self.distance_matrix[tour[-1]][tour[0]]  # 回到起点
        return total
    
    def estimate(self, selected_tours: List[List[int]]):
        """从选中个体估计概率分布"""
        # 统计每个位置上各城市出现的频率
        new_probs = np.zeros((self.n, self.n))
        
        for tour in selected_tours:
            for pos, city in enumerate(tour):
                new_probs[pos][city] += 1
        
        # 归一化
        for pos in range(self.n):
            row_sum = new_probs[pos].sum()
            if row_sum > 0:
                new_probs[pos] /= row_sum
            else:
                new_probs[pos] = np.ones(self.n) / self.n
        
        # 平滑更新
        self.probabilities = 0.8 * self.probabilities + 0.2 * new_probs
    
    def select(self, tours: List[List[int]], 
               fitnesses: List[float]) -> List[List[int]]:
        """选择精英"""
        sorted_pairs = sorted(zip(tours, fitnesses), key=lambda x: x[1])
        return [tour for tour, _ in sorted_pairs[:self.select_size]]
    
    def evolve(self, generations: int = 100) -> Tuple[List[int], float]:
        """进化"""
        # 初始化种群
        population = [list(range(self.n)) for _ in range(self.pop_size)]
        for tour in population:
            random.shuffle(tour)
        
        best_tour = None
        best_distance = float('inf')
        
        for gen in range(generations):
            # 评估
            fitnesses = [1 / self.evaluate(tour) for tour in population]  # 距离越小适应度越高
            
            # 选择
            selected = self.select(population, fitnesses)
            
            # 估计
            self.estimate(selected)
            
            # 采样新个体
            new_population = [self.sample() for _ in range(self.pop_size)]
            
            # 合并
            all_tours = selected + new_population[:self.pop_size - len(selected)]
            population = all_tours
            
            # 更新最优
            distances = [self.evaluate(tour) for tour in population]
            min_idx = np.argmin(distances)
            if distances[min_idx] < best_distance:
                best_distance = distances[min_idx]
                best_tour = population[min_idx].copy()
            
            if gen % 20 == 0:
                print(f"Gen {gen}: Best distance = {best_distance:.2f}")
        
        return best_tour, best_distance


def generate_tsp_instance(n_cities: int, seed: int = 42) -> np.ndarray:
    """生成随机TSP实例"""
    random.seed(seed)
    np.random.seed(seed)
    
    # 城市坐标
    coords = np.random.rand(n_cities, 2) * 100
    
    # 计算距离矩阵
    dist_matrix = np.zeros((n_cities, n_cities))
    for i in range(n_cities):
        for j in range(i + 1, n_cities):
            d = np.sqrt(np.sum((coords[i] - coords[j]) ** 2))
            dist_matrix[i, j] = d
            dist_matrix[j, i] = d
    
    return dist_matrix


if __name__ == "__main__":
    # 生成10城市的TSP实例
    n_cities = 20
    dist_matrix = generate_tsp_instance(n_cities)
    
    # 用UMDA求解
    umda = UMDATSP(n_cities, dist_matrix, pop_size=200, select_size=100)
    best_tour, best_distance = umda.evolve(generations=100)
    
    print(f"\nBest tour: {best_tour}")
    print(f"Best distance: {best_distance:.2f}")
```

## EDA vs GA对比：什么时候用EDA更有效

### EDA的优势场景

**1. 变量之间有明显依赖关系**

当问题有复杂的约束或结构时，EDA的概率模型可以捕获这些关系，而GA的交叉操作可能很难做到。

**2. 高维问题**

对于高维问题，GA的交叉操作可能会产生大量不可行解，而EDA的概率模型可以更好地引导搜索。

**3. 需要快速收敛**

EDA通常比GA收敛更快，因为它直接学习好的解的分布。

**4. 组合优化**

对于组合优化问题，EDA的概率模型特别有效。

### EDA的劣势

**1. 概率估计的计算成本**

对于大规模问题，估计高维概率分布可能很昂贵。

**2. 模型选择困难**

如何选择合适的概率模型（单变量、多变量、贝叶斯网络）需要领域知识。

**3. 局部最优**

EDA可能陷入"概率质量集中"的困境，需要适当的机制来维持多样性。

**4. 离散 vs 连续**

EDA天然适合离散问题，对于连续问题需要特殊处理（如高斯过程）。

### 实战决策指南

```
问题类型                    推荐方法
────────────────────────────────────────────
低维连续优化                贝叶斯优化 > GA
高维连续优化                CMA-ES > GA
离散组合优化                EDA (UMDA/PBIL)
有明显依赖结构的问题        贝叶斯网络EDA
混合整数规划                混合方法
黑盒优化、函数评估昂贵      贝叶斯优化
快速原型、需要鲁棒性         GA
```

### 混合策略

EDA和GA可以结合使用：

```python
class HybridEDA_GA:
    """EDA + GA 混合算法"""
    
    def __init__(self, n_variables, pop_size=100):
        self.n = n_variables
        self.pop_size = pop_size
        
        # EDA部分
        self.probabilities = np.ones(n_variables) * 0.5
        
        # GA部分
        self.crossover_rate = 0.8
        self.mutation_rate = 0.02
    
    def eda_step(self, selected):
        """EDA概率更新"""
        elite_mean = np.mean(selected, axis=0)
        self.probabilities = 0.7 * self.probabilities + 0.3 * elite_mean
    
    def ga_step(self, population):
        """GA交叉变异"""
        offspring = []
        
        for i in range(0, len(population) - 1, 2):
            p1, p2 = population[i], population[i+1]
            
            if random.random() < self.crossover_rate:
                c1, c2 = self.crossover(p1, p2)
            else:
                c1, c2 = p1.copy(), p2.copy()
            
            c1 = self.mutate(c1)
            c2 = self.mutate(c2)
            
            offspring.extend([c1, c2])
        
        return offspring
    
    def evolve(self, fitness_func, generations=100):
        """混合进化"""
        population = [self.sample() for _ in range(self.pop_size)]
        
        for gen in range(generations):
            # 评估
            fitnesses = [fitness_func(ind) for ind in population]
            
            # 选择
            sorted_pairs = sorted(zip(population, fitnesses), 
                                key=lambda x: x[1], reverse=True)
            selected = [p for p, _ in sorted_pairs[:self.pop_size // 2]]
            
            # EDA更新概率
            self.eda_step(selected)
            
            # 用EDA概率生成部分后代
            eda_offspring = [self.sample() for _ in range(self.pop_size // 2)]
            
            # 用GA生成部分后代
            ga_offspring = self.ga_step(selected[:len(selected)//2])
            
            # 合并
            population = selected + eda_offspring + ga_offspring
            population = population[:self.pop_size]
        
        return max(population, key=fitness_func)
```

## 总结

EDA是进化算法的一个重要分支，它用概率模型替代传统的遗传算子，提供了一种更"智能"的搜索方式。

**核心要点**：

1. **EDA的本质**：学习好解的概率分布，而不是盲目杂交
2. **PBIL**：最简单的EDA，维护一个概率向量
3. **UMDA**：假设变量独立，基于边缘分布
4. **MIMIC**：考虑变量之间的互信息，用链式结构建模
5. **贝叶斯网络EDA**：最灵活，可以建模任意依赖结构
6. **高斯过程EDA**：用于连续优化

**实战建议**：

- 小规模离散问题：先试UMDA或PBIL
- 有依赖结构的复杂问题：试贝叶斯网络EDA
- 连续优化：考虑与贝叶斯优化结合
- 不确定用什么：先试GA，EDA作为备选

EDA和GA不是对立的，而是互补的。选择哪种方法，取决于你的问题特点。
