---
layout: post
title: "线程同步机制分析"
categories: operating-system
tags: [Thread]
---

互斥锁，条件变量，信号量，原子操作，读写锁，自旋锁

# 背景

进程有自己的独立地址空间，因此进程之间重点关注通信，通信方式包括：管道Pipe、命名管道FIFO、消息队列MessageQueue、共享存储SharedMemory、信号量Semaphore、套接字Socket和信号Signal。

线程除了线程栈外其他数据都是共享的，如果同时读写数据可能造成数据不一致甚至程序崩溃的后果，因此线程之间重点关注同步。

## 互斥锁（mutex）

互斥锁强调的是资源之间的访问互斥：每个线程在对共享资源操作前都会尝试先加锁，加锁成功才能操作，操作结束之后解锁。

某个线程对互斥量加锁后，任何其他试图再对互斥量加锁的线程都将被阻塞直到当前线程释放该互斥锁。如果释放互斥锁时有多个线程阻塞，所有在该互斥锁上的阻塞线程都会变成可运行状态。第一个变成运行状态的线程可以对互斥量加锁，其余线程将会看到互斥量依然被锁住，只能回去再次等待它重新变为可用。

{% highlight ruby %}
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void threadFunction() {
    mtx.lock();
    // 访问共享资源的代码
    std::cout << "Hello from thread!" << std::endl;
    mtx.unlock();
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);

    t1.join();
    t2.join();

    return 0;
}
{% endhighlight %}

## 条件变量

并发有互斥和等待两大需求，前者是因为线程间存在共享数据依赖而后者是线程间存在依赖，条件变量正是为了解决等待需求。

在主线程睡眠的过程中，t1线程可能会在等待条件变量的过程中被阻塞，因为它在等待条件变量时会释放互斥锁。但这不会影响主线程的睡眠或继续执行。

### std::lock_guard和std::unique_lock的区别

如果只需要在作用域内自动锁定和解锁互斥锁，并且不需要手动控制锁定和解锁的时机，那么选择std::lock_guard。但如果需要更多的灵活性，比如手动锁定和解锁互斥锁，或者需要与条件变量一起使用，那么std::unique_lock可能更适合。

{% highlight ruby %}
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void threadFunction() {
    std::unique_lock<std::mutex> lck(mtx);
    // 等待条件满足
    cv.wait(lck, [] { return ready; });
    // 条件满足后执行的代码
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    std::thread t1(threadFunction);

    // 一段时间后设置条件为true
    std::this_thread::sleep_for(std::chrono::seconds(2));
    {
        std::unique_lock  <std::mutex> lck(mtx);
        ready = true;
    }
    // 通知等待的线程条件已满足
    cv.notify_one();

    t1.join();

    return 0;
}
{% endhighlight %}

## 信号量

那信号量值n代表什么意思？

- n>0：当前有可用资源，可用资源数量为n
- n=0：资源都被占用，可用资源数量为0
- n<0：资源都被占用，并且还有n个进程正在排队

![My helpful screenshot](/assets/Thread-synchronization/1.png)

## 原子操作

原子操作是不可分割的操作，能够保证多个线程并发执行时对共享变量的操作不会产生竞态条件。在C++中，可以使用std::atomic模板来创建原子变量。

{% highlight ruby %}
#include <iostream>
#include <thread>
#include <atomic>

std::atomic<int> counter(0);

void threadFunction() {
    for (int i = 0; i < 1000; ++i) {
        counter++;
    }
}

int main() {
    std::thread t1(threadFunction);
    std::thread t2(threadFunction);

    t1.join();
    t2.join();

    std::cout << "Counter: " << counter << std::endl;

    return 0;
}
{% endhighlight %}

## 读写锁

允许多个读线程同时访问共享资源，但在写线程访问时，会阻止所有其他线程的访问。读写锁适用于读多写少的场景，因为它可以提高并发性能。

shared_mutex是在C++17中使用的一个类，该类主要作为同步基元使用。该类可以保护共享资源不被多个线程同时访问，与其他的锁相比，该类具有两个锁类型：

- 共享锁
- 独占锁

共享锁和独占锁之间的关系决定了shared_mutex的特性，即

- 若某个线程获取了独占锁，那么其余所有线程都无法获取独占锁和共享锁。
- 若某个线程获取了共享锁，那么其余线程只可以获取共享锁却不可以获取独占锁。
- 每个线程只可以获取一种锁

这三条性质意味着只有在独占锁没有被任何线程获取时，多个线程才可以使用共享锁。

### std::shared_lock和std::unique_lock区别

- std::shared_lock用于共享（读）锁定。这意味着多个线程可以同时持有共享锁，但只有当没有任何线程持有独占（写）锁时，才能获取共享锁。
- std::unique_lock用于独占（写）锁定。这意味着只有一个线程可以持有独占锁，其他线程（无论是读还是写）在该线程释放锁之前都将被阻塞。

{% highlight ruby %}
#include <iostream>
//std::unique_lock
#include <mutex> 
#include <shared_mutex>
#include <thread>
 
class ThreadSafeCounter {
public:
	ThreadSafeCounter() = default;
 
	// 多个线程/读者能同时读计数器的值。
	unsigned int get() const {
		std::shared_lock<std::shared_mutex> lock(mutex_);
		return value_;
	}
 
	// 只有一个线程/写者能增加/写线程的值。
	void increment() {
		std::unique_lock<std::shared_mutex> lock(mutex_);
		value_++;
	}
 
	// 只有一个线程/写者能重置/写线程的值。
	void reset() {
		std::unique_lock<std::shared_mutex> lock(mutex_);
		value_ = 0;
	}
 
private:
	mutable std::shared_mutex mutex_;
	unsigned int value_ = 0;
};
 
int main() {
	ThreadSafeCounter counter;
 
	auto increment_and_print = [&counter]() {
		for (int i = 0; i < 3; i++) {
			counter.increment();
			std::cout << std::this_thread::get_id() << '\t' << counter.get() << std::endl;
		}
	};
 
	std::thread thread1(increment_and_print);
	std::thread thread2(increment_and_print);
 
	thread1.join();
	thread2.join();
 
	system("pause");
	return 0;
}
{% endhighlight %}

## 自旋锁

自旋锁是一种锁机制，它允许线程在等待锁的过程中不断地循环检查锁的状态，而不是像互斥锁那样让线程进入睡眠状态。自旋锁的主要优点是在锁的持有时间很短时，它可以避免线程上下文切换的开销。自旋锁通常用于多处理器系统上的短临界区。

### 函数解释

test_and_set函数原子地将std::atomic_flag的标志位设置为true，并返回设置之前的值。

如果之前的值为false，则设置成功，表示获得了锁；如果之前的值为true，则表示锁已被其他线程持有，当前线程将继续忙等待。

clear函数原子地将std::atomic_flag的标志位设置为false，释放锁。

{% highlight ruby %}
#include <atomic>
#include <thread>
#include <iostream>
#include <vector>

class SpinLock {
private:
    std::atomic_flag flag = ATOMIC_FLAG_INIT;

public:
    void lock() {
        while (flag.test_and_set(std::memory_order_acquire)) {
            // Busy-wait (spin) until the lock is acquired
        }
    }

    void unlock() {
        flag.clear(std::memory_order_release);
    }
};

SpinLock spinLock;
int counter = 0;

void incrementCounter() {
    for (int i = 0; i < 100000; ++i) {
        spinLock.lock();
        ++counter;
        spinLock.unlock();
    }
}

int main() {
    std::vector<std::thread> threads;

    // 创建多个线程
    for (int i = 0; i < 10; ++i) {
        threads.emplace_back(incrementCounter);
    }

    // 等待所有线程完成
    for (auto& thread : threads) {
        thread.join();
    }

    std::cout << "Final counter value: " << counter << std::endl;

    return 0;
}
{% endhighlight %}