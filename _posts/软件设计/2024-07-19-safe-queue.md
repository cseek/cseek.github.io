---
title: 使用 c++ 实现安全队列
category: [软件设计]
tags: [安全队列, queue]
---
> 在多线程开发中，例如"生产者-消费者模型"中，常常需要用到线程安全队列，线程安全队列主要分为阻塞或者非阻塞的两个版本，阻塞版本在 pop 数据时会阻塞等待数据的到来，能节省 cpu 资源，非阻塞版本则适合需要跑满 cpu 的场景。
{: .prompt-info }

## 阻塞版安全队列

```c++
#ifndef __SAFE_QUEUE_H__
#define __SAFE_QUEUE_H__

#include <iostream>
#include <cstdint>
#include <cstddef>
#include <queue>
#include <mutex>
#include <condition_variable>

template<typename T>
class SafeQueue {
public:
    SafeQueue(uint32_t cap = 200)
        : m_cap(cap)
        , m_stop(false) {
    }

    void push(const T &value) {
        std::lock_guard<std::mutex> glck(m_mutex);
        if (m_queue.size() >= m_cap) {
            std::cout << "Queue is full, waiting for pop\n";
        } else {
            m_queue.push(value);
        }
        m_cv.notify_one();
    }

    // 使用while循环等待队列不为空
    // 防止虚假唤醒（spurious wakeup）
    // 当多个线程等待在条件变量上时，一个线程被唤醒后可能发现队列为空，
    // 所以需要循环检查队列是否为空
    bool pop(T &value) {
        std::unique_lock<std::mutex> ulck(m_mutex);
        while (m_queue.empty() && !m_stop) {
            m_cv.wait(ulck, [this]() { 
                return !m_queue.empty() || m_stop;
            }); // 阻塞当前线程，并释放锁
        }
        if (m_stop) {
            return false;
        }
        value = m_queue.front();
        m_queue.pop();
        return true;
    }

    void stop() {
        m_stop = true;
        m_cv.notify_all();
    }

    void clear() {
        std::lock_guard<std::mutex> glck(m_mutex);
        while (!m_queue.empty()){
            m_queue.pop();
        }
    }

    size_t size() {
        std::lock_guard<std::mutex> glck(m_mutex);
        return m_queue.size();
    }

private:
    bool m_stop;                   // 用于唤醒等待, 退出线程
    uint32_t m_cap;                // 容量
    std::queue<T> m_queue;         // 存储数据的队列
    std::mutex m_mutex;            // 互斥锁，保证对队列的访问是线程安全的
    std::condition_variable m_cv;  // 条件变量，用于实现线程间的同步
};
#endif // __SAFE_QUEUE_H__
```

## 非阻塞版安全队列

```c++
#ifndef __SAFE_QUEUE_H__
#define __SAFE_QUEUE_H__

#include <iostream>
#include <cstdint>
#include <cstddef>
#include <queue>
#include <mutex>

template<typename T>
class SafeQueue {
public:
    SafeQueue(uint32_t cap = 200)
        : m_cap(cap) {
    }

    void push(const T &value) {
        std::lock_guard<std::mutex> glck(m_mutex);
        if (m_queue.size() >= m_cap) {
            std::cout << "Queue is full, waiting for pop\n";
        } else {
            m_queue.push(value);
        }
    }

    bool pop(T &value) {
        std::lock_guard<std::mutex> glck(m_mutex);
        if (m_queue.empty()) {
            return false;
        }
        value = m_queue.front();
        m_queue.pop();
        return true;
    }

    void clear() {
        std::lock_guard<std::mutex> glck(m_mutex);
        while (!m_queue.empty()){
            m_queue.pop();
        }
    }

    size_t size() {
        std::lock_guard<std::mutex> glck(m_mutex);
        return m_queue.size();
    }

private:
    uint32_t m_cap;                // 容量
    std::queue<T> m_queue;         // 存储数据的队列
    std::mutex m_mutex;            // 互斥锁，保证对队列的访问是线程安全的
};
#endif // __SAFE_QUEUE_H__
```