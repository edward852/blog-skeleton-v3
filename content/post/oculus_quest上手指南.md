---
title: "Oculus Quest上手指南"
date: 2019-09-01T22:50:08+08:00
lastmod: 2021-02-01T22:50:08+08:00
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
- 亚马逊海外购 (**推荐**)
- 淘宝
- 闲鱼收购
- 其它

大小的话看个人情况，喜欢玩游戏的可以买大容量版本，只是串流玩PC游戏的话64GB已经够用。  
现在通过 **亚马逊海外淘官方小程序** 购买还能免邮费、叠加优惠券(有15/60/90)，赶紧扫码购买吧！  
<div>
    <img src="/images/amazon_quest2_64gb_promo.png" width="45%" />
    <img src="/images/amazon_quest2_256gb_promo.png" width="45%" />
</div>

# 激活、配置设备

## 科学上网 {#FQ}
由于Oculus是Facebook公司的，因此需要科学上网才能进行激活、注册帐号、固件升级等操作。  
方法主要有下面几种：  
- 电脑科学上网 + SSTap + 共享热点  
  首先电脑打开热点，并且通过SS/SSR/v2ray/trojan任意一种客户端科学上网。  
  然后重点是SSTap创建的虚拟网卡共享给热点。  
  ![sstap share](/images/conn-sstap-share.png)   
  最后手机、Quest连接热点即可。  
  另外SSTap好久没更新了，能用是能用的，不过不喜欢的话还可以选择 [Netch](https://github.com/NetchX/Netch/releases) 。  
- FQ路由器  
  可以购买支持科学上网的路由器，比如说AC86U或者老毛子路由器。  
  AC86U可以刷梅林固件，然后在软件中心离线安装 [科学上网](https://github.com/hq450/fancyss_history_package/tree/master/fancyss_hnd) 软件。  
  科学上网软件需要设置成全局模式或者游戏模式。  
- Quest安装科学上网软件  
  这个当然要激活之后才能实现:sweat_smile:。通过SideQuest安装，然后开启全局模式。  
  节点可以通过[在线剪贴板](https://netcut.cn)或者文本文件导入。  

另外这篇 [Oculus Quest 国内玩家无障碍使用指南](http://www.vr42.com/t/187) 写得更全面一些。  

## Oculus app安装、运行
等设备到手之后，我们就可以下载手机端 `Oculus` 软件来激活设备。  
[Android版](https://play.google.com/store/apps/details?id=com.oculus.twilight)需要科学上网下载，而[IOS版](https://itunes.apple.com/us/app/oculus-vr/id1366478176)需要国外apple id才能下载。  
如果有Link数据线也可以通过PC端[OculusSetup](https://www.oculus.com/download_app/?id=1582076955407037)软件激活。  

安装之后，运行app就会引导配对和设置你的设备，按照提示做即可。  
另外建议使用国外邮箱注册Facebook帐号，否则可能收不到Oculus发送的验证码。  
一般来说，如果不能科学上网，就会出现访问不了商店、更新卡在50%、Wi-Fi连接不了等问题。  

# 软件安装
配置完成之后，就可以安装软件了。  
不过oculus商店的游戏比较贵，建议先下免费的试玩版体验，比如说 `Beat Saber`。  
收费游戏的话，土豪随意，普通玩家建议还是尽量在Steam购买，然后通过无线串流玩。  

## SideQuest {#sq}
`Quest`也是安卓系统，所以可以通过`apk`文件安装程序和游戏。  
根据这里的 [官方说明](https://sidequestvr.com/setup-howto) 安装SideQuest、完成配置即可。  
启用开发者模式可能要先设置支付方式，背面有3位数字安全码的银行卡应该都可以用来支付。  
实在不行还可以让别人拉你进开发者。  

# 串流
如果你有 `Steam VR Ready` 的电脑(主要是显卡要好，显存够大)，那么可以通过串流玩Steam的游戏。  
同样的一款游戏，Oculus商店一般都比Steam商店要贵不少，而Steam的画质更佳。。。  
不仅如此，还可以玩[VIVEPORT](https://www.viveport.com) 、[Oculus Rift](https://www.oculus.com/experiences/rift)的游戏。基本上支持VR模式的游戏都可以玩。  

## 无线串流
无线串流的软件有收费的 [Virtual Desktop](https://www.vrdesktop.net) 和免费开源的 [ALVR](https://github.com/polygraphene/ALVR)。  
建议还是 `Virtual Desktop` ， 效果确实是一分价钱一分货。  
Virtual Desktop串流主要有以下步骤：  
- PC安装 [Streamer App](https://download.vrdesktop.net/files/VirtualDesktop.Streamer.Setup.exe) 以及PC版[Oculus软件](https://www.oculus.com/download_app/?id=1582076955407037)。  
- PC在Oculus商店购买 **Oculus Quest** 版的Virtual Desktop (20美元，可以用WELCOME25优惠券)
- Oculus Quest通过 [SideQuest](#sq) 安装支持串流的Virtual Desktop版本

然后PC和Oculus Quest在同一个局域网即可无线串流。  
为了降低延迟，建议PC有线接入路由器，Oculus Quest接入5G Wi-Fi。  

目前体验的情况，延迟不是很大，个人觉得挺流畅的。Beat Saber至少可以流畅玩困难级别。  

## 有线串流
如果对延迟比较敏感，那么可以购买Oculus Link数据线进行有线串流。  
也可以买第三方的(参考[兼容性](https://support.oculus.com/444256562873335/)说明)，比如官方推荐的 [Anker](https://www.amazon.cn/gp/product/B01MZIPYPY)，可以在中亚海外淘直接购买。  
具体设置可以参考 [这里](https://support.oculus.com/525406631321134/#setup) 的说明。  

# 视频
可以直接通过内置浏览器在线看VR视频，也可以通过免费的`SKYBOX VR`视频播放器观看下载好的视频。  
特别赞的是，SKYBOX VR可以播放处于同一局域网内电脑上的视频，彻底释放Quest的存储空间！  
另外2K或者4K左右的视频观看效果会比较好(小姐姐更好看:blush:)，更低的看起来会有点模糊。  

# 游戏
不是重度游戏玩家，这里就简单推荐 `Half-Life: Alyx`、`VR Kanojo`以及`Beat Saber` 好了。  

# 固件升级
通过设置Quest里Wifi的HTTP代理实现科学上网的话大概率能激活，但是不能顺利升级固件版本。  
虽然可以[手动升级](https://www.reddit.com/r/oculusquest/wiki/guides/manualupdate) ，但是帖子里面的固件没有历史版本，不一定适合你的设备升级。  
比较靠谱的方法还是上面 [科学上网](#FQ) 部分说的，连接可以科学上网的Wifi进行升级。  

# FAQ
- 为什么访问不了Oculus商店？  
  请看 [科学上网](#FQ) 部分确认Oculus Quest可以科学上网
- 为什么更新进度卡在50%？  
  请看 [科学上网](#FQ) 部分确认Oculus Quest可以科学上网

还有其它问题的话可以在下面留言。  

