---
title: ubuntu 安装 docker-cli
category: [开发环境]
tags: [docker-cli]
---

> docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 linux 系统上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
{: .prompt-info }

## 安装步骤

```bash
# 更新存储库
sudo apt update && sudo apt upgrade
# 安装依赖
sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common -y
# 导入 docker GPG 密钥
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# 添加官方源
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 更新
sudo apt update
# 安装
sudo apt install docker-ce

```
## 卸载步骤

```bash
sudo apt-get purge docker-ce
```