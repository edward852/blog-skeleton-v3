---
title: "Hugo对Markdown支持情况测试"
date: 2020-02-21T15:00:28+08:00
lastmod: 2020-02-21T21:17:23+08:00
draft: false
tags: ["markdown", "GFM", "LaTex", "emoji", "PicGo"]
categories: ["language"]
mathjax: true
mathjaxEnableSingleDollar: true
---

本文主要内容是测试Hugo对Markdown的支持情况，方便后续写作时查阅。  
<!--more-->

# 标题
在行首以井号`#`加空格指定，几个井号就代表几级标题。

```markdown
# 1级标题
## 2级标题
...
###### 6级标题
```

# 段落
段落之间以至少一行空行为分隔，也可以通过在行尾加 **2个空格** 强制段落内换行。

段落一的第一行  
段落一的第二行  
...  
段落一的第N行

段落二的第一行

# 文本样式 {#字体样式}

字体样式常用的有粗体、斜体、删除线等。  
`*斜体字*` 对应 *斜体字*  
`**粗体字**` 对应 **粗体字**  
`~~删除线~~` 对应 ~~删除线~~  
`` `行内代码` `` 对应 `行内代码`  

还有一些样式原生不支持，但是可以通过html间接支持。  
`<u>下划线</u>` 对应 <u>下划线</u>  
`H<sub>2</sub>O` 对应 H<sub>2</sub>O  
`爆米<sup>TM</sup>` 对应 爆米<sup>TM</sup>


# 列表
列表分为有序列表和无序列表。  
以数字+ `.` 表示有序列表项：
1. 有序子项目1

以 `-` 或 `+` 表示无序列表项：
-   无序子项目

可以混合使用，注意同级列表以空格对齐。  
1. 有序子项目一  
    - 无序子项目 1
    - 无序子项目 2
2. 有序子项目二
    - 无序子项目 a
    - 无序子项目 b

# 图片
一般格式是 `![文字描述](链接)` ：  
![org-mode-logo](http://orgmode.org/img/org-mode-unicorn-logo.png)

这种格式显示图片不能控制对齐和大小，需要的话可以采取raw html的方法：  
```html
<p align="center">
    <img src="http://orgmode.org/img/org-mode-unicorn-logo.png" width="100" />
</p>
```
<p align="center">
    <img src="http://orgmode.org/img/org-mode-unicorn-logo.png" width="100" />
</p>

{{% admonition tip "tip" %}}
需要注意的是，Hugo 0.60以上版本需要设置markup.goldmark.renderer的unsafe值为true才支持
{{% /admonition %}}

链接可以是网络URL，也可以是相对路径。  
相对路径的话需要注意本地路径和(网站)部署路径可能不一致的问题。  
一般还是通过工具 (如 [PicGo](https://github.com/Molunerfinn/PicGo)) 上传到图床更方便。

# 链接 {#links}

链接分为外部链接和内部链接，格式都是 `[文字描述](链接)` 。  
外部链接就是我们常用的URL，这里不赘述。  
内部链接，也叫anchor，用来引用当前文档的某个位置。  
可以通过以下方式标记位置：  
```
{#target}
```

其中 `target` 需要取一个有意义的、文档内唯一的名字。  
然后通过以下方式引用，与html的锚类似：  
```
[文字描述](#target)
```

跳转回本章节起始位置☞ [back](#links) 。  


# 引用与代码

引用名人名言：
```markdown
> Stay Hungry. Stay Foolish -- Steve Jobs
```
> Stay Hungry. Stay Foolish -- Steve Jobs

引用代码：  
````
```语言(可以不填，但是没有语法高亮)
代码内容
```
````

C代码：  
````
```c
int sayHello()
{
    printf("Hello!\n"); // hello everyone
}
```
````
```c
int sayHello()
{
    printf("Hello!\n"); // hello everyone
}
```

# 表格

创建表格十分简便，基本上组合使用 `|` `-` 即可。  
对齐不是硬性要求，第一列 `|` 也可以省略。  
```
简写版本：
key | value
--- | ---
long key | long测试中文

美观版本：
| key      | value       |
|----------|-------------|
| long key | long测试中文 |
```

key | value
--- | ---
long key | long测试中文

另外在 [spacemacs](https://www.spacemacs.org/) 中使用 `Tab` 和 `Shift Tab` 可以自动对齐表格和在单元格内跳转，谁用谁知道 :smile:  
![](https://i.loli.net/2020/02/21/beFw7Af9GLijI4T.gif)

# 分隔线

3个以上横杠 `-` 单独为一行可以显示分隔线：  

---

# emoji表情
支持Github的emoji表情，格式为 `:emoji代码:` ，具体可以查阅 :point_right: [这里](https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md) 。

# 公式

比较考验排版能力的数学公式，通过LaTex表示毫无压力、逼格满满。  
正文显示公式： `$LaTex公式$` $\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$  
单独显示公式： `$$LaTex公式$$`  
$$\sum_{i=0}^N\int_{a}^{b}g(t,i)\text{d}t$$  

{{% admonition tip "tip" %}}
支持显示公式需要设置mathjax选项为true，支持单个$还需要设置mathjaxEnableSingleDollar为true。
{{% /admonition %}}

# 脚注Footnote

脚注主要有数字脚注[^1]和命名脚注[^org]。  
在文章末尾行首以下面的方式可以定义脚注：  
`[^数字]: 数字脚注内容`  
`[^名字]: 命名脚注内容`  
在文中通过 `[^数字]` 或者 `[^名字]` 引用即可。  
不过命名脚注显示在页面上还是数字脚注的形式。。。  

# 任务列表
任务列表是在无序列表的基础上扩展的，属于GFM标准。  
```
- [ ] 待测试
- [x] 已测试
```

- [ ] 待测试
- [x] 已测试

# 杂项
Even主题本身还支持一些 [Shortcodes](https://blog.olowolo.com/example-site/post/shortcodes/) 以及流程图、时序图。  
由于不是Markdown标准，所以这里就不测试了。  

[^1]: <http://orgmode.org/org.html#Footnotes>
[^org]: <http://orgmode.org/org.html>
