---
title: "使用hashcat恢复密码"
date: 2020-11-07T14:14:23+08:00
lastmod: 2021-03-07T14:14:23+08:00
draft: false
tags: ["hashcat", "password", "recovery", "rar"]
categories: ["security", "tool"]
mathjax: false
---

本文主要记录如何使用hashcat和John the Ripper恢复密码。  
<!--more-->

# 前提条件
满足以下其中一个条件就有很大可能恢复密码：  
- 弱密码  
- 密码有规律  

hashcat支持恢复密码的情况很多(具体参考[这里](https://hashcat.net/hashcat/#features-algos))，常见的rar、zip、7z和office文件都是支持的。  

# 工具
- hashcat  
  下载 [hashcat](https://hashcat.net/hashcat/#download)。  
- John the Ripper  
  在 [官网](https://www.openwall.com/john) 搜索 `Windows binaries` 下载。  
- Catpasswd Convert (可选)  
  下载 [特征提取工具](https://www.catpasswd.com/assets/Catpasswd-Convert.exe)，不推荐使用`catpasswd.com`恢复密码服务。  

# 使用
建议在有独立显卡的Windows电脑上面运行工具，hashcat能够利用显卡提高恢复密码速度。  

## 提取特征码
使用 `John the Ripper` 或者 `Catpasswd Convert` 都可以，后者有界面。  
以rar为例，`rar2john.exe` 或者 `Catpasswd-Convert.exe` 可以提取到类似下面的特征码：  
```
$rar5$16$a96ee7558fdb214744397649e01e90e9$15$45a3579057c8845d696d2da2b86e4742$8$93748e7016fd8cdd
```
rar5代表rar 5.x版本，rar3代表rar 3.x版本，下面恢复密码的时候需要用到。  

## 恢复密码

```shell
hashcat [options]... hash|hashfile|hccapxfile [dictionary|mask|directory]

# -a 3 暴力破解, 0 字典攻击, 1 组合攻击
# -m 12500 rar3, 13000 rar5
# -1 自定义字符集1
# -i 密码长度自增(不确定密码长度时使用)
# -w 负载配置方式，3会提高负载和速度
# --status 自动更新进度

.\hashcat.exe -a 3 -m 13000 -1 ?l?d --status -w 3 test.rar.hash ?1?1?1?1?1?1
```
以上面的特征码为例，前缀是rar5所以指定 `-m 13000`。`test.rar.hash`是保存了特征码的文件。  
`-1 ?l?d`定义字符集1为小写字母和数字，`?1?1?1?1?1?1`表示密码由6个字符集1里面的字符组成。  

自定义字符集可以根据自己常用的字符以及内置的字符集组合使用(具体参考[这里](https://hashcat.net/wiki/doku.php?id=mask_attack#built-in_charsets))。  
内置字符集：  
- ?l = abcdefghijklmnopqrstuvwxyz
- ?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
- ?d = 0123456789
- ?s = «space»!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
- ?a = ?l?u?d?s
- ?b = 0x00 - 0xff

最多支持4个自定义字符集，通过 `-1`、`-2`、`-3`、`-4` 分别指定。  
不确定密码长度可以加上`-i`选项。`--increment-min`可以指定最小长度。  

# 耗时
耗时很大程度跟密码的强弱、规律性有关。显卡好的话也能提高恢复密码速度。  
1个特殊字符(如!)+4位数字+4位小写字母就比9位任意字符(数字、字母、特殊字符)容易恢复得多。  

