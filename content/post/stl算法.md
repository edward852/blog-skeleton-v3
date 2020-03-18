---
title: "STL算法"
date: 2018-03-18T22:21:23+08:00
lastmod: 2020-03-17T22:21:23+08:00
draft: true
tags: ["cpp", "stl"]
categories: ["language", "algorithm"]
mathjax: false
---

本文主要记录STL算法的使用笔记。  
<!--more-->

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
## find
通用的查找，但只是顺序查找。  
容器若有find方法应该优先使用自带的方法(set, map之类)。  

## find_if

# 修改序列操作
## swap
交换两个变量的值。  

## remove
不会(也没法)修改容器的实际大小，只是把与val值不相等的元素前移覆盖与val值相等的元素(实际是move)。  
remove返回新的结束迭代器(end)，容器可以通过它执行实际的删除操作。  

## replace
## reverse
反转序列。  

## shuffle
随机打乱序列。  

# 大小堆
## make_heap
## pop_heap
## push_heap
## sort_heap

# 排序
## sort
默认从小到大排序(`<`)。  

## nth_element

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
