---
layout: post
title: Go 学习笔记
date: 2025-12-09 19:44:38+0800
last_updated: 2026-01-10 12:04:23+0800
description: 本文记录了我在学习 Go 语言过程中的一些笔记和思考。
tags:
  - 中文文章
categories: Go
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
| %v    | 默认格式                   |
| %+v   | 带字段名的结构体格式       |
| %#v   | Go 语法表示的值            |
| %T    | 类型信息                   |
| %t    | 布尔值                     |
| %c    | 对应的 Unicode 字符        |
| %U    | Unicode 格式的字符         |
| %%    | 字符 '%'                   |
| %s    | 字符串                     |
| %q    | 带双引号的字符串           |
| %p    | 指针地址                   |
| %b    | 二进制                     |
| %o/%O | 八进制（是否带 `0o` 前缀） |
| %x/%X | 十六进制（大小写）         |
| %d/%i | 十进制整数                 |
| %f    | 十进制浮点数               |
| %g    | 最简洁的十进制或科学计数法 |
| %e/%E | 科学计数法（大小写）       |
| %w    | 用于错误包装               |

---

在 `Go` 中的语句后面通常都不需要使用 `;` 来结尾，
并且在条件分支和循环分支中也不需要使用 `()` 来包裹条件表达式。
除此之外，在 `Go` 语言的条件分支中可以添加一条初始化语句，这条语句会在条件判断之前执行，
并且其作用域仅限于该条件分支内。例如：

```go
if err := doSomething(); err != nil {
  // 处理错误
} else {
  // 正常处理
}
// err 在这里不可见
```

`Go` 中的 `switch` 语句会自动在每个 `case` 分支后面添加一个隐式的 `break`，
因此不需要显式地使用 `break` 语句来终止分支。
如果想要在某个 `case` 分支中继续执行下一个分支，可以使用 `fallthrough` 关键字。

`fallthrough` 的作用是强制执行下一个 `case` 分支的代码，而不进行条件判断。
如果想在执行下一个 `case` 后继续执行下下个 `case`，
则需要在下一个 `case` 分支中再次使用 `fallthrough`。

---

`Go` 中可以使用 `type` 关键字来定义新类型或者给已有的类型取别名。

```go
type (
  MyInt int              // 定义新类型 MyInt，底层类型为 int
  YourInt = int          // 给 int 类型取别名 YourInt
)
```

别名类型和原类型是完全相同的类型，可以互相赋值和转换。
而新类型和原类型是不同的类型，往往需要进行显式的转换。

---

`Go` 中的 `var`、`const`、`type`、`import` 等关键字都可以使用块语法来声明多个变量、常量或类型。
块语法使用 `()` 将多个声明包裹在一起。例如：

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

`Go` 中可以定义 `label`，`label` 可以用来进行 `continue` 或者 `break` 操作，
从而跳出多层循环或者指定跳出某个循环。例如：

```go
OuterLoop:
for i := 0; i < 3; i++ {
  for j := 0; j < 3; j++ {
    if i == 1 && j == 1 {
      continue OuterLoop // 跳出当前内层循环，进入下一次外层循环
    }
    if i == 2 && j == 2 {
      break OuterLoop // 跳出外层循环
    }
    fmt.Printf("i=%d, j=%d\n", i, j)
  }
}
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

`Go` 语言中可以使用变长参数来接收不定数量的参数。
变长参数使用 `t ...T` 语法来定义，表示可以传入任意数量的该类型参数。
在函数体内，变长参数会被视为一个切片。
变长参数可以匹配多个类型为 `T` 的参数或者一个类型为 `[]T` 的参数，但是不能同时匹配两种形式：

```go
func sum(nums ...int) int {
  total := 0
  for _, num := range nums {
    total += num
  }
  return total
}
func main() {
  result1 := sum(1, 2, 3, 4)       // 传入多个 int 参数
  result2 := sum([]int{5, 6, 7}...) // 传入一个 []int 参数，注意要加上 ...
  // result3 := sum(1, 2, []int{3, 4}...) // 错误，不能同时传入多种形式的参数
  fmt.Println(result1) // 输出: 10
  fmt.Println(result2) // 输出: 18
}
```

不过上面的规则有个例外：在使用 `append` 将一个 `string` 变量追加到 `[]byte` 切片中的时候是可行的，
编译器会自动将 `string` 转换为 `[]byte`，然后再进行追加操作：

```go
var b []byte
b = append(b, "hello"...) // OK
```

---

`Go` 语言中可以使用 `import` 进行包的导入，导入时可以给导入的包取别名，
也可以使用 `.` 来导入包中的所有标识符。

通常而言，在 `Go` 导入的包必须要被使用，否则会导致编译错误。
但是可以给包取别名为 `_`，这种方式仅执行包的 `init` 函数。例如：

```go
import (
  fmt "fmt"      // 给包取别名 fmt
  . "math"       // 导入包中的所有标识符
  _ "net/http"   // 给包取别名 _，仅执行其 init 函数
)
```

当两个同名的包（通常指路径中的最后一段相同）导入时就需要使用别名来区分它们。

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

ch1 := '\u4e2d'     // Unicode 字符字面量，表示中文“中”
ch2 := '\U00004e2d' // Unicode 字符字面量，表示中文“中”
ch3 := '\x27'       // 字符字面量，表示单引号字符 `'`
ch4 := '\047'       // 字符字面量，八进制表示的字符 `'`

s1 := "abc\n"                // 字符串字面量，包含转义字符
s2 := "\u4e2d\u6587"         // 字符串字面量，表示“中文”
s3 := "\U00004e2d\U00006587" // 字符串字面量，表示“中文”
s4 := `This is a raw string.
This is the second line.
This is the third line.
\n will not be interpreted.` // 原始字符串字面量

arr1 := [6]int { 0, 1, 2, 3, 4, 5 }
arr2 := [...]int { 0, 1, 2, 3, 4, 5 }
arr3 := [...]int { 5: 5 } // 数组字面量，长度均为 6

sp1 := []int { 0, 1, 2 } // 切片字面量

mp1 := map[string]int { "a": 1, "b": 2 } // map 字面量
```

在使用十六进制的科学计数法进行表示的时候，`p` 和 `P` 表示幂运算的底数是 `2`。
需要注意的是有效数字的整数部分和小数部分用十六进制来表示，而指数部分用十进制来表示。

如果想要在 `raw string` 中包含反引号，则可以使用 `+` 进行字符串的拼接来实现。

---

`Go` 语言中可以使用 `array[low:high:max]` 来基于一个已经存在的数组创建一个切片。
当省略 `max` 时，默认 `max` 的值为数组的长度。
这个切片的长度是 `high - low`，容量是 `max - low`。
这也表明 `[low, high)` 和 `[low, max)` 都是左闭右开区间。

需要注意的是基于数组创建的切片的底层是原数组，所以对切片的修改往往会直接影响原数组。
不过在切片发生扩容的时候会创建一个新的底层数组，此时对切片的修改就不会影响原数组了。

```go
arr := [5]int{0, 1, 2, 3, 4}
s1 := arr[1:4:5]              // 创建切片 s1，包含 arr[1], arr[2], arr[3]
fmt.Println(s1)               // 输出: [1 2 3]
fmt.Println(len(s1), cap(s1)) // 输出: 3 4
s1[0] = 10                    // 修改切片 s1
fmt.Println(arr)              // 输出: [0 10 2 3 4]，arr 也被修改了
s2 := append(s1, 20, 30)      // 切片 s1 发生扩容，创建了新的底层数组
s2[0] = 30
fmt.Println(arr) // 输出: [0 10 2 3 4]，arr 不受影响
fmt.Println(s2)  // 输出: [30 2 3 20 30]
```

可以使用 `len` 和 `cap` 函数来获取切片的长度和容量。
但是对于 `map` 而言则只能使用 `len` 函数来获取其长度。

---

`Go` 语言中通过下标运算符去获取一个 `map` 中不存在的键时会返回该类型的零值。
为了区分一个键是不存在还是其值就是类型的零值，可以使用双赋值的形式来获取键对应的值和一个布尔值，
该布尔值表示该键是否存在于 `map` 中。

```go
m := make(map[string]int)
v, ok = m["key"] // 如果 "key" 不存在，v 为 0，ok 为 false
if !ok {
// 处理键不存在的情况
} else {
// 使用 v 进行后续操作
}
```

`Go` 语言的 `map` 是基于 `hash` 的，对其进行遍历时的顺序是随机的。
`Go` 为了让开发者不依赖于 `map` 的遍历顺序，特意设计成每次遍历的顺序都有可能不一样。

---

`Go` 语言中的字符串、字节切片、`rune` 切片之间可以方便地进行相互转换。

```go
var s string = "Hello 世界"
var b []byte = []byte(s) // 字符串转换为字节切片
var r []rune = []rune(s) // 字符串转换为 rune 切片
s2 := string(b)          // 字节切片转换为字符串
s3 := string(r)          // rune 切片转换为字符串
```

---

`Go` 语言中的字符串是通过 `UTF-8` 编码进行存储的，因此可以直接存储和处理多字节的 `Unicode` 字符。

`Go` 语言中的 `len` 获取的字符串的字节数而不是字符数，如果要获取字符数可以使用
`utf8.RuneCountInString` 函数。同理通过下标访问字符串时获取的是字节而不是字符。
不过 `for range` 遍历时获取的是字符。

```go
var s string = "Hello 世界"
lengthInBytes := len(s)                    // 获取字符串的字节数
lengthInRunes := utf8.RuneCountInString(s) // 获取字符串的字符数
for i, r := range s {                      // 遍历字符串中的字符
  fmt.Printf("Character %d: %c\n", i, r)
  // i 是字符的起始字节索引，r 是对应的 rune 值
  // Character 5: 世
  // Character 8: 界
}
for i := 0; i < len(s); i++ {
  fmt.Printf("Byte %d: %x\n", i, s[i])      // 访问字符串中的字节
}
```

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

嵌入按照嵌入类型可以分为以下几种：

- 接口中嵌入接口（只能是 `I`）
- 结构体中嵌入结构体（可以是 `T` 或 `*T`）
- 结构体中嵌入接口（只能是 `I`）

这里重点介绍一下在一个结构体中嵌入接口的情况。
在结构体中嵌入接口后，这个结构体类型就实现了该接口。
但是我们必须在使用接口中的方法前，为嵌入的接口字段赋值，否则会导致运行时错误：

```go
type I interface {
  M1()
  M2()
}
type S struct{}
func (s S) M1() {
  println("M1 called")
}
func (s S) M2() {
  println("M2 called")
}
type T struct {
  I // T 中嵌入接口 I
}
func main() {
  t := T{
    I: S{}, // 为嵌入的接口字段赋值，否则会导致运行时错误
  }
  t.M1()
  t.M2()
}
```

我们也可以自己在 `T` 中实现接口 `I` 的方法，当接口变量被赋值且 `T` 实现了接口的方法时，
`T` 实现的方法会优先被调用：

```go
// ...
func (t T) M1() {
  println("T's M1 called")
}
func main() {
  t := T{
    I: S{},
  }
  t.M1() // 调用 T 实现的 M1 方法
  t.M2() // 调用 S 实现的 M2 方法
}
```

在结构体内嵌入多个接口时，如果这些接口中有同名的方法：

- 若同名方法的签名不同，则编译时会报错，提示方法冲突。
- 若同名方法的签名相同，则必须在结构体中实现该方法，否则调用时编译器会报错。

```go
type I1 interface {
  M()
  M1()
}
type I2 interface {
  M()
  M2()
}
type T struct {
  I1
  I2
}
func (t T) M() { println("T.M") } // 必须实现 M 方法，否则 t.M() 会导致编译错误
func main() {
  t := T{}
  t.M()
}
```

---

`Go` 语言中不可以在结构类型 `T` 中定义类型为 `T` 的字段，也不可以递归定义，
但是可以包含`*T`、`[]T`、`map[type]T` 等类型的字段。

```go
type T struct {
  // F T         // 错误，不能包含类型为 T 的字段
  F *T           // 正确，可以包含类型为 *T 的字段
  G []T          // 正确，可以包含类型为 []T 的字段
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

`Go` 语言中主要有多种方法可以对自定义类型进行初始化。以下面的自定义类型为例。

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

不过官方推荐使用一种名为 `WithOption` 的设计模式来进行复杂类型的初始化，
这种方式可以通过传入不同的选项函数来灵活地配置初始化参数。 例如：

```go
type Person struct {
  Name string
  Age  int
}
type PersonOption func(*Person)
func WithName(name string) PersonOption {
  return func(p *Person) {
    p.Name = name
  }
}
func WithAge(age int) PersonOption {
  return func(p *Person) {
    p.Age = age
  }
}
func NewPerson(opts ...PersonOption) *Person {
  p := &Person{
    Name: "Unknown",
    Age:  0, // 默认值
  }
  for _, opt := range opts {
    opt(p)
  }
  return p
}
p4 := NewPerson(WithName("Alice"), WithAge(30))
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

`Go` 语言中不能给接口或者指针类型定义方法：

```go
type MyInt *int
// CE: invalid receiver type MyInt (MyInt is a pointer type)
func (r MyInt) IsNil() bool { return r == nil }

type MyReader io.Reader
// CE: invalid receiver type MyReader (MyReader is an interface type)
func (r MyReader) Read(p []byte) (n int, err error) {}
```

---

`Go` 语言对接口的实现是通过 `iface` 和 `eface` 两种内部数据结构来实现的。
`iface` 用于存储非空接口类型的变量，而 `eface` 用于存储空接口类型的变量。

空接口是指接口中没有任何方法的接口类型，表示可以存储任意类型的值。
`Go` 语言中的 `any` 类型实际上就是空接口类型的别名。
非空接口是指除了空接口之外的所有接口类型。

在 `eface` 内部，包含了两个字段：

- `_type`：表示具体的类型信息。
- `data`：表示具体的值。

而在 `iface` 内部，包含了两个字段：

- `tab`：表示类型信息和方法集合的表。
- `data`：表示具体的值。

接口的比较是基于这两个字段的。

不管是空接口还是非空接口只有当其两个字段均为 `nil` 时才表示接口变量为 `nil`：

```go
var a1 any
var a2 any = nil
var e1 error
var e2 error = nil
```

上面的四个变量和 `nil` 进行比较时均为 `true`。
因为上面的四个变量都为 `nil` 所以他们在通过 `==` 互相比较时都会返回 `true`。

一旦两个字段中的任意一个不为 `nil`，那么接口变量就不再等于 `nil`：

```go
var i *int = nil
var m *MyError = nil
var a any = i
var e error = m
```

上面例子中的 `a` 和 `e` 都不等于 `nil`，因为他们的 `data` 字段虽然为 `nil`，
但是 `_type`（或 `tab` ）字段不为 `nil`。

在接口变量之间进行比较时：

- 当比较的两个接口变量是空接口（`eface`）或两个接口变量是非空接口时，
当且仅当其 `_type`（或 `tab`）字段相等、`data` 字段指向的数据内容一致时才相等。
- 当比较的两个接口变量一个是空接口（`eface`），另一个是非空接口（`iface`）时，
当且仅当空接口的 `_type` 字段等于非空接口的 `tab` 字段中的 `_type`字段、
`data` 字段指向的数据内容一致时才相等。

这里的 `data` 字段指向的数据内容一致指的是：

- 如果接口与指针类型绑定，那么比较的是指针的值是否相等。
- 如果接口与非指针类型绑定，那么比较的是值的内容是否相等。

**NOTE**：`iface` 代表的是 `interface` 的意思，而 `eface` 代表的是 `empty interface` 的意思。

---

`Go` 语言中的方法集合是指某个类型所拥有的方法的集合。
其主要作用是用于判断一个类型是否实现了某个接口。
类型 `T` 的方法集合是由所有接收者为 `T` 的方法组成的集合。
类型 `*T` 的方法集合是由所有接收者为 `T` 或 `*T` 的方法组成的集合。
如果一个类型的方法集合是一个接口的超集，那么该类型就实现了该接口。

这里解释一下为什么会是这样的定义。
关键在于 `Go` 语言中的接口的实现方式。`Go` 语言的接口中保存了两部分内容：

* 具体的类型信息
* 对应的实例数据

当一个非指针类型的变量被赋值给一个接口变量的时候，这个变量的实例数据是原对象的拷贝。
而将一个指针类型的变量赋值给一个接口变量的时候，接口变量数据部分保存的是该指针的值。
这也就意味着，如果真的可以通过一个非指针变量去调用接收者为指针的方法的话，
那么这个方法中对接收者的任何修改都不会反应到原对象上。为了禁止这样反直觉的行为，
`Go` 语言就采用了上面所说的方法集合的定义。

```go
type Animal interface {
  Grow() string
}
type Dog struct {}
func (d *Dog) Grow() {
 // some code will change Dog instance
}

func main() {
  var a Animal
  d := Dog{}
  a = d
  a.Grow() // 假设这里可以通过编译，d 的状态也不会改变
}
```

如果理解了上述的规则我们不难得出嵌套类型的方法集合：

* 如果在类型 `T` 中嵌入了类型 `U`，那么类型 `T` 的方法集合包含了类型 `U` 的方法集合。
* 如果在类型 `T` 中嵌入了类型 `*U`，那么类型 `T` 的方法集合包含了类型 `*U` 的方法集合。
* 不论嵌入类型是 `U` 还是 `*U`，类型 `*T` 的方法集合都包含了 `*U` 的方法集合。

需要注意的是一个变量的类型对应的方法集合中没有的方法并不意味着该变量不能调用。
对于可寻址的对象，`Go` 语言会自动将其取地址从而调用接收者为指针的方法。
同样地，对于指针类型的变量，`Go` 语言也会自动解引用从而调用接收者为非指针的方法。

当我们使用 `type` 给一个变量取别名或者基于已有的类型定义新的类型时，方法集合会根据原类型而有所不同。

* 取别名不会改变原类型的方法集合；
* 基于接口类型创建的新类型，其方法集合与原接口类型一致；
* 而基于非接口类型创建的新类型，其方法集合为空。

---

`Go` 语言中，`map` 存储的元素是不可寻址的，而切片中的元素确是可寻址的。
关于这样设计的原因主要有两个：

* 对于 `map` 而言，其底层实现是基于哈希表的，
  如果允许对 `map` 中的元素进行寻址，那么在 `map` 发生扩容或者重新哈希时，
  这些寻址的指针就会变得无效，从而导致不可预期的行为，这里应该主要是考虑红黑树上节点变化的问题。
* 对于切片而言，其底层实现是基于数组的，数组的元素在内存中是连续存储的，
  因此允许对切片中的元素进行寻址是安全且高效的；
  且即使切片在使用过程中发生了扩容，如果此时有指针指向原数组上的元素，原数组就不会被释放。
* 在 `map` 中，如果一个值不存在，那么通过下标运算符获取该值时会返回该类型的零值。
  如果允许对 `map` 中的元素进行寻址，那么对于不存在的键，其对应的值将无法寻址，
  这会导致代码变得复杂且容易出错。

正因为 `map` 不可寻址，如果 `map` 存储的 `value` 部分是非指针，我们是不能调用接收者为指针的方法的。

---

`Go` 语言中可以通过类型断言来获取一个接口变量的具体类型和值。

```go
a := 10
var x any = a
v1, ok1 := x.(int)    // ok1 为 true，v1 的值为 10
v2, ok2 := x.(string) // ok2 为 false，v2 的值为 string 类型的零值 ""
v3 := x.(float64)     // 如果断言失败会引发 panic
```

需要注意的是，如果断言的类型是一个接口则语义变成了“变量是否实现了该接口”的判断。
如果断言成功，返回值的类型为实际类型而不是所实现的接口类型。

---

在对内置的函数进行 `defer` 操作的时候，
`append`、`cap`、`len`、`make`、`new` 等并不能作为 `deferred` 函数调用。

---

`Go` 中只能通过 `make` 来创建 `channel`，`make` 接收两个参数，
第一个是 `channel` 的类型，第二个是 `channel` 的缓冲区大小。
缓冲区大小默认为 `0`，表示无缓冲 `channel`。

```go
ch := make(chan int)       // 无缓冲 channel
chBuf := make(chan int, 5) // 有缓冲 channel，缓冲区大小为 5
```

在使用无缓冲 `channel` 的时候，发送方和接收方一定要放在两个不同的 `goroutine` 中，
这是因为如果放在同一个 `goroutine` 中，无缓冲 `channel` 在发送和接收时都会阻塞当前的 `goroutine`，
进而引发协程泄漏。

`Go` 语言中使用 `channel` 的时候往往是由发送方来关闭，
这是因为接收主有安全的手段来检查 `channel` 是否已经关闭，而发送方并没有这样安全的手段。

```go
n := <-ch     // 当 channel 关闭时，n 会被赋值为类型的零值
m, ok := <-ch // ok 为 false 表示 channel 已经关闭
for v := range ch {
  // 当 channel 关闭时，循环会自动结束
}
```

---

`Go` 语言中可以使用 `select` 原语，其可以一次监听多个 `channel` 的操作。
当其中某个 `channel` 准备好进行发送或接收操作时，`select` 会执行对应的 `case` 分支。

```go
select {
case msg1 := <-ch1:
  fmt.Printf("Received message from ch1: %s\n", msg1)
case ch2 <- msg2:
  fmt.Printf("Sent message to ch2: %s\n", msg2)
default:
  fmt.Println("No channel is ready")
}
```

当没有使用 `default` 的时候，`select` 会一直阻塞直到某个 `case` 分支可以执行。

下面三种是 `select` 原语常用的方式：

- 使用 `default` 分支可以实现 `try` 语义。
- 配合 `time` 包可以实现超时控制。
- 配合 `time` 包的 `Ticker` 可以实现周期任务。

```go
fun TrySend(ch chan<- int, value int) bool {
  select {
  case ch <- value:
    return true // 发送成功
  default:
    return false // channel 未准备好，发送失败
  }
}

func ReceiveWithTimeout(ch <-chan int, timeout time.Duration) (int, error) {
  select {
  case value := <-ch:
    return value, nil // 成功接收数据
  case <-time.After(timeout):
    return 0, errors.New("receive timeout") // 超时
  }
}

func PeriodicTask(interval time.Duration, stopCh <-chan struct{}) {
  ticker := time.NewTicker(interval)
  defer ticker.Stop()
  for {
    select {
    case <-ticker.C:
      // 执行周期任务
    case <-stopCh:
      return // 停止任务
    }
  }
}
```

有一点需要注意的是，在实现超时控制的时候，如果使用无缓冲 `channel` 则可能出现协程泄漏的问题：

```go
ch := make(chan struct{})
go func() {
  // do some work
  ch <- struct{}
}()
select {
case <-ch:
  fmt.Println("任务完成！")
case <-time.After(2 * time.Second):
  fmt.Println("任务超时!")
}
```

在上面的代码中，如果任务在 `2` 秒内没有完成，
那么超时分支会被执行，而任务协程仍然会继续运行并尝试向 `ch` 发送数据。
由于 `ch` 是一个无缓冲的 `channel`，如果没有其他协程在接收数据，
那么任务协程会一直阻塞在发送操作上，导致协程泄漏。
为了解决这个问题，可以使用有缓冲的 `channel` 或者使用 `context` 进行超时控制。

---

`Go` 语言中可以使用 `type switch` 来方便的判断一个接口变量所属于的类型。

```go
var x interface{} = 10
switch v := x.(type) { // 只能接口类型可以使用 type switch
case int:
  fmt.Printf("x is int: %d\n", v)
case string:
  fmt.Printf("x is string: %s\n", v)
default:
  fmt.Printf("x is of unknown type\n")
}
```

注意 `case` 后面的类型必须是实现了该接口的类型，否则会导致编译错误。

---

`Go` 语言的泛型不支持在类型里面内嵌泛型本身，也不支持在泛型方法中接着定义泛型。

```go
type MyType[T any] struct {
  // T          // 错误，不能在泛型类型中内嵌泛型本身
}
// 错误，不能在泛型方法中接着定义泛型
func (m MyType[T]) MyMethod[U any]() {
}
```

---

`Go` 语言的类型约束在有些情况下可以简写，例如：

```go
func a1[I interface { int | int32 | ~int64 }](param I) {}
func a2[T int | int32 | ~int64](param T) {}
```

**注意**：在类型约束中如果使用 `~` 则表示只要底层类型是该类型即可，
而不使用 `~` 则表示必须是该类型本身。

---

`Go` 语言中 `panic` 表示程序发生了不可恢复的错误，通常会导致程序崩溃。
任意一个 `goroutine` 中发生的 `panic` 都会导致整个程序崩溃。
可以使用 `recover` 函数来捕获 `panic`，从而防止程序崩溃。
而 `recover` 函数只能在 `defer` 函数中调用。

```go
func safeFunction() {
  defer func() {
    if r := recover(); r != nil {
      fmt.Println("Recovered from panic:", r)
    }
  }()
  // 可能引发 panic 的代码
}
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

---

`Go` 中命名的一些规范：

- 循环和条件变量多采用单个字母命名。
- 函数/方法的参数和返回值以单个单词或字母为主。
- 方法的命名以单个单词为主。
- 函数/类型多以多个单词的复合形式命名。
- 变量中不携带类型信息。
- 包名往往由单个单词进行命名，且尽量与导入路径的最后一个路径分段一致。
- `Go` 中如果接口类型只有一个方法，则接口往往命名为该方法名加上 `-er` 后缀。
  比如 `Read` 方法对应的接口名为 `Reader`，`Write` 方法对应的接口名为 `Writer`。

---

在 `Go` 语言中零值可用是非常重要的概念，比如对于一个指向 `net.TCPAddr` 的指针，
如果我们使用 `fmt.Print` 对其进行输出则会调用 `func (*TCPAddr) String() string` 方法，
在该方法中，其检查了调用的实例是不是一个空指针，并对于空指针直接返回了 `<nil>` 字符串。

---

`Go` 语言中的参数传递是值传递的，对于一个数组参数其在进行参数传递的时候会对整个数组进行拷贝。
而因为切片、字典等对象实际存储的是指针，所以其在作为参数进行传递时开销会小很多。
除此之外，`Go` 语言中 `for i, v := range x` 中的 `v` 变量也会有一次拷贝，
即使 `x` 的类型是切片这样的类型，修改 `v` 也不会作用在原切片上。

```go
r := []int{0, 1, 2}
// CE: declared and not used: v
for _, v := range r {
  v = 0
}
```

上面的例子中我们尝试通过修改 `v` 来实现修改 `r` 的目的，但是实际上是不可能的，
因为我们只是对 `v` 进行了赋值，`Go` 编译器认为我们没有使用 `v`。

```go
var a = [5]int{1, 2, 3, 4, 5}
var r [5]int
for i, v := range a {
// for i, v := range &a {
// for i, v := range a[:] {
  if i == 0 {
    // 注意这里我们改的是还没有遍历到的数据
    a[1] = 12
    a[2] = 13
    // 不管是使用哪种形式的 for range
    // 对当前遍历对象的修改都不会作用到 v 上
    // 因为 v 已经是 a[0] 的拷贝了
    // a[0] = 10
  }
  r[i] = v
}
fmt.Println("r = ", r) // r = [1 2 3 4 5]
fmt.Println("a = ", a) // a = [1 12 13 4 5]
```

由于在调用 `range` 的时候会将 `a` 进行一次拷贝，
即使我们第一次进行循环的时候修改了 `a[1]` 和 `a[2]`，
我们也不能看到修改的值。

而如果我们使用 `for i, v := range &a` 拷贝的就是指针，此时编译器会帮我们解引用而因为是同一个地址，
所以 `v` 可以看到这样的影响。

同理，当我们使用 `for i, v := range a[:]` 拷贝的就是切片，而切片的底层数组是同一个，
所以 `v` 也可以看到这样的影响。

对于 `map` 而言，如果在 `for` 过程中添加或者删除键值，则循环的次数是不确定的。
而对于切片而言，其循环次数在一开始就已经确定了。

对于 `channel` 而言，`for range` 只有在 `channel` 被关闭后才会结束循环。
如果 `channel` 的变量是 `nil` 则循环将会永远被阻塞。

在 `go 1.22` 版本之前，`for` 循环中变量只会被创建一次，
这会导致一些奇怪的现象：

```go
type Customer struct {
  ID      string
  Balance float64
}
type Store struct {
  Customers map[string]*Customer
}

func (s *Store) storeCustomers(customers []Customer) {
  for _, customer := range customers {
    // 在 go 1.22 之前需要这样写以避免问题，保证每次创建一个新的 customer 变量
    // customer := customer
    s.Customers[customer.ID] = &customer
  }
}

func main() {
  s := Store{Customers: make(map[string]*Customer)}
  s.storeCustomers([]Customer{
    {ID: "1", Balance: 10},
    {ID: "2", Balance: -10},
    {ID: "3", Balance: 0},
  })
  for i, v := range s.Customers {
    fmt.Printf("id=%s,customer=%+v\n", i, v)
  }
}
```

上面的代码在 `go 1.22` 之前会输出：

```text
id=1, customer=&main.Customer{ID:"3", Balance:0}
id=2, customer=&main.Customer{ID:"3", Balance:0}
id=3, customer=&main.Customer{ID:"3", Balance:0}
```

这是因为在 `for range` 循环中变量 `customer` 只会被创建一次，
所以三次赋值都指向了同一个地址，从而导致最后的结果都是相同的。

在 `go 1.22` 及之后的版本中，`for range` 循环中变量 `v` 会在每次循环时创建新的变量，
从而避免了上述的问题。

---

`Go` 语言中除了常见的方法调用方式外，还可以通过 `Method Expression` 进行调用：

```go
type T struct{}
func (t T) Get() {}
func (t *T) Set(value int) {}
var t T
t.Get()
t.Set(10)
// or
T.Get(t)
(*T).Set(&t, 10)
```

除了 `Method Expression` 以外，还可以使用 `Method Value` 进行调用：

```go
type T struct{}
func (t T) Get() {}
func (t *T) Set(value int) {}
var t T
getFunc := t.Get
setFunc := (&t).Set
getFunc()   // 等价于 t.Get()
setFunc(10) // 等价于 t.Set(10)
```

---

`Go` 语言中的 `select` 的执行过程分为求值阶段和选择阶段。

在求值阶段，`select` 会在进入后按照从上到下、从左到右进行求值：

```go
select {
case getAChannel() <- computeValue1():           // getAChannel() 和 computeValue1() 会被调用
case (getAStorageArray())[0] := <-getAChannel(): // getAStorageArray() 不会被调用，getAChannel() 会被调用
}
```

在上面的代码中，会依次执行 `<-` 左右两部分，但是在赋值语句中，赋值号左边的表达式不会被执行，
只有在其被选中时才会执行。
