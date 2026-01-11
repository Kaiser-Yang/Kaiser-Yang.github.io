---
layout: post
title: 多重背包优化
date: 2025-10-24 18:39:56+0800
last_updated: 2025-10-24 18:39:56+0800
description: 本文介绍多重背包问题的两种优化方法。
tags:
  - Knapsack Problem
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

多重背包问题可以通过如下的转移方程来解决：

$$
dp_{i,j} = \max(dp_{i-1,j-k \cdot w_i} + k \cdot v_i) \quad (0 \leq k \leq c_i \text{且} j - k \cdot w_i \geq 0)
$$

其中，$$ dp_{i,j} $$ 表示前 $$ i $$ 种物品放入容量为 $$ j $$ 的背包所能获得的最大价值，
$$ w_i $$ 和 $$ v_i $$ 分别表示第 $$ i $$ 种物品的重量和价值，$$ c_i $$ 表示第 $$ i $$ 种物品的数量。

通过上面的方法进行转移时，时间复杂度为 $$ O(N \cdot M \cdot C) $$，
其中 $$ N $$ 是物品种类数，$$ M $$ 是背包容量，$$ C $$ 是物品数量的最大值。
当 $$ C $$ 较大时，时间复杂度会变得非常高，这里介绍两种优化方法来降低时间复杂度。

## 二进制优化

对于第 $$ i $$ 件物品我们将其拆分成 $$ 1, 2, 4, \ldots, 2^k $$ 件，使得
$$ 1 + 2 + 4 + \ldots + 2^k \leq c_i $$，
再加上剩余的 $$ x := c_i - (1 + 2 + 4 + \ldots + 2^k) $$ 件，其中的 $$ k $$ 满足
$$ 2^{k+1} > c_i $$。

经过上述的拆分后，我们可以将多重背包问题转化为 0-1 背包问题，从而将时间复杂度降低到
$$ O(N \cdot M \cdot \log C) $$。

转换的依据是 $$ 1, 2, 4, \ldots, 2^k $$ 件物品可以组成 $$0, 1, 2, \ldots, 2^{k+1} - 1$$ 件物品，
而加上剩余的 $$ x $$ 件物品后，可以组成 $$ 0, 1, 2, \ldots, 2^{k+1} - 1 + x = c_i $$ 件物品。

综上所述，这里使用 0-1 背包的方式进行转移依然可以覆盖所有的情况。这里给出代码：

```cpp
for (int i = 0; i < N; ++i) {
    int count = c[i];
    for (int k = 1; count > 0; k <<= 1) {
        int num = std::min(k, count);
        int weight = num * w[i];
        int value = num * v[i];
        for (int j = M; j >= weight; --j) {
            dp[j] = std::max(dp[j], dp[j - weight] + value);
        }
        count -= num;
    }
}
```

## 单调栈优化

我们回到最开始的转移方程：

$$
dp_{i,j} = \max(dp_{i-1,j-k \cdot w_i} + k \cdot v_i) \quad (0 \leq k \leq c_i \text{且} j - k \cdot w_i \geq 0)
$$

在上面的方程中我们可以发现 $$ dp_{i, j} $$ 只依赖于 $$ dp_{i-1, j-k \cdot w_i} $$，
因此我们可以以 $$ w_i $$ 为步长将 $$ j $$ 分成若干组，
即 $$ j \equiv r \text{mod} w_i $$，其中 $$ 0 \leq r < w_i $$。

对于每一组，我们可以将其转移方程改写为：

$$
dp_{i, r + k \cdot w_i} = \max(dp_{i-1, r + k' \cdot w_i} + (k - k') \cdot v_i) \quad (0 \leq k - k' \leq c_i)
$$

整理一下可以得到：

$$
dp_{i, r + k \cdot w_i} = \max(dp_{i-1, r + k' \cdot w_i} - k' \cdot v_i) + k \cdot v_i \quad (0 \leq k - k' \leq c_i)
$$

不难发现上面的 $$ \max(dp_{i-1, r + k' \cdot w_i} - k' \cdot v_i) $$ 可以通过单调队列来进行优化，
从而将时间复杂度降低到 $$ O(N \cdot M) $$。

这里给出代码：

```cpp
for (int i = 0; i < N; ++i) {
    std::vector<int> ndp(M + 1);
    for (int r = 0; r < w[i]; ++r) {
        std::deque<int> q;
        for (int k = 0, j = r; j <= M; k++, j += w[i]) {
            auto val = dp[j] - k * v[i];
            while (!q.empty() && q.front() < k - c[i]) { q.pop_front(); }
            while (!q.empty() && dp[r + q.back() * w[i]] - q.back() * v[i] <= val) { q.pop_back(); }
            q.push_back(k);
            ndp[j] = dp[r + q.front() * w[i]] + (k - q.front()) * v[i];
        }
    }
    dp = std::move(ndp);
}
```
