# Python 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Python 3.13 新特性、虚拟环境管理、类型注解系统、异步编程，以及 Python 在 AI/ML 领域的核心应用。

---

## 目录

1. [[#Python 概述与生态定位]]
2. [[#Python 3.13 新特性]]
3. [[#版本特性对比]]
4. [[#虚拟环境管理]]
5. [[#类型注解系统]]
6. [[#异步编程 asyncio]]
7. [[#FastAPI 与现代 Web 开发]]
8. [[#数据科学生态]]
9. [[#Python 在 AI/ML 中的角色]]
10. [[#实战场景与代码示例]]
11. [[#选型建议]]

---

## Python 概述与生态定位

### 为什么选择 Python

Python 诞生于 1991 年，由 Guido van Rossum 创造。经过三十余年的发展，Python 已成为数据科学、机器学习、Web 开发、自动化运维等领域的首选语言。在 2026 年的 TIOBE 编程语言排行榜中，Python 稳居第一。

Python 的核心优势体现在三个维度：

**简洁优雅**：Python 的语法设计哲学是「用一种方法，最好是只有一种方法来做一件事」。代码可读性极高，学习曲线平缓。

**生态丰富**：PyPI（Python Package Index）托管了超过 50 万个第三方包，从 Web 框架到科学计算，从 AI 模型到游戏开发，几乎无所不包。

**AI 时代的主角**：TensorFlow、PyTorch、Keras 等主流深度学习框架均以 Python 为首选接口。OpenAI、Anthropic、Google 等厂商的 Python SDK 是调用大模型 API 的标准工具。

### Python 在 AI 编程中的不可替代性

在 vibecoding（AI 辅助编程）时代，Python 保持着独特的地位：

- **原型验证**：用 Python 快速验证 AI 应用的可行性
- **数据处理**：Pandas、Polars 等工具处理结构化数据的首选
- **模型集成**：LangChain、LlamaIndex 等框架以 Python 为核心
- **API 开发**：FastAPI 是构建 AI 后端服务的最佳选择之一

---

## Python 3.13 新特性

Python 3.13 于 2024 年 10 月发布，是迄今为止变化最大的 Python 版本之一。

### 核心新特性

#### 1. GIL 移除预览（实验性）

Python 3.13 引入了实验性的「无 GIL（Global Interpreter Lock）构建」，这是 Python 历史上最重大的变革之一：

```python
# 需要使用专门的构建：python3.13t（experimental free-threaded build）
# 运行方式：python3.13t your_script.py

# 在无 GIL 构建中，以下代码可以真正并行执行
import threading

def compute(data):
    result = sum(x * x for x in data)
    return result

# 真正的并行计算
data1 = range(1_000_000)
data2 = range(1_000_000, 2_000_000)

with threading.Thread(target=lambda: print(compute(data1))) as t1:
    with threading.Thread(target=lambda: print(compute(data2))) as t2:
        t1.start()
        t2.start()
        t1.join()
        t2.join()
```

> [!IMPORTANT]
> 无 GIL 构建目前处于实验阶段，生产环境建议使用 Python 3.12 或等待 3.15 稳定版。

#### 2. 改进的错误消息

Python 3.13 提供了更清晰、更实用的错误信息：

```python
# Python 3.12
def greet(name):
    return f"Hello, {name"

greet("World")
# File "example.py", line 2
#     return f"Hello, {name"
#                         ~^^^
# SyntaxError: f-string: unmatched '('

# Python 3.13
# 错误信息更详细，指出具体问题和修复建议
```

#### 3. 实时交互解释器（REPL）改进

```python
# Python 3.13 REPL 新特性

# 多行编辑支持
# 自动缩进
# 语法高亮（部分终端）
# 更详细的 traceback

# 输入历史增强
# 使用 Alt+N/Alt+P 访问历史
# 增量搜索 (Ctrl+R)
```

#### 4. 类型注解增强

```python
# PEP 749: 改进的 typing.TypeIs
from typing import TypeIs

def is_str_list(val: list[object]) -> TypeIs[list[str]]:
    return all(isinstance(x, str) for x in val)

# 使用方式
items: list[object] = ["a", "b", "c"]
if is_str_list(items):
    # Python 3.13 正确推断 items 类型为 list[str]
    print(items[0].upper())  # 不再需要额外断言
```

#### 5. 新的 `typing.ReadOnly` 注解

```python
from typing import ReadOnly

# 标记只读类型
class Config:
    # API_KEY 在初始化后不应被修改
    API_KEY: ReadOnly[str]
    
    def __init__(self, key: str):
        self.API_KEY = key

# mypy 和 IDE 会警告修改只读字段
config = Config("secret")
# config.API_KEY = "new"  # 类型检查器会报错
```

### Python 3.13 其他改进

```python
# 1. 性能提升：JIT 编译器改进，启动速度提升约 10%
# 2. 更快的导入：优化了模块查找和加载
# 3. 改进的 hash 随机化：安全性提升
# 4. 更好的异步调试：asyncio.run() 提供更详细的错误信息
```

---

## 版本特性对比

### Python 3.x 版本特性对比表

| 特性 | 3.8 | 3.9 | 3.10 | 3.11 | 3.12 | 3.13 |
|------|-----|-----|------|------|------|------|
| **walrus 运算符 (:=)** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **类型标注语法优化** | - | ✅ | ✅ | ✅ | ✅ | ✅ |
| **match-case 语句** | - | - | ✅ | ✅ | ✅ | ✅ |
| **结构化模式匹配** | - | - | ✅ | ✅ | ✅ | ✅ |
| **协程改进** | - | - | - | ✅ | ✅ | ✅ |
| **异常组 (ExceptionGroup)** | - | - | - | ✅ | ✅ | ✅ |
| **无 GIL 构建** | - | - | - | - | - | ✅ |
| **改进的错误信息** | - | - | - | - | ✅ | ✅ |
| **f-string 调试** | - | - | - | ✅ | ✅ | ✅ |
| **typing.TypeIs** | - | - | - | - | - | ✅ |
| **typing.ReadOnly** | - | - | - | - | - | ✅ |
| **性能提升** | - | - | - | 25% | 25% | 5-10% |

### 版本选择建议

| 场景 | 推荐版本 | 原因 |
|------|----------|------|
| **生产环境（保守）** | Python 3.11/3.12 | 稳定、性能优秀 |
| **AI/ML 应用** | Python 3.11+ | 对 PyTorch、TensorFlow 支持最佳 |
| **新项目尝试** | Python 3.13 | 体验最新特性 |
| **需要无 GIL** | Python 3.13 (experimental) | 实验性特性 |
| **依赖库限制** | 根据库要求 | 查看库的 pyproject.toml |

---

## 虚拟环境管理

### 工具对比表

| 工具 | 速度 | 功能 | 依赖解析 | 适用场景 |
|------|------|------|----------|----------|
| **uv** | 极快 (10-100x pip) | 完整 | 确定性 | 新项目首选 |
| **pip + venv** | 慢 | 基础 | 基础 | 简单项目 |
| **Poetry** | 中等 | 丰富 | 较好 | 中型项目 |
| **PDM** | 快 | 完整 | PEP 517 | 追求 PEP 规范 |
| **Conda** | 慢 | 生态 | 复杂 | 数据科学 |

### uv：新一代 Python 包管理

uv 是 Astral 公司（ruff 开发者）推出的新一代 Python 包管理工具，号称比 pip 快 10-100 倍：

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或者通过 pip 安装
pip install uv

# 创建虚拟环境
uv venv myproject

# 激活虚拟环境
source myproject/bin/activate

# 安装依赖（极速）
uv pip install fastapi uvicorn pandas polars

# 从 requirements.txt 安装
uv pip install -r requirements.txt

# 同步依赖
uv pip sync requirements.txt

# 创建项目（推荐）
uv init myproject
cd myproject
uv add fastapi uvicorn
```

### uv 的高级用法

```bash
# 使用 uv 运行脚本（自动创建环境）
uv run python main.py

# 使用 uv 管理多个 Python 版本
uv python list
uv python pin 3.12

# 创建特定版本的环境
uv venv --python 3.11 myproject

# 锁定依赖版本
uv pip freeze > requirements.txt
uv pip compile requirements.in -o requirements.txt
```

### 传统 venv 用法

```bash
# 创建虚拟环境
python -m venv myproject

# 激活
source myproject/bin/activate  # Linux/macOS
myproject\Scripts\activate      # Windows

# 安装依赖
pip install fastapi uvicorn

# 导出依赖
pip freeze > requirements.txt

# 退出
deactivate
```

### Poetry 用法

```bash
# 安装 Poetry
pip install poetry

# 初始化项目
poetry init
poetry add fastapi uvicorn

# 安装依赖
poetry install

# 运行脚本
poetry run python main.py

# 锁定版本
poetry lock
```

---

## 类型注解系统

### Python 3.11+ 类型注解增强

Python 3.11 引入了更简洁的类型注解语法：

```python
# Python 3.11+ 可以在更多位置使用类型注解

# 类变量和实例变量
class User:
    name: str  # 可以直接注解类变量
    age: int
    
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

# self 参数的类型注解（使用 self 类型）
class MyClass:
    def method(self, x: int) -> str:
        return str(x)

# 更简洁的泛型语法
from collections.abc import Callable

# 旧语法
def process(items: list[str], func: Callable[[str], int]) -> dict[str, int]:
    return {item: func(item) for item in items}

# 注意：Python 3.9+ 可直接使用内置类型作为泛型
# list[str] 替代 typing.List[str]
# dict[str, int] 替代 typing.Dict[str, int]
```

### Pydantic 运行时验证

Pydantic 是 Python 中最流行的数据验证库，与类型注解完美结合：

```python
from pydantic import BaseModel, Field, field_validator
from datetime import datetime
from typing import Optional

class User(BaseModel):
    """用户模型：注册时自动验证数据"""
    
    id: int
    username: str = Field(..., min_length=3, max_length=50)
    email: str
    age: Optional[int] = Field(default=None, ge=0, le=150)
    created_at: datetime = Field(default_factory=datetime.now)
    
    @field_validator("email")
    @classmethod
    def validate_email(cls, v: str) -> str:
        if "@" not in v:
            raise ValueError("Invalid email address")
        return v.lower()
    
    @field_validator("username")
    @classmethod
    def validate_username(cls, v: str) -> str:
        if not v.isalnum():
            raise ValueError("Username must be alphanumeric")
        return v

# 自动验证
try:
    user = User(
        id=1,
        username="alice",
        email="Alice@example.com",
        age=25
    )
    print(user.email)  # "alice@example.com" - 自动转为小写
except Exception as e:
    print(f"Validation error: {e}")

# 从字典创建（API 响应常用）
data = {
    "id": "1",  # 字符串会被自动转换
    "username": "alice",
    "email": "alice@example.com"
}
user = User.model_validate(data)
```

### 类型守卫与 TypeIs

```python
from typing import TypeIs

def is_str_list(val: list[object]) -> TypeIs[list[str]]:
    """判断列表是否全是字符串"""
    return all(isinstance(x, str) for x in val)

def process_items(items: list[object]) -> None:
    if is_str_list(items):
        # Python 3.13+ 正确推断 items 为 list[str]
        # 可以直接调用 str 方法
        upper_items = [item.upper() for item in items]
        print(upper_items)
    else:
        print("Not all items are strings")

# TypeGuard vs TypeIs
# TypeGuard: 缩小条件分支中的类型
# TypeIs: 同时缩小 if 和 else 分支的类型
```

---

## 异步编程 asyncio

### asyncio 核心概念

asyncio 是 Python 异步 I/O 的标准库，适合 I/O 密集型任务：

```python
import asyncio
from typing import Optional

# 基础协程
async def fetch_data(url: str) -> dict:
    """模拟异步 I/O 操作"""
    await asyncio.sleep(1)  # 模拟网络请求
    return {"url": url, "data": "some data"}

async def main():
    # 并发执行多个协程
    urls = [
        "https://api.example.com/1",
        "https://api.example.com/2",
        "https://api.example.com/3"
    ]
    
    # gather: 并发执行，返回结果列表
    results = await asyncio.gather(
        *[fetch_data(url) for url in urls]
    )
    
    for result in results:
        print(result)

# 运行
asyncio.run(main())

# asyncio.wait: 更灵活，可配合 return_when 参数
async def with_timeout():
    """带超时的异步操作"""
    try:
        result = await asyncio.wait_for(
            fetch_data("https://example.com"),
            timeout=5.0
        )
        return result
    except asyncio.TimeoutError:
        return None
```

### 异步上下文管理器

```python
import asyncio
from contextlib import asynccontextmanager

class AsyncDatabaseConnection:
    """异步数据库连接"""
    
    async def __aenter__(self):
        # 建立连接
        print("Connecting to database...")
        await asyncio.sleep(0.5)
        print("Connected!")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        # 关闭连接
        print("Closing connection...")
        await asyncio.sleep(0.1)
        print("Closed!")
    
    async def query(self, sql: str) -> list[dict]:
        await asyncio.sleep(0.3)  # 模拟查询
        return [{"id": 1, "name": "Alice"}]

# 使用 async with
async def main():
    async with AsyncDatabaseConnection() as conn:
        result = await conn.query("SELECT * FROM users")
        print(result)

asyncio.run(main())
```

### 异步迭代器与生成器

```python
import asyncio
from typing import AsyncIterator

async def async_data_generator(n: int) -> AsyncIterator[int]:
    """异步数据生成器"""
    for i in range(n):
        await asyncio.sleep(0.1)  # 模拟数据获取延迟
        yield i

async def consume_async_stream():
    """消费异步数据流"""
    async for item in async_data_generator(10):
        print(f"Received: {item}")

# 异步列表推导
async def async_comprehension():
    # 异步生成器表达式
    result = [x async for x in async_data_generator(10)]
    print(result)
    
    # 带条件的异步推导
    evens = [x async for x in async_data_generator(10) if x % 2 == 0]
    print(evens)

asyncio.run(consume_async_stream())
```

### asyncio 与 FastAPI 的结合

```python
from fastapi import FastAPI, HTTPException
import asyncio

app = FastAPI()

# 模拟数据库操作
async def get_user_from_db(user_id: int) -> dict | None:
    await asyncio.sleep(0.1)
    if user_id == 1:
        return {"id": 1, "name": "Alice", "email": "alice@example.com"}
    return None

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    """异步获取用户信息"""
    user = await get_user_from_db(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

@app.get("/users")
async def list_users():
    """并发获取多个用户"""
    # 模拟多个用户 ID
    user_ids = [1, 2, 3, 4, 5]
    
    # 并发执行
    tasks = [get_user_from_db(uid) for uid in user_ids]
    results = await asyncio.gather(*tasks)
    
    # 过滤 None 值
    return [r for r in results if r is not None]
```

---

## FastAPI 与现代 Web 开发

### FastAPI 核心特性

FastAPI 是一个现代、快速的 Python Web 框架，基于 Starlette 和 Pydantic：

```python
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from pydantic import BaseModel, Field, EmailStr
from typing import Optional
import asyncio

app = FastAPI(title="AI Service API", version="1.0.0")

# Pydantic 请求/响应模型
class ChatRequest(BaseModel):
    """聊天请求"""
    model: str = Field(default="gpt-4o", description="模型名称")
    messages: list[dict] = Field(..., description="消息历史")
    temperature: float = Field(default=0.7, ge=0, le=2)
    max_tokens: Optional[int] = Field(default=None, ge=1, le=32000)

class ChatResponse(BaseModel):
    """聊天响应"""
    id: str
    content: str
    model: str
    usage: dict

class ErrorResponse(BaseModel):
    """错误响应"""
    detail: str
    error_code: Optional[str] = None

# 依赖注入：认证
security = HTTPBearer()

async def verify_token(credentials: HTTPAuthorizationCredentials = Depends(security)):
    """验证 API Token"""
    if credentials.scheme != "Bearer":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication scheme"
        )
    
    token = credentials.credentials
    # 实际应该验证 token
    if not token.startswith("sk-"):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token"
        )
    
    return {"token": token}

# API 端点
@app.post("/chat", response_model=ChatResponse)
async def chat(
    request: ChatRequest,
    auth: dict = Depends(verify_token)
):
    """处理聊天请求"""
    try:
        # 模拟 AI 调用
        await asyncio.sleep(0.5)
        
        return ChatResponse(
            id="chatcmpl-123",
            content=f"Response to your message about: {request.messages[-1]['content'][:50]}...",
            model=request.model,
            usage={
                "prompt_tokens": 100,
                "completion_tokens": 50,
                "total_tokens": 150
            }
        )
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/models")
async def list_models(auth: dict = Depends(verify_token)):
    """列出可用模型"""
    return {
        "models": [
            {"id": "gpt-4o", "name": "GPT-4o", "context_length": 128000},
            {"id": "gpt-4o-mini", "name": "GPT-4o Mini", "context_length": 128000},
            {"id": "claude-3.5-sonnet", "name": "Claude 3.5 Sonnet", "context_length": 200000}
        ]
    }
```

### 中间件与事件处理

```python
from fastapi import FastAPI, Request, Response
from fastapi.middleware.cors import CORSMiddleware
from contextlib import asynccontextmanager
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期管理"""
    logger.info("Application starting up...")
    
    # 初始化资源
    # await init_database()
    # await init_redis()
    
    yield
    
    # 清理资源
    logger.info("Application shutting down...")
    # await close_database()
    # await close_redis()

app = FastAPI(lifespan=lifespan)

# CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 自定义中间件：请求日志
@app.middleware("http")
async def log_requests(request: Request, call_next):
    start_time = time.time()
    
    logger.info(f"Request: {request.method} {request.url.path}")
    
    response = await call_next(request)
    
    process_time = time.time() - start_time
    logger.info(f"Response: {response.status_code} ({process_time:.3f}s)")
    
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

---

## 数据科学生态

### NumPy 核心用法

```python
import numpy as np

# 数组创建
arr1 = np.array([1, 2, 3, 4, 5])
arr2 = np.zeros((3, 4))        # 全零矩阵
arr3 = np.random.rand(2, 3)    # 随机矩阵
arr4 = np.linspace(0, 10, 100)  # 线性分布

# 向量化操作
result = arr1 * 2 + 1  # [3, 5, 7, 9, 11]

# 矩阵运算
matrix = np.random.rand(3, 3)
eigenvalues, eigenvectors = np.linalg.eig(matrix)

# 统计函数
data = np.random.randn(1000)
mean = np.mean(data)
std = np.std(data)
percentile = np.percentile(data, 95)
```

### Pandas 数据处理

```python
import pandas as pd

# 创建 DataFrame
df = pd.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35],
    "score": [85.5, 90.2, 78.3]
})

# 数据筛选
adults = df[df["age"] >= 30]

# 聚合操作
grouped = df.groupby("age").agg({
    "score": ["mean", "max", "min"]
})

# 处理缺失值
df_cleaned = df.dropna()  # 删除缺失行
df_filled = df.fillna(0)  # 填充默认值

# 合并数据
df1 = pd.DataFrame({"id": [1, 2], "name": ["Alice", "Bob"]})
df2 = pd.DataFrame({"id": [1, 2], "score": [90, 85]})
merged = pd.merge(df1, df2, on="id")
```

### Polars：更快的选择

```python
import polars as pl

# Polars 以速度著称，比 Pandas 快 10-50 倍
df = pl.DataFrame({
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35],
    "score": [85.5, 90.2, 78.3]
})

# Lazy API：延迟执行，更优化
result = (
    df.lazy()
    .filter(pl.col("age") >= 30)
    .select(["name", "score"])
    .sort("score", descending=True)
    .collect()
)

# 使用表达式
df = df.with_columns([
    (pl.col("score") / 100 * 100).alias("percentage")
])
```

---

## Python 在 AI/ML 中的角色

### AI 应用架构

```
┌─────────────────────────────────────────────────────────────┐
│                      AI 应用架构                              │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: 用户接口层                                          │
│  ├── Streamlit / Gradio (快速原型)                            │
│  ├── FastAPI + React (生产环境)                               │
│  └── Discord/Slack Bots                                      │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: 编排层                                              │
│  ├── LangChain / LangGraph (流程编排)                         │
│  ├── LlamaIndex (知识检索)                                    │
│  └── CrewAI / AutoGen (多代理协作)                            │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: 模型层                                              │
│  ├── OpenAI / Anthropic API                                  │
│  ├── 本地模型 (Ollama, vLLM)                                  │
│  └── 开源模型 (Hugging Face)                                 │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: 数据层                                              │
│  ├── PostgreSQL + pgvector (向量存储)                        │
│  ├── Redis (缓存)                                            │
│  └── S3 (文件存储)                                           │
└─────────────────────────────────────────────────────────────┘
```

### LangChain 示例

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain.chains import LLMChain

# 初始化模型
llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    api_key="your-api-key"
)

# 创建提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一位专业的{subject}顾问。"),
    ("user", "{question}")
])

# 创建链
chain = prompt | llm | StrOutputParser()

# 执行
result = chain.invoke({
    "subject": "技术写作",
    "question": "如何写好技术文档？"
})
print(result)
```

### LlamaIndex 知识检索

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader
from llama_index.llms.openai import OpenAI

# 加载文档
documents = SimpleDirectoryReader("./data").load_data()

# 创建索引
llm = OpenAI(model="gpt-4o")
index = VectorStoreIndex.from_documents(
    documents,
    llm=llm
)

# 创建查询引擎
query_engine = index.as_query_engine(
    similarity_top_k=3,
    response_mode="compact"
)

# 查询
response = query_engine.query("文档的主要内容是什么？")
print(response)
```

---

## 实战场景与代码示例

### 场景一：构建 AI 对话服务

```python
"""
AI 对话服务完整实现
包含：流式响应、Token 计数、错误处理、历史记录
"""

from fastapi import FastAPI, HTTPException, Request
from fastapi.responses import StreamingResponse
from pydantic import BaseModel, Field
from typing import Optional, AsyncIterator
import asyncio
import json
import time

app = FastAPI(title="AI Chat Service")

class Message(BaseModel):
    role: str = Field(..., pattern="^(system|user|assistant)$")
    content: str

class ChatRequest(BaseModel):
    model: str = "gpt-4o"
    messages: list[Message]
    temperature: float = Field(default=0.7, ge=0, le=2)
    stream: bool = False
    max_tokens: Optional[int] = Field(default=None, ge=1, le=32000)

class ChatResponse(BaseModel):
    id: str
    model: str
    content: str
    usage: dict

# 模拟的对话历史存储
conversation_history: dict[str, list[Message]] = {}

@app.post("/chat")
async def chat(request: ChatRequest) -> ChatResponse:
    """处理非流式聊天请求"""
    start_time = time.time()
    
    # 验证模型
    valid_models = ["gpt-4o", "gpt-4o-mini", "claude-3.5-sonnet"]
    if request.model not in valid_models:
        raise HTTPException(
            status_code=400,
            detail=f"Invalid model. Choose from: {valid_models}"
        )
    
    # 计算 Token
    total_tokens = sum(
        len(msg.content.split()) * 1.3  # 粗略估算
        for msg in request.messages
    )
    
    # 模拟 AI 响应
    await asyncio.sleep(0.5)
    
    last_message = request.messages[-1].content
    response_content = f"Echo: {last_message[:100]}..."
    
    return ChatResponse(
        id=f"chatcmpl-{int(time.time())}",
        model=request.model,
        content=response_content,
        usage={
            "prompt_tokens": int(total_tokens),
            "completion_tokens": int(len(response_content.split()) * 1.3),
            "total_tokens": int(total_tokens + len(response_content.split()) * 1.3)
        }
    )

@app.post("/chat/stream")
async def chat_stream(request: ChatRequest):
    """处理流式聊天请求"""
    async def generate() -> AsyncIterator[str]:
        last_message = request.messages[-1].content
        
        # 逐词生成响应
        words = f"Echo: {last_message}".split()
        for i, word in enumerate(words):
            chunk = {
                "id": f"chatcmpl-{int(time.time())}",
                "choices": [{
                    "delta": {"content": word + " "},
                    "index": 0,
                    "finish_reason": None
                }]
            }
            yield f"data: {json.dumps(chunk)}\n\n"
            await asyncio.sleep(0.05)  # 模拟打字延迟
        
        # 发送结束信号
        yield f"data: [DONE]\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )

@app.get("/health")
async def health_check():
    """健康检查"""
    return {"status": "healthy", "timestamp": time.time()}
```

### 场景二：AI 与数据库集成

```python
"""
AI + PostgreSQL + pgvector 完整示例
实现：语义搜索、RAG（检索增强生成）
"""

from typing import Optional
import asyncio

# 模拟的数据库操作
class VectorDatabase:
    """向量数据库操作类"""
    
    def __init__(self):
        self.embeddings = []  # 存储嵌入向量
        self.documents = []   # 存储文档内容
        self.metadata = []    # 存储元数据
    
    async def add_document(
        self,
        content: str,
        embedding: list[float],
        metadata: dict
    ):
        """添加文档到向量数据库"""
        self.documents.append(content)
        self.embeddings.append(embedding)
        self.metadata.append(metadata)
    
    async def similarity_search(
        self,
        query_embedding: list[float],
        top_k: int = 5,
        filter_metadata: Optional[dict] = None
    ) -> list[dict]:
        """向量相似度搜索（简化版）"""
        if not self.embeddings:
            return []
        
        # 计算余弦相似度
        scores = []
        for i, emb in enumerate(self.embeddings):
            # 简化计算：点积
            score = sum(a * b for a, b in zip(query_embedding, emb))
            
            # 应用元数据过滤
            if filter_metadata:
                if not all(
                    self.metadata[i].get(k) == v
                    for k, v in filter_metadata.items()
                ):
                    continue
            
            scores.append((i, score))
        
        # 排序并返回 top_k
        scores.sort(key=lambda x: x[1], reverse=True)
        
        return [
            {
                "content": self.documents[idx],
                "score": score,
                "metadata": self.metadata[idx]
            }
            for idx, score in scores[:top_k]
        ]

async def rag_pipeline(
    query: str,
    db: VectorDatabase,
    llm: Any  # 模拟 LLM
) -> str:
    """RAG 流程：检索 + 生成"""
    
    # Step 1: 将查询转为向量
    query_embedding = await embed_text(query)
    
    # Step 2: 检索相关文档
    results = await db.similarity_search(
        query_embedding,
        top_k=5,
        filter_metadata={"source": "knowledge_base"}
    )
    
    # Step 3: 构建上下文
    context = "\n\n".join([
        f"[文档 {i+1}]: {r['content']}"
        for i, r in enumerate(results)
    ])
    
    # Step 4: 发送给 LLM 生成答案
    prompt = f"""
基于以下参考文档回答问题。如果文档中没有相关信息，请如实说明。

参考文档：
{context}

问题：{query}

回答：
"""
    
    response = await llm.generate(prompt)
    return response
```

---

## 选型建议

### 工具选择对照表

| 需求场景 | 推荐方案 | 说明 |
|----------|----------|------|
| **快速原型** | uv + FastAPI | 零配置，快速启动 |
| **数据处理** | Polars (非 Pandas) | 性能最优 |
| **机器学习** | PyTorch + transformers | 主流选择 |
| **AI 应用** | LangChain/LlamaIndex | 生态完善 |
| **API 开发** | FastAPI | 异步、高性能、自动文档 |
| **类型安全** | Pydantic v2 | 运行时验证 |
| **环境管理** | uv | 速度最优 |

### 版本推荐

| 场景 | 推荐版本 | 理由 |
|------|----------|------|
| **生产环境** | Python 3.12 | 稳定、性能优秀 |
| **AI/ML 项目** | Python 3.11+ | 对 PyTorch 支持最佳 |
| **学习入门** | Python 3.12 | 语法简洁、文档丰富 |
| **实验项目** | Python 3.13 | 体验最新特性 |

---

## Python 高级主题

### 装饰器深入

```python
import functools
import time
from typing import Callable, TypeVar

T = TypeVar('T')

# 带参数的装饰器
def retry(max_attempts: int, delay: float = 1.0):
    """重试装饰器"""
    def decorator(func: Callable[..., T]) -> Callable[..., T]:
        @functools.wraps(func)
        def wrapper(*args, **kwargs) -> T:
            last_exception = None
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    last_exception = e
                    if attempt < max_attempts - 1:
                        time.sleep(delay * (attempt + 1))
            raise last_exception
        return wrapper
    return decorator

# 使用类作为装饰器
class Cache:
    """缓存装饰器"""
    def __init__(self, ttl: int = 60):
        self.cache = {}
        self.ttl = ttl
    
    def __call__(self, func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            key = (args, tuple(sorted(kwargs.items())))
            if key in self.cache:
                result, timestamp = self.cache[key]
                if time.time() - timestamp < self.ttl:
                    return result
            result = func(*args, **kwargs)
            self.cache[key] = (result, time.time())
            return result
        return wrapper

@retry(max_attempts=3, delay=0.5)
def fetch_data(url: str) -> dict:
    """带重试的数据获取"""
    import requests
    response = requests.get(url)
    return response.json()
```

### 上下文管理器

```python
from contextlib import contextmanager
import sqlite3

# 类形式
class DatabaseConnection:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.conn = None
    
    def __enter__(self):
        self.conn = sqlite3.connect(self.db_path)
        return self.conn.cursor()
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.conn:
            if exc_type is None:
                self.conn.commit()
            else:
                self.conn.rollback()
            self.conn.close()

# 生成器形式
@contextmanager
def transaction(conn):
    cursor = conn.cursor()
    try:
        yield cursor
        conn.commit()
    except Exception:
        conn.rollback()
        raise

# 使用
with DatabaseConnection('app.db') as cursor:
    cursor.execute('SELECT * FROM users')
    users = cursor.fetchall()
```

### 元编程

```python
# 动态创建类
def create_model_class(table_name: str, fields: dict):
    """动态创建数据模型类"""
    return type(table_name, (), {
        '__tablename__': table_name,
        **{
            field_name: property(
                lambda self, _field=field_name: self._data.get(_field),
                lambda self, value, _field=field_name: self._data.update({_field: value})
            )
            for field_name in fields
        }
    })

# 类装饰器
def add_repr(cls):
    """自动添加 __repr__"""
    def __repr__(self):
        attrs = ', '.join(f'{k}={v!r}' for k, v in self.__dict__.items() if not k.startswith('_'))
        return f'{self.__class__.__name__}({attrs})'
    cls.__repr__ = __repr__
    return cls

@add_repr
class User:
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email

# 描述符
class Validator:
    """字段验证描述符"""
    def __init__(self, field_type: type):
        self.field_type = field_type
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get(self.name)
    
    def __set__(self, obj, value):
        if not isinstance(value, self.field_type):
            raise TypeError(f'{self.name} must be {self.field_type.__name__}')
        obj.__dict__[self.name] = value

class Person:
    name = Validator(str)
    age = Validator(int)
```

### 类型注解进阶

```python
from typing import TypeVar, Generic, Protocol, Callable, Awaitable

T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

# 泛型类
class Stack(Generic[T]):
    def __init__(self):
        self._items: list[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        if not self._items:
            raise IndexError("pop from empty stack")
        return self._items.pop()
    
    def peek(self) -> T:
        return self._items[-1]

# Protocol 定义结构化类型
class Drawable(Protocol):
    def draw(self) -> None: ...
    def move(self, dx: float, dy: float) -> None: ...

def render_all(drawables: list[Drawable]) -> None:
    for d in drawables:
        d.draw()

# Callable 类型
Operation = Callable[[int, int], int]
def apply_operation(a: int, b: int, op: Operation) -> int:
    return op(a, b)
```

---

## Python 数据库高级操作

### SQLAlchemy 核心用法

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import declarative_base, relationship, sessionmaker
from datetime import datetime

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    posts = relationship('Post', back_populates='author', cascade='all, delete-orphan')

class Post(Base):
    __tablename__ = 'posts'
    
    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(String, nullable=False)
    user_id = Column(Integer, ForeignKey('users.id'), nullable=False)
    
    author = relationship('User', back_populates='posts')

# 创建引擎和会话
engine = create_engine('sqlite:///app.db')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)

# CRUD 操作
def create_user(name: str, email: str) -> User:
    with Session() as session:
        user = User(name=name, email=email)
        session.add(user)
        session.commit()
        session.refresh(user)
        return user

def get_user_posts(user_id: int) -> list[Post]:
    with Session() as session:
        user = session.query(User).get(user_id)
        return user.posts if user else []
```

### SQLAlchemy 进阶查询

```python
from sqlalchemy import and_, or_, func

# 复杂查询
def advanced_search(name: str | None = None, min_posts: int = 0):
    with Session() as session:
        query = session.query(User)
        
        if name:
            query = query.filter(User.name.ilike(f'%{name}%'))
        
        if min_posts > 0:
            query = query.join(Post).group_by(User.id).having(
                func.count(Post.id) >= min_posts
            )
        
        return query.all()

# 批量操作
def bulk_create_posts(user_id: int, titles: list[str]):
    with Session() as session:
        posts = [
            Post(title=title, content='', user_id=user_id)
            for title in titles
        ]
        session.bulk_save_objects(posts)
        session.commit()
```

### Async SQLAlchemy

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

engine = create_async_engine('sqlite+aiosqlite:///app.db')
AsyncSessionLocal = async_sessionmaker(engine, class_=AsyncSession)

async def async_get_user(user_id: int) -> User | None:
    async with AsyncSessionLocal() as session:
        result = await session.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalar_one_or_none()

async def async_create_user(name: str, email: str) -> User:
    async with AsyncSessionLocal() as session:
        user = User(name=name, email=email)
        session.add(user)
        await session.commit()
        await session.refresh(user)
        return user
```

---

## Python 设计模式

### 创建型模式

```python
# 单例模式
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    def __init__(self):
        if not self._initialized:
            self._initialized = True
            self.value = None

# 工厂模式
class Document:
    def __init__(self, content: str):
        self.content = content

class DocumentFactory:
    @staticmethod
    def create_markdown(content: str) -> Document:
        return Document(f'# {content}')
    
    @staticmethod
    def create_html(content: str) -> Document:
        return Document(f'<h1>{content}</h1>')
    
    @staticmethod
    def create_plain(content: str) -> Document:
        return Document(content)
```

### 结构型模式

```python
# 装饰器模式
from functools import wraps

def log_calls(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f'Calling {func.__name__}')
        result = func(*args, **kwargs)
        print(f'{func.__name__} returned {result}')
        return result
    return wrapper

class DataProcessor:
    @log_calls
    def process(self, data):
        return data.upper()

# 适配器模式
class LegacyAPI:
    def get_data_v1(self) -> dict:
        return {'legacy_key': 'legacy_value'}

class Adapter:
    def __init__(self, legacy: LegacyAPI):
        self.legacy = legacy
    
    def get_data(self) -> dict:
        raw = self.legacy.get_data_v1()
        return {'modern_key': raw['legacy_key']}
```

### 行为型模式

```python
# 观察者模式
class Observable:
    def __init__(self):
        self._observers: list[callable] = []
    
    def add_observer(self, observer: callable):
        self._observers.append(observer)
    
    def notify(self, *args, **kwargs):
        for observer in self._observers:
            observer(*args, **kwargs)

class NewsAgency(Observable):
    def __init__(self):
        super().__init__()
        self._news = []
    
    def publish(self, news: str):
        self._news.append(news)
        self.notify(news)

# 使用
agency = NewsAgency()
agency.add_observer(lambda news: print(f'Alert: {news}'))
agency.publish('Breaking news!')
```

---

## Python 异步编程进阶

### 异步上下文管理器

```python
import asyncio
from contextlib import asynccontextmanager

@asynccontextmanager
async def async_database_session(dsn: str):
    conn = await connect(dsn)
    try:
        yield conn
        await conn.commit()
    except Exception:
        await conn.rollback()
        raise
    finally:
        await conn.close()

async def main():
    async with async_database_session('postgresql://...') as conn:
        result = await conn.execute('SELECT * FROM users')
        return await result.fetchall()
```

### 异步生成器

```python
async def fetch_urls(urls: list[str], batch_size: int = 10):
    """批量异步获取 URL"""
    for i in range(0, len(urls), batch_size):
        batch = urls[i:i + batch_size]
        tasks = [fetch(url) for url in batch]
        results = await asyncio.gather(*tasks, return_exceptions=True)
        for result in results:
            if isinstance(result, Exception):
                yield {'error': str(result)}
            else:
                yield result

async def main():
    async for item in fetch_urls(['http://a.com', 'http://b.com']):
        print(item)
```

### 信号量与并发控制

```python
import asyncio

async def bounded_task(semaphore: asyncio.Semaphore, task_id: int):
    async with semaphore:
        print(f'Task {task_id} started')
        await asyncio.sleep(1)
        print(f'Task {task_id} completed')
        return task_id

async def main():
    # 限制同时运行的任务数
    semaphore = asyncio.Semaphore(3)
    tasks = [bounded_task(semaphore, i) for i in range(10)]
    results = await asyncio.gather(*tasks)
    print(f'Completed: {results}')
```

---

## Python 性能优化

### cProfile 性能分析

```python
import cProfile
import pstats
from io import StringIO

def profile_function(func, *args, **kwargs):
    profiler = cProfile.Profile()
    profiler.enable()
    
    result = func(*args, **kwargs)
    
    profiler.disable()
    
    stream = StringIO()
    stats = pstats.Stats(profiler, stream=stream)
    stats.sort_stats('cumulative')
    stats.print_stats(20)
    
    print(stream.getvalue())
    return result

# 使用装饰器分析
def profile(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return profile_function(func, *args, **kwargs)
    return wrapper
```

### numba 加速

```python
from numba import jit
import numpy as np

@jit(nopython=True)
def matrix_multiply(a: np.ndarray, b: np.ndarray) -> np.ndarray:
    n = a.shape[0]
    m = b.shape[1]
    k = a.shape[1]
    result = np.zeros((n, m))
    
    for i in range(n):
        for j in range(m):
            for l in range(k):
                result[i, j] += a[i, l] * b[l, j]
    
    return result

# 加速效果显著
a = np.random.rand(500, 500)
b = np.random.rand(500, 500)
c = matrix_multiply(a, b)
```

### 内存优化

```python
import sys
from typing import List

# 使用 __slots__ 减少内存
class OptimizedPoint:
    __slots__ = ['x', 'y', 'z']
    
    def __init__(self, x: float, y: float, z: float):
        self.x = x
        self.y = y
        self.z = z

# 生成器替代列表
def range_sum(n: int) -> int:
    """使用生成器求和避免创建大列表"""
    return sum(i * i for i in range(n))

# 使用 array 模块
from array import array
def create_large_array():
    # 使用 array 节省内存
    arr = array('i', range(1000000))
    return arr
```

---

## Python AI 与机器学习

### PyTorch 基础

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# 定义神经网络
class SimpleNet(nn.Module):
    def __init__(self, input_size, hidden_size, num_classes):
        super().__init__()
        self.layer1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.layer2 = nn.Linear(hidden_size, num_classes)
    
    def forward(self, x):
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        return x

# 训练循环
def train(model, train_loader, criterion, optimizer, epochs=10):
    model.train()
    for epoch in range(epochs):
        total_loss = 0
        for X, y in train_loader:
            optimizer.zero_grad()
            outputs = model(X)
            loss = criterion(outputs, y)
            loss.backward()
            optimizer.step()
            total_loss += loss.item()
        print(f'Epoch {epoch+1}, Loss: {total_loss/len(train_loader):.4f}')

# 使用示例
X = torch.randn(1000, 20)
y = torch.randint(0, 5, (1000,))
dataset = TensorDataset(X, y)
train_loader = DataLoader(dataset, batch_size=32, shuffle=True)

model = SimpleNet(20, 50, 5)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

train(model, train_loader, criterion, optimizer)
```

### Transformers 库使用

```python
from transformers import pipeline, AutoTokenizer, AutoModelForSequenceClassification
import torch

# 情感分析
classifier = pipeline('sentiment-analysis')
result = classifier('I love using Python for AI projects!')
print(result)  # [{'label': 'POSITIVE', 'score': 0.9998}]

# 问答系统
qa_pipeline = pipeline('question-answering')
context = """
Python is a high-level programming language known for its 
readability and versatility. It supports multiple programming 
paradigms including procedural, object-oriented, and functional 
programming.
"""
question = "What is Python known for?"
result = qa_pipeline(question=question, context=context)
print(result['answer'])

# 自定义模型加载
model_name = 'distilbert-base-uncased-finetuned-emotion'
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)
```

### LangChain 集成

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import PromptTemplate
from langchain.chains import LLMChain

llm = ChatOpenAI(
    model='gpt-4',
    temperature=0.7,
    api_key='your-api-key'
)

prompt = PromptTemplate(
    input_variables=['topic'],
    template='请用 100 字介绍 {topic}，用中文回答。'
)

chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run('Python 编程语言')
print(result)
```

### LlamaIndex 使用

```python
from llama_index import SimpleDirectoryReader, VectorStoreIndex

# 加载文档
documents = SimpleDirectoryReader('./data').load_data()

# 构建索引
index = VectorStoreIndex.from_documents(documents)

# 创建查询引擎
query_engine = index.as_query_engine()
response = query_engine.query('文档的主要内容是什么？')
print(response)

# 带过滤的查询
from llama_index.query_engine import RetrieverQueryEngine
from llama_index.retrievers import VectorIndexRetriever

retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=3
)
query_engine = RetrieverQueryEngine.from_args(retriever)
```

---

## Python Web 开发

### FastAPI 进阶

```python
from fastapi import FastAPI, Depends, HTTPException, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, EmailStr, field_validator
from typing import Optional
import asyncio

app = FastAPI(title='Advanced API', version='2.0')

# 添加 CORS 中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=['https://example.com'],
    allow_credentials=True,
    allow_methods=['*'],
    allow_headers=['*'],
)

class UserCreate(BaseModel):
    name: str
    email: EmailStr
    age: int
    
    @field_validator('age')
    @classmethod
    def validate_age(cls, v):
        if v < 0 or v > 150:
            raise ValueError('Age must be between 0 and 150')
        return v

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    age: int

# 模拟数据库
users_db = {}
next_id = 1

@app.post('/users/', response_model=UserResponse, status_code=201)
async def create_user(user: UserCreate):
    global next_id
    user_dict = user.model_dump()
    user_dict['id'] = next_id
    users_db[next_id] = user_dict
    next_id += 1
    return user_dict

@app.get('/users/{user_id}', response_model=UserResponse)
async def get_user(user_id: int):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail='User not found')
    return users_db[user_id]

def send_email(email: str):
    # 模拟发送邮件
    print(f'Sending email to {email}')

@app.post('/users/{user_id}/welcome')
async def send_welcome_email(user_id: int, background_tasks: BackgroundTasks):
    if user_id not in users_db:
        raise HTTPException(status_code=404, detail='User not found')
    email = users_db[user_id]['email']
    background_tasks.add_task(send_email, email)
    return {'message': 'Welcome email scheduled'}
```

### 中间件与依赖注入

```python
from fastapi import Depends, Request
from functools import wraps
import time

# 自定义中间件
@app.middleware('http')
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers['X-Process-Time'] = str(process_time)
    return response

# 依赖注入
async def verify_token(token: str = Depends(lambda: 'mock-token')):
    if token != 'valid-token':
        raise HTTPException(status_code=401, detail='Invalid token')
    return token

@app.get('/protected')
async def protected_route(token: str = Depends(verify_token)):
    return {'message': 'Access granted', 'token': token}

# 可选依赖
async def get_optional_user(
    authorization: Optional[str] = None
) -> Optional[dict]:
    if authorization:
        return {'user': 'authenticated'}
    return None
```

---

## Python 测试

### pytest 高级用法

```python
import pytest
from unittest.mock import Mock, patch, AsyncMock
import asyncio

# 参数化测试
@pytest.mark.parametrize('input,expected', [
    ('hello', 'HELLO'),
    ('world', 'WORLD'),
    ('Python', 'PYTHON'),
])
def test_uppercase(input, expected):
    assert input.upper() == expected

# 异步测试
@pytest.mark.asyncio
async def test_async_function():
    result = await asyncio.sleep(0.1, 'done')
    assert result == 'done'

# Mock 和 Patch
@patch('requests.get')
def test_api_call(mock_get):
    mock_response = Mock()
    mock_response.json.return_value = {'data': 'test'}
    mock_get.return_value = mock_response
    
    import requests
    response = requests.get('https://api.example.com')
    assert response.json() == {'data': 'test'}

# Fixture 工厂模式
@pytest.fixture
def user_factory():
    def create_user(name='Test', email='test@example.com'):
        return {'name': name, 'email': email}
    return create_user

def test_user_factory(user_factory):
    user = user_factory()
    assert user['name'] == 'Test'
    assert '@' in user['email']

# 跳过测试
@pytest.mark.skip(reason='Not implemented yet')
def test_future_feature():
    pass

# 预期失败
@pytest.mark.xfail(reason='Known bug')
def test_known_bug():
    assert False
```

### 测试覆盖率

```bash
# 安装 pytest-cov
pip install pytest-cov

# 运行测试并生成覆盖率报告
pytest --cov=src --cov-report=html tests/

# 查看详细报告
pytest --cov=src --cov-report=term-missing tests/
```

---

## Python 部署与容器化

### Docker 配置

```dockerfile
# Dockerfile
FROM python:3.12-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 运行
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/mydb
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Gunicorn + Uvicorn

```bash
# 启动多个 worker
gunicorn main:app \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \
    -b 0.0.0.0:8000
```

---

## 参考资料

| 资源 | 链接 |
|------|------|
| Python 官方文档 | https://docs.python.org/ |
| Real Python | https://realpython.com/ |
| Python Cheatsheet | https://pythoncheatsheet.org/ |
| Awesome Python | https://awesome-python.com/ |
| Typing 文档 | https://docs.python.org/3/library/typing.html |

---

> [!TIP]
> 对于 AI 应用开发，推荐使用 `uv` 作为包管理器，配合 `FastAPI` + `Pydantic` + `LangChain` 的技术栈。

---

> [!SUCCESS]
> 本文档全面介绍了 Python 的核心特性、3.13 新功能、类型注解系统、异步编程以及在 AI/ML 领域的应用。Python 凭借其简洁的语法、丰富的生态和强大的 AI 库支持，在 vibecoding 时代继续保持着不可替代的地位。
