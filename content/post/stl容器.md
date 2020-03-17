---
title: "STL容器"
date: 2018-03-17T22:21:16+08:00
lastmod: 2020-03-17T22:21:16+08:00
draft: false
tags: ["cpp", "stl", "container"]
categories: ["language"]
mathjax: false
---

本文主要记录STL容器的使用笔记。  
<!--more-->

# array
静态数组，对C风格数组简单封装。  

# vector
动态数组。空间不够的时候会realloc。  
如果能够预估容量，调用`reserve`方法能够提高性能。  

# queue
先进先出。  

## queue

## priority_queue
优先队列。  

# stack
后进先出。  

# 大小堆
不算容器类，在stl算法类中实现。  
Top N类需求或者优先队列(可以使用priority_queue)、定时器比较常用。  

# set
基于红黑树的set，内部有序(可有序遍历)。  
比unordered_set要慢。  

## set
存储不重复数据的集合。  

## multiset
可存储重复数据的集合。  

# unordered_set
基于哈希表的set，无序但比set性能更优。  

## unordered_set

## unordered_multiset

# map
基于红黑树的map，内部有序(可有序遍历)。存储键值型数据。  
比unordered_map要慢。  

## map
可存储不重复键值型数据。  

## multimap
可存储重复键值型数据。  

# unordered_map
基于哈希表的map，无序但比map性能更优。  

# deque
双端队列，可以在前后快速插入或取出数据。  

# list
双向链表。  

# forward_list
单向链表。  

# bitset
bitmap，用位存储数据(01, bool)，节省空间。  
