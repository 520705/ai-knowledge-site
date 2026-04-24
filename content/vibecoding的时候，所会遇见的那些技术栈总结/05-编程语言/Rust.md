# Rust 完全指南

> [!NOTE]
> 本文档最后更新于 **2026年4月**，涵盖 Rust 2024 Edition 新特性、所有权系统、并发模型、Web 开发、以及 Rust 在 AI 和系统编程中的核心应用。

---

## 目录

1. [[#Rust 概述与核心优势]]
2. [[#Rust 2024 Edition 新特性]]
3. [[#所有权与生命周期]]
4. [[#基础语法与数据结构]]
5. [[#trait 与泛型]]
6. [[#错误处理]]
7. [[#并发编程]]
8. [[#异步编程 async/await]]
9. [[#Web 开发与框架]]
10. [[#数据库操作]]
11. [[#AI 应用集成]]
12. [[#测试与性能优化]]
13. [[#工具链与生态系统]]
14. [[#选型建议]]

---

## Rust 概述与核心优势

### 为什么选择 Rust

Rust 由 Mozilla 研究院的 Graydon Hoare 于 2010 年创造。Rust 的设计目标是提供 C++ 的性能和控制力，同时消除内存安全漏洞。Rust 连续多年在 Stack Overflow 开发者调查中被评为「最受喜爱」的编程语言。

Rust 的核心优势体现在三个维度：

**内存安全**：Rust 的所有权系统和借用检查器在编译时就能消除空指针解引用、数据竞争等内存安全问题，无需垃圾回收器。

**零成本抽象**：Rust 提供的高级抽象（trait、泛型、闭包）在编译时会被优化为机器码，不产生运行时开销。

**并发安全**：Rust 的类型系统保证线程间数据传递的安全性，数据竞争在编译时就能被检测。

### Rust 在 AI 编程中的独特价值

在 AI 应用开发中，Rust 保持着独特的优势：

- **高性能推理引擎**：用 Rust 编写的 AI 推理服务可以高效调用 ONNX、GGUF 等格式的模型
- **边缘计算**：Rust 的小体积和低内存占用使其成为边缘 AI 的理想选择
- **Python 绑定**：PyO3 允许用 Rust 编写高性能的 Python 扩展
- **LLM 服务**：llm-chain、distill 等框架支持构建 LLM 应用

---

## Rust 2024 Edition 新特性

Rust 2024 Edition 是 Rust 语言的重要里程碑，带来了多项改进。

### 核心新特性

#### 1. Rust 2024 Edition 预览特性

```rust
// Rust 2024 Edition 启用的特性

// 1. 改进的 trait 解析
// 现在可以更清晰地处理复杂的 trait 约束

use std::collections::HashMap;

fn process<K, V>(map: &HashMap<K, V>)
where
    K: std::hash::Hash + Eq,
    V: Clone,
{
    // 处理逻辑
}

// 2. 改进的 const泛型
const fn ArraySize<T, const N: usize>() -> [T; N] {
    [std::mem::zeroed::<T>(); N]
}

// 3. 改进的模式匹配
type Tree<T> = Option<Box<TreeNode<T>>>;

struct TreeNode<T> {
    value: T,
    left: Tree<T>,
    right: Tree<T>,
}
```

#### 2. 异步改进

```rust
// 异步函数的返回值改进
async fn fetch_data() -> Result<String, reqwest::Error> {
    let response = reqwest::get("https://api.example.com").await?;
    response.text().await
}

// async trait (稳定化)
trait AsyncTrait {
    async fn fetch(&self) -> Result<String, Error>;
}

// 使用 Box<dyn AsyncTrait>
async fn call_async_trait(obj: &dyn AsyncTrait) -> Result<String, Error> {
    obj.fetch().await
}
```

#### 3. 改进的错误处理

```rust
// ? 操作符与更多类型
use std::num::ParseIntError;

fn parse_and_add(a: &str, b: &str) -> Result<i32, ParseIntError> {
    let a: i32 = a.parse()?;
    let b: i32 = b.parse()?;
    Ok(a + b)
}

// anyhow 库的广泛使用
use anyhow::{Context, Result};

fn read_config() -> Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    
    let config: Config = toml::from_str(&content)
        .context("Failed to parse config")?;
    
    Ok(config)
}
```

### 版本特性对比表

| 特性 | 2018 | 2021 | 2024 |
|------|-------|-------|-------|
| **所有权系统** | 基础 | 完善 | 优化 |
| **async/await** | 稳定 | 稳定 | 稳定 |
| **const 泛型** | 基础 | 改进 | 完善 |
| **let-chaining** | ❌ | ✅ | ✅ |
| **async trait** | ❌ | 预览 | 稳定 |
| **GAT (泛型关联类型)** | 预览 | 稳定 | 稳定 |
| **负 trait bound** | ❌ | 预览 | 预览 |
| **泛型 const 表达式** | ❌ | ❌ | 改进 |

---

## 所有权与生命周期

### 所有权规则

```rust
fn main() {
    // 所有权规则：
    // 1. 每个值有一个所有者
    // 2. 同一时间只有一个所有者
    // 3. 当所有者离开作用域，值被丢弃
    
    let s1 = String::from("hello");  // s1 拥有字符串
    let s2 = s1;                     // 所有权转移到 s2
    
    // println!("{}", s1);  // 错误！s1 不再有效
    println!("{}", s2);              // 正确！s2 是所有者
    
    // 克隆（深拷贝）
    let s3 = s2.clone();
    println!("{} {}", s2, s3);     // 两个都有效
    
    // 引用（借用）
    let s4 = String::from("world");
    let len = calculate_length(&s4);  // 借用 s4
    println!("{} 的长度是 {}", s4, len);  // s4 仍然有效
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

### 借用规则

```rust
fn main() {
    let mut s = String::from("hello");
    
    // 可变借用
    change(&mut s);
    println!("{}", s);
    
    // 借用规则：
    // 1. 可以有任意数量的不可变借用
    // 2. 只能有一个可变借用
    // 3. 可变和不可变借用不能同时存在
    
    let r1 = &s;       // 第一个不可变借用
    let r2 = &s;       // 第二个不可变借用（可以）
    // let r3 = &mut s; // 错误！不能同时有可变借用
    
    println!("{} and {}", r1, r2);
    
    // r1 和 r2 在这里之后不再使用，可以创建可变借用
    let r3 = &mut s;   // 现在可以
}

fn change(s: &mut String) {
    s.push_str(", world");
}
```

### 生命周期标注

```rust
// 生命周期标注语法
// &'a str 表示一个引用，其生命周期不短于 'a

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

fn main() {
    let s1 = String::from("long string");
    let result;
    {
        let s2 = String::from("xyz");
        result = longest(s1.as_str(), s2.as_str());
        println!("最长的: {}", result);  // 正确
    }
    // println!("{}", result);  // 错误！s2 已离开作用域
}

// 结构体中的生命周期
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
    
    fn announce_and_return(&self, announcement: &str) -> &str {
        println!("公告: {}", announcement);
        self.part
    }
}
```

### 生命周期省略规则

```rust
// 编译器自动推断生命周期的规则：
// 1. 每个引用参数获得自己的生命周期
// 2. 如果只有一个输入生命周期，所有输出继承它
// 3. 如果有 &self 或 &mut self，输出生命周期继承 self

fn first_word(s: &str) -> &str {  // 自动推断
    // ...
    ""
}

fn get_name(&self) -> &str {  // 自动继承 self 生命周期
    &self.name
}
```

---

## 基础语法与数据结构

### 变量与类型

```rust
fn main() {
    // 变量声明
    let x = 5;          // 不可变
    let mut y = 6;      // 可变
    y = 7;              // 正确
    
    // 类型标注
    let z: i32 = 10;
    let f: f64 = 3.14;
    let b: bool = true;
    let c: char = 'A';
    
    // 元组
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let (tx, ty, tz) = tup;         // 解构
    let first = tup.0;                // 索引访问
    
    // 数组
    let arr: [i32; 5] = [1, 2, 3, 4, 5];
    let first = arr[0];
    
    // 切片
    let slice = &arr[1..4];  // [2, 3, 4]
    
    // 字符串
    let s = String::from("hello");
    let s2 = "hello";        // 字符串字面量（&str）
}
```

### 函数

```rust
// 函数定义
fn add(a: i32, b: i32) -> i32 {
    a + b  // 注意：没有分号的返回值
}

fn process(x: i32) -> i32 {
    let result = if x > 0 { x * 2 } else { x };
    result  // 或直接写表达式
}

// 方法
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    // 关联函数（构造函数）
    fn new(width: u32, height: u32) -> Self {
        Self { width, height }
    }
    
    // 方法
    fn area(&self) -> u32 {
        self.width * self.width
    }
    
    // 可变方法
    fn scale(&mut self, factor: u32) {
        self.width *= factor;
        self.height *= factor;
    }
    
    // 带多个参数的方法
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

### 枚举与模式匹配

```rust
// 枚举定义
enum Message {
    Quit,                           // 无数据
    Move { x: i32, y: i32 },      // 命名字段
    Write(String),                 // 元组结构
    ChangeColor(u8, u8, u8),     // RGB
}

// impl 枚举
impl Message {
    fn call(&self) {
        match self {
            Message::Quit => println!("Quit"),
            Message::Move { x, y } => println!("Move to ({}, {})", x, y),
            Message::Write(s) => println!("Write: {}", s),
            Message::ChangeColor(r, g, b) => println!("Color: {}, {}, {}", r, g, b),
        }
    }
}

fn main() {
    let m = Message::Move { x: 10, y: 20 };
    m.call();
    
    // Option 枚举（替代 null）
    let some_value: Option<i32> = Some(5);
    let no_value: Option<i32> = None;
    
    if let Some(x) = some_value {
        println!("Got value: {}", x);
    }
    
    // match 表达式
    match some_value {
        Some(x) => println!("Value: {}", x),
        None => println!("No value"),
    }
}
```

### 结构体

```rust
// 普通结构体
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// 元组结构体
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

// 单元结构体（用于 trait 实现）
struct AlwaysEqual;

// 结构体更新
fn main() {
    let user1 = User {
        email: String::from("alice@example.com"),
        username: String::from("alice123"),
        active: true,
        sign_in_count: 1,
    };
    
    // 结构体更新语法
    let user2 = User {
        email: String::from("bob@example.com"),
        ..user1  // 其余字段从 user1 复制
    };
}
```

---

## trait 与泛型

### trait 定义

```rust
// trait 定义
pub trait Summary {
    fn summarize(&self) -> String;
    
    // 默认实现
    fn summarize_author(&self) -> String {
        String::from("(Unknown)")
    }
}

// 为结构体实现 trait
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", 
            self.headline, 
            self.author, 
            self.location)
    }
    
    fn summarize_author(&self) -> String {
        format!("@{}", self.author)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

### trait 约束

```rust
// trait 约束语法
fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// 完整约束语法
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// 多个约束
use std::fmt::{Display, Debug};

fn some_function<T, U>(t: &T, u: &U)
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}

// 使用 trait 作为返回类型
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

### 泛型函数

```rust
// 泛型函数
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    
    largest
}

// 泛型结构体
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

// 泛型枚举
enum Result<T, E> {
    Ok(T),
    Err(E),
}

// impl 块针对具体类型
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

### 高级 trait

```rust
// 关联类型
pub trait Iterator {
    type Item;
    
    fn next(&mut self) -> Option<Self::Item>;
}

// 默认泛型类型
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

// supertrait
use std::fmt::*;

trait Person: Display {
    fn name(&self) -> &str;
}

impl Person for User {
    fn name(&self) -> &str {
        &self.name
    }
}
```

---

## 错误处理

### Option 和 Result

```rust
fn main() {
    // Option 处理
    let some_value = Some(5);
    let no_value: Option<i32> = None;
    
    // match 处理
    match some_value {
        Some(x) => println!("Got: {}", x),
        None => println!("None"),
    }
    
    // if let
    if let Some(x) = some_value {
        println!("Got: {}", x);
    }
    
    // unwrap 和 expect
    let x = some_value.unwrap();       // 危险！
    let x = some_value.expect("should have a value");  // 更好
    
    // unwrap_or 和 unwrap_or_else
    let x = no_value.unwrap_or(0);
    let x = no_value.unwrap_or_else(|| {
        // 懒加载
        expensive_computation()
    });
    
    // map
    let x = some_value.map(|v| v * 2);
    
    // and_then
    let x = some_value.and_then(|v| {
        if v > 0 {
            Some(v * 2)
        } else {
            None
        }
    });
}
```

### Result 和错误传播

```rust
use std::io;
use std::num::ParseIntError;

// 自定义错误类型
#[derive(Debug)]
enum MyError {
    Io(io::Error),
    Parse(ParseIntError),
    Custom(String),
}

impl From<io::Error> for MyError {
    fn from(err: io::Error) -> Self {
        MyError::Io(err)
    }
}

impl From<ParseIntError> for MyError {
    fn from(err: ParseIntError) -> Self {
        MyError::Parse(err)
    }
}

// 使用 ? 操作符
fn read_and_parse(path: &str) -> Result<i32, MyError> {
    let content = std::fs::read_to_string(path)?;  // 错误自动转换
    let number: i32 = content.trim().parse()?;    // 错误自动转换
    Ok(number)
}

// main 函数返回 Result
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let number = read_and_parse("number.txt")?;
    println!("Number: {}", number);
    Ok(())
}
```

### anyhow 和 thiserror

```rust
// 使用 anyhow 进行灵活的错误处理
use anyhow::{Context, Result};

fn read_config() -> Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    
    let config: Config = toml::from_str(&content)
        .with_context(|| format!("Failed to parse config from {}", "config.toml"))?;
    
    Ok(config)
}

// 使用 thiserror 定义自定义错误
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DatabaseError {
    #[error("Connection failed: {0}")]
    ConnectionFailed(String),
    
    #[error("Query failed: {0}")]
    QueryFailed(String),
    
    #[error("Not found: {entity} with id {id}")]
    NotFound { entity: String, id: u64 },
}

fn find_user(id: u64) -> Result<User, DatabaseError> {
    if id == 0 {
        Err(DatabaseError::NotFound { 
            entity: "User".to_string(), 
            id 
        })
    } else {
        Ok(User { id, name: "Alice".to_string() })
    }
}
```

---

## 并发编程

### 线程

```rust
use std::thread;
use std::time::Duration;

fn main() {
    // 创建线程
    let handle = thread::spawn(|| {
        for i in 0..5 {
            println!("Spawned thread: {}", i);
            thread::sleep(Duration::from_millis(10));
        }
    });
    
    // 等待线程完成
    handle.join().unwrap();
    
    // 线程参数和返回值
    let data = vec![1, 2, 3];
    let handle = thread::spawn(move || {
        let sum: i32 = data.iter().sum();
        sum
    });
    
    let result = handle.join().unwrap();
    println!("Sum: {}", result);
}
```

### 消息传递

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // 创建通道
    let (tx, rx) = mpsc::channel();
    
    // 发送端克隆
    let tx1 = tx.clone();
    
    // 线程 1
    thread::spawn(move || {
        tx.send(1).unwrap();
        tx.send(2).unwrap();
    });
    
    // 线程 2
    thread::spawn(move || {
        tx1.send(3).unwrap();
        tx1.send(4).unwrap();
    });
    
    // 接收
    for received in rx {
        println!("Got: {}", received);
    }
    
    // 多次接收
    let received = rx.recv().unwrap();
    let received = rx.try_recv();  // 非阻塞
}
```

### 互斥锁

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    // Arc: 原子引用计数（线程安全）
    // Mutex: 互斥锁
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

### RwLock 和 Atomic

```rust
use std::sync::{Arc, RwLock};
use std::sync::atomic::{AtomicI32, Ordering};

fn main() {
    // RwLock: 读写锁
    let data = Arc::new(RwLock::new(vec![1, 2, 3]));
    
    // 读锁
    let read_handle = {
        let data = data.read().unwrap();
        println!("Read: {:?}", data);
        drop(data);
    };
    
    // 写锁
    let mut write_handle = {
        let mut data = data.write().unwrap();
        data.push(4);
        drop(data);
    };
    
    // Atomic 类型
    let atomic_counter = Arc::new(AtomicI32::new(0));
    
    let handles: Vec<_> = (0..10).map(|_| {
        let counter = Arc::clone(&atomic_counter);
        thread::spawn(move || {
            counter.fetch_add(1, Ordering::SeqCst);
        })
    }).collect();
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Atomic counter: {}", atomic_counter.load(Ordering::SeqCst));
}
```

---

## 异步编程 async/await

### 异步函数基础

```rust
use tokio;

#[tokio::main]
async fn main() {
    // 异步函数
    let result = async_function().await;
    println!("Result: {}", result);
    
    // 并发执行
    let (a, b) = tokio::join!(
        async_operation_1(),
        async_operation_2()
    );
    
    println!("{} {}", a, b);
}

async fn async_function() -> i32 {
    // 模拟异步操作
    tokio::time::sleep(tokio::time::Duration::from_secs(1)).await;
    42
}

async fn async_operation_1() -> String {
    tokio::time::sleep(tokio::time::Duration::from_millis(100)).await;
    "Operation 1".to_string()
}

async fn async_operation_2() -> String {
    tokio::time::sleep(tokio::time::Duration::from_millis(50)).await;
    "Operation 2".to_string()
}
```

### async/await 进阶

```rust
use tokio::time::{sleep, Duration};
use futures::future::join_all;

// select! 并发执行多个异步操作
async fn select_example() {
    tokio::select! {
        result1 = operation_a() => {
            println!("A completed: {}", result1);
        }
        result2 = operation_b() => {
            println!("B completed: {}", result2);
        }
        _ = sleep(Duration::from_secs(3)) => {
            println!("Timeout!");
        }
    }
}

// 并发任务
async fn concurrent_tasks() {
    let tasks = vec![
        tokio::spawn(async { 1 }),
        tokio::spawn(async { 2 }),
        tokio::spawn(async { 3 }),
    ];
    
    let results = join_all(tasks).await;
    println!("Results: {:?}", results);
}

// 流处理
use futures::StreamExt;

async fn stream_processing() {
    let stream = tokio_stream::iter(vec![1, 2, 3, 4, 5]);
    
    stream
        .for_each(|n| async move {
            println!("Processing: {}", n);
        })
        .await;
}
```

### Pin 和 Unpin

```rust
use std::pin::Pin;
use std::future::Future;

// Pin 的必要性
async fn async_example() {
    let future = async {
        // ...
    };
    
    // 自我引用的 future 需要 Pin
    let pinned: Pin<Box<dyn Future<Output = ()>>> = Box::pin(future);
    
    pinned.await;
}

// 使用 Arc<Mutex<T>>
use std::sync::{Arc, Mutex};

async fn shared_state() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..5 {
        let counter = Arc::clone(&counter);
        let handle = tokio::spawn(async move {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.await.unwrap();
    }
    
    println!("Final: {}", *counter.lock().unwrap());
}
```

---

## Web 开发与框架

### Axum 框架

```rust
use axum::{
    extract::{Path, State},
    http::StatusCode,
    response::Json,
    routing::{get, post},
    Router,
};
use serde::{Deserialize, Serialize};
use std::sync::Arc;

// App State
#[derive(Clone)]
struct AppState {
    db: Arc<Database>,
}

// 响应模型
#[derive(Serialize)]
struct User {
    id: u64,
    name: String,
    email: String,
}

// 请求模型
#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Serialize, Deserialize)]
struct Database {
    users: Vec<User>,
}

impl Database {
    fn new() -> Self {
        Self { users: vec![] }
    }
    
    fn create_user(&mut self, name: String, email: String) -> User {
        let id = self.users.len() as u64 + 1;
        let user = User { id, name, email };
        self.users.push(user.clone());
        user
    }
    
    fn get_user(&self, id: u64) -> Option<&User> {
        self.users.iter().find(|u| u.id == id)
    }
}

// 处理器
async fn get_users(State(state): State<Arc<AppState>>) -> Json<Vec<User>> {
    let users = state.db.users.clone();
    Json(users)
}

async fn get_user(
    Path(id): Path<u64>,
    State(state): State<Arc<AppState>>,
) -> Result<Json<User>, StatusCode> {
    state.db.get_user(id)
        .map(|u| Json(u.clone()))
        .ok_or(StatusCode::NOT_FOUND)
}

async fn create_user(
    State(state): State<Arc<AppState>>,
    Json(payload): Json<CreateUser>,
) -> StatusCode {
    let user = state.db.create_user(payload.name, payload.email);
    println!("Created: {:?}", user);
    StatusCode::CREATED
}

#[tokio::main]
async fn main() {
    let state = Arc::new(AppState {
        db: Arc::new(Database::new()),
    });
    
    let app = Router::new()
        .route("/users", get(get_users).post(create_user))
        .route("/users/:id", get(get_user))
        .with_state(state);
    
    let listener = tokio::net::TcpListener::bind("127.0.0.1:3000")
        .await
        .unwrap();
    
    println!("Server running on http://127.0.0.1:3000");
    axum::serve(listener, app).await.unwrap();
}
```

### Actix-web

```rust
use actix_web::{web, App, HttpResponse, HttpServer, Responder};
use serde::{Deserialize, Serialize};

#[derive(Serialize)]
struct User {
    id: u64,
    name: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

async fn index() -> impl Responder {
    "Hello, World!"
}

async fn get_users() -> impl Responder {
    let users = vec![
        User { id: 1, name: "Alice".to_string() },
        User { id: 2, name: "Bob".to_string() },
    ];
    web::Json(users)
}

async fn create_user(
    body: web::Json<CreateUser>,
) -> impl Responder {
    let user = User {
        id: 3,
        name: body.name.clone(),
    };
    HttpResponse::Created().json(user)
}

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::get().to(index))
            .route("/users", web::get().to(get_users))
            .route("/users", web::post().to(create_user))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

---

## 数据库操作

### SQLx

```rust
use sqlx::{postgres::PgPoolOptions, PgPool, Row};
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize)]
struct User {
    id: i32,
    name: String,
    email: String,
}

#[tokio::main]
async fn main() -> Result<(), sqlx::Error> {
    // 创建连接池
    let pool = PgPoolOptions::new()
        .max_connections(5)
        .connect("postgres://user:password@localhost/mydb")
        .await?;
    
    // 查询单行
    let user: (i32, String, String) = sqlx::query_as(
        "SELECT id, name, email FROM users WHERE id = $1"
    )
    .bind(1)
    .fetch_one(&pool)
    .await?;
    
    // 查询多行
    let users: Vec<(i32, String, String)> = sqlx::query_as(
        "SELECT id, name, email FROM users"
    )
    .fetch_all(&pool)
    .await?;
    
    // 命名类型查询
    let user: User = sqlx::query_as!(
        User,
        "SELECT id, name, email FROM users WHERE id = $1",
        1
    )
    .fetch_one(&pool)
    .await?;
    
    // 插入
    let result = sqlx::query(
        "INSERT INTO users (name, email) VALUES ($1, $2)"
    )
    .bind("Charlie")
    .bind("charlie@example.com")
    .execute(&pool)
    .await?;
    
    println!("Inserted rows: {}", result.rows_affected());
    
    // 事务
    let mut tx = pool.begin().await?;
    
    sqlx::query("UPDATE users SET name = $1 WHERE id = $2")
        .bind("New Name")
        .bind(1)
        .execute(&mut tx)
        .await?;
    
    tx.commit().await?;
    
    Ok(())
}
```

### Diesel

```rust
// schema.rs
diesel::table! {
    users (id) {
        id -> Int4,
        name -> Varchar,
        email -> Varchar,
        created_at -> Timestamp,
    }
}

// models.rs
use diesel::prelude::*;

#[derive(Queryable, Selectable)]
#[diesel(table_name = users)]
pub struct User {
    pub id: i32,
    pub name: String,
    pub email: String,
    pub created_at: chrono::NaiveDateTime,
}

#[derive(Insertable)]
#[diesel(table_name = users)]
pub struct NewUser<'a> {
    pub name: &'a str,
    pub email: &'a str,
}

// lib.rs
use diesel::prelude::*;

pub fn create_user<'a>(
    conn: &mut PgConnection,
    name: &'a str,
    email: &'a str,
) -> QueryResult<User> {
    use crate::schema::users;
    
    let new_user = NewUser { name, email };
    
    diesel::insert_into(users::table)
        .values(&new_user)
        .get_result(conn)
}

pub fn get_users(conn: &mut PgConnection) -> QueryResult<Vec<User>> {
    use crate::schema::users::dsl::*;
    
    users.load(conn)
}
```

---

## AI 应用集成

### 通用 LLM 调用

```rust
use reqwest;
use serde::{Deserialize, Serialize};
use anyhow::Result;

#[derive(Serialize)]
struct ChatMessage {
    role: String,
    content: String,
}

#[derive(Serialize)]
struct ChatRequest {
    model: String,
    messages: Vec<ChatMessage>,
}

#[derive(Deserialize)]
struct ChatResponse {
    choices: Vec<Choice>,
}

#[derive(Deserialize)]
struct Choice {
    message: ChatMessage,
}

#[tokio::main]
async fn chat(messages: Vec<ChatMessage>) -> Result<String> {
    let client = reqwest::Client::new();
    
    let request = ChatRequest {
        model: "gpt-4o".to_string(),
        messages,
    };
    
    let response = client
        .post("https://api.openai.com/v1/chat/completions")
        .header("Authorization", "Bearer YOUR_API_KEY")
        .json(&request)
        .send()
        .await?;
    
    let chat_response: ChatResponse = response.json().await?;
    
    Ok(chat_response.choices[0].message.content.clone())
}
```

### Hugging Face Transformers (Candle)

```rust
// 使用 candle 加载模型
use candle::{Device, Tensor, Result};
use candle_transformers::models::bert::{BertModel, Config};

fn main() -> Result<()> {
    let device = Device::Cpu;
    
    // 加载配置
    let config = Config::from_pretrained("bert-base-uncased")?;
    
    // 创建模型
    let model = BertModel::load(&device, &config)?;
    
    // 准备输入
    let input_ids = Tensor::new(&[1i64, 2, 3, 4, 5], &device)?;
    
    // 前向传播
    let output = model.forward(&input_ids, None, None)?;
    
    println!("Output shape: {:?}", output.shape());
    
    Ok(())
}
```

---

## 测试与性能优化

### 单元测试

```rust
// src/lib.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_add() {
        assert_eq!(add(2, 2), 4);
    }
    
    #[test]
    fn test_add_negative() {
        assert_eq!(add(-1, -1), -2);
    }
    
    #[test]
    #[should_panic(expected = "assertion failed")]
    fn test_add_fail() {
        assert_eq!(add(2, 2), 5);
    }
    
    #[test]
    fn test_with_result() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err("2 + 2 should equal 4".to_string())
        }
    }
}
```

### 集成测试

```rust
// tests/integration_test.rs
use my_crate::*;

#[test]
fn test_integration() {
    let result = my_function();
    assert!(result.is_ok());
}

#[tokio::test]
async fn test_async() {
    let result = async_function().await;
    assert!(result.is_ok());
}
```

### 性能测试

```rust
#[cfg(test)]
mod benches {
    use test::Bencher;
    
    #[bench]
    fn bench_iteration(b: &mut Bencher) {
        let data: Vec<i32> = (0..1000).collect();
        
        b.iter(|| {
            data.iter().map(|x| x * 2).collect::<Vec<_>>()
        });
    }
    
    #[bench]
    fn bench_loop(b: &mut Bencher) {
        let data: Vec<i32> = (0..1000).collect();
        
        b.iter(|| {
            let mut result = Vec::new();
            for x in &data {
                result.push(x * 2);
            }
            result
        });
    }
}
```

---

## 工具链与生态系统

### Cargo

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2024"
authors = ["Author Name <author@example.com>"]

[dependencies]
# 异步运行时
tokio = { version = "1", features = ["full"] }

# Web 框架
axum = "0.7"

# 序列化
serde = { version = "1", features = ["derive"] }
serde_json = "1"

# 错误处理
anyhow = "1"
thiserror = "1"

# 数据库
sqlx = { version = "0.7", features = ["runtime-tokio", "postgres"] }

# 日志
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }

# 测试
#[dev-dependencies]
criterion = "0.5"
tokio-test = "0.4"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
```

### 常用命令

```bash
# 创建新项目
cargo new my_project
cargo init

# 构建
cargo build
cargo build --release

# 运行
cargo run
cargo run --bin <bin_name>

# 测试
cargo test
cargo test --doc
cargo test --lib
cargo test --test <test_name>

# 格式化
cargo fmt

# 检查
cargo check
cargo clippy

# 依赖管理
cargo add <crate>
cargo remove <crate>
cargo update

# 文档
cargo doc --open

# 发布
cargo publish --dry-run
cargo publish
```

---

## 选型建议

### Rust 适用场景

- ✅ 系统编程（OS、驱动、游戏引擎）
- ✅ Web API 和微服务
- ✅ CLI 工具
- ✅ WebAssembly
- ✅ 嵌入式开发
- ✅ 高性能服务
- ✅ AI 推理引擎

### Rust 不适用场景

- ❌ 快速原型开发
- ❌ 简单脚本
- ❌ 需要大量动态特性的场景
- ❌ GC 友好的场景（Rust 没有垃圾回收）
- ❌ 需要大量反射的场景

### 版本推荐

| 场景 | 推荐版本 | 理由 |
|------|---------|------|
| **生产环境** | Rust 1.75+ | 稳定、async trait 稳定 |
| **新项目** | Rust 2024 Edition | 最新语法、标准库改进 |
| **学习** | Rust 1.75+ | 文档完善、生态成熟 |

---

## 参考资料

| 资源 | 链接 |
|------|------|
| Rust 官方文档 | https://doc.rust-lang.org/ |
| The Book | https://doc.rust-lang.org/book/ |
| Rust by Example | https://doc.rust-lang.org/rust-by-example/ |
| Rustlings | https://github.com/rust-lang/rustlings/ |
| crates.io | https://crates.io/ |
| awesome-rust | https://github.com/rust-unofficial/awesome-rust |

---

> [!TIP]
> 对于 AI 应用开发，推荐使用 Rust 构建高性能推理服务和 Python 扩展（通过 PyO3），配合 Python 处理模型训练和数据处理。

---

## 附录：Rust 高级配置与生态工具

### Cargo.toml 完整配置

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2024"
authors = ["Author <author@example.com>"]
description = "A comprehensive Rust project"
license = "MIT OR Apache-2.0"
homepage = "https://github.com/author/my_project"
repository = "https://github.com/author/my_project"
documentation = "https://docs.rs/my_project"
readme = "README.md"
keywords = ["rust", "web", "async"]
categories = ["web-programming", "asynchronous"]

[package.metadata.docs.rs]
all-features = true
rustdoc-args = ["--cfg", "docsrs"]

[workspace]
members = ["crates/core", "crates/cli", "crates/api"]

[lib]
name = "my_project"
path = "src/lib.rs"

[[bin]]
name = "my_project"
path = "src/main.rs"

[dependencies]
# 异步运行时
tokio = { version = "1.40", features = ["full"] }
async-trait = "0.1"
futures = "0.3"
futures-util = "0.3"

# Web 框架
axum = "0.7"
tower = "0.4"
tower-http = { version = "0.5", features = ["cors", "trace"] }

# 数据库
sqlx = { version = "0.8", features = ["runtime-tokio", "postgres", "uuid", "chrono", "rust_decimal"] }
sqlx-cli = { version = "0.8", optional = true }
diesel = { version = "2.2", features = ["postgres", "r2d2", "chrono", "uuid"] }

# 序列化
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
serde_yaml = "0.9"
toml = "0.8"

# 错误处理
anyhow = "1.0"
thiserror = "2.0"
eyre = "0.6"
color-eyre = "0.6"

# 日志和追踪
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter", "json"] }
tracing-opentelemetry = "0.25"
opentelemetry = { version = "0.25", features = ["trace"] }
opentelemetry-otlp = { version = "0.15", features = ["tonic"] }

# 验证
validator = { version = "0.18", features = ["derive"] }

# 密码学
bcrypt = "0.16"
argon2 = "0.5"
jwt = "0.16"
jsonwebtoken = "9.3"

# HTTP 客户端
reqwest = { version = "0.12", features = ["json", "cookies"] }

# 日期时间
chrono = { version = "0.4", features = ["serde"] }
time = "0.3"

# UUID
uuid = { version = "1.10", features = ["v4", "serde"] }

# 唯一 ID 生成
ulid = "1.1"

# 配置管理
config = "0.14"
dotenv = "0.15"

# 缓存
dashmap = "5.5"
moka = "0.12"

# 限流
governor = "0.7"
ratelimit = "0.9"

# 邮件
lettre = "0.11"

# API 文档
utoipa = "4.2"
utoipa-swagger-ui = "6.0"

[dev-dependencies]
# 测试
tokio-test = "0.4"
proptest = "1.4"
criterion = { version = "0.5", features = ["html_reports"] }
fake = { version = "3.0", features = ["derive"] }
futures-test = "0.3"

# 模拟
mockall = "0.13"
wiremock = "1.3"

# 集成测试
testcontainers = "1.20"
testcontainers-postgres = "1.20"

# 变异测试
mutagen = "25.1"

# 模糊测试
cargo-fuzz = "0.11"

# 基准测试比较
prettybench = "0.2"

# 日志测试
tracing-test = "0.2"

[build-dependencies]
protox = "0.3"
prost-build = "0.13"

[profile.dev]
opt-level = 0
debug = true
split-debuginfo = "unpacked"

[profile.dev.package."*"]
opt-level = 0

[profile.release]
opt-level = 3
lto = "thin"
codegen-units = 1
panic = "abort"
strip = "symbols"

[profile.release.package."*"]
opt-level = 3
lto = true

[profile.bench]
inherits = "release"
debug = true

[profile.test]
opt-level = 0
debug = true
```

### 项目结构最佳实践

```
my_project/
├── Cargo.toml              # Workspace 根配置
├── Cargo.lock              # 锁定依赖版本
├── rust-toolchain.toml     # Rust 版本配置
├── .cargo/
│   └── config.toml         # Cargo 配置
├── src/
│   ├── lib.rs             # 库入口
│   ├── main.rs            # 二进制入口
│   ├── bin/               # 其他二进制
│   ├── prelude.rs         # 公共导入
│   ├── error.rs           # 错误类型定义
│   └── macros.rs          # 宏定义
├── crates/
│   ├── core/              # 核心业务逻辑
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── domain/    # 领域模型
│   │       ├── service/   # 服务层
│   │       └── repository/ # 仓储接口
│   ├── api/               # API 层
│   │   ├── Cargo.toml
│   │   └── src/
│   │       ├── lib.rs
│   │       ├── routes/    # 路由
│   │       ├── handlers/  # 请求处理
│   │       ├── middleware/# 中间件
│   │       └── dto/       # 数据传输对象
│   └── cli/               # CLI 工具
│       ├── Cargo.toml
│       └── src/
│           └── main.rs
├── migrations/            # 数据库迁移
│   └── 2024-01-01-000000_create_users
├── tests/                # 集成测试
│   ├── common/
│   │   └── mod.rs
│   ├── api_tests.rs
│   └── integration_tests.rs
├── benches/              # 基准测试
├── examples/             # 示例代码
├── docs/                 # 文档
├── scripts/              # 构建脚本
│   └── docker-build.sh
├── Dockerfile
├── docker-compose.yml
├── .env.example
├── .gitignore
└── README.md
```

### 常用 Cargo 命令

```bash
# 项目管理
cargo new my_project           # 创建新项目
cargo init                     # 初始化现有目录
cargo generate-lockfile        # 生成锁文件
cargo update                   # 更新依赖
cargo search <crate>           # 搜索 crates.io
cargo info <crate>            # 查看 crate 信息

# 构建和运行
cargo build                   # 调试构建
cargo build --release         # 发布构建
cargo run                     # 运行
cargo run --bin <name>        # 运行特定二进制
cargo run --example <name>    # 运行示例

# 测试
cargo test                    # 运行所有测试
cargo test <test_name>        # 运行特定测试
cargo test --lib              # 仅库测试
cargo test --doc              # 文档测试
cargo test --test <name>      # 集成测试
cargo bench                   # 基准测试

# 代码质量
cargo check                   # 类型检查（快速）
cargo clippy                  # 代码审查
cargo fmt                     # 格式化
cargo fix                     # 自动修复
cargo audit                   # 安全审计

# 文档
cargo doc                     # 生成文档
cargo doc --open              # 生成并打开
cargo doc --no-deps           # 仅项目文档

# 清理和依赖
cargo clean                   # 清理构建产物
cargo tree                    # 依赖树
cargo tree --invert          # 反向依赖树
cargo outdated               # 检查过期依赖

# 发布
cargo package                 # 创建 crate 包
cargo publish                 # 发布到 crates.io
cargo yank --version <ver>   # 撤销版本

# 其他
cargo install <crate>        # 安装工具
cargo uninstall <crate>      # 卸载工具
cargo metadata               # 输出项目元数据（JSON）
```

### Docker 部署配置

```dockerfile
# Stage 1: Builder
FROM rust:1.82 AS builder

WORKDIR /app

# 安装依赖
RUN apt-get update && apt-get install -y \
    pkg-config \
    libssl-dev \
    protobuf-compiler \
    && rm -rf /var/lib/apt/lists/*

# 复制代码
COPY Cargo.toml Cargo.lock* ./
COPY src ./src
COPY crates ./crates

# 下载依赖（利用缓存）
RUN mkdir src && \
    echo "fn main() {}" > src/main.rs && \
    echo "fn main() {}" > crates/core/src/lib.rs && \
    cargo build --release && \
    rm -rf src crates

# 复制源代码并构建
COPY . .
RUN cargo build --release --bin my_project

# Stage 2: Runtime
FROM debian:bookworm-slim AS runtime

WORKDIR /app

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    ca-certificates \
    libssl3 \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# 复制二进制文件
COPY --from=builder /app/target/release/my_project /usr/local/bin/
COPY --from=builder /app/migrations /app/migrations
COPY --from=builder /app/.env.example /app/.env

# 创建非 root 用户
RUN groupadd -r appgroup && \
    useradd -r -g appgroup appuser && \
    chown -R appuser:appgroup /app

USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8080/health || exit 1

EXPOSE 8080

CMD ["my_project"]
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
      - DATABASE_URL=postgres://postgres:postgres@db:5432/myapp
      - REDIS_URL=redis://redis:6379
      - RUST_LOG=info
      - RUST_BACKTRACE=1
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migrations:/docker-entrypoint-initdb.d
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  postgres_data:
  redis_data:
  prometheus_data:
  grafana_data:
```

### 错误处理模式

```rust
// 错误类型定义
use thiserror::Error;
use std::fmt;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),
    
    #[error("Validation error: {0}")]
    Validation(String),
    
    #[error("Not found: {resource} with id {id}")]
    NotFound { resource: String, id: String },
    
    #[error("Unauthorized: {0}")]
    Unauthorized(String),
    
    #[error("Forbidden: {0}")]
    Forbidden(String),
    
    #[error("Conflict: {0}")]
    Conflict(String),
    
    #[error("Internal error: {0}")]
    Internal(String),
}

// 转换为 HTTP 响应
impl AppError {
    pub fn status_code(&self) -> u16 {
        match self {
            AppError::Database(_) => 500,
            AppError::Validation(_) => 400,
            AppError::NotFound { .. } => 404,
            AppError::Unauthorized(_) => 401,
            AppError::Forbidden(_) => 403,
            AppError::Conflict(_) => 409,
            AppError::Internal(_) => 500,
        }
    }
    
    pub fn error_code(&self) -> &'static str {
        match self {
            AppError::Database(_) => "DATABASE_ERROR",
            AppError::Validation(_) => "VALIDATION_ERROR",
            AppError::NotFound { .. } => "NOT_FOUND",
            AppError::Unauthorized(_) => "UNAUTHORIZED",
            AppError::Forbidden(_) => "FORBIDDEN",
            AppError::Conflict(_) => "CONFLICT",
            AppError::Internal(_) => "INTERNAL_ERROR",
        }
    }
}

// 结果类型别名
pub type AppResult<T> = Result<T, AppError>;

// 使用示例
pub async fn get_user(id: Uuid) -> AppResult<User> {
    sqlx::query_as!(User, "SELECT * FROM users WHERE id = $1", id)
        .fetch_optional(&*pool)
        .await
        .map_err(|e| AppError::Database(e))?
        .ok_or_else(|| AppError::NotFound {
            resource: "User".to_string(),
            id: id.to_string()
        })
}
```

### 日志和追踪配置

```rust
use tracing::{info, warn, error, Level};
use tracing_subscriber::{fmt, prelude::*, EnvFilter};
use tracing_opentelemetry::OpenTelemetryLayer;
use opentelemetry_otlp::Protocol;

fn init_tracing() {
    // 构建日志过滤
    let env_filter = EnvFilter::try_from_default_env()
        .unwrap_or_else(|_| EnvFilter::new("info"));
    
    // 格式化日志
    let fmt_layer = fmt::layer()
        .with_target(true)
        .with_thread_ids(true)
        .with_thread_names(true)
        .with_file(true)
        .with_line_number(true)
        .compact();
    
    // OpenTelemetry 追踪
    let otlp_layer = opentelemetry_otlp::new_pipeline()
        .tracing()
        .with_exporter(
            opentelemetry_otlp::new_exporter()
                .tonic()
                .with_endpoint("http://localhost:4317")
                .with_protocol(Protocol::Grpc),
        )
        .with_trace_config(
            trace::config()
                .with_sampler(Sampler::AlwaysOn)
                .with_resource(Resource::new(vec![
                    Attribute::new("service.name", "my_service"),
                ])),
        )
        .install_batch(opentelemetry::runtime::Tokio)
        .ok();
    
    // 注册 subscriber
    tracing_subscriber::registry()
        .with(env_filter)
        .with(fmt_layer)
        .with(otlp_layer)
        .init();
}

// 使用宏
info!("Starting application");
info!(user_id = %user.id, "User logged in");
warn!("Retry attempt {attempt} failed", attempt = 3);
error!(error = %err, "Failed to process request");
```

### 参考资料

| 资源 | 链接 |
|------|------|
| Rust 官方文档 | https://doc.rust-lang.org/ |
| The Book | https://doc.rust-lang.org/book/ |
| Rust by Example | https://doc.rust-lang.org/rust-by-example/ |
| Rustlings | https://github.com/rust-lang/rustlings/ |
| Rust API Guidelines | https://rust-lang.github.io/api-guidelines/ |
| crates.io | https://crates.io/ |
| awesome-rust | https://github.com/rust-unofficial/awesome-rust |
| This Week in Rust | https://this-week-in-rust.org/ |
| Rust Magazine | https://rustmagazine.org/ |
| Learning Rust | https://learning-rust.github.io/ |

---

## 常见问题与解决方案

### 问题1：所有权借用检查错误

**问题描述**：Rust 编译器报 `borrowed value does not live long enough` 或 `cannot borrow as mutable`。

```rust
// ❌ 常见错误：返回借用的值
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// ❌ 错误：可变借用冲突
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
let r3 = &mut s; // ❌ 错误：不可变借用和可变借用同时存在
```

**解决方案**：

```rust
// ✅ 解决方案1：使用生命周期标注
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// ✅ 解决方案2：确保借用不重叠
let mut s = String::from("hello");
let r1 = &s;        // 第一个不可变借用
let r2 = &s;        // 第二个不可变借用
println!("{} {}", r1, r2); // r1, r2 在这里不再使用
let r3 = &mut s;   // 现在可以创建可变借用
r3.push_str(" world");

// ✅ 解决方案3：克隆数据而非借用
fn longest_clone(x: String, y: String) -> String {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 问题2：生命周期标注过于复杂

**问题描述**：代码中充满了生命周期标注，难以阅读。

```rust
// ❌ 过度标注
fn foo<'a, 'b: 'a, 'c: 'b>(x: &'a str, y: &'b str, z: &'c str) -> &'a str {
    x
}
```

**解决方案**：

```rust
// ✅ 简化生命周期：只在必要时标注
fn first_word(s: &str) -> &str { // 编译器自动推断
    match s.find(' ') {
        Some(i) => &s[..i],
        None => s
    }
}

// ✅ 使用结构体封装生命周期
struct Parser<'a> {
    input: &'a str,
}

impl<'a> Parser<'a> {
    fn new(input: &'a str) -> Self {
        Parser { input }
    }
    
    fn parse_word(&self) -> &str { // 返回值生命周期与 self 相同
        &self.input[..5]
    }
}

// ✅ 避免返回内部引用的模式
fn get_longest(x: String, y: String) -> String {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

### 问题3：类型不匹配

**问题描述**：`mismatched types` 错误。

```rust
// ❌ 常见错误
let s: &str = "hello"; // ✅ 字符串字面量
let s2: String = s;     // ❌ &str 不能直接转为 String

let v: Vec<i32> = vec![1, 2, 3];
let slice: &[i32] = v;  // ❌ Vec 不能直接转为 slice

// ❌ 类型推断失败
let x = vec![1, 2, 3].iter().sum(); // ❌ 需要类型标注
```

**解决方案**：

```rust
// ✅ 使用 as_ref 或 clone
let s: &str = "hello";
let s2: String = s.to_string(); // ✅
let s3: String = s.to_owned();  // ✅
let s4: String = String::from(s); // ✅

let v: Vec<i32> = vec![1, 2, 3];
let slice: &[i32] = &v;      // ✅ 使用引用
let slice2: &[i32] = v.as_slice(); // ✅

// ✅ 显式类型标注
let x: i32 = vec![1, 2, 3].iter().sum(); // ✅

// ✅ 使用类型推断辅助
fn identity<T>(x: T) -> T { x }
let x = identity(vec![1, 2, 3]); // T 被推断为 Vec<i32>
```

### 问题4：异步代码问题

**问题描述**：`future cannot be sent between threads safely`。

```rust
// ❌ 常见错误：Future 未实现 Send
async fn process_data(data: Rc<Vec<u8>>) -> Vec<u8> {
    // Rc 不是线程安全的
    data.iter().map(|x| x * 2).collect()
}

// ❌ 在 async fn 中返回非 Send 类型
async fn bad_function() -> Rc<String> {
    Rc::new(String::from("hello"))
}
```

**解决方案**：

```rust
// ✅ 使用 Arc 代替 Rc
use std::sync::Arc;

async fn process_data(data: Arc<Vec<u8>>) -> Vec<u8> {
    data.iter().map(|x| x * 2).collect()
}

// ✅ 使用 Mutex 包装可变状态
use std::sync::{Arc, Mutex};

async fn increment(counter: Arc<Mutex<i32>>) {
    let mut num = counter.lock().unwrap();
    *num += 1;
}

// ✅ 使用 tokio::sync 的类型
use tokio::sync::{Mutex, RwLock};

async fn increment_tokio(counter: Arc<Mutex<i32>>) {
    let mut num = counter.lock().await;
    *num += 1;
}

// ✅ 必要时使用 tokio::spawn 显式处理
#[tokio::main]
async fn main() {
    let data = Arc::new(vec![1, 2, 3]);
    
    // 使用 spawn 创建独立任务
    let handle = tokio::spawn(async move {
        process_data(data).await
    });
    
    let result = handle.await.unwrap();
    println!("{:?}", result);
}
```

### 问题5：迭代器链式调用性能问题

**问题描述**：迭代器链式调用不如手写循环快。

```rust
// ❌ 可能导致多次迭代
let result: i32 = (0..1000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .sum();

// ✅ 性能更好（但更不简洁）
let mut sum = 0i32;
for x in 0..1000 {
    if x % 2 == 0 {
        sum += x * 2;
    }
}
```

**解决方案**：

```rust
// ✅ 现代 Rust 优化：迭代器几乎总是足够快
let result: i32 = (0..1000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .sum();

// ✅ 使用并行迭代器处理大数据
use rayon::prelude::*;

let result: i32 = (0..1_000_000)
    .into_par_iter()
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .sum();

// ✅ 预分配 Vector
let data: Vec<i32> = (0..1000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .collect(); // 预分配会更快
```

### 问题6：trait 约束冲突

**问题描述**：`conflicting trait implementation` 或 `type already used`。

```rust
// ❌ 常见错误：重复实现
trait Printable {
    fn print(&self);
}

struct User;
impl Printable for User {
    fn print(&self) { println!("User"); }
}
impl Printable for User { // ❌ 重复实现
    fn print(&self) { println!("User v2"); }
}
```

**解决方案**：

```rust
// ✅ 使用默认实现
trait Printable {
    fn print(&self) {
        println!("Default");
    }
}

struct User;
impl Printable for User {
    // 只覆盖需要的方法
    // fn print() 使用默认实现
}

// ✅ 使用 newtype 模式避免冲突
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

// ✅ 使用泛型约束
fn print_all<T: fmt::Display>(items: &[T]) {
    for item in items {
        println!("{}", item);
    }
}
```

### 问题7：编译时间过长

**问题描述**：Rust 编译时间过长影响开发效率。

**解决方案**：

```rust
// ✅ Cargo.toml 优化
[package]
name = "my_project"

[dependencies]
# 使用更快的 serde 特性
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# 仅在需要时引入重型依赖
tokio = { version = "1.40", features = ["full"] } # 仅用于异步主程序
# 或者选择性启用特性
log = "0.4"

[profile.dev]
opt-level = 0
debug = true

[profile.release]
opt-level = 3
lto = "thin"
codegen-units = 1
```

```toml
# .cargo/config.toml
[build]
rustflags = ["-C", "link-arg=-Wl,--gc-sections"]
```

```bash
# ✅ 使用 cargo check 快速检查
cargo check  # 比 cargo build 快很多

# ✅ 使用 sccache 缓存编译结果
cargo install sccache
export RUSTC_WRAPPER=sccache

# ✅ 使用 cargo-nextest 加速测试
cargo install cargo-nextest
cargo nextest run

# ✅ 分拆依赖
# 将大型依赖拆分到单独的 crate
```

### 问题8：宏使用问题

**问题描述**：宏展开后代码难以调试。

```rust
// ❌ 宏中的错误信息不友好
macro_rules! bad_vec {
    ($($x:expr),*) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

// ✅ 使用更安全的宏
macro_rules! safer_vec {
    ($( $x:expr ),* $(,)?) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

// ✅ 添加编译时检查
macro_rules! ensure_same_type {
    ($($xs:expr),* $(,)?) => {
        {
            #[compiler_assert]
            const _: () = ();
        }
    };
}
```

**解决方案**：

```rust
// ✅ 使用 proc_macro 提供更好的错误信息
use proc_macro::TokenStream;

#[proc_macro]
pub fn construct_struct(input: TokenStream) -> TokenStream {
    // 解析输入并生成更友好的错误
    // ...
}

// ✅ 使用 quote! 生成清晰代码
use quote::quote;

let name = /* parse name */;
let expanded = quote! {
    struct #name {
        data: String,
    }
    
    impl #name {
        fn new() -> Self {
            Self {
                data: String::from("initialized"),
            }
        }
    }
};

expanded.into()
```

### 问题9：错误处理过于繁琐

**问题描述**：每个函数都要处理 Result 导致代码膨胀。

**解决方案**：

```rust
// ✅ 使用 ? 操作符传播错误
use anyhow::{Context, Result};

fn read_config() -> Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    
    let config: Config = toml::from_str(&content)
        .with_context(|| format!("Failed to parse config"))?;
    
    Ok(config)
}

// ✅ 使用 thiserror 定义清晰错误
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),
    
    #[error("Not found: {entity} with id {id}")]
    NotFound { entity: String, id: String },
    
    #[error("Validation error: {0}")]
    Validation(String),
}

// ✅ 使用 Box<dyn Error> 处理多种错误
fn risky_function() -> Result<T, Box<dyn std::error::Error>> {
    // 可以返回任何实现了 Error trait 的类型
    Ok(try_something()?)
}

// ✅ 使用自定义 Result 别名
type Result<T, E = AppError> = std::result::Result<T, E>;
```

### 问题10：多线程数据竞争

**问题描述**：虽然 Rust 防止了大多数数据竞争，但仍有可能遗漏。

```rust
// ❌ 潜在数据竞争
use std::thread;

let mut data = vec![1, 2, 3];
let handles: Vec<_> = (0..3).map(|i| {
    let data_ref = &data; // 借用 data
    thread::spawn(move || {
        data_ref.push(i); // ❌ 可能导致数据竞争
    })
}).collect();
```

**解决方案**：

```rust
// ✅ 使用 Arc<Mutex<T>> 保护共享数据
use std::sync::{Arc, Mutex};
use std::thread;

let data = Arc::new(Mutex::new(vec![1, 2, 3]));
let handles: Vec<_> = (0..3).map(|i| {
    let data = Arc::clone(&data);
    thread::spawn(move || {
        let mut data = data.lock().unwrap();
        data.push(i);
    })
}).collect();

for handle in handles {
    handle.join().unwrap();
}

// ✅ 读写锁适用于读多写少场景
use std::sync::RwLock;

let data = Arc::new(RwLock::new(vec![1, 2, 3]));

// 读操作
let data = data.read().unwrap();
println!("{:?}", data);

// 写操作
let mut data = data.write().unwrap();
data.push(4);

// ✅ 使用消息传递替代共享内存
use std::sync::mpsc;

let (tx, rx) = mpsc::channel();

for i in 0..3 {
    let tx = tx.clone();
    thread::spawn(move || {
        tx.send(i).unwrap();
    });
}

drop(tx); // 发送端关闭
let results: Vec<_> = rx.iter().collect();
```

---

## 实战项目示例

### 项目一：高性能 Web API 服务

**功能特性**：

- RESTful API
- 数据库集成（SQLx）
- JWT 认证
- 限流中间件
- OpenTelemetry 追踪

```rust
// src/main.rs
use axum::{
    body::Body,
    extract::{Path, Query, State},
    http::{HeaderMap, StatusCode},
    middleware,
    response::{IntoResponse, Response},
    routing::{delete, get, post},
    Json, Router,
};
use chrono::{DateTime, Utc};
use sqlx::{postgres::PgPoolOptions, PgPool};
use std::net::SocketAddr;
use std::sync::Arc;
use tower_http::{cors::CorsLayer, trace::TraceLayer};
use tracing::{info, Level};
use tracing_subscriber::FmtSubscriber;

mod auth;
mod db;
mod error;
mod handlers;
mod middleware as app_middleware;
mod models;
mod repositories;

pub use error::{AppError, AppResult};

#[derive(Clone)]
struct AppState {
    db: PgPool,
    jwt_secret: String,
}

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // 初始化日志
    let subscriber = FmtSubscriber::builder()
        .with_max_level(Level::INFO)
        .with_target(true)
        .with_thread_ids(true)
        .with_file(true)
        .with_line_number(true)
        .compact()
        .init();

    // 数据库连接池
    let database_url = std::env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    
    let pool = PgPoolOptions::new()
        .max_connections(10)
        .connect(&database_url)
        .await?;

    // JWT 密钥
    let jwt_secret = std::env::var("JWT_SECRET")
        .unwrap_or_else(|_| "dev-secret-key".to_string());

    // 应用状态
    let state = AppState { db: pool, jwt_secret };

    // 路由
    let app = Router::new()
        .route("/health", get(handlers::health_check))
        .route("/api/v1/auth/register", post(handlers::register))
        .route("/api/v1/auth/login", post(handlers::login))
        .route("/api/v1/users", get(handlers::list_users))
        .route("/api/v1/users/:id", get(handlers::get_user))
        .route("/api/v1/posts", get(handlers::list_posts))
        .route("/api/v1/posts", post(handlers::create_post))
        .route("/api/v1/posts/:id", get(handlers::get_post))
        .route("/api/v1/posts/:id", delete(handlers::delete_post))
        .layer(TraceLayer::new_for_http())
        .layer(
            CorsLayer::new()
                .allow_origin(["http://localhost:3000".parse()])
                .allow_methods(tower_http::cors::AllowMethods::any())
                .allow_headers(tower_http::cors::AllowHeaders::any()),
        )
        .with_state(state);

    // 启动服务器
    let addr = SocketAddr::from(([0, 0, 0, 0], 8080));
    info!("Server starting on {}", addr);
    
    let listener = tokio::net::TcpListener::bind(addr).await?;
    axum::serve(listener, app).await?;

    Ok(())
}
```

```rust
// src/error.rs
use axum::{
    http::StatusCode,
    response::{IntoResponse, Response},
    Json,
};
use serde_json::json;
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Not found: {0}")]
    NotFound(String),

    #[error("Unauthorized: {0}")]
    Unauthorized(String),

    #[error("Forbidden: {0}")]
    Forbidden(String),

    #[error("Bad request: {0}")]
    BadRequest(String),

    #[error("Conflict: {0}")]
    Conflict(String),

    #[error("Internal error: {0}")]
    Internal(String),

    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Validation error: {0}")]
    Validation(String),
}

pub type AppResult<T> = Result<T, AppError>;

impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, message) = match &self {
            AppError::NotFound(msg) => (StatusCode::NOT_FOUND, msg.clone()),
            AppError::Unauthorized(msg) => (StatusCode::UNAUTHORIZED, msg.clone()),
            AppError::Forbidden(msg) => (StatusCode::FORBIDDEN, msg.clone()),
            AppError::BadRequest(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
            AppError::Conflict(msg) => (StatusCode::CONFLICT, msg.clone()),
            AppError::Internal(msg) => (StatusCode::INTERNAL_SERVER_ERROR, msg.clone()),
            AppError::Database(e) => {
                tracing::error!("Database error: {:?}", e);
                (StatusCode::INTERNAL_SERVER_ERROR, "Database error".to_string())
            }
            AppError::Validation(msg) => (StatusCode::BAD_REQUEST, msg.clone()),
        };

        let body = Json(json!({
            "error": message,
            "status": status.as_u16()
        }));

        (status, body).into_response()
    }
}
```

```rust
// src/models.rs
use chrono::{DateTime, Utc};
use serde::{Deserialize, Serialize};
use sqlx::FromRow;

#[derive(Debug, Clone, Serialize, Deserialize, FromRow)]
pub struct User {
    pub id: uuid::Uuid,
    pub email: String,
    pub username: String,
    pub password_hash: String,
    pub first_name: Option<String>,
    pub last_name: Option<String>,
    pub role: String,
    pub is_active: bool,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Clone, Serialize, Deserialize, FromRow)]
pub struct Post {
    pub id: uuid::Uuid,
    pub author_id: uuid::Uuid,
    pub title: String,
    pub slug: String,
    pub content: String,
    pub excerpt: Option<String>,
    pub status: String,
    pub published_at: Option<DateTime<Utc>>,
    pub created_at: DateTime<Utc>,
    pub updated_at: DateTime<Utc>,
}

#[derive(Debug, Deserialize)]
pub struct CreateUserRequest {
    pub email: String,
    pub username: String,
    pub password: String,
    pub first_name: Option<String>,
    pub last_name: Option<String>,
}

#[derive(Debug, Deserialize)]
pub struct LoginRequest {
    pub email: String,
    pub password: String,
}

#[derive(Debug, Serialize)]
pub struct AuthResponse {
    pub user: UserResponse,
    pub access_token: String,
    pub refresh_token: String,
}

#[derive(Debug, Serialize)]
pub struct UserResponse {
    pub id: uuid::Uuid,
    pub email: String,
    pub username: String,
    pub first_name: Option<String>,
    pub last_name: Option<String>,
    pub role: String,
}

#[derive(Debug, Deserialize)]
pub struct CreatePostRequest {
    pub title: String,
    pub content: String,
    pub excerpt: Option<String>,
    pub status: String,
}

#[derive(Debug, Deserialize)]
pub struct PaginationParams {
    pub page: Option<i64>,
    pub limit: Option<i64>,
}

impl Default for PaginationParams {
    fn default() -> Self {
        Self {
            page: Some(1),
            limit: Some(10),
        }
    }
}
```

```rust
// src/handlers.rs
use crate::error::{AppError, AppResult};
use crate::models::*;
use crate::AppState;
use axum::{
    extract::{Path, Query, State},
    http::StatusCode,
    Json,
};
use uuid::Uuid;

pub async fn health_check() -> &'static str {
    "OK"
}

pub async fn register(
    State(state): State<AppState>,
    Json(req): Json<CreateUserRequest>,
) -> AppResult<Json<AuthResponse>> {
    // 验证输入
    if req.email.is_empty() || !req.email.contains('@') {
        return Err(AppError::Validation("Invalid email".to_string()));
    }

    if req.password.len() < 8 {
        return Err(AppError::Validation(
            "Password must be at least 8 characters".to_string(),
        ));
    }

    // 检查用户是否已存在
    let existing = sqlx::query_as::<_, (i64,)>(
        "SELECT COUNT(*) FROM users WHERE email = $1 OR username = $2",
    )
    .bind(&req.email)
    .bind(&req.username)
    .fetch_one(&state.db)
    .await?;

    if existing.0 > 0 {
        return Err(AppError::Conflict("User already exists".to_string()));
    }

    // 密码哈希
    let password_hash = bcrypt::hash(&req.password, bcrypt::DEFAULT_COST)?;

    // 创建用户
    let user: User = sqlx::query_as(
        r#"
        INSERT INTO users (email, username, password_hash, first_name, last_name, role)
        VALUES ($1, $2, $3, $4, $5, $6)
        RETURNING *
        "#,
    )
    .bind(&req.email)
    .bind(&req.username)
    .bind(&password_hash)
    .bind(&req.first_name)
    .bind(&req.last_name)
    .bind("user")
    .fetch_one(&state.db)
    .await?;

    // 生成 JWT
    let access_token = auth::generate_token(&state.jwt_secret, &user)?;
    let refresh_token = auth::generate_refresh_token(&state.jwt_secret, &user)?;

    Ok(Json(AuthResponse {
        user: UserResponse {
            id: user.id,
            email: user.email,
            username: user.username,
            first_name: user.first_name,
            last_name: user.last_name,
            role: user.role,
        },
        access_token,
        refresh_token,
    }))
}

pub async fn login(
    State(state): State<AppState>,
    Json(req): Json<LoginRequest>,
) -> AppResult<Json<AuthResponse>> {
    // 查找用户
    let user: User = sqlx::query_as(
        "SELECT * FROM users WHERE email = $1 AND is_active = true",
    )
    .bind(&req.email)
    .fetch_optional(&state.db)
    .await?
    .ok_or_else(|| AppError::Unauthorized("Invalid credentials".to_string()))?;

    // 验证密码
    if !bcrypt::verify(&req.password, &user.password_hash)? {
        return Err(AppError::Unauthorized("Invalid credentials".to_string()));
    }

    // 生成 JWT
    let access_token = auth::generate_token(&state.jwt_secret, &user)?;
    let refresh_token = auth::generate_refresh_token(&state.jwt_secret, &user)?;

    Ok(Json(AuthResponse {
        user: UserResponse {
            id: user.id,
            email: user.email,
            username: user.username,
            first_name: user.first_name,
            last_name: user.last_name,
            role: user.role,
        },
        access_token,
        refresh_token,
    }))
}

pub async fn list_users(
    State(state): State<AppState>,
    Query(params): Query<PaginationParams>,
) -> AppResult<Json<Vec<UserResponse>>> {
    let page = params.page.unwrap_or(1).max(1);
    let limit = params.limit.unwrap_or(10).min(100);
    let offset = (page - 1) * limit;

    let users: Vec<User> = sqlx::query_as(
        "SELECT * FROM users WHERE is_active = true ORDER BY created_at DESC LIMIT $1 OFFSET $2",
    )
    .bind(limit)
    .bind(offset)
    .fetch_all(&state.db)
    .await?;

    let response: Vec<UserResponse> = users
        .into_iter()
        .map(|u| UserResponse {
            id: u.id,
            email: u.email,
            username: u.username,
            first_name: u.first_name,
            last_name: u.last_name,
            role: u.role,
        })
        .collect();

    Ok(Json(response))
}

pub async fn get_user(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> AppResult<Json<UserResponse>> {
    let user: User = sqlx::query_as("SELECT * FROM users WHERE id = $1")
        .bind(id)
        .fetch_optional(&state.db)
        .await?
        .ok_or_else(|| AppError::NotFound("User not found".to_string()))?;

    Ok(Json(UserResponse {
        id: user.id,
        email: user.email,
        username: user.username,
        first_name: user.first_name,
        last_name: user.last_name,
        role: user.role,
    }))
}

pub async fn list_posts(
    State(state): State<AppState>,
    Query(params): Query<PaginationParams>,
) -> AppResult<Json<Vec<Post>>> {
    let page = params.page.unwrap_or(1).max(1);
    let limit = params.limit.unwrap_or(10).min(100);
    let offset = (page - 1) * limit;

    let posts: Vec<Post> = sqlx::query_as(
        r#"
        SELECT * FROM posts 
        WHERE status = 'published' 
        ORDER BY published_at DESC 
        LIMIT $1 OFFSET $2
        "#,
    )
    .bind(limit)
    .bind(offset)
    .fetch_all(&state.db)
    .await?;

    Ok(Json(posts))
}

pub async fn create_post(
    State(state): State<AppState>,
    Json(req): Json<CreatePostRequest>,
) -> AppResult<Json<Post>> {
    let slug = slug::slugify(&req.title);

    let post: Post = sqlx::query_as(
        r#"
        INSERT INTO posts (author_id, title, slug, content, excerpt, status, published_at)
        VALUES ($1, $2, $3, $4, $5, $6, $7)
        RETURNING *
        "#,
    )
    .bind(Uuid::new_v4()) // 临时使用
    .bind(&req.title)
    .bind(&slug)
    .bind(&req.content)
    .bind(&req.excerpt)
    .bind(&req.status)
    .bind(if req.status == "published" {
        Some(chrono::Utc::now())
    } else {
        None
    })
    .fetch_one(&state.db)
    .await?;

    Ok(Json(post))
}

pub async fn get_post(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> AppResult<Json<Post>> {
    let post: Post = sqlx::query_as(
        "SELECT * FROM posts WHERE id = $1 AND status = 'published'",
    )
    .bind(id)
    .fetch_optional(&state.db)
    .await?
    .ok_or_else(|| AppError::NotFound("Post not found".to_string()))?;

    Ok(Json(post))
}

pub async fn delete_post(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> AppResult<StatusCode> {
    let result = sqlx::query("DELETE FROM posts WHERE id = $1")
        .bind(id)
        .execute(&state.db)
        .await?;

    if result.rows_affected() == 0 {
        return Err(AppError::NotFound("Post not found".to_string()));
    }

    Ok(StatusCode::NO_CONTENT)
}
```

### 项目二：CLI 工具

**功能特性**：

- 命令行参数解析（clap）
- 配置文件支持
- 进度条显示
- 彩色输出

```rust
// src/main.rs
use anyhow::{Context, Result};
use clap::{Parser, Subcommand};
use console::{style, Emoji};
use indicatif::{ProgressBar, ProgressStyle};
use std::path::PathBuf;

mod commands;
mod config;
mod utils;

#[derive(Parser)]
#[command(
    name = "mycli",
    about = "A powerful CLI tool",
    version = "1.0.0",
    author = "Author"
)]
struct Cli {
    #[command(subcommand)]
    command: Commands,

    /// Enable verbose output
    #[arg(short, long, global = true)]
    verbose: bool,

    /// Config file path
    #[arg(short, long, global = true, default_value = "~/.mycli.toml")]
    config: PathBuf,
}

#[derive(Subcommand)]
enum Commands {
    /// Process files
    Process {
        /// Input file or directory
        #[arg(required = true)]
        input: PathBuf,

        /// Output directory
        #[arg(short, long, default_value = "./output")]
        output: PathBuf,

        /// Number of parallel workers
        #[arg(short, long, default_value_t = 4)]
        workers: usize,

        /// Recursively process directories
        #[arg(short, long)]
        recursive: bool,
    },

    /// List available resources
    List {
        /// Filter by type
        #[arg(short, long)]
        filter: Option<String>,

        /// Show detailed information
        #[arg(short, long)]
        detailed: bool,
    },

    /// Configure the CLI
    Config {
        /// Show current configuration
        #[arg(short, long)]
        show: bool,

        /// Set a configuration value
        #[arg(short, long, num_args = 2, value_names = ["key", "value"])]
        set: Option<Vec<String>>,
    },

    /// Initialize a new project
    Init {
        /// Project name
        #[arg(required = true)]
        name: String,

        /// Template to use
        #[arg(short, long, default_value = "default")]
        template: String,
    },
}

#[tokio::main]
async fn main() -> Result<()> {
    let cli = Cli::parse();

    // 初始化日志
    if cli.verbose {
        tracing_subscriber::fmt()
            .with_max_level(tracing::Level::DEBUG)
            .init();
    }

    // 加载配置
    let config = config::load(&cli.config)
        .context("Failed to load configuration")?;

    // 执行命令
    match cli.command {
        Commands::Process {
            input,
            output,
            workers,
            recursive,
        } => commands::process(input, output, workers, recursive, &config).await?,

        Commands::List { filter, detailed } => commands::list(filter, detailed, &config)?,

        Commands::Config { show, set } => {
            if show {
                commands::show_config(&config)?;
            } else if let Some(values) = set {
                commands::set_config(&values[0], &values[1], &cli.config)?;
            }
        }

        Commands::Init { name, template } => commands::init(&name, &template, &config).await?,
    }

    Ok(())
}
```

```rust
// src/commands.rs
use crate::config::Config;
use crate::utils;
use anyhow::{Context, Result};
use console::style;
use indicatif::{ProgressBar, ProgressStyle};
use std::fs;
use std::path::Path;

pub async fn process(
    input: std::path::PathBuf,
    output: std::path::PathBuf,
    workers: usize,
    recursive: bool,
    config: &Config,
) -> Result<()> {
    println!(
        "{} {} Processing files...",
        style("[1/3]").cyan().bold(),
        style("Collecting").green()
    );

    // 收集文件
    let files = if input.is_dir() {
        utils::collect_files(&input, recursive)?
    } else {
        vec![input]
    };

    if files.is_empty() {
        println!("{}", style("No files found").yellow());
        return Ok(());
    }

    println!(
        "{} {} Found {} files",
        style("[2/3]").cyan().bold(),
        style("Processing").green(),
        style(files.len()).cyan()
    );

    // 创建输出目录
    fs::create_dir_all(&output).context("Failed to create output directory")?;

    // 进度条
    let pb = ProgressBar::new(files.len() as u64);
    pb.set_style(
        ProgressStyle::default_bar()
            .template("{spinner:.green} [{elapsed_precise}] [{bar:40.cyan/blue}] {pos}/{len} {msg}")
            .unwrap()
            .progress_chars("=>-"),
    );

    // 并行处理
    use rayon::prelude::*;

    let results: Vec<Result<()>> = files
        .par_iter()
        .map(|file| {
            let _ = pb.update(|_| {
                pb.set_message(file.file_name().unwrap_or_default().to_string_lossy().to_string());
            });

            // 模拟处理
            let content = fs::read_to_string(file)
                .with_context(|| format!("Failed to read {:?}", file))?;

            // 处理逻辑
            let processed = utils::process_content(&content, config)?;

            // 写入输出
            let output_file = output.join(file.file_name().unwrap());
            fs::write(&output_file, processed)?;

            pb.inc(1);
            Ok(())
        })
        .collect();

    pb.finish_with_message("Done");

    // 报告错误
    let errors: Vec<_> = results.iter().filter(|r| r.is_err()).collect();
    if !errors.is_empty() {
        eprintln!(
            "{} {} errors occurred",
            style("!").red().bold(),
            style(errors.len()).red()
        );
    }

    println!(
        "\n{} {} Successfully processed {} files",
        style("✓").green().bold(),
        style("Complete").green(),
        style(files.len() - errors.len()).cyan()
    );

    Ok(())
}

pub fn list(filter: Option<String>, detailed: bool, config: &Config) -> Result<()> {
    println!(
        "{} {} Listing resources...",
        style("[1/1]").cyan().bold(),
        style("Fetching").green()
    );

    // 模拟获取资源列表
    let items = vec![
        ("resource1", "Type A", "1.0 MB"),
        ("resource2", "Type B", "2.5 MB"),
        ("resource3", "Type A", "500 KB"),
    ];

    for (name, r#type, size) in items.iter() {
        if let Some(ref f) = filter {
            if !type.contains(f) && !name.contains(f) {
                continue;
            }
        }

        if detailed {
            println!(
                "{}  {}  {}  {}",
                style(name).cyan(),
                style(type).yellow(),
                style(size).magenta(),
                style("Available").green()
            );
        } else {
            println!("- {}", style(name).cyan());
        }
    }

    Ok(())
}

pub fn show_config(config: &Config) -> Result<()> {
    println!("{}", style("Current Configuration:").cyan().bold());
    println!("  API Endpoint: {}", style(&config.api_endpoint).yellow());
    println!("  Timeout: {}s", style(config.timeout).yellow());
    println!("  Max Retries: {}", style(config.max_retries).yellow());

    Ok(())
}

pub fn set_config(key: &str, value: &str, config_path: &Path) -> Result<()> {
    println!(
        "Setting {} = {}",
        style(key).cyan(),
        style(value).yellow()
    );

    // 实现配置更新逻辑
    println!("{}", style("Configuration updated successfully!").green());

    Ok(())
}

pub async fn init(name: &str, template: &str, config: &Config) -> Result<()> {
    println!(
        "{} {} Initializing project '{}'...",
        style("[1/3]").cyan().bold(),
        style("Creating").green(),
        style(name).cyan()
    );

    // 创建项目目录
    let project_path = Path::new(name);
    fs::create_dir(project_path)?;

    // 复制模板文件
    println!(
        "{} {} Using template '{}'",
        style("[2/3]").cyan().bold(),
        style("Copying").green(),
        style(template).yellow()
    );

    // 创建基本文件
    let readme = format!("# {}\n\nA project created with mycli.", name);
    fs::write(project_path.join("README.md"), readme)?;

    let gitignore = r#"target/
*.log
.env
.DS_Store
"#;
    fs::write(project_path.join(".gitignore"), gitignore)?;

    println!(
        "{} {} Project initialized successfully!",
        style("[3/3]").cyan().bold(),
        style("Complete").green()
    );

    println!(
        "\n{} {} to get started:",
        style("Run").cyan().bold(),
        style("cd").yellow()
    );
    println!("  cd {}", style(name).cyan());
    println!("  mycli process ./data");

    Ok(())
}
```

```rust
// src/utils.rs
use crate::config::Config;
use anyhow::{Context, Result};
use std::fs;
use std::path::{Path, PathBuf};

pub fn collect_files(path: &Path, recursive: bool) -> Result<Vec<PathBuf>> {
    let mut files = Vec::new();

    if path.is_file() {
        files.push(path.to_path_buf());
        return Ok(files);
    }

    fn visit_dir(dir: &Path, recursive: bool, files: &mut Vec<PathBuf>) -> Result<()> {
        for entry in fs::read_dir(dir)? {
            let entry = entry?;
            let path = entry.path();

            if path.is_file() {
                // 只处理文本文件
                if let Some(ext) = path.extension() {
                    if matches!(ext.to_str(), Some("txt" | "md" | "json" | "rs" | "ts")) {
                        files.push(path.clone());
                    }
                }
            } else if path.is_dir() && recursive {
                visit_dir(&path, recursive, files)?;
            }
        }
        Ok(())
    }

    visit_dir(path, recursive, &mut files)?;

    Ok(files)
}

pub fn process_content(content: &str, config: &Config) -> Result<String> {
    // 示例处理：统计行数并添加元数据
    let lines: Vec<&str> = content.lines().collect();
    let word_count: usize = content.split_whitespace().count();

    let header = format!(
        "// Lines: {}, Words: {}, Chars: {}\n\n",
        lines.len(),
        word_count,
        content.len()
    );

    Ok(format!("{}{}", header, content))
}

pub fn format_size(bytes: u64) -> String {
    const KB: u64 = 1024;
    const MB: u64 = KB * 1024;
    const GB: u64 = MB * 1024;

    if bytes >= GB {
        format!("{:.2} GB", bytes as f64 / GB as f64)
    } else if bytes >= MB {
        format!("{:.2} MB", bytes as f64 / MB as f64)
    } else if bytes >= KB {
        format!("{:.2} KB", bytes as f64 / KB as f64)
    } else {
        format!("{} B", bytes)
    }
}
```

```rust
// src/config.rs
use anyhow::{Context, Result};
use serde::{Deserialize, Serialize};
use std::fs;
use std::path::Path;

#[derive(Debug, Clone, Deserialize, Serialize)]
pub struct Config {
    pub api_endpoint: String,
    pub timeout: u64,
    pub max_retries: u32,
    pub api_key: Option<String>,
}

impl Default for Config {
    fn default() -> Self {
        Self {
            api_endpoint: "https://api.example.com".to_string(),
            timeout: 30,
            max_retries: 3,
            api_key: None,
        }
    }
}

pub fn load(path: &Path) -> Result<Config> {
    // 展开 ~ 路径
    let expanded_path = expand_path(path);

    if !expanded_path.exists() {
        return Ok(Config::default());
    }

    let content = fs::read_to_string(&expanded_path)
        .with_context(|| format!("Failed to read config from {:?}", expanded_path))?;

    toml::from_str(&content)
        .with_context(|| "Failed to parse config file")
}

pub fn save(path: &Path, config: &Config) -> Result<()> {
    let expanded_path = expand_path(path);

    // 确保父目录存在
    if let Some(parent) = expanded_path.parent() {
        fs::create_dir_all(parent)?;
    }

    let content = toml::to_string_pretty(config)
        .context("Failed to serialize config")?;

    fs::write(&expanded_path, content)?;

    Ok(())
}

fn expand_path(path: &Path) -> std::path::PathBuf {
    let path_str = path.to_string_lossy();
    if path_str.starts_with("~/") {
        dirs::home_dir()
            .map(|home| home.join(path_str.trim_start_matches("~/")))
            .unwrap_or_else(|| path.to_path_buf())
    } else {
        path.to_path_buf()
    }
}
```

---

> [!SUCCESS]
> 本文档全面介绍了 Rust 的核心特性、所有权系统、trait 与泛型、并发编程、异步编程、Web 开发、测试与优化以及在 AI 和系统编程中的实战应用。Rust 凭借其内存安全保证、零成本抽象和强大的并发支持，是构建高性能、可靠系统的理想选择。

---

## 附录：Rust 与 AI 生态深度集成

### LLMChain 框架

```rust
// llm-chain 是 Rust 中的 LLM 链式调用框架
// 项目地址: https://github.com/sohich/llm-chain

use llm_chain::{Executor, parameters, traits::StepExt};
use llm_chain_openai::chatgpt::Executor as ChatGPTExecutor;

#[tokio::main]
async fn main() {
    // 创建执行器
    let exec = ChatGPTExecutor::new().unwrap();
    
    // 创建提示
    let chain = (
        llm_chain::prompt!(r#"你是一个助手。
        用户名是 {name}，他们的余额是 ${balance}。
        "#),
        llm_chain::output_parser::streaming(),
    ).into();
    
    // 执行链
    let result = chain.run(
        parameters! {
            "name": "Alice",
            "balance": 100.0
        }
    ).await;
    
    println!("{:?}", result);
}
```

### Candle 机器学习框架

```rust
// Candle 是 Rust 的 ML 框架，支持 CPU 和 CUDA
// 项目地址: https://github.com/huggingface/candle

use candle::{Device, Result, Tensor};
use candle_nn::{Module, Linear, VarBuilder};
use candle_transformers::models::llama::Llama;

struct ModelConfig {
    hidden_size: usize,
    num_layers: usize,
    num_heads: usize,
    vocab_size: usize,
}

impl ModelConfig {
    fn tiny() -> Self {
        Self {
            hidden_size: 288,
            num_layers: 6,
            num_heads: 6,
            vocab_size: 32000,
        }
    }
}

fn main() -> Result<()> {
    let device = Device::Cpu;
    
    // 创建张量
    let tensor = Tensor::new(&[1.0f32, 2.0, 3.0], &device)?;
    println!("Tensor: {:?}", tensor);
    
    // 矩阵运算
    let a = Tensor::randn(0.0, 1.0, (3, 4), &device)?;
    let b = Tensor::randn(0.0, 1.0, (4, 5), &device)?;
    let c = a.matmul(&b)?;
    
    println!("Result shape: {:?}", c.shape());
    
    Ok(())
}

// 自定义神经网络层
struct SimpleMLP {
    fc1: Linear,
    fc2: Linear,
}

impl SimpleMLP {
    fn new(vocab_size: usize, hidden_size: usize) -> Result<Self> {
        let fc1 = Linear::new(vocab_size, hidden_size, true);
        let fc2 = Linear::new(hidden_size, vocab_size, true);
        Ok(Self { fc1, fc2 })
    }
}

impl Module for SimpleMLP {
    fn forward(&self, xs: &Tensor) -> Result<Tensor> {
        let hidden = xs.apply(&self.fc1)?;
        let hidden = hidden.relu()?;
        hidden.apply(&self.fc2)
    }
}
```

### PyO3 Python 绑定

```rust
// PyO3 允许编写 Rust 扩展模块供 Python 调用
// 项目地址: https://github.com/PyO3/pyo3

use pyo3::prelude::*;
use pyo3::types::PyDict;

#[pyclass]
struct VectorStore {
    data: Vec<Vec<f32>>,
    dimension: usize,
}

#[pymethods]
impl VectorStore {
    #[new]
    fn new(dimension: usize) -> Self {
        VectorStore {
            data: Vec::new(),
            dimension,
        }
    }

    fn add(&mut self, embedding: Vec<f32>) -> PyResult<usize> {
        if embedding.len() != self.dimension {
            return Err(pyo3::exceptions::PyValueError::new_err(
                format!("Expected dimension {}, got {}", 
                    self.dimension, embedding.len())
            ));
        }
        self.data.push(embedding);
        Ok(self.data.len() - 1)
    }

    fn search(&self, query: Vec<f32>, k: usize) -> PyResult<Vec<(usize, f32)>> {
        if query.len() != self.dimension {
            return Err(pyo3::exceptions::PyValueError::new_err(
                "Query dimension mismatch"
            ));
        }

        let mut results: Vec<(usize, f32)> = self.data
            .iter()
            .enumerate()
            .map(|(i, embedding)| {
                let similarity = cosine_similarity(&query, embedding);
                (i, similarity)
            })
            .collect();
        
        results.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());
        results.truncate(k);
        Ok(results)
    }

    fn __len__(&self) -> usize {
        self.data.len()
    }
}

fn cosine_similarity(a: &[f32], b: &[f32]) -> f32 {
    let dot: f32 = a.iter().zip(b.iter()).map(|(x, y)| x * y).sum();
    let norm_a: f32 = a.iter().map(|x| x * x).sum::<f32>().sqrt();
    let norm_b: f32 = b.iter().map(|x| x * x).sum::<f32>().sqrt();
    dot / (norm_a * norm_b)
}

#[pymodule]
fn rust_vector_store(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_class::<VectorStore>()?;
    Ok(())
}
```

### 使用 maturin 构建 Python 扩展

```toml
# Cargo.toml 配置
[package]
name = "rust_vector_store"
version = "0.1.0"
edition = "2021"

[lib]
name = "rust_vector_store"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.20", features = ["extension-module"] }

[build-dependencies]
pyo3-build-config = "0.20"
```

```bash
# 安装 maturin
pip install maturin

# 构建 Python 包
maturin develop

# 或者发布到 PyPI
maturin build
maturin publish
```

### 边缘计算与 WASM

```rust
// 使用 wasm-bindgen 构建 WebAssembly 扩展
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn process_text(input: &str) -> String {
    input.to_uppercase()
}

#[wasm_bindgen]
pub struct TextProcessor {
    words: Vec<String>,
}

#[wasm_bindgen]
impl TextProcessor {
    #[wasm_bindgen(constructor)]
    pub fn new(text: &str) -> Self {
        TextProcessor {
            words: text.split_whitespace().map(String::from).collect(),
        }
    }

    pub fn word_count(&self) -> usize {
        self.words.len()
    }

    pub fn unique_words(&self) -> usize {
        use std::collections::HashSet;
        let unique: HashSet<_> = self.words.iter().collect();
        unique.len()
    }

    pub fn longest_word(&self) -> String {
        self.words
            .iter()
            .max_by_key(|w| w.len())
            .cloned()
            .unwrap_or_default()
    }
}

// 性能基准测试
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_process_text() {
        assert_eq!(process_text("hello"), "HELLO");
    }

    #[test]
    fn test_word_processor() {
        let processor = TextProcessor::new("hello world hello");
        assert_eq!(processor.word_count(), 3);
        assert_eq!(processor.unique_words(), 2);
    }
}
```

### ONNX Runtime Rust 绑定

```rust
// 使用 ort (ONNX Runtime Rust) 加载和运行模型
// 项目地址: https://github.com/pykeio/ort

use ort::{Environment, Session, Value};
use std::sync::Arc;

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 初始化环境
    let environment = Arc::new(
        Environment::builder()
            .with_execution_providers([
                "CUDAExecutionProvider",  // GPU 支持
                "CPUExecutionProvider",
            ])
            .build()?
    );

    // 加载模型
    let session = Session::load(
        &environment,
        "model.onnx",
        None  // 使用默认配置
    )?;

    // 准备输入
    let input_tensor: Value = Value::from_array(
        vec![1, 3, 224, 224].as_slice(),
        &[0.0f32; 150528]  // 1x3x224x224 的零张量
    )?;

    // 运行推理
    let outputs = session.run(vec![input_tensor])?;

    // 处理输出
    if let Some(output) = outputs.first() {
        let tensor = output.as_tensor::<f32>()?;
        println!("Output shape: {:?}", tensor.dimensions());
        println!("First 10 outputs: {:?}", &tensor.as_slice()[..10]);
    }

    Ok(())
}
```

### Rust 中的向量数据库客户端

```rust
// 与 Qdrant 向量数据库集成
// 项目地址: https://github.com/polbut/async-nats

use qdrant_client::client::QdrantClient;
use qdrant_client::client::Payload;
use qdrant_client::qdrant::{SearchParams, Distance, VectorParams, point_id::PointId};

#[derive(Clone)]
struct VectorStore {
    client: QdrantClient,
    collection: String,
}

impl VectorStore {
    async fn new(url: &str, collection: &str) -> Result<Self, Box<dyn std::error::Error>> {
        let client = QdrantClient::from_url(url).build()?;
        
        // 创建集合
        let collection_info = client.collection_exists(collection).await?;
        
        if !collection_info {
            client.create_collection(collection, 4, Distance::Cosine).await?;
        }

        Ok(Self {
            client,
            collection: collection.to_string(),
        })
    }

    async fn upsert(
        &self,
        id: u64,
        vector: Vec<f32>,
        payload: Payload,
    ) -> Result<(), Box<dyn std::error::Error>> {
        self.client
            .upsert_point(
                &self.collection,
                PointId::from(id),
                vector,
                Some(payload),
            )
            .await?;
        Ok(())
    }

    async fn search(
        &self,
        vector: Vec<f32>,
        limit: usize,
    ) -> Result<Vec<(u64, f32)>, Box<dyn std::error::Error>> {
        let results = self.client
            .search_points(
                &self.collection,
                vector,
                limit,
                None,
                SearchParams::default(),
                None,
            )
            .await?;

        let mut search_results: Vec<(u64, f32)> = results
            .result
            .iter()
            .filter_map(|point| {
                let id = match &point.id {
                    PointId::Num(n) => *n as u64,
                    _ => return None,
                };
                let score = point.score;
                Some((id, score))
            })
            .collect();

        search_results.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());
        Ok(search_results)
    }
}
```

###Tokio 异步运行时深度配置

```rust
use tokio::runtime::{Builder, Runtime};
use tokio::time::{sleep, Duration};

fn main() {
    // 创建多线程运行时
    let runtime = Builder::new_multi_thread()
        .worker_threads(8)  // 8 个工作线程
        .thread_name("tokio-worker")
        .thread_stack_size(2 * 1024 * 1024)  // 2MB 栈大小
        .enable_all()
        .build()
        .unwrap();

    // 配置调度器
    runtime.block_on(async {
        // 任务组
        let handles: Vec<_> = (0..10)
            .map(|i| {
                tokio::spawn(async move {
                    sleep(Duration::from_millis(100)).await;
                    println!("Task {} completed", i);
                    i
                })
            })
            .collect();

        // 等待所有任务完成
        let results = futures::future::join_all(handles).await;
        println!("All tasks completed: {:?}", results);
    });
}

// 自定义运行时配置
#[derive(Clone)]
struct AppConfig {
    max_connections: usize,
    request_timeout: Duration,
}

impl Default for AppConfig {
    fn default() -> Self {
        Self {
            max_connections: 1000,
            request_timeout: Duration::from_secs(30),
        }
    }
}

async fn run_server(config: AppConfig) {
    // 使用配置启动服务
    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await.unwrap();
    
    loop {
        let (socket, _) = listener.accept().await.unwrap();
        
        tokio::spawn(async move {
            process_connection(socket, config.clone()).await;
        });
    }
}
```

### Tracing 分布式追踪

```rust
use tracing::{info, warn, error, Span, instrument};
use tracing_opentelemetry::OpenTelemetrySpanExt;
use opentelemetry::trace::Tracer;
use opentelemetry_otlp::Protocol;

#[instrument(skip_all, fields(user_id = %user.id))]
async fn process_user_request(
    user: User,
    request: Request,
    state: AppState,
) -> Result<Response> {
    let span = Span::current();
    
    // 添加自定义属性
    span.set_attribute("request.type", request.kind.clone());
    
    info!("Processing request for user {}", user.id);
    
    let result = state
        .service
        .handle_request(request)
        .await;
    
    match &result {
        Ok(response) => {
            span.set_attribute("response.status", "success");
            info!("Request completed successfully");
        }
        Err(e) => {
            span.set_attribute("response.status", "error");
            span.set_attribute("error.message", e.to_string());
            error!("Request failed: {}", e);
        }
    }
    
    result
}

// 追踪客户端请求
async fn call_external_api(
    client: &reqwest::Client,
    endpoint: &str,
) -> Result<String> {
    let span = tracing::info_span!("external_api_call", 
        endpoint = %endpoint);
    
    async move {
        let response = client
            .get(endpoint)
            .send()
            .await?;
        
        span.set_attribute("http.status_code", response.status().as_u16());
        
        let body = response.text().await?;
        Ok(body)
    }
    .instrument(span)
    .await
}
```

### 数据验证与序列化

```rust
use validator::{Validate, ValidationError};
use serde::{Deserialize, Serialize};

#[derive(Validate, Deserialize, Serialize)]
struct CreatePostRequest {
    #[validate(length(min = 1, max = 200, message = "Title must be 1-200 characters"))]
    title: String,
    
    #[validate(length(min = 10, message = "Content must be at least 10 characters"))]
    content: String,
    
    #[validate(email(message = "Invalid email format"))]
    author_email: String,
    
    #[validate(range(min = 1, message = "Category ID must be positive"))]
    category_id: Option<u64>,
    
    #[validate(custom(function = "validate_tags"))]
    tags: Vec<String>,
}

fn validate_tags(tags: &[String]) -> Result<(), ValidationError> {
    if tags.len() > 5 {
        return Err(ValidationError::new("max_tags"));
    }
    
    for tag in tags {
        if tag.len() > 20 {
            return Err(ValidationError::new("tag_too_long"));
        }
    }
    
    Ok(())
}

impl CreatePostRequest {
    fn validate(&self) -> Result<(), Vec<ValidationError>> {
        validator::Validate::validate(self)
    }
}

// 使用示例
fn handle_create_post(body: String) -> Result<Post> {
    let request: CreatePostRequest = serde_json::from_str(&body)?;
    
    if let Err(errors) = request.validate() {
        return Err(AppError::Validation(errors.to_string()));
    }
    
    // 继续处理...
}
```

### 常见陷阱与最佳实践

1. **所有权陷阱**：避免在闭包中捕获可变引用导致生命周期问题。使用 `move` 关键字或适当克隆。

2. **性能陷阱**：不要在热路径中进行不必要的分配。使用 `String` 和 `Vec` 的预分配。

3. **异步陷阱**：避免在同步代码中调用异步函数。确保使用正确的运行时。

4. **错误处理**：不要忽略 `Result` 和 `Option`。使用 `?` 操作符传播错误。

5. **类型安全**：使用强类型定义领域模型，避免使用 `serde_json::Value` 传递数据。

6. **并发安全**：优先使用 `Arc<Mutex<T>>` 而不是 `Rc<RefCell<T>>`。在多线程环境中使用 `Arc`。

7. **内存泄漏**：注意循环引用。使用 `Weak` 引用打破循环。

8. **编译时间**：使用增量编译。分离大型模块减少编译时间。

9. **测试覆盖**：编写单元测试、集成测试和性能基准测试。

10. **文档注释**：使用 `cargo doc --document-private-items` 生成完整文档。

---

## 附录：Rust 项目模板与脚手架

### Cargo-generate 模板

```bash
# 安装 cargo-generate
cargo install cargo-generate

# 使用模板创建项目
cargo generate --git https://github.com/.../template

# 常用模板
# - axum-web-template
# - actix-web-template  
# - tokio-tracing-template
```

### 项目启动清单

- [ ] 配置 Cargo.toml（依赖、特性、profile）
- [ ] 配置 rust-toolchain.toml（指定 Rust 版本）
- [ ] 设置 `.cargo/config.toml`（构建配置）
- [ ] 配置日志系统（tracing）
- [ ] 配置错误处理（anyhow, thiserror）
- [ ] 设置 CI/CD（GitHub Actions）
- [ ] 配置代码格式化（rustfmt.toml）
- [ ] 配置 Clippy 检查（clippy.toml）
- [ ] 编写 README.md
- [ ] 添加 LICENSE
- [ ] 配置安全策略（SECURITY.md）

### 代码质量检查脚本

```bash
#!/bin/bash
# scripts/quality-check.sh

set -e

echo "=== Running Rust Quality Checks ==="

echo "1. Checking formatting..."
cargo fmt --check

echo "2. Running clippy..."
cargo clippy --all-targets --all-features -- -D warnings

echo "3. Running tests..."
cargo test --all-features

echo "4. Building documentation..."
cargo doc --no-deps --document-private-items

echo "5. Checking for security vulnerabilities..."
cargo audit

echo "6. Checking for outdated dependencies..."
cargo outdated -dR

echo "=== All checks passed! ==="
```

---

> [!TIP]
> Rust 生态系统正在快速发展。在选择依赖时，优先选择活跃维护、文档完善、社区活跃的 crate。定期更新依赖以获得最新特性和安全修复。

