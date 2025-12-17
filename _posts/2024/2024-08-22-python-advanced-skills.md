---
layout: post
title: Python 高级技巧学习
date: 2024-08-22 20:34:45+0800
last_updated: 2025-12-17 17:02:44+0800
description:
tags:
categories: Python
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

本文介绍一些 `Python` 高级技巧。

## 拆包

在 `Python` 中，我们可以使用拆包的方式来将一个序列或者字典中的元素赋值给多个变量。
下面一些例子：

```python
a, b = (1, 2)
print(a, b) # 1 2
a, *b = (1, 2, 3)
print(a, b) # 1 [2, 3]
a, *b, c = (1, 2, 3, 4)
print(a, b, c) # 1 [2, 3] 4
*a, b = (1, 2, 3)
print(a, b) # [1, 2] 3

a = [1, 2, 3]
print(a) # [1, 2, 3]
print(*a) # 1 2 3

a, *b = {'a': 1, 'b': 2, 'c': 3}
print(a, b) # 'a' ['b', 'c']
```

## `args` 和 `kwargs`

`*` 可以用在函数的参数中，这样可以将参数拆包为多个参数：

```python
def f1(a, b, *args):
    return a, b, args
print(f1(1, 2, 3, 4, 5)) # (1, 2, (3, 4, 5))
```

`**` 可以用在函数的参数中，这样可以将参数记录成字典：

```python
def g1(a, b, **kwargs):
    return a, b, c
print(g1(1, 2, x=3, y=4, z=5)) # (1, 2, {'x': 3, 'y': 4, 'z': 5})
```

**NOTE**：`args` 是 `arguments` 的缩写，而 `kwargs` 是 `keyword arguments` 的缩写。

## `fstring`

`fstring` 是 `Python 3.6` 新增的字符串格式化方法，可以使用 `{}` 和 `f` 前缀来格式化字符串。

### `!r`, `!s` 和 `!a`

`fstring` 中的 `!r` 和 `!s` 分别表示调用对象的 `__repr__` 和 `__str__` 方法。

`!a` 则会调用 `ascii`，注意这里没有以 `__` 开头结尾，这是一个 `Python` 提供的方法，
不是一个特殊方法。

在默认情况下 `f'{number}'` 会调用 `number.__str__` 方法，
而 `f'{number!r}'` 会调用 `number.__repr__` 方法。

### `:f`, `:e`, `:E`, `:g` 和 `:%`

`fstring` 中的 `:f`, `:e`, `:E`, `:g` 和 `:%`
分别表示浮点数、科学计数法、科学计数法大写、通用格式和百分比。

```python
number = 123.456
print(f'{number:f}') # '123.456000'
print(f'{number:e}') # '1.234560e+02'
print(f'{number:E}') # '1.234560E+02'
print(f'{number:g}') # '123.456'
print(f'{number:%}') # '12345.600000%'
```

### `:>`, `:<` 和 `:^`

`fstring` 中的 `:>`, `:<` 和 `:^` 分别表示右对齐、左对齐和居中对齐。

```python
number = 123
print(f'{number:>8}') # '     123'
print(f'{number:<8}') # '123     '
print(f'{number:^8}') # '  123   '
```

我们还可以指定填充字符，例如 `f'{number:0>8}'` 表示右对齐，不足 8 位的部分用 `0` 填充。

### `:b`, `:d`, `:o`, `:x` 和 `:X`

`fstring` 中的 `:b`, `:d`, `:o`, `:x` 和 `:X`
分别表示二进制、十进制、八进制、十六进制小写和十六进制大写。

例如 `f'{number:b}'` 表示将 `number` 转换为二进制字符串。
还可以通过设置宽度和填充字符来格式化字符串，例如
`f'{number:08b}'` 表示将 `number` 转换为 `8` 位的二进制字符串，不足 `8` 位的部分用 `0` 填充。

### 千分位分隔符

`fstring` 中可以使用 `,` 等来添加千分位分隔符。

```python
number = 1234567890
print(f'{number:,}') # 1,234,567,890
print(f'{number:_}') # 1_234_567_890
```

### 自定义 `__format__` 方法

我们可以通过自定义 `__format__` 方法来实现自定义的格式化方法。

```python
class Number:
    def __init__(self, number):
        self.number = number

    def __format__(self, format_spec):
        if format_spec == 'b':
            return f'{self.number:08b}'
        elif format_spec == 'd':
            return f'{self.number:08d}'
        elif format_spec == 'o':
            return f'{self.number:08o}'
        elif format_spec == 'x':
            return f'{self.number:08x}'
        elif format_spec == 'X':
            return f'{self.number:08X}'
        else:
            return str(self.number)

number = Number(10)
print(f'{number:b}') # '00001010'
print(f'{number:d}') # '00000010'
print(f'{number:o}') # '00000012'
print(f'{number:x}') # '0000000a'
print(f'{number:X}') # '0000000A'
```

上面的代码中，我们更改了 `Number` 类的 `__format__` 方法，
使得 `Number` 类的 `:b`, `:d`, `:o`, `:x` 和 `:X`
变成了 `8` 位的二进制、十进制、八进制、十六进制小写和十六进制大写。

## `match/case` 语句

`Python 3.10` 新增了 `match/case` 语句，这是一种新的模式匹配语法，可以替代 `if/elif/else` 语句。

```python
def match_color(color):
    match color:
        case 'red':
            print('Red')
        case 'green':
            print('Green')
        case 'blue':
            print('Blue')
        case _:
            print('Unknown')

match_color('red') # Red
```

`match` 语句会依次匹配 `case` 语句，如果匹配成功则执行对应的代码块，
如果没有匹配成功则执行 `_` 代码块。

`match` 语句还可以使用 `|` 来匹配多个值：

```python
def match_color(color):
    match color:
        case 'red' | 'green' | 'blue':
            print('Primary')
        case 'cyan' | 'magenta' | 'yellow':
            print('Secondary')
        case _:
            print('Unknown')

match_color('red') # Primary
```

`match` 语句还可以使用 `if` 来添加条件：

```python
def match_color(color, light):
    match color:
        case 'red' if light:
            print('Light Red')
        case 'red':
            print('Dark Red')
        case 'green' if light:
            print('Light Green')
        case 'green':
            print('Dark Green')
        case _:
            print('Unknown')

match_color('red', True) # Light Red
```

在 `case` 语句中，我们可以指定匹配模式：

```python
def match_color(info):
    match info:
        case ['red', _, _, light] if light:
            print('Light Red')
        case ['red', _, _, light] if not light:
            print('Dark Red')
        case ['green', _, _, light] if light:
            print('Light Green')
        # 也可以使用 *_ 来匹配任意项
        case ['green', *_, light] if not light:
            print('Dark Green')
        case _:
            print('Unknown')

match_color(['red', 1, 2, True]) # Light Red
match_color(['red', 1, 2, False]) # Dark Red
```

还可以使用 `as` 来重新命名变量：

```python
def match_color(info):
    match info:
        case [('red', light) as color, _] if light:
            print(f'Light {color}')
        case [('red', light) as color, _] if not light:
            print(f'Dark {color}')
        case [('green', light) as color, _] if light:
            print(f'Light {color}')
        case [('green', light) as color, _] if not light:
            print(f'Dark {color}')
        case _:
            print('Unknown')

match_color([('red', True), 1]) # Light ('red', True)
```

需要注意的是，只有以下的类型支持对序列进行拆分的写法：

- `list`
- `memoryview`
- `array.array`
- `tuple`
- `range`
- `collections.deque`

需要注意的是在序列结构中使用 `str()` 等是标明匹配项必须是 `str` 类型，而不是进行字符串转换：

```python
def match_color(info):
    match info:
        case [str(name), bool(light)] if light:
            print('Light ' + name)
        case _:
            print('Unknown')

match_color((1, True)) # Unknown
match_color(('red', True)) # Light red
```

## 特殊方法

`Python` 中的特殊方法也叫魔术方法 (`Magic Method`)，这些方法以 `__` 开头和结尾，比如 `__init__`、
`__str__`、`__repr__` 等。
这些方法是 `Python` 解释器调用的，我们可以重写这些方法，从而实现自定义的功能。

这里给出常用的特殊方法的定义与调用方式。

### 类型转换

| 方法                | 说明     | 调用方式       |
| ------------------- | -------- | -------------- |
| `__bool__(self)`    | 布尔化   | `bool(obj)`    |
| `__int__(self)`     | 整型化   | `int(obj)`     |
| `__float__(self)`   | 浮点化   | `float(obj)`   |
| `__complex__(self)` | 复数化   | `complex(obj)` |
| `__bytes__(self)`   | 字节化   | `bytes(obj)`   |
| `__str__(self)`     | 字符串化 | `str(obj)`     |

### 一元运算符

| 方法                       | 说明     | 调用方式              |
| -------------------------- | -------- | --------------------- |
| `__neg__(self)`            | 取负     | `-obj`                |
| `__pos__(self)`            | 取正     | `+obj`                |
| `__abs__(self)`            | 取绝对值 | `abs(obj)`            |
| `__round__(self, ndigits)` | 四舍五入 | `round(obj, ndigits)` |

### 算数运算

| 方法                        | 说明   | 调用方式       |
| --------------------------- | ------ | -------------- |
| `__add__(self, other)`      | 加法   | `obj + other`  |
| `__sub__(self, other)`      | 减法   | `obj - other`  |
| `__mul__(self, other)`      | 乘法   | `obj * other`  |
| `__truediv__(self, other)`  | 除法   | `obj / other`  |
| `__floordiv__(self, other)` | 整除   | `obj // other` |
| `__mod__(self, other)`      | 取模   | `obj % other`  |
| `__pow__(self, other)`      | 幂运算 | `obj ** other` |

说明：上述的所有方法增加了 `r` 前缀，比如 `__radd__`、`__rsub__` 等，表示反向运算，即 `other + obj`。
增加了 `i` 前缀，比如 `__iadd__`、`__isub__` 等，表示增量运算，即 `obj += other`。

### 比较

| 方法                  | 说明     | 调用方式       |
| --------------------- | -------- | -------------- |
| `__eq__(self, other)` | 等于     | `obj == other` |
| `__ne__(self, other)` | 不等于   | `obj != other` |
| `__lt__(self, other)` | 小于     | `obj < other`  |
| `__le__(self, other)` | 小于等于 | `obj <= other` |
| `__gt__(self, other)` | 大于     | `obj > other`  |
| `__ge__(self, other)` | 大于等于 | `obj >= other` |

### 迭代

| 方法                            | 说明       | 调用方式           |
| ------------------------------- | ---------- | ------------------ |
| `__len__(self)`                 | 长度       | `len(obj)`         |
| `__getitem__(self, pos)`        | 获取元素   | `obj[pos]`         |
| `__setitem__(self, pos, value)` | 设置元素   | `obj[pos] = value` |
| `__iter__(self)`                | 迭代器     | `for i in obj:`    |
| `__contains__(self, value)`     | 是否包含   | `value in obj`     |
| `__next__(self)`                | 下一个     | `next(obj)`        |
| `__reversed__(self)`            | 反向迭代器 | `reversed(obj)`    |

### 位运算

| 方法                      | 说明     | 调用方式       |
| ------------------------- | -------- | -------------- |
| `__and__(self, other)`    | 与运算   | `obj & other`  |
| `__or__(self, other)`     | 或运算   | `obj | other`  |
| `__xor__(self, other)`    | 异或运算 | `obj ^ other`  |
| `__lshift__(self, other)` | 左移     | `obj << other` |
| `__rshift__(self, other)` | 右移     | `obj >> other` |
| `__invert__(self)`        | 取反     | `~obj`         |

说明：上述的所有方法增加了 `r` 前缀，比如 `__rand__`、`__ror__` 等，表示反向运算，即 `other & obj`。
增加了 `i` 前缀，比如 `__iand__`、`__ior__` 等，表示增量运算，即 `obj &= other`。

### 构造与析构

| 方法                  | 说明     | 调用方式               |
| --------------------- | -------- | ---------------------- |
| `__init__(self, ...)` | 构造方法 | `obj = ClassName(...)` |
| `__new__(cls, ...)`   | 创建对象 | `obj = ClassName(...)` |
| `__del__(self)`       | 析构方法 | `del obj`              |

其中的 `__new__` 方法是一个类方法，用于创建对象，而 `__init__` 方法是一个实例方法，用于初始化对象。
`__new__` 方法需要返回一个对象，通常是调用父类的 `__new__` 方法，这个返回值会传递给 `__init__` 方法，
而 `__init__` 方法不需要返回值。

### 其他

| 方法                                             | 说明         | 调用方式                   |
| ------------------------------------------------ | ------------ | -------------------------- |
| `__repr__(self)`                                 | 表示形式     | `repr(obj)`                |
| `__format__(self, format_spec)`                  | 格式化       | `format(obj, format_spec)` |
| `__hash__(self)`                                 | 哈希值       | `hash(obj)`                |
| `__index__`                                      | 索引值       | `index(obj)`               |
| `__call__(self, ...)`                            | 调用         | `obj(...)`                 |
| `__delitem__(self, pos)`                         | 删除元素     | `del obj[pos]`             |
| `__getattr__(self, name)`                        | 获取属性     | `obj.name`                 |
| `__setattr__(self, name, value)`                 | 设置属性     | `obj.name = value`         |
| `__delattr__(self, name)`                        | 删除属性     | `del obj.name`             |
| `__enter__(self)`                                | 上下文管理器 | `with obj as ...:`         |
| `__exit__(self, exc_type, exc_value, traceback)` | 上下文管理器 | `with obj as ...:`         |

注意：实现方法后的调用方式可能并不只有一种，
比如 `__str__` 方法可以通过 `str(obj)` 或 `print(obj)` 来调用。
而如果实现了 `__bool__`，那么 `not`、`and` 和 `or` 也会调用这个方法。

注意：`Python` 中的很多特殊方法即使不进行实现也能使用其支持的调用形式，
例如对于一个只实现了 `__add__` 方法的类，也可以使用 `a += b` 这种形式，
此时会被解释为 `a = a + b`，即先调用 `__add__` 方法，然后进行赋值。

## 海象运算符 `:=`

这个符号被称为海象运算符，仅仅是因为它的形状像海象。

`:=` 运算符是 `Python 3.8` 新增的，它可以在表达式中赋值，这样我们就可以在表达式中使用赋值的变量：

```python
if (n := len('hello')) > 5:
    print(f'Length of hello is {n}')

# 如果不使用 := 运算符，那么代码会变得更加冗长
n = len('hello')
if n > 5:
    print(f'Length of hello is {n}')
```

一个更加常见的例子是在循环中使用 `:=` 运算符：

```python
file = open(filename, "r")
while (line := file.readline()):
    print(line.strip())

# 如果不使用 := 运算符，那么代码会变得更加冗长
file = open(filename, "r")
while True:
    line = file.readline()
    if not line:
        break
    print(line.strip())
```

我们还可以在列表推导式中使用 `:=` 运算符。这种情况下，我们往往需要将计算后的结果放到列表中：

```python
numbers = [1, 2, 3, 4, 5]
evens = [x for n in numbers if (x := n * 2) >= 10]

# 如果不使用 := 运算符，那么代码会变得更加冗长
evens = [n * 2 for n in numbers if n * 2 >= 10]
```

上面的代码能够帮我简化一次 `n * 2` 的计算，对于一些复杂的计算，这种方式会更加简洁且效率更高。

### 海象运算符变量的作用域

`:=` 运算符定义的变量的作用域是当前函数，除非目标变量使用 `global` 或者 `nonlocal` 修饰：

```python
if (n := len('hello')) > 5:
    print(f'Length of hello is {n}')

print(n) # OK
```

## 切片

`Python` 中的切片是一个非常强大的功能，我们可以使用切片来获取列表、元组、字符串等序列的子序列。

### 切片对象

除了基础的切片用法之外，我们可以通过 `slice` 类来创建切片对象，这样我们就可以在多个地方使用同一个切片对象：

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9]
s1 = slice(1, 5) # start = 1, end = 5, step = 1
s2 = slice(1) # start = 1, end = None, step = 1
print(numbers[s1]) # [2, 3, 4, 5]
print(numbers[s2]) # [2, 3, 4, 5, 6, 7, 8, 9]

numbers[1:5] = [10, 20, 30, 40]
print(numbers) # [1, 10, 20, 30, 40, 6, 7, 8, 9]
```

上面的例子说明切片获取到的是引用，对于可变对象而言我们修改切片后，原对象也会发生变化：

```python
test = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
# 修改切片的第一个元素的第一个值
test[0:2][0][0] = -1 # 等价于 test[0][0] = -1
print(test) # [[-1, 2, 3], [4, 5, 6], [7, 8, 9]]

# test 中保存的是不可变对象，所以 test 本身不会发生变化
test = [1, 2, 3]
test[0:2][0] = -1
print(test) # [1, 2, 3]
```

## `memoryview`

`memoryview` 是一个内置类，它可以让我们直接操作内存，而不需要复制数据。
`memoryview` 对象可以通过 `memoryview()` 函数创建。

```python
numbers = array.array('B', [1, 2, 3, 4, 5, 6])
# 此时 mem 和 number 共享内存，但是此时我们可以随意更改 mem 的形状等
mem = memoryview(numbers)
mem = mem.cast('B', [2, 3]) # 2 行 3 列
print(mem.tolist()) # [[1, 2, 3], [4, 5, 6]]
mem[1, 2] = 100
print(numbers) # [1, 2, 3, 4, 5, 100]
```

## 字典

### 字典推导式

我们可以使用列表推导式快速创建列表，实际上，对于字典的创建也有类似的语法：

```python
{key: value for key, value in iterable}
```

### 在 `match/case` 中匹配字典

`match/case` 语句还可以用来匹配字典：

```python
def match_color(info):
    match info:
        case {'color': 'red', 'light': True}:
            print('Light Red')
        case {'color': 'red', 'light': False}:
            print('Dark Red')
        case {'color': 'green', 'light': True}:
            print('Light Green')
        case {'color': 'green', 'light': False}:
            print('Dark Green')
        case _:
            print('Unknown')

match_color({'color': 'red', 'light': True}) # Light Red
```

上面的匹配式的意思是只要含有这些键值对的都会被匹配，而且顺序不重要，
即使是一个 `OrderDict` 也会被忽略顺序。

## 集合

### 集合推导式

也可以使用集合推导式创建集合：

```python
{value for value in iterable}
```

### 判断集合的包含关系

使用比较运算符可以判断集合的包含关系：

| 运算符 | 说明   |
| ------ | ------ |
| `<=`   | 子集   |
| `<`    | 真子集 |
| `>=`   | 超集   |
| `>`    | 真超集 |
| `==`   | 相等   |
| `!=`   | 不相等 |

## 可变对象与不可变对象

`Python` 中的对象分为可变对象和不可变对象，可变对象是指对象的值可以改变，
而不可变对象是指对象的值不可以改变。

`Python` 中的不可变对象有：`int`、`float`、`complex`、`str`、`tuple`、`frozenset` 等。

`Python` 中的可变对象有：`list`、`dict`、`set` 等。

一个对象如果可以通过 `hash` 函数计算哈希值且不抛出异常，那么这个对象就是不可变对象。
例如我们可以通过以下的代码来判断一个对象是否是可变对象：

```python
def is_mutable(obj):
    try:
        hash(obj)
        return False
    except TypeError:
        return True

print(is_mutable(1)) # False
print(is_mutable('hello')) # False
print(is_mutable([1, 2, 3])) # True
```

### 避免可变对象作为默认值

在 `Python` 中如果使用可变对象绑定到默认值，
那么在函数调用时会共享这个可变对象，这样会导致默认值的改变：

```python
def f(a=[]):
    a.append(1)
    return a

print(f()) # [1]
print(f()) # [1, 1]
```

如果我们想要避免这种情况，我们可以使用 `None` 作为默认值，
然后在函数内部判断是否为 `None`，然后创建一个新的对象：

```python
def f(a=None):
    if a is None:
        a = []
    a.append(1)
    return a

print(f()) # [1]
print(f()) # [1]
```

## 类属性和实例属性

在 `Python` 中，存在类属性和实例属性的区分，一般而言，类属性可以通过在类的开头定义，
而实例对象在 `__init__` 方法中定义。类属性可以通过类名或者实例对象访问，
而实例属性则可以通过实例对象访问。例如：

```python
class A:
    class_attr = 1

    def __init__(self):
        self.instance_attr = 2

a = A()
print(a.class_attr) # 1
print(a.instance_attr) # 2
print(A.class_attr) # 1
```

如果通过实例对象去修改类属性，那么实际上是创建了一个新的实例属性：

```python
# 创建一个新的实例对象，名字与类属性相同
a.class_attr = 3
print(a.class_attr) # 3
print(A.class_attr) # 1
# 可以通过 del 删除实例属性
del a.class_attr
# 由于实例对象的 class_attr 属性被删除，所以会访问类属性
print(a.class_attr) # 1
```

## 静态方法、类方法和实例方法

在 `Python` 的类中，有三种方法：静态方法、类方法和实例方法。

首先介绍最常见的实例方法，实例方法的第一个参数是 `self`，表示实例对象本身：

```python
class A:
    def instance_method(self):
        # 实例方法可以访问实例属性
        return self
```

实例方法可以通过实例对象调用，也可以通过类对象调用，但是需要传入实例对象：

```python
a = A()
print(a.instance_method()) # <__main__.A object at 0x7f8b3c7b3d30>
print(A.instance_method(a)) # <__main__.A object at 0x7f8b3c7b3d30>
```

类方法使用 `@classmethod` 装饰器来定义，类方法的第一个参数是 `cls`，表示类对象本身：

```python
class A:
    @classmethod
    def class_method(cls):
        # 类方法可以访问类属性
        return cls
```

类方法可以通过类对象调用，也可以通过实例对象调用，但是不需要传入实例对象：

```python
a = A()
print(a.class_method()) # <class '__main__.A'>
print(A.class_method()) # <class '__main__.A'>
```

静态方法使用 `@staticmethod` 装饰器来定义，静态方法没有 `self` 和 `cls` 参数：

```python
class A:
    @staticmethod
    def static_method():
        return 'static method'
```

静态方法可以通过类对象调用，也可以通过实例对象调用，但是不需要传入实例对象：

```python
a = A()
print(a.static_method()) # static method
print(A.static_method()) # static method
```

静态方法其实更像是一个普通的函数，它不能访问类属性和实例属性，但是他可以被实例方法或者类方法调用，
所以如果需要处理的逻辑只会被当前的类多次调用，那么可以使用静态方法实现代码的复用。
