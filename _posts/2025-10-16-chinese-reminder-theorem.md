---
layout: post
title: 中国剩余定理及其扩展
date: 2025-10-16 15:22:05+0800
last_updated: 2025-10-16 15:22:05+0800
description: 本文介绍中国剩余定理以及扩展中国剩余定理。
tags:
  - Chinese Remainder Theorem
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 中国剩余定理

假如给定一组同余方程：

$$
\begin{cases}
x \equiv a_1 \mod m_1 \\
x \equiv a_2 \mod m_2 \\
\vdots \\
x \equiv a_k \mod m_k
\end{cases}
$$

其中 $$ m_1, m_2, \ldots, m_k $$ 两两互质，下面介绍如何求解该组方程。

我们令 $$ M = m_1 \cdot m_2 \cdots m_k $$，并且对于每个 $$ i $$，定义 $$ M_i = \frac{M}{m_i} $$。
由于 $$ m_i $$ 和 $$ M_i $$ 互质，可以知道 $$ M_i $$ 在模 $$ m_i $$ 意义下是有逆元的，
记为 $$ M_i^{-1} $$。不难发现 $$ M_i \cdot M_i^{-1} \equiv 1 \mod m_i $$ 和
$$ M_i \cdot M_i^{-1} \equiv 0 \mod m_j $$（$$ j \neq i $$）都成立。
记 $$ c_i = a_i \cdot M_i \cdot M_i^{-1} $$，则有：

$$
\begin{cases}
c_i \equiv a_i \mod m_i \\
n_i \equiv 0 \mod m_j \quad (j \neq i)
\end{cases}
$$

由同余的线性性质可知，$$ x' = \sum_{i=1}^{k} c_i $$ 即为一个特解。

不难证明通解可以写成 $$ x = x' + t \cdot M \quad (t \in \mathbb{Z}) $$。

注意在上述的过程中，我们要求解 $$ M_i $$ 在模 $$ m_i $$ 意义下的逆元，
只有在 $$ m_i $$ 两两互质的情况下才能保证每次求解的逆元存在。

代码如下：

```cpp
template <typename T>
static void crt(const std::vector<T> &a, const std::vector<T> &m, T &x, T &l) {
    assert(a.size() == m.size());
    x = 0;
    l = 1;
    for (auto &&e : m) { l *= e; }
    for (int i = 0; i < a.size(); i++) {
        x = (x + a[i] * (l / m[i]) % l * inverse_of(l / m[i], m[i]) % l) % l;
    }
}
```

## 扩展中国剩余定理

扩展中国剩余定理是在中国剩余定理的基础上，放宽了对模数的互质要求。
假设给定一组同余方程：

$$
\begin{cases}
x \equiv a_1 \mod m_1 \\
x \equiv a_2 \mod m_2 \\
\vdots \\
x \equiv a_k \mod m_k
\end{cases}
$$

并不保证 $$ m_1, m_2, \ldots, m_k $$ 两两互质，下面介绍如何求解该组方程。

不难发现，对于 $$ x = x' + t \cdot M \quad (t \in \mathbb{Z}) $$，
其是同余方程 $$ x \equiv x' \mod M $$ 的解。
现在假设我们已经求出了前 $$ i - 1 $$ 个方程的解
$$ x = x_{i-1} + t \cdot M_{i-1} \quad (t \in \mathbb{Z}) $$，
也就是 $$ x \equiv x_{i-1} \mod M_{i-1} $$ 的解，
现在考虑如何求其与 $$ x \equiv a_i \mod m_i $$ 的解。
将当前的解代入到第二个方程中，有：
$$ x_{i-1} + t \cdot M_{i-1} \equiv a_i \mod m_i $$，
即 $$ t \cdot M_{i-1} \equiv a_i - x_{i-1} \mod m_i $$。
设 $$ d = \gcd(M_{i-1}, m_i) $$，则上式有解的充分必要条件是 $$ d \mid (a_i - x_{i-1}) $$。
如果有解，则可以将上式两边同时除以 $$ d $$，得到
$$ t \cdot \frac{M_{i-1}}{d} \equiv \frac{a_i - x_{i-1}}{d} \mod \frac{m_i}{d} $$。
由于 $$ \frac{M_{i-1}}{d} $$ 和 $$ \frac{m_i}{d} $$ 互质，
所以可以使用扩展欧几里德算法来求解该方程，得到 $$ t_0 $$ 为一个特解。
则通解可以表示为
$$ t = t_0 + k \cdot \frac{m_i}{d} \quad (k \in \mathbb{Z}) $$。
将其代入到 $$ x = x_{i-1} + t \cdot M_{i-1} $$ 中，有
$$ x = x_{i-1} + t_0 \cdot M_{i-1} + k \cdot \frac{m_i}{d} \cdot M_{i-1} \quad (k \in \mathbb{Z}) $$。
因此，新的解可以表示为
$$ x = x_i + k \cdot M_i \quad (k \in \mathbb{Z}) $$，
其中 $$ x_i = x_{i-1} + t_0 \cdot M_{i-1} $$，$$ M_i = \frac{m_i}{d} \cdot M_{i-1} $$。

特别地，我们可以增加 一个方程 $$ x \equiv 0 \mod 1 $$ 作为初始条件，
此时 $$ x_0 = 0 $$，$$ M_0 = 1 $$。这里给出代码：

```cpp
template <typename T>
static void ex_crt(const std::vector<T> &a, const std::vector<T> &m, T &x, T &l) {
    x = 0;
    l = 1;
    for (int i = 0; i < a.size(); i++) {
        T t0, _;
        T d = gcd(l, m[i]);
        if ((a[i] - x) % d != 0) {
            x = l = -1;
            return;
        }
        ex_gcd(l / d, m[i] / d, t0, _);
        t0 = (a[i] - x) / d * t0 % (m[i] / d);
        t0 = (t0 % (m[i] / d) + (m[i] / d)) % (m[i] / d);
        x = x + t0 * l;
        l = m[i] / d * l;
        x %= l;
    }
}
```
