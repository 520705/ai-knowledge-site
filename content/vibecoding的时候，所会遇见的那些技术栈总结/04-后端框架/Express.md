# Express 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Express 5 alpha 新特性、中间件系统、路由进阶、错误处理及在 AI 应用中的角色。

---

## 目录

1. [[#Express 概述与市场定位]]
2. [[#Express 5 新特性]]
3. [[#中间件系统详解]]
4. [[#路由进阶]]
5. [[#错误处理]]
6. [[#MVC 架构实践]]
7. [[#Express vs Fastify vs Koa 对比]]
8. [[#AI 应用集成]]
9. [[#实战场景与选型建议]]

---

## Express 概述与市场定位

### Express 在 Node.js 生态中的地位

Express 是 Node.js 生态中最经典、最广泛使用的 Web 框架，自 2010 年发布以来，已成为 Web 服务的「Hello World」。其设计哲学——「最小化、灵活、可扩展」——影响了后续几乎所有 Node.js 框架。

### Express 的核心优势

| 优势 | 说明 |
|------|------|
| **简单易学** | 极低的入门门槛，文档完善 |
| **生态系统丰富** | 20,000+ npm 包 |
| **中间件模式** | 模块化、可组合 |
| **灵活性** | 无opinionated限制 |
| **社区庞大** | 大量教程和解决方案 |
| **企业认可** | 长期生产验证 |

### Express 的局限性

| 局限 | 说明 |
|------|------|
| **性能一般** | 对比 Fastify/Elysia 较慢 |
| **非类型安全** | TypeScript 支持需额外配置 |
| **异步错误处理** | 需手动 try/catch |
| **无内置验证** | 需配合 ajv/zod |
| **更新缓慢** | Express 5 长期 alpha |

---

## Express 5 新特性

### Express 5 alpha 概览

Express 5 正在进行 alpha 测试，带来以下改进：

| 特性 | 说明 | 状态 |
|------|------|------|
| **Promise 支持** | 路由处理器支持 async/await | ✅ |
| **Router 改进** | `router.param()` 语义变更 | ✅ |
| **移除 API** | `res.json(obj, status)` 移除 | ❌ |
| **性能优化** | 改进中间件执行 | ✅ |
| **更好的错误栈** | 更清晰的错误追踪 | ✅ |

### 主要变更

```javascript
// Express 4 - 必须使用回调
app.get('/users/:id', (req, res, next) => {
  User.findById(req.params.id)
    .then(user => {
      if (!user) return next(new Error('Not found'));
      res.json(user);
    })
    .catch(next);
});

// Express 5 - 原生支持 async/await
app.get('/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) throw new Error('Not found');
    res.json(user);
  } catch (err) {
    next(err);  // 自动传播
  }
});
```

```javascript
// Express 4 - router.param 处理
router.param('userId', (req, res, next, id) => {
  req.user = { id };
  next();
});

// Express 5 - 新语义
router.param('userId', (req, res, next) => {
  req.user = { id: req.params.userId };
  next();
});
```

> [!IMPORTANT]
> Express 5 仍处于 alpha 阶段，生产环境建议使用 Express 4。升级前请测试所有路由和中间件。

---

## 中间件系统详解

### 中间件执行流程

```
请求 → [中间件1] → [中间件2] → [路由处理器] → 响应
              ↓           ↓           ↓
            next()      next()    res.send()
```

### 中间件类型

| 类型 | 说明 | 示例 |
|------|------|------|
| **应用级** | 绑定到 app 实例 | `app.use()` |
| **路由级** | 绑定到 router | `router.use()` |
| **错误处理** | 4 个参数 | `app.use((err, req, res, next)` |
| **内置** | Express 提供 | `express.static()`, `express.json()` |
| **第三方** | npm 安装 | `cors`, `helmet`, `morgan` |

### 自定义中间件

```javascript
// middleware/logger.js
// 请求日志中间件
function logger(req, res, next) {
  const start = Date.now();
  
  // 响应完成时记录
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log({
      method: req.method,
      url: req.url,
      status: res.statusCode,
      duration: `${duration}ms`,
      timestamp: new Date().toISOString(),
    });
  });
  
  next();
}

// 带配置的工厂函数
function createLogger(options = {}) {
  const { format = 'short', includeBody = false } = options;
  
  return (req, res, next) => {
    const log = {
      method: req.method,
      url: req.url,
      timestamp: new Date().toISOString(),
    };
    
    if (includeBody) {
      log.body = req.body;
    }
    
    console.log(format === 'short' 
      ? `${log.method} ${log.url}` 
      : JSON.stringify(log)
    );
    
    next();
  };
}

module.exports = { logger, createLogger };
```

### 常用中间件

#### body-parser / 内置 express.json

```javascript
const express = require('express');
const app = express();

// 内置 JSON 解析（Express 4.16+）
app.use(express.json({ limit: '10mb' }));

// URL 编码
app.use(express.urlencoded({ extended: true }));

// 原始请求体
app.use(express.raw({ type: 'application/octet-stream' }));

// 文本
app.use(express.text({ type: 'text/html' }));
```

#### cors - 跨域资源共享

```javascript
const cors = require('cors');

// 允许所有来源（开发环境）
app.use(cors());

// 配置更精细的 CORS
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com'],
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  exposedHeaders: ['X-Total-Count'],
  credentials: true,
  maxAge: 86400,  // 预检请求缓存 24 小时
}));

// 路由级 CORS
app.get('/api/public', cors(), (req, res) => {
  res.json({ message: 'Public endpoint' });
});

app.get('/api/private', cors({
  origin: 'https://app.example.com'
}), (req, res) => {
  res.json({ message: 'Private endpoint' });
});
```

#### helmet - 安全头

```javascript
const helmet = require('helmet');

// 使用所有安全头
app.use(helmet());

// 或选择性使用
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    scriptSrc: ["'self'"],
  },
}));

app.use(helmet.hsts({
  maxAge: 31536000,
  includeSubDomains: true,
}));

app.use(helmet.noSniff());
app.use(helmet.frameguard({ action: 'deny' }));
```

#### morgan - HTTP 日志

```javascript
const morgan = require('morgan');

// 预定义格式
app.use(morgan('combined'));    // Apache 格式
app.use(morgan('common'));     // Apache 简短格式
app.use(morgan('dev'));         // 彩色开发格式
app.use(morgan('short'));       // 简短格式
app.use(morgan('tiny'));        // 最小格式

// 自定义格式
morgan.format('custom', (req, res) => {
  const status = res.statusCode;
  const color = status >= 500 ? 31 : status >= 400 ? 33 : status >= 300 ? 36 : 32;
  return `\x1b[${color}m${req.method}\x1b[0m ${req.originalUrl} ${res.statusCode} ${res.responseTime}ms`;
});

app.use(morgan('custom'));

// 流配置（写入文件）
const fs = require('fs');
const path = require('path');

const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'logs', 'access.log'),
  { flags: 'a' }
);

app.use(morgan('combined', { stream: accessLogStream }));
```

### 中间件执行顺序

```javascript
// 1. 全局中间件
app.use((req, res, next) => {
  console.log('1. 全局 - 始终执行');
  next();
});

// 2. 路径匹配中间件
app.use('/api', (req, res, next) => {
  console.log('2. /api 路径 - 只对 /api/* 生效');
  next();
});

// 3. 路由处理器前
app.use((req, res, next) => {
  console.log('3. 路由前 - 接近终点');
  next();
});

// 4. 路由处理器
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});
```

---

## 路由进阶

### 路由参数与验证

```javascript
const express = require('express');
const { z } = require('zod');
const Joi = require('joi');

const router = express.Router();

// Joi 验证
const userSchema = Joi.object({
  name: Joi.string().min(1).max(100).required(),
  email: Joi.string().email().required(),
  age: Joi.number().integer().min(0).max(150).optional(),
});

// 路由级中间件验证
router.post('/users', async (req, res, next) => {
  try {
    const { error, value } = userSchema.validate(req.body);
    if (error) {
      return res.status(400).json({ 
        error: 'Validation failed', 
        details: error.details 
      });
    }
    req.validatedBody = value;
    next();
  } catch (err) {
    next(err);
  }
}, async (req, res) => {
  // 使用验证后的数据
  const user = await User.create(req.validatedBody);
  res.status(201).json(user);
});

// Zod 验证（推荐）
const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8),
});

router.post('/users', async (req, res, next) => {
  const result = createUserSchema.safeParse(req.body);
  
  if (!result.success) {
    return res.status(400).json({
      error: 'Validation failed',
      issues: result.error.issues,
    });
  }
  
  // result.data 已验证
  const user = await User.create(result.data);
  res.status(201).json(user);
});
```

### 路由分组与嵌套

```javascript
const express = require('express');
const router = express.Router();

// 用户路由组
const userRouter = require('./routes/users');
const postRouter = require('./routes/posts');
const commentRouter = require('./routes/comments');

// 前缀路由
app.use('/api/users', userRouter);
app.use('/api/posts', postRouter);
app.use('/api/comments', commentRouter);

// 嵌套路由
// comments.js
const express = require('express');
const router = express.Router({ mergeParams: true });  // 合并父路由参数

router.get('/', (req, res) => {
  // 可以访问 /posts/:postId
  res.json({ postId: req.params.postId });
});

module.exports = router;
```

### RESTful 路由设计

```javascript
// 标准 RESTful
app.get('/users', listUsers);           // GET    /users      - 列表
app.post('/users', createUser);         // POST   /users      - 创建
app.get('/users/:id', getUser);         // GET    /users/:id  - 详情
app.put('/users/:id', updateUser);      // PUT    /users/:id  - 全量更新
app.patch('/users/:id', patchUser);     // PATCH  /users/:id  - 部分更新
app.delete('/users/:id', deleteUser);   // DELETE /users/:id  - 删除

// 资源嵌套（谨慎使用）
app.get('/users/:userId/posts', getUserPosts);      // GET  /users/:userId/posts
app.post('/users/:userId/posts', createUserPost);    // POST /users/:userId/posts

// 操作型路由
app.post('/users/:id/activate', activateUser);        // POST /users/:id/activate
app.post('/users/:id/deactivate', deactivateUser);   // POST /users/:id/deactivate
```

---

## 错误处理

### 同步/异步错误处理

```javascript
// 同步错误 - 自动捕获
app.get('/error-sync', (req, res) => {
  throw new Error('同步错误');
});

// 异步错误 - 需传递给 next
app.get('/error-async', async (req, res, next) => {
  try {
    await someAsyncOperation();
  } catch (err) {
    next(err);  // 必须传递
  }
});

// Express 5 - async 错误自动传播
app.get('/error-async-5', async (req, res, next) => {
  await someAsyncOperation();  // 错误自动传递给 next
});
```

### 错误处理器中间件

```javascript
// 错误处理器必须是 4 个参数
function errorHandler(err, req, res, next) {
  // 日志记录
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    timestamp: new Date().toISOString(),
    ip: req.ip,
  });
  
  // 区分开发/生产环境
  if (process.env.NODE_ENV === 'production') {
    res.status(err.status || 500).json({
      error: {
        message: err.message || 'Internal server error',
        code: err.code || 'INTERNAL_ERROR',
      },
    });
  } else {
    res.status(err.status || 500).json({
      error: {
        message: err.message,
        stack: err.stack,
        code: err.code,
      },
    });
  }
}

// 404 处理
function notFoundHandler(req, res, next) {
  res.status(404).json({
    error: {
      message: 'Resource not found',
      code: 'NOT_FOUND',
      path: req.path,
    },
  });
}

// 注册错误处理器（最后注册）
app.use(notFoundHandler);
app.use(errorHandler);
```

### 自定义错误类

```javascript
// errors/AppError.js
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;  // 区分可处理错误和编程错误
    
    Error.captureStackTrace(this, this.constructor);
  }
}

class ValidationError extends AppError {
  constructor(message, details = []) {
    super(message, 400, 'VALIDATION_ERROR');
    this.details = details;
  }
}

class NotFoundError extends AppError {
  constructor(resource = 'Resource') {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

class UnauthorizedError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

class ForbiddenError extends AppError {
  constructor(message = 'Forbidden') {
    super(message, 403, 'FORBIDDEN');
  }
}

module.exports = {
  AppError,
  ValidationError,
  NotFoundError,
  UnauthorizedError,
  ForbiddenError,
};
```

```javascript
// 使用示例
const { NotFoundError, ValidationError } = require('./errors/AppError');

app.get('/users/:id', async (req, res, next) => {
  const user = await User.findById(req.params.id);
  
  if (!user) {
    throw new NotFoundError('User');
  }
  
  if (!user.isActive) {
    throw new ForbiddenError('User account is inactive');
  }
  
  res.json(user);
});
```

---

## MVC 架构实践

### 项目结构

```
src/
├── app.js              # Express 应用入口
├── server.js           # 服务器启动
├── config/
│   └── index.js        # 配置
├── controllers/        # 控制器
│   ├── userController.js
│   └── postController.js
├── models/             # 数据模型
│   ├── User.js
│   └── Post.js
├── routes/             # 路由定义
│   ├── index.js
│   ├── userRoutes.js
│   └── postRoutes.js
├── middleware/         # 中间件
│   ├── auth.js
│   ├── validate.js
│   └── errorHandler.js
├── services/           # 业务逻辑
│   ├── userService.js
│   └── emailService.js
├── utils/              # 工具函数
│   └── logger.js
└── errors/             # 自定义错误
    └── AppError.js
```

### 控制器模式

```javascript
// controllers/userController.js
const User = require('../models/User');
const { NotFoundError, ValidationError } = require('../errors/AppError');

// 高阶函数包装器，处理错误传播
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

const userController = {
  // GET /users
  list: asyncHandler(async (req, res) => {
    const { page = 1, limit = 20, sort = '-createdAt' } = req.query;
    
    const [users, total] = await Promise.all([
      User.find()
        .sort(sort)
        .skip((page - 1) * limit)
        .limit(Number(limit)),
      User.countDocuments(),
    ]);
    
    res.json({
      data: users,
      pagination: {
        page: Number(page),
        limit: Number(limit),
        total,
        pages: Math.ceil(total / limit),
      },
    });
  }),
  
  // GET /users/:id
  show: asyncHandler(async (req, res) => {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw new NotFoundError('User');
    }
    
    res.json({ data: user });
  }),
  
  // POST /users
  create: asyncHandler(async (req, res) => {
    const user = await User.create(req.body);
    res.status(201).json({ data: user });
  }),
  
  // PUT /users/:id
  update: asyncHandler(async (req, res) => {
    const user = await User.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );
    
    if (!user) {
      throw new NotFoundError('User');
    }
    
    res.json({ data: user });
  }),
  
  // DELETE /users/:id
  destroy: asyncHandler(async (req, res) => {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      throw new NotFoundError('User');
    }
    
    res.status(204).send();
  }),
};

module.exports = userController;
```

### 模型层

```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true,
  },
  password: {
    type: String,
    required: true,
    minlength: 8,
    select: false,  // 默认不返回
  },
  name: {
    type: String,
    required: true,
    trim: true,
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user',
  },
  avatar: String,
  isActive: {
    type: Boolean,
    default: true,
  },
}, {
  timestamps: true,
});

// 密码哈希中间件
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  
  this.password = await bcrypt.hash(this.password, 12);
  next();
});

// 密码比对方法
userSchema.methods.comparePassword = async function(candidatePassword) {
  return bcrypt.compare(candidatePassword, this.password);
};

// JSON 转换时移除敏感字段
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password;
  return obj;
};

const User = mongoose.model('User', userSchema);

module.exports = User;
```

---

## Express vs Fastify vs Koa 对比

### 核心架构对比

| 维度 | Express | Fastify | Koa |
|------|---------|--------|-----|
| **版本** | 4.x/5.alpha | 4.x | 2.x |
| **性能** | 一般 | 极佳（2x Express） | 良好 |
| **中间件模型** | callback | Promise/async | async/await |
| **插件生态** | 丰富 | 增长中 | 较少 |
| **Schema 验证** | 需手动集成 | 内置 Fastify-Schema | 需手动集成 |
| **日志** | 需集成 morgan | 内置 | 需手动 |
| **类型安全** | 需配置 | 原生 TypeScript | 需配置 |
| **学习曲线** | 平缓 | 平缓 | 平缓 |

### 中间件模型对比

```javascript
// Express - 回调 + next()
app.use((req, res, next) => {
  // 同步代码
  next();
});

// Fastify - Promise
fastify.addHook('onRequest', async (request, reply) => {
  // 同步或异步
});

// Koa - async/await（无 next）
app.use(async (ctx, next) => {
  await next();
  ctx.body = 'response';
});
```

### 性能对比（基准测试）

| 框架 | 请求/秒 | 延迟 | 内存 |
|------|---------|------|------|
| **Express** | ~15,000 | ~65ms | ~75MB |
| **Fastify** | ~30,000+ | ~30ms | ~55MB |
| **Koa** | ~20,000 | ~50ms | ~60MB |
| **Elysia** | ~50,000+ | ~15ms | ~40MB |

> [!TIP]
> 性能差异在大多数应用中不明显，但高并发场景下 Fastify 优势显著。

### 生态对比

| 方面 | Express | Fastify | Koa |
|------|---------|--------|-----|
| **中间件数量** | 20,000+ | 500+ | 数百 |
| **企业采用** | 大量 | 增长中 | 中等 |
| **社区活跃度** | 高（稳定） | 高（活跃） | 中等 |
| **维护状态** | 维护（更新慢） | 活跃 | 维护 |

---

## AI 应用集成

### 构建 AI API 代理

```javascript
// routes/ai.js
const express = require('express');
const router = express.Router();
const { body, validationResult } = require('express-validator');
const { z } = require('zod');
const OpenAI = require('openai');
const rateLimit = require('express-rate-limit');

// 速率限制
const aiLimiter = rateLimit({
  windowMs: 60 * 1000,  // 1 分钟
  max: 20,              // 每分钟 20 次
  message: { error: '请求过于频繁，请稍后再试' },
});

const chatSchema = z.object({
  messages: z.array(z.object({
    role: z.enum(['system', 'user', 'assistant']),
    content: z.string(),
  })),
  model: z.enum(['gpt-4o', 'gpt-4o-mini']).optional(),
  temperature: z.number().min(0).max(2).optional(),
  max_tokens: z.number().min(1).max(4096).optional(),
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

router.post('/chat', aiLimiter, async (req, res, next) => {
  try {
    const validation = chatSchema.safeParse(req.body);
    
    if (!validation.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: validation.error.issues,
      });
    }
    
    const { messages, model = 'gpt-4o', temperature, max_tokens } = validation.data;
    
    const completion = await openai.chat.completions.create({
      model,
      messages,
      temperature,
      max_tokens,
    });
    
    res.json({
      id: completion.id,
      model: completion.model,
      choices: completion.choices,
      usage: completion.usage,
    });
  } catch (err) {
    if (err.status === 401) {
      return res.status(401).json({ error: 'Invalid API key' });
    }
    if (err.status === 429) {
      return res.status(429).json({ error: 'Rate limit exceeded' });
    }
    next(err);
  }
});

// 流式响应
router.post('/chat/stream', aiLimiter, async (req, res, next) => {
  try {
    const { messages, model = 'gpt-4o' } = req.body;
    
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');
    
    const stream = await openai.chat.completions.create({
      model,
      messages,
      stream: true,
    });
    
    for await (const chunk of stream) {
      const content = chunk.choices[0]?.delta?.content;
      if (content) {
        res.write(`data: ${JSON.stringify({ content })}\n\n`);
      }
    }
    
    res.write('data: [DONE]\n\n');
    res.end();
  } catch (err) {
    next(err);
  }
});

module.exports = router;
```

### WebSocket 实时通信（AI 对话）

```javascript
// 使用 express-ws 或独立 WebSocket
const WebSocket = require('ws');
const wss = new WebSocket.Server({ server: app });

wss.on('connection', (ws, req) => {
  const sessionId = generateSessionId();
  
  ws.on('message', async (message) => {
    const data = JSON.parse(message);
    
    if (data.type === 'chat') {
      const completion = await openai.chat.completions.create({
        model: 'gpt-4o',
        messages: [{ role: 'user', content: data.content }],
        stream: true,
      });
      
      for await (const chunk of completion) {
        const content = chunk.choices[0]?.delta?.content;
        if (content) {
          ws.send(JSON.stringify({ type: 'chunk', content }));
        }
      }
      
      ws.send(JSON.stringify({ type: 'done' }));
    }
  });
});
```

---

## 实战场景与选型建议

### Express + Socket.io 实时应用

```javascript
const { createServer } = require('http');
const express = require('express');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: { origin: '*' },
});

app.use(express.json());

// REST API
app.get('/api/status', (req, res) => {
  res.json({ status: 'ok', connections: io.engine.clientsCount });
});

// Socket.io 事件
io.on('connection', (socket) => {
  console.log('Client connected:', socket.id);
  
  socket.on('join-room', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('user-joined', socket.id);
  });
  
  socket.on('message', ({ roomId, content }) => {
    io.to(roomId).emit('message', {
      id: Date.now(),
      content,
      sender: socket.id,
    });
  });
  
  socket.on('disconnect', () => {
    console.log('Client disconnected:', socket.id);
  });
});

httpServer.listen(3000);
```

### 成本估算

| 方案 | 月成本（100万请求） | 说明 |
|------|-------------------|------|
| **AWS EC2 t3.medium** | ~$30 | 2vCPU, 4GB |
| **Railway** | $5-50 | 按使用计费 |
| **Render** | $7-50 | 自动扩缩容 |
| **Vercel Serverless** | $20+ | 函数计算 |

---

---

## 完整安装与环境配置

### Node.js 环境配置

```bash
# 检查 Node.js 和 npm 版本
node --version  # v20.x LTS 或更高
npm --version  # 10.x

# 推荐使用 nvm 管理 Node.js 版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20
nvm alias default 20

# 创建项目目录
mkdir my-express-api && cd my-express-api
npm init -y

# 安装 Express 和核心依赖
npm install express
npm install --save-dev typescript @types/express @types/node ts-node nodemon

# 初始化 TypeScript 配置
npx tsc --init
```

### TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 项目结构推荐

```
my-express-api/
├── src/
│   ├── app.ts              # 应用入口
│   ├── server.ts          # 服务器启动
│   ├── config/
│   │   ├── index.ts      # 配置加载
│   │   ├── database.ts   # 数据库配置
│   │   └── env.ts       # 环境变量
│   ├── controllers/
│   │   ├── userController.ts
│   │   └── postController.ts
│   ├── models/
│   │   ├── User.ts
│   │   └── Post.ts
│   ├── routes/
│   │   ├── index.ts
│   │   ├── userRoutes.ts
│   │   └── postRoutes.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── validate.ts
│   │   ├── rateLimiter.ts
│   │   └── errorHandler.ts
│   ├── services/
│   │   ├── userService.ts
│   │   └── emailService.ts
│   ├── utils/
│   │   ├── logger.ts
│   │   └── helper.ts
│   ├── errors/
│   │   ├── AppError.ts
│   │   └── errorCodes.ts
│   ├── types/
│   │   └── index.ts
│   └── validators/
│       ├── userValidator.ts
│       └── postValidator.ts
├── tests/
│   ├── unit/
│   │   ├── userService.test.ts
│   │   └── postService.test.ts
│   └── integration/
│       └── api.test.ts
├── prisma/                  # 如果使用 Prisma
│   └── schema.prisma
├── dist/                   # 编译输出
├── node_modules/
├── .env                    # 环境变量
├── .env.example
├── .gitignore
├── .prettierrc
├── .eslintrc.js
├── package.json
├── tsconfig.json
├── nodemon.json           # 开发热重载配置
└── Dockerfile
```

### nodemon.json 配置

```json
{
  "watch": ["src"],
  "ext": "ts,json",
  "ignore": ["src/**/*.spec.ts"],
  "exec": "ts-node --transpile-only src/server.ts",
  "env": {
    "NODE_ENV": "development"
  },
  "verbose": true,
  "delay": "1000"
}
```

### package.json scripts

```json
{
  "name": "my-express-api",
  "version": "1.0.0",
  "main": "dist/server.js",
  "type": "module",
  "scripts": {
    "dev": "nodemon",
    "build": "tsc",
    "start": "node dist/server.js",
    "start:prod": "NODE_ENV=production node dist/server.js",
    "lint": "eslint src --ext .ts",
    "lint:fix": "eslint src --ext .ts --fix",
    "format": "prettier --write \"src/**/*.ts\"",
    "test": "NODE_ENV=test jest",
    "test:watch": "NODE_ENV=test jest --watch",
    "test:coverage": "NODE_ENV=test jest --coverage",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:generate": "prisma generate",
    "db:seed": "ts-node prisma/seed.ts",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "express": "^4.18.2",
    "cors": "^2.8.5",
    "helmet": "^7.1.0",
    "compression": "^1.7.4",
    "dotenv": "^16.3.1",
    "express-rate-limit": "^7.1.5",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "zod": "^3.22.4",
    "winston": "^3.11.0",
    "morgan": "^1.10.0",
    "express-async-errors": "^3.1.1"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^20.10.0",
    "@types/cors": "^2.8.17",
    "@types/compression": "^1.7.5",
    "@types/jsonwebtoken": "^9.0.5",
    "@types/bcryptjs": "^2.4.6",
    "@types/morgan": "^1.9.9",
    "typescript": "^5.3.2",
    "ts-node": "^10.9.2",
    "nodemon": "^3.0.2",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.11",
    "ts-jest": "^29.1.1",
    "eslint": "^8.55.0",
    "prettier": "^3.1.0"
  }
}
```

### 环境变量配置

```bash
# .env.example
NODE_ENV=development
PORT=3000
HOST=localhost

# 数据库
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
DATABASE_URL_TEST=postgresql://user:password@localhost:5432/mydb_test

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key-change-in-production
JWT_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d

# 第三方服务
OPENAI_API_KEY=sk-...
SENDGRID_API_KEY=SG....
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...

# CORS
ALLOWED_ORIGINS=http://localhost:3000,https://example.com

# 限流
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW_MS=60000

# 日志
LOG_LEVEL=info
LOG_FILE_PATH=./logs/app.log
```

### Docker 部署配置

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# 复制依赖文件
COPY package*.json ./
RUN npm ci --only=production

# 复制源码并构建
COPY tsconfig.json ./
COPY src ./src
RUN npm run build

# 生产镜像
FROM node:20-alpine AS production

WORKDIR /app

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && adduser -S express -u 1001

# 从构建阶段复制
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

# 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000

# 切换用户
USER express

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# 启动命令
CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
version: "3.8"

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - app-network

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: mydb
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - app-network

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

### PM2 进程管理

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: 'my-express-api',
      script: 'dist/server.js',
      instances: 'max',
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 3000
      },
      max_memory_restart: '1G',
      error_file: './logs/error.log',
      out_file: './logs/out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
      autorestart: true,
      watch: false,
      max_restarts: 10,
      min_uptime: '10s',
      restart_delay: 4000
    }
  ]
};
```

```bash
# 常用 PM2 命令
npm install -g pm2

# 启动应用
pm2 start ecosystem.config.js

# 集群模式（充分利用多核）
pm2 start ecosystem.config.js --env production

# 查看状态
pm2 list
pm2 monit              # 实时监控
pm2 logs my-express-api

# 重启和重载
pm2 restart my-express-api
pm2 reload my-express-api  # 零 downtime 重载

# 停止和删除
pm2 stop my-express-api
pm2 delete my-express-api

# 保存进程列表
pm2 save

# 开机自启
pm2 startup
pm2 unstartup

# 负载均衡状态
pm2 show my-express-api
```

---

## 数据库集成详解

### Prisma ORM 集成

```bash
npm install prisma @prisma/client
npm install --save-dev prisma
npx prisma init
```

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  username  String   @unique
  password  String
  firstName String?
  lastName  String?
  avatar    String?
  role      Role     @default(USER)
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  lastLoginAt DateTime?

  posts     Post[]
  comments  Comment[]

  @@index([email])
  @@index([createdAt])
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

model Post {
  id        String   @id @default(uuid())
  title     String
  slug      String   @unique
  content   String
  excerpt   String?
  featuredImage String?
  published  Boolean  @default(false)
  publishedAt DateTime?
  viewCount  Int      @default(0)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  comments  Comment[]

  @@index([slug])
  @@index([authorId])
  @@index([publishedAt])
  @@index([published])
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String
  post      Post     @relation(fields: [postId], references: [id])
  postId    String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([authorId])
}
```

```typescript
// src/config/database.ts
import { PrismaClient } from '@prisma/client';

declare global {
  var prisma: PrismaClient | undefined;
}

export const prisma = globalThis.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' 
    ? ['query', 'error', 'warn']
    : ['error'],
});

if (process.env.NODE_ENV !== 'production') {
  globalThis.prisma = prisma;
}

// 优雅关闭
process.on('beforeExit', async () => {
  await prisma.$disconnect();
});

process.on('SIGINT', async () => {
  await prisma.$disconnect();
  process.exit(0);
});
```

```typescript
// src/services/userService.ts
import { prisma } from '../config/database.js';
import { AppError } from '../errors/AppError.js';
import bcrypt from 'bcryptjs';

export class UserService {
  async findAll(options: {
    page?: number;
    limit?: number;
    search?: string;
    role?: string;
  }) {
    const { page = 1, limit = 20, search, role } = options;
    const skip = (page - 1) * limit;

    const where: any = {};
    
    if (search) {
      where.OR = [
        { email: { contains: search, mode: 'insensitive' } },
        { username: { contains: search, mode: 'insensitive' } },
      ];
    }
    
    if (role) {
      where.role = role;
    }

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        select: {
          id: true,
          email: true,
          username: true,
          firstName: true,
          lastName: true,
          role: true,
          isActive: true,
          createdAt: true,
          _count: {
            select: { posts: true, comments: true }
          }
        },
        skip,
        take: limit,
        orderBy: { createdAt: 'desc' }
      }),
      prisma.user.count({ where })
    ]);

    return {
      data: users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    };
  }

  async findById(id: string) {
    const user = await prisma.user.findUnique({
      where: { id },
      select: {
        id: true,
        email: true,
        username: true,
        firstName: true,
        lastName: true,
        avatar: true,
        role: true,
        isActive: true,
        createdAt: true,
        updatedAt: true,
        posts: {
          take: 10,
          orderBy: { createdAt: 'desc' },
          select: {
            id: true,
            title: true,
            slug: true,
            published: true,
            createdAt: true
          }
        }
      }
    });

    if (!user) {
      throw new AppError('User not found', 404, 'USER_NOT_FOUND');
    }

    return user;
  }

  async create(data: {
    email: string;
    username: string;
    password: string;
    firstName?: string;
    lastName?: string;
    role?: string;
  }) {
    const existingUser = await prisma.user.findFirst({
      where: {
        OR: [
          { email: data.email },
          { username: data.username }
        ]
      }
    });

    if (existingUser) {
      throw new AppError(
        existingUser.email === data.email 
          ? 'Email already in use' 
          : 'Username already taken',
        409,
        'USER_EXISTS'
      );
    }

    const hashedPassword = await bcrypt.hash(data.password, 12);

    const user = await prisma.user.create({
      data: {
        ...data,
        password: hashedPassword
      },
      select: {
        id: true,
        email: true,
        username: true,
        role: true,
        createdAt: true
      }
    });

    return user;
  }

  async update(id: string, data: {
    email?: string;
    username?: string;
    firstName?: string;
    lastName?: string;
    avatar?: string;
  }) {
    const user = await prisma.user.findUnique({ where: { id } });
    
    if (!user) {
      throw new AppError('User not found', 404, 'USER_NOT_FOUND');
    }

    if (data.email || data.username) {
      const existingUser = await prisma.user.findFirst({
        where: {
          AND: [
            { id: { not: id } },
            {
              OR: [
                data.email ? { email: data.email } : {},
                data.username ? { username: data.username } : {}
              ].filter(Boolean)
            }
          ]
        }
      });

      if (existingUser) {
        throw new AppError('Email or username already in use', 409, 'USER_EXISTS');
      }
    }

    return prisma.user.update({
      where: { id },
      data,
      select: {
        id: true,
        email: true,
        username: true,
        firstName: true,
        lastName: true,
        avatar: true,
        role: true,
        updatedAt: true
      }
    });
  }

  async delete(id: string) {
    const user = await prisma.user.findUnique({ where: { id } });
    
    if (!user) {
      throw new AppError('User not found', 404, 'USER_NOT_FOUND');
    }

    // 软删除（推荐）
    await prisma.user.update({
      where: { id },
      data: { isActive: false }
    });

    // 或硬删除：
    // await prisma.user.delete({ where: { id } });
    
    return { success: true };
  }

  async updatePassword(id: string, currentPassword: string, newPassword: string) {
    const user = await prisma.user.findUnique({
      where: { id },
      select: { id: true, password: true }
    });

    if (!user) {
      throw new AppError('User not found', 404, 'USER_NOT_FOUND');
    }

    const isValid = await bcrypt.compare(currentPassword, user.password);
    if (!isValid) {
      throw new AppError('Current password is incorrect', 401, 'INVALID_PASSWORD');
    }

    const hashedPassword = await bcrypt.hash(newPassword, 12);
    
    await prisma.user.update({
      where: { id },
      data: { password: hashedPassword }
    });

    return { success: true };
  }
}

export const userService = new UserService();
```

### Knex.js 查询构建器

```bash
npm install knex pg
npm install --save-dev @types/knex
```

```typescript
// src/config/database.ts
import knex, { Knex } from 'knex';

let db: Knex;

if (process.env.NODE_ENV === 'production') {
  db = knex({
    client: 'pg',
    connection: process.env.DATABASE_URL,
    pool: {
      min: 2,
      max: 10,
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 2000
    },
    migrations: {
      directory: './migrations',
      tableName: 'knex_migrations'
    },
    seeds: {
      directory: './seeds'
    }
  });
} else {
  db = knex({
    client: 'pg',
    connection: process.env.DATABASE_URL || 'postgresql://localhost:5432/mydb',
    pool: {
      min: 2,
      max: 5
    },
    migrations: {
      directory: './migrations'
    }
  });
}

export default db;
```

```typescript
// src/models/userModel.ts
import db from '../config/database.js';

export interface User {
  id: string;
  email: string;
  username: string;
  password: string;
  first_name?: string;
  last_name?: string;
  avatar?: string;
  role: 'USER' | 'ADMIN' | 'MODERATOR';
  is_active: boolean;
  created_at: Date;
  updated_at: Date;
}

export class UserModel {
  async findAll(options: {
    page?: number;
    limit?: number;
    search?: string;
    role?: string;
  } = {}) {
    const { page = 1, limit = 20, search, role } = options;
    const offset = (page - 1) * limit;

    let query = db('users')
      .select(
        'id',
        'email',
        'username',
        'first_name',
        'last_name',
        'role',
        'is_active',
        'created_at'
      );

    if (search) {
      query = query.where((builder) => {
        builder
          .whereILike('email', `%${search}%`)
          .orWhereILike('username', `%${search}%`);
      });
    }

    if (role) {
      query = query.where({ role });
    }

    const [users, [{ count }]] = await Promise.all([
      query
        .orderBy('created_at', 'desc')
        .limit(limit)
        .offset(offset),
      db('users').count('id as count').first()
    ]);

    return {
      data: users,
      pagination: {
        page,
        limit,
        total: Number(count),
        pages: Math.ceil(Number(count) / limit)
      }
    };
  }

  async findById(id: string) {
    return db('users')
      .select(
        'id',
        'email',
        'username',
        'first_name',
        'last_name',
        'avatar',
        'role',
        'is_active',
        'created_at',
        'updated_at'
      )
      .where({ id })
      .first();
  }

  async findByEmail(email: string) {
    return db('users')
      .select('*')
      .where({ email })
      .first();
  }

  async create(data: Partial<User>) {
    const [user] = await db('users')
      .insert(data)
      .returning([
        'id',
        'email',
        'username',
        'role',
        'created_at'
      ]);
    
    return user;
  }

  async update(id: string, data: Partial<User>) {
    const [user] = await db('users')
      .where({ id })
      .update({
        ...data,
        updated_at: db.fn.now()
      })
      .returning([
        'id',
        'email',
        'username',
        'first_name',
        'last_name',
        'avatar',
        'role',
        'updated_at'
      ]);
    
    return user;
  }

  async delete(id: string) {
    return db('users')
      .where({ id })
      .delete();
  }

  async count(where: Partial<User> = {}) {
    const [{ count }] = await db('users')
      .count('id as count')
      .where(where);
    return Number(count);
  }

  async transaction<T>(callback: (trx: Knex.Transaction) => Promise<T>) {
    return db.transaction(callback);
  }
}

export const userModel = new UserModel();
```

---

## 认证授权详解

### JWT 完整实现

```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { AppError } from '../errors/AppError.js';
import { prisma } from '../config/database.js';

export interface JwtPayload {
  sub: string;
  email: string;
  role: string;
  iat: number;
  exp: number;
}

export interface AuthenticatedRequest extends Request {
  user?: {
    id: string;
    email: string;
    role: string;
  };
}

declare global {
  namespace Express {
    interface Request {
      user?: JwtPayload;
    }
  }
}

// JWT 验证中间件
export const authenticate = async (
  req: AuthenticatedRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader?.startsWith('Bearer ')) {
      throw new AppError('No token provided', 401, 'NO_TOKEN');
    }

    const token = authHeader.substring(7);
    const secret = process.env.JWT_SECRET;

    if (!secret) {
      throw new AppError('JWT secret not configured', 500, 'CONFIG_ERROR');
    }

    const decoded = jwt.verify(token, secret) as JwtPayload;
    
    // 可选：验证用户仍然存在且活跃
    const user = await prisma.user.findUnique({
      where: { id: decoded.sub, isActive: true },
      select: { id: true, email: true, role: true, isActive: true }
    });

    if (!user) {
      throw new AppError('User no longer exists', 401, 'USER_NOT_FOUND');
    }

    req.user = user;
    next();
  } catch (error) {
    if (error instanceof jwt.JsonWebTokenError) {
      next(new AppError('Invalid token', 401, 'INVALID_TOKEN'));
    } else if (error instanceof jwt.TokenExpiredError) {
      next(new AppError('Token expired', 401, 'TOKEN_EXPIRED'));
    } else {
      next(error);
    }
  }
};

// 角色验证中间件工厂
export const authorize = (...allowedRoles: string[]) => {
  return (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return next(new AppError('Not authenticated', 401, 'NOT_AUTHENTICATED'));
    }

    if (!allowedRoles.includes(req.user.role)) {
      return next(new AppError(
        'You do not have permission to perform this action',
        403,
        'FORBIDDEN'
      ));
    }

    next();
  };
};

// 资源所有权验证
export const ownership = (
  resourceOwnerId: string | ((req: Request) => string | Promise<string>)
) => {
  return async (req: AuthenticatedRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return next(new AppError('Not authenticated', 401, 'NOT_AUTHENTICATED'));
    }

    // Admin 可以访问所有资源
    if (req.user.role === 'ADMIN') {
      return next();
    }

    const ownerId = typeof resourceOwnerId === 'function'
      ? await resourceOwnerId(req)
      : resourceOwnerId;

    if (req.user.sub !== ownerId) {
      return next(new AppError(
        'You do not have permission to access this resource',
        403,
        'FORBIDDEN'
      ));
    }

    next();
  };
};

// 签发 Token
export const generateTokens = async (user: {
  id: string;
  email: string;
  role: string;
}) => {
  const secret = process.env.JWT_SECRET!;
  const accessExpiresIn = process.env.JWT_EXPIRES_IN || '15m';
  const refreshExpiresIn = process.env.JWT_REFRESH_EXPIRES_IN || '7d';

  const accessToken = jwt.sign(
    { sub: user.id, email: user.email, role: user.role },
    secret,
    { expiresIn: accessExpiresIn }
  );

  const refreshToken = jwt.sign(
    { sub: user.id, type: 'refresh' },
    secret,
    { expiresIn: refreshExpiresIn }
  );

  return { accessToken, refreshToken };
};

// 刷新 Token
export const refreshAccessToken = async (refreshToken: string) => {
  const secret = process.env.JWT_SECRET!;

  try {
    const decoded = jwt.verify(refreshToken, secret) as JwtPayload & { type?: string };
    
    if (decoded.type !== 'refresh') {
      throw new AppError('Invalid refresh token', 401, 'INVALID_TOKEN');
    }

    const user = await prisma.user.findUnique({
      where: { id: decoded.sub, isActive: true },
      select: { id: true, email: true, role: true }
    });

    if (!user) {
      throw new AppError('User not found', 401, 'USER_NOT_FOUND');
    }

    const tokens = await generateTokens(user);
    return tokens;
  } catch (error) {
    if (error instanceof jwt.JsonWebTokenError) {
      throw new AppError('Invalid refresh token', 401, 'INVALID_TOKEN');
    }
    throw error;
  }
};
```

### Session 认证实现

```typescript
// src/middleware/session.ts
import session from 'express-session';
import connectRedis from 'connect-redis';
import { RedisStore } from 'connect-redis';
import { prisma } from '../config/database.js';

// Redis Store 配置
const redisStore = new RedisStore({
  client: redisClient,
  prefix: 'sess:'
});

// Session 配置
export const sessionMiddleware = session({
  store: redisStore,
  secret: process.env.SESSION_SECRET!,
  name: 'sessionId',
  resave: false,
  saveUninitialized: false,
  rolling: true,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 天
    sameSite: 'lax'
  }
});

// Session 用户扩展
declare module 'express-session' {
  interface SessionData {
    userId?: string;
    userRole?: string;
  }
}

// Session 中间件
export const sessionAuth = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (!req.session.userId) {
    return next(new AppError('Not authenticated', 401, 'NOT_AUTHENTICATED'));
  }

  const user = await prisma.user.findUnique({
    where: { id: req.session.userId },
    select: { id: true, email: true, role: true, isActive: true }
  });

  if (!user || !user.isActive) {
    req.session.destroy((err) => {
      if (err) console.error('Session destroy error:', err);
    });
    return next(new AppError('Session invalid', 401, 'SESSION_INVALID'));
  }

  (req as any).user = user;
  next();
};
```

### OAuth 2.0 实现

```typescript
// src/routes/authRoutes.ts
import { Router } from 'express';
import crypto from 'crypto';
import axios from 'axios';
import { prisma } from '../config/database.js';
import { generateTokens } from '../middleware/auth.js';

const router = Router();

// Google OAuth
const googleOAuth = {
  clientId: process.env.GOOGLE_CLIENT_ID!,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
  redirectUri: process.env.GOOGLE_REDIRECT_URI!
};

router.get('/auth/google', (req, res) => {
  const state = crypto.randomBytes(16).toString('hex');
  req.session.oauthState = state;

  const params = new URLSearchParams({
    client_id: googleOAuth.clientId,
    redirect_uri: googleOAuth.redirectUri,
    response_type: 'code',
    scope: 'email profile',
    state,
    access_type: 'offline',
    prompt: 'consent'
  });

  res.redirect(`https://accounts.google.com/o/oauth2/v2/auth?${params}`);
});

router.get('/auth/google/callback', async (req, res, next) => {
  try {
    const { code, state, error } = req.query;

    if (error) {
      throw new AppError(`OAuth error: ${error}`, 400, 'OAUTH_ERROR');
    }

    if (state !== req.session.oauthState) {
      throw new AppError('Invalid state', 400, 'INVALID_STATE');
    }

    // 交换 Access Token
    const tokenResponse = await axios.post(
      'https://oauth2.googleapis.com/token',
      {
        code,
        client_id: googleOAuth.clientId,
        client_secret: googleOAuth.clientSecret,
        redirect_uri: googleOAuth.redirectUri,
        grant_type: 'authorization_code'
      }
    );

    const { access_token, refresh_token } = tokenResponse.data;

    // 获取用户信息
    const userResponse = await axios.get(
      'https://www.googleapis.com/oauth2/v2/userinfo',
      {
        headers: { Authorization: `Bearer ${access_token}` }
      }
    );

    const { email, name, picture } = userResponse.data;

    // 查找或创建用户
    let user = await prisma.user.findUnique({
      where: { email }
    });

    if (!user) {
      user = await prisma.user.create({
        data: {
          email,
          username: email.split('@')[0],
          firstName: name,
          avatar: picture,
          role: 'USER',
          isActive: true
        }
      });
    }

    // 生成 JWT
    const tokens = await generateTokens(user);

    // 返回 token（实际应用中建议通过 HttpOnly Cookie）
    res.json({
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
        role: user.role
      },
      ...tokens
    });
  } catch (error) {
    next(error);
  }
});

// GitHub OAuth
router.get('/auth/github', (req, res) => {
  const state = crypto.randomBytes(16).toString('hex');
  req.session.oauthState = state;

  const params = new URLSearchParams({
    client_id: process.env.GITHUB_CLIENT_ID!,
    redirect_uri: process.env.GITHUB_REDIRECT_URI!,
    scope: 'user:email',
    state
  });

  res.redirect(`https://github.com/login/oauth/authorize?${params}`);
});

export default router;
```

---

## API 设计最佳实践

### RESTful API 设计规范

```typescript
// src/routes/api/v1/userRoutes.ts
import { Router } from 'express';
import { userController } from '../controllers/userController.js';
import { authenticate, authorize } from '../middleware/auth.js';
import { validateRequest } from '../middleware/validate.js';
import { z } from 'zod';

const router = Router();

// 查询参数验证 Schema
const listUsersSchema = z.object({
  query: z.object({
    page: z.coerce.number().int().positive().default(1),
    limit: z.coerce.number().int().min(1).max(100).default(20),
    search: z.string().optional(),
    role: z.enum(['USER', 'ADMIN', 'MODERATOR']).optional(),
    sort: z.enum(['created_at', 'email', 'username']).default('created_at'),
    order: z.enum(['asc', 'desc']).default('desc')
  })
});

// 创建用户 Schema
const createUserSchema = z.object({
  body: z.object({
    email: z.string().email('Invalid email format'),
    username: z.string()
      .min(3, 'Username must be at least 3 characters')
      .max(30, 'Username must be at most 30 characters')
      .regex(/^[a-zA-Z0-9_]+$/, 'Username can only contain letters, numbers, and underscores'),
    password: z.string()
      .min(8, 'Password must be at least 8 characters')
      .regex(
        /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
        'Password must contain uppercase, lowercase and number'
      ),
    firstName: z.string().max(50).optional(),
    lastName: z.string().max(50).optional(),
    role: z.enum(['USER', 'ADMIN', 'MODERATOR']).optional()
  })
});

// 更新用户 Schema
const updateUserSchema = z.object({
  body: z.object({
    email: z.string().email().optional(),
    username: z.string()
      .min(3)
      .max(30)
      .regex(/^[a-zA-Z0-9_]+$/)
      .optional(),
    firstName: z.string().max(50).optional().nullable(),
    lastName: z.string().max(50).optional().nullable(),
    avatar: z.string().url().optional().nullable()
  }),
  params: z.object({
    id: z.string().uuid()
  })
});

// 密码更新 Schema
const updatePasswordSchema = z.object({
  body: z.object({
    currentPassword: z.string().min(1),
    newPassword: z.string()
      .min(8)
      .regex(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/)
  }),
  params: z.object({
    id: z.string().uuid()
  })
});

// 路由定义
router.get(
  '/',
  authenticate,
  authorize('ADMIN'),
  validateRequest(listUsersSchema),
  userController.list
);

router.get('/:id', authenticate, userController.show);

router.post(
  '/',
  authenticate,
  authorize('ADMIN'),
  validateRequest(createUserSchema),
  userController.create
);

router.patch(
  '/:id',
  authenticate,
  validateRequest(updateUserSchema),
  userController.update
);

router.patch(
  '/:id/password',
  authenticate,
  validateRequest(updatePasswordSchema),
  userController.updatePassword
);

router.delete(
  '/:id',
  authenticate,
  authorize('ADMIN'),
  userController.delete
);

export default router;
```

### OpenAPI/Swagger 文档

```typescript
// src/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';

const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My Express API',
      version: '1.0.0',
      description: 'A RESTful API with Express and TypeScript',
      contact: {
        name: 'API Support',
        email: 'support@example.com'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com',
        description: 'Production server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      },
      schemas: {
        User: {
          type: 'object',
          properties: {
            id: { type: 'string', format: 'uuid' },
            email: { type: 'string', format: 'email' },
            username: { type: 'string' },
            firstName: { type: 'string' },
            lastName: { type: 'string' },
            role: { type: 'string', enum: ['USER', 'ADMIN', 'MODERATOR'] },
            createdAt: { type: 'string', format: 'date-time' }
          }
        },
        Error: {
          type: 'object',
          properties: {
            error: {
              type: 'object',
              properties: {
                message: { type: 'string' },
                code: { type: 'string' }
              }
            }
          }
        },
        Pagination: {
          type: 'object',
          properties: {
            page: { type: 'integer' },
            limit: { type: 'integer' },
            total: { type: 'integer' },
            pages: { type: 'integer' }
          }
        }
      }
    },
    tags: [
      { name: 'users', description: 'User management endpoints' },
      { name: 'auth', description: 'Authentication endpoints' }
    ]
  },
  apis: ['./src/routes/*.ts']
};

export const swaggerSpec = swaggerJsdoc(options);
```

---

## 性能优化深度指南

### 请求处理优化

```typescript
// src/middleware/requestOptimizer.ts
import compression from 'compression';
import express from 'express';

// GZip 压缩中间件
export const compressionMiddleware = compression({
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  },
  level: 6,
  threshold: 1024
});

// Response Time 头
export const responseTimeHeader = (
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) => {
  const start = process.hrtime.bigint();
  
  res.on('finish', () => {
    const end = process.hrtime.bigint();
    const duration = Number(end - start) / 1e6; // 转换为毫秒
    res.setHeader('X-Response-Time', `${duration.toFixed(2)}ms`);
  });
  
  next();
};

// 请求 ID 生成
export const requestId = (
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) => {
  const id = req.headers['x-request-id'] as string 
    || crypto.randomUUID();
  
  res.setHeader('X-Request-Id', id);
  (req as any).requestId = id;
  next();
};

// 禁用 X-Powered-By
export const hidePoweredBy = (
  req: express.Request,
  res: express.Response,
  next: express.NextFunction
) => {
  res.removeHeader('X-Powered-By');
  next();
};
```

### 缓存策略

```typescript
// src/middleware/cache.ts
import { Request, Response, NextFunction } from 'express';

interface CacheOptions {
  duration: number; // 秒
  varyBy?: string[];
}

export const cache = (options: CacheOptions) => {
  const { duration, varyBy = [] } = options;

  return (req: Request, res: Response, next: NextFunction) => {
    // 仅缓存 GET 请求
    if (req.method !== 'GET') {
      return next();
    }

    const varyHeaders = varyBy.map(h => req.headers[h.toLowerCase()] as string).join(':');
    const cacheKey = `${req.originalUrl}:${varyHeaders}`;

    // 检查内存缓存
    const cached = memoryCache.get(cacheKey);
    if (cached) {
      res.setHeader('X-Cache', 'HIT');
      return res.json(cached);
    }

    // 拦截响应进行缓存
    const originalJson = res.json.bind(res);
    res.json = (body: any) => {
      if (res.statusCode === 200) {
        memoryCache.set(cacheKey, body, duration * 1000);
      }
      return originalJson(body);
    };

    next();
  };
};

// 简单内存缓存实现
class MemoryCache {
  private cache = new Map<string, { data: any; expiresAt: number }>();

  get(key: string): any | null {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return item.data;
  }

  set(key: string, data: any, ttl: number) {
    this.cache.set(key, {
      data,
      expiresAt: Date.now() + ttl
    });
  }

  delete(key: string) {
    this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  // 定期清理过期项
  startCleanup(intervalMs = 60000) {
    setInterval(() => {
      const now = Date.now();
      for (const [key, item] of this.cache) {
        if (now > item.expiresAt) {
          this.cache.delete(key);
        }
      }
    }, intervalMs);
  }
}

export const memoryCache = new MemoryCache();
memoryCache.startCleanup();
```

### 数据库查询优化

```typescript
// src/middleware/queryOptimizer.ts
import { Request, Response, NextFunction } from 'express';

// N+1 查询检测（开发环境）
export const queryCounter = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (process.env.NODE_ENV !== 'development') {
    return next();
  }

  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    if (duration > 1000) {
      console.warn(`⚠️ Slow query: ${req.method} ${req.originalUrl} took ${duration}ms`);
    }
  });

  next();
};

// 响应压缩
export const responseCompression = (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const acceptEncoding = req.headers['accept-encoding'] || '';
  
  if (acceptEncoding.includes('gzip')) {
    res.setHeader('Content-Encoding', 'gzip');
  } else if (acceptEncoding.includes('br')) {
    res.setHeader('Content-Encoding', 'br');
  }
  
  next();
};
```

### 限流策略

```typescript
// src/middleware/rateLimiter.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { redisClient } from '../config/redis.js';

// 通用 API 限流
export const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100,
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    error: {
      message: 'Too many requests, please try again later',
      code: 'RATE_LIMIT_EXCEEDED'
    }
  },
  skip: (req) => req.path === '/health'
});

// 登录限流（更严格）
export const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,
  message: {
    error: {
      message: 'Too many login attempts',
      code: 'LOGIN_LIMIT_EXCEEDED'
    }
  }
});

// 注册限流
export const registerLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 小时
  max: 3,
  message: {
    error: {
      message: 'Too many registration attempts',
      code: 'REGISTER_LIMIT_EXCEEDED'
    }
  }
});

// API 密钥限流（使用 Redis）
export const apiKeyLimiter = rateLimit({
  store: new RedisStore({
    sendCommand: (...args: string[]) => redisClient.sendCommand(args)
  }),
  windowMs: 60 * 1000,
  max: 60,
  keyGenerator: (req) => req.headers['x-api-key'] as string || req.ip,
  message: {
    error: {
      message: 'API rate limit exceeded',
      code: 'API_LIMIT_EXCEEDED'
    }
  }
});
```

---

## 测试策略

### Jest 配置

```javascript
// jest.config.js
/** @type {import('jest').Config} */
export default {
  testEnvironment: 'node',
  extensionsToTreatAsEsm: ['.ts'],
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1',
    '^(\\.{1,2}/.*)\\.ts$': '$1'
  },
  transform: {
    '^.+\\.tsx?$': [
      'ts-jest',
      {
        useESM: true,
        tsconfig: {
          module: 'NodeNext',
          moduleResolution: 'NodeNext'
        }
      }
    ]
  },
  testMatch: [
    '**/__tests__/**/*.test.ts',
    '**/?(*.)+(spec|test).ts'
  ],
  testPathIgnorePatterns: ['/node_modules/', '/dist/'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/types/**'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  },
  setupFilesAfterEnv: ['./tests/setup.ts'],
  verbose: true
};
```

### 测试示例

```typescript
// tests/setup.ts
import { jest } from '@jest/globals';

// 全局超时设置
jest.setTimeout(10000);

// Mock 环境变量
process.env.JWT_SECRET = 'test-secret-key';
process.env.DATABASE_URL = 'postgresql://localhost:5432/test_db';
process.env.NODE_ENV = 'test';
```

```typescript
// tests/unit/userService.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach, jest } from '@jest/globals';
import { UserService } from '../../src/services/userService.js';
import { prisma } from '../../src/config/database.js';

// Mock Prisma
jest.mock('../../src/config/database.js', () => ({
  prisma: {
    user: {
      findMany: jest.fn(),
      findUnique: jest.fn(),
      findFirst: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
      count: jest.fn()
    }
  }
}));

describe('UserService', () => {
  let userService: UserService;

  beforeAll(() => {
    userService = new UserService();
  });

  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('findAll', () => {
    it('should return paginated users', async () => {
      const mockUsers = [
        { id: '1', email: 'user1@test.com', username: 'user1' },
        { id: '2', email: 'user2@test.com', username: 'user2' }
      ];

      (prisma.user.findMany as jest.Mock).mockResolvedValue(mockUsers);
      (prisma.user.count as jest.Mock).mockResolvedValue(2);

      const result = await userService.findAll({ page: 1, limit: 10 });

      expect(result.data).toEqual(mockUsers);
      expect(result.pagination.total).toBe(2);
      expect(result.pagination.pages).toBe(1);
    });

    it('should filter by search query', async () => {
      (prisma.user.findMany as jest.Mock).mockResolvedValue([]);
      (prisma.user.count as jest.Mock).mockResolvedValue(0);

      await userService.findAll({ search: 'test' });

      expect(prisma.user.findMany).toHaveBeenCalledWith(
        expect.objectContaining({
          where: expect.objectContaining({
            OR: expect.arrayContaining([
              expect.objectContaining({ email: expect.anything() })
            ])
          })
        })
      );
    });
  });

  describe('create', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'new@test.com',
        username: 'newuser',
        password: 'Password123'
      };

      const createdUser = {
        id: '123',
        ...userData,
        role: 'USER',
        createdAt: new Date()
      };

      (prisma.user.findFirst as jest.Mock).mockResolvedValue(null);
      (prisma.user.create as jest.Mock).mockResolvedValue(createdUser);

      const result = await userService.create(userData);

      expect(result.email).toBe(userData.email);
      expect(result.username).toBe(userData.username);
    });

    it('should throw error if email exists', async () => {
      (prisma.user.findFirst as jest.Mock).mockResolvedValue({ email: 'existing@test.com' });

      await expect(
        userService.create({
          email: 'existing@test.com',
          username: 'newuser',
          password: 'Password123'
        })
      ).rejects.toThrow('Email already in use');
    });
  });
});
```

```typescript
// tests/integration/api.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from '@jest/globals';
import request from 'supertest';
import { app } from '../../src/app.js';
import { prisma } from '../../src/config/database.js';

describe('User API', () => {
  let authToken: string;
  let testUserId: string;

  beforeAll(async () => {
    // 创建测试用户并获取 token
    const response = await request(app)
      .post('/api/v1/auth/register')
      .send({
        email: 'test@example.com',
        username: 'testuser',
        password: 'TestPass123!'
      });

    authToken = response.body.accessToken;
    testUserId = response.body.user.id;
  });

  afterAll(async () => {
    // 清理测试数据
    await prisma.user.deleteMany({
      where: { email: { contains: '@example.com' } }
    });
  });

  describe('GET /api/v1/users', () => {
    it('should require authentication', async () => {
      const response = await request(app)
        .get('/api/v1/users');

      expect(response.status).toBe(401);
    });

    it('should return paginated users for admin', async () => {
      const response = await request(app)
        .get('/api/v1/users')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.data).toBeInstanceOf(Array);
      expect(response.body.pagination).toBeDefined();
    });
  });

  describe('GET /api/v1/users/:id', () => {
    it('should return user by id', async () => {
      const response = await request(app)
        .get(`/api/v1/users/${testUserId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.id).toBe(testUserId);
    });

    it('should return 404 for non-existent user', async () => {
      const response = await request(app)
        .get('/api/v1/users/00000000-0000-0000-0000-000000000000')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });
  });
});
```

---

## 调试与监控

### Winston 日志配置

```typescript
// src/utils/logger.ts
import winston from 'winston';
import path from 'path';

const { combine, timestamp, printf, colorize, errors, json } = winston.format;

// 自定义日志格式
const logFormat = printf(({ level, message, timestamp, stack, ...metadata }) => {
  let log = `${timestamp} [${level}]: ${message}`;
  
  if (Object.keys(metadata).length > 0) {
    log += ` ${JSON.stringify(metadata)}`;
  }
  
  if (stack) {
    log += `\n${stack}`;
  }
  
  return log;
});

// 开发环境格式
const devFormat = combine(
  colorize({ all: true }),
  timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
  errors({ stack: true }),
  logFormat
);

// 生产环境格式
const prodFormat = combine(
  timestamp({ format: 'YYYY-MM-DD HH:mm:ss.SSS Z' }),
  errors({ stack: true }),
  json()
);

const logLevel = process.env.LOG_LEVEL || 'info';

// 创建 logger 实例
export const logger = winston.createLogger({
  level: logLevel,
  defaultMeta: {
    service: 'express-api',
    version: process.env.npm_package_version
  },
  format: process.env.NODE_ENV === 'production' ? prodFormat : devFormat,
  transports: [
    // 控制台输出
    new winston.transports.Console({
      handleExceptions: true
    })
  ]
});

// 生产环境添加文件输出
if (process.env.NODE_ENV === 'production') {
  logger.add(
    new winston.transports.File({
      filename: path.join(process.env.LOG_FILE_PATH || './logs', 'error.log'),
      level: 'error',
      maxsize: 5242880, // 5MB
      maxFiles: 5
    })
  );
  
  logger.add(
    new winston.transports.File({
      filename: path.join(process.env.LOG_FILE_PATH || './logs', 'combined.log'),
      maxsize: 5242880,
      maxFiles: 5
    })
  );
}

// 替换全局 console
if (process.env.NODE_ENV === 'production') {
  Object.assign(console, logger);
}
```

### 健康检查端点

```typescript
// src/routes/healthRoutes.ts
import { Router, Request, Response } from 'express';
import { prisma } from '../config/database.js';
import { redisClient } from '../config/redis.js';

const router = Router();

interface HealthStatus {
  status: 'healthy' | 'degraded' | 'unhealthy';
  timestamp: string;
  uptime: number;
  checks: {
    database: { status: 'up' | 'down'; latency?: number };
    redis: { status: 'up' | 'down'; latency?: number };
  };
}

router.get('/health', async (req: Request, res: Response) => {
  const startMemory = process.memoryUsage();
  
  const checks: HealthStatus['checks'] = {
    database: { status: 'down' },
    redis: { status: 'down' }
  };

  // 数据库检查
  try {
    const dbStart = Date.now();
    await prisma.$queryRaw`SELECT 1`;
    checks.database = {
      status: 'up',
      latency: Date.now() - dbStart
    };
  } catch (error) {
    checks.database = { status: 'down' };
  }

  // Redis 检查
  try {
    const redisStart = Date.now();
    await redisClient.ping();
    checks.redis = {
      status: 'up',
      latency: Date.now() - redisStart
    };
  } catch (error) {
    checks.redis = { status: 'down' };
  }

  // 计算整体状态
  const allHealthy = Object.values(checks).every(c => c.status === 'up');
  const allDown = Object.values(checks).every(c => c.status === 'down');

  const status: HealthStatus['status'] = allHealthy 
    ? 'healthy' 
    : allDown 
      ? 'unhealthy' 
      : 'degraded';

  const health: HealthStatus = {
    status,
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    checks
  };

  const statusCode = status === 'healthy' ? 200 : status === 'degraded' ? 200 : 503;
  
  res.status(statusCode).json(health);
});

// 存活探针（Kubernetes）
router.get('/health/live', (req: Request, res: Response) => {
  res.status(200).json({ status: 'alive' });
});

// 就绪探针（Kubernetes）
router.get('/health/ready', async (req: Request, res: Response) => {
  try {
    await prisma.$queryRaw`SELECT 1`;
    await redisClient.ping();
    res.status(200).json({ status: 'ready' });
  } catch (error) {
    res.status(503).json({ status: 'not ready' });
  }
});

export default router;
```

---

> [!SUCCESS]
> Express 作为 Node.js 生态中最经典的 Web 框架，以其简洁的 API、丰富的中间件生态和庞大的社区支持，在 2026 年仍然是构建 API 服务和轻量级后端的首选。虽然在性能上不如 Fastify 等新兴框架，但其稳定性和学习曲线优势使其在企业级应用中占有一席之地。结合 TypeScript 和现代中间件，Express 可以构建安全、高性能的 AI 应用后端。
