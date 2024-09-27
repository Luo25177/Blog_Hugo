---
title: "Hexo加载动画"
date: 2024-03-22 11:46:19
categories: "网站制作"
tags: ["Hexo"]
summary: "Hexo加载动画设置"
---

## 前言

hexo 默认是没有加载动画的，偶然在做博客时看到了这个教程，在此基础上做了一些小改动，效果很不错，所以记录下来

原文链接： [Hexo站点加载动画修改](https://cnhuazhu.top/butterfly/2021/02/25/Hexo%E9%AD%94%E6%94%B9/Hexo%E7%AB%99%E7%82%B9%E5%8A%A0%E8%BD%BD%E5%8A%A8%E7%94%BB%E4%BF%AE%E6%94%B9%EF%BC%88%E9%BD%BF%E8%BD%AE%E6%95%88%E6%9E%9C%EF%BC%89/)

## 加载动画设置

### 创建js文件

在 `themes\butterfly\layout\includes\loading` 文件夹下创建一个名为 `loading.ejs` 的文件，并且写入

```JS
<% if (theme.preloader.enable) { %>
<div id='loader'>
    <% if(theme.preloader.layout == 'gear' ) {%>
    <div class="outer_box">
    <div class='loader_overlay'></div>
    <div class='loader_cogs'>
        <div class='loader_cogs__top'>
            <div class='top_part'></div>
            <div class='top_part'></div>
            <div class='top_part'></div>
            <div class='top_hole'></div>
        </div>
        <div class='loader_cogs__left'>
            <div class='left_part'></div>
            <div class='left_part'></div>
            <div class='left_part'></div>
            <div class='left_hole'></div>
        </div>
        <div class='loader_cogs__bottom'>
            <div class='bottom_part'></div>
            <div class='bottom_part'></div>
            <div class='bottom_part'></div>
            <div class='bottom_hole'></div>
        </div>
        <p style="text-align:center">&nbsp;&nbsp;&nbsp;loading...</p>
    </div>
    </div>
    <% } else if(theme.preloader.layout == 'spinner-box') { %>
    <div class="loading-left-bg"></div>
    <div class="loading-right-bg"></div>
    <div class="spinner-box">
        <div class="configure-border-1">
            <div class="configure-core"></div>
        </div>
        <div class="configure-border-2">
            <div class="configure-core"></div>
        </div>
        <div class="loading-word">加载中...</div>
    </div>
    <% } %>
</div>
    
<script>
    var endLoading = function () {
    document.body.style.overflow = 'auto';
    document.getElementById('loader').classList.add("loading");
    }
    window.addEventListener('load',endLoading);
</script>
<% } %>
```

### 主题配置文件修改

打开主题配置文件，在 `inject` 的 `head` 处引入两个 `css` 文件

```YAML
inject:
  head:
    - <link rel="stylesheet" href="https://gcore.jsdelivr.net/gh/zyoushuo/Blog@latest/hexo/css/loading_style_1.css" > # spinner-box风格样式文件
    - <link rel="stylesheet" href="https://gcore.jsdelivr.net/gh/zyoushuo/Blog@latest/hexo/css/loading_style_2.css" > # gear风格样式文件
```

将代码

```YAML
preloader: true
```

修改为

```YAML
preloader:
  enable: true
  layout: gear # gear, spinner-box 两种样式可选
```

### 修改 layout 文件

打开 `\themes\butterfly\layout\includes\layout.pug` 文件，将

```PUG
if theme.preloader
  !=partial('includes/loading/loading', {}, {cache: true})
```

替换为

```PUG
if theme.preloader
  !=partial('includes/loading/loaded.ejs', {}, {cache: true})
```
