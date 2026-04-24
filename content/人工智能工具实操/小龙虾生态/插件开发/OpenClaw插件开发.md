---
title: OpenClaw 插件开发完全指南
date: 2026-04-24
tags:
  - OpenClaw
  - 插件开发
  - Plugin
  - Python
  - 工具开发
categories:
  - 人工智能工具实操
  - 小龙虾生态
description: 从零开始学习 OpenClaw 插件开发，手把手教你写一个天气查询插件。
---

# OpenClaw 插件开发完全指南

> 想给 OpenClaw 增加新功能？或者想让 AI 能做特定的事？这篇教你从零开发一个插件。

---

## 插件是什么？

### 一句话解释

**插件 = 给 AI 装一个"超能力"**

就像手机装 App 获得新功能，OpenClaw 装插件获得新能力：
- 装个天气插件 → AI 能查天气
- 装个翻译插件 → AI 能翻译
- 装个 GitHub 插件 → AI 能管代码

### 为什么要有插件系统？

因为 AI 本身不会这些事！

GPT/Claude 只知道文字处理，但：
- 查天气 → 需要调用天气 API
- 读写文件 → 需要文件系统访问
- 管 GitHub → 需要 GitHub API

插件就是连接 AI 和这些"外部能力"的桥梁。

---

## 插件是怎么工作的？

### 整体流程

```
用户说："帮我查下北京天气"

    │
    ▼

OpenClaw 收到消息

    │
    ▼

AI 大脑（Pi Agent）分析：
"用户想查天气，我需要调用 weather 工具"

    │
    ▼

OpenClaw 调用插件里的 weather 工具

    │
    ▼

weather 插件调用天气 API

    │
    ▼

API 返回天气数据

    │
    ▼

插件把数据返回给 AI

    │
    ▼

AI 把结果告诉用户
```

### 插件的组成

```
my_plugin/
├── plugin.json      # 插件信息
├── __init__.py    # 入口文件
├── tools/          # 工具目录
│   ├── __init__.py
│   └── weather.py  # 天气工具
└── handlers/        # 事件处理（可选）
    └── event.py
```

---

## 开始开发：做一个天气插件

### 第一步：创建项目结构

```bash
# 创建插件目录
mkdir -p my_weather_plugin/tools
mkdir -p my_weather_plugin/handlers

# 进入目录
cd my_weather_plugin
```

### 第二步：创建 plugin.json

这是插件的"身份证"：

```json
{
  "name": "my-weather",
  "version": "1.0.0",
  "description": "查询天气的插件",
  "author": "你的名字",
  "license": "MIT",

  "config_schema": {
    "api_key": {
      "type": "string",
      "required": true,
      "description": "天气 API 的 Key"
    },
    "default_city": {
      "type": "string",
      "default": "北京",
      "description": "默认查询城市"
    }
  }
}
```

### 第三步：写工具代码

创建 `tools/__init__.py`：

```python
from .weather import WeatherTool

__all__ = ["WeatherTool"]
```

创建 `tools/weather.py`：

```python
from openclaw.tools import Tool, ToolResult
from pydantic import BaseModel, Field
import httpx
from typing import Optional

# 定义输入参数
class WeatherInput(BaseModel):
    """天气查询的输入参数"""
    city: str = Field(description="城市名称，比如'北京'、'上海'")
    units: str = Field(default="celsius", description="温度单位：celsius（摄氏度）或 fahrenheit（华氏度）")

# 定义工具类
class WeatherTool(Tool):
    """天气查询工具"""

    name = "weather"
    description = "查询指定城市的天气信息，返回温度、天气状况、湿度等"
    input_model = WeatherInput

    def __init__(self, api_key: str):
        super().__init__()
        self.api_key = api_key
        self.base_url = "https://api.openweathermap.org/data/2.5"

    async def execute(self, input_data: WeatherInput) -> ToolResult:
        """执行天气查询"""

        try:
            # 调用天气 API
            params = {
                "q": input_data.city,
                "appid": self.api_key,
                "units": "metric" if input_data.units == "celsius" else "imperial"
            }

            async with httpx.AsyncClient() as client:
                response = await client.get(
                    f"{self.base_url}/weather",
                    params=params
                )
                response.raise_for_status()
                data = response.json()

            # 解析结果
            temp = data["main"]["temp"]
            weather = data["weather"][0]["description"]
            humidity = data["main"]["humidity"]
            city = data["name"]

            # 构建返回
            unit_symbol = "°C" if input_data.units == "celsius" else "°F"
            result = f"{city}当前天气：\n"
            result += f"- 温度：{temp}{unit_symbol}\n"
            result += f"- 天气：{weather}\n"
            result += f"- 湿度：{humidity}%"

            return ToolResult(
                success=True,
                output=result,
                metadata={
                    "city": city,
                    "temperature": temp,
                    "weather": weather,
                    "humidity": humidity
                }
            )

        except httpx.HTTPStatusError as e:
            return ToolResult(
                success=False,
                error=f"API 请求失败：{e.response.status_code}"
            )
        except Exception as e:
            return ToolResult(
                success=False,
                error=f"查询失败：{str(e)}"
            )
```

### 第四步：创建入口文件

创建 `__init__.py`：

```python
from openclaw.plugins import Plugin
from .tools import WeatherTool
from typing import List

class Plugin(Plugin):
    """天气插件"""

    name = "my-weather"
    version = "1.0.0"
    description = "查询天气的插件"

    def __init__(self):
        super().__init__()
        self._weather_tool = None

    async def on_load(self) -> bool:
        """插件加载时调用"""

        # 获取 API Key
        api_key = self.context.config.get("api_key")
        if not api_key:
            self.context.logger.error("缺少 api_key 配置")
            return False

        # 初始化工具
        self._weather_tool = WeatherTool(api_key=api_key)

        self.context.logger.info("天气插件加载成功")
        return True

    def get_tools(self) -> List:
        """返回插件提供的工具"""
        return [self._weather_tool] if self._weather_tool else []
```

---

## 插件配置文件

### plugin.json 完整示例

```json
{
  "name": "my-plugin",
  "version": "1.0.0",
  "author": "你的名字",
  "description": "插件描述",

  "license": "MIT",
  "homepage": "https://github.com/you/my-plugin",

  "dependencies": [],
  "compatible_versions": ">=2.0.0",

  "config_schema": {
    "api_key": {
      "type": "string",
      "required": true,
      "secret": true,
      "description": "API 密钥"
    },
    "timeout": {
      "type": "integer",
      "default": 30,
      "description": "超时时间（秒）"
    }
  },

  "permissions": [
    "network:external"
  ]
}
```

### 配置项类型

| 类型 | 说明 | 示例 |
|------|------|------|
| `string` | 字符串 | `"hello"` |
| `integer` | 整数 | `123` |
| `boolean` | 布尔值 | `true` |
| `array` | 数组 | `["a", "b"]` |
| `object` | 对象 | `{"key": "value"}` |

---

## 工具类详解

### 基础工具类

```python
from openclaw.tools import Tool, ToolResult

class MyTool(Tool):
    name = "my_tool"           # 工具名（AI 会看到这个）
    description = "工具描述"   # AI 根据这个决定何时调用
    input_model = MyInput       # 输入参数模型

    async def execute(self, input_data: MyInput) -> ToolResult:
        # 工具逻辑
        return ToolResult(
            success=True,       # 是否成功
            output="结果",       # 成功时返回
            metadata={}          # 额外数据
        )
```

### ToolResult 返回结构

```python
ToolResult(
    success=True,              # 必须：是否成功
    output="结果内容",         # 成功时返回的内容
    error="错误信息",          # 失败时返回的错误
    error_code="ERROR_CODE",   # 错误码（可选）
    metadata={                 # 额外元数据（可选）
        "extra_info": "..."
    }
)
```

### 输入参数模型

用 Pydantic 定义输入参数：

```python
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    """搜索工具的输入参数"""

    query: str = Field(
        description="搜索关键词",
        min_length=1,
        max_length=200
    )

    limit: int = Field(
        default=10,
        description="返回结果数量",
        ge=1,
        le=100
    )

    language: str = Field(
        default="zh",
        description="搜索语言"
    )
```

---

## 事件处理

### 什么是事件处理？

除了提供工具，插件还能"监听"某些事件：

```python
class MyPlugin(Plugin):
    def get_event_handlers(self):
        return {
            "on_message_received": self.handle_message,
            "on_agent_start": self.handle_agent_start,
            "on_agent_end": self.handle_agent_end,
        }

    async def handle_message(self, message):
        """收到消息时调用"""
        pass

    async def handle_agent_start(self, context):
        """AI 开始处理时调用"""
        pass

    async def handle_agent_end(self, context, response):
        """AI 处理完成时调用"""
        pass
```

### 可用的事件

| 事件 | 触发时机 |
|------|----------|
| `on_startup` | OpenClaw 启动时 |
| `on_shutdown` | OpenClaw 关闭时 |
| `on_message_received` | 收到用户消息 |
| `on_message_sent` | 发送消息后 |
| `on_agent_start` | AI 开始处理 |
| `on_agent_end` | AI 处理完成 |
| `on_error` | 发生错误时 |
| `on_tool_call` | 调用工具时 |

---

## 本地测试插件

### 方式一：本地安装

```bash
# 进入插件目录
cd my_weather_plugin

# 本地安装
openclaw skill install .

# 查看是否安装成功
openclaw skill list | grep weather

# 测试
openclaw skill test my-weather

# 卸载
openclaw skill uninstall my-weather
```

### 方式二：开发模式

```bash
# 启动开发模式（会自动重载）
openclaw dev --plugin ./my_weather_plugin

# 查看日志
openclaw logs --follow
```

---

## 发布插件到 ClawHub

### 第一步：注册 ClawHub 账号

访问 https://clawhub.openclaw.ai 注册账号。

### 第二步：准备发布

```bash
# 确保版本号正确
# 修改 plugin.json 里的 version

# 打包
cd ..
tar -czf my-weather-1.0.0.tar.gz my_weather_plugin/

# 发布
openclaw skill publish --path ./my-weather-1.0.0.tar.gz
```

### 第三步：审核

提交后需要通过安全审核：
- VirusTotal 扫描
- 代码审查
- 功能测试

审核通过后，你的插件就会出现在 ClawHub 上！

---

## 完整示例：翻译插件

### 项目结构

```
my-translate/
├── plugin.json
├── __init__.py
└── tools/
    ├── __init__.py
    └── translate.py
```

### plugin.json

```json
{
  "name": "my-translate",
  "version": "1.0.0",
  "description": "翻译插件，支持多种语言互译",
  "author": "你",
  "config_schema": {
    "api_key": {
      "type": "string",
      "required": true,
      "description": "翻译 API 密钥"
    },
    "default_target": {
      "type": "string",
      "default": "zh",
      "description": "默认目标语言"
    }
  }
}
```

### translate.py

```python
from openclaw.tools import Tool, ToolResult
from pydantic import BaseModel, Field
import httpx

class TranslateInput(BaseModel):
    text: str = Field(description="要翻译的文本")
    source: str = Field(default="auto", description="源语言，auto 表示自动检测")
    target: str = Field(default="zh", description="目标语言代码，如 zh/en/ja")

class TranslateTool(Tool):
    name = "translate"
    description = "将文本从一种语言翻译到另一种语言"
    input_model = TranslateInput

    def __init__(self, api_key: str):
        self.api_key = api_key

    async def execute(self, input_data: TranslateInput) -> ToolResult:
        try:
            # 这里用免费的 LibreTranslate API
            async with httpx.AsyncClient() as client:
                response = await client.post(
                    "https://libretranslate.com/translate",
                    json={
                        "q": input_data.text,
                        "source": input_data.source,
                        "target": input_data.target,
                        "api_key": self.api_key
                    }
                )
                data = response.json()

            translated = data["translatedText"]

            return ToolResult(
                success=True,
                output=f"原文：{input_data.text}\n翻译：{translated}"
            )

        except Exception as e:
            return ToolResult(success=False, error=str(e))
```

### __init__.py

```python
from openclaw.plugins import Plugin
from .tools import TranslateTool
from typing import List

class Plugin(Plugin):
    name = "my-translate"
    version = "1.0.0"

    def __init__(self):
        self._translate_tool = None

    async def on_load(self) -> bool:
        api_key = self.context.config.get("api_key", "")
        self._translate_tool = TranslateTool(api_key=api_key)
        return True

    def get_tools(self) -> List:
        return [self._translate_tool] if self._translate_tool else []
```

---

## 常见问题

### Q: 插件装上了但不管用？

1. **检查日志**
```bash
docker logs openclaw 2>&1 | grep -i error
```

2. **检查配置**
```bash
openclaw skill config my-weather
```

3. **重启服务**
```bash
docker restart openclaw
```

### Q: 工具返回的数据 AI 看不懂？

确保返回的是**纯文本**，不要返回复杂对象：

```python
# ✅ 好例子
return ToolResult(success=True, output="北京今天晴天，22度")

# ❌ 坏例子
return ToolResult(success=True, output=str({"temp": 22, "weather": "晴"}))
```

### Q: 工具总是被 AI 忽略？

检查 description 是否描述清楚：

```python
# ✅ 好描述
description = "查询天气，返回温度、湿度、天气状况"

# ❌ 模糊描述
description = "查询信息"
```

### Q: 想让工具支持更多参数？

用 Pydantic 的 Field 添加更多参数和校验：

```python
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(..., description="搜索关键词")
    limit: int = Field(default=10, ge=1, le=100, description="结果数量")
    safe_search: bool = Field(default=True, description="是否安全搜索")
```

---

## 下一步

学会了插件开发？继续探索：

| 教程 | 内容 |
|------|------|
| [[ClawHub技能市场]] | 学习发布和分享插件 |
| [[OpenClaw架构解析]] | 了解插件怎么接入系统 |
| [[OpenClaw高级用法]] | 更多高级技巧 |

---

**文档状态**：插件开发指南  
**更新时间**：2026年4月
