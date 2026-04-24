---
title: FewShot示例选择
date: 2026-04-24
tags:
  - few-shot
  - example-selection
  - prompting
  - demonstration
  - dynamic-prompting
categories:
  - context-engineering
  - fewshot-selection
---

> [!abstract] 摘要
> Few-shot prompting是让AI"照葫芦画瓢"的神器，但示例选得好不好，直接决定效果。本文为零基础小白详细讲解：什么是Few-shot、怎么选示例、怎么动态选择、k值怎么定。看完就能上手，用代码示例让你学得明明白白。

## 先搞清楚：什么是Few-shot？

### 打比方：学做菜

想象你要学做宫保鸡丁：

**Zero-shot（零样本）**：只看菜名"宫保鸡丁"，自己琢磨怎么做
```
结果：可能做出来四不像
```

**One-shot（一样本）**：看一个宫保鸡丁的做法，然后自己做
```
结果：照着做，大概率能做个七八分像
```

**Few-shot（少样本）**：看3-5个不同厨师做的宫保鸡丁，然后自己做
```
结果：综合各家之长，可能做出自己的风格
```

**这就是Few-shot的核心思想：给AI几个示例，让它"照葫芦画瓢"！**

### Few-shot的本质

```
用户：帮我把英文翻译成中文

Zero-shot:
用户：请把以下英文翻译成中文：Hello, world!

One-shot:
用户：请把英文翻译成中文。
示例：
输入：Good morning
输出：早上好
输入：Hello
输出：你好

Few-shot:
用户：请把英文翻译成中文。
示例：
输入：Good morning → 输出：早上好
输入：Nice to meet you → 输出：很高兴认识你
输入：Thank you very much → 输出：非常感谢
输入：Hello
输出：
```

### Few-shot为什么有效？

1. **示范效应**：AI能从示例中学习输出格式
2. **模式识别**：AI能从示例中推断任务规则
3. **减少歧义**：示例比文字描述更清晰

---

## 一、Few-shot基础：三种模式对比

### Zero-shot vs One-shot vs Few-shot

| 模式 | 示例数量 | 适用场景 | 效果 |
|------|---------|---------|------|
| Zero-shot | 0个 | 简单任务、强大模型 | 依赖模型固有能力 |
| One-shot | 1个 | 格式说明、简单分类 | 明确输出格式 |
| Few-shot | 2-10个 | 中等复杂度任务 | 平衡效果与成本 |
| Multi-shot | 10+个 | 复杂模式学习 | 高成本、边际效益递减 |

### 代码示例：基本调用

```python
# 一个完整的Few-shot调用示例
import anthropic

client = anthropic.Anthropic()

def fewshot_translation(query: str, examples: list) -> str:
    """
    Few-shot翻译示例
    
    examples格式：[{"input": "英文", "output": "中文"}]
    """
    
    # 构建Few-shot prompt
    prompt_parts = ["请将以下英文翻译成中文。\n"]
    prompt_parts.append("示例：")
    
    for ex in examples:
        prompt_parts.append(f"输入：{ex['input']}")
        prompt_parts.append(f"输出：{ex['output']}")
    
    prompt_parts.append(f"\n请翻译：\n输入：{query}")
    prompt_parts.append("输出：")
    
    prompt = "\n".join(prompt_parts)
    
    # 调用API
    message = client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )
    
    return message.content

# 使用示例
examples = [
    {"input": "Good morning", "output": "早上好"},
    {"input": "How are you?", "output": "你好吗？"},
    {"input": "Thank you very much", "output": "非常感谢"}
]

result = fewshot_translation("Nice to meet you", examples)
print(result)  # 应该是：很高兴认识你
```

---

## 二、示例池构建：高质量示例从哪来？

### 示例来源

```
┌─────────────────────────────────────────────────────────────┐
│                   示例池构建流程                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│   1️⃣ 收集原始数据                                          │
│      - 历史问答记录                                         │
│      - 专家标注数据                                         │
│      - 人工构造示例                                         │
│                                                              │
│   2️⃣ 质量筛选                                              │
│      - 去除错误标注                                         │
│      - 去除模糊不清的示例                                   │
│      - 确保输入-输出映射正确                                 │
│                                                              │
│   3️⃣ 多样性覆盖                                            │
│      - 覆盖不同类型/难度                                    │
│      - 覆盖边界情况                                         │
│      - 覆盖不同表达方式                                      │
│                                                              │
│   4️⃣ 示例池存储                                            │
│      - 结构化存储（JSON/数据库）                            │
│      - 支持快速检索                                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 示例池数据结构

```python
from dataclasses import dataclass
from typing import List, Dict, Optional
import json

@dataclass
class Example:
    """单个示例"""
    id: str
    input_text: str
    output_text: str
    metadata: Dict  # 标签、难度、来源等

class ExamplePool:
    """示例池管理器"""
    
    def __init__(self):
        self.examples: List[Example] = []
        self._embeddings = None  # 缓存
    
    def add(self, example: Example):
        """添加示例"""
        self.examples.append(example)
        self._embeddings = None  # 使缓存失效
    
    def add_batch(self, examples: List[Example]):
        """批量添加"""
        self.examples.extend(examples)
        self._embeddings = None
    
    def load_from_json(self, filepath: str):
        """从JSON文件加载"""
        with open(filepath, 'r', encoding='utf-8') as f:
            data = json.load(f)
        for item in data:
            self.add(Example(
                id=item['id'],
                input_text=item['input'],
                output_text=item['output'],
                metadata=item.get('metadata', {})
            ))
    
    def save_to_json(self, filepath: str):
        """保存到JSON文件"""
        data = [
            {
                'id': ex.id,
                'input': ex.input_text,
                'output': ex.output_text,
                'metadata': ex.metadata
            }
            for ex in self.examples
        ]
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)
    
    def filter_by_metadata(self, **kwargs) -> List[Example]:
        """根据元数据筛选"""
        results = []
        for ex in self.examples:
            match = True
            for key, value in kwargs.items():
                if ex.metadata.get(key) != value:
                    match = False
                    break
            if match:
                results.append(ex)
        return results
```

### 示例质量标准

| 标准 | 说明 | 检查方法 |
|------|------|---------|
| **代表性** | 覆盖任务的典型场景 | 聚类分析 |
| **准确性** | 输入-输出映射正确 | 人工审核 |
| **清晰性** | 格式规范、无歧义 | 格式验证 |
| **多样性** | 不同类型/难度均衡 | 分布检查 |
| **无泄露** | 不含测试集信息 | 数据隔离 |

```python
class ExampleQualityFilter:
    """示例质量过滤器"""
    
    def __init__(self, min_length: int = 10, max_length: int = 2000):
        self.min_length = min_length
        self.max_length = max_length
    
    def is_quality_example(self, example: Example) -> bool:
        """检查示例质量"""
        # 长度检查
        input_len = len(example.input_text)
        output_len = len(example.output_text)
        
        if not (self.min_length <= input_len <= self.max_length):
            return False
        if not (self.min_length <= output_len <= self.max_length):
            return False
        
        # 格式检查：必须有清晰的结构标记
        required_markers = ['##', '###', '**']
        has_structure = any(marker in example.input_text for marker in required_markers)
        if not has_structure:
            return False
        
        # 去重检查
        if self._is_duplicate(example):
            return False
        
        return True
    
    def _is_duplicate(self, example: Example) -> bool:
        """检查是否重复"""
        for ex in self.examples:
            if ex.input_text == example.input_text:
                return True
        return False
```

---

## 三、示例选择策略：怎么选出最好的示例？

### 策略1：基于相似度选择

这是最常用的策略——**找和问题最像的示例**。

```python
from sentence_transformers import SentenceTransformer
import numpy as np

class SimilarityBasedSelector:
    """基于相似度的示例选择器"""
    
    def __init__(self, model_name: str = 'all-MiniLM-L6-v2'):
        """
        初始化选择器
        
        使用sentence-transformers进行语义嵌入
        model_name可选：'all-MiniLM-L6-v2', ' paraphrase-multilingual-MiniLM-L12-v2'等
        """
        self.model = SentenceTransformer(model_name)
        self._pool_embeddings = None
        self.pool = None
    
    def build_index(self, pool: ExamplePool):
        """
        为示例池构建索引
        
        这一步很关键！必须在选择之前调用
        """
        texts = [ex.input_text for ex in pool.examples]
        self._pool_embeddings = self.model.encode(texts, show_progress_bar=True)
        self.pool = pool
    
    def select_k_nearest(
        self, 
        query: str, 
        k: int = 5,
        threshold: float = 0.0
    ) -> List[Example]:
        """
        选择k个最相似的示例
        
        Args:
            query: 用户当前的问题
            k: 选择多少个示例
            threshold: 相似度阈值，低于此值的示例不会选中
        
        Returns:
            选中的示例列表
        """
        if self._pool_embeddings is None:
            raise ValueError("需要先调用build_index构建索引！")
        
        # 1️⃣ 计算查询的嵌入向量
        query_embedding = self.model.encode([query])
        
        # 2️⃣ 计算与所有示例的余弦相似度
        similarities = self._compute_similarity(query_embedding, self._pool_embeddings)
        
        # 3️⃣ 按相似度排序
        sorted_indices = np.argsort(similarities)[::-1]
        
        # 4️⃣ 选择top-k
        selected = []
        for idx in sorted_indices:
            if len(selected) >= k:
                break
            if similarities[idx] >= threshold:
                selected.append(self.pool.examples[idx])
        
        return selected
    
    def _compute_similarity(self, query_emb, pool_embs) -> np.ndarray:
        """计算余弦相似度"""
        # 归一化
        query_norm = query_emb / np.linalg.norm(query_emb, axis=1, keepdims=True)
        pool_norm = pool_embs / np.linalg.norm(pool_embs, axis=1, keepdims=True)
        # 余弦相似度
        return (query_norm @ pool_norm.T).flatten()

# 使用示例
pool = ExamplePool()
pool.load_from_json("examples/translation_pool.json")

selector = SimilarityBasedSelector()
selector.build_index(pool)

# 选择最相关的3个示例
query = "How to say goodbye in Chinese?"
selected = selector.select_k_nearest(query, k=3)

for ex in selected:
    print(f"相似度得分: {ex.score}")
    print(f"输入: {ex.input_text}")
    print(f"输出: {ex.output_text}")
```

### 策略2：多样性采样

有时候光选最相似的不够，还需要**保证多样性**——选的示例要覆盖不同方面。

```python
class DiverseSelector:
    """多样性感知的选择器"""
    
    def __init__(self, diversity_weight: float = 0.3):
        """
        多样性权重：0=只看相似度，1=只看多样性
        """
        self.diversity_weight = diversity_weight
    
    def select_diverse(
        self,
        pool: List[Example],
        query: str,
        k: int = 5,
        category_key: str = 'category'
    ) -> List[Example]:
        """选择既相似又多样的示例"""
        
        # 按类别分组
        from collections import defaultdict
        categories = defaultdict(list)
        for i, ex in enumerate(pool):
            cat = ex.metadata.get(category_key, 'unknown')
            categories[cat].append((i, ex))
        
        # 从每个类别选择
        selected = []
        categories_to_sample = min(len(categories), k)
        selected_cats = list(categories.keys())[:categories_to_sample]
        
        for cat in selected_cats:
            cat_examples = categories[cat]
            # 从每个类别随机选一个
            chosen = random.choice(cat_examples)
            selected.append(chosen[1])
        
        return selected[:k]
```

### 策略3：最大边际相关性（MMR）

MMR是一个经典的平衡相似度和多样性的方法：

```
MMR核心公式：
选择同时满足：
1. 和查询高相似度
2. 和已选示例低相似度（多样化）
```

```python
class MMRSelector:
    """最大边际相关性选择器 (Maximal Marginal Relevance)"""
    
    def __init__(self, lambda_param: float = 0.5):
        """
        lambda_param: 相似度与多样性的权衡
        - 0 = 只看相似度
        - 0.5 = 平衡
        - 1 = 只看多样性
        """
        self.lambda_param = lambda_param
    
    def select(
        self, 
        query_emb: np.ndarray, 
        pool_embs: np.ndarray, 
        k: int,
        pool: ExamplePool
    ) -> List[Example]:
        """MMR选择"""
        n = len(pool_embs)
        selected = []
        remaining = set(range(n))
        
        while len(selected) < k and remaining:
            best_score = -float('inf')
            best_idx = None
            
            for idx in remaining:
                # 与查询的相似度
                sim_to_query = self._cosine_sim(query_emb, pool_embs[idx])
                
                # 与已选示例的最大相似度（多样性惩罚）
                if selected:
                    max_sim_to_selected = max(
                        self._cosine_sim(pool_embs[idx], pool_embs[s]) 
                        for s in selected
                    )
                else:
                    max_sim_to_selected = 0
                
                # MMR分数
                mmr_score = (
                    self.lambda_param * sim_to_query - 
                    (1 - self.lambda_param) * max_sim_to_selected
                )
                
                if mmr_score > best_score:
                    best_score = mmr_score
                    best_idx = idx
            
            if best_idx is not None:
                selected.append(best_idx)
                remaining.remove(best_idx)
        
        return [pool.examples[i] for i in selected]
    
    @staticmethod
    def _cosine_sim(a: np.ndarray, b: np.ndarray) -> float:
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b) + 1e-8)
```

---

## 四、动态Few-shot：根据情况自动调整

### 自适应k值选择

不同问题可能需要不同数量的示例：

```
简单问题："翻译hello" → 1-2个示例足够
复杂问题："翻译这段技术文档，包含多个专业术语" → 可能需要5+个示例
```

```python
class DynamicFewShot:
    """动态Few-shot选择器"""
    
    def __init__(
        self,
        example_pool: ExamplePool,
        selector,  # 任意选择器实例
        k_range: tuple = (2, 8)
    ):
        self.pool = example_pool
        self.selector = selector
        self.k_min, self.k_max = k_range
    
    def build_prompt(
        self, 
        query: str, 
        task_description: str,
        output_format: str = None
    ) -> str:
        """构建动态Few-shot提示"""
        
        # 1️⃣ 根据问题复杂度自适应选择k
        k = self._adaptive_k(query)
        
        # 2️⃣ 选择示例
        selected = self.selector.select_k_nearest(query, k=k)
        
        # 3️⃣ 构建提示
        prompt_parts = [task_description]
        
        if output_format:
            prompt_parts.append(f"\n输出格式：\n{output_format}")
        
        prompt_parts.append("\n\n## 示例：")
        
        for ex in selected:
            prompt_parts.append(f"\n输入：\n{ex.input_text}")
            prompt_parts.append(f"\n输出：\n{ex.output_text}")
        
        prompt_parts.append(f"\n\n## 请根据以上示例完成以下任务：")
        prompt_parts.append(f"\n输入：\n{query}")
        prompt_parts.append("\n输出：")
        
        return '\n'.join(prompt_parts)
    
    def _adaptive_k(self, query: str) -> int:
        """
        根据问题复杂度自适应选择k
        
        复杂度指标：
        - 长度
        - 关键词（详细、分析、比较等）
        - 专业术语
        """
        complexity_indicators = [
            '详细', '全面', '深入', '分析', '比较',
            '解释', '评估', '论述', '探讨', '详细说明'
        ]
        
        # 计算复杂度得分
        score = sum(1 for indicator in complexity_indicators 
                   if indicator in query)
        
        # 额外：长度因素
        if len(query) > 100:
            score += 1
        if len(query) > 200:
            score += 1
        
        # 映射到k范围
        if score <= 1:
            return self.k_min
        elif score >= 5:
            return self.k_max
        else:
            return self.k_min + int((score - 1) / 4 * (self.k_max - self.k_min))
```

### 基于难度的选择

```python
class DifficultyAwareSelector:
    """难度感知的选择器"""
    
    def __init__(self, difficulty_key: str = 'difficulty'):
        self.difficulty_key = difficulty_key
    
    def select_by_difficulty(
        self,
        pool: List[Example],
        query_difficulty: int,
        k: int = 5,
        tolerance: int = 1
    ) -> List[Example]:
        """
        选择与问题难度匹配的示例
        
        如果问题是简单的，就选简单示例；
        如果问题是复杂的，就选复杂示例
        """
        # 按难度分组
        by_difficulty = defaultdict(list)
        for ex in pool:
            diff = ex.metadata.get(self.difficulty_key, 3)  # 默认中等难度
            by_difficulty[diff].append(ex)
        
        # 选择难度接近的示例
        target = query_difficulty
        selected = []
        
        # 先从精确匹配开始
        if target in by_difficulty:
            selected.extend(by_difficulty[target])
        
        # 再扩展到容差范围
        for offset in range(1, tolerance + 1):
            for delta in [-offset, offset]:
                diff_level = target + delta
                if diff_level in by_difficulty and len(selected) < k:
                    needed = k - len(selected)
                    candidates = [e for e in by_difficulty[diff_level] if e not in selected]
                    selected.extend(candidates[:needed])
        
        return selected[:k]
```

---

## 五、k值选择：几个示例最好？

### k值选择指南

| 场景 | 推荐k值 | 选择策略 | 注意事项 |
|------|---------|---------|---------|
| 简单分类 | 1-3 | 最相似 | 1-shot往往足够 |
| 格式转换 | 2-3 | 最相似 | 确保格式一致 |
| 复杂推理 | 4-8 | 多样性 + 相似度 | 覆盖推理路径 |
| 代码生成 | 3-5 | 语义相似 | 包含边界情况 |
| 创意写作 | 2-4 | 质量优先 | 避免风格偏移 |

### 找最优k值

```python
class OptimalKFinder:
    """最优k值查找器"""
    
    def __init__(self, test_pool: List[Example], eval_func: callable):
        """
        test_pool: 测试用的示例池
        eval_func: 评估函数，输入示例列表，返回准确率等指标
        """
        self.test_pool = test_pool
        self.eval_func = eval_func
    
    def find_optimal_k(
        self,
        k_range: range,
        n_trials: int = 3
    ) -> Dict[int, Dict]:
        """
        通过实验找到最优k值
        
        Args:
            k_range: 要测试的k值范围，如range(1, 10)
            n_trials: 每个k值测试几次取平均
        
        Returns:
            每个k值的评估结果
        """
        results = {}
        
        for k in k_range:
            scores = []
            for trial in range(n_trials):
                # 随机分割
                train, test = self._split_pool(k)
                
                # 评估
                score = self._evaluate_with_k(train, test, k)
                scores.append(score)
            
            results[k] = {
                'mean': np.mean(scores),
                'std': np.std(scores),
                'scores': scores
            }
        
        return results
    
    def _split_pool(self, k: int) -> tuple:
        """分割示例池"""
        selected = random.sample(self.test_pool, min(k, len(self.test_pool)))
        remaining = [ex for ex in self.test_pool if ex not in selected]
        return selected, remaining[:20]
    
    def _evaluate_with_k(self, examples: List[Example], test_set: List[Example], k: int) -> float:
        """评估给定k值的效果"""
        # 构建提示
        prompt = self._build_prompt(examples)
        
        # 在测试集上评估
        correct = 0
        for test_ex in test_set:
            response = self._call_llm(prompt, test_ex.input_text)
            if self._check_correct(response, test_ex.output_text):
                correct += 1
        
        return correct / len(test_set) if test_set else 0
```

### 最佳实践建议

```
┌─────────────────────────────────────────────────────────────┐
│                   Few-shot k值选择建议                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ✅ 从少量开始                                               │
│     先尝试k=2-3，效果不好再增加                              │
│                                                              │
│  ✅ 监控成本                                                │
│     k值翻倍 = token消耗翻倍 = 成本翻倍                        │
│                                                              │
│  ✅ 质量 > 数量                                             │
│     5个高质量示例 > 20个低质量示例                           │
│                                                              │
│  ✅ 动态调整                                                │
│     不同问题可能适合不同的k值                                  │
│                                                              │
│  ✅ 多样性很重要                                            │
│     既要选相似的，也要覆盖不同类型                            │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 六、Chain-of-Thought集成

### 什么是Chain-of-Thought？

Chain-of-Thought（CoT，思维链）就是**让AI展示推理过程**：

```
普通Few-shot：
输入：2 + 2 = ?
输出：4

CoT Few-shot：
输入：2 + 2 = ?
思考：2加2，就是数两个2，先数2，再数2，一共有4个
输出：4
```

### CoT Few-shot实现

```python
class CoTFewShot:
    """Chain-of-Thought集成的Few-shot"""
    
    def __init__(self, pool: ExamplePool):
        self.pool = pool
    
    def build_cot_prompt(
        self,
        query: str,
        k: int = 3
    ) -> str:
        """构建带推理过程的Few-shot提示"""
        
        # 选择包含推理过程的示例
        selected = self._select_cot_examples(query, k)
        
        prompt_parts = ["请逐步推理并回答问题。\n"]
        
        for ex in selected:
            prompt_parts.append(f"问题：{ex.input_text}")
            # 如果有推理过程，加上
            if 'reasoning' in ex.metadata:
                prompt_parts.append(f"推理过程：{ex.metadata['reasoning']}")
            prompt_parts.append(f"答案：{ex.output_text}\n")
        
        prompt_parts.append(f"问题：{query}")
        prompt_parts.append("推理过程：")
        
        return '\n'.join(prompt_parts)
    
    def _select_cot_examples(self, query: str, k: int) -> List[Example]:
        """选择包含推理过程的示例"""
        # 只选择有推理过程的示例
        cot_pool = [
            ex for ex in self.pool.examples 
            if 'reasoning' in ex.metadata
        ]
        
        if not cot_pool:
            return random.sample(self.pool.examples, min(k, len(self.pool.examples)))
        
        # 按相似度选择
        # （简化实现）
        return random.sample(cot_pool, min(k, len(cot_pool)))
```

---

## 七、实战配置模板

### 完整Few-shot配置

```python
# Few-shot配置模板
FEWSHOT_CONFIG = {
    # 默认选择几个示例
    "default_k": 3,
    
    # k值范围
    "k_range": (2, 8),
    
    # 选择器配置
    "selector": {
        "type": "mmr",  # 可选: "mmr", "similarity", "diverse", "balanced"
        "similarity_weight": 0.5,  # MMR的lambda参数
        "category_key": "category",  # 多样性分组的key
        "diversity_weight": 0.3   # 多样性权重
    },
    
    # 示例池配置
    "example_pool": {
        "path": "examples/fewshot_pool.json",
        "min_length": 20,
        "max_length": 1500,
        "required_fields": ["input", "output", "category"]
    },
    
    # 高级选项
    "cot_enabled": False,  # 是否启用Chain-of-Thought
    "balance_labels": True  # 是否平衡标签
}

class FewShotEngine:
    """统一的Few-shot引擎"""
    
    def __init__(self, config: dict):
        # 加载示例池
        self.pool = ExamplePool()
        self.pool.load_from_json(config["example_pool"]["path"])
        
        # 初始化选择器
        selector_type = config["selector"]["type"]
        if selector_type == "mmr":
            self.selector = MMRSelector(
                lambda_param=config["selector"]["similarity_weight"]
            )
        elif selector_type == "diverse":
            self.selector = DiverseSelector(
                diversity_weight=config["selector"]["diversity_weight"]
            )
        elif selector_type == "balanced":
            self.selector = BalancedSelector()
        else:
            self.selector = SimilarityBasedSelector()
        
        # 构建索引
        if hasattr(self.selector, 'build_index'):
            self.selector.build_index(self.pool)
        
        self.k = config["default_k"]
        self.cot_enabled = config.get("cot_enabled", False)
    
    def generate(self, query: str, task_desc: str) -> str:
        """生成Few-shot提示"""
        if self.cot_enabled:
            return CoTFewShot(self.pool).build_cot_prompt(query, k=self.k)
        else:
            return DynamicFewShot(
                self.pool, 
                self.selector, 
                k_range=(self.k, self.k)
            ).build_prompt(query, task_desc)

# 使用示例
engine = FewShotEngine(FEWSHOT_CONFIG)

task_desc = "请将以下英文翻译成中文。"
query = "What's the weather like today?"

prompt = engine.generate(query, task_desc)
print(prompt)
```

---

## 八、一图总结

```
┌─────────────────────────────────────────────────────────────┐
│                    Few-shot选择速查表                           │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📊 三种模式                                                │
│  ├─ Zero-shot：0个示例，依赖模型本身                        │
│  ├─ One-shot：1个示例，明确格式                            │
│  └─ Few-shot：2-10个示例，平衡效果与成本                    │
│                                                              │
│  🎯 选择策略                                               │
│  ├─ 相似度选择：选和问题最像的                             │
│  ├─ 多样性选择：覆盖不同类型                               │
│  ├─ MMR：平衡相似度与多样性                               │
│  └─ 动态选择：根据复杂度调整                               │
│                                                              │
│  ⚙️ k值选择                                               │
│  ├─ 简单任务：1-3个示例                                   │
│  ├─ 复杂推理：4-8个示例                                    │
│  └─ 质量 > 数量                                           │
│                                                              │
│  💡 最佳实践                                               │
│  ├─ 从少量开始，效果不好再增加                             │
│  ├─ 示例要高质量、格式正确                                  │
│  ├─ 覆盖不同类型和难度                                     │
│  └─ 考虑CoT展示推理过程                                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 相关主题

- [[上下文窗口管理]] - 示例会占用窗口空间
- [[上下文结构化]] - 良好的格式让示例更有效
- [[提示词工程基础]] - Few-shot是提示工程的一部分

---

## 参考文献

1. Brown, T. B., et al. (2020). **Language Models are Few-Shot Learners.** NeurIPS.
2. Liu, J., et al. (2022). **What Makes Good In-Context Examples for GPT-3?** ACL Workshop.
3. Rubin, O., et al. (2022). **Learning to Retrieve Prompts for In-Context Learning.** NAACL.
4. Su, H., et al. (2022). **Selective Annotation Makes Language Models Better Few-Shot Learners.** ICLR.
5. Wei, J., et al. (2022). **Chain-of-Thought Prompting Elicits Reasoning in Large Language Models.** NeurIPS.
