---
layout: post
title: "单例模式"
categories: C++
tags: [123]
---

## 单例模式的分类

- 懒汉模式
- 饿汉模式

### 饿汉模式

在全局区创建单例对象，程序编译时这个单例就已经存在

饿汉模式是线程安全的

### 懒汉模式

在程序第一次需要使用实例的时候才创建这个单例

懒汉模式并非现场安全

看如下一段代码：

{% highlight ruby %}
static std::shared_ptr<ShoppingManager> GetInstance()
{
    if (s_Instance == nullptr)
    {
        std::unique_lock<std::mutex> lock(s_ShoppingMutex);
        if (s_Instance == nullptr)
        {
            s_Instance = std::shared_ptr<ShoppingManager>(new ShoppingManager());
        }
    }
    return s_Instance;
}
{% endhighlight %}

这段代码中，多个线程可能同时检测到s_Instance == nullptr条件为真，从而创建多个实例

同时这段代码使用了***双检锁***的技术，避免多个线程都要获取锁而带来的性能开销

### 懒汉模式代码实现

#### 用智能指针+双检锁优化

***有一个悬而未决的问题：下面这段代码不能将析构函数设为私有，否则会报错***

{% highlight ruby %}
class ShoppingManager
{
public:
	static std::shared_ptr<ShoppingManager> GetInstance()
	{
		if (s_Instance == nullptr)
		{
			std::unique_lock<std::mutex> lock(s_ShoppingMutex);
			if (s_Instance == nullptr)
			{
				s_Instance = std::shared_ptr<ShoppingManager>(new ShoppingManager());
			}
		}
		return s_Instance;
	}
	void PushItem(const std::string& item_name, int num)
	{
		std::unique_lock<std::mutex> lock(s_VectorMutex);
		m_ShoppingCart.push_back(std::make_pair(item_name, num));
	}
	void PrintInfo()
	{
		for (const auto& elem : m_ShoppingCart)
		{
			std::cout << elem.first << " " << elem.second << std::endl;
		}
	}
private:
	ShoppingManager() 
	{ 
		m_ShoppingCart.reserve(100);
	}
	ShoppingManager& operator=(const ShoppingManager& other) = delete;
	ShoppingManager(const ShoppingManager& other) = delete;
private:
	static std::shared_ptr<ShoppingManager> s_Instance;
	std::vector<std::pair<std::string, int>> m_ShoppingCart;
	static std::mutex s_ShoppingMutex;
	static std::mutex s_VectorMutex;
};

std::shared_ptr<ShoppingManager> ShoppingManager::s_Instance = nullptr;
std::mutex ShoppingManager::s_ShoppingMutex;
std::mutex ShoppingManager::s_VectorMutex;

int main() 
{
	std::string item_name; 
	int num;
	while (std::cin >> item_name >> num)
	{
		ShoppingManager::GetInstance()->PushItem(item_name, num);
	}
	ShoppingManager::GetInstance()->PrintInfo();
	return 0;
}
{% endhighlight %}

#### 用智能指针+static局部变量优化

static局部变量保证了线程安全

避免使用锁而带来的开销

{% highlight ruby %}
static std::shared_ptr<ShoppingManager> GetInstance()
{
    // 使用 C++11 的线程安全静态局部变量初始化单例
    static std::shared_ptr<ShoppingManager> s_Instance(new ShoppingManager());
    return s_Instance;
}
{% endhighlight %}

