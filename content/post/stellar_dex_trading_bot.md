---
title: "Stellar DEX交易机器人"
date: 2021-03-03T21:20:08+08:00
lastmod: 2021-03-03T21:20:08+08:00
draft: false
tags: ["stellar", "xlm", "dex", "bot"]
categories: ["fun", "cryptocurrency"]
mathjax: false
---

本文主要讲述一个玩具性质的Stellar DEX交易机器人的实现思路。  
<!--more-->

# 背景
只是一个不开源、玩具性质、大概率亏钱的交易机器人实现思路:joy:。  
Stellar官方有一个免费、开源的 [Kelp](https://github.com/stellar/kelp) 做市机器人，有需要的可以自行收藏、使用。  

# 参考文档
Stellar官方[说明文档](https://developers.stellar.org/docs)以及[API文档](https://developers.stellar.org/api)。
Go SDK参考[Horizonclient](https://pkg.go.dev/github.com/stellar/go/clients/horizonclient)和[txnbuild](https://pkg.go.dev/github.com/stellar/go/txnbuild)。  

# 交易策略
简单来说就是定期调整+追跌杀涨的反套路交易策略。  
想法来源于香农投资手法，定期均衡资产(股票和现金)，在股价上涨时卖出、下跌时买入。  
听说香农投资手法由于交易费较贵在现实股市不太可行，还不如买指数基金。  
Stellar网络本身使用费用基本可以忽略不记，所以尝试使用该策略(变种)在DEX上面自动交易。  

# 实现
## 私钥安全
因为加密货币由私钥保管，所以代码和配置中不能明文保存。  
私钥经过加密、转换后分开几部分保存。  
另外可执行文件去掉调试信息和符号表。  

## 趋势判断
通过多个周期MA判断，有一定的滞后性。  

## 定期均衡
上升期降低加密货币占比，下降期提高加密货币占比，其余时候1:1分配。  
这里也是反套路，不适合持续上升、持续下降的情况。  

## 追跌杀涨
涨得多卖出，跌得多买入，提前根据收市价设置多级买单和卖单。  

## 杂项
- 交易额控制  
  买单和卖单控制交易额，避免引起不必要的波动。  
- 总额限制  
  交易机器人帐号总额达到一定程度会转账部分盈利到硬件钱包。  

# 后记
这个交易机器人大概率是亏钱的，不过还是交由时间和币市来验证吧~  
后面不定期更新，看看实际效果。  

