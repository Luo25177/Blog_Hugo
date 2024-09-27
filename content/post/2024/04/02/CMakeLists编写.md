---
title: "CMakeLists编写"
date: 2024-04-02 21:31:09
categories: "小工具"
tags: ["cmake", "C", "Cpp"]
summary: "CMakeLists编写基础语法"
---

## 前言

CMake是一个跨平台的构建系统，能自动生成各种平台和编译器的构建文件，相对于直接编写 `makefile` 文件，编写 `CMakeLists` 文件不需要考虑太多的文件之间的依赖关系，它可以自动帮你管理复杂项目的构建过程，检测依赖关系，生成目标文件，编译静态库和动态库等，减少手动管理的负担

可以看看这个知乎专题 [CMake实践应用专题 - 知乎 (zhihu.com)](https://www.zhihu.com/column/c_1369781372333240320) ，下面的内容几乎都是从这里整理的

## CMake**一般使用流程**

### 生成构建系统

通过 `cmake` 指令生成构建系统，可以在命令行中输入 `cmake --help` 来看到 `cmake` 指令支持的详细参数，常用参数如下

| 参数 | 作用                                                   |
| ---- | ------------------------------------------------------ |
| -S   | 指定源文件根目录，该目录下必须包含 CMakeLists.txt 文件 |
| -B   | 指定构建目录，构建生成的中间文件和目标文件的生成路径   |
| -D   | 指定变量，格式为 -D <var> = <value>， -D 后的空格可省  |

例如，使用当前目录为源文件目录，其中包含 `CMakeLists.txt` 文件，使用 `build` 目录作为构建目录，设定变量 `CMAKE_BUILD_TYPE` 为 `Debug`

```bash
 cmake -S . -B build -DCMAKE_BUILD_TYPE=Debug
```

### 执行构建

使用 `cmake --build [<dir> | --preset <preset>]` 进行构建

这里指定的目录就是生成构建系统时的指定的构建目录，常用的一些参数

| 参数                   | 含义                                         |
| ---------------------- | -------------------------------------------- |
| —target                | 指定构建目标代替默认的构建目标，可以指定多个 |
| —parallel / -j[<jobs>] | 指定构建目标时使用的进程数                   |

### 执行测试，安装或打包

安装打包的内容可以看后面

测试主要是使用的 `gtest` 库，所以可以看看我的 `gtest` 使用的文章

## CMake基础语法

### 注释

使用 `#` 开头的行是注释行，会被忽略，块注释为 `#[[ ... ]]`

### 普通变量

使用 `set` 指令来**定义变量**

```
set(VALUE_NAME value)
```

使用 `$` 来读取变量

```
${VALUE_NAME}
```

使用 `unset` 来取消设置变量

```
unset(VALUE_NAME)
```

1. 如果要设置的变量值包含空格，需要使用双引号把整个或者 `\` 转义
2. 如果设置多个值或者字符串值的中间有 `;` ，则保存成 `list` 
3. 变量可以被 `list` 命令操作，单个值的变量相当于只有一个元素的列表
4. 在`if()`条件判断中可以简化为只用变量名

### Cache变量

作用是为了提供用户配置选项，若未指定则使用默认值，使用如下方法设置

```
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

使用 `$CACHE` 引用变量， `CACHE` 变量会被保存在构建目录下的 `CMakeCache.txt` 中，缓存之后是不变的的，除非重新配置

### 环境变量

修改当前处理进程的环境变量，设置格式如下

```
set(ENV{<variable>} [<value>])
```

可以使用 `$ENV` 来引用环境变量

### 条件语句

```
if(CONDITION1)
	...
elseif(CONDITION2)
	...
else()
	...
endif()
```

用于比较的语法

1. 字符串： `STREQUAL` ， `STRLESS` ， `STRGREATER`
2. 数值比较： `EQUAL` ， `LESS` ， `GREATER`
3. 布尔运算： `AND` ， `OR` ， `NOT`
4. 路径判断： `EXISTS` ， `IS_DIRECTORY` **，** `IS_ABSOLUTE`
5. 多个条件语句组合： `(cond1) AND (cond2 OR (cond3))`
6. 常量： `ON` ， `YES` ， `TRUE` ， `Y` 和非 0 值被视为 `True` ，对于 `0` ， `OFF` ， `NO` ， `FALSE` ， `N` ， `IGNORE` ，空字符串， `NOTFOUND` ，以及以 `-NOTFOUND` 结尾的字符串都被视为 `False`

### 循环语句

类似于 `python` 中的循环语句的定义

```
foreach(item IN LISTS items)
	...
endforeach()
```

### 函数

可以自定义自己的函数

```
function(FUNCTION_NAME arg1 arg2)
	...
endfunction()
```

### 宏

```
macro(MACRO_NAME arg1 arg2)
	...
endmacro()
```

### 消息打印

利用 `message` 指令来把信息打印出来，如下

```
message([<mode>] "message text" ... )
```

其中 `mode` 的选值如下

1. 空或者是 `NOTICE` ：比较重要的信息
2. `DEBUG` ：调试信息
3. `STATUS` ：状态信息
4. `WARNING` ： `cmake` 警告，不会打断进程
5. `SEND_ERROR` ： `cmake` 错误，会继续执行，但是跳过生成构建系统
6. `FATAL_ERROR` ： `cmake` 致命错误，会终止进程

### 列表操作

设置多个值或者字符串值的中间有 `;` ，则保存成 `list` ， `list` 存在于 `cmake` 中，有很多子命令

1. `APPEND` 向列表中添加元素
2. `LENGTH` 获取列表中元素的个数
3. `JOIN` 将列表元素用指定的分隔符连接起来

```
set(LIST A B)
set(LIST A;B)
set(LIST "A;B")
list(APPEND LIST "C")
list(LENGTH LIST LIST_LEN)
list(JOIN LIST "," LIST_SPLIT)
```

### 文件操作

`cmake` 的 `file` 命令支持的操作比较多，可以读写，创建或复制文件和目录，计算文件hash，下载文件，压缩文件等，例如

```
file(GLOB_RECURSE ALLSRC src/*.c)
```

其中 `GLOB_RECURSE` 表示执行递归查找，查找目录下的所有符合指定的正则表达式的文件

### 执行系统命令

使用 `execute_process` 指令可以执行一条或者顺序执行多条系统命令

`execute_process(COMMAND bash "-c" "git rev-parse --short HEAD" OUTPUT_VARIABLE COMMIT_ID)`

上述是获取当前仓库最新提交的 `commit` 的 `commit id`

### 设置项目和版本

如果需要在项目中**标明版本号、Git的hash号、编译时间**等信息，可以利用 `configure_file` 来实现

```
configure_file(<input> <output>
               [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
               [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
```

其中

- `input` 输入文件，一般是以 `.h.in` 为后缀
- `output` 输出文件，一般是 `.h` 后缀
- `COPYONLY` 只拷贝文件，不进行任何变量替换，与 `NEWLINE_STYLE` 冲突（指定 `NEWLINE_STYLE` 之后无效）
- `ESCAPE_QUOTES` 避开所有反斜杠的转义
- `@ONLY` 限制变量替换，让其只替换被 `@VAR@` 引用的变量（那么 `${VAR}` 格式的变量将不会被替换）。这在配置`${VAR}`语法的脚本时是非常有用的
- `NEWLINE_STYLE` 指定输出文件中的新行格式。 `UNIX` 和 `LF` 的新行是 `\n`， `DOS` 和 `WIN32` 和 `CRLF` 的新行格式是 `\r\n`。这个选项在指定了 `COPYONLY` 选项时不能使用(无效)

在 `cmake` 教程中对它的解释是将一个文件复制到另一个位置并且修改其内容，这里的修改并非是随意修改，而是**将input文件复制到output文件，并在输入文件内容中的变量，替换引用为@VAR@或${VAR}的变量值。每个变量引用将替换为该变量的当前值，如果未定义该变量，则为空字符串，**如下

```
set(BUILD_Version 1)

// input file
#define BUILD_Version @BUILD_Version@

// output file
#define BUILD_Version 1
```

在 `CMakeLists.txt` 文件中使用 `project` 指令可以指定版本号

```
project(ProjectName VERSION 1.0.0 LANGUAGES C CXX)
```

其中第一个参数为项目名称，并且可以通过 `VERSION` 指定版本号，格式为 `major.minor.patch.tweak` 并且 `cmake` 会将对应的值分别赋给以下变量，如果没有将会是空字符串

```
PROJECT_VERSION, <PROJECT-NAME>_VERSION
PROJECT_VERSION_MAJOR, <PROJECT-NAME>_VERSION_MAJOR
PROJECT_VERSION_MINOR, <PROJECT-NAME>_VERSION_MINOR
PROJECT_VERSION_PATCH, <PROJECT-NAME>_VERSION_PATCH
PROJECT_VERSION_TWEAK, <PROJECT-NAME>_VERSION_TWEAK
```

利用上述的 `configure_file` 命令可以配置自动生成版本头文件，将头文件版本号定义成对应的宏，或者定义为接口，便于了解当前版本

### 指定编程语言版本

可以使得在不同的机器上都能够有统一的编译，可以指定语言版本

```
set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)
```

`CMAKE_` ， `_CMAKE` 或者是下划线开头后面加上任意 `cmake` 命令的变量名都是 `cmake` 保留的，都是 `cmake` 的内置变量，可以通过修改这些变量的值来配置 `cmake` 的构建

### 配置编译选项

通过指令 `add_compile_options` 指令可以为所有编译器配置编译选项，同时对多个编译器有效

```
add_compile_options(-g -Wall -Werror)
```

通过设置变量 `CMAKE_C_FLAGS` 可以配置 C 编译器的编译选项

```
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pipe -std=c99")
```

而设置变量 `CMAKE_CXX_FLAGS` 可以配置 C++ 编译器的选项

```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pipe -std=c++11")
```

### 配置编译类型

通过设置变量 `CMAKE_BUILD_TYPE` 来配置编译类型，可以设置为 `Debug` ， `Release` ， `RelWithDebInfo` ， `MinSizeRel` 等

```
set(CMAKE_BUILD_TYPE Debug)
```

也可以在执行 `cmake` 命令的时候通过参数 `-D` 来指定， `cmake` 会检查是否有对应编译类型的编译选项，如果有就将它的内容追加到其中

对于不同的编译类型可以设置不同的编译选项，比如对于 `Debug` 类型，开启调试信息

```
set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g")
```

### 添加全局宏定义

通过命令 `add_definitions` 可以添加全局的宏定义，在源代码中就可以通过判断不同的宏定义来实现相应的代码逻辑，例如

```
add_definitions(-DDEBUG -DREAL_COOL_ENGINEER)
```

### 添加include目录

通过命令 `include_directories` 来设置头文件的搜索目录

```
include_directories(.) # 表示所有的头文件
include_directories(inc/)
```

### 编译目标文件

一般来说，编译目标的类型一般有静态库，动态库和可执行文件，编写 `CMakeLists.txt` 主要包括两步

- 编译：确定编译目标所需要的源文件
- 链接：确定链接的时候需要依赖的额外的库

### 编译生成库文件

将项目目录路径下的源文件编译为静态库，需要获取编译此静态库需要的文件列表，可以使用`set`命令，或者`file`命令来进行设置

```
file(GLOB_RECURSE FUNCTION_LIB_SRC /function/*.c)
# set(FUNCTION_LIB_SRC /function/*.c)
add_library(function_lib STATIC ${FUNCTION_LIB_SRC })
```

其中使用 `add_library` 指令编译名为 `function_lib` 的静态库，第二个参数表示库的类型， `STATIC` 表示静态链接库， `SHARED` 表示动态链接库

### 编译可执行文件

通过 `add_executable` 命令来向构建系统中添加一个可执行构建目标，指定编译需要的源文件。但是对于可执行文件来说，有时候还会依赖其它的库则需要使用 `target_link_libraries` 命令来声明构建此可执行文件需要链接的库

```
add_executable(demo main.c)
target_link_libraries(demo function_lib)
```

## CMake模块化构建

`CMakeLists.txt` 是定义一个目录的构建系统，所谓的模块化构建是分别为每一个子模块目录编写一个 `CMakeLists.txt` 文件，并且在其上层目录中导入子目录构建系统生成对应的目标，以便在上层目录中使用

这里使用的是一个 `cmake` 模板的开源项目 [cmake-template-gitee](https://gitee.com/RealCoolEngineer/cmake-template)

在这个项目中，将 `math` 目录视为子模块，为其单独定义构建系统，而整个项目的编译依赖于 `math` 模块的编译结果来生成其它目标文件

### 定义子目录的构建系统

在需要构建的子目录下创建一个 `CMakeLists.txt` 文件，在其中写入的内容只需要**编译构建库文件**，以及针对对外提供的接口的功能进行测试就行

对于子目录，也有自己的 `project` 指令，同时可以指定自己的版本号

`aux_source_directory` 可以搜索指定目录（第一个参数）下的所有源文件，将源文件的列表保存到指定的变量（第二个参数）

在 `src/math` 目录下新建  `CMakeLists.txt` 文件，写入内容，将其编译为一个静态链接库

```
project(CMakeTemplateMath VERSION 0.0.1 LANGUAGES C CXX)

aux_source_directory(. MATH_SRC)
message("MATH_SRC: ${MATH_SRC}")

add_library(math STATIC ${MATH_SRC})
```

### 包含子目录

使用 `add_subdirectory` 命令可以包含一个子目录的构建系统，如下

```
add_subdirectory(dir [binary_dir] [EXCLUDE_FROM_ALL])
```

其中 

1. `dir` 就是需要包含的目标目录，该目录下必须存在一个 `CMakeLists.txt` 文件，一般相对于当前的 `CMakeLists.txt` 的目录路径，也可以是绝对路径
2. `binary_dir` 是可选的参数，用于指定子构建系统输出文件的路径，相对于当前的 `binary_tree` ，也可以是绝对路径。如果 `dir` 是当前目录的子目录，那 `binary_dir` 是不做任何相对路径展开的 `dir` ，但是如果 `dir` 不是当前的子目录，必须指定 `binary_dir` ， 从而 `cmake` 才能确定文件的生成目录
3. `EXCLUDE_FROM_ALL` 如果指定了，那子路径下的目标默认不会被包含到父路径的 ALL 目标里，并且也会被排除在工程文件之外，但是如果在父级项目显式声明依赖子目录的目标文件，那么对应的目标文件还是会被构建以满足父级项目的依赖需求

可以在根目录下的 `CMakeLists.txt` 写入命令来链接编译的静态库

```
add_subdirectory(src/c/math)
add_executable(demo src/c/main.c)
target_link_libraries(demo math)
```

而且此时构建和编译的命令没有任何改变，依旧是在终端中输入 `cmake` 指令

## 导入目标文件

上述中所用的 `add_subdirectory` 方法实际上就是通过源文件来构建项目所依赖的目标文件，但是在 `cmake` 中也可以通过命令导入已经编译好的目标文件

### 导入库文件

使用 `add_library` 指令可以通过指定 `IMPORTED` 选项表明是一个导入的库文件，通过 `set_property` 设置属性来指明路径

```
add_library(math STATIC IMPORTED)
set_property(TARGET math PROPERTY 
						IMPORTED_LOCATION "./lib/libmath.a")
```

也可以使用 `find_library` 命令来查找，比如在 `lib` 目录下查找 `math` 的 `Realse` 版本和 `Debug` 版本

```
find_library(LIB_MATH_DEBUG mathd HINTS "./lib")
find_library(LIB_MATH_RELEASE math HINTS "./lib")
```

可以通过指令 `set_target_properties` 来设置不同的编译选项，例如

```
set_target_properties(math PROPERTIES
  IMPORTED_LOCATION "${LIB_MATH_RELEASE}"
  IMPORTED_LOCATION_DEBUG "${LIB_MATH_DEBUG}"
  IMPORTED_CONFIGURATIONS "RELEASE;DEBUG"
)
```

导入之后，就可以将该库链接到其它目标上，但是导入该库的目标不能被 `install` 

### 导入可执行文件

在 `add_executable` 通过指定 `IMPORTED` 来指定可执行文件是外部导入的 

```
add_executable(demo IMPORTED)
set_property(TARGET demo PROPERTY
             IMPORTED_LOCATION "./bin/demo")
```

## 库依赖

主要是对于 `target_link_libraries` 命令的几个关键字，就是目标文件依赖项的使用范围，对于两个动态链接库 `[a.so](http://a.so)` 和 `b.so`

- `PRIVATE` 就是 `a` 使用了 `b` ，但是并不对外暴漏 `b` 的接口
- `INTERFACE` 就是 `a` 未使用 `b` ，但是对外暴漏 `b` 的接口
- `PUBLIC` 就是 `a` 使用了 `b` 并且对外暴漏 `b` 的接口

```
target_link_libraries(a PRIVATE/INTERFACE/PUBLIC b)
target_include_directories(a PRIVATE/INTERFACE/PUBLIC b)
```

## 安装

### **install指令**

安装时使用 `install` 指令，用于指定一个项目的安装规则，命令格式如下

```cpp
install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
```

可以看出 `install` 命令可以安装的目标类型为：构建目标、文件、程序、目录等，对应的关键字后面跟上对应要安装的目标，安装不同目标时有一些通用的关键字
**DESTINATION**

就是安装对象的目标安装路径，可以是绝对路径，也可以是相对路径

```
install(TARGETS function_lib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
```

其中

- `TARGETS` 用于指定需要安装的目标
- `RUNTIME DESTINATION` 指定可执行文件的安装路径
- `LIBRARY DESTINATION` 指定共享库文件的安装路径
- `ARCHIVE DESTINATION` 指定静态库文件的安装路径

如果指定 `CMAKE_INSTALL_PREFIX` 目录之后，那库文件将会被安装在对应目录下的 `lib` 文件夹中，可执行文件将会被安装在对应目录下的 `bin` 文件夹中，不同类型的目标文件安装到不同子目录。需要注意的是， `CMAKE_INSTALL_PREFIX` 在不同的系统上会有不同的默认值，使用时最好显式指定

头文件也可以使用上述的命令安装

**CONFIGURATIONS**

为不同的配置设置不同的安装规则，例如对于 `Debug` 和 `Release` 配置不同的安装路径，如下

```cpp
install(TARGETS target
        CONFIGURATIONS Debug
        RUNTIME DESTINATION Debug/bin)
install(TARGETS target
        CONFIGURATIONS Release
        RUNTIME DESTINATION Release/bin)
```

**PERMISSIONS**

设置安装目标权限，接受的是一个权限关键字列表，例如

```cpp
install(TARGETS target
        RUNTIME PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE)
```

### **安装构建目标**

安装一个构建好的目标，命令格式为

```
install(TARGETS function_lib
        RUNTIME DESTINATION bin
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib)
```

完整的命令格式为

```
install(TARGETS targets... [EXPORT <export-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```

其中参数中的`TARGET`可以是很多种目标文件，最常见的是通过 `ADD_EXECUTABLE` 或者 `ADD_LIBRARY` 定义的目标文件，即可执行二进制，动态库，静态库

| 目标文件       | 内容                    | 安装目录变量                | 默认安装文件夹 |
| -------------- | ----------------------- | --------------------------- | -------------- |
| ARCHIVE        | 静态库                  | ${CMAKE_INSTALL_LIBDIR}     | lib            |
| LIBRARY        | 动态库                  | ${CMAKE_INSTALL_LIBDIR}     | lib            |
| RUNTIME        | 可执行二进制文件        | ${CMAKE_INSTALL_BINDIR}     | bin            |
| PUBLIC_HEADER  | 与库关联的PUBLIC头文件  | ${CMAKE_INSTALL_INCLUDEDIR} | include        |
| PRIVATE_HEADER | 与库关联的PRIVATE头文件 | ${CMAKE_INSTALL_INCLUDEDIR} | include        |

### **安装目录**

安装一个目录，一般用于将头文件安装到目标路径，实际中，一般将需要安装的头文件放在一个特定的目录下，然后直接安装整个目录，如下

```
install(DIRECTORY "${PROJECT_SOURCE_DIR}/include/"
	      DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
```

更加完整的命令格式为

```
install(DIRECTORY dirs...
        TYPE <type> | DESTINATION <dir>
        [FILES_MATCHING]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]])
```

**TYPE/DESTINATION**

安装目录必须指定安装的目录类型 `TYPE` 或者安装的目标路径 `DESTINATION` ，但是不可以同时指定， `TYPE` 用于指定安装的目录中的文件类型，并且 `cmake` 会自动按照类型分配安装位置，不同类型对应的安装路径如下

| TYPE类型    | 安装目录变量                   | 默认安装文件夹 |
| ----------- | ------------------------------ | -------------- |
| BIN         | ${CMAKE_INSTALL_BINDIR}        | bin            |
| SBIN        | ${CMAKE_INSTALL_SBINDIR}       | sbin           |
| LIB         | ${CMAKE_INSTALL_LIBDIR}        | lib            |
| INCLUDE     | ${CMAKE_INSTALL_INCLUDEDIR}    | include        |
| SYSCONF     | ${CMAKE_INSTALL_SYSCONFDIR}    | etc            |
| SHAREDSTATE | ${CMAKE_INSTALL_SHARESTATEDIR} | com            |
| LOCALSTATE  | ${CMAKE_INSTALL_LOCALSTATEDIR} | var            |
| RUNSTATE    | ${CMAKE_INSTALL_RUNSTATEDIR}   | /run           |
| DATA        | ${CMAKE_INSTALL_DATADIR}       |                |
| INFO        | ${CMAKE_INSTALL_INFODIR}       | /info          |
| LOCALE      | ${CMAKE_INSTALL_LOCALEDIR}     | /locale        |
| MAN         | ${CMAKE_INSTALL_MANDIR}        | /man           |
| DOC         | ${CMAKE_INSTALL_DOCDIR}        | /doc           |

也可以选择使用 `DESTINATION` 显式指定安装目录

**FILES_MATCHING**

安装目录的时候默认会安装所有的文件，如果使用 `FILES_MATCHING` 关键字（在第一个 `PATTERN` 或者 `REGEX` 之前），则表示必须要满足对应的模式或者正则的文件才能被安装

例如，对于安装一个目录下的头文件，但是头文件与源文件混合，如下

```
install(DIRECTORY src/ DESTINATION include/
        FILES_MATCHING PATTERN "*.h")
```

**PATTERN/REGEX**

- `PATTERN` 文件名完全一致才会被安装
- `REGEX` 通过正则表达式匹配之后才会安装

在这两个表达式后面还可以加上 `EXCLUDE` 表示反选，或者使用 `PERMISSIONS` 指定匹配的目标文件的权限

### 安装文件

与安装目录类似，命令格式为

```
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
```

- `FILE` 指的是文件的默认权限是一般文件
- `PROGRAMS` 指的是文件为可执行文件，默认有可执行权限

### 自定义安装脚本

使用 `install` 命令可以在安装的时候执行自定义脚本，格式为

```
install([[SCRIPT <file>] [CODE <code>]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL] [...])
```

其中

- `SCRIPT` 指定安装时需要执行的脚本
- `CODE` 指定的是 `cmake` 的指令，也就是在安装期间执行的 `cmake` 指令

### 执行安装

在构建编译完成之后，执行命令安装

```
cmake --build . --target install
```

针对 `make` 工具，安装的指令为

```
make install
```

在 `cmake-3.15` 之后，可以使用如下指令安装

```
cmake --install . --prefix "../output"
```

其中 `--install` 指定构建目录， `--prefix` 指定安装路径，指定之后会覆盖安装路径变量 `CMAKE_INSTALL_PREFIX`

## 打包

### CPack

使用打包功能，需要先使用命令 `include(CPack)` 启用相关的功能，构建之后会在构建目录下生成两个 `cpack` 配置文件，`CPackConfig.cmake`和`CPackSourceConfig.cmake` ，分别对应着 `package` 和 `package_source`

在执行构建编译之后使用 `cpack` 命令行工具进行打包安装。可以使用 `-G` 参数指定生成器，常用的为 `ZIP` ， `TGZ` ， `7Z` 等，也可以同时指定多种类型，利用 `CMake` 语法中的列表，在构建目录下使用如下指令

```
cpack -G ZIP --config CPackConfig.cmake
cpack -G ZIP --config CPackSourceConfig.cmake
```

对于 `cmake` 指令，可以在构建目录下使用如下指令

```
cmake --build . --target package
cmake --build . --target package_source
```

对于 `make` 工具，也可以使用命令 `make package` 或者 `make package_source` 进行打包安装

### 内置变量

打包的内容就是 `install` 指令安装的内容，还需要设置一些变量

| 变量                     | 作用                                                                                                                                                      |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CPACK_GENERATOR          | 打包使用的工具                                                                                                                                            |
| CPACK_OUTPUT_FILE_PREFIX | 打包安装的路径前缀，如果是相对路径，那就是相对于构建路径                                                                                                  |
| CPACK_INSTALL_PREFIX     | 打包压缩包的内部目录                                                                                                                                      |
| CPACK_PACKAGE_FILE_NAME  | 打包压缩包的名称，由CPACK_PACKAGE_NAME，CPACK_PACKAGE_VERSION，CPACK_SYSTEM_NAME三部分构成                                                                |
| CPACK_SET_DESTDIR        | 如果没有 CPACK_SET_DESTDIR，CPack 使用 CPACK_PACKAGING_INSTALL_PREFIX 作为前缀，而设置 CPACK_SET_DESTDIR 后，CPack 将使用 CMAKE_INSTALL_PREFIX 作为前缀。 |
| CPACK_OUTPUT_CONFIG_FILE | 配置文件，默认为CPackConfig.cmake                                                                                                                         |
| CPACK_PACKAGE_VERSION    | 默认为项目版本号，默认值为 project 所设置的版本号，如果没有设置就是 cmake 的默认值                                                                        |

`cpack` 有一些参数是可以覆盖 `CMakeLists.txt` 中所设置的参数的，所以一般来说以上变量的设置在 `include(CPack)` 之前

## 编译过程

实际上 `cmake` 的编译过程中的命令与 `gcc` 的编译过程是有关联的

### 预处理

可以使用下列指令定义一个宏定义

```bash
add_definitions(-Dname)
```

### 编译

使用如下指令给编译过程添加编译选项

```bash
add_compile_options(-O0 -g ...)
```

需要注意的是，因为CMake的构建目标必须是库或者可执行文件，并没有命令仅生成 `.o` 文件。如果使用生成器表达式的话，对于多个编译选项，需要使用双引号把生成器表达式括起来，且在选项之间使用分号隔开

### 链接

使用如下指令来进行指定编译过程中的链接参数

```bash
add_link_options(-pthread ...)
```

## 合并静态库

### 基于cmake

`**add_custom_command`** 

向目标添加规则，并通过执行命令生成输出。

```
add_custom_command(OUTPUT output1 [output2 ...]
                   COMMAND command1 [ARGS] [args1...]
                   [COMMAND command2 [ARGS] [args2...] ...]
                   [MAIN_DEPENDENCY depend]
                   [DEPENDS [depends...]]
                   [BYPRODUCTS [files...]]
                   [IMPLICIT_DEPENDS <lang1> depend1
                                    [<lang2> depend2] ...]
                   [WORKING_DIRECTORY dir]
                   [COMMENT comment]
                   [DEPFILE depfile]
                   [JOB_POOL job_pool]
                   [VERBATIM] [APPEND] [USES_TERMINAL]
                   [COMMAND_EXPAND_LISTS])
```

其中

- OUTPUT：指定命令预期生成的输出文件。3.20版本以后，输出参数可使用一组受限的生成器表达式。
- COMMAND：指定生成时要执行的命令行。
- MAIN_DEPENDENCY：指定命令的主输入源文件。类似于DEPENDS。
- DEPENDS：指定命令所依赖的文件。
- BYPRODUCTS：指定命令预期生成的文件，但其修改时间可能比依赖项的新，也可能不比依赖项的新。
- IMPLICIT_DEPENDS：请求扫描输入文件的隐式依赖项。此选项不能与DEPFILE选项同时指定。
- WORKING_DIRECTORY：指定在何处执行命令。
- COMMENT：指定在生成时执行命令之前显示的消息。
- DEPFILE：指定保存自定义命令依赖项的depfile。它通常由自定义命令本身发出。仅当生成器支持此关键字时，才能使用此关键字。
- JOB_POOL：为Ninja生成器指定一个池。
- VERBATIM：对于构建工具，命令的所有参数都将被正确转义，以便被调用的命令接收到的每个参数不变。请注意，在add_custom_command甚至看到参数之前，CMake语言处理器仍然使用一级转义。建议使用VERBATIM，因为它可以保证正确的行为。如果不指定VERBATIM，则行为是依赖于平台的，因为CMake没有针对于特定工具中特殊字符的保护措施。
- APPEND：将COMMAND和DEPENDS 附加到第一个指定输出的自定义命令。
- USES_TERMINAL：如果可能，该命令将被授予直接访问终端的权限。
- COMMAND_EXPAND_LISTS：命令参数中的列表将展开，包括使用生成器表达式创建的列表

**`add_custom_target`**

在很多时候，需要在`cmake`中创建一些目标，如`clean`、`copy`等等，这就需要通过`add_custom_target`来指定。同时，`add_custom_command`可以用来完成对`add_custom_target`生成的`target`的补充

用于增加一个没有输出的目标，使得它总是被构建

```
add_custom_target(Name [ALL] [command1 [args1...]]
                  [COMMAND command2 [args2...] ...]
                  [DEPENDS depend depend depend ... ]
                  [BYPRODUCTS [files...]]
                  [WORKING_DIRECTORY dir]
                  [COMMENT comment]
                  [JOB_POOL job_pool]
                  [VERBATIM] [USES_TERMINAL]
                  [COMMAND_EXPAND_LISTS]
                  [SOURCES src1 [src2...]])
```

其中

- ALL：表明该目标会被添加到默认的构建目标，使得它每次都被运行
- COMMAND：指定要在构建时执行的命令行；
- DEPENDS：指定命令所依赖的文件；
- COMMENT：在构建时执行命令之前显示给定消息；
- WORKING_DIRECTORY：使用给定的当前工作目录执行命令。如果它是相对路径，它将相对于对应于当前源目录的构建树目录；
- BYPRODUCTS：指定命令预期产生的文件。

**具体流程**

1. 合并静态库
    
    利用 `add_custom_command` 配合 `add_custom_target` 命令，将 `liba.a` 和 `libb.a` 合并为 `libmerge.a`
    
    对于 macos 系统中，在根目录下的 `CMakeLists.txt` 文件中写入
    
    ```
    add_custom_command(OUTPUT libmerge.a
    									COMMAND libtool -static -o libmerge.a $<TARGET_FILE:math> $<TARGET_FILE:nn>
    									DEPENDS a b)
    ```
    
    对于其它系统中，在根目录下的 `CMakeLists.txt` 文件中写入
    
    ```
    add_custom_command(OUTPUT libmerge.a
    							    COMMAND ar crsT libmerge.a $<TARGET_FILE:math> $<TARGET_FILE:nn>
    							    DEPENDS a b)
    ```
    
    还需要在后面添加一句，且必须得有
    
    ```
    add_custom_target(_merge ALL DEPENDS libmerge.a)
    ```
    
    在合并静态库时，需要知道每个静态库的路径，在 `cmake` 中，目标静态库 `a` 的路径可以通过生成器表达式 `$<TARGET_FILE:a>` 获取
    
    如果使用 `find_library`  查找到的静态库，这时候就 `DEPENDS` 不需要加入 c ，如下
    
    ```
    find_library(LIB_C c HINTS ${SEARCH_PATH})
    ```
    
2. 导入合并的静态库
    
    把静态库导入，并且链接到可执行程序使用
    
    ```
    add_library(merge STATIC IMPORTED GLOBAL)
    set_target_properties(merge PROPERTIES
        IMPORTED_LOCATION ${CMAKE_CURRENT_BINARY_DIR}/libmerge.a
    )
    add_executable(main src/c/main.c)
    target_link_libraries(main PRIVATE merge)
    ```
    
    由于 `libmerge` 是由 `add_custom_command` 指定的输出，所以会标记为自动生成的文件 `GENERATED`
    
    链接项目中依赖于导入的静态库文件 `merge` ，而对于 `merge` 被指定了 `IMPORTED_LOCATION` ，也就是被指定为自动生成的，所以 cmake 就会等待 `libmerge.a` 生成之后再进行链接
    

### 其它方法

1. 方法1
    
    先使用 `ar` 把静态库 `liba` 和 `libb` 拆解为多个 `.o` 文件
    
    ```
    ar x liba.a
    ar x libb.a
    ```
    
    再把所有的 `.o` 文件打包为一个静态库
    
    ```
    ar crs libmerge.a *.o
    ```
    
    其中
    
    - `x` 拆解静态库文件为其包含的内容
    - `c` 封装 `.o` 文件为静态库文件
    - `r` 覆盖同名库文件或者创建新的目标库文件
    - `s` 相当于对结果执行一次 `ranlib` 为静态库的内容添加索引，以此提高访问效率
2. 方法2
    
    上述方法1中一个更简洁的表达
    
    ```
    ar crsT libmerge.a liba.a libb.a
    ```
    
    其中
    
    - `T` 表示将后续所有静态库中的 `.o` 文件打包到第一个参数指定的静态库文件中。如果不加该参数，得到的将会是后面几个 `.a` 文件的集合
    - 可以使用 `ar -t` 查看打包的内容
3. 方法3
    
    使用 MRI 脚本，首先写一个 MRI 脚本， `merge.mri` ，写入如下内容
    
    ```
    create libmerge.a
    addlib liba.a 
    addlib libb.a
    save
    end
    ```
    
    然后使用命令
    
    ```
    ar -M < merge.mri
    ```
    
4. 方法
    
    使用 `libtool` 命令
    
    ```
    libtool -static -o libmerge.a liba.a libb.a
    ```
    

上述方法中，方法1，4适用于macos，方法1，2，3均适用于linux

## 生成器表达式

就是再 `cmake` 生成构建系统时根据不同配置动态生成特定的内容，着就依赖于不同的条件。生成器表达式的格式为 `$<...>` ，可以嵌套，可以在很多构建目标的属性设置和特定的 `cmake` 指令中，并且在生成构建系统时表达式被展开，所以不能通过解析配置 `CMakeLists.txt` 阶段的 `message` 指令打印

### 布尔生成器表达式

**逻辑运算符**

1. `$<BOOL:string>` 如果字符串为空，0。不区分大小写的 `FALSE` ， `OFF` ， `N` ， `NO` ， `IGNORE` ， `NOTFOUND` 。或者区分大小写以 `-NOTFOUND` 结尾的字符串，则为 0，否则为 1
2. `$<AND:condition>` 逻辑与， `condition` 是以逗号分割的条件列表
3. `$<OR:condition>` 逻辑或
4. `$<NOT:condition>` 逻辑非

**字符串比较**

1. `$<STREQUAL:str1, str2>` 判断字符串是否相等
2. `$<EQUAL:val1, val2>` 判断数值是否相等
3. `$<IN_LIST:str, list>` 判断字符串是否在列表 `list` 中

**变量查询**

1. `$<TARGET_EXISTS:target>` 判断目标是否存在
2. `$<CONFIG:cfg>` 判断编译类型配置是否包含在 `cfg` 列表中，不区分大小写
3. `$<PLATFORM_ID:platform_ids>` 判断cmake定义的平台ID是否包含在 `platform_ids` 列表中
4. `$<COMPILE_LANGUAGE:languages>` 判断编译语言是否包含在 `languages` 列表中

### 字符串值生成器表达式

使用生成表达式的目的是生成特殊的字符串

**条件表达式**

主要是两种格式

1. `$<condition:true_string>` 如果条件为真，则结果为 `true_string` ，否则为空
2. `$<IF:condition,str1,str2>` 如果条件为真，结果为 `str1` ，否则 `str2`

**转义字符**

由于有些字符有特殊含义，可能需要转义，例如

- `$<COMMA>` 表示 `,`
- `$<SEMICOLON>` 表示 `;`

**字符串操作**

- `$<LOWER_CASE:string>` 将字符串转为小写
- `$<UPPER_CASE:string>` 将字符串转为大写

**获取变量值**

与上文中的变量查询很类似

1. `$<CONFIG>` 获取变量值
2. `$<CONFIG:cfgs>` 判断是否存在于列表中

**编译目标信息**

更多信息：[target-dependent-queries](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html#target-dependent-queries)

编译目标：是指通过 `add_execuate()` 和 `add_library()` 指令生成的目标文件和库文件

1. `$<TARGET_FILE:tgt>` 获取编译目标 `tgt` 的文件路径
2. `$<TARGET_FILE_NAME:tgt>` 获取编译目标的文件名
3. `$<TARGET_FILE_BASE_NAME:tgt>` 获取编译目标的基础名字，也就是文件名去除前缀和扩展名

### 调试信息

1. 通过输出到文件的方式，在 `cmake` 执行完之后检查是否符合预期
    
    ```
    file(GENERATE OUTPUT "./ouput.txt" CONTENT "$<$<CONFIG:Debug>:-g;-O0>,$<PLATFORM_ID>\n")
    ```
    
    执行完 `cmake` 之后在文件中得到生成器表达式的内容
    
2. 添加一个自定义目标
    
    ```
    add_custom_target(gentest COMMAND ${CMAKE_COMMAND} -E echo "\"$<$<CONFIG:Debug>:-g;-O0>,$<PLATFORM_ID>\"")
    ```
    
    这里需要将双引号转义，以便确保生成器表达式展开之后为字符串
    
    在执行 `cmake` 之后，可以使用 `make gentest` 命令输出到生成器表达式的内容
