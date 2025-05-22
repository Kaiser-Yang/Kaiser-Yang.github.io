---
layout: post
title: C++ Multi-threading
date: 2023-12-12 21:16:38+0800
last_updated: 2025-05-22 11:36:08
description: This post introduces the C++ multi-threading library and its usage.
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

Before `C++11`, we must use the operating system's interface
(such as `pthread` in `Linux`)
to implement multi-threading,
which is not cross-platform.
Since `C++11`, the `std` library has provided support for multi-threading,
so we only need to write the code once for different operating systems.

In this post, we will record the new features since `C++11`.
Those below are included:

* `std::function`
* `std::result_of / std::invoke_result`
* `std::bind`
* `std::thread`
* `std::packaged_task`
* `std::future`
* `std::promise`
* `std::shared_future`
* `std::async`
* `std::mutex`
* `std::timed_mutex`
* `std::recursive_mutex`
* `std::recursive_timed_mutex`
* `std::shared_mutex`
* `std::lock_guard`
* `std::unique_lock`
* `std::shared_lock`
* `std::try_lock`
* `std::lock`
* `std::condition_variable`

**NOTE**: Unless otherwise noted, all methods and types are supported since `C++11`.

## Functions

### Use `std::function` to Replace Function Pointers

With `std::function`,
we can replace function pointers and `typedef` or `using` to define a function type:

```cpp
void callback(int a, int b) {}
using CallbackFunction = void(*)(int, int);
CallbackFunction cb1 = callback;
// using std::function has the same effects.
std::function<void(int, int)> cb2 = callback;
cb1(1, 2);
cb2(1, 2);
```

We can use `std::function` to receive the callback function,
which is more flexible than function pointers:

```cpp
template<class T, class ...Args>
void test(std::function<T> cb, Args ...args) {
    cb(std::forward<Args>(args)...);
}
```

There are some examples of using `std::function`:

* Using `std::function` to implement a recursive function inside an anonymous function;

```cpp
auto factorial = [](int n) {
    std::function<int(int)> fac = [&](int n) { return (n < 2) ? 1 : n * fac(n - 1); };
    // this doesn't work, cause it cannot get &fac1 in itself.        
    // auto fac1 = [&](int n) { return (n < 2) ? 1 : n * fac1(n - 1); };
    return fac(n);
};
```

* Binding a member function to `std::function`:

```cpp
struct Foo {
    Foo(int num) : num_(num) {}
    Foo(const Foo &other) = default;
    void print_add(int i) { std::cout << num_ + i << '\n'; }
    int num_;
};
std::function<void(Foo&, int)> f_add_display = &Foo::print_add;
Foo foo(314159);
f_add_display(foo, 1); // same as foo.print_add(1);

std::function<void(Foo, int)> copy_f_add_display = &Foo::print_add;
copy_f_add_display(foo, 1); // same as Foo(foo).print_add(1);
```

**NOTE**: In the code above, if the first parameter is a reference or pointer,
the operation is performed on the original object.
If the first parameter is not a reference or pointer,
a copy constructor is used to bind the object to the parameter.

* Binding a member variable to `std::function` to get the value of the member variable:

```cpp
std::function<int(Foo &)> f_num = &Foo::num_;
f_num(foo); // same as foo.num_, but f_num(foo) is a rvalue, rather than lvalue;
std::function<int(Foo)> copy_f_num = &Foo::num_;
copy_f_num(foo); // same as Foo(foo).num_, but copy_f_num(foo) is a rvalue, rather than lvalue;
```

**NOTE**: In the code above,
you should also note the difference
between the first parameter type being a reference or pointer and not being a reference or pointer.
And you should also note that
if the return value is defined as a non-reference,
the result is an rvalue;
when the return value is defined as a reference type,
the result is an lvalue.

### Use `std::result_of` or `std::invoke_result` to Get the Return Type of a Function

`std::result_of<F(Args...)>` is deprecated in `C++17` and removed in `C++20`.
`std::invoke_result<F, Args...>` will replace it.

From `cppreference`, the main reason for deprecating `std::result_of` is that
when the template passed in is not a callable type,
`std::result_of`'s behavior is undefined;
`std::invoke_result` will use a SFINAE rather than an UB.
Besides, `std::result_of` has some strange characteristics in some cases:

* `F` can not be a function type or an array type (but can be a reference to them);

```cpp
int test(int, int);
// You can not use the function type directly, but this is OK in `std::invoke_result`
// std::result_of<decltype(test)(int, int)>::type a;
// This is OK
std::result_of<decltype(*test)(int, int)>::type a;
```

* Array as a parameter or return type will be converted to a pointer;
* The types of return value or parameters cannot be abstract classes;
* Top-level `const` and `volatile` qualifiers will be discarded;
* The parameter type cannot be `void`;

Those below are some examples of `std::result_of` and `std::invoke_result`:

```cpp
int test(int, int);
std::result_of<decltype(*test)(int, int)>::type a;
std::invoke_result<decltype(*test), int, int>::type b; // since C++ 17
// or
std::invoke_result<decltype(test), int, int>::type c; // since C++ 17
// a, b, and c are all int.

// Using invoke_of in template function, we must using && to get the type.
template<class Func, class ...Args>
typename std::invoke_result<Func, Args...>::type test1(Func&& cb, Args&& ...args) {
    using ReturnType = typename std::invoke_result<Func, Args...>::type;
    return cb(std::forward<Args>(args)...);
}

// Same effects with decltype
template<class Func, class ...Args>
auto test2(Func&& cb, Args&& ...args) -> decltype(cb(std::forward<Args>(args)...)) {
    return cb(std::forward<Args>(args)...);
}

// When parameter is std::funciton
template<class T, class ...Args>
auto test3(std::function<T>&& cb, Args&& ...args)
-> typename std::invoke_result<std::function<T>, Args...>::type {
    return cb(std::forward<Args>(args)...);
}

// When parameter is std::funciton and using decltype
template<class T, class ...Args>
auto test4(std::function<T>&& cb, Args&& ...args) -> decltype(cb(std::forward<Args>(args)...)) {
    return cb(std::forward<Args>(args)...);
}

int x() { return 0; }
int x1 = test1(x);
int x2 = test2(x);
int x3 = test3(std::function<int()>{x});
int x4 = test4(std::function<int()>{x});
```

### `std::bind`

`std::bind` is used to give a function an alias
or redefine the order of function parameters and default values.

For example, we have the following function:

```cpp
void func(int a, int b, int c) {
    // do someting...
    std::cout << a << b << c << sdt::endl;
}
func(1, 2, 3);
```

In some cases, we may need to pass in default values,
so the function may be written in the following form:

```cpp
void func(int a, int b, int c = 3) {
    // do something...
    std::cout << a << b << c << sdt::endl;
}
func(1, 2); // this is OK, it is same with func(1, 2, 3);
```

In some cases, we may need use `func` as `func(1, x, y)` frequently,
but the default parameter can only appear after the non-default parameter.
In this case, we need to change the order of the parameters.
So we may need to redefine a function:

```cpp
void func(int x, int y) {
    func(1, x, y);
}
int x = 0, y = 0;
// func(int, int) and func(int, int, int = 3) are same, 
// so we need convert to make sure the call unambiguous.
((void(*)(int, int))func)(x, y);
```

The above code is a little bit ugly, and every time we call it, we need to convert it.
If we only need use `func(1, x, y)` in a certain scope,
using `std::bind` is a better choice:

```cpp
using namespace std::placeholders; // this if for placeholders: _1 and _2
auto newFunc = std::bind(func, 1, _1, _2);
int x = 0, y = 0;
newFunc(x, y); // this is same with func(1, x, y);
```

`_1` and `_2` are the placeholders for the parameters.
`_1` is for the first actual parameter, and `_2` is for the second actual parameter.
There is an example to show different order of the parameters:

```cpp
auto newFunc = std::bind(func, 1, _2, _1); // this will call func(1, _2, _1);
newFunc(x, y); // this is same with func(1, y, x); because y is _2 and x is _1.
```

The return value can be assigned to `std::function`:

```cpp
// the template parameter is decided by the return type of func and the num of placehoders.
std::function<void(int, int)> newFunc = std::bind(func, 1, _1, _2);
std::function<void(int, int, int)> newFunc2 = std::bind(func, _1, _2, _3);
```

When the parameters are reference types,
we need to use `std::ref` or `std::cref` (for constant reference types) to wrap the parameters.
For example:

```cpp
void modifyA(int &a, int b, int c) {
    a = -1;
    std::cout << a << b << c << std::endl;
}

int a = 0, b = 0, c = 0;
auto badFunc = std::bind(modifyA, a, b, c);
badFunc(); // After this, a will be still 0.
auto goodFunc = std::bind(modifyA, std::ref(a), b, c);
goodFunc(); // After this, a will become -1.
```

The above examples can also be implemented with lambda functions:

```cpp
auto newFunc = [] (int x, int y) {
    func(1, x, y);
};

int x = 0, y = 0;
newFunc(x, y); // Same with func(1, x, y);

auto anotherGoodFunc = [&a, b, c]() {
    modifyA(a, b, c);
};
anotherGoodFunc(); // After this, a is -1.
```

## Multi-threading

### `std::thread`

`std::thread` is very easy to use, you only need to pass a callable object and the parameters:

```cpp
void func(int a, int b, int c) {}
std::thread t1(func, 1, 2, 3); // t1 is another thread running func(1, 2, 3);
// t2 is another thread running a lambda function.
std::thread t2([]() { std::cout << "Hello Thread!\n"; });
t1.join();
t2.join();
```

`std::thread::join` is to wait for the thread to finish.
Without `join`, the thread will be killed when the main thread finished.
Using join is to make the main thread wait for the threads finish.

Since `C++20`, you can use `std::jthread`, which will call `join` automatically.

There is another important function called `detach` of `std::thread`,
which makes the thread run independently.
Using `detach` can make resounds released timely.
For example:

```cpp
void daemonFunc() {
    // do something...
}
void oneThreadFunc() {
    shared_ptr<int> integerP(new int[(int)8e6]); // big data here
    // do something...
    std::thread daemonT(daemonFunc);
    daemonT.detach();
    // if we use daemonT.join() rather than daemonT.detach() to make sure daemonT finish, 
    // the integerP will be released after the daemonT thread.
} // integerP is released after this function.
std::thread t1(oneThreadFunc);
t1.join();
```

In the destructor of `std::thread`, `std::terminate` is called if:

* the thread was not joined (with `join()`);
* or the thread was not detached (with `detach()`).

Thus, you should always either `join` or `detach` the thread before it goes out of scope.

You should be careful with `detach`.
When you detach a thread, and the main thread may finish before the detached thread,
which will cause the shared data to be released before the detached thread finish.
Using the released shared data in the detached thread may cause an UB.
Therefore, using `std::thread::detach()` is not recommended.

### `std::packaged_task`

The creation of `std::packaged_task` is very similar with `std::function`.

`std::thread` is very useful,
but what if we need to handle the return value of a multi-thread function?

We can implement this without `std::packaged_task`.
We just need to create a shared variable,
and set the variable to the return value of the function when the thread function ends.
However, this implementation often requires the use of condition variables or locks.
When we need to consider locks, the program often becomes complex.
Our requirement is very simple: to get the return value of a multi-thread function.
Can we implement this without the programmer managing the locks?

For a function with a return value,
we can bind it to `std::packaged_task`,
and then get the `std::future` object through `std::packaged_task::get_future`.
After that, we bind the `std::packaged_task` to a thread,
and when we call `std::future::get`, the main thread will wait for the thread to finish,
and we can get the return value of the thread function.
For example:

```cpp
int threadFunc() { return 0; }
std::packaged_task<int()> threadTask{threadFunc};
std::future<int> result = threadTask.get_future();
std::thread t{std::move(threadTask)};
std::cout << result.get() << std::endl;
t.join();
```

In the code above, we must use `t.join()` even if the thread function has returned.
This is because when the function returns,
we can use `std::future::get` to get the value,
but the thread may not have completely stopped (it may be releasing resources).

`std::future` also can be used in a single-threaded program:

```cpp
int threadFunc() { return 0; }
std::packaged_task<int()> threadTask{threadFunc};
std::future<int> result = threadTask.get_future();
// run it in the main thread
threadTask();
std::cout << result.get() << std::endl; // 0

// same implementation:
std::function<int()> func{threadFunc};
std::cout << func() << std::endl; // 0
```

### `std::promise`

`std::promise` is used to pass a value (of any type).
Through `set_value`, we can set the value,
and through `get_future`, we can get the `std::future` object.
Similarly, when we call `std::future::get`, the thread will be blocked until the value is set.

The functionality of `std::promise` can be implemented with condition variables,
but using `std::promise` can simplify the code in some scenarios.
Also, we don't need to manually lock or unlock.

The example from `cppreference`:

```cpp
void acc(std::vector<int>::iterator first,
                std::vector<int>::iterator last,
                std::promise<int> accumulate_promise) {
    int sum = std::accumulate(first, last, 0);
    accumulate_promise.set_value(sum); // Notify future
}
 
void do_work(std::promise<void> barrier) {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    barrier.set_value();
}
 
int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6};
    std::promise<int> accumulate_promise;
    std::future<int> accumulate_future = accumulate_promise.get_future();
    std::thread work_thread(acc, numbers.begin(), numbers.end(),
                            std::move(accumulate_promise));
    std::cout << "result=" << accumulate_future.get() << '\n';
    work_thread.join(); // wait for thread completion

    // For void we can use this as a barrier
    std::promise<void> barrier;
    std::future<void> barrier_future = barrier.get_future();
    std::thread new_work_thread(do_work, std::move(barrier));
    barrier_future.wait();
    new_work_thread.join();
    return 0;
}
```

### `std::shared_future`

`std::shared_future` is a wrapper for `std::future`.
It allows multiple threads to access the return value of a function through `get`.

For example:

```cpp
std::promise<void> ready;
std::shared_future<void> sf = ready.get_future();
std::thread t1{[sf]() { sf.get(); std::cout << "Thread 1 has started\n"; }};
std::thread t2{[sf]() { sf.wait(); std::cout << "Thread 2 has started\n"; }};
std::thread t3{[sf]() { sf.wait(); std::cout << "Thread 3 has started\n"; }}; 
// sf.wait() and sf.get() are the same here.
std::this_thread::sleep_for(2000ms);
std::cout << "Finish Some Preparation.\n"; // This line will be output first.
ready.set_value();
t1.join();
t2.join();
t3.join();
std::cout.flush();
```

### `std::async`

`std::async` is a high-level interface for multi-threading.
`std::async` is a function that can be used to run a function asynchronously.
For example:

```cpp
int threadFunc(int &i) { i = -1; return -1; }
int x = 0;
// Don't forget the std::ref to make it pass as a reference.
std::future<int> result = std::async(threadFunc, std::ref(x)); 
std::cout << result.get() << std::endl;
```

`std::async` also supports lazy execution,
which means that the thread will only start executing
when `std::future::wait` or `std::future::get` is called.
For example:

```cpp
void threadFunc() { std::cout << "Thread is running...\n"; }
std::future<void> result = std::async(std::launch::deferred, threadFunc);
std::this_thread::sleep_for(2000ms);
result.wait(); // threadFunc starts to run.
```

## Locks

All the locks introduced below are **unfair locks**,
which means that the blocked processes do not acquire the lock in the order they were blocked.

### `std::mutex`

`std::mutex` can only be acquired by one thread at any time. It is very easy to use:

```cpp
std::mutex mu;
mu.lock();
// do something.
mu.unlock();
```

`std::mutex::lock` will block current thread, if it can not acquire the lock.
Use `std::mutex::try_lock` will not block current thread, but return `false`
when the lock is acquired by another thread:

```cpp
std::mutex mu;
if (mu.try_lock()) {
    // do something... 
    mu.unlock();
} else {
    std::cerr << "failed to get mutex\n";
}
```

### `std::timed_mutex`

`std::mutex::try_lock` only tries to acquire the lock once.
In many cases, we want to keep trying to acquire the lock for a period of time.
`std::timed_mutex` can achieve this.
`std::timed_mutex` is very similar to `std::mutex`,
the only difference is that `std::timed_mutex` provides `std::timed_mutex::try_lock_for`
and `std::timed_mutex::try_lock_until` to decide how long to try to acquire the lock.
When the lock is acquired within the specified time, it returns `true`,
otherwise it returns `false`:

```cpp
std::timed_mutex t_mu;
if (t_mu.try_lock_for(2000ms)) {
    // ... do something
    t_mu.unlock();
} else {
    std::cerr << "Failed to get lock in 2000ms\n";
}
```

**NOTE**: `lock` and `try_lock` are still available in `std::timed_mutex`.

### `std::recursive_mutex`

`std::recursive_mutex` is a recursive lock,
which means that the same thread can acquire the same lock multiple times.
If you try to acquire a `std::mutex` that has already been acquired by the same thread,
the thread will be blocked (this is a deadlock for a single thread).
`std::recursive_mutex` is used in recursive situations:

```cpp
std::recursive_mutex r_mu;
int fabonacci(int n) {
    r_mu.lock();
    // using lock to make sure the cout will be outputting right.
    std::cout << "trying to get fabonacci(" << n << ")\n";
    if (n == 1 || n == 2) {
        r_mu.unlock();
        return 1;
    } else {
        int f1 = fabonacci(n - 1);
        int f2 = fabonacci(n - 2);
        std::cout << n << " " << f1 << " " << f2  << " " << f1 + f2 << std::endl;
        r_mu.unlock();
        return f1 + f2;
    }
}
```

Every time we call `fabonacci`, we need to acquire the lock,
and when we return from the function, we need to release the lock,
which is not meet the `RAII` principle.

### `std::recursive_timed_mutex`

`std::recursive_timed_mutex` is a recursive lock with time-out.
It is very similar to `std::timed_mutex`,
and the only difference is that it provides `try_lock_for` and `try_lock_until`.
The usage is similar with `std::timed_mutex`.

### `std::shared_mutex`

`std::shared_mutex` is introduced in `C++17`.

`std::shared_mutex` is like read-write lock.
It allows multiple threads to read at the same time,
but only one thread can write at a time.
When a thread is writing, no other threads can read or write (`std::shared_mutex::lock`).
When a thread is reading, other threads can read but not write (`std::shared_mutex::shared_lock`).

We can use `std::shared_mutex::lock` to implement the produce-consume model.

The example from `cppreference`:

```cpp
class ThreadSafeCounter {
public:
    ThreadSafeCounter() = default;
    // Multiple threads/readers can read the counter's value at the same time.
    unsigned int get() const {
        std::shared_lock lock(mutex_);
        return value_;
    }
 
    // Only one thread/writer can increment/write the counter's value.
    void increment() {
        std::unique_lock lock(mutex_);
        ++value_;
    }
 
    // Only one thread/writer can reset/write the counter's value.
    void reset() {
        std::unique_lock lock(mutex_);
        value_ = 0;
    }
 
private:
    mutable std::shared_mutex mutex_;
    unsigned int value_{};
};
 
int main() {
    ThreadSafeCounter counter;
    auto increment_and_print = [&counter]() {
        for (int i{}; i != 3; ++i) {
            counter.increment();
            std::osyncstream(std::cout)
                << std::this_thread::get_id() 
                << ' ' << counter.get() << '\n'; // this osyncstream is since C++ 20.
        }
    };
    std::thread thread1(increment_and_print);
    std::thread thread2(increment_and_print);
    thread1.join();
    thread2.join();
    return 0;
}
```

#### Writer Starvation

There is an important point to note when using `std::shared_mutex`.
When a write thread is blocked by a read thread that has acquired `lock_shared`,
the write thread will be blocked.
And during the time the write thread is blocked,
if there are many new read threads that acquire `lock_shared` to perform read operations,
the write thread must wait for all read threads to finish their operations
and release all read locks before it can acquire the write lock.
This can cause serious delays in write operations.
The reason for this is that all `C++` locks are unfair locks.

One simple solution is to add a `std::mutex`.
Regardless of whether it is a read operation or a write operation,
before acquiring the lock, we need to acquire this `std::mutex`.
At the same time, after acquiring the read lock (or write lock),
we release this `std::mutex`.
This ensures that the read thread and write thread will first compete for the lock
before acquiring the lock.
This way, we can ensure that the semantics are not violated
while preventing writer starvation.
However, this will add some overhead.

### RAII

RAII is a C++ programming idiom
that binds the lifecycle of a resource to the lifetime of an object.
In other words, when an object is created,
the resource is acquired, and when the object is destroyed,
the resource is released.

In C++, the most common example of RAII is the use of smart pointers.
When a smart pointer is created, it acquires the resource (memory),
and when the smart pointer is destroyed, it releases the resource (memory).

In multi-threading development,
when we successfully acquire a lock,
we also need to release the lock through `unlock`.
This does not conform to the RAII principle.
Therefore, `std` provides some types and methods
that can automatically release the lock when the destructor is called.

#### `std::lock_guard`

`std::lock_guard` can bind to any object that has `lock` and `unlock` methods.
When the object is created, it acquires the lock,
and when the object is destroyed, it releases the lock.
For example:

```cpp
std::mutex mu;
void threadFunc() {
    std::lock_guard lg{mu};
    // acquire the lock
    // do something...
} // there is no need to unlock
```

#### `std::unique_lock`

`std::unique_lock` is a more flexible version of `std::lock_guard`.
It can be used to bind to any object that has `lock` and `unlock` methods.
When the object is created, it acquires the lock,
and when the object is destroyed, it releases the lock.
Besides, `std::unique_lock` can also use `lock` and `unlock`
to acquire and release the lock after the object is created:

```cpp
std::mutex mu;
void threadFunc() {
    std::unique_lock ul{mu};
    // acquire the lock
    // do something...
    ul.unlock(); // we can unlock if we need this lock no more.
    // do something...
    ul.lock(); // we can lock if we need this lock again.
} // there is no need to unlock
```

And `std::unique_lock` can also release the second parameter:

* `std::adopt_lock`: The default behavior, when the object is created, it acquires the lock;
* `std::try_lock`: The object will try to acquire the lock when it is created;
`std::unique_lock` overloads the `bool` operator,
so we can use `if (ul)` to check whether the lock is acquired successfully;
* `std::defer_lock`: The object will not acquire the lock when it is created,
but we can use `lock` to acquire the lock later. This is usually used with `std::lock`
to achieve the effect of acquiring multiple locks at the same time.

#### `std::shared_lock`

`std::shared_lock` is supported since `C++14`.

`std::shared_lock` is used to bind to any object that has `lock_shared` and `unlock_shared` methods.
When the object is created, it acquires the lock (call `lock_shared`),
and when the object is destroyed, it releases the lock (call `unlock_shared`).
This usually used with `std::shared_mutex`.

It can also receive the second parameter, which is the same as `std::unique_lock`'s.

### Acquire Multiple Locks

#### `std::try_lock`

`std::try_lock` is a function that accepts multiple lock objects
that support `try_lock`.
It tries to acquire multiple locks at the same time.
If any of the `try_lock` calls fail,
the other locks that have been successfully acquired will be released,
and it will return the index of the failed lock (starting from `0`).
If all `try_lock` calls succeed,
it will return `-1`.
For example:

```cpp
std::mutex m1, m2;

int ret = std::try_lock(m1, m2);
if (ret == -1) {
    // do something...
    m1.unlock();
    m2.unlock(); // don't forget release the locks.
} else {
    // m1 and m2 are not got by this thread.
}
```

We must manually release the locks after using `std::try_lock`.
We can also use `std::unique_lock` to bind `std::mutex`, then use `std::try_lock`:

```cpp
std::mutex m1, m2;

std::unique_lock<std::mutex> ul1{m1, std::defer_lock}, ul2{m2, std::defer_lock};
int ret = std::try_lock(m1, m2);
if (ret == -1) {
 // do something...
 // no need to release the locks.
} else {
 // m1 and m2 are not got by this thread.
}
```

#### `std::lock`

`std::lock` is a function that accepts multiple lock objects
that support `lock`.
It tries to acquire multiple locks at the same time.
If any of the `lock` calls fail,
the other locks that have been successfully acquired will be released,
and it will block until all locks are acquired.

```cpp
std::mutex m1, m2;

std::lock(m1, m2); // get m1 and m2 at same time.
// do something...
m1.unlock();
m2.unlock(); // don't forget release the locks.
```

Similarly, we can also use `std::unique_lock` to bind `std::mutex`, then use `std::lock`:

```cpp
std::mutex m1, m2;

std::unique_lock<std::mutex> ul1{m1, std::defer_lock}, ul2{m2, std::defer_lock};
std::lock(m1, m2); // get m1 and m2 at same time.
// do something...
// no need to release the locks.
```

#### Question

Why is there no `std::lock_shared` to acquire multiple shared locks at the same time?

> `std::lock_shared` is not needed because `std::shared_mutex` is a read-write lock.
> When we acquire a shared lock, we can acquire multiple shared locks at the same time.

## `std::condition_variable`

Condition variables should be used with locks.
`std::condition_variable` supports `wait` and `notify` operations.
When we call `wait`, it will block the current thread
and release the lock associated with the condition variable.
When we call `notify`, it will wake up the blocked thread.
When the thread is woken up, it will reacquire the lock that was released during `wait`.
If the lock is already acquired by another thread,
the thread will be blocked until it successfully acquires the lock.

There is an example to show how to use `std::condition_variable`:

```cpp
int sum = -1;
std::mutex mu;
std::condition_variable cv;
void acc(std::vector<int>::iterator first,
                std::vector<int>::iterator last) {
    std::unique_lock<std::mutex> locker{mu};
    sum = std::accumulate(first, last, 0);
    cv.notify_all(); // we usually use notify_all, cause we don't know who are waiting.
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6};
    std::thread work_thread(acc, numbers.begin(), numbers.end());
    std::unique_lock<std::mutex> locker{mu};
    cv.wait(locker, []() { return sum != -1;}); // wait until sum != -1;
    std::cout << sum << std::endl;
    work_thread.join();
    return 0;
}
```

When using `std::condition_variable::wait`, we usually use `notify_all`
to wake up all blocked threads.
This is because we don't know which thread is waiting.
When we wake up all blocked threads,
the threads that do not meet the exit condition of `wait`
will continue to be blocked.
At the same time, it is worth noting that `wait` will first check
whether the exit condition is met,
rather than blocking first and then waking up to check again,
which means if the condition is met when `wait` is called,
the thread will not be blocked.

#### Question

We must acquire the lock before calling `wait`,
and do we need to acquire the lock before calling `notify`?

> It depends on the situation.
> In most cases, we need to acquire the lock before calling `notify`
> because we often need to modify the value of the shared variable.
> However, sometimes we may not need to modify the value of the shared variable
> before calling `notify`, so we can skip acquiring the lock.
> In other words, the lock is not associated with `notify`.
> Whether we need to acquire the lock
> is determined by whether we need to modify the shared variable,
> not by whether we need to call `notify`.
> The code below works well:

```cpp
int sum = -1;
std::mutex mu;
std::condition_variable cv;
void acc(std::vector<int>::iterator first,
                std::vector<int>::iterator last) {
    std::unique_lock<std::mutex> locker{mu};
    sum = std::accumulate(first, last, 0);
    locker.unlock();
    cv.notify_all(); // we can notify without any locks.
}

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5, 6};
    std::thread work_thread(acc, numbers.begin(), numbers.end());
    std::unique_lock<std::mutex> locker{mu};
    cv.wait(locker, []() { return sum != -1;}); // wait until sum != -1;
    std::cout << sum << std::endl;
    work_thread.join();
    return 0;
}
```

## References

* [std::function cppreference](https://en.cppreference.com/w/cpp/utility/functional/function)
* [std::result_of / std::invoke_result cppreference](https://en.cppreference.com/w/cpp/types/result_of)
* [std::bind cppreference](https://en.cppreference.com/w/cpp/utility/functional/bind)
* [std::thread cppreference](https://en.cppreference.com/w/cpp/thread/thread)
* [std::packaged_task cppreference](https://en.cppreference.com/w/cpp/thread/packaged_task)
* [std::future cppreference](https://en.cppreference.com/w/cpp/thread/future)
* [std::promise cppreference](https://en.cppreference.com/w/cpp/thread/promise)
* [std::shared_future cppreference](https://en.cppreference.com/w/cpp/thread/shared_future)
* [std::async cppreference](https://en.cppreference.com/w/cpp/thread/async)
* [std::mutex cppreference](https://en.cppreference.com/w/cpp/thread/mutex)
* [std::timed_mutex cppreference](https://en.cppreference.com/w/cpp/thread/timed_mutex)
* [std::recursive_mutex cppreference](https://en.cppreference.com/w/cpp/thread/recursive_mutex)
* [std::recursive_timed_mutex cppreference](https://en.cppreference.com/w/cpp/thread/recursive_timed_mutex)
* [std::shared_mutex cppreference](https://en.cppreference.com/w/cpp/thread/shared_mutex)
* [std::lock_guard cppreference](https://en.cppreference.com/w/cpp/thread/lock_guard)
* [std::unique_lock cppreference](https://en.cppreference.com/w/cpp/thread/unique_lock)
* [std::shared_lock cppreference](https://en.cppreference.com/w/cpp/thread/shared_lock)
* [std::condition_variable cppreference](https://en.cppreference.com/w/cpp/thread/condition_variable)
* [The Answer from Stack Overflow](https://stackoverflow.com/a/22804813)
