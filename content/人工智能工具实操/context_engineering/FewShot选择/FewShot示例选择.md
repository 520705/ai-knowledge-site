---
title: FewShot示例选择
date: 2026-04-18
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
> Few-shot prompting是充分利用LLM能力的关键技术，而示例选择的质量直接决定最终效果。本文深入探讨示例池构建、选择策略、动态Few-shot机制以及数量权衡的实战技巧，帮助开发者构建高效、精准的示例系统。

## 关键词速览

| 术语 | 英文 | 说明 |
|------|------|------|
| Few-shot | Few-shot | 少样本提示技术 |
| 示例池 | Example Pool | 候选示例的集合 |
| 语义相似度 | Semantic Similarity | 示例与查询的匹配程度 |
| 动态选择 | Dynamic Selection | 基于上下文选择示例 |
| KNN | K-Nearest Neighbors | K近邻选择算法 |
| 多样性采样 | Diversity Sampling | 确保示例覆盖多样性 |
| 标签平衡 | Label Balancing | 均衡正负样本比例 |
| 链式思维 | Chain of Thought | 推理过程展示 |
| 零样本 | Zero-shot | 无示例基准 |
| 元学习 | Meta-learning | 学习如何学习 |

## 一、Few-shot核心原理

### 1.1 为什么Few-shot有效

Few-shot prompting的有效性源于大语言模型的以下特性：

1. **上下文学习能力 (In-context Learning)**：LLM能够从上下文中提取模式并泛化到新样本
2. **隐式梯度下降**：通过观察示例，模型隐式地"调整"了输出分布
3. **任务概念学习**：示例帮助模型理解具体任务的需求和格式

> [!note] 核心机制
> Few-shot不是简单的"记忆-匹配"过程，而是模型利用有限的示例信息推断任务本质的过程。示例的质量比数量更重要。

### 1.2 Few-shot vs Zero-shot vs One-shot

| 模式 | 示例数量 | 适用场景 | 效果 |
|------|---------|---------|------|
| Zero-shot | 0 | 简单任务、强模型 | 依赖模型固有能力 |
| One-shot | 1 | 格式说明、简单分类 | 明确输出格式 |
| Few-shot | 2-10 | 中等复杂度任务 | 平衡效果与成本 |
| Multi-shot | 10+ | 复杂模式学习 | 高成本、边际效益递减 |

## 二、示例池构建

### 2.1 示例来源

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
        self._embeddings = None
        
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
```

### 2.2 示例质量标准

高质量示例应满足以下标准：

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
            
        # 格式检查
        if not self._check_format(example):
            return False
            
        # 去重检查
        if self._is_duplicate(example):
            return False
            
        return True
    
    def _check_format(self, example: Example) -> bool:
        """检查格式规范"""
        # 检查是否包含必要的结构标记
        required_markers = ['##', '###', '**']
        has_structure = any(marker in example.input_text for marker in required_markers)
        return has_structure or example.metadata.get('format_verified', False)
```

## 三、示例选择策略

### 3.1 基于相似度的选择

```python
from sentence_transformers import SentenceTransformer
import numpy as np

class SimilarityBasedSelector:
    """基于相似度的示例选择器"""
    
    def __init__(self, model_name: str = 'all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self._pool_embeddings = None
        
    def build_index(self, pool: ExamplePool):
        """构建嵌入索引"""
        texts = [ex.input_text for ex in pool.examples]
        self._pool_embeddings = self.model.encode(texts, show_progress_bar=True)
        self.pool = pool
        
    def select_k_nearest(
        self, 
        query: str, 
        k: int = 5,
        threshold: float = 0.0
    ) -> List[Example]:
        """选择k个最相似的示例"""
        if self._pool_embeddings is None:
            raise ValueError("需要先调用build_index构建索引")
            
        # 计算查询嵌入
        query_embedding = self.model.encode([query])
        
        # 计算余弦相似度
        similarities = self._compute_similarity(query_embedding, self._pool_embeddings)
        
        # 排序并选择
        sorted_indices = np.argsort(similarities)[::-1]
        
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
        return (query_norm @ pool_norm.T).flatten()
```

### 3.2 多样性采样策略

```python
import random
from collections import defaultdict

class DiverseSelector:
    """多样性感知的选择器"""
    
    def __init__(self, diversity_weight: float = 0.3):
        self.diversity_weight = diversity_weight  # 多样性权重
        
    def select_diverse(
        self,
        pool: List[Example],
        query: str,
        k: int = 5,
        category_key: str = 'category'
    ) -> List[Example]:
        """选择既相似又多样的示例"""
        
        # 按类别分组
        categories = defaultdict(list)
        for i, ex in enumerate(pool):
            cat = ex.metadata.get(category_key, 'unknown')
            categories[cat].append((i, ex))
        
        # 从每个类别选择
        selected = []
        categories_to_sample = len(categories) if len(categories) <= k else k
        selected_cats = random.sample(list(categories.keys()), categories_to_sample)
        
        for cat in selected_cats:
            cat_examples = categories[cat]
            # 从每个类别选一个
            chosen = random.choice(cat_examples)
            selected.append(chosen[1])
            
        return selected[:k]

class MMRSelector:
    """最大边际相关性选择器 (Maximal Marginal Relevance)"""
    
    def __init__(self, lambda_param: float = 0.5):
        """
        lambda_param: 相似度与多样性之间的权衡
                     0 = 只看相似度, 1 = 只看多样性
        """
        self.lambda_param = lambda_param
        
    def select(self, query_emb: np.ndarray, pool_embs: np.ndarray, k: int) -> List[int]:
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
                
                # 与已选示例的最大相似度
                if selected:
                    sim_to_selected = max(
                        self._cosine_sim(pool_embs[idx], pool_embs[s]) 
                        for s in selected
                    )
                else:
                    sim_to_selected = 0
                    
                # MMR分数
                mmr_score = self.lambda_param * sim_to_query - \
                           (1 - self.lambda_param) * sim_to_selected
                
                if mmr_score > best_score:
                    best_score = mmr_score
                    best_idx = idx
                    
            if best_idx is not None:
                selected.append(best_idx)
                remaining.remove(best_idx)
                
        return selected
    
    def _cosine_sim(self, a: np.ndarray, b: np.ndarray) -> float:
        return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b) + 1e-8)
```

### 3.3 标签平衡选择

```python
class BalancedSelector:
    """标签平衡的选择器"""
    
    def __init__(self, label_key: str = 'label'):
        self.label_key = label_key
        
    def select_balanced(
        self,
        pool: List[Example],
        k: int,
        prefer_positive: bool = True
    ) -> List[Example]:
        """均衡正负样本的选择"""
        
        # 分组
        positives = [ex for ex in pool if ex.metadata.get(self.label_key) == 1]
        negatives = [ex for ex in pool if ex.metadata.get(self.label_key) == 0]
        
        # 均衡分配
        if prefer_positive:
            n_positive = (k + 1) // 2
            n_negative = k // 2
        else:
            n_positive = k // 2
            n_negative = (k + 1) // 2
            
        selected = []
        selected.extend(random.sample(positives, min(n_positive, len(positives))))
        selected.extend(random.sample(negatives, min(n_negative, len(negatives))))
        
        # 填充
        while len(selected) < k and pool:
            remaining = [ex for ex in pool if ex not in selected]
            if remaining:
                selected.append(random.choice(remaining))
            else:
                break
                
        random.shuffle(selected)
        return selected[:k]
```

## 四、动态Few-shot机制

### 4.1 实时选择器

```python
class DynamicFewShot:
    """动态Few-shot选择器"""
    
    def __init__(
        self,
        example_pool: ExamplePool,
        selector: object,
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
        
        # 根据查询复杂度自适应选择k
        k = self._adaptive_k(query)
        
        # 选择示例
        selected = self.selector.select_k_nearest(query, k=k)
        
        # 构建提示
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
        """根据查询复杂度自适应选择k"""
        complexity_indicators = [
            '详细', '全面', '深入', '分析', '比较',
            '解释', '评估', '论述', '探讨'
        ]
        
        score = sum(1 for indicator in complexity_indicators if indicator in query)
        
        # 映射到k范围
        if score <= 1:
            return self.k_min
        elif score >= 5:
            return self.k_max
        else:
            return self.k_min + int((score - 1) / 4 * (self.k_max - self.k_min))
```

### 4.2 基于难度的选择

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
        """选择与查询难度匹配的示例"""
        
        # 按难度分组
        by_difficulty = defaultdict(list)
        for ex in pool:
            diff = ex.metadata.get(self.difficulty_key, 3)
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

## 五、数量权衡与最佳实践

### 5.1 k值选择的艺术

```python
class OptimalKFinder:
    """最优k值查找器"""
    
    def __init__(self, test_pool: List[Example], eval_func: callable):
        self.test_pool = test_pool
        self.eval_func = eval_func  # 评估函数，返回准确率等指标
        
    def find_optimal_k(
        self,
        k_range: range,
        n_trials: int = 3
    ) -> Dict[int, Dict]:
        """通过实验找到最优k值"""
        results = {}
        
        for k in k_range:
            scores = []
            for trial in range(n_trials):
                # 随机分割
                train, test = self._split_pool(k)
                
                # 构建提示并评估
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
        return selected, remaining[:20]  # 留出测试集
    
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

### 5.2 最佳实践指南

| 场景 | 推荐k值 | 选择策略 | 注意事项 |
|------|---------|---------|---------|
| 简单分类 | 1-3 | 最相似 | 1-shot往往足够 |
| 格式转换 | 2-3 | 最相似 | 确保格式一致 |
| 复杂推理 | 4-8 | 多样性 + 相似度 | 覆盖推理路径 |
| 代码生成 | 3-5 | 语义相似 | 包含边界情况 |
| 创意写作 | 2-4 | 质量优先 | 避免风格偏移 |

> [!tip] 实用建议
> 1. **从少量开始**：先尝试k=2-3，效果不好再增加
> 2. **监控成本**：k值翻倍意味着token消耗翻倍
> 3. **质量 > 数量**：5个高质量示例 > 20个低质量示例
> 4. **动态调整**：不同查询可能适合不同的k值

### 5.3 Chain-of-Thought集成

```python
class CoTFewShot:
    """Chain-of-Thought集成的Few-shot"""
    
    def __init__(self, pool: ExamplePool):
        self.pool = pool
        
    def build_cot_prompt(
        self,
        query: str,
        k: int = 3,
        include_intermediate: bool = True
    ) -> str:
        """构建带推理过程的Few-shot提示"""
        
        # 选择包含推理过程的示例
        selected = self._select_cot_examples(query, k)
        
        prompt_parts = ["请逐步推理并回答问题。\n"]
        
        for ex in selected:
            prompt_parts.append(f"问题：{ex.input_text}")
            if include_intermediate:
                prompt_parts.append(f"推理过程：{ex.metadata.get('reasoning', '')}")
            prompt_parts.append(f"答案：{ex.output_text}\n")
            
        prompt_parts.append(f"问题：{query}")
        prompt_parts.append("推理过程：")
        
        return '\n'.join(prompt_parts)
    
    def _select_cot_examples(self, query: str, k: int) -> List[Example]:
        """选择适合的CoT示例"""
        # 只选择有推理过程的示例
        cot_pool = [
            ex for ex in self.pool.examples 
            if 'reasoning' in ex.metadata
        ]
        
        # 按相似度选择
        # 简化实现
        return random.sample(cot_pool, min(k, len(cot_pool)))
```

## 六、实战配置模板

```python
# Few-shot配置模板
FEWSHOT_CONFIG = {
    "default_k": 3,
    "k_range": (2, 8),
    "selector": {
        "type": "mmr",  # 或 "similarity", "diverse", "balanced"
        "similarity_weight": 0.5,
        "category_key": "category",
        "diversity_weight": 0.3
    },
    "example_pool": {
        "path": "examples/fewshot_pool.json",
        "min_length": 20,
        "max_length": 1500,
        "required_fields": ["input", "output", "category"]
    },
    "cot_enabled": False,
    "balance_labels": True
}

class FewShotEngine:
    """统一的Few-shot引擎"""
    
    def __init__(self, config: dict):
        self.pool = ExamplePool()
        self.pool.load_from_json(config["example_pool"]["path"])
        
        # 初始化选择器
        selector_type = config["selector"]["type"]
        if selector_type == "mmr":
            self.selector = MMRSelector(lambda_param=config["selector"]["similarity_weight"])
        elif selector_type == "diverse":
            self.selector = DiverseSelector(diversity_weight=config["selector"]["diversity_weight"])
        elif selector_type == "balanced":
            self.selector = BalancedSelector()
        else:
            self.selector = SimilarityBasedSelector()
            
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
```

## 七、相关主题

- [[上下文窗口管理]]
- [[上下文结构化]]
- [[Markdown格式最佳实践]]
- [[分隔符与标记系统]]
- [[提示词工程基础]]

## 八、参考文献

1. Brown, T. B., et al. (2020). Language Models are Few-Shot Learners. *NeurIPS*.
2. Liu, J., et al. (2022). What Makes Good In-Context Examples for GPT-3? *ACL Workshop*.
3. Rubin, O., et al. (2022). Learning to Retrieve Prompts for In-Context Learning. *NAACL*.
4. Su, H., et al. (2022). Selective Annotation Makes Language Models Better Few-Shot Learners. *ICLR*.
