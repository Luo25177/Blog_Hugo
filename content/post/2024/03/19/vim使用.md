---
title: "vim使用"
date: 2024-03-19 00:43:47
categories: "小工具"
tags: ["vim"]
summary: "我愿称之为神中之神，心目中的最强文本编辑器"
---

## 前言

作为一个历史上最出色的编辑器，拥有自定义按键功能，并且可以大大提高工作效率

## 在linux中的安装

在linux中只需要一行指令安装

```shell
sudo apt install vim
```

## linux 上的配置文件

```c
" etc/vim/vimrc
" the call to :runtime you can find below.  If you wish to change any of those
" settings, you should do it in this file (/etc/vim/vimrc), since debian.vim
" will be overwritten everytime an upgrade of the vim packages is performed.
" It is recommended to make changes after sourcing debian.vim since it alters
" the value of the 'compatible' option.

runtime! debian.vim

" Vim will load $VIMRUNTIME/defaults.vim if the user does not have a vimrc.
" This happens after /etc/vim/vimrc(.local) are loaded, so it will override
" any settings in these files.
" If you don't want that to happen, uncomment the below line to prevent
" defaults.vim from being loaded.
" let g:skip_defaults_vim = 1

" Uncomment the next line to make Vim more Vi-compatible
" NOTE: debian.vim sets 'nocompatible'.  Setting 'compatible' changes numerous
" options, so any other options should be set AFTER setting 'compatible'.
"set compatible

" Vim5 and later versions support syntax highlighting. Uncommenting the next
" line enables syntax highlighting by default.

if has("syntax")
syntax on
endif

" If using a dark background within the editing area and syntax highlighting
" turn on this option as well
"set background=dark

" Uncomment the following to have Vim jump to the last position when
" reopening a file
"au BufReadPost * if line("'\"") > 1 && line("'\"") <= line("$") | exe "normal! g'\"" | endif

" Uncomment the following to have Vim load indentation rules and plugins
" according to the detected filetype.
"filetype plugin indent on

" The following are commented out as they cause vim to behave a lot
" differently from regular Vi. They are highly recommended though.
"set showcmd		" Show (partial) command in status line.
"set showmatch		" Show matching brackets.
"set ignorecase		" Do case insensitive matching
"set smartcase		" Do smart case matching
"set incsearch		" Incremental search
"set autowrite		" Automatically save before commands like :next and :make
"set hidden		" Hide buffers when they are abandoned
"set mouse=a		" Enable mouse usage (all modes)
" Source a global configuration file if available

autocmd BufNewFile *.py,*.sh,*.java exec":call SetTitleJ()"
func SetTitleJ()
call setline(1,"\#########################################################################")
call append(line("."), "\# File Name: ".expand("%"))
call append(line(".")+1, "\# Author: Dragon")
call append(line(".")+2, "\# mail: [beloved25177@126.com](mailto:beloved25177@126.com)")
call append(line(".")+3, "\# Created Time: ".strftime("%c"))
call append(line(".")+4, "\# Description:" )
call append(line(".")+5, "\#########################################################################")
call append(line(".")+6, "")
endfunc

autocmd BufNewFile *.c,*.cpp,*.h exec":call SetTitleC()"
func SetTitleC()
call setline(1,"\//------------------------------------------------------------------------")
call append(line("."), "\// File Name: ".expand("%"))
call append(line(".")+1, "\// Author: Dragon")
call append(line(".")+2, "\// mail: [beloved25177@126.com](mailto:beloved25177@126.com)")
call append(line(".")+3, "\// Created Time: ".strftime("%c"))
call append(line(".")+4, "\// Description:" )
call append(line(".")+5, "\//------------------------------------------------------------------------")
call append(line(".")+6, "")
endfunc

autocmd BufNewFile *.md exec":call SetTitleM()"
func SetTitleM()
call setline(1,"\<!-- ########################################################################")
call append(line("."), "\File Name: ".expand("%"))
call append(line(".")+1, "\Author: Dragon")
call append(line(".")+2, "\mail: [beloved25177@126.com](mailto:beloved25177@126.com)")
call append(line(".")+3, "\Created Time: ".strftime("%c"))
call append(line(".")+4, "\Description:" )
call append(line(".")+5, "\######################################################################### --!>")
call append(line(".")+6, "")
endfunc

nmap <space>w :w!
nmap L g_
nmap H ^
nmap <space>d dd
nmap <C-n> :nohl
nmap <space>s :w
nmap <C-x> gb
nmap <space>q :q
nmap <space>r <C-r>
nmap gh gt
nmap g+ :!g++<space>%
nmap gc :!gcc<space>%
nmap K Ojj
nmap dL dg_
nmap yL yg_
nmap cL cg_
nmap dH d^
nmap yH y^
nmap cH c^
nmap nm :nohlsearch

vmap L g_
vmap H ^

imap jj <Esc>
imap jk <Esc>

map <C-A> ggVGY
map! <C-A> <Esc>ggVGY

vmap L g_

set autowrite
set ruler                   " 打开状态栏标尺
set cursorline              " 突出显示当前行
set magic                   " 设置魔术
set guioptions-=T           " 隐藏工具栏
set guioptions-=m           " 隐藏菜单栏
syntax on
set tabstop=2
set softtabstop=2
set autoindent
set cindent
set number
set hlsearch
set noexpandtab " 不使用空格代替tab
set smartindent
set ignorecase
set number
set nocompatible
set backspace=2
setlocal noswapfile " 不要生成swap文件
set shiftwidth=2 " 设定 << 和 >> 命令移动时的宽度为 4
set bufhidden=hide " 当buffer被丢弃的时候隐藏它
set backupcopy=yes " 设置备份时的行为为覆盖
set backspace=indent,eol,start " 不设定在插入状态无法用退格键和 Delete 键删除回车符
set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)
set autochdir " 自动切换当前目录为当前文件所在的目录

if filereadable("/etc/vim/vimrc.local")
source /etc/vim/vimrc.local
endif
```

## vim使用

### 移动光标

- `h j k l` 上 下 左 右
- `ctrl-y1` 上移一行
- `ctrl-e1` 下移一行
- `ctrl-u1` 上翻半页
- `ctrl-d1` 下翻半页
- `ctrl-f1` 上翻一页
- `ctrl-b1` 下翻一页
- `w` 跳到下一个词首，按标点或单词分割
- `W` 跳到下一个词首，以空格分割
- `e` 跳到下一个词尾，以标点分割
- `E` 跳到下一个词尾
- `b` 跳到上一个词，以标点分割
- `B` 跳到上一个词首
- `0` 跳至行首，不管有无缩进，就是跳到第0个字符
- `^` 跳至行首的第一个字符
- `$` 跳至行尾
- `gg` 跳至文首
- `G` 调至文尾
- `ngg/nG` 调至第n行
- `gd` 跳至当前光标所在的变量的声明处
- `fx` 在当前行向后找x字符，找到了就跳转到字符位置
- `;` 重复上一个f命令，而不用重复的输入fx
- `*` 查找光标所在处的单词，向下查找
- `#` 查找光标所在处的单词，向上查找
- `g_` 移动到本行最后一个不是 blank 字符的位置

### 删除复制

- `dd` 删除光标所在行，应该说是剪切，删除之后 vim 的缓冲区中会存放删除的内容
- `dw` 删除一个字符
- `diw` 删除一个单词
- `dg_` 或 `D` 从当前位置删除到行末
- `dta` 删除当前行位置之后的所有内容直到遇到字符 `a`
- `dTa` 删除当前行位置之前的所有内容直到遇到字符 `a`
- `x` 删除当前字符
- `X` 删除前一个字符
- `yy` 复制一行，会复制换行符
- `yw` 复制一个字符
- `yiw` 复制一个单词
- `yg_` 或 `Y` 复制到行末
- `p` 粘贴粘贴板的内容到当前光标之前
- `P` 粘贴粘贴板的内容到当前光标之后

### 模式转换

- `i` 从当前光标之前进入插入模式
- `I` 在行首进入插入模式
- `a` 追加模式，当前光标之后进入插入模式
- `A` 在行尾进入插入模式
- `o` 在当前行之下新加一行，并进入插入模式
- `O` 在当前行之上新加一行，并进入插入模式
- `Esc` 退出当前模式
- `v` 进入可视模式 visual mode，进入可视模式之后可以使用在普通模式的按键进行相应的操作
- `V` 进入行可视模式 line visual mode
- `ctrl-v` 进入块可视模式 block visual mode，但是要注意在 windows 中的 ideavim 中，默认是 `ctrl-q`

### 编辑

- `J` 将下一行和当前行连接为一行
- `cc` 删除当前行并进入编辑模式
- `cw` 删除当前字符，并进入编辑模式
- `c$` 擦除从当前位置至行末的内容，并进入编辑模式
- `s` 删除当前字符并进入编辑模式
- `S` 删除光标所在行并进入编辑模式
- `xp` 交换当前字符和下一个字符
- `u` 撤销
- `<C-r>` 返回撤销
- `r` 替换为输入的字符
- `R` 连续替换
- `ctrl+r` 重做
- `guw` 当前字符切换为小写
- `gUw` 当前字符切换为大写
- `guiw` 当前单词切换为小写
- `gUiw` 当前单词切换为大写
- `~` 切换当前字符的大小写
- `>>` 将当前行右移一个tab
- `<<` 将当前行左移一个tab
- `==` 自动缩进当前行
- `gg=G` 文件缩进
- `<C-n>` 插入模式自动补齐
- `<C-p>` 插入模式自动补齐

### 查找替换

- `/pattern` 向后搜索字符串pattern
- `?pattern` 向前搜索字符串pattern
- `"\c"` 忽略大小写
- `"\C"` 大小写敏感
- `n` 下一个匹配(如果是/搜索，则是向下的下一个，?搜索则是向上的下一个)
- `N` 上一个匹配(同上)
- `:%s/old/new/g` 搜索整个文件，将所有的old替换为new
- `:%s/old/new/gc` 搜索整个文件，将所有的old替换为new，每次都要你确认是否替换
- `fa` 查找当前行中下一个为 `a` 的字符，并且跳到该位置
- `nfa` 查找当前行中下 `n` 个为 `a` 的字符，并且跳到该位置
- `ta` 查找当前行中上一个为 `a` 的字符，并且跳到该位置
- `nta` 查找当前行中上 `n` 个为 `a` 的字符，并且跳到该位置

### 区域选择

- `<action>a<object>` 区域选择
- `<action>i<object>` 区域选择
  - `action` 可以是任何命令
    - `d` 删除
    - `v` 选中
    - `y` 复制
  - `object` 可以是很多，下面是一部分
    - `w` 一个单词
    - `W` 一个以空格为分隔的单词
    - `s` 一个句子
    - `p` 一个段落
    - `"` 引号中内容
    - `(` 括号中内容
- `viw` 选中当前单词
- `vi"` 选中当前在 `"` 中的内容，不包括 `"`
- `va"` 选中当前在 `"` 中的内容，包括 `"`
- `vni(` 选中当前在 `n` 级括号中的内容，不包括括号
- `vna(` 选中当前在 `n` 级括号中的内容，包括括号

### 退出文件

- `:w` 将缓冲区写入文件，即保存修改
- `:wq` 保存修改并退出
- `:wq!` 强制保存修改并退出
- `:x` 保存修改并退出
- `:q` 退出，如果对缓冲区进行过修改，则会提示
- `:q!` 强制退出，放弃修改
- `ZZ` 保存退出

### 多文件编辑（多窗口操作）

- `vim file1..` 同时打开多个文件
- `:args` 显示当前编辑文件
- `:next` 切换到下个文件
- `:prev` 切换到前个文件
- `:next!` 不保存当前编辑文件并切换到下个文件
- `:prev!` 不保存当前编辑文件并切换到上个文件
- `:wnext` 保存当前编辑文件并切换到下个文件
- `:wprev` 保存当前编辑文件并切换到上个文件
- `:first` 定位首文件
- `:last` 定位尾文件
- `ctrl+^` 快速在最近打开的两个文件间切换
- `:split` 或者 `:sp` 把当前文件水平分割
- `:vsplit` 或者 `:vsp` 把当前文件垂直分割
- `:split file` 把当前窗口水平分割，并且打开文件file
- `:vsplit[vsp] file` 把当前窗口垂直分割，并且打开文件file
- `:new file` 同 `split file`
- `:close` 关闭当前窗口
- `:only` 只显示当前窗口, 关闭所有其他的窗口
- `:all` 打开所有的窗口
- `:vertical all` 打开所有的窗口, 垂直打开
- `:qall` 对所有窗口执行：q操作
- `:qall!` 对所有窗口执行：q!操作
- `:wall` 对所有窗口执行：w操作
- `:wqall` 对所有窗口执行：wq操作
- `ctrl-w h` 跳转到左边的窗口
- `ctrl-w j` 跳转到下面的窗口
- `ctrl-w k` 跳转到上面的窗口
- `ctrl-w l` 跳转到右边的窗口
- `ctrl-w t` 跳转到最顶上的窗口
- `ctrl-w b` 跳转到最底下的窗口
- `ctrl-w +` 扩大窗口
- `ctrl-w -` 缩小窗口
- `ctrl-w H` 将窗口移动到最左边
- `ctrl-w J` 将窗口移动到最下边
- `ctrl-w K` 将窗口移动到最上边
- `ctrl-w L` 将窗口移动到最右边
- `{height}CTRL-W _` 将窗口设置成指定高度
- `:saveas <path/to/file>` 另存为 `<path/to/file>`
- `:bn` 下一个文件
- `:bp` 上一个文件
- `:all` 为参数列表中每一个文件打开一个窗户口
- `:vertical all` 以垂直分割的方法打开窗口

**命令行中vim指令的参数**

- `vim -o <file1 file2 ...>` 以水平分屏的方式打开多个文件
- `vim -O <file1 file2 ...>` 以垂直分屏的方式打开多个文件

### 多标签编辑

- `:tabedit file` 在新标签中打开文件file
- `:tab split file` 在新标签中打开文件file
- `:tabp` 切换到前一个标签
- `:tabn` 切换到后一个标签
- `:tabc` 关闭当前标签
- `:tabo` 关闭其他标签
- `gt` 到下一个tab
- `gT` 到上一个tab

### buffer操作

- `-` 非活动的缓冲区
- `=` 只读缓冲区
- `a` 当前被激活缓冲区
- `h` 隐藏的缓冲区
- `%` 当前的缓冲区
- `#` 交换缓冲区
- `+` 已经更改的缓冲区

### 文件操作

- `vim path/` 打开目录
- `h j k l` 左，下，上，右
- `-` 返回上一目录
- `enter` 进入目录或者文件
- `D` 删除目录或文件
- `R` 重命名目录或文件
- `ctrl+^` 回退上次操作
- `:q` 退出vim
- `c` 使当前打开的目录成为当前目录
- `d` 创建目录
- `%` 创建文件
- `gb` 转到上一个标记的文件夹
- `i` 改变目标文件列表方式时
- `^i` 刷新当前打开的目录，更改排列样式
- `mf` 标记文件
- `mu` unmark all marked files
- `mz` compress/decompress marked file
- `gh` 显示/不显示隐藏文件
- `^h` 标记隐藏文件列表
- `a` 转换显示模式，显示所有文件-只显示不隐藏文件-只显示隐藏文件
- `qf` 显示文件信息
- `qb` 列出被标记的文件历史
- `gi` 显示文件信息
- `md` 将标记的文件使用 `diff` 模式
- `me` 编辑标记的文件，只显示一个，其余放入 `buffer` 中
- `mm` 移动标记的文件到标记的文件目标路径中
- `mc` copy
- `mt` 移动到的目录

**复制移动文件**

1. `mt` 移动到的目录
2. `mf` 标记要移动的文件
3. `mc` 移动/复制

### 宏录制

- `qa` 把操作记录在寄存器 `a` 中
- `@a` 从寄存器 `a` 中读取记录，并且 replay
- `@@` 取出最新的录制记录，并且 replay
- `n@@` 取出最新的录制记录，并且执行 `n` 次 replay

### 文件差异对比

- `vimdiff file1 file2 [file3 [file4]]` 文件对比，会以垂直分割的方式打开文件
- `vim -d file1 file2 [file3 [file4]]` 文件对比，会以垂直分割的方式打开文件
  - 折叠的行用 `+ +--n lines: /* a` 表示+ +--123 lines: /* a
  - 被删除的行用 `---` 表示
  - 不同的的行会用高亮显示
- `:diffsplit file` 打开文件 `file` 并且进行差异对比
- `:diffupdate` 把文本从一个窗口移到另一个，并以此来消除差异
- `:dp` 即 `:diff put` 将把文字从左边拷到右边，从而消除两边的差异
