[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Rust 语法速览、实践技巧与开源工具清单

Rust CheatSheet 是对于 Rust 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。 本文参考了许多优秀的文章与代码示范，统一声明在了 [Rust Links](https://github.com/wxyyxc1992/Awesome-Reference/blob/master/ProgrammingLanguage/Rust/Rust-Links.md)；如果希望深入了解某方面的内容，可以继续阅读[]()，或者前往 [coding-snippets/rust]() 查看使用 Rust 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

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
