---
title: c++ 内存泄漏排查
category: [常用技巧]
tags: [valgrind, ASan, AddressSanitizer]
---

> 内存泄漏是比较常见的问题，下面记录一下我在遇到内存泄漏时的排查思路以及如何降低内存泄漏的风险。
{: .prompt-info }

## 排查示例

假如有下面这样一个 c++ 程序:

```c++
// demo.cpp
#include <cstdlib>
int main() {
    char *p = (char *)malloc(123);
    return 0;
}
```
下面是两种排查的方法：
### 1、使用外部工具 valgrind 排查
```bash
# 安装 valgrind
sudo apt install valgrind
g++ demo.cpp -o demo -g
➜  ~ valgrind --leak-check=full ./demo
==4222== Memcheck, a memory error detector
==4222== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==4222== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==4222== Command: ./demo
==4222==
==4222==
==4222== HEAP SUMMARY:
==4222==     in use at exit: 123 bytes in 1 blocks
==4222==   total heap usage: 1 allocs, 0 frees, 123 bytes allocated
==4222==
==4222== 123 bytes in 1 blocks are definitely lost in loss record 1 of 1
==4222==    at 0x483B7F3: malloc (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==4222==    by 0x10915E: main (demo.cpp:3)
==4222==
==4222== LEAK SUMMARY:
==4222==    definitely lost: 123 bytes in 1 blocks
==4222==    indirectly lost: 0 bytes in 0 blocks
==4222==      possibly lost: 0 bytes in 0 blocks
==4222==    still reachable: 0 bytes in 0 blocks
==4222==         suppressed: 0 bytes in 0 blocks
==4222==
==4222== For lists of detected and suppressed errors, rerun with: -s
==4222== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```
可以看到 `==4222==    by 0x10915E: main (demo.cpp:3)` 中指明了 demo.cpp 第 3 行存在内存泄漏

### 2、使用 gcc 内部的 AddressSanitizer 排查
还是上面的 demo.cpp

```bash
➜  ~ g++ -fsanitize=address -g demo.cpp -o demo
➜  ~ ./demo

=================================================================
==4270==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 123 byte(s) in 1 object(s) allocated from:
    #0 0x7fb80137a808 in __interceptor_malloc ../../../../src/libsanitizer/asan/asan_malloc_linux.cc:144
    #1 0x56352cc0419e in main /home/cyber/demo.cpp:3
    #2 0x7fb800d53082 in __libc_start_main ../csu/libc-start.c:308

SUMMARY: AddressSanitizer: 123 byte(s) leaked in 1 allocation(s).
```
如上，检查出了内存泄漏的位置。

## 如何降低内存泄漏的风险？
可以使用下面几种方式：
### 1、尽量避免 RAW 指针的使用
### 2、使用 RAII 自动释放
如果非要使用智能指针，可以采用智能指针包装一层，利用 RAII 机制在生命周期结束的时候自动释放，参见我前面的这个贴子[如何防止裸指针内存泄漏](https://cseek.github.io/posts/raw-pointer/)。
### 3、new/delete 追踪分配
通过重载 new/delete 关键字，并且维护一个全局的变量来记录分配总数，这样就可以随时了解分配情况，还可以将变量数据导出到 csv 表格，再使用 python 来绘制图表，从而观察分配走势。
```c++
#include <iostream>

// 全局计数器跟踪内存分配
static size_t allocated = 0;

void* operator new(size_t size) {
    allocated += size;
    std::cout << "Allocated " << size << " bytes. Total: " << allocated << std::endl;
    return malloc(size);
}

void operator delete(void* ptr) noexcept {
    free(ptr);
    // 注意：无法追踪具体释放大小（简化示例）
}

int main() {
    int* p = new int(42);
    delete p; // 若注释此行，程序退出时会打印未释放的内存总量
    return 0;
}
```