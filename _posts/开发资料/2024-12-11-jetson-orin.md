---
title: jetson orin 开发资料汇总
categories: [开发资料]
tag: [nvidia, jetson, gpu]
---

> nvidia 的官方文档没有汇总的清单，查阅起来真的很头大，下面是我平时开发过程中有用到的一些资料，做一个简单的汇总。
{: .prompt-info }

## 计算 GPIO 编号

[https://developer.nvidia.com/docs/drive/drive-os/6.0.8.1/public/drive-os-linux-sdk/common/topics/sys_components/CalculatingGPIOIndexinLinux32.html](https://developer.nvidia.com/docs/drive/drive-os/6.0.8.1/public/drive-os-linux-sdk/common/topics/sys_components/CalculatingGPIOIndexinLinux32.html)

## 开发指南之系统烧录 (l4t35.4.1)
[https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/FlashingSupport.html](https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/SD/FlashingSupport.html)

## 开发指南之快速入门 (l4t35.4.1)
[https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/IN/QuickStart.html](https://docs.nvidia.com/jetson/archives/r35.4.1/DeveloperGuide/text/IN/QuickStart.html)

## 系统固件下载
+ l4t35.3.1 (ubuntu 20.04)

[https://developer.nvidia.com/embedded/jetson-linux-r3531](https://developer.nvidia.com/embedded/jetson-linux-r3531)

+ l4t36.4.0 (ubuntu 22.04)

[https://developer.nvidia.com/embedded/jetson-linux-r3640](https://developer.nvidia.com/embedded/jetson-linux-r3640)

+ l4t36.4.3 (ubuntu 22.04)

[https://developer.nvidia.com/embedded/jetson-linux-r3643](https://developer.nvidia.com/embedded/jetson-linux-r3643)

## 官方资料下载中心

[https://developer.nvidia.com/embedded/downloads](https://developer.nvidia.com/embedded/downloads)

## Jetson Orin pinmux 引脚配置模板(设备树)

[https://developer.nvidia.com/downloads/jetson-orin-nx-and-orin-nano-series-pinmux-config-template](https://developer.nvidia.com/downloads/jetson-orin-nx-and-orin-nano-series-pinmux-config-template)

## 通过寄存器配置 GPIO

+ 搜索 pinmux

[https://docs.nvidia.com/jetson/archives/r36.4/DeveloperGuide/HR/JetsonModuleAdaptationAndBringUp/JetsonOrinNxNanoSeries.html?highlight=pinmux](https://docs.nvidia.com/jetson/archives/r36.4/DeveloperGuide/HR/JetsonModuleAdaptationAndBringUp/JetsonOrinNxNanoSeries.html?highlight=pinmux)

## 技术开发指南

+ 里面可以查到gpio的寄存器地址

[https://developer.nvidia.com/orin-series-soc-technical-reference-manual](https://developer.nvidia.com/orin-series-soc-technical-reference-manual)

## 版本发布历史
+ 包含了 jetpack 版本；
+ 包含了 l4t 版本对应的开发指引；
+ 包含了 l4t 版本对应的 API 文档。
[https://docs.nvidia.com/jetson/archives/](https://docs.nvidia.com/jetson/archives/)