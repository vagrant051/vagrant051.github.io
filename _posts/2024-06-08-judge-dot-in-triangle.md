---
layout: post
title: "判断点是否在三角形内"
categories: Computer-Graphics
tags: [图形学的数学原理]
---

重心法，同向法

## 重心法

AP = u*AC + v*AB

![My helpful screenshot](/assets/dot-in-triangle/1.png)

如果点在三角形内部：

- u > 0
- v > 0
- u + v < 1

## 同向法

假设点P位于三角形内，会有这样一个规律，当我们沿着ABCA的方向在三条边上行走时，会发现点P始终位于边AB，BC和CA的右侧。

![My helpful screenshot](/assets/dot-in-triangle/2.png)

连接PA，将PA和AB做叉积，再将CA和AB做叉积，如果两个叉积的结果方向一致，那么两个点在同一侧。