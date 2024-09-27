---
title: Linux上配置ipopt和casadi求解二次规划
date: 2024-09-23 21:12:40
categories: "机器人"
tags: ["ipopt", "casadi", "二次规划"]
summary: "Linux 配置 ipopt 和 casadi 以求解二次规划问题"
---

## 前言

二次规划问题是是很多机器人控制器中所涉及到的问题，好的求解方式也非常重要。这里使用 `casadi` 来实现二次规划的求解

## ipopt 安装

### 安装依赖

由于需要编译 C++，所以需要一些基础的依赖

```bash
apt-get update
apt-get -y upgrade
apt install build-essential
apt-get install -y gcc g++ gfortran git patch wget pkg-config liblapack-dev libmetis-dev libblas-dev vim
```

### 下载所需要的代码文件

一共有 5 个比较重要的包

```bash
git clone https://github.com/coin-or/Ipopt.git # ipopt
git clone https://github.com/coin-or-tools/ThirdParty-ASL.git # ThirdParty-ASL
git clone https://github.com/coin-or-tools/ThirdParty-HSL.git # ThirdParty-HSL
git clone https://github.com/coin-or-tools/ThirdParty-Mumps.git # ThirdParty-Mumps
git clone https://github.com/Luo25177/coinhsl.git # coinhsl
```

### 编译文件

**编译 ThirdParty-ASL**

```bash
cd ThirdParty-ASL
./get.ASL
./configure
make
make install
```

**编译 ThirdParty-HSL**

注意需要将 `coinhsl` 放在 `ThirdParty-HSL` 文件夹下

```bash
cd ThirdParty-HSL
git clone https://github.com/Luo25177/coinhsl.git
./configure
make
make install
```

**编译 ThirdParty_Mumps**

```bash
cd ThirdParty_Mumps
./get.Mumps
./configure
make
make install
```

**编译 Ipopt**

```bash
cd Ipopt
mkdir build && cd build
../configure
make
make test
make install
```

### 配置环境变量

```bash
cd /usr/local/include
cp coin-or coin -r
ln -s /usr/local/lib/libcoinmumps.so.3 /usr/lib/libcoinmumps.so.3
ln -s /usr/local/lib/libcoinhsl.so.2 /usr/lib/libcoinhsl.so.2
ln -s /usr/local/lib/libipopt.so.3 /usr/lib/libipopt.so.3
echo "Add the/usr/local/lib directory to the configuration file of the shared library"
echo "/usr/local/lib" >> /etc/ld.so.conf
ldconfig
```

### 测试

```bash
cd Ipopt/build/examples/Cpp_example
sudo make
./cpp_example
```

能正常运行出结果且不报错即可

## casadi 安装

```bash
git clone https://github.com/casadi/casadi.git
cd casadi
mkdir build
cd build
cmake .. -DWITH_IPOPT=ON -DWITH_EXAMPLES=OFF
make
sudo make install
```

### 测试

从 `casadi/docs/examples/cplusplus` 找一个例子 `rocket_ipopt.cpp` 文件，将其内容复制到一个文件夹内，然后创建 `CMakeLists.txt` 文件，配置方式如下

```bash
cmake_minimum_required(VERSION 3.21)
project(testCPP)

set(CMAKE_CXX_STANDARD 14)

add_executable(cppad_ipopt_demo a.cpp)
target_link_libraries(cppad_ipopt_demo ipopt)
target_link_libraries(cppad_ipopt_demo casadi)
```

然后在该文件夹下执行以下指令

```bash
mkdir build && cd build
cmake ..
make
./cppad_ipopt_demo
```

成功执行即可

## 参考

[https://www.cnblogs.com/aimoboshu/p/18220353](https://www.cnblogs.com/aimoboshu/p/18220353)

[https://blog.csdn.net/qq_43066145/article/details/139202657](https://blog.csdn.net/qq_43066145/article/details/139202657)
