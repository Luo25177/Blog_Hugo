---
title: "matlab中LMI工具箱使用"
date: 2024-03-18 16:47:38
categories: "小工具"
tags: ["matlab", "LMI", "矩阵不等式"]
summary: "matlab中LMI线性矩阵不等式子求解工具箱使用"
---

### 初始化一个 LMI 系统

```matlab
setlmis(lmi0);
setlmis([]); % 常用
```

### 向 LMI 系统中添加矩阵变量 `lmivar`

```matlab
X = lmivar(type, struct); % 常用
[X, ndec, xdec] = lmivar(type, struct); % ndec表述与X有关的决策变量的个数，xdec表示X对这些决策变量的初始依赖关系
```

| type类型对应的数值 | 含义                                   |
| ------------------ | -------------------------------------- |
| 1                  | 对角线对称矩阵格式，每个对角块都是满块 |
| 2                  | 矩形块，struct=[m,n] 表示 m x n 阶矩阵 |
| 3                  | 描述复杂类型的矩阵                     |

1. `type=1`

    struct 包含两个基本变量

    - 第一个描述矩阵块的阶数
    - 第二个描述矩阵块的类型

        | 0   | 标量 |
        | --- | ---- |
        | 1   | 满块 |
        | -1  | 零块 |
2. `type=2`
3. `type=3`

    描述复杂类型的矩阵

### 返回 LMI 的函数的内部描述 `getlmis`

```matlab
lmisys = getlmis % lmisys 称为存储在机器内部线性矩阵不等式系统的名称，一个线性矩阵不等式以 setlmis 开始，以 getlmis 结束
```

### 确定线性矩阵不等式中各项的内容 `lmiterm`

```matlab
lmiterm(termID, A, B, flag);
```

`termID`：是一个四项的整数向量，用于指定 LMI 中相的位置和所涉及的矩阵变量

- `termID(1) = +p/-p`，其中 +p 表示第 p 个 LMI 左侧项，-p 表示第 p 个 LMI 右侧的相，也就是 $X_1<X_2$ 中不等式的左右
- `termID(2 : 3) = [0, 0]`，对应于外部的矩阵
- `termID(2 : 3) = [i, j]`，对应于左/右侧因子的第 (i, j) 块中的项
- `termID(4) = 0` 对应与外部的变量
- `termID(4) = x` 对应于变量 `AXB` ，X 是所需要反馈的矩阵变量，A，B是设定好的矩阵
- `termID(4) = -x` 对应于变量 `AX’B`

对于 `(AXB) + (AXB)' = AXB + B'XA'` 设置 `flag = 's'` ，允许使用单个 lmiterm 命令指定类别

### 约束条件下求 LMI 可行性问题 `feasp`

```matlab
[tmin, xfeas] = feasp(lmisys, options, target); % 如果返回 tmin <= 0 说明系统可行
```

- `lmisys` 称为存储在机器内部线性矩阵不等式系统的名称
- `options` 是一个五维向量，用来描述迭代过程中的一些控制参数
  - `option(1)` 分量不可用
  - `option(2)` 该参数设定优化迭代过程中允许的最大迭代次数，默认值是 100
  - `option(3)` 该参数设定了可行域的半径， `option(3) = R > 0` 表示限制决策变量在球体内 $\sum_{i=1}^Nx_i^2<R^2$ 中，或者说向量 `xfeas` 的欧式范数不超过 R，该参数默认值为 $R=10^9$。可行域半径的设定可以避免产生具有很大数值的解 x，同时也可以加快计算过程，改进数值稳定性
  - `option(4)` 该参数可用于加快迭代过程的结束，它提供了反映优化过程中迭代速度和解得到精度之间的一个折中指标。当该参数取值为一个正整数 n 时，表示在最后的 n 次迭代中如果每次迭代后 t 的减小幅度不超过 1%，则优化迭代过程就停止，默认是 10
  - `option(5)` `option(5)=1` 表示不显示迭代过程中的数据， `option(5)=0` 则相反
  - 将对应的维度设置为 0 就是使用默认值
- `target` 为 `tmin` 设定了目标值，使得只要 `tmin < target` 则优化过程停止，默认值为 0

为了调用 `feasp` 函数，需要首先确定线性矩阵不等式系统

### 在 LMI 约束下最小化线性目标 `mincx`

$$
\min C^Tx\\L(x)<R(x)
$$

```matlab
[copt, xopt] = mincx(lmisys, c, options, xinit, target);
```

LMI 系统由 `lmisys` 描述。 向量 c 的长度必须与 x 相同。 该长度对应于函数 `decnbr` 返回的决策变量的数量。 对于用矩阵变量表示的线性目标，可以使用 `defcx` 轻松导出足够的c向量。

- `target` 是目标函数的一个设定目标，只要某个可行的 x 满足 $e^Tx≤target$，求解过程就停止
- `option` 是一个 5 维向量，用来描述优化迭代过程中的一些控制参数
  - `option(1)` 该参数确定了最优值 `copt` 所要求的精度，默认是 $10^{-2}$
  - `option(2)` 该参数设定优化迭代过程中允许的最大迭代次数，默认值是 100
  - `option(3)` 该参数设定了可行域的半径，与 `feasp` 中的定义一致
  - `option(4)` 该参数用于加快迭代过程的结束，当该参数取值为一个正整数 n 时，表示在最后的 n 次迭代中如果每次迭代后 $C^Tx$ 的减小幅度在给定的京都内，则优化迭代过程就停止，默认是 5
  - `option(5)` `option(5)=1` 表示不显示迭代过程中的数据， `option(5)=0` 则相反
  - 将对应的维度设置为 0 就是使用默认值

### others

`Mat2dec` 根据矩阵变量值构造决策变量向量

`Dec2mat` 从决策变量向量中提取矩阵变量值。将求解器求出的向量，根据对称性转换为普通的矩阵
