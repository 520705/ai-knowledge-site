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

> [!abstract]+ 一句话理解MDP
> **想象你在玩一盘不知道规则的棋类游戏，每走一步系统会给你一个"感觉还行"或"好像不太对"的反馈，你需要通过不断尝试来推断规则并找到最佳走法——这就是MDP要数学化描述的问题。**

## 零、前言：为什么要学MDP？

很多同学学强化学习，上来就砸Q学习、DQN、PPO，结果发现公式推着推着就蒙了，根本原因是没有把MDP这个地基打好。

MDP是什么？它是**所有强化学习算法的通用语言**。不管你用Q学习还是策略梯度，不管你训练的是打游戏AI还是机器人控制器，底层都是在一个MDP里找最优策略。

所以别急，先把这章吃透，后面学算法会事半功倍。

---

## 一、盲人走路：什么是马尔可夫性？

### 1.1 场景设定

想象你是一个盲人，站在一个陌生的房间里。地上有地毯、瓷砖、木地板混在一起。你的任务是走到房间角落的一扇门。

作为盲人，你只能靠脚底的感觉来判断自己在哪、地面是什么材质。你还带着一根盲杖，每走一步可以探查前方一小块区域。

好，现在问你一个问题：**要决定下一步往哪走，你应该记住过去多少信息？**

### 1.2 三种"记忆"策略

**策略一：记住全部历史**

你决定记住从出发到现在走过的每一步：先往左两步、往前三步、往右一步……然后分析这些历史轨迹，试图推断房间的布局。

听起来很谨慎对吧？但问题是，你的大脑很快就会被这些细节填满，而且这些信息大部分都是冗余的——你真正需要知道的只有：**我现在在哪块地面上？**

**策略二：只记住当前位置**

你换了个思路：既然我每一步都能感知当前脚下的材质，那干嘛不直接根据这个信息做决策？过去走了几步、拐了几个弯，和我现在该往哪走，有直接关系吗？

这个"只根据当前状态做决策"的思路，就是**马尔可夫性的核心**。

**策略三：马尔可夫式的"刚刚好"**

马尔可夫性的精髓在于：**过去的信息都已经压缩进了当前状态里**。

还是那个盲人的例子。你现在踩在瓷砖上——这个"瓷砖"本身就是过去所有经历的浓缩：你从哪个方向走来、中间绕了什么弯，都不重要了。重要的是你现在站在瓷砖上，而瓷砖告诉你：往右上角走的方向是瓷砖和地毯的边界，再往前两步就是门。

> [!tip] 关键洞察
> 马尔可夫性不是说"过去完全不重要"，而是说**过去的影响都已经体现在当前状态里了**。就像你查地图找路时，不需要知道前几任房主是谁，只需要知道你现在站在哪条街。

### 1.3 马尔可夫性的数学表达

把上面的直觉翻译成数学语言：

$$
P(S_{t+1} | S_t) = P(S_{t+1} | S_1, S_2, ..., S_t)
$$

翻译成人话：**给定当前状态 $S_t$，未来状态 $S_{t+1}$ 的概率分布，和过去所有状态 $S_1, S_2, ..., S_{t-1}$ 无关。**

换句话说，$S_t$ 已经包含了所有决定未来的信息。就像你知道现在是几点几分，就不需要再问"一小时前是几点"——因为一小时前是几点已经决定了现在是几点。

### 1.4 为什么马尔可夫性是个好假设？

你可能会反驳：现实世界明明是连续的，过去怎么可能完全不影响未来？

这其实是个好问题。马尔可夫性是一个**理想化的假设**，它的价值在于：

1. **简化计算**：如果每次决策都要考虑完整历史，状态空间会爆炸
2. **实践中够用**：很多情况下，把历史压缩到"当前状态"里确实够用
3. **理论基础扎实**：有了马尔可夫性，我们才能用动态规划那套工具

真实世界的问题往往不是严格的马尔可夫过程，但通过精心地设计状态表示（比如用LSTM记忆历史信息），我们可以让非马尔可夫问题"看起来像"马尔可夫问题。

---

## 二、MDP四元组的生活类比

### 2.1 把MDP想象成一场寻宝游戏

假设你和几个朋友在一个游乐场里玩寻宝游戏。你们有一张地图，但地图上只标注了大致的区域划分，具体的路线和机关你们不知道。任务是找到藏在迷宫深处的宝藏。

这场寻宝游戏完美对应了MDP的四个要素：

| MDP要素 | 寻宝游戏中的对应物 | 通俗解释 |
|:-------|:-----------------|:-------|
| **状态 S** | 你当前站在哪个位置、周围是什么环境 | 你在哪 |
| **动作 A** | 往前走、往左拐、跳过去、爬上去 | 你能做什么 |
| **转移 P** | 每做一个动作，可能到达不同位置（有概率） | 做了之后会怎样 |
| **奖励 R** | 每走一步，感觉是离宝藏更近了还是更远了 | 做得好不好 |

### 2.2 状态：你在哪

状态 $s \in \mathcal{S}$ 是智能体对环境的"快照"。

好的状态设计应该像一张**精确的地图**——它应该告诉你所有做决策需要的信息，但又不包含无关的噪音。

举几个例子：

- **走迷宫**：状态就是每个格子的坐标 $(x, y)$
- **围棋**：状态就是整个棋盘的局面
- **机器人控制**：状态可能是关节角度、速度、末端位置
- **自动驾驶**：状态可能是前方路况、自身速度、与其他车辆的距离

> [!note]+ 状态设计的坑
> 如果状态设计得太粗糙（漏掉了重要信息），智能体就算再聪明也学不到好策略。如果状态设计得太复杂（包含了太多无关信息），智能体会被淹没在噪音里，学起来巨慢。

### 2.3 动作：你能做什么

动作 $a \in \mathcal{A}$ 是智能体可以采取的行为。

动作空间分两种：

**离散动作空间**：动作是可以枚举的选项
- 超级玛丽：上、下、左、右、跳
- 迷宫：北、南、东、西

**连续动作空间**：动作是一个连续的数值
- 机械臂：关节角度可以是0到360度之间的任意值
- 汽车：方向盘转角可以是-90度到+90度之间的任意值

### 2.4 转移：做了之后会怎样

转移概率 $P(s'|s, a)$ 描述的是：**在状态 $s$ 下做动作 $a$，有多少概率会到达状态 $s'$**。

这里有个微妙的地方：MDP假设转移是**随机的**，不是确定的。

这听起来反直觉——我按下开关灯不就应该亮吗？

但仔细想想，现实世界确实充满随机性：
- 你推门，有时候门会卡住
- 你投篮，有时候进有时候不进
- 你发微信消息，服务器有时候快有时候慢

MDP用概率来建模这种不确定性。对于确定性的环境（比如确定性的游戏），转移概率就是0或1；对于随机环境，转移概率可能是0.7、0.2、0.1这样的分布。

### 2.5 奖励：做得好不好

奖励函数 $R(s, a, s')$ 给出的是：**做完动作到达新状态后，得了多少分**。

奖励的设计决定了"什么才算好"：

- **走迷宫**：每走一步-1分，到达目标+100分
- **围棋**：最终赢了+1分，输了-1分（这个奖励是延迟的）
- **机器人抓取**：成功抓起物体+10分，每次碰撞-1分

> [!warning] 奖励设计的坑
> 设计奖励函数是强化学习中最 tricky 的事情之一。设计得不好，智能体会找到你意想不到的"作弊"方法。比如你让机器人移动速度越快奖励越高，它可能会直接冲下楼梯摔坏。

---

## 三、用MDP建模实际问题

学会了MDP的四元组，现在来看怎么把实际问题建模成MDP。

### 3.1 走迷宫

迷宫大概是MDP最经典的应用场景了。

```
# 迷宫示意图
# S = 起点, G = 目标, # = 墙, . = 通道
. . . # .
S # . . .
. . . # .
. . . . .
. . # # G
```

**建模过程**：

- **状态空间**：每个可达格子是一个状态，共 $(5 \times 5 - \text{墙的数量})$ 个状态
- **动作空间**：上、下、左、右（4个动作）
- **转移概率**：
  - 往某个方向走，如果前面是墙，留在原地（100%概率）
  - 往某个方向走，如果前面是通道，到达目标格子（100%概率）
  - 如果考虑"滑倒"等意外，可能是80%到达目标，20%滑到旁边
- **奖励设计**：
  - 每走一步-1（鼓励快速找到出路）
  - 到达目标+100
  - 撞墙-10（可选）

### 3.2 机器人控制

让机器人学会抓取桌上的物体。

**建模过程**：

- **状态空间**：$(x, y, z)$ 机械臂末端位置 + 物体位置 + 是否接触
- **动作空间**：在三维空间中的位移 $(\Delta x, \Delta y, \Delta z)$，可以离散化或连续
- **转移概率**：机器人的运动有误差，执行动作A可能到达预期位置，也可能有偏差
- **奖励设计**：
  - 拿起物体+10
  - 物体掉落-5
  - 碰撞桌面-1
  - 能耗（动作幅度越大负奖励越多）

### 3.3 金融交易

用MDP建模股票交易也很有意思。

**建模过程**：

- **状态空间**：当前价格、技术指标（MA、RSI等）、持仓状态、账户余额
- **动作空间**：买入、卖出、持有
- **转移概率**：从历史数据估计价格变动的概率分布
- **奖励设计**：每次交易的盈亏

> [!tip]+ MDP建模 checklist
> 把实际问题变成MDP， checklist 如下：
> 1. **状态是什么？** 完整描述当前局面需要哪些信息？
> 2. **动作是什么？** 智能体能做哪些操作？
> 3. **转移是什么？** 做动作后环境会怎么变化？概率分布是什么？
> 4. **奖励是什么？** 怎么量化"做得好"？

---

## 四、策略：智能体的大脑

### 4.1 策略是什么？

策略 $\pi$ 是智能体的"决策手册"——**给定状态，返回动作**。

数学上，策略有两种形式：

**确定性策略**：$\pi(s) = a$

就像一个古板的老教授，每个问题只给一个标准答案。

**随机性策略**：$\pi(a|s) = P(a|s)$

就像一个睿智的商人，面对同样的情况有时候这样做，有时候那样做，取决于他对概率的把控。

### 4.2 为什么需要随机性策略？

你可能会想：既然要优化，为什么不直接学一个确定性策略？

因为**探索是强化学习的核心挑战**。

想象你是个婴儿，第一次探索世界：
- 如果你每次都做同样的动作，你永远不知道"如果我做了别的选择会怎样"
- 适度的随机性让你能探索更多的可能性

随机策略 $\pi(a|s)$ 本质上是在"利用已知最优"和"探索未知选项"之间做平衡。

### 4.3 从策略到轨迹

有了策略，智能体就能和环境交互，产生一连串的"状态-动作-奖励"序列：

$$
s_0 \xrightarrow{a_0} r_0, s_1 \xrightarrow{a_1} r_1, s_2 \xrightarrow{a_2} r_2, \cdots
$$

这叫做**轨迹（trajectory）**或**episode**。

强化学习的任务，就是找到一个策略 $\pi^*$，使得按照这个策略产生的轨迹，整体回报最大。

---

## 五、价值函数：V(s)和Q(s,a)分别回答什么问题？

### 5.1 先理解"折扣回报"

智能体的目标是最大化**累积折扣回报**：

$$
G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
$$

这里的 $\gamma$ 是**折扣因子**，通常取0.9到0.99之间。

折扣因子有两层含义：

1. **时间偏好**：明天的100块钱不如今天的100块钱值钱，所以乘以 $\gamma$ 打折扣
2. **数学便利**：没有折扣，无限 horizon 的回报可能不收敛

### 5.2 V(s)：这个位置值多少钱？

状态价值函数 $V^\pi(s)$ 的定义：

$$
V^\pi(s) = \mathbb{E}_\pi [G_t | S_t = s]
$$

翻译：**如果你现在站在状态 $s$，之后一直乖乖按照策略 $\pi$ 行事，平均能拿多少回报？**

类比：想象你在下棋，$V^\pi(s)$ 就像是"这个局面值多少分"——一个好的局面分高，一个坏的局面分低。

### 5.3 Q(s,a)：这个动作值多少钱？

动作价值函数 $Q^\pi(s,a)$ 的定义：

$$
Q^\pi(s,a) = \mathbb{E}_\pi [G_t | S_t = s, A_t = a]
$$

翻译：**如果你现在站在状态 $s$，第一步先做动作 $a$，之后乖乖按照策略 $\pi$ 行事，平均能拿多少回报？**

类比：$Q^\pi(s,a)$ 像是"在这个局面下，这个动作值多少分"。

### 5.4 V和Q的关系

$V$ 和 $Q$ 之间有这样的关系：

$$
V^\pi(s) = \sum_a \pi(a|s) \cdot Q^\pi(s,a)
$$

翻译：**一个局面的分数 = 所有可能动作的加权平均**。权重就是在该状态下选择各动作的概率。

反过来：

$$
Q^\pi(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]
$$

翻译：**这个动作的分数 = 立即获得的奖励 + 打折后的未来收益**。

### 5.5 什么时候用V，什么时候用Q？

| 场景 | 用V还是用Q | 原因 |
|:-----|:----------|:-----|
| 评估一个位置好不好 | V(s) | 直接告诉你"这位置值多少" |
| 评估一个动作对不对 | Q(s,a) | 告诉你"在这种情况下做这个动作值多少" |
| 已知策略，想评估好坏 | V(s)就够了 | 只需要知道每个状态值多少 |
| 不知道策略，想改进 | 需要Q(s,a) | 要比较不同动作的好坏才能选最优 |

---

## 六、最优策略：什么才是最好的？

### 6.1 最优策略的定义

所有的策略里，有没有一个"最好的"？

**最优策略 $\pi^*$ 的定义**：对于所有状态 $s$，都有

$$
V^{\pi^*}(s) \geq V^\pi(s), \quad \forall \pi
$$

翻译：**不管你在哪个状态，按最优策略行事，平均回报都不会比按任何其他策略差**。

### 6.2 最优策略的特点

**好消息是**：最优策略**一定存在**（对于有限状态的MDP）。

**另一个好消息是**：最优策略**可能是确定性的**——也就是说，我们总能找到一种策略，在每个状态都给出唯一的最优动作。

这很实用，意味着我们可以把最优策略想象成一张"攻略地图"：在每个状态，你应该走哪条路，全都标得清清楚楚。

### 6.3 最优价值函数

与最优策略对应的价值函数叫做**最优价值函数**：

$$
V^*(s) = \max_\pi V^\pi(s)
$$

以及**最优Q函数**：

$$
Q^*(s,a) = \max_\pi Q^\pi(s,a)
$$

最优V和最优Q之间的关系：

$$
V^*(s) = \max_a Q^*(s,a)
$$

翻译：**最好的局面 = 挑那个最好的动作来做**。

---

## 七、Bellman方程：从递归思考到动态规划

### 7.1 递归思考的智慧

你有没有过这种体验：做重大决策的时候，你会想"如果我选A，将来会怎样？如果我选B，将来又会怎样？"

这就是**递归思考**的精髓：**当前的价值 = 立即获得的奖励 + 未来的价值**。

Bellman方程就是这种递归思考的数学表达。

### 7.2 Bellman期望方程

对于给定的策略 $\pi$，价值函数满足：

$$
V^\pi(s) = \sum_a \pi(a|s) \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]
$$

这个公式在说什么？

1. **当前状态的价值** = **平均** ( **做每个动作的价值** )
2. **做某个动作的价值** = **立即奖励** + **打折后的下一个状态的价值**

Q函数的版本：

$$
Q^\pi(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s',a') \right]
$$

### 7.3 Bellman最优方程

当我们追求最优策略时，价值函数满足更强的条件：

$$
V^*(s) = \max_a \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^*(s') \right]
$$

这里多了个 $\max_a$，意思是：**不是平均所有动作的价值，而是挑最好的那个**。

最优Q函数：

$$
Q^*(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \max_{a'} Q^*(s',a') \right]
$$

### 7.4 Bellman方程为什么重要？

Bellman方程是强化学习的**基石**，原因有三：

1. **揭示递归结构**：把一个无限的问题变成递归定义
2. **连接现在与未来**：当前价值 = 立即奖励 + 未来价值
3. **为迭代求解奠基**：我们可以从某个初始猜测开始，反复用Bellman方程来更新价值估计，直到收敛

---

## 八、策略迭代 vs 价值迭代：两种找最优策略的方法

### 8.1 策略迭代：先评估再改进

策略迭代像是一个**精益求精的完美主义者**：

```
初始化一个策略 π
repeat:
    1. 策略评估：计算 V^π（当前策略值多少）
    2. 策略改进：对于每个状态，选让 V 最大的动作
until 策略不再变化
```

**策略评估**：已知策略 $\pi$，解线性方程 $V^\pi = \mathcal{T}^\pi V^\pi$ 得到 $V^\pi$

**策略改进**：对于每个状态 $s$，找能让 $Q(s,a)$ 最大的动作 $a$

### 8.2 价值迭代：一步到位

价值迭代像是一个**直接锁定目标的狙击手**：

```
初始化 V(s) = 0，对所有 s
repeat:
    对于每个状态 s:
        计算每个动作的 Q(s,a)
        V(s) = max_a Q(s,a)
until V 变化很小
```

价值迭代直接求解 Bellman 最优方程，不显式存储策略——策略是"隐含"在价值函数里的。

### 8.3 两种方法的对比

| 方面 | 策略迭代 | 价值迭代 |
|:-----|:--------|:--------|
| 思路 | 交替评估和改进 | 直接逼近最优 |
| 每次迭代 | 评估整个价值函数 | 只做一次更新 |
| 收敛速度 | 通常更快（策略稳定） | 更慢（每步只改一点点） |
| 适用场景 | 小规模问题 | 大规模问题 |
| 代码实现 | 更复杂（需要解方程） | 更简单（纯迭代） |

> [!tip]+ 实践建议
> 对于状态空间较小的问题（如10-100个状态），用策略迭代。对于大状态空间，用价值迭代或其变种。

---

## 九、用Python手写MDP求解器

### 9.1 基础MDP类

```python
import numpy as np

class MDP:
    """
    简化的MDP类，支持策略迭代和价值迭代。
    """
    def __init__(self, n_states, n_actions, gamma=0.99):
        self.n_states = n_states
        self.n_actions = n_actions
        self.gamma = gamma
        
        # 转移概率 P(s'|s, a)
        self.transition = np.zeros((n_states, n_actions, n_states))
        
        # 奖励 R(s, a, s')
        self.reward = np.zeros((n_states, n_actions, n_states))
    
    def value_iteration(self, theta=1e-8, max_iterations=1000):
        """
        价值迭代算法。
        
        思路：直接从Bellman最优方程出发，迭代更新价值函数。
        """
        V = np.zeros(self.n_states)
        policy = np.zeros(self.n_states, dtype=int)
        
        for iteration in range(max_iterations):
            delta = 0  # 最大变化量
            
            for s in range(self.n_states):
                old_v = V[s]
                
                # 计算每个动作的价值
                action_values = []
                for a in range(self.n_actions):
                    v = 0
                    for s_next in range(self.n_states):
                        # Bellman最优方程的核心
                        v += self.transition[s, a, s_next] * (
                            self.reward[s, a, s_next] + 
                            self.gamma * 0  # 这里用旧的价值
                        )
                    action_values.append(v)
                
                # 取最大值
                V[s] = max(action_values)
                policy[s] = np.argmax(action_values)
                
                delta = max(delta, abs(V[s] - old_v))
            
            # 检查收敛
            if delta < theta:
                print(f"价值迭代在第 {iteration + 1} 次迭代后收敛")
                break
        
        return V, policy
    
    def policy_iteration(self, max_iterations=100):
        """
        策略迭代算法。
        
        思路：先评估策略的价值，再改进策略，交替进行直到收敛。
        """
        # 随机初始化策略
        policy = np.random.randint(0, self.n_actions, self.n_states)
        
        for iteration in range(max_iterations):
            # 步骤1：策略评估
            V = self._evaluate_policy(policy)
            
            # 步骤2：策略改进
            new_policy = self._improve_policy(V, policy)
            
            # 检查是否收敛
            if np.array_equal(policy, new_policy):
                print(f"策略迭代在第 {iteration + 1} 次迭代后收敛")
                break
            
            policy = new_policy
        
        return V, policy
    
    def _evaluate_policy(self, policy, theta=1e-8, max_iter=1000):
        """
        评估给定策略的价值函数。
        
        使用迭代方法解 Bellman 期望方程。
        """
        V = np.zeros(self.n_states)
        
        for _ in range(max_iter):
            delta = 0
            
            for s in range(self.n_states):
                old_v = V[s]
                a = policy[s]
                
                # 计算该策略下状态s的价值
                v = 0
                for s_next in range(self.n_states):
                    v += self.transition[s, a, s_next] * (
                        self.reward[s, a, s_next] + 
                        self.gamma * V[s_next]
                    )
                
                V[s] = v
                delta = max(delta, abs(v - old_v))
            
            if delta < theta:
                break
        
        return V
    
    def _improve_policy(self, V, old_policy):
        """
        基于价值函数改进策略。
        
        对于每个状态，选择能带来最大Q值的动作。
        """
        new_policy = np.zeros(self.n_states, dtype=int)
        
        for s in range(self.n_states):
            best_action = 0
            best_value = float('-inf')
            
            for a in range(self.n_actions):
                # 计算 Q(s, a)
                q = 0
                for s_next in range(self.n_states):
                    q += self.transition[s, a, s_next] * (
                        self.reward[s, a, s_next] + 
                        self.gamma * V[s_next]
                    )
                
                if q > best_value:
                    best_value = q
                    best_action = a
            
            new_policy[s] = best_action
        
        return new_policy
```

### 9.2 走迷宫MDP

```python
class MazeMDP(MDP):
    """
    用MDP建模走迷宫问题。
    """
    def __init__(self, maze, start, goal, gamma=0.99):
        """
        maze: 2D array, 0=通道, 1=墙, 2=目标
        start: (row, col) 起点
        goal: (row, col) 目标
        """
        self.maze = maze
        self.height, self.width = maze.shape
        self.start = start
        self.goal = goal
        
        # 计算状态数量
        n_states = self.height * self.width
        n_actions = 4  # 上、下、左、右
        
        super().__init__(n_states, n_actions, gamma)
        
        # 构建转移和奖励
        self._build_mdp()
    
    def _pos_to_state(self, row, col):
        """坐标转状态索引"""
        return row * self.width + col
    
    def _state_to_pos(self, s):
        """状态索引转坐标"""
        return s // self.width, s % self.width
    
    def _build_mdp(self):
        """构建MDP的转移概率和奖励"""
        # 动作：0=上, 1=下, 2=左, 3=右
        self.action_deltas = [(-1, 0), (1, 0), (0, -1), (0, 1)]
        
        for row in range(self.height):
            for col in range(self.width):
                if self.maze[row, col] == 1:  # 墙
                    continue
                
                s = self._pos_to_state(row, col)
                
                for a, (dr, dc) in enumerate(self.action_deltas):
                    nr, nc = row + dr, col + dc
                    
                    # 检查是否在边界内
                    if 0 <= nr < self.height and 0 <= nc < self.width:
                        ns = self._pos_to_state(nr, nc)
                        
                        # 墙不能通行
                        if self.maze[nr, nc] != 1:
                            self.transition[s, a, ns] = 1.0
                            
                            # 奖励设计
                            if (nr, nc) == self.goal:
                                self.reward[s, a, ns] = 100.0  # 到达目标
                            else:
                                self.reward[s, a, ns] = -0.1  # 每步小惩罚
                        else:
                            # 撞墙，留在原地
                            self.transition[s, a, s] = 1.0
                            self.reward[s, a, s] = -1.0
                    else:
                        # 出界，留在原地
                        self.transition[s, a, s] = 1.0
                        self.reward[s, a, s] = -1.0
    
    def visualize(self, V=None, policy=None):
        """可视化价值和策略"""
        symbols = {
            0: '↑', 1: '↓', 2: '←', 3: '→',
            'wall': '█', 'start': 'S', 'goal': 'G', 'empty': '·'
        }
        
        for row in range(self.height):
            line = ""
            for col in range(self.width):
                if self.maze[row, col] == 1:
                    line += symbols['wall']
                elif (row, col) == self.start:
                    line += symbols['start']
                elif (row, col) == self.goal:
                    line += symbols['goal']
                elif policy is not None:
                    s = self._pos_to_state(row, col)
                    line += symbols[policy[s]]
                else:
                    line += symbols['empty']
            print(line)
```

### 9.3 使用示例

```python
# 创建一个简单的迷宫
maze = np.array([
    [0, 0, 0, 1, 0],
    [0, 1, 0, 0, 0],
    [0, 0, 0, 1, 0],
    [1, 1, 0, 0, 0],
    [0, 0, 0, 1, 2],  # 2是目标
])

# 创建MDP
mdp = MazeMDP(maze, start=(0, 0), goal=(4, 4), gamma=0.95)

# 用价值迭代求解
V, policy = mdp.value_iteration(theta=1e-6)
print("\n价值迭代结果：")
print("价值函数：", np.round(V, 2))
print("\n最优策略：")
mdp.visualize(policy=policy)

# 用策略迭代求解
V2, policy2 = mdp.policy_iteration()
print("\n策略迭代结果：")
print("价值函数：", np.round(V2, 2))
print("\n最优策略：")
mdp.visualize(policy=policy2)
```

---

## 十、MDP的局限与扩展

### 10.1 MDP的核心局限

MDP有两个关键假设：

1. **智能体能完全观测环境状态**（满足马尔可夫性）
2. **环境是确定性的或随机但稳定的**（转移概率不随时间变化）

现实世界往往不满足这些假设。

### 10.2 POMDP：部分可观测的世界

当你**看不到**完整状态时，怎么办？

举个例子：你玩牌的时候，你看不到对手手里的牌。这是一个**部分可观测（Partially Observable）**的场景。

POMDP（Partially Observable Markov Decision Process）通过引入**观测（Observation）**来建模这种情况：

- **真实状态** $s$（你不知道）
- **观测** $o$（你能看到的）
- **观测函数** $O(o|s)$：在状态 $s$ 下，看到观测 $o$ 的概率

POMDP的核心思路是：**维护一个"信念状态"（belief state）**，也就是对真实状态的概率分布估计。

这就像打扑克：你虽然不知道对手有什么牌，但你可以根据他之前的行为来推断他的牌力分布。

### 10.3 其他扩展方向

| 扩展类型 | 解决的问题 | 典型应用 |
|:--------|:----------|:--------|
| **POMDP** | 状态不可完全观测 | 机器人导航、对话系统 |
| **CMDP** | 有约束条件 | 安全强化学习 |
| **层次MDP** | 任务太复杂，需要分层 | 机器人操作、自然语言任务 |
| **多智能体MDP** | 多个智能体交互 | 团队协作、游戏AI |
| **鲁棒MDP** | 转移模型不确定 | 自动驾驶、控制系统 |

---

## 十一、相关文档

- [[Q学习深度指南|Q学习算法]] — 无模型强化学习的经典方法
- [[DQN深度指南|深度Q网络]] — 结合深度学习的Q学习扩展
- [[策略梯度方法详解|策略梯度方法]] — 直接优化策略的强化学习方法
- [[PPO深度指南|PPO算法]] — 主流的策略优化算法
- [[多智能体RL详解|多智能体RL]] — 多个智能体协同决策
- [[RL应用场景|RL应用场景]] — 强化学习的实际应用

---

## 参考文献

1. Sutton, R. S., & Barto, A. G. (2018). *Reinforcement Learning: An Introduction* (2nd ed.). MIT Press.
2. Bellman, R. (1957). *Dynamic Programming*. Princeton University Press.
3. Puterman, M. L. (1994). *Markov Decision Processes: Discrete Stochastic Dynamic Programming*. Wiley.
4. Bertsekas, D. P. (2017). *Dynamic Programming and Optimal Control* (Vol. I-II). Athena Scientific.
5. Mnih, V., et al. (2015). Human-level control through deep reinforcement learning. *Nature*, 518(7540), 529-533.

---

*MDP是强化学习的数学基础。掌握好这一章，后面的Q学习、DQN、PPO等算法理解起来会轻松很多。多做习题，多动手写代码，概念自然就清晰了。*
