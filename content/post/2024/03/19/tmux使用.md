---
title: "tmux使用"
date: 2024-03-19 00:39:35
categories: "小工具"
tags: ["tmux"]
summary: "tmux安装配置与使用"
---

## tmux安装

在 linux 命令行终端只需要一行指令就可以

```shell
sudo apt install tmux
```

## tmux 使用指令 针对于同一个窗口不同的分割操作

竖直分割窗口 crtl-b "
水平分割窗口 crtl-b %
关闭窗口 crtl-b &
切换窗口 crtl-b 上下左右按键，或者 ;

tmux 窗口指令，针对于对不同的窗口操作
& 关闭当前窗口
l 前后窗口间互相切换
. 修改当前窗口编号，相当于重新排序
f 在所有窗口中查找关键词
, 重命名当前窗口
w 列出所有窗口
% 水平分割窗口
" 竖直分割窗口
n 选择下一个窗口
p 选择上一个窗口
0~9 选择 0~9 对应的窗口
空格 更改竖直与水平的窗格
x根据提示关闭 关闭当前窗格 或者直接 crtl-D 关闭
z 当前窗口全屏显示

tmux的分割窗口实现实际上是多个终端，并且可以通过echo对终端的文件进行操作，然后会出现在对应终端上 `/dev/pts/n`

## 配置文件

```c
#~/.tmux.conf
#tmux source-file ~/.tmux.conf
#设置终端颜色为256色  
set -g default-terminal "screen-256color"  
#设置pan前景色  
#set -g pane-border-fg green  
#设置status-bar颜色  
set -g status-fg white  
set -g status-bg black  
#设置窗口列表颜色  
#设置status bar格式  
set -g status-left-length 40  
set -g status-left "#[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"  
set -g status-right "#[fg=cyan]%b %d %R"  
set -g status-interval 60  
set -g status-justify centre 

set -g base-index         1     # 窗口编号从 1 开始计数
set -g display-panes-time 10000 # PREFIX-Q 显示编号的驻留时长，单位 ms
set -g mouse              on    # 开启鼠标
set -g pane-base-index    1     # 窗格编号从 1 开始计数
set -g renumber-windows   on    # 关掉某个窗口后，编号重排

# 启用活动警告
setw -g monitor-activity on
set -g visual-activity on
set -g default-command /bin/fish

# split panes using | and -
bind l split-window -h #扩展窗口
bind j split-window -v
unbind '"'
unbind %

#移动扩展出的窗口
bind C-k select-pane -U	#向上
bind C-j select-pane -D	#向下
bind C-h select-pane -L	#向左
bind C-l select-pane -R	#向右
bind b resize-pane -Z	#b全屏
# Enable mouse mode (tmux 2.1 and above)
set -g mouse on

bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel 'xclip -in -selection clipboard' #复制到系统剪切板
bind-key -T copy-mode-vi C-h select-pane -L #复制模式时移动光标 
bind-key -T copy-mode-vi C-j select-pane -D
bind-key -T copy-mode-vi C-k select-pane -U
bind-key -T copy-mode-vi C-l select-pane -R

# set -g prefix C-a
# unbind C-b
# bind C-a send-prefix

set-window-option -g mode-keys vi
```