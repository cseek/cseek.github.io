---
title: 使用 c++ 实现一个精简的线程池
category: [软件设计]
tags: [线程池, thread_pool]
---

> 线程池可以避免频繁地创建和销毁线程，从而减少了系统资源的消耗。它可以控制并发线程的数量，避免资源过度占用，并提供任务队列来存储等待执行的任务。线程池还可以根据需要动态调整线程的数量，以适应系统的负载情况。通过使用线程池，我们可以更好地管理线程的生命周期，提高程序的稳定性和可维护性。下面这个线程池实现借鉴自极飞科技的 xnet_sdk。
{: .prompt-info }

## 代码实现
`thread_pool.h`

```c++
#ifndef THREAD_POOL_H
#define THREAD_POOL_H

#include <cstdint>
#include <vector>
#include <condition_variable>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>
#include <memory>

class ThreadPool {
public:
    ThreadPool()
        : stop_(false) {
    }

    ~ThreadPool() {
        deinit();
    }

    void init(uint32_t thread_count) {
        for (size_t i = 0; i < thread_count; ++i) {
            workers_.emplace_back([this]() {
                for (;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> ul(mtx_);
                        cv_.wait(ul, [this]() { return stop_ || !tasks_.empty(); });
                        if (stop_) {
                            return;
                        }
                        task = std::move(tasks_.front());
                        tasks_.pop();
                    }
                    task();
                }
            });
        }
    }

    void deinit() {
        {
            std::lock_guard<std::mutex> gl(mtx_);
            stop_ = true;
        }
        cv_.notify_all();
        for (auto &worker : workers_) {
            if (worker.joinable()) {
                worker.join();
            }
        }
        workers_.clear();
        std::queue<std::function<void()>> tmp;
        tasks_.swap(tmp);
    }

    template<typename F, typename... Args>
    auto submit(F &&f, Args &&...args) -> std::future<decltype(f(args...))> {
        if (stop_) {
            return {};
        }
        auto task_ptr =
            std::make_shared<std::packaged_task<decltype(f(args...))()>>(std::bind(std::forward<F>(f), std::forward<Args>(args)...));
        {
            std::lock_guard<std::mutex> lg(mtx_);
            if (!stop_) {
                tasks_.emplace([task_ptr]() { (*task_ptr)(); });
            }
        }
        cv_.notify_one();
        return task_ptr->get_future();
    }

private:
    bool                              stop_;
    std::mutex                        mtx_;
    std::condition_variable           cv_;
    std::vector<std::thread>          workers_;
    std::queue<std::function<void()>> tasks_;
};
#endif // THREAD_POOL_H
```

## 使用示例

```c++
#include "thread_pool.h"
#include <iostream>

int main() {
    ThreadPool tp;
    tp.init(1);
    auto future = tp.submit([]() {
        std::cout << "thread 1\n";
        std::this_thread::sleep_for(std::chrono::seconds(3));
    });
    future.wait();
    tp.deinit();

    return 0;
}

```
