---
title: "Golang错误处理"
date: 2020-09-05T16:04:21+08:00
lastmod: 2021-03-05T16:04:21+08:00
draft: false
tags: ["golang", "error"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang如何处理错误。  
<!--more-->

# 处理方法
## 错误码或字符串
一般是对外接口采用的方法，特别是需要序列化(如JSON)的情况。  

## errors package
使用 [errors](https://pkg.go.dev/github.com/pkg/errors) 包来处理错误：  

- New, Errorf, Wrap, and Wrapf  
  生成或封装错误。  
- Cause  
  提取封装的错误，方便后续根据具体错误类型进行处理。  
- `%+v` 或 StackTrace  
  打印产生错误时的栈回溯信息。  

## 辅助类型
具体参考《Errors are values[^2]》和《Eliminate error handling by eliminating errors[^3]》。  
- 错误是实现`error`接口的值  
- 错误是可以编程的值  

合适的编程可以消除冗长的`if err != nil`判断。  
```go
type errWriter struct {
    w   io.Writer
    err error
}

func (ew *errWriter) write(buf []byte) {
    if ew.err != nil {
        return
    }
    
    _, ew.err = ew.w.Write(buf)
}

func writeData() error {
    // ...
    ew := &errWriter{w: fd}
    
    ew.write(p0[a:b])
    ew.write(p1[c:d])
    ew.write(p2[e:f])

    // check error in the end
    if ew.err != nil {
        return ew.err
    }
}
```

## panic+defer+recover
具体参考《The Way to Go[^1]》的处理方式，也是一种消除冗长`if err != nil`判断的方法。  
可以实现类似于异常处理的效果。  
```go
package main

import (
	"fmt"

	"github.com/pkg/errors"
)

func check(err error) {
	if nil != err {
		panic(err)
	}
}

func outer() (err error) {
	defer func() {
		if rerr, ok := recover().(error); ok {
			err = errors.Wrap(rerr, "outer")
		}
	}()

	err = inner()
	check(err)

	return nil
}

func inner() error {
	return errors.New("inner error")
}

func main() {
	err := outer()
	fmt.Printf("%v\n\n", err)
	fmt.Printf("%+v\n", errors.Cause(err))
}
```

## Go 2 check+handle
Go 2预计支持新的错误处理[^4]方式，感兴趣的可以先了解一下。  

# 参考文档
- [Error handling and Go](https://blog.golang.org/error-handling-and-go)  
- [Don’t just check errors, handle them gracefully](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)  


[^1]: [一种用闭包处理错误的模式](https://github.com/unknwon/the-way-to-go_ZH_CN/blob/master/eBook/13.5.md)  
[^2]: [Errors are values](https://blog.golang.org/errors-are-values)  
[^3]: [Eliminate error handling by eliminating errors](https://dave.cheney.net/2019/01/27/eliminate-error-handling-by-eliminating-errors)  
[^4]: [Go 2 Error Handling](https://go.googlesource.com/proposal/+/master/design/go2draft-error-handling-overview.md)  
