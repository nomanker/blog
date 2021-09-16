---
title: In Rust we trust
categories:
  - 学习之路
date: 2021-09-16 19:02:11
tags:
- Rust
---

## 所有权

**所有权规则**：

1. Rust中的每个值都有一个被称为其所有者的变量
2. 值在任一时刻有且只有一个所有者
3. 当所有者（变量）离开作用域，这个值将被丢弃

### 移动

将一个字符串赋值给另一个字符串，只是从栈上拷贝了它的指针、长度和容量，并没有复制指针指向的堆上数据。如下图。

<img src="https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210907084256806.png" alt="image-20210907084256806" style="zoom: 67%;" />

**二次释放**（double free）：两次释放相同的内存会导致内存污染。

上述例子可以理解为s1被移动到了s2中，同时s1将不再有效，因此不需要在s1离开作用域后清理任何东西。隐含的rust设计选择：永远也不会自动创建数据的“深拷贝”。

### 克隆

深度复制堆上的数据，使用clone的通用函数。

只在栈上的数据则是拷贝，copy注解可以用在存储在栈上的类型上，一个旧的变量在将其赋值给其他变量后依旧可用，同时不允许自身或其任何部分实现了drop trait的类型使用copy。

### 函数中的所有权

使用copy的类型的参数在移出函数作用域时不会失效，当持有堆中数据值的变量离开作用域时，其值将通过drop被清理掉，除非数据被移动为另一个变量所有（返回值移出给调用函数）

### 引用（&）

允许使用值但不获取其所有权。

~~~rust
let s1 = String::from("Hello");
let len = calculate_length(&s1);
~~~

&创建一个**指向**s1的引用，但是并不拥有它，当引用离开作用域时其指向的值也不会被丢弃。

默认不允许修改引用的值。

**借用**：将获取引用作为函数参数称为借用

### 可变引用

被引用的值必须为可变类型。

==在特定作用域中的特定数据只能有一个可变引用==，可以在编译时就避免数据竞争。

**数据竞争**可由以下三个行为造成：

1. 两个或更多指针同时访问同一数据
2. 至少有一个指针被用来写入数据
3. 没有同步数据访问的机制

==不能在拥有不可变引用的同时拥有可变引用==，但是可以==同时拥有多个不可变引用==。

### 悬垂引用

被引用的对象在离开作用域时被释放，引用的指向无效。

### slice（没有所有权）

字符串的类型声明写作&str

字符串字面值是slice

**..range语法**

~~~rust
#![allow(unused)]
fn main() {
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
    
let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
}
~~~

例：

~~~rust
fn main() {
    let a = [1,2,3,4,5];
    let slice = &a[2..a.len()];
    println!("{:?}",slice);
}
~~~

***

## 结构体

### 结构体定义

~~~rust
#![allow(unused)]
fn main() {
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
}
~~~

### 创建结构体

~~~rust
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
~~~

当变量与字段同名时，使用**字段初始化简写语法**

### 元组结构体

没有具体的字段名，只有字段的类型。

~~~rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
~~~

### 类单元结构体

TODO

### #[derive(Debug)]

使得结构体能投打印出调试信息，在print中使用{:?}或者{:#?}

### 方法

与函数类似，它们的第一个参数总是**self**，代表调用该方法的结构体实例。

### 关联函数

允许在impl块中定义不以self为参数的函数，使用结构体名和::语法来调用关联函数。

***

## 枚举

例：

~~~rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
~~~

### Option

变量总是空值和非空值状态之一。

Option<T>定义如下：

~~~rust
enum Option<T> {
    Some(T),
    None,
}
~~~

Option已经被包含在了预导入包中，因此不需要将其显示引入作用域

如果使用None而不是Some，需要标明Option<T>是什么类型的

~~~rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
~~~

==在对Option<T>进行运算之前必须将其转换为T==，只要一个值不是Option类型，它就一定不是空值。

### match控制流

每个分支相关联的代码是一个表达式，其结果将作为整个match表达式的返回值

==匹配必须是穷尽的==，为此Rust提供了_通配符用于不想列举出所有可能值的场景，例：

~~~rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
~~~



### 绑定匹配的模式的部分值

例：

~~~rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
~~~

### 匹配Option<T>

例：

~~~rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
~~~

### if let 简单控制流

例：

~~~rust
fn main() {
    // All have type `Option<i32>`
    let number = Some(7);
    let letter: Option<i32> = None;
    let emoticon: Option<i32> = None;

    // The `if let` construct reads: "if `let` destructures `number` into
    // `Some(i)`, evaluate the block (`{}`).
    if let Some(i) = number {
        println!("Matched {:?}!", i);
    }

    // If you need to specify a failure, use an else:
    if let Some(i) = letter {
        println!("Matched {:?}!", i);
    } else {
        // Destructure failed. Change to the failure case.
        println!("Didn't match a number. Let's go with a letter!");
    }

    // Provide an altered failing condition.
    let i_like_letters = false;

    if let Some(i) = emoticon {
        println!("Matched {:?}!", i);
    // Destructure failed. Evaluate an `else if` condition to see if the
    // alternate failure branch should be taken:
    } else if i_like_letters {
        println!("Didn't match a number. Let's go with a letter!");
    } else {
        // The condition evaluated false. This branch is the default:
        println!("I don't like letters. Let's go with an emoticon :)!");
    }
}
~~~

***

## 模块系统

模块系统包括：

- **包**（*Packages*）： Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates** ：一个模块的树形结构，它形成了库或二进制项目。
- **模块**（*Modules*）和 **use**： 允许你控制作用域和路径的私有性。
- **路径**（*path*）：一个命名例如结构体、函数或模块等项的方式

crate root是一个源文件，编译器以它为起始点，构成crate的跟模块。包是提供一系列功能的一个或者多个crate，一个包会含有一个Cargo.toml文件。

包的规则：

1. 一个包中至多**只能**包含一个库crate
2. 包中可以包含任意多个二进制crate
3. 包中至少包含一个crate，不管是库还是二进制

Cargo遵循的一个约定：src/main.rs或者src/lib.rs是与包同名的crate根，根文件将由cargo传递给rustc来实际构建库或者二进制项目。

通过将文件放在src/bin目录下，一个包可以拥有多个二进制crate，每个文件都会被编译成一个独立的二进制文件。

一个crate的功能是在自身的作用域进行命名的，可以通过标明crate名来避免编译器混淆。

### 定义模块来控制作用域与私有性

使用mod关键字定义一个模块，模块树如下图所示：

![image-20210907195122940](https://ggssh.oss-cn-beijing.aliyuncs.com/mdimg/image-20210907195122940.png)

### 路径

路径有两种形式：

* **绝对路径**：从crate根开始，以crate名或者字面值crate开头
* **相对路径**：从当前模块开始，以self、super或当前模块的标识符开头

Rust中默认所有项（函数、方法、结构体、枚举、模块和常量）都是私有的。父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用它们父模块中的项。

==模块共有并不表示其内容也是私有的==

**使用super起始的相对路径**

~~~rust
fn serve_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::serve_order();
    }

    fn cook_order() {}
}
~~~

### use

使用use关键字将路径一次性引入**作用域**，然后调用该路径中的项。

* 使用use将函数引入作用域的习惯用法：要想使用use将函数的父模块引入作用域，必须在调用函数时指定父模块，这样可以清晰的表明函数不是在本地定义的，同时使完整路径的重复度最小化

* 在使用use引入结构体、枚举和其他项时，习惯是指定它们的完整路径。

* 使用父模块将两个具有相同名称的类型引入同一作用域
* 使用as可以重命名导入的类型名（类似于Python）

**pub use**

通过pub use使名称可以引入任何代码的作用域中

***

## 集合（collections）

### vector

[rust Vec 常用操作](https://blog.csdn.net/qq_39308071/article/details/114063758)

### 字符串

format!宏返回一个带有结果内容的String，并且不会获取任何参数的所有权。

Rust的字符串不支持索引

**遍历字符串**

如果需要操作单独的Unicode标量值，最好使用chars方法；bytes方法返回每一个原始字节

### HashMap

像String那样拥有所有权的值，其值将被移动而HashMap会成为这些值的所有者；如果将值的引用插入HashMap，这些值本身将不会被移动进HashMap，但是这些引用指向的值必须至少在HashMap有效时也是有效的。

**只在键没有对应值时插入**

entry的or_insert方法在键对应的值存在时就返回这个值的可变引用，如果不存在则将参数作为新值插入并返回新值的可变引用。

***

## 错误处理

当出现panic时，程序默认会开始展开，另一种选择是直接终止，这会不清理数据就退出程序，那么程序所使用的内存需要由操作系统来清理。如果你需要项目的最终二进制文件越小越好，panic 时通过在 *Cargo.toml* 的 `[profile]` 部分增加 `panic = 'abort'`，可以由展开切换为终止。例如，如果你想要在release模式中 panic 时直接终止：

~~~toml
[profile.release]
panic = 'abort'
~~~

backtrace是一个执行到目前位置所有被调用的函数的列表，可以通过将RUST_BACKTRACE环境变量设置为任何不是0的值来获取backtrace。

为了获取带有这些信息的 backtrace，必须启用 debug 标识。当不使用 `--release` 参数运行 cargo build 或 cargo run 时 debug 标识会默认启用。

**匹配不同的错误**

~~~rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
~~~

***

## 泛型、trait与生命周期

重复冗余的代码是非常容易出错的，并且意味着当更新逻辑时需要修改多处地方的代码。

例：

~~~rust
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
~~~

### 结构体中的泛型

### 枚举定义中的泛型

例：

~~~rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

enum Option<T> {
    Some(T),
    None,
}
~~~

### 方法定义中的泛型

例：

~~~rust
struct Point<T> {
    x:T,
    y:T,
}
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
~~~

~~~rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
~~~

注意必须在impl后面声明T，这样就可以在Point<T>上实现的方法中使用它了，在impl之后声明泛型T，这样Rust就知道Point的尖括号中的类型是泛型而不是具体类型。

### 泛型的性能

**单态化**：通过填充编译时使用的具体类型，将通用代码转换为特定代码的过程。编译器寻找所有泛型代码被调用的位置并使用泛型代码针对具体类型生成代码。

Rust将为每一个实例编译其特定类型的代码，这意味着在使用泛型的过程中没有运行时开销，当代码执行时，它的执行效率就像手写每个具体定义的重复代码一样。

### 为类型实现trait

~~~rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
~~~

**实现trait的限制**：

* 只有当trait或者要实现的trait的类型位于crate的本地作用域中，才能为该类型实现trait
* 不能为外部类型实现外部trait，例如，不能在 `aggregator` crate 中为 `Vec<T>` 实现 `Display` trait。这是因为 `Display` 和 `Vec<T>` 都定义于标准库中，它们并不位于 `aggregator` crate 本地作用域中。这个限制是被称为 **相干性**（*coherence*） 的程序属性的一部分，或者更具体的说是 **孤儿规则**（*orphan rule*），其得名于不存在父类型。这条规则确保了其他人编写的代码不会破坏你代码。

可以选择保留或者重载每个方法的**默认行为**，例：

~~~rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
~~~

### trait作为参数

例：

~~~rust
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
~~~

我们指定了 `impl` 关键字和 trait 名称，而不是具体的类型。该参数支持任何实现了指定 trait 的类型。在 `notify` 函数体中，可以调用任何来自 `Summary` trait 的方法。

### trait bound

例：

~~~rust
pub fn notify<T: Summary>(item1: T, item2: T){}
~~~

泛型T被指定为item1和item2的参数限制，如此传递给参数item1和item2值的具体类型必须一致。

#### 通过 + 指定多个trait bound

例：

~~~rust
pub fn notify(item: impl Summary + Display) {
}

// 同样适用于泛型的trait bound
pub fn notify<T: Summary + Display>(item: T) {
}
~~~

#### 使用where来简化trait bound

例如，以下两种形式等价：

~~~rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
~~~

使用where从句

~~~rust
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
~~~

#### 使用trait bound有条件地实现方法

通过使用带有trait bound地泛型参数的impl块，可以有条件地只为那些实现了特定trait地类型实现方法。

### 返回实现了trait的类型

可以在返回值中使用impl trait语法，来返回实现了某个trait的类型，例：

~~~rust
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
~~~

### 生命周期

Rust中的每一个引用都有其**生命周期**（Rust最最与众不同的功能）

#### 悬垂引用

悬垂引用的意思就是当源数据已经释放了，而对源数据的引用还存在，这个引用就被称为悬垂引用，C和C++中需要开发者手动保证不会发生悬垂引用，而Rust以安全著称，编译器保证不会发生这种情况。

发生悬垂引用的两种情况：

1. 返回值来源于函数体内创建的变量，必然发生的悬垂引用：

   ~~~rust
   fn foo() -> &String {
     let s = String::new();
     &s
   }
   ~~~

   在函数体内创建的变量会在函数调用结束后销毁，所以返回的&S就是悬垂引用。

2. 返回值是来源于参数，但不清楚返回哪一个，有可能发生的悬垂引用：

   ~~~rust
   fn longer(s1: &str, s2: &str) -> &str {
     if s1.len() > s2.len() {
       s1
     } else {
       s2
     }
   }
   ~~~

   对于上面情况，因为代码里存在判断条件，rust在编译阶段是不能确定，最后的返回值是s1还是s2。

   当我们不清楚参数的声明周期时，就有可能会发生悬垂引用：

   ~~~rust
   let a = String::from("a");
   let c;
     {
       let b = String::from("bb");
       // 在编译阶段，rust只能知道调用longer函数
       // 但是不清楚返回值是&a还是&b
       c = longer(&a, &b);
       // 如果返回了&b，将&b绑定给了c
       // 但是b在这个作用域结束后被销毁了，
       // 所以c变成了悬垂引用
     }
     println!("{}", c);
   ~~~

   防止上面情况的解决方案是，rust在编译阶段能够知道在调用longer后返回值的生命周期，也就是上面c的生命周期，因为rust不清楚longer返回值的生命周期，但是为了防止在运行时上面这种悬垂指针，所以只能在编译期间报错：“不清楚longer函数的返回值生命周期”。

#### 借用检查器

Rust编译器有一个借用检查器，用来比较作用域来确保所有的借用都是有效的，例：

~~~rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}      
~~~

这是一个有效的引用，因为数据比引用有着更长的生命周期。

#### 生命周期注解语法

生命周期注解并不改变任何引用的生命周期的长短，与当函数签名中指定了泛型类型参数后就可以接受任何类型一样，当指定了泛型生命周期后函数也能接受任何生命周期的引用。生命周期注解描述了多个引用生命周期相互的关系，而不影响其生命周期。

**生命周期的语法**：必须以 ' 开头，其名称通常全是小写，类似于泛型其名称非常短。'a 是大多数人默认使用的名称，生命周期参数注解位于引用的 & 之后。并有一个空格来将引用类型与生命周期注解分隔开。

例：

~~~rust
&i32        // 引用
&'a i32     // 带有显式生命周期的引用
&'a mut i32 // 带有显式生命周期的可变引用
~~~

#### 结构体定义中的生命周期注解

~~~rust
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
~~~

#### 生命周期省略

函数或方法的参数的生命周期被称为**输入生命周期**，而返回值的生命周期被称为**输出生命周期**。

**生命周期省略规则**：

1. 每一个是引用的参数都有它自己的生命周期参数。即，有一个引用参数的函数有一个生命周期参数，有两个引用参数的函数有两个不同的生命周期参数。
2. 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数。
3. 如果方法有多个输入生命周期参数并且其中一个参数是&self或&mut self，说明是个对象的方法，那么所有输出生命周期参数被赋予self的生命周期。

#### 方法定义中的生命周期注解

例：

~~~rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
~~~

#### 静态生命周期

'static ，其生命周期能够存活于整个程序期间，所有的字符串字面值都拥有'static 生命周期，例：

~~~rust
let s: &'static str = "I have a static lifetime.";
~~~

这个字符串的文本被直接储存在程序的二进制文件中而这个文件总是可用的。因此所有的字符串字面值都是 `'static` 的。

#### 结合泛型类型参数、trait bounds 和生命周期

~~~rust
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
~~~

生命周期参数要写在最前面。

***

## 闭包

闭包可以保存进变量或者作为参数传递给其他函数的匿名函数，可以在一个地方创建闭包，然后在不同的上下文中执行闭包运算。

### 重构使用闭包存储代码

~~~rust
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
~~~

闭包的定义以一对竖线开始，在竖线中指定闭包的参数，如果有一个多于一个参数，可以使用逗号分隔，比如|param1,param2|

参数之后是存放闭包体的大括号 —— 如果闭包体只有一行则大括号是可以省略的。大括号之后闭包的结尾，需要用于 `let` 语句的分号。因为闭包体的最后一行没有分号（正如函数体一样），所以闭包体（`num`）最后一行的返回值作为调用闭包时的返回值 。

### 闭包类型推断和注解

因为闭包并不用于暴露在外的接口：它们储存在变量中并被使用，不用命名它们或暴露给库的用户调用。

~~~rust
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
~~~

闭包定义会为每个参数和返回值推断一个具体类型，如果尝试调用闭包两次，第一次使用String类型作为参数，而第二次使用u32，则会报错。

### 带有泛型和Fn trait的闭包

#### memoization/lazy evaluation（惰性求值）

创建一个存放闭包和调用闭包结果的结构体，该结构体只会在需要结果时执行闭包，并会缓存结果值，这样余下的代码就不必再负责保存结果并可以复用该值。例：

~~~rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
~~~

闭包可以通过三种方式捕获其环境，他们直接对应函数的三种获取参数的方式：获取所有权，可变借用和不可变借用。这三种捕获值的方式被编码为如下三个 `Fn` trait：

- `FnOnce` 消费从周围作用域捕获的变量，闭包周围的作用域被称为其 **环境**，*environment*。为了消费捕获到的变量，闭包必须获取其所有权并在定义闭包时将其移动进闭包。其名称的 `Once` 部分代表了闭包不能多次获取相同变量的所有权的事实，所以它只能被调用一次。
- `FnMut` 获取可变的借用值所以可以改变其环境
- `Fn` 从其环境获取不可变的借用值

所有闭包都可以被调用至少一次，所以所有闭包都实现了 `FnOnce` 。那些并没有移动被捕获变量的所有权到闭包内的闭包也实现了 `FnMut` ，而不需要对被捕获的变量进行可变访问的闭包则也实现了 `Fn` 。

使用move关键字会获取所有权。

***

## 迭代器

在Rust中，迭代器是**惰性的**，这意味着在调用方法使用迭代器之前它都不会有效果。

例：

~~~rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
~~~

要使用next()的话，iter需要是可变的

### 消费迭代器的方法

`sum` 方法获取迭代器的所有权并反复调用 `next` 来遍历迭代器，因而会消费迭代器。当其遍历每一个项时，它将每一个项加总到一个总和并在迭代完成时返回总和。

例：调用迭代器适配器map来创建一个新迭代器

~~~rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
~~~

例：调用filter将迭代器适配成一个只含有那些闭包返回true的元素的新迭代器

~~~rust
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

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
~~~

### 实现Iterator trait来创建自定义迭代器

实现Iterator trait的唯一要求是实现next方法，例：

~~~rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
~~~

***

## 智能指针

- `Box<T>`，用于在堆上分配值
- `Rc<T>`，一个引用计数类型，其数据可以有多个所有者
- `Ref<T>` 和 `RefMut<T>`，通过 `RefCell<T>` 访问。（ `RefCell<T>` 是一个在运行时而不是在编译时执行借用规则的类型）。

### Deref强制转换如何与可变性交互

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当 `T: Deref<Target=U>` 时从 `&T` 到 `&U`。
- 当 `T: DerefMut<Target=U>` 时从 `&mut T` 到 `&mut U`。
- 当 `T: Deref<Target=U>` 时从 `&mut T` 到 `&U`。

头两个情况除了可变性之外是相同的：第一种情况表明如果有一个 `&T`，而 `T` 实现了返回 `U` 类型的 `Deref`，则可以直接得到 `&U`。第二种情况表明对于可变引用也有着相同的行为。

第三个情况有些微妙：Rust 也会将可变引用强转为不可变引用。但是反之是 **不可能** 的：==不可变引用永远也不能强转为可变引用==。因为根据借用规则，如果有一个可变引用，其必须是这些数据的唯一引用（否则程序将无法编译）。将一个可变引用转换为不可变引用永远也不会打破借用规则。将不可变引用转换为可变引用则需要数据只能有一个不可变引用，而借用规则无法保证这一点。因此，Rust 无法假设将不可变引用转换为可变引用是可能的。

### Drop Trait 运行清理代码

通过std::mem::drop提早丢弃值，例：

~~~rust
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
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
~~~

### Rc 引用计数智能指针

引用计数意味着记录一个值的引用的数量来确定这个值是否仍在被使用，如果某个值有零个引用，就代表没有任何有效引用并可以被清理。因此Rc用于希望在堆上分配一些内存供程序的多个部分读取，而且无法在编译时确定程序的哪一部分会最后结束使用它时。==Rc只能用于单线程场景==。

#### 使用Rc共享数据

不能用两个Box的列表尝试共享第三个列表的所有权。改进使用Rc如下：

~~~rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
~~~

上述main函数中也可以调用a.clone()而不是Rc::clone(&a)，不过习惯使用Rc::clone，它只会增加引用计数，并不像大部分类型的clone实现那样对所有的数据进行深度拷贝，从而减少了时间。

Rc<T>不允许多个可变引用，会违反借用规则之一：相同位置的多个可变借用可能造成数据竞争和不一致，引出RefCell<T>。

### RefCell<T>和内部可变性模式

**内部可变性**：允许即使在有不可变引用时也可以改变数据，为了改变数据，该模式在数据结构中使用unsafe来模糊Rust通常的可变性和借用规则。

**借用规则**：

1. 在任意给定时刻，只能同时拥有一个可变引用或任意数量的不可变引用之一（而不是两者）
2. 引用必须总是有效的

对于引用和 `Box<T>`，借用规则的不可变性作用于编译时。对于 `RefCell<T>`，这些不可变性作用于 **运行时**。对于引用，如果违反这些规则，会得到一个编译错误。而对于 `RefCell<T>`，如果违反这些规则程序会 panic 并退出。

`RefCell<T>` 正是用于当你确信代码遵守借用规则，而编译器不能理解和确定的时候。==同样只能用于单线程场景==。

**总结**：

- `Rc<T>` 允许相同数据有多个所有者；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时执行不可变或可变借用检查；`Rc<T>`仅允许在编译时执行不可变借用检查；`RefCell<T>` 允许在运行时执行不可变或可变借用检查。
- 因为 `RefCell<T>` 允许在运行时执行可变借用检查，所以我们可以在即便 `RefCell<T>` 自身是不可变的情况下修改其内部的值。

例：使用RefCell<T>能够在外部值被认为是不可变的情况下修改内部值

~~~rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
~~~

#### RefCell<T>在运行时记录借用

当创建不可变和可变引用时，我们分别使用 `&` 和 `&mut` 语法。对于 `RefCell<T>` 来说，则是 `borrow` 和 `borrow_mut` 方法，这属于 `RefCell<T>` 安全 API 的一部分。`borrow` 方法返回 `Ref<T>` 类型的智能指针，`borrow_mut` 方法返回 `RefMut` 类型的智能指针。这两个类型都实现了 `Deref`，所以可以当作常规引用对待。

`RefCell<T>` 记录当前有多少个活动的 `Ref<T>` 和 `RefMut<T>` 智能指针，在任何时候只允许有多个不可变借用或一个可变借用。

#### 结合Rc<T>和RefCell<T>来拥有多个可变数据所有者

使用存储了RefCell<T>的Rc<T>可以实现同时拥有多个所有者并且能够修改值。

例：

~~~rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
~~~

### 引用循环与内存泄漏

#### 避免引用循环：将Rc<T>变为Weak<T>

通过调用Rc::downgrade传递Rc<T>实例的引用来创建其值的弱引用，调用Rc::downgrade会得到Weak<T>类型的智能指针，会将weak_count加1.`Rc<T>` 类型使用 `weak_count` 来记录其存在多少个 `Weak<T>` 引用，类似于 `strong_count`。其区别在于 ==`weak_count` 无需计数为 0 就能使 `Rc<T>` 实例被清理==。

强引用代表如何共享 `Rc<T>` 实例的所有权，但弱引用并不属于所有权关系。他们不会造成引用循环，因为任何弱引用的循环会在其相关的强引用计数为 0 时被打断。

调用Weak<T>实例的upgrade方法，会返回一个Option<T>

***

## 线程

线程会出现的问题：

* 竞争状态：多个线程以不一致的顺序访问数据或资源
* 死锁：两个线程相互等待对方停止使用其所拥有的资源，这会组织它们继续运行
* 只会发生在特定情况且难以稳定重现和修复的bug

编程语言提供的线程被称为绿色线程，使用绿色线程的语言会在不同数量的OS线程的上下文中执行它们，绿色线程被称为M:N模型：M个绿色线程对应N个OS线程，这里M和N不必相同。

绿色线程的M:N模型需要更大的语言运行时来管理这些线程，因此，rust标准库只提供了1:1线程模型实现，由于Rust是较为底层的语言，可以使用实现了M:N线程模型的crate来牺牲性能换取抽象，以获得对线程运行更精细的控制及更低的上下文切换成本。

### 创建新线程

~~~rust
let handle = thread::spawn(|| {
    for i in 1..10{
        println!("{}",i);
        thread::sleep(Duration::from_millis(1));
    }
});

for i in 1..5{
    println!("{}",i);
    thread::sleep(Duration::from_millis(1));
}
handle.join().unwrap();
~~~

### 线程与move闭包

例：

~~~rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
~~~

### 使用消息传递在线程间传送数据

来源于Go编程语言文档中的口号：“不要通过共享内存来通讯，而是通过通讯来共享内存。”

通道（channel）

* 发送者
* 接收者

当发送者或接收者任一被丢弃时可以认为通道被关闭了。

mpsc是**多个生产者，单个消费者**（multiple producer，single consumer），简言之就是一个通道可以有多个产生值的发送端，但只能有一个消费这些值的接收端。

### 通过克隆发送者来创建多个生产者

例：

~~~rust
let (tx, rx) = mpsc::channel();

let tx1 = tx.clone();
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

~~~



***

## Rust的面向对象编程特性

**鸭子类型**：如果它走起来像一只鸭子，叫起来像一只鸭子，那么它就是一只鸭子。即只关心值所反映的信息而不是其具体类型。



### Trait对象要求对象安全

如果一个trait中的所有方法有如下属性是，则该trait是对象安全的额

* 返回值类型不为Self
* 方法没有任何泛型类型参数

***

## 高级特征

### 不安全的Rust

- 解引用裸指针
- 调用不安全的函数或方法
- 访问或修改可变静态变量
- 实现不安全 trait
- 访问 `union` 的字段

#### 解引用裸指针

裸指针是可变的或不可变的：

* *const T
* *mut T

在裸指针的上下文中，不可变意味着指针解引用之后不能直接赋值。

裸指针与引用和智能指针的区别：

1. 允许忽略借用规则，可以同时拥有不可变和可变的指针，或多个指向相同位置的可变指针
2. 不保证指向有效的内存
3. 允许为空
4. 不能实现任何自动清理功能

从引用同时创建不可变和可变裸指针：

~~~rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
~~~

#### 调用不安全函数或方法

例：

~~~rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
~~~

#### 创建不安全代码的安全抽象

例：在split_at_mut函数的实现中使用不安全代码

~~~rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.add(mid), len - mid))
    }
}
~~~

#### 使用extern函数调用外部代码

外部函数接口（FFI）是一个编程语言用以定义函数的方式，其允许不同（外部）编程语言调用这些函数。

例：声明并调用另一个语言中定义的extern函数

~~~rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
~~~

`"C"` 部分定义了外部函数所使用的 **应用二进制接口**（*application binary interface*，ABI） —— ABI 定义了如何在汇编语言层面调用此函数。`"C"` ABI 是最常见的，并遵循 C 编程语言的 ABI。

**从其他语言调用Rust函数**（无需unsafe）

使用extern来创建一个允许其他语言调用rust函数的接口，不同于 `extern` 块，就在 `fn` 关键字之前增加 `extern` 关键字并指定所用到的 ABI。还需增加 `#[no_mangle]` 注解来告诉 Rust 编译器不要 mangle 此函数的名称。*Mangling* 发生于当编译器将我们指定的函数名修改为不同的名称时，这会增加用于其他编译过程的额外信息，不过会使其名称更难以阅读。每一个编程语言的编译器都会以稍微不同的方式 mangle 函数名，所以为了使 Rust 函数能在其他语言中指定，必须禁用 Rust 编译器的 name mangling。

例：一旦其编译为动态库并从 C 语言中链接，`call_from_c` 函数就能够在 C 代码中访问

~~~rust
#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
~~~

#### 访问或修改可变静态变量

例：

~~~rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
~~~

静态变量只能储存拥有'static生命周期的引用， 访问不可变静态变量是安全的。

常量和不可变静态变量的区别：

* 静态变量中的值有一个固定的内存地址，使用这个值总是会访问相同的地址
* 常量允许在任何被用到的时候复制其数据
* 静态变量是可变的

例：读取或修改一个可变静态变量是不安全的

~~~rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
~~~

#### 实现不安全trait

当trait中至少有一个方法中包含编译器无法验证的不变式（invariant）时，trait时不安全的。例：

~~~rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
~~~

#### 访问联合体中的字段

union和struct类似，但是在一个实例中同时只能使用一个声明的字段，联合体主要用于和C代码中的联合体交互。

### 高级trait

#### 关联类型

#### 默认泛型类型参数和运算符重载

当使用泛型类型参数时，可以为泛型指定一个默认的具体类型，如果默认类型就足够的话，就消除了为具体类型实现trait的需要，为泛型类型指定默认类型的语法是在声明泛型类型时使`<PlaceholderType=ConcreteType>`。

Rust 并不允许创建自定义运算符或重载任意运算符，不过 `std::ops` 中所列出的运算符和相应的 trait 可以通过实现运算符相关 trait 来重载。例如，示例 19-14 中展示了如何在 `Point` 结构体上实现 `Add` trait 来重载 `+` 运算符，这样就可以将两个 `Point` 实例相加了：

~~~rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
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

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
~~~

**默认类型参数**

RHS：泛型类型参数（right hand side）

默认参数类型主要用于以下两个方面：

* 扩展类型而不破坏现有代码
* 在大部分用户都不需要的特定情况进行自定义
