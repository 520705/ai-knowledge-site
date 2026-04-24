---
title: AI应用API化部署：从设计到实现
date: 2026-04-24
tags:
  - API部署
  - REST API
  - FastAPI
  - 认证授权
  - 限流
  - Webhook
  - 灰度发布
  - OpenAPI
categories:
  - 智能体搭建
  - API部署
alias: AI Application API Deployment
description: 深入讲解AI应用的REST API设计、FastAPI框架实现、认证授权机制、限流策略、Webhook配置、多版本管理与灰度发布策略，通过完整的项目结构和代码示例，帮助构建生产级别的AI API服务。
---

# AI应用API化部署：从设计到实现

> [!NOTE] 这篇指南讲什么
> 把AI能力封装成API是构建AI应用的基础。这篇指南从API设计原则讲起，详细介绍FastAPI框架下的完整实现，包括认证、限流、Webhook、灰度发布等生产级功能，让你能够构建稳定可靠的AI API服务。

## 核心关键词速览

| 关键词 | 说明 | 关键词 | 说明 |
|--------|------|--------|------|
| REST API | REST风格接口 | JWT认证 | JSON Web Token |
| API Key | 密钥认证 | 限流策略 | Rate Limiting |
| Webhook | 回调通知 | API版本 | Versioning |
| 灰度发布 | Canary Release | OpenAPI | API规范 |
| Swagger | API文档 | API网关 | API Gateway |
| 流式响应 | Streaming | 异步处理 | Async |

## 1. 为什么需要API化部署？

### 1.1 API化 vs 直接调用

很多新手直接这样用AI：

```python
# 直接调用 - 小朋友的做法
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
print(response.choices[0].message.content)
```

问题在哪？
- API密钥暴露在前端，风险极大
- 无法限流，容易被刷
- 没有日志，不知道谁在调用
- 无法追踪用量和计费
- 无法做缓存和优化

**API化部署才是正确的做法：**

```
┌──────────┐      ┌──────────┐      ┌──────────┐      ┌──────────┐
│  前端    │ ---> │  API    │ ---> │  业务   │ ---> │  LLM    │
│  应用    │      │  网关    │      │  逻辑   │      │  服务    │
└──────────┘      └──────────┘      └──────────┘      └──────────┘
                        │                                   │
                        ↓                                   ↓
                 ┌──────────┐                        ┌──────────┐
                 │ 限流     │                        │ 模型调用  │
                 │ 日志     │                        │ 缓存     │
                 │ 认证     │                        │ 监控     │
                 └──────────┘                        └──────────┘
```

### 1.2 API化的优势

| 优势 | 说明 |
|------|------|
| **安全性** | 密钥不暴露，保护后端服务 |
| **可控性** | 限流、日志、监控全都有 |
| **可扩展性** | 轻松增加实例，支持更多用户 |
| **可计费** | 追踪用量，实现按需计费 |
| **可复用** | 一次开发，多端使用 |
| **可维护** | 集中管理，易于维护 |

## 2. API设计原则

### 2.1 RESTful规范

RESTful API设计要遵循几个原则：

1. **用名词不用动词**： `/users` 而不是 `/getUsers`
2. **使用HTTP方法**：
   - GET：查询
   - POST：创建
   - PUT：完整更新
   - PATCH：部分更新
   - DELETE：删除
3. **使用复数名词**：`/agents` 而不是 `/agent`
4. **使用嵌套路径**：`/agents/{id}/messages`

### 2.2 API端点设计

```
┌─────────────────────────────────────────────────────────────┐
│                      API端点设计                               │
│                                                            │
│  /v1/agents                                               │
│    ├── POST   /agents              # 创建Agent              │
│    ├── GET    /agents              # 列出Agent              │
│    └── GET    /agents/{id}        # 获取Agent详情           │
│                                                            │
│  /v1/agents/{id}                                         │
│    ├── PUT    /agents/{id}        # 更新Agent              │
│    ├── DELETE /agents/{id}        # 删除Agent              │
│    └── POST   /agents/{id}/chat   # 对话                   │
│                                                            │
│  /v1/knowledge                                            │
│    ├── POST   /knowledge           # 创建知识库             │
│    ├── POST   /knowledge/{id}/documents  # 上传文档        │
│    └── GET    /knowledge/{id}/search    # 搜索             │
│                                                            │
│  /v1/webhooks                                             │
│    ├── POST   /webhooks            # 创建Webhook           │
│    └── GET    /webhooks            # 列出Webhook           │
└─────────────────────────────────────────────────────────────┘
```

### 2.3 标准响应格式

```python
from typing import Generic, TypeVar, Optional
from pydantic import BaseModel
from datetime import datetime

T = TypeVar('T')

class BaseResponse(BaseModel, Generic[T]):
    """标准响应基类"""
    success: bool = True
    message: Optional[str] = None
    
    class Config:
        json_schema_extra = {
            "example": {
                "success": True,
                "message": "操作成功",
                "data": {},
                "request_id": "req_abc123"
            }
        }


class DataResponse(BaseResponse[T]):
    """带数据的响应"""
    data: Optional[T] = None


class ErrorDetail(BaseModel):
    """错误详情"""
    code: str
    message: str
    field: Optional[str] = None


class ErrorResponse(BaseModel):
    """错误响应"""
    success: bool = False
    error: ErrorDetail
    request_id: str


class PaginatedResponse(BaseResponse):
    """分页响应"""
    data: list
    total: int
    page: int
    page_size: int
    has_more: bool
```

## 3. FastAPI完整项目结构

### 3.1 目录结构

```
ai_api/
├── app/
│   ├── __init__.py
│   ├── main.py                    # 应用入口
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py                # 依赖注入
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── agents.py          # Agent管理
│   │       ├── chat.py            # 对话接口
│   │       ├── knowledge.py       # 知识库接口
│   │       ├── webhooks.py        # Webhook接口
│   │       └── users.py           # 用户接口
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py              # 配置管理
│   │   ├── security.py            # 安全认证
│   │   ├── rate_limit.py          # 限流
│   │   ├── webhook.py             # Webhook
│   │   └── canary.py              # 灰度
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── agent.py
│   │   └── conversation.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── agent.py
│   │   ├── chat.py
│   │   └── common.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── agent_service.py
│   │   ├── chat_service.py
│   │   └── llm_service.py
│   └── db/
│       ├── __init__.py
│       ├── database.py
│       └── redis.py
├── tests/
├── docker/
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
└── README.md
```

### 3.2 配置管理

```python
# app/core/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache
from typing import List
import os


class Settings(BaseSettings):
    """应用配置"""
    
    # 应用基础
    APP_NAME: str = "AI Agent API"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False
    
    # 服务器
    HOST: str = "0.0.0.0"
    PORT: int = 8000
    WORKERS: int = 4
    
    # 数据库
    DATABASE_URL: str = "postgresql+asyncpg://user:pass@localhost/ai_db"
    REDIS_URL: str = "redis://localhost:6379/0"
    
    # 安全
    SECRET_KEY: str = "your-secret-key-change-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    REFRESH_TOKEN_EXPIRE_DAYS: int = 7
    
    # CORS
    CORS_ORIGINS: List[str] = ["http://localhost:3000"]
    
    # 限流
    RATE_LIMIT_PER_MINUTE: int = 60
    RATE_LIMIT_PER_HOUR: int = 1000
    
    # LLM配置
    OPENAI_API_KEY: str = ""
    OPENAI_BASE_URL: str = "https://api.openai.com/v1"
    DEFAULT_MODEL: str = "gpt-4o"
    
    # 日志
    LOG_LEVEL: str = "INFO"
    LOG_FILE: str = "logs/app.log"
    
    class Config:
        env_file = ".env"
        case_sensitive = True


@lru_cache()
def get_settings() -> Settings:
    """获取配置单例"""
    return Settings()


settings = get_settings()
```

### 3.3 依赖注入

```python
# app/api/deps.py
from typing import Annotated, AsyncGenerator
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import jwt

from app.core.config import settings
from app.core.security import decode_token
from app.db.database import get_db
from app.db.redis import get_redis

security = HTTPBearer(auto_error=False)


async def get_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
) -> dict:
    """获取当前用户"""
    if not credentials:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="未提供认证信息"
        )
    
    try:
        payload = decode_token(credentials.credentials)
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Token已过期"
        )
    except jwt.InvalidTokenError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="无效的Token"
        )


async def get_current_user_optional(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
) -> dict | None:
    """获取当前用户（可选）"""
    if not credentials:
        return None
    
    try:
        return decode_token(credentials.credentials)
    except:
        return None


async def get_api_key(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)]
) -> dict:
    """验证API Key"""
    if not credentials:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="缺少API Key"
        )
    
    # 查询API Key
    api_key_data = await verify_api_key(credentials.credentials)
    
    if not api_key_data:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="无效的API Key"
        )
    
    return api_key_data


# 类型别名
CurrentUser = Annotated[dict, Depends(get_current_user)]
CurrentUserOptional = Annotated[dict | None, Depends(get_current_user_optional)]
DbSession = Annotated[AsyncSession, Depends(get_db)]
RedisClient = Annotated, Depends(get_redis)]
```

### 3.4 应用入口

```python
# app/main.py
from contextlib import asynccontextmanager
from fastapi import FastAPI, Request
from fastapi.middleware.cors import CORSMiddleware
from fastapi.responses import JSONResponse
import logging

from app.core.config import settings
from app.core.rate_limit import RateLimitMiddleware
from app.core.security import auth_middleware
from app.api.v1 import agents, chat, knowledge, webhooks, users
from app.db.database import init_db, close_db
from app.services.llm_service import init_llm_clients

# 配置日志
logging.basicConfig(
    level=getattr(logging, settings.LOG_LEVEL),
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s"
)
logger = logging.getLogger(__name__)


@asynccontextmanager
async def lifespan(app: FastAPI):
    """应用生命周期"""
    # 启动
    logger.info("正在启动应用...")
    
    # 初始化数据库
    await init_db()
    
    # 初始化LLM客户端
    await init_llm_clients()
    
    logger.info("应用启动完成")
    
    yield
    
    # 关闭
    logger.info("正在关闭应用...")
    await close_db()
    logger.info("应用已关闭")


# 创建应用
app = FastAPI(
    title=settings.APP_NAME,
    description="""
    AI Agent API服务
    
    ## 功能
    - Agent管理
    - 智能对话
    - 知识库管理
    - Webhook通知
    """,
    version=settings.APP_VERSION,
    docs_url="/docs",
    redoc_url="/redoc",
    openapi_url="/openapi.json",
    lifespan=lifespan
)

# CORS中间件
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 限流中间件
app.add_middleware(RateLimitMiddleware)

# 注册路由
app.include_router(
    agents.router,
    prefix="/v1",
    tags=["Agent管理"]
)

app.include_router(
    chat.router,
    prefix="/v1",
    tags=["对话接口"]
)

app.include_router(
    knowledge.router,
    prefix="/v1",
    tags=["知识库"]
)

app.include_router(
    webhooks.router,
    prefix="/v1",
    tags=["Webhook"]
)

app.include_router(
    users.router,
    prefix="/v1",
    tags=["用户"]
)


# 全局异常处理
@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    logger.error(f"全局异常: {exc}", exc_info=True)
    
    return JSONResponse(
        status_code=500,
        content={
            "success": False,
            "error": {
                "code": "INTERNAL_ERROR",
                "message": "服务器内部错误"
            },
            "request_id": request.headers.get("X-Request-ID", "")
        }
    )


# 健康检查
@app.get("/health", tags=["系统"])
async def health_check():
    return {"status": "healthy", "version": settings.APP_VERSION}


# 就绪检查
@app.get("/ready", tags=["系统"])
async def ready_check():
    # 检查数据库连接
    # 检查Redis连接
    # 检查LLM服务
    return {"status": "ready"}
```

## 4. 完整API实现

### 4.1 Agent管理接口

```python
# app/schemas/agent.py
from typing import Optional, List
from pydantic import BaseModel, Field
from datetime import datetime
from enum import Enum


class AgentType(str, Enum):
    """Agent类型"""
    CHAT = "chat"
    RAG = "rag"
    AGENT = "agent"


class AgentStatus(str, Enum):
    """Agent状态"""
    ACTIVE = "active"
    INACTIVE = "inactive"
    ERROR = "error"


class AgentConfig(BaseModel):
    """Agent配置"""
    model: str = Field(default="gpt-4o", description="使用的模型")
    temperature: float = Field(default=0.7, ge=0, le=2)
    max_tokens: int = Field(default=2000, ge=1, le=32000)
    system_prompt: Optional[str] = None
    tools: List[str] = Field(default_factory=list)
    knowledge_base_ids: List[str] = Field(default_factory=list)


class AgentCreate(BaseModel):
    """创建Agent"""
    name: str = Field(..., min_length=1, max_length=100)
    description: Optional[str] = Field(None, max_length=500)
    agent_type: AgentType = AgentType.CHAT
    config: AgentConfig = Field(default_factory=AgentConfig)


class AgentUpdate(BaseModel):
    """更新Agent"""
    name: Optional[str] = Field(None, min_length=1, max_length=100)
    description: Optional[str] = None
    config: Optional[AgentConfig] = None
    status: Optional[AgentStatus] = None


class AgentResponse(BaseModel):
    """Agent响应"""
    id: str
    name: str
    description: Optional[str]
    agent_type: AgentType
    status: AgentStatus
    config: AgentConfig
    created_at: datetime
    updated_at: datetime
    user_id: str

    class Config:
        from_attributes = True


class AgentListResponse(BaseModel):
    """Agent列表响应"""
    items: List[AgentResponse]
    total: int
    page: int
    page_size: int
```

```python
# app/api/v1/agents.py
from typing import Annotated
from fastapi import APIRouter, Depends, HTTPException, Query, status
from uuid import uuid4

from app.api.deps import CurrentUser, DbSession
from app.schemas.agent import (
    AgentCreate, AgentUpdate, AgentResponse, 
    AgentListResponse, AgentConfig
)
from app.services.agent_service import AgentService

router = APIRouter()


@router.post("/agents", response_model=AgentResponse, status_code=status.HTTP_201_CREATED)
async def create_agent(
    agent_data: AgentCreate,
    current_user: CurrentUser,
    db: DbSession
):
    """创建新的Agent"""
    service = AgentService(db)
    
    # 检查名称唯一性
    existing = await service.get_by_name(agent_data.name, current_user["sub"])
    if existing:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail=f"Agent名称 '{agent_data.name}' 已存在"
        )
    
    agent = await service.create(
        user_id=current_user["sub"],
        data=agent_data
    )
    
    return AgentResponse.model_validate(agent)


@router.get("/agents", response_model=AgentListResponse)
async def list_agents(
    current_user: CurrentUser,
    db: DbSession,
    page: int = Query(1, ge=1),
    page_size: int = Query(20, ge=1, le=100),
    agent_type: str = None,
    status: str = None
):
    """列出用户的所有Agent"""
    service = AgentService(db)
    
    agents, total = await service.list_by_user(
        user_id=current_user["sub"],
        page=page,
        page_size=page_size,
        agent_type=agent_type,
        status=status
    )
    
    return AgentListResponse(
        items=[AgentResponse.model_validate(a) for a in agents],
        total=total,
        page=page,
        page_size=page_size
    )


@router.get("/agents/{agent_id}", response_model=AgentResponse)
async def get_agent(
    agent_id: str,
    current_user: CurrentUser,
    db: DbSession
):
    """获取Agent详情"""
    service = AgentService(db)
    agent = await service.get_by_id(agent_id)
    
    if not agent:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Agent不存在"
        )
    
    # 权限检查
    if agent.user_id != current_user["sub"]:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="无权限访问此Agent"
        )
    
    return AgentResponse.model_validate(agent)


@router.put("/agents/{agent_id}", response_model=AgentResponse)
async def update_agent(
    agent_id: str,
    update_data: AgentUpdate,
    current_user: CurrentUser,
    db: DbSession
):
    """更新Agent"""
    service = AgentService(db)
    
    agent = await service.get_by_id(agent_id)
    if not agent or agent.user_id != current_user["sub"]:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Agent不存在"
        )
    
    updated = await service.update(agent_id, update_data)
    return AgentResponse.model_validate(updated)


@router.delete("/agents/{agent_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_agent(
    agent_id: str,
    current_user: CurrentUser,
    db: DbSession
):
    """删除Agent"""
    service = AgentService(db)
    
    success = await service.delete(agent_id, current_user["sub"])
    
    if not success:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Agent不存在"
        )
```

### 4.2 对话接口

```python
# app/schemas/chat.py
from typing import Optional, List, Dict, Any
from pydantic import BaseModel, Field
from datetime import datetime


class Message(BaseModel):
    """消息"""
    role: str = Field(..., description="角色: user/assistant/system")
    content: str = Field(..., description="消息内容")
    metadata: Optional[Dict[str, Any]] = None


class ChatRequest(BaseModel):
    """对话请求"""
    message: str = Field(..., min_length=1, max_length=10000)
    session_id: Optional[str] = None
    context: Optional[Dict[str, Any]] = None
    stream: bool = Field(default=False, description="是否流式响应")
    webhook_url: Optional[str] = None
    metadata: Optional[Dict[str, Any]] = None
    
    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "message": "你好，请介绍一下你自己",
                    "stream": False
                }
            ]
        }
    }


class ChatResponse(BaseModel):
    """对话响应"""
    message_id: str
    content: str
    session_id: str
    usage: Optional[Dict[str, int]] = None
    model: str
    created_at: datetime
    metadata: Optional[Dict[str, Any]] = None


class StreamChunk(BaseModel):
    """流式响应块"""
    type: str = Field(..., description="chunk/error/done")
    content: Optional[str] = None
    message_id: Optional[str] = None
    error: Optional[str] = None


class ConversationHistory(BaseModel):
    """对话历史"""
    session_id: str
    messages: List[Message]
    created_at: datetime
    updated_at: datetime
```

```python
# app/api/v1/chat.py
from typing import Annotated, AsyncIterator
from fastapi import APIRouter, Depends, BackgroundTasks, Query
from fastapi.responses import StreamingResponse
import json
import asyncio

from app.api.deps import CurrentUser, DbSession
from app.schemas.chat import (
    ChatRequest, ChatResponse, StreamChunk,
    ConversationHistory, Message
)
from app.services.chat_service import ChatService
from app.core.webhook import webhook_manager

router = APIRouter()


@router.post("/agents/{agent_id}/chat", response_model=ChatResponse)
async def chat(
    agent_id: str,
    request: ChatRequest,
    current_user: CurrentUser,
    db: DbSession
):
    """同步对话接口"""
    service = ChatService(db)
    
    response = await service.chat(
        agent_id=agent_id,
        user_id=current_user["sub"],
        message=request.message,
        session_id=request.session_id,
        context=request.context,
        metadata=request.metadata
    )
    
    # 触发Webhook
    if request.webhook_url:
        await webhook_manager.trigger(
            url=request.webhook_url,
            event="chat.completed",
            data={
                "agent_id": agent_id,
                "message_id": response.message_id,
                "content": response.content,
                "session_id": response.session_id
            }
        )
    
    return response


@router.post("/agents/{agent_id}/chat/stream")
async def chat_stream(
    agent_id: str,
    request: ChatRequest,
    current_user: CurrentUser
):
    """流式对话接口"""
    async def generate() -> AsyncIterator[str]:
        service = ChatService(None)
        message_id = f"msg_{uuid4().hex[:8]}"
        
        try:
            async for chunk in service.chat_stream(
                agent_id=agent_id,
                user_id=current_user["sub"],
                message=request.message,
                session_id=request.session_id
            ):
                yield f"data: {json.dumps({'type': 'chunk', 'content': chunk})}\n\n"
            
            yield f"data: {json.dumps({'type': 'done', 'message_id': message_id})}\n\n"
            
        except Exception as e:
            yield f"data: {json.dumps({'type': 'error', 'error': str(e)})}\n\n"
        
        yield "data: [DONE]\n\n"
    
    return StreamingResponse(
        generate(),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
            "X-Accel-Buffering": "no"
        }
    )


@router.post("/agents/{agent_id}/chat/async")
async def chat_async(
    agent_id: str,
    request: ChatRequest,
    background_tasks: BackgroundTasks,
    current_user: CurrentUser,
    db: DbSession
):
    """异步对话接口"""
    task_id = f"task_{uuid4().hex[:12]}"
    
    # 异步处理
    background_tasks.add_task(
        process_chat_async,
        task_id=task_id,
        agent_id=agent_id,
        user_id=current_user["sub"],
        message=request.message,
        webhook_url=request.webhook_url
    )
    
    return {
        "task_id": task_id,
        "status": "processing",
        "message": "任务已提交，请在Webhook中接收结果"
    }


@router.get("/agents/{agent_id}/history", response_model=ConversationHistory)
async def get_conversation_history(
    agent_id: str,
    current_user: CurrentUser,
    db: DbSession,
    session_id: str = Query(..., description="会话ID"),
    limit: int = Query(50, ge=1, le=200),
    before: str = Query(None, description="获取此ID之前的消息")
):
    """获取对话历史"""
    service = ChatService(db)
    
    messages, session = await service.get_history(
        agent_id=agent_id,
        user_id=current_user["sub"],
        session_id=session_id,
        limit=limit,
        before=before
    )
    
    return ConversationHistory(
        session_id=session_id,
        messages=[Message(**m) for m in messages],
        created_at=session.created_at,
        updated_at=session.updated_at
    )
```

### 4.3 服务层实现

```python
# app/services/chat_service.py
from typing import AsyncIterator, List, Dict, Any, Optional
from uuid import uuid4
from datetime import datetime
import json
import logging

from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, and_, or_
from sqlalchemy.orm import selectinload

from app.models.conversation import Conversation, Message as MessageModel
from app.services.llm_service import llm_service
from app.db.redis import get_redis

logger = logging.getLogger(__name__)


class ChatService:
    """对话服务"""
    
    def __init__(self, db: AsyncSession):
        self.db = db
        self.redis = None  # 懒加载
    
    async def chat(
        self,
        agent_id: str,
        user_id: str,
        message: str,
        session_id: str = None,
        context: Dict[str, Any] = None,
        metadata: Dict[str, Any] = None
    ) -> ChatResponse:
        """处理对话"""
        # 获取或创建会话
        session = await self._get_or_create_session(
            agent_id, user_id, session_id
        )
        
        # 保存用户消息
        user_msg = await self._save_message(
            session.id, "user", message
        )
        
        # 构建上下文
        history = await self._get_conversation_history(
            session.id, limit=10
        )
        
        # 调用LLM
        response_text = await llm_service.chat(
            agent_id=agent_id,
            messages=history + [{"role": "user", "content": message}],
            context=context
        )
        
        # 保存助手消息
        assistant_msg = await self._save_message(
            session.id, "assistant", response_text
        )
        
        return ChatResponse(
            message_id=assistant_msg.id,
            content=response_text,
            session_id=session.id,
            usage={"input_tokens": 100, "output_tokens": 50},  # TODO: 实际获取
            model="gpt-4o",
            created_at=datetime.now()
        )
    
    async def chat_stream(
        self,
        agent_id: str,
        user_id: str,
        message: str,
        session_id: str = None
    ) -> AsyncIterator[str]:
        """流式处理对话"""
        session = await self._get_or_create_session(
            agent_id, user_id, session_id
        )
        
        # 保存用户消息
        await self._save_message(session.id, "user", message)
        
        # 获取历史
        history = await self._get_conversation_history(
            session.id, limit=10
        )
        
        # 流式调用
        full_response = ""
        async for chunk in llm_service.chat_stream(
            agent_id=agent_id,
            messages=history + [{"role": "user", "content": message}]
        ):
            full_response += chunk
            yield chunk
        
        # 保存助手消息
        if full_response:
            await self._save_message(
                session.id, "assistant", full_response
            )
    
    async def get_history(
        self,
        agent_id: str,
        user_id: str,
        session_id: str,
        limit: int = 50,
        before: str = None
    ) -> tuple[List[Dict], Conversation]:
        """获取对话历史"""
        # 获取会话
        stmt = select(Conversation).where(
            and_(
                Conversation.id == session_id,
                Conversation.user_id == user_id
            )
        )
        result = await self.db.execute(stmt)
        session = result.scalar_one_or_none()
        
        if not session:
            raise ValueError("会话不存在")
        
        # 获取消息
        stmt = select(MessageModel).where(
            MessageModel.conversation_id == session_id
        ).order_by(MessageModel.created_at.desc()).limit(limit)
        
        result = await self.db.execute(stmt)
        messages = result.scalars().all()
        
        # 转换为字典
        msg_list = [
            {"role": m.role, "content": m.content}
            for m in reversed(messages)
        ]
        
        return msg_list, session
    
    async def _get_or_create_session(
        self,
        agent_id: str,
        user_id: str,
        session_id: str = None
    ) -> Conversation:
        """获取或创建会话"""
        if session_id:
            stmt = select(Conversation).where(Conversation.id == session_id)
            result = await self.db.execute(stmt)
            session = result.scalar_one_or_none()
            if session:
                return session
        
        # 创建新会话
        session = Conversation(
            id=f"sess_{uuid4().hex[:12]}",
            agent_id=agent_id,
            user_id=user_id
        )
        self.db.add(session)
        await self.db.commit()
        await self.db.refresh(session)
        
        return session
    
    async def _save_message(
        self,
        conversation_id: str,
        role: str,
        content: str
    ) -> MessageModel:
        """保存消息"""
        message = MessageModel(
            id=f"msg_{uuid4().hex[:12]}",
            conversation_id=conversation_id,
            role=role,
            content=content
        )
        self.db.add(message)
        await self.db.commit()
        await self.db.refresh(message)
        
        return message
    
    async def _get_conversation_history(
        self,
        conversation_id: str,
        limit: int = 10
    ) -> List[Dict[str, str]]:
        """获取对话历史"""
        stmt = select(MessageModel).where(
            MessageModel.conversation_id == conversation_id
        ).order_by(MessageModel.created_at.desc()).limit(limit)
        
        result = await self.db.execute(stmt)
        messages = result.scalars().all()
        
        return [
            {"role": m.role, "content": m.content}
            for m in reversed(messages)
        ]
```

## 5. 认证与安全

### 5.1 JWT认证实现

```python
# app/core/security.py
from datetime import datetime, timedelta
from typing import Optional, Dict, Any
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import HTTPException, status
from fastapi.security import HTTPAuthorizationCredentials
import hashlib

from app.core.config import settings

# 密码哈希
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")


def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password: str) -> str:
    """哈希密码"""
    return pwd_context.hash(password)


def create_access_token(
    data: Dict[str, Any],
    expires_delta: Optional[timedelta] = None
) -> str:
    """创建访问令牌"""
    to_encode = data.copy()
    
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(
            minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
        )
    
    to_encode.update({
        "exp": expire,
        "iat": datetime.utcnow(),
        "type": "access"
    })
    
    encoded_jwt = jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM
    )
    return encoded_jwt


def create_refresh_token(user_id: str) -> str:
    """创建刷新令牌"""
    expire = datetime.utcnow() + timedelta(days=settings.REFRESH_TOKEN_EXPIRE_DAYS)
    
    to_encode = {
        "sub": user_id,
        "exp": expire,
        "iat": datetime.utcnow(),
        "type": "refresh"
    }
    
    return jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM
    )


def decode_token(token: str) -> Dict[str, Any]:
    """解码令牌"""
    try:
        payload = jwt.decode(
            token,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM]
        )
        return payload
    except JWTError as e:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=f"令牌无效: {str(e)}",
            headers={"WWW-Authenticate": "Bearer"}
        )


def verify_token(token: str, token_type: str = "access") -> Dict[str, Any]:
    """验证令牌"""
    payload = decode_token(token)
    
    if payload.get("type") != token_type:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail=f"令牌类型错误，需要 {token_type}"
        )
    
    return payload


def hash_api_key(api_key: str) -> str:
    """哈希API Key"""
    return hashlib.sha256(api_key.encode()).hexdigest()
```

### 5.2 API Key认证

```python
# app/services/api_key_service.py
from typing import Optional, Dict, Any, List
from uuid import uuid4
import secrets
import hashlib
from datetime import datetime

from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, and_

from app.models.user import APIKey


class APIKeyService:
    """API Key服务"""
    
    def __init__(self, db: AsyncSession):
        self.db = db
    
    async def create_key(
        self,
        user_id: str,
        name: str,
        permissions: List[str],
        expires_days: int = None
    ) -> tuple[str, str]:
        """
        创建新的API Key
        
        Returns:
            (raw_key, hashed_key) - 原始Key只返回这一次
        """
        # 生成Key
        key_id = f"key_{uuid4().hex[:8]}"
        raw_key = f"sk_{secrets.token_urlsafe(32)}"
        key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
        
        # 计算过期时间
        expires_at = None
        if expires_days:
            from datetime import timedelta
            expires_at = datetime.utcnow() + timedelta(days=expires_days)
        
        # 存储
        api_key = APIKey(
            id=key_id,
            user_id=user_id,
            name=name,
            key_hash=key_hash,
            permissions=permissions,
            expires_at=expires_at
        )
        
        self.db.add(api_key)
        await self.db.commit()
        
        return f"{key_id}:{raw_key}", key_hash
    
    async def verify_key(self, raw_key: str) -> Optional[Dict[str, Any]]:
        """验证API Key"""
        # 解析Key
        if ":" not in raw_key:
            return None
        
        key_id, key_value = raw_key.split(":", 1)
        key_hash = hashlib.sha256(key_value.encode()).hexdigest()
        
        # 查询
        stmt = select(APIKey).where(
            and_(
                APIKey.id == key_id,
                APIKey.key_hash == key_hash,
                APIKey.is_active == True
            )
        )
        result = await self.db.execute(stmt)
        api_key = result.scalar_one_or_none()
        
        if not api_key:
            return None
        
        # 检查过期
        if api_key.expires_at and api_key.expires_at < datetime.utcnow():
            return None
        
        return {
            "key_id": api_key.id,
            "user_id": api_key.user_id,
            "name": api_key.name,
            "permissions": api_key.permissions
        }
    
    async def revoke_key(self, key_id: str, user_id: str) -> bool:
        """撤销API Key"""
        stmt = select(APIKey).where(
            and_(
                APIKey.id == key_id,
                APIKey.user_id == user_id
            )
        )
        result = await self.db.execute(stmt)
        api_key = result.scalar_one_or_none()
        
        if not api_key:
            return False
        
        api_key.is_active = False
        await self.db.commit()
        
        return True
    
    async def list_keys(self, user_id: str) -> List[APIKey]:
        """列出用户的API Key"""
        stmt = select(APIKey).where(
            APIKey.user_id == user_id
        ).order_by(APIKey.created_at.desc())
        
        result = await self.db.execute(stmt)
        return result.scalars().all()
```

### 5.3 OAuth 2.0认证

```python
# app/services/oauth_service.py
from typing import Optional, Dict, Any
from urllib.parse import urlencode
import secrets
import hashlib

from app.core.config import settings


class OAuthService:
    """OAuth 2.0服务"""
    
    def __init__(self, provider: str):
        self.provider = provider
        self._config = self._get_provider_config(provider)
    
    def _get_provider_config(self, provider: str) -> Dict[str, Any]:
        """获取提供商配置"""
        providers = {
            "google": {
                "client_id": settings.GOOGLE_CLIENT_ID,
                "client_secret": settings.GOOGLE_CLIENT_SECRET,
                "auth_url": "https://accounts.google.com/o/oauth2/v2/auth",
                "token_url": "https://oauth2.googleapis.com/token",
                "userinfo_url": "https://www.googleapis.com/oauth2/v2/userinfo",
                "scopes": ["openid", "email", "profile"]
            },
            "github": {
                "client_id": settings.GITHUB_CLIENT_ID,
                "client_secret": settings.GITHUB_CLIENT_SECRET,
                "auth_url": "https://github.com/login/oauth/authorize",
                "token_url": "https://github.com/login/oauth/access_token",
                "userinfo_url": "https://api.github.com/user",
                "scopes": ["user:email"]
            }
        }
        return providers.get(provider, {})
    
    def get_authorization_url(self, redirect_uri: str, state: str = None) -> str:
        """获取授权URL"""
        if not self._config:
            raise ValueError(f"Unknown OAuth provider: {self.provider}")
        
        # 生成state
        if not state:
            state = secrets.token_urlsafe(32)
        
        params = {
            "client_id": self._config["client_id"],
            "redirect_uri": redirect_uri,
            "scope": " ".join(self._config["scopes"]),
            "response_type": "code",
            "state": state
        }
        
        return f"{self._config['auth_url']}?{urlencode(params)}"
    
    async def exchange_code_for_token(
        self,
        code: str,
        redirect_uri: str
    ) -> Dict[str, Any]:
        """交换授权码"""
        import httpx
        
        async with httpx.AsyncClient() as client:
            response = await client.post(
                self._config["token_url"],
                data={
                    "client_id": self._config["client_id"],
                    "client_secret": self._config["client_secret"],
                    "code": code,
                    "redirect_uri": redirect_uri,
                    "grant_type": "authorization_code"
                }
            )
            
            if response.status_code != 200:
                raise ValueError(f"Token exchange failed: {response.text}")
            
            return response.json()
    
    async def get_user_info(self, access_token: str) -> Dict[str, Any]:
        """获取用户信息"""
        import httpx
        
        async with httpx.AsyncClient() as client:
            response = await client.get(
                self._config["userinfo_url"],
                headers={"Authorization": f"Bearer {access_token}"}
            )
            
            if response.status_code != 200:
                raise ValueError(f"Get user info failed: {response.text}")
            
            return response.json()
```

## 6. 限流策略

### 6.1 滑动窗口限流

```python
# app/core/rate_limit.py
from fastapi import Request, HTTPException, status
from fastapi.middleware.base import BaseHTTPMiddleware
from typing import Dict, List
import time
import asyncio
from collections import defaultdict

from app.core.config import settings


class RateLimitMiddleware(BaseHTTPMiddleware):
    """限流中间件 - 滑动窗口算法"""
    
    def __init__(
        self,
        app,
        requests_per_minute: int = None,
        requests_per_hour: int = None
    ):
        super().__init__(app)
        self.requests_per_minute = requests_per_minute or settings.RATE_LIMIT_PER_MINUTE
        self.requests_per_hour = requests_per_hour or settings.RATE_LIMIT_PER_HOUR
        
        # 存储请求记录
        self.minute_requests: Dict[str, List[float]] = defaultdict(list)
        self.hour_requests: Dict[str, List[float]] = defaultdict(list)
        
        # 清理任务
        self._cleanup_task = None
    
    async def dispatch(self, request: Request, call_next):
        # 排除健康检查
        if request.url.path in ["/health", "/ready", "/docs", "/openapi.json"]:
            return await call_next(request)
        
        # 获取客户端ID
        client_id = await self._get_client_id(request)
        
        # 检查限流
        if not self._check_rate_limit(client_id):
            raise HTTPException(
                status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                detail="请求过于频繁，请稍后再试",
                headers={
                    "Retry-After": "60",
                    "X-RateLimit-Limit": str(self.requests_per_minute),
                    "X-RateLimit-Remaining": "0"
                }
            )
        
        # 处理请求
        response = await call_next(request)
        
        # 添加限流头
        remaining = self._get_remaining(client_id, "minute")
        response.headers["X-RateLimit-Limit"] = str(self.requests_per_minute)
        response.headers["X-RateLimit-Remaining"] = str(remaining)
        
        return response
    
    async def _get_client_id(self, request: Request) -> str:
        """获取客户端ID"""
        # 优先使用API Key
        api_key = request.headers.get("X-API-Key", "")
        if api_key:
            import hashlib
            return f"api:{hashlib.md5(api_key.encode()).hexdigest()[:16]}"
        
        # 使用用户ID
        if hasattr(request.state, "user"):
            return f"user:{request.state.user.get('sub', 'anonymous')}"
        
        # 使用IP
        return f"ip:{request.client.host}"
    
    def _check_rate_limit(self, client_id: str) -> bool:
        """检查限流"""
        now = time.time()
        
        # 清理过期记录
        self._cleanup(client_id, now)
        
        # 检查分钟限流
        if len(self.minute_requests[client_id]) >= self.requests_per_minute:
            return False
        
        # 检查小时限流
        if len(self.hour_requests[client_id]) >= self.requests_per_hour:
            return False
        
        # 记录请求
        self.minute_requests[client_id].append(now)
        self.hour_requests[client_id].append(now)
        
        return True
    
    def _get_remaining(self, client_id: str, window: str = "minute") -> int:
        """获取剩余请求数"""
        now = time.time()
        self._cleanup(client_id, now)
        
        if window == "minute":
            limit = self.requests_per_minute
            requests = self.minute_requests[client_id]
        else:
            limit = self.requests_per_hour
            requests = self.hour_requests[client_id]
        
        return max(0, limit - len(requests))
    
    def _cleanup(self, client_id: str, now: float):
        """清理过期记录"""
        # 清理1分钟前的记录
        minute_threshold = now - 60
        self.minute_requests[client_id] = [
            t for t in self.minute_requests[client_id] if t > minute_threshold
        ]
        
        # 清理1小时前的记录
        hour_threshold = now - 3600
        self.hour_requests[client_id] = [
            t for t in self.hour_requests[client_id] if t > hour_threshold
        ]
        
        # 清理空记录节省内存
        if not self.minute_requests[client_id]:
            del self.minute_requests[client_id]
        if not self.hour_requests[client_id]:
            del self.hour_requests[client_id]
```

### 6.2 Redis限流

```python
# app/core/rate_limit_redis.py
from typing import Optional
import redis.asyncio as redis
import time
import json


class RedisRateLimiter:
    """基于Redis的限流器"""
    
    def __init__(
        self,
        redis_url: str,
        key_prefix: str = "ratelimit"
    ):
        self.redis = redis.from_url(redis_url)
        self.key_prefix = key_prefix
    
    async def check_rate_limit(
        self,
        identifier: str,
        limit: int,
        window_seconds: int
    ) -> tuple[bool, int, int]:
        """
        检查限流
        
        Returns:
            (is_allowed, remaining, reset_time)
        """
        key = f"{self.key_prefix}:{identifier}"
        now = time.time()
        window_start = now - window_seconds
        
        # 使用Lua脚本保证原子性
        lua_script = """
        local key = KEYS[1]
        local limit = tonumber(ARGV[1])
        local window = tonumber(ARGV[2])
        local now = tonumber(ARGV[3])
        local window_start = tonumber(ARGV[4])
        
        -- 删除窗口外的记录
        redis.call('ZREMRANGEBYSCORE', key, 0, window_start)
        
        -- 获取当前请求数
        local count = redis.call('ZCARD', key)
        
        if count < limit then
            -- 添加新请求
            redis.call('ZADD', key, now, now .. ':' .. math.random())
            redis.call('EXPIRE', key, window)
            return {1, limit - count - 1, window}
        else
            return {0, 0, window}
        end
        """
        
        result = await self.redis.eval(
            lua_script, 1, key, limit, window_seconds, now, window_start
        )
        
        is_allowed = bool(result[0])
        remaining = int(result[1])
        reset_time = int(result[2])
        
        return is_allowed, remaining, reset_time
    
    async def get_usage(
        self,
        identifier: str,
        window_seconds: int
    ) -> int:
        """获取当前使用量"""
        key = f"{self.key_prefix}:{identifier}"
        now = time.time()
        window_start = now - window_seconds
        
        await self.redis.zremrangebyscore(key, 0, window_start)
        return await self.redis.zcard(key)
    
    async def reset(self, identifier: str):
        """重置限流"""
        key = f"{self.key_prefix}:{identifier}"
        await self.redis.delete(key)


# 分布式限流装饰器
def rate_limit(limit: int, window_seconds: int):
    """限流装饰器"""
    def decorator(func):
        async def wrapper(*args, **kwargs):
            # 从请求中获取标识
            request = kwargs.get("request")
            if not request:
                for arg in args:
                    if hasattr(arg, "client"):
                        request = arg
                        break
            
            if not request:
                return await func(*args, **kwargs)
            
            # 获取客户端ID
            client_id = get_client_id(request)
            
            # 检查限流
            limiter = RedisRateLimiter(settings.REDIS_URL)
            is_allowed, remaining, reset_time = await limiter.check_rate_limit(
                client_id, limit, window_seconds
            )
            
            if not is_allowed:
                raise HTTPException(
                    status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                    detail="请求过于频繁",
                    headers={
                        "Retry-After": str(reset_time),
                        "X-RateLimit-Limit": str(limit),
                        "X-RateLimit-Remaining": "0"
                    }
                )
            
            return await func(*args, **kwargs)
        
        return wrapper
    return decorator
```

## 7. Webhook配置

### 7.1 Webhook管理器

```python
# app/core/webhook.py
from typing import Callable, Dict, Any, Optional
import httpx
import asyncio
import hmac
import hashlib
import json
from datetime import datetime
from queue import Queue
import logging

logger = logging.getLogger(__name__)


class WebhookEvent:
    """Webhook事件"""
    
    def __init__(
        self,
        event_type: str,
        data: Dict[str, Any],
        webhook_id: str = None,
        secret: str = None
    ):
        self.event_type = event_type
        self.data = data
        self.webhook_id = webhook_id
        self.secret = secret
        self.timestamp = datetime.utcnow().isoformat()
        self.delivery_id = f"del_{uuid4().hex[:12]}"
    
    def to_payload(self) -> Dict[str, Any]:
        """转换为发送载荷"""
        return {
            "event": self.event_type,
            "delivery_id": self.delivery_id,
            "timestamp": self.timestamp,
            "data": self.data
        }
    
    def generate_signature(self) -> str:
        """生成签名"""
        if not self.secret:
            return ""
        
        payload = json.dumps(self.data, sort_keys=True)
        signature = hmac.new(
            self.secret.encode(),
            payload.encode(),
            hashlib.sha256
        ).hexdigest()
        
        return f"sha256={signature}"


class WebhookManager:
    """Webhook管理器"""
    
    def __init__(self):
        self.handlers: Dict[str, Callable] = {}
        self.retry_config = {
            "max_retries": 3,
            "base_delay": 5,
            "timeout": 30
        }
        self._queue: Queue = Queue()
    
    def register_handler(self, event_type: str, handler: Callable):
        """注册事件处理器"""
        self.handlers[event_type] = handler
    
    async def trigger(
        self,
        url: str,
        event_type: str,
        data: Dict[str, Any],
        secret: str = None
    ) -> bool:
        """触发Webhook"""
        event = WebhookEvent(
            event_type=event_type,
            data=data,
            secret=secret
        )
        
        return await self._send_with_retry(url, event)
    
    async def _send_with_retry(
        self,
        url: str,
        event: WebhookEvent,
        attempt: int = 0
    ) -> bool:
        """带重试发送"""
        headers = {
            "Content-Type": "application/json",
            "X-Webhook-Event": event.event_type,
            "X-Webhook-Delivery": event.delivery_id,
            "X-Webhook-Timestamp": event.timestamp
        }
        
        # 添加签名
        signature = event.generate_signature()
        if signature:
            headers["X-Webhook-Signature"] = signature
        
        payload = event.to_payload()
        
        for attempt in range(self.retry_config["max_retries"]):
            try:
                async with httpx.AsyncClient() as client:
                    response = await client.post(
                        url,
                        json=payload,
                        headers=headers,
                        timeout=self.retry_config["timeout"]
                    )
                
                if response.status_code in [200, 201, 202, 204]:
                    logger.info(f"Webhook delivered: {event.delivery_id}")
                    return True
                
                # 4xx错误不重试
                if 400 <= response.status_code < 500:
                    logger.warning(
                        f"Webhook rejected: {event.delivery_id}, "
                        f"status={response.status_code}"
                    )
                    return False
                
            except (httpx.TimeoutException, httpx.RequestError) as e:
                logger.warning(
                    f"Webhook failed: {event.delivery_id}, "
                    f"attempt={attempt + 1}, error={str(e)}"
                )
            
            # 指数退避重试
            if attempt < self.retry_config["max_retries"] - 1:
                delay = self.retry_config["base_delay"] * (2 ** attempt)
                await asyncio.sleep(delay)
        
        logger.error(f"Webhook failed after {self.retry_config['max_retries']} attempts")
        return False
    
    async def verify_signature(
        self,
        payload: bytes,
        signature: str,
        secret: str
    ) -> bool:
        """验证Webhook签名"""
        if not signature or not secret:
            return False
        
        expected = hmac.new(
            secret.encode(),
            payload,
            hashlib.sha256
        ).hexdigest()
        
        expected_sig = f"sha256={expected}"
        
        return hmac.compare_digest(signature, expected_sig)


# 全局实例
webhook_manager = WebhookManager()


# 常用事件类型
class WebhookEvents:
    """Webhook事件类型常量"""
    CHAT_COMPLETED = "chat.completed"
    CHAT_FAILED = "chat.failed"
    AGENT_CREATED = "agent.created"
    AGENT_UPDATED = "agent.updated"
    AGENT_DELETED = "agent.deleted"
    KNOWLEDGE_UPLOADED = "knowledge.uploaded"
    USER_REGISTERED = "user.registered"
    ERROR_OCCURRED = "error.occurred"
```

### 7.2 Webhook端点

```python
# app/api/v1/webhooks.py
from typing import List
from fastapi import APIRouter, Depends, HTTPException, BackgroundTasks, Request

from app.api.deps import CurrentUser, DbSession
from app.schemas.webhook import (
    WebhookCreate, WebhookUpdate, WebhookResponse, WebhookTest
)
from app.services.webhook_service import WebhookService
from app.core.webhook import webhook_manager

router = APIRouter()


@router.post("/webhooks", response_model=WebhookResponse, status_code=201)
async def create_webhook(
    data: WebhookCreate,
    current_user: CurrentUser,
    db: DbSession
):
    """创建Webhook"""
    service = WebhookService(db)
    
    webhook = await service.create(
        user_id=current_user["sub"],
        data=data
    )
    
    return WebhookResponse.model_validate(webhook)


@router.get("/webhooks", response_model=List[WebhookResponse])
async def list_webhooks(
    current_user: CurrentUser,
    db: DbSession
):
    """列出Webhook"""
    service = WebhookService(db)
    webhooks = await service.list_by_user(current_user["sub"])
    return [WebhookResponse.model_validate(w) for w in webhooks]


@router.post("/webhooks/{webhook_id}/test")
async def test_webhook(
    webhook_id: str,
    background_tasks: BackgroundTasks,
    current_user: CurrentUser,
    db: DbSession
):
    """测试Webhook"""
    service = WebhookService(db)
    webhook = await service.get_by_id(webhook_id, current_user["sub"])
    
    if not webhook:
        raise HTTPException(status_code=404, detail="Webhook不存在")
    
    # 发送测试事件
    background_tasks.add_task(
        webhook_manager.trigger,
        url=webhook.url,
        event_type="webhook.test",
        data={"message": "这是一条测试消息"},
        secret=webhook.secret
    )
    
    return {"status": "test_sent"}


@router.post("/webhooks/{webhook_id}/verify")
async def verify_webhook(
    webhook_id: str,
    request: Request,
    current_user: CurrentUser,
    db: DbSession
):
    """验证Webhook签名"""
    body = await request.body()
    signature = request.headers.get("X-Webhook-Signature", "")
    
    service = WebhookService(db)
    webhook = await service.get_by_id(webhook_id, current_user["sub"])
    
    if not webhook or not webhook.secret:
        raise HTTPException(status_code=404, detail="Webhook不存在")
    
    is_valid = await webhook_manager.verify_signature(
        body, signature, webhook.secret
    )
    
    return {"valid": is_valid}


@router.delete("/webhooks/{webhook_id}", status_code=204)
async def delete_webhook(
    webhook_id: str,
    current_user: CurrentUser,
    db: DbSession
):
    """删除Webhook"""
    service = WebhookService(db)
    success = await service.delete(webhook_id, current_user["sub"])
    
    if not success:
        raise HTTPException(status_code=404, detail="Webhook不存在")
```

## 8. 灰度发布

### 8.1 灰度路由

```python
# app/core/canary.py
from typing import Callable, Dict, Any, Optional
from fastapi import Request, HTTPException
import hashlib
import random
import time

from app.services.llm_service import llm_service_v1, llm_service_v2


class CanaryRouter:
    """灰度路由"""
    
    def __init__(self):
        self.rules: list[Dict[str, Any]] = []
        self._cache: Dict[str, tuple] = {}
        self._cache_ttl = 60  # 缓存1分钟
    
    def add_rule(
        self,
        name: str,
        condition: Callable[[Dict], bool],
        handler: Callable,
        weight: float = 0.0,
        description: str = ""
    ):
        """添加灰度规则"""
        self.rules.append({
            "name": name,
            "condition": condition,
            "handler": handler,
            "weight": weight,
            "description": description
        })
    
    async def route(
        self,
        request: Request,
        default_handler: Callable,
        context: Dict[str, Any] = None
    ) -> Any:
        """路由请求"""
        context = context or {}
        context["request"] = request
        
        # 按权重灰度
        for rule in self.rules:
            if rule["weight"] > 0:
                if self._check_weight(request, rule["name"], rule["weight"]):
                    return await rule["handler"](context)
        
        # 按条件灰度
        for rule in self.rules:
            if rule["condition"](context):
                return await rule["handler"](context)
        
        return await default_handler(context)
    
    def _check_weight(self, request: Request, rule_name: str, weight: float) -> bool:
        """检查权重 - 使用一致性哈希"""
        # 使用用户ID或IP做一致性哈希
        user_id = self._get_user_identifier(request)
        
        # 生成哈希值
        hash_input = f"{rule_name}:{user_id}"
        hash_value = int(
            hashlib.md5(hash_input.encode()).hexdigest(),
            16
        )
        
        # 判断是否在灰度范围内
        percentage = (hash_value % 10000) / 10000 * 100
        return percentage < weight * 100
    
    def _get_user_identifier(self, request: Request) -> str:
        """获取用户标识"""
        # 优先使用用户ID
        if hasattr(request.state, "user") and request.state.user:
            return f"user:{request.state.user.get('sub', '')}"
        
        # 使用API Key
        api_key = request.headers.get("X-API-Key", "")
        if api_key:
            return f"api:{hashlib.md5(api_key.encode()).hexdigest()[:16]}"
        
        # 使用IP
        return f"ip:{request.client.host}"


# 创建灰度路由实例
canary_router = CanaryRouter()

# 注册灰度规则
canary_router.add_rule(
    name="new_llm_model",
    condition=lambda ctx: ctx.get("force_version") == "v2",
    handler=lambda ctx: process_with_v2(ctx),
    weight=0.1,  # 10%流量
    description="新模型灰度"
)

canary_router.add_rule(
    name="premium_users",
    condition=lambda ctx: ctx.get("user_tier") == "premium",
    handler=lambda ctx: process_with_v2(ctx),
    weight=1.0,  # 100%
    description="付费用户优先体验"
)


async def process_with_v1(context: Dict) -> Dict:
    """使用v1处理"""
    request = context["request"]
    # 使用v1 LLM服务
    return {"model": "v1", "version": "1.0.0"}


async def process_with_v2(context: Dict) -> Dict:
    """使用v2处理"""
    request = context["request"]
    # 使用v2 LLM服务
    return {"model": "v2", "version": "2.0.0"}


# 中间件方式使用灰度
class CanaryMiddleware:
    """灰度中间件"""
    
    async def __call__(self, request: Request, call_next):
        # 检查是否需要灰度
        route_key = f"{request.url.path}:{request.method}"
        
        if route_key in self._canary_routes:
            context = await self._build_context(request)
            return await canary_router.route(
                request,
                lambda ctx: call_next(ctx["request"]),
                context
            )
        
        return await call_next(request)
    
    async def _build_context(self, request: Request) -> Dict:
        """构建上下文"""
        return {
            "path": request.url.path,
            "method": request.method,
            "user": getattr(request.state, "user", None),
            "query_params": dict(request.query_params)
        }
```

### 8.2 A/B测试

```python
# app/core/ab_test.py
from typing import Dict, Any, Optional, Callable
from dataclasses import dataclass
from datetime import datetime
import hashlib
import json


@dataclass
class ABTest:
    """A/B测试配置"""
    name: str
    variants: Dict[str, float]  # variant_name -> weight
    condition: Optional[Callable] = None
    start_time: datetime = None
    end_time: datetime = None


class ABTestManager:
    """A/B测试管理器"""
    
    def __init__(self):
        self.tests: Dict[str, ABTest] = {}
        self._assignments: Dict[str, Dict[str, str]] = {}  # user_id -> {test_name: variant}
    
    def register_test(self, test: ABTest):
        """注册测试"""
        # 验证权重
        total_weight = sum(test.variants.values())
        if abs(total_weight - 1.0) > 0.001:
            raise ValueError(f"Test weights must sum to 1.0, got {total_weight}")
        
        self.tests[test.name] = test
    
    def get_variant(
        self,
        test_name: str,
        user_id: str,
        force_variant: str = None
    ) -> Optional[str]:
        """获取用户对应的变体"""
        test = self.tests.get(test_name)
        if not test:
            return None
        
        # 检查时间范围
        now = datetime.utcnow()
        if test.start_time and now < test.start_time:
            return None
        if test.end_time and now > test.end_time:
            return None
        
        # 检查条件
        if test.condition and not test.condition(user_id):
            return None
        
        # 检查缓存
        cache_key = f"{test_name}:{user_id}"
        if cache_key in self._assignments:
            return self._assignments[cache_key].get(test_name)
        
        # 计算变体
        variant = self._calculate_variant(test, user_id)
        
        # 缓存
        if test_name not in self._assignments:
            self._assignments[test_name] = {}
        self._assignments[test_name][user_id] = variant
        
        return variant
    
    def _calculate_variant(self, test: ABTest, user_id: str) -> str:
        """计算变体"""
        # 使用一致性哈希确保同一用户总是分配到同一变体
        hash_input = f"{test.name}:{user_id}"
        hash_value = int(
            hashlib.md5(hash_input.encode()).hexdigest(),
            16
        )
        
        percentage = (hash_value % 10000) / 10000 * 100
        
        # 按权重分配
        cumulative = 0
        for variant, weight in test.variants.items():
            cumulative += weight * 100
            if percentage < cumulative:
                return variant
        
        # 默认返回最后一个
        return list(test.variants.keys())[-1]
    
    def track_event(
        self,
        test_name: str,
        user_id: str,
        event_name: str,
        properties: Dict[str, Any] = None
    ):
        """追踪事件"""
        variant = self.get_variant(test_name, user_id)
        if not variant:
            return
        
        # 发送到分析系统
        event = {
            "test_name": test_name,
            "variant": variant,
            "user_id": user_id,
            "event": event_name,
            "properties": properties or {},
            "timestamp": datetime.utcnow().isoformat()
        }
        
        # TODO: 发送到数据分析服务
        print(f"AB Test Event: {json.dumps(event)}")


# 使用示例
ab_manager = ABTestManager()

ab_manager.register_test(ABTest(
    name="new_chat_ui",
    variants={
        "control": 0.5,  # 50%用户
        "variant_a": 0.3,  # 30%用户
        "variant_b": 0.2   # 20%用户
    }
))

# 在请求中使用
async def chat_endpoint(request: Request):
    user_id = request.state.user["sub"]
    
    variant = ab_manager.get_variant("new_chat_ui", user_id)
    
    if variant == "control":
        return await render_normal_chat()
    elif variant == "variant_a":
        return await render_new_chat_a()
    else:
        return await render_new_chat_b()
```

## 9. API文档与测试

### 9.1 OpenAPI扩展

```python
# app/docs.py
from fastapi.openapi.utils import get_openapi
from app.core.config import settings


def custom_openapi():
    """自定义OpenAPI文档"""
    if app.openapi_schema:
        return app.openapi_schema
    
    openapi_schema = get_openapi(
        title=settings.APP_NAME,
        version=settings.APP_VERSION,
        description="""
        AI Agent API - 让AI能力触手可及
        
        ## 认证
        
        本API支持两种认证方式：
        
        1. **JWT Token**: 在请求头中添加 `Authorization: Bearer <token>`
        2. **API Key**: 在请求头中添加 `X-API-Key: <key>`
        
        ## 限流
        
        默认限流策略：
        - 每分钟60次请求
        - 每小时1000次请求
        
        ## 错误码
        
        | 错误码 | 说明 |
        |--------|------|
        | 400 | 请求参数错误 |
        | 401 | 未认证或认证失败 |
        | 403 | 无权限访问 |
        | 404 | 资源不存在 |
        | 429 | 请求过于频繁 |
        | 500 | 服务器内部错误 |
        """,
        routes=app.routes,
    )
    
    # 添加服务器信息
    openapi_schema["servers"] = [
        {"url": "https://api.example.com", "description": "生产环境"},
        {"url": "https://api-staging.example.com", "description": "预发环境"},
        {"url": "http://localhost:8000", "description": "本地开发"}
    ]
    
    # 添加标签
    openapi_schema["tags"] = [
        {
            "name": "Agent管理",
            "description": "Agent的创建、更新、删除和查询"
        },
        {
            "name": "对话接口",
            "description": "与Agent进行对话，支持同步、流式和异步模式"
        },
        {
            "name": "知识库",
            "description": "知识库的创建和文档管理"
        },
        {
            "name": "Webhook",
            "description": "Webhook配置和事件通知"
        },
        {
            "name": "系统",
            "description": "系统健康检查和状态查询"
        }
    ]
    
    app.openapi_schema = openapi_schema
    return openapi_schema
```

### 9.2 请求示例

```python
# app/schemas/examples.py
from app.schemas.agent import AgentCreate, AgentConfig
from app.schemas.chat import ChatRequest

# Agent创建示例
AGENT_CREATE_EXAMPLE = {
    "name": "我的AI助手",
    "description": "一个智能助手，可以回答问题和提供帮助",
    "agent_type": "chat",
    "config": {
        "model": "gpt-4o",
        "temperature": 0.7,
        "max_tokens": 2000,
        "system_prompt": "你是一个友好的AI助手"
    }
}

# 对话请求示例
CHAT_REQUEST_EXAMPLE = {
    "message": "你好，请介绍一下你自己",
    "session_id": "sess_abc123",
    "stream": False
}

# 流式对话请求示例
STREAM_CHAT_REQUEST_EXAMPLE = {
    "message": "写一首关于春天的诗",
    "session_id": "sess_abc123",
    "stream": True
}
```

## 10. 总结

### API设计检查清单

| 检查项 | 说明 |
|--------|------|
| ✅ RESTful规范 | 使用正确的HTTP方法和URL结构 |
| ✅ 统一响应格式 | 成功/错误有固定格式 |
| ✅ 认证授权 | JWT/API Key/OAuth |
| ✅ 限流保护 | 防止滥用 |
| ✅ 输入验证 | Pydantic模型验证 |
| ✅ 错误处理 | 友好的错误信息 |
| ✅ 日志记录 | 请求追踪 |
| ✅ API文档 | OpenAPI/Swagger |
| ✅ 版本管理 | 支持多版本 |
| ✅ 灰度发布 | 平滑升级 |

### 常见问题

| 问题 | 解决方案 |
|------|----------|
| 跨域问题 | 配置CORS中间件 |
| Token过期 | 实现刷新Token机制 |
| 限流误杀 | 设置合理的阈值+白名单 |
| 流式中断 | 前端处理重连 |
| Webhook失败 | 重试机制+失败告警 |

---

## 相关资源

- [[AI应用生产部署]] - 企业级部署最佳实践
- [[工作流设计模式]] - 工作流设计原则
- [[多Agent系统设计]] - 多智能体架构
- [[插件开发]] - AI平台插件开发

---

*本文档由归愚知识系统生成 last updated: 2026-04-24*
