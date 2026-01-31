---
layout: post
title: C++ 临时对象生命周期
date: 2023-12-06 15:50:57+0800
last_updated: 2026-01-31 19:55:43+0800
description: 本文介绍一个关于 C++ 中临时对象生命周期的例子
tags:
  - 中文文章
categories: C/C++
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 引言

朋友给了我一段代码：

```cpp
const std::string & foo(const std::string & a, const std::string & b) {
    return a.empty() ? b : a;
}

int main () {
    const std::string & s = foo("", "foo");
    cout << s << '\n';
    return 0;
}
```

可以思考一下上面的代码能否通过编译？如果可以会输出什么？

## UB

对于上面的代码，是可以通过编译的。使用 `GCC` 的话输出 `foo` ，那么代码似乎没有问题。

其实不然，上述的代码发生了 `UB`。

**UB**: `UB` 是 `Undefined Behaviour` 的缩写，意思是未定义行为既具体会发生什么没有任何保证。

我们将上面的代码稍作更改：

```cpp
const std::string & foo(const std::string & a, const std::string & b) {
    return a.empty() ? b : a;
}

int main () {
    auto & s = foo("", "foo"); // auto is const std::string
    int a[100] = {0};
    cout << s << '\n';
    return 0;
}
```

在使用 `GCC` 的情况下，上面的代码输出了空串。

## 为什么？临时对象的生命周期

要知道为什么会出现上面的问题，我们需要先了解临时对象的生命周期。

- 临时对象：临时对象往往是指右值（纯右值和将亡值）。
- 生命周期：一个对象的生命周期可以理解为从调用构造函数开始到调用析构函数结束的整个过程。

当一个对象的生命周期结束后（调用析构函数后）其不能再被继续使用否则会发生未定义行为。例如：通过 `new` 申请的对象，在 `delete` 之后继续进行解引用，此时会发生未定义行为（访问野指针）。

对于上面提到的代码实际上，在输出的时候，`""` 与 `"foo"` 的生命周期已然结束，所以出现了未定义行为。这是因为在 `std::string` 中用于存放数据的内存已经被回收了（临时对象调用过析构函数），但是 `s` 中依然有指向数据的指针（或者引用），当没有数据对对应地址进行写操作的时候，可能依然能够读出之前的数据，但是增加 `int a[100] = {0}` 后，之前的内存已经被覆盖，因此此时输出空串（但实际上由于是 `UB` 此时发生什么都是可以的，这里只是在根据结果解释）。

一个问题：`std::string` 不是存放在堆上吗？申请的 `int a[100]` 存放在栈上，为什么可以覆盖堆上的内容？

> 在 `C++` 中 `std::string` 的长度大于某个值时（这个值可能是 `16`），其数据才会被放置在堆上，否则还是存放在栈上。同时 `std::string` 中存放着在栈上的数据，例如字符串长度变量，以及指针存放在栈上（使用 `new` 可以使其存放在堆上），通过 `int a[100] = {0}` 可以使长度清 `0` 或指针变成空指针。

简单说明了上面代码的问题之后，知道是因为临时对象已经被析构了，导致其发生了未定义行为。那么，临时对象的生命周期究竟如何呢？

- 对于没有绑定对引用的临时变量其创建完成后，即开始进行析构。例如 `std::string("aaa");` 从该语句的下一行开始，临时变量已经被析构。需要注意的是：`std::string a = std::string("aaa");` 实际上会调用移动构造函数，所以该临时变量已经绑定到引用上了，此时不属于未绑定到引用上的临时变量。
- 对于绑定到引用的临时变量，其生命周期在引用脱离作用域时结束。例如：

```cpp
{
    std::string &&a = std::string("xxx");
} // std::string("xxx") is finalized here.
```

- 特殊地，对于将同一个临时变量绑定到两个不同的引用上，其生命周期以第一个引用为准。例如：

```cpp
{
    std::string &&a = std::string("xxx");
    {
        std::string &&b = std::move(a);
    }
    // using a here is OK
} // std::string("xxx") is finalized here.
```

所以对于引言中的例子，在函数返回之后返回值赋值之前，临时变量已经被析构了。

## 参考

[Lifetime of a temporary cppreference](https://en.cppreference.com/w/cpp/language/reference_initialization#Lifetime_of_a_temporary)
