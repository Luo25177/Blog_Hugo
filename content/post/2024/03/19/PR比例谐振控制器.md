---
title: "PR比例谐振控制器"
date: 2024-03-19 00:08:18
categories: ["控制理论"]
tags: ["比例谐振"]
summary: "PR比例谐振控制器"
---

[github仓库](https://github.com/Luo25177/modelControl)

[比例谐振控制算法分析 - 百度文库 (baidu.com)](https://wenku.baidu.com/view/bd6ad8d85022aaea998f0fdb.html?_wkts_=1718038799743)

## 前言

在传统的矢量控制系统中，广泛的采用了坐标变换技术，将三相静止坐标系下的电流电压等正弦量转化为同步旋转坐标系下的直流量，这个实现了简化系统，并且能够很好的使电机实现解耦控制。这么做的原因就是 PI 控制器无法对正弦量实现无静差控制，坐标变换简化了系统外环控制的设计，却造成内环结构复杂，设计困难。而且在电机运行中，电机的电感，电阻等电机参数会随着磁路的饱和，温度的升高而发生改变，从而使交叉耦合项不准确，进而使系统的控制精度下降。

PR 控制器可以实现对交流的无静差控制，将 PR 控制器用于网侧变换器的控制系统中，可以在两个相对静止的坐标系下对电流进行调节，可以简化控制过程中的坐标变换，消除电流 d，q 轴分量之间的耦合关系，并且可以忽略电网电压对系统的扰动作用。并且使用 PR 控制器更容易实现低次滤波补偿，这些都有利于简化系统的结构

### 傅里叶变换

$$
F(\omega)=\int_{-\infty}^{\infty}f(t)e^{-j\omega t}dt\\f(t)=\frac{1}{2\pi}\int_{-\infty}^{\infty}F(\omega)e^{j\omega t}d\omega
$$

### laplace 变换

$$
F(s)=\int_{-\infty}^{\infty}f(t)e^{-st}dt\\f(t)=\frac{1}{2\pi j}\int_{\sigma-j\infty}^{\sigma+j\infty}F(s)e^{st}ds
$$

### z 变换

$$
F(z)=\sum f[n]z^{-n}\\f[n]=\frac{1}{2\pi j}\textcircled{\int} F(z)z^{n-1}dz
$$

### 小结

上述三种变换可以利用如下映射关系实现传递函数之间的等价转换

$$
s=j\omega=\frac{1}{T}\ln z\\z=e^{sT}=e^{j\omega T}
$$

### 稳定性分析

· 连续型传递函数稳定的充要条件为，所有极点均位于 s 域左半平面；离散型传递函数稳定的充要条件为，所有极点均位于 z 域原点为圆心的单位圆内。对于连续传函离散化问题，s 域左半平面所有极点，经 $z=e^{sT}$ 映射后，均位于 z 域原点为圆心的单位圆内。因此，严格离散化前后，连续系统稳定，则离散系统稳定

## PR 控制器

### 介绍

PR 控制器即比例谐振控制器，由比例环节和谐振环节组成，可对正弦量实现无静差控制。理想 PR 控制器的传递函数如下

$$
G(s)=K_P+\frac{K_Rs}{s^2+w_0^2}
$$

其中

- $K_P$ 比例项系数
- $K_R$ 谐振项系数
- $w_0$ 谐振频率

PR 控制器中的积分环节又称为广义积分器，可以对谐振频率的正弦量进行幅值积分

### 小例子

对于同频的输入信号 $M\sin(\omega t+\varphi)$，对输入的信号进行拉普拉斯变换为

$$
L(M\sin(\omega t+\varphi))=M\cos(\varphi)\frac{\omega}{s^2+\omega^2}+M\sin(\varphi)\frac{s}{s^2+\omega^2}
$$

经过谐振项之后，表示变为（其中 $\omega_0=\omega$ ）

$$
\frac{K_Rs}{s^2+w_0^2}(M\cos(\varphi)\frac{\omega}{s^2+\omega^2}+M\sin(\varphi)\frac{s}{s^2+\omega^2})\\=K_RM(\cos(\varphi)\frac{\omega s}{(s^2+\omega^2)^2}+\sin(\varphi)\frac{s^2}{(s^2+\omega^2)^2})\\=K_RM(\cos(\varphi)\frac{\omega s}{(s^2+\omega^2)^2}+\frac{\sin(\varphi)}{2}(\frac{1}{s^2+\omega^2}+\frac{s^2-w^2}{(s^2+\omega^2)^2}))
$$

分别推导 $t\cos(\omega t)$ 和 $t\sin(\omega t)$ 的拉普拉斯变换为

$$
L(t\cos(\omega t))=\frac{s^2-\omega^2}{(s^2+\omega^2)}\\L(t\sin(\omega t))=\frac{2\omega s}{(s^2+\omega^2)^2}
$$

所以上式的拉普拉斯反变换为

$$
K_RM(\frac{\cos(\varphi)}{2}t\sin(\omega t)+\frac{\sin(\varphi)}{2}(t\cos(\omega t)+\frac{1}{\omega}\sin(\omega t)))
$$

整理之后得到

$$
\frac{K_RM}{2}((t\cos(\varphi)+\frac{\sin(\varphi)}{\omega})\sin(\omega t)+t\sin(\varphi)\cos(\omega t))
$$

由上述可知，当 $\varphi=0$ 时，输出信号为

$$
\frac{K_RM}{2}t\sin(\omega t)
$$

如图，与输入信号相位相同，幅值呈时间线性上升

![1718072679746.png](./1718072679746.png)

当 $\varphi=90$ 时，输出信号为

$$
\frac{K_RM}{2}(\frac{\sin(\omega t)}{\omega}+t\cos(\omega t))
$$

当时间稍大时，该值贴近于 $t\cos(\omega t)$ ，从整体上看，谐振器是对误差信号按时间递增的

![1718072827101.png](./1718072827101.png)

如下图所示，当 PR 控制器中的积分部分 $\frac{K_Rs}{s^2+w_0^2}$ 在谐振频率点处达到无穷大的增益，在这个频率点之外几乎没有衰减，因此，为了选择地补偿谐波，它可以作为一个直角滤波器，如图所示

![1718078434417.png](./1718078434417.png)

### 参数对控制器的影响

- $K_P$ 的影响

    选取 $K_R=1, \omega_0=100$

    ![1718080395579.png](./1718080395579.png)

    $K_P$ 越大，系统的幅值图就会约尖锐，幅值也越大，而相应的相位图就会在 $\omega_c$ 处就会变化更加急

- $K_R$ 的影响

    选取 $K_P=1, \omega_0=100$

    ![Untitled](./Untitled.png)

    可见 $K_R$ 会对幅值图的幅值有影响，且 $K_R$ 越大，幅值越大，而 $K_R$ 越大，使得相位图在 $\omega_0$ 处相位变化减缓

- $\omega_0$ 的影响

    选取 $K_P=1,K_R=1$

    ![1718081260701.png](./1718081260701.png)

    可见， $\omega_0$ 对幅值和相位也是有一定一影响的，并且 $\omega_0$ 越大，幅值越小，并且相位变化越剧烈

## 准 PR 控制器

### 介绍

与 PI 控制器相比，PR 控制器可以达到零稳态误差，提高有选择地抗电网电压干扰的能力，但是在实际的使用中，PR 控制器的实现存在两个问题

- 由于模拟系统元器件参数精度和数字系统精度的限制，PR 控制器不容易实现
- PR 控制器在非基频处的增益非常小，当电网频率产生偏移时，就无法有效抑制电网产生的谐波

因此，在 PR 的基础上，提出了一种容易实现的准 PR 控制器，既可以保持 PR 控制器的高增益，同时还可以有效减小电网频率偏移对逆变器输出电感电流的影响

准 PR 控制器传递函数为

$$
G(s)=K_P+\frac{2K_R\omega_cs}{s^2+2\omega_cs+\omega_0^2}
$$

控制器的伯德图如下所示，选择参数

$$
K_P=10\\K_R=100\\\omega_0=100\\\omega_c=1
$$

![1718079135424.png](./1718079135424.png)

### 参数设置

除了比例系数之外，准 PR 控制器还有 $K_R,\omega_c$ 两个参数，分析每个参数对控制器的影响，可先假设其它参数不变，然后观察各个参数对性能的影响

- $\omega_c=0$ ，调节 $K_R$

    ![1718089885780.png](./1718089885780.png)

    从图中可以看出，当 $K_R$ 参数增大时，控制器的峰值的增益也增大，而控制器的带宽没有变化

    ![1718083724228.png](./1718083724228.png)

    但是当有比例项的时候，带宽也不是一定的，会与 $K_P$ 项有关系

- $K_R=1$ ，调节 $\omega_c$

    ![1718089798564.png](./1718089798564.png)

    如图可知， $\omega_c$ 不仅影响控制器的增益，同时还影响控制器截止频率的带宽，随着 $\omega_c$ 的增加，控制器的增益和带宽都会增加（基频增益为 $K_R$ 不变），将 $s=j\omega$ 带入到传递函数

    $$
    G(j\omega)=\frac{2K_R\omega_cj\omega}{-\omega^2+2\omega_cj\omega+\omega_0^2}
    $$

    根据对带宽的定义， $|G(j\omega)|=\frac{K_R}{\sqrt(2)}$ 时，此时计算得到两个频率之差即为带宽，令 $|\frac{\omega^2-\omega_0^2}{2w_cw}|=1$ ，经过计算之后得到的准谐振控制器的带宽为 $\frac{\omega_c}{\pi}Hz$

    假设电网电压频率波动的允许的范围为 $\plusmn 0.8Hz$ ，则有 $\frac{\omega_c}{\pi}=1.6Hz$ ，也就是 $\omega_c=5Hz$

## 准 PR 控制器的离散化

模拟控制器的离散化有两种方式，分别为脉冲响应不变法和双线性变换法

### 脉冲响应不变法

PR 控制器的数字实现方法主要有两种，分别是采用 Z 算符和采用 $\delta$ 算符对其进行离散化

$$
G(s)=\frac{2K_R\omega_cs}{s^2+2\omega_cs+\omega_0^2}\\=\frac{2K_R\omega_cs}{(s-\frac{-2\omega_c+\sqrt{4\omega_c^2-4\omega_0^2}}{2})(s-\frac{-2\omega_c-\sqrt{4\omega_c^2-4\omega_0^2}}{2})}\\=\frac{2K_R\omega_cs}{(s+\omega_c-\sqrt{\omega_c^2-\omega_0^2})(s+\omega_c+\sqrt{\omega_c^2-\omega_0^2})}\\=\frac{A}{s+\omega_c-\sqrt{\omega_c^2-\omega_0^2}}+\frac{B}{s+\omega_c+\sqrt{\omega_c^2-\omega_0^2}}
$$

其中

$$
A=K_R\omega_c(1-\frac{\omega_c}{\sqrt{\omega_c^2-\omega_0^2}})\\B=K_R\omega_c(1+\frac{\omega_c}{\sqrt{\omega_c^2-\omega_0^2}})
$$

上式通过脉冲响应不变法转为 Z 变换，Z 变换实际上就是连续系统离散化的结果，在 matlab 中使用函数

```matlab
F = (2 * KR * WC * s) / (s ^ 2 + 2 * WC * s + W0 ^ 2);
c2d(F, t, 'z');
```

得到

$$
G(z)=\frac{Az}{z-e^{-(\omega_c-\sqrt{\omega_c^2-\omega_0^2})T}}+\frac{Bz}{z-e^{-(\omega_c+\sqrt{\omega_c^2-\omega_0^2})T}}\\=\frac{A}{1-z^{-1}e^{-(\omega_c-\sqrt{\omega_c^2-\omega_0^2})T}}+\frac{B}{1-z^{-1}e^{-(\omega_c+\sqrt{\omega_c^2-\omega_0^2})T}}
$$

设

$$
C=e^{-(\omega_c-\sqrt{\omega_c^2-\omega_0^2})T}\\D=e^{-(\omega_c+\sqrt{\omega_c^2-\omega_0^2})T}
$$

则有

$$
G(z)=\frac{Az}{1-z^{-1}C}+\frac{Bz}{1-z^{-1}D}\\=\frac{(A+B)-(AD-BC)z^{-1}}{1-(C+D)z^{-1}+CDz^{-2}}
$$

设 $Y=GX$ ，转为差分函数之后，该式子可以表示为

$$
y(n)=(C+D)y(n-1)-CDy(n-2)+(A+B)x(n)-(AD-BC)x(n-1)
$$

其中

$$
A=K_R\omega_c(1-\frac{\omega_c}{\sqrt{\omega_c^2-\omega_0^2}})\\B=K_R\omega_c(1+\frac{\omega_c}{\sqrt{\omega_c^2-\omega_0^2}})\\C=e^{-(\omega_c-\sqrt{\omega_c^2-\omega_0^2})T}\\D=e^{-(\omega_c+\sqrt{\omega_c^2-\omega_0^2})T}
$$

## 例子

对于一个一阶系统

$$
G(s)=\frac{1}{s+1}
$$

跟踪一个正弦信号量 $\sin(100\pi t)$ ，设计 PR 控制器和 PI 控制器进行对比，选取参数

$$
K_p=1000\\K_R=5000\\\omega_c=10\pi\\w_0=100\pi\\K_i=500
$$

最终得到的结果为

![1718252168267.png](./1718252168267.png)

可见 PR 控制器对于高频信号的跟踪要比 PI 控制器好，这是因为 PI 控制器中的 i 项需要累计误差才能有效果，而对于高频信号变化太快，导致累计误差很小而影响小
