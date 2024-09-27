---
title: "ubuntu指令"
date: 2024-03-19 00:48:52
categories: "小工具"
tags: ["Linux"]
summary: "ubuntu一些常用的总结的指令"
---

## ubuntu关于安装的指令

`bus restart`重置输入法

`sudo apt search package` 搜索包

`sudo apt show package` 获取包的相关信息，如说明、大小、版本等  

`sudo apt depends package` 了解使用依赖  

`sudo apt rdepends package` 查看该包被哪些包依赖  

`sudo apt-cache pkgnames` 执行pkgnames子命令列出当前所有可用的软件包 

`sudo apt policy package` 使用policy命令显示软件包的安装状态和版本信息。

`sudo apt install package` 安装包  

`sudo apt install package=version` 安装指定版本的包  

`sudo apt install package --reinstall` 重新安装包  

`sudo apt -f install` 修复安装 `-f = --fix-missing`  

`sudo apt remove package` 删除包

`sudo apt purge package` 删除包，包括删除配置文件等

`sudo apt autoremove` 自动卸载所有未使用的软件包

`sudo apt source package` 下载该包的源代码   

`sudo apt update` 更新apt软件源信息  

`sudo apt upgrade` 更新已安装的包

`sudo apt full-upgrade` 在升级软件包时自动处理依赖关系  

`sudo apt dist-upgrade` 升级系统  

`sudo apt dselect-upgrade` 使用dselect升级  

`sudo apt build-dep package` 安装相关的编译环境  

`sudo apt clean && sudo apt autoclean` 清理无用的包

`sudo apt clean` 清理已下载的软件包，实际上是清除 `/var/cache/apt/archives` 目录中的软件包

`sudo apt autoclean` 删除已经卸载的软件包备份  

`sudo apt-get check` 检查是否有损坏的依赖

`sudo cp src dst` 复制文件到目标路径

`sudo rm -r filename` 删除文件 

`sudo mv src dst` 移动文件

`bash filename/filename.sh` 运行filename shell脚本文件

`lsb_realse -a` 查看ubuntu版本

`ls -l` 查看文件信息

`ls -lh` 查看文件大小

`du -sh` 查看文件夹大小

`unzip filename.zip` 解压文件

`lsb_release -c` 查看自己的版本，如果与镜像源的版本不一致不能够下载

## 镜像版本

### focal
    
deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal main restricted universe multiverse

deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-security main restricted universe multiverse

deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-updates main restricted universe multiverse

deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-proposed main restricted universe multiverse

deb [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-backports main restricted universe multiverse

deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal main restricted universe multiverse

deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-security main restricted universe multiverse

deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-updates main restricted universe multiverse

deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-proposed main restricted universe multiverse

deb-src [http://mirrors.aliyun.com/ubuntu/](http://mirrors.aliyun.com/ubuntu/) focal-backports main restricted universe multiverse
    
### bionic
### eoan


## ubuntu 终端的 ctrl 指令

- Ctrl + B 向前移动
- Ctrl + F 向后移动
- Ctrl + -> 向左跳跃单词
- Ctrl + -> 向右跳跃单词
- Ctrl + U 删除光标左边的全部内容
- Ctrl + K 删除光标右边的全部内容
- Ctrl + H 删除光标前的一个字符
- Ctrl + T 将光标右边的一个字符向后移动，一直按一直后移
- Ctrl + L 将最新的命令移动到终端最上面
- Ctrl + P 使用之前的命令
- Ctrl + A 直接跳到命令的开头
- Ctrl + E 直接跳到命令的结尾

## ubuntu 终端可以做的一些配置

- fish/zsh
- vim
- gcc
- cmake
- ctags
- tmux

## linux终端指令

1. `crtl-l` 清屏
2. `crtl-q` 清除正在写的一行
3. `crtl-w` 删除正在写的一行的一个单词
通过命令行得到统计结果
`./a.out | head -n 1000 | sort | uniq -c`
使用 `gcc -static minhello.c` (静态编译)
使用 `objdump -d a.o`(出现的是预编译代码) 查看会发现 a.o 的代码量很大
使用 `ls -l a.out` 来查看 a.out 的大小，显示的数字就是代码的内存大小B
可以使用 `size a.out` 来查看代码的大小
`rr record ./a.out` 记录a.out 的指令
`timeout 3 ./a.out` 表示执行3秒该程序
`env` 查看应用程序执行的环境变量 `env | grep path`
    - PATH 可执行文件的搜索路径
    - PWD 当前路径
    - HOME home目录
    - DISPLAY 图形输出
    - PS1 shell指示符 
    export 设置环境变量
    使用gcc静态编译的时候会舍去动态链接相关的系统调用
4. `head` 指令
显示输出或者是显示读取的文件，可以通过通道来输出，也可以直接指令后面跟文件名，输出文件内容
    - `head -n num`
    显示num行
    - `head -c num`
    显示num个字节
5.  `uniq` 指令，只能用于判断相邻的两行是否相同来计算
    - `uniq -c` 计算重复次数
    - `uniq -d` 仅显示重复的行
    - `uniq -u` 仅显示不重复的行
    - `uniq -s` 选项用于忽略比较的字符， 它将忽略指定数量的字符并将结果显示到标准输出
6.   `cat` 指令，用于查看文件内容
7.  `free`查看系统的内存
    - -b 以byte为单位显示内存使用情况
    - - k 以kb为单位显示内存使用情况
    - -m 以mb为单位显示内存使用情况
    - -h 以合适的单位显示内存使用情况
8. 通道和多条命令 | 
9. 重定向 > <
    
    
    | command > file  | 将输出重定向到file                               |
    | --------------- | ------------------------------------------------ |
    | command < file  | 将输入重定向到file                               |
    | command >> file | 将输出以追加的方式重定向到file                   |
    | n > file        | 将文件描述为n的文件方重定向到file                |
    | n >> file       | 将文件描述为n的文件以追加的方重定向到file        |
    | n >& m          | 将输出文件m和n合并                               |
    | n <& m          | 将输入文件m和n合并                               |
    | << tag          | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |
    | n > m           | 重定向正确输出                                   |
    | n 2> m          | 重定向错误输出                                   |
    | n &> m          | 重定向所有输出                                   |
    
    `echo hello > /dev/pts/1` 可以把输出重定向到另一个终端
    
10. 后台 &
11. `less/more`  用于查看内容 
    - Up arrow – 向上移动一行
    - Down arrow – 向下移动一行
    - Space 或者 PgDn – 向下移动一页
    - b 或者 PgUp – Move one page up
    - g – 移到文件的开头
    - G – 移动到文件的末尾
    - ng – 移到第n行
    - -N - 显示行号
    
    文本中指令：
    
    1. /+content 查找对应的内容
12. `wc` 指令，用于查看文件
    - l , --lines : 显示行数；
    - w , --words : 显示字数；
    - m , --chars : 显示字符数；
    - c , --bytes : 显示字节数；
    - L , --max-line-length : 显示最长行的长度；
13. `$(), <()`预处理
14. `tail` 指令
    
    可以查看文件的内容默认显示文件的最后10行，格式： `tail [param] [filename]`
    
    - -f 循环读取
    - -q 不显示处理信息
    - -v 显示详细的处理信息
    - -c<num> 显示的字节数
    - -n<num> 显示文件的尾部n行内容
    - --pid=PID 与-f 一起使用，表示在进程ID为PID的进程死掉之后结束
    - -q, --quiet, --silent 从不输出给出文件名的首部
    - -s, --sleep-interval=S 与-f一起使用，表示在每次反复的间隔休眠S秒
15. `strace` 指令
    
    在linux系统中，strace命令是一个集诊断，调试，统计于一体的工具，可以与其他命令搭配使用 查看手册 `man strace`
    
    **安装**
    
    CentOS/EulerOS`yum install strace`
    
    Ubuntu `apt-get install strace -y`
    
    **指令格式**
    
    `strace [param] [api]`
    
    **参数**
    
    - -c 统计每一个系统调用的所执行时间，次数和出错的次数等
    - -d 显示有关标准错误的strace本身的一些调试输出
    - -f 跟踪子进程，这些进程是由 `fork` 创建的
    - -i 在系统调用时打印指令指针
    - -t 跟踪的每一行都以时间为前缀
    - -tt 打印的时间包括微秒
    - -ttt 打印时间包括微秒，并且前导部分将打印为自该**以来的秒数
    - -T 显示花费在系统调用上的时间，记录每一个系统调用的开始和结束之间的时间差
    - -v 打印环境，统计信息，termios等调用的未缩写的版本，可获取所有详细信息
    - -V 打印strace的版本号
    - -e expr 限定表达式，用于修改跟踪的时间或者如何跟踪它们
    - -e trace=set 只跟踪指定的系统调用集，-c选项可以确定跟踪那些系统调用
        - -e trace=file 跟踪所有以文件名作为参数的系统调用。
        - -e trace=process 跟踪涉及过程管理的所有系统调用。
        - -e trace=network 跟踪所有与网络相关的系统调用。
        - -e trace=signal 跟踪所有与信号相关的系统调用。
        - -e trace=ipc 跟踪所有与IPC相关的系统调用。
    - -o 文件名 将跟踪输出写入文件名除了stderr标准错误
    - -p pid 使用进程id为pid 并且以pid的进程号开始跟踪，可以随时中断
    - -S 按指定条件对-c选项打印的直方图输出进行排序。
16. `linux的进程管理指令`
    - nohup
        
        不挂断地运行命令
        
        语法： `nohup command [args] [&]`
        
        默认情况下，当前目录会有一个 `nohup.out`文件，如果当前的目录不可写，那就定向输出到 `$HOME/nohup.out` 文件中，如果没有文件能创建或者打开以用于追加输出，那么 `command`指令不可调用
        
        **参数**
        
        command：需要执行的指令
        
        args： 参数，可以指定输出文件
        
        &：让命令可以后台运行，并且终端退出之后依旧可以运行
        
        2>&1 将标准错误定向输出到标准输出&1
        
    - &
        
        用在一个指令最后，可以把命令放到后台执行
        
    - jobs
        
        用于查看正在执行的后台进程，但是只能看到当前终端启动的进程，如果关闭当前终端之后，另一个终端下jobs无法看到该进程
        
        - -l 显示进程号
        - -p 仅任务对应的显示进程号
        - -n 显示任务状态的变化
        - -r 仅输出运行状态running的任务
        - -s 仅输出停止状态stoped的任务
    - ps
        
        用于查看当前系统运行的进程信息，操作字可以叠加使用
        
        - a 显示所有程序
        - x 显示所有程序，不区分终端机
        - u 以用户为主的格式来显示
        - -f 显示程序间的关系
        - -e 显示所有程序
        - aux 观察系统所有进程数据
        - -ef 显示所有进程的基本信息
    - kill
        - `kill -9 pid` 杀掉进程，不做善后工作
        - `kill -15 pid` 调用destory等方法善后
        
        优先使用-15
        
    - ctrl+c 停止当前正在执行的进程，相当于直接kill掉
    - ctrl+z 将当前正在执行的进程放到后台，并且暂停执行，此时进程处于stop状态
    - fg 将后台中的进程调至前台继续运行。如果后台中有多个命令，可以用fg %jobnumbe将选中的命令调出，%jobnumber是通过jobs命令查到的后台任务的编号，不是进程的pid号
    - bg 将一个在后台暂停的命令，变成继续执行。如果后台中有多个命令，可以用bg %jobnumber将选中的命令调出。
17. `pmap linux的进程内存分析` 显示的是进程
    - -x extended 显示扩展格式
        - Address:  start address ofmap  映像起始地址
        - Kbytes: size of map in kilobytes  映像大小
        - RSS:  resident set size inkilobytes  驻留集大小
        - Dirty:  dirty pages (both sharedand private) in kilobytes  脏页大小
        - Mode:  permissions on map 映像权限
        - Mapping: 映像支持文件,[anon]为已分配内存[stack]为程序堆栈
        - Offset: offset into the file  文件偏移
        - Device:  device name(major:minor)  设备名
    - -q quiet 显示设备格式
    - -d device 不显示首尾行
    - -V 版本号
18. `pidof` 查找某个进程的进程号
19. `env` 打印出所有的环境变量，手册查看 `man 7 environ`
    
    其中定义了一个符号 `extern char **environ;`
    
20. 使用 `top` 指令查看cpu占有率
21. `chmod` 指令，修改文件权限
    
    格式 `chmod [command] filename`
    
    command 格式 `[ugoa...][+-=][rwxX....][...]`
    
    - u 表示文件拥有者，g 表示与文件拥有者属于同一群体的，o 表示其他以外的人，a 表示三者都可以
    - + 表示增加权限，- 表示取消权限，= 表示设定权限
    - r 可读，w 可写，x 可执行，X表示只有当该文件是个子目录或者该文件已经被设定过为可执行
    - -c 如果该文件权限确实已经更改，才显示其更改动作
    - -f 若该文件权限无法更改也不显示错误
    - -v 显示权限变更的详细资料
    - -R 对当前目录下的所有文件与子目录进行相同的权限变更（以递归的方式逐个变更）
    - —help 辅助说明
    - —version 显示版本
22. `Binutils`工具集**(**[Binutils](https://www.gnu.org/software/binutils/)**)**
    1. `ld` GNU 链接器
    2. `as` GNU 汇编器 汇编代码→可执行文件
    3. `gold` 一种新的、更快的、仅限ELF的连接器。
    - `objdump` 查看目标文件或者可执行的目标文件的构成的gcc工具，反汇编命令
        - -a 显示文件头信息
        - -b 指定目标对象格式为BFDNAME
        - -C 将C++符号名逆向解析
        - -d 将代码段反汇编，显示可执行的部分的汇编内容
        - -D 将所有的部分反汇编
        - -S 将代码段反汇编的同时，将反汇编代码与源代码交替显示，但是需要 -g 编译参数，即需要调试信息
        - -s 显示所有请求部分的完整内容
        - -g 显示目标文件中的调试信息
        - -e 使用C标签样式显示调试信息
        - -G 显示(以原始形式)文件中的任何stab信息
        - -l 反汇编代码中插入文件名和行号，在输出中包括行号和文件名
        - -j section 仅反汇编指定的部分
        - -r 重定位条目显示
        - -R 动态重定位的条目显示
        - -h 显示指定部分的文件头，ELF头
        - -x 显示所有的文件头
        - -t 显示符号表的内容
        - -T 显示动态链接的符号表的内容
        - -i 列出文件支持的对象格式和体系结构
        - -f 显示整个文件头的内容
        - -F 在显示信息时包括文件偏移量
        - -I 添加DIR到源文件的搜索列表
        - -p 显示对象格式特定的文件头内容
        - -P 显示对象格式特定的内容
    - `objcopy` 可用于创建一个有特殊排列和格式的目标文件或者可执行文件。  `objcopy` 也可以将目标文件和可执行文件中的某些部分拷贝到新的文件中，或创建一个空的目标文件或可执行文件，复制二进制文件，并且可以在过程中变换
        - -I 输入文件的格式为<bfdname>
        - -O 创建格式为<bfdname>的输出文件
        - -B 当输出无arch时，输出arch
        - -F 设置输入输出文件格式都是 <bfdname>
        - -P 将修改的/访问的时间戳复制到输出
        - -D 在剥离档案时产生确定性输出
        - -j 仅复制 <name> 部分到输出
        - -R 从输出中移除 <name> 部分
        - -S 从输出中移除所有的标志和重定位信息
        - -g 移除所有的调试信息和选定部分
        - -N 不复制 <name> 标志
        - -K 不要剥离符号<name>以上翻译结果来自有道神经网络翻译（YNMT）· 通用场景
        - -L 强制符号<name>标记为本地
        - -G 本地化除<name>以外的所有符号
        - -W 强制符号<name>标记为弱值
        - -w 允许在符号比较中使用通配符
        - -x 删除所有非全局符号
        - -X 删除任何编译器生成的符号
        - -i 只从每个<number>字节中复制N
        - -b 在每个交错块中选择字节<num>
    - `nm` 文本分析工具 来源于name的简写，用于列出指定文件中的符号（如常用的函数名，变量等，以及这些符号存储的区域）。显示指定文件中的符号信息，文件可以是对象文件，可执行文件或对象文件库。如果文件中没有包含符号信息，nm报告该情况，不把它解释为错误。
        - -A/-o/--print-file-name：输出时加上文件名
        - -a/—debug-syms：输出所有符号，包含 `debugger-only symbols`
        - -B/—format=bsd：bsd码显示，兼容 MIPS nm
        - -C/—demangle：将低级符号名解析为用户级名字，可以使得C++函数名更具备可读性
        - -D/—dynamic：显示动态符号，该选项只对动态目标（例如特定的共享库）有意义
        - -f format/—format=format：默认使用format格式输出。format可以选取 bsd，sysv 或 posix，默认为bsd
        - -g/—extern-only：只显示外部符号
        - l/–line-numbers: 对于每个符号，使用debug信息找到文件名和行号
        - -n/-v/–numeric-sort: 按符号对应地址的顺序排序，而非按符号名字字符顺序排序
        - -P/–portability: 按照POSIX2.0标准格式输出，等同于使用 -f posix
        - -p/–no-sort: 按照目标文件中遇到的符号顺序显示，不排序
        - -r/–reverse-sort: 反转排序
        - -s/–print-armap： 当列出库成员符号时，包含索引。索引的内容：模块和其包含名字的映射
        - -u/–undefined-only： 只显示未定义符号
        - –defined-only: 只显示定义了的符号
    - `addr2line` 将地址转换为文件名和行号
    - `ar` 一个用于创建、修改和提取档案的实用程序
    - `c++filt` 过滤器，过滤出要求编码的c++符号
    - `dlltool` 创建用于构建和使用dll的文件
    - `elfedit` 允许更改ELF格式文件
    - `gprof` （[gprof使用和介绍](https://blog.csdn.net/kentyu001/article/details/53886493)）可以显示程序运行的“flat profile”，包括每个函数的调用次数，每个函数消耗的处理器时间。也可以显示“调用图”，包括函数的调用关系，每个函数调用花费了多少时间。还可以显示“注释的源代码”，是程序源代码的一个复本，标记有程序中每行代码的执行次数。需要使用 -pg 的编译和链接选项
        
        编译完成之后需要先运行一下，程序会把运行的信息存储到 gmon.out 文件中，然后gprof 对该文件进行解析
        
    - `gprofng`
    - `nlmconv`
    - `ranlib`
    - `readelf`
        - -S 对某个重定位目标文件的节头表进行解析
    - `size`
    - `strings`
    - `strip`
    - `windmc`
    - `windres`
23. `pstree` 打开一个文件扫描目录
    - -p 打印每一个进程的进程号
    - -n 按照pid从小到大输出一个进程的孩子
24. `grep+name` 过滤缓冲区，获得想要看到的内容所在的行的内容 
25. `make clean` 删除编译生成的文件
26. `strip` 丢弃目标文件中的符号
27. `time` 指令，显示程序运行的时间
28. `taskset -c 0,...` 表示把进程分配到几个cpu上
29. `stat` 指令，显示文件或者文件系统的状态参数等
30. `ulimit`
    - -a 查看当前所有限制
    - -c 设置最大core文件的大小，默认是0
    - -d 进程数据的最大段
    - -f 被 shell 创建的最大文件的大小限制
    - -l 被锁进内存的最大限制
    - -m 最大常驻尺寸集
    - -n 打开文件表述符最大数量限制
    - -s 最大栈尺寸
    - -t 一秒钟中最大 CPU 时间
    - -u 对单个用户最大数量的可执行的进程
    - -v  shell 可使用的最大虚拟内存的大小
31. lsmod
32. insmod
33. mknod
34. lsblk 查看所有设备的挂载点
35. bash xx.sh 可以运行在bash中运行的脚本，xx.sh 内容就相当于是输入命令行的指令
36. mkfs.fat 格式化文件
37. hd hexdump 实用程序是一个过滤器，它以用户指定的格式显示指定的文件或标准输入(如果没有指定文件)
38. 根目录 /proc 下有一些系统定义的结构体，可以查看内核数据结构的内容
39. ubuntu 安装驱动 `sudo ubuntu-drivers autoinstall`
40. `df -h` 查看磁盘使用情况
41. `sort -nk num` 排序
    1. `n` 就是对应的数据当作数字来排序
    2. `k` 表示按照列来排序
    3. `num` 就是指的第 `num` 列
42. `while true; do ./a.out; done` 指令就是不断执行 `./a.out` 程序
