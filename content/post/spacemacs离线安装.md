---
title: "spacemacs离线安装"
date: 2019-01-12T15:10:44+08:00
lastmod: 2019-01-12T15:10:44+08:00
draft: false
tags: ["spacemacs", "offline"]
categories: ["ide"]
mathjax: false
---

本文主要介绍如何离线安装spacemacs。  
<!--more-->

# 前提条件
- 网络条件  
  一台能访问外网的电脑P，一台需要安装spacemacs但访问不了外网的电脑S。  
  电脑P和电脑S能够传输文件。  
- Emacs  
  安装Emacs25.1以上版本。  

# 下载依赖
可以访问外网的电脑把依赖的压缩包下载好，然后传到访问不了外网的电脑。  

- elpa镜像  
  在 [这里](https://github.com/syl20bnr/spacemacs-elpa-mirror/archive/master.zip) 下载压缩包。  
- spacemacs  
  下载 [稳定版](https://github.com/syl20bnr/spacemacs/archive/master.zip) 或者 [开发版](https://github.com/syl20bnr/spacemacs/archive/develop.zip) 压缩包。  

# 安装
注意：如果是Windows电脑，请先设置 `HOME` 环境变量指定用户home目录(等同于 `~/`)。  
在访问不了外网的电脑上进行如下操作：  

- 备份或删除  
  如果Emacs没什么重要配置，那么可以删掉 `.emacs.d` 和 `.emacs` 。  
  ```sh
  # 进入home目录
  cd ~/
  mv .emacs.d .emacs.d.bak
  mv .emacs .emacs.bak
  ```
  
- 解压依赖包  
  ```sh
  # 进入到压缩包所在目录
  unzip -q -d ~/ spacemacs-elpa-mirror-master.zip
  mv ~/spacemacs-elpa-mirror-master ~/elpa_local
  
  unzip -q -d ~/ spacemacs-develop.zip
  mv ~/spacemacs-develop ~/.emacs.d
  ```
  elpa_local目录下有 `gnu`, `melpa`, `org` 等文件夹。  
  稳定版压缩包名字应该是 `spacemacs-master.zip` ，相应调整即可。  
  Windows用户类似处理，解压到之前 `HOME` 环境变量指定的目录(等同于 `~/`)。  
- 启动Emacs  
  启动Emacs之后，默认回车两次即可，接着安装package但是会报错，不用管先关闭Emacs。  
- 修改 `.spacemacs` 文件  
  在 `.spacemacs` 文件中的 `dotspacemacs/user-init` 函数中添加以下内容：  
  ```lisp
  (defun dotspacemacs/user-init ()
    ;; 以上内容忽略
    (setq configuration-layer-elpa-archives
      '(
        ("melpa-lc"  . "~/elpa_local/melpa/")
        ("org-lc"    . "~/elpa_local/org/")
        ("gnu-lc"    . "~/elpa_local/gnu/")
       ))
    (setq package-check-signature nil)
    ;; 以下内容忽略
  )
  ```
  重新启动Emacs会在本地安装package，绝大部分都是可以装上的。  
  如果还有部分依赖包缺失，可以在 `.spacemacs` 文件中的 `dotspacemacs-excluded-packages` 加入。  
  比如说 `font-lock+` 和 `evil-textobj-line` 安装报错，则 `dotspacemacs-excluded-packages` 改动如下：  
  ```lisp
  dotspacemacs-excluded-packages '(font-lock+ evil-textobj-line)
  ```
以上步骤都没问题的话，那么spacemacs就已经安装好了，可以尽情享用~ :tada:  

# 问题排查
如果还有其他问题，可以通过 `emacs --debug-init` 启动Emacs进行调试。  

# 后记
如果你们公司有elpa镜像源的话就更好了，以腾讯软件源为例，参考以下内容修改即可。  
```lisp
(setq configuration-layer-elpa-archives
    '(
      ;; 本地镜像源
      ("melpa-lc"  . "~/elpa_local/melpa/")
      ("org-lc"    . "~/elpa_local/org/")
      ("gnu-lc"    . "~/elpa_local/gnu/")
      
      ;; 腾讯软件源
      ("melpa"   . "https://mirrors.cloud.tencent.com/elpa/melpa/")
      ("org"     . "https://mirrors.cloud.tencent.com/elpa/org/")
      ("gnu"     . "https://mirrors.cloud.tencent.com/elpa/gnu/")
     ))
```
