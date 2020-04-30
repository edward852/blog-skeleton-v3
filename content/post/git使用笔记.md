---
title: "Git使用笔记"
date: 2019-11-09T15:15:51+08:00
lastmod: 2020-04-29T14:55:51+08:00
draft: false
tags: ["git"]
categories: ["note"]
mathjax: false
---

本文主要记录git使用的一些笔记。  
<!--more-->

# 安装
- Windows  
  安装TortoiseGit或者[Git官方工具](http://git-scm.com/download/win)  
- Linux  
  `yum` 或 `apt-get` 安装即可。  
  如果版本低不支持HTTP，那么在安装curl-devel和expat-devel后下载Git源码编译安装。  
- MacOS  
  通过 `brew` 安装即可。  

# 配置
```sh
# 用户名和邮箱
git config --global user.name  "edward"
git config --global user.email "edward@example.com"

# 保存凭证
git config --global credential.helper store

# 命令别名
git config --global alias.st status
git config --global alias.ci commit
git config --global alias.di diff
git config --global alias.br branch
git config --global alias.co checkout
git config --global alias.rst reset
git config --global alias.cp cherry-pick
git config --global alias.hist "log --oneline"
git config --global alias.update '!git pull && git submodule update --init --recursive'

# 中文路径乱码
git config --global core.quotepath false
```

# 自动补全
在 `~/.bashrc` 加上:  
```sh
source /etc/bash_completion.d/git
```

# commit id
通过 `git log` 可以查看到某个提交对应的commit id(提交id)，是一长串的hash值。  
对于需要指定commit id的命令，并不需要输入完整，取前6~8个字符即可，比如说98b19bcd。  

# 常用命令
## 仓库管理
### 新建
```sh
# 进入需要建立仓库的目录
git init
```
### 克隆
```sh
git clone git仓库路径 本地文件夹名
git clone -b 分支名 git仓库路径 本地文件夹名
```

## 修改相关
这里的修改包括修改、新增、删除。  

### 添加
- 普通文件或目录  
  ```sh
  # 一般先git status、git diff查看修改
  git add 文件或目录
  ```
  如果 `git add` 后又有修改，也需要再add一次。  

- .gitignore  
  `.so`, `.a` 文件可能会被git add忽略。  
  ```sh
  # 添加包括忽略的文件
  git add --force 文件或目录
  git add -a 文件或目录
  ```
  如果想忽略某些文件，可以添加到 `.gitignore` 文件内。  

- 空文件夹  
  空文件夹内新建.gitkeep空文件，然后再git add。  
  .gitkeep只是习惯用法，非git官方支持。  

- 添加执行权限  
  ```sh
  git update-index --chmod=+x 可执行文件名
  ```


### 撤消
- 个人分支或本地修改  
  ```sh
  git checkout -- 文件
  # 撤消删除
  git checkout HEAD 文件或目录
  
  # 回退(回滚)到指定提交id(比如说HEAD)
  git reset --hard 提交id
  git push --force origin 分支名
  ```
- 协作分支的修改  
  生成新的提交来撤消指定的提交。  
  ```sh
  git revert 提交id
  ```

## 查看状态
```sh
git status
git status -s
```

## 查看差异
```sh
git diff
git diff 提交id1 提交id2
git diff HEAD^1 HEAD

git diff --staged
git diff --cached 提交id
```

## 提交
提交修改到本地仓库。  
```sh
git commit -m "commit message"
```

## 查看日志
```sh
# 查看最近8条日志
git log -n 8

# 精简版日志
git log --oneline --graph
```

## 推送
推送本地改动到远端仓库。  
```sh
# 推送本地分支到远端仓库对应分支(不存在则新建)
git push origin 分支名

# rebase/reset之后
git push --force origin 分支名
```

## 拉取
拉取远端仓库更新到本地仓库。  
```sh
# git fetch + git merge
git pull

# git fetch + git rebase
git pull --rebase
```

# 分支管理
## 查看
```sh
# 查看本地分支以及当前分支
git branch

# 查看所有分支
git branch -a
```

## 检出 / 切换
```sh
git checkout 分支名
```

## 创建
```sh
# 基于当前分支新建分支
git checkout -b 新分支名

# 基于某分支或某次提交新建分支
git checkout -b 新分支名 提交id
git checkout -b 新分支名 基于的分支名
```

## 推送
```sh
# 推送本地分支到远端仓库
git push origin 分支名
```

## 删除
```sh
# 删除本地分支
# 已合入则使用-d即可，否则使用-D强制删除
git branch -D 分支名

# 删除远程分支
git push origin --delete 分支名
```

## 重命名
```sh
# 本地分支重命名
git branch -m 旧分支名 新分支名
```

# tag管理
## 打tag
```sh
git tag -m "export bug fix" -a v1.1
git tag -m "release" -a v1.2 提交id
```

## 查看
```sh
git tag
```

## 推送
```sh
git push origin --tags
```

## 删除
```sh
git push origin --delete tag tag名
```

## 检出
```sh
git checkout tag名
```

# 子模块 submodule
## 添加
```sh
# 添加子模块
git submodule add 外部git仓库 当前仓库内相对路径
```
注意git clone时需要带上 `--recursive` 参数。  

## 更新 / 补拉
更新子模块或者拉取遗漏的子模块(git clone时没带 `--recursive` 参数)：  
```sh
# 补拉
git submodule update --init --recursive

# 更新
git submodule update --remote
```

# rebase
rebase操作只用于整理 **个人开发分支** 的提交历史，不对(远端已有的)其他分支执行rebase操作。  

## 修改变基
所基于的分支拉取最新的、别人的提交后，在此基础上重放自己的修改(可能有冲突)。  
这样可以减少后续合入的冲突，提交历史更清晰易懂。  
```sh
# 改变所基于分支的版本(可能有冲突)
git rebase 基于的分支名

# rebase有冲突，解决冲突后(参考本文对应章节)
git rebase --continue

# 放弃rebase
git rebase --abort

# 忘了rebase --continue，后面又有提交
git rebase --skip
```

## 整理提交历史
整理指定提交id之后的提交历史：  
```sh
git rebase -i 提交id
# 交互模式下通过pick, squash, fixup等选择需要的提交
```

# 解决冲突 {#conflict}
## 预防
定期执行 [rebase](#rebase) 操作可以减少合入时的冲突。  

## 处理
通过工具解决冲突，这里推荐使用 [Beyond Compare](http://www.scootersoftware.com/download.php) 。  
参考 [这里](http://www.scootersoftware.com/support.php?zz=kb_vcs#gitlinux) 设置git使用的diff和merge工具。  
```sh
git mergetool
git mergetool 冲突文件
```

# 紧急切换分支
## 方法1
提交本地修改，然后切换分支。  
本地修改可能有问题(编译不过，未测试等)，不过没关系，后续可以通过 [rebase](#rebase) 整理提交历史后合入。  

## 方法2
通过 `git stash` 我们可以快速恢复工作区干净，后续通过 `git stash apply` 恢复。  
```sh
git stash

# 查看储藏
git stash list

# 恢复指定储藏
git stash apply stash@{1}

# 删除指定储藏
git stash drop stash@{0}
```

# 同步提交到其他分支
```sh
# git log查看待同步提交的id
git checkout 待同步分支名
git cherry-pick 待同步提交id
```

# 有问题提交的定位
## 确定责任人
```sh
git blame 文件
```

## bisect二分法定位
设置待定位提交的区间：  
```sh
git bisect start 有问题提交id 没问题提交id
```

### 手动
```sh
# 当前提交是好的
git bisect good

# 当前提交是坏的
git bisect bad
```

### 自动
```sh
# 测试脚本：若是好的提交返回0，否则返回其它值
git bisect run ./test.sh
```

# 生成diff文件、应用
```sh
# 也可以 git format-patch
git diff file > xxx.diff
git apply xxx.diff
```

# svn迁移git
具体参考 [svn迁移git](/post/svn迁移git/) 的说明。  

# 小号操作
git clone后马上修改仓库下 `.git/config` 文件的如下字段：  
```sh
[remote "origin"]
    url = https://小号名@github.com/xxx.git

[user]
    name=小号名
    email=小号邮箱
```
第一次push会提示输入密码，后续就不用了。  
