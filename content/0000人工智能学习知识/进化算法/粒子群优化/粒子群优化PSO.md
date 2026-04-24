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

# PSO入门：从鸟群觅食说起

## 一个关于鸟群的故事

想象一下这个场景：周末你去公园，看到一群鸟在草地上觅食。它们看起来漫无目的地四处走动，但实际上这套"无组织无纪律"的觅食方式暗藏玄机。

鸟群中的每只鸟其实都在遵循三条简单的规则：

第一条，**自己找食吃**。每只鸟都会记住自己飞过的路径中哪里食物最多，下次可能还会去那片区域碰碰运气。

第二条，**跟着同伴走**。如果某只鸟发现附近有同伴找到了好吃的，它可能会凑过去看看。

第三条，**不忘探索新地方**。虽然跟着同伴走挺省事，但完全放弃自主探索的话，万一哪天那片区域食物没了，整个鸟群都得饿死。

这三条规则看起来平平无奇对吧？但正是这种"个体自主探索 + 群体信息共享"的机制，让鸟群能够快速找到食物丰富的区域。单个鸟的能力很有限，但群体合作起来就厉害了——这在生物学上叫"涌现"（emergence），就是个体简单行为的叠加产生了复杂的群体智慧。

1995年，Kennedy和Eberhart两位老哥从鸟群觅食这个现象中得到启发，提出了粒子群优化算法（PSO）。他们把每只鸟看成一个"粒子"（particle），把觅食区域看成一个需要优化的"解空间"，把食物最密集的位置看做"最优解"。粒子的飞行方向和速度会受三个因素影响：自己飞过的最好位置、邻居找到的最好位置、以及惯性（继续保持原来的飞行方向）。就这么简单的一个机制，在很多优化问题上效果出奇的好。

## 为什么PSO值得关注？

你在网上搜优化算法，遗传算法（GA）、模拟退火（SA）、粒子群优化（PSO）这些都是老面孔了。但PSO有几个特点让它特别适合某些场景：

第一个是**简单粗暴**。PSO的核心公式就一行，代码写起来比遗传算法短多了。遗传算法还要设计交叉、变异、选择三种操作，PSO只需要更新速度和位置两个公式。你要是想快速验证一个想法，PSO能让你少写不少代码。

第二个是**收敛快**。很多场景下PSO比遗传算法收敛得快，尤其在连续优化问题上。当然代价是可能更容易陷入局部最优，这个我们后面会详细说。

第三个是**参数少**。PSO需要调的参数不多，主要就是粒子数量、惯性权重、两个学习因子，新手也能玩得转。

当然PSO不是万能的。对于离散优化问题、组合优化问题，你需要用改进版的PSO。对于多目标优化，标准PSO也得加点东西才能用。这些我们都会讲到。

---

# PSO vs 遗传算法：本质区别是什么？

很多初学者学完GA（遗传算法）再学PSO，会有个困惑：这两个都是进化算法，看起来都是在"种群"里搜索，有什么本质区别？

## 信息传递的方式不同

这是最核心的区别。

遗传算法靠**"繁殖"**来传递信息。父代通过交叉生成子代，子代继承父代的部分基因，本质上是在"组合"已有的信息。如果一个好的解决方案包含A特征和B特征，GA通过交叉有概率把这两个特征组合到同一个个体上。

PSO靠**"通信"**来传递信息。粒子之间直接共享自己找到的好位置，粒子会朝着自己历史去过的最好位置和群体发现的最好位置飞去。信息是"流动"的，而不是"继承"的。

打个比方，GA像是**婚姻制度**——两个人结婚生孩子，孩子的基因是父母基因的混合。PSO像是**朋友圈**——你看到朋友去了某个好地方，你就直接打车过去跟他汇合，不需要等他生个孩子继承他的属性。

## 记忆机制不同

GA的种群是没有记忆的。每一代都是全新的个体，老一代死亡、新一代诞生，父辈的经验只能通过基因编码传递给子代。如果某一代突然出现了一个特别好的个体，但它没来得及繁殖就死了，那这个好信息就永远丢失了。

PSO的每个粒子都有记忆。每个粒子会记住自己飞过的最好位置（pbest），整个群体会记住找到的最好位置（gbest或lbest）。这些信息不会丢失，会一直引导后续的搜索。

## 探索与开发的平衡机制不同

GA的探索-开发平衡主要靠**变异算子**。交叉负责"开发"——在已有解的基础上小修小补；变异负责"探索"——跳到解空间的未知区域。参数设置直接影响这两者的比例。

PSO的探索-开发平衡靠**惯性权重**。惯性权重控制粒子保持原方向飞行的倾向：惯性大，粒子探索能力强但收敛慢；惯性小，粒子开发能力强但可能错过好区域。

## 该用哪个？

给你一个简单的决策参考：

如果你解决的是**连续函数优化**问题，维度不太高（几十维以内），优先试试PSO，速度快、实现简单。

如果你解决的是**组合优化**问题，比如调度、路由、装箱，GA及其变体更成熟。

如果你需要**多目标优化**，NSGA-II这种专门设计的多目标遗传算法更适合，PSO需要做不少改造。

如果你的问题**维度特别高**（几百上千维），试试协同PSO或者降维策略。

当然这些都是经验之谈，具体问题还得具体分析。很多时候两个算法都试试，比一比哪个效果好，这是最靠谱的做法。

---

# 速度-位置更新公式的直觉理解

## 先看公式

PSO的核心就是这两个公式：

速度更新：

$$
\mathbf{v}_i(t+1) = w \cdot \mathbf{v}_i(t) + c_1 \cdot r_1 \cdot (\mathbf{p}_i - \mathbf{x}_i(t)) + c_2 \cdot r_2 \cdot (\mathbf{g} - \mathbf{x}_i(t))
$$

位置更新：

$$
\mathbf{x}_i(t+1) = \mathbf{x}_i(t) + \mathbf{v}_i(t+1)
$$

刚看到公式可能有点懵，我来帮你拆解一下每个部分的含义。

## 三个组成部分

**第一项：惯性项** $w \cdot \mathbf{v}_i(t)$

这一项的意思是"继续按原来的方向飞"。$w$是惯性权重，通常取0到1之间的值。如果$w=0.9$，粒子会保留90%的原速度方向；如果$w=0.1$，粒子基本会改变方向。

惯性项的作用是**维持搜索的连续性**。想象一下，如果粒子每一步都完全改变方向，那它的飞行轨迹就会非常不稳定，像个无头苍蝇一样乱窜。惯性让粒子的移动更平滑，同时也能帮助跳出局部最优。

**第二项：认知项** $c_1 \cdot r_1 \cdot (\mathbf{p}_i - \mathbf{x}_i(t))$

这一项的意思是"向自己曾经找到的最好位置飞"。$\mathbf{p}_i$是第$i$个粒子历史上到过的最优位置，$\mathbf{p}_i - \mathbf{x}_i(t)$就是从当前位置指向历史最优的向量。

$c_1$是认知学习因子，控制"相信自己的经验"的程度。$r_1$是0到1之间的随机数，用来增加一点随机性，避免所有粒子都走同一条路。

**第三项：社会项** $c_2 \cdot r_2 \cdot (\mathbf{g} - \mathbf{x}_i(t))$

这一项的意思是"向群体找到的最好位置飞"。$\mathbf{g}$是整个粒子群找到的最优位置。

$c_2$是社会学习因子，控制"相信群体的智慧"的程度。$r_2$同样是随机数。

## 一个形象的比喻

把这三个项想象成你做人生决策时考虑的三种声音：

惯性项代表"惯性"——你习惯的生活方式、按部就班的发展路径。想改变惯性需要额外的努力。

认知项代表"自我反思"——你过去踩过的坑、学到的经验。你会参考自己的历史来调整下一步。

社会项代表"社会学习"——朋友建议、行业趋势、专家意见。你会参考别人的成功经验来指导自己。

PSO的粒子就是同时听取这三种声音的"理性决策者"。三种声音的权重不同，最终的行为模式也不同。

## 随机数为什么存在？

细心的人会问：$\mathbf{p}_i - \mathbf{x}_i(t)$已经是一个确定的方向了，为什么还要乘以$r_1$和$r_2$？

这其实是PSO设计的一个巧妙之处。

如果没有随机项，所有粒子在同样的初始条件下会沿着完全相同的轨迹移动（因为公式是确定性的）。这叫"确定性崩溃"——所有粒子聚集到同一条路径上，失去了多样性。

加入随机项后，即使初始条件相同，不同粒子也会因为随机数的差异走出不同的轨迹。$r_1$让粒子"不一定完全听从自己的经验"，$r_2$让粒子"不一定完全跟随群体的方向"。这种有限度的"叛逆"保证了种群的多样性，避免早熟收敛。

---

# 从零实现PSO：Python完整代码

## 最简版本的PSO

我们先实现一个最简单、最容易理解的版本，不追求任何优化，只追求清晰表达PSO的核心思想：

```python
import numpy as np

def basic_pso(objective, bounds, n_particles=30, max_iter=100):
    """
    最基础的PSO实现
    
    参数:
        objective: 目标函数，接受一个numpy数组，返回一个标量
        bounds: 解空间边界，格式为 [(min, max), (min, max), ...]
        n_particles: 粒子数量
        max_iter: 最大迭代次数
    返回:
        best_position: 找到的最优解
        best_fitness: 最优解对应的目标函数值
    """
    dim = len(bounds)
    
    # 第一步：初始化所有粒子的位置和速度
    # 位置：在边界范围内随机初始化
    positions = np.array([
        np.random.uniform(b[0], b[1], n_particles) for b in bounds
    ]).T  # 转置后每行是一个粒子的位置
    
    # 速度：初始化为0
    velocities = np.zeros((n_particles, dim))
    
    # 第二步：评估初始适应度
    fitness = np.array([objective(pos) for pos in positions])
    
    # 第三步：记录每个粒子的历史最优位置
    personal_best_positions = positions.copy()
    personal_best_fitness = fitness.copy()
    
    # 第四步：找到群体的全局最优
    global_best_idx = np.argmin(fitness)
    global_best_position = positions[global_best_idx].copy()
    global_best_fitness = fitness[global_best_idx]
    
    # 开始迭代
    for iteration in range(max_iter):
        # 参数设置
        w = 0.7   # 惯性权重
        c1 = 1.5  # 认知系数（相信自己）
        c2 = 1.5  # 社会系数（相信群体）
        
        for i in range(n_particles):
            # 生成随机数
            r1 = np.random.random(dim)
            r2 = np.random.random(dim)
            
            # 更新速度
            velocities[i] = (
                w * velocities[i] +  # 惯性项
                c1 * r1 * (personal_best_positions[i] - positions[i]) +  # 认知项
                c2 * r2 * (global_best_position - positions[i])  # 社会项
            )
            
            # 更新位置
            positions[i] = positions[i] + velocities[i]
            
            # 边界处理：把超出边界的粒子拉回来
            for d in range(dim):
                if positions[i, d] < bounds[d][0]:
                    positions[i, d] = bounds[d][0]
                if positions[i, d] > bounds[d][1]:
                    positions[i, d] = bounds[d][1]
            
            # 评估当前适应度
            current_fitness = objective(positions[i])
            
            # 更新个体最优
            if current_fitness < personal_best_fitness[i]:
                personal_best_positions[i] = positions[i].copy()
                personal_best_fitness[i] = current_fitness
                
                # 如果这是新的全局最优
                if current_fitness < global_best_fitness:
                    global_best_position = positions[i].copy()
                    global_best_fitness = current_fitness
        
        # 每50轮输出一次进度
        if iteration % 50 == 0:
            print(f"迭代 {iteration}: 当前最优 = {global_best_fitness:.8f}")
    
    return global_best_position, global_best_fitness
```

这个版本大概60行代码，已经能跑了。核心就是两层循环：外层迭代，内层遍历每个粒子更新速度和位置。

## 测试一下

用几个经典的测试函数来验证代码是否正确：

```python
# 测试1：Sphere函数 f(x) = sum(x^2)
# 全局最优在原点，最优值 = 0
def sphere(x):
    return np.sum(x**2)

bounds = [(-10, 10)] * 10
best_pos, best_fit = basic_pso(sphere, bounds, n_particles=50, max_iter=200)
print(f"\nSphere函数测试:")
print(f"找到的最优位置: {best_pos}")
print(f"最优值: {best_fit}")

# 测试2：Rosenbrock函数（香蕉函数）
# 全局最优在 (1, 1, ..., 1)，最优值 = 0
def rosenbrock(x):
    return sum(100*(x[i+1] - x[i]**2)**2 + (1 - x[i])**2 for i in range(len(x)-1))

bounds = [(-5, 10)] * 10
best_pos, best_fit = basic_pso(rosenbrock, bounds, n_particles=50, max_iter=200)
print(f"\nRosenbrock函数测试:")
print(f"最优值: {best_fit}")
```

如果代码正确，Sphere函数应该能收敛到接近0的位置，Rosenbrock函数应该能收敛到接近1的位置。

## 优化版本的PSO

基础版本能工作，但在工程实践中，我们需要考虑更多细节。下面是一个功能更完整的版本，加入了速度限制、早停机制、收敛判断等功能：

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
        
        # 初始化速度限制（通常为搜索范围的10%-20%）
        self.v_max = np.array([(high - low) * 0.2 
                              for low, high in bounds])
        
        # 历史记录（用于可视化）
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
        
        # 速度限制（防止爆炸）
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
            
            # 早停条件：达到容忍度
            if self.gbest_fitness < self.tol:
                print(f"Converged at iteration {iteration}")
                break
        
        return self.gbest_position, self.gbest_fitness
```

---

# PSO参数设置：经验之谈

## 惯性权重w

惯性权重$w$是PSO最重要的参数，它控制了粒子的"探索-开发"平衡。

**$w$ 大（接近1）**：粒子倾向于保持原有方向飞行，探索能力强，但收敛慢。适合多峰函数、需要全局搜索的问题。

**$w$ 小（接近0）**：粒子更容易被吸引到好的位置，开发能力强，收敛快，但可能错过全局最优。适合单峰函数、问题结构已知的情况。

常用的$w$设置策略：

第一种是**固定值**。$w=0.729$是一个理论推导出来的稳定值，很多情况下效果不错。

第二种是**线性递减**。开始时$w$较大（0.9）鼓励探索，随着迭代进行逐渐减小（到0.4）加速收敛：

```python
w = w_max - (w_max - w_min) * iteration / max_iter
```

第三种是**自适应**。根据种群多样性动态调整。如果粒子太集中了，增加$w$；如果粒子太分散了，减小$w$。

## 认知系数c1和社会系数c2

$c_1$控制粒子"相信自己经验"的倾向，$c_2$控制粒子"相信群体智慧"的倾向。

常见的配置：

**$c_1 = c_2 = 1.49445$**：Clerc推导出保证收敛的理论最优值，最常用。

**$c_1 = c_2 = 2.0$**：经典配置，各占50%，平衡型。

**$c_1 > c_2$**：强调个体经验，探索能力强。

**$c_1 < c_2$**：强调群体智慧，开发能力强。

## 粒子数量

粒子数量通常和问题维度、问题复杂度有关。

对于**低维问题**（10维以下），20-30个粒子就够了。

对于**中高维问题**（10-50维），30-50个粒子比较合适。

对于**高维问题**（50维以上），可能需要50-100个粒子。

有个经验公式：$n_{particles} = 10 \times \dim$，但最多不超过100。

## 速度限制v_max

$v_{max}$通常设为搜索范围的10%-20%。如果$v_{max}$太大，粒子会"飞过"最优区域；如果太小，粒子移动太慢，收敛时间太长。

## 参数组合推荐

针对不同类型的问题，我整理了一个参数推荐表：

| 问题类型 | w | c1 | c2 | 拓扑 | 粒子数 |
|---------|----|----|----|------|-------|
| 单峰函数 | 0.729 | 1.494 | 1.494 | gbest | 30 |
| 多峰函数 | 0.5-0.7 | 1.5 | 1.5 | ring | 50 |
| 高维问题 | 0.6 | 1.5 | 1.5 | von_neumann | 10×dim |
| 噪声环境 | 0.4 | 2.0 | 0.5 | ring | 100 |

---

# PSO变体：带收缩因子的PSO与量子PSO

## 带收缩因子的PSO

标准PSO有个问题：当参数设置不当时，粒子的速度可能会越来越大，最终"爆炸"飞遍整个搜索空间也找不到好解。

Clerc和Kennedy在2002年提出了收缩因子（constriction factor）的方法来解决这个问题。核心思想是把速度更新公式改写成：

$$
\mathbf{v}_i(t+1) = \chi \left[ \mathbf{v}_i(t) + c_1 r_1 (\mathbf{p}_i - \mathbf{x}_i) + c_2 r_2 (\mathbf{g} - \mathbf{x}_i) \right]
$$

其中收缩因子$\chi$的计算公式是：

$$
\chi = \frac{2}{|2 - \phi - \sqrt{\phi^2 - 4\phi}|}, \quad \phi = c_1 + c_2 > 4
$$

常用的配置是$c_1 = c_2 = 2.05$，此时$\phi = 4.1$，$\chi \approx 0.729$。

代码实现：

```python
class ConstrictionPSO:
    """带收缩因子的PSO"""
    
    def __init__(self, objective, dim, bounds, n_particles=30, max_iter=1000):
        self.objective = objective
        self.dim = dim
        self.bounds = bounds
        self.n_particles = n_particles
        self.max_iter = max_iter
        
        # 收缩因子参数
        self.phi = 4.1  # c1 + c2
        self.chi = 2 / abs(2 - self.phi - (self.phi**2 - 4*self.phi)**0.5)
        
        # 学习因子
        self.c1 = 2.05
        self.c2 = 2.05
    
    def step(self):
        """一步迭代"""
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        # 速度更新（带收缩因子）
        self.velocities = self.chi * (
            self.velocities +
            self.c1 * r1 * (self.pbest - self.positions) +
            self.c2 * r2 * (self.gbest - self.positions)
        )
        
        # 位置更新
        self.positions += self.velocities
        # ... 边界处理和评估 ...
```

收缩因子的好处是不需要显式的速度限制，理论上保证收敛，更稳定。

## 量子PSO

标准PSO的粒子有确定的轨迹——给定当前位置、速度和历史最优，下一步的位置是确定的。这既是优点（可预测）也是缺点（可能陷入局部最优）。

量子PSO（QPSO）引入了量子力学的思想：粒子的位置不再是确定的，而是在某个概率分布中存在。代表性的模型是Sun等人提出的"Delta势阱"模型。

核心思想：假设每个粒子在一个以自身历史最优pbest和全局最优gbest为焦点之一的势阱中运动，粒子的位置更新为：

```python
class QuantumPSO:
    """量子粒子群优化"""
    
    def __init__(self, *args, beta=0.5, **kwargs):
        super().__init__(*args, **kwargs)
        self.beta = beta  # 扩张-收缩因子
    
    def update_quantum_position(self, i):
        """量子位置更新"""
        phi1 = np.random.random(self.dim)
        phi2 = np.random.random(self.dim)
        
        # 吸引中心点
        p = (phi1 * self.pbest_positions[i] + phi2 * self.gbest_position) / (phi1 + phi2)
        
        # 量子旋转
        u = np.random.random(self.dim)
        
        # Delta势阱中的位置采样
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

QPSO的优势是搜索范围更广，更容易跳出局部最优。但收敛速度通常比标准PSO慢。

---

# PSO的应用场景

## 函数优化

这是PSO最直接的应用。几乎所有连续函数的优化问题都可以用PSO试试。

常用测试函数：

```python
# Sphere函数 - 简单的单峰函数
def sphere(x):
    return np.sum(x**2)

# Rastrigin函数 - 多峰函数，有大量局部最优
def rastrigin(x):
    A = 10
    n = len(x)
    return A * n + np.sum(x**2 - A * np.cos(2 * np.pi * x))

# Ackley函数 - 多峰，广泛用于测试全局优化算法
def ackley(x):
    a = 20
    b = 0.2
    c = 2 * np.pi
    sum1 = np.sum(x**2)
    sum2 = np.sum(np.cos(c * x))
    term1 = -a * np.exp(-b * np.sqrt(sum1 / len(x)))
    term2 = -np.exp(sum2 / len(x))
    return term1 + term2 + a + np.e

# Griewank函数 - 多峰，有大量局部最优
def griewank(x):
    part1 = np.sum(x**2) / 4000
    part2 = np.prod(np.cos(x / np.sqrt(np.arange(1, len(x)+1))))
    return part1 - part2 + 1
```

## 神经网络训练

PSO可以用来训练神经网络的权重，比梯度下降更不容易陷入局部最优，但计算量通常更大。

```python
class PSOTrainedNN:
    def __init__(self, layer_sizes):
        """
        layer_sizes: 各层神经元数量，如 [784, 128, 64, 10]
        """
        self.layer_sizes = layer_sizes
        self.dim = sum(
            layer_sizes[i] * layer_sizes[i+1] + layer_sizes[i+1]
            for i in range(len(layer_sizes)-1)
        )
    
    def encode_weights(self, flat_weights):
        """将扁平化的权重向量解码为网络权重矩阵列表"""
        weights = []
        start = 0
        for i in range(len(self.layer_sizes)-1):
            rows = self.layer_sizes[i+1]
            cols = self.layer_sizes[i]
            # 权重矩阵
            w_size = rows * cols
            W = flat_weights[start:start+w_size].reshape(rows, cols)
            start += w_size
            # 偏置向量
            b = flat_weights[start:start+rows]
            start += rows
            weights.append((W, b))
        return weights
    
    def fitness(self, flat_weights, X, y):
        """计算适应度（这里是分类误差，越小越好）"""
        weights = self.encode_weights(flat_weights)
        # 前向传播
        x = X
        for W, b in weights:
            x = x @ W.T + b
            x = np.maximum(0, x)  # ReLU激活
        predictions = np.argmax(x, axis=1)
        accuracy = np.mean(predictions == y)
        return 1 - accuracy  # 转换为最小化问题
    
    def train(self, X_train, y_train, n_particles=50, max_iter=200):
        """用PSO训练神经网络"""
        optimizer = ParticleSwarmOptimizer(
            objective=lambda w: self.fitness(w, X_train, y_train),
            dim=self.dim,
            bounds=[(-1, 1)] * self.dim,
            n_particles=n_particles,
            max_iter=max_iter
        )
        best_weights, best_fitness = optimizer.run()
        return best_weights, best_fitness
```

## 路径规划

在机器人导航、游戏AI、物流配送等领域，PSO常被用于路径优化。

```python
def path_planning_pso(start, goal, obstacles, n_waypoints=10, n_particles=50):
    """
    简单路径规划示例
    
    参数:
        start: 起点坐标
        goal: 终点坐标
        obstacles: 障碍物列表，每个障碍物是 (中心, 半径) 的元组
        n_waypoints: 路径中间点数量
        n_particles: 粒子数量
    """
    dim = 2 * n_waypoints  # x和y坐标
    
    def path_cost(path):
        """计算路径代价：距离 + 障碍物惩罚"""
        total_distance = 0
        prev_point = start
        
        for point in path:
            total_distance += np.linalg.norm(point - prev_point)
            prev_point = point
        total_distance += np.linalg.norm(goal - prev_point)
        
        # 障碍物惩罚
        penalty = 0
        for point in path:
            for obs_center, obs_radius in obstacles:
                dist = np.linalg.norm(point - obs_center)
                if dist < obs_radius:
                    penalty += (obs_radius - dist) * 100
        
        return total_distance + penalty
    
    def decode(particle):
        """将粒子解码为路径"""
        path = []
        for i in range(n_waypoints):
            t = (i + 1) / (n_waypoints + 1)
            # 在起点和终点之间线性插值
            base = start + t * (goal - start)
            # 加上粒子的偏移
            point = base + particle[2*i:2*i+2] * 10  # 10是搜索范围
            path.append(point)
        return np.array(path)
    
    bounds = [(-5, 5)] * dim
    optimizer = ParticleSwarmOptimizer(
        objective=lambda p: path_cost(decode(p)),
        dim=dim,
        bounds=bounds,
        n_particles=n_particles
    )
    best_particle, _ = optimizer.run()
    return decode(best_particle)
```

## 超参数优化

深度学习模型的超参数调优是个大麻烦。PSO可以作为贝叶斯优化的替代方案，尤其在超参数空间维度不高但评估代价大的场景。

```python
class PSOHyperparameterTuner:
    def __init__(self, train_data, val_data, model_fn):
        """
        train_data: 训练数据
        val_data: 验证数据
        model_fn: 接受超参数字典，返回训练好的模型
        """
        self.train_data = train_data
        self.val_data = val_data
        self.model_fn = model_fn
    
    def define_search_space(self):
        """定义超参数搜索空间"""
        return {
            'learning_rate': (1e-4, 1e-2),      # 连续
            'batch_size': [16, 32, 64, 128],    # 离散
            'hidden_units': (32, 256),
            'dropout_rate': (0.0, 0.5),
            'num_layers': [2, 3, 4],
        }
    
    def fitness(self, params):
        """评估超参数配置"""
        model = self.model_fn(params)
        model.fit(self.train_data[0], self.train_data[1], epochs=10, verbose=0)
        _, val_loss = model.evaluate(self.val_data[0], self.val_data[1])
        return val_loss
    
    def tune(self, n_particles=30, max_iter=50):
        """运行超参数优化"""
        space = self.define_search_space()
        # 编码为连续空间 [0, 1]
        dim = len(space)
        
        def encoded_fitness(encoded):
            # 解码
            params = {}
            idx = 0
            for name, spec in space.items():
                if isinstance(spec, tuple):  # 连续
                    params[name] = spec[0] + encoded[idx] * (spec[1] - spec[0])
                else:  # 离散
                    params[name] = spec[int(encoded[idx] * len(spec))]
                idx += 1
            return self.fitness(params)
        
        bounds = [(0, 1)] * dim
        optimizer = ParticleSwarmOptimizer(
            objective=encoded_fitness,
            dim=dim,
            bounds=bounds,
            n_particles=n_particles,
            max_iter=max_iter
        )
        best_encoded, _ = optimizer.run()
        
        # 解码最优参数
        params = {}
        idx = 0
        for name, spec in space.items():
            if isinstance(spec, tuple):
                params[name] = spec[0] + best_encoded[idx] * (spec[1] - spec[0])
            else:
                params[name] = spec[int(best_encoded[idx] * len(spec))]
            idx += 1
        return params
```

---

# 约束优化中的PSO

实际工程问题很少是没有约束的。压力容器设计、弹簧参数优化、工程结构设计……这些都有各种约束条件。

## 罚函数法

最简单的方法是把约束违反程度加到目标函数上：

```python
class ConstrainedPSO(ParticleSwarmOptimizer):
    def __init__(self, objective, constraints, *args, penalty_coef=1000, **kwargs):
        super().__init__(objective, *args, **kwargs)
        self.constraints = constraints
        self.penalty_coef = penalty_coef
    
    def evaluate_constraints(self, x):
        """评估约束违反程度"""
        violations = {'inequality': [], 'equality': []}
        
        # 不等式约束 g(x) >= 0
        for g in self.constraints.get('ineq', []):
            value = g(x)
            violations['inequality'].append(max(0, -value))
        
        # 等式约束 h(x) = 0
        for h in self.constraints.get('eq', []):
            value = abs(h(x))
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
    
    def evaluate(self):
        """重写的评估方法"""
        for i in range(self.n_particles):
            fitness = self.constrained_objective(self.positions[i])
            self.fitness[i] = fitness
            
            if fitness < self.pbest_fitness[i]:
                self.pbest_positions[i] = self.positions[i].copy()
                self.pbest_fitness[i] = fitness
                
                if fitness < self.gbest_fitness:
                    self.gbest_position = self.positions[i].copy()
                    self.gbest_fitness = fitness
```

## 可行性规则

罚函数法有个问题：惩罚系数不好确定。太大了可能导致搜索只关注可行性而忽略最优性，太小了又可能找到不可行的解。

一种改进方法是"可行性规则"：优先选择可行解，在可行解之间比较目标函数值。

```python
def is_feasible(x, constraints, tolerance=1e-6):
    """检查解是否可行"""
    for g in constraints.get('ineq', []):
        if g(x) < -tolerance:
            return False
    for h in constraints.get('eq', []):
        if abs(h(x)) > tolerance:
            return False
    return True

def constraint_violation(x, constraints):
    """计算总约束违反度"""
    total = 0
    for g in constraints.get('ineq', []):
        total += max(0, -g(x))
    for h in constraints.get('eq', []):
        total += abs(h(x))
    return total
```

---

# 离散PSO：用于组合优化问题

PSO天生是为连续优化设计的，但实际问题中有很多离散、组合优化的问题：旅行商问题（TSP）、调度问题、特征选择问题……

## 二进制PSO

Kennedy和Eberhart在1997年提出了二进制PSO，用于离散/组合优化问题。核心思想是把位置解释为0/1的概率：

```python
class BinaryPSO:
    """二进制PSO：适用于离散优化"""
    
    def __init__(self, objective, dim, n_particles=30, max_iter=100):
        self.objective = objective
        self.dim = dim
        self.n_particles = n_particles
        self.max_iter = max_iter
    
    def sigmoid(self, v):
        """Sigmoid函数：将速度映射到[0,1]"""
        return 1 / (1 + np.exp(-np.clip(v, -500, 500)))
    
    def initialize(self):
        self.positions = np.random.randint(0, 2, (self.n_particles, self.dim))
        self.velocities = np.random.uniform(-4, 4, (self.n_particles, self.dim))
        self.fitness = np.array([self.objective(p) for p in self.positions])
        self.pbest = self.positions.copy()
        self.pbest_fit = self.fitness.copy()
        
        gbest_idx = np.argmin(self.fitness)
        self.gbest = self.positions[gbest_idx].copy()
        self.gbest_fit = self.fitness[gbest_idx]
    
    def update_velocities(self, c1=2, c2=2):
        """速度更新（与连续版相同）"""
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        self.velocities += (
            c1 * r1 * (self.pbest - self.positions) +
            c2 * r2 * (self.gbest - self.positions)
        )
        self.velocities = np.clip(self.velocities, -4, 4)
    
    def update_positions(self):
        """位置更新：用sigmoid将速度转为概率"""
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

## 离散PSO示例：0-1背包问题

```python
def knapsack_pso(values, weights, capacity, n_particles=30, max_iter=200):
    """
    用PSO解决0-1背包问题
    
    参数:
        values: 物品价值列表
        weights: 物品重量列表
        capacity: 背包容量
    """
    n_items = len(values)
    
    def objective(x):
        """目标函数：最大化总价值，同时惩罚超重"""
        total_value = np.sum(x * values)
        total_weight = np.sum(x * weights)
        if total_weight > capacity:
            return -(total_value - (total_weight - capacity) * 100)  # 惩罚
        return -total_value  # 最小化问题，所以取负
    
    bounds = [(0, 1)] * n_items
    optimizer = BinaryPSO(objective, n_items, n_particles, max_iter)
    optimizer.initialize()
    
    for _ in range(max_iter):
        optimizer.step()
    
    best_solution = optimizer.gbest
    best_value = np.sum(best_solution * values)
    best_weight = np.sum(best_solution * weights)
    
    return best_solution, best_value, best_weight

# 测试
values = [60, 100, 120, 80, 50, 70, 90, 110]
weights = [10, 20, 30, 15, 10, 25, 35, 18]
capacity = 100

solution, value, weight = knapsack_pso(values, weights, capacity)
print(f"选择的物品: {np.where(solution == 1)[0]}")
print(f"总价值: {value}, 总重量: {weight}")
```

---

# PSO调参实战：如何找到最优参数组合？

## 手动调参的经验法则

如果你不想搞得太复杂，按照以下经验法则来设置参数，通常能获得不错的效果：

**第一步：先用默认参数跑一遍**

设置$w=0.729, c_1=1.49445, c_2=1.49445, n_{particles}=30$，看看效果如何。如果收敛太快（早熟），提高$w$；如果收敛太慢，降低$w$。

**第二步：根据问题类型调整**

单峰函数（只有一个最优解）：$w$可以小一些，0.5-0.7，加快收敛。

多峰函数（多个局部最优）：$w$要大一些，0.7-0.9，增强探索。

**第三步：平衡认知和社会**

$c_1 > c_2$（强调探索）：如果发现种群多样性下降，早熟风险高。

$c_1 < c_2$（强调开发）：如果收敛太慢，可以试试。

## 网格搜索找最优参数

如果你想系统性地找最优参数，可以用网格搜索：

```python
from itertools import product

def grid_search_pso(objective, dim, bounds, n_eval_per_config=10):
    """网格搜索最优PSO参数"""
    
    # 定义参数网格
    w_values = [0.4, 0.5, 0.6, 0.7, 0.8, 0.9]
    c1_values = [1.0, 1.5, 2.0]
    c2_values = [1.0, 1.5, 2.0]
    
    best_fitness = float('inf')
    best_params = None
    results = []
    
    for w, c1, c2 in product(w_values, c1_values, c2_values):
        fitnesses = []
        
        for _ in range(n_eval_per_config):
            pso = ParticleSwarmOptimizer(
                objective, dim, bounds,
                w=w, c1=c1, c2=c2, n_particles=30
            )
            _, fitness = pso.run()
            fitnesses.append(fitness)
        
        mean_fitness = np.mean(fitnesses)
        results.append({
            'w': w, 'c1': c1, 'c2': c2,
            'mean_fitness': mean_fitness,
            'std_fitness': np.std(fitnesses)
        })
        
        if mean_fitness < best_fitness:
            best_fitness = mean_fitness
            best_params = {'w': w, 'c1': c1, 'c2': c2}
    
    return best_params, results

# 使用示例
best_params, all_results = grid_search_pso(
    objective=sphere,
    dim=10,
    bounds=[(-10, 10)] * 10,
    n_eval_per_config=5
)
print(f"最优参数: {best_params}")
```

网格搜索计算量大（参数组合指数增长），但对于小规模参数空间很有效。

## 自适应参数调整

更高级的做法是让PSO自己学会调参：

```python
class AdaptivePSO(ParticleSwarmOptimizer):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.w = 0.9  # 初始高探索
        self.fitness_history = []
        self.diversity_history = []
    
    def calculate_diversity(self):
        """计算种群多样性"""
        centroid = np.mean(self.positions, axis=0)
        distances = np.linalg.norm(self.positions - centroid, axis=1)
        return np.mean(distances)
    
    def adjust_parameters(self, iteration):
        """根据当前状态自适应调整参数"""
        # 记录历史
        self.fitness_history.append(self.gbest_fitness)
        diversity = self.calculate_diversity()
        self.diversity_history.append(diversity)
        
        # 基于多样性的调整
        if len(self.diversity_history) > 10:
            recent_div = np.mean(self.diversity_history[-10:])
            initial_div = self.diversity_history[0]
            div_ratio = recent_div / (initial_div + 1e-10)
            
            if div_ratio < 0.2:  # 多样性太低
                self.w = min(0.95, self.w * 1.1)  # 增加探索
            elif div_ratio > 0.8:  # 多样性太高
                self.w = max(0.3, self.w * 0.95)  # 增加开发
        
        # 基于收敛的调整
        if len(self.fitness_history) > 20:
            improvement = self.fitness_history[-20] - self.fitness_history[-1]
            if improvement < 1e-6:  # 停滞
                self.w = min(0.95, self.w + 0.05)  # 增加探索
```

---

# 动手实验：用PSO求解多峰函数优化问题

理论讲了一大堆，是时候动手实验了。我们用PSO来求解几个经典的测试函数，观察不同参数下的表现。

## 实验一：Rastrigin函数

Rastrigin函数是个"坑爹"的多峰函数：有大量的局部最优，全局最优藏在密密麻麻的小坑里。公式是：

$$
f(x) = 10n + \sum_{i=1}^n \left[ x_i^2 - 10\cos(2\pi x_i) \right]
$$

全局最优在原点，值为0。

```python
import numpy as np
import matplotlib.pyplot as plt

def rastrigin(x):
    A = 10
    n = len(x)
    return A * n + np.sum(x**2 - A * np.cos(2 * np.pi * x))

# 实验设置
dim = 10
bounds = [(-5.12, 5.12)] * dim
n_runs = 10

# 测试不同惯性权重
w_values = [0.3, 0.5, 0.7, 0.9]
results = {}

for w in w_values:
    run_results = []
    for run in range(n_runs):
        pso = ParticleSwarmOptimizer(
            objective=rastrigin,
            dim=dim,
            bounds=bounds,
            n_particles=50,
            w=w, c1=1.49445, c2=1.49445,
            max_iter=500
        )
        _, fitness = pso.run()
        run_results.append(fitness)
    results[w] = {
        'mean': np.mean(run_results),
        'std': np.std(run_results),
        'min': np.min(run_results),
        'max': np.max(run_results)
    }

# 打印结果
print("Rastrigin函数 (10维) 测试结果:")
print("="*60)
for w, stats in results.items():
    print(f"w={w}: 均值={stats['mean']:.2f}, 标准差={stats['std']:.2f}, "
          f"最小={stats['min']:.2f}, 最大={stats['max']:.2f}")

# 可视化收敛曲线
plt.figure(figsize=(10, 6))
for w in w_values:
    pso = ParticleSwarmOptimizer(
        objective=rastrigin,
        dim=dim,
        bounds=bounds,
        n_particles=50,
        w=w, c1=1.49445, c2=1.49445,
        max_iter=500
    )
    pso.run()
    plt.plot(pso.gbest_history, label=f'w={w}')

plt.xlabel('迭代次数')
plt.ylabel('最优适应度')
plt.title('Rastrigin函数：不同惯性权重的收敛曲线')
plt.legend()
plt.grid(True)
plt.yscale('log')
plt.savefig('pso_rastrigin_convergence.png')
```

运行这个实验，你应该能看到：
- $w=0.3$：收敛很快但容易陷入局部最优
- $w=0.9$：探索能力强但收敛慢
- $w=0.5$或$w=0.7$：平衡型参数，通常表现最好

## 实验二：Ackley函数

Ackley函数是另一个经典的多峰函数，形似一个"碗"底加上一系列"小山"：

```python
def ackley(x):
    a = 20
    b = 0.2
    c = 2 * np.pi
    sum1 = np.sum(x**2)
    sum2 = np.sum(np.cos(c * x))
    term1 = -a * np.exp(-b * np.sqrt(sum1 / len(x)))
    term2 = -np.exp(sum2 / len(x))
    return term1 + term2 + a + np.e

# 高维测试
dims = [10, 30, 50]
results_ackley = {}

for dim in dims:
    bounds = [(-32.768, 32.768)] * dim
    pso = ParticleSwarmOptimizer(
        objective=ackley,
        dim=dim,
        bounds=bounds,
        n_particles=50,
        max_iter=1000
    )
    _, fitness = pso.run()
    results_ackley[dim] = fitness
    print(f"Ackley {dim}维: 最优值 = {fitness:.6f}")
```

## 实验三：可视化粒子轨迹

想直观理解PSO是怎么搜索的？我们可以可视化二维情况下粒子的轨迹：

```python
def visualize_2d_pso(objective, bounds, n_particles=30, max_iter=100):
    """可视化二维PSO搜索过程"""
    
    pso = ParticleSwarmOptimizer(
        objective=objective,
        dim=2,
        bounds=bounds,
        n_particles=n_particles,
        max_iter=max_iter
    )
    pso.initialize()
    
    # 记录轨迹
    trajectory = [pso.positions.copy()]
    
    for _ in range(max_iter):
        pso.update_velocities()
        pso.update_positions()
        pso.evaluate()
        trajectory.append(pso.positions.copy())
    
    # 绘制等高线
    fig, ax = plt.subplots(figsize=(10, 8))
    x = np.linspace(bounds[0][0], bounds[0][1], 100)
    y = np.linspace(bounds[1][0], bounds[1][1], 100)
    X, Y = np.meshgrid(x, y)
    Z = np.array([[objective([xi, yi]) for xi, yi in zip(xrow, yrow)] 
                  for xrow, yrow in zip(X, Y)])
    
    ax.contourf(X, Y, Z, levels=20, alpha=0.8)
    ax.colorbar()
    
    # 绘制初始位置
    ax.scatter(trajectory[0][:, 0], trajectory[0][:, 1], 
               c='blue', s=50, marker='o', label='初始位置')
    
    # 绘制最终位置
    ax.scatter(trajectory[-1][:, 0], trajectory[-1][:, 1], 
               c='red', s=50, marker='x', label='最终位置')
    
    # 绘制全局最优轨迹
    gbest_traj = [pso.gbest_position]
    ax.scatter(gbest_traj[-1][0], gbest_traj[-1][1], 
               c='green', s=200, marker='*', label='全局最优')
    
    ax.legend()
    ax.set_xlabel('x1')
    ax.set_ylabel('x2')
    ax.set_title('PSO搜索轨迹可视化')
    
    plt.savefig('pso_trajectory.png')
    plt.show()

# 运行可视化
visualize_2d_pso(rastrigin, [(-5.12, 5.12)] * 2)
```

---

# 全局版与局部版PSO

## gbest模型（全局版）

gbest模型是PSO的默认配置：每个粒子都能看到整个种群找到的最优位置。

**优点**：
- 收敛速度快
- 实现简单
- 对于单峰问题效果很好

**缺点**：
- 容易早熟收敛
- 对于复杂多峰问题可能陷入局部最优

```python
class GBestPSO(ParticleSwarmOptimizer):
    """全局最优模型PSO"""
    
    def update_velocities(self):
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
        social = self.c2 * r2 * (self.gbest_position - self.positions)
        
        self.velocities = self.w * self.velocities + cognitive + social
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
```

## lbest模型（局部版）

lbest模型采用局部拓扑结构：每个粒子只与邻居交换信息，不直接知道全局最优。

最常用的是环形拓扑：粒子$i$只和粒子$i-1$、$i+1$交换信息。

**优点**：
- 全局搜索能力强
- 不容易早熟收敛
- 对于多峰问题效果更好

**缺点**：
- 收敛速度较慢
- 需要更多迭代次数

```python
class LBestPSO(ParticleSwarmOptimizer):
    """局部最优模型PSO（环形拓扑）"""
    
    def __init__(self, *args, k=2, **kwargs):
        super().__init__(*args, **kwargs)
        self.k = k  # 每侧邻居数量
    
    def initialize_topology(self):
        """初始化环形拓扑"""
        self.neighbors = {}
        for i in range(self.n_particles):
            neighbors = [(i + j) % self.n_particles for j in range(-self.k, self.k+1)]
            self.neighbors[i] = neighbors
    
    def find_lbest(self):
        """找到每个粒子的局部最优"""
        lbest_positions = np.zeros((self.n_particles, self.dim))
        for i in range(self.n_particles):
            neighbor_indices = self.neighbors[i]
            neighbor_fitness = self.pbest_fitness[neighbor_indices]
            best_neighbor_idx = neighbor_indices[np.argmin(neighbor_fitness)]
            lbest_positions[i] = self.pbest_positions[best_neighbor_idx]
        return lbest_positions
    
    def update_velocities(self):
        self.initialize_topology()
        lbest = self.find_lbest()
        
        r1 = np.random.random((self.n_particles, self.dim))
        r2 = np.random.random((self.n_particles, self.dim))
        
        cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
        social = self.c2 * r2 * (lbest - self.positions)
        
        self.velocities = self.w * self.velocities + cognitive + social
        self.velocities = np.clip(self.velocities, -self.v_max, self.v_max)
```

## 拓扑结构对比

| 拓扑类型 | 收敛速度 | 全局搜索 | 稳定性 | 适用场景 |
|---------|---------|---------|--------|---------|
| gbest | 最快 | 最弱 | 较差 | 单峰问题 |
| Ring | 较慢 | 最强 | 较好 | 多峰问题 |
| Grid | 中等 | 中等 | 中等 | 通用 |
| Von Neumann | 中等 | 较强 | 较好 | 大规模问题 |

---

# 常见问题与解决方案

## 早熟收敛

**症状**：适应度在早期就停滞了，之后几乎不变。

**原因**：种群多样性丧失，所有粒子聚集到局部最优附近。

**解决方案**：
1. 增加惯性权重$w$
2. 改用lbest拓扑
3. 定期重新随机化部分粒子
4. 加入变异操作

## 收敛太慢

**症状**：迭代很多轮后仍然没有明显改进。

**原因**：$w$太小，或者$c_1, c_2$太小。

**解决方案**：
1. 减小惯性权重$w$
2. 增大学习因子$c_1, c_2$
3. 增大粒子数量
4. 使用更小的搜索空间（如果问题允许）

## 粒子"爆炸"

**症状**：粒子的位置和速度变得非常大，超出边界。

**原因**：速度没有限制，或者限制太大。

**解决方案**：
1. 减小速度限制$v_{max}$
2. 使用收缩因子
3. 检查边界处理代码

## 发散

**症状**：适应度值越来越大，完全不收敛。

**原因**：目标函数计算可能有bug，或者参数设置完全不对。

**解决方案**：
1. 检查目标函数实现
2. 尝试更保守的参数（$w$小一些）
3. 降低学习因子

---

# 总结

PSO是一个简洁但强大的优化算法。核心思想就是：每个粒子在解空间里飞，一边参考自己的经验，一边参考同伴的经验，最终找到最优解。

学会PSO，你就能：
- 优化连续函数
- 训练神经网络权重
- 解决组合优化问题
- 调参超参数优化
- 工程设计问题

关键是理解三个参数的作用：惯性权重$w$控制探索-开发平衡，认知系数$c_1$控制"相信自己"，社会系数$c_2$控制"相信群体"。

多峰问题用大$w$和lbest拓扑，单峰问题用小$w$和gbest拓扑，这是最基本的调参思路。

剩下的就是多实践、多实验。纸上得来终觉浅，绝知此事要躬行。

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
