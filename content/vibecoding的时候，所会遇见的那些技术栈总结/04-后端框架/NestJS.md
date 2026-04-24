# NestJS 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 NestJS 架构、依赖注入、核心模块、Guards/Interceptors/Pipes、GraphQL 集成及在 AI 应用中的定位。

---

## 目录

1. [[#NestJS 概述与核心哲学]]
2. [[#NestJS vs Express vs Fastify 对比]]
3. [[#核心架构]]
4. [[#依赖注入详解]]
5. [[#核心模块]]
6. [[#Guards/Interceptors/Pipes/Decorators]]
7. [[#微服务与消息队列]]
8. [[#GraphQL 与 WebSocket 集成]]
9. [[#AI 应用实战]]
10. [[#选型建议]]

---

## NestJS 概述与核心哲学

### 什么是 NestJS

NestJS 是一个用于构建高效、可扩展的 Node.js 服务器端应用的**渐进式框架**。它受 Angular 启发，采用模块化架构，使用 TypeScript 编写（但支持纯 JavaScript），提供了依赖注入、装饰器模式、模块化组织等企业级特性。

**NestJS 核心特点：**

| 特性 | 说明 |
|------|------|
| **模块化架构** | Module/Controller/Provider 组织代码 |
| **依赖注入** | 强大的 IoC 容器 |
| **装饰器模式** | 类和方法的元数据标记 |
| **TypeScript 原生** | 完整的类型安全 |
| **可扩展性** | 插件和中间件生态丰富 |
| **多协议支持** | HTTP/GraphQL/WebSocket/gRPC |

### NestJS 发展历程

| 版本 | 发布年份 | 核心特性 |
|------|---------|---------|
| NestJS 1.0 | 2017 | 初始版本，核心架构 |
| NestJS 4.0 | 2017 | 稳定版发布 |
| NestJS 6.0 | 2019 | 微服务支持增强 |
| NestJS 8.0 | 2021 | 性能优化，Fastify 支持 |
| NestJS 10.0 | 2023 | 改进性能，懒加载模块 |
| **NestJS 11** | **2025** | **更好的 TypeScript 5 支持** |

### 核心理念

**NestJS 的设计哲学：**

1. **渐进式**：从简单到复杂，无缝扩展
2. **模块化**：每个功能封装在独立模块中
3. **约定优于配置**：遵循 Angular 风格的约定
4. **企业级**：依赖注入、AOP、测试支持
5. **多范式**：OOP/FP/FRP 混合编程

> [!IMPORTANT]
> NestJS 不是"另一个 Express"，而是一套完整的后端应用框架。它强制采用模块化架构，适合构建中大型企业应用。

---

## NestJS vs Express vs Fastify 对比

### 框架对比

| 特性 | NestJS | Express | Fastify |
|------|--------|---------|---------|
| **架构** | 模块化 | 中间件栈 | 插件系统 |
| **依赖注入** | 内置 | 无 | 需额外库 |
| **TypeScript** | 原生 | 需配置 | 需配置 |
| **性能** | 中等 | 良好 | 最快 |
| **学习曲线** | 较陡 | 低 | 低 |
| **生态** | 丰富 | 极丰富 | 增长中 |
| **GraphQL 支持** | 原生 | 需库 | 需库 |
| **微服务** | 内置 | 无 | 无 |
| **适用场景** | 企业应用 | API/REST | 高性能 API |

### 性能对比

| 指标 | NestJS + Express | NestJS + Fastify | Express | Fastify |
|------|-----------------|-----------------|---------|---------|
| **吞吐量** | ~15K req/s | ~30K req/s | ~25K req/s | ~45K req/s |
| **延迟** | ~5ms | ~2ms | ~3ms | ~1ms |
| **内存** | ~120MB | ~80MB | ~60MB | ~50MB |

### 代码风格对比

**Express：**

```typescript
import express from 'express';

const app = express();

app.get('/users/:id', async (req, res) => {
  const user = await getUserById(req.params.id);
  res.json(user);
});

app.post('/users', async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).json(user);
});

app.listen(3000);
```

**Fastify：**

```typescript
import Fastify from 'fastify';

const app = Fastify();

app.get('/users/:id', async (req, res) => {
  const user = await getUserById(req.params.id);
  return user;
});

app.post('/users', async (req, res) => {
  const user = await createUser(req.body);
  res.status(201).send(user);
});

app.listen({ port: 3000 });
```

**NestJS：**

```typescript
// users.controller.ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }
}
```

---

## 核心架构

### Module（模块）

模块是 NestJS 应用的基本组织单位：

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './user.entity';
import { AuthModule } from '../auth/auth.module';

@Module({
  // 导入其他模块
  imports: [
    TypeOrmModule.forFeature([User]),
    AuthModule
  ],
  // 控制器
  controllers: [UsersController],
  // 服务提供者
  providers: [UsersService],
  // 导出服务供其他模块使用
  exports: [UsersService]
})
export class UsersModule {}
```

### Controller（控制器）

控制器处理 HTTP 请求：

```typescript
// users.controller.ts
import {
  Controller,
  Get,
  Post,
  Body,
  Param,
  Query,
  HttpCode,
  HttpStatus
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto, UpdateUserDto } from './dto';
import { PaginationQueryDto } from '../common/dto';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get()
  async findAll(@Query() query: PaginationQueryDto) {
    return this.usersService.findAll(query);
  }

  @Get(':id')
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  @Patch(':id')
  async update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto
  ) {
    return this.usersService.update(id, updateUserDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

### Provider（服务）

服务是业务逻辑的主要载体：

```typescript
// users.service.ts
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { User } from './user.entity';
import { CreateUserDto, UpdateUserDto, PaginationQueryDto } from './dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>
  ) {}

  async findAll(query: PaginationQueryDto) {
    const { page = 1, limit = 10 } = query;
    const skip = (page - 1) * limit;

    const [users, total] = await this.userRepository.findAndCount({
      skip,
      take: limit,
      order: { createdAt: 'DESC' }
    });

    return {
      data: users,
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit)
      }
    };
  }

  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });
    if (!user) {
      throw new NotFoundException(`User with ID ${id} not found`);
    }
    return user;
  }

  async create(createUserDto: CreateUserDto): Promise<User> {
    const user = this.userRepository.create(createUserDto);
    return this.userRepository.save(user);
  }

  async update(id: string, updateUserDto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id);
    Object.assign(user, updateUserDto);
    return this.userRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await this.userRepository.remove(user);
  }
}
```

### 应用入口

```typescript
// main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // 全局前缀
  app.setGlobalPrefix('api/v1');

  // 全局管道 - 请求验证
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      forbidNonWhitelisted: true,
      transform: true,
      transformOptions: {
        enableImplicitConversion: true
      }
    })
  );

  // CORS
  app.enableCors();

  // Swagger 文档
  const config = new DocumentBuilder()
    .setTitle('API Documentation')
    .setDescription('API description')
    .setVersion('1.0')
    .addBearerAuth()
    .build();

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('docs', app, document);

  await app.listen(3000);
  console.log(`Application is running on: ${await app.getUrl()}`);
}

bootstrap();
```

---

## 依赖注入详解

### 依赖注入原理

NestJS 使用 TypeScript 装饰器和反射实现依赖注入：

```typescript
// 1. 定义 Token（接口或字符串）
const USER_SERVICE_TOKEN = 'USER_SERVICE';

// 2. 定义 Provider
@Injectable()
class UserService {
  findAll() { return []; }
}

// 3. 注册 Provider
@Module({
  providers: [
    { provide: USER_SERVICE_TOKEN, useClass: UserService }
  ]
})

// 4. 注入 Provider
@Controller()
export class UsersController {
  constructor(
    @Inject(USER_SERVICE_TOKEN)
    private readonly userService: UserService
  ) {}
}
```

### Provider 类型

```typescript
@Module({
  providers: [
    // 类 Provider（最常用）
    UsersService,

    // 值 Provider
    { provide: 'CONFIG', useValue: { apiKey: 'xxx' } },

    // 工厂 Provider
    {
      provide: 'DATABASE_POOL',
      useFactory: async () => {
        const pool = await createPool({ host: 'localhost' });
        return pool;
      }
    },

    // 别名 Provider
    { provide: 'AliasedService', useExisting: UsersService }
  ]
})
export class AppModule {}
```

### 作用域

```typescript
// 请求作用域 - 每次请求创建新实例
@Injectable({ scope: Scope.REQUEST })
class RequestService {
  constructor() {
    console.log('New instance per request');
  }
}

// 瞬态作用域 - 每次注入创建新实例
@Injectable({ scope: Scope.TRANSIENT })
class TransientService {}

// 单例作用域（默认）- 应用生命周期内唯一实例
@Injectable()
class SingletonService {}
```

### 模块间的依赖

```typescript
// auth.module.ts
@Module({
  providers: [AuthService],
  exports: [AuthService]  // 导出才能被其他模块使用
})
export class AuthModule {}

// users.module.ts
@Module({
  imports: [AuthModule],  // 导入其他模块
  controllers: [UsersController],
  providers: [UsersService]
})
export class UsersModule {}
```

---

## 核心模块

### 常用内置模块

| 模块 | 功能 | npm 包 |
|------|------|--------|
| `@nestjs/core` | 核心功能 | 内置 |
| `@nestjs/common` | 公共装饰器/管道 | 内置 |
| `@nestjs/platform-express` | Express 适配器 | 内置 |
| `@nestjs/platform-fastify` | Fastify 适配器 | 内置 |
| `@nestjs/typeorm` | TypeORM 集成 | @nestjs/typeorm |
| `@nestjs/sequelize` | Sequelize 集成 | @nestjs/sequelize |
| `@nestjs/graphql` | GraphQL 集成 | @nestjs/graphql |
| `@nestjs/microservices` | 微服务支持 | 内置 |
| `@nestjs/websockets` | WebSocket 支持 | 内置 |
| `@nestjs/swagger` | Swagger 文档 | @nestjs/swagger |
| `@nestjs/jwt` | JWT 认证 | @nestjs/jwt |
| `@nestjs/passport` | Passport 认证 | @nestjs/passport |

### TypeORM 集成

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'user',
      password: 'password',
      database: 'db',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: false,  // 生产环境禁用
      logging: true
    })
  ]
})
export class AppModule {}
```

### GraphQL 集成

```typescript
// app.module.ts
import { Module } from '@nestjs/common';
import { GraphQLModule } from '@nestjs/graphql';
import { ApolloDriver, ApolloDriverConfig } from '@nestjs/apollo';
import { join } from 'path';

@Module({
  imports: [
    GraphQLModule.forRoot<ApolloDriverConfig>({
      driver: ApolloDriver,
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      sortSchema: true,
      playground: true,
      subscriptions: {
        'graphql-ws': true
      }
    })
  ]
})
export class AppModule {}
```

---

## Guards/Interceptors/Pipes/Decorators

### Guards（守卫）

守卫用于权限控制和路由保护：

```typescript
// auth.guard.ts
import {
  Injectable,
  CanActivate,
  ExecutionContext,
  UnauthorizedException
} from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private reflector: Reflector
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException('No token provided');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request['user'] = payload;
    } catch {
      throw new UnauthorizedException('Invalid token');
    }

    return true;
  }

  private extractTokenFromHeader(request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [
      context.getHandler(),
      context.getClass()
    ]);

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.roles?.includes(role));
  }
}

// 使用守卫
@Controller('users')
@UseGuards(JwtAuthGuard, RolesGuard)
export class UsersController {
  @SetMetadata('roles', ['admin'])
  @Get('protected')
  getProtected() {
    return 'This is protected';
  }
}
```

### Interceptors（拦截器）

拦截器用于请求/响应转换：

```typescript
// logging.interceptor.ts
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const now = Date.now();

    return next
      .handle()
      .pipe(
        tap(() => {
          const response = context.switchToHttp().getResponse();
          console.log(`${method} ${url} - ${response.statusCode} - ${Date.now() - now}ms`);
        })
      );
  }
}

// transform.interceptor.ts
@Injectable()
export class TransformInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
        timestamp: new Date().toISOString()
      }))
    );
  }
}

// caching.interceptor.ts
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(private reflector: Reflector) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const cacheTTL = this.reflector.get('cacheTTL', context.getHandler());

    if (!cacheTTL) {
      return next.handle();
    }

    const request = context.switchToHttp().getRequest();
    const cacheKey = `cache:${request.url}`;

    const cachedData = await this.cacheService.get(cacheKey);
    if (cachedData) {
      return of(cachedData);
    }

    return next.handle().pipe(
      tap((data) => {
        this.cacheService.set(cacheKey, data, cacheTTL);
      })
    );
  }
}
```

### Pipes（管道）

管道用于数据转换和验证：

```typescript
// validation.pipe.ts
import {
  PipeTransform,
  Injectable,
  ArgumentMetadata,
  BadRequestException
} from '@nestjs/common';
import { validate } from 'class-validator';
import { plainToInstance } from 'class-transformer';

@Injectable()
export class CustomValidationPipe implements PipeTransform<any> {
  async transform(value: any, { metatype }: ArgumentMetadata) {
    if (!metatype || !this.toValidate(metatype)) {
      return value;
    }

    const object = plainToInstance(metatype, value);
    const errors = await validate(object);

    if (errors.length > 0) {
      throw new BadRequestException('Validation failed');
    }

    return value;
  }

  private toValidate(metatype: Function): boolean {
    const types: Function[] = [String, Boolean, Number, Array, Object];
    return !types.includes(metatype);
  }
}

// parse-int.pipe.ts
@Injectable()
export class ParseIntPipe implements PipeTransform<string, number> {
  transform(value: string, metadata: ArgumentMetadata): number {
    const val = parseInt(value, 10);
    if (isNaN(val)) {
      throw new BadRequestException('Validation failed');
    }
    return val;
  }
}

// 使用管道
@Get(':id')
async findOne(@Param('id', ParseIntPipe) id: number) {
  return this.usersService.findOne(id);
}

@Post()
async create(
  @Body(new CustomValidationPipe()) createUserDto: CreateUserDto
) {
  return this.usersService.create(createUserDto);
}
```

### 自定义装饰器

```typescript
// current-user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    const user = request.user;

    return data ? user?.[data] : user;
  }
);

// 使用
@Get('profile')
async getProfile(@CurrentUser() user: User) {
  return user;
}

@Get('profile/:field')
async getProfileField(
  @CurrentUser('email') email: string
) {
  return email;
}

// headers.decorator.ts
import { createParamDecorator } from '@nestjs/common';

export const Headers = createParamDecorator(
  (key: string | undefined, ctx: ExecutionContext) => {
    const headers = ctx.switchToHttp().getRequest().headers;
    return key ? headers[key.toLowerCase()] : headers;
  }
);
```

---

## 微服务与消息队列

### 微服务架构

```typescript
// main.ts - HTTP 服务
import { NestFactory } from '@nestjs/core';
import { MicroserviceOptions, Transport } from '@nestjs/microservices';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.connectMicroservice<MicroserviceOptions>({
    transport: Transport.RMQ,
    options: {
      urls: ['amqp://localhost:5672'],
      queue: 'users_queue'
    }
  });

  await app.startAllMicroservices();
  await app.listen(3000);
}

bootstrap();
```

### Redis 消息队列

```typescript
// 使用 Redis 适配器
app.connectMicroservice<MicroserviceOptions>({
  transport: Transport.REDIS,
  options: {
    host: 'localhost',
    port: 6379
  }
});

// 消息处理器
@MessagePattern('user:create')
async handleUserCreate(@Payload() data: CreateUserDto) {
  return this.usersService.create(data);
}

// 事件发射
@Injectable()
export class UsersService {
  constructor(
    private readonly eventEmitter: EventEmitter2
  ) {}

  async create(dto: CreateUserDto) {
    const user = await this.userRepository.save(dto);

    // 发射事件
    this.eventEmitter.emit('user.created', { userId: user.id });

    return user;
  }
}
```

---

## GraphQL 与 WebSocket 集成

### GraphQL Resolver

```typescript
// users.resolver.ts
import { Resolver, Query, Mutation, Args, Subscription, Int } from '@nestjs/graphql';
import { User, UserConnection } from './user.entity';
import { UsersService } from './users.service';
import { CreateUserInput, UpdateUserInput } from './dto';

@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User])
  async users() {
    return this.usersService.findAll();
  }

  @Query(() => User, { nullable: true })
  async user(@Args('id') id: string) {
    return this.usersService.findOne(id);
  }

  @Query(() => UserConnection)
  async usersPaginated(
    @Args('page', { type: () => Int, defaultValue: 1 }) page: number,
    @Args('limit', { type: () => Int, defaultValue: 10 }) limit: number
  ) {
    return this.usersService.findAllPaginated(page, limit);
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput) {
    return this.usersService.create(input);
  }

  @Subscription(() => User)
  userCreated() {
    return this.pubSub.asyncIterator('userCreated');
  }
}
```

### WebSocket 网关

```typescript
// events.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: {
    origin: '*'
  }
})
export class EventsGateway {
  @WebSocketServer()
  server: Server;

  @SubscribeMessage('message')
  handleMessage(
    @MessageBody() data: string,
    @ConnectedSocket() client: Socket
  ) {
    console.log(`Message from ${client.id}: ${data}`);
    this.server.emit('message', { client: client.id, message: data });
  }

  @SubscribeMessage('join')
  handleJoin(
    @MessageBody() room: string,
    @ConnectedSocket() client: Socket
  ) {
    client.join(room);
    return { event: 'joined', room };
  }

  @SubscribeMessage('leave')
  handleLeave(
    @MessageBody() room: string,
    @ConnectedSocket() client: Socket
  ) {
    client.leave(room);
    return { event: 'left', room };
  }
}
```

---

## AI 应用实战

### AI API 代理

```typescript
// ai.service.ts
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { OpenAI } from 'openai';

@Injectable()
export class AIService {
  private openai: OpenAI;

  constructor(private configService: ConfigService) {
    this.openai = new OpenAI({
      apiKey: this.configService.get('OPENAI_API_KEY')
    });
  }

  async chat(messages: Array<{ role: string; content: string }>) {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4o',
      messages
    });

    return {
      content: completion.choices[0].message.content,
      usage: completion.usage
    };
  }

  async streamChat(messages: Array<{ role: string; content: string }>) {
    return this.openai.chat.completions.create({
      model: 'gpt-4o',
      messages,
      stream: true
    });
  }
}
```

### AI 控制器

```typescript
// ai.controller.ts
import {
  Controller,
  Post,
  Body,
  Sse,
  Header
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { AIService } from './ai.service';
import { CreateChatCompletionDto } from './dto';

@Controller('ai')
export class AIController {
  constructor(private readonly aiService: AIService) {}

  @Post('chat')
  async chat(@Body() dto: CreateChatCompletionDto) {
    return this.aiService.chat(dto.messages);
  }

  @Post('chat/stream')
  async streamChat(@Body() dto: CreateChatCompletionDto) {
    const stream = await this.aiService.streamChat(dto.messages);

    return new ReadableStream({
      async start(controller) {
        for await (const chunk of stream) {
          const text = chunk.choices[0]?.delta?.content || '';
          if (text) {
            controller.enqueue(`data: ${JSON.stringify({ text })}\n\n`);
          }
        }
        controller.close();
      }
    });
  }
}
```

---

## 选型建议

### 何时选择 NestJS

| 场景 | 推荐理由 |
|------|---------|
| **企业级应用** | 模块化架构，依赖注入 |
| **大型 API** | TypeScript 原生，强类型 |
| **微服务架构** | 内置微服务支持 |
| **GraphQL API** | 原生 GraphQL 集成 |
| **团队有 Angular 经验** | 风格相似 |
| **需要测试支持** | 完善的测试工具 |

### 何时考虑其他方案

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| **简单 REST API** | Express / Fastify | 轻量，灵活 |
| **高性能 API** | Fastify | 极快速度 |
| **Serverless** | Express / 框架无关 | 冷启动优化 |
| **小型项目** | Express | 学习成本低 |
| **实时应用** | Socket.io / 原生 | 更底层 |

> [!TIP]
> NestJS 的模块化架构和依赖注入系统非常适合构建中大型企业应用。如果项目需要长期维护和团队协作，NestJS 是很好的选择。

---

## 完整安装与项目初始化

### 环境要求

```bash
# Node.js 版本要求
node --version  # >= 18.0.0 (推荐 20.x LTS)

# npm 版本
npm --version   # >= 9.0.0

# NestJS CLI
npm install -g @nestjs/cli
```

### 项目创建

```bash
# 创建新项目
nest new project-name

# 或者使用 yarn
nest new project-name --package-manager yarn

# 指定包管理器
nest new project-name --package-manager pnpm

# 跳过安装
nest new project-name --skip-git
```

### 手动初始化

```bash
mkdir my-nestjs-api && cd my-nestjs-api

# 初始化 npm
npm init -y

# 安装 NestJS 核心依赖
npm install @nestjs/common @nestjs/core @nestjs/platform-express @nestjs/config
npm install @nestjs/typeorm @nestjs/passport @nestjs/jwt @nestjs/schedule
npm install @nestjs/terminus @nestjs/swagger @nestjs/graphql @nestjs/apollo
npm install reflect-metadata rxjs

# 开发依赖
npm install --save-dev @nestjs/cli @nestjs/schematics @nestjs/testing
npm install typescript @types/node ts-node

# 初始化 TypeScript
npx tsc --init
```

### TypeScript 配置

```json
// tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "declaration": true,
    "removeComments": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "allowSyntheticDefaultImports": true,
    "target": "ES2021",
    "sourceMap": true,
    "outDir": "./dist",
    "baseUrl": "./",
    "incremental": true,
    "skipLibCheck": true,
    "strictNullChecks": true,
    "noImplicitAny": true,
    "strictBindCallApply": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "resolveJsonModule": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### 项目结构

```
my-nestjs-api/
├── src/
│   ├── main.ts                    # 应用入口
│   ├── app.module.ts              # 根模块
│   ├── app.controller.ts
│   ├── app.service.ts
│   ├── app.config.ts             # 配置
│   ├── common/
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   ├── roles.decorator.ts
│   │   │   └── public.decorator.ts
│   │   ├── guards/
│   │   │   ├── auth.guard.ts
│   │   │   ├── roles.guard.ts
│   │   │   └── throttle.guard.ts
│   │   ├── interceptors/
│   │   │   ├── logging.interceptor.ts
│   │   │   ├── transform.interceptor.ts
│   │   │   └── exception.interceptor.ts
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts
│   │   ├── pipes/
│   │   │   ├── validation.pipe.ts
│   │   │   └── parse-int.pipe.ts
│   │   ├── dto/
│   │   │   └── pagination.dto.ts
│   │   └── interfaces/
│   │       └── pagination.interface.ts
│   ├── config/
│   │   ├── configuration.ts
│   │   ├── database.config.ts
│   │   └── redis.config.ts
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.module.ts
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── strategies/
│   │   │   │   ├── jwt.strategy.ts
│   │   │   │   └── local.strategy.ts
│   │   │   ├── guards/
│   │   │   └── dto/
│   │   ├── users/
│   │   │   ├── users.module.ts
│   │   │   ├── users.controller.ts
│   │   │   ├── users.service.ts
│   │   │   ├── entities/
│   │   │   │   └── user.entity.ts
│   │   │   └── dto/
│   │   ├── posts/
│   │   │   └── ...
│   │   └── health/
│   │       └── ...
│   ├── database/
│   │   ├── database.module.ts
│   │   └── migrations/
│   └── redis/
│       └── redis.module.ts
├── test/
│   ├── jest-e2e.json
│   └── *.spec.ts
├── prisma/
│   └── schema.prisma
├── .env
├── .env.example
├── .gitignore
├── tsconfig.json
├── nest-cli.json
├── package.json
└── Dockerfile
```

### nest-cli.json 配置

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true,
    "assets": ["**/*.graphql"],
    "watchAssets": true,
    "webpack": true,
    "tsConfigPath": "tsconfig.build.json"
  },
  "generateOptions": {
    "spec": {
      "default": true,
      "grouping": "folders"
    }
  }
}
```

### package.json scripts

```json
{
  "name": "my-nestjs-api",
  "version": "1.0.0",
  "description": "NestJS REST API",
  "author": "",
  "private": true,
  "license": "MIT",
  "scripts": {
    "build": "nest build",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:e2e": "jest --config ./test/jest-e2e.json",
    "test:e2e:watch": "jest --config ./test/jest-e2e.json --watch",
    "prisma:generate": "prisma generate",
    "prisma:migrate": "prisma migrate dev",
    "prisma:push": "prisma db push",
    "prisma:studio": "prisma studio"
  },
  "dependencies": {
    "@nestjs/common": "^10.3.0",
    "@nestjs/core": "^10.3.0",
    "@nestjs/platform-express": "^10.3.0",
    "@nestjs/config": "^3.1.0",
    "@nestjs/typeorm": "^10.0.1",
    "@nestjs/passport": "^10.0.3",
    "@nestjs/jwt": "^10.2.0",
    "@nestjs/bull": "^10.0.1",
    "@nestjs/schedule": "^4.0.0",
    "@nestjs/swagger": "^7.2.0",
    "@nestjs/graphql": "^12.0.0",
    "@nestjs/apollo": "^12.0.0",
    "@nestjs/terminus": "^10.2.0",
    "typeorm": "^0.3.19",
    "pg": "^8.11.3",
    "passport": "^0.7.0",
    "passport-jwt": "^4.0.1",
    "passport-local": "^1.0.0",
    "bcryptjs": "^2.4.3",
    "class-validator": "^0.14.0",
    "class-transformer": "^0.5.1",
    "reflect-metadata": "^0.2.1",
    "rxjs": "^7.8.1",
    "ioredis": "^5.3.2",
    "bull": "^4.12.0",
    "@apollo/server": "^4.10.0",
    "graphql": "^16.8.1",
    "@graphql-tools/schema": "^10.0.0",
    "pino": "^8.17.0",
    "pino-pretty": "^10.3.0"
  },
  "devDependencies": {
    "@nestjs/cli": "^10.3.0",
    "@nestjs/schematics": "^10.1.0",
    "@nestjs/testing": "^10.3.0",
    "@types/express": "^4.17.21",
    "@types/node": "^20.10.0",
    "@types/bcryptjs": "^2.4.6",
    "@types/passport-jwt": "^4.0.0",
    "@types/passport-local": "^1.0.38",
    "typescript": "^5.3.2",
    "ts-node": "^10.9.2",
    "jest": "^29.7.0",
    "@types/jest": "^29.5.11",
    "ts-jest": "^29.1.1",
    "eslint": "^8.55.0",
    "prisma": "^6.0.0"
  }
}
```

---

## 数据库集成

### TypeORM 配置

```typescript
// src/config/database.config.ts
import { TypeOrmModuleOptions } from '@nestjs/typeorm';

export const databaseConfig = (): TypeOrmModuleOptions => ({
  type: 'postgres',
  host: process.env.DB_HOST || 'localhost',
  port: parseInt(process.env.DB_PORT, 10) || 5432,
  username: process.env.DB_USERNAME || 'postgres',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'mydb',
  entities: [__dirname + '/../**/*.entity{.ts,.js}'],
  migrations: [__dirname + '/../database/migrations/*{.ts,.js}'],
  subscribers: [__dirname + '/../**/*.subscriber{.ts,.js}'],
  synchronize: process.env.NODE_ENV !== 'production',
  logging: process.env.NODE_ENV === 'development',
  logger: 'advanced-console',
  ssl: process.env.NODE_ENV === 'production',
  extra: {
    ssl:
      process.env.NODE_ENV === 'production'
        ? { rejectUnauthorized: false }
        : undefined,
    connectionLimit: 10,
  },
  autoLoadEntities: true,
  keepConnectionAlive: true,
});
```

### Prisma 集成

```bash
npm install prisma @prisma/client
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

  posts    Post[]
  comments Comment[]

  @@index([email])
  @@index([username])
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

model Post {
  id          String    @id @default(uuid())
  title       String
  slug        String    @unique
  content     String
  excerpt     String?
  published   Boolean   @default(false)
  publishedAt DateTime?
  viewCount   Int       @default(0)
  authorId    String
  author      User      @relation(fields: [authorId], references: [id])
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  comments Comment[]

  @@index([slug])
  @@index([authorId])
  @@index([published])
}

model Comment {
  id        String   @id @default(uuid())
  content   String
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  postId    String
  post      Post     @relation(fields: [postId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([postId])
  @@index([authorId])
}
```

```typescript
// src/prisma/prisma.service.ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy
{
  constructor() {
    super({
      log:
        process.env.NODE_ENV === 'development'
          ? ['query', 'info', 'warn', 'error']
          : ['error'],
    });
  }

  async onModuleInit() {
    await this.$connect();
  }

  async onModuleDestroy() {
    await this.$disconnect();
  }
}
```

```typescript
// src/prisma/prisma.module.ts
import { Module, Global } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```

---

## 认证授权

### JWT 认证模块

```typescript
// src/modules/auth/dto/auth.dto.ts
import { IsEmail, IsString, MinLength, IsOptional, IsEnum } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class RegisterDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'username123' })
  @IsString()
  @MinLength(3)
  username: string;

  @ApiProperty({ example: 'Password123!' })
  @IsString()
  @MinLength(8)
  password: string;

  @ApiProperty({ required: false })
  @IsOptional()
  @IsString()
  firstName?: string;

  @ApiProperty({ required: false })
  @IsOptional()
  @IsString()
  lastName?: string;
}

export class LoginDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail()
  email: string;

  @ApiProperty({ example: 'Password123!' })
  @IsString()
  password: string;
}

export class RefreshTokenDto {
  @ApiProperty()
  @IsString()
  refreshToken: string;
}
```

```typescript
// src/modules/auth/strategies/jwt.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { ConfigService } from '@nestjs/config';
import { PrismaService } from '../../../prisma/prisma.service';

export interface JwtPayload {
  sub: string;
  email: string;
  role: string;
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private configService: ConfigService,
    private prisma: PrismaService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
    });
  }

  async validate(payload: JwtPayload) {
    const user = await this.prisma.user.findUnique({
      where: { id: payload.sub, isActive: true },
    });

    if (!user) {
      throw new UnauthorizedException('User not found or inactive');
    }

    return {
      id: user.id,
      email: user.email,
      role: user.role,
      username: user.username,
    };
  }
}
```

```typescript
// src/modules/auth/guards/jwt-auth.guard.ts
import { Injectable, ExecutionContext } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';
import { Reflector } from '@nestjs/core';
import { IS_PUBLIC_KEY } from '../../../common/decorators/public.decorator';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (isPublic) {
      return true;
    }

    return super.canActivate(context);
  }
}
```

```typescript
// src/modules/auth/guards/roles.guard.ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { ROLES_KEY } from '../../../common/decorators/roles.decorator';
import { Role } from '../../../prisma/client';

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>(ROLES_KEY, [
      context.getHandler(),
      context.getClass(),
    ]);

    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some((role) => user.role === role);
  }
}
```

```typescript
// src/modules/auth/auth.service.ts
import {
  Injectable,
  UnauthorizedException,
  ConflictException,
} from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import * as bcrypt from 'bcryptjs';
import { PrismaService } from '../../prisma/prisma.service';
import { RegisterDto, LoginDto } from './dto/auth.dto';

@Injectable()
export class AuthService {
  constructor(
    private prisma: PrismaService,
    private jwtService: JwtService,
  ) {}

  async register(dto: RegisterDto) {
    const existingUser = await this.prisma.user.findFirst({
      where: {
        OR: [{ email: dto.email }, { username: dto.username }],
      },
    });

    if (existingUser) {
      throw new ConflictException(
        existingUser.email === dto.email
          ? 'Email already registered'
          : 'Username already taken',
      );
    }

    const hashedPassword = await bcrypt.hash(dto.password, 12);

    const user = await this.prisma.user.create({
      data: {
        email: dto.email,
        username: dto.username,
        password: hashedPassword,
        firstName: dto.firstName,
        lastName: dto.lastName,
      },
    });

    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
        role: user.role,
      },
      ...tokens,
    };
  }

  async login(dto: LoginDto) {
    const user = await this.prisma.user.findUnique({
      where: { email: dto.email },
    });

    if (!user || !user.isActive) {
      throw new UnauthorizedException('Invalid credentials');
    }

    const isPasswordValid = await bcrypt.compare(dto.password, user.password);

    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    await this.prisma.user.update({
      where: { id: user.id },
      data: { lastLoginAt: new Date() },
    });

    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
        role: user.role,
      },
      ...tokens,
    };
  }

  async refreshToken(refreshToken: string) {
    try {
      const payload = this.jwtService.verify(refreshToken, {
        secret: process.env.JWT_REFRESH_SECRET,
      });

      if (payload.type !== 'refresh') {
        throw new UnauthorizedException('Invalid refresh token');
      }

      const user = await this.prisma.user.findUnique({
        where: { id: payload.sub, isActive: true },
      });

      if (!user) {
        throw new UnauthorizedException('User not found');
      }

      return this.generateTokens(user);
    } catch (error) {
      throw new UnauthorizedException('Invalid refresh token');
    }
  }

  private async generateTokens(user: any) {
    const payload = { sub: user.id, email: user.email, role: user.role };

    const accessToken = this.jwtService.sign(payload, {
      secret: process.env.JWT_SECRET,
      expiresIn: '15m',
    });

    const refreshToken = this.jwtService.sign(
      { ...payload, type: 'refresh' },
      {
        secret: process.env.JWT_REFRESH_SECRET,
        expiresIn: '7d',
      },
    );

    return { accessToken, refreshToken };
  }
}
```

```typescript
// src/modules/auth/auth.controller.ts
import {
  Controller,
  Post,
  Body,
  HttpCode,
  HttpStatus,
  Get,
  UseGuards,
} from '@nestjs/common';
import { ApiTags, ApiOperation, ApiResponse, ApiBearerAuth } from '@nestjs/swagger';
import { AuthService } from './auth.service';
import { RegisterDto, LoginDto, RefreshTokenDto } from './dto/auth.dto';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { CurrentUser } from '../../common/decorators/current-user.decorator';

@ApiTags('auth')
@Controller('auth')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  @ApiOperation({ summary: 'Register a new user' })
  @ApiResponse({ status: 201, description: 'User registered successfully' })
  @ApiResponse({ status: 409, description: 'User already exists' })
  async register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @Post('login')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Login user' })
  @ApiResponse({ status: 200, description: 'Login successful' })
  @ApiResponse({ status: 401, description: 'Invalid credentials' })
  async login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }

  @Post('refresh')
  @HttpCode(HttpStatus.OK)
  @ApiOperation({ summary: 'Refresh access token' })
  @ApiResponse({ status: 200, description: 'Token refreshed' })
  async refresh(@Body() dto: RefreshTokenDto) {
    return this.authService.refreshToken(dto.refreshToken);
  }

  @Get('me')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: 'Get current user' })
  @ApiResponse({ status: 200, description: 'Current user data' })
  async getProfile(@CurrentUser() user: any) {
    return user;
  }
}
```

```typescript
// src/modules/auth/auth.module.ts
import { Module } from '@nestjs/common';
import { JwtModule } from '@nestjs/jwt';
import { PassportModule } from '@nestjs/passport';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { AuthController } from './auth.controller';
import { AuthService } from './auth.service';
import { JwtStrategy } from './strategies/jwt.strategy';
import { JwtAuthGuard } from './guards/jwt-auth.guard';
import { RolesGuard } from './guards/roles.guard';
import { PrismaModule } from '../../prisma/prisma.module';

@Module({
  imports: [
    PassportModule.register({ defaultStrategy: 'jwt' }),
    JwtModule.registerAsync({
      imports: [ConfigModule],
      useFactory: async (configService: ConfigService) => ({
        secret: configService.get<string>('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
      inject: [ConfigService],
    }),
    PrismaModule,
  ],
  controllers: [AuthController],
  providers: [AuthService, JwtStrategy, JwtAuthGuard, RolesGuard],
  exports: [AuthService, JwtAuthGuard, RolesGuard],
})
export class AuthModule {}
```

---

## 常见陷阱与最佳实践

### 陷阱 1：循环依赖

```typescript
// ❌ 错误：UserModule 和 PostModule 循环依赖
// user.module.ts
@Module({
  imports: [PostModule], // 错误
})
export class UserModule {}

// post.module.ts
@Module({
  imports: [UserModule], // 错误
})
export class PostModule {}

// ✅ 正确：使用 forwardRef
@Module({
  imports: [forwardRef(() => PostModule)],
})
export class UserModule {}
```

### 陷阱 2：未正确处理异步

```typescript
// ❌ 错误：Controller 直接调用异步方法
@Post()
async create(@Body() dto: CreateDto) {
  return this.usersService.create(dto); // 忘记 await
}

// ✅ 正确
@Post()
async create(@Body() dto: CreateDto) {
  return await this.usersService.create(dto);
}
```

### 陷阱 3：未验证输入

```typescript
// ❌ 错误：没有使用 DTO 和 ValidationPipe
@Post()
async create(@Body() dto: any) { // 没有类型验证
  return this.usersService.create(dto);
}

// ✅ 正确
@Post()
async create(@Body() dto: CreateUserDto) {
  return await this.usersService.create(dto);
}

// 在 main.ts 中启用全局验证
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
    forbidNonWhitelisted: true,
  }),
);
```

### 最佳实践清单

1. **模块化设计**：每个功能模块独立，使用 `forwardRef` 避免循环依赖。
2. **DTO 验证**：使用 class-validator 和 class-transformer。
3. **依赖注入**：使用构造函数注入，优先使用 `inject` 装饰器。
4. **异常处理**：使用内置 HttpException 或自定义异常。
5. **异步处理**：始终使用 async/await。
6. **数据库连接**：在 OnModuleInit 中连接，OnModuleDestroy 中断开。
7. **配置管理**：使用 ConfigModule 和 @ConfigService。
8. **日志记录**：使用内置 Logger 或 Pino。
9. **测试覆盖**：使用 Jest 进行单元测试和 E2E 测试。
10. **文档生成**：使用 @nestjs/swagger 生成 OpenAPI 文档。

---

## 与其他框架对比

### NestJS vs Express

| 特性 | NestJS | Express |
|------|--------|---------|
| **架构** | 模块化，依赖注入 | 扁平化 |
| **学习曲线** | 较陡 | 平缓 |
| **TypeScript** | 原生支持 | 需要配置 |
| **装饰器** | 核心特性 | 可选 |
| **测试** | 完善的测试工具 | 需要配置 |
| **适用场景** | 企业级应用 | 轻量级 API |

### NestJS vs Fastify

| 特性 | NestJS | Fastify |
|------|--------|---------|
| **性能** | 高 | 极高 |
| **架构** | 模块化 | 扁平化 |
| **依赖注入** | 原生支持 | 插件实现 |
| **学习曲线** | 较陡 | 中等 |
| **适用场景** | 企业级应用 | 高性能 API |

### NestJS vs Spring Boot

| 特性 | NestJS | Spring Boot |
|------|--------|-------------|
| **语言** | TypeScript/JavaScript | Java/Kotlin |
| **依赖注入** | 装饰器 | 注解 |
| **ORM** | TypeORM/Prisma | JPA/Hibernate |
| **性能** | 高 | 高 |
| **生态** | 增长中 | 成熟庞大 |
| **适用场景** | Node.js 团队 | Java 团队 |

---

## 常见问题与解决方案

### 问题1：模块循环依赖

**问题描述**：NestJS 中两个模块相互导入导致循环依赖错误。

```typescript
// ❌ 错误示例
// cats.module.ts
@Module({
  imports: [DogsModule], // 错误：循环依赖
  controllers: [CatsController],
  providers: [CatsService]
})
export class CatsModule {}

// dogs.module.ts
@Module({
  imports: [CatsModule], // 错误：循环依赖
  controllers: [DogsController],
  providers: [DogsService]
})
export class DogsModule {}
```

**解决方案**：使用 `forwardRef` 或重构模块结构。

```typescript
// ✅ 解决方案1：使用 forwardRef
@Module({
  imports: [forwardRef(() => DogsModule)],
  controllers: [CatsController],
  providers: [CatsService]
})
export class CatsModule {}

// ✅ 解决方案2：提取共享模块
@Module({
  exports: [SharedService] // 只导出需要共享的服务
})
export class SharedModule {}

// ✅ 解决方案3：使用事件而非直接依赖
@Injectable()
class CatsService {
  constructor(private readonly eventEmitter: EventEmitter2) {}
  
  async createCat(name: string) {
    const cat = await this.catRepo.create({ name });
    this.eventEmitter.emit('cat.created', cat); // 发布事件而非直接调用
    return cat;
  }
}

@Injectable()
class DogsModule {
  constructor(private readonly eventEmitter: EventEmitter2) {
    this.eventEmitter.on('cat.created', (cat) => this.onCatCreated(cat));
  }
}
```

### 问题2：依赖注入失败

**问题描述**：`Nest can't resolve dependencies` 错误。

```typescript
// ❌ 错误：抽象类未正确注册
@Injectable()
class MyService {
  constructor(private readonly repo: Repository<User>) {} // Repository 是抽象类
}
```

**解决方案**：使用 `@Inject` 装饰器和自定义 provider。

```typescript
// ✅ 解决方案：定义 provider token
export const USER_REPOSITORY = 'USER_REPOSITORY';

@Injectable()
class MyService {
  constructor(
    @Inject(USER_REPOSITORY)
    private readonly repo: Repository<User>
  ) {}
}

// 在模块中注册
@Module({
  providers: [
    {
      provide: USER_REPOSITORY,
      useClass: TypeOrmRepository // 实际的实现类
    }
  ]
})
export class UsersModule {}
```

### 问题3：守卫不生效

**问题描述**：自定义守卫没有按预期执行。

```typescript
// ❌ 错误：守卫未在正确位置应用
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    // 守卫逻辑
    return true;
  }
}

@Controller('users')
@UseGuards(AuthGuard) // 守卫应用在控制器级别
export class UsersController {
  @Get('public') // 这个路由也被守卫保护了
  getPublic() {}
}
```

**解决方案**：使用 `@Public` 装饰器标记公开路由。

```typescript
// 定义 Public 装饰器
export const IS_PUBLIC_KEY = 'is_public';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

// 创建可以跳过守卫的 AuthGuard
@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) {
    super();
  }

  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      context.getHandler(),
      context.getClass()
    ]);
    
    if (isPublic) {
      return true;
    }
    
    return super.canActivate(context);
  }
}

// 使用
@Controller('users')
@UseGuards(JwtAuthGuard)
export class UsersController {
  @Public() // 这个路由不需要认证
  @Get('public')
  getPublic() {}
  
  @Get('protected') // 这个路由需要认证
  getProtected() {}
}
```

### 问题4：请求体验证失败

**问题描述**：`class-validator` 装饰器不生效。

```typescript
// ❌ 错误：未启用全局管道
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // 缺少 ValidationPipe
  await app.listen(3000);
}

// DTO 定义
export class CreateUserDto {
  @IsEmail()
  email: string;
  
  @IsString()
  @MinLength(8)
  password: string;
}
```

**解决方案**：启用全局 ValidationPipe。

```typescript
// ✅ 正确配置
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // 全局验证管道
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,           // 移除不在 DTO 中的属性
      forbidNonWhitelisted: true, // 拒绝额外属性
      transform: true,          // 自动类型转换
      transformOptions: {
        enableImplicitConversion: true
      }
    })
  );
  
  await app.listen(3000);
}
```

### 问题5：跨域配置问题

**问题描述**：浏览器报错 `CORS policy blocked`。

```typescript
// ❌ 错误：配置不完整
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.enableCors(); // 默认配置可能不够
  await app.listen(3000);
}
```

**解决方案**：配置完整的 CORS 选项。

```typescript
// ✅ 正确配置
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  app.enableCors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:3000'],
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
    exposedHeaders: ['Content-Range', 'X-Request-Id'],
    credentials: true,
    maxAge: 86400
  });
  
  await app.listen(3000);
}
```

### 问题6：数据库连接池耗尽

**问题描述**：高并发时出现 `connection timeout` 错误。

```typescript
// ❌ 错误：默认连接池配置
TypeOrmModule.forRoot({
  type: 'postgres',
  host: 'localhost',
  port: 5432,
  username: 'user',
  password: 'password',
  database: 'db'
  // 缺少连接池配置
})
```

**解决方案**：配置连接池参数。

```typescript
// ✅ 正确配置
TypeOrmModule.forRoot({
  type: 'postgres',
  host: process.env.DB_HOST,
  port: parseInt(process.env.DB_PORT, 10),
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  // 连接池配置
  extra: {
    max: 20,              // 最大连接数
    min: 5,               // 最小连接数
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 5000,
    acquireTimeoutMillis: 10000
  },
  // 连接池复用
  keepConnectionAlive: true,
  // 重连配置
  retryAttempts: 3,
  retryDelay: 3000
})
```

### 问题7：热重载不工作

**问题描述**：修改代码后应用不自动重启。

```typescript
// ❌ 错误：未正确配置 watch 模式
// nest-cli.json
{
  "$schema": "...",
  "sourceRoot": "src"
  // 缺少 watchOptions
}
```

**解决方案**：配置完整的 watch 选项。

```typescript
// ✅ 正确配置 nest-cli.json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true,
    "assets": ["**/*.graphql"],
    "watchOptions": {
      "watchman": {
        "lucene": true
      }
    },
    "webpack": false, // 使用 NestJS 原生编译而非 webpack
    "tsConfigPath": "tsconfig.build.json"
  }
}

// 或者使用 --watch 参数启动
// nest start --watch
// 或
// nest start --watch --preserveWatchOutput
```

### 问题8：GraphQL 解析错误

**问题描述**：GraphQL Playground 无法加载或查询报错。

```typescript
// ❌ 错误：缺少必要配置
GraphQLModule.forRoot({
  autoSchemaFile: true,
  playground: true
  // 缺少 subscriptions 配置
})
```

**解决方案**：配置完整的 GraphQL 选项。

```typescript
// ✅ 正确配置
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
  sortSchema: true,
  playground: process.env.NODE_ENV !== 'production',
  introspection: true,
  subscriptions: {
    'graphql-ws': true,
    'subscriptions-transport-ws': true
  },
  context: ({ req, connection }) => {
    if (connection) {
      return { req: connection.context };
    }
    return { req };
  },
  formatError: (error) => {
    // 生产环境隐藏详细错误
    if (process.env.NODE_ENV === 'production') {
      return new GraphQLError('Internal server error');
    }
    return error;
  }
})
```

### 问题9：JWT 令牌失效处理

**问题描述**：令牌过期时前端没有正确处理。

```typescript
// ❌ 错误：未处理 401 响应
@Injectable()
export class AuthService {
  constructor(private http: HttpClient) {}
  
  async getData() {
    return this.http.get('/api/data').toPromise(); // 未处理 401
  }
}
```

**解决方案**：实现令牌刷新机制。

```typescript
// ✅ 解决方案：使用拦截器处理令牌刷新
@Injectable()
export class TokenInterceptor implements HttpInterceptor {
  constructor(
    private authService: AuthService,
    private router: Router
  ) {}

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      catchError((error) => {
        if (error.status === 401 && !req.url.includes('/auth/refresh')) {
          return this.handle401Error(req, next);
        }
        return throwError(() => error);
      })
    );
  }

  private handle401Error(req: HttpRequest<any>, next: HttpHandler) {
    return this.authService.refreshToken().pipe(
      tap((tokens) => {
        this.authService.setTokens(tokens);
      }),
      switchMap(() => {
        req = req.clone({
          setHeaders: {
            Authorization: `Bearer ${this.authService.getAccessToken()}`
          }
        });
        return next.handle(req);
      }),
      catchError(() => {
        this.authService.logout();
        this.router.navigate(['/login']);
        return throwError(() => new Error('Token refresh failed'));
      })
    );
  }
}
```

### 问题10：内存泄漏排查

**问题描述**：应用运行一段时间后内存持续增长。

**排查步骤**：

```bash
# 1. 使用 Node.js 内存分析
node --inspect dist/main.js

# 2. 使用 heapdump
npm install heapdump
import * as heapdump from 'heapdump';

// 定期生成快照
setInterval(() => {
  heapdump.writeSnapshot('./' + Date.now() + '.heapsnapshot');
}, 60000); // 每分钟
```

**常见原因和解决方案**：

```typescript
// 原因1：事件监听器未清理
@Injectable()
export class MyService implements OnDestroy {
  private subscriptions: Subscription[] = [];
  
  ngOnInit() {
    // ❌ 错误：未保存订阅
    this.http.get('/api/data').subscribe(data => {});
    
    // ✅ 正确：保存订阅并在销毁时取消
    this.subscriptions.push(
      this.http.get('/api/data').subscribe(data => {})
    );
  }
  
  ngOnDestroy() {
    this.subscriptions.forEach(sub => sub.unsubscribe());
  }
}

// 原因2：定时器未清除
@Injectable()
export class MyService implements OnDestroy {
  private timer: NodeJS.Timeout;
  
  ngOnInit() {
    // ❌ 错误：定时器未清除
    this.timer = setInterval(() => this.doSomething(), 1000);
  }
  
  ngOnDestroy() {
    // ✅ 正确：清除定时器
    clearInterval(this.timer);
  }
}

// 原因3：缓存无限增长
@Injectable()
export class MyService {
  private cache = new Map<string, any>(); // 可能无限增长
  
  getData(key: string) {
    if (!this.cache.has(key)) {
      this.cache.set(key, this.fetchData(key));
    }
    return this.cache.get(key);
  }
}

// ✅ 解决方案：使用 LRU 缓存
import { LRUCache } from 'lru-cache';

@Injectable()
export class MyService {
  private cache = new LRUCache<string, any>({
    max: 500,           // 最大缓存条目数
    maxSize: 5000,      // 最大缓存大小（字节）
    ttl: 1000 * 60 * 5 // 5分钟过期
  });
  
  getData(key: string) {
    if (!this.cache.has(key)) {
      this.cache.set(key, this.fetchData(key));
    }
    return this.cache.get(key);
  }
}
```

---

## 实战项目示例

### 项目一：用户认证系统

**项目结构**：

```
src/
├── auth/
│   ├── auth.module.ts
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   ├── strategies/
│   │   ├── jwt.strategy.ts
│   │   └── local.strategy.ts
│   ├── guards/
│   │   ├── jwt-auth.guard.ts
│   │   └── roles.guard.ts
│   └── dto/
│       ├── login.dto.ts
│       └── register.dto.ts
├── users/
│   ├── users.module.ts
│   ├── users.service.ts
│   └── entities/
│       └── user.entity.ts
└── common/
    └── decorators/
        ├── current-user.decorator.ts
        ├── roles.decorator.ts
        └── public.decorator.ts
```

**完整实现**：

```typescript
// src/users/entities/user.entity.ts
import {
  Entity,
  Column,
  PrimaryGeneratedColumn,
  CreateDateColumn,
  UpdateDateColumn
} from 'typeorm';

export enum UserRole {
  ADMIN = 'admin',
  USER = 'user',
  MODERATOR = 'moderator'
}

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column({ unique: true })
  username: string;

  @Column()
  password: string;

  @Column({ nullable: true })
  firstName?: string;

  @Column({ nullable: true })
  lastName?: string;

  @Column({ nullable: true })
  avatar?: string;

  @Column({
    type: 'enum',
    enum: UserRole,
    default: UserRole.USER
  })
  role: UserRole;

  @Column({ default: true })
  isActive: boolean;

  @Column({ nullable: true })
  lastLoginAt?: Date;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

```typescript
// src/auth/dto/register.dto.ts
import { IsEmail, IsString, MinLength, MaxLength, Matches, IsOptional } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class RegisterDto {
  @ApiProperty({ example: 'user@example.com' })
  @IsEmail({}, { message: 'Please provide a valid email address' })
  email: string;

  @ApiProperty({ example: 'johndoe' })
  @IsString()
  @MinLength(3, { message: 'Username must be at least 3 characters' })
  @MaxLength(20, { message: 'Username cannot exceed 20 characters' })
  @Matches(/^[a-zA-Z0-9_]+$/, {
    message: 'Username can only contain letters, numbers, and underscores'
  })
  username: string;

  @ApiProperty({ example: 'Password123!' })
  @IsString()
  @MinLength(8, { message: 'Password must be at least 8 characters' })
  @MaxLength(50, { message: 'Password cannot exceed 50 characters' })
  @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, {
    message: 'Password must contain at least one uppercase letter, one lowercase letter, and one number'
  })
  password: string;

  @ApiPropertyOptional({ example: 'John' })
  @IsOptional()
  @IsString()
  @MaxLength(50)
  firstName?: string;

  @ApiPropertyOptional({ example: 'Doe' })
  @IsOptional()
  @IsString()
  @MaxLength(50)
  lastName?: string;
}
```

```typescript
// src/auth/auth.service.ts
import {
  Injectable,
  UnauthorizedException,
  ConflictException,
  BadRequestException
} from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';
import * as bcrypt from 'bcryptjs';
import { PrismaService } from '../../prisma/prisma.service';
import { RegisterDto, LoginDto } from './dto';
import { User, UserRole } from '../users/entities/user.entity';

export interface TokenPayload {
  sub: string;
  email: string;
  role: UserRole;
}

export interface AuthTokens {
  accessToken: string;
  refreshToken: string;
  expiresIn: number;
}

@Injectable()
export class AuthService {
  private readonly saltRounds = 12;

  constructor(
    private readonly prisma: PrismaService,
    private readonly jwtService: JwtService,
    private readonly configService: ConfigService
  ) {}

  async register(dto: RegisterDto): Promise<{
    user: Partial<User>;
    tokens: AuthTokens;
  }> {
    // 检查邮箱是否已存在
    const existingByEmail = await this.prisma.user.findUnique({
      where: { email: dto.email }
    });

    if (existingByEmail) {
      throw new ConflictException('Email already registered');
    }

    // 检查用户名是否已存在
    const existingByUsername = await this.prisma.user.findUnique({
      where: { username: dto.username }
    });

    if (existingByUsername) {
      throw new ConflictException('Username already taken');
    }

    // 密码哈希
    const hashedPassword = await bcrypt.hash(dto.password, this.saltRounds);

    // 创建用户
    const user = await this.prisma.user.create({
      data: {
        email: dto.email,
        username: dto.username,
        password: hashedPassword,
        firstName: dto.firstName,
        lastName: dto.lastName
      }
    });

    // 生成令牌
    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
        role: user.role
      },
      tokens
    };
  }

  async login(dto: LoginDto): Promise<{
    user: Partial<User>;
    tokens: AuthTokens;
  }> {
    const user = await this.prisma.user.findUnique({
      where: { email: dto.email }
    });

    if (!user) {
      throw new UnauthorizedException('Invalid credentials');
    }

    if (!user.isActive) {
      throw new UnauthorizedException('Account is disabled');
    }

    const isPasswordValid = await bcrypt.compare(dto.password, user.password);

    if (!isPasswordValid) {
      throw new UnauthorizedException('Invalid credentials');
    }

    // 更新最后登录时间
    await this.prisma.user.update({
      where: { id: user.id },
      data: { lastLoginAt: new Date() }
    });

    const tokens = await this.generateTokens(user);

    return {
      user: {
        id: user.id,
        email: user.email,
        username: user.username,
        role: user.role
      },
      tokens
    };
  }

  async refreshToken(refreshToken: string): Promise<AuthTokens> {
    try {
      const payload = this.jwtService.verify<TokenPayload & { type: string }>(
        refreshToken,
        {
          secret: this.configService.get<string>('JWT_REFRESH_SECRET')
        }
      );

      if (payload.type !== 'refresh') {
        throw new UnauthorizedException('Invalid refresh token');
      }

      const user = await this.prisma.user.findUnique({
        where: { id: payload.sub }
      });

      if (!user || !user.isActive) {
        throw new UnauthorizedException('User not found or disabled');
      }

      return this.generateTokens(user);
    } catch (error) {
      if (error instanceof UnauthorizedException) {
        throw error;
      }
      throw new UnauthorizedException('Invalid refresh token');
    }
  }

  async logout(userId: string): Promise<void> {
    // 在实际应用中，这里可能需要将刷新令牌加入黑名单
    // 可以使用 Redis 存储已撤销的令牌
    await this.prisma.user.update({
      where: { id: userId },
      data: { lastLoginAt: new Date() } // 记录登出时间
    });
  }

  async validateUser(userId: string): Promise<User> {
    const user = await this.prisma.user.findUnique({
      where: { id: userId }
    });

    if (!user || !user.isActive) {
      throw new UnauthorizedException('User not found or disabled');
    }

    return user;
  }

  private async generateTokens(user: User): Promise<AuthTokens> {
    const payload: TokenPayload = {
      sub: user.id,
      email: user.email,
      role: user.role
    };

    const accessToken = this.jwtService.sign(payload, {
      secret: this.configService.get<string>('JWT_SECRET'),
      expiresIn: '15m'
    });

    const refreshToken = this.jwtService.sign(
      { ...payload, type: 'refresh' },
      {
        secret: this.configService.get<string>('JWT_REFRESH_SECRET'),
        expiresIn: '7d'
      }
    );

    return {
      accessToken,
      refreshToken,
      expiresIn: 900 // 15 minutes in seconds
    };
  }
}
```

### 项目二：文件上传与处理系统

**功能特性**：

- 支持多种文件类型（图片、文档、音频、视频）
- 文件大小限制和类型验证
- 存储桶隔离和 RLS 策略
- 缩略图生成（图片）
- 文件元数据存储

```typescript
// src/storage/dto/upload-file.dto.ts
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';
import { Type } from 'class-transformer';
import { IsEnum, IsInt, IsOptional, IsString, Max, Min } from 'class-validator';

export enum FileCategory {
  IMAGE = 'image',
  DOCUMENT = 'document',
  AUDIO = 'audio',
  VIDEO = 'video',
  OTHER = 'other'
}

export class UploadFileDto {
  @ApiProperty({ enum: FileCategory, description: 'File category' })
  @IsEnum(FileCategory)
  category: FileCategory;

  @ApiPropertyOptional({ description: 'Optional folder path' })
  @IsOptional()
  @IsString()
  folder?: string;

  @ApiPropertyOptional({ description: 'Cache control in seconds' })
  @IsOptional()
  @Type(() => Number)
  @IsInt()
  @Min(0)
  @Max(31536000)
  cacheControl?: number;
}

export class FileResponseDto {
  @ApiProperty()
  id: string;

  @ApiProperty()
  name: string;

  @ApiProperty()
  url: string;

  @ApiProperty()
  size: number;

  @ApiProperty()
  mimeType: string;

  @ApiProperty()
  category: FileCategory;

  @ApiProperty()
  createdAt: Date;
}
```

```typescript
// src/storage/storage.service.ts
import {
  Injectable,
  BadRequestException,
  NotFoundException
} from '@nestjs/common';
import { SupabaseClient, StorageFileApi } from '@supabase/supabase-js';
import { ConfigService } from '@nestjs/config';
import { v4 as uuidv4 } from 'uuid';

@Injectable()
export class StorageService {
  private readonly bucket: string;
  private readonly maxFileSize: number;
  private readonly allowedMimeTypes: Record<FileCategory, string[]>;

  constructor(
    private readonly supabase: SupabaseClient,
    private readonly configService: ConfigService
  ) {
    this.bucket = this.configService.get<string>('STORAGE_BUCKET', 'files');
    this.maxFileSize = 10 * 1024 * 1024; // 10MB default
    
    this.allowedMimeTypes = {
      [FileCategory.IMAGE]: ['image/jpeg', 'image/png', 'image/gif', 'image/webp', 'image/svg+xml'],
      [FileCategory.DOCUMENT]: ['application/pdf', 'application/msword', 
        'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'application/vnd.ms-excel',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        'text/plain', 'text/markdown'
      ],
      [FileCategory.AUDIO]: ['audio/mpeg', 'audio/wav', 'audio/ogg', 'audio/mp4'],
      [FileCategory.VIDEO]: ['video/mp4', 'video/webm', 'video/quicktime'],
      [FileCategory.OTHER]: ['*/*']
    };
  }

  async upload(
    file: Buffer,
    fileName: string,
    options: {
      userId: string;
      category: FileCategory;
      folder?: string;
      mimeType?: string;
    }
  ): Promise<{
    id: string;
    name: string;
    url: string;
    size: number;
    mimeType: string;
    category: FileCategory;
  }> {
    const { userId, category, folder, mimeType } = options;

    // 验证文件大小
    if (file.length > this.maxFileSize) {
      throw new BadRequestException(
        `File size exceeds maximum allowed size of ${this.maxFileSize / 1024 / 1024}MB`
      );
    }

    // 验证文件类型
    if (!this.isAllowedType(mimeType || 'application/octet-stream', category)) {
      throw new BadRequestException(
        `File type ${mimeType} is not allowed for category ${category}`
      );
    }

    // 生成唯一文件名
    const ext = fileName.split('.').pop();
    const uniqueName = `${uuidv4()}.${ext}`;
    const path = this.buildPath(userId, category, folder, uniqueName);

    // 上传到 Supabase Storage
    const { data, error } = await this.supabase.storage
      .from(this.bucket)
      .upload(path, file, {
        cacheControl: options.cacheControl || 3600,
        contentType: mimeType,
        upsert: false
      });

    if (error) {
      throw new BadRequestException(`Failed to upload file: ${error.message}`);
    }

    // 获取公开 URL
    const { data: urlData } = this.supabase.storage
      .from(this.bucket)
      .getPublicUrl(path);

    return {
      id: uuidv4(),
      name: fileName,
      url: urlData.publicUrl,
      size: file.length,
      mimeType: mimeType || 'application/octet-stream',
      category
    };
  }

  async uploadMultiple(
    files: Array<{
      buffer: Buffer;
      fileName: string;
      mimeType?: string;
    }>,
    options: {
      userId: string;
      category: FileCategory;
      folder?: string;
    }
  ): Promise<Array<{
    id: string;
    name: string;
    url: string;
    size: number;
    mimeType: string;
    category: FileCategory;
    success: boolean;
    error?: string;
  }>> {
    const results = await Promise.allSettled(
      files.map(file =>
        this.upload(file.buffer, file.fileName, {
          userId: options.userId,
          category: options.category,
          folder: options.folder,
          mimeType: file.mimeType
        })
      )
    );

    return results.map((result, index) => {
      if (result.status === 'fulfilled') {
        return { ...result.value, success: true };
      } else {
        return {
          id: '',
          name: files[index].fileName,
          url: '',
          size: 0,
          mimeType: files[index].mimeType || '',
          category: options.category,
          success: false,
          error: result.reason.message
        };
      }
    });
  }

  async getFile(userId: string, filePath: string): Promise<Buffer> {
    const path = this.buildFilePath(userId, filePath);

    const { data, error } = await this.supabase.storage
      .from(this.bucket)
      .download(path);

    if (error || !data) {
      throw new NotFoundException('File not found');
    }

    return Buffer.from(await data.arrayBuffer());
  }

  async deleteFile(userId: string, filePath: string): Promise<void> {
    const path = this.buildFilePath(userId, filePath);

    const { error } = await this.supabase.storage
      .from(this.bucket)
      .remove([path]);

    if (error) {
      throw new BadRequestException(`Failed to delete file: ${error.message}`);
    }
  }

  async listFiles(
    userId: string,
    options: {
      category?: FileCategory;
      folder?: string;
      limit?: number;
      offset?: number;
    } = {}
  ): Promise<{
    files: Array<{
      name: string;
      url: string;
      size: number;
      mimeType: string;
      createdAt: Date;
    }>;
    total: number;
  }> {
    const folder = this.buildPath(userId, options.category, options.folder);

    const { data, error } = await this.supabase.storage
      .from(this.bucket)
      .list(folder, {
        limit: options.limit || 50,
        offset: options.offset || 0,
        sortBy: { column: 'created_at', order: 'desc' }
      });

    if (error) {
      throw new BadRequestException(`Failed to list files: ${error.message}`);
    }

    const files = await Promise.all(
      (data || []).map(async file => {
        const fullPath = `${folder}/${file.name}`;
        const { data: urlData } = this.supabase.storage
          .from(this.bucket)
          .getPublicUrl(fullPath);

        return {
          name: file.name,
          url: urlData.publicUrl,
          size: file.metadata?.size || 0,
          mimeType: file.metadata?.mimetype || 'application/octet-stream',
          createdAt: new Date(file.created_at || Date.now())
        };
      })
    );

    return {
      files,
      total: files.length
    };
  }

  private isAllowedType(mimeType: string, category: FileCategory): boolean {
    const allowed = this.allowedMimeTypes[category];
    return allowed.includes('*/*') || allowed.includes(mimeType);
  }

  private buildPath(
    userId: string,
    category?: FileCategory,
    folder?: string,
    fileName?: string
  ): string {
    const parts = [userId, category || 'other'];
    if (folder) parts.push(folder);
    if (fileName) parts.push(fileName);
    return parts.join('/');
  }

  private buildFilePath(userId: string, filePath: string): string {
    return `${userId}/${filePath}`;
  }
}
```

---

> [!SUCCESS]
> NestJS 以 Angular 风格的模块化架构为 Node.js 后端开发带来了企业级的工程实践。依赖注入、AOP、装饰器模式等特性使得代码组织清晰、易于测试。对于需要构建可扩展、可维护后端服务的团队，NestJS 是一个值得认真考虑的选择。

---

## 附录：NestJS 常用配置模板

### 开发环境配置

```typescript
// .env.development
NODE_ENV=development
PORT=3000
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=myapp_dev
DATABASE_USER=postgres
DATABASE_PASSWORD=dev_password
JWT_SECRET=dev_jwt_secret_change_in_production
JWT_EXPIRES_IN=1d
REDIS_HOST=localhost
REDIS_PORT=6379
```

```typescript
// .env.production
NODE_ENV=production
PORT=8080
DATABASE_HOST=prod-db.example.com
DATABASE_PORT=5432
DATABASE_NAME=myapp_prod
DATABASE_USER=prod_user
DATABASE_PASSWORD=<from_secrets_manager>
JWT_SECRET=<from_secrets_manager>
JWT_EXPIRES_IN=15m
REDIS_HOST=prod-redis.example.com
REDIS_PORT=6379
```

### Docker 配置示例

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:20-alpine AS runner

WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package*.json ./

RUN addgroup -g 1001 -S nodejs && adduser -S nestjs -u 1001
USER nestjs

EXPOSE 8080
CMD ["node", "dist/main"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
      - DATABASE_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
    restart: unless-stopped

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"

volumes:
  postgres_data:
  redis_data:
```

### CI/CD 配置示例

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run lint
      - run: npm run test:cov
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: |
            myapp:${{ github.sha }}
            myapp:latest
```

### 健康检查配置

```typescript
// health.controller.ts
import { Controller, Get } from '@nestjs/common';
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from '@nestjs/terminus';

@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator,
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck('database'),
      
      () => this.memory.checkRSS(
        'memory_rss',
        300 * 1024 * 1024, // 300MB
      ),
      
      () => this.disk.checkStorage(
        'disk',
        { thresholdPercent: 0.9, path: '/' },
      ),
    ]);
  }
}
```

### 性能监控配置

```typescript
// metrics.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { Counter, Histogram, Gauge, Registry, collectDefaultMetrics } from 'prom-client';

@Injectable()
export class MetricsService implements OnModuleInit {
  private readonly registry: Registry;
  private readonly httpRequestsTotal: Counter;
  private readonly httpRequestDuration: Histogram;
  private readonly activeConnections: Gauge;

  constructor() {
    this.registry = new Registry();
    
    this.httpRequestsTotal = new Counter({
      name: 'http_requests_total',
      help: 'Total number of HTTP requests',
      labelNames: ['method', 'path', 'status'],
      registers: [this.registry],
    });

    this.httpRequestDuration = new Histogram({
      name: 'http_request_duration_seconds',
      help: 'Duration of HTTP requests in seconds',
      labelNames: ['method', 'path'],
      buckets: [0.01, 0.05, 0.1, 0.5, 1, 5],
      registers: [this.registry],
    });

    this.activeConnections = new Gauge({
      name: 'active_connections',
      help: 'Number of active connections',
      registers: [this.registry],
    });
  }

  onModuleInit() {
    collectDefaultMetrics({ register: this.registry });
  }

  incrementRequest(method: string, path: string, status: number) {
    this.httpRequestsTotal.inc({ method, path, status });
  }

  observeRequestDuration(method: string, path: string, duration: number) {
    this.httpRequestDuration.observe({ method, path }, duration);
  }

  async getMetrics(): Promise<string> {
    return this.registry.metrics();
  }
}
```
