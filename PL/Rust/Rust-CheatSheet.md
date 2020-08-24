> Rust CheatSheet 是对于 Rust 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wx-chevalier/Awesome-CheatSheets/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。本文参考了许多优秀的文章与代码示范，统一声明在了 [Awesome Rust List](https://github.com/wx-chevalier/Awesome-Lists)；如果希望深入了解某方面的内容，可以继续阅读[Rust-Series](https://github.com/wx-chevalier/Rust-Series)，或者前往 [rust-examples](https://github.com/wx-chevalier/rust-examples) 查看使用 Rust 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

# Rust 语法速览、实践技巧与开源工具清单

Rust 是为工业应用而生，并不拘泥于遵循某个范式（Paradigm），笔者认为其最核心的特性为 Ownership 与 Lifetime；能够在没有 GC 与 Runtime 的情况下，防止近乎所有的段错误，并且保证线程安全(prevents nearly all segfaults, and guarantees thread safety )。Rust 为每个引用与指针设置了 Lifetime，对象则不允许在同一时间有两个和两个以上的可变引用，并且在编译阶段即进行了内存分配(栈或者堆)；Rust 还提供了 Closure 等函数式编程语言的特性、编译时多态(Compile-time Polymorphism)、衍生的错误处理机制、灵活的模块系统等。

对于 Rust 的语法速览可以参考本目录下的 [rust-snippets](./rust-snippets.rs)。

# 表达式与控制流

```rs
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>

let v = vec![0; 10]; // A vector of ten zeroes.

let v = vec![1, 2, 3, 4, 5];

println!("The third element of v is {}", v[2]);
```

```rs
fn main() {
    println!("{}", "These
are
three lines");
}

fn main() {
println!("{}", "These\n\
    are\n\
    three lines");
}
```

## 变量与代码块声明

使用 let 声明一个变量（声明一个变量相当于告诉 Rust 创建一个变量）。

```rs
fn main() {
    let my_number = 8;
    println!("Hello, number {}", my_number);
}
```

变量在代码块 {} 中开始和结束。在此示例中，my_number 在我们调用 println！之前结束，因为它在其自己的代码块内。

```rs
fn main() {
    {
        let my_number = 8; // my_number starts here
                           // my_number ends here!
    }

    println!("Hello, number {}", my_number); // ⚠️ there is no my_number and
                                             // println!() can't find it
}
```

您可以使用代码块返回值：

```rs
fn main() {
    let my_number = {
    let second_number = 8;
        second_number + 9 // No semicolon, so the code block returns 8 + 9.
                          // It works just like a function
    };

    println!("My number is: {}", my_number);
}
```

如果在块内添加分号，它将返回 ()（无）：

```rs
fn main() {
    let my_number = {
    let second_number = 8; // declare second_number,
        second_number + 9; // add 9 to second_number
                           // but we didn't return it!
                           // second_number dies now
    };

    println!("My number is: {:?}", my_number); // my_number is ()
}
```

## 展示与调试

# 基本数据类型

Rust 具有称为原始类型的简单类型。我们将从整数和 char（字符）开始。

## 整数

整数是没有小数点的整数。整数有两种类型：有符号整数，无符号整数。ed Integers）。符号表示 +（加号）和 -（减号），因此带符号的整数可以为正数或负数（例如 +8，-8）。但是无符号整数只能是正数，因为它们没有符号。有符号整数是：`i8`，`i16`，`i32`，`i64`，`i128` 和 `isize`。无符号整数是：`u8`，`u16`，`u32`，`u64`，`u128` 和 `usize`。

i 或 u 后面的数字表示该数字的位数，因此位数较多的数字可以更大。1 个字节包含 8 位，因此 i8 是 1 个字节，i64 是 8 个字节，依此类推。较大的数字类型可以容纳更大的数字。例如，一个 u8 最多可以容纳 255 个，但是一个 u16 最多可以容纳 65535 个。一个 u128 最多可以容纳 340282366920938463463374374431431768211455。那么 isize 和 usize 是什么？这意味着您的计算机类型上的位数（这称为计算机的体系结构）。因此，在 32 位计算机上进行 isize 和 usize 就像 i32 和 u32，而在 64 位计算机上进行 isize 和 usize 就像 i64 和 u64。

整数类型不同的原因很多。原因之一是计算机性能：较少的字节数处理速度更快。但这还有其他用途：Rust 中的字符称为 char。每个字符都有一个数字：字母 A 为数字 65，而字符`友`（中文的“friend”）为数字 21451。数字列表称为“Unicode”。 Unicode 对使用更多的字符使用较小的数字，例如 A 到 Z 或数字 0 到 9，或空格。

```rust
fn main() {
    let first_letter = 'A';
    let space = ' '; // A space inside ' ' is also a char
    let other_language_char = 'Ꮔ'; // Thanks to Unicode, other languages like Cherokee display just fine too
    let cat_face = '😺'; // Emojis are characters too
}
```

使用最多的字符得到的数字小于 256，可以放入 u8。这意味着 Rust 可以使用 as 安全地将 u8 转换为 char（将 u8 转换为 char 意味着“假装 u8 是一个 char”）。使用 as 进行转换非常有用，因为 Rust 非常严格。它总是需要知道类型，并且即使它们都是整数，也不会让您同时使用两种不同的类型。例如，这将不起作用：

```rust
fn main() { // main() is where Rust programs start to run. Code goes inside {} (curly brackets)

    let my_number = 100; // We didn't write a type of integer,
                         // so Rust chooses i32. Rust always
                         // chooses i32 for integers if you don't
                         // tell it to use a different type

    println!("{}", my_number as char); // ⚠️
}

error[E0604]: only `u8` can be cast as `char`, not `i32`
 --> src\main.rs:3:20
  |
3 |     println!("{}", my_number as char);
  |                    ^^^^^^^^^^^^^^^^^
```

幸运的是，我们可以使用 as 轻松解决此问题。我们不能将 i32 设为 char，但是可以将 i32 设为 u8。然后我们可以将 u8 设为 char。因此，在一行中，我们使用 as 将 my_number 设置为 u8，然后再次将其设置为 char。现在它将编译：

```rust
fn main() {
    let my_number = 100;
    println!("{}", my_number as u8 as char);
}
```

这是大小不同的另一个原因：usize 是 Rust 用于索引的大小（索引的意思是“哪个项目是第一个项目”，“哪个项目是第二个项目”等）。usize 是索引的最佳大小，因为：索引不能为负，因此它必须是一个带有 u 的数字，它应该很大，因为有时您需要索引很多东西，但是它不能是 u64，因为 32 位计算机不能使用它。因此，Rust 使用 usize，以便您的计算机可以获得最大数量的可读取索引。

## char

让我们进一步了解 char。您看到一个 char 始终是一个字符，并使用 `''` 而不是 `""`。所有字符均为 4 个字节。它们是 4 个字节，因为字符串中的某些字符超过一个字节。一直在计算机上使用的基本字母是 1 个字节，后面的字符是 2 个字节，其他字符是 3 和 4。一个 char 是 4 个字节，因此可以容纳所有这些字符。

```rust
fn main() {
    println!("{}", "a".len()); // .len() gives the size in bytes
    println!("{}", "ß".len());
    println!("{}", "国".len());
    println!("{}", "𓅱".len());
}

1
2
3
4
```

您可以看到 `a` 是一个字节，德语 `ß` 是两个字节，日本 `国` 字是三个字节，古埃及语 `𓅱` 是 4 个字节。

```rust
fn main() {
    let slice = "Hello!";
    println!("Slice is {} bytes.", slice.len());
    let slice2 = "안녕!"; // Korean for "hi"
    println!("Slice2 is {} bytes.", slice2.len());
}
```

slice 的长度为六个字符和六个字节，而 slice2 的长度为三个字符和七个字节。 char 需要以任何语言适合任何字符，因此它是 4 个字节长。如果 `.len()` 以字节为单位提供大小，那么以字符为单位的大小呢？稍后我们将学习这些方法，但是您只需记住 `.chars().count()` 即可完成。

```rust
fn main() {
    let slice = "Hello!";
    println!("Slice is {} characters.", slice.chars().count());
    let slice2 = "안녕!";
    println!("Slice2 is {} characters.", slice2.chars().count());
}

Slice is 6 characters.
Slice2 is 3 characters.
```

## Type inference

类型推断意味着，如果您不告诉编译器类型，但是它可以自行决定，它将决定。编译器总是需要知道变量的类型，但是您并不需要总是告诉它。例如，对于让 `my_number = 8`，my_number 将是一个 i32。那是因为如果您不告诉编译器，编译器会选择 i32 作为整数。但是，如果您说 `my_number：u8 = 8`，它将使 my_number 为 u8，因为您已将其告知 u8。因此通常编译器可以猜测。但是有时由于两个原因，开发者需要指定具体的类型：您做的事情非常复杂，编译器不知道所需的类型；您需要其他类型（例如，您需要 i128，而不是 i32）。

要指定类型，请在变量名称后添加一个冒号。

```rs
fn main() {
    let small_number: u8 = 10;
}
```

对于数字，您可以在数字后面说出类型。您不需要空格，只需在数字后面输入即可。

```rs
fn main() {
    let small_number = 10u8; // 10u8 = 10 of type u8
}
```

如果您想使数字易于阅读，也可以添加 `_`。

```rs
fn main() {
    let small_number = 10_u8; // This is easier to read
    let big_number = 100_000_000_i32; // 100 million is easy to read with _
}
```

`_` 不会更改数字。这只是为了使您易于阅读。不管您使用多少 `_`：

```rs
fn main() {
    let number = 0________u8;
    let number2 = 1___6______2____4______i32;
    println!("{}, {}", number, number2);
}

// 0, 1624
```

## Floats

浮点数是带小数点的数字。 5.5 是浮点数，而 6 是整数。 5.0 也是浮点数，甚至 5. 也是浮点数。

```rs
fn main() {
    let my_float = 5.; // Rust sees . and knows that it is a float
}
```

但是这些类型不称为 float，它们分别称为 f32 和 f64。它与整数相同：f 后面的数字表示位数。如果您不输入类型，Rust 将选择 f64。当然，只能将相同类型的浮标一起使用。因此，您无法将 f32 添加到 f64。

```rs
fn main() {
    let my_float: f64 = 5.0; // This is an f64
    let my_other_float: f32 = 8.5; // This is an f32

    let third_float = my_float + my_other_float; // ⚠️
}

error[E0308]: mismatched types
 --> src\main.rs:5:34
  |
5 |     let third_float = my_float + my_other_float;
  |                                  ^^^^^^^^^^^^^^ expected `f64`, found `f32`
```

当您使用错误的类型时，编译器会写 `expected(type), found(type)`。它会像这样读取您的代码：

```rs
fn main() {
    let my_float: f64 = 5.0; // The compiler sees an f64
    let my_other_float: f32 = 8.5; // The compiler sees an f32. It is a different type.
    let third_float = my_float + // The compiler sees a new variable. It must be an f64 plus another f64. Now it expects an f64...
    let third_float = my_float + my_other_float;  // ⚠️ it found an f32. It can't add them.
}
```

当然，使用简单的数字很容易解决。您可以使用以下命令将 f32 转换为 f64：

```rs
fn main() {
    let my_float: f64 = 5.0;
    let my_other_float: f32 = 8.5;

    let third_float = my_float + my_other_float as f64; // my_other_float as f64 = use my_other_float like an f64
}
```

甚至更简单地，删除类型声明。 Rust 将选择可以加在一起的类型。

```rs
fn main() {
    let my_float = 5.0; // Rust will choose f64
    let my_other_float = 8.5; // Here again it will choose f64

    let third_float = my_float + my_other_float;
}
```

Rust 编译器很聪明，如果需要 f32，则不会选择 f64：

```rs
fn main() {
    let my_float: f32 = 5.0;
    let my_other_float = 8.5; // Rust will choose f32,

    let third_float = my_float + my_other_float; // because it knows you need to add it to an f32
}
```

# 集合类型

## Vector

![Rust Vector](https://s1.ax1x.com/2020/08/02/aYugeI.png)

# 链接

- https://colobu.com/2020/03/05/A-half-hour-to-learn-Rust/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io
- https://github.com/Dhghomon/easy_rust#const-and-static
- https://cheats.rs/#data-structures
