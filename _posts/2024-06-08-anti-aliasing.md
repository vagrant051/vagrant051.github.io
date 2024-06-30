---
layout: post
title: "抗锯齿技术"
categories: Computer-Graphics
tags: [图形学的数学原理]
---

SSAA,MSAA

## 超采样抗锯齿（SSAA）

使用比正常分辨率更高的分辨率（下采样）来渲染场景，带来了很大的性能开销。这项技术已经被废弃了。

## 多重采样抗锯齿（MSAA）

### 光栅器的原理

光栅器：图元装配之后，片段着色之前的一系列算法，作用是将图元的所有顶点作为输入，转化为一系列的片段

每个像素的中心包含有一个采样点(Sample Point)，它会被用来决定这个三角形是否遮盖了某个像素。图中红色的采样点被三角形所遮盖，在每一个遮住的像素处都会生成一个片段。

![My helpful screenshot](/assets/anti-aliasing/1.png)

![My helpful screenshot](/assets/anti-aliasing/2.png)

无论三角形遮盖了多少个子采样点，***（每个图元中）每个像素只运行一次片段着色器。***

片段着色器所使用的顶点数据会插值到每个像素的中心，所得到的结果颜色会被储存在每个被遮盖住的子采样点中。当颜色缓冲的子样本被图元的所有颜色填满时，所有的这些颜色将会在每个像素内部平均化。

![My helpful screenshot](/assets/anti-aliasing/3.png)

![My helpful screenshot](/assets/anti-aliasing/4.png)