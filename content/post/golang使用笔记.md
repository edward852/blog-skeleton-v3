---
title: "golang使用笔记"
date: 2019-07-21T11:13:04+08:00
lastmod: 2019-07-21T11:13:04+08:00
draft: false
tags: ["golang"]
categories: ["language"]
mathjax: false
---

本文主要记录golang使用的一些笔记。  
<!--more-->

# 安装
参考 [官网安装说明](https://golang.org/doc/install) 操作即可。  

## Linux
```sh
tar -C /usr/local -xf go*.tar.gz
```
然后把 `/usr/local/go/bin` 加入到 `PATH`，可以在 `~/.bash_profile` 文件加入以下内容：  
```sh
export PATH=$PATH:/usr/local/go/bin
```
重新登录一下或者重启之后生效。  

# IDE
推荐使用 [VSCode](https://code.visualstudio.com/Download) 或者 [GoLand](https://www.jetbrains.com/go/download/) 。  
测试代码片段使用 [playground](https://play.golang.org) 也挺不错。  

# 教程
可以参考以下教程，本文主要记录一些没讲到的或者需要注意的点。  
另外可能需要科学上网才能访问。  
- [A Tour of Go](https://tour.golang.org)
- [Effective Go](https://golang.org/doc/effective_go.html) 

# module
新版go已经支持module，可以通过 `go mod` 管理依赖。  
```sh
go mod init 项目名

# 新增依赖包之后执行
go mod tidy
```

# doc
通过 `go doc` 命令可以查看package相关文档。  
比如说查看内置函数make的使用说明：
```sh
go doc builtin make
```

# import
```go
// 引入单个包
import "fmt"

// 引入多个包
import (
    "net/http"
    "time"
    m "math"    // 以别名m引入math包
)

```

# 导出规则
只有首字母大写的`symbol`才会被导出，包括package的函数、变量以及各种协议的编解码等。  

# 入口
package main的main函数为程序入口。  
每个package的init函数会自动调用。  

# 变量声明
函数内才可以使用 `:=` 自动推导声明，否则需要完整的声明。  

# array
array是静态数组。  
```go
intArr1 := [3]int{1,2,3}

// 使用...可以由编译器确定数组大小
intArr2 := [...]int{1,2,3}
```

# 变参
具体参考 [这里](https://golang.org/ref/spec#Passing_arguments_to_..._parameters) 的说明。  
```go
func Greeting(prefix string, who ...string) {
	fmt.Printf(prefix)
	for _, v := range who {
		fmt.Printf(" %s", v)
	}
	fmt.Println()
}

func main() {
	s := []string{"James", "Jasmine"}

	Greeting("nobody")                          // 变参为 nil
	Greeting("hello:", "Joe", "Anna", "Eileen") // 变参为 []string{"Joe", "Anna", "Eileen"}
	Greeting("goodbye:", s...)                  // 变参就是s本身(因为类型一致)
}
```

# 闭包
支持闭包，即函数持有外层上下文(变量)的引用。  
```go
func sentenceFactory(str string) func(before, after string) string {
    // 返回的函数持有str变量
    return func(before, after string) string {
        return fmt.Sprintf("%s %s %s", before, str, after)
    }
}
```

# 流程控制
## if
可以有for类似的初始化表达式，定义的变量在if else代码块中有效。  
多行条件，运算符要放在最后。  

## switch
分支默认break，如果不需要break，要么case写在一起，要么使用 `fallthrough` 关键字。  

## defer
defer后进先出，参数会先求值。  
defer在函数退出时调用，而不是离开代码块时，因此for循环应避免使用defer。

# 接口与反射
参考 [golang反射](/post/golang反射的使用/) 的说明。  
另外接口是可以组合的。  

# 指针
指针解引用也是使用 `.` 运算符(而不是`->`)。  

# make, new, &Struct{k:v}
make主要用于创建map, slice, chan。  
new相当于malloc+赋零值(zero value)。  
make和new都是在堆上创建对象。  
&Struct{k:v}用于初始化结构体，默认从栈上分配，但有可能逃逸到堆(比如说调用`fmt.Println`)。

# JSON
参考 [JSON操作](/post/golang操作json/) 的说明。  

# protobuf
参考 [protobuf操作](/post/golang操作protobuf/) 的说明。  

# cgo
cgo申请的内存需要显式释放，因为gc不处理。  
```go
cs := C.CString(s)
defer C.free(unsafe.Pointer(cs))
```
cgo尽量只调用编解码接口，不调用(外部库中)可能导致阻塞的收发包接口。  

# 执行外部命令
参考 [执行外部命令](/post/golang执行外部命令/) 的说明。  

# context
参考 [context包](/post/golang_context包/) 的说明。  
