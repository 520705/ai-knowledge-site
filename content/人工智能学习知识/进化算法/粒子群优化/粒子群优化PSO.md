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

---

# 十、实用框架与工具

## 10.1 PyPSO库

```python
from pypso import PSOOptimizer

# 简单用法
optimizer = PSOOptimizer(
    objective=my_function,
    n_particles=30,
    dimensions=10,
    bounds=((-10, 10),) * 10,
    w=0.729,
    c1=1.49445,
    c2=1.49445
)

result = optimizer.optimize(max_iterations=1000)
```

## 10.2 pyswarms库

```python
import pyswarms as ps

# 全局最优PSO
optimizer = ps.single.GeneralOptimizerPSO(
    n_particles=30,
    dimensions=10,
    options={'w': 0.729, 'c1': 1.49445, 'c2': 1.49445},
    bounds=(np.zeros(10), np.ones(10) * 10)
)

best_cost, best_pos = optimizer.optimize(
    objective_func=my_function,
    iters=1000,
    verbose=True
)

# 离散二进制PSO
from pyswarms.discrete import BinaryPSO
optimizer = BinaryPSO(
    n_particles=30,
    dimensions=50,
    options={'c1': 2, 'c2': 2, 'w': 0.9}
)
```

## 10.3 参数推荐表

| 问题类型 | n_particles | w | c1 | c2 | v_max比例 |
|---------|------------|---|---|-----|----------|
| 通用连续 | 30-50 | 0.729 | 1.49445 | 1.49445 | 10-20% |
| 高维问题 | 50-100 | 0.6-0.8 | 1.5 | 1.5 | 5-10% |
| 多峰问题 | 40-60 | 0.5-0.7 | 1.0 | 2.0 | 15-25% |
| 离散问题 | 20-40 | 0.9 | 2.0 | 2.0 | - |

---

# 参考文献

1. Kennedy, J., & Eberhart, R. (1995). Particle Swarm Optimization. *Proceedings of IEEE International Conference on Neural Networks*, 4, 1942-1948.
2. Shi, Y., & Eberhart, R. C. (1998). A Modified Particle Swarm Optimizer. *Proceedings of IEEE International Conference on Evolutionary Computation*, 69-73.
3. Clerc, M., & Kennedy, J. (2002). The Particle Swarm - Explosion, Stability, and Convergence in a Multidimensional Complex Space. *IEEE Transactions on Evolutionary Computation*, 6(1), 58-73.
4. Kennedy, J., & Eberhart, R. C. (1997). A Discrete Binary Version of the Particle Swarm Algorithm. *Proceedings of IEEE International Conference on Systems, Man, and Cybernetics*, 4104-4108.
5. Poli, R., Kennedy, J., & Blackwell, T. (2007). Particle Swarm Optimization: An Overview. *Swarm Intelligence*, 1(1), 33-57.

---

## 相关文档

- [[遗传算法深度指南]]
- [[差分进化算法]]
- [[进化策略深度指南]]
- [[神经进化详解]]
- [[遗传编程详解]]
