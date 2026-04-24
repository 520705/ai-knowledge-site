---
title: React进阶完全指南
date: 2026-04-24
tags:
  - React
  - 前端开发
  - 高级Hooks
  - 状态管理
  - 性能优化
  - Context
categories:
  - 前端框架
---

> [!NOTE]
> 本文档面向已经掌握 React 基础的开发者，深入讲解高级 Hooks、状态管理方案和性能优化技巧。建议先阅读 [[React入门完全指南]] 打好基础。

---

## 高级 Hooks 深度解析

### useRef：不仅是 DOM 引用

useRef 是 React 中最容易被低估的 Hook 之一。很多初学者只知道用它来获取 DOM 元素，但实际上 useRef 的真正价值在于**持久化值**——它能在组件的整个生命周期内保持一个值的引用，而这个值的变化不会触发重新渲染。

```tsx
import { useRef, useEffect, useState, useCallback } from 'react';

// 基础：访问 DOM 元素
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  useEffect(() => {
    // 组件挂载后自动聚焦
    inputRef.current?.focus();
  }, []);
  
  return (
    <div>
      <input ref={inputRef} type="text" placeholder="会自动聚焦" />
      <button onClick={() => inputRef.current?.focus()}>聚焦输入框</button>
    </div>
  );
}

// 存储上一次的 props 或 state
function PreviousValueDemo() {
  const [value, setValue] = useState('');
  const prevValueRef = useRef('');
  
  useEffect(() => {
    // 每次渲染后更新，这样在下次渲染时就能拿到上一次的值
    prevValueRef.current = value;
  }, [value]);
  
  return (
    <div>
      <input
        value={value}
        onChange={(e) => setValue(e.target.value)}
        placeholder="输入内容..."
      />
      <p>当前值: {value}</p>
      <p>上一次的值: {prevValueRef.current}</p>
    </div>
  );
}

// 使用 ref 存储计时器 ID
function TimerWithRef() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef<NodeJS.Timeout | null>(null);
  
  const startTimer = useCallback(() => {
    if (intervalRef.current) return; // 防止重复启动
    
    intervalRef.current = setInterval(() => {
      setCount(c => c + 1);
    }, 1000);
  }, []);
  
  const stopTimer = useCallback(() => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  }, []);
  
  useEffect(() => {
    return () => {
      // 组件卸载时清理
      if (intervalRef.current) {
        clearInterval(intervalRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={startTimer}>开始</button>
      <button onClick={stopTimer}>停止</button>
    </div>
  );
}

// ref 对象的特性
function RefObjectDemo() {
  const [renderCount, setRenderCount] = useState(0);
  const countRef = useRef(0);
  
  const increment = () => {
    // ref 的值变化不会触发重新渲染
    countRef.current += 1;
    console.log('countRef.current:', countRef.current);
  };
  
  const forceRender = () => {
    // 手动触发重新渲染来查看 ref 的当前值
    setRenderCount(c => c + 1);
  };
  
  return (
    <div>
      <p>countRef 的值（需要重新渲染才能看到）: {countRef.current}</p>
      <p>组件渲染次数: {renderCount}</p>
      <button onClick={increment}>增加 ref（不触发渲染）</button>
      <button onClick={forceRender}>强制重新渲染</button>
    </div>
  );
}
```

> [!TIP]
> useRef 和 useState 的区别在于：useState 会触发重新渲染，适合需要在界面上显示的值；useRef 不会触发重新渲染，适合存储那些不需要显示但需要在组件生命周期内保持的值。

### useMemo：缓存计算结果

当你的组件需要执行昂贵的计算时，使用 useMemo 可以避免每次渲染都重新计算。只有当依赖项发生变化时，计算才会重新执行。

```tsx
import { useMemo, useState } from 'react';

// 昂贵的计算函数
function expensiveCalculation(n: number): number {
  console.log('执行昂贵计算...');
  // 模拟一个耗时的计算
  let result = 0;
  for (let i = 0; i < n * 1000000; i++) {
    result += i;
  }
  return result % 1000000;
}

function ExpensiveCalculation() {
  const [number, setNumber] = useState(5);
  const [multiplier, setMultiplier] = useState(2);
  
  // 只有 number 变化时才重新计算
  const expensiveResult = useMemo(() => {
    return expensiveCalculation(number);
  }, [number]);
  
  // multiplier 变化不会导致昂贵计算重新执行
  const finalResult = expensiveResult * multiplier;
  
  return (
    <div>
      <p>基础数字: {number}</p>
      <p>乘数: {multiplier}</p>
      <p>昂贵计算结果: {expensiveResult}</p>
      <p>最终结果: {finalResult}</p>
      <button onClick={() => setNumber(n => n + 1)}>增加基础数字</button>
      <button onClick={() => setMultiplier(m => m + 1)}>增加乘数</button>
    </div>
  );
}

// 缓存对象引用
function MemoizedObject() {
  const [count, setCount] = useState(0);
  
  // 每次渲染都会创建新对象
  const badConfig = { pageSize: 20, theme: 'dark' };
  
  // 使用 useMemo，只有 count 变化时才重新创建
  const goodConfig = useMemo(() => ({
    pageSize: 20,
    theme: 'dark',
    userId: count  // 这个可能需要基于某些 state
  }), [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c + 1)}>增加</button>
      <p>配置对象: {JSON.stringify(goodConfig)}</p>
    </div>
  );
}

// 依赖其他原子状态的派生状态
function DerivedState() {
  const [users, setUsers] = useState([
    { id: 1, name: '张三', age: 25, active: true },
    { id: 2, name: '李四', age: 30, active: false },
    { id: 3, name: '王五', age: 28, active: true },
  ]);
  const [filterText, setFilterText] = useState('');
  
  // 缓存过滤后的用户列表
  const filteredUsers = useMemo(() => {
    console.log('过滤用户...');
    return users.filter(user =>
      user.name.toLowerCase().includes(filterText.toLowerCase())
    );
  }, [users, filterText]);
  
  // 缓存统计数据
  const stats = useMemo(() => {
    const activeCount = users.filter(u => u.active).length;
    const averageAge = users.reduce((sum, u) => sum + u.age, 0) / users.length;
    return { activeCount, totalCount: users.length, averageAge };
  }, [users]);
  
  return (
    <div>
      <input
        value={filterText}
        onChange={(e) => setFilterText(e.target.value)}
        placeholder="搜索用户..."
      />
      
      <p>统计：{stats.activeCount}/{stats.totalCount} 活跃用户，平均年龄 {stats.averageAge.toFixed(1)} 岁</p>
      
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>
            {user.name} - {user.age}岁 - {user.active ? '活跃' : '不活跃'}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### useCallback：稳定函数引用

每次组件渲染时，函数都会被重新创建。这意味着如果把这个函数作为 prop 传给子组件，子组件会认为这个 prop 变了（因为引用不同），从而可能触发不必要的重新渲染。useCallback 可以帮助你保持函数引用的稳定。

```tsx
import { useCallback, useState } from 'react';
import { memo } from 'react';

// 被 memo 包装的子组件
const TodoItem = memo(function TodoItem({ 
  todo, 
  onToggle, 
  onDelete 
}: { 
  todo: { id: string; text: string; completed: boolean };
  onToggle: (id: string) => void;
  onDelete: (id: string) => void;
}) {
  console.log('TodoItem 渲染:', todo.id);
  
  return (
    <div className="flex items-center p-2 border-b">
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span className="flex-1 ml-2">{todo.text}</span>
      <button onClick={() => onDelete(todo.id)} className="text-red-500">
        删除
      </button>
    </div>
  );
});

function TodoListWithCallback() {
  const [todos, setTodos] = useState([
    { id: '1', text: '学习 React', completed: false },
    { id: '2', text: '完成项目', completed: false },
    { id: '3', text: '写文档', completed: false },
  ]);
  const [, setRenderCount] = useState(0);
  
  // 不使用 useCallback：每次渲染都会创建新函数
  const badToggle = (id: string) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
  };
  
  // 使用 useCallback：只有 todos 变化时才创建新函数
  const goodToggle = useCallback((id: string) => {
    setTodos(prev => prev.map(t => 
      t.id === id ? { ...t, completed: !t.completed } : t
    ));
  }, []);
  
  const handleDelete = useCallback((id: string) => {
    setTodos(prev => prev.filter(t => t.id !== id));
  }, []);
  
  return (
    <div>
      <button onClick={() => setRenderCount(c => c + 1)}>
        强制重新渲染（不改变 todos）
      </button>
      
      <ul>
        {todos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={goodToggle}  // 使用稳定的引用
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}

// useCallback vs useMemo
function CallbackVsMemo() {
  const [a, setA] = useState(1);
  const [b, setB] = useState(2);
  
  // useMemo 缓存计算结果
  const sum = useMemo(() => {
    console.log('计算 a + b...');
    return a + b;
  }, [a, b]);
  
  // useCallback 缓存函数（等价于 useMemo 返回函数）
  const logSum = useCallback(() => {
    console.log('Sum:', a + b);
  }, [a, b]);
  
  // 另一种写法
  const memoizedFunction = useMemo(() => () => {
    console.log('Sum:', a + b);
  }, [a, b]);
  
  return (
    <div>
      <p>a = {a}, b = {b}</p>
      <p>a + b = {sum}</p>
      <button onClick={() => setA(a + 1)}>a + 1</button>
      <button onClick={() => setB(b + 1)}>b + 1</button>
      <button onClick={logSum}>打印结果</button>
    </div>
  );
}
```

> [!WARNING]
> 不要滥用 useMemo 和 useCallback。它们本身也有性能开销——计算依赖、比较旧值和新值都需要时间。对于简单的计算或函数，或者组件本身就不复杂的情况，使用这些 Hook 可能反而会更慢。一般来说，只有当计算确实很耗时、或者组件重渲染会带来明显性能问题时，才需要使用这些优化手段。

### useReducer：复杂状态逻辑

当状态逻辑变得复杂，涉及多个子值和多种操作时，useReducer 比多个 useState 更合适。它让你把所有的状态更新逻辑集中在一个地方，代码更加清晰。

```tsx
import { useReducer } from 'react';

// State 类型
interface CartItem {
  id: string;
  name: string;
  price: number;
  quantity: number;
}

interface CartState {
  items: CartItem[];
  couponCode: string | null;
  discount: number;
}

// Action 类型 - 更好的类型安全性
type CartAction =
  | { type: 'ADD_ITEM'; payload: Omit<CartItem, 'quantity'> }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'UPDATE_QUANTITY'; payload: { id: string; quantity: number } }
  | { type: 'APPLY_COUPON'; payload: string }
  | { type: 'REMOVE_COUPON' }
  | { type: 'CLEAR_CART' };

// Reducer 函数
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case 'ADD_ITEM': {
      const existingItem = state.items.find(
        item => item.id === action.payload.id
      );
      
      if (existingItem) {
        // 如果商品已存在，增加数量
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      
      // 添加新商品
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    }
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    case 'UPDATE_QUANTITY': {
      if (action.payload.quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(
            item => item.id !== action.payload.id
          )
        };
      }
      
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
    }
    
    case 'APPLY_COUPON': {
      // 模拟优惠券逻辑
      const coupons: Record<string, number> = {
        'SAVE10': 10,
        'SAVE20': 20,
      };
      const discount = coupons[action.payload] || 0;
      
      return {
        ...state,
        couponCode: action.payload,
        discount
      };
    }
    
    case 'REMOVE_COUPON':
      return {
        ...state,
        couponCode: null,
        discount: 0
      };
    
    case 'CLEAR_CART':
      return {
        items: [],
        couponCode: null,
        discount: 0
      };
    
    default:
      return state;
  }
}

// 初始化状态
const initialState: CartState = {
  items: [],
  couponCode: null,
  discount: 0
};

// 购物车组件
function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  const [couponInput, setCouponInput] = useState('');
  
  // 计算总价
  const subtotal = state.items.reduce(
    (sum, item) => sum + item.price * item.quantity, 
    0
  );
  const total = subtotal - (subtotal * state.discount / 100);
  
  const addItem = () => {
    dispatch({
      type: 'ADD_ITEM',
      payload: {
        id: Date.now().toString(),
        name: `商品 ${state.items.length + 1}`,
        price: Math.floor(Math.random() * 100) + 10
      }
    });
  };
  
  return (
    <div className="p-4 max-w-md mx-auto">
      <h2 className="text-xl font-bold mb-4">购物车</h2>
      
      <button 
        onClick={addItem}
        className="w-full bg-blue-500 text-white p-2 rounded mb-4"
      >
        添加随机商品
      </button>
      
      {state.items.length === 0 ? (
        <p className="text-gray-500 text-center py-4">购物车是空的</p>
      ) : (
        <ul className="space-y-2 mb-4">
          {state.items.map(item => (
            <li key={item.id} className="flex items-center justify-between border p-2 rounded">
              <div>
                <span className="font-medium">{item.name}</span>
                <span className="text-gray-500 ml-2">¥{item.price}</span>
              </div>
              <div className="flex items-center space-x-2">
                <button
                  onClick={() => dispatch({
                    type: 'UPDATE_QUANTITY',
                    payload: { id: item.id, quantity: item.quantity - 1 }
                  })}
                  className="px-2 py-1 bg-gray-200 rounded"
                >
                  -
                </button>
                <span>{item.quantity}</span>
                <button
                  onClick={() => dispatch({
                    type: 'UPDATE_QUANTITY',
                    payload: { id: item.id, quantity: item.quantity + 1 }
                  })}
                  className="px-2 py-1 bg-gray-200 rounded"
                >
                  +
                </button>
                <button
                  onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: item.id })}
                  className="text-red-500 ml-2"
                >
                  删除
                </button>
              </div>
            </li>
          ))}
        </ul>
      )}
      
      <div className="border-t pt-4">
        <div className="flex justify-between mb-2">
          <span>小计</span>
          <span>¥{subtotal.toFixed(2)}</span>
        </div>
        
        {state.couponCode && (
          <div className="flex justify-between mb-2 text-green-600">
            <span>折扣 ({state.couponCode})</span>
            <span>-¥{(subtotal * state.discount / 100).toFixed(2)}</span>
          </div>
        )}
        
        <div className="flex justify-between font-bold text-lg">
          <span>总计</span>
          <span>¥{total.toFixed(2)}</span>
        </div>
      </div>
      
      <div className="mt-4 flex space-x-2">
        <input
          type="text"
          value={couponInput}
          onChange={(e) => setCouponInput(e.target.value)}
          placeholder="输入优惠券代码"
          className="flex-1 border p-2 rounded"
        />
        <button
          onClick={() => {
            dispatch({ type: 'APPLY_COUPON', payload: couponInput });
            setCouponInput('');
          }}
          className="bg-green-500 text-white px-4 py-2 rounded"
        >
          使用
        </button>
      </div>
      
      {state.items.length > 0 && (
        <button
          onClick={() => dispatch({ type: 'CLEAR_CART' })}
          className="w-full mt-4 text-red-500 border border-red-500 p-2 rounded"
        >
          清空购物车
        </button>
      )}
    </div>
  );
}
```

### 自定义 Hooks：提取和复用逻辑

自定义 Hook 是 React 最有用的特性之一——它让你把组件逻辑提取到可复用的函数中。任何使用到其他 Hook 的函数都可以成为自定义 Hook，只要函数名以 `use` 开头。

```tsx
import { useState, useEffect, useCallback } from 'react';

// 防抖 Hook
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// 本地存储 Hook
function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('读取 localStorage 失败:', error);
      return initialValue;
    }
  });
  
  const setValue = useCallback((value: T | ((val: T) => T)) => {
    try {
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('写入 localStorage 失败:', error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setValue] as const;
}

// 异步数据获取 Hook
interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

function useFetch<T>(url: string): FetchState<T> & { refetch: () => void } {
  const [state, setState] = useState<FetchState<T>>({
    data: null,
    loading: true,
    error: null
  });
  
  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    
    try {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP error: ${response.status}`);
      }
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { ...state, refetch: fetchData };
}

// 在线状态 Hook
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(
    typeof navigator !== 'undefined' ? navigator.onLine : true
  );
  
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  return isOnline;
}

// 窗口尺寸 Hook
function useWindowSize() {
  const [size, setSize] = useState({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// 使用示例
function CustomHooksDemo() {
  const [input, setInput] = useState('');
  const debouncedInput = useDebounce(input, 300);
  
  const [name, setName] = useLocalStorage('userName', '');
  
  const isOnline = useOnlineStatus();
  const { width, height } = useWindowSize();
  
  return (
    <div className="p-4 space-y-4">
      <div>
        <h3>useDebounce 示例</h3>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="输入内容（防抖 300ms）..."
          className="border p-2 rounded w-full"
        />
        <p className="text-gray-500 mt-1">
          原始值: {input} | 防抖后的值: {debouncedInput}
        </p>
      </div>
      
      <div>
        <h3>useLocalStorage 示例</h3>
        <input
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="输入你的名字..."
          className="border p-2 rounded w-full"
        />
        <p className="text-gray-500 mt-1">
          刷新页面后，名字会被保留
        </p>
      </div>
      
      <div>
        <h3>useOnlineStatus 示例</h3>
        <p className={isOnline ? 'text-green-500' : 'text-red-500'}>
          {isOnline ? '在线' : '离线'}
        </p>
      </div>
      
      <div>
        <h3>useWindowSize 示例</h3>
        <p>窗口尺寸: {width} x {height}</p>
      </div>
    </div>
  );
}
```

---

## Context API：跨组件状态共享

### Context 的基本用法

Context 提供了一种在组件树中传递数据的方式，而不需要在每一层手动传递 props。当有些数据需要在组件树的不同层级中被访问时，Context 是一个很好的选择——比如主题色、用户信息、语言设置等。

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. 创建 Context
interface Theme {
  colors: {
    primary: string;
    secondary: string;
    background: string;
    text: string;
  };
  isDark: boolean;
}

interface ThemeContextValue {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextValue | undefined>(undefined);

// 主题定义
const lightTheme: Theme = {
  colors: {
    primary: '#007bff',
    secondary: '#6c757d',
    background: '#ffffff',
    text: '#333333'
  },
  isDark: false
};

const darkTheme: Theme = {
  colors: {
    primary: '#0d6efd',
    secondary: '#adb5bd',
    background: '#1a1a1a',
    text: '#ffffff'
  },
  isDark: true
};

// 2. Provider 组件
export function ThemeProvider({ children }: { children: ReactNode }) {
  const [isDark, setIsDark] = useState(() => {
    const saved = localStorage.getItem('theme');
    return saved === 'dark';
  });
  
  const toggleTheme = () => {
    setIsDark(prev => {
      const newValue = !prev;
      localStorage.setItem('theme', newValue ? 'dark' : 'light');
      return newValue;
    });
  };
  
  const value = {
    theme: isDark ? darkTheme : lightTheme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. 自定义 Hook 消费 Context
export function useTheme() {
  const context = useContext(ThemeContext);
  
  if (context === undefined) {
    throw new Error('useTheme 必须在 ThemeProvider 内部使用');
  }
  
  return context;
}

// 4. 消费 Context 的组件
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      onClick={toggleTheme}
      className="px-4 py-2 rounded transition-colors"
      style={{ 
        background: theme.colors.primary,
        color: theme.colors.background
      }}
    >
      {theme.isDark ? '切换亮色模式' : '切换暗色模式'}
    </button>
  );
}

function ThemedCard() {
  const { theme } = useTheme();
  
  return (
    <div
      className="p-6 rounded-lg shadow"
      style={{ 
        background: theme.colors.background,
        color: theme.colors.text
      }}
    >
      <h2 className="text-xl font-bold mb-2">主题卡片</h2>
      <p>这是一个支持主题切换的卡片组件。</p>
      <div
        className="mt-4 p-4 rounded"
        style={{ background: theme.colors.secondary }}
      >
        副色区域
      </div>
    </div>
  );
}

// 完整示例
function ThemeProviderDemo() {
  return (
    <ThemeProvider>
      <div className="space-y-4 p-4">
        <ThemeToggle />
        <ThemedCard />
        <ThemedCard />
      </div>
    </ThemeProvider>
  );
}
```

### Context 性能优化

Context 看起来很方便，但如果使用不当，可能会导致性能问题。每次 Provider 的值改变时，所有消费这个 Context 的组件都会重新渲染。幸运的是，有几种方法可以优化这个问题。

```tsx
import { createContext, useContext, useState, useMemo, useCallback, ReactNode } from 'react';

// 问题演示
// Context1：每次渲染都创建新对象
const BadContext1 = createContext<{ count: number; increment: () => void } | undefined>();

// Context2：拆分后更优
const CountContext = createContext<number | undefined>();
const IncrementContext = createContext<() => void | undefined>();

// 优化方案1：使用 useMemo 缓存 Context 值
function OptimizedProvider({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  // 使用 useMemo 缓存对象
  const value = useMemo(() => ({
    count,
    increment
  }), [count, increment]);
  
  return (
    <BadContext1.Provider value={value}>
      {children}
    </BadContext1.Provider>
  );
}

// 优化方案2：拆分 Context
function SplitProvider({ children }: { children: ReactNode }) {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  return (
    <CountContext.Provider value={count}>
      <IncrementContext.Provider value={increment}>
        {children}
      </IncrementContext.Provider>
    </CountContext.Provider>
  );
}

// 使用拆分的 Context
function CounterDisplay() {
  const count = useContext(CountContext);
  return <p>计数: {count}</p>;
}

function IncrementButton() {
  const increment = useContext(IncrementContext);
  return <button onClick={increment}>+1</button>;
}

// 优化方案3：使用 useContextSelector
// 这是一个更高级的模式，需要额外的库支持
// 思想是：只订阅 Context 中需要的部分，而不是整个 Context
```

---

## 状态管理方案

### 何时需要全局状态管理

不是所有状态都需要全局管理。React 的状态管理有层次之分：

**本地状态**（useState）：组件内部使用的状态，不需要与其他组件共享。

**提升的状态**：当多个相邻的组件需要共享状态时，把状态提升到它们的公共父组件。

**全局状态**：当状态需要在整个应用的多个不相关位置被访问时，使用全局状态管理。

简单来说：能用本地状态解决的问题，就不要用全局状态。全局状态会增加应用的复杂度，应该谨慎使用。

### Zustand：轻量级状态管理

Zustand 是一个极简的状态管理库，相比 Redux，它没有那么多样板代码，同时性能优秀。Zustand 的设计哲学是"少即是多"——它只提供你需要的功能，不多不少。

```tsx
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

// 基本 Store
interface CounterStore {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
}

const useCounterStore = create<CounterStore>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));

// 带中间件的 Store（开发工具 + 持久化）
interface AuthStore {
  user: { id: string; name: string; email: string } | null;
  isAuthenticated: boolean;
  login: (user: { id: string; name: string; email: string }) => void;
  logout: () => void;
}

const useAuthStore = create<AuthStore>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        isAuthenticated: false,
        login: (user) => set({ user, isAuthenticated: true }),
        logout: () => set({ user: null, isAuthenticated: false }),
      }),
      {
        name: 'auth-storage',  // localStorage 的 key
        partialize: (state) => ({ user: state.user }),  // 只持久化部分状态
      }
    )
  )
);

// 带异步操作的 Store
interface UserStore {
  users: { id: string; name: string; email: string }[];
  loading: boolean;
  error: string | null;
  fetchUsers: () => Promise<void>;
  addUser: (user: { id: string; name: string; email: string }) => void;
  removeUser: (id: string) => void;
}

const useUserStore = create<UserStore>((set, get) => ({
  users: [],
  loading: false,
  error: null,
  
  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      if (!response.ok) throw new Error('获取用户失败');
      const users = await response.json();
      set({ users, loading: false });
    } catch (error) {
      set({ error: (error as Error).message, loading: false });
    }
  },
  
  addUser: (user) => {
    set((state) => ({ users: [...state.users, user] }));
  },
  
  removeUser: (id) => {
    set((state) => ({
      users: state.users.filter((u) => u.id !== id)
    }));
  },
}));

// 使用 Store
function ZustandDemo() {
  const { count, increment, decrement, reset } = useCounterStore();
  
  const { users, loading, error, fetchUsers } = useUserStore();
  
  return (
    <div className="p-4 space-y-6">
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">计数器</h3>
        <p className="text-2xl mb-4">{count}</p>
        <div className="space-x-2">
          <button onClick={increment} className="px-3 py-1 bg-blue-500 text-white rounded">+</button>
          <button onClick={decrement} className="px-3 py-1 bg-gray-500 text-white rounded">-</button>
          <button onClick={reset} className="px-3 py-1 bg-red-500 text-white rounded">重置</button>
        </div>
      </div>
      
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">用户列表</h3>
        <button onClick={fetchUsers} disabled={loading} className="mb-4 px-3 py-1 bg-blue-500 text-white rounded disabled:bg-gray-300">
          {loading ? '加载中...' : '获取用户'}
        </button>
        
        {error && <p className="text-red-500 mb-2">{error}</p>}
        
        <ul className="space-y-2">
          {users.map(user => (
            <li key={user.id} className="border p-2 rounded">
              {user.name} - {user.email}
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
}
```

> [!TIP]
> Zustand 的一个强大特性是可以在组件外部访问 Store。这意味着你可以在事件处理器、setTimeout、甚至 Redux DevTools 中查看和修改状态。

### Jotai：原子化状态管理

Jotai 是另一种状态管理方案，它的核心概念是"原子"（Atom）。每个原子是一个独立的状态单元，组件可以订阅任意数量的原子，只有被订阅的原子变化时组件才会重新渲染。

```tsx
import { atom, useAtom } from 'jotai';

// 基础原子
const countAtom = atom(0);
const nameAtom = atom('张三');

// 派生原子（类似 useMemo）
const doubledCountAtom = atom((get) => get(countAtom) * 2);
const upperCaseNameAtom = atom((get) => get(nameAtom).toUpperCase());

// 异步原子
const userAtom = atom(async (get) => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users/1');
  return response.json();
});

// 带写入的原子
const counterAtom = atom(
  (get) => get(countAtom),  // 读取函数
  (get, set, newValue) => {  // 写入函数
    set(countAtom, newValue);
  }
);

// 复合原子
const userListAtom = atom([
  { id: '1', name: '张三', age: 25 },
  { id: '2', name: '李四', age: 30 },
]);

const userCountAtom = atom((get) => get(userListAtom).length);
const averageAgeAtom = atom((get) => {
  const users = get(userListAtom);
  if (users.length === 0) return 0;
  const sum = users.reduce((acc, user) => acc + user.age, 0);
  return sum / users.length;
});

// Jotai 组件示例
function JotaiDemo() {
  // 使用基础原子
  const [count, setCount] = useAtom(countAtom);
  
  // 使用派生原子
  const [doubledCount] = useAtom(doubledCountAtom);
  
  // 组合多个原子
  const [name, setName] = useAtom(nameAtom);
  const [upperCaseName] = useAtom(upperCaseNameAtom);
  
  return (
    <div className="p-4 space-y-4">
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">基础计数</h3>
        <p>Count: {count}</p>
        <p>Doubled: {doubledCount}</p>
        <div className="space-x-2 mt-2">
          <button onClick={() => setCount(c => c + 1)} className="px-3 py-1 bg-blue-500 text-white rounded">+</button>
          <button onClick={() => setCount(c => c - 1)} className="px-3 py-1 bg-gray-500 text-white rounded">-</button>
        </div>
      </div>
      
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">名字</h3>
        <p>原始: {name}</p>
        <p>大写: {upperCaseName}</p>
        <input
          value={name}
          onChange={(e) => setName(e.target.value)}
          className="border p-2 rounded w-full mt-2"
        />
      </div>
    </div>
  );
}

// 异步数据获取示例
function AsyncJotaiDemo() {
  const [user, setUser] = useAtom(userAtom);
  const [refreshCount, forceRefresh] = useAtom(0);
  
  return (
    <div className="p-4 border rounded">
      <h3 className="font-bold mb-2">异步用户数据</h3>
      {user ? (
        <div>
          <p>姓名: {user.name}</p>
          <p>邮箱: {user.email}</p>
        </div>
      ) : (
        <p>加载中...</p>
      )}
      <button onClick={() => forceRefresh(n => n + 1)} className="mt-2 px-3 py-1 bg-blue-500 text-white rounded">
        重新加载
      </button>
    </div>
  );
}
```

### Redux Toolkit：企业级状态管理

Redux Toolkit 是 Redux 的官方推荐方式，它大幅简化了 Redux 的使用，减少了样板代码。对于大型企业应用，Redux 仍然是一个可靠的选择。

```tsx
import { createSlice, configureStore } from '@reduxjs/toolkit';
import { useSelector, useDispatch } from 'react-redux';

// 定义 Slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0, step: 1 },
  reducers: {
    increment: (state) => {
      state.value += state.step;
    },
    decrement: (state) => {
      state.value -= state.step;
    },
    setStep: (state, action) => {
      state.step = action.payload;
    },
    reset: (state) => {
      state.value = 0;
    },
  },
});

const userSlice = createSlice({
  name: 'user',
  initialState: { profile: null as { id: string; name: string } | null, loading: false },
  reducers: {
    setUser: (state, action) => {
      state.profile = action.payload;
    },
    clearUser: (state) => {
      state.profile = null;
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    },
  },
});

// 创建 Store
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
    user: userSlice.reducer,
  },
});

// 导出 actions
export const { increment, decrement, setStep, reset } = counterSlice.actions;
export const { setUser, clearUser, setLoading } = userSlice.actions;

// 类型定义
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// Redux 组件
function ReduxDemo() {
  const dispatch = useDispatch<AppDispatch>();
  
  // 选择状态
  const { value, step } = useSelector((state: RootState) => state.counter);
  const user = useSelector((state: RootState) => state.user.profile);
  
  return (
    <div className="p-4 space-y-4">
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">Redux 计数器</h3>
        <p>当前值: {value}</p>
        <p>步长: {step}</p>
        
        <div className="space-x-2 mt-2">
          <button onClick={() => dispatch(increment())} className="px-3 py-1 bg-blue-500 text-white rounded">+</button>
          <button onClick={() => dispatch(decrement())} className="px-3 py-1 bg-gray-500 text-white rounded">-</button>
          <button onClick={() => dispatch(setStep(5))} className="px-3 py-1 bg-green-500 text-white rounded">步长=5</button>
          <button onClick={() => dispatch(reset())} className="px-3 py-1 bg-red-500 text-white rounded">重置</button>
        </div>
      </div>
      
      <div className="border p-4 rounded">
        <h3 className="font-bold mb-2">用户信息</h3>
        <p>{user ? user.name : '未登录'}</p>
        <div className="space-x-2 mt-2">
          <button onClick={() => dispatch(setUser({ id: '1', name: '张三' }))} className="px-3 py-1 bg-blue-500 text-white rounded">登录</button>
          <button onClick={() => dispatch(clearUser())} className="px-3 py-1 bg-red-500 text-white rounded">登出</button>
        </div>
      </div>
    </div>
  );
}
```

### 状态管理方案对比

| 方案 | 适用场景 | 复杂度 | 性能 | 推荐指数 |
|------|---------|--------|------|---------|
| **useState** | 组件内部状态 | ⭐ | 最佳 | ⭐⭐⭐⭐⭐ |
| **useReducer** | 中等复杂度状态逻辑 | ⭐⭐ | 良好 | ⭐⭐⭐⭐ |
| **Context** | 跨组件共享少量数据 | ⭐⭐ | 一般 | ⭐⭐⭐ |
| **Zustand** | 中大型应用 | ⭐⭐ | 优秀 | ⭐⭐⭐⭐⭐ |
| **Jotai** | 细粒度状态管理 | ⭐⭐ | 优秀 | ⭐⭐⭐⭐ |
| **Redux Toolkit** | 大型企业应用 | ⭐⭐⭐ | 良好 | ⭐⭐⭐⭐ |

> [!IMPORTANT]
> 选择状态管理方案时，遵循"够用就好"的原则。对于大多数应用，一个简单的 useState + Context 的组合就足够了。只有当你遇到性能问题或者需要更复杂的功能时，才考虑引入额外的状态管理库。

---

## React.memo 与性能优化

### React.memo：避免不必要的渲染

React.memo 是一个高阶组件，它会记忆组件的渲染结果，只有当 props 变化时才重新渲染。这就像是你在问一个人问题，他记住了之前的答案，除非你改变问题（props），否则他会一直给出相同的答案。

```tsx
import { memo, useState, useCallback } from 'react';

// 普通组件
function RegularComponent({ name, age }: { name: string; age: number }) {
  console.log('RegularComponent 渲染');
  return (
    <div className="border p-4 rounded">
      <p>姓名: {name}</p>
      <p>年龄: {age}</p>
    </div>
  );
}

// 使用 memo 的组件
const MemoizedComponent = memo(function MemoizedComponent({ 
  name, 
  age 
}: { 
  name: string; 
  age: number 
}) {
  console.log('MemoizedComponent 渲染');
  return (
    <div className="border p-4 rounded bg-gray-50">
      <p>姓名: {name}</p>
      <p>年龄: {age}</p>
    </div>
  );
});

function MemoDemo() {
  const [count, setCount] = useState(0);
  const [, forceRender] = useState({});
  
  // 这些 props 不会变化
  const person = { name: '张三', age: 25 };
  
  // 这个函数引用每次渲染都变化（如果不使用 useCallback）
  const handleClick = () => {
    console.log('点击');
  };
  
  // 使用 useCallback 保持函数引用稳定
  const handleClickMemoized = useCallback(() => {
    console.log('点击（记忆化）');
  }, []);
  
  return (
    <div className="p-4 space-y-4">
      <div>
        <p>计数器: {count}</p>
        <button onClick={() => setCount(c => c + 1)} className="mr-2 px-3 py-1 bg-blue-500 text-white rounded">增加</button>
        <button onClick={() => forceRender({})} className="px-3 py-1 bg-gray-500 text-white rounded">强制重新渲染</button>
      </div>
      
      <div className="space-y-2">
        <h3>普通组件（每次父组件渲染都会重新渲染）</h3>
        <RegularComponent name={person.name} age={person.age} />
        
        <h3>记忆化组件（props 变化时才重新渲染）</h3>
        <MemoizedComponent name={person.name} age={person.age} />
        
        <h3>带函数 props 的组件</h3>
        <MemoizedComponentWithCallback 
          name={person.name} 
          age={person.age}
          onClick={handleClickMemoized}  // 稳定的引用
        />
      </div>
    </div>
  );
}

// 接收函数作为 props 的组件
const MemoizedComponentWithCallback = memo(function MemoizedComponentWithCallback({ 
  name, 
  age,
  onClick
}: { 
  name: string; 
  age: number;
  onClick: () => void;
}) {
  console.log('MemoizedComponentWithCallback 渲染');
  return (
    <div className="border p-4 rounded bg-green-50">
      <p>姓名: {name}</p>
      <p>年龄: {age}</p>
      <button onClick={onClick} className="mt-2 px-3 py-1 bg-green-500 text-white rounded">点击</button>
    </div>
  );
});
```

### 自定义比较函数

默认情况下，React.memo 只进行浅比较。如果你的 props 是对象，你可以提供自定义的比较函数来精确控制何时重新渲染。

```tsx
import { memo, useState } from 'react';

// 使用自定义比较函数
const OptimizedList = memo(
  function OptimizedList({ 
    items, 
    onItemClick 
  }: { 
    items: { id: string; name: string; score: number }[];
    onItemClick: (id: string) => void;
  }) {
    console.log('OptimizedList 渲染');
    return (
      <ul className="space-y-2">
        {items.map(item => (
          <li key={item.id} className="border p-2 rounded flex justify-between">
            <span>{item.name}: {item.score}分</span>
            <button onClick={() => onItemClick(item.id)} className="text-blue-500">
              查看详情
            </button>
          </li>
        ))}
      </ul>
    );
  },
  // 自定义比较函数
  (prevProps, nextProps) => {
    // 返回 true 表示 props 相等，不需要重新渲染
    // 返回 false 表示 props 不相等，需要重新渲染
    
    // 只比较 items 数组
    const itemsEqual = 
      prevProps.items.length === nextProps.items.length &&
      prevProps.items.every((prev, index) => 
        prev.id === nextProps.items[index].id &&
        prev.name === nextProps.items[index].name &&
        prev.score === nextProps.items[index].score
      );
    
    // 忽略 onItemClick 的变化（假设这个函数总是稳定的）
    return itemsEqual;
  }
);

function CustomComparisonDemo() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([
    { id: '1', name: '数学', score: 95 },
    { id: '2', name: '语文', score: 88 },
    { id: '3', name: '英语', score: 92 },
  ]);
  
  const handleItemClick = (id: string) => {
    console.log('点击了项目:', id);
  };
  
  // 修改分数 - 这会触发重新渲染
  const updateScore = () => {
    setItems(prev => prev.map(item => ({
      ...item,
      score: item.score + 1
    })));
  };
  
  return (
    <div className="p-4 space-y-4">
      <div>
        <p>父组件渲染次数: {count}</p>
        <button onClick={() => setCount(c => c + 1)} className="mr-2 px-3 py-1 bg-blue-500 text-white rounded">
          重新渲染父组件
        </button>
        <button onClick={updateScore} className="px-3 py-1 bg-green-500 text-white rounded">
          增加分数（会真正重新渲染）
        </button>
      </div>
      
      <OptimizedList items={items} onItemClick={handleItemClick} />
    </div>
  );
}
```

---

## 高级模式

### Error Boundaries：错误处理

Error Boundaries 是 React 错误处理的核心机制，它们能捕获子组件树中的 JavaScript 错误，显示备用的 UI，而不是让整个应用崩溃。

```tsx
import { Component, ReactNode } from 'react';

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  // 当子组件抛出错误时调用
  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  // 记录错误信息
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('错误边界捕获了错误:', error, errorInfo);
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }
      return (
        <div className="p-4 bg-red-50 border border-red-200 rounded">
          <h2 className="text-red-600 font-bold">出错了</h2>
          <p className="text-red-500">{this.state.error?.message}</p>
          <button 
            onClick={() => this.setState({ hasError: false, error: null })}
            className="mt-2 px-3 py-1 bg-red-500 text-white rounded"
          >
            重试
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 会抛出错误的组件
function BuggyComponent() {
  const [shouldError, setShouldError] = useState(false);
  
  if (shouldError) {
    throw new Error('这是一个故意的错误！');
  }
  
  return (
    <div className="p-4 border rounded">
      <p>这个组件正常工作</p>
      <button 
        onClick={() => setShouldError(true)}
        className="mt-2 px-3 py-1 bg-red-500 text-white rounded"
      >
        触发错误
      </button>
    </div>
  );
}

function ErrorBoundaryDemo() {
  return (
    <div className="p-4 space-y-4">
      <ErrorBoundary fallback={<div>加载失败</div>}>
        <BuggyComponent />
      </ErrorBoundary>
      
      <ErrorBoundary 
        fallback={
          <div className="p-4 bg-yellow-50 border border-yellow-200 rounded">
            <p>加载失败，请稍后重试</p>
          </div>
        }
      >
        <BuggyComponent />
      </ErrorBoundary>
    </div>
  );
}
```

### Portal：渲染到 DOM 树的其他位置

Portal 允许你将子组件渲染到 DOM 树的其他位置，这在创建模态框、工具提示等需要脱离当前 DOM 层级的 UI 时非常有用。

```tsx
import { createPortal } from 'react-dom';
import { useState } from 'react';

interface ModalProps {
  isOpen: boolean;
  onClose: () => void;
  title: string;
  children: ReactNode;
}

function Modal({ isOpen, onClose, title, children }: ModalProps) {
  if (!isOpen) return null;
  
  // 渲染到 document.body
  return createPortal(
    <div className="fixed inset-0 flex items-center justify-center z-50">
      {/* 遮罩层 */}
      <div 
        className="absolute inset-0 bg-black bg-opacity-50"
        onClick={onClose}
      />
      
      {/* 模态框内容 */}
      <div className="relative bg-white rounded-lg shadow-xl max-w-md w-full mx-4 p-6">
        <div className="flex justify-between items-center mb-4">
          <h2 className="text-xl font-bold">{title}</h2>
          <button 
            onClick={onClose}
            className="text-gray-500 hover:text-gray-700 text-2xl leading-none"
          >
            ×
          </button>
        </div>
        <div>{children}</div>
      </div>
    </div>,
    document.body
  );
}

function PortalDemo() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  return (
    <div className="p-4">
      <button 
        onClick={() => setIsModalOpen(true)}
        className="px-4 py-2 bg-blue-500 text-white rounded"
      >
        打开模态框
      </button>
      
      <Modal 
        isOpen={isModalOpen}
        onClose={() => setIsModalOpen(false)}
        title="欢迎"
      >
        <p className="mb-4">
          这是一个使用 Portal 渲染的模态框。它被挂载在 document.body 下，
          所以不会受到父组件样式的限制（比如 overflow: hidden）。
        </p>
        <button 
          onClick={() => setIsModalOpen(false)}
          className="px-4 py-2 bg-blue-500 text-white rounded"
        >
          关闭
        </button>
      </Modal>
    </div>
  );
}
```

---

## 进阶内容总结

本文档涵盖了 React 进阶的核心知识：

**高级 Hooks** 包括 useRef 的多种用法、useMemo 和 useCallback 的性能优化技巧、useReducer 处理复杂状态逻辑、以及自定义 Hooks 的最佳实践。

**Context API** 讲解了如何创建和使用 Context，以及如何通过拆分 Context 和 useMemo 来优化性能。

**状态管理方案** 深入介绍了 Zustand、Jotai 和 Redux Toolkit 三种主流方案，帮助你根据项目需求选择合适的工具。

**性能优化** 涵盖了 React.memo 的使用、自定义比较函数、以及常见的性能陷阱。

**高级模式** 包括 Error Boundaries 的错误处理和 Portal 的 DOM 渲染技巧。

在 [[React新特性与工程实践.md]] 中，你将学习 React 19 新特性、Next.js 集成、测试策略和生产部署等工程实践内容。

---

## 参考资料

| 资源 | 说明 |
|------|------|
| React Hooks 文档 | https://react.dev/reference/react |
| Zustand 官方文档 | https://zustand.docs.pmnd.rs |
| Jotai 官方文档 | https://jotai.org |
| Redux Toolkit 文档 | https://redux-toolkit.js.org |

> [!SUCCESS]
> 恭喜你完成了 React 进阶内容的学习！现在你已经掌握了 React 的核心高级特性，能够构建复杂、高性能的 React 应用。接下来可以学习工程实践内容，了解如何在生产环境中测试和部署 React 应用。
