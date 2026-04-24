---
title: 分层强化学习Hierarchical RL
date: 2026-04-24
tags:
  - 强化学习
  - 分层强化学习
  - Options框架
  - FeUdal Networks
  - HIRO
  - 课程学习
aliases:
  - 分层强化学习完全指南
  - Hierarchical RL详解
---

# 分层强化学习Hierarchical RL

## 分层强化学习的动机

想象你要完成一个任务：收拾房间。

这个任务太复杂了——你需要把衣服叠好放进衣柜，把书放到书架，把垃圾扔进垃圾桶，还要擦桌子。如果让你直接从"观察状态"到"执行动作"一步搞定，那强化学习根本学不会，因为奖励信号太稀疏了（可能只有把所有东西收拾完才给一个正向奖励）。

但人类不是这么干活的。我们会**分层抽象**：

- **高层目标**：先收拾卧室
- **中层目标**：叠衣服 → 放进衣柜
- **低层动作**：伸手、抓取、移动、放下

这就是分层强化学习（Hierarchical RL, HRL）的核心思想：**把复杂任务分解成多层嵌套的子任务，每层关注不同的时间尺度和抽象级别。**

分层RL的优势：
1. **解决稀疏奖励**：子任务有自己的内部奖励信号
2. **迁移泛化**：高层学到的技能可以迁移到新任务
3. **样本效率**：低层技能可以在多个任务中复用
4. **长时程规划**：不同层次的策略处理不同时间尺度的决策

## Options框架：子策略和终止条件

Options框架是Sutton、Precup和Singh在1999年提出的经典理论，是分层RL的基石。

核心概念是**Option**——一个"半自动"的策略，包含三个部分：

1. **内部策略** $\pi_u(a|s)$：在option内部如何选择动作
2. **终止条件** $\beta_u(s)$：在什么状态下option结束
3. **起始条件** $I_u$：在什么状态下可以开始这个option

```python
class Option:
    def __init__(self, policy, termination_fn, action_space):
        self.policy = policy  # 内部策略网络
        self.termination_fn = termination_fn  # 终止条件
        self.action_space = action_space
    
    def act(self, state):
        return self.policy(state)
    
    def should_terminate(self, state):
        return self.termination_fn(state)

class OptionCritic:
    def __init__(self, state_dim, num_options, action_dim):
        self.num_options = num_options
        
        # 每个option有自己的策略
        self.policies = [MLP(state_dim, action_dim) for _ in range(num_options)]
        
        # 每个option有自己的Q值函数
        self.Q = [MLP(state_dim, num_options) for _ in range(num_options)]
        
        # 每个option有自己的终止概率
        self.beta = [MLP(state_dim, 1) for _ in range(num_options)]
        
        # 当前激活的option
        self.current_option = 0
    
    def select_option(self, state):
        """选择进入哪个option"""
        # 用meta-policy选择
        state_t = torch.FloatTensor(state).unsqueeze(0)
        with torch.no_grad():
            q_values = torch.stack([Q(state_t) for Q in self.Q])
            self.current_option = q_values.argmax().item()
        return self.current_option
    
    def act(self, state):
        """在当前option下选择动作"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        with torch.no_grad():
            action = self.policies[self.current_option](state_t)
        return action
    
    def should_terminate(self, state):
        """检查是否终止当前option"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        with torch.no_grad():
            prob = torch.sigmoid(self.beta[self.current_option](state_t))
        return random.random() < prob.item()
```

**Option的更新**比较特殊，需要同时更新策略和终止条件：

```python
def update_option(self, option_idx, states, actions, rewards, next_states, dones):
    """更新一个option"""
    states = torch.FloatTensor(states)
    actions = torch.FloatTensor(actions)
    rewards = torch.FloatTensor(rewards)
    next_states = torch.FloatTensor(next_states)
    
    # Option-Internal Q值
    Q_u = self.Q[option_idx](states, actions)
    
    # 终止概率
    termination_prob = torch.sigmoid(self.beta[option_idx](next_states))
    
    # 下一个状态的最佳option
    with torch.no_grad():
        next_Q_values = torch.stack([Q(next_states) for Q in self.Q])
        next_best_option = next_Q_values.argmax(dim=1)
        
        # 如果终止，用V值；否则用Q值
        next_Q = (1 - termination_prob) * \
                 torch.gather(next_Q_values, 1, next_best_option.unsqueeze(1)).squeeze()
        
        target = rewards + 0.99 * next_Q
    
    # 更新Q函数
    q_loss = F.mse_loss(Q_u, target)
    
    # 更新内部策略（和标准策略梯度一样）
    log_probs = self.policies[option_idx].log_prob(states, actions)
    policy_loss = -(log_probs * Q_u.detach()).mean()
    
    # 更新终止条件：希望在高价值状态终止
    with torch.no_grad():
        v_next = next_Q_values.max(dim=1)[0]
    beta_loss = termination_prob * v_next  # 鼓励在高价值状态终止
    
    # 组合损失
    total_loss = q_loss + policy_loss + beta_loss.mean()
    total_loss.backward()
```

Options框架的挑战在于**option discovery**——怎么自动发现有用的options？手工设计太麻烦，自动学习又可能学出无意义的options。后续有很多研究解决这个问题，比如基于状态抽象的方法、基于技能发现的方法等。

## FeUdal Networks：管理者-工作者架构

FeUdal Networks（FuN）是DeepMind在2017年提出的分层架构。它的设计灵感来自 feudal system（封建制度）：管理者（Manager）设定目标，工作者（Worker）执行动作达成目标。

**核心思想**：不同层次有不同的时间抽象粒度。Manager的决策周期长，Worker的决策周期短。

```python
class FeudalNetwork(nn.Module):
    def __init__(self, state_dim, action_dim, goal_dim=8, hidden=256):
        super().__init__()
        self.goal_dim = goal_dim
        
        # ============= Manager =============
        # Manager观察状态，输出"目标"（goal）
        self.manager = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, goal_dim)  # 输出goal向量
        )
        
        # Manager的RNN，用于记忆和规划
        self.manager_rnn = nn.GRUCell(goal_dim, goal_dim)
        
        # Manager的内在奖励：当Worker达到目标时给奖励
        self.manager_reward_weight = 0.1
        
        # ============= Worker =============
        # Worker同时观察状态和目标，输出动作
        self.worker = nn.Sequential(
            nn.Linear(state_dim + goal_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, action_dim * 2)  # mean和std
        )
        
        # Worker的价值函数
        self.worker_value = nn.Sequential(
            nn.Linear(state_dim + goal_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, 1)
        )
        
        # Worker的RNN
        self.worker_rnn = nn.GRUCell(state_dim + goal_dim, hidden)
        
        self.hidden = hidden
        self.manager_hidden = None
        self.worker_hidden = None
    
    def reset_hidden(self):
        self.manager_hidden = torch.zeros(1, self.goal_dim)
        self.worker_hidden = torch.zeros(1, self.hidden)
    
    def manager_step(self, state):
        """Manager: 状态 -> 目标"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        
        with torch.no_grad():
            # 当前goal经过GRU
            new_goal = self.manager_rnn(self.manager_hidden, state_t)
        
        self.manager_hidden = new_goal.detach()
        return new_goal.squeeze().numpy()
    
    def worker_step(self, state, goal):
        """Worker: 状态 + 目标 -> 动作"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        goal_t = torch.FloatTensor(goal).unsqueeze(0)
        
        # 状态和目标拼接
        combined = torch.cat([state_t, goal_t], dim=1)
        
        # Worker RNN
        worker_input = combined
        self.worker_hidden = self.worker_rnn(self.worker_hidden, worker_input)
        
        # 输出动作
        output = self.worker(self.worker_hidden)
        mean, log_std = output.chunk(2, dim=1)
        
        dist = torch.distributions.Normal(mean, torch.exp(log_std))
        action = dist.sample()
        log_prob = dist.log_prob(action)
        
        return action.squeeze().numpy(), log_prob.squeeze(), dist.entropy().sum()
    
    def update_manager(self, states, goals, goal_rewards, manager_hidden_init):
        """
        更新Manager
        goal_rewards: Worker达到目标时给Manager的内在奖励
        """
        goals = torch.FloatTensor(goals)
        goal_rewards = torch.FloatTensor(goal_rewards)
        
        # Manager的梯度上升（最大化累积goal奖励）
        manager_loss = -goal_rewards.mean()
        manager_loss.backward()
    
    def update_worker(self, states, goals, actions, log_probs_old, returns):
        """更新Worker"""
        states = torch.FloatTensor(states)
        goals = torch.FloatTensor(goals)
        actions = torch.FloatTensor(actions)
        returns = torch.FloatTensor(returns)
        
        combined = torch.cat([states, goals], dim=1)
        
        # Worker的价值
        values = self.worker_value(combined).squeeze()
        
        # 优势
        advantages = returns - values.detach()
        
        # 策略梯度
        mean, log_std = self.worker(combined).chunk(2, dim=1)
        dist = torch.distributions.Normal(mean, torch.exp(log_std))
        log_probs = dist.log_prob(actions).sum(dim=-1)
        
        policy_loss = -(log_probs * advantages).mean()
        value_loss = F.mse_loss(values, returns)
        
        return policy_loss + 0.5 * value_loss
```

**FuN的关键创新**：

1. **目标表示**：Manager输出的不是具体动作，而是一个"目标向量"，这个向量编码了"Worker应该往哪个方向移动"
2. **时间尺度分离**：Manager每隔$T$步更新一次目标，Worker每步都执行动作
3. **跨任务迁移**：只要目标空间定义得好，Manager学到的技能可以迁移

## MAXQ分解：任务分解的另一种视角

MAXQ是Tom Dietterich提出的另一种分层框架，它关注的是**任务分解的结构**。

MAXQ的核心是把一个MDP分解成一个层次任务图（Hierarchical Task Decomposition）：

```
MAX (root task)
├── Subtask A
│   ├── Primitive 1
│   └── Primitive 2
└── Subtask B
    ├── Subtask B1
    └── Subtask B2
```

每个子任务有自己的Q函数，父任务通过调用子任务来完成任务。

```python
class MAXQDecomposition:
    def __init__(self, task_graph):
        """
        task_graph: 定义任务分解结构
        例如: {'root': ['navigate', 'pick', 'place'],
              'navigate': ['move_forward', 'turn'],
              ...}
        """
        self.task_graph = task_graph
        self.Q_functions = {}
        self.value_functions = {}
        
        for task in task_graph:
            self.Q_functions[task] = {}
            self.value_functions[task] = {}
    
    def get_action(self, state, task):
        """给定状态和任务，返回下一步应该执行什么"""
        if task in self.Q_functions:
            # 这是叶子节点（原始动作），直接选择
            return max(self.Q_functions[task].items(), 
                      key=lambda x: x[1])[0]
        else:
            # 这是复合任务，选择最优子任务
            return max(self.Q_functions[task].items(),
                      key=lambda x: x[1])[0]
    
    def recursive_update(self, trajectory, task):
        """
        递归更新MAXQ的Q值
        """
        states, actions, rewards = trajectory
        
        if len(actions) == 1:
            # 叶子节点，更新原始Q函数
            self.update_primitive_q(states[0], actions[0], sum(rewards))
        else:
            # 分解为子任务
            subtasks = self.task_graph[task]
            
            # 对每个子任务递归更新
            for subtask in subtasks:
                self.recursive_update(trajectory, subtask)
```

MAXQ的优点是**可解释性强**——你能清楚看到任务是怎么分解的。但缺点是**需要手工设计任务分解结构**，自动发现还是个难题。

## HIRO：从数据中学习内在奖励

HIRO（HIerarchical Reinforcement with Off-Policy correction）是DeepMind 2018年的工作，解决的是Options框架的一个实际问题：**怎么在没有显式标签的情况下学习好的高层策略？**

关键洞察是：**可以用数据驱动的思想，让高层策略学习"如何让低层策略做它本来就会做的事"**。

```python
class HIRO:
    def __init__(self, state_dim, action_dim, goal_dim=6, hidden=256):
        self.goal_dim = goal_dim
        
        # ============= Low-Level Policy =============
        self.low_level = nn.Sequential(
            nn.Linear(state_dim + goal_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, action_dim * 2)
        )
        
        # ============= High-Level Policy =============
        self.high_level = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, goal_dim)
        )
        
        # ============= Q函数 =============
        self.Q_high = nn.Sequential(
            nn.Linear(state_dim + goal_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, 1)
        )
        
        self.Q_low = nn.Sequential(
            nn.Linear(state_dim + goal_dim + action_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, 1)
        )
        
        self.replay_buffer = ReplayBuffer(capacity=100000)
        self.goal_replay_buffer = ReplayBuffer(capacity=100000)
        
        self.c = 10  # 高层策略更新频率（每隔10步）
        
    def sample_goal(self, state, next_state):
        """从(state, next_state)对中反推出goal"""
        # 简单版本：goal = next_state - state
        goal = next_state[:self.goal_dim] - state[:self.goal_dim]
        
        # 更好的版本：用学到的goal embedding
        return goal
    
    def high_level_action(self, state):
        """高层策略：状态 -> 目标"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        goal = self.high_level(state_t)
        return goal.squeeze().detach().numpy()
    
    def low_level_action(self, state, goal):
        """低层策略：状态 + 目标 -> 动作"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        goal_t = torch.FloatTensor(goal).unsqueeze(0)
        
        combined = torch.cat([state_t, goal_t], dim=1)
        mean, log_std = self.low_level(combined).chunk(2, dim=1)
        
        dist = torch.distributions.Normal(mean, torch.exp(log_std))
        action = dist.sample()
        return action.squeeze().numpy()
    
    def compute_intrinsic_reward(self, state, goal, next_state):
        """
        内在奖励：鼓励低层策略去达成高层设定的目标
        """
        # 目标达成度 = 和目标的接近程度
        goal_error = np.linalg.norm(next_state[:self.goal_dim] - state[:self.goal_dim] - goal)
        return -goal_error
    
    def update(self, batch):
        """Off-policy更新"""
        states, actions, rewards, next_states, goals = batch
        
        # 外在奖励
        extrinsic_rewards = torch.FloatTensor(rewards)
        
        # 内在奖励
        intrinsic_rewards = torch.FloatTensor([
            self.compute_intrinsic_reward(s, g, ns) 
            for s, g, ns in zip(states, goals, next_states)
        ])
        
        total_rewards = extrinsic_rewards + 0.1 * intrinsic_rewards
        
        # 更新低层策略
        for s, g, a, r, ns in zip(states, goals, actions, total_rewards, next_states):
            s_t = torch.FloatTensor(s).unsqueeze(0)
            g_t = torch.FloatTensor(g).unsqueeze(0)
            a_t = torch.FloatTensor(a).unsqueeze(0)
            r_t = r.unsqueeze(0)
            ns_t = torch.FloatTensor(ns).unsqueeze(0)
            
            # 内在奖励
            intr_r = self.compute_intrinsic_reward(s, g, ns)
            
            # Q值更新
            combined = torch.cat([s_t, g_t, a_t], dim=1)
            q_value = self.Q_low(combined)
            
            with torch.no_grad():
                next_combined = torch.cat([ns_t, g_t], dim=1)
                next_actions = self.low_level(next_combined)[:, :a_t.shape[1]]
                next_combined = torch.cat([next_combined, next_actions], dim=1)
                next_q = self.Q_low(next_combined)
                
                target = r_t + 0.99 * next_q
            
            loss = F.mse_loss(q_value, target)
            loss.backward()
    
    def off_policy_correction(self, goals, states, next_states, actions):
        """
        Off-policy correction: 把on-policy的goal转换成off-policy的
        这是HIRO的核心创新
        """
        corrected_goals = []
        
        for i in range(len(goals)):
            goal = goals[i]
            state = states[i]
            next_state = next_states[i]
            action = actions[i]
            
            # 在下一状态执行同一动作，反推需要的goal
            # 这是一个反向的"what goal would lead to this action?"
            state_t = torch.FloatTensor(state).unsqueeze(0)
            next_state_t = torch.FloatTensor(next_state).unsqueeze(0)
            action_t = torch.FloatTensor(action).unsqueeze(0)
            
            # 简化：用状态差分作为goal
            corrected_goal = next_state[:self.goal_dim] - state[:self.goal_dim]
            corrected_goals.append(corrected_goal)
        
        return corrected_goals
```

**HIRO的核心贡献**是解决了off-policy训练的难题。因为高层策略是on-policy的（每个episode重新采样goal），但低层策略需要off-policy更新（从replay buffer中学习）。HIRO通过off-policy correction来处理这个不匹配。

## 跨层次迁移学习

分层RL的一个重要优势是**迁移学习**：高层学到的技能可以迁移到新任务。

```python
class HierarchicalTransfer:
    def __init__(self, source_tasks, target_task):
        self.source_tasks = source_tasks
        self.target_task = target_task
        
    def transfer_high_level(self, source_model, target_model):
        """迁移高层策略"""
        # 高层策略往往更抽象，迁移性更强
        target_model.high_level.load_state_dict(source_model.high_level.state_dict())
        
        # 但需要重新训练goal空间到新任务的映射
        for param in target_model.high_level.parameters():
            param.requires_grad = True  # 还是微调
    
    def transfer_low_level(self, source_model, target_model, skill_ids):
        """
        迁移低层技能
        skill_ids: 哪些skill可以迁移
        """
        for skill_id in skill_ids:
            # 低层skill通常是primitive的，迁移性更强
            target_model.low_levels[skill_id].load_state_dict(
                source_model.low_levels[skill_id].state_dict()
            )
    
    def curriculum_learning(self, model, tasks):
        """
        课程学习：从简单任务到复杂任务
        """
        # 按难度排序
        sorted_tasks = sorted(tasks, key=lambda t: t.difficulty)
        
        for task in sorted_tasks:
            print(f"Training on task: {task.name}")
            
            # 用前一个任务的策略初始化
            if task != sorted_tasks[0]:
                model.load_state_dict(previous_model.state_dict())
            
            # 训练当前任务
            train_task(model, task)
            
            previous_model = model
```

迁移学习的关键洞察：**越高层越抽象，迁移性越好；越低层越具体，迁移性越差**。所以实践中通常迁移高层策略，低层策略在新任务上重新训练或微调。

## 代码实战：实现Options框架

下面实现一个完整的Options框架来解决子任务发现问题。

```python
import gymnasium as gym
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
from collections import deque, defaultdict
import random

# ============== Options框架实现 ==============
class OptionNetwork(nn.Module):
    """单个Option的策略网络"""
    def __init__(self, state_dim, action_dim, hidden=64):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, action_dim * 2)
        )
    
    def forward(self, x):
        return self.network(x)
    
    def get_action(self, state, deterministic=False):
        output = self.network(state)
        mean, log_std = output.chunk(2, dim=-1)
        std = torch.exp(log_std)
        
        if deterministic:
            return torch.tanh(mean)
        
        dist = torch.distributions.Normal(mean, std)
        action = dist.sample()
        return torch.tanh(action)

class TerminationNetwork(nn.Module):
    """Option的终止条件网络"""
    def __init__(self, state_dim, hidden=64):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, 1)
        )
    
    def forward(self, x):
        return torch.sigmoid(self.network(x))  # [0, 1] 终止概率

class OptionCritic:
    def __init__(self, state_dim, action_dim, num_options=4):
        self.num_options = num_options
        
        # 每个option有自己的策略
        self.options = [OptionNetwork(state_dim, action_dim) for _ in range(num_options)]
        
        # 每个option有自己的终止条件
        self.terminations = [TerminationNetwork(state_dim) for _ in range(num_options)]
        
        # Q函数：state-option -> Q值
        self.Q = nn.Sequential(
            nn.Linear(state_dim + num_options, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, num_options)
        )
        
        # 优化器
        self.opt_optimizer = optim.Adam(
            list(self.Q.parameters()) + 
            sum([[opt.parameters(), term.parameters()] 
                for opt, term in zip(self.options, self.terminations)], []),
            lr=3e-4
        )
    
    def select_option(self, state, epsilon=0.0):
        """选择进入哪个option"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        
        with torch.no_grad():
            q_values = self.Q(torch.cat([state_t, torch.zeros(1, self.num_options)], dim=1))
            q_values = q_values.squeeze()
            
            if random.random() < epsilon:
                return random.randint(0, self.num_options - 1)
            return q_values.argmax().item()
    
    def get_action(self, state, option):
        """在当前option下选择动作"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        return self.options[option].get_action(state_t).squeeze().numpy()
    
    def should_terminate(self, state, option):
        """判断是否终止当前option"""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        with torch.no_grad():
            prob = self.terminations[option](state_t).item()
        return random.random() < prob
    
    def update(self, states, actions, rewards, next_states, options, dones):
        """更新所有option"""
        states_t = torch.FloatTensor(np.array(states))
        actions_t = torch.FloatTensor(np.array(actions))
        rewards_t = torch.FloatTensor(np.array(rewards))
        next_states_t = torch.FloatTensor(np.array(next_states))
        
        # 创建option的one-hot编码
        options_t = torch.zeros(len(options), self.num_options)
        for i, opt in enumerate(options):
            options_t[i, opt] = 1.0
        
        # 当前Q值
        combined = torch.cat([states_t, options_t], dim=1)
        current_Q = self.Q(combined)
        
        # 目标Q值
        with torch.no_grad():
            # 每个option的终止概率
            termination_probs = torch.stack([
                self.terminations[opt](next_states_t) 
                for opt in range(self.num_options)
            ], dim=1)  # [batch, num_options]
            
            # 每个option的V值
            next_q_values = self.Q(torch.cat([next_states_t, options_t], dim=1))
            
            # 如果终止，用V值；否则用Q值
            next_Q = (1 - termination_probs) * next_q_values + \
                     termination_probs * next_q_values.max(dim=1, keepdim=True)[0]
            
            target = rewards_t.unsqueeze(1) + 0.99 * next_Q
            target = target.detach()
        
        # Q函数损失
        q_loss = F.mse_loss(current_Q, target)
        
        # Option策略损失：最大化当前option的Q值
        current_option_actions = torch.stack([
            self.options[opt](states_t) for opt in range(self.num_options)
        ])  # [num_options, batch, action_dim]
        
        # 计算每个option的策略梯度
        policy_loss = 0
        for opt in range(self.num_options):
            mask = (torch.argmax(options_t, dim=1) == opt).float().unsqueeze(1)
            if mask.sum() > 0:
                # 选中当前option的样本才更新
                q_grad = current_Q[:, opt].detach()
                policy_loss -= (q_grad * mask).mean()
        
        total_loss = q_loss + 0.01 * policy_loss
        
        self.opt_optimizer.zero_grad()
        total_loss.backward()
        self.opt_optimizer.step()
        
        return q_loss.item(), policy_loss.item()

# ============== 训练循环 ==============
def train_options(env_name='FetchReach-v2', num_episodes=500):
    env = gym.make(env_name)
    
    state_dim = env.observation_space.shape[0]
    action_dim = env.action_space.shape[0] if hasattr(env.action_space, 'shape') else env.action_space.n
    
    agent = OptionCritic(state_dim, action_dim, num_options=4)
    
    episode_rewards = []
    
    for episode in range(num_episodes):
        state, _ = env.reset()
        episode_reward = 0
        episode_history = defaultdict(list)
        
        # 选择初始option
        current_option = agent.select_option(state, epsilon=0.3)
        option_start = True
        
        while True:
            # 执行当前option的步数（最多10步）
            option_steps = 0
            option_rewards = []
            
            while option_steps < 10 and not agent.should_terminate(state, current_option):
                # 选择动作
                action = agent.get_action(state, current_option)
                action_clipped = np.clip(action, -1, 1)
                
                # 执行
                next_state, reward, terminated, truncated, _ = env.step(action_clipped)
                done = terminated or truncated
                
                # 记录
                episode_history['states'].append(state)
                episode_history['actions'].append(action)
                episode_history['rewards'].append(reward)
                episode_history['next_states'].append(next_state)
                episode_history['options'].append(current_option)
                episode_history['dones'].append(done)
                
                episode_reward += reward
                option_rewards.append(reward)
                state = next_state
                option_steps += 1
                
                if done:
                    break
            
            # 如果到了最后一个状态，终止
            if done:
                break
            
            # 更新当前option
            if len(episode_history['states']) > 32:
                batch = {
                    'states': episode_history['states'][-32:],
                    'actions': episode_history['actions'][-32:],
                    'rewards': episode_history['rewards'][-32:],
                    'next_states': episode_history['next_states'][-32:],
                    'options': episode_history['options'][-32:],
                    'dones': episode_history['dones'][-32:]
                }
                q_loss, policy_loss = agent.update(**batch)
            
            # 选择新option
            current_option = agent.select_option(state, epsilon=0.3)
        
        episode_rewards.append(episode_reward)
        
        if episode % 10 == 0:
            avg_reward = np.mean(episode_rewards[-10:])
            print(f"Episode {episode}: Avg Reward = {avg_reward:.2f}")
        
        if episode > 50 and np.mean(episode_rewards[-20:]) > 450:
            print(f"Solved at episode {episode}!")
            break
    
    return agent

if __name__ == '__main__':
    print("Training with Options framework...")
    agent = train_options('FetchReach-v2', num_episodes=300)
```

## 实战案例：机械臂多步骤操作

下面用一个更复杂的例子展示分层RL在机械臂任务中的应用。

```python
class RoboticManipulationHRL:
    """
    机械臂分层强化学习系统
    
    任务：Pick and Place
    分层结构：
    - Level 2 (High): 任务规划 - 决定当前应该pick还是place
    - Level 1 (Mid): 目标姿态 - 决定要移动到哪里
    - Level 0 (Low): 关节控制 - 具体的关节角度
    """
    
    def __init__(self, object_dim=10, target_dim=3):
        self.object_dim = object_dim
        self.target_dim = target_dim
        
        # Level 2: 任务管理器
        self.task_manager = TaskManager(object_dim)
        
        # Level 1: 姿态规划器
        self.pose_planner = PosePlanner(object_dim + 3)  # 物体位置 + 目标位置
        
        # Level 0: 关节控制器
        self.joint_controller = JointController(object_dim, action_dim=7)
        
    def high_level_plan(self, observation):
        """
        高级规划：观察 -> 任务类型
        返回: {'task': 'pick' or 'place', 'object_id': int, 'target_id': int}
        """
        state = self.task_manager.encode(observation)
        task_embedding = self.task_manager.forward(state)
        return self.task_manager.decode_task(task_embedding)
    
    def mid_level_plan(self, observation, high_level_plan):
        """
        中级规划：计算目标姿态
        返回: target_pose (位置+朝向)
        """
        state = observation['object_pos']
        target = observation['target_pos']
        
        if high_level_plan['task'] == 'pick':
            # 移动到物体上方
            goal_pose = state + np.array([0, 0, 0.1])
        else:
            # 移动到目标位置
            goal_pose = target
        
        return goal_pose
    
    def low_level_control(self, observation, target_pose, max_steps=20):
        """
        低级控制：跟踪目标姿态
        """
        state = observation['joint_positions']
        action = self.joint_controller.get_action(state, target_pose)
        
        return action
    
    def execute_task(self, observation):
        """
        执行完整任务
        """
        # 1. 高级规划
        high_plan = self.high_level_plan(observation)
        
        # 2. 中级规划
        mid_plan = self.mid_level_plan(observation, high_plan)
        
        # 3. 低级执行（滚动horizon）
        total_reward = 0
        for _ in range(max_steps):
            action = self.low_level_control(observation, mid_plan)
            observation, reward, done, _ = self.env.step(action)
            total_reward += reward
            if done:
                break
        
        return total_reward

def train_robotic_hrl():
    """
    训练机械臂分层RL
    """
    env = gym.make('FetchPickAndPlace-v2')
    agent = RoboticManipulationHRL()
    
    optimizer_high = optim.Adam(agent.task_manager.parameters(), lr=1e-4)
    optimizer_mid = optim.Adam(agent.pose_planner.parameters(), lr=3e-4)
    optimizer_low = optim.Adam(agent.joint_controller.parameters(), lr=3e-4)
    
    for episode in range(1000):
        state, _ = env.reset()
        episode_reward = 0
        
        # 收集经验
        for step in range(100):
            # 分层执行
            action = agent.execute_step(state)
            
            next_state, reward, done, _, _ = env.step(action)
            
            # 分层更新
            if step % 5 == 0:
                # 更新高级策略（稀疏奖励，更新频率低）
                if reward > 0:
                    high_loss = agent.update_high_level(state, reward)
                    optimizer_high.step()
            
            if step % 1 == 0:
                # 更新低级策略（密集奖励，更新频率高）
                low_loss = agent.update_low_level(state, action, reward, next_state)
                optimizer_low.step()
            
            episode_reward += reward
            state = next_state
            
            if done:
                break
        
        print(f"Episode {episode}: Reward = {episode_reward:.2f}")
    
    return agent
```

## 总结

分层强化学习是解决复杂长时程任务的重要范式。核心思想是**分而治之**——把复杂任务分解成多层嵌套的子问题。

关键要点：

1. **Options框架**是理论基础：option = 策略 + 终止条件
2. **FeUdal Networks**展示了两层分离的好处：manager设目标，worker执行
3. **HIRO**解决了off-policy训练的问题
4. **迁移学习**是分层RL的天然优势：高层更抽象，迁移性更好
5. **课程学习**可以加速训练：从简单任务开始

分层RL的挑战主要在**自动发现有用的子任务**和**层次之间的协调**。未来的研究方向包括元学习驱动的层次发现、终身学习框架等。
