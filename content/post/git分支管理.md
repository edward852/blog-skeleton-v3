---
title: "一种Git分支管理方式"
date: 2019-05-25T21:06:51+08:00
lastmod: 2019-05-25T21:06:51+08:00
draft: false
tags: ["git", "branch"]
categories: ["tool"]
mathjax: false
---

本文主要介绍一种Git分支管理方法(非GitFlow, GitHub Flow)。  
<!--more-->

# 分支类型
- master 主干分支  
- feature/xxx 开发分支  
- release/xxx 发布分支  
- hotfix/xxx 紧急修复分支  

# 日常操作
## 主干分支
不允许直接提交。  

## 开发分支
研发同事从主干分支创建，并定期(每星期1~2次)rebase主干分支的变更(减少合入冲突)。  
开发完成之后，rebase主干分支后提MR/PR合入下期发布分支。  

## 发布分支
发布分支按规划从主干分支创建，部署到测试环境验证，验证通过后发布。  
测试发现的bug在发布分支修复。  
发布后没问题合入主干分支并打tag。  

## 紧急修复分支
紧急修复分支从主干分支创建，发布后没问题提MR/PR合入主干分支并打tag。  

