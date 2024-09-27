---
title: "Hexo标题旋转风车"
date: 2024-03-22 11:19:30
categories: "网站制作"
tags: ["ButterFly", "Hexo"]
summary: "Hexo设置小标题旋转风车"
---

### 前言

这个特效是看到了别人的博客之后学到的，很不错，可以看看原文 [Hexo小标题旋转风车设置](https://cnhuazhu.top/butterfly/2021/02/24/Hexo%E9%AD%94%E6%94%B9/Hexo%E5%B0%8F%E6%A0%87%E9%A2%98%E6%97%8B%E8%BD%AC%E9%A3%8E%E8%BD%A6%E8%AE%BE%E7%BD%AE/)

为了以后自己换电脑的话方便找，所以做一些记录

### 设置旋转风车

打开主题配置文件，查找 `beautify` 并且修改为如下代码

```YAML
beautify:
  enable: true
  title-prefix-icon: '\f863'
```

并且在 `inject` 的 `head` 处引入文件

```YAML
inject:
  head:
    - "<style>#article-container.post-content h1:before, h2:before, h3:before, h4:before, h5:before, h6:before { -webkit-animation: avatar_turn_around 1s linear infinite; -moz-animation: avatar_turn_around 1s linear infinite; -o-animation: avatar_turn_around 1s linear infinite; -ms-animation: avatar_turn_around 1s linear infinite; animation: avatar_turn_around 1s linear infinite; }</style>"
```
