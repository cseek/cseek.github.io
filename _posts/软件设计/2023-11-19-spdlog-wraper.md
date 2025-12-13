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
```c++
// logger.h
#ifndef XLOGGER_H
#define XLOGGER_H

#include "spdlog/async.h"
#include "spdlog/fmt/bin_to_hex.h"
#include "spdlog/sinks/basic_file_sink.h"
#include "spdlog/sinks/rotating_file_sink.h"
#include "spdlog/sinks/stdout_color_sinks.h"
#include "spdlog/spdlog.h"
#include <memory>
#include <string>
#include <functional>

#define XLOGT(fmt, ...) Xlogger::instance().log_trace(fmt, ##__VA_ARGS__)
#define XLOGD(fmt, ...) Xlogger::instance().log_debug(fmt, ##__VA_ARGS__)
#define XLOGI(fmt, ...) Xlogger::instance().log_info(fmt, ##__VA_ARGS__)
#define XLOGW(fmt, ...) Xlogger::instance().log_warn(fmt, ##__VA_ARGS__)
#define XLOGE(fmt, ...) Xlogger::instance().log_error(fmt, ##__VA_ARGS__)
#define XLOGC(fmt, ...) Xlogger::instance().log_critical(fmt, ##__VA_ARGS__)

class Xlogger
{
private:
    Xlogger(const Xlogger &) = delete;
    Xlogger &operator=(const Xlogger &) = delete;
    Xlogger() = default;
    ~Xlogger()
    {
        if (m_logger)
        {
            m_logger->flush();
        }
    }

public:
    using Callback = std::function<void(const std::string &log)>;
    static Xlogger &instance() {
        static Xlogger logger;
        return logger;
    }

    bool init(const std::string &name,                                           // 日志器名称
              const std::string &file = "./log/app.log",                         // 日志文件名
              const std::string &pattern = "[%Y-%m-%d %H:%M:%S.%f] [%^%L%$] %v", // 日志样式
              size_t rotation = 4,                                               // 日志文件满4个时开始滚动日志
              size_t file_size = 1024 * 1024 * 6,                                // 单个日志文件大小为6MB
              spdlog::level::level_enum level = spdlog::level::debug,            // 日志级别
              spdlog::level::level_enum flush_on = spdlog::level::warn)          // 当打印这个级别日志时flush
    {
        m_level = level;
        if (m_logger)
        {
            return true;
        }
        auto &&function = [&]()
        {
            // 创建sinks
            auto stdout_sink = std::make_shared<stdout_sink_t>();
            auto rotating_sink = std::make_shared<rotating_sink_t>(file, file_size, rotation);
            std::vector<spdlog::sink_ptr> sinks{stdout_sink, rotating_sink};
            // 创建线程池和异步日志器
            m_thread_pool = std::make_shared<spdlog::details::thread_pool>(8192, 1);
            m_logger = std::make_shared<spdlog::async_logger>(name, sinks.begin(), sinks.end(), m_thread_pool, spdlog::async_overflow_policy::block);
            // 设置日志属性
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

    void set_callback(const Callback &callback)
    {
        m_callback = callback;
    }

    inline void flush()
    {
        if (m_logger)
        {
            m_logger->flush();
        }
    }

    template <typename... Args>
    inline void log_critical(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->critical(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::critical))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

    template <typename... Args>
    inline void log_error(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->error(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::err))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

    template <typename... Args>
    inline void log_warn(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->warn(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::warn))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

    template <typename... Args>
    inline void log_info(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->info(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::info))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

    template <typename... Args>
    inline void log_debug(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->debug(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::debug))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

    template <typename... Args>
    inline void log_trace(const char *fmt, Args... args)
    {
        if (m_logger)
        {
            m_logger->trace(fmt, args...);
        }
        if (m_callback && (m_level <= spdlog::level::trace))
        {
            m_callback(spdlog::fmt_lib::format(fmt, args...));
        }
    }

private:
    using stdout_sink_t = spdlog::sinks::stdout_color_sink_mt;
    using rotating_sink_t = spdlog::sinks::rotating_file_sink_mt;
    std::shared_ptr<spdlog::logger> m_logger;
    std::shared_ptr<spdlog::details::thread_pool> m_thread_pool;
    spdlog::level::level_enum m_level;
    Callback m_callback = nullptr;
};

#endif // XLOGGER_H
```

## 使用例子

```c++
#include "logger.h"

int main() {
    // 初始化日志器
    Xlogger::instance().init("app", 
                             "./log/app.log", 
                             "[%Y-%m-%d %H:%M:%S.%f] [%^%L%$] %v", 
                             4, 
                             1024 * 1024 * 6, 
                             spdlog::level::debug, 
                             spdlog::level::warn);
    
    // 设置回调函数处理日志
    Xlogger::instance().set_callback([](const std::string &log) {
        // 可以在这里处理日志，比如上传到服务器
    });
    
    // 记录日志
    XLOGI("Application started");
    XLOGW("This is a warning");
    XLOGE("This is an error");
    
    // 刷新日志缓冲
    Xlogger::instance().flush();
    
    return 0;
} 
```

## 核心特性

| 特性 | 说明 |
|------|------|
| 单例模式 | 全局唯一实例 |
| 异步日志 | 线程池处理，不阻塞主线程 |
| 双输出 | 同时输出到控制台和文件 |
| 日志轮转 | 自动管理文件大小和数量 |
| 回调机制 | 支持自定义日志处理 |

## 参数说明

| 参数 | 说明 |
|------|------|
| name | 日志器名称 |
| file | 日志文件路径 |
| pattern | 输出格式 |
| rotation | 文件轮转数 |
| file_size | 单个文件大小限制 |
| level | 最低日志级别 |
| flush_on | 触发立即刷新的日志级别 |

## 注意事项

- **线程安全**：`_mt` 后缀表示线程安全的 sink
- **文件权限**：确保日志目录可写
- **性能**：异步处理不阻塞主线程
- **缓冲**：程序退出前调用 `flush()` 确保日志写入
