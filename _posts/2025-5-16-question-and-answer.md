---
layout: post
title: Q&A
date: 2025-05-16 21:16:38+0800
last_updated: 2025-05-16 21:16:38+0800
description: Some questions that I have encountered in my work.
tags:
categories: Potpourri
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

- At least `3` copies of your data. This ensures redundancy; if one copy fails,
you have others to rely on.
- At least `2` copies on different storage media. This avoids points of failure.
- At least `1` copy off-site. This protects against local disasters like fire or flood.

## What is the difference between Docker and a virtual machine?

Docker is based on a more general concept called containers.

The main difference between containers and virtual machines is
that virtual machines will execute an entire OS stack,
including the kernel, even if the kernel is the same as the host machine.

Unlike VMs, containers avoid running another instance of the kernel
and instead share the kernel with the host.
In Linux, this is achieved through a mechanism called LXC (Linux Containers),
and it makes use of a series of isolation mechanisms to spin up a program
that thinks it’s running on its own hardware
but it’s actually sharing the hardware and kernel with the host.

Therefore, containers have a lower overhead than a full VM.
On the flip side, containers have a weaker isolation and only work if the host runs the same kernel.
For instance if you run Docker on MacOS,
Docker needs to spin up a Linux virtual machine to get an initial Linux kernel
and thus the overhead is still significant.
Lastly, Docker is a specific implementation of containers
and it is tailored for software deployment.
Because of this, it has some quirks.
For example, Docker containers will not persist any form of storage between reboots by default.

## How to Upgrade Application When DB Schema Changes?

When developing new features, the DB schema may changes, such as adding new columns or tables.
We must make sure that the data is not lost after upgrading the application.

Versioned schema migrations are a common solution to this problem.
The steps can be summarized as follows:

- Store the version of the schema in the database.
- Creating incremental migration scripts
that can be run to update the schema between two contiguous versions.
Each migration script should be idempotent,
meaning that it can be run multiple times without causing any issues.
We should also update the base schema to the latest version
to support installation the latest version directly.
- Automatically run the migration scripts when the application starts.
For the first installation, we call the base schema script.
For upgrading, we check the current version of the schema,
and run the migration scripts until the specific version.

## How to Build Shared or Static Library?

For shared library, we can use `-fPIC` to generate position-independent object code.
Then we can use `-shared` to generate shared library.

```bash
# Generate position-independent object code
g++ -c -fPIC a.cpp -o a.o
g++ -c -fPIC b.cpp -o b.o
g++ -c -fPIC c.cpp -o c.o
# Generate shared library
g++ -shared -o libfoo.so a.o b.o c.o
```

After that, you may need to update the environment variable `LD_LIBRARY_PATH`
to include the directory where the shared library is located.

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/library
```

Then you can use `-L` and `-l` to link the shared library when compiling your program.

You may be asking why we need update the `LD_LIBRARY_PATH` environment variable
even if we have already specified the path to the shared library using `-L`.
The reason is that the `-L` option only specifies the path to the library at compile time,
which is used to find the library when linking the program.
At runtime, the dynamic linker needs to know where to find the shared library.
The `LD_LIBRARY_PATH` environment variable tells the dynamic linker
where to look for shared libraries when the program is executed.
If the library is not in a standard location (like `/usr/lib` or `/usr/local/lib`),
the dynamic linker will not be able to find it unless you specify the `LD_LIBRARY_PATH`.

For static library, we can use `ar` to archive object files into a static library.

```bash
g++ -c a.cpp -o a.o
g++ -c b.cpp -o b.o
g++ -c c.cpp -o c.o
ar rcs libfoo.a a.o b.o c.o
```

The options for `ar` are as follows:

- `r`: Insert or replace the files in the archive.
- `c`: Create the archive if it does not exist.
- `s`: Create or update the index for the archive.
The index helps the linker find the symbols faster.
- `t`: List the contents of the archive.
- `x`: Extract the files from the archive.
- `d`: Delete the files from the archive.
