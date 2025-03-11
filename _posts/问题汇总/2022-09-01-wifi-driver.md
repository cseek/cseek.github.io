---
title: linux 编译、安装 wifi 驱动
category: [问题汇总]
tags: [rtl8852b]
---

> 最近买了一台联想的 thinkbook14 pro 笔记本，装了 ubuntu22.04 x系统后，发现没有 wifi 驱动，几经周折，终于解决了，下面就把这个方法分享给大家。

## 现象

开机进入系统后，右上角的设置条目里没有 wifi 选项

## 解决办法

### 1、** 在 BIOS 里关闭 Security Boot**

### 2、** 安装 rtl8852be 驱动**

如果内核版本小于 5.18

```bash
git clone https://github.com/HRex39/rtl8852be.git
cd rtl8852be
make -j8
sudo make install
sudo modprobe 8852be
```

如果内核版本大于 5.18

```bash
git clone https://github.com/HRex39/rtl8852be.git -b dev
cd rtl8852be
make -j8
sudo make install
sudo modprobe 8852be
```
