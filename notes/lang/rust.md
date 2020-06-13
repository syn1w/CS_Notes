# 一、入门

## 1. cargo

```sh
cargo --version
cargo new hello --bin # 新建
```

```toml
# Cargo.toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["czn <iamczn.cpp@gmail.com>"]

[dependencies]
```

```sh
cargo build  # 构建
cargo check  # 确保项目可以编译，但是不产生可执行文件
cargo run    # 运行

cargo build --release
```

## 2. 猜猜看游戏

```rust
extern crate rand;

use std::io;            // 引入作用域
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess a number");

    let num = rand::thread_rng().gen_range(1, 100);

    loop {
        println!("Enter your guess: ");
        let mut guess = String::new();    // mut 表示可变

        io::stdin().read_line(&mut guess)    // & 为引用， &mut 可变引用
            .expect("failed to read line");  // 错误处理

        let guess: u32 = match guess.trim().parse() { // : u32 显示说明类型
            Ok(n) => n,                               // match 表达式由分支构成，和 switch 类似
            Err(_) => continue,
        };
    
        match guess.cmp(&num) {
            Ordering::Less => println!("too small!"),
            Ordering::Greater => println!("too big!"),
            Ordering::Equal =>  {
                println!("you win!");
                break;
            }
        }
    }
}

```

# 二、通用编程概念

## 1. 变量和可变性

**变量默认是不可变的**

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;                                 // cannot assign twice to immutable variable
    println!("The value of x is: {}", x);
}
```

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}

The value of x is: 5
The value of x is: 6
```



```rust
// const 不允许使用 mut，必须注明类型。
// 只能用于常量表达式，不能作为函数调用的结果。
// 可以在如何作用域声明，包括全局
const MAX_POINTS: u32 = 100_000;
```

**隐藏**

```rust
fn main() {
    let x = 5;
    let x = x + 1;
    let x = x * 2;

    println!("The value of x is: {}", x);
}

// The value of x is: 12
// 再次使用 let，实际创建了新变量，只是复用了这个变量名，类型可变
// 猜测：一般是不同作用域有作用
```

## 2. 数据类型

任何值都属于一种明确的**类型**。内建的类型：标量和复合。  

Rust 是静态类型语言。  

**标量**（*scalar*）类型代表一个单独的值。Rust 有四种基本的标量类型：整型、浮点型、布尔类型和字符类型。   

|  长度  | 有符号 | 无符号 |
| :----: | :----: | :----: |
| 8-bit  |   i8   |   u8   |
| 16-bit |  i16   |  u16   |
| 32-bit |  i32   |  u32   |
| 64-bit |  i64   |  u64   |
|  arch  | isize  | usize  |



|    数字字面值    |     例子      |
| :--------------: | :-----------: |
|     Decimal      |   `98_222`    |
|       Hex        |    `0xff`     |
|      Octal       |    `0o77`     |
|      Binary      | `0b1111_0000` |
| Byte (`u8` only) |    `b'A'`     |



数字的默认类型是 `i32`，通常是最快的，甚至是 64 位系统上也是。  

`isize` 或者 `usize` 主要是某些集合的索引。  



**浮点型**  

分为 `f32` 和 `f64`。 默认类型是 `f64`。   



**布尔类型**  

`bool`，`true` 或者 `false`  

```rust
let t = true;
let f: bool = false;
```



**字符类型**  

Rust 的 `char` 代表一个 unicode 标量。  



**复合类型**  

元组(*tuple*)和数组(*array*)  

```rust
// tuple
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    let tup = (500, 6.4, 1);
    let (x, y, z) = tup;
    
    println!("The value of y is: {}", y);
    println!("The value of y is: {}", tup.1); // 6.4
}
```

```rust
// array
fn main() {
    let a = [1,2,3,4,5];
    let first = a[0];
    let second = a[1];
    
    let index = 10;
    let element = a[index];   // index out of bounds, runtime error
}
```



## 3. 函数

在函数签名中，必须声明每个参数的类型。  

函数体是由一系列的语句和一个可选的表达式构成。  

Rust 是一个基于表达式的语句。  

```rust
fn main() {
    func(42);
}

fn func(x: i32) {
    println!("The value of x is: {}", x);
}
```

**语句与表达式**  

**语句**（*Statements*）是执行一些操作但不返回值的指令。**表达式**（*Expressions*）计算并产生一个值。 

`x = y = 6` 在 Rust 是非法的。`y = 6` 是一个语句，并不返回值。  

在 Rust 中，函数调用是表达式，宏调用也是表达式，`{}` 作用域也是表达式   

```rust
fn main() {
    let x = 5;
    
    let y = {
        let x = 3;
        x + 1;
    }
    
    println!("The value of y is: {}", y);    // 4
}
```



**返回值**

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();
    println!("The value of x is: {}", x);
}
```



```rust
fn plus_one(x: i32) -> i32 {
    
}
```



## 2.4 控制语句

```rust
// if 语句
let number = 3;
if number < 5 {
    println!("true");
}
else {
    println!("false");
}
```

`if` 条件中的类型必须是 `bool` 类型

```rust
if cond1 {
    // ...
}
else if cond2 {
    // ...
}
else if cond3 {
    // ...
}
else {
    // ...
}
```

在 `let` 语句中使用 `if` 表达式

```rust
fn main() {
    let cond = true;
    let number = if cond {
        5
    }
    else {
        6
    };    
}
// 分支的类型必须相同
```

```rust
// loop
fn main() {
    loop {
        println!("again");
    }
}
// 无限循环
```

```rust
// while
fn main() {
    let mut n = 3;
    
    while n != 0 {
        println!("{}!", n);
        n = n - 1;
    }
}
```

```rust
// for range with iterator
fn main() {
    let a = [10, 20, 30, 40, 50];
    for e in a.iter() {
        println!("The value is: {}", e);
    }
}

// for range
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);  // 4 3 2 1
    }
}

```



# 三、所有权

## 1. 是什么?

Rust 的核心功能（之一）是 **所有权**（*ownership*）。  

内存被一个所有权系统管理，它拥有一系列的规则使编译器在编译时进行检查。任何所有权系统的功能都不会导致运行时开销。   



**所有权规则**

- Rust 中每一个值都有一个称之为其 **所有者**（*owner*）的变量。
- 值有且只能有一个所有者。
- 当所有者（变量）离开作用域，这个值将被丢弃。



需要寻找一个储存在堆上的数据来探索 Rust 是如何知道该在何时清理数据的。 下面将用 `String` 类型来解释所有权。  

所以先对 `String` 类型(类似于 `std::string`)进行介绍：  

我们已经见过 字符串字面值了(类似于 `const char*`)，它被硬编码进程序里。 `let s = "42";`  

在需要可变字符串的场景怎么办？比如获取用户输入。为此，`String` 就产生了。  

```rust
let s = String::from("hello");
s.push_str(", world!");
println!("{}", s);
```

`String::from` 请求其所需的内存，内存在拥有它的变量离开作用域后被自动释放。  

Rust 会在作用域结束调用一个特殊的函数，`drop`，类似于 c++ 中的析构。也就是采用 RAII 机制。



当多个变量和同一数据交互时会发生什么？



### (1) 变量与数据交互方式：移动

```rust
let x = 5;   // 第一个 5 入栈，x 的地址就是第一个 5 的地址
let y = x;   // 生成 x 的拷贝，第二个 5 入栈。y 的地址就是第二个 5 的地址。
```

接下来看在堆上分配内存的数据结构，`String`

```rust
let s1 = String::from("hello");
let s2 = s1;
```

![s1](https://kaisery.github.io/trpl-zh-cn/img/trpl04-01.svg)

当把 `s1` 赋值给 `s2`，`String` 的数据被复制了，也就是复制了 `ptr, len, capacity`，但是并没有复制堆上的数据。

![s1=s2](https://kaisery.github.io/trpl-zh-cn/img/trpl04-02.svg)

这样就产生了一个问题，当出作用域时，自动调用 `drop` 清理堆内存。`s1` 和 `s2` 都尝试释放相同内存，产生了 *double free* 问题。两次释放（相同）内存会导致内存污染，它可能会导致潜在的安全漏洞。 

在 Rust 中的处理，直接使用移动(*move*)语义。Rust 认为 `s2 = s1` 之后，`s1` 不再有效。

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s1);
// 编译错误：value used here after move 
```



### (2) 变量与数据交互方式：拷贝

*注：原文为克隆 clone，为了和 c++ 做出统一，使用 拷贝*

```rust
let s1 = String::from("hello");
let s2 = s1.clone();
println!("s1 = {}, s2 = {}", s1, s2);
```

只在栈上的数据类型，赋值语句进行的操作是拷贝，而在堆上分配空间的类型，赋值操作进行的操作是移动。  

Rust 有一个 `Copy` trait 的特殊注解，可以用类似整型那样的方式在栈上存储对象。不允许实现了 `Drop` trait 的类型使用 `Copy` trait。

一些可以 `Copy` 的类型：

- 所有的整型
- 布尔类型
- 浮点类型
- 所有包含类型是 `Copy` 的元组



### (3) 所有权与函数

将值传递给函数在语义上与给变量赋值相似。向函数传递值可能会移动或者复制，就像赋值语句一样。

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope.

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so s is no longer valid here.

    let x = 5;                      // x comes into scope.

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it’s okay to still
                                    // use x afterward.
}

fn takes_ownership(some_string: String) { // some_string comes into scope.
    println!("{}", some_string);
}

fn makes_copy(some_integer: i32) { // some_integer comes into scope.
    println!("{}", some_integer);
}
```

返回值也可以转移作用域。   

变量的所有权总是遵循相同的模式：将值赋值给另一个变量时移动它。当持有堆中数据值的变量离开作用域时，其值将通过 `drop` 被清理掉，除非数据被移动为另一个变量所有。 



也可以使用 元组 来返回多个值。

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String.

    (s, length)
}
```

但是这未免有些形式主义，而且这种场景应该很常见。幸运的是，Rust 对此提供了一个功能，叫做 **引用**（*references*）。

 

## 2. 引用和借用

```rust
fn main() {
    let s1 = String::from("hello");
    let len = get_len(s1);
    println!("The length of {} is {}", s1, len);
}

fn get_len(s: &String) -> usize {
    s.len()
}
```

![reference](https://kaisery.github.io/trpl-zh-cn/img/trpl04-05.svg)

`&` 符号就是引用，相对的操作是解引用`*`。

Rust 的引用类似于 c/c++ 的 `const T*`。也就是没有所有权而已。

获取引用作为函数参数成为**借用(*borrowing*)**。正如现实生活中， 如果一个人拥有某样东西，你可以从他那里借来。当你使用完毕，必须还回去。 

如果尝试修改借用的变量，发生编译错误，cannot borrow as mutable。引用默认是不可变的。



**可变引用**

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(str: &mut String) {
    str.push_str(", world");
}
```

不过可变引用有一个很大的限制：在特定作用域中的特定数据有且只有一个可变引用。 下面的代码会产生编译错误：

```rust
let mut s = String::from("hello");
let r1 = &mut s1;
let r2 = &mut s2;  // second mutable borrow occurs here
```

这个限制的好处是 Rust 可以在编译时就避免数据竞争。 



**数据竞争**（*data race*）是一种特定类型的竞争状态，它可由这三个行为造成：

- 两个或更多指针同时访问同一数据。
- 至少有一个这样的指针被用来写入数据。
- 不存在同步数据访问的机制。

数据竞争会导致未定义行为，难以在运行时追踪，并且难以诊断和修复；Rust 避免了这种情况的发生，因为它甚至不会编译存在数据竞争的代码！ 

```rust
// 使用大括号来创建一个新的作用域来允许拥有多个可变引用
let mut s = String::from("hello");
{
    let r1 = &mut s;
}
let r2 = &mut s;
```

我们**也不能在拥有不可变引用的同时拥有可变引用。**

<br>

**空悬引用(Dangling references)**

在 Rust 中编译器确保引用永远也不会变成悬垂状态：当我们拥有一些数据的引用，编译器确保数据不会在其引用之前离开作用域。

让我们尝试创建一个悬垂引用，Rust 会通过一个编译时错误来避免： 

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {           
    //         ^ expected lifetime parameter            
    let s = String::from("hello");
    &s
}
```

当 `dangle` 代码执行完毕后，`s` 被释放，先前 `return` 的 `&s` 也就是一个无效的 pointer。Rust 不会允许这样。  



**引用规则总结**

这里的引用类似于与 c/c++ 的指针

- 在任意给定时间，**只能** 拥有如下中的一个：
  - 一个可变引用。
  - 任意数量的不可变引用。
- 引用必须总是有效的。



## 3. slice

另一个没有所有权的类型是 `slice`，`slice` 允许引用集合中的一段连续的元素序列，而不用引用这个集合。

编写一个在字符串中找到第一个单词的函数。(第一个空格之前就算一个单词，无空格就返回整个字符串)。

```rust
fn first_word(s: &String) -> ?
```

并没有获取部分字符串的方法，所以可以返回单词结尾的索引。

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i
        }
    }
    s.len()
}
```

`enumerate ` 包装 `iter` 的结果并返回一个元组，每个元素是元组的一部分。返回元组的第一个元素是索引，第二个元素是集合中元素的引用。 

如果在字符串发生改变之后，再次使用索引值来获取第一个单词，将会产生错误。

如果记录单词的开始和结束位置就更麻烦了。

Rust 提供了一种解决方案，就是 `String slice`



**String slice**

`String slice` 是 `String` 中一部分值的引用

```rust
let s = String::from("hello world");
let hello = &s[0..5];
let world = &s[6...11];
```

`[start(default = 0)...end(default = len)]` 语法代表 $[start, end)$，类似于 c++ 中的操作。

![slice](https://kaisery.github.io/trpl-zh-cn/img/trpl04-06.svg)

> 注意：字符串 slice range 的索引必须位于有效的 UTF-8 字符边界内，如果尝试从一个多字节字符的中间位置创建字符串 slice，则程序将会因错误而退出。 



使用 `slice` 来重写上面的那个函数

```rust
fn first_word(s: &String) -> &str { // slice 类型声明为 &str
    let bytes = s.as_bytes();
    
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i]
        }
    }
    
    &s[..]
}
```

`slice` 就类似于 `struct slice { const char* ptr, size_t len };`

使用上面的函数

```rust
fn main() {
    let mut s = "hello world";
    let word = first_word(s); // immutable borrow occurs here
    s.clear(); // Error, mutable borrow occurs here
}
```

字符串字面值就是 `slice`

```rust
let s = "hello world"; // type of s is &str
```

字符串 slice 作为参数

```rust
fn first_word(s: &str) -> &str {
    
}

first_word(string);
first_word(string_literal);
```

所有权系统影响了 Rust 中很多其他部分的工作方式，所以我们还会继续讲到这些概念，这将贯穿本书的余下内容。 

# 四、结构体

## 1. 定义和实例化

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

```rust
let user1 = User {
    email: String::from("xxx@yyy.com"),
    username: String::from("someusername");
    active: true,
    sign_in_count: 1,
}
```

如果想要结构体实例可变

```rust
let mut user1 = User {
    email: String::from("xxx@yyy.com"),
    username: String::from("someusername");
    active: true,
    sign_in_count: 1,
}
user1.email = String::from("zzz@example.com");
```

注意整个实例必须是可变的；Rust 并不允许只将特定字段标记为可变。另外需要注意同其他任何表达式一样，我们可以在函数体的最后一个表达式中构造一个结构体的新实例，来隐式地返回这个实例。 

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

如果参数名与字段名都完全相同，我们可以使用 **字段初始化简写语法** 

```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```



**使用结构体更新语法从其他实例创建新实例**

使用旧实例的大部分值但改变其部分值来创建一个新的结构体实例通常是很有帮助的。这可以通过 **结构体更新语法**（*struct update syntax*）实现。  

```rust
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```



**元组结构体**

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);
let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

注意 `black` 和 `origin` 值是不同的类型，元组结构体实例类似于元组：可以将其解构为单独的部分，也可以使用 `.` 后跟索引来访问单独的值。  



**没有任何字段的结构体**  

被称为 **类单元结构体**（*unit-like structs*）因为它们类似于 `()`，即 unit 类型。   

常常在你想要在某个类型上实现 trait 但不需要在类型内存储数据的时候发挥作用。 C++ 中也很常见。



**结构体数据的所有权**

`User` 结构体的定义中，我们使用了自身拥有所有权的 `String` 类型而不是 `&str`字符串 slice 类型。 这是一个有意而为之的选择，因为我们想要这个结构体拥有它所有的数据，为此只要整个结构体是有效的话其数据也是有效的。 

可以使结构体储存被其他对象拥有的数据的引用，不过这么做的话需要用上 **生命周期**（*lifetimes*），这是一个第十章会讨论的 Rust 功能。 



## 2. 使用结构体

**使用元组**

```rust
fn main() {
    let rect = (30, 50);
    println!("area: {}", area(rect));
}

fn area(rect: (u32, u32)) -> u32 {
    rect.0 * rect.1
}
```



**使用结构体**

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect = Rectangle{width: 30, height: 50};
    println!("area: {}", area(rect));
}

fn area(rect: &Rectangle) -> u32 {
    rect.width * rect.height
}
```



**通过派生 trait 增加功能**

如果尝试 `println!("rect: {}", rect)` 

将会产生编译错误 `the trait bound Rectangle: std::fmt::Display is not satisfied`

Rust 不尝试猜测我们的意图所以结构体并没有提供一个 `Display` 实现。 

```rust
#[derive(Debug)]                        // 派生 Debug trait
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    println!("rect1 is {:?}", rect1);   // :? 表示使用 `Debug` 的输出格式
}

// rect1 is Rectangle { width: 30, height: 50 }
```

如果使用 `{:#?}` 可以处理更大的结构体，输出像这样

```txt
rect1 is Rectangle {
    width: 30,
    height: 50
}
```



## 3. 方法

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
    
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

开始使用 `&self` 来替代 `rectangle: &Rectangle` 

方法可以选择获取 `self` 的所有权，或者像我们这里一样不可变地借用 `self`，或者可变地借用 `self`，就跟其他别的参数一样。 



**关联函数**

`impl` 块的另一个有用的功能是：允许在 `impl` 块中定义 **不** 以 `self` 作为参数的函数。这被称为 **关联函数**（*associated functions*），因为它们与结构体相关联。 

关联函数经常被用作返回一个结构体新实例的构造函数。 

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```



可以分散为多个 `impl` 块。



# 五、枚举和模式匹配

枚举允许你通过列举可能的值来定义一个类型。  

探索一个特别有用的枚举，叫做 `Option`，它代表一个值要么是某个值要么什么都不是。   

在 `match` 表达式中用模式匹配，针对不同的枚举值编写相应要执行的代码 。  

最后会涉及到 `if let`，另一个简洁方便处理代码中枚举的结构。   

Rust 中的枚举与 F#、Haskell 等函数式编程语言中的**代数数据类型**最为相似。  



## 1. 枚举

```rust
enum IpAddrKind {
    V4,
    V6,
}

let ipv4 = IpAddrKind::V4;

fn route(iptype: IpAddrKind) { }
route(IpAddrKing::V4);
```

和结构体一起使用

```rust
struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```



使用枚举并将数据直接放进每一个枚举成员而不是将枚举作为结构体的一部分。 

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let ipv4 = IpAddr::V4(String::from("127.0.0.1"));
let ipv6 = IpAddr::V6(String::from("::1"));
```

枚举中使用不同类型也是可以的

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let ipv4 = IpAddr::V4(127, 0, 0, 1);
let ipv6 = IpAddr::V6(String::from("::1"));
```



**标准库中 IpAddr 的定义**

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

可以将任意类型的数据放入枚举成员中：例如字符串、数字类型或者结构体。甚至可以包含另一个枚举！   



**再来看一个实际例子**  

如果有一个用于处理 Message 类型的实际案例。  

在过程式语言中，只能判断消息种类，只后调用对应的方法  

```c
typedef enum MessageType {
    Quit, Move, Write, ChangeColor
} MessageType;

typedef union {
    QuitMessage, MoveMessage, WriteMessage, ChangeColorMessage    
} MessageValue;

struct Message {
    MessageType type;
    MessageValue value;
};

void call(const Message* pMsg) {
    switch (pMsg->type) {
        // ...
    }
}
```

在面向对象语言中，可以使用继承

```c++
class Message {
public:
    // ...
    virtual void call() = 0;
    virtual ~Message() {}
};

class QuitMessage : public Message {
public:
    // ...
    void call() override {
        // ...
    }
};

class MoveMessage : public Message {
public:
    // ...
    void call() override {
        // ...
    }
};

// ...

std::unique_ptr<Message> msg = std::make_unique<QuitMessage>();
msg->call();  // quit
```

在 Rust 中，可以直接使用枚举来解决

```rust
enum Message {
    Quit,                    // 没有关联任何数据。
    Move { x: i32, y: i32 }, // 包含一个匿名结构体
    Write(String), 
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // ....
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```



**`Option`**  

Rust 并没有其他的语言的空值 NULL。但是可以拥有一个可以编码存在或者不存在概念的枚举，这个枚举就是 `Option<T>`。而且定义在标准库中。

```rust
enum Option<T> {
    Some(T),
    None,
}
```

`Option<T>` 不需要显式引入作用域。另外成员也不需要。可以直接使用 `Some` 和 `None`

`<T>` 语法是泛型，之后再讲。

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number： Option<i32> = None;
```

`Option<T>` 和 `T` 是不同的类型。编译器不允许像一个 `T` 类型那样使用 `Option<T>`。

```rust
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
//          ^ no implementation for `i8 + std::option::Option<i8>`
```

当在 Rust 中拥有一个像 `i8` 这样类型的值时，编译器确保它总是有一个有效的值。我们可以自信使用而无需判空。只有当使用 `Option<i8>`（或者任何用到的类型）的时候需要担心可能没有一个值，而编译器会确保我们在使用值之前处理为空的情况。   

在对 `Option<T>` 进行 `T` 的运算之前必须将其转换为 `T`。   

为了拥有一个可能为空的值，我们必须要显式的将其放入对应类型的 `Option<T>` 中。接着，当使用这个值时，必须明确的处理值为空的情况。任何地方一个值不是 `Option<T>` 类型的话，我们就 **可以** 安全的认为它的值不为空。   



## 2. match 控制流运算符

`match` 的力量来源于模式的表现力以及编译器检查，它确保了所有可能的情况都得到处理。   

同样地，值也会通过 `match` 的每一个模式，并且在遇到第一个 “符合” 的模式时，值会进入相关联的代码块并在执行中被使用。   

```rust
enum Coin {
    Penny,     // 1
    Nickel,    // 5
    Dime,      // 10
    Quarter,   // 25
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => { 
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

每个分支相关联的代码是一个表达式，而表达式的结果值将作为整个 `match` 表达式的返回值。   

如果分支代码较短的话通常不使用大括号，正如示例 6-3 中的每个分支都只是返回一个值。如果想要在分支中运行多行代码，可以使用大括号。 



**绑定值的模式 ** 

枚举成员存放数据，像上一节的 Message 中那样。我们仍然以上一节例子为例(因为上一节使用类型要多一些)  

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) -> u32 {
        match self {
            Message::Quit => {
                println!("quit");
                0
            },
            Message::Move{x, y} => {
                println!("Move ({}, {})", x, y);
                1
            },
            Message::Write(msg) => {
                println!("Write {}", msg);
                2
            },
            Message::ChangeColor(r, g, b) => {
                println!("ChangeColor {}, {}, {}", r, g, b);
                3
            },
        }
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```





**匹配 `Option<T>`**

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i+1)
    }
}
```

匹配是穷尽的，如果没有处理所有情况，将产生编译错误。



**`_` 通配符**  

`_` 会匹配所有模式。通常将其放在其他分支之后。

```rust
let some_u8 = 0u8;
match some_u8 -> u8 {
    1 => 1,
    3 => 3,
    5 => 5,
    7 => 7,
    _ => 0
}
```

如果 `match` 只关心很少的分支时，`match` 就显得冗长。所以提供了 `if let`  

## 3. `if let`

```rust
// match 只关心 Some(3) 时的执行代码
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

可以使用更简短的方式来编写上面的代码

```rust
if let Some(3) = some_u8 {
    println!("three");
}
else {
    // 匹配 `_`
}
```

`if let` 通过 `=` 分割一个模式和一个表达式。工作方式和 match 相同，不过会失去 `match` 的穷尽性检查(`_`)。可以认为 `if let` 是 `match` 的一个语法糖，它当值匹配某一模式时执行代码而忽略所有其他值。 



# 六、模块

为了复用和更好地组织代码，最终你会将功能移动到其他函数中。  

**模块**（*module*）是一个包含函数或类型定义的命名空间，你可以选择这些定义能（公有）或不能（私有）在其模块外可见。  

- 使用 `mod` 关键字声明新模块。此模块中的代码要么直接位于声明之后的大括号中，要么位于另一个文件。
- 函数、类型、常量和模块默认都是私有的。可以使用 `pub` 关键字将其变成公有并在其命名空间之外可见。
- `use` 关键字将模块或模块中的定义引入到作用域中以便于引用它们。



## 1. `mod` 与文件系统

```shell
cargo new communicator --lib
cd communicator
```

在 `src/lib.rs` 中，自动生成

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

之后使用 `super` 访问父模块部分会介绍 `#[]` 和 `mod tests` 语法



**模块定义**

```rust
mod network {
    fn connect() {
    }
}
```

`mod` 后为模块的名字 `network`，代码位于 `network` 命名空间中。如果想要在模块外调用这个函数，指定模块名并使用命名空间语法 `::`。比如 `network::connect()`  

在一个文件中，可以存在多个模块。也可以将模块放到其他模块中。

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}

// mod client {
//    fn connect() {
//    }
//}
```

作为库入口点的文件是 `src/lib.rs`。对于创建库模块来说，`src/lib.rs` 没有什么特殊意义，也可以用 crate 的 `src/main.rs` 创建模块。 

```txt
communicator
 └── network
     └── client
```



**把模块移动到其他文件**  

接下来只为说明模块和文件，感觉例子在真实项目中并不符合逻辑。

```rust
// src/lib.rs
mod client;    // 声明 `client` 模块，Rust 会去寻找另一个定义代码的位置

mod network {
    fn connect() {
        
    }
    
    mod server() {
        fn connect() {
            
        }
    }
}
```

```rust
// src/client.rs， 对应模块名的外部文件
// 这个文件并不需要 mod 声明，仅仅提供 client 模块的内容
// 这里加上 mod，相当于增加了一个叫做 client 的模块
fn connect() {
}
```



Rust 只知道 `src/lib.rs`，如果需要在 `src/lib.rs` 中告诉 Rust 去寻找其他文件，`clinet` 模块和文件同名  



把 `network` 模块也提取到外部文件

```rust
// src/lib.rs
mod client;
mod network;
```

```rust
// src/network.rs
fn connect() {
}

mod server {
    fn connect() {
    }
}
```



如果要提取嵌套模块 `network/server` 也需要对应的目录   

```txt
└── src
    ├── client.rs
    ├── lib.rs
    └── network
        ├── mod.rs      // 原先的 network.rs
        └── server.rs
```



## 2. 使用 `pub`

在同一目录下创建使用库的代码

```rust
// src/main.rs
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

`extern crate` 指令将 `communicator` 库 crate 引入到作用域。  

我们的包现在包含 两个crate。 一个是 `src/main.rs`，另一个是 `src/lib.rs`  

`communicator` 为**根模块**  

之后 build 出现编译错误   

```txt
error[E0603]: module `client` is private
error[E0603]: function `connect` is private
```

增加 `pub`  

```rust
// src/lib.rs
pub mod client;

// src/client.rs
pub fn connect() {
}
```

如果想要去掉未使用的编译警告，需要把未使用的模块和函数声明为 `pub`



**私有性规则**

- 如果一个项是公有的，它能被任何父模块访问
- 如果一个项是私有的，它能被其直接父模块及其任何子模块访问



## 3. 引入不同模块中的名称

使用 `use` 将名称导入作用域

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {
            }
        }
    }
}

use a::series:of;

fn main() {
    of::nested_modules();
}
```

枚举也可以使用 `use` 来引入命名空间，因为枚举也像模块一样组成了某种命名空间

```rust
enum Color {
    Red, 
    Yellow,
    Green,
}

use Color::{Red, Yellow};

fn main() {
    let red = Red;
}
```



**使用 glob 将所有名称引入作用域**

```rust
enum Color {
    Red, Yellow, Green,
}
use Color::*;
```



**使用 `super` 访问父模块**

在创建 lib 后，会生成一个 `tests` 模块，这个是为了测试库的模块，模块层次结构像这样：

```txt
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```



```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}

// 使用 cargo test 命令之后，产生了编译错误 Use of undeclared type or module `client`
// 失败的原因是路径是相对于当前模块的，在这里就是 tests。
// 就类似于 cd 到了 tests 路径下，找不到 client 模块必须 cd .. 返回上级目录
// 使用 `::` 来从根目录开始
::client::connect();

// cargo test 编译成功
```



但是在实际使用中，如果说模块路径有一定深度的话，可以使用 `super` 来转移到上一级模块

```rust
super::client::connect();
// 也可以使用 use 语句
use super::client;
client::connect();
```



# 七、通用集合类型

## 1. vector

`Vec<T>`

```rust
let v: Vec<i32> = Vec::new();
let mut v = vec![1,2,3];   // vec! 宏

v.push(4);
v.pop();

let third: &i32 = &v[2];
let third: Opetion<&i32> = v.get(2);

// 如果尝试访问越界元素
let not_exist = &v[100];     // panic! 程序崩溃
let not_exist = v.get(100);  // None
```

**无效引用** 

```rust
// 类似于迭代器失效的问题
let mut v = vec![1,2,3,4,5];
let first = &v[0];
v.push(6); // mutable borrow occurs here
```



**遍历**

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;      // 改变 i 指向的值，必须使用 * 获取 i 中的值
}
```



**使用枚举来存储多种类型**

vector 只能存储相同类型的元素，这是很不方便的。如果不同类型的值时，可以使用枚举。

```rust
enum SheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SheetCell::Int(42),
    SheetCell::Text(String::from("blue")),
    SheetCell::Float(3.14),
]
```



## 2. String

Rust 语言层面只有一种字符串类型：`str`，字符串的 `slice`。通常以借用的形式出现，`&str`。类似于 c/c++ 中的 `const char*`。 

`String` 是由标准库提供的。可增长的、可变的、有所有权的、UTF-8 编码的字符串类型。  

标准库中还有其他的字符串类型，比如 `OsString`、`OsStr`、`CString` 和 `CStr`。  

但是这里着重介绍 `String`  

```rust
let mut s = String::new();  // empty string


let data = "initial contents";
let s = data.to_string();

let s = "initial contents".to_string();

let s = String::from("initial contents");
```

```rust
let mut s = String::from("foo");
s.push_str("bar");
```



**使用 `+` 和 `format!` 宏连接字符串**

```rust
let s1 = String::from("hello, ");
let s2 = String::from("world");
let s3 = s1 + &s2;
```

使用 `+` 运算符的调用签名，不过标准库中式泛型，这儿具化了

```rust
fn add(self, s: &str) -> String { ... }
```

上面 `&s2` 的类型是 `&String`。但是需要 `&str` 类型，这儿发生了隐式强制类型转换，`&String` 强制转化为 `&str`。在 Rust 中，叫做**解引用强制多态**（*deref coercion*），相当于 `&s2` 变成了 `&s2[..]`。  

需要注意的是 `add` 的第一个参数为 `self`，没有使用 `&`。实际上会获取 `s1` 的所有权，在 `+` 操作之后，`s1` 将会失效。  



如果拼接多个字符串，`+` 就显得笨重了

```rust
let s = s1 + "-" + &s2 + "-" + &s3;
```

使用 `format!` 来格式化字符串，不会获取任何参数的所有权

```rust
let s = format!("{}-{}-{}", s1, s2, s3); 
```



**索引字符串**

```rust
let s1 = String::from("hello");
let h = s1[0];
        ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
```

错误说明了 Rust 字符串不支持索引。



**内部表现**

`String` 是 `Vec<u8>` 的封装，由于是 `utf-8 ` 编码，很可能出现多个字节表示一个字形的情况。  

在 Rust 中，不允许使用索引方式获取 `String` 字符的原因索引操作预期总是需要常数时间 (O(1))。但是对于 `String` 不可能保证这样的性能，因为 Rust 不得不检查从字符串的开头到索引位置的内容来确定这里有多少有效的字符。   



**字符串 slice 必须包含完整的 UTF 字符。如果从字符中间截断，会出现 panic 崩溃**  



**字节、标量值、字形簇**  

比如梵文书写的 नमस्ते   

在 `Vec<u8>` 中字节为：  

```rust
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164, 224, 165, 135]
```

从 Unicode 标量值的角度理解，就像  Rust 中的 `char` 那样为：  

```rust
['न', 'म', 'स', '्', 'त', 'े']
```

这儿有 6 个 `char`。不过第四个和第六个都不是字母，是发音符号本身没有意义，如果以字形簇的角度理解，就会得到：  

```rust
["न", "म", "स्", "ते"]
```



**遍历字符串**

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}

// न
// म
// स
// ्
// त
// े
```

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}

// 224
// 164
// 168
// 224
// --skip--
// 165
// 135
```

**字符串并不简单**



## 3. HashMap

`HashMap<K, V>`

```rust
use std::collections::HashMap;
```

HashMap 是同质的，所有的键必须是相同类型，值也必须都是相同类型。   

**新建**  

```rust
// empty HashMap
let mut scoresMap = HashMap::new();

// 使用 Vec 来构造
let colors = vec![String::from("Blue"), String::from("Yellow")];
let scores = vec![10, 50];

let scoresMap: HashMap<_, _> = colors.iter().zip(scores.iter()).collect();
// iterator::zip() 把迭代器两两绑定
// iterator::collect() 返回一个集合，需要显示说明类型(2 种方法)
```



**插入**  

```rust
scoresMap.insert(String::from("Blue"), 10);
scoresMap.insert(String::from("Yellow"), 50);
```



**访问**  

```rust
let color = String::from("Blue");
let score = scoresMap.get(&color);

// score 对应的是 Some(10)，因为 get 返回 Option<V>
```



**遍历**  

```rust
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// Yellow: 50
// Blue: 10
```



**更新 HashMap**  

```rust
let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
// 打印出 {"Blue": 25}，原先的 10 被覆盖
```



检查某个特定的键是否有值，如果没有就插入一个值。 

```rust
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);

// entry 返回一个枚举 Entry，代表可能存在或者不存在的键。
// Entry 的 or_insert 方法在键对应的值存在时就返回这个值的 Entry
// 如果不存在则将参数作为新值插入并返回修改过的 Entry。
```



**根据旧值更新一个值**

```rust
let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
// {"world": 2, "hello": 1, "wonderful": 1}
```

`or_insert` 方法返回这个值的可变引用(`&mut V`)。`*` 解引用 `count`。



**所有权**  

该 copy 的 copy，该 move 的 move。   



**哈希函数**  

默认使用一种密码学安全的哈希函数，为了安全性牺牲了性能，也可以自己自定义 Hasher。  



# 八、错误处理

Rust 将错误组合成两个主要类别：**可恢复错误**（*recoverable*）和 **不可恢复错误**（*unrecoverable*）。  

对于可恢复错误有 `Result<T, E>` 值，以及 `panic!`，它在遇到不可恢复错误时停止程序执行。  

## 1. `panic!`

当出现 `panic!` 时，程序默认会开始 **展开**（*unwinding*），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。另一种选择是直接 **终止**（*abort*），这会不清理数据就退出程序。那么程序所使用的内存需要由操作系统来清理。如果你需要项目的最终二进制文件越小越好，可以由 panic 时展开切换为终止，通过在 *Cargo.toml* 的 `[profile]` 部分增加 `panic = 'abort'`。例如，如果你想要在发布模式中 panic 时直接终止：

```toml
[profile.release]
panic = 'abort'
```



**调用**

```rust
fn main() {
    panic!("crash and burn");
}
```



**使用 `panic!` 的 backtrace**  

当设置 `RUST_BACKTRACE = 1` 环境变量时，`panic!` 会显示 backtrace 信息。



## 2. `Result`

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` 代表成功时返回的 `Ok` 成员中的数据的类型，而 `E` 代表失败时返回的 `Err` 成员中的错误的类型。 



一个错误处理的例子

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("can't open the file: {:?}", error);
        },
    };
}
```



**匹配不同的错误**

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");
    let f = match f {
        Ok(file) => file,
        Error(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!("create the file error: {:?}", e)
                },
            }
        },
        Error(error) => {
            panic!("can't open the file: {:?}", error)
        }
    };
}
```

`match` 中的条件 `if ...` 被称为 *match guard* 。进一步完善 `match` 分支模式的额外条件，这个条件为真才能使分支的代码被执行。  

在模式的上下文中，`&` 匹配一个引用并返回它的值，而 `ref` 匹配一个值并返回一个引用。   

`ref` 是为了使用引用，而不是移动到 *guard* 条件中。之后会详细讲为什么使用 `ref` 而不是 `&`  



但是这样写错误处理显得冗长和复杂。所以提供了一些简写方式。



**`panic!` 的简写 `unwrap` 和 `expect`**

以下代码省略 `use` 语句

```rust
fn main() {
    let f = File::open("hello.txt").unwrap();
}

// thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
// repr: Os { code: 2, message: "No such file or directory" } }',
// src/libcore/result.rs:906:4
```



```rust
fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}

// thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
// 2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```



**传播错误**

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),    // open file error
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),           // read file error
    }
}
```

错误传播的简写方式 `?`

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;  // ? 简写 
    let mut s = String::new();
    f.read_to_string(&mut s)?;             // ? 简写
    Ok(s)
}
```

`?` 所使用的错误值被传递给了 `from` 函数，它定义于标准库的 `From` trait 中，其用来将错误从一种类型转换为另一种类型。到问号运算符调用 `from` 函数时，收到的错误类型被转换为定义为当前函数返回的错误类型。   

这在当一个函数返回一个错误类型来代表所有可能失败的方式时很有用，即使其可能会因很多种原因失败。只要每一个错误类型都实现了 `from` 函数来定义如将其转换为返回的错误类型，问号运算符会自动处理这些转换。  

`?` 消除了大量的样板代码使函数变得简单  

```rust
fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();
    File::open("hello.txt")?.read_to_string(&mut s)?;
    Ok(s)
}
```

只能在返回 `Result` 的函数中使用问号运算符 `?`。



## 3. `panic!` 还是 `Result`

选择返回 `Result` 值的话，就将选择权交给了调用者，而不是代替他们做出决定。调用者可能会选择以符合他们场景的方式尝试恢复，或者也可能干脆就认为 `Err` 是不可恢复的，所以他们也可能会调用 `panic!` 并将可恢复的错误变成了不可恢复的错误。因此返回 `Result` 是定义可能会失败的函数的一个好的默认选择。   

有一些情况 panic 比返回 `Result` 更为合适，不过他们并不常见。 示例、代码原型和测试都非常适合 panic!  



**错误处理的原则**

在当有可能会导致有害状态的情况下建议使用 `panic!`，在这里，有害状态是指当一些假设、保证、协议或不可变性被打破的状态，例如无效的值、自相矛盾的值或者被传递了不存在的值 —— 外加如下几种情况： 

- 有害状态并不包含 **预期** 会偶尔发生的错误
- 之后的代码的运行依赖于处于这种有害状态
- 当没有可行的手段来将有害状态信息编码进所使用的类型中的情况

函数通常都遵循 **契约**（*contracts*）：他们的行为只有在输入满足特定条件时才能得到保证。当违反契约时 panic 是有道理的，因为这通常代表调用方的 bug，而且这也不是那种你希望调用方必须处理的错误。 



一个猜数字处理错误的例子

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {    // construct
            value
        }
    }

    pub fn value(&self) -> u32 {  // getter
        self.value
    }
}
```



# 九、泛型、trait 和生命周期

## 1. 泛型

**函数定义中使用泛型**

```rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
}
```

出现报错

```txt
binary operation `>` cannot be applied to type `T`
`T` might need a bound for `std::cmp::PartialOrd`
```



**结构体中的泛型**

```rust
struct Point<T> {
    x: T,
    y: T,
}

// 方法和结构体相同的泛型
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}

// 和 c++ 一样，编译器自动推断类型。在 rust 中，字段 `x` 和 `y` 必须是相同类型的
// 如果想要不一样的类型，可以：
struct Point<T, U> {
    x: T,
    y: U, 
}

// 方法和结构体不同的泛型
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x
            y: other.y
        }
    }
}
```



**枚举中的泛型**

```rust
enum Option<T> {
    Some(T),
    None.
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```



Rust 中的泛型和 c++ 模板一样是编译器进行的。



## 2. trait

trait 允许我们进行另一种抽象：他们让我们可以抽象类型所通用的行为。*trait* 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。在使用泛型类型参数的场景中，可以使用 *trait bounds* 在编译时指定泛型可以是任何实现了某个 trait 的类型，并由此在这个场景下拥有我们希望的功能。  

类似于一些语言中接口(interface) 的功能。  

**定义 trait**  

我们想要创建一个多媒体聚合库用来显示可能储存在 `NewsArticle` 或 `Tweet` 实例中的数据的总结。 

```rust
// lib.rs
pub trait Summarizable {
    fn summary(&self) -> String;
}
```



**为类型实现 trait**

```rust
// lib.rs
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```



**实现不同作用域的 trait**

```rust
extern crate namespaceName;

use namespaceName::traitName;
```



**使用** 

```rust
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

trait 实现的一个需要注意的**限制**是：只能在 trait 或对应类型位于我们 crate 本地的时候为其实现 trait。换句话说，不允许对外部类型实现外部 trait。  

例如，不能在 `Vec` 上实现 `Display` trait，因为 `Display` 和 `Vec` 都定义于标准库中。允许在像 `Tweet` 这样作为我们 `aggregator`crate 部分功能的自定义类型上实现标准库中的 trait `Display`。也允许在 `aggregator`crate 中为 `Vec` 实现 `Summarizable`，因为 `Summarizable` 定义于此。   

个限制是我们称为 **孤儿规则**（*orphan rule*）的一部分，它被称为 orphan rule 是因为其父类型不存在。

没有这条规则的话，两个 crate 可以分别对相同类型实现相同的 trait，因而这两个实现会相互冲突。  



**默认实现**

```rust
pub trait Summarizable {
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}

impl Summarizable for NewsArticle {}  
```



**特殊的默认实现**

```rust
trait Summerizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("Read More from, {}", self.author_summary())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
}

impl Summerizable for Tweet {
    fn author_summary(&self) -> String {
        format!("@{}", self.username)
    }
}
```

**trait bounds**  

我们可以限制泛型不再适用于任何类型，编译器会确保其被限制为那些实现了特定 trait 的类型，由此泛型就会拥有我们希望其类型所拥有的功能。这被称为指定泛型的 *trait bounds*。   

```rust
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

之后我们可以在 `T` 上使用 trait bounds 来指定 `item` 必须是实现了 `Summarizable` trait 的类型

如果有多个 trait 可以使用 `+` 来并列 trait  

```rust
fn foo<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
    // ...
}

fn foo<T, U>(t: T, u: U) -> i32 
    where T: Display + Clone,
          U: Clone + Debug 
{
    
}
```



**使用 trait bounds 来修复 `largest` 函数**  

原先的报错信息  

```txt
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

使用 `>` 来比较两个 `T` 类型的值，这个运算符被定义为标准库中 trait `std::cmp::PartialOrd` 的一个默认方法。所以为了能够使用大于运算符，需要在 `T` 的 trait bounds 中指定 `PartialOrd`，这样 `largest` 函数可以用于任何可以比较大小的类型的 slice。因为 `PartialOrd` 位于 prelude 中所以并不需要手动将其引入作用域。

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    // ...
}
```

之后编译，仍然出现错误

```txt
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src\main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       help: consider using a reference instead: `&list[0]`

error[E0507]: cannot move out of borrowed content
 --> src\main.rs:3:9
  |
3 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

改为泛型之后，`T` 的类型可能为没有实现 `Copy` trait 的，意味着不能将 `list[0]` 的值移动到 `largest` 变量中。

所以需要 `Copy` trait。完整代码：

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];
    let result = largest(&number_list);
    println!("{}", result);
}
```



**其他的一些例子**

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}

fn main() {
    let p = Pair::new(0, 42);
    p.cmp_display();
}
```



```rust
impl<T: Display> ToString for T {
    // ...
}

let s = 3.to_string();
```



还有一种泛型，一直在使用，但是没有察觉到，就是**生命周期**



## 3. 生命周期和引用有效性

编译器的有一部分叫做 **借用检查器**（*borrow checker*），它比较作用域来确保所有的借用都是有效的。 

如果编写一个函数，返回两个 `&str` 中较长的 `&str`  

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

但是并不能编译。报错：

```txt
error[E0106]: missing lifetime specifier
   |
1  | fn longest(x: &str, y: &str) -> &str {
   |                                 ^ expected lifetime parameter
   |
   = help: this function's return type contains a borrowed value, but the
   signature does not say whether it is borrowed from `x` or `y`
```

返回是借用值，而且不知道是从 `x` 还是 `y` 借用，也不知道传入引用的生命周期。需要增加泛型生命周期参数来定义引用间的关系，以便借用检查器可以进行分析。



**生命周期注解语法**

生命周期注解并不改变任何引用的生命周期的长短。与当函数签名中指定了泛型类型参数后就可以接受任何类型一样，当指定了泛型生命周期后函数也能接受任何生命周期的引用。生命周期注解所做的就是将多个引用的生命周期联系起来。  

 生命周期参数以 `'` 开头通常为小写，类似于泛型类型，其名称非常短。`'a` 是大多数人默认使用的名称，位于 `&` 之后，并有空格将引用类型与生命周期注解分隔开。   

```rust
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

单个的生命周期注解本身没有多少意义：生命周期注解告诉 Rust 多个引用的泛型生命周期参数如何相互联系。如果函数有一个生命周期 `'a` 的 `i32` 的引用的参数 `first`，还有另一个同样是生命周期 `'a` 的 `i32`的引用的参数 `second`，这两个生命周期注解有相同的名称意味着 `first` 和 `second` 必须与这相同的泛型生命周期存在得一样久。  



**函数中的生命周期注解**

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

通过在函数签名中指定生命周期参数，我们并没有改变任何传入后返回的值的生命周期，而是指出任何不遵守这个协议的传入值都将被借用检查器拒绝。这个函数并不知道（或需要知道）`x` 和 `y` 具体会存在多久，而只需要知道有某个可以被 `'a` 替代的作用域将会满足这个签名。   

如果 `x` 和 `y` 的生命周期不遵守协议的话，会发生编译错误：  

```rust
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```



**深入理解生命周期**  

指定生命周期参数的正确方式依赖函数具体的功能。 下面的代码只返回 `x`，不需要为参数 `y` 指定一个生命周期。如下代码将能够编译：  

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

我们为参数 `x` 和返回值指定了生命周期参数 `'a`，不过没有为参数 `y` 指定，因为 `y` 的生命周期与参数 `x` 和返回值的生命周期没有任何关系。  

当从函数返回一个引用，返回值的生命周期参数需要与一个参数的生命周期参数相匹配。如果返回的引用 **没有** 指向任何一个参数，那么唯一的可能就是它指向一个函数内部创建的值，它将会是一个悬垂引用，导致编译错误。 

最好的解决方案是返回一个有所有权的数据类型而不是一个引用，这样函数调用者就需要负责清理这个值了。  



**结构体定义中的生命周期注解**

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

类似于泛型参数类型，必须在结构体名称后面的尖括号中声明泛型生命周期参数，以便在结构体定义中使用生命周期参数。  



**生命周期省略**  

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

这个函数没有生命周期注解，即便参数和返回值都是引用，但是仍然能够成功编译。 

本来的函数应该是 

```rust
fn first_word<'a>(s: &'a str) -> &'a str {
    // ...
}
```

被编码进 Rust 引用分析的模式被称为 **生命周期省略规则**（*lifetime elision rules*）。这并不是需要程序员遵守的规则；这些规则是一系列特定的场景，此时编译器会考虑，如果代码符合这些场景，就无需明确指定生命周期。  

如果 Rust 在明确遵守这些规则的前提下变量的生命周期仍然是模棱两可的话，它不会猜测剩余引用的生命周期应该是什么。在这种情况，编译器会给出一个错误，这可以通过增加对应引用之间相联系的生命周期注解来解决。  

函数或方法的参数的生命周期被称为 **输入生命周期**（*input lifetimes*），而返回值的生命周期被称为 **输出生命周期**（*output lifetimes*）。 

**编译器用于判断引用何时不需要明确生命周期注解的规则：**

- 每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：`fn foo<'a>(x: &'a i32)`，有两个引用参数的函数有两个不同的生命周期参数，`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`，依此类推。 
- 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。 
- 如果方法有多个输入生命周期参数，不过其中之一因为方法的缘故为 `&self` 或 `&mut self`，那么 `self` 的生命周期被赋给所有输出生命周期参数。这使得方法编写起来更简洁。 



上面规则的一些例子：

第一条规则变为：

```rust
fn first_word(s: &str) -> &str {
    // ...
}
```

```rust
// 每个引用参数都有其自己的生命周期, 现在签名看起来像这样:
fn first_word<'a>(s: &'a str) -> &str {
    // ...
}
```

第二条规则变为：

```rust
// 输入参数的生命周期将被赋予输出生命周期参数，所以现在签名看起来像这样：
fn first_word<'a>(s: &'a str) -> &'a str {
    // ...
}
```



第三条规则有点复杂，现在从另一个例子开始

```rust
fn longest(x: &str, y: &str) -> &str {
```

应用第一条规则之后

```rust
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

再来使用第二条规则，发现并不只有一个输入生命周期参数。

再来看第三条。也并没有 `self`，然后也没有更多的规则了，所以会产生一个编译错误。



接下来看一个适合第三条规则的例子：

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```



**静态生命周期**

```rust
let s: &'static str = "I have a static lifetime"
```

不过将引用指定为 `'static` 之前，思考一下这个引用是否真的在整个程序的生命周期里都有效（或者哪怕你希望它一直有效，如果可能的话）。大部分情况，代码中的问题是尝试创建一个悬垂引用或者可用的生命周期不匹配，请解决这些问题而不是指定一个 `'static` 的生命周期。 



**综合以上**

在同一函数中指定泛型类型参数、trait bounds 和生命周期的语法！ 

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```



# 十、测试

## 1. 编写测试

Rust 提供了测试功能：`test` 属性、一些宏和 `should_panic` 属性

为了将一个函数变成测试函数，需要在 `fn` 行之前加上 `#[test]`。当使用 `cargo test` 命令运行测试时，Rust 会构建一个测试执行程序用来调用标记了 `test` 属性的函数，并报告每一个测试是通过还是失败。 

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");  // test failed
    }
}
```



**使用 `assert!` 宏来检查结果**  

**使用 `assert_eq!` 和 `assert_ne!` 宏来测试相等**  

`assert_eq!` 和 `assert_ne!` 宏在底层分别使用了 `==` 和 `!=`。当断言失败时，这些宏会使用调试格式打印出其参数，这意味着被比较的值必需实现了 `PartialEq` 和 `Debug` trait。   

对于自定义的结构体和枚举，需要实现 `PartialEq` 才能断言他们的值是否相等。需要实现 `Debug` 才能在断言失败时打印他们的值。因为这两个 trait 都是派生 trait，通常可以直接在结构体或枚举上添加 `#[derive(PartialEq, Debug)]` 注解。   



**自定义失败信息**

```rust
assert!(condition, fmt, ...);
```



**使用 `should_panic!` 来检查 `panic`**

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

 `should_panic` 可以增加 `expected` 额外错误信息。

```rust
    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
```



## 2. 运行测试

`cargo test` 可以对测试案例进行测试。可以指定命令行参数来改变 `cargo test` 的默认行为。   

例如，`cargo test` 生成的二进制文件的默认行为是并行的运行所有测试，并截获测试运行过程中产生的输出，阻止他们被显示出来，使得阅读测试结果相关的内容变得更容易。   

首先国际惯例：`cargo test -- help` 进行查看帮助  

**并行或串行的运行测试**  

当运行多个测试时，他们默认使用线程来并行的运行。这意味着测试会更快的运行完毕，所以你可以更快的得到代码能否工作的反馈。   

因为测试是在同时运行的，你应该确保测试不能相互依赖，或依赖任何共享的状态，包括依赖共享的环境，比如当前工作目录或者环境变量。   

如果你不希望测试并行运行，或者想要更加精确的控制线程的数量，可以传递 `--test-threads` 参数和希望使用线程的数量给测试二进制文件。例如：  

```sh
cargo test -- --test-threads=1
```

**显示函数输出**  

默认情况下，当测试通过时，Rust 的测试库会截获打印到标准输出的所有内容。 如果想禁止标准输出 

```sh
cargo test -- --nocapture
```



**通过名字来运行测试的子集**

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```



- 运行单个测试。比如 `cargo test one_hundred `

- 过滤运行多个测试。比如 `cargo test add` 会测试名称中带有 `add` 的测试案例

- 忽略某些测试 。比如某些测试时非常耗时的，不能每次都进行测试：

  ```rust
  #[test]
  #[ignore]
  fn expensive_test() {
      // code that takes an hour to run
  }
  ```

  执行 `cargo test` 会自动过滤有 `ignore` 标签的测试案例

  如果希望只执行 `ignore` 标签的测试案例，需要 `cargo test -- --ignored `



## 3. 测试的组织结构

**单元测试**  

测试模块的 `#[cfg(test)]` 注解告诉 Rust 只在执行 `cargo test` 时才编译和运行测试代码，而在运行 `cargo build` 时不这么做。这在只希望构建库的时候可以节省编译时间，并且因为它们并没有包含测试，所以能减少编译产生的文件的大小。   

与之对应的集成测试因为位于另一个文件夹，所以它们并不需要 `#[cfg(test)]` 注解。然而单元测试位于与源码相同的文件中，所以你需要使用 `#[cfg(test)]` 来指定他们不应该被包含进编译结果中。   

 `cfg` 是 *configuration*，通过 `cfg` 标签，Cargo 只会在我们主动使用 `cargo test` 运行测试时才编译测试代码。   



```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```



**集成测试**  

集成测试对于你需要测试的库来说完全是外部的。 为了编写集成测试，需要在项目根目录创建一个 *tests* 目录，与 *src* 同级。   

```rust
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

使用 `cargo test` 进行测试。测试一共有三部分单元测试、集成测试和文档测试。



**集成测试的子模块**

```txt
├─src ...
└─tests
   ├─common
   │   └─mod.rs
   ├─integration_test.rs
```



```rust
// tests/common/mod.rs
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```



```rust
// tests/integration_test.rs
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```



**二进制 crate 集成测试**  

如果项目是二进制 crate 并且只包含 *src/main.rs* 而没有 *src/lib.rs*，这样就不可能在 *tests* 目录创建集成测试并使用 `extern crate` 导入 *src/main.rs* 中定义的函数。   

只有库 crate 才会向其他 crate 暴露了可供调用和使用的函数；二进制 crate 只意在单独运行。   

为什么 Rust 二进制项目的结构明确采用 *src/main.rs* 调用 *src/lib.rs* 中的逻辑的方式？因为通过这种结构，集成测试 **就可以** 通过 `extern crate` 测试库 crate 中的主要功能了，而如果这些重要的功能没有问题的话，*src/main.rs* 中的少量代码也就会正常工作且不需要测试。 



# 十一、I/O 项目

编写一个简单的 grep 程序

```sh
grep pattern filename
```



## 1. 接受命令行参数

**读取参数值**

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

> `std::env::args` 在其任何参数包含无效 Unicode 字符时会 panic。如果你需要接受包含无效 Unicode 字符的参数，使用 `std::env::args_os` 代替。这个函数返回 `OsString` 值而不是 `String`值。 

`collect` 来创建了一个包含迭代器所有值的集合。

 

**把参数保存再变量**

```rust
use std::env;

fn main() {
    let args = env::args().collect();

    let pattern = &args[1];
    let filename = &args[2];
}
```

## 2. 读取文件

```rust
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
    let args: Vec<String> = env::args().collect();

    let pattern = &args[1];
    let filename = &args[2];

    let mut f = File::open(filename).expect("file not found");
    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("error while reading the file");
    println!("{}", contents);
}
```

需要注意 Windows 环境下，命令行和文件必须改为 UTF-8 编码才可以  

在中文 Windows 中，cmd 和 powershell 默认编码为 UTF-8，由于这儿命令行参数全部是英文字母，GBK 和 UTF-8 表示相同，所以不用改变也可以有效。  

还有需要改变文件编码为 UTF-8。



## 3. 重构改进模块性和错误处理

**二进制项目关注分离**

- 将程序拆分成 *main.rs* 和 *lib.rs* 并将程序的逻辑放入 *lib.rs* 中 
- 当程序逻辑比较小时，可以保留在 *main.rs* 中。



**提取参数解析器**

```rust
fn parse_config(args: &[String]) -> (&str, &str) {
    let pattern = &args[1];
    let filename = &args[2];
    (pattern, filename)
}
```



**组合配置值**

```rust
struct Config {
    pattern: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let pattern = args[1].clone();
    let filename = args[2].clone();
    Config {pattern, filename}
}
```

**合并为构造函数**

```rust
struct Config {
    pattern: String,
    filename: String,
}

impl Config {
    fn new(args: &[String]) -> Config {
        let pattern = args[1].clone();
        let filename = args[2].clone();
        Config {pattern, filename}
    }
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let config = Config::new(&args);
    // ...
}
```

**修复错误处理**

```rust
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let pattern = args[1].clone();
        let filename = args[2].clone();
        Ok(Config {pattern, filename})
    }
}
```

```rust
fn main() {
    let args: Vec<String> = env::args().collect();
    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });
    // ...
}

// unwrap_or_else 可以进行一些自定义的非 panic! 的错误处理。
// 当 Result 是 Ok 时，这个方法的行为类似于 unwrap：它返回 Ok 内部封装的值
// 当其值是 Err 时，该方法会调用一个闭包（closure），也就是匿名函数。类似于 lambda
```



**主要逻辑模块提取**

```rust
use std::error::Error;

fn grep(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;
    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    Ok(())
}

// 使用了 trait 对象 Box<Error>。
// 目前只需知道 Box<Error> 意味着函数会返回实现了 Error trait 的类型
// 不过无需指定具体将会返回的值的类型。
```



**处理 main 中 grep 的错误**

```rust
if let Err(e) = grep(config) {
        println!("Application error: {}", e);
        process::exit(1);
}
```



**将代码拆分到 crate 中**

```rust
// lib.rs
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    // ...
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        // ...
    }
}

pub fn grep(config: Config) -> Result<(), Box<Error>> {
    // ...
}
```



## 4. 采用测试驱动开发完善库的功能

**编写测试案例** 

```rust
// lib.rs
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```



**实现逻辑功能**

```rust
// lib.rs
pub fn search<'a>(pattern: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();
    
    for line in contents.lines() {
        if line.contains(pattern) {
            results.push(line);
        }
    }
    
    results
}

// 匹配的 contents 需要存放到 Vec 中，还有两个输入参数，所以输出需要生命周期
```

之后使用 `cargo test` 

```txt
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```



使用 `cargo run pattern filename` 就可以得到输出了



## 5. 处理环境变量

```rust
use std::env;

let var = env::var("ENV_NAME").is_err();
let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

// 如果 CASE_INSENSITIVE 环境变量被设置为任何值，is_err 会返回 false 并将进行大小写不敏感搜索。
```



## 6. 标准错误

```rust
eprintln!(fmt, ...);
```



# 十二、函数式功能

## 1. 闭包

Rust 的 **闭包**（*closures*）是可以保存进变量或作为参数传递给其他函数的匿名函数。可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。 不同于函数，闭包允许捕获调用者作用域中的值。  



我们在一个通过 app 生成自定义健身计划的初创企业工作。 其后端使用 Rust 编写， 而生成健身计划的算法需要考虑很多不同的因素， 这个计算只花费几秒钟。   

所有输入有 一个用户的 intensity 数字，一个随机数

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(
        simulated_user_specified_value,
        simulated_random_number
    );
}

fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_result = simulated_expensive_calculation(intensity);
    if intensity < 25 {
        println!("Today, do {} pushups!", expensive_result);
        println!("Next, do {} situps!", expensive_result);
    } 
    else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } 
        else {
            println!("Today, run for {} minutes!", expensive_result);
        }
    }
}

use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: u32) -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```



**使用闭包来重构上述代码**

```rust
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

`||` 中间的是闭包的参数，多个参数使用  `, ` 隔开。  

`{}` 是闭包体，如果只有一行大括号可以省略。  



**闭包的类型推断和注解**

闭包通常很短并只与对应相对任意的场景较小的上下文中。在这些有限制的上下文中，编译器能可靠的推断参数和返回值的类型，类似于它是如何能够推断大部分变量的类型一样。 强制在这些小的匿名函数中注明类型是很恼人的，并且与编译器已知的信息存在大量的重复。 

```	rust
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

```rust
// 一些基本等价的写法
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```



**一个错误的例子，说明闭包的类型推断**

```rust
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
                        ^ expected struct `std::string::String`, found
```

```c++
// 类似的 c++14 代码，可以在参数自动推断不同类型
auto lambda = [](const auto& x) { cout << x << endl; };
lambda(42);
lambda("hello");
```



**使用带有泛型和 `Fn` trait 的闭包**  

回到刚刚的 app，由于第一个分支需要调用两次 闭包，造成了不必要的时间损失  

一种方法可以将结果保存进变量以供复用，这样就可以使用变量而不是再次调用闭包。但是这样就会有很多重复的保存结果变量的地方。 (个人这个挺好的，浪费一点点空间，换取实现上的简洁，而且考虑到 cache)  

另一个可用的方案。可以创建一个存放闭包和调用闭包结果的结构体。该结构体只会在需要结果时执行闭包，并会缓存结果值，这样余下的代码就不必再负责保存结果并可以复用该值。你可能见过这种模式被称 *memoization* 或 *lazy evaluation*。   

`Fn` 系列 trait 由标准库提供。所有的闭包都实现了 trait `Fn`、`FnMut` 或 `FnOnce` 中的一个。   

```rust
struct Cacher<T> 
    where T: Fn(u32) -> u32 
{
    calculation: T,
    value: Option<u32>,
}

impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }
    
    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

```rust
let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
});
```

上面实现的只要第一次计算出值之后，之后一直会缓存该值，使用其他的值进行计算将会发生错误

可以使用 HashMap 来解决缓存的问题。  



**闭包会捕获其环境**  

当闭包从环境中捕获一个值，闭包会在闭包体中储存这个值以供使用。这会使用内存并产生额外的开销，当执行不会捕获环境的更通用的代码场景中我们不希望有这些开销。因为函数从未允许捕获环境，定义和使用函数也就从不会有这些额外开销。   

闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，不可变借用和可变借用。这三种捕获值的方式被编码为如下三个 `Fn` trait： 

- `FnOnce` 消费从周围作用域捕获的变量，闭包周围的作用域被称为其 **环境**，*environment*。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 `Once` 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
- `Fn` 从其环境不可变的借用值
- `FnMut` 可变的借用值所以可以改变其环境

```rust
let equal_to_x = move |z| z == x;
```

## 2. 迭代器

```rust
let v = vec![1, 2, 3];
let iter = v.iter();

for val in iter {
    // ... use val
}
```



有三种迭代器

- `iter(), &T`
- `iter_mut(), &mut T`
- `into_iter(), T`



**迭代器 trait**

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

`type Item` 和 `Self::Item` 是关联类型（*associated type*）。 `Item`类型将是迭代器返回元素的类型。 

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

需要注意的是 `v1_iter` 是 `mut` 的，在迭代器上调用 `next` 方法改变迭代器记录位置的状态。  

因为 `Vec` 没有 `iter` 方法。来自 `Deref` 到 `[T]` 。`[T]` 有 `iter` 方法。返回 `std::slice::Iter<T>`

`next` 返回的类型是 `Option<&'a i32>`，`Some(&1)`  是 `Option<&i32>`。如果使用 `Some(1)` 会发生错误。  

当然也可以使用获取所有权的迭代器 `into_iter()`  



**消费迭代器方法**

一些方法在其定义中调用了 `next` 方法，这些调用 `next` 方法的方法被称为 **消费适配器**（*consuming adaptors*），因为调用他们会消耗迭代器。比如 `sum` 方法

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];
    let v1_iter = v1.iter();
    let total: i32 = v1_iter.sum();
    assert_eq!(total, 6);
}
```



**产生其他迭代器的方法**

 `Iterator` trait 中定义了另一类方法，被称为 **迭代器适配器**（*iterator adaptors*），他们允许我们将当前迭代器变为不同类型的迭代器。   

```rust
let v1: Vec<i32> = vec![1, 2, 3];
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
assert_eq!(v2, vec![2, 3, 4]);
```

调用 `map` 方法重新映射为新迭代器，接着调用 `collect` 方法消费新迭代器并创建一个 `vector`



**使用闭包获取环境**

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}
```

`shoes_in_my_size` 获取 `shoes` 和所有权和一个鞋子大小作为参数，移动一个 `Vec<Shoe>`。

`into_iter()` 获取 `shoes` 所有权的迭代器，过滤器只保留指定大小的鞋子。



**自定义迭代器**

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        
        if self.count < 6 {
            Some(self.count)
        }
        else {
            None
        }
    }
}
```



**自定义迭代器的其他方法**

```rust
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```



## 12.3. 改进之前的 I/O 项目

```rust
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```





**使用迭代器使代码更简洁**

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```



## 12.4. 性能比较

迭代器是 Rust 的 **零成本抽象**（*zero-cost abstractions*）之一，它意味着抽象并不会强加运行时开销。和 C++ 相似。

**一个综合使用的例子**

```rust
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```



# 十三、cargo 和 crates.io

## 1. 自定义构建

主要有两个配置：`cargo build` 所采用的 `dev` 配置和 `cargo build --release` 的 `release` 配置。

```toml
# Cargo.toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

## 2. 将 crate 发布到 crates.io

[crates.io](https://crates.io/) 用来分发包的源代码，所以它主要托管开源代码。 



**编写文档注释**

```rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

![doc](https://kaisery.github.io/trpl-zh-cn/img/trpl14-01.png)



文档注释中常用标题

- Examples 
- Panics: 可能会 `panic!` 的场景
- Errors
- Safety: 使用 `unsafe` 的代码应该标注



**文档测试**

之前使用 `cargo test` 时，忽略了文档测试案例。在文档中的示例代码，可以在测试时进行测试。



**包含注释的项**

```rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

![doc](https://kaisery.github.io/trpl-zh-cn/img/trpl14-02.png)

`My Crate` 下面有一系列的说明注释



**使用 `pub use` 来导出合适的公有 API**

```rust
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

![doc](https://kaisery.github.io/trpl-zh-cn/img/trpl14-04.png)



关于[发布 crates.io](https://kaisery.github.io/trpl-zh-cn/img/trpl14-04.png) 暂时略过



## 3. Cargo 工作空间

https://kaisery.github.io/trpl-zh-cn/ch14-03-cargo-workspaces.html



之后关于 Cargo 的内容暂略



# 十四、智能指针

## 1. Box\<T\>

类似于 `std::unique_ptr<T>`

```rust
fn main() {
    let b = Box::new(5);   // Box<i32>
    println!("b = {}", b);
}
```



`Box<T>` 允许递归类型，比如链表，树等等数据结构

以 `cons list` 为例，一个函数式编程语言中的常见类型。cons 是 construct function 的缩写，递归终止条件是 `Nil`，表示无效或者缺失的值

```rust
enum List {
    Cons(i32, List),
    Nil,
}

let list = Cons(1 Cons(2, Cons(3, Nil)));
```

但是编译上面的代码出现错误

```txt
 recursive type `List` has infinite size
 也就是无法计算 List 需要多少空间
```



**使用  `Box<T>` 给递归类型一个已知的大小**

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

现在 Cons 看起来像下面那样

![box](https://kaisery.github.io/trpl-zh-cn/img/trpl15-02.svg)

## 2. 通过 `Deref` trait 将智能指针当作常规引用

实现 `Deref` trait 允许重载 解引用运算符 `*`。

使用 `*` 解引用指针指向的值

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

 使用 Box

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```



**自定义 Box**

```rust
struct MyBox<T>(T);    // 包含一个元素的元组结构体

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

// *y ==> *(y.deref())
```



**隐式解引用强制多态**

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

`&m` 的类型是 `&MyBox<String>`，通过 `deref` 调用变为 `&String`。标准库中 `String` 上提供了 `deref` 调用返回 `slice`，符合函数签名了



**解引用强制多态和可变性交互**

Rust 在发现类型和 trait 实现满足三种情况时会进行解引用强制多态： 

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

最后一个情况有些微妙：Rust 也会将可变引用强转为不可变引用。但是反之是 **不可能** 的：不可变引用永远也不能强转为可变引用。 



## 3. `Drop` trait 运行清理代码

Rust 当值离开作用域时自动插入 `drop` 调用，不能直接禁用这个功能。   

也不允许显式调用 `drop`，错误信息 `explicit destructor calls not allowed `。析构函数，好亲切~  

如果需要在离开作用域之前显式清理资源，需要使用 `std::mem::drop`   

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c);   // c.drop() 将发生 explicit destructor
    println!("CustomSmartPointer dropped before the end of main.");
}
```



## 4. `Rc<T>`

`Rc<T>` 只能用于单线程场景 

如果要实现下图的数据结构，b, c 共享 a 的所有权

![cons](https://kaisery.github.io/trpl-zh-cn/img/trpl15-03.svg)

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));  // 也可以 a.clone()
    let c = Cons(4, Rc::clone(&a));
}
```

`Rc::clone` 会增加引用计数，而不是深拷贝



## 5. `RefCell<T>` 和内部可变性模式

**内部可变性**（*Interior mutability*）是 Rust 中的一个设计模式，它允许你即使在有不可变引用时改变数据，这通常是借用规则所不允许的。 为此，该模式在数据结构中使用 `unsafe` 代码来模糊 Rust 通常的可变性和借用规则。   

**通过 `RefCell<T>` 在运行时检查借用规则**  

不同于 `Rc<T>`，`RefCell<T>` 代表其数据的唯一的所有权。 和 `Box<T>` 的区别在在运行时检查借用规则。  

对于引用和 `Box<T>`，借用规则的不可变性作用于编译时。对于 `RefCell<T>`，这些不可变性作用于 **运行时**。 对于引用，如果违反这些规则，会得到一个编译错误。而对于`RefCell<T>`，违反这些规则会 `panic!`。   

如下为选择 `Box<T>`，`Rc<T>` 或 `RefCell<T>` 的理由：

- `Rc<T>` 允许相同数据有多个所有者；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时执行不可变（或可变）借用检查；`Rc<T>`仅允许在编译时执行不可变借用检查；`RefCell<T>` 允许在运行时执行不可变（或可变）借用检查。
- 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。



**内部可变性：不可变值的可变引用**

```rust
fn main() {
    let x = 5;
    let y = &mut x;
}

// 编译错误：cannot borrow mutably
```

**内部可变性用例：mock 对象**  

**测试替身**（*test double*）是一个通用编程概念，它代表一个在测试中替代某个类型的类型。**mock 对象** 是特定类型的测试替身，它们记录测试过程中发生了什么以便可以断言操作是正确的。 可以创建一个与 mock 对象有着相同功能的结构体。   

编写一个记录某个值与最大值的差距的库，并根据当前值与最大值的差距来发送消息。例如，这个库可以用于记录用户所允许的 API 调用数量限额。   

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: 'a + Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 0.75 && percentage_of_max < 0.9 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        } else if percentage_of_max >= 0.9 && percentage_of_max < 1.0 {
            self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        }
    }
}
```







# 十五、总结

本人对 C++ 比较熟悉，在调试 C++ 代码的时候可能过于劳心费神，被 Rust 的描述：“系统级编程语言，惊人的运行速度，内存安全和线程安全”所吸引，之后入门稍微学习了一下 Rust。  

Rust 相较于 C++ 保证各种安全问题，编译器自带的静态检测有效减少 bug，把 C++ 在运行时面临的一些难搞的问题提前到编译期，加入了函数式编程的 pattern match 还有一些语法糖，使得语法更加抽象简洁(但也有一些鬼画符语法)。包管理工具和构建系统 cargo 也是比较香的，不像 C++ 需要学习 vcproject，makefile，cmake 等等。  

不过 Rust 学习曲线比较陡峭，某些语法个人认为不太直观，现在的环境来看，Rust 的学习成本和开发效率不太合适。   

继续回去写 C++，还是可以期待一下 Rust 之后的发展~