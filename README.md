#RUST基础
## 基本构造
![rust](rust.jpg "Magic Gardens")
## cargo
```
$ cargo new hello_opensource
$ cd hello_opensource/
//安装依赖，生成exe
$ cargo build
//自动执行 rustc？
$ cargo run 
//检查代码是否可编译
$ cargo check
```
可以使用 cargo new 创建项目（构建了一个包）。
可以使用 cargo build 构建项目。
可以使用 cargo run 一步构建并运行项目。
可以使用 cargo check 在不生成二进制文件的情况下构建项目来检查错误。
有别于将构建结果放在与源码相同的目录，Cargo 会将其放到 target/debug 目录。

### Cargo.toml 
安装依赖
### Cargo.lock
cargo build后生成的实际依赖，不需要碰

#### 变量和可变性
1. 变量默认是不可变的
2. 记住，Rust 是 静态类型（statically typed）语言，也就是说在编译时就必须知道所有变量的类型

## 类型
### 字符串
#### String 和 &str 的区别
String是一个可变引用，而&str是对该字符串的不可变引用，即可以更改String的数据，但是不能操作&str的数据。String包含其数据的所有权，而&str没有所有权，它从另一个变量借用它。


## 复合类型
1. 元组 类型可不同，长度一旦生成不可变
```
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```
2. 数组 类型必须相同，长度固定
   但是数组并不如 vector 类型灵活。vector 类型是标准库提供的一个 允许 增长和缩小长度的类似数组的集合类型。当不确定是应该使用数组还是 vector 的时候，那么很可能应该使用 vector
   ```
   let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
   let a: [i32; 5] = [1, 2, 3, 4, 5];
   ```
## 函数
```
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```
## if
```
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}

```
### loop
loop前面可以加标签，用于break指定循环。loop后面没有条件语句.
```
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```
### for
```
fn first_word(s: &String) -> usize {
    // 返回数组
    let bytes = s.as_bytes();
    // enumerate返回包装过的元组(index,item)
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {}
```
### while
```
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}

```
### 认识所有权
栈：后进先出 必须占用已知且固定大小
堆：后进后出 大小未知或变化的数据
```
fn main() {
    let s1 = String::from("hello");
    let s2 = s1;
    //此时s1已经无效
    println!("{}, world!", s1);
}
```

函数传递时,堆类型无效，栈类型有效
```
fn main() {
    let s = String::from("hello");  // s 进入作用域

    takes_ownership(s);             // s 的值移动到函数里 ...
                                    // ... 所以到这里不再有效

    let x = 5;                      // x 进入作用域

    makes_copy(x);                  // x 应该移动函数里，
                                    // 但 i32 是 Copy 的，
                                    // 所以在后面可继续使用 x

} // 这里, x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
  // 没有特殊之处

fn takes_ownership(some_string: String) { // some_string 进入作用域
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 `drop` 方法。
  // 占用的内存被释放

fn makes_copy(some_integer: i32) { // some_integer 进入作用域
    println!("{}", some_integer);
} // 这里，some_integer 移出作用域。没有特殊之处
```
### 引用与借用
```
fn main() {
    let s1 = String::from("hello");
    //&即引用，不获取所有权
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}
//&即引用，不获取所有权
fn calculate_length(s: &String) -> usize {
    s.len()
}
//注意：与使用 & 引用相反的操作是 解引用（dereferencing），它使用解引用运算符，*
```

### 引用的规则
1. 在任意给定时间，要么 只能有一个可变引用，要么 只能有多个不可变引用。
2. 引用必须总是有效的。

### Slice 类型
slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。slice 是一类引用，所以它没有所有权
```
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

    s.clear(); // 错误!

    println!("the first word is: {}", word);
}

```
回忆一下借用规则，当拥有某值的不可变引用时，就不能再获取一个可变引用。因为 clear 需要清空 String，它尝试获取一个可变引用
```
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src\main.rs:30:5
   |
28 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
29 |
30 |     s.clear(); // 错误!
   |     ^^^^^^^^^ mutable borrow occurs here
31 |
32 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `variables` due to previous error
```
### 结构体
```
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}

fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    // 结构体必须是可变的才能赋值
    user1.email = String::from("anotheremail@example.com");
}
```
### 结构体数据的所有权
### 结构体示例程序
```
fn main() {
    let width1 = 30;
    let height1 = 50;
    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}
// 传两个变量不美观
fn area(width:u32, heigth:u32) -> u32 {
    width * heigth
}
```
```
fn main() {
    let rect1 = (30, 50);
    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}
// 使用元组，没有语义，不知道哪个是宽那个是高
fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```
```
struct Rectangle {
    width : u32,
    height: u32,
}

fn area(rectangle: &Rectangle) -> u32{
    // 访问对结构体的引用的字段不会移动字段的所有权
    rectangle.width * rectangle.height
}
fn main(){
    let rect1 = Rectangle{
        width:30,
        height:50,
    };
    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );

}
```
### 运行时输出结构体内容
```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
```
### 方法语法(struct中的方法)
```
struct Rectangle {
    width : u32,
    height: u32,
}
impl Rectangle{
    fn area(&self) -> u32{
        self.width * self.height
    }
}

fn main(){

    let rect1 = Rectangle{
        width:30,
        height:50,
    };
    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```
### 关联函数
```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}

fn main() {
    let sq = Rectangle::square(3);
}
```
### 多个 impl 块
## 枚举和模式匹配
结构体给予你将字段和数据聚合在一起的方法，像 Rectangle 结构体有 width 和 height 两个字段。而枚举给予你将一个值成为一个集合之一的方法。比如，我们想让 Rectangle 是一些形状的集合，包含 Circle 和 Triangle 。为了做到这个，Rust提供了枚举类型。
### Option 枚举和其相对于空值的优势
这一部分会分析一个 Option 的案例，Option 是标准库定义的另一个枚举。Option 类型应用广泛因为它编码了一个非常普遍的场景，即一个值要么有值要么没值。
Option<T> 枚举是如此有用以至于它甚至被包含在了 prelude 之中，你不需要将其显式引入作用域。另外，它的成员也是如此，可以不需要 Option:: 前缀来直接使用 Some 和 None。即便如此 Option<T> 也仍是常规的枚举，Some(T) 和 None 仍是 Option<T> 的成员。
### match 控制流结构
```
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```
```
fn main() {
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        // 占位符
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
}

```
## 使用包、Crate 和模块管理不断增长的项目
1. 包（Packages）： Cargo 的一个功能，它允许你构建、测试和分享 crate。
2. Crates ：一个模块的树形结构，它形成了库或二进制项目。
3. 模块（Modules）和 use： 允许你控制作用域和路径的私有性。
4. 路径（path）：一个命名例如结构体、函数或模块等项的方式

### 包和 Crate
1. crate 是 Rust 在编译时最小的代码单位
2. crate 有两种形式：二进制项（可编译成可执行文件）和库
3. 包（package） 是提供一系列功能的一个或者多个 crate。一个包会包含一个 Cargo.toml 文件。

### 包拆分前
```
// src/lib.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
### 包拆分后
```
// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```
```
// src/front_of_house.rs
pub mod hosting;
```
```
// src/lib.rs
mod front_of_house;
pub use crate::front_of_house::hosting;
pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```
## 常见集合
1. vector 允许我们一个挨着一个地储存一系列数量可变的值
2. 字符串（string）是字符的集合。我们之前见过 String 类型，不过在本章我们将深入了解。
3. 哈希 map（hash map）允许我们将值与一个特定的键（key）相关联。这是一个叫做 map 的更通用的数据结构的特定实现。
### 使用 Vector 储存列表
```
fn main() {
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
}
```
```
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {}", third);

    match v.get(2) {
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
}
```
### 使用枚举来储存多种类型
vector 只能储存相同类型的值。这是很不方便的；绝对会有需要储存一系列不同类型的值的用例。幸运的是，枚举的成员都被定义为相同的枚举类型，所以当需要在 vector 中储存不同类型值时，我们可以定义并使用一个枚举！
```
fn main() {
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}
```
### 使用字符串储存 UTF-8 编码的文本
```
fn main() {
    let mut s1 = String::from("张");
    s1.push_str("world");
    s1.push('!');

    for c in s1.chars(){
        println!("{c}");
    }
}
```
### 使用 Hash Map 储存键值对
```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```
```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
}
```
```
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```
## 错误处理
Rust 将错误分为两大类：可恢复的（recoverable）和 不可恢复的（unrecoverable）错误
Rust 没有异常。相反，它有 Result<T, E> 类型，用于处理可恢复的错误，还有 panic! 宏，在程序遇到不可恢复的错误时停止执行
### 用 panic! 处理不可恢复的错误
```
fn main() {
    panic!("crash and burn");
}
```
### 用 Result 处理可恢复的错误
unwrap_or_else，它定义于标准库的 Result<T, E> 上。使用 unwrap_or_else 可以进行一些自定义的非 panic! 的错误处理。当 Result 是 Ok 时，这个方法的行为类似于 unwrap：它返回 Ok 内部封装的值。然而，当其值是 Err 时，该方法会调用一个 闭包（closure），也就是一个我们定义的作为参数传递给 unwrap_or_else 的匿名函数
```
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```
```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```
```
#![allow(unused)]
fn main() {
    use std::fs::File;
    use std::io::{self, Read};

    let s = read_username_from_file().expect("file error");
    println!("{s}");

    fn read_username_from_file() -> Result<String, io::Error> {
        let f = File::open("Failed to open hello.txtFailed to open hello.txt");

        let mut f = match f {
            Ok(file) => file,
            Err(e) => return Err(e),
        };

        let mut s = String::new();

        match f.read_to_string(&mut s) {
            Ok(_) => Ok(s),
            Err(e) => Err(e),
        }
    }
}

```
#### 失败时 panic 的简写：unwrap 和 expect
match 能够胜任它的工作，不过它可能有点冗长并且不总是能很好的表明其意图。Result<T, E> 类型定义了很多辅助方法来处理各种情况。其中之一叫做 unwrap，它的实现就类似于示例 9-4 中的 match 语句。如果 Result 值是成员 Ok，unwrap 会返回 Ok 中的值。如果 Result 是成员 Err，unwrap 会为我们调用 panic!。这里是一个实践 unwrap 的例子：
```
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}

```
传播错误的简写：? 运算符
不同于遇到错误就 panic!，? 会从函数中返回错误值并让调用者来处理它。
```
#![allow(unused)]
fn main() {
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
}
```
### 要不要 panic!
错误处理指导原则
```
#![allow(unused)]
fn main() {
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
}

```
## 泛型、Trait 和生命周期
```
fn largest<T>(list: &[T]) -> T {
```
### 结构体定义中的泛型
```
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```
### 枚举定义中的泛型
```
enum Option<T> {
    Some(T),
    None,
}
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
### 方法定义中的泛型
```
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```
### Trait：定义共同行为
trait 告诉 Rust 编译器某个特定类型拥有可能与其他类型共享的功能。可以通过 trait 以一种抽象的方式定义共享的行为。可以使用 trait bounds 指定泛型是任何拥有特定行为的类型。
注意：trait 类似于其他语言中的常被称为 接口（interfaces）的功能，虽然有一些不同。
#### 定义 trait
```
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}

```
#### trait 作为参数
```
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```
#### Trait Bound 语法（语法糖）
```
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```
#### 通过 + 指定多个 trait bound
```
pub fn notify(item: &(impl Summary + Display)) {}
pub fn notify<T: Summary + Display>(item: &T) {}
```
#### 通过 where 简化 trait bound
```
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {}

// or
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}

```
#### 返回实现了 trait 的类型
```
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```
### 生命周期确保引用有效
Rust 中的每一个引用都有其生命周期（lifetime），也就是引用保持有效的作用域
函数返回的引用的生命周期应该与传入参数的生命周期中较短那个保持一致
#### 生命周期避免了悬垂引用
```
fn main() {
    {
        let r;

        {
            let x = 5;
            r = &x; // x已经无效，所以此处报错
        }

        println!("r: {}", r);
    }
}
```
#### 函数中的泛型生命周期
```
// 编译报错，因为编译器并不知道应该返回x,y谁的引用
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
现在函数签名表明对于某些生命周期 'a，函数会获取两个参数，
他们都是与生命周期 'a 存在的一样长的字符串 slice
函数会返回一个同样也与生命周期 'a 存在的一样长的字符串 slice
```
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```
#### 生命周期注解语法
```
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
```
#### 结构体定义中的生命周期注解
这个注解意味着 ImportantExcerpt 的实例不能比其 part 字段中的引用存在的更久。
```
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```
#### 生命周期省略（Lifetime Elision）
第一条 规则是每一个是引用的参数都有它自己的生命周期参数。换句话说就是，有一个引用参数的函数有一个生命周期参数：fn foo<'a>(x: &'a i32)，有两个引用参数的函数有两个不同的生命周期参数，
fn foo<'a, 'b>(x: &'a i32, y: &'b i32)，依此类推。
第二条 规则是如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数：
fn foo<'a>(x: &'a i32) -> &'a i32。

#### 方法定义中的生命周期注解
#### 静态生命周期
### 编写自动化测试
#### 如何编写测试
#### 测试函数剖析
#### 使用 assert! 宏来检查结果
#### 使用 should_panic 检查 panic
### 控制测试如何运行
### 测试的组织结构
## 一个 I/O 项目：构建一个命令行程序
#### 二进制项目的关注分离
1. 将程序拆分成 main.rs 和 lib.rs 并将程序的逻辑放入 lib.rs 中。
2. 当命令行解析逻辑比较小时，可以保留在 main.rs 中。
3. 当命令行解析开始变得复杂时，也同样将其从 main.rs 提取到 lib.rs 中。

## Rust 中的函数式语言功能：迭代器与闭包
### 闭包：可以捕获环境的匿名函数
Rust 的 闭包（closures）是可以保存在一个变量中或作为参数传递给其他函数的匿名函数。
可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。不同于函数，闭包允许捕获被定义时所在作用域中的值。
我们将展示闭包的这些功能如何复用代码和自定义行为。
#### 使用闭包创建行为的抽象

#### 使用迭代器处理元素序列

## 杂文
vscode提示让初学者眼花缭乱，还是先屏蔽掉
```
"rust-analyzer.inlayHints.typeHints.enable": false,
"rust-analyzer.inlayHints.parameterHints.enable": false
```
## 疑问
:: 与 . 的区别是什么？
```
```


