---
title: "Golang操作JSON"
date: 2018-05-05T18:57:46+08:00
lastmod: 2018-05-05T18:57:46+08:00
draft: false
tags: ["json", "golang"]
categories: ["language"]
mathjax: false
---

本文主要介绍golang如何操作JSON。  
<!--more-->

# 标准包
golang可以通过标准包 [encoding/json](https://golang.org/pkg/encoding/json) 进行JSON操作。  
具体可以参考 [JSON and Go](https://blog.golang.org/json-and-go) , 下文主要提下需要注意的点。  

## 编码
```go
b, err := json.Marshal(m)
```
编码可以通过 `json.Marshal` 把数据转成JSON格式。  
注意只有结构体中大写的字段才会导出，这是golang特有的规则，比较容易踩坑。  
另外如果想改变字段的导出方式(别名、大小写)，可以通过 `struct tag`[^1] 指定。  

## 解码
```go
err := json.Unmarshal(b, &m)
```
解码可以通过 `json.Unmarshal` 把JSON数据解析成golang类型(结构体、数组)。  
Unmarshal只会解析指定的字段，其他会忽略。  

# 第三方包 jsoniter
第三方包json iterator(简称jsoniter) 有以下优点:  
- 性能更优
- 与标准JSON完全兼容
- 编码更便捷，无需提前定义结构

具体可以参考 [jsoniter文档](http://jsoniter.com/migrate-from-go-std.html) 。  
总而言之，推荐使用 `jsoniter` 替代标准JSON包，只要简单几行代码即可。  
```go
//import "encoding/json"
import "github.com/json-iterator/go"

var json = jsoniter.ConfigCompatibleWithStandardLibrary
json.Marshal(&data)
json.Unmarshal(input, &data)
```

[^1]: https://golang.org/pkg/reflect/#StructTag
