---
title: "SSH免密登录"
date: 2018-04-21T17:27:20+08:00
lastmod: 2021-02-01T17:27:20+08:00
draft: false
tags: ["ssh", "password", "login"]
categories: ["tutorial"]
mathjax: false
---

本文主要介绍SSH免密登录服务器的配置步骤。  
<!--more-->

# 生成密钥对
```sh
ssh-keygen -t rsa
```
默认私钥文件名 `id_rsa` , 可以指定其他名字，不过后续ssh的时候需要加上 `-i` 选项。  
`passphrase` 留空即可，否则ssh的时候还需要输入passphrase。  
这一步会生成私钥和公钥文件，公钥文件名一般是私钥文件名基础上再加 `.pub` 。  

# 添加公钥到服务器
```sh
ssh-copy-id [可选选项] 用户名@服务器IP地址
# 可选选项
# -i 指定公钥文件路径，默认是 ~/.ssh/id_rsa.pub
# -p 指定SSH端口号，默认是22
```
这一步主要是把上一步生成的公钥加入到服务器的 `authorized_keys` 文件中。  

# 登录服务器
```sh
ssh [可选选项] 用户名@服务器IP地址 
# 可选选项
# -i 指定私钥文件路径，默认是 ~/.ssh/id_rsa
# -p 指定SSH端口号，默认是22
```
正常的话，我们现在就可以通过上面的命令免密ssh到服务器了。  
不过每次都输入这么长是不是有点麻烦？ 没关系，我们可以配置服务器别名。  
在 `~/.ssh/config` (没有就新建，注意权限是600) 中加上下面的配置：  
```
Host 服务器别名
  HostName 机器IP地址
  User 用户名
  Port 22
  IdentityFile 私钥文件路径
```
然后我们就可以直接ssh服务器别名登录了。 :tada: :tada: :tada:  
```
ssh 服务器别名
```
当然使用一些ssh工具也是可以的，比如说 `xshell`, `PuTTY`，网上有教程这里就不赘述了 。  

# 问题排查
- 服务器sshd配置  
  ```
  PubkeyAuthentication yes
  AuthorizedKeysFile .ssh/authorized_keys
  ```
  检查一下 `/etc/ssh/sshd_config` 中上述两项配置是否正常。  
  `PubkeyAuthentication` 默认是开启的，所以只要不是 `no` 即可。  
  `AuthorizedKeysFile` 检查一下路径是否正确，对应文件中是否有你的公钥。  
  如果有问题，修正之后重启sshd服务后生效。  

- 防火墙  
  一般来说登录不了，防火墙的嫌疑比较大，包括服务器本身的防火墙以及国内的GFW。  
  ```
  traceroute -T -p 端口 IP地址
  ```
  可以通过 `traceroute` 测试连通性，又或者在服务器抓包确认。  

