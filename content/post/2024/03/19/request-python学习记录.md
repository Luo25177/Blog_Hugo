---
title: "request-python学习记录"
date: 2024-03-19 09:56:56
categories: "爬虫"
tags: ["python", "request"]
summary: "python爬虫的学习记录和使用"
---

## requests 学习

**静态网络与动态网络**

- 动态xml页面: 如果网络界面是由js加载上去的，那么网络就是动态的
- 静态xml页面: 没有使用js加载
- 通用爬虫: 通常指搜索引擎的爬虫 面向互联网上所有的网站
- 聚焦爬虫: 针对特定网站的爬虫 针对几个网站，特定的网站

**步骤**

1. url list
2. 响应内容 提取url
3. 提取数据
4. 入库

**DNS服务器**: 根据客户给定的域名，来返回对应的 ip 地址，客户通过 ip 地址请求页面，返回页面的 html+js+css+.jpg

### url形式:

- scheme://host[:port# ]/path/.../[?query-string ][#anchor ]
- scheme: 协议(http,https,ftp)
- host: 服务器的ip地址或者域名
- port: 服务器的端口(如果是默认端口就是 80 / 443)
- path: 访问资源的路径
- query-string: 参数，发送给http服务器的数据
- anchor: 锚点(跳转到网页的指定冒点位置)
url地址带上锚点与不带锚点是一样的

### HTTP

http

- 超文本传输协议
- 默认端口号: 80
https / [网址后面加.cn](http://xn--yfr16ad4eh64dhfza.cn/)
- http + SSL(安全套接字层)
- 默认端口号: 443
https 比 http 更安全，但是性能更低
https 在发送数据之前会对数据进行加密，服务端接收到数据之后还需要对数据进行解密，就可以获得内容

超文本传输协议（HTTP，HyperText Transfer Protocol)是互联网上应用最为广泛的一种网络协议。所有的WWW文件都必须遵守这个标准。设计HTTP最初的目的是为了提供一种发布和接收HTML页面的方法,HTTP是一种基于"请求与响应"模式的、无状态的应用层协议。HTTP协议采用URL作为定位网络资源的的标识符。
[http://host](http://host/)[:post][path]
host:合法的Internet主机域名或ip地址
port:端口号，缺省为80
path:请求资源的路径

**user-agent**: 包括电脑的版本号，浏览器的版本号，浏览器的内核版本，操作系统的信息(linux,windows,macos 以及对应的版本)
通过user-agent能够让网页知道访问它的对象的信息等，如果发现
**cookie**: 就是保存在浏览器本地的本人浏览器信息，但是不太安全，而且一个站点的cookie都是有限的，不能无线申请
**session**: 就是保存在网址上的个人信息数据，一般可以存储无限个

我们在申请网址的时候只需要带上name和value就可以

Ajax

### HTTP URl的理解:

url是通过HTTP协议存取资源的的Internet路径，一个URL对应一个数据资源

### HTTP协议对资源的操作

| 方法                                                                                                                                                             | 说明                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| GET                                                                                                                                                              | 请求获取URL位置的资源                                   |
| HEAD                                                                                                                                                             | 请求获取URL位置资源的响应消息报告，即获得资源的头部信息 |
| POST                                                                                                                                                             | 请求向URL位置的资源后附加新的消息                       |
| PUT                                                                                                                                                              | 请求向URL位置存储一个资源，覆盖原URL位置的资源          |
| PATCH                                                                                                                                                            | 请求局部更新URL位置的资源,即改变该处资源的部分内容      |
| DELETE                                                                                                                                                           | 请求删除URL位置存储的资源                               |
| 以上方法中，GET,HEAD是从服务器获取信息到本地，PUT,POST,PATCH,DELETE是从本地向服务器提交信息。通过URL和命令管理资源，操作独立无状态，网络通道及服务器成了黑盒子。 |                                                         |
| 文档                                                                                                                                                             |                                                         |

### requests库7个主要方法

| 方法             | 说明                                                      |
| ---------------- | --------------------------------------------------------- |
| requsts.requst() | 构造一个请求，最基本的方法，是下面方法的支撑              |
| requsts.get()    | 获取网页，对应HTTP中的GET方法                             |
| requsts.post()   | 向网页提交信息，对应HTTP中的POST方法                      |
| requsts.head()   | 获取html网页的头信息，对应HTTP中的HEAD方法                |
| requsts.put()    | 向html提交put方法，对应HTTP中的PUT方法                    |
| requsts.patch()  | 向html网页提交局部请求修改的的请求，对应HTTP中的PATCH方法 |
| requsts.delete() | 向html提交删除请求，对应HTTP中的DELETE方法                |

只有post请求才能够请求数据

应用场景:
get(): 除了post请求之外的指令一般都使用get请求
post(): 提交表单，传输数据非常多的时候，例如:利用百度翻译或者传输图片

### [响应状态码](https://blog.csdn.net/qq_45477063/article/details/124810709)

- 200: 成功
- 302: 临时转移至新的url
- 307: 临时转移至新的url
- 404: not found
- 500: 服务器内部错误，可能是对方反爬虫

assert 断言
str 可以通过endode方法转化为 bytes
bytes 也可以通过decode方法转化为 str

### 爬虫的流程

- url--->发送请求，获取响应--->提取数据--->保存
- 发送请求，获取相应--->提取url
爬虫要根据当前的url地址对应的相应为准，当前的url地址的elements的内容和url的响应不一样

## get()

get()请求的返回值就是一个响应数据，也就是请求的结果，响应状态码
get()的返回值包括text，就是返回值的一个属性

- text
    - 类型 str
    - 解码类型 根据http头部对响应的编码做出有根据的推测来推测出来的文本编码
    - 修改解码方式: response.text.encoding = "utf8"
    encoding，响应内容的编码方式
- content
    - 类型 bytes
    - 解码类型 没有指定
    - 可以修改解码方式 response.content.decode("utf8")

### 判断是否请求成功

使用assert，后面加入判断内容，如果判断内容是真的，那就不报错，否则就会报错

### response 的常用方法

- response.text
- response.content
- response.status_code
- response.request.headers
- response.headers
- response.url 会打印出当前的网址，并且带参数，是一种url的编码方式，可以使用url解码

### 字符串格式化的方式

```python
url = "<http://www.baidu.com/s?wd={}".format("pyhton>")
```

## post()

### 使用场景

- 登录注册(post比get更安全)
- 需要传输大文本内容的时候

### 用法

```python
reponse = requests.post("<http://www.baidu.com>",data = data,headers = headers)
```

data:字典的形式，用来存储需要传输的数据
传回的数据是 json 类型的一个数据

返回的数据的类型:json,text,html

### 使用代理ip

- 使用爬虫需要使用代理
- 根据ip地址能够追查到某个地址某个人，防止真实的ip地址暴漏
- 代理ip:
    - 透明代理 可以追查到真实的ip地址
    - 普匿代理
    - 高匿代理 不易追查
- 检查代理ip的可用性:
    - 可以使用requests添加超时参数，判断ip地址的质量
- 可以准备一堆ip地址，组成ip池，随机选择一个ip来用
    - 随机选择ip代理地址
        - {"ip":ip,"times":0}
        - [{},{},{}]用列表来进行排序，按照使用次数来排列
        - 选择使用较少的10个ip地址，从中选取一个

### cookie 和 session

- cookie存放在客户的浏览器上，session保存在服务器上
- cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗
- session会在一定时间内保存在服务器上，当访问增多，会比较占用你的服务器的性能
- 单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点只能保存20个cookie
- 带上cookie和session的好处:能够请求到登录之后的界面
- 带上cookie和session的弊端:
    - 一套cookie和session往往之和一个用户对应
    - 请求太快，请求太多，容易被服务器识别为爬虫
- 不需要cookie的时候尽量不使用cookie
- 但是为了获取登录之后的页面，必须使用cookie

### 携带cookie进行请求

- 携带一堆cookie进行请求，把cookie组成一个cookie池
- cookie储存为一个字典，然后字典组成列表，进行排序使用

### requests中的session类

session来实现客户端和服务端的会话保持
使用方法:

- 实例化一个session对象
- 让session来发送get或者post请求
- session = requests.session()
- response = session.get(url,header)
- 这个session中保存了对方服务器中设置的cookie,其实就是获取到了之前登陆的信息

请求登陆网站的思路:

- 实例化session
- 先使用session发送请求，登录对应网站，把cookie保存在session中
- 在使用session请求登陆之后才能访问的网站，session能够自动携带登陆成功时保存在其中的cookie信息，进行请求

### 不发送post请求，使用cookie获取登陆后的页面

- cookie过期时间太长的网站
- cookie过期之前能够拿到所有的数据
- 配合其他的程序一起使用，其他的程序获取cookie，当前程序专门请求页面

### 字典生成 字典推导式

能够把cookie转化为字典:

```python
cookies = {i.split("=")[0]:i.split("=")[1] for i in cookies.splits(";")}
```

就是能把 ; 作为分割的工具，把一个个元素分割，而且等号前面的作为 key，等号后面的作为 value
cookie是存在于headers中的，也可以直接放到请求里去

### 获取登陆后页面的三种方式

1. 实例化session，使用session发送post请求，在使用session来获取登录之后的界面
2. headers中添加cookie键，值为cookie字符串
3. 在请求方法中添加cookie作为参数，接受字典形式的cookie，字典形式的cookie中的键是cookie的name，值是cookie的valuek

### 寻找登陆的 post 地址

- 在form表单中寻找action地址
    - post的数据是input标签中name的值作为变量，真正的用户名密码作为值的字典，post的url地址就是action对应的地址
    - 如果没有action的url地址的解决方法:
        - 抓包,但是登陆的时候页面会跳转就看不到之前的界面的源码了
            - 使用错误的密码登录
            - 在检查界面勾选上 preserve log

### requests 小技巧

```python
import requests.utils
requests.utils.dict_from_cookiejar  # 就是把cookie对象转化为字典
requests.utils.cookiejar_from_dict  # 就是把为字典转化为cookies
requests.utils.unquote(未解码的url地址) # 就是对url地址解码
requests.utils.quote(url) # 就是把url进行编码，但是不会把/编码，会对:?进行编码
response = requests.get(url,timeout = 10) # 设置超时，10s钟之内没有返回响应就会报错
response = requests.get("<https://www.12306.cn/mormhweb/>",verify = False) # 获取证书验证 verify 就是表示是否证书验证，false就是不验证，默认为true
assert response.status_code == 200 # 断言，如果为真就继续运行，如果为假就会报错
```

cookie 是存在于 response 中的，就是网页保存到我们本地的 cookie

alias:可以对指令进行重命名，但是只能在linux中使用，例如
alias fanyi="Python 绝对路径/xxx.py",就相当于在终端种输入了 python 绝对路径/xxx.py