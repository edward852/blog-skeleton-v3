---
title: "Golang限时执行外部命令"
date: 2018-05-06T20:46:12+08:00
lastmod: 2018-05-06T20:46:12+08:00
draft: false
tags: ["exec", "golang"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang如何限时执行外部命令。  
<!--more-->

golang可以通过 [os/exec](https://golang.org/pkg/os/exec) 执行外部命令。  
不过 `exec` 并不是调用系统的shell实现的，效果上更像C的`exec`系列函数。  
当然，直接调用shell去执行外部命令也是支持的，但是风险比较大，需要特别注意过滤恶意输入。  

另外我们一般希望外部命令的执行时间在合适范围内，防止资源的浪费。  
因此下文主要讲限时执行外部命令。  

# 直接执行命令
```go
ctx, cancel := context.WithTimeout(context.Background(), 1*time.Minute)
defer cancel()

url := `http://q.url.cn/ABH0Oi`
cmdStr := `curl -Ls -w %{url_effective} -o /dev/null -- ` + url
cmdList := strings.Split(cmdStr, " ")
out, err := exec.CommandContext(ctx, cmdList[0], cmdList[1:]...).Output()
if err != nil {
    log.Fatal(err)
} else {
    log.Println(string(out[:]))
}
```
直接调用 `exec.CommandContext` 并传入限时上下文和命令(这里是`curl`)即可实现限时执行外部命令。  
另外 `--` 可以分隔命令的选项参数和位置参数(避免注入)。  

# 通过shell执行命令
调用shell执行，适合命令比较复杂的情况，包括重定向、管道、涉及通配符等情况。  
需要特别注意检查命令输入参数的合法性，防止注入。  
```go
out, err := exec.CommandContext(ctx, "sh", "-c", cmdStr).Output()
```
