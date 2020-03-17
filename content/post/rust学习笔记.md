---
title: "Rust学习笔记"
date: 2020-03-12T20:08:38+08:00
lastmod: 2020-03-17T20:08:38+08:00
draft: false
tags: ["rust"]
categories: ["language"]
mathjax: false
---

本文主要记录Rust学习过程中的一些笔记。  
<!--more-->

# 安装
参考 [官方安装页面](https://www.rust-lang.org/tools/install) 的说明操作即可。  

# IDE
使用 [VSCode](https://code.visualstudio.com/Download) 以及`cargo`工具即可。  

# 教程
可以参考以下教程，本文主要记录一些没讲到的或者需要注意的点。  
- [Rust编程语言(英文版)](https://doc.rust-lang.org/book/)  
- [Rust编程语言(中文版)](https://kaisery.github.io/trpl-zh-cn/)  

# 变量与可变性
变量默认不可变，添加`mut`关键字允许修改。  

# 所有权
## 所有权规则
> 值有且只有一个拥有所有权的变量，变量离开作用域后值被丢弃。

有点类似于cpp的RAII模式，资源依附于变量的生命周期，在构造时申请、在析构时释放。  
可以`Copy`的值赋值给其他变量实际是复制一份，而可以`Drop`的值则是移交所有权，原变量不再有效。  
移交所有权类似于cpp的`move`语义，而且还有静态分析确保失效变量不再使用。  
所有权规则确保资源得以在合适的时机正确释放，不会内存泄露、不会`double free`，不需要运行时GC。  

## 借用与生命周期
### 借用
引用就是值的借用，不拥有值的所有权(也就不负责值的释放)。  
Rust的引用更像cpp的指针(因为需要解引用，实现`Deref`)。  
引用的作用域从声明处开始，直到最后一次使用结束。  
为了避免数据竞争，Rust规定引用作用域内要么有多个不可变引用，要么最多一个可变引用。  
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let mut s = String::from("hello world");
    let word = first_word(&s);

    s.clear(); // 编译错误!
    println!("the first word is: {}", word); // word引用无效值
}
```
借用规则有效避免引用到无效的值。  

### 生命周期
另外Rust还引入生命周期的概念，确保引用的有效性，防止野指针(悬垂指针)。  
比如说返回引用的函数，返回值必须依附于某个参数的生命周期。  
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

# 枚举与模式匹配
Rust的枚举比较特别，并不是其他语言常见的数值枚举。  
Rust的枚举可以绑定数据，有点类似于cpp的union。  
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```
内置的`Option`和`Result`两种枚举很好展示了其实用性。  
```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
`Option`很好地解决空值情况(特别是空指针)。  
`Result`增强了错误处理的能力，不再是单一的错误码，还附带了错误信息。  

# 错误处理
不可恢复错误，使用 `panic` 系列宏。  
可恢复错误，使用模式匹配处理 `Result` 错误类型。  

# 泛型
支持泛型，与其他语言概念类似，增加了生命周期的概念。  
```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

# 测试
支持单元测试、集成测试。  

# 闭包
支持闭包，与其他语言概念类似，增加获取所有权、借用的支持。  

# 智能指针
- `Box<T>`  
  类似于cpp的`unique_ptr`
- `Rc<T>, Arc<T>`  
  类似于cpp的`shared_ptr`，`Arc<T>`是并发安全的。  
- `Weak<T>`  
  类似于cpp的`weak_ptr`
- `RefCell<T>`  
  运行时检查借用规则的引用。  

# unsafe
可以执行裸指针操作、调用外部函数接口([FFI](https://doc.rust-lang.org/book/first-edition/ffi.html)) 等。  
```rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}

extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

# 宏
类似lisp的宏，主要为了生成代码。常用的`vec!`,`panic!`等就是宏实现的。  

# 异步IO
可以使用 [tokio](https://github.com/tokio-rs/tokio) ，支持 `async, await`。  

# 反射
不支持。  

# 参考链接
- [现代化的内存管理：Cpp vs Rust](https://rust.cc/article?id=0c72fb25-2b3a-4896-9b3b-50ff8980728f) 

