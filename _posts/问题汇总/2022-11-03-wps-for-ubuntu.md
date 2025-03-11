---
title: ubuntu 下 wps 字体缺失解决办法
category: [问题汇总]
tags: [wps]
---

> 在 ubuntu 系统上安装 wps 后，启动报错 `some formula symbols might be not display`，原因是 wps 字体缺失，解决办法如下：
{: .prompt-info }

```bash
git clone https://github.com/IamDH4/ttf-wps-fonts.git
cd ttf-wps-fonts
sudo ./install.sh
```