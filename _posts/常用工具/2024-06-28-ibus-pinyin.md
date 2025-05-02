---
title: ubuntu 24.04 安装中文输入法
category: [常用工具]
tags: [输入法]
---

> 之前用过搜狗拼音、百度输入法、谷歌输入法，说实话，除了谷歌安装方便点，另外两个真不建议用，对系统侵入性太大，不过今天我们的主角并不是上面几个，而是智能拼音，安装简单，不改变原有的 ibus，使用也方便。
{: .prompt-info }

## 安装软件包

```bash
sudo apt update
sudo apt install ibus
sudo apt install ibus-pinyin
sudo apt install ibus-libpinyin
```

## 安装语言包
1. 点击`System`->`Manage Installed Languages`;
![](/assets/img/post/常用工具/intell-pinyin1.webp)
2. 点击 `Install/Remove Languages`;
![](/assets/img/post/常用工具/intell-pinyin2.webp)
3. 点击 `Chinese(simplified)`->`Apply`;
![](/assets/img/post/常用工具/intell-pinyin3.webp)
4. 出现`汉语（中国）`即可。
![](/assets/img/post/常用工具/intell-pinyin4.webp)

## 配置
1. 点击 `Keyboard`->`Add Input Source`->`Chinese`->`Chinese Intelligent Pinyin`->`Add`;
![](/assets/img/post/常用工具/intell-pinyin5.webp)
2. 按下 `Shift` 即可切换中英文输入法。