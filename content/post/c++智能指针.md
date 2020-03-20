---
title: "C++智能指针"
date: 2018-03-25T00:13:42+08:00
lastmod: 2020-03-20T00:13:42+08:00
draft: false
tags: ["cpp", "smart", "pointer"]
categories: ["language"]
mathjax: false
---

本文主要记录C++智能指针的使用。  
<!--more-->

# unique_ptr
unique_ptr是拥有管理对象所有权的指针，在离开作用域时自动释放管理对象的内存。  
unique_ptr通过`make_unique<T>`创建，通过`std::move`显式移交所有权。  

# shared_ptr
shared_ptr是通过引用计数共享管理对象所有权的指针。  
每个指针离开作用域时(或者手动reset)计数减一，减至0则释放管理对象的内存。  
shared_ptr通过`make_shared<T>`创建，通过赋值给同类型指针共享所有权。  

# weak_ptr
weak_ptr是一种借用型指针，不拥有所有权，也不负责释放。  
必要时可以通过`lock`尝试获取对应shared_ptr(可能失败)，使用前需要检查是否为空。  
weak_ptr可以防止循环引用。  

# 示例
```cpp
auto up1 = make_unique<Widget>();
if (up1) up1->display("up1"); // 打印

auto up2 = move(up1);   // 移交所有权给up2
if (up2) up2->display("up2"); // 打印
if (up1) up1->display("up1"); // 不打印，up1已经移交所有权

auto wp1 = weak_ptr<Widget>();
{
    auto sp1 = make_shared<Widget>();
    cout << "sp count:" << sp1.use_count() << endl; // 1
    auto sp2 = sp1; // 引用计数+1
    cout << "sp count:" << sp2.use_count() << endl; // 2

    sp1->display("sp1");    // 打印
    sp2->display("sp2");    // 打印

    auto wp2 = weak_ptr<Widget>(sp1);
    auto ptr = wp2.lock();
    if (ptr) ptr->display("lock wp2"); // 打印，sp1和sp2都未释放

    wp1 = sp1;
    // 离开代码块时sp1对应的Widget析构
}

auto ptr = wp1.lock();
if (ptr) ptr->display("lock wp1");
else { cout << "wp1 expired" << endl; }  // 打印，wp1指向的对象已经析构
```

# 小结
合理使用智能指针，可以减少手工`new, delete`遗漏导致的内存泄露。  
另外尽量依附于作用域(如局部变量)或者对象的生命周期(如实例成员变量)，通过RAII方式管理资源。  

# 参考链接
- [Lifetime Safety By Default](https://github.com/hsutter/gcpp) 
