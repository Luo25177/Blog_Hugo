---
title: "ButterFly导航栏居中"
date: 2024-03-22 09:08:36
categories: "网站制作"
tags: ["ButterFly", "Hexo"]
summary: "ButterFly导航栏居中解决方案"
---

## 前言

之前样式的导航栏实际上倒也还好，只是在查找 ButterFly 美化时恰巧看到了这个，所以就做一下并且记录下来，原文请参考 [butterfly博客导航栏居中](https://b.leonus.cn/2022/hexoCenter.html)

## 搜索按钮修改

按照如下代码修改文件 `themes\butterfly\layout\includes\header\nav.pug`

```PUG
nav#nav
  span#blog-info
    a(href=url_for('/') title=config.title)
      if theme.nav.logo
        img.site-icon(src=url_for(theme.nav.logo))
      if theme.nav.display_title
        span.site-name=config.title
    
  #menus
    !=partial('includes/header/menu_item', {}, {cache: true})
    #nav-right
      if (theme.algolia_search.enable || theme.local_search.enable || theme.docsearch.enable)
        #search-button
          a.site-page.social-icon.search(href="javascript:void(0);")
            i.fas.fa-search.fa-fw
          //- span=' '+_p('search.title')

    #toggle-menu
      a.site-page(href="javascript:void(0);")
        i.fas.fa-bars.fa-fw
```

## 导航栏居中

导航栏只需要绝对定位，然后距离屏幕左边 50% ，然后再往左移动自身的 50% 就可以了，这里需要写一个自定义的 `css` 文件，在文件夹下创建文件 `themes\butterfly\source\css\Navigation_style.css` ，在其中写入如下内容

```CSS
#nav .menus_items {
    position: absolute;
    width: fit-content;
    left: 50%;
    transform: translateX(-50%);
}
```

这里使用 `fit-content` 使 `menus_items` 的宽度为内容宽度，可以避免一些奇怪的 bug

## 子菜单栏横向

子菜单居中其实也非常的简单，只需要使其所有子菜单先变成横向，然后再居中就好了

### 子菜单横向

再上述所建立的 `.css` 文件中再添加如下代码

```CSS
#nav .menus_items .menus_item:hover .menus_item_child{
  display: flex;
}
```

### 子菜单居中

居中可以单独给每一个 `ul` 元素添加样式，可以把每一个子菜单调节到满意的位置，但是如果新增菜单需要修改一下这个文件

然后给每一个子菜单设置样式，这里拿其中一个做演示，如果有多个折叠子菜单，多复制几个就可以

```CSS
/* 这里的2是代表导航栏的第二个元素，即有子菜单的元素，可以按自己需求修改 */
.menus_items .menus_item:nth-child(2) .menus_item_child {
    left: -65px;
}
```
