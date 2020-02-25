---
title: "Golang临时文件操作"
date: 2018-04-15T18:52:32+08:00
lastmod: 2018-04-15T18:52:32+08:00
draft: false
tags: ["golang", "temp", "file"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang中临时文件的操作。  
<!--more-->

# 创建
```go
tmpFile, err := ioutil.TempFile(os.TempDir(), "xxxSvc-")
if err != nil {
    log.Fatal("Cannot create temporary file", err)
}

defer func() {
    tmpFile.Close()
    os.Remove(tmpFile.Name())
}()
```
通过 [ioutil.TempFile](https://golang.org/pkg/io/ioutil/#TempFile) 我们可以创建临时文件。  
`TempFile` 第一个参数是创建临时文件所在的目录，传入空字符串的话其实也相当于传入 `os.TempDir()` 。  
第二个参数是指定临时文件的名字的模式，随机字符串会替换其中的 `*` ，如果没有会加到最后。  
另外临时文件需要手动调用 `os.Remove` 删除。  

# 读写
```go
text := []byte("content of temporary file")
if _, err = tmpFile.Write(text); err != nil {
    log.Fatal("Failed to write to temporary file", err)
}
```
临时文件创建之后的操作跟普通文件一样，调用 `Read`, `Write` 等方法进行读写即可。  
同样的，文件操作完需要调用 `Close` 关闭，避免资源泄露。  

# 参考链接
- https://golangcode.com/creating-temp-files
