---
title: 基于模型的RL与World Model
date: 2026-04-24
tags:
  - 强化学习
  - Model-based RL
  - World Model
  - Dreamer
  - MuZero
  - 模型预测控制
aliases:
  - 基于模型的强化学习
  - World Model详解
  - Dreamer系列算法
---

# 基于模型的RL与World Model

## Model-based RL vs Model-free RL

先说说这两种RL的根本区别。想象你在学打台球：

**Model-free RL**就像你每次击球后靠直觉判断"这球打得不错"或"这球偏了"，你不知道台球桌上的物理规律，但通过大量练习，你还是能学会怎么打。但代价是：**你可能需要打几千杆才能学会，而且每一杆的学习都很"盲目"。**

**Model-based RL**就像你在打之前先学了牛顿力学——你知道球会怎么滚动、碰到边缘会怎么反弹。虽然物理模型可能有误差，但有了这个模型，你可以**在脑子里预演**各种击球方案，选择最优的再动手。

用更技术的话说：

| 特性 | Model-free RL | Model-based RL |
|------|---------------|----------------|
| 环境模型 | 无 | 学一个$p(s_{t+1}|s_t, a_t)$ |
| 样本效率 | 低（需要大量交互） | 高（可以在模型中想象） |
| 规划能力 | 无 | 有（可以用模型预测未来） |
| 泛化能力 | 可能过拟合 | 可能受模型误差影响 |
| 调试难度 | 相对简单 | 模型误差难诊断 |

Model-based RL的核心优势是**样本效率高**。在真实环境中交互很贵（比如机器人控制），用学到的模型做"梦境训练"可以大大减少真实交互次数。

但Model-based RL也有自己的坑：**模型误差会累积**。预测下一步还好，预测10步以后可能就跑偏了——误差会像滚雪球一样越滚越大。

## Dreamer系列：世界模型+想象力规划

Dreamer是DeepMind在2020年提出的基于世界模型的强化学习方法，核心思想是：**学一个世界模型，在模型里训练策略，然后部署到真实环境。**

Dreamer的架构分三部分：

1. **World Model（世界模型）**：学习环境的压缩表示和动态预测
2. **Dreamer Actor-Critic**：在世界模型的"梦境"中训练策略
3. **Environment Interaction**：把训练好的策略部署到真实环境

```python
class Dreamer:
    def __init__(self, obs_dim, action_dim, hidden=512, 
                 deter_dim=200, stoch_dim=32):
        self.obs_dim = obs_dim
        self.action_dim = action_dim
        
        # ============= 世界模型 =============
        # Encoder: 观测 -> 潜在表示
        self.encoder = nn.Sequential(
            nn.Linear(obs_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, deter_dim + stoch_dim)
        )
        
        # Dynamics Model: 隐状态 + 动作 -> 下一隐状态
        self.rnn = nn.GRUCell(deter_dim + stoch_dim + action_dim, deter_dim)
        self.stoch_prior = nn.Sequential(
            nn.Linear(deter_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, stoch_dim * 2)  # mean, std
        )
        self.stoch_posterior = nn.Sequential(
            nn.Linear(deter_dim + obs_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, stoch_dim * 2)
        )
        
        # Decoder: 隐状态 -> 预测观测和奖励
        self.decoder = nn.Sequential(
            nn.Linear(deter_dim + stoch_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, obs_dim + 1)  # 观测 + 奖励
        )
        
        # ============= 策略和价值网络 =============
        self.actor = nn.Sequential(
            nn.Linear(deter_dim + stoch_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, action_dim * 2)
        )
        
        self.critic = nn.Sequential(
            nn.Linear(deter_dim + stoch_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, 1)
        )
        
        # 目标 critic（延迟更新）
        self.critic_target = copy.deepcopy(self.critic)
        
    def encode(self, obs):
        """编码观测到潜在表示"""
        return self.encoder(obs)
    
    def imagine(self, state, action, next_obs=None):
        """
        想象一步：给定当前隐状态和动作，预测下一步
        """
        # 分离确定性和随机部分
        deter = state[:, :200]
        stoch = state[:, 200:]
        
        # 送入RNN
        rnn_input = torch.cat([deter, stoch, action], dim=1)
        deter_next = self.rnn(rnn_input, deter)
        
        # 先验分布
        prior = self.stoch_prior(deter_next)
        prior_mean, prior_std = prior.chunk(2, dim=1)
        prior_std = torch.exp(prior_std)
        
        if next_obs is not None:
            # 有观测，用后验更新
            posterior = self.stoch_posterior(torch.cat([deter_next, next_obs], dim=1))
            posterior_mean, posterior_std = posterior.chunk(2, dim=1)
            posterior_std = torch.exp(posterior_std)
            stoch_next = posterior_mean + posterior_std * torch.randn_like(posterior_std)
        else:
            # 没观测，用先验
            stoch_next = prior_mean + prior_std * torch.randn_like(prior_std)
        
        # 重建观测和奖励
        latent = torch.cat([deter_next, stoch_next], dim=1)
        recon = self.decoder(latent)
        obs_pred, reward_pred = recon[:, :-1], recon[:, -1:]
        
        return latent, obs_pred, reward_pred
    
    def update_world_model(self, obs, actions, rewards):
        """
        更新世界模型：重构观测和奖励
        """
        batch_size = obs.shape[0]
        horizon = obs.shape[1]
        
        # 初始隐状态
        state = torch.zeros(batch_size, 232)  # deter + stoch
        
        total_recon_loss = 0
        total_kl_loss = 0
        
        for t in range(horizon - 1):
            action = actions[:, t]
            next_obs = obs[:, t + 1]
            
            # 想象一步
            latent, obs_pred, reward_pred = self.imagine(state, action, next_obs)
            
            # 重构损失
            recon_loss = F.mse_loss(obs_pred, next_obs) + \
                         F.mse_loss(reward_pred, rewards[:, t])
            
            # KL散度（正则化先验和后验）
            prior = self.stoch_prior(state[:, :200])
            prior_mean, prior_std = prior.chunk(2, dim=1)
            posterior = self.stoch_posterior(torch.cat([state[:, :200], next_obs], dim=1))
            posterior_mean, posterior_std = posterior.chunk(2, dim=1)
            
            kl_loss = torch.distributions.kl_divergence(
                torch.distributions.Normal(posterior_mean, torch.exp(posterior_std)),
                torch.distributions.Normal(prior_mean, torch.exp(prior_std))
            ).sum(dim=-1).mean()
            
            total_recon_loss += recon_loss
            total_kl_loss += kl_loss
            
            state = latent
        
        # 更新世界模型
        world_loss = total_recon_loss + 0.1 * total_kl_loss
        
        optimizer_world = optim.Adam(self.parameters(), lr=1e-4)
        optimizer_world.zero_grad()
        world_loss.backward()
        optimizer_world.step()
        
        return world_loss.item()
    
    def update_actor_critic(self, initial_states, imagination_horizon=15):
        """
        在梦境中更新策略和价值函数
        """
        # 从初始状态开始想象
        states = initial_states
        rewards_list = []
        values_list = []
        actions_list = []
        
        for _ in range(imagination_horizon):
            # 策略采样动作
            mean, std = self.actor(states).chunk(2, dim=1)
            std = torch.exp(std)
            dist = torch.distributions.Normal(mean, std)
            actions = dist.sample()
            log_probs = dist.log_prob(actions).sum(dim=-1)
            
            # 价值估计
            values = self.critic(states).squeeze()
            
            # 在模型中前进一步
            latent, _, rewards = self.imagine(states, actions)
            
            rewards_list.append(rewards)
            values_list.append(values)
            actions_list.append(actions)
            
            states = latent
        
        # 计算回报（GAE）
        returns = []
        R = 0
        for r in reversed(rewards_list):
            R = r + 0.99 * R
            returns.insert(0, R)
        
        returns = torch.cat(returns, dim=1)
        values = torch.cat(values_list, dim=1)
        
        # 优势
        advantages = returns - values.detach()
        
        # 策略损失（REINFORCE）
        log_probs = torch.cat([
            torch.distributions.Normal(*self.actor(states).chunk(2, dim=1)).log_prob(a).sum(dim=-1)
            for states, a in zip(initial_states, torch.cat(actions_list, dim=1))
        ])
        actor_loss = -(log_probs * advantages.mean()).mean()
        
        # 价值损失
        critic_loss = F.mse_loss(values, returns.detach())
        
        # 更新
        optimizer_ac = optim.Adam(
            list(self.actor.parameters()) + list(self.critic.parameters()),
            lr=3e-4
        )
        optimizer_ac.zero_grad()
        (actor_loss + 0.5 * critic_loss).backward()
        optimizer_ac.step()
        
        return actor_loss.item(), critic_loss.item()
```

Dreamer的核心洞察是：**把环境压缩到低维潜在空间，在这个空间里做规划就快多了。** 训练时，智能体可以在"梦境"里跑几千步，而不需要在真实环境里交互那么多次。

## 世界模型：VAE+RNN组合压缩环境规律

World Model是Dreamer之前的一个更简化的版本，出自David Ha和Jürgen Schmidhuber之手。它的核心是三个组件：

1. **Vision (V)**：用VAE把高维图像压缩成低维向量
2. **Memory (M)**：用RNN存储历史信息
3. **Controller (C)**：简单的线性控制器选择动作

```python
class WorldModel:
    """
    简化的World Model: VAE + RNN
    """
    
    def __init__(self, obs_channels=3, z_dim=32, hidden_dim=256):
        self.z_dim = z_dim
        
        # VAE encoder: 图像 -> z
        self.encoder = nn.Sequential(
            nn.Conv2d(obs_channels, 32, 4, stride=2),
            nn.ReLU(),
            nn.Conv2d(32, 64, 4, stride=2),
            nn.ReLU(),
            nn.Conv2d(64, 128, 4, stride=2),
            nn.ReLU(),
            nn.Flatten(),
            nn.Linear(128 * 26 * 26, hidden_dim),
            nn.ReLU()
        )
        
        self.fc_mean = nn.Linear(hidden_dim, z_dim)
        self.fc_std = nn.Linear(hidden_dim, z_dim)
        
        # VAE decoder: z -> 图像
        self.decoder = nn.Sequential(
            nn.Linear(z_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, 128 * 26 * 26),
            nn.ReLU(),
            nn.Unflatten(1, (128, 26, 26)),
            nn.ConvTranspose2d(128, 64, 4, stride=2),
            nn.ReLU(),
            nn.ConvTranspose2d(64, 32, 4, stride=2),
            nn.ReLU(),
            nn.ConvTranspose2d(32, obs_channels, 4, stride=2)
        )
        
        # RNN: 历史信息压缩
        self.rnn = nn.GRUCell(z_dim + action_dim, hidden_dim)
        
        # Controller: 简单的线性层
        self.controller = nn.Linear(hidden_dim, action_dim)
        
    def forward(self, obs, hidden=None):
        """前向传播"""
        # 编码观测
        h = self.encoder(obs)
        mean = self.fc_mean(h)
        log_std = self.fc_std(h)
        
        # 采样z
        z = mean + torch.exp(log_std) * torch.randn_like(mean)
        
        return z, hidden
    
    def dream(self, initial_hidden, action, num_steps):
        """
        在模型中做梦
        """
        hidden = initial_hidden
        predictions = []
        
        for _ in range(num_steps):
            # RNN更新
            rnn_input = torch.cat([action, torch.zeros_like(action)], dim=1)  # 简化：用零向量
            hidden = self.rnn(rnn_input, hidden)
            
            # 解码生成预测观测
            obs_pred = self.decoder(hidden[:, :self.z_dim])
            
            # 用预测更新
            next_z = hidden[:, :self.z_dim]
            next_action = self.controller(hidden)
            
            predictions.append({
                'obs': obs_pred,
                'action': next_action,
                'hidden': hidden
            })
            
            action = next_action
        
        return predictions
```

World Model的训练是分阶段的：
1. 先训练VAE，让它学会压缩图像
2. 固定VAE，训练RNN学习动态
3. 用CEM（交叉熵方法）在模型中优化控制器

## 梦境训练：在世界模型中训练策略

"梦境训练"（Dreaming）的好处是可以**并行、加速、而且不用担心真实环境的物理限制**。

```python
def dream_training(model, initial_obs, num_dreams=1000, horizon=50):
    """
    在梦境中训练策略
    """
    optimizer = optim.Adam(model.controller.parameters(), lr=1e-3)
    
    for dream in range(num_dreams):
        # 重置隐藏状态
        hidden = torch.zeros(1, model.hidden_dim)
        
        total_reward = 0
        
        for step in range(horizon):
            # 在模型中选择动作
            action = model.controller(hidden)
            
            # 模拟一步
            with torch.no_grad():
                obs_pred, reward_pred = model.imagine_step(hidden, action)
            
            # 计算奖励
            reward = reward_pred.item()
            total_reward += reward
            
            # 更新隐藏状态
            rnn_input = torch.cat([action, obs_pred], dim=1)
            hidden = model.rnn(rnn_input, hidden)
            
            # 在梦境中更新策略
            # 简化：用随机策略作为baseline
            if step > 0:
                advantage = reward - total_reward / (step + 1)
                action_loss = -advantage * action.mean()
                
                optimizer.zero_grad()
                action_loss.backward()
                optimizer.step()
        
        if dream % 100 == 0:
            print(f"Dream {dream}: Total Reward = {total_reward:.2f}")
```

但梦境训练有个致命问题：**模型误差会累积**。预测10步还凑合，预测100步可能就跑偏了。所以实践中：
- 梦境长度不能太长（通常10-20步）
- 需要定期用真实环境的数据校正模型
- 越接近真实的模型越有价值

## MuZero：不学习显式模型，但有规划能力

MuZero是DeepMind 2020年的工作，它不走寻常路：**不显式学习$p(s_{t+1}|s_t, a_t)$这个转移模型，而是学习一个隐式的"价值等效"模型。**

MuZero的核心思想是：**我不需要知道状态的具体表示是什么，只需要知道"这个状态的价值是多少"。** 换句话说，学一个从"状态+动作"到"预期价值"的映射。

```python
class MuZero:
    def __init__(self, obs_dim, action_dim, hidden=256, num_simulations=50):
        self.action_dim = action_dim
        self.num_simulations = num_simulations
        
        # 编码器：观测 -> 隐状态
        self.representation = nn.Sequential(
            nn.Linear(obs_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden)
        )
        
        # 动力学网络：隐状态 + 动作 -> 奖励 + 下一隐状态
        self.dynamics = nn.Sequential(
            nn.Linear(hidden + action_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU()
        )
        self.reward_head = nn.Linear(hidden, 1)
        self.transition_head = nn.Linear(hidden, hidden)
        
        # 预测网络：隐状态 -> 策略 + 价值
        self.prediction = nn.Sequential(
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU()
        )
        self.policy_head = nn.Linear(hidden, action_dim)
        self.value_head = nn.Linear(hidden, 1)
        
    def initial_inference(self, obs):
        """从观测初始化隐状态"""
        h = self.representation(obs)
        policy = F.softmax(self.policy_head(h), dim=-1)
        value = torch.tanh(self.value_head(h))
        return h, policy, value
    
    def recurrent_inference(self, state, action):
        """
        循环推理：给定隐状态和动作，预测奖励和下一隐状态
        """
        # one-hot动作
        action_onehot = F.one_hot(action, self.action_dim).float()
        
        # 动力学更新
        combined = torch.cat([state, action_onehot], dim=1)
        h_next = self.dynamics(combined)
        
        # 预测奖励
        reward = self.reward_head(h_next)
        
        # 转移（不需要显式建模）
        state_next = self.transition_head(h_next)
        
        # 预测策略和价值
        policy = F.softmax(self.policy_head(state_next), dim=-1)
        value = torch.tanh(self.value_head(state_next))
        
        return state_next, reward, policy, value
    
    def mcts_search(self, obs):
        """
        MCTS搜索：MuZero的核心
        """
        # 初始化根节点
        root_h, root_policy, root_value = self.initial_inference(obs)
        
        # 构建搜索树
        root = MCTSNode(
            hidden=root_h,
            policy=root_policy,
            value=root_value,
            reward=0,
            parent=None
        )
        
        for _ in range(self.num_simulations):
            node = root
            path = [node]
            
            # 选择：UCB引导
            while not node.is_leaf():
                action = node.select()
                next_state, reward, policy, value = self.recurrent_inference(
                    node.hidden, action
                )
                
                if next_state not in node.children:
                    # 创建新节点
                    child = MCTSNode(
                        hidden=next_state,
                        policy=policy,
                        value=value,
                        reward=reward,
                        parent=node
                    )
                    node.children[action] = child
                else:
                    child = node.children[action]
                
                node = child
                path.append(node)
            
            # 扩展：如果没有游戏结束，继续扩展
            if not node.is_terminal():
                action = node.select()  # 用UCB选动作
                next_state, reward, policy, value = self.recurrent_inference(
                    node.hidden, action
                )
                
                child = MCTSNode(
                    hidden=next_state,
                    policy=policy,
                    value=value,
                    reward=reward,
                    parent=node
                )
                node.children[action] = child
                path.append(child)
            
            # 回溯：更新Q值
            bootstrap_value = 0
            for node in reversed(path):
                bootstrap_value = node.reward + 0.99 * bootstrap_value
                node.visit_count += 1
                node.value_sum += bootstrap_value
        
        # 返回根节点的访问分布作为策略
        pi = root.get_visit_counts()
        return pi, root_h
    
    def update(self, batch):
        """
        训练更新
        """
        obs_batch, actions_batch, rewards_batch, policy_batch, value_batch = batch
        
        total_loss = 0
        
        for obs, actions, rewards, policy_target, value_target in zip(
            obs_batch, actions_batch, rewards_batch, policy_batch, value_batch
        ):
            # 初始推理
            h, policy_pred, value_pred = self.initial_inference(obs)
            
            losses = []
            
            # 重构奖励和价值
            reward_loss = F.cross_entropy(policy_pred, torch.tensor(policy_target))
            value_loss = F.mse_loss(value_pred, torch.tensor(value_target))
            losses.extend([reward_loss, value_loss])
            
            # 循环推理
            state = h
            for t, action in enumerate(actions):
                state, reward_pred, policy_pred, value_pred = self.recurrent_inference(
                    state, action
                )
                
                reward_loss = F.mse_loss(reward_pred.squeeze(), torch.tensor(rewards[t]))
                value_loss = F.mse_loss(value_pred.squeeze(), torch.tensor(value_target))
                
                losses.extend([reward_loss, value_loss])
            
            # 总损失
            loss = sum(losses) / len(losses)
            total_loss += loss
        
        optimizer = optim.Adam(self.parameters(), lr=1e-4)
        optimizer.zero_grad()
        total_loss.backward()
        optimizer.step()
        
        return total_loss.item()
```

MuZero的美妙之处在于：**它既不需要显式建模环境，又具备规划能力。** 通过MCTS搜索，它可以在隐状态空间中"想清楚"未来几步会发生什么，从而做出更好的决策。

## PETS：基于模型的规划

PETS（Probabilistic Ensemble Task-Aware）是用模型做规划的另一个代表。它用**集成的方法**来应对模型不确定性，同时用**交叉熵方法（CEM）**来做规划。

```python
class PETS:
    """
    PETS: Probabilistic Ensemble Task-Aware Planning
    """
    
    def __init__(self, state_dim, action_dim, ensemble_size=5, horizon=10):
        self.ensemble_size = ensemble_size
        self.horizon = horizon
        
        # 集成模型：每个模型预测 mean 和 variance
        self.models = nn.ModuleList([
            EnsembleDynamics(state_dim, action_dim) 
            for _ in range(ensemble_size)
        ])
        
        self.action_dim = action_dim
        self.action_mean = None
        self.action_std = None
    
    def predict(self, state, action, model_idx):
        """
        单个模型的预测
        """
        x = torch.cat([state, action], dim=1)
        output = self.models[model_idx](x)
        
        mean = output[:, :state_dim]
        std = torch.exp(output[:, state_dim:])
        
        return mean, std
    
    def cem_planning(self, state, num_samples=1000, num_elites=50, num_iterations=5):
        """
        交叉熵方法做MPC规划
        """
        # 初始化动作分布
        if self.action_mean is None:
            self.action_mean = torch.zeros(self.horizon, self.action_dim)
            self.action_std = torch.ones(self.horizon, self.action_dim)
        
        for iteration in range(num_iterations):
            # 采样动作序列
            actions = self.action_mean + self.action_std * torch.randn(
                num_samples, self.horizon, self.action_dim
            )
            actions = torch.clamp(actions, -1, 1)
            
            # 对每个样本 rollout
            returns = []
            for i in range(num_samples):
                sim_state = state.clone()
                total_reward = 0
                
                for t in range(self.horizon):
                    # 用集成模型预测
                    model_idx = random.randint(0, self.ensemble_size - 1)
                    mean, std = self.predict(sim_state, actions[i, t], model_idx)
                    
                    # 采样下一状态
                    next_state = mean + std * torch.randn_like(mean)
                    
                    # 计算奖励
                    reward = self.compute_reward(sim_state, actions[i, t])
                    total_reward += reward
                    
                    sim_state = next_state
                
                returns.append(total_reward)
            
            returns = torch.tensor(returns)
            
            # 选择精英样本
            elite_indices = returns.topk(num_elites).indices
            elite_actions = actions[elite_indices]
            
            # 更新动作分布
            self.action_mean = elite_actions.mean(dim=0)
            self.action_std = elite_actions.std(dim=0) + 1e-6
        
        # 返回第一个动作
        return self.action_mean[0].numpy()
    
    def compute_reward(self, state, action):
        """计算奖励（任务相关）"""
        # 简化：假设奖励是到达目标的能力
        return -((state - self.target_state) ** 2).sum() - 0.01 * (action ** 2).sum()
    
    def update(self, replay_buffer, batch_size=256):
        """
        更新动态模型
        """
        batch = replay_buffer.sample(batch_size)
        states, actions, next_states = batch
        
        # 用所有模型训练
        for model in self.models:
            optimizer = optim.Adam(model.parameters(), lr=1e-3)
            
            # 预测
            x = torch.cat([states, actions], dim=1)
            output = model(x)
            mean_pred = output[:, :states.shape[1]]
            std_pred = torch.exp(output[:, states.shape[1]:])
            
            # 高斯负对数似然
            loss = -torch.distributions.Normal(mean_pred, std_pred).log_prob(next_states).mean()
            
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
```

PETS的核心流程：
1. **收集数据**：用随机策略或当前策略在环境中收集数据
2. **训练模型**：用集成高斯过程建模动态
3. **CEM规划**：用交叉熵方法优化动作序列
4. **执行第一个动作**：执行规划的第一个动作，然后把真实观测送回规划器

## MPC：用CEM做滚动优化

MPC（Model Predictive Control）是经典的控制理论方法，在RL里被借鉴来做基于模型的规划。核心思想是**滚动优化**——每次只执行规划的第一个动作，然后重新规划。

```python
class MPC:
    """
    简化的MPC控制器
    """
    
    def __init__(self, dynamics_model, reward_fn, horizon=20, 
                 num_samples=1000, num_elites=50):
        self.dynamics = dynamics_model
        self.reward_fn = reward_fn
        self.horizon = horizon
        self.num_samples = num_samples
        self.num_elites = num_elites
        
        # CEM参数
        self.action_mean = None
        self.action_std = None
        
    def plan(self, state, num_iterations=5):
        """
        用CEM规划动作序列
        """
        if self.action_mean is None:
            self.action_mean = torch.zeros(self.horizon, self.dynamics.action_dim)
            self.action_std = torch.ones(self.horizon, self.dynamics.action_dim)
        
        for _ in range(num_iterations):
            # 采样
            actions = self.action_mean + self.action_std * torch.randn(
                self.num_samples, self.horizon, self.dynamics.action_dim
            )
            actions = torch.tanh(actions)  # 限制在[-1, 1]
            
            # Rollout计算回报
            returns = self.rollout(state, actions)
            
            # 选择精英
            elite_values, elite_indices = returns.topk(self.num_elites, largest=True)
            elite_actions = actions[elite_indices]
            
            # 更新CEM分布
            self.action_mean = elite_actions.mean(dim=0)
            self.action_std = elite_actions.std(dim=0) + 1e-3
        
        return self.action_mean[0].numpy()
    
    def rollout(self, state, actions):
        """
        对采样的动作序列做rollout
        """
        returns = torch.zeros(actions.shape[0])
        current_state = state
        
        for t in range(self.horizon):
            # 预测下一步
            next_states = []
            for i in range(actions.shape[0]):
                next_state = self.dynamics.predict(current_state, actions[i, t])
                next_states.append(next_state)
            
            next_states = torch.stack(next_states)
            
            # 计算奖励
            rewards = self.reward_fn(current_state, actions[:, t])
            returns += rewards
            
            current_state = next_states
        
        return returns
    
    def step(self, state):
        """
        MPC控制器的单步
        """
        action = self.plan(state)
        return action
```

## 代码实战：简化的DreamerV1

下面实现一个简化但完整的DreamerV1版本：

```python
import gymnasium as gym
import numpy as np
import torch
import torch.nn as nn
import torch.nn.functional as F
import copy

class WorldModel(nn.Module):
    """
    DreamerV1 的世界模型
    """
    def __init__(self, obs_dim, action_dim, hidden=400, 
                 deter_dim=200, stoch_dim=32):
        super().__init__()
        self.deter_dim = deter_dim
        self.stoch_dim = stoch_dim
        self.hidden = hidden
        
        # Encoder
        self.encoder = nn.Sequential(
            nn.Linear(obs_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, hidden),
            nn.ELU(),
            nn.Linear(hidden, deter_dim + stoch_dim)
        )
        
        # Dynamics (RNN)
        self.rnn = nn.GRUCell(deter_dim + stoch_dim + action_dim, deter_dim)
        
        # Prior (先验分布)
        self.prior_net = nn.Sequential(
            nn.Linear(deter_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, stoch_dim * 2)
        )
        
        # Posterior (后验分布)
        self.posterior_net = nn.Sequential(
            nn.Linear(deter_dim + obs_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, stoch_dim * 2)
        )
        
        # Decoder
        self.decoder = nn.Sequential(
            nn.Linear(deter_dim + stoch_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, hidden),
            nn.ELU(),
            nn.Linear(hidden, obs_dim + 1)  # obs reconstruction + reward
        )
        
        # 隐状态
        self.stoch = None
        self.deter = None
        
    def init_hidden(self, batch_size):
        self.stoch = torch.zeros(batch_size, self.stoch_dim)
        self.deter = torch.zeros(batch_size, self.deter_dim)
    
    def encode(self, obs):
        """编码观测"""
        feat = self.encoder(obs)
        self.stoch = feat[:, :self.stoch_dim]
        self.deter = feat[:, self.stoch_dim:]
        return self.get_latent()
    
    def get_latent(self):
        return torch.cat([self.deter, self.stoch], dim=1)
    
    def forward(self, action, obs=None):
        """
        一步前向
        """
        # 先验
        prior_mean_std = self.prior_net(self.deter)
        prior_mean = prior_mean_std[:, :self.stoch_dim]
        prior_std = torch.exp(prior_mean_std[:, self.stoch_dim:])
        
        if obs is not None:
            # 有观测，用后验
            posterior_mean_std = self.posterior_net(
                torch.cat([self.deter, obs], dim=1)
            )
            posterior_mean = posterior_mean_std[:, :self.stoch_dim]
            posterior_std = torch.exp(posterior_mean_std[:, self.stoch_dim:])
            
            # 采样
            self.stoch = posterior_mean + posterior_std * torch.randn_like(posterior_std)
        else:
            # 没观测，用先验
            self.stoch = prior_mean + prior_std * torch.randn_like(prior_std)
        
        # RNN更新
        rnn_input = torch.cat([self.deter, self.stoch, action], dim=1)
        self.deter = self.rnn(rnn_input, self.deter)
        
        return self.get_latent()
    
    def decode(self):
        """解码"""
        latent = self.get_latent()
        output = self.decoder(latent)
        obs_pred = output[:, :-1]
        reward_pred = output[:, -1:]
        return obs_pred, reward_pred

class Actor(nn.Module):
    """Dreamer的策略网络"""
    def __init__(self, latent_dim, action_dim, hidden=400):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(latent_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, hidden),
            nn.ELU(),
            nn.Linear(hidden, action_dim * 2)
        )
    
    def forward(self, latent):
        mean, log_std = self.network(latent).chunk(2, dim=-1)
        log_std = torch.clamp(log_std, -20, 2)
        return mean, log_std
    
    def get_action(self, latent, deterministic=False):
        mean, log_std = self.forward(latent)
        if deterministic:
            return torch.tanh(mean)
        std = torch.exp(log_std)
        dist = torch.distributions.Normal(mean, std)
        action = torch.tanh(dist.sample())
        return action

class Critic(nn.Module):
    """Dreamer的价值网络"""
    def __init__(self, latent_dim, hidden=400):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(latent_dim, hidden),
            nn.ELU(),
            nn.Linear(hidden, hidden),
            nn.ELU(),
            nn.Linear(hidden, 1)
        )
    
    def forward(self, latent):
        return self.network(latent)

class DreamerAgent:
    def __init__(self, obs_dim, action_dim, lr=3e-4):
        self.world_model = WorldModel(obs_dim, action_dim)
        self.actor = Actor(232, action_dim)  # deter + stoch
        self.critic = Critic(232)
        self.critic_target = Critic(232)
        self.critic_target.load_state_dict(self.critic.state_dict())
        
        self.obs_dim = obs_dim
        self.action_dim = action_dim
        
        # 优化器
        self.opt_world = optim.Adam(self.world_model.parameters(), lr=lr)
        self.opt_actor = optim.Adam(self.actor.parameters(), lr=lr)
        self.opt_critic = optim.Adam(self.critic.parameters(), lr=lr)
    
    def act(self, obs, deterministic=False):
        obs_t = torch.FloatTensor(obs).unsqueeze(0)
        with torch.no_grad():
            self.world_model.init_hidden(1)
            self.world_model.encode(obs_t)
            latent = self.world_model.get_latent()
            action = self.actor.get_action(latent, deterministic).squeeze().numpy()
        return action
    
    def update(self, batch, horizon=15, lambda_=0.95):
        states, actions, rewards, dones = batch
        
        # ============= 更新世界模型 =============
        self.world_model.init_hidden(len(states))
        
        recon_losses = []
        kl_losses = []
        
        for t in range(len(states) - 1):
            obs_t = torch.FloatTensor(states[t]).unsqueeze(0)
            action_t = torch.FloatTensor(actions[t]).unsqueeze(0)
            obs_next = torch.FloatTensor(states[t+1]).unsqueeze(0)
            
            # 想象一步
            latent = self.world_model.forward(action_t, obs_next)
            obs_pred, reward_pred = self.world_model.decode()
            
            # 重构损失
            recon_loss = F.mse_loss(obs_pred, obs_t) + \
                        F.mse_loss(reward_pred, torch.FloatTensor([[rewards[t]]]))
            recon_losses.append(recon_loss)
            
            # KL损失
            prior = self.world_model.prior_net(self.world_model.deter)
            posterior = self.world_model.posterior_net(
                torch.cat([self.world_model.deter, obs_t], dim=1)
            )
            kl = torch.distributions.kl_divergence(
                torch.distributions.Normal(posterior[:, :16], torch.exp(posterior[:, 16:])),
                torch.distributions.Normal(prior[:, :16], torch.exp(prior[:, 16:]))
            ).sum(dim=-1).mean()
            kl_losses.append(kl)
        
        world_loss = sum(recon_losses) + 0.1 * sum(kl_losses)
        
        self.opt_world.zero_grad()
        world_loss.backward()
        self.opt_world.step()
        
        # ============= 在梦境中更新Actor和Critic =============
        # 从初始状态开始想象
        self.world_model.init_hidden(len(states))
        self.world_model.encode(torch.FloatTensor(states[0]))
        
        imag_states = []
        imag_rewards = []
        imag_actions = []
        
        for h in range(horizon):
            latent = self.world_model.get_latent()
            action = self.actor.get_action(latent.detach())
            
            imag_states.append(latent)
            imag_actions.append(action)
            
            self.world_model.forward(action)
            _, reward = self.world_model.decode()
            imag_rewards.append(reward.squeeze())
        
        imag_states = torch.stack(imag_states)
        imag_actions = torch.stack(imag_actions)
        imag_rewards = torch.stack(imag_rewards)
        
        # 计算回报
        returns = []
        R = 0
        for r in reversed(imag_rewards):
            R = r + 0.99 * lambda_ * R
            returns.insert(0, R)
        returns = torch.stack(returns).squeeze()
        
        # Critic更新
        values = self.critic(imag_states.detach()).squeeze()
        critic_loss = F.mse_loss(values, returns.detach())
        
        self.opt_critic.zero_grad()
        critic_loss.backward()
        self.opt_critic.step()
        
        # Actor更新 (REINFORCE)
        values = self.critic(imag_states).squeeze()
        advantages = returns - values.detach()
        
        mean, log_std = self.actor(imag_states)
        std = torch.exp(log_std)
        dist = torch.distributions.Normal(mean, std)
        log_probs = dist.log_prob(imag_actions).sum(dim=-1)
        
        actor_loss = -(log_probs * advantages.detach()).mean()
        
        self.opt_actor.zero_grad()
        actor_loss.backward()
        self.opt_actor.step()
        
        return world_loss.item(), actor_loss.item(), critic_loss.item()

def train_dreamer(env_name='Pendulum-v1', num_episodes=500):
    env = gym.make(env_name)
    
    obs_dim = env.observation_space.shape[0]
    action_dim = env.action_space.shape[0]
    
    agent = DreamerAgent(obs_dim, action_dim)
    
    episode_rewards = []
    
    for episode in range(num_episodes):
        state, _ = env.reset()
        episode_reward = 0
        trajectory = {'states': [], 'actions': [], 'rewards': []}
        
        for step in range(200):
            action = agent.act(state)
            action_clipped = np.clip(action, -2, 2)
            
            next_state, reward, done, _, _ = env.step(action_clipped)
            
            trajectory['states'].append(state)
            trajectory['actions'].append(action)
            trajectory['rewards'].append(reward)
            
            episode_reward += reward
            state = next_state
            
            if done:
                break
        
        # 更新
        batch = (
            trajectory['states'],
            trajectory['actions'],
            trajectory['rewards'],
            [False] * len(trajectory['states'])
        )
        
        if len(trajectory['states']) > 10:
            world_loss, actor_loss, critic_loss = agent.update(batch)
        
        episode_rewards.append(episode_reward)
        
        if episode % 10 == 0:
            avg = np.mean(episode_rewards[-10:])
            print(f"Episode {episode}: Avg Reward = {avg:.1f}")
    
    return agent

if __name__ == '__main__':
    print("Training Dreamer agent...")
    agent = train_dreamer('Pendulum-v1', num_episodes=300)
```

## 实战经验：模型误差累积问题

这是model-based RL里最让人头疼的问题。说几个亲测有效的应对方法：

### 1. 用集成模型量化不确定性

单一模型的预测即使均值准确，也不代表它知道"自己不知道什么"。集成模型可以同时给出均值和方差：

```python
class EnsembleDynamics:
    def __init__(self, state_dim, action_dim, num_models=5):
        self.models = nn.ModuleList([
            SingleDynamics(state_dim, action_dim)
            for _ in range(num_models)
        ])
    
    def forward(self, state, action, return_std=True):
        # 每个模型独立预测
        predictions = []
        for model in self.models:
            pred = model(state, action)
            predictions.append(pred)
        
        predictions = torch.stack(predictions)
        
        if return_std:
            mean = predictions.mean(dim=0)
            std = predictions.std(dim=0)
            return mean, std
        return predictions.mean(dim=0)
    
    def uncertainty(self, state, action):
        """不确定性估计"""
        _, std = self.forward(state, action)
        return std.mean(dim=-1)  # 不确定性 = 平均std
```

### 2. 不确定性驱动的探索

在模型不确定性大的区域多探索，收集更多数据改善模型：

```python
def uncertainty_based_exploration(state, action, ensemble):
    """基于不确定性的探索策略"""
    uncertainty = ensemble.uncertainty(state, action)
    
    # 高不确定性区域给额外奖励（收集数据）
    bonus = uncertainty * 0.1
    
    return bonus
```

### 3. 短视野 + 频繁重规划

把规划视野缩短（比如5步而不是50步），频繁用真实观测重置模型状态：

```python
class ShortHorizonMPC:
    def __init__(self, model, horizon=5):
        self.horizon = horizon
        self.model = model
    
    def reset(self, real_state):
        """用真实观测重置模型"""
        self.model.reset(real_state)
    
    def step(self, state):
        """短视野规划"""
        for _ in range(self.horizon):
            action = self.model.plan(state)
            next_state = self.model.predict(state, action)
            
            # 定期用真实观测校正
            if np.random.random() < 0.2:
                self.model.update(next_state)
            
            state = next_state
        
        return self.get_first_action()
```

### 4. 误差截断

防止误差无限累积：

```python
def bounded_predict(model, state, action, max_deviation=1.0):
    """有界预测，防止误差爆炸"""
    mean, std = model.predict(state, action)
    
    # 限制预测范围
    deviation = torch.abs(mean - state)
    if deviation.max() > max_deviation:
        # 裁剪偏差
        mean = state + (mean - state) * max_deviation / deviation.max()
    
    return mean + std * torch.randn_like(std)
```

## 总结

Model-based RL是强化学习的"高阶玩法"。核心优势是**样本效率高**——可以在模型里"脑补"无数次。

关键方法：
1. **Dreamer系列**：把环境压缩到隐空间，在隐空间里训练策略
2. **World Model**：VAE + RNN的经典组合
3. **MuZero**：不显式建模但有规划能力，用MCTS搜索
4. **PETS/CEM**：用概率模型 + 交叉熵方法做MPC

实战经验：
- 模型误差是主要敌人，用集成模型+不确定性估计来对抗
- 规划视野不要太长，频繁用真实观测校正
- 必要时做误差截断

如果你的任务样本采集很贵（比如机器人控制），model-based RL值得投入。刚开始可能觉得比model-free麻烦，但一旦模型学准了，训练速度会快好几个量级。
