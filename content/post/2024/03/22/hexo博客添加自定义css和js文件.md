---
title: "Hexo博客添加自定义css和js文件"

date: 2024-03-22 10:51:16
categories: "网站制作"
tags: ["Hexo"]
summary: "Hexo博客添加自定义css和js文件"
---

### 创建文件

进入到主题目录资源文件夹下，也就是 `themes\butterfly\source` 文件夹下，然后就会看到三个文件夹 `css` ， `img` ， `js` ， `css` 和 `js` 文件就可以建在这里

当然这样的话会导致主题更新的时候会把你的文件更新掉，所以可以在根目录下创建 `css` 和 `js` 文件夹，并且把对应的文件放在这里面

在引用文件时要注意，一般来说是在主题配置文件里的 inject 引入。 `css` 一般在 `head` 引入，而 `js` 文件一般在 `bottom` 中引入，当然某些特殊情况除外

### 使用文件

```YAML
inject:
  head:
    # 自定义css
    - <link rel="stylesheet" href="/css/style.css?1">
    # 静态文件后面加个 ?任意内容  可以在每次更新之后用户自动重新请求.
    # 例如添加 ?1 ,在修改此文件后改成 ?2 ,用户访问你的网站时,不会使用本地的缓存,而是请求新的内容。没修改的话就不用动。
  bottom:
    # 自定义js
    - <script src="/js/script.js?1"></script>
    # 引入多个文件就直接往下复制一行改一下文件名即可，如下：
    - <script src="/js/script1.js?1"></script>
    - <script src="/js/script2.js?1"></script>
```
