[![返回目录](https://parg.co/UCb)](https://github.com/wxyyxc1992/Awesome-CheatSheet)

# Go 语法速览与实践清单

Go CheatSheet 是对于 Go 学习/实践过程中的语法与技巧进行盘点，其属于 [Awesome CheatSheet](https://github.com/wxyyxc1992/Awesome-CheatSheet/) 系列，致力于提升学习速度与研发效能，即可以将其当做速查手册，也可以作为轻量级的入门学习资料。 本文参考了许多优秀的文章与代码示范，统一声明在了 [Go Links](https://github.com/wxyyxc1992/Awesome-Reference/blob/master/ProgrammingLanguage/Go/Go-Links.md)；如果希望深入了解某方面的内容，可以继续阅读 [Go 开发：语法基础与工程实践](https://github.com/wxyyxc1992/ProgrammingLanguage-Series/blob/master/Go/README.md)，或者前往 [coding-snippets/go](https://github.com/wxyyxc1992/coding-snippets/) 查看使用 Go 解决常见的数据结构与算法、设计模式、业务功能方面的代码实现。

# 环境配置与语法基础

可以前往[这里](https://golang.org/dl/)下载 Go SDK 安装包，或者使用 brew 等包管理器安装。go 命令依赖于 $GOPATH 环境变量进行代码组织，多项目情况下也可以使用 ln 进行目录映射以方便进行项目管理。GOPATH 允许设置多个目录，每个目录都会包含三个子目录：src 用于存放源代码，pkg 用于存放编译后生成的文件，bin 用于存放编译后生成的可执行文件。

环境配置完毕后，可以使用 go get 获取依赖，go run 运行程序，go build 来编译项目生成与包名（文件夹名）一致的可执行文件。Golang 1.8 之后支持 dep 依赖管理工具，对于空的项目使用 dep init 初始化依赖配置，其会生成 `Gopkg.toml Gopkg.lock vendor/` 这三个文件（夹）。

```go
package main

import "github.com/astaxie/beego"

func main() {
	beego.Run()
}
```

Go 并没有相对路径引入，而是以文件夹为单位定义模块，譬如我们新建名为 math 的文件夹，然后使用 `package math` 来声明该文件中函数所属的模块。外部引用该模块是需要使用工作区间或者 vendor 相对目录，其目录索引情况如下：

```sh
cannot find package "sub/math" in any of:
    ${PROJECTROOT}/vendor/sub/math (vendor tree)
    /usr/local/Cellar/go/1.10/libexec/src/sub/math (from $GOROOT)
    ${GOPATH}/src/sub/math (from $GOPATH)
```

# 表达式与控制流

## 变量声明与赋值

作为强类型静态语言，Go 允许我们在变量之后标识数据类型，也为我们提供了自动类型推导的功能。

```go
// 声明三个变量，皆为 bool 类型
var c, python, java bool

// 声明不同类型的变量，并且赋值
var i bool, j int = true, 2

// 复杂变量声明
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)

// 短声明变量
c, python, java := true, false, "no!"

// 声明常量
const constant = "This is a constant"
```

# Function: 函数

# 数据结构

## 基本数据类型

# 结构体与接口

## Pointer: 指针

```go
// p 是 Vertex 类型
p := Vertex{1, 2}  

// q 是指向 Vertex 的指针
q := &p

r := &Vertex{1, 2} // r is also a pointer to a Vertex

// The type of a pointer to a Vertex is *Vertex

var s *Vertex = new(Vertex) // new creates a pointer to a new struct instance
```

## Interface: 接口

Go 允许我们通过定义接口的方式来实现多态性，惯用的思路是先定义接口，再定义实现，最后定义使用的方法：

```go
package animals

type Animal interface {
	Speaks() string
}

// implementation of Animal
type Dog struct{}
func (a Dog) Speaks() string { return "woof" }

/****/

package circus

import "animals"

func Perform(a animal.Animal) { return a.Speaks() }
```

Go 也为我们提供了另一种接口的实现方案，我们可以不在具体的实现处定义接口，而是在需要用到该接口的地方，该模式为：

```go
func funcName(a INTERFACETYPE) CONCRETETYPE
```

定义接口：

```go
package animals

type Dog struct{}
func (a Dog) Speaks() string { return "woof" }

/****/
package circus

type Speaker interface {
	Speaks() string
}

func Perform(a Speaker) { return a.Speaks() }
```

# 并发编程

# Web 编程

# 开发实践

## 测试

VSCode 可以为函数自动生成基础测试用例，并且提供了方便的用例执行与调试的功能。

```go
/** 交换函数 */
func swap(x *int, y *int) {
	x, y = y, x
}

/** 自动生成的测试函数 */
func Test_swap(t *testing.T) {
	type args struct {
		x *int
		y *int
	}
	tests := []struct {
		name string
		args args
	}{
		// TODO: Add test cases.
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			swap(tt.args.x, tt.args.y)
		})
	}
}
```
