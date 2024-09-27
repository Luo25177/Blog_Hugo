---
title: "ButterFly主题美化"
date: 2024-03-22 08:57:49
categories: "网站制作"
tags: ["ButterFly", "Hexo"]
summary: "ButterFly主题的一些基础的配置"
---

我这里只总结了自己用到的，可以看原教程：[hexo主题butterfly配置](https://blog.csdn.net/mjh1667002013/article/details/129290903)

### 主题安装

在项目的根目录 `Blog` 右键打开终端，输入如下指令安装

```powershell
git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
```

安装 `pug` 和 `stylus` 渲染器

```powershell
npm install hexo-renderer-pug hexo-renderer-stylus --save
```

修改占点配置文件，开启主题

```yaml
theme: butterfly
```

为了減少升级主题带来的不便，可以把主题文件夹中的 `_config.yml` 重命名为 `_config.butterfly.yml`，复制到根目录下与站点配置文件同级。

`Hexo` 会自动合并主题中的 `_config.yml` 和 `_config.butterfly.yml` ，如果存在同名配置，会使用 `_config.butterfly.yml` 的配置，其优先度较高。所以像和博客网址相关联的固定资料可以设置在 `_config.yml` 中，比如博客的标题、作者信息和邮箱等等资料，而和主题样式相关的配置放在 `_config.butterfly.yml` 中，那么在将来你想换一个主题是很方便的。

### 设置博客个人资料

修改站点配置文件，可以修改网站的各种基本资料

```yaml
title: 无风之地
subtitle: ''
summary: 或许从未开始，或许已经结束，但是我该走了
keywords:
author: 落
language: zh-CN
timezone: Asia/Shanghai
```

### 导航栏菜单

修改主题配置文件，如下

```yaml
# Menu 目錄
menu:
  主页: / || fas fa-home
  博文 || fa fa-graduation-cap:
    分类: /categories/ || fa fa-archive
    标签: /tags/ || fa fa-tags
    归档: /archives/ || fa fa-folder-open
  生活 || fas fa-list:
    分享: /show/ || fa fa-external-link
    相册: /photos/ || fa fa-camera-retro
    音乐: /music/ || fa fa-music
  友链: /link/ || fa fa-link
  # 留言板: /comment/ || fa fa-paper-plane
  #留言板: /messageboard/ || fa fa-paper-plane
  关于笔者: /about/ || fas fa-user
```

### 代码块显示设置

根据如下修改主题配置文件

```yaml
highlight_theme: mac #  darker / pale night / light / ocean / mac / mac light / false
highlight_copy: true # copy button
highlight_lang: true # show the code language
highlight_shrink: false # true: shrink the code blocks / false: expand the code blocks | none: expand code blocks and hide the button
highlight_height_limit: false # unit: px
code_word_wrap: false # 代码自动换行，关闭滚动条
```

同时将站点配置文件的 `highlight` 的 `enable` 设置为 `false`

```yaml
highlight:
  enable: false
  line_number: true
  auto_detect: false
```

### 本地搜索功能

- 安装搜索插件
    
    ```powershell
    npm install hexo-generator-search --save
    ```
    
- 在主题配置文件中查找修改
    
    ```yaml
    local_search:
      enable: true
      labels:
        input_placeholder: Search for Posts
        hits_empty: "We didn't find any results for the search: ${query}" # 如果没有查到内容相关内容显示
    ```
    

### 创建文件夹

创建对应的文件夹，需要保证创建的文件夹名称与导航栏菜单里的 `/ /` 中的内容一致才能打开对应的栏目。在根目录下右键打开终端输入指令

```powershell
hexo new page name
```

这个 `name` 就是你要创建的文件夹名称，例如

```powershell
hexo new page categories // 创建分类
hexo new page tags // 创建标签
```

指令输入之后会在 `/source/` 文件夹下会生成你所创建的文件夹，并且其中都包含着一个 index.md 文件，并且做如下修改， `title` 是对应的文件夹下的 index.md 文件做的修改

```markdown
---
title: categories
date: 2024-03-18 14:19:13
type: "categories"
comments: false
---

---
title: tags
date: 2024-03-18 14:19:18
type: "tags"
comments: false
---

---
title: about
date: 2024-03-18 14:19:22
type: "about"
comments: false
---
```

### 修改副标题

在主题配置文件中，查找并修改

```yaml
# the subtitle on homepage (主頁subtitle)
subtitle:
  enable: true
  # Typewriter Effect (打字效果)
  effect: true
  # Customize typed.js (配置typed.js)
  loop: true
  # https://github.com/mattboldt/typed.js/#customization
  typed_option:
  # source 調用第三方服務
  # source: false 關閉調用
  # source: 1  調用一言網的一句話（簡體） https://hitokoto.cn/
  # source: 2  調用一句網（簡體） https://yijuzhan.com/
  # source: 3  調用今日詩詞（簡體） https://www.jinrishici.com/
  # subtitle 會先顯示 source , 再顯示 sub 的內容
  source: false
  # 如果關閉打字效果，subtitle 只會顯示 sub 的第一行文字
  sub:
  - 你的心界愈空灵，愈不觉得物界喧嚣
```

### 图片设置

- 网站图标
    
    ```yaml
    favicon: /img/favicon.png
    ```
    
- 头像
    
    ```yaml
    avatar:
      img: /img/avatar.jpg #图片路径
      effect: false #头像会一直转圈  
    ```
    
- 网站背景图
    
    ```yaml
    # Website Background (設置網站背景)
    # can set it to color or image (可設置圖片 或者 顔色)
    # The formal of image: url(http://xxxxxx.com/xxx.jpg)
    background: url(/img/background.jpg)
    ```
    
- 主页头部图
    
    ```yaml
    index_img: /img/index_img.jpg
    ```
    
- 文章详情页图
    
    ```yaml
    # If the banner of page not setting, it will show the top_img
    default_top_img: /img/top_img.jpg
    ```
    
- 归档页顶部图
    
    ```yaml
    # The banner image of archive page
    archive_img: /img/archive_img.jpg
    ```
    
- tag 标签页顶部图
    
    ```yaml
    # If the banner of tag page not setting, it will show the top_img
    # note: tag page, not tags page (子標籤頁面的 top_img)
    tag_img: /img/tag_img.jpg
    ```
    
- categories 分类页顶部图
    
    ```yaml
    # If the banner of category page not setting, it will show the top_img
    # note: category page, not categories page (子分類頁面的 top_img)
    category_img: /img/category_img.jpg
    ```
    
- 统一文章封面
    
    ```yaml
    cover:
      # display the cover or not (是否顯示文章封面)
      index_enable: true
      aside_enable: true
      archives_enable: true
      # the position of cover in home page (封面顯示的位置)
      # left/right/both
      position: both
      # When cover is not set, the default cover is displayed (當沒有設置cover時，默認的封面顯示)
      default_cover:
        - https:
        - http:
        - http:
        - http:
        - http:
        - http:
    ```
    
- 为每一篇文章设置不同的封面，可以在文章自己的 markdown 文件中添加配置
    
    ```markdown
    ---
    title: Hello World
    tags: [hello]
    categories:
    summary: hello word~
    top_img: /img/hello-1.png # 顶部背景图
    cover: /img/hello-1.png # 文章封面
    ---
    ```
    

### 图片大图查看

修改主题配置文件

```yaml
medium_zoom: false
fancybox: true
```

### 版权样式

修改主题配置文件

- 复制内容增加版权信息
    
    ```yaml
    # copyright: Add the copyright information after copied content (複製的內容後面加上版權信息)
    copy:
      enable: true
      copyright:
        enable: false
        limit_count: 50
    ```
    
- 文章版权信息
    
    ```yaml
    post_copyright:
      enable: true
      decode: true
      author_href: https://seashore.top
      license: CC BY-NC-SA 4.0
      license_url: https://creativecommons.org/licenses/by-nc-sa/4.0/
    ```
    

### 相关文章

在文章最下面出现推送信息

```yaml
# Related Articles
related_post:
  enable: true
  limit: 6 # 显示文章的最大数量
  date_type: created # 创建日期或者更新日期 created update
```

### 侧边栏样式

修改主题配置文件

```yaml
aside:
  enable: true
  hide: false
  button: true
  mobile: true # display on mobile
  position: right # left or right 
```

### 个人信息

修改主题配置文件

```yaml
social:
   fab fa-github: https://github.com/xxx || Github
   fab fa-qq: tencent://AddContact/?fromId=45&fromSubId=1&subcmd=all&uin=xxx&website=www.oicqzone.com || QQ
   fas fa-envelope-open-text: mailto:xxx|| Email

```

### 公告栏

修改主题配置文件

```yaml
card_announcement:
    enable: true
    content: This is my Blog #修改公告栏信息
```

### TOC目录

修改主题配置文件

```yaml
# toc (目錄)
toc:
  post: true
  page: false
  number: false
  expand: true # 是否展开
  style_simple: false # for post
  scroll_percent: true
```

### 背景特效美化

修改主题配置文件

- 鼠标点击效果
    
    ```yaml
    # Typewriter Effect (打字效果)
    # https://github.com/disjukr/activate-power-mode
    activate_power_mode:
      enable: false
      colorful: true # open particle animation (冒光特效)
      shake: true #  open shake (抖动特效)
      mobile: false
    
    # Mouse click effects: fireworks (鼠标点击效果:萤火特效)
    fireworks:
      enable: false
      zIndex: 9999 # -1 or 9999
      mobile: false
    
    # Mouse click effects: Heart symbol (鼠标点击效果: 爱心)
    click_heart:
      enable: false
      mobile: false
    
    # Mouse click effects: words (鼠标点击效果: 文字)
    ClickShowText:
      enable: false
      text:
         - 富强
         - 民主
         - 文明
         - 和谐
         - 平等
         - 公正
         - 法治
         - 爱国
         - 敬业
         - 诚信
         - 友善
      fontSize: 15px
      random: true
      mobile: true
    ```
    
- 打字效果
    
    ```yaml
    # Typewriter Effect (打字效果)
    # https://github.com/disjukr/activate-power-mode
    activate_power_mode:
      enable: false
      colorful: true # open particle animation (冒光特效)
      shake: true #  open shake (抖动特效)
      mobile: true
    ```
    
- 背景特效
    
    ```yaml
    # Background effects (背景特效)
    # canvas_ribbon (静止彩带)
    # See: https://github.com/hustcc/ribbon.js
    canvas_ribbon:
      enable: false
      size: 150
      alpha: 0.6
      zIndex: -1
      click_to_change: false
      mobile: false
    # Fluttering Ribbon (动态彩带)
    canvas_fluttering_ribbon:
      enable: false
      mobile: false
    #星空特效
    # canvas_nest
    # https://github.com/hustcc/canvas-nest.js
    canvas_nest:
      enable: true
      color: '0,0,255' #color of lines, default: '0,0,0'; RGB values: (R,G,B).(note: use ',' to separate.)
      opacity: 0.7 # the opacity of line (0~1), default: 0.5.
      zIndex: -1 # z-index property of the background, default: -1.
      count: 99 # the number of lines, default: 99.
      mobile: false
    ```
    
- footer 背景
    
    ```yaml
    footer_bg: true # 设置为 false 将于主题色一致，设置为 true 与 top_img 一致
    ```
    

### 字数统计

- 安装字数统计插件
    
    ```yaml
    npm install hexo-wordcount --save
    ```
    
- 修改主题配置文件
    
    ```yaml
    # wordcount (字數統計)
    wordcount:
      enable: true
      post_wordcount: true
      min2read: true
      total_wordcount: true
    ```
    

### 分享

修改主题配置文件，在 `addThis` ， `sharejs` ， `addtoany` 中选择一个开启就可以

```yaml
# Share System (分享功能)
# --------------------------------------

# AddThis
# https://www.addthis.com/
addThis:
  enable: false
  pubid:  #访问 AddThis 官网, 找到你的 pub-id

# Share.js
# https://github.com/overtrue/share.js
sharejs:
  enable: true  
  sites: facebook,twitter,wechat,weibo,qq  #想要显示的内容

# AddToAny
# https://www.addtoany.com/
addtoany:
  enable: false
  item: facebook,twitter,wechat,sina_weibo,facebook_messenger,email,copy_link
```
