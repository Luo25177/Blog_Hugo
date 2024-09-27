---
title: "clang-tidy配置"
date: 2024-09-05 20:51:43
categories: "程序员的自我涵养"
tags: ["clang-tidy", "clang"]
summary: "clang-tidy 基础配置，用于约束代码书写的规范形式，养成良好的代码书写习惯"
---

## 前言

### 介绍

Clang-Tidy 是一个由 LLVM 项目提供的开源静态分析工具，用于进行静态代码分析和代码质量改进。利用 Clang 编译器强大的功能来对 C++ 代码进行静态分析，并且提供一系列代码改进建议和警告

Clang-Tidy 是基于 Clang 的 AST 抽象语法树进行分析的，并且能检测到许多常见的编码错误和代码风格问题，包括语法，逻辑，性能问题和风格问题等

### 安装

**windows**

[Download LLVM releases](https://releases.llvm.org/)

直接从上述网站上找到一个对应的发行版安装即可

然后在命令行中输入如下指令来查看是否安装完成，一般来说安装完成之后需要重新启动一下

```yaml
clang-tidy -version
```

**ubuntu**

[LLVM](https://apt.llvm.org/)

根据该文章的内容来安装

安装最新版 LLVM，需要注意，该指令只能在 `bash` 中使用

```bash
sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
```

安装 Clang-format

```bash
 apt-get install clang-18 clang-tools-18 clang-18-doc libclang-common-18-dev libclang-18-dev libclang1-18 clang-format-18 python3-clang-18 clangd-18 clang-tidy-18 
```

之后可以验证

```bash
llvm-config-18 --version
clang-tidy-18 --version
```

### 原理

Clang-Tidy 的工作原理是将源代码传递给 Clang 编译器，然后通过静态分析找到代码中的问题。Clang 编译器是一个基于 LLVM 的项目，它提供了一个强大的 C++ 前端，能解析，编译和优化 C++ 代码。而 Clang-Tidy 则利用了 Clang 编译器这些功能，对源代码进行深度分析，并且找出其中可能存在的问题

### 原文地址

[Clang 20.0.0git 文档 (llvm.org)](https://clang.llvm.org/docs/analyzer/checkers.html#core-bitwiseshift)

[clang-tidy - Clang-Tidy Checks — Extra Clang Tools 20.0.0git documentation (llvm.org)](https://clang.llvm.org/extra/clang-tidy/checks/list.html)

## 配置文件

### 编写规则

- 配置文件名必须为 `.clang-tidy`
- 该文件必须位于项目的根目录或者编译目录下
- 文件采用 `YAML` 语法
- 每个 `YAML` 条目包含一下几个关键字
    - `Checks`
    - `WarningsAsErrors`
    - `IncludePaths`
    - `User`
    - `CheckOptions`

### Checks

- **静态分析器**
    - `clang-analyzer-core`
        - `BitwiseShift` 查找由按位左移和右移运算符引起的未定义行为，对整数类型进行操作。默认此检查器仅报告右操作数为负数或左操作数右操作数类型的位宽
        - `CallAndMessage` 检查函数调用和 ObjC 消息表达式的逻辑错误，主要是未初始化的参数，空函数指针
        - `DivideZero` 检查是否除以 0
        - `NonNullParamChecker` 检查作为参数传递给标记为 `nonnull` 属性的函数的是否为 `null` 指针
        - `NullDereference` 检查 `null` 指针的非法使用，野指针
        - `StackAddressEscape` 检查栈的内存地址不会被外部引用，栈的内存会在函数结束时被释放，所以如果被外部引用，就会引发安全漏洞
        - `UndefinedBinaryOperatorResult` 检查二元运算是否有未定义的结果，一般是未初始化
        - `VLASize` 检查数组定义时，数组的长度是否为 0，负数和未定义。还会对未知的长度做出警告（除非做判断保证为正数）
        - `uninitialized.ArraySubscript` 检查定义数组长度的变量是否未定义
        - `uninitialized.Assign` 检查并避免为变量分配未初始化的值
        - `uninitialized.Branch` 检查未初始化的值是否用于分支控制语句
        - `uninitialized.CapturedBlockVariable` 检查在代码块中是否使用未初始化的值
        - `uninitialized.UndefReturn` 返回未初始化的值
        - `uninitialized.NewArraySize` 在 `new array` 中是否使用未初始化的值
    - `clang-analyzer-cplusplus`
        - `cplusplus.ArrayDelete` 使用父类指针指向 `new array` 子类的结构体，然后试图 `delete array` 这个父类指针。对类型不正确的指针使用 `delete array`
        - `cplusplus.InnerPointer` 检查 C++ 容器内部指针在 `realloc` 或者 `dealloc` 之后使用。C++标准库中的许多容器方法都已知会使指向容器元素的引用（包括实际的引用，迭代器和原始指针）失效，这是 C++ 常见的内存错误来源
        - `cplusplus.Move` 检查 C++ 中是否出现变量移动之后使用，也就是对象被移动之后依旧对对象做一些不当行为
        - `cplusplus.NewDelete` 检查是否有二次 `delete` 和 `delete` 之后再次使用的问题
        - `cplusplus.NewDeleteLeaks` 检查是否内存被申请之后未被释放，应当通过 `new/delete` 进行内存管理
        - `cplusplus.PlacementNew` 检查是否提供了默认的 `new` 操作符，并且是否以指向足够内存空间的指针作为参数
        - `cplusplus.SelfAssignment` 检查 C++ 中的拷贝和移动运算符是否存在自赋值
        - `cplusplus.StringChecker` 字符串检查，检查用于构造字符串的参数是否有效
    - `clang-analyzer-deadcode`
        - `DeadStores` 检查是否存在赋值给一个从未被读取的变量
    - `clang-analyzer-nullability`
        - `NullPassedToNonnull` 当向一个被标记为 `_Nonnull` 类型的指针传递一个空指针（null pointer）时，编译器或静态分析工具会发出警告
        - `NullReturnedFromNonnull` 当一个函数的返回类型被标记为 `_Nonnull`（即不允许返回空指针），但该函数实际上返回了一个空指针时，编译器或静态分析工具会发出警告。
        - `NullableDereferenced` 当尝试访问（或“解引用”）一个可能为 `null` 的指针时，会发出警告
        - `NullablePassedToNonnull` 当一个可能为空的指针（即可空指针）被传递给一个期望非空指针（通过 `_Nonnull` 标记）的函数或方法时，编译器或静态分析工具会发出警告
        - `NullableReturnedFromNonnull` 在编程时，如果某个函数被声明为返回一个非空 `_Nonnull` 指针，但是该函数实际上返回了一个可能为空的指针（即 `nullable` 指针），编译器或静态分析工具会发出警告
    - `clang-analyzer-optin`
        - `core.EnumCastOutOfRange` 在检查将整数转换为枚举（enumeration）类型时，是否存在转换成了一个没有对应枚举量的值的情况，即超出了枚举的范围
        - `cplusplus.UninitializedObject` 此检查器报告在构造函数调用后创建的对象中未初始化的字段。它不仅查找直接的未初始化字段，而且对对象进行深入检查，分析其字段的所有子字段。检查器将继承字段视为直接字段，因此也会收到未初始化继承数据成员的警告
        - `cplusplus.VirtualCall` 在构造或销毁过程中检查是否虚函数调用
        - `mpi.MPI-Checker` 检查 MPI 代码
        - `osx.cocoa.localizability.EmptyLocalizationContextChecker` 检查 `NSLocalizedString` 宏是否包含上下文注释
        - `osx.cocoa.localizability.NonLocalizedStringChecker` 警告使用传递给期望本地化 `nsstring` 的 UI 方法的非本地化 `nsstring`
        - `performance.GCDAntipattern` 在使用 Grand Central Dispatch 时检查性能反模式
        - `performance.Padding` 检查过度填充的结构。此检查器检测具有过多填充的结构，这可能导致内存浪费，从而通过降低处理器缓存的有效性来降低性能。填充字节由编译器添加以对齐数据访问，因为某些处理器需要将数据对齐到某些边界。在其他情况下，不对齐的数据访问是可能的，但会带来更大的延迟。
        - `portability.UnixAPI` 在 UNIX/Posix 函数中查找实现定义的行为。
        - `taint.TaintedAlloc` 当无法证明申请分配的内存在合法范围内时警告
    - `clang-analyzer-security`
        - `cert.env.InvalidPtr` 对应 SEI CERT 规则 ENV31-C 和 ENV34-C
        - `FloatLoopCounter` 警告使用浮点值作为循环计数器
        - `insecureAPI.UncheckedReturn` 对必须始终检查返回值的函数的使用发出警告
        - `insecureAPI.bcmp` 对使用 `bcmp` 函数发出警告
        - `insecureAPI.bcopy` 对使用 `bcopy` 函数发出警告
        - `insecureAPI.bzero` 对使用 `bzero` 函数发出警告
        - `insecureAPI.getpw` 对使用 `getpw` 函数发出警告
        - `insecureAPI.gets` 对使用 `gets` 函数发出警告
        - `insecureAPI.mkstemp` 当传入函数 `mkstemp` 的格式字符串少于 6 个 X 时发出警告
        - `insecureAPI.mktemp` 对使用 `mktemp` 函数发出警告
        - `insecureAPI.rand` 警告使用劣质随机数生成函数
        - `insecureAPI.strcpy` 对使用 `strcpy`函数发出警告
        - `insecureAPI.vfork` 对使用 `vfork`函数发出警告
        - `insecureAPI.DeprecatedOrUnsafeBufferHandling` 警告不安全或已弃用的缓冲区处理函数，这些函数现在有一个安全的变体
        - `MmapWriteExec` 对具有可写和可执行权限的 `mmap` 函数调用发出警告
        - `PutenvStackArray` 查找对传递指向堆栈分配（自动）数组的指针作为参数的函数的调用。函数不复制传递的字符串，只存储指向数据的指针，并且该数据甚至可以被其他线程读取。从函数 `putenvputenv` 退出后，堆栈分配数组的内容可能会被覆盖。这个问题可以通过使用静态数组变量或动态分配内存来解决。更好的方法是避免使用 `putenvputenv` （它有其他与内存泄漏相关的问题），而是使用 `putenvsetenv`
        - `SetgidSetuidOrder` 当通过使用和调用来删除程序中的用户级和组级特权时，首先重置组级特权是很重要的。如果超级用户权限已经被删除，函数可能会失败检查器检查和调用的序列(按此顺序)。如果找到这样一个序列，并且中间没有其他改变特权的函数调用（和它们的 GID 版本），则会生成一个警告
    - `clang-analyzer-unix`
        - `API` 检查对各种 UNIX/Posix 函数的调用
        - `BlockInCriticalSection` 检查临界区内对阻塞函数的调用
        - `Errno` 检查使用不当
        - `Malloc` 检查内存泄漏、双重自由和自由后使用问题。跟踪由 `malloc()/free()` 管理的内存。
        - `MallocSizeof` 检查涉及 `malloc(sizeof())` 的可疑参数
        - `MismatchedDeallocator` 检查不匹配的析构函数
        - `Vfork` 检查 `vfork` 函数的正确使用
        - `cstring.BadSizeArg` 检查传入 C 字符串函数的 `size` 参数是否存在常见的错误模式。使用编译器选项来静音其他相关的编译器警告
        - `cstring.NullArg` 检查作为参数传递给C字符串函数的空指针
        - `StdCLibraryFunctions` 检查标准库函数的调用是否违反预定义的参数约束
        - `Stream` 检查 C 流处理函数
    - `clang-analyzer-osx`
        - `API` 检查各种 Apple api 的正确使用
        - `NumberObjectConversion` 检查表示数字的对象转换到数字的错误转换
        - `ObjCProperty` 检查对 ObjC 属性的正确使用
        - `SecKeychainAPI` 检查安全钥匙链 api 的正确使用
        - `cocoa.AtSync` 检查 ObjC 中 `@synchronized` 中用作互斥锁的 `nil` 指针
        - `cocoa.AutoreleaseWrite` 警告在 ObjC 中从不同的自动释放池写入自动释放对象可能会崩溃
        - `cocoa.ClassRelease` 检查是否将 `retain` ， `release` 或 `autorelease` 直接发送给类
        - `cocoa.Dealloc` 警告缺少 `-dealloc` 正确实现的 ObjC 类
        - `cocoa.IncompatibleMethodTypes` 警告带有类型不兼容的 ObjC 方法签名
        - `cocoa.Loops` 使用 Cocoa 集合类型改进循环建模
        - `cocoa.MissingSuperCall` 警告缺少必要的 super 调用的 ObjC 方法
        - `cocoa.NSAutoreleasePool` 在 ObjC GC 模式下对 `NSAutoreleasePool` 的次优使用发出警告
        - `cocoa.NSError` 检查 NSError 参数的使用情况
        - `cocoa.NilArg` 检查 ObjC 方法调用中禁止的 nil 参数
        - `cocoa.NonNilReturnValue` 对保证返回非 nil 值的 api 进行建模
        - `cocoa.ObjCGenerics` 在使用 ObjC 泛型时检查类型错误
        - `cocoa.RetainCount` 检查泄漏和不适当的参考计数管理
        - `cocoa.RunLoopAutoreleaseLeak` 检查自动释放池中永远不会耗尽的泄漏内存
        - `cocoa.SelfInit` 检查' self '是否在初始化方法中被正确初始化
        - `cocoa.SuperDealloc` 警告在 ObjC 中不正确使用 `[super dealloc]`
        - `cocoa.UnusedIvars` 警告永远不会使用的私有变量
        - `cocoa.VariadicMethodTypes` 检查是否将非 ObjC 类型传递给只期望 ObjC 类型的可变集合初始化方法
        - `coreFoundation.CFError` 检查 CFErrorRef 参数的使用情况
        - `coreFoundation.CFNumber` 检查 CFNumber api 的正确使用
        - `coreFoundation.CFRetainRelease` 检查 CFRetain/CFRelease/CFMakeCollectable 的空参数
        - `coreFoundation.containers.OutOfBounds` 当使用 CFArray api 时检查索引越界
        - `coreFoundation.containers.PointerSizedValues` 如果 CFArray， CFDictionary， CFSet 使用非指针大小的值创建，则发出警告
    - `clang-analyzer-Fuchsia`
        - `HandleChecker` 句柄标识资源。与指针类似，它们可以泄漏、双重释放或在释放后使用。此检查试图查找此类问题
    - `clang-analyzer-WebKit`
        - `RefCntblBaseVirtualDtor` 所有用作基类的未计数类型都必须具有虚析构函数。引用计数类型通过原始指针保存其可引用的数据，并允许从引用计数的派生类型指针隐式向上转换为指向基类型的引用计数指针。这可能导致(动态)派生类型的对象通过指向基类类型（C++标准定义为 UB）的指针被删除，如果基类没有虚析构函数
        - `NoUncountedMemberChecker` 未计数类型的原始指针和引用不能用作类成员。只允许重新计数的类型
        - `UncountedLambdaCapturesChecker` 在 `lambdas` 中无法捕获未计数类型的原始指针和引用。只允许重新计数的类型
- **现代化**
    - `modernize-avoid-bind` 使用 `lambda` 替换 `std::binding`
    - `modernize-deprecated-headers` 将 C 标准库的头文件 `include` 替换成 C++style， `#include <assert.h>` => `#include <cassert>`
    - `modernize-loop-convert` 使用 `for-range loop` 替换 `for(...;...;...;)` ，并更新`for`语句的相关变量。
    - `modernize-make-shared` 找出所有显式创建 `std::shared_ptr` 变量的表达式，并使用 `make_shared` 替换
    - `modernize-make-unique` 跟 `make-shared` 一样，使用 `std::make_unique` 替换所有 `std::unique_ptr` 显式创建表达式
    - `modernize-pass-by-value` 在构造函数中使用 `move` 语义
    - `modernize-raw-string-literal` 用 C++11 的 `raw string literal(R"...")` 替换原来的 `string literal` ，这样的好处就是不用再添加转义符 `\` 了
    - `modernize-redundant-void-arg` 去掉`void`函数参数
    - `modernize-replace-auto-ptr` 用 `std::unique_ptr` 替换 `std::shared_ptr` ， `std::shared_ptr` 是不推荐使用的，即使在 C++98
    - `modernize-shrink-to-fit` 在 C++03 中，如果我们想修改STL容器的 `capacity` ，只能通过 `copy & swap` 的方式，C++11 提供了 `shink_to_fit` 的方法
    - `modernize-use-auto` 在变量定义的时候，使用 `auto` 代替显式的类型声明，这个在定义 STL 容器类的 `Iterator` 特别方便
    - `modernize-use-bool-literals` 找出所有隐式从 `int` 转成 `bool` 的 `literal` ，使用 `true` 或者 `false` 代替
    - `modernize-use-default` 对于没有任何自定义行为（定义为 `{}` ）的特殊的成员函数，构造函数，析构函数，移动/复制构造函数，用 `=default` 代替掉 `{}`
    - `modernize-use-emplace` 使用STL容器中的 `emplace` 代替 `push_back`
    - `modernize-use-equals-delete` 在 C++98 中，类设计为了实现禁止调用某些特殊的成员函数，通常把它们声明成 `private` 。在 C++11 中，只需要在声明中添加 `=delete` ，找出所有 `private` 的特殊成员函数，并将它们标记成 `=delete`
    - `modernize-use-nullptr` 用 `nullptr` 代替 `NULL`
    - `modernize-use-override` 对于子类改写父类的 `virtual` 方法，在方法后面添加`override` ，并删掉 `virtual` 前缀
    - `modernize-use-using` 用 `using` 代替 `typedef`
    - `modernize-use-equals-default` 此检查将特殊成员函数的默认体替换为 `= default` 。显式默认的函数声明为优化提供了更多的机会，因为编译器可能会将显式默认的函数视为微不足道的
- **Google 代码风格**
    - `google-explicit-constructor` 检查并建议在单参数构造函数中使用 `explicit` 关键字。防止隐式转换带来的意外行为
    - `google-readability-casting` 检查并建议使用 C++ 风格的类型转换（如 `static_cast` ， `dynamic_cast` ， `const_cast` 和 `reinterpret_cast`）代替 C 风格的类型转换。提高类型转换的安全性和可读性
    - `google-build-explicit-make-pair` 检查 `make_pair` 的模板参数是否推导出来了。
    - `google-build-namespaces` 在标头中查找匿名名称空间
    - `google-default-arguments` 检查虚拟方法是否给出默认参数。
    - `google-readability-avoid-underscore-in-googletest-name` 检查在 googletest 测试套件名称和测试宏中的测试名称中是否有下划线
    - `google-runtime-int` 查找有符号整型，并且尝试使用对应的无符号整型代替
    - `google-runtime-operator` 查找一元操作符 & 的重载
- **可读性**
    - `readability-braces-around-statements` 检查判断语句和循环语句的代码提是否在大括号内
    - `readability-non-const-parameter` 该检查查找指针类型的函数形参，可以将其更改为 `const` 类型的指针，也就是常量指针
    - `readability-avoid-const-params-in-decls` 检查函数声明是否具有顶层 `const` 形参
    - `readability-const-return-type` 检查具有 `const` 限定返回类型的函数，并建议删除 `const` 关键字。这样使用 `const` 通常是多余的，并且会妨碍有价值的编译器优化
    - `readability-container-size-empty` 检查对 `size()/length()` 方法的调用是否可以替换为对 `empty()` 的调用。
    - `readability-convert-member-functions-to-static` 查找可以设置为静态的非静态成员函数，因为这些函数不使用 `this`
    - `readability-delete-null-pointer` 在检查指针是否存在的 `if` 语句中检查，然后删除该指针。该检查是不必要的，因为删除空指针没有效果
    - `readability-deleted-default` 检查标记为 `= default` 的构造函数和赋值操作符是否被编译器实际删除
    - `readability-make-member-function-const` 查找可以设置为 `const` 的非静态成员函数，因为这些函数没有以非 `const` 的方式使用它
    - `readability-misplaced-array-index` 此检查警告不寻常的数组索引语法
    - `readability-qualified-auto` 向推导为指针的自动类型化变量添加指针限定条件。LLVM 编码标准建议，如果自动类型化变量是指针，则应明确表示。当类型被推断为指针时，此检查将 `auto` 转换为 `auto *`
    - `readability-redundant-access-specifiers` 查找包含冗余成员（字段和方法）访问说明符的类，结构和联合
    - `readability-redundant-control-flow` 查找在不返回值的函数末尾有返回语句，这样的返回语句是多余的
    - `readability-redundant-function-ptr-dereference` 查找函数指针的冗余解引用
    - `readability-redundant-smartptr-get` 查找并删除对智能指针的 `get()` 方法的冗余调用
    - `readability-redundant-string-cstr` 查找对 `std::string::c_str()` 和 `std::string::data()` 的不必要调用
    - `readability-redundant-string-init` 查找不必要的字符串初始化
    - `readability-static-definition-in-anonymous-namespace` 在匿名命名空间中查找静态函数和变量定义，不能存在静态函数和变量
    - `readability-string-compare` 使用 `compare` 方法查找字符串比较，不再使用 `compare` 来比较字符串，而是直接使用 `=,!=` 来比较
    - `readability-uniqueptr-delete-release` 将 `delete` 和 `<unique_ptr>.release()` 替换为 `<unique_ptr> = nullptr` 。后者更短，更简单，并且不需要使用原始指针 api
    - `readability-redundant-member-init` 查找不必要的成员初始化，因为如果不存在相同的默认构造函数，则会调用这些初始化
    - `readability-simplify-subscript-expr` 这种检查简化了下标表达式。目前这包括调用 `.data()` 并立即执行数组下标操作以获取单个元素，在这种情况下，只需调用 `operator[]` 就足够了
    - `readability-simplify-boolean-expr` 查找包含布尔常量的布尔表达式，并将其简化为直接使用适当的布尔表达式。应用德摩根定理简化布尔表达式
    - `readability-inconsistent-declaration-parameter-name` 查找参数名称不同的函数声明，在不同文件中函数声明中的形参不一样的情况
    - `readability-identifier-naming` 此检查将尝试在标识符命名上强制执行编码准则。它支持以下一种套管类型，并在检测到不匹配时尝试从一种转换为另一种
        - `lower_case` 小写下划线
        - `UPPER_CASE` 大写下划线
        - `camelBack` 小写开头驼峰
        - `CamelCase` 大驼峰
        - `camel_Snake_Back` 小驼峰+下划线
        - `Camel_Snake_Case` 大驼峰+下划线
        - `aNy_CasE`
        - `Leading_upper_snake_case` 大写开头+小写下划线
- **CERT 安全编码标准**
    - `cert-dcl21-cpp`
    - `cert-dcl50-cpp` 此检查标记c风格可变函数的所有函数定义（但不标记声明）
    - `cert-env33-c` 此检查标记对 `system()` ， `popen()` 和 `_popen()` 的调用，它们执行命令处理程序。它不会用空指针参数标记对 `system()` 的调用，因为这样的调用会检查是否存在命令处理器，但实际上不会尝试执行命令
    - `cert-err34-c` 此检查标记对字符串到数字转换函数的调用，这些函数不验证转换的有效性，例如 `atoi()` 或 `scanf()` 。它不标记对 `strtol()` 或其他相关转换函数的调用，这些函数确实执行了更好的错误检查
    - `cert-err52-cpp` 此检查标记所有涉及 `setjmp()` 和 `longjmp()` 的调用表达式。
    - `cert-flp30-c` 此检查标记归纳表达式为浮点类型的循环
    - `cert-mem57-cpp` 当类型具有扩展对齐（大于基本对齐）时，此检查标记使用默认操作符 `new` 。（如果请求的对齐小于或等于基本对齐，则默认操作符 `new` 保证提供正确的对齐）。只有在操作符 `new` 不是用户定义的并且不是位置 `new` 的情况下（原因是在这些情况下，我们假设用户提供了正确的内存分配）才会检测到
    - `cert-msc50-cpp` 伪随机数生成器使用数学算法生成具有良好统计特性的数字序列，但生成的数字并不是真正随机的。 `rand()` 函数接受一个种子（数字），对其进行数学运算并返回结果。通过操纵种子，结果是可以预测的。这个检查警告 `std::rand()` 的使用
    - `cert-oop58-cpp` 在复制构造函数和复制赋值操作符中查找对复制对象及其直接或间接成员的赋值
- **Bug 检测**
    - `bugprone-undelegated-constructor` 查找构造函数中临时对象的创建，这些对象看起来像对同一类的另一个构造函数的函数调用，用户最有可能使用委托构造函数或基类初始化项。
    - `bugprone-macro-parentheses` 查找由于缺少括号而可能产生意外行为的宏
    - `bugprone-macro-repeated-side-effects` 检查宏中具有副作用的重复参数
    - `bugprone-forward-declaration-namespace` 检查未使用的前向声明是否在错误的命名空间中。该检查检查所有未使用的前向声明，并检查是否存在同名的声明/定义，这可能表明前向声明位于可能错误的名称空间中
    - `bugprone-bool-pointer-implicit-conversion` 根据从 `bool` 指针到 `bool` 指针的隐式转换检查条件
    - `bugprone-misplaced-widening-cast` 当将计算结果强制转换为更大的类型时，此检查将发出警告。如果 `cast` 的目的是为了避免精度的损失，那么 `cast` 是错位的，并且可能会有精度的损失。否则 `cast` 是无效的
    - `bugprone-argument-comment` 检查实参注释是否与形参名称匹配
    - `bugprone-bad-signal-to-kill-thread` 当线程通过引发 `SIGTERM` 信号终止时，查找 `pthread_kill` 函数调用，该信号会终止整个进程，而不仅仅是单个线程。使用除 `SIGTERM` 以外的任何信号
    - `bugprone-copy-constructor-init` 查找未调用基类复制构造函数的复制构造函数
    - `bugprone-dangling-handle` 检测值句柄中的悬空引用，如 `std::string_view` 。这些悬空引用可能是由临时值构造句柄的结果，其中临时句柄在创建后很快被销毁
    - `bugprone-fold-init-type` 在像 `std::accumulate` 这样的折叠中，检查标志类型不匹配可能导致精度损失。 `std::accumulate` 使用后者的类型将输入范围折叠成初始值，默认情况下使用操作符 `+` 。
    - `bugprone-inaccurate-erase` 检查 `erase()` 方法的不正确使用。像 `remove()` 这样的算法实际上并没有从容器中删除任何元素，而是返回一个迭代器，指向容器末尾的第一个冗余元素。这些多余的元素必须使用 `erase()` 方法删除。当由于使用不适当的重载而不能删除所有元素时，此检查将发出警告
    - `bugprone-incorrect-roundings` 检查已知会产生不正确舍入的模式的使用情况
    - `bugprone-integer-division` 查找浮点上下文中整数除法可能导致意外精度损失的情况
    - `bugprone-misplaced-operator-in-strlen-in-alloc` 查找在 `strlen()` ， `strnlen()` ， `strnlen_s()` ， `wcslen()` ， `wcsnlen()` 和 `wcsnlen_s()` 的参数中添加1的字符串而不是结果的情况，并且该值用作内存分配函数（ `malloc()` ， `calloc()` ， `realloc()` ， `alloca()` ）或 C++ 中的新[]运算符的参数。即使这些函数中的一个（ `new[]` 操作符除外）被常量函数指针调用，该检查也会检测错误情况。将参数加 1 和 `strlen()` 类函数的结果的情况将被忽略，整个加法被额外的圆括号包围的情况也是如此
    - `bugprone-misplaced-pointer-arithmetic-in-alloc` 查找在内存分配函数（ `malloc()` ， `calloc()` ， `realloc()` ， `alloca()` ）的结果中添加或减去整数表达式而不是其参数的情况。即使这些函数中的一个被常量函数指针调用，检查也会检测错误情况
    - `bugprone-move-forwarding-reference` 如果在转发引用上调用 `std::move` ，则发出警告
    - `bugprone-multiple-statement-macro` 检测在无括号条件中使用的多个语句宏。只有宏的第一个语句将在条件语句中，其他语句将无条件执行
    - `bugprone-parent-virtual-call` 检测并修复对父类虚拟方法的调用，而不是对重写父类虚拟方法的调用
    - `bugprone-posix-return` 检查是否有任何调用 `pthread_*` 或 `posix_*` 函数（ `posix_openpt` 除外）期望负返回值。这些函数成功时返回 0，失败时返回 `errno` ，但是他们返回值仅为正数
    - `bugprone-reserved-identifier` 检查为实现保留使用的标识符的用法。C 标准还保留了以双下划线开头的名称，而 C++ 标准加强了这一点，保留了出现在任何地方的双下划线名称。所以不能使用这些标识符
    - `bugprone-signed-char-misuse` 查找可能指示编程错误的 `signed char -> int` 转换。带符号字符的基本问题是，它可能将非 ascii 字符存储为负值。当显式转换和隐式转换发生时，这种行为都可能导致对所编写代码的误解
    - `bugprone-sizeof-container` 该检查在 STL 容器类型的表达式中查找 `sizeof` 的用法。`*sizeof() doesn't return the size of the container.*`
    - `bugprone-sizeof-expression` 检查查找 `sizeof` 表达式的用法，这些用法最有可能是错误的。 `sizeof` 操作符产生其操作数的大小（以字节为单位），该操作数可以是表达式或类型的括号名称。误用此操作符可能会导致错误和可能的软件漏洞
    - `bugprone-string-constructor` 查找可疑且可能错误的字符串构造函数
    - `bugprone-string-integer-assignment` 查找用整形对 `std::basic_string<CharT>` （ `std::string` ， `std::wstring` 等）的赋值
    - `bugprone-string-literal-with-embedded-nul` 查找包含 `NULL` 字符的字符串字面值并验证其用法
    - `bugprone-suspicious-enum-usage` 检查器检测枚举可能被滥用的各种情况（作为位掩码）
    - `bugprone-suspicious-include` 当包含引用了看起来像是源文件的内容时，检查会检测到各种情况，这通常会导致难以跟踪的 ODR 违规
    - `bugprone-suspicious-memset-usage` 此检查查找参数中可能存在错误的 `memset()` 调用
    - `bugprone-suspicious-missing-comma` 并排放置的字符串字面值在翻译阶段6（在预处理器之后）被连接起来。此特性用于表示多行上的长字符串文字
    - `bugprone-suspicious-string-compare` 查找运行时字符串比较函数的可疑用法。该检查在 C 和 C++ 中有效
    - `bugprone-swapped-arguments` 通过检查隐式转换查找可能交换的参数。它分析传递给函数的参数类型，并将它们与相应参数的预期类型进行比较。如果存在不匹配或指示潜在交换的隐式转换，则会引发警告
    - `bugprone-terminating-continue` 检测带有 `continue` 语句的条件总是求值为 `false` 的 `do while` 循环，因为该 `continue` 语句有效地终止了循环
    - `bugprone-throw-keyword-missing` 警告可能缺少 `throw` 关键字。如果创建了一个临时对象，但该对象的类型派生于一个名称中包含 `EXCEPTION` ， `EXCEPTION` 或 `EXCEPTION` 的类相同，我们可以假设程序员的意图是抛出该对象
    - `bugprone-too-small-loop-variable` 检测那些具有太小类型的循环变量的 `for` 循环，这意味着该类型不能表示迭代范围内的所有值。也就是循环使用的数据类型大小小于循环次数
    - `bugprone-undefined-memory-manipulation` 查找对 `non-TriviallyCopyable` 对象调用内存操作函数 `memset()` ， `memcpy()` 和 `memmove()` 导致未定义行为
    - `bugprone-unhandled-self-assignment` 查找用户定义的复制赋值操作符，这些操作符不通过显式检查自赋值或使用复制-交换或复制-移动方法来保护代码免受自赋值。默认情况下，这个检查只搜索那些有指针或C数组字段的类，以避免误报。在指针或C数组的情况下，如果没有小心地编写复制赋值操作符，则自复制赋值很可能会破坏对象
    - `bugprone-unused-raii` 查找看起来像 RAII 对象的临时对象
    - `bugprone-unused-return-value` 警告未使用的函数返回值。带有赋值语义的操作符重载将被忽略
    - `bugprone-use-after-move` 如果对象在移动后被使用，则发出警告
    - `bugprone-virtual-near-miss` 如果函数与基类中的虚函数近似（即名称非常相似且函数签名相同），则发出警告
- **C++ 核心指南**
    - `cppcoreguidelines-narrowing-conversions`
    - `cppcoreguidelines-pro-type-reinterpret-cast` 该检查标记 C++ 代码中所有 `reinterpret_cast` 的使用。
    - `cppcoreguidelines-pro-type-member-init` ****该检查标记了用户提供的构造函数定义，这些定义没有初始化所有字段，这些字段将在默认构造函数中处于未定义状态，例如内置函数，指针和记录类型，而用户提供的默认构造函数中至少包含一个这样的类型。如果这些字段没有初始化，构造函数将使一些内存处于未定义状态
- **杂项**
    - `misc-unconventional-assign-operator` 查找返回值或参数类型错误的赋值操作符声明，以及返回值类型良好但返回语句错误的赋值操作符定义
    - `misc-unused-parameters` 查找未使用的函数参数。未使用的参数可能表示代码中存在错误（例如，当使用不同的参数时）。如果函数的所有调用者都在相同的翻译单元中并且可以更新，则建议修复注释参数名称或完全删除参数
    - `misc-throw-by-value-catch-by-reference` 查找违反**按值抛出，按引用捕获**规则的情况
    - `misc-misplaced-const` 当 `const` 限定符应用于类型定义/使用指针类型而不是指针类型时，该检查会进行诊断，因为这样的结构通常会误导开发人员，因为 `const` 应用于指针而不是指针数据
    - `misc-redundant-expression` 检测多余的表达式，这些表达式通常是由于复制粘贴而导致的错误
    - `misc-static-assert` 如果条件在编译时可求值，则将 `assert()` 替换为 `static_assert()`
    - `misc-uniqueptr-reset-release` 查找并替换 `unique_ptr::reset(release())` 为 `std::move()`
    - `misc-unused-alias-decls` 查找未使用的命名空间别名声明
    - `misc-unused-using-decls` 查找未使用的 `using` 声明
- `hicpp`
    - `hicpp-exception-baseclass` 确保 `throw` 表达式中的每个值都是 `std::exception` 的实例
    - `hicpp-ignored-remove-result` 确保 `std::remove` ， `std::remove_if` 和 `std::unique` 的结果不会被忽略。变异算法 `std::remove` ， `std::remove_if` 和 `std::unique` 都是通过交换或移动它们所操作的范围内的元素来进行操作的。完成后，它们返回一个指向最后一个有效元素的迭代器。在大多数情况下，正确的做法是将此结果用作调用 `std::erase` 的第一个操作数
    - `hicpp-multiway-paths-covered` 此检查发现代码路径未完全覆盖的情况。此外，如果代码更清晰，建议使用 `if` 而不是 `switch` 。缺失最后一个 `else` 分支的 `if-else if` 链可能会导致意外的程序执行，并成为逻辑错误的结果。如果遗漏的else分支是有意的，可以将其保留为空，并添加澄清注释。在某些代码库中，此警告可能会产生噪声，因此默认情况下禁用此警告
    - `hicpp-no-assembler` 检查汇编语句。应该避免使用内联汇编，因为它限制了代码的可移植性
    - `hicpp-signed-bitwise` 查找对有符号整数类型的位操作的使用，这可能导致未定义或实现定义的行为
- `performance`
    - `performance-faster-string-find` 优化调用 `std::string::find()` 和友元，当传递的指针是单个字符串字面值时。字符文字重载更有效
    - `performance-for-range-copy` 查找 C++11 中循环变量在每次迭代中复制的范围，但通过 `const` 引用获得它就足够了。这个检查仅适用于复制成本较高的类型的循环变量，这意味着它们不是普通的可复制的，或者具有非普通的复制构造函数或析构函数
    - `performance-implicit-conversion-in-loop` 此警告出现在具有 `const ref` 类型循环变量的基于范围的循环中，其中变量的类型与迭代器返回的类型不匹配。这意味着会发生隐式转换，例如，这可能导致代价高昂的深度拷贝
    - `performance-inefficient-algorithm` 警告在关联容器上低效地使用 STL 算法。关联容器将一些算法作为方法实现，这些方法应该优先于算法头中的算法。这些方法可以利用元素的顺序
    - `performance-inefficient-vector-operation` 查找可能导致不必要内存重新分配的低效 `std::vector` 操作（例如 `push_back, emplace_back` ）
    - `performance-move-constructor-init` 检查标志用户定义的移动构造函数，这些构造函数具有通过复制构造函数而不是移动构造函数初始化成员或基类的变量初始化器
    - `performance-no-automatic-move` 在某些条件下，从函数返回时将自动移出局部值。一个常见的错误是将局部左值变量声明为 `const` ，这会阻止移动
    - `performance-trivially-destructible` 查找可以通过删除外部默认析构函数声明而使其成为 `trivially destructible` 类型的析构函数。 `trivially destructible` 类型就是那些其析构函数不执行任何操作的类型，这在资源管理性能优化与 C 语言接口等方面很重要，这种类型允许编译器进行更加高效的内存管理和数据传递。
    - `performance-unnecessary-copy-initialization` 查找使用非平凡可复制类型的复制构造函数初始化的局部变量声明，但它足以获得 `const` 引用。非平凡可复制类型是指那些具有自定义拷贝构造函数，拷贝赋值运算符，析构函数或移动构造函数/赋值运算符之一的类型。这些类型在复制时可能需要执行额外的操作，比如内存分配，资源管理等，这些操作可能会比简单地获取一个引用要昂贵得多
- `boost`
    - `boost-use-to-string` 该检查查找使用 `boost::lexical_cast` 从整数类型到 `std::string` 或 `std::wstring` 的转换，并将其替换为对 `std::to_string` 和 `std::to_wstring` 的调用。
    - `boost-use-ranges` 检测对标准库迭代器算法的调用，这些调用可以替换为 Boost 范围版本
- 一般来说会添加额外的一项为 `-*` ，意为未提到的都默认不生效

### WarningsAsErrors

这个就是将对应的警告设置为错误，参数与上述的 `Checks` 中的一致

### CheckOptions

这里面可以添加一些参数，主要是一些用户个性化定义的参数

- `readability-identifier-naming`
    - `AbstractClassCase` 抽象类名称的命名规则
    - `ClassCase` 类名称的命名规则
    - `ClassConstantCase` 类常量名大小写命名规则
    - `ClassMemberCase` 类成员大小写命名规则
    - `ClassMethodCase` 类方法命名规则
    - `ConceptCase` `concept` 修饰符修饰的变量的命名规则
    - `ConstantCase` 常量变量的命名规则
    - `ConstantMemberCase` 常量成员的命名规则
    - `ConstantParameterCase` 常量形式参数的命名规则
    - `ConstantPointerParameterCase` 常量指针形式参数的命名规则
    - `ConstexprFunctionCase` `constexpr` 修饰的函数的命名规则
    - `ConstexprMethodCase` `constexpr` 修饰的方法的命名规则
    - `ConstexprVariableCase` `constexpr` 修饰的变量的命名规则
    - `EnumCase` 枚举名称的命名规则
    - `EnumConstantCase` 枚举常量的命名规则
    - `FunctionCase` 函数命名规则
    - `GlobalConstantCase` 全局常量命名规则
    - `GlobalConstantPointerCase` 全局指针常量命名规则
    - `GlobalFunctionCase` 全局函数命名规则
    - `GlobalPointerCase` 全局指针命名规则
    - `GlobalVariableCase` 全局变量命名规则
    - `InlineNamespaceCase` 内联命名空间名称命名规则
    - `LocalConstantCase` 局部常量名称命名规则
    - `LocalConstantPointerCase` 局部常量指针名称命名规则
    - `LocalPointerCase` 局部指针名称命名规则
    - `LocalVariableCase` 局部变量名称命名规则
    - `MacroDefinitionCase` 宏定义命名规则
    - `MemberCase` 成员名命名规则
    - `MethodCase` 方法名命名规则
    - `NamespaceCase` 命名空间命名规则
    - `ParameterCase` 形式参数名命名规则
    - `ParameterPackCase` 参数包名命名规则，也就是多可变参数类型的参数名
    - `PointerParameterCase` 指针形参命名规则
    - `PrivateMemberCase` 私有成员命名规则
    - `PrivateMethodCase` 私有方法命名规则
    - `ProtectedMemberCase` 保护成员命名规则
    - `ProtectedMethodCase` 保护方法命名规则
    - `PublicMemberCase` 公开成员命名规则
    - `PublicMethodCase` 公开方法命名规则
    - `ScopedEnumConstantCase` 限定范围的枚举常量名命名规则
    - `StaticConstantCase` 静态常量命名规则
    - `StaticVariableCase` 静态变量命名规则
    - `StructCase` 结构体命名规则
    - `TemplateParameterCase` 模板参数命名规则
    - `TemplateTemplateParameterCase` 模板模板参数命名规则
    - `TypeAliasCase` 类型别名命名规则
    - `TypedefCase` `typedef` 名称命名规则
    - `TypeTemplateParameterCase` 类型模板参数名称命名规则
    - `UnionCase` 联合体名称命名规则
    - `ValueTemplateParameterCase` 值模板参数名称命名规则
    - `VariableCase` 变量名称命名规则
    - `VirtualMethodCase` 虚方法命名规则
    - `*IgnoredRegexp` 对于匹配此正则表达式的对应名称标识符，不会强制执行标识符命名检查
    - `*HungarianPrefix` 启用后，检查将确保声明的标识符具有基于声明类型的匈牙利符号前缀
    - `*Prefix` 对应标识符的前缀
    - `*Suffix` 对应标识符的后缀
    - `AggressiveDependentMemberLookup` 当设置为 `true` 时，检查将在依赖基类中查找需要更改的依赖成员引用。这可能导致模板专门化出错，因此默认值为 `false`
    - `CheckAnonFieldInParent` 当设置为 `true` 时，匿名记录中的字段(即匿名联合和结构体)将被视为封闭作用域中的名称，而不是匿名记录的公共成员，以进行名称检查
    - `GetConfigPerFile` 当为 `true` 时，检查将查找声明标识符的配置。当包含的头文件使用不同的样式时非常有用。默认值为 `true`
    - `IgnoreMainLikeFunctions` 当设置为 `true` 时，与 `main` 或 `wmain` 具有相似签名的函数将不会对其参数的名称进行检查。默认值为 `false`

### 一个可供参考的版本

```yaml
Checks: '-*,
  clang-analyzer-core.BitwiseShift,
  clang-analyzer-core.CallAndMessage,
  clang-analyzer-core.DivideZero,
  clang-analyzer-core.NonNullParamChecker,
  clang-analyzer-core.NullDereference,
  clang-analyzer-core.StackAddressEscape,
  clang-analyzer-core.UndefinedBinaryOperatorResult,
  clang-analyzer-core.VLASize,
  clang-analyzer-core.uninitialized.ArraySubscript,
  clang-analyzer-core.uninitialized.Assign,
  clang-analyzer-core.uninitialized.Assign,
  clang-analyzer-core.uninitialized.Branch,
  clang-analyzer-core.uninitialized.CapturedBlockVariable,
  clang-analyzer-core.uninitialized.UndefReturn,
  clang-analyzer-core.uninitialized.NewArraySize,
  
  clang-analyzer-cplusplus.ArrayDelete,
  clang-analyzer-cplusplus.InnerPointer,
  clang-analyzer-cplusplus.Move,
  clang-analyzer-cplusplus.NewDelete,
  clang-analyzer-cplusplus.NewDeleteLeaks,
  clang-analyzer-cplusplus.PlacementNew,
  clang-analyzer-cplusplus.SelfAssignment,
  clang-analyzer-cplusplus.StringChecker,
  
  clang-analyzer-deadcode.DeadStores,
  
  clang-analyzer-nullability.NullPassedToNonnull,
  clang-analyzer-nullability.NullReturnedFromNonnull,
  clang-analyzer-nullability.NullableDereferenced,
  clang-analyzer-nullability.NullablePassedToNonnull,
  clang-analyzer-nullability.NullableReturnedFromNonnull,
  
  clang-analyzer-optin.core.EnumCastOutOfRange,
  clang-analyzer-optin.cplusplus.UninitializedObject,
  clang-analyzer-optin.cplusplus.VirtualCall,
  clang-analyzer-optin.mpi.MPI-Checker,
  clang-analyzer-optin.osx.cocoa.localizability.EmptyLocalizationContextChecker,
  clang-analyzer-optin.osx.cocoa.localizability.NonLocalizedStringChecker,
  clang-analyzer-optin.performance.GCDAntipattern,
  clang-analyzer-optin.performance.Padding,
  clang-analyzer-optin.portability.UnixAPI,
  clang-analyzer-optin.taint.TaintedAlloc,
  
  clang-analyzer-security.FloatLoopCounter,
  clang-analyzer-security.insecureAPI.bcmp,
  clang-analyzer-security.insecureAPI.bcopy,
  clang-analyzer-security.insecureAPI.bzero,
  clang-analyzer-security.insecureAPI.getpw,
  clang-analyzer-security.insecureAPI.gets,
  clang-analyzer-security.insecureAPI.mkstemp,
  clang-analyzer-security.insecureAPI.mktemp,
  clang-analyzer-security.insecureAPI.rand,
  clang-analyzer-security.insecureAPI.strcpy,
  clang-analyzer-security.insecureAPI.vfork,
  clang-analyzer-security.insecureAPI.DeprecatedOrUnsafeBufferHandling,
  clang-analyzer-security.MmapWriteExec,
  clang-analyzer-security.PutenvStackArray,
  
  clang-analyzer-unix.API,
  clang-analyzer-unix.BlockInCriticalSection,
  clang-analyzer-unix.Errno,
  clang-analyzer-unix.Malloc,
  clang-analyzer-unix.MallocSizeof,
  clang-analyzer-unix.MismatchedDeallocator,
  clang-analyzer-unix.Vfork,
  clang-analyzer-unix.cstring.BadSizeArg,
  clang-analyzer-unix.cstring.NullArg,
  clang-analyzer-unix.StdCLibraryFunctions,
  clang-analyzer-unix.Stream,
  
  clang-analyzer-Fuchsia.HandleChecker,
  
  clang-analyzer-WebKit.RefCntblBaseVirtualDtor,
  clang-analyzer-WebKit.NoUncountedMemberChecker,
  clang-analyzer-WebKit.UncountedLambdaCapturesChecker,

  google-build-explicit-make-pair,
  google-build-namespaces,
  google-default-arguments,
  google-explicit-constructor,
  google-readability-casting,
  google-readability-avoid-underscore-in-googletest-name,
  google-runtime-int,
  google-runtime-operator,

  readability-non-const-parameter,
  readability-avoid-const-params-in-decls,
  readability-const-return-type,
  readability-container-size-empty,
  readability-convert-member-functions-to-static,
  readability-delete-null-pointer,
  readability-deleted-default,
  readability-make-member-function-const,
  readability-misplaced-array-index,
  readability-qualified-auto,
  readability-redundant-access-specifiers,
  readability-redundant-control-flow,
  readability-redundant-function-ptr-dereference,
  readability-redundant-smartptr-get,
  readability-redundant-string-cstr,
  readability-redundant-string-init,
  readability-static-definition-in-anonymous-namespace,
  readability-string-compare,
  readability-uniqueptr-delete-release,
  readability-redundant-member-init,
  readability-simplify-subscript-expr,
  readability-simplify-boolean-expr,
  readability-inconsistent-declaration-parameter-name,

  cert-dcl21-cpp,
  cert-dcl50-cpp,
  cert-env33-c,
  cert-err34-c,
  cert-err52-cpp,
  cert-flp30-c,
  cert-mem57-cpp,
  cert-msc50-cpp,
  cert-oop58-cpp,

  modernize-avoid-bind,
  modernize-deprecated-headers,
  modernize-loop-convert,
  modernize-make-shared,
  modernize-make-unique,
  modernize-pass-by-value,
  modernize-raw-string-literal,
  modernize-redundant-void-arg,
  modernize-replace-auto-ptr,
  modernize-shrink-to-fit,
  modernize-use-auto,
  modernize-use-bool-literals,
  modernize-use-default,
  modernize-use-emplace,
  modernize-use-equals-delete,
  modernize-use-nullptr,
  modernize-use-override,
  modernize-use-using,
  modernize-use-equals-default,

  cppcoreguidelines-pro-type-reinterpret-cast,
  cppcoreguidelines-narrowing-conversions,
  cppcoreguidelines-pro-type-member-init,

  misc-unconventional-assign-operator,
  misc-unused-parameters,
  misc-throw-by-value-catch-by-reference,
  misc-misplaced-const,
  misc-redundant-expression,
  misc-static-assert,
  misc-uniqueptr-reset-release,
  misc-unused-alias-decls,
  misc-unused-using-decls,

  bugprone-undelegated-constructor,
  bugprone-macro-parentheses,
  bugprone-macro-repeated-side-effects,
  bugprone-forward-declaration-namespace,
  bugprone-bool-pointer-implicit-conversion,
  bugprone-misplaced-widening-cast,
  bugprone-argument-comment,
  bugprone-bad-signal-to-kill-thread,
  bugprone-copy-constructor-init,
  bugprone-dangling-handle,
  bugprone-fold-init-type,
  bugprone-inaccurate-erase,
  bugprone-incorrect-roundings,
  bugprone-integer-division,
  bugprone-misplaced-operator-in-strlen-in-alloc,
  bugprone-misplaced-pointer-arithmetic-in-alloc,
  bugprone-move-forwarding-reference,
  bugprone-multiple-statement-macro,
  bugprone-parent-virtual-call,
  bugprone-posix-return,
  bugprone-reserved-identifier,
  bugprone-signed-char-misuse,
  bugprone-sizeof-container,
  bugprone-sizeof-expression,
  bugprone-string-constructor,
  bugprone-string-integer-assignment,
  bugprone-string-literal-with-embedded-nul,
  bugprone-suspicious-enum-usage,
  bugprone-suspicious-include,
  bugprone-suspicious-memset-usage,
  bugprone-suspicious-missing-comma,
  bugprone-suspicious-string-compare,
  bugprone-swapped-arguments,
  bugprone-terminating-continue,
  bugprone-throw-keyword-missing,
  bugprone-too-small-loop-variable,
  bugprone-undefined-memory-manipulation,
  bugprone-unhandled-self-assignment,
  bugprone-unused-raii,
  bugprone-unused-return-value,
  bugprone-use-after-move,
  bugprone-virtual-near-miss,

  hicpp-exception-baseclass,
  hicpp-ignored-remove-result,
  hicpp-no-assembler,
  hicpp-signed-bitwise,

  performance-faster-string-find,
  performance-for-range-copy,
  performance-implicit-conversion-in-loop,
  performance-inefficient-algorithm,
  performance-inefficient-vector-operation,
  performance-move-constructor-init,
  performance-no-automatic-move,
  performance-trivially-destructible,
  performance-unnecessary-copy-initialization,
  
  boost-use-to-string,
  boost-use-ranges,

  readability-identifier-naming,
'

WarningsAsErrors: '*'

CheckOptions:
  - key: readability-identifier-naming.AbstractClassCase
    value: CamelCase
  - key: readability-identifier-naming.AbstractClassPrefix
    value: ''
  - key: readability-identifier-naming.AbstractClassSuffix
    value: Base
  - key: readability-identifier-naming.ClassCase
    value: CamelCase
  - key: readability-identifier-naming.ClassPrefix
    value: ''
  - key: readability-identifier-naming.ClassSuffix
    value: ''
  - key: readability-identifier-naming.ClassConstantCase
    value: lower_case
  - key: readability-identifier-naming.ClassConstantPrefix
    value: ''
  - key: readability-identifier-naming.ClassConstantSuffix
    value: ''
  - key: readability-identifier-naming.ClassMemberCase
    value: lower_case
  - key: readability-identifier-naming.ClassMemberPrefix
    value: ''
  - key: readability-identifier-naming.ClassMemberSuffix
    value: ''
  - key: readability-identifier-naming.ClassMethodCase
    value: lower_case
  - key: readability-identifier-naming.ClassMethodPrefix
    value: ''
  - key: readability-identifier-naming.ClassMethodSuffix
    value: ''
  - key: readability-identifier-naming.ConceptCase
    value: lower_case
  - key: readability-identifier-naming.ConceptPrefix
    value: ''
  - key: readability-identifier-naming.ConceptSuffix
    value: ''
  - key: readability-identifier-naming.ConstantCase
    value: lower_case
  - key: readability-identifier-naming.ConstantPrefix
    value: ''
  - key: readability-identifier-naming.ConstantSuffix
    value: ''
  - key: readability-identifier-naming.ConstantMemberCase
    value: lower_case
  - key: readability-identifier-naming.ConstantMemberPrefix
    value: ''
  - key: readability-identifier-naming.ConstantMemberSuffix
    value: ''
  - key: readability-identifier-naming.ConstantParameterCase
    value: lower_case
  - key: readability-identifier-naming.ConstantParameterPrefix
    value: ''
  - key: readability-identifier-naming.ConstantParameterSuffix
    value: ''
  - key: readability-identifier-naming.ConstantPointerParameterCase
    value: lower_case
  - key: readability-identifier-naming.ConstantPointerParameterPrefix
    value: ''
  - key: readability-identifier-naming.ConstantPointerParameterSuffix
    value: ''
  - key: readability-identifier-naming.ConstexprFunctionCase
    value: lower_case
  - key: readability-identifier-naming.ConstexprFunctionPrefix
    value: ''
  - key: readability-identifier-naming.ConstexprFunctionSuffix
    value: ''
  - key: readability-identifier-naming.ConstexprMethodCase
    value: lower_case
  - key: readability-identifier-naming.ConstexprMethodPrefix
    value: ''
  - key: readability-identifier-naming.ConstexprMethodSuffix
    value: ''
  - key: readability-identifier-naming.ConstexprVariableCase
    value: lower_case
  - key: readability-identifier-naming.ConstexprVariablePrefix
    value: ''
  - key: readability-identifier-naming.ConstexprVariableSuffix
    value: ''
  - key: readability-identifier-naming.EnumCase
    value: CamelCase
  - key: readability-identifier-naming.EnumPrefix
    value: ''
  - key: readability-identifier-naming.EnumSuffix
    value: ''
  - key: readability-identifier-naming.EnumConstantCase
    value: UPPER_CASE
  - key: readability-identifier-naming.EnumConstantPrefix
    value: ''
  - key: readability-identifier-naming.EnumConstantSuffix
    value: ''
  - key: readability-identifier-naming.FunctionCase
    value: lower_case
  - key: readability-identifier-naming.FunctionPrefix
    value: ''
  - key: readability-identifier-naming.FunctionSuffix
    value: ''
  - key: readability-identifier-naming.GlobalConstantCase
    value: lower_case
  - key: readability-identifier-naming.GlobalConstantPrefix
    value: ''
  - key: readability-identifier-naming.GlobalConstantSuffix
    value: ''
  - key: readability-identifier-naming.GlobalConstantPointerCase
    value: lower_case
  - key: readability-identifier-naming.GlobalConstantPointerPrefix
    value: ''
  - key: readability-identifier-naming.GlobalConstantPointerSuffix
    value: ''
  - key: readability-identifier-naming.GlobalFunctionCase
    value: lower_case
  - key: readability-identifier-naming.GlobalFunctionPrefix
    value: ''
  - key: readability-identifier-naming.GlobalFunctionSuffix
    value: ''
  - key: readability-identifier-naming.GlobalPointerCase
    value: lower_case
  - key: readability-identifier-naming.GlobalPointerPrefix
    value: ''
  - key: readability-identifier-naming.GlobalPointerSuffix
    value: ''
  - key: readability-identifier-naming.GlobalVariableCase
    value: lower_case
  - key: readability-identifier-naming.GlobalVariablePrefix
    value: ''
  - key: readability-identifier-naming.GlobalVariableSuffix
    value: ''
  - key: readability-identifier-naming.InlineNamespaceCase
    value: lower_case
  - key: readability-identifier-naming.InlineNamespacePrefix
    value: ''
  - key: readability-identifier-naming.InlineNamespaceSuffix
    value: ''
  - key: readability-identifier-naming.LocalConstantCase
    value: lower_case
  - key: readability-identifier-naming.LocalConstantPrefix
    value: ''
  - key: readability-identifier-naming.LocalConstantSuffix
    value: ''
  - key: readability-identifier-naming.LocalConstantPointerCase
    value: lower_case
  - key: readability-identifier-naming.LocalConstantPointerPrefix
    value: ''
  - key: readability-identifier-naming.LocalConstantPointerSuffix
    value: ''
  - key: readability-identifier-naming.LocalPointerCase
    value: lower_case
  - key: readability-identifier-naming.LocalPointerPrefix
    value: ''
  - key: readability-identifier-naming.LocalPointerSuffix
    value: ''
  - key: readability-identifier-naming.LocalVariableCase
    value: lower_case
  - key: readability-identifier-naming.LocalVariablePrefix
    value: ''
  - key: readability-identifier-naming.LocalVariableSuffix
    value: ''
  - key: readability-identifier-naming.MacroDefinitionCase
    value: UPPER_CASE
  - key: readability-identifier-naming.MacroDefinitionPrefix
    value: ''
  - key: readability-identifier-naming.MacroDefinitionSuffix
    value: ''
  - key: readability-identifier-naming.MemberCase
    value: lower_case
  - key: readability-identifier-naming.MemberPrefix
    value: ''
  - key: readability-identifier-naming.MemberSuffix
    value: ''
  - key: readability-identifier-naming.MethodCase
    value: lower_case
  - key: readability-identifier-naming.MethodPrefix
    value: ''
  - key: readability-identifier-naming.MethodSuffix
    value: ''
  - key: readability-identifier-naming.NamespaceCase
    value: lower_case
  - key: readability-identifier-naming.NamespacePrefix
    value: ''
  - key: readability-identifier-naming.NamespaceSuffix
    value: ''
  - key: readability-identifier-naming.ParameterCase
    value: lower_case
  - key: readability-identifier-naming.ParameterPrefix
    value: ''
  - key: readability-identifier-naming.ParameterSuffix
    value: ''
  - key: readability-identifier-naming.ParameterPackCase
    value: lower_case
  - key: readability-identifier-naming.ParameterPackPrefix
    value: ''
  - key: readability-identifier-naming.ParameterPackSuffix
    value: ''
  - key: readability-identifier-naming.PointerParameterCase
    value: lower_case
  - key: readability-identifier-naming.PointerParameterPrefix
    value: ''
  - key: readability-identifier-naming.PointerParameterSuffix
    value: ''
  - key: readability-identifier-naming.PrivateMemberCase
    value: lower_case
  - key: readability-identifier-naming.PrivateMemberPrefix
    value: ''
  - key: readability-identifier-naming.PrivateMemberSuffix
    value: ''
  - key: readability-identifier-naming.PrivateMethodCase
    value: lower_case
  - key: readability-identifier-naming.PrivateMethodPrefix
    value: ''
  - key: readability-identifier-naming.PrivateMethodSuffix
    value: ''
  - key: readability-identifier-naming.ProtectedMemberCase
    value: lower_case
  - key: readability-identifier-naming.ProtectedMemberPrefix
    value: ''
  - key: readability-identifier-naming.ProtectedMemberSuffix
    value: ''
  - key: readability-identifier-naming.ProtectedMethodCase
    value: lower_case
  - key: readability-identifier-naming.ProtectedMethodPrefix
    value: ''
  - key: readability-identifier-naming.ProtectedMethodSuffix
    value: ''
  - key: readability-identifier-naming.PublicMemberCase
    value: lower_case
  - key: readability-identifier-naming.PublicMemberPrefix
    value: ''
  - key: readability-identifier-naming.PublicMemberSuffix
    value: ''
  - key: readability-identifier-naming.PublicMethodCase
    value: lower_case
  - key: readability-identifier-naming.PublicMethodPrefix
    value: ''
  - key: readability-identifier-naming.PublicMethodSuffix
    value: ''
  - key: readability-identifier-naming.ScopedEnumConstantCase
    value: UPPER_CASE
  - key: readability-identifier-naming.ScopedEnumConstantPrefix
    value: ''
  - key: readability-identifier-naming.ScopedEnumConstantSuffix
    value: ''
  - key: readability-identifier-naming.StaticConstantCase
    value: lower_case
  - key: readability-identifier-naming.StaticConstantPrefix
    value: ''
  - key: readability-identifier-naming.StaticConstantSuffix
    value: ''
  - key: readability-identifier-naming.StaticVariableCase
    value: lower_case
  - key: readability-identifier-naming.StaticVariablePrefix
    value: ''
  - key: readability-identifier-naming.StaticVariableSuffix
    value: ''
  - key: readability-identifier-naming.StructCase
    value: CamelCase
  - key: readability-identifier-naming.StructPrefix
    value: ''
  - key: readability-identifier-naming.StructSuffix
    value: ''
  - key: readability-identifier-naming.TemplateParameterCase
    value: CamelCase
  - key: readability-identifier-naming.TemplateParameterPrefix
    value: ''
  - key: readability-identifier-naming.TemplateParameterSuffix
    value: ''
  - key: readability-identifier-naming.TemplateTemplateParameterCase
    value: CamelCase
  - key: readability-identifier-naming.TemplateTemplateParameterPrefix
    value: ''
  - key: readability-identifier-naming.TemplateTemplateParameterSuffix
    value: ''
  - key: readability-identifier-naming.TypeAliasCase
    value: lower_case
  - key: readability-identifier-naming.TypeAliasPrefix
    value: ''
  - key: readability-identifier-naming.TypeAliasSuffix
    value: ''
  - key: readability-identifier-naming.TypedefCase
    value: lower_case
  - key: readability-identifier-naming.TypedefPrefix
    value: ''
  - key: readability-identifier-naming.TypedefSuffix
    value: ''
  - key: readability-identifier-naming.TypeTemplateParameterCase
    value: lower_case
  - key: readability-identifier-naming.TypeTemplateParameterPrefix
    value: ''
  - key: readability-identifier-naming.TypeTemplateParameterSuffix
    value: ''
  - key: readability-identifier-naming.UnionCase
    value: lower_case
  - key: readability-identifier-naming.UnionPrefix
    value: ''
  - key: readability-identifier-naming.UnionSuffix
    value: ''
  - key: readability-identifier-naming.ValueTemplateParameterCase
    value: lower_case
  - key: readability-identifier-naming.ValueTemplateParameterPrefix
    value: ''
  - key: readability-identifier-naming.ValueTemplateParameterSuffix
    value: ''
  - key: readability-identifier-naming.VariableCase
    value: lower_case
  - key: readability-identifier-naming.VariablePrefix
    value: ''
  - key: readability-identifier-naming.VariableSuffix
    value: ''
  - key: readability-identifier-naming.VirtualMethodCase
    value: lower_case
  - key: readability-identifier-naming.VirtualMethodPrefix
    value: ''
  - key: readability-identifier-naming.VirtualMethodSuffix
    value: ''
  - key: readability-identifier-naming.AggressiveDependentMemberLookup
    value: false
  - key: readability-identifier-naming.CheckAnonFieldInParent
    value: true
  - key: readability-identifier-naming.GetConfigPerFile
    value: true
  - key: readability-identifier-naming.IgnoreMainLikeFunctions
    value: true
```
