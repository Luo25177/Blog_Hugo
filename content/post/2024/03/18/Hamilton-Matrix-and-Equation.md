---
title: "Hamilton Matrix and Equation"
date: 2024-03-18 16:35:40
categories: ["控制理论"]
tags: ["数理知识", "Hamilton"]
summary: "哈密尔顿矩阵和哈密尔顿方程定义和原理推导"
---

最优化问题求解中的汉密尔顿方程是最优控制方法解决动态优化问题的一阶必要条件

[汉密尔顿方程（Hamilton Equation）](https://zhuanlan.zhihu.com/p/436150206)

### 理论推导

首先看需要解决的问题

$$
max\int_{t_0}^{t_1}f(x,u)dt\\\\
\dot{x}=g(x,u)\\\\
x(t_0)=x_0\\\\
x(t_1):free
$$

引入一个函数 $\lambda (t)$ 表示一个定义在 $t_0\leq t\leq t_1$ 上的连续的可微函数，那么对于任何满足上述约束的 $x(t),u(t)$ 有

$$
\int_{t_0}^{t_1}f(x,u)dt=\int_{t_0}^{t_1}[f(x,u)+\lambda g(x,u)-\lambda \dot{x}]dt
$$

对上式最右侧右侧部分进行分部积分可以得到

$$
-\int_{t_0}^{t_1}\lambda \dot{x}dt=-\lambda(t_1)x(t_1)+\lambda (t_0)x(t_0)+\int_{t_0}^{t_1}x\dot{\lambda}dt
$$

将上式带回原来的方程

$$
\int_{t_0}^{t_1}f(x,u)dt=\int_{t_0}^{t_1}[f(x,u)+\lambda g(x,u)+\dot{\lambda} x]dt-\lambda(t_1)x(t_1)+\lambda (t_0)x(t_0)
$$

在最优控制方法中，可以发现，只要确定了控制，系统状态自然就被确定了，这个是由函数 $g(x,u)$ 确定，所以问题变为求解最优的控制 $u^\star $

引入一个扰动项函数 $h(t)$ 是任意的但是人为给定，令

$$
u=u^\star +ah
$$

并且定义函数 $y(t,a)$ 为关于 $u^\star +ah$ 的状态变量，并且满足

$$
y(t,0)=x^\star \\\\
y(t_0,a)=x_0
$$

之前的目标函数可以写作为

$$
\int_{t_0}^{t_1}f(y(t,a),u^\star +ah)dt
$$

并且得到

$$
J(a)=\int_{t_0}^{t_1}f(y,u^\star +ah)dt=\int_{t_0}^{t_1}[f(y,u^\star +ah)+\lambda g(y,u^\star +ah)+\dot{\lambda} y]dt-\lambda(t_1)y(t_1,a)+\lambda (t_0)y(t_0,a)
$$

由于 $u^\star $ 是最优控制变量，所以该函数在 $a=0$ 处取得最大值，也就是 $\dot{J}(0)=0$，也就是

$$
\dot{J}(0)=\int_{t_0}^{t_1}[(f_x+\lambda g_x+\dot{\lambda})y_a+(f_u+\lambda g_u)h]dt-\lambda (t_1)y_a(t_1,0)
$$

由于 $y(t_0,a)$ 是常数，所以 $y_a(t_0,0)=0$

上式中难以确定的是 $y_a$ ，因为不知道 $y(t,a)$ 的具体形式，但是可以通过构造 $\lambda$ 来消去含有 $y$ 的项，定义

$$
\dot\lambda=-[f_x(x^\star ,u^\star )+\lambda g_x(x^\star ,u^\star )]\\\lambda(t_1)=0
$$

所以因此可以得到

$$
\int_{t_0}^{t_1}-[f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )]hdt=0
$$

并且对于任意连续的 $h$ 都成立，所以令

$$
h=f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )
$$

得

$$
\int_{t_0}^{t_1}-[f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )]^2dt=0
$$

即

$$
f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )=0~~t_0\leq t\leq t_1
$$

也就是说，如果 $u^\star ,x^\star $ 是最优解，存在一个连续函数 $\lambda(t)$ ，他们同时满足

$$
State~Equation:\\\dot{x}=g(x,u)\\x(t_0)=x_0\\Multiplier~Equation:\\\dot\lambda=-[f_x(x^\star ,u^\star )+\lambda g_x(x^\star ,u^\star )]\\\lambda(t_1)=0\\Optimality~Equation:\\f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )=0
$$

这就是汉密尔顿方程，普遍的表达形式为

$$
H(t,x(t),u(t),\lambda(t))=f(t,x,u)+\lambda g(t,x,u)
$$

并且方程满足

$$
\frac{\partial H}{\partial u}=f_u+\lambda g_u=0\\\\
-\frac{\partial H}{\partial x}=-(f_x+\lambda g_x)=\dot\lambda\\\\
\frac{\partial H}{\partial \lambda}=g=\dot{x}
$$

求解：

$$
State~Equation:\\\\
\dot{x}=g(x,u)\\\\
x(t_0)=x_0\\\\
Multiplier~Equation:\\\\
\dot\lambda=-[f_x(x^\star ,u^\star )+\lambda g_x(x^\star ,u^\star )]\\\\
\lambda(t_1)=0\\\\
Optimality~Equation:\\\\
f_u(x^\star ,u^\star )+\lambda g_u(x^\star ,u^\star )=0
$$

根据 $Optimality~Equation$ 可以得到 $u$ 关于 $x$ 和 $\lambda$ 的方程，然后带入 $Multiplier~Equation$ 和 $State~Equation$ 中可以得到两个微分方程，并且注意两个边界条件，这就可以解出两个微分方程，从而求解

对于方程最大值问题，还要求解 $H_{uu}(t,x^\star ,u^\star ,\lambda)\leq 0$，对于最小值，需要保证 $H_{uu}(t,x^\star,u^\star,\lambda)\geq 0$

**求解例子**

对于一个系统

$$
\dot{x}=Ax+Bu\\y=Cx\\J=\frac{1}{2} x(t_f)^TSx(t_f)+\frac{1}{2}\int_{0}^{t_f}{[x^TQx+u^TRu]}dt
$$

求解 $min~J$ 相当于求解 $min~\int_{0}^{t_f}{[x^TQx+u^TRu]}dt$

所以引入 $hamilton$ 函数

$$
H(t,x,u)=x^TQx+u^TRu+\lambda^T(Ax+Bu)
$$

$$
\frac{\partial H}{\partial u}=(R+R^T)u+B^T\lambda=0\\\\
\Downarrow\\\\
u=-(R+R^T)^{-1}B^T\lambda
$$

$$
-\frac{\partial H}{\partial x}=-(f_x+\lambda g_x)=-[(Q+Q^T)x+A^T\lambda]=\dot\lambda\\\\
\frac{\partial H}{\partial \lambda}=g=Ax+Bu=Ax-B(R+R^T)^{-1}B^T\lambda=\dot{x}
$$

由于我们设定 $u=-Kx$，所以可以知道 $\lambda=Px$，并且 $-(R+R^T)^{-1}B^TP=K$

带入上述的方程得到

$$
\dot\lambda=-[(Q+Q^T)x+A^T\lambda]=-(Q+Q^T+A^TP)x\\\\
\dot{x}=Ax-B(R+R^T)^{-1}B^T\lambda=[A-B(R+R^T)^{-1}B^TP]x
$$

联立得到

$$
\dot{\lambda}=\dot{P}x+P\dot{x}=[\dot{P}+P(A-B(R+R^T)^{-1}B^TP)]x=-(Q+Q^T+A^TP)x
$$

所以得到

$$
\dot{P}+PA-PB(R+R^T)^{-1}B^TP+Q+Q^T+A^TP=0
$$

令 $Q_1=Q+Q^T,Q_2=R+R^T$，得到

$$
\dot{P}+PA-PBQ_2^{-1}B^TP+Q_1+A^TP=0
$$

最终得到一个 $Riccati$ 方程

## Hamiltonian Matrix

首先引入一种特殊的矩阵

$$
J=\begin{bmatrix}0&I\\-I&0\end{bmatrix}~~J\in R^{2n\times 2n}
$$

这个矩阵具有如下性质

$$
J^T=-J\\J^{-1}=J^T\\J^TJ=I_{2n}\\J^TJ^T=-I_{2n}\\J^2=-I_{2n}\\detJ=\pm1
$$

定义 $Hamiltonian~ matrix$

如果一个矩阵 $A=JS$，其中 $J$ 是上面的特殊矩阵， $S=S^T$，那么矩阵 $A$ 是一个 $Hamiltonian$ 矩阵

或者说，如果矩阵 $A$ 满足 $(JA)^T=JA$ ， $J$ 是上面的特殊矩阵，那么矩阵 $A$ 是一个 $Hamiltonian$ 矩阵

**性质**

1. $A,B\in H^n ⇒ A+B\in H^n$
2. $A\in H^n ⇒ \alpha A\in H^n$
3. $\begin{bmatrix}A&B\end{bmatrix}\in H^n⇒det\begin{bmatrix}A&B\end{bmatrix}=AB-BA$
4. 对于 $A\in H^n$，定义 $pA(x)$ 为矩阵 A 的特征多项式
    - $pA(x)=pA(-x)$
    - $if~pA(c)=0,c\in R,then~pA(-c)=pA(\overline{c})=pA(-\overline{c})=0$