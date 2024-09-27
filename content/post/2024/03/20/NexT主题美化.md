---
title: "NexT主题美化"
date: 2024-03-20 20:41:39
categories: "网站制作"
tags: ["NexT", "Hexo"]
summary: "NexT主题美化和一些问题的解决"
---

### 主题安装

当前用得最多的是next主题，下载地址：[theme-next/hexo-theme-next](https://link.zhihu.com/?target=https%3A//github.com/theme-next/hexo-theme-next)

可以直接在博客根目录中打开终端，输入代码将主题下载到目录 Blog/themes 中

```powershell
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

### 主题配置

打开根目录下的 `_config.yml` 文件（站点配置文件），查找并且修改

```yaml
title: 无风之地
subtitle: ''
summary: 或许从未开始，或许已经结束，但是我该走了
keywords:
author: 落
language: zh-CN
timezone: Asia/Shanghai

# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

将主题修改为 `next` ，主题的语言需要查看 `themes/next/language` 文件夹中的简体中文是 `zh-CN` 还是 `zh-Hans`

next的主题一共有4种，分别是 Muse，Mist，Pisces，Gemini。 可以挨个试试找一个喜欢的就行。选定主题需要在文件 `themes/next/_config.yml` 中查找并且修改

```yaml
# Schemes
scheme: Muse
# scheme: Mist
# scheme: Pisces
# scheme: Gemini
```

然后在根目录打开终端，输入下列指令

```powershell
hexo clean
hexo g
hexo d
```

完成之后打开你的博客就能看到了

### 设置菜单

```powershell
menu:
  home: / || home                      #首页
  archives: /archives/ || archive      #归档
  categories: /categories/ || th       #分类
  tags: /tags/ || tags                 #标签
  about: /about/ || user               #关于
  #schedule: /schedule/ || calendar    #日历
  #sitemap: /sitemap.xml || sitemap    #站点地图，供搜索引擎爬取
  #commonweal: /404/ || heartbeat      #腾讯公益404
```

“||”前面的是目标链接，后面的是图标名称，next使用的图标全是 [图标库 - Font Awesome 中文网](https://link.zhihu.com/?target=http%3A//www.fontawesome.com.cn/faicons/%23web-application) 这一网站的，有想用的图标直接在该网站上面找图标的名称就行。新添加的菜单需要翻译对应的中文，打开 `theme/next/languages/zh-CN.yml` ，在 `menu` 下设置

```powershell
menu:
  home: 首页
  archives: 归档
  categories: 分类
  tags: 标签
  about: 关于
  search: 搜索
```

在根目录下打开终端，输入如下代码

```powershell
hexo new page "categories"
hexo new page "tags"
hexo new page "about"
hexo new page "resources"
```

此时在根目录的sources文件夹下会生成 categories，tags，about，resources 四个文件，每个文件中有一个`index.md`文件，修改内容分别如下。如果有启用评论，默认页面带有评论。需要关闭的话，添加字段comments并将值设置为false。

```powershell
---
title: 分类
date: 2020-02-10 22:07:08
type: "categories"
comments: false
---

---
title: 标签
date: 2020-02-10 22:07:08
type: "tags"
comments: false
---

---
title: 关于
date: 2020-02-10 22:07:08
type: "about"
comments: false
---

---
title: 资源
date: 2020-02-10 22:07:08
type: "resources"
comments: false
---
```

### 设置建站时间

打开主题配置文件即 `themes/next` 下的 `_config.yml` ，查找 `since`

```powershell
footer:
  # Specify the date when the site was setup. If not defined, current year will be used.
  since: 2020-02   #建站时间
```

### 设置头像

打开主题配置文件即 `themes/next` 下的 `_config.yml` ，查找 `avatar` ， `url` 后是图片的链接地址

```powershell
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.jpg
  # If true, the avatar will be dispalyed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: false
```

这里头像图片是需要保存在 `themes/next/source/images` 文件夹中

### 网站图标设置

只需要修改 `small` 和 `medium` 为图标路径就可以了

```powershell
favicon:
  small: /images/favicon-16x16.png
  medium: /images/favicon-32x32.png
  apple_touch_icon: /images/apple-touch-icon-next.png
  safari_pinned_tab: /images/logo.svg
  #android_manifest: /images/manifest.json
  #ms_browserconfig: /images/browserconfig.xml
```

### 设置背景图片

打开主题配置文件，将 `style:source/_data/style.stl` 取消注释

```powershell
custom_file_path:
  style: source/_data/styles.styl
```

打开根目录 `source` 创建文件 `_data/styles.styl` ，添加以下内容

```powershell
// 添加背景图片
body {
      background: url(images/background.png);//自己喜欢的图片地址
      background-size: cover;
      background-repeat: no-repeat;
      background-attachment: fixed;
      background-position: 50% 50%;
}
```

### 主页文章添加阴影效果

两种方法

1. 打开 `themes/next/source/css/_common/conponents/post/post.styl` ，修改 `.post-block`
    
    ```powershell
    .use-motion {
      if (hexo-config('motion.transition.post_block')) {
        .post-block {
          opacity: 0;
          margin-top: 60px;
          margin-bottom: 60px;
          padding: 25px;
          background:rgba(255,255,255,0.9) none repeat scroll !important;
          -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
          -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
    
        }
        .pagination, .comments{
          opacity: 0;
        }
      }
    ```
    
2. 打开 `source/_date/style.styl` 文件，添加以下代码
    
    ```powershell
    .post {
       margin-top: 60px;
       margin-bottom: 60px;
       padding: 25px;
       -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
       -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
     }
    ```
    

### 设置预览摘要

一般来说，上传完文章之后，会在主页上显示，但是显示的是原文，所以就显得十分难看。所以在每一个创建的 `.md` 文章上面添加如下代码，并且填写上对应信息，之后主页上显示的文章的内容就是 `description` 中的内容

```powershell
---
title: 
date: 
categories: 
summary: 
---
```

### 设置侧边栏显示效果

打开主题配置文件，找到 `Sidebar Settings` ，设置

```powershell
sidebar:
  # Sidebar Position. #设置侧边栏位置
  position: left
  #position: right

  #  - post    默认显示模式
  #  - always  一直显示
  #  - hide    初始隐藏
  #  - remove  移除侧边栏
  display: post
```

### 添加社交链接

打开主题配置文件，搜索并且修改

```powershell
social:
  GitHub: https://github.com/Luo25177|| github
  E-Mail: mailto:beloved25177@126.com || envelope
```

### 将博文内链接设置为蓝色

打开 `themes/next/source/css/_common/components/post/post.styl` 文件，将下面代码复制到最后

```powershell
.post-body p a{
     color: #0593d3;
     border-bottom: none;
     &:hover {
       color: #0477ab;
       text-decoration: underline;
     }
   }
```

### 显示文章字数和阅读时长

需要用到插件 `hexo-wordcount` ，在 `Blog` 目录下打开终端，执行如下命令

```powershell
npm install hexo-wordcount --save
```

然后打开站点配置文件，在文件末加上下列代码

```powershell
symbols_count_time:
  symbols: true                # 文章字数统计
  time: true                   # 文章阅读时长
  total_symbols: true          # 站点总字数统计
  total_time: true             # 站点总阅读时长
  exclude_codeblock: false     # 排除代码字数统计
```

### 显示站点文章总字数

需要用到插件 `hexo-wordcount` ，在 `Blog` 目录下打开终端，执行如下命令

```powershell
npm install hexo-wordcount --save
```

然后在 `/themes/next/layout/_partials/footer.swig` 文件尾部加上下列代码

```powershell
<div class="theme-info">
  <div class="powered-by"></div>
  <span class="post-count">博客全站共{{ totalcount(site) }}字</span>
</div>
```

### 设置文章末尾”本文结束”标记

在路径 `/themes/next/layout/_macro` 中新建 `passage-end-tag.swig` ，文件，并且添加如下代码

```powershell
<div>
    {% if not is_index %}
        <div style="text-align:center;color: #ccc;font-size:24px;">-------------本文结束<i class="fa fa-paw"></i>感谢您的阅读-------------</div>
    {% endif %}
</div>
```

接着打开 `/themes/next/layout/_macro/post.swig` 文件，在 `post-footer` 前， `END POST BODY` 之后添加下列代码

```powershell
 {% if not is_index and theme.passage_end_tag.enabled %}
   <div>
     {% include 'passage-end-tag.swig' %}
   </div>
 {% endif %}
```

在主题配置文件的末尾添加

### 文章末尾添加版权说明

查找主题配置文件中的 `creative_commons`

```powershell
creative_commons:
  license: by-nc-sa
  sidebar: false
  post: true  # 将false改为true即可显示版权信息
  language:
```

### 添加访问量统计

打开主题配置文件，查找并且修改下列代码

```powershell
busuanzi_count:
  enable: true
```

打开 `/themes/next/layout/_partials/footer.swig` ，在文件最后添加下列代码

```powershell
{% if theme.busuanzi_count.enable %}
    <script async src="//dn-lbstatics.qbox.me/busuanzi/2.3/busuanzi.pure.mini.js"></script>

    <span id="busuanzi_container_site_pv">总访问量<span id="busuanzi_value_site_pv"></span>次</span>
    <span class="post-meta-divider">|</span>
    <span id="busuanzi_container_site_uv">总访客数<span id="busuanzi_value_site_uv"></span>人</span>
    <span class="post-meta-divider">|</span>
<!-- 不蒜子计数初始值纠正 -->
<script>
$(document).ready(function() {

    var int = setInterval(fixCount, 50);  // 50ms周期检测函数
    var countOffset = 20000;  // 初始化首次数据

    function fixCount() {            
       if (document.getElementById("busuanzi_container_site_pv").style.display != "none")
        {
            $("#busuanzi_value_site_pv").html(parseInt($("#busuanzi_value_site_pv").html()) + countOffset); 
            clearInterval(int);
        }                  
        if ($("#busuanzi_container_site_pv").css("display") != "none")
        {
            $("#busuanzi_value_site_uv").html(parseInt($("#busuanzi_value_site_uv").html()) + countOffset); // 加上初始数据 
            clearInterval(int); // 停止检测
        }  
    }
       	
});
</script> 
{% endif %}
```

### **本地搜索**

在 `Blog` 目录下右键打开终端并且输入

```powershell
npm install hexo-generator-search --save
```

在主题配置文件中，查找并且做如下修改

```powershell
local_search:
  enable: true
  trigger: manual   #手动，按回车键或搜索按钮触发搜索
```

### 代码块样式自定义

打开文件 `Blog/source/_data/styles.styl` ，添加以下内容：

```powershell
// Custom styles.
code {
    color: #ff7600;
    background: #fbf7f8;
    margin: 2px;
}
// 大代码块的自定义样式
.highlight, pre {
    margin: 5px 0;
    padding: 5px;
    border-radius: 3px;
}
.highlight, code, pre {
    border: 1px solid #d6d6d6;
}
```

### 公式显示

对于我来说，博客中有很多做控制的文章，所以会有很多公式，而公式显示不好就令作为一个完美主义者的我很头疼，所以查找了很多文章才把效果做好

`NexT` 主题内部已经集成了 `MathJax` 和 `KaTeX` ，相对来说配置起来已经是很方便了。但在一些细节方面上还是存在各种小问题，比如说不支持行内渲染，公式换行，渲染冲突和超长公式等，所以下面会对这些问题做出解决方案

1. 启用 `MathJax`
    
    打开主题配置文件，查找 `math` 并作如下修改
    
    ```powershell
    math:
      enable: true
      per_page: true
    ```
    
    - `per_page`默认为`true`，表示每篇文章需要额外地单独启用 MathJax。 也就是说，还需要在其文章头部添加`mathjax: true`，否则其中的 MathJax 公式依然不会被解析。
    - `engine`默认使用的是`mathjax`，相比`katex`功能更强大，同时也会慢一点
2. 更换渲染引擎
    
    启用完 MathJax 之后，就可以写一些简单的公示了，但是对于一些比较复杂的公式就会渲染出错，主要原因就是Hexo默认的渲染引擎`hexo-renderer-marked`对MathJax的支持很不好，会出现各种莫名其妙的问题。主题官方也推荐更换其它渲染引擎。这里推荐使用 `hexo-renderer-kramed`
    
    在 `Blog` 文件夹下打开终端并且输入如下指令来实现渲染引擎的更换
    
    ```powershell
    npm un hexo-renderer-marked --save
    npm i hexo-renderer-kramed --save
    ```
    
3. 渲染冲突问题
    
    对于以下公式
    
    ```markdown
    $ \alpha\beta $
    
    $ \alpha_\beta $
    
    $ \alpha_\beta = \gamma_\delta $
    ```
    
    最终渲染效果为 $\alpha\beta$， $\alpha_\beta$ 和 $ \alpha*\beta = \gamma*\delta $
    
    实际上是把同一行中的下划线渲染成了斜体符号，所以就出现上述问题，有个好办法就是直接修改 `hexo-renderer-kramed` 的源码
    
    打开文件， `node_modules/kramed/lib/rules/inline.js` ，做如下修改
    
    ```jsx
    var inline = {
      // escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
      escape: /^\\([`*\[\]()#$+\-.!_>])/,
      // em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
    	em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
    };
    ```
    
    注意只改这两行，其它不需要改
    
4. 公式超长
    
    NexT主题对公式的样式配置存在一些小问题，导致当公式太长的时候，显示内容会超出文章区域。
    
    如果使用默认的配置，行内公式超长时总会超出文章显示区域，而块公式时候会超出区域则取决于选择的渲染方式（HTML-CSS不会超出区域，其他情况会）。
    
    进行了修改后，总是会在公式超出区域时生成滚动条，效果更佳理想。
    
    打开文件 `themes/next/layout/_third-party/math/mathjax.swig` 文件，在文件末端添加如下代码
    
    ```jsx
    <script type="text/x-mathjax-config">
       MathJax.Hub.Queue(function() {
         var all = MathJax.Hub.getAllJax(), i;
           for (i = 0; i < all.length; i += 1) {
             document.getElementById(all[i].inputID + '-Frame').parentNode.classList.add('has-jax');
           }
       });
     </script>
     <script src="{{ theme.math.mathjax.cdn }}"></script>
     
     <style>
    .has-jax {
       overflow-x: auto;
       overflow-y: hidden;
     }
     </style>
    ```
    
    最终也得到了解决，但是也有一些其它的问题
    

### 参考

[从零开始搭建个人博客（超详细） - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/102592286)

[Hexo+NexT使用MathJax问题 | I'm dev (i-m.dev)](https://i-m.dev/posts/20190304-210101.html)
