# Liveblocks：实时协作基础设施的深度解析

> [!NOTE]
> 本文袼最后更新于 **2026年4月**，深入剖析 Liveblocks 的核心功能、Presence/Storage/Threads 机制，以及与 Convex/Yjs 的对比分析。

---

## 目录

1. [[#概述与核心概念]]
2. [[#核心功能详解]]
3. [[#Presence 实时状态]]
4. [[#Storage 持久存储]]
5. [[#Threads 评论区]]
6. [[#与其他方案对比]]
7. [[#AI 应用场景]]
8. [[#实战开发指南]]
9. [[#选型建议]]

---

## 概述与核心概念

### 什么是 Liveblocks

Liveblocks 是一个专为构建实时协作应用设计的 infrastructure 平台。与自建 WebSocket 服务不同，Liveblocks 提供了完整的协作能力抽象，包括用户在线状态（Presence）、共享文档存储（Storage）、评论系统（Threads）等。

Liveblocks 的核心理念：**让开发者专注于产品逻辑，而不是实时协作的基础设施**。

### 核心功能矩阵

| 功能 | 说明 | 典型场景 |
|------|------|---------|
| **Presence** | 用户在线状态、游标位置 | 多人编辑、在线用户 |
| **Storage** | 持久化共享文档 | 协作文档、白板 |
| **Threads** | 评论/讨论系统 | 文档评论、反馈 |
| **Broadcast** | 广播消息 | 通知、实时事件 |
| **Liveblocks AI** | AI 辅助功能 | AI 审查、自动补全 |

### 支持的平台

| 平台 | SDK | 状态 |
|------|-----|------|
| **React** | `@liveblocks/react` | ⭐⭐⭐⭐⭐ 官方最佳 |
| **React Native** | `@liveblocks/react-native` | ⭐⭐⭐⭐ 官方 |
| **Vue** | `@liveblocks/vue` | ⭐⭐⭐ 社区 |
| **Svelte** | `@liveblocks/svelte` | ⭐⭐⭐ 社区 |
| **JavaScript** | `@liveblocks/client` | ⭐⭐⭐⭐ 官方 |
| **Node.js** | `@liveblocks/node` | ⭐⭐⭐⭐ 官方 |
| **Python** | `liveblocks-python` | ⭐⭐⭐ 社区 |

---

## 核心功能详解

### 安装与配置

```bash
# 安装
npm install @liveblocks/client @liveblocks/react

# 安装 Liveblocks CLI
npm install -g @liveblocks/cli

# 初始化
npx liveblocks init
```

```typescript
// liveblocks.config.ts
import { createClient } from '@liveblocks/client'
import { createRoomContext } from '@liveblocks/react'

const client = createClient({
  publicApiKey: 'pk_dev_xxxxx',
  throttle: 100,  // 节流更新
})

export const {
  RoomProvider,
  useRoom,
  useMyPresence,
  useUpdateMyPresence,
  useOthers,
  useStorage,
  useMutation,
} = createRoomContext(client)
```

---

## Presence 实时状态

### 什么是 Presence

Presence 是 Liveblocks 提供的用户实时状态同步能力。它用于在用户加入房间时共享其当前状态：

| 典型 Presence 数据 | 说明 |
|------------------|------|
| **在线状态** | 用户是否在线 |
| **游标位置** | 鼠标/手指位置 |
| **选择内容** | 当前选中的文本/元素 |
| **当前编辑位置** | 光标所在行 |
| **用户信息** | 头像、名称、颜色 |

### Presence 基础用法

```tsx
// types.ts
declare global {
  interface Liveblocks {
    // 定义 Presence 类型
    Presence: {
      cursor: { x: number; y: number } | null
      selectedId: string | null
      user: {
        id: string
        name: string
        color: string
        avatar?: string
      }
    }
    
    // 定义 Storage 类型
    Storage: {
      document: LiveObject<{
        title: string
        content: string
      }>
    }
    
    // 定义 UserMeta
    UserMeta: {
      id: string
      info: {
        name: string
        avatar: string
        color: string
      }
    }
    
    // 定义广播事件
    RoomEvent: {
      type: 'NOTIFICATION'
      message: string
    }
  }
}
```

```tsx
// components/Room.tsx
import { RoomProvider } from '../liveblocks.config'
import { LiveMap } from 'liveblocks-client'

export function Room({ roomId, user }: { roomId: string; user: any }) {
  return (
    <RoomProvider
      id={roomId}
      initialPresence={{
        cursor: null,
        selectedId: null,
        user: {
          id: user.id,
          name: user.name,
          color: user.color,
        },
      }}
      initialStorage={{
        document: new LiveObject({
          title: '',
          content: '',
        }),
      }}
    >
      <CollaborationRoom />
    </RoomProvider>
  )
}
```

### 游标同步

```tsx
// components/CursorOverlay.tsx
import { useOthers, useMyPresence } from '../liveblocks.config'

export function CursorOverlay() {
  const [myPresence, updateMyPresence] = useMyPresence()
  const others = useOthers()
  
  return (
    <div
      className="absolute inset-0 pointer-events-none"
      onMouseMove={(e) => {
        updateMyPresence({
          cursor: { x: e.clientX, y: e.clientY }
        })
      }}
      onMouseLeave={() => {
        updateMyPresence({ cursor: null })
      }}
    >
      {/* 自己的游标 */}
      {myPresence.cursor && (
        <Cursor color={myPresence.user.color}>
          {myPresence.user.name}
        </Cursor>
      )}
      
      {/* 其他人的游标 */}
      {others.map(({ connectionId, presence }) => (
        presence.cursor && (
          <Cursor key={connectionId} color={presence.user.color}>
            {presence.user.name}
          </Cursor>
        )
      ))}
    </div>
  )
}
```

---

## Storage 持久存储

### Liveblocks Storage 数据结构

```typescript
// Liveblocks 支持的数据结构
import { LiveObject, LiveList, LiveMap, LiveBlock } from 'liveblocks'

// LiveObject: 类似普通对象，但支持协同编辑
const doc = new LiveObject({
  title: 'My Document',
  meta: {
    createdAt: Date.now(),
    author: 'user-123'
  }
})

// LiveList: 有序列表
const items = new LiveList(['item1', 'item2', 'item3'])

// LiveMap: 键值对映射
const users = new LiveMap([
  ['user-1', { name: 'Alice', score: 100 }],
  ['user-2', { name: 'Bob', score: 200 }]
])
```

### 协作文档

```tsx
// components/Editor.tsx
import { useStorage, useMutation } from '../liveblocks.config'
import { LiveObject } from 'liveblocks-client'

export function CollaborativeEditor() {
  // 读取 Storage（响应式）
  const doc = useStorage((root) => root.document)
  
  // 写入 Storage
  const updateTitle = useMutation((ctx, newTitle: string) => {
    const doc = ctx.root.get('document')
    doc.set('title', newTitle)
    doc.set('updatedAt', Date.now())
  }, [])
  
  const updateContent = useMutation((ctx, newContent: string) => {
    const doc = ctx.root.get('document')
    doc.set('content', newContent)
  }, [])
  
  if (!doc) return <div>Loading...</div>
  
  return (
    <div>
      <input
        value={doc.get('title')}
        onChange={(e) => updateTitle(e.target.value)}
        placeholder="Document Title"
      />
      <textarea
        value={doc.get('content')}
        onChange={(e) => updateContent(e.target.value)}
        placeholder="Start writing..."
      />
      <div className="text-sm text-gray-500">
        Last updated: {new Date(doc.get('updatedAt')).toLocaleString()}
      </div>
    </div>
  )
}
```

### 协同白板

```tsx
// types.ts
declare global {
  interface Liveblocks {
    Storage: {
      canvas: LiveObject<{
        elements: LiveList<LiveObject<CanvasElement>>
      }>
    }
  }
}

interface CanvasElement {
  id: string
  type: 'rect' | 'circle' | 'text' | 'image'
  x: number
  y: number
  width?: number
  height?: number
  color: string
  text?: string
  imageUrl?: string
}
```

```tsx
// components/Canvas.tsx
import { useStorage, useMutation } from '../liveblocks.config'
import { LiveObject, LiveList } from 'liveblocks-client'

export function Canvas() {
  const elements = useStorage((root) => 
    root.canvas.get('elements')
  )
  
  const addElement = useMutation((ctx, element: CanvasElement) => {
    const elements = ctx.root.get('canvas').get('elements')
    elements.push(new LiveObject(element))
  }, [])
  
  const updateElement = useMutation((ctx, id: string, updates: Partial<CanvasElement>) => {
    const elements = ctx.root.get('canvas').get('elements')
    const element = elements.find((el) => el.get('id') === id)
    if (element) {
      Object.entries(updates).forEach(([key, value]) => {
        element.set(key, value)
      })
    }
  }, [])
  
  const deleteElement = useMutation((ctx, id: string) => {
    const elements = ctx.root.get('canvas').get('elements')
    const index = elements.findIndex((el) => el.get('id') === id)
    if (index !== -1) {
      elements.delete(index)
    }
  }, [])
  
  return (
    <div className="canvas">
      {elements?.map((element, index) => (
        <CanvasElement
          key={element.get('id')}
          element={element}
          onUpdate={(updates) => updateElement(element.get('id'), updates)}
          onDelete={() => deleteElement(element.get('id'))}
        />
      ))}
      <button onClick={() => addElement({
        id: crypto.randomUUID(),
        type: 'rect',
        x: 100,
        y: 100,
        width: 100,
        height: 100,
        color: '#3b82f6'
      })}>
        Add Rectangle
      </button>
    </div>
  )
}
```

---

## Threads 评论区

### Threads 功能

Liveblocks Threads 提供了完整的评论系统，支持：

| 功能 | 说明 |
|------|------|
| **房间内评论** | 针对整个文档的评论 |
| **锚点评论** | 针对特定内容的评论 |
| **@提及** | 提及用户 |
| **已解决/未解决** | 评论状态管理 |
| **通知** | 邮件/Webhook 通知 |

### 评论实现

```tsx
// components/Comments.tsx
import { Thread, useThreads } from '@liveblocks/react'
import { Composer, useCreateThread } from '@liveblocks/react'

export function Comments() {
  const { threads } = useThreads()
  
  return (
    <div className="comments-panel">
      <h2>Comments ({threads.length})</h2>
      
      {/* 新建评论 */}
      <Composer
        onSubmit={({ body, anchors }) => {
          // 创建新评论
          createThread({ body, anchors })
        }}
      />
      
      {/* 评论列表 */}
      {threads.map((thread) => (
        <Thread
          key={thread.id}
          thread={thread}
          indentCommentComponents={Comment}
        />
      ))}
    </div>
  )
}

function Comment({
  thread,
  comment,
}: {
  thread: ThreadMetadata
  comment: CommentMetadata
}) {
  return (
    <div className="comment">
      <div className="comment-header">
        <img src={comment.user.info.avatar} alt={comment.user.info.name} />
        <span>{comment.user.info.name}</span>
        <span>{new Date(comment.createdAt).toLocaleString()}</span>
      </div>
      <div className="comment-body">
        {comment.body}
      </div>
      <div className="comment-actions">
        <button>Reply</button>
        <button>Resolve</button>
      </div>
    </div>
  )
}
```

---

## 与其他方案对比

### 功能矩阵

| 功能 | Liveblocks | Convex | Yjs + y-websocket |
|------|------------|--------|-------------------|
| **实时同步** | ✅ | ✅ | ✅ |
| **Presence** | ✅ 原生支持 | ✅ | ⚠️ 需扩展 |
| **Storage** | ✅ CRDT | ✅ 实时数据库 | ⚠️ 需扩展 |
| **评论系统** | ✅ 原生 | ❌ | ❌ |
| **数据类型** | LiveObject | TypeScript | Yjs Types |
| **离线支持** | ✅ | ⚠️ | ✅ |
| **冲突解决** | CRDT 自动 | 乐观更新 | CRDT |
| **学习曲线** | 低 | 低 | 高 |
| **定价** | 按座位计费 | 按量计费 | 免费（自托管）|

### CRDT vs 乐观更新

```
┌─────────────────────────────────────────────────────────┐
│              冲突解决机制对比                             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Liveblocks (CRDT)                                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │  User A: "Hello"           User B: "World"      │   │
│  │         ↓                          ↓           │   │
│  │    [H][e][l][l][o]        [W][o][r][l][d]     │   │
│  │         ↓                          ↓           │   │
│  │    ┌─────────────────────────────┐             │   │
│  │    │     自动合并: "HelloWorld"   │             │   │
│  │    └─────────────────────────────┘             │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  Convex (乐观更新)                                      │
│  ┌─────────────────────────────────────────────────┐   │
│  │  User A: 编辑            User B: 编辑             │   │
│  │         ↓                          ↓           │   │
│  │    本地立即更新           本地立即更新            │   │
│  │         ↓                          ↓           │   │
│  │    服务器广播             服务器广播              │   │
│  │         ↓                          ↓           │   │
│  │    最后写入胜出（Last Write Wins）              │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 选型对比

| 场景 | 推荐 | 理由 |
|------|------|------|
| 协作文档/白板 | **Liveblocks** | CRDT + 完整功能 |
| 实时聊天应用 | **Convex** | 实时数据库 + 函数 |
| 简单多人游戏 | **Yjs** | 轻量 + 高性能 |
| 需要评论系统 | **Liveblocks** | 原生 Threads |
| 预算有限 | **Yjs** | 完全免费（自托管）|

---

## AI 应用场景

### Liveblocks AI 集成

Liveblocks 提供了 AI 辅助功能：

```tsx
// components/AIAssistant.tsx
import { AiTextEditor } from '@liveblocks/react-ai'
import { LiveObject } from 'liveblocks-client'

export function AIAssistant() {
  return (
    <AiTextEditor
      Prompt={
        <div>
          <h3>AI Writing Assistant</h3>
          <p>Select text and use AI to:</p>
          <ul>
            <li>Fix grammar</li>
            <li>Improve clarity</li>
            <li>Change tone</li>
            <li>Summarize</li>
          </ul>
        </div>
      }
      defaultCompletion="Suggest improvements to your text..."
      LiveblocksConfiguration={liveblocksConfig}
    />
  )
}
```

### AI 审查评论

```tsx
// components/AIReview.tsx
import { useMutation, useStorage } from '../liveblocks.config'
import { openai } from '@liveblocks/azure-openai'

export function CommentReview() {
  const threads = useThreads()
  
  const analyzeComment = useMutation(async (ctx, commentId: string) => {
    const comment = threads.find(t => t.id === commentId)
    if (!comment) return
    
    // 使用 AI 分析评论
    const analysis = await openai.chat.completions.create({
      model: 'gpt-4',
      messages: [{
        role: 'system',
        content: 'Analyze this comment for tone, sentiment, and suggestions.'
      }, {
        role: 'user',
        content: comment.body
      }]
    })
    
    return analysis.choices[0].message.content
  }, [])
  
  return (
    <div>
      {threads.map(thread => (
        <div key={thread.id}>
          <Thread thread={thread} />
          <button onClick={() => analyzeComment(thread.id)}>
            AI Analyze
          </button>
        </div>
      ))}
    </div>
  )
}
```

---

## 实战开发指南

### 完整示例：协作文本编辑器

```tsx
// app/page.tsx
import { ClientSideSuspense } from '@liveblocks/react'
import { RoomProvider } from '../liveblocks.config'
import { CollaborativeEditor } from '../components/Editor'
import { Users } from '../components/Users'
import { Comments } from '../components/Comments'

export default function DocumentPage({ params }: { params: { id: string } }) {
  return (
    <RoomProvider
      id={`document-${params.id}`}
      initialPresence={{
        cursor: null,
        selectedId: null,
        user: null,
      }}
      initialStorage={{
        document: new LiveObject({
          title: '',
          content: '',
          createdAt: Date.now(),
        }),
      }}
    >
      <ClientSideSuspense fallback={<div>Loading...</div>}>
        <div className="app-layout">
          <header>
            <CollaborativeEditor />
          </header>
          <aside>
            <Users />
            <Comments />
          </aside>
        </div>
      </ClientSideSuspense>
    </RoomProvider>
  )
}
```

### 错误处理

```tsx
// components/ErrorBoundary.tsx
import { RoomProvider, ErrorBoundary as LiveblocksErrorBoundary } from '@liveblocks/react'

export function SafeRoom({ roomId, children }: { roomId: string; children: React.ReactNode }) {
  return (
    <LiveblocksErrorBoundary
      fallback={(error) => (
        <div>
          <h2>Something went wrong</h2>
          <p>{error.message}</p>
          <button onClick={() => window.location.reload()}>
            Reload
          </button>
        </div>
      )}
    >
      <RoomProvider id={roomId}>
        {children}
      </RoomProvider>
    </LiveblocksErrorBoundary>
  )
}
```

---

## 选型建议

### 选择 Liveblocks 的条件

- ✅ 需要协作文档/白板
- ✅ 需要 Presence 功能
- ✅ 需要评论系统
- ✅ React 项目
- ✅ 想要快速上线

### 不选择 Liveblocks 的场景

- ❌ 需要完全免费（考虑 Yjs 自托管）
- ❌ 非 React 项目（考虑 Convex）
- ❌ 需要复杂查询（考虑 Convex）
- ❌ 预算有限（考虑 Yjs）

---

## 参考资料

| 资源 | 链接 |
|------|------|
| 官方文档 | https://liveblocks.io/docs |
| GitHub | https://github.com/liveblocks/liveblocks |
| Playground | https://liveblocks.io/examples |
| Discord | https://liveblocks.io/community |
| 定价页面 | https://liveblocks.io/pricing |

---

> [!SUCCESS]
> Liveblocks 是构建实时协作应用的完美选择。其 CRDT 基础的冲突解决、内置的 Presence 和评论系统，使开发者能够快速构建 Figma、Notion 风格的协作产品。对于 vibecoding 工作流中的协作功能开发，Liveblocks 提供了开箱即用的完整解决方案。

---

## Liveblocks 高级配置与最佳实践

### 完整的房间配置

```typescript
// liveblocks.config.ts
import { createClient } from '@liveblocks/client';
import { createRoomContext } from '@liveblocks/react';

export const client = createClient({
  // 公开密钥
  publicApiKey: process.env.NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY!,
  
  // 认证
  authEndpoint: '/api/liveblocks-auth',
  
  // throttle（节流）
  throttle: 16,
  
  // 离线支持
  offlineSupport: 'ENTER_SNAPSHOT',
});

// 类型定义
type Presence = {
  cursor: { x: number; y: number } | null;
  selectedIds: string[];
  currentView: 'canvas' | 'list';
};

type Storage = {
  canvas: LiveList<Shape>;
  document: Document;
};

type UserMeta = {
  id: string;
  info: {
    name: string;
    avatar: string;
    color: string;
  };
};

type RoomEvent = {
  type: 'COMMENT_ADDED' | 'SHAPE_DELETED';
  data: Record<string, unknown>;
};

// 创建房间上下文
export const {
  RoomProvider,       // 房间提供者
  useRoom,           // 获取房间信息
  useMyPresence,     // 我的状态
  useOthers,         // 其他用户状态
  useSelf,           // 当前用户
  useStorage,         // 持久化存储
  useMutation,        // 修改存储
  useBroadcastEvent,  // 广播事件
  useEventListener,   // 监听事件
} = createRoomContext<Presence, Storage, UserMeta, RoomEvent>(client);
```

### Presence 系统深度使用

```typescript
// components/CollaborativeCursor.tsx
'use client';

import { useOthers, useMyPresence } from '@/liveblocks.config';

export function CollaborativeCursors() {
  const others = useOthers();
  const [, updateMyPresence] = useMyPresence();
  
  return (
    <>
      {others.map(({ connectionId, presence, info }) => (
        <div
          key={connectionId}
          className="cursor"
          style={{
            transform: `translate(${presence.cursor?.x || 0}px, ${presence.cursor?.y || 0}px)`,
          }}
        >
          <div className="cursor-pointer">
            <svg width="24" height="24" viewBox="0 0 24 24">
              <path
                d="M5.5 3.21V20.79c0 .45.54.67.85.35l4.86-4.86a.5.5 0 0 1 .35-.15h6.87a.5.5 0 0 0 .35-.85L6.35 2.86a.5.5 0 0 0-.85.35Z"
                fill={info?.color || '#666'}
              />
            </svg>
            <span style={{ background: info?.color || '#666' }}>
              {info?.name || 'Anonymous'}
            </span>
          </div>
        </div>
      ))}
    </>
  );
}

// 跟踪鼠标位置
export function CursorTracker() {
  const [, updateMyPresence] = useMyPresence();
  
  const handleMouseMove = useCallback((e: MouseEvent) => {
    updateMyPresence({
      cursor: { x: e.clientX, y: e.clientY },
    });
  }, [updateMyPresence]);
  
  const handleMouseLeave = useCallback(() => {
    updateMyPresence({
      cursor: null,
    });
  }, [updateMyPresence]);
  
  useEffect(() => {
    document.addEventListener('mousemove', handleMouseMove);
    document.addEventListener('mouseleave', handleMouseLeave);
    
    return () => {
      document.removeEventListener('mousemove', handleMouseMove);
      document.removeEventListener('mouseleave', handleMouseLeave);
    };
  }, [handleMouseMove, handleMouseLeave]);
  
  return null;
}
```

### Storage 持久化存储

```typescript
// components/Canvas.tsx
'use client';

import { useStorage, useMutation } from '@/liveblocks.config';
import { LiveList, LiveObject } from '@liveblocks/client';

export function Canvas() {
  const shapes = useStorage((root) => root.canvas);
  const document = useStorage((root) => root.document);
  
  const addShape = useMutation(({ storage }, shape: Shape) => {
    const canvas = storage.get('canvas');
    canvas.push(new LiveObject(shape));
  }, []);
  
  const updateShape = useMutation(({ storage }, id: string, updates: Partial<Shape>) => {
    const canvas = storage.get('canvas');
    const shape = canvas.find((s) => s.get('id') === id);
    if (shape) {
      Object.entries(updates).forEach(([key, value]) => {
        shape.set(key as keyof Shape, value);
      });
    }
  }, []);
  
  const deleteShape = useMutation(({ storage }, id: string) => {
    const canvas = storage.get('canvas');
    const index = canvas.findIndex((s) => s.get('id') === id);
    if (index !== -1) {
      canvas.delete(index);
    }
  }, []);
  
  const clearCanvas = useMutation(({ storage }) => {
    const canvas = storage.get('canvas');
    while (canvas.length > 0) {
      canvas.delete(0);
    }
  }, []);
  
  return (
    <div className="canvas">
      {shapes.map((shape) => (
        <ShapeComponent
          key={shape.get('id')}
          shape={shape}
          onUpdate={(updates) => updateShape(shape.get('id'), updates)}
          onDelete={() => deleteShape(shape.get('id'))}
        />
      ))}
      <button onClick={() => addShape(createNewShape())}>Add Shape</button>
    </div>
  );
}
```

### Liveblocks 与 React 状态管理

```typescript
// hooks/useRoomStore.ts
import { create } from 'zustand';
import { useStorage, useMutation } from '@/liveblocks.config';

export function useSelectionStore() {
  const selectedIds = useMyPresence((presence) => presence.selectedIds);
  const updateMyPresence = useMyPresence()[1];
  
  const select = (id: string) => {
    updateMyPresence((prev) => ({
      ...prev,
      selectedIds: [...prev.selectedIds, id],
    }));
  };
  
  const deselect = (id: string) => {
    updateMyPresence((prev) => ({
      ...prev,
      selectedIds: prev.selectedIds.filter((i) => i !== id),
    }));
  };
  
  const clearSelection = () => {
    updateMyPresence((prev) => ({
      ...prev,
      selectedIds: [],
    }));
  };
  
  return { selectedIds, select, deselect, clearSelection };
}

// 工具栏
export function Toolbar() {
  const { selectedIds, clearSelection } = useSelectionStore();
  const shapes = useStorage((root) => root.canvas);
  const updateShape = useMutation(/* ... */);
  
  const deleteSelected = () => {
    selectedIds.forEach((id) => {
      // 删除逻辑
    });
    clearSelection();
  };
  
  const copySelected = async () => {
    const selectedShapes = shapes.filter((s) => 
      selectedIds.includes(s.get('id'))
    );
    await navigator.clipboard.writeText(JSON.stringify(selectedShapes));
  };
  
  return (
    <div className="toolbar">
      <button onClick={deleteSelected} disabled={selectedIds.length === 0}>
        Delete
      </button>
      <button onClick={copySelected} disabled={selectedIds.length === 0}>
        Copy
      </button>
    </div>
  );
}
```

### 实时评论系统

```typescript
// types.ts
type Comment = {
  id: string;
  userId: string;
  userName: string;
  userAvatar: string;
  body: string;
  createdAt: number;
  resolved: boolean;
  replies: Comment[];
  position?: { x: number; y: number }; // 如果是画布评论
};

type CommentsStorage = {
  comments: LiveList<LiveObject<Comment>>;
};

// components/CommentsPanel.tsx
'use client';

import { useStorage, useMutation } from '@/liveblocks.config';
import { LiveObject } from '@liveblocks/client';

export function CommentsPanel() {
  const comments = useStorage((root) => root.comments);
  
  const addComment = useMutation(({ storage }, comment: Omit<Comment, 'id'>) => {
    const commentsList = storage.get('comments');
    commentsList.push(
      new LiveObject({
        ...comment,
        id: crypto.randomUUID(),
        replies: [],
        resolved: false,
      })
    );
  }, []);
  
  const resolveComment = useMutation(({ storage }, commentId: string) => {
    const commentsList = storage.get('comments');
    const comment = commentsList.find((c) => c.get('id') === commentId);
    if (comment) {
      comment.set('resolved', true);
    }
  }, []);
  
  const addReply = useMutation(({ storage }, commentId: string, reply: Omit<Comment, 'id'>) => {
    const commentsList = storage.get('comments');
    const comment = commentsList.find((c) => c.get('id') === commentId);
    if (comment) {
      const replies = comment.get('replies') || [];
      replies.push({
        ...reply,
        id: crypto.randomUUID(),
      });
      comment.set('replies', [...replies]);
    }
  }, []);
  
  return (
    <div className="comments-panel">
      <h2>Comments ({comments.length})</h2>
      
      {comments.map((comment) => (
        <CommentCard
          key={comment.get('id')}
          comment={comment}
          onResolve={() => resolveComment(comment.get('id'))}
          onReply={(reply) => addReply(comment.get('id'), reply)}
        />
      ))}
    </div>
  );
}

// 画布评论标记
export function CommentMarkers() {
  const comments = useStorage((root) => root.comments);
  const others = useOthers();
  
  return (
    <>
      {comments.map((comment) => {
        const position = comment.get('position');
        if (!position) return null;
        
        return (
          <div
            key={comment.get('id')}
            className="comment-marker"
            style={{ left: position.x, top: position.y }}
          >
            <span className="marker-count">
              {comment.get('replies').length + 1}
            </span>
          </div>
        );
      })}
    </>
  );
}
```

### Liveblocks Auth 认证

```typescript
// pages/api/liveblocks-auth.ts
import { Liveblocks } from '@liveblocks/node';
import { currentUser } from '@clerk/nextjs';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

export async function POST(req: Request) {
  const user = await currentUser();
  
  if (!user) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  const body = await req.json();
  const { room } = body;
  
  // 准备用户信息
  const userInfo = {
    id: user.id,
    info: {
      name: user.fullName || user.emailAddresses[0]?.emailAddress,
      avatar: user.imageUrl,
      color: getRandomColor(),
    },
  };
  
  // 检查房间权限
  const hasAccess = await checkRoomAccess(user.id, room);
  
  if (!hasAccess) {
    return new Response('Forbidden', { status: 403 });
  }
  
  // 创建会话
  const session = liveblocks.prepareSession(user.id, {
    userInfo,
  });
  
  // 授予房间权限
  const roomAccess = getRoomAccess(user.id, room);
  Object.entries(roomAccess).forEach(([action, allowed]) => {
    if (allowed) {
      session.allow(action, session.resources);
    }
  });
  
  const { body: sessionBody, status } = await session.authorize();
  return new Response(sessionBody, { status });
}

// 检查房间访问权限
async function checkRoomAccess(userId: string, room: string): Promise<boolean> {
  // 实现你的权限检查逻辑
  const roomDoc = await db.room.findUnique({
    where: { id: room },
  });
  
  if (!roomDoc) return false;
  
  return roomDoc.collaborators.includes(userId);
}

// 获取房间权限
function getRoomAccess(userId: string, room: string) {
  return {
    [Liveblocks.ACTION_READ]: true,
    [Liveblocks.ACTION_UPDATE_PRESENCE]: true,
    [Liveblocks.ACTION_UPDATE_STORAGE]: true,
  };
}

// 随机颜色
function getRandomColor(): string {
  const colors = ['#E57373', '#64B5F6', '#81C784', '#FFD54F', '#BA68C8'];
  return colors[Math.floor(Math.random() * colors.length)];
}
```

### Liveblocks 与 tRPC 集成

```typescript
// server/router/liveblocks.ts
import { router, publicProcedure, protectedProcedure } from '../trpc';
import { z } from 'zod';
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

export const liveblocksRouter = router({
  // 识别用户
  identify: protectedProcedure
    .input(z.object({ room: z.string() }))
    .mutation(async ({ ctx, input }) => {
      const session = liveblocks.prepareSession(ctx.user.id, {
        userInfo: {
          name: ctx.user.name,
          avatar: ctx.user.avatar,
          color: ctx.user.color || '#666',
        },
      });
      
      session.allow(`${input.room}:presence:read`, session.resources);
      session.allow(`${input.room}:presence:write`, session.resources);
      session.allow(`${input.room}:storage:read`, session.resources);
      session.allow(`${input.room}:storage:write`, session.resources);
      session.allow(`${input.room}:event:write`, session.resources);
      
      const { body, status } = await session.authorize();
      
      return JSON.parse(body);
    }),
  
  // 获取房间活跃用户
  getActiveUsers: protectedProcedure
    .input(z.object({ room: z.string() }))
    .query(async ({ input }) => {
      const room = await liveblocks.getRoom(input.room);
      return {
        activeUsers: room.users,
        onlineCount: room.users.length,
      };
    }),
});
```

### Liveblocks 性能优化

```typescript
// 1. 使用 suspense 优化加载
import { Suspense } from 'react';

function App() {
  return (
    <RoomProvider id="my-room" initialPresence={/* ... */} initialStorage={/* ... */}>
      <Suspense fallback={<Loading />}>
        <Canvas />
      </Suspense>
    </RoomProvider>
  );
}

// 2. 选择性订阅
const shapes = useStorage((root) => 
  root.canvas.map((s) => ({  // 只选择需要的字段
    id: s.get('id'),
    type: s.get('type'),
    position: s.get('position'),
  }))
);

// 3. 使用 shallow 避免不必要更新
const myCursor = useMyPresence({
  shallow: true,
});

// 4. 批量更新
const batchUpdate = useMutation(({ storage }, updates: ShapeUpdate[]) => {
  storage.batch(() => {
    updates.forEach(({ id, changes }) => {
      const shape = storage.get('canvas').find((s) => s.get('id') === id);
      if (shape) {
        Object.entries(changes).forEach(([key, value]) => {
          shape.set(key, value);
        });
      }
    });
  });
}, []);
```

### Liveblocks 冲突解决

```typescript
// Liveblocks 使用 CRDT 自动解决冲突
// 但你可以通过回调自定义行为

// 使用 LiveObject 的 observe
const position = new LiveObject({ x: 0, y: 0 });

position.subscribe((value) => {
  console.log('Position changed:', value);
});

// 监听特定字段变化
position.subscribe((value) => {
  console.log('X changed:', value.x);
}, (value) => value.x);

// 事务操作
const moveShape = useMutation(({ storage }, id: string, dx: number, dy: number) => {
  storage.batch(() => {
    // 在事务中执行多个操作，确保原子性
    const shape = storage.get('canvas').find((s) => s.get('id') === id);
    if (shape) {
      shape.set('x', shape.get('x') + dx);
      shape.set('y', shape.get('y') + dy);
    }
  });
}, []);
```

### Liveblocks 与 Webhooks

```typescript
// pages/api/webhooks/liveblocks.ts
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

export async function POST(req: Request) {
  const body = await req.json();
  const { type, payload, timestamp } = body;
  
  switch (type) {
    case 'ASSET_CREATED': {
      // 处理新资源创建
      const { id, userId, roomId, type } = payload;
      console.log(`Asset ${id} created in room ${roomId}`);
      break;
    }
    
    case 'ASSET_DELETED': {
      // 处理资源删除
      break;
    }
    
    case 'ROOM_CREATED': {
      // 处理房间创建
      const { id, userId } = payload;
      await db.room.create({
        data: {
          id,
          ownerId: userId,
        },
      });
      break;
    }
    
    case 'USER_JOINED': {
      // 用户加入房间
      break;
    }
    
    case 'USER_LEFT': {
      // 用户离开房间
      break;
    }
  }
  
  return new Response('OK');
}
```

### Liveblocks 调试技巧

```typescript
// 1. 开发工具面板
import { LiveblocksDevTools } from '@liveblocks/client';

if (process.env.NODE_ENV === 'development') {
  // 在开发环境启用调试
  console.log('Liveblocks connection:', {
    room: roomId,
    presence: myPresence,
    storage: 'Loading...',
  });
}

// 2. 监控连接状态
function ConnectionStatus() {
  const room = useRoom();
  
  return (
    <div className={`status ${room.status}`}>
      {room.status === 'connected' ? '🟢' : 
       room.status === 'connecting' ? '🟡' : '🔴'}
      {room.status}
    </div>
  );
}

// 3. 查看在线用户
function OnlineUsers() {
  const others = useOthers();
  
  return (
    <div className="online-users">
      {others.map(({ connectionId, info }) => (
        <div key={connectionId} className="user-avatar">
          <img src={info?.avatar} alt={info?.name} />
        </div>
      ))}
    </div>
  );
}
```

### Liveblocks 离线支持

```typescript
// 配置离线支持
const client = createClient({
  publicApiKey: process.env.NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY!,
  
  // 离线支持模式
  offlineSupport: 'ENTER_SNAPSHOT',
  
  // 当重新连接时
  reconnectOnIdle: true,
});

// 检测离线状态
function OfflineBanner() {
  const room = useRoom();
  
  if (room.status === 'disconnected') {
    return (
      <div className="offline-banner">
        ⚠️ You're offline. Changes will sync when you reconnect.
      </div>
    );
  }
  
  return null;
}

// 手动同步
const syncStorage = useMutation(async ({ storage }) => {
  // 获取最新状态
  await storage.sync();
}, []);
```

---

---

## Liveblocks 高级安全配置

### 房间级权限控制

```typescript
// lib/liveblocks/permissions.ts
import { Liveblocks } from '@liveblocks/node';
import { auth, currentUser } from '@clerk/nextjs';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

export type RoomPermission = 'full_access' | 'can_edit' | 'can_view' | 'none';

export interface PermissionConfig {
  read: boolean;
  presence: boolean;
  updatePresence: boolean;
  updateStorage: boolean;
  deleteStorage: boolean;
  canComment: boolean;
}

export const PERMISSION_MATRIX: Record<RoomPermission, PermissionConfig> = {
  full_access: {
    read: true,
    presence: true,
    updatePresence: true,
    updateStorage: true,
    deleteStorage: true,
    canComment: true,
  },
  can_edit: {
    read: true,
    presence: true,
    updatePresence: true,
    updateStorage: true,
    deleteStorage: false,
    canComment: true,
  },
  can_view: {
    read: true,
    presence: true,
    updatePresence: false,
    updateStorage: false,
    deleteStorage: false,
    canComment: false,
  },
  none: {
    read: false,
    presence: false,
    updatePresence: false,
    updateStorage: false,
    deleteStorage: false,
    canComment: false,
  },
};

// 获取用户房间权限
export async function getUserRoomPermission(
  userId: string,
  roomId: string
): Promise<RoomPermission> {
  const db = await import('@/lib/db');
  
  const membership = await db.userRoomMembership.findUnique({
    where: {
      userId_roomId: { userId, roomId },
    },
  });
  
  if (!membership) return 'none';
  return membership.permission as RoomPermission;
}

// Liveblocks 认证处理
export async function handleLiveblocksAuth(
  roomId: string,
  userId: string
) {
  const user = await currentUser();
  
  if (!user) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  const permission = await getUserRoomPermission(userId, roomId);
  const config = PERMISSION_MATRIX[permission];
  
  if (permission === 'none') {
    return new Response('Forbidden', { status: 403 });
  }
  
  const session = liveblocks.prepareSession(userId, {
    userInfo: {
      id: user.id,
      name: user.fullName || user.emailAddresses[0]?.emailAddress,
      avatar: user.imageUrl,
      color: getUserColor(user.id),
    },
  });
  
  // 根据权限配置授予访问
  if (config.read) {
    session.allow(`${roomId}:presence:read`, session.resources);
  }
  if (config.updatePresence) {
    session.allow(`${roomId}:presence:write`, session.resources);
  }
  if (config.updateStorage) {
    session.allow(`${roomId}:storage:read`, session.resources);
    session.allow(`${roomId}:storage:write`, session.resources);
  }
  if (config.canComment) {
    session.allow(`${roomId}:thread:write`, session.resources);
  }
  
  const { body, status } = await session.authorize();
  return new Response(body, { status });
}

function getUserColor(userId: string): string {
  const colors = ['#E57373', '#64B5F6', '#81C784', '#FFD54F', '#BA68C8', '#4DB6AC', '#FF8A65'];
  const hash = userId.split('').reduce((acc, char) => acc + char.charCodeAt(0), 0);
  return colors[hash % colors.length];
}
```

### 敏感操作审计

```typescript
// lib/liveblocks/audit.ts
import { db } from '@/lib/db';

interface AuditLogEntry {
  timestamp: Date;
  userId: string;
  roomId: string;
  action: 'view' | 'edit' | 'delete' | 'comment' | 'presence_update';
  details: Record<string, any>;
}

export async function logLiveblocksAction(entry: AuditLogEntry) {
  await db.liveblocksAuditLog.create({
    data: {
      ...entry,
      timestamp: new Date(),
    },
  });
  
  // 生产环境发送到日志服务
  if (process.env.NODE_ENV === 'production') {
    await sendToLogService({
      service: 'liveblocks',
      ...entry,
    });
  }
}

// 监控异常行为
export async function detectSuspiciousActivity(userId: string, roomId: string) {
  const recentActions = await db.liveblocksAuditLog.findMany({
    where: {
      userId,
      roomId,
      timestamp: {
        gte: new Date(Date.now() - 60 * 1000), // 最近1分钟
      },
    },
    orderBy: { timestamp: 'desc' },
  });
  
  // 检测异常：每分钟超过100次操作
  if (recentActions.length > 100) {
    await sendSecurityAlert({
      type: 'high_activity',
      userId,
      roomId,
      count: recentActions.length,
    });
  }
  
  // 检测异常：短时间内大量删除
  const deletions = recentActions.filter(a => a.action === 'delete');
  if (deletions.length > 10) {
    await sendSecurityAlert({
      type: 'mass_deletion',
      userId,
      roomId,
      count: deletions.length,
    });
  }
}
```

---

## Liveblocks 与 Next.js 深度集成

### App Router 集成

```tsx
// app/room/[roomId]/page.tsx
import { Suspense } from 'react';
import { RoomProvider } from '@/liveblocks.config';
import { LiveMap } from '@liveblocks/client';
import { RoomCanvas } from '@/components/RoomCanvas';
import { RoomPresence } from '@/components/RoomPresence';

interface PageProps {
  params: { roomId: string };
}

export default async function RoomPage({ params }: PageProps) {
  const { roomId } = params;
  
  // 服务端获取房间信息
  const roomInfo = await getRoomInfo(roomId);
  
  return (
    <RoomProvider
      id={roomId}
      initialPresence={{
        cursor: null,
        selectedId: null,
        user: {
          id: roomInfo.currentUserId,
          name: roomInfo.currentUserName,
          color: roomInfo.currentUserColor,
        },
      }}
      initialStorage={{
        canvas: new LiveMap(),
        metadata: new LiveMap({
          title: roomInfo.title,
          createdAt: roomInfo.createdAt,
        }),
      }}
    >
      <div className="room-container">
        <RoomPresence />
        <Suspense fallback={<div>Loading canvas...</div>}>
          <RoomCanvas />
        </Suspense>
      </div>
    </RoomProvider>
  );
}
```

### Server Components 集成

```tsx
// components/RoomCanvas.tsx
'use client';

import { useStorage, useMutation } from '@/liveblocks.config';
import { LiveMap, LiveObject } from '@liveblocks/client';
import { useEffect, useState } from 'react';

export function RoomCanvas() {
  const canvas = useStorage((root) => root.canvas as LiveMap<string, LiveObject<any>>);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    // 当 Storage 加载完成时
    if (canvas) {
      setIsLoading(false);
    }
  }, [canvas]);
  
  if (isLoading) {
    return <div className="loading-skeleton" />;
  }
  
  return (
    <div className="canvas-wrapper">
      {/* 渲染画布元素 */}
    </div>
  );
}
```

### 中间件配置

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // 处理 Liveblocks WebSocket 升级请求
  if (request.nextUrl.pathname.startsWith('/api/liveblocks')) {
    // 设置 CORS 头
    const response = NextResponse.next();
    response.headers.set('Access-Control-Allow-Origin', '*');
    response.headers.set('Access-Control-Allow-Methods', 'POST, OPTIONS');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    return response;
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/api/liveblocks/:path*'],
};
```

---

## Liveblocks 企业级功能

### 多房间管理

```typescript
// lib/liveblocks/roomManager.ts
import { Liveblocks } from '@liveblocks/node';
import { db } from '@/lib/db';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

export interface RoomConfig {
  id: string;
  name: string;
  type: 'canvas' | 'document' | 'spreadsheet';
  visibility: 'private' | 'team' | 'public';
  maxUsers: number;
  metadata: Record<string, string>;
}

export async function createRoom(config: RoomConfig) {
  // 创建数据库记录
  const room = await db.room.create({
    data: {
      id: config.id,
      name: config.name,
      type: config.type,
      visibility: config.visibility,
      maxUsers: config.maxUsers,
    },
  });
  
  // 创建 Liveblocks 房间
  await liveblocks.createRoom(config.id, {
    defaultAccessLevels: {
      presence: config.visibility === 'public' ? ['public'] : [],
      storage: config.visibility === 'public' ? ['public'] : [],
    },
    metadata: config.metadata,
  });
  
  return room;
}

export async function archiveRoom(roomId: string) {
  // 归档房间（保留数据但不接受新用户）
  await db.room.update({
    where: { id: roomId },
    data: { status: 'archived' },
  });
  
  // 更新 Liveblocks 房间配置
  await liveblocks.updateRoom(roomId, {
    metadata: { archived: 'true', archivedAt: new Date().toISOString() },
  });
}

export async function getRoomAnalytics(roomId: string) {
  const room = await liveblocks.getRoom(roomId);
  
  const dbStats = await db.roomAccessLog.groupBy({
    by: ['action'],
    where: { roomId },
    _count: true,
  });
  
  return {
    id: roomId,
    status: room.metadata.status,
    activeUsers: room.activeUsers?.length || 0,
    totalEdits: dbStats.find(s => s.action === 'edit')?._count || 0,
    totalViews: dbStats.find(s => s.action === 'view')?._count || 0,
    createdAt: room.createdAt,
    lastActivity: room.lastInActivityAt,
  };
}
```

### 批量操作与性能优化

```typescript
// lib/liveblocks/batchOperations.ts
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

// 批量更新多个房间
export async function batchUpdateRooms(
  roomIds: string[],
  updates: Record<string, any>
) {
  const results = [];
  
  for (const roomId of roomIds) {
    try {
      const room = await liveblocks.updateRoom(roomId, {
        metadata: updates,
      });
      results.push({ roomId, success: true, room });
    } catch (error) {
      results.push({ roomId, success: false, error });
    }
  }
  
  return results;
}

// 获取多个房间状态
export async function getRoomsStatus(roomIds: string[]) {
  const rooms = await Promise.all(
    roomIds.map(roomId => liveblocks.getRoom(roomId))
  );
  
  return rooms.map(room => ({
    id: room.id,
    status: room.metadata.status,
    activeUsers: room.activeUsers?.length || 0,
  }));
}

// 迁移房间数据
export async function migrateRoomData(
  sourceRoomId: string,
  targetRoomId: string
) {
  // 获取源房间的 Storage
  const sourceRoom = await liveblocks.getRoom(sourceRoomId);
  const storage = await sourceRoom.getStorage();
  
  // 在目标房间中创建相同的 Storage
  const targetRoom = await liveblocks.getRoom(targetRoomId);
  
  // 复制 Storage 数据
  await targetRoom.updateStorage({
    document: storage.get('document'),
  });
  
  // 记录迁移历史
  await db.roomMigrationLog.create({
    data: {
      sourceRoomId,
      targetRoomId,
      migratedAt: new Date(),
    },
  });
}
```

---

## Liveblocks 与数据库集成

### Prisma 同步

```typescript
// lib/liveblocks/prismaSync.ts
import { db } from '@/lib/db';
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

// 从 Liveblocks 同步数据到 Prisma
export async function syncRoomFromLiveblocks(roomId: string) {
  const room = await liveblocks.getRoom(roomId);
  
  await db.room.update({
    where: { id: roomId },
    data: {
      name: room.metadata.name as string,
      status: room.metadata.status as string,
      lastActivityAt: room.lastInActivityAt 
        ? new Date(room.lastInActivityAt) 
        : undefined,
    },
  });
}

// 定期同步所有房间
export async function syncAllRooms() {
  // 获取所有活跃房间
  const rooms = await db.room.findMany({
    where: { status: 'active' },
  });
  
  for (const room of rooms) {
    try {
      await syncRoomFromLiveblocks(room.id);
    } catch (error) {
      console.error(`Failed to sync room ${room.id}:`, error);
    }
  }
}

// 导出房间数据
export async function exportRoomData(roomId: string) {
  const room = await liveblocks.getRoom(roomId);
  const storage = await room.getStorage();
  
  const dbRoom = await db.room.findUnique({
    where: { id: roomId },
    include: { members: true, accessLogs: true },
  });
  
  return {
    metadata: {
      id: room.id,
      name: room.metadata.name,
      createdAt: room.createdAt,
    },
    storage: storage.toImmutable(),
    members: dbRoom?.members,
    auditLog: dbRoom?.accessLogs,
  };
}
```

### 事件驱动的数据更新

```typescript
// lib/liveblocks/eventDriven.ts
import { db } from '@/lib/db';
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

// 处理 Liveblocks Webhook 事件
export async function handleLiveblocksWebhook(payload: any) {
  const { type, payload: data, timestamp } = payload;
  
  switch (type) {
    case 'ASSET_CREATED': {
      const { id, userId, roomId, type } = data;
      await db.asset.create({
        data: {
          id,
          roomId,
          createdById: userId,
          type,
          createdAt: new Date(timestamp),
        },
      });
      break;
    }
    
    case 'ASSET_DELETED': {
      const { id } = data;
      await db.asset.update({
        where: { id },
        data: { deletedAt: new Date(timestamp) },
      });
      break;
    }
    
    case 'ROOM_ARCHIVED': {
      const { id } = data;
      await db.room.update({
        where: { id },
        data: { status: 'archived' },
      });
      break;
    }
    
    case 'USER_JOINED': {
      const { userId, roomId } = data;
      await db.roomAccessLog.create({
        data: {
          roomId,
          userId,
          action: 'join',
          timestamp: new Date(timestamp),
        },
      });
      break;
    }
    
    case 'USER_LEFT': {
      const { userId, roomId } = data;
      await db.roomAccessLog.create({
        data: {
          roomId,
          userId,
          action: 'leave',
          timestamp: new Date(timestamp),
        },
      });
      break;
    }
  }
}
```

---

## Liveblocks 实时通知系统

### 通知服务架构

```typescript
// lib/liveblocks/notifications.ts
import { db } from '@/lib/db';
import { Liveblocks } from '@liveblocks/node';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

interface NotificationPayload {
  type: 'mention' | 'comment' | 'edit' | 'system';
  roomId: string;
  targetUserId: string;
  actorUserId: string;
  message: string;
  link?: string;
}

export async function sendNotification(payload: NotificationPayload) {
  // 1. 保存到数据库
  const notification = await db.notification.create({
    data: {
      type: payload.type,
      roomId: payload.roomId,
      targetUserId: payload.targetUserId,
      actorUserId: payload.actorUserId,
      message: payload.message,
      link: payload.link,
      read: false,
    },
  });
  
  // 2. 通过 Liveblocks Broadcast 实时推送
  const room = await liveblocks.getRoom(payload.roomId);
  await room.broadcastEvent({
    type: 'NOTIFICATION',
    payload: {
      id: notification.id,
      type: payload.type,
      message: payload.message,
      link: payload.link,
    },
  }, { targetIds: [payload.targetUserId] });
  
  // 3. 发送邮件（如果用户开启通知）
  const userSettings = await db.userNotificationSettings.findUnique({
    where: { userId: payload.targetUserId },
  });
  
  if (userSettings?.emailNotifications) {
    await sendEmailNotification(payload);
  }
  
  return notification;
}

// 提及通知
export async function sendMentionNotification(
  roomId: string,
  mentionedUserId: string,
  mentionerUserId: string,
  context: string
) {
  const mentionedUser = await db.user.findUnique({
    where: { id: mentionedUserId },
  });
  
  const mentioner = await db.user.findUnique({
    where: { id: mentionerUserId },
  });
  
  return sendNotification({
    type: 'mention',
    roomId,
    targetUserId: mentionedUserId,
    actorUserId: mentionerUserId,
    message: `${mentioner?.name} mentioned you: "${context.slice(0, 50)}..."`,
    link: `/room/${roomId}`,
  });
}

// 评论通知
export async function sendCommentNotification(
  roomId: string,
  commentOwnerId: string,
  commenterId: string,
  commentPreview: string
) {
  const commenter = await db.user.findUnique({
    where: { id: commenterId },
  });
  
  return sendNotification({
    type: 'comment',
    roomId,
    targetUserId: commentOwnerId,
    actorUserId: commenterId,
    message: `${commenter?.name} commented: "${commentPreview.slice(0, 50)}..."`,
    link: `/room/${roomId}`,
  });
}
```

### 实时通知组件

```tsx
// components/Notifications.tsx
'use client';

import { useBroadcastEvent, useBroadcastEventListener } from '@/liveblocks.config';
import { useEffect, useState } from 'react';
import Link from 'next/link';

interface Notification {
  id: string;
  type: string;
  message: string;
  link?: string;
  read: boolean;
  createdAt: Date;
}

export function NotificationBell() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [isOpen, setIsOpen] = useState(false);
  const broadcast = useBroadcastEvent();
  
  // 监听实时通知
  useBroadcastEventListener((event) => {
    if (event.type === 'NOTIFICATION') {
      setNotifications(prev => [
        {
          id: event.payload.id,
          type: event.payload.type,
          message: event.payload.message,
          link: event.payload.link,
          read: false,
          createdAt: new Date(),
        },
        ...prev,
      ]);
    }
  });
  
  const unreadCount = notifications.filter(n => !n.read).length;
  
  const markAsRead = async (id: string) => {
    await fetch(`/api/notifications/${id}/read`, { method: 'POST' });
    setNotifications(prev =>
      prev.map(n => n.id === id ? { ...n, read: true } : n)
    );
  };
  
  return (
    <div className="notification-bell">
      <button onClick={() => setIsOpen(!isOpen)}>
        🔔 {unreadCount > 0 && <span className="badge">{unreadCount}</span>}
      </button>
      
      {isOpen && (
        <div className="notification-dropdown">
          <h3>Notifications</h3>
          {notifications.length === 0 ? (
            <p>No notifications</p>
          ) : (
            <ul>
              {notifications.map(notification => (
                <li key={notification.id} className={notification.read ? '' : 'unread'}>
                  <Link href={notification.link || '#'}>
                    {notification.message}
                  </Link>
                </li>
              ))}
            </ul>
          )}
        </div>
      )}
    </div>
  );
}
```

---

## Liveblocks 性能监控

### 性能指标收集

```typescript
// lib/liveblocks/performance.ts
import { db } from '@/lib/db';

interface PerformanceMetrics {
  roomId: string;
  userId: string;
  metric: 'connection_time' | 'sync_time' | 'render_time' | 'operation_count';
  value: number;
  timestamp: Date;
}

export async function recordPerformanceMetric(metric: PerformanceMetrics) {
  await db.liveblocksPerformance.create({
    data: metric,
  });
}

// 客户端性能监控
export function usePerformanceMonitor(roomId: string, userId: string) {
  useEffect(() => {
    const startTime = performance.now();
    
    // 监控连接时间
    const connectionObserver = () => {
      const connectionTime = performance.now() - startTime;
      recordPerformanceMetric({
        roomId,
        userId,
        metric: 'connection_time',
        value: connectionTime,
        timestamp: new Date(),
      });
    };
    
    // 监控渲染时间
    let renderCount = 0;
    const renderObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'measure') {
          recordPerformanceMetric({
            roomId,
            userId,
            metric: 'render_time',
            value: entry.duration,
            timestamp: new Date(),
          });
        }
      }
    });
    
    renderObserver.observe({ entryTypes: ['measure'] });
    
    return () => {
      renderObserver.disconnect();
    };
  }, [roomId, userId]);
}

// 分析性能数据
export async function analyzeRoomPerformance(roomId: string) {
  const metrics = await db.liveblocksPerformance.findMany({
    where: { roomId },
    orderBy: { timestamp: 'desc' },
    take: 1000,
  });
  
  const byMetric = {
    connection_time: metrics.filter(m => m.metric === 'connection_time'),
    sync_time: metrics.filter(m => m.metric === 'sync_time'),
    render_time: metrics.filter(m => m.metric === 'render_time'),
    operation_count: metrics.filter(m => m.metric === 'operation_count'),
  };
  
  const calculateStats = (values: number[]) => ({
    avg: values.reduce((a, b) => a + b, 0) / values.length,
    min: Math.min(...values),
    max: Math.max(...values),
    p95: values.sort((a, b) => a - b)[Math.floor(values.length * 0.95)],
  });
  
  return {
    connection: calculateStats(byMetric.connection_time.map(m => m.value)),
    sync: calculateStats(byMetric.sync_time.map(m => m.value)),
    render: calculateStats(byMetric.render_time.map(m => m.value)),
    operations: byMetric.operation_count.length,
  };
}
```

### 实时监控面板

```tsx
// components/PerformanceDashboard.tsx
'use client';

import { useOthers, useRoom } from '@/liveblocks.config';
import { useState, useEffect } from 'react';

export function PerformanceDashboard({ roomId }: { roomId: string }) {
  const room = useRoom();
  const others = useOthers();
  const [metrics, setMetrics] = useState<any>(null);
  
  useEffect(() => {
    // 获取性能数据
    fetch(`/api/rooms/${roomId}/performance`)
      .then(res => res.json())
      .then(setMetrics);
    
    // 定期刷新
    const interval = setInterval(() => {
      fetch(`/api/rooms/${roomId}/performance`)
        .then(res => res.json())
        .then(setMetrics);
    }, 30000);
    
    return () => clearInterval(interval);
  }, [roomId]);
  
  return (
    <div className="performance-dashboard">
      <h3>Room Performance</h3>
      
      <div className="metrics-grid">
        <div className="metric-card">
          <h4>Active Users</h4>
          <p className="metric-value">{others.length + 1}</p>
        </div>
        
        <div className="metric-card">
          <h4>Connection</h4>
          <p className="metric-value">
            {metrics?.connection?.avg?.toFixed(0) || '...'}ms
          </p>
          <p className="metric-label">avg</p>
        </div>
        
        <div className="metric-card">
          <h4>Sync</h4>
          <p className="metric-value">
            {metrics?.sync?.avg?.toFixed(0) || '...'}ms
          </p>
          <p className="metric-label">avg</p>
        </div>
        
        <div className="metric-card">
          <h4>Render</h4>
          <p className="metric-value">
            {metrics?.render?.avg?.toFixed(0) || '...'}ms
          </p>
          <p className="metric-label">avg</p>
        </div>
      </div>
      
      <div className="connection-status">
        <span className={`status ${room.status}`}>
          {room.status}
        </span>
      </div>
    </div>
  );
}
```

---

## Liveblocks 迁移指南

### 从 Yjs 迁移

```typescript
// scripts/migrate-from-yjs.ts
import { Liveblocks } from '@liveblocks/node';
import * as Y from 'yjs';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

interface YjsDocument {
  id: string;
  ydoc: Y.Doc;
}

export async function migrateYjsDocument(
  sourceDoc: YjsDocument
): Promise<string> {
  const { id, ydoc } = sourceDoc;
  
  // 创建新的 Liveblocks 房间
  const roomId = `migrated-${id}-${Date.now()}`;
  await liveblocks.createRoom(roomId, {
    defaultAccessLevels: {
      presence: [],
      storage: [],
    },
  });
  
  const room = await liveblocks.getRoom(roomId);
  const storage = await room.getStorage();
  
  // 转换 Yjs 数据到 Liveblocks 格式
  const root = storage.root;
  
  ydoc.getMap('data').forEach((value, key) => {
    root.set(key, value);
  });
  
  // 转换数组
  const arrayData = ydoc.getArray('items');
  const liveList = new LiveList(
    arrayData.toArray().map(item => new LiveObject(item))
  );
  root.set('items', liveList);
  
  // 保存 Storage
  await room.saveStorage(storage);
  
  return roomId;
}

// 批量迁移
export async function migrateAllDocuments(documents: YjsDocument[]) {
  const results = [];
  
  for (const doc of documents) {
    try {
      const newRoomId = await migrateYjsDocument(doc);
      results.push({ id: doc.id, newRoomId, success: true });
    } catch (error) {
      results.push({ id: doc.id, error, success: false });
    }
  }
  
  return results;
}
```

### 从 Firebase Realtime Database 迁移

```typescript
// scripts/migrate-from-firebase.ts
import { Liveblocks } from '@liveblocks/node';
import { ref, get } from 'firebase/database';

const liveblocks = new Liveblocks({
  secret: process.env.LIVEBLOCKS_SECRET_KEY!,
});

interface FirebaseDocument {
  id: string;
  data: Record<string, any>;
  lastModified: number;
}

export async function migrateFromFirebase(
  firebaseRef: any,
  documentId: string
): Promise<string> {
  // 从 Firebase 获取数据
  const snapshot = await get(ref(firebaseRef, `documents/${documentId}`));
  const firebaseData = snapshot.val();
  
  if (!firebaseData) {
    throw new Error('Document not found in Firebase');
  }
  
  // 创建 Liveblocks 房间
  const roomId = `migrated-${documentId}-${Date.now()}`;
  await liveblocks.createRoom(roomId, {
    defaultAccessLevels: {
      presence: [],
      storage: [],
    },
    metadata: {
      migratedFrom: 'firebase',
      originalId: documentId,
      migratedAt: new Date().toISOString(),
    },
  });
  
  const room = await liveblocks.getRoom(roomId);
  const storage = await room.getStorage();
  
  // 转换 Firebase 数据到 Liveblocks 格式
  const root = storage.root;
  
  // 处理嵌套对象
  Object.entries(firebaseData.data || firebaseData).forEach(([key, value]) => {
    if (typeof value === 'object' && value !== null) {
      root.set(key, new LiveObject(value));
    } else {
      root.set(key, value);
    }
  });
  
  await room.saveStorage(storage);
  
  return roomId;
}
```

---

## Liveblocks 最佳实践总结

### 开发环境配置

```bash
# .env.local.development
NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY=pk_dev_xxxxx
LIVEBLOCKS_SECRET_KEY=sk_dev_xxxxx

# 开发环境优化
LIVEBLOCKS_THROTTLE=50
LIVEBLOCKS_LOG_LEVEL=debug
```

### 生产环境配置

```bash
# .env.production
NEXT_PUBLIC_LIVEBLOCKS_PUBLIC_KEY=pk_prod_xxxxx
LIVEBLOCKS_SECRET_KEY=sk_prod_xxxxx

# 生产环境优化
LIVEBLOCKS_THROTTLE=100
LIVEBLOCKS_ENABLE_COMPRESSION=true
```

### 安全检查清单

```markdown
## Liveblocks 安全检查清单

- [ ] 实现房间级权限控制
- [ ] 配置 Liveblocks Auth 认证
- [ ] 启用 Webhook 签名验证
- [ ] 记录审计日志
- [ ] 监控异常行为
- [ ] 配置 CORS 策略
- [ ] 定期备份房间数据
- [ ] 实现速率限制
- [ ] 敏感操作二次验证
- [ ] 监控实时性能指标
```

### 性能优化清单

```markdown
## Liveblocks 性能优化清单

- [ ] 使用 Suspense 优化加载
- [ ] 选择性订阅 Storage
- [ ] 使用 shallow 避免不必要更新
- [ ] 批量更新操作
- [ ] 配置合适的 throttle
- [ ] 启用 Storage 压缩
- [ ] 实现增量同步
- [ ] 缓存热点数据
- [ ] 监控并优化慢查询
- [ ] 定期清理无用房间
```

---

## Liveblocks 与竞品深度对比

### 与 Convex 对比

| 维度 | Liveblocks | Convex |
|------|------------|--------|
| **核心定位** | 实时协作 UI | 实时后端 |
| **数据结构** | CRDT (LiveObject) | TypeScript 对象 |
| **离线支持** | ✅ 完善 | ⚠️ 基础 |
| **Presence** | ✅ 原生 | ✅ 原生 |
| **Storage** | ✅ CRDT 持久化 | ✅ 实时数据库 |
| **评论系统** | ✅ 原生 | ❌ 需自建 |
| **AI 集成** | ✅ Liveblocks AI | ⚠️ 需自建 |
| **定价模型** | 按座位 | 按量 |
| **学习曲线** | 低 | 中 |
| **调试工具** | 完善 | 一般 |

### 与自建 WebSocket 对比

| 维度 | Liveblocks | 自建 WebSocket |
|------|------------|---------------|
| **开发时间** | 1-2 天 | 2-4 周 |
| **基础设施** | 托管 | 自建 |
| **扩展性** | 自动 | 需手动 |
| **冲突解决** | CRDT 自动 | 需自实现 |
| **离线支持** | 开箱即用 | 需自实现 |
| **成本** | 按座位 | 服务器成本 |
| **维护** | 托管 | 全栈维护 |
| **定制性** | 中等 | 完全控制 |

### 选择建议

**选择 Liveblocks 如果：**
- 需要快速实现协作功能
- 需要现成的评论系统
- 需要 AI 辅助功能
- 预算充足，按座位计费可接受
- 需要托管基础设施

**选择 Convex 如果：**
- 需要实时后端能力
- 已经有后端逻辑
- 需要 TypeScript 原生集成
- 愿意接受定价按量计费
- 需要更多后端控制

**选择自建 WebSocket 如果：**
- 需要完全控制
- 有特殊安全要求
- 有 WebSocket 经验
- 预算极其有限
- 需要高度定制

---

> [!TIP]
> Liveblocks 的实时协作能力非常强大，合理利用 Presence、Storage 和 Events，可以构建出类似 Figma、Google Docs 的实时协作体验。
