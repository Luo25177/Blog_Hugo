---
title: "makefile编写"
date: 2024-04-02 21:28:03
categories: "小工具"
tags: ["makefile"]
summary: "makefile编写规则"
---

### 功能

makefile定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，不同产商的make各不相同，也有不同的语法，但其本质都是在“文件依赖性”上做文章。Makefile执行之后所进行的操作是编译和链接，就是先将源文件编译为中间文件（object file），然后把大量的中间文件链接起来合成执行文件

### 规则

1. 如果这个工程没有编译过，那么我们的所有源文件都要编译并被链接（按需）
2. 如果这个工程某些源文件被修改，那只编译被修改的源文件，并链接目标程序
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的源文件，并链接目标程序

### 基本格式

```makefile
target ...: prerequisites ...
	command
	...
```

- `target` 目标文件，是命令执行后生成的目标，可以是 `Object File` ，也可以是执行问价，还可以是一个标签，类似于 `clean`
- `prerequisites` 依赖，也就是生成目标文件所需要的文件或者是目标
- `command` `make` 需要执行的指令，可以是任意的 `shell` 指令

### 核心规则

- target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中
- 如果prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行

### 清除

使用 `make` 清除所有生成的目标文件

```makefile
clean:
	rm target1 ...
```

### 注释

`makefile` 文件中的注释格式为 `#` 

```makefile
target ...: prerequisites ... # target1
	command
	...
```

### 使用

在使用之前需要先安装 `make`

```bash
sudo apt install make
```

然后进入 `makefile` 文件就可以编辑了，但是需要注意，命名必须是 `makefile`

注意在命令前一定需要 `tab` 键

### 例子

```makefile
main: hello.c complex.o
	gcc hello.c complex.o -o main
	
complex.0: complex.c complex.h
	gcc -c complex.c -0 complex.o

clean:
	rm main complex.o
```