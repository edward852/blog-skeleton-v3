---
title: "CPU使用率高问题定位"
date: 2019-04-28T16:52:32+08:00
lastmod: 2019-04-28T16:52:32+08:00
draft: false
tags: ["cpu", "profile"]
categories: ["tool"]
mathjax: false
---

本文介绍如果利用工具定位CPU使用率高的问题。  
<!--more-->

# Java
Java工程可以使用 [VisualVM](https://visualvm.github.io/) 启动并分析。  
另外还可以使用 [jvmtop](https://github.com/patric-r/jvmtop) 分析指定的服务进程。  
```sh
jvmtop.sh --profile <PID>
```

# cpp
## perf
编译时加上-fno-omit-frame-pointer选项，进程需要带有符号表(不能strip)。  
```sh
# 实时查看
perf top -p <pid>

# 离线查看
perf record -a -g -p <pid>
perf report -g 
```
`-a` 是采样所有CPU，`-g` 是采样调用关系。  

## VTune
如果CPU是Intel的，那么还可以使用 [VTune](https://software.intel.com/en-us/vtune/choose-download#standalone) 分析。  
有时候perf看不出来的调用，VTune也能够继续深入查看，进而定位到问题所在。  

# go
## pprof
`go tool pprof`

## trace
`go tool trace`

具体使用可以参考 [High Performance Go Workshop](https://dave.cheney.net/high-performance-go-workshop/gopherchina-2019.html) 。  
