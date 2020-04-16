---
title: "AddressSanitizer定位内存问题"
date: 2020-04-16T23:51:59+08:00
lastmod: 2020-04-16T23:51:59+08:00
draft: false
tags: ["memory", "address", "sanitizer"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何通过`AddressSanitizer`定位各类内存问题(溢出、泄露...)。  
<!--more-->

[AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) 主要支持定位以下内存问题：  
- stack buffer overflow
- heap buffer overflow
- global buffer overflow
- memory leak
- use after free
- ...

# 安装
安装4.8以上GCC或3.1以上LLVM即可。  

## GCC
尽量使用新版本，不然个别内存问题可能不支持定位。  
安装可以参考 [GCC7安装](/post/编译安装gcc7.2.0/)的说明。下文以GCC7.3为例进行说明。  

## libasan
安装对应版本libasan：  
```sh
# CentOS系统
# GCC 4.8安装libasan, 7安装libasan4, 9安装libasan5
yum install -y libasan4
```

## LLVM (可选) {#symbolizer}
显示符号和行号需要用到`llvm-symbolizer`，需要安装LLVM，可以参考 [LLVM安装](/post/编译clang+llvm/)。  
```sh
yum install -y llvm7.0

# 方法1 建立软链
ln -fs /usr/bin/llvm-symbolizer-7.0 /usr/bin/llvm-symbolizer

# 方法2 设置环境变量
export ASAN_SYMBOLIZER_PATH=`which /usr/bin/llvm-symbolizer-7.0`
```
如果不方便安装LLVM的话，也可以通过`addr2line`手动定位符号和行号：  
```sh
addr2line -e 二进制文件 -sfC 地址
```

# 使用
## 编译
根据情况添加编译和链接选项重新编译程序。  
编译选项： `-fsanitize=address -fno-omit-frame-pointer` 。  
链接选项：  
- 动态 `-lasan`
- 静态 `-static-libasan`

## 运行
一般直接运行即可定位各种内存问题，有问题可以设置ASAN_OPTIONS，具体参考 [Asan Flags](https://github.com/google/sanitizers/wiki/AddressSanitizerFlags) 。  
```sh
export ASAN_OPTIONS=symbolize=1:detect_leaks=1
```

# FAQ
- 为什么只显示地址，看不到符号和行号?  
  参考 [llvm-symbolizer](#symbolizer) 说明。  
- 运行报错 `ASan runtime does not come first in initial library list...`  
  + 方法一  
    加上 `LD_PRELOAD=libasan动态库路径` 运行程序。   
  + 方法二  
    程序静态链接libasan库(-static-libasan选项)后重新运行。  
- 检查不了内存泄露?  
  可能版本较低(建议升级新版本)或者尝试设置`detect_leaks=1`。  
  ```sh
  export ASAN_OPTIONS=symbolize=1:detect_leaks=1
  ```
