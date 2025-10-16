---
layout: post
title: Miller Rabin 素数测试
date: 2025-10-13 21:11:22+0800
last_updated: 2025-10-13 21:11:22+0800
description: 本文介绍 Miller Rabin 素数测试的原理及其实现方式。
tags:
  - Miller Rabin
  - Primality Test
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 前置知识

### 费马小定理（Fermat's Little Theorem）

如果 $$ p $$ 是一个素数，且 $$ a $$ 不是 $$ p $$ 的倍数，
则 $$ a^{p-1} \equiv 1 \text{mod} p $$。

### 二次探测定理（Quadratic Residue Theorem）

如果 $$ p $$ 是一个素数，且 $$ x^2 \equiv 1 \text{mod} p $$，
则 $$ x \equiv 1 \text{mod} p $$ 或 $$ x \equiv -1 \text{mod} p $$。

## Miller Rabin 素数测试原理

我们可以发现，费马小定理和二次探测定理都给出了素数的必要条件，
但并不是充分条件。也就是说，如果一个数不满足这些条件，
那么它一定不是素数。

Miller Rabin 素数测试是一种基于概率的素数测试算法，
它通过多次随机选择基数 $$ a $$ 来验证一个数是否为素数。

对于一个奇数 $$ n $$ 而言其可以被写成 $$ n - 1 = 2^s \cdot d $$ 的形式，
其中 $$ d $$ 是奇数，$$ s \geq 1 $$。
根据费马小定理，如果 $$ n $$ 是素数，
则对于任意 $$ a $$，都有 $$ a^{2^s \cdot d} \equiv 1 \text{mod} n $$。
而由二次探测定理可知，我们可以对 $$ a^{2^s \cdot d} \equiv 1 \text{mod} n $$
执行开方的操作，其结果一定要是 $$ 1 $$ 或 $$ n - 1 $$。
同时当其结果为 $$ 1 $$ 时，且当前还可以进行开方操作时，
则继续进行开方操作，直到结果为 $$ n - 1 $$ 或无法继续开方为止。
如果在某次开方的过程中，结果既不是 $$ 1 $$ 也不是 $$ n - 1 $$，
则 $$ n $$ 一定不是素数。

根据上面的流程我们可以选择多个不同的 $$ a $$ 来进行测试，
如果所有的测试都通过了，则 $$ n $$ 很可能是素数。

特别地，对于64位无符号整数，选择 $$ 2, 325, 9375, 28178, 450775, 9780504, 1795265022 $$
可以保证不会出现伪素数。

另外在实现的过程中，我们往往不会进行开方的操作，取而代之的是平方操作：

1. 将待测试的数 $$ n $$ 表示为 $$ n - 1 = 2^s \cdot d $$，其中 $$ d $$ 是奇数，$$ s \geq 1 $$。
2. 选择一个基数 $$ a $$。
3. 计算 $$ x = a^d \text{mod} n $$。
4. 如果 $$ x \equiv 1 \text{mod} n $$ 或 $$ x \equiv n - 1 \text{mod} n $$，此时进行平方的结果一定是 $$ 1 $$，
   所以可以直接认为通过本轮的测试。
5. 否则，重复以下步骤 $$ s - 1 $$ 次：
   - 计算 $$ x \leftarrow x^2 \text{mod} n $$。
   - 如果 $$ x \equiv n - 1 \text{mod} n $$，则通过本轮测试。
6. 如果所有测试都未通过，则 $$ n $$ 不是素数。

在上述过程的5中，我们只检查了结果是否等于 $$ n - 1 $$，而没有检查结果是否等于 $$ 1 $$。
这是因为如果当前的结果第一次等于 $$ 1 $$，则说明在前一次平方操作中，
结果既不是 $$ 1 $$ 也不是 $$ n - 1 $$，这就违背了二次探测定理。

最后给出Miller Rabin素数测试的代码：

```cpp
template <typename T>
static bool is_prime(T n) {
    if (n < 2) { return false; }
    if (n == 2) { return true; }
    if (n % 2 == 0) { return false; }
    int s = 0;
    T d = n - 1;
    while (d % 2 == 0) {
        d >>= 1;
        s++;
    }
    for (auto &&a : miller_rabin_test) {
        if (a % n == 0) { continue; }
        auto x = pow(a, d, n);
        if (x == 1 || x == n - 1) { continue; }
        bool ok = false;
        for (int r = 1; r < s; r++) {
            x = pow(x, 2, n);
            if (x == n - 1) {
                ok = true;
                break;
            }
        }
        if (!ok) { return false; }
    }
    return true;
}
```
