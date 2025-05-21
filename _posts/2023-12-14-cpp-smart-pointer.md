---
layout: post
title: The Introduction of C++ Smart Pointers
date: 2023-12-14 17:29:32
last_updated: 2025-05-21 17:30:57
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

## Forward

In order to utilize the RAII concept to the fullest,
`C++ 11` introduced smart pointers.
This article will introduce RAII and three types of smart pointers:

* `std::unique_ptr`
* `std::shared_ptr`
* `std::weak_ptr`

Besides, the article will also introduce those methods below:

* `std::make_unique`
* `std::make_shared`

## RAII (Resources Acquisition Is Initialization)

RAII is the concept of `C++`,
which refers to the idea of acquiring resources during initialization
and automatically releasing them during destruction.

There are some examples of RAII in `C++`:

* `std::unique_lock` or `std::lock_guard` to lock and unlock mutexes automatically in multithreading;
* smart pointers for automatic memory allocation and deallocation;
* `std::jthread` to automatically execute `join`.

RAII is to better organize the code and reduce the possibility of programmer errors.
For example, a programmer may forget to release the memory or lock that has been allocated,
while using RAII, the resource will be automatically released when the object is destructed.

## Smart Pointers

When we need to allocate memory to use,
we often use the `new` keyword to allocate memory in `C++`:

```cpp
class Base {
public:
 int num{10};
    Base() {}
    ~Base() { std::cout << "~Base()" << std::endl; }
};
Base *b = new Base;
delete b;
```

For the above code, we need to use `new` to allocate memory,
and `delete` to release the memory.

However, we may forget to release the memory in some cases:

```cpp
// memory leakage.
int test() {
    Base *b = new Base;
    if (!check()) { // do some check, but failed.
     // delete b; // this may be forgotten easily.
        return -1;
    }
    delete b;
    return 0;
}
```

We may forget to release the memory when we use `return` or `throw` to exit the function.
Smart pointers are designed to solve this problem.

### `std::unique_ptr`

We can use `std::unique_ptr` to substitute the original pointer:

```cpp
int test()
{
    std::unique_ptr<Base> b(new Base);
    if (!check()) { // do some check, but failed.
        return -1;
    }
    return 0;
}
```

With this code above, we do not need to use `delete` to release the memory.

The usage of `std::unique_ptr` is similar to that of a normal pointer:

```cpp
std::unique_ptr<Base> b(new Base);
std::unique_ptr<int> p(new int);
*p = 10;
std::cout << *p << std::endl;
std::cout << b->num << std::endl;
```

We can also use `std::unique_ptr` to manage arrays:

```cpp
std::unique_ptr<int[]> p(new int[10]);
for (int i = 0; i < 10; i++) {
    p[i] = i;
}
for (int i = 0; i < 10; i++) {
    std::cout << p[i] << std::endl;
}
```

`std::unique_ptr` can also bind to a raw pointer directly,
but we need to be careful that if the lifecycle of the smart pointer ends,
the raw pointer will become a dangling pointer:

```cpp
int *p = new int{0};
{
    std::unique_ptr<int> up(p);
    std::cout << *up << std::endl;
}
// we cannot use p here, it's a dangling pointer.
// the dereference of a dangling pointer is an UB.
```

We can also use `std::unique_ptr` to manage polymorphic types:

```cpp
class Base {
public:
    Base() { std::cout << "Base()" << std::endl; }
    virtual ~Base() { std::cout << "~Base()" << std::endl; }
    virtual void test() { std::cout << "Base test()" << std::endl; }
};
class Derived : public Base {
public:
    void test() { std::cout << "Derived test()" << std::endl; }
};

std::unique_ptr<Base> base(new Derived);
base->test(); // "Derived test()"
```

It is worth noting that you should not bind two `unique_ptr` to the same address in memory,
this will cause double free.
In the standard library,
the copy constructor and copy assignment operator of `std::unique_ptr` are deleted,
which can prevent this problem to some extend.
With the reason, it is not recommended to bind a `std::unique_ptr` to a raw pointer directly.

#### The Deleter of `std::unique_ptr`

`std::unique_ptr` is a template class,
the first template parameter is the type of the pointer,
and the second template parameter is a deleter.

The default deleter is like `delete` and `delete[]`.

You can also use a custom deleter to release the resource, for example:

```cpp
// The custom deleter to close a file.
void close_file(std::FILE* fp) {
    std::fclose(fp);
    std::cout << "File closed" << std::endl;
}

{
    using unique_file_t = std::unique_ptr<std::FILE, decltype(&close_file)>;
    // make sure there is demo.txt in current directory.
    // otherwise the fp is nullptr
    unique_file_t fp(std::fopen("demo.txt", "r"), &close_file);
} // here fp is finalized, so the close_file() will be called.
```

### `std::shared_ptr`

`std::shared_ptr` is a smart pointer that can be shared by multiple pointers.
You can bind many `std::shared_ptr` to a same address in memory.

There is a reference count in `std::shared_ptr`
to indicate how many `std::shared_ptr` objects are bound to the same address.
When a new `std::shared_ptr` object is created (it needs to be copied from another `std::shared_ptr`),

It is still not recommended to bind a `std::shared_ptr` to a raw pointer directly, you should use
`std::make_shared` instead.

There is an example for `std::shared_ptr`:

```cpp
int *p = new int;
std::shared_ptr<int> sp1(p);
{
    // this is wrong, when we bind p with a shared_ptr, its ref_count is 1.
    // so this will cause double free.
    // only copy from a shared_ptr can make the ref_count increase correctly.
    // sdt::shared_ptr<int> sp2(p);
    std::shared_ptr<int> sp2(sp1);
    std::cout << sp2.use_count() << std::endl; // 2
    std::cout << sp1.use_count() << std::endl; // 2
}
std::cout << sp1.use_count() << std::endl; // 1
```

`std::shared_ptr` is designed to be used in a multi-threaded environment,
and it is thread-safe.
But the data it points to still needs to be synchronized by other means.

### `std::weak_ptr`

`std::weak_ptr` can be created from a `std::shared_ptr` object
or copied from another `std::weak_ptr` object.

When a `std::weak_ptr` object is created from a `std::shared_ptr` object,
it will not increase the reference count of the `std::shared_ptr` object.
For example:

```cpp
std::shared_ptr<Base> sp(new Base);
std::weak_ptr<Base> wp = sp;
std::cout << sp.use_count() << std::endl; // 1
std::cout << wp.use_count() << std::endl; // 1
```

You can use the `std::weak_ptr<T>::lock` to create a new `std::shared_ptr` object,
which will increase the reference count of the `std::shared_ptr` object,
if the memory has not been released yet, and the reference count will be increased by `1`:

```cpp
std::shared_ptr<Base> sp(new Base);
std::weak_ptr<Base> wp = sp;
std::shared_ptr<Base> newSp = wp.lock();
std::cout << newSp.use_count() << std::endl; // 2
std::cout << wp.use_count() << std::endl; // 2
std::cout << sp.use_count() << std::endl; // 2
sp.reset();
// Still available here.
std::cout << newSp.use_count() << std::endl; // 1
std::cout << wp.use_count() << std::endl; // 1
```

If the memory has been released when calling `std::weak_ptr<T>::lock`,
the `std::weak_ptr` object holds a `nullptr` pointer.
Therefore we should check if the created `std::shared_ptr` object is `nullptr` before using:

```cpp
std::shared_ptr<Base> sp(new Base);
std::weak_ptr<Base> wp = sp;
sp.reset(); // "~Base()"
std::shared_ptr<Base> newSp = wp.lock(); // newSp is bind with nullptr;
if (newSp != nullptr) {
    // do something...
}
std::cout << sp.use_count() << std::endl; // 0
```

`std::weak_ptr<T>::lock` is thread-safe,
which means that if the returned `std::shared_ptr` object is not bound to `nullptr`,
the resource is not released at this time.
If the returned `std::shared_ptr` is bound to `nullptr`,
the resource is released at this time.
The access to the data still needs to be synchronized by other means.

## Better Ways to Create Smart Pointers

In the previous examples, we use `new` and the constructors to create smart pointers.
However, this is not very good,
we can use `std::make_unique` and `std::make_shared` to create smart pointers
so that we do not need to use `new` and `delete` keywords.

Using the `new` and constructors may be unsafe when exceptions are thrown. For example:

```cpp
// In some header file:
void f( std::unique_ptr<T1>, std::unique_ptr<T2> );
 
// At some call site:
f(std::unique_ptr<T1>{ new T1 }, std::unique_ptr<T2>{ new T2 });
```

The compiler may optimize the function call with the order below:

* Allocate memory for `T1`
* Allocate memory for `T2`
* Construct `T1`
* Construct `T2`
* Construct `unique_ptr<T1>`
* Construct `unique_ptr<T2>`
* Call `f()`

If there is an exception thrown when constructing `T1` or `T2`, the memory may never be released.
Using `std::make_unique` can solve this problem.

### `std::make_unique` (since `C++14`)

This is very easy to use, for example:

```cpp
class Base {
public:
 int num1;
 int num2;
 Base() = default;
 Base(int i, int j) : num1(i), num2(j) {}
};
std::unique_ptr<Base> up = std::make_unique<Base>(1, 2); // new Base(1, 2);
// create an array.
// only one parameter is OK, the parameter is the size of the array.
// make sure that the Base() constructor exists.
std::unique_ptr<Base[]> upArray = std::make_unique<Base[]>(3); // new Base[3];
```

### `std::make_shared` (since `C++11`)

The usage of `std::make_shared` is similar to that of `std::make_unique`.

## One Funny Thing

Herb Sutter, chair of the ISO C++ standards committee, writes on his blog that:

> The `C++11` doesn't include `std::make_unique` is partly an oversight, and it will almost
> certainly be added in the future.

## References

[GotW #102: Exception-Safe Function Calls (Difficulty: 7/10)](https://herbsutter.com/gotw/_102/)
[std::unique_ptr cppreference](https://en.cppreference.com/w/cpp/memory/unique_ptr)
[std::shared_ptr cppreference](https://en.cppreference.com/w/cpp/memory/shared_ptr)
[std::weak_ptr cppreference](https://en.cppreference.com/w/cpp/memory/weak_ptr)
[std::make_unique cppreference](https://en.cppreference.com/w/cpp/memory/unique_ptr/make_unique)
[std::make_shared cppreference](https://en.cppreference.com/w/cpp/memory/shared_ptr/make_shared)
