---
title: "Node.js基础回顾"
date: 2017-09-18T14:16:28+08:00
draft: false
tags: ["node.js", "npm"]
categories: ["language"]
---

认识 [Node.js](https://nodejs.org) 是在一个偶然的机会，当时搭建页面框架需要压缩页面，用到了Node.js模块
[html-minifier](https://www.npmjs.com/package/html-minifier) 。
<!--more-->

要使用html-minifier首先要安装Node.js，然后再通过npm安装模块。\
不同平台安装Node.js都比较简单，按照官网下载页面上说的做就好了。\
Node.js我个人的使用感受是Windows上比较慢、问题较多，而Linux和Mac上比较快、问题较少。

# npm与cnpm

npm是Node.js的包管理器，主要用来安装模块。不过国内用过的人都知道，npm下载非常的慢！\
幸运的是我们有国内的镜像源，也就是[ 淘宝NPM镜像](http://npm.taobao.org/)
, 通过它可以大大加快模块安装的速度。\
使用方法一是设置npm的registry为https://registry.npm.taobao.org
，设置命令是npm set registry。\
使用方法二是安装cnpm：

```bash
$ npm install -g cnpm --registry=https://registry.npm.taobao.org
```

后续就可以通过cnpm像npm一样的安装模块了。个人比较推荐第二种方法。

# Node版本管理

Node.js版本更新很快，记得我最早使用的是0.10.21版本，现在最新都已经到了8.5.0版本。\
更新或者管理node版本，可以通过n模块实现：

``` {.bash org-language="sh"}
$ npm install -g n
```

常用的就是n latest或者n lts命令了。没有特别需求的话，建议安装lts(Long
Term Support)版本，不用频繁更新。

# 离线安装模块

正如国内被墙一样，公司内一般也会限制访问外部网站，所以经常出现安装模块失败的情况，根源都在于某些url访问不了。\
申请访问权限手续麻烦是一方面，另一方面是我们不清楚模块安装的过程都要访问哪些url，来回多次申请肯定是不现实的。\
还好已经有现成的模块可以解决这个问题，就是
[npmbox](https://www.npmjs.com/package/npmbox) 。\
npmbox其实有个鸡生蛋还是蛋生鸡的问题，就是离线安装模块需要用到npmbox，而npmbox本身也需要安装，也需要访问许多外网地址...\
npmbox解决这个问题的方法还是挺巧妙的，就是在能访问外网的电脑上(比如说你自己家里的电脑)先把npmbox需要的资源都先下载好弄成压缩包，然后再转到公司网络内(作为研发人员这种权限都有吧)，最后解压、作为npm的cache文件夹安装npmbox。\
说到这里，可能有人会问，公司内怎么安装Node.js？如果外网访问权限高当然是直接安装就好了，只有传入文件的权限，那么就只能通过下载安装包进行安装了。\
下载、安装完npmbox后，安装其他模块的基本流程也是差不多的：

``` {.bash org-language="sh"}
$ npmunbox xxxModule.npmbox
```

其中xxxModule.npmbox就是在外网通过npmbox下载、生成的压缩包，可以在内网离线安装。

# Hello World

Node.js官方文档上有Hello World的
[example](https://nodejs.org/api/synopsis.html#synopsis_example)
，是一个简单的Web服务器返回Hello
World页面的例子，展示了Node.js的简洁与强大。\
我这里也简单的举个例子，首先新建一个example.js文件，内容如下：

``` {.javascript}
console.log('Hello World!');
```

然后执行命令：

``` {.bash org-language="sh"}
$ node example.js
```

一个超简单的hello world就出来啦。

# 小结

本文主要是解决搭建Node.js开发环境可能遇到的一些比较基础的问题，深入的掌握还需要通过查看
[api文档](https://nodejs.org/api/) 和实践来解决。
