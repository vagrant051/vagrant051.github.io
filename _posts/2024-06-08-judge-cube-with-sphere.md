---
layout: post
title: "判断点球和正方体是否相交"
categories: Computer-Graphics
tags: [图形学的数学原理]
---

先判断面相交，再判断点相交，最后判断边相交

***球心坐标为center[3]，半径为r，立方体左下角坐标为cube[3]，边长为a，dis[3]表示球心到立方体中心的三轴距离的绝对值。***

## 去除显然不满足区域 

首先可以筛去很多区域，可以想到球心位于中心在立方体中心，边长为（a+2r）的立方体之外时，球不可能与立方体相交。

## 球与立方体面接触

![My helpful screenshot](/assets/cube-with-sphere/1.png)

{% highlight ruby %}
int cnt = 0;
for (int i = 0; i < 3;i++)if (dis[i] < a / 2)cnt++;
if (cnt >= 2)return true;
{% endhighlight %}

## 球与立方体顶点接触

![My helpful screenshot](/assets/cube-with-sphere/2.png)

{% highlight ruby %}
double xd = dis[0] - a / 2;
double yd = dis[1] - a / 2;
double zd = dis[2] - a / 2;
return xd * xd + yd * yd + zd * zd < r * r;
{% endhighlight %}

## 球与立方体边接触

![My helpful screenshot](/assets/cube-with-sphere/3.png)

{% highlight ruby %}
double xd = max(dis[0] - a / 2, 0);
double yd = max(dis[1] - a / 2, 0);
double zd = max(dis[2] - a / 2, 0);
return xd * xd + yd * yd + zd * zd < r * r;
{% endhighlight %}