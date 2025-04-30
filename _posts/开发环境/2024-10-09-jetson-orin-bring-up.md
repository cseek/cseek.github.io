---
title: jetson orin 基础环境搭建
category: [开发环境]
tags: [jetson, orin, l4t, jetpack, nvidia]
---

> 本文主要介绍如何烧录 jetson orin 的 BSP 包和 RootFS，以及一些基础环境。
{: .prompt-info }

## 硬件说明

Nvidia Jetson Orin NX 系列模块面向 AI 智能系统的计算平台，具备 100 TOPS 浮点运算的 AI 处理能力，采用小巧的外形，具备优秀的散热能力，丰富的传感器接口和出色的性能，为所有嵌入式 AI 和边缘系统带来新功能。具有计算能力强、可靠性高、集成度高、功耗低特点，可应用于商用机器人，医疗仪器，智能相机，高分辨率传感器，自动光学检查，智能工厂和其他 AIoT 嵌入式系统。

![](/assets/img/post/开发环境/orin-NX.webp)

## 引脚功能

![](/assets/img/post/开发环境/orin-NX-pin.webp)

## 风扇说明（以图为科技的为例）

Jetson NX 套件自带风扇,该风扇默认是跟随温度的变化值自动开启,可以通过修改
/sys/devices/pwm-fan/target_pwm 进行风扇速度控制,数值范围是 0-255,该控制在完全断电在开机后会重置,所以仅对当次设置有效,另外可通过安装 jtop 对设备风扇进行控制.

## 注意事项

CSI 摄像头不支持热插拔，请上电之前操作；
电源范围： 9 - 19 V DC， 电流建议最大不超过 5A；
工作温度： -20℃ - 50℃

## 操作系统

+ BSP

BSP 包括 Linux 内核、UEFI 引导加载程序、NVIDIA 驱动程序。

+ RootFS
RootFS 是基于 Ubuntu 的示例文件系统。

## 下载 BSP 和 RootFS

以 l4t 35.3.1（ubuntu 20.04）为例：
+ 下载 BSP
[https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v3.1/release/jetson_linux_r35.3.1_aarch64.tbz2](https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v3.1/release/jetson_linux_r35.3.1_aarch64.tbz2)

+ 下载 RootFS
[https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v3.1/release/tegra_linux_sample-root-filesystem_r35.3.1_aarch64.tbz2](https://developer.nvidia.com/downloads/embedded/l4t/r35_release_v3.1/release/tegra_linux_sample-root-filesystem_r35.3.1_aarch64.tbz2)
## 烧录

1. 准备一台带有 `ubuntu20.04` 的 PC;
2. 使用 USB 数据线连接 Jetson orin NX 设备和 PC；
3. 解压 BSP 和 RootFS;
```bash
$ tar xf Jetson_Linux_R35.3.1_aarch64.tbz2
$ sudo tar xf Tegra_Linux_Sample-Root-Filesystem_R35.3.1_aarch64.tbz2 -C Linux_for_Tegra/rootfs/
```
4. 开启烧录模式并接通电源（如果有 rec 按键，则长按 rec 键后再接通电源，如果没有此按键，则使用母头杜邦线短接 rec 引脚到 GND 再接通电源）；
5. 运行烧录前的两个脚本；
```bash
$ cd Linux_for_Tegra/
$ sudo ./tools/l4t_flash_prerequisites.sh
$ sudo ./apply_binaries.sh
```
6. 运行烧录命令，并等待 5 秒，松开 REC 按键，或者拔掉杜邦线；
```bash
  sudo ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 -c tools/kernel_flash/flash_l4t_external.xml -p "-c bootloader/t186ref/cfg/flash_t234_qspi.xml" --showlogs --network usb0 jetson-orin-nano-devkit internal
```
7. 等待烧录成功后，开机配置用户名、密码即可。

## 安装基础软件包

1. 安装 jetpack;
```bash
sudo apt update
sudo apt upgrade
sudo apt install nvidia-jetpack
```
2. 安装性能监测工具;
```bash
sudo apt install python3-pip
sudo -H pip3 install -U jetson-stats
sudo jtop
sudo jetson_release   #查看当前的设备的信息
```

## 配置 IO

```bash
sudo /opt/nvidia/jetson-io/jetson-io.py
```