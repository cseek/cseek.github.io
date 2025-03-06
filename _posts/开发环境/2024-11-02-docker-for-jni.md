---
title: jni 无图形界面最小编译环境搭建
tags: [jni, ndk, android]
category: [开发环境]
# image:
#   path: /assets/img/headers/batcat.webp
---

> 在 Jni 开发过程中，可以采用 Docker 搭建编译环境来提升开发效率，减少因环境搭建而浪费时间。
{: .prompt-info }

## 构建 docker 镜像
### 构建脚本

```Dockerfile
# 基础镜像
FROM ubuntu:focal
# 设置环境变量，避免交互式安装时的提示
ENV DEBIAN_FRONTEND=noninteractive
# 工作空间
WORKDIR /workspace
RUN apt update
# 防止添加源后证书相关的问题
RUN yes | apt install ca-certificates
# 添加国内源
RUN echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal main restricted universe multiverse" > /etc/apt/sources.list
RUN echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-security main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-updates main restricted universe multiverse" >> /etc/apt/sources.list
RUN echo "deb https://mirrors.ustc.edu.cn/ubuntu/ focal-backports main restricted universe multiverse" >> /etc/apt/sources.list
RUN apt update
# 安装编译环境
RUN yes | apt install build-essential
# 安装java环境
RUN yes | apt install openjdk-17-jdk
# 安装unzip
RUN yes | apt install unzip
ENV HOME=/root
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH=$PATH:$JAVA_HOME/bin
# 安装命令工具
RUN yes | apt install wget
RUN cd /workspace && wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip
RUN mkdir ~/.android-sdk -p
RUN unzip -d ~/.android-sdk /workspace/commandlinetools-linux-11076708_latest.zip
RUN mv ~/.android-sdk/cmdline-tools ~/.android-sdk/latest
RUN mkdir ~/.android-sdk/cmdline-tools
RUN mv ~/.android-sdk/latest ~/.android-sdk/cmdline-tools
ENV ANDROID_HOME=$HOME/.android-sdk
ENV PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
# 安装android sdk
RUN yes | sdkmanager "platform-tools" "platforms;android-33"
ENV PATH=$PATH:$ANDROID_HOME/platform-tools
# 安装android ndk
RUN yes | sdkmanager --install "ndk;25.1.8937393" --channel=3
ENV ANDROID_NDK_HOME=$ANDROID_HOME/ndk
ENV PATH=$PATH:$ANDROID_NDK_HOME/25.1.8937393
RUN yes | apt remove unzip
RUN yes | apt remove wget
RUN rm -rf /var/lib/apt/lists/*
RUN rm /workspace/commandlinetools-linux-11076708_latest.zip
# 设置默认的命令（可选）
CMD ["/bin/bash"]
```
### 构建命令

```bash
sudo docker build -t jni_buildenv .
```

## 运行 docker 容器
```bash
#!/bin/bash
CURRENT_DIR=$(pwd)
MOUNT_TO_DIR="/workspace"
sudo docker run -it -v ${CURRENT_DIR}:${MOUNT_TO_DIR} jni_buildenv /bin/bash
```