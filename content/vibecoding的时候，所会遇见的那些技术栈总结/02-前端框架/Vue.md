# Vue - 渐进式 JavaScript 框架

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Vue 3 Composition API、响应式原理、Pinia 状态管理与实战技巧。

---

## 目录

1. [[#Vue 概述与版本演进]]
2. [[#Vue 3 Composition API]]
3. [[#响应式原理]]
4. [[#单文件组件 (SFC)]]
5. [[#Pinia 状态管理]]
6. [[#Vue Router]]
7. [[#Vue 3 新特性]]
8. [[#与 React 对比]]
9. [[#Nuxt.js]]
10. [[#实战技巧]]
11. [[#参考资料]]

---

## Vue 概述与版本演进

### 什么是 Vue

Vue 是由尤雨溪于 2014 年创建的中国开源 JavaScript 框架，核心理念是**渐进式框架**——可以从一个简单的视图层库逐步扩展为完整的企业级框架。

Vue 的设计哲学是"渐进式增强"，开发者可以根据项目需求选择性地使用 Vue 的各个部分，而不必一次性引入整套解决方案。

### 版本演进表

| 版本 | 发布年份 | 重大特性 |
|------|---------|---------|
| Vue 0.11 | 2014 | 首个公开版本 |
| Vue 1.0 | 2015 | 首个正式版本，组件系统 |
| Vue 2.0 | 2016 | 虚拟 DOM，Server-Side Rendering |
| Vue 2.7 | 2022 | Composition API 引入（Options API 兼容） |
| **Vue 3.0** | 2022 | TypeScript 重写，Composition API，Proxy 响应式 |
| Vue 3.1+ | 2022-2024 | 持续优化，性能提升 |

### Vue 3 vs Vue 2 核心对比

| 特性 | Vue 2 | Vue 3 |
|------|-------|-------|
| **响应式系统** | Object.defineProperty | **Proxy** |
| **TypeScript 支持** | 部分支持 | **完整支持** |
| **API 风格** | Options API | **Composition API + Options** |
| **打包体积** | 较大 | **更小（约 50%）** |
| **虚拟 DOM** | 传统实现 | **重写，性能更好** |
| **新特性** | 有限 | Teleport、Suspense、Fragments |
| **维护状态** | 低维护 | **活跃维护** |

> [!IMPORTANT]
> Vue 3 已成为默认版本，Vue 2 于 2023 年底停止维护。建议新项目直接使用 Vue 3。

### 核心特点

| 特点 | 说明 |
|------|------|
| **渐进式** | 可逐步引入，无需全量使用 |
| **响应式** | 数据变化自动更新视图 |
| **组件化** | 基于组件的开发模式 |
| **单文件组件** | 模板、逻辑、样式一体化 |
| **中文友好** | 文档和社区活跃度高 |
| **生态完善** | Router、状态管理、构建工具齐全 |

---

## Vue 3 Composition API

### 为什么要用 Composition API

| 问题 | Options API 解决方案 | Composition API 解决方案 |
|------|---------------------|-------------------------|
| 代码复用 | Mixins（来源不清晰） | Composables（清晰可控） |
| 逻辑组织 | 按选项类型组织 | 按功能逻辑组织 |
| 类型推导 | 困难 | **完整支持** |
| Tree-shaking | 困难 | **天然支持** |
| 大型组件 | 代码分散 | **代码内聚** |

### 基础用法

```vue
<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue'

// 响应式状态
const count = ref(0)
const message = ref('Hello Vue')

// 计算属性
const doubledCount = computed(() => count.value * 2)

// 监听器
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// 生命周期钩子
onMounted(() => {
  console.log('组件已挂载')
})

// 方法
function increment() {
  count.value++
}
</script>

<template>
  <div>
    <p>{{ message }} - {{ count }}</p>
    <p>翻倍: {{ doubledCount }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

### 响应式 API 详解

#### ref 与 reactive

```typescript
import { ref, reactive, toRefs, toRef } from 'vue'

// ref - 基础类型响应式
const count = ref(0)
console.log(count.value) // 访问值需要 .value

// ref - 对象类型（自动解包）
const obj = ref({ name: 'Vue' })
console.log(obj.value.name) // 不需要 .value.name

// reactive - 深层响应式（对象/数组）
const state = reactive({
  count: 0,
  user: {
    name: 'John',
    age: 30
  },
  tags: ['JavaScript', 'Vue']
})

// reactive 需要 toRefs 解构
const { count, user, tags } = toRefs(state)

// toRef - 创建单个响应式引用
const name = toRef(state, 'user').name
```

#### computed

```typescript
import { ref, computed } from 'vue'

const firstName = ref('John')
const lastName = ref('Doe')

// 只读计算属性
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

// 可写计算属性
const fullNameWritable = computed({
  get() {
    return `${firstName.value} ${lastName.value}`
  },
  set(value: string) {
    const [first, last] = value.split(' ')
    firstName.value = first
    lastName.value = last
  }
})

// 使用
fullNameWritable.value = 'Jane Smith'
console.log(firstName.value) // 'Jane'
```

#### watch 与 watchEffect

```typescript
import { ref, watch, watchEffect } from 'vue'

const count = ref(0)
const name = ref('Vue')

// watch - 监视特定数据源
watch(count, (newVal, oldVal) => {
  console.log(`count: ${oldVal} → ${newVal}`)
})

// watch - 监视多个数据源
watch([count, name], ([newCount, newName], [oldCount, oldName]) => {
  console.log(`变化: ${oldCount},${oldName} → ${newCount},${newName}`)
}, { immediate: true }) // 立即执行

// watch - 深度监视
const obj = ref({ nested: { count: 0 } })
watch(obj, (newVal) => {
  console.log('obj 变化了', newVal)
}, { deep: true })

// watchEffect - 自动追踪依赖
watchEffect(() => {
  // 自动追踪 count 和 name
  console.log(`${name.value} 计数: ${count.value}`)
})
```

#### 生命周期钩子

| Options API | Composition API | 执行时机 |
|------------|-----------------|---------|
| `beforeCreate` | - | 实例初始化前 |
| `created` | - | 实例创建后 |
| `beforeMount` | `onBeforeMount` | 挂载前 |
| `mounted` | `onMounted` | 挂载后 |
| `beforeUpdate` | `onBeforeUpdate` | 更新前 |
| `updated` | `onUpdated` | 更新后 |
| `beforeUnmount` | `onBeforeUnmount` | 卸载前 |
| `unmounted` | `onUnmounted` | 卸载后 |
| `errorCaptured` | `onErrorCaptured` | 错误捕获 |
| `activated` | `onActivated` | Keep-alive 激活 |
| `deactivated` | `onDeactivated` | Keep-alive 停用 |

```typescript
import { 
  onMounted, 
  onUpdated, 
  onUnmounted,
  onBeforeMount,
  onBeforeUpdate,
  onBeforeUnmount
} from 'vue'

// 组合式生命周期
onMounted(() => {
  console.log('组件挂载完成')
  // 初始化第三方库
  // 添加事件监听
})

onUnmounted(() => {
  console.log('组件卸载')
  // 清理资源
  // 移除事件监听
})
```

### Composables（可组合函数）

Composables 是 Vue 3 的代码复用模式，类似于 React 的 Hooks：

```typescript
// composables/useCounter.ts
import { ref, computed } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  
  const increment = () => count.value++
  const decrement = () => count.value--
  const reset = () => count.value = initialValue
  
  const doubled = computed(() => count.value * 2)
  
  return {
    count,
    increment,
    decrement,
    reset,
    doubled
  }
}

// 使用
<script setup>
import { useCounter } from '@/composables/useCounter'

const { count, increment, decrement } = useCounter(10)
</script>
```

```typescript
// composables/useFetch.ts
import { ref, onUnmounted } from 'vue'

export function useFetch<T>(url: string) {
  const data = ref<T | null>(null)
  const error = ref<Error | null>(null)
  const loading = ref(true)
  
  let abortController: AbortController | null = null
  
  async function fetchData() {
    abortController = new AbortController()
    loading.value = true
    
    try {
      const response = await fetch(url, {
        signal: abortController.signal
      })
      data.value = await response.json()
    } catch (e) {
      if ((e as Error).name !== 'AbortError') {
        error.value = e as Error
      }
    } finally {
      loading.value = false
    }
  }
  
  // 组件卸载时取消请求
  onUnmounted(() => {
    abortController?.abort()
  })
  
  fetchData()
  
  return { data, error, loading, refetch: fetchData }
}
```

---

## 响应式原理

### Vue 2 vs Vue 3 响应式对比

| 维度 | Vue 2 (defineProperty) | Vue 3 (Proxy) |
|------|------------------------|---------------|
| **实现方式** | 遍历对象属性 | 代理整个对象 |
| **新增属性** | 需要 Vue.set | **自动响应** |
| **删除属性** | 需要 Vue.delete | **自动响应** |
| **数组索引** | 需要 Vue.set | **自动响应** |
| **性能** | 较慢 | **更快** |
| **内存** | 额外属性开销 | 无额外开销 |
| **IE 兼容** | 支持 IE9+ | **不支持 IE** |

### Vue 3 Proxy 响应式原理

```javascript
// 简化版响应式实现
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key, receiver) {
      // 收集依赖
      track(target, key)
      return Reflect.get(target, key, receiver)
    },
    
    set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver)
      // 触发更新
      trigger(target, key)
      return result
    },
    
    deleteProperty(target, key) {
      const result = Reflect.deleteProperty(target, key)
      trigger(target, key)
      return result
    }
  })
}
```

### 依赖追踪流程

```
数据读取 (get)
    ↓
track() 收集依赖
    ↓
存储到 Dep
    ↓
数据修改 (set)
    ↓
trigger() 通知所有依赖
    ↓
重新渲染组件
```

### ref vs reactive 对比

| 特性 | ref | reactive |
|------|-----|----------|
| 适用类型 | 任意值 | 对象/数组 |
| 访问方式 | .value | 直接访问 |
| 解构丢失响应 | 会丢失 | 需要 toRefs |
| 重新赋值 | 响应式 | 不响应（替换整个对象则响应） |
| 类型支持 | 自动类型推导 | 需要泛型参数 |

---

## 单文件组件 (SFC)

### SFC 结构

```vue
<script setup lang="ts">
// 组件逻辑
import { ref, computed } from 'vue'
import ChildComponent from './ChildComponent.vue'
import type { User } from '@/types'

// Props 定义
interface Props {
  title: string
  users?: User[]
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  users: () => [],
  loading: false
})

// Emits 定义
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete', id: string): void
}>()

// 响应式数据
const inputValue = ref('')

// 方法
function handleSubmit() {
  emit('update', inputValue.value)
  inputValue.value = ''
}
</script>

<template>
  <div class="container">
    <h1>{{ title }}</h1>
    
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <ul v-else>
      <li v-for="user in users" :key="user.id">
        {{ user.name }}
        <button @click="emit('delete', user.id)">删除</button>
      </li>
    </ul>
    
    <form @submit.prevent="handleSubmit">
      <input v-model="inputValue" />
      <button type="submit">提交</button>
    </form>
    
    <ChildComponent>
      <template #header>插槽内容</template>
    </ChildComponent>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.loading {
  color: #666;
}
</style>
```

### Script Setup 语法

```vue
<script setup lang="ts">
// 自动暴露给模板的变量
const message = 'Hello'

// Props 使用 defineProps（自动暴露）
defineProps<{
  name: string
  age?: number
}>()

// Emits 使用 defineEmits
const emit = defineEmits<{
  (e: 'click', id: number): void
}>()

// 无法在模板直接访问的私有变量
const privateData = 'not exposed'
</script>

<template>
  <!-- message 可访问 -->
  <!-- name, age 可访问 -->
  <!-- emit 可调用 -->
  <!-- privateData 不可访问 -->
</template>
```

---

## Pinia 状态管理

### Pinia vs Vuex 对比

| 特性 | Pinia | Vuex |
|------|-------|------|
| **API 设计** | 现代化，直观 | 较复杂 |
| **TypeScript** | 完整支持 | 支持但繁琐 |
| **体积** | 约 1KB | 较大 |
| **模块化** | 自动模块化 | 需要手动模块化 |
| **DevTools** | 支持 | 支持 |
| **Vue 3 支持** | 原生 | 需要适配 |
| **维护状态** | 活跃 | 维护缓慢 |

### Pinia 基础使用

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref(0)
  const history = ref<number[]>([])
  
  // Getters (计算属性)
  const doubleCount = computed(() => count.value * 2)
  const canUndo = computed(() => history.value.length > 0)
  
  // Actions
  function increment() {
    history.value.push(count.value)
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  function undo() {
    if (history.value.length > 0) {
      count.value = history.value.pop()!
    }
  }
  
  return {
    // State
    count,
    history,
    // Getters
    doubleCount,
    canUndo,
    // Actions
    increment,
    decrement,
    undo
  }
})
```

```vue
<!-- components/Counter.vue -->
<script setup lang="ts">
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()
</script>

<template>
  <div>
    <p>计数: {{ counter.count }}</p>
    <p>翻倍: {{ counter.doubleCount }}</p>
    <button @click="counter.increment">+</button>
    <button @click="counter.decrement">-</button>
    <button @click="counter.undo" :disabled="!counter.canUndo">
      撤销
    </button>
  </div>
</template>
```

### Pinia Store 实战

```typescript
// stores/user.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User, LoginCredentials } from '@/types'

export const useUserStore = defineStore('user', () => {
  // State
  const user = ref<User | null>(null)
  const token = ref<string | null>(localStorage.getItem('token'))
  const loading = ref(false)
  const error = ref<string | null>(null)
  
  // Getters
  const isLoggedIn = computed(() => !!user.value && !!token.value)
  const userRole = computed(() => user.value?.role ?? 'guest')
  const isAdmin = computed(() => user.value?.role === 'admin')
  
  // Actions
  async function login(credentials: LoginCredentials) {
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch('/api/auth/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      })
      
      if (!response.ok) {
        throw new Error('登录失败')
      }
      
      const data = await response.json()
      user.value = data.user
      token.value = data.token
      localStorage.setItem('token', data.token)
      
      return true
    } catch (e) {
      error.value = (e as Error).message
      return false
    } finally {
      loading.value = false
    }
  }
  
  async function logout() {
    user.value = null
    token.value = null
    localStorage.removeItem('token')
  }
  
  async function fetchUser() {
    if (!token.value) return
    
    loading.value = true
    try {
      const response = await fetch('/api/user/me', {
        headers: { 'Authorization': `Bearer ${token.value}` }
      })
      user.value = await response.json()
    } catch (e) {
      console.error('获取用户信息失败', e)
    } finally {
      loading.value = false
    }
  }
  
  return {
    // State
    user,
    token,
    loading,
    error,
    // Getters
    isLoggedIn,
    userRole,
    isAdmin,
    // Actions
    login,
    logout,
    fetchUser
  }
})
```

---

## Vue Router

### 路由配置

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: () => import('@/views/Home.vue')
  },
  {
    path: '/about',
    name: 'About',
    component: () => import('@/views/About.vue')
  },
  {
    path: '/users/:id',
    name: 'UserProfile',
    component: () => import('@/views/UserProfile.vue'),
    props: true // 路由参数作为 props
  },
  {
    path: '/admin',
    component: () => import('@/views/Admin.vue'),
    children: [
      {
        path: 'dashboard',
        name: 'AdminDashboard',
        component: () => import('@/views/admin/Dashboard.vue')
      },
      {
        path: 'users',
        name: 'AdminUsers',
        component: () => import('@/views/admin/Users.vue')
      }
    ],
    meta: { requiresAuth: true, role: 'admin' }
  },
  {
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: () => import('@/views/NotFound.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    return { top: 0 }
  }
})

// 导航守卫
router.beforeEach((to, from, next) => {
  const isAuthenticated = !!localStorage.getItem('token')
  
  if (to.meta.requiresAuth && !isAuthenticated) {
    next({ name: 'Login', query: { redirect: to.fullPath } })
  } else {
    next()
  }
})

export default router
```

### 组件中使用路由

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 编程式导航
function goToUser(id: string) {
  router.push({ name: 'UserProfile', params: { id } })
}

// 获取路由参数
const userId = route.params.id // string
const query = route.query // { page: '1', sort: 'name' }

// 监听路由变化
import { watch } from 'vue'
watch(() => route.params.id, (newId) => {
  console.log('用户 ID 变化:', newId)
})
</script>

<template>
  <nav>
    <router-link to="/">首页</router-link>
    <router-link :to="{ name: 'About' }">关于</router-link>
    
    <!-- 编程式 -->
    <button @click="goToUser('123')">用户详情</button>
    
    <!-- 当前路由高亮 -->
    <router-link to="/admin" active-class="active-link">
      管理后台
    </router-link>
  </nav>
  
  <!-- 路由出口 -->
  <router-view v-slot="{ Component }">
    <transition name="fade" mode="out-in">
      <component :is="Component" />
    </transition>
  </router-view>
</template>

<style scoped>
.active-link {
  color: #42b983;
  font-weight: bold;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

---

## Vue 3 新特性

### Teleport

将组件渲染到 DOM 任意位置：

```vue
<script setup>
import { ref } from 'vue'

const showModal = ref(false)
</script>

<template>
  <button @click="showModal = true">打开弹窗</button>
  
  <!-- 传送到 body -->
  <Teleport to="body">
    <div v-if="showModal" class="modal-overlay" @click="showModal = false">
      <div class="modal-content" @click.stop>
        <h2>弹窗内容</h2>
        <p>这个弹窗会被传送到 body 下渲染</p>
        <button @click="showModal = false">关闭</button>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
}
</style>
```

### Suspense

异步组件加载状态：

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

// 异步组件
const AsyncUserList = defineAsyncComponent(() => 
  import('./UserList.vue')
)
</script>

<template>
  <Suspense>
    <!-- 主要内容 -->
    <template #default>
      <AsyncUserList />
    </template>
    
    <!-- Loading 状态 -->
    <template #fallback>
      <div class="loading">加载中...</div>
    </template>
  </Suspense>
</template>
```

### Fragments

组件可以有多根节点：

```vue
<!-- Vue 2: 必须包裹一个根元素 -->
<template>
  <div>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
  </div>
</template>

<!-- Vue 3: 可以有多个根节点 -->
<template>
  <header>...</header>
  <main>...</main>
  <footer>...</footer>
</template>
```

### Provide/Inject

跨层级组件通信：

```vue
<!-- 祖先组件 -->
<script setup>
import { provide, ref } from 'vue'

const theme = ref('dark')
provide('theme', theme)
provide('updateTheme', (newTheme: string) => {
  theme.value = newTheme
})
</script>

<!-- 后代组件 -->
<script setup>
import { inject } from 'vue'

const theme = inject('theme')
const updateTheme = inject('updateTheme')

// 带默认值的注入
const config = inject('config', { apiUrl: '/api' })
</script>
```

---

## 与 React 对比

### 核心对比表

| 维度 | Vue | React |
|------|-----|-------|
| **模板** | HTML 模板 + 指令 | JSX |
| **状态管理** | Pinia/Vuex | Redux/Zustand/Jotai |
| **学习曲线** | 较平缓 | 中等 |
| **TypeScript** | 支持 | 支持 |
| **社区** | 中文活跃 | 更大 |
| **灵活性** | 较灵活 | 非常灵活 |
| **性能** | 相当 | 相当 |
| **打包体积** | 较小 | 较小 |

### 选型建议

| 场景 | 推荐 | 原因 |
|------|------|------|
| 中文团队 | **Vue** | 文档和社区中文友好 |
| 需要快速上手 | **Vue** | 单文件组件直观 |
| 团队有 React 经验 | **React** | 技术栈一致 |
| 需要最大灵活性 | **React** | 更灵活 |
| 喜欢模板语法 | **Vue** | 声明式模板 |
| 喜欢 JSX | **React** | JavaScript 表达式 |

### 代码风格对比

```vue
<!-- Vue 3 -->
<template>
  <div :class="{ active: isActive }">
    <p v-if="show">{{ message }}</p>
    <ul>
      <li v-for="item in items" :key="item.id">
        {{ item.name }}
      </li>
    </ul>
    <button @click="handleClick">点击</button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const isActive = ref(true)
const message = 'Hello'
const show = computed(() => isActive.value)
const items = ref([{ id: 1, name: 'A' }])

function handleClick() {
  console.log('clicked')
}
</script>
```

```tsx
// React
import { useState, useMemo } from 'react'

function Component() {
  const [isActive, setIsActive] = useState(true)
  const [items] = useState([{ id: 1, name: 'A' }])
  
  const show = useMemo(() => isActive, [isActive])
  
  const handleClick = () => {
    console.log('clicked')
  }
  
  return (
    <div className={isActive ? 'active' : ''}>
      {show && <p>Hello</p>}
      <ul>
        {items.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
      <button onClick={handleClick}>点击</button>
    </div>
  )
}
```

---

## Nuxt.js

### Nuxt.js 概述

Nuxt.js 是 Vue 的全栈框架，提供：
- **SSR/SSG** - 服务器端渲染和静态生成
- **文件路由** - 基于文件结构的自动路由
- **自动导入** - 组件和组合式函数自动导入
- **SEO 优化** - 更好的搜索引擎可见性

### 目录结构

```
my-nuxt-app/
├── nuxt.config.ts
├── app.vue
├── pages/
│   ├── index.vue          → /
│   ├── about.vue          → /about
│   └── users/
│       ├── index.vue      → /users
│       └── [id].vue        → /users/:id
├── components/
│   └── UserCard.vue       → 自动全局注册
├── composables/
│   └── useAuth.ts         → 自动全局可用
├── layouts/
│   ├── default.vue
│   └── admin.vue
├── server/
│   └── api/
│       └── users/
│           └── index.get.ts  → /api/users
└── public/
```

### 快速入门

```bash
npx nuxi@latest init my-app
cd my-app
npm run dev
```

---

## 实战技巧

### 1. AI 辅助组件生成

```typescript
// 使用 AI 生成 Vue 组件的 prompt
const generateComponentPrompt = `
创建一个 Vue 3 用户卡片组件 UserCard：
- 使用 <script setup lang="ts">
- Props 定义：
  - user: { id: string, name: string, email: string, avatar?: string }
  - size: 'sm' | 'md' | 'lg'
  - onEdit: (id: string) => void
  - onDelete: (id: string) => void
- 功能：
  - 显示用户头像（无头像显示首字母）
  - 点击编辑按钮触发 onEdit
  - 点击删除按钮触发 onDelete
- 样式：使用 Tailwind CSS
- 包含 <style scoped>
`;
```

### 2. 性能优化

| 优化点 | 方法 |
|-------|------|
| 组件懒加载 | `defineAsyncComponent(() => import(...))` |
| 大列表虚拟滚动 | vue-virtual-scroller |
| 图片懒加载 | v-lazy |
| computed vs method | 优先使用 computed |
| 避免不必要的响应式 | 使用 markRaw/shallowRef |
| KeepAlive 缓存 | `<KeepAlive include="['Home']">` |

---

## 参考资料

### 官方资源

| 资源 | 链接 |
|------|------|
| Vue 官方文档 | https://vuejs.org |
| Vue 3 文档 | https://vuejs.org/guide |
| Nuxt.js | https://nuxt.com |
| Pinia | https://pinia.vuejs.org |
| Vue Router | https://router.vuejs.org |

### 学习资源

| 资源 | 说明 |
|------|------|
| Vue Mastery | 官方推荐的视频课程 |
| Vue School | 高质量 Vue 课程 |
| 官方博客 | https://blog.vuejs.org |

### 工具链

| 工具 | 用途 |
|------|------|
| Vite | 推荐构建工具 |
| Vue CLI | 传统脚手架 |
| Vitest | Vue 单元测试 |
| Vue Test Utils | 组件测试 |

---

> [!SUCCESS]
> Vue 3 凭借其渐进式设计、Composition API 和出色的 TypeScript 支持，已成为 2026 年最受欢迎的前端框架之一。其简洁的语法、丰富的生态和活跃的中文社区使其特别适合中国团队和追求开发效率的项目。

---

## 核心理念与设计哲学

### Vue 的设计哲学

Vue 的设计哲学强调"渐进式框架"理念，开发者可以根据项目需求逐步引入 Vue 的各个部分。

**1. 渐进式增强**

Vue 的核心理念是渐进式增强：从一个简单的视图层库逐步扩展为完整的企业级框架。

```
┌─────────────────────────────────────────────────────────────┐
│                      Vue 渐进式架构                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   视图层 ──────────────────────────────────────────► 完整框架 │
│                                                             │
│   ├─ 只用响应式系统（Reactivity）                           │
│   ├─ + 组件系统（Components）                               │
│   ├─ + 路由系统（Vue Router）                              │
│   ├─ + 状态管理（Pinia）                                   │
│   └─ + SSR/SSG（Nuxt.js）                                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**2. 响应式优先**

Vue 的响应式系统是其核心特性，数据变化自动触发视图更新。

```typescript
// Vue 3 响应式系统示例
import { ref, reactive, computed, watch } from 'vue'

// 基本类型使用 ref
const count = ref(0)
console.log(count.value) // 0

// 对象类型使用 reactive
const user = reactive({
  name: '张三',
  age: 30
})
console.log(user.name) // 张三

// 计算属性
const doubled = computed(() => count.value * 2)

// 监听变化
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// 修改值
count.value++
user.age = 31
```

**3. 组件化架构**

Vue 鼓励组件化开发，每个组件包含模板、逻辑和样式。

```vue
<!-- UserCard.vue -->
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  name: string
  email: string
  avatar?: string
  role: 'admin' | 'user' | 'guest'
}

const props = withDefaults(defineProps<Props>(), {
  avatar: '',
  role: 'user'
})

const roleLabel = computed(() => {
  const labels = {
    admin: '管理员',
    user: '普通用户',
    guest: '访客'
  }
  return labels[props.role]
})

const roleClass = computed(() => `role-${props.role}`)
</script>

<template>
  <div class="user-card">
    <img v-if="avatar" :src="avatar" :alt="name" class="avatar" />
    <div v-else class="avatar-placeholder">
      {{ name.charAt(0) }}
    </div>
    <div class="info">
      <h3>{{ name }}</h3>
      <p>{{ email }}</p>
      <span :class="roleClass">{{ roleLabel }}</span>
    </div>
  </div>
</template>

<style scoped>
.user-card {
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
</style>
```

**4. 约定优于配置**

Vue 采用约定优于配置的原则，减少样板代码，同时保持灵活性。

```typescript
// Vue 自动导入约定
// src/components/Button.vue → 组件名为 Button
// src/composables/useCounter.ts → 自动导入为 useCounter)

// 自定义配置文件
// vite.config.ts
export default defineConfig({
  vue: {
    autoImport: true
  }
})
```

### Vue 解决的问题域

| 问题域 | Vue 解决方案 | 优势 |
|--------|-------------|------|
| **渐进式采用** | 可选引入 | 降低迁移成本 |
| **响应式开发** | Proxy 响应式系统 | 自动追踪依赖 |
| **组件复用** | 单文件组件 + Composables | 代码组织清晰 |
| **性能优化** | 编译时优化 | Virtual DOM + 静态提升 |
| **类型安全** | TypeScript 原生支持 | 完整类型推导 |
| **开发体验** | Vite + HMR | 秒级热更新 |

---

## 完整安装与项目创建

### 环境准备

**Node.js 版本要求：**
- Vue 3 需要 Node.js 12+ 或 14+（推荐 18+）
- 使用 pnpm 作为包管理器

```bash
# 检查 Node.js 版本
node --version
# v20.11.0

# 安装 pnpm
npm install -g pnpm

# 验证 pnpm
pnpm --version
# 8.x.x
```

### 创建 Vue 项目的多种方式

**方式一：Vite（推荐）**

```bash
# 创建 Vue + TypeScript 项目
pnpm create vite@latest my-vue-app --template vue-ts

# 进入项目目录
cd my-vue-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev

# 构建生产版本
pnpm build

# 预览生产构建
pnpm preview
```

**方式二：Vue CLI（传统方式，不推荐新项目）**

```bash
# 全局安装 Vue CLI
npm install -g @vue/cli

# 创建项目
vue create my-vue-app

# 启动开发服务器
cd my-vue-app
npm run serve
```

**方式三：Nuxt.js（全栈框架）**

```bash
# 创建 Nuxt 项目
npx nuxi@latest init my-nuxt-app

# 进入目录
cd my-nuxt-app

# 安装依赖
pnpm install

# 启动开发服务器
pnpm dev
```

**方式四：在线体验**

- Vue 官方 playground: https://play.vuejs.org
- StackBlitz: https://stackblitz.com/@vue/quick-start

### 项目结构详解

**Vite + Vue 项目标准结构：**

```
my-vue-app/
├── public/                    # 静态资源
│   ├── favicon.ico
│   └── robots.txt
├── src/                      # 源代码目录
│   ├── assets/              # 需要处理的静态资源
│   │   ├── images/
│   │   └── styles/
│   ├── components/          # Vue 组件
│   │   ├── common/         # 通用组件
│   │   ├── layout/         # 布局组件
│   │   └── features/       # 业务组件
│   ├── composables/         # 组合式函数（Hooks）
│   │   ├── useAuth.ts
│   │   ├── useFetch.ts
│   │   └── useLocalStorage.ts
│   ├── views/              # 页面组件（路由页面）
│   │   ├── Home.vue
│   │   ├── About.vue
│   │   └── NotFound.vue
│   ├── router/             # 路由配置
│   │   └── index.ts
│   ├── stores/             # Pinia 状态管理
│   │   ├── counter.ts
│   │   └── user.ts
│   ├── services/           # API 服务
│   │   └── api.ts
│   ├── types/              # TypeScript 类型
│   │   └── index.ts
│   ├── utils/              # 工具函数
│   │   └── format.ts
│   ├── App.vue            # 根组件
│   ├── main.ts            # 入口文件
│   └── shims-vue.d.ts    # Vue 类型声明
├── index.html              # HTML 入口
├── package.json
├── tsconfig.json
├── vite.config.ts
└── .gitignore
```

**Nuxt.js 项目结构：**

```
my-nuxt-app/
├── assets/                 # 资源目录
├── components/            # 组件（自动全局导入）
│   ├── common/
│   └── features/
├── composables/           # 组合式函数（自动全局导入）
├── layouts/               # 布局
│   ├── default.vue
│   └── admin.vue
├── middleware/            # 中间件
├── pages/                 # 页面（文件系统路由）
│   ├── index.vue          # /
│   ├── about.vue          # /about
│   └── users/
│       ├── index.vue      # /users
│       └── [id].vue       # /users/:id
├── plugins/               # 插件
├── public/                # 静态资源
├── server/                # 服务端代码
│   └── api/              # API 路由
│       └── users/
│           └── index.get.ts  # GET /api/users
├── stores/               # Pinia stores
├── nuxt.config.ts
└── package.json
```

---

## 组件系统详解

### 单文件组件（SFC）

Vue SFC 是 Vue 组件的标准格式，将模板、逻辑和样式封装在一个 `.vue` 文件中。

**SFC 基本结构：**

```vue
<script setup lang="ts">
// 1. 逻辑部分
import { ref, computed, onMounted } from 'vue'

interface Props {
  title: string
  items?: string[]
  loading?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  items: () => [],
  loading: false
})

const emit = defineEmits<{
  (e: 'select', item: string): void
  (e: 'update', value: string): void
}>()

const selectedItem = ref('')

const itemCount = computed(() => props.items.length)

const handleSelect = (item: string) => {
  selectedItem.value = item
  emit('select', item)
}

onMounted(() => {
  console.log('组件已挂载')
})
</script>

<template>
  <!-- 2. 模板部分 -->
  <div class="item-list">
    <h2>{{ title }}</h2>
    <p>共 {{ itemCount }} 项</p>
    
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <ul v-else>
      <li
        v-for="item in items"
        :key="item"
        :class="{ active: selectedItem === item }"
        @click="handleSelect(item)"
      >
        {{ item }}
      </li>
    </ul>
  </div>
</template>

<style scoped>
/* 3. 样式部分 */
.item-list {
  padding: 1rem;
  border: 1px solid #e5e7eb;
  border-radius: 8px;
}

.item-list ul {
  list-style: none;
  padding: 0;
}

.item-list li {
  padding: 0.5rem;
  cursor: pointer;
  transition: background 0.2s;
}

.item-list li:hover {
  background: #f3f4f6;
}

.item-list li.active {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  color: #6b7280;
  font-style: italic;
}
</style>
```

### Props 与 Emits

**Props 定义方式：**

```vue
<script setup lang="ts">
// 方式一：类型化 Props（推荐）
interface Props {
  title: string
  count?: number
  items?: string[]
  config?: {
    theme: string
    size: 'sm' | 'md' | 'lg'
  }
}

const props = withDefaults(defineProps<Props>(), {
  count: 0,
  items: () => [],
  config: () => ({ theme: 'light', size: 'md' })
})

// 方式二：使用运行时声明
// defineProps({
//   title: String,
//   count: {
//     type: Number,
//     default: 0
//   }
// })

// 访问 Props
console.log(props.title)
</script>
```

**Emits 定义方式：**

```vue
<script setup lang="ts">
// 方式一：类型化 Emits（推荐）
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete', id: number): void
  (e: 'submit', data: FormData): void
}>()

// 触发事件
const handleClick = () => {
  emit('update', 'new value')
}

// 方式二：使用数组声明
// const emit = defineEmits(['update', 'delete'])
</script>
```

### Slots（插槽）

**基础插槽：**

```vue
<!-- BaseCard.vue -->
<template>
  <div class="card">
    <div class="card-header">
      <slot name="header">
        默认标题
      </slot>
    </div>
    <div class="card-body">
      <slot></slot>
    </div>
    <div class="card-footer">
      <slot name="footer"></slot>
    </div>
  </div>
</template>

<!-- 使用 -->
<BaseCard>
  <template #header>
    <h2>自定义标题</h2>
  </template>
  
  <p>卡片内容</p>
  
  <template #footer>
    <button>确定</button>
  </template>
</BaseCard>
```

**作用域插槽：**

```vue
<!-- DataTable.vue -->
<script setup lang="ts">
import { computed } from 'vue'

interface Props {
  data: any[]
  columns: { key: string; label: string }[]
}

const props = defineProps<Props>()

const emit = defineEmits<{
  (e: 'sort', key: string): void
}>()
</script>

<template>
  <table>
    <thead>
      <tr>
        <th
          v-for="col in columns"
          :key="col.key"
          @click="emit('sort', col.key)"
        >
          {{ col.label }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="row in data" :key="row.id">
        <td v-for="col in columns" :key="col.key">
          <slot :name="col.key" :row="row" :value="row[col.key]">
            {{ row[col.key] }}
          </slot>
        </td>
      </tr>
    </tbody>
  </table>
</template>

<!-- 使用 -->
<DataTable :data="users" :columns="columns">
  <template #name="{ row }">
    <strong>{{ row.name }}</strong>
  </template>
  <template #actions="{ row }">
    <button @click="editUser(row.id)">编辑</button>
    <button @click="deleteUser(row.id)">删除</button>
  </template>
</DataTable>
```

---

## 状态管理详解

### Pinia 完整指南

**Pinia vs Vuex 对比：**

| 特性 | Pinia | Vuex |
|------|-------|------|
| API 设计 | 现代化、直观 | 复杂 |
| TypeScript | 完整支持 | 支持但繁琐 |
| 模块化 | 自动模块化 | 需要手动模块化 |
| DevTools | 支持 | 支持 |
| 体积 | ~1KB | 较大 |
| 维护状态 | 活跃 | 缓慢 |

**Pinia Store 基础：**

```typescript
// stores/counter.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'

export const useCounterStore = defineStore('counter', () => {
  // State
  const count = ref(0)
  const history = ref<number[]>([])
  
  // Getters（计算属性）
  const doubled = computed(() => count.value * 2)
  const canUndo = computed(() => history.value.length > 0)
  const lastValue = computed(() => 
    history.value.length > 0 
      ? history.value[history.value.length - 1] 
      : null
  )
  
  // Actions
  function increment() {
    history.value.push(count.value)
    count.value++
  }
  
  function decrement() {
    count.value--
  }
  
  function incrementBy(amount: number) {
    history.value.push(count.value)
    count.value += amount
  }
  
  function undo() {
    if (history.value.length > 0) {
      count.value = history.value.pop()!
    }
  }
  
  function reset() {
    count.value = 0
    history.value = []
  }
  
  return {
    // State
    count,
    history,
    // Getters
    doubled,
    canUndo,
    lastValue,
    // Actions
    increment,
    decrement,
    incrementBy,
    undo,
    reset
  }
})
```

**在组件中使用：**

```vue
<script setup lang="ts">
import { useCounterStore } from '@/stores/counter'

const counter = useCounterStore()

// 访问 state 和 getters
console.log(counter.count)
console.log(counter.doubled)

// 调用 actions
counter.increment()
counter.incrementBy(5)
counter.undo()
</script>

<template>
  <div>
    <p>计数: {{ counter.count }}</p>
    <p>翻倍: {{ counter.doubled }}</p>
    <p>可撤销: {{ counter.canUndo ? '是' : '否' }}</p>
    
    <button @click="counter.increment">+1</button>
    <button @click="counter.decrement">-1</button>
    <button @click="counter.undo" :disabled="!counter.canUndo">撤销</button>
    <button @click="counter.reset">重置</button>
  </div>
</template>
```

**带持久化的 Store：**

```typescript
import { defineStore } from 'pinia'
import { ref, watch } from 'vue'

export const useUserStore = defineStore('user', () => {
  const user = ref<User | null>(null)
  const token = ref<string | null>(null)
  
  // 从 localStorage 恢复
  const savedToken = localStorage.getItem('token')
  if (savedToken) {
    token.value = savedToken
  }
  
  // 自动持久化
  watch(token, (newToken) => {
    if (newToken) {
      localStorage.setItem('token', newToken)
    } else {
      localStorage.removeItem('token')
    }
  })
  
  async function login(credentials: Credentials) {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    })
    
    if (!response.ok) {
      throw new Error('登录失败')
    }
    
    const data = await response.json()
    user.value = data.user
    token.value = data.token
  }
  
  function logout() {
    user.value = null
    token.value = null
  }
  
  return {
    user,
    token,
    login,
    logout
  }
})
```

---

## 路由系统详解

### Vue Router 4 完整指南

**路由配置：**

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import type { RouteRecordRaw } from 'vue-router'

// 懒加载页面组件
const Home = () => import('@/views/Home.vue')
const About = () => import('@/views/About.vue')
const UserProfile = () => import('@/views/UserProfile.vue')
const UserList = () => import('@/views/UserList.vue')
const NotFound = () => import('@/views/NotFound.vue')
const Login = () => import('@/views/Login.vue')

const routes: RouteRecordRaw[] = [
  {
    path: '/',
    name: 'Home',
    component: Home,
    meta: { title: '首页' }
  },
  {
    path: '/about',
    name: 'About',
    component: About,
    meta: { title: '关于' }
  },
  {
    path: '/login',
    name: 'Login',
    component: Login,
    meta: { title: '登录', guest: true }
  },
  {
    // 动态路由参数
    path: '/users/:id',
    name: 'UserProfile',
    component: UserProfile,
    props: true, // 将路由参数作为 props
    meta: { title: '用户资料', requiresAuth: true }
  },
  {
    // 查询参数
    path: '/search',
    name: 'Search',
    component: () => import('@/views/Search.vue'),
    props: (route) => ({
      q: route.query.q,
      page: Number(route.query.page) || 1
    })
  },
  {
    // 嵌套路由
    path: '/admin',
    component: () => import('@/views/admin/Layout.vue'),
    meta: { requiresAuth: true, role: 'admin' },
    children: [
      {
        path: '',
        redirect: '/admin/dashboard'
      },
      {
        path: 'dashboard',
        name: 'AdminDashboard',
        component: () => import('@/views/admin/Dashboard.vue'),
        meta: { title: '仪表盘' }
      },
      {
        path: 'users',
        name: 'AdminUsers',
        component: () => import('@/views/admin/Users.vue'),
        meta: { title: '用户管理' }
      }
    ]
  },
  {
    // 通配符路由（404）
    path: '/:pathMatch(.*)*',
    name: 'NotFound',
    component: NotFound,
    meta: { title: '页面未找到' }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) {
      return savedPosition
    }
    return { top: 0 }
  }
})

// 导航守卫
router.beforeEach((to, from, next) => {
  // 设置页面标题
  document.title = (to.meta.title as string) || 'Vue App'
  
  // 检查认证
  const isAuthenticated = !!localStorage.getItem('token')
  const requiresAuth = to.meta.requiresAuth
  
  if (requiresAuth && !isAuthenticated) {
    next({ 
      name: 'Login', 
      query: { redirect: to.fullPath } 
    })
  } else if (to.name === 'Login' && isAuthenticated) {
    next({ name: 'Home' })
  } else {
    next()
  }
})

export default router
```

**在组件中使用路由：**

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 编程式导航
function goToUser(id: string) {
  router.push({ name: 'UserProfile', params: { id } })
}

function goToSearch(query: string) {
  router.push({ name: 'Search', query: { q: query, page: '1' } })
}

function replaceWithHome() {
  router.replace('/')
}

// 监听路由变化
watch(() => route.params.id, (newId) => {
  if (newId) {
    console.log('用户 ID 变化:', newId)
    fetchUser(newId as string)
  }
})

// 获取路由信息
const userId = computed(() => route.params.id as string)
const query = computed(() => route.query.q as string)
const isActive = (name: string) => route.name === name
</script>

<template>
  <nav>
    <router-link to="/" :class="{ active: isActive('Home') }">
      首页
    </router-link>
    
    <router-link :to="{ name: 'About' }" active-class="active-link">
      关于
    </router-link>
    
    <button @click="goToUser('123')">用户详情</button>
    
    <!-- 保留当前查询参数 -->
    <router-link 
      :to="{ path: '/search', query: { q: 'new' } }"
      preserve-query
    >
      搜索
    </router-link>
  </nav>
  
  <!-- 路由出口 -->
  <router-view v-slot="{ Component, route }">
    <transition name="fade" mode="out-in">
      <component :is="Component" :key="route.path" />
    </transition>
  </router-view>
</template>

<style scoped>
.active {
  color: #42b983;
  font-weight: bold;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
```

---

## 表单处理详解

### v-model 双向绑定

**基础用法：**

```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

// 文本输入
const text = ref('')

// 复选框
const checked = ref(false)
const fruits = ref<string[]>([])

// 单选框
const selected = ref('option1')

// 下拉选择
const city = ref('')

// 修饰符
const lazyText = ref('')
const trimText = ref('')
const numberValue = ref(0)
</script>

<template>
  <!-- 文本输入 -->
  <input v-model="text" placeholder="输入文本" />
  <p>你输入了: {{ text }}</p>
  
  <!-- 复选框 -->
  <input type="checkbox" v-model="checked" id="agree" />
  <label for="agree">同意协议</label>
  
  <!-- 多选复选框 -->
  <div>
    <label v-for="fruit in ['苹果', '香蕉', '橙子']" :key="fruit">
      <input type="checkbox" :value="fruit" v-model="fruits" />
      {{ fruit }}
    </label>
  </div>
  <p>选择的水果: {{ fruits }}</p>
  
  <!-- 单选框 -->
  <div>
    <label>
      <input type="radio" value="option1" v-model="selected" />
      选项 1
    </label>
    <label>
      <input type="radio" value="option2" v-model="selected" />
      选项 2
    </label>
  </div>
  <p>选择: {{ selected }}</p>
  
  <!-- 下拉选择 -->
  <select v-model="city">
    <option value="">请选择城市</option>
    <option value="beijing">北京</option>
    <option value="shanghai">上海</option>
    <option value="guangzhou">广州</option>
  </select>
  <p>城市: {{ city }}</p>
  
  <!-- 修饰符 -->
  <input v-model.lazy="lazyText" placeholder="lazy 修饰符" />
  <input v-model.trim="trimText" placeholder="trim 修饰符" />
  <input v-model.number="numberValue" type="number" placeholder="number 修饰符" />
</template>
```

**自定义组件 v-model：**

```vue
<!-- Switch.vue -->
<script setup lang="ts">
interface Props {
  modelValue: boolean
  activeText?: string
  inactiveText?: string
}

const props = withDefaults(defineProps<Props>(), {
  activeText: '开启',
  inactiveText: '关闭'
})

const emit = defineEmits<{
  (e: 'update:modelValue', value: boolean): void
}>()

const toggle = () => {
  emit('update:modelValue', !props.modelValue)
}
</script>

<template>
  <button 
    class="switch"
    :class="{ active: modelValue }"
    @click="toggle"
  >
    <span class="text">{{ modelValue ? activeText : inactiveText }}</span>
  </button>
</template>

<!-- 使用 -->
<Switch v-model="isEnabled" active-text="启用" inactive-text="禁用" />
```

### 表单验证

**使用 VeeValidate：**

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useForm, useField } from 'vee-validate'
import * as yup from 'yup'

// 定义验证 schema
const schema = yup.object({
  name: yup
    .string()
    .required('请输入姓名')
    .min(2, '姓名至少2个字符'),
  email: yup
    .string()
    .required('请输入邮箱')
    .email('请输入有效的邮箱'),
  age: yup
    .number()
    .required('请输入年龄')
    .min(18, '年龄必须大于18')
    .max(100, '年龄不能超过100'),
  password: yup
    .string()
    .required('请输入密码')
    .min(6, '密码至少6个字符')
})

const { values, errors, handleSubmit, isSubmitting } = useForm({
  validationSchema: schema,
  initialValues: {
    name: '',
    email: '',
    age: undefined,
    password: ''
  }
})

const { value: name, errorMessage: nameError } = useField('name')
const { value: email, errorMessage: emailError } = useField('email')
const { value: age, errorMessage: ageError } = useField('age')
const { value: password, errorMessage: passwordError } = useField('password')

const onSubmit = handleSubmit(async (values) => {
  try {
    await fetch('/api/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(values)
    })
    console.log('注册成功')
  } catch (error) {
    console.error('注册失败', error)
  }
})
</script>

<template>
  <form @submit="onSubmit">
    <div class="form-group">
      <label for="name">姓名</label>
      <input id="name" v-model="name" type="text" />
      <span v-if="nameError" class="error">{{ nameError }}</span>
    </div>
    
    <div class="form-group">
      <label for="email">邮箱</label>
      <input id="email" v-model="email" type="email" />
      <span v-if="emailError" class="error">{{ emailError }}</span>
    </div>
    
    <div class="form-group">
      <label for="age">年龄</label>
      <input id="age" v-model="age" type="number" />
      <span v-if="ageError" class="error">{{ ageError }}</span>
    </div>
    
    <div class="form-group">
      <label for="password">密码</label>
      <input id="password" v-model="password" type="password" />
      <span v-if="passwordError" class="error">{{ passwordError }}</span>
    </div>
    
    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? '提交中...' : '注册' }}
    </button>
  </form>
</template>

<style scoped>
.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.25rem;
}

.error {
  color: #dc2626;
  font-size: 0.875rem;
}
</style>
```

---

## 样式方案详解

### Scoped CSS

```vue
<template>
  <div class="container">
    <h1 class="title">标题</h1>
    <p class="content">内容</p>
    <button class="btn">按钮</button>
  </div>
</template>

<style scoped>
/* scoped 确保样式只作用于当前组件 */
.container {
  padding: 2rem;
}

.title {
  font-size: 2rem;
  color: #1f2937;
}

.content {
  color: #6b7280;
  line-height: 1.6;
}

.btn {
  padding: 0.5rem 1rem;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;
}

/* 深度选择器 */
:deep(.external-component) {
  /* 可以穿透到子组件 */
}

/* 全局样式 */
:global(body) {
  margin: 0;
}

/* 插槽内容样式 */
:slotted(.slot-content) {
  /* 可以设置插槽内容的样式 */
}
</style>
```

### Tailwind CSS

```bash
pnpm add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

```vue
<script setup lang="ts">
// 用户卡片
interface Props {
  name: string
  email: string
  avatar?: string
  role: 'admin' | 'user' | 'guest'
}

const props = withDefaults(defineProps<Props>(), {
  avatar: ''
})

const roleStyles = {
  admin: 'bg-red-100 text-red-800',
  user: 'bg-blue-100 text-blue-800',
  guest: 'bg-gray-100 text-gray-800'
}

const roleLabels = {
  admin: '管理员',
  user: '普通用户',
  guest: '访客'
}
</script>

<template>
  <div class="flex items-center gap-4 p-4 bg-white rounded-lg shadow">
    <img
      v-if="avatar"
      :src="avatar"
      :alt="name"
      class="w-12 h-12 rounded-full"
    />
    <div
      v-else
      class="w-12 h-12 rounded-full bg-gray-200 flex items-center justify-center"
    >
      {{ name.charAt(0) }}
    </div>
    
    <div class="flex-1">
      <h3 class="font-semibold text-gray-900">{{ name }}</h3>
      <p class="text-gray-500 text-sm">{{ email }}</p>
    </div>
    
    <span
      :class="[
        'px-2 py-1 text-xs font-medium rounded-full',
        roleStyles[role]
      ]"
    >
      {{ roleLabels[role] }}
    </span>
  </div>
</template>
```

### UnoCSS

```bash
pnpm add -D unocss
```

```typescript
// vite.config.ts
import UnoCSS from 'unocss/vite'

export default defineConfig({
  plugins: [
    UnoCSS()
  ]
})
```

```vue
<template>
  <!-- UnoCSS 自动补全 -->
  <div class="flex items-center gap-4 p-4 bg-white rounded-lg shadow">
    <button class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
      按钮
    </button>
  </div>
</template>
```

---

## 性能优化详解

### 响应式系统优化

```typescript
// 避免不必要的响应式
import { shallowRef, markRaw, shallowReactive } from 'vue'

// 1. shallowRef - 浅层响应式
const state = shallowRef({
  count: 0,
  nested: { value: 1 }  // 嵌套对象不深层响应
})

// 手动触发更新
state.value = { ...state.value, count: 1 }

// 2. markRaw - 标记非响应式
const obj = { count: 0 }
const reactiveObj = reactive({
  raw: markRaw(obj)  // obj 不会被代理
})

// 3. shallowReactive - 浅层响应式
const state = shallowReactive({
  count: 0,
  nested: { value: 1 }  // nested 不是响应式的
})
```

### 组件优化

```vue
<script setup lang="ts">
import { shallowRef } from 'vue'

// 1. 使用 shallowRef 优化大列表
const items = shallowRef<Data[]>([])

async function fetchItems() {
  const data = await fetchLargeDataset()
  items.value = data // 替换整个引用，触发更新
}

// 2. v-memo 缓存模板
const list = ref([1, 2, 3, 4, 5])

function updateItem(index: number) {
  // 只有索引变化时更新
  list.value[index]++
}
</script>

<template>
  <!-- 只有 item 或 selectedId 变化时更新 -->
  <div v-for="item in list" :key="item.id" v-memo="[item.id, selectedId]">
    <Item :item="item" :selected="item.id === selectedId" />
  </div>
</template>
```

### 异步组件

```vue
<script setup lang="ts">
import { defineAsyncComponent, ref, onMounted } from 'vue'

// 基础异步组件
const AsyncUserList = defineAsyncComponent(() => 
  import('./components/UserList.vue')
)

// 带加载状态的异步组件
const AsyncDashboard = defineAsyncComponent({
  loader: () => import('./components/Dashboard.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorBoundary,
  delay: 200,      // 延迟显示 loading
  timeout: 3000    // 超时时间
})

// 条件加载
const shouldLoadHeavy = ref(false)
const HeavyChart = defineAsyncComponent({
  loader: () => import('./components/HeavyChart.vue'),
  loadingComponent: () => import('./components/ChartSkeleton.vue')
})
</script>

<template>
  <Suspense>
    <template #default>
      <AsyncDashboard />
    </template>
    <template #fallback>
      <div class="loading">加载中...</div>
    </template>
  </Suspense>
</template>
```

---

## 生命周期详解

### Vue 3 生命周期图

```
┌─────────────────────────────────────────────────────────────┐
│                      Vue 3 生命周期                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  创建阶段                                                    │
│  ├── setup() ← Composition API 入口                         │
│  ├── onBeforeCreate()                                       │
│  ├── created()                                              │
│  └── onCreated()                                            │
│                                                             │
│  挂载阶段                                                    │
│  ├── onBeforeMount()                                        │
│  ├── template/script compiled                               │
│  └── onMounted() ← DOM 已可用                               │
│                                                             │
│  更新阶段                                                    │
│  ├── onBeforeUpdate() ← 数据变化，DOM 更新前                 │
│  ├── DOM updated                                            │
│  └── onUpdated() ← DOM 更新完成                             │
│                                                             │
│  卸载阶段                                                    │
│  ├── onBeforeUnmount() ← 清理副作用                         │
│  └── onUnmounted() ← 组件已卸载                              │
│                                                             │
│  错误处理                                                    │
│  └── onErrorCaptured() ← 捕获后代组件错误                    │
│                                                             │
│  KeepAlive                                                  │
│  ├── onActivated() ← 激活                                   │
│  └── onDeactivated() ← 停用                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 生命周期使用示例

```vue
<script setup lang="ts">
import { 
  ref, 
  onMounted, 
  onUnmounted, 
  onBeforeMount,
  onUpdated,
  onBeforeUpdate,
  onBeforeUnmount
} from 'vue'

const count = ref(0)
let intervalId: number | null = null

onBeforeMount(() => {
  console.log('组件即将挂载')
})

onMounted(() => {
  console.log('组件已挂载，DOM 可用')
  
  // 启动定时器
  intervalId = window.setInterval(() => {
    count.value++
  }, 1000)
  
  // 添加事件监听
  window.addEventListener('resize', handleResize)
})

onBeforeUpdate(() => {
  console.log('组件即将更新')
})

onUpdated(() => {
  console.log('组件已更新')
})

onBeforeUnmount(() => {
  console.log('组件即将卸载')
  
  // 清理定时器
  if (intervalId) {
    clearInterval(intervalId)
    intervalId = null
  }
  
  // 移除事件监听
  window.removeEventListener('resize', handleResize)
})

onUnmounted(() => {
  console.log('组件已卸载')
})

function handleResize() {
  console.log('窗口大小变化')
}
</script>

<template>
  <div>
    <p>计数: {{ count }}</p>
  </div>
</template>
```

---

## Composables 最佳实践

### 常用 Composables

**useLocalStorage：**

```typescript
// composables/useLocalStorage.ts
import { ref, watch } from 'vue'

export function useLocalStorage<T>(
  key: string,
  defaultValue: T
) {
  const storedValue = localStorage.getItem(key)
  const data = ref<T>(
    storedValue ? JSON.parse(storedValue) : defaultValue
  )
  
  watch(
    data,
    (newValue) => {
      if (newValue === null || newValue === undefined) {
        localStorage.removeItem(key)
      } else {
        localStorage.setItem(key, JSON.stringify(newValue))
      }
    },
    { deep: true }
  )
  
  return data
}

// 使用
const theme = useLocalStorage('theme', 'light')
const userPreferences = useLocalStorage('preferences', {
  fontSize: 16,
  language: 'zh-CN'
})
```

**useFetch：**

```typescript
// composables/useFetch.ts
import { ref, computed, watchEffect, onUnmounted } from 'vue'

interface FetchState<T> {
  data: T | null
  loading: boolean
  error: Error | null
}

export function useFetch<T>(
  url: string | (() => string)
) {
  const state = ref<FetchState<T>>({
    data: null,
    loading: false,
    error: null
  })
  
  let abortController: AbortController | null = null
  
  async function execute() {
    abortController?.abort()
    abortController = new AbortController()
    
    state.value.loading = true
    state.value.error = null
    
    try {
      const finalUrl = typeof url === 'function' ? url() : url
      const response = await fetch(finalUrl, {
        signal: abortController.signal
      })
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`)
      }
      
      state.value.data = await response.json()
    } catch (e) {
      if ((e as Error).name !== 'AbortError') {
        state.value.error = e as Error
      }
    } finally {
      state.value.loading = false
    }
  }
  
  onUnmounted(() => {
    abortController?.abort()
  })
  
  return {
    ...toRefs(state.value),
    execute
  }
}

// 使用
const { data: users, loading, error, execute } = useFetch<User[]>(
  () => `/api/users?page=${currentPage.value}`
)

// 监听依赖并自动重新请求
watchEffect(() => {
  execute()
})
```

**useMediaQuery：**

```typescript
// composables/useMediaQuery.ts
import { ref, onMounted, onUnmounted } from 'vue'

export function useMediaQuery(query: string) {
  const matches = ref(false)
  let mediaQuery: MediaQueryList | null = null
  
  function updateMatches() {
    if (mediaQuery) {
      matches.value = mediaQuery.matches
    }
  }
  
  onMounted(() => {
    mediaQuery = window.matchMedia(query)
    updateMatches()
    
    mediaQuery.addEventListener('change', updateMatches)
  })
  
  onUnmounted(() => {
    mediaQuery?.removeEventListener('change', updateMatches)
  })
  
  return matches
}

// 使用
const isMobile = useMediaQuery('(max-width: 768px)')
const prefersDarkMode = useMediaQuery('(prefers-color-scheme: dark)')
const isLandscape = useMediaQuery('(orientation: landscape)')
```

---

## DevTools 与调试

### Vue DevTools 使用指南

**主要功能：**

1. **Components 面板**
   - 查看组件树
   - 检查组件 props 和 state
   - 修改组件数据（实时预览）
   - 时间旅行调试

2. **Timeline 面板**
   - 查看性能事件
   - 分析组件渲染时间
   - 追踪响应式依赖

3. **Pinia 面板**（如果使用 Pinia）
   - 查看 store 状态
   - 修改 store 数据
   - 查看 actions 历史

### 调试技巧

```typescript
// 1. Vue DevTools 标记
import { getCurrentInstance } from 'vue'

const instance = getCurrentInstance()
instance.proxy.$options.name = 'MyComponent'

// 2. 调试响应式
import { reactive, toRaw } from 'vue'

const state = reactive({ count: 0 })

// 打印原始对象（非代理）
console.log(toRaw(state))

// 3. 调试 watch
watch(count, () => {
  console.log('count changed')
}, {
  onTrigger(e) {
    debugger // 在依赖变化时进入调试
  }
})
```

---

## 学习路径与资源

### Vue 学习路线图

```
第一阶段：基础（1-2周）
├── HTML/CSS/JavaScript 基础
├── Vue 3 响应式系统
├── 模板语法
├── 条件渲染与列表
├── 事件处理
└── 表单绑定

第二阶段：进阶（2-3周）
├── Composition API
├── 组件通信
├── 插槽系统
├── Vue Router 基础
└── Pinia 状态管理

第三阶段：生态（2-3周）
├── TypeScript 集成
├── Vue Router 进阶
├── Pinia 进阶
├── 测试基础
└── Nuxt.js 入门

第四阶段：高级（持续学习）
├── Nuxt.js 进阶
├── SSR/SSG 优化
├── 性能优化
├── 微前端
└── 全栈开发
```

### 推荐学习资源

**官方资源：**
- Vue 官方文档：https://vuejs.org/guide
- Vue 3 文档（中文）：https://cn.vuejs.org/guide
- Vue Mastery：https://www.vuemastery.com
- Nuxt.js 文档：https://nuxt.com

**教程与课程：**
| 资源 | 类型 | 难度 | 链接 |
|------|------|------|------|
| Vue 官方教程 | 交互式 | 入门 | vuejs.org/tutorial |
| 官方指南 | 文档 | 全阶段 | vuejs.org/guide |
| Vue School | 视频 | 全阶段 | vueschool.io |
| Vue Mastery | 视频 | 全阶段 | vuemastery.com |

**优质博客：**
- Vue Blog：https://blog.vuejs.org
- 尤雨溪博客：https://blog.evanyou.me
- Anthony Fu Blog：https://antfu.me

---

## 高级模式与最佳实践

### Teleport 组件深度使用

Teleport 允许将组件传送到 DOM 树的其他位置：

```vue
<script setup lang="ts">
import { ref } from 'vue'

const showModal = ref(false)
const showTooltip = ref(false)
</script>

<template>
  <!-- 基础用法：传送到 body -->
  <button @click="showModal = true">打开弹窗</button>
  
  <Teleport to="body">
    <div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
      <div class="modal-content">
        <h2>模态框</h2>
        <p>这是一个传送到 body 的模态框</p>
        <button @click="showModal = false">关闭</button>
      </div>
    </div>
  </Teleport>

  <!-- 条件传送 -->
  <Teleport to="body" :disabled="!isMobile">
    <div class="mobile-menu">
      <!-- 移动端菜单内容 -->
    </div>
  </Teleport>

  <!-- 多重传送 -->
  <Teleport to="#modal-root">
    <div class="modal">Modal Content</div>
  </Teleport>
  
  <Teleport to="#overlay-root">
    <div class="overlay">Overlay Content</div>
  </Teleport>
</template>

<style scoped>
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 2rem;
  border-radius: 8px;
  max-width: 500px;
}
</style>
```

### 异步组件与 Suspense

Suspense 用于处理异步组件的加载状态：

```vue
<script setup lang="ts">
import { defineAsyncComponent, ref, computed } from 'vue'

// 基础异步组件
const AsyncUserList = defineAsyncComponent(() => 
  import('./components/UserList.vue')
)

// 带选项的异步组件
const AsyncDashboard = defineAsyncComponent({
  loader: () => import('./components/Dashboard.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorBoundary,
  delay: 200,
  timeout: 3000,
  suspensible: true
})

// 条件加载
const shouldLoadHeavy = ref(false)
const HeavyChart = defineAsyncComponent({
  loader: () => import('./components/HeavyChart.vue'),
  loadingComponent: () => import('./components/ChartSkeleton.vue')
})
</script>

<template>
  <!-- Suspense 处理 -->
  <Suspense>
    <template #default>
      <AsyncUserList />
    </template>
    
    <template #fallback>
      <div class="loading">
        <div class="spinner"></div>
        <p>加载中...</p>
      </div>
    </template>
  </Suspense>

  <!-- 条件渲染异步组件 -->
  <div v-if="shouldLoadHeavy">
    <Suspense>
      <template #default>
        <HeavyChart :data="chartData" />
      </template>
      <template #fallback>
        <ChartSkeleton />
      </template>
    </Suspense>
  </div>
</template>
```

### 依赖注入（Provide/Inject）深度使用

Provide/Inject 用于跨层级组件通信：

```vue
<script setup lang="ts">
import { provide, inject, ref, computed, readonly } from 'vue'

// 1. 基础 Provide/Inject
const theme = ref('light')
provide('theme', theme)

const childTheme = inject('theme') // 自动追踪响应式

// 2. 带默认值的 Inject
const config = inject('config', { apiUrl: '/api', version: '1.0' })

// 3. 类型安全的 Inject
interface User {
  id: string
  name: string
  email: string
}

const UserSymbol = Symbol('user')

provide(UserSymbol, {
  user: ref<User | null>(null),
  updateUser: (user: User) => {}
})

// 在子组件中使用
const userContext = inject(UserSymbol)
if (userContext) {
  console.log(userContext.user.value)
}

// 4. 只读注入
provide('readOnlyValue', readonly(theme))

// 5. 响应式注入
const userRepository = {
  user: ref<User | null>(null),
  async fetchUser(id: string) {
    this.user.value = await api.getUser(id)
  }
}
provide('userRepository', userRepository)

// 6. 祖父注入（跨多层级）
// 可以在任意层级注入和获取
</script>

<template>
  <!-- 模板中使用 -->
  <div :class="theme">
    <slot />
  </div>
</template>
```

### 动态组件与异步加载

动态组件用于根据条件渲染不同组件：

```vue
<script setup lang="ts">
import { ref, shallowRef, defineAsyncComponent } from 'vue'

// 静态组件
import HomePage from './pages/HomePage.vue'
import AboutPage from './pages/AboutPage.vue'
import ContactPage from './pages/ContactPage.vue'

// 动态组件映射
const components = {
  home: HomePage,
  about: AboutPage,
  contact: ContactPage
}

// 异步组件映射
const asyncComponents = {
  dashboard: () => import('./pages/Dashboard.vue'),
  settings: () => import('./pages/Settings.vue'),
  profile: () => import('./pages/Profile.vue')
}

const currentTab = ref('home')
const activeComponent = computed(() => components[currentTab.value])

// 使用 shallowRef 优化大型组件切换
const heavyComponent = shallowRef(null)

async function loadHeavyComponent() {
  heavyComponent.value = (await import('./HeavyComponent.vue')).default
}
</script>

<template>
  <!-- 静态组件切换 -->
  <component :is="activeComponent" />

  <!-- 异步组件 -->
  <component :is="asyncComponents.dashboard" />

  <!-- 动态标签 -->
  <component :is="'div'" class="container">
    Content
  </component>

  <!-- KeepAlive 缓存组件状态 -->
  <KeepAlive include="HomePage,AboutPage">
    <component :is="activeComponent" />
  </KeepAlive>
</template>
```

### 自定义指令

Vue 3 支持自定义指令：

```typescript
// directives/focus.ts
export const vFocus = {
  mounted: (el: HTMLElement) => {
    el.focus()
  }
}

// directives/click-outside.ts
export const vClickOutside = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    el._clickOutside = (event: MouseEvent) => {
      if (!(el === event.target || el.contains(event.target as Node))) {
        binding.value(event)
      }
    }
    document.addEventListener('click', el._clickOutside)
  },
  unmounted(el: HTMLElement) {
    document.removeEventListener('click', el._clickOutside)
  }
}

// directives/intersect.ts
export const vIntersect = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    const observer = new IntersectionObserver(
      (entries) => {
        if (entries[0].isIntersecting) {
          binding.value()
        }
      },
      { threshold: binding.arg || 0 }
    )
    observer.observe(el)
    el._observer = observer
  },
  unmounted(el: HTMLElement) {
    el._observer?.disconnect()
  }
}

// directives/loading.ts
export const vLoading = {
  mounted(el: HTMLElement, binding: DirectiveBinding) {
    if (binding.value) {
      el.classList.add('is-loading')
    }
  },
  updated(el: HTMLElement, binding: DirectiveBinding) {
    if (binding.value) {
      el.classList.add('is-loading')
    } else {
      el.classList.remove('is-loading')
    }
  }
}
```

```vue
<script setup lang="ts">
import { vFocus, vClickOutside, vIntersect, vLoading } from '@/directives'

// 注册全局指令
// app.directive('focus', vFocus)
</script>

<template>
  <!-- 自动聚焦 -->
  <input v-focus placeholder="自动聚焦" />

  <!-- 点击外部触发 -->
  <div v-click-outside="closeDropdown">
    <Dropdown />
  </div>

  <!-- 进入视口触发 -->
  <img v-intersect.5="loadImage" src="placeholder.jpg" />

  <!-- 加载状态 -->
  <div v-loading="isLoading">
    <HeavyContent />
  </div>
</template>
```

### 插件开发

创建和发布 Vue 插件：

```typescript
// plugins/my-plugin/index.ts
import type { App, Plugin } from 'vue'

// 1. 简单插件
export const MyPlugin: Plugin = {
  install(app: App) {
    app.provide('myPlugin', { message: 'Hello from plugin' })
  }
}

// 2. 带配置的插件
export interface MyPluginOptions {
  prefix?: string
  global?: boolean
}

export function createMyPlugin(options: MyPluginOptions = {}) {
  const { prefix = 'app', global = true } = options

  return {
    install(app: App) {
      if (global) {
        app.config.globalProperties.$myPrefix = prefix
      }
      
      app.provide('myPlugin', { prefix })
      
      // 注册全局组件
      app.component('MyButton', {})
      app.component('MyInput', {})
      
      // 注册全局指令
      app.directive('myfocus', {})
    }
  }
}

// 3. 组合式函数插件
export function createComposablesPlugin() {
  return {
    install(app: App) {
      // 自动注册 composables 为全局属性
      app.config.globalProperties.$useAuth = useAuth
      app.config.globalProperties.$useTheme = useTheme
    }
  }
}
```

```typescript
// main.ts
import { createApp } from 'vue'
import { createMyPlugin } from '@/plugins/my-plugin'

const app = createApp(App)

app.use(createMyPlugin({ prefix: 'myapp' }))
app.use(createComposablesPlugin())

app.mount('#app')
```

### 测试进阶

Vitest + Vue Testing Library 深度使用：

```typescript
// tests/unit/userCard.spec.ts
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { render, screen, fireEvent, waitFor } from '@testing-library/vue'
import { userEvent } from '@testing-library/user-event'
import UserCard from '@/components/UserCard.vue'
import { createTestingPinia } from '@pinia/testing'

// Mock 依赖
vi.mock('@/services/userService', () => ({
  updateUser: vi.fn().mockResolvedValue({ success: true }),
  deleteUser: vi.fn().mockResolvedValue(undefined)
}))

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: '张三',
    email: 'zhangsan@example.com',
    role: 'admin' as const,
    avatar: undefined
  }

  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('renders user information', () => {
    render(UserCard, {
      props: { user: mockUser }
    })

    expect(screen.getByText('张三')).toBeInTheDocument()
    expect(screen.getByText('zhangsan@example.com')).toBeInTheDocument()
    expect(screen.getByText('管理员')).toBeInTheDocument()
  })

  it('displays avatar or initial letter', () => {
    const { rerender } = render(UserCard, {
      props: { user: mockUser }
    })

    // 无头像时显示首字母
    expect(screen.getByText('张')).toBeInTheDocument()

    // 有头像时显示图片
    rerender({
      props: { user: { ...mockUser, avatar: '/avatar.jpg' } }
    })
    expect(screen.getByRole('img')).toBeInTheDocument()
  })

  it('emits edit event when edit button is clicked', async () => {
    const user = userEvent.setup()
    const { emitted } = render(UserCard, {
      props: { user: mockUser }
    })

    const editButton = screen.getByRole('button', { name: /编辑/i })
    await user.click(editButton)

    expect(emitted()).toHaveProperty('edit')
    expect(emitted().edit[0]).toEqual(['1'])
  })

  it('emits delete event with confirmation', async () => {
    const user = userEvent.setup()
    vi.spyOn(window, 'confirm').mockReturnValue(true)

    const { emitted } = render(UserCard, {
      props: { user: mockUser }
    })

    const deleteButton = screen.getByRole('button', { name: /删除/i })
    await user.click(deleteButton)

    expect(window.confirm).toHaveBeenCalledWith('确定要删除吗？')
    expect(emitted()).toHaveProperty('delete')
  })
})

describe('UserCard with Pinia', () => {
  it('uses store for user operations', async () => {
    const pinia = createTestingPinia({
      stubActions: false
    })

    render(UserCard, {
      props: { user: mockUser },
      global: {
        plugins: [pinia]
      }
    })

    // 测试 store 交互
  })
})
```

### 性能监控与分析

Vue 应用性能监控：

```typescript
// composables/usePerformance.ts
import { ref, onMounted, onUnmounted } from 'vue'

export interface PerformanceMetrics {
  fcp: number      // First Contentful Paint
  lcp: number      // Largest Contentful Paint
  fid: number       // First Input Delay
  cls: number       // Cumulative Layout Shift
  ttfb: number     // Time to First Byte
}

export function usePerformance() {
  const metrics = ref<PerformanceMetrics>({
    fcp: 0,
    lcp: 0,
    fid: 0,
    cls: 0,
    ttfb: 0
  })

  const isSupported = 'PerformanceObserver' in window

  onMounted(() => {
    if (!isSupported) return

    // 观察 LCP
    const lcpObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries()
      const lastEntry = entries[entries.length - 1] as LargestContentfulPaint
      metrics.value.lcp = lastEntry.startTime
    })
    lcpObserver.observe({ entryTypes: ['largest-contentful-paint'] })

    // 观察 FID
    const fidObserver = new PerformanceObserver((list) => {
      const entries = list.getEntries()
      const firstEntry = entries[0] as PerformanceEventTiming
      metrics.value.fid = firstEntry.processingStart - firstEntry.startTime
    })
    fidObserver.observe({ entryTypes: ['first-input'] })

    // 观察 CLS
    const clsObserver = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          metrics.value.cls += (entry as any).value
        }
      }
    })
    clsObserver.observe({ entryTypes: ['layout-shift'] })

    // 获取 Navigation Timing
    const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming
    if (navigation) {
      metrics.value.ttfb = navigation.responseStart - navigation.requestStart
    }
  })

  onUnmounted(() => {
    // 清理 observers
  })

  return {
    metrics,
    isSupported
  }
}
```

```vue
<script setup lang="ts">
import { usePerformance } from '@/composables/usePerformance'

const { metrics } = usePerformance()

function reportMetrics() {
  // 上报性能数据
  navigator.sendBeacon('/analytics', JSON.stringify({
    url: window.location.href,
    metrics: metrics.value,
    timestamp: Date.now()
  }))
}
</script>

<template>
  <div v-if="metrics.fcp > 0">
    <p>FCP: {{ metrics.fcp.toFixed(2) }}ms</p>
    <p>LCP: {{ metrics.lcp.toFixed(2) }}ms</p>
    <p>FID: {{ metrics.fid.toFixed(2) }}ms</p>
    <p>CLS: {{ metrics.cls.toFixed(4) }}</p>
  </div>
</template>
```

### SSR/SSG 深度理解

Nuxt.js SSR/SSG 实现：

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true,
  
  routeRules: {
    // 混合策略
    '/': { prerender: true },
    '/blog/**': { swr: 3600 }, // ISR
    '/api/**': { cache: false },
    '/admin/**': { ssr: false }, // CSR
    
    // 预渲染
    '/about': { prerender: true },
    '/contact': { prerender: true }
  },
  
  nitro: {
    prerender: {
      crawlLinks: true,
      routes: ['/sitemap.xml']
    }
  }
})
```

```vue
<script setup lang="ts">
// 服务端数据获取
const { data: users, pending, error } = await useFetch('/api/users', {
  transform: (data) => data.users,
  pick: ['users', 'total']
})

// 带缓存的数据获取
const { data: posts } = useFetch('/api/posts', {
  getCachedData: (key, nuxtApp) => nuxtApp.payload.data[key],
  transform: (response) => response.posts
})

// 客户端导航时重新获取
const { data: notifications } = useFetch('/api/notifications', {
  lazy: true, // 客户端加载
  default: () => []
})

// 动态路由
const route = useRoute()
const { data: user } = await useFetch(`/api/users/${route.params.id}`)
</script>

<template>
  <div>
    <h1>用户列表</h1>
    
    <div v-if="pending">
      <LoadingSpinner />
    </div>
    
    <div v-else-if="error">
      <ErrorMessage :error="error" />
    </div>
    
    <div v-else>
      <UserCard v-for="user in users" :key="user.id" :user="user" />
    </div>
  </div>
</template>
```

```typescript
// server/api/users/[id].get.ts
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  
  if (!id) {
    throw createError({
      statusCode: 400,
      message: 'User ID is required'
    })
  }
  
  const user = await db.user.findUnique({
    where: { id },
    include: { posts: true }
  })
  
  if (!user) {
    throw createError({
      statusCode: 404,
      message: 'User not found'
    })
  }
  
  return { user }
})
```

### 国际化（i18n）

Vue I18n 深度使用：

```typescript
// locales/en.json
{
  "common": {
    "welcome": "Welcome",
    "loading": "Loading..."
  },
  "user": {
    "greeting": "Hello, {name}!",
    "profile": "User Profile",
    "settings": "Settings"
  }
}

// locales/zh.json
{
  "common": {
    "welcome": "欢迎",
    "loading": "加载中..."
  },
  "user": {
    "greeting": "你好，{name}！",
    "profile": "用户资料",
    "settings": "设置"
  }
}
```

```typescript
// i18n/index.ts
import { createI18n } from 'vue-i18n'
import en from '@/locales/en.json'
import zh from '@/locales/zh.json'

export default createI18n({
  legacy: false,
  locale: localStorage.getItem('locale') || 'zh',
  fallbackLocale: 'en',
  messages: { en, zh },
  datetimeFormats: {
    en: {
      short: { year: 'numeric', month: 'short', day: 'numeric' },
      long: { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' }
    },
    zh: {
      short: { year: 'numeric', month: 'short', day: 'numeric' },
      long: { year: 'numeric', month: 'long', day: 'numeric', weekday: 'long' }
    }
  },
  numberFormats: {
    en: { currency: { style: 'currency', currency: 'USD' } },
    zh: { currency: { style: 'currency', currency: 'CNY' } }
  }
})
```

```vue
<script setup lang="ts">
import { useI18n } from 'vue-i18n'

const { t, locale, availableLocales, setLocale } = useI18n()

const messages = {
  welcome: computed(() => t('common.welcome')),
  greeting: computed(() => t('user.greeting', { name: '张三' })),
  formattedDate: computed(() => {
    const date = new Date()
    return new Intl.DateTimeFormat(locale.value).format(date)
  }),
  formattedCurrency: computed(() => {
    const amount = 1234.56
    return new Intl.NumberFormat(locale.value, { style: 'currency' }).format(amount)
  })
}
</script>

<template>
  <div>
    <h1>{{ messages.welcome }}</h1>
    <p>{{ messages.greeting }}</p>
    <p>{{ messages.formattedDate }}</p>
    <p>{{ messages.formattedCurrency }}</p>
    
    <select v-model="locale">
      <option v-for="l in availableLocales" :key="l" :value="l">
        {{ l }}
      </option>
    </select>
    
    <button @click="setLocale('zh')">中文</button>
    <button @click="setLocale('en')">English</button>
  </div>
</template>
```

---

## Vue 生态工具链详解

### 构建工具配置

Vite 深度配置：

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath } from 'url'

export default defineConfig({
  plugins: [vue()],
  
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
      '@components': fileURLToPath(new URL('./src/components', import.meta.url)),
      '@composables': fileURLToPath(new URL('./src/composables', import.meta.url))
    }
  },
  
  server: {
    port: 3000,
    host: true,
    proxy: {
      '/api': {
        target: 'http://localhost:5000',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  },
  
  build: {
    outDir: 'dist',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus', '@ant-design/vue']
        }
      }
    },
    chunkSizeWarningLimit: 1000
  },
  
  optimizeDeps: {
    include: ['vue', 'vue-router', 'pinia']
  }
})
```

### 代码分割与懒加载

```typescript
// router/index.ts
import { createRouter, createWebHistory } from 'vue-router'

const routes = [
  {
    path: '/',
    component: () => import('@/pages/Home.vue'),
    meta: { preload: ['About'] } // 预加载其他路由
  },
  {
    path: '/about',
    component: () => import('@/pages/About.vue')
  },
  {
    path: '/dashboard',
    component: () => import('@/pages/Dashboard.vue'),
    children: [
      {
        path: '',
        redirect: 'overview'
      },
      {
        path: 'overview',
        component: () => import('@/pages/dashboard/Overview.vue')
      },
      {
        path: 'analytics',
        component: () => import('@/pages/dashboard/Analytics.vue')
      }
    ]
  }
]

// 预加载组件
export function preloadRoute(routeName: string) {
  const route = router.getRoutes().find(r => r.name === routeName)
  if (route?.components) {
    const component = route.components.default
    if (typeof component === 'function') {
      component()
    }
  }
}
```

### 样式方案对比

| 方案 | 优点 | 缺点 | 适用场景 |
|------|------|------|----------|
| **Scoped CSS** | 原生支持，无需配置 | 全局样式处理复杂 | 中小型项目 |
| **CSS Modules** | 隔离彻底，类名可预测 | 语法稍复杂 | 大型项目 |
| **Tailwind CSS** | 极快，原子化 | 学习曲线 | 快速开发 |
| **UnoCSS** | 极快，按需生成 | 社区相对较小 | 追求性能 |
| **PostCSS** | 灵活，可扩展 | 需要配置 | 定制化需求 |

```vue
<!-- Scoped + CSS Variables -->
<template>
  <div class="card">
    <h2>Card Title</h2>
  </div>
</template>

<style scoped>
.card {
  --card-bg: white;
  --card-border: #e5e7eb;
  
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-radius: 8px;
  padding: 1rem;
}

.card :deep(h2) {
  color: #1f2937;
  font-size: 1.25rem;
}
</style>
```

---

## 企业级架构模式

### 微前端架构 (Micro Frontends)

微前端将微服务的理念引入前端，实现大型应用的独立开发和部署：

```typescript
// 1. Module Federation 配置 (Webpack 5)
const ModuleFederationPlugin = require('webpack/lib/container/ModuleFederationPlugin');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remoteApp: 'remoteApp@https://remote.example.com/remoteEntry.js',
      },
      shared: {
        vue: { singleton: true, requiredVersion: '^3.4.0' },
      },
    }),
  ],
};
```

```typescript
// 2. Vue 3 应用中动态加载远程模块
const RemoteApp = defineAsyncComponent(() => import('remoteApp/CartModule'));

// 3. Host 应用中使用
<template>
  <div>
    <Header />
    <Suspense>
      <template #default>
        <RemoteApp />
      </template>
      <template #fallback>
        <Loading />
      </template>
    </Suspense>
    <Footer />
  </div>
</template>

<script setup lang="ts">
import { defineAsyncComponent } from 'vue';
import Header from '@/components/Header.vue';
import Footer from '@/components/Footer.vue';
import Loading from '@/components/Loading.vue';

const RemoteApp = defineAsyncComponent(() => import('remoteApp/CartModule'));
</script>
```

**微前端实现方案对比：**

| 方案 | 实现方式 | 优点 | 缺点 | 适用场景 |
|------|---------|------|------|----------|
| **Module Federation** | Webpack 共享 | 运行时集成，按需加载 | 依赖统一版本 | 大型团队协作 |
| **iframe** | HTML iframe | 隔离彻底，技术栈无关 | 通信困难 | 独立子应用 |
| **single-spa** | 生命周期管理 | 框架无关，成熟方案 | 配置复杂 | 多框架集成 |
| **qiankun** | 基于 single-spa | 阿里方案，生态完善 | 配置复杂 | 国内项目 |

### Module Federation 完整示例

```typescript
// host/webpack.config.js
import { defineConfig } from 'vite';
import { ModuleFederationPlugin } from 'webpack/container/plugin';

export default defineConfig({
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        remote: 'remote@http://localhost:3001/assets/remoteEntry.js',
      },
      shared: ['vue'],
    }),
  ],
});
```

```typescript
// remote/webpack.config.js
import { defineConfig } from 'vite';
import { ModuleFederationPlugin } from 'webpack/container/plugin';

export default defineConfig({
  plugins: [
    new ModuleFederationPlugin({
      name: 'remote',
      filename: 'remoteEntry.js',
      exposes: {
        './Button': './src/components/Button.vue',
        './Modal': './src/components/Modal.vue',
      },
      shared: ['vue'],
    }),
  ],
});
```

### Composables 企业级架构

```typescript
// composables/useAsyncData.ts
import { ref, shallowRef, onMounted, onUnmounted } from 'vue';

export interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
  refresh: () => Promise<void>;
}

export function useAsyncData<T>(
  fetchFn: () => Promise<T>,
  options?: {
    immediate?: boolean;
    initialData?: T | null;
    onSuccess?: (data: T) => void;
    onError?: (error: Error) => void;
  }
): AsyncState<T> {
  const data = ref<T | null>(options?.initialData ?? null) as any;
  const loading = ref(false);
  const error = ref<Error | null>(null);
  const executed = ref(false);

  const execute = async () => {
    loading.value = true;
    error.value = null;
    
    try {
      const result = await fetchFn();
      data.value = result;
      options?.onSuccess?.(result);
      return result;
    } catch (e) {
      error.value = e as Error;
      options?.onError?.(error.value);
      throw e;
    } finally {
      loading.value = false;
    }
  };

  const refresh = async () => {
    return execute();
  };

  if (options?.immediate && !executed.value) {
    executed.value = true;
    execute();
  }

  return {
    get data() { return data.value; },
    get loading() { return loading.value; },
    get error() { return error.value; },
    execute,
    refresh,
  };
}

// 使用示例
const { data: users, loading, error, execute, refresh } = useAsyncData(
  () => fetch('/api/users').then(r => r.json()),
  { immediate: true }
);
```

```typescript
// composables/usePagination.ts
import { ref, computed, watch } from 'vue';

export interface PaginationOptions {
  page?: number;
  pageSize?: number;
  total?: number;
  onPageChange?: (page: number) => void;
}

export function usePagination(options: PaginationOptions = {}) {
  const page = ref(options.page ?? 1);
  const pageSize = ref(options.pageSize ?? 10);
  const total = ref(options.total ?? 0);
  
  const totalPages = computed(() => Math.ceil(total.value / pageSize.value));
  
  const hasNext = computed(() => page.value < totalPages.value);
  const hasPrev = computed(() => page.value > 1);
  
  const startIndex = computed(() => (page.value - 1) * pageSize.value);
  const endIndex = computed(() => Math.min(startIndex.value + pageSize.value, total.value));
  
  function goTo(targetPage: number) {
    if (targetPage < 1 || targetPage > totalPages.value) return;
    page.value = targetPage;
    options.onPageChange?.(page.value);
  }
  
  function next() {
    goTo(page.value + 1);
  }
  
  function prev() {
    goTo(page.value - 1);
  }
  
  function setPageSize(size: number) {
    pageSize.value = size;
    page.value = 1;
  }
  
  return {
    page,
    pageSize,
    total,
    totalPages,
    hasNext,
    hasPrev,
    startIndex,
    endIndex,
    goTo,
    next,
    prev,
    setPageSize,
  };
}
```

```typescript
// composables/useDebounce.ts
import { ref, watch, onUnmounted } from 'vue';

export function useDebounce<T>(value: Ref<T>, delay: number = 300) {
  const debouncedValue = ref(value.value) as Ref<T>;
  let timeout: ReturnType<typeof setTimeout>;
  
  watch(value, (newValue) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      debouncedValue.value = newValue;
    }, delay);
  });
  
  onUnmounted(() => {
    clearTimeout(timeout);
  });
  
  return debouncedValue;
}

// 立即响应 + 防抖版本
export function useDebouncedRef<T>(initialValue: T, delay: number = 300): Ref<T> {
  const value = ref(initialValue) as Ref<T>;
  const debouncedValue = ref(initialValue) as Ref<T>;
  let timeout: ReturnType<typeof setTimeout>;
  
  function update(newValue: T) {
    value.value = newValue;
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      debouncedValue.value = newValue;
    }, delay);
  }
  
  return {
    get value() { return value.value; },
    set value(newVal: T) { update(newVal); },
  } as Ref<T>;
}
```

### Vue 3 插件系统深度应用

```typescript
// plugins/logger.ts - 日志插件
import type { App, Plugin } from 'vue';

interface LoggerOptions {
  level: 'debug' | 'info' | 'warn' | 'error';
  enableServer?: boolean;
  serverEndpoint?: string;
}

const loggerPlugin: Plugin = {
  install(app: App, options: LoggerOptions = { level: 'info' }) {
    const logger = {
      debug: (...args: any[]) => {
        if (options.level === 'debug') {
          console.debug('[DEBUG]', new Date().toISOString(), ...args);
        }
      },
      info: (...args: any[]) => {
        if (['debug', 'info'].includes(options.level)) {
          console.info('[INFO]', new Date().toISOString(), ...args);
        }
      },
      warn: (...args: any[]) => {
        if (['debug', 'info', 'warn'].includes(options.level)) {
          console.warn('[WARN]', new Date().toISOString(), ...args);
        }
      },
      error: (...args: any[]) => {
        console.error('[ERROR]', new Date().toISOString(), ...args);
        if (options.enableServer && options.serverEndpoint) {
          fetch(options.serverEndpoint, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ args, timestamp: Date.now() }),
          }).catch(() => {});
        }
      },
    };

    app.config.globalProperties.$logger = logger;
    app.provide('logger', logger);
  },
};

export default loggerPlugin;

// 使用
// app.use(loggerPlugin, { level: 'debug', enableServer: true });
```

```typescript
// plugins/pinia.ts - Pinia 持久化插件
import type { PiniaPluginContext } from 'pinia';
import { watch } from 'vue';

interface PersistOptions {
  key?: string;
  storage?: Storage;
  paths?: string[];
}

function persistPlugin({ store, options }: PiniaPluginContext) {
  const persistOptions = options.persist as PersistOptions;
  if (!persistOptions) return;

  const storage = persistOptions.storage ?? localStorage;
  const key = persistOptions.key ?? `pinia-${store.$id}`;
  const paths = persistOptions.paths;

  // 初始化时恢复数据
  const savedState = storage.getItem(key);
  if (savedState) {
    try {
      store.$patch(JSON.parse(savedState));
    } catch (e) {
      storage.removeItem(key);
    }
  }

  // 状态变化时保存
  watch(
    () => store.$state,
    (state) => {
      const toSave = paths ? pick(state, paths) : state;
      storage.setItem(key, JSON.stringify(toSave));
    },
    { deep: true }
  );
}

export { persistPlugin, persistOptions };
```

---

## CI/CD 部署与自动化

### GitHub Actions 完整配置

```yaml
# .github/workflows/vue-ci-cd.yml
name: Vue 3 CI/CD

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
    name: ESLint & Type Check
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
      
      - name: Type check
        run: npm run type-check

  # 单元测试
  test:
    name: Unit Tests (Vitest)
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
      url: https://my-vue-app.example.com
    
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
        run: npm run build
        env:
          VITE_API_URL: ${{ secrets.PROD_API_URL }}
          VITE_SENTRY_DSN: ${{ secrets.SENTRY_DSN }}
      
      - name: Run PageSpeed Insights
        uses: treosh/lighthouse-ci-action@v10
        with:
          urls: https://my-vue-app.example.com
          budgetPath: ./lighthouse-budget.json
      
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./dist
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Netlify 部署配置

```toml
# netlify.toml
[build]
  command = "npm run build"
  publish = "dist"

[build.environment]
  NODE_VERSION = "20"
  NPM_VERSION = "10"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-Content-Type-Options = "nosniff"
    X-XSS-Protection = "1; mode=block"
    Referrer-Policy = "strict-origin-when-cross-origin"

[[headers]]
  for = "/sw.js"
  [headers.values]
    Cache-Control = "public, max-age=0, must-revalidate"
    Service-Worker-Allowed = "/"

[[headers]]
  for = "/assets/*"
  [headers.values]
    Cache-Control = "public, max-age=31536000, immutable"

[context.production]
  environment = { VITE_API_URL = "https://api.production.com" }

[context.develop]
  environment = { VITE_API_URL = "https://api.staging.com" }
```

### Docker 容器化部署

```dockerfile
# Dockerfile
# 多阶段构建
FROM node:20-alpine AS builder

WORKDIR /app

# 安装依赖
COPY package*.json ./
RUN npm ci

# 复制源代码
COPY . .

# 构建
RUN npm run build

# 生产镜像
FROM nginx:alpine AS runner

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 非 root 用户
RUN addgroup -g 101 -S nginx && \
    adduser -S nginx -G nginx && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /var/log/nginx && \
    touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid

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

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/json application/xml;

    # 静态资源缓存
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # SPA 路由 fallback
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

```vue
<script setup lang="ts">
import DOMPurify from 'dompurify';
import { marked } from 'marked';

// 安全渲染 Markdown
const renderMarkdown = (content: string) => {
  const html = marked.parse(content);
  return DOMPurify.sanitize(html as string, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u', 'code', 'pre', 'blockquote', 'ul', 'ol', 'li', 'h1', 'h2', 'h3', 'a'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
  });
};

// v-html 安全封装
const SafeHtml = defineComponent({
  props: {
    html: { type: String, required: true },
    options: { type: Object, default: () => ({}) },
  },
  setup(props) {
    const sanitized = computed(() => {
      return DOMPurify.sanitize(props.html, {
        ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
        ALLOWED_ATTR: ['href', 'target'],
        ...props.options,
      });
    });
    return () => h('div', { innerHTML: sanitized.value });
  },
});
</script>

<template>
  <!-- 使用安全组件 -->
  <SafeHtml :html="userContent" />
</template>
```

### CSRF 防护

```typescript
// composables/useCsrfToken.ts
import { ref } from 'vue';

const csrfToken = ref<string | null>(null);
let tokenPromise: Promise<string> | null = null;

export function useCsrfToken() {
  const fetchToken = async (): Promise<string> => {
    if (csrfToken.value) return csrfToken.value;
    
    if (!tokenPromise) {
      tokenPromise = fetch('/api/csrf-token', {
        credentials: 'include',
      })
        .then((res) => res.json())
        .then((data) => {
          csrfToken.value = data.token;
          return data.token;
        })
        .finally(() => {
          tokenPromise = null;
        });
    }
    
    return tokenPromise;
  };

  const getHeaders = async (): Promise<HeadersInit> => {
    const token = await fetchToken();
    return {
      'X-CSRF-Token': token,
      'Content-Type': 'application/json',
    };
  };

  return {
    csrfToken,
    fetchToken,
    getHeaders,
  };
}
```

```typescript
// utils/request.ts
import { useCsrfToken } from '@/composables/useCsrfToken';

const { getHeaders } = useCsrfToken();

export async function securePost<T>(url: string, data: unknown): Promise<T> {
  const headers = await getHeaders();
  const response = await fetch(url, {
    method: 'POST',
    headers,
    body: JSON.stringify(data),
    credentials: 'include',
  });
  
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  return response.json();
}
```

### 内容安全策略 (CSP)

```typescript
// nuxt.config.ts (Nuxt.js)
export default defineNuxtConfig({
  app: {
    head: {
      meta: [
        { name: 'default-src', content: "'self'" },
      ],
    },
  },
  nitro: {
    routeRules: {
      '/**': {
        headers: {
          'Content-Security-Policy': [
            "default-src 'self'",
            "script-src 'self' 'nonce-{NONCE}' 'strict-dynamic'",
            "style-src 'self' 'unsafe-inline'",
            "img-src 'self' data: https:",
            "font-src 'self' https: data:",
            "connect-src 'self' https://api.example.com",
            "frame-ancestors 'none'",
          ].join('; '),
        },
      },
    },
  },
});
```

### 敏感信息管理

```typescript
// .env 文件结构
# .env - 默认值
VITE_API_URL=http://localhost:3000
VITE_ENABLE_DEBUG=true

# .env.production - 生产环境（不提交）
VITE_API_URL=https://api.production.com
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
VITE_STRIPE_PUBLIC_KEY=pk_live_xxx

# .env.local - 本地覆盖（不提交）
VITE_API_URL=http://localhost:8080
```

```typescript
// utils/env.ts
interface ImportMetaEnv {
  readonly VITE_API_URL: string;
  readonly VITE_SENTRY_DSN: string;
  readonly VITE_STRIPE_PUBLIC_KEY: string;
}

function validateEnv(): void {
  const required = ['VITE_API_URL'];
  const missing = required.filter((key) => !(key in import.meta.env));
  
  if (missing.length > 0 && import.meta.env.PROD) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### 安全 Headers 中间件

```typescript
// server/middleware/securityHeaders.ts
export default defineEventHandler((event) => {
  const res = event.node.res;
  
  // 防止点击劫持
  res.setHeader('X-Frame-Options', 'DENY');
  
  // 防止 MIME 类型嗅探
  res.setHeader('X-Content-Type-Options', 'nosniff');
  
  // XSS 防护
  res.setHeader('X-XSS-Protection', '1; mode=block');
  
  // 引用来源策略
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // 权限策略
  res.setHeader(
    'Permissions-Policy',
    'camera=(), microphone=(), geolocation=(), payment=()'
  );
  
  // HSTS
  res.setHeader(
    'Strict-Transport-Security',
    'max-age=31536000; includeSubDomains; preload'
  );
});
```

---

## 可访问性 (A11y) 最佳实践

### ARIA 属性完整指南

```vue
<template>
  <!-- 1. 按钮 vs 链接 -->
  <div>
    <button @click="handleSave" aria-describedby="save-description">
      保存
    </button>
    <p id="save-description">保存当前编辑的内容到服务器</p>
    
    <a href="/dashboard" aria-current="page">
      前往仪表盘
    </a>
  </div>

  <!-- 2. 表单错误提示 -->
  <form @submit.prevent="handleSubmit">
    <div>
      <label for="email">邮箱地址</label>
      <input
        id="email"
        v-model="email"
        type="email"
        :aria-invalid="errors.email ? 'true' : undefined"
        :aria-describedby="errors.email ? 'email-error' : undefined"
        @blur="touchField('email')"
      />
      <p v-if="errors.email && touched.email" id="email-error" role="alert">
        {{ errors.email }}
      </p>
    </div>
  </form>

  <!-- 3. 模态对话框 -->
  <Teleport to="body">
    <div
      v-if="isOpen"
      ref="modalRef"
      role="dialog"
      aria-modal="true"
      :aria-labelledby="titleId"
      tabindex="-1"
      class="modal"
      @keydown.esc="close"
    >
      <h2 :id="titleId">{{ title }}</h2>
      <button @click="close" aria-label="关闭对话框">×</button>
      <slot />
    </div>
  </Teleport>

  <!-- 4. 实时区域 -->
  <div aria-live="polite" aria-atomic="true" class="sr-only">
    {{ notification }}
  </div>

  <!-- 5. 复杂表格 -->
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
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, nextTick } from 'vue';

const props = defineProps<{
  isOpen: boolean;
  title: string;
}>();

const emit = defineEmits<{
  (e: 'close'): void;
}>();

const modalRef = ref<HTMLElement>();
const previousFocus = ref<HTMLElement | null>(null);
const titleId = 'modal-title-' + Math.random().toString(36).substr(2, 9);
const notification = ref('');

const close = () => emit('close');

const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') {
    close();
  }
  
  // 陷阱焦点
  if (e.key === 'Tab') {
    const focusable = modalRef.value?.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );
    if (!focusable?.length) return;
    
    const first = focusable[0] as HTMLElement;
    const last = focusable[focusable.length - 1] as HTMLElement;
    
    if (e.shiftKey && document.activeElement === first) {
      e.preventDefault();
      last.focus();
    } else if (!e.shiftKey && document.activeElement === last) {
      e.preventDefault();
      first.focus();
    }
  }
};

onMounted(() => {
  if (props.isOpen) {
    previousFocus.value = document.activeElement as HTMLElement;
    nextTick(() => modalRef.value?.focus());
    document.addEventListener('keydown', handleKeydown);
  }
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown);
  previousFocus.value?.focus();
});
</script>
```

### 键盘导航支持

```vue
<template>
  <!-- Roving Tabindex -->
  <nav role="menubar" aria-label="主菜单">
    <button
      v-for="(item, index) in menuItems"
      :key="item"
      role="menuitem"
      :tabindex="index === activeIndex ? 0 : -1"
      @click="handleSelect(item)"
      @keydown="handleKeydown"
    >
      {{ item }}
    </button>
  </nav>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const menuItems = ['首页', '关于', '产品', '联系'];
const activeIndex = ref(0);

const handleKeydown = (e: KeyboardEvent) => {
  switch (e.key) {
    case 'ArrowRight':
    case 'ArrowDown':
      e.preventDefault();
      activeIndex.value = (activeIndex.value + 1) % menuItems.length;
      break;
    case 'ArrowLeft':
    case 'ArrowUp':
      e.preventDefault();
      activeIndex.value = (activeIndex.value - 1 + menuItems.length) % menuItems.length;
      break;
    case 'Home':
      e.preventDefault();
      activeIndex.value = 0;
      break;
    case 'End':
      e.preventDefault();
      activeIndex.value = menuItems.length - 1;
      break;
    case 'Enter':
    case ' ':
      e.preventDefault();
      handleSelect(menuItems[activeIndex.value]);
      break;
  }
};

const handleSelect = (item: string) => {
  console.log('Selected:', item);
};
</script>
```

### 颜色对比度和视觉辅助

```css
/* WCAG AA 标准对比度检查 */
/* 正常文本: 4.5:1 */
/* 大文本 (18px+): 3:1 */
/* UI 组件和图形: 3:1 */

/* 使用 CSS 自定义属性管理颜色 */
:root {
  /* 主色调 */
  --color-primary: #409eff;
  --color-primary-dark: #337ecc;
  
  /* 文本颜色 - 确保对比度 */
  --text-primary: #303133;
  --text-secondary: #606266;
  --text-hint: #909399;
  
  /* 背景 */
  --background-light: #ffffff;
  --background-dark: #1f1f1f;
}

/* Focus 可见性 */
:focus-visible {
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
@media (prefers-contrast: high) {
  :root {
    --text-primary: #000000;
    --text-secondary: #333333;
  }
}

@media (forced-colors: active) {
  .button {
    border: 2px solid currentColor;
  }
}
```

---

## 环境配置与环境变量管理

### 多种环境配置

```bash
# .env                 - 默认值（提交到版本控制）
# .env.local           - 本地覆盖（不提交）
# .env.[mode]          - 环境特定值
# .env.[mode].local    - 环境本地覆盖
```

```bash
# .env.development
VITE_API_URL=http://localhost:3000
VITE_ENABLE_DEBUG=true
VITE_MOCK_ENABLED=true
VITE_LOG_LEVEL=debug

# .env.production
VITE_API_URL=https://api.production.com
VITE_ENABLE_DEBUG=false
VITE_MOCK_ENABLED=false
VITE_LOG_LEVEL=error

# .env.test
VITE_API_URL=http://localhost:8080
VITE_ENABLE_DEBUG=true
VITE_MOCK_ENABLED=true
VITE_LOG_LEVEL=warn
```

```typescript
// src/config/environment.ts
interface AppConfig {
  apiUrl: string;
  sentryDsn: string | null;
  featureFlags: FeatureFlags;
  isDev: boolean;
  isProd: boolean;
}

interface FeatureFlags {
  enableNewDashboard: boolean;
  enableDarkMode: boolean;
  maxUploadSize: number;
}

export function getEnvironmentConfig(): AppConfig {
  const env = import.meta.env;
  
  return {
    apiUrl: env.VITE_API_URL || 'http://localhost:3000',
    sentryDsn: env.VITE_SENTRY_DSN || null,
    isDev: env.DEV,
    isProd: env.PROD,
    featureFlags: {
      enableNewDashboard: env.VITE_FLAG_NEW_DASHBOARD === 'true',
      enableDarkMode: env.VITE_FLAG_DARK_MODE === 'true',
      maxUploadSize: parseInt(env.VITE_MAX_UPLOAD_SIZE || '10485760', 10),
    },
  };
}
```

```typescript
// vite.config.ts - 环境变量验证
import { defineConfig, loadEnv } from 'vite';

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '');
  
  const required = ['VITE_API_URL'];
  const missing = required.filter((key) => !env[key]);
  
  if (missing.length > 0 && mode === 'production') {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
  
  return {
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
    },
  };
});
```

---

## 企业级项目结构模板

### 小型项目结构

适用于简单应用和快速原型：

```
src/
├── assets/              # 静态资源
│   ├── images/
│   └── fonts/
├── components/          # 通用组件
│   ├── Button.vue
│   ├── Card.vue
│   └── Modal.vue
├── views/              # 页面组件
│   ├── Home.vue
│   ├── About.vue
│   └── NotFound.vue
├── composables/         # 组合式函数
│   ├── useAuth.ts
│   └── useLocalStorage.ts
├── services/           # API 服务
│   └── api.ts
├── types/              # 类型定义
│   └── index.ts
├── utils/              # 工具函数
│   └── format.ts
├── router/             # 路由配置
│   └── index.ts
├── App.vue            # 根组件
├── main.ts            # 入口文件
└── shims-vue.d.ts    # Vue 类型声明
```

### 中型项目结构

适用于团队协作的中等规模应用：

```
src/
├── assets/             # 静态资源
│   ├── images/
│   ├── icons/
│   └── styles/
├── components/         # 组件库
│   ├── ui/           # 基础 UI 组件
│   │   ├── Button/
│   │   │   ├── Button.vue
│   │   │   ├── Button.vue.ts
│   │   │   └── index.ts
│   │   ├── Input/
│   │   └── Modal/
│   ├── layout/        # 布局组件
│   │   ├── Header/
│   │   ├── Footer/
│   │   ├── Sidebar/
│   │   └── Container/
│   └── common/        # 公共组件
│       ├── UserAvatar.vue
│       └── PageHeader.vue
├── features/           # 业务功能模块
│   ├── auth/          # 认证模块
│   │   ├── components/
│   │   │   ├── LoginForm.vue
│   │   │   ├── RegisterForm.vue
│   │   │   └── UserProfile.vue
│   │   ├── composables/
│   │   │   ├── useLogin.ts
│   │   │   └── useAuth.ts
│   │   ├── services/
│   │   │   └── authService.ts
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   └── views/
│   │       ├── LoginView.vue
│   │       └── RegisterView.vue
│   │
│   ├── dashboard/     # 仪表盘模块
│   │   ├── components/
│   │   ├── composables/
│   │   └── views/
│   │
│   └── settings/       # 设置模块
│       ├── components/
│       ├── composables/
│       └── views/
│
├── composables/         # 全局组合式函数
│   ├── useFetch.ts
│   ├── useLocalStorage.ts
│   ├── useDebounce.ts
│   └── useMediaQuery.ts
│
├── stores/             # Pinia 状态管理
│   ├── counter.ts
│   ├── user.ts
│   └── settings.ts
│
├── services/           # API 服务层
│   ├── apiClient.ts    # Axios 实例
│   ├── userService.ts
│   └── productService.ts
│
├── router/            # 路由配置
│   ├── index.ts
│   ├── routes.ts
│   └── guards.ts
│
├── utils/              # 工具函数
│   ├── format/
│   │   ├── formatDate.ts
│   │   └── formatCurrency.ts
│   ├── validation/
│   │   ├── email.ts
│   │   └── password.ts
│   └── helpers/
│       └── debounce.ts
│
├── types/              # 全局类型定义
│   ├── index.ts
│   ├── api.d.ts
│   └── user.d.ts
│
├── constants/           # 常量定义
│   ├── api.ts
│   └── config.ts
│
├── views/              # 页面组件
│   ├── HomeView.vue
│   ├── AboutView.vue
│   ├── DashboardView.vue
│   └── NotFoundView.vue
│
├── layouts/            # 布局组件
│   ├── DefaultLayout.vue
│   └── AdminLayout.vue
│
├── config/             # 配置文件
│   ├── env.ts
│   └── features.ts
│
├── App.vue             # 根组件
├── main.ts            # 入口文件
└── shims-vue.d.ts    # Vue 类型声明
```

### 大型企业项目结构

适用于大型团队和复杂业务系统：

```
src/
├── apps/                # 微前端应用入口
│   ├── admin/          # 管理后台应用
│   │   ├── src/
│   │   │   ├── components/
│   │   │   ├── views/
│   │   │   ├── stores/
│   │   │   ├── composables/
│   │   │   ├── router/
│   │   │   ├── layouts/
│   │   │   └── main.ts
│   │   ├── public/
│   │   ├── index.html
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   └── portal/         # 用户门户应用
│       └── ...
│
├── packages/            # 共享包（Monorepo）
│   ├── ui/             # 设计系统
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Button/
│   │   │   │   │   ├── Button.vue
│   │   │   │   │   ├── Button.vue.ts
│   │   │   │   │   ├── Button.test.ts
│   │   │   │   │   ├── Button.stories.ts
│   │   │   │   │   └── index.ts
│   │   │   │   └── ...
│   │   │   ├── theme/
│   │   │   ├── composables/
│   │   │   ├── index.ts
│   │   │   └── package.json
│   │   ├── README.md
│   │   └── tsconfig.json
│   │
│   ├── utils/          # 共享工具
│   │   └── src/
│   │       ├── format/
│   │       ├── validation/
│   │       └── index.ts
│   │
│   └── types/          # 共享类型
│       └── src/
│           ├── api.ts
│           └── index.ts
│
├── src/
│   ├── main.ts         # 应用入口
│   ├── App.vue        # 根组件
│   │
│   ├── components/     # 组件库
│   │   ├── ui/        # 基础 UI 组件
│   │   ├── layout/    # 布局组件
│   │   └── common/    # 公共组件
│   │
│   ├── features/       # 功能模块（核心组织方式）
│   │   └── [feature]/
│   │       ├── components/
│   │       │   └── [FeatureComponent]/
│   │       │       ├── [FeatureComponent].vue
│   │       │       ├── [FeatureComponent].vue.ts
│   │       │       ├── [FeatureComponent].test.ts
│   │       │       └── index.ts
│   │       │
│   │       ├── composables/
│   │       │   ├── use[Feature].ts
│   │       │   └── use[Feature]Detail.ts
│   │       │
│   │       ├── services/
│   │       │   └── [feature].service.ts
│   │       │
│   │       ├── stores/
│   │       │   └── [feature].store.ts
│   │       │
│   │       ├── types/
│   │       │   └── [feature].types.ts
│   │       │
│   │       ├── utils/
│   │       │   └── [feature].utils.ts
│   │       │
│   │       ├── constants/
│   │       │   └── [feature].constants.ts
│   │       │
│   │       ├── views/
│   │       │   ├── [Feature]ListView.vue
│   │       │   └── [Feature]DetailView.vue
│   │       │
│   │       ├── router/
│   │       │   └── [feature].routes.ts
│   │       │
│   │       └── index.ts
│   │
│   ├── composables/    # 全局组合式函数
│   │   ├── useFetch.ts
│   │   ├── useLocalStorage.ts
│   │   ├── useDebounce.ts
│   │   └── useMediaQuery.ts
│   │
│   ├── stores/        # Pinia 状态管理
│   │   ├── counter.ts
│   │   ├── user.ts
│   │   └── settings.ts
│   │
│   ├── services/       # 服务层
│   │   ├── api/       # API 客户端
│   │   │   ├── index.ts
│   │   │   ├── client.ts
│   │   │   ├── interceptors.ts
│   │   │   └── errorHandler.ts
│   │   └── modules/   # 按模块组织
│   │       ├── auth.service.ts
│   │       └── user.service.ts
│   │
│   ├── router/        # 路由配置
│   │   ├── index.ts
│   │   ├── routes.ts
│   │   ├── guards.ts
│   │   └── scrollBehavior.ts
│   │
│   ├── layouts/       # 布局组件
│   │   ├── DefaultLayout.vue
│   │   ├── AuthLayout.vue
│   │   ├── AdminLayout.vue
│   │   └── BlankLayout.vue
│   │
│   ├── views/         # 页面入口
│   │   ├── HomeView.vue
│   │   ├── AboutView.vue
│   │   └── [404].vue
│   │
│   ├── utils/         # 工具函数
│   │   ├── format/
│   │   ├── validation/
│   │   ├── crypto/
│   │   └── helpers/
│   │
│   ├── types/         # 全局类型
│   │   ├── api.ts
│   │   ├── user.ts
│   │   └── common.ts
│   │
│   ├── constants/     # 常量
│   │   ├── api.ts
│   │   ├── routes.ts
│   │   └── config.ts
│   │
│   ├── config/        # 配置
│   │   ├── index.ts
│   │   ├── env.ts
│   │   └── features.ts
│   │
│   ├── styles/        # 全局样式
│   │   ├── variables.css
│   │   ├── reset.css
│   │   ├── transition.css
│   │   └── global.css
│   │
│   └── plugins/       # Vue 插件
│       ├── router.ts
│       ├── pinia.ts
│       └── i18n.ts
│
├── public/             # 静态资源
│   ├── favicon.ico
│   ├── manifest.json
│   └── robots.txt
│
├── tests/              # 测试配置
│   ├── unit/
│   ├── e2e/
│   └── setup.ts
│
├── scripts/           # 构建脚本
│   ├── analyze.ts
│   └── generate-types.ts
│
├── docs/              # 项目文档
│
├── .storybook/        # Storybook 配置
├── .eslintrc.js       # ESLint 配置
├── .prettierrc        # Prettier 配置
├── tsconfig.json      # TypeScript 配置
├── tsconfig.node.json # Node TypeScript 配置
├── vite.config.ts     # Vite 配置
├── package.json
└── .env.example      # 环境变量示例
```

### Feature-First 项目结构

现代 Vue 项目推荐的组织方式（与 Nuxt.js 理念一致）：

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm/
│   │   │   │   ├── LoginForm.vue
│   │   │   │   ├── LoginForm.vue.ts
│   │   │   │   ├── LoginForm.test.ts
│   │   │   │   └── index.ts
│   │   │   ├── RegisterForm/
│   │   │   └── UserMenu/
│   │   │
│   │   ├── composables/
│   │   │   ├── useLogin.ts
│   │   │   ├── useLogout.ts
│   │   │   └── useAuth.ts
│   │   │
│   │   ├── services/
│   │   │   └── auth.service.ts
│   │   │
│   │   ├── stores/
│   │   │   └── auth.store.ts
│   │   │
│   │   ├── types/
│   │   │   └── auth.types.ts
│   │   │
│   │   ├── utils/
│   │   │   └── auth.utils.ts
│   │   │
│   │   ├── constants/
│   │   │   └── auth.constants.ts
│   │   │
│   │   └── index.ts
│   │
│   ├── products/
│   │   ├── components/
│   │   │   ├── ProductCard/
│   │   │   ├── ProductList/
│   │   │   └── ProductDetail/
│   │   │
│   │   ├── composables/
│   │   │   ├── useProducts.ts
│   │   │   └── useProductDetail.ts
│   │   │
│   │   ├── services/
│   │   │   └── products.service.ts
│   │   │
│   │   ├── stores/
│   │   │   └── products.store.ts
│   │   │
│   │   └── index.ts
│   │
│   └── cart/
│       ├── components/
│       ├── composables/
│       ├── stores/
│       └── index.ts
│
├── components/         # 共享组件
│   ├── ui/           # UI 基础组件
│   │   ├── Button/
│   │   ├── Input/
│   │   ├── Modal/
│   │   └── ...
│   │
│   └── layout/        # 布局组件
│       ├── Header/
│       ├── Footer/
│       └── Sidebar/
│
├── composables/       # 全局组合式函数
│   ├── useFetch.ts
│   ├── useLocalStorage.ts
│   ├── useDebounce.ts
│   └── useApi.ts
│
├── lib/               # 库代码
│   ├── api/          # API 客户端
│   │   ├── client.ts
│   │   └── errors.ts
│   │
│   ├── auth/         # 认证工具
│   └── utils/        # 通用工具
│
├── stores/            # Pinia 状态管理
│   ├── counter.ts
│   └── settings.ts
│
├── router/           # 路由配置
│   ├── index.ts
│   └── routes.ts
│
├── views/             # 页面入口
│   ├── index.vue
│   ├── about.vue
│   └── [404].vue
│
├── layouts/           # 布局
│   ├── DefaultLayout.vue
│   └── AuthLayout.vue
│
├── styles/           # 样式
│   ├── variables.css
│   └── global.css
│
├── App.vue
└── main.ts
```

### Nuxt.js 项目结构

Nuxt.js 的约定式结构：

```
my-nuxt-app/
├── .nuxt/              # Nuxt 生成文件（自动）
├── .output/            # 构建输出
├── assets/             # 需要处理的资源
│   ├── images/
│   ├── css/
│   │   ├── main.css
│   │   └── variables.css
│   └── fonts/
│
├── components/         # 组件（自动全局导入）
│   ├── ui/            # UI 组件
│   │   ├── Button/
│   │   │   ├── Button.vue
│   │   │   └── index.ts
│   │   └── ...
│   │
│   ├── layout/        # 布局组件
│   ├── common/        # 公共组件
│   └── features/      # 功能组件
│       ├── auth/
│       └── dashboard/
│
├── composables/        # 组合式函数（自动全局导入）
│   ├── useAuth.ts
│   ├── useFetch.ts
│   └── useLocalStorage.ts
│
├── content/            # 内容（Nuxt Content 模块）
│   ├── blog/
│   │   ├── getting-started.md
│   │   └── advanced-topics.md
│   └── docs/
│
├── layouts/            # 布局
│   ├── default.vue
│   ├── auth.vue
│   ├── admin.vue
│   └── blank.vue
│
├── middleware/          # 中间件
│   ├── auth.ts        # 认证中间件
│   ├── admin.ts       # 管理员中间件
│   └── analytics.ts   # 分析中间件
│
├── pages/              # 页面（文件系统路由）
│   ├── index.vue      # → /
│   ├── about.vue      # → /about
│   │
│   ├── auth/
│   │   ├── login.vue      # → /auth/login
│   │   └── register.vue  # → /auth/register
│   │
│   ├── users/
│   │   ├── index.vue     # → /users
│   │   ├── [id].vue      # → /users/:id
│   │   └── [id]/
│   │       ├── index.vue  # → /users/:id
│   │       └── edit.vue  # → /users/:id/edit
│   │
│   └── [...slug].vue  # → /:slug(.*)*
│
├── plugins/            # 插件
│   ├── auth.ts        # 认证插件
│   ├── analytics.ts   # 分析插件
│   └── init.ts        # 初始化插件
│
├── public/            # 静态资源（直接复制）
│   ├── favicon.ico
│   ├── robots.txt
│   └── og-image.png
│
├── server/             # 服务端代码
│   ├── api/           # API 路由
│   │   ├── users/
│   │   │   ├── index.get.ts    # GET /api/users
│   │   │   ├── index.post.ts   # POST /api/users
│   │   │   └── [id].get.ts    # GET /api/users/:id
│   │   └── auth/
│   │       ├── login.post.ts
│   │       └── logout.post.ts
│   │
│   ├── middleware/     # 服务端中间件
│   │   ├── auth.ts
│   │   └── cors.ts
│   │
│   ├── utils/         # 服务端工具
│   │   ├── db.ts
│   │   └── jwt.ts
│   │
│   └── plugins/       # 服务端插件
│
├── stores/             # Pinia 状态管理
│   ├── user.ts
│   ├── cart.ts
│   └── settings.ts
│
├── types/              # 类型定义
│   ├── index.ts
│   └── api.d.ts
│
├── utils/              # 工具函数
│   ├── format.ts
│   └── validation.ts
│
├── app.vue            # 应用根组件
├── nuxt.config.ts     # Nuxt 配置
├── package.json
└── tsconfig.json
```

### 命名规范与最佳实践

**组件文件命名：**
```typescript
// ✅ 推荐：PascalCase
UserCard.vue
ProductList.vue
DashboardLayout.vue

// ✅ 子组件使用父组件前缀
UserCardHeader.vue
UserCardBody.vue
UserCardFooter.vue

// ❌ 避免：混合命名
userCard.vue      // 应该用 PascalCase
ProductListView.vue  // 不必要的 View 后缀
```

**目录命名：**
```typescript
// ✅ 推荐：kebab-case
components/
feature-components/
ui-components/
composables/

// ❌ 避免：驼峰或 PascalCase
Components/      // 不要用 PascalCase
featureComponents/  // 不要用 camelCase
```

**测试文件组织：**
```
components/
├── Button/
│   ├── Button.vue
│   ├── Button.test.ts
│   ├── Button.stories.ts
│   └── index.ts

features/
├── auth/
│   ├── __tests__/
│   │   ├── LoginForm.test.ts
│   │   ├── useAuth.test.ts
│   │   └── auth.service.test.ts
│   │
│   └── __mocks__/
│       └── auth.service.ts
```

**Barrel 文件（index.ts）模式：**
```typescript
// components/Button/index.ts
export { default as Button } from './Button.vue';
export type { default as ButtonProps } from './Button.vue';

// features/auth/index.ts
export { default as LoginForm } from './components/LoginForm.vue';
export { useAuth } from './composables/useAuth';
export { useLogin } from './composables/useLogin';
export * from './types/auth.types';
```

**Composables 命名约定：**
```typescript
// ✅ 推荐：use 前缀
useAuth.ts
useFetch.ts
useLocalStorage.ts
useDebounce.ts

// ✅ 推荐：组合式命名
useFetchUserList.ts
useProductDetail.ts
useCartTotal.ts

// ❌ 避免：无前缀
auth.ts        // 应该用 useAuth
fetchData.ts   // 应该用 useFetch
```

---

> [!SUCCESS]
> 本文档涵盖了 Vue 3 的核心理念、安装配置、组件系统、状态管理、路由、表单处理、高级模式、SSR进阶、国际化、测试等全方位内容。Vue 3 的 Composition API 和 TypeScript 原生支持使其成为现代前端开发的优秀选择。持续实践和深入理解响应式系统是提升 Vue 技能的关键。
