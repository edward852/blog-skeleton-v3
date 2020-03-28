---
title: "STL算法"
date: 2018-03-18T22:21:23+08:00
lastmod: 2020-03-27T22:21:23+08:00
draft: false
tags: ["cpp", "stl"]
categories: ["language", "algorithm"]
mathjax: false
---

本文主要记录STL算法的使用笔记。  
<!--more-->

# lamda, 仿函数, 函数指针
需要传入函数的算法，优先使用lamda和仿函数。  
lamda简洁、仿函数通用，而函数指针在个别算法或容器使用比较麻烦。  

# min, max
## min
返回最小值。  

## max
返回最大值。  

## min_element
返回序列中最小值的迭代器。  

## max_element
返回序列中最大值的迭代器。  

# 只读序列操作
## for_each
与C++11 for迭代语法类似。  

## find
通用的查找，但只是顺序查找。  
容器若有find方法应该优先使用自带的方法(set, map之类)。  

## find_if
查找符合条件的首个元素。  

# 修改序列操作
## swap
交换两个变量的值。  

## transform
序列转换。  

## remove
不会(也没法)修改容器的实际大小，只是把与val值不相等的元素前移覆盖与val值相等的元素(实际是move)。  
remove返回新的结束迭代器(end)，容器可以通过它执行实际的删除操作。  

## remove_if
"删除"符合条件的所有元素，删除效果参考上面remove的说明。  

## replace
更新所有值为val元素的值。  

## replace_if
更新所有符合条件元素的值。  

## reverse
反转序列。  

## random_shuffle
随机打乱序列。  

# 大小堆
优先队列、Top N需求、定时器等常用的算法。  

## make_heap
建堆，默认大堆(默认通过 `<` 比较)。  

## pop_heap
从堆中删除元素。  

## push_heap
插入元素到堆中。  

## sort_heap
对堆排序，默认升序。  
比较函数需要与建堆使用的一致。  

# 排序
## sort
默认从小到大排序(`<`)。  

## stable_sort
效果与sort类似，另外还保证值相等的元素在排序前、排序后的相对位置不变。  
使用场景：商品默认按价格排序，在某些场景下还需要兼顾按销量排序(数据已经按价格排序)。  

## is_sorted
默认检查序列是否为升序。可以传入比较函数定制顺序。  

## nth_element
部分排序直到找到排在N位置的元素，之前的元素都不大于它，之后的元素都不小于它。  
不需要完全有序，所以比sort快。  

# 二分搜索
使用前提是序列已经是有序的。  

## lower_bound
`>=`语义查找满足条件的首个元素。  

## upper_bound
`>`语义查找满足条件的首个元素。  

## binary_search
`==`语义查看是否有满足条件的元素。

# 有序序列Merge
## set_intersection
## set_union
## set_difference

# 排列组合
## next_permutation

# 数值类

## accumulate
默认对序列求和，也可以传入二元函数根据元素值和当前累计值算出新累计值。  
```cpp
int sum = accumulate(v.begin(), v.end(), 0);
```

## partial_sum
部分和，每个元素的新值是其前面元素以及自身的总和。  
主要是统计方面使用，减少重复计算。  
比如说统计大于a的数量有多少、大于b(满足a<b)的数量又有多少。  
```
y0 = x0
y1 = x0 + x1
y2 = x0 + x1 + x2
y3 = x0 + x1 + x2 + x3
y4 = x0 + x1 + x2 + x3 + x4
```

## 最大公约数
可以使用 `__gcd` 内置函数。  
```cpp
uint64_t gcd(uint64_t a, uint64_t b)
{
    return 0==b? a: gcd(b, a%b);
}
```

# 参考资料
- 《STL源码剖析》 by 侯捷  
- [EASTL](https://github.com/electronicarts/EASTL)  
- [cpp reference](http://www.cplusplus.com/reference/stl/)  

