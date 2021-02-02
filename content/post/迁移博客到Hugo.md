---
title: "迁移博客到Hugo"
date: 2020-02-21T11:58:18+08:00
lastmod: 2021-02-02T19:10:18+08:00
draft: false
tags: ["blog", "Hugo", "Travis CI", "Markdown", "Netlify", "Algolia", "GitHub Actions"]
categories: ["essay", "CI"]
mathjax: false
---

有段时间没折腾博客了 :joy:，之前很多都是纯手撸的，现在忘的差不多了 :joy:。  
虽然可以重新温习一下再加点功能，但是感觉还是太折腾、太浪费时间了。  
网上又调查了一下 **SSG** 的新动态，经过一番对比最后敲定用 [Hugo](https://zilongshanren.com/post/move-from-hexo-to-hugo/
)  + [Even主题](https://themes.gohugo.io/hugo-theme-even/) 。
<!--more-->

# 为什么迁移
- 通过配置获取更多功能  
  简单的配置即可维持之前博客的功能，更好维护
- 生成更快  
  Hugo使用Go语言，生成速度更快
- 切换 org mode 为 Markdown  
  Markdown更为主流，编辑器支持情况更好
- 多端适配更好  
  针对手机、PC、Pad都有相应适配
- 支持emoji :thumbsup:

# 如何迁移

- 安装Hugo以及Even主题
- 配置 `config.toml`
- 通过 [Pandoc](https://pandoc.org/) 转换原博文为md格式，调整Front Matter
- 通过 [GitHub Actions](https://docs.github.com/en/actions) 部署

# 部署
如果不涉及主题的修改，其实通过 [Netlify](https://app.netlify.com/) 部署更简单，基本上一路点击即可。  
~~不过如果你想调整主题或者做些定制操作，那么还是用Travis CI部署更好。~~  
~~具体参考本站的 `.travis.yml` 文件即可。~~  
通过 [GitHub Actions](https://docs.github.com/en/actions) 部署，具体参考本站的 `blog-skeleton.yml` 文件。  

# 添加搜索支持
添加搜索支持麻烦一些，因为Even主题不支持 :sweat_smile: 。  
Hugo本身推荐了一些方法，本站采用 [Algolia](https://www.algolia.com/) 支持站内搜索。  
主要有以下步骤：
- 生成索引文件  
  通过Hugo的 [Template](https://gohugo.io/templates/) 生成
- 上传索引文件  
  主要通过 [atomic-algolia](https://www.npmjs.com/package/atomic-algolia) 上传
- 添加前端UI  
  通过 [docsearch.min.js](https://github.com/algolia/docsearch) 生成界面

具体可以参考 [Hugo添加Algolia搜索支持](/post/hugo添加algolia搜索支持) 的说明。  

# 杂项
相对以前的博客，还有些新的小调整，修改 `config.toml` 文件即可。
- 评论系统切换为utterances
- 开启流量统计busuanzi
- 开启Google Analytics网站分析

# 写博客流程
- 新建md文件，使用任意编辑器撰写  
  Vscode、Typora、在线Markdown编辑器都可以。  
  ```bash
  hugo new post/xxx.md
  ```
- 本地查看效果  
  ```bash
  hugo serve -D
  ```
- 修改draft为false后提交  
  默认是草稿，修改draft为false之后提交Git仓库就可以触发Travis CI自动部署了。  
  休息一下，倒杯茶喝，然后部署完成 :v:

# 后记
上面的迁移步骤其实写得比较简略，有不懂的可以在下面留言，我再补充一下。  
