---
title: cmake 导入闭源库
category: [软件设计]
tags: [cmake, IMPORTED]
---

> 在 cmake 中, IMPORTED 目标是指那些在项目外部预先构建好的库或可执行文件。通过将这些库或可执行文件作为导入的目标引入，可以在项目中使用它们，就像使用项目内构建的目标一样。这样做的好处是可以方便地重用已有的二进制文件，而无需从源代码重新构建它们，这对于依赖于第三方库的项目尤其有用。
{: .prompt-tip }

## 静态库

目录结构如下

```bash
myapp
├── other_lib
│   ├── include
│   │   └── other_lib.h
│   ├── lib
│   │   └── libother_lib.a
│   └── CMakeLists.txt
├── CMakeLists.txt
└── main.cpp
```

其中 your_lib 下的 CMakeLists.txt 内容如下

```bash
project(other_lib)
add_library(${PROJECT_NAME} STATIC IMPORTED GLOBAL)
add_library(deps::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
set_target_properties(
    ${PROJECT_NAME} PROPERTIES
    IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/libother_lib.a
    INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/include
)
```

顶层目录的 CMakeLists.txt 内容如下

```bash
cmake_minimum_required(VERSION 3.10)
project(myapp)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE deps::other_lib)
add_subdirectory(other_lib)
```

## 动态库
动态库导入方式和静态库的基本一致，但不建议使用 IMPORTED_LOCATION 导入动态库，因为如果导入的动态库本身不存在 SONAME，运行的时候可能会出问题，建议使用 INTERFACE, 参考下面这个：

```bash
project(other_lib)
add_library(${PROJECT_NAME} INTERFACE)
add_library(deps::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories(
    ${PROJECT_NAME} INTERFACE
    {CMAKE_CURRENT_SOURCE_DIR}/include
)
target_link_libraries(
    ${PROJECT_NAME} INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}/lib/libother_lib.so
)
```



