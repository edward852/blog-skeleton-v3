---
title: "CentOS 8编译安装emacs 26.3"
date: 2020-03-03T23:34:28+08:00
lastmod: 2020-03-03T23:34:28+08:00
draft: false
tags: ["emacs"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何编译、安装Emacs 26.3，以CentOS 8为例。  
<!--more-->

# Windows和MacOS
Windows直接在 [附近镜像源](http://ftpmirror.gnu.org/emacs/windows) 下载编译好的版本。  
MacOS通过 [brew](https://brew.sh) 安装即可：  
```sh
brew cask install emacs
```
Linux系统参考下文从源码编译安装。  

# 下载源码
在 [附近镜像源](http://ftpmirror.gnu.org/emacs/) 下载emacs的源代码，本文是26.3版本。  

# 安装依赖
```sh
yum install -y gtk3-devel gnutls-devel
yum install -y libXpm-devel libpng-devel libtiff-devel libjpeg-devel
yum install -y ncurses-devel gpm-devel

#yum install -y giflib-devel
dnf --enablerepo=PowerTools install -y giflib-devel
```
`giflib-devel` 可能yum安装不了，可以通过dnf安装。  

# 编译安装
```sh
./configure

make -j$(nproc)
sudo make install
```

