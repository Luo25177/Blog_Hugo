---
title: "gdb使用"
date: 2024-03-19 00:34:41
categories: "小工具"
tags: ["gdb"]
summary: "GNU发行的调试器GDB的基本使用方法"
---

## 前言

### gdb 功能

- 动态改变程序的执行环境。
- 自定义启动运行需要调试的程序。
- 在指定位置使用条件表达式设置断点。
- 在程序暂停时观察代码内变量值的变化。

### 开始前准备

主要是生成调试信息，在编译的时候，添加 `-g` 选项来生成调试信息，以此来使用 gdb 的调试

## gdb 外部指令

- `gdb 可执行文件名` 启动 gdb 调试
- `gdb --help` 查看指令帮助
- `man gdb` 查看 gdb 手册
- `gdb -x xxx` 在 gdb 启动时运行一些脚本
- `gdb --version` 查看 gdb 版本

## gdb 内部指令

### 开始运行

- `r/run` 开始运行，会一直运行，直到断点
- `start` 开始运行，在 `main` 处会停下来
- `starti` 开始运行，在第一条指令处停下来
- `r/run param` 开始运行，会一直运行，直到断点，后面的 `param` 就是可执行文件后面需要跟的参数

### 运行中指令

- `c/continue` 继续运行，一直运行，直到下一个断点
- `c/continue n` 继续运行，一直运行，会忽略 `n` 个断点
- `s/step` 单步，不进入函数内部
- `si/stepi` 单步进入，进入函数内部，是在机器层面的，单步到下一个机器指令
- `n/next` 单步，进入函数内部，是源码层面的下一行
- `ni/nexti` 单步，是机器指令层面的，单步到下一个机器指令
- `finish` 函数会继续执行完，并且打印返回值，然后等待接下来的指令
- `return` 函数不会继续执行下面的指令，直接返回
- `return expression` 直接返回，指定返回值为 `expression`
- `u` 退出当前循环体
- `step, next, stepi, nexti` 后面加次数都表示执行一定次数该指令

### 调试中变量相关

- `info args` 打印当前函数的参数名及值
- `info locals` 打印当前函数中所有局部变量及其值
- `set var arg=val` 设置变量的值
- `ptype arg` 查看 `arg` 的类型
- `disable mem addr` 禁用 `mem` 命令定义的内存区域
- `enable mem addr` 启用 `mem` 命令定义的内存区域
- `wha/whatis arg` 显示变量的类型
- `show arg` 可以显示历史中变量的最后十个值

### 断点和监视点

**断点**

- `b func_name` 在函数处添加断点
- `b *func_name` 在函数处添加断点，这里使用 `*` 的主要是为了使断点设置到汇编语言层的函数开头
- `b +n` 在当前位置的 `n` 个偏移处添加断点
- `b *addr` 在地址 `addr` 处添加断点
- `b filename: n` 在文件 `filename` 的 `n` 行号处添加断点
- `b filename:func_name` 在文件 `filename` 的 `func_name` 函数处添加断点
- `info b` 查看断点信息
- `ignore n nums` 忽略 `n` 号断点 `nums` 次
- `disable n` 临时禁用 `n` 号断点
- `enable n` 重新启用 `n` 号断点
- `enable n once` 重新启用 `n` 号断点，中断一次之后禁用断点
- `enable n delete` 重新启用 `n` 号断点，中断一次之后删除断点
- `clear func_name/n/*addr/filename:n/filename:function` 删除已经指定的断点
- `break point if condition` 测试给定条件，如果条件为真就暂停运行
- `condition n` 删除指定断点的出发条件
- `condition n con` 为指定断点添加出发条件

**断点命令**

可以定义在断点中断后执行的命令，一般使用要在定义断点之后执行

```shell
b main
commands n
p *b
end
```

其中 `n` 就是断点号，也就是在对应断点号之后执行命令， `command` 需要与 `end` 对应结束

**监视点**

- `watch expression` 表达式发生变化时暂停运行
- `awatch expression` 表达式被访问/被改变时暂停运行
- `rwatch expression` 表达式被访问时暂停运行
- `info watchpoints` 查看当前设置的观察点
- `delete` 删除所有断点和监视点
- `delete n` 删除 `n` 号断点或者监视点

### 窗口指令

- `layout src` 显示源码
- `layout asm` 显示汇编代码
- `layout split` 显示源代码和汇编代码
- `layout regs` 显示寄存器
- `layout next` 切换到下一个布局模式
- `layout prev` 切换到上一个布局模式
- `info win` 显示窗口大小
- `focus next` 聚焦到下一个窗口
- `focus prev` 聚焦到上一个窗口
- `focus cmd` 聚焦到命令行
- `focus src` 聚焦到源码窗口
- `focus asm` 聚焦到汇编代码窗口
- `focus regs` 聚焦到寄存器窗口
- `refresh` 刷新所有窗口
- `winheight name +/- line` 调整 `name` 窗口高度
  - `reg` 寄存器
  - `cmd` 命令行
  - `src` 源码
  - `asm` 汇编代码
- `update` 更新源代码窗口和当前执行点
- `ctrl-l` 刷新窗口
- `ctrl-x + n` 也就是先按下 `ctrl-x` 然后再按 `n`
  - `1` 单窗口模式，显示一个窗口
  - `2` 双窗口模式，显示两个窗口
  - `a` 回到传统模式，也就是退出 `layout`

### 寄存器指令

- `i/info reg/register` 显示寄存器的值，不包括浮点寄存器和向量寄存器的内容
- `i/info all-register/reg` 显示所有寄存器的内容
- `i regsiter/reg reg_name` 显示 `reg_name` 寄存器的内容
- `p/print $reg` 显示对应寄存器 `reg` 的值
- `set var $reg=n` 设置寄存器 `reg` 的值为 `n`

### 栈帧操作

- `bt/backtrace` 查看栈回溯信息，执行该栈回溯命令后，会显示程序执行到什么位置，包含哪些帧等信息。每一帧都有一个编号，从 0 开始。0 表示当前正在执行的函数，1 表示调用当前函数的函数，以此类推。栈回溯是倒序排列的
- `bt/backtrace n` 只显示开头 `n` 个栈帧
- `bt/backtrace -n` 只显示最后 `n` 个栈帧
- `bt/backtrace full` 打印栈帧并打印局部变量的值
- `bt/backtrace full n` 打印开头的 `n` 个栈帧并打印局部变量的值
- `bt/backtrace full -n` 打印结尾的 `n` 个栈帧并打印局部变量的值
- `f/frame n` 切换栈帧为 `n` 号栈帧
- `f/frame addr` 切换为栈帧地址 `addr`
- `up n` 切换栈帧为向上 `n` 个栈帧，跳到上一层函数
- `down n` 切换栈帧为向下 `n` 个栈帧，跳到下一层函数
- `info frame` 显示栈帧的信息
- `info frame n` 显示 `n` 号栈帧的信息
- `info stack` 查看堆栈使用情况
- `whe/where` 查看调用的堆栈
- `disas/disassemble` 反汇编当前的函数或者指定函数

### 显示指令

- `p/print /格式 arg/$reg` 用于指定显示格式
- `x /格式 addr` 用指定格式显示 `addr` 内存中的内容
- 格式
  - `x` 显示为 16 进制数字
  - `d` 显示为 10 进制数字
  - `u` 显示为无符号 10 进制数字
  - `o` 显示为 8 进制数字
  - `t` 显示为 2 进制数字 two
  - `a` 显示为地址
  - `c` 显示为字符
  - `f` 显示为浮点小数
  - `s` 显示为字符串
  - `i` 显示为机器语言(仅在显示内存的X命令中可以使用)
- `x /NFU addr` 其中 `N` 表示向下显示多少个， `F` 表示格式，如上， `U` 表示单位大小
  - `b` 字节
  - `h` 半字
  - `w` 字
  - `g` 双字
- `set print elements n` 设置打印的字节长度为 `n` ，其中 `n=0` 时表示不受限制
- `l/list n` 显示代码，默认显示 10 行，指定显示 `n` 行代码
- `display arg/$reg` 跟踪打印变量值或者寄存器值
- `undisplay n` 取消跟踪第 `n` 个跟踪变量的值
- `info display` 查看跟踪的变量号和对应的变量
- `disable display n` 禁用 `n` 号跟踪变量
- `enable display n` 启用 `n` 号跟踪变量

### 反向调试

- `record` 用于开始记录程序反向调试所需要的信息，其中包括保存程序的每一步每一行的结果等信息，而且反向调试只能到执行这条指令的位置，如果没有执行这条指令是没有办法进行反向调试的
- `record stop` 停止反向调试，反向调试的记录停止，就可以开始正常运行了
- `rc/reverse-continue` 开始反向运行，直到碰到一个断点
- `rs/reverse-step` 反向运行到上一次被执行的源代码行，会进入函数
- `rsi/reverse-stepi` 反向运行程序到上一条机器指令
- `rn/reverse-next` 反相运行到上一次被执行的源代码行，不会进入函数
- `rni/reverse-nexti` 反相运行到上一条机器指令
- `reverse-finish` 反向运行程序回到调用当前函数的地方
- `set exec-direction [forward|reverse]` 设置程序运行方向，就可以使用 `stop` 和 `continue` 等指令来执行反向的调试指令

### 抓取和释放进程

- `detach` 将正在调试的进程释放掉，不再进行调试，该进程会继续执行
- `attach pid` 可以调试已经启动的进程，或者调试陷入死循环而无法返回的控制台进程

### 结束

- `q/quit` 退出当前调试
- `k/kill` 杀死当前正在调试的进程，不会退出，可以重新使用开始运行指令开始

### 其它一些操作

- `edit` 编辑文件或者函数
- `find dir xxx` 寻找 `dir` 路径下名为 `xxx` 的文件
- `dir` 用于指定源文件目录
- `print-object arg` 显示目标信息
- `sharelibrary` 加载共享的符号

## gdb 初始化文件

linux 环境下初始化文件为 `.gdbinit` 文件，会在用户的主目录下寻找一个名为 `.gdbinit` 的文件，如果存在该文件，gdb 就会在启动之前将其作为命令文件运行，初始化文件和命令文件运行顺序如下

1. `~/.gdbinit`
2. 运行命令选项
3. `./.gdbinit`
4. 通过 `-x` 选项给出的命令文件 `gdb -x xxx`

该文件的定义应遵循以下格式

```shell
define <command>
<code>
end
document <command>
<help text>
end
```

在 `.gdbinit` 中可以写 gdb 的内部指令，gdb 会执行该指令的

```shell
layout src
```

但是一般来说，当在一个非 `home` 目录下添加一个 `.gdbinit` 时，就必须要在 `home` 目录下添加一个 `.gdbinit` 文件，并且此文件中必须声明

```shell
add-auto-load-safe-path xxx/.gdbinit
```

## 后记

虽然 gdb 调试很强大，但是没有一个交互的页面其实也是有点难受的，但是 gdb 调试适用性很广。现在大部分都可以直接使用 vscode+cmake 来调试，其实很方便，能把所有变量都放在侧边栏以便调试确实很不错。但是丝毫不影响 gdb 的强大。
