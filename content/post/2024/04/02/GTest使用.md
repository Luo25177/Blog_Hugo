---
title: "GTest使用"
date: 2024-04-02 21:33:16
categories: "小工具"
tags: ["GTest", "Cpp"]
summary: "google—test使用，一种测试代码运行正确性的库"
---

## 前言

在编写代码中有 `bug` 很常见，通过编写完备的**单元测试**，可以及时发现问题，并且在后续的代码改进中持续观测是否引入了新的 `bug` ，使用 `GTest` 是一个不错的选择

`GTest` 可以用作代码的检测，一般来说可以搭配 `CTest` 来进行测试

### 单元测试

就是对程序中最小的单元进行测试，最小的可测试单元可以是一个函数，一个调用过程，一个类等

对于 `C/C++` 来说，测试一般是一个函数，检测目标函数在所有可能的输入下，函数执行过程和输出是否符合预期

在测试功能中，每一个单元都是一个可执行文件，实现了 `main` 函数，所以要在 `CMakeLists.txt` 中使用 `add_test` 命令来添加测试用例

### 测试用例

指的是对一项特定的任务进行测试，体现着测试方案，方法，技术和策略等，包括测试目标，环境，输入数据，步骤，预期结果和测试脚本等

对于上述单元测试，就是在不同输入下，目标函数的预期执行过程和输出，不同的情形可以有一个或者多个测试用例，编写测试用例**尽量覆盖所有输入情况**，特别是边界值，特殊值和异常

## GTest介绍

是 Google 开源的一个跨平台的 `C++` 单元测试框架（ `Google Test` ），提供了非常丰富的测试断言、判断宏，便于开发者编写测试用例。

## GTest配置

下载地址 [google-test](https://github.com/google/googletest)

### 安装——linux

在终端命令行中输入指令

```
sudo apt-get install libgtest-dev
```

然后对应的源码会下载到 `/usr/src/` 文件夹

进入 `/usr/src/` 文件夹之后，会看到 `gtest` 文件夹，进入其中并且执行下列指令（在该文件夹下的操作需要有 `root` 权限）

```
sudo mkdir build
cd build
sudo cmake ..
sudo make
```

执行结束之后，会在 `build` 文件夹下看到 `lib` 文件夹，里面是编译好的静态库文件，进入其中执行以下指令

```
sudo cp libgtest*.a /usr/local/lib
```

将静态库文件复制到 `lib` 目录下，之后就可以在项目中使用 `find_package` 来找到 `gtest` 并且使用 `target_link_libraries` 指令在项目中使用了

### 配置——windows

在 `windows` 上的配置，可以直接将 `GTest` 添加到项目中。例如在根目录下新建一个文件夹为 `third_party` ，下载 `realse` 版本，并且解压

需要修改根目录 `CMakeLists.txt` 文件，使用指令 `add_subdirectory` 导入 `gtest` 到项目中，在开启单元测试时，添加 `gtest` 为子模块，并将对应的头文件路径添加进来

```
enable_testing()
add_subdirectory(third_party/googletest-release-1.12.1)
include_directories(third_party/googletest-release-1.12.1/googletest/include)
```

此时执行 `cmake -B build` 可以看到构建目录下多了一个 `build/third_party/googletest-release-1.12.1` ，在构建的目录编译 `cmake` 之后会在构建的目录下的 `lib` 文件夹中生成四个库文件

- `libgtest.a` 提供单元测试相关的功能
- `libgtest_main.a` 提供单元测试的主入口，只有链接该库，测试用例就会编译生成可执行文件
- `libgmock.a` 主要提供数据库交互，网络连接等方面的模拟测试
- `libgmock_main.a` 提供数据库交互，网络连接的主入口，只有链接该库，测试用例就会编译生成可执行文件

## GTest语法

`gtest` 中的断言分为两类

### `ASSERT_*`

如果检测出失败就直接退出当前函数

- 条件判断
    - `ASSERT_TRUE(condition);`  判断条件是否为真
    - `ASSERT_FALSE(condition);`  判断条件是否为假
- 数值比较
    - `ASSERT_EQ(val1, val2);` 判断是否相等
    - `ASSERT_NE(val1, val2);` 判断是否不相等
    - `ASSERT_LT(val1, val2);` 判断是否小于
    - `ASSERT_LE(val1, val2);` 判断是否小于等于
    - `ASSERT_GT(val1, val2);` 判断是否大于
    - `ASSERT_GE(val1, val2);` 判断是否大于等于
- 字符串比较
    - `ASSERT_STREQ(str1,str2);` 判断字符串是否相等
    - `ASSERT_STRNE(str1,str2);` 判断字符串是否相等
    - `ASSERT_STRCASEEQ(str1,str2);` 判断字符串是否相等，忽视大小写
    - `ASSERT_STRCASENESTREQ(str1,str2);` 判断字符串是否不相等，忽视大小写

### `EXPECT_*`

如果检测失败就发出提示，但是继续执行

- 条件判断
    - `EXPECT_TRUE(condition);`  判断条件是否为真
    - `EXPECT_FALSE(condition);`  判断条件是否为假
- 数值比较
    - `EXPECT_EQ(val1, val2);` 判断是否相等
    - `EXPECT_NE(val1, val2);` 判断是否不相等
    - `EXPECT_LT(val1, val2);` 判断是否小于
    - `EXPECT_LE(val1, val2);` 判断是否小于等于
    - `EXPECT_GT(val1, val2);` 判断是否大于
    - `EXPECT_GE(val1, val2);` 判断是否大于等于
- 字符串比较
    - `EXPECT_STREQ(str1,str2);` 判断字符串是否相等
    - `EXPECT_STRNE(str1,str2);` 判断字符串是否相等
    - `EXPECT_STRCASEEQ(str1,str2);` 判断字符串是否相等，忽视大小写
    - `EXPECT_STRCASENESTREQ(str1,str2);` 判断字符串是否不相等，忽视大小写

## 测试

需要注意的是，在测试项目中需要链接 `libgtest` ，如果在测试的程序中没有写 `main` 函数，就需要链接 `libgtest_main` 

### 编写测试用例 CMakeLists

在 `src` 文件夹下新建一个文件 `test.cpp` ，并且在其中写入

```cpp
#include <gtest/gtest.h>

int add(int _a, int _b) {
  return _a + _b;
}

TEST(test_case_name, test_name) {
  int res = add(10, 20);
  EXPECT_EQ(res, 30);
}
```

然后在根目录下的 `CMakeLists.txt` 文件中继续写入下列内容

```
set(GTEST_LIB gtest gtest_main)

add_executable(test src/test.cpp)
target_link_libraries(test gtest gtest_main)
add_test(NAME test COMMAND test)
set(CTEST_OUPUT_ON_FAILURE TRUE)
set(GTEST_COLOR TRUE)
```

然后进行构建和编译，在根目录中输入如下指令完成构建和编译

```bash
cmake -B build
cd build
make
```

然后就可以进行测试， `ctest` 执行测试

![1712026397325.png](./1712026397325.png)

### 编写测试程序 g++

直接新建源文件 `test.cpp` ，并且在其中写入

```cpp
#include <gtest/gtest.h>

int add(int _a, int _b) {
  return _a + _b;
}

TEST(test_case_name, test_name) {
  EXPECT_EQ(add(10, 20), 30);
  EXPECT_EQ(add(12, 318), 330);
}
```

在终端执行下列指令，进行编译

```
g++ test1.cpp -lgtest -lpthread -lgtest_main
```

然后运行生成的 `a.out` 文件，就可以看到测试结果了

![1711853331118.png](./1711853331118.png)