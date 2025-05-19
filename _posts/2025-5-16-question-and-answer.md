---
layout: post
title: Q&A
date: 2025-05-16 21:16:38+0800
last_updated: 2025-05-16 21:16:38+0800
description: Some questions that I have encountered in my work.
tags:
  - Potpourri
categories:
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## What is A/B testing?

A/B testing is a statistical method used to compare two versions of an application to determine
which performs better based on specific metrics.


In A/B testing, users are split into two groups: the control group (A) and the treatment group (B).
The control group is exposed to the original version of the application, while the treatment group
is exposed to the modified version. By analyzing the performance of both groups, you can determine
which version is more effective in achieving the desired outcome.

There are some examples for A/B testing:

- **Website Design**: Testing two different layouts of a webpage to see which one leads to more
conversions.
- **Algorithm Optimization**: Comparing two different algorithms to see which one performs better
in terms of speed or accuracy.

## What are overload, override and overwrite?

- **Overload**: In programming, overloading refers to the ability to define multiple functions or
methods with the same name but different parameters. This allows you to use the same function name
for different purposes, depending on the context.
- **Override**: Overriding is a feature in object-oriented programming where a subclass provides a
specific implementation of a method that is already defined in its superclass. This allows the
subclass to customize or extend the behavior of the inherited method.
- **Overwrite**: Overwriting in `C++` refers to hiding a non-virtual base class member function.

```cpp
// Overload
void print(int content);
void print(String content);

// Override
class Base {
public:
    virtual void print();
};
class Derived : public Base {
public:
    void print() override; // Override the Base class method
};

// Overwrite
class Base {
public:
    void print();
};
class Derived : public Base {
public:
    void print(); // Hides the Base class method
};
```

## What is the `3-2-1 rule` for backups?

* At least `3` copies of your data. This ensures redundancy; if one copy fails,
you have others to rely on.
* At least `2` copies on different storage media. This avoids points of failure.
* At least `1` copy off-site. This protects against local disasters like fire or flood.

