---
date: 2026-04-24
---

# Python 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Python 3.13 新特性、现代项目配置、类型注解系统深度解析、异步编程模式、以及 Python 在 AI/ML 领域的核心应用场景。

---

## Python 概述与生态定位

### 为什么选择 Python

Python 是一门诞生于1991年的编程语言，由荷兰程序员 Guido van Rossum 创造。三十多年来，Python 从一个小众的脚本语言发展成为全球最受欢迎的编程语言之一。在2026年的各项编程语言排行榜中，Python 稳居榜首，这并非偶然，而是其设计哲学和生态系统共同作用的结果。

Python 的成功可以归结为三个核心支柱。首先是**简洁优雅的语法设计**。Python 遵循「用一种方法，最好是只有一种方法来做一件事」的设计原则，这使得 Python 代码具有极高的可读性。对于初学者而言，Python 的学习曲线相对平缓，几天之内就能写出有实际功能的程序；对于专业开发者而言，Python 的简洁性能够大幅提升开发效率，减少维护成本。其次是**庞大而成熟的生态系统**。PyPI（Python Package Index）目前托管着超过50万个第三方包，从 Web 框架到科学计算，从游戏开发到人工智能，几乎无所不包。无论你需要做什么，总能在 PyPI 上找到现成的解决方案。第三是**在人工智能时代的核心地位**。TensorFlow、PyTorch、Keras 等主流深度学习框架都以 Python 为首选接口，OpenAI、Anthropic、Google 等厂商的 Python SDK 更是调用大模型 API 的行业标准。

在 vibecoding（AI 辅助编程）时代，Python 保持着独特的地位。对于快速原型验证，Python 能够让你在最短时间内验证想法的可行性；对于数据处理，Pandas、Polars 等工具已经成为处理结构化数据的行业标准；对于 AI 应用开发，LangChain、LlamaIndex 等框架让构建 AI 应用变得前所未有的简单；对于 API 开发，FastAPI 是构建高性能后端服务的最佳选择之一。

### Python 3.13 新特性

Python 3.13 于2024年10月发布，这是 Python 语言历史上变化最大的版本之一。这个版本不仅带来了性能提升，还引入了一些革命性的新特性。

#### GIL 移除预览（实验性）

Python 3.13 最引人注目的变化是引入了实验性的「无 GIL（Global Interpreter Lock）构建」。GIL 是 Python 长期被诟病的一个设计缺陷，它限制了 Python 在多线程场景下的并行执行能力。无 GIL 构建允许真正的并行计算，这对于 CPU 密集型任务来说是一个重大利好。不过需要注意的是，这个特性目前仍处于实验阶段，生产环境建议使用 Python 3.12 或等待更稳定的版本。

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

#### 改进的错误消息

Python 3.13 提供了更清晰、更实用的错误信息。之前的版本在遇到语法错误时，错误消息往往不够直观；而在新版本中，Python 会提供更详细的上下文信息和修复建议。这对于初学者来说尤其有帮助，能够大大缩短调试时间。

#### 实时交互解释器改进

Python 3.13 的 REPL（Read-Eval-Print Loop）得到了显著改进。新版本支持多行编辑、自动缩进、语法高亮（部分终端支持），以及更详细的 traceback 信息。输入历史也得到了增强，支持使用 Alt+N/Alt+P 访问历史，Ctrl+R 进行增量搜索。

#### 类型注解增强

Python 3.13 引入了 `typing.TypeIs` 的改进版本，以及新的 `typing.ReadOnly` 注解。这些改进使得静态类型检查更加严格和精确。

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

# 新的 ReadOnly 注解
from typing import ReadOnly

class Config:
    # API_KEY 在初始化后不应被修改
    API_KEY: ReadOnly[str]
    
    def __init__(self, key: str):
        self.API_KEY = key
```

### 版本特性对比表

| 特性 | 3.10 | 3.11 | 3.12 | 3.13 |
|------|------|------|------|------|
| match-case 语句 | ✅ | ✅ | ✅ | ✅ |
| 协程改进 | - | ✅ | ✅ | ✅ |
| 异常组 (ExceptionGroup) | - | ✅ | ✅ | ✅ |
| 无 GIL 构建 | - | - | - | ✅ |
| 改进的错误信息 | - | - | ✅ | ✅ |
| typing.TypeIs | - | - | - | ✅ |
| typing.ReadOnly | - | - | - | ✅ |
| 性能提升 | - | 25% | 25% | 5-10% |

> [!TIP]
> 对于 AI/ML 项目，推荐使用 Python 3.11 或 3.12，这些版本对 PyTorch、TensorFlow 的支持最为完善。

---

## 现代 Python 项目配置

### uv：新一代包管理工具

在2026年，uv 已经成为 Python 项目管理的首选工具。uv 由 Astral 公司（ruff 开发者）推出，号称比 pip 快10-100倍。uv 的核心优势在于其极致的性能和确定性安装。

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

# 创建新项目（推荐方式）
uv init myproject
cd myproject
uv add fastapi uvicorn
```

uv 的高级用法包括使用 `uv run` 自动创建环境并运行脚本，使用 `uv python` 管理多个 Python 版本，以及使用 `uv pip compile` 生成锁定的依赖文件。这些功能使得 uv 成为现代 Python 项目管理的利器。

### pyproject.toml 配置

现代 Python 项目使用 pyproject.toml 作为配置中心。这个文件定义了项目的元数据、依赖、构建配置等所有相关内容。

```toml
[project]
name = "my-ai-project"
version = "0.1.0"
description = "An AI-powered application"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.32.0",
    "pydantic>=2.10.0",
    "langchain-openai>=0.3.0",
    "llama-index>=0.12.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.3.0",
    "pytest-asyncio>=0.25.0",
    "ruff>=0.8.0",
    "mypy>=1.13.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.ruff]
line-length = 100
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
ignore = ["E501"]

[tool.mypy]
python_version = "3.11"
strict = true

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

---

## 类型注解系统深度解析

### 基础类型注解

Python 的类型注解系统自 3.5 引入以来不断完善。Python 3.9 开始可以直接使用内置类型作为泛型参数，不再需要从 typing 导入。

```python
# Python 3.11+ 可以在更多位置使用类型注解

# 类变量和实例变量
class User:
    name: str  # 可以直接注解类变量
    age: int
    
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age

# 更简洁的泛型语法
from collections.abc import Callable

def process(items: list[str], func: Callable[[str], int]) -> dict[str, int]:
    return {item: func(item) for item in items}
```

### TypedDict 与类型安全

TypedDict 允许你定义结构化字典的类型注解，这在处理 JSON 数据时特别有用。

```python
from typing import TypedDict, NotRequired, Required

class User(TypedDict):
    name: Required[str]  # 必填字段
    email: str
    age: int
    bio: NotRequired[str]  # 可选字段

def create_user(data: User) -> str:
    return f"{data['name']} ({data['email']})"

# 正确使用
user: User = {
    "name": "Alice",
    "email": "alice@example.com",
    "age": 30
}

# mypy 会检查缺失的必填字段
# user2: User = {"name": "Bob"}  # Type checker error!
```

### Protocol 与结构化子类型

Protocol 定义了一种「协议」，只要一个类型实现了协议中定义的方法，就可以在需要该协议的地方使用。这是 Python 实现「鸭子类型」与静态类型检查结合的最佳方式。

```python
from typing import Protocol, TypeVar

class Drawable(Protocol):
    def draw(self) -> None: ...
    def move(self, dx: float, dy: float) -> None: ...

class Circle:
    def __init__(self, x: float, y: float, radius: float):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self) -> None:
        print(f"Drawing circle at ({self.x}, {self.y})")
    
    def move(self, dx: float, dy: float) -> None:
        self.x += dx
        self.y += dy

# Circle 实现了 Drawable 协议，可以在任何需要 Drawable 的地方使用
def render_all(drawables: list[Drawable]) -> None:
    for d in drawables:
        d.draw()

circle = Circle(0, 0, 5)
render_all([circle])  # 正常工作
```

### Generic 与泛型类

泛型允许你编写可以适用于多种类型的代码，同时保持类型安全。

```python
from typing import TypeVar, Generic

T = TypeVar('T')
K = TypeVar('K')
V = TypeVar('V')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []
    
    def push(self, item: T) -> None:
        self._items.append(item)
    
    def pop(self) -> T:
        if not self._items:
            raise IndexError("pop from empty stack")
        return self._items.pop()
    
    def peek(self) -> T:
        return self._items[-1]
    
    def is_empty(self) -> bool:
        return len(self._items) == 0

# 使用泛型栈
int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
print(int_stack.pop())  # 2

str_stack: Stack[str] = Stack()
str_stack.push("hello")
str_stack.push("world")
print(str_stack.pop())  # world
```

### @overload 装饰器

当函数可以返回不同类型时，使用 @overload 可以精确描述每种情况。

```python
from typing import overload

@overload
def process(data: int) -> int: ...
@overload
def process(data: str) -> list[str]: ...
@overload
def process(data: bytes) -> dict[str, int]: ...

def process(data):
    """实际实现"""
    if isinstance(data, int):
        return data * 2
    elif isinstance(data, str):
        return data.split()
    elif isinstance(data, bytes):
        return {"bytes": len(data)}
    raise TypeError(f"Unsupported type: {type(data)}")

# 类型检查器知道：
# process(5) -> int
# process("hello world") -> list[str]
# process(b"hello") -> dict[str, int]
```

---

## 异步编程 asyncio

### asyncio 核心概念

asyncio 是 Python 异步 I/O 的标准库，特别适合 I/O 密集型任务，如网络请求、文件操作、数据库查询等。理解 asyncio 的关键在于区分「并发」和「并行」：asyncio 提供的是并发而非并行，它通过事件循环在一个线程内切换执行多个协程。

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

# asyncio.wait_for: 带超时的异步操作
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

### TaskGroup 与结构化并发

Python 3.11 引入了 TaskGroup，这是结构化并发的实现。相比传统的 asyncio.gather，TaskGroup 提供了更好的错误处理和资源管理。

```python
import asyncio

async def fetch_user(user_id: int) -> dict:
    await asyncio.sleep(0.1)
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_users_concurrent(user_ids: list[int]) -> list[dict]:
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch_user(uid)) for uid in user_ids]
    
    # 所有任务完成后，tasks 中包含结果
    return [task.result() for task in tasks]

async def main():
    users = await fetch_users_concurrent([1, 2, 3, 4, 5])
    for user in users:
        print(user)

asyncio.run(main())
```

### 信号量与并发控制

信号量（Semaphore）用于限制同时运行的异步任务数量，这在处理资源有限的场景（如 API 调用限制、数据库连接池）时非常有用。

```python
import asyncio

async def bounded_api_call(
    semaphore: asyncio.Semaphore,
    api_name: str,
    call_id: int
) -> str:
    async with semaphore:
        print(f"{api_name} call {call_id} started")
        await asyncio.sleep(0.5)  # 模拟 API 调用
        return f"{api_name} response {call_id}"

async def main():
    # 限制同时只有 2 个 API 调用
    semaphore = asyncio.Semaphore(2)
    
    tasks = [
        bounded_api_call(semaphore, "API-A", i)
        for i in range(5)
    ]
    
    results = await asyncio.gather(*tasks)
    for result in results:
        print(result)

asyncio.run(main())
```

### 异步上下文管理器

异步上下文管理器用于管理需要初始化和清理资源的异步操作。

```python
import asyncio
from contextlib import asynccontextmanager

class AsyncDatabaseConnection:
    """异步数据库连接"""
    
    async def __aenter__(self):
        print("Connecting to database...")
        await asyncio.sleep(0.5)  # 模拟连接建立
        print("Connected!")
        self.connected = True
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Closing connection...")
        await asyncio.sleep(0.1)  # 模拟连接关闭
        print("Closed!")
    
    async def query(self, sql: str) -> list[dict]:
        if not self.connected:
            raise RuntimeError("Not connected")
        await asyncio.sleep(0.3)  # 模拟查询
        return [{"id": 1, "name": "Alice"}]

# 使用 async with
async def main():
    async with AsyncDatabaseConnection() as conn:
        result = await conn.query("SELECT * FROM users")
        print(result)

asyncio.run(main())

# 使用装饰器形式的异步上下文管理器
@asynccontextmanager
async def async_transaction(db_pool):
    conn = await db_pool.acquire()
    try:
        yield conn
        await conn.commit()
    except Exception:
        await conn.rollback()
        raise
    finally:
        await db_pool.release(conn)
```

---

## 上下文管理器与装饰器

### 上下文管理器深度解析

上下文管理器是 Python 中处理资源初始化和清理的最佳实践。无论代码是否抛出异常，with 块结束时的清理代码都会被执行。

```python
from contextlib import contextmanager
import sqlite3
import time

# 类形式的上下文管理器
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
        # 返回 False 表示不抑制异常，True 表示抑制
        return False

# 使用
with DatabaseConnection('app.db') as cursor:
    cursor.execute('SELECT * FROM users')
    users = cursor.fetchall()

# 生成器形式的上下文管理器
@contextmanager
def timer(label: str):
    """计时上下文管理器"""
    start = time.time()
    try:
        yield
    finally:
        elapsed = time.time() - start
        print(f"{label}: {elapsed:.2f}s")

# 使用
with timer("Data processing"):
    # 耗时操作
    time.sleep(1)
```

### 装饰器详解

装饰器是 Python 中修改函数或类行为的强大工具。理解装饰器需要掌握闭包的概念。

```python
import functools
import time
from typing import Callable, TypeVar

T = TypeVar('T')

# 带参数的装饰器
def retry(max_attempts: int, delay: float = 1.0):
    """重试装饰器：失败时自动重试"""
    def decorator(func: Callable[..., T]) -> Callable[..., T]:
        @functools.wraps(func)  # 保留原函数元数据
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

# 使用类作为装饰器（带状态）
class Cache:
    """缓存装饰器"""
    def __init__(self, ttl: int = 60):
        self.cache = {}
        self.ttl = ttl
    
    def __call__(self, func: Callable) -> Callable:
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # 创建缓存键
            key = (args, tuple(sorted(kwargs.items())))
            if key in self.cache:
                result, timestamp = self.cache[key]
                if time.time() - timestamp < self.ttl:
                    return result
            result = func(*args, **kwargs)
            self.cache[key] = (result, time.time())
            return result
        return wrapper

# 组合多个装饰器
@retry(max_attempts=3, delay=0.5)
@Cache(ttl=300)
def fetch_data(url: str) -> dict:
    """带重试和缓存的数据获取"""
    import requests
    response = requests.get(url)
    return response.json()
```

---

## 数据类与验证库对比

### dataclasses 基础

dataclasses 是 Python 3.7 引入的特性，用于简化数据容器的创建。

```python
from dataclasses import dataclass, field
from typing import Optional
from datetime import datetime

@dataclass
class User:
    name: str
    email: str
    age: int
    created_at: datetime = field(default_factory=datetime.now)
    tags: list[str] = field(default_factory=list)
    bio: Optional[str] = None
    
    def __post_init__(self):
        # 数据验证
        if self.age < 0:
            raise ValueError("Age cannot be negative")
        self.email = self.email.lower()

# 使用
user = User(name="Alice", email="Alice@Example.com", age=30)
print(user)
# User(name='Alice', email='alice@example.com', age=30, 
#       created_at=datetime(...), tags=[], bio=None)

# dataclasses 的优势
@dataclass
class Point:
    x: float
    y: float
    
    def distance_to(self, other: 'Point') -> float:
        return ((self.x - other.x)**2 + (self.y - other.y)**2) ** 0.5

p1 = Point(0, 0)
p2 = Point(3, 4)
print(p1.distance_to(p2))  # 5.0
```

### Pydantic 运行时验证

Pydantic 是 Python 中最流行的数据验证库，在 FastAPI 中被广泛使用。它不仅能验证数据，还能在验证过程中进行类型转换。

```python
from pydantic import BaseModel, Field, field_validator, model_validator
from datetime import datetime
from typing import Optional, List

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

class Order(BaseModel):
    items: List[dict]
    total: float
    
    @model_validator(mode='after')
    def check_total(self):
        calculated = sum(item.get('price', 0) * item.get('quantity', 0) 
                        for item in self.items)
        if abs(calculated - self.total) > 0.01:
            raise ValueError(f"Total mismatch: expected {calculated}, got {self.total}")
        return self

# 自动验证和类型转换
user_data = {
    "id": "1",  # 字符串会被自动转换
    "username": "alice",
    "email": "Alice@Example.com",
    "age": 25
}
user = User.model_validate(user_data)
print(user.email)  # "alice@example.com" - 自动转为小写
```

### Dataclasses vs Pydantic vs Attrs

| 特性 | dataclasses | Pydantic | attrs |
|------|-------------|----------|-------|
| 运行时验证 | ❌ | ✅ | ⚠️ (需额外配置) |
| 类型转换 | ❌ | ✅ | ❌ |
| JSON 序列化 | ❌ | ✅ | ⚠️ |
| 依赖重量 | 无 | 中等 | 轻量 |
| 适用场景 | 内部数据结构 | API 请求/响应 | 内部数据结构 |

---

## FastAPI 深度教程

### 从模型到部署的完整流程

FastAPI 是一个现代、快速的 Python Web 框架，基于 Starlette 和 Pydantic。它的核心优势在于自动 API 文档、高性能、类型安全和异步支持。

```python
from fastapi import FastAPI, Depends, HTTPException, status, BackgroundTasks
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field, EmailStr, field_validator
from typing import Optional, AsyncIterator
from contextlib import asynccontextmanager
import asyncio
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# 生命周期管理
@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用启动和关闭时的资源管理"""
    logger.info("Application starting up...")
    
    # 初始化数据库连接
    # await init_database()
    
    yield
    
    # 清理资源
    logger.info("Application shutting down...")
    # await close_database()

app = FastAPI(
    title="AI Service API",
    version="1.0.0",
    lifespan=lifespan
)

# CORS 配置
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Pydantic 请求/响应模型
class ChatRequest(BaseModel):
    """聊天请求"""
    model: str = Field(default="gpt-4o", description="模型名称")
    messages: list[dict] = Field(..., description="消息历史")
    temperature: float = Field(default=0.7, ge=0, le=2)
    max_tokens: Optional[int] = Field(default=None, ge=1, le=32000)
    stream: bool = Field(default=False, description="是否使用流式响应")

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

# 认证依赖
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
    start_time = time.time()
    
    # 模拟 AI 调用
    await asyncio.sleep(0.5)
    
    return ChatResponse(
        id=f"chatcmpl-{int(time.time())}",
        content=f"Response to your message about: {request.messages[-1]['content'][:50]}...",
        model=request.model,
        usage={
            "prompt_tokens": 100,
            "completion_tokens": 50,
            "total_tokens": 150
        }
    )

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

# 自定义中间件
@app.middleware("http")
async def log_requests(request, call_next):
    start_time = time.time()
    
    logger.info(f"Request: {request.method} {request.url.path}")
    
    response = await call_next(request)
    
    process_time = time.time() - start_time
    logger.info(f"Response: {response.status_code} ({process_time:.3f}s)")
    
    response.headers["X-Process-Time"] = str(process_time)
    return response
```

### 流式响应实现

流式响应是 AI 应用的常见需求，FastAPI 提供了简洁的流式响应支持。

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
import asyncio
import json

app = FastAPI()

class StreamRequest(BaseModel):
    prompt: str
    model: str = "gpt-4o"

async def generate_stream(prompt: str) -> AsyncIterator[str]:
    """模拟流式生成"""
    # 模拟分词生成
    words = prompt.split()
    for i, word in enumerate(words):
        chunk = {
            "id": f"chatcmpl-stream",
            "choices": [{
                "delta": {"content": word + " "},
                "index": 0,
                "finish_reason": None if i < len(words) - 1 else "stop"
            }]
        }
        yield f"data: {json.dumps(chunk)}\n\n"
        await asyncio.sleep(0.05)  # 模拟生成延迟
    
    yield f"data: [DONE]\n\n"

@app.post("/chat/stream")
async def chat_stream(request: StreamRequest):
    return StreamingResponse(
        generate_stream(request.prompt),
        media_type="text/event-stream"
    )
```

---

## Python 在 AI/LLM 中的应用

### LangChain 集成

LangChain 是构建 LLM 应用的流行框架，提供了链式调用、记忆管理、工具调用等核心功能。

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain.chains import LLMChain
from langchain_community.vectorstores import Chroma
from langchain_community.embeddings import OpenAIEmbeddings

# 初始化模型
llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    api_key="your-api-key"
)

# 创建提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一位专业的{subject}顾问，用简洁专业的语言回答问题。"),
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

LlamaIndex 专注于知识检索增强生成（RAG），让你能够基于私有数据构建问答系统。

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

# 带过滤的查询
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core.retrievers import VectorIndexRetriever

retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=3
)
query_engine = RetrieverQueryEngine.from_args(retriever)
```

---

## 测试实践

### pytest 高级用法

pytest 是 Python 最流行的测试框架，提供了丰富的测试功能。

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
```

---

## JavaScript/TypeScript 开发者常见陷阱

对于来自 JavaScript/TypeScript 背景的开发者，Python 有一些容易踩坑的地方。

### 缩进与作用域

Python 使用缩进来定义代码块，而不是花括号。这是最容易出错的地方。

```python
# JavaScript: 使用花括号
if (true) {
    let x = 1;
    if (true) {
        let y = 2;
    }
    console.log(y); // 仍然可以访问 y
}

# Python: 使用缩进
if True:
    x = 1
    if True:
        y = 2
    print(y)  # 正常，可以访问 y
```

### 列表操作的区别

Python 的列表是 mutable 的，切片操作会创建新的列表副本，而引用则共享同一个列表。

```python
# JavaScript: spread 操作符创建副本
const arr1 = [1, 2, 3];
const arr2 = [...arr1];
arr2.push(4);
console.log(arr1); // [1, 2, 3] - arr1 不变

# Python: 切片创建副本
arr1 = [1, 2, 3]
arr2 = arr1[:]  # 创建副本
arr2.append(4)
print(arr1)  # [1, 2, 3] - arr1 不变

# 但是直接赋值是引用
arr1 = [1, 2, 3]
arr2 = arr1  # 引用
arr2.append(4)
print(arr1)  # [1, 2, 3, 4] - arr1 也变了！
```

### None vs undefined/null

Python 只有 None，而 JavaScript 有 undefined 和 null。判断 None 应该使用 `is` 而不是 `==`。

```python
# Python: 使用 is 判断 None
x = None
if x is None:
    print("x is None")

# 错误：虽然可能工作，但不推荐
if x == None:
    print("x is None")

# JavaScript: 需要区分 undefined 和 null
let x;
console.log(x === undefined);  // true
let y = null;
console.log(y === null);  // true
```

---

## Python vs TypeScript 对比表

| 特性 | Python | TypeScript |
|------|--------|------------|
| 类型系统 | 运行时验证 + 可选静态 | 编译时强制 |
| 泛型 | Python 3.12+ | 原生支持 |
| 异步 | asyncio | async/await |
| 空值处理 | None | undefined/null |
| 装饰器 | 原生支持 | 需要配置 |
| 包管理 | pip/uv | npm/pnpm |
| 运行环境 | CPython | Node.js |

---

## 选型建议

### 工具选择对照表

| 需求场景 | 推荐方案 | 说明 |
|----------|----------|------|
| 快速原型 | uv + FastAPI | 零配置，快速启动 |
| 数据处理 | Polars (非 Pandas) | 性能最优 |
| 机器学习 | PyTorch + transformers | 主流选择 |
| AI 应用 | LangChain/LlamaIndex | 生态完善 |
| API 开发 | FastAPI | 异步、高性能、自动文档 |
| 类型安全 | Pydantic v2 | 运行时验证 |
| 环境管理 | uv | 速度最优 |

### 版本推荐

| 场景 | 推荐版本 | 理由 |
|------|----------|------|
| 生产环境 | Python 3.12 | 稳定、性能优秀 |
| AI/ML 项目 | Python 3.11+ | 对 PyTorch 支持最佳 |
| 学习入门 | Python 3.12 | 语法简洁、文档丰富 |
| 实验项目 | Python 3.13 | 体验最新特性 |

---

## 参考资料

| 资源 | 链接 |
|------|------|
| Python 官方文档 | https://docs.python.org/ |
| Real Python | https://realpython.com/ |
| Python Cheatsheet | https://pythoncheatsheet.org/ |
| Awesome Python | https://awesome-python.com/ |
| Typing 文档 | https://docs.python.org/3/library/typing.html |
| FastAPI 文档 | https://fastapi.tiangolo.com/ |
| Pydantic 文档 | https://docs.pydantic.dev/ |

---

> [!TIP]
> 对于 AI 应用开发，推荐使用 `uv` 作为包管理器，配合 `FastAPI` + `Pydantic` + `LangChain` 的技术栈。

---

> [!SUCCESS]
> 本文档全面介绍了 Python 的核心特性、3.13 新功能、类型注解系统、异步编程以及在 AI/ML 领域的应用。Python 凭借其简洁的语法、丰富的生态和强大的 AI 库支持，在 vibecoding 时代继续保持着不可替代的地位。
