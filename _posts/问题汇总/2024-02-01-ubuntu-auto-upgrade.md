---
title: 禁止 ubuntu 自动更新的方法
category: [问题汇总]
tags: [upgrade, update]
---

> 直接上干货。
{: .prompt-info }

## 步骤

+ 1、 `sudo vi /etc/apt/apt.conf.d/10periodic`
后面部分全部改成 “0”

```bash
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Download-Upgradeable-Packages "0";
APT::Periodic::AutocleanInterval "0";
```

+ 2、`sudo vi /etc/apt/apt.conf.d/20auto-upgrades`
后面部分全部改成 “0”

```bash
APT::Periodic::Update-Package-Lists "0";
APT::Periodic::Unattended-Upgrade "0";
```

+ 3、重启