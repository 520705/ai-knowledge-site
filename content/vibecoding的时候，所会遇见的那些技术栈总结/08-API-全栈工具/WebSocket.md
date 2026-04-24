# WebSocket：实时双向通信协议的深度解析

> [!NOTE]
> 本文档最后更新于 **2026年4月**，深入剖析 WebSocket 协议原理、与 SSE/轮询的对比、Node.js 实现，以及在 AI 应用中的实战场景。

---

## 目录

1. [[#协议原理]]
2. [[#WebSocket vs SSE vs 轮询]]
3. [[#ws 库详解]]
4. [[#Socket.IO 抽象层]]
5. [[#与 WebRTC 的关系]]
6. [[#AI 应用场景]]
7. [[#实战开发指南]]
8. [[#选型建议]]

---

## 协议原理

### 什么是 WebSocket

WebSocket 是一种在单个 TCP 连接上提供全双工通信的协议。与 HTTP 的请求-响应模型不同，WebSocket 允许服务器主动向客户端推送数据，实现了真正的双向实时通信。

WebSocket 的核心特点：

| 特点 | 说明 |
|------|------|
| **全双工** | 客户端和服务器可同时发送数据 |
| **持久连接** | 建立一次，长期保持 |
| **低延迟** | 无 HTTP 头部开销 |
| **轻量** | 帧格式简单，开销小 |
| **同源策略** | 受浏览器同源策略限制（可配置 CORS）|

### HTTP Upgrade 握手

WebSocket 连接始于 HTTP 请求，通过协议升级握手转换：

```
┌─────────────────────────────────────────────────────────┐
│                    WebSocket 握手                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  客户端 → 服务器 (HTTP 请求)                              │
│  ─────────────────────────────────────────────────────  │
│  GET /ws HTTP/1.1                                       │
│  Host: api.example.com                                  │
│  Upgrade: websocket          ← 关键：Upgrade 头         │
│  Connection: Upgrade           ← 关键：Connection 头     │
│  Sec-WebSocket-Key: dGhl...    ← 随机密钥（Base64）     │
│  Sec-WebSocket-Version: 13    ← 版本号                  │
│  Sec-WebSocket-Extensions: permessage-deflate          │
│                                                         │
│  服务器 → 客户端 (HTTP 响应)                             │
│  ─────────────────────────────────────────────────────  │
│  HTTP/1.1 101 Switching Protocols  ← 101 状态码        │
│  Upgrade: websocket                                      │
│  Connection: Upgrade                                     │
│  Sec-WebSocket-Accept: s3pPL... ← 验证密钥（RFC 6455）  │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │           握手完成，现在进入 WebSocket 模式          │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  客户端 ↔ 服务器 (WebSocket 帧)                          │
│  ─────────────────────────────────────────────────────  │
│  数据帧：文本、二进制、控制帧（Ping/Pong/Close）          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 帧结构

WebSocket 传输的基本单位是帧（Frame）：

```
┌─────────────────────────────────────────────────────────┐
│                     WebSocket 帧结构                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  0                   1                   2                   3
│  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
│ +-+---------------+-+---------------+-+---------------+-+
│ |F|R|R|R| Opcode  |M|     Mask      |          Payload          |
│ |1|2|3|4|         |5|    (bit)       |           Length          |
│ +-+---------------+-+---------------+-+---------------+-+
│ |         Masking-Key (optional)      |          Payload Data    |
│ +-+---------------+-+---------------+-+---------------+-+
│ |                  Extended Payload Length (optional)             |
│ +-+---------------+-+---------------+-+---------------+-+
│ |                                                               |
│ +---------------------------------------------------------------+
│ |                        Payload Data                           |
│ +---------------------------------------------------------------+
│                                                         │
└─────────────────────────────────────────────────────────┘

字段说明：
- FIN (1 bit): 是否是消息的最后一帧
- Opcode (4 bits): 帧类型
  - 0x0: Continuation
  - 0x1: Text frame
  - 0x2: Binary frame
  - 0x8: Close
  - 0x9: Ping
  - 0xA: Pong
- Mask (1 bit): 是否masked（客户端→服务器必须为1）
- Payload Length: 数据长度
- Masking-Key: 掩码密钥
```

### 连接生命周期

```
连接建立 → 连接打开 → 数据传输 → 保持活跃(Ping/Pong) → 关闭握手 → 连接关闭
    ↓           ↓           ↓            ↓                ↓           ↓
  Handshake   Ready    Bidirectional   Heartbeat        Close Frame   Done
```

---

## WebSocket vs SSE vs 轮询

### 技术对比

| 特性 | WebSocket | Server-Sent Events | HTTP Long Polling | HTTP Short Polling |
|------|-----------|-------------------|-------------------|-------------------|
| **方向** | 双向 | 单向（服务器→客户端）| 请求-响应 | 请求-响应 |
| **连接类型** | 持久 | 持久 | 重复建立 | 重复建立 |
| **延迟** | 极低 | 低 | 中等 | 高 |
| **实时性** | 真正实时 | 准实时 | 准实时 | 低 |
| **HTTP/2** | 支持 | 支持 | 支持 | 支持 |
| **自动重连** | 需手动实现 | 内置 | 需手动实现 | 内置 |
| **浏览器支持** | 现代浏览器 | 现代浏览器 | 所有浏览器 | 所有浏览器 |
| **二进制数据** | ✅ | ❌ | ✅ | ✅ |
| **防火墙兼容性** | 一般 | 较好 | 最好 | 最好 |

### 协议选择决策树

```
┌─────────────────────────────────────────────────────────┐
│                   选择实时通信协议                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  需要服务器→客户端推送？                                   │
│       │                                                  │
│    是 ─┴─ 否                                            │
│       │       │                                          │
│   需要客户端→服务器通信？                                 │
│       │       │                                          │
│    是 ─┴─ 否  ──→ SSE (Server-Sent Events)              │
│       │                                                  │
│   纯文本数据？                                            │
│       │       │                                          │
│    是 ─┴─ 否  ──→ WebSocket (二进制)                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 性能对比

| 场景 | WebSocket | SSE | 轮询（1s）|
|------|-----------|-----|-----------|
| **消息延迟** | < 10ms | < 50ms | 500-1000ms |
| **每秒消息数** | 10,000+ | 10,000+ | 1 |
| **带宽效率** | 极好 | 极好 | 差 |
| **服务器资源** | 低 | 低 | 高 |
| **客户端复杂度** | 中等 | 低 | 低 |

---

## ws 库详解

### Node.js WebSocket 实现

`ws` 是 Node.js 生态中最流行的 WebSocket 库：

```bash
# 安装
npm install ws
```

### 服务器实现

```typescript
import { WebSocketServer, WebSocket } from 'ws'

const wss = new WebSocketServer({ port: 8080 })

// 连接处理
wss.on('connection', (ws: WebSocket, req: http.IncomingMessage) => {
  console.log('Client connected:', req.socket.remoteAddress)
  
  // 发送欢迎消息
  ws.send(JSON.stringify({
    type: 'welcome',
    message: 'Connected to WebSocket server'
  }))
  
  // 接收消息
  ws.on('message', (data: Buffer) => {
    try {
      const message = JSON.parse(data.toString())
      console.log('Received:', message)
      
      // 处理不同类型的消息
      switch (message.type) {
        case 'ping':
          ws.send(JSON.stringify({ type: 'pong', timestamp: Date.now() }))
          break
          
        case 'chat':
          // 广播给所有客户端
          broadcast({
            type: 'chat',
            user: message.user,
            content: message.content,
            timestamp: Date.now()
          })
          break
          
        case 'subscribe':
          // 订阅主题
          handleSubscribe(ws, message.topic)
          break
      }
    } catch (err) {
      console.error('Failed to parse message:', err)
    }
  })
  
  // 错误处理
  ws.on('error', (err) => {
    console.error('WebSocket error:', err)
  })
  
  // 关闭处理
  ws.on('close', (code, reason) => {
    console.log(`Client disconnected: ${code} - ${reason}`)
    removeClient(ws)
  })
})

// 广播消息给所有客户端
function broadcast(message: object) {
  const data = JSON.stringify(message)
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(data)
    }
  })
}

// 向特定客户端发送消息
function sendTo(ws: WebSocket, message: object) {
  if (ws.readyState === WebSocket.OPEN) {
    ws.send(JSON.stringify(message))
  }
}

// 心跳检测
setInterval(() => {
  wss.clients.forEach((ws) => {
    if ((ws as any).isAlive === false) {
      ws.terminate()
      return
    }
    ;(ws as any).isAlive = false
    ws.ping()
  })
}, 30000)

wss.on('connection', (ws) => {
  ;(ws as any).isAlive = true
  ws.on('pong', () => {
    ;(ws as any).isAlive = true
  })
})
```

### 客户端实现

```typescript
class WebSocketClient {
  private ws: WebSocket
  private reconnectAttempts = 0
  private maxReconnectAttempts = 5
  
  constructor(url: string) {
    this.ws = new WebSocket(url)
    this.setupEventHandlers()
  }
  
  private setupEventHandlers() {
    this.ws.onopen = () => {
      console.log('Connected to server')
      this.reconnectAttempts = 0
    }
    
    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data)
        this.handleMessage(data)
      } catch (err) {
        console.error('Failed to parse message:', err)
      }
    }
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error)
    }
    
    this.ws.onclose = (event) => {
      console.log(`Connection closed: ${event.code} - ${event.reason}`)
      this.handleClose()
    }
  }
  
  private handleMessage(data: any) {
    console.log('Received:', data)
    
    switch (data.type) {
      case 'welcome':
        console.log('Server welcome:', data.message)
        break
      case 'chat':
        this.onChatMessage?.(data)
        break
      case 'notification':
        this.onNotification?.(data)
        break
    }
  }
  
  private handleClose() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++
      const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000)
      console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`)
      setTimeout(() => this.reconnect(), delay)
    }
  }
  
  private reconnect() {
    this.ws = new WebSocket(this.ws.url)
    this.setupEventHandlers()
  }
  
  send(message: object) {
    if (this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(message))
    } else {
      console.warn('WebSocket not connected')
    }
  }
  
  close() {
    this.ws.close(1000, 'Client closing')
  }
  
  // 事件回调
  onChatMessage?: (data: any) => void
  onNotification?: (data: any) => void
}

// 使用示例
const client = new WebSocketClient('ws://localhost:8080')

client.onChatMessage = (data) => {
  console.log(`[${data.user}]: ${data.content}`)
}

client.send({
  type: 'chat',
  user: 'Alice',
  content: 'Hello, World!'
})
```

### Express 集成

```typescript
import express from 'express'
import { createServer } from 'http'
import { WebSocketServer, WebSocket } from 'ws'

const app = express()
const server = createServer(app)
const wss = new WebSocketServer({ server })

// REST API
app.get('/api/status', (req, res) => {
  res.json({
    status: 'ok',
    connections: wss.clients.size
  })
})

// WebSocket
wss.on('connection', (ws) => {
  console.log('New WebSocket connection')
  
  ws.on('message', (data) => {
    // 处理消息
  })
})

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000')
  console.log('WebSocket available at ws://localhost:3000')
})
```

---

## Socket.IO 抽象层

### Socket.IO 简介

Socket.IO 是一个构建在 WebSocket 之上的抽象层，提供了额外的功能：

| 功能 | Socket.IO | WebSocket (ws) |
|------|-----------|---------------|
| **自动重连** | ✅ 内置 | ❌ 需手动实现 |
| **心跳检测** | ✅ 自动 | ⚠️ 需手动实现 |
| **广播/房间** | ✅ 简单 API | ❌ 需手动实现 |
| **降级支持** | ✅ 自动降级轮询 | ❌ 不支持 |
| **二进制支持** | ✅ 自动处理 | ⚠️ 需手动处理 |
| **包大小** | 较大 | 极小 |

### Socket.IO 服务器

```typescript
import { Server } from 'socket.io'

const io = new Server(3000, {
  cors: {
    origin: '*',
    methods: ['GET', 'POST']
  }
})

// 连接处理
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id)
  
  // 加入房间
  socket.on('join-room', (roomId: string) => {
    socket.join(roomId)
    console.log(`Client ${socket.id} joined room ${roomId}`)
    
    // 通知房间内其他人
    socket.to(roomId).emit('user-joined', {
      userId: socket.id,
      timestamp: Date.now()
    })
  })
  
  // 离开房间
  socket.on('leave-room', (roomId: string) => {
    socket.leave(roomId)
    socket.to(roomId).emit('user-left', {
      userId: socket.id
    })
  })
  
  // 发送消息给房间内所有人（包括发送者）
  socket.on('room-message', (data: { roomId: string, content: string }) => {
    io.to(data.roomId).emit('new-message', {
      from: socket.id,
      content: data.content,
      timestamp: Date.now()
    })
  })
  
  // 发送消息给房间内其他人（不包括发送者）
  socket.on('private-message', (data: { to: string, content: string }) => {
    io.to(data.to).emit('message', {
      from: socket.id,
      content: data.content
    })
  })
  
  // 断开连接
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id)
  })
})
```

### Socket.IO 客户端

```typescript
import { io, Socket } from 'socket.io-client'

class ChatClient {
  private socket: Socket
  
  constructor(serverUrl: string) {
    this.socket = io(serverUrl)
    this.setupListeners()
  }
  
  private setupListeners() {
    this.socket.on('connect', () => {
      console.log('Connected:', this.socket.id)
    })
    
    this.socket.on('new-message', (data) => {
      console.log(`Message from ${data.from}: ${data.content}`)
    })
    
    this.socket.on('user-joined', (data) => {
      console.log(`User joined: ${data.userId}`)
    })
  }
  
  joinRoom(roomId: string) {
    this.socket.emit('join-room', roomId)
  }
  
  sendToRoom(roomId: string, content: string) {
    this.socket.emit('room-message', { roomId, content })
  }
  
  sendPrivate(to: string, content: string) {
    this.socket.emit('private-message', { to, content })
  }
  
  disconnect() {
    this.socket.disconnect()
  }
}

// 使用
const chat = new ChatClient('http://localhost:3000')
chat.joinRoom('general')
chat.sendToRoom('general', 'Hello everyone!')
```

---

## 与 WebRTC 的关系

### 协议层级对比

```
┌─────────────────────────────────────────────────────────┐
│                   通信协议层级                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  高层应用协议                                            │
│  ┌─────────┬─────────┬─────────┬─────────┐              │
│  │WebSocket│  SSE   │  HTTP   │  WebRTC │              │
│  └────┬────┴────┬────┴────┬────┴────┬────┘              │
│       │         │         │         │                   │
│  ─────┴─────────┴─────────┴─────────┴────  ──────────────  │
│       │         │         │         │                    │
│  传输层协议                                            │
│  ┌─────────────────────────────────────────┐           │
│  │            TCP / UDP                    │           │
│  └─────────────────────────────────────────┘           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 互补关系

| 场景 | 推荐协议 | 原因 |
|------|---------|------|
| **文本聊天** | WebSocket | 低延迟、简单 |
| **文件传输** | WebRTC (DataChannel) | P2P、低延迟 |
| **音视频通话** | WebRTC | 内置媒体支持 |
| **实时协作** | WebSocket + CRDT | 文本同步 |
| **股票行情** | WebSocket | 高频、低延迟 |
| **通知推送** | SSE | 单向、简单 |

---

## AI 应用场景

### AI 对话应用

WebSocket 是构建 AI 实时对话的核心技术：

```typescript
// AI Chat 服务器
const conversations = new Map<string, any[]>()

wss.on('connection', (ws) => {
  let conversationId: string | null = null
  
  ws.on('message', async (data: Buffer) => {
    const message = JSON.parse(data.toString())
    
    switch (message.type) {
      case 'start':
        // 开始新对话
        conversationId = generateId()
        conversations.set(conversationId, [])
        ws.send(JSON.stringify({
          type: 'started',
          conversationId
        }))
        break
        
      case 'message':
        if (!conversationId) {
          ws.send(JSON.stringify({
            type: 'error',
            message: 'No active conversation'
          }))
          return
        }
        
        // 添加用户消息
        const conversation = conversations.get(conversationId)!
        conversation.push({
          role: 'user',
          content: message.content,
          timestamp: Date.now()
        })
        
        // 发送"正在输入"状态
        ws.send(JSON.stringify({ type: 'thinking' }))
        
        try {
          // 调用 AI API
          const response = await callAI(conversation)
          
          // 添加 AI 响应
          conversation.push({
            role: 'assistant',
            content: response,
            timestamp: Date.now()
          })
          
          // 发送完整响应
          ws.send(JSON.stringify({
            type: 'response',
            content: response,
            done: true
          }))
        } catch (err) {
          ws.send(JSON.stringify({
            type: 'error',
            message: 'AI service error'
          }))
        }
        break
        
      case 'stream':
        // 流式响应
        const stream = await callAIStream(message.content)
        
        for await (const chunk of stream) {
          ws.send(JSON.stringify({
            type: 'stream',
            content: chunk,
            done: false
          }))
        }
        
        ws.send(JSON.stringify({
          type: 'stream',
          content: '',
          done: true
        }))
        break
    }
  })
})

// 流式 AI 调用示例
async function* callAIStream(messages: any[]) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4o',
    messages,
    stream: true,
  })
  
  for await (const chunk of response) {
    const content = chunk.choices[0]?.delta?.content || ''
    if (content) {
      yield content
    }
  }
}
```

### 实时 AI 助手

```typescript
// 前端流式处理
class AIStreamClient {
  private ws: WebSocket
  
  constructor() {
    this.ws = new WebSocket('ws://localhost:8080')
  }
  
  async sendMessage(content: string, onChunk: (text: string) => void) {
    return new Promise<void>((resolve, reject) => {
      let fullResponse = ''
      
      this.ws.onmessage = (event) => {
        const data = JSON.parse(event.data)
        
        if (data.type === 'thinking') {
          console.log('AI is thinking...')
          return
        }
        
        if (data.type === 'stream') {
          fullResponse += data.content
          onChunk(fullResponse)
          
          if (data.done) {
            resolve()
          }
        }
        
        if (data.type === 'error') {
          reject(new Error(data.message))
        }
      }
      
      this.ws.send(JSON.stringify({
        type: 'stream',
        content
      }))
    })
  }
}

// 使用
const client = new AIStreamClient()
const messageEl = document.getElementById('message')!

await client.sendMessage('Explain quantum computing', (text) => {
  messageEl.textContent = text
})
```

---

## 实战开发指南

### 生产环境注意事项

```typescript
// 1. 使用安全的 WebSocket (WSS)
const wss = new WebSocketServer({
  port: 8080,
  // 生产环境使用 TLS
})

// 2. 实现连接限制
const MAX_CONNECTIONS = 1000
const clients = new Set<WebSocket>()

wss.on('connection', (ws) => {
  if (clients.size >= MAX_CONNECTIONS) {
    ws.close(1013, 'Too many connections')
    return
  }
  clients.add(ws)
  ws.on('close', () => clients.delete(ws))
})

// 3. 消息大小限制
const MAX_MESSAGE_SIZE = 1024 * 1024 // 1MB

ws.on('message', (data) => {
  if (data.length > MAX_MESSAGE_SIZE) {
    ws.close(1009, 'Message too large')
    return
  }
})

// 4. 实现认证
wss.on('connection', (ws, req) => {
  const token = parseToken(req)
  if (!validateToken(token)) {
    ws.close(4001, 'Unauthorized')
    return
  }
})
```

---

## 选型建议

### 选择 WebSocket 的场景

- ✅ 需要双向实时通信
- ✅ 高频消息交换
- ✅ 聊天/协作应用
- ✅ 游戏服务器
- ✅ AI 实时对话

### 不选择 WebSocket 的场景

- ❌ 单向数据传输（SSE 更简单）
- ❌ 低频更新（轮询足够）
- ❌ 需要穿透严格防火墙（HTTP 更安全）

---

## 参考资料

| 资源 | 链接 |
|------|------|
| WebSocket 规范 | https://datatracker.ietf.org/doc/html/rfc6455 |
| ws 库文档 | https://github.com/websockets/ws |
| Socket.IO | https://socket.io/docs/ |
| WebRTC 规范 | https://webrtc.org/ |

---

## WebSocket 高级模式与最佳实践

### 连接管理与心跳检测

```typescript
// 健壮的 WebSocket 连接管理器
class WebSocketManager {
  private ws: WebSocket | null = null;
  private reconnectAttempts = 0;
  private maxReconnectAttempts = 10;
  private reconnectDelay = 1000;
  private heartbeatInterval: number | null = null;
  private missedHeartbeats = 0;
  private maxMissedHeartbeats = 3;
  
  constructor(
    private url: string,
    private handlers: {
      onMessage: (data: unknown) => void;
      onConnect: () => void;
      onDisconnect: () => void;
      onError: (error: Event) => void;
    }
  ) {}
  
  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectAttempts = 0;
      this.reconnectDelay = 1000;
      this.startHeartbeat();
      this.handlers.onConnect();
    };
    
    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      this.stopHeartbeat();
      this.handlers.onDisconnect();
      this.scheduleReconnect();
    };
    
    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
      this.handlers.onError(error);
    };
    
    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        
        if (data.type === 'pong') {
          this.missedHeartbeats = 0;
          return;
        }
        
        this.handlers.onMessage(data);
      } catch (error) {
        console.error('Failed to parse message:', error);
      }
    };
  }
  
  private startHeartbeat() {
    this.heartbeatInterval = window.setInterval(() => {
      if (this.missedHeartbeats >= this.maxMissedHeartbeats) {
        console.log('Too many missed heartbeats, reconnecting...');
        this.ws?.close();
        return;
      }
      
      this.missedHeartbeats++;
      this.send({ type: 'ping' });
    }, 30000);
  }
  
  private stopHeartbeat() {
    if (this.heartbeatInterval) {
      clearInterval(this.heartbeatInterval);
      this.heartbeatInterval = null;
    }
  }
  
  private scheduleReconnect() {
    if (this.reconnectAttempts >= this.maxReconnectAttempts) {
      console.log('Max reconnect attempts reached');
      return;
    }
    
    this.reconnectAttempts++;
    const delay = Math.min(
      this.reconnectDelay * Math.pow(2, this.reconnectAttempts - 1),
      30000
    );
    
    console.log(`Reconnecting in ${delay}ms (attempt ${this.reconnectAttempts})`);
    
    setTimeout(() => this.connect(), delay);
  }
  
  send(data: unknown) {
    if (this.ws?.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify(data));
    }
  }
  
  disconnect() {
    this.reconnectAttempts = this.maxReconnectAttempts; // 阻止重连
    this.stopHeartbeat();
    this.ws?.close();
  }
}

// 使用示例
const manager = new WebSocketManager('wss://api.example.com/ws', {
  onMessage: (data) => console.log('Received:', data),
  onConnect: () => console.log('Connected!'),
  onDisconnect: () => console.log('Disconnected'),
  onError: (error) => console.error('Error:', error),
});

manager.connect();
```

### 消息队列与离线处理

```typescript
// 消息队列实现
class MessageQueue {
  private queue: Array<{ type: string; data: unknown; timestamp: number }> = [];
  private maxQueueSize = 100;
  private isOnline = false;
  
  constructor(private ws: WebSocket) {
    window.addEventListener('online', () => this.flush());
    window.addEventListener('offline', () => this.setOnline(false));
  }
  
  setOnline(online: boolean) {
    this.isOnline = online;
    if (online) {
      this.flush();
    }
  }
  
  enqueue(type: string, data: unknown) {
    if (this.queue.length >= this.maxQueueSize) {
      this.queue.shift(); // 删除最旧的
    }
    
    this.queue.push({
      type,
      data,
      timestamp: Date.now(),
    });
    
    if (this.isOnline) {
      this.flush();
    }
  }
  
  private flush() {
    while (this.queue.length > 0) {
      const message = this.queue.shift();
      if (message) {
        this.send(message.type, message.data);
      }
    }
  }
  
  private send(type: string, data: unknown) {
    if (this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type, data, timestamp: Date.now() }));
    }
  }
  
  getQueuedCount() {
    return this.queue.length;
  }
}
```

### 消息分片与压缩

```typescript
// 消息分片协议
interface FragmentedMessage {
  messageId: string;
  totalFragments: number;
  fragmentIndex: number;
  data: string;
  isCompressed: boolean;
}

class FragmentedMessenger {
  private fragments: Map<string, FragmentedMessage[]> = new Map();
  
  sendLargeMessage(ws: WebSocket, data: unknown, maxFragmentSize = 16 * 1024) {
    const messageId = crypto.randomUUID();
    const json = JSON.stringify(data);
    
    // 压缩大消息
    const compressed = this.compress(json);
    const chunks = this.splitIntoChunks(compressed, maxFragmentSize);
    
    chunks.forEach((chunk, index) => {
      const fragment: FragmentedMessage = {
        messageId,
        totalFragments: chunks.length,
        fragmentIndex: index,
        data: chunk,
        isCompressed: compressed !== json,
      };
      
      ws.send(JSON.stringify({
        type: 'fragment',
        ...fragment,
      }));
    });
  }
  
  receiveFragment(fragment: FragmentedMessage): unknown | null {
    const { messageId, totalFragments, fragmentIndex, data, isCompressed } = fragment;
    
    if (!this.fragments.has(messageId)) {
      this.fragments.set(messageId, []);
    }
    
    const fragments = this.fragments.get(messageId)!;
    fragments[fragmentIndex] = fragment;
    
    // 检查是否完整
    if (fragments.filter(Boolean).length === totalFragments) {
      const sorted = fragments.sort((a, b) => a.fragmentIndex - b.fragmentIndex);
      const combined = sorted.map(f => f.data).join('');
      const decompressed = isCompressed ? this.decompress(combined) : combined;
      this.fragments.delete(messageId);
      return JSON.parse(decompressed);
    }
    
    return null;
  }
  
  private compress(data: string): string {
    // 简单的 base64 编码，实际使用 compress-commons 或 pako
    return btoa(data);
  }
  
  private decompress(data: string): string {
    return atob(data);
  }
  
  private splitIntoChunks(data: string, size: number): string[] {
    const chunks: string[] = [];
    for (let i = 0; i < data.length; i += size) {
      chunks.push(data.slice(i, i + size));
    }
    return chunks;
  }
}
```

### 负载均衡与水平扩展

```typescript
// 使用 Redis Pub/Sub 实现多实例 WebSocket
// server/index.ts
import { createServer } from 'http';
import { Server } from 'socket.io';
import { createClient } from 'redis';

const httpServer = createServer();
const io = new Server(httpServer, {
  cors: { origin: '*' },
});

// Redis 适配器用于跨实例通信
const redisClient = createClient({ url: 'redis://localhost:6379' });
const redisAdapter = require('@socket.io/redis-adapter');
const pubClient = redisClient.duplicate();
const subClient = redisClient.duplicate();

io.adapter(redisAdapter(pubClient, subClient));

// 连接处理
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);
  
  // 加入房间
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    console.log(`Client ${socket.id} joined room ${roomId}`);
  });
  
  // 离开房间
  socket.on('leave-room', (roomId) => {
    socket.leave(roomId);
    console.log(`Client ${socket.id} left room ${roomId}`);
  });
  
  // 广播消息
  socket.on('message', (data) => {
    // 广播到同一房间的所有客户端
    io.to(data.roomId).emit('message', {
      from: socket.id,
      content: data.content,
      timestamp: Date.now(),
    });
  });
  
  // 断开连接
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

// 集群消息处理
redisClient.subscribe('notifications', (message) => {
  const { roomId, data } = JSON.parse(message);
  io.to(roomId).emit('notification', data);
});

httpServer.listen(3000);
```

### WebSocket 与 Next.js App Router

```typescript
// app/api/socket/route.ts
import { NextRequest } from 'next/server';

// 这个文件用于 Vercel 等无持久连接的环境
// 在这些平台上建议使用 Socket.IO 或第三方服务如 Pusher

export async function POST(req: NextRequest) {
  const body = await req.json();
  const { event, data } = body;
  
  // 处理 WebSocket 连接升级请求
  // 注意: Next.js API Routes 不支持 WebSocket 升级
  // 需要使用独立的 WebSocket 服务器或 Server Actions
  
  return new Response(JSON.stringify({ success: true }), {
    status: 200,
  });
}

// 推荐: 使用自定义服务器
// server.ts
import { createServer } from 'http';
import { parse } from 'url';
import next from 'next';
import { Server as SocketIOServer } from 'socket.io';

const dev = process.env.NODE_ENV !== 'production';
const app = next({ dev });
const handle = app.getRequestHandler();

app.prepare().then(() => {
  const server = createServer((req, res) => {
    const parsedUrl = parse(req.url!, true);
    handle(req, res, parsedUrl);
  });
  
  const io = new SocketIOServer(server, {
    path: '/api/socket',
    cors: { origin: '*' },
  });
  
  io.on('connection', (socket) => {
    console.log('Client connected');
    
    socket.on('message', (data) => {
      console.log('Received:', data);
      socket.emit('response', { message: 'Received!' });
    });
  });
  
  server.listen(3000, () => {
    console.log('Server running on http://localhost:3000');
  });
});
```

### WebSocket 安全加固

```typescript
// 1. 认证中间件
const authMiddleware = (socket: Socket, next: (err?: Error) => void) => {
  const token = socket.handshake.auth.token;
  
  if (!token) {
    return next(new Error('Authentication required'));
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    socket.data.user = decoded;
    next();
  } catch (error) {
    next(new Error('Invalid token'));
  }
};

// 2. 速率限制
const rateLimiter = new Map<string, { count: number; resetTime: number }>();
const RATE_LIMIT = 100; // 每分钟最多 100 条消息
const RATE_WINDOW = 60000; // 1 分钟

socket.on('message', (data) => {
  const identifier = socket.id;
  const now = Date.now();
  
  if (!rateLimiter.has(identifier)) {
    rateLimiter.set(identifier, { count: 0, resetTime: now + RATE_WINDOW });
  }
  
  const limit = rateLimiter.get(identifier)!;
  
  if (now > limit.resetTime) {
    limit.count = 0;
    limit.resetTime = now + RATE_WINDOW;
  }
  
  limit.count++;
  
  if (limit.count > RATE_LIMIT) {
    socket.emit('error', { message: 'Rate limit exceeded' });
    return;
  }
  
  // 处理消息
  handleMessage(socket, data);
});

// 3. 输入验证
const z = require('zod');

const messageSchema = z.object({
  type: z.enum(['chat', 'notification', 'command']),
  content: z.string().max(10000),
  roomId: z.string().optional(),
});

socket.on('message', (rawData) => {
  try {
    const data = JSON.parse(rawData);
    const validated = messageSchema.parse(data);
    handleMessage(socket, validated);
  } catch (error) {
    socket.emit('error', { message: 'Invalid message format' });
  }
});
```

### WebSocket 性能监控

```typescript
// 性能监控中间件
class WebSocketMonitor {
  private metrics = {
    connections: 0,
    messagesSent: 0,
    messagesReceived: 0,
    errors: 0,
    averageLatency: 0,
  };
  
  private latencies: number[] = [];
  
  trackConnection(socket: Socket) {
    this.metrics.connections++;
    
    socket.on('disconnect', () => {
      this.metrics.connections--;
    });
  }
  
  trackMessage(socket: Socket, direction: 'sent' | 'received') {
    if (direction === 'sent') {
      this.metrics.messagesSent++;
    } else {
      this.metrics.messagesReceived++;
      
      // 记录延迟
      socket.on('pong', (timestamp: number) => {
        const latency = Date.now() - timestamp;
        this.latencies.push(latency);
        
        if (this.latencies.length > 100) {
          this.latencies.shift();
        }
        
        this.metrics.averageLatency = 
          this.latencies.reduce((a, b) => a + b, 0) / this.latencies.length;
      });
    }
  }
  
  trackError() {
    this.metrics.errors++;
  }
  
  getMetrics() {
    return {
      ...this.metrics,
      latencyP50: this.percentile(this.latencies, 0.5),
      latencyP95: this.percentile(this.latencies, 0.95),
      latencyP99: this.percentile(this.latencies, 0.99),
    };
  }
  
  private percentile(arr: number[], p: number): number {
    if (arr.length === 0) return 0;
    const sorted = [...arr].sort((a, b) => a - b);
    const index = Math.ceil(sorted.length * p) - 1;
    return sorted[index];
  }
}

// Prometheus 格式输出
io.on('connection', (socket) => {
  monitor.trackConnection(socket);
  
  socket.on('message', (data) => {
    monitor.trackMessage(socket, 'received');
    monitor.trackMessage(socket, 'sent');
  });
});

// /metrics 端点
app.get('/metrics', (req, res) => {
  const metrics = monitor.getMetrics();
  
  const prometheusFormat = Object.entries(metrics)
    .map(([key, value]) => `websocket_${key} ${value}`)
    .join('\n');
  
  res.set('Content-Type', 'text/plain');
  res.send(prometheusFormat);
});
```

### WebSocket 与 AI 流式响应

```typescript
// AI 流式响应的 WebSocket 处理
// server/aiSocket.ts
import { OpenAI } from 'openai';

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// AI 聊天处理器
export async function handleAIChat(
  socket: Socket,
  message: { content: string; model?: string; temperature?: number }
) {
  const { content, model = 'gpt-4o', temperature = 0.7 } = message;
  
  try {
    // 发送开始信号
    socket.emit('ai_start', { timestamp: Date.now() });
    
    const stream = await openai.chat.completions.create({
      model,
      messages: [{ role: 'user', content }],
      stream: true,
      temperature,
    });
    
    let fullResponse = '';
    
    for await (const chunk of stream) {
      const token = chunk.choices[0]?.delta?.content;
      
      if (token) {
        fullResponse += token;
        socket.emit('ai_token', { token, timestamp: Date.now() });
      }
    }
    
    // 发送完成信号
    socket.emit('ai_complete', {
      response: fullResponse,
      timestamp: Date.now(),
    });
    
  } catch (error) {
    console.error('AI error:', error);
    socket.emit('ai_error', {
      message: error instanceof Error ? error.message : 'AI request failed',
    });
  }
}

// 客户端使用
const socket = io('wss://api.example.com');

socket.on('ai_token', (data) => {
  appendToMessage(data.token);
});

socket.on('ai_complete', (data) => {
  finishMessage();
});

socket.on('ai_error', (data) => {
  showError(data.message);
});

function sendMessage(content: string) {
  socket.emit('ai_chat', { content });
}
```

---

## WebSocket 与其他实时技术对比

### 详细对比表

| 特性 | WebSocket | Server-Sent Events | Socket.IO | GraphQL Subscriptions |
|------|-----------|-------------------|-----------|----------------------|
| **协议** | 双向 | 单向 | 双向 | 双向 |
| **浏览器支持** | 广泛 | 现代浏览器 | 广泛 | 广泛 |
| **自动重连** | 需手动实现 | 需手动实现 | 内置 | 取决于实现 |
| **二进制支持** | ✅ | ❌ | ✅ | ❌ |
| **多路复用** | 需自定义 | ❌ | ✅ | ❌ |
| **IE 支持** | 10+ | ❌ | ✅ | ✅ |
| **断线重连** | 手动 | 手动 | 自动 | 取决于实现 |
| **体积** | ~0KB | ~0kb | ~15KB | 取决于库 |

### 选择指南

```typescript
// 决策树
function chooseRealTimeTech() {
  const questions = [
    {
      q: '需要双向通信吗？',
      options: [
        { answer: '是', next: 'needBidirectional' },
        { answer: '否', next: 'useSSE' },
      ],
    },
    {
      q: '需要 IE 支持吗？',
      options: [
        { answer: '是', next: 'useSocketIO' },
        { answer: '否', next: 'needAdvanced' },
      ],
    },
    {
      q: '需要 HTTP/2 多路复用吗？',
      options: [
        { answer: '是', next: 'useGraphQLSub' },
        { answer: '否', next: 'useWebSocket' },
      ],
    },
  ];
  
  // 返回推荐方案
  return {
    recommended: 'WebSocket',
    alternatives: ['Socket.IO', 'GraphQL Subscriptions'],
    reasons: [
      'WebSocket 是标准协议，兼容性最好',
      '支持双向通信和二进制数据',
      '服务器资源占用低',
    ],
  };
}
```

---

---

## WebSocket 高级安全配置

### 认证与授权

```typescript
// server/auth/wsAuth.ts
import { WebSocketServer, WebSocket } from 'ws';
import { createServer } from 'http';
import jwt from 'jsonwebtoken';
import { authMiddleware } from './auth';

interface AuthenticatedWebSocket extends WebSocket {
  userId?: string;
  userRole?: string;
  isAlive?: boolean;
}

// 创建 HTTP 服务器
const server = createServer(app);

// 创建 WebSocket 服务器
const wss = new WebSocketServer({ 
  server,
  path: '/ws',
  clientTracking: true,
});

// JWT 认证中间件
wss.on('connection', async (ws: AuthenticatedWebSocket, req) => {
  // 从 URL 获取 token
  const url = new URL(req.url || '', `http://${req.headers.host}`);
  const token = url.searchParams.get('token');
  
  if (!token) {
    ws.close(4001, 'Authentication required');
    return;
  }
  
  try {
    // 验证 JWT
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
      userId: string;
      role: string;
    };
    
    ws.userId = decoded.userId;
    ws.userRole = decoded.role;
    ws.isAlive = true;
    
    console.log(`User ${decoded.userId} connected`);
    
  } catch (error) {
    ws.close(4002, 'Invalid token');
    return;
  }
});

// 角色授权装饰器
function requireRole(roles: string[]) {
  return (target: any, propertyKey: string, descriptor: PropertyDescriptor) => {
    const originalMethod = descriptor.value;
    
    descriptor.value = function(ws: AuthenticatedWebSocket, ...args: any[]) {
      if (!ws.userRole || !roles.includes(ws.userRole)) {
        ws.send(JSON.stringify({
          type: 'error',
          code: 'FORBIDDEN',
          message: 'Insufficient permissions',
        }));
        return;
      }
      
      return originalMethod.apply(this, [ws, ...args]);
    };
    
    return descriptor;
  };
}
```

### 速率限制

```typescript
// server/middleware/rateLimit.ts
import { RateLimiterMemory } from 'rate-limiter-flexible';

interface RateLimitConfig {
  windowMs: number;      // 时间窗口（毫秒）
  maxConnections: number;  // 最大连接数
  maxMessages: number;    // 最大消息数
}

class WebSocketRateLimiter {
  private connectionLimiter: RateLimiterMemory;
  private messageLimiter: RateLimiterMemory;
  private messageCounts: Map<string, { count: number; resetTime: number }>;
  
  constructor(config: RateLimitConfig) {
    this.connectionLimiter = new RateLimiterMemory({
      points: config.maxConnections,
      duration: config.windowMs / 1000,
    });
    
    this.messageLimiter = new RateLimiterMemory({
      points: config.maxMessages,
      duration: 1, // 每秒
    });
    
    this.messageCounts = new Map();
  }
  
  async checkConnection(ip: string): Promise<boolean> {
    try {
      await this.connectionLimiter.consume(ip);
      return true;
    } catch {
      return false;
    }
  }
  
  async checkMessage(wsId: string): Promise<boolean> {
    try {
      await this.messageLimiter.consume(wsId);
      return true;
    } catch {
      return false;
    }
  }
  
  getMessageCount(wsId: string): number {
    const record = this.messageCounts.get(wsId);
    if (!record || record.resetTime < Date.now()) {
      return 0;
    }
    return record.count;
  }
  
  resetMessageCount(wsId: string): void {
    this.messageCounts.delete(wsId);
  }
}

const rateLimiter = new WebSocketRateLimiter({
  windowMs: 60000,     // 1分钟窗口
  maxConnections: 10,   // 每 IP 10 个连接
  maxMessages: 100,    // 每秒 100 条消息
});

// 应用速率限制
wss.on('connection', async (ws: AuthenticatedWebSocket, req) => {
  const ip = req.socket.remoteAddress || 'unknown';
  
  if (!(await rateLimiter.checkConnection(ip))) {
    ws.close(4003, 'Too many connections');
    return;
  }
  
  ws.on('message', async (data) => {
    const wsId = ws.userId || ip;
    
    if (!(await rateLimiter.checkMessage(wsId))) {
      ws.send(JSON.stringify({
        type: 'error',
        code: 'RATE_LIMITED',
        message: 'Too many messages',
      }));
      return;
    }
    
    // 处理消息...
  });
});
```

### 数据加密

```typescript
// server/middleware/encryption.ts
import crypto from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const SECRET_KEY = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex');

// 加密消息
function encryptMessage(message: string): Buffer {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(ALGORITHM, SECRET_KEY, iv);
  
  let encrypted = cipher.update(message, 'utf8');
  encrypted = Buffer.concat([encrypted, cipher.final()]);
  
  const authTag = cipher.getAuthTag();
  
  // 返回格式: iv + authTag + encrypted
  return Buffer.concat([iv, authTag, encrypted]);
}

// 解密消息
function decryptMessage(encryptedBuffer: Buffer): string {
  const iv = encryptedBuffer.subarray(0, 16);
  const authTag = encryptedBuffer.subarray(16, 32);
  const encrypted = encryptedBuffer.subarray(32);
  
  const decipher = crypto.createDecipheriv(ALGORITHM, SECRET_KEY, iv);
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted);
  decrypted = Buffer.concat([decrypted, decipher.final()]);
  
  return decrypted.toString('utf8');
}

// 安全消息发送
function secureSend(ws: WebSocket, data: any): void {
  const message = typeof data === 'string' ? data : JSON.stringify(data);
  const encrypted = encryptMessage(message);
  ws.send(encrypted);
}

// 安全消息接收
function handleSecureMessage(ws: AuthenticatedWebSocket, buffer: Buffer): any {
  const decrypted = decryptMessage(buffer);
  
  try {
    return JSON.parse(decrypted);
  } catch {
    return decrypted;
  }
}
```

---

## WebSocket 与 Kubernetes

### 高可用部署

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-server
spec:
  replicas: 3
  selector:
    matchLabels:
      app: websocket-server
  template:
    metadata:
      labels:
        app: websocket-server
    spec:
      containers:
        - name: ws-server
          image: your-registry/ws-server:latest
          ports:
            - containerPort: 8080
          env:
            - name: REDIS_HOST
              valueFrom:
                secretKeyRef:
                  name: redis-config
                  key: host
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: websocket-service
spec:
  selector:
    app: websocket-server
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: websocket-service-lb
spec:
  selector:
    app: websocket-server
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

### Session Affinity 配置

```yaml
# 对于需要会话粘性的场景
apiVersion: v1
kind: Service
metadata:
  name: websocket-service-sticky
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800  # 3小时
  selector:
    app: websocket-server
  ports:
    - port: 80
      targetPort: 8080
```

### 水平 Pod 自动伸缩

```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: websocket-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: websocket-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    - type: Pods
      pods:
        metric:
          name: websocket_active_connections
        target:
          type: AverageValue
          averageValue: "100"
```

---

## WebSocket 与 Redis 集群

### Redis Pub/Sub 消息分发

```typescript
// server/redis/cluster.ts
import Redis from 'ioredis';
import { WebSocketServer, WebSocket } from 'ws';

// Redis 集群配置
const redis = new Redis.Cluster([
  { host: '10.0.0.1', port: 6379 },
  { host: '10.0.0.2', port: 6379 },
  { host: '10.0.0.3', port: 6379 },
]);

// WebSocket 服务器实例
const wss = new WebSocketServer({ port: 8080 });

// 频道订阅
const CHANNEL = 'broadcast:messages';

// 订阅 Redis 频道
redis.subscribe(CHANNEL, (err) => {
  if (err) {
    console.error('Failed to subscribe:', err);
  } else {
    console.log(`Subscribed to ${CHANNEL}`);
  }
});

// 处理 Redis 消息
redis.on('message', (channel, message) => {
  if (channel === CHANNEL) {
    const data = JSON.parse(message);
    broadcastToAll(data);
  }
});

// 广播消息给所有连接
function broadcastToAll(data: any) {
  wss.clients.forEach((client) => {
    if (client.readyState === WebSocket.OPEN) {
      client.send(JSON.stringify(data));
    }
  });
}

// 处理 WebSocket 连接
wss.on('connection', (ws: WebSocket, req) => {
  ws.on('message', async (message) => {
    const data = JSON.parse(message.toString());
    
    // 发布到 Redis
    await redis.publish(CHANNEL, JSON.stringify({
      ...data,
      timestamp: Date.now(),
    }));
  });
});
```

### Redis 分布式锁

```typescript
// server/redis/distributedLock.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL!);

interface LockOptions {
  ttl: number;       // 锁过期时间（毫秒）
  retries: number;   // 重试次数
  delay: number;     // 重试延迟（毫秒）
}

class DistributedLock {
  private redis: Redis;
  
  constructor(redis: Redis) {
    this.redis = redis;
  }
  
  async acquire(
    resource: string, 
    owner: string, 
    options: LockOptions = { ttl: 10000, retries: 3, delay: 100 }
  ): Promise<boolean> {
    const key = `lock:${resource}`;
    
    for (let i = 0; i < options.retries; i++) {
      // 尝试获取锁
      const result = await this.redis.set(
        key, 
        owner, 
        'PX', 
        options.ttl, 
        'NX'
      );
      
      if (result === 'OK') {
        return true;
      }
      
      // 等待后重试
      if (i < options.retries - 1) {
        await new Promise(resolve => setTimeout(resolve, options.delay));
      }
    }
    
    return false;
  }
  
  async release(resource: string, owner: string): Promise<boolean> {
    const key = `lock:${resource}`;
    
    // Lua 脚本确保只释放自己的锁
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("del", KEYS[1])
      else
        return 0
      end
    `;
    
    const result = await this.redis.eval(script, 1, key, owner);
    return result === 1;
  }
  
  async extend(
    resource: string, 
    owner: string, 
    ttl: number
  ): Promise<boolean> {
    const key = `lock:${resource}`;
    
    // Lua 脚本确保只延续自己的锁
    const script = `
      if redis.call("get", KEYS[1]) == ARGV[1] then
        return redis.call("pexpire", KEYS[1], ARGV[2])
      else
        return 0
      end
    `;
    
    const result = await this.redis.eval(script, 1, key, owner, ttl);
    return result === 1;
  }
}

const lock = new DistributedLock(redis);

// 使用分布式锁
async function handleExclusiveOperation(
  ws: AuthenticatedWebSocket, 
  operation: string, 
  resource: string
) {
  const lockAcquired = await lock.acquire(resource, ws.userId!);
  
  if (!lockAcquired) {
    ws.send(JSON.stringify({
      type: 'error',
      code: 'LOCK_FAILED',
      message: 'Resource is currently locked',
    }));
    return;
  }
  
  try {
    // 执行操作
    await performOperation(operation, resource);
  } finally {
    // 释放锁
    await lock.release(resource, ws.userId!);
  }
}
```

---

## WebSocket 与数据库集成

### 事件溯源模式

```typescript
// server/es/eventStore.ts
import { WebSocket } from 'ws';

interface Event {
  id: string;
  aggregateId: string;
  aggregateType: string;
  eventType: string;
  payload: any;
  timestamp: number;
  metadata: {
    userId?: string;
    correlationId?: string;
  };
}

class EventStore {
  private db: any;
  
  constructor(db: any) {
    this.db = db;
  }
  
  async append(event: Omit<Event, 'id' | 'timestamp'>): Promise<Event> {
    const fullEvent: Event = {
      ...event,
      id: crypto.randomUUID(),
      timestamp: Date.now(),
    };
    
    await this.db.event.create({
      data: {
        id: fullEvent.id,
        aggregateId: fullEvent.aggregateId,
        aggregateType: fullEvent.aggregateType,
        eventType: fullEvent.eventType,
        payload: JSON.stringify(fullEvent.payload),
        metadata: JSON.stringify(fullEvent.metadata),
        createdAt: new Date(fullEvent.timestamp),
      },
    });
    
    return fullEvent;
  }
  
  async getEvents(
    aggregateId: string, 
    fromTimestamp?: number
  ): Promise<Event[]> {
    const events = await this.db.event.findMany({
      where: {
        aggregateId,
        createdAt: fromTimestamp 
          ? { gte: new Date(fromTimestamp) } 
          : undefined,
      },
      orderBy: { createdAt: 'asc' },
    });
    
    return events.map((e: any) => ({
      ...e,
      payload: JSON.parse(e.payload),
      metadata: JSON.parse(e.metadata),
    }));
  }
  
  async subscribe(
    aggregateId: string, 
    ws: WebSocket
  ): Promise<() => void> {
    // 定期检查新事件
    let lastTimestamp = Date.now();
    
    const interval = setInterval(async () => {
      const events = await this.getEvents(aggregateId, lastTimestamp);
      
      if (events.length > 0) {
        lastTimestamp = events[events.length - 1].timestamp;
        
        events.forEach(event => {
          ws.send(JSON.stringify({
            type: 'event',
            event,
          }));
        });
      }
    }, 100);
    
    return () => clearInterval(interval);
  }
}

const eventStore = new EventStore(db);

// WebSocket 订阅聚合
wss.on('connection', (ws: AuthenticatedWebSocket, req) => {
  const url = new URL(req.url || '', 'http://localhost');
  const aggregateId = url.searchParams.get('aggregateId');
  
  if (!aggregateId) {
    ws.close(4004, 'Missing aggregateId');
    return;
  }
  
  // 订阅事件流
  const unsubscribe = eventStore.subscribe(aggregateId, ws);
  
  ws.on('close', () => {
    unsubscribe();
  });
});
```

### CQRS 读写分离

```typescript
// server/cqrs/queryHandler.ts
import { WebSocket } from 'ws';

interface Query {
  type: 'read';
  model: string;
  id?: string;
  filter?: Record<string, any>;
  projection?: string[];
}

interface Command {
  type: 'write';
  model: string;
  operation: 'create' | 'update' | 'delete';
  data: any;
}

// 查询处理器
async function handleQuery(
  ws: AuthenticatedWebSocket, 
  query: Query
): Promise<void> {
  let result;
  
  switch (query.model) {
    case 'user':
      result = await queryUser(query.id, query.filter, query.projection);
      break;
    case 'order':
      result = await queryOrder(query.id, query.filter, query.projection);
      break;
    default:
      ws.send(JSON.stringify({
        type: 'error',
        code: 'UNKNOWN_MODEL',
        message: `Unknown model: ${query.model}`,
      }));
      return;
  }
  
  ws.send(JSON.stringify({
    type: 'query_result',
    correlationId: (query as any).correlationId,
    data: result,
  }));
}

// 命令处理器
async function handleCommand(
  ws: AuthenticatedWebSocket, 
  command: Command
): Promise<void> {
  const correlationId = crypto.randomUUID();
  
  // 发布命令到消息队列
  await publishCommand({
    ...command,
    correlationId,
    userId: ws.userId,
    timestamp: Date.now(),
  });
  
  ws.send(JSON.stringify({
    type: 'command_accepted',
    correlationId,
  }));
}

// WebSocket 消息路由
wss.on('connection', (ws: AuthenticatedWebSocket, req) => {
  ws.on('message', async (data) => {
    const message = JSON.parse(data.toString());
    
    switch (message.type) {
      case 'read':
        await handleQuery(ws, message);
        break;
      case 'write':
        await handleCommand(ws, message);
        break;
      default:
        ws.send(JSON.stringify({
          type: 'error',
          code: 'UNKNOWN_MESSAGE_TYPE',
          message: `Unknown message type: ${message.type}`,
        }));
    }
  });
});
```

---

## WebSocket 性能调优

### 连接池优化

```typescript
// server/pool/connectionPool.ts
import { WebSocket, WebSocketServer } from 'ws';

interface ConnectionPool {
  connections: Map<string, WebSocket>;
  groups: Map<string, Set<string>>;  // groupId -> Set<wsId>
}

class OptimizedConnectionPool implements ConnectionPool {
  connections: Map<string, WebSocket> = new Map();
  groups: Map<string, Set<string>> = new Map();
  
  private maxConnections = 10000;
  private cleanupInterval = 60000; // 1分钟
  
  constructor() {
    // 定期清理无效连接
    setInterval(() => this.cleanup(), this.cleanupInterval);
  }
  
  add(wsId: string, ws: WebSocket): boolean {
    if (this.connections.size >= this.maxConnections) {
      return false;
    }
    
    this.connections.set(wsId, ws);
    return true;
  }
  
  remove(wsId: string): void {
    this.connections.delete(wsId);
    
    // 从所有组中移除
    this.groups.forEach((members, groupId) => {
      members.delete(wsId);
      if (members.size === 0) {
        this.groups.delete(groupId);
      }
    });
  }
  
  get(wsId: string): WebSocket | undefined {
    return this.connections.get(wsId);
  }
  
  addToGroup(groupId: string, wsId: string): void {
    if (!this.groups.has(groupId)) {
      this.groups.set(groupId, new Set());
    }
    this.groups.get(groupId)!.add(wsId);
  }
  
  removeFromGroup(groupId: string, wsId: string): void {
    const members = this.groups.get(groupId);
    if (members) {
      members.delete(wsId);
      if (members.size === 0) {
        this.groups.delete(groupId);
      }
    }
  }
  
  broadcastToGroup(groupId: string, message: any): void {
    const members = this.groups.get(groupId);
    if (!members) return;
    
    const data = typeof message === 'string' ? message : JSON.stringify(message);
    
    members.forEach(wsId => {
      const ws = this.connections.get(wsId);
      if (ws && ws.readyState === WebSocket.OPEN) {
        ws.send(data);
      }
    });
  }
  
  private cleanup(): void {
    const toRemove: string[] = [];
    
    this.connections.forEach((ws, wsId) => {
      if (ws.readyState !== WebSocket.OPEN) {
        toRemove.push(wsId);
      }
    });
    
    toRemove.forEach(wsId => this.remove(wsId));
    
    console.log(`Cleaned up ${toRemove.length} invalid connections`);
  }
  
  getStats() {
    return {
      totalConnections: this.connections.size,
      totalGroups: this.groups.size,
      membersPerGroup: Array.from(this.groups.entries()).map(
        ([id, members]) => ({ groupId: id, count: members.size })
      ),
    };
  }
}

const pool = new OptimizedConnectionPool();
```

### 消息批处理

```typescript
// server/batch/batchProcessor.ts
interface BatchConfig {
  maxSize: number;       // 最大批次大小
  maxWait: number;       // 最大等待时间（毫秒）
}

class BatchProcessor<T> {
  private queue: T[] = [];
  private handlers: Array<(batch: T[]) => Promise<void>> = [];
  private timers: Map<string, NodeJS.Timeout> = new Map();
  
  constructor(private config: BatchConfig) {}
  
  add(item: T): void {
    this.queue.push(item);
    
    // 如果达到最大大小，立即处理
    if (this.queue.length >= this.config.maxSize) {
      this.flush();
      return;
    }
    
    // 设置延迟刷新计时器
    if (!this.timers.has('flush')) {
      const timer = setTimeout(() => this.flush(), this.config.maxWait);
      this.timers.set('flush', timer);
    }
  }
  
  onBatch(handler: (batch: T[]) => Promise<void>): void {
    this.handlers.push(handler);
  }
  
  private async flush(): void {
    if (this.queue.length === 0) return;
    
    // 清除计时器
    const timer = this.timers.get('flush');
    if (timer) {
      clearTimeout(timer);
      this.timers.delete('flush');
    }
    
    // 获取当前批次并清空队列
    const batch = this.queue.splice(0, this.config.maxSize);
    
    // 并行处理所有处理器
    await Promise.all(
      this.handlers.map(handler => handler(batch))
    );
  }
}

// 使用批处理器优化消息发送
const outboundBatch = new BatchProcessor<{ ws: WebSocket; data: any }>({
  maxSize: 100,
  maxWait: 50, // 50ms 内累积
});

outboundBatch.onBatch(async (batch) => {
  // 按连接分组
  const byConnection = new Map<WebSocket, any[]>();
  
  batch.forEach(({ ws, data }) => {
    if (!byConnection.has(ws)) {
      byConnection.set(ws, []);
    }
    byConnection.get(ws)!.push(data);
  });
  
  // 批量发送
  byConnection.forEach((messages, ws) => {
    if (ws.readyState === WebSocket.OPEN) {
      ws.send(JSON.stringify({
        type: 'batch',
        messages,
      }));
    }
  });
});

// 处理入站消息
const inboundBatch = new BatchProcessor<{ wsId: string; data: any }>({
  maxSize: 50,
  maxWait: 10, // 10ms 内累积
});

inboundBatch.onBatch(async (batch) => {
  // 批量处理数据库操作
  await db.$transaction(
    batch.map(({ wsId, data }) => 
      db.message.create({
        data: {
          connectionId: wsId,
          payload: data,
        },
      })
    )
  );
});
```

---

## WebSocket 故障排除

### 常见问题与解决方案

```typescript
// 问题 1: 连接频繁断开
// 原因: 心跳配置不当或网络不稳定

// 解决方案: 优化心跳配置
const wss = new WebSocketServer({ 
  port: 8080,
  clientTracking: true,
});

wss.on('connection', (ws: AuthenticatedWebSocket, req) => {
  ws.isAlive = true;
  
  // Ping-Pong 机制
  ws.on('pong', () => {
    ws.isAlive = true;
  });
  
  // 定期 ping 所有客户端
  const pingInterval = setInterval(() => {
    wss.clients.forEach((ws) => {
      if ((ws as AuthenticatedWebSocket).isAlive === false) {
        return ws.terminate();
      }
      
      (ws as AuthenticatedWebSocket).isAlive = false;
      ws.ping();
    });
  }, 30000);
  
  ws.on('close', () => {
    clearInterval(pingInterval);
  });
});

// 问题 2: 内存泄漏
// 原因: 未清理的事件监听器或缓存积累

// 解决方案: 显式清理
wss.on('connection', (ws: WebSocket, req) => {
  const cleanup = () => {
    // 移除所有监听器
    ws.removeAllListeners();
    
    // 清理缓存
    connectionCache.delete(req.url);
    
    // 清理定时器
    clearTimeout(heartbeatTimer);
    
    console.log('Connection cleaned up');
  };
  
  ws.on('close', cleanup);
  ws.on('error', cleanup);
});

// 问题 3: 消息顺序错乱
// 原因: 并发处理导致消息乱序

// 解决方案: 序列号机制
interface OrderedMessage {
  sequence: number;
  data: any;
}

const pendingMessages = new Map<string, Map<number, any>>();
const lastProcessed = new Map<string, number>();

function processOrderedMessage(
  connectionId: string, 
  message: OrderedMessage
) {
  if (!pendingMessages.has(connectionId)) {
    pendingMessages.set(connectionId, new Map());
  }
  
  const pending = pendingMessages.get(connectionId)!;
  pending.set(message.sequence, message.data);
  
  let expected = lastProcessed.get(connectionId) || 0;
  
  // 按顺序处理消息
  while (pending.has(expected + 1)) {
    const data = pending.get(expected + 1)!;
    pending.delete(expected + 1);
    expected++;
    
    // 处理消息
    handleMessage(data);
  }
  
  lastProcessed.set(connectionId, expected);
}
```

### 调试工具

```typescript
// server/debug/wsDebugger.ts
import { WebSocket, WebSocketServer } from 'ws';

interface DebugConfig {
  logConnections: boolean;
  logMessages: boolean;
  logErrors: boolean;
  maxMessageSize: number;  // 限制日志中的消息大小
}

class WebSocketDebugger {
  private config: DebugConfig;
  private inspector: any;  // 可以是 Chrome DevTools Protocol
  
  constructor(config: DebugConfig) {
    this.config = config;
    this.setupInspector();
  }
  
  logConnection(ws: WebSocket, req: any, action: 'connect' | 'disconnect') {
    if (!this.config.logConnections) return;
    
    const info = {
      timestamp: new Date().toISOString(),
      action,
      remoteAddress: req.socket.remoteAddress,
      url: req.url,
      readyState: ws.readyState,
    };
    
    console.log('[WS Connection]', JSON.stringify(info));
    
    // 发送到调试面板
    this.sendToInspector('connection', info);
  }
  
  logMessage(ws: WebSocket, direction: 'in' | 'out', data: any) {
    if (!this.config.logMessages) return;
    
    // 限制日志大小
    let logData = data;
    if (typeof data === 'string' && data.length > this.config.maxMessageSize) {
      logData = data.substring(0, this.config.maxMessageSize) + '...[truncated]';
    }
    
    const info = {
      timestamp: new Date().toISOString(),
      direction,
      data: logData,
      size: typeof data === 'string' ? data.length : data.byteLength,
    };
    
    console.log(`[WS ${direction === 'in' ? 'Recv' : 'Send'}]`, JSON.stringify(info));
    
    this.sendToInspector('message', info);
  }
  
  logError(ws: WebSocket, error: Error) {
    if (!this.config.logErrors) return;
    
    const info = {
      timestamp: new Date().toISOString(),
      error: {
        message: error.message,
        stack: error.stack,
      },
      readyState: ws.readyState,
    };
    
    console.error('[WS Error]', JSON.stringify(info));
    
    this.sendToInspector('error', info);
  }
  
  private sendToInspector(type: string, data: any) {
    // 发送到 WebSocket 调试端口
    // 或通过 IPC 发送到主进程
  }
  
  private setupInspector() {
    // 初始化调试协议
  }
}

const debugger_ = new WebSocketDebugger({
  logConnections: true,
  logMessages: true,
  logErrors: true,
  maxMessageSize: 1000,
});

// 应用到 WebSocket 服务器
wss.on('connection', (ws, req) => {
  debugger_.logConnection(ws, req, 'connect');
  
  ws.on('message', (data) => {
    debugger_.logMessage(ws, 'in', data.toString());
  });
  
  ws.on('error', (error) => {
    debugger_.logError(ws, error);
  });
});
```

---

## WebSocket 最佳实践总结

### 开发环境配置

```typescript
// config/ws.dev.ts
export const wsConfig = {
  server: {
    port: 8080,
    path: '/ws',
    maxConnections: 100,
    verifyClient: null, // 开发环境跳过验证
  },
  
  heartbeat: {
    enabled: true,
    interval: 30000,
    timeout: 10000,
  },
  
  logging: {
    level: 'debug',
    logConnections: true,
    logMessages: true,
    maxMessageSize: 500,
  },
  
  performance: {
    throttle: 50,
    batchSize: 10,
    batchTimeout: 100,
  },
  
  redis: {
    enabled: false, // 开发环境禁用 Redis
  },
};
```

### 生产环境配置

```typescript
// config/ws.prod.ts
export const wsConfig = {
  server: {
    port: 8080,
    path: '/ws',
    maxConnections: 10000,
    verifyClient: verifyToken, // 生产环境必须验证
  },
  
  heartbeat: {
    enabled: true,
    interval: 30000,
    timeout: 5000,
  },
  
  logging: {
    level: 'warn',
    logConnections: true,
    logMessages: false, // 生产环境禁用消息日志
    maxMessageSize: 0,
  },
  
  performance: {
    throttle: 100,
    batchSize: 50,
    batchTimeout: 50,
  },
  
  redis: {
    enabled: true,
    cluster: [
      { host: '10.0.0.1', port: 6379 },
      { host: '10.0.0.2', port: 6379 },
      { host: '10.0.0.3', port: 6379 },
    ],
  },
  
  security: {
    enableRateLimit: true,
    maxConnectionsPerIp: 5,
    maxMessagesPerSecond: 50,
    enableEncryption: true,
  },
};
```

### 安全检查清单

```markdown
## WebSocket 安全检查清单

- [ ] 实现 JWT/token 认证
- [ ] 配置速率限制
- [ ] 验证客户端输入
- [ ] 加密敏感消息
- [ ] 配置 CORS
- [ ] 实现心跳检测
- [ ] 记录审计日志
- [ ] 监控异常行为
- [ ] 设置连接超时
- [ ] 防止 XSS 攻击
```

### 性能优化清单

```markdown
## WebSocket 性能优化清单

- [ ] 使用连接池
- [ ] 实现消息批处理
- [ ] 配置心跳间隔
- [ ] 启用压缩
- [ ] 使用 Redis 分布式
- [ ] 监控连接数
- [ ] 设置背压机制
- [ ] 优化消息序列化
- [ ] 定期清理无效连接
- [ ] 配置自动伸缩
```

---

> [!TIP]
> WebSocket 是构建实时应用的基础技术，但并非所有场景都需要它。对于简单的单向更新，Server-Sent Events 可能是更好的选择。选择正确的技术可以简化实现并提高性能。
