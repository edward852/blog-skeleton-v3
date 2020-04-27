---
title: "CPU使用率高问题定位"
date: 2019-04-28T16:52:32+08:00
lastmod: 2020-04-27T16:52:32+08:00
draft: false
tags: ["cpu", "profile"]
categories: ["tool"]
mathjax: false
---

本文介绍如果利用工具定位CPU使用率高的问题。  
<!--more-->

# 总体思路
总体思路是逐步定位到引起CPU使用率高的进程、线程及其调用栈。  
先使用 top 或者 htop 定位问题进程。  
再通过 `top -H -p 问题进程号` 打开线程模式(或者打开top或htop后按H)定位问题线程。  
最后使用(语言)相关的工具分析该线程的调用栈。  

# Java
可本地运行复现的，使用 [VisualVM](https://visualvm.github.io/) 分析比较方便。  
不方便本地复现的服务进程，可以使用 `jstack`(需要转换为16进制线程号) 或者 [jvmtop](https://github.com/patric-r/jvmtop) 分析。  
```sh
jvmtop.sh --profile <PID>
```

# cpp
## perf
```sh
# 实时查看
perf top -p <pid>

# 离线查看
perf record -a -g -p <pid>
perf report -g 
```
`-a` 是采样所有CPU，`-g` 是采样调用关系。  
具体可以参考 [perf使用](/post/使用perf分析性能瓶颈/) 的说明。  

## VTune
如果CPU是Intel的，那么还可以使用 [VTune](https://software.intel.com/en-us/vtune/choose-download#standalone) 分析。  
有时候perf看不出来的调用，VTune也能够继续深入查看，进而定位到问题所在。  

# go
## pprof
`go tool pprof`

## trace
`go tool trace`

具体使用可以参考 [High Performance Go Workshop](https://dave.cheney.net/high-performance-go-workshop/gopherchina-2019.html) 。  
