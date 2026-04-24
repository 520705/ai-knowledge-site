---
title: 粒子群优化PSO
date: 2026-04-18
tags:
  - 进化算法
  - 粒子群优化
  - PSO
  - Kennedy
  - Eberhart
  - 群智能
  - 鸟群算法
  - 粒子更新
  - 离散PSO
  - 社会学习
categories:
  - 人工智能
  - 进化算法
  - 人工智能学习知识
alias: Particle Swarm Optimization
---

## 关键词

| 粒子群优化 | 粒子更新 | 惯性权重 | 学习因子 | 全局最优 |
|-----------|---------|---------|---------|---------|
| 局部最优 | 速度更新 | 位置更新 | 收敛性 | 拓扑结构 |
| gbest模型 | lbest模型 | 离散PSO | 二进制PSO | PSO变体 |
| 自适应PSO | 协同PSO | 多目标PSO | 量子PSO | 混沌PSO |

---

## 摘要

粒子群优化（Particle Swarm Optimization, PSO）是由James Kennedy和Russell Eberhart于1995年提出的群体智能优化算法。该算法的灵感来源于鸟群觅食行为的社会动力学模型，通过模拟群体中个体之间的信息共享与协作机制，在解空间中进行高效的协同搜索。PSO以其算法简洁、实现方便、收敛速度快著称，已成为进化计算领域应用最广泛的算法之一，在函数优化、神经网络训练、调度优化、控制系统设计等领域得到成功应用。

---

# 一、PSO的起源与理论基础

## 1.1 生物社会学起源

PSO的诞生源于对生物群体行为的观察与建模。Kennedy和Eberhart在研究人工生命和群智能时，发现鸟群、鱼群等群体在觅食过程中表现出一种"集体智慧"——尽管每个个体只拥有有限的认知能力，但群体通过简单的局部交互能够实现复杂的全局行为。

**核心观察**：
- 鸟类在飞行时会调整自身轨迹，既参考自身经验（认知部分），也参考邻近个体的经验（社会部分）
- 群体的运动呈现一种"动态平衡"，既有探索新区域的趋势，也有向已发现优质区域聚集的趋势

## 1.2 社会认知理论

PSO的数学模型融合了社会心理学中的几个核心概念：

1. **社会影响（Social Influence）**：个体受邻近个体行为的影响
2. **社会学习（Social Learning）**：个体从群体经验中获取信息
3. **认知一致性（Cognitive Consistency）**：个体倾向于保持自身经验的一致性

## 1.3 与其他进化算法的对比

| 特征 | PSO | 遗传算法 | 差分进化 |
|------|-----|---------|---------|
| 信息共享 | 粒子相互通信 | 交叉重组 | 差分向量 |
| 参数数量 | 少 | 中等 | 中等 |
| 记忆机制 | pbest/gbest | 种群记忆 | 选择压力 |
| 收敛速度 | 快 | 较慢 | 中等 |

---

# 二、基本数学模型

## 2.1 速度和位置更新

PSO中每个粒子代表解空间中的一个候选解，其状态由位置和速度描述：

**位置更新公式**：

$$
\mathbf{x}_i(t+1) = \mathbf{x}_i(t) + \mathbf{v}_i(t+1)
$$

**速度更新公式**：

$$
\mathbf{v}_i(t+1) = \underbrace{w \cdot \mathbf{v}_i(t)}_{\text{惯性项}} + 
\underbrace{c_1 \cdot r_1 \cdot (\mathbf{p}_i - \mathbf{x}_i(t))}_{\text{认知项}} + 
\underbrace{c_2 \cdot r_2 \cdot (\mathbf{g} - \mathbf{x}_i(t))}_{\text{社会项}}
$$

其中：
- $\mathbf{x}_i(t)$：第$i$个粒子在时刻$t$的位置
- $\mathbf{v}_i(t)$：第$i$个粒子在时刻$t$的速度
- $w$：惯性权重（inertia weight）
- $c_1$：认知学习因子（cognitive parameter）
- $c_2$：社会学习因子（social parameter）
- $r_1, r_2 \sim U(0,1)$：均匀分布的随机数
- $\mathbf{p}_i$：第$i$个粒子的历史最优位置（pbest）
- $\mathbf{g}$：全局最优粒子的位置（gbest）

## 2.2 速度限制

为防止速度过大导致粒子越过最优区域，通常对速度进行限制：

$$
v_{ij} \in [-v_{\max}, v_{\max}]
$$

其中$v_{\max}$通常设为搜索空间范围的10%-20%。

## 2.3 位置限制

粒子的位置也需要限制在搜索空间内：

$$
x_{ij} \in [x_{\min}^j, x_{\max}^j]
$$

---

# 三、完整算法实现

## 3.1 标准PSO算法

```python
import numpy as np
import random

class ParticleSwarmOptimizer:
    def __init__(self, objective, dim, bounds, n_particles=30,
                 w=0.729, c1=1.49445, c2=1.49445,
                 max_iter=1000, tol=1e-8):
        """
        参数初始化
        
        Args:
            objective: 目标函数（最小化）
            dim: 问题维度
            bounds: 边界 [(x_min, x_max), ...]
            n_particles: 粒子数量
            w: 惯性权重
            c1: 认知学习因子
            c2: 社会学习因子
            max_iter: 最大迭代次数
            tol: 收敛容忍度
        """
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
        self.n_particles = n_particles
        self.w = w
        self.c1 = c1
        self.c2 = c2
        self.max_iter = max_iter
        self.tol = tol
        
        # 初始化速度限制
        self.v_max = np.array([(high - low) * 0.2 
                              for low, high in bounds])
        
        # 历史记录
        self.gbest_history = []
        self.pbest_history = []
    
    def initialize(self):
        """初始化粒子群"""
        bounds_array = np.array(self.bounds)
        low_bounds = bounds_array[:, 0]
        high_bounds = bounds_array[:, 1]
        
        # 随机初始化位置
        self.positions = np.random.uniform(
            low_bounds, high_bounds, 
            (self.n_particles, self.dim)
        )
        
        # 随机初始化速度
        self.velocities = np.random.uniform(
            -self.v_max, self.v_max,
            (self.n_particles, self.dim)
        )
        
        # 评估初始适应度
        self.fitness = np.array([
            self.objective(pos) for pos in self.positions
        ])
        
        # 初始化个体最优
        self.pbest_positions = self.positions.copy()
        self.pbest_fitness = self.fitness.copy()
        
        # 初始化全局最优
        gbest_idx = np.argmin(self.fitness)
        self.gbest_position = self.positions[gbest_idx].copy()
        self.gbest_fitness = self.fitness[gbest_idx]
    
    def update_velocities(self):
        """更新所有粒子的速度"""
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        # 认知项：向自身历史最优移动
        cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
        
        # 社会项：向全局最优移动
        social = self.c2 * r2 * (self.gbest_position - self.positions)
        
        # 更新速度
        self.velocities = self.w * self.velocities + cognitive + social
        
        # 速度限制
        for i in range(self.n_particles):
            for j in range(self.dim):
                if self.velocities[i, j] > self.v_max[j]:
                    self.velocities[i, j] = self.v_max[j]
                elif self.velocities[i, j] < -self.v_max[j]:
                    self.velocities[i, j] = -self.v_max[j]
    
    def update_positions(self):
        """更新所有粒子的位置"""
        self.positions += self.velocities
        
        # 位置限制
        bounds_array = np.array(self.bounds)
        low_bounds = bounds_array[:, 0]
        high_bounds = bounds_array[:, 1]
        
        self.positions = np.clip(self.positions, low_bounds, high_bounds)
    
    def evaluate(self):
        """评估当前粒子群"""
        for i in range(self.n_particles):
            fitness = self.objective(self.positions[i])
            self.fitness[i] = fitness
            
            # 更新个体最优
            if fitness < self.pbest_fitness[i]:
                self.pbest_positions[i] = self.positions[i].copy()
                self.pbest_fitness[i] = fitness
                
                # 更新全局最优
                if fitness < self.gbest_fitness:
                    self.gbest_position = self.positions[i].copy()
                    self.gbest_fitness = fitness
    
    def run(self):
        """运行PSO算法"""
        self.initialize()
        
        for iteration in range(self.max_iter):
            self.update_velocities()
            self.update_positions()
            self.evaluate()
            
            # 记录历史
            self.gbest_history.append(self.gbest_fitness)
            
            if iteration % 50 == 0:
                print(f"Iter {iteration}: gbest = {self.gbest_fitness:.8f}")
            
            # 早停条件
            if self.gbest_fitness < self.tol:
                print(f"Converged at iteration {iteration}")
                break
        
        return self.gbest_position, self.gbest_fitness
```

## 3.2 带有收敛分析的详细版本

```python
class ConvergentPSO:
    def __init__(self, objective, dim, bounds):
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
        self.reset()
    
    def reset(self):
        """重置算法状态"""
        bounds_array = np.array(self.bounds)
        self.positions = np.random.uniform(
            bounds_array[:, 0], bounds_array[:, 1], (30, self.dim)
        )
        self.velocities = np.zeros((30, self.dim))
        self.fitness = np.array([self.objective(p) for p in self.positions])
        self.pbest = self.positions.copy()
        self.pbest_fit = self.fitness.copy()
        
        gbest_idx = np.argmin(self.fitness)
        self.gbest = self.positions[gbest_idx].copy()
        self.gbest_fit = self.fitness[gbest_idx]
        
        self.history = [self.gbest_fit]
    
    def step(self, w=0.729, c1=1.49445, c2=1.49445):
        """单步迭代"""
        r1 = np.random.random((30, self.dim))
        r2 = np.random.random((30, self.dim))
        
        # 速度更新
        self.velocities = (w * self.velocities + 
            c1 * r1 * (self.pbest - self.positions) +
            c2 * r2 * (self.gbest - self.positions)
        )
        
        # 位置更新
        self.positions += self.velocities
        bounds = np.array(self.bounds)
        self.positions = np.clip(self.positions, bounds[:, 0], bounds[:, 1])
        
        # 评估
        self.fitness = np.array([self.objective(p) for p in self.positions])
        
        # 更新pbest
        improved = self.fitness < self.pbest_fit
        self.pbest[improved] = self.positions[improved]
        self.pbest_fit[improved] = self.fitness[improved]
        
        # 更新gbest
        gbest_idx = np.argmin(self.fitness)
        if self.fitness[gbest_idx] < self.gbest_fit:
            self.gbest = self.positions[gbest_idx].copy()
            self.gbest_fit = self.fitness[gbest_idx]
        
        self.history.append(self.gbest_fit)
```

---

# 四、全局版与局部版PSO

## 4.1 gbest模型（全局版）

gbest模型使用全局最优粒子作为社会信息的来源：

```python
class GBestPSO(ParticleSwarmOptimizer):
    """全局最优模型PSO"""
    
    def update_velocities(self):
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
        social = self.c2 * r2 * (self.gbest_position - self.positions)
        
        self.velocities = self.w * self.velocities + cognitive + social
        
        # 限制速度
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
```

**特点**：
- 收敛速度快
- 适合单峰问题
- 容易陷入局部最优

## 4.2 lbest模型（局部版）

lbest模型使用环形拓扑结构，每个粒子只与邻居交换信息：

```python
class LBestPSO(ParticleSwarmOptimizer):
    """局部最优模型PSO"""
    
    def __init__(self, *args, k=3, **kwargs):
        super().__init__(*args, **kwargs)
        self.k = k  # 每个粒子的邻居数量（每侧）
        self.lbest = np.zeros(self.dim)
    
    def initialize_topology(self):
        """初始化环形拓扑"""
        self.neighbors = {}
        for i in range(self.n_particles):
            neighbors = [(i + j) % self.n_particles for j in range(-self.k, self.k+1)]
            self.neighbors[i] = neighbors
    
    def find_lbest(self):
        """找到每个粒子的局部最优"""
        for i in range(self.n_particles):
            neighbor_fitness = self.pbest_fitness[self.neighbors[i]]
            best_neighbor = self.neighbors[i][np.argmin(neighbor_fitness)]
            self.lbest = self.pbest_positions[best_neighbor]
    
    def update_velocities(self):
        self.find_lbest()
        
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
        
        # 使用lbest替代gbest
        lbest_expanded = np.array([self.pbest_positions[self.neighbors[i][
            np.argmin(self.pbest_fitness[self.neighbors[i]])]]
            for i in range(self.n_particles)])
        social = self.c2 * r2 * (lbest_expanded - self.positions)
        
        self.velocities = self.w * self.velocities + cognitive + social
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
```

**拓扑结构类型**：

```
环形拓扑（lbest k=1）:
    ○——○——○——○——○——○——○
    ↑__________________|

网格拓扑:
    ○ ○ ○
    ○ ● ○
    ○ ○ ○

冯诺依曼拓扑:
    ○ ○ ○
    ○ ● ○
    ○ ○ ○
```

## 4.3 拓扑结构对比

| 拓扑类型 | 收敛速度 | 全局搜索 | 稳定性 |
|---------|---------|---------|--------|
| gbest | 最快 | 最弱 | 较差 |
| Ring | 较慢 | 最强 | 较好 |
| Grid | 中等 | 中等 | 中等 |
| Von Neumann | 中等 | 较强 | 较好 |

> [!note]
> 局部版PSO虽然收敛速度较慢，但在复杂多峰问题上表现出更强的全局搜索能力，能够有效避免早熟收敛。

---

# 五、惯性权重与学习因子

## 5.1 线性递减惯性权重

Shi和Eberhart提出的线性递减策略：

```python
class LinearDecreasingWPSO(ParticleSwarmOptimizer):
    def __init__(self, *args, w_max=0.9, w_min=0.4, **kwargs):
        super().__init__(*args, **kwargs)
        self.w_max = w_max
        self.w_min = w_min
    
    def get_inertia_weight(self, iteration):
        """线性递减"""
        return self.w_max - (self.w_max - self.w_min) * iteration / self.max_iter
```

**数学表达**：

$$
w(t) = w_{\max} - \frac{w_{\max} - w_{\min}}{T_{\max}} \cdot t
$$

## 5.2 非线性递减惯性权重

```python
class NonLinearWPSO(ParticleSwarmOptimizer):
    def get_inertia_weight(self, iteration):
        t = iteration / self.max_iter
        # 指数递减
        return 0.9 * 0.99 ** iteration
```

## 5.3 自适应惯性权重

```python
class AdaptiveWPSO(ParticleSwarmOptimizer):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.w = 0.9  # 初始值
        self.fitness_improvement = []
    
    def update_adaptive_w(self, iteration):
        """基于适应度改进率的自适应调整"""
        if len(self.history) >= 10:
            recent_improvement = self.history[-1] - self.history[-10]
            if recent_improvement < 1e-6:
                self.w *= 1.1  # 增加探索
            elif recent_improvement > 0.1:
                self.w *= 0.9  # 增加开发
```

## 5.4 学习因子的影响

| 参数配置 | 特性 | 适用场景 |
|---------|------|---------|
| $c_1 > c_2$ | 强调个体经验 | 探索阶段 |
| $c_1 < c_2$ | 强调群体经验 | 开发阶段 |
| $c_1 = c_2$ | 平衡型 | 通用 |
| $c_1 = c_2 = 1.49445$ | Clerc标准配置 | 理论最优 |

**Clerc建议的标准参数**：

$$
c_1 = c_2 = \frac{1}{2} + \ln(2) \approx 1.19345
$$

> [!tip]
> 学习因子的自适应调整可以帮助PSO在探索和开发之间保持平衡。一种常见策略是在迭代早期增大$c_1$以鼓励探索，后期增大$c_2$以加速收敛。

---

# 六、离散与二进制PSO

## 6.1 二进制PSO

Kennedy和Eberhart于1997年提出的离散版本：

```python
class BinaryPSO:
    def __init__(self, objective, dim, n_particles=30):
        self.objective = objective
        self.dim = dim
        self.n_particles = n_particles
    
    def sigmoid(self, v):
        """Sigmoid函数"""
        return 1 / (1 + np.exp(-np.clip(v, -500, 500)))
    
    def initialize(self):
        self.positions = np.random.randint(0, 2, (self.n_particles, self.dim))
        self.velocities = np.random.uniform(-6, 6, (self.n_particles, self.dim))
        self.fitness = np.array([self.objective(p) for p in self.positions])
        self.pbest = self.positions.copy()
        self.pbest_fit = self.fitness.copy()
        
        gbest_idx = np.argmin(self.fitness)
        self.gbest = self.positions[gbest_idx].copy()
        self.gbest_fit = self.fitness[gbest_idx]
    
    def update_velocities(self, c1=2, c2=2):
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        self.velocities += (c1 * r1 * (self.pbest - self.positions) +
                           c2 * r2 * (self.gbest - self.positions))
        
        # 速度限制
        self.velocities = np.clip(self.velocities, -4, 4)
    
    def update_positions(self):
        """位置（二进制化）"""
        probability = self.sigmoid(self.velocities)
        self.positions = (np.random.random((self.n_particles, self.dim)) < probability).astype(int)
    
    def step(self):
        self.update_velocities()
        self.update_positions()
        self.fitness = np.array([self.objective(p) for p in self.positions])
        
        # 更新pbest和gbest
        improved = self.fitness < self.pbest_fit
        self.pbest[improved] = self.positions[improved]
        self.pbest_fit[improved] = self.fitness[improved]
        
        gbest_idx = np.argmin(self.fitness)
        if self.fitness[gbest_idx] < self.gbest_fit:
            self.gbest = self.positions[gbest_idx].copy()
            self.gbest_fit = self.fitness[gbest_idx]
```

## 6.2 离散PSO（整数编码）

```python
class DiscretePSO:
    """离散变量PSO"""
    
    def __init__(self, objective, dim, n_categories):
        self.objective = objective
        self.dim = dim
        self.n_categories = n_categories  # 每个维度的离散值数量
    
    def initialize(self):
        self.positions = np.random.randint(0, self.n_categories, 
                                          (self.n_particles, self.dim))
        self.velocities = np.random.random((self.n_particles, self.dim))
        # ... 初始化其他属性 ...
    
    def update_positions(self):
        """基于概率的位置更新"""
        for i in range(self.n_particles):
            for d in range(self.dim):
                if self.velocities[i, d] > 0.5:
                    self.positions[i, d] = self.pbest[i, d]
                if self.velocities[i, d] > 0.75:
                    self.positions[i, d] = self.gbest[d]
```

## 6.3 排列问题PSO

```python
class PermutationPSO:
    """用于旅行商等排列问题的PSO"""
    
    def __init__(self, objective, n_cities):
        self.objective = objective
        self.n_cities = n_cities
    
    def initialize(self):
        """随机排列初始化"""
        self.positions = np.array([
            np.random.permutation(self.n_cities) 
            for _ in range(self.n_particles)
        ])
        self.pbest = self.positions.copy()
        self.gbest = self.positions[0].copy()
    
    def crossover_position(self, p1, p2):
        """位置交叉产生新排列"""
        n = len(p1)
        # 随机选择一段保留
        start, end = sorted(random.sample(range(n), 2))
        
        child = np.zeros(n, dtype=int)
        child[start:end] = p1[start:end]
        
        # 填补剩余位置
        remaining = [city for city in p2 if city not in child]
        idx = 0
        for i in range(n):
            if child[i] == 0:
                child[i] = remaining[idx]
                idx += 1
        
        return child
```

---

# 七、PSO变体与改进

## 7.1 协同PSO

```python
class CooperativePSO(ParticleSwarmOptimizer):
    """
    协同PSO：将高维问题分解为多个低维子问题
    """
    def __init__(self, *args, sub_dim=5, **kwargs):
        super().__init__(*args, **kwargs)
        self.sub_dim = sub_dim
        self.n_sub_swarms = self.dim // sub_dim
    
    def evaluate_cooperative(self, full_position):
        """评估完整解"""
        return self.objective(full_position)
    
    def step(self):
        """分阶段更新"""
        # 更新每个子种群
        for sub_idx in range(self.n_sub_swarms):
            sub_start = sub_idx * self.sub_dim
            sub_end = sub_start + self.sub_dim
            
            # 只更新当前维度
            # ...
```

## 7.2 混沌PSO

```python
import numpy as np

class ChaoticPSO(ParticleSwarmOptimizer):
    """
    使用混沌序列代替随机数
    """
    def __init__(self, *args, chaos_type='logistic', **kwargs):
        super().__init__(*args, **kwargs)
        self.chaos_type = chaos_type
        self.chaos_param = 0.7  # 混沌参数
    
    def logistic_map(self, x):
        """Logistic混沌映射"""
        return self.chaos_param * x * (1 - x)
    
    def generate_chaos(self, n):
        """生成混沌序列"""
        x = np.random.random()
        sequence = []
        for _ in range(n):
            x = self.logistic_map(x)
            sequence.append(x)
        return np.array(sequence)
    
    def step(self):
        """使用混沌序列更新"""
        # 生成混沌序列
        chaos_r1 = self.generate_chaos(self.n_particles * self.dim)
        chaos_r2 = self.generate_chaos(self.n_particles * self.dim)
        
        r1 = chaos_r1.reshape(self.n_particles, self.dim)
        r2 = chaos_r2.reshape(self.n_particles, self.dim)
        
        # 更新速度
        self.velocities = (self.w * self.velocities +
            self.c1 * r1 * (self.pbest_positions - self.positions) +
            self.c2 * r2 * (self.gbest_position - self.positions)
        )
        
        # ...
```

## 7.3 量子PSO

```python
class QuantumPSO(ParticleSwarmOptimizer):
    """
    量子粒子群优化
    引入量子力学原理
    """
    def __init__(self, *args, beta=0.5, **kwargs):
        super().__init__(*args, **kwargs)
        self.beta = beta  # 收缩-扩张因子
    
    def update_quantum_position(self, i):
        """量子位置更新"""
        phi1 = np.random.random(self.dim)
        phi2 = np.random.random(self.dim)
        
        p = (phi1 * self.pbest_positions[i] + phi2 * self.gbest_position) / (phi1 + phi2)
        
        # 量子旋转
        u = np.random.random(self.dim)
        
        new_position = p + self.beta * np.abs(self.positions[i] - p) * np.log(1/u)
        
        return new_position
    
    def step(self):
        """量子PSO步骤"""
        for i in range(self.n_particles):
            # 量子位置更新
            self.positions[i] = self.update_quantum_position(i)
        
        # 评估并更新pbest和gbest
        self.evaluate()
```

## 7.4 多目标PSO

```python
class MultiObjectivePSO:
    """
    多目标粒子群优化
    """
    def __init__(self, objectives, dim, bounds, n_particles=100):
        self.objectives = objectives
        self.dim = dim
        self.bounds = bounds
        self.n_particles = n_particles
    
    def dominates(self, x1, x2):
        """判断x1是否支配x2"""
        f1 = np.array([f(x1) for f in self.objectives])
        f2 = np.array([f(x2) for f in self.objectives])
        return all(f1 <= f2) and any(f1 < f2)
    
    def update_leader_archive(self):
        """更新非支配解存档"""
        self.archive = []
        for particle in self.particles:
            is_dominated = False
            to_remove = []
            
            for i, archived in enumerate(self.archive):
                if self.dominates(particle, archived):
                    to_remove.append(i)
                elif self.dominates(archived, particle):
                    is_dominated = True
            
            if not is_dominated:
                self.archive.append(particle)
                for i in reversed(to_remove):
                    self.archive.pop(i)
    
    def select_leader(self):
        """从存档中选择领导者（拥挤距离）"""
        # 拥挤距离计算...
        return random.choice(self.archive)
```

---

# 八、收敛性分析

## 8.1 稳定性分析

PSO的收敛性可通过分析速度更新公式的稳定性来确定。将速度更新公式简化为：

$$
v_i(t+1) = w \cdot v_i(t) + c_1 \cdot r_1 \cdot (p_i - x_i(t)) + c_2 \cdot r_2 \cdot (g - x_i(t))
$$

**收敛条件**（Clerc & Kennedy, 2002）：

$$
| w - 1 | < \sqrt{c_1 \cdot p + c_2 \cdot q}
$$

其中$p, q$为认知和社会影响的权重。

## 8.2 参数约束

为保证算法收敛，参数需满足：

$$
w < \frac{3 - \sqrt{5}}{2} \approx 0.382
$$

或采用收缩-扩张因子$\chi$：

$$
\chi = \frac{2}{|2 - \phi - \sqrt{\phi^2 - 4\phi}|}, \quad \phi = c_1 + c_2 > 4
$$

## 8.3 收敛速度估计

```python
class ConvergenceAnalyzer:
    def analyze(self, history):
        """分析PSO收敛行为"""
        history = np.array(history)
        
        # 计算收敛代数
        converged_gen = self.find_convergence_generation(history)
        
        # 计算收敛率
        convergence_rate = self.calculate_convergence_rate(history)
        
        # 分析探索-开发平衡
        exploration_score = self.estimate_exploration(history)
        
        return {
            'converged_at': converged_gen,
            'convergence_rate': convergence_rate,
            'exploration_score': exploration_score
        }
```

---

# 九、应用实例

## 9.1 函数优化

```python
def optimize_function():
    """使用PSO优化测试函数"""
    # Sphere函数
    sphere = lambda x: sum(x**2)
    
    optimizer = ParticleSwarmOptimizer(
        objective=sphere,
        dim=10,
        bounds=[(-10, 10)] * 10,
        n_particles=30,
        max_iter=1000
    )
    
    best_pos, best_fit = optimizer.run()
    print(f"Best position: {best_pos}")
    print(f"Best fitness: {best_fit}")
    
    return best_pos, best_fit

# Rosenbrock函数
def rosenbrock(x):
    return sum(100 * (x[i+1] - x[i]**2)**2 + (x[i] - 1)**2 for i in range(len(x)-1))

# Ackley函数
def ackley(x):
    return -20 * np.exp(-0.2 * np.sqrt(np.mean(x**2))) - \
           np.exp(np.mean(np.cos(2*np.pi*x))) + 20 + np.e
```

## 9.2 神经网络训练

```python
class PSOTrainableNN:
    def __init__(self, nn_architecture):
        self.nn = nn_architecture
        self.dim = sum(w.size for w in nn.get_weights())
    
    def fitness_function(self, weights_flat):
        """使用扁平化权重向量作为PSO的位置"""
        self.nn.set_weights(weights_flat)
        predictions = self.nn.predict(X_train)
        loss = self.nn.compute_loss(predictions, y_train)
        return loss
    
    def train(self, X_train, y_train, n_particles=30, max_iter=500):
        self.nn.set_training_data(X_train, y_train)
        
        optimizer = ParticleSwarmOptimizer(
            objective=self.fitness_function,
            dim=self.dim,
            bounds=[(-1, 1)] * self.dim,
            n_particles=n_particles,
            max_iter=max_iter
        )
        
        best_weights, best_loss = optimizer.run()
        self.nn.set_weights(best_weights)
        
        return best_loss
```

## 9.3 聚类优化

```python
class PSOKMeans:
    def __init__(self, n_clusters, max_iter=100):
        self.n_clusters = n_clusters
        self.max_iter = max_iter
    
    def fitness(self, centers, X):
        """计算聚类质量（簇内距离和）"""
        labels = self.assign_labels(centers, X)
        distance = 0
        for k in range(self.n_clusters):
            cluster_points = X[labels == k]
            if len(cluster_points) > 0:
                distance += np.sum((cluster_points - centers[k])**2)
        return distance
    
    def pso_optimize(self, X):
        dim = X.shape[1] * self.n_clusters
        
        optimizer = ParticleSwarmOptimizer(
            objective=lambda c: self.fitness(c.reshape(self.n_clusters, -1), X),
            dim=dim,
            bounds=[(X.min(), X.max())] * dim,
            n_particles=30,
            max_iter=self.max_iter
        )
        
        best_centers = optimizer.run()[0]
        return best_centers.reshape(self.n_clusters, -1)
```

# 十、PSO的理论深度分析

## 10.1 动力学系统视角

### 10.1.1 PSO作为非线性动力学系统

PSO的更新公式可以被建模为一个高维非线性离散动力系统：

$$
\mathbf{x}_i(t+1) = f(\mathbf{x}_i(t), \mathbf{v}_i(t), \mathbf{p}_i, \mathbf{g})
$$

**固定点分析**：当算法收敛时，有：

$$
\mathbf{x}^* = \mathbf{x}^* + w \cdot \mathbf{v}^* + c_1 r_1 (\mathbf{p}_i - \mathbf{x}^*) + c_2 r_2 (\mathbf{g} - \mathbf{x}^*)
$$

这要求$\mathbf{v}^* = \mathbf{0}$，即速度收敛到零。

### 10.1.2 相轨迹分析

```python
class PhaseTrajectoryAnalyzer:
    """分析PSO粒子的相轨迹"""
    
    def __init__(self, optimizer):
        self.optimizer = optimizer
        self.position_history = []
        self.velocity_history = []
    
    def track_trajectory(self, particle_idx, steps=100):
        """跟踪单个粒子的相轨迹"""
        self.optimizer.reset()
        
        for _ in range(steps):
            self.position_history.append(
                self.optimizer.positions[particle_idx].copy()
            )
            self.velocity_history.append(
                self.optimizer.velocities[particle_idx].copy()
            )
            self.optimizer.step()
        
        return np.array(self.position_history), np.array(self.velocity_history)
    
    def plot_phase_diagram(self):
        """绘制相图"""
        import matplotlib.pyplot as plt
        
        pos = np.array(self.position_history)
        vel = np.array(self.velocity_history)
        
        # 使用前两个维度
        fig, axes = plt.subplots(2, 2, figsize=(12, 10))
        
        # 位置-时间图
        axes[0, 0].plot(pos[:, 0], pos[:, 1])
        axes[0, 0].set_xlabel('x1')
        axes[0, 0].set_ylabel('x2')
        axes[0, 0].set_title('Position Trajectory')
        
        # 速度-时间图
        axes[0, 1].plot(vel[:, 0], vel[:, 1])
        axes[0, 1].set_xlabel('v1')
        axes[0, 1].set_ylabel('v2')
        axes[0, 1].set_title('Velocity Trajectory')
        
        # 速度范数衰减
        vel_norms = np.linalg.norm(vel, axis=1)
        axes[1, 0].plot(vel_norms)
        axes[1, 0].set_xlabel('Iteration')
        axes[1, 0].set_ylabel('||v||')
        axes[1, 0].set_title('Velocity Magnitude Decay')
        
        # 相轨迹（速度vs位置）
        axes[1, 1].scatter(pos[:, 0], vel[:, 0], c=range(len(pos)))
        axes[1, 1].set_xlabel('Position x1')
        axes[1, 1].set_ylabel('Velocity v1')
        axes[1, 1].set_title('Phase Plot')
        
        plt.tight_layout()
        plt.show()
```

### 10.1.3 吸引域分析

吸引域（Basin of Attraction）定义了初始条件最终收敛到的吸引子位置：

```python
class BasinOfAttractionAnalyzer:
    """
    分析PSO的吸引域
    """
    def __init__(self, optimizer, objective):
        self.optimizer = optimizer
        self.objective = objective
    
    def sample_attraction_basin(self, n_samples=1000):
        """采样吸引域"""
        basins = {}
        
        for _ in range(n_samples):
            # 随机初始化
            self.optimizer.reset()
            initial_pos = self.optimizer.positions[0].copy()
            
            # 运行到收敛
            for _ in range(self.optimizer.max_iter):
                self.optimizer.step()
            
            # 记录收敛位置
            final_pos = tuple(self.optimizer.gbest_position.round(4))
            if final_pos not in basins:
                basins[final_pos] = []
            basins[final_pos].append(initial_pos)
        
        return basins
    
    def visualize_basins(self, basins):
        """可视化吸引域"""
        import matplotlib.pyplot as plt
        
        colors = plt.cm.rainbow(np.linspace(0, 1, len(basins)))
        
        for (final_pos, initial_points), color in zip(basins.items(), colors):
            initials = np.array(initial_points)
            plt.scatter(initials[:, 0], initials[:, 1], 
                       c=[color], alpha=0.5, s=10)
            plt.scatter(final_pos[0], final_pos[1], c=[color], 
                       marker='*', s=200, edgecolors='black')
        
        plt.xlabel('x1')
        plt.ylabel('x2')
        plt.title('Attraction Basins')
        plt.show()
```

## 10.2 概率分析

### 10.2.1 收敛概率分析

```python
class ConvergenceProbabilityAnalyzer:
    """分析PSO的全局收敛概率"""
    
    def __init__(self, objective, dim, bounds, n_trials=100):
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
        self.n_trials = n_trials
    
    def estimate_convergence_probability(self, tolerance=1e-6):
        """估计找到全局最优的概率"""
        successes = 0
        
        for _ in range(self.n_trials):
            pso = ParticleSwarmOptimizer(
                self.objective, self.dim, self.bounds,
                max_iter=500
            )
            _, fitness = pso.run()
            
            if fitness < tolerance:
                successes += 1
        
        return successes / self.n_trials
    
    def confidence_interval(self, p, n, confidence=0.95):
        """计算置信区间"""
        from scipy import stats
        
        z = stats.norm.ppf((1 + confidence) / 2)
        se = np.sqrt(p * (1 - p) / n)
        
        return (p - z * se, p + z * se)
```

### 10.2.2 期望收敛时间分析

```python
class ExpectedConvergenceTime:
    """估计期望收敛时间"""
    
    def analyze_convergence_time(self, optimizer, true_optimum):
        """分析收敛时间统计"""
        convergence_times = []
        
        for _ in range(30):
            optimizer.reset()
            converged = False
            
            for t in range(optimizer.max_iter):
                optimizer.step()
                
                distance = np.linalg.norm(
                    optimizer.gbest_position - true_optimum
                )
                
                if distance < 1e-4:
                    convergence_times.append(t)
                    converged = True
                    break
            
            if not converged:
                convergence_times.append(optimizer.max_iter)
        
        return {
            'mean': np.mean(convergence_times),
            'std': np.std(convergence_times),
            'median': np.median(convergence_times),
            'min': np.min(convergence_times),
            'max': np.max(convergence_times),
        }
```

## 10.3 邻域拓扑的数学分析

### 10.3.1 拓扑结构与信息流动

不同的拓扑结构决定了粒子间的信息流动模式：

```python
class TopologyAnalyzer:
    """分析不同拓扑结构的影响"""
    
    TOPOLOGIES = {
        'gbest': 'global_best',
        'lbest_ring': 'local_best_ring',
        'lbest_star': 'local_best_star',
        'pyramid': 'pyramid',
        'voronoi': 'voronoi',
    }
    
    def analyze_information_flow(self, topology):
        """分析拓扑中的信息流动"""
        n_particles = 30
        adjacency_matrix = self.build_adjacency(topology, n_particles)
        
        # 计算图的连通性
        eigenvalues = np.linalg.eigvalsh(adjacency_matrix)
        
        # 代数连通性（第二小特征值）
        algebraic_connectivity = sorted(eigenvalues)[1]
        
        # 直径
        diameter = self.calculate_diameter(adjacency_matrix)
        
        # 平均路径长度
        avg_path_length = self.calculate_avg_path(adjacency_matrix)
        
        return {
            'algebraic_connectivity': algebraic_connectivity,
            'diameter': diameter,
            'avg_path_length': avg_path_length,
        }
    
    def build_adjacency(self, topology, n):
        """构建邻接矩阵"""
        adj = np.zeros((n, n))
        
        if topology == 'lbest_ring':
            for i in range(n):
                adj[i, (i-1) % n] = 1
                adj[i, (i+1) % n] = 1
        elif topology == 'gbest':
            adj[:] = 1  # 全连接
        # ... 其他拓扑
        
        return adj
```

### 10.3.2 小世界拓扑

```python
class SmallWorldPSO(ParticleSwarmOptimizer):
    """
    小世界拓扑PSO
    结合了局部搜索和全局搜索的优势
    """
    def __init__(self, *args, rewire_prob=0.1, **kwargs):
        super().__init__(*args, **kwargs)
        self.rewire_prob = rewire_prob
        self.build_ring_topology()
    
    def build_ring_topology(self):
        """构建环形基础拓扑"""
        self.neighbors = {}
        for i in range(self.n_particles):
            self.neighbors[i] = [(i-1) % self.n_particles,
                                 (i+1) % self.n_particles]
    
    def rewire_edges(self):
        """随机重连边"""
        for i in range(self.n_particles):
            for j_idx, j in enumerate(self.neighbors[i]):
                if random.random() < self.rewire_prob:
                    # 随机选择新邻居
                    new_neighbor = random.choice([
                        k for k in range(self.n_particles)
                        if k != i and k not in self.neighbors[i]
                    ])
                    self.neighbors[i][j_idx] = new_neighbor
    
    def get_best_neighbor(self, i):
        """获取邻居中最优的粒子"""
        neighbor_indices = self.neighbors[i]
        neighbor_fitness = self.pbest_fitness[neighbor_indices]
        best_idx = np.argmin(neighbor_fitness)
        return self.pbest_positions[neighbor_indices[best_idx]]
```

---

# 十一、高级变体与前沿研究

## 11.1 耗散粒子群优化（DPSO）

### 11.1.1 理论基础

耗散PSO通过引入负熵流来抵抗早熟收敛：

```python
class DissipativePSO(ParticleSwarmOptimizer):
    """
    耗散粒子群优化
    通过随机扰动引入熵
    """
    def __init__(self, *args, entropy_threshold=1e-6, 
                 perturbation_strength=0.1, **kwargs):
        super().__init__(*args, **kwargs)
        self.entropy_threshold = entropy_threshold
        self.perturbation_strength = perturbation_strength
    
    def calculate_entropy(self):
        """计算粒子群熵"""
        # 使用速度分布的熵
        velocities = self.velocities.flatten()
        hist, _ = np.histogram(velocities, bins=50, density=True)
        hist = hist[hist > 0]  # 去除零
        return -np.sum(hist * np.log(hist + 1e-10))
    
    def inject_entropy(self):
        """注入熵以跳出局部最优"""
        entropy = self.calculate_entropy()
        
        if entropy < self.entropy_threshold:
            # 随机选择粒子添加扰动
            n_perturbed = max(1, self.n_particles // 10)
            perturbed_indices = random.sample(
                range(self.n_particles), n_perturbed
            )
            
            for idx in perturbed_indices:
                # 添加随机扰动
                perturbation = np.random.uniform(
                    -self.perturbation_strength,
                    self.perturbation_strength,
                    self.dim
                )
                self.positions[idx] += perturbation
                
                # 重新评估
                self.fitness[idx] = self.objective(self.positions[idx])
                
                # 更新pbest
                if self.fitness[idx] < self.pbest_fitness[idx]:
                    self.pbest_positions[idx] = self.positions[idx].copy()
                    self.pbest_fitness[idx] = self.fitness[idx]
    
    def step(self):
        self.update_velocities()
        self.update_positions()
        self.evaluate()
        self.inject_entropy()  # 关键差异
```

### 11.1.2 耗散机制的效果分析

```python
class EntropyAnalysis:
    """分析熵注入的效果"""
    
    def compare_with_standard(self, objective, dim=10, n_runs=20):
        """对比标准PSO和耗散PSO"""
        standard_results = []
        dissipative_results = []
        
        for _ in range(n_runs):
            # 标准PSO
            pso_std = ParticleSwarmOptimizer(
                objective, dim, [(-10, 10)] * dim
            )
            _, fitness_std = pso_std.run()
            standard_results.append(fitness_std)
            
            # 耗散PSO
            pso_diss = DissipativePSO(
                objective, dim, [(-10, 10)] * dim
            )
            _, fitness_diss = pso_diss.run()
            dissipative_results.append(fitness_diss)
        
        return {
            'standard': {
                'mean': np.mean(standard_results),
                'std': np.std(standard_results),
            },
            'dissipative': {
                'mean': np.mean(dissipative_results),
                'std': np.std(dissipative_results),
            },
            'improvement': np.mean(standard_results) - np.mean(dissipative_results)
        }
```

## 11.2 协同进化PSO

### 11.2.1 协同进化框架

```python
class CoevolutionPSO:
    """
    协同进化PSO
    将复杂问题分解为多个子问题协同优化
    """
    def __init__(self, objective, dim, bounds, n_subpopulations=4):
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
        self.n_subpopulations = n_subpopulations
        
        # 子种群规模
        self.sub_dim = dim // n_subpopulations
        self.n_particles_per_sub = 20
        
        # 创建子种群
        self.sub_swarms = []
        for i in range(n_subpopulations):
            sub_bounds = bounds[i*self.sub_dim:(i+1)*self.sub_dim]
            self.sub_swarms.append(
                ParticleSwarmOptimizer(
                    lambda x, si=i: self._evaluate_subsolution(x, si),
                    self.sub_dim, sub_bounds,
                    n_particles=self.n_particles_per_sub
                )
            )
    
    def _evaluate_subsolution(self, sub_pos, sub_idx):
        """评估子解（需要组合其他子解）"""
        # 从其他子种群获取当前最优
        combined = np.zeros(self.dim)
        
        for i, swarm in enumerate(self.sub_swarms):
            if i == sub_idx:
                combined[sub_idx*self.sub_dim:(i+1)*self.sub_dim] = sub_pos
            else:
                combined[i*self.sub_dim:(i+1)*self.sub_dim] = \
                    swarm.gbest_position
        
        return self.objective(combined)
    
    def run(self, max_iter=1000):
        """协同进化主循环"""
        for swarm in self.sub_swarms:
            swarm.initialize()
        
        best_fitness = float('inf')
        best_solution = None
        
        for iteration in range(max_iter):
            for swarm in self.sub_swarms:
                swarm.update_velocities()
                swarm.update_positions()
                swarm.evaluate()
            
            # 重组并评估全局适应度
            global_best = self._reconstruct_solution()
            fitness = self.objective(global_best)
            
            if fitness < best_fitness:
                best_fitness = fitness
                best_solution = global_best.copy()
        
        return best_solution, best_fitness
    
    def _reconstruct_solution(self):
        """重组全局解"""
        solution = np.zeros(self.dim)
        for i, swarm in enumerate(self.sub_swarms):
            solution[i*self.sub_dim:(i+1)*self.sub_dim] = \
                swarm.gbest_position
        return solution
```

## 11.3 稀疏PSO

### 11.3.1 稀疏编码优化

```python
class SparsePSO:
    """
    稀疏PSO
    专门用于稀疏优化问题
    """
    def __init__(self, objective, dim, sparsity_target=0.1):
        self.objective = objective
        self.dim = dim
        self.sparsity_target = sparsity_target
    
    def sparse_fitness(self, x, alpha=0.01):
        """
        稀疏惩罚适应度
        L1正则化
        """
        reconstruction_error = self.objective(x)
        sparsity_penalty = alpha * np.sum(np.abs(x))
        return reconstruction_error + sparsity_penalty
    
    def threshold_solution(self, x, threshold=1e-3):
        """阈值化使解更稀疏"""
        x_sparse = x.copy()
        x_sparse[np.abs(x_sparse) < threshold] = 0
        return x_sparse
    
    def optimize(self, n_particles=30, max_iter=500):
        """运行稀疏PSO"""
        bounds = [(-10, 10)] * self.dim
        
        optimizer = ParticleSwarmOptimizer(
            lambda x: self.sparse_fitness(x),
            self.dim, bounds, n_particles=n_particles
        )
        
        best_pos, _ = optimizer.run()
        best_sparse = self.threshold_solution(best_pos)
        
        # 评估原始目标函数
        original_fitness = self.objective(best_sparse)
        actual_sparsity = 1 - np.count_nonzero(best_sparse) / self.dim
        
        return {
            'solution': best_sparse,
            'sparsity': actual_sparsity,
            'fitness': original_fitness
        }
```

## 11.4 批量适应度PSO

### 11.4.1 批量评估策略

```python
class BatchFitnessPSO(ParticleSwarmOptimizer):
    """
    批量适应度评估PSO
    利用向量化加速适应度计算
    """
    def __init__(self, objective_vectorized, *args, **kwargs):
        super().__init__(lambda x: objective_vectorized(x[None, :])[0], 
                        *args, **kwargs)
        self.objective_vectorized = objective_vectorized
    
    def evaluate_batch(self):
        """批量评估所有粒子"""
        # 向量化评估
        fitness_values = self.objective_vectorized(self.positions)
        self.fitness = fitness_values
        
        # 更新pbest
        improved = self.fitness < self.pbest_fitness
        self.pbest_positions[improved] = self.positions[improved]
        self.pbest_fitness[improved] = self.fitness[improved]
        
        # 更新gbest
        gbest_idx = np.argmin(self.fitness)
        if self.fitness[gbest_idx] < self.gbest_fitness:
            self.gbest_position = self.positions[gbest_idx].copy()
            self.gbest_fitness = self.fitness[gbest_idx]


class VectorizedFitnessExample:
    """向量化的适应度函数示例"""
    
    @staticmethod
    def rastrigin_batch(X):
        """
        批量Rastrigin函数
        X shape: (n_particles, dim)
        """
        A = 10
        n = X.shape[1]
        return A * n + np.sum(X**2 - A * np.cos(2 * np.pi * X), axis=1)
    
    @staticmethod
    def rosenbrock_batch(X):
        """
        批量Rosenbrock函数
        """
        x = X[:, :-1]
        y = X[:, 1:]
        return np.sum(100 * (y - x**2)**2 + (1 - x)**2, axis=1)
```

---

# 十二、性能基准与测试套件

## 12.1 标准测试函数集

### 12.1.1 单峰函数

```python
class BenchmarkFunctions:
    """标准基准测试函数"""
    
    @staticmethod
    def sphere(x):
        """Sphere函数 - 最简单的单峰函数"""
        return np.sum(x**2)
    
    @staticmethod
    def rosenbrock(x):
        """Rosenbrock函数 - 经典优化问题"""
        return sum(100*(x[i+1] - x[i]**2)**2 + (1-x[i])**2 
                   for i in range(len(x)-1))
    
    @staticmethod
    def schwefel(x):
        """Schwefel函数"""
        n = len(x)
        return 418.9829 * n - np.sum(x * np.sin(np.sqrt(np.abs(x))))
    
    @staticmethod
    def step(x):
        """Step函数"""
        return np.sum(np.floor(x + 0.5)**2)
    
    @staticmethod
    def quartic(x):
        """四次函数（含噪声）"""
        n = len(x)
        return np.sum((n - i) * x[i]**4 for i in range(n)) + random.random()
```

### 12.1.2 多峰函数

```python
    @staticmethod
    def rastrigin(x):
        """Rastrigin函数 - 多峰，有大量局部最优"""
        A = 10
        n = len(x)
        return A * n + np.sum(x**2 - A * np.cos(2 * np.pi * x))
    
    @staticmethod
    def griewank(x):
        """Griewank函数"""
        part1 = np.sum(x**2) / 4000
        part2 = np.prod(np.cos(x / np.sqrt(np.arange(1, len(x)+1))))
        return part1 - part2 + 1
    
    @staticmethod
    def ackley(x):
        """Ackley函数"""
        a = 20
        b = 0.2
        c = 2 * np.pi
        
        sum1 = np.sum(x**2)
        sum2 = np.sum(np.cos(c * x))
        
        term1 = -a * np.exp(-b * np.sqrt(sum1 / len(x)))
        term2 = -np.exp(sum2 / len(x))
        
        return term1 + term2 + a + np.e
    
    @staticmethod
    def perm(x, beta=0.5):
        """Perm函数"""
        n = len(x)
        result = 0
        for k in range(1, n + 1):
            temp = 0
            for j in range(1, n + 1):
                temp += (j + beta) * (x[j-1]**k - 1/j**k)
            result += temp**2
        return result
```

### 12.1.3 组合函数

```python
    @staticmethod
    def zakharov(x):
        """Zakharov函数"""
        sum1 = np.sum(x**2)
        sum2 = np.sum(0.5 * (i+1) * x[i] for i in range(len(x)))
        return sum1 + sum2**2 + sum2**4
    
    @staticmethod
    def dixon_price(x):
        """Dixon-Price函数"""
        n = len(x)
        term1 = (x[0] - 1)**2
        term2 = sum((i+2) * (2*x[i]**2 - x[i-1])**2 for i in range(1, n))
        return term1 + term2
```

## 12.2 基准测试框架

```python
class BenchmarkSuite:
    """PSO基准测试套件"""
    
    FUNCTIONS = {
        'sphere': BenchmarkFunctions.sphere,
        'rosenbrock': BenchmarkFunctions.rosenbrock,
        'rastrigin': BenchmarkFunctions.rastrigin,
        'griewank': BenchmarkFunctions.griewank,
        'ackley': BenchmarkFunctions.ackley,
        'schwefel': BenchmarkFunctions.schwefel,
    }
    
    def __init__(self, dimensions=[10, 30, 50]):
        self.dimensions = dimensions
    
    def run_benchmark(self, optimizer_class, n_runs=30):
        """运行完整基准测试"""
        results = {}
        
        for func_name, func in self.FUNCTIONS.items():
            results[func_name] = {}
            
            for dim in self.dimensions:
                runs = []
                
                for _ in range(n_runs):
                    optimizer = optimizer_class(
                        func, dim, [(-100, 100)] * dim,
                        max_iter=1000
                    )
                    _, fitness = optimizer.run()
                    runs.append(fitness)
                
                results[func_name][dim] = {
                    'mean': np.mean(runs),
                    'std': np.std(runs),
                    'min': np.min(runs),
                    'max': np.max(runs),
                    'median': np.median(runs),
                }
        
        return results
    
    def generate_report(self, results, output_path='benchmark_report.md'):
        """生成基准测试报告"""
        report = "# PSO Benchmark Report\n\n"
        
        for func_name in self.FUNCTIONS:
            report += f"## {func_name}\n\n"
            report += "| Dim | Mean | Std | Min | Max |\n"
            report += "|-----|------|-----|-----|-----|\n"
            
            for dim, stats in results[func_name].items():
                report += f"| {dim} | {stats['mean']:.6e} | "
                report += f"{stats['std']:.6e} | {stats['min']:.6e} | "
                report += f"{stats['max']:.6e} |\n"
            
            report += "\n"
        
        with open(output_path, 'w') as f:
            f.write(report)
        
        return report
```

## 12.3 统计分析工具

```python
class StatisticalAnalysis:
    """统计显著性分析"""
    
    @staticmethod
    def wilcoxon_signed_rank_test(algorithm1_results, algorithm2_results):
        """
        Wilcoxon符号秩检验
        配对样本非参数检验
        """
        from scipy import stats
        
        # 差值
        d = np.array(algorithm1_results) - np.array(algorithm2_results)
        
        # 排除零差
        d_nonzero = d[d != 0]
        
        if len(d_nonzero) == 0:
            return {'p_value': 1.0, 'significant': False}
        
        # 计算统计量
        stat, p_value = stats.wilcoxon(algorithm1_results, algorithm2_results)
        
        return {
            'statistic': stat,
            'p_value': p_value,
            'significant': p_value < 0.05,
            'effect_size': np.mean(d) / np.std(d),
        }
    
    @staticmethod
    def friedman_test(all_algorithms_results):
        """
        Friedman检验
        多算法比较
        """
        from scipy import stats
        
        n_problems = len(all_algorithms_results[0])
        k_algorithms = len(all_algorithms_results)
        
        # 计算每个问题的排名
        rankings = []
        for problem_idx in range(n_problems):
            fitnesses = [results[problem_idx] for results in all_algorithms_results]
            ranks = stats.rankdata(fitnesses)
            rankings.append(ranks)
        
        rankings = np.array(rankings)
        
        # Friedman统计量
        mean_ranks = np.mean(rankings, axis=0)
        
        chi2 = (12 * n_problems / (k_algorithms * (k_algorithms + 1))) * \
               np.sum(mean_ranks**2) - 3 * n_problems * (k_algorithms + 1)
        
        # 自由度为k-1
        p_value = 1 - stats.chi2.cdf(chi2, k_algorithms - 1)
        
        return {
            'chi2': chi2,
            'p_value': p_value,
            'mean_ranks': mean_ranks,
        }
```

---

# 十三、工程实践指南

## 13.1 参数调优策略

### 13.1.1 网格搜索

```python
class ParameterGridSearch:
    """PSO参数网格搜索"""
    
    PARAM_GRID = {
        'w': [0.4, 0.5, 0.6, 0.7, 0.8, 0.9],
        'c1': [1.0, 1.5, 2.0, 2.5],
        'c2': [1.0, 1.5, 2.0, 2.5],
        'n_particles': [20, 30, 50, 100],
    }
    
    def __init__(self, objective, dim, bounds):
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
    
    def grid_search(self, n_eval_per_config=10):
        """网格搜索最优参数"""
        from itertools import product
        
        best_fitness = float('inf')
        best_params = None
        all_results = []
        
        param_combinations = product(
            self.PARAM_GRID['w'],
            self.PARAM_GRID['c1'],
            self.PARAM_GRID['c2'],
            self.PARAM_GRID['n_particles']
        )
        
        for w, c1, c2, n_particles in param_combinations:
            fitnesses = []
            
            for _ in range(n_eval_per_config):
                pso = ParticleSwarmOptimizer(
                    self.objective, self.dim, self.bounds,
                    w=w, c1=c1, c2=c2,
                    n_particles=n_particles
                )
                _, fitness = pso.run()
                fitnesses.append(fitness)
            
            mean_fitness = np.mean(fitnesses)
            all_results.append({
                'w': w, 'c1': c1, 'c2': c2, 
                'n_particles': n_particles,
                'mean_fitness': mean_fitness
            })
            
            if mean_fitness < best_fitness:
                best_fitness = mean_fitness
                best_params = {'w': w, 'c1': c1, 'c2': c2, 
                              'n_particles': n_particles}
        
        return best_params, all_results
```

### 13.1.2 自适应参数调整框架

```python
class AdaptiveParameterController:
    """
    自适应参数控制器
    根据搜索进度动态调整参数
    """
    
    def __init__(self, initial_params):
        self.params = initial_params.copy()
        self.history = {
            'fitness': [],
            'diversity': [],
            'params': []
        }
    
    def calculate_diversity(self, positions):
        """计算种群多样性"""
        centroid = np.mean(positions, axis=0)
        distances = np.linalg.norm(positions - centroid, axis=1)
        return np.mean(distances)
    
    def adjust_parameters(self, generation, fitness, positions):
        """根据当前状态调整参数"""
        # 记录历史
        self.history['fitness'].append(fitness)
        self.history['diversity'].append(
            self.calculate_diversity(positions)
        )
        
        # 计算进度
        progress = generation / self.max_iter
        
        # 基于多样性的调整
        current_diversity = self.history['diversity'][-1]
        initial_diversity = self.history['diversity'][0]
        diversity_ratio = current_diversity / (initial_diversity + 1e-10)
        
        # 基于收敛的调整
        if len(self.history['fitness']) > 10:
            recent_improvement = self.history['fitness'][-10] - \
                               self.history['fitness'][-1]
        else:
            recent_improvement = 0
        
        # 自适应规则
        if diversity_ratio < 0.1:
            # 多样性过低，增加探索
            self.params['w'] = min(0.95, self.params['w'] * 1.1)
            self.params['c1'] = min(2.5, self.params['c1'] * 1.05)
            self.params['c2'] = max(0.5, self.params['c2'] * 0.95)
        elif recent_improvement < 1e-8:
            # 停滞，增加扰动
            self.params['w'] = min(0.9, self.params['w'] + 0.05)
        else:
            # 正常搜索
            self.params['w'] *= 0.999
        
        # 限制范围
        self.params['w'] = np.clip(self.params['w'], 0.1, 0.95)
        self.params['c1'] = np.clip(self.params['c1'], 0.5, 2.5)
        self.params['c2'] = np.clip(self.params['c2'], 0.5, 2.5)
        
        self.history['params'].append(self.params.copy())
        
        return self.params
```

## 13.2 早熟收敛检测

### 13.2.1 多样性度量

```python
class PrematureConvergenceDetector:
    """早熟收敛检测器"""
    
    def __init__(self, threshold=1e-6, window_size=20):
        self.threshold = threshold
        self.window_size = window_size
        self.fitness_history = []
    
    def check_diversity(self, positions):
        """检查种群多样性"""
        # 方法1：位置标准差
        position_std = np.std(positions, axis=0)
        mean_std = np.mean(position_std)
        
        # 方法2：到质心的平均距离
        centroid = np.mean(positions, axis=0)
        distances = np.linalg.norm(positions - centroid, axis=1)
        mean_distance = np.mean(distances)
        
        # 方法3：唯一位置比例
        unique_ratio = len(np.unique(positions.round(4), axis=0)) / len(positions)
        
        return {
            'position_std': mean_std,
            'mean_distance': mean_distance,
            'unique_ratio': unique_ratio,
            'is_converged': mean_std < self.threshold
        }
    
    def check_stagnation(self, current_fitness):
        """检查适应度停滞"""
        self.fitness_history.append(current_fitness)
        
        if len(self.fitness_history) < self.window_size:
            return {'stagnant': False}
        
        recent = self.fitness_history[-self.window_size:]
        
        # 检查是否所有值都相同
        if len(set(recent)) == 1:
            return {'stagnant': True, 'reason': 'identical_values'}
        
        # 检查变化是否足够小
        max_change = max(recent) - min(recent)
        if max_change < self.threshold:
            return {'stagnant': True, 'reason': 'minimal_change'}
        
        return {'stagnant': False}
    
    def detect_and_respond(self, positions, fitness):
        """检测并响应早熟"""
        diversity_info = self.check_diversity(positions)
        stagnation_info = self.check_stagnation(fitness)
        
        if diversity_info['is_converged'] or stagnation_info['stagnant']:
            return {
                'premature': True,
                'diversity': diversity_info,
                'stagnation': stagnation_info,
                'action': self.suggest_action()
            }
        
        return {'premature': False}
    
    def suggest_action(self):
        """建议的应对动作"""
        actions = []
        
        # 增加随机扰动
        actions.append('inject_random_particles')
        
        # 重新初始化部分粒子
        actions.append('reinitialize_subpopulation')
        
        # 增大惯性权重
        actions.append('increase_inertia')
        
        # 切换拓扑结构
        actions.append('change_topology')
        
        return actions
```

## 13.3 混合优化策略

### 13.3.1 PSO + 局部搜索

```python
class HybridPSOLocalSearch(ParticleSwarmOptimizer):
    """
    PSO与局部搜索混合
    PSO负责全局探索，局部搜索负责精细调优
    """
    def __init__(self, *args, local_search_freq=50,
                 local_search_iters=20, **kwargs):
        super().__init__(*args, **kwargs)
        self.local_search_freq = local_search_freq
        self.local_search_iters = local_search_iters
    
    def local_search(self, position):
        """使用L-BFGS-B进行局部搜索"""
        from scipy.optimize import minimize
        
        result = minimize(
            self.objective,
            position,
            method='L-BFGS-B',
            options={'maxiter': self.local_search_iters}
        )
        
        return result.x, result.fun
    
    def step(self, iteration):
        # 标准的PSO更新
        self.update_velocities()
        self.update_positions()
        self.evaluate()
        
        # 周期性局部搜索
        if iteration % self.local_search_freq == 0:
            self.apply_local_search()
    
    def apply_local_search(self):
        """对全局最优应用局部搜索"""
        new_pos, new_fitness = self.local_search(self.gbest_position)
        
        if new_fitness < self.gbest_fitness:
            self.gbest_position = new_pos
            self.gbest_fitness = new_fitness
            
            # 更新一个粒子的位置
            worst_idx = np.argmax(self.fitness)
            self.positions[worst_idx] = new_pos
            self.fitness[worst_idx] = new_fitness
            self.pbest_positions[worst_idx] = new_pos
            self.pbest_fitness[worst_idx] = new_fitness
```

### 13.3.2 PSO + 差分进化

```python
class HybridPSODE(ParticleSwarmOptimizer):
    """
    PSO与差分进化混合
    利用DE的变异策略增强PSO的探索能力
    """
    def __init__(self, *args, de_probability=0.3, **kwargs):
        super().__init__(*args, **kwargs)
        self.de_probability = de_probability
    
    def de_mutation(self, i):
        """差分进化变异"""
        NP = self.n_particles
        
        # 选择三个不同于i的个体
        candidates = [j for j in range(NP) if j != i]
        r1, r2, r3 = random.sample(candidates, 3)
        
        F = 0.5  # 缩放因子
        mutant = self.positions[r1] + F * (
            self.positions[r2] - self.positions[r3]
        )
        
        # 边界处理
        mutant = np.clip(mutant,
                        [b[0] for b in self.bounds],
                        [b[1] for b in self.bounds])
        
        return mutant
    
    def hybrid_step(self):
        """混合更新策略"""
        for i in range(self.n_particles):
            if random.random() < self.de_probability:
                # 使用DE变异
                mutant = self.de_mutation(i)
                
                # 评估
                fitness_mutant = self.objective(mutant)
                
                if fitness_mutant < self.fitness[i]:
                    self.positions[i] = mutant
                    self.fitness[i] = fitness_mutant
                    
                    if fitness_mutant < self.pbest_fitness[i]:
                        self.pbest_positions[i] = mutant
                        self.pbest_fitness[i] = fitness_mutant
                        
                        if fitness_mutant < self.gbest_fitness:
                            self.gbest_position = mutant
                            self.gbest_fitness = fitness_mutant
            else:
                # 使用标准PSO更新
                self.update_velocities()
                self.update_positions()
        
        self.evaluate()
```

---

# 十四、深度学习中的应用

## 14.1 超参数优化

```python
class PSOHyperparameterOptimizer:
    """
    使用PSO优化神经网络超参数
    """
    
    def __init__(self, train_data, val_data, search_space):
        self.train_data = train_data
        self.val_data = val_data
        self.search_space = search_space
    
    def decode_particle(self, particle):
        """将粒子编码转换为超参数"""
        params = {}
        
        idx = 0
        for param_name, (low, high) in self.search_space.items():
            params[param_name] = low + particle[idx] * (high - low)
            idx += 1
        
        return params
    
    def evaluate_config(self, params):
        """评估超参数配置"""
        # 构建模型
        model = self.build_model(params)
        
        # 训练
        history = model.fit(
            self.train_data[0], self.train_data[1],
            validation_data=self.val_data,
            epochs=int(params.get('epochs', 10)),
            verbose=0
        )
        
        # 返回验证集损失
        return min(history.history['val_loss'])
    
    def build_model(self, params):
        """根据参数构建模型"""
        import tensorflow as tf
        from tensorflow import keras
        from tensorflow.keras import layers
        
        model = keras.Sequential([
            layers.Dense(int(params['hidden_units']),
                        activation=params.get('activation', 'relu'),
                        input_shape=self.train_data[0].shape[1:]),
            layers.Dropout(params.get('dropout', 0.5)),
            layers.Dense(int(params['hidden_units2']),
                        activation=params.get('activation', 'relu')),
            layers.Dropout(params.get('dropout', 0.5)),
            layers.Dense(10, activation='softmax')
        ])
        
        model.compile(
            optimizer=keras.optimizers.Adam(
                learning_rate=params.get('lr', 0.001)
            ),
            loss='sparse_categorical_crossentropy',
            metrics=['accuracy']
        )
        
        return model
    
    def optimize(self, n_particles=30, max_iter=50):
        """运行超参数优化"""
        dim = len(self.search_space)
        bounds = [(0, 1)] * dim
        
        optimizer = ParticleSwarmOptimizer(
            objective=lambda x: self.evaluate_config(
                self.decode_particle(x)
            ),
            dim=dim,
            bounds=bounds,
            n_particles=n_particles
        )
        
        best_pos, best_fitness = optimizer.run()
        
        return {
            'best_params': self.decode_particle(best_pos),
            'best_val_loss': best_fitness
        }
```

## 14.2 神经网络结构搜索

```python
class PSONeuralArchitectureSearch:
    """
    基于PSO的神经网络架构搜索
    """
    
    def __init__(self, input_dim, output_dim):
        self.input_dim = input_dim
        self.output_dim = output_dim
    
    def encode_architecture(self, particle):
        """编码粒子为网络架构"""
        n_layers = int(3 + particle[0] * 3)  # 3-6层
        architecture = []
        
        idx = 1
        for _ in range(n_layers):
            units = int(32 + particle[idx] * 256)
            activation = ['relu', 'tanh', 'sigmoid'][int(particle[idx+1] * 3)]
            use_bn = particle[idx+2] > 0.5
            use_dropout = particle[idx+3] > 0.5
            dropout_rate = particle[idx+4]
            
            layer_config = {
                'units': units,
                'activation': activation,
                'batch_norm': use_bn,
                'dropout': use_dropout,
                'dropout_rate': dropout_rate
            }
            architecture.append(layer_config)
            idx += 5
        
        return architecture
    
    def build_network(self, architecture):
        """根据架构构建网络"""
        import tensorflow as tf
        from tensorflow.keras import layers, Model
        
        inputs = tf.keras.Input(shape=(self.input_dim,))
        x = inputs
        
        for layer_config in architecture:
            x = layers.Dense(
                layer_config['units'],
                activation=layer_config['activation']
            )(x)
            
            if layer_config['batch_norm']:
                x = layers.BatchNormalization()(x)
            
            if layer_config['dropout']:
                x = layers.Dropout(layer_config['dropout_rate'])(x)
        
        outputs = layers.Dense(self.output_dim, activation='softmax')(x)
        
        return Model(inputs=inputs, outputs=outputs)
    
    def fitness_function(self, particle):
        """评估架构的适应度"""
        architecture = self.encode_architecture(particle)
        model = self.build_network(architecture)
        
        # 简化的评估（实际应用中需要完整的训练和验证）
        model.compile(
            optimizer='adam',
            loss='categorical_crossentropy',
            metrics=['accuracy']
        )
        
        # 训练少量epoch
        # 这里使用随机数据作为示例
        x_train = np.random.randn(1000, self.input_dim)
        y_train = np.random.randint(0, self.output_dim, 1000)
        
        model.fit(x_train, y_train, epochs=3, verbose=0)
        
        # 返回负的准确率（因为我们最小化）
        _, accuracy = model.evaluate(x_train, y_train, verbose=0)
        
        return -accuracy  # 最小化问题
```

---

# 十五、约束优化问题

## 15.1 约束处理技术

### 15.1.1 罚函数法

```python
class ConstrainedPSO(ParticleSwarmOptimizer):
    """
    带约束的PSO
    使用罚函数法处理约束
    """
    
    def __init__(self, objective, constraints, *args,
                 penalty_coef=1000, **kwargs):
        super().__init__(objective, *args, **kwargs)
        self.constraints = constraints
        self.penalty_coef = penalty_coef
    
    def evaluate_constraints(self, x):
        """评估约束违反程度"""
        violations = {
            'inequality': [],
            'equality': []
        }
        
        for constraint in self.constraints.get('ineq', []):
            value = constraint(x)
            violations['inequality'].append(max(0, value))
        
        for constraint in self.constraints.get('eq', []):
            value = abs(constraint(x))
            violations['equality'].append(value)
        
        return violations
    
    def constrained_objective(self, x):
        """带惩罚的目标函数"""
        obj = self.objective(x)
        violations = self.evaluate_constraints(x)
        
        penalty = self.penalty_coef * (
            sum(violations['inequality']) +
            sum(violations['equality'])
        )
        
        return obj + penalty
    
    def is_feasible(self, x):
        """检查解是否可行"""
        violations = self.evaluate_constraints(x)
        total_violation = (
            sum(violations['inequality']) +
            sum(violations['equality'])
        )
        return total_violation < 1e-6
```

### 15.1.2 可行性规则

```python
class FeasibilityRulePSO(ParticleSwarmOptimizer):
    """
    使用可行性规则处理约束
    优先选择可行解
    """
    
    def __init__(self, objective, constraints, *args, **kwargs):
        super().__init__(objective, *args, **kwargs)
        self.constraints = constraints
    
    def dominates(self, x1, x2):
        """判断x1是否支配x2（约束优化版）"""
        v1 = self.constraint_violation(x1)
        v2 = self.constraint_violation(x2)
        f1 = self.objective(x1)
        f2 = self.objective(x2)
        
        # x1可行且x2不可行
        if v1 == 0 and v2 > 0:
            return True
        
        # 都不可行，违反度小的支配违反度大的
        if v1 > 0 and v2 > 0:
            return v1 < v2
        
        # 都可行，目标值小的支配目标值大的
        if v1 == 0 and v2 == 0:
            return f1 < f2
        
        return False
    
    def constraint_violation(self, x):
        """计算总约束违反度"""
        violations = []
        
        for constraint in self.constraints.get('ineq', []):
            violations.append(max(0, constraint(x)))
        
        for constraint in self.constraints.get('eq', []):
            violations.append(abs(constraint(x)))
        
        return sum(violations)
    
    def selection_with_constraints(self):
        """基于可行性规则的选择"""
        # 更新pbest
        for i in range(self.n_particles):
            if self.dominates(self.positions[i], self.pbest_positions[i]):
                self.pbest_positions[i] = self.positions[i].copy()
                self.pbest_fitness[i] = self.fitness[i]
        
        # 更新gbest
        feasible_indices = [
            i for i in range(self.n_particles)
            if self.constraint_violation(self.positions[i]) == 0
        ]
        
        if feasible_indices:
            # 选择最优的可行解
            feasible_fitness = self.pbest_fitness[feasible_indices]
            best_idx = feasible_indices[np.argmin(feasible_fitness)]
            self.gbest_position = self.pbest_positions[best_idx].copy()
            self.gbest_fitness = self.pbest_fitness[best_idx]
        else:
            # 没有可行解，选择违反度最小的
            violations = [self.constraint_violation(self.pbest_positions[i])
                         for i in range(self.n_particles)]
            best_idx = np.argmin(violations)
            self.gbest_position = self.pbest_positions[best_idx].copy()
            self.gbest_fitness = violations[best_idx]
```

## 15.2 约束优化示例

```python
class EngineeringConstraintExample:
    """工程约束优化示例"""
    
    @staticmethod
    def pressure_vessel_design(x):
        """
        压力容器设计问题
        最小化材料成本
        约束：壁厚、容量等
        """
        # 解编码：[Ts, Th, R, L]
        Ts, Th, R, L = x
        
        # 目标：成本最小化
        cost = 0.6224*Ts*R*L + 1.7781*Th*R**2 + 3.1661*Ts**2*L + 19.84*Ts**2*R
        
        # 约束
        g1 = -Ts + 0.0193*R  # 壁厚约束
        g2 = -Th + 0.00954*R
        g3 = -np.pi*R**2*L - (4/3)*np.pi*R**3 + 1296000  # 体积约束
        g4 = L + 2*R - 240  # 长度约束
        
        # 罚函数
        penalty = 0
        for g in [g1, g2, g3, g4]:
            if g < 0:
                penalty += -g * 1000
        
        return cost + penalty
    
    @staticmethod
    def spring_design(x):
        """
        弹簧设计问题
        最小化重量
        """
        # 解编码：[d, D, N]
        d, D, N = x
        
        # 目标
        weight = (N + 2) * D * d**2
        
        # 约束
        C = D / d  # 弹簧指数
        
        g1 = 1 - C**3 * N / 7178  # 剪切应力
        g2 = C - (140 + 0.1) / (120 + 0.3)  # 弹簧指数
        g3 = (4*C - C**2) / (4*C - 3*C**2 + C**3 - 0.00185) - 1/750  # 偏转
        g4 = d - 0.2  # 直径下限
        
        penalty = 0
        for g in [g1, g2, g3, g4]:
            if g < 0:
                penalty += -g * 1000
        
        return weight + penalty
```

---

# 十六、代码模板与最佳实践

## 16.1 通用PSO模板

```python
"""
PSO通用优化模板
开箱即用，适用于大多数连续优化问题
"""

import numpy as np
import random
from typing import Callable, List, Tuple, Optional
from dataclasses import dataclass


@dataclass
class PSOConfig:
    """PSO配置参数"""
    n_particles: int = 30
    max_iter: int = 1000
    w: float = 0.729
    c1: float = 1.49445
    c2: float = 1.49445
    v_max_ratio: float = 0.2
    tol: float = 1e-8
    early_stop_patience: int = 50


class GenericPSO:
    """通用PSO优化器"""
    
    def __init__(self, objective: Callable,
                 bounds: List[Tuple[float, float]],
                 config: Optional[PSOConfig] = None):
        self.objective = objective
        self.bounds = bounds
        self.config = config or PSOConfig()
        
        self.dim = len(bounds)
        self._setup()
    
    def _setup(self):
        """初始化"""
        bounds_array = np.array(self.bounds)
        self.low_bounds = bounds_array[:, 0]
        self.high_bounds = bounds_array[:, 1]
        
        self.v_max = (self.high_bounds - self.low_bounds) * \
                     self.config.v_max_ratio
        
        # 初始化粒子
        self.positions = np.random.uniform(
            self.low_bounds, self.high_bounds,
            (self.config.n_particles, self.dim)
        )
        
        self.velocities = np.random.uniform(
            -self.v_max, self.v_max,
            (self.config.n_particles, self.dim)
        )
        
        self.fitness = np.array([
            self.objective(pos) for pos in self.positions
        ])
        
        self.pbest = self.positions.copy()
        self.pbest_fit = self.fitness.copy()
        
        best_idx = np.argmin(self.fitness)
        self.gbest = self.positions[best_idx].copy()
        self.gbest_fit = self.fitness[best_idx]
        
        self.history = [self.gbest_fit]
        self.no_improvement_count = 0
    
    def step(self):
        """执行一步迭代"""
        r1 = np.random.random((self.config.n_particles, self.dim))
        r2 = np.random.random((self.config.n_particles, self.dim))
        
        # 更新速度
        cognitive = self.config.c1 * r1 * (self.pbest - self.positions)
        social = self.config.c2 * r2 * (self.gbest - self.positions)
        
        self.velocities = self.config.w * self.velocities + cognitive + social
        
        # 限制速度
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
        
        # 更新位置
        self.positions += self.velocities
        self.positions = np.clip(self.positions, self.low_bounds, self.high_bounds)
        
        # 评估
        self.fitness = np.array([self.objective(pos) for pos in self.positions])
        
        # 更新pbest
        improved = self.fitness < self.pbest_fit
        self.pbest[improved] = self.positions[improved]
        self.pbest_fit[improved] = self.fitness[improved]
        
        # 更新gbest
        if self.fitness.min() < self.gbest_fit:
            self.gbest_fit = self.fitness.min()
            self.gbest = self.positions[np.argmin(self.fitness)].copy()
            self.no_improvement_count = 0
        else:
            self.no_improvement_count += 1
        
        self.history.append(self.gbest_fit)
    
    def run(self) -> Tuple[np.ndarray, float]:
        """运行优化"""
        for _ in range(self.config.max_iter):
            self.step()
            
            if self.gbest_fit < self.config.tol:
                break
            
            if self.no_improvement_count > self.config.early_stop_patience:
                break
        
        return self.gbest.copy(), self.gbest_fit


# 使用示例
if __name__ == "__main__":
    # 定义优化问题
    def sphere(x):
        return np.sum(x**2)
    
    # 设置边界
    bounds = [(-10, 10)] * 10
    
    # 创建优化器
    pso = GenericPSO(sphere, bounds)
    
    # 运行优化
    best_pos, best_fitness = pso.run()
    
    print(f"Best position: {best_pos}")
    print(f"Best fitness: {best_fitness}")
```

## 16.2 面向对象PSO框架

```python
"""
面向对象的PSO框架
支持扩展和定制
"""

from abc import ABC, abstractmethod
from typing import List, Tuple, Callable
import numpy as np


class TopologyStrategy(ABC):
    """拓扑策略接口"""
    
    @abstractmethod
    def get_leader(self, fitness: np.ndarray, 
                   pbest_positions: np.ndarray,
                   positions: np.ndarray) -> np.ndarray:
        pass


class GlobalBestTopology(TopologyStrategy):
    """全局最优拓扑"""
    
    def get_leader(self, fitness: np.ndarray, 
                   pbest_positions: np.ndarray,
                   positions: np.ndarray) -> np.ndarray:
        best_idx = np.argmin(fitness)
        return pbest_positions[best_idx]


class RingTopology(TopologyStrategy):
    """环形拓扑"""
    
    def __init__(self, k: int = 2):
        self.k = k
    
    def get_leader(self, fitness: np.ndarray,
                   pbest_positions: np.ndarray,
                   positions: np.ndarray) -> np.ndarray:
        n = len(fitness)
        leaders = np.zeros_like(positions)
        
        for i in range(n):
            neighbors = [(i + j) % n for j in range(-self.k, self.k + 1)]
            neighbor_fitness = fitness[neighbors]
            leaders[i] = pbest_positions[neighbors[np.argmin(neighbor_fitness)]]
        
        return leaders


class VelocityUpdateStrategy(ABC):
    """速度更新策略接口"""
    
    @abstractmethod
    def update(self, velocities: np.ndarray,
               positions: np.ndarray,
               pbest: np.ndarray,
               leader: np.ndarray,
               params: dict) -> np.ndarray:
        pass


class StandardVelocityUpdate(VelocityUpdateStrategy):
    """标准速度更新"""
    
    def update(self, velocities: np.ndarray,
               positions: np.ndarray,
               pbest: np.ndarray,
               leader: np.ndarray,
               params: dict) -> np.ndarray:
        w = params.get('w', 0.729)
        c1 = params.get('c1', 1.49445)
        c2 = params.get('c2', 1.49445)
        
        r1 = np.random.random(positions.shape)
        r2 = np.random.random(positions.shape)
        
        return (w * velocities +
                c1 * r1 * (pbest - positions) +
                c2 * r2 * (leader - positions))


class CompletePSOFramework:
    """
    完整的PSO框架
    可配置的组件化设计
    """
    
    def __init__(self,
                 objective: Callable,
                 bounds: List[Tuple[float, float]],
                 topology: TopologyStrategy = None,
                 velocity_update: VelocityUpdateStrategy = None):
        self.objective = objective
        self.bounds = bounds
        self.dim = len(bounds)
        
        self.topology = topology or GlobalBestTopology()
        self.velocity_update = velocity_update or StandardVelocityUpdate()
        
        self.bounds_array = np.array(bounds)
        self.low_bounds = self.bounds_array[:, 0]
        self.high_bounds = self.bounds_array[:, 1]
        
        self.params = {'w': 0.729, 'c1': 1.49445, 'c2': 1.49445}
    
    def initialize(self, n_particles: int = 30):
        """初始化种群"""
        self.n_particles = n_particles
        
        self.positions = np.random.uniform(
            self.low_bounds, self.high_bounds,
            (n_particles, self.dim)
        )
        
        self.v_max = (self.high_bounds - self.low_bounds) * 0.2
        self.velocities = np.random.uniform(
            -self.v_max, self.v_max,
            (n_particles, self.dim)
        )
        
        self.fitness = np.array([self.objective(p) for p in self.positions])
        self.pbest = self.positions.copy()
        self.pbest_fit = self.fitness.copy()
    
    def step(self):
        """执行一步"""
        # 获取领导者
        leader = self.topology.get_leader(
            self.pbest_fit, self.pbest, self.positions
        )
        
        # 更新速度
        self.velocities = self.velocity_update.update(
            self.velocities, self.positions, self.pbest, leader, self.params
        )
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
        
        # 更新位置
        self.positions += self.velocities
        self.positions = np.clip(self.positions, self.low_bounds, self.high_bounds)
        
        # 评估
        self.fitness = np.array([self.objective(p) for p in self.positions])
        
        # 更新pbest
        improved = self.fitness < self.pbest_fit
        self.pbest[improved] = self.positions[improved]
        self.pbest_fit[improved] = self.fitness[improved]
    
    def run(self, max_iter: int = 1000):
        """运行优化"""
        for _ in range(max_iter):
            self.step()
        
        best_idx = np.argmin(self.pbest_fit)
        return self.pbest[best_idx], self.pbest_fit[best_idx]
```

---

# 十七、常见问题与解决方案

## 17.1 收敛问题

| 问题 | 症状 | 解决方案 |
|------|------|----------|
| 早熟收敛 | 适应度早期停滞 | 增加惯性权重，使用lbest拓扑 |
| 收敛过慢 | 大量迭代后仍不收敛 | 减小惯性权重，增加学习因子 |
| 振荡 | 粒子速度过大 | 减小速度限制，增大c1, c2 |
| 发散 | 适应度越来越大 | 检查边界处理，增加阻尼 |

## 17.2 参数调优指南

### 17.2.1 问题导向的参数选择

```python
class ProblemBasedParameterSelector:
    """基于问题特征选择参数"""
    
    @staticmethod
    def select_for_unimodal(objective_gradients=None):
        """单峰问题"""
        return {
            'w': 0.729,
            'c1': 1.49445,
            'c2': 1.49445,
            'topology': 'gbest'
        }
    
    @staticmethod
    def select_for_multimodal():
        """多峰问题"""
        return {
            'w': 0.5,  # 较低惯性以增加探索
            'c1': 1.5,
            'c2': 1.5,
            'topology': 'ring'
        }
    
    @staticmethod
    def select_for_high_dim(dim):
        """高维问题"""
        return {
            'w': 0.6,
            'c1': 1.5,
            'c2': 1.5,
            'n_particles': min(100, 10 * dim),
            'topology': 'von_neumann'
        }
    
    @staticmethod
    def select_for_noisy():
        """噪声环境"""
        return {
            'w': 0.4,  # 低惯性
            'c1': 2.0,  # 强调认知
            'c2': 0.5,
            'n_particles': 100,
            'topology': 'ring'
        }
```

## 17.3 调试清单

```python
class PSODebugger:
    """PSO调试工具"""
    
    def __init__(self, optimizer):
        self.optimizer = optimizer
        self.debug_info = {
            'velocity_magnitude': [],
            'position_spread': [],
            'fitness_variance': [],
            'diversity': []
        }
    
    def collect_debug_info(self):
        """收集调试信息"""
        # 速度幅度
        vel_mag = np.mean(np.linalg.norm(self.optimizer.velocities, axis=1))
        self.debug_info['velocity_magnitude'].append(vel_mag)
        
        # 位置分散度
        spread = np.mean(np.std(self.optimizer.positions, axis=0))
        self.debug_info['position_spread'].append(spread)
        
        # 适应度方差
        fit_var = np.var(self.optimizer.fitness)
        self.debug_info['fitness_variance'].append(fit_var)
        
        # 多样性
        centroid = np.mean(self.optimizer.positions, axis=0)
        diversity = np.mean(np.linalg.norm(
            self.optimizer.positions - centroid, axis=1
        ))
        self.debug_info['diversity'].append(diversity)
    
    def diagnose_issues(self):
        """诊断问题"""
        issues = []
        
        vm = self.debug_info['velocity_magnitude']
        if vm and vm[-1] < 0.01:
            issues.append("Velocity too low - consider increasing w")
        if vm and vm[-1] > 10:
            issues.append("Velocity too high - consider decreasing w or v_max")
        
        div = self.debug_info['diversity']
        if div and div[-1] < 0.1:
            issues.append("Low diversity - risk of premature convergence")
        
        return issues
    
    def generate_report(self):
        """生成诊断报告"""
        return {
            'final_velocity_magnitude': self.debug_info['velocity_magnitude'][-1],
            'final_position_spread': self.debug_info['position_spread'][-1],
            'final_fitness_variance': self.debug_info['fitness_variance'][-1],
            'final_diversity': self.debug_info['diversity'][-1],
            'issues': self.diagnose_issues()
        }
```

---

# 十八、参考文献

1. Kennedy, J., & Eberhart, R. (1995). Particle Swarm Optimization. *Proceedings of IEEE International Conference on Neural Networks*, 4, 1942-1948.
2. Shi, Y., & Eberhart, R. C. (1998). A Modified Particle Swarm Optimizer. *Proceedings of IEEE International Conference on Evolutionary Computation*, 69-73.
3. Clerc, M., & Kennedy, J. (2002). The Particle Swarm - Explosion, Stability, and Convergence in a Multidimensional Complex Space. *IEEE Transactions on Evolutionary Computation*, 6(1), 58-73.
4. Kennedy, J., & Eberhart, R. C. (1997). A Discrete Binary Version of the Particle Swarm Algorithm. *Proceedings of IEEE International Conference on Systems, Man, and Cybernetics*, 4104-4108.
5. Poli, R., Kennedy, J., & Blackwell, T. (2007). Particle Swarm Optimization: An Overview. *Swarm Intelligence*, 1(1), 33-57.
6. Eberhart, R. C., & Shi, Y. (2001). Particle Swarm Optimization: Developments, Applications and Resources. *Proceedings of the 2001 Congress on Evolutionary Computation*, 1, 81-86.
7. Van den Bergh, F., & Engelbrecht, A. P. (2004). A Cooperative Approach to Particle Swarm Optimization. *IEEE Transactions on Evolutionary Computation*, 8(3), 225-239.
8. Mendes, R., Kennedy, J., & Neves, J. (2004). The Fully Informed Particle Swarm: Simpler, Maybe Better. *IEEE Transactions on Evolutionary Computation*, 8(3), 204-210.
9. Zhan, Z. H., et al. (2009). Adaptive Particle Swarm Optimization. *IEEE Transactions on Systems, Man, and Cybernetics*, 39(6), 1362-1381.
10. Liu, Q., et al. (2015). Order of Magnitude Improved Particle Swarm Optimizer. *Information Sciences*, 320, 73-94.

---

## 相关文档

- [[遗传算法深度指南]]
- [[差分进化算法]]
- [[进化策略深度指南]]
- [[神经进化详解]]
- [[遗传编程详解]]
