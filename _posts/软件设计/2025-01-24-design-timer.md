---
title: 使用 c++ 实现一个高精度的定时器
category: [软件设计]
tags: [timer, timerfd]
---

> 在 Linux 应用开发中常常会用到定时器，其实定时器的实现方式有五六种，但是好多定时器的使用容易破坏 c++ 的封装，举个例子，你无法把一个类的非静态成员函数赋值给一个 struct sigaction 的成员 sa_handler 指针，如果你非要这样，只能采用静态成员函数，如果这个函数里要访问到类的多个成员变量，这些成员变量全部得改成静态成员变量，天哪，这违背了我的初心，我仅仅是想执行一个定时任务而已，却要破坏类原有得结构，所以最优雅的方式就是自己实现一个，下面我将以 timerfd 为例，实现一个高精度的定时器类。
{: .prompt-info }

## 具体实现

`timer.h`

```c++
#ifndef __TIMER_H__
#define __TIMER_H__

#include <stdio.h>
#include <sys/poll.h>
#include <sys/timerfd.h>
#include <unistd.h>
#include <thread>
#include <atomic>
#include <cstdint>
#include <cstring>
#include <functional>

class Timer {
public:
    using TaskCallback = std::function<void(void)>;
    Timer()
    : m_is_running(false)
    , m_timer_fd(-1)
    , m_interval_ms(0) {

    }
    ~Timer() {
        stop();
    }
    void set_interval(uint64_t ms) {
        m_interval_ms = ms;
    }
    void set_callback(const TaskCallback &cb) {
        m_callback = cb;
    }
    void start() {
        if (m_is_running) {
            return;
        }

        // 创建定时器文件描述符
        m_timer_fd = timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK);
        if (m_timer_fd == -1) {
            perror("timerfd_create failed");
            return;
        }

        // 设置定时器参数
        struct itimerspec its;
        memset(&its, 0, sizeof(its));
        its.it_interval.tv_sec = m_interval_ms / 1000;
        its.it_interval.tv_nsec = (m_interval_ms % 1000) * 1000000;
        its.it_value = its.it_interval; // 首次超时时间与间隔相同

        if (timerfd_settime(m_timer_fd, 0, &its, nullptr) == -1) {
            perror("timerfd_settime failed");
            close(m_timer_fd);
            m_timer_fd = -1;
            return;
        }

        m_is_running = true;
        m_thread = std::thread([this]() {
            struct pollfd pfd;
            pfd.fd = m_timer_fd;
            pfd.events = POLLIN;

            while (m_is_running) {
                int ret = poll(&pfd, 1, -1); // 阻塞等待事件
                if (ret < 0) {
                    if (errno == EINTR) {
                        continue; // 被信号中断，继续等待
                    }
                    perror("poll failed");
                    break;
                }

                if (pfd.revents & POLLIN) {
                    uint64_t expirations;
                    ssize_t bytes_read = read(m_timer_fd, &expirations, sizeof(expirations));
                    if (bytes_read != sizeof(expirations)) {
                        perror("read timerfd failed");
                        continue;
                    }

                    if (m_callback && expirations > 0) {
                        for (uint64_t i = 0; i < expirations; ++i) {
                            m_callback();
                        }
                    }
                }
            }

            // 清理资源
            if (m_timer_fd != -1) {
                close(m_timer_fd);
                m_timer_fd = -1;
            }
        });
    }

    void stop() {
        if (!m_is_running) {
            return;
        }

        m_is_running = false;
        if (m_thread.joinable()) {
            m_thread.join();
        }
    }

private:
    int m_timer_fd;
    uint64_t m_interval_ms;
    TaskCallback m_callback;
    std::thread m_thread;
    std::atomic<bool> m_is_running;
};

#endif // __TIMER_H__
```

## 使用案例

`main.cpp`

```c++
#include "timer.h"
#include <unistd.h>
#include <iostream>

int main() {
    Timer timer;
    timer.set_interval(500);
    timer.set_callback([] {
        std::cout << "hello world\n";
    });
    timer.start();
    sleep(20);
    timer.stop();
    return 0;
}
```
