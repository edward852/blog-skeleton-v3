---
title: "Foxmail加DavMail访问Exchange邮箱"
date: 2017-09-07T21:17:28+08:00
draft: false
toc: true
tags: ["foxmail", "davmail", "exchange"]
categories: ["mail"]
---

由于公司邮箱服务器改用Exchange，而Windows XP的Outlook实在太卡，所以只能寻找替代方案。
<!--more-->
方案一是Thunderbird+ExQuilla插件，效果差强人意，比Outlook要好一点。  
方案一还有个缺点就是ExQuilla插件只能免费使用一段时间，之后每年都需要购买license。。。  
另外就是更新Foxmail到支持Exchange的版本，不过实测还是不行(可能跟邮箱服务器设置有关)。  
后面在网上经过一番查找，终于找到了方案二：Foxmail+DavMail。  
Foxmail并不是必要的，任何其他支持POP、IMAP、SMTP的邮箱客户端都可以。  
方案二的核心是DavMail，作为中介连接邮箱客户端和邮箱服务器。  

# DavMail
## 安装
DavMail的安装比较简单，自己到
[官方下载页面](http://davmail.sourceforge.net/download.html)
下载安装文件然后安装即可。

## 配置
如下图配置即可：  
![](/images/170907_DavMailCfg.png)  
其中OWA(Exchange) URL一栏填入公司邮箱服务器的OWA URL： https://mail.xxx.com.cn/owa  
如果点击Save按钮保存配置报错说无法绑定端口的话，则自行选择 **没有被占用** 的端口号。  
设置DavMail开机启动：开始菜单->所有程序->启动->右键选择打开，放入DavMail的快捷方式。  

# Foxmail
## 安装最新版本（可选）
如前面所述，安装Foxmail并不是必要的，大家可以选择自己常用的邮箱客户端，配置方法是类似的。  
下文以新版Foxmail为例说明配置方法。

## 配置
- 新建帐号  
如下图所示选择帐号管理菜单项：  
![](/images/170907_FoxmailCfg_menu.png)  
选择新建->手动设置，如下图配置：  
![](/images/170907_FoxmailCfg_account.png)  
图中的邮件帐号根据实际情况填写，端口号要与上面DavMail配置的 **保持一致** 。  

- 导入通讯录  
如下图所示切换到通讯录标签页：  
![](/images/170907_FoxmailCfg_tab.png)  
选择导入->CSV文件，然后按照提示选择contacts.csv文件（后面会说明如何提取通讯录）导入即可：  
![](/images/170907_FoxmailCfg_contacts.png)  
至此配置结束，enjoy!!!  

# 通讯录提取与更新(可选)
通讯录根据需要，可以每半年或者一年更新一次，毕竟常联系的也就那么几个。  
通过Address Magic可以导出通讯录，关键操作如下：  

- 源格式选择GAL：  
  ![](/images/170907_AddrMagic_src.png)
- 目标格式选择CSV：  
  ![](/images/170907_AddrMagic_dst.png)
- 导出columns按照下图选择：  
  ![](/images/170907_AddrMagic_col.png)

在导出的CSV文件第一行前面插入新的一行，填入三列内容分别为姓名、电子邮件地址、部门。  
保存文件后通过前面所述的方法打开Foxmail导入通讯录即可。
