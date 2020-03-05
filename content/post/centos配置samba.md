---
title: "CentOS配置samba"
date: 2019-03-09T14:37:46+08:00
lastmod: 2019-03-09T14:37:46+08:00
draft: false
tags: ["samba"]
categories: ["tool"]
mathjax: false
---

本文主要介绍CentOS如何配置samba服务。  
<!--more-->

# 安装
```sh
yum install -y samba
```

# 创建用户
```sh
smbpasswd -a 你的用户名
```
设置专门用于samba的密码，可以与系统帐号的密码不同。  

# 配置smb.conf
在 `/etc/samba/smb.conf` 最后新增以下内容：  
```
[share]
      comment = Share Folder
      path = samba共享文件夹路径
      browseable = yes
      writable = yes
      create mask = 0664
```
`path` 调整为需要共享的文件夹路径。  

# 设置防火墙
放开samba的服务端口：  
```sh
firewall-cmd --permanent --add-service=samba
firewall-cmd --reload
```

# 设置SELinux
如果SELinux没有关闭，那么需要如下设置：  
```sh
chcon -R -t samba_share_t samba共享文件夹路径
```

# 启动smb服务
```sh
# 启动samba
systemctl start smb

# 开机启动samba
systemctl enable smb
```

# 连接使用
- Windows  
  Windows在资源管理器输入 `\\IP地址\share` 并输入用户名和密码即可。  
  为了方便日后使用，可以在"此电脑"右击选择"映射网络驱动器"映射为网络文件夹。  
- MacOS  
  MacOS的处理类似，在访达选择"前往”->"连接服务器”，连接成功可以收藏。  
