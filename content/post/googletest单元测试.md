---
title: "Google Test单元测试"
date: 2020-04-21T21:45:56+08:00
lastmod: 2020-04-21T21:45:56+08:00
draft: false
tags: ["unit", "test"]
categories: ["tool"]
mathjax: false
---

本文主要记录如何使用GTest(google test)对cmake工程进行单元测试。  
<!--more-->

# 安装依赖
## cmake
安装3.10以上版本cmake，一般到 [官网](https://cmake.org/download) 下载安装最新版本即可。  
```sh
# ubuntu 18.04
sudo apt install -y cmake
```

## GTest
下载、安装 [GTest最新版本](https://github.com/google/googletest/releases) 。  
```sh
# ubuntu 18.04
sudo apt install -y libgtest-dev

# 编译、安装GTest
# 进入GTest源码目录
cd /usr/src/gtest
mkdir build && cd build
cmake ..
make
sudo make install
```

# 撰写CMakeLists.txt

## 主目录
引入测试目录(test)的CMakeLists.txt文件。  
```cmake
add_subdirectory(test)
```

## 测试目录
CMakeLists.txt文件内容如下：  
```cmake
cmake_minimum_required(VERSION 3.10)

enable_testing()
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

find_package(Threads REQUIRED)

aux_source_directory(. TEST_FILES)
add_executable(tests ${TEST_FILES})

# binary_search替换为项目生成的库目标名
target_link_libraries(tests binary_search ${GTEST_BOTH_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
gtest_discover_tests(tests)
```

# 完整示例
示例代码在 [这里](https://github.com/edward852/algorithm/tree/master/binary_search) 。  
```sh
# 进入示例代码所在目录
mkdir build && cd build
cmake ..
make

# 主程序
./main
# 测试程序
./test/tests
```
