---
title: "Docker使用笔记"
date: 2020-01-09T11:05:00+08:00
lastmod: 2020-01-09T11:05:00+08:00
draft: true
tags: ["docker"]
categories: ["note"]
mathjax: false
---

本文主要记录docker使用过程中一些笔记。  
<!--more-->

# 安装
可以访问外网的话，参考 [官网](https://hub.docker.com/search?q=&type=edition&offering=community) 对应系统安装的说明操作即可。  

## Linux离线安装
访问不了外网的话，可以在 [这里](https://download.docker.com/linux/static/stable/x86_64/) 下载编译好的版本安装。  
```sh
tar -xf docker-*-ce.tgz
sudo cp docker/* /usr/bin/
```

又或者在 [这里](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/) 下载rpm安装 `docker-ce docker-ce-cli containerd.io`。  

修改 `/etc/docker/daemon.json` 文件：  
```json
{
  "debug": true,
  "bridge": "",
  "iptables": false,
  "hosts": ["tcp://127.0.0.1:2375", "unix:///var/run/docker.sock"],
  "insecure-registries": [],
  "exec-opts": ["native.cgroupdriver=cgroupfs"],
  "labels": ["cluster=chong"],
  "graph": "/data/docker"
}
```
`insecure-registries` 可以加上公司内部的docker仓库。  

# 代理
```sh
mkdir /etc/systemd/system/docker.service
touch /etc/systemd/system/docker.service.d/http-proxy.conf
```
编辑 `http-proxy.conf` 文件，添加内容如下：  
```
[Service]    
Environment="HTTP_PROXY=http://dev-proxy.oa.com:8080/" "HTTPS_PROXY=http://dev-proxy.oa.com:8080/" "NO_PROXY=127.0.0.1,localhost"
```
`HTTP_PROXY` 和 `HTTPS_PROXY` 的值替换为公司的代理地址， `NO_PROXY` 可以添加不需要代理的地址。  
确认配置后重启docker服务：  
```sh
systemctl daemon-reload
# 确认配置的Environment
systemctl show --property=Environment docker
systemctl restart docker
```

# 日常使用
```sh
docker build --network=host -t xxx-mod-dev:v1 .
docker run -it --net=host --name xxxModDev -v /home/用户名:/home/用户名 xxx-mod-dev:v1 /bin/bash

# 容器中
cd /home/用户名

# 退出容器
exit

# 再次进入容器
docker start -i xxxModDev
```

# 参考链接
- [Cannot download Docker images behind a proxy](https://stackoverflow.com/questions/23111631/cannot-download-docker-images-behind-a-proxy) 
