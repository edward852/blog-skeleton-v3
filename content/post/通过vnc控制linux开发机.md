---
title: "通过vnc控制linux开发机"
date: 2018-10-20T16:32:58+08:00
lastmod: 2018-10-20T16:32:58+08:00
draft: false
tags: ["vnc"]
categories: ["tool"]
mathjax: false
---

本文主要介绍如何通过 `vnc` 控制linux开发机。  
<!--more-->

下文以CentOS为例进行说明，其他系统可以参考一下。  

# 安装图形界面
一般来说开发机默认没有图形界面，主要通过命令行操作。  
如果你的开发机也没有，那么可以输入以下命令进行安装：  
```sh
yum groupinstall -y "X Window system"
# 安装gnome或者kde也可以
yum groupinstall -y "Xfce"
yum groupinstall -y "Chinese support"
```
当然安装GNOME或KDE都是可以的，不过Xfce依赖少些，比较简洁。  

# 安装vnc server
```sh
yum install -y tigervnc-server
```

# vnc server配置
- 设置vnc密码  
  切换到你的用户之后，通过 `vncpasswd` 命令设置vnc的密码，后续连接的时候要用到。  
- 复制 `vncserver@.service` 模板文件并调整   
  ```sh
  cp /lib/systemd/system/vncserver@.service /etc/systemd/system/vncserver@:1.service
  ```
  修改 `vncserver@:1.service` 文件内容，替换 `<USER>` 为你的用户名(以 `edward` 为例)。  
  ```
  ExecStart=/usr/sbin/runuser -l edward -c "/usr/bin/vncserver %i -geometry 1280x960"
  PIDFile=/home/edward/.vnc/%H%i.pid
  ```
  `-geometry` 选项可以指定分辨率。  
- 启用vnc服务  
  ```sh
  systemctl daemon-reload
  systemctl start vncserver@:1.service
  systemctl status vncserver@:1.service
  ```
  这一步检查vnc服务启用是否正常并尝试连接(可以看 [连接](#connect) 章节)。  
  如果你跟我一样装的是Xfce，而且连接不正常，那么先停止vnc服务:  
  ```sh
  systemctl stop vncserver@:1.service
  ```
  修改 `~/.vnc/xstartup` 文件：  
  ```sh
  # exec /etc/X11/xinit/xinitrc
  exec startxfce4
  ```
  注释掉 `xinitrc`，换成 `startxfce4` ，然后执行前面的命令重新启用vnc服务。  
- 开机启动vnc服务  
  测试vnc功能正常后再设置开机启动vnc服务:  
  ```sh
  systemctl enable vncserver@:1.service
  ```

# 安装vnc viewer
在 [这里](https://www.realvnc.com/en/connect/download/viewer) 下载 `vnc viewer`， 最好选绿色版，否则有些公司的安全软件会告警。  

# 安装noVNC(可选)
安装 [noVNC](https://github.com/novnc/noVNC)，可以通过浏览器控制你的开发机。  
为什么要通过浏览器？因为vnc的管理端口`5901`有可能被公司封堵了。  
一般来说Web端口还是放开的，比如说 `80, 8080, 8081` 等。  

- 启动noVNC  
```sh
# 进入noVNC安装目录
# 查看帮助
./utils/launch.sh -h

# 监听8080端口
./utils/launch.sh --vnc localhost:5901 --listen 8080
```

# 连接 {#connect}
- vnc viewer  
  安装好vnc viewer之后我们就可以连接开发机了。  
  在vnc viewer地址栏输入 `开发机IP地址:1` 或者 `开发机IP地址:5901` 即可。  
- noVNC(可选)  
  启动noVNC后，我们就可以通过浏览器管理开发机了。  
  ```
  http://开发机IP地址:8080/vnc.html
  ```

# 异常解决
- 异常关机导致vnc服务无法正常启用  
  可以尝试删除文件后重试：  
  ```sh
  rm /tmp/.X11-unix/X1
  ```
- 不能复制、粘贴  
  ```sh
  vncconfig -nowin &
  ```
