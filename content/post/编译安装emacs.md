---
title: "编译安装emacs"
date: 2018-12-23T23:34:28+08:00
lastmod: 2018-12-23T23:34:28+08:00
draft: true
tags: ["emacs"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何编译、安装Emacs 26.3，以CentOS 8为例。  
<!--more-->

# 下载源码
[Emacs 26.3](https://mirrors.tuna.tsinghua.edu.cn/gnu/emacs/emacs-26.3.tar.xz)

# 安装依赖
```sh
yum install -y gtk3-devel
yum install -y gnutls

yum install -y gtk+-devel gtk2-devel
yum install -y libXpm-devel
yum install -y libpng-devel
yum install -y giflib-devel
yum install -y libtiff-devel libjpeg-devel
yum install -y ncurses-devel
yum install -y gpm-devel dbus-devel dbus-glib-devel dbus-python
yum install -y GConf2-devel pkgconfig
yum install -y libXft-devel
```

# 编译安装
```sh
./configure

make -j$(nproc)
sudo make install
```

# spacemacs依赖包缺失
- 本地镜像  
  https://github.com/syl20bnr/spacemacs-elpa-mirror
- 国内镜像源  
  https://mirrors.cloud.tencent.com/elpa

```lisp
(setq configuration-layer-elpa-archives
    '(
      ("melpa-lc"  . "~/elpa_local/melpa/")
      ("org-lc"    . "~/elpa_local/org/")
      ("gnu-lc"    . "~/elpa_local/gnu/")
      
      ("melpa"   . "https://mirrors.cloud.tencent.com/elpa/melpa/")
      ("org"     . "https://mirrors.cloud.tencent.com/elpa/org/")
      ("gnu"     . "https://mirrors.cloud.tencent.com/elpa/gnu/")
     ))
```

另外如果是离线安装，那么可以把触发 `git clone` 的package加到 `dotspacemacs-excluded-packages`：  
```lisp
dotspacemacs-excluded-packages '(font-lock+ evil-unimpaired)
```
