---
title: "Golang时间转换"
date: 2018-04-14T16:56:48+08:00
lastmod: 2018-04-14T16:56:48+08:00
draft: false
tags: ["golang", "time"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang如何转换时间以及需要注意的地方。  
<!--more-->

golang中 `Mon Jan 2 15:04:05 -0700 MST 2006` 是一个特殊的时间，时间的格式化和解析都用到了。  
这个时间特殊之处在于不同时间字段对应的字符串(数字)是不同的，因此可以作为时间格式的模板。  
这样处理好处就是时间的表达和解析能力更加灵活，可以适应不同的时间格式。  

# string转time.Time
```go
func Str2Time(str string) time.Time {
	if tm, err := time.Parse("2006-01-02 15:04:05", str); err != nil {
		return time.Now()
	} else {
		return tm
	}
}

func Str2TimeLoc(str string) time.Time {
	if tm, err := time.ParseInLocation("2006-01-02 15:04:05", str, time.Local); err != nil {
		return time.Now()
	} else {
		return tm
	}
}
```
需要注意的是 `time.Parse` 在缺少时区信息的时候，默认是UTC时间(零时区)。  
因此为了确保是本地时区的时间，可以通过 `time.ParseInLocation` 解析。  

# time.Time转string
```go
func Time2Str(t time.Time) string {
	return t.Format("2006-01-02 15:04:05")
}
```

# 参考链接
- https://golang.org/pkg/time
- https://studygolang.com/static/pkgdoc/pkg/time.htm
