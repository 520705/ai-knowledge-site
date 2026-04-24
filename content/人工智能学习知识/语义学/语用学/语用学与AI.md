---
title: 语用学与AI
date: 2026-04-18
tags:
  - 语用学
  - 言语行为理论
  - 合作原则
  - 关联理论
  - 指示语
  - 预设
  - 隐涵
  - 大语言模型
  - LLM
  - 语用推理
categories:
  - 人工智能/语义学
  - 人工智能/语言理解
  - 认知科学/语用学
alias: [Pragmatics, Pragmatics in AI, 语用推理]
---

# 语用学与AI

> [!note] 文档概述
> 本文档系统探讨语用学的核心理论体系及其与人工智能的深度关联。语用学研究语言使用中的意义，包括言语行为、合作原则、关联理论等核心理论，以及这些理论在大语言模型中的实现与应用挑战。

## 关键词速览

| 术语 | 英文 | 核心定义 |
|------|------|----------|
| 言语行为理论 | Speech Act Theory | 研究语言作为行为工具的理论 |
| 合作原则 | Cooperative Principle | 对话遵循的基本准则 |
| 关联理论 | Relevance Theory | 交际基于最大关联原则 |
| 指示语 | Deixis | 依赖语境的指代表达 |
| 预设 | Presupposition | 话语隐含的背景假设 |
| 隐涵 | Implicature | 话语的隐含意义 |
| 会话含义 | Conversational Implicature | 违反合作原则产生的隐含 |
| 语用推理 | Pragmatic Inference | 语境依赖的推理过程 |
| 指示语 | Indexical | 指向特定语境的表达式 |
| 面子理论 | Face Theory | 面子威胁行为与礼貌策略 |

---

## 一、语用学的学科定位

### 1.1 语用学的定义与范围

语用学（Pragmatics）是研究语言使用者在特定语境中如何理解和产生意义的学科。与语义学研究字面意义不同，语用学关注的是：

1. **说话者意图**：语言背后想要传达的真正含义
2. **语境依赖**：意义如何随语境变化
3. **隐性信息**：未明确表达但隐含的信息
4. **言语效果**：语言使用产生的交际效果

**语义学 vs 语用学的对比**

| 维度 | 语义学 | 语用学 |
|------|--------|--------|
| 研究对象 | 语言的字面意义 | 语言的使用意义 |
| 语境依赖 | 低 | 高 |
| 主观性 | 低 | 高 |
| 核心问题 | "X是什么意思" | "说X意味着什么" |
| 典型问题 | 词义、句子结构 | 意图、礼貌、讽刺 |

### 1.2 语用学的历史渊源

**哲学根源**
语用学的哲学根源可追溯至：
- **Peirce的指号学**：符号在使用中才有意义
- **Wittgenstein的语言游戏**：意义在于使用
- **Austin的言语行为理论**：说话就是做事

**语言学发展**
- 1970年代：Austin和Searle的言语行为理论
- 1975年：Grice的合作原则与会话含义
- 1980年代：Sperber和Wilson的关联理论
- 1990年代至今：形式语用学与计算语用学

---

## 二、言语行为理论（Speech Act Theory）

### 2.1 Austin的言语行为三分说

约翰·奥斯汀（John Austin）区分了三种言语行为：

#### 2.1.1 话语行为（Locutionary Act）

话语行为指说话发出的物理声音及其表达的具体意义：

$$
\text{Locution}(S) = \text{Phonetic}(S) + \text{Phatic}(S) + \text{Rhetic}(S)
$$

其中：
- Phonetic：语音/书写行为
- Phatic：语法/结构行为
- Rhetic：意义/指称行为

**示例**
"门开着。"这句话的话语行为是表达了一个关于门的状态的陈述。

#### 2.1.2 话语施事行为（Illocutionary Act）

话语施事行为是指话语在说出时执行的某种行为：

| 言语行为类型 | 说明 | 示例 |
|-------------|------|------|
| 断言类 (Assertives) | 描述世界状态 | "猫在垫子上" |
| 指令类 (Directives) | 要求他人做某事 | "请关门" |
| 承诺类 (Commissives) | 承诺未来行动 | "我会来" |
| 表达类 (Expressives) | 表达心理状态 | "我很高兴" |
| 宣告类 (Declarations) | 改变世界状态 | "我宣布..." |

**施事动词（Illocutionary Force）**

施事动词是标记言语行为类型的关键：

```python
def classify_speech_act(verb, utterance):
    """
    基于施事动词的言语行为分类
    
    实际系统需要更复杂的推理
    """
    speech_act_taxonomy = {
        'assertives': ['说', '陈述', '宣称', '认为', '相信'],
        'directives': ['请', '要求', '命令', '建议', '禁止'],
        'commissives': ['承诺', '发誓', '保证', '约定'],
        'expressives': ['感谢', '祝贺', '道歉', '抱怨'],
        'declarations': ['宣布', '任命', '命名', '判决']
    }
    
    for act_type, verbs in speech_act_taxonomy.items():
        if any(v in verb for v in verbs):
            return act_type
    return 'unknown'
```

#### 2.1.3 话语施效行为（Perlocutionary Act）

话语施效行为指话语对听者产生的效果：

$$
\text{Perlocutionary Effect}(U \text{ on } H) = \text{某种心理或行为变化}
$$

**常见施效效果**
- **认识效果**：改变听者的知识或信念
- **情感效果**：引起听者的情感反应
- **行为效果**：促使听者采取行动

```
话语: "外面在下雨。"
    
话语行为: 陈述天气状态
话语施事行为: 暗示/警告
话语施效行为: 听者决定带伞
```

### 2.2 Searle的言语行为分类

约翰·塞尔（John Searle）进一步完善了言语行为理论：

#### 2.2.1 分类体系

```python
class SpeechActClassifier:
    """
    基于Searle分类的言语行为分类器
    
    使用规则和上下文信息进行分类
    """
    
    def __init__(self):
        self.speech_acts = {
            'assertives': {
                'description': '断言或描述世界状态',
                '命题内容方向': '世界→命题',
                '心理状态': '相信',
                'direction': 'word-to-world',
                'examples': ['陈述', '断言', '宣称', '结论']
            },
            'directives': {
                'description': '试图使听者做某事',
                '命题内容方向': '未确定',
                '心理状态': '想要/希望',
                'direction': 'world-to-word',
                'examples': ['请求', '命令', '建议', '询问']
            },
            'commissives': {
                'description': '承诺说话者未来行动',
                '命题内容方向': '未来行动',
                '心理状态': '意图',
                'direction': 'world-to-word',
                'examples': ['承诺', '发誓', '保证', '发誓']
            },
            'expressives': {
                'description': '表达心理状态',
                '命题内容方向': '以说话者为中心',
                '心理状态': '表达',
                'direction': 'none',
                'examples': ['感谢', '道歉', '祝贺', '哀悼']
            },
            'declarations': {
                'description': '改变世界状态',
                '命题内容方向': '自我实现',
                '心理状态': '无需特定',
                'direction': 'self-verifying',
                'examples': ['任命', '命名', '宣布', '祝福']
            }
        }
    
    def classify(self, utterance, context=None):
        """分类言语行为"""
        # 使用关键词和上下文线索
        ...
```

### 2.3 言语行为与AI对话系统

言语行为理论在对话系统中有重要应用：

#### 2.3.1 对话意图识别

```python
class DialogueIntentClassifier:
    """
    对话系统中的意图识别
    
    基于言语行为理论设计意图分类体系
    """
    
    def __init__(self):
        self.intent_taxonomy = {
            # 信息获取类
            'query': ['询问', '查询', '想知道', '什么'],
            'clarification': ['你的意思是', '具体指'],
            
            # 任务执行类
            'request': ['帮我', '请', '能不能', '请帮我'],
            'command': ['执行', '去做', '完成'],
            'suggestion': ['建议', '可以试试', '不妨'],
            
            # 社会交互类
            'greeting': ['你好', '早上好', 'hi', 'hello'],
            'farewell': ['再见', '拜拜', '回头见'],
            'thanks': ['谢谢', '感谢', '多谢'],
            
            # 情感表达类
            'complaint': ['不满', '失望', '糟糕'],
            'praise': ['不错', '很好', '厉害']
        }
    
    def predict_intent(self, utterance):
        """预测用户意图"""
        scores = {}
        for intent, patterns in self.intent_taxonomy.items():
            score = sum(1 for p in patterns if p in utterance)
            if score > 0:
                scores[intent] = score
        
        if scores:
            return max(scores, key=scores.get)
        return 'unknown'
```

---

## 三、合作原则与会话含义

### 3.1 Grice的合作原则

保罗·格赖斯（Paul Grice）提出，交际遵循"合作原则"（Cooperative Principle）：

> 使你的话语在其所发生的阶段符合当前交谈的期望。

#### 3.1.1 四大量准则

**量的准则（Quantity Maxim）**
- 所说的话应包含当前交谈目的所需的信息
- 所说的话不应包含超出所需的信息

**质的准则（Quality Maxim）**
- 不要说你认为是虚假的话
- 不要说缺乏足够证据的话

**关系准则（Relevance Maxim）**
- 要有关联性（相关性）

**方式准则（Manner Maxim）**
- 避免表达模糊
- 避免歧义
- 简洁（避免不必要的冗长）
- 有条理

```
合作原则示意图:

        交际目标
            ↓
    ┌─────────────┐
    │  量的准则  │ ← 提供适量信息
    │  质的准则  │ ← 说真话
    │  关系的    │ ← 说相关的
    │  方式的    │ ← 清楚地说
    └─────────────┘
            ↓
      有效交际
```

#### 3.1.2 会话含义的类型

**标准会话含义**
直接遵循准则产生的推论：

```
A: "外面有人吗？"
B: "地上有湿脚印。"

标准含义：B在暗示外面有人（从脚印推断）
```

**特殊会话含义**
通过**违反**某一准则产生的隐含意义：

```
A: "张三能借我500块吗？"
B: "他最近手头有点紧。"

违反质的准则（没有直接回答能或不能）
→ 特殊含义：张三可能没法借
```

### 3.2 会话含义的计算模型

```python
class ConversationalImplicatureCalculator:
    """
    会话含义计算器
    
    基于Grice的合作原则计算隐含意义
    """
    
    def __init__(self):
        self.maxims = ['quantity', 'quality', 'relation', 'manner']
    
    def calculate_implicature(self, utterance, context, expected_response):
        """
        计算违反合作原则产生的会话含义
        
        Args:
            utterance: 实际话语
            context: 对话上下文
            expected_response: 符合合作原则的预期回答
        
        Returns:
            inferred_implicature: 推断的隐含意义
        """
        # 分析违反的准则
        violated_maxims = self._detect_violations(utterance, expected_response)
        
        # 计算隐含意义
        implicatures = []
        for maxim in violated_maxims:
            implicature = self._derive_implicature(utterance, maxim, context)
            implicatures.append(implicature)
        
        return implicatures
    
    def _detect_violations(self, utterance, expected):
        """检测违反的准则"""
        violations = []
        
        # 量的违规检测
        if len(utterance) < len(expected) * 0.5:
            violations.append('quantity_insufficient')
        elif len(utterance) > len(expected) * 2:
            violations.append('quantity_excessive')
        
        # 质的违规检测
        if 'not sure' in utterance.lower() or 'maybe' in utterance.lower():
            violations.append('quality_insufficient')
        
        # 关系违规检测
        if not self._is_relevant(utterance, expected):
            violations.append('relation_irrelevant')
        
        return violations
    
    def _derive_implicature(self, utterance, violation, context):
        """推导隐含意义"""
        # 这是一个简化的模型
        # 实际需要更复杂的推理
        implicature_map = {
            'quantity_insufficient': '说话者有意隐瞒部分信息',
            'quantity_excessive': '说话者在强调或补充细节',
            'quality_insufficient': '说话者对信息不确定',
            'relation_irrelevant': '话语有其他隐含意图'
        }
        return implicature_map.get(violation, '无法确定隐含意义')
```

### 3.3 礼貌原则

列文森（Leech）提出礼貌原则作为合作原则的补充：

> 最小限度地让别人受损，最大限度地让别人得益。

#### 3.3.1 礼貌策略

```python
class PolitenessModel:
    """
    礼貌语用模型
    
    基于Brown和Levinson的面子理论
    """
    
    def __init__(self):
        self.politeness_strategies = {
            'bald_on_record': '直接表达，无修饰',
            'positive_politeness': '强调共同点，满足积极面子',
            'negative_politeness': '给面子，留余地',
            'off_record': '间接表达，可否认'
        }
    
    def select_strategy(self, utterance_type, social_distance, power_diff):
        """
        选择礼貌策略
        
        Args:
            utterance_type: 话语类型（请求、道歉等）
            social_distance: 社会距离（亲密-陌生）
            power_diff: 权力差异（平等-上下级）
        """
        # 计算面子威胁程度
        face_threat = self._calculate_face_threat(
            utterance_type, social_distance, power_diff
        )
        
        # 根据威胁程度选择策略
        if face_threat < 0.3:
            return 'bald_on_record'
        elif face_threat < 0.6:
            return 'positive_politeness'
        elif face_threat < 0.8:
            return 'negative_politeness'
        else:
            return 'off_record'
    
    def polite_transform(self, request):
        """将直接请求转化为礼貌表达"""
        transformations = {
            'want': 'Would you like to...',
            'need': 'Could you please...',
            'must': 'Would it be possible to...',
            'imperative': 'Would you mind...'
        }
        # 实现转换逻辑
        ...
```

---

## 四、关联理论（Relevance Theory）

### 4.1 理论概述

丹·斯珀伯（Dan Sperber）和戴尔德丽·威尔逊（Deirdre Wilson）在1986年提出关联理论：

> 人类的交际由对最大关联（maximal relevance）的期望驱动。

#### 4.1.1 核心概念

**认知关联原则**
> 人类的认知倾向于最大化关联。

**交际关联原则**
> 每个话语都传递最大关联的假定。

**关联性的定义**

$$
\text{关联性} = \frac{\text{认知效果}}{\text{处理努力}}
$$

认知效果包括：
- 新信息与旧信息的结合
- 假设的强化或削弱
- 假设的消除

### 4.2 关联理论的形式化

```python
class RelevanceTheoreticProcessor:
    """
    关联理论处理器
    
    实现基于关联原则的语用推理
    """
    
    def __init__(self):
        self.context_beliefs = {}  # 语境信念
        self.entailed_propositions = []  # 蕴含命题
    
    def process_utterance(self, utterance, context=None):
        """
        处理话语，计算关联性
        
        步骤:
        1. 构建假设集合
        2. 计算认知效果
        3. 评估处理努力
        4. 判断是否达到关联阈值
        """
        # 1. 命题识别
        proposition = self.extract_proposition(utterance)
        
        # 2. 语境扩展
        enriched_context = self.expand_context(context, proposition)
        
        # 3. 计算认知效果
        cognitive_effects = self.compute_effects(proposition, enriched_context)
        
        # 4. 计算处理努力
        processing_effort = self.estimate_processing_effort(utterance)
        
        # 5. 计算关联性
        relevance = cognitive_effects / processing_effort
        
        return {
            'proposition': proposition,
            'cognitive_effects': cognitive_effects,
            'processing_effort': processing_effort,
            'relevance': relevance,
            'is_relevant': relevance > self.threshold
        }
    
    def compute_effects(self, proposition, context):
        """
        计算认知效果
        
        认知效果包括:
        - 新旧信息的结合
        - 假设的强化/削弱
        - 矛盾假设的消除
        """
        effects = 0
        
        # 检查与现有假设的一致性
        for assumption in self.entailed_propositions:
            if self.supports(proposition, assumption):
                effects += 1  # 强化
            elif self.contradicts(proposition, assumption):
                effects += 2  # 矛盾消除，效果更大
            else:
                effects += 0.5  # 新信息组合
        
        return effects
```

### 4.3 隐含意义与推理

关联理论强调交际中的**隐含**（Implicature）与**显义**（Explicature）的区分：

```
话语处理流程:

显义（Explicature）
    ↓
话语意义 ← 话语解码 + 语境充实
    ↓
隐含（Implicature）
    ↓
语境假设 ← 推理过程
    ↓
话语含义 ← 逻辑结构 + 百科知识
```

---

## 五、指示语（Deixis）

### 5.1 指示语的类型

指示语（Deixis）是语用学的核心概念，指依赖语境才能确定指称的表达式。

#### 5.1.1 人称指示语（Person Deixis）

```python
def resolve_person_deixis(pronoun, speaker, hearer, context):
    """
    解决人称指示语
    
    人称系统:
    - 第一人称: 说话者 (I, we)
    - 第二人称: 听者 (you)
    - 第三人称: 其他 (he, she, it, they)
    """
    person_mapping = {
        '我': speaker,
        '我们': f"{speaker}等人",
        '你': hearer,
        '你们': f"{hearer}等人",
        '他': context.get('third_party_male', '未知男性'),
        '她': context.get('third_party_female', '未知女性'),
        '它': context.get('third_party_object', '未知物体')
    }
    return person_mapping.get(pronoun, pronoun)
```

#### 5.1.2 时间指示语（Temporal Deixis）

```
时间指示语的参照系统:
                    说话时间（现在）
                         ↓
    ┌──────────────────┼──────────────────┐
    │                  │                  │
  过去              现在                未来
   │                  │                  │
 yesterday      today              tomorrow
   │                  │                  │
  last week     now                  next week
```

```python
def resolve_temporal_deixis(deictic_time, speech_time):
    """
    解决时间指示语
    
    时间指示语的计算
    """
    import datetime
    
    time_mapping = {
        '今天': speech_time.date(),
        '昨天': speech_time.date() - datetime.timedelta(days=1),
        '明天': speech_time.date() + datetime.timedelta(days=1),
        '现在': speech_time,
        '刚才': speech_time - datetime.timedelta(minutes=5),
        '马上': speech_time + datetime.timedelta(minutes=1)
    }
    
    return time_mapping.get(deictic_time, deictic_time)
```

#### 5.1.3 空间指示语（Space Deixis）

```python
def resolve_spatial_deixis(deictic_location, speaker_location):
    """
    解决空间指示语
    
    空间指示语基于:
    - 说话者位置 (deictic center)
    - 听者位置
    - 参照物体
    """
    spatial_system = {
        '这里': speaker_location,  # 说话者所在
        '那里': '听者或第三方位置',  # 需要上下文
        '这个': '靠近说话者的物体',
        '那个': '远离说话者的物体'
    }
    return spatial_system.get(deictic_location, deictic_location)
```

### 5.2 指示语的计算消解

```python
class DeicticResolver:
    """
    指示语消解系统
    
    将指示表达式映射到具体实体/时间/空间
    """
    
    def __init__(self):
        self.deictic_types = ['person', 'temporal', 'spatial', 'discourse']
    
    def resolve(self, utterance, context):
        """
        统一消解指示语
        """
        resolved = utterance
        
        # 人称指示语
        resolved = self.resolve_person(resolved, context)
        
        # 时间指示语
        resolved = self.resolve_temporal(resolved, context)
        
        # 空间指示语
        resolved = self.resolve_spatial(resolved, context)
        
        # 话语指示语
        resolved = self.resolve_discourse(resolved, context)
        
        return resolved
    
    def resolve_person(self, text, context):
        """人称消解"""
        person_patterns = [
            (r'\b我\b', context.get('speaker', 'Unknown')),
            (r'\b你\b', context.get('hearer', 'Unknown')),
            (r'\b他\b', context.get('third_party_male', 'Unknown')),
            (r'\b她\b', context.get('third_party_female', 'Unknown'))
        ]
        
        for pattern, replacement in person_patterns:
            text = re.sub(pattern, f'[{replacement}]', text)
        
        return text
    
    def resolve_discourse(self, text, context):
        """话语指示语消解（回指）"""
        # 处理回指，如"这个"、"那件事"等
        discourse_patterns = [
            (r'这(.+?)', f"前文提到的\\1"),
            (r'那(.+?)', f"前文提到的\\1")
        ]
        ...
```

---

## 六、预设（Presupposition）

### 6.1 预设的定义与性质

预设是话语中隐含的、说话者假定为真的背景信息：

```
"他又迟到了。"

预设: [他之前迟到过]
断言: 他现在迟到了

预设不为断言的否定所取消:
"他没有迟到。" → 预设仍然存在：[他之前迟到过]
```

#### 6.1.1 预设触发语

| 类型 | 示例 | 预设内容 |
|------|------|----------|
| 事实动词 | "知道"、"忘记" | 某事发生 |
| 状态变化 | "停止"、"开始" | 先前状态 |
| 限定性描述 | "又"、"再次" | 重复性 |
| 评价性形容词 | "重新" | 某事已做过 |
| 分裂句 | "是...才..." | 焦点信息 |

```python
class PresuppositionTriggers:
    """
    预设触发语识别与提取
    """
    
    def __init__(self):
        self.triggers = {
            'factives': {
                '知道': '命题P为真',
                '意识到': '命题P为真',
                '忘记': '事件发生过'
            },
            'change_of_state': {
                '停止': '状态S1之前成立',
                '开始': '状态S1之前不成立',
                '继续': '状态持续'
            },
            'iteratives': {
                '又': '事件E之前发生过',
                '再': '事件E重复',
                '重新': 'E已完成'
            },
            'temporal': {
                '在...之前': '时间T先于某事件',
                '已经': '事件E发生'
            }
        }
    
    def extract_presuppositions(self, utterance):
        """提取话语中的预设"""
        presuppositions = []
        
        for category, triggers in self.triggers.items():
            for trigger, presupposition in triggers.items():
                if trigger in utterance:
                    presuppositions.append({
                        'trigger': trigger,
                        'type': category,
                        'presupposition': presupposition,
                        'utterance': utterance
                    })
        
        return presuppositions
```

### 6.2 预设的投射问题

预设如何从复合句的各成分投射到整句：

```
"张三知道李四又迟到了。"

成分预设:
- 张三知道: 李四迟到了
- 又: 李四之前迟到过

投射结果: 张三知道"李四之前迟到过"（隐含）
```

```python
class PresuppositionProjector:
    """
    预设投射计算
    
    使用 Heim's ATLAS 或类似模型
    """
    
    def __init__(self):
        self.operators = {
            'negation': self.negate,
            'question': self.question,
            'belief': self.believe,
            'conditional': self.conditional
        }
    
    def project(self, presupposition, operator):
        """
        将预设通过算子投射
        
        算子类型影响预设的保留/消除
        """
        projector = self.operators.get(operator, self.default)
        return projector(presupposition)
    
    def negate(self, prop):
        """否定算子：预设保留"""
        return {
            'presupposition': prop,
            'assertion': f"NOT {prop}",
            'presupposition_preserved': True
        }
    
    def conditional(self, prop):
        """条件算子：预设投射到结果部"""
        return {
            'presupposition': prop,
            'condition': f"IF condition THEN {prop}",
            'presupposition_preserved': True
        }
```

---

## 七、大语言模型中的语用推理

### 7.1 LLM语用能力的现状

大语言模型（LLM）在语用理解方面展现出一定能力，但仍存在挑战：

#### 7.1.1 成功案例

**间接言语理解**

```
用户: "外面很冷"
LLM理解: [用户可能想要关窗/开暖气/表达不舒服]
→ 响应: "我帮你把窗户关上吧，或者开一下暖气？"
```

**隐含意图识别**

```
用户: "我的打印机好像有问题"
LLM理解: [用户需要打印相关的帮助]
→ 响应: "让我帮你检查一下打印机设置"
```

#### 7.1.2 局限性

```python
# LLM语用能力的典型失败案例

failures = {
    'irony_sarcasm': "你说'这计划真棒'时，实际意思是批评",
    'indirect_refusal': "用户问'你能帮我...吗'，LLM有时无法识别这是拒绝的机会",
    'implied_request': "'我有点累了' 隐含想要休息的帮助",
    'cultural_pragmatics': "不同文化中'可以'的含义差异"
}
```

### 7.2 增强LLM语用能力的方法

#### 7.2.1 提示工程方法

```python
class PragmaticPromptEngineer:
    """
    语用感知提示工程
    
    通过提示增强LLM的语用理解能力
    """
    
    def add_pragmatic_context(self, utterance):
        """
        添加语用上下文提示
        """
        pragmatic_prompts = {
            'implicit_request': """
            注意：以下话语可能包含隐含请求。
            请尝试推断说话者可能的真实意图。
            """,
            'irony_sarcasm': """
            注意：以下话语可能包含反语或讽刺。
            请从语境和语气中识别真实的情感倾向。
            """,
            'politeness': """
            注意：以下话语使用了礼貌策略。
            请分析话语的直接含义和礼貌效果。
            """
        }
        
        return pragmatic_prompts
    
    def generate_pragmatic_prompt(self, utterance, context):
        """
        生成包含语用分析的完整提示
        """
        base_prompt = f"用户说: {utterance}"
        
        # 添加上下文信息
        if context.get('situation'):
            base_prompt += f"\n场景: {context['situation']}"
        if context.get('relationship'):
            base_prompt += f"\n关系: {context['relationship']}"
        
        # 添加分析指导
        base_prompt += """
        
        请进行以下语用分析:
        1. 识别言语行为类型（请求、陈述、提问等）
        2. 推断隐含意图
        3. 分析礼貌策略
        4. 考虑文化/语境因素
        """
        
        return base_prompt
```

#### 7.2.2 微调方法

```python
class PragmaticsFineTuner:
    """
    语用能力微调
    
    使用语用标注数据微调LLM
    """
    
    def __init__(self, base_model):
        self.model = base_model
    
    def prepare_pragmatic_dataset(self):
        """
        准备语用数据集
        
        标注类型:
        - 言语行为标签
        - 隐含意图
        - 预设
        - 礼貌等级
        """
        pragmatic_data = [
            {
                'utterance': '能帮我倒杯水吗？',
                'speech_act': 'directive',
                'intent': '请求',
                'politeness': '中等',
                'implicature': '说话者口渴'
            },
            {
                'utterance': '你的表现真是一如既往啊',
                'speech_act': 'assertive',
                'intent': '讽刺',
                'politeness': '低（负面）',
                'implicature': '对表现不满'
            },
            # ... 更多标注数据
        ]
        
        return pragmatic_data
    
    def fine_tune(self, dataset):
        """使用语用数据微调"""
        formatted_data = self._format_for_training(dataset)
        
        # 标准的指令微调流程
        # ...
```

#### 7.2.3 外部知识增强

```python
class PragmaticsKnowledgeAugmented:
    """
    外部知识增强的语用推理
    
    结合知识图谱和语用规则
    """
    
    def __init__(self, knowledge_graph):
        self.kg = knowledge_graph
        self.pragmatic_rules = self._load_pragmatic_rules()
    
    def infer_implicature(self, utterance, context):
        """
        推断话语隐含意义
        
        结合:
        1. 语用规则（Grice原则等）
        2. 世界知识（知识图谱）
        3. 上下文信息
        """
        # 1. 提取语义内容
        semantic_content = self.extract_semantics(utterance)
        
        # 2. 应用语用规则
        rule_based_implicature = self.apply_pragmatic_rules(
            semantic_content, context
        )
        
        # 3. 结合世界知识
        knowledge_based_inference = self.infer_with_knowledge(
            semantic_content, context
        )
        
        # 4. 融合结果
        final_implicature = self.fuse_inferences(
            rule_based_implicature,
            knowledge_based_inference
        )
        
        return final_implicature
```

### 7.3 评估语用能力

```python
class PragmaticsEvaluator:
    """
    语用能力评估基准
    """
    
    def __init__(self):
        self.benchmarks = {
            # 言语行为识别
            'speech_act_classification': {
                'description': '识别话语的言语行为类型',
                'metrics': ['accuracy', 'F1']
            },
            # 隐含意图识别
            'implicature_detection': {
                'description': '识别话语的隐含意图',
                'metrics': ['precision', 'recall']
            },
            # 预设消解
            'presupposition_resolution': {
                'description': '消解话语中的预设',
                'metrics': ['accuracy']
            },
            # 间接言语理解
            'indirect_speech_understanding': {
                'description': '理解间接言语行为',
                'metrics': ['success_rate']
            },
            # 讽刺识别
            'irony_detection': {
                'description': '识别反语和讽刺',
                'metrics': ['F1']
            }
        }
    
    def evaluate(self, model, test_data):
        """评估模型的语用能力"""
        results = {}
        
        for task, config in self.benchmarks.items():
            task_data = test_data[task]
            predictions = model.predict(task_data)
            
            results[task] = self.compute_metrics(
                predictions, task_data['labels'], config['metrics']
            )
        
        return results
```

---

## 八、前沿研究方向

### 8.1 计算语用学的热点问题

1. **多模态语用**：结合视觉、语音等模态的语用理解
2. **跨语言语用**：不同语言中语用策略的差异与翻译
3. **文化语用**：文化因素对语用解释的影响
4. **动态语用**：对话过程中的语用推理演变
5. **神经语用学**：大脑如何处理语用信息

### 8.2 LLM语用的未来发展

```
未来发展方向:

    ┌─────────────────┐
    │  多模态语用      │ ← 视觉-语言联合理解
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  个性化语用     │ ← 用户偏好建模
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  文化感知语用   │ ← 跨文化理解
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  可解释语用推理 │ ← 推理过程透明化
    └─────────────────┘
```

---

## 参考文献与推荐阅读

1. Austin, J. L. (1962). *How to Do Things with Words*. Oxford University Press.
2. Searle, J. R. (1969). *Speech Acts: An Essay in the Philosophy of Language*. Cambridge University Press.
3. Grice, H. P. (1975). Logic and conversation. *Syntax and Semantics*, 3, 41-58.
4. Sperber, D., & Wilson, D. (1986). *Relevance: Communication and Cognition*. Blackwell.
5. Levinson, S. C. (1983). *Pragmatics*. Cambridge University Press.
6. Brown, P., & Levinson, S. C. (1987). *Politeness: Some Universals in Language Usage*. Cambridge University Press.

---

> [!tip] 关联文档
> - [[计算语义学深度指南]]：语义学整体框架
> - [[词向量与分布式语义]]：语义表示方法
> - [[形式语义学基础]]：形式语义学理论
> - [[概念语义学详解]]：概念表示
> - [[分布式语义学详解]]：分布式语义理论
