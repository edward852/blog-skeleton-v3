---
title: "golang context包"
date: 2018-08-05T15:14:19+08:00
lastmod: 2018-08-05T15:14:19+08:00
draft: true
tags: ["context", "golang"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang context包的使用。  
<!--more-->

[context包](https://golang.org/pkg/context) 可以手动或者超时取消进行中的协程，传递上下文数据。  
统一使用context包可以实现全链路的超时控制，以及确保信息的安全传递。  

# 取消
```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
```
取消主要分为手动取消和超时取消。  
手动取消一般是出错的时候主动调用cancel。  
超时取消是未能在指定的时间期限内完成而被取消。  
cancel即使不主动调用，也不影响资源的最终释放，但主动调用可加快释放，避免等待时间内的资源浪费。  

# 上下文数据传递
```go
func WithValue(parent Context, key interface{}, val interface{}) Context
```
context中的上下文数据并不是全局的，它只查询本节点以及父节点们的数据，不能查询兄弟节点的数据。  

# 参考链接
- https://golang.org/pkg/context
- https://blog.golang.org/context
- https://studygolang.com/articles/12566
