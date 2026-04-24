---
title: NestJS 完全指南
date: 2026-04-24
tags:
  - 后端框架
  - Node.js
  - NestJS
  - TypeScript
  - 企业级
description: NestJS 是一个受 Angular 启发的渐进式 Node.js 框架，提供模块化架构、依赖注入、AOP 等企业级特性，本指南涵盖其架构、核心模块、认证授权及与 Spring Boot 的对比。
---

# NestJS 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 NestJS 架构、依赖注入、核心模块、Guards/Interceptors/Pipes、GraphQL 集成及在 AI 应用中的定位。

---

## NestJS 概述：Angular 的灵魂，Node.js 的躯壳

NestJS 是由 Kamil Myśliwiec 于 2017 年创建的一个 Node.js 后端框架，它的设计灵感直接来源于 Angular——如果你曾经使用过 Angular，你会发现 NestJS 的模块、依赖注入、装饰器和分层架构都与 Angular 如出一辙。这种设计选择并非偶然：Angular 团队在构建大型前端应用时积累的工程化经验，被成功地移植到了后端领域。

NestJS 的官方定位是「**一个用于构建高效、可扩展的 Node.js 服务器端应用的渐进式框架**」。这里的「渐进式」有三个层面的含义：首先是**学习曲线的渐进性**——你可以从最简单的单文件应用开始，逐步引入模块、依赖注入等概念；其次是**功能的渐进性**——从简单的 HTTP API 到复杂的微服务、GraphQL、WebSocket，NestJS 都有内置支持；第三是**架构的渐进性**——随着项目规模增长，NestJS 的模块化架构让你可以优雅地扩展代码库，而不会出现难以维护的「意大利面条式」代码。

### 为什么选择 NestJS

很多开发者在选择后端框架时会问：Express 和 Fastify 已经足够用了，为什么还需要 NestJS？这个问题触及了框架选择的核心矛盾——**灵活性 vs 结构性的权衡**。

想象一下，你接手了一个有 50 个路由、20 个中间件、同时操作 3 个数据库的中型 Express 项目。现在产品经理要求你新增一个「审计日志」功能——记录每个用户的操作历史。在 Express 中，你可能需要在每个路由处理器中添加审计逻辑，或者写一个中间件但需要处理各种边界情况。这个功能做起来很痛苦，维护起来更痛苦，因为没有人能说清楚审计逻辑的完整边界在哪里。

NestJS 通过「**切面编程（AOP）**」的思路解决了这个问题。审计日志不需要分散在各个路由处理器中，而是可以通过一个「**拦截器（Interceptor）**」统一处理。这个拦截器可以被精细地应用到特定的路由上，而不影响其他路由。依赖注入则让你可以像拼积木一样组合服务，每个服务的依赖由框架自动管理，你不需要手动处理初始化顺序。

```typescript
// 审计日志拦截器：只需要写一次，到处使用
@Injectable()
export class AuditInterceptor implements NestInterceptor {
  constructor(private auditService: AuditService) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const userId = request.user?.id;

    return next.handle().pipe(
      tap((response) => {
        // 请求成功时记录审计日志
        this.auditService.log({
          userId,
          action: method,
          resource: url,
          status: 'success',
          timestamp: new Date(),
        });
      }),
      catchError((error) => {
        // 请求失败时也记录
        this.auditService.log({
          userId,
          action: method,
          resource: url,
          status: 'error',
          error: error.message,
          timestamp: new Date(),
        });
        throw error;
      }),
    );
  }
}

// 使用拦截器：只需要一行装饰器
@Controller('users')
@UseInterceptors(AuditInterceptor)
export class UsersController {
  // 这里的每个方法都会被审计
  @Get()
  findAll() { /* ... */ }

  @Post()
  create(@Body() dto: CreateUserDto) { /* ... */ }

  @Delete(':id')
  remove(@Param('id') id: string) { /* ... */ }
}
```

### NestJS 的适用边界

NestJS 并不是万能的。理解它的适用边界，才能做出正确的技术选型。

NestJS **适合**的场景包括：大型企业级应用、团队成员有 Angular 背景、需要强类型安全和完整的代码结构、构建微服务架构、需要 GraphQL 和 REST 并存的项目、需要大量测试覆盖的长期维护项目。

NestJS **不适合**的场景包括：简单的脚本或工具类、需要极致性能的超高并发 API（此时 Fastify 更合适）、需要部署到边缘计算环境（此时 Hono 更合适）、快速原型和 MVP 开发（Express 更快）、Serverless 冷启动敏感的场景。

> [!TIP]
> **选型小技巧**：如果你需要在 Express 的灵活性和 NestJS 的结构性之间做选择，问自己一个问题：「我的项目在 2 年后会是什么规模？」如果答案是「更大」，NestJS 的前期投入是值得的。如果答案是「保持小」，用 Express 会更轻便。

---

## 核心架构：Module、Controller、Provider

NestJS 的整个应用都是围绕「**模块化**」这个核心理念构建的。一个 NestJS 应用由多个模块组成，每个模块封装了相关的功能，而模块之间通过导入导出来建立依赖关系。

### Module（模块）：代码组织的基本单元

模块是 NestJS 中组织代码的最高层概念。每个应用至少有一个「根模块」（通常是 `AppModule`），但在实际项目中，你应该根据功能领域来划分模块。

```typescript
// users.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';
import { User } from './entities/user.entity';
import { AuthModule } from '../auth/auth.module';

@Module({
  // 导入其他模块（获取它们的导出）
  imports: [
    TypeOrmModule.forFeature([User]), // 在当前模块中使用 User 实体
    AuthModule, // 导入认证模块以使用其导出的服务
  ],
  // 控制器（处理 HTTP 请求）
  controllers: [UsersController],
  // 服务提供者（业务逻辑）
  providers: [UsersService],
  // 导出（让其他模块可以使用）
  exports: [UsersService],
})
export class UsersModule {}
```

模块的设计原则是「**高内聚、低耦合**」。一个好的模块应该：专注于一个功能领域、拥有清晰的公共 API（通过 exports）、不过度依赖其他模块。一个常见的问题是「上帝模块」——把所有东西都塞进一个巨大的 AppModule。这在小型项目中可能无所谓，但随着项目增长，这种做法会严重影响代码的可维护性。

### Controller（控制器）：请求的入口

控制器负责处理 HTTP 请求和响应，它不包含业务逻辑——那是 Provider（服务）的职责。控制器的工作是：接收请求、验证参数、调用服务、返回响应。

```typescript
// users.controller.ts
import {
  Controller,
  Get,
  Post,
  Put,
  Patch,
  Delete,
  Body,
  Param,
  Query,
  Headers,
  HttpCode,
  HttpStatus,
  UseGuards,
  UseInterceptors,
  ParseIntPipe,
  DefaultValuePipe,
} from '@nestjs/common';
import { UsersService } from './users.service';
import { CreateUserDto, UpdateUserDto } from './dto';
import { PaginationQueryDto } from '../common/dto';
import { JwtAuthGuard } from '../auth/guards/jwt-auth.guard';
import { RolesGuard } from '../auth/guards/roles.guard';
import { AuditInterceptor } from '../common/interceptors/audit.interceptor';
import { SetMetadata } from '@nestjs/common';

// 定义角色元数据的辅助函数
const ROLES_KEY = 'roles';
const Roles = (...roles: string[]) => SetMetadata(ROLES_KEY, roles);

@Controller('users')
@UseInterceptors(AuditInterceptor) // 审计日志
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // GET /users - 获取用户列表
  @Get()
  @UseGuards(JwtAuthGuard) // JWT 认证
  async findAll(@Query() query: PaginationQueryDto) {
    return this.usersService.findAll(query);
  }

  // GET /users/:id - 获取单个用户
  @Get(':id')
  @UseGuards(JwtAuthGuard)
  async findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);
  }

  // POST /users - 创建用户
  @Post()
  @HttpCode(HttpStatus.CREATED)
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin') // 只有管理员可以创建用户
  async create(@Body() createUserDto: CreateUserDto) {
    return this.usersService.create(createUserDto);
  }

  // PUT /users/:id - 完整更新
  @Put(':id')
  @UseGuards(JwtAuthGuard)
  async replace(
    @Param('id') id: string,
    @Body() createUserDto: CreateUserDto,
  ) {
    return this.usersService.replace(id, createUserDto);
  }

  // PATCH /users/:id - 部分更新
  @Patch(':id')
  @UseGuards(JwtAuthGuard)
  async update(
    @Param('id') id: string,
    @Body() updateUserDto: UpdateUserDto,
  ) {
    return this.usersService.update(id, updateUserDto);
  }

  // DELETE /users/:id - 删除用户
  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  @UseGuards(JwtAuthGuard, RolesGuard)
  @Roles('admin')
  async remove(@Param('id') id: string) {
    return this.usersService.remove(id);
  }
}
```

### Provider（服务）：业务逻辑的载体

服务是 NestJS 中「**依赖注入**」的主要载体。在 Angular 的术语中，Provider 也被称为「Service」，但在 NestJS 中「Service」只是 Provider 的一种形式。

```typescript
// users.service.ts
import { Injectable, NotFoundException, ConflictException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import * as bcrypt from 'bcryptjs';
import { User } from './entities/user.entity';
import { CreateUserDto, UpdateUserDto, PaginationQueryDto } from './dto';

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,
  ) {}

  async findAll(query: PaginationQueryDto) {
    const { page = 1, limit = 10, search, sort = 'createdAt', order = 'DESC' } = query;
    const skip = (page - 1) * limit;

    const queryBuilder = this.userRepository.createQueryBuilder('user');

    if (search) {
      queryBuilder.where(
        '(user.email ILIKE :search OR user.username ILIKE :search)',
        { search: `%${search}%` },
      );
    }

    queryBuilder
      .orderBy(`user.${sort}`, order)
      .skip(skip)
      .take(limit);

    const [users, total] = await queryBuilder.getManyAndCount();

    return {
      data: users.map(user => this.sanitizeUser(user)),
      meta: {
        total,
        page,
        limit,
        totalPages: Math.ceil(total / limit),
      },
    };
  }

  async findOne(id: string): Promise<User> {
    const user = await this.userRepository.findOne({ where: { id } });

    if (!user) {
      throw new NotFoundException(`用户 ${id} 不存在`);
    }

    return user;
  }

  async findByEmail(email: string): Promise<User | null> {
    return this.userRepository.findOne({ where: { email } });
  }

  async create(dto: CreateUserDto): Promise<User> {
    // 检查邮箱唯一性
    const existing = await this.userRepository.findOne({
      where: [{ email: dto.email }, { username: dto.username }],
    });

    if (existing) {
      const field = existing.email === dto.email ? '邮箱' : '用户名';
      throw new ConflictException(`${field}已被注册`);
    }

    // 密码哈希
    const hashedPassword = await bcrypt.hash(dto.password, 12);

    const user = this.userRepository.create({
      ...dto,
      password: hashedPassword,
    });

    return this.userRepository.save(user);
  }

  async update(id: string, dto: UpdateUserDto): Promise<User> {
    const user = await this.findOne(id);

    if (dto.email && dto.email !== user.email) {
      const existing = await this.userRepository.findOne({ where: { email: dto.email } });
      if (existing) {
        throw new ConflictException('该邮箱已被使用');
      }
    }

    Object.assign(user, dto);
    return this.userRepository.save(user);
  }

  async remove(id: string): Promise<void> {
    const user = await this.findOne(id);
    await this.userRepository.remove(user);
  }

  // 移除敏感字段
  private sanitizeUser(user: User) {
    const { password, ...sanitized } = user;
    return sanitized;
  }
}
```

---

## 依赖注入容器：NestJS 的心脏

依赖注入（Dependency Injection，简称 DI）是 NestJS 最核心的概念。简单来说，依赖注入就是：不是让对象自己创建它所需要的依赖，而是让外部（框架）把依赖「注入」进来。

### 依赖注入的基本原理

要理解依赖注入，我们先来看一个没有依赖注入的代码会是什么样：

```typescript
// 没有 DI 的代码：Service 自己创建依赖
class UsersService {
  private repository: UserRepository;
  private emailService: EmailService;
  private logger: Logger;

  constructor() {
    // 手动创建依赖实例
    this.repository = new UserRepository();
    this.emailService = new EmailService();
    this.logger = new Logger('UsersService');
  }
}
```

这种写法的问题在于：测试时无法 mock 依赖、依赖的初始化顺序需要手动管理、代码耦合度高。

使用依赖注入后：

```typescript
// 有 DI 的代码：依赖由框架注入
@Injectable()
class UsersService {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly emailService: EmailService,
    private readonly logger: Logger,
  ) {}
}
```

你不需要关心这些依赖是如何创建和初始化的，NestJS 的 IoC（控制反转）容器会处理这一切。

### Provider 的多种形式

Provider 不仅仅指服务类，NestJS 支持多种形式的 Provider：

```typescript
@Module({
  providers: [
    // 形式 1：类 Provider（最常用）
    UsersService,

    // 形式 2：值 Provider（注入常量或配置）
    {
      provide: 'CONFIG',
      useValue: {
        apiVersion: 'v1',
        maxUsers: 10000,
      },
    },

    // 形式 3：工厂 Provider（动态创建，可注入其他依赖）
    {
      provide: 'DATABASE_POOL',
      inject: [ConfigService], // 可以注入其他 Provider
      useFactory: async (configService: ConfigService) => {
        const size = configService.get('DB_POOL_SIZE', 10);
        return createPool({ size });
      },
    },

    // 形式 4：类 Provider（使用不同的类实现同一个接口）
    {
      provide: 'CACHE_STRATEGY',
      useClass: process.env.NODE_ENV === 'production'
        ? RedisCacheService
        : InMemoryCacheService,
    },

    // 形式 5：别名 Provider
    {
      provide: 'USER_SERVICE',
      useExisting: UsersService,
    },
  ],
})
export class UsersModule {}
```

### 模块间的依赖管理

模块之间通过导入导出建立依赖关系：

```typescript
// auth.module.ts - 定义和导出 AuthService
@Module({
  providers: [AuthService, JwtService],
  exports: [AuthService, JwtService], // 导出后其他模块可以使用
})
export class AuthModule {}

// users.module.ts - 导入并使用 AuthModule
@Module({
  imports: [AuthModule], // 导入 AuthModule
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}

// 在 UsersController 中使用 AuthService
@Controller('users')
export class UsersController {
  constructor(private readonly authService: AuthService) {
    // AuthService 从 AuthModule 注入进来
  }
}
```

### 全局模块

有些服务（如日志、配置）是几乎所有模块都需要的，如果每个模块都要显式导入会非常繁琐。这时可以使用全局模块：

```typescript
// config.module.ts
@Global()
@Module({
  providers: [ConfigService],
  exports: [ConfigService],
})
export class ConfigModule {}

// app.module.ts
@Module({
  imports: [ConfigModule], // 只需导入一次
  modules: [UsersModule, PostsModule, CommentsModule], // 所有子模块都能访问 ConfigService
})
export class AppModule {}
```

> [!WARNING]
> **全局模块的陷阱**：过度使用全局模块会导致隐式依赖，使代码变得难以追踪。如果一个服务只在少数几个地方用到，不要设为全局的。好的做法是：只有真正「全局」的服务（如日志、配置）才设为全局模块。

---

## Guards、Interceptors、Pipes 与 Decorators

NestJS 提供了四类核心工具来分离横切关注点（Cross-cutting Concerns）：

### Guards（守卫）：权限控制

守卫用于控制对特定路由的访问权限。它们实现了 `CanActivate` 接口，在请求到达处理器之前执行。

```typescript
// jwt-auth.guard.ts - JWT 认证守卫
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private readonly jwtService: JwtService,
    private readonly reflector: Reflector,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractToken(request);

    if (!token) {
      throw new UnauthorizedException('请先登录');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = payload;
    } catch {
      throw new UnauthorizedException('登录已过期，请重新登录');
    }

    return true;
  }

  private extractToken(request: any): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }
}

// roles.guard.ts - 角色授权守卫
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    // 从处理器或控制器获取所需角色
    const requiredRoles = this.reflector.getAllAndOverride<string[]>(
      ROLES_KEY,
      [context.getHandler(), context.getClass()],
    );

    // 没有角色要求，直接通过
    if (!requiredRoles) {
      return true;
    }

    const { user } = context.switchToHttp().getRequest();

    // 检查用户是否拥有所需角色之一
    return requiredRoles.some(role => user?.roles?.includes(role));
  }
}

// 使用守卫
@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard)
export class PostsController {
  @Get()
  findAll() { /* 所有登录用户都可以访问 */ }

  @Delete(':id')
  @Roles('admin') // 只有管理员可以删除
  remove(@Param('id') id: string) { /* ... */ }
}
```

### Interceptors（拦截器）：请求/响应转换

拦截器提供了在请求-响应生命周期中「包裹」逻辑的能力，非常适合用于日志、缓存、响应转换等功能。

```typescript
// logging.interceptor.ts - 日志拦截器
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  constructor(private readonly logger: Logger) {}

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const request = context.switchToHttp().getRequest();
    const { method, url } = request;
    const now = Date.now();

    this.logger.log(`➜ ${method} ${url} - started`);

    return next.handle().pipe(
      tap(() => {
        const response = context.switchToHttp().getResponse();
        const duration = Date.now() - now;
        this.logger.log(`➜ ${method} ${url} - ${response.statusCode} - ${duration}ms`);
      }),
      catchError((error) => {
        const duration = Date.now() - now;
        this.logger.error(`✗ ${method} ${url} - ${error.status} - ${duration}ms`, error.stack);
        throw error;
      }),
    );
  }
}

// transform.interceptor.ts - 响应转换拦截器
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, Response<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<Response<T>> {
    return next.handle().pipe(
      map((data) => ({
        success: true,
        data,
        timestamp: new Date().toISOString(),
      })),
    );
  }
}

// cache.interceptor.ts - 缓存拦截器
@Injectable()
export class CacheInterceptor implements NestInterceptor {
  constructor(
    private readonly reflector: Reflector,
    private readonly cacheService: CacheService,
  ) {}

  async intercept(context: ExecutionContext, next: CallHandler): Promise<Observable<any>> {
    const cacheTTL = this.reflector.get('cacheTTL', context.getHandler());
    const request = context.switchToHttp().getRequest();
    const cacheKey = `cache:${request.method}:${request.url}`;

    // 缓存命中
    const cached = await this.cacheService.get(cacheKey);
    if (cached && cacheTTL) {
      return of(cached);
    }

    // 缓存未命中，继续处理
    return next.handle().pipe(
      tap((data) => {
        if (cacheTTL) {
          this.cacheService.set(cacheKey, data, cacheTTL);
        }
      }),
    );
  }
}

// 使用缓存拦截器
@Controller('posts')
export class PostsController {
  @Get()
  @UseInterceptors(CacheInterceptor)
  @SetMetadata('cacheTTL', 60000) // 缓存 60 秒
  findAll() { /* ... */ }
}
```

### Pipes（管道）：数据转换与验证

管道用于数据转换和验证。最常用的内置管道包括 `ParseIntPipe`、`ParseUUIDPipe`、`ValidationPipe` 等。

```typescript
// 自定义管道：解析枚举
@Injectable()
export class ParseEnumPipe<T> implements PipeTransform<string, T> {
  constructor(
    private readonly enumType: T,
    private readonly errorMessage?: string,
  ) {}

  transform(value: string, metadata: ArgumentMetadata): T {
    const enumValues = Object.values(this.enumType);
    const enumKey = enumValues.find(v => v === value || v === +value);

    if (enumKey === undefined) {
      throw new BadRequestException(
        this.errorMessage || `无效的枚举值: ${value}`,
      );
    }

    return enumKey as T;
  }
}

// 自定义管道：模糊搜索转换
@Injectable()
export class SearchPipe implements PipeTransform<string, string[]> {
  transform(value: string, metadata: ArgumentMetadata): string[] {
    if (!value) {
      return [];
    }
    return value.split(',').map(v => v.trim().toLowerCase());
  }
}

// ValidationPipe 的高级配置
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,           // 移除不在 DTO 中的属性
    forbidNonWhitelisted: true, // 拒绝额外属性
    transform: true,            // 自动类型转换
    transformOptions: {
      enableImplicitConversion: true, // 允许隐式类型转换
    },
    errorHttpStatusCode: HttpStatus.UNPROCESSABLE_ENTITY, // 422 错误码
    stopAtFirstError: false,   // 显示所有错误而非只显示第一个
  }),
);

// 使用管道
@Controller('posts')
export class PostsController {
  @Get(':id')
  async findOne(
    @Param('id', ParseUUIDPipe) id: string, // 必须是有效 UUID
  ) { /* ... */ }

  @Get()
  async findAll(
    @Query('page', new DefaultValuePipe(1), ParseIntPipe) page: number,
    @Query('status', new DefaultValuePipe('published'), 
            new ParseEnumPipe(PostStatus)) status: PostStatus,
    @Query('tags', SearchPipe) tags: string[],
  ) { /* ... */ }
}
```

---

## TypeORM 和 Prisma 集成

### TypeORM 集成

TypeORM 是 NestJS 生态中最成熟的 ORM 之一，NestJS 官方提供了 `@nestjs/typeorm` 包来简化集成。

```typescript
// app.module.ts - TypeORM 配置
@Module({
  imports: [
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        type: 'postgres',
        host: config.get('DB_HOST'),
        port: config.get('DB_PORT'),
        username: config.get('DB_USERNAME'),
        password: config.get('DB_PASSWORD'),
        database: config.get('DB_NAME'),
        entities: [__dirname + '/**/*.entity{.ts,.js}'],
        synchronize: config.get('NODE_ENV') !== 'production', // 生产环境禁用
        logging: config.get('NODE_ENV') === 'development',
        poolSize: 20,
        ssl: config.get('NODE_ENV') === 'production',
      }),
    }),
  ],
})
export class AppModule {}

// entities/user.entity.ts - 实体定义
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

  @Column({ type: 'enum', enum: UserRole, default: UserRole.USER })
  role: UserRole;

  @Column({ default: true })
  isActive: boolean;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;

  @OneToMany(() => Post, (post) => post.author)
  posts: Post[];

  @OneToMany(() => Comment, (comment) => comment.author)
  comments: Comment[];
}
```

### Prisma 集成

Prisma 是近年来崛起的新一代 ORM，以类型安全和直观的 API著称。

```typescript
// prisma.service.ts - Prisma 服务
@Injectable()
export class PrismaService
  extends PrismaClient
  implements OnModuleInit, OnModuleDestroy {
  constructor() {
    super({
      log: process.env.NODE_ENV === 'development'
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

// prisma.module.ts - Prisma 模块
@Global()
@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}

// 在服务中使用 Prisma
@Injectable()
export class UsersService {
  constructor(private readonly prisma: PrismaService) {}

  async findAll(query: PaginationQueryDto) {
    const { page = 1, limit = 20, search } = query;

    const where = search ? {
      OR: [
        { email: { contains: search, mode: 'insensitive' } },
        { username: { contains: search, mode: 'insensitive' } },
      ],
    } : {};

    const [users, total] = await this.prisma.$transaction([
      this.prisma.user.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: 'desc' },
        select: {
          id: true,
          email: true,
          username: true,
          role: true,
          createdAt: true,
        },
      }),
      this.prisma.user.count({ where }),
    ]);

    return {
      data: users,
      meta: { page, limit, total, totalPages: Math.ceil(total / limit) },
    };
  }

  async create(dto: CreateUserDto) {
    const existing = await this.prisma.user.findFirst({
      where: { OR: [{ email: dto.email }, { username: dto.username }] },
    });

    if (existing) {
      const field = existing.email === dto.email ? '邮箱' : '用户名';
      throw new ConflictException(`${field}已被注册`);
    }

    return this.prisma.user.create({
      data: {
        ...dto,
        password: await bcrypt.hash(dto.password, 12),
      },
      select: {
        id: true,
        email: true,
        username: true,
        role: true,
        createdAt: true,
      },
    });
  }
}
```

---

## 认证授权：JWT 与 Passport

### JWT 认证实现

JWT（JSON Web Token）是现代 Web 应用中最常用的无状态认证方案。NestJS 通过 `@nestjs/jwt` 和 `@nestjs/passport` 提供了完整的认证支持。

```typescript
// auth.service.ts - 认证服务
@Injectable()
export class AuthService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly jwtService: JwtService,
    private readonly configService: ConfigService,
  ) {}

  async register(dto: RegisterDto) {
    const existing = await this.prisma.user.findFirst({
      where: { OR: [{ email: dto.email }, { username: dto.username }] },
    });

    if (existing) {
      throw new ConflictException('用户已存在');
    }

    const hashedPassword = await bcrypt.hash(dto.password, 12);

    const user = await this.prisma.user.create({
      data: { ...dto, password: hashedPassword },
      select: { id: true, email: true, username: true, role: true },
    });

    const tokens = await this.generateTokens(user);

    return { user, ...tokens };
  }

  async login(dto: LoginDto) {
    const user = await this.prisma.user.findUnique({
      where: { email: dto.email },
    });

    if (!user || !user.isActive) {
      throw new UnauthorizedException('用户名或密码错误');
    }

    const isPasswordValid = await bcrypt.compare(dto.password, user.password);

    if (!isPasswordValid) {
      throw new UnauthorizedException('用户名或密码错误');
    }

    const tokens = await this.generateTokens({
      id: user.id,
      email: user.email,
      role: user.role,
    });

    return { user, ...tokens };
  }

  async refreshToken(refreshToken: string) {
    try {
      const payload = this.jwtService.verify(refreshToken, {
        secret: this.configService.get('JWT_REFRESH_SECRET'),
      });

      if (payload.type !== 'refresh') {
        throw new UnauthorizedException('无效的刷新令牌');
      }

      const user = await this.prisma.user.findUnique({
        where: { id: payload.sub },
      });

      if (!user || !user.isActive) {
        throw new UnauthorizedException('用户不存在或已被禁用');
      }

      return this.generateTokens({
        id: user.id,
        email: user.email,
        role: user.role,
      });
    } catch {
      throw new UnauthorizedException('令牌已过期');
    }
  }

  private async generateTokens(payload: { id: string; email: string; role: string }) {
    const [accessToken, refreshToken] = await Promise.all([
      this.jwtService.signAsync(payload, {
        secret: this.configService.get('JWT_SECRET'),
        expiresIn: '15m',
      }),
      this.jwtService.signAsync(
        { ...payload, type: 'refresh' },
        {
          secret: this.configService.get('JWT_REFRESH_SECRET'),
          expiresIn: '7d',
        },
      ),
    ]);

    return { accessToken, refreshToken };
  }
}
```

```typescript
// auth.controller.ts - 认证控制器
@Controller('auth')
@ApiTags('认证')
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post('register')
  @HttpCode(HttpStatus.CREATED)
  @Public() // 不需要认证
  @ApiOperation({ summary: '用户注册' })
  async register(@Body() dto: RegisterDto) {
    return this.authService.register(dto);
  }

  @Post('login')
  @HttpCode(HttpStatus.OK)
  @Public()
  @ApiOperation({ summary: '用户登录' })
  async login(@Body() dto: LoginDto) {
    return this.authService.login(dto);
  }

  @Post('refresh')
  @HttpCode(HttpStatus.OK)
  @Public()
  @ApiOperation({ summary: '刷新令牌' })
  async refresh(@Body() dto: RefreshTokenDto) {
    return this.authService.refreshToken(dto.refreshToken);
  }

  @Get('me')
  @UseGuards(JwtAuthGuard)
  @ApiBearerAuth()
  @ApiOperation({ summary: '获取当前用户' })
  async getProfile(@CurrentUser() user: any) {
    return user;
  }
}
```

---

## 微服务与消息队列

NestJS 内置了强大的微服务支持，可以轻松构建分布式系统。

### Redis 消息队列

```typescript
// main.ts - 启动微服务
async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.connectMicroservice({
    transport: Transport.REDIS,
    options: {
      host: 'localhost',
      port: 6379,
    },
  });

  await app.startAllMicroservices();
  await app.listen(3000);
}

// users.service.ts - 事件发射
@Injectable()
export class UsersService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async create(dto: CreateUserDto) {
    const user = await this.prisma.user.create({ data: dto });

    // 发射领域事件
    this.eventEmitter.emit('user.created', {
      userId: user.id,
      email: user.email,
      timestamp: new Date(),
    });

    return user;
  }
}

// events.listener.ts - 事件监听
@Injectable()
export class UserEventListener {
  constructor(
    private readonly emailService: EmailService,
    private readonly auditService: AuditService,
  ) {}

  @OnEvent('user.created')
  async handleUserCreated(payload: { userId: string; email: string }) {
    // 发送欢迎邮件
    await this.emailService.sendWelcomeEmail(payload.email);

    // 记录审计日志
    await this.auditService.log({
      action: 'user.created',
      resourceId: payload.userId,
    });
  }

  @OnEvent('user.deleted')
  async handleUserDeleted(payload: { userId: string }) {
    // 清理相关数据
    await this.cleanupUserData(payload.userId);
  }
}
```

### RabbitMQ 消息队列

```typescript
// 微服务连接
app.connectMicroservice({
  transport: Transport.RMQ,
  options: {
    urls: ['amqp://localhost:5672'],
    queue: 'notifications_queue',
    queueOptions: { durable: true },
  },
});

// 消息处理器
@Controller()
export class NotificationsController {
  @MessagePattern('notification.email')
  async handleEmailNotification(@Payload() data: EmailPayload) {
    await this.emailService.send(data);
    return { success: true };
  }

  @MessagePattern('notification.push')
  async handlePushNotification(@Payload() data: PushPayload) {
    await this.pushService.send(data);
    return { success: true };
  }
}
```

---

## GraphQL 集成

NestJS 提供了完整的 GraphQL 支持，包括代码优先和 schema 优先两种开发方式。

```typescript
// app.module.ts - GraphQL 配置
@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: join(process.cwd(), 'src/schema.gql'),
      sortSchema: true,
      playground: process.env.NODE_ENV !== 'production',
      introspection: true,
      subscriptions: {
        'graphql-ws': true,
      },
      context: ({ req, connection }) => {
        if (connection) {
          return { req: connection.context };
        }
        return { req };
      },
    }),
  ],
})
export class AppModule {}

// users.resolver.ts - GraphQL Resolver
@Resolver(() => User)
export class UsersResolver {
  constructor(private readonly usersService: UsersService) {}

  @Query(() => [User], { name: 'users' })
  async findAll(
    @Args('page', { type: () => Int, defaultValue: 1 }) page: number,
    @Args('limit', { type: () => Int, defaultValue: 20 }) limit: number,
  ) {
    return this.usersService.findAll({ page, limit });
  }

  @Query(() => User, { name: 'user', nullable: true })
  async findOne(@Args('id', { type: () => ID }) id: string) {
    return this.usersService.findOne(id);
  }

  @Mutation(() => User)
  async createUser(@Args('input') input: CreateUserInput) {
    return this.usersService.create(input);
  }

  @Mutation(() => User)
  async updateUser(
    @Args('id', { type: () => ID }) id: string,
    @Args('input') input: UpdateUserInput,
  ) {
    return this.usersService.update(id, input);
  }

  @Subscription(() => User)
  userCreated() {
    return this.pubSub.asyncIterator('userCreated');
  }
}
```

---

## NestJS vs Spring Boot：Java 开发者的视角

很多从 Java 阵营转来的开发者会拿 NestJS 和 Spring Boot 做对比，因为两者在很多设计理念上确实非常相似。

| 维度 | NestJS | Spring Boot |
|------|--------|-------------|
| **语言** | TypeScript/JavaScript | Java/Kotlin |
| **依赖注入** | 装饰器 + TypeScript 反射 | 注解 + Java 反射 |
| **ORM** | TypeORM/Prisma | JPA/Hibernate/MyBatis |
| **配置** | ConfigModule + .env | @ConfigurationProperties |
| **验证** | class-validator | Jakarta Validation |
| **安全** | Guards + Passport | Spring Security |
| **API 文档** | @nestjs/swagger | SpringDoc OpenAPI |
| **测试** | Jest | JUnit + Mockito |
| **生态** | Node.js 生态 | Java 生态 |

如果你有 Spring Boot 背景，学习 NestJS 会非常轻松——核心概念几乎是一一对应的：

- `@Injectable()` 类比 Spring 的 `@Service` 或 `@Component`
- `@Controller()` 类比 Spring 的 `@RestController`
- `@Module()` 类比 Spring 的 `@Configuration`
- Guards 类比 Spring Security 的 `HandlerInterceptor`
- Interceptors 类比 Spring 的 `HandlerInterceptor` 和 AOP 切面
- Pipes 类比 Spring 的 `HandlerMethodArgumentResolver`

> [!TIP]
> **从 Java 迁移的建议**：NestJS 的好处是可以用同样的工程化思维来构建 Node.js 应用，但需要注意的是 Java 和 JavaScript/TypeScript 的运行时特性不同。Java 是强类型、预编译语言，而 TypeScript 最终编译为 JavaScript 运行。某些在 Java 中习以为常的做法（如深度反射）在 TypeScript 中可能会遇到性能问题。

---

## 选型建议：何时使用 NestJS

### 适合使用 NestJS 的场景

**第一，团队有 Angular 背景。** 如果你的前端团队使用 Angular，他们会对 NestJS 的模块、依赖注入和装饰器非常熟悉，可以快速上手。

**第二，需要构建企业级应用。** NestJS 的模块化架构、依赖注入和 AOP 特性非常适合构建需要长期维护的中大型项目。

**第三，需要 GraphQL 和 REST 并存。** NestJS 同时支持 RESTful API 和 GraphQL，可以满足复杂的 API 需求。

**第四，需要微服务架构。** NestJS 内置了微服务支持，可以轻松构建分布式系统。

**第五，团队有 Java 背景。** 如果你的团队熟悉 Spring Boot 的开发模式，NestJS 可以说是 Node.js 版本的 Spring Boot。

### 不适合使用 NestJS 的场景

**第一，简单脚本或工具。** 对于一次性脚本、CLI 工具等，不需要 NestJS 的复杂性。

**第二，极致性能场景。** 如果你的应用需要处理超高并发（>100K QPS），Fastify 可能是更好的选择。

**第三，边缘计算。** NestJS 目前无法运行在 Cloudflare Workers 等边缘环境中，这种场景下应该选择 Hono。

**第四，快速原型。** 对于 MVP 和原型开发，Express 的灵活性更适合快速迭代。

---

## 常见陷阱与最佳实践

### 陷阱一：上帝模块

```typescript
// ❌ 错误：所有东西都塞进 AppModule
@Module({
  imports: [TypeOrmModule, AuthModule, UsersModule, PostsModule, CommentsModule, TagsModule, ...],
  controllers: [UsersController, PostsController, CommentsController, TagsController, ...],
  providers: [UsersService, PostsService, CommentsService, TagsService, ...],
})
export class AppModule {}

// ✅ 正确：按功能领域划分模块
@Module({
  imports: [UsersModule, PostsModule, CommentsModule],
  controllers: [HealthController],
  providers: [AppService],
})
export class AppModule {}
```

### 陷阱二：循环依赖

```typescript
// ❌ 错误：两个模块相互导入造成循环依赖
@Module({ imports: [PostsModule] })
export class UsersModule {}

@Module({ imports: [UsersModule] })
export class PostsModule {}

// ✅ 正确：使用 shared 模块或事件通信
// 方案 1：提取共享模块
@Module({ exports: [UsersService] })
export class UsersModule {}

@Module({ imports: [UsersModule] })
export class PostsModule {}

// 方案 2：使用事件（事件驱动的解耦）
@Injectable()
class PostsService {
  constructor(private readonly eventEmitter: EventEmitter2) {}

  async createPost(dto) {
    const post = await this.postRepository.create(dto);
    this.eventEmitter.emit('post.created', { postId: post.id });
    return post;
  }
}
```

### 陷阱三：全局守卫滥用

```typescript
// ❌ 错误：所有路由都应用了认证守卫
app.useGlobalGuards(new JwtAuthGuard());

// ✅ 正确：按需应用守卫
@Module({
  providers: [
    {
      provide: APP_GUARD,
      useClass: JwtAuthGuard,
    },
  ],
})
export class AuthModule {}

// 然后在需要认证的路由上应用守卫
@Controller('posts')
@UseGuards(JwtAuthGuard)
export class PostsController {}
```

### 最佳实践清单

1. **模块划分要清晰**：每个模块应该专注于一个功能领域，拥有明确的边界。

2. **使用 DTO 进行数据验证**：不要在服务中直接使用请求体，使用 class-validator 的 DTO。

3. **合理使用依赖注入**：服务之间的依赖通过构造函数注入，不要直接创建实例。

4. **使用全局异常过滤器**：统一处理错误响应格式。

5. **实现健康检查**：使用 `@nestjs/terminus` 提供健康检查端点。

6. **编写测试**：利用 NestJS 的测试工具为每个服务编写单元测试。

7. **使用 Swagger 文档**：使用 `@nestjs/swagger` 生成 API 文档。

8. **配置分离**：开发、测试、生产环境使用不同的配置。

9. **合理使用 `forwardRef`**：只有确实存在循环依赖时才使用 forwardRef，并尽快重构消除循环依赖。

10. **性能监控**：集成 Prometheus 或其他监控工具来追踪性能指标。

> [!SUCCESS]
> NestJS 以 Angular 风格的模块化架构为 Node.js 后端开发带来了企业级的工程实践。依赖注入、AOP、装饰器模式等特性使得代码组织清晰、易于测试。对于需要构建可扩展、可维护后端服务的团队，NestJS 是一个值得认真考虑的选择——特别是当你的团队有 Angular 或 Spring Boot 背景时。
