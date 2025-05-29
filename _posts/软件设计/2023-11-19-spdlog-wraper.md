---
title: spdlog 包装器
category: [软件设计]
tags: [spdlog, log]
---

> spdlog 是一个快速、可扩展的 c++ 日志库，它提供了简单易用的接口和灵活的配置选项。spdlog 支持多种日志级别、多线程安全，可以将日志输出到终端、文件或者其他自定义的目标。它具有高性能和低开销的特点，适用于各种规模的应用程序和系统，有很多知名的项目都用到了 spdlog，比如 fastDDS,下面记录一下我在工作中是怎么用的。
{: .prompt-info }

## 获取源码

如果要将源码添加进你的工程里，请从 github 获取

```bash
https://github.com/gabime/spdlog
```
+ 如果使用的是 conan 包管理，可以从这里获取

```bash
https://conan.io/center/recipes/spdlog
```

## 包装器

singleton.h
```c++
// 单例模板
#ifndef __SINGLETON_H__
#define __SINGLETON_H__

#include <iostream>
#include <string>

template<class T>
class Singleton {
protected:
    Singleton(const Singleton &) = delete;
    Singleton &operator=(const Singleton &) = delete;
    Singleton() = default;
    ~Singleton() = default;

public:
    template<typename... Args>
    static T &instance(Args &&...args) {
        static T obj(std::forward<Args>(args)...);
        return obj;
    }
};
#endif // __SINGLETON_H__

```c++
// spdlog.h

#ifndef __LOGGER_H__
#define __LOGGER_H__

#include "singleton.h"
#include "spdlog/async.h"
#include "spdlog/fmt/bin_to_hex.h"
#include "spdlog/sinks/basic_file_sink.h"
#include "spdlog/sinks/rotating_file_sink.h"
#include "spdlog/sinks/stdout_color_sinks.h"
#include "spdlog/spdlog.h"
#include <memory>

/* 将数组转成 16 进制，然后再打印输出，
 * 例如: LOGI("{:X}", TO_HEX(data, len));*/
#define TO_HEX(data, len) spdlog::to_hex(data, data + len)
#define LOGE(...) Singleton<Logger>::instance().log_error(__VA_ARGS__)
#define LOGW(...) Singleton<Logger>::instance().log_warn(__VA_ARGS__)
#define LOGI(...) Singleton<Logger>::instance().log_info(__VA_ARGS__)
#define LOGD(...) Singleton<Logger>::instance().log_debug(__VA_ARGS__)
#define LOGC(...) Singleton<Logger>::instance().log_critical(__VA_ARGS__)

class Logger
{
private:
    std::unique_ptr<spdlog::logger> m_logger; // 日志器

public:
    Logger(const Logger &) = delete;
    Logger &operator=(const Logger &) = delete;
    Logger() = default;
    ~Logger()
    {
        if (m_logger)
        {
            m_logger->flush();
        }
    }

    bool init(const std::string &name,                                           // 日志器名称
              const std::string &file = "./log/spdlog.log",                      // 日志文件名
              const std::string &pattern = "[%Y-%m-%d %H:%M:%S.%f] [%^%L%$] %v", // 日志样式
              size_t rotation = 4,                                               // 日志文件满4个时开始滚动日志
              size_t file_size = 1024 * 1024 * 6,                                // 单个日志文件大小为6MB
              spdlog::level::level_enum level = spdlog::level::debug,            // 日志级别
              spdlog::level::level_enum flush_on = spdlog::level::warn)          // 当打印这个级别日志时flush
    {
        if (m_logger)
        {
            return true;
        }
        auto &&function = [&]()
        {
            auto stdout_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
            auto rotating_sink = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(file, file_size, rotation);
            std::vector<spdlog::sink_ptr> sinks{stdout_sink, rotating_sink};
            m_logger = std::make_unique<spdlog::logger>(name, sinks.begin(), sinks.end());
            m_logger->set_pattern(pattern);
            m_logger->set_level(level);
            m_logger->flush_on(flush_on);
        };
        try
        {
            function();
            return true;
        }
        catch (const std::exception &e)
        {
            SPDLOG_ERROR("Construct logger error: {}", e.what());
            return false;
        }
    }

    inline void flush()
    {
        if (m_logger)
        {
            m_logger->flush();
        }
    }

    template <typename... Args>
    inline void log_error(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->error(fmt, args...);
        }
        else
        {
            SPDLOG_ERROR(fmt, args...);
        }
    }

    template <typename... Args>
    inline void log_warn(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->warn(fmt, args...);
        }
        else
        {
            SPDLOG_WARN(fmt, args...);
        }
    }

    template <typename... Args>
    inline void log_info(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->info(fmt, args...);
        }
        else
        {
            SPDLOG_INFO(fmt, args...);
        }
    }

    template <typename... Args>
    inline void log_debug(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->debug(fmt, args...);
        }
        else
        {
            SPDLOG_DEBUG(fmt, args...);
        }
    }

    template <typename... Args>
    inline void log_critical(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->critical(fmt, args...);
        }
        else
        {
            SPDLOG_CRITICAL(fmt, args...);
        }
    }
};
#endif // __LOGGER_H__

```

## 使用例子

```c++
#include "logger.h"

int main() {
    // 初始化两个日志
    Singleton<Logger>::instance().init("./log/app.log", "[%Y-%m-%d %H:%M:%S.%f] [%^%L%$] %v", 4, 1024 * 1024 * 6, spdlog::level::debug, spdlog::level::warn);
    LOGI("hello");
    return 0;
} 
```
