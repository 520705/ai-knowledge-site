---
title: OpenClaw多平台集成
date: 2026-04-18
tags:
  - openclaw
  - 多平台
  - telegram
  - discord
  - whatsapp
  - slack
  - matrix
  - 邮件集成
  - webhook
  - 跨平台通讯
categories:
  - 人工智能工具实操/小龙虾生态/多平台集成
alias: OpenClaw Multi-Platform Integration
---

## 关键词

| 关键词 | 说明 |
|--------|------|
| Telegram Bot | 即时通讯平台集成 |
| Discord Webhook | 游戏社区消息推送 |
| WhatsApp Business | 商业通讯解决方案 |
| Slack App | 企业协作工具 |
| Matrix Protocol | 去中心化通讯协议 |
| SMTP/IMAP | 邮件收发协议 |
| WebSocket | 实时双向通信 |
| REST API | 标准化接口设计 |
| Bot Framework | 机器人框架核心 |
| Event Adapter | 事件适配器模式 |

---

## 概述

OpenClaw 的多平台集成架构是其作为通用 Agent 执行环境最核心的能力之一。通过统一的抽象层，OpenClaw 可以同时连接 Telegram、Discord、WhatsApp、Slack、Matrix、邮件系统以及任何支持 Webhook 的自定义平台，实现真正的全渠道消息统一处理。

> [!note] 核心设计理念
> OpenClaw 采用「事件驱动 + 适配器模式」的架构，所有平台消息都被标准化为统一的 `AgentMessage` 对象，无论来源如何，后端的 Agent 逻辑都无需关心具体平台差异。

---

## 平台适配器架构

### 核心抽象层

```python
# openclaw/platforms/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import Optional, Dict, Any, List
from datetime import datetime
from enum import Enum

class MessageType(Enum):
    TEXT = "text"
    IMAGE = "image"
    AUDIO = "audio"
    VIDEO = "video"
    DOCUMENT = "document"
    LOCATION = "location"
    CONTACT = "contact"

@dataclass
class PlatformMessage:
    """统一消息格式"""
    message_id: str
    platform: str
    chat_id: str
    user_id: str
    message_type: MessageType
    content: str
    raw_payload: Dict[str, Any]
    timestamp: datetime = field(default_factory=datetime.now)
    metadata: Dict[str, Any] = field(default_factory=dict)
    
    def to_agent_input(self) -> "AgentInput":
        """转换为Agent可处理的标准化输入"""
        return AgentInput(
            text=self.content,
            source_platform=self.platform,
            user_id=self.user_id,
            chat_id=self.chat_id,
            message_id=self.message_id,
            attachments=self._extract_attachments()
        )
    
    def _extract_attachments(self) -> List["Attachment"]:
        # 平台特定附件提取逻辑
        pass

class PlatformAdapter(ABC):
    """平台适配器基类"""
    
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.message_queue: List[PlatformMessage] = []
        self._running = False
    
    @abstractmethod
    async def connect(self) -> bool:
        """建立平台连接"""
        pass
    
    @abstractmethod
    async def disconnect(self) -> None:
        """断开平台连接"""
        pass
    
    @abstractmethod
    async def send_message(
        self, 
        chat_id: str, 
        content: str,
        message_type: MessageType = MessageType.TEXT,
        **kwargs
    ) -> str:
        """发送消息，返回消息ID"""
        pass
    
    @abstractmethod
    async def edit_message(
        self, 
        chat_id: str, 
        message_id: str, 
        new_content: str
    ) -> bool:
        """编辑已发送的消息"""
        pass
    
    @abstractmethod
    async def delete_message(
        self, 
        chat_id: str, 
        message_id: str
    ) -> bool:
        """删除消息"""
        pass
    
    async def handle_webhook(self, payload: Dict[str, Any]) -> PlatformMessage:
        """处理接收到的Webhook数据"""
        message = self._parse_payload(payload)
        return message
    
    @abstractmethod
    def _parse_payload(self, payload: Dict[str, Any]) -> PlatformMessage:
        """子类实现：解析平台特定payload"""
        pass
```

### 适配器注册中心

```python
# openclaw/platforms/registry.py
from typing import Dict, Type, Optional
from .base import PlatformAdapter

class PlatformRegistry:
    """平台适配器注册中心"""
    
    _adapters: Dict[str, Type[PlatformAdapter]] = {}
    
    @classmethod
    def register(cls, platform_name: str):
        """装饰器：注册平台适配器"""
        def decorator(adapter_class: Type[PlatformAdapter]):
            cls._adapters[platform_name.lower()] = adapter_class
            return adapter_class
        return decorator
    
    @classmethod
    def get_adapter(
        cls, 
        platform_name: str, 
        config: Dict[str, Any]
    ) -> Optional[PlatformAdapter]:
        """获取适配器实例"""
        adapter_class = cls._adapters.get(platform_name.lower())
        if adapter_class:
            return adapter_class(config)
        return None
    
    @classmethod
    def list_platforms(cls) -> list:
        """列出所有已注册平台"""
        return list(cls._adapters.keys())
```

---

## Telegram 集成详解

### Bot API 配置

```yaml
# config/telegram.yaml
telegram:
  bot_token: "${TELEGRAM_BOT_TOKEN}"
  api_id: "${TELEGRAM_API_ID}"
  api_hash: "${TELEGRAM_API_HASH}"
  
  # 消息处理
  message_queue_size: 1000
  max_message_length: 4096
  parse_mode: "HTML"  # HTML / MarkdownV2
  
  # 功能开关
  features:
    inline_queries: true
    commands: true
    callbacks: true
    media_group: true
    
  # 群组配置
  group_settings:
    allow_group_messages: true
    admin_only_commands: ["shutdown", "reload"]
    bot_commands:
      - ["start", "开始与机器人对话"]
      - ["help", "获取帮助信息"]
      - ["status", "查看系统状态"]
```

### Telegram 适配器实现

```python
# openclaw/platforms/telegram_adapter.py
from aiogram import Bot, Dispatcher, types
from aiogram.filters import Command, StateFilter
from aiogram.fsm.context import FSMContext
from aiogram.fsm.state import State, StatesGroup
from .base import PlatformAdapter, PlatformMessage, MessageType
from .registry import PlatformRegistry

@PlatformRegistry.register("telegram")
class TelegramAdapter(PlatformAdapter):
    """Telegram 平台适配器"""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.bot = Bot(token=config["bot_token"])
        self.dp = Dispatcher()
        self._setup_handlers()
    
    def _setup_handlers(self):
        """注册消息处理器"""
        
        @self.dp.message(Command("start"))
        async def cmd_start(message: types.Message):
            await message.answer(
                "欢迎使用 OpenClaw！我是您的 AI 助手。\n"
                "直接发送消息开始对话。"
            )
        
        @self.dp.message()
        async def handle_message(message: types.Message):
            platform_msg = self._parse_telegram_message(message)
            await self._route_to_agent(platform_msg)
        
        @self.dp.callback_query()
        async def handle_callback(callback: types.CallbackQuery):
            # 处理按钮回调
            await callback.answer()
    
    async def connect(self) -> bool:
        try:
            await self.dp.start_polling(self.bot)
            self._running = True
            return True
        except Exception as e:
            logger.error(f"Telegram connection failed: {e}")
            return False
    
    async def send_message(
        self, 
        chat_id: str, 
        content: str,
        message_type: MessageType = MessageType.TEXT,
        **kwargs
    ) -> str:
        parse_mode = kwargs.get("parse_mode", "HTML")
        
        if message_type == MessageType.TEXT:
            msg = await self.bot.send_message(
                chat_id, 
                content,
                parse_mode=parse_mode
            )
        elif message_type == MessageType.IMAGE:
            msg = await self.bot.send_photo(
                chat_id,
                photo=content,  # URL 或 file_id
                caption=kwargs.get("caption", "")
            )
        elif message_type == MessageType.DOCUMENT:
            msg = await self.bot.send_document(
                chat_id,
                document=content,
                caption=kwargs.get("caption", "")
            )
        
        return str(msg.message_id)
    
    def _parse_telegram_message(self, message: types.Message) -> PlatformMessage:
        """解析 Telegram 消息为统一格式"""
        
        msg_type = MessageType.TEXT
        content = message.text or message.caption or ""
        
        if message.photo:
            msg_type = MessageType.IMAGE
            content = message.photo[-1].file_id
        elif message.document:
            msg_type = MessageType.DOCUMENT
            content = message.document.file_id
        elif message.voice:
            msg_type = MessageType.AUDIO
            content = message.voice.file_id
        elif message.video:
            msg_type = MessageType.VIDEO
            content = message.video.file_id
        
        return PlatformMessage(
            message_id=str(message.message_id),
            platform="telegram",
            chat_id=str(message.chat.id),
            user_id=str(message.from_user.id),
            message_type=msg_type,
            content=content,
            raw_payload=message.model_dump(),
            metadata={
                "first_name": message.from_user.first_name,
                "username": message.from_user.username,
                "is_bot": message.from_user.is_bot
            }
        )
```

> [!tip] Telegram 特定功能
> - 使用 `reply_markup` 参数添加内联键盘按钮
> - 通过 `parse_mode` 控制文本格式（HTML 更稳定）
> - 使用 `disable_web_page_preview` 控制链接预览

---

## Discord 集成详解

### Webhook vs Bot 对比

| 方式 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| Webhook | 简单，无需长期连接 | 只读，无法响应交互 | 单向通知推送 |
| Bot | 完整功能，双向通信 | 需要申请 Bot Token | 完整对话交互 |

### Discord 配置

```yaml
# config/discord.yaml
discord:
  # Webhook 模式
  webhooks:
    - name: "general-alerts"
      url: "${DISCORD_WEBHOOK_GENERAL}"
      channel_id: "123456789"
    - name: "error-logs"
      url: "${DISCORD_WEBHOOK_ERROR}"
      filter:
        level: ">=WARNING"
  
  # Bot 模式
  bot:
    token: "${DISCORD_BOT_TOKEN}"
    intents:
      - GUILDS
      - GUILD_MESSAGES
      - MESSAGE_CONTENT
      - DIRECT_MESSAGES
    
    # Slash Commands
    commands:
      - name: "ask"
        description: "向 AI 提问"
        options:
          - name: "question"
            type: 3  # STRING
            required: true
```

### Discord Embed 消息

```python
# openclaw/platforms/discord_embeds.py
import discord
from discord import Embed

class DiscordEmbedBuilder:
    """Discord 富文本消息构建器"""
    
    @staticmethod
    def create_agent_response(
        user: str,
        response: str,
        metadata: Dict = None
    ) -> Embed:
        """创建 Agent 响应嵌入消息"""
        
        embed = Embed(
            title="🤖 OpenClaw 响应",
            color=0x5865F2,  # Discord Blurple
            timestamp=datetime.now()
        )
        
        embed.add_field(
            name="👤 用户",
            value=f"**{user}**",
            inline=True
        )
        
        embed.add_field(
            name="📝 响应",
            value=response[:1024] if len(response) > 1024 else response,
            inline=False
        )
        
        if metadata:
            embed.add_field(
                name="📊 元数据",
                value=f"模型: {metadata.get('model', 'N/A')}\n"
                      f"Token: {metadata.get('tokens', 'N/A')}",
                inline=True
            )
        
        embed.set_footer(text="OpenClaw Agent")
        
        return embed
    
    @staticmethod
    def create_error_embed(error: str, context: str = None) -> Embed:
        """创建错误消息嵌入"""
        
        embed = Embed(
            title="❌ 执行错误",
            description=error,
            color=0xED4245  # Discord Red
        )
        
        if context:
            embed.add_field(
                name="📋 上下文",
                value=context[:1024],
                inline=False
            )
        
        return embed
```

---

## Slack 集成详解

### Slack App 配置

```yaml
# config/slack.yaml
slack:
  app_token: "${SLACK_APP_TOKEN}"      # xapp-xxx 格式
  bot_token: "${SLACK_BOT_TOKEN}"      # xoxb-xxx 格式
  signing_secret: "${SLACK_SIGNING_SECRET}"
  
  # Socket Mode (推荐)
  socket_mode:
    enabled: true
    auto_reconnect: true
    
  # 事件订阅
  events:
    app_mention: true
    message.channels: true
    message.im: true  # Direct Message
    
  # Interactivity
  interactive:
    shortcuts: true
    buttons: true
    modals: true
    
  # 命令
  slash_commands:
    - command: "/ask"
      description: "向 OpenClaw 提问"
      usage_hint: "/ask <你的问题>"
    - command: "/status"
      description: "查看系统状态"
```

### Slack 消息格式

```python
# openclaw/platforms/slack_formatters.py
from slack_sdk.models.blocks import (
    SectionBlock, ButtonElement, DividerBlock,
    InputBlock, PlainTextObject, MarkdownTextObject
)

class SlackMessageBuilder:
    """Slack 消息构建工具"""
    
    @staticmethod
    def create_threaded_response(
        question: str,
        answer: str,
        thread_ts: str = None
    ) -> Dict:
        """创建线程化响应消息"""
        
        blocks = [
            SectionBlock(
                text=MarkdownTextObject(
                    text=f"*👤 你:*\n{question}"
                )
            ),
            DividerBlock(),
            SectionBlock(
                text=MarkdownTextObject(
                    text=f"*🤖 OpenClaw:*\n{answer}"
                )
            ),
            SectionBlock(
                text=MarkdownTextObject(
                    text="---"
                ),
                accessory=ButtonElement(
                    text="↗️ 新对话",
                    action_id="new_conversation"
                )
            )
        ]
        
        return {
            "blocks": blocks,
            "thread_ts": thread_ts  # 保持线程连续性
        }
    
    @staticmethod
    def create_modal(title: str, blocks: List) -> Dict:
        """创建交互式 Modal"""
        
        return {
            "type": "modal",
            "title": PlainTextObject(text=title),
            "blocks": blocks,
            "close": PlainTextObject(text="关闭"),
            "submit": PlainTextObject(text="提交")
        }
```

---

## Matrix 协议集成

Matrix 是一个去中心化开源通讯协议，OpenClaw 通过 `matrix-nio` 库实现集成。

```yaml
# config/matrix.yaml
matrix:
  homeserver: "https://matrix.org"
  user_id: "@openclaw:matrix.org"
  access_token: "${MATRIX_ACCESS_TOKEN}"
  
  # 房间管理
  auto_join: true
  rooms:
    - "#openclaw-general:matrix.org"
    - "!random:matrix.org"
  
  # 同步设置
  sync:
    timeout_ms: 30000
    full_state: false
    filter_id: null
```

```python
# openclaw/platforms/matrix_adapter.py
import nio
from nio import AsyncClient, RoomMessageText

@PlatformRegistry.register("matrix")
class MatrixAdapter(PlatformAdapter):
    """Matrix 协议适配器"""
    
    def __init__(self, config: Dict[str, Any]):
        super().__init__(config)
        self.homeserver = config["homeserver"]
        self.user_id = config["user_id"]
        self.access_token = config["access_token"]
        self.client = AsyncClient(self.homeserver)
    
    async def connect(self) -> bool:
        self.client.access_token = self.access_token
        await self.client.sync()
        self._running = True
        return True
    
    async def send_message(
        self, 
        room_id: str, 
        content: str,
        **kwargs
    ) -> str:
        response = await self.client.room_send(
            room_id=room_id,
            message_type="m.room.message",
            content={
                "msgtype": "m.text",
                "body": content
            }
        )
        return response.event_id
```

---

## 邮件系统集成

### SMTP/IMAP 配置

```yaml
# config/email.yaml
email:
  smtp:
    host: "smtp.gmail.com"
    port: 587
    username: "${SMTP_USERNAME}"
    password: "${SMTP_PASSWORD}"
    starttls: true
    
  imap:
    host: "imap.gmail.com"
    port: 993
    username: "${IMAP_USERNAME}"
    password: "${IMAP_PASSWORD}"
    
  # 邮件处理规则
  rules:
    - name: "ai-commands"
      subject_pattern: "[OpenClaw] *"
      action: "process"
      priority: 1
```

```python
# openclaw/platforms/email_adapter.py
import smtplib
import imaplib
from email.mime.text import MIMEText
from email.parser import Parser
import email

@PlatformRegistry.register("email")
class EmailAdapter(PlatformAdapter):
    """邮件系统适配器"""
    
    async def connect(self) -> bool:
        self.smtp = smtplib.SMTP(self.config["smtp"]["host"])
        self.smtp.starttls()
        self.smtp.login(
            self.config["smtp"]["username"],
            self.config["smtp"]["password"]
        )
        
        self.imap = imaplib.IMAP4_SSL(self.config["imap"]["host"])
        self.imap.login(
            self.config["imap"]["username"],
            self.config["imap"]["password"]
        )
        return True
    
    async def send_message(
        self,
        to: str,
        subject: str,
        body: str,
        **kwargs
    ) -> str:
        msg = MIMEText(body, "html")
        msg["From"] = self.config["smtp"]["username"]
        msg["To"] = to
        msg["Subject"] = subject
        
        self.smtp.send_message(msg)
        return f"sent_to_{to}"
    
    def check_inbox(self, unread_only: bool = True) -> List[PlatformMessage]:
        """检查收件箱新邮件"""
        self.imap.select("INBOX")
        status, messages = self.imap.search(
            None, 
            "UNSEEN" if unread_only else "ALL"
        )
        
        messages = messages[0].split()
        results = []
        
        for num in messages[-50:]:  # 最近50封
            status, data = self.imap.fetch(num, "(RFC822)")
            raw_email = data[0][1]
            email_msg = Parser().parsestr(raw_email.decode())
            
            platform_msg = PlatformMessage(
                message_id=email_msg["Message-ID"],
                platform="email",
                chat_id=email_msg["From"],
                user_id=email_msg["From"],
                message_type=MessageType.TEXT,
                content=email_msg.get_payload(),
                raw_payload={"subject": email_msg["Subject"]}
            )
            results.append(platform_msg)
        
        return results
```

---

## 自定义 Webhook 集成

OpenClaw 支持通过 Webhook 方式接入任何 HTTP-based 平台。

```yaml
# config/custom_webhooks.yaml
webhooks:
  # GitHub Webhook
  github:
    endpoint: "/webhooks/github"
    secret: "${GITHUB_WEBHOOK_SECRET}"
    events:
      - "push"
      - "pull_request"
      - "issues"
      
  # 自定义 HTTP 源
  custom_http:
    endpoint: "/webhooks/custom"
    auth:
      type: "bearer"  # bearer / basic / api_key
      token: "${CUSTOM_WEBHOOK_TOKEN}"
    rate_limit:
      requests: 100
      window_seconds: 60
```

### Webhook 处理器

```python
# openclaw/platforms/webhook_handler.py
from fastapi import FastAPI, Request, HTTPException
from fastapi.responses import JSONResponse
import hmac
import hashlib
import time

class WebhookHandler:
    """通用 Webhook 处理器"""
    
    def __init__(self, app: FastAPI):
        self.app = app
        self.handlers: Dict[str, Callable] = {}
        self._register_routes()
    
    def _register_routes(self):
        """注册 Webhook 路由"""
        
        @self.app.post("/webhooks/{provider}")
        async def handle_webhook(
            provider: str,
            request: Request,
            x_hub_signature: str = None,
            x_hub_signature_256: str = None,
            x_slack_signature: str = None
        ):
            # 验证签名
            body = await request.body()
            if not await self._verify_signature(
                provider, body,
                x_hub_signature or x_hub_signature_256 or x_slack_signature
            ):
                raise HTTPException(status_code=403, detail="Invalid signature")
            
            # 解析 payload
            content_type = request.headers.get("content-type", "")
            if "application/json" in content_type:
                payload = await request.json()
            else:
                payload = dict(await request.form())
            
            # 分发到对应处理器
            handler = self.handlers.get(provider)
            if handler:
                result = await handler(payload)
                return JSONResponse(content=result)
            
            return JSONResponse({"status": "received"})
    
    async def _verify_signature(
        self,
        provider: str,
        body: bytes,
        signature: str
    ) -> bool:
        """验证 Webhook 签名"""
        
        if provider == "github":
            secret = self.config["webhooks"]["github"]["secret"].encode()
            expected = "sha1=" + hmac.new(
                secret, body, hashlib.sha1
            ).hexdigest()
            return hmac.compare_digest(expected, signature)
        
        elif provider == "slack":
            timestamp = request.headers.get("X-Slack-Request-Timestamp")
            if abs(time.time() - int(timestamp)) > 60 * 5:
                return False  # 防重放攻击
            sig_basestring = f"v0:{timestamp}:{body.decode()}"
            expected = "v0=" + hmac.new(
                self.config["webhooks"]["slack"]["signing_secret"].encode(),
                sig_basestring.encode(),
                hashlib.sha256
            ).hexdigest()
            return hmac.compare_digest(expected, signature)
        
        return True
    
    def register_handler(self, provider: str, handler: Callable):
        """注册 Webhook 处理器"""
        self.handlers[provider] = handler
```

---

## 统一消息路由

```python
# openclaw/core/router.py
class UnifiedMessageRouter:
    """统一消息路由器"""
    
    def __init__(self, agent: "Agent"):
        self.agent = agent
        self.platforms: Dict[str, PlatformAdapter] = {}
        self.routing_rules: List[RoutingRule] = []
    
    def register_platform(self, adapter: PlatformAdapter):
        """注册平台适配器"""
        self.platforms[adapter.platform_name] = adapter
        adapter.on_message(self._handle_platform_message)
    
    async def _handle_platform_message(
        self, 
        message: PlatformMessage
    ):
        """处理来自任意平台的消息"""
        
        # 1. 应用路由规则
        route = self._match_routing_rule(message)
        
        # 2. 转换为 Agent 输入
        agent_input = message.to_agent_input()
        agent_input.context = route.context
        
        # 3. 发送给 Agent 处理
        response = await self.agent.process(agent_input)
        
        # 4. 根据路由规则发送响应
        await self._send_response(message, response, route)
    
    async def _send_response(
        self,
        message: PlatformMessage,
        response: "AgentResponse",
        route: RoutingRule
    ):
        """发送响应到原始平台或其他指定平台"""
        
        target_platform = route.target_platform or message.platform
        target_chat = route.target_chat or message.chat_id
        
        adapter = self.platforms.get(target_platform)
        if adapter:
            await adapter.send_message(
                chat_id=target_chat,
                content=response.text,
                **route.send_options
            )
```

> [!summary] 关键要点
> 1. **适配器模式**：每个平台一个适配器，统一消息格式
> 2. **事件驱动**：所有消息异步处理，支持高并发
> 3. **配置驱动**：YAML 配置管理所有平台参数
> 4. **安全验证**：Webhook 签名验证，防重放攻击
> 5. **统一路由**：跨平台消息统一路由和响应

---

## 相关文档

- [[../插件开发/OpenClaw插件开发]] - 扩展平台能力
- [[../记忆系统/OpenClaw记忆系统]] - 跨平台上下文保持
- [[../高级用法/OpenClaw高级用法]] - 工作流与多Agent集成
- [[OpenClaw概览]] - 系统整体架构
