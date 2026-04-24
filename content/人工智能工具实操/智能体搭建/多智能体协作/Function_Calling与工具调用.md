---
title: Function Calling与工具调用：AI Agent的核心能力
date: 2026-04-24
tags:
  - Function Calling
  - 工具调用
  - OpenAI
  - Claude
  - Gemini
  - API规范
  - 智能体
categories:
  - 智能体搭建
  - 多智能体协作
description: 详细介绍Function Calling的原理与实现，对比OpenAI、Claude、Gemini等主流模型的工具调用规范，提供完整的工具定义、执行流程和实战代码示例。
---

# Function Calling与工具调用：AI Agent的核心能力

> [!NOTE] 这篇指南讲什么
> Function Calling（函数调用）是现代AI Agent的核心能力，让大语言模型能够"行动"起来。这篇指南从原理到实现，详细讲解主流模型的工具调用规范和实战技巧。

## 什么是Function Calling？

### 从"能说"到"能做"

普通的大语言模型只能"说话"——你问它问题，它给你文字回答。但Function Calling让AI能够"行动"：

**没有Function Calling：**
```
用户: 帮我查一下北京今天的天气
AI: 北京今天天气晴朗，气温15-25度，适宜出行。
```

**有Function Calling：**
```
用户: 帮我查一下北京今天的天气
AI: (调用get_weather) → 工具返回结果 → AI整合回答
```

关键区别在于：AI不只是在"回答"问题，而是能真正"做"事情。

### Function Calling的工作原理

```
┌─────────────────────────────────────────────────────────────┐
│                    Function Calling 完整流程                   │
│                                                            │
│  用户问题                                                     │
│      ↓                                                      │
│  ┌──────────────────┐                                       │
│  │  LLM理解用户意图   │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│  ┌──────────────────┐                                       │
│  │  判断需要调用工具   │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│  ┌──────────────────┐                                       │
│  │  生成调用请求      │                                       │
│  │  (函数名+参数)     │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│  ┌──────────────────┐                                       │
│  │  执行工具函数      │                                       │
│  │  (API调用/计算等)  │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│  ┌──────────────────┐                                       │
│  │  返回结果给LLM    │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│  ┌──────────────────┐                                       │
│  │  LLM整合结果      │                                       │
│  │  生成最终回答      │                                       │
│  └────────┬─────────┘                                       │
│           ↓                                                  │
│        用户得到答案                                           │
└─────────────────────────────────────────────────────────────┘
```

## 主流模型Function Calling对比

### 平台对比

| 平台 | 规范名称 | 特点 | 适用场景 |
|------|----------|------|----------|
| **OpenAI** | function_call | 结构化强、广泛支持 | GPT-4o、GPT-4 |
| **Claude** | tool_use | 简洁清晰、安全优先 | Claude 3.5 Sonnet |
| **Gemini** | function_declarations | 多模态集成 | Gemini 1.5 Pro |
| **DeepSeek** | tools | 国产、成本低 | DeepSeek-V3 |

### OpenAI Function Calling

#### 工具定义格式

```yaml
# OpenAI tools格式
tools:
  - type: function
    function:
      name: "get_weather"           # 函数名称（必须唯一）
      description: "获取城市天气信息"  # 描述模型如何使用
      
      parameters:                   # 参数Schema（JSON Schema格式）
        type: object
        properties:
          city:
            type: string
            description: "城市名称，如北京、上海"
            # 支持正则约束
          unit:
            type: string
            enum: ["celsius", "fahrenheit"]  # 枚举值
            default: "celsius"                  # 默认值
            description: "温度单位"
        required: ["city"]  # 必填参数
        additionalProperties: false
```

#### 完整调用示例

```python
from openai import OpenAI
import json
from typing import List, Dict, Any

client = OpenAI(api_key="your-api-key")

# 1. 定义工具
tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "获取指定城市的天气信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {
                        "type": "string",
                        "description": "城市名称，支持中英文"
                    },
                    "unit": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "default": "celsius",
                        "description": "温度单位"
                    }
                },
                "required": ["city"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "send_email",
            "description": "发送电子邮件",
            "parameters": {
                "type": "object",
                "properties": {
                    "to": {
                        "type": "string",
                        "description": "收件人邮箱地址"
                    },
                    "subject": {
                        "type": "string",
                        "description": "邮件主题"
                    },
                    "body": {
                        "type": "string",
                        "description": "邮件正文内容"
                    }
                },
                "required": ["to", "subject", "body"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "search_database",
            "description": "查询数据库中的订单信息",
            "parameters": {
                "type": "object",
                "properties": {
                    "order_id": {
                        "type": "string",
                        "description": "订单号"
                    },
                    "customer_id": {
                        "type": "string",
                        "description": "客户ID"
                    }
                },
                "required": []
            }
        }
    }
]

# 2. 构建对话消息
messages = [
    {
        "role": "system",
        "content": """你是一个智能助手，可以调用工具来完成任务。

当用户询问天气时，调用 get_weather 工具。
当用户要求发送邮件时，调用 send_email 工具。
当用户查询订单时，调用 search_database 工具。

如果用户的问题不需要工具，直接回答即可。"""
    },
    {
        "role": "user",
        "content": "帮我查一下北京今天的天气，如果下雨就发邮件提醒我带伞"
    }
]

# 3. 调用API（第一次）
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice="auto"  # auto让模型决定，none禁止调用，"function_name"强制调用
)

assistant_message = response.choices[0].message

# 4. 处理响应
if assistant_message.tool_calls:
    # 模型请求调用工具
    for tool_call in assistant_message.tool_calls:
        function_name = tool_call.function.name
        arguments = json.loads(tool_call.function.arguments)
        
        print(f"需要调用函数: {function_name}")
        print(f"参数: {arguments}")
        
        # 执行工具
        if function_name == "get_weather":
            result = execute_weather_api(arguments['city'], arguments.get('unit', 'celsius'))
            
            # 添加工具结果到消息
            messages.append({
                "role": "assistant",
                "content": assistant_message.content,
                "tool_calls": [
                    {
                        "id": tool_call.id,
                        "function": {
                            "name": function_name,
                            "arguments": tool_call.function.arguments
                        },
                        "type": "function"
                    }
                ]
            })
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(result)
            })
        
        elif function_name == "send_email":
            result = execute_send_email(arguments['to'], arguments['subject'], arguments['body'])
            # 处理结果...
    
    # 5. 第二次调用（带工具结果）
    final_response = client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=tools
    )
    
    print("最终回答:")
    print(final_response.choices[0].message.content)
else:
    # 没有工具调用，直接回答
    print("直接回答:", assistant_message.content)


# 工具执行函数示例
def execute_weather_api(city: str, unit: str = "celsius") -> Dict[str, Any]:
    """模拟天气API调用"""
    # 实际项目中，这里会调用真实的天气API
    return {
        "city": city,
        "temperature": 22,
        "unit": unit,
        "condition": "晴",
        "humidity": 45,
        "wind": "东南风3级"
    }

def execute_send_email(to: str, subject: str, body: str) -> Dict[str, Any]:
    """模拟发送邮件"""
    # 实际项目中，这里会调用真实的邮件服务
    return {
        "success": True,
        "message_id": f"email_{int(time.time())}",
        "recipient": to,
        "sent_at": datetime.now().isoformat()
    }
```

#### 强制调用指定工具

```python
# 强制使用特定工具
response = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=tools,
    tool_choice={
        "type": "function",
        "function": {"name": "get_weather"}
    }
)
```

### Claude Function Calling

#### 工具定义格式

```yaml
# Claude tools格式（更简洁）
tools:
  - name: "get_weather"
    description: "获取城市天气信息"
    input_schema:
      type: object
      properties:
        city:
          type: string
          description: "城市名称"
        unit:
          type: string
          enum: ["celsius", "fahrenheit"]
          default: "celsius"
      required: ["city"]
```

#### 完整调用示例

```python
from anthropic import Anthropic

client = Anthropic(api_key="your-api-key")

# 1. 定义工具
tools = [
    {
        "name": "get_weather",
        "description": "获取城市天气信息",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "城市名称"
                },
                "unit": {
                    "type": "string",
                    "enum": ["celsius", "fahrenheit"],
                    "description": "温度单位"
                }
            },
            "required": ["city"]
        }
    },
    {
        "name": "calculate",
        "description": "执行数学计算",
        "input_schema": {
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "数学表达式，如 2+3*4"
                }
            },
            "required": ["expression"]
        }
    }
]

# 2. 构建消息
messages = [
    {
        "role": "user",
        "content": "北京今天多少度？帮我算一下20度转换成华氏度是多少。"
    }
]

# 3. 调用API
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    tools=tools,
    messages=messages
)

# 4. 处理响应
for content in response.content:
    if content.type == "tool_use":
        tool_name = content.name
        tool_input = content.input
        
        print(f"调用工具: {tool_name}")
        print(f"参数: {tool_input}")
        
        # 执行工具
        if tool_name == "get_weather":
            result = execute_weather_api(tool_input['city'])
        elif tool_name == "calculate":
            result = eval(tool_input['expression'])
        
        # 添加工具结果
        messages.append({
            "role": "assistant",
            "content": response.content
        })
        messages.append({
            "role": "user",
            "content": [{
                "type": "tool_result",
                "tool_use_id": content.id,
                "content": str(result)
            }]
        })

# 5. 第二次调用
final_response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=1024,
    tools=tools,
    messages=messages
)

print(final_response.content[0].text)
```

#### Claude的特殊能力

```python
# Claude的扩展思考模式
response = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=8192,
    thinking={
        "type": "enabled",
        "budget_tokens": 1024  # 思考预算
    },
    tools=tools,
    messages=messages
)
```

### Gemini Function Calling

#### 工具定义格式

```yaml
# Gemini function declarations
tools:
  - function_declarations:
      - name: "get_weather"
        description: "获取城市天气信息"
        parameters:
          type: "OBJECT"
          properties:
            city:
              type: "STRING"
              description: "城市名称"
            unit:
              type: "STRING"
              enum: ["celsius", "fahrenheit"]
          required: ["city"]
```

#### 完整调用示例

```python
import google.generativeai as genai

genai.configure(api_key="your-api-key")

# 1. 定义工具
tools = [
    {
        "function_declarations": [
            {
                "name": "get_weather",
                "description": "获取城市天气信息",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "city": {
                            "type": "string",
                            "description": "城市名称"
                        },
                        "unit": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"]
                        }
                    },
                    "required": ["city"]
                }
            },
            {
                "name": "search_web",
                "description": "搜索网络信息",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "query": {
                            "type": "string",
                            "description": "搜索关键词"
                        }
                    },
                    "required": ["query"]
                }
            }
        ]
    }
]

# 2. 创建模型
model = genai.GenerativeModel(
    model_name="gemini-1.5-pro",
    tools=tools
)

# 3. 开始对话
chat = model.start_chat()

response = chat.send_message("北京今天天气如何？")

# 4. 处理函数调用
for candidate in response.candidates:
    for part in candidate.content.parts:
        if part.function_call:
            func = part.function_call
            print(f"调用函数: {func.name}")
            print(f"参数: {dict(func.args)}")
            
            # 执行函数
            if func.name == "get_weather":
                result = execute_weather_api(
                    func.args['city'],
                    func.args.get('unit', 'celsius')
                )
            
            # 发送函数结果
            response = chat.send_message(
                Part(
                    function_response=FunctionResponse(
                        name=func.name,
                        response={"result": result}
                    )
                )
            )

print(response.text)
```

## 工具执行框架

### 统一工具注册表

```python
from typing import Dict, Callable, Any, Optional, List
from dataclasses import dataclass, field
import inspect
import json
from enum import Enum

class ToolRegistry:
    """统一工具注册表"""
    
    def __init__(self):
        self._tools: Dict[str, Dict[str, Any]] = {}
    
    def register(
        self,
        name: str,
        description: str,
        parameters: Optional[Dict] = None
    ):
        """装饰器注册工具"""
        def decorator(func: Callable):
            sig = inspect.signature(func)
            
            # 自动从函数签名提取参数定义
            if parameters is None:
                params = self._extract_params_from_signature(sig)
            else:
                params = parameters
            
            self._tools[name] = {
                "name": name,
                "description": description,
                "func": func,
                "parameters": params,
                "is_async": inspect.iscoroutinefunction(func)
            }
            return func
        return decorator
    
    def _extract_params_from_signature(
        self,
        sig: inspect.Signature
    ) -> Dict[str, Any]:
        """从函数签名提取参数定义"""
        params = {
            "type": "object",
            "properties": {},
            "required": []
        }
        
        for name, param in sig.parameters.items():
            # 跳过 self 参数
            if name in ('self', 'cls'):
                continue
            
            # 根据类型注解确定参数类型
            param_type = "string"
            if param.annotation == int:
                param_type = "integer"
            elif param.annotation == float:
                param_type = "number"
            elif param.annotation == bool:
                param_type = "boolean"
            elif param.annotation == List[str] or param.annotation == list:
                param_type = "array"
            elif param.annotation == Dict or param.annotation == dict:
                param_type = "object"
            
            param_info = {"type": param_type}
            
            # 添加描述（如果有）
            if param.default != inspect.Parameter.empty:
                param_info["default"] = param.default
            elif name not in params["required"]:
                params["required"].append(name)
            
            params["properties"][name] = param_info
        
        return params
    
    def get_openai_format(self) -> List[Dict]:
        """转换为OpenAI工具格式"""
        return [
            {
                "type": "function",
                "function": {
                    "name": tool["name"],
                    "description": tool["description"],
                    "parameters": tool["parameters"]
                }
            }
            for tool in self._tools.values()
        ]
    
    def get_claude_format(self) -> List[Dict]:
        """转换为Claude工具格式"""
        return [
            {
                "name": tool["name"],
                "description": tool["description"],
                "input_schema": tool["parameters"]
            }
            for tool in self._tools.values()
        ]
    
    async def execute(
        self,
        name: str,
        arguments: Dict[str, Any]
    ) -> Any:
        """执行工具"""
        if name not in self._tools:
            raise ValueError(f"Unknown tool: {name}")
        
        tool = self._tools[name]
        
        # 参数验证
        validated_args = self._validate_arguments(name, arguments, tool)
        
        # 执行
        if tool["is_async"]:
            return await tool["func"](**validated_args)
        else:
            return tool["func"](**validated_args)
    
    def _validate_arguments(
        self,
        name: str,
        arguments: Dict,
        tool: Dict
    ) -> Dict:
        """验证参数"""
        validated = {}
        required = tool["parameters"].get("required", [])
        properties = tool["parameters"].get("properties", {})
        
        for param_name in required:
            if param_name not in arguments:
                raise ValueError(f"Missing required parameter: {param_name}")
            validated[param_name] = arguments[param_name]
        
        for param_name, value in arguments.items():
            if param_name in properties:
                validated[param_name] = value
        
        return validated


# 使用示例
registry = ToolRegistry()

@registry.register(
    name="get_weather",
    description="获取城市天气信息"
)
def get_weather(city: str, unit: str = "celsius") -> dict:
    """获取天气"""
    # 实际API调用
    return {
        "city": city,
        "temperature": 22,
        "unit": unit,
        "condition": "晴"
    }

@registry.register(
    name="search_web",
    description="搜索网络信息"
)
async def search_web(query: str, limit: int = 5) -> list:
    """网络搜索"""
    # 实际搜索逻辑
    return [{"title": "result", "url": "..."}]

@registry.register(
    name="send_email",
    description="发送邮件"
)
def send_email(to: str, subject: str, body: str) -> dict:
    """发送邮件"""
    # 实际发送逻辑
    return {"success": True, "message_id": "..."}

# 获取所有工具
tools = registry.get_openai_format()
```

### Agent执行循环

```python
import asyncio
from typing import List, Dict, Any, Optional
import json
from dataclasses import dataclass

@dataclass
class AgentConfig:
    """Agent配置"""
    model: str = "gpt-4o"
    max_turns: int = 10
    temperature: float = 0.7
    system_prompt: str = ""

class ToolUsingAgent:
    """工具调用Agent"""
    
    def __init__(
        self,
        registry: ToolRegistry,
        config: AgentConfig = None
    ):
        self.registry = registry
        self.config = config or AgentConfig()
        self.client = OpenAI()
    
    async def chat(
        self,
        user_message: str,
        context: Optional[Dict] = None
    ) -> str:
        """对话循环"""
        messages = []
        
        # 添加系统提示词
        if self.config.system_prompt:
            messages.append({
                "role": "system",
                "content": self.config.system_prompt
            })
        
        # 添加初始上下文
        if context:
            messages.append({
                "role": "system",
                "content": f"当前上下文：{json.dumps(context)}"
            })
        
        # 添加用户消息
        messages.append({
            "role": "user",
            "content": user_message
        })
        
        # 执行循环
        for turn in range(self.config.max_turns):
            # 调用模型
            response = self.client.chat.completions.create(
                model=self.config.model,
                messages=messages,
                tools=self.registry.get_openai_format(),
                tool_choice="auto",
                temperature=self.config.temperature
            )
            
            assistant_message = response.choices[0].message
            messages.append(assistant_message)
            
            # 检查是否有工具调用
            if not assistant_message.tool_calls:
                # 没有工具调用，返回最终回答
                return assistant_message.content
            
            # 处理工具调用
            tool_results = []
            for tool_call in assistant_message.tool_calls:
                try:
                    result = await self.registry.execute(
                        tool_call.function.name,
                        json.loads(tool_call.function.arguments)
                    )
                    
                    tool_results.append({
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "content": json.dumps(result)
                    })
                    
                except Exception as e:
                    tool_results.append({
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "content": json.dumps({"error": str(e)})
                    })
            
            # 添加工具结果
            messages.extend(tool_results)
        
        return "对话达到最大轮次限制"
    
    async def stream_chat(
        self,
        user_message: str,
        context: Optional[Dict] = None
    ):
        """流式对话"""
        # 构建消息（与上面类似）
        messages = self._build_messages(user_message, context)
        
        # 收集完整响应
        full_content = ""
        tool_calls_to_process = []
        
        # 流式调用
        stream = self.client.chat.completions.create(
            model=self.config.model,
            messages=messages,
            tools=self.registry.get_openai_format(),
            stream=True
        )
        
        for chunk in stream:
            delta = chunk.choices[0].delta
            
            # 收集文本内容
            if delta.content:
                full_content += delta.content
                yield {"type": "content", "content": delta.content}
            
            # 收集工具调用
            if delta.tool_calls:
                for tc in delta.tool_calls:
                    if len(tool_calls_to_process) <= tc.index:
                        tool_calls_to_process.append({
                            "id": tc.id,
                            "name": "",
                            "arguments": ""
                        })
                    tool_calls_to_process[tc.index]["id"] = tc.id or tool_calls_to_process[tc.index]["id"]
                    if tc.function:
                        tool_calls_to_process[tc.index]["name"] = tc.function.name or tool_calls_to_process[tc.index]["name"]
                        tool_calls_to_process[tc.index]["arguments"] += tc.function.arguments or ""
        
        # 如果有工具调用，执行工具
        if tool_calls_to_process:
            for tc in tool_calls_to_process:
                try:
                    result = await self.registry.execute(
                        tc["name"],
                        json.loads(tc["arguments"])
                    )
                    yield {"type": "tool_result", "tool": tc["name"], "result": result}
                except Exception as e:
                    yield {"type": "error", "tool": tc["name"], "error": str(e)}


# 使用示例
async def main():
    registry = ToolRegistry()
    
    # 注册工具
    @registry.register(name="get_weather", description="获取天气")
    def get_weather(city: str, unit: str = "celsius"):
        return {"city": city, "temp": 22, "unit": unit}
    
    # 创建Agent
    agent = ToolUsingAgent(
        registry=registry,
        config=AgentConfig(
            system_prompt="你是一个智能助手，可以用工具完成任务。"
        )
    )
    
    # 对话
    response = await agent.chat("北京今天天气怎么样？")
    print(response)


# 运行
asyncio.run(main())
```

## 工具设计最佳实践

### 设计原则

**原则一：单一职责**

```yaml
# ✅ 好的设计：一个工具做一件事
tools:
  - name: get_weather
    description: 获取城市天气信息
  
  - name: send_email
    description: 发送邮件

# ❌ 不好的设计：一个工具做很多事
tools:
  - name: handle_everything
    description: 处理天气、邮件、订单、客服...（太宽泛）
```

**原则二：清晰的命名**

```yaml
# ✅ 清晰的命名
tools:
  - name: get_order_status
    description: 查询订单物流状态
  
  - name: search_product_catalog
    description: 搜索产品目录

# ❌ 模糊的命名
tools:
  - name: get_info
    description: 获取信息
  
  - name: do_something
    description: 做点什么
```

**原则三：准确的描述**

```yaml
# ✅ 好的描述
tools:
  - name: get_weather
    description: |
      获取指定城市的天气信息
      
      输入：城市名称（必填，支持中英文）
      输出：温度、天气状况、湿度、风力等信息
      
      适用：用户问"天气怎么样"、查温度、问穿衣建议等
      
      不适用：查询天气预报超过7天

# ❌ 差的描述
tools:
  - name: get_weather
    description: 查询天气
```

**原则四：参数设计规范**

```yaml
# ✅ 参数设计规范
tools:
  - name: create_task
    parameters:
      type: object
      properties:
        title:
          type: string
          description: "任务标题，2-100个字符"
          minLength: 2
          maxLength: 100
        
        due_date:
          type: string
          format: date
          description: "截止日期，格式YYYY-MM-DD"
        
        priority:
          type: string
          enum: ["low", "medium", "high"]
          default: "medium"
          description: "优先级"
      
      required: ["title"]
```

### 常见问题处理

**问题一：参数验证失败**

```python
# 工具执行时进行参数验证
def execute_tool(tool_name: str, arguments: Dict):
    try:
        # 参数验证
        validated = validate_arguments(tool_name, arguments)
        
        # 执行
        result = tool_functions[tool_name](**validated)
        return {"success": True, "data": result}
    
    except ValidationError as e:
        return {"success": False, "error": f"参数错误: {e}"}
    except ToolExecutionError as e:
        return {"success": False, "error": f"执行失败: {e}"}
    except Exception as e:
        return {"success": False, "error": f"未知错误: {e}"}
```

**问题二：工具超时**

```python
import asyncio

async def execute_with_timeout(tool_func, args, timeout=30):
    """带超时的工具执行"""
    try:
        result = await asyncio.wait_for(
            tool_func(**args),
            timeout=timeout
        )
        return {"success": True, "data": result}
    except asyncio.TimeoutError:
        return {"success": False, "error": "工具执行超时"}
```

**问题三：工具不可用**

```python
# 工具注册时标记可选
class ToolRegistry:
    def __init__(self):
        self._tools = {}
        self._fallbacks = {}
    
    def register(self, name, func, fallback=None):
        self._tools[name] = func
        if fallback:
            self._fallbacks[name] = fallback
    
    async def execute(self, name, args):
        if name not in self._tools:
            if name in self._fallbacks:
                return await self._fallbacks[name](args)
            raise ValueError(f"Tool {name} not found")
        
        try:
            return await self._tools[name](**args)
        except Exception as e:
            if name in self._fallbacks:
                return await self._fallbacks[name](args, error=e)
            raise
```

## 安全考虑

### 权限控制

```python
class SecureToolRegistry(ToolRegistry):
    """安全工具注册表"""
    
    def __init__(self):
        super().__init__()
        self._permissions = {}
        self._audit_log = []
    
    def register(self, name, func, required_permissions=None):
        """注册带权限控制的工具"""
        super().register(name, func)
        if required_permissions:
            self._permissions[name] = required_permissions
    
    async def execute(self, name, args, user_context=None):
        """带权限检查的执行"""
        # 权限检查
        required = self._permissions.get(name, [])
        if required:
            user_perms = user_context.get("permissions", [])
            if not any(p in user_perms for p in required):
                raise PermissionError(f"User lacks required permissions for {name}")
        
        # 审计日志
        self._audit_log.append({
            "tool": name,
            "args": args,
            "user": user_context.get("user_id"),
            "timestamp": datetime.now().isoformat()
        })
        
        return await super().execute(name, args)
```

### 输入过滤

```python
class SecureToolRegistry:
    """带输入过滤的工具注册表"""
    
    def __init__(self):
        super().__init__()
        self._dangerous_patterns = [
            "DROP TABLE",
            "DELETE FROM",
            "exec(",
            "eval(",
            "<script>",
            "javascript:"
        ]
    
    def _sanitize_input(self, value: Any) -> Any:
        """清理危险输入"""
        if isinstance(value, str):
            for pattern in self._dangerous_patterns:
                if pattern.lower() in value.lower():
                    raise ValueError(f"Potentially dangerous input detected")
            return value.strip()
        elif isinstance(value, dict):
            return {k: self._sanitize_input(v) for k, v in value.items()}
        elif isinstance(value, list):
            return [self._sanitize_input(v) for v in value]
        return value
    
    async def execute(self, name, args):
        """执行前过滤输入"""
        sanitized_args = self._sanitize_input(args)
        return await super().execute(name, sanitized_args)
```

## 高级技巧

### 动态工具选择

```python
class DynamicToolAgent:
    """动态选择可用工具的Agent"""
    
    def __init__(self, registry: ToolRegistry):
        self.registry = registry
    
    async def chat(self, message, available_tools=None):
        """根据上下文动态选择工具"""
        # 分析用户需求
        needs = self._analyze_needs(message)
        
        # 过滤可用工具
        if available_tools:
            filtered_tools = [
                t for t in self.registry.get_all_tools()
                if t["name"] in available_tools
            ]
        else:
            filtered_tools = self.registry.get_all_tools()
        
        # 调用模型
        response = self.client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": message}],
            tools=filtered_tools
        )
        
        return response
```

### 工具链式调用

```python
class ToolChain:
    """工具链 - 顺序执行多个工具"""
    
    def __init__(self, registry: ToolRegistry):
        self.registry = registry
    
    async def execute(
        self,
        chain: List[Dict[str, Any]]
    ) -> List[Dict]:
        """
        执行工具链
        
        chain: [
            {"tool": "get_user", "args": {"user_id": "123"}},
            {"tool": "get_orders", "args": {"customer_id": "{{get_user.customer_id}}"}}
        ]
        """
        results = []
        context = {}
        
        for step in chain:
            tool_name = step["tool"]
            args = step["args"]
            
            # 支持引用前一步结果
            resolved_args = self._resolve_args(args, context)
            
            # 执行
            result = await self.registry.execute(tool_name, resolved_args)
            results.append(result)
            context[tool_name] = result
        
        return results
    
    def _resolve_args(self, args, context):
        """解析参数中的引用"""
        resolved = {}
        for key, value in args.items():
            if isinstance(value, str) and value.startswith("{{") and value.endswith("}}"):
                # 解析引用
                ref = value[2:-2]  # 去掉 {{ 和 }}
                parts = ref.split(".")
                result = context
                for part in parts:
                    result = result.get(part)
                resolved[key] = result
            else:
                resolved[key] = value
        return resolved
```

### 并行工具执行

```python
class ParallelToolExecutor:
    """并行工具执行器"""
    
    def __init__(self, registry: ToolRegistry, max_concurrency=5):
        self.registry = registry
        self.semaphore = asyncio.Semaphore(max_concurrency)
    
    async def execute_parallel(
        self,
        calls: List[Dict[str, Any]]
    ) -> List[Dict]:
        """并行执行多个工具调用"""
        async def execute_with_limit(call):
            async with self.semaphore:
                return await self.registry.execute(
                    call["tool"],
                    call["args"]
                )
        
        tasks = [execute_with_limit(call) for call in calls]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        
        # 处理异常
        processed_results = []
        for i, result in enumerate(results):
            if isinstance(result, Exception):
                processed_results.append({
                    "success": False,
                    "error": str(result),
                    "tool": calls[i]["tool"]
                })
            else:
                processed_results.append({
                    "success": True,
                    "data": result,
                    "tool": calls[i]["tool"]
                })
        
        return processed_results
```

## 总结

Function Calling是AI Agent的核心能力，让AI从"能说"进化到"能做"。

**核心要点：**

1. **工具定义要规范** - 清晰的描述、准确的参数
2. **执行要有容错** - 验证参数、处理异常、设置超时
3. **安全要重视** - 权限控制、输入过滤、审计日志
4. **体验要优化** - 流式响应、并行执行、智能选择

---

## 相关资源

- [[多Agent系统设计]] - 多智能体协作架构
- [[n8n与LLM集成]] - n8n工具调用配置
- [[扣子Bot开发]] - Coze插件开发
- [[AI平台插件开发]] - 插件开发指南

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
