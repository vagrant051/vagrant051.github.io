---
layout: post
title: "C++复杂继承结构中的内存模型"
categories: C++
tags: [Inheritance]
---

Memory Model in Complex Inheritance Structures in C++

结论：

- 虚继承会把虚基类在内存当中的位置向后翻折，作用可以理解为向自己和子类屏蔽掉父类以上的继承关系

- 在已知对象的内存模型的情况下，如果想知道虚函数表指针所指向的虚函数表的内容，需要查看函数是否被重写，不用在意虚函数与普通函数的区别，只要是重写过的，都会如实反映在虚函数表中

- 虚基类表指针指向的虚基类表中的偏移量，指向的是父类及以上的虚函数表指针的地址

## 菱形继承（不带虚继承）

{% highlight ruby %}
                     A
                    / \
                   B   C
                    \ /
                     D
{% endhighlight %}

![My helpful screenshot](/assets/Inheritance/1.png)

{% highlight ruby %}
class A
{
public:
	int intA;
	virtual void functionA() { std::cout << "A::functionA" << std::endl; }
};

class B : public A
{
public:
	int intB;
	virtual void functionA() { std::cout << "B::functionA" << std::endl; }
	virtual void functionB() { std::cout << "B::functionB" << std::endl; }
};

class C : public A
{
public:
	int intC;
	virtual void functionC() { std::cout << "C::functionC" << std::endl; }
};

class D : public B, public C
{
public:
	int intD;
	virtual void functionD() { std::cout << "D::functionD" << std::endl; }
};
{% endhighlight %}

## 菱形继承（带虚继承）

{% highlight ruby %}
                     A
        (virtual)   / \ (virtual)
                   B   C
                    \ /
                     D
{% endhighlight %}

![My helpful screenshot](/assets/Inheritance/2.png)

![My helpful screenshot](/assets/Inheritance/3.png)

## 自定义继承结构

{% highlight ruby %}
                              A
                    (virtual)/ \
                            B   C
                            |    \ (virtual)
                            |     E
                            |    /
                            |   /
                            |  /
                             D
{% endhighlight %}

![My helpful screenshot](/assets/Inheritance/5.png)

![My helpful screenshot](/assets/Inheritance/6.png)

{% highlight ruby %}
class A
{
public:
	int intA;
	virtual void functionA() { std::cout << "A::functionA" << std::endl; }
};

class B : virtual public A
{
public:
	int intB;
	virtual void functionA() { std::cout << "B::functionA" << std::endl; }
	virtual void functionB() { std::cout << "B::functionB" << std::endl; }
};

class C : public A
{
public:
	int intC;
	virtual void functionC() { std::cout << "C::functionC" << std::endl; }
};

class E : virtual public C
{
public:
	int intE;
	virtual void functionB() { std::cout << "E::functionB" << std::endl; }
	virtual void functionA() { std::cout << "E::functionA" << std::endl; }
	virtual void functionE() { std::cout << "E::functionE" << std::endl; }
};

class D : public E, public B
{
public:
	int intD;
	virtual void functionD() { std::cout << "D::functionD" << std::endl; }
};
{% endhighlight %}