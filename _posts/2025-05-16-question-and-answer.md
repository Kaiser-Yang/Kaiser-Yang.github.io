---
layout: post
title: Q&A
date: 2025-05-16 21:16:38+0800
last_updated: 2025-06-29 19:52:31+0800
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

When developing new features, the DB schema may change, such as adding new columns or tables.
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
- `x`: Extract files from the archive.
- `d`: Delete files from the archive.

## What is CDNs?

A Content Delivery Network (CDN) is a geographically distributed network of servers
that work together to deliver web content
(e.g., images, videos, CSS, JavaScript)
to users more efficiently.
The primary goal is to reduce latency,
improve load times,
and decrease bandwidth consumption by serving content from servers closest to the end user.

The CDN works by caching content on multiple servers located in various geographic locations.
When a user requests content, the CDN routes the request to the nearest server,
which reduces the distance the data has to travel.

## What are SOP and CORS?

SOP is an abbreviation for Same-Origin Policy.
It is a security measure implemented in web browsers.
It restricts web pages from making requests to a different domain
than the one that served the web page.

This policy is in place to prevent malicious websites from accessing sensitive data
from another domain without the user's consent. For example,
if a user is logged into their bank account in one tab,
and then visits a malicious website in another tab,
the malicious website should not be able to access the user's bank account information.
Therefore, the SOP is for security reasons and protects users from cross-site attacks.

**NOTE**: If two URLs have the same protocol, domain, and port,
then they are considered to have the same origin.

However, there are some cases where we need to allow cross-origin requests,
such as when we are using APIs from different domains or when we are using CDNs.

CORS is an abbreviation for Cross-Origin Resource Sharing.
It is a browser mechanism that allows restricted resources on a web page
to be requested from another domain outside the domain from which the first resource was served.
With CORS, the browser will send an `OPTIONS` request to the server
to check if the server allows cross-origin requests.
Then the server will respond with the appropriate headers
(such as `Access-Control-Allow-Origin`,
`Access-Control-Allow-Methods`,
`Access-Control-Allow-Headers` etc.)
to indicate whether the request is allowed or not.

From the introduction above, we can figure out that SOP and CORS are only in
the browser level. Therefore, we can use reverse proxy to bypass the SOP.
For example, we can use `nginx` to set up a reverse proxy to forward requests to the target server.
This way, the browser will only see the requests going to the same origin.

## What is memory alignment?

Memory alignment refers to the way data is arranged and accessed in memory. For example,
the 1-byte data type can be stored at any address,
while the 4-byte data type should be stored at an address that is a multiple of 4.

In `C/C++`, you can not convert pointers of different memory alignment.
However, you can convert any types of pointers to `void*` or `char*` pointers,
because they are 1-byte aligned.

You can use the `alignof` operator in `C++` to get the alignment of a type. There is a function
that can check if a pointer can be safely cast to a pointer of a different type:

```cpp
bool is_aligned(void* ptr, size_t alignment) {
    return reinterpret_cast<uintptr_t>(ptr) % alignment == 0;
}

// Usage
is_aligned(ptr, alignof(int)); // Check if ptr is aligned for int
is_aligned(ptr, alignof(long long)); // Check if ptr is aligned for long long
```

When `is_aligned` returns `true`, it means that the pointer can be safely cast to the target type.

In some cases, you may be asked how to implement `memcpy` effectively.
One common approach is to copy multiple bytes at a time, you may think of trying to
convert the `void *` to `long long *` and copy 8 bytes at a time. Before doing that,
you should check if the pointer is aligned for `long long` type. If it is not aligned,
you can copy the data byte by byte until the pointer is aligned, and then copy the rest of the data
in chunks of 8 bytes.

Actually, there are SIMD (Single Instruction, Multiple Data) instructions, with which you can
loading and storing data in larger chunks, such as 128 bits or 256 bits.
But using SIMD is not portable.

## What is the process of SSL/TLS handshake?

1. The TLS client sends a `Client Hello` message that lists cryptographic information
such as the TLS version and, in the client's order of preference,
the Cipher Suites supported by the client.
The message also contains a random byte string that is used in subsequent computations.
2. The TLS server responds with a `Server Hello` message that contains the Cipher Suite
chosen by the server from the list provided by the client,
and another random byte string.
3. The server also sends its digital certificate (including the public key) to the client.
4. The server sends a `Server Hello Done` message.
5. The TLS client verifies the server's digital certificate.
The TLS client generate another random byte string, then generate a secret key
by all the three random byte strings. The client sends the third random string the server,
and this package will be encrypted with the public key of the server. This is a `Finished`
message, indicating that the client part of the handshake is complete.
6. The TLS server sends can decrypt the `Finished` message using its private key.
Then it can generate the same secret key using the three random byte strings.
The server then sends its own `Finished` message, encrypted with the secret key.
7. Now both the client and server have the same secret key,
and they can start exchanging application data securely.

```txt
+------------+                                      +------------+
| TLS Client |                                      | TLS Server |
+------------+                                      +------------+
       |                                                   |
       | 1. Client Hello                                   |
       |    (Version, Cipher Suites, Random1)              |
       |-------------------------------------------------->|
       |                                                   |
       |                       2. Server Hello             |
       |                          (Chosen Cipher, Random2) |
       |<--------------------------------------------------|
       |                                                   |
       |                            3. Digital Certificate |
       |                               (PubKey)            |
       |<--------------------------------------------------|
       |                                                   |
       |                              4. Server Hello Done |
       |<--------------------------------------------------|
       |                                                   |
       | 5. Verify Cert                                    |
       |    Generate Random3                               |
       |    Derive Secret Key                              |
       |    (Random1 + Random2 + Random3)                  |
       |    Finished                                       |
       |    (Encrypted with Server's PubKey)               |
       |    [Contains Random3]                             |
       |-------------------------------------------------->|
       |                                                   |
       |           6. Decrypt with Private Key             |
       |              Derive Secret Key                    |
       |              (Random1 + Random2 + Random3)        |
       |              Finished (Encrypted with Secret Key) |
       |<--------------------------------------------------|
       |                                                   |
       |      7. Secure Application Data Exchange          |
       |<=================================================>|
       |                                                   |
```

## What does `:(){:|:&};:` do in Bash?

For better understanding, we can substitute the colon `:` with a more descriptive name,
such as `forkbomb` and format it as follows:

```bash
forkbomb() {
    forkbomb | forkbomb &
};
forkbomb
```

From the above code, it is clear that the function `forkbomb` calls itself twice
and runs in the background. This will create an infinite number of processes,
eventually consuming all system resources and causing the system to become unresponsive.

## What are debounce and throttle?

Debounce and throttle are techniques used to control the rate at which a function is executed,
especially in response to events like scrolling, resizing, or key presses.

- **Debounce**: This technique ensures
that a function is only executed after a certain period of time,
and every time the event is triggered, the timer is reset.
- **Throttle**: This technique ensures that a function is executed at most once
within a specified time interval, regardless of how many times the event is triggered.

Debounce is useful for scenarios
where you want to wait until the user has stopped performing an action,
such as typing in a search box or resizing a window.

Throttle is useful for scenarios
where you want to limit the frequency of function execution,
such as handling scroll events or API requests.

## What is ORM?

ORM stands for Object-Relational Mapping, which is a programming technique
that allows developers to interact with a relational database
using object-oriented programming languages.

## How to read pointers in C/C++?

The most common and easy way to read pointers in `C/C++` is the Right-Left Rule.
The Right-Left Rule states that you should start at the identifier, move right when possible,
then left, and repeat. During each step, you should stop when you reach a parenthesis.

For example, we can use it to tell apart `const char * p1`, `char const * p2`, `char * const p3`,
`const char * const p4` and `const * char p5`:

For `const char * p1`:

1. Start at `p1`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `*`, so we stop and get that `p1` is a pointer to something.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `const char`, so we stop and get that `p1` is a pointer to a `const char`.

For `char const * p2`:

1. Start at `p2`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `*`, so we stop and get that `p2` is a pointer to something.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `char const`, so we stop and get that `p2` is a pointer to a `char const`.

For `char * const p3`:

1. Start at `p3`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `const`, so we stop and get that `p3` is constant.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `*`, so we stop and get that `p3` is a constant pointer to something.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `char`, so we stop and get that `p3` is a constant pointer to a `char`.

For `const char * const p4`:

1. Start at `p4`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `const`, so we stop and get that `p4` is constant.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `*`, so we stop and get that `p4` is a constant pointer to something.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `const char`,
so we stop and get that `p4` is a constant pointer to a `const char`.

For `const * char p5`:

1. Start at `p5`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `char`, so we stop and get that `p5` is a `char`.
1. Move right. There is nothing on the right, so we stop.
1. Move left. We see `*`, so we stop and get that `p5` is a pointer to something.
Error occurs, `p5` can not be a pointer and a `char` at the same time.
Therefore, this is not a valid declaration.

There are some complicated examples you can try to practice the Right-Left Rule:

* `int (*(*foo)(int))[5]`
* `int (*(*foo)(int, int))(int, int)`
* `int (*(*foo)(int, int))(int, int) const`

I'll explain `int (*(*foo)(int))[5]`:

1. Start at `foo`.
1. Move right. We see `)` so we stop.
1. Move left. We see `*`, so we stop and get that `foo` is a pointer to something.
1. Move right. We see `(int)`, so we stop and get that `foo` is a pointer to a function
that takes an `int` as an argument.
1. Move left. We see `*`, so we stop and get that `foo` is a pointer to a function
that takes an `int` as an argument and returns a pointer to something.
1. Move right. We see `[5]`, so we stop and get that `foo` is a pointer to a function
that takes an `int` as an argument and returns a pointer to an array of 5 elements.
1. Move left. We see `int`, so we stop and get that `foo` is a pointer to a function
that takes an `int` as an argument and returns a pointer to an array of 5 `int` elements.

## What are Big Endian and Little Endian?

Endianness refers to the order in which bytes are stored in memory.

- **Big Endian**: The most significant byte (MSB) is stored at the lowest memory address.
For example, the number `0x12345678` would be stored as `12 34 56 78`.
- **Little Endian**: The least significant byte (LSB) is stored at the lowest memory address.
For example, the number `0x12345678` would be stored as `78 56 34 12`.

## What is the P0 Test?

The P0 test, also known as the "P0 test case" or "P0 test scenario," is a type of software testing
that focuses on the most critical and high-priority functionalities of a software application.

The P0 stands Priority 0, which means that these test cases are of the highest priority
and must be executed first. The goal of P0 testing is to ensure that the core functionalities
of the application are working correctly before moving on to lower-priority test cases.

Similar to P0, there are also P1, P2, and P3 tests,
which represent lower priority test cases.

## What is the Gray Release?

A gray release is a software deployment strategy that allows for gradual rollout of new features
or changes to a subset of users before a full release. This approach helps to minimize risks
and allows for real-world testing of new features in a controlled manner.

## What is the SaaS?

SasS stands for Software as a Service. It is a software distribution model
where applications are hosted in the cloud and made available to users over the internet.
SaaS eliminates the need for users to install and maintain software on their local devices,
as the software is accessed through a web browser or an application interface.

## References

- [An overview of the SSL/TLS handshake](https://www.ibm.com/docs/en/ibm-mq/9.3.x?topic=tls-overview-ssltls-handshake)
- [Debounce vs throttle](https://www.kaidohussar.dev/posts/debounce-vs-throttle)
- [Streamlining Test Case Execution: Understanding P0 to P4 Prioritization](https://www.linkedin.com/pulse/streamlining-test-case-execution-understanding-p0-p4-sai-teja-vuthuri-ywvoc)
