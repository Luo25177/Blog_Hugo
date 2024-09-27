---
title: "bochs+gdb调试工具链"
date: 2024-05-30 14:37:18
categories: "计算机"
tags: ["bochs", "gdb"]
summary: "bochs+gdb 调试工具链，用于操作系统的开发"
---

## 前言

`bochs` 中自带一个反汇编器，这只能查看可执行文件的反汇编之后的指令，但是如果书写 C 语言的话，就使得调试非常麻烦（无计可施），所以利用 `bochs` 来写操作系统还是挺困难的，同时也没有办法查看各个变量的状态和 `cpu` 的状态。一旦遇到问题就很被动，所以介绍一个 bochs+gdb 的一个调试工具链

## bochs安装与配置

### 安装环境

```bash
sudo apt-get install g++ 
sudo apt-get install make
sudo apt-get install libx11-dev xserver-xorg-dev xorg-dev
```

### 安装 `bochs`

```bash
wget https://udomain.dl.sourceforge.net/project/bochs/bochs/2.6.8/bochs-2.6.8.tar.gz
tar zxvf bochs-2.6.8.tar.gz
cd bochs-2.6.8/
./configure --prefix=/your_path/bochs --enable-gdb-stub --enable-disasm --enable-iodebug --enable-x86-debugger --with-x --with-x11 LDFLAGS='-pthread'
make -j12
make install
```

其中

- `--enable-debugger` 将 Bochs 使用内置的反汇编器进行调试
- `--enable-gdb-stub` 开放端口提供 GDB 远程调试
- `--enable-readline` 使用 Bochs 内置调试器时使用 readline 库提供的自动补全和历史命令功能

### 配置

进入到安装目录中，也就是在 `config` 中指定的路径 `/your_path/bochs`

新建一个配置文件 `bochsrc.disk` ，然后向其中写入

```bash
#第一步，首先设置 Bochs 在运行过程中能够使用的内存，本例为 32MB 
megs: 32 

#第二步，设置对应真实机器的 BIOS 和 VGA BIOS 
romimage: file=/your_path/bochs/share/bochs/BIOS-bochs-latest 
vgaromimage: file=/your_path/bochs/share/bochs/VGABIOS-lgpl-latest 

#第三步，设置 Bochs 所使用的磁盘，软盘的关键字为 floppy。 
#若只有一个软盘，则使用 floppya 即可，若有多个，则为 floppya，floppyb… 
#floppya: 1_44=a.img, status=inserted 

#第四步，选择启动盘符 
#boot: floppy #默认从软盘启动，将其注释 
boot: disk #改为从硬盘启动。我们的任何代码都将直接写在硬盘上，所以不会再有读写软盘的操作 

#第五步，设置日志文件的输出 
log: bochsout.txt 

#第六步，开启或关闭某些功能 
#下面是关闭鼠标，并打开键盘 
mouse: enabled=0 
keyboard: keymap=/your_path/bochs/share/bochs/keymaps/x11-pc-us.map 

# 硬盘设置 
ata0: enabled=1, ioaddr1=0x1f0, ioaddr2=0x3f0, irq=14

# 打开 gdb 调试端口 1234
gdbstub:enabled=1,port=1234,text_base=0,data_base=0,bss_base=0
```

完成之后在目录下输入 `bin/bochs` 测试结果，后面会跳出一个选项，已经有一个默认配置 6 了，所以输入 6，可以发现跳出一个 `bochs` 虚拟机。退出需要在终端输入指令 `exit`

### 创建虚拟硬盘

使用其中的工具 `bximage` 来创建虚拟硬盘，使用 `bin/bximage` 来创建

- `bin/bximage --help` 查看各个选项参数的含义
- `bin/bximage -mode="create" -hd=60 -imgmode="flat" -q hd60M.img` 创建虚拟硬盘，将出现的最后依据加入到配置文件 `bochsrc` 中，把硬盘设置那一句换掉，然后可以运行测试

之后运行测试都使用 `bin/bochs -f bochsrc.disk` 来运行，但是应该会出现一些无法读取或者写入的错误，这时候就需要使用 `sudo bin/bochs -f bochsrc.disk` 执行

### 安装编译工具

由于使用的是 `nasm` ，所以下载对应的工具

```bash
sudo apt install nasm
```

### bochs的调试

对于要写一个操作系统来说，不能调试可太麻烦了，所以可以使用 bochs 的调试功能，它的调试指令基本和 gdb 类似，但也有不同之处，可以自己上网查一查手册

其次有个配置在代码里可以实现断点功能的，在 `bochsrc.disk` 中添加下面这句话

```nasm
 magic_break: enabled=1
```

然后在代码中想要暂停的地方可以写下，就会在这条语句中停止，然后就可以开始调试

```nasm
xchg bx, bx
```

但是这个很不实用，只能调试汇编语言，所以下面使用 gdb 就可以调试 c 语言

## 编译配置

对于编译方面，需要将所有的编译选项加上 `-g` ，并且对链接之后得到的可执行目标文件 `kernel.bin` 提取符号，得到 `kernel.sym` ，如下

```bash
 objcopy --only-keep-debug $(KERNEL_BUILD_PATH)/kernel.bin $(KERNEL_BUILD_PATH)/kernel.sym
```

## gdb 配置

上述配置完成之后，使用 `bin/bochs -f bochsrc.disk` 指令进入运行之后会卡死，会等待 `gdb` 链接端口进行调试。这时候打开 `gdb` 输入如下指令之后就可以进行调试了

```bash
target remote localhost:1234
symbol-file ./build/kernel/kernel.sym
```

但是还是有点麻烦，所以这时候可以创建一个 `.gdbinit` 来做 `gdb` 的初始化，实际上就是在打开 `gdb` 时会自动执行其中的指令，从而完成配置。但是需要在 `/home/user` 之下创建一个 `gdbinit` 文件来使得将 `,gdbinit` 链接到期望的 `.gdbinit` 。具体操作可以在当前文件夹下运行一下 `gdb` 就可以得到来自于 `gdb` 的教程了

配置结束之后就可以使用 `gdb` 来进行调试了，调试的指令与 `gdb` 一致
