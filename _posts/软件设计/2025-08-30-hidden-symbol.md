---
title: 记录一个 zip_open 函数签名冲突的问题
category: [软件设计]
tags: [export, hidden, symbol, zip]
---

> 在设计 c++ sdk 时，我们常常会隐藏大部分接口，只保留要暴露给客户的那部分接口，但是如果 我们的 sdk 依赖了第三方库，第三库编译时如果不带 hidden 属性，第三方库的函数签名就会被导出，如果客户刚好有同名的函数，就有可能会链接到我们引入的第三方库，从而出现一些未知的问题。下面记录一下我是如何发现这个问题并解决的。
{: .prompt-info }

## 问题发现
在下面的伪代码中，客户在 func 函数中调用了 sdk_api，客户的另一个 sdk 里调用了 zip_open 函数，且客户把调用 func 函数的地方注释掉了的，但是程序运行后就崩溃了，奇怪的是，把 func 里的 sdk_api 函数注释掉，客户的程序就正常运行。所以猜测是编译的时候，客户调用的zip_open 链接到了我们的 sdk 里的某个库上了。

```c++

void func() {
    sdk_api(); // sdk 里的 open 函数
}

int main() {
    // func();
    zip_open(); // 另一个库里的 open 函数
    return 0;
}

```

## 问题解决

我使用 `readelf -s libmysdk.so | grep zip_open` 发现我们的库里却是有 zip_open 这么个函数，且 zip_open 函数是 GLOBAL + DEFAULT, 也就是全局导出的，所以在编译第三方库的时候，我们给 gcc 或者 g++ 加入了 hidden 属性就解决了。如下：

```bash
target_compile_options(
    ${PROJECT_NAME} PRIVATE
    $<$<CONFIG:DEBUG>: -g -fvisibility=hidden>
    $<$<CONFIG:RELEASE>: -DNDEBUG -fvisibility=hidden>
)

```
