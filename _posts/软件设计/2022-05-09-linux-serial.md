---
title: 使用 c++ 实现一个串口类
category: [软件设计]
tags: [serial, uart]
---

> 记录一下在项目中我常用到的串口操作代码。采用非阻塞 + select 监听事件代替轮询或阻塞的方式。
{: .prompt-info }

## 串口代码

serial.h

```c++
#ifndef __SERIAL_H__
#define __SERIAL_H__

#include <fcntl.h>
#include <sys/select.h>
#include <termios.h>
#include <unistd.h>

#include <cstdint>
#include <iostream>
#include <memory>
#include <string>

class Serial {
public:
    Serial(
        const std::string &dev_node,
        uint32_t baud_rate = 115200,
        uint8_t data_bits = 8,
        uint8_t stop_bits = 1,
        char parity = 'N') // N:无校验, O:奇校验, E:偶校验
        : _dev_node(dev_node),
          _baud_rate(baud_rate),
          _data_bits(data_bits),
          _stop_bits(stop_bits),
          _parity(parity) {
        std::cout << "Device node :" << _dev_node << "\n";
        std::cout << "BaudRate    :" << _baud_rate << "\n";
        std::cout << "DataBits    :" << _data_bits << "\n";
        std::cout << "StopBits    :" << stop_bits << "\n";
        std::cout << "Parity      :" << _parity << "\n";
        open();
    }

    ~Serial() {
        close();
    }

    int32_t open() {
        /* 防止重复 open */
        if (_fd > 0) {
            return 0;
        }
        /* 非阻塞 */
        _fd = ::open(_dev_node.c_str(), O_RDWR | O_NOCTTY | O_NDELAY);
        if (_fd < 0) {
            std::cout << "Failed to open node: " << _dev_node << "\n";
            return -1;
        }
        /* 清空输入输出缓冲区 */
        tcflush(_fd, TCIOFLUSH);

        return config();
    }

    int32_t close() {
        if (_fd > 0) {
            ::close(_fd);
            _fd = -1;
        }

        return 0;
    }

    int32_t write(uint8_t *data, uint32_t data_len) {
        int ret = 0;
        int sent = 0; // 已经发送的字节数
        while (data_len - sent > 0) {
            ret = ::write(_fd, data + sent, (size_t)(data_len - sent));
            if (ret < 0) {
                if (errno == 11) {
                    continue;
                } else {
                    break;
                }
            }
            sent += ret;
        }
        tcdrain(_fd); // 保证数据都发送完成

        return sent;
    }

    int32_t read(uint8_t *buffer, uint32_t buffer_len, uint32_t timeout_ms = 0) {
        if (_fd < 0) {
            std::cout << "Not open device node: " << _dev_node << "\n";
            return -1;
        }

        int ret = 0;
        int ava = 0;
        struct timeval tv;
        tv.tv_sec = timeout_ms / 1000;           // seconds
        tv.tv_usec = (timeout_ms % 1000) * 1000; // microseconds

        while ((buffer_len - ava) > 0) {
            fd_set read_fds;
            FD_ZERO(&read_fds);
            FD_SET(_fd, &read_fds);
            ret = select(_fd + 1, &read_fds, nullptr, nullptr, &tv);
            if (ret < 0) {
                std::cout << "Errno=" << errno << "\n";
                // 错误码为11表示重试
                if (errno == 11) {
                    continue;
                } else {
                    ava = ret;
                    break;
                }
            } else if (ret == 0) {
                // 超时
                break;
            } else {
                ret = ::read(_fd, buffer + ava, buffer_len - ava);

                if (ret < 0) {
                    ava = ret;
                    break;
                }
                ava += ret;
            }
        }

        return ava;
    }

    int32_t reopen(uint32_t baud_rate) {
        close();
        _baud_rate = baud_rate;
        return open();
    }

private:
    int32_t config() {
        struct termios options;
        if (tcgetattr(_fd, &options) != 0) {
            std::cout << "Tcgetattr error\n";
            return -1;
        }
        options.c_cflag |= (CLOCAL | CREAD); //(本地连接（不改变端口所有者)|接收使能)
        options.c_cflag &= ~CSIZE;           // 屏蔽字符大小位
        options.c_cflag &= ~CRTSCTS;         // 不使用流控制

        switch (_data_bits) {
        case 7:
            options.c_cflag |= CS7;
            break;
        case 8:
            options.c_cflag |= CS8;
            break;
        default:
            std::cout << "Unsupported data size\n";
            return -1;
        }
        switch (_parity) {
        case 'O': // 奇校验
            options.c_cflag |= PARENB;
            options.c_cflag |= PARODD;
            options.c_iflag |= (INPCK | ISTRIP);
            break;
        case 'E': // 偶校验
            options.c_iflag |= (INPCK | ISTRIP);
            options.c_cflag |= PARENB;
            options.c_cflag &= ~PARODD;
            break;
        case 'N': // 无校验
            options.c_cflag &= ~PARENB;
            break;
        default:
            std::cout << "Unsupported parity\n";
            return -1;
        }
        if (_stop_bits == 1) {
            options.c_cflag &= ~CSTOPB;
        } else if (_stop_bits == 2) {
            options.c_cflag |= CSTOPB;
        } else {
            std::cout << "Unsupported stop bits\n";
            return -1;
        }
        options.c_iflag &= ~(ICRNL | IXON); //~(将CR映射到NL|启动出口硬件流控)
        options.c_oflag = 0;                // 输出模式标志
        options.c_lflag = 0;                // 本地模式标志
        options.c_cc[VMIN] = 1;             // 设置最小返回字节数

        speed_t speed = B115200;
        switch (_baud_rate) {
        case 115200:
            speed = B115200;
            break;

        case 230400:
            speed = B230400;
            break;

        case 460800:
            speed = B460800;
            break;

        case 921600:
            speed = B921600;
            break;

        default:
            speed = B115200;
            break;
        }

        cfsetispeed(&options, speed);      // 设置输入速度,波特率
        cfsetospeed(&options, speed);      // 设置输出速度,波特率
        tcsetattr(_fd, TCSANOW, &options); // 设置属性(termios结构)

        return 0;
    }

private:
    int32_t _fd;           // 文件描述符
    std::string _dev_node; // 设备节点
    uint32_t _baud_rate;   // 波特率
    uint8_t _data_bits;    // 数据位
    uint8_t _stop_bits;    // 停止位
    char _parity;          // 奇偶校验位
};

#endif // __SERIAL_H__
```

## 使用示例

略
