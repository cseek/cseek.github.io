---
title: cmake 手动安装
category: [开发环境]
tags: [cmake]
---

> 安装步骤如下。
{: .prompt-info }

## 下载

```bash
wget https://cmake.org/files/v3.31/cmake-3.31.6.zip
unzip cmake-3.31.6.zip
```
## 编译

```bash
cd cmake-3.31.6
./configure
make -j20
```
## 卸载旧版本

```bash
sudo apt remove cmake
```

## 安装

```bash
sudo make install
```