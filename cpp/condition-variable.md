---
title: c++11 条件变量
categories: cpp
date: 2019-08-01 15:18:10
tags: [cpp11, cpp]
---

比较常见的一个使用 std::condition_variable  场合就是线程池的消息队列。逻辑线程（可能多个）将消息推入消息队列，线程池中的工作线程（多个）会从消息队列中取出消息进行处理，如果队列中没有消息则进入睡眠状态等待消息。

本文将通过这种消息队列的实现，来分析如何使用 std::condition_variable 以及使用过程中的注意事项。
<!--more-->

先看下这个消息队列的最终实现：

```cpp
    void Push(void *msg)
    {
        std::unique_lock<std::mutex> lock(m_mutex);
        m_queue.push(msg);
        lock.unlock();
        m_cond.notify_one();

        return;
    }

    void * WaitAndPop()
    {
        void *msg = nullptr;

        while (true)
        {
            std::unique_lock<std::mutex> lock(m_mutex);
            if (!m_queue.empty())
            {
                msg = m_queue.front();
                m_queue.pop();
                return msg;
            }

            while(m_queue.empty()) m_cond.wait(lock);
        }

        // return nullptr;
    }
```

***
## 为什么需要搭配一个互斥量使用？

先假设不需要搭配互斥量使用，代码如下

```cpp
    // WaitAndPop
    mutex.lock();
    if (!queue.empty)
    {
        // pop msg
        ...
    }
    mutex.unlock();
    // 标注
    cond.wait();
```

queue 会被不同线程使用，所以需要一个锁来同步。
这个锁必须在 cond.wait 前解锁，否则工作线程进入睡眠状态导致逻辑线程的 Push 无法获得锁。
那么问题来了，当 WaitAndPop 执行到 mutex.unlock 后 cond.wait 前时，逻辑线程执行了 Push ，意味着 **cond.notify_one 在 cond.wait 前执行了**。结果就是 *工作线程进入睡眠，但是消息队列中还有一个消息没被处理* 。如果后续没有新消息，那这个消息就只能永远呆在队列中了。
std::condition_variable::wait 需要一个锁作参数基本上避免了这种情况，但是不排除有的同学将这个锁和用来同步queue操作的锁分开来而导致这种情况。
    
***
## Push 中调用 lock.unlock 和 cond.notify_one 的顺序问题

这是个性能优化的问题，谁先谁后对结果并没有影响。

* unlock 在前，notify_one 在后。
    工作线程在被唤醒前，逻辑线程已经解锁，这使得工作线程在唤醒后就能直接获得锁进入处理流程。

* notify_one  在前，unlock 在后。
    工作线程在被唤醒后，逻辑线程可能还没有解锁，这将导致工作线程无法获得锁而又进入睡眠状态等待锁。这里多了一次上下文切换，会损失一定性能。
    

***
## 虚假唤醒

虚假唤醒的意思是即使没有调用 cond.notify_one , cond.wait 也有可能返回。
留意下面这段代码：

```cpp
    // WaitAndPop
    std::unique_lock<std::mutex> lock(m_mutex);
    if (!m_queue.empty())  // 位置1
    {
        ...
    }

    while(m_queue.empty()) m_cond.wait(lock); // 位置2
```

*位置1* 就是对虚假唤醒的判断处理，这一步一定要做，而且还要在获得锁后做。

*位置2* 是对虚假唤醒的优化，避免虚假唤醒后去争夺锁。
while(m_queue.empty()) 在虚假唤醒后就不在 lock 的同步范围内了，但是因为这里只读，所以不存在同步问题。

