---
layout: post
title: "C++类型转换"
categories: C++
tags: [cast]
---

static_cast,reinterpret_cast,const_cast,dynamic_cast

## static_cast

### 基本数据类型和枚举类型之间的转换

可以将一种基础数据类型转换为另一种基础数据类型。例如，将 double 转换为 int，或将 float 转换为 double 等。

{% highlight ruby %}
enum Color { RED, GREEN, BLUE };

int main() {
    int i = 42;
    float f = static_cast<float>(i); // int 转换为 float

    Color c = static_cast<Color>(i); // int 转换为枚举类型
    int i2 = static_cast<int>(c); // 枚举类型转换为 int

    return 0;
}
{% endhighlight %}

### 指向派生类的指针或引用转换为指向基类的指针或引用

{% highlight ruby %}
class Base {};
class Derived : public Base {};
Derived derivedObj;
Base* basePtr = static_cast<Base*>(&derivedObj);
{% endhighlight %}

### 指向基类的指针或引用转换为指向派生类的指针或引用

但这是不安全的，因为在转换过程中没有运行时检查。如果确实需要运行时检查，应使用 dynamic_cast。

{% highlight ruby %}
Base* basePtr = new Base();
Derived* derivedPtr = static_cast<Derived*>(basePtr); // 不安全！
{% endhighlight %}

## reinterpret_cast

### 指针类型之间的转换

reinterpret_cast 可以在不同的指针类型之间进行转换。这种转换只是重新解释指针的位模式，而不进行类型检查。

{% highlight ruby %}
int x = 42;
    int* intPtr = &x;

    // 将 int* 转换为 char*
    char* charPtr = reinterpret_cast<char*>(intPtr);

    // 输出指针地址和值
    std::cout << "intPtr: " << intPtr << ", value: " << *intPtr << std::endl;
    std::cout << "charPtr: " << static_cast<void*>(charPtr) << ", value: " << *charPtr << std::endl;

    return 0;
{% endhighlight %}

### 指针和整数进行转换

{% highlight ruby %}
int i_val = 100;
	int* i_ptr = &i_val;
	int reinterpret_val = reinterpret_cast<int>(i_ptr); // 将整数指针转换为整数
	cout << reinterpret_val << endl;
{% endhighlight %}  

## const_cast

{% highlight ruby %}
void fun(int* a)
{
    cout << *a << endl;
}

class myClass
{
public:
    int a = 10;
};

int main()
{
    const myClass* c = new myClass();
    cout << c->a << endl;
    //c->a = 100; // failed,const常量不能修改
    myClass* cc = const_cast<myClass*>(c);
    cc->a = 100; // success
    cout<< c->a <<" "<<cc->a<< endl;
    return 0;
}
{% endhighlight %}

## dynamic_cast

在没有多态类类型的转换中，编译器会提示报错。
dynamic_cast主要是用在进行下行转换（父类转换到子类）过程中的安全判定，上行转换（也就是子类转换为父类一定是安全的，例如下面代码中的情况3）；如果出现一些不安全的转换，则返回值就是nullptr，例如下面代码中的情况2；下行转换什么情况下是安全的呢？例如，当父类指针指向子类对象时，然后将这个父类指针利用dynamic_cast转换为子类，这种情况是安全的，因为这个指针指向的对象就是子类，例如下面代码中的情况2。

{% highlight ruby %}
#include<iostream>
using namespace std;

class Base  // 父类
{
public:
	virtual void f()
	{
		cout << "this is base class !" << endl;
	}
};

class Child :public Base { // 子类
public:
	void f()
	{
		cout << "this is child class !" << endl;
	}

};


int main()
{
	//情况1
	Base* b_ptr0 = new Base(); // 指向父类的父类指针
	Child* c_ptr0 = dynamic_cast<Child*>(b_ptr0);// 下行转换，此时由于父类指针实际上是指向父类的，
												//这个转换不安全，所以c_ptr0的值为nullptr
	if (!c_ptr0)
	{
		cout << "can not cast the type !" << endl; // 代码会走到这里
	}
	else
	{
		c_ptr0->f();
	}
	//情况2
	Base* b_ptr = new Child();// 指向子类的父类指针
	Child* c_ptr = dynamic_cast<Child*>(b_ptr); // 下行转换，此时由于父类指针实际上是指向子类的
												//所以这个转换是可以的
	if (!c_ptr)
	{
		cout << "can not cast the type !" << endl;
	}
	else
	{
		c_ptr->f();// 代码会走到这里
	}
	//情况3
	Child* c_ptr1 = new Child();// 指向子类的子类指针
	Base* b_ptr1 = dynamic_cast<Base*>(c_ptr1); // 将子类转换为父类是安全的，上行转换一定是允许的
	if (!b_ptr1)
	{
		cout << "can not cast the type !" << endl;
	}
	else
	{
		b_ptr1->f();// 代码会走到这里
	}
	/*
	* 输出是
	* can not cast the type !
	* this is child class !
	* this is child class !
	* 
	*/

}

{% endhighlight %}
