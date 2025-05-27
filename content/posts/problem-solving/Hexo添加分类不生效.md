---
title: Hexo添加分类不生效
date: 2024-09-06
categories:
  - Technology
tags:
  - ProblemSloving
nolastmod: true
cover: /img/pic-16.jpg
---
## 步骤
1. `hexo new page categories`
   生成`source/categories/index.md`
2. 在新生成的index.md中front-matter添加type: categories
3. 在_config.yml中开启，搜索menu，去掉categories前的#
4. 在博客的front-matter中添加categories: 任意名称

## 附录
1. next theme已经写好了categories的样式，在配置中去掉#是为了开启它，于是new page必须是categories，来配合配置中的路径，同理在配置中的其他选项如果要用它，也要new对应名称的page，比如tags，就是`hexo new page tags`。
2. 每篇博客categories只能有一个，tags可以有多个。
3. 我自己在配置的时候使用vscode下的终端运行的hexo command，会出现改完categories后不生效的情况，换成本地终端就解决了，不是很懂。