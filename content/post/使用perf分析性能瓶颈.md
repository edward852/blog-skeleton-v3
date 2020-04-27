---
title: "使用perf分析性能瓶颈"
date: 2020-04-26T20:07:34+08:00
lastmod: 2020-04-26T20:07:34+08:00
draft: false
tags: ["profiling", "perf", "performance", "bottleneck"]
categories: ["tool"]
mathjax: false
---

本文主要记录如何使用perf分析性能瓶颈。  
<!--more-->

# 安装
## centos
```sh
yum install -y perf
```

## ubuntu
```sh
apt install -y linux-tools-generic
apt install -y linux-tools-`uname -r` || ln -fs /usr/lib/linux-tools/*-generic/perf /usr/bin/perf
```

## 火焰图支持
```sh
wget https://github.com/brendangregg/FlameGraph/archive/v1.0.tar.gz -O FlameGraph.gz

tar -xf FlameGraph.gz
cp FlameGraph-1.0/{flamegraph.pl,stackcollapse-perf.pl} /usr/bin/
rm -fr FlameGraph*
```

# 设置
有些配置项需要设置，否则非root用户无法达到最佳perf效果。  
## 临时
```sh
sysctl -w kernel.kptr_restrict=0
sysctl -w kernel.perf_event_paranoid=-1
```

## 永久
修改 `/etc/sysctl.conf` ，设置相应值。  
```sh
kernel.kptr_restrict=0
kernel.perf_event_paranoid=-1
```

# 使用
## 命令行
通过 `perf record` 可以搜集指定监控指标的数据，然后通过 `perf report` 查看结果。  
其中 `-e` 指定监控指标，通过 `perf list` 查看完整列表。  
`-g` 或者 `--call-graph` 记录调用关系。  
```sh
# 默认通过fp(frame pointer)获取调用关系，最好带-fno-omit-frame-pointer选项重编程序
perf record -a -e cpu-clock -g ./profiling ../moby.txt

# 通过dwarf获取调用关系(推荐)
perf record -a -e cpu-clock --call-graph dwarf ./profiling ../moby.txt

# 查看结果
perf report 或者 perf report --stdio
```

## 火焰图
`perf report` 的结果可能不是那么直观，可以生成火焰图查看。  
```sh
# 对perf record搜集的数据再加工处理
perf script > perf.unfold
stackcollapse-perf.pl perf.unfold > perf.folded
flamegraph.pl perf.folded > perf.svg
```
生成的 `perf.svg` 用浏览器打开查看即可，越宽代表耗时越长，性能瓶颈的可能性越大。  
![](https://i.loli.net/2020/04/27/WwjQT2uPYFNHtry.gif)  

# 示例代码
示例代码在 [这里](https://github.com/edward852/os/tree/master/cpu/profiling) ，可以实际操作体验。  
