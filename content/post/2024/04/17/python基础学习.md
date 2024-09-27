---
title: "python基础学习"
date: 2024-04-17 11:21:45
categories: "编程语言"
tags: ["python"]
summary: "python 基础语言学习，基础语法很简单，但是要学会用好很多第三方库才是最难的"
---

# 前言

Python 是一个高层次的结合了解释性、编译性、互动性和面向对象的脚本语言

- **Python 是解释型语言：** 这意味着开发过程中没有了编译这个环节。类似于 `PHP` 和 `Perl` 语言
- **Python 是交互式语言：** 这意味着可以在终端中的 `Python` 提示符 `>>>` 后直接执行代码
- **Python 是面向对象语言:** 这意味着 `Python` 支持面向对象的风格或代码封装在对象的编程技术
- **Python 是初学者的语言：** `Python` 对初级程序员而言，是一种伟大的语言，它支持广泛的应用程序开发，从简单的文字处理到 `WWW` 浏览器再到游戏

# 安装

## linux

1. 安装依赖环境

    ```bash
    yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
    ```

2. 下载python

    ```bash
    wget https://www.python.org/ftp/python/3.9.0/Python-3.9.0.tgz
    ```

3. 编译安装

    首先是创建安装目录 `/usr/local/python3.9`

    ```bash
    sudo mkdir -p /usr/local/python3.9
    ```

    解压安装包

    ```bash
    tar -zxvf Python-3.9.0.tgz
    ```

    进入解压后的目录，编译安装

    ```bash
    ./configure --prefix=/usr/local/python3.9
    ```

    其中 `--prefix` 就是指定安装目录，如果没有指定安装目录，会在 `/usr/local/bin` 或 `/usr/local/lib` 目录下看到

    编译

    ```bash
    make
    ```

    安装

    ```bash
    make install
    ```

    然后检查

    ```bash
    /usr/local/python3.9/bin/python3.9
    ```

    成功进入 `python` 的命令行就是安装完成了

4. 配置环境变量

    在 `/etc/profile` 文件的最底下输入

    ```bash
    export PYTHON_HOME=/usr/local/python3.9
    export PATH=${PYTHON_HOME}/bin:$PATH
    ```

    然后更新环境变量

    ```bash
    source /etc/profile
    ```

    如果是上边安装目录没有指定，就不需要配置环境变量。因为默认安装的目录 `/usr/local/bin` 是在环境变量中的

## windows

windows上安装会比较简单，一般来说可以直接下载 python 的安装包然后安装就好，或者可以下载 python 的源文件，然后将对应的 `bin` 目录添加导环境变量中就好

# 标准数据类型

- `Number` 数字，不可变，是一个常量
  - `int` 整形，为长整型
  - `float` 浮点数
  - `bool` 布尔类型
  - `complex` 复数类型 `1+2j`
- `String` 字符串，不可变
  - python 中的 `'` 与 `"` 使用方法一致
  - 使用三引号可以指定一个多行字符串
  - 转义字符 `\`
  - 反斜杠可用作转义，使用 `r` 可使之不发生转义
  - 字符串可以使用 `+` 连接在一起，使用 `*` 使得字符串重复
  - 索引方式从前到后，或者是从后到前
  - 字符串切片为 `str[start:end]` 表示切片范围 `[start, end)`
  - 字符串切片可以定义步长 `str[start:end:step]`
- `bool` 布尔类型
  - 只有 `True` 和 `False`
  - 布尔类型与其他类型比较时，会把 `True` 视为 1， `False` 视为 0
- `List` 列表
  - 列表中元素可变
  - 列表写在方括号内，元素之间逗号隔开 `l = [a, b, c]`
  - 和字符串类似，列表可被切片和索引
  - 列表可以使用 `+` 进行拼接
  - `l = []`
- `Tuple` 元组，是不可改变的列表
  - 元组中的元素不可改变
  - 元组可以索引，也可以截取
  - 元素写在小括号内，元素之间使用逗号隔开 `t = (a, b, c)`
  - 如果只有一个元素，需要在元素之后添加逗号 `t = (a, )`
  - 元组可以使用 `+` 操作符拼接
  - `t = tuple()`
- `Set` 集合
  - 是一种无序并且可变的数据类型，用于存储唯一的元素，类似于 C++ 中的哈希表
  - 元素不会重复，可以使用交集，并集，差集，异或等集合操作
  - 使用大括号表示，元素之间用逗号分割 `s = {a, b, c}`
  - 空集合为 `s = set()`
- `Dictionary` 字典
  - 用大括号表示，是无序的键和值的集合，类似于 C++ 的哈希map
  - 在字典中，键必须是唯一的
  - 使用构造函数 `dict()` 可以直接从键值对序列中构建字典
- `bytes`
  - 是不可变的二进制序列
  - 其中的元素类型是 `0~255` 之间的整数，不是 unicode 字符
  - 支持许多操作和方法，如切片、拼接、查找、替换
  - 是整数值，比较时需要使用相应的整数值

## 数据类型转换

- `int(x [, base])` 将 x 转为整数
- `float(x)` 将 x 转为浮点数
- `complex(real [, imag])` 创建一个复数，实数部分为 `real` ，虚数部分默认为 0
- `str(x)` 将 x 转为字符串
- `repr(x)` 将 x 转为字符串表达式
- `eval(str)` 用来计算在字符串中的有效 python 表达式，返回一个对象，这个对象名就是这个字符串的内容，需要提前定义
- `tuple(s)` 将序列 s 转为元组
- `list(s)` 将序列 s 转化为列表
- `set(s)` 转换为可变集合
- `dict(d)` 创建一个字典，其中 `d` 必须是 `(key, value)` 元组序列
- `frozenset(s)` 转为不可变的集合
- `chr(x)` 将一个整数转为字符
- `ord(x)` 将一个字符转为它的整数值
- `hex(x)` 将整数转为 16 进制
- `oct(x)` 将整数转为 8 进制
- 在隐式类型转换中，Python 会自动将一种数据类型转换为另一种数据类型，不需要我们去干预。两种不同类型的数据进行运算，较低数据类型就会转换为较高数据类型以避免数据丢失

## 运算符

### 逻辑运算符

- `and` 与 C 语言中的  `&&` 一致，先执行运算符之前的条件，如果是 `False` 直接返回，否则运算符之后的条件
- `or` 如果运算符之前为 `True` 直接返回，否则返回运算符之后的条件
- `not` 非

### 成员运算符

- `in` 在指定的序列中返回 `True`
- `not in` 不在指定的序列中返回 `False`

### 身份运算符

- `is` 判断两个标识符是否引用自同一个对象
- `not is` 两个标识符是否不是引自同一个对象

### 运算符优先级

| 运算符                                                 | 描述                               |
| ------------------------------------------------------ | ---------------------------------- |
| (expressions...),                                      |
| [expressions...], {key: value...}, {expressions...}    | 圆括号的表达式                     |
| x[index], x[index:index], x(arguments...), x.attribute | 读取，切片，调用，属性引用         |
| await x                                                | await 表达式                       |
| **                                                     | 乘方(指数)                         |
| +x, -x, ~x                                             | 正，负，按位非 NOT                 |
| *, @, /, //, %                                         | 乘，矩阵乘，除，整除，取余         |
| +, -                                                   | 加和减                             |
| <<, >>                                                 | 移位                               |
| &                                                      | 按位与 AND                         |
| ^                                                      | 按位异或 XOR                       |
|                                                        |                                    | 按位或 OR |
| in,not in, is,is not, <, <=, >, >=, !=, ==             | 比较运算，包括成员检测和标识号检测 |
| not x                                                  | 逻辑非 NOT                         |
| and                                                    | 逻辑与 AND                         |
| or                                                     | 逻辑或 OR                          |
| if -- else                                             | 条件表达式                         |
| lambda                                                 | lambda 表达式                      |
| :=                                                     | 赋值表达式                         |

## 列表

### 创建列表

元素在方括号内，并且元素之间用逗号分割

```bash
l = [1, 2, 3, 4, 5]
l = []
```

### 访问

类似于 C 语言中的数组，利用方括号访问

```python
print(l[0])
```

### 更新

直接对列表中元素赋值就可以完成更新

```python
l[0] = 2
# l = [2, 2, 3, 4, 5]
```

### 删除

使用 `del` 语句删除列表元素

```python
del l[0]
l = [2, 3, 4, 5]
```

### 组合

使用 `+` 号运算符直接组合两个列表

```python
l = l + l
# l = [2, 3, 4, 5, 2, 3, 4, 5]
```

### 嵌套列表

在列表中可以包含其他列表，也就是列表的列表

```python
ll = [l, l]
```

### 比较

需要引入 `operator` 模块中的 `eq` 方法

```python
import operator
l1 = [0, 1]
l2 = [1, 2]
operator.eq(l1, l2)
```

### 其他方法

- `append` 追加元素
- `count` 统计列表中某元素个数
- `extend` 在末尾一次性追加一个列表
- `index` 在列表中找出某个值第一个匹配项的索引位置
- `insert` 将对象插入列表中
- `pop` 移除列表中一个元素，默认为最后一个元素
- `remove` 移除列表中某个值的第一个匹配项
- `reverse` 反向列表
- `sort` 对列表中元素进行排序
- `clear` 清空列表
- `copy` 复制列表

### 测试

```python
l = [1, 2, 3, 4, 5]

print(l[0])

l[0] = 2
print(l)

del l[0]
print(l)

ll = l + l
print(ll)

l_l = [l, l]
print(l_l)

import operator
l1 = [0, 1]
l2 = [1, 2]
l3 = [1, 2]
print(operator.eq(l1, l2))
print(operator.eq(l2, l3))

l.append(1)
print(l)

print(l.count(1))

l.extend(l1)
print(l)

print(l.index(3))

l.insert(1, 3)
print(l)

l.pop()
print(l)

l.remove(3)
print(l)

l.reverse()
print(l)

l.sort()
print(l)

l1 = l.copy()
print(l1)

l.clear()
print(l)
```

## 字符串

### 内部函数

- `capitalize` 将字符串第一个字符转为大写
- `center` 返回一个指定宽度的字符串，其中间部分为调用函数的字符串，两边用第二个参数填充（字符）
- `count` 返回一个字符串出现的次数，能指定范围
- `bytes.decode(encoding="utf-8", errors="strict")` Python3 中没有 `decode` 方法，可以使用 `bytes` 对象的 `decode()` 方法来解码给定的 `bytes` 对象，这个 `bytes` 对象可以由 `str.encode()` 来编码返回。
- `encode(encoding='UTF-8',errors='strict')` 以 encoding 指定的编码格式编码字符串，如果出错默认报一个ValueError 的异常，除非 errors 指定的是'ignore'或者'replace'
- `endswith(suffix, beg=0, end=len(string))` 检查字符串是否以 suffix 结束，如果 beg 或者 end 指定则检查指定的范围内是否以 suffix 结束，如果是，返回 True,否则返回 False。
- `expandtabs(tabsize=8)` 把字符串 string 中的 tab 符号转为空格，tab 符号默认的空格数是 8
- `find(str, beg=0, end=len(string))` 检测 str 是否包含在字符串中，如果指定范围 beg 和 end ，则检查是否包含在指定范围内，如果包含返回开始的索引值，否则返回-1
- `index(str, beg=0, end=len(string))` 跟find()方法一样，只不过如果str不在字符串中会报一个异常。
- `isalnum()` 如果字符串至少有一个字符并且所有字符都是字母或数字则返 回 True，否则返回 False
- `isalpha()` 如果字符串至少有一个字符并且所有字符都是字母或中文字则返回 True, 否则返回 False
- `isdigit()` 如果字符串只包含数字则返回 True 否则返回 False..
- `islower()` 如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是小写，则返回 True，否则返回 False
- `isnumeric()` 如果字符串中只包含数字字符，则返回 True，否则返回 False
- `isspace()` 如果字符串中只包含空白，则返回 True，否则返回 False.
- `istitle()` 如果字符串是标题化的(见 title())则返回 True，否则返回 False
- `isupper()` 如果字符串中包含至少一个区分大小写的字符，并且所有这些(区分大小写的)字符都是大写，则返回 True，否则返回 False
- `join(seq)` 以指定字符串作为分隔符，将 seq 中所有的元素(的字符串表示)合并为一个新的字符串
- `len(string)` 返回字符串长度
- `ljust(width[, fillchar])` 返回一个原字符串左对齐,并使用 fillchar 填充至长度 width 的新字符串，fillchar 默认为空格。
- `lower()` 转换字符串中所有大写字符为小写.
- `lstrip()` 截掉字符串左边的空格或指定字符。
- `maketrans()` 创建字符映射的转换表，对于接受两个参数的最简单的调用方式，第一个参数是字符串，表示需要转换的字符，第二个参数也是字符串表示转换的目标。
- `max(str)` 返回字符串 str 中最大的字母。
- `min(str)` 返回字符串 str 中最小的字母。
- `replace(old, new [, max])` 把 将字符串中的 old 替换成 new,如果 max 指定，则替换不超过 max 次。
- `rfind(str, beg=0,end=len(string))` 类似于 find()函数，不过是从右边开始查找.
- `rindex( str, beg=0, end=len(string))` 类似于 index()，不过是从右边开始.
- `rjust(width,[, fillchar])` 返回一个原字符串右对齐,并使用 `fillchar(默认空格)` 填充至长度 width 的新字符串
- `rstrip()` 删除字符串末尾的空格或指定字符。
- `split(str="", num=string.count(str))` 以 str 为分隔符截取字符串，如果 num 有指定值，则仅截取 num+1 个子字符串
- `splitlines([keepends])` 按照行('\r', '\r\n', \n')分隔，返回一个包含各行作为元素的列表，如果参数 keepends 为 False，不包含换行符，如果为 True，则保留换行符。
- `startswith(substr, beg=0,end=len(string))` 检查字符串是否是以指定子字符串 substr 开头，是则返回 True，否则返回 False。如果beg 和 end 指定值，则在指定范围内检查。
- `strip([chars])` 在字符串上执行 lstrip()和 rstrip()
- `swapcase()` 将字符串中大写转换为小写，小写转换为大写
- `title()` 返回"标题化"的字符串,就是说所有单词都是以大写开始，其余字母均为小写
- `translate(table, deletechars="")` 根据 table 给出的表(包含 256 个字符)转换 string 的字符, 要过滤掉的字符放到 deletechars 参数中
- `upper()` 转换字符串中的小写字母为大写
- `zfill (width)` 返回长度为 width 的字符串，原字符串右对齐，前面填充0
- `isdecimal()` 检查字符串是否只包含十进制字符，如果是返回 true，否则返回 false。

## 元组

元组中的内容是不可变的

### 创建元组

使用小括号创建，要特别注意 0 或 1 个元素的元组

```python
t = (1, 2, 3)
t = (1, )
t = tuple()
```

### 访问修改

类似于列表的访问，直接使用中括号访问，直接赋值即可修改

```python
t[0] = 1
```

### 删除

元组不允许删除单个元素，只能删除整个元组

```python
del t
```

删除后再使用会报错

### 内部函数

- `len` 获取元组元素个数
- `max` 获取元组中元素最大值
- `min` 获取元组元素最小值
- `tuple` 将可迭代系列转为元组

## 字典

### 创建

使用大括号创建字典

```python
d = {}
d = {key : val}
```

### 访问修改

把相应的键放入到方括号中来访问字典

```python
val = d[key]
d[key] = val1
```

## 集合

### 创建

使用大括号创建，元素之间用逗号分隔，也可以使用函数创建集合

```python
s = set()
s = {1, 2, ...}
```

### 方法

- `add()` 为集合添加元素
- `clear()` 移除集合中的所有元素
- `copy()` 拷贝一个集合
- `difference()` 返回多个集合的差集
- `difference_update()` 移除集合中的元素，该元素在指定的集合也存在。
- `discard()` 删除集合中指定的元素
- `intersection()` 返回集合的交集
- `intersection_update()` 返回集合的交集。
- `isdisjoint()` 判断两个集合是否包含相同的元素，如果没有返回 True，否则返回 False。
- `issubset()` 判断指定集合是否为该方法参数集合的子集。
- `issuperset()` 判断该方法的参数集合是否为指定集合的子集
- `pop()` 随机移除元素
- `remove()` 移除指定元素
- `symmetric_difference()` 返回两个集合中不重复的元素集合。
- `symmetric_difference_update()` 移除当前集合中在另外一个指定集合相同的元素，并将另外一个指定集合中不同的元素插入到当前集合中。
- `union()` 返回两个集合的并集
- `update()` 给集合添加元素
- `len()` 计算集合元素个数

# 基础语法

## 编码格式

默认情况下python的编码格式为 `UTF-8` ，所有字符串都是 `unicode` 字符串

## 保留字

可以通过下列指令查看当前 python 的所有保留字

```python
import keyboard
keyboard.kwlist
```

## 注释

以 `#` 开头

## 行与缩进

在 python 中，通过缩进来表示代码块，所以 python 是缩进敏感的，缩进的长度是可变的，但是对于同一个代码块，缩进需要是相同的长度

python 中可以通过 `\` 来实现多行语句，在括号中的多行语句不需要使用反斜杠

## 用户输入

```python
x = input()
```

## 同一行显示多条语句

在同一行中使用多条语句，语句之间使用分号分割

```python
a = 1; print(a)
```

## print输出

在 `print` 中的输出默认是换行的，如果要实现不换行，需要在打印的内容之后加上 `end=" "`

## import

在 python 中使用 `import` 或者是 `from...import` 来导入相应的模块

## 条件语句

有两种表达形式

### if-elif-else语句

```python
if condition1:
 print(condition1)
elif condition2:
 print(condition2)
else:
 print(condition3)
```

### match-case语句

```python
match status:
 case 1:
  print(1)
 case 2:
  print(2)
 case 3:
  print(3)
 case _:
  print("default")
```

## 循环语句

### while循环

`while` 循环语句有两种表达形式，一般形式

```python
while condition:
 command
```

`while-else` 形式

```python
while condition:
 command
else:
 command_else
```

### for循环

`for` 循环有两种表达形式，一般形式

```python
for item in items:
 command
```

`for-else` 形式

```python
for item in items:
 command
else:
 command-else
```

其中的 `items` 可以使用 `range()` 语句来实现类似于 C 语言中循环的效果

## 推导式

### 列表推导式

格式如下

```python
[表达式 for 变量 in 列表] 
[out_exp_res for out_exp in input_list]
```

或者可以添加判断条件

```python
[表达式 for 变量 in 列表 if 条件]
[out_exp_res for out_exp in input_list if condition]
```

其中 `if condition` 可以过滤掉列表中不符合条件的值

### 字典推导式

如下

```python
{ key_expr: value_expr for value in collection }
{ key_expr: value_expr for value in collection if condition }
```

例子

```python
list = {1, 2, 3, 4}
dict_sqrt = {key : sqrt(key) for key in list}
```

### 集合推导式

如下

```python
{ expression for item in Sequence }
{ expression for item in Sequence if condition }
```

例如

```python
s = { i**2 for i in range(1,3)}
```

### 元组推导式

```python
(expression for item in Sequence )
(expression for item in Sequence if condition )
```

例如

```python
t = (i for i in range(1, 10))
```

## 迭代器与生成器

### 迭代器

迭代器是 Python 最强大的功能之一，是访问元素集合的一种方式。迭代器是可以记住遍历的位置的对象。迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退

- `iter()` 使用这个函数创建对应变量的迭代器

    ```python
    list = [1, 2, 3, 4, 5]
    it = iter(list)
    for i in it:
     print(i)
    ```

- `next()` 使用该函数用于从集合的第一个元素向后不断地访问，只能前进，不能后退，相当于一个迭代器就是一个数组，每运行一次从中取出一个元素，一直到取完为止。对于上述的代码可以写作

    ```python
    list = [1, 2, 3, 4, 5]
    it = iter(list)
    while(True):
     try:
      print(next(it))
     except StopIteration:
      sys.exit()
    ```

**创建迭代器类**

创建一个类作为迭代器需要在类中实现两个方法

- `__iter__()` 方法返回一个特殊的迭代器对象， 这个迭代器对象实现了 `next()` 方法并通过 `StopIteration` 异常标识迭代的完成
- `__next__()` 返回下一个迭代器的对象
- `StopIteration` 该异常用于标识迭代的完成，防止出现无限循环的情况。可以在 `__next__()` 方法中我们设置在完成指定循环次数之后就触发该异常来结束迭代

```python
class mynumber:
  def __iter__(self):
    self.a = 1
    return self

  def __next__(self):
    if self.a <= 20:
      self.a += 1
      return self.a
    else:
      raise StopIteration
```

### 生成器

在 Python 中，使用了 `yield` 的函数被称为生成器函数。是一种特殊的函数，可以在迭代过程中逐步产生值，而不是一次返回所有结果，调用一个生成器函数，返回的是一个迭代器对象

```python
def count(n):
  while n > 0:
    yield n
    n -= 1

generator = count(10)
while True:
  try:
    print(next(generator))
  except StopIteration:
    break
```

## 匿名函数

python 中使用 `lambda` 关键字来创建匿名函数。 `lambda` 可以具有任意数量的参数，但只能有一个表达式，不需要使用 `def` 来定义完整的函数。用于编写简单的单行的函数

### **特点**

- `lambda` 函数是匿名的，没有函数名称
- `lambda` 通常只包含一行代码，适用于编写简单的函数

### **语法**

```python
lambda arguments: expression
```

### **示例**

```python
 func = lambda a, b : a + b
 a = 1
 b = 2
 c = func(a, b)
 print(c)
```

## 装饰器

是 python 中的一个高级功能，允许动态的修改函数或类的行为。是一种函数，接收一个函数作为参数，并且返回一个新的函数或修改原来的函数，修饰器语法使用 `@decorator_name` 来应用在函数或者方法上

python 中有一些内置装饰器可定义静态方法和类方法： `@staticmethod` 和 `@classmethod`

### 基本语法

允许在不修改原函数代码的基础上，动态的增加或修改函数的功能，装饰器本质上是一个接收函数作为输入并返回一个新的包装过后的函数的对象

当一个函数使用 `@decorator_name` 进行修饰时，会在函数生成之前就把该函数作为参数写入装饰器函数中，相当于一开始就是生成的就是最新的函数

```python
def old_func(a, b):
  return a + b

def decorator_func(oldfunc):
  def newfunc(*args, **kwargs):
    sum = oldfunc(*args, **kwargs)
    return sum + 1
  return newfunc

print(old_func(1, 2))
new_func = decorator_func(old_func)
print(new_func(1, 2))

@decorator_func
def add2(a, b, c):
  return a + b + c

print(add2(1, 2, 3))
```

### 带参数的装饰器

可以在函数的修饰装饰器后添加参数，并且在定义装饰器函数时需要外嵌一层用于获取参数，可以将这个参数在函数内使用，如下

```python
def repeat(n):
  def decorator_func1(oldfunc):
    def newfunc(*args, **kwargs):
      sum = 0;
      for _ in range(n):
        sum += oldfunc(*args, **kwargs)
      return sum
    return newfunc
  return decorator_func1

@repeat(3)
def add3(a, b):
  return a + b

print(add3(1, 2))
```

### 类装饰器

生成一个装饰器类需要在函数内实现 `__call__` 方法，这个类接收一个函数作为参数，并且返回一个新的函数

```python
class DecoratorClass:
  def __init__(self, func):
    self.func = func
  def __call__(self, *args, **kwargs):
    result = self.func(*args, **kwargs)
    return result + 1
@DecoratorClass
def add4(a, b):
  return a + b

print(add(1, 2))
```

## 模块化

python 中可以把一些定义存放在其它文件夹中，然后导入该文件，这个文件被称为模块。但是需要注意的是，自己的文件名一定不能与第三方库的文件名一致，不然会有冲突错误

使用 `import` 语句导入模块，并且可以导入多个模块，语法为

```python
import module1[, module2, ...]
```

当解释器遇到 import 语句，如果模块在当前的搜索路径就会被导入

## 命名空间

### 命名空间（namespace）

1. 命名空间指的是变量存储的位置，每一个变量都需要存储到指定的命名空间当中
2. 每一个作用域都会有一个它对应的命名空间
3. 全局命名空间，用来保存全局变量。函数命名空间用来保存函数中的变量
4. 命名空间实际上就是一个字典，是一个专门用来存储变量的字典

### locals()

获取当前作用域的命名空间

1. 在全局作用域中调用locals()则获取-全局命名空间
2. 在函数作用域中调用locals()则获取-函数命名空间
3. 它返回的是一个字典

### globals()

可以用来在任意位置获取全局命名空间

## 文件操作

### 打开文件

使用 `open()` 函数来打开函数，语法如下

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)
```

其中

- `file` 必需，文件路径（相对或者绝对路径）
- `mode` 可选，文件打开模式
- `buffering` 设置缓冲
- `encoding` 一般使用utf8
- `errors` 报错级别
- `newline` 区分换行符
- `closefd` 传入的file参数类型
- `opener` 设置自定义开启器，开启器的返回值必须是一个打开的文件描述符

其中 `mode` 的参数有

| 模式 | 描述                                                                                                                                                               |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| t    | 文本模式 (默认)。                                                                                                                                                  |
| x    | 写模式，新建一个文件，如果该文件已存在则会报错。                                                                                                                   |
| b    | 二进制模式。                                                                                                                                                       |
| +    | 打开一个文件进行更新(可读可写)。                                                                                                                                   |
| U    | 通用换行模式（Python 3 不支持）。                                                                                                                                  |
| r    | 以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。                                                                                                   |
| rb   | 以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。一般用于非文本文件如图片等。                                                           |
| r+   | 打开一个文件用于读写。文件指针将会放在文件的开头。                                                                                                                 |
| rb+  | 以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。一般用于非文本文件如图片等。                                                                         |
| w    | 打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。                                           |
| wb   | 以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。   |
| w+   | 打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。                                             |
| wb+  | 以二进制格式打开一个文件用于读写。如果该文件已存在则打开文件，并从开头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。一般用于非文本文件如图片等。     |
| a    | 打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。             |
| ab   | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |
| a+   | 打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。                                 |
| ab+  | 以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。                                             |

默认为文本模式，如果要以二进制模式打开，需要在对应的模式之后加上 `b`

### 文件方法

- `file.close()` 关闭文件。关闭后文件不能再进行读写操作
- `file.flush()` 刷新文件内部缓冲，直接把内部缓冲区的数据立刻写入文件, 而不是被动的等待输出缓冲区写入
- `file.fileno()` 返回一个整型的文件描述符(file descriptor FD 整型), 可以用在如 `os` 模块的 `read` 方法等一些底层操作上
- `file.isatty()` 如果文件连接到一个终端设备返回 `True` ，否则返回 `False`
- `file.next()` 返回文件下一行
- `file.read([size])` 从文件读取指定的字节数，如果未给定或为负则读取所有
- `file.readline([size])` 读取整行，包括 `\n` 字符
- `file.readlines([sizeint])` 读取所有行并返回列表，若给定 `sizeint>0` ，返回总和大约为 `sizeint` 字节的行, 实际读取值可能比 `sizeint` 较大, 因为需要填充缓冲区
- `file.seek(offset[, whence])` 移动文件读取指针到指定位置
- `file.tell()` 返回文件当前位置
- `file.truncate([size])` 从文件的首行首字符开始截断，截断文件为 `size` 个字符，无 `size` 表示从当前位置截断。截断之后后面的所有字符被删除，其中 `windows` 系统下的换行代表2个字符大小
- `file.write(str)` 将字符串写入文件，返回的是写入的字符长度
- `file.writelines(sequence)` 向文件写入一个序列字符串列表，如果需要换行则要自己加入每行的换行符

## 正则表达式

正则表达式是一个特殊的字符序列，它能帮助你方便的检查一个字符串是否与某种模式匹配。在 Python 中，使用 `re` 模块来处理正则表达式。 `re` 模块提供了一组函数，允许在字符串中进行模式匹配、搜索和替换操作

### **re.match**

尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话， `match()` 就返回 `None`

```python
re.match(pattern, string, flags = 0)
```

其中

- `pattern` 匹配的正则表达式
- `string` 要匹配的字符串
- `flags` 标志位，用于控制正则表达式的匹配方式

可以使用 `group(num)` 或者 `groups()` 匹配对象函数来获取匹配表达式

- `group(num=0)` 匹配的整个表达式的字符串， `group()` 可以一次输入多个组号，在这种情况下它将返回一个包含那些组所对应值的元组
- `groups()` 返回一个包含所有小组字符串的元组，从 1 到所有的小组号

### **re.search**

用于扫描整个字符串，返回第一个成功匹配的字符串

```python
re.search(pattern, string, flags = 0)
```

参数与上述 `match` 一致

### 区别

`re.match` 只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回 `None` ，而 `re.search` 匹配整个字符串，直到找到一个匹配

### 检索和替换

使用 `re.sub` 函数来替换字符串中的匹配项

```python
re.sub(pattern, repl, string, count = 0, flags = 0)
```

其中

- `pattern` : 正则中的模式字符串，必选
- `repl` : 替换的字符串，也可为一个函数，必选
- `string` : 要被查找替换的原始字符串，必选
- `count` : 模式匹配后替换的最大次数，默认 0 表示替换所有的匹配，可选
- `flags` : 编译时用的匹配模式，数字形式，可选

**repl 函数**

可以传入参数，调用时默认传入参数就是祖匹配的字符串

```python
def double(matched):
    value = int(matched.group('value'))
    return str(value * 2)
 
s = 'A23G4HFD567'
print(re.sub('(?P<value>\d+)', double, s))
```

### compile

函数用于编译正则表达式，生成一个正则表达式（ Pattern ）对象，供 `match()` 和 `search()` 这两个函数使用。语法格式为

```python
re.compile(pattern[, flags])
```

- `pattern` : 一个字符串形式的正则表达式
- `flags` 可选，表示匹配模式，比如忽略大小写，多行模式等，具体参数为：
  - `re.IGNORECASE` 或 `re.I` 使匹配对大小写不敏感
  - `re.L` 表示特殊字符集 `\w, \W, \b, \B, \s, \S` 依赖于当前环境
  - `re.MULTILINE` 或 `re.M` 多行模式，改变 `^` 和 `$` 的行为，使它们匹配字符串的每一行的开头和结尾
  - `re.DOTALL` 或 `re.S` 使 `.` 匹配包括换行符在内的任意字符
  - `re.ASCII` 使 `\w, \W, \b, \B, \d, \D, \s, \S` 仅匹配 ASCII 字符
  - `re.VERBOSE` 或 `re.X` 忽略空格和注释，可以更清晰地组织复杂的正则表达式
  - 这些标志可以单独使用，也可以通过按位或 `|` 组合使用

### findall

在字符串中找到正则表达式所匹配的所有子串，并返回一个列表，如果有多个匹配模式，则返回元组列表，如果没有找到匹配的，则返回空列表，有两种表达式

```python
re.findall(pattern, string, flags=0)
pattern.findall(string[, pos[, endpos]])
```

其中

- `pattern` 匹配模式
- `string` 待匹配的字符串
- `pos` 可选参数，指定字符串的起始位置，默认为 0
- `endpos` 可选参数，指定字符串的结束位置，默认为字符串的长度

### **re.finditer**

和 `findall` 类似，在字符串中找到正则表达式所匹配的所有子串，并把它们作为一个迭代器返回。

```python
re.finditer(pattern, string, flags=0)
```

其中

- `pattern` 匹配的正则表达式
- `string` 要匹配的字符串。
- `flags` 标志位，用于控制正则表达式的匹配方式

### re.split

按照能够匹配的子串将字符串分割后返回列表，它的使用形式如下

```python
re.split(pattern, string[, maxsplit=0, flags=0])
```

- `pattern` 匹配的正则表达式
- `string` 要匹配的字符串
- `maxsplit` 分割次数，默认为 0，表示不限制次数。
- `flags` 标志位，用于控制正则表达式的匹配方式

### 说明

正则表达式太多了，也很复杂，可以看看这个，讲的还是挺全的

[Python3 正则表达式 | 菜鸟教程 (runoob.com)](https://www.runoob.com/python3/python3-reg-expressions.html)
