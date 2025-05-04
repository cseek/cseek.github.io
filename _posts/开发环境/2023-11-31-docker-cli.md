---
title: ubuntu 安装 docker-cli
category: [开发环境]
tags: [docker-cli]
---

> docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 linux 系统上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
{: .prompt-info }

## 安装步骤

```bash
# 更新你的包索引：
sudo apt update
# 安装必要的依赖：
sudo apt install lsb-release apt-transport-https ca-certificates curl software-properties-common
# 添加Docker的官方GPG密钥：
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
# 添加Docker仓库：
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# 再次更新包索引：
sudo apt-get update
# 安装Docker CE：
sudo apt-get install docker-ce
# 查看Docker服务状态：
sudo systemctl status docker
# 设置开机自启动
sudo systemctl enable docker
```
## 卸载步骤

```bash
sudo apt-get purge docker-ce
```

## 使用国内源
+ 添加配置:
`sudo vim /etc/docker/daemon.json`
```bash
{ "registry-mirrors": ["https://docker.1ms.run"] }
```
+ 重启服务
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```