---
title: "golang使用笔记"
date: 2019-07-21T11:13:04+08:00
lastmod: 2020-03-17T11:13:04+08:00
draft: false
tags: ["golang"]
categories: ["language"]
mathjax: false
---

本文主要记录golang使用的一些笔记。  
<!--more-->

# 安装
参考官网 [安装说明](https://golang.org/doc/install) 操作即可。  

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
- [golang构建web应用](https://github.com/astaxie/build-web-application-with-golang) 
- [Go Data Structures](https://research.swtch.com/godata)

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
还可以安装godoc，支持离线模式查看文档(不用经常科学上网)：  
```sh
# 安装godoc
go get golang.org/x/tools/cmd/godoc

# localhost:8080提供文档服务
godoc -http=:8080
```

# import
```go
// 引入单个包
import "fmt"

// 引入多个包
import (
    "net/http"
    . "time"    // 引入time包，使用包中的函数、变量可以不加包名前缀
    m "math"    // 以别名m引入math包
    
    "database/sql"
    _ "github.com/ziutek/mymysql/godrv" // 只调用包的init函数
)
```
更详细的可以看 [Import declarations](https://golang.org/ref/spec#Import_declarations) 的说明。  

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
支持闭包，即持有外层上下文(变量)引用的函数。  
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

# slice
slice可以当作动态数组来使用。  
```go
// 第2参数指定实际大小，第3参数指定(预计)容量大小
a := make([]int, 5)
b := make([]int, 0, 5)

// 通过append新增
a = append(a, 2, 3, 4)
```
注意`append`不直接修改slice，而是返回新的slice，所以一般都要赋值回去。  

# struct
当struct内匿名嵌入一个struct字段，能够继承其所有变量和方法。  
嵌入单个匿名struct字段类似于单继承，嵌入多个匿名struct字段则类似于多继承。  
嵌入具名的struct字段则类似于组合。  
多态通过interface实现。  
公有、私有控制通过symbol名称(变量名、函数名、字段名等)首字母的大小写实现。  

# 接口与反射
参考 [golang反射](/post/golang反射的使用/) 的说明。  
另外接口是可以组合的。  
```go
// io.ReadWriter
type ReadWriter interface {
    Reader
    Writer
}
```

# 指针
指针调用方法或访问变量使用 `.` 运算符(而不是`->`)。  

# 并发
参考文档  [Share Memory By Communicating](https://blog.golang.org/share-memory-by-communicating) 以及 [代码示例](https://golang.org/doc/codewalk/sharemem/) 。还可以参考 [stackoverflow](https://stackoverflow.com/questions/36391421/explain-dont-communicate-by-sharing-memory-share-memory-by-communicating) 上面的解释。  

> Do not communicate by sharing memory; instead, share memory by communicating.

简单来说就是不要通过加锁共享内存的方式通信，而要通过通信让渡使用权(所有权)来共用内存。  

## goroutine

## channel

# make, new, &Struct{k:v}
make主要用于创建map, slice, chan。返回对应类型的对象，而不是指针。  
```go
func make(t Type, size ...IntegerType) Type
```
new相当于malloc+赋零值(zero value)，返回的是指针。  
```go
func new(Type) *Type
```
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
