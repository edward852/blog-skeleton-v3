---
title: "Docker使用笔记"
date: 2020-01-09T11:05:00+08:00
lastmod: 2020-01-09T11:05:00+08:00
draft: false
tags: ["docker"]
categories: ["note"]
mathjax: false
---

本文主要记录docker使用过程中一些笔记。  
<!--more-->

# 安装
可以访问外网的话，参考 [官网](https://hub.docker.com/search?q=&type=edition&offering=community) 对应系统安装的说明操作即可。  

## CentOS离线安装
安装依赖包:  
```sh
yum install -y libcgroup container-selinux
```

在 [这里](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/) 下载`docker-ce`, `docker-ce-cli`, `containerd.io` 的rpm，然后通过rpm安装：  
```sh
yum remove -y runc

rpm -ivh containerd.io-*.rpm docker-ce-*.rpm
```
启动docker服务：  
```sh
systemctl start docker
systemctl enable docker
systemctl status docker
```

# 非root用户使用
默认情况下，非root用户需要sudo才能执行docker相关命令。  
加入docker组之后，可以不带sudo执行。  
```sh
groupadd docker
usermod -aG docker 你的用户名
```
需要重新登录或者重启电脑才生效。  

# 代理
如果机器设置了代理，那么docker也需要同步代理的设置。  
```sh
mkdir /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/http-proxy.conf
```
编辑 `http-proxy.conf` 文件，添加内容如下：  
```
[Service]    
Environment="HTTP_PROXY=http://dev-proxy.oa.com:8080/" "HTTPS_PROXY=http://dev-proxy.oa.com:8080/" "NO_PROXY=127.0.0.1,localhost"
```
`HTTP_PROXY` 和 `HTTPS_PROXY` 的值替换为公司的代理地址， `NO_PROXY` 添加不需要代理的地址。  
确认配置后重启docker服务：  
```sh
systemctl daemon-reload
# 确认配置的Environment
systemctl show --property=Environment docker
systemctl restart docker
```

# 私有docker仓库
如果需要拉取私有的docker仓库镜像，一般需要把仓库地址加入到 `insecure-registries` 。  
在 `/etc/docker/daemon.json` 文件(没有则新建)中加入以下内容：  
```json
{
  "insecure-registries" : ["mycompany.com:8080"]
}
```

# 日常使用
```sh
# 生成镜像
docker build --network=host -t xxx-mod-dev:v1 .
docker run -it --net=host --name xxxModDev -v /home/用户名:/home/用户名 xxx-mod-dev:v1 /bin/bash

# 容器中
cd /home/用户名

# 退出容器
exit

# 再次进入容器
docker start -i xxxModDev
```

# 权限管理
方法一是给用户添加sudo权限，方法二是在已启动容器以root身份执行命令。  
```sh
docker exec -it -u root 容器Id或名称 bash
```

# 参考链接
- [Cannot download Docker images behind a proxy](https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy) 
- [docker预编译二进制文件](https://download.docker.com/linux/static/stable/x86_64/) 
