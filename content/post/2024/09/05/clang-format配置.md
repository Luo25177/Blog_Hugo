---
title: "clang-format配置"
date: 2024-09-05 20:51:34
categories: "程序员的自我涵养"
tags: ["clang-format", "clang"]
summary: "clang-format 基础配置，用于自动格式化代码，设置自己喜欢的格式"
---

## 前言

### 简介

Clang-format 是一种代码格式化的工具，可以用来格式化各种代码，支持的语言如下

- C/C++
- Java
- JavaScript
- Objective-C
- Protobuf
- C#

Clang-format 的配置文件可以取名为 `.clang-format` 或者 `_clang-format` ，该文件被放在项目的根目录下，一般来说 Clang-format 会自己去寻找配置文件的

### 安装

**windows**

[Download LLVM releases](https://releases.llvm.org/)

直接从上述网站上找到一个对应的发行版安装即可

然后在命令行中输入如下指令来查看是否安装完成，一般来说安装完成之后需要重新启动一下

```bash
clang-format -version
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
clang-format-18 --version
```

### 原文地址

[Clang-Format Style Options — Clang 20.0.0git documentation (llvm.org)](https://clang.llvm.org/docs/ClangFormatStyleOptions.html)

## 原理

Clang-format 的底层原理是基于 Clang 编译器的前端技术，包括一下几个步骤

- 词法分析：Clang-format 使用 Clange 的词法分析器，将源代码分解成标记，这些标记是代码的基本构建块，如关键词，标识符，运算符和分隔符
- 语法分析：Clang-format 解析代码的语法结构，生成抽象语法树，抽象语法树是代码结构的树形表示形式，用于进一步分析和转换
- 格式化规则应用：Clang-format 根据用户定义的格式化规则，对 AST 进行遍历和调整。规则涵盖了代码缩进，空格，换行和代码块布局等各个方面
- 代码生成：Clang-format 将调整之后的 AST 转换回源代码文本，并且输出格式化之后的代码

## vscode 配置 Clang-format

需要下载插件 C/C++，但是需要注意的是，下载的插件的版本必须是 1.17.4 以及之前的版本（后面更新的新版本好像用不了）

然后在设置里找到 `format-enbale` 并且将其打开即可

然后可以在项目的根目录下添加 `.clang-format` 文件，里面就可以写配置了，然后在格式化时 C/C++ 插件会自动寻找 `.clang-format` 文件的，一般来说会在该项目的根目录中寻找

## 命令行语法

```c
clang-format [options] [filename]
```

常用的选项如下

- `-i` 直接修改文件而不是输出到标准输出
- `-style` 指定格式化风格，如下几种预定风格
    - [LLVM](https://llvm.org/docs/CodingStandards.html)
    - [Google](https://google.github.io/styleguide/cppguide.html)
    - [Chromium](https://chromium.googlesource.com/chromium/src/+/refs/heads/main/styleguide/styleguide.md)
    - [Mozilla](https://firefox-source-docs.mozilla.org/code-quality/coding-style/index.html)
    - [WebKit](https://www.webkit.org/coding/coding-style.html)
    - [Microsoft](https://docs.microsoft.com/en-us/visualstudio/ide/editorconfig-code-style-settings-reference)
    - [GNU](https://www.gnu.org/prep/standards/standards.html)
- `-assume-filename` 指定文件类型，以便应用相应的格式化规则

## 配置文件语法

在进行配置时，可以一次使用多个语言配置，如下

```yaml
---
Language: Cpp
DerivePointerAlignment: false
---
Language: JavaScript
ColumnLimit: 100
---
Language: Proto
DisableFormat: true
---
Language: CSharp
ColumnLimit: 100
...
```

可以在源代码中进行如下注释，表示这一段代码不需要被格式化

```c
// clang-format off
void unformat_code();
// clang-format on
```

### 编写规则

- 配置文件名必须为 `.clang-format`
- 该文件必须位于项目的根目录或者编译目录下
- 文件采用 `YAML` 语法

### 配置字段解释

- `Language` 语言，可以指定当前配置文件为那些文件配置的
    - Cpp
    - Java
    - JavaScript
    - ObjC
    - Proto
    - TableGen
    - TextPorto
    - CSharp
- `BaseOnStyle` 基础风格，指定基本的风格样式
    - LLVM
    - Google
    - Chromium
    - Mozilla
    - WebKit
    - Microsoft
    - GNU
- `AccessModifierOffset` 访问性修饰符偏移，这个偏移量是相对于原来已有的缩进而言的，需要在原来就有的偏移基础上进行的偏移
- `AlignAfterOpenBracket` 开括号之后的对齐
    - `Align` 开阔号之后对齐变量
    - `DontAlign` 不对齐，换行后使用缩进
    - `AlwaysBreak` 一行放不下时，总是在一个开括号之后换行
    - `BlockIndext` 一行放不下时，总是在一个开括号之后换行，并且在新行关闭尾括号，但是仅限于圆括号
- `AlignArrayOfStructures` 对齐结构体数组，当数组列数相等时，会对齐每行的文本
    - `Left` 向左对齐
    - `Right` 向右对齐
    - `None` 不对齐
- `AlignConsecutiveAssignments` 连续赋值对齐，连续几行的赋值语句中，等号对齐。其中有几个变量来控制
    - `bool Enabled` 是否使能对齐
    - `bool AcrossEmptyLines` 可以跨越空白行进行对齐
    - `bool AcrossComments` 可以跨越注释行进行对齐
    - `bool AlignCompound` 对于混合运算 `+=` 会对齐于 `=`
    - `bool PadOperators`  混合运算符填补对齐到所有左值的最右边界，仅限于连续赋值
- `AlignConsecutiveBitFields` 位段对齐，主要用于结构体位阈段的对齐。由如下几个变量控制
    - `Enabled` 使能对齐
    - `AcrossEmptyLines` 可以跨越空行进行对齐
    - `AcrossComments` 可以跨越注释进行对齐
- `AlignConsecutiveDeclarations` 连续对齐声明，可以对齐变量等连续声明的语句。需要注意的是，这个对齐尽量不要跨越空白行，容易把函数定义也对齐了。由如下几个变量控制
    - `Enabled` 使能对齐
    - `AcrossEmptyLines` 可以跨越空行进行对齐
    - `AcrossComments` 可以跨越注释进行对齐
- `AlignEscapedNewlines` 对齐分割语法行的斜杠
    - `DontAlign` 不对齐
    - `Left` 左对齐，尽可能向左对齐
    - `Right` 右对齐，对齐到列的最右边
- `AlignOperands` 竖直对齐表达式的操作数，竖直对齐二元或三元表达式的操作数
    - `DontAlign` 不对齐
    - `Align` 对齐
    - `AlignAfterOperators`  在操作符之后对齐，也就是是否对齐操作数
    - 外部属性 `BreakBeforeBinaryOperators` 在二元操作符之前打断，如果配置文件中设置了这个属性，操作符也会换行
- `AlignTrailingComments` 对齐尾部注释
    - `Kind`
        - `Leave` 不做格式化
        - `Always` 对齐尾部注释
        - `Never` 不对齐，但是进行缩进之类的操作
    - `OverEmptyLines` 跨越空白行的行数，超过这个行数的注释不会对齐
- `AllowAllArgumentsOnNextLine` 允许参数在下一行上，如果函数调用或者带括号的函数初始化列表不在一行中，允许所有参数放到下一行
- `AllowAllParametersOfDeclarationOnNextLine` 允许声明的参数在下一行上，函数声明无法放在一行中，允许将所有变量放在下一行上
- ****`AllowShortBlocksOnASingleLine` 允许短语法块在单行上
    - `Never` 从不合并到单行上
    - `Always` 只合并空的块到单行上
    - `Empty` 总是合并短块到单行上
- `AllowShortEnumsOnASingleLine` 允许短的枚举类到单行上
- `AllowShortFunctionsOnASingleLine` 允许短函数在单行上
    - `None` 从不合并
    - `InlineOnly` 仅合并被定义在类里面的短函数
    - `Empty` 只合并空函数
    - `inline` 合并在类里面的短函数和外部的空函数
    - `ALL` 合并所有合适的短函数在单行上
- `AllowShortIfStatementsOnASingleLine` 允许 `if` 块在单行上
    - `Never` 从不把 `if` 放在单行上
    - `WithoutElse` 没有 `else` 时放在单行上
    - `OnlyFirstIf` 只把第一个 `if` 语句放在单行上，后面的 `else if` 和 `else` 都不放在单行上
    - `AllIfsAndElse` 短语句放在单行上
- `AllowShortLambdasOnASingleLine` 允许短匿名函数 `lambda` 放在单行上
    - `None` 从不合并
    - `Empty` 只合并空的匿名函数
    - `Inline` 当匿名函数作为实参时，合并在单行上
    - `All` 合并所有短的匿名函数
- `AllowShortLoopsOnASingleLine` 允许合并短循环到单行上
- `AlwaysBreakAfterReturnType` 函数声明返回类型的换行风格
    - `None` 从不在返回类型之后自动换行
    - `All` 总是在返回类型之后自动换行
    - `TopLevel` 在顶级函数顶级返回类型处换行
    - `AllDefinitions` 在函数定义的返回类型处换行
    - `TopLevelDefinitions` 在顶级函数定义的返回类型处换行
- `AlwaysBreakBeforeMultilineStrings` 多行字符串换行，在多行字符串字面量之前换行
- `AlwaysBreakTemplateDeclarations` 模板声明换行
    - `No` 不会强制在模板声明处换行
    - `MultiLine` 仅在接下来的函数/类声明参数需要跨行时才会在模板处换行
    - `Yes` 总是在模板声明之后换行
- `AttributeMacros` 属性宏，这个对于语言扩展或静态分析器注释非常有用
- `BinPackArguments` 装箱变量
    - `false` 变量要么都在同一行，要么每个变量单独一行
    - `true` 会把变量合理打包放在一行上，更紧凑一点
- `BitFieldColonSpacing` 位阈冒号的空格风格
    - `Both` 在冒号每边都增加一个空格
    - `None` 不添加任何空格
    - `Before` 在冒号之前添加空格
    - `After` 在冒号之后添加空格
- `BraceWrapping` 大括号换行风格。必须将 `BreakBeforeBraces` 设置为 `Custom` 时才有效，否则被忽略。由以下几个参数来定义
    - `bool AfterCaseLabel` 在 `case` 之后的大括号换行
    - `bool AfterClass` 类定义之后换行
    - `AfterControlStatement` 控制语句之后换行
        - `Never` 从不换行
        - `MultiLine` 只在一个多行控制语句之后换行
        - `Always` 总是换行
    - `bool AfterEnum` 在枚举类之后换行
    - `bool AfterFunction` 在函数定义之后大括号换行
    - `bool AfterNamespace` 命名空间之后换行
    - `bool AfterStruct` 在结构体之后换行
    - `bool AfterUnion` 在联合定义之后换行
    - `bool AfterExternBlock` 在 `extern` 声明之后换行
    - `bool BeforeCatch` 在 `catch` 之前换行
    - `bool BeforeElse` 在 `else` 之前换行
    - `bool BeforeLambdaBody` 在匿名函数表达式块之前换行
    - `bool BeforeWhile` 在 `while` 之前换行
    - `bool IndentBraces` 对换行的大括号缩进
    - `bool SplitEmptyFunction` 如果为 `false` ，空函数体可以放在单行上。此选项仅在函数的左大括号已经被换行的情况下使用，即设置了 `AfterFunction` 大括号换行模式，并且函数不应该放在单行上
    - `bool SplitEmptyRecord` 如果为 `false` ，空的类，结构体和联合体的主体可以放在单行上。此选项仅在记录的开始大括号已经被换行的情况下使用，即设置了`AfterClass` （用于类）大括号换行模式
    - `bool SplitEmptyNamespace` 如果为 `false` ，空的 `namespace` 主体可以放在单行上。此选项仅在命名空间的开始大括号已经被换行的情况下使用，即设置了 `AfterNamespace` 大括号换行模式
    - `bool AfterObjCDeclaration` 在 ObjC 定义之后换行
- `BreakAfterJavaFieldAnnotations` 在修饰器之后换行，适用于 Java
- `BreakArrays` 仅用于支持 Json 数组换行
- `BreakBeforeBinaryOperators` 二元操作符换行
    - `None` 在操作符之后换行
    - `NonAssignment` 只在非赋值操作之前换行
    - `All` 在操作符之前换行
- `BreakBeforeBraces` 大括号换行格式
    - `Attach` 总是将大括号附加到周围上下文
    - `Linux` 与上述类似，但是会在函数，命名空间和类定义处换行
    - `Mozilla` 与 `Attach` 类似，但是会在枚举和函数定义处换行
    - `Stroustrup` 与 `Attach` 类似，但是会在函数定义， `catch` ， `else` 处换行
    - `Allman`总是会在大括号处换行
    - `Whitesmiths` 与 `Allman` 类似，但是始终要缩进大括号并使用大括号排列代码
    - `GNU` 总是在大括号之前换行，并在控制语句的大括号中增加额外的缩进级别，而在类、函数或其他定义的大括号中不会缩进大括号
    - `WebKit` 与 `Attach` 类似，但是会在函数定义处换行
    - `Custom` 客制化配置每个大括号情况
- `BreakBeforeConceptDeclarations` 概念声明换行风格
    - `Never` 保持模板声明行和 `concept` 在一行
    - `Allowed` 允许模板声明和 `concept` 之间换行，实际的表现取决于上下文和换行规则
    - `Always` 永远在 `concept` 之前换行，并且将改行放在模板声明之前
- `BreakBeforeTernaryOperators` 三元操作符换行规则
    - `true` 在三元操作符之前换行
    - `false` 在三元操作符之后换行
- `BreakConstructorInitializers` 构造函数换行风格
    - `BeforeColon` 冒号前和逗号之后换行
    - `BeforeComma` 逗号和冒号之前换行，并且对齐
    - `AfterColon` 在冒号和逗号之后换行
    - `AfterComma`仅在逗号之后进行换行
- `BreakInheritanceList` 继承换行风格
    - `BeforeColon` 冒号前和逗号之后换行
    - `BeforeComma` 逗号和冒号之前换行，并且对齐
    - `AfterColon` 在冒号和逗号之后换行
    - `AfterComma`仅在逗号之后进行换行
- `BreakStringLiterals` 字符串常量换行，允许对字符串字面量进行断行
- `ColumnLimit` 列数限制，设置为 0 表示没有限制，不会在一行太长的语句中换行
- `CommentPragmas` 注释表示。值为字符串，含有正则表达式，其描述具有特殊含义的注释，这些注释不应被分割成行或以其他方式更改
- `CompactNamespaces` 紧凑命名空间
    - `true` 将连续的命名空间声明在一行中
    - `false` 每个命名空间在新行中声明
- `ConstructorInitializerIndentWidth` 构造初始化缩进宽度，用于构造函数初始化列表和继承列表缩进的字符数，无符号整数
- `ContinuationIndentWidth` 延续下一行缩进宽度，原来一行放不下时，换行后，新行缩进的字符数
- `Cpp11BracedListStyle` 大括号列表风格，其中大括号列表中没有空格
    - `false` 就是大括号内有空格与变量分隔
- `DeriveLineEnding` 提取行结尾
- `DerivePointerAlignment` 提取指针对齐
    - `true` 分析格式化文件中的 `&` 和 `*` 最常见的对齐方式，然后指针和引用对齐将根据在文件中找到的对齐样式进行更新
- `DisableFormat` 禁用格式化
- `EmptyLineAfterAccessModifier` 定义何时在访问修饰符之后放置空行
    - `Never` 移除访问修饰符之后所有的空行
    - `Leave` 在访问修饰符之后保持现有的空行
    - `Always` 如果访问修饰符之后没有空行，总是在后面添加空行
- `EmptyLineBeforeAccessModifier` 访问修饰符前空行
    - `Never` 移除访问修饰符之前所有的空行
    - `Leave` 在访问修饰符之前保持现有的空行
    - `Always` 如果访问修饰符之前没有空行，总是在后面添加空行，除非访问修饰符位于结构或类定义的开头
    - `LogicalBlock` 有当访问修饰符开始一个新的逻辑块时才添加空行。逻辑块是由一个或多个成员字段或函数组成的一组
- `ExperimentalAutoDetectBinPacking` 实验性的功能，不要在配置文件中使用，使用风险自负
- `FixNamespaceComments` 修复命名空间描述
    - `true` Clang-format 将为短命名空间添加丢失的命名空间结束注释，并且修复无效的现有注释
- `ForEachMacros` 迭代循环宏，被解释为 `foreach` 循环而不是函数调用的宏向量
- `IfMacros` 条件判断宏，被解释为条件语句而不是函数调用的宏向量
- `IncludeBlocks` `include` 块风格
    - `Preserve` 每个 `include` 块单独排序
    - `Merge` 合并多个 `include` 块，并且整体排序
    - `Regroup` 合并多个 `include` 块，并且整体排序，根据类别优先级分组，可参考 `IncludeCategories`
- `IncludeCategories` `include` 种类，正则表达式表示不同的 `include` 类别，然后进行排序。支持 POSIX 扩展正则表达式，每个分类都包含如下四个参数，下面是一个示例
    - `Regex` 正则表达式
    - `Priority` 优先级
    - `SortPriority` 默认优先级
    - `CaseSensitive` 是否区分大小写
    
    ```yaml
    IncludeCategories:
      - Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
        Priority:        2
        SortPriority:    2
        CaseSensitive:   true # 字段标记为区分大小写，默认情况下不区分大小写
      - Regex:           '^((<|")(gtest|gmock|isl|json)/)'
        Priority:        3
      - Regex:           '<[[:alnum:].]+>'
        Priority:        4
      - Regex:           '.*'
        Priority:        1
        SortPriority:    0
    ```
    
- `IncludeIsMainRegex` 指定一个正则表达式，其表明了文件到主包含映射中被允许的后缀名
- `IncludeIsMainSourceRegex` 判断源文件的正则表达式。为被格式化的文件指定正则表达式，匹配正则表达式的这些文件允许在文件到 `main-include` 映射中被视为 `main`
- `IndentAccessModifiers` 访问修饰符缩进，用于指定访问修饰符是否应该有自己的缩进级别
    - `false` 访问修饰符相对于成员缩进（或向外缩进），根据 `AccessModifierOffset` 。记录成员的缩进位置比记录低一级
    - `true` 访问修饰符获得自己的缩进级别，而且成员的缩进是在此基础上的
- `IndentCaseBlocks` `case` 块的缩进
    - `false` `case` 标签之后的块与 `case` 有着相同的缩进级别
    - `true` `case` 标签之后的缩进为作用域块，相对于 `case` 还有一级缩进
- `IndentCaseLabels` `case label` 标签缩进
    - `false` `case` 标签与 `switch` 标签有着相同的缩进等级
    - `true` `case` 标签在 `switch` 基础上有一级缩进
- `IndentExternBlock` `Extern` 扩展块缩进
    - `NoIndent` 不缩进
    - `Indent` 缩进
- `IndentGotoLabels` 缩进 `goto` 跳转符号
- `IndentPPDirectives` 预处理指令缩进
    - `None` 不缩进任何预处理指令
    - `AfterHash` 缩进 `#` 之后的内容
    - `BeforeHash` 缩进时包括 `#`
- `IndentRequiresClause` 是否缩进模板中的 `require` 子句。这只适用于 `RequiresClausePosition` 为 `OwnLine` 或 `WithFollowing` 时
- `IndentWidth` 缩进宽度，用于缩进的列数
- `IndentWrappedFunctionNames` 如果函数定义或声明在类型之后换行，则函数名缩进
- `InsertBraces` 插入括号。在控制语句之后插入大括号，除非控制语句位于宏定义中，或者大括号包含预处理指令。但是需要注意的是，设置之后有可能会导致错误的代码格式，需要小心使用
- `InsertTrailingCommas` 插入尾部冒号
    - `Wrapped` 会在跨多行换行的容器字面量（数组和对象）中插入尾随逗号。但是使用之后将禁用 `BinPackArguments`
    - `None` 不插入
- `JavaImportGroups` 为 Java 导入按照所需组排序的前缀向量
- `JavaScriptQuotes` Java 的引号风格
    - `Leave` 保持原样
    - `Single` 总是使用单引号
    - `Double` 总是使用双引号
- `JavaScriptWrapImports` Java 中 `import/export` 语句换行
- `KeepEmptyLinesAtTheStartOfBlocks` 在一个语法块的开始刘空行
- `LambdaBodyIndentation` 匿名函数表达式主体缩进样式
    - `Signature` 将匿名函数主体相对于 该匿名函数签名对齐
    - `OuterScope` 相对于匿名函数签名所在的外部作用域的缩进级别对齐匿名函数主体
- `MacroBlockBegin` 开始块的宏，匹配启动块的宏的正则表达式
- `MacroBlockEnd` 结束块的宏
- `MaxEmptyLinesToKeep` 最大持续空行，就是保留的最大空行数
- `NamespaceIndentation` 命名空间内部是否缩进
    - `None` 不缩进
    - `Inner` 只缩进嵌套的命名空间（最外层命名空间内部不缩进）
    - `All` 所有命名空间都缩进
- `NamespaceMacro` 命名空间宏，用于打开名称空间块的宏向量
- `ObjCBinPackProtocolList` ObjC 的打包风格
    - `Never/Auto(BinPackParameters=false)` 则在超出 `ColumnLimit` 时将 `Objective-C` 协议一致性列表项布局到单独的行上
    - `Always/Auto(BinPackParameters=true)` 当代码长度超过 `ColumnLimit` 时，就会将 `Objective-C` 协议一致性列表项打包成尽可能少的行
- `ObjCBlockIndentWidth` 用于 ObjC 块缩进的字符数
- `ObjCBreakBeforeNestedBlockParam` 当函数调用中有嵌套的块参数时，将参数列表分解成行
- `ObjCSpaceAfterProperty` 在 ObjC 的属性之后添加一个空格
- `ObjCSpaceBeforeProtocolList` 在 ObjC 协议列表前面添加一个空格
- `PPIndentWidth` 用于预处理器语句缩进的列数
- `PackConstructorInitializers` 打包构造函数初始化列表
    - `Never` 总是将每个构造函数初始化列表放在一个单独的行中
    - `BinPack` 打包构造函数初始化列表
    - `CurrentLine` 如果合适，将所有构造函数初始化式放在当前行中，否则将每一个放在单独的一行上
    - `NextLine` 与上述相似，如果所有的构造函数初始化式都不适合当前行，则尝试将它们适合下一行
- `PenaltyXxxx` 各类情况惩罚
- `PointerAlignment` 指针对齐风格
    - `Left` 向左对齐
    - `Right` 向右对齐
    - `Middle` 中间对齐
- `QualifierAlignment` 限定符对齐，不同的排列说明符和限定符的方法，主要是针对 `const` 和 `volatile`
    - `Leave` 保持原状，不强制
    - `Left` 左对齐说明符或者限定符
    - `Right` 右对齐说明符或者限定符
    - `Custom` 将说明符/限定符更改为基于 `QualifierOrder` 对齐
- `QualifierOrder` 说明符和限定符的顺序，限定符出现的顺序
    - 包括 `const, inline, static, constexpr, volatile, restrict, type`
    - 必须包含 `type` 。在 `type` 左边的项目将被放置在类型的左边，并按提供的顺序排列。 `type` 右侧的项目将被放置在类型的右侧，并按提供的顺序排列
    - 例子 `QualifierOrder: ['inline', 'static', 'const', 'volatile', 'type']`
- `RawStringFormats` 原始字符串格式
- `ReferenceAlignment` 引用格式对齐
    - `Left` 向左对齐
    - `Right` 向右对齐
    - `Middle` 中间对齐
- `ReflowComments` 是否重新排版注释
- `RemoveBracesLLVM` 移除括号，用来移除部分可以去掉的括号(大括号)，即 `if/else/for/while` 等语句。通过 LLVM 风格实现。可能导致未知错误，小心使用
- `RemoveSemicolon` 移除分号，移除非空函数的分号。有可能带来未知风险
- `RequiresClausePosition` `require` 子句位置
    - `OwnLine` 总是把要求子句放在单独一行上
    - `WithPreceding` 试着把从句和声明的前面部分放在一起
    - `WithFollowing` 尝试把 `requires` 子句和函数声明放在一起
    - `SingleLine` 如果可能的话，尽量把所有内容放在同一行。否则正常的断行规则就会被取代
- `RequiresExpressionIndentation` `require` 表达式缩进
    - `OuterScope` 相对于 `require` 表达式所在的外部范围的缩进级别
    - `Keyword` 相对于 `require` 关键字的缩进
- `SeparateDefinitionBlocks` 分离定义语句块，指定使用空行分隔定义块，包括类、结构、枚举和函数
    - `Leave` 保持原样
    - `Always` 在定义块之间插入空行
    - `Never` 在定义块之间移除空行
- `ShortNamespaceLines` 短命名空间行数，这通过计算未换行行(即既不包含开始名称空间也不包含结束名称空间大括号)来确定短名称空间的最大长度，并使 `FixNamespaceComments` 忽略为这些名称空间添加结束注释
- `SortIncludes` 对 `include` 排序
    - `Never` 不进行排序
    - `CaseSensitive` 区分大小写，按照 ascii 排序
    - `CaseInsensitive` 不区分大小写，按照字母顺序排序
- `SortJavaStaticImport` 排序 Java 静态导入
    - `Before` 静态导入被放在非静态导入前面
    - `After` 静态导入被放在非静态导入前面
- `SortUsingDeclarations` 对 `using` 声明进行排序，按照字母顺序排序
- `SpaceAfterCStyleCast` C 风格类型转换，是否在 C 风格的类型转换之后插入空白符
- `SpaceAfterLogicalNot` 是否在逻辑非操作符之后插入空格
- `SpaceAfterTemplateKeyword` 是否在模板关键字之后插入空格
- `SpaceAroundPointerQualifiers` 定义在什么情况下在指针限定符之前或之后放置空格
    - `Default` 不用确保指针限定符周围的空格，而是使用 `PointerAlignment` 代替
    - `Before` 确保在指针限定符之前有一个空格
    - `After` 确保在指针限定符之后有一个空格
    - `Both` 确保在指针限定符之后和之前都有一个空格
- `SpaceBeforeAssignmentOperators` 赋值操作符控空格，赋值操作符之前是否插入空格
- `SpaceBeforeCaseColon` `case` 之后的冒号前是否插入空格
- `SpaceBeforeCpp11BracedList` 大括号列表空格，是否在大括号列表之前插入空格
- `SpaceBeforeCtorInitializerColon` 构造函数初始化中的冒号前是否插入空格
- `SpaceBeforeInheritanceColon` 继承冒号前是否插入空格
- `SpaceBeforeParens` 圆括号空格，圆括号前是否插入空格
    - `Never` 从不在一个开圆括号之前放一个空格
    - `ControlStatement` 只在控制语句关键字之后的开括号前加空格
    - `ControlStatementsExceptControlMacros`  与上一个类似，但是排除宏定义
    - `NonEmptyParentheses` 只有当圆括号非空时
    - `Always` 总是在开括号之前放一个空格
    - `Custom` 自定义
- `SpaceBeforeParensOptions` 圆括号前空格控制，控制括号前的个别空格，必须在 `SpaceBeforeParens` 设置为 `Custom` 时才有效
    - `bool AfterControlStatements` 在控制语句关键字和圆括号之间插入空格
    - `bool AfterForeachMacros` 在 `foreach` 宏和圆括号之间插入空格
    - `bool AfterFunctionDeclarationName` 在函数声明和开圆括号之前放置空格
    - `bool AfterFunctionDefinitionName` 在函数定义名称和开圆括号之间用空格隔开
    - `bool AfterIfMacros` 在 `if` 宏和开圆括号之间放空格
    - `bool AfterOverloadedOperator` 在操作符重载和开括号之间放一个空格
    - `bool AfterRequiresInClause` 在 `required` 子句中的 `required` 关键字和开括号（如果有的话）之间放空格
    - `bool AfterRequiresInExpression` 在 `required` 表达式中的 `required` 关键字和开括号之间放空格
    - `bool BeforeNonEmptyParentheses` 只有当圆括号不是空的时候，才在圆括号前放空格
- `SpaceBeforeRangeBasedForLoopColon` 是否在循环范围里的冒号前插入空格
- `SpaceBeforeSquareBrackets` 方括号前插入空格，匿名函数不受影响，只有第一个方括号才会收到影响（如果有多个的话）
- `SpaceInEmptyBlock` 空块中是否插入空格
- `SpaceInEmptyParentheses` 空圆括号之间是否插入空格
- `SpacesBeforeTrailingComments` 尾部注释前的空格数，不影响块注释
- `SpacesInAngles` 角括号是否插入空格
    - `Never` 不插入
    - `Always` 在角括号之间插入空格
    - `Leave` 如果有空格，则在 `<` 之后和 `>` 之前保留且只保留一个空格
- `SpacesInCStyleCastParentheses` 是否向 C 类型转换中插入空格
- `SpacesInConditionalStatement` 条件表达式中的空格
- `SpacesInContainerLiterals` 容器中的空格，在容器中插入空格（方括号和花括号中初始化数组之类的）
- `SpacesInLineCommentPrefix` 行注释的开头允许有多少空格，要禁用最大值，请将其设置为 `-1` ，除此之外，最大值优先于最小值。在行注释区中，后面的行保持相对缩进。有两个参数
    - `Minimum` 最小值
    - `Maximum` 最大值
- `SpacesInParentheses` 在圆括号里的空格
- `SpacesInSquareBrackets` 方括号中的空格
- `Standard` C++ 标准，可以为
    - `c++03` 解析格式化为 `c++03`
    - `c++11`
    - `c++14`
    - `c++17`
    - `c++20`
    - `Latest` 解析和格式化为最新的支持版本
    - `Auto` 根据输入和输出自动检测语言
- `StatementAttributeLikeMacros` 在语句前被忽略的宏，就像它们是一个属性一样。这样它们就不会被解析为标识符
- `StatementMacros` 完整语句的宏向量
- `TabWidth` Tab 的宽度
- `TypenameMacros` 类型名宏定义，类型声明而不是函数调用的宏向量
- `UseCRLF` 换行时使用 `\r\n` 而不是 `\n` 。如果 `DeriveLineEnding` 为真，也可用作回退
- `UseTab` 在文件中使用制表符的情况
    - `Never` 从不使用
    - `ForIndentation` 仅用于缩进
    - `ForContinuationAndIndentation` 用制表符填充所有前导空白，并使用空格对齐出现在一行内
    - `AlignWithSpaces` 使用制表符进行行继续和缩进，使用空格进行对齐
    - `Always` 每当需要填充至少跨越一个制表位到下一个制表位的空白时，就使用制表符
- `WhitespaceSensitiveMacros` 对空白敏感且不应被触及的宏向量

### 一个可供参考的版本

```yaml
---
Language: Cpp
DisableFormat: false
BaseOnStyle: LLVM
AccessAccessModifierOffset: -2
AlignAfterOpenBracket: Align
AlignArrayOfStructures: Right
AlignConsecutiveAssignments:
  Enabled: false
  AcrossEmptyLines: true
  AcrossComments: true
  AlignCompound: true
  PadOperators: true
AlignConsecutiveBitFields:
  Enabled: true
  AcrossEmptyLines: true
  AcrossComments: true
AlignConsecutiveDeclarations:
  Enabled: true
  AcrossEmptyLines: false
  AcrossComments: true
AlignEscapedNewlines: Left
AlignOperands: DontAlign
AlignTrailingComments:
  Kind: Always
  OverEmptyLines: 0
AllowAllArgumentsOnNextLine: false
AllowAllParametersOfDeclarationOnNextLine: false
AllowShortBlocksOnASingleLine: Never
AllowShortEnumsOnASingleLine: false
AllowShortFunctionsOnASingleLine: InlineOnly
AllowShortIfStatementsOnASingleLine: AllIfsAndElse
AllowShortLambdasOnASingleLine: All
AllowShortLoopsOnASingleLine: true
AlwaysBreakAfterReturnType: None
AlwaysBreakBeforeMultilineStrings: false
AlwaysBreakTemplateDeclarations: Yes
BinPackArguments: true
BitFieldColonSpacing: Both
BreakBeforeBraces: Custom
BraceWrapping:
  AfterCaseLabel: false
  AfterClass: false
  AfterControlStatement: Never
  AfterEnum: false
  AfterFunction: false
  AfterNamespace: false
  AfterStruct: false
  AfterUnion: false
  AfterExternBlock: false
  BeforeCatch: false
  BeforeElse: false
  BeforeLambdaBody: false
  BeforeWhile: false
  IndentBraces: false
  SplitEmptyFunction: false
  SplitEmptyRecord: false
  SplitEmptyNamespace: false
  AfterObjCDeclaration: false
BreakAfterJavaFieldAnnotations: true
BreakArrays: false
BreakBeforeBinaryOperators: None
BreakBeforeConceptDeclarations: Never
BreakBeforeTernaryOperators: false
BreakConstructorInitializers: AfterColon
BreakInheritanceList: AfterComma
BreakStringLiterals: false
ColumnLimit: 100
CompactNamespaces: false
ConstructorInitializerIndentWidth: 0
ContinuationIndentWidth: 0
Cpp11BracedListStyle: false
DerivePointerAlignment: false
DeriveLineEnding: false
EmptyLineAfterAccessModifier: Never
EmptyLineBeforeAccessModifier: Always
FixNamespaceComments: true
ForEachMacros: ['RANGES_FOR', 'FOREACH']
IfMacros: ['IF']
IncludeBlocks: Regroup
IncludeCategories:
  - Regex:           '^"(llvm|llvm-c|clang|clang-c)/'
    Priority:        2
    SortPriority:    2
    CaseSensitive:   true # 字段标记为区分大小写，默认情况下不区分大小写
  - Regex:           '^((<|")(gtest|gmock|isl|json)/)'
    Priority:        3
  - Regex:           '<[[:alnum:].]+>'
    Priority:        4
  - Regex:           '.*'
    Priority:        1
    SortPriority:    0
IndentAccessModifiers: false
IndentCaseBlocks: false
IndentCaseLabels: true
IndentExternBlock: NoIndent
IndentGotoLabels: false
IndentPPDirectives: None
IndentRequiresClause: true
IndentWidth: 2
IndentWrappedFunctionNames: true
InsertBraces: false
InsertTrailingCommas: None
JavaScriptQuotes: Double
JavaScriptWrapImports: true
KeepEmptyLinesAtTheStartOfBlocks: false
LambdaBodyIndentation: Signature
MaxEmptyLinesToKeep: 1
NamespaceIndentation: All
ObjCBinPackProtocolList: Never
ObjCBlockIndentWidth: 2
ObjCBreakBeforeNestedBlockParam: false
ObjCSpaceAfterProperty: false
PPIndentWidth: 0
PackConstructorInitializers: NextLine
PointerAlignment: Left
QualifierAlignment: Left
ReferenceAlignment: Right
ReflowComments: true
RequiresClausePosition: OwnLine
RequiresExpressionIndentation: Keyword
SeparateDefinitionBlocks: Always
ShortNamespaceLines: 0
SortIncludes: CaseSensitive
SortJavaStaticImport: Before
SortUsingDeclarations: true
SpaceAfterCStyleCast: true
SpaceAfterLogicalNot: false
SpaceAfterTemplateKeyword: false
SpaceAroundPointerQualifiers: Default
SpaceBeforeAssignmentOperators: true
SpaceBeforeCaseColon: false
SpaceBeforeCpp11BracedList: true
SpaceBeforeCtorInitializerColon: true
SpaceBeforeInheritanceColon: true
SpaceBeforeParens: Custom
SpaceBeforeParensOptions: 
  AfterControlStatements: true
  AfterForeachMacros: true
  AfterFunctionDeclarationName: false
  AfterFunctionDefinitionName: false
  AfterIfMacros: true
  AfterOverloadedOperator: true
  AfterRequiresInClause: true
  AfterRequiresInExpression: true
  BeforeNonEmptyParentheses: false
SpaceBeforeRangeBasedForLoopColon: true
SpaceBeforeSquareBrackets: false
SpaceInEmptyBlock: true
SpaceInEmptyParentheses: false
SpacesBeforeTrailingComments: 1
SpacesInAngles: Never
SpacesInCStyleCastParentheses: false
SpacesInConditionalStatement: false
SpacesInContainerLiterals: true
SpacesInLineCommentPrefix:
  Minimum: 1
  Maximum: 1
SpacesInParentheses: false
SpacesInSquareBrackets: false
Standard: Auto
TabWidth: 2
UseTab: Never
```
