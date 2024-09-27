---
title: "C++内存管理"
date: 2024-03-19 19:32:24
categories: "编程语言"
tags: ["Cpp"]
summary: "C++内存管理"
---

### 四种内存分配和释放的方式

| 分配                     | 释放                       | 类型            | 是否可重载                 |
| ------------------------ | -------------------------- | --------------- | -------------------------- |
| malloc                   | free                       | C-function      | 否                         |
| new                      | delete                     | C++-expressions | 否                         |
| ::operator new()         | ::operator delete()        | C++-function    | 可                         |
| allocator<T>::allocate() | allocator<T>::deallocate() | C++-STL         | 可自由设计并且搭配任何容器 |

### **memory primitives**

| malloc                   | free                       | 不可重载                   |
| ------------------------ | -------------------------- | -------------------------- |
| new                      | delete                     | 不可重载                   |
| ::operator new()         | ::operator delete()        | 可重载                     |
| allocator<T>::allocate() | allocator<T>::deallocate() | 可以自由设计并搭配任何容器 |

### **new expression**

new 有两个作用，申请内存→构造函数，申请内存使用 operator new 实际上使用的还是 malloc 函数来申请内存

```cpp
Complex *pc = new Complex(1,2);

Complex *pc;
try
{
    void* mem = operator new(sizeof(Complex)); // 分配内存
    pc = static_cast<Complex*>(mem); // cast，就是指针转型，转化为Complex*类型
    pc->Complex::Complex(1,2); // 指针调用构造函数，用户不能直接使用该指令
}
catch(std::bad_alloc)
{
    // 申请失败就不执行构造函数
}
// 如果想要直接调用构造函数，可以用 placement new
new(p)Complex(1,2);

```

过程:

1. 调用operator new分配内存
2. 调用构造函数生成类象
3. 返回相应指针

### **delete expression**

就是先调用析构函数，在调用 delete 函数，delete函数实际上调用的还是 free 函数

```cpp
delete pc:
pc->~Complex();
operator delete(pc);

```

过程:

1. 调用析构函数
2. 调用operator new()

### **Ctor & Dtor**

构造函数不能直接调用，但是析构函数能够直接调用

### **array new & array delete**

就是创建一个数组，创建时会调用 placement new 函数来创建，调用构造函数的顺序 0，1，2

在delete时创建几个就需要delete几个，会调用多次析构函数，调用顺序 2，1，0

### **placement new  &  placement delete**

并不会分配内存，只是会使用已分配的内存来定义一个变量

```cpp
char *buf = new char[size];
Complex *pc = new(buf)Complex(1,2);

delete []buf;

```

### **per-class allocator**

想利用类内重载operator new去接管内存的分配，然后利用内存池的观念，即创建出一大段连续空间的内存，然后将其切割成一小段一小段，将创建的元素对象放在内存池切分好的各分段小内存片中，这样避免了多次调用new而造成生成多个带有cookie的内存空间。通过内存池的观念，可以生成一大段只带有两个头尾cookie的内存空间，而该一大段内存空间又被切分成每一小段的内存空间，且其中的每一小段内存空间片都可以共享这一整体的cookie信息。其实就是连续new多个变量，用得时候直接调用就可以。为了能将一大段内存空间切分成一小段一小段，然后通过单向链表的形式串接起来，所以必须多引入一个next指针。但这又会增加class的大小

### **static allocator**

可以认为是有一条指针指向一条链表,其实就是提前创建一个元素指针的数组，使用的时候在调用赋值，这个allocator就是用来维护这个链表的

### **macro for static allocator**

macro 宏，就是创建数组，每个数组元素都是静态的

### **global allocator**

这个全局的 allocator 里有16个链表，用来为所有的元素类型进行服务

### **new handler**

当 operator new 没有能力为你分配一个你所申请的 memory，会抛出一个异常，但是以前的老编译器会 返回0，所以就需要检查一下所申请的指针
如果想要不抛出异常，而是返回一个0，可以

```
new(nothrow) Foo;

```

在抛出异常之前，会先调用一个可以由 client 指定的 handler ，以下是 new handler 的形式和设定方法

```
typedef void(*new_handler)(); // 就是定义一个函数，在执行报错之前运行
new_handler set_new_handler(new_handler p)throw();

```

设计良好的 new_handler 只有两个选择

- 让更多的 memory 内存可用，就是先看看有哪些分配的内存可以释放掉，不影响整个运作的，释放完之后再次分配一次内存，看能否分配成功
- 调用 abort() 或者 exit()

### **malloc**

malloc 内存必须是 16 的倍数
cookie 必须是 8 个位

**内存布局:**

| cookie       | 记录区块的大小                                         |
| ------------ | ------------------------------------------------------ |
| debug header |                                                        |
| block size   |                                                        |
| debug tail   |                                                        |
| pad          | 这一块就是为了能够使申请的内存区块是16的倍数而做的填充 |

### **allocator**

```
class allocator
```

allocator 里面最重要的函数，没有操作

```
pointer allocate(size_type _N, const void *)
{
    return (_Allocate((difference_type) _N,(pointer)0));
}
void deallocator(void _FARQ *_P,size_type)
{
    operator delete(_P);
}

```

```
pointer allocate(size_type _N, const void *)
{
    return (::allocate((difference_type) _N,(pointer)0));
}
void deallocator(void _FARQ *_P,size_type)
{
    ::operator delete(_P);
}

```

在 GNU2.9 中，所有的容器都使用着 alloc

### **std::alloc vs __pool_alloc**

在 G2.9 的 `std::alloc` 变成了 G4.9 的 `_pool_alloc` 变成了编制外的 `__pool_alloc` 存在于 `__gnu_cxx` 里面，不再是标准库里的函数

### G4.9 标准分配器

`allocator` 实际上就是引用 `new()` 和 `delete()` 函数

`pool_alloc` 里面没有任何的 data 只有函数，所以大小为1

### **[std::alloc](https://blog.csdn.net/qq_34269632/article/details/115636008?spm=1001.2014.3001.5506)**

在G2.9中， `std::alloc` 中有 16 个容器，可以满足所有的元素类型,每一个位置上分配的元素大小为 `8x(n+1)` 所以当元素的大小超出当前的最大的元素大小时就会调用 `malloc` 来创建容器，在申请使用的时候，就会分配给容器一个容器内存，创建的时候会根据容器内的元素类型大小来分配内存，例如元素大小为 32 位的话就会分配 32 x 20 个内存，如果不够用的话，就会再次申请，每次申请之后会分配内存 32 x 20 x 2 位内存，其中的一半会被分割用来分配所申请的内存，另一半就挂载在 `alloc` 容器上，RounndUp  就是能把所申请的内存给补充为8的倍数

**embedded pointer**
`embedded pointer` 指针，是用来管理内存块的，相当于一个链表。 内存块，在客户使用的时候，就会把一整块内存当作是元素，就会把内存块中的指针给覆盖掉

### **[malloc](https://blog.csdn.net/qq_34269632/article/details/115704696)**

```
LPVOID VirtualAlloc{
    LPVOID lpAddress,       // 要分配的内存区域的地址 (当实参为0时，由操作系统指定地址)
    DWORD dwSize,           // 分配的大小
    DWORD flAllocationType, // 分配的类型
    DWORD flProtect         // 该内存的初始保护属性
};
```

1. `virtualAlloc` 是一个 Window API 函数，该函数的功能是在调用进程的虚拟地址空间预定或者提交一部分页
2. `flAllocationType` ：
    - `MEM_RESERVE` 保留分配地址，不分配物理内存。这样可以阻止其它分配函数 `malloc` 和 `LocalAlloc` 等再使用已保留的内存范围，直到它被释放
    - `MEM_COMMIT` 为指定地址空间提交物理内存。这个函数初始化内在为零

在 window 下的 virtualloc 有两种格式

- `VirtualAlloc(0, 1Mb, MEM_RESERVE,...)` 表示保留，不是真的分配
- `VirtualAlloc(0, 1Mb, MEM_COMMIT,...)` 表示真实分配出去

### **free(p)**

内存中的分级:

header → 32个group → 64个 free-list，程序中每个 header 管理 32 个组，每个组管理 64 条链表

使用 free(p) 先确定 header→group→free-list，在程序初始化的时候就创建了 16 个 header，看 p 的地址在那个 header 之间，在确定位置

### **内存分段管理**

在申请内存的时候不是直接申请一大块，而是申请 32 kB，里面有 16 个header，每个header中有 32 个 group，每个group 中有 64 条链表，在 64 条链表上有一个整数。在内存申请中，对于已经申请来的内存，如果里面的存储数据已经被释放掉，也不会当时就还给系统，而是等程序结束后所用使用的释放的内存一起归还

_sbh_pHeaderDefer 是个指针，指向一个全回收 group 所属的 header，这个 group 原本打算被释放掉，但是暂时保留，当有第二个全回收 group 之后就会释放掉，并且把第二个 group 设为 defer。想要继续分配内存的话，可以直接从 defer 中取出 block 来进行分配，这时候 defer 指针将会被取消

SBH 分配好 block 之后，最终会将 cntEntries 累加1，但是会检查它原本是否为0，如果是，那就看此次所在的 header 是否defer 并且此次所用值 Group 是否 defer，都吻合就取消其defer身份(令 _sbh_pHeaderDefer = null)

这意味着 defer 必须是一个全回收的group，全新的 group不会是 defer，_sbh_inGroupDefer 是一个索引，指出 Region 中哪个 group 是 defer

### **Loki allocator**

里面有3个class

1. Chunk 最低阶
2. FixedAllocator 中阶
3. SmallObjectAllocator 高阶，也就是使用者面对的 allocator

```
Chunk
pData_: unsigned char*;  // 就是一个指针，将来会分配一块较大的内存
firstAvailableBlock_: unsigned char;   // 第一个可用区块的位置
// 一般的分配器都是把分配的内存切成小块，用链表连接，并且引入一个嵌入式指针来引用
// 但是这里不是使用指针，这个数据就相当于一个索引，就是表示当前指针指向的是第几号内存块，这个索引值不是按顺序排列的,就相当于一个最高优先权
blocksAvailable_: unsigned char;
// 表示还有多少内存可以分配

FixedAllocator
chunks_: vector<Chunk>; // 表示生成多个Chunk
allocChunk_:Chunk*; // 指向Chunk
deallocChunk_:Chunk*;// 指向Chunk
// 会指向其中的某两个

SmallObjAllocator
pool_:vector<FixedAllocator>;// 申请了多个FixedAllocator
pLastAlloc:FixedAllocator*; // 指向某两个 FixedAllocator
pLastDealloc:FixedAllocator*;
chunkSize:size_t;
maxObjectSize:size_t;

```

特点:

1. 精简强悍，手段暴力
2. 使用特殊手段(用array代替list，用index来取代pointer)
3. 能使用简单的方式判断chunk是否全回收，进而将memory归还
4. 有deferring(暂缓归还)的能力
5. 这是个allocator，用来分配大量小块的，不带cookie的memory block，最佳客户是容器，本身就使用vector
6. 用到了标准库的容器和分配器，也用到了 loki 的分配器和容器

### **GUN C++对allocator的描述**

对于元素类型为 T 的容器的 `Allocator` 的模板实参默认是 `allocator<T>`. 其接口只有大约20个 public 声明，包括嵌套的 `typedefs` 和成员函数

1. `T* allocate(size_type n, const void* hint = 0)`
2. `void deallocator(T* p,size_type n)`
这里的 n 表示客户申请的元素个数，实际空间大小应该是 `n*sizeof(T)`
    - 这些内存通常使用 `operator::new` 来获得，但是何时调用和调用频率无具体指定
        - `__gnu_cxx::new_allocator` 实际上分配内存调用的是 `::operator new` 和 `::operator delete`
        - `__gnu_cxx::malloc_allocator` 实际上调用的是 std::malloc 和 std::free
    - 另一种方法就是使用智能型 allocator，将分配所得的内存加以缓存(cache)，实现机制:
3. 可以是一个 `bitmap index`，用于索引至一个以 2 的指数倍成长的篮子
4. 也可以是一个相较之下比较简易的 fixed-size pooling cache ，这里的cache 被程序内的所有容器共享，而 `operator new` 和 `operator delete` 不经常被调用，这可以带来速度上的优势。使用这个技巧的 `allocator` 包括:
    - `__gnu_cxx::bitmap_allocator` 使用bit-map来追踪被使用和未被使用的内存块
    - `__gnu_cxx::pool_allocator` 申请一个内存池
    - `__gnu_cxx::__mt_alloc`
    - `__gnu_cxx::debug_allocator` 可以包覆于任何 `allocator` 之上,它把客户的申请量添加一些，然后由 `allocator` 回应，并且以那一小块额外内存放置 `_size` 信息 `deallocator()` 收到一个pointer，就会检查size并且以 assert()保证吻合
    - `__gnu_cxx::array_allocator` 允许分配一个已知且固定大小的内存块，内存来自 `std::array objects` ，使用这个 `allocator` 之后，固定大小的容器就不需再调用 `::operator new` 和 `::operator delete` 。这就允许我们使用 `STL abstractions` 而无需再进行时添乱，增加开销。甚至再 `program startup()` 情况下也可以使用
    - `__gnu_cxx::new_alloc` 实现出简单的 `operator new` 和 `operator delete` 语义
    - `__gnu_cxx::malloc_alloc` 与上例唯一不同的是，使用C语言的 `std::malloc 和 std::free`。 存在于 `stdio.h` 库里

`Class allocator` 只拥有 `typedef, constructor, rebind` 等成员，继承自一个 `high-speed-extension allocators`(高速扩充分配器)，也因此所有的分配和归还都取决于这个 `base class` ，这个 `base class` 用户无法触碰

GNU C++ 提供了三项综合测试来完成 `C++ allocator` 之间的对比:
`Insertion` 经过多次 `iterations` 之后各种 STL 容器将拥有某些的极大量
多线程环境中的 `insertion and erasure`，这个测试展示 `allocator` 归还内存的能力，以及测量线程之间对内存的竞争
A threaded producer/consumer model 分别测试循序式 `sequence` 和关联式 `associative` 的容器

### **__gnu_cxx::new_allocator**

`#include <ext/new_allocator.h>`

所有分配器的根源就是 memory 中 `include <xmemory>`， `xmemory` 中 `include <xmemory0>` ，里面有两个关于 `allocate` 的重载

```
typedef value*pointer;
typedef size_t size_type;
void deallocate(pointer _Ptr, size_type)
{ // deallocate 在 _Ptr 处的元素，不考虑 size
    ::operator delete(_Ptr);
}
pointer allocator(size_type_Count)
{ // allocate 一定数量的元素
    return (_Allocate(_Count,(pointer)0)); // 就是调用 ::operator new
}
pointer allocator(size_type_Count,const void*)
{// allocate 一定数量的元素，不考虑 隐含参数
    return (allocate(_Count)); // 调用上面的 allocate
}

```

标准库 `<bits/allocator.h>` 中的 `allocator` 继承于 `new_allocator` ，list vector deque 的分配器就是标准库的分配器

### **__gnu_cxx::malloc_allocator**

里面的 `allocate` 就是调用的 `std::malloc` , `deallocate` 调用的是 `std::free`

### **__gnu_cxx::array_allocator**

`#include < ext/array_allocator.h>`

内部就是一个c++数组，它的第二模板参数就是一个小版本的标准

```
teplate<typename _Tp,typename _Array = std::tr1::array<_Tp,1>>
class array_allocator:public array_allocator_base<_Tp>
{
public :
    typedef size_t size_type;
    typedef _Tp value_type;
    typedef _Array array_type;

private:
    array_type* _M_array; // 就是一根指针，用来指向一种 array_type*,指向静态的数组，不需要释放，只有动态才需要释放
    size_type _M_used;
public:
    array_allocator(array_type* __array = NULL)throw():_M_array(_array),M_used(size_type()){}
    pointer allocate(size_type __n,const void * = 0)
    {
       if(_M_array == 0||_M_used+_n>_M_array->size())
            std::__throw_bad_alloc();
        pointer __ret = _M_array ->begin()+_M_used;
        _M_used += __n;
        return __ret;
    }
};

```

### **debug_allocator**

就是把外部的一个分配器包装起来，包装进 `debug_allocator` 中
`<ext/debug_allocator.h>`

```
pointer allocate(size_type __n)
{
  pointer __res = _M_allocator.allocate(__n + _M_extra);  // 申请内存的时候申请额外内存
  size_type* __ps = reinterpret_cast<size_type*>(__res);
  *__ps = __n;
  return __res + _M_extra;  // 返回 额外 的数据之后的数据，但是不适用额外值
}
void deallocate(pointer __p, size_type __n)
{
  if (__p)
  {
    pointer __real_p = __p - _M_extra; // 要释放掉的时候就把extra也释放掉， extra 作用是记录分配的内存的数量
    if (*reinterpret_cast<size_type*>(__real_p) != __n)
    {
      throw std::runtime_error("debug_allocator::deallocate wrong size");
    }
    _M_allocator.deallocate(__real_p, __n + _M_extra);  // 用所包装的分配器归还内存
  }
  else
    throw std::runtime_error("debug_allocator::deallocate null pointer");
}

```

### **__pool_alloc**

`<ext/pool_allocator.h>`
`__gnu_cxx::allocator<T>`

### **bitmap_allocator**

如果只申请单个元素，调用下面两个函数

- `_M_allocate_sigle_object()`  只申请单个元素
- `_M_deallocate_sigle_object()` 只释放一个元素

如果申请多个元素，那就调用下面的函数

- `::operator new`
- ::operator delete

申请内存的时候，直接申请 64 个 blocks，每个blocks都是8个字节

![20210501161118199.png](./20210501161118199.png)

count (记录整个区块的大小) - usecount(就是用户使用掉的blocks数量) - `bitmap[1]` - `bitmap[0]` - 64个blocks(将来会成倍增长)

`bitmap[0]` 和 `bitmap[1]` 就是两个整数，一个整数是 32 位，所以要用两个整数来表示 64 位的blocks,最开始的时候 `bitmap[0]` ， `bitmap[1]` 都是 FFFFFFFF，usecount是0

随着不断地使用， `bitmap[0]` 和 `bitmap[1]` 会不断变小

例如如果已经使用了 63 个区块的话:  `usecount = 63` ， `bitmap[1],bitmap[0]` 数为 80000000 00000000 ，每个blocks的大小就是  8 16 32 64...
`__mini_vector` 在这个 `bitmap_allocator` 中自己设计的一个容器，用来放置这个 blocks 有三个指针:

```
_M_start // 指向blocks开头的指针
_M_finish // 指向 blocks 结尾的下一个元素
_M_end_of_storge // 指向
```

如果已经把第一个 super blocks 用光之后，就会使用第二个 super blocks，第二个 super blocks 中有 128 个blocks，是第一个的两倍，这时候会使用 4 个 bitmap 来记录 super blocks size，而且负责这个blocks的 __mini_vector 也会扩充编程四个指针

![2021050116201663.png](./2021050116201663.png)

不同类型的 blocks 不会混在一起，是分开存储的，只是在 `__mini_vector` 上有体现， `__mini_vector` 的两个指针中间的东西会增长，中间的区块就是表示的存储区块

如果在分配过程中没有运行全回收，那么申请的内存的区块大小会成倍增长，每次的全回收会导致申请的区块减半

**全回收** 

如果第一个blocks 中已经回收了几个 block，并未发生全回收时，接下来如果要分配 blocks 的话，是从 已经申请好的但是并未用完的 blocks 中分配

如果第二个 blocks 用光了，还要再分配几个blick，而且已经回收了的blocks 中有空余，就会调用，如果用光了就会再次申请

其实每个blocks就是相当于一个 __mini_vector 的元素，元素排列以blocks的大小为依据，而且vector中最多可以有 64 个元素

如果__mini_vector 中元素个数已经满了，还要新进来元素，那就判断新进来的blocks的大小，如果比原来里面的最末者还要大，就直接delete掉，如果小于最末者，就插入适当的位置，并且把最末者delete掉

### **const**

当成员函数的 const 和 non-const 两个版本都存在，那么 const object 只会调用const版本，non-const object 只能调用 non-const 版本

|                                                                                                                                       | const object( data members 不变 ) | non-const object(data members 可变动) |
| ------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- | ------------------------------------- |
| const member function(保证不更改data members)                                                                                         | true                              | true                                  |
| non-const member function(不保证不更改data members)                                                                                   | false                             | true                                  |
| 当设计成员函数的时候，一定要注意，如果使用 const 元素，就必须搭配 const member function，否则会默认调用的是 non-const member function |                                   |                                       |
| non-const member object 可以调用 const member function                                                                                |                                   |                                       |
| const 属于签名部分                                                                                                                    |                                   |                                       |

### **new delete**

new 先分配内存 再调用构造函数

delete 先调用析构函数 再释放内存

可以重载 new 和 delete 函数

array new 一定要搭配 array delete

如果有重载就调用重载的，否则调用全局的

```
void* operator new(size_t size)
{return malloc(size);}
void* operator new(size_t size,void*start)
{
    return start;
}
void* operator new(size_t size, long extra)
{
    return malloc(size+extra);
}
void* operator new(size_t size,long extra,char init)
{
    return malloc(size+extra);
}
```

### **malloc free**

在代码中可以直接利用 `malloc` 申请一个固定大小的内存

```
int *p = (int *)malloc(sizeof(int));
```

**c++的结构体内存大小**

1. 空结构体占用1个字节
2. 结构体中包含的数组则不占用空间
3. 结构体中的成员函数不占用空间
4. 结构体中的一个或多个虚函数只占用一个指针大小的空间
    
    虚函数表