---
layout: post
title: C++ Value Categories, References, Move Semantics and Perfect Forwarding
date: 2023-12-3 21:16:38+0800
last_updated: 2025-05-18 21:16:38+0800
description:
tags:
categories: C/C++
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

In this post, those concepts will be introduced:

* Value Categories: glvalue, xvalue, lvalue, prvalue and rvalue
* lvalue reference and rvalue reference
* Move and Copy Semantics
* Reference collapsing, universal reference and perfect forwarding

These contents are core features in `C++11`.

## Value Categories

In `C++ 98/03`, it's OK if you don't know what value categories are.
Since `C++11`, the concept of value categories has changed a lot. If you don't understand
them, you may not write the code correctly, and you will not understand the move semantics,
which is one of the most important feature in `C++11`.

Before we dive in, we need to know the following concepts:

* identity: i.e. an address, a pointer the user can determine whether two copies are identical,
etc.

### glvalue (generalized left value)

glvalue are those that have identity.

You may still don't know what glvalue is. Don't worry, glvalue is
also the union of lvalue and xvalue.
After I introduce lvalue and xvalue, you will know what glvalue is.

### lvalue (left value)

There is a very detailed definition of lvalue in `cppreference`, but it's too detailed to remember.
We can simply understand lvalue as those whose address can be obtained by `&`.

There are some cases of lvalue:

* String literal, e.g. `"Hello World"`;
* The built-in pre-increment and pre-decrement operations, e.g. `--a, ++a`;
* The built-in assignment and compound assignment operations, e.g. `a = b`, `a += b`;
* `a, b`, the built-in comma expression, where `b` is an lvalue;
* A cast expression to lvalue reference type, e.g. `static_cast<int&>(a)`;
* `a.*mp` or `p->*mp`, the built-in pointer to member of objecct or pointer to member of pointer
expression, where `mp` is a data member;
* `a[n]` where `a` is an array lvalue.
* A constant template parameter of an lvalue reference type;

```cpp
template <int& v>
void set() {
    v = 5; // template parameter is lvalue
}
 
int a{3}; // static variable, fixed address is known at compile-time
 
void foo() {
    set<a>();
}
```

**NOTE**: An array lvalue is an array that has an identifier, e.g. `int a[3]`, and `a` is an
array lvalue.

### prvalue (pure right value)

prvalue has no identity, and you can not get its address by `&`.

There are some cases of prvalue:

* The built-in post-increment and post-decrement operations, e.g. `a++, a--`;
* The built-in logical expression, e.g. `a && b`, `a || b`;
* The built-in arithmetic expression, e.g. `a + b`, `a - b`;
* The built-in comparison expression, e.g. `a == b`, `a != b`;
* The built-in address-of expression, e.g. `&a`;
* `a, b`, the built-in comma expression, where `b` is an prvalue;
* `a.m` or `p->m` where `m` is a non-static member function or member enumeration;
* `a.*mp` or `p->*mp`, the built-in pointer to member of objecct or pointer to member of pointer
expression, where `mp` is a member function;
* A cast expression to non-reference type, e.g. `static_cast<int>(a)`;
* `this` pointer;
* An enumerator;
* A lambda expression;
* A constant template parameter of a scalar type;

```cpp
template <int v>
void foo()
{
    // not an lvalue, `v` is a template parameter of scalar type int
    const int* a = &v; // ill-formed
    v = 3; // ill-formed: lvalue required as left operand of assignment
}
```

### xvalue (expiring value)

xvalue has identity, but you can not get its address by `&`.

There are some cases of xvalue:

* `a.m` where `a` is an rvalue and `m` is non-static data member;
* `a.*mp` where `a` is an rvalue and `mp` is pointer to data member;
* `a, b` where `b` is an xvalue;
* A function call whose return type of value is an rvalue reference type, e.g. `std::move(x)`;
* A cast expression to rvalue reference type, e.g. `static_cast<int&&>(a)`;
* `a[n]`, the built-in subscript operator, where `a` is an array rvalue;
* Any expression that designates a temporary object, after temporary materialization,
e.g. `A().m`;

Some expressions consisting of names of variables are lvalue expressions, but in some special cases
, they are move-eligible. They are:

* A return statement;
* A co_return statement (Coroutines are the feature of `C++20`);
* A throw expression;

Move-eligible expressions are defined as xvalue since `C++23`.

**NOTE**: An array rvalue is an array that has no identifier, e.g. `(int[3]){1, 2, 3}`.

### rvalue (right value)

rvalue is a union of prvalue and xvalue.

> Actually, there should be four types of value categories:
>
> * has identity and can not be moved (lvalue);
> * has identity and can be moved (xvalue);
> * has no identity and can be moved (prvalue);
> * has no identity and can not be moved.
>
> The last one is useless in `C++`.

## References

## lvalue reference

lvalue reference is a reference to an lvalue.

In my opinion, lvalue reference is syntactic sugar for constant pointer.

Without lvalue reference, we must use pointer to pass parameters to a function, so that the
modification of the parameter can be reflected to the argument.

For example:

```cpp
void badSwapTwoInt(int a, int b) {
    int temp = a;
    a = b;
    b = a;
}
```

The code above can not swap two integers correctly,
because the formal parameters are copies of the actual parameters.
We must use pointers (before lvalue reference):

```cpp
void goodSwapTwoInt(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
} 
```

With lvalue reference, we can use the following code:

```cpp
void goodSwapTwoIntByRef(int &a, int &b) {
    int temp = a;
    a = b;
    b = temp;
}
```

### Possible Implementation

The above code is almost equivalent to the following code:

```cpp
void goodSwapTwoIntAct(int* const a, int* const b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

There are still some differences between the two codes.
`cppreference` mentions that a reference has no extra space,
but the above code needs extra space to store the address of the pointer.

## rvalue reference

### Why rvalue reference?

rvalue reference is for move semantics. Before `C++11`, there is no move semantics,
only copy semantics (copy constructor and copy assignment operator).
For the class whose member variable contains pointers or user-defined types,
we often need to implement a copy constructor and copy assignment operator by ourselves
so that the data can be successfully copied (usually called deep copy), rather than just
let the pointer point to the same memory (usually called shallow copy).

When we pass a large object to a function,
whose formal parameter is a non-pointer or lvalue reference type. The copy constructor
or copy assignment operator will be called. For example:

```cpp
// when calling function a,
// and reaching c,
// there are three copies of data (a, b, c) in memory.
void C(BigData c) { // do something }
void B(BigData b) { C(b); }
void A(BigData a) { B(a); }
```

In some situations, we only need one copy of data in memory, so we can use
lvalue reference or pointer to solve the problem. For example, the above code can be
changed to:

```cpp
// there are only one data (a, b and c are the same data) in memory.
void C(BigData &c) { // do something }
void B(BigData &b) { C(b); }
void A(BigData &a) { B(a); }
```

However, the above code can not be used to pass a temporary object to a function.
In order to support this, we can use `const` to pass a temporary object to a function:

```cpp
// there are only one data (a, b and c are the same data) in memory.
void C(const BigData &c) { // do something }
void B(const BigData &b) { C(b); }
void A(const BigData &a) { B(a); }

// OK, we can bind a temporary object to a const lvalue reference even before C++11
A(BigData{"a.txt"}); // read big data from data.txt
```

You may think this is good, and no need for rvalue reference. But NO!
In some situations, we may need to update the parameter in the function.
However, for this code, we can not.

rvalue reference can be binded to any rvalue and also can be modified.

For example:

```cpp
BigData &&bigData = BigData{"a.txt"};
```

## Move Semantics

If you've learned other languages, you may know shallow copy and deep copy.
Move semantics in `C++` is similar to shallow copy.
In `C++`, we implement move semantics by make the pointer point to the same memory
and then set the original pointer to `nullptr`, which may be a little bit different
from shallow copy in other languages.

```cpp
// Move constructor
ClassType(ClassType &&other);
// Move assignment operator
ClassType& operator=(ClassType &&other);
```

The compiler will automatically generate a move constructor and move assignment operator
if you don't define any of a copy constructor, copy assignment operator,
move constructor or move assignment operator.
The default move constructor and move assignment operator are same with the default copy constructor
and copy assignment operator. So in most cases, we need to implement our own move constructor
and move assignment operator to make them work with move semantics.

There are some examples for move semantics:

Initialization phase, if the right side of the assignment is an rvalue,
then it will trigger the move constructor:

```cpp
// std::string("HelloWorld") is rvalue, this will trigger move constructor
std::string temp = std::string("HelloWorld");
std::string value{std::move(temp)};
```

Without move semantics, the above code will create a temporary object
and then copy it to `value` through the copy constructor;
with move semantics, the above code will create a temporary object
and then move it to `value` through the move constructor
(if we don't consider the optimization of the compiler).
Move semantics reduces one time copy operation.

Assignment phase, if the right side of the assignment is an rvalue,
then it will trigger the move assignment operator:

```cpp
std::string value;
// std::string("HelloWorld") is rvalue, this will trigger move assignment.
value = std::string("HelloWorld");
```

If we bind a rvalue to a rvalue reference, no move semantics will occur:

```cpp
std::string value{"HelloWorld"};
// No movement occurs, value is still "HelloWorld".
std::string &&rRef = std::move(value);
```

In the above code, if we modify `rRef`, it will affect `value`,

If we assign a rvalue reference to a variable, no move semantics will occur unless we use
`std::move`:

```cpp
std::string &&rRef = std::string("HelloWorld");
// No movement occurs, rRef is still "HelloWorld".
std::string value = rRef;
// Move occurs, rRef is now "", and you should not use it any more.
// std::string value = std::move(rRef);
```

Now we can finish the previous problem with move semantics:

```cpp
// Now, we can modify the parameter in the function
void C(BigData &&c) { // do something }
void B(BigData &&b) { C(std::move(b)); }
void A(BigData &&a) { B(std::move(a)); }
A(BigData("data.txt"));
```

In this code above, we must use `std::move` to convert the lvalue to an rvalue.
It is worth noting that no matter what the type of a variable is, the expression
is always an lvalue unless we use `std::move` to convert it to an rvalue.

**NOTE**: If we move a variable to the other, we should not use the original variable any more.

There is an example in `cppreference`:

```cpp
std::string str = "Salut";
std::vector<std::string> v;

// Uses the push_back(const T&) overload, which means 
// we'll incur the cost of copying str
v.push_back(str);
std::cout << "After copy, str is " << std::quoted(str) << '\n';

// Uses the rvalue reference push_back(T&&) overload,
// which means no strings will be copied; instead, the contents
// of str will be moved into the vector. This is less
// expensive, but also means str might now be empty.
v.push_back(std::move(str));
std::cout << "After move, str is " << std::quoted(str) << '\n';

std::cout << "The contents of the vector are { " << std::quoted(v[0])
          << ", " << std::quoted(v[1]) << " }\n";
```

Output:

```
After copy, str is "Salut"
After move, str is ""
The contents of the vector are { "Salut", "Salut" }
```

After the second call to `push_back`, the contents of `str` are moved into the vector, and the
original `str` is now empty. We should not use `str` any more.

## lvalue VS rvalue

The simplest way to distinguish between lvalue and rvalue is to check
if the expression can be binded to a lvalue reference or rvalue reference.

Actually, if the expression can be binded to a lvalue reference, then it is an lvalue;
if the expression can be binded to a rvalue reference, then it is an rvalue.

## Perfect Forwarding

### Reference Collapsing

Reference collapsing is a rule that determines the type of a reference
when two references are combined. The rules are as follows:

```cpp
typedef int&  lref;
typedef int&& rref;
int n;

lref&  r1 = n; // type of r1 is int&
lref&& r2 = n; // type of r2 is int&
rref&  r3 = n; // type of r3 is int&
rref&& r4 = 1; // type of r4 is int&&
```

From the code above, we can find that only when two rvalue references
are combined, the result is an rvalue reference.

### Universal Reference

Universal reference is also called forwarding reference,
which is proposed to solve the problem of perfect forwarding.
The reason why it is called universal reference is that it can bind to both lvalue and rvalue.
In general, universal reference is defined as follows:

```cpp
template<class T>
void f(T &&t) {};

// NOTE:
// This is not a forwarding reference, because forwarding reference must be cv-unqualified
// template<class T>
// void f(const T &&t);

int x = 0;
f(0); // call f(int &&t), T is int;
f(x); // call f(int &t), T is int&;
f(std::move(x)); // call f(int &&t), T is int&&;
int &lRef = x;
int &&rRef = 0;
f(lRef); // call f(int &t), T is int&;
f(rRef); // call f(int &t), T is int&;
f(std::move(lRef)); // call f(int &&t), T is int&&;
f(std::move(rRef)); // call f(int &&t), T is int&&;
```

In the code above, reference collapsing occurs unless the argument is a pure rvalue.
We can find that when the actual parameter is an lvalue, the type of `t` is an lvalue reference,
and when the actual parameter is an rvalue, the type of `t` is an rvalue reference.

### What is Perfect Forwarding?

The term forwading here refers to the process of passing parameters during function calls.

Perfect forwarding means the process of passing parameters to another function
while maintaining the original value category of the parameters.
For example, if the parameter is an lvalue,
it will be passed as an lvalue so that it can be binded to an lvalue reference;
if the parameter is an rvalue,
it will be passed as an rvalue so that it can be binded to an rvalue reference.

### Why Perfect Forwarding?

Perfect forwarding can reduce the number of copies of data in memory.
When the actual parameter is an rvalue,
we can use move semantics to pass the parameter to another function,
so that we can avoid the copy constructor and copy assignment operator.

### An Example of Perfect Forwarding

```cpp
// The receiver
void handleInt(int &x) { }
void handleInt(int &&xx) { }

// The forwarding function
template<class T>
void perfectForwarding(T &&t) {
    // do something ...

    // call another function:
    if (std::is_same<int, std::remove_reference_t<T>>::value) {
        handleInt(std::forward<T>(t));
    } else {
        // do something ...
    }
}
int x = 0;
int &lRef = x;
int &&rRef = 0;
perfectForwarding(1); // rvalue, so call handleInt(int &&x);
perfectForwarding(x); // lvalue, so call handleInt(int &x);
perfectForwarding(std::move(x)); //  rvalue, so call handleInt(int &&x);
perfectForwarding(lRef); // lvalue, so call handleInt(int &x);
perfectForwarding(rRef); // lvalue, so call handleInt(int &x);
perfectForwarding(std::move(lRef)); // rvalue, so call handleInt(int &&x);
perfectForwarding(std::move(rRef)); // rvalue, so call handleInt(int &&x);
```

The source code of `std::forward` is as follows:

```cpp
template<class T>
T&& forward(typename std::remove_reference<T>::type& t) noexcept { return static_cast<T&&>(t); };
template<class T>
T&& forward(typename std::remove_reference<T>::type&& t) noexcept { return static_cast<T&&>(t); };
```

This source code using the reference collapsing rules to determine the type of the return value.
If the type of the parameter is an lvalue reference, the return value is an lvalue reference;
if the type of the parameter is an rvalue reference, the return value is an rvalue reference.

The `std::remove_reference` can be implemented as follows:

```cpp
template<typename _Tp>
struct remove_reference { typedef _Tp type; };

// Template specialization
template<typename _Tp>
struct remove_reference<_Tp&> { typedef _Tp type; };

// Template specialization
template<typename _Tp>
struct remove_reference<_Tp&&> { typedef _Tp type; };
```

## References

* [value category cppreference](https://en.cppreference.com/w/cpp/language/value_category)
* [std::move cppreference](https://en.cppreference.com/w/cpp/utility/move#:~:text=C%2B%2B%20Utilities%20library%20std%3A%3Amove%20is%20used%20to%20indicate,to%20a%20static_cast%20to%20an%20rvalue%20reference%20type.)
* [reference cppreference](https://en.cppreference.com/w/cpp/language/reference)
* [std::forward cppreference](https://en.cppreference.com/w/cpp/utility/forward)
