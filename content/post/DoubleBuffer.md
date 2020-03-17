---
title: "DoubleBuffer"
date: 2018-05-12T22:30:39+08:00
lastmod: 2018-05-12T22:30:39+08:00
draft: false
tags: ["buffer"]
categories: ["data-structure"]
mathjax: false
---

DoubleBuffer是一种以空间换取并发安全的数据结构。  
<!--more-->

`DoubleBuffer.h` 头文件内容如下：  
```cpp
#pragma once

#include <stdint.h>

template<typename T>
class DoubleBuffer
{
	T _buffer[2];
	uint8_t index;
    
public:
	DoubleBuffer() : index(0) { }
	DoubleBuffer(T const &data) : index(0) { _buffer[index] = data; }

	T const &get() const { return _buffer[index]; }
	T &get() { return _buffer[index]; }

	T const *operator->() const { return &get(); }
	T *operator->() { return &get(); }

	T const &buffer() const { return _buffer[!index]; }
	T &buffer() { return _buffer[!index]; }

	void commit() { index ^= 1; }
};
```
