---
title: 使用 rust 访问串口
category: [软件设计]
tags: [串口, serial, serialport]

# image:
#     path: /assets/img/headers/rust-serial.webp
---

> 在嵌入式开发中经常使用到串口，这里演示一下如何快速地使用 rust 来编写串口的访问代码，我们使用 [serialport](https://crates.io/crates/serialport) crate 来实现, 一起来感受一下 rust 惊人的开发效率吧！！
{: .prompt-info }

## 环境搭建
把 RX 和 TX短接，接入电脑

## 创建工程

```bash
cargo new serial_demo
```

## 添加依赖

在 Cargo.toml 文件里指定 serialport 的版本

```toml
[dependencies]
serialport = "4.3.0"
```

## 编写代码

```rust
use std::{thread::sleep, time::Duration};

use serialport::SerialPort;

fn main() -> serialport::Result<()> {
    // 列出所有可用的串行端口
    let ports = serialport::available_ports()?;
    for p in ports {
        println!("找到串行端口: {:?}", p);
    }

    // 实例化一个串口
    let mut port = serialport::new("/dev/ttyUSB0", 115_200)
    .timeout(Duration::from_millis(3000))
    .open().expect("打开串口失败");

    let state: &[u8; 15] = &[0x11, 0x22, 0x33, 0x44, 0x55, 0x66];
    port.write(state).expect("串口写失败");
    print_buf(&read_ack(& mut port, 6));

    Ok(())
}

fn read_ack(port: &mut Box<dyn SerialPort>, len: usize) -> Vec<u8> {
    let mut read_buf: Vec<u8> = vec![0; len];
    port.read(read_buf.as_mut_slice()).expect("没有数据");
    read_buf
}

fn print_buf(buf: &Vec<u8>) {
    for byte in buf {
        print!("{:02X} ", byte);
    }
    println!();
}

```
## 编译运行
```bash
cargo build
cargo run
```


