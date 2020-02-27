---
title: "Oculus Quest上手指南"
date: 2019-09-01T22:50:08+08:00
lastmod: 2019-09-01T22:50:08+08:00
draft: false
tags: ["oculus", "quest"]
categories: ["vr", "fun"]
mathjax: false
---

本文主要介绍如何上手 [Oculus Quest](https://www.oculus.com/quest) 并提供一些共性问题的解决方法。  
<!--more-->

# 购买
首先，不会科学上网的建议不要买，否则折腾半天都激活不了，最后只能降价挂闲鱼。  
OK，如果没问题，那么我们就可以从以下途径购买了：  
- 亚马逊海外购
- 淘宝
- 闲鱼收购
- 其它

# 激活、配置设备

## 科学上网 {#FQ}
由于oculus是Facebook公司的，因此需要科学上网才能进行后续的激活操作。  
方法主要有下面几种：  
- 改DNS  
  可以参考"虚幻岛"公众号的教程设置DNS，不过大概率不行 :joy:  
- SS + SSTap + 共享热点  
  主要思路是电脑(SS)科学上网，SSTap代理所有流量，再通过热点分享给手机和Oculus Quest  
- FQ路由器  
  可以购买支持科学上网的路由器，比如说AC86U或者老毛子路由器  

这里建议使用FQ路由器的方法。  
AC86U可以刷梅林固件，然后在软件中心离线安装 [科学上网](https://github.com/hq450/fancyss_history_package/tree/master/fancyss_hnd) 软件。  

## oculus app安装、运行
等设备到手之后，我们就可以下载手机端 `oculus` 软件来激活设备。  
android需要科学上网下载，而ios需要国外apple id才能下载。  
如果不会弄，也可以关注"虚幻岛"公众号下载安装。  

安装之后，运行app就会引导配对和设置你的设备，按照提示做即可。  
一般来说，如果不能科学上网，就会出现访问不了商店、更新卡在50%等问题。  

# 软件安装
配置完成之后，就可以安装软件了。  
不过oculus商店的游戏比较贵，建议先下免费的试玩版体验，比如说 `Beat Saber`。  
收费游戏的话，土豪随意，普通玩家建议还是尽量在Steam购买，然后通过无线串流玩。  

# 无线串流
如果你有一台 `Steam VR Ready` 的电脑(主要是显卡要好)，那么可以通过无线串流的方法玩Steam的游戏。  
同样的一款游戏，Oculus商店一般都比Steam商店要贵不少，而Steam的画质更佳。。。  
无线串流的软件有收费的 [Virtual Desktop](https://www.vrdesktop.net) 和免费开源的 [ALVR](https://github.com/polygraphene/ALVR)。  
建议还是 `Virtual Desktop` ， 效果确实是一分价钱一分货。  
Virtual Desktop串流主要有以下步骤：  
- PC安装 `Streamer App`
- PC在Oculus商店购买 **Oculus Quest** 版的Virtual Desktop (20美元)
- Oculus Quest通过 [SideQuest](https://sidequestvr.com) 安装支持串流的Virtual Desktop版本

然后PC和Oculus Quest在同一个局域网即可无线串流。  
为了降低延迟，建议PC有线接入路由器，Oculus Quest接入5G Wifi。  
另外Virtual Desktop每次启动的时候都会检查是否正版(Oculus商店已购买)，因此也需要科学上网。。。

目前体验的情况，延迟不是很大，反正我觉得挺流畅的。  
如果对延迟比较敏感，那么可以购买Oculus Link进行有线串流。  

# FAQ
- 为什么访问不了Oculus商店？  
  请看 [科学上网](#FQ) 部分确认Oculus Quest可以科学上网
- 为什么更新进度卡在50%？  
  请看 [科学上网](#FQ) 部分确认Oculus Quest可以科学上网
- Oculus Quest怎么更新固件版本?  
  旧版本的需要连接到Wifi并接入电源，待机一段不确定的时间就会自动下载更新。  
  新版本的话可以在设置那里直接点击更新按钮。  

