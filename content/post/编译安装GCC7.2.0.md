---
title: "编译安装GCC 7.2.0"
date: 2018-04-22T22:41:39+08:00
lastmod: 2018-04-22T22:41:39+08:00
draft: false
tags: ["gcc", "scl"]
categories: ["tool"]
mathjax: false
---

服务器默认安装的GCC版本一般都比较低，而一些开源库已经开始依赖高版本GCC才有的特性。  
因此我们需要编译安装新版本的GCC，本文以7.2.0为例进行说明。  
<!--more-->

# 预编译版本
先搜索一下第三方软件仓库是否有高版本的GCC，有的话直接安装即可。  
以CentOS7为例，可以通过SCL下载安装：  
```sh
yum install -y centos-release-scl
yum install -y devtoolset-7-gcc-c++

# 临时使用新版本
scl enable devtoolset-7 bash

# 固定使用新版本，添加下述命令到 .bashrc 或 .bash_profile
source scl_source enable devtoolset-7
```

# 下载源码

## GCC源码
可以在 [官网](ftp://ftp.gnu.org/gnu/gcc/gcc-7.2.0/gcc-7.2.0.tar.gz) ，也可以在附近的镜像源下载。  
我这边就在 [清华镜像源](https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.2.0/gcc-7.2.0.tar.gz) 下载了。  

## 依赖包
依赖包根据编译机器的外网连通情况采用手动下载或者自动下载。  

- 自动下载  
  如果可以访问外网，那么可以直接通过GCC源码包中内置的脚本下载。  
  ```sh
  cd gcc-7.2.0
  ./contrib/download_prerequisites
  ```

- 手动下载  
  如果不方便访问外网，那么可以在通外网的机器上先下载好依赖包。  
  具体依赖哪些包、什么版本，可以参考上面提到的内置脚本。  
  ```sh
  cd gcc-7.2.0
  # 把gmp-6.1.0, mpfr-3.1.4, mpc-1.0.3, isl-0.16.1压缩包解压到该目录
  ln -fs gmp-6.1.0 gmp
  ln -fs isl-0.16.1 isl
  ln -fs mpc-1.0.3 mpc
  ln -fs mpfr-3.1.4 mpfr
  ```

# 编译安装
```sh
./configure --prefix=/usr/local/gcc-7.2.0 --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j$(nproc)
sudo make install
```
`--prefix` 指定安装的目录，这个要选择有写权限的目录，否则安装不了。  
另外不建议安装在旧版本GCC的目录，避免产生不必要的影响。  

# 设置环境变量PATH
```sh
export PATH=/usr/local/gcc-7.2.0/bin:$PATH
```
设置PATH环境变量，优先选择高版本的GCC，需要的话可以加到 `~/.bashrc` 或者 `~/.bash_profile` 里面。  
这样处理可以达到不同版本GCC共存、减少影响的目的，特别是多个开发人员共用服务器的情况尤为重要。  

# libstdc++.so
编译目录下的 `libstdc++.so.6.0.24` 可以复制到 `/usr/lib64/` 并新建软链 `libstdc++.so.6` 指向新的so。  
这样可以解决如下一类 `GLIBCXX` 找不到的问题。   
```
version `GLIBCXX_3.4.20' not found
version `GLIBCXX_3.4.21' not found
```


# 参考链接
- http://caosiyang.github.io/2016/05/04/installing-gcc

