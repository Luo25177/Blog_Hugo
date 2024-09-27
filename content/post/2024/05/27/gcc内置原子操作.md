---
title: "gcc内置原子操作"
date: 2024-05-27 20:34:52
categories: "小工具"
tags: ["gcc", "C", "Cpp"]
summary: "gcc 内置的原子操作"
---

## 前言

Gcc 4.1.2版本之后，对 X86 或 X86-64 支持内置原子操作。也就是不需要引入第三方库（如pthread）的锁保护，即可对1、2、4、8字节的数值或指针类型，进行原子加/减/与/或/异或等操作

## __sync 接口

具体信息查看 [Built-in functions for atomic memory access](https://gcc.gnu.org/onlinedocs/gcc-4.1.2/gcc/Atomic-Builtins.html)

- `type __sync_fetch_and_add (type *ptr, type value, ...)` 将 `value` 加到 `*ptr` 上，并且结果保存在 `*ptr` 上，返回操作之前的 `*ptr` 的值
- `type __sync_fetch_and_sub (type *ptr, type value, ...)` 从 `*ptr` 减去 `value` ，结果保存到 `*ptr` ，并返回操作之前 `*ptr` 的值
- `type __sync_fetch_and_or (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相或操作，结果保存在 `*ptr` 中，返回操作之前 `*ptr` 的值
- `type __sync_fetch_and_and (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相与操作，结果保存在 `*ptr` 中，返回操作之前 `*ptr` 的值
- `type __sync_fetch_and_xor (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相异或操作，结果保存在 `*ptr` 中，返回操作之前 `*ptr` 的值
- `type __sync_fetch_and_nand (type *ptr, type value, ...)` 将 `*ptr` 取反后与 `value` 相与操作，结果保存在 `*ptr` 中，返回操作之前 `*ptr` 的值
- `type __sync_add_and_fetch (type *ptr, type value, ...)` 将 `value` 加到 `*ptr` 上，结果保存在 `*ptr` 中，返回操作之后的 `*ptr` 的值
- `type __sync_sub_and_fetch (type *ptr, type value, ...)` 将 `*ptr` 减去 `value` 之后，结果保存在 `*ptr` 中，返回操作之后的 `*ptr` 的值
- `type __sync_or_and_fetch (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相或操作，结果保存在 `*ptr` 中，返回操作之后 `*ptr` 的值
- `type __sync_and_and_fetch (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相与操作，结果保存在 `*ptr` 中，返回操作之后 `*ptr` 的值
- `type __sync_xor_and_fetch (type *ptr, type value, ...)` 将 `*ptr` 与 `value` 相异或操作，结果保存在 `*ptr` 中，返回操作之后 `*ptr` 的值
- `type __sync_nand_and_fetch (type *ptr, type value, ...)` 将 `*ptr` 取反后与 `value` 相与操作，结果保存在 `*ptr` 中，返回操作之后 `*ptr` 的值
- `bool __sync_bool_compare_and_swap (type *ptr, type oldval, type newval, ...)` 比较 `*ptr` 与 `oldval` 的值，如果两者相等，将 `newval` 更新到 `*ptr` 中并且返回 `true` ，否则返回 `false`
- `type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)` 比较 `*ptr` 与 `oldval` 的值，如果两者相等，将 `newval` 更新到 `*ptr` 中并且返回操作之前 `*ptr` 的值
- `__sync_synchronize (...)` 相当于是内存栅栏，必须保证之前的操作完成之后才进行下一步的操作，保证执行的顺序性
- `type __sync_lock_test_and_set (type *ptr, type value, ...)` 将 `value` 写入 `*ptr` ，对 `*ptr` 加锁，并且发返回操作之前的 `*ptr` 的值
- `void __sync_lock_release (type *ptr, ...)` 将 0 写入到 `*ptr` 并且对 `*ptr` 解锁

## __atomic 接口

这些功能旨在取代遗留的 `__sync` 内置函数。主要区别在于，请求的内存顺序是函数的参数。新代码应始终使用 `__atomic` 内置函数而不是 `__sync builtins`

具体信息查看 [Built-in Functions for Memory Model Aware Atomic Operations](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html)

- `type __atomic_load_n (type *ptr, int memorder)` 原子加载操作，返回 `*ptr` 的内容
- `void __atomic_load (type *ptr, type *ret, int memorder)` 原子加载的通用版本，返回 `*ret` 中 `*ptr` 的内容
- `void __atomic_store_n (type *ptr, type val, int memorder)` 实现原子存储，将 `val` 写入 `*ptr` 中
- `void __atomic_store (type *ptr, type *val, int memorder)` 原子存储的通用版本，将 `*val` 中的值存储到 `*ptr` 中
- `type __atomic_exchange_n (type *ptr, type val, int memorder)` 内置原子交换操作，将 `val` 写入到 `*ptr` 中，并且返回 `*ptr` 操作之前的内容
- `void __atomic_exchange (type *ptr, type *val, type *ret, int memorder)` 将 `*val` 的内容存储到 `*ptr` 中，并且 `*ptr` 的原始值被复制到 `*ret` 中
- `bool __atomic_compare_exchange_n (type *ptr, type *expected, type desired, bool weak, int success_memorder, int failure_memorder)` 原子比较和交换操作，将 `*ptr` 中的内容与 `*expected` 的内容进行比较，如果相等则将 `desired` 的内容写入 `*ptr` 中，如果不相等，则将 `*ptr` 当前内容写入到 `*expected` 中， `weak` 用于若比较交换的 `true` ，可能会错误的失败，而 `false` 则用于强变化，永远不会错误的失败。如果将 `desired` 写入 `*ptr` 就返回 `true` ，并且根据 `success_memorder` 指定的内存顺序影响内存，否则返回 `false` 并且根据 `failure_memorder` 堆内存进行操作
- `bool __atomic_compare_exchange (type *ptr, type *expected, type *desired, bool weak, int success_memorder, int failure_memorder)` 该函数内置了 `__atomic_compare_exchange` 的通用版本，该函数实际上与 `__atomic_compare_exchange_n` 相同，只是所需要的值也是一个指针
- `type __atomic_add_fetch (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相加操作，结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_sub_fetch (type *ptr, type val, int memorder)` 执行 `*ptr` 减去 `val` ，并且将结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_and_fetch (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相与操作，结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_xor_fetch (type *ptr, type val, int memorder)`  将 `*ptr` 与 `val` 执行相异或操作，结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_or_fetch (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相或操作之后，将结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_nand_fetch (type *ptr, type val, int memorder)` 将 `*ptr` 取反之后再与 `val` 执行相与操作，结果保存在 `*ptr` 中，返回操作之后的 `*ptr`
- `type __atomic_fetch_add (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相加操作，结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `type __atomic_fetch_sub (type *ptr, type val, int memorder)` 执行 `*ptr` 减去 `val` ，并且将结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `type __atomic_fetch_and (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相与操作，结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `type __atomic_fetch_xor (type *ptr, type val, int memorder)`  将 `*ptr` 与 `val` 执行相异或操作，结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `type __atomic_fetch_or (type *ptr, type val, int memorder)` 将 `*ptr` 与 `val` 执行相或操作之后，将结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `type __atomic_fetch_nand (type *ptr, type val, int memorder)` 将 `*ptr` 取反之后再与 `val` 执行相与操作，结果保存在 `*ptr` 中，返回操作之前的 `*ptr`
- `bool __atomic_test_and_set (void *ptr, int memorder)` 对 `*ptr` 的字节执行原子测试和设置操作，当前仅当当前内容被设置时，该字节被设置成某个实现定义的非 0 设置值，并且返回值为 `true` ，只能用于 `bool` 或者 `char` 类型的操作数，对于其它类型，只能设置一部分 (1byte)
- `void __atomic_clear (bool *ptr, int memorder)` 在 `*ptr` 上执行原子清除操作，运算之后 `*ptr` 中内容为 0，只能用于 `bool` 或者 `char` 类型的操作数，并且与 `__atomic_test_and_set` 结合使用，对于其它类型只能完成部分操作 (1byte)
- `void __atomic_thread_fence (int memorder)` 指定执行的内存顺序当作进程之间的同步的栅栏
- `void __atomic_signal_fence (int memorder)` 充当线程和基于同一线程的信号处理程序之间的同步栅栏
- `bool __atomic_always_lock_free (size_t size, void *ptr)` 如果 `size` 字节的对象始终为目标体系结构生成无锁原子指令，则返回 `true` ， `size` 必须解析为编译时常量，并且结果也解释为编译时常量。 `ptr` 是一个可选的指针，指向可以用于确定对齐的对象，为 0 表示应该使用典型的对齐方式，编译器也可能忽略此参数
- `bool __atomic_is_lock_free (size_t size, void *ptr)` 如果 `size` 字节的对象始终为目标体系结果生成无锁原子指令，则返回 `true` 。如果未知内置函数是无锁的，则会调用 `__atomic_is_lock_free` 的运行时例程。 `ptr` 是一个可选的指针，指向可以用于确定对齐的对象，为 0 表示应该使用典型的对齐方式，编译器也可能忽略此参数
