---
layout: post
title: "工厂模式"
categories: C++
tags: [design-pattern]
---

## 简单工厂模式

- 纯虚类Shape
- 继承自Shape的Circle和Square类
- 工厂：包含接受函数和打印函数并且***创建实例***并且***包含判断语句***

{% highlight ruby %}
class Shape
{
public:
	virtual ~Shape() = default;
	virtual void Display() = 0;
};

class Circle :public Shape
{
public:
	virtual void Display() override
	{
		std::cout << "Circle Block" << std::endl;
	}
};

class Square :public Shape
{
public:
	virtual void Display() override
	{
		std::cout << "Square Block" << std::endl;
	}
};

class Factory
{
public:
	Factory()
	{
		production_vector.reserve(100);
	}
	void AcceptSystem(std::string type, int count)
	{
		for (int i = 0; i < count; i++)
		{
			if (type == "Circle")
			{
				production_vector.push_back(std::make_unique<Circle>());
			}
			else if (type == "Square")
			{
				production_vector.push_back(std::make_unique<Square>());
			}
		}
	}
	void ProduceSystem()
	{
		for (const auto& elem : production_vector)
		{
			elem->Display();
		}
	}
private:
	std::vector<std::unique_ptr<Shape>> production_vector;
};

int main()
{
	std::unique_ptr<Factory> factory = std::make_unique<Factory>();

	int num;
	std::cin >> num;
	std::string shape;
	int item_count;

	for (int i = 0; i < num; i++)
	{
		std::cin >> shape >> item_count;
		factory->AcceptSystem(shape, item_count);
	}
	factory->ProduceSystem();

	return 0;
}
{% endhighlight %}

## 工厂方法模式

- Shape的实现同简单工厂
- SquareFactory创建Square,CircleFactory创建Circle,只提供CreateInstance功能
- FactoryManager提供对外的接口，其中AcceptSystem以Factory类型为接受形参，接受SquareFactory和CircleFactory类型的实参（从主函数传过来），实现多态，从而调用CircleFactory和SquareFactory的创建方法

{% highlight ruby %}
class Shape
{
public:
	virtual ~Shape() = default;
	virtual void Display() = 0;
};

class Circle :public Shape
{
public:
	virtual void Display() override
	{
		std::cout << "Circle Block" << std::endl;
	}
};

class Square :public Shape
{
public:
	virtual void Display() override
	{
		std::cout << "Square Block" << std::endl;
	}
};

class Factory
{
public:
	virtual ~Factory() = default;
	virtual std::unique_ptr<Shape> CreateInstance() = 0;
};

class CircleFactory :public Factory
{
public:
	virtual std::unique_ptr<Shape> CreateInstance() override
	{
		return std::make_unique<Circle>();
	}
};

class SquareFactory :public Factory
{
public:
	virtual std::unique_ptr<Shape> CreateInstance() override
	{
		return std::make_unique<Square>();
	}
};

class FactoryManager
{
public:
	void AcceptSystem(const std::unique_ptr<Factory>& factory, int count)
	{
		for (int i = 0; i < count; i++)
		{
			production_vector.push_back(factory->CreateInstance());
		}	
	}
	void PrintSystem() const
	{
		for (const auto& elem : production_vector)
		{
			elem->Display();
		}
	}
private:
	std::vector<std::unique_ptr<Shape>> production_vector;
};

int main()
{
	FactoryManager factory_manager;

	int num;
	std::cin >> num;
	std::string shape;
	int item_count;

	for (int i = 0; i < num; i++)
	{
		std::cin >> shape >> item_count;
		if (shape == "Circle")
		{
			factory_manager.AcceptSystem(std::make_unique<CircleFactory>(), item_count);
		}
		else if (shape == "Square")
		{
			factory_manager.AcceptSystem(std::make_unique<SquareFactory>(), item_count);
		}
	}
	factory_manager.PrintSystem();

	return 0;
}
{% endhighlight %}