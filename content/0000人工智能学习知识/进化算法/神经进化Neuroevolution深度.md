---
title: 神经进化Neuroevolution深度指南
date: 2026-04-24
tags:
  - 进化算法
  - 神经网络
  - 神经进化
  - NEAT
  - WANN
  - AutoML
aliases:
  - Neuroevolution
  - 神经进化算法
---

# 神经进化 Neuroevolution 深度指南

你有没有想过，如果不用梯度下降，神经网络还能怎么训练？答案就藏在神经进化里——用进化算法的思路来"养"神经网络，让它一代一代地"进化"，最终找到合适的结构和权重。这听起来有点反直觉，但神经进化在很多场景下确实能打出漂亮的翻身仗。

## 什么是神经进化

神经进化（Neuroevolution）简单来说就是用进化算法来训练神经网络。传统的方法比如反向传播，需要计算梯度，然后用梯度下降来更新权重。但神经进化不走这条路——它把整个神经网络的结构和权重当作染色体，用选择、交叉、变异这些进化算子来产生新一代的网络，然后一代一代地淘汰差的、保留好的。

为什么要搞这么麻烦？原因有几个：

第一，**梯度不可用的情况**。有些问题压根就没有梯度——比如强化学习中奖励信号是稀疏的，或者环境是动态变化的。这种时候梯度下降就抓瞎了，但神经进化照样能玩。

第二，**结构搜索**。如果你想同时优化网络的结构和权重，传统方法需要先固定结构再调权重，而神经进化可以一步到位，直接进化出最佳的网络拓扑。

第三，**避免局部最优**。梯度下降容易被局部最优卡住，但进化算法的搜索更加多样化，有更大的概率探索到全局最优解附近。

神经进化这个概念其实不新鲜，早在90年代就有人玩了。但随着深度学习火起来，它又被重新包装，以Neuroevolution of Augmenting Topologies（NEAT）为代表的技术在结构搜索、AutoML等领域焕发了第二春。

## NEAT：神经进化增强拓扑

NEAT是神经进化领域最著名的算法之一，由Kenneth Stanley在2002年提出。它的核心思想是从一个极简的网络开始，逐步进化出越来越复杂的结构。这个"从简单到复杂"的过程叫做**复杂化**（complexification），是NEAT区别于其他方法的关键。

### 历史上的创新

NEAT引入了几个重要的创新：

**1. 历史标记（Historical Markings）**

这是NEAT最聪明的设计之一。当通过变异添加一个新的节点或连接时，系统会给这个变化打上一个唯一的"历史标记"。这个标记本质上是一个全局递增的编号。比如第5次变异产生的连接编号是5，第12次变异产生的节点编号是12。

为什么要搞这个？因为在交叉操作中，不同的染色体可能用完全不同的方式来表示同一个网络结构。如果没有历史标记，交叉操作很可能会把不兼容的基因组合在一起，产生出荒谬的网络。有了历史标记，交叉时就很容易知道哪些基因是同源的，可以正确对齐。

**2. 创新编号（Innovation Numbers）**

每个新的拓扑变化（节点或连接）都会被分配一个全局唯一的创新编号。这样当两个染色体交叉时，系统可以通过比较创新编号来判断哪些基因是对应的。

**3. 物种形成（Speciation）**

NEAT用物种形成来保护创新。思路是这样的：如果每次都只保留最优秀的个体，那些有潜力但暂时不优秀的结构可能会被淘汰，导致搜索陷入局部最优。NEAT把相似的个体归为同一个"物种"，让它们在各自的种群内竞争，而不是在整个种群内竞争。这样新产生的结构变体就有机会存活下来，即使它们暂时表现不佳。

具体来说，两个个体是否属于同一物种，是通过**兼容性距离**来判定的：

$$distance = \frac{c_1 \cdot E}{N} + \frac{c_2 \cdot D}{N} + c_3 \cdot \bar{W}$$

其中 $E$ 是超额基因的数量（只在一方出现的基因），$D$ 是不匹配基因的数量（两边都有但权重不同），$\bar{W}$ 是权重差异的平均值，$N$ 是基因数量（用于归一化），$c_1, c_2, c_3$ 是权重系数。

### NEAT的算法流程

NEAT的基本流程是这样的：

1. **初始化**：创建一个极简的网络，只有一层输入和一层输出，之间没有隐藏层，所有连接权重为随机值。

2. **评估**：让每个个体（网络）在环境中运行，计算适应度。

3. **选择与繁殖**：根据适应度选择父代，通过交叉产生后代。

4. **变异**：以一定概率对网络进行结构变异（添加节点、添加连接）或参数变异（修改权重）。

5. **物种形成**：计算个体之间的兼容性距离，将个体分配到不同的物种。

6. **灭绝**：对每个物种内的个体，根据物种年龄和适应度决定是否灭绝某些物种，给年轻物种更多机会。

7. **精英保留**：每个物种的最优个体直接进入下一代。

8. **重复**：回到步骤2，直到达到终止条件。

这个流程看起来和标准遗传算法差不多，但关键是它同时进化结构和权重，而且结构变异是逐步增加的——这就是所谓的"通过增强拓扑进化"。

### NEAT的Python实现

下面是一个简化版的NEAT实现，帮助你理解核心逻辑：

```python
import random
import numpy as np
from typing import List, Tuple, Dict, Optional
from dataclasses import dataclass, field

# 全局创新编号计数器
innovation_counter = 0

@dataclass
class ConnectionGene:
    """连接基因"""
    in_node: int
    out_node: int
    weight: float
    innovation: int
    enabled: bool = True

@dataclass 
class NodeGene:
    """节点基因"""
    node_id: int
    node_type: str  # 'input', 'output', 'hidden'
    activation: str = 'sigmoid'
    aggregation: str = 'sum'

@dataclass
class Genome:
    """基因组"""
    nodes: Dict[int, NodeGene] = field(default_factory=dict)
    connections: Dict[int, ConnectionGene] = field(default_factory=dict)
    fitness: float = 0.0
    
    def create_node(self, node_type: str) -> int:
        """创建新节点"""
        global innovation_counter
        node_id = len(self.nodes)
        self.nodes[node_id] = NodeGene(node_id, node_type)
        return node_id
    
    def create_connection(self, in_node: int, out_node: int, weight: float = None) -> Optional[ConnectionGene]:
        """创建新连接，避免重复"""
        # 检查是否已存在
        for conn in self.connections.values():
            if conn.in_node == in_node and conn.out_node == out_node:
                return None
        
        global innovation_counter
        innovation_counter += 1
        
        weight = weight if weight is not None else random.uniform(-1, 1)
        conn = ConnectionGene(in_node, out_node, weight, innovation_counter)
        self.connections[innovation_counter] = conn
        return conn
    
    def mutate_add_node(self):
        """变异：添加节点"""
        if not self.connections:
            return
        
        # 随机选择一个连接来分割
        conn = random.choice(list(self.connections.values()))
        if not conn.enabled:
            return
        
        # 创建新节点
        new_node_id = self.create_node('hidden')
        
        # 禁用原连接
        conn.enabled = False
        
        # 添加两条新连接
        self.create_connection(conn.in_node, new_node_id, 1.0)
        self.create_connection(new_node_id, conn.out_node, conn.weight)
    
    def mutate_add_connection(self):
        """变异：添加连接"""
        # 随机选择两个不同的节点（输入->隐藏->输出的拓扑顺序）
        node_ids = list(self.nodes.keys())
        if len(node_ids) < 2:
            return
        
        # 避免前馈约束
        input_nodes = [n for n in node_ids if self.nodes[n].node_type == 'input']
        output_nodes = [n for n in node_ids if self.nodes[n].node_type == 'output']
        hidden_nodes = [n for n in node_ids if self.nodes[n].node_type == 'hidden']
        
        # 随机选择一个源节点和目标节点
        all_sources = input_nodes + hidden_nodes
        all_targets = hidden_nodes + output_nodes
        
        if not all_sources or not all_targets:
            return
        
        in_node = random.choice(all_sources)
        out_node = random.choice(all_targets)
        
        if in_node == out_node:
            return
        
        self.create_connection(in_node, out_node)
    
    def mutate_weights(self, rate: float = 0.8, step: float = 0.1):
        """变异：修改权重"""
        for conn in self.connections.values():
            if random.random() < rate:
                if random.random() < 0.1:
                    # 重新随机
                    conn.weight = random.uniform(-1, 1)
                else:
                    # 微调
                    conn.weight += random.uniform(-step, step)
    
    def forward(self, inputs: List[float]) -> List[float]:
        """前向传播"""
        # 拓扑排序
        node_values = {}
        
        # 初始化输入节点
        input_nodes = sorted([n for n in self.nodes if self.nodes[n].node_type == 'input'])
        for i, node_id in enumerate(input_nodes):
            node_values[node_id] = inputs[i] if i < len(inputs) else 0
        
        # 按拓扑顺序计算
        output_nodes = sorted([n for n in self.nodes if self.nodes[n].node_type == 'output'])
        hidden_nodes = sorted([n for n in self.nodes if self.nodes[n].node_type == 'hidden'])
        
        for node_id in hidden_nodes + output_nodes:
            # 收集所有输入连接
            total = 0
            for conn in self.connections.values():
                if conn.out_node == node_id and conn.enabled:
                    total += node_values.get(conn.in_node, 0) * conn.weight
            node_values[node_id] = 1 / (1 + np.exp(-total))  # sigmoid
        
        return [node_values.get(n, 0) for n in output_nodes]


class NEAT:
    """简化版NEAT算法"""
    
    def __init__(self, 
                 num_inputs: int, 
                 num_outputs: int,
                 population_size: int = 150,
                 species_threshold: float = 3.0):
        self.num_inputs = num_inputs
        self.num_outputs = num_outputs
        self.population_size = population_size
        self.species_threshold = species_threshold
        self.species = {}
        self.best_fitness = 0
        self.generation = 0
        
    def create_initial_population(self) -> List[Genome]:
        """创建初始种群"""
        population = []
        for _ in range(self.population_size):
            genome = Genome()
            
            # 添加输入节点
            for i in range(self.num_inputs):
                genome.create_node('input')
            
            # 添加输出节点
            for i in range(self.num_outputs):
                genome.create_node('output')
            
            # 初始化时全连接（可选）
            for i in range(self.num_inputs):
                for j in range(self.num_outputs):
                    genome.create_connection(i, self.num_inputs + j)
            
            population.append(genome)
        
        return population
    
    def compatibility_distance(self, g1: Genome, g2: Genome) -> float:
        """计算两个基因组的兼容性距离"""
        # 简化版本：比较连接的差异
        conn1 = set((c.in_node, c.out_node) for c in g1.connections.values())
        conn2 = set((c.in_node, c.out_node) for c in g2.connections.values())
        
        disjoint = len(conn1 ^ conn2)
        excess = len(conn1 - conn2) + len(conn2 - conn1)
        
        # 简化距离计算
        return disjoint * 0.5 + excess * 1.0
    
    def speciate(self, population: List[Genome]):
        """物种形成"""
        self.species = {}
        
        for genome in population:
            found_species = False
            for species_id, reps in self.species.items():
                if self.compatibility_distance(genome, reps) < self.species_threshold:
                    self.species[species_id].append(genome)
                    found_species = True
                    break
            
            if not found_species:
                species_id = len(self.species)
                self.species[species_id] = [genome]
    
    def evolve(self, fitness_function, generations: int = 100) -> Genome:
        """进化主循环"""
        population = self.create_initial_population()
        
        for gen in range(generations):
            # 评估适应度
            for genome in population:
                genome.fitness = fitness_function(genome)
            
            # 找最佳个体
            best = max(population, key=lambda g: g.fitness)
            if best.fitness > self.best_fitness:
                self.best_fitness = best.fitness
            
            print(f"Generation {gen}: Best fitness = {self.best_fitness:.4f}")
            
            # 物种形成
            self.speciate(population)
            
            # 选择与繁殖
            new_population = []
            
            for species_id, members in self.species.items():
                # 计算物种健康度
                species_fitness = [m.fitness for m in members]
                avg_fitness = sum(species_fitness) / len(species_fitness)
                
                # 每个物种的繁殖数量按适应度比例分配
                num_offspring = max(1, int(self.population_size * avg_fitness / sum([m.fitness for m in population]) + 0.5))
                
                # 精英保留
                members.sort(key=lambda m: m.fitness, reverse=True)
                if members:
                    new_population.append(members[0])  # 精英直接复制
                    num_offspring -= 1
                
                # 锦标赛选择 + 交叉
                for _ in range(num_offspring):
                    parent1 = random.choice(members)
                    parent2 = random.choice(members)
                    
                    # 简化版：直接变异复制
                    child = self.crossover(parent1, parent2)
                    
                    # 变异
                    if random.random() < 0.1:
                        child.mutate_add_node()
                    if random.random() < 0.1:
                        child.mutate_add_connection()
                    child.mutate_weights()
                    
                    new_population.append(child)
            
            # 截断或填充
            population = new_population[:self.population_size]
            while len(population) < self.population_size:
                population.append(self.create_initial_population()[0])
        
        return max(population, key=lambda g: g.fitness)
    
    def crossover(self, parent1: Genome, parent2: Genome) -> Genome:
        """交叉操作（简化版）"""
        # 简化：直接复制父本1
        child = Genome()
        child.nodes = parent1.nodes.copy()
        child.connections = {k: ConnectionGene(
            c.in_node, c.out_node, c.weight, c.innovation, c.enabled
        ) for k, c in parent1.connections.items()}
        return child


# 测试：XOR问题
def xor_fitness(genome: Genome) -> float:
    """XOR问题的适应度函数"""
    test_cases = [
        ([0, 0], [0]),
        ([0, 1], [1]),
        ([1, 0], [1]),
        ([1, 1], [0])
    ]
    
    total_error = 0
    for inputs, target in test_cases:
        output = genome.forward(inputs)
        total_error += (output[0] - target[0]) ** 2
    
    # 适应度 = 1 / (1 + error)，误差越小适应度越高
    return 1 / (1 + total_error)


if __name__ == "__main__":
    neat = NEAT(num_inputs=2, num_outputs=1, population_size=150)
    best = neat.evolve(xor_fitness, generations=50)
    print(f"\nBest genome fitness: {best.fitness:.4f}")
    print(f"Number of nodes: {len(best.nodes)}")
    print(f"Number of connections: {len(best.connections)}")
    
    # 测试结果
    print("\nXOR Test Results:")
    for inputs in [[0,0], [0,1], [1,0], [1,1]]:
        output = best.forward(inputs)
        print(f"  {inputs} -> {output[0]:.3f}")
```

这个实现比较简化，但涵盖了NEAT的核心思想。如果你想要完整版，推荐直接用 `neat-python` 库，那是Kenneth Stanley的官方实现。

## WANN：权重无关神经网络

WANN（Weight-Aware Neural Networks）是另一个有意思的方向。和NEAT不同，WANN在进化过程中**根本不优化权重**，所有连接统一用1或随机值。你可能会问，这样做出来的网络能有什么效果？

答案是：WANN赌的是**结构本身的信息传递能力**。如果一个网络结构足够好，即使权重都是固定的1或-1，它仍然能完成一些简单的任务。WANN的研究者发现，很多任务的最佳结构其实不需要精细调参也能工作。

WANN的搜索空间比传统神经进化小很多，因为它不需要编码权重。这让它在大规模结构搜索时效率更高。

WANN的典型应用场景包括：
- 强化学习中的结构发现
- 快速原型验证
- 需要迁移学习的场景（结构可以迁移，权重不一定）

## 拓扑进化：如何表示和变异网络结构

聊完NEAT和WANN，我们来深入聊聊拓扑进化本身。拓扑进化最核心的问题有两个：**怎么表示网络结构**和**怎么变异结构**。

### 结构表示方法

**1. 直接编码（Direct Encoding）**

每个可能的连接都有一个基因位，0表示不存在，1表示存在。这种方法简单直接，但随着网络变大，基因长度爆炸式增长。而且对于大网络，大部分基因都是0，非常浪费。

NEAT用的就是直接编码，但用了历史标记来高效处理。

**2. 间接编码（Indirect Encoding）**

借鉴生物发育的概念，用一套规则来生成网络结构。比如你可以定义：
- 每隔N个神经元添加一个侧连接
- 每一层比上一层少M个神经元

这种方法的搜索空间更紧凑，但映射关系可能很复杂。

**3. 基于图的编码**

把网络结构看作一个图，用邻接矩阵或邻接表来表示。变异操作包括添加/删除节点、添加/删除边、改变边属性等。

### 常用变异算子

- **添加节点**：选择一个连接断开，插入一个新节点
- **添加连接**：在两个不相连的节点之间创建新连接
- **删除节点**：移除一个节点及其所有连接
- **删除连接**：移除一个连接
- **改变激活函数**：切换节点的激活函数类型

实战中，添加节点和添加连接是最常用的变异，因为它们能持续增加网络的表达能力。

## 协同进化：竞争与合作的进化动力学

协同进化（Coevolution）是神经进化中最有趣的分支之一。在协同进化中，有多个种群同时进化，它们的适应度互相依赖。

### 捕食者-猎物模型

想象一个场景：捕食者种群和猎物种群同时进化。捕食者越强，就越容易抓到猎物；猎物越强，就越难被抓住。这是一个经典的"军备竞赛"。

协同进化的挑战在于适应度评估：你怎么知道一个捕食者好不好？答案是看它相对于当前猎物种群的表现，而不是一个固定的测试集。

### 合作进化

协同进化也可以是正向的。比如在多智能体强化学习中，多个智能体需要合作完成任务。每个智能体的适应度取决于整个团队的表现。

**DeepMind的QMIX和QTRAN**等算法就用到了类似的思路，但那是基于值函数的方法。有研究尝试用神经进化来做多智能体合作，比如**Neuroevolution of Multiagent Strategies**。

协同进化的一个常见问题是**循环依赖**：A好是因为B，B好是因为C，C好是因为A，形成闭环。解决方法是引入历史记录，确保每次比较都是在相近的时间点上进行。

## 神经进化在AutoML中的应用

AutoML的最终目标是自动化机器学习的所有环节——特征工程、模型选择、超参数调优、网络结构搜索。神经进化天然适合做结构搜索，因为它能同时优化很多东西。

### NEATO

NEATO是一个用神经进化做神经网络结构搜索的框架。它和NASNet、DARTS这些基于梯度的方法不太一样——它完全用进化来搜索结构。

NEATO的搜索空间包括：
- 卷积层数
- 每层的滤波器数量
- 核大小
- 跳跃连接的位置

搜索过程在大数据集上可能很慢，但它能找到梯度方法找不到的结构。

### AutoML-Zero

AutoML-Zero是Google在2020年提出的一个野心勃勃的项目——它试图从零开始进化出机器学习算法。

AutoML-Zero的搜索空间非常底层：
- 怎么初始化参数
- 怎么计算预测
- 怎么计算梯度
- 怎么更新参数

结果你猜怎么着？它真的进化出了一些看起来像简单机器学习算法的结构，比如类似线性回归和梯度下降的东西。

当然，AutoML-Zero的规模有限，不可能进化出Transformer这种东西。但它证明了神经进化的潜力——在足够长的进化时间和足够大的计算资源下，可能涌现出意想不到的算法。

## 神经进化 vs 梯度下降

这是很多人关心的问题：到底用哪个？

**梯度下降的优势：**
- 效率高，尤其是大网络
- 理论保证（凸优化）
- 适合大规模数据

**神经进化的优势：**
- 不需要梯度
- 能同时优化结构和权重
- 全局搜索能力强
- 适合小规模、稀疏奖励问题

**实战建议：**
- 小数据集、小网络：试试神经进化
- 大数据集、大网络：梯度下降
- 结构搜索：神经进化
- 强化学习：神经进化可能更合适
- 复杂结构如Transformer：老老实实用梯度

实际上，**两者结合**才是王道。很多state-of-the-art的工作都是先用神经进化搜索出一个好的结构，然后用梯度下降微调权重。这叫"两阶段搜索"，既能发挥神经进化的全局搜索能力，又能利用梯度下降的效率。

## 代码实战：用NEAT解决CartPole平衡问题

CartPole是一个经典的强化学习任务：让小车上的杆保持平衡。杆会自然倒下，你的目标是控制小车的左右移动来保持杆竖直。

```python
import gym
import numpy as np
from neat import NEAT

# 创建环境
env = gym.make('CartPole-v1')

def evaluate_genome(genome, env, episodes=3, max_steps=500):
    """评估一个基因组的适应度"""
    total_reward = 0
    
    for _ in range(episodes):
        state = env.reset()
        for step in range(max_steps):
            # 用神经网络决定动作
            action_probs = genome.forward(state)
            action = 0 if action_probs[0] < 0.5 else 1
            
            state, reward, done, _ = env.step(action)
            total_reward += reward
            
            if done:
                break
    
    return total_reward / episodes

# 简化的NEAT实现（使用neat-python库）
# 实际使用时推荐安装：pip install neat-python

def train_cartpole():
    """训练CartPole"""
    import neat
    
    def eval_genomes(genomes, config):
        for genome_id, genome in genomes:
            net = neat.nn.FeedForwardNetwork.create(genome, config)
            
            total_reward = 0
            for _ in range(3):
                observation = env.reset()
                for t in range(500):
                    action = 0 if net.activate(observation)[0] < 0.5 else 1
                    observation, reward, done, _ = env.step(action)
                    total_reward += reward
                    if done:
                        break
            
            genome.fitness = total_reward / 3
    
    # NEAT配置
    config = neat.Config(
        neat.DefaultGenome,
        neat.DefaultReproduction,
        neat.DefaultSpeciesSet,
        neat.DefaultStagnation,
        'config-feedforward.txt'  # 需要NEAT配置文件
    )
    
    # 创建种群
    population = neat.Population(config)
    population.add_reporter(neat.StdOutReporter(True))
    stats = neat.StatisticsReporter()
    population.add_reporter(stats)
    
    # 进化
    winner = population.run(eval_genomes, 100)
    
    # 测试
    env = gym.make('CartPole-v1')
    net = neat.nn.FeedForwardNetwork.create(winner, config)
    
    for episode in range(5):
        observation = env.reset()
        total_reward = 0
        for t in range(500):
            action = 0 if net.activate(observation)[0] < 0.5 else 1
            observation, reward, done, _ = env.step(action)
            total_reward += reward
            if done:
                print(f"Episode {episode+1}: {total_reward} steps")
                break

if __name__ == "__main__":
    # 注意：需要安装neat-python和gym
    # pip install neat-python gymnasium
    print("CartPole NEAT training example")
    print("Install requirements: pip install neat-python gymnasium")
```

运行这个代码需要安装依赖并创建配置文件。配置文件 `config-feedforward.txt` 的内容大致如下：

```ini
[NEAT]
fitness_criterion     = max
fitness_threshold     = 475
pop_size              = 150
reset_on_extinction   = False

[DefaultGenome]
num_inputs              = 4
num_outputs             = 1
initial_connection      = unweighted
node_add_prob           = 0.2
node_delete_prob        = 0.2
connection_add_prob     = 0.5
connection_delete_prob  = 0.5

# ... 更多配置项
```

神经进化在CartPole这种简单任务上通常能收敛到接近最优解。对于更复杂的任务，可能需要：
- 更大的种群
- 更多的进化代数
- 专门的适应度塑形（reward shaping）

## 实战经验：神经进化的收敛速度与效果评估

### 收敛速度

神经进化的收敛速度通常比梯度下降慢。原因很明显：进化是在离散、非光滑的搜索空间中探索，而梯度下降可以利用函数的局部几何信息。

但是，这个"慢"是有代价的——梯度下降可能在局部最优卡住，而进化有更大的概率逃出来。

**加速技巧：**

1. **种群初始化**：不要从完全随机的网络开始。用预训练的小网络初始化，或者从专家设计的简单结构开始。

2. **适应度塑形**：设计更好的适应度函数，让进化有更明确的方向。比如在CartPole任务中，不仅看最终坚持了多少步，还要看杆的平均角度。

3. **参数自适应**：交叉率、变异率这些参数可以在进化过程中自适应调整。

4. **知识迁移**：当发现一个好的结构时，可以把它作为新种群的起点。

### 效果评估

神经进化的评估比梯度下降更复杂，因为你很难说"这是局部最优"或"这已经收敛了"。

**实用指标：**

1. **Best-so-far曲线**：记录每一代最佳个体的适应度。这条曲线应该总体向上，但也会有波动。

2. **种群多样性**：衡量种群中个体的差异程度。如果多样性太低，进化可能已经陷入局部最优。

3. **结构复杂度**：记录进化出来的网络大小。如果网络越来越复杂但适应度没有提升，说明在过拟合。

4. **多次运行方差**：神经进化有随机性，同一个实验跑多次会有不同结果。报告时应该包含均值和方差。

### 常见坑

**1. 维度灾难**：当输入维度很高时（比如图像），纯神经进化很难处理。通常需要先做降维或特征提取。

**2. 计算成本**：种群规模×个体复杂度×进化代数，这个乘积可能很大。用GPU并行化可以帮助，但代码复杂度会增加。

**3. 超参数敏感**：神经进化本身就有很多超参数（种群大小、变异率等），调参本身就是一门艺术。

**4. 过拟合**：进化出来的网络可能对训练环境过度适配。换一个环境就失效。

**5. 不可复现**：由于随机性，每次运行结果可能不同。保存最佳个体的基因很重要。

## 总结

神经进化是一个"老树开新花"的领域。它不像深度学习那样有大量的工程技巧积累，但它的思想内核——用进化来搜索——在很多场景下依然有效。

现在的趋势是把神经进化和梯度方法结合起来：
- 用进化搜索结构
- 用梯度优化权重
- 用进化 + 梯度联合优化

如果你正在做一个新任务，不知道该用什么方法，不妨试试神经进化。尤其是当梯度不可用或者你需要同时搜索结构时，它可能是你的菜。

最后，实践出真知。找个简单的任务（比如XOR或CartPole），用本文学的代码跑一跑，感受一下进化的力量。
