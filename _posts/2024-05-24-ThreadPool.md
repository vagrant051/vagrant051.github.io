---
layout: post
title: "线程池的实现与分析"
categories: operating-system
tags: [Thread]
---

这是一个固定线程数量的线程池

## 背景

通过使用线程池，我们可以有效降低多线程操作中任务申请和释放产生的性能消耗。特别是当我们每个线程的任务处理比较快时，系统大部分性能消耗都花在了pthread_create以及释放线程的过程中。那既然是这样的话，何不在程序开始运行阶段提前创建好一堆线程，等我们需要用的时候只要去这一堆线程中领一个线程，用完了再放回去，等程序运行结束时统一释放这一堆线程呢？按照这个想法，线程池出现了。

## 线程池原理

![My helpful screenshot](/assets/Thread-Pool/1.png)

使用了队列来存储任务，并使用互斥量和条件变量来实现线程之间的同步。enqueue()函数用于将任务添加到队列中，线程池中的线程将从队列中取出任务并执行。

## 各部分代码解释

{% highlight ruby %}
ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back(
                [this] {
                    while (true) {
                        std::function<void()> task;
                        {
                            std::unique_lock<std::mutex> lock(this->queue_mutex);
                            this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                            if (this->stop && this->tasks.empty())
                                return;
                            task = std::move(this->tasks.front());
                            this->tasks.pop();
                        }
                        task();
                    }
                }
            );
        }
    }
{% endhighlight %}

当线程启动时，它会进入 while 循环，准备从任务队列中获取任务并执行。

然后，线程会获取任务队列的互斥锁，以确保在访问任务队列时的线程安全。

一旦获取了互斥锁，线程会调用条件变量 condition 的 wait() 函数，等待条件变为真。这个条件是 stop 标志位被设置为 true 或者任务队列不为空。如果条件为假，线程会被阻塞，等待条件变为真时再继续执行。

当有任务被添加到任务队列中，或者线程池被要求停止时，条件变为真，线程会从 wait() 函数中返回，并继续执行。

在条件变为真后，线程会检查 stop 标志位是否为 true，以及任务队列是否为空。如果 stop 标志位为 true 且任务队列为空，则线程会直接返回，退出循环，结束线程的执行。

否则，线程会从任务队列中取出一个任务，释放互斥锁，并执行局部变量 task 中存储的任务。

循环会继续迭代，线程会再次尝试从任务队列中获取任务并执行，直到收到停止信号并且任务队列为空时，线程才会退出循环，结束执行。

{% highlight ruby %}
template<class F>
    void enqueue(F&& f) {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            tasks.emplace(std::forward<F>(f));
        }
        condition.notify_one();
    }
{% endhighlight %}

在添加任务到任务队列之后，调用 condition 条件变量的 notify_one() 函数，通知一个等待中的线程有新任务可执行。这样可以避免因为任务队列中有新任务而导致的线程处于等待状态。

## 代码

{% highlight ruby %}
class ThreadPool {
public:
    ThreadPool(size_t numThreads) : stop(false) {
        for (size_t i = 0; i < numThreads; ++i) {
            workers.emplace_back(
                [this] {
                    while (true) {
                        std::function<void()> task;
                        {
                            std::unique_lock<std::mutex> lock(this->queue_mutex);
                            this->condition.wait(lock, [this] { return this->stop || !this->tasks.empty(); });
                            if (this->stop && this->tasks.empty())
                                return;
                            task = std::move(this->tasks.front());
                            this->tasks.pop();
                        }
                        task();
                    }
                }
            );
        }
    }

    template<class F>
    void enqueue(F&& f) {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            tasks.emplace(std::forward<F>(f));
        }
        condition.notify_one();
    }

    ~ThreadPool() {
        {
            std::unique_lock<std::mutex> lock(queue_mutex);
            stop = true;
        }
        condition.notify_all();
        for (std::thread &worker : workers)
            worker.join();
    }

private:
    std::vector<std::thread> workers;
    std::queue<std::function<void()>> tasks;
    std::mutex queue_mutex;
    std::condition_variable condition;
    bool stop;
};

void printHello() {
    std::cout << "Hello from thread " << std::this_thread::get_id() << std::endl;
}

int main() {
    ThreadPool pool(4); // 创建一个包含4个线程的线程池

    // 添加打印任务到线程池
    for (int i = 0; i < 8; ++i) {
        pool.enqueue([] { printHello(); });
    }

    return 0;
}
{% endhighlight %}


