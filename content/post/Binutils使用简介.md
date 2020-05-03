---
title: "GNU Binutils使用简介"
date: 2017-10-03T14:32:58+08:00
lastmod: 2020-05-03T14:32:58+08:00
draft: false 
tags: ["binutils", "ld", "nm", "addr2line", "objdump", "readelf", "strings"]
categories: ["tool"]
mathjax: false
---

本文主要记录GNU Binutils的日常使用。  
<!--more-->

# objdump
```sh
objdump -dS 二进制程序 > dump.txt
```
可以反汇编二进制程序，结合错误信息定位到问题代码的具体位置。  

# addr2line
显示地址所在的函数和行号。  
动态库的话，先要查看动态库加载地址，把出错地址换算为相对地址后再调用addr2line。  
```sh
addr2line -e 二进制文件 -sfC 地址
```

# nm
显示二进制程序的符号信息。  
没有符号信息可能是没有带 `-g` 选项编译或者被下文提到的 `strip` 去除了。  
```sh
nm -Cn 二进制程序 | grep 指定符号
```

# readelf
显示elf格式文件的信息。  
```sh
# 文件头信息
readelf -h 二进制程序

# 段信息 .bss .data .text 等
readelf -S 二进制程序

# 符号信息
readelf -s 二进制程序 | grep 指定符号
```

# strip
去掉二进制程序中的符号表。  
可以减小二进制程序的大小，但是同时增加了调试的难度(出问题后要带 `-g` 选项重编对应版本)。  

# ld
链接器，编译的时候就会自动调用。嵌入式开发偶尔会手动调用。  
日常使用主要是指定链接选项。  
通过 `-Wl` 可以指定逗号分隔的链接选项。  
```sh
-Wl,-rpath,/usr/local/gcc-7.2.0/lib64
```

# strings
输出二进制程序(库)中可打印的字符串。  
比如说查看当前标准c++库兼容的版本：  
```sh
strings /usr/lib/x86_64-linux-gnu/libstdc++.so.6 | grep ^GLIBCXX
```
