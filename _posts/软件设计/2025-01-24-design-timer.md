---
title: 使用 c++ 实现一个定时器
category: [软件设计]
tags: [timer, timerfd]
---

> 在 Linux 应用开发中常常会用到定时器，其实定时器的实现方式有五六种，但是好多定时器的使用容易破坏 c++ 的封装，举个例子，你无法把一个类的非静态成员函数赋值给一个 struct sigaction 的成员 sa_handler 指针，如果你非要这样，只能采用静态成员函数，如果这个函数里要访问到类的多个成员变量，这些成员变量全部得改成静态成员变量，天哪，这违背了我的初心，我仅仅是想执行一个定时任务而已，却要破坏类原有得结构，所以最优雅的方式就是自己实现一个，下面我将以 timerfd 为例，实现一个毫秒级定时器类，误差 100 微妙左右。
{: .prompt-info }

## 具体实现

`timer.h`

```c++
#ifndef __TIMER_H__
#define __TIMER_H__

#include <stdio.h>
#include <sys/epoll.h>
#include <sys/eventfd.h>
#include <sys/timerfd.h>
#include <unistd.h>
#include <thread>
#include <atomic>
#include <cstdint>
#include <cstring>
#include <functional>
#include <chrono>
#include <cerrno>

class Timer {
public:
    using TaskCallback = std::function<void(void)>;
    Timer(uint64_t ms, const TaskCallback &callback)
    : m_is_running(false)
    , m_timer_fd(-1)
    , m_interval_ms(ms)
    , m_callback(callback) {}

    ~Timer() {}
    void start() {
        if (m_is_running) {
            return;
        }
    
        m_timer_fd = timerfd_create(CLOCK_MONOTONIC, 0);
        if (m_timer_fd == -1) {
            perror("timerfd_create failed");
            return;
        }
    
        struct itimerspec its;
        memset(&its, 0, sizeof(its));
        uint64_t ms = m_interval_ms;
        its.it_interval.tv_sec = ms / 1000;
        its.it_interval.tv_nsec = (ms % 1000) * 1000000;
        its.it_value = its.it_interval;
    
        if (timerfd_settime(m_timer_fd, 0, &its, nullptr) == -1) {
            perror("timerfd_settime failed");
            close(m_timer_fd);
            m_timer_fd = -1;
            return;
        }
    
        m_is_running = true;
    
        m_thread = std::thread([&]() {
            while (m_is_running) {
                uint64_t expirations;
                ssize_t bytes = read(m_timer_fd, &expirations, sizeof(expirations));
                if (bytes == sizeof(expirations)) {
                    if (expirations > 0) {
                        for (uint64_t i = 0; i < expirations; ++i) {
                            m_callback();
                        }
                    }
                } else {
                    if (errno == EBADF || !m_is_running) {
                        break;
                    }
                    perror("read timerfd failed");
                }
            }
        });
    }
    
    void stop() {
        if (!m_is_running) {
            return;
        }
    
        m_is_running = false;
    
        if (m_timer_fd != -1) {
            close(m_timer_fd);
            m_timer_fd = -1;
        }
    
        if (m_thread.joinable()) {
            m_thread.join();
        }
    }
    
private:
    int m_timer_fd;
    std::atomic<bool> m_is_running;
    std::thread m_thread;
    uint64_t m_interval_ms;
    TaskCallback m_callback;
};

#endif // __TIMER_H__

#include <iostream>
#include <signal.h>

bool is_running(true);
void handle(int sig) {
    is_running = false;
}

int main() {
    signal(SIGINT, handle);
    auto now = std::chrono::high_resolution_clock::now();
    Timer timer(1000, [&]() {
        auto end = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::nanoseconds>(end - now).count();
        std::cout << "cost: " << duration / 1000.0 << " us" << std::endl;
        now = end;
    });
    timer.start();
    while (is_running) {
        std::this_thread::sleep_for(std::chrono::milliseconds(50));
    }
    timer.stop();
    return 0;
}
```
