---
title: "通过iframe在博客园cnblogs转载博文"
date: 2017-11-16T15:26:28+08:00
lastmod: 2020-03-19T15:26:28+08:00
draft: false
tags: ["blog"]
categories: ["trick"]
---

本文主要介绍如何通过iframe在 [博客园](https://www.cnblogs.com/) 转载博文。  
<!--more-->

**需要注意的是，该方法如果广泛使用可能会被博客园封堵，请低调地使用。**  

# 背景

博客园目前只能通过手动复制粘贴的方式转载博文，那么有没有更方便的方法？  
通过iframe转载博文是否可以?  

处于草稿状态的博文可以，但是一旦发布就会发现iframe不见了 :joy: 。  
由此猜测博客园后台有自动删除iframe元素的机制。  

# 原理

怎么绕过博客园删除iframe元素的机制？  
通过js插入iframe元素是否可以？  

测试发现，正常情况下js插入iframe元素的代码会被过滤掉。  
接着测试发现，含有`<iframe`和`iframe>`的部分会被过滤掉。  

那么拼接出来的iframe元素是否可以插入？ 幸运的是，答案是肯定的。  
于是解决方法出来了：通过js拼接出iframe元素来转载博文。  

# 步骤

## 申请开通js权限

默认情况下博文页面并不能执行js，需要在 `设置` 里面申请开通js权限。  

## 拼接出iframe元素并插入

可以 `'<' + 'iframe'` ，也可以 `'<i' + 'frame'`，甚至可以通过编码、解码得到完整的`<iframe`。  
而iframe的src属性从博文中元素提取，比如说 `<div id="blog-main-div" src="转载博文链接" >` 。  

参考代码如下，大家可以根据自己的情况调整：  
```html
<script type="text/javascript" src="https://cdn.bootcss.com/iframe-resizer/4.2.10/iframeResizer.min.js"></script>
<script type="text/javascript">
try
{

    let trick = 'iframe style="width: 100%;" src="xxxx" frameborder="0" scrolling="no"></iframe';
    let content = '<'+trick+'>';
    let elem = $('#blog-main-div');
    let srcUrl = elem.attr('src');

    content = content.replace('xxxx', srcUrl);

    console.log('src url:', srcUrl);
    console.log('Try to append', content);
    elem.append(content);
    iFrameResize({
        log:true
    });		
}
catch (err)
{
    console.log('Failed to add iframe');
}
</script>
```
iframe-resizer不是必要的，主要用于根据转载博文的长度自动调整iframe大小。  
iframe-resizer需要嵌入方和被嵌入方网站的支持才行(需要添加js代码)，适合转载个人博客的情况使用。  
转载其他网站博文的话，iframe可以固定一个大小，scrolling改为auto。  

上面弄好之后，后面转载博文参考下面模板使用即可：  
```html
<div id="blog-main-div" src="转载博文链接"></div>
原文地址：<a href="转载博文链接"></a>
```

如果有什么疑问的话，欢迎在下面留言。  
