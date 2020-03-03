---
title: "生成compile_commands.json文件"
date: 2018-04-29T12:01:24+08:00
lastmod: 2018-04-29T12:01:24+08:00
draft: false
tags: ["compile", "json"]
categories: ["trick"]
mathjax: false
---

`compile_commands.json` 文件能够有效提高一些工具(比如说ccls[^1], vscode[^2])的代码跳转、补全等功能。  
因此，本文将会说明如何生成 `compile_commands.json` 文件，特别是使用 `make` 的老工程。  
<!--more-->

# cmake工程
cmake工程生成 `compile_commands.json` 文件比较简单，定义 `CMAKE_EXPORT_COMPILE_COMMANDS` 即可。  
```sh
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=1
```

# make工程
不过很多(旧的)工程都是用 `make` 来编译的，没有现成的选项生成 `compile_commands.json` 文件。  
虽然也可以使用 `ctags`, `gtags` 等，但是跳转其实不是很准确。  
我们可以通过 [Bear](https://github.com/rizsotto/Bear) 来生成，而且不需要改动代码。  
具体Bear的安装这里就不赘述了，按照 [官方文档](https://github.com/rizsotto/Bear#how-to-install) 来即可。  
安装之后，执行以下命令即可生成：  
```sh
bear 编译命令
# 比如说 bear make
```
也就是在原有编译命令之前加上 `bear` 即可，一般都是 `bear make` 。  
生成之后我们就可以愉快地享受更精准的跳转和补全了。:blush: :v:   

## 非原编译环境调整
有的时候不同模块使用的编译环境是不同的，但是想在同一开发机上编码?  
其中一种方法是远程开发(比如说vscode remote[^3])，另外一种就是使用 `docker` 。  
代码checkout在同一台开发机，不同编译环境对应不同docker镜像，代码以volume挂载。  
这样可以避免编译的交叉影响，省去代码的同步以及日后编译机的裁撤。  

不过编译环境的库依赖可能会冲突，所以在开发机上需要存放到不同的目录。  
因此生成的 `compile_commands.json` 文件中的路径需要调整。  
我们先通过 `-o` 选项生成一个备份文件：  
```sh
bear --append -o compile_commands_org.json 编译命令
```

然后通过 `compile_cmds_replace.sh` 脚本替换依赖路径。  
```sh
#!/bin/bash

sed 's#/usr/local/svcfrm2#/data/dev-deps/XxxMod/svcfrm2#g' compile_commands_org.json > compile_commands.json
sed -i 's#/usr/local/commlib#/data/dev-deps/XxxMod/commlib#g' compile_commands.json

# -I单独一行，下一行还是-I
perl -i -0pe 's/"-I",\s+"-I/"-I/gms' compile_commands.json
# -I单独一行，下一行是绝对路径
perl -i -0pe 's/"-I",\s+"\//"-I\//gms' compile_commands.json
# -I单独一行，下一行是相对路径
perl -i -0pe 's/"-I",\s+"\./"-I\./gms' compile_commands.json
```
上面的脚本示例了如何把 `XxxMod` 依赖的库 `svcfrm2` 和 `commlib` 调整为开发机上对应库的路径。  

[^1]: https://github.com/MaskRay/ccls
[^2]: https://code.visualstudio.com
[^3]: https://code.visualstudio.com/docs/remote/remote-overview
