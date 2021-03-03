---
title: "Golang操作protobuf"
date: 2019-08-18T23:21:41+08:00
lastmod: 2019-08-18T23:21:41+08:00
draft: false
tags: ["golang", "protobuf"]
categories: ["language"]
mathjax: false
---

本文记录golang如何操作protobuf。  
<!--more-->

# 标准包与教程
具体参考 [Protocol Buffer Basics: Go](https://developers.google.com/protocol-buffers/docs/gotutorial) 的说明。  

# 第三方库
使用 [github.com/gogo/protobuf](https://github.com/gogo/protobuf#installation) 提高性能。  

# protoc
在[这里](https://github.com/protocolbuffers/protobuf/releases)下载安装`protoc`。  

# 调试
- DebugString  
  可以输出调试信息。bytes类型可见字符直接打印，其他八进制打印。  
  可以通过如下方式转为十六进制字符串：  
  ```go
  byteSlice := []byte("$\016\000\341\251\000\000\020\000\000\000\000\000\000\000\036")
  fmt.Printf("hex str: %x\n", byteSlice)
  ```
