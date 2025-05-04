---
title: 快速实现一个流媒体服务器
category: [软件设计]
tags: [node-media-server, nodejs]
---

> 之前在公司主要是使用 Mediamtx 来实现流媒体服务器，今天分享一个使用 Nodejs 实现的流媒体服务器---Node-media-server。
{: .prompt-info }

## 安装 nodejs
```bash
sudo apt install nodejs
```

## 安装 npm 包管理器
```bash
sudo apt install npm
```

## 添加服务器代码
+ 创建工程
```bash
mkdir mediaserver
cd mediaserver
vim app.js
```
+ 添加代码
```js
const NodeMediaServer = require('node-media-server');
const config = {
  rtmp: {
    port: 1935,
    chunk_size: 60000,
    gop_cache: true,
    ping: 30,
    ping_timeout: 60
  },
  http: {
    port: 8000,
    allow_origin: '*'
  }
};

var nms = new NodeMediaServer(config)
nms.run();
```

## 安装依赖
```bash
npm i node-media-server
```

## 运行
```bash
node app.js
```

## 推流到流服务器
下面的命令经过 `node-media-server` 后会生成 rtmp(可以在电脑上播放) 和 flv(可以在手机和电脑上播放) 流。
+ rtmp 地址为:`rtmp://127.0.0.1:1935/live/my_stream_name`
+ flv 地址为:`http://127.0.0.1:8000/live/my_stream_name.flv`

```bash
ffmpeg -re -i ./test.mp4 -vcodec copy -acodec copy -f flv rtmp://127.0.0.1:1935/live/my_stream_name
# 或者
ffmpeg -re -i ./video.mp4 -c copy -f flv rtmp://127.0.0.1:1935/live/my_stream_name
```