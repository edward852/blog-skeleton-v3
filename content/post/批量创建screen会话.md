---
title: "批量创建screen会话"
date: 2018-05-13T22:54:13+08:00
lastmod: 2018-05-13T22:54:13+08:00
draft: false
tags: ["screen"]
categories: ["tool"]
mathjax: false
---

对于跳板机登录和执行耗时任务的情况，我们希望能够快速恢复会话并且不受办公电脑断电影响。  
这个时候 [screen](https://www.gnu.org/software/screen) 就派上用场了，我们可以创建screen会话。  
<!--more-->

# createScreen.sh
`createScreen.sh` 可以创建一个screen会话并执行任意命令(包括需要交互的命令)。  
```sh
#!/bin/bash

if [ $# -lt 2 ]
then
echo "Usage: ${prog##*/} sessName cmd1 cmd2 ..."
exit -1
fi

sessName=$1
shift

# 杀掉旧的同名session
screen -X -S $sessName quit > /dev/null 2>&1

# 创建session
screen -dmS $sessName

until [ $# -eq 0 ]
do
    cmd=$1
    echo "exec cmd: $cmd"
    # stuff是命令参数，不能省略
    screen -S $sessName -p 0 -X stuff "$cmd"$'\n'
    shift;
done
```

# createScreens.sh
```sh
#!/bin/bash

screen -wipe

# 免密登录的服务器
sh createScreen.sh test-srv 'ssh 服务器IP' 'sudo -i'

# 权限受限需要密码登录的服务器
sh createScreen.sh dev-srv  'ssh 服务器IP' '密码' '可选执行的命令'
```
由于 `createScreen.sh` 支持交换命令的执行，因此可以ssh到服务器(是否需要输入密码都可以)。  
这样我们就可以批量创建screen会话，达到类似自动登录的效果，快速恢复日常办公的场景。  

# screen常用操作
```sh
# 列出所有会话
screen -ls

# 恢复会话
screen -r 会话名
```
通过按键 `Ctrl-a d` 可以从当前会话detach离开。  
