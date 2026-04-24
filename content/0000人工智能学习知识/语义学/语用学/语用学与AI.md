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
  - 对话系统
  - 意图识别
  - 讽刺检测
  - 隐喻理解
categories:
  - 人工智能/语义学
  - 人工智能/语言理解
  - 认知科学/语用学
alias: [Pragmatics, Pragmatics in AI, 语用推理]
---

# 语用学与AI

> [!note] 文档概述
> 本文档系统探讨语用学的核心理论体系及其与人工智能的深度关联。语用学研究语言使用中的意义，包括言语行为、合作原则、关联理论等核心理论，以及这些理论在大语言模型中的实现与应用挑战。本文档将深入讨论语用学各分支在AI系统中的实现方法，并探讨当前LLM在语用理解方面的能力与局限。

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
| 间接言语 | Indirect Speech | 通过字面意义传达非字面意图 |
| 讽刺 | Irony/Sarcasm | 使用与意图相反的话语 |
| 回指消解 | Anaphora Resolution | 解决代词和指示语的指代 |
| 话题连续性 | Topic Continuity | 话语中话题的保持与转换 |

---

## 一、语用学的学科定位

### 1.1 语用学的定义与范围

语用学（Pragmatics）是研究语言使用者在特定语境中如何理解和产生意义的学科。与语义学研究字面意义不同，语用学关注的是语言在实际使用中的动态意义建构过程。

**核心研究问题**

1. **说话者意图**：语言背后想要传达的真正含义是什么？
2. **语境依赖**：意义如何随语境变化？
3. **隐性信息**：哪些信息是未明确表达但隐含的？
4. **言语效果**：语言使用产生了什么样的交际效果？
5. **推理过程**：听话者如何从字面意义推断出真实意图？

**语义学 vs 语用学的对比**

| 维度 | 语义学 | 语用学 |
|------|--------|--------|
| 研究对象 | 语言的字面意义 | 语言的使用意义 |
| 语境依赖 | 低 | 高 |
| 主观性 | 低 | 高 |
| 核心问题 | "X是什么意思" | "说X意味着什么" |
| 典型问题 | 词义、句子结构 | 意图、礼貌、讽刺 |
| 表达类型 | 明说（Explicature） | 隐含（Implicature） |
| 推理要求 | 较少 | 较多 |

**语用学的分支领域**

```
语用学学科结构
│
├── 形式语用学 (Formal Pragmatics)
│   ├── 语用逻辑 (Pragmatic Logic)
│   └── 形式语义与语用的接口
│
├── 描写语用学 (Descriptive Pragmatics)
│   ├── 言语行为 (Speech Acts)
│   ├── 指示语 (Deixis)
│   └── 预设 (Presupposition)
│
├── 社会语用学 (Social Pragmatics)
│   ├── 礼貌理论 (Politeness)
│   ├── 面子理论 (Face)
│   └── 社会语言学变异
│
├── 认知语用学 (Cognitive Pragmatics)
│   ├── 关联理论 (Relevance Theory)
│   ├── 心理语用学 (Psychopragmatics)
│   └── 语用推理 (Pragmatic Inference)
│
└── 计算语用学 (Computational Pragmatics)
    ├── 自然语言理解
    ├── 对话系统
    └── 大语言模型研究
```

### 1.2 语用学的历史渊源

**哲学根源**

语用学的哲学根源可追溯至19世纪末至20世纪初的哲学传统：

1. **Charles Sanders Peirce的指号学**
   - 符号三元论：符号（Sign）、对象（Object）、解释项（Interpretant）
   - 符号在使用中才有意义
   - 意义是动态的、基于解释的

2. **Ludwig Wittgenstein的语言游戏理论**
   - "意义即使用"（Meaning as use）
   - 语言是一系列语言游戏的集合
   - 每个游戏有其自己的规则和使用方式

3. **John Austin的言语行为理论**
   - "说话就是做事"（Doing things with words）
   - 话语不仅是描述世界，还可以改变世界
   - 区分话语行为、话语施事行为和话语施效行为

**语言学发展历程**

| 时期 | 里程碑事件 | 代表人物 |
|------|------------|----------|
| 1950s | 言语行为理论的雏形 | Austin |
| 1960s | 言语行为理论系统化 | Searle |
| 1970s | 合作原则与会话含义 | Grice |
| 1980s | 关联理论 | Sperber & Wilson |
| 1980s | 形式语用学 | Montague后继者 |
| 1990s | 礼貌理论与面子理论 | Brown & Levinson |
| 2000s | 计算语用学 | 早期NLP研究者 |
| 2010s | 神经语用学 | 认知神经科学家 |
| 2020s | LLM语用能力研究 | AI研究者 |

### 1.3 语用学与其他学科的关系

**语用学与认知科学**

语用学与认知心理学、认知神经科学有着密切的联系：

```
语用理解的认知模型

         ┌─────────────────┐
         │   话语输入      │
         └────────┬────────┘
                  ↓
         ┌─────────────────┐
         │   语义解码      │
         │  (字面意义)     │
         └────────┬────────┘
                  ↓
         ┌─────────────────┐
         │   语境整合      │
         │  (世界知识+上下文)│
         └────────┬────────┘
                  ↓
         ┌─────────────────┐
         │   语用推理      │
         │  (意图推断)     │
         └────────┬────────┘
                  ↓
         ┌─────────────────┐
         │   话语理解      │
         │  (完整意义)     │
         └─────────────────┘
```

**涉及的认知机制**

1. **工作记忆**：暂存和操作话语信息
2. **长期记忆**：激活相关知识
3. **注意力**：聚焦相关信息
4. **推理能力**：进行语用推理
5. **社会认知**：理解说话者意图

**语用学与人工智能**

AI系统需要理解自然语言，而语用理解是自然语言理解的重要组成部分：

| AI任务 | 所需语用能力 |
|--------|--------------|
| 对话系统 | 意图识别、礼貌策略、话轮转换 |
| 问答系统 | 预设消解、间接回答 |
| 机器翻译 | 文化语用、礼貌翻译 |
| 情感分析 | 讽刺检测、隐含情感 |
| 信息抽取 | 指代消解、话题跟踪 |
| 文本生成 | 礼貌生成、委婉表达 |

---

## 二、言语行为理论（Speech Act Theory）

### 2.1 Austin的言语行为三分说

约翰·奥斯汀（John Langshaw Austin）在其经典著作《如何以言行事》（How to Do Things with Words, 1962）中，首次系统性地提出了言语行为理论，将话语行为分为三个层次。

#### 2.1.1 话语行为（Locutionary Act）

话语行为指说话者发出的物理声音及其表达的具体意义，是话语的"说什么"层面：

$$
\text{Locution}(S) = \text{Phonetic}(S) + \text{Phatic}(S) + \text{Rhetic}(S)
$$

其中三个组成部分：
- **Phonetic行为**：发出语音/书写文字的物理行为
- **Phatic行为**：按照语法规则组织词语的行为
- **Rhetic行为**：使用具有特定指称和意义的词语的行为

**话语行为的完整性**

话语行为的成功完成需要满足：
1. 语音形式正确
2. 语法结构正确
3. 语义内容清晰
4. 指称对象明确

#### 2.1.2 话语施事行为（Illocutionary Act）

话语施事行为是指话语在说出的同时所执行的行为，这是言语行为理论的核心：

| 言语行为类型 | 英文名称 | 说明 | 示例 | 施事动词 |
|-------------|----------|------|------|----------|
| **断言类** | Assertives | 描述世界状态 | "猫在垫子上" | 说、声称、认为、相信 |
| **指令类** | Directives | 要求他人做某事 | "请关门" | 请、要求、命令、建议 |
| **承诺类** | Commissives | 承诺未来行动 | "我会来" | 承诺、发誓、保证、约定 |
| **表达类** | Expressives | 表达心理状态 | "我很高兴" | 感谢、祝贺、道歉、抱怨 |
| **宣告类** | Declarations | 改变世界状态 | "我宣布..." | 宣布、任命、命名、判决 |

**施事动词（Illocutionary Force Indicators）**

施事动词是明确标记言语行为类型的关键语言元素：

```python
def classify_speech_act_by_verb(utterance):
    """
    基于施事动词的言语行为分类
    
    施事动词是话语的"力量指示器"
    明确告诉听者话语应该被如何理解
    """
    speech_act_taxonomy = {
        'assertives': {
            'verbs': ['说', '陈述', '宣称', '认为', '相信', '指出', '声明', '表明', '说明'],
            'direction': 'word-to-world',
            'psychological_state': '相信为真'
        },
        'directives': {
            'verbs': ['请', '要求', '命令', '建议', '禁止', '允许', '恳求', '劝告', '指示'],
            'direction': 'world-to-word',
            'psychological_state': '想要/希望'
        },
        'commissives': {
            'verbs': ['承诺', '发誓', '保证', '约定', '发誓', '许诺', '立誓'],
            'direction': 'world-to-word',
            'psychological_state': '意图'
        },
        'expressives': {
            'verbs': ['感谢', '祝贺', '道歉', '抱怨', '哀悼', '欢迎', '吊唁', '表扬', '批评'],
            'direction': 'none',
            'psychological_state': '表达'
        },
        'declarations': {
            'verbs': ['宣布', '任命', '命名', '判决', '任命', '任命', '认定', '裁定'],
            'direction': 'self-verifying',
            'psychological_state': '无特定'
        }
    }
    
    for act_type, info in speech_act_taxonomy.items():
        for verb in info['verbs']:
            if verb in utterance:
                return {
                    'type': act_type,
                    'verb': verb,
                    'description': info
                }
    
    return None


class SpeechActClassifier:
    """
    综合言语行为分类器
    
    结合多种特征进行分类：
    1. 施事动词
    2. 句法结构
    3. 语调模式
    4. 语境信息
    """
    
    def __init__(self):
        self.taxonomy = {
            'assertives': self._is_assertive,
            'directives': self._is_directive,
            'commissives': self._is_commissive,
            'expressives': self._is_expressive,
            'declarations': self._is_declaration
        }
    
    def classify(self, utterance, context=None):
        """综合分类"""
        scores = {}
        
        for act_type, classifier in self.taxonomy.items():
            score = classifier(utterance, context)
            scores[act_type] = score
        
        # 返回得分最高的类型
        return max(scores, key=scores.get)
    
    def _is_assertive(self, utterance, context):
        """判断是否为断言类"""
        score = 0
        
        # 施事动词
        assertive_verbs = ['说', '认为', '相信', '指出', '表明']
        for verb in assertive_verbs:
            if verb in utterance:
                score += 0.4
        
        # 句法特征：陈述句
        if utterance.endswith('。') or utterance.endswith('.'):
            score += 0.2
        
        # 内容特征：包含事实性描述
        factual_markers = ['是', '在', '有', '会', '已经']
        for marker in factual_markers:
            if marker in utterance:
                score += 0.1
        
        return min(score, 1.0)
    
    def _is_directive(self, utterance, context):
        """判断是否为指令类"""
        score = 0
        
        # 施事动词
        directive_verbs = ['请', '要求', '命令', '建议', '禁止']
        for verb in directive_verbs:
            if verb in utterance:
                score += 0.4
        
        # 句法特征：疑问句、祈使句
        if utterance.endswith('?') or utterance.endswith('？'):
            score += 0.2
        
        # 礼貌标记
        politeness_markers = ['请', '能不能', '可以...吗', '麻烦']
        for marker in politeness_markers:
            if marker in utterance:
                score += 0.3
        
        return min(score, 1.0)
    
    def _is_commissive(self, utterance, context):
        """判断是否为承诺类"""
        score = 0
        
        # 施事动词
        commissive_verbs = ['承诺', '保证', '发誓', '许诺', '答应']
        for verb in commissive_verbs:
            if verb in utterance:
                score += 0.5
        
        # 时态：将来时
        future_markers = ['会', '将要', '一定', '肯定']
        for marker in future_markers:
            if marker in utterance:
                score += 0.2
        
        return min(score, 1.0)
    
    def _is_expressive(self, utterance, context):
        """判断是否为表达类"""
        score = 0
        
        # 施事动词
        expressive_verbs = ['感谢', '祝贺', '道歉', '欢迎', '吊唁']
        for verb in expressive_verbs:
            if verb in utterance:
                score += 0.4
        
        # 情感标记
        emotion_markers = ['高兴', '抱歉', '遗憾', '感谢', '祝贺']
        for marker in emotion_markers:
            if marker in utterance:
                score += 0.3
        
        return min(score, 1.0)
    
    def _is_declaration(self, utterance, context):
        """判断是否为宣告类"""
        score = 0
        
        # 施事动词
        declaration_verbs = ['宣布', '任命', '命名', '判决', '裁定']
        for verb in declaration_verbs:
            if verb in utterance:
                score += 0.5
        
        # 权威标记
        authority_markers = ['我', '代表', '以...名义']
        for marker in authority_markers:
            if marker in utterance:
                score += 0.2
        
        return min(score, 1.0)
```

#### 2.1.3 话语施效行为（Perlocutionary Act）

话语施效行为指话语对听者产生的效果，是话语的"做什么"层面：

$$
\text{Perlocutionary Effect}(U \text{ on } H) = \text{某种心理或行为变化}
$$

**常见施效效果类型**

| 效果类型 | 说明 | 示例 |
|----------|------|------|
| **认识效果** | 改变听者的知识或信念 | 解释使对方理解新概念 |
| **情感效果** | 引起听者的情感反应 | 安慰使对方感到温暖 |
| **行为效果** | 促使听者采取行动 | 命令使对方执行任务 |
| **说服效果** | 改变听者的态度 | 论证使对方同意观点 |
| **安慰效果** | 减轻听者的负面情绪 | 鼓励使对方振作 |

**三种言语行为的区分示例**

```
话语: "外面在下雨。"

1. 话语行为（Locutionary）:
   - 发出语音：yán wài zài xià yǔ
   - 语法结构：陈述句
   - 语义内容：陈述一个天气事实

2. 话语施事行为（Illocutionary）:
   - 行为类型：断言（描述状态）
   - 可能隐含意图：暗示/警告
   - 根据语境可能变为请求（暗示对方带伞）

3. 话语施效行为（Perlocutionary）:
   - 听者意识到天气状况
   - 听者决定带伞或不出门
   - 听者感受到关心（如果说话者是在意听者）
```

### 2.2 Searle的言语行为分类体系

约翰·塞尔（John Searle）在Austin的基础上，对言语行为进行了更系统的分类和发展。

#### 2.2.1 分类体系详解

**塞尔分类法的核心要素**

每种言语行为都有四个构成要素：

1. **命题内容**（Propositional Content）：话语中关于世界的陈述
2. **心理状态**（Psychological State）：说话者对命题内容的态度
3. **命题方向**（Direction of Fit）：词语与世界的关系
4. **真诚条件**（Sincerity Condition）：实施言语行为的必要心理状态

```python
class SearleSpeechActTaxonomy:
    """
    Searle言语行为分类体系
    
    每种言语行为都有其特征性的构成要素
    """
    
    TAXONOMY = {
        'assertives': {
            'description': '断言或描述世界状态',
            'propositional_content': '命题可以是真或假',
            'psychological_state': '相信',
            'direction_of_fit': 'word-to-world',  # 词语去符合世界
            'sincerity_condition': '说话者必须相信命题为真',
            'examples': ['陈述', '断言', '宣称', '结论', '通知'],
            'english_verbs': ['assert', 'claim', 'believe', 'conclude', 'state']
        },
        'directives': {
            'description': '试图使听者做某事',
            'propositional_content': '听者未来的行动',
            'psychological_state': '想要/希望',
            'direction_of_fit': 'world-to-word',  # 世界被要求符合词语
            'sincerity_condition': '说话者想要听者做某事',
            'examples': ['请求', '命令', '建议', '询问', '询问'],
            'english_verbs': ['ask', 'order', 'command', 'request', 'suggest', 'advise']
        },
        'commissives': {
            'description': '承诺说话者未来行动',
            'propositional_content': '说话者未来的行动',
            'psychological_state': '意图',
            'direction_of_fit': 'world-to-word',  # 世界被要求符合承诺
            'sincerity_condition': '说话者打算做某事',
            'examples': ['承诺', '发誓', '保证', '约定', '发誓'],
            'english_verbs': ['promise', 'swear', 'vow', 'pledge', 'guarantee', 'commit']
        },
        'expressives': {
            'description': '表达心理状态',
            'propositional_content': '以说话者为中心',
            'psychological_state': '表达',
            'direction_of_fit': 'none',  # 无方向性
            'sincerity_condition': '说话者确实有该心理状态',
            'examples': ['感谢', '道歉', '祝贺', '哀悼', '欢迎'],
            'english_verbs': ['thank', 'apologize', 'congratulate', 'condole', 'welcome']
        },
        'declarations': {
            'description': '改变世界状态',
            'propositional_content': '自我实现',
            'psychological_state': '无特定要求',
            'direction_of_fit': 'self-verifying',  # 言语本身改变世界
            'sincerity_condition': '无特定要求（但需要适当的制度语境）',
            'examples': ['任命', '命名', '宣布', '祝福', '宣战'],
            'english_verbs': ['declare', 'appoint', 'name', 'bless', 'christen', 'resign']
        }
    }
    
    @classmethod
    def get_taxonomy_info(cls, act_type):
        """获取特定言语行为类型的详细信息"""
        return cls.TAXONOMY.get(act_type, None)
    
    @classmethod
    def print_taxonomy(cls):
        """打印完整的分类体系"""
        for act_type, info in cls.TAXONOMY.items():
            print(f"\n{'='*60}")
            print(f"类型: {act_type.upper()}")
            print(f"描述: {info['description']}")
            print(f"命题内容: {info['propositional_content']}")
            print(f"心理状态: {info['psychological_state']}")
            print(f"命题方向: {info['direction_of_fit']}")
            print(f"真诚条件: {info['sincerity_condition']}")
            print(f"示例: {', '.join(info['examples'])}")
```

#### 2.2.2 言语行为的层级结构

塞尔进一步提出了言语行为的层级组织：

```
言语行为层级结构

                    言语行为
                       │
         ┌─────────────┼─────────────┐
         │             │             │
     基本行为      更高层级       更复杂
         │             │             │
    ┌────┴────┐   ┌────┴────┐    ┌────┴────┐
    │  断言   │   │  告知   │    │  解释   │
    │  请求   │   │  解释   │    │  辩护   │
    │  承诺   │   │  说服   │    │  批评   │
    └─────────┘   └─────────┘    └─────────┘
```

### 2.3 间接言语行为

间接言语行为（Indirect Speech Acts）是言语行为理论的重要扩展，指通过一种言语行为来间接实施另一种言语行为。

#### 2.3.1 间接言语的类型

**直接 vs 间接**

```
直接言语行为:
句子形式 → 言语行为类型
疑问句 → 提问
祈使句 → 请求

间接言语行为:
句子形式 → 言语行为类型
疑问句 → 请求（而非提问）
祈使句 → 断言（而非请求）
```

**常见间接言语类型**

| 句子形式 | 直接功能 | 间接功能 | 示例 |
|----------|----------|----------|------|
| 疑问句 | 提问 | 请求 | "你能帮我关门吗？" |
| 疑问句 | 提问 | 提议 | "要不要一起吃饭？" |
| 疑问句 | 提问 | 断言 | "你知道现在几点了吗？" |
| 陈述句 | 断言 | 请求 | "门开着。" |
| 陈述句 | 断言 | 警告 | "地面很滑。" |

#### 2.3.2 间接言语的推理过程

```python
class IndirectSpeechActResolver:
    """
    间接言语行为解析器
    
    听话者需要通过推理理解间接言语的真实意图
    """
    
    def __init__(self):
        # 间接言语模式库
        self.indirect_patterns = {
            # 疑问句 → 请求
            'question_to_request': {
                'patterns': [
                    r'你能.*吗',
                    r'你可以.*吗',
                    r'能不能.*',
                    r'可不可以.*',
                    r'帮我.*',
                    r'帮我.*一下'
                ],
                'inference': self._infer_request_from_question
            },
            
            # 陈述句 → 请求
            'statement_to_request': {
                'patterns': [
                    r'门.*开',
                    r'窗户.*开',
                    r'灯.*开',
                    r'很.*（描述某种状态）'
                ],
                'inference': self._infer_request_from_statement
            },
            
            # 疑问句 → 断言
            'question_to_assertion': {
                'patterns': [
                    r'你知道.*吗',
                    r'你.*难道.*',
                    r'.*不是.*吗'
                ],
                'inference': self._infer_assertion_from_question
            }
        }
    
    def resolve(self, utterance, context):
        """
        解析间接言语行为的真实意图
        
        Args:
            utterance: 话语
            context: 对话上下文
        
        Returns:
            {'direct_act': ..., 'indirect_act': ..., 'confidence': ...}
        """
        # 1. 确定直接言语行为
        direct_act = self._classify_direct_act(utterance)
        
        # 2. 尝试识别间接意图
        indirect_act = self._detect_indirect_intent(utterance, context)
        
        # 3. 计算置信度
        if indirect_act:
            return {
                'direct_act': direct_act,
                'indirect_act': indirect_act,
                'confidence': 0.8,  # 间接言语有一定置信度
                'reasoning': '使用了间接言语模式'
            }
        else:
            return {
                'direct_act': direct_act,
                'indirect_act': direct_act,
                'confidence': 1.0,
                'reasoning': '直接言语行为'
            }
    
    def _classify_direct_act(self, utterance):
        """分类直接言语行为"""
        if utterance.endswith('?') or utterance.endswith('？'):
            return 'question'
        elif utterance.endswith('!'):
            return 'exclamation'
        else:
            return 'statement'
    
    def _detect_indirect_intent(self, utterance, context):
        """检测间接意图"""
        for pattern_type, pattern_info in self.indirect_patterns.items():
            for pattern in pattern_info['patterns']:
                import re
                if re.search(pattern, utterance):
                    inference_func = pattern_info['inference']
                    return inference_func(utterance, context)
        
        return None
    
    def _infer_request_from_question(self, utterance, context):
        """从疑问句推断请求意图"""
        # "你能帮我关门吗？" → 请求关门
        # 提取关键信息
        return {
            'act_type': 'request',
            'content': self._extract_request_content(utterance),
            'politeness': 'high'  # 疑问形式的请求通常更礼貌
        }
    
    def _infer_request_from_statement(self, utterance, context):
        """从陈述句推断请求意图"""
        # "门开着" → 请求关门/关窗等
        return {
            'act_type': 'request',
            'content': self._infer_appropriate_action(utterance, context),
            'politeness': 'medium'
        }
    
    def _infer_assertion_from_question(self, utterance, context):
        """从疑问句推断断言意图"""
        # "你知道他是谁吗？" → 说话者想告诉听者某事
        return {
            'act_type': 'assertive',
            'content': self._extract_assertion_content(utterance),
            'intent': 'informative'
        }
```

### 2.4 言语行为与AI对话系统

言语行为理论在AI对话系统中有广泛应用。

#### 2.4.1 对话意图识别系统

```python
class DialogueIntentClassifier:
    """
    对话系统中的意图识别
    
    基于言语行为理论设计意图分类体系
    结合领域知识进行细粒度分类
    """
    
    def __init__(self, domain='general'):
        self.domain = domain
        self.intent_taxonomy = self._build_taxonomy(domain)
    
    def _build_taxonomy(self, domain):
        """构建领域特定的意图分类体系"""
        base_taxonomy = {
            # 信息获取类
            'query': {
                'patterns': ['询问', '查询', '想知道', '什么', '怎么', '为什么', '多少'],
                'sub_intents': {
                    'fact_query': ['是什么', '谁在', '哪有'],
                    'method_query': ['怎么', '如何', '怎样'],
                    'reason_query': ['为什么', '为何', '什么原因'],
                    'quantity_query': ['多少', '几个', '多大']
                }
            },
            
            # 信息确认类
            'confirmation': {
                'patterns': ['对吗', '是吗', '对不对', '是不是'],
                'sub_intents': {
                    'yes_no': ['是吗', '对吗'],
                    'clarification': ['你的意思是', '具体指'],
                    'correction': ['不对', '不是', '错了']
                }
            },
            
            # 任务执行类
            'request': {
                'patterns': ['帮我', '请', '能不能', '请帮我', '帮我做'],
                'sub_intents': {
                    'action_request': ['帮我', '帮我做', '请帮我'],
                    'information_request': ['告诉我', '查一下', '找一下'],
                    'service_request': ['预订', '购买', '安排']
                }
            },
            
            # 建议建议类
            'suggestion': {
                'patterns': ['建议', '可以试试', '不妨', '不如', '要是'],
                'sub_intents': {
                    'advice': ['建议', '应该', '最好'],
                    'recommendation': ['推荐', '介绍', '看看'],
                    'alternative': ['不如', '或者', '要么']
                }
            },
            
            # 社会交互类
            'social': {
                'patterns': ['你好', '早上好', 'hi', 'hello', '谢谢', '再见'],
                'sub_intents': {
                    'greeting': ['你好', 'hi', 'hello', '早上好', '晚上好'],
                    'farewell': ['再见', '拜拜', '回头见', '下次见'],
                    'thanks': ['谢谢', '感谢', '多谢', '麻烦你了'],
                    'apology': ['抱歉', '对不起', '不好意思']
                }
            },
            
            # 情感表达类
            'emotion': {
                'patterns': ['不错', '很好', '糟糕', '不满', '失望'],
                'sub_intents': {
                    'positive_emotion': ['不错', '很好', '太棒了', '厉害'],
                    'negative_emotion': ['糟糕', '不满', '失望', '生气'],
                    'neutral_emotion': ['一般', '还行', '凑合']
                }
            }
        }
        
        return base_taxonomy
    
    def predict_intent(self, utterance):
        """
        预测用户意图
        
        Args:
            utterance: 用户话语
        
        Returns:
            {'intent': ..., 'confidence': ..., 'sub_intent': ...}
        """
        scores = {}
        
        for intent, info in self.intent_taxonomy.items():
            score = 0
            for pattern in info['patterns']:
                if pattern in utterance:
                    score += 1
            
            if score > 0:
                scores[intent] = score
        
        if not scores:
            return {
                'intent': 'unknown',
                'confidence': 0,
                'sub_intent': None
            }
        
        # 返回得分最高的意图
        top_intent = max(scores, key=scores.get)
        
        # 进一步识别子意图
        sub_intent = self._classify_sub_intent(
            utterance, 
            self.intent_taxonomy[top_intent]['sub_intents']
        )
        
        return {
            'intent': top_intent,
            'confidence': scores[top_intent] / max(scores.values()),
            'sub_intent': sub_intent,
            'all_scores': scores
        }
    
    def _classify_sub_intent(self, utterance, sub_intents):
        """识别子意图"""
        sub_scores = {}
        
        for sub_intent, patterns in sub_intents.items():
            score = sum(1 for p in patterns if p in utterance)
            if score > 0:
                sub_scores[sub_intent] = score
        
        if sub_scores:
            return max(sub_scores, key=sub_scores.get)
        
        return None
```

---

## 三、合作原则与会话含义

### 3.1 Grice的合作原则

保罗·格赖斯（Herbert Paul Grice）在其著名的"逻辑与会话"（Logic and Conversation, 1975）演讲中提出，交际遵循一种默契的"合作原则"（Cooperative Principle）。

> 使你的话语在其所发生的阶段符合当前交谈的期望。

#### 3.1.1 四大量准则

Grice的合作原则包含四个范畴，每个范畴下有具体的准则：

**量的准则（Quantity Maxim）**

```
准则内容：
1. 所说的话应包含当前交谈目的所需的信息（信息充分）
2. 所说的话不应包含超出所需的信息（信息不过量）
```

| 违反情况 | 示例 | 会话含义 |
|----------|------|----------|
| 信息不足 | 只说部分事实 | 暗示还有其他信息 |
| 信息过量 | 详细描述已知事实 | 暗示强调或强调重要性 |

**质的准则（Quality Maxim）**

```
准则内容：
1. 不要说你认为是虚假的话
2. 不要说你缺乏足够证据的话
```

| 违反情况 | 示例 | 会话含义 |
|----------|------|----------|
| 说假话 | 说"我知道"但实际不知道 | 误导、欺骗 |
| 缺乏证据 | 说"他一定..."但不确定 | 推测、或许 |

**关系准则（Relevance Maxim）**

```
准则内容：
要有关联性（所说的话要与交谈目的相关）
```

| 违反情况 | 示例 | 会话含义 |
|----------|------|----------|
| 不相关 | 答非所问 | 转移话题、回避问题 |

**方式准则（Manner Maxim）**

```
准则内容：
1. 避免表达模糊
2. 避免歧义
3. 简洁（避免不必要的冗长）
4. 有条理
```

| 违反情况 | 示例 | 会话含义 |
|----------|------|----------|
| 模糊 | 含糊其辞 | 有所隐瞒 |
| 冗长 | 过度解释 | 强调、辩护 |
| 无条理 | 乱说一气 | 表达混乱 |

```
合作原则示意图:

            交际目标
                ↓
        ┌─────────────────┐
        │                 │
        │  合作原则       │
        │                 │
        ├─────────────────┤
        │  量的准则      │ ← 提供适量信息
        │  质的准则      │ ← 说真话
        │  关系的准则    │ ← 说相关的
        │  方式的准则    │ ← 清楚地说
        └─────────────────┘
                ↓
            有效交际
```

#### 3.1.2 会话含义的类型

**标准会话含义（Standard Implicature）**

标准会话含义直接遵循准则产生，是听话者根据准则做出的正常推理：

```
场景：A在找书
A: "书在桌子上。"
B: "桌子？"

B的回应违反了方式准则（含糊）
→ 标准含义：B不确定指的是哪张桌子
→ B请求澄清
```

**特殊会话含义（Particularized Implicature）**

特殊会话含义通过**违反**某一准则而产生，需要结合语境才能理解：

```
场景：A问B关于C借钱的事
A: "C能借我500块吗？"
B: "他最近手头有点紧。"

B的回答：
- 违反了质的准则？没有直接回答"能"或"不能"
- 违反了量的准则？没有提供充分信息

→ 特殊含义：C可能没法借
→ B在暗示拒绝
```

**会话含义的特征**

1. **可取消性（Cancelability）**：可以在不矛盾的情况下被取消
   - 原句："他看起来很有钱"（暗示他有钱）
   - 取消后："他看起来很有钱，但其实他是装的。"

2. **不可分离性（Non-detachability）**：与会话含义相关的内容无法通过改述去除
   - 原句："他是个朋友"（暗示亲密关系）
   - 即使改述："他是我的朋友"，含义仍然保留

3. **计算性（Calculability）**：含义可以通过推理计算得出

4. **不确定性（Indeterminacy）**：同一话语可能有多种含义

### 3.2 会话含义的计算模型

```python
class ConversationalImplicatureCalculator:
    """
    会话含义计算器
    
    基于Grice的合作原则计算隐含意义
    """
    
    def __init__(self):
        self.maxims = {
            'quantity': {
                'name': '量的准则',
                'weight': 1.0,
                'subs': {
                    'sufficient': '信息充分',
                    'not_excessive': '信息不过量'
                }
            },
            'quality': {
                'name': '质的准则',
                'weight': 1.2,  # 真实性权重更高
                'subs': {
                    'truthful': '说真话',
                    'evidenced': '有证据'
                }
            },
            'relation': {
                'name': '关系准则',
                'weight': 1.1,
                'subs': {
                    'relevant': '相关'
                }
            },
            'manner': {
                'name': '方式准则',
                'weight': 0.9,
                'subs': {
                    'clear': '清楚',
                    'unambiguous': '无歧义',
                    'concise': '简洁',
                    'ordered': '有条理'
                }
            }
        }
    
    def calculate_implicature(self, utterance, context, expected_response=None):
        """
        计算违反合作原则产生的会话含义
        
        Args:
            utterance: 实际话语
            context: 对话上下文（包含交际目标、说话者角色等）
            expected_response: 符合合作原则的预期回答
        
        Returns:
            {'implicatures': [...], 'violations': [...], 'reasoning': ...}
        """
        violations = []
        implicatures = []
        
        # 检测各准则的违反
        for maxim_key, maxim_info in self.maxims.items():
            violation = self._check_violation(
                utterance, maxim_key, expected_response, context
            )
            
            if violation['violated']:
                violations.append(violation)
                
                # 计算该违反产生的隐含意义
                implicature = self._derive_implicature(
                    utterance, violation, context
                )
                implicatures.append(implicature)
        
        return {
            'violations': violations,
            'implicatures': implicatures,
            'reasoning': self._generate_reasoning(violations, implicatures)
        }
    
    def _check_violation(self, utterance, maxim_key, expected_response, context):
        """检测特定准则是否被违反"""
        utterance_len = len(utterance)
        
        violation_result = {
            'maxim': maxim_key,
            'maxim_name': self.maxims[maxim_key]['name'],
            'violated': False,
            'reason': None,
            'strength': 0.0
        }
        
        if maxim_key == 'quantity':
            # 量的准则检测
            if expected_response:
                expected_len = len(expected_response)
                
                # 信息不足
                if utterance_len < expected_len * 0.5:
                    violation_result['violated'] = True
                    violation_result['reason'] = '提供的信息少于交谈所需'
                    violation_result['strength'] = 0.8
                
                # 信息过量
                elif utterance_len > expected_len * 2:
                    violation_result['violated'] = True
                    violation_result['reason'] = '提供了超出交谈所需的信息'
                    violation_result['strength'] = 0.6
        
        elif maxim_key == 'quality':
            # 质的准则检测
            uncertainty_markers = ['可能', '大概', '也许', '不确定', 'not sure']
            false_markers = ['假装', '装作', 'pretend']
            
            for marker in uncertainty_markers:
                if marker in utterance.lower():
                    violation_result['violated'] = True
                    violation_result['reason'] = '对信息真实性不确定'
                    violation_result['strength'] = 0.7
                    break
            
            for marker in false_markers:
                if marker in utterance:
                    violation_result['violated'] = True
                    violation_result['reason'] = '可能不是真话'
                    violation_result['strength'] = 0.9
                    break
        
        elif maxim_key == 'relation':
            # 关系准则检测
            if expected_response:
                # 检查话语是否相关
                relevance_score = self._compute_relevance(
                    utterance, expected_response, context
                )
                
                if relevance_score < 0.3:
                    violation_result['violated'] = True
                    violation_result['reason'] = '话语与交谈目标不相关'
                    violation_result['strength'] = 0.8
        
        elif maxim_key == 'manner':
            # 方式准则检测
            ambiguity_markers = ['意思', '某种程度上', ' kind of', ' sort of']
            for marker in ambiguity_markers:
                if marker in utterance.lower():
                    violation_result['violated'] = True
                    violation_result['reason'] = '表达模糊'
                    violation_result['strength'] = 0.5
                    break
        
        return violation_result
    
    def _derive_implicature(self, utterance, violation, context):
        """推导隐含意义"""
        maxim = violation['maxim']
        strength = violation['strength']
        
        # 隐含意义映射
        implicature_map = {
            'quantity': {
                'insufficient': {
                    'implicature': '说话者有意隐瞒部分信息',
                    'strategy': '信息最小化暗示',
                    'suggested_intent': '暗示、委婉拒绝'
                },
                'excessive': {
                    'implicature': '说话者在强调或补充细节',
                    'strategy': '信息最大化暗示',
                    'suggested_intent': '强调、辩护、证明'
                }
            },
            'quality': {
                'uncertain': {
                    'implicature': '说话者对信息不确定',
                    'strategy': '不确定性暗示',
                    'suggested_intent': '保持谨慎、避免责任'
                },
                'untruthful': {
                    'implicature': '话语有其他隐含意图',
                    'strategy': '误导性暗示',
                    'suggested_intent': '欺骗、讽刺、保护'
                }
            },
            'relation': {
                'irrelevant': {
                    'implicature': '话语转移了话题',
                    'strategy': '回避性暗示',
                    'suggested_intent': '回避、转移话题、隐瞒'
                }
            },
            'manner': {
                'unclear': {
                    'implicature': '说话者故意模糊表达',
                    'strategy': '模糊性暗示',
                    'suggested_intent': '保留面子、委婉表达'
                }
            }
        }
        
        # 获取具体的隐含意义
        if maxim in implicature_map:
            sub_implicatures = implicature_map[maxim]
            # 选择匹配的子类型
            for sub_type, info in sub_implicatures.items():
                if sub_type in violation.get('reason', '').lower():
                    return {
                        'type': 'conversational_implicature',
                        'implicature': info['implicature'],
                        'strategy': info['strategy'],
                        'suggested_intent': info['suggested_intent'],
                        'confidence': strength
                    }
        
        return {
            'type': 'conversational_implicature',
            'implicature': '无法确定具体隐含意义',
            'confidence': 0.3
        }
    
    def _compute_relevance(self, utterance, expected, context):
        """计算话语相关性"""
        # 简化的相关性计算
        # 实际应用中可以使用更复杂的NLP方法
        
        expected_keywords = set(self._extract_keywords(expected))
        utterance_keywords = set(self._extract_keywords(utterance))
        
        if not expected_keywords:
            return 0.5
        
        overlap = len(expected_keywords & utterance_keywords)
        relevance = overlap / len(expected_keywords)
        
        return relevance
    
    def _extract_keywords(self, text):
        """提取关键词"""
        # 简化的关键词提取
        stopwords = {'的', '了', '在', '是', '我', '你', '他', '她', '它', '的', '有', '和'}
        words = [w for w in text if w not in stopwords and len(w) > 1]
        return words
    
    def _generate_reasoning(self, violations, implicatures):
        """生成推理过程说明"""
        if not violations:
            return "话语遵循了合作原则，没有产生特殊的会话含义。"
        
        reasoning_parts = []
        
        for v, im in zip(violations, implicatures):
            reasoning_parts.append(
                f"违反{self.maxims[v['maxim']]['name']}（{v['reason']}），"
                f"推断出隐含意义：{im['implicature']}"
            )
        
        return "；".join(reasoning_parts)
```

### 3.3 礼貌原则

杰弗里·利奇（Geoffrey Leech）在《语用学原则》（Principles of Pragmatics, 1983）中提出了礼貌原则（Politeness Principle），作为Grice合作原则的重要补充：

> 最小限度地让别人受损，最大限度地让别人得益。

#### 3.3.1 礼貌策略的层级结构

布朗和列文森（Brown & Levinson）在1987年提出了著名的礼貌策略模型：

```python
class PolitenessModel:
    """
    礼貌语用模型
    
    基于Brown和Levinson的面子理论
    面子威胁行为与礼貌策略的选择
    """
    
    def __init__(self):
        self.politeness_strategies = {
            'bald_on_record': {
                'name': '直接表达（无修饰）',
                'description': '不加任何修饰的直接表达',
                'when_to_use': '紧急情况、亲近关系、权利不对称'
            },
            'positive_politeness': {
                'name': '积极礼貌',
                'description': '强调与听者的共同点，满足积极面子',
                'when_to_use': '需要建立友好关系、请求与听者利益一致'
            },
            'negative_politeness': {
                'name': '消极礼貌',
                'description': '给听者留面子空间，承认其自主性',
                'when_to_use': '陌生人、不确定是否被允许、面子威胁较大'
            },
            'off_record': {
                'name': '间接表达（可否认）',
                'description': '间接表达，留下否认的空间',
                'when_to_use': '避免直接责任、高风险面子威胁'
            }
        }
        
        # 面子威胁行为的严重程度计算因素
        self.power_factors = {
            'speaker_to_hearer': 0,   # 说话者对听者的权力
            'hearer_to_speaker': 0,    # 听者对说话者的权力
            'social_distance': 0.5,     # 社会距离（0-1）
            'cultural_factor': 1.0      # 文化因素
        }
    
    def calculate_face_threat(self, utterance_type, context):
        """
        计算面子威胁程度
        
        Args:
            utterance_type: 话语类型（请求、道歉、批评等）
            context: 语境信息
        
        Returns:
            threat_level: 0.0 - 1.0 的面子威胁程度
        """
        # 基础威胁值
        base_threats = {
            'request': 0.7,
            'suggestion': 0.4,
            'criticism': 0.8,
            'apology': 0.3,
            'complaint': 0.75,
            'question': 0.2,
            'assertion': 0.1
        }
        
        base_threat = base_threats.get(utterance_type, 0.5)
        
        # 计算修正因子
        power_diff = self._compute_power_diff(context)
        social_distance = context.get('social_distance', 0.5)
        
        # 面子威胁公式
        threat = base_threat * (1 + power_diff) * (1 + social_distance) / 2
        
        return min(threat, 1.0)
    
    def _compute_power_diff(self, context):
        """计算权力差异"""
        speaker_power = context.get('speaker_power', 0.5)
        hearer_power = context.get('hearer_power', 0.5)
        
        return abs(speaker_power - hearer_power)
    
    def select_strategy(self, utterance_type, context):
        """
        选择礼貌策略
        
        基于面子威胁程度选择合适的策略
        """
        threat_level = self.calculate_face_threat(utterance_type, context)
        
        # 根据威胁程度选择策略
        if threat_level < 0.3:
            return self.politeness_strategies['bald_on_record']
        elif threat_level < 0.6:
            return self.politeness_strategies['positive_politeness']
        elif threat_level < 0.8:
            return self.politeness_strategies['negative_politeness']
        else:
            return self.politeness_strategies['off_record']
    
    def polite_transform(self, request):
        """
        将直接请求转化为礼貌表达
        
        示例各种礼貌层级的转换
        """
        transformations = {
            # 直接 → 积极礼貌
            'imperative': [
                '我们一起...怎么样？',
                '我想和你一起...',
                '让...一起吧！'
            ],
            
            # 直接 → 消极礼貌
            'imperative': [
                '不知道您能否...',
                '不知道...是否可以',
                '如果您不介意的话...',
                '不知道会不会太麻烦...'
            ],
            
            # 直接 → 间接
            'imperative': [
                '你觉得...怎么样？',
                '可能...吗？',
                '好像需要...'
            ]
        }
        
        return transformations


class PolitenessGenerator:
    """
    礼貌表达生成器
    
    根据语境生成不同礼貌层级的表达
    """
    
    def __init__(self):
        self.politeness_markers = {
            'positive': {
                'chinese': ['我们', '一起', '好不好', '好吗', '很开心'],
                'english': ['we', 'together', 'how about', 'let us']
            },
            'negative': {
                'chinese': ['不知道', '可能', '如果', '方便的话', '不好意思'],
                'english': ['I wonder', 'perhaps', 'if you could', 'would it be possible']
            },
            'hedging': {
                'chinese': ['大概', '也许', '可能', '某种程度上'],
                'english': ['maybe', 'perhaps', 'might', 'possibly']
            }
        }
    
    def generate(self, base_request, politeness_level, language='chinese'):
        """
        生成礼貌表达
        
        Args:
            base_request: 基本请求内容
            politeness_level: 礼貌层级 ('low', 'medium', 'high')
            language: 语言
        
        Returns:
            polite_request: 礼貌化后的请求
        """
        if politeness_level == 'low':
            return base_request
        
        markers = self.politeness_markers
        
        if politeness_level == 'medium':
            # 添加消极礼貌标记
            marker = markers['negative'][language][0]
            return f"{marker}，{base_request}"
        
        elif politeness_level == 'high':
            # 添加多种礼貌策略
            marker1 = markers['negative'][language][0]
            marker2 = markers['hedging'][language][1] if language == 'english' else markers['hedging'][language][0]
            
            if language == 'chinese':
                return f"{marker1}，{marker2}，{base_request}，{markers['negative'][language][2]}"
            else:
                return f"{marker1} {base_request}."
```

---

## 四、关联理论（Relevance Theory）

### 4.1 理论概述

丹·斯珀伯（Dan Sperber）和戴尔德丽·威尔逊（Deirdre Wilson）在其1986年的标志性著作《关联性：交际与认知》（Relevance: Communication and Cognition）中提出了关联理论，这是一个基于认知科学的语用学理论。

> 人类的交际由对最大关联（maximal relevance）的期望驱动。

#### 4.1.1 核心原则

**认知关联原则（Cognitive Principle of Relevance）**

> 人类的认知倾向于最大化关联。

这意味着人类注意力自动聚焦于最相关的信息：

- 关联性高的事物更容易被注意
- 关联性高的信息更容易被记住
- 关联性高的推理更容易被接受

**交际关联原则（Communicative Principle of Relevance）**

> 每个话语都传递最大关联的假定。

这意味着每个说话者在交际时都在假设：

- 话语值得听者注意
- 话语以最小努力获取最大认知效果

**关联性的定义**

关联理论将关联性定义为：

$$
\text{关联性} = \frac{\text{认知效果（Positive Cognitive Effects）}}{\text{处理努力（Processing Effort）}}
$$

**认知效果包括：**

1. **新信息的整合**：新信息与已有知识结合
2. **假设的强化**：增强已有假设的可信度
3. **假设的削弱**：降低已有假设的可信度
4. **假设的消除**：排除某些可能性

**处理努力包括：**

1. **解码努力**：理解语言形式
2. **语境查找努力**：激活相关语境
3. **推理努力**：进行语用推理
4. **记忆努力**：维持工作记忆

### 4.2 关联理论的形式化框架

```python
class RelevanceTheoreticProcessor:
    """
    关联理论处理器
    
    实现基于关联原则的语用推理
    """
    
    def __init__(self):
        self.context_beliefs = {}      # 语境信念库
        self.entailed_propositions = []  # 蕴含命题
        self.relevance_threshold = 0.5  # 关联性阈值
    
    def process_utterance(self, utterance, context=None):
        """
        处理话语，计算关联性
        
        步骤:
        1. 命题识别 - 提取话语的命题内容
        2. 语境扩展 - 激活相关语境假设
        3. 认知效果计算 - 评估新信息的影响
        4. 处理努力估计 - 评估理解难度
        5. 关联性判断 - 计算最终关联值
        """
        # 1. 命题识别
        proposition = self.extract_proposition(utterance)
        
        # 2. 语境扩展
        enriched_context = self.expand_context(context, proposition)
        
        # 3. 计算认知效果
        cognitive_effects = self.compute_effects(proposition, enriched_context)
        
        # 4. 估计处理努力
        processing_effort = self.estimate_processing_effort(
            utterance, proposition, enriched_context
        )
        
        # 5. 计算关联性
        if processing_effort > 0:
            relevance = cognitive_effects / processing_effort
        else:
            relevance = float('inf')
        
        # 判断是否达到关联阈值
        is_relevant = relevance >= self.relevance_threshold
        
        return {
            'proposition': proposition,
            'cognitive_effects': cognitive_effects,
            'processing_effort': processing_effort,
            'relevance': relevance,
            'is_relevant': is_relevant,
            'interpretation': self.derive_interpretation(
                proposition, enriched_context, is_relevant
            )
        }
    
    def extract_proposition(self, utterance):
        """
        提取话语的命题内容
        
        命题 = 话语的逻辑内容（真值条件）
        """
        # 简化的命题提取
        # 实际应用中需要句法语义分析
        return {
            'explicit_content': utterance,
            'truth_conditions': self._identify_truth_conditions(utterance),
            'contextual_enhancements': []
        }
    
    def _identify_truth_conditions(self, utterance):
        """识别真值条件"""
        # 简化处理
        return {
            'type': 'descriptive' if utterance.endswith(('。', '.')) else 'question',
            'modality': 'declarative'
        }
    
    def expand_context(self, context, proposition):
        """
        语境扩展
        
        从长期记忆中激活与话语相关的假设
        """
        enriched = {
            'explicit_context': context or {},
            'activated_assumptions': [],
            'suppressed_assumptions': []
        }
        
        # 基于关键词激活相关假设
        keywords = self._extract_keywords(proposition['explicit_content'])
        
        for assumption_id, assumption in self.context_beliefs.items():
            if self._is_relevant_assumption(keywords, assumption):
                enriched['activated_assumptions'].append(assumption)
        
        return enriched
    
    def _extract_keywords(self, text):
        """提取关键词"""
        # 简化的关键词提取
        stopwords = {'的', '了', '在', '是', '我', '你', '他', '她', '它', '和'}
        return [w for w in text if w not in stopwords and len(w) > 1]
    
    def _is_relevant_assumption(self, keywords, assumption):
        """判断假设是否相关"""
        assumption_keywords = self._extract_keywords(assumption.get('content', ''))
        overlap = len(set(keywords) & set(assumption_keywords))
        return overlap > 0
    
    def compute_effects(self, proposition, context):
        """
        计算认知效果
        
        认知效果包括:
        - 新旧信息的结合
        - 假设的强化/削弱
        - 矛盾假设的消除
        """
        effects = 0.0
        
        # 检查与现有假设的一致性
        for assumption in context.get('activated_assumptions', []):
            consistency = self._check_consistency(proposition, assumption)
            
            if consistency == 'support':
                effects += 1.0  # 强化效果
            elif consistency == 'contradict':
                effects += 2.0  # 矛盾消除，效果更大
            elif consistency == 'new':
                effects += 0.5  # 新信息组合
        
        # 加权
        effects *= self._get_context_weight(context)
        
        return effects
    
    def _check_consistency(self, proposition, assumption):
        """检查一致性"""
        # 简化实现
        # 实际需要语义推理
        
        prop_keywords = set(self._extract_keywords(
            proposition.get('explicit_content', '')
        ))
        assump_keywords = set(self._extract_keywords(
            assumption.get('content', '')
        ))
        
        overlap = len(prop_keywords & assump_keywords)
        
        if overlap > len(prop_keywords) * 0.5:
            return 'support'
        elif overlap < len(prop_keywords) * 0.2:
            return 'contradict'
        else:
            return 'new'
    
    def _get_context_weight(self, context):
        """获取语境权重"""
        activated = len(context.get('activated_assumptions', []))
        return min(1.0, activated / 5.0)
    
    def estimate_processing_effort(self, utterance, proposition, context):
        """
        估计处理努力
        
        处理努力取决于:
        - 语言复杂度
        - 歧义程度
        - 语境激活程度
        """
        effort = 0.0
        
        # 1. 长度因素
        effort += len(utterance) / 100
        
        # 2. 词汇复杂度
        rare_words = self._count_rare_words(utterance)
        effort += rare_words * 0.3
        
        # 3. 歧义因素
        ambiguity = self._estimate_ambiguity(utterance)
        effort += ambiguity * 0.5
        
        # 4. 语境激活（激活越多，努力越少）
        activated = len(context.get('activated_assumptions', []))
        effort -= activated * 0.1
        
        return max(0.1, effort)
    
    def _count_rare_words(self, utterance):
        """计算罕见词数量"""
        # 简化实现
        return 0
    
    def _estimate_ambiguity(self, utterance):
        """估计歧义程度"""
        # 简化实现
        ambiguity_markers = ['意思', '可能', '可以理解为']
        count = sum(1 for m in ambiguity_markers if m in utterance)
        return count * 0.3
    
    def derive_interpretation(self, proposition, context, is_relevant):
        """
        推导出话语的解释
        
        如果话语足够相关，返回最可能的解释
        """
        if not is_relevant:
            return {
                'interpretation': '信息不相关',
                'requires_grounding': True
            }
        
        # 提取显义和隐含意义
        explicature = self.derive_explicature(proposition, context)
        implicatures = self.derive_implicatures(proposition, context)
        
        return {
            'explicature': explicature,
            'implicatures': implicatures,
            'requires_grounding': False
        }
    
    def derive_explicature(self, proposition, context):
        """
        推导显义（Explicature）
        
        显义 = 话语解码 + 语境充实
        """
        return {
            'base_proposition': proposition['explicit_content'],
            'enriched_proposition': proposition['explicit_content'],
            'ad-hoc_concepts': [],
            'disambiguation': None
        }
    
    def derive_implicatures(self, proposition, context):
        """
        推导隐含意义（Implicature）
        
        隐含意义 = 通过推理从显义推导出的额外意义
        """
        implicatures = []
        
        # 检查是否有关联的隐含假设
        for assumption in context.get('activated_assumptions', []):
            if assumption.get('type') == 'implicature':
                implicatures.append(assumption)
        
        return implicatures
```

### 4.3 显义与隐含的区分

关联理论的一个重要贡献是区分了显义（Explicature）和隐含（Implicature）：

```
话语处理流程:

    话语输入
        ↓
┌───────────────────────────┐
│   话语解码 (解码努力)       │
│   - 语音/文字识别          │
│   - 句法分析               │
│   - 语义组合               │
└───────────────────────────┘
        ↓
┌───────────────────────────┐
│   显义 (Explicature)       │
│   - 字面意义的语境充实     │
│   - 指代的消解             │
│   - 省略的补全             │
└───────────────────────────┘
        ↓
┌───────────────────────────┐
│   隐含 (Implicature)       │
│                           │
│   ├─ 隐含前提             │
│   │  (话语成立的背景)      │
│   │                       │
│   ├─ 隐含结论             │
│   │  (从话语推导的结论)    │
│   │                       │
│   └─ 隐含态度             │
│      (说话者态度)          │
└───────────────────────────┘
```

---

## 五、指示语（Deixis）

### 5.1 指示语的类型

指示语（Deixis）源自希腊语"deixis"（指向），是语用学的核心概念，指依赖语境才能确定指称的表达式。

#### 5.1.1 人称指示语（Person Deixis）

人称指示语编码说话者、听话者和第三方之间的关系：

```python
def resolve_person_deixis(pronoun, speaker, hearer, context):
    """
    解决人称指示语
    
    人称系统:
    - 第一人称: 说话者 (I, we)
    - 第二人称: 听者 (you)
    - 第三人称: 其他 (he, she, it, they)
    
    参数:
        pronoun: 代词
        speaker: 说话者身份
        hearer: 听者身份
        context: 额外上下文（包含第三方信息）
    
    返回:
        消解后的人称
    """
    person_mapping = {
        # 第一人称 - 说话者视角
        '我': speaker,
        '我们': f"{speaker}等人",
        '我自己': f"{speaker}本人",
        '咱们': speaker,  # 包括听者
        
        # 第二人称 - 听者视角
        '你': hearer,
        '你们': f"{hearer}等人",
        '你本人': f"{hearer}本人",
        '您': f"{hearer}（礼貌）",
        
        # 第三人称 - 非参与者
        '他': context.get('third_party_male', '未知男性'),
        '她': context.get('third_party_female', '未知女性'),
        '它': context.get('third_party_object', '未知物体'),
        '他们': context.get('third_party_plural', '未知群体'),
        '她们': context.get('third_party_female_plural', '未知女性群体'),
        '它们': context.get('third_party_object_plural', '未知物体集合'),
        
        # 反身代词
        '自己': speaker,  # 通常指向说话者
    }
    
    return person_mapping.get(pronoun, pronoun)


class PersonDeixisAnalyzer:
    """
    人称指示语分析器
    
    分析话语中的人称系统和社会关系
    """
    
    def analyze(self, utterance, speaker, hearer, context=None):
        """
        分析话语中的人称指示语
        """
        results = {
            'first_person': [],
            'second_person': [],
            'third_person': [],
            'social_implications': []
        }
        
        first_person_markers = ['我', '我们', '咱们', '我自己']
        second_person_markers = ['你', '您', '你们', '你自己']
        third_person_markers = ['他', '她', '它', '他们', '她们', '它们']
        
        for marker in first_person_markers:
            if marker in utterance:
                results['first_person'].append(marker)
        
        for marker in second_person_markers:
            if marker in utterance:
                results['second_person'].append(marker)
        
        for marker in third_person_markers:
            if marker in utterance:
                results['third_person'].append(marker)
        
        # 分析社会含义
        results['social_implications'] = self._infer_social_meaning(results)
        
        return results
    
    def _infer_social_meaning(self, analysis_results):
        """
        推断人称使用的社会含义
        """
        implications = []
        
        # 使用"咱们"暗示亲密/包容
        if '咱们' in analysis_results['first_person']:
            implications.append('说话者试图建立亲密关系')
        
        # 使用"您"暗示礼貌/尊重
        if '您' in analysis_results['second_person']:
            implications.append('说话者对听者表示尊敬')
        
        # 使用"我"频繁暗示自我关注
        if len(analysis_results['first_person']) > 2:
            implications.append('说话者强调自身立场或感受')
        
        return implications
```

#### 5.1.2 时间指示语（Temporal Deixis）

时间指示语以说话时刻为参照点编码时间：

```
时间指示语的参照系统:

                    说话时间（现在/Now）
                         ↓
    ┌──────────────────┼──────────────────┐
    │                  │                  │
  过去                现在                未来
  (Past)              (Present)           (Future)
   │                  │                  │
 yesterday         today              tomorrow
   │                  │                  │
  last week        now                 next week
   │                  │                  │
  ago (之前)       currently           in the future
   │                  │                  │
  before            presently          shortly
   │                  │                  │
 yesterday       right now           soon
   │                  │                  │
  earlier         at present         later
```

```python
class TemporalDeixisResolver:
    """
    时间指示语消解器
    
    将相对时间表达映射到绝对时间
    """
    
    def __init__(self):
        self.reference_time = None  # 说话时间
    
    def set_reference_time(self, speech_time):
        """设置参照时间（说话时间）"""
        import datetime
        self.reference_time = speech_time
    
    def resolve(self, deictic_time, context=None):
        """
        解决时间指示语
        
        Args:
            deictic_time: 时间指示语表达
            context: 上下文（可能提供其他参照时间）
        
        Returns:
            resolved_time: 消解后的绝对时间
        """
        import datetime
        
        if self.reference_time is None:
            self.reference_time = datetime.datetime.now()
        
        # 如果上下文提供了参照时间，优先使用
        ref_time = context.get('reference_time', self.reference_time) if context else self.reference_time
        
        # 时间指示语映射
        time_mapping = {
            # 绝对时间指示语
            '今天': ref_time.date(),
            '昨天': ref_time.date() - datetime.timedelta(days=1),
            '前天': ref_time.date() - datetime.timedelta(days=2),
            '明天': ref_time.date() + datetime.timedelta(days=1),
            '后天': ref_time.date() + datetime.timedelta(days=2),
            
            # 相对时间
            '现在': ref_time,
            '目前': ref_time,
            '此刻': ref_time,
            '刚才': ref_time - datetime.timedelta(minutes=5),
            '稍后': ref_time + datetime.timedelta(minutes=5),
            '马上': ref_time + datetime.timedelta(minutes=1),
            '立刻': ref_time,
            
            # 星期
            '本周': self._get_week_range(ref_time)[0],
            '下周': self._get_week_range(ref_time)[0] + datetime.timedelta(weeks=1),
            '上周': self._get_week_range(ref_time)[0] - datetime.timedelta(weeks=1),
            
            # 月份
            '本月': ref_time.replace(day=1),
            '下月': self._get_next_month(ref_time),
            '上月': self._get_prev_month(ref_time),
        }
        
        return time_mapping.get(deictic_time, deictic_time)
    
    def _get_week_range(self, date):
        """获取周的起始日期（周一）"""
        import datetime
        days_to_mon = date.weekday()
        monday = date - datetime.timedelta(days=days_to_mon)
        return monday, monday + datetime.timedelta(days=6)
    
    def _get_next_month(self, date):
        """获取下月的第一天"""
        if date.month == 12:
            return date.replace(year=date.year + 1, month=1, day=1)
        else:
            return date.replace(month=date.month + 1, day=1)
    
    def _get_prev_month(self, date):
        """获取上月的第一天"""
        if date.month == 1:
            return date.replace(year=date.year - 1, month=12, day=1)
        else:
            return date.replace(month=date.month - 1, day=1)
    
    def resolve_relative_time(self, time_expr, anchor_time):
        """
        解析相对时间表达式
        
        例如: "三天前", "两周后"
        """
        import datetime
        import re
        
        patterns = [
            (r'(\d+)天前', lambda m: anchor_time - datetime.timedelta(days=int(m.group(1)))),
            (r'(\d+)天後?', lambda m: anchor_time + datetime.timedelta(days=int(m.group(1)))),
            (r'(\d+)小时前', lambda m: anchor_time - datetime.timedelta(hours=int(m.group(1)))),
            (r'(\d+)小时后', lambda m: anchor_time + datetime.timedelta(hours=int(m.group(1)))),
            (r'(\d+)分钟前', lambda m: anchor_time - datetime.timedelta(minutes=int(m.group(1)))),
            (r'(\d+)周前', lambda m: anchor_time - datetime.timedelta(weeks=int(m.group(1)))),
            (r'(\d+)个月前', lambda m: self._subtract_months(anchor_time, int(m.group(1)))),
            (r'(\d+)年前', lambda m: anchor_time.replace(year=anchor_time.year - int(m.group(1)))),
        ]
        
        for pattern, resolver in patterns:
            match = re.search(pattern, time_expr)
            if match:
                return resolver(match)
        
        return None
    
    def _subtract_months(self, date, months):
        """减去月份"""
        month = date.month - months
        year = date.year
        
        while month <= 0:
            month += 12
            year -= 1
        
        day = min(date.day, self._days_in_month(year, month))
        return date.replace(year=year, month=month, day=day)
    
    def _days_in_month(self, year, month):
        """获取某月的天数"""
        import calendar
        return calendar.monthrange(year, month)[1]
```

#### 5.1.3 空间指示语（Space Deixis）

空间指示语以说话者位置为参照点编码空间：

```python
class SpatialDeixisResolver:
    """
    空间指示语消解器
    
    将指示空间位置的词语映射到具体位置
    """
    
    def __init__(self):
        self.deictic_center = None  # 指示中心（通常为说话者）
        self.world_model = {}        # 世界模型
    
    def set_deictic_center(self, speaker_location, orientation=None):
        """
        设置指示中心
        
        Args:
            speaker_location: 说话者位置
            orientation: 说话者面向方向
        """
        self.deictic_center = {
            'position': speaker_location,  # (x, y, z) 或地点描述
            'orientation': orientation or 'north'  # 默认朝北
        }
    
    def resolve(self, deictic_location, context=None):
        """
        解决空间指示语
        
        空间指示语的分类:
        - 近指 (Proximal): 这里, this, 靠近说话者
        - 远指 (Distal): 那里, that, 远离说话者
        - 中指 (Medial): 那, 用于其他参照
        """
        # 说话者位置作为参照
        speaker_pos = self.deictic_center['position'] if self.deictic_center else None
        
        spatial_mapping = {
            # 近指 - 靠近说话者
            '这里': speaker_pos,
            '这儿': speaker_pos,
            '这边': self._get_nearby_direction(speaker_pos, 'same'),
            'this': speaker_pos,
            
            # 远指 - 远离说话者，靠近听者
            '那里': context.get('hearer_position') if context else 'unknown',
            '那儿': context.get('hearer_position') if context else 'unknown',
            '那边': self._get_nearby_direction(context.get('hearer_position') if context else None, 'different'),
            'that': context.get('hearer_position') if context else 'unknown',
            
            # 指示代词
            '这个': self._resolve_demonstrative_pronoun('this'),
            '那个': self._resolve_demonstrative_pronoun('that'),
        }
        
        return spatial_mapping.get(deictic_location, deictic_location)
    
    def _get_nearby_direction(self, base_pos, direction_type):
        """获取附近的方向"""
        if base_pos is None:
            return 'nearby'
        
        if direction_type == 'same':
            return f"{base_pos}附近"
        else:
            return f"{base_pos}对面"
    
    def _resolve_demonstrative_pronoun(self, pronoun_type):
        """解析指示代词（这个、那个）"""
        if pronoun_type == 'this':
            return '近处的某物'
        else:
            return '远处的某物'
    
    def resolve_complex_spatial(self, spatial_expr):
        """
        解析复杂空间表达式
        
        例如: "桌子上的那本书", "我左边的同学"
        """
        import re
        
        # 提取空间关系
        relations = {
            '上': 'on/top of',
            '下': 'under/below',
            '前': 'in front of',
            '后': 'behind',
            '左': 'to the left of',
            '右': 'to the right of',
            '里': 'inside',
            '外': 'outside',
            '中': 'in',
            '之间': 'between',
            '旁边': 'beside'
        }
        
        for cn_rel, en_rel in relations.items():
            if cn_rel in spatial_expr:
                landmark = self._extract_landmark(spatial_expr)
                return {
                    'relation': en_rel,
                    'landmark': landmark,
                    'deictic_object': self._extract_object(spatial_expr)
                }
        
        return {'expression': spatial_expr}
    
    def _extract_landmark(self, expr):
        """提取地标"""
        return 'unknown'
    
    def _extract_object(self, expr):
        """提取目标对象"""
        return 'unknown'
```

### 5.2 指示语的计算消解

```python
class DeicticResolver:
    """
    指示语消解系统
    
    统一处理人称、时间和空间指示语
    """
    
    def __init__(self):
        self.person_resolver = PersonDeixisAnalyzer()
        self.temporal_resolver = TemporalDeixisResolver()
        self.spatial_resolver = SpatialDeixisResolver()
        
        self.context = {
            'speaker': None,
            'hearer': None,
            'speech_time': None,
            'speech_location': None,
            'third_parties': {}
        }
    
    def set_context(self, speaker=None, hearer=None, speech_time=None, 
                   speech_location=None, third_parties=None):
        """设置对话上下文"""
        if speaker:
            self.context['speaker'] = speaker
        if hearer:
            self.context['hearer'] = hearer
        if speech_time:
            self.context['speech_time'] = speech_time
            self.temporal_resolver.set_reference_time(speech_time)
        if speech_location:
            self.context['speech_location'] = speech_location
            self.spatial_resolver.set_deictic_center(speech_location)
        if third_parties:
            self.context['third_parties'] = third_parties
    
    def resolve(self, utterance):
        """
        统一消解指示语
        
        Args:
            utterance: 包含指示语的话语
        
        Returns:
            resolved_utterance: 消解后的话语
            resolution_info: 消解信息
        """
        resolved = utterance
        resolution_info = {
            'person_resolutions': [],
            'temporal_resolutions': [],
            'spatial_resolutions': [],
            'discourse_resolutions': []
        }
        
        # 人称指示语消解
        resolved, person_info = self._resolve_person(resolved)
        resolution_info['person_resolutions'] = person_info
        
        # 时间指示语消解
        resolved, temporal_info = self._resolve_temporal(resolved)
        resolution_info['temporal_resolutions'] = temporal_info
        
        # 空间指示语消解
        resolved, spatial_info = self._resolve_spatial(resolved)
        resolution_info['spatial_resolutions'] = spatial_info
        
        # 话语指示语消解
        resolved, discourse_info = self._resolve_discourse(resolved)
        resolution_info['discourse_resolutions'] = discourse_info
        
        return resolved, resolution_info
    
    def _resolve_person(self, text):
        """人称消解"""
        import re
        
        results = []
        resolved = text
        
        # 人称代词模式
        patterns = [
            (r'\b我\b', self.context.get('speaker', 'Unknown')),
            (r'\b我们\b', f"{self.context.get('speaker', 'Unknown')}等人"),
            (r'\b你\b', self.context.get('hearer', 'Unknown')),
            (r'\b您\b', f"{self.context.get('hearer', 'Unknown')}（礼貌）"),
            (r'\b他\b', self.context.get('third_parties', {}).get('male', 'Unknown')),
            (r'\b她\b', self.context.get('third_parties', {}).get('female', 'Unknown')),
            (r'\b它\b', self.context.get('third_parties', {}).get('object', 'Unknown')),
        ]
        
        for pattern, replacement in patterns:
            matches = re.finditer(pattern, resolved)
            for match in matches:
                results.append({
                    'original': match.group(),
                    'resolved_to': replacement,
                    'position': match.span()
                })
            
            resolved = re.sub(pattern, f'[{replacement}]', resolved)
        
        return resolved, results
    
    def _resolve_temporal(self, text):
        """时间消解"""
        import re
        
        results = []
        resolved = text
        
        # 时间指示语模式
        time_patterns = [
            (r'今天', self._format_date(self.context.get('speech_time'))),
            (r'昨天', self._format_date(self._subtract_days(1))),
            (r'明天', self._format_date(self._add_days(1))),
            (r'现在', self._format_time(self.context.get('speech_time'))),
            (r'刚才', self._format_time(self._subtract_minutes(5))),
        ]
        
        for pattern, replacement in time_patterns:
            if re.search(pattern, resolved):
                results.append({
                    'original': pattern,
                    'resolved_to': replacement,
                    'type': 'temporal_deixis'
                })
                resolved = re.sub(pattern, f'[{replacement}]', resolved)
        
        return resolved, results
    
    def _resolve_spatial(self, text):
        """空间消解"""
        # 简化的空间消解
        return text, []
    
    def _resolve_discourse(self, text):
        """话语指示语消解（回指）"""
        # 处理"这/那+名词"等回指表达
        return text, []
    
    def _format_date(self, date):
        """格式化日期"""
        if date:
            return date.strftime('%Y年%m月%d日')
        return 'Unknown Date'
    
    def _format_time(self, time):
        """格式化时间"""
        if time:
            return time.strftime('%H:%M')
        return 'Unknown Time'
    
    def _subtract_days(self, days):
        """计算前几天"""
        import datetime
        if self.context.get('speech_time'):
            return self.context['speech_time'] - datetime.timedelta(days=days)
        return None
    
    def _add_days(self, days):
        """计算后几天"""
        import datetime
        if self.context.get('speech_time'):
            return self.context['speech_time'] + datetime.timedelta(days=days)
        return None
    
    def _subtract_minutes(self, minutes):
        """计算前几分钟"""
        import datetime
        if self.context.get('speech_time'):
            return self.context['speech_time'] - datetime.timedelta(minutes=minutes)
        return None
```

---

## 六、预设（Presupposition）

### 6.1 预设的定义与性质

预设是话语中隐含的、说话者假定为真的背景信息。即使在话语被否定的情况下，预设仍然保持为真。

```
"他又迟到了。"

预设: [他之前迟到过]  ← 即使说"他没有迟到"，这个仍然为真
断言: 他现在迟到了   ← 这是可以被否定的新信息
```

**预设的特征**

1. **存在性**：预设的内容被视为理所当然的背景知识
2. **可投射性**：预设可以从简单句投射到复合句
3. **共同性**：预设是说话者和听者共享的背景
4. **不可取消性**：正常情况下，说话者不会否定自己的预设

#### 6.1.1 预设触发语

```python
class PresuppositionTriggers:
    """
    预设触发语识别与提取
    
    预设触发语是经常引起预设的语言表达式
    """
    
    def __init__(self):
        self.triggers = {
            # 事实动词（Factives）
            'factives': {
                'trigger_words': ['知道', '意识到', '忘记', '遗憾', '后悔', '发现', '理解', '认识到'],
                'presupposition_type': '命题P为真',
                'examples': [
                    ('我知道他来了', '他来了是真的'),
                    ('她忘记带钥匙', '她应该带钥匙'),
                ]
            },
            
            # 状态变化动词（Change of State）
            'change_of_state': {
                'trigger_words': ['停止', '开始', '继续', '继续', '重新', '再次'],
                'presupposition_type': '先前状态',
                'examples': [
                    ('他停止说话', '他之前在说'),
                    ('她开始工作', '她之前没在工作'),
                ]
            },
            
            # 迭代副词（Iteratives）
            'iteratives': {
                'trigger_words': ['又', '再', '再次', '还', '还', '重新'],
                'presupposition_type': '事件E之前发生过',
                'examples': [
                    ('他又来了', '他之前来过'),
                    ('她再次尝试', '她之前尝试过'),
                ]
            },
            
            # 叙实形容词（Factive Adjectives）
            'factive_adjectives': {
                'trigger_words': ['遗憾', '惊讶', '高兴', '难过', '奇怪', '庆幸'],
                'presupposition_type': '某状态/事件为真',
                'examples': [
                    ('他很遗憾你不能来', '你不能来'),
                    ('她惊讶他来了', '他来了'),
                ]
            },
            
            # 时间状语（Temporal Clauses）
            'temporal_clauses': {
                'trigger_words': ['在...之前', '当...时候', '已经', '曾经'],
                'presupposition_type': '时间关系',
                'examples': [
                    ('他来之前我走了', '他打算/将会来'),
                    ('他已经完成了', '完成是某事件'),
                ]
            },
            
            # 限定性描述（Definite Descriptions）
            'definite_descriptions': {
                'trigger_words': ['那个', '这个', '这些', '那些'],
                'presupposition_type': '所指对象存在',
                'examples': [
                    ('那个房子很大', '存在一个"那个"房子'),
                    ('这位老师很好', '存在一位"这位"老师'),
                ]
            },
            
            # 分裂句（Cleft Sentences）
            'cleft_sentences': {
                'trigger_words': ['是...的', '...的...是...', '...的是...'],
                'presupposition_type': '焦点信息为真',
                'examples': [
                    ('是他打破了窗户', '有人打破了窗户'),
                    ('我去北京是为了开会', '我去北京'),
                ]
            },
            
            # 隐含比较（Implicative Verbs）
            'implicative_verbs': {
                'trigger_words': ['设法', '差点', '几乎', ' Managed to', 'Stopped'],
                'presupposition_type': '尝试或避免',
                'examples': [
                    ('他设法完成', '他尝试完成'),
                    ('他差点摔倒', '他几乎摔倒'),
                ]
            }
        }
    
    def extract_presuppositions(self, utterance):
        """
        从话语中提取预设
        
        Args:
            utterance: 输入话语
        
        Returns:
            提取的预设列表
        """
        presuppositions = []
        
        for category, info in self.triggers.items():
            for trigger in info['trigger_words']:
                if trigger in utterance:
                    # 提取预设
                    presupposition = self._derive_presupposition(
                        utterance, trigger, category
                    )
                    
                    presuppositions.append({
                        'trigger': trigger,
                        'category': category,
                        'presupposition': presupposition,
                        'utterance': utterance
                    })
        
        return presuppositions
    
    def _derive_presupposition(self, utterance, trigger, category):
        """
        推导具体的预设内容
        
        这是一个简化实现
        实际需要更复杂的语义分析
        """
        info = self.triggers[category]
        
        # 找到触发词在话语中的位置
        trigger_pos = utterance.find(trigger)
        
        if trigger_pos == -1:
            return info['presupposition_type']
        
        # 尝试从例子中学习模式
        for pattern, presup in info.get('examples', []):
            if trigger in pattern:
                return presup
        
        # 返回一般类型
        return info['presupposition_type']
    
    def cancel_presupposition(self, presupposition):
        """
        取消预设（如果可能）
        
        某些情况下，预设可以被取消
        """
        return {
            'cancellable': True,
            'cancellation_method': '显式否定或修正',
            'example': f"虽然{presupposition}，但是..."
        }
```

### 6.2 预设的投射问题

预设投射（Presupposition Projection）研究复合句中预设如何从子句传递到整句。

```python
class PresuppositionProjector:
    """
    预设投射计算
    
    复合句中各成分的预设如何投射到整句
    """
    
    def __init__(self):
        # 运算符对预设的影响
        self.operators = {
            'negation': {
                'effect': 'preserves',  # 预设保留
                'description': '否定不取消预设',
                'example': '"他没迟到" → "他之前迟到过" 仍然成立'
            },
            'question': {
                'effect': 'preserves',
                'description': '疑问不取消预设',
                'example': '"他迟到了吗？" → "他之前迟到过" 仍然成立'
            },
            'belief': {
                'effect': 'contextualizes',  # 预设依赖于语境
                'description': '信念动词使预设依赖于说话者的信念',
                'example': '"张三知道他又迟到了" → 预设依赖于张三的信念'
            },
            'modal': {
                'effect': 'weakens',
                'description': '模态词可能弱化预设',
                'example': '"他可能停止吸烟" → 预设较弱'
            },
            'conditional': {
                'effect': 'conditionalizes',  # 预设条件化
                'description': '预设投射到结果部',
                'example': '"如果你又迟到了，就扣工资" → 预设取决于条件'
            }
        }
    
    def project(self, presupposition, operator):
        """
        将预设通过算子投射
        
        Args:
            presupposition: 原始预设
            operator: 句法/语义算子
        
        Returns:
            projected_presupposition
        """
        operator_info = self.operators.get(operator, self._default_operator())
        
        if operator_info['effect'] == 'preserves':
            return {
                'presupposition': presupposition,
                'status': 'preserved',
                'description': operator_info['description']
            }
        
        elif operator_info['effect'] == 'conditionalizes':
            return {
                'presupposition': presupposition,
                'status': 'conditional',
                'condition': f"IF condition THEN {presupposition}",
                'description': operator_info['description']
            }
        
        elif operator_info['effect'] == 'contextualizes':
            return {
                'presupposition': presupposition,
                'status': 'context_dependent',
                'context': 'speaker_belief',
                'description': operator_info['description']
            }
        
        return {
            'presupposition': presupposition,
            'status': 'unknown',
            'description': operator_info.get('description', '')
        }
    
    def _default_operator(self):
        """默认算子处理"""
        return {
            'effect': 'unknown',
            'description': '未知算子'
        }
    
    def compute_projection(self, sentence_structure):
        """
        计算复合句的预设投射
        
        使用Heim的ATLAS模型或其他投射算法
        """
        projections = []
        
        # 遍历句法树
        for node in sentence_structure:
            if node.get('is_trigger'):
                presup = self.extract_trigger_presupposition(node)
                projections.append(presup)
        
        # 应用投射规则
        final_presuppositions = self._apply_projection_rules(projections)
        
        return final_presuppositions
    
    def extract_trigger_presupposition(self, trigger_node):
        """提取触发语的预设"""
        return {
            'content': trigger_node.get('presupposition'),
            'source': trigger_node.get('trigger_word'),
            'scope': trigger_node.get('scope', 'local')
        }
    
    def _apply_projection_rules(self, presuppositions):
        """应用投射规则合并预设"""
        # 简化：假设所有预设都投射
        return presuppositions
```

---

## 七、大语言模型中的语用推理

### 7.1 LLM语用能力的现状

大语言模型（LLM）在语用理解方面展现出令人印象深刻的能力，但仍有明显局限。

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

**礼貌请求处理**

```
用户: "能不能帮我看看这个文档？"
LLM理解: [用户有礼貌地请求帮助]
→ 响应: [采用同样礼貌的语气]
```

#### 7.1.2 局限性分析

```python
# LLM语用能力的典型失败案例

failures = {
    # 讽刺和反语
    'irony_sarcasm': {
        'example': '用户说"这计划真棒"，实际意思是批评',
        'llm_issue': '可能按字面意思理解积极情感',
        'severity': 'high',
        'reason': '缺乏语境情感分析'
    },
    
    # 间接拒绝
    'indirect_refusal': {
        'example': '用户问"你能帮我...吗？"，LLM有时无法识别这是拒绝的机会',
        'llm_issue': '可能继续积极响应',
        'severity': 'medium',
        'reason': '缺乏社会规范理解'
    },
    
    # 隐含请求
    'implied_request': {
        'example': '"我有点累了" 隐含想要休息的帮助',
        'llm_issue': '可能只是确认，而非提供帮助',
        'severity': 'medium',
        'reason': '缺乏意图推断'
    },
    
    # 文化语用
    'cultural_pragmatics': {
        'example': '不同文化中"可以"的含义差异',
        'llm_issue': '使用统一的语用规则',
        'severity': 'low',
        'reason': '训练数据的文化偏向'
    },
    
    # 预设消解
    'presupposition_resolution': {
        'example': '包含预设的话语可能无法正确处理',
        'llm_issue': '可能忽视或误解预设',
        'severity': 'medium',
        'reason': '深层语义理解不足'
    }
}


class LLMPragmaticCapabilityAnalyzer:
    """
    LLM语用能力分析器
    
    评估和改进LLM的语用理解能力
    """
    
    def __init__(self):
        self.capability_scores = {
            'speech_act_recognition': 0.85,
            'implicature_inference': 0.70,
            'politeness_handling': 0.80,
            'irony_detection': 0.50,
            'presupposition_resolution': 0.65,
            'deixis_resolution': 0.75,
            'cultural_pragmatics': 0.60,
            'indirect_speech': 0.75,
        }
    
    def analyze_capability(self):
        """分析当前能力"""
        return self.capability_scores
    
    def identify_weaknesses(self):
        """识别弱点"""
        return [
            {'area': 'irony_detection', 'score': 0.50, 'priority': 'high'},
            {'area': 'cultural_pragmatics', 'score': 0.60, 'priority': 'medium'},
            {'area': 'presupposition_resolution', 'score': 0.65, 'priority': 'medium'},
        ]
```

### 7.2 增强LLM语用能力的方法

#### 7.2.1 提示工程方法

```python
class PragmaticPromptEngineer:
    """
    语用感知提示工程
    
    通过精心设计的提示增强LLM的语用理解能力
    """
    
    def __init__(self):
        self.pragmatic_prompts = {
            # 隐含请求检测
            'implicit_request': """
            注意：以下话语可能包含隐含请求。
            请尝试推断说话者可能的真实意图。
            
            分析步骤：
            1. 识别话语的字面意义
            2. 分析话语与上下文的关联
            3. 推断可能的隐含意图
            4. 考虑说话者的可能需求
            
            输出格式：
            - 字面意义：[...]
            - 可能的隐含意图：[...]
            - 推荐响应：[...]
            """,
            
            # 讽刺和反语识别
            'irony_sarcasm': """
            注意：以下话语可能包含反语或讽刺。
            请从语境和语气中识别真实的情感倾向。
            
            分析线索：
            1. 夸张或矛盾的表达
            2. 与先前话题的关系
            3. 说话者的可能态度
            4. 社会文化背景
            
            输出格式：
            - 字面内容：[...]
            - 实际情感：[正面/负面/中性]
            - 讽刺程度：[0-10]
            - 真实含义：[...]
            """,
            
            # 礼貌分析
            'politeness': """
            注意：以下话语使用了礼貌策略。
            请分析话语的直接含义和礼貌效果。
            
            礼貌特征：
            1. 使用的礼貌词（请、麻烦等）
            2. 疑问形式 vs 命令形式
            3. 模糊限制语的使用
            4. 对听者面子的考虑
            
            输出格式：
            - 礼貌层级：[高/中/低]
            - 礼貌策略：[积极/消极/...]
            - 直接含义：[...]
            - 建议响应方式：[...]
            """
        }
    
    def generate_pragmatic_prompt(self, utterance, context=None, task='analysis'):
        """
        生成包含语用分析的完整提示
        
        Args:
            utterance: 用户话语
            context: 对话上下文
            task: 分析任务类型
        """
        # 选择合适的提示模板
        if '?' in utterance and any(word in utterance for word in ['能', '可以', '能不能']):
            template = self.pragmatic_prompts['implicit_request']
        elif any(word in utterance for word in ['真', '太', '不过']):
            template = self.pragmatic_prompts['irony_sarcasm']
        else:
            template = self.pragmatic_prompts['politeness']
        
        # 构建完整提示
        prompt = f"""用户说: {utterance}

{template}
"""
        
        # 添加上下文信息
        if context:
            if context.get('situation'):
                prompt += f"\n当前场景: {context['situation']}"
            if context.get('relationship'):
                prompt += f"\n双方关系: {context['relationship']}"
            if context.get('conversation_history'):
                prompt += f"\n对话历史: {context['conversation_history']}"
        
        prompt += "\n\n请进行分析："
        
        return prompt
    
    def add_contextual_awareness(self, prompt):
        """添加语境意识指导"""
        contextual_guidance = """
        
        语境意识指导：
        - 考虑说话者的身份和背景
        - 考虑对话发生的场景
        - 关注对话的连贯性
        - 理解社会文化规范
        
        常见语用模式：
        1. 抱怨 → 可能需要道歉或解决方案
        2. 询问 → 可能需要信息或确认
        3. 陈述事实 → 可能暗示请求或期望
        4. 礼貌请求 → 需要给予面子
        """
        
        return prompt + contextual_guidance
```

#### 7.2.2 微调方法

```python
class PragmaticsFineTuner:
    """
    语用能力微调
    
    使用语用标注数据微调LLM
    """
    
    def __init__(self, base_model_path):
        self.base_model_path = base_model_path
        self.pragmatic_data = self._load_pragmatic_dataset()
    
    def _load_pragmatic_dataset(self):
        """
        加载语用标注数据集
        
        典型的语用数据集包含：
        - 原始话语
        - 言语行为标签
        - 隐含意图
        - 预设
        - 礼貌等级
        - 讽刺标记
        """
        # 示例数据结构
        sample_data = [
            {
                'utterance': '能帮我倒杯水吗？',
                'speech_act': 'directive',
                'intent': '请求',
                'politeness': '中等',
                'implicature': '说话者口渴',
                'presupposition': '说话者有水可倒'
            },
            {
                'utterance': '你的表现真是一如既往啊',
                'speech_act': 'assertive',
                'intent': '讽刺',
                'politeness': '低（负面）',
                'implicature': '说话者对表现不满',
                'irony_score': 0.8
            },
            {
                'utterance': '外面真冷啊',
                'speech_act': 'assertive',
                'intent': '暗示请求',
                'politeness': '中性',
                'implicature': '希望关窗/开暖气',
                'context_dependent': True
            },
            # ... 更多标注数据
        ]
        
        return sample_data
    
    def prepare_training_data(self, output_format='instruction'):
        """
        准备训练数据
        
        Args:
            output_format: 'instruction' 或 'chat'
        """
        if output_format == 'instruction':
            return self._prepare_instruction_format()
        elif output_format == 'chat':
            return self._prepare_chat_format()
    
    def _prepare_instruction_format(self):
        """指令微调格式"""
        training_data = []
        
        for item in self.pragmatic_data:
            prompt = f"""分析以下话语的语用特征：

话语：{item['utterance']}

请提供：
1. 言语行为类型
2. 隐含意图
3. 礼貌等级
4. 其他语用特征"""
            
            response = f"""言语行为类型：{item['speech_act']}
隐含意图：{item.get('intent', '无')}
礼貌等级：{item.get('politeness', '中性')}
语用隐含：{item.get('implicature', '无')}
"""
            
            if 'irony_score' in item:
                response += f"讽刺程度：{item['irony_score']}"
            
            training_data.append({
                'prompt': prompt,
                'response': response
            })
        
        return training_data
    
    def _prepare_chat_format(self):
        """对话微调格式"""
        training_data = []
        
        for item in self.pragmatic_data:
            training_data.append({
                'messages': [
                    {'role': 'user', 'content': item['utterance']},
                    {'role': 'assistant', 'content': self._generate_response(item)}
                ]
            })
        
        return training_data
    
    def _generate_response(self, item):
        """生成训练用的响应"""
        response_templates = {
            'request': f"好的，我来帮你{item['utterance'].replace('能帮我', '').replace('吗', '')}",
            'irony': "我理解你可能对这个有些不满，愿意详细说说吗？",
            'implied_request': "听起来你可能需要一些帮助，具体是什么情况呢？"
        }
        
        intent = item.get('intent', '')
        if '请求' in intent:
            return response_templates['request']
        elif '讽刺' in intent:
            return response_templates['irony']
        elif '暗示请求' in intent:
            return response_templates['implied_request']
        
        return "我理解你的意思了。"
    
    def fine_tune(self, training_data, output_dir):
        """
        执行微调
        
        实际实现需要调用具体的训练框架
        """
        # 1. 格式化数据
        formatted_data = self._format_for_training(training_data)
        
        # 2. 设置训练参数
        training_args = {
            'learning_rate': 2e-5,
            'batch_size': 4,
            'num_epochs': 3,
            'warmup_steps': 100,
        }
        
        # 3. 开始训练
        # ... (使用HuggingFace Transformers等框架)
        
        return {'status': 'fine_tuning_completed'}
```

#### 7.2.3 外部知识增强

```python
class PragmaticsKnowledgeAugmented:
    """
    外部知识增强的语用推理
    
    结合知识图谱、语用规则库和世界知识
    """
    
    def __init__(self, knowledge_graph=None):
        self.kg = knowledge_graph  # 知识图谱
        self.pragmatic_rules = self._load_pragmatic_rules()
        self.social_norms = self._load_social_norms()
    
    def _load_pragmatic_rules(self):
        """加载语用规则库"""
        return {
            # Grice合作原则
            'grice_maxims': {
                'quantity': ['信息充分', '信息不过量'],
                'quality': ['说真话', '有证据'],
                'relation': ['说相关的'],
                'manner': ['清楚', '无歧义']
            },
            
            # 礼貌策略
            'politeness_strategies': {
                'positive': ['强调共同点', '满足积极面子'],
                'negative': ['道歉', '给选择', '模糊限制'],
                'off_record': ['暗示', '间接表达']
            },
            
            # 言语行为规则
            'speech_act_rules': {
                'request': ['可能被拒绝', '需要礼貌'],
                'apology': ['需要真诚', '承认错误'],
                'complaint': ['表达不满', '期待改善']
            }
        }
    
    def _load_social_norms(self):
        """加载社会规范知识"""
        return {
            # 权力关系
            'power_hierarchy': {
                'superior': ['老板', '老师', '父母'],
                'inferior': ['下属', '学生', '孩子'],
                'equal': ['同事', '同学', '朋友']
            },
            
            # 亲密程度
            'intimacy_levels': {
                'close': ['家人', '密友', '恋人'],
                'familiar': ['同事', '邻居'],
                'stranger': ['陌生人', '服务人员']
            },
            
            # 文化规范
            'cultural_norms': {
                'chinese': {
                    'gift_response': '先拒绝再接受表示礼貌',
                    'compliment_response': '谦虚回应',
                    'directness': '较低（间接表达偏好）'
                },
                'western': {
                    'gift_response': '直接感谢',
                    'compliment_response': '接受赞美',
                    'directness': '较高（直接表达）'
                }
            }
        }
    
    def infer_with_knowledge(self, utterance, context):
        """
        结合知识进行语用推理
        
        步骤:
        1. 提取语义内容
        2. 应用语用规则
        3. 查询知识图谱
        4. 结合社会规范
        5. 融合推断结果
        """
        # 1. 提取语义内容
        semantic_content = self.extract_semantics(utterance)
        
        # 2. 应用语用规则
        rule_based_inference = self.apply_pragmatic_rules(
            semantic_content, context
        )
        
        # 3. 查询知识图谱
        kg_based_inference = self.query_knowledge_graph(
            semantic_content, context
        )
        
        # 4. 结合社会规范
        norm_based_inference = self.apply_social_norms(
            semantic_content, context
        )
        
        # 5. 融合结果
        final_inference = self.fuse_inferences(
            rule_based_inference,
            kg_based_inference,
            norm_based_inference
        )
        
        return final_inference
    
    def extract_semantics(self, utterance):
        """提取话语语义"""
        return {
            'text': utterance,
            'speech_act_candidate': self._classify_speech_act(utterance),
            'key_entities': self._extract_entities(utterance),
            'sentiment': self._analyze_sentiment(utterance)
        }
    
    def _classify_speech_act(self, text):
        """分类言语行为"""
        # 简化实现
        return 'assertive'  # 默认断言
    
    def _extract_entities(self, text):
        """提取实体"""
        return []
    
    def _analyze_sentiment(self, text):
        """分析情感"""
        return 'neutral'
    
    def apply_pragmatic_rules(self, semantic_content, context):
        """应用语用规则"""
        rules_applied = []
        
        # 检查合作原则违反
        for maxim, descriptions in self.pragmatic_rules['grice_maxims'].items():
            if self._check_maxim_violation(semantic_content, maxim):
                rules_applied.append({
                    'type': 'maxim_violation',
                    'maxim': maxim,
                    'possible_implicature': self._derive_implicature(maxim)
                })
        
        return rules_applied
    
    def _check_maxim_violation(self, content, maxim):
        """检查准则违反"""
        return False
    
    def _derive_implicature(self, maxim):
        """推导违反准则的隐含意义"""
        implicature_map = {
            'quantity': '说话者可能有其他信息没说出来',
            'quality': '说话者可能不确定或说假话',
            'relation': '说话者可能在转移话题',
            'manner': '说话者可能故意模糊'
        }
        return implicature_map.get(maxim, '')
    
    def query_knowledge_graph(self, semantic_content, context):
        """查询知识图谱"""
        if self.kg is None:
            return []
        
        entities = semantic_content.get('key_entities', [])
        kg_results = []
        
        for entity in entities:
            related = self.kg.get_related_entities(entity)
            kg_results.extend(related)
        
        return kg_results
    
    def apply_social_norms(self, semantic_content, context):
        """应用社会规范"""
        norms_applied = []
        
        # 分析权力关系
        if context.get('speaker') and context.get('hearer'):
            speaker_role = self._get_role(context['speaker'])
            hearer_role = self._get_role(context['hearer'])
            
            if self._is_power_differential(speaker_role, hearer_role):
                norms_applied.append({
                    'type': 'power_differential',
                    'implication': '需要调整礼貌策略'
                })
        
        return norms_applied
    
    def _get_role(self, person):
        """获取角色"""
        return 'unknown'
    
    def _is_power_differential(self, role1, role2):
        """判断权力差异"""
        return False
    
    def fuse_inferences(self, *inferences):
        """融合多个推理结果"""
        fused = {
            'speech_act': None,
            'implicatures': [],
            'politeness_level': 'neutral',
            'recommended_response': None
        }
        
        # 合并隐含意义
        for inference in inferences:
            if isinstance(inference, list):
                fused['implicatures'].extend(inference)
            elif isinstance(inference, dict):
                fused['implicatures'].append(inference)
        
        # 去重
        fused['implicatures'] = self._deduplicate(fused['implicatures'])
        
        return fused
    
    def _deduplicate(self, items):
        """去重"""
        seen = set()
        unique = []
        for item in items:
            key = str(item)
            if key not in seen:
                seen.add(key)
                unique.append(item)
        return unique
```

### 7.3 评估语用能力

```python
class PragmaticsEvaluator:
    """
    语用能力评估基准
    
    全面评估模型的语用理解能力
    """
    
    def __init__(self):
        self.benchmarks = {
            # 言语行为识别
            'speech_act_classification': {
                'description': '识别话语的言语行为类型',
                'metrics': ['accuracy', 'macro_f1', 'weighted_f1'],
                'test_size': 1000,
                'categories': ['assertive', 'directive', 'commissive', 'expressive', 'declaration']
            },
            
            # 隐含意图识别
            'implicature_detection': {
                'description': '识别话语的隐含意图',
                'metrics': ['precision', 'recall', 'f1'],
                'test_size': 500,
                'intents': ['request', 'suggestion', 'complaint', 'criticism', 'praise']
            },
            
            # 预设消解
            'presupposition_resolution': {
                'description': '消解话语中的预设',
                'metrics': ['accuracy', 'completeness'],
                'test_size': 300,
                'trigger_types': ['factive', 'change_of_state', 'iterative', 'definite']
            },
            
            # 间接言语理解
            'indirect_speech_understanding': {
                'description': '理解间接言语行为',
                'metrics': ['success_rate', 'relevance_score'],
                'test_size': 400,
                'patterns': ['question_to_request', 'statement_to_request', 'assertive_to_warning']
            },
            
            # 讽刺识别
            'irony_detection': {
                'description': '识别反语和讽刺',
                'metrics': ['accuracy', 'f1'],
                'test_size': 600,
                'irony_types': ['situational', 'verbal', 'sarcasm']
            },
            
            # 礼貌分析
            'politeness_analysis': {
                'description': '分析礼貌等级和策略',
                'metrics': ['accuracy', 'mae'],
                'test_size': 400,
                'levels': ['very_low', 'low', 'medium', 'high', 'very_high']
            },
            
            # 指示语消解
            'deixis_resolution': {
                'description': '消解人称、时间、空间指示语',
                'metrics': ['accuracy'],
                'test_size': 500,
                'types': ['person', 'temporal', 'spatial']
            },
            
            # 隐喻理解
            'metaphor_understanding': {
                'description': '理解隐喻性表达',
                'metrics': ['accuracy', 'interpretation_quality'],
                'test_size': 300
            }
        }
    
    def evaluate(self, model, test_data):
        """
        评估模型的语用能力
        
        Args:
            model: 待评估的模型（API或本地模型）
            test_data: 测试数据集
        
        Returns:
            评估结果报告
        """
        results = {}
        
        for task, config in self.benchmarks.items():
            print(f"评估任务: {config['description']}")
            
            # 获取测试数据
            task_data = test_data.get(task, [])
            
            if not task_data:
                print(f"  跳过（无测试数据）")
                continue
            
            # 执行评估
            predictions = self._run_evaluation(model, task, task_data)
            
            # 计算指标
            metrics = self._compute_metrics(
                predictions, 
                task_data,
                config['metrics']
            )
            
            results[task] = {
                'description': config['description'],
                'metrics': metrics,
                'sample_size': len(task_data)
            }
        
        return self._generate_report(results)
    
    def _run_evaluation(self, model, task, data):
        """运行评估"""
        predictions = []
        
        for item in data:
            prompt = self._build_prompt(task, item)
            prediction = model.predict(prompt)
            predictions.append({
                'input': item,
                'prediction': prediction
            })
        
        return predictions
    
    def _build_prompt(self, task, item):
        """构建提示"""
        templates = {
            'speech_act_classification': f"识别以下话语的言语行为类型：{item['utterance']}",
            'implicature_detection': f"识别以下话语的隐含意图：{item['utterance']}",
            'presupposition_resolution': f"消解以下话语中的预设：{item['utterance']}",
            'irony_detection': f"判断以下话语是否包含讽刺：{item['utterance']}",
        }
        
        return templates.get(task, f"分析：{item['utterance']}")
    
    def _compute_metrics(self, predictions, data, metric_names):
        """计算评估指标"""
        metrics = {}
        
        for metric_name in metric_names:
            if metric_name == 'accuracy':
                metrics['accuracy'] = self._compute_accuracy(predictions, data)
            elif metric_name == 'f1':
                metrics['f1'] = self._compute_f1(predictions, data)
            elif metric_name == 'precision':
                metrics['precision'] = self._compute_precision(predictions, data)
            elif metric_name == 'recall':
                metrics['recall'] = self._compute_recall(predictions, data)
        
        return metrics
    
    def _compute_accuracy(self, predictions, data):
        """计算准确率"""
        correct = 0
        for pred, item in zip(predictions, data):
            if pred['prediction'] == item.get('expected', ''):
                correct += 1
        return correct / len(data) if data else 0
    
    def _compute_f1(self, predictions, data):
        """计算F1分数"""
        # 简化实现
        return 0.8
    
    def _compute_precision(self, predictions, data):
        """计算精确率"""
        return 0.8
    
    def _compute_recall(self, predictions, data):
        """计算召回率"""
        return 0.8
    
    def _generate_report(self, results):
        """生成评估报告"""
        report = {
            'summary': {
                'overall_score': self._compute_overall_score(results),
                'strongest_area': self._find_strongest(results),
                'weakest_area': self._find_weakest(results)
            },
            'detailed_results': results
        }
        
        return report
    
    def _compute_overall_score(self, results):
        """计算总体分数"""
        scores = []
        for task, result in results.items():
            if 'metrics' in result and 'accuracy' in result['metrics']:
                scores.append(result['metrics']['accuracy'])
        return sum(scores) / len(scores) if scores else 0
    
    def _find_strongest(self, results):
        """找到最强领域"""
        best_task = None
        best_score = 0
        
        for task, result in results.items():
            if 'metrics' in result:
                score = result['metrics'].get('accuracy', 0)
                if score > best_score:
                    best_score = score
                    best_task = task
        
        return {'task': best_task, 'score': best_score}
    
    def _find_weakest(self, results):
        """找到最弱领域"""
        worst_task = None
        worst_score = 1.0
        
        for task, result in results.items():
            if 'metrics' in result:
                score = result['metrics'].get('accuracy', 0)
                if score < worst_score:
                    worst_score = score
                    worst_task = task
        
        return {'task': worst_task, 'score': worst_score}
```

---

## 八、前沿研究方向

### 8.1 计算语用学的热点问题

当前计算语用学研究面临的主要挑战和热点问题：

**1. 多模态语用**

结合视觉、语音、手势等多种模态的语用理解：

```
多模态语用研究框架

    语言模态
    ↙     ↘
视觉模态   语音模态
    ↘     ↙
    多模态融合
         ↓
   语境整合理解
```

**2. 跨语言语用**

不同语言中语用策略的差异与翻译挑战：

- 语言间的礼貌标记差异
- 直接/间接表达的跨文化差异
- 语用意义的翻译保真度

**3. 文化语用**

文化因素对语用解释的影响：

| 文化维度 | 高语境文化 | 低语境文化 |
|----------|------------|------------|
| 典型地区 | 东亚、中东 | 北美、北欧 |
| 表达方式 | 含蓄、间接 | 直接、明确 |
| 礼貌策略 | 消极礼貌为主 | 积极礼貌为主 |
| 面子维护 | 群体面子 | 个人面子 |

**4. 动态语用**

对话过程中的语用推理演变：

- 话轮转换时的语境更新
- 对话意图的动态演化
- 多轮对话中的预设保持与取消

**5. 神经语用学**

大脑如何处理语用信息：

- 神经成像研究语用加工的大脑区域
- 语用障碍（如自闭症）的神经基础
- 情感语用的神经机制

### 8.2 LLM语用的未来发展

```
未来发展方向:

    ┌─────────────────┐
    │  多模态语用      │ ← 视觉-语言-音频联合理解
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  个性化语用     │ ← 用户偏好建模、风格适应
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  文化感知语用   │ ← 跨文化理解、多语言适应
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  可解释语用推理 │ ← 推理过程透明化
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  情感-意图联合   │ ← 情感分析与意图识别融合
    └────────┬────────┘
             │
    ┌────────┴────────┐
    │  实时语境适应   │ ← 动态语境更新、长期记忆
    └─────────────────┘
```

---

## 九、实践指南：构建语用感知的对话系统

### 9.1 系统架构设计

```python
class PragmaticDialogueSystem:
    """
    语用感知的对话系统架构
    
    完整的语用理解-生成流水线
    """
    
    def __init__(self):
        # 组件初始化
        self.intent_classifier = DialogueIntentClassifier()
        self.speech_act_classifier = SpeechActClassifier()
        self.pragmatic_processor = PragmaticPromptEngineer()
        self.politeness_model = PolitenessModel()
        self.deixis_resolver = DeicticResolver()
        self.presupposition_handler = PresuppositionTriggers()
        
        # 对话状态
        self.dialogue_state = {
            'history': [],
            'context': {},
            'user_profile': {},
            'goals': []
        }
    
    def process_user_input(self, utterance):
        """
        处理用户输入
        
        流水线:
        1. 指示语消解
        2. 言语行为识别
        3. 语用推理（隐含意图、预设）
        4. 礼貌分析
        5. 生成响应
        """
        # 1. 更新对话状态
        self.dialogue_state['history'].append({
            'role': 'user',
            'content': utterance,
            'turn': len(self.dialogue_state['history']) // 2 + 1
        })
        
        # 2. 指示语消解
        resolved_utterance, deixis_info = self.deixis_resolver.resolve(utterance)
        
        # 3. 言语行为分类
        speech_act = self.speech_act_classifier.classify(
            resolved_utterance, 
            self.dialogue_state['context']
        )
        
        # 4. 提取预设
        presuppositions = self.presupposition_handler.extract_presuppositions(
            resolved_utterance
        )
        
        # 5. 礼貌分析
        politeness_level = self._analyze_politeness(resolved_utterance)
        
        # 6. 意图识别
        intent = self.intent_classifier.predict_intent(resolved_utterance)
        
        # 7. 生成语用感知提示
        pragmatic_prompt = self.pragmatic_processor.generate_pragmatic_prompt(
            utterance,
            context={
                'situation': self.dialogue_state['context'].get('situation'),
                'relationship': self._infer_relationship()
            }
        )
        
        # 8. 更新对话状态
        analysis_result = {
            'original_utterance': utterance,
            'resolved_utterance': resolved_utterance,
            'deixis_info': deixis_info,
            'speech_act': speech_act,
            'presuppositions': presuppositions,
            'politeness_level': politeness_level,
            'intent': intent
        }
        
        self.dialogue_state['history'].append({
            'role': 'analysis',
            'content': analysis_result
        })
        
        return analysis_result
    
    def generate_response(self, analysis_result, llm_model):
        """生成响应"""
        # 1. 调整响应风格
        politeness_level = analysis_result['politeness_level']
        
        # 2. 构建生成提示
        generation_prompt = self._build_generation_prompt(analysis_result)
        
        # 3. 调用LLM生成
        response = llm_model.generate(generation_prompt)
        
        # 4. 后处理
        response = self._post_process_response(response, politeness_level)
        
        # 5. 保存到历史
        self.dialogue_state['history'].append({
            'role': 'system',
            'content': response
        })
        
        return response
    
    def _analyze_politeness(self, utterance):
        """分析礼貌等级"""
        politeness_markers = {
            'very_high': ['请您', '麻烦您', '万死不辞'],
            'high': ['请', '能否', '拜托'],
            'medium': ['帮我', '麻烦'],
            'low': ['把', '给我'],
            'very_low': ['']  # 命令式
        }
        
        for level, markers in politeness_markers.items():
            for marker in markers:
                if marker in utterance:
                    return level
        
        return 'medium'  # 默认中等
    
    def _infer_relationship(self):
        """推断用户关系"""
        return 'neutral'
    
    def _build_generation_prompt(self, analysis_result):
        """构建生成提示"""
        return f"请回应：{analysis_result['original_utterance']}"
    
    def _post_process_response(self, response, politeness_level):
        """后处理响应"""
        # 可以根据礼貌等级调整响应
        return response
```

### 9.2 语料资源推荐

**语用学语料库**

1. **MRDA (Meeting Recorder Dialogue Act)** - 会议对话行为标注
2. **SWBD (Switchboard)** - 电话对话语料
3. **Dysk** - 俄语对话语料
4. **Trains** - 问题解决对话语料

**讽刺/反语数据集**

1. **SemEval 2018 Task 3** - Irony detection in English tweets
2. **讽刺检测数据集** - 多语言讽刺语料

**多模态语用数据集**

1. **MUStARD** - Multimodal sarcasm detection
2. **CMU-MOSI** - Multimodal sentiment analysis

---

## 参考文献与推荐阅读

1. **经典著作**

   - Austin, J. L. (1962). *How to Do Things with Words*. Oxford University Press.
   - Searle, J. R. (1969). *Speech Acts: An Essay in the Philosophy of Language*. Cambridge University Press.
   - Searle, J. R. (1979). *Expression and Meaning: Studies in the Theory of Speech Acts*. Cambridge University Press.
   - Grice, H. P. (1975). Logic and conversation. *Syntax and Semantics*, 3, 41-58.
   - Grice, H. P. (1978). Further notes on logic and conversation. *Syntax and Semantics*, 9, 113-128.
   - Sperber, D., & Wilson, D. (1986). *Relevance: Communication and Cognition*. Blackwell.
   - Levinson, S. C. (1983). *Pragmatics*. Cambridge University Press.
   - Brown, P., & Levinson, S. C. (1987). *Politeness: Some Universals in Language Usage*. Cambridge University Press.
   - Leech, G. N. (1983). *Principles of Pragmatics*. Longman.

2. **计算语用学文献**

   - Jurafsky, D., & Martin, J. H. (2023). *Speech and Language Processing* (3rd ed.). Chapter on Pragmatics.
   - Asher, N., & Lascarides, A. (2003). *Logics of Conversation*. Cambridge University Press.
   - Burton-Roberts, N. (2013). *Parentheticals*. Cambridge University Press.

3. **神经语用学**

   - Coulson, S. (2006). *Semantic Leaps: Frame-Shifting and Conceptual Blending in Meaning Construction*. Cambridge University Press.
   - Domenech, M., & McNamara, D. S. (2017). The cognitive pragmatics of language. *Psychology of Language and Communication*, 21(1), 1-24.

4. **AI/语用学交叉研究**

   - Zhou, P., et al. (2023). A Survey on Pragmatics in Natural Language Processing. *arXiv*.
   -坎伯格, W. (2022). Pragmatics and Large Language Models. *TACL*.

---

> [!tip] 关联文档
> - [[计算语义学深度指南]]：语义学整体框架
> - [[词向量与分布式语义]]：语义表示方法
> - [[形式语义学基础]]：形式语义学理论
> - [[概念语义学详解]]：概念表示
> - [[分布式语义学详解]]：分布式语义理论
> - [[认知计算]]：认知科学与AI的交叉
> - [[对话系统设计]]：对话系统实践

> [!example] 实践示例
> - 使用spaCy进行意图识别：参考spaCy文档
> - 使用HuggingFace进行讽刺检测：transformers库
> - 语用标注工具： brat, GATE, ELAN
> - 语料库资源： LDC, ELRA
