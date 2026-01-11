---
layout: post
title: C++ Pack, from Static Loop to Static Prime Check
date: 2024-01-05 16:52:11
last_updated: 2025-05-19 16:52:11
description: "This post introduces C++ Pack and Meta Programming: Static Loop and Static Prime Check."
tags:
  - English Posts
  - Meta Programming
categories: C/C++
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## Forward

Originally, I wanted to find a solution to this problem:

> Using `C++` to implement compile-time determination of prime numbers between `1~100`.

However, I couldn't find a good tutorial,
so I decided to implement a simple solution step by step from compile-time loops.

This article will cover the following content:

* `C++ Pack`;
* Meta Programming: Static Loop and Static Prime Check;

## C++ Core Feature: Pack

`Pack` is a new template feature introduced in `C++11`,
which can be used to receive any number
(as long as the stack memory allows)
of the same or different types of parameters.

Its basic syntax is as follows:

```cpp
template<class ...Args>
void test(Args ...args) {}

test();
test(1); // test(int);
test(1, 2, 3); // test(int, int, int);
test(0, 1.0, 'f', "OK"); // test(int, double, const char *);
```

All the above calls are valid. In addition to the basic usage,
`Pack` can also be used with regular template parameters,
but in this case, the `Pack` must be the last parameter:

```cpp
template<class T0, class ...Args>
void test(T0 t0, Args ...args) {}

// test() ill-formed t0 cannot be empty.
test(1); // T0 is int; Args... is empty.
test(0, 1, 1.0, "f"); // T0 is int, Args... are double and const char *;
```

### Pack Expansion

You can expand parameter types or parameters using `...`:

```cpp
// Provided calling test(int, double);
template<class ...Args>
void test(Args ...args) {
    // Args... -> int, double
    // args... -> arg0, arg1
    std::tuple<Args...>(args...);
}
```

There are some more examples of `Pack` expansion:

```cpp
// Given call test<int, double>(1, 2.0); and the formal parameters are arg0 and arg1.
// Args*... -> int*, double*
// &args... -> &arg0, &arg1
// ++args...-> ++arg0, ++arg1
// (std::cout << args)... -> std::cout << arg0, std::cout << arg1
// std::forward<Args>(args)... -> std::forward<int>(arg0), std::forward<double>(arg1)
```

**NOTE**: The above expansions do not end with a semicolon.

### Static Maximum with Parameter Pack

Methods to find the minimum, sum, product, etc., can be implemented with slight modifications:

```cpp
template<class T>
constexpr T max(T t) { return t; }

template<class T0, class ...Args>
constexpr auto max(T0 t0, Args ...args) {
    // in c++11 constexpr function must have only one return statement
    return t0 > max(args...) ? t0 : max(args...); 
}
```


We can also use the non-recursive way to implement the above code:

```cpp
template<class ...Args>
auto max(Args ...args) {
    using T = typename std::common_type<Args...>::type;
    // sizeof...(Args) can get the number of parameters in Args
    T t[sizeof...(Args)] = {args...};
    T res = t[0];
    for (int i = 1; i < sizeof...(Args); i++) {
        if (t[i] > res) {
            res = t[i];
        }
    }
    return res;
}
```

## Meta Programming

In this non-recursive way, we used `for` to implement, this is not efficient,
we can use this code below to eliminate the `for` loop:

```cpp
template<class ...Args>
constexpr auto sum(Args ...args) {
    return (args + ...); // return (arg0 + arg1 + arg2 + ...);
}
```

The above code used `C++17`'s fold expression to eliminate the `for` loop,
You can use this method below to implement the same function in `C++11`:

```cpp
template<int ...args>
struct Sum {};

template<int t0>
struct Sum<t0> {
    static constexpr int value = t0;
};
template<int t0, int ...args>
struct Sum<t0, args...> {
    static constexpr int value = t0 + Sum<args...>::value;
};

// Usage:
Sum<1, 2, 3, 4>::value; // 10
```

The code above make the time complexity of the `sum` function possible to be `O(1)`.
This is because the compiler may optimize `1 + 2 + 3 + 4` into `10` at compile time.

This is the concept of `Meta Programming`,
we let the compiler do some computations at compile time to improve the performance of the program.

### Static Loop

The static loop refers to the loop that only uses compile-time constants.

We can check if an expression is a compile-time constant
by checking if it is possible to assign it to a `constexpr` variable.

```cpp
int x = 10;
constexpr int x1 = x;  // WRONG, x is not constexpr
constexpr int x2 = 10; // OK, 10 is constexpr
constexpr int x3 = x2; // OK, x2 is constexpr
```

We can use this code below to implement the static loop:

```cpp
template<class T, T val>
struct constexprT {
    static constexpr T value = val;
};

template<size_t begin, size_t end, class Lambda>
constexpr void staticFor(Lambda lambda) {
    // C++ 17 required to use constexpr if
    if constexpr (begin < end) {
        // lambda is the body of the loop
        // The paramter is like the index i in the for
        lambda(constexprT<size_t, begin>{});
        staticFor<begin + 1, end>(lambda);
    }
}

// Usage:
staticFor<0, 100>([] (auto i) {
    // i.value is constexpr
    std::cout << i.value << std::endl;
});
```

There is `std::make_index_sequence` to simplify the above code:

```cpp
// Index... -> 0, 1, 2, 3, ..., 99
template<size_t ...Index, class Lambda>
constexpr void staticForImpl(Lambda lambda, std::index_sequence<Index...>) {
    // Fold expression in C++ 17
    // Expand to (lambda(0), lambda(1), ..., lambda(99));
    (lambda(std::integral_constant<size_t, Index>{}), ...);
}

template<size_t N, class Lambda>
constexpr void staticFor(Lambda lambda) {
    staticFor(lambda, std::make_index_sequence<N>{});
}

// Usage:
staticFor<100>([] (auto i) {
    std::cout << i.value << std::endl;
});
```

### Static Prime Check

For a number, we can check if it is a prime number by the following steps:

1. Calculate $$ Number\ \%\ i, i = 2,3,\cdots,Number-1 $$;
2. Check if the result is all $$ 0 $$;

We can use this code below to check if there is a `0` in the result:

```cpp
template<class ...Args>
constexpr bool anyZero(Args ...args) {
    return (false || ... || (args == 0));
}

// Usage:
anyZero(0, 1, 2, 3); // true
anyZero(1, 2, 3, 4); // false
```

The above code also can be implemented with template classes:

```cpp
template<class Type, Type ...number>
struct AnyZero { };

template<class Type, Type number>
struct AnyZero<Type, number> {
    static constexpr bool value = (number == 0);
};

template<class Type, Type number0, Type ...number>
struct AnyZero<Type, number0, number...> {
    static constexpr bool value = (number0 == 0 || AnyZero<Type, number...>::value);
};

// Usage:
AnyZero<int, 0, 1, 2, 3>::value; // true
AnyZero<int, 1, 2, 3, 4>::value; // false
```

Now we can use the implemented static loop to calculate the remainder of the number:

```cpp
constexpr int MAX_NUMBER = 100;
template<size_t ...Index>
constexpr bool isPrimeImpl(std::index_sequence<Index...>) {
    // The conditional operator here is to avoid dividing by 0 and 1
    return !anyZero(((sizeof...(Index) %
                    ((std::integral_constant<size_t, Index>().value == 0 ||
                      std::integral_constant<size_t, Index>().value == 1) ?
                      MAX_NUMBER + 1 :
                      std::integral_constant<size_t, Index>().value)))...);
}
template <int Number>
constexpr bool isPrime()
{
    // Special case for 0 and 1
    if constexpr (Number == 0 || Number == 1) { return false; }
    return isPrimeImpl(std::make_index_sequence<Number>{});
}

// Usage:
isPrime<10>(); // false;
isPrime<7>(); // true;
```

We can use this function with static loop to output the prime numbers between `1~100`:

```cpp
template<size_t ...Index, class Lambda>
void staticForImpl(Lambda lambda, std::index_sequence<Index...>) {
    bool isPrime[sizeof...(Index)] = {(lambda(std::integral_constant<size_t, Index>{}))...};
    for (size_t i = 0; i < sizeof...(Index); i++) {
        if (isPrime[i]) {
            std::cout << i << " ";
        }
    }
    std::cout << std::endl;
}

template<size_t N, class Lambda>
void staticFor(Lambda lambda) {
    staticForImpl(lambda, std::make_index_sequence<N>{});
}

staticFor<MAX_NUMBER>([](auto i) {
    return isPrime<i.value>();
});

// Usage:
staticFor<100>([](auto i) {
    return isPrime<i.value>();
});
```

## References

* [parameter pack cppreference](https://en.cppreference.com/w/cpp/language/parameter_pack)
* [C++ 变长模板参数与折叠表达式教学 bilibili](https://www.bilibili.com/video/BV1NM4y1e7Hn/?spm_id_from=333.788&vd_source=d042cf319136676c69986086c618b3bb)
* [std::get cppreference](https://en.cppreference.com/w/cpp/utility/tuple/get)
* [std::integral_constant cppreference](https://en.cppreference.com/w/cpp/types/integral_constant)
