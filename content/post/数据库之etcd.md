---
title: "数据库之etcd"
date: 2020-07-24T20:20:31+08:00
lastmod: 2021-02-24T20:20:31+08:00
draft: false
tags: ["etcd", "raft"]
categories: ["database"]
mathjax: false
---

本文主要介绍分布式键值数据库 [etcd](https://etcd.io)。etcd主要用于共享配置和服务发现。  
<!--more-->

# 安装
可以在 [这里](https://github.com/etcd-io/etcd/releases) 下载安装最新的版本。  
```shell
curl -LO $(curl -Ls https://api.github.com/repos/etcd-io/etcd/releases/latest | grep browser_ | cut -d\" -f4 | grep 'amd64\.tar\.gz')
tar xzf etcd-*-linux-amd64.tar.gz
cp etcd-*-linux-amd64/{etcd,etcdctl} /usr/local/bin/
rm -fr etcd-*-linux-amd64*

# 检查是否安装成功
etcd --version
etcdctl version

# 创建配置和数据文件夹
mkdir -p /etc/etcd
mkdir -p /var/lib/etcd

# 创建etcd组和用户
groupadd --system etcd
useradd -s /sbin/nologin --system -g etcd etcd
chown -R etcd:etcd /var/lib/etcd
```

# 开机启动
创建`/etc/systemd/system/etcd.service`文件，内容参考 [这里](https://github.com/etcd-io/etcd/blob/master/contrib/systemd/etcd.service)。  
注意`ExecStart`那里根据自己的情况修正，本文实际路径为`/usr/local/bin/etcd`。  
```shell
# 启动etcd服务
systemctl start etcd.service
systemctl status etcd.service

# 开机启动
systemctl enable etcd.service
```


# Raft一致性算法
具体参考 [这里](https://raft.github.io/)，以及 [动画演示](http://thesecretlivesofdata.com/raft/)。  

