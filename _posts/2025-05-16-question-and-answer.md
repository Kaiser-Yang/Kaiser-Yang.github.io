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
On Linux, this is achieved through a mechanism called LXC (Linux Containers),
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

RSA key exchange process:

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
6. The TLS server can decrypt the `Finished` message using its private key.
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

RSA key exchange is not secure enough, and it has been deprecated in TLS 1.3.
The main problem of RSA key exchange is that if the private key of the server
is compromised, all past communications can be decrypted.
Diffie-Hellman key exchange process can solve this problem, and its process is as follows:

1. The TLS client sends a `Client Hello` message containing Supported TLS version,
list of supported Cipher Suites (ordered by client preference) and a random byte string
(`Client Random`) for later cryptographic operations.
2. The server responds with a `Server Hello` message containing,
Chosen Cipher Suite from the client’s list, a second random byte string (`Server Random`).
3. The server transmits its digital certificate, which includes its long-term public key.
4. The server sends a `Server Key Exchange` message with Diffie-Hellman (DH) public parameters:
prime modulus `p` and generator `g`,
server’s temporary DH public key
(`g^a mod p`, where `a` is the server’s short-term private exponent).
This message is signed with the server’s long-term private key to authenticate the server.
5. The server signals the end of the initial handshake phase.
6. The client verifies the server’s certificate (checks validity, revocation status, and signature).
The client generates its own DH key pair: temporary private exponent `b` and public key
(`g^b mod p`). The client computes the pre-master secret as `g^(a*b) mod p`
(using the server’s public key `g^a` and its own private key `b`).
The client sends a `Client Key Exchange` message containing its DH public key (`g^b mod p`).
The client derives the master secret using `Client Random`, `Server Random`,
and the pre-master secret.
The client generates session keys (symmetric encryption keys, HMAC keys, etc.)
from the master secret.
7. The client sends a `Finished` message that contains a hash of all handshake messages exchanged
so far (from `Client Hello` to `Client Key Exchange`), and is encrypted with the session key
(proves the client knows the correct key).
This confirms the client’s side of the handshake is complete.
8. The server computes the pre-master secret as `g^(a*b) mod p`
(using the client’s public key `g^b` and its own private key `a`),
derives the same master secret using `Client Random`, `Server Random`, and the pre-master secret,
generates session keys from the master secret, and sends its own `Finished` message
which contains a hash of all handshake messages (from `Client Hello` to `Server Key Exchange`)
and is encrypted with the session key (proves the server knows the correct key).
9. Both client and server now share the same session keys.
They use these keys to encrypt/decrypt application data,
ensuring confidentiality and integrity for all subsequent communication.

```txt
+------------+                                      +------------+
| TLS Client |                                      | TLS Server |
+------------+                                      +------------+
       |                                                   |
       | 1. Client Hello                                   |
       |    (TLS Version, Supported Cipher Suites, Random1)|
       |-------------------------------------------------->|
       |                                                   |
       |                       2. Server Hello             |
       |                     (Chosen Cipher Suite, Random2)|
       |<--------------------------------------------------|
       |                                                   |
       |                            3. Digital Certificate |
       |                              (Server's Public Key)|
       |<--------------------------------------------------|
       |                                                   |
       |               4. Server Key Exchange              |
       |                 (DH Public Params: p, g, g^a)     |
       |       [Signed with Server's Long-Term Private Key]|
       |<--------------------------------------------------|
       |                                                   |
       |                              5. Server Hello Done |
       |<--------------------------------------------------|
       |                                                   |
       | 6. Client Actions:                                |
       |    - Verify Server Certificate                    |
       |    - Generate DH Key Pair (g^b mod p)             |
       |    - Compute Pre-Master Secret: g^(a*b) mod p     |
       |    - Derive Master Secret                         |
       |    - Generate Session Keys (from Master Secret)   |
       |    - Client Key Exchange: Send g^b mod p          |
       |-------------------------------------------------->|
       |                                                   |
       | 7. Server Actions:                                |
       |    - Compute Pre-Master Secret: g^(a*b) mod p     |
       |    - Derive Master Secret                         |
       |    - Generate Session Keys (from Master Secret)   |
       |    - Verify Client's Finished Message             |
       |    - Send Finished Message                        |
       |<--------------------------------------------------|
       |                                                   |
       | 8. Client Actions:                                |
       |    - Verify Server's Finished Message             |
       |                                                   |
       | 9. Secure Application Data Exchange (Session Key) |
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

**NOTE**: In networking protocols, big-endian is often referred to as "network byte order".

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

## What is CGI?

CGI stands for Common Gateway Interface. It is a standard protocol that allows web servers
to execute external programs or scripts to generate dynamic content for web pages.
CGI scripts can be written in various programming languages, such as Perl, Python, or C.
When a user requests a web page that requires dynamic content,
the web server invokes the CGI script, which processes the request and generates the response.

## What is the Turning Completeness?

Turning Completeness is a concept in computer science
that refers to the ability of a computational system to perform any computation
that can be expressed algorithmically.
A system is considered Turing complete if it can simulate a Turing machine,
which is a theoretical model of computation that can perform any calculation
that can be described by an algorithm.

## What is socket programming?

Socket programming is a way to enable communication between two computers over a network.
Actually, socket is a group of APIs that provided by the operating system
to enable network communication. With these APIs, we can:
1. Create a socket: `socket()`
2. Bind the socket to an IP address and port: `bind()`
3. Listen for incoming connections: `listen()`
4. Accept incoming connections: `accept()`
5. Send and receive data: `send()`, `recv()`

For clients, we can use `connect()` to connect to a server,
and then use `send()` and `recv()` to communicate.

## What is sticky packet problem? How to solve it?

The sticky packet problem occurs in TCP communication
when multiple messages are sent in quick succession,
and the receiver cannot distinguish where one message ends and the next begins.
This can lead to data being "stuck" together in a single read operation,
making it difficult to parse the individual messages.

To solve the sticky packet problem, we can use the following methods:

1. **Fixed-Length Messages**: Define a fixed length for each message.
The receiver reads exactly that many bytes for each message.
2. **Delimiter-Based Messages**: Use a special character or sequence of characters
to indicate the end of a message.
3. **Length-Prefixed Messages**: Prepend each message with a fixed-size header
that specifies the length of the message.
4. **Application-Level Protocols**: Use established protocols like HTTP or WebSocket
that have built-in mechanisms for message framing.

Why sticky package problem not occur in UDP?

> UDP is a connectionless protocol that sends messages, called datagrams,
> without establishing a connection between the sender and receiver.
> Each datagram is sent independently and contains all the necessary information
> (such as source and destination addresses) to be routed through the network.
> Because of this, each UDP datagram is treated as a separate entity,
> and there is no concept of a continuous stream of data like in TCP.
> Therefore, the sticky packet problem does not occur in UDP,
> as each datagram is received in its entirety and can be processed independently.

## What is TCP three-way handshake? Why three-way?

The TCP three-way handshake is a process used to establish a reliable connection
between a client and a server over a TCP/IP network. The three-way handshake involves three steps:

1. **SYN**: The client sends a TCP segment with the SYN (synchronize) flag set
to the server, indicating that it wants to establish a connection.
2. **SYN-ACK**: The server responds with a TCP segment
that has both the SYN and ACK (acknowledge) flags set. The SYN flag indicates
that the server is willing to establish a connection,
and the ACK flag acknowledges the client's initial SYN request.
3. **ACK**: The client sends a final TCP segment with the ACK flag set,
acknowledging the server's SYN-ACK response. At this point, the connection is established,
and data can be exchanged between the client and server.

The reason for using a three-way handshake instead of a two-way handshake
is to ensure that both the client and server are ready to communicate
and to prevent certain types of attacks, such as SYN flooding.

In a two-way handshake, the client would send a SYN request,
and the server would respond with a SYN-ACK.
However, if the SYN-ACK response is lost or delayed,
the client will try to resend the SYN request,
potentially leading to multiple connections being established.

In a four-way handshake, the last handshake is redundant,
as the three-way handshake already ensures that both parties are ready to communicate:

* For the first handshake, the client indicates that it wants to communicate.
* For the second handshake, the server indicates that it is ready to communicate.
And this also ensure that the server can receive data from the client.
* For the third handshake, the client let the server know that "I know you are ready."
And this also ensure that the client can receive data from the server.

## References

- [An overview of the SSL/TLS handshake](https://www.ibm.com/docs/en/ibm-mq/9.3.x?topic=tls-overview-ssltls-handshake)
- [Debounce vs throttle](https://www.kaidohussar.dev/posts/debounce-vs-throttle)
- [Streamlining Test Case Execution: Understanding P0 to P4 Prioritization](https://www.linkedin.com/pulse/streamlining-test-case-execution-understanding-p0-p4-sai-teja-vuthuri-ywvoc)
