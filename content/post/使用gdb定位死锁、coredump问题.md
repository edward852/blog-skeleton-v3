---
title: "使用gdb定位死锁、coredump"
date: 2017-11-28T22:32:34+08:00
lastmod: 2020-04-28T22:32:34+08:00
draft: false
tags: ["gdb", "deadlock", "coredump"]
categories: ["tool"]
mathjax: false
---

本文主要记录如何通过gdb定位死锁、coredump问题。  
<!--more-->

# 安装
通过软件包管理器(apt, yum等)安装gdb即可，不赘述。  

# 死锁
死锁发生后我们需要知道触发死锁的线程及其调用栈。  
gdb命令 `thread apply all bt` 结合 `-p` 选项可以打印出指定进程所有线程的调用栈。  
```sh
gdb --batch -ex "thread apply all bt" -p 问题进程号
```

# coredump
进程收到SEGV信号(一般是访问非法地址)时，如果开启coredump的话就会生成core文件。  
通过 `ulimit` 可以开启coredump功能。  
```sh
# 确保coredump启用
ulimit -c unlimited
```
core文件一般在程序同级目录，没有的话可以查看 `/proc/sys/kernel/core_pattern` 。  
gdb打开core文件之后，就可以调用 `thread apply all bt` 查看程序挂掉时所有线程的调用栈。  
```sh
gdb 二进制程序路径 core文件路径

# gdb command
thread apply all bt
```
另外注册SEGV信号处理函数，调用ucontext.h的接口进行栈回溯也是可以的。一般结合自动拉起使用。  

# 示例代码
示例代码在 [这里](https://github.com/edward852/os/tree/master/process_thread/deadlock) ，可以实际操作体验。  
