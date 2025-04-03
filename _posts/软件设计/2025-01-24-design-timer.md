---
title: 使用 c++ 实现一个高精度的定时器
category: [软件设计]
tags: [timer, timerfd]
---

> 在 Linux 应用开发中常常会用到定时器，其实定时器的实现方式有五六种，但是好多定时器的使用容易破坏 c++ 的封装，举个例子，你无法把一个类的非静态成员函数赋值给一个 struct sigaction 的成员 sa_handler 指针，如果你非要这样，只能采用静态成员函数，如果这个函数里要访问到类的多个成员变量，这些成员变量全部得改成静态成员变量，天哪，这违背了我的初心，我仅仅是想执行一个定时任务而已，却要破坏类原有得结构，所以最优雅的方式就是自己实现一个，下面我将以 timerfd 为例，实现一个高精度的定时器类。
{: .prompt-info }

## 代码

```c++
#ifndef __TIMER_H__
#define __TIMER_H__

#include <sys/timerfd.h>
#include <sys/poll.h>
#include <functional>
#include <cstdint>
#include <thread>

class Timer {
public:
    using TaskCallback = std::function<void(void)>;
    Timer()
    : m_is_running(false) {
        // TODO
    }

    ~Timer() {
        // TODO
    }

    void start() {
        m_thread = std::move(std::thread([&](){
            while(m_is_running) {
                // TODO
            }
        }));
    }

    void stop() {
        m_is_running = false;
        m_thread.join();
    }

private:
    bool m_is_running;
    std::thread m_thread;
};

#endif //__TIMER_H__
```