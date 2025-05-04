---
title: Ubuntu 24.04 永久修改 DNS
category: [开发环境]
tags: [dns, resolv, nameserver]
---

> 在 Ubuntu 24.04 中，直接修改 /etc/resolv.conf 可能因系统服务（如 systemd-resolved 或 NetworkManager）自动覆盖而失效。以下是分步解决方案。
{: .prompt-info }

## 编辑配置文件
取消注释并设置 DNS 服务器：
`sudo vim /etc/systemd/resolved.conf`

```bash
[Resolve]
DNS=8.8.8.8 1.1.1.1
```

## 更新符号链接（确保指向非临时文件）

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

## 重启服务

```bash
sudo systemctl restart systemd-resolved
```

## 验证 
```bash
# 查看是否可以解析域名
nslookup baidu.com
```