---
title: "博客自动部署从Travis CI迁移到GitHub Actions"
date: 2021-02-02T16:39:09+08:00
lastmod: 2021-02-03T16:39:09+08:00
draft: false 
tags: ["blog", "Hugo", "Travis CI", "GitHub Actions"]
categories: ["essay", "CI"]
mathjax: false
---

由于 [Travis CI](https://www.travis-ci.org) 部署排队时间长达一个小时，所以迁移到 [GitHub Actions](https://docs.github.com/en/actions) ，顺便更新下Hugo和Even主题。  
<!--more-->

# 背景
最近更新博客，发现Travis CI自动部署排队时间竟然长度一个小时，而实际执行只要1分多钟。  
这个可能跟Travis CI正在迁移域名有关，不过也不知要弄到什么时候，还是迁移到GitHub Actions吧。  

# 问题记录
GitHub Actions本身还是挺简单的，跟Travis CI大同小异，这里就简单记录下遇到的问题。  

- Checkout代码  
  默认不拉子模块代码，通过`submodules`选项控制拉取的方式。  
- 触发部署  
  默认是push的时候触发，个人觉得通过`workflow_dispatch`事件手动触发更方便调试。  
- GitHub Pages部署  
  没有官方的action支持部署到GitHub Pages，可以使用 [第三方action](https://github.com/JamesIves/github-pages-deploy-action) 。  
- Hugo版本  
  Even主题新版本需要扩展版本的Hugo，修改主题不用yarn重新生成了。  

有需要的话可以参考本站的 `.github/workflows/blog-skeleton.yml` 文件。  

# 后记
迁移之后，自动部署基本上不用等待了，执行时间也比之前快了十几秒。 :thumbsup: :thumbsup: :thumbsup:  
希望GitHub能够继续保持下去。:pray: :pray: :pray:  

