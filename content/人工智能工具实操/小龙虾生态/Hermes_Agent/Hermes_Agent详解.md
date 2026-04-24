---
title: Hermes Agent详解
date: 2026-04-18
tags:
  - Hermes-Agent
  - Nous-Research
  - 自进化
  - 持久记忆
  - MLOps
  - SQLite-FTS5
  - 技能生成
  - 强化学习
  - 轨迹导出
  - 40+技能
categories:
  - 小龙虾生态
  - Hermes_Agent
alias: Hermes Agent Deep Dive
---

# Hermes Agent 详解

> [!note] 文档信息
> 本文深入解析 Hermes Agent 的技术细节，涵盖 Nous Research 的自进化设计理念、持久记忆系统、40+ 内置技能、MLOps 能力等核心特性。

---

## 关键词速览

| 关键词 | 说明 |
|--------|------|
| Hermes Agent | Nous Research 的自进化 AI Agent |
| Nous Research | 开发公司，专注于自进化 AI |
| 自进化 | Agent 从交互中自动学习和改进 |
| 持久记忆 | SQLite + FTS5 跨会话记忆系统 |
| 技能生成 | AI 自动从解决问题中生成技能 |
| MLOps | 机器学习运维能力 |
| 轨迹导出 | RLHF 训练数据收集 |
| 强化学习 | 自进化核心机制 |
| 上下文注入 | 智能记忆检索 |
| 向量检索 | 语义相似度匹配 |

---

## 一、项目概述

### 1.1 Hermes Agent 起源

Hermes Agent 是由 **Nous Research**（一家专注于自进化 AI 的研究公司）开发的下一代 AI Agent 框架，于 **2026 年 2 月正式发布**。与 OpenClaw 的「广度优先」策略不同，Hermes 采用「深度优先」策略，专注于让 AI Agent **越用越聪明**。

**项目命名由来**：Hermes（赫尔墨斯）是希腊神话中的信使之神，象征着信息的传递与智慧的流通，这也契合了 AI Agent 作为「智能信使」的定位。

### 1.2 核心设计理念

Hermes Agent 的核心理念可以概括为：

> *"Agents that learn, evolve, and improve from every interaction."*

**三大支柱**：

| 支柱 | 说明 |
|------|------|
| **持久记忆** | 基于 SQLite + FTS5 的跨会话记忆系统 |
| **自进化** | 从交互中自动生成可复用技能 |
| **MLOps** | 支持 RLHF 训练数据收集与模型优化 |

### 1.3 与 OpenClaw 的定位差异

| 维度 | Hermes Agent | OpenClaw |
|------|--------------|----------|
| 设计哲学 | 深度优先（越用越聪明） | 广度优先（多渠道覆盖） |
| 记忆系统 | SQLite + FTS5 智能检索 | Markdown 文件静态存储 |
| 自进化 | 支持 AI 自动生成技能 | 依赖手动插件开发 |
| MLOps | 完整 RLHF 流水线 | 无 |
| 多渠道 | 5 大主流平台 | 20+ 平台 |

---

## 二、技术架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      User Interface                           │
│   (CLI / Web UI / Telegram / Discord / Slack)               │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Hermes Core Runtime                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ Skill Engine │  │ Memory     │  │ MLOps Pipeline      │  │
│  │ (技能生成)   │  │ Manager    │  │ (轨迹收集/训练)     │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Layer                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │ SQLite DB   │  │ Vector Store│  │ File System         │  │
│  │ (记忆)      │  │ (语义索引)  │  │ (技能/配置)         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 核心组件详解

#### 2.2.1 Skill Engine（技能引擎）

技能引擎是 Hermes 的核心创新，负责：

1. **问题解决记录**：当 Agent 成功解决一个问题时，记录解决过程
2. **技能抽象**：将具体解决方案泛化为可复用的技能
3. **技能存储**：将生成的技能保存到技能库
4. **技能执行**：在遇到类似问题时调用已有技能

```python
class SkillEngine:
    """技能引擎核心类"""
    
    async def generate_skill(
        self,
        problem: str,
        solution: Solution,
        context: Context
    ) -> Skill:
        """从问题解决过程中生成技能"""
        
        # 1. 分析问题类型
        problem_type = await self.classify_problem(problem)
        
        # 2. 提取可复用模式
        pattern = await self.extract_pattern(solution, context)
        
        # 3. 生成技能定义
        skill = Skill(
            name=self._generate_skill_name(problem_type),
            description=f"处理 {problem_type} 类型问题",
            trigger_conditions=self._extract_triggers(problem),
            implementation=pattern,
            examples=[{"problem": problem, "solution": solution}],
            confidence=0.8,
            usage_count=0,
            success_rate=1.0
        )
        
        # 4. 验证技能有效性
        if await self.validate_skill(skill):
            await self.store_skill(skill)
        
        return skill
    
    async def apply_skill(
        self,
        problem: str,
        context: Context
    ) -> Optional[Skill]:
        """应用相关技能解决问题"""
        
        # 1. 检索相似技能
        candidates = await self.retrieve_similar_skills(problem)
        
        # 2. 评估技能适用性
        for skill in candidates:
            if self._check_trigger_conditions(skill, problem):
                # 3. 增强上下文
                enhanced_context = self._inject_skill_context(
                    context, skill
                )
                
                # 4. 执行带技能的推理
                result = await self._execute_with_skill(
                    problem, enhanced_context, skill
                )
                
                # 5. 更新技能统计
                await self.update_skill_stats(
                    skill, 
                    success=result.success
                )
                
                return skill
        
        return None
```

#### 2.2.2 Memory Manager（记忆管理器）

Hermes 采用 **SQLite + FTS5** 实现高性能记忆系统：

```python
class HermesMemoryManager:
    """Hermes 记忆管理器"""
    
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.conn = sqlite3.connect(db_path)
        self._init_schema()
    
    def _init_schema(self):
        """初始化数据库 schema"""
        self.conn.execute("""
            CREATE VIRTUAL TABLE IF NOT EXISTS memories 
            USING fts5(
                content,
                context,
                timestamp,
                importance,
                category,
                tokenize='porter unicode61'
            )
        """)
        
        self.conn.execute("""
            CREATE TABLE IF NOT EXISTS memory_metadata (
                id INTEGER PRIMARY KEY,
                memory_id TEXT UNIQUE,
                access_count INTEGER DEFAULT 0,
                last_accessed TIMESTAMP,
                source_session TEXT,
                tags TEXT
            )
        """)
    
    async def store(
        self,
        content: str,
        context: dict,
        importance: float = 0.5,
        category: str = "general"
    ) -> str:
        """存储记忆"""
        memory_id = str(uuid.uuid4())
        timestamp = datetime.now().isoformat()
        
        self.conn.execute("""
            INSERT INTO memories 
            (rowid, content, context, timestamp, importance, category)
            VALUES (?, ?, ?, ?, ?, ?)
        """, (memory_id, content, json.dumps(context), 
              timestamp, importance, category))
        
        self.conn.commit()
        
        # 触发记忆压缩检查
        await self._check_compaction()
        
        return memory_id
    
    async def retrieve(
        self,
        query: str,
        limit: int = 10,
        category: str = None
    ) -> List[Memory]:
        """检索记忆（支持 FTS5 全文检索）"""
        
        # FTS5 相似度搜索
        sql = """
            SELECT memory_id, content, context, importance,
                   bm25(memories) as score
            FROM memories
            WHERE memories MATCH ?
        """
        params = [query]
        
        if category:
            sql += " AND category = ?"
            params.append(category)
        
        sql += " ORDER BY score LIMIT ?"
        params.append(limit)
        
        cursor = self.conn.execute(sql, params)
        results = cursor.fetchall()
        
        return [
            Memory(
                id=row[0],
                content=row[1],
                context=json.loads(row[2]),
                importance=row[3],
                score=row[4]
            )
            for row in results
        ]
    
    async def retrieve_semantic(
        self,
        embedding: List[float],
        limit: int = 10
    ) -> List[Memory]:
        """语义检索（需要向量扩展）"""
        
        # 计算余弦相似度
        sql = """
            SELECT memory_id, content, context, importance,
                   cosine_similarity(embedding, ?) as similarity
            FROM memories_with_embeddings
            ORDER BY similarity DESC
            LIMIT ?
        """
        
        cursor = self.conn.execute(sql, [embedding, limit])
        # ... 返回结果
```

#### 2.2.3 MLOps Pipeline（机器学习运维流水线）

```python
class MLOpsPipeline:
    """MLOps 流水线"""
    
    def __init__(self, config: MLOpsConfig):
        self.trajectory_collector = TrajectoryCollector()
        self.reward_computer = RewardComputer()
        self.trainer = DistributedTrainer(config.trainer)
    
    async def collect_trajectory(
        self,
        session: Session,
        interaction: Interaction
    ) -> Trajectory:
        """收集交互轨迹用于 RLHF 训练"""
        
        trajectory = Trajectory(
            session_id=session.id,
            user_message=interaction.user_message,
            assistant_response=interaction.response,
            tool_calls=interaction.tool_calls,
            tool_results=interaction.tool_results,
            reward=await self.compute_reward(interaction),
            metadata={
                "timestamp": interaction.timestamp,
                "duration": interaction.duration,
                "success": interaction.success
            }
        )
        
        # 存储轨迹
        await self.trajectory_collector.store(trajectory)
        
        # 检查是否满足训练条件
        if await self.should_trigger_training():
            await self.trigger_training()
        
        return trajectory
    
    async def compute_reward(self, interaction: Interaction) -> float:
        """计算奖励信号"""
        
        rewards = []
        
        # 1. 任务完成奖励
        if interaction.success:
            rewards.append(1.0)
        else:
            rewards.append(-0.5)
        
        # 2. 效率奖励（工具调用次数越少越好）
        efficiency = 1.0 / (1.0 + len(interaction.tool_calls) * 0.1)
        rewards.append(efficiency)
        
        # 3. 用户反馈奖励
        if interaction.user_feedback:
            if interaction.user_feedback == "helpful":
                rewards.append(0.5)
            elif interaction.user_feedback == "not_helpful":
                rewards.append(-0.3)
        
        # 4. 一致性奖励（与历史行为一致）
        consistency = await self.compute_consistency_reward(
            interaction
        )
        rewards.append(consistency)
        
        return sum(rewards) / len(rewards)
    
    async def trigger_training(self):
        """触发模型训练"""
        
        # 1. 收集足够轨迹
        trajectories = await self.trajectory_collector.get_batch(
            min_samples=1000
        )
        
        # 2. 数据预处理
        dataset = self._preprocess_trajectories(trajectories)
        
        # 3. 分布式训练
        await self.trainer.train(
            dataset=dataset,
            epochs=config.epochs,
            learning_rate=config.learning_rate
        )
        
        # 4. 评估新模型
        evaluation = await self._evaluate_model()
        
        # 5. A/B 测试或全量部署
        if evaluation.improvement > threshold:
            await self._deploy_model(evaluation.checkpoint)
```

---

## 三、40+ 内置技能详解

### 3.1 技能分类总览

| 类别 | 技能数量 | 代表技能 |
|------|----------|----------|
| **信息检索** | 8 | web_search, wiki_lookup, code_search |
| **文件操作** | 6 | file_read, file_write, file_search |
| **代码执行** | 5 | python_exec, bash, git_operations |
| **通信** | 4 | send_email, send_message, calendar |
| **数据处理** | 7 | data_analysis, csv_process, json_transform |
| **媒体** | 5 | image_process, audio_transcribe, video_extract |
| **系统** | 4 | system_info, process_manager, network_check |
| **ML/AI** | 6 | model_inference, prompt_tuning, dataset_create |

### 3.2 核心技能详解

#### 3.2.1 web_search

```python
@skill(name="web_search", description="网络搜索")
class WebSearchSkill:
    """网络搜索技能"""
    
    def __init__(self):
        self.search_engines = ["google", "bing", "duckduckgo"]
        self.default_engine = "duckduckgo"  # 保护隐私
    
    async def execute(
        self,
        query: str,
        engine: str = None,
        num_results: int = 5
    ) -> SearchResult:
        """执行网络搜索"""
        
        engine = engine or self.default_engine
        
        if engine == "duckduckgo":
            results = await self._search_duckduckgo(query, num_results)
        elif engine == "google":
            results = await self._search_google(query, num_results)
        
        return SearchResult(
            query=query,
            engine=engine,
            results=[
                Result(
                    title=r.title,
                    url=r.url,
                    snippet=r.snippet,
                    relevance=self._compute_relevance(query, r)
                )
                for r in results
            ]
        )
    
    async def _search_duckduckgo(
        self, 
        query: str, 
        num_results: int
    ) -> List[RawResult]:
        """DuckDuckGo 搜索"""
        from duckduckgo_search import DDGS
        
        with DDGS() as ddgs:
            results = list(ddgs.text(query, max_results=num_results))
        return results
```

#### 3.2.2 code_analysis

```python
@skill(name="code_analysis", description="代码分析")
class CodeAnalysisSkill:
    """代码分析技能"""
    
    async def execute(
        self,
        code: str,
        language: str,
        analysis_type: str = "full"
    ) -> CodeAnalysis:
        """执行代码分析"""
        
        # 1. 语法分析
        syntax_tree = self._parse_code(code, language)
        
        # 2. 复杂度分析
        complexity = self._analyze_complexity(syntax_tree)
        
        # 3. 潜在问题检测
        issues = await self._detect_issues(syntax_tree, code)
        
        # 4. 优化建议
        suggestions = self._generate_suggestions(
            complexity, issues
        )
        
        return CodeAnalysis(
            language=language,
            complexity=complexity,
            issues=issues,
            suggestions=suggestions,
            metrics={
                "lines": len(code.splitlines()),
                "functions": self._count_functions(syntax_tree),
                "classes": self._count_classes(syntax_tree),
                "cyclomatic": complexity.cyclomatic,
                "cognitive": complexity.cognitive
            }
        )
```

### 3.3 技能使用示例

```python
# 在 Hermes 中使用技能
async def main():
    hermes = HermesAgent()
    
    # 直接调用技能
    result = await hermes.execute_skill(
        "web_search",
        query="OpenClaw vs Hermes Agent comparison",
        num_results=5
    )
    
    # 技能链
    chain = await hermes.execute_chain([
        {"skill": "web_search", "params": {"query": "..."}},
        {"skill": "code_analysis", "params": {"code": "${previous_result.code}"}}
    ])
    
    # 条件技能调用
    result = await hermes.execute_conditional(
        condition="code" in problem,
        if_true={"skill": "code_analysis", "params": {...}},
        if_false={"skill": "web_search", "params": {...}}
    )
```

---

## 四、自进化机制详解

### 4.1 自进化工作流

```
用户问题 → Agent 分析 → 解决方案 → 技能生成 → 技能验证 → 技能入库
                ↑                                          ↓
                └──────── 相似问题检索 ←←←←←←←←←←←←←←←←←←←←←┘
```

### 4.2 技能生成算法

```python
class SkillGenerator:
    """技能生成器"""
    
    async def generate_from_solution(
        self,
        problem: Problem,
        solution: Solution,
        session: Session
    ) -> Optional[Skill]:
        """从解决方案生成技能"""
        
        # 1. 问题抽象
        problem_pattern = await self._abstract_problem(problem)
        
        # 2. 解决方案泛化
        generalized_steps = await self._generalize_solution(
            solution.steps
        )
        
        # 3. 参数提取
        params = self._extract_parameters(
            problem_pattern, generalized_steps
        )
        
        # 4. 前置条件定义
        preconditions = self._define_preconditions(
            problem_pattern
        )
        
        # 5. 技能签名生成
        skill_signature = self._generate_signature(
            problem_pattern.type,
            params,
            preconditions
        )
        
        # 6. 生成技能代码
        skill_code = await self._generate_skill_code(
            signature=skill_signature,
            steps=generalized_steps,
            examples=[{"input": problem.text, "output": solution.result}]
        )
        
        # 7. 验证技能
        if await self._validate_skill(skill_code):
            return Skill(
                name=skill_signature.name,
                description=skill_signature.description,
                params=params,
                preconditions=preconditions,
                implementation=skill_code,
                confidence=0.8,
                generated_at=datetime.now()
            )
        
        return None
    
    async def _abstract_problem(self, problem: Problem) -> ProblemPattern:
        """问题抽象"""
        
        # 使用 LLM 提取问题模式
        prompt = f"""
        分析以下问题，提取其抽象模式：
        
        问题：{problem.text}
        类型：{problem.category}
        
        请提取：
        1. 问题类型（如：文件处理、网络请求、数据转换等）
        2. 关键实体（如：文件路径、URL、数据格式等）
        3. 约束条件（如：时间限制、格式要求等）
        
        以 JSON 格式返回。
        """
        
        response = await self.llm.complete(prompt)
        return ProblemPattern(**json.loads(response.content))
```

### 4.3 技能进化机制

```python
class SkillEvolution:
    """技能进化器"""
    
    async def evolve_skill(
        self,
        skill: Skill,
        new_interactions: List[Interaction]
    ) -> Skill:
        """基于新交互进化技能"""
        
        # 1. 收集成功案例
        success_cases = [
            i for i in new_interactions 
            if i.success and self._matches_skill(i, skill)
        ]
        
        if not success_cases:
            return skill
        
        # 2. 分析失败模式
        failure_cases = [
            i for i in new_interactions 
            if not i.success and self._matches_skill(i, skill)
        ]
        
        # 3. 提取改进点
        improvements = await self._extract_improvements(
            success_cases, failure_cases, skill
        )
        
        # 4. 更新技能
        updated_skill = skill.copy()
        
        if improvements.add_examples:
            updated_skill.examples.extend(
                [case.to_example() for case in success_cases]
            )
        
        if improvements.param_extensions:
            updated_skill.params.update(
                improvements.param_extensions
            )
        
        if improvements.new_capabilities:
            updated_skill.capabilities.extend(
                improvements.new_capabilities
            )
        
        # 5. 重新验证
        updated_skill.confidence = await self._recompute_confidence(
            updated_skill
        )
        
        return updated_skill
    
    async def _recompute_confidence(self, skill: Skill) -> float:
        """重新计算置信度"""
        
        total = skill.usage_count + len(skill.examples)
        if total == 0:
            return 0.5
        
        successes = sum(
            1 for ex in skill.examples if ex.success
        ) + skill.successful_uses
        
        return min(0.99, successes / total * 0.8 + 0.2)
```

---

## 五、MLOps 能力

### 5.1 轨迹收集系统

```python
class TrajectoryCollector:
    """轨迹收集器"""
    
    def __init__(self, storage_path: str):
        self.storage_path = Path(storage_path)
        self.storage_path.mkdir(parents=True, exist_ok=True)
        self.buffer: List[Trajectory] = []
        self.buffer_size = 1000
    
    async def collect(
        self,
        session_id: str,
        interaction: Interaction,
        reward: float
    ) -> Trajectory:
        """收集交互轨迹"""
        
        trajectory = Trajectory(
            id=str(uuid.uuid4()),
            session_id=session_id,
            timestamp=datetime.now(),
            interaction=interaction,
            reward=reward,
            metadata={
                "user_id": interaction.user_id,
                "workspace": interaction.workspace,
                "model": interaction.model,
                "latency_ms": interaction.latency_ms
            }
        )
        
        self.buffer.append(trajectory)
        
        # 定期写入磁盘
        if len(self.buffer) >= self.buffer_size:
            await self._flush_buffer()
        
        return trajectory
    
    async def export_for_rlhf(
        self,
        output_path: str,
        format: str = "huggingface"
    ) -> str:
        """导出 RLHF 训练格式"""
        
        if format == "huggingface":
            return await self._export_hf_format(output_path)
        elif format == "openai":
            return await self._export_openai_format(output_path)
        elif format == "custom":
            return await self._export_custom_format(output_path)
    
    async def _export_hf_format(self, output_path: str) -> str:
        """导出为 HuggingFace 格式"""
        
        dataset = Dataset.from_list([
            {
                "messages": [
                    {"role": "user", "content": t.interaction.user_message},
                    {"role": "assistant", "content": t.interaction.response}
                ],
                "reward": t.reward
            }
            for t in self.buffer
        ])
        
        dataset.save_to_disk(output_path)
        return output_path
```

### 5.2 批量处理能力

```python
class BatchProcessor:
    """批量处理器"""
    
    async def process_batch(
        self,
        tasks: List[Task],
        max_concurrency: int = 5
    ) -> List[TaskResult]:
        """批量处理任务"""
        
        semaphore = asyncio.Semaphore(max_concurrency)
        
        async def process_with_limit(task: Task) -> TaskResult:
            async with semaphore:
                return await self._process_single(task)
        
        results = await asyncio.gather(
            *[process_with_limit(t) for t in tasks],
            return_exceptions=True
        )
        
        return [r for r in results if not isinstance(r, Exception)]
    
    async def schedule_batch(
        self,
        tasks: List[Task],
        schedule: Schedule
    ) -> str:
        """批量任务调度"""
        
        batch_id = str(uuid.uuid4())
        
        # 创建批处理任务
        batch = BatchTask(
            id=batch_id,
            tasks=tasks,
            schedule=schedule,
            status="pending"
        )
        
        await self.batch_store.save(batch)
        
        # 启动调度器
        asyncio.create_task(self._run_scheduler(batch_id))
        
        return batch_id
```

---

## 六、部署与配置

### 6.1 Docker 部署

```bash
# 快速启动
docker run -d \
  --name hermes-agent \
  -p 8080:8080 \
  -v ~/hermes/data:/app/data \
  -v ~/hermes/skills:/app/skills \
  -e OPENAI_API_KEY="${OPENAI_API_KEY}" \
  -e ANTHROPIC_API_KEY="${ANTHROPIC_API_KEY}" \
  nousresearch/hermes-agent:latest
```

### 6.2 配置文件

```yaml
# hermes.yaml
hermes:
  version: "1.0"
  
model:
  provider: "anthropic"
  model: "claude-sonnet-4-20250514"
  api_key: "${ANTHROPIC_API_KEY}"
  
memory:
  type: "sqlite_fts5"
  db_path: "./data/hermes.db"
  max_memory_items: 10000
  
skills:
  auto_generate: true
  validation: true
  confidence_threshold: 0.7
  
mlops:
  trajectory_collection: true
  batch_processing: true
  rlhf_export: true
  
channels:
  telegram:
    enabled: true
    bot_token: "${TELEGRAM_BOT_TOKEN}"
    
  discord:
    enabled: true
    bot_token: "${DISCORD_BOT_TOKEN}"
```

---

## 七、相关文档

- [[OpenClaw完整指南]] - OpenClaw 完整指南
- [[OpenClaw架构解析]] - OpenClaw 架构
- [[Hermes_vs_OpenClaw对比]] - 两者详细对比
- [[Pi_Framework深度解析]] - Pi Framework
- [[OpenClaw记忆系统]] - 记忆系统对比

---

*文档更新于 2026年4月18日*
