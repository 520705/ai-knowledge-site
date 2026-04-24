# FastAPI 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 FastAPI 核心原理、Pydantic 数据验证、AI 集成及企业级应用实践。

---

## 目录

1. [[#FastAPI 概述与定位]]
2. [[#核心原理：ASGI 与 Starlette]]
3. [[#Pydantic 数据验证体系]]
4. [[#依赖注入系统]]
5. [[#异步编程与性能优化]]
6. [[#路由与中间件]]
7. [[#后台任务与 WebSocket]]
8. [[#FastAPI 与 AI 框架集成]]
9. [[#FastAPI vs Flask vs Django 对比]]
10. [[#实战场景与选型建议]]
11. [[#参考资料]]

---

## FastAPI 概述与定位

FastAPI 是一个现代、快速的 Python Web 框架，于 2019 年由 Sebastián Ramírez 发布。与传统 Python Web 框架相比，FastAPI 的核心优势在于：

- **类型安全**：基于 Python 类型注解（type hints）实现自动数据验证
- **性能卓越**：基于 Starlette（ASGI）异步架构，接近 Node.js 和 Go 的性能
- **自动文档**：自动生成 OpenAPI/Swagger/ReDoc 文档
- **AI 友好**：与 LangChain、LlamaIndex 等 AI 框架天然集成
- **现代设计**：依赖注入、后台任务、WebSocket 等企业级特性开箱即用

### 核心定位

| 维度 | 定位 |
|------|------|
| **目标用户** | 需要高性能 API 的全栈开发者、AI 应用开发者 |
| **应用场景** | RESTful API、微服务、AI 服务端点、数据管道 |
| **性能等级** | Tiers 1（接近 Node.js/Go） |
| **学习曲线** | 低（Python 开发者 1-2 天可上手） |
| **生态系统** | 快速成长，与 AI 框架深度集成 |

> [!TIP]
> FastAPI 在 2026 年已成为 AI 应用后端的首选框架，LangChain、LlamaIndex、AutoGen 等主流框架的官方示例均基于 FastAPI。

---

## 核心原理：ASGI 与 Starlette

### ASGI 协议

ASGI（Asynchronous Server Gateway Interface）是 WSGI 的异步继承者，允许处理长连接（WebSocket）、Server-Sent Events（SSE）和 HTTP/2。ASGI 应用程序是一个异步可调用对象：

```python
# ASGI 应用程序签名
async def app(scope: Scope, receive: Receive, send: Send) -> None:
    """
    scope: 包含连接信息的字典（如 'type', 'path', 'headers', 'query_string'）
    receive: 接收客户端消息的协程
    send: 发送响应给客户端的协程
    """
    pass
```

### Starlette 架构

FastAPI 构建于 Starlette 之上，Starlette 提供了 ASGI 应用程序的基础构建块：

| 组件 | 功能 | 说明 |
|------|------|------|
| **Application** | 根对象 | 管理中间件、路由、事件 |
| **Router** | 路由系统 | 路径匹配与请求分发 |
| **Request/Response** | 请求/响应对象 | 封装 ASGI 消息 |
| **HTTPExceptions** | HTTP 异常 | 统一的错误处理 |
| **StaticFiles** | 静态文件服务 | 内置静态资源支持 |
| **Templates** | 模板引擎 | Jinja2 集成 |
| **WebSocket** | WebSocket 支持 | 双向通信 |

### 请求处理流程

```
客户端请求
    ↓
ASGI Server（如 Uvicorn）接收连接
    ↓
创建 Scope（连接上下文）
    ↓
依次通过中间件链
    ↓
Router 匹配路径到对应路径操作函数
    ↓
依赖注入解析
    ↓
路径操作函数执行
    ↓
Pydantic 验证请求/响应数据
    ↓
返回 Response
    ↓
中间件逆向处理
    ↓
客户端收到响应
```

---

## Pydantic 数据验证体系

Pydantic 是 FastAPI 数据验证的核心，其基于 Python 类型注解的验证机制革新了数据处理方式。

### Pydantic 模型基础

```python
from pydantic import BaseModel, Field, validator
from typing import Optional, List
from datetime import datetime

class User(BaseModel):
    """用户数据模型示例"""
    
    # 基础字段验证
    user_id: int = Field(..., gt=0, description="用户ID，必须大于0")
    username: str = Field(..., min_length=3, max_length=50)
    email: str
    age: Optional[int] = Field(None, ge=0, le=150)  # 0-150 岁
    
    # 列表字段验证
    tags: List[str] = Field(default_factory=list, max_items=10)
    
    # 嵌套模型
    profile: Optional["UserProfile"] = None
    
    # 自定义验证器
    @validator("email")
    def validate_email_format(cls, v: str) -> str:
        """验证邮箱格式"""
        if "@" not in v or "." not in v.split("@")[-1]:
            raise ValueError("邮箱格式不正确")
        return v.lower()  # 自动转换为小写
    
    @validator("username")
    def validate_username(cls, v: str) -> str:
        """验证用户名只包含字母数字和下划线"""
        if not v.replace("_", "").isalnum():
            raise ValueError("用户名只能包含字母、数字和下划线")
        return v

class UserProfile(BaseModel):
    """用户资料嵌套模型"""
    bio: Optional[str] = Field(None, max_length=500)
    avatar_url: Optional[str] = None
    created_at: datetime = Field(default_factory=datetime.now)
```

### 请求体验证

```python
from fastapi import FastAPI, Body
from pydantic import BaseModel

app = FastAPI()

class Item(BaseModel):
    """商品数据模型"""
    name: str = Field(..., min_length=1, max_length=200)
    price: float = Field(..., gt=0)  # 价格必须大于 0
    description: Optional[str] = None
    tax: Optional[float] = None

@app.post("/items/")
async def create_item(item: Item):
    """
    FastAPI 自动验证请求体：
    1. 解析 JSON
    2. 类型转换
    3. 字段验证
    4. 返回 422 错误或执行业务逻辑
    """
    # item 已经过完整验证
    item_dict = item.model_dump()
    if item.tax:
        price_with_tax = item.price * (1 + item.tax)
        item_dict["price_with_tax"] = price_with_tax
    return {"message": "创建成功", "data": item_dict}
```

### 查询参数验证

```python
from fastapi import Query, Path, Header

@app.get("/users/")
async def get_users(
    # 分页参数，带默认值和范围限制
    skip: int = Query(0, ge=0, description="跳过的记录数"),
    limit: int = Query(10, ge=1, le=100, description="返回的记录数"),
    # 搜索和过滤
    search: Optional[str] = Query(None, min_length=2, max_length=50),
    status: Optional[str] = Query(None, regex="^(active|inactive|pending)$"),
    # 排序
    sort_by: str = Query("created_at", enum=["name", "created_at", "price"]),
    order: str = Query("desc", enum=["asc", "desc"])
):
    """带完整验证的查询参数"""
    return {
        "skip": skip,
        "limit": limit,
        "search": search,
        "status": status,
        "sort_by": sort_by,
        "order": order
    }

@app.get("/files/{file_path:path}")
async def get_file(
    file_path: str = Path(..., description="文件路径"),
    if_none_match: Optional[str] = Header(None, alias="If-None-Match")
):
    """路径参数验证"""
    return {"file_path": file_path, "etag": if_none_match}
```

### Pydantic vs Marshmallow vs Serializer 对比

| 特性 | Pydantic | Marshmallow | Django REST Framework Serializer |
|------|----------|-------------|----------------------------------|
| **验证方式** | 类型注解 | 装饰器/Schema | 类定义 |
| **运行时性能** | 极高（Cython 优化） | 高 | 中等 |
| **JSON Schema 生成** | 自动 | 需额外库 | 有限 |
| **数据转换** | 自动 | 需手动定义 | 自动 |
| **可选字段** | `Optional[T]` | `missing=` | `required=False` |
| **嵌套验证** | 原生支持 | Schema 嵌套 | Serializer 嵌套 |
| **异步支持** | 原生 | 有限 | 有限 |
| **序列化** | `.dict()` / `.model_dump()` | `.dump()` | `.data` |
| **反序列化** | `.parse_obj()` / `.model_validate()` | `.load()` | `.validated_data` |
| **生态整合** | FastAPI 原生 | SQLAlchemy-ecosystem | Django 原生 |

> [!IMPORTANT]
> Pydantic v2（2023年发布）在性能上有质的飞跃，比 v1 快 50 倍以上。推荐使用 `model_validate()` 和 `model_dump()` 方法替代旧的 `parse_obj()` 和 `dict()` 方法。

---

## 依赖注入系统

FastAPI 的依赖注入系统是其最强大的特性之一，允许声明式地管理共享逻辑、数据库连接、认证等。

### 基础依赖

```python
from fastapi import Depends, HTTPException, status
from typing import Optional, Generator
import asyncio

# 简单依赖
def get_query_param(q: Optional[str] = None):
    """简单查询参数依赖"""
    return q or "default"

@app.get("/items/")
async def read_items(q: str = Depends(get_query_param)):
    """使用 Depends() 注入依赖"""
    return {"query": q}

# 类依赖
class CommonQueryParams:
    """可复用的查询参数依赖类"""
    def __init__(self, q: str, skip: int = 0, limit: int = 10):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/users/")
async def read_users(params: CommonQueryParams = Depends()):
    """类依赖使用"""
    return {
        "q": params.q,
        "skip": params.skip,
        "limit": params.limit
    }
```

### 异步依赖

```python
from databases import Database
import asyncpg

# 模拟数据库连接池
database_url = "postgresql://user:pass@localhost/db"

async def get_db() -> Generator:
    """
    异步数据库依赖：
    1. 获取连接
    2. 传递给路径操作
    3. 请求结束后自动归还/关闭连接
    """
    db = Database(database_url)
    await db.connect()
    try:
        yield db
    finally:
        await db.disconnect()

@app.get("/posts/")
async def get_posts(db: Database = Depends(get_db)):
    """使用数据库依赖"""
    query = "SELECT * FROM posts LIMIT :limit OFFSET :skip"
    rows = await db.fetch_all(query, {"limit": 10, "skip": 0})
    return [dict(row) for row in rows]

# 异步依赖函数
async def fetch_user_data(user_id: int) -> dict:
    """模拟异步获取用户数据"""
    await asyncio.sleep(0.1)  # 模拟 IO 操作
    return {
        "id": user_id,
        "name": "张三",
        "email": "zhangsan@example.com"
    }

@app.get("/users/{user_id}")
async def get_user(user_data: dict = Depends(fetch_user_data)):
    """异步依赖注入"""
    return user_data
```

### 带参数的依赖

```python
from fastapi import Depends

# 可配置的依赖工厂
def create_retry_dependency(max_retries: int = 3, delay: float = 1.0):
    """带参数的依赖工厂"""
    async def retry_logic():
        for attempt in range(max_retries):
            try:
                # 执行业务逻辑
                result = await perform_operation()
                return result
            except Exception as e:
                if attempt == max_retries - 1:
                    raise e
                await asyncio.sleep(delay * (attempt + 1))
    
    return retry_logic

# 在路径操作中使用
@app.get("/data/")
async def fetch_data(
    data: dict = Depends(create_retry_dependency(max_retries=5, delay=0.5))
):
    return data

# 子依赖
def verify_api_key(x_api_key: str = Header(...)):
    """验证 API Key 依赖"""
    if x_api_key != "secret-key":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="无效的 API Key"
        )
    return x_api_key

def get_current_user(api_key: str = Depends(verify_api_key)):
    """依赖链：先验证 API Key，再获取用户"""
    user = find_user_by_api_key(api_key)
    return user

@app.get("/protected/")
async def protected_endpoint(user: dict = Depends(get_current_user)):
    """受保护的端点"""
    return {"message": f"欢迎 {user['name']}"}
```

### 依赖覆盖（测试）

```python
from fastapi import FastAPI
from unittest.mock import MagicMock

app = FastAPI()

# 业务逻辑依赖
async def get_database():
    """真实的数据库连接"""
    db = Database(database_url)
    await db.connect()
    try:
        yield db
    finally:
        await db.disconnect()

# 测试时覆盖依赖
@pytest.fixture
def override_dependencies():
    """测试用的依赖覆盖"""
    mock_db = MagicMock()
    mock_db.fetch_all = AsyncMock(return_value=[
        {"id": 1, "name": "测试用户"}
    ])
    
    app.dependency_overrides[get_database] = mock_db
    yield
    app.dependency_overrides.clear()

def test_get_users(override_dependencies):
    """测试受保护的端点"""
    response = client.get("/users/")
    assert response.status_code == 200
```

---

## 异步编程与性能优化

### async/await 基础

```python
import asyncio
from fastapi import FastAPI

app = FastAPI()

@app.get("/sync")
async def sync_endpoint():
    """同步端点（实际异步执行）"""
    return {"message": "这是一个同步端点"}

@app.get("/async")
async def async_endpoint():
    """异步端点"""
    # 模拟 IO 密集型操作
    await asyncio.sleep(1)
    return {"message": "异步处理完成"}

@app.get("/parallel")
async def parallel_endpoint():
    """并行执行多个异步任务"""
    # 同时发起多个请求
    task1 = fetch_user_data(1)
    task2 = fetch_user_data(2)
    task3 = fetch_user_data(3)
    
    # 等待所有任务完成（并行）
    results = await asyncio.gather(task1, task2, task3)
    
    return {"users": results}
```

### 并发请求处理

```python
import aiohttp
from asyncio import Semaphore

@app.get("/batch")
async def batch_request(user_ids: str):
    """
    批量请求处理，支持并发控制和超时
    user_ids: 逗号分隔的用户ID，如 "1,2,3"
    """
    ids = [int(id) for id in user_ids.split(",")]
    
    # 限制并发数
    semaphore = Semaphore(5)
    
    async def fetch_with_limit(user_id: int) -> dict:
        async with semaphore:
            try:
                return await fetch_user_with_timeout(user_id, timeout=5)
            except asyncio.TimeoutError:
                return {"id": user_id, "error": "请求超时"}
            except Exception as e:
                return {"id": user_id, "error": str(e)}
    
    # 使用 gather 并发执行，带 return_exceptions 处理异常
    results = await asyncio.gather(
        *[fetch_with_limit(uid) for uid in ids],
        return_exceptions=True
    )
    
    return {"results": results}

async def fetch_user_with_timeout(user_id: int, timeout: float) -> dict:
    """带超时的用户数据获取"""
    async with aiohttp.ClientSession() as session:
        async with session.get(
            f"https://api.example.com/users/{user_id}",
            timeout=aiohttp.ClientTimeout(total=timeout)
        ) as response:
            return await response.json()
```

### 性能优化技巧

| 优化策略 | 实现方式 | 效果 |
|----------|----------|------|
| **连接池** | `asyncpg.create_pool(min_size=5, max_size=20)` | 复用数据库连接 |
| **批量操作** | `executemany()` / `bulk_insert()` | 减少数据库往返 |
| **缓存** | Redis/In-memory cache | 减少重复计算 |
| **异步流** | `StreamingResponse` | 大文件分块传输 |
| **后台任务** | `BackgroundTasks` | 非关键路径异步化 |
| **索引优化** | 数据库索引 + 查询优化 | 加速数据检索 |

```python
from fastapi.responses import StreamingResponse

@app.get("/large-file/{file_id}")
async def download_large_file(file_id: str):
    """
    使用流式响应处理大文件
    避免将整个文件加载到内存
    """
    async def file_iterator():
        # 分块读取文件
        chunk_size = 64 * 1024  # 64KB
        with open(f"/data/files/{file_id}", "rb") as f:
            while chunk := f.read(chunk_size):
                yield chunk
    
    return StreamingResponse(
        file_iterator(),
        media_type="application/octet-stream",
        headers={"Content-Disposition": f"attachment; filename={file_id}"}
    )
```

---

## 路由与中间件

### 路由系统

```python
from fastapi import APIRouter, Path, Query, Body, Header, Cookie
from typing import Annotated

# 创建路由分组
api_router = APIRouter(prefix="/api/v1", tags=["v1 API"])
admin_router = APIRouter(prefix="/admin", tags=["管理后台"])

@api_router.get("/users/{user_id}")
async def get_user(
    user_id: Annotated[int, Path(gt=0, description="用户ID")],
    include_profile: bool = Query(False, description="包含用户资料")
):
    """获取用户信息"""
    user = fetch_user(user_id)
    if include_profile:
        user["profile"] = fetch_profile(user_id)
    return user

@api_router.post("/users/")
async def create_user(
    user_data: Annotated[dict, Body(embed=True)]
):
    """创建新用户"""
    return {"id": 1, **user_data}

@api_router.websocket("/ws/{client_id}")
async def websocket_endpoint(websocket: WebSocket, client_id: str):
    """WebSocket 端点"""
    await websocket.accept()
    try:
        while True:
            data = await websocket.receive_text()
            await websocket.send_text(f"收到: {data}")
    except WebSocketDisconnect:
        await websocket.close()

# 注册路由
app.include_router(api_router)
app.include_router(admin_router)
```

### 中间件

```python
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware
import time
from starlette.middleware.base import BaseHTTPMiddleware

app = FastAPI()

# 内置中间件：CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://example.com"],
    allow_credentials=True,
    allow_methods=["GET", "POST"],
    allow_headers=["*"],
)

# 内置中间件：GZip 压缩
app.add_middleware(GZipMiddleware, minimum_size=1000)

# 自定义中间件
class RequestTimingMiddleware(BaseHTTPMiddleware):
    """请求耗时统计中间件"""
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        
        # 处理请求
        response = await call_next(request)
        
        # 添加耗时头
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        
        return response

class RateLimitMiddleware(BaseHTTPMiddleware):
    """简单限流中间件"""
    def __init__(self, app, max_requests: int = 100, window: int = 60):
        super().__init__(app)
        self.max_requests = max_requests
        self.window = window
        self.requests = {}
    
    async def dispatch(self, request: Request, call_next):
        client_ip = request.client.host
        now = time.time()
        
        # 清理过期记录
        self.requests = {
            ip: times 
            for ip, times in self.requests.items()
            if now - times[-1] < self.window
        }
        
        # 检查限流
        if client_ip in self.requests:
            if len(self.requests[client_ip]) >= self.max_requests:
                return JSONResponse(
                    status_code=429,
                    content={"detail": "请求过于频繁"}
                )
            self.requests[client_ip].append(now)
        else:
            self.requests[client_ip] = [now]
        
        return await call_next(request)

app.add_middleware(RequestTimingMiddleware)
app.add_middleware(RateLimitMiddleware, max_requests=60, window=60)
```

---

## 后台任务与 WebSocket

### 后台任务

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import sendgrid
import logging

app = FastAPI()
logger = logging.getLogger(__name__)

class EmailRequest(BaseModel):
    to: str
    subject: str
    body: str

def send_email_task(to: str, subject: str, body: str):
    """发送邮件的后台任务（同步函数）"""
    try:
        sg = sendgrid.SendGridAPIClient(api_key="YOUR_API_KEY")
        data = {
            "personalizations": [{"to": [{"email": to}]}],
            "from": {"email": "noreply@example.com"},
            "subject": subject,
            "content": [{"type": "text/plain", "value": body}]
        }
        response = sg.send(data)
        logger.info(f"邮件发送成功: {response.status_code}")
    except Exception as e:
        logger.error(f"邮件发送失败: {e}")

async def process_report_task(report_id: str):
    """处理报告的后台任务（异步函数）"""
    # 模拟耗时操作
    await asyncio.sleep(10)
    logger.info(f"报告 {report_id} 处理完成")
    # 更新数据库状态
    await db.execute(
        "UPDATE reports SET status = 'completed' WHERE id = ?",
        (report_id,)
    )

@app.post("/reports/{report_id}/process")
async def process_report(
    report_id: str,
    background_tasks: BackgroundTasks
):
    """
    添加后台任务：
    后台任务在响应发送后执行，不阻塞客户端
    """
    # 立即返回响应
    background_tasks.add_task(process_report_task, report_id)
    return {"message": "报告处理任务已加入队列", "report_id": report_id}

@app.post("/contact/")
async def contact_form(
    email_req: EmailRequest,
    background_tasks: BackgroundTasks
):
    """联系表单：异步发送邮件"""
    background_tasks.add_task(
        send_email_task,
        email_req.to,
        email_req.subject,
        email_req.body
    )
    return {"message": "邮件已加入发送队列"}
```

### WebSocket

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from typing import List
import json

app = FastAPI()

class ConnectionManager:
    """WebSocket 连接管理器"""
    def __init__(self):
        # 活跃连接：room_id -> [WebSocket]
        self.active_connections: dict[str, List[WebSocket]] = {}
    
    async def connect(self, websocket: WebSocket, room_id: str):
        """接受连接并加入房间"""
        await websocket.accept()
        if room_id not in self.active_connections:
            self.active_connections[room_id] = []
        self.active_connections[room_id].append(websocket)
    
    def disconnect(self, websocket: WebSocket, room_id: str):
        """断开连接并离开房间"""
        if room_id in self.active_connections:
            self.active_connections[room_id].remove(websocket)
            if not self.active_connections[room_id]:
                del self.active_connections[room_id]
    
    async def broadcast(self, room_id: str, message: dict):
        """向房间内所有连接广播消息"""
        if room_id in self.active_connections:
            disconnected = []
            for connection in self.active_connections[room_id]:
                try:
                    await connection.send_json(message)
                except Exception:
                    disconnected.append(connection)
            # 清理断开连接
            for conn in disconnected:
                self.disconnect(conn, room_id)

manager = ConnectionManager()

@app.websocket("/ws/chat/{room_id}")
async def chat_websocket(websocket: WebSocket, room_id: str):
    """
    聊天 WebSocket 端点：
    支持房间隔离、消息广播、历史记录
    """
    await manager.connect(websocket, room_id)
    
    try:
        while True:
            # 接收消息
            data = await websocket.receive_text()
            message = json.loads(data)
            
            # 处理不同类型的消息
            if message["type"] == "chat":
                # 广播消息给房间内所有人
                await manager.broadcast(room_id, {
                    "type": "message",
                    "user": message["user"],
                    "content": message["content"],
                    "timestamp": asyncio.get_event_loop().time()
                })
            elif message["type"] == "typing":
                # 通知其他用户有人正在输入
                await manager.broadcast(room_id, {
                    "type": "typing",
                    "user": message["user"]
                }, exclude=websocket)
    
    except WebSocketDisconnect:
        manager.disconnect(websocket, room_id)
        await manager.broadcast(room_id, {
            "type": "leave",
            "user": "anonymous"
        })
```

---

## FastAPI 与 AI 框架集成

### LangChain 集成

```python
from fastapi import FastAPI, Request
from pydantic import BaseModel
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.chains import LLMChain
from langchain.schema import HumanMessage
import os

app = FastAPI()

# 初始化 LangChain
llm = ChatOpenAI(
    model="gpt-4o",
    api_key=os.getenv("OPENAI_API_KEY"),
    temperature=0.7
)

prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的{domain}助手。请用简洁专业的语言回答。"),
    ("human", "{question}")
])

chain = LLMChain(llm=llm, prompt=prompt)

class QuestionRequest(BaseModel):
    domain: str
    question: str

@app.post("/ai/ask")
async def ask_question(req: QuestionRequest):
    """
    AI 问答接口：
    使用 LangChain 链式调用处理问答
    """
    result = await chain.ainvoke({
        "domain": req.domain,
        "question": req.question
    })
    
    return {
        "answer": result["text"],
        "domain": req.domain
    }

# 流式响应
@app.post("/ai/chat")
async def chat_stream(req: QuestionRequest):
    """
    流式聊天接口：
    支持 Server-Sent Events 流式输出
    """
    async def generate():
        async for chunk in llm.astream(
            [HumanMessage(content=req.question)]
        ):
            yield f"data: {chunk.content}\n\n"
        yield "data: [DONE]\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream"
    )
```

### LlamaIndex 集成

```python
from llama_index import VectorStoreIndex, SimpleDirectoryReader
from llama_index.llms import OpenAI
from llama_index.storage.storage_context import StorageContext
from llama_index.vector_stores import PineconeVectorStore
from fastapi import FastAPI, UploadFile, File
import os

app = FastAPI()

# 初始化向量存储
pinecone_api_key = os.getenv("PINECONE_API_KEY")
pinecone_index = "my-knowledge-base"

# 创建/加载索引
def get_index():
    """获取或创建知识库索引"""
    vector_store = PineconeVectorStore(
        api_key=pinecone_api_key,
        index_name=pinecone_index
    )
    storage_context = StorageContext.from_defaults(
        vector_store=vector_store
    )
    
    # 如果索引不存在，创建新的
    if not vector_store.get_all():
        documents = SimpleDirectoryReader("./data").load_data()
        index = VectorStoreIndex.from_documents(
            documents,
            storage_context=storage_context
        )
    else:
        index = VectorStoreIndex.from_existing_index(
            vector_store=vector_store
        )
    
    return index

@app.post("/knowledge/query")
async def query_knowledge(question: str):
    """
    知识库问答接口：
    基于 RAG（检索增强生成）模式
    """
    index = get_index()
    query_engine = index.as_query_engine(
        similarity_top_k=5,
        streaming=True
    )
    
    response = query_engine.query(question)
    
    return {
        "answer": str(response),
        "source_nodes": [
            {
                "score": node.score,
                "content": node.text[:200] + "..."
            }
            for node in response.source_nodes
        ]
    }

@app.post("/knowledge/upload")
async def upload_documents(files: list[UploadFile] = File(...)):
    """
    上传文档到知识库：
    支持 PDF、TXT、Markdown 等格式
    """
    # 保存上传的文件
    for file in files:
        content = await file.read()
        with open(f"./data/{file.filename}", "wb") as f:
            f.write(content)
    
    # 重新索引
    documents = SimpleDirectoryReader("./data").load_data()
    index = get_index()
    index.insert_documents(documents)
    
    return {"message": f"已上传 {len(files)} 个文档并重新索引"}
```

### 与 AI 代理框架集成

```python
from agent import Agent  # AutoGen 或其他代理框架

@app.post("/ai/agent/task")
async def run_agent_task(task: str):
    """
    AI Agent 执行任务接口：
    支持多轮对话、工具调用、任务分解
    """
    agent = Agent(
        name="assistant",
        llm=ChatOpenAI(model="gpt-4o"),
        tools=[
            search_web,
            calculator,
            code_executor
        ]
    )
    
    result = await agent.run(task)
    
    return {
        "result": result,
        "steps": agent.get_execution_trace()
    }
```

---

## FastAPI vs Flask vs Django 对比

### 核心对比表

| 特性 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| **定位** | 现代 API 框架 | 微框架 | 全栈框架 |
| **诞生时间** | 2019 | 2010 | 2005 |
| **核心哲学** | 性能 + 类型安全 | 简约 + 灵活 | 全功能 + 约定优于配置 |
| **异步支持** | 原生 async/await | 需 Flask-COROUTING | 有限（Channels） |
| **数据验证** | Pydantic（原生） | 需手动/扩展 | Form/Serializer |
| **自动文档** | OpenAPI/Swagger/ReDoc | 需扩展 | DRF API Browser |
| **ORM** | SQLAlchemy（推荐） | SQLAlchemy（扩展） | Django ORM（内置） |
| **管理后台** | 无（需扩展） | 无 | Django Admin（内置） |
| **认证系统** | 需自行实现 | 扩展实现 | Django Auth（内置） |
| **学习曲线** | 低-中 | 低 | 中-高 |
| **性能（JSON API）** | 极高 | 高 | 中 |
| **生态成熟度** | 成长中 | 成熟 | 非常成熟 |
| **AI 集成** | 最佳 | 一般 | 一般 |
| **适用场景** | API、微服务、AI | 微服务、轻量 API | 全栈应用、内容网站 |

### 性能对比

| 场景 | FastAPI | Flask | Django |
|------|---------|-------|--------|
| **简单 JSON API** | 50,000 req/s | 35,000 req/s | 15,000 req/s |
| **数据库查询** | 8,000 req/s | 6,000 req/s | 4,000 req/s |
| **文件上传** | 800 MB/s | 600 MB/s | 400 MB/s |
| **WebSocket** | 10,000 msg/s | 需扩展 | 5,000 msg/s |

> [!TIP]
> **选型建议**：
> - **AI 应用后端**：FastAPI（LangChain/LlamaIndex 官方支持）
> - **微服务架构**：FastAPI 或 Flask
> - **内容管理系统**：Django（自带 Admin）
> - **快速原型**：Flask（最灵活）
> - **企业级全栈**：Django（生态完整）

### 迁移路径

如果项目从 Flask/Django 迁移到 FastAPI：

```python
# Flask -> FastAPI 最小改动迁移
from flask import Flask, jsonify, request
from fastapi import FastAPI, HTTPException

# 原有 Flask 代码
flask_app = Flask(__name__)

@flask_app.route("/users/<int:user_id>")
def get_user(user_id):
    return jsonify({"id": user_id, "name": "张三"})

# FastAPI 等效实现
app = FastAPI()

@app.get("/users/{user_id}")
async def get_user(user_id: int):
    if user_id <= 0:
        raise HTTPException(status_code=400, detail="Invalid user ID")
    return {"id": user_id, "name": "张三"}

# 保持向后兼容：包装 Flask 应用
from fastapi.middleware.wsgi import WSGIMiddleware

app.add_middleware(WSGIMiddleware, wsgi_app=flask_app)
```

---

## 实战场景与选型建议

### 场景一：AI 对话服务后端

```python
# 完整的 AI 对话服务架构
from fastapi import FastAPI, WebSocket
from langchain_openai import ChatOpenAI
from pydantic import BaseModel
import redis.asyncio as redis
import json

app = FastAPI(title="AI Chat Service", version="1.0.0")

# 配置
redis_client = redis.from_url("redis://localhost:6379")
llm = ChatOpenAI(model="gpt-4o", streaming=True)

class ChatRequest(BaseModel):
    session_id: str
    message: str
    system_prompt: str = "你是一个有帮助的AI助手。"

@app.post("/chat")
async def chat(req: ChatRequest):
    """
    AI 对话接口：
    - 支持多会话
    - 上下文记忆
    - 流式响应
    """
    # 获取历史上下文
    history_key = f"chat:{req.session_id}"
    history = await redis_client.get(history_key)
    messages = json.loads(history) if history else []
    
    # 添加新消息
    messages.append({"role": "user", "content": req.message})
    
    # 调用 LLM
    response = await llm.agenerate([messages])
    answer = response.generations[0][0].text
    
    # 保存上下文
    messages.append({"role": "assistant", "content": answer})
    await redis_client.setex(
        history_key, 
        3600,  # 1小时过期
        json.dumps(messages[-10:])  # 只保留最近10轮
    )
    
    return {"answer": answer, "session_id": req.session_id}

@app.get("/health")
async def health_check():
    """健康检查端点"""
    return {"status": "healthy"}
```

### 场景二：RAG 知识库 API

```python
from fastapi import FastAPI, UploadFile, File
from llama_index import VectorStoreIndex, SimpleDirectoryReader
from pydantic import BaseModel

app = FastAPI(title="RAG Knowledge Base API")

@app.post("/documents/upload")
async def upload_documents(files: list[UploadFile] = File(...)):
    """
    文档上传接口：
    自动分块、嵌入、存储
    """
    # 实现省略...
    pass

@app.post("/documents/query")
async def query_documents(question: str, top_k: int = 5):
    """
    知识库问答：
    检索 + 生成
    """
    # 实现省略...
    pass
```

### 场景三：实时数据管道

```python
from fastapi import FastAPI, WebSocket
from fastapi.responses import StreamingResponse
import asyncio

app = FastAPI(title="Real-time Data Pipeline")

@app.websocket("/stream/{source_id}")
async def data_stream(websocket: WebSocket, source_id: str):
    """
    实时数据流：
    支持多源聚合、过滤、转换
    """
    await websocket.accept()
    
    try:
        while True:
            # 从数据源获取数据
            data = await fetch_from_source(source_id)
            
            # 转换和过滤
            transformed = transform_data(data)
            
            # 发送
            await websocket.send_json(transformed)
            
            # 控制频率
            await asyncio.sleep(0.1)
    
    except WebSocketDisconnect:
        pass
```

### 选型决策树

```
项目类型
├── AI 应用后端
│   └── → FastAPI（LangChain/LlamaIndex 原生支持）
├── 微服务/API
│   ├── 需要高性能 → FastAPI
│   ├── 追求轻量 → Flask
│   └── 需要快速开发 → Django
├── 内容管理/博客
│   └── → Django（自带 Admin）
├── 实时应用（WebSocket）
│   └── → FastAPI（原生支持）
├── 企业级全栈
│   └── → Django（生态完整）
└── 学习/原型
    ├── 快速原型 → Flask
    └── 现代化 API → FastAPI
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| FastAPI 官方文档 | https://fastapi.tiangolo.com |
| Pydantic 文档 | https://docs.pydantic.dev |
| Starlette 文档 | https://www.starlette.io |
| Uvicorn 文档 | https://www.uvicorn.org |

### 社区资源

| 资源 | 说明 |
|------|------|
| FastAPI GitHub | 22k+ Stars |
| FastAPI Reddit | r/FastAPI 社区 |
| FastAPI Discord | 官方 Discord |

### 性能基准

| 测试 | 框架 | 结果 |
|------|------|------|
| TechEmpower JSON | FastAPI + Uvicorn | 50,000+ RPS |
| 异步数据库查询 | FastAPI + asyncpg | 15,000+ RPS |
| WebSocket 广播 | FastAPI | 10,000 msg/s |

---

## 完整安装与环境配置

### 环境要求

```bash
# Python 版本要求
python --version  # >= 3.9 (推荐 3.11+)

# pip 版本
pip --version  # >= 21.0
```

### 项目创建

```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Linux/Mac
source venv/bin/activate

# Windows
venv\Scripts\activate

# 安装 FastAPI 和核心依赖
pip install fastapi
pip install uvicorn[standard]  # ASGI 服务器
pip install pydantic  # 数据验证

# 开发依赖
pip install --save-dev pytest pytest-asyncio httpx
pip install --save-dev black isort mypy
pip install --save-dev pytest-cov
```

### 推荐的完整依赖

```bash
# 核心框架
pip install fastapi uvicorn[standard] pydantic pydantic-settings

# 数据验证与序列化
pip install email-validator

# 数据库
pip install sqlalchemy asyncpg alembic
pip install databases

# Redis
pip install redis aioredis

# 认证
pip install python-jose[cryptography] passlib[bcrypt] python-multipart

# API 文档
pip install fastapi[all]  # 包含 openapi 和 swagger

# 异步任务队列
pip install celery

# 测试
pip install pytest pytest-asyncio httpx
pip install pytest-cov

# 开发工具
pip install black isort mypy
```

### 项目结构

```
my-fastapi-app/
├── app/
│   ├── __init__.py
│   ├── main.py              # 应用入口
│   ├── config.py            # 配置管理
│   ├── database.py          # 数据库连接
│   ├── deps.py              # 依赖注入
│   ├── models/              # SQLAlchemy 模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── post.py
│   ├── schemas/             # Pydantic 模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   └── common.py
│   ├── routers/             # API 路由
│   │   ├── __init__.py
│   │   ├── users.py
│   │   ├── posts.py
│   │   └── auth.py
│   ├── services/            # 业务逻辑
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── post_service.py
│   ├── core/                # 核心功能
│   │   ├── __init__.py
│   │   ├── security.py
│   │   └── exceptions.py
│   └── utils/               # 工具函数
│       ├── __init__.py
│       └── helpers.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_users.py
│   └── test_posts.py
├── alembic/                 # 数据库迁移
│   ├── env.py
│   └── versions/
├── .env
├── .env.example
├── .gitignore
├── requirements.txt
├── requirements-dev.txt
├── pyproject.toml
├── Dockerfile
└── docker-compose.yml
```

### pyproject.toml 配置

```toml
[project]
name = "my-fastapi-app"
version = "1.0.0"
description = "FastAPI REST API"
requires-python = ">=3.9"
dependencies = [
    "fastapi>=0.109.0",
    "uvicorn[standard]>=0.27.0",
    "pydantic>=2.5.0",
    "pydantic-settings>=2.1.0",
    "sqlalchemy>=2.0.0",
    "asyncpg>=0.29.0",
    "alembic>=1.13.0",
    "redis>=5.0.0",
    "python-jose[cryptography]>=3.3.0",
    "passlib[bcrypt]>=1.7.4",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-asyncio>=0.23.0",
    "httpx>=0.26.0",
    "pytest-cov>=4.1.0",
    "black>=24.1.0",
    "isort>=5.13.0",
    "mypy>=1.8.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'

[tool.isort]
profile = "black"
line_length = 88

[tool.mypy]
python_version = "3.11"
strict = true
warn_return_any = true
warn_unused_configs = true

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

### 配置管理

```python
# app/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache
from typing import Literal


class Settings(BaseSettings):
    # 应用配置
    APP_NAME: str = "My FastAPI App"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    ENVIRONMENT: Literal["development", "staging", "production"] = "development"

    # 服务器配置
    HOST: str = "0.0.0.0"
    PORT: int = 8000

    # 数据库配置
    DATABASE_URL: str = "postgresql://user:password@localhost:5432/mydb"
    DATABASE_POOL_SIZE: int = 20
    DATABASE_MAX_OVERFLOW: int = 10

    # Redis 配置
    REDIS_URL: str = "redis://localhost:6379"

    # JWT 配置
    JWT_SECRET_KEY: str  # 必须设置
    JWT_ALGORITHM: str = "HS256"
    JWT_ACCESS_TOKEN_EXPIRE_MINUTES: int = 15
    JWT_REFRESH_TOKEN_EXPIRE_DAYS: int = 7

    # CORS 配置
    CORS_ORIGINS: list[str] = ["http://localhost:3000"]

    # 限流配置
    RATE_LIMIT_PER_MINUTE: int = 60

    class Config:
        env_file = ".env"
        case_sensitive = True


@lru_cache
def get_settings() -> Settings:
    return Settings()


settings = get_settings()
```

---

## 数据库集成

### SQLAlchemy 异步配置

```python
# app/database.py
from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import declarative_base
from sqlalchemy.pool import NullPool
from contextlib import asynccontextmanager
from typing import AsyncGenerator

from app.config import settings

# 创建异步引擎
engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
    pool_size=settings.DATABASE_POOL_SIZE,
    max_overflow=settings.DATABASE_MAX_OVERFLOW,
    pool_pre_ping=True,  # 连接前测试
    poolclass=NullPool if settings.ENVIRONMENT == "testing" else None,
)

# 创建会话工厂
async_session_factory = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,
    autoflush=False,
    autocommit=False,
)

# Base 类
Base = declarative_base()


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """依赖注入：获取数据库会话"""
    async with async_session_factory() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()


@asynccontextmanager
async def get_db_context() -> AsyncGenerator[AsyncSession, None]:
    """上下文管理器方式获取数据库会话"""
    async with async_session_factory() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

### 数据库模型定义

```python
# app/models/user.py
from sqlalchemy import Column, String, Boolean, DateTime, Enum as SQLEnum, func
from sqlalchemy.orm import relationship
import uuid
import enum

from app.database import Base


class UserRole(str, enum.Enum):
    USER = "USER"
    ADMIN = "ADMIN"
    MODERATOR = "MODERATOR"


class User(Base):
    __tablename__ = "users"

    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    email = Column(String(255), unique=True, nullable=False, index=True)
    username = Column(String(50), unique=True, nullable=False, index=True)
    hashed_password = Column(String(255), nullable=False)
    first_name = Column(String(100), nullable=True)
    last_name = Column(String(100), nullable=True)
    avatar = Column(String(500), nullable=True)
    role = Column(SQLEnum(UserRole), default=UserRole.USER, nullable=False)
    is_active = Column(Boolean, default=True, nullable=False)
    is_verified = Column(Boolean, default=False, nullable=False)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
    last_login_at = Column(DateTime(timezone=True), nullable=True)

    # 关系
    posts = relationship("Post", back_populates="author", lazy="selectin")
    comments = relationship("Comment", back_populates="author", lazy="selectin")

    def __repr__(self):
        return f"<User {self.email}>"
```

```python
# app/models/post.py
from sqlalchemy import Column, String, Text, Boolean, DateTime, Integer, ForeignKey, func
from sqlalchemy.orm import relationship
import uuid

from app.database import Base


class Post(Base):
    __tablename__ = "posts"

    id = Column(String(36), primary_key=True, default=lambda: str(uuid.uuid4()))
    title = Column(String(255), nullable=False)
    slug = Column(String(255), unique=True, nullable=False, index=True)
    content = Column(Text, nullable=False)
    excerpt = Column(Text, nullable=True)
    featured_image = Column(String(500), nullable=True)
    published = Column(Boolean, default=False, nullable=False, index=True)
    published_at = Column(DateTime(timezone=True), nullable=True)
    view_count = Column(Integer, default=0, nullable=False)
    author_id = Column(String(36), ForeignKey("users.id"), nullable=False, index=True)
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())

    # 关系
    author = relationship("User", back_populates="posts", lazy="selectin")
    comments = relationship("Comment", back_populates="post", lazy="selectin")

    def __repr__(self):
        return f"<Post {self.title}>"
```

### Alembic 迁移配置

```python
# alembic/env.py
from logging.config import fileConfig
from sqlalchemy import pool
from sqlalchemy.ext.asyncio import async_engine_from_config
from alembic import context
import asyncio

from app.config import settings
from app.database import Base
from app.models import user, post  # 导入所有模型

config = context.config

# 设置数据库 URL
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection):
    context.configure(connection=connection, target_metadata=target_metadata)

    with context.begin_transaction():
        context.run_migrations()


async def run_migrations_online() -> None:
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


if context.is_offline_mode():
    run_migrations_offline()
else:
    asyncio.run(run_migrations_online())
```

---

## Pydantic Schemas

```python
# app/schemas/user.py
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from typing import Optional, List
from datetime import datetime
from enum import Enum


class UserRole(str, Enum):
    USER = "USER"
    ADMIN = "ADMIN"
    MODERATOR = "MODERATOR"


class UserBase(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    first_name: Optional[str] = Field(None, max_length=100)
    last_name: Optional[str] = Field(None, max_length=100)


class UserCreate(UserBase):
    password: str = Field(..., min_length=8)
    # Pydantic V2 密码验证
    password: str = Field(
        ...,
        min_length=8,
        pattern=r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)",
    )


class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    first_name: Optional[str] = Field(None, max_length=100)
    last_name: Optional[str] = Field(None, max_length=100)
    avatar: Optional[str] = None


class UserPasswordUpdate(BaseModel):
    current_password: str
    new_password: str = Field(..., min_length=8)


class UserResponse(UserBase):
    model_config = ConfigDict(from_attributes=True)

    id: str
    role: UserRole
    is_active: bool
    is_verified: bool
    avatar: Optional[str] = None
    created_at: datetime
    updated_at: Optional[datetime] = None


class UserListResponse(BaseModel):
    data: List[UserResponse]
    total: int
    page: int
    per_page: int
    pages: int
```

---

## 认证授权

### JWT 安全模块

```python
# app/core/security.py
from datetime import datetime, timedelta, timezone
from typing import Optional, Any
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession

from app.config import settings
from app.database import get_db

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()


def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)


def hash_password(password: str) -> str:
    return pwd_context.hash(password)


def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.now(timezone.utc) + expires_delta
    else:
        expire = datetime.now(timezone.utc) + timedelta(
            minutes=settings.JWT_ACCESS_TOKEN_EXPIRE_MINUTES
        )
    to_encode.update({"exp": expire, "type": "access"})
    return jwt.encode(to_encode, settings.JWT_SECRET_KEY, algorithm=settings.JWT_ALGORITHM)


def create_refresh_token(data: dict) -> str:
    to_encode = data.copy()
    expire = datetime.now(timezone.utc) + timedelta(
        days=settings.JWT_REFRESH_TOKEN_EXPIRE_DAYS
    )
    to_encode.update({"exp": expire, "type": "refresh"})
    return jwt.encode(to_encode, settings.JWT_SECRET_KEY, algorithm=settings.JWT_ALGORITHM)


def decode_token(token: str) -> dict:
    try:
        payload = jwt.decode(
            token, settings.JWT_SECRET_KEY, algorithms=[settings.JWT_ALGORITHM]
        )
        return payload
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Could not validate credentials",
            headers={"WWW-Authenticate": "Bearer"},
        )


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db),
) -> dict:
    token = credentials.credentials
    payload = decode_token(token)

    if payload.get("type") != "access":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token type",
        )

    from app.services.user_service import UserService
    user_service = UserService(db)
    user = await user_service.get_by_id(payload.get("sub"))

    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found or inactive",
        )

    return {
        "id": user.id,
        "email": user.email,
        "username": user.username,
        "role": user.role.value,
    }


def require_role(*roles: str):
    """角色验证装饰器工厂"""
    async def role_checker(current_user: dict = Depends(get_current_user)) -> dict:
        if current_user.get("role") not in roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Insufficient permissions",
            )
        return current_user
    return role_checker
```

### 认证路由

```python
# app/routers/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession
from datetime import timedelta

from app.database import get_db
from app.schemas.user import UserCreate, UserResponse, Token
from app.services.user_service import UserService
from app.core.security import (
    hash_password,
    verify_password,
    create_access_token,
    create_refresh_token,
    decode_token,
    get_current_user,
)
from app.config import settings

router = APIRouter(prefix="/auth", tags=["Authentication"])


@router.post("/register", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def register(
    user_data: UserCreate,
    db: AsyncSession = Depends(get_db),
):
    user_service = UserService(db)

    # 检查邮箱是否存在
    existing_user = await user_service.get_by_email(user_data.email)
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Email already registered",
        )

    # 检查用户名是否存在
    existing_username = await user_service.get_by_username(user_data.username)
    if existing_username:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Username already taken",
        )

    # 创建用户
    user = await user_service.create(
        email=user_data.email,
        username=user_data.username,
        password=hash_password(user_data.password),
        first_name=user_data.first_name,
        last_name=user_data.last_name,
    )

    return user


@router.post("/login", response_model=Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db),
):
    user_service = UserService(db)
    user = await user_service.get_by_email(form_data.username)

    if not user or not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User account is inactive",
        )

    # 更新最后登录时间
    await user_service.update_last_login(user.id)

    # 生成 token
    access_token = create_access_token(
        data={"sub": user.id, "email": user.email, "role": user.role.value}
    )
    refresh_token = create_refresh_token(data={"sub": user.id})

    return {
        "access_token": access_token,
        "refresh_token": refresh_token,
        "token_type": "bearer",
    }


@router.post("/refresh", response_model=Token)
async def refresh_token(
    refresh_token: str,
    db: AsyncSession = Depends(get_db),
):
    payload = decode_token(refresh_token)

    if payload.get("type") != "refresh":
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid token type",
        )

    user_service = UserService(db)
    user = await user_service.get_by_id(payload.get("sub"))

    if not user or not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="User not found or inactive",
        )

    access_token = create_access_token(
        data={"sub": user.id, "email": user.email, "role": user.role.value}
    )
    new_refresh_token = create_refresh_token(data={"sub": user.id})

    return {
        "access_token": access_token,
        "refresh_token": new_refresh_token,
        "token_type": "bearer",
    }


@router.get("/me", response_model=UserResponse)
async def get_current_user_info(
    current_user: dict = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    user_service = UserService(db)
    user = await user_service.get_by_id(current_user["id"])
    return user
```

---

## 常见陷阱与最佳实践

### 陷阱 1：忘记 await 异步操作

```python
# ❌ 错误：异步操作没有 await
@router.get("/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    users = db.execute(select(User))  # 缺少 await
    return users.all()

# ✅ 正确：使用 await
@router.get("/users")
async def get_users(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User))
    users = result.scalars().all()
    return users
```

### 陷阱 2：N+1 查询问题

```python
# ❌ 错误：循环中查询
@router.get("/posts")
async def get_posts(db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(Post))
    posts = result.scalars().all()

    for post in posts:
        author = await db.execute(
            select(User).where(User.id == post.author_id)
        )
        post.author = author.scalar_one()

    return posts

# ✅ 正确：使用 joinedload 预加载
@router.get("/posts")
async def get_posts(db: AsyncSession = Depends(get_db)):
    from sqlalchemy.orm import selectinload

    result = await db.execute(
        select(Post).options(selectinload(Post.author))
    )
    posts = result.scalars().all()
    return posts
```

### 陷阱 3：未正确处理异常

```python
# ❌ 错误：未捕获的异常
@router.get("/users/{user_id}")
async def get_user(user_id: str, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)  # 如果不存在会抛出异常
    return user

# ✅ 正确：处理不存在的情况
@router.get("/users/{user_id}")
async def get_user(user_id: str, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### 最佳实践清单

1. **使用异步**：使用 `async/await` 和异步驱动程序（如 `asyncpg`、`aioredis`）。
2. **Pydantic V2**：使用 `model_config = ConfigDict(from_attributes=True)` 替代 `Config` 类。
3. **依赖注入**：使用 FastAPI 的 `Depends()` 进行依赖注入。
4. **错误处理**：使用自定义异常处理器统一处理错误。
5. **数据库会话**：在每个请求结束时正确关闭会话。
6. **索引**：为经常查询的列添加数据库索引。
7. **分页**：对列表接口使用分页，避免大结果集。
8. **验证**：使用 Pydantic 进行输入验证。
9. **类型注解**：使用完整的类型注解。
10. **文档**：利用 FastAPI 自动生成的 OpenAPI 文档。

---

## 与其他框架对比

### FastAPI vs Flask

| 特性 | FastAPI | Flask |
|------|---------|-------|
| **性能** | 极高 | 中等 |
| **异步支持** | 原生 | 需要扩展 |
| **数据验证** | Pydantic 内置 | 需要扩展 |
| **类型安全** | 完整类型推断 | 有限 |
| **自动文档** | OpenAPI/Swagger | 需要扩展 |
| **学习曲线** | 中等 | 平缓 |
| **适用场景** | 现代 API | 轻量应用 |

### FastAPI vs Django

| 特性 | FastAPI | Django |
|------|---------|--------|
| **架构** | 轻量 | 全功能 |
| **ORM** | SQLAlchemy/其他 | Django ORM |
| **Admin** | 无 | 内置 |
| **认证** | DIY 或扩展 | 内置 |
| **表单** | Pydantic | Django Forms |
| **适用场景** | API-first | 全栈 |

### FastAPI vs Express.js

| 特性 | FastAPI | Express.js |
|------|---------|-----------|
| **语言** | Python | JavaScript |
| **类型** | 强类型 | 弱类型/TS |
| **性能** | 极高 | 高 |
| **生态** | Python 生态 | Node.js 生态 |
| **适用场景** | AI/数据应用 | 通用 Web |

---

> [!SUCCESS]
> 本文档已全面覆盖 FastAPI 的核心原理、Pydantic 数据验证、依赖注入、异步编程、与 AI 框架集成等关键知识点。FastAPI 在 2026 年已成为 AI 应用后端的首选框架，其类型安全、高性能和丰富的 AI 集成生态使其成为 vibecoding 场景下的最佳 Python Web 框架选择。
