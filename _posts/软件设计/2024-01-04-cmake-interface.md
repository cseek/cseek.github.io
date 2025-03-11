---
title: cmake 导入接口库
category: [软件设计]
tags: [cmake, INTERFACE]
---

> 在 cmake 中, INTERFACE 关键字用于定义接口库（Interface Libraries），这是一种不直接编译成二进制文件，而是用来传递使用要求（如编译选项、定义、包含目录等）给其他目标（如可执行文件或库）的特殊目标。
{: .prompt-tip }

## 例一

假设你有一个接口库，你想为链接到它的目标添加一些编译器定义：

```bash
cmake
add_library(my_interface_lib INTERFACE)

target_compile_definitions(my_interface_lib INTERFACE
    MY_DEFINE=1
)
```

然后，当其他目标链接到这个接口库时：

```bash
add_executable(my_exe main.cpp)
target_link_libraries(my_exe PRIVATE my_interface_lib)
```
my_exe 目标会自动接收到 my_interface_lib 中定义的编译器定义 MY_DEFINE=1 。 

## 例二

假如你使用的第三方库只有头文件，你就可以把它单独编译成接口库，然后链接到你的项目中，下面用 nlohmann json 举个例子，目录结构如下

```bash
├── nlohmann
│   ├── json.hpp
│   ├── json_fwd.hpp
│   └── CMakeLists.txt
├── CMakeLists.txt
└── main.cpp
```

```bash
project(nlohmann)

add_library(${PROJECT_NAME} INTERFACE)
target_include_directories(
    ${PROJECT_NAME} INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}
)
```

顶层目录的 CMakeLists.txt 内容如下

```bash
cmake_minimum_required(VERSION 3.10)
project(main)

add_executable(main main.cpp)
target_link_libraries(main nlohmann)
add_subdirectory(nlohmann)
```

