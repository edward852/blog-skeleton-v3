---
title: "Hugo添加Algolia搜索支持"
date: 2020-02-23T15:32:13+08:00
lastmod: 2020-02-23T15:32:13+08:00
draft: false
tags: ["Algolia", "Searching", "Hugo"]
categories: ["tutorial"]
mathjax: false
---

本文主要介绍如何在 [Hugo](https://gohugo.io) 中添加 [Algolia](https://www.algolia.com) 搜索支持。  
主要以 [Even主题](https://themes.gohugo.io/hugo-theme-even) 为例进行说明，不过理论上只需要小的调整即可适用于其他主题。  
<!--more-->

添加Algolia搜索支持主要有3大步骤：  
- 生成索引文件
- 上传索引文件
- 添加前端UI

下面会详细介绍每个步骤的操作，而且每一步都可以马上看到效果，确认是否操作正确。  

# 生成索引文件
## Algolia创建空索引

- 创建应用，自取名字(比如说blog)  
  ![](https://i.loli.net/2020/02/23/FpQDbZuhBNn671q.png)
- 创建索引，自取名字(比如说blog)  
  ![](https://i.loli.net/2020/02/23/ubxvpWh8wXYnIKd.png)
- 点击侧栏 `API Keys`，记录信息(Application ID, Search-Only API Key, Admin API Key)  
  ![](https://i.loli.net/2020/02/23/RoMfLZOBc8Dv3Fr.png)

## Hugo生成索引文件

- 修改 `config.toml` 文件声明需要生成Algolia索引文件以及相关信息
  ```toml
  [outputs]
      home = ["HTML", "RSS", "Algolia"]
  
  ...中间省略

  [outputFormats.Algolia]
    baseName = "algolia"
    isPlainText = true
    mediaType = "application/json"
    notAlternative = true
  
  [params.algolia]
    appId = "你的Application ID"
    indexName = "你的索引名字"
    searchOnlyKey = "你的Search-Only API Key"
  ```
  注意outputs下面home的末尾有`"Algolia"`，漏了就不会生成了。  
  `appId`, `indexName`, `searchOnlyKey` 填写刚才记录的Algolia信息。   

- 根目录下 `layouts/_default` (没有就新建) 文件夹中新建 `list.algolia.json` 文件，内容如下：  
  ```go
  {{/* 生成Algolia搜索索引文件 */}}
  {{- $.Scratch.Add "index" slice -}}
  {{/* content/posts或content/post目录下的博文才生成索引 */}}
  {{- range where (where .Site.Pages "Type" "in" (slice "posts" "post")) "IsPage" true -}}
    {{- if and (not .Draft) (not .Params.private) -}}
      {{- $.Scratch.Add "index" (dict "objectID" .File.UniqueID "url" .Permalink "content" (.Summary | plainify) "tags" .Params.Tags "lvl0" .Title "lvl1" .Params.Categories "lvl2" "摘要") -}}
    {{- end -}}
  {{- end -}}
  {{- $.Scratch.Get "index" | jsonify -}}
  ```
  如果你的博文没有categories，那么 `"lvl1"` 后面的 `.Params.Categories` 可以换成其他内容。  

- 执行 `hugo` 命令之后，在 `public` 目录下就会生成 `algolia.json` 文件

至此，如果生成了 `algolia.json` 文件且内容正常，那么就可以继续下一步。  
如果有问题，可以回头再仔细看看有没有什么地方漏了，也可以在下面评论反馈问题。  

# 上传索引文件
生成索引文件之后，我们需要上传到Algolia的服务器。  

## 手动上传
这一步是可选的，不过还是建议跟着做一下。  
点击侧栏 `Indices` ，点击 `Upload record(s)` 按钮上传上一步生成的 `algolia.json` 文件。  
![](https://i.loli.net/2020/02/23/ywmXf1G9idcrTK3.png)  
![](https://i.loli.net/2020/02/23/jeihUXDxBuwmpf9.png)  

上传成功之后，我们就可以马上尝试搜索了：
![](https://i.loli.net/2020/02/23/yGsPjqCQBeZVA8U.png)

可以看到搜索的关键词有相应的匹配结果，说明我们生成的索引文件是正确的。

## 自动上传
每次写完博文都手动上传索引文件无疑是痛苦的、无意义的重复劳动。  
因此我们需要把上传索引文件的操作自动化，在自动部署的时候顺便完成即可。  
这里我们采用npm包 [atomic-algolia](https://www.npmjs.com/package/atomic-algolia) 来完成上传操作。  
- 安装 atomic-algolia 包  
  ```
  npm init  // 不懂的就回车好了
  npm install atomic-algolia --save
  ```

- 修改目录下的 `package.json` 文件，在 `scripts` 下添加 `"algolia": "atomic-algolia"`
  ```json
  "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1",
      "algolia": "atomic-algolia"
  },
  ```
  注意 `"test"` 那一行末尾有个英文逗号，不要漏了。  

- 根目录下新建 `.env` 文件，内容如下：  
  ```
  ALGOLIA_APP_ID=你的Application ID
  ALGOLIA_INDEX_NAME=你的索引名字
  ALGOLIA_INDEX_FILE=public/algolia.json
  ALGOLIA_ADMIN_KEY=你的Admin API Key
  ```
  注意替换你自己Algolia索引的信息。  
  另外特别注意 `ALGOLIA_ADMIN_KEY` 可以用来管理你的索引，所以尽量不要提交到公共仓库。  
  一般的做法是在你的自动化部署工具中设置环境变量，比如说本站就是在 [Travis CI](https://travis-ci.org) 中设置环境变量。  

- 上传索引的命令  
  你可以本地执行 `npm run algolia` 查看运行效果：  
  ![](https://i.loli.net/2020/02/23/C6u38blAWG7Xnke.png)  
  可以看到我们成功添加了记录。  
  后续就是把下面的命令加到你的部署脚本即可：  
  ```
  npm install
  npm run algolia // 在hugo命令后面执行
  ```

至此，如果你运行 `npm run algolia` 没有报错的话，就可以继续下一步了。  
同样的，如果有问题欢迎在下面留言。  

# 添加前端UI
剩下的工作就是添加搜索框了，这一步是跟主题相关的，不过其他主题应该只需要小的调整就可以了。  

- 引入CSS文件  
  在主题 `layouts/partials` 目录下的 `head.html` 中引入docsearch的CSS文件  
  ```go
  {{- if .Site.Params.algolia.appId -}}
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/docsearch.js@2/dist/cdn/docsearch.min.css" />
  {{- end -}}
  ```

- 引入JS文件并初始化  
  在主题 `layouts/partials` 目录下的 `scripts.html` 中引入docsearch的JS文件并初始化  
  ```go
  {{- if .Site.Params.algolia.appId -}}
  <script src="https://cdn.jsdelivr.net/npm/docsearch.js@2/dist/cdn/docsearch.min.js"></script>
  <script>
      docsearch({
      apiKey: {{ .Site.Params.algolia.searchOnlyKey }},
      indexName: {{ .Site.Params.algolia.indexName }},
      appId: {{ .Site.Params.algolia.appId }},
      inputSelector: '.docsearch-input',
      debug: false,
      });
  </script>
  {{- end -}}
  ```

- 添加搜索框入口  
  在主题 `layouts/partials` 目录下的 `header.html` 中导航栏合适的位置添加搜索框  
  ```go
  {{- if .Site.Params.algolia.appId -}}
    <li style="display:inline-block;margin-right:10px;">
      <input type="search" class="docsearch-input" placeholder="Search" />
    </li>
  {{- end -}}
  ```
  主要是添加 `<input>` 元素，`<li>` 元素根据各自主题的情况确定是否添加。  
  另外注意外层如果有 `overflow:hidden` 的CSS样式，可能会导致搜索结果框显示不了。:sweat_smile:   

你使用的主题不一定有上面的 `head.html`, `scripts.html`, `header.html` 文件。  
不过没关系，主题布局是类似的，肯定可以找到对应的文件(可能分散在2个文件，甚至在同一个文件)。  
如果实在找不到，可以在下面留言具体的Hugo主题，我后面有空可以帮忙找下。  

好了，话不多说，完成上述步骤之后，千呼万唤的Algolia搜索来了！:clap: :clap: :clap:   
![](https://i.loli.net/2020/02/23/QVpFEs7Bg8PZYic.gif)  

# 补充说明
其实添加前端UI也可以通过Algolia的 [autocomplete.js](https://github.com/algolia/autocomplete.js) ，定制性更强，适合前端高手使用。

# 参考教程
- [采用 Algolia 作为 Hugo 搜索方案](https://10101.io/2018/11/23/search-with-algolia-in-hugo)
- [Add Algolia Search To Hugo Static Website](https://code.luasoftware.com/tutorials/algolia/add-algolia-search-to-hugo-static-website/)
- [Static site search with Hugo + Algolia](https://forestry.io/blog/search-with-algolia-in-hugo/)
- [Static Site Search Made Easy (with Hugo Tutorial)](https://snipcart.com/blog/static-site-search)
