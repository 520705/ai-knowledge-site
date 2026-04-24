# Go 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Go 1.23 新特性、并发模型、内存管理、Web 开发、以及 Go 在云原生和 AI 应用中的核心应用。

---

## 目录

1. [[#Go 概述与核心优势]]
2. [[#Go 1.23 新特性]]
3. [[#基础语法与数据结构]]
4. [[#Go 并发模型]]
5. [[#内存管理与垃圾回收]]
6. [[#标准库核心模块]]
7. [[#Web 开发与 API]]
8. [[#数据库操作]]
9. [[#测试与性能优化]]
10. [[#云原生与微服务]]
11. [[#AI 应用集成]]
12. [[#实战项目结构]]
13. [[#选型建议]]

---

## Go 概述与核心优势

### 为什么选择 Go

Go（又称 Golang）由 Google 三位大师 Robert Griesemer、Rob Pike 和 Ken Thompson 于 2009 年正式发布。Go 的设计目标是结合 Python 的开发效率和 C 的系统级性能，创造一门「简单、高效，并发」的编程语言。

Go 的核心优势体现在三个维度：

**简洁优雅**：Go 的语法设计极为简洁，关键字仅 25 个，没有类继承、没有泛型（Go 1.18 前）、没有异常机制。代码可读性极高，学习曲线平缓。

**性能卓越**：Go 编译为机器码，执行效率接近 C，远超 Java、Python 等语言。特别适合对性能有较高要求的场景。

**原生并发**：Go 以 goroutine 和 channel 为核心的并发模型，是处理高并发、I/O 密集型任务的理想选择。

### Go 在 AI 编程中的独特价值

在 AI 应用开发中，Go 保持着独特的优势：

- **高性能推理服务**：用 Go 编写的 AI 推理服务可以高效调用 Python 训练的模型
- **数据处理管道**：Go 的并发模型非常适合构建高效的数据预处理管道
- **边缘计算**：Go 的小体积和低内存占用使其成为边缘 AI 的理想选择
- **API 网关**：用 Go 构建的 API 网关可以高效路由 AI 请求

---

## Go 1.23 新特性

Go 1.23 于 2024 年 8 月发布，带来了多项重大改进。

### 核心新特性

#### 1. 内置迭代器（range over func）

Go 1.23 引入了革命性的 range over func 特性，允许对函数进行迭代：

```go
package main

import "fmt"

func main() {
    // 使用内置迭代器
    iterator := func(yield func(int) bool) {
        for i := 0; i < 10; i++ {
            if !yield(i) {
                return // 提前终止
            }
        }
    }
    
    // 类似 Python 的生成器
    for v := range iterator {
        fmt.Println(v)
    }
    
    // 使用内置的 slices.Concat 和 maps.Keys
    import "slices"
    
    s1 := []int{1, 2, 3}
    s2 := []int{4, 5, 6}
    
    for v := range slices.Concat(s1, s2) {
        fmt.Println(v) // 1, 2, 3, 4, 5, 6
    }
}
```

#### 2. 改进的泛型类型推断

Go 1.23 改进了泛型的类型推断，减少了显式类型参数的需要：

```go
package main

func Map[T, U any](s []T, f func(T) U) []U {
    result := make([]U, len(s))
    for i, v := range s {
        result[i] = f(v)
    }
    return result
}

func main() {
    nums := []int{1, 2, 3, 4, 5}
    
    // Go 1.23: 不需要显式指定类型参数
    strings := Map(nums, func(n int) string {
        return fmt.Sprintf("num-%d", n)
    })
    
    fmt.Println(strings) // [num-1 num-2 num-3 num-4 num-5]
}
```

#### 3. 新的 built-in 函数

```go
package main

import (
    "cmp"
    "iter"
    "slices"
    "maps"
)

func main() {
    // cmp.Or: 返回第一个非零值
    name := cmp.Or("", "default", "fallback")
    fmt.Println(name) // "default"
    
    // iter.Pull: 将迭代器转换为 channel 和停止函数
    seq := slices.Values([]int{1, 2, 3, 4, 5})
    
    next, stop := iter.Pull(seq)
    defer stop()
    
    for v, ok := next(); ok; v, ok = next() {
        fmt.Println(v)
    }
}
```

#### 4. 增强的 maps 包

```go
package main

import "maps"

func main() {
    m1 := map[string]int{"a": 1, "b": 2}
    m2 := map[string]int{"b": 2, "c": 3}
    
    // 并集
    merged := maps.Grow(maps.Clone(m1), m2)
    for k, v := range m2 {
        merged[k] = v
    }
    
    // 判断是否相等（Go 1.21+）
    if maps.Equal(m1, m2) {
        println("Maps are equal")
    }
    
    // 判断是否按序相等
    if maps.EqualFunc(m1, m2, func(a, b int) bool {
        return a == b
    }) {
        println("Maps are equal")
    }
}
```

### 版本特性对比表

| 特性 | 1.20 | 1.21 | 1.22 | 1.23 |
|------|------|------|------|------|
| **泛型类型推断** | 基础 | 改进 | 改进 | 显著改进 |
| **range over func** | ❌ | ❌ | ❌ | ✅ |
| **iter 包** | ❌ | ❌ | ❌ | ✅ |
| **slices.Concat** | ❌ | ✅ | ✅ | ✅ |
| **maps.Equal** | ❌ | ✅ | ✅ | ✅ |
| **结构化日志** | ❌ | ✅ | ✅ | ✅ |
| **match 表达式** | ❌ | ❌ | ❌ | 预览 |

---

## 基础语法与数据结构

### 变量与类型

```go
package main

import "fmt"

func main() {
    // 基础类型
    var name string = "Alice"
    var age int = 30
    var isActive bool = true
    var score float64 = 95.5
    
    // 类型推断
    var city = "Beijing" // 推断为 string
    
    // 短变量声明
    country := "China" // 最常用的方式
    
    // 多个变量
    x, y, z := 1, 2, 3
    
    // 常量
    const Pi = 3.14159
    const (
        StatusOK    = 200
        StatusError = 500
    )
    
    fmt.Println(name, age, city, country)
    fmt.Println(x, y, z)
}
```

### 数组、切片和 Map

```go
package main

import "fmt"

func main() {
    // 数组：固定长度
    arr := [5]int{1, 2, 3, 4, 5}
    fmt.Println("Array:", arr)
    
    // 切片：动态数组
    slice := []int{1, 2, 3}
    slice = append(slice, 4, 5)
    fmt.Println("Slice:", slice)
    
    // 切片操作
    sub := slice[1:3]     // [2, 3]
    fmt.Println("Sub-slice:", sub)
    
    // Map
    ages := map[string]int{
        "Alice": 30,
        "Bob":   25,
    }
    ages["Charlie"] = 35
    
    // Map 操作
    if age, ok := ages["Alice"]; ok {
        fmt.Printf("Alice's age: %d\n", age)
    }
    
    delete(ages, "Bob")
    
    // 遍历 Map
    for name, age := range ages {
        fmt.Printf("%s: %d\n", name, age)
    }
}
```

### 结构体与方法

```go
package main

import "fmt"

type User struct {
    ID    int
    Name  string
    Email string
    Age   int
}

// 结构体方法
func (u *User) Greet() string {
    return fmt.Sprintf("Hello, %s!", u.Name)
}

// 值接收者方法
func (u User) IsAdult() bool {
    return u.Age >= 18
}

// 结构体嵌套
type Address struct {
    City    string
    Country string
}

type Employee struct {
    User
    Address
    Title string
}

func main() {
    user := User{
        ID:    1,
        Name:  "Alice",
        Email: "alice@example.com",
        Age:   30,
    }
    
    fmt.Println(user.Greet())
    fmt.Printf("Is adult: %v\n", user.IsAdult())
    
    // 匿名结构体
    point := struct {
        X, Y int
    }{10, 20}
    fmt.Printf("Point: (%d, %d)\n", point.X, point.Y)
}
```

### 接口

```go
package main

import "fmt"

// 接口定义
type Animal interface {
    Speak() string
    Move() string
}

// 空接口（类似 any）
type Any interface{}

// 接口组合
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}

// Dog 实现
type Dog struct {
    Name string
}

func (d Dog) Speak() string {
    return "Woof!"
}

func (d Dog) Move() string {
    return "runs"
}

func main() {
    var animal Animal = Dog{Name: "Buddy"}
    fmt.Printf("%s %s\n", animal.Speak(), animal.Move())
    
    // 接口检查
    if v, ok := animal.(Dog); ok {
        fmt.Printf("It's a dog named %s\n", v.Name)
    }
    
    // 类型断言
    switch v := animal.(type) {
    case Dog:
        fmt.Printf("Dog: %s\n", v.Name)
    default:
        fmt.Println("Unknown animal")
    }
}
```

### 错误处理

```go
package main

import (
    "errors"
    "fmt"
)

// 自定义错误
var ErrNotFound = errors.New("resource not found")
var ErrInvalidInput = errors.New("invalid input")

func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

// 使用哨兵错误
func findUser(id int) (*User, error) {
    if id <= 0 {
        return nil, ErrInvalidInput
    }
    if id > 1000 {
        return nil, ErrNotFound
    }
    return &User{ID: id, Name: "User"}, nil
}

type User struct {
    ID   int
    Name string
}

func main() {
    // 基本错误处理
    result, err := divide(10, 2)
    if err != nil {
        fmt.Println("Error:", err)
    } else {
        fmt.Println("Result:", result)
    }
    
    // 使用 errors.Is 检查错误链
    user, err := findUser(2000)
    if err != nil {
        if errors.Is(err, ErrNotFound) {
            fmt.Println("User not found")
        } else if errors.Is(err, ErrInvalidInput) {
            fmt.Println("Invalid input")
        }
    }
    
    // 错误包装
    err = fmt.Errorf("failed to process: %w", ErrNotFound)
    if errors.Is(err, ErrNotFound) {
        fmt.Println("Caught wrapped error")
    }
}
```

---

## Go 并发模型

### Goroutine 基础

```go
package main

import (
    "fmt"
    "time"
)

func task(name string, duration time.Duration) {
    fmt.Printf("Task %s started\n", name)
    time.Sleep(duration)
    fmt.Printf("Task %s completed\n", name)
}

func main() {
    // 启动 goroutine
    go task("A", 1*time.Second)
    go task("B", 500*time.Millisecond)
    
    // 匿名函数
    go func() {
        fmt.Println("Anonymous task")
    }()
    
    // 等待所有任务完成
    time.Sleep(2 * time.Second)
    fmt.Println("All tasks done")
}
```

### Channel 通信

```go
package main

import "fmt"

func producer(ch chan<- int) {
    for i := 1; i <= 5; i++ {
        ch <- i
    }
    close(ch) // 关闭 channel
}

func consumer(ch <-chan int, done chan<- bool) {
    for v := range ch {
        fmt.Printf("Received: %d\n", v)
    }
    done <- true
}

func main() {
    // 创建 channel
    ch := make(chan int)
    done := make(chan bool)
    
    go producer(ch)
    go consumer(ch, done)
    
    <-done // 等待完成
    fmt.Println("Done")
}
```

### Select 多路复用

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan string)
    ch2 := make(chan string)
    
    go func() {
        time.Sleep(1 * time.Second)
        ch1 <- "Result from source 1"
    }()
    
    go func() {
        time.Sleep(500 * time.Millisecond)
        ch2 <- "Result from source 2"
    }()
    
    // 多路 select
    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-ch1:
            fmt.Println(msg1)
        case msg2 := <-ch2:
            fmt.Println(msg2)
        case <-time.After(2 * time.Second):
            fmt.Println("Timeout!")
            return
        }
    }
}
```

### 同步原语

```go
package main

import (
    "fmt"
    "sync"
    "sync/atomic"
)

func main() {
    // WaitGroup: 等待一组 goroutine 完成
    var wg sync.WaitGroup
    
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(id int) {
            defer wg.Done()
            fmt.Printf("Worker %d started\n", id)
        }(i)
    }
    
    wg.Wait()
    fmt.Println("All workers completed")
    
    // Mutex: 互斥锁
    var mu sync.Mutex
    counter := 0
    
    for i := 0; i < 1000; i++ {
        go func() {
            mu.Lock()
            counter++
            mu.Unlock()
        }()
    }
    
    // 使用 atomic
    var atomicCounter int64
    
    for i := 0; i < 1000; i++ {
        go func() {
            atomic.AddInt64(&atomicCounter, 1)
        }()
    }
    
    fmt.Printf("Counter: %d\n", counter)
    fmt.Printf("AtomicCounter: %d\n", atomicCounter)
}
```

### Context 上下文

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func longRunningTask(ctx context.Context) error {
    for {
        select {
        case <-ctx.Done():
            return ctx.Err()
        default:
            fmt.Println("Processing...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}

func main() {
    // 带超时的 context
    ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
    defer cancel()
    
    err := longRunningTask(ctx)
    if err != nil {
        fmt.Printf("Task cancelled: %v\n", err)
    }
    
    // 带取消的 context
    ctx2, cancel2 := context.WithCancel(context.Background())
    
    go func() {
        time.Sleep(1 * time.Second)
        cancel2() // 取消
    }()
    
    select {
    case <-ctx2.Done():
        fmt.Println("Context cancelled")
    }
}
```

### 并发模式

```go
package main

import (
    "fmt"
    "sync"
)

// Pipeline 模式
func pipeline() {
    // Stage 1: 生成
    gen := func(done <-chan struct{}) <-chan int {
        ch := make(chan int)
        go func() {
            defer close(ch)
            for i := 1; i <= 10; i++ {
                select {
                case <-done:
                    return
                case ch <- i:
                }
            }
        }()
        return ch
    }
    
    // Stage 2: 平方
    square := func(done <-chan struct{}, in <-chan int) <-chan int {
        ch := make(chan int)
        go func() {
            defer close(ch)
            for v := range in {
                select {
                case <-done:
                    return
                case ch <- v * v:
                }
            }
        }()
        return ch
    }
    
    // Stage 3: 打印
    done := make(chan struct{})
    defer close(done)
    
    for v := range square(done, gen(done)) {
        fmt.Println(v)
    }
}

// Fan-out/Fan-in 模式
func fanOutFanIn() {
    jobs := make(chan int, 100)
    results := make(chan int, 100)
    
    // 分发 worker
    for w := 1; w <= 3; w++ {
        go func(workerID int) {
            for j := range jobs {
                results <- j * j
            }
        }(w)
    }
    
    // 发送 jobs
    go func() {
        for j := 1; j <= 9; j++ {
            jobs <- j
        }
        close(jobs)
    }()
    
    // 收集结果
    for r := 1; r <= 9; r++ {
        fmt.Printf("Result: %d\n", <-results)
    }
}

func main() {
    pipeline()
    fanOutFanIn()
}
```

---

## 内存管理与垃圾回收

### 内存分配

```go
package main

import "fmt"

func main() {
    // new: 返回指针
    p := new(int)
    *p = 42
    fmt.Println(*p)
    
    // make: 用于 slice, map, channel
    s := make([]int, 0, 10) // len=0, cap=10
    m := make(map[string]int)
    ch := make(chan int, 5)
    
    s = append(s, 1, 2, 3)
    m["key"] = 100
    
    fmt.Println(s, m)
    fmt.Printf("Channel len=%d, cap=%d\n", len(ch), cap(ch))
}
```

### 逃逸分析

```go
package main

import "fmt"

// 不会逃逸：参数和返回值在栈上
func add(a, b int) int {
    return a + b
}

// 会逃逸：返回指针指向堆
func createUser() *User {
    user := &User{Name: "Alice"} // 逃逸到堆
    return user
}

// 逃逸分析示例
func escapeAnalysis() {
    // 编译时使用 -gcflags="-m" 查看逃逸分析
    // go build -gcflags="-m" main.go
    
    // 以下情况会发生逃逸：
    
    // 1. 返回指针
    p := createUser()
    fmt.Println(p.Name)
    
    // 2. interface 类型
    var i interface{} = 42 // int 逃逸
    fmt.Println(i)
    
    // 3. 切片增长超过容量
    s := make([]int, 0)
    for i := 0; i < 100; i++ {
        s = append(s, i) // 大切片可能逃逸
    }
}
```

### 性能优化

```go
package main

import (
    "sync"
    "sync/atomic"
)

// 对象池
func objectPool() {
    type User struct {
        Name string
        Age  int
    }
    
    pool := sync.Pool{
        New: func() interface{} {
            return &User{}
        },
    }
    
    // 获取对象
    user := pool.Get().(*User)
    user.Name = "Alice"
    user.Age = 30
    
    // 归还对象
    pool.Put(user)
}

// 减少内存分配
func optimizeAllocation() {
    // 预分配切片容量
    s := make([]int, 0, 1000) // 预分配容量
    
    for i := 0; i < 1000; i++ {
        s = append(s, i)
    }
    
    // 复用已分配的切片
    s2 := s[:0] // 重置长度，但不释放内存
    for i := 0; i < 100; i++ {
        s2 = append(s2, i)
    }
}

// sync.Map 用于高并发场景
func concurrentMap() {
    var m sync.Map
    
    // 存储
    m.Store("key1", "value1")
    m.Store("key2", "value2")
    
    // 加载
    if v, ok := m.Load("key1"); ok {
        fmt.Println(v)
    }
    
    // 原子更新
    m.Store("counter", int64(0))
    
    // 原子递增
    newVal, _ := m.LoadOrStore("counter", int64(0))
    atomic.AddInt64(newVal.(*int64), 1)
}
```

---

## 标准库核心模块

### Strings 和 Bytes

```go
package main

import (
    "bytes"
    "fmt"
    "strings"
)

func stringOps() {
    // 基础操作
    s := "Hello, World!"
    
    fmt.Println(strings.ToUpper(s))    // HELLO, WORLD!
    fmt.Println(strings.ToLower(s))    // hello, world!
    fmt.Println(strings.TrimSpace(s))  // Hello, World!
    
    // 分割和连接
    parts := strings.Split(s, ",")      // [Hello World!]
    joined := strings.Join(parts, "-")  // Hello- World!
    
    // 查找
    idx := strings.Index(s, "World")    // 7
    hasPrefix := strings.HasPrefix(s, "Hello")
    hasSuffix := strings.HasSuffix(s, "!")
    
    // 构建器（高效拼接）
    var builder strings.Builder
    builder.WriteString("Hello")
    builder.WriteString(", ")
    builder.WriteString("World")
    fmt.Println(builder.String())
    
    // Reader
    reader := strings.NewReader("Hello World")
    buf := make([]byte, 5)
    n, _ := reader.Read(buf)
    fmt.Printf("Read %d bytes: %s\n", n, buf)
}

func bytesOps() {
    // Buffer
    buf := bytes.NewBuffer([]byte{})
    buf.WriteString("Hello")
    buf.WriteByte(',')
    buf.Write([]byte(" World"))
    
    fmt.Println(buf.String())
    
    // Buffer methods
    data := []byte("Hello, World!")
    
    // 查找
    idx := bytes.Index(data, []byte("World"))
    
    // 分割
    parts := bytes.Split(data, []byte(", "))
    
    // 比较
    cmp := bytes.Compare([]byte("abc"), []byte("abd"))
    
    fmt.Println(idx, parts, cmp)
}
```

### Time 和 Duration

```go
package main

import (
    "fmt"
    "time"
)

func timeOps() {
    // 当前时间
    now := time.Now()
    fmt.Println(now)
    
    // 时间戳
    unix := now.Unix()
    unixMilli := now.UnixMilli()
    unixNano := now.UnixNano()
    
    // 解析时间
    t, _ := time.Parse(time.RFC3339, "2024-01-01T12:00:00Z")
    
    // 格式化
    formatted := now.Format("2006-01-02 15:04:05")
    
    // 时间计算
    tomorrow := now.AddDate(0, 0, 1)
    later := now.Add(2 * time.Hour)
    
    // Duration
    dur, _ := time.ParseDuration("1h30m")
    future := now.Add(dur)
    
    // 时间差
    diff := future.Sub(now)
    
    // 计时器
    start := time.Now()
    time.Sleep(100 * time.Millisecond)
    elapsed := time.Since(start)
    
    fmt.Printf("Unix: %d\n", unix)
    fmt.Printf("Formatted: %s\n", formatted)
    fmt.Printf("Tomorrow: %s\n", tomorrow.Format("2006-01-02"))
    fmt.Printf("Duration: %s\n", diff)
    fmt.Printf("Elapsed: %v\n", elapsed)
}
```

### IO 和 Formatting

```go
package main

import (
    "bytes"
    "fmt"
    "io"
    "strings"
)

func ioOps() {
    // Reader
    reader := strings.NewReader("Hello, World!")
    data := make([]byte, 5)
    
    for {
        n, err := reader.Read(data)
        if err == io.EOF {
            break
        }
        fmt.Printf("Read %d bytes: %s\n", n, data[:n])
    }
    
    // Writer
    writer := bytes.NewBuffer(nil)
    n, _ := writer.Write([]byte("Hello"))
    fmt.Printf("Wrote %d bytes\n", n)
    
    // TeeReader
    source := strings.NewReader("Hello")
    var dest bytes.Buffer
    tee := io.TeeReader(source, &dest)
    
    io.Copy(os.Stdout, tee)
    fmt.Printf("\nCaptured: %s\n", dest.String())
    
    // MultiWriter
    var buf1, buf2 bytes.Buffer
    multi := io.MultiWriter(&buf1, &buf2)
    io.WriteString(multi, "Written to multiple")
    
    fmt.Printf("Buffer1: %s\n", buf1.String())
    fmt.Printf("Buffer2: %s\n", buf2.String())
}

func fmtOps() {
    // 基本格式化
    fmt.Printf("String: %s, Int: %d, Float: %.2f\n", "hello", 42, 3.14159)
    
    // 宽度和对齐
    fmt.Printf("|%10s|%5d|%-10s|\n", "Right", 100, "Left")
    
    // 格式化动词
    type Point struct{ X, Y int }
    p := Point{10, 20}
    fmt.Printf("%v\n", p)      // {10 20}
    fmt.Printf("%+v\n", p)     // {X:10 Y:20}
    fmt.Printf("%#v\n", p)     // main.Point{X:10, Y:20}
    
    // Sprintf
    s := fmt.Sprintf("Error: %v", "something went wrong")
    fmt.Println(s)
}
```

### Encoding

```go
package main

import (
    "encoding/json"
    "encoding/xml"
    "fmt"
)

func jsonOps() {
    // 结构体标签控制 JSON 序列化
    type User struct {
        ID    int    `json:"id"`
        Name  string `json:"name"`
        Email string `json:"email,omitempty"`
        Age   int    `json:"-"`
    }
    
    user := User{ID: 1, Name: "Alice", Age: 30}
    
    // 序列化
    data, _ := json.Marshal(user)
    fmt.Printf("JSON: %s\n", data)
    
    // 格式化序列化
    data, _ = json.MarshalIndent(user, "", "  ")
    fmt.Printf("Pretty JSON:\n%s\n", data)
    
    // 反序列化
    var user2 User
    json.Unmarshal(data, &user2)
    fmt.Printf("Parsed: %+v\n", user2)
    
    // Decoder
    decoder := json.NewDecoder(strings.NewReader(`{"id":2,"name":"Bob"}`))
    var user3 User
    decoder.Decode(&user3)
}

func xmlOps() {
    type Person struct {
        XMLName xml.Name `xml:"person"`
        Name    string   `xml:"name"`
        Age     int      `xml:"age"`
    }
    
    p := Person{Name: "Alice", Age: 30}
    
    // 序列化
    data, _ := xml.MarshalIndent(p, "", "  ")
    fmt.Printf("XML:\n%s\n", data)
    
    // 反序列化
    var p2 Person
    xml.Unmarshal(data, &p2)
    fmt.Printf("Parsed: %+v\n", p2)
}
```

---

## Web 开发与 API

### HTTP 基础

```go
package main

import (
    "encoding/json"
    "fmt"
    "io"
    "net/http"
)

func httpClient() {
    // GET 请求
    resp, err := http.Get("https://api.example.com/users")
    if err != nil {
        panic(err)
    }
    defer resp.Body.Close()
    
    body, _ := io.ReadAll(resp.Body)
    fmt.Printf("Status: %s\n", resp.Status)
    fmt.Printf("Body: %s\n", string(body))
    
    // 带查询参数的请求
    req, _ := http.NewRequest("GET", "https://api.example.com/users", nil)
    q := req.URL.Query()
    q.Add("page", "1")
    q.Add("limit", "10")
    req.URL.RawQuery = q.Encode()
    
    // POST 请求
    payload := map[string]interface{}{"name": "Alice", "email": "alice@example.com"}
    jsonData, _ := json.Marshal(payload)
    
    resp2, err := http.Post(
        "https://api.example.com/users",
        "application/json",
        bytes.NewBuffer(jsonData),
    )
    if err != nil {
        panic(err)
    }
    defer resp2.Body.Close()
}
```

### HTTP 服务器

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "net/http"
)

type User struct {
    ID    int    `json:"id"`
    Name  string `json:"name"`
    Email string `json:"email"`
}

var users = []User{
    {ID: 1, Name: "Alice", Email: "alice@example.com"},
    {ID: 2, Name: "Bob", Email: "bob@example.com"},
}

func homeHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Welcome to the API!")
}

func getUsers(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(users)
}

func getUserByID(w http.ResponseWriter, r *http.Request) {
    id := r.PathValue("id")
    for _, u := range users {
        if fmt.Sprintf("%d", u.ID) == id {
            w.Header().Set("Content-Type", "application/json")
            json.NewEncoder(w).Encode(u)
            return
        }
    }
    http.Error(w, "User not found", http.StatusNotFound)
}

func createUser(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }
    
    var user User
    if err := json.NewDecoder(r.Body).Decode(&user); err != nil {
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }
    
    user.ID = len(users) + 1
    users = append(users, user)
    
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(http.StatusCreated)
    json.NewEncoder(w).Encode(user)
}

func jsonResponse(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func middlewareExample(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // CORS
        w.Header().Set("Access-Control-Allow-Origin", "*")
        w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE")
        w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
        
        // 日志
        log.Printf("%s %s", r.Method, r.URL.Path)
        
        next.ServeHTTP(w, r)
    })
}

func main() {
    mux := http.NewServeMux()
    
    // 路由
    mux.HandleFunc("/", homeHandler)
    mux.HandleFunc("GET /users", getUsers)
    mux.HandleFunc("POST /users", createUser)
    mux.HandleFunc("GET /users/{id}", getUserByID)
    
    // 中间件
    handler := middlewareExample(mux)
    
    fmt.Println("Server starting on :8080")
    log.Fatal(http.ListenAndServe(":8080", handler))
}
```

### Gin 框架

```go
package main

import (
    "net/http"
    
    "github.com/gin-gonic/gin"
)

type User struct {
    ID    uint   `json:"id" gorm:"primaryKey"`
    Name  string `json:"name" binding:"required,min=2,max=50"`
    Email string `json:"email" binding:"required,email"`
    Age   int    `json:"age" binding:"gte=0,lte=150"`
}

var users = []User{
    {ID: 1, Name: "Alice", Email: "alice@example.com", Age: 30},
    {ID: 2, Name: "Bob", Email: "bob@example.com", Age: 25},
}

func main() {
    r := gin.Default()
    
    // 全局中间件
    r.Use(gin.Logger())
    r.Use(gin.Recovery())
    
    // 自定义中间件
    r.Use(func(c *gin.Context) {
        c.Header("X-Custom-Header", "value")
        c.Next()
    })
    
    // 路由组
    api := r.Group("/api/v1")
    {
        api.GET("/users", getUsers)
        api.GET("/users/:id", getUserByID)
        api.POST("/users", createUser)
        api.PUT("/users/:id", updateUser)
        api.DELETE("/users/:id", deleteUser)
    }
    
    r.Run(":8080")
}

func getUsers(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "data": users,
        "total": len(users),
    })
}

func getUserByID(c *gin.Context) {
    id := c.Param("id")
    
    for _, u := range users {
        if fmt.Sprintf("%d", u.ID) == id {
            c.JSON(http.StatusOK, u)
            return
        }
    }
    
    c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
}

func createUser(c *gin.Context) {
    var user User
    
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    
    user.ID = uint(len(users) + 1)
    users = append(users, user)
    
    c.JSON(http.StatusCreated, user)
}

func updateUser(c *gin.Context) {
    id := c.Param("id")
    
    for i, u := range users {
        if fmt.Sprintf("%d", u.ID) == id {
            var updated User
            if err := c.ShouldBindJSON(&updated); err != nil {
                c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
                return
            }
            users[i] = updated
            c.JSON(http.StatusOK, updated)
            return
        }
    }
    
    c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
}

func deleteUser(c *gin.Context) {
    id := c.Param("id")
    
    for i, u := range users {
        if fmt.Sprintf("%d", u.ID) == id {
            users = append(users[:i], users[i+1:]...)
            c.JSON(http.StatusOK, gin.H{"message": "User deleted"})
            return
        }
    }
    
    c.JSON(http.StatusNotFound, gin.H{"error": "User not found"})
}
```

---

## 数据库操作

### 数据库/SQL

```go
package main

import (
    "database/sql"
    "fmt"
    
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    // 连接数据库
    db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/mydb")
    if err != nil {
        panic(err)
    }
    defer db.Close()
    
    // 设置连接池
    db.SetMaxOpenConns(25)
    db.SetMaxIdleConns(5)
    
    // Ping
    if err := db.Ping(); err != nil {
        panic(err)
    }
    
    // 查询
    rows, err := db.Query("SELECT id, name, email FROM users WHERE age > ?", 18)
    if err != nil {
        panic(err)
    }
    defer rows.Close()
    
    for rows.Next() {
        var id int
        var name, email string
        if err := rows.Scan(&id, &name, &email); err != nil {
            panic(err)
        }
        fmt.Printf("User: %d, %s, %s\n", id, name, email)
    }
    
    // 查询单行
    var name string
    err = db.QueryRow("SELECT name FROM users WHERE id = ?", 1).Scan(&name)
    
    // 插入
    result, err := db.Exec(
        "INSERT INTO users (name, email, age) VALUES (?, ?, ?)",
        "Charlie", "charlie@example.com", 35,
    )
    
    id, _ := result.LastInsertId()
    fmt.Printf("Inserted user with ID: %d\n", id)
    
    // 更新
    affected, _ := result.RowsAffected()
    fmt.Printf("Affected rows: %d\n", affected)
    
    // 事务
    tx, err := db.Begin()
    if err != nil {
        panic(err)
    }
    
    _, err = tx.Exec("UPDATE users SET age = ? WHERE id = ?", 31, id)
    if err != nil {
        tx.Rollback()
        panic(err)
    }
    
    tx.Commit()
}
```

### GORM ORM

```go
package main

import (
    "fmt"
    "time"
    
    "gorm.io/driver/postgres"
    "gorm.io/gorm"
)

type User struct {
    ID        uint      `gorm:"primaryKey"`
    Name     string    `gorm:"size:100;not null"`
    Email    string    `gorm:"size:255;uniqueIndex;not null"`
    Age      int       `gorm:"default:0"`
    CreatedAt time.Time
    UpdatedAt time.Time
    
    // 关联
    Posts []Post `gorm:"foreignKey:UserID"`
}

type Post struct {
    ID        uint      `gorm:"primaryKey"`
    Title     string    `gorm:"size:200;not null"`
    Content   string    `gorm:"type:text"`
    UserID    uint      `gorm:"index"`
    CreatedAt time.Time
    UpdatedAt time.Time
}

func main() {
    // 连接数据库
    dsn := "host=localhost user=gorm password=gorm dbname=gorm port=5432 sslmode=disable"
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{})
    if err != nil {
        panic(err)
    }
    
    // 自动迁移
    db.AutoMigrate(&User{}, &Post{})
    
    // 创建
    user := User{Name: "Alice", Email: "alice@example.com", Age: 30}
    result := db.Create(&user)
    fmt.Printf("Created user with ID: %d\n", user.ID)
    
    // 批量创建
    users := []User{
        {Name: "Bob", Email: "bob@example.com", Age: 25},
        {Name: "Charlie", Email: "charlie@example.com", Age: 35},
    }
    db.CreateInBatches(users, 100)
    
    // 查询
    var u User
    db.First(&u, 1)           // 按主键查询
    db.First(&u, "email = ?", "alice@example.com")
    
    // 条件查询
    var adults []User
    db.Where("age > ?", 18).Find(&adults)
    
    // 使用结构体查询
    db.Where(&User{Name: "Alice", Age: 30}).First(&u)
    
    // 更新
    db.Model(&u).Update("Email", "newalice@example.com")
    db.Model(&u).Updates(User{Email: "newalice2@example.com", Age: 31})
    
    // 删除
    db.Delete(&u, 1)
    
    // 关联查询
    var userWithPosts User
    db.Preload("Posts").First(&userWithPosts, 1)
    fmt.Printf("User: %s, Posts: %d\n", userWithPosts.Name, len(userWithPosts.Posts))
    
    // 关联创建
    db.Create(&User{
        Name:  "Diana",
        Email: "diana@example.com",
        Posts: []Post{
            {Title: "Hello", Content: "World"},
            {Title: "GORM", Content: "Is great"},
        },
    })
    
    // 软删除
    db.Delete(&user, 2) // 会设置 deleted_at
    
    // 原生 SQL
    db.Raw("SELECT * FROM users WHERE age > ?", 20).Scan(&adults)
}
```

---

## 测试与性能优化

### 单元测试

```go
package main

import (
    "errors"
    "testing"
)

// 待测试函数
func Add(a, b int) int {
    return a + b
}

func Divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

func Fibonacci(n int) int {
    if n <= 1 {
        return n
    }
    return Fibonacci(n-1) + Fibonacci(n-2)
}

// 测试函数
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"zero", 0, 5, 5},
        {"negative", -1, 1, 0},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d, want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}

func TestDivide(t *testing.T) {
    result, err := Divide(10, 2)
    if err != nil {
        t.Errorf("unexpected error: %v", err)
    }
    if result != 5 {
        t.Errorf("Divide(10, 2) = %f, want 5", result)
    }
    
    _, err = Divide(10, 0)
    if err == nil {
        t.Error("expected error for division by zero")
    }
}

func TestFibonacci(t *testing.T) {
    tests := []struct {
        n        int
        expected int
    }{
        {0, 0},
        {1, 1},
        {2, 1},
        {3, 2},
        {10, 55},
    }
    
    for _, tt := range tests {
        result := Fibonacci(tt.n)
        if result != tt.expected {
            t.Errorf("Fibonacci(%d) = %d, want %d", tt.n, result, tt.expected)
        }
    }
}

// Benchmark
func BenchmarkFibonacci(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Fibonacci(20)
    }
}

func BenchmarkFibonacciIterative(b *testing.B) {
    for i := 0; i < b.N; i++ {
        fibIterative(20)
    }
}

func fibIterative(n int) int {
    if n <= 1 {
        return n
    }
    a, b := 0, 1
    for i := 2; i <= n; i++ {
        a, b = b, a+b
    }
    return b
}
```

### Table-Driven Tests

```go
package main

import (
    "strings"
    "testing"
)

func TestStringOps(t *testing.T) {
    tests := []struct {
        name   string
        input string
        op    string
        want  string
    }{
        {"uppercase", "hello", "upper", "HELLO"},
        {"lowercase", "WORLD", "lower", "world"},
        {"trim", "  hello  ", "trim", "hello"},
        {"reverse", "hello", "reverse", "olleh"},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            var got string
            switch tt.op {
            case "upper":
                got = strings.ToUpper(tt.input)
            case "lower":
                got = strings.ToLower(tt.input)
            case "trim":
                got = strings.TrimSpace(tt.input)
            case "reverse":
                runes := []rune(tt.input)
                for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
                    runes[i], runes[j] = runes[j], runes[i]
                }
                got = string(runes)
            }
            
            if got != tt.want {
                t.Errorf("got %s, want %s", got, tt.want)
            }
        })
    }
}
```

### 性能分析

```go
package main

import (
    "runtime"
    "testing"
)

func BenchmarkMemory(b *testing.B) {
    // 记录内存分配
    b.ReportAllocs()
    
    var result []int
    for i := 0; i < b.N; i++ {
        result = make([]int, 0, 1000)
        for j := 0; j < 1000; j++ {
            result = append(result, j)
        }
    }
}

func BenchmarkMemoryPreallocated(b *testing.B) {
    b.ReportAllocs()
    
    for i := 0; i < b.N; i++ {
        result := make([]int, 1000)
        for j := 0; j < 1000; j++ {
            result[j] = j
        }
    }
}

func analyzeMemory() {
    var m runtime.MemStats
    runtime.ReadMemStats(&m)
    
    println("Alloc:", m.Alloc/1024, "KB")
    println("TotalAlloc:", m.TotalAlloc/1024, "KB")
    println("Sys:", m.Sys/1024, "KB")
    println("NumGC:", m.NumGC)
}
```

---

## 云原生与微服务

### gRPC 服务

```go
// proto/user.proto
syntax = "proto3";

package user;

option go_package = "github.com/example/user;user";

service UserService {
    rpc GetUser(GetUserRequest) returns (User);
    rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
    rpc CreateUser(CreateUserRequest) returns (User);
    rpc DeleteUser(DeleteUserRequest) returns (Empty);
}

message User {
    uint64 id = 1;
    string name = 2;
    string email = 3;
    int32 age = 4;
}

message GetUserRequest {
    uint64 id = 1;
}

message ListUsersRequest {
    int32 page = 1;
    int32 page_size = 2;
}

message ListUsersResponse {
    repeated User users = 1;
    int32 total = 2;
}

message CreateUserRequest {
    string name = 1;
    string email = 2;
    int32 age = 3;
}

message DeleteUserRequest {
    uint64 id = 1;
}

message Empty {}
```

```go
// server/main.go
package main

import (
    "context"
    "log"
    "net"
    
    "google.golang.org/grpc"
    "google.golang.org/grpc/reflection"
    
    pb "github.com/example/user"
)

type server struct {
    pb.UnimplementedUserServiceServer
    users map[uint64]*pb.User
    nextID uint64
}

func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    if user, ok := s.users[req.Id]; ok {
        return user, nil
    }
    return nil, nil
}

func (s *server) ListUsers(ctx context.Context, req *pb.ListUsersRequest) (*pb.ListUsersResponse, error) {
    var users []*pb.User
    for _, u := range s.users {
        users = append(users, u)
    }
    return &pb.ListUsersResponse{
        Users: users,
        Total: int32(len(users)),
    }, nil
}

func (s *server) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.User, error) {
    user := &pb.User{
        Id:    s.nextID,
        Name:  req.Name,
        Email: req.Email,
        Age:   req.Age,
    }
    s.users[s.nextID] = user
    s.nextID++
    return user, nil
}

func (s *server) DeleteUser(ctx context.Context, req *pb.DeleteUserRequest) (*pb.Empty, error) {
    delete(s.users, req.Id)
    return &pb.Empty{}, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    
    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{
        users:  make(map[uint64]*pb.User),
        nextID: 1,
    })
    
    reflection.Register(s)
    
    log.Println("gRPC server starting on :50051")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("failed to serve: %v", err)
    }
}
```

### Docker 部署

```dockerfile
# 构建阶段
FROM golang:1.23-alpine AS builder

WORKDIR /app

# 复制 go.mod 和 go.sum
COPY go.mod go.sum ./
RUN go mod download

# 复制源码
COPY . .

# 编译
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

# 运行阶段
FROM alpine:latest

WORKDIR /app

# 安装 CA 证书
RUN apk --no-cache add ca-certificates

# 复制二进制文件
COPY --from=builder /app/main .

# 复制静态资源
COPY static/ ./static/

EXPOSE 8080

CMD ["./main"]
```

### Kubernetes 部署

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: go-api
  template:
    metadata:
      labels:
        app: go-api
    spec:
      containers:
        - name: api
          image: your-registry/go-api:latest
          ports:
            - containerPort: 8080
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
```

---

## AI 应用集成

### OpenAI SDK

```go
package main

import (
    "context"
    "fmt"
    
    "github.com/sashabaranov/go-openai"
)

func main() {
    client := openai.NewClient("your-api-key")
    
    ctx := context.Background()
    
    // 文本补全
    resp, err := client.CreateChatCompletion(
        ctx,
        openai.ChatCompletionRequest{
            Model: openai.GPT4o,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: "Hello, how are you?",
                },
            },
        },
    )
    
    if err != nil {
        panic(err)
    }
    
    fmt.Println(resp.Choices[0].Message.Content)
    
    // 流式响应
    stream, err := client.CreateChatCompletionStream(
        ctx,
        openai.ChatCompletionRequest{
            Model: openai.GPT4oMini,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: "Write a short story about a robot",
                },
            },
        },
    )
    defer stream.Close()
    
    for {
        chunk, err := stream.Recv()
        if err != nil {
            break
        }
        fmt.Print(chunk.Choices[0].Delta.Content)
    }
}
```

### 结构化输出

```go
package main

import (
    "context"
    "encoding/json"
    "fmt"
    
    "github.com/sashabaranov/go-openai"
)

type Article struct {
    Title    string   `json:"title"`
    Summary string   `json:"summary"`
    Tags    []string `json:"tags"`
    Author  string   `json:"author"`
}

func main() {
    client := openai.NewClient("your-api-key")
    ctx := context.Background()
    
    // 使用 JSON Schema 进行结构化输出
    schema := `{
        "type": "object",
        "properties": {
            "title": {"type": "string"},
            "summary": {"type": "string"},
            "tags": {
                "type": "array",
                "items": {"type": "string"}
            },
            "author": {"type": "string"}
        },
        "required": ["title", "summary", "tags", "author"]
    }`
    
    resp, err := client.CreateChatCompletion(
        ctx,
        openai.ChatCompletionRequest{
            Model: openai.GPT4o,
            Messages: []openai.ChatCompletionMessage{
                {
                    Role:    openai.ChatMessageRoleSystem,
                    Content: "You are a helpful assistant that outputs JSON.",
                },
                {
                    Role:    openai.ChatMessageRoleUser,
                    Content: "Write an article about Go programming language",
                },
            },
            ResponseFormat: &openai.ChatCompletionResponseFormat{
                Type: openai.ChatCompletionResponseFormatTypeJSONSchema,
                JSONSchema: &openai.ChatCompletionResponseFormatJSONSchema{
                    Name:        "article",
                    Description: "Article information",
                    Schema:       json.RawMessage(schema),
                },
            },
        },
    )
    
    if err != nil {
        panic(err)
    }
    
    content := resp.Choices[0].Message.Content
    
    var article Article
    json.Unmarshal([]byte(content), &article)
    
    fmt.Printf("Title: %s\n", article.Title)
    fmt.Printf("Summary: %s\n", article.Summary)
    fmt.Printf("Tags: %v\n", article.Tags)
}
```

---

## 实战项目结构

### Clean Architecture

```
myapp/
├── cmd/
│   └── server/
│       └── main.go           # 应用入口
├── internal/
│   ├── domain/               # 领域层
│   │   ├── entity/           # 实体
│   │   │   └── user.go
│   │   ├── repository/       # 仓储接口
│   │   │   └── user_repository.go
│   │   └── service/          # 领域服务
│   │       └── user_service.go
│   ├── usecase/              # 用例层
│   │   └── user_usecase.go
│   ├── adapter/              # 适配器层
│   │   ├── handler/          # HTTP 处理
│   │   │   └── user_handler.go
│   │   ├── repository/       # 仓储实现
│   │   │   └── postgres_user_repository.go
│   │   └── middleware/      # 中间件
│   │       └── auth.go
│   └── pkg/                 # 共享包
│       ├── errors/
│       │   └── errors.go
│       └── validator/
│           └── validator.go
├── config/
│   └── config.go
├── migrations/
│   └── 001_create_users.sql
├── scripts/
│   └── migrate.sh
├── Makefile
├── go.mod
└── go.sum
```

### 配置文件

```go
// config/config.go
package config

import (
    "os"
    "strconv"
)

type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Redis    RedisConfig
    JWT      JWTConfig
}

type ServerConfig struct {
    Port string
    Mode string // debug, release, test
}

type DatabaseConfig struct {
    Host     string
    Port     int
    User     string
    Password string
    DBName   string
    SSLMode  string
}

type RedisConfig struct {
    Host     string
    Port     int
    Password string
    DB       int
}

type JWTConfig struct {
    Secret     string
    Expiration int // hours
}

func Load() *Config {
    return &Config{
        Server: ServerConfig{
            Port: getEnv("SERVER_PORT", "8080"),
            Mode: getEnv("GIN_MODE", "debug"),
        },
        Database: DatabaseConfig{
            Host:     getEnv("DB_HOST", "localhost"),
            Port:     getEnvInt("DB_PORT", 5432),
            User:     getEnv("DB_USER", "postgres"),
            Password: getEnv("DB_PASSWORD", "postgres"),
            DBName:   getEnv("DB_NAME", "mydb"),
            SSLMode:  getEnv("DB_SSLMODE", "disable"),
        },
        Redis: RedisConfig{
            Host:     getEnv("REDIS_HOST", "localhost"),
            Port:     getEnvInt("REDIS_PORT", 6379),
            Password: getEnv("REDIS_PASSWORD", ""),
            DB:       getEnvInt("REDIS_DB", 0),
        },
        JWT: JWTConfig{
            Secret:     getEnv("JWT_SECRET", "secret"),
            Expiration: getEnvInt("JWT_EXPIRATION", 24),
        },
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}

func getEnvInt(key string, defaultValue int) int {
    if value := os.Getenv(key); value != "" {
        if intVal, err := strconv.Atoi(value); err == nil {
            return intVal
        }
    }
    return defaultValue
}
```

---

## 选型建议

### Go 适用场景

- ✅ 高性能 Web API
- ✅ 微服务架构
- ✅ 云原生应用
- ✅ CLI 工具
- ✅ 网络编程
- ✅ 容器和编排工具
- ✅ 数据库和缓存系统

### Go 不适用场景

- ❌ 复杂的业务逻辑（类型系统限制）
- ❌ 大量使用反射的场景
- ❌ 需要大量泛型操作的场景
- ❌ 前端开发（需要 WebAssembly）

### 版本推荐

| 场景 | 推荐版本 | 理由 |
|------|---------|------|
| **生产环境** | Go 1.22 | 稳定、广泛支持 |
| **新项目** | Go 1.23 | 最新特性、性能改进 |
| **学习** | Go 1.23 | 最新语法、更好错误信息 |

---

## 参考资料

| 资源 | 链接 |
|------|------|
| Go 官方文档 | https://go.dev/doc/ |
| 标准库参考 | https://pkg.go.dev/ |
| Go by Example | https://gobyexample.com/ |
| Go Wiki | https://github.com/golang/go/wiki |
| Effective Go | https://go.dev/doc/effective_go |

---

> [!TIP]
> 对于 AI 应用开发，推荐使用 Go 构建高性能推理服务和 API 网关，配合 Python 处理模型训练和数据处理。

---

> [!SUCCESS]
> 本文档全面介绍了 Go 的核心特性、1.23 新功能、并发模型、Web 开发、测试与优化以及在云原生和 AI 应用中的实战应用。Go 凭借其简洁的语法、卓越的性能和强大的并发支持，是构建现代后端服务的理想选择。
