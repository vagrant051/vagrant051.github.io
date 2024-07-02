---
layout: post
title: "适配器模式"
categories: C++
tags: [design-pattern]
---

## 适配器模式

使得原本由于接口不兼容而不能一起工作的类能够一起工作。适配器模式通过将一个类的接口转换成客户端希望的另一个接口，从而使原本接口不匹配的类可以合作。

## 利用多继承实现

{% highlight ruby %}
#include <iostream>

// 旧接口
class OldInterface {
public:
    virtual void oldRequest() {
        std::cout << "Old Interface Request" << std::endl;
    }
};

// 新接口
class NewInterface {
public:
    virtual void newRequest() = 0;
};

// 适配器类，继承旧接口和新接口
class Adapter : public NewInterface, private OldInterface {
public:
    void newRequest() override {
        oldRequest();
    }
};

int main() {
    NewInterface* adapter = new Adapter();
    adapter->newRequest();  // 输出：Old Interface Request
    delete adapter;
    return 0;
}

{% endhighlight %}

## 利用适配器类中包含一个旧接口的对象实现

{% highlight ruby %}
#include <iostream>

// 旧接口
class OldInterface {
public:
    void oldRequest() {
        std::cout << "Old Interface Request" << std::endl;
    }
};

// 新接口
class NewInterface {
public:
    virtual void newRequest() = 0;
};

// 适配器类，通过组合旧接口的对象来实现适配
class Adapter : public NewInterface {
private:
    OldInterface* oldInterface;

public:
    Adapter(OldInterface* oldInterface) : oldInterface(oldInterface) {}

    void newRequest() override {
        oldInterface->oldRequest();
    }
};

int main() {
    OldInterface* oldInterface = new OldInterface();
    NewInterface* adapter = new Adapter(oldInterface);
    adapter->newRequest();  // 输出：Old Interface Request
    delete adapter;
    delete oldInterface;
    return 0;
}

{% endhighlight %}