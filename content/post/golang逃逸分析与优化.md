---
title: "Golang逃逸分析与优化"
date: 2020-06-20T15:41:24+08:00
lastmod: 2021-02-20T15:41:24+08:00
draft: false
tags: ["golang", "escape"]
categories: ["language"]
mathjax: false
---

本文主要介绍如何分析Golang的逃逸以及优化方向。  
<!--more-->

# 逃逸
逃逸简单来说就是Golang编译器经过分析决定变量分配到堆上而不是栈上。  

# 基本规则
自己总结的规则，不一定完全正确，仅供参考：  

- 占用内存空间过大，不适合分配到栈上  
  比如说在我的机器上面分配10MB以上的数组就会改为堆上分配。  
- 变量引用生命周期长，变量不适合分配到栈上  
  比如说变量被堆上的数据引用了、返回入参或函数内新定义变量的引用。  
  情况特别多，包括全局变量、闭包、接口方法调用、指针类型的`slice`,`map`,`chan`等  

其实Golang本身还在发展优化中，所以会出现不应该逃逸但实际逃逸的情况。  
当这种情况成为性能瓶颈的时候，我们需要去分析并优化。  

# 分析

## go build
```sh
go build -gcflags '-m=1 -l'
```
`-l`是禁止内联，`-m`是打印优化决定，后面指定的数字越大越详细。  
重点关注 `escapes to heap` 和 `moved to heap` 附近的打印。  

## go compile
```sh
go tool compile -S xxx.go
# 可以使用grep过滤
```
编译成汇编查看变量的内存分配情况。  

## 例子
可以参考 [Golang escape analysis](http://www.agardner.me/golang/garbage/collection/gc/escape/analysis/2015/10/18/go-escape-analysis.html) 和 [golang 逃逸分析详解](https://zhuanlan.zhihu.com/p/91559562) 上面的例子。  

下面模拟了一下 `fmt.Println` 逃逸的情况。  
```go
package main

import (
	"sync"
)

type pp struct {
	arg interface{}
}

var ppFree = sync.Pool{
	New: func() interface{} { return new(pp) },
}

func (p *pp) printInt(i interface{}) {
	p.arg = i

	if n, ok := i.(int); ok {
		println(n)
	}
}

func printInt(si ...interface{}) {
	for _, i := range si {
		if n, ok := i.(int); ok {
			println(n)
		}
	}
}

func main() {
	n1 := 0
	n2 := 1

	printInt(n1)

	p := ppFree.Get().(*pp)
	p.printInt(n2)
}
```
`fmt.Println(n2)` 导致`n2`逃逸的情况跟 `p.printInt(n2)` 比较类似。  
调用路径是 `Println -> Fprintf -> doPrintf -> printArg`。  
`new(pp)` 毫无疑问逃逸了到堆上，那么 `p.arg` 也就在堆上，`p.arg = i` 导致`i`引用的值`n2`也逃逸。  

# 优化
可以参考 [Allocation efficiency in high-performance Go services](https://segment.com/blog/allocation-efficiency-in-high-performance-go-services/) 的说明，干货很多。  
附上原文的结论：  
- 不要过早优化，以性能剖析工具的数据指导优化  
- 栈上分配是廉价、快速的，堆上分配是昂贵、耗时的  
- 理解逃逸分析的规则有助于编写更高效的代码  
- 指针使得栈上分配不可行(少用指针多传值)  
- 代码中性能至关重要的地方考虑使用能够控制内存分配的接口  
- 热点代码减少接口类型的使用(接口方法调用)  
