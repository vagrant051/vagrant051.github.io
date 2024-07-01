---
layout: post
title: "观察者模式"
categories: C++
tags: [design-pattern]
---

## 观察者模式

- 包含抽象观察者类和具体观察者类，包含update函数
- 包含抽象主题类和具体主题类，包含observer_list和notify函数，其中notify函数搜索observer_list并调用各个observer的update函数

{% highlight ruby %}
{
public:
	virtual ~Observer() = default;
	virtual void Update(int hour) = 0;
};

class StudentObserver :public Observer
{
public:
	StudentObserver() = default;
	StudentObserver(const std::string& name) :m_Name(name) {}
	virtual void Update(int hour) override
	{
		std::cout << m_Name << " " << hour << std::endl;
	}
private:
	std::string m_Name;
};

class Subject
{
public:
	virtual ~Subject() = default;
	virtual void Attach(std::shared_ptr<Observer> observer) = 0;
	virtual void Detach(std::shared_ptr<Observer> observer) = 0;
	virtual void Notify() = 0;
};

class ClockSubject :public Subject
{
public:
	virtual void Attach(std::shared_ptr<Observer> observer) override
	{
		if (std::find(observer_list.begin(), observer_list.end(), observer) == observer_list.end())
		{
			observer_list.emplace_back(observer);
		}
	}

	virtual void Detach(std::shared_ptr<Observer> observer) override
	{
		observer_list.remove(observer);
	}
	virtual void Notify() override
	{
		hour++;
		for (auto elem : observer_list)
		{
			elem->Update(hour);
		}
	}
private:
	int hour = 0;
	std::list<std::shared_ptr<Observer>> observer_list;
};

int main()
{
	std::unique_ptr<Subject> clock = std::make_unique<ClockSubject>();

	std::shared_ptr<Observer> Alice = std::make_shared<StudentObserver>("Alice");
	std::shared_ptr<Observer> Bob = std::make_shared<StudentObserver>("Bob");

	clock->Attach(Alice);
	clock->Attach(Bob);

	clock->Notify();
	clock->Notify();
	clock->Notify();

	return 0;
}
{% endhighlight %}
