---
layout: post
title: IEEE Xtreme 18.0 题解
date: 2024-10-28 17:10:49+0800
last_updated: 2025-04-15 16:02:12+0800
description:
tags:
  - Algorithm
  - IEEExtreme
categories: Algorithm
featured:
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

# [Two Fridges](https://csacademy.com/ieeextreme-practice/task/two-fridges)

由于题目中的温度范围非常小，我们只需要从小到大枚举温度，对于每个枚举，检查是否所有区间都被覆盖。
第一个覆盖所有区间的温度对即是答案。如果找不到输出 $$ -1 $$ 即可。

代码：[two_fridges.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/two_fridges.cpp)

# [Star Road](https://csacademy.com/ieeextreme-practice/task/star-road)

对于本题我们可以使用线段树和线段树的合并来解决这个问题。

具体的，我们首先需要将 $$ star $$ 离散化，使得其值在 $$ [1, len] $$ 之间，
其中 $$ len $$ 是不同的星星的个数。
然后我们从任意一个结点开始进行 `DFS`，当我们到达一个结点时，我们为这个结点建立一个线段树。

对于线段树的叶子结点 (其表示的区间设为 $$ [l, l] $$ ) ，它会保存两个值：

* $$ LIS $$：以 $$ l $$ 结尾的最长递增子序列的长度。
* $$ LDS $$：以 $$ l $$ 结尾的最长递减子序列的长度。

对于线段树的非叶子结点 (其表示的区间为 $$ [l, r] $$ )，它会保存两个值：

* $$ LIS $$：其子结点的 $$ LIS $$ 的最大值。
* $$ LDS $$：其子结点的 $$ LDS $$ 的最大值。

当我们开始回溯时，我们可以得到：

* $$ son[i].LIS $$：子结点 $$ i $$ 所在线段树在区间 $$ [1, star[u] - 1] $$ 的 $$ LIS $$。
* $$ son[i].LDS $$：子结点 $$ i $$ 所在线段树在区间 $$ [star[u] + 1, len] $$ 的 $$ LDS $$。

我们使用 $$ star[u] - 1 $$ 和 $$ star[u] + 1 $$ 是为了保证 $$ star[u] $$ 可以被选中，
因此如果我们选中 $$ u $$，
那么我们可以得到 $$ ans = max(ans, son[i].LIS + 1 + son[j].LDS), i \ne j $$，
这里可以分别按照 $$ LID $$ 和 $$ LDS $$ 排序来规避掉枚举 $$ i, j $$ 的问题。

那么如果我们不选中 $$ u $$，我们如何得到这部分的答案呢？
我们可以在合并线段树的过程中解决这个问题。
在合并线段树 $$ a $$ 和线段树 $$ b $$ 时，设我们当前处于区间 $$ [l, r] $$，
我们可以得到 $$ a $$ (或 $$ b $$) 在 $$ [l, mid] $$ 区间的 $$ LIS $$，
以及 $$ b $$ (或 $$ a $$) 的 $$ [mid + 1, r] $$ 区间的 $$ LDS $$，
那么这两部分可以合并，这些合并包括了不选中 $$ u $$ 的部分。
也就是 $$ ans = max(ans, LIS[lc[a]] + LDS[rc[b]], LIS[lc[b]] + LDS[rc[a]]) $$。

当我们完成了所有的结点的合并后，我们需要进行两次单点更新：

* 如果 $$ max(son[i].LIS + 1) $$ 比线段树 $$ u $$ 在 $$ star[u] $$ 处的 $$ LIS $$ 大，
我们需要将其更新为 $$ max(son[i].LIS + 1) $$，
这表示以 $$ star[u] $$ 结尾的最长递增子序列的长度发生了变化。
* 如果 $$ max(son[i].LDS + 1) $$ 比线段树 $$ u $$ 在 $$ star[u] $$ 处的 $$ LDS $$ 大，
我们需要将其更新为 $$ max(son[i].LDS + 1) $$，
这表示以 $$ star[u] $$ 结尾的最长递减子序列的长度发生了变化。

代码：[star_road.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/star_road.cpp)

# [Increasing table](https://csacademy.com/ieeextreme-practice/task/increasing-table)

考虑到当第一行的元素确定后，第二行的元素也就确定了，因此我们只需要计算第一行的方案数即可。
对于输入，我们可以维护一个序列，序列中的元素表示第一行可以填入的数，
以及这个数是否一定要填入到第一行，序列按照可以填入的数从小到大排序。
现在我们需要计算从这个序列中选出 $$ N $$ 个数的方案数。
如果我们不考虑每一列是否满足递增关系，那么我们可以很轻松的用动态规划解决这个问题。具体的，
用 $$ dp[i][j][0] $$ 表示前 $$ i $$ 个数中选出 $$ j $$ 个数，且第 $$ i $$ 个数不被选择时的方案数；
用 $$ dp[i][j][1] $$ 表示前 $$ i $$ 个数中选出 $$ j $$ 个数，且第 $$ i $$ 个数被选择时的方案数。

$$
\begin{aligned}
dp[i][j][0] & = \begin{cases}
0，\text{当前必选} \\
dp[i - 1][j][0] + dp[i - 1][j][1]，\text{其他}
\end{cases} \\
dp[i][j][1] & = dp[i - 1][j - 1][0] + dp[i - 1][j - 1][1]
\end{aligned}
$$

我们现在考虑必须要让列也递增的情况，我们来依次来考虑第一行的每个数最大可能的取值：

* 对于第一个数，其只能是 $$ 1 $$。
* 对于第二个数，其最大是 $$ 3 $$，因为当第二个数是 $$ 4 $$ 的时候，
第二行一定得出现 $$ 2，3 $$ 此时第二列一定不可能递增。
* 对于第三个数，其最大是 $$ 5 $$，因为当第三个数是 $$ 6 $$ 的时候，
第二行一定得出现 $$3，4，5 $$ (或者更小的数字), 此时第三列一定不可能递增。
* ……
* 对于第 $$ i $$ 个数，其最大是 $$ 2i - 1 $$。

不难发现，对于一个方案中的第 $$ i $$ 个数，若其均小于等于 $$ 2i - 1 $$，那么方案一定满足列是递增的。

因此，我们只需要对上面的转移方程增加一个限制条件即可：

$$
\begin{aligned}
dp[i][j][0] & = \begin{cases}
0，\text{当前必选} \\
dp[i - 1][j][0] + dp[i - 1][j][1]，\text{其他}
\end{cases} \\
dp[i][j][1] & = \begin{cases}
dp[i - 1][j - 1][0] + dp[i - 1][j - 1][1]，\text{当前小于等于} 2i - 1 \\
0，\text{其他}
\end{cases}
\end{aligned}
$$

代码：[increasing_table.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/increasing_table.cpp)

# [Bounded tuple]()

不会。

# [Laser Defense](https://csacademy.com/ieeextreme-practice/task/laser-defense)

我们将激光分成四种：

* $$ a_u $$：$$ a_u[i] $$ 表示 $$ A $$ 点发出的第 $$ i $$ 条与上边界的交点座标。
* $$ a_r $$：$$ a_r[i] $$ 表示 $$ A $$ 点发出的第 $$ i $$ 条与右边界的交点座标。
* $$ b_u $$：$$ b_u[i] $$ 表示 $$ B $$ 点发出的第 $$ i $$ 条与上边界的交点座标。
* $$ b_l $$：$$ b_l[i] $$ 表示 $$ B $$ 点发出的第 $$ i $$ 条与左边界的交点座标。

我们分别对上面四种激光进行排序。

我们首先考虑只有 $$ A $$ 点激光的情况，此时被分成了 $$ len(a_u) + len(a_r) + 1 $$ 个区域。

接着对于某个 $$ B $$ 点发出的激光，如果其到达左边界，
那么此时区域个数会增加 $$ len(a_u) + len(a_r) + 1 $$；如果其到达上边界，
此时我们可以用二分查找在 $$ a_u $$ 中找到有多少个激光出现在其左边，不妨设为 $$ x $$，
那么此时区域个数会增加 $$ len(a_u) + len(a_r) + 1 - x $$。

代码：[laser_defense.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/laser_defense.cpp)

# [Another Sliding Window Problem](https://csacademy.com/ieeextreme-practice/task/another-sliding-window-problem)

首先我们不难发现要获得一个序列的 `optimal cost`，如果序列有偶数个元素，那么最大的要和最小的配对，
第二大的要和第二小的配对，以此类推；如果序列有奇数个元素，那么最大的元素要单独拎出来，
而其余元素按照序列有偶数个元素的情况处理。

有了上面的结论，我们可以有以下推论：

> 如果一个序列的 `optimal cost` 小于等于 $$ x $$，
那么删除最大的元素后的序列的 `optimal cost` 一定小于等于 $$ x $$。

证明如下：

> 如果一开始序列有奇数个元素，那么删除最大的元素后，序列有偶数个元素，
那么 `optimal cost` 的取值集合少了最后一个元素，因此 `optimal cost` 不会增加。
如果一开始序列有偶数个元素，那么删除最大的元素后，序列有奇数个元素，
那么 `optimal cost` 的取值集合中某一个元素变小了，因此 `optimal cost` 不会增加。

利用上面的推论，当我们计算出一个满足 `optimal cost` 小于等于 $$ x $$ 的区间 $$ [l, r] $$ 后，
那么 $$ [l, r - 1], [l, r - 2], \cdots, [l, l] $$ 一定也满足 `optimal cost` 小于等于 $$ x $$，
如果我们用 $$ s[i] $$ 表示 $$ [1, i] $$ 的前缀和，
那么这些区间的对答案的贡献为：$$ s[r] - s[l - 1] - (r - l + 1) \times a[l] $$。

接下来我们考虑对于一个给定的 $$ l $$，
如何快速求出最大的 $$ r $$ 使得 $$ [l, r] $$ 满足 `optimal cost` 小于等于 $$ x $$。

我们还是需要利用 `optimal cost` 的配对性质，首先我们先找到最后一个小于等于 $$ x $$ 的元素的位置，
不妨设为 $$ r $$，初始 $$ l = r $$，那么对于当前的区间 $$ [l, r] $$ 中的 $$ r $$ 一定是满足
`optimal cost` 小于等于 $$ x $$ 最大的。接下来我们考虑如何求解 $$ l - 1 $$ 对应的最大的 $$ r' $$。
实际上 $$ r' $$ 只有可能是 $$ \{r - 1, r, r + 1\} $$ 中的一个。这是因为：

* 如果 $$ [l, r] $$ 的长度是偶数，那么 $$ [l - 1, r] $$ 的长度是奇数，
此时 $$ [l - 1, r] $$ 一定是满足 `optimal cost` 小于等于 $$ x $$ 的，
因为此时可以让 $$ a[l - 1] $$ 单独一组；同时按照匹配规则，
如果 $$ a[l - 1] + a[r + 1] \le x $$, 
那么 $$ [l - 1, r + 1] $$ 也一定是满足 `optimal cost` 小于等于 $$ x $$ 的。
而对于 $$ [l - 1, r + 2] $$ 一定是不满足的，因为如果该区间满足，
那么 $$ [l - 1, r + 1] $$ 也一定满足，而 $$ [l - 1, r + 1] $$ 长度是偶数，
此时我们把 $$ a[l - 1] $$ 删除掉，那么 $$ [l, r + 1] $$ 一定满足，这与 $$ r $$ 是最大的矛盾。
所以当前情况下 $$ r' $$ 只有可能是 $$ \{r, r + 1\} $$ 中的一个。
* 如果 $$ [l, r] $$ 的长度是奇数，那么 $$ [l - 1, r] $$ 的长度是偶数，
此时如果 $$ a[l - 1] + a[r] \le x $$，
那么 $$ [l - 1, r] $$ 一定是满足 `optimal cost` 小于等于 $$ x $$ 的，
进一步的有如果 $$ a[r + 1] \le x $$ 成立，
那么 $$ [l - 1, r + 1] $$ 也一定是满足 `optimal cost` 小于等于 $$ x $$ 的。
而对于 $$ [l - 1, r + 2] $$ 一定是不满足的，因为如果该区间满足，
而 $$ [l - 1, r + 2] $$ 长度是偶数，
此时我们把 $$ a[l - 1], a[r + 2] $$ 同时删除掉，那么 $$ [l, r + 1] $$ 一定满足，
这与 $$ r $$ 是最大的矛盾。而如果一开始就不满足 $$ a[l - 1] + a[r] \le x $$，
那么由于 $$ [l, r] $$ 区间长度是奇数，我们可以删除 $$ a[r] $$，增加 $$ a[l - 1] $$，
由于 $$ [l, r] $$ 满足，
我们用一个更小的数字取替换了一个最大的数字形成的 $$ [l - 1, r - 1] $$ 一定满足。
所以当前情况下 $$ r' $$ 只有可能是 $$ \{r - 1, r, r + 1\} $$ 中的一个。

因此我们只需要按照上面的过程，每次减少 $$ l $$ 后，计算出新的 $$ r $$ 对答案进行统计即可。

代码：[another_sliding_window_problem.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/another_sliding_window_problem.cpp)

# [IEEE754 Emulator](https://csacademy.com/contest/ieeextreme-practice/task/ieee754-emulator/)

逻辑并不复杂，有几点需要注意的：

* 不能直接计算 $$ a * b + c $$，应该使用相关的库函数，
例如 `C++` 的 `std::fma`，`Python` 的 `Math.fma` 等。
* 类型之间的转换，例如 `int` 和 `float` 之间的转换，应该使用 `union` 或者 `memcpy` 等方法，
* 输出要保留前导零，也就是在结果长度小于 `8` 的时候需要在前面补零。

代码：[ieee754_emulator.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/ieee754_emulator.cpp)

# [Triumvirates]()

不会。

# [Stick](https://csacademy.com/ieeextreme-practice/task/stick)

除去第一个正方形外，每增加一个正方形，所增加的面积是一个定值，
其增加值为单个正方形的面积减去两个正方形的公共部分的面积。因此最终答案为 $$ 4NL^2 - (N-1)S $$。
其中 $$ S $$ 为两个连续正方形相交部分的面积。

如果直接使用上面的公式，那么乘法的时候可能会溢出
，因此我们可以将上面的公式进行变形成 $$ N(4L^2 - S) + S $$。
并且最后得使用 `unsigned long long` 才行。当然 `Life is short; you need Python`。

代码：[stick.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/stick.cpp)

# [Increasing-decreasing permutations]()

不会。

# [Cheap Construction](https://csacademy.com/ieeextreme-practice/task/cheap-construction)

首先，对于本题，答案中所选的字符串一定可以是原始串的子串。这是因为如果我们不选择原始串的子串，
那么最终的联通块一定是 $$ N $$，所以对于联通块个数小于 $$ N $$ 情况我们一定要选择原始串的子串。
而对于要让联通块个数为 $$ N $$ 的情况，我们可以选择一个长度为 $$ 1 $$ 的子串。

有了上面的结论我们只需要枚举所有的子串，计算每个子串会产生的联通块个数即可。

对于如何计算一个子串的联通块个数，我们需要先计算出所有子串的出现位置。这一步当然不能用 `KMP` 算法，
如果使用 `KMP` 算法整体复杂度就变成 $$ O(N^3) $$ 了，我们可以直接枚举所有的子串的起始位置，
将其记录下来，而不是单独去计算每个子串的出现位置。

想到这一步之后又存在新的问题，我们不能直接使用一个字典去存储每个子串出现的位置，
因为不管是有序字典还是无序字典，其在对一个字符串进行映射的时候必须要遍历整个字符串，
这样复杂度又变成了 $$ O(N^3) $$。

正确的做法是使用字符串哈希，这样可以在 $$ O(N) $$ 的复杂度预处理后，
只需要 $$ O(1) $$ 复杂度即可获取任意子串的哈希值 (本题中得使用双哈希)。

经过上面的操作后，我们获取到了每个子串的出现位置，接下来我们考虑如何计算一个子串的联通块个数。
首先我们需要保证一个子串的出现位置是从小到大排序的，这一步可以在前面枚举的时候实现，
接着我们遍历出现位置，同时记录上一个出现位置的结尾 $$ las $$ (即 $$ las\_start\_pos + len - 1$$ )，
比较 $$ las $$ 和当前起始位置 $$ now\_start\_pos $$ ，
如果 $$ las \ge now\_start\_pos $$ ，这意味着当前的子串和上一次的子串有重叠，此时不会产生新的联通块；
如果 $$ las \lt now\_start\_pos $$ ，那么从上一次的结尾到当前位置之前的将会形成新的联通块，
也就是个数会增加 $$ now\_start\_pos - las $$。

最后一点，本题对常数要求非常高，如果你按照上面的思路写，那么会 `TLE`。这里给出一种可行的优化方案：

> 我们并不在一开始枚举所有的子串，而是按照长度递增的方式枚举子串，
每计算出一个长度的所有子串出现位置后就进行一次答案的更新，
这样能够让字典中存储的子串个数不超过 $$ N $$ 个，从而实现常数优化的目的，
即使是这样你也需要注意其他地方的细节，尽可能的优化常数，
因为我的代码在最慢的一个测试点上花费了 $$ 946ms $$，这非常接近 `TLE`。

代码：[cheap_construction.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/cheap_construction.cpp)

# [Disparate Date Sets](https://csacademy.com/ieeextreme-practice/task/disparate-datasets)

本题逻辑实际上没有任何的难点，但是如果我使用以下的方式对输入进行处理，将会出现问题：

```cpp
while (getline(cin, str)) {
    string tmp;
    Record tmp_record;
    int cnt = 0;
    for (auto &&ch : str) {
        if (ch == ',' && cnt % 2 == 0) {
            tmp_record.push_back(tmp);
            tmp = "";
            cnt = 0;
        } else {
            tmp.push_back(ch);
            if (ch == '\"') { cnt++; }
        }
    }
    tmp_record.push_back(tmp);
}
```

而如果我将上面的读入替换成 `Python` 中的 `csv` 进行处理，就能够得到正确的结果。

我目前仍然没有发现上面的读取方式对于合法的输入会存在什么问题。我们已经确定了输入一定满足以下的情况：

* `title` 和 `acronyms` 中开头和结尾一定为单个双引号。
* 除去 `title` 和 `acronyms` 开头和结尾的双引号后，其中不存在奇数个连续的双引号。

如果你发现这段读入会对某些输入产生错误，请告诉我。

代码部分将会提供使用 `csv` 读取数据并处理的 `Python` 代码。

代码：[disparate_datasets.py](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/disparate_datasets.py)

# [Queries](https://csacademy.com/ieeextreme-practice/task/queries)

首先感谢 [cancaneed](https://codeforces.com/profile/cancaneed) 提供的思路。

本题我们可以采用两次分块来进行实现。分块维护区间和。

其中第一个分块用于处理在原始区间上面的更新，我们记为 $$ ori $$，
第二个分块用于维护在 $$ p $$ 上面的更新，我们记为 $$ perm $$。

对于在原始区间上的更新，我们在 $$ ori $$ 上更新后 ($$ ori.update(l, r, c) $$)，
还需要考虑其对 $$ p $$ 上查询的影响。例如，
如果之前存在 $$ [l, r] $$ 区间上的更新增加 $$ c $$，
此时我们查询索引为 $$ p_{l'}, p_{l'+1}, \cdots, p_{r'} $$ 的和，
那么我们需要知道 $$ [l, r] $$ 中有多少个下标在集合 $$ \{p_{l'}, p_{l'+1}, \cdots, p_{r'}\} $$ 中，
如果我们将这个数字记为 $$ cnt $$，那么此次查询就需要增加 $$ c \times cnt $$。
所以在执行原始区间的更新时，我们还需要更新此次操作对 $$ perm $$ 的影响，
我们将这一部分记为 $$ perm\_sum $$，
其中 $$ perm\_sum[i] $$ 表示对原始区间上的更新会使 $$ perm $$ 的第 $$ i $$ 块增加 $$ perm\_sum[i] $$。
具体的, 对于更新 $$ (l, r, c) $$，
我们需要进行如下更新：

$$
perm\_sum[i] = perm\_sum[i] + c \times (perm\_cnt[i][r] - perm\_cnt[i][l - 1]), 0 \le i \le BLOCK\_CNT
$$

这里的 $$ perm\_cnt[i][j] $$ 表示在 $$ ori $$ 的第 $$ i $$ 块维护的下标对应到 $$ p $$ 后，
有多少是小于等于 $$ j $$ 的。

对于在 $$ p $$ 上的更新，同理我们需要考虑其对 $$ ori $$ 的影响。我们将这一部分记为 $$ ori\_sum $$，
其中 $$ ori\_sum[i] $$ 表示对 $$ p $$ 上的更新会使 $$ ori $$ 的第 $$ i $$ 块增加 $$ ori\_sum[i] $$。
具体的，对于更新 $$ (l, r, c) $$，
我们需要进行如下更新：

$$
ori\_sum[i] = ori\_sum[i] + c \times (ori\_cnt[i][r] - ori\_cnt[i][l - 1]), 0 \le i \le BLOCK\_CNT
$$

这里的 $$ ori\_cnt[i][j] $$ 表示在 $$ perm $$ 的第 $$ i $$ 块维护的下标对应到 $$ ori $$ 后，
有多少是小于等于 $$ j $$ 的。

对于原始区间上的查询操作，首先需要查询 $$ ori.query(l, r) $$，
对于覆盖到整个块 $$ id $$ 的部分，我们需要累加 $$ ori\_sum[id] $$，对于部分块，
我们必须依次在 $$ perm $$ 上进行单点查询，
因此我们需要维护 $$ inv\_p[i] $$ 表示原始下标 $$ i $$ 在 $$ perm $$ 上的位置，
即 $$ inv\_p[p[i]] = i $$。

对于 $$ p $$ 上的查询操作，首先需要查询 $$ perm.query(l, r) $$，
对于覆盖到整个块 $$ i $$ 的部分，我们需要累加 $$ perm\_sum[i] $$，对于部分块，
我们必须依次在 $$ ori $$ 上进行单点查询，也就是去查询 $$ ori $$ 中 $$ p[i] $$ 处的值。

上面的操作时间复杂度均为 $$ O({N \over BLOCK\_SIZE} + BLOCK\_SIZE) $$ 的。

最后我们还要考虑计算 $$ perm\_cnt $$ 和 $$ ori\_cnt $$ 的时间复杂度，这一部分，
我们预处理的时候枚举块数量以及下标即可，因此时间复杂度为 $$ O({N^2 \over BLOCK\_SIZE}) $$。

由于空间的限制，我们的 $$ BLOCK\_SIZE $$ 不能太小，
既不能选择理论最优值 $$ BLOCK\_SIZE = \sqrt{N} $$，
此时我们可以选择 $$ BLOCK\_SIZE = 600 $$，能够保证通过。

代码：[queries.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/queries.cpp)

# [Doubled Sequence]()

不会。

# [Icarus](https://csacademy.com/ieeextreme-practice/task/icarus)

我们记 $$ l_c $$ 表示 $$ S $$ 中 `L` 的出现次数，$$ r_c $$ 表示 $$ S $$ 中 `R` 的出现次数，
$$ u_c $$ 表示 $$ S $$ 中 `U` 的出现次数。

我们总可以通过构建一条链使其满足题目条件。这里我们以 $$ l_c \le u_c $$ 为例，
其余的情况可以进行类似的讨论。

当 $$ l_c \le u_c $$ 时，我们可以构建一条有 $$ u_c + 2 $$ 个结点的链，
链上的结点均以左结点的方式进行连接。这样构造后，输入中的 `R` 将可以被忽略，
同时我们可以在这个链上找到一个结点，
其满足从当前结点触发的第一轮操作中所有的 `L` 和 `U` 都是合法移动且最深的结点不会被访问到。

如何找到这样的点呢？我们可以先假设我们在一个两端无限长的链上的某个结点，
初始化 $$ depth $$ 为 $$ 0 $$，我们遍历输入中的移动操作，对于 `L` 我们将 $$ depth $$ 减一，
对于 `U` 我们将 $$ depth $$ 加一，记录这个过程中最小的 $$ depth $$，不妨记为 $$ high $$，
如果我们将链上的结点按照深度从低到高进行编号，那么我们从编号为 $$ -high + 1 $$ 的结点出发，
后续的所有操作一定不能访问到编号为 $$ u_c + 2 $$ 的结点。

这是因为我们从编号为 $$ -high + 1 $$ 的点出发的话，第一轮操作肯定都是合法的，
且不会到达编号为 $$ u_c + 2 $$ 的结点；
对于第二轮操作，我们的起点一定是编号 $$ -high + 1 $$ 的结点或者其祖先。

如果第二轮起点是编号为 $$ -high + 1 $$ 的结点，那么此后续操作便开始循环，不会到达 $$ u_c + 2 $$；

如果第二轮起点是编号为 $$ -high + 1 $$ 的结点的祖先，那么第二轮可能会有一些 `U` 操作失效，
实际上第二轮的起点与第一轮的起点的深度差的绝对值 (不妨记为 $$ x $$) 就是 `U` 操作最多失效的次数，
下面来解释为什么第二轮的 `U` 操作最多失效 $$ x $$ 次。如果有多于 $$ x $$ 次的 `U` 操作失效，
不妨记为 $$ x' $$，此时我们假设链变成无限长，
那么这 $$ x' $$ 次失效的 `U` 操作将会使到达的最低深度减少 $$ x' $$，
这意味着我们的起点深度较上一轮应该减少 $$ x' $$，这就出现了矛盾。

这个结论意味着，当 `U` 的失效次数达到最大值时，我们第二轮的终点也不过是回到了第一轮的起点，
且这个过程中到达的最大深度也不过是第一轮到达的最大深度；而其余情况下，
我们的终点深度一定小于第一轮的起点深度，到达的最大深度也一定小于第一轮到达的最大深度。
继续这样推下去，我们便可以发现之后每一轮都不会到达 $$ u_c + 2 $$。

这里还有一点需要注意的是，如果 $$ l_c = 0 $$，而 $$ u_c = 1 $$，
此时按照上面的方法会构造出 $$ 3 $$ 个结点的树，这一点不满足题目的限制条件。
因此我们需要对这一部分进行特判。这里我们只需要构造一个有 $$ 2 $$ 个结点的树，
$$ 2 $$ 为 $$ 1 $$ 的左结点，那么从 $$ 1 $$ 号结点出发永远不会到达 $$ 2 $$ 号结点。
当然我们可以在上面的思路中选择构造结点个数为 $$ 2 (l_c + u_c + r_c) $$ 的链，
这样也可以解决这一部分。

其余的情况可以进行类似的讨论。

代码：[icarus.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/icarus.cpp)

# [Power of three](https://csacademy.com/ieeextreme-practice/task/power-of-three)

设 $$ M $$ 表示输入 $$ N $$ 的位数，我们计算出 $$ x_{min} = (M - 1) \lfloor \log_3 10 \rfloor $$，
不难发现如果有解 $$ x $$，那么 $$ x \ge x_{min} $$ 时，且在理论上 $$ x - x_{min} \le 3 $$，
这是因为 $$ 3^3 = 27 $$，也就是乘以 $$ 3 $$ 个 $$ 3 $$ 之后，位数会增加 $$ 1 $$。

同时对于解 $$ x $$ 我们有 $$ 3^x \equiv N \pmod{P} $$，其中 $$ P $$ 是任意的数，
因此我们可以提前确定多个质数，从 $$ x = x_{min} $$ 开始，
然后检查是否对于这些质数都有 $$ 3^x \equiv N \pmod{P} $$，如果成立，
我们可以大概率认为这个 $$ x $$ 是答案。如果 $$ x $$ 在 $$ 3 $$ 次递增的过程中没有找到答案，
那么我们输出 $$ -1 $$ 即可。

代码：[power_of_three.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/power_of_three.cpp)

# [Halving](https://csacademy.com/ieeextreme-practice/task/halving)

我们先考虑什么情况下无解。首先我们先尝试将能确定的元素进行确定，因为这些元素对方案数没有贡献。
哪些元素能够被确定下来？一共有两种：

* 已经给定的元素。
* 对于第 $$ i $$ 对，其中一个元素为 $$ -1 $$，另一个元素不为 $$ B_i $$，
那么可以将为 $$ -1 $$ 的元素确定为 $$ B_i $$。

经过上述操作后，我们只需要检查确定的元素是否只出现了一次，
以及如果第 $$ i $$ 对中两个元素同时被确定其是否满足最小值或最大值为 $$ B_i $$ 来判断是否有解。

接下来我们考虑如何求解有解情况下的方案数。我们可以使用动态规划的方法，
原问题等价于求使用没有在 $$ B $$ 中出现的元素与在 $$ B $$ 中的元素进行配对 (不考虑顺序) 的方案数，
该方案数与答案相比只是少乘了 $$ 2^k $$，这是因为第一步填入确定元素的操作后，
我们需要确定的元素对只可能出现以下两种情况：

* 两个元素均为 $$ -1 $$。
* 一个元素为 $$ -1 $$，另一个元素为 $$ B_i $$。

对于第一种情况，我们可以确定其中一个为 $$ B_i $$，另一个为比 $$ B_i $$ 大或者小的元素，
由于两者位置不确定，所以此时等价于求解能与 $$ B_i $$ 配对的元素的个数再乘以 $$ 2 $$。

对于第二种情况，等价于求解能与 $$ B_i $$ 配对的元素的个数。

因此我们可以用 $$ dp[i][j][k] $$ 来表示用 $$ 1, 2, ..., i $$ 中不在 $$ B $$ 中的元素与在 $$ B $$ 中的元素配对，
且在 $$ 1, 2, ..., i $$ 中有 $$ j $$ 个在 $$ B $$ 中的元素未配对有 $$ k $$ 个不在 $$ B $$ 中的元素未配对
(这个定义有点拗口)。

接下来我们考虑如何进行转移，首先我们需要确定 $$ i $$ 的三种情况：

* $$ i $$ 是一个已经拥有确定匹配的元素。
* $$ i $$ 未拥有确定匹配且在 $$ B $$ 中。
* $$ i $$ 未拥有确定匹配且不在 $$ B $$ 中。

这里的拥有确定匹配是指：如果一对元素的两个元素均在第一步中被确定，那么这两个元素就拥有确定匹配。

对于第一种情况，显然 $$ dp[i][j][k] = dp[i - 1][j][k] $$。

对于第二种情况，我们需要根据是需要找比 $$ i $$ 大的元素还是比 $$ i $$ 小的元素进行转移：

$$
dp[i][j][k] = \begin{cases}
dp[i - 1][j][k]，\text{找比} i \text{大的元素} \\
dp[i - 1][j[k + 1] \times (k + 1)，\text{找比} i \text{小的元素}
\end{cases}
$$

第二种情况的第一个转移方程是因为我们不能在 $$ 1, 2, 3, ..., i - 1 $$ 中找到比 $$ i $$ 大的元素，
所以此时只能暂时不对 $$ i $$ 进行配对。
第二个转移方程则是所有在 $$ 1, 2, 3, ..., i - 1 $$ 中不在 $$ B $$ 中的元素未配对的元素均可以与 $$ i $$ 配对，
且配对后这一部分未配对的元素个数会减少 $$ 1 $$。

对于第三种情况，由于在第二种情况中我们保证了 $$ B $$ 中未参与配对的元素一定是比 $$ i $$ 小的，
所以此时我们有以下的转移方程：

$$
dp[i][j][k] = dp[i - 1][j][k - 1] + dp[i - 1][j + 1][k] \times (j + 1)
$$

第三种情况中的 $$ dp[i - 1][j][k - 1] $$ 表示我们可以暂时不对 $$ i $$ 进行配对，
而 $$ dp[i - 1][j + 1][k] \times (j + 1) $$ 代表我们可以选择任意一个在 $$ B $$ 中未参与配对的元素与 $$ i $$ 配对，
配对之后这一部分未配对的元素个数会减少 $$ 1 $$。

通过观察三部分的方程不难发现我们可以使用滚动数组将空间复杂度从 $$ O(N^3) $$ 优化到 $$ O(N^2) $$。

最后的答案即为 $$ dp[2N][0][0] \times 2^k $$，其中 $$ k $$ 表示初始有多少对元素对中的元素同时为 $$ -1 $$。

代码：[halving.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/halving.cpp)

# [King's Order](https://csacademy.com/ieeextreme-practice/task/kings-order)

直接使用拓扑排序即可，只是在拓扑排序中需要将普通队列替换成优先队列。
优先队列的比较器设置为题目要求即可。

代码：[kings_order.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/kings_order.cpp)

# [Balls](https://csacademy.com/ieeextreme-practice/task/balls)

本题是 `Codeforces` 在十三年前某场比赛的原题，
原题的链接：[Codeforces 93 E. Lostborn](https://codeforces.com/problemset/problem/93/E)。

本题的思路来源于题解：[Codeforces Beta Round 76 - задача Е div 1 глазами ее автора.](https://codeforces.com/blog/entry/2216)

首先我们需要反向考虑问题，具体的，
我们定义 $$ f_{E_1, E_2, \dots, E_K}(N) $$ 表示给定 $$ N $$ 和 $$ K $$ 个球的情况下有多少个点没有被命中，
那么命中的点即为 $$ N - f_{E_1, E_2, \dots, E_K}(N) $$。

首先我们考虑 $$ f_{E_1, E_2, \dots, E_K}(N) $$ 是否存在某种递推关系。
事实上，由容斥原理我们有以下的递推关系：

$$
f_{E_1, E_2, \dots, E_K}(N) = f_{E_2, E_3, \dots, E_K}(N) - f_{E_2, E_3, \dots, E_K}(\lfloor \frac{N}{E_1} \rfloor)
$$

为了理解上面的递推公式，
我们可以考虑对于给定 $$ N $$ 和 $$ E_2, E_3, \dots, E_K $$ 的情况下增加 $$ E_1 $$ 会有多少个新的点被覆盖。

我们知道如果增加 $$ E_1 $$，
那么 $$ 1 \times E_1, 2 \times E_1, \dots, \lfloor \frac{N}{E_1} \rfloor \times E_1 $$ 这些点会被覆盖，
那么这些点里面哪些点是新增的呢？由于 $$ E_1 $$ 与其他的 $$ E_i $$ 互质，
在 $$ 1, 2, \dots, \lfloor \frac{N}{E_1} \rfloor $$
这些数中如果某个数与所有的 $$ E_i $$ 互质，那么这个数一定是新增的点，
这一部分刚好对应 $$ f_{E_2, E_3, \dots, E_K}(\lfloor \frac{N}{E_1} \rfloor) $$。
这也是为什么我们会有以上的递推式。

我们对上面的递推式起个别名：

$$
dp[i][j] = f_{E_i, E_{i+1}, \dots, E_K}(j)
$$

那么就有如下的动态转移方程：

$$
dp[i][j] = dp[i + 1][j] - dp[i + 1][\lfloor \frac{j}{E_i} \rfloor]
$$

接下来我们来考虑上面式子的复杂度。你可能会说这不是显然的 $$ O(NK) $$ 吗？是的如果使用递推来做的话，
复杂度确实是 $$ O(NK) $$。但是实际上并不是所有的状态都是有用的状态，
因此如果我们使用递归来实现，我们的时间复杂度实际上为有用的状态个数。

如何估计有用状态的个数？首先上面式子 $$ i $$ 的取值数一定是 $$ K $$ 种，
接下来我们考虑 $$ j $$ 取值种数。
不难发现对于任何一个可能的 $$ j $$ 其一定可以写成 $$ \lfloor \frac{N}{a} \rfloor $$ 的形式，
其中 $$ a $$ 是某个正整数。也就是说要估计 $$ j $$ 的种类数，
我们只需要考虑 $$ \lfloor \frac{N}{a} \rfloor $$ 的种类数，
而 $$ \lfloor \frac{N}{a} \rfloor $$ 的种类数不会超过 $$ min(a 的种类数, \lfloor \frac{N}{2} \rfloor + 2) $$，
这里第二部分是因为当 $$ a = \lfloor \frac{N}{2} \rfloor, N \gt 3 $$ 时，$$ \lfloor \frac{N}{a} \rfloor $$ 为 $$ 2 $$，
就算对于 $$ a \in [1, \lfloor \frac{N}{2} \rfloor] $$，$$ \lfloor \frac{N}{a} \rfloor $$ 均获得了不同的值，
此时继续增加 $$ a $$ 也只能获得 $$ 1, 0 $$ 两种结果。

考虑到显然有下式成立：

$$
min(a, \lfloor \frac{N}{a} \rfloor) \le \sqrt{N}
$$

故 $$ j $$ 的种类数不会超过 $$ 2 \sqrt{N} $$，因此有用状态的个数不会超过 $$ 2K \sqrt{N} $$。
这对应的时间复杂度为 $$ O(K \sqrt{N}) $$。但实际上如果我们对 $$ E $$ 按照从大到小的顺序排序，
那么 $$ N $$ 会下降的非常快，这样我们实际上的复杂度会远远小于 $$ O(K \sqrt{N}) $$。

我们当然不能对所有可能的状态都进行记忆化，这样会导致我们不得不使用字典，
这往往会让我们的时间复杂度变为 $$ O(K \sqrt{N} log(K \sqrt{N})) $$，且常数非常大。
正确的做法是我们只在 $$ N $$ 较小时进行记忆化的操作。
这是因为对于递归而言，我们越小的部分被重复计算的次数相较较大的部分会更多。

代码：[balls.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/balls.cpp)

# [Corporation](https://csacademy.com/ieeextreme-practice/task/corporation)

首先感谢 [yanire](https://codeforces.com/profile/yanire) 提供的思路。

我们可以使用分块来解决这个问题，具体的我们将整个工资序列分成大小为 $$ \sqrt{N} $$ 的块，
最后一块的大小可能小于 $$ \sqrt{N} $$。每块维护以下信息：

* `sum_salary`：块内所有工资的和。
* `sum_happiness`：块内幸福值的和。
* `lazy_salary`：块内每个员工的工资增量。
* `lazy_happiness`：块内每个员工的幸福值增量。
* `all_same`：块内每个员工的工资值是否相同。

对于增加操作：

* 如果覆盖整个块，那么可以在 $$ O(1) $$ 的时间内完成更新，最多重复 $$ O(\sqrt{N}) $$ 次，
时间复杂度为 $$ O(\sqrt{N}) $$。
* 如果覆盖部分块，那么需要遍历当前被覆盖的块，然后重新计算块内的信息，
最多会出现两次 (两个边界块) 这种情况，时间复杂度为 $$ O(\sqrt{N}) $$。

对于设置操作：

* 如果覆盖整个块且整个块的每个员工工资值相同，那么可以在 $$ O(1) $$ 的时间内完成更新，
最多出现 $$ O(\sqrt{N}) $$ 次，时间复杂度为 $$ O(\sqrt{N}) $$。
* 如果覆盖整个块但整个块的每个员工工资值不同，那么需要遍历整个块，然后重新计算块内的信息，
时间复杂度为 $$ O(\sqrt{N}) $$。对于这种情况下，
我们考虑一开始最多有 $$ O(\sqrt{N}) $$ 个 `all_same` 为 `false` 的块，
我们每次通过设置操作遍历一整个块的时候，`all_same` 为 `false` 的块的数量会减少一，
而每次增加操作最多可能会让 `all_same` 为 `false` 的块数量增加二，
这意味着 $$ Q $$ 次操作中最多会执行 $$ O(\sqrt{N} + Q) $$ 次遍历整个块的操作，
均摊下来每次操作只会执行 $$ O(1) $$ 次。
* 如果覆盖部分块，那么需要遍历当前被覆盖的块，然后重新计算块内的信息，
最多会出现两次 (两个边界块) 这种情况，时间复杂度为 $$ O(\sqrt{N}) $$。

对于查询操作：

* 如果覆盖整个块，那么可以在 $$ O(1) $$ 的时间内完成查询，最多重复 $$ O(\sqrt{N}) $$ 次，
时间复杂度为 $$ O(\sqrt{N}) $$。
* 如果覆盖部分块，那么需要遍历当前被覆盖的块，
最多会出现两次 (两个边界块) 这种情况，时间复杂度为 $$ O(\sqrt{N}) $$。

综上我们可以发现上述方法的时间复杂度为 $$ O(\sqrt{N}N+\sqrt{N}Q) $$。

最后需要使用快速读写减少常数的影响。

代码：[corporation.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/corporation.cpp)

# [This is not an optimization problem](https://csacademy.com/ieeextreme-practice/task/this-is-not-an-optimization-problem)

首先感谢 [cancaneed](https://codeforces.com/profile/cancaneed) 提供的思路。

首先我们考虑计算大小为 $$ k $$ 时的结果。
此时我们依次考虑每个结点 $$ u, 1 \le u \le N $$ 的权重对答案的贡献。
对于结点 $$ u $$ 而言，
其贡献次数显然为从 $$ N $$ 个结点中选择 $$ k $$ 个结点的方案数减去没有选中 $$ u $$ 的方案数。

对于从 $$ N $$ 个结点中选择出 $$ k $$ 个结点的方案数，我们可以使用组合数的方法计算，
即为 $$ {N \choose k} $$。

接下来我们考虑什么情况下不会选择 $$ u $$。
如果选择了 $$ k $$ 个结点并通过增加一些其他结点构成一棵树且没有选择 $$ u $$ 那么所有选择的结点一定在 $$ u $$ 的同一个相邻分支中。
这是显然的，如果存在两个结点在 $$ u $$ 的不同分支中，
那么这两个结点之间一定有一条路径需要经过 $$ u $$ 结点。
那么这一部分的方案数即为 $$ \sum_{v \in adj(u)} {sz[v] \choose k} $$。
其中 $$ sz[v] $$ 表示删除 $$ u $$ 结点后，以 $$ v $$ 为根的子树的结点个数。

结合上面两部分那么大小为 $$ k $$ 的答案即为：

$$
\sum_u w[u]{N \choose k} - \sum_u \sum_{v \in adj(u)} w[u]{sz[v] \choose k}
$$

接下来我们考虑将所有的相同组合数进行合并，设最后的系数为 $$ b[i] $$，那么我们有：

$$
\sum_{i = 0}^{N} b[i] {i \choose k} = \sum_{i = 0}^{N} \frac{i!b[i]}{k!(i-k)!} = \frac{1}{k!} \sum_{i = 0}^{N} \frac{i!b[i]}{(i-k)!}
$$

首先我们考虑如何计算系数 $$ b[i] $$。我们只需要考虑每个结点 $$ u $$ 对其邻居的影响即可。具体的，
我们在 `DFS` 的过程中，设当前到达的结点为 $$ u $$，$$ u $$ 的父结点为 $$ par $$ 子结点为 $$ v $$，
那么 $$ u $$ 可以作为一整棵子树与 $$ par $$ 相连，
也就是我们在计算 $$ par $$ 贡献的时候可以在以 $$ u $$ 为根的子树中选择 $$ k $$ 个结点，
此时 $$ b[sz[u]] $$ 会减少 $$ w[par] $$ (系数为负)；同时，我们在计算 $$ v $$ 贡献的时候，
可以在除去以 $$ v $$ 为根的子树后的其他结点中选择 $$ k $$ 个结点，
此时对应 $$ b[n - sz[v]] $$ 减少 $$ w[v] $$。
最后不要忘记 $$ b[n] $$ 还需要增加 $$ w[u] $$ (对应 $$ \sum_u {N \choose k} $$)。

接下来我们考虑如何计算 $$ \frac{1}{k!} \sum_{i = 0}^{N} \frac{i!b[i]}{(i-k)!} $$，
这里我们考虑计算 $$ \sum_{i = 0}^{N} \frac{i!b[i]}{(i-k)!} $$，
最后的 $$ \frac{1}{k!} $$ 只需要在最后乘上乘法逆元即可。

我们构造两个 $$ N $$ 次多项式：

$$
\begin{aligned}
P_1(x) &= n!b[n] + (n - 1)!b[n-1]x + (n - 2)!b[n-2]x^2 + \dots + 0!b[0]x^n \\
P_2(x) &= \frac{1}{0!} + \frac{1}{1!}x + \frac{1}{2!}x^2 + \dots + \frac{1}{n!}x^n
\end{aligned}
$$

我们考虑求 $$ P_1(x)P_2(X) $$ 的 $$ x^{n-k} $$ 的系数：

$$
Coef(x^{n-k}, P_1(x)P_2(x)) = \frac{n!b[n]}{(n-k)!} + \frac{(n-1)!b[n-1]}{(n-k-1)!} + \dots + \frac{(n-k)!b[n-k]}{(0)!} = \sum_{i = 0}^{n} \frac{i!b[i]}{(i-k)!}
$$

这意味着我们只需要计算 $$ P_1(x)P_2(x) $$ 的 $$ x^{n-k} $$ 的系数再乘上 $$ {1 \over k!} $$ 即为答案，
此过程通过任意模数的 `NTT` 算法即可完成。

代码：[this_is_not_an_optimization_problem.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/this_is_not_an_optimization_problem.cpp)

# [Digits swap](https://csacademy.com/ieeextreme-practice/task/digits-swap)

直接暴力搜索即可，搜索的时候注意只有当当前这一位与其最大可能性不同时才进行搜索。
理论时间复杂度为 $$ O(N^K) $$，但实际上跑得飞快。

给出几种常见贪心的反例：

* 每次选靠前最大的和当前位置交换：例如 $$ 12344\ 2 $$，如果按照这种方法会得到 $$ 44312 $$ ，
实际最优为 $$ 44321 $$。
* 每次选靠后最大的和当前位置交换：例如 $$ 21344\ 2 $$，如果按照这种方法会得到 $$ 44312 $$ ，
实际最优为 $$ 44321 $$。
* 每次选靠后最大的和当前位置交换，交换完某种数字后进行一次从大到小的排序：例如 $$ 45366\ 3 $$，
选择靠后最大交换两次后 $$ 66345 $$，对 $$ 45 $$ 进行排序得到 $$ 66354 $$，
再进行一次交换得到 $$ 66534 $$，实际最优为 $$ 66543 $$。

代码：[digits_swap.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/digits_swap.cpp)

# [Brick stacks](https://csacademy.com/ieeextreme-practice/task/brick-stacks)

我们先从小到大排序，然后依次处理每一个元素，对于当前元素，如果已经形成了 $$ pile $$ 堆，
我们只需要记录形成的堆的最下方的元素，不妨用 $$ pile[i] $$ 表示第 $$ i $$ 堆最下方的元素，
然后尝试将当前元素放到这些堆中，
实际上我们只需要检查当前元素是否可以放到最小的 $$ pile[i] $$ 所在的堆中，
这是因为：

* 如果当前的元素不能放入最小的 $$ pile[i] $$ 所在的堆中，那么它一定不能放入其他堆中；
* 如果设 $$ pile_{min} $$ 表示最小的 $$ pile_[i] $$，$$ pile_j $$ 表示其他任意一个非最小的 $$ pile[i] $$，
那么不会存在当前元素可以放入 $$ pile_{min} $$ 和 $$ pile_j $$，但在当前元素放入 $$ pile_{min} $$ 后，
下一个元素不能放入 $$ pile_j $$，且如果当前元素放入 $$ pile_j $$ 后，
下一个元素可以放入 $$ pile_{min} $$ 的情况。

对于上述的第二点，可以有如下的证明：

> 首先 $$ pile_{min} + x \gt pile_j $$，否则可以将 $$ pile_j $$ 的最后一块放到 $$ pile_{min} $$ 后面。
其次如果 $$ A_i + x \le A_{i+1} $$，那么下一个元素一定是能放在 $$ A_i $$ 所在堆的；
如果 $$ A_i + x \gt A_{i+1} $$，而我们将上面第二点写成不等式与该不等式进行联立：
>
> $$
\begin{cases}
pile_j > pile_{min} \\
pile_{min} + x \gt pile_j \\
A_i + x \gt A_{i+1} \\
pile_{min} + x \le A_i \\
pile_{min} + x \le A_{i+1} \\
pile_j + x \gt A_{i+1} \\
pile_j + x \le A_i \\
A_{i+1} \gt A_{i} \\
\end{cases}
> $$
>
> 上面的最后三个存在矛盾，因此不可能有解。

这就意味着，我们可以用一个小根堆维护形成的堆的最后一个元素，每次尝试将新的数放入到堆顶所形成的堆，
如果不能放置，则形成一个新的堆加入到堆中。

代码：[brick_stacks.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/brick_stacks.cpp)

# [Stones](https://csacademy.com/ieeextreme-practice/task/stones)

首先感谢 [cancaneed](https://codeforces.com/profile/cancaneed) 提供的思路。

我们记 $$ (R1, B1, R2, B2) $$ 为藏球方红球有 $$ R1 $$ 个，蓝球有 $$ B1 $$ 个；
猜球方红球有 $$ R2 $$ 个，蓝球有 $$ B2 $$ 个的状态时猜球方的最优策略下的最大获胜概率。

如果我们设 $$ p $$ 为藏红球的概率，$$ q $$ 为猜红球的概率，
同时我们设

$$
rr := 1 - (R2, B2, R1-1, B1) \\
rb := 1 - (R2, B2-1, R1, B1) \\
br := 1 - (R2-1, B2, R1, B1) \\
bb := 1 - (R2, B2, R1, B1 - 1) \\
$$

我们不难写出以下的转移方程：

$$
(R1, B1, R2, B2) = p \cdot q \cdot rr \cdot + p \cdot (1-q) \cdot rb + (1-p) \cdot q \cdot br + (1-p) \cdot (1-q) \cdot bb
$$

由于双方均追求最大化获胜结果，所以我们需要对上面的式子分别关于 $$ p, q $$ 求导并令其为 $$ 0 $$：

$$
\begin{cases}
\frac{\partial (R1, B1, R2, B2)}{\partial p} = q \cdot rr + (1-q) \cdot rb - q \cdot br - (1-q) \cdot bb = 0 \\
\frac{\partial (R1, B1, R2, B2)}{\partial q} = p \cdot rr - p \cdot rb + (1-p) \cdot br - (1-p) \cdot bb = 0
\end{cases}
$$

解上面的方程可以得到：

$$
\begin{cases}
p = \frac{bb - br}{rr - rb - br + br} \\
q = \frac{bb - rb}{rr - rb - br + bb} \\
\end{cases}
$$

最后带入 $$ (R1, B1, R2, B2) $$ 即可。

代码：[stones.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/stones.cpp)

# [Rectangles and arrays](https://csacademy.com/ieeextreme-practice/task/rectangles-and-arrays-ieeextreme-18)

我们首先考虑不进行修改的情况，那么我们可以使用单调栈来解决这个问题，具体的，我们需要计算出：

* $$ l1[i] $$：表示第 $$ i $$ 个元素左侧第一个比它小的元素的位置，如果不存在则设置为最小下标减一。
* $$ r1[i] $$：表示第 $$ i $$ 个元素右侧第一个比它小的元素的位置，如果不存在则设置为最大下标加一。

那么统计不进行修改的部分就是 $$ \max\limits_{1 \le i \le N}(r1[i] - l1[i] - 1)A_i $$，
这里公式表示以 $$ A_i $$ 为最小值所能形成的最大的正方形。

接下来我们考虑需要修改的情况，首先如果要修改某个元素，
我们直接将其修改为 $$ X $$ 一定会比修改成一个比 $$ X $$ 小的数更优。
这意味着对于 $$ A_i \ge X $$ 的元素，我们没有必要修改，
所有后面的修改将会针对满足 $$ A_i \lt X $$ 的元素。

接下来我们考虑如果将 $$ A_i $$ 修改成 $$ X $$，会对哪些部分产生影响。
显然对于 $$ l1[j] = i, j \gt i $$ 和 $$ r1[j] = i, j \lt i$$ 的位置 $$ j $$，
其 $$ l1[j], r1[j] $$ 可能会发生变化。
对于 $$ i $$ 位置，其 $$ l1[i], r1[i] $$ 也可能会发生变化。

因此，
我们只需要计算出修改 $$ A_i $$ 为 $$ X $$ 后的新的 $$ l1_i, r1_i $$ 即可按照之前的公式进行更新。

实际上对于非 $$ i $$ 位置的变化，显然 $$ l1, r1 $$ 会变成左右两侧第二个比它小的元素的位置。
为了避免混淆，我们增加以下定义：

* $$ l2_i $$：表示第 $$ i $$ 个元素左侧第二个比它小的元素的位置，如果不存在则设置为最小下标减一。
* $$ r2_i $$：表示第 $$ i $$ 个元素右侧第二个比它小的元素的位置，如果不存在则设置为最大下标加一。
* $$ l3_i $$: 表示将第 $$ i $$ 个元素修改为 $$ X $$ 后左侧第一个比它小的元素的位置，如果不存在则设置为最小下标减一。
* $$ r3_i $$: 表示将第 $$ i $$ 个元素修改为 $$ X $$ 后右侧第一个比它小的元素的位置，如果不存在则设置为最大下标加一。

而修改 $$ i $$ 位置对 $$ j $$ 造成影响实际上就是在修改 $$ j $$ 位置左侧或者右侧第一个比它小的元素。

因此这一部分的贡献就是 $$ \max\limits_{1 \le i \le N}\{(r2[i] - l1[i] - 1)A_i, (r1[i] - l2[i] - 1)A_i, (r3[i] - l3[i] - 1)X\} $$。

上面使用 $$ r2[i] - l1[i] - 1 $$ 和 $$ r1[i] - l2[i] - 1$$ 而不直接使用 $$ r2[i] - l2[i] - 1 $$ 是因为我们只能修改一个元素，
而不能同时将左右两个第一个比当前小的元素修改为 $$ X $$。

下面我们来介绍如何计算 $$ l2, r2, l3, r3 $$。

这里以计算 $$ l2 $$ 为例介绍如何计算 $$ l2, r2 $$。计算这一部分，我们需要用到三个栈，
第一个栈与在计算 $$ l1 $$ 时作用一样，而第二个栈用于保存从第一个栈中弹出的元素，由于栈是先进后出的，
第三个栈用于将从第一个栈中弹出的元素逆序放入到第二个栈中。
当然也可以将第二个栈换成队列，每次从队首取元素即可。这样操作后当从第二个栈弹出元素的时候，
也就找到了左侧第二个比栈顶元素小的元素。代码如下：

```c++
for (int i = n; i >= 1; i--) {
    while (!s2.empty() && a[s2.top()] > a[i]) {
        l2[s2.top()] = i;
        s2.pop();
    }
    while (!s1.empty() && a[s1.top()] >= a[i]) {
        s3.push(s1.top());
        s1.pop();
    }
    s1.push(i);
    while (!s3.empty()) {
        s2.push(s3.top());
        s3.pop();
    }
}
while (!s1.empty()) {
    l2[s1.top()] = 0;
    s1.pop();
}
while (!s2.empty()) {
    l2[s2.top()] = 0;
    s2.pop();
}
```

计算完成后不要忘记排除 $$ X < A_i $$ 的情况：

```c++
for (int i = 1; i <= n; i++) {
    if (x < a[i]) {
        l2[i] = l1[i];
    }
}
```

这里以计算 $$ l3 $$ 为例介绍如何计算 $$ l3, r3 $$。计算 $$ l3 $$ 与计算 $$ l1 $$ 类似。
只是我们需要比较的值从 $$ A_i $$ 变成了 $$ X $$，
但是这样会导致后续计算的时候有一些元素被提前弹出了栈，
实际上，我们可以先将弹出的元素放到一个队列里面，等到计算完 $$ l3[i] $$ 后再将这些元素放回栈中。
代码如下：

```c++
for (int i = 1; i <= n; i++) {
    queue<int> now_pop;
    while (!s.empty() && a[s.top()] >= x) {
        now_pop.push(s.top());
        s.pop();
    }
    l3[i] = (s.empty() ? 0 : s.top());
    while (!now_pop.empty()) {
        s.push(now_pop.front());
        now_pop.pop();
    }
    while (!s.empty() && a[s.top()] >= a[i]) { s.pop(); }
    s.push(i);
}
```

代码：[rectangles_and_arrays.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/rectangles_and_arrays.cpp)

# [Invertible Pairs](https://csacademy.com/ieeextreme-practice/task/invertible-pairs)

我们可以使用动态规划解决这一题，具体的我们用 $$ dp[i][0] $$ 表示前 $$ i $$ 个数，
以 $$ i $$ 结尾且第 $$ i $$ 个数不发生翻转的最大和，用 $$ dp[i][1] $$ 表示前 $$ i $$ 个数，
以 $$ i $$ 结尾且第 $$ i $$ 个数发生翻转的最大和。我们可以写出以下的转移方程：

$$
\begin{aligned}
dp[i][0] &= \begin{cases}
max(a[i], dp[i - 1][0] + a[i])，i \equiv 0 \pmod{2} \\
max(a[i], dp[i - 1][0] + a[i], dp[i - 1][1] + a[i])，i \equiv 1 \pmod{2} \\
\end{cases} \\
dp[i][1] &= \begin{cases}
max(-a[i], dp[i - 1][1] - a[i])，i \equiv 0 \pmod{2} \\
max(-a[i], dp[i - 1][0] - a[i], dp[i - 1][1] - a[i])，i \equiv 1 \pmod{2}
\end{cases}
\end{aligned}
$$

代码：[invertible_pairs.cpp](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/invertible_pairs.cpp)

# [Sierpinski](https://csacademy.com/ieeextreme18/task/sierpinski)

我们可以直接使用递归的方法来解决，我们首先需要确定当前行号，需要经过多少次构建才能构建出来，
不妨设为 $$ cnt $$，这样的操作是 $$ O(logx) $$ 的，因为每次构建行数为上一次的两倍加一。

同时我们也可以获取到 $$ cnt - 1 $$ 次构造后会有多少行，不妨设为 $$ last\_end $$，
那么第 $$ x $$ 行的 $$ [x - las\_end，las\_end + 1] $$ 列一定全是蓝色，
因为这一部分对应的是中间的大蓝三角形。因此我们有以下递归操作：

* 如果 $$ y \in [x - las\_end, las\_end + 1] $$，我们直接返回蓝色；
* 如果 $$ y \lt x - las\_end $$，不难发现当前颜色与 $$ (x - las\_end - 1, y) $$ 相同，此时递归调用；
* 如果 $$ y \gt las\_end + 1 $$，不难发现当前颜色与 $$ (x - las\_end - 1, y - las\_end - 1) $$ 相同，
此时递归调用。

代码：[sierpinski](https://github.com/Kaiser-Yang/OJProblems/blob/main/IEEExtreme/18/sierpinski.cpp)
