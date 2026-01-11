---
layout: post
title: 扩展欧几里德算法
date: 2025-10-14 18:42:49+0800
last_updated: 2025-10-14 18:42:49+0800
description: 本文介绍扩展欧几里德算法的原理及应用。
tags:
  - 中文文章
  - Extended Euclidean Algorithm
  - gcd
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

## 前置知识

裴蜀定理（Bézout's Identity）：
对于任意两个整数 $$ a $$ 和 $$ b $$，存在整数 $$ x $$ 和 $$ y $$ 使得
$$ ax + by = \gcd(a, b) $$ 成立。

欧几里德算法：对于两个非负整数 $$ a $$ 和 $$ b $$，其最大公约数可以通过以下递归关系计算：

$$
gcd(a, b) = gcd(b, a \text{mod} b)
$$

## 线性丢番图方程

线性丢番图方程是形如 $$ ax + by = c $$ 的不定方程，
其中 $$ a $$、$$ b $$ 和 $$ c $$ 是已知整数，$$ x $$ 和 $$ y $$ 是未知整数。

根据裴蜀定理，线性丢番图方程有整数解的充分必要条件是 $$ \gcd(a, b) $$ 整除 $$ c $$。

## 扩展欧几里德算法

扩展欧几里德算法是用来求解 $$ ax + by = \gcd(a, b) $$ 的一组特解。

算法的基本思想是利用欧几里德算法的递归结构，同时在每一步记录下 $$ x $$ 和 $$ y $$ 的变化。

考虑当我们已经知道 $$ gcd(b, a \text{mod} b) = bx_1 + (a \text{mod} b)y_1 $$ 的解时，
如何求出 $$ gcd(a, b) = ax + by $$ 的解。

我们记 $$ a \text{mod} b = a - \lfloor \frac{a}{b} \rfloor \cdot b $$，
则有：

$$
gcd(b, a \text{mod} b) = bx_1 + (a - \lfloor \frac{a}{b} \rfloor \cdot b)y_1
$$

注意到 $$ gcd(b, a \text{mod} b) = gcd(a, b) $$，我们可以将上式改写为：

$$
gcd(a, b) = ay_1 + b(x_1 - \lfloor \frac{a}{b} \rfloor \cdot y_1) = ax + by
$$

从而得到：

$$
\left\{
\begin{array}{l}
x = y_1 \\
y = x_1 - \lfloor \frac{a}{b} \rfloor \cdot y_1
\end{array}
\right.
$$

而不难发现当 $$ b = 0 $$ 时，$$ gcd(a, 0) = a $$，此时方程的解为 $$ (1, 0) $$。

这样我们就可以通过递归的方式来求解 $$ ax + by = \gcd(a, b) $$。

对于 $$ ax + by = c $$ 的情况，我们只需要先求出 $$ ax_0 + by_0 = \gcd(a, b) $$ 的一组解，
此时 $$ x = x_0 \cdot \frac{c}{\gcd(a, b)} $$，$$ y = y_0 \cdot \frac{c}{\gcd(a, b)} $$
即为 $$ ax + by = c $$ 的一组特解。

接下来我们考虑在知道一组解的情况下，如何求出通解。这里先给出结论：

若 $$ (x_0, y_0) $$ 是 $$ ax + by = c $$ 的一组解，则通解可以表示为：

$$
\left\{
\begin{array}{l}
x = x_0 + k \cdot \frac{b}{\gcd(a, b)} \\
y = y_0 - k \cdot \frac{a}{\gcd(a, b)}
\end{array}
\right.
$$

容易证明上述形式的解都满足方程。我们接下来证明所有解都可以表示为上述形式。

设 $$ (x_1, y_1) $$ 也是方程的解，则有 $$ a(x_1 - x_0) + b(y_1 - y_0) = 0 $$ 成立。

由此可得

$$
a(x_1 - x_0) = -b(y_1 - y_0)
$$

两边同时除以 $$ \gcd(a, b) $$，有

$$
\frac{a}{\gcd(a, b)}(x_1 - x_0) = -\frac{b}{\gcd(a, b)}(y_1 - y_0)
$$

可以知道

$$
\frac{a}{\gcd(a, b)} \mid \frac{b}{\gcd(a, b)}(y_1 - y_0)
$$

因为 $$ \frac{a}{\gcd(a, b)} $$ 和 $$ \frac{b}{\gcd(a, b)} $$ 互质，所以

$$
\frac{a}{\gcd(a, b)} \mid  y_1 - y_0
$$

即存在整数 $$ k $$ 使得：

$$
y_1 - y_0 = k \cdot \frac{a}{\gcd(a, b)}
$$

即

$$
y_1 = y_0 + k \cdot \frac{a}{\gcd(a, b)}
$$

代入前面的等式，有：

$$
a(x_1 - x_0) = -b \cdot k \cdot \frac{a}{\gcd(a, b)}
$$

即

$$
x_1 = x_0 - k \cdot \frac{b}{\gcd(a, b)}
$$

综上所述，所有解可以表示为上述通解的形式。

最后给出扩展欧几里德算法的代码实现：

```cpp
// return the greatest common divisor of a and b,
// and find x and y such that ax + by = gcd(a, b)
template <typename T>
static T ex_gcd(T a, T b, T &x, T &y) {
    if (b == 0) {
        x = 1;
        y = 0;
        return a;
    } else {
        T d = ex_gcd(b, a % b, y, x);
        y -= (a / b) * x;
        return d;
    }
}
```
