---
layout: post
title: Go 学习笔记
date: 2025-12-09 19:44:38+0800
last_updated: 2025-12-12 16:15:55+0800
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
如果想要在某个 `case` 分支中继续执行下一个分支，可以使用 `fallthrough` 关键字。

---

`Go` 中可以使用 `type` 关键字来定义新类型或者给已有的类型取别名。

```go
type (
  MyInt int               // 定义新类型 MyInt，底层类型为 int
  YourInt = int          // 给 int 类型取别名 YourInt
)
```

别名类型和原类型是完全相同的类型，可以互相赋值和转换。
而新类型和原类型是不同的类型，往往需要进行显式的转换。

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
变长参数使用 `...` 语法来定义，表示可以传入任意数量的该类型参数。
在函数体内，变长参数会被视为一个切片。例如：

```go
func sum(nums ...int) int {
  total := 0
  for _, num := range nums {
    total += num
  }
  return total
}
s := []int{1, 2, 3}
result1 := sum(1, 2, 3, 4, 5) // 传入多个参数
result2 := sum(s...)          // 传入切片，使用 ... 展开切片
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

ch1 := '\u4e2d'              // Unicode 字符字面量，表示中文“中”
ch2 := '\U00004e2d'          // Unicode 字符字面量，表示中文“中”
ch3 := '\x27'                // 字符字面量，表示单引号字符 `'`
ch4 := '\047'                // 字符字面量，八进制表示的字符 `'`

s1 := "abc\n"                // 字符串字面量，包含转义字符
s2 := "\u4e2d\u6587"         // 字符串字面量，表示“中文”
s3 := "\U00004e2d\U00006587" // 字符串字面量，表示“中文”
s4 := `This is a raw string.
This is the second line.
This is the third line.
\n will not be interpreted.` // 原始字符串字面量

arr1 := [6]int { 0, 1, 2, 3, 4, 5 }
arr2 := [...]int { 0, 1, 2, 3, 4, 5 }
arr3 := [...]int { 5: 5 }   // 数组字面量，长度均为 6

sp1 := []int { 0, 1, 2 }    // 切片字面量

mp1 := map[string]int { "a": 1, "b": 2 } // 映射字面量
```

在使用十六进制的科学计数法进行表示的时候，`p` 和 `P` 表示幂运算的底数是 `2`。
需要注意的是整数部分和小数部分用十六进制来表示，而幂运算的指数部分仍然使用十进制来表示。

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
var b []byte = []byte(s)       // 字符串转换为字节切片
var r []rune = []rune(s)       // 字符串转换为 rune 切片
s2 := string(b)                // 字节切片转换为字符串
s3 := string(r)                // rune 切片转换为字符串
```

---

`Go` 语言中的字符串是通过 `UTF-8` 编码进行存储的，因此可以直接存储和处理多字节的 `Unicode` 字符。

`Go` 语言中的 `len` 获取的字符串的字节数而不是字符数，如果要获取字符数可以使用
`utf8.RuneCountInString` 函数。同理通过下标访问字符串时获取的是字节而不是字符。
不过如果是使用 `for range` 来遍历字符串时获取的是字符。

```go
var s string = "Hello 世界"
lengthInBytes := len(s)                      // 获取字符串的字节数
lengthInRunes := utf8.RuneCountInString(s)   // 获取字符串的字符数
for i, r := range s {                        // 遍历字符串中的字符
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

`Go` 语言中不可以在结构类型 `T` 中类型为 `T` 的字段，也不可以递归定义，
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

`Go` 语言中的方法集合是指某个类型所拥有的方法的集合。 方法集合会根据接收者的类型而有所不同。
如果接收者是值类型，那么方法集合中包含所有值接收者和指针接收者的方法。
如果接收者是指针类型，那么方法集合中只包含指针接收者的方法。
如果一个类型的方法集合是一接口的超集，那么该类型就实现了该接口。

---

`Go` 语言中可以通过类型断言来获取一个接口变量的具体类型和值。

```go
a := 10
var x any = a
v1, ok1 := x.(int)    // ok1 为 true，v1 的值为 10
v2, ok2 := x.(string) // ok2 为 false，v2 的值为 string 类型的零值 ""
v3 := x.(float64)     // 如果断言失败会引发 panic
```

需要注意的是，如果断言的类型是一个接口则语义变成了「变量是否实现了该接口」的判断。
如果断言成功，会返回变量的实际类型和值而不是返回接口类型和值。

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

`Go` 中只能通过 `make` 来创建 `channel`，`make` 接收两个参数，
第一个是 `channel` 的类型，第二个是 `channel` 的缓冲区大小。
缓冲区大小默认为 `0`，表示无缓冲 `channel`。

```go
ch := make(chan int)       // 无缓冲 channel
chBuf := make(chan int, 5) // 有缓冲 channel，缓冲区大小为 5
```

在使用无缓冲 `channel` 的时候，发送方和接收方一定要放在两个不同的 `goroutine` 中，
这是因为如果放在同一个 `goroutine` 中，无缓冲 `channel` 在发送和接收时都会阻塞当前的 `goroutine`。

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

不过要注意的是，如果是单独的指针类型则需要加上 `,` 来进行简写：

```go
func b1[I interface { *int  }](param I) {}
func b2[T *int,](param T) {} // 需要加上逗号
```

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

正是因为 `Go` 语言是值传递的，在使用 `for` 作用于数组的时候可能会有意外结果：

```go
var a = [5]int{1, 2, 3, 4, 5}
var r [5]int
for i, v := range a {
  if i == 0 {
    a[1] = 12
    a[2] = 13
  }
  r[i] = v
}
fmt.Println("r = ", r) // r = [1 2 3 4 5]
fmt.Println("a = ", a) // a = [1 12 13 4 5]
```

因为在上面的代码中 `Go` 语言是值传递的语义，在 `for i, v := range a` 的过程中会对 `a` 进行一次拷贝，
这也就导致了变量 `v` 并没有随着 `a` 的改变而改变，要想解决上述的问题只需要使用 `range &a`，
这样 `range` 的参数就变成了 `*[5]int`，这样就可以保证对 `a` 的修改可以立即反应在变量 `v` 上。
或者也可以使用 `range a[:]` 来对原数组进行切片操作来实现同样的效果。
