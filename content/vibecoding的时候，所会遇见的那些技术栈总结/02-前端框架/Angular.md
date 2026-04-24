# Angular - 企业级 TypeScript 框架

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Angular 19 新特性、模块系统、依赖注入、RxJS 与实战技巧。

---

## 目录

1. [[#Angular 概述与版本演进]]
2. [[#Angular 19 新特性]]
3. [[#模块系统 vs Standalone Components]]
4. [[#依赖注入 (DI)]]
5. [[#RxJS 响应式编程]]
6. [[#Angular CLI]]
7. [[#Angular Material]]
8. [[#SSR 支持]]
9. [[#与 React/Vue 对比]]
10. [[#实战技巧]]
11. [[#参考资料]]

---

## Angular 概述与版本演进

### 什么是 Angular

Angular 是 Google 维护的 TypeScript 原生前端框架，于 2016 年 Angular 2 发布时进行了完全重写。与 AngularJS（Angular 1.x）不同，Angular 是基于 TypeScript 的现代框架，强调**企业级应用开发**和**工程化**。

Angular 的核心理念是" batteries included"（一站式解决方案），提供了从路由、表单、HTTP 客户端到依赖注入、响应式编程的完整工具链。

### 版本演进表

| 版本 | 发布年份 | 重大特性 |
|------|---------|---------|
| AngularJS | 2010 | 初始版本，双向绑定 |
| **Angular 2** | 2016 | 完全重写，TypeScript，组件化 |
| Angular 4 | 2017 | 更小更快，动画包 |
| Angular 5 | 2017 | 构建优化器，HttpClient |
| Angular 6 | 2018 | Angular Elements，CLI 工作区 |
| Angular 7 | 2018 | 虚拟滚动，拖放，CLI 提示 |
| Angular 8 | 2019 | 差异化加载，Ivy 预览 |
| Angular 9 | 2020 | **Ivy 渲染引擎默认开启** |
| Angular 10 | 2020 | TypeScript 3.9，更严格的检查 |
| Angular 11 | 2020 | 延迟加载改进，内联字体 |
| Angular 12 | 2021 | 无样式组件，Webpack 5 |
| Angular 13 | 2021 | 移除 View Engine |
| Angular 14 | 2022 | **独立组件（Standalone Components）** |
| Angular 15 | 2022 | 独立组件稳定，Router 平滑升级 |
| Angular 16 | 2023 | **Signals 响应式系统** |
| Angular 17 | 2023 | **SSR 改进，内置控制流** |
| Angular 18 | 2024 | 改进的 SSR，更好的 Signals |
| **Angular 19** | 2024 | **完整的 Signals 支持，改进的 zoneless** |

### 核心特点

| 特点 | 说明 |
|------|------|
| **TypeScript 原生** | 从设计之初就支持 TypeScript |
| **企业级** | 适合大型复杂应用 |
| **完整解决方案** | 路由、表单、HTTP、DI 全家桶 |
| **依赖注入** | 强大的依赖注入系统 |
| **RxJS** | 内置响应式编程 |
| **严格检查** | TypeScript 严格模式 |
| **Google 维护** | 长期稳定支持 |

---

## Angular 19 新特性

### 1. 完整的 Signals 支持

Signals 是 Angular 16 引入的响应式原语，Angular 19 实现了完整支持：

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <p>计数: {{ count() }}</p>
    <p>翻倍: {{ doubled() }}</p>
    <button (click)="increment()">增加</button>
  `
})
export class CounterComponent {
  // Signal 定义（可响应值）
  count = signal(0);
  
  // 计算属性
  doubled = computed(() => this.count() * 2);
  
  constructor() {
    // Effect - 响应式副作用
    effect(() => {
      console.log('count 变化了:', this.count());
    });
  }
  
  increment() {
    // Signal 更新
    this.count.update(c => c + 1);
  }
}
```

### 2. Zoneless 变更检测

Angular 19 改进了 zoneless 支持，减少包体积：

```typescript
import { bootstrapApplication } from '@angular/platform-browser';
import { provideExperimentalZonelessChangeDetection } from '@angular/core';

bootstrapApplication(AppComponent, {
  providers: [
    provideExperimentalZonelessChangeDetection()
  ]
});
```

### 3. 改进的内置控制流

```typescript
// @if, @for, @switch (Angular 17+)
@Component({
  template: `
    @if (user) {
      <p>欢迎, {{ user.name }}</p>
    } @else {
      <p>请登录</p>
    }
    
    @for (item of items; track item.id) {
      <li>{{ item.name }}</li>
    }
    
    @switch (status) {
      @case ('active') { <span>活跃</span> }
      @case ('inactive') { <span>不活跃</span> }
      @default { <span>未知</span> }
    }
  `
})
```

### 4. 延迟加载视图

```typescript
@Component({
  template: `
    <nav>...</nav>
    
    @defer (on interaction; prefetch on idle) {
      <heavy-component />
    } @loading (minimum 200ms) {
      <skeleton-loader />
    } @placeholder {
      <placeholder-content />
    }
  `
})
```

---

## 模块系统 vs Standalone Components

### 传统模块系统 (NgModule)

```typescript
// user.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';

import { UserListComponent } from './user-list.component';
import { UserDetailComponent } from './user-detail.component';
import { UserService } from './user.service';

@NgModule({
  declarations: [
    UserListComponent,
    UserDetailComponent
  ],
  imports: [
    CommonModule,
    RouterModule.forChild([
      { path: '', component: UserListComponent },
      { path: ':id', component: UserDetailComponent }
    ]),
    FormsModule
  ],
  providers: [UserService],
  exports: [UserListComponent] // 可导出供其他模块使用
})
export class UserModule {}
```

### Standalone Components（Angular 14+）

```typescript
// user-list.component.ts
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';
import { FormsModule } from '@angular/forms';

import { UserService } from './user.service';
import { UserCardComponent } from './user-card.component';

@Component({
  selector: 'app-user-list',
  standalone: true, // 标记为独立组件
  imports: [
    CommonModule,
    RouterModule, // 直接导入需要的模块
    FormsModule,
    UserCardComponent
  ],
  template: `
    <div class="user-list">
      @for (user of users; track user.id) {
        <app-user-card [user]="user" (edit)="onEdit($event)" />
      }
    </div>
  `
})
export class UserListComponent {
  private userService = inject(UserService);
  
  users = this.userService.getUsers();
  
  onEdit(id: string) {
    console.log('编辑用户:', id);
  }
}
```

### 两种方式对比

| 维度 | NgModule | Standalone |
|------|----------|------------|
| **学习曲线** | 较陡 | 较平缓 |
| **代码量** | 较多 | 较少 |
| **可复用性** | 模块级别 | 组件级别 |
| **懒加载** | 模块级别 | 路由级别 |
| **推荐度** | 传统项目 | **新项目推荐** |
| **混合使用** | - | 支持 |

> [!TIP]
> 新项目推荐使用 Standalone Components，可以减少大量样板代码。除非有特殊需求（如动态模块），否则没有必要使用 NgModule。

### 路由配置对比

```typescript
// 传统方式 - 使用模块
const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./user/user.module').then(m => m.UserModule)
  }
];

// Standalone 方式 - 直接路由
const routes: Routes = [
  {
    path: 'users',
    loadComponent: () => import('./user/user-list.component').then(m => m.UserListComponent)
  }
];
```

---

## 依赖注入 (DI)

### 什么是依赖注入

依赖注入是一种设计模式，允许组件通过构造函数或注入器获取依赖，而不是自己创建。Angular 拥有强大的 DI 系统。

### 基本用法

```typescript
// user.service.ts
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root' // 全局单例
})
export class UserService {
  private users: User[] = [];
  
  getUsers(): User[] {
    return this.users;
  }
  
  getUserById(id: string): User | undefined {
    return this.users.find(u => u.id === id);
  }
  
  addUser(user: Omit<User, 'id'>): User {
    const newUser = { ...user, id: crypto.randomUUID() };
    this.users.push(newUser);
    return newUser;
  }
}
```

```typescript
// component.ts
import { Component, inject } from '@angular/core';
import { UserService } from './user.service';

@Component({
  selector: 'app-user-list',
  template: `<div>{{ userService.getUsers().length }} users</div>`
})
export class UserListComponent {
  // 注入服务（推荐方式）
  userService = inject(UserService);
}
```

### 注入令牌 (Injection Tokens)

```typescript
import { Injectable, Inject, InjectionToken } from '@angular/core';

// 定义配置接口
export interface AppConfig {
  apiUrl: string;
  theme: string;
}

// 创建 Injection Token
export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

// 在 provider 中使用
@Injectable()
export class AppComponent {
  constructor(@Inject(APP_CONFIG) private config: AppConfig) {
    console.log(config.apiUrl);
  }
}

// 注册 provider
bootstrapApplication(AppComponent, {
  providers: [
    { provide: APP_CONFIG, useValue: { apiUrl: '/api', theme: 'dark' } }
  ]
});
```

### Provider 类型

```typescript
// 多种 provider 方式

// 1. useClass - 类实例
{ provide: LoggerService, useClass: BetterLoggerService }

// 2. useValue - 任意值
{ provide: 'API_URL', useValue: 'https://api.example.com' }

// 3. useFactory - 工厂函数
{ 
  provide: UserService, 
  useFactory: (http: HttpClient, config: AppConfig) => {
    return new UserService(http, config.apiUrl);
  },
  deps: [HttpClient, APP_CONFIG]
}

// 4. useExisting - 别名
{ provide: NewLoggerService, useExisting: LoggerService }
```

### 层级注入

| 层级 | 说明 | providedIn |
|------|------|------------|
| **root** | 应用级单例 | `providedIn: 'root'` |
| **platform** | 同一页面多个 Angular 应用 | `providedIn: 'platform'` |
| **any** | 每个模块新实例 | `providedIn: 'any'` |
| **Component** | 组件级 | `providers: [...]` |

```typescript
@Component({
  selector: 'app-child',
  providers: [
    // 组件级别的服务实例
    { provide: CounterService, useClass: CounterService }
  ]
})
export class ChildComponent {
  counter = inject(CounterService);
}
```

---

## RxJS 响应式编程

### RxJS 核心概念

| 概念 | 说明 |
|------|------|
| **Observable** | 可观察对象，数据流 |
| **Observer** | 观察者，订阅 Observable |
| **Subscription** | 订阅关系 |
| **Operators** | 操作符，转换数据流 |
| **Subject** | 特殊 Observable，可多播 |

### 常用操作符表

| 操作符 | 类别 | 说明 |
|-------|------|------|
| **map** | 转换 | 转换每个发出的值 |
| **filter** | 过滤 | 过滤符合条件的值 |
| **switchMap** | 组合 | 取消前一个订阅，切换到新订阅 |
| **mergeMap** | 组合 | 保持多个订阅同时进行 |
| **concatMap** | 组合 | 等待前一个完成后执行下一个 |
| **tap** | 工具 | 执行副作用，不改变流 |
| **debounceTime** | 时间 | 防抖 |
| **distinctUntilChanged** | 过滤 | 过滤重复值 |
| **catchError** | 错误 | 错误处理 |
| **retry** | 错误 | 重试 |
| **take** | 过滤 | 取前 n 个值 |
| **combineLatest** | 组合 | 组合多个流 |
| **forkJoin** | 组合 | 并行执行，等待所有完成 |

### HTTP 请求示例

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, catchError, of, map } from 'rxjs';

interface User {
  id: string;
  name: string;
  email: string;
}

@Component({
  selector: 'app-user-list',
  template: `
    @if (loading) {
      <p>加载中...</p>
    }
    
    @for (user of users$ | async; track user.id) {
      <p>{{ user.name }}</p>
    }
    
    @if (error) {
      <p class="error">{{ error }}</p>
    }
  `
})
export class UserListComponent {
  private http = inject(HttpClient);
  
  loading = true;
  error = '';
  
  // 使用 async pipe
  users$: Observable<User[]> = this.http.get<User[]>('/api/users').pipe(
    map(users => users.map(u => ({ ...u, name: u.name.toUpperCase() }))),
    catchError(error => {
      this.error = '加载失败';
      return of([]);
    })
  );
  
  // 手动订阅
  constructor() {
    this.users$.subscribe({
      next: () => this.loading = false,
      error: () => this.loading = false
    });
  }
}
```

### Subject 用法

```typescript
import { Component, inject } from '@angular/core';
import { Subject, BehaviorSubject, ReplaySubject, AsyncSubject } from 'rxjs';

// Subject - 基础多播
const subject = new Subject<number>();
subject.subscribe(x => console.log('A:', x));
subject.subscribe(x => console.log('B:', x));
subject.next(1); // A: 1, B: 1
subject.next(2); // A: 2, B: 2

// BehaviorSubject - 有初始值，记住最后一个值
const behaviorSubject = new BehaviorSubject<string>('initial');
behaviorSubject.subscribe(x => console.log('订阅者:', x));
behaviorSubject.next('新值'); // 订阅者: 新值

// ReplaySubject - 记住最后 N 个值
const replaySubject = new ReplaySubject<number>(2);
replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);
replaySubject.subscribe(x => console.log('Replay:', x)); // Replay: 2, 3

// AsyncSubject - 只发送最后一个值（complete 时）
const asyncSubject = new AsyncSubject<number>();
asyncSubject.next(1);
asyncSubject.next(2);
asyncSubject.subscribe(x => console.log('Async:', x));
asyncSubject.complete(); // Async: 2
```

### 搜索防抖示例

```typescript
import { Component, inject, OnInit, OnDestroy } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subject, debounceTime, distinctUntilChanged, switchMap, takeUntil } from 'rxjs';

@Component({
  selector: 'app-search',
  template: `
    <input 
      [value]="searchTerm" 
      (input)="onSearch($event)"
      placeholder="搜索..."
    />
    <div class="results">
      @for (result of results; track result.id) {
        <p>{{ result.name }}</p>
      }
    </div>
  `
})
export class SearchComponent implements OnInit, OnDestroy {
  private http = inject(HttpClient);
  private searchSubject = new Subject<string>();
  private destroy$ = new Subject<void>();
  
  searchTerm = '';
  results: any[] = [];
  
  ngOnInit() {
    // 防抖 + 去重 + 切换搜索
    this.searchSubject.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => this.http.get(`/api/search?q=${term}`)),
      takeUntil(this.destroy$) // 组件销毁时自动取消订阅
    ).subscribe(results => {
      this.results = results;
    });
  }
  
  onSearch(event: Event) {
    const input = event.target as HTMLInputElement;
    this.searchTerm = input.value;
    this.searchSubject.next(input.value);
  }
  
  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## Angular CLI

### 常用命令

```bash
# 创建新项目
ng new my-app --style=scss --routing=true --strict

# 启动开发服务器
ng serve
ng serve --port 4200 --open

# 构建生产版本
ng build --configuration=production

# 运行测试
ng test
ng test --watch=false --code-coverage

# 生成组件/服务/模块
ng generate component components/user-card
ng generate service services/user
ng generate module modules/admin
ng generate guard guards/auth
ng generate directive directives/highlight

# 简写
ng g c components/user-card
ng g s services/user
ng g m modules/admin
```

### angular.json 配置

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "version": 1,
  "newProjectRoot": "projects",
  "projects": {
    "my-app": {
      "projectType": "application",
      "root": "",
      "sourceRoot": "src",
      "architect": {
        "build": {
          "options": {
            "outputPath": "dist/my-app",
            "index": "src/index.html",
            "main": "src/main.ts",
            "polyfills": ["zone.js"],
            "tsConfig": "tsconfig.app.json",
            "assets": ["src/favicon.ico", "src/assets"],
            "styles": ["src/styles.scss"],
            "scripts": []
          },
          "configurations": {
            "production": {
              "budgets": [
                {
                  "type": "initial",
                  "maximumWarning": "500kb",
                  "maximumError": "1mb"
                },
                {
                  "type": "anyComponentStyle",
                  "maximumWarning": "2kb",
                  "maximumError": "4kb"
                }
              ],
              "outputHashing": "all"
            }
          }
        }
      }
    }
  }
}
```

---

## Angular Material

### 快速开始

```bash
ng add @angular/material
```

### 常用组件示例

```typescript
import { Component } from '@angular/core';
import { MatButtonModule } from '@angular/material/button';
import { MatInputModule } from '@angular/material/input';
import { MatCardModule } from '@angular/material/card';
import { MatFormFieldModule } from '@angular/material/form-field';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-material-demo',
  standalone: true,
  imports: [
    MatButtonModule,
    MatInputModule,
    MatCardModule,
    MatFormFieldModule,
    FormsModule
  ],
  template: `
    <mat-card>
      <mat-card-header>
        <mat-card-title>Angular Material</mat-card-title>
      </mat-card-header>
      
      <mat-card-content>
        <mat-form-field appearance="outline">
          <mat-label>邮箱</mat-label>
          <input matInput [(ngModel)]="email" type="email">
        </mat-form-field>
        
        <mat-form-field appearance="outline">
          <mat-label>密码</mat-label>
          <input matInput [(ngModel)]="password" type="password">
        </mat-form-field>
      </mat-card-content>
      
      <mat-card-actions>
        <button mat-raised-button color="primary" (click)="login()">
          登录
        </button>
        <button mat-button>注册</button>
      </mat-card-actions>
    </mat-card>
  `
})
export class MaterialDemoComponent {
  email = '';
  password = '';
  
  login() {
    console.log('登录:', this.email);
  }
}
```

### 常用组件列表

| 组件 | 用途 | 模块 |
|------|------|------|
| MatButton | 按钮 | MatButtonModule |
| MatInput | 输入框 | MatInputModule |
| MatCard | 卡片 | MatCardModule |
| MatTable | 表格 | MatTableModule |
| MatDialog | 对话框 | MatDialogModule |
| MatSnackbar | 提示条 | MatSnackBarModule |
| MatTabs | 标签页 | MatTabsModule |
| MatSelect | 下拉选择 | MatSelectModule |
| MatCheckbox | 复选框 | MatCheckboxModule |
| MatRadioButton | 单选框 | MatRadioModule |
| MatSidenav | 侧边导航 | MatSidenavModule |
| MatToolbar | 工具栏 | MatToolbarModule |

---

## SSR 支持

### Angular Universal

Angular Universal 用于服务端渲染（SSR）和预渲染（SSG）：

```bash
ng add @angular/ssr
```

### 服务端渲染优势

| 优势 | 说明 |
|------|------|
| **SEO** | 搜索引擎更好抓取 |
| **首屏加载** | 更快显示内容 |
| **低配设备** | 减少客户端计算 |

### 路由配置

```typescript
// app.routes.ts
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: '**', component: NotFoundComponent }
];

// 预渲染配置
export const config: ApplicationConfig = {
  providers: [
    provideClientHydration(),
    provideFileReuseStrategy(),
    withComponentInputBinding()
  ]
};
```

---

## 与 React/Vue 对比

### 核心对比表

| 维度 | Angular | React | Vue |
|------|---------|-------|-----|
| **类型** | 完整框架 | UI 库 | 渐进式框架 |
| **模板** | HTML + 指令 | JSX | HTML 模板 |
| **学习曲线** | 较陡 | 中等 | 较平缓 |
| **TypeScript** | **原生支持** | 支持 | 支持 |
| **状态管理** | Services/NgRx | Redux/Zustand | Pinia/Vuex |
| **响应式** | RxJS | Hooks | Proxy |
| **生态** | 完整 | 灵活 | 完整 |
| **社区** | 稳定 | 活跃 | 活跃 |
| **Google 维护** | ✅ | Meta | 独立 |

### 选型建议

| 场景 | 推荐框架 | 原因 |
|------|---------|------|
| 企业级大型应用 | **Angular** | 完整解决方案，严格架构 |
| 追求 TypeScript 最佳体验 | **Angular** | 原生 TypeScript 支持 |
| 中小型项目 | Vue | 灵活，门槛低 |
| 需要最大灵活性 | React | 最灵活 |
| 中文团队 | Vue | 中文文档友好 |
| 需要 RxJS 响应式 | **Angular/React** | Angular 内置 RxJS |
| 快速原型 | Vue | 上手最快 |

### 代码风格对比

```typescript
// Angular (TypeScript 原生)
@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    <div class="user-list">
      @for (user of users; track user.id) {
        <app-user-card [user]="user" (edit)="onEdit($event)" />
      }
    </div>
  `
})
export class UserListComponent {
  users: User[] = [];
  
  constructor(private userService: UserService) {
    this.users = this.userService.getUsers();
  }
  
  onEdit(id: string) {
    console.log('编辑:', id);
  }
}
```

```tsx
// React
import { useState, useEffect } from 'react';

function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  
  useEffect(() => {
    setUsers(getUsers());
  }, []);
  
  const handleEdit = (id: string) => {
    console.log('编辑:', id);
  };
  
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard 
          key={user.id} 
          user={user} 
          onEdit={handleEdit} 
        />
      ))}
    </div>
  );
}
```

---

## 实战技巧

### 1. AI 辅助组件生成

```typescript
// 使用 AI 生成 Angular 组件的 prompt
const generateComponentPrompt = `
创建一个 Angular 19 Standalone 组件 UserCard：
- 使用 <script setup> 类似语法
- Props 定义（使用 @Input）：
  - user: { id: string, name: string, email: string, avatar?: string }
  - size: 'sm' | 'md' | 'lg'
- Events 定义（使用 @Output）：
  - editEvent: EventEmitter<string>
  - deleteEvent: EventEmitter<string>
- 功能：
  - 显示用户头像（无头像显示首字母）
  - 点击编辑按钮触发 editEvent
  - 点击删除按钮触发 deleteEvent
- 样式：使用 SCSS
- 包含完整的 TypeScript 类型
- 导入 CommonModule, MatButtonModule 等
`;
```

### 2. 性能优化清单

| 优化点 | 方法 |
|-------|------|
| 变化检测 | OnPush 策略 |
| 懒加载 | loadComponent/lazyModules |
| 虚拟滚动 | CdkVirtualScrollViewport |
| 图片优化 | NgOptimizedImage |
| Bundle 大小 | source-map-explorer 分析 |
| 订阅清理 | async pipe 或 takeUntil |
| Signal 优化 | 优先使用 Signals |

### 3. OnPush 变化检测

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-optimized',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{ data.name }}</p>
    <button (click)="update()">更新</button>
  `
})
export class OptimizedComponent {
  data = { name: 'Angular' };
  
  update() {
    this.data = { ...this.data }; // 创建新引用
  }
}
```

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Angular 官方文档 | https://angular.dev |
| Angular Material | https://material.angular.io |
| Angular CLI | https://angular.io/cli |
| Angular Blog | https://blog.angular.dev |

### 学习资源

| 资源 | 说明 |
|------|------|
| Angular University | 高质量 Angular 课程 |
| Angular In Depth | 深度技术文章 |
| NG-BE Conference | Angular 欧洲会议 |

### 工具链

| 工具 | 用途 |
|------|------|
| Angular CLI | 项目脚手架和构建 |
| Angular Material | UI 组件库 |
| Angular CDK | 组件开发工具包 |
| NgRx | 状态管理 |
| Angular Universal | SSR 支持 |
| Jest | 单元测试 |
| Playwright | E2E 测试 |

---

> [!SUCCESS]
> Angular 作为 TypeScript 原生的企业级框架，在 2026 年通过 Signals、Standalone Components 和 zoneless 等新特性进一步现代化。对于需要构建大型企业应用、追求严格架构和 TypeScript 最佳实践的团队，Angular 仍然是不可替代的选择。

---

## 核心理念与设计哲学

### Angular 的设计哲学

Angular 是 Google 维护的企业级 TypeScript 框架，其设计哲学强调"完整解决方案"和"工程化"。

**1. Batteries Included 理念**

Angular 提供从路由、表单、HTTP 到依赖注入的完整工具链：

```
┌─────────────────────────────────────────────────────────────┐
│                    Angular 完整生态                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  @angular/core ──────────────────────────────────── 核心    │
│  ├── 组件系统                                              │
│  ├── 依赖注入                                              │
│  ├── 变化检测                                              │
│  └── 响应式编程                                            │
│                                                             │
│  @angular/router ─────────────────────────── 路由系统       │
│  ├── 导航守卫                                              │
│  ├── 懒加载                                               │
│  └── 路由动画                                              │
│                                                             │
│  @angular/forms ──────────────────────────── 表单处理        │
│  ├── 响应式表单                                            │
│  └── 模板驱动表单                                          │
│                                                             │
│  @angular/http ──────────────────────────── HTTP 客户端     │
│  @angular/animations ───────────────────── 动画系统         │
│  @angular/material ──────────────────────── UI 组件库       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**2. TypeScript 原生支持**

Angular 从设计之初就基于 TypeScript，提供完整的类型安全和 IDE 支持。

```typescript
// Angular 中的 TypeScript 示例
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  createdAt: Date;
}

// 强类型组件
@Component({
  selector: 'app-user-card',
  template: `<div>{{ user.name }}</div>`
})
export class UserCardComponent {
  @Input() user!: User; // 非空断言
  @Output() edit = new EventEmitter<string>();
  
  onEdit(): void {
    this.edit.emit(this.user.id);
  }
}
```

**3. 依赖注入（DI）**

Angular 的依赖注入系统是其核心特性之一：

```typescript
// 定义服务
@Injectable({
  providedIn: 'root' // 全局单例
})
export class UserService {
  private users: User[] = [];
  
  getUsers(): User[] {
    return this.users;
  }
  
  getUserById(id: string): User | undefined {
    return this.users.find(u => u.id === id);
  }
}

// 注入服务
@Component({
  selector: 'app-user-list',
  template: `<div>{{ users.length }} users</div>`
})
export class UserListComponent {
  constructor(private userService: UserService) {
    this.users = this.userService.getUsers();
  }
}
```

**4. 严格的模块化**

Angular 强调模块化架构，确保大型应用的可维护性：

```typescript
// feature.module.ts
@NgModule({
  declarations: [
    UserListComponent,
    UserCardComponent,
    UserDetailComponent
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    RouterModule.forChild(routes)
  ],
  exports: [
    UserListComponent // 导出供其他模块使用
  ]
})
export class UserModule {}
```

### Angular 解决的问题域

| 问题域 | Angular 解决方案 | 优势 |
|--------|----------------|------|
| **企业级架构** | 模块系统 + DI | 大型应用可维护 |
| **类型安全** | TypeScript 原生 | 开发时类型检查 |
| **一致性** | 完整工具链 | 团队协作标准化 |
| **性能** | Ivy + Signals | 高效渲染 |
| **测试** | 内置测试工具 | 易于测试 |
| **SEO** | SSR 支持 | 搜索引擎友好 |

---

## 完整安装与项目创建

### 环境准备

**Node.js 版本要求：**
- Angular 17+ 需要 Node.js 18.13 或更高版本
- Angular 19 需要 Node.js 20.13 或更高版本
- 推荐使用 Node.js 20 LTS

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 检查 npm 版本
npm --version
# 10.2.4

# 安装 Angular CLI
npm install -g @angular/cli

# 验证安装
ng version
```

### 创建 Angular 项目的多种方式

**方式一：Angular CLI（推荐）**

```bash
# 创建新项目
ng new my-angular-app

# 选项：
# ? Would you like to add Angular routing? Yes
# ? Which stylesheet format would you like to use? SCSS
# ? Would you like to enable Server-Side Rendering (SSR) and Static Site Generation (SSG)? No

# 进入目录
cd my-angular-app

# 启动开发服务器
ng serve
ng serve --port 4200 --open

# 构建生产版本
ng build
ng build --configuration=production

# 运行测试
ng test
ng test --watch=false --code-coverage
```

**方式二：使用特定模板**

```bash
# 创建带路由的项目
ng new my-app --routing

# 创建带 SCSS 的项目
ng new my-app --style=scss

# 创建严格模式的 TypeScript 项目
ng new my-app --strict

# 创建所有选项
ng new my-app --routing --style=scss --strict --ssr=false --skip-git
```

**方式三：离线创建**

```bash
# 使用离线模式
ng new my-app --skip-install

# 稍后手动安装
cd my-app
npm install
```

### 项目结构详解

**标准 Angular 项目结构：**

```
my-angular-app/
├── src/                      # 源代码目录
│   ├── app/                 # 应用根目录
│   │   ├── components/      # 共享组件
│   │   │   ├── header/
│   │   │   ├── footer/
│   │   │   └── shared/
│   │   ├── services/        # 服务
│   │   │   ├── user.service.ts
│   │   │   └── auth.service.ts
│   │   ├── guards/          # 路由守卫
│   │   ├── interceptors/    # HTTP 拦截器
│   │   ├── pipes/           # 管道
│   │   ├── models/          # 数据模型
│   │   │   └── user.model.ts
│   │   ├── pages/           # 页面组件
│   │   │   ├── home/
│   │   │   ├── about/
│   │   │   └── users/
│   │   ├── app.component.ts       # 根组件
│   │   ├── app.component.html
│   │   ├── app.component.scss
│   │   ├── app.config.ts          # 应用配置
│   │   ├── app.routes.ts          # 路由配置
│   │   └── app.component.spec.ts  # 测试文件
│   ├── assets/              # 静态资源
│   ├── environments/         # 环境配置
│   ├── styles.scss          # 全局样式
│   ├── index.html
│   └── main.ts              # 入口文件
├── angular.json             # Angular CLI 配置
├── package.json
├── tsconfig.json           # TypeScript 配置
└── .gitignore
```

**Standalone 组件项目结构（Angular 14+）：**

```
my-angular-app/
├── src/
│   ├── app/
│   │   ├── components/     # 共享组件
│   │   ├── services/       # 服务
│   │   ├── pages/          # 页面
│   │   ├── app.component.ts
│   │   ├── app.config.ts
│   │   └── app.routes.ts
│   ├── main.ts
│   └── index.html
└── angular.json
```

---

## 组件系统详解

### Standalone Components

Angular 14 引入了 Standalone Components，大幅简化了组件创建。

**基础 Standalone 组件：**

```typescript
// user-card.component.ts
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterModule } from '@angular/router';

@Component({
  selector: 'app-user-card',
  standalone: true, // 标记为独立组件
  imports: [CommonModule, RouterModule], // 导入需要的模块
  template: `
    <div class="card">
      <img [src]="user.avatar" [alt]="user.name" class="avatar" />
      <div class="info">
        <h3>{{ user.name }}</h3>
        <p>{{ user.email }}</p>
        <span [class]="'role role-' + user.role">
          {{ user.role }}
        </span>
      </div>
      <button (click)="onEdit()">编辑</button>
      <button (click)="onDelete()">删除</button>
    </div>
  `,
  styles: [`
    .card {
      display: flex;
      gap: 1rem;
      padding: 1rem;
      border: 1px solid #e5e7eb;
      border-radius: 8px;
    }
    .avatar {
      width: 48px;
      height: 48px;
      border-radius: 50%;
    }
    .role-admin { background: #fee2e2; color: #991b1b; }
    .role-user { background: #dbeafe; color: #1e40af; }
    .role-guest { background: #f3f4f6; color: #4b5563; }
  `]
})
export class UserCardComponent {
  @Input({ required: true }) user!: User;
  @Output() edit = new EventEmitter<string>();
  @Output() delete = new EventEmitter<string>();
  
  onEdit(): void {
    this.edit.emit(this.user.id);
  }
  
  onDelete(): void {
    this.delete.emit(this.user.id);
  }
}
```

**使用 inject() 函数注入依赖：**

```typescript
import { Component, inject } from '@angular/core';
import { UserService } from './services/user.service';

@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, UserCardComponent]
})
export class UserListComponent {
  // 使用 inject() 注入服务
  private userService = inject(UserService);
  
  users$ = this.userService.getUsers();
  
  onEditUser(id: string): void {
    console.log('编辑用户:', id);
  }
  
  onDeleteUser(id: string): void {
    this.userService.deleteUser(id);
  }
}
```

### Props 与 Events

**使用 @Input 和 @Output：**

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-button',
  standalone: true,
  template: `
    <button 
      [class]="variant"
      [disabled]="disabled"
      (click)="handleClick($event)"
    >
      <ng-content></ng-content>
    </button>
  `
})
export class ButtonComponent {
  @Input() variant: 'primary' | 'secondary' | 'danger' = 'primary';
  @Input() disabled = false;
  @Output() clicked = new EventEmitter<MouseEvent>();
  
  handleClick(event: MouseEvent): void {
    if (!this.disabled) {
      this.clicked.emit(event);
    }
  }
}

// 使用
@Component({
  selector: 'app-demo',
  standalone: true,
  imports: [ButtonComponent],
  template: `
    <app-button 
      variant="primary"
      (clicked)="onButtonClick($event)"
    >
      点击我
    </app-button>
  `
})
export class DemoComponent {
  onButtonClick(event: MouseEvent): void {
    console.log('按钮点击', event);
  }
}
```

### 双向绑定

**使用 [(ngModel)]：**

```typescript
import { Component } from '@angular/core';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-form',
  standalone: true,
  imports: [FormsModule],
  template: `
    <div>
      <input 
        [(ngModel)]="name" 
        placeholder="姓名"
      />
      <p>你好，{{ name }}！</p>
      
      <select [(ngModel)]="role">
        <option value="admin">管理员</option>
        <option value="user">普通用户</option>
        <option value="guest">访客</option>
      </select>
      <p>角色: {{ role }}</p>
      
      <label>
        <input type="checkbox" [(ngModel)]="remember" />
        记住我
      </label>
    </div>
  `
})
export class FormComponent {
  name = '';
  role = 'user';
  remember = false;
}
```

---

## 依赖注入详解

### 依赖注入系统

Angular 的依赖注入系统是其核心特性之一，提供强大的解耦能力。

**服务定义：**

```typescript
// user.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root' // 全局单例
})
export class UserService {
  private http = inject(HttpClient);
  private apiUrl = '/api/users';
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }
  
  getUserById(id: string): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }
  
  createUser(user: CreateUserDto): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }
  
  updateUser(id: string, user: UpdateUserDto): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }
  
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

**多层级注入：**

```typescript
// 层级注入示例

// 1. 应用级单例（providedIn: 'root'）
@Injectable({ providedIn: 'root' })
export class AppService {}

// 2. 平台级（providedIn: 'platform'）
@Injectable({ providedIn: 'platform' })
export class PlatformService {}

// 3. 任意模块级（providedIn: 'any'）
@Injectable({ providedIn: 'any' })
export class ModuleService {}

// 4. 组件级
@Component({
  selector: 'app-child',
  providers: [
    { provide: ComponentService, useClass: ComponentService }
  ]
})
export class ChildComponent {
  private service = inject(ComponentService);
}
```

### Injection Tokens

**使用 InjectionToken：**

```typescript
import { Injectable, Inject, InjectionToken } from '@angular/core';

// 定义配置接口
export interface AppConfig {
  apiUrl: string;
  appName: string;
  theme: 'light' | 'dark';
}

// 创建 Injection Token
export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');

// 使用
@Injectable()
export class ConfigService {
  constructor(@Inject(APP_CONFIG) private config: AppConfig) {
    console.log('App Name:', this.config.appName);
  }
}

// 注册
bootstrapApplication(AppComponent, {
  providers: [
    { 
      provide: APP_CONFIG, 
      useValue: {
        apiUrl: 'https://api.example.com',
        appName: 'My App',
        theme: 'light'
      }
    }
  ]
});
```

---

## RxJS 响应式编程

### RxJS 核心概念

**Observable vs Subject：**

```typescript
import { 
  Observable, 
  Subject, 
  BehaviorSubject, 
  ReplaySubject, 
  AsyncSubject,
  interval,
  of,
  from,
  fromEvent,
  throwError
} from 'rxjs';
import { 
  map, 
  filter, 
  switchMap, 
  mergeMap, 
  concatMap,
  take, 
  debounceTime, 
  distinctUntilChanged,
  catchError,
  retry,
  tap,
  delay
} from 'rxjs/operators';

// 1. Observable - 冷数据源
const observable = new Observable(subscriber => {
  console.log('Observable started');
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  subscriber.complete();
});

// 2. Subject - 热数据源，多播
const subject = new Subject<number>();
subject.subscribe(x => console.log('A:', x));
subject.subscribe(x => console.log('B:', x));
subject.next(1); // A: 1, B: 1
subject.next(2); // A: 2, B: 2

// 3. BehaviorSubject - 有初始值，记住最后一个值
const behaviorSubject = new BehaviorSubject<string>('initial');
behaviorSubject.subscribe(x => console.log('Subscriber:', x));
behaviorSubject.next('new value'); // Subscriber: new value

// 4. ReplaySubject - 重放最后 N 个值
const replaySubject = new ReplaySubject<number>(2);
replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);
replaySubject.subscribe(x => console.log('Replay:', x)); // Replay: 2, 3

// 5. AsyncSubject - 只发送最后一个值（complete 时）
const asyncSubject = new AsyncSubject<number>();
asyncSubject.next(1);
asyncSubject.next(2);
asyncSubject.subscribe(x => console.log('Async:', x)); // 不输出
asyncSubject.complete(); // Async: 2
```

### HTTP 请求与操作符

**常用操作符：**

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable, of, throwError } from 'rxjs';
import { 
  map, 
  filter, 
  catchError, 
  retry, 
  tap,
  debounceTime,
  distinctUntilChanged,
  switchMap,
  finalize
} from 'rxjs/operators';

@Component({
  selector: 'app-users',
  standalone: true,
  imports: [CommonModule],
  template: `
    @if (loading) {
      <div>加载中...</div>
    }
    
    @for (user of users; track user.id) {
      <div>{{ user.name }}</div>
    }
    
    @if (error) {
      <div class="error">{{ error }}</div>
    }
  `
})
export class UsersComponent {
  private http = inject(HttpClient);
  
  loading = false;
  users: User[] = [];
  error = '';
  
  loadUsers(): void {
    this.loading = true;
    this.error = '';
    
    this.http.get<User[]>('/api/users').pipe(
      map(users => users.map(u => ({
        ...u,
        name: u.name.toUpperCase()
      }))),
      retry(3), // 重试 3 次
      catchError(error => {
        this.error = '加载失败: ' + error.message;
        return of([]); // 返回空数组
      }),
      finalize(() => {
        this.loading = false;
      })
    ).subscribe(users => {
      this.users = users;
    });
  }
}
```

**搜索防抖示例：**

```typescript
import { Component, OnInit, OnDestroy, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subject, debounceTime, distinctUntilChanged, switchMap, takeUntil } from 'rxjs';

@Component({
  selector: 'app-search',
  standalone: true,
  imports: [FormsModule],
  template: `
    <input 
      [ngModel]="searchTerm"
      (ngModelChange)="onSearch($event)"
      placeholder="搜索..."
    />
    <div class="results">
      @for (result of results; track result.id) {
        <div>{{ result.name }}</div>
      }
    </div>
  `
})
export class SearchComponent implements OnInit, OnDestroy {
  private http = inject(HttpClient);
  private searchSubject = new Subject<string>();
  private destroy$ = new Subject<void>();
  
  searchTerm = '';
  results: SearchResult[] = [];
  
  ngOnInit(): void {
    this.searchSubject.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(term => this.http.get<SearchResult[]>(`/api/search?q=${term}`)),
      takeUntil(this.destroy$)
    ).subscribe(results => {
      this.results = results;
    });
  }
  
  onSearch(term: string): void {
    this.searchTerm = term;
    this.searchSubject.next(term);
  }
  
  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**合并请求：**

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { forkJoin, combineLatest, of } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  imports: [CommonModule]
})
export class DashboardComponent {
  private http = inject(HttpClient);
  
  // forkJoin - 并行请求，全部完成
  loadDashboard(): void {
    forkJoin({
      users: this.http.get<User[]>('/api/users'),
      posts: this.http.get<Post[]>('/api/posts'),
      comments: this.http.get<Comment[]>('/api/comments')
    }).pipe(
      catchError(error => {
        console.error('加载失败', error);
        return of({ users: [], posts: [], comments: [] });
      })
    ).subscribe(data => {
      console.log('Users:', data.users);
      console.log('Posts:', data.posts);
      console.log('Comments:', data.comments);
    });
  }
  
  // combineLatest - 组合多个流，任一变化时更新
  combinedData$ = combineLatest([
    this.http.get<User[]>('/api/users'),
    this.http.get<Role[]>('/api/roles')
  ]).pipe(
    map(([users, roles]) => {
      return users.map(user => ({
        ...user,
        roleName: roles.find(r => r.id === user.roleId)?.name || 'Unknown'
      }));
    })
  );
}
```

---

## 路由系统详解

### Angular Router 完整配置

**路由配置：**

```typescript
// app.routes.ts
import { Routes } from '@angular/router';
import { inject } from '@angular/core';
import { AuthService } from './services/auth.service';

export const routes: Routes = [
  {
    path: '',
    redirectTo: '/home',
    pathMatch: 'full'
  },
  {
    path: 'home',
    loadComponent: () => import('./pages/home/home.component')
      .then(m => m.HomeComponent)
  },
  {
    path: 'about',
    loadComponent: () => import('./pages/about/about.component')
      .then(m => m.AboutComponent)
  },
  {
    path: 'users',
    loadChildren: () => import('./pages/users/users.routes')
      .then(m => m.USER_ROUTES)
  },
  {
    path: 'admin',
    loadComponent: () => import('./pages/admin/admin.component')
      .then(m => m.AdminComponent),
    canMatch: [() => {
      const auth = inject(AuthService);
      return auth.hasRole('admin');
    }]
  },
  {
    path: 'profile/:id',
    loadComponent: () => import('./pages/profile/profile.component')
      .then(m => m.ProfileComponent),
    resolve: {
      user: () => inject(UserService).getCurrentUser()
    }
  },
  {
    path: '**',
    loadComponent: () => import('./pages/not-found/not-found.component')
      .then(m => m.NotFoundComponent)
  }
];
```

**路由守卫：**

```typescript
// auth.guard.ts
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);
  
  if (auth.isAuthenticated()) {
    return true;
  }
  
  // 保存要访问的路径，登录后跳转
  router.navigate(['/login'], {
    queryParams: { returnUrl: state.url }
  });
  return false;
};

// 角色守卫
export const roleGuard = (requiredRole: string): CanActivateFn => {
  return () => {
    const auth = inject(AuthService);
    const router = inject(Router);
    
    if (auth.hasRole(requiredRole)) {
      return true;
    }
    
    router.navigate(['/unauthorized']);
    return false;
  };
};

// 延迟加载守卫
export const loadGuard: CanActivateFn = () => {
  const auth = inject(AuthService);
  return auth.isReady();
};

// 使用
{
  path: 'dashboard',
  canActivate: [authGuard, roleGuard('admin')],
  loadComponent: () => import('./dashboard.component')
}
```

**路由参数：**

```typescript
import { Component, inject } from '@angular/core';
import { Router, ActivatedRoute, RouterModule } from '@angular/router';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-profile',
  standalone: true,
  imports: [CommonModule, RouterModule],
  template: `
    <h1>用户资料</h1>
    
    <p>用户ID: {{ userId }}</p>
    <p>Tab: {{ tab }}</p>
    
    <nav>
      <a routerLink="info" routerLinkActive="active">基本信息</a>
      <a routerLink="posts" routerLinkActive="active">帖子</a>
      <a routerLink="settings" routerLinkActive="active">设置</a>
    </nav>
    
    <button (click)="goBack()">返回</button>
  `
})
export class ProfileComponent {
  private route = inject(ActivatedRoute);
  private router = inject(Router);
  
  // 静态获取参数
  userId = this.route.snapshot.paramMap.get('id');
  
  // 响应式获取参数
  tab = this.route.snapshot.queryParamMap.get('tab') || 'info';
  
  // 监听参数变化
  constructor() {
    this.route.paramMap.subscribe(params => {
      console.log('用户ID变化:', params.get('id'));
    });
    
    this.route.queryParamMap.subscribe(params => {
      console.log('查询参数变化:', params.get('tab'));
    });
  }
  
  goBack(): void {
    this.router.navigate(['/users']);
  }
}
```

---

## 表单处理详解

### 响应式表单

**基础响应式表单：**

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div class="form-group">
        <label for="name">姓名</label>
        <input id="name" formControlName="name" />
        @if (form.get('name')?.hasError('required') && form.get('name')?.touched) {
          <span class="error">姓名是必填项</span>
        }
        @if (form.get('name')?.hasError('minlength')) {
          <span class="error">姓名至少2个字符</span>
        }
      </div>
      
      <div class="form-group">
        <label for="email">邮箱</label>
        <input id="email" formControlName="email" type="email" />
        @if (form.get('email')?.hasError('email')) {
          <span class="error">请输入有效的邮箱</span>
        }
      </div>
      
      <div formGroupName="address">
        <h3>地址</h3>
        <input formControlName="street" placeholder="街道" />
        <input formControlName="city" placeholder="城市" />
      </div>
      
      <button type="submit" [disabled]="form.invalid">
        提交
      </button>
      
      <button type="button" (click)="fillDefaults()">
        填充默认值
      </button>
    </form>
  `
})
export class UserFormComponent {
  private fb = inject(FormBuilder);
  
  form: FormGroup = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(2)]],
    email: ['', [Validators.required, Validators.email]],
    age: [18, [Validators.min(0), Validators.max(150)]],
    role: ['user'],
    address: this.fb.group({
      street: [''],
      city: [''],
      zipCode: ['', Validators.pattern(/^\d{6}$/)]
    })
  });
  
  fillDefaults(): void {
    this.form.patchValue({
      name: '张三',
      email: 'zhangsan@example.com',
      age: 25,
      address: {
        street: '中山路123号',
        city: '北京',
        zipCode: '100000'
      }
    });
  }
  
  onSubmit(): void {
    if (this.form.valid) {
      console.log('表单值:', this.form.value);
    } else {
      this.form.markAllAsTouched();
    }
  }
}
```

**动态表单：**

```typescript
import { Component, inject } from '@angular/core';
import { FormBuilder, FormArray, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';

@Component({
  selector: 'app-dynamic-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div formArrayName="items">
        @for (item of items.controls; track $index; let i = $index) {
          <div [formGroupName]="i">
            <input formControlName="name" placeholder="名称" />
            <input formControlName="quantity" type="number" placeholder="数量" />
            <button type="button" (click)="removeItem(i)">删除</button>
          </div>
        }
      </div>
      
      <button type="button" (click)="addItem()">添加项目</button>
      <button type="submit">提交</button>
    </form>
  `
})
export class DynamicFormComponent {
  private fb = inject(FormBuilder);
  
  form: FormGroup = this.fb.group({
    items: this.fb.array([this.createItem()])
  });
  
  get items(): FormArray {
    return this.form.get('items') as FormArray;
  }
  
  createItem(): FormGroup {
    return this.fb.group({
      name: ['', Validators.required],
      quantity: [1, [Validators.required, Validators.min(1)]]
    });
  }
  
  addItem(): void {
    this.items.push(this.createItem());
  }
  
  removeItem(index: number): void {
    this.items.removeAt(index);
  }
  
  onSubmit(): void {
    console.log('表单值:', this.form.value);
  }
}
```

---

## 样式方案详解

### SCSS 最佳实践

**使用 SCSS 创建设计系统：**

```scss
// styles/_variables.scss
$colors: (
  primary: #3b82f6,
  secondary: #6b7280,
  success: #10b981,
  warning: #f59e0b,
  danger: #ef4444,
  info: #06b6d4
);

$spacing: (
  xs: 0.25rem,
  sm: 0.5rem,
  md: 1rem,
  lg: 1.5rem,
  xl: 2rem
);

$breakpoints: (
  sm: 640px,
  md: 768px,
  lg: 1024px,
  xl: 1280px
);

// styles/_mixins.scss
@mixin respond-to($breakpoint) {
  @if map-has-key($breakpoints, $breakpoint) {
    @media (min-width: map-get($breakpoints, $breakpoint)) {
      @content;
    }
  }
}

@mixin flex-center {
  display: flex;
  align-items: center;
  justify-content: center;
}

@mixin card {
  background: white;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  padding: 1rem;
}
```

**组件样式：**

```scss
// button.component.scss
@use '../../styles/variables' as *;
@use '../../styles/mixins' as *;

.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  padding: 0.5rem 1rem;
  font-size: 0.875rem;
  font-weight: 500;
  border-radius: 0.375rem;
  border: none;
  cursor: pointer;
  transition: all 0.2s;
  
  &:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  
  @each $name, $color in $colors {
    &-#{$name} {
      background: $color;
      color: white;
      
      &:hover:not(:disabled) {
        background: darken($color, 10%);
      }
      
      &:active:not(:disabled) {
        transform: translateY(1px);
      }
    }
  }
  
  &-outline {
    background: transparent;
    border: 1px solid currentColor;
    
    @each $name, $color in $colors {
      &-#{$name} {
        color: $color;
        &:hover:not(:disabled) {
          background: rgba($color, 0.1);
        }
      }
    }
  }
  
  &-sm {
    padding: 0.25rem 0.5rem;
    font-size: 0.75rem;
  }
  
  &-lg {
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
  }
}
```

### Tailwind CSS

```bash
ng add @tailwindcss/angular
```

```typescript
// 使用 Tailwind
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="max-w-sm bg-white rounded-lg shadow-md p-6">
      <h2 class="text-xl font-bold text-gray-900 mb-2">
        {{ title }}
      </h2>
      <p class="text-gray-600 mb-4">{{ content }}</p>
      <button 
        class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
        (click)="onAction()"
      >
        {{ actionLabel }}
      </button>
    </div>
  `
})
export class CardComponent {
  @Input() title = '';
  @Input() content = '';
  @Input() actionLabel = '确定';
  @Output() action = new EventEmitter<void>();
  
  onAction(): void {
    this.action.emit();
  }
}
```

---

## 性能优化详解

### OnPush 变化检测

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-optimized',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <p>{{ data.name }}</p>
    <button (click)="update()">更新</button>
  `
})
export class OptimizedComponent {
  data = { name: 'Angular' };
  
  update(): void {
    // 创建新引用触发更新
    this.data = { ...this.data };
  }
}
```

### Signals 响应式系统

```typescript
import { Component, signal, computed, effect } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <p>计数: {{ count() }}</p>
    <p>翻倍: {{ doubled() }}</p>
    <button (click)="increment()">增加</button>
    <button (click)="reset()">重置</button>
  `
})
export class CounterComponent {
  // Signals
  count = signal(0);
  
  // Computed - 派生值
  doubled = computed(() => this.count() * 2);
  status = computed(() => {
    const c = this.count();
    if (c < 0) return 'negative';
    if (c === 0) return 'zero';
    return 'positive';
  });
  
  // Effects - 副作用
  constructor() {
    effect(() => {
      console.log('count 变化了:', this.count());
      document.title = `计数: ${this.count()}`;
    });
  }
  
  increment(): void {
    this.count.update(c => c + 1);
  }
  
  reset(): void {
    this.count.set(0);
  }
}
```

### 延迟加载

```typescript
// 路由级延迟加载
{
  path: 'users',
  loadChildren: () => import('./users/users.routes')
    .then(m => m.USER_ROUTES)
}

// 组件级延迟加载
const HeavyChart = () => import('./components/heavy-chart.component')
  .then(m => m.HeavyChartComponent);

// 延迟加载视图
@Component({
  template: `
    <nav>...</nav>
    
    @defer (on viewport) {
      <heavy-component />
    } @loading (minimum 200ms) {
      <skeleton-loader />
    } @placeholder {
      <placeholder-content />
    }
  `
})
```

---

## DevTools 与调试

### Angular DevTools

**主要功能：**

1. **Components 面板**
   - 查看组件树
   - 检查组件状态
   - 修改组件数据
   - 查看注入器

2. **Profiler 面板**
   - 性能分析
   - 变化检测分析
   - 识别性能问题

### 调试技巧

```typescript
// 1. DebugElement
import { ComponentFixture } from '@angular/core/testing';

fixture.debugElement.injector.get(UserService);

// 2. JSON pipe 调试
{{ data | json }}

// 3. ng inspect
// 在模板中添加
<div ngNonBindable>{{ secret }}</div>
```

---

## 测试工具与实践

### 测试框架概述

Angular 提供完整的测试解决方案，包括单元测试、集成测试和端到端测试。

|| 测试类型 | 工具 | 说明 |
||---------|------|------|
|| **单元测试** | Jasmine + Karma/Jest | 组件、服务、Pipe 测试 |
|| **集成测试** | Testing Library | 组件交互测试 |
|| **端到端测试** | Playwright / Cypress | 完整用户流程 |

### Karma 与 Jasmine 基础

**Karma 配置：**

```typescript
// karma.conf.js
module.exports = function(config) {
  config.set({
    basePath: '',
    frameworks: ['jasmine', '@angular-devkit/build-angular'],
    plugins: [
      require('karma-jasmine'),
      require('karma-chrome-launcher'),
      require('karma-jasmine-html-reporter'),
      require('@angular-devkit/build-angular/plugins/karma')
    ],
    client: {
      jasmine: {
        random: true,
        seed: 42
      }
    },
    coverageReporter: {
      dir: require('path').join(__dirname, './coverage'),
      reporters: [{ type: 'html' }, { type: 'text-summary' }]
    }
  });
};
```

### 组件单元测试

**基础组件测试：**

```typescript
// user-card.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { UserCardComponent } from './user-card.component';

describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  const mockUser = {
    id: '1',
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const
  };

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [UserCardComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;
    component.user = mockUser;
    fixture.detectChanges();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display user name', () => {
    const compiled = fixture.nativeElement;
    expect(compiled.querySelector('.name').textContent).toContain('张三');
  });

  it('should emit edit event', () => {
    spyOn(component.edit, 'emit');
    const button = fixture.nativeElement.querySelector('.edit-btn');
    button.click();
    expect(component.edit.emit).toHaveBeenCalledWith('1');
  });
});
```

### 服务测试

**HTTP 服务测试（使用 HttpClientTestingModule）：**

```typescript
// user.service.spec.ts
import { TestBed } from '@angular/core/testing';
import {
  HttpClientTestingModule,
  HttpTestingController
} from '@angular/common/http/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch users', () => {
    const mockUsers = [
      { id: '1', name: '张三' },
      { id: '2', name: '李四' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users[0].name).toBe('张三');
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  it('should handle error', () => {
    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error.status).toBe(404);
      }
    });

    const req = httpMock.expectOne('/api/users');
    req.flush('Not found', { status: 404, statusText: 'Not Found' });
  });
});
```

### 表单测试

**响应式表单测试：**

```typescript
// user-form.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ReactiveFormsModule } from '@angular/forms';
import { UserFormComponent } from './user-form.component';

describe('UserFormComponent', () => {
  let component: UserFormComponent;
  let fixture: ComponentFixture<UserFormComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [UserFormComponent],
      imports: [ReactiveFormsModule]
    }).compileComponents();

    fixture = TestBed.createComponent(UserFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create form with empty values', () => {
    expect(component.form.valid).toBeFalsy();
  });

  it('should validate required fields', () => {
    component.form.patchValue({
      name: '',
      email: 'invalid'
    });
    fixture.detectChanges();

    expect(component.form.get('name')?.valid).toBeFalsy();
    expect(component.form.get('email')?.valid).toBeFalsy();
  });

  it('should submit form', () => {
    spyOn(component.submitForm, 'emit');

    component.form.patchValue({
      name: '张三',
      email: 'zhangsan@example.com'
    });
    fixture.detectChanges();

    const form = fixture.nativeElement.querySelector('form');
    form.dispatchEvent(new Event('submit'));
    fixture.detectChanges();

    expect(component.submitForm.emit).toHaveBeenCalled();
  });
});
```

### 路由测试

**路由守卫测试：**

```typescript
// auth.guard.spec.ts
import { TestBed } from '@angular/core/testing';
import { RouterTestingModule } from '@angular/router/testing';
import { inject } from '@angular/core';
import { authGuard } from './auth.guard';

describe('authGuard', () => {
  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [RouterTestingModule]
    });
  });

  it('should allow authenticated users', () => {
    localStorage.setItem('token', 'valid-token');

    const guard = TestBed.inject(authGuard);
    const canActivate = guard(null as any, { url: '/dashboard' } as any);

    expect(canActivate).toBeTruthy();
    localStorage.removeItem('token');
  });

  it('should redirect unauthenticated users', () => {
    const guard = TestBed.inject(authGuard);
    const canActivate = guard(null as any, { url: '/dashboard' } as any);

    expect(canActivate).toBeFalsy();
  });
});
```

### Jest 替代方案

**配置 Jest：**

```bash
npm install -D jest @types/jest ts-jest @angular-builders/jest
```

```typescript
// jest.config.js
module.exports = {
  preset: 'jest-preset-angular',
  setupFilesAfterFramework: ['<rootDir>/setup-jest.ts'],
  testPathPattern: ['\\.spec\\.ts$'],
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.module.ts',
    '!src/main.ts'
  ],
  coverageDirectory: 'coverage'
};
```

**Jest 测试示例：**

```typescript
// counter.component.spec.ts
import { render, screen, fireEvent } from '@testing-library/angular';
import { CounterComponent } from './counter.component';

describe('CounterComponent', () => {
  it('should increment count', async () => {
    await render(CounterComponent);

    const button = screen.getByRole('button', { name: /increment/i });
    fireEvent.click(button);

    expect(screen.getByText('Count: 1')).toBeTruthy();
  });
});
```

### 端到端测试（Playwright）

**Playwright 配置：**

```typescript
// e2e/config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:4200',
    trace: 'on-first-retry'
  },
  projects: [
    {
      name: 'chromium',
      use: { browserName: 'chromium' }
    }
  ]
});
```

**E2E 测试示例：**

```typescript
// e2e/user.spec.ts
import { test, expect } from '@playwright/test';

test.describe('User Management', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/users');
  });

  test('should display user list', async ({ page }) => {
    await expect(page.locator('h1')).toHaveText('Users');
    await expect(page.locator('.user-card')).toHaveCount(3);
  });

  test('should create new user', async ({ page }) => {
    await page.click('button:has-text("Add User")');
    await page.fill('input[name="name"]', '王五');
    await page.fill('input[name="email"]', 'wangwu@example.com');
    await page.click('button[type="submit"]');

    await expect(page.locator('.user-card')).toHaveCount(4);
  });

  test('should edit user', async ({ page }) => {
    await page.locator('.user-card').first().click('button:has-text("Edit")');
    await page.fill('input[name="name"]', '更新后的名称');
    await page.click('button[type="submit"]');

    await expect(page.locator('.user-card').first()).toContainText('更新后的名称');
  });

  test('should delete user', async ({ page }) => {
    const initialCount = await page.locator('.user-card').count();
    await page.locator('.user-card').first().click('button:has-text("Delete")');

    await expect(page.locator('.user-card')).toHaveCount(initialCount - 1);
  });
});
```

### 测试覆盖率

**覆盖率报告：**

```bash
# 生成覆盖率报告
ng test --code-coverage

# 查看 HTML 报告
open coverage/index.html
```

**覆盖率配置：**

```json
{
  "angularCompilerOptions": {
    "strictInjectionParameters": true,
    "strictInputAccessors": true,
    "strictTemplates": true
  }
}
```

### 测试最佳实践

|| 实践 | 说明 |
||-----|------|
|| **AAA 模式** | Arrange（准备）→ Act（执行）→ Assert（断言） |
|| **单一职责** | 每个测试只验证一个行为 |
|| **隔离依赖** | 使用 Mock/Spy 隔离外部依赖 |
|| **可读的测试名** | 使用描述性测试名称 |
|| **快速反馈** | 单元测试应快速执行 |
|| **独立测试** | 测试之间不应有依赖 |

---

## 学习路径与资源

### Angular 学习路线图

```
第一阶段：基础（2-3周）
├── TypeScript 基础
├── Angular CLI 使用
├── 组件基础
├── 模板语法
├── 数据绑定
└── 依赖注入基础

第二阶段：进阶（3-4周）
├── 依赖注入进阶
├── RxJS 完整掌握
├── 响应式表单
├── 路由进阶
├── HTTP 客户端
└── 测试基础

第三阶段：生态（3-4周）
├── Angular Material
├── Angular CDK
├── NgRx 状态管理
├── Angular Animations
└── SSR/SSG

第四阶段：高级（持续学习）
├── Angular Signals
├── Zoneless
├── 微前端
├── 企业架构
└── 性能优化
```

### 推荐学习资源

**官方资源：**
- Angular 官方文档：https://angular.dev
- Angular 中文文档：https://angular.cn
- Angular Material：https://material.angular.io

**教程与课程：**
| 资源 | 类型 | 难度 |
|------|------|------|
| Angular 官方教程 | 交互式 | 入门 |
| Angular University | 视频 | 全阶段 |
| Ultimate Angular | 视频 | 进阶 |

**工具与插件：**
- Angular DevTools：Chrome 商店
- Angular Language Service：VS Code 扩展

---

## 高级模式与最佳实践

### 内容投影（Content Projection）深度使用

Angular 的 `<ng-content>` 用于内容投影：

```typescript
// components/card.component.ts
import { Component, ContentChild, ContentChildren, QueryList } from '@angular/core'
import { Directive, Input } from '@angular/core'

// 多重插槽投影
@Component({
  selector: 'app-multi-slot-card',
  standalone: true,
  template: `
    <div class="card">
      <header class="card-header">
        <ng-content select="[card-title]"></ng-content>
      </header>
      
      <div class="card-body">
        <ng-content select="[card-body]"></ng-content>
      </div>
      
      <footer class="card-footer">
        <ng-content select="[card-footer]"></ng-content>
      </footer>
    </div>
  `
})
export class MultiSlotCardComponent {}

// 使用
@Component({
  standalone: true,
  template: `
    <app-multi-slot-card>
      <h2 card-title>卡片标题</h2>
      <p card-body>卡片内容</p>
      <button card-footer>操作按钮</button>
    </app-multi-slot-card>
  `
})
export class ParentComponent {}
```

```typescript
// 获取投影内容的引用
@Component({
  selector: 'app-smart-card',
  standalone: true,
  template: `
    <div class="card">
      <ng-content></ng-content>
    </div>
  `
})
export class SmartCardComponent {
  // 获取投影内容的第一个元素
  @ContentChild('header') header!: ElementRef
  @ContentChild('body') body!: ElementRef

  ngAfterContentInit() {
    console.log('Header:', this.header)
    console.log('Body:', this.body)
  }
}
```

### 动态组件加载

运行时动态加载组件：

```typescript
import { Component, ComponentRef, ViewChild, ViewContainerRef, OnDestroy } from '@angular/core'
import { Injector, ReflectiveInjector } from '@angular/core'

// 动态组件接口
interface DynamicComponent {
  data: any
  onClose: () => void
}

@Component({
  selector: 'app-dynamic-host',
  standalone: true,
  template: `
    <div class="host-container">
      <ng-container #dynamicHost></ng-container>
    </div>
  `
})
export class DynamicHostComponent implements OnDestroy {
  @ViewChild('dynamicHost', { read: ViewContainerRef }) host!: ViewContainerRef
  
  private componentRef?: ComponentRef<any>

  loadComponent(componentClass: Type<any>, data: any) {
    // 清除旧组件
    this.host.clear()
    
    // 创建新组件
    this.componentRef = this.host.createComponent(componentClass, {
      injector: this.createInjector(data)
    })
  }

  private createInjector(data: any): Injector {
    return ReflectiveInjector.fromProviderTokens({
      providers: [{ provide: 'COMPONENT_DATA', useValue: data }]
    })
  }

  ngOnDestroy() {
    this.componentRef?.destroy()
  }
}
```

### 指令深度使用

创建复杂指令：

```typescript
import { Directive, ElementRef, Renderer2, HostListener, HostBinding, Input, Output, EventEmitter } from '@angular/core'

// 拖拽指令
@Directive({
  selector: '[appDraggable]',
  standalone: true
})
export class DraggableDirective {
  @Input() dragHandle: string = ''
  @Output() dragStart = new EventEmitter<{ x: number, y: number }>()
  @Output() dragEnd = new EventEmitter<{ x: number, y: number }>()

  private isDragging = false
  private startX = 0
  private startY = 0

  @HostBinding('style.cursor') cursor = 'move'
  @HostBinding('style.userSelect') userSelect = 'none'

  @HostListener('mousedown', ['$event'])
  onMouseDown(event: MouseEvent) {
    if (this.dragHandle && !(event.target as HTMLElement).closest(this.dragHandle)) {
      return
    }
    
    this.isDragging = true
    this.startX = event.clientX - this.el.nativeElement.offsetLeft
    this.startY = event.clientY - this.el.nativeElement.offsetTop
    this.dragStart.emit({ x: event.clientX, y: event.clientY })
  }

  @HostListener('document:mousemove', ['$event'])
  onMouseMove(event: MouseEvent) {
    if (!this.isDragging) return
    
    const left = event.clientX - this.startX
    const top = event.clientY - this.startY
    
    this.renderer.setStyle(this.el.nativeElement, 'left', `${left}px`)
    this.renderer.setStyle(this.el.nativeElement, 'top', `${top}px`)
  }

  @HostListener('document:mouseup', ['$event'])
  onMouseUp(event: MouseEvent) {
    if (this.isDragging) {
      this.isDragging = false
      this.dragEnd.emit({ x: event.clientX, y: event.clientY })
    }
  }
}

// 使用
@Component({
  standalone: true,
  template: `
    <div appDraggable 
         dragHandle=".handle"
         (dragStart)="onDragStart($event)"
         (dragEnd)="onDragEnd($event)">
      <div class="handle">拖拽手柄</div>
      <div class="content">可拖拽内容</div>
    </div>
  `
})
```

### 动画系统（Angular Animations）

```typescript
import { Component } from '@angular/core'
import { 
  trigger, 
  state, 
  style, 
  animate, 
  transition, 
  keyframes,
  group,
  query,
  stagger
} from '@angular/animations'

@Component({
  selector: 'app-animated-list',
  standalone: true,
  animations: [
    trigger('items', [
      transition(':enter', [
        style({ opacity: 0, transform: 'translateX(-100%)' }),
        animate('500ms ease-out', 
          style({ opacity: 1, transform: 'translateX(0)' })
        )
      ]),
      transition(':leave', [
        animate('300ms ease-in',
          style({ opacity: 0, transform: 'translateX(100%)' })
        )
      ])
    ]),
    
    trigger('listAnimation', [
      transition('* => *', [
        query(':enter', [
          style({ opacity: 0 }),
          stagger(100, [
            animate('400ms ease-out', style({ opacity: 1 }))
          ])
        ], { optional: true })
      ])
    ])
  ],
  template: `
    <ul [@listAnimation]="items.length">
      <li *ngFor="let item of items; trackBy: trackByFn" [@items]>
        {{ item }}
      </li>
    </ul>
  `
})
export class AnimatedListComponent {
  items = ['Item 1', 'Item 2', 'Item 3']

  trackByFn(index: number, item: string) {
    return index
  }
}
```

### 状态管理：NgRx 进阶

```typescript
import { createAction, props, createReducer, on, createFeatureSelector, createSelector } from '@ngrx/store'
import { createActionGroup, createFeature, emptyProps, on, createReducer, createSelector, createFeatureSelector, on } from '@ngrx/store'

// 1. State 接口
export interface UserState {
  users: User[]
  selectedUser: User | null
  loading: boolean
  error: string | null
}

// 2. Actions
export const UserActions = createActionGroup({
  source: 'User',
  events: {
    'Load Users': emptyProps(),
    'Load Users Success': props<{ users: User[] }>(),
    'Load Users Failure': props<{ error: string }>(),
    'Select User': props<{ userId: string }>(),
    'Add User': props<{ user: User }>(),
    'Update User': props<{ user: User }>(),
    'Delete User': props<{ userId: string }>()
  }
})

// 3. Reducer
export const initialState: UserState = {
  users: [],
  selectedUser: null,
  loading: false,
  error: null
}

export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUsers, (state) => ({
    ...state,
    loading: true,
    error: null
  })),
  on(UserActions.loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  }))
)

// 4. Selectors
export const selectUserState = createFeatureSelector<UserState>('users')

export const selectAllUsers = createSelector(
  selectUserState,
  (state) => state.users
)

export const selectSelectedUser = createSelector(
  selectUserState,
  (state) => state.selectedUser
)

export const selectLoading = createSelector(
  selectUserState,
  (state) => state.loading
)

// 5. Effects
@Injectable()
export class UserEffects {
  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => UserActions.loadUsersSuccess({ users })),
          catchError(error => of(UserActions.loadUsersFailure({ error: error.message })))
        )
      )
    )
  )

  constructor(
    private actions$: Actions,
    private userService: UserService
  ) {}
}
```

```typescript
// Component中使用NgRx
@Component({
  standalone: true,
  template: `
    <div *ngIf="loading$ | async">加载中...</div>
    <div *ngIf="error$ | async as error">{{ error }}</div>
    <ul>
      <li *ngFor="let user of users$ | async">
        {{ user.name }}
        <button (click)="selectUser(user.id)">选择</button>
      </li>
    </ul>
  `
})
export class UserListComponent {
  users$ = this.store.select(selectAllUsers)
  loading$ = this.store.select(selectLoading)
  error$ = this.store.select(selectUserState).pipe(
    map(state => state.error)
  )

  constructor(private store: Store) {}

  selectUser(userId: string) {
    this.store.dispatch(UserActions.selectUser({ userId }))
  }
}
```

### HTTP 拦截器深度使用

```typescript
import { Injectable, inject } from '@angular/core'
import { HttpInterceptor, HttpRequest, HttpHandler, HttpEvent, HttpResponse, HttpErrorResponse } from '@angular/common/http'
import { Observable, throwError, BehaviorSubject } from 'rxjs'
import { catchError, filter, take, switchMap } from 'rxjs/operators'

// 1. 认证拦截器
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const token = localStorage.getItem('token')
    
    if (token) {
      const cloned = req.clone({
        headers: req.headers.set('Authorization', `Bearer ${token}`)
      })
      return next.handle(cloned)
    }
    
    return next.handle(req)
  }
}

// 2. 错误处理拦截器
@Injectable()
export class ErrorInterceptor implements HttpInterceptor {
  private retryCount = 3
  private retryDelay = 1000

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    return next.handle(req).pipe(
      retry({
        count: this.retryCount,
        delay: (error, retryCount) => {
          if (error.status === 0 || error.status === 503) {
            return timer(this.retryDelay * retryCount)
          }
          throw error
        }
      }),
      catchError((error: HttpErrorResponse) => {
        if (error.status === 401) {
          // 处理未授权
          return this.handle401Error(req, next)
        } else if (error.status === 403) {
          // 处理禁止访问
          this.showError('您没有权限访问此资源')
        } else if (error.status >= 500) {
          // 处理服务器错误
          this.showError('服务器错误，请稍后重试')
        }
        return throwError(() => error)
      })
    )
  }

  private handle401Error(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    // 刷新 token 或重定向到登录页
    return this.authService.refreshToken().pipe(
      switchMap(token => {
        const cloned = req.clone({
          setHeaders: { Authorization: `Bearer ${token}` }
        })
        return next.handle(cloned)
      }),
      catchError(() => {
        this.authService.logout()
        return throwError(() => new Error('Token refresh failed'))
      })
    )
  }
}

// 3. 缓存拦截器
@Injectable()
export class CacheInterceptor implements HttpInterceptor {
  private cache = new Map<string, { data: HttpResponse<any>, expiry: number }>()

  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    if (req.method !== 'GET') {
      return next.handle(req)
    }

    const cached = this.cache.get(req.url)
    if (cached && cached.expiry > Date.now()) {
      return of(cached.data)
    }

    return next.handle(req).pipe(
      tap(event => {
        if (event instanceof HttpResponse) {
          this.cache.set(req.url, {
            data: event,
            expiry: Date.now() + 5 * 60 * 1000 // 5分钟缓存
          })
        }
      })
    )
  }
}
```

### 路由守卫与解析器

```typescript
import { inject } from '@angular/core'
import { Router, CanActivateFn, ActivatedRouteSnapshot, RouterStateSnapshot, ResolveFn } from '@angular/router'
import { Store } from '@ngrx/store'

// 1. 函数式守卫
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService)
  const router = inject(Router)

  if (authService.isAuthenticated()) {
    return true
  }

  router.navigate(['/login'], {
    queryParams: { returnUrl: state.url }
  })
  return false
}

// 2. 带角色检查的守卫
export const roleGuard = (allowedRoles: string[]): CanActivateFn => {
  return (route, state) => {
    const authService = inject(AuthService)
    const router = inject(Router)
    const userRole = authService.getCurrentUserRole()

    if (allowedRoles.includes(userRole)) {
      return true
    }

    router.navigate(['/unauthorized'])
    return false
  }
}

// 3. 数据解析器
export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService)
  return userService.getUserById(route.paramMap.get('id'))
}

// 4. 延迟加载守卫
export const loadGuard: CanActivateFn = async (route, state) => {
  const module = await import('./admin/admin.module')
  return true
}
```

```typescript
// 路由配置中使用守卫
export const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.routes'),
    canActivate: [authGuard, roleGuard(['admin'])],
    resolve: {
      user: userResolver
    }
  },
  {
    path: 'profile/:id',
    component: ProfileComponent,
    resolve: {
      user: userResolver
    }
  }
]
```

### 测试进阶

Jasmine + Angular Testing Library：

```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing'
import { By } from '@angular/platform-browser'
import { provideAnimations } from '@angular/platform-browser/animations'
import { UserCardComponent } from './user-card.component'
import { UserService } from '../../services/user.service'
import { of, throwError } from 'rxjs'

describe('UserCardComponent', () => {
  let component: UserCardComponent
  let fixture: ComponentFixture<UserCardComponent>
  let mockUserService: jasmine.SpyObj<UserService>

  const mockUser = {
    id: '1',
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const
  }

  beforeEach(async () => {
    mockUserService = jasmine.createSpyObj('UserService', [
      'updateUser',
      'deleteUser'
    ])
    mockUserService.updateUser.and.returnValue(of(mockUser))
    mockUserService.deleteUser.and.returnValue(of(undefined))

    await TestBed.configureTestingModule({
      imports: [UserCardComponent],
      providers: [
        { provide: UserService, useValue: mockUserService }
      ]
    }).compileComponents()

    fixture = TestBed.createComponent(UserCardComponent)
    component = fixture.componentInstance
    component.user = mockUser
    fixture.detectChanges()
  })

  it('should create', () => {
    expect(component).toBeTruthy()
  })

  it('should display user name', () => {
    const nameEl = fixture.debugElement.query(By.css('.user-name'))
    expect(nameEl.nativeElement.textContent).toContain('张三')
  })

  it('should emit edit event on edit button click', () => {
    spyOn(component.editUser, 'emit')
    
    const editButton = fixture.debugElement.query(By.css('.edit-btn'))
    editButton.nativeElement.click()
    
    expect(component.editUser.emit).toHaveBeenCalledWith(mockUser.id)
  })

  it('should call deleteUser on delete button click', async () => {
    spyOn(window, 'confirm').and.returnValue(true)
    
    const deleteButton = fixture.debugElement.query(By.css('.delete-btn'))
    deleteButton.nativeElement.click()
    
    await fixture.whenStable()
    expect(mockUserService.deleteUser).toHaveBeenCalledWith(mockUser.id)
  })

  it('should handle delete cancellation', () => {
    spyOn(window, 'confirm').and.returnValue(false)
    
    const deleteButton = fixture.debugElement.query(By.css('.delete-btn'))
    deleteButton.nativeElement.click()
    
    expect(mockUserService.deleteUser).not.toHaveBeenCalled()
  })

  it('should display role badge', () => {
    const badge = fixture.debugElement.query(By.css('.role-badge'))
    expect(badge.nativeElement.textContent).toContain('管理员')
  })
})
```

### 国际化（i18n）

Angular i18n 实现：

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  },
  {
    path: 'about',
    component: AboutComponent
  }
]
```

```html
<!-- 使用 i18n 属性 -->
<h1 i18n="@@welcome">欢迎</h1>
<p i18n="@@userGreeting">你好，{{ userName }}！</p>
<p i18n="@@userGreetingDescription">用户 {userName} 的邮箱是 {userEmail}</p>

<!-- 复数和性别 -->
<p i18n="{count, plural, =0 {没有项目} =1 {有 {count} 个项目} other {有 {count} 个项目}}">
</p>

<!-- 性别 -->
<span i18n="@@gender">
  {gender, select, male {他} female {她} other {他们}}
</span>

<!-- ICU 表达式 -->
<span i18n="@@notification">
  {newMessages, plural,
    =0 {没有新消息}
    =1 {有 1 条新消息}
    other {有 # 条新消息}
  }
</span>
```

```bash
# 构建多语言版本
ng build --localize

# 或使用 Angular CLI i18n
ng xi18n --output-path src/locale
```

### 性能优化

```typescript
// 1. Change Detection 优化
@Component({
  selector: 'app-optimized-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div *ngFor="let item of items; trackBy: trackByFn">
      {{ item.name }}
    </div>
  `
})
export class OptimizedListComponent {
  @Input() items: Item[] = []
  
  trackByFn(index: number, item: Item): string {
    return item.id
  }
}

// 2. 虚拟滚动
import { ScrollingModule } from '@angular/cdk/scrolling'

@Component({
  standalone: true,
  imports: [ScrollingModule],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items; trackBy: trackByFn" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
export class VirtualScrollComponent {
  items = Array.from({ length: 10000 }, (_, i) => ({ id: i, name: `Item ${i}` }))
  trackByFn = (i: number) => i
}

// 3. 延迟加载图片
@Directive({
  selector: 'img[lazyLoad]',
  standalone: true
})
export class LazyLoadDirective {
  @Input('lazyLoad') src!: string
  
  ngOnInit() {
    if ('IntersectionObserver' in window) {
      const observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            this.el.nativeElement.src = this.src
            observer.unobserve(this.el.nativeElement)
          }
        })
      })
      observer.observe(this.el.nativeElement)
    } else {
      this.el.nativeElement.src = this.src
    }
  }
}
```

---

## 企业级架构模式

### 微前端架构 (Micro Frontends)

微前端将微服务的理念引入前端，实现大型应用的独立开发和部署：

```typescript
// 1. Module Federation 配置 (Angular CLI + Webpack)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@https://remote.example.com/remoteEntry.js',
      },
      shared: {
        '@angular/core': { singleton: true, strictVersion: true },
        '@angular/common': { singleton: true, strictVersion: true },
        '@angular/router': { singleton: true, strictVersion: true },
      },
    }),
  ],
};
```

```typescript
// 2. Shell 应用中动态加载远程模块
import { loadRemoteModule } from '@angular-architects/module-federation';

@Component({
  selector: 'app-shell',
  template: `
    <nav>...</nav>
    <router-outlet></router-outlet>
    <div #remoteContainer></div>
  `,
})
export class ShellComponent implements OnInit {
  @ViewChild('remoteContainer', { read: ViewContainerRef }) remoteContainer!: ViewContainerRef;

  async ngOnInit() {
    try {
      const module = await loadRemoteModule({
        type: 'module',
        remoteEntry: 'https://remote.example.com/remoteEntry.js',
        exposedModule: './RemoteModule',
      });
      
      const factory = module.RemoteModuleNgFactory;
      const ref = this.remoteContainer.createComponent(factory);
    } catch (error) {
      console.error('Failed to load remote module:', error);
    }
  }
}
```

**微前端实现方案对比：**

| 方案 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|---------|------|------|----------|
| **Module Federation** | Webpack 共享 | 运行时集成，按需加载 | 依赖统一版本 | 大型团队协作 |
| **Single-SPA** | 生命周期管理 | 框架无关，成熟方案 | 需要适配层 | 多框架共存 |
| **iframe** | HTML iframe | 隔离彻底，技术栈无关 | 通信困难 | 独立子应用 |
| **Nx Monorepo** | 仓库管理 | 代码共享，CI/CD 集成 | 学习曲线 | 大型项目 |

### Angular 架构模式 (NgModules vs Standalone)

```typescript
// 1. Feature Module 模式
@NgModule({
  declarations: [UserComponent, UserListComponent, UserDetailComponent],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    UserRoutingModule,
  ],
  exports: [UserComponent],
})
export class UserModule {}

// 2. Core Module 模式 (Singleton 服务)
@Injectable({ providedIn: 'root' })
export class AuthService {
  // ...
}

@Injectable({ providedIn: 'root' })
export class LoggingService {
  // ...
}

@NgModule({
  providers: [ApiService, ConfigService],
})
export class CoreModule {}

// 3. Shared Module 模式 (可复用组件)
@NgModule({
  declarations: [ButtonComponent, ModalComponent, CardComponent],
  imports: [CommonModule, FormsModule],
  exports: [ButtonComponent, ModalComponent, CardComponent, CommonModule, FormsModule],
})
export class SharedModule {}

// 4. Shell + Microfrontend 模式
@Injectable({ providedIn: 'root' })
export class MicroFrontendService {
  private remoteConfig = {
    remoteApp: 'https://remote.example.com/remoteEntry.js',
  };

  async loadRemoteModule(config: RemoteConfig): Promise<void> {
    // 动态加载远程模块逻辑
  }
}
```

### Angular 响应式架构

```typescript
// 1. Smart/Dumb Components 模式
// Smart Component - 容器组件
@Component({
  selector: 'app-user-list-container',
  template: `
    <app-user-list
      [users]="users$ | async"
      [loading]="loading$ | async"
      [error]="error$ | async"
      (userSelected)="onUserSelected($event)"
      (userDeleted)="onUserDeleted($event)"
    ></app-user-list>
  `,
})
export class UserListContainerComponent {
  users$ = this.userService.users$;
  loading$ = this.userService.loading$;
  error$ = this.userService.error$;

  constructor(private userService: UserService) {}

  onUserSelected(userId: string) {
    this.router.navigate(['/users', userId]);
  }

  onUserDeleted(userId: string) {
    this.userService.deleteUser(userId);
  }
}

// Dumb Component - 展示组件
@Component({
  selector: 'app-user-list',
  template: `
    <div *ngIf="loading" class="loading">加载中...</div>
    <div *ngIf="error" class="error">{{ error }}</div>
    <div *ngFor="let user of users; trackBy: trackByUserId" class="user-card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="userSelected.emit(user.id)">查看详情</button>
      <button (click)="userDeleted.emit(user.id)">删除</button>
    </div>
  `,
})
export class UserListComponent {
  @Input() users: User[] = [];
  @Input() loading = false;
  @Input() error: string | null = null;
  @Output() userSelected = new EventEmitter<string>();
  @Output() userDeleted = new EventEmitter<string>();

  trackByUserId(index: number, user: User): string {
    return user.id;
  }
}
```

```typescript
// 2. 状态管理模式 (Facade Pattern)
@Injectable({ providedIn: 'root' })
export class UserStateFacade {
  // 选择器
  users$ = this.store.select(selectAllUsers);
  selectedUser$ = this.store.select(selectSelectedUser);
  loading$ = this.store.select(selectLoading);
  error$ = this.store.select(selectError);

  constructor(private store: Store) {}

  // Actions
  loadUsers() {
    this.store.dispatch(UserActions.loadUsers());
  }

  selectUser(userId: string) {
    this.store.dispatch(UserActions.selectUser({ userId }));
  }

  createUser(user: CreateUserDto) {
    this.store.dispatch(UserActions.createUser({ user }));
  }

  updateUser(user: UpdateUserDto) {
    this.store.dispatch(UserActions.updateUser({ user }));
  }

  deleteUser(userId: string) {
    this.store.dispatch(UserActions.deleteUser({ userId }));
  }
}
```

### Angular 插件系统

```typescript
// 1. Angular Library 创建
// projects/my-ui-lib/src/lib/my-button/my-button.component.ts
@Component({
  selector: 'lib-my-button',
  standalone: true,
  template: `
    <button 
      [class]="buttonClass" 
      [disabled]="disabled"
      [type]="type"
      (click)="handleClick($event)"
    >
      <ng-content></ng-content>
    </button>
  `,
})
export class MyButtonComponent {
  @Input() variant: 'primary' | 'secondary' | 'danger' = 'primary';
  @Input() size: 'sm' | 'md' | 'lg' = 'md';
  @Input() disabled = false;
  @Input() type: 'button' | 'submit' = 'button';
  @Output() clicked = new EventEmitter<MouseEvent>();

  get buttonClass() {
    return `btn btn-${this.variant} btn-${this.size}`;
  }

  handleClick(event: MouseEvent) {
    if (!this.disabled) {
      this.clicked.emit(event);
    }
  }
}

// 2. 库模块导出
@NgModule({
  declarations: [MyButtonComponent, MyCardComponent, MyModalComponent],
  imports: [CommonModule],
  exports: [MyButtonComponent, MyCardComponent, MyModalComponent],
})
export class MyUIModule {}
```

---

## CI/CD 部署与自动化

### GitHub Actions 完整配置

```yaml
# .github/workflows/angular-ci-cd.yml
name: Angular CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20.x'

jobs:
  # 代码质量检查
  lint:
    name: ESLint & Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run ESLint
        run: npm run lint
      
      - name: Check formatting
        run: npm run format:check

  # 单元测试
  test:
    name: Unit Tests (Jest)
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests with coverage
        run: npm run test:coverage
        env:
          CI: true
      
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  # E2E 测试
  e2e:
    name: E2E Tests (Playwright)
    runs-on: ubuntu-latest
    needs: test
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # 构建和部署
  build-and-deploy:
    name: Build & Deploy
    runs-on: ubuntu-latest
    needs: e2e
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://my-angular-app.example.com
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build application
        run: npm run build:production
        env:
          API_URL: ${{ secrets.PROD_API_URL }}
          SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
      
      - name: Run PageSpeed Insights
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: https://my-angular-app.example.com
          budgetPath: ./lighthouse-budget.json
      
      - name: Deploy to Firebase Hosting
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./dist/my-angular-app
          production-branch: main
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Firebase Hosting 部署配置

```json
// firebase.json
{
  "hosting": {
    "public": "dist/my-angular-app",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ],
    "headers": [
      {
        "source": "/**",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      },
      {
        "source": "/assets/**",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000, immutable"
          }
        ]
      },
      {
        "source": "/*.js",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000, immutable"
          }
        ]
      },
      {
        "source": "/*.css",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "max-age=31536000, immutable"
          }
        ]
      }
    ],
    "securityHeaders": [
      {
        "key": "X-Frame-Options",
        "value": "DENY"
      },
      {
        "key": "X-Content-Type-Options",
        "value": "nosniff"
      },
      {
        "key": "X-XSS-Protection",
        "value": "1; mode=block"
      },
      {
        "key": "Referrer-Policy",
        "value": "strict-origin-when-cross-origin"
      }
    ]
  }
}
```

### Docker 容器化部署

```dockerfile
# Dockerfile
# 多阶段构建
FROM node:20-alpine AS builder

WORKDIR /app

# 安装 Angular CLI
RUN npm install -g @angular/cli@19

# 复制 package files
COPY package*.json ./

# 安装依赖
RUN npm ci

# 复制源代码
COPY . .

# 构建应用
RUN ng build --configuration=production

# 生产镜像
FROM nginx:alpine AS runner

# 复制构建产物
COPY --from=builder /app/dist/my-angular-app /usr/share/nginx/html

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 安全配置
RUN addgroup -g 101 -S nginx && \
    adduser -S nginx -G nginx && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx

USER nginx

EXPOSE 80

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:80/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name _;
    root /usr/share/nginx/html;
    index index.html;

    # 安全 headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/json application/xml+rss;

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        access_log off;
    }

    # Angular 路由 fallback
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 健康检查
    location /health {
        access_log off;
        return 200 "OK";
    }
}
```

---

## 安全最佳实践

### XSS 防护

```typescript
// 1. Angular 自动转义
// Angular 默认会转义所有插值表达式
@Component({
  selector: 'app-safe-display',
  template: `
    <!-- 安全：自动转义 -->
    <div>{{ userInput }}</div>
    
    <!-- 危险：使用 [innerHTML] 需要清理 -->
    <div [innerHTML]="userContent"></div>
    
    <!-- 安全：使用 DomSanitizer -->
    <div [innerHTML]="sanitizedContent"></div>
  `,
})
export class SafeDisplayComponent {
  userInput = '<script>alert("xss")</script>';
  
  constructor(private sanitizer: DomSanitizer) {}
  
  get sanitizedContent() {
    // 清理 HTML 内容
    return this.sanitizer.bypassSecurityTrustHtml(
      this.sanitizeHtml(this.userContent)
    );
  }
  
  private sanitizeHtml(dirty: string): string {
    // 实现 HTML 清理逻辑
    return dirty
      .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
      .replace(/on\w+="[^"]*"/gi, '')
      .replace(/on\w+='[^']*'/gi, '');
  }
}
```

```typescript
// 2. 安全 URL
@Component({
  template: `
    <!-- 安全：Angular 自动处理 -->
    <a [href]="safeUrl">{{ linkText }}</a>
    
    <!-- 安全：邮件链接 -->
    <a [href]="'mailto:' + email">发送邮件</a>
    
    <!-- 安全：电话链接 -->
    <a [href]="'tel:' + phone">拨打电话</a>
  `,
})
export class LinkComponent {
  safeUrl = this.sanitizer.bypassSecurityTrustUrl(this.rawUrl);
  
  constructor(private sanitizer: DomSanitizer) {}
}
```

### CSRF 防护

```typescript
// 1. CSRF Token 拦截器
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';

export const csrfInterceptor: HttpInterceptorFn = (req, next) => {
  // 读取 CSRF token
  const csrfToken = getCsrfToken();
  
  // 只对 mutations 添加 token
  const mutationMethods = ['POST', 'PUT', 'PATCH', 'DELETE'];
  if (mutationMethods.includes(req.method)) {
    const cloned = req.clone({
      headers: req.headers.set('X-CSRF-Token', csrfToken),
    });
    return next(cloned);
  }
  
  return next(req);
};

// 2. 获取 CSRF token
function getCsrfToken(): string {
  const name = 'csrftoken';
  let cookieValue = '';
  if (document.cookie && document.cookie !== '') {
    const cookies = document.cookie.split(';');
    for (let i = 0; i < cookies.length; i++) {
      const cookie = cookies[i].trim();
      if (cookie.substring(0, name.length + 1) === name + '=') {
        cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
        break;
      }
    }
  }
  return cookieValue;
}

// 3. 在应用配置中注册
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([csrfInterceptor])),
  ],
};
```

### HTTP 安全 Headers

```typescript
// 1. 安全 HTTP 拦截器
import { HttpInterceptorFn } from '@angular/common/http';

export const securityHeadersInterceptor: HttpInterceptorFn = (req, next) => {
  // 验证请求来源
  if (!isValidOrigin(req.url)) {
    console.error('Invalid request origin:', req.url);
    return next(req);
  }
  
  return next(req);
};

// 2. Content Security Policy
// 在 index.html 或服务器配置中添加 CSP
```

```html
<!-- index.html -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' 'nonce-{NONCE}';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' https: data:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
">
```

### 敏感信息管理

```typescript
// 1. 环境配置
// environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000',
  appVersion: '1.0.0',
};

// environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.production.com',
  appVersion: '1.0.0',
};
```

```typescript
// 2. 环境变量验证
@Injectable({ providedIn: 'root' })
export class ConfigService {
  private config: AppConfig;

  constructor() {
    this.config = this.loadConfig();
    this.validateConfig();
  }

  private loadConfig(): AppConfig {
    return {
      apiUrl: environment.apiUrl,
      sentryDsn: (window as any).__env?.SENTRY_DSN || null,
      featureFlags: {
        enableNewFeature: environment.enableNewFeature ?? false,
      },
    };
  }

  private validateConfig(): void {
    if (!this.config.apiUrl) {
      throw new Error('API URL is required');
    }
    
    if (environment.production && !this.config.apiUrl.startsWith('https://')) {
      console.warn('Production API should use HTTPS');
    }
  }

  get<T extends keyof AppConfig>(key: T): AppConfig[T] {
    return this.config[key];
  }
}
```

---

## 可访问性 (A11y) 最佳实践

### ARIA 属性完整指南

```typescript
// 1. 按钮 vs 链接
@Component({
  selector: 'app-accessible-actions',
  template: `
    <!-- 按钮 - 执行操作 -->
    <button 
      (click)="handleSave()" 
      aria-describedby="save-description">
      保存
    </button>
    <p id="save-description">保存当前编辑的内容到服务器</p>
    
    <!-- 链接 - 导航到新页面 -->
    <a href="/dashboard" aria-current="page">
      前往仪表盘
    </a>
  `,
})
export class AccessibleActionsComponent {}

// 2. 表单错误提示
@Component({
  selector: 'app-accessible-form',
  template: `
    <form [formGroup]="form">
      <div>
        <label for="email">邮箱地址</label>
        <input
          id="email"
          formControlName="email"
          type="email"
          [attr.aria-invalid]="emailControl.invalid"
          [attr.aria-describedby]="emailControl.invalid ? 'email-error' : null"
        />
        <p *ngIf="emailControl.errors?.['required'] && emailControl.touched" 
           id="email-error" 
           role="alert">
          邮箱地址是必填项
        </p>
        <p *ngIf="emailControl.errors?.['email'] && emailControl.touched" 
           id="email-error" 
           role="alert">
          请输入有效的邮箱地址
        </p>
      </div>
    </form>
  `,
})
export class AccessibleFormComponent {
  form = new FormGroup({
    email: new FormControl('', [Validators.required, Validators.email]),
  });
  
  get emailControl() {
    return this.form.get('email')!;
  }
}

// 3. 模态对话框
@Component({
  selector: 'app-modal',
  template: `
    <div
      *ngIf="isOpen"
      role="dialog"
      aria-modal="true"
      [attr.aria-labelledby]="titleId"
      [attr.aria-describedby]="descriptionId"
      class="modal"
      tabindex="-1"
      (keydown.escape)="close()"
    >
      <h2 [id]="titleId">{{ title }}</h2>
      <p [id]="descriptionId">{{ description }}</p>
      <button (click)="close()" aria-label="关闭对话框">×</button>
      <ng-content></ng-content>
    </div>
  `,
})
export class ModalComponent {
  @Input() isOpen = false;
  @Input() title = '';
  @Input() description = '';
  @Output() closed = new EventEmitter<void>();

  titleId = `modal-title-${Math.random().toString(36).substr(2, 9)}`;
  descriptionId = `modal-desc-${Math.random().toString(36).substr(2, 9)}`;

  close() {
    this.closed.emit();
  }
}

// 4. 实时区域
@Component({
  selector: 'app-notifications',
  template: `
    <div aria-live="polite" aria-atomic="true" class="sr-only">
      {{ notification }}
    </div>
    
    <button (click)="showNotification('保存成功')">保存</button>
  `,
})
export class NotificationsComponent {
  notification = '';

  showNotification(message: string) {
    this.notification = message;
    setTimeout(() => (this.notification = ''), 3000);
  }
}

// 5. 复杂表格
@Component({
  selector: 'app-accessible-table',
  template: `
    <table>
      <caption>2024年季度销售报表</caption>
      <thead>
        <tr>
          <th scope="col" rowspan="2">产品</th>
          <th scope="colgroup" colspan="4">季度</th>
        </tr>
        <tr>
          <th scope="col">Q1</th>
          <th scope="col">Q2</th>
          <th scope="col">Q3</th>
          <th scope="col">Q4</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <th scope="row">产品A</th>
          <td>100万</td>
          <td>120万</td>
          <td>150万</td>
          <td>180万</td>
        </tr>
      </tbody>
    </table>
  `,
})
export class AccessibleTableComponent {}
```

### 键盘导航支持

```typescript
// Roving Tabindex
@Component({
  selector: 'app-accessible-menu',
  template: `
    <nav role="menubar" aria-label="主菜单">
      <button
        *ngFor="let item of menuItems; let i = index"
        role="menuitem"
        [attr.tabindex]="i === activeIndex ? 0 : -1"
        [attr.aria-haspopup]="item.hasSubmenu"
        (click)="handleSelect(item)"
        (keydown)="handleKeydown($event)"
      >
        {{ item.label }}
      </button>
    </nav>
  `,
})
export class AccessibleMenuComponent {
  menuItems = [
    { label: '首页', hasSubmenu: false },
    { label: '关于', hasSubmenu: false },
    { label: '产品', hasSubmenu: true },
    { label: '联系', hasSubmenu: false },
  ];
  
  activeIndex = 0;

  handleKeydown(event: KeyboardEvent) {
    switch (event.key) {
      case 'ArrowRight':
      case 'ArrowDown':
        event.preventDefault();
        this.activeIndex = (this.activeIndex + 1) % this.menuItems.length;
        break;
      case 'ArrowLeft':
      case 'ArrowUp':
        event.preventDefault();
        this.activeIndex = (this.activeIndex - 1 + this.menuItems.length) % this.menuItems.length;
        break;
      case 'Home':
        event.preventDefault();
        this.activeIndex = 0;
        break;
      case 'End':
        event.preventDefault();
        this.activeIndex = this.menuItems.length - 1;
        break;
      case 'Enter':
      case ' ':
        event.preventDefault();
        this.handleSelect(this.menuItems[this.activeIndex]);
        break;
    }
  }

  handleSelect(item: MenuItem) {
    console.log('Selected:', item);
  }
}
```

### 颜色对比度和视觉辅助

```scss
/* WCAG AA 标准对比度检查 */
/* 正常文本: 4.5:1 */
/* 大文本 (18px+): 3:1 */
/* UI 组件和图形: 3:1 */

/* 使用 CSS 变量管理颜色 */
:root {
  /* 主色调 */
  --color-primary: #1976d2;
  --color-primary-dark: #1565c0;
  
  /* 文本颜色 - 确保对比度 */
  --text-primary: #212121;    /* 对比度 ~15.5:1 on white */
  --text-secondary: #616161;  /* 对比度 ~7.6:1 on white */
  --text-hint: #9e9e9e;       /* 对比度 ~4.6:1 on white */
  
  /* 背景 */
  --background-light: #ffffff;
  --background-dark: #121212;
}

/* Focus 可见性 */
*:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

/* 隐藏但保持可访问 */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}

/* 高对比度模式支持 */
@media (forced-colors: active) {
  .mat-button, .mat-raised-button {
    border: 2px solid currentColor;
  }
}
```

---

## 环境配置与环境变量管理

### 多种环境配置

```typescript
// 1. Angular 环境文件
// src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000',
  enableDebug: true,
  logLevel: 'debug',
  featureFlags: {
    newDashboard: false,
    darkMode: false,
  },
};

// src/environments/environment.prod.ts
export const environment = {
  production: true,
  apiUrl: 'https://api.production.com',
  enableDebug: false,
  logLevel: 'error',
  featureFlags: {
    newDashboard: true,
    darkMode: true,
  },
};

// 2. 环境文件在构建时替换
// angular.json
{
  "configurations": {
    "production": {
      "fileReplacements": [
        {
          "replace": "src/environments/environment.ts",
          "with": "src/environments/environment.prod.ts"
        }
      ]
    }
  }
}
```

```typescript
// 3. 动态环境配置
@Injectable({ providedIn: 'root' })
export class EnvironmentService {
  private config: DynamicConfig;

  constructor() {
    this.config = this.loadConfig();
  }

  private loadConfig(): DynamicConfig {
    // 从 window 对象或 localStorage 加载配置
    const env = (window as any).__ENV__;
    
    return {
      apiUrl: env?.API_URL || '/api',
      sentryDsn: env?.SENTRY_DSN || null,
      analyticsId: env?.ANALYTICS_ID || null,
      featureFlags: {
        enableNewFeature: env?.ENABLE_NEW_FEATURE === 'true',
        maxUploadSize: parseInt(env?.MAX_UPLOAD_SIZE || '10485760', 10),
      },
    };
  }

  get<T extends keyof DynamicConfig>(key: T): DynamicConfig[T] {
    return this.config[key];
  }

  isProduction(): boolean {
    return environment.production;
  }

  isDevelopment(): boolean {
    return !environment.production;
  }
}
```

```bash
# 4. 环境变量注入
# 使用 Docker 或 CI/CD
# docker-compose.yml
services:
  app:
    environment:
      - API_URL=https://api.production.com
      - SENTRY_DSN=${SENTRY_DSN}
      - ENABLE_NEW_FEATURE=true
```

---

> [!SUCCESS]
> 本文档涵盖了 Angular 19 的核心理念、安装配置、组件系统、依赖注入、RxJS、路由、表单处理、高级模式、动画、NgRx进阶、HTTP拦截器等全方位内容。Angular 作为 TypeScript 原生的企业级框架，在 2026 年通过 Signals 和 Standalone Components 进一步现代化，为大型应用提供了完整的解决方案。
