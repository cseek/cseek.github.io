---
title: 在 jetson orin nx 上 使用 sysfs 操作 gpio
categories: [常用技巧]
tags: [gpio]
---

> 在前面我介绍了如何查早资料并通过 libgpiod 来操作 GPIO。本篇主要是记录一下使用 sysfs 操作 GPIO，下面就以 `jetson orin nx` 为例讲述一下如何使用。
{: .prompt-info }

## 怎样计算 GPIO index?
在 “在 jetson orin nx 上 使用 libgpiod 库操作 gpio“ 中，通过 libgpiod 的命令行工具 gpioinfo 是可以输出 GPIO port 对应的 line ， 然后通过 line 来控制 GPIO；但是在旧的系统中是没有 libgpiod 的支持的，所以只能通过公式换算 gpio index, 用来通过 sysfs 控制 GPIO；下面是换算的方法：

1. Orin GPIO 以 bank(组) 分组，每个 bank 最多包含 8 个引脚。GPIO 的 bank 和引脚信息是计算其索引的基础。GPIO 表示为X.Y，其中：
+ X 表示对 bank 进行编号，可以具有以下值：PA、PB、...、PY 或 PAA、...PFF，如下表所列。
+ 对于采用 P* 命名方案的 bank，必须忽略“P”，即“PA”指的是 bank A。
+ Y 表示 bank 内引脚的编号。例如，00 表示引脚 0，01 表示引脚 1，……，07 表示引脚 7。

2. 这些 bank 与一个 base 相关联，也可以按顺序转换为 bank_index，如下表所示：

![](/assets/img/post/常用技巧/orin-bank.webp)

如果在内核中进行了更改，则 base 编号可能会发生变化，可以使用 dmesg 日志进行识别：

```bash
nvidia@tegra-ubuntu:~$ cat /sys/kernel/debug/gpio | grep gpiochip
[ 9.965908] gpiochip0: registered GPIOs 348 to 511 on tegra234-gpio
[ 9.970024] gpiochip1: registered GPIOs 316 to 347 on tegra234-gpio-aon
```
从上面的日志可以看出，tegra234-gpio 的 base 为 348，tegra234-gpio-aon 的 base 为 316。

3. GPIO index 可以计算为：GPIO_index = base + bank_start_index + pin。

以下是计算 PX.01 的 GPIO index 的示例。

PX.01 代表 bank X 和引脚 1。
Bank X 的 bank_index 为 114，base 为 348（tegra-gpio base）
GPIO index 为 463（463 = 348 + 114 + 1）

## 怎样控制 GPIO ？

下面是使用 c++ 实现的控制接口

`gpio.h`
```c++
#ifndef __GPIO_H__
#define __GPIO_H__

#include <cstdint>
#include <string>

class Gpio {
public:
    enum class Mode 
        : uint8_t {
        OUTPUT,  // 输出
        INPUT    // 输入
    };
    enum class Value
        : uint8_t {
        LOW  = 0, // 低电平
        HIGH = 1  // 高电平
    };
    enum class Trigger
        : uint8_t {
        NONE,
        RISING,   // 上升沿
        FALLING,  // 下降沿
        BOTH      // 双边沿
    };
    Gpio(const std::string &port, int gpio_num);
    // 初始化, export
    bool init();
    // 反初始化，unexport
    bool deinit();
    // 设置方向
    bool set_direction(const Mode &mode);
    // 设置触发方式
    bool set_trigger(const Trigger& trigger);
    // 设置电平
    bool set_value(const Value &value);
    // 获取电平
    auto get_value() -> Value;

private:
    int m_gpio_num;
    std::string m_port;
};

#endif //__GPIO_H__
```

`gpio.cpp`

```c++
#include "gpio.h"
#include "logger.h" // 我在其他文章中提到过，自己封装的 spdlog
#include "spdlog/fmt/fmt.h"
#include <fstream>
#include <string>
#include <sys/stat.h>
#include <unistd.h>

Gpio::Gpio(const std::string &port, int gpio_num)
    : m_port(port)
    , m_gpio_num(gpio_num)
    {}

bool Gpio::init() {
    struct stat st;
    auto gpio_dir = fmt::format("/sys/class/gpio/{}", m_port);
    // 检查 GPIO 是否已导出
    if (stat(gpio_dir.c_str(), &st) == 0) {
        return true;
    }
    std::ofstream export_file("/sys/class/gpio/export");
    if (!export_file) {
        LOGW("Failed to open export file");
        return false;
    }
    export_file << m_gpio_num;
    if (export_file.fail()) {
        LOGW("Failed to write export file");
        return false;
    }
    // 等待导出完成
    usleep(200000);
    return true;
}

bool Gpio::deinit() {
    struct stat st;
    auto gpio_dir = fmt::format("/sys/class/gpio/{}", m_port);
    // 检查 GPIO 是否已取消导出
    if (stat(gpio_dir.c_str(), &st) != 0) {
        return true;
    }
    std::ofstream unexport_file("/sys/class/gpio/unexport");
    if (!unexport_file) {
        LOGW("Failed to open unexport file");
        return false;
    }
    unexport_file << m_gpio_num;
    if (unexport_file.fail()) {
        LOGW("Failed to write unexport file");
        return false;
    }
    usleep(100000);
    return true;
}

bool Gpio::set_direction(const Mode &mode) {
    auto path = fmt::format("/sys/class/gpio/{}/direction", m_port);
    std::ofstream file(path);
    if (!file) {
        LOGW("Failed to open {}", path);
        return false;
    }
    switch (mode) {
        case Mode::OUTPUT:
            file << "out";
            break;
        case Mode::INPUT:
            file << "in";
            break;
        default:
            return false;
    }
    return !file.fail();
}

bool Gpio::set_trigger(const Trigger &trigger) {
    auto path = fmt::format("/sys/class/gpio/{}/edge", m_port);
    std::ofstream file(path);
    if (!file) {
        LOGW("Failed to open {}", path);
        return false;
    }
    switch (trigger) {
        case Trigger::NONE:
            file << "none";
            break;
        case Trigger::RISING:
            file << "rising";
            break;
        case Trigger::FALLING:
            file << "falling";
            break;
        case Trigger::BOTH:
            file << "both";
            break;
        default:
            return false;
    }
    return !file.fail();
}

bool Gpio::set_value(const Value &value) {
    auto path = fmt::format("/sys/class/gpio/{}/value", m_port);
    std::ofstream file(path);
    if (!file) {
        LOGW("Failed to open {}", path);
        return false;
    }

    file << static_cast<int>(value);
    return !file.fail();
}

Gpio::Value Gpio::get_value() {
    auto path = fmt::format("/sys/class/gpio/{}/value", m_port);
    std::ifstream file(path);
    if (!file) {
        LOGW("Failed to open {}", path);
        return Value::LOW; // 默认返回低电平
    }
    int val;
    file >> val;
    return (val == 0) ? Value::LOW : Value::HIGH;
}
```

使用

```c++
// 注意：此代码只在米尔的 STM32MP257 开发板上验证过，此板运行基于 debian 的衍生 linux 系统；
// 由于每块板子的分组习惯以及命名习惯略有差异，所以，代码不一定完全适配，但是思路是一样的，按照
// 同样的思路更改即可。
#include "gpio.h"

int main() {
    Gpio gpio("PX.01", 463);
    gpio.init();
    gpio.set_direction(Gpio::Mode::OUTPUT);
    gpio.set_value(Gpio::Value::LOW);
    return 0;
}
```