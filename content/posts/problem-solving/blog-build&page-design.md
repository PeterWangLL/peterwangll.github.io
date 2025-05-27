---
title: Hugo搭建博客&Home Page设计
date: 2024-10-27
categories:
  - Technology
tags:
  - ProblemSloving
nolastmod: true
cover: /img/pic-22.jpg
---
### 博客搭建
之前博客是使用Hexo搭建的，比较普通，正好听说了Hugo，就来试试。
### Quick Start
根据Hugo官方文档的<a herf="https://gohugo.io/getting-started/quick-start/">Quick Start</a>有一个大概了解，选择一个喜欢的主题，我选择的是<a herf="https://g1en.site/hugo-theme-dream">Dream</a>，文档写的很详细。

下载dream主题放到themes文件夹下，修改hugo.toml文件，添加theme = "dream"。

`hugo server`启动本地服务调试，然后`hugo`生成到`public`文件夹下，再`push`到`GitHub`上，通过`GitHub Pages`部署。
### Tips
图片统一放到public/img下，然后在文章中使用cover = /img/pic.jpg来引用，<a href="https://pixabay.com">素材</a>。
### My Config
文章写在content/posts文件夹下，可以再通过文件夹分类，我是根据categories分类的，这是我每篇文章的开头设置：
```txt
title: Hugo搭建博客
date: 2024-10-27 ｜ 我不喜欢精确到秒，所以就只精确到天
categories:
  - Technology
tags:
  - Atag ｜ tags可以有多个
  - Btag
nolastmod: true ｜ 不显示更新时间
cover: /img/pic-22.jpg ｜ 封面图片，这里的路径是public下的img
```
下面是我的hugo.toml文件：
```txt
baseurl = "https://peterwangll.github.io/"
defaultContentLanguage = "en"
languageCode = "en"
title = "PeterWang's Blog"
theme = "dream"
[params]
zenMode = true | dream有两种模式，default和zen
lightTheme = "emerald"
darkTheme = "forest"
author = "Peter Wang"
description = "an interesting soul"
avatar = "img/avator.jpg"
headerTitle = "PeterWang's Blog"
motto = "No time to sorrow, we're building tomorrow"
footerBottomText = "" | 默认是powered by hugo。。。
直接在这里填的话文字没有效果，可以全局搜powered by。。然后改源码
email = "18136892032@163.com"
siteStartYear = 2024
# Syntax highlighting
customSyntaxHighlighting = true | 语法高亮，文档讲的很详细，
需要下载css到hugo的assets/css下，而不是dream的
noDefaultSummaryCover = true
showTableOfContents = true
# showSummaryCoverInPost = true ｜ 在文章最上方显示图片
showPrevNextPost = true
[markup]
[markup.goldmark]
[markup.goldmark.renderer]
unsafe = true ｜ hugo的bug，设置后可以在markdown中加载html标签

[markup.highlight]
noClasses = false ｜ 语法高亮设置
```
### Home Page Design
通过<a herf="https://github.com/abhisheknaiidu/awesome-github-profile-readme?tab=readme-ov-file">awesome-github-profile-readme</a>收集的别人的主页，寻找灵感，我的主页设计主要参考了以下两位大佬：<a herf="https://github.com/thmsgbrt/thmsgbrt?tab=readme-ov-file#things-i-code-with">thmsgbrt</a>和<a herf="https://github.com/SP-XD/SP-XD?tab=readme-ov-file">SP-XD</a>

Tech icon是通过<a herf="https://shields.io/">shields</a>生成的，
图标从<a herf="https://simpleicons.org/">simpleicons</a>获取。