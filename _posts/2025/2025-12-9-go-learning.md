---
layout: post
title: Go 学习笔记
date: 2025-12-09 19:44:38+0800
last_updated: 2025-12-10 20:01:31+0800
description: 本文记录了我在学习 Go 语言过程中的一些笔记和心得。
tags:
  - Go
categories: Potpourri
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

使用 `fmt.Printf` 时可能涉及到相关的格式化动词，以下是一些常用的格式化动词及其含义：

| 动词  | 含义                       |
| ----- | -------------------------- |
| %T    | 值的类型                   |
| %t    | 布尔值                     |
| %%    | 字符 '%'                   |
| %s    | 字符串                     |
| %p    | 指针地址                   |
| %b    | 二进制                     |
| %o/%O | 八进制（是否带 `0o` 前缀） |
| %x/%X | 十六进制（大小写）         |
| %d/%i | 十进制整数                 |
| %f    | 十进制浮点数               |
| %e/%E | 科学计数法（大小写）       |

---

在 `Go` 中所以的语句后面都不需要使用 `;` 来结尾，
并且在条件分支中也不需要使用 `()` 来包裹条件表达式。
除此之外，在 `Go` 语言的条件分支中可以添加一个初始化语句，这个初始化语句会在条件判断之前执行，
并且其作用域仅限于该条件分支内。例如：

```go
if err := doSomething(); err != nil {
  // 处理错误
} else {
  // 正常处理
}
```

`Go` 中的 `switch` 语句会自动在每个 `case` 分支后面添加一个隐式的 `break`，
因此不需要显式地使用 `break` 语句来终止分支。

---

`Go` 中的 `var`、`const`、`type`、`import` 等关键字都可以使用块语法来声明多个变量、常量或类型。
块语法使用大括号 `{}` 将多个声明包裹在一起。例如：

```go
var (
  a int
  b string
  c bool
)
```

这种方式可以使代码更加整洁，尤其是在需要声明多个相关变量时。

---

在 `Go` 语言的 `const` 块中，后续的变量会重复使用前一个变量的表达式，除非显式地为其赋值。
例如：

```go
const (
  A = 1
  B        // B 的值为 1
  C = 2
  D        // D 的值为 2
)
```

---

`Go` 语言的 `const` 块中可以使用 `iota`，它的值是当前变量所在的偏移位置（从0开始计算）。
每当遇到一个新的 `const` 块时，`iota` 会被重置为0，并且在每一行中递增1。
也可以在 `const` 块中使用 `_` 来忽略某些值。
例如：

```go
const (
  A = iota           // A 的值为 0
  B                  // B 的值为 1
  _                  // 忽略该值，iota 递增到 2
  C, D = iota, iota  // C 和 D 的值均为 3
)
```

---

`Go` 中可以使用有显示名称的返回值，
这样只需要在函数体中对这些返回值进行赋值而不需要显式地使用 `return` 语句返回它们。
这样的写法在需要根据条件分支返回多个值时非常有用。
例如：

```go
// 使用有显示名称的返回值
func divide(a, b int) (quotient int, remainder int) {
  quotient = a / b
  remainder = a % b
  return // 直接返回命名的返回值，这里的 return 不能省略
}

// 使用无名称的返回值
func divide2(a, b int) (int, int) {
  return a / b, a % b
}
```

---

`Go` 语言中可以使用 `import` 进行包的导入，可以给导入的包取别名，
也可以使用 `.` 来导入包中的所有标识符。通常而言，在 `Go` 导入的包必须要被使用，否则会导致编译错误。
但是有一种特殊的导入方式是使用 `_`，这种方式仅导入包以执行其 `init` 函数，
而不会导入包中的其他标识符。
例如：

```go
import (
  fmt "fmt"      // 给包取别名
  . "math"       // 导入包中的所有标识符
  _ "net/http"  // 仅导入包以执行其 init 函数
)
```

当两个同名的包导入时就需要使用别名来区分它们。

---

`Go` 中有各种字面量可以使用，同时对于数字字面量还支持使用下划线 `_` 来分隔数字以提高可读性。
例如：

```go
a := 53_700        // 十进制
b := 0_700         // 0前缀表示八进制
c1 := 0x_aa_bb_cc
c2 := 0X_dd_ee_ff  // 0x或0X前缀表示十六进制
d1 := 0b_1000_0001
d2 := 0B_1000_0001 // 0b或0B前缀表示二进制
e := .15           // 浮点数，可以省略整数部分的0
f := 82.           // 浮点数，可以省略小数部分的0
g1 := 1.5e2        // 科学计数法表示
g2 := 1.5E3        // 科学计数法表示
h1 := 0x2.p10      // 十六进制浮点数
h2 := 0X1.Fp0      // 十六进制浮点数
```

在使用十六进制的科学计数法进行表示的时候，`p` 和 `P` 表示幂运算的底数是 `2`。
需要注意的是整数部分和小数部分用十六进制来表示，而幂运算的指数部分仍然使用十进制来表示。

---

`Go` 语言中支持 `raw string` 只需要使用反引号进行包裹即可。 `raw string` 中的内容会原样输出，
不会对其中的转义字符进行处理。例如：

```go
var s string = `This is a raw string.
This is the second line.
This is the third line. \n will not be interpreted.`
```

如果想要在 `raw string` 中包含反引号，则可以使用 `+` 进行字符串的拼接来实现。

---

`Go` 语言的结构体中可以使用嵌入字段来实现类似于继承的效果。
嵌入字段是指在结构体中直接包含另一个结构体类型，而不需要为其指定字段名。
这样可以直接访问嵌入结构体的字段和方法。例如：

```go
type Person struct {
  Name string
  Age  int
}

type Employee struct {
  Person  // 嵌入 Person 结构体
  ID      string
}

var emp Employee
emp.Name = "Alice"
emp.Age = 30  // 直接访问嵌入结构体的字段
```

`Go` 语言中不可以在结构类型 `T` 中类型为 `T` 的字段，也不可以递归定义，
但是可以包含`*T`、`[]T`、`map[type]T` 等类型的字段。

```go
type T struct {
  // F T          // 错误，不能包含类型为 T 的字段
  F *T          // 正确，可以包含类型为 *T 的字段
  G []T         // 正确，可以包含类型为 []T 的字段
  H map[string]T // 正确，可以包含类型为 map[type]T 的字段
}

type T1 struct {
  t2 T2
}
type T2 struct {
  t1 T1  // 错误，不能递归定义
}
```

---

`Go` 语言中主要有三种方法可以对自定义类型进行初始化。以下面的自定义类型为例。

```go
type Person struct {
  Name string
  Age  int
}
type Book struct {
  Title  string
  Author Person
}
```

一是可以按照顺序对结构体的字段进行赋值，例如：

```go
p1 := Person{"Alice", 30}
b1 := Book{"Go Programming", Person{"Bob", 40}}
```

但是这样的弊端也很明显：如果结构体的字段顺序发生变化，那么初始化的代码也需要进行相应的修改。
且当结构体的字段较多时，代码的可读性也会变差。这就引出了第二种初始化的方式：使用字段名进行赋值。

```go
p2 := Person{Name: "Alice", Age: 30}
b2 := Book{Title: "Go Programming", Author: Person{Name: "Bob", Age: 40}}
```

除此之外还可以自定义方法来根据传入的参数进行初始化，例如：

```go
func NewPerson(name string, age int) *Person {
  return &Person{Name: name, Age: age}
}
// or
// func NewPerson(name string, age int) Person {
//   return Person{Name: name, Age: age}
// }

p3 := NewPerson("Alice", 30) // p3 在这里是 *Person 类型
```

---

`Go` 语言可以对底层类型相同的元素进行隐式转换，编译器会保证这种转换的安全。

```go
type MyInt int

var a MyInt = 10
b := 10
c := a + b // 隐式转换，MyInt 和 int 可以进行运算
```

---

`Go` 语言中的包导入时可以在结尾增加版本信息，例如：

```go
import "github.com/user/project/v2"
```

`Go` 语言的包符合 `Major.Minor.Patch` 版本规范，官方规定只有当 `Major` 变化时才会出现兼容性的问题。
在不书写版本号时，默认导入的是 `v0` 或 `v1` 版本的包。可以使用 `go list -m -versions <package_name>`
来查看某个包的所有可用版本。

如果想要移除一个依赖，需要使用 `go get <package_name>` 的形式在版本部分添加 `@none`，
例如 `go get github.com/go-redis/redis/v8@none` 会移除已经添加的 `go-redis v8` 依赖。
