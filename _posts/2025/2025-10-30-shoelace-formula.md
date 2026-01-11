---
layout: post
title: 鞋带公式
date: 2025-10-30 16:51:37+0800
last_updated: 2025-10-30 16:51:37+0800
description: 本文介绍鞋带公式的原理及其实现方式。
tags:
  - 中文文章
  - Shoelace Formula
categories: Algorithm
featured: false
giscus_comments: true
toc:
  sidebar: left
related_posts: true
pretty_table: true
---

简单多边形：如果一个多边形的任意两条边都不相交，则称该多边形为简单多边形。

鞋带公式是用来求一个简单多边形面积的公式。

鞋带公式要求按照顺时针或逆时针的顺序给出多边形的顶点坐标。

这里我们定义 $$ x_i, y_i $$ 分别表示一个简单 $$ n $$ 边形的第 $$ i $$ 个顶点的横坐标和纵坐标，
这里选择下标从 $$ 0 $$ 开始编号。特别地，我们定义 $$ x_n = x_0 $$，$$ y_n = y_0 $$。

鞋带公式的表达式为：

$$
\text{Area} = \frac{1}{2} \left| \sum_{i=0}^{n-1} (x_i y_{i+1} - x_{i+1} y_i) \right|
$$

## 原理

鞋带公式实际是通过计算多个梯形的代数面积来求解多边形的面积。
具体地，我们可以将多边形划分为多个梯形，
每个梯形的顶点分别是多边形的两个相邻顶点和 x 轴上的两个投影点。
通过计算每个梯形的面积并累加起来，我们可以得到整个多边形的面积。

而上述的过程可以写成如下的方程: 

$$
\text{Area} = \frac{1}{2} \left| \sum_{i=0}^{n-1} (y_i + y_{i+1}) (x_{i+1} - x_i) \right|
$$

整理后就可以得到最开始给出的鞋带公式。

## 实现

```cpp
template <typename T, typename R = T>
R polygon_area(const std::vector<std::array<T, 2>> &points, R _ = R()) {
    R area = 0;
    for (int i = 0; i < points.size(); i++) {
        int j = (i + 1) % points.size();
        area += (R)points[i][0] * points[j][1];
        area -= (R)points[i][1] * points[j][0];
    }
    return std::abs(area) / 2;
}
```
