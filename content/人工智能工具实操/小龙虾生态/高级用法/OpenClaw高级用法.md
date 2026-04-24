---
title: OpenClaw高级用法
date: 2026-04-18
tags:
  - openclaw
  - 高级用法
  - 自定义工具
  - 工作流引擎
  - CoT推理
  - 多Agent系统
  - 定时任务
  - 自动化
  - 编排
  - 调度
categories:
  - 人工智能工具实操/小龙虾生态/高级用法
alias: OpenClaw Advanced Usage
---

## 关键词

| 关键词 | 说明 |
|--------|------|
| Custom Tool | 自定义工具开发 |
| Workflow Engine | 工作流编排引擎 |
| Chain of Thought | 思维链推理 |
| Multi-Agent | 多代理协作系统 |
| Cron Scheduler | 定时任务调度 |
| Task Queue | 异步任务队列 |
| Pipeline | 数据处理管道 |
| Orchestration | 流程编排 |
| Event-Driven | 事件驱动架构 |
| State Machine | 状态机模式 |

---

## 概述

OpenClaw 的高级用法涵盖了一系列面向生产环境和企业级应用的能力，包括自定义工具开发、工作流引擎、Chain of Thought 推理、多 Agent 协作系统以及定时任务调度等。这些功能使 OpenClaw 能够处理复杂的自动化场景和长周期任务。

> [!note] 适用场景
> 当你需要构建复杂的 AI 应用、自动化工作流程、多步骤数据处理或需要多个 AI Agent 协作时，OpenClaw 的高级功能能够提供完整的解决方案。

---

## 自定义工具开发

### 工具注册系统

```python
# openclaw/tools/registry.py
from typing import Dict, List, Type, Callable, Optional, Any
from dataclasses import dataclass, field
from pydantic import BaseModel
import asyncio

class ToolRegistry:
    """工具注册中心"""
    
    _tools: Dict[str, Type["Tool"]] = {}
    _instances: Dict[str, "Tool"] = {}
    _middlewares: List[Callable] = []
    
    @classmethod
    def register(cls, name: str, tool_class: Type["Tool"]):
        """注册工具类"""
        cls._tools[name] = tool_class
        return tool_class
    
    @classmethod
    def get_tool(cls, name: str) -> Optional["Tool"]:
        """获取工具实例（单例）"""
        if name not in cls._instances:
            tool_class = cls._tools.get(name)
            if tool_class:
                cls._instances[name] = tool_class()
        return cls._instances.get(name)
    
    @classmethod
    def list_tools(cls) -> List[str]:
        """列出所有工具"""
        return list(cls._tools.keys())
    
    @classmethod
    def add_middleware(cls, middleware: Callable):
        """添加工具调用中间件"""
        cls._middlewares.append(middleware)

class ToolConfig(BaseModel):
    """工具配置"""
    name: str
    description: str
    parameters: Dict[str, Any]
    returns: Dict[str, Any]
    examples: List[Dict[str, Any]] = []
    version: str = "1.0.0"
    deprecated: bool = False
    aliases: List[str] = []

@dataclass
class ToolResult:
    """工具执行结果"""
    success: bool
    data: Any = None
    error: str = None
    error_code: str = None
    execution_time_ms: float = 0
    metadata: Dict[str, Any] = field(default_factory=dict)

class Tool(ABC):
    """工具基类"""
    
    name: str = ""
    description: str = ""
    input_model: Type[BaseModel] = None
    
    def __init__(self):
        self._cache = {}
    
    @abstractmethod
    async def execute(self, input_data: BaseModel) -> ToolResult:
        """执行工具逻辑"""
        pass
    
    async def validate_input(self, raw_input: Dict) -> BaseModel:
        """验证并转换输入"""
        if self.input_model:
            return self.input_model(**raw_input)
        return raw_input
    
    def get_schema(self) -> ToolConfig:
        """获取工具配置模式"""
        schema = {
            "name": self.name,
            "description": self.description,
            "parameters": {},
            "returns": {}
        }
        
        if self.input_model:
            schema["parameters"] = self.input_model.model_json_schema()
        
        return ToolConfig(**schema)
```

### 自定义工具开发模板

```python
# 自定义工具示例：Web搜索工具
from pydantic import BaseModel, Field
from typing import List, Optional, Dict, Any
import httpx

class WebSearchInput(BaseModel):
    """Web搜索工具输入"""
    query: str = Field(description="搜索查询内容")
    num_results: int = Field(default=10, description="返回结果数量", ge=1, le=50)
    language: str = Field(default="zh", description="搜索语言")
    time_range: Optional[str] = Field(default=None, description="时间范围：day/week/month/year")
    safe_search: bool = Field(default=True, description="安全搜索")

class WebSearchResult(BaseModel):
    """单个搜索结果"""
    title: str
    url: str
    snippet: str
    published_date: Optional[str] = None
    source: Optional[str] = None

class WebSearchOutput(BaseModel):
    """Web搜索工具输出"""
    query: str
    total_results: int
    results: List[WebSearchResult]

@ToolRegistry.register("web_search")
class WebSearchTool(Tool):
    """Web搜索工具"""
    
    name = "web_search"
    description = """
    在互联网上搜索相关信息。支持多语言搜索、时间范围过滤和安全搜索模式。
    
    适用场景：
    - 查找最新新闻和资讯
    - 查询公司、产品信息
    - 获取技术文档和教程
    - 研究特定主题的背景资料
    """
    input_model = WebSearchInput
    
    def __init__(self):
        super().__init__()
        self.client = None
        self.api_keys = {
            "google": None,
            "duckduckgo": None,
            "bing": None
        }
    
    async def _get_client(self) -> httpx.AsyncClient:
        """获取 HTTP 客户端"""
        if not self.client:
            self.client = httpx.AsyncClient(
                timeout=30.0,
                follow_redirects=True
            )
        return self.client
    
    async def execute(self, input_data: WebSearchInput) -> ToolResult:
        """执行搜索"""
        import time
        start_time = time.time()
        
        try:
            # 优先使用 DuckDuckGo（无需API Key）
            results = await self._search_duckduckgo(input_data)
            
            return ToolResult(
                success=True,
                data=results.model_dump(),
                execution_time_ms=(time.time() - start_time) * 1000,
                metadata={
                    "engine": "duckduckgo",
                    "query": input_data.query
                }
            )
            
        except Exception as e:
            return ToolResult(
                success=False,
                error=str(e),
                error_code="SEARCH_ERROR"
            )
    
    async def _search_duckduckgo(self, input_data: WebSearchInput) -> WebSearchOutput:
        """DuckDuckGo 搜索"""
        from duckduckgo_search import DDGS
        
        params = {
            "keywords": input_data.query,
            "region": f"wt-wt",
            "safesearch": "moderate" if input_data.safe_search else "off",
            "timelimit": input_data.time_range,
            "max_results": input_data.num_results
        }
        
        results = []
        with DDGS() as ddgs:
            for r in ddgs.text(**params):
                results.append(WebSearchResult(
                    title=r.get("title", ""),
                    url=r.get("href", ""),
                    snippet=r.get("body", ""),
                    published_date=r.get("date"),
                    source=r.get("source")
                ))
        
        return WebSearchOutput(
            query=input_data.query,
            total_results=len(results),
            results=results
        )
```

### 工具中间件

```python
# 工具调用中间件示例
async def cache_middleware(tool_name: str, input_data, next_handler):
    """缓存中间件"""
    cache_key = f"{tool_name}:{hash(str(input_data))}"
    
    if cache_key in ToolRegistry._cache:
        return ToolRegistry._cache[cache_key]
    
    result = await next_handler(input_data)
    
    if result.success:
        ToolRegistry._cache[cache_key] = result
    
    return result

async def retry_middleware(tool_name: str, input_data, next_handler):
    """重试中间件"""
    max_retries = 3
    for attempt in range(max_retries):
        try:
            return await next_handler(input_data)
        except Exception as e:
            if attempt == max_retries - 1:
                raise
            await asyncio.sleep(2 ** attempt)  # 指数退避

async def logging_middleware(tool_name: str, input_data, next_handler):
    """日志中间件"""
    logger.info(f"Calling tool: {tool_name} with input: {input_data}")
    result = await next_handler(input_data)
    logger.info(f"Tool {tool_name} returned: {result.success}")
    return result

# 注册中间件
ToolRegistry.add_middleware(logging_middleware)
ToolRegistry.add_middleware(retry_middleware)
ToolRegistry.add_middleware(cache_middleware)
```

---

## 工作流引擎

### 工作流定义 DSL

```yaml
# workflow_example.yaml
name: "数据采集分析工作流"
version: "1.0.0"
description: "从多个数据源采集数据，进行处理和分析"

variables:
  target_topic: "AI发展趋势"
  date_range: "last_week"
  output_format: "json"

triggers:
  - type: "manual"
    description: "手动触发"
  - type: "schedule"
    cron: "0 9 * * *"  # 每天早上9点执行

steps:
  - id: "step_1_fetch"
    name: "数据采集"
    tool: "web_search"
    input:
      query: "${variables.target_topic}"
      num_results: 20
      time_range: "${variables.date_range}"
    output_var: "search_results"
    
  - id: "step_2_filter"
    name: "数据过滤"
    type: "transform"
    input: "${step_1_fetch.output_var}"
    transform:
      type: "filter"
      conditions:
        - field: "snippet"
          operator: "contains"
          value: "AI|人工智能"
    output_var: "filtered_results"
    
  - id: "step_3_dedupe"
    name: "去重处理"
    type: "transform"
    input: "${step_2_filter.output_var}"
    transform:
      type: "dedupe"
      keys: ["url"]
    output_var: "deduped_results"
    
  - id: "step_4_analyze"
    name: "内容分析"
    type: "agent"
    input: "${step_3_dedupe.output_var}"
    agent:
      model: "gpt-4"
      prompt: |
        分析以下文章，提取关键信息：
        - 主要观点
        - 涉及的公司/产品
        - 发展趋势
        
        文章列表：
        ${input}
    output_var: "analysis_results"
    
  - id: "step_5_save"
    name: "保存结果"
    type: "action"
    action:
      type: "save_file"
      path: "output/${date}.json"
      format: "${variables.output_format}"
      content: "${step_4_analyze.output_var}"

error_handling:
  on_step_failure:
    strategy: "retry"
    max_retries: 2
    backoff: "exponential"
  on_workflow_failure:
    strategy: "notify"
    channels: ["email", "slack"]

retry:
  max_attempts: 3
  backoff_seconds: [1, 5, 30]
```

### 工作流引擎实现

```python
# openclaw/workflows/engine.py
from typing import Dict, List, Any, Optional, Callable
from dataclasses import dataclass, field
from enum import Enum
import asyncio
import logging

class StepStatus(Enum):
    PENDING = "pending"
    RUNNING = "running"
    SUCCESS = "success"
    FAILED = "failed"
    SKIPPED = "skipped"
    RETRYING = "retrying"

class StepType(Enum):
    TOOL = "tool"
    TRANSFORM = "transform"
    AGENT = "agent"
    CONDITION = "condition"
    PARALLEL = "parallel"
    LOOP = "loop"
    ACTION = "action"

@dataclass
class WorkflowStep:
    """工作流步骤"""
    id: str
    name: str
    step_type: StepType
    depends_on: List[str] = field(default_factory=list)
    input_mapping: Dict[str, str] = field(default_factory=dict)
    config: Dict[str, Any] = field(default_factory=dict)
    
    status: StepStatus = StepStatus.PENDING
    output: Any = None
    error: str = None
    start_time: float = None
    end_time: float = None
    
    retry_count: int = 0

@dataclass
class WorkflowContext:
    """工作流执行上下文"""
    workflow_id: str
    variables: Dict[str, Any]
    step_outputs: Dict[str, Any] = {}
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def get_input(self, mapping: Dict[str, str]) -> Dict[str, Any]:
        """解析输入映射"""
        result = {}
        for key, value in mapping.items():
            result[key] = self._resolve_value(value)
        return result
    
    def _resolve_value(self, value: str) -> Any:
        """解析变量引用"""
        if not isinstance(value, str):
            return value
        
        # ${step_id.output_var} 引用
        if value.startswith("${") and value.endswith("}"):
            ref = value[2:-1]
            if "." in ref:
                step_id, var_name = ref.split(".", 1)
                return self.step_outputs.get(step_id, {}).get(var_name)
            return self.variables.get(ref)
        
        # ${variables.xxx} 引用
        if value.startswith("${variables."):
            var_name = value[12:-1]
            return self.variables.get(var_name)
        
        return value

class WorkflowEngine:
    """工作流引擎"""
    
    def __init__(self, config: Dict[str, Any] = None):
        self.config = config or {}
        self.tools = ToolRegistry
        self.logger = logging.getLogger("workflow.engine")
        self._running_workflows: Dict[str, "Workflow"] = {}
    
    async def execute_workflow(self, workflow: "Workflow") -> Dict[str, Any]:
        """执行工作流"""
        
        context = WorkflowContext(
            workflow_id=workflow.id,
            variables=workflow.variables.copy()
        )
        
        self._running_workflows[workflow.id] = workflow
        
        try:
            # 拓扑排序确定执行顺序
            execution_order = self._topological_sort(workflow.steps)
            
            for step in execution_order:
                # 检查依赖
                if not self._check_dependencies(step, workflow.steps):
                    step.status = StepStatus.SKIPPED
                    continue
                
                # 执行步骤
                result = await self._execute_step(step, context, workflow)
                
                if not result:
                    # 处理失败
                    if not await self._handle_step_failure(step, context, workflow):
                        return {"success": False, "failed_step": step.id}
                
                workflow.steps[step.id] = step
            
            return {
                "success": True,
                "outputs": context.step_outputs
            }
            
        finally:
            del self._running_workflows[workflow.id]
    
    async def _execute_step(
        self,
        step: WorkflowStep,
        context: WorkflowContext,
        workflow: "Workflow"
    ) -> bool:
        """执行单个步骤"""
        
        step.status = StepStatus.RUNNING
        import time
        step.start_time = time.time()
        
        self.logger.info(f"Executing step: {step.id}")
        
        try:
            if step.step_type == StepType.TOOL:
                result = await self._execute_tool_step(step, context)
            elif step.step_type == StepType.TRANSFORM:
                result = await self._execute_transform_step(step, context)
            elif step.step_type == StepType.AGENT:
                result = await self._execute_agent_step(step, context)
            elif step.step_type == StepType.CONDITION:
                result = await self._execute_condition_step(step, context)
            elif step.step_type == StepType.PARALLEL:
                result = await self._execute_parallel_step(step, context, workflow)
            else:
                result = await self._execute_action_step(step, context)
            
            step.output = result
            step.status = StepStatus.SUCCESS
            step.end_time = time.time()
            
            # 保存到上下文
            context.step_outputs[step.id] = result
            
            return True
            
        except Exception as e:
            step.status = StepStatus.FAILED
            step.error = str(e)
            step.end_time = time.time()
            self.logger.error(f"Step {step.id} failed: {e}")
            return False
    
    async def _execute_tool_step(
        self,
        step: WorkflowStep,
        context: WorkflowContext
    ) -> Any:
        """执行工具步骤"""
        
        tool_name = step.config.get("tool")
        tool = self.tools.get_tool(tool_name)
        
        if not tool:
            raise ValueError(f"Tool not found: {tool_name}")
        
        # 解析输入
        input_mapping = step.input_mapping.get("__all__", step.input_mapping)
        raw_input = context.get_input(input_mapping)
        
        # 验证输入
        validated_input = await tool.validate_input(raw_input)
        
        # 执行工具
        result = await tool.execute(validated_input)
        
        if not result.success:
            raise RuntimeError(f"Tool error: {result.error}")
        
        return result.data
    
    async def _execute_transform_step(
        self,
        step: WorkflowStep,
        context: WorkflowContext
    ) -> Any:
        """执行转换步骤"""
        
        transform_config = step.config.get("transform", {})
        transform_type = transform_config.get("type")
        input_data = context.get_input(step.input_mapping)
        
        if transform_type == "filter":
            return self._apply_filter(input_data, transform_config)
        elif transform_type == "map":
            return self._apply_map(input_data, transform_config)
        elif transform_type == "reduce":
            return self._apply_reduce(input_data, transform_config)
        elif transform_type == "dedupe":
            return self._apply_dedupe(input_data, transform_config)
        else:
            raise ValueError(f"Unknown transform type: {transform_type}")
    
    def _apply_filter(self, data: List, config: Dict) -> List:
        """应用过滤条件"""
        conditions = config.get("conditions", [])
        
        filtered = []
        for item in data:
            if self._check_conditions(item, conditions):
                filtered.append(item)
        
        return filtered
    
    def _check_conditions(self, item: Any, conditions: List[Dict]) -> bool:
        """检查条件是否满足"""
        
        for cond in conditions:
            field = cond["field"]
            operator = cond["operator"]
            value = cond["value"]
            
            item_value = self._get_nested_value(item, field)
            
            if operator == "equals":
                if item_value != value:
                    return False
            elif operator == "contains":
                if value not in str(item_value):
                    return False
            elif operator == "gt":
                if not (item_value > value):
                    return False
            elif operator == "lt":
                if not (item_value < value):
                    return False
            
        return True
    
    def _get_nested_value(self, obj: Any, path: str) -> Any:
        """获取嵌套属性值"""
        for key in path.split("."):
            if isinstance(obj, dict):
                obj = obj.get(key)
            else:
                obj = getattr(obj, key, None)
            if obj is None:
                return None
        return obj
    
    async def _execute_agent_step(
        self,
        step: WorkflowStep,
        context: WorkflowContext
    ) -> str:
        """执行 Agent 步骤"""
        
        agent_config = step.config.get("agent", {})
        input_data = context.get_input(step.input_mapping)
        
        prompt_template = agent_config.get("prompt", "")
        
        # 渲染提示模板
        if isinstance(input_data, dict):
            prompt = prompt_template.replace("${input}", str(input_data))
        else:
            prompt = prompt_template.replace("${input}", str(input_data))
        
        # 调用 Agent
        from openclaw.agent import Agent
        agent = Agent(model=agent_config.get("model", "gpt-4"))
        
        response = await agent.process(prompt, context=context.variables)
        
        return response
    
    def _topological_sort(self, steps: Dict[str, WorkflowStep]) -> List[WorkflowStep]:
        """拓扑排序"""
        
        visited = set()
        result = []
        
        def visit(step_id: str):
            if step_id in visited:
                return
            visited.add(step_id)
            
            step = steps.get(step_id)
            if step:
                for dep in step.depends_on:
                    visit(dep)
                result.append(step)
        
        for step_id in steps:
            visit(step_id)
        
        return result
    
    def _check_dependencies(
        self,
        step: WorkflowStep,
        all_steps: Dict[str, WorkflowStep]
    ) -> bool:
        """检查依赖是否满足"""
        for dep_id in step.depends_on:
            dep_step = all_steps.get(dep_id)
            if dep_step and dep_step.status != StepStatus.SUCCESS:
                return False
        return True
    
    async def _handle_step_failure(
        self,
        step: WorkflowStep,
        context: WorkflowContext,
        workflow: "Workflow"
    ) -> bool:
        """处理步骤失败"""
        
        max_retries = workflow.retry_config.get("max_attempts", 3)
        
        if step.retry_count < max_retries:
            step.status = StepStatus.RETRYING
            step.retry_count += 1
            
            backoff = workflow.retry_config.get("backoff_seconds", [1, 5, 30])
            wait_time = backoff[min(step.retry_count - 1, len(backoff) - 1)]
            
            await asyncio.sleep(wait_time)
            
            return await self._execute_step(step, context, workflow)
        
        return False
```

---

## Chain of Thought 集成

### CoT 推理引擎

```python
# openclaw/reasoning/chain_of_thought.py
from typing import List, Dict, Any, Optional, Callable
from dataclasses import dataclass
from enum import Enum
import json

class ReasoningStrategy(Enum):
    ZERO_SHOT = "zero_shot"
    FEW_SHOT = "few_shot"
    CHAIN_OF_THOUGHT = "chain_of_thought"
    TREE_OF_THOUGHT = "tree_of_thought"
    SELF_ASK = "self_ask"
    REACT = "react"

@dataclass
class Thought:
    """思考步骤"""
    step_number: int
    thought: str
    action: Optional[str] = None
    action_input: Optional[Dict] = None
    observation: Optional[str] = None
    result: Optional[Any] = None
    confidence: float = 1.0

@dataclass
class ReasoningResult:
    """推理结果"""
    strategy: ReasoningStrategy
    thoughts: List[Thought]
    final_answer: str
    reasoning_trace: str
    confidence: float
    metadata: Dict[str, Any]

class ChainOfThoughtEngine:
    """Chain of Thought 推理引擎"""
    
    def __init__(self, agent: "Agent"):
        self.agent = agent
        self.strategies = {
            ReasoningStrategy.CHAIN_OF_THOUGHT: self._cot_reasoning,
            ReasoningStrategy.TREE_OF_THOUGHT: self._tot_reasoning,
            ReasoningStrategy.SELF_ASK: self._self_ask_reasoning,
            ReasoningStrategy.REACT: self._react_reasoning
        }
    
    async def reason(
        self,
        query: str,
        strategy: ReasoningStrategy = ReasoningStrategy.CHAIN_OF_THOUGHT,
        max_steps: int = 10,
        **kwargs
    ) -> ReasoningResult:
        """执行推理"""
        
        reasoning_func = self.strategies.get(strategy)
        
        if reasoning_func:
            return await reasoning_func(query, max_steps, **kwargs)
        
        # 默认直接返回
        response = await self.agent.process(query)
        return ReasoningResult(
            strategy=strategy,
            thoughts=[],
            final_answer=response,
            reasoning_trace="No reasoning trace available",
            confidence=1.0
        )
    
    async def _cot_reasoning(
        self,
        query: str,
        max_steps: int = 10,
        **kwargs
    ) -> ReasoningResult:
        """Chain of Thought 推理"""
        
        cot_prompt = f"""请逐步思考并回答以下问题。

问题: {query}

请按照以下格式思考：
1. [第一步思考] - 描述你的思考过程
2. [第二步思考] - 基于上一步继续推理
...

最后给出答案。
"""
        
        thoughts = []
        response = await self.agent.process(cot_prompt)
        
        # 解析思考步骤
        lines = response.split("\n")
        step_num = 0
        for line in lines:
            if line.strip() and any(line.strip().startswith(str(i) + ".") for i in range(1, 20)):
                step_num += 1
                thoughts.append(Thought(
                    step_number=step_num,
                    thought=line.strip()
                ))
        
        # 提取最终答案
        final_answer = self._extract_final_answer(response)
        
        return ReasoningResult(
            strategy=ReasoningStrategy.CHAIN_OF_THOUGHT,
            thoughts=thoughts,
            final_answer=final_answer,
            reasoning_trace=response,
            confidence=0.9
        )
    
    async def _react_reasoning(
        self,
        query: str,
        max_steps: int = 10,
        tools: List = None,
        **kwargs
    ) -> ReasoningResult:
        """ReAct (Reason + Act) 推理"""
        
        thoughts = []
        current_query = query
        step_num = 0
        
        while step_num < max_steps:
            step_num += 1
            
            # 思考
            thought_prompt = f"""基于以下问题，给出你的思考和行动。

问题: {current_query}

历史步骤:
{self._format_thoughts(thoughts)}

请按以下格式回答：
Thought: [你的思考]
Action: [要执行的动作，如 tool_name]
Action Input: [动作的参数]
"""
            
            response = await self.agent.process(thought_prompt)
            
            # 解析响应
            thought, action, action_input = self._parse_react_response(response)
            
            if not action:
                # 没有更多行动，完成推理
                break
            
            # 执行行动
            observation = await self._execute_action(action, action_input, tools)
            
            thoughts.append(Thought(
                step_number=step_num,
                thought=thought,
                action=action,
                action_input=action_input,
                observation=observation
            ))
            
            # 更新查询
            current_query = f"Based on observation: {observation}\n\nOriginal question: {query}"
        
        final_answer = await self.agent.process(
            f"基于以下推理过程，给出最终答案。\n\n"
            f"{self._format_thoughts(thoughts)}\n\n"
            f"问题: {query}"
        )
        
        return ReasoningResult(
            strategy=ReasoningStrategy.REACT,
            thoughts=thoughts,
            final_answer=final_answer,
            reasoning_trace=self._format_thoughts(thoughts),
            confidence=0.95
        )
    
    async def _tot_reasoning(
        self,
        query: str,
        max_steps: int = 5,
        branches: int = 3,
        **kwargs
    ) -> ReasoningResult:
        """Tree of Thought 推理"""
        
        all_paths = []
        
        async def explore_branch(
            branch_query: str,
            depth: int,
            path: List[Thought]
        ):
            if depth >= max_steps:
                all_paths.append(path)
                return
            
            # 生成多个候选思考
            candidates_prompt = f"""对于问题: {branch_query}

请提出 {branches} 个不同的思考方向或解决方案。

格式：
1. [第一个方向]
2. [第二个方向]
3. [第三个方向]
"""
            
            response = await self.agent.process(candidates_prompt)
            candidates = self._parse_candidates(response, branches)
            
            for i, candidate in enumerate(candidates):
                new_path = path + [Thought(
                    step_number=depth + 1,
                    thought=candidate,
                    confidence=1.0 / (i + 1)
                )]
                await explore_branch(candidate, depth + 1, new_path)
        
        # 开始探索
        await explore_branch(query, 0, [])
        
        # 评估并选择最佳路径
        best_path = await self._evaluate_paths(all_paths, query)
        
        final_answer = await self.agent.process(
            f"基于以下思考路径给出最终答案。\n\n"
            f"{self._format_thoughts(best_path)}\n\n"
            f"问题: {query}"
        )
        
        return ReasoningResult(
            strategy=ReasoningStrategy.TREE_OF_THOUGHT,
            thoughts=best_path,
            final_answer=final_answer,
            reasoning_trace=self._format_thoughts(best_path),
            confidence=0.9
        )
    
    def _format_thoughts(self, thoughts: List[Thought]) -> str:
        """格式化思考过程"""
        lines = []
        for t in thoughts:
            lines.append(f"Step {t.step_number}:")
            if t.thought:
                lines.append(f"  Thought: {t.thought}")
            if t.action:
                lines.append(f"  Action: {t.action}")
            if t.observation:
                lines.append(f"  Observation: {t.observation}")
            if t.result:
                lines.append(f"  Result: {t.result}")
            lines.append("")
        return "\n".join(lines)
    
    def _parse_react_response(self, response: str) -> tuple:
        """解析 ReAct 格式响应"""
        thought = action = action_input = None
        
        for line in response.split("\n"):
            if line.startswith("Thought:"):
                thought = line[8:].strip()
            elif line.startswith("Action:"):
                action = line[7:].strip()
            elif line.startswith("Action Input:"):
                action_input = line[13:].strip()
        
        return thought, action, action_input
    
    def _extract_final_answer(self, text: str) -> str:
        """提取最终答案"""
        # 尝试多种模式
        patterns = [
            "最终答案：",
            "答案：",
            "Therefore, ",
            "So the answer is ",
            "答案是"
        ]
        
        for pattern in patterns:
            if pattern in text:
                idx = text.index(pattern) + len(pattern)
                return text[idx:].strip()
        
        # 返回最后一段
        paragraphs = text.split("\n\n")
        return paragraphs[-1].strip() if paragraphs else text
    
    async def _execute_action(
        self,
        action: str,
        action_input: Dict,
        tools: List
    ) -> str:
        """执行动作"""
        
        if not tools:
            return f"No tools available to execute: {action}"
        
        tool = self.tools.get_tool(action)
        if tool:
            result = await tool.execute(action_input)
            return str(result.data) if result.success else result.error
        
        return f"Unknown action: {action}"
```

---

## 多 Agent 系统

### Agent 协作架构

```python
# openclaw/multi_agent/coordination.py
from typing import List, Dict, Any, Optional
from dataclasses import dataclass, field
from enum import Enum
import asyncio

class AgentRole(Enum):
    COORDINATOR = "coordinator"    # 协调者
    RESEARCHER = "researcher"      # 研究员
    CRITIC = "critic"              # 评论员
    EXECUTOR = "executor"          # 执行者
    REPORTER = "reporter"          # 报告者

@dataclass
class AgentConfig:
    """Agent 配置"""
    name: str
    role: AgentRole
    model: str = "gpt-4"
    instructions: str = ""
    tools: List[str] = field(default_factory=list)
    max_iterations: int = 10
    temperature: float = 0.7

@dataclass
class AgentMessage:
    """Agent 间消息"""
    from_agent: str
    to_agent: str  # 或 "all"
    content: str
    message_type: str = "message"  # message, request, response, broadcast
    metadata: Dict[str, Any] = field(default_factory=dict)

class MultiAgentSystem:
    """多 Agent 协作系统"""
    
    def __init__(self):
        self.agents: Dict[str, "Agent"] = {}
        self.roles: Dict[AgentRole, List[str]] = {role: [] for role in AgentRole}
        self.message_queue: asyncio.Queue = asyncio.Queue()
        self.coordinator: Optional[str] = None
    
    def register_agent(self, agent_id: str, config: AgentConfig) -> "Agent":
        """注册 Agent"""
        
        from openclaw.agent import Agent
        
        agent = Agent(
            name=agent_id,
            model=config.model,
            system_prompt=config.instructions,
            tools=[ToolRegistry.get_tool(t) for t in config.tools if ToolRegistry.get_tool(t)]
        )
        
        self.agents[agent_id] = agent
        self.roles[config.role].append(agent_id)
        
        if config.role == AgentRole.COORDINATOR:
            self.coordinator = agent_id
        
        return agent
    
    async def execute_task(self, task: str) -> str:
        """协调执行任务"""
        
        if not self.coordinator:
            raise ValueError("No coordinator agent registered")
        
        # 协调者分解任务
        decomposition = await self._decompose_task(task)
        
        # 并行执行子任务
        sub_tasks = decomposition.get("sub_tasks", [])
        results = await self._execute_parallel(sub_tasks)
        
        # 整合结果
        final_result = await self._synthesize_results(results)
        
        return final_result
    
    async def _decompose_task(self, task: str) -> Dict:
        """协调者分解任务"""
        
        coordinator = self.agents[self.coordinator]
        
        decomposition_prompt = f"""将以下任务分解为可并行执行的子任务。

任务: {task}

考虑以下角色可用：
- Coordinator: 协调整体流程
- Researcher: 收集信息和研究
- Critic: 评估和提出改进建议
- Executor: 执行具体操作
- Reporter: 汇总和报告结果

请提供 JSON 格式的分解方案。
"""
        
        response = await coordinator.process(decomposition_prompt)
        
        # 解析 JSON 响应
        import json
        try:
            # 尝试提取 JSON
            start = response.find("{")
            end = response.rfind("}") + 1
            if start >= 0 and end > start:
                return json.loads(response[start:end])
        except:
            pass
        
        return {"sub_tasks": [{"description": task, "role": "Researcher"}]}
    
    async def _execute_parallel(self, sub_tasks: List[Dict]) -> List[Dict]:
        """并行执行子任务"""
        
        async def execute_single(task: Dict) -> Dict:
            role = AgentRole(task.get("role", "researcher"))
            agent_id = self._get_agent_by_role(role)
            
            if not agent_id:
                return {"task": task, "result": "No agent available", "success": False}
            
            agent = self.agents[agent_id]
            
            result = await agent.process(task.get("description", ""))
            
            return {
                "task": task,
                "result": result,
                "agent": agent_id,
                "success": True
            }
        
        # 并行执行所有子任务
        tasks = [execute_single(t) for t in sub_tasks]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        return [r if isinstance(r, dict) else {"error": str(r)} for r in results]
    
    async def _synthesize_results(self, results: List[Dict]) -> str:
        """整合结果"""
        
        synthesis_prompt = f"""整合以下研究结果，形成最终报告。

结果列表:
{json.dumps(results, ensure_ascii=False, indent=2)}

请提供结构化的综合报告。
"""
        
        reporter_id = self._get_agent_by_role(AgentRole.REPORTER)
        
        if reporter_id:
            reporter = self.agents[reporter_id]
            return await reporter.process(synthesis_prompt)
        
        # 如果没有 Reporter Agent，使用 Coordinator
        if self.coordinator:
            return await self.agents[self.coordinator].process(synthesis_prompt)
        
        # 默认直接返回第一个成功结果
        for r in results:
            if r.get("success"):
                return r.get("result", "")
        
        return "No results available"
    
    def _get_agent_by_role(self, role: AgentRole) -> Optional[str]:
        """根据角色获取 Agent"""
        agents = self.roles.get(role, [])
        return agents[0] if agents else None
```

---

## 定时任务调度

### 调度器实现

```python
# openclaw/scheduler/scheduler.py
from typing import Dict, List, Any, Optional, Callable
from dataclasses import dataclass, field
from datetime import datetime, timedelta
import asyncio
import croniter
import logging

@dataclass
class ScheduledTask:
    """定时任务"""
    id: str
    name: str
    func: Callable
    cron_expression: str
    enabled: bool = True
    timezone: str = "Asia/Shanghai"
    last_run: Optional[datetime] = None
    next_run: Optional[datetime] = None
    config: Dict[str, Any] = field(default_factory=dict)
    
    def calculate_next_run(self, base_time: datetime = None) -> datetime:
        """计算下次运行时间"""
        if base_time is None:
            base_time = datetime.now()
        
        cron = croniter.croniter(self.cron_expression, base_time)
        self.next_run = cron.get_next(datetime)
        return self.next_run

class TaskScheduler:
    """任务调度器"""
    
    def __init__(self):
        self.tasks: Dict[str, ScheduledTask] = {}
        self.logger = logging.getLogger("scheduler")
        self._running = False
        self._check_interval = 60  # 每分钟检查一次
    
    def schedule_task(
        self,
        task_id: str,
        name: str,
        func: Callable,
        cron_expression: str,
        **config
    ) -> ScheduledTask:
        """添加定时任务"""
        
        task = ScheduledTask(
            id=task_id,
            name=name,
            func=func,
            cron_expression=cron_expression,
            config=config
        )
        
        task.calculate_next_run()
        
        self.tasks[task_id] = task
        self.logger.info(
            f"Scheduled task '{name}' with cron '{cron_expression}', "
            f"next run at {task.next_run}"
        )
        
        return task
    
    def schedule_interval(
        self,
        task_id: str,
        name: str,
        func: Callable,
        interval_seconds: int,
        **config
    ) -> ScheduledTask:
        """添加间隔任务"""
        
        # 将间隔转换为 cron 表达式（近似）
        if interval_seconds < 60:
            cron_expr = f"*/{interval_seconds} * * * *"
        elif interval_seconds < 3600:
            minute = interval_seconds // 60
            cron_expr = f"*/{minute} * * * *"
        else:
            hour = interval_seconds // 3600
            cron_expr = f"0 */{hour} * * *"
        
        return self.schedule_task(task_id, name, func, cron_expr, **config)
    
    async def start(self):
        """启动调度器"""
        self._running = True
        self.logger.info("Task scheduler started")
        
        while self._running:
            try:
                now = datetime.now()
                
                # 检查并执行到期的任务
                for task in self.tasks.values():
                    if not task.enabled:
                        continue
                    
                    if task.next_run and now >= task.next_run:
                        await self._execute_task(task)
                
                await asyncio.sleep(self._check_interval)
                
            except Exception as e:
                self.logger.error(f"Scheduler error: {e}")
                await asyncio.sleep(self._check_interval)
    
    def stop(self):
        """停止调度器"""
        self._running = False
        self.logger.info("Task scheduler stopped")
    
    async def _execute_task(self, task: ScheduledTask):
        """执行任务"""
        
        task.last_run = datetime.now()
        task.calculate_next_run()
        
        self.logger.info(f"Executing task: {task.name}")
        
        try:
            if asyncio.iscoroutinefunction(task.func):
                await task.func(**task.config)
            else:
                task.func(**task.config)
            
            self.logger.info(f"Task '{task.name}' completed successfully")
            
        except Exception as e:
            self.logger.error(f"Task '{task.name}' failed: {e}")
            
            # 处理重试
            if task.config.get("retry_on_failure"):
                await self._retry_task(task)
    
    async def _retry_task(self, task: ScheduledTask):
        """重试任务"""
        max_retries = task.config.get("max_retries", 3)
        retry_delay = task.config.get("retry_delay_seconds", 60)
        
        for i in range(max_retries):
            await asyncio.sleep(retry_delay)
            
            try:
                await task.func(**task.config)
                self.logger.info(f"Task '{task.name}' succeeded on retry {i+1}")
                return
            except Exception as e:
                self.logger.error(f"Task '{task.name}' retry {i+1} failed: {e}")
    
    def get_next_runs(self, limit: int = 10) -> List[Dict]:
        """获取即将执行的任务"""
        
        upcoming = [
            {
                "id": task.id,
                "name": task.name,
                "next_run": task.next_run,
                "cron": task.cron_expression
            }
            for task in self.tasks.values()
            if task.next_run
        ]
        
        upcoming.sort(key=lambda x: x["next_run"])
        return upcoming[:limit]
```

### 调度任务使用示例

```python
# 定时任务示例
async def daily_summary_task():
    """每日摘要任务"""
    agent = Agent(model="gpt-4")
    
    # 获取当天数据
    memories = await memory_manager.search_memory(
        query="",
        memory_types=[MemoryType.EPISODIC],
        since=datetime.now() - timedelta(days=1)
    )
    
    # 生成摘要
    summary = await agent.process(
        f"根据以下会话记录生成今日摘要：\n{memories}"
    )
    
    # 保存或发送
    await storage.save("daily_summary", summary)

async def periodic_cleanup_task():
    """定期清理任务"""
    consolidation = ConsolidationService(memory_manager)
    deleted = await consolidation._prune_old_memories()
    logger.info(f"Cleaned up {deleted} old memories")

# 注册定时任务
scheduler = TaskScheduler()

scheduler.schedule_task(
    task_id="daily_summary",
    name="每日摘要生成",
    func=daily_summary_task,
    cron_expression="0 21 * * *",  # 每天晚上9点
    timezone="Asia/Shanghai"
)

scheduler.schedule_task(
    task_id="memory_cleanup",
    name="记忆清理",
    func=periodic_cleanup_task,
    cron_expression="0 3 * * *",  # 每天凌晨3点
    timezone="Asia/Shanghai"
)

# 启动调度器
await scheduler.start()
```

> [!tip] 最佳实践
> - 使用有意义的任务 ID 和名称
> - 为长时间运行的任务设置超时
> - 实现重试机制以处理临时故障
> - 使用幂等操作避免重复执行问题

---

## 完整集成示例

```python
# 完整的多Agent + 工作流 + 定时任务集成示例
async def research_and_report_pipeline():
    """研究并生成报告的完整流程"""
    
    # 1. 创建多 Agent 系统
    multi_agent = MultiAgentSystem()
    
    # 注册各种角色
    multi_agent.register_agent(
        "coordinator_1",
        AgentConfig(
            name="项目协调者",
            role=AgentRole.COORDINATOR,
            instructions="你是一个项目协调专家，负责分解任务和协调团队工作。"
        )
    )
    
    multi_agent.register_agent(
        "researcher_1",
        AgentConfig(
            name="研究员小A",
            role=AgentRole.RESEARCHER,
            instructions="你是一个专业的研究员，擅长收集和分析信息。"
        )
    )
    
    # 2. 定义工作流
    workflow = Workflow(
        id="research_report",
        variables={
            "topic": "2026年AI发展趋势",
            "scope": "全球市场"
        }
    )
    
    workflow.add_step(WorkflowStep(
        id="research",
        name="信息收集",
        step_type=StepType.AGENT,
        config={"agent": "researcher_1"}
    ))
    
    workflow.add_step(WorkflowStep(
        id="analyze",
        name="深度分析",
        step_type=StepType.AGENT,
        depends_on=["research"],
        config={"agent": "researcher_1"}
    ))
    
    # 3. 调度执行
    scheduler = TaskScheduler()
    scheduler.schedule_task(
        task_id="weekly_report",
        name="每周研究报告",
        func=lambda: workflow_engine.execute_workflow(workflow),
        cron_expression="0 9 * * 1"  # 每周一早上9点
    )
    
    await scheduler.start()

# 启动完整系统
asyncio.run(research_and_report_pipeline())
```

> [!summary] 关键要点
> 1. **自定义工具**：通过标准化接口扩展 Agent 能力，支持中间件拦截
> 2. **工作流引擎**：YAML 定义流程，支持条件、循环、并行等复杂逻辑
> 3. **CoT 集成**：内置 Chain of Thought、ReAct、Tree of Thought 等推理策略
> 4. **多 Agent**：角色化设计，支持协作、批评、报告等多种模式
> 5. **定时任务**：Cron 表达式调度，支持重试、时区等高级功能

---

## 相关文档

- [[../多平台集成/OpenClaw多平台集成]] - 多平台消息处理
- [[../插件开发/OpenClaw插件开发]] - 扩展工具开发
- [[../记忆系统/OpenClaw记忆系统]] - 上下文管理
- [[OpenClaw概览]] - 系统整体架构
