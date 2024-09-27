---
title: "Riccati方程"
date: 2024-03-18 16:34:16
categories: ["控制理论"]
tags: ["数理知识"]
summary: "Riccati方程求解"
---

## 前言

[第四讲：李群和李代数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/33156814)

### 辛矩阵

数学中，辛矩阵是指存在一个 $2n\times 2n$ 的矩阵 M，使之满足

$$
M^T\Omega M=\Omega
$$

其中 $M^T$ 为 M 的转置矩阵，而辛矩阵 $\Omega$ 是一个固定的可逆斜对称矩阵，这类矩阵在适当的变化后皆能表为

$$
\Omega=\begin{bmatrix}0&I\\-I&0\end{bmatrix}
$$

或者

$$
\Omega=\begin{bmatrix}0&1\\-1&0\\&&...\\&&&0&1\\&&&-1&0\\\end{bmatrix}
$$

两者的差异仅在于基的排列，其中 $I$ 是 $n\times n$ 的单位矩阵，此外 $\det\Omega=1$ 并且 $\Omega ^{-1}=-\Omega$

**性质**

1. $\Omega^T=-\Omega=\Omega^{-1}$
2. $\Omega^T\Omega=\Omega\Omega^T=I$
3. $\Omega\Omega=-I$
4. $\det\Omega=1$

## 代数Riccati方程

黎卡提方程是最简单的一类非线性方程。例如

$$
y' = P(x)y^2+Q(x)y+R(x)
$$

是最优控制的非线性方程，和连续性时间或者是离散时间下无限时间的最优控制有关。

标准的Raccati方程分为两种：

1. 连续时间代数Riccati方程(CARE):
    
    $$
    A^TP+PA-PBR^{-1}B^TP+Q=0
    $$
    
2. 离散时间代数Riccati方程(DARE):
    
    $$
    P=A^TPA-(A^TPB)(R+B^TPB)^{-1}(B^TPA)+Q
    $$
    

在无限时间的最佳控制问题中，关注的是一些变数在相当时间之后的数值，因此需要在现在选定的控制变数的数值，让系统在之后的时间都在最佳的状态下运作，控制变数在任意时间下的最佳值可以使用 Riccati 方程的解以及状态变数当时的观测值求得，若观测变数及控制变数都不止一个，Riccati 方程就会是矩阵方程

其中P 是未知数的n x n对称矩阵，A，B，Q及 R  是已知实系数矩阵。一般而言此方程式有许多的解，不过若有存在稳定解的话，希望可以找到稳定解。

若代数 Riccati 方程存在稳定解，求解器一般会设法找到唯一的稳定解。稳定解的意思是指用此解控制相关的 LQR 系统，可以使闭回路的系统稳定。

### 连续时间代数Riccati方程(CARE)

针对连续时间代数的Riccati方程(CARE)，控制规律为：

$$
K=R^{-1}B^TP
$$

带入 Riccati 方程

$$
A^TP+PA-PBK+Q=0
$$

闭回路递移矩阵为：

$$
A-BK=A-BR^{-1}B^TP
$$

稳定的充分必要条件是所有的特征值都有负的实部

### 离散时间代数Riccati方程(DARE)

针对离散时间代数的Riccati方程(DARE)

$$
K=-(R+B^TPB)^{-1}B^TPA
$$

带入 Riccati 方程

$$
P=A^TPA+A^TPBK+Q
$$

闭回路递移矩阵为：

$$
A-BK=A-B(R+B^TPB)^{-1}B^TPA
$$

稳定的充要条件是所有的特征值在复数平面的单位圆内

代数 Riccati 方程的解可以用 Riccati 方程的的迭代或是矩阵因式分解求得。离散时间问题的一种迭代方式是由有限时间问题下的动态 Riccati 方程，每一次迭代时，矩阵中的值都是从最终时间往前一段有限时间内的最佳解，若进行无限长的迭代。就会分敛到特定矩阵，是无限时间内的最佳解。

### 求解

**迭代法**

给定初始值 $K_0,P_0$ 使之满足上述 Riccati 方程，不断迭代，公式为

$$
CARE:A^TP_k+P_kA-P_kBK_k+Q=0\\DARE:P_k=A^TP_kA+A^TP_kBK_k+Q\\K_k=R^{-1}B^TP_{k-1}
$$

当 $K$ 和 $P$ 收敛时，得到最优控制策略，迭代一定次数，使得本次迭代的结果与上次的结果的差值在某个范围之内认为是收敛了，就是最终的解

**不变子空间法**

持续更新中...