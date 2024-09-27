---
title: "ButterFly公式显示问题"
date: 2024-03-22 15:36:35
categories: "网站制作"
tags: ["ButterFly", "Hexo"]
summary: "解决ButterFly公式显示问题"
---

## 公式渲染

首先需要换一个渲染器，要先把安装的渲染器卸载，然后安装 `hexo-renderer-markdown-it-katex` ，在根目录打开终端输入指令

```shell
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it-katex --save
```

然后在站点配置文件添加下列内容

```YAML
markdown:
  render:
    html: true
    xhtmlOut: false
    breaks: true
    linkify: true
    typographer: true
  plugins:
  anchors:
    level: 1
    collisionSuffix: ''
```

如果在之前配置过 `MathJax` 的话，在主题配置文件中找到，并修改为如下

```YAML
# MathJax
mathjax:
  enable: false
  per_page: false

# KaTeX
katex:
  enable: true
  per_page: false
  hide_scrollbar: true
```

然后刷新预览就可以看到效果了

## 公式过长

上述完成公式渲染之后，会出现公式过长超过页面的效果，查找了很多都没有处理这种问题的，偶然间看到有处理 `NexT` 主题中的公式过长的效果的，说是对于公示的渲染是有一个特定的标签的，然后就开始了我的探索

### 查找解决方向

首先打开自己的博客，打开一篇带有公式渲染的博客，然后右键打开检查，然后找到公式块

![1.png](./1.png)

然后就发现，当给 `katex-display` 添加图中属性之后，公式块中就出现了滑动条，并且公式也不再超出页面了，找到目标了

### 解决问题

新建文件 `themes\butterfly\source\css\formuladisplay.css` 并且在文件中写入下列内容
  
```CSS
.katex-display{
  width: 100%;
  overflow-x: auto;
  overflow-y: hidden;
}
```

然后打开主题配置文件，在 `inject` 的 `head` 处添加对该文件的引用

```YAML
inject:
  head:
    - <link rel="stylesheet" href="/css/formuladisplay.css">
```

最后刷新预览就可以看到效果了。当然这中寻找问题的方法是之前在学爬虫的时候学到的方法，让我这个几乎不懂网站前端的人也能把网页做好
