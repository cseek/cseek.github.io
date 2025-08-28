---
title: 解决 ubuntu24.04 git gui 显示字体大小无法设置的问题
category: [问题汇总]
tags: [git-gui, git]
---

> 使用 git gui 有六七个年头了，最近在有的 ubuntu24.04 物理机上发现一个很诡异的现象， 运行 git gui，字体会很大，且无法通过设置里修改，几经摸索，总结了下面这个可行的方法，所以记录一下。

## 现象
忘了截图了，窗口字体很大，是正常字体的四五倍这样。

## 解决办法

1. 打开配置文件，添加如下内容

`vim ~/.gitconfig`

```bash
[gui]
        fontdiff = -family \"DejaVu Sans\" -size 10 -weight normal -slant roman -underline 0 -overstrike 0
        fontui = -family \"DejaVu Sans\" -size 10 -weight normal -slant roman -underline 0 -overstrike 0
[user]
        email = jassimxiong@gmail.com
        name = aurson
[http]
        postBuffer = 524288000
```
2. 重启 git gui 即可。
