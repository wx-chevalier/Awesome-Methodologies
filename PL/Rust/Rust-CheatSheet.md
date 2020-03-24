> Rust CheatSheet 是对于 Rust 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wx-chevalier/Awesome-CheatSheets/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。本文参考了许多优秀的文章与代码示范，统一声明在了 [Rust Links](https://github.com/wx-chevalier/Awesome-Lists/blob/master/ProgrammingLanguage/Rust)；如果希望深入了解某方面的内容，可以继续阅读[]()，或者前往 [coding-snippets/rust]() 查看使用 Rust 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

# Rust 语法速览、实践技巧与开源工具清单

Rust 是为工业应用而生，并不拘泥于遵循某个范式( Paradigm )，笔者认为其最核心的特性为 Ownership 与 Lifetime；能够在没有 GC 与 Runtime 的情况下，防止近乎所有的段错误，并且保证线程安全(prevents nearly all segfaults, and guarantees thread safety )。Rust 为每个引用与指针设置了 Lifetime，对象则不允许在同一时间有两个和两个以上的可变引用，并且在编译阶段即进行了内存分配(栈或者堆)；Rust 还提供了 Closure 等函数式编程语言的特性、编译时多态(Compile-time Polymorphism)、衍生的错误处理机制、灵活的模块系统等。

对于 Rust 的语法速览可以参考本目录下的 [rust-snippets](https://parg.co/QN9)。

# 语法基础

```rs
fn main() {
    print!("Hello, world!");
}
```

print: It is the name of a macro defined in the Rust standard library.
!: It specifies that the preceding name indicates a macro. Without such a symbol, print would instead indicate a function. There is no such function in the Rust standard library, and so you would get a compilation error. A macro is a thing similar to a function - it’s some Rust code to which a name is associated. By using this name, you ask to insert such code in this point.

```sh
rustc $\* --color always 2>&1 | more
```

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

# 基本数据类型

# 集合类型

# Vector

![](https://parg.co/U8w)

# 链接

- https://colobu.com/2020/03/05/A-half-hour-to-learn-Rust/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io
