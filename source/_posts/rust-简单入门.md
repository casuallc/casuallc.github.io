---
title: rust 简单入门
author: changqing
categories:
  - 2024
date: 2024-02-19 14:39:35
updated: 2024-02-19 14:39:35
tags:
---

# Rust 简介
Rust 是由 Mozilla 主导开发的通用、编译型编程语言。设计准则为“安全、并发、实用”，支持函数式、并行式、过程式以及面向对象的程式设计风格。

Rust 是一种系统编程语言，通常用于构建快速可靠的软件，特别是在需要内存安全和线程安全至关重要的情况下，如操作系统、游戏引擎和Web浏览器。

<!-- more -->

Rust 的一些关键特性：
- 无 GC：Rust 通过其所有权系统实现内存安全，该系统确保有效地管理内存而无需垃圾收集器。

- 零成本抽象：Rust 允许使用高级抽象而不产生运行时开销，使得编写既安全又高效的代码成为可能。

- 高并发：Rust 的所有权模型和类型系统在编译时防止数据竞争，确保线程安全，使开发人员可以放心地编写并发代码。

- Cargo 包管理器：Rust 配备了 Cargo，一个强大的包管理器和构建系统，简化了管理依赖项和构建项目的过程。

- 社区驱动的开发：Rust 拥有一个充满活力和积极的社区，为其开发、文档和生态系统做出贡献。

# 简答使用
> 参考：https://doc.rust-lang.org/book/title-page.html

## 安装（windows）
到 [Rust 官网](https://www.rust-lang.org/tools/install) 下载 Rust 安装器安装即可（可能需要先安装 Visual Studio）。

安装后打开 PowerShell 执行 rustc --version，如下：
``` powershell
PS C:\Users\casua> rustc --version
rustc 1.75.0 (82e1608df 2023-12-21)
PS C:\Users\casua>
```

有如上输出则证明安装成功，下面就可以编写测试代码进行功能测试了。

## Hello World
和其他编程语言一样，从 Hello World 开始了解 Rust。

首先我们需要创建一个文件夹：
```powershell
 mkdir rust-test 
```

然后创建 main.rs 文件，并写入如下代码：
``` rust
fn main() {
  println!("Hello World");
}
```

编译成可执行文件：
```powershell
powershell
```

编译成功后会生成 main.exe 和 main.pbd 文件，exe 是可执行文件，pbd 是包含调试信息的中间文件。
``` txt
PS E:\rust-test> ls


    目录: E:\rust-test


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2024/2/19     15:12         148480 main.exe
-a----         2024/2/19     15:12        1339392 main.pdb
-a----         2024/2/19     15:11             42 main.rs
```

执行：
```powershell
PS E:\rust-test> ./main
Hello World
PS E:\rust-test>
```

至此，我们第一个 rust 程序运行成功了。

## 包管理工具 Cargo
用 rustc 只能编译单个文件，如果是一个比较大的项目则需要使用 Cargo，Cargo 是一个包管理工具，可以管理和编译多个 Rust 文件。

Cargo 在安装 Rust 时已经默认安装了，所以我们可以直接使用。

首先创建一个工程：
```powershell
PS E:\rust-test> cargo new hello
     Created binary (application) `hello` package
PS E:\rust-test> cd .\hello\
PS E:\rust-test\hello> ls


    目录: E:\rust-test\hello


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2024/2/19     15:24                src
-a----         2024/2/19     15:24              8 .gitignore
-a----         2024/2/19     15:24            174 Cargo.toml


PS E:\rust-test\hello>
```

创建成功后会生成 Cargo.toml 文件，文件中记录了项目信息，包含版本、依赖等内容，如下：
```txt
[package]
name = "hello"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```

然后我们在 src 目录创建 main.rs，加上刚才测试的代码并使用 cargo 构建：
```powershell
PS E:\rust-test\hello> cargo build
   Compiling hello v0.1.0 (E:\rust-test\hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.51s
PS E:\rust-test\hello>
```

构建成功后在 target/debug 目录可以看到可执行文件 hello.exe：
```powershell
PS E:\rust-test\hello> .\target\debug\hello.exe
Hello World
PS E:\rust-test\hello>
```

我们还可以直接使用 cargo run 执行代码：
```powershell
PS E:\rust-test\hello> cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target\debug\hello.exe`
Hello World
PS E:\rust-test\hello>
```

# 所有权
Rust 没有垃圾回收，也不用像 C/C++ 那样手动申请内存、释放内存。在 Rust 中，每个值在某个时刻都有唯一一个所有者，所有者发生改变后，该值就不能再次使用。
- Rust 中每个值都有一个 owner。
- 同一时刻只能有一个 owner。
- 当离开 owner 范围后，该值不能再次使用。

## 方法调用
举例如下：
```rust
fn main() {
  let param = String::from("hello");
  owner_change(param);
  
  println!("value: {}", param); // 报错；param 的所有权已经转移到其他地方，因此这里不能再次使用了

}

fn owner_change(v: String) {
  println!("value in method: {}", v);
}

```

```powershell
PS E:\rust-test\hello> cargo run
   Compiling hello v0.1.0 (E:\rust-test\hello)
error[E0382]: borrow of moved value: `param`
 --> src\main.rs:5:25
  |
2 |   let param = String::from("hello");
  |       ----- move occurs because `param` has type `String`, which does not implement the `Copy` trait
3 |   owner_change(param);
  |                ----- value moved here
4 |
5 |   println!("value: {}", param); // 报错；param 的所有权已经转移到其他地方，因此这里不能再次使用了
      ...
  |                         ^^^^^ value borrowed here after move
  |
note: consider changing this parameter type in function `owner_change` to borrow instead if owning the value isn't necessary
 --> src\main.rs:9:20
  |
9 | fn owner_change(v: String) {
  |    ------------    ^^^^^^ this parameter takes ownership of the value
  |    |
  |    in this function
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |   owner_change(param.clone());
  |                     ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` (bin "hello") due to previous error
PS E:\rust-test\hello>
```


但我们可以在方法执行完再次获取到所有权，如下：
```rust
fn main() {
  let param = String::from("hello");
  let param = owner_change(param);
  
  println!("value: {}", param); // 正常输出
}

fn owner_change(v: String) -> String {
  println!("value in method: {}", v);
  v
}
```

## 变量传递
同样，如果一个变量赋予另外一个变量，则之前的变量不能再次使用，如下：
```rust
fn main() {
  let s1 = String::from("hello");
  let s2 = s1;
  println("s1: {}, s2: {}", s1, s2); // 报错
}
```

```powershell
PS E:\rust-test\hello> cargo run
   Compiling hello v0.1.0 (E:\rust-test\hello)
error[E0382]: borrow of moved value: `s1`
 --> src\main.rs:4:30
  |
2 |   let s1 = String::from("hello");
  |       -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |   let s2 = s1;
  |            -- value moved here
4 |   println!("s1: {}, s2: {}", s1, s2);
  |                              ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |   let s2 = s1.clone();
  |              ++++++++

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` (bin "hello") due to previous error
PS E:\rust-test\hello>
```

## 传递地址
在方法中传递引用所有权会发生改变，传递地址所有权不会发生改变，因此如下代码可以正常运行：
```rust
fn main() {
  let param = String::from("hello");
  let len = owner_change(&param);
  
  println!("value: {}, len: {}", param, len);

}

fn owner_change(v: &String) -> usize {
  println!("value in method: {}", v);
  v.len() // 非分号结尾表示返回值
}
```

运行结果如下：
```powershell
PS E:\rust-test\hello> cargo run
   Compiling hello v0.1.0 (E:\rust-test\hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target\debug\hello.exe`
value in method: hello
value: hello, len: 5
PS E:\rust-test\hello>
```

# 最后
上面体验了下 Rust 的基本用法，主要是所有权这块。Rust 在编译时强制用户按照约定的规则编写代码，增加了代码编写难度。一旦编译成功，则基本不可能出现内存泄漏问题。