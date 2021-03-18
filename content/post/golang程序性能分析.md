---
title: "Golang程序性能分析"
date: 2021-03-18T18:31:32+08:00
lastmod: 2021-03-18T18:31:32+08:00
draft: false
tags: ["golang", "profile"]
categories: ["language"]
mathjax: false
---

本文主要介绍如何使用golang自带的工具剖析程序性能瓶颈。  
<!--more-->

# 声明
主要代码来自[Dave Cheney](https://dave.cheney.net)，原博文[^1]对应的代码在[这里](https://github.com/davecheney/high-performance-go-workshop/tree/master/examples/words)。  

# 示例代码
本文为了展示方便，对部分代码作了一点调整，统一放在[这里](https://github.com/edward852/language/tree/master/go/profiling/words)。  

# 前置准备
- 安装新版本go  
- 安装chrome  
- 安装graphviz  

# benchmark
用`testing`包测试程序的benchmark，比较简单不多说。  
```go
func BenchmarkFib20(b *testing.B) {
	for n := 0; n < b.N; n++ {
		Fib(20) // run the Fib function b.N times
	}
}
```
执行benchmark测试：  
```shell
go test -bench=.
```

# pprof
## cpu
使用`profile.CPUProfile`收集cpu使用情况：  
```go
import "github.com/pkg/profile"

func main() {
    defer profile.Start(profile.CPUProfile, profile.ProfilePath(".")).Stop()
    
    // ...
}
```
编译、运行后会在当前目录下生成cpu.pprof，用自带的pprof工具查看：  
```shell
go build

# 统计moby.txt有多少单词
./words ../books/moby.txt

go tool pprof -http=:8080 cpu.pprof
```
可以看到`syscall read`占据了绝大部分时间：  
<p align="center">
    <img src="/images/syscall_read.png" width="200" />
</p>

查看源码就知道，readbyte每次只读1个字节，每次都触发`syscall read`。  
比较自然的优化方案就是每次syscall多读一些并且缓存起来，使用`bufio.NewReader`即可实现。  
```go
b := bufio.NewReader(f)
// ...
for {
    r, err := readbyte(b)
    // ...
}
```
改完再继续分析，发现修改有效，大部分时间耗费在了`mallocgc`。  
<p align="center">
    <img src="/images/mallocgc.png" width="250" />
</p>
这个表明时间耗费在了创建新对象时的内存分配，我们继续使用pprof分析内存(见下一小节)。

## memory
使用profile.MemProfile收集内存使用情况：
```go
import "github.com/pkg/profile"

func main() {
    defer profile.Start(profile.MemProfile, profile.ProfilePath(".")).Stop()
    
    // ...
}
```
编译、运行后会在当前目录下生成mem.pprof，继续使用pprof工具查看：
```shell
go build

# 统计moby.txt有多少单词
./words ../books/moby.txt

go tool pprof -http=:8080 mem.pprof
```
很明显`readbyte`又是罪灰祸首：  
<p align="center">
    <img src="/images/readbyte_heap_escape.png" width="200" />
</p>

这里涉及到golang的逃逸分析，感兴趣的看[这里](/post/golang逃逸分析与优化)。  
```
./main.go:15:6: buf escapes to heap:
./main.go:15:6:   flow: {heap} = &buf:
./main.go:15:6:     from buf (address-of) at ./main.go:16:22
./main.go:15:6:     from buf[:] (slice) at ./main.go:16:22
./main.go:15:6:     from r.Read(buf[:]) (call parameter) at ./main.go:16:18
./main.go:14:15: parameter r leaks to {heap} with derefs=0:
./main.go:14:15:   flow: {heap} = r:
./main.go:14:15:     from r.Read(buf[:]) (call parameter) at ./main.go:16:18
./main.go:14:15: leaking param: r
./main.go:15:6: moved to heap: buf
```
简单来说就是接口方法调用引用了`buf`变量，导致`buf`改为在堆上分配内存。  
这里不讨论为什么不能分配在栈上，集中注意力在如何优化上面。  
既然分配耗费时间，是否可以少分配操作？  
其实readbyte使用的`buf`分配一次即可，后续可以复用。  
```go
type bytereader struct {
	buf [1]byte
	r   io.Reader
}

func (b *bytereader) next() (rune, error) {
	_, err := b.r.Read(b.buf[:])
	return rune(b.buf[0]), err
}

// 分配一次
br := bytereader{
	r: bufio.NewReader(f),
}

for {
    r, err := br.next()
    // ...
}
```
修改完之后通过`time`测量发现，程序已经跟`wc -w`速度差不多了:tada:。  

# trace
统计单个文件已经满足不了`words`了，火速加上统计指定文件夹内txt文件单词数的功能。  
速度一般般，cpu和内存剖析看了也符合预期。不过可以更快一点吗？  
这个时候就轮到大杀器trace上场了。  
```go
import "github.com/pkg/profile"

func main() {
    defer profile.Start(profile.TraceProfile, profile.ProfilePath(".")).Stop()
    
    // ...
}
```
编译、运行后会在当前目录下生成trace.out，这次使用trace工具查看：

```go
go build

./words ../books

go tool trace -http=:8080 trace.out
```

不同于pprof基于采样，trace集成在runtime中，提供毫秒级别的时序执行记录。  
trace记录的信息更多(对程序性能影响也更大)，基本上每个时间点程序发生了什么都能准确知道。  
<p align="center">
    <img src="/images/trace_single_co.png" width="600" />
</p>

从图中可以看到，程序一直在忙着执行代码，但是只有一个协程在忙！  
竟然忘记了golang最有力的武器goroutine，赶紧加上~ `go countWords`~  

# http pprof && trace
在开发多协程版`words`同时，另外一名同事已经上线了`words`服务(天知道谁会使用~)。  
同样的，`words`也有性能问题，但是应该怎么分析？  
对于线上运行的服务，我们可以通过`net/http/pprof`包来pprof和trace程序。  
```go
import _ "net/http/pprof"

func startTrace(w http.ResponseWriter, r *http.Request) {
	f, err := os.Create("trace.out")
	if err != nil {
		fmt.Fprintln(w, "Failed to create trace file")
		return
	}

	err = trace.Start(f)
	if err != nil {
		fmt.Fprintln(w, "Failed to start trace:", err)
		return
	}

	fmt.Fprintln(w, "Trace started")
}

func stopTrace(w http.ResponseWriter, r *http.Request) {
	trace.Stop()
	fmt.Fprintln(w, "Trace stopped")
}

func serveDebug(stop <-chan struct{}) error {
	mux := http.DefaultServeMux

	// 方法一：
	// http://127.0.0.1:8081/start
	// http://127.0.0.1:8081/stop
	mux.HandleFunc("/start", startTrace)
	mux.HandleFunc("/stop", stopTrace)
	// 方法二：
	// curl -o trace.out http://127.0.0.1:8081/debug/pprof/trace?seconds=5

	return serve("127.0.0.1:8081", mux, stop)
}
```
import完、启动监听服务之后就可以了：  
## 方法一
手动访问`127.0.0.1:8081/start`和`127.0.0.1:8081/stop`开始和停止trace，生成`trace.out`文件。  

## 方法二
```shell
curl -o trace.out http://127.0.0.1:8081/debug/pprof/trace?seconds=10
```

拿到`trace.out`之后就可以按照之前的步骤分析了。  

# 后记
其实官方的Profiling Go Programs[^2]也是不错的，不过示例代码比较复杂，注意力更多集中在了代码逻辑。  

# 参考资料

[^1]: [GopherCon 2019 Dave Cheney](https://dave.cheney.net/high-performance-go-workshop/gophercon-2019.html)  
[^2]: [Profiling Go Programs](https://blog.golang.org/pprof)  

