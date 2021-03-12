---
title: "C++中的memory order"
date: 2020-12-12T15:27:37+08:00
lastmod: 2021-03-12T15:27:37+08:00
draft: false
tags: ["cpp", "memory"]
categories: ["language"]
mathjax: false
---

本文主要记录个人对C++中memory order的一点理解。  
<!--more-->

# what
memory order指定了(编译器和CPU)如何安排(当前线程)原子操作附近的内存访问指令。  
> memory order specifies how memory accesses, including regular, non-atomic memory accesses, are to be ordered around an atomic operation. 

# why

为什么需要指定memory order？  
- 一般语句并不是原子操作  
- 多线程访问共享变量没有限制则结果不确定  
- 编译器或CPU可能调整指令执行顺序  
- CPU cache导致不同核心不一定得到一致的内存数据  

![sb_gmem](/images/store_buffer_global_mem.png)  

# 类型
## Relaxed ordering
这种排序比较宽松，主要确保是原子操作，不要求同步、其它读写没限制。  
典型使用场景是计数器。  
```cpp
#include <vector>
#include <iostream>
#include <thread>
#include <atomic>
 
std::atomic<int> cnt = {0};
 
void inc_counter()
{
    for (int n = 0; n < 1000; ++n) {
        cnt.fetch_add(1, std::memory_order_relaxed);
    }
}
 
int main()
{
    std::vector<std::thread> v;
    for (int n = 0; n < 10; ++n) {
        v.emplace_back(inc_counter);
    }
    for (auto& t : v) {
        t.join();
    }
    std::cout << "Final counter value is " << cnt << '\n';
}
```

## Release-Acquire ordering
| 取值                 | 含义                                                                                                                                                                             |
|----------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| memory_order_release | no reads or writes in the current thread can be reordered after this store. All writes in the current thread are visible in other threads that acquire the same atomic variable. |
| memory_order_acquire | no reads or writes in the current thread can be reordered before this load. All writes in other threads that release the same atomic variable are visible in the current thread  |

![acq_rel](/images/acquire_release.png)  

memory_order_release限制当前线程store操作之前的读写(指令)不能重排到store后面执行。  
memory_order_acquire限制当前线程load操作之后的读写(指令)不能重排到load前面执行。  
副作用是线程A的store操作之前的所有写对于load到该值的线程B是可见的。  
常见的互斥锁、自旋锁就是这种释放、获取的内存访问同步模式。  

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");
    data = 42;              // 写入data不能重排到store后面
    ptr.store(p, std::memory_order_release);
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_acquire)))
        ;
    assert(*p2 == "Hello"); // 断言绝不会失败
    assert(data == 42);     // 断言绝不会失败；读取data不能重排到load前面
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```

## Release-Consume ordering
| 取值                 | 含义                                                                                                                                                                                                                                             |
|----------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| memory_order_release | no reads or writes in the current thread can be reordered after this store. Writes that carry a dependency into the atomic variable become visible in other threads that consume the same atomic.                                                |
| memory_order_consume | no reads or writes in the current thread dependent on the value currently loaded can be reordered before this load. Writes to data-dependent variables in other threads that release the same atomic variable are visible in the current thread. |

memory_order_release限制当前线程store操作之前**原子变量依赖**的读写(指令)不能重排到store后面执行。
memory_order_consume限制当前线程load操作之后**依赖原子变量**的读写(指令)不能重排到load前面执行。
与Release-Acquire ordering类似，不过副作用没那么大。  
线程A中只有原子变量所依赖数据的写才对给load到值的线程B中依赖原子变量的运算和操作可见。  

```cpp
#include <thread>
#include <atomic>
#include <cassert>
#include <string>
 
std::atomic<std::string*> ptr;
int data;
 
void producer()
{
    std::string* p  = new std::string("Hello");
    data = 42;              // p不依赖data
    ptr.store(p, std::memory_order_release);
}
 
void consumer()
{
    std::string* p2;
    while (!(p2 = ptr.load(std::memory_order_consume)))
        ;
    assert(*p2 == "Hello"); // 断言绝不会失败，p2依赖ptr
    assert(data == 42);     // 断言可能会失败，data不依赖ptr
}
 
int main()
{
    std::thread t1(producer);
    std::thread t2(consumer);
    t1.join(); t2.join();
}
```

## Sequentially-consistent ordering
| 取值                 | 含义                                                        |
|----------------------|-------------------------------------------------------------|
| memory_order_seq_cst | 不允许重排+所有原子变量的修改顺序对于相关线程来说都是一致的 |

限制性最强、对性能有些影响、标准库默认使用的memory order。  
下面的例子只有使用memory_order_seq_cst才能保证线程c和d观察到一致的原子变量x和y的修改顺序。  
要么都是x先改y后改，要么都是y先改x后改，不可能线程c观察到x先改y后改、线程d观察到y先改x后改。  

> Any other ordering may trigger the assert because it would be possible for the threads c and d to observe changes to the atomics x and y in opposite order.
```cpp
#include <thread>
#include <atomic>
#include <cassert>
 
std::atomic<bool> x = {false};
std::atomic<bool> y = {false};
std::atomic<int> z = {0};
 
void write_x()
{
    x.store(true, std::memory_order_seq_cst);
}
 
void write_y()
{
    y.store(true, std::memory_order_seq_cst);
}
 
void read_x_then_y()
{
    while (!x.load(std::memory_order_seq_cst))
        ;
    if (y.load(std::memory_order_seq_cst)) {
        ++z;
    }
}
 
void read_y_then_x()
{
    while (!y.load(std::memory_order_seq_cst))
        ;
    if (x.load(std::memory_order_seq_cst)) {
        ++z;
    }
}
 
int main()
{
    std::thread a(write_x);
    std::thread b(write_y);
    std::thread c(read_x_then_y);
    std::thread d(read_y_then_x);
    a.join(); b.join(); c.join(); d.join();
    assert(z.load() != 0);  // 断言绝不会失败，z只有1或2两种取值
}
```

# 与volatile的关系
volatile只是对当前线程有效，对多线程无效(无限制)，而且不是原子操作。  

# 参考资料
- [memory order](https://en.cppreference.com/w/cpp/atomic/memory_order)  
- [atomic Weapons: The C++ Memory Model and Modern Hardware](https://herbsutter.com/2013/02/11/atomic-weapons-the-c-memory-model-and-modern-hardware)  
- [如何理解 C++11 的六种 memory order？](https://www.zhihu.com/question/24301047)  
- [理解 C++ 的 Memory Order](http://senlinzhan.github.io/2017/12/04/cpp-memory-order)  

