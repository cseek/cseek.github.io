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
```

spdlog_wraper.h
```c++
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
#include <iostream>

#define LOG_TAG "myapp"                              // 日志tag
#define LOG_FILE_NAME "./log/" LOG_TAG ".log"        // 日志文件名
#define LOG_FILE_SIZE 1024 * 1024 * 3                // 单个日志文件大小为3MB
#define LOG_ROTATION 10                              // 日志文件满10个时开始滚动日志
#define LOG_FLUSH_ON spdlog::level::warn             // 当打印这个级别日志时flush
#define PATTERN "[%Y-%m-%d %H:%M:%S.%f] [%^%L%$] %v" // 日志样式

// 将数组转成 16 进制，然后再打印输出，
// 例如: LOGI("{:X}", TO_HEX(data, len));
#define TO_HEX(data, len) spdlog::to_hex(data, data + len)
#define LOG_FLUSH(...) Singleton<Logger>::instance().flush()
#define LOGE(...) Singleton<Logger>::instance().log_error(__VA_ARGS__)
#define LOGW(...) Singleton<Logger>::instance().log_warn(__VA_ARGS__)
#define LOGI(...) Singleton<Logger>::instance().log_info(__VA_ARGS__)
#define LOGD(...) Singleton<Logger>::instance().log_debug(__VA_ARGS__)
#define LOGC(...) Singleton<Logger>::instance().log_critical(__VA_ARGS__)

class Logger {
private:
    std::unique_ptr<spdlog::logger> m_logger;

private:
    bool is_debug_mode() {
        char *var = getenv("LOG_DEBUG");
        if (nullptr == var) {
            return false;
        }
        if (0 == strcmp(var, "on")) {
            return true;
        }
        return false;
    }

public:
    Logger() {
        auto &&function = [&](){
            auto stdout_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
            auto rotating_sink = std::make_shared<spdlog::sinks::rotating_file_sink_mt>(
                LOG_FILE_NAME,
                LOG_FILE_SIZE,
                LOG_ROTATION
            );
            std::vector<spdlog::sink_ptr> sinks{
                stdout_sink,
                rotating_sink
            };
            m_logger = std::make_unique<spdlog::logger>(LOG_TAG, sinks.begin(), sinks.end());
            m_logger->set_pattern(PATTERN);
            if (is_debug_mode()) {
                m_logger->set_level(spdlog::level::debug);
            } else {
                m_logger->set_level(spdlog::level::info);
            }
            m_logger->flush_on(LOG_FLUSH_ON);
        };
        try {
            function();
        } catch (const std::exception &e) {
            std::cout << "Construct logger error: " << e.what() << std::endl;
        }
    }

    void flush() {
        m_logger->flush();
    }

    template <typename... Args>
    inline void log_error(const char *fmt, Args... args) {
        m_logger->error(fmt, args...);
    }

    template <typename... Args>
    inline void log_warn(const char *fmt, Args... args) {
        m_logger->warn(fmt, args...);
    }

    template <typename... Args>
    inline void log_info(const char *fmt, Args... args) {
        m_logger->info(fmt, args...);
    }

    template <typename... Args>
    inline void log_debug(const char *fmt, Args... args) {
        m_logger->debug(fmt, args...);
    }

    template <typename... Args>
    inline void log_critical(const char *fmt, Args... args) {
        m_logger->critical(fmt, args...);
    }
};

#endif // __LOGGER_H__
```
