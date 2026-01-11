---
layout: post
title: 模逆元
date: 2025-10-15 21:34:31+0800
last_updated: 2025-10-15 21:34:31+0800
description: 本文介绍模逆元的定义及其计算方法。
tags:
  - 中文文章
  - Modular Inverse
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

模逆元的定义如下：

给定一个正整数 $$ a $$ 和一个正模数 $$ p $$ ，如果存在一个正整数 $$ b $$ 满足：

$$
a \cdot b \equiv 1 \ (\text{mod} \ p), 1 \leq b \le p
$$

则称 $$ b $$ 为 $$ a $$ 关于模数 $$ p $$ 的模逆元，记作 $$ a^{-1} \ (\text{mod} \ p) $$ 。

## 计算方法

### 单个数的模逆元

对于模逆元的计算，其本质是在求解 $$ ax + py = 1 $$ 这个不定方程的正整数解 $$ x $$。

由裴蜀定理可知，只有当 $$ \gcd(a, p) = 1 $$ 时，方程才有整数解。

所以我们可以使用扩展欧几里得算法来计算模逆元。

这里给出计算代码：

```cpp
// return the inverse of a modulo mod
template <typename T>
static T inverse_of(T a, T mod) {
    T x, y;
    (void)ex_gcd(a, mod, x, y);
    return (x % mod + mod) % mod;
}
```

特别地，当 $$ p $$ 是质数时，由费马小定理可知：

$$
a^{p-1} \equiv 1 \ (\text{mod} \ p)
$$

因此，$$ a^{p-2} \ (\text{mod} \ p) $$ 即为 $$ a $$ 关于模数 $$ p $$ 的模逆元。

### 多个数的模逆元

逆元存在一个重要的性质：对于 $$ a $$ 和 $$ b $$，有

$$
(a \cdot b)^{-1} \equiv a^{-1} \cdot b^{-1} (\text{mod} \ p)
$$

有了这个性质，我们可以通过预处理前缀积来计算多个数的模逆元。

具体地，我们用 $$ prod_i $$ 表示 $$ [1, i) $$ 的前缀积取模后的结果，即：

$$
prod_i = a_{i-1} \cdot prod_{i-1} \ (\text{mod} \ p)
$$

那么就有 $$ a^{-1}_i \equiv prod^{-1}_{i+1} \cdot prod_i \ (\text{mod} \ p) $$。

我们只需要计算出 $$ prod_n $$ 的模逆元，然后从后往前依次计算出每个数的模逆元即可。

这里给出代码：

```cpp
// return the inverse of each element in a modulo mod
template <typename T>
static std::vector<T> inverse(const std::vector<T> &a, T mod) {
    std::vector<T> prod(a.size() + 1, 1);
    for (int i = 0; i < a.size(); i++) { prod[i + 1] = prod[i] * (a[i] % mod) % mod; }
    std::vector<T> inv(a.size());
    auto s = inverse_of(prod.back(), mod);
    for (int i = a.size() - 1; i >= 0; i--) {
        inv[i] = s * prod[i] % mod;
        s = s * a[i] % mod;
    }
    return inv;
}
```
