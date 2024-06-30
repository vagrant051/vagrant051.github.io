---
layout: post
title: "STL vector"
categories: C++
tags: [STL]
---

vector接口与底层原理

## 接口

### vector的构造和析构

![My helpful screenshot](/assets/vector/3.png)

### vector的非更易型操作

![My helpful screenshot](/assets/vector/4.png)

### vector的赋值操作

![My helpful screenshot](/assets/vector/5.png)

### vector的元素访问

![My helpful screenshot](/assets/vector/6.png)

### vector的迭代器相关函数

![My helpful screenshot](/assets/vector/7.png)

### vector的插入与删除

![My helpful screenshot](/assets/vector/8.png)

## 底层原理

### vector是序列式容器

![My helpful screenshot](/assets/vector/2.png)

其中的元素都可序（ordered），但未必有序（sorted）。C++语言本身提供了一个序列式容器array，STL 另外再提供vector，1ist，dequestackqueue，prority-queue 等等序列式容器。其中stack和 queue 由于只是将deque改头换面而成，技术上被归类为一种配接器（adapter）。

### array和vector的区别

array是静态空间，一旦配置了就不能改变；要换个大（或小）一点的房子，可以，一切项细得由客户端自己来：首先配置一块新空间，然后将元素从旧址一一搬往新址，再把原来的空间释还给系统。

vector是动态空间，随着元素的加人，它的内部机制会自行扩充空间以容纳新元素。因此，vector的运用对于内存的合理利用与运用的灵活性有很大的帮助，我们再也不必因为害怕空间不足而一开始就要求一个大块头array了，我们可以安心使用vector，吃多少用多少。vector 的实现技术，关键在于其对大小的控制以及重新配置时的数据移动效率。
	  
一且vector有空间满载，如果客户端每新增一个元素，vector内部只是扩充一个元素的空间，实为不智，因为所谓扩充空间（不论多大），一如稍早所说，是“配置新空间/数据移动/释还旧空间”的大工程，时间成本很高，应该加入某种未雨绸缓的考虑。

![My helpful screenshot](/assets/vector/1.png)

vector所采用的数据结构非常简单：线性连续空间。它以两个送代器start和finish分别指向配置得来的连续空间中目前已被使用的范围，并以送代器endofstorage指向整块连续空间（含备用空间）的尾端

{% highlight ruby %}
template <class T, class Alloc = alloc>
class vector {
protected:
    iterator start;              //表示前使用空间的头
    iterator finish;             //表示前使用空间的尾
    iterator end_of_storage;     //表示目前可用空间的尾
}
{% endhighlight %}

### vector的扩容

为什么选择1.5倍或者2倍方式扩容，而不是3倍、4倍？

扩容原理为：申请新空间，拷贝元素，释放旧空间，理想的分配方案是在第N次扩容时如果能复用之前N-1次释放的空间就太好了，如果按照2倍方式扩容，第i次扩容空间大小如下：

每次扩容时，前面释放的空间都不能使用。比如：第4次扩容时，前2次空间已经释放，第3次空间还没有释放(开辟新空间、拷贝元素、释放旧空间)，即前面释放的空间只有1 + 2 = 3，假设第3次空间已经释放才只有1+2+4=7，而第四次需要8个空间，因此无法使用之前已释放的空间，但是按照小于2倍方式扩容，多次扩容之后就可以复用之前释放的空间了。如果倍数超过2倍(包含2倍)方式扩容会存在：

- 空间浪费可能会比较高，比如：扩容后申请了64个空间，但只存了33个元素，有接近一半的空间没有使用。

- 无法使用到前面已释放的内存。

### insert操作

![My helpful screenshot](/assets/vector/9.png)

![My helpful screenshot](/assets/vector/10.png)

![My helpful screenshot](/assets/vector/11.png)