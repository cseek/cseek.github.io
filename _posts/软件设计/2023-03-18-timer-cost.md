---
title: 利用 raii 机制实现一个记录程序耗时的类
category: [软件设计]
tags: [time]
---

> c++ 中提供了一个计时的模板 std::chrono, 里面有三种时钟: steady_clock， system_clock 和 high_resolution_clock。
{: .prompt-tip }

+ std::steady_clock 类似秒表, 适合用于记录程序耗时；
+ std::system_clock 是系统的时钟, 因为系统的时钟可以修改, 甚至可以网络对时, 所以用系统时间计算时间差可能不准。
+ std::high_resolution_clock 是当前系统能够提供的最高精度的时钟, 它也是不可以修改的, 相当于 steady_clock 的高精度版本。

下面使用高精度始终实现一个记录程序耗时的类。

```c++
#include <chrono>
#include <iostream>
#include <string>

class TimeCost {
public:
    enum class Unit : uint8_t {
        S = 0,  // 秒
        MS = 1, // 毫秒
        US = 2, // 微秒
        NS = 3  // 纳秒
    };
    TimeCost(const std::string &m_tag, const Unit &unit = Unit::MS)
        : m_tag(m_tag), m_unit(unit),
          m_start_time(std::chrono::high_resolution_clock::now()) {}

    ~TimeCost() {
        auto end_time = std::chrono::high_resolution_clock::now();
        auto duration = std::chrono::duration_cast<std::chrono::nanoseconds>(end_time - m_start_time).count();
        if (Unit::S == m_unit) {
            std::cout << "[" << m_tag << "] cost: " << duration / (1000.0 * 1000.0 * 1000.0) << " s" << std::endl;
        } else if (Unit::MS == m_unit) {
            std::cout << "[" << m_tag << "] cost: " << duration / (1000.0 * 1000.0) << " ms" << std::endl;
        } else if (Unit::NS == m_unit) {
            std::cout << "[" << m_tag << "] cost: " << duration / 1000.0 << " us" << std::endl;
        } else {
            std::cout << "[" << m_tag << "] cost: " << duration << " ns" << std::endl;
        }
    }

private:
    Unit m_unit;
    std::string m_tag;
    std::chrono::time_point<std::chrono::high_resolution_clock> m_start_time;
};

// 测试
#include <unistd.h>

int main() {
    {
        TimeCost t("tag1");
        sleep(1);
    }
    {
        TimeCost t("tag1", TimeCost::Unit::S);
        sleep(1);
    }
    {
        TimeCost t("tag2", TimeCost::Unit::MS);
        sleep(1);
    }
    {
        TimeCost t("tag3", TimeCost::Unit::US);
        sleep(1);
    }
    {
        TimeCost t("tag3", TimeCost::Unit::NS);
        sleep(1);
    }
    return 0;
}
```