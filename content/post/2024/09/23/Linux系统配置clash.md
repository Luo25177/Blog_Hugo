---
title: "Linux系统配置clash"
date: 2024-09-23 21:09:46
categories: "小工具"
tags: ["Linux", "clash"]
summary: "Linux配置clash，上网查资料必备"
---

### 前言

在 Linux 上进行项目开发时，经常会用到 github 来进行对项目的管理，但是苦于没有好的代理，经过一番查找，找到了 clash，是一个非常好用的代理软件 

### 安装clash

文件下载： https://zywang.lanzn.com/iPMwp2aneodc

下载完成之后，直接将其解压到 `Documents` 目录下，保证自己不会误删这个文件

### 使用clash

使用 `ctrl-alt-t` 打开终端，然后进入该文件目录中，按照里面的 `readme.md` 文件做一些配置

在终端输入

```bash
vim .env
```

使用 vim 修改该配置文件，将其中的 `CLASH_URL` 修改为自己的订阅地址，并且将 `CLASH_SECRET` 设置为一个比较简单的就行，例如 123。然后保存退出即可

之后在终端中输入

```bash
sudo bash start.sh
```

启动程序，然后会出现如下情况，并且会有一些开始使用的提示

![Screenshot from 2024-09-23 17-00-07.png](././Screenshot_from_2024-09-23_17-00-07.png)

然后如果是初次使用的话，需要进行一些基础的配置

在终端中输入如下指令，打开配置文件

```bash
vim /etc/profile.d/clash.sh
```

将其中的所有 `127.0.0.1` 修改为自己的 `ip` 地址，后面的端口号也可以自己修改，一般来说会使用 `127.0.0.1:7890` 端口，之后保存文件退出。退出之后还需要将这些设置应用一下。需要注意的是，这条指令只能在 `bash` 终端中使用。

```bash
source /etc/profile.d/clash.sh
```

之后在浏览器中输入 `127.0.0.1:9090/ui/#/conifgs` 进入如下页面

![Screenshot from 2024-09-23 17-08-47.png](././Screenshot_from_2024-09-23_17-08-47.png)

将 API Base URL 修改为 `127.0.0.1:9090` ，对应的密码可以设置为配置文件中 `CLASH_SECRET` 的值，然后点击 Add 即可添加一个按钮，这个就是 clash 界面的按钮，然后点击就可以进入 clash 的配置界面了，但是需要注意，只有打开代理之后才能看到配置界面

然后继续按照提示执行下面代码

```bash
source /etc/profile.d/clash.sh # 更新配置文件
proxy_on # 打开代理
```

之后就可以使用了

在第一次配置之后再次使用时，就不需要上述那么麻烦了，只需要在终端输入如下指令即可

```bash
sudo bash start.sh
source /etc/profile.d/clash.sh # 更新配置文件
proxy_on # 打开代理
```

### 参考

https://zhuanlan.zhihu.com/p/656856050

https://zhuanlan.zhihu.com/p/692414049
