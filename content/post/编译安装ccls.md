---
title: "编译安装ccls"
date: 2018-07-01T16:25:53+08:00
lastmod: 2020-03-19T16:25:53+08:00
draft: false
tags: ["ccls", "lsp"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何编译、安装 [ccls](https://github.com/MaskRay/ccls) 。  
<!--more-->

ccls是符合微软 [LSP标准](https://microsoft.github.io/language-server-protocol/specification.html) 的C/C++ language server。  
ccls能够提供更好的跳转定义/引用、代码补全、符号搜索和重命名等功能，我们赶紧安装ccls来体验吧！  

# 检出代码
```sh
git clone --depth=1 --recursive https://github.com/MaskRay/ccls
```

# 安装依赖
- cmake  
  到 [cmake官网](https://cmake.org/download) 下载安装即可  
- GCC  
  参考 [编译安装GCC 7.2.0](/post/编译安装gcc7.2.0) 操作即可  
- clang+llvm  
  参考 [编译安装clang+llvm](/post/编译clang+llvm) 操作即可  

# 编译安装
```sh
cd ccls
cmake . -DCMAKE_PREFIX_PATH=/usr/local/ -DCMAKE_CXX_COMPILER=/usr/local/gcc-7.2.0/bin/g++ -DCMAKE_CXX_LINK_FLAGS="-Wl,-rpath,/usr/local/gcc-7.2.0/lib64 -L/usr/local/gcc-7.2.0/lib64"
make -j$(nproc)
sudo make install
```
注意根据新版本GCC的安装目录调整上面的cmake命令参数。  

# 使用
vscode安装相应的ccls插件即可。  
spacemacs中添加lsp layer，指定c-c++-backend为lsp-ccls即可。  
```lisp
dotspacemacs-configuration-layers
'(
 ;;c-c++
 (c-c++ :variables
        c-c++-backend 'lsp-ccls
        c-c++-default-mode-for-headers 'c++-mode)
 ;; other layers
)
```

# compile_commands.json
虽然ccls支持 `.ccls` 和 `compile_commands.json`来配置项目，不过后者能够提供更准确的编译信息。  
具体可以参考 [生成compile_commands.json文件](/post/生成compile_commands.json文件/) 的说明，基于makefile的旧项目也可以生成。  

# 参考链接
- https://github.com/MaskRay/ccls/wiki/Build
