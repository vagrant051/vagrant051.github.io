---
layout: post
title: "STL deque"
categories: C++
tags: [STL]
---

deque底层原理

## 底层原理

### vector与deque的区别

vector是单向开口的连续线性空间，deque则是一种双向开口的连续线性空间。所谓双向开口，意思是可以在头尾两端分别做元素的插人和删除操作，vector当然也可以在头尾两端进行操作（从技术观点），但是其头部操作效率奇差，无法被接受。

deque和vector的最大差异，一在于deque允许于常数时间内对起头端进行元素的插入或移除操作，二在于deque没有所谓容量（capacity）观念，因为它是动态地以分段连续空间组合而成，随时可以增加一段新的空间并链接起来。

换句话说，像vector那样“因旧空间不足而重新配置一块更大空间，然后复制元素，再释放旧空间”这样的事情在deque是不会发生的。也因此，deque没有必要提供所谓的空间保留（reserve）功能。

### deque的结构

deque采用一块map作为中控器，map是一小段连续空间，而map中的每个元素（node）都指向另外一段较大的连续空间，称为缓冲区。我们可以指定缓冲区的大小，512字节为默认大小。

map其实是一种T** 指针，因为它所指向的又是一个T*指针，代表存放T类型元素的缓冲区。
![My helpful screenshot](/assets/deque/1.png)

### deque的迭代器

制造“整体连续”假象的任务落在operator++和operator--身上。

迭代器必须知道自己是否处于缓冲区的边缘，一旦是，就需要跳跃到下一个或上一个缓冲区。

迭代器的基本数据如下：

{% highlight ruby %}
T* cur; //指向当前缓冲区的最后元素的下一个位置
T* first;  
T* last;
map_pointer node; //指向map中控器中的节点node
{% endhighlight %}

![My helpful screenshot](/assets/deque/2.png)

operator++操作符

![My helpful screenshot](/assets/deque/3.png)

实现随机存取

![My helpful screenshot](/assets/deque/4.png)

### deque的数据结构

deque需要维护的对象：

- map指针
- start,finish两个迭代器,其cur指针分别指向第一个缓冲区的第一个元素和最后一个缓冲区的最后一个元素
- map的大小,一旦map的节点不足，接需要重新配置更大的一块map

### 缓冲区的扩容

push_back（缓冲区已满的情况）:

![My helpful screenshot](/assets/deque/5.png)

该函数一开始便调用reserve_map_at_back();

push_front（缓冲区已满的情况）：

![My helpful screenshot](/assets/deque/6.png)

该函数一开始便调用reserve_map_at_front();

***值得一提的是调用这两个函数的边界条件不一样,push_back时的条件是finish.cur==finish.last-1,而push_back时的条件是start.cur==start.first，一个条件是尚且有一个空间，另一个则是没有任何空间***

同理，如果是pop操作，则需要判断边界条件，并析构掉缓冲区

### 中控器的扩容

然而reserve_map_at_back()和reserve_map_at_front()不一定导致map的重新分配，因为当map后面的节点要越界是，前面可能还有很多节点没有使用，这种情况下不会导致map的重新分配：

![My helpful screenshot](/assets/deque/7.png)

### erase操作

- 如果清除点之前的元素比较少，就移动清楚点之前的元素
- 如果清除点之后的元素比较少，就移动清楚点之后的元素
- 同时还要注意要释放多余的缓冲区