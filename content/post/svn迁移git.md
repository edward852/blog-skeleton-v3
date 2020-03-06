---
title: "svn迁移git"
date: 2019-10-20T13:59:15+08:00
lastmod: 2019-10-20T13:59:15+08:00
draft: false
tags: ["git", "svn"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何把项目从svn迁移到git。  
<!--more-->

# 迁移
## 不保留历史记录
如果不需要保留历史记录，那么只要svn checkout干净版本后推送到git仓库即可。  
```sh
svn checkout svn仓库路径 本地文件夹名

cd 本地文件夹
git init
git add .
git commit -m "svn迁移到git"
git remote add origin git仓库路径
git push -u origin --all
git push origin --tags
```

## 保留历史记录
```sh
git svn clone svn仓库路径 本地文件夹名 其他参数
```
如果svn仓库是标准布局可以使用 `-s` 或者 `--stdlayout` 参数。  
svn仓库不是标准布局，则需要用到 `-T`, `-b`, `-t` 参数。  
另外如果只是迁移，建议加上 `--no-metadata` 参数。  

```sh
# 标准布局 trunk branches tags
git svn clone svn仓库路径 本地文件夹名 -s --no-metadata

# 非标准布局 trunk/modX branches/modX tags/modX
git svn clone svn仓库路径 本地文件夹名 -T trunk/modX -b branches/modX -t tags/modX --no-metadata

# 非标准布局，且没有branches、tags
git svn clone svn仓库路径 本地文件夹名 --no-metadata
```

# FAQ
- svn external怎么处理？  
  转化为git submodule处理：  
  ```sh
  git submodule add 外部git仓库 当前仓库内相对路径
  # 注意带--recursive参数才能把submodule代码一并拉取
  git clone --recursive git仓库路径
  ```
- 提交者的信息不对怎么处理？  
  git svn clone时通过 `--authors-prog` 参数指定转换脚本。示例脚本如下：  
  ```sh
  #!/bin/sh
  
  echo "$1 <$1@example.com>"
  ```

# 参考链接
- [git-svn文档](https://git-scm.com/docs/git-svn)
