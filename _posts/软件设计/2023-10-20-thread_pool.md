---
title: 使用 c++11 实现一个线程池
category: [软件设计]
tags: [线程池, thread_pool]
---

> 线程池可以避免频繁地创建和销毁线程，从而减少了系统资源的消耗。它可以控制并发线程的数量，避免资源过度占用，并提供任务队列来存储等待执行的任务。线程池还可以根据需要动态调整线程的数量，以适应系统的负载情况。通过使用线程池，我们可以更好地管理线程的生命周期，提高程序的稳定性和可维护性。
{: .prompt-info }

## 代码实现

```c++
#include <vector>
#include <condition_variable>
#include <functional>
#include <future>
#include <mutex>
#include <queue>
#include <thread>

class ThreadPool {
public:
    explicit ThreadPool(size_t thread_count)
        : stop_(false) {
        for (size_t i = 0; i < thread_count; ++i) {
            workers_.emplace_back([this]() {
                for (;;) {
                    std::function<void()> task;
                    {
                        std::unique_lock<std::mutex> ul(mtx_);
                        cv_.wait(ul, [this]() { return stop_ || !tasks_.empty(); });
                        if (stop_ && tasks_.empty()) {
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

    ~ThreadPool() {
        {
            std::lock_guard<std::mutex> gl(mtx_);
            stop_ = true;
        }
        cv_.notify_all();
        for (auto &worker : workers_) {
            worker.join();
        }
    }

    template<typename F, typename... Args>
    auto submit(F &&f, Args &&...args) -> std::future<decltype(f(args...))> {
        auto task_ptr =
            std::make_shared<std::packaged_task<decltype(f(args...))()>>(std::bind(std::forward<F>(f), std::forward<Args>(args)...));
        {
            std::lock_guard<std::mutex> lg(mtx_);
            if (stop_) {
                throw std::runtime_error("submit on stopped ThreadPool");
            }
            tasks_.emplace([task_ptr]() { (*task_ptr)(); });
        }
        cv_.notify_one();
        return task_ptr->get_future();
    }

private:
    bool stop_;
    std::vector<std::thread> workers_;
    std::queue<std::function<void()>> tasks_;
    std::mutex mtx_;
    std::condition_variable cv_;
};
```