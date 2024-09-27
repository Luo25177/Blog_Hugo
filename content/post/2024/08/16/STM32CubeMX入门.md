---
title: "STM32CubeMX入门"
date: 2024-08-16 22:46:34
categories: "单片机"
tags: ["stm32f4"]
summary: "stm32芯片的一些知识"
---

## 前言

cubemx是一个可以自动生成代码的单片机学习和使用的软件，非常方便，但同时配置时需要注意芯片以及外设的各个引脚的配置以及使用情况

[C语言：内存四区](https://blog.csdn.net/qq_45396672/article/details/119155585)

### 下载

直接在官网上下载

### 基础配置

需要完成的如下

1. 新建工程
2. 在右侧查找自己所使用的芯片型号，然后选择对应的芯片
3. 打开 RCC，并且设置为外部时钟源
4. 然后配置时钟树

### 一个需要注意的问题

在生成代码之后，在第一次编译时会有一个报错，就是找不到对应的启动文件，也就是 `startup` 开头的汇编文件（ `startup_stm32f407xx.s` ），是用来启动整个项目的，这个文件可以在 CubeMX 保存在 C 盘中的系统文件中找到

### 开启调试

在 SYS 中的第一个选项 `Debug` 中选择 `Serial Wire` 即可开启 SWD 的调试，如下

![1722497530169.png](./1722497530169.png)

## 时钟树的配置

### 时钟介绍

**时钟是单片机运行的基础，时钟信号推动单片机内各个部分执行相应的指令。**时钟系统就是CPU的脉搏，决定cpu速率，像人的心跳一样 只有有了心跳，人才能做其他的事情，而单片机有了时钟，才能够运行执行指令，才能够做其他的处理 (点灯，串口，[ADC](https://so.csdn.net/so/search?q=ADC&spm=1001.2101.3001.7020))，时钟的重要性不言而喻。

STM32本身十分复杂，外设非常多  但我们实际使用的时候只会用到有限的几个外设，使用任何外设都需要时钟才能启动，但并不是所有的外设都需要系统时钟那么高的频率，为了兼容不同速度的设备，有些高速，有些低速，如果都用高速时钟，势必造成浪费。并且，**同一个电路，时钟越快功耗越快，同时抗电磁干扰能力也就越弱**，所以较为复杂的MCU都是采用多时钟源的方法来解决这些问题。所以便有了STM32的时钟系统和时钟树

### 时钟系统的作用

- STM32 时钟系统主要目的时给相对独立的外设模块提供时钟，也是为了降低整个芯片的耗能。
- 系统时钟，是处理器运行时间基准（每一条机器指令一个时钟周期）
- 时钟是单片机运行的基础，时钟信号推动单片机内各个部分执行相应的指令
- 一个单片机内部提供多个不同的时钟，可以适用于更多场景
- 不同的功能模块会有不同的时钟上限，因此提供不同的时钟，也能在一个单片机内放置更多的功能模块。对不同模块的时钟增加开启和关闭功能，可以降低单片机的功耗
- STM32 为了低功耗，他将所有的外设时钟都设置为 disable(不使能)，用到什么外设，只要打开对应外设的时钟就可以， 其他的没用到的可以还是 disable(不使能)，这样耗能就会减少

### 系统时钟树图示

![c70c141b505e8884a77a94b6ce45d45d.png](./c70c141b505e8884a77a94b6ce45d45d.png)

### 四个时钟源和锁相环倍频输出时钟

图中可以看出 stm32 有四个时钟源和锁相环倍频输出时钟

1. HSE：高速外部时钟，高速外部时钟信号有两个时钟源
    - HSE 外部晶振/陶瓷谐振器
    - HSE 外部用户时钟
2. HSI：高速内部时钟，HSI 高速时钟信号由内部 16 MHz RC 振荡器生成，可以直接作为系统时钟或者用作 PLL 输入
3. LSE：低速外部时钟，LSE 晶振是 32.768 kHz 低速外部晶振或者陶瓷谐振器，可作为实时时钟外设 RTC 的时钟源来提供时钟/日历或者其它定时功能，具有低功耗且精度高的优点
4. LSI：低速内部时钟，LSI RC 可以作为低功耗时钟源在停机和待机模式下保持运行，供独立看门狗（IWDG）和自动唤醒单元（AWU）使用。时钟频率在 32 kHz 左右
5. PLL：倍频输出时钟
    - 主 PLL 由 HSE 或者 HSI 提供的时钟信号（通过选择器）。主 PLL 时钟计算方式为： $PLL=12MHz*N/(MP)=168MHz$ ，在其中外部晶振选择 12 MHz ，M=6，分频器分频系数 P=2，倍频器倍频系数 N=168，并且具有两个不同的输出时钟。
        - 第一个输出 PLLP 用于生成高速的系统时钟（最高 168 MHz）
        - 第二个输出 PLLQ 用于生成 USB OTG FS 的时钟（48 MHz），随机数发生器的时钟和 SDIO 时钟
    - 专用 PLL（PLLI2S）用于生成精确的时钟，从而在 I2S 接口实现高品质音频性能

### 锁相环原理

![pll.jpg](./pll.jpg)

**锁相环**（Phase-Locked Loop，PLL）是一种电子电路，主要用于频率锁定，即当输入信号的频率发生变化时，它能够自动调整自身产生的信号频率，使其保持与输入信号同步。PLL 通常由以下几个部分组成：

1. **鉴相器**（Phase Comparator）：专用于提供一种频率会在合理的范围内随输入信号的电压幅值变化而变化的输出信号。将输入信号与参考信号比较，产生一个误差电压，这个电压反映了两个信号之间的相位差。
2. **电压控制放大器**（VCO，Voltage-Controlled Oscillator）：根据鉴相器输出的误差电压调整自身的输出频率。当误差减小时，VCO会增加输出频率；反之，当误差增大时，VCO会减少输出频率。
3. **低通滤波器**（Loop Filter）：用于平滑并限制鉴相器输出的变化，帮助 PLL 达到稳定的锁定状态。
4. **电荷放大器：**有点像电路里的放大器

通过这种反馈机制，锁相环可以在宽频范围内跟踪和稳定频率，常被应用在通信系统、定时基准、雷达测距以及数字信号处理等领域。

**基本应用**

1. 时钟恢复
2. 偏移校正：在信号的传输过程中，工艺、温度、电压会影响时钟沿与数据采样窗口的延时，这一延时限制了数据发送的频率。可以在数据的接收端使用偏移校正 PLL 来消除这个延时，这样每一个采样触发器的时钟信号都与接收时钟保持相位匹配
3. 产生时钟：当今大多数电子系统中都包含有不同种类的处理器。典型情况下，外部为处理器提供一个较低的时钟频率，然后在处理器中使用PLL将其倍频或分频到处理器需要的时钟频率

### MCO 两个向外输出的时钟口

图中还可以看到两个向外输出的时钟口如下

- STM32 时钟信号输出 MCO1（PA8）：MCO1 用户可以通过可配置的预分频器（从 1 到 5）向 MCO1 引脚输出四个不同的时钟源
    - HSI 时钟
    - LSE 时钟
    - HSE 时钟
    - PLL 时钟
- STM32 时钟信号输出 MCO2（PC9）：MCO2 用户可以通过可配置的预分频器（从 1 到 5）向 MCO2 引脚输出四个不同的时钟源
    - HSE 时钟
    - PLL 时钟
    - 系统时钟
    - PLLI2S 时钟

一般来说这两个向外输出的时钟口是不工作的，就是灰色，只要在 RCC 设置中打开 Master Clock Output 即可启用。然后即可引出时钟输出

### FCLK 自由运行时钟

自由运行时钟。如果为了节省电量，STM32 可以进入低功耗模式中的停止模式时，AHB 总线会停止运行，同时 HCLK 会停止传输时钟脉冲从而所有连接到 HCLK 的外设都会停止工作。此时要想重新唤醒 STM32 就需要依赖外部中断了，此时 FCLK 的作用就显现出来了，它是连接在 AHB 的预分频器之下的，并不是位于 HCLK 下，所以当 HCLK 停止运作之后，FCLK 还是会继续进行工作的，为中断采样提供时钟信号

### CSS 时钟安全系统

时钟安全系统的作用就是当外部高速时钟出错时，会将时钟源切换到高速内部时钟，并且可以产生中断，从而可以进行紧急制动等紧急处理

### RTC 和 IWDG 时钟线

- **RTC：**这一路时钟线的时钟源只能是低速内部时钟，低速外部时钟和外部时钟的 128 分频
- **IWDG：**由内部低速时钟提供信号

### PTP 时钟

这是一个通过网络同步的时钟的一个协议，当硬件支持时，PTP 精度高达亚微秒

### 时钟配置

1. 对于 HSI，HSE，PLL 等时钟源的配置，没有专门的固件库函数，可以通过 `SystemInit` 函数来操作配置，该函数具体实现如下
    - 系统复位之后，先调用 `SystemInit` 函数，该函数会初始化系统时钟，设置 PLL
    - 打开 HSE，等待其稳定
    - 设置 AHB，APBx 等分频系数
    - 设置 HSE 为主的 PLL 时钟源，并且配置主 PLL 里的分频和倍频系数，然后产生 PLLCLK 并将其使能，并且选择系统时钟（SYSCLK）为 PLLCLK
2. 初始化之后
    - SYSCLK 系统时钟 = 168 MHz
    - AHB 总线时钟（HCLK=SYSCLK）= 168 MHz
    - APB1 总线时钟（PCLK1=SYSCLK/4）= 42 MHz
    - APB2 总线时钟（PCLK2=SYSCLK/2）= 84 MHz
    - PLL 主时钟 = 168 MHz
    
    初始化之后可以通过变量 `SystemCoreClock` 获取系统变量，如果 SYSCLK=168MHz，那么变量等于 168000000
    
    系统复位之后会先调用 `SystemInit` 函数，其次才是 `main` 函数，这一点在启动文件中有写
    
    ```nasm
    ...
    IMPORT  SystemInit
    IMPORT  __main
    	LDR     R0, =SystemInit
    	BLX     R0
    	LDR     R0, =__main
    	BX      R0
    	ENDP
    ...
    ```
    

### SysTick 系统定时器使用方法

`SysTick` 为定时器寄存器，24 位，只能递减，该寄存器存在于内核，嵌套在 NVIC 中，所有的 Cortex-M 内核单片机都具有该定时器。 `SysTick(uint32_t ticks)` 初始化函数存在于 `Core_m4.h` 中，计数器每计数一次的时间位 `1/SysTick` ，一般会设置 `SysTick=168MHz`

还有一些与 `SysTick` 相关的寄存器（位于 core_cm4.h 中）

```c
typedef struct
{
  __IOM uint32_t CTRL;                   /*!< Offset: 0x000 (R/W)  SysTick Control and Status Register */
  __IOM uint32_t LOAD;                   /*!< Offset: 0x004 (R/W)  SysTick Reload Value Register */
  __IOM uint32_t VAL;                    /*!< Offset: 0x008 (R/W)  SysTick Current Value Register */
  __IM  uint32_t CALIB;                  /*!< Offset: 0x00C (R/ )  SysTick Calibration Register */
} SysTick_Type;
/* SysTick Control / Status Register Definitions */
#define SysTick_CTRL_COUNTFLAG_Pos         16U                                            /*!< SysTick CTRL: COUNTFLAG Position */
#define SysTick_CTRL_COUNTFLAG_Msk         (1UL << SysTick_CTRL_COUNTFLAG_Pos)            /*!< SysTick CTRL: COUNTFLAG Mask */

#define SysTick_CTRL_CLKSOURCE_Pos          2U                                            /*!< SysTick CTRL: CLKSOURCE Position */
#define SysTick_CTRL_CLKSOURCE_Msk         (1UL << SysTick_CTRL_CLKSOURCE_Pos)            /*!< SysTick CTRL: CLKSOURCE Mask */

#define SysTick_CTRL_TICKINT_Pos            1U                                            /*!< SysTick CTRL: TICKINT Position */
#define SysTick_CTRL_TICKINT_Msk           (1UL << SysTick_CTRL_TICKINT_Pos)              /*!< SysTick CTRL: TICKINT Mask */

#define SysTick_CTRL_ENABLE_Pos             0U                                            /*!< SysTick CTRL: ENABLE Position */
#define SysTick_CTRL_ENABLE_Msk            (1UL /*<< SysTick_CTRL_ENABLE_Pos*/)           /*!< SysTick CTRL: ENABLE Mask */

/* SysTick Reload Register Definitions */
#define SysTick_LOAD_RELOAD_Pos             0U                                            /*!< SysTick LOAD: RELOAD Position */
#define SysTick_LOAD_RELOAD_Msk            (0xFFFFFFUL /*<< SysTick_LOAD_RELOAD_Pos*/)    /*!< SysTick LOAD: RELOAD Mask */

/* SysTick Current Register Definitions */
#define SysTick_VAL_CURRENT_Pos             0U                                            /*!< SysTick VAL: CURRENT Position */
#define SysTick_VAL_CURRENT_Msk            (0xFFFFFFUL /*<< SysTick_VAL_CURRENT_Pos*/)    /*!< SysTick VAL: CURRENT Mask */

/* SysTick Calibration Register Definitions */
#define SysTick_CALIB_NOREF_Pos            31U                                            /*!< SysTick CALIB: NOREF Position */
#define SysTick_CALIB_NOREF_Msk            (1UL << SysTick_CALIB_NOREF_Pos)               /*!< SysTick CALIB: NOREF Mask */

#define SysTick_CALIB_SKEW_Pos             30U                                            /*!< SysTick CALIB: SKEW Position */
#define SysTick_CALIB_SKEW_Msk             (1UL << SysTick_CALIB_SKEW_Pos)                /*!< SysTick CALIB: SKEW Mask */

#define SysTick_CALIB_TENMS_Pos             0U                                            /*!< SysTick CALIB: TENMS Position */
#define SysTick_CALIB_TENMS_Msk            (0xFFFFFFUL /*<< SysTick_CALIB_TENMS_Pos*/)    /*!< SysTick CALIB: TENMS Mask */
```

- `CTRL` `SysTick` 控制以及状态寄存器
- `LOAD` `SysTick` 重装载数值寄存器，保存着 `SysTick` 计数到 0 时重新装载到 `SysTick` 的值
- `VAL` `SysTick` 当前数值寄存器
- `CALIB` `SysTick` 校准数值寄存器
- `COUNTFLAG` 16 位寄存器，如果在上次读取本寄存器之后， `SysTick` 已经计数到 0，则该位置位
- `CLKSOURCE` 2 位，时钟源选择位
    - 0：AHB / 8
    - 1：处理器时钟 AHB
- `TICKINT` 1 位
    - 1： `SysTick` 倒数计数到 0 时产生 `SysTick` 一场请求
    - 0：数到 0 时无动作
- `ENABLE` `SysTick` 定时器的使能位
- `RELOAD` 当 `SysTick` 计数到 0 时，将被重新载入到 `SysTick` 的值

设置 `SysTick` 定时器，只需要调用 `SysTick_Config(uint32 ticks)`  函数，通过函数向 `SysTick` 中写入值，如果时钟源选择的是 `AHB=168MHz` ，那每递减一次的时间就是 1/168M，需要多少时间就设定多少初始值。当减到零时会触发异常请求中断。该函数具体实现如下

```c
__STATIC_INLINE uint32_t SysTick_Config(uint32_t ticks)
{
  if ((ticks - 1UL) > SysTick_LOAD_RELOAD_Msk)
  {
    return (1UL);
  }

  SysTick->LOAD  = (uint32_t)(ticks - 1UL);
  NVIC_SetPriority (SysTick_IRQn, (1UL << __NVIC_PRIO_BITS) - 1UL);
  SysTick->VAL   = 0UL;
  SysTick->CTRL  = SysTick_CTRL_CLKSOURCE_Msk |
                   SysTick_CTRL_TICKINT_Msk   |
                   SysTick_CTRL_ENABLE_Msk;
  return (0UL);
}
```

### 系统时钟通过 AHB 分频器给外设提供时钟

上述从左到右可以为：系统时钟——AHB 分频器——外设分频/倍频器——外设时钟设置

系统时钟的 `SYSCLK` 通过 AHB 分频器分频之后送给各模块使用，AHB 分频器可以选择 1，2，4，8，…，512 分频，其中 AHB 的分频器输出的时钟给 5 个模块使用

- 内核总线 AHB：送给 AHB 总线，内核，内存和 DMA使用的 HCLK 时钟
- `Tick` 定时器：通过 8 分频之后送给 Cortex 的系统定时器时钟
- I2S 总线：直接送给 Cortex 的空闲运行时钟 FCLK
- APB1 外设：送给 APB1 分频器。APB1 可以选择 1，2，4，8，16 分频，其输出一路供 APB1 外设使用（PCLK1 最大频率为 42MHz），另一路送给通用定时器使用。该倍频器可以选择 1 或者 2 倍频，时钟输出供定时器 2-7 使用
- APB2 外设：送给 APB2 分频器。APB2 分频器可选择 1，2，4，8，16 分频，其输出一路供 APB2 外设使用（PCLK2 最大频率 84MHz），另外一路送给高级定时器。该倍频器可以选择 1 或者 2 倍频，时钟输出供定时器 1 和 定时器 8 使用。另外还有一路输出供 ADC 分频器使用，分频后送给 ADC 模块使用。ADC 分频器可以选择 2，4，6，8 分频。

需要注意的是：如果 APB 预分频器的分频系数为 1，则定时器时钟频率（TIMxCLK）为 PCLKx。否则定时器频率将为 APB 的频率的两倍 $TIMxCLK = 2\times PCLKx$

### APB 对应的外设

![9415d6d866e86c309a243d980377ee14.png](./9415d6d866e86c309a243d980377ee14.png)

APB1 上连接的是低速外设，包括：电源接口，CAN，USB，I2C1，I2C2，USART2，USART3，UART4，UART5，SPI2，SPI3，通用定时器 2-5 和 12-14，基本定时器 6-7

APB2 上连接的是高速外设，包括：USART1，UASRT6，SPI1，ADC1，ADC2，ADC3，所有的普通 IO 口，第二功能的 IO 口，高级定时器 1 和 8，通用定时器 9-11

### RCC初始化

首先是 RCC 寄存器

```c
typedef struct
{
  __IO uint32_t CR;         /*!< RCC clock control register,                                  Address offset: 0x00 */
  __IO uint32_t PLLCFGR;    /*!< RCC PLL configuration register,                              Address offset: 0x04 */
  __IO uint32_t CFGR;       /*!< RCC clock configuration register,                            Address offset: 0x08 */
  __IO uint32_t CIR;        /*!< RCC clock interrupt register,                                Address offset: 0x0C */
  __IO uint32_t AHB1RSTR;   /*!< RCC AHB1 peripheral reset register,                          Address offset: 0x10 */
  __IO uint32_t AHB2RSTR;   /*!< RCC AHB2 peripheral reset register,                          Address offset: 0x14 */
  __IO uint32_t AHB3RSTR;   /*!< RCC AHB3 peripheral reset register,                          Address offset: 0x18 */
  uint32_t RESERVED0;       /*!< Reserved, 0x1C                                                                    */
  __IO uint32_t APB1RSTR;   /*!< RCC APB1 peripheral reset register,                          Address offset: 0x20 */
  __IO uint32_t APB2RSTR;   /*!< RCC APB2 peripheral reset register,                          Address offset: 0x24 */
  uint32_t RESERVED1[2];    /*!< Reserved, 0x28-0x2C                                                               */
  __IO uint32_t AHB1ENR;    /*!< RCC AHB1 peripheral clock register,                          Address offset: 0x30 */
  __IO uint32_t AHB2ENR;    /*!< RCC AHB2 peripheral clock register,                          Address offset: 0x34 */
  __IO uint32_t AHB3ENR;    /*!< RCC AHB3 peripheral clock register,                          Address offset: 0x38 */
  uint32_t RESERVED2;       /*!< Reserved, 0x3C                                                                    */
  __IO uint32_t APB1ENR;    /*!< RCC APB1 peripheral clock enable register,                   Address offset: 0x40 */
  __IO uint32_t APB2ENR;    /*!< RCC APB2 peripheral clock enable register,                   Address offset: 0x44 */
  uint32_t RESERVED3[2];    /*!< Reserved, 0x48-0x4C                                                               */
  __IO uint32_t AHB1LPENR;  /*!< RCC AHB1 peripheral clock enable in low power mode register, Address offset: 0x50 */
  __IO uint32_t AHB2LPENR;  /*!< RCC AHB2 peripheral clock enable in low power mode register, Address offset: 0x54 */
  __IO uint32_t AHB3LPENR;  /*!< RCC AHB3 peripheral clock enable in low power mode register, Address offset: 0x58 */
  uint32_t RESERVED4;       /*!< Reserved, 0x5C                                                                    */
  __IO uint32_t APB1LPENR;  /*!< RCC APB1 peripheral clock enable in low power mode register, Address offset: 0x60 */
  __IO uint32_t APB2LPENR;  /*!< RCC APB2 peripheral clock enable in low power mode register, Address offset: 0x64 */
  uint32_t RESERVED5[2];    /*!< Reserved, 0x68-0x6C                                                               */
  __IO uint32_t BDCR;       /*!< RCC Backup domain control register,                          Address offset: 0x70 */
  __IO uint32_t CSR;        /*!< RCC clock control & status register,                         Address offset: 0x74 */
  uint32_t RESERVED6[2];    /*!< Reserved, 0x78-0x7C                                                               */
  __IO uint32_t SSCGR;      /*!< RCC spread spectrum clock generation register,               Address offset: 0x80 */
  __IO uint32_t PLLI2SCFGR; /*!< RCC PLLI2S configuration register,                           Address offset: 0x84 */
} RCC_TypeDef;
```

相应的 RCC 初始化的流程如下

1. 将RCC寄存器重新设置为默认值
2. 打开外部高速时钟晶振HSE 
3. 等待外部高速时钟晶振工作
4. 设置AHB时钟
5. 设置高速AHB时钟
6. 设置低速速AHB时钟
7. 设置PLL
8. 打开PLL
9. 等待PLL工作
10. 设置系统时钟
11. 判断是否PLL是系统时钟 
12. 打开要使用的外设时钟

RCC 初始化就是系统时钟初始化，这个在 CubeMX 生成的代码中可以看到，也就是如下函数 ，这个函数的内部实现就是在 CubeMX 中配置的时钟树框图的配置

```c
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  /** Configure the main internal regulator output voltage
  */
  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE; // 将时钟源设置为外部高速晶振
  RCC_OscInitStruct.HSEState = RCC_HSE_ON; // 打开外部高速时钟晶振
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON; // 打开 PLL
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE; // 设置 PLL 时钟源
  RCC_OscInitStruct.PLL.PLLM = 6; // 分频 M
  RCC_OscInitStruct.PLL.PLLN = 168; // 主PLL的倍频
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2; // 分频 P
  RCC_OscInitStruct.PLL.PLLQ = 4; // 分频 Q
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  // 将系统时钟，AHB 总线时钟，APB1 总线时钟，APB2 总线时钟打开
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  // 时钟源为 PLL
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  // 设置 AHB 分频
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  // 设置 APB1 分频
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
  // 设置 APB2 分频
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;
  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5) != HAL_OK)
  {
    Error_Handler();
  }
}
```

### RCC 配置

![1722517373318.png](./1722517373318.png)

RCC 配置只需要选择 `High Speed Clock` ，这里将其设置为 `Crystal/Ceramic Resonator` 也就是晶体/陶瓷晶振，另外一个选项是 `BYPASS Clock Source` （旁路时钟）

### 时钟监视系统

STM32 还提供了一个时钟监视系统（CSS），用于监视高速外部时钟（HSE）的工作状态。倘若HSE失效，会自动切换（高速内部时钟）HSI作为系统时钟的输入，保证系统的正常运行。

## LED 配置

### 原理

LED 即发光二极管。当电子与空穴复合时能辐射出可见光，因而可以用来制成发光二极管。LED是正向导通，反向截止的，它在电路设计中的符号如下图所示

![2020040818075816.png](./2020040818075816.png)

其中

- 1 端接电源正极
- 2 端接电源负极

### 配置

在 CubeMX 中配置 LED 时，首先需要先看板子的手册或者 PCB 或原理图，找到连接 LED 的引脚，这里使用的是大疆的开发板 C 板，所以对应的 LED 灯的引脚就是 PH10-PH12，所以这里就会将该引脚的功能设置为输出模式，配置如下

- `GPIO output level` 初始化完成之后输出的电平状态为低
- `GPIO mode` 设置输出模式为推挽输出
- `GPIO Pull-up/Pull-down` 设置上拉或者下拉模式，这里设置为不上拉也不下拉
- `Maximum output speed` 最大输出速度，这里设置为低
- `User Label` 用户标签，这里没有设置，相当于是把引脚起了个别名

并且设置为推挽输出模式，输出速度设置为慢就可以，然后就完成设置，就可以点击右上方的 GENERATE CODE 生成代码了

![1722262601028.png](./1722262601028.png)

可以通过如下的指令来控制 LED 灯的亮灭

```c
// 写入引脚
void HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
// 读取引脚
GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
// 反转引脚
void HAL_GPIO_TogglePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
// 锁定引脚
HAL_StatusTypeDef HAL_GPIO_LockPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
```

### 呼吸灯

由于 LED 的引脚为 `PH10~PH12` ，而且对应着 `TIM5_CH1~TIM5_CH3` 。设置预分频为 `42-1` ，重载计数值为 `1000-1`

![1722355023742.png](./1722355023742.png)

**代码实现**

```c
void argb_led_show(uint32_t argb) {
  static uint8_t alpha;
  static uint16_t red, green, blue;

  alpha = (argb & 0xFF000000) >> 24;
  red = ((argb & 0x00FF0000) >> 16) * alpha;
  green = ((argb & 0x0000FF00) >> 8) * alpha;
  blue = ((argb & 0x000000FF) >> 0) * alpha;

  __HAL_TIM_SetCompare(&htim5, TIM_CHANNEL_1, blue);
  __HAL_TIM_SetCompare(&htim5, TIM_CHANNEL_2, green);
  __HAL_TIM_SetCompare(&htim5, TIM_CHANNEL_3, red);
}

#define RGB_FLOW_COLOR_CHANGE_TIME 1000
#define RGB_FLOW_COLOR_LENGHT 6
//blue-> green(dark)-> red -> blue(dark) -> green(dark) -> red(dark) -> blue
uint32_t rgb_flow_color[RGB_FLOW_COLOR_LENGHT + 1] = {0xFF0000FF, 0x0000FF00, 0xFFFF0000, 0x000000FF, 0xFF00FF00, 0x00FF0000, 0xFF0000FF};

void led_normal_task(void *argument) {
  uint16_t i, j;
  float delta_alpha, delta_red, delta_green, delta_blue;
  float alpha, red, green, blue;
  uint32_t argb;

  for (;;) {
    for (i = 0; i < RGB_FLOW_COLOR_LENGHT; i++) {
      alpha = (rgb_flow_color[i] & 0xFF000000) >> 24;
      red = ((rgb_flow_color[i] & 0x00FF0000) >> 16);
      green = ((rgb_flow_color[i] & 0x0000FF00) >> 8);
      blue = ((rgb_flow_color[i] & 0x000000FF) >> 0);

      delta_alpha = (float) ((rgb_flow_color[i + 1] & 0xFF000000) >> 24) - (float) ((rgb_flow_color[i] & 0xFF000000) >> 24);
      delta_red = (float) ((rgb_flow_color[i + 1] & 0x00FF0000) >> 16) - (float) ((rgb_flow_color[i] & 0x00FF0000) >> 16);
      delta_green = (float) ((rgb_flow_color[i + 1] & 0x0000FF00) >> 8) - (float) ((rgb_flow_color[i] & 0x0000FF00) >> 8);
      delta_blue = (float) ((rgb_flow_color[i + 1] & 0x000000FF) >> 0) - (float) ((rgb_flow_color[i] & 0x000000FF) >> 0);

      delta_alpha /= RGB_FLOW_COLOR_CHANGE_TIME;
      delta_red /= RGB_FLOW_COLOR_CHANGE_TIME;
      delta_green /= RGB_FLOW_COLOR_CHANGE_TIME;
      delta_blue /= RGB_FLOW_COLOR_CHANGE_TIME;
      for (j = 0; j < RGB_FLOW_COLOR_CHANGE_TIME; j++) {
        alpha += delta_alpha;
        red += delta_red;
        green += delta_green;
        blue += delta_blue;

        argb = ((uint32_t) (alpha)) << 24 | ((uint32_t) (red)) << 16 | ((uint32_t) (green)) << 8 | ((uint32_t) (blue)) << 0;
        argb_led_show(argb);
        osDelay(1);
      }
    }
  }
}
```

## 蜂鸣器配置

### 介绍

蜂鸣器是一种一体化结构的电子讯响器，采用直流电压供电蜂鸣器。主要分为压电式蜂鸣器和电磁式蜂鸣器两种类型。有源蜂鸣器自带了震荡电路，一通电就会发声，无源蜂鸣器则没有自带震荡电路，必须外部提供 2~5Khz 左右的方波驱动才能发声。

### 配置

这里选择引脚为 PD14，这里设置为

- `GPIO output level` 初始化完成之后输出的电平状态为低
- `GPIO mode` 设置输出模式为推挽输出
- `GPIO Pull-up/Pull-down` 设置上拉或者下拉模式，这里设置为下拉
- `Maximum output speed` 最大输出速度，这里设置为高速
- `User Label` 用户标签，这里没有设置，相当于是把引脚起了个别名

![1722263960231.png](./1722263960231.png)

然后即可点击生成代码，这里使用如下指令控制蜂鸣器

```c
void HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)
```

但是上述对蜂鸣器的配置是一般蜂鸣器的配置，对于大疆 C 板，蜂鸣器是需要 PWM 来控制的，所以上述生成的代码并不能使蜂鸣器发出响声

## 定时器配置

### 介绍

在 STM32F4 系列中共有 15 个定时器

- 基本定时器：TIM6，TIM7
- 通用定时器：TIM2~TIM5，TIM9~TIM14
- 高级定时器：TIM1，TIM8

其中，各自功能如下

- 基本定时器
    - 16位向上、向下、向上/下自动装载计数器
    - 16位可编（(可以实时修改）预分频器，计数器时钟频率的分频系数为1～65535之间的任意数值
    - 位于APB1总线上
    - 单纯的定时计数器
- 通用定时器
    - 16位向上、向下、向上/下自动装载计数器
    - 16位可编（(可以实时修改）预分频器，计数器时钟频率的分频系数为1～65535之间的任意数值
    - 四个独立通道（TIMx_CH1~TIMx_CH4），可用作
        - 测量输入信号的脉冲长度（输入捕获）
        - 输出比较
        - 单脉冲模式输出
        - PWM 输出（边缘或者中间对齐模式）
    - 支持增量编码器和霍尔传感器电路
    - 产生中断的原因
        - 更新：计数器向上/向下溢出，计数器初始化（通过软件或者内部/外部触发）
        - 触发事件（计数器启动，停止，初始化或者由内部/外部触发计数）
        - 输入捕获
        - 输出比较
    - 位于 APB1 总线上
- 高级定时器
    - 具有基本定时器和通用定时器的所有功能
    - 能控制交，直流电机所有功能
    - 输出 6 路互补带死区的信号，刹车功能等
    - 位于 APB2 总线上

### 定时器计数模式

- 向上计数模式：计数器从 0 开始计数到自动加载值（ `TIMx_ARR` ），然后重新从 0 开始计数并且产生一个计数器溢出事件
- 向下计数模式：计数器从自动装入的值（ `TIMx_ARR` ）开始计数直到 0，然后从自动装入的值重新开始，并且产生一个计数器向下溢出的事件
- 中央对齐模式（向上/向下计数）：计数器从 0 开始计数到自动装填的值 - 1，产生一个计数器溢出事件，然后向下计数到 1 并且产生一个计数器溢出事件，不断循环

### 计数时钟的选择

计数器时钟可由下列时钟源提供

- 内部时钟（TIMx_CLK）
- 外部时钟模式1：外部捕捉比较引脚（TIMx）
- 外部时钟模式2：外部引脚输入（TIMx_ETR），仅适用于 TIM2~TIM4
- 内部触发输入（ITRx）：使用一个定时器作为另一个定时器的预分频器

### 配置

这里选择 TIM2，设置时钟源为内部时钟，定时器的频率计算公式为 $f=\frac{f_1}{(PSC+1)(ARR+1)}$

这里将定时器设置为 1ms，并且由于该定时器是接入在 APB1 总线上的，所以 $f_1=42MHz$ ，同时选择预分频系数 $PSC=42-1$ ，选择自动重载数 $ARR=1000-1$ ，定时器频率为 $f=1000$ ，也就是 1ms

![1722328376974.png](./1722328376974.png)

然后打开定时器中断，如下

![1722329575570.png](./1722329575570.png)

使能 TIM2 全局中断，然后生成代码，这里需要注意，生成代码之后需要在程序中做一些修改才能使用

在代码中查找，可以看到在定时器中断函数 `TIM2_IRQHandler` （位于 `stm32f4xx_it.c` 中）中调用了 `HAL_TIM_IRQHandler` （位于 `stm32f4xx_hal_tim.c` 文件中）函数，用来处理定时器中断，而这些函数都存在于标准库或者霍尔库中，并且查看 `HAL_TIM_IRQHandler` 函数时，发现该函数会调用 `HAL_TIM_PeriodElapsedCallback` 函数，而这个函数在下面定义了一个虚函数版本。所以需要我们自定义该函数，所以在 `main.c` 文件中创建定义该函数，如下

```c
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
  if(htim->Instance == TIM2) {
    HAL_GPIO_TogglePin(GPIOH, GPIO_PIN_10); // 闪烁灯，用来验证
  }
}
```

做完这些之后，实际上代码依旧不能进入中断函数，所以这里就需要在 `main` 函数中将定时器计数中断打开，也就是如下

```c
  MX_TIM2_Init(); // TIM2 初始化
  /* USER CODE BEGIN 2 */
  HAL_TIM_Base_Start_IT(&htim2); // 中断开始计数
```

然后编译之后就能成功运行了

## PWM 配置

### 简介

脉冲宽度调制，是英文“Pulse Width Modulation”的缩写，简称**脉宽调制**，是利用微处理器的数字输出来对模拟电路进行控制的一种非常有效的技术，广泛应用在从测量、通信到功率控制与变换的许多领域中。 

### 原理

STM32F4 共有 15 个定时器：**高级定时器（TIM1、TIM8）；通用定时器（TIM2、TIM3、TIM4、TIM5、TIM9~TIM14）；基本定时器（TIM6、TIM7）**

而且 STM32 的每个通用定时器都有独立的 4 个通道可以用来作为：输入捕捉，输出比较，PWM 输出，单脉冲输出模式等。STM32 的定时器除了 TIM6 和 TIM7 之外，其它的定时器都可以产生 PWM 输出。其中高级定时器可以产生 7 路的 PWM 输出

下面是向上计数模式

![7381a5266713552b5880d294f3242deb.png](./7381a5266713552b5880d294f3242deb.png)

在 PWM 输出模式下，除了 CNT（计数器当前值），ARR（自动重装载值）之外，还多了一个值就是 CRRx（捕获/比较寄存器值）。当 CNT 小于 CRRx 时，TIMx_CHx 通道输出低电平，否则输出高电平

**具体工作流程**

- 定时器从 0 开始向上计数
- 当 0~t1 段，定时器计数器 TIMx_CNT 值小于 CCRx 值，输出低电平
- 在 t1~t2 段时，定时器计数器 TIMx_CNT 值大于 CCRx 值，输出高电平
- 当 TIMx_CNT 值达到 ARR 时，定时器溢出，重新开始向上计数

**原理总结**

- 每个定时器有四个通道,每一个通道都有一个捕获比较寄存器
- 将寄存器值和计数器值比较，通过比较结果输出高低电平，便可以实现脉冲宽度调制模式（PWM 信号），也就是在定时器开始之后，计数值也就是 `CNT` 累增，一直到计数值之后就会重新开始计数，在这个过程中 `CNT` 一直增加，输出也与 `CRRx` 比较来输出从而实现在一个周期之内，输出
- `TIMx_ARR` 寄存器确定 PWM 频率
- `TIMx_CCRx` 寄存器确定占空比

**PWM 工作模式**

- PWM 模式1
    - 在向上计数时，一旦 `TIMx_CNT<TIMx_CCR1` 时通道 1 为有效电平（ `OC1REF=1` ），否则为无效电平（ `OC1REF=0` ）
    - 在向下计数时，一旦 `TIMx_CNT>TIMx_CCR1` 时通道 1 为无效电平，否则为有效电平
- PWM 模式2
    - 在向上计数时，一旦 `TIMx_CNT<TIMx_CCR1` 时通道 1 为无效电平，否则为有效电平
    - 在向下计数时，一旦 `TIMx_CNT>TIMx_CCR1` 时通道 1 为有效电平，否则为无效电平。

在两种模式下 `TIMx_CNT` （计数器当前值）与 `TIMx_CCR1` （捕获/比较值）只是决定是有效电平还是无效电平

![1722352914606.png](./1722352914606.png)

有效电平可以是高电平也可以是低电平，这需要结合 `CCER` 寄存器的 `CC1P` 位的值来确定。如果为向上计数，且 `CCER` 寄存器的 `CC1P` 位为 0，则当 `TIMx_CNT<TIMx_CCR1` 时，输出高电平；如果 `CCER` 寄存器的 `CC1P` 位为 1，则当 `TIMx_CNT<TIMx_CCR1` 时，输出低电平（手册 P379）

![73ee22e396417b6aca6f715b6620a8d9.png](./73ee22e396417b6aca6f715b6620a8d9.png)

1. `CCR1` 寄存器：捕获/比较值寄存器：设置比较值。计数器值 `TIMx_CNT` 与通道 1 捕获比较寄存器 `CCR1` 进行比较,通过比较结果输出有效电平和无效电平
    - `OC1REF=0` 无效电平
    - `OC1REF=1` 无效电平
2. `TIMx_CCMR1` 寄存器： `OC1M[2:0]` 位：用于设置 PWM 模式
    - 110：PWM 模式 1
    - 111：PWM 模式 2
3. `CCER` 寄存器： `CC1P` 位：输入/捕获 1 输出极性。
    - 0：高电平为有效电平
    - 1：低电平为有效电平
4. `CCER` 寄存器： `CC1E` 位：输入/捕获1输出使能。
    - 0：关闭使能
    - 1：打开使能
5. 输出电平信号

### 参数状态

**占空比**

`duty circle = TIM3->CRR1 / ARR`

其中 `TIM->CRR1` 为用户设定值， `ARR` 为重装载值

**频率**

`Fpwm = Tclk / ((ARR + 1)*(PSC + 1))`

其中 `ARR` 为重装载值， `PSC` 为预分频值

### 配置

![1722356084358.png](./1722356084358.png)

由于板子上蜂鸣器的引脚是 PD14，而且这个引脚是接在 `TIM4_CH3` 上的，所以可以直接点击该引脚，并且设置为 `TIM4_CH3` 。然后再设置 `TIM4` 定时器，设置时钟源和通道 3 为 PWM 工作模式，并且设置其工作模式为 PWM1 号工作模式。设置预分频为 `1-1` ，设置 `ARR` 为 `21000-1` 表示 PWM 的频率为 4000 Hz（84000000/1/21000）。

然后生成代码，注意要在初始化时，将 `TIM4` 计数开始，并且启动对应定时器中对应的通道的 PWM

**HAL 库中 PWM 的控制函数**

```c
//PWM启动函数
HAL_StatusTypeDef HAL_TIM_PWM_Start(TIM_HandleTypeDef *htim, uint32_t Channel)
//PWM停止函数
HAL_StatusTypeDef HAL_TIM_PWM_Stop(TIM_HandleTypeDef *htim, uint32_t Channel)
//设置PWM占空比
__HAL_TIM_SET_COMPARE
//设置PWM周期
__HAL_TIM_SET_AUTORELOAD
// 设置预分频系数
__HAL_TIM_PRESCALER
```

设置占空比可以直接修改对应通道的 `CRRx` 寄存器即可

## 时钟输入捕获

### 简介

定时器输入捕获脉宽，测量时间。所谓捕获就是通过检测捕获通道上的边沿信号。在边沿信号发生跳变（比如上升沿/下降沿）的时候，将当前定时器的值（TIMx_CNT）存放到对应的通道的捕获/比较寄存器（TIMx_CCR）里面，完成一次捕获

### 原理

通用定时器在输入捕获（Input Capture, IC）模式下，当通道输入引脚出现指定电平跳变时，当前 CNT 的值将被锁存到 CCR 中，常被用于测量 PWM 波形的频率，占空比，脉冲间隔，电平持续时间等参数。每个高级定时器和通用定时器都拥有 4 个输入捕获通道，并且可配置为  PWMI 模式，用来同时测量频率和占空比。可配合主从触发模式，实现硬件全自动测量

可以用于测量脉冲宽度或者测量频率。下图是输入捕获测量高电平脉宽的原理

![e0452bbde77a2cf42de94a6fbcc66022.png](./e0452bbde77a2cf42de94a6fbcc66022.png)

图中的 t1~t2 时刻就是需要测量的高电平的时间。测量方法：设置定时器通道 x 为上升沿捕获，这样在 t1 时刻就会捕获到当前的 CNT 值，然后立即清零 CNT，并且设置通道 x 为下降沿捕获，这样到 t2 时就又会发生捕获事件，得到此时的 CNT 的值。然后根据定时器的计数频率，就可以算出 t1~t2 之间的时间，从而得到高电平脉冲的时间

在 t1~t2 之间，可能产生 N 次定时器溢出，这就要求我们对定时器的溢出做处理，防止高电平太长，导致数据不准确。对于上图中，由于 t1~t2 之间可能产生 N 次中断，所以我们需要对每次溢出中断做处理，记录其中溢出的次数 N，从而得到持续总时间 `T = N * ARR + CCRx2 - CCRx1`

就是通过检测 TIMx_CHx 上的边沿信号，在边沿信号发生跳变（比如上升沿/下降沿）的时候，将当前定时器的值（TIMx_CNT）存放到对应的通道的捕获/比较寄存器（TIMx_CCRx）里面，完成一次捕获。同时还可以配置捕获时是否触发中断/DMA  等

### 寄存器

- `TIMx_ARR` 自动重装载值
- `TIMx_PSC` 时钟预分频系数
- `OCxREF` 参考信号
- `TIMx_CCMR1` 捕获/比较模式寄存器，各段描述如下。通道方向通过配置相应的 `CCxS` 位进行定义。此寄存器的所有其他位再输入模式和输出模式下的功能均不同。对于任一给定位 `OCxx` 用于说明通道配置为输出时该位所对应的功能， `ICxx` 则用于说明通道配置为输入时该位对应的功能。所以需要注意同一个位在输入阶段和输出阶段都具有不同的涵义
    
    ![1723553716246.png](./1723553716246.png)
    
    - `OC2CE` 输出比较 2 清零使能
        - 0： `OC2Ref` 不受 `ETRF` 输入影响
        - 1： `ETRF` 输入上检测到高电平时， `OC2Ref` 立即清零
    - `OC2M[2:0]` 输出比较 2 模式
        - 这些位提供 OC2 和 OC2N 的输出参考信号 `OC2REF` 的行为。 `OC2REF` 为高电平有效，而 OC2 和 OC2N 的有效电平则取决于 CC2P 和 CC2NP 位
        - 000：冻结——输出比较寄存器 `TIMx_CCR1` 与计数器 `TIMx_CNT` 进行比较不会对输出造成任何影响
        - 001：将通道 2 设置为匹配时输出有效电平。当计数器 `TIMx_CNT` 与捕获/比较寄存器 2 `TIMx_CCR2` 匹配时， `OC1REF` 信号强制变为高电平
        - 010：将通道 2 设置为匹配时输出无效电平。当计数器 `TIMx_CNT` 与捕获/比较寄存器 2 `TIMx_CCR2` 匹配时， `OC2REF` 信号强制变为低电平
        - 011：翻转，当 `TIMx_CNT=TIMx_CCR2` 时， `OC2REF` 发生翻转
        - 100：强制变为无效电平， `OC1REF` 强制变为低电平
        - 101：强制变为有效电平， `OC1REF` 强制变为高电平
        - 110：PWM 模式 1，在递增计数模式下，只要 `TIMx_CNT<TIMx_CCR2` ，通道 2 便为有效状态，否则为无效状态。在递减计数模式下，只要 `TIMx_CNT>TIMx_CCR2` ，通道 2 便为无效状态（ `OC1REF=0` ），否则为有效状态（ `OC1REF=1` ）
        - 111：PWM 模式 2，在递增计数模式下，只要 `TIMx_CNT<TIMx_CCR2` ，通道 2 便为无效状态，否则为有效状态。在递减计数模式下，只要 `TIMx_CNT>TIMx_CCR2` ，通道 2 便为有效状态，否则为无效状态。
        - 在 PWM 模式 1 或 PWM 模式 2 下，仅当比较结果发生改变或输出比较模式由冻结模式切换到 PWM 模式时， `OCREF` 电平才会发生更改。
    - `OC2PE` 输出比较 2 预装载使能
        - 0：禁止与 `TIMx_CCR2` 相关的预装载寄存器。可随时向 `TIMx_CCR2` 写入数据，写入后将立即使用新值。
        - 1：使能与 `TIMx_CCR2` 相关的预装载寄存器。可读/写访问预装载寄存器。 `TIMx_CCR1` 预装载值在每次生成更新事件时都会装载到活动寄存器中
    - `OC2FE` 输出比较 2 快速使能：434
    - `CC2S[1:0]` 捕获/比较 2 选择
        - 00：CC2 通道配置为输出
        - 01：CC2 通道配置位输入，IC2 映射到 TI2 上
        - 10：CC2 通道配置为输入，IC2 映射到 TI1 上
        - 11：CC2 通道配置为输入，IC2 映射到 TRC 上。此模式仅在通过 TS 位（ `TIMx_SMCR` 寄存器）选择内部触发输入时有效
        - 只有当通道关闭时（ `TIMx_CCER` 中的 `CC2E=0` ），才可向 `CC2S` 位写入数据
    - `IC2F[3:0]` 输入捕获 2 滤波器，此位域可定义 `TI2` 输入的采样频率和适用于 `TI2` 的数字滤波器带宽。数字滤波器由事件计数器组成，每 N 个事件才视为一个有效边沿
        - 0000：无滤波器，按 fDTS 频率进行采样 1000：fSAMPLING=fDTS/8，N=6
        - 0001：fSAMPLING=fCK_INT，N=2
        - 1001：fSAMPLING=fDTS/8，N=8
        - 0010：fSAMPLING=fCK_INT，N=4
        - 0011：fSAMPLING=fCK_INT，N=8
        - 0100：fSAMPLING=fDTS/2，N=6
        - 0101：fSAMPLING=fDTS/2，N=8
        - 0110：fSAMPLING=fDTS/4，N=6
        - 0111：fSAMPLING=fDTS/4，N=8
        - 1010：fSAMPLING=fDTS/16，N=5
        - 1011：fSAMPLING=fDTS/16，N=6
        - 1100：fSAMPLING=fDTS/16，N=8
        - 1101：fSAMPLING=fDTS/32，N=5
        - 1110：fSAMPLING=fDTS/32，N=6
        - 1111：fSAMPLING=fDTS/32，N=8
    - `IC2PSC[1:0]` 输入捕获 2 预分频器，此位域定义 `CC2` 输入（ `IC1` ）的预分频比，只要 `CC2E=0` （ `TIMx_CCER` 寄存器），预分频器便立即复位
        - 00：无预分频器，捕获输入上每检测到一个边沿便执行捕获
        - 01：每发生 2 个事件便执行一次捕获
        - 10：每发生 4 个事件便执行一次捕获
        - 11：每发生 8 个事件便执行一次捕获
- `TIMx_CCER` 捕获/比较使能寄存器，此处使用到这个寄存器的最低 2 位
    
    ![1723603872709.png](./1723603872709.png)
    
    - `CC1P` 捕获比较 1 的输出极性
        - CC1 通道被设置为输出
            - 0：OC1 高电平有效
            - 1：OC1 低电平有效
        - CC1 通道被设置为输入： `CC1NP, CC1P` 位可针对触发或捕获操作选择 `TI1FP1` 和 `TI2FP1` 的极性
            - 00：非反相/上升沿触发。电路对 `TIxFP1` 上升沿敏感（在复位模式、外部时钟模式或触发模式下执行捕获或触发操作）， `TIxFP1` 未反相（在门控模式或编码器模式下执行触发操作）
            - 01：反相/下降沿触发。电路对 `TIxFP1` 下降沿敏感（在复位模式、外部时钟模式或触发模式下执行捕获或触发操作）， `TIxFP1` 反相（在门控模式或编码器模式下执行触发操作）
            - 10：保留，不使用此配置
            - 11：非反相/上升沿和下降沿均触发。电路对 `TIxFP1` 上升沿和下降沿都敏感（在复位模式、外部时钟模式或触发模式下执行捕获或触发操作）， `TIxFP1` 未反相（在门控模式下执行触发操作）。编码器模式下不得使用此配置。
    - `CC1E` 捕获/比较 1 输出使能
        - CC1 通道配置为输出
            - 0：关闭，OC1 未激活
            - 1：开启，在相应输出引脚上输出 OC1 信号
        - CC1 通道配置为输入，此位决定了是否可以实际将计数器值捕获到输入/捕获寄存器中
            - 0：禁止捕获
            - 1： 使能捕获
    - `CC1NP` 捕获/比较 1 输出的极性
        - CC1 通道被配置为输出，CC1NP 必须保持清零
        - CC1 通道被配置为输入，与 CC1P 配合使用，用以定义 `TI1FP1, TI2FP1` 的极性
    - 保留位：必须保持复位值
- `TIMx_DIER` DMA/中断使能寄存器
    
    ![1723616508309.png](./1723616508309.png)
    
    - 保留位：必须保持复位值
    - `TDE` 触发 DMA 请求使能
        - 0：禁止触发 DMA 请求
        - 1：使能触发 DMA 请求
    - `CCxDE` 捕获/比较 x DMA 请求使能
        - 0：禁止 CCx DMA 请求
        - 1：使能 CCx DMA 请求
    - `UDE` 更新 DMA 请求使能
        - 0：禁止更新 DMA 请求
        - 1：使能更新 DMA 请求
    - `TIE` 触发信号 `TRGI` 中断使能
        - 0：禁止触发信号中断
        - 1：使能触发信号中断
    - `CCxIE` 捕获/比较 x 中断使能
        - 0：禁止 CCx 中断
        - 1：使能 CCx 中断
    - `UIE` 更新中断使能
        - 0：禁止更新中断
        - 1：使能更新中断
- `TIMx_CCRx` 捕获比较寄存器
    
    ![1723617530626.png](./1723617530626.png)
    
    - `CCRx[31:16]` 捕获比较 x 的高 16 位
    - `CCRx[15:0]` 捕获比较 x 的低 16 位
        - 如果通道 CCx 配置为输出： `CCRx` 是捕获/比较寄存器 x 的预装载值。如果没有通过 `TIMx_CCMR` 寄存器中的 `OC1PE` 位来使能预装载功能，写入的数值会被直接传输到当前寄存器中。否则只在发生更新事件时生效（拷贝到实际起作用的捕获/比较寄存器 x）
        - 如果通道 CCx 配置为输入： `CCRx` 是上一个输入捕获 x 事件发生时的计数器值

### 配置

![1723472293254.png](./1723472293254.png)

这里配置引脚为 PB0，选择 TIM3_CH1，然后将其设置为 Input Capture direct mode 直接输入捕获模式

设置参数如上图，为了保证获得输入脉冲准确，所以设置的比较精细

然后直接生成代码，生成代码之后还需要做一些初始化的配置捕获中断和定时器中断回调函数，函数介绍以及例子如下

```c
// 设置时钟的 CNT
__HAL_TIM_SET_COUNTER(__HANDLE__, __COUNTER__) 
// 开启输入捕获中断
HAL_StatusTypeDef HAL_TIM_IC_Start_IT(TIM_HandleTypeDef *htim, uint32_t Channel);
// 关闭输入捕获中断
HAL_StatusTypeDef HAL_TIM_IC_Stop_IT(TIM_HandleTypeDef *htim, uint32_t Channel);
// 捕获中断回调函数
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim);
// 定时器中断回调函数
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim);
// 设置上升沿/下降沿触发模式
__HAL_TIM_SET_CAPTUREPOLARITY(__HANDLE__, __CHANNEL__, __POLARITY__)
// - TIM_INPUTCHANNELPOLARITY_RISING：上升沿触发
// - TIM_INPUTCHANNELPOLARITY_FAILING：下降沿触发

// MX_TIM3_Init 中
/* USER CODE BEGIN TIM3_Init 2 */
// 开启计数
if(HAL_TIM_Base_Start_IT(&htim3) != HAL_OK) Error_Handler();
// 开启捕获中断
if(HAL_TIM_IC_Start_IT(&htim3, TIM_CHANNEL_3) != HAL_OK) Error_Handler();
/* USER CODE END TIM3_Init 2 */

// 实现函数
uint8_t dected = 0;
uint8_t n = 0;
float during_time = 0;
uint32_t CCR3 = 0;
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
	if(htim->Instance == TIM3) {
		if(!dected) {
			__HAL_TIM_SET_CAPTUREPOLARITY(&htim3, TIM_CHANNEL_3, TIM_INPUTCHANNELPOLARITY_FALLING);
			CCR3 = TIM3->CNT;
		}
		else {
			__HAL_TIM_SET_CAPTUREPOLARITY(&htim3, TIM_CHANNEL_3, TIM_INPUTCHANNELPOLARITY_RISING);
			during_time = (float) (n * 1000 * 84 + TIM3->CCR3 - CCR3) / 168000000;
			n = 0;
		}
		dected = !dected;
	}
}
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
	if(htim->Instance == TIM3) {
		if(dected) {
			++n;
		}
	}
}
```

## 定时器 PWM 输入模式

### 简介

为了方便用户使用，STM32 设置了专门的 PWM 输入捕获模式，在这种模式下，输入捕获更加快捷，相比于普通的输入捕获模式来说，该模式代码量很小，能够快速实现对 PWM 周期，占空比，频率的测量。但是在 STM32F405 芯片上只有 TIM8 和 TIM1 可以被配置为 PWM 输入模式

### 原理

该模式是输入捕获模式的一个特例，与输入捕获模式的区别如下

- 两个 `ICx` 信号被映射到同一个 `TIx` 输入
- 这两个 `ICx` 信号为边沿有效，但是极性相反
- 其中一个 `TIxFP` 信号被作为触发输入信号，而从模式控制器被配置为复位模式
- 选择 `TIMx_CCR1` 的有效输入：设置 `TIMx_CCMR1` 寄存器的 `CC1S=01` （选中 `TI1` ）
- 选择 `TI1FP1` 的有效极性（用来捕获数据到 `TIMx_CCR1` 中和清除计数器），置 `CC1P=0` （上升沿有效）
- 选择 `TIMx_CCR2` 的有效输入：设置 `TIMx_CCMR1` 寄存器的 `CC2S=10` （选中 `TI1` ）
- 选择 `TI1FP1` 的有效极性（用来捕获数据到 `TIMx_CCR2` ），置 `CC1P=1` （下降沿有效）
- 选择有效的触发输入信号：设置 `TIMx_SMCR` 寄存器中的 `TS=101` （选择 `TI1FP1` ）
- 配置从模式控制器为复位模式：设置 `TIMx_SMCR` 寄存器中的 `SMS=100`
- 使能捕获：设置 `TIMx_CCER` 寄存器中的 `CC1E=1` 且 `CC2E=1`

![1723710560023.png](./1723710560023.png)

### 配置

![1723722054235.png](./1723722054235.png)

这里配置将输入模式通道 1 设置为直接捕获输入，并且为上升沿捕获

将输入模式通道 2 配置为非直接捕获输入，配置为下降沿捕获

打开时钟捕获比较中断

![1723726084638.png](./1723726084638.png)

**代码使用**

```c
HAL_StatusTypeDef HAL_TIM_PWM_Start(TIM_HandleTypeDef *htim, uint32_t Channel); // 开启PWM
HAL_StatusTypeDef HAL_TIM_IC_Stop(TIM_HandleTypeDef *htim, uint32_t Channel); // 开启中断捕获
uint32_t HAL_TIM_ReadCapturedValue(const TIM_HandleTypeDef *htim, uint32_t Channel); // 读取时钟捕获值
```

**代码实现**

```c
// 启动 PWM 和 输入捕获
HAL_TIM_IC_Start_IT (&htim8, TIM_CHANNEL_1);
HAL_TIM_IC_Start_IT (&htim8, TIM_CHANNEL_2);

// 中断回调函数
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim) {
	// 中断来源判断
	if(htim->Instance == TIM8 && htim ->Channel == HAL_TIM_ACTIVE_CHANNEL_1) {
		// 获取寄存器的值
		Cap_val1 = HAL_TIM_ReadCapturedValue (&htim1 ,TIM_CHANNEL_1 );
		Cap_val2 = HAL_TIM_ReadCapturedValue (&htim1 ,TIM_CHANNEL_2 );
		// 计算占空比和频率
		if(Cap_val1 != 0) {
			Duty = (float )(Cap_val2+ 1)*100 / (Cap_val1 +1);
			Frequency = 72000000 / 72 / (float )(Cap_val1 +1);
		}
	}
}
```

## 单脉冲模式

### 介绍

单脉冲模式是 PWM 模式的一种特例，在这种模式下，计数器可以在一个激励信号的触发下启动，并且可以在一段可编程的延迟之后产生一个脉宽可编程的脉冲

可以通过从模式控制器启动计数器。可以在输出比较模式或 PWM 模式下生成波形。将 `TIMx_CR1` 寄存器中的 `OPM` 位置 1，即可选择单脉冲模式。这样，发生下一更新事件 UEV 时，计数器将自动停止

当然需要注意的是，只有当比较直与计数器的初始值不同时，才能正确产生一个脉冲。启动前，也就是定时器等待触发时，必须完成配置为 `CNT < CCRx < ARR` ，当然也要注意 `CCRx > 0`

如下图所示，单脉冲模式实际上就是产生一个周期的 PWM 输出之后，定时器更新的时候停止计数，然后就不会有下一周期的脉冲产生了，以此来实现单脉冲模式输出

![1723729212517.png](./1723729212517.png)

### 配置

![1723728683848.png](./1723728683848.png)

将 TIM1 的 CH2 配置为 PWM Generation CH2 模式，也就是 PWM 输出模式，使能 One Pulse Mode

![1723728784137.png](./1723728784137.png)

配置参数，设置 CH Polarity 为 Low。这里设置预分频系数和 `ARR` 如图中所示，然后设置 PWM 输出参数的 Pulse 为 50 也就是 `CRRx` 。剩下的分频系数，计数周期，输出比较值等都可根据实际需求进行设置，用于控制延时事件和脉冲宽度

## 串口配置

### 简介

**UART：通用异步收发传输器**（Universal Asynchronous Receiver/Transmitter)，通常称作 UART。它将要传输的资料在串行通信与并行通信之间加以转换。作为把并行输入信号转成串行输出信号的芯片， UART 通常被集成于其他通讯接口的连结上。

**USART：通用同步/异步串行接收/发送器，**(Universal Synchronous/Asynchronous Receiver/Transmitter) USART 是一个全双工通用同步/异步串行收发模块，该接口是一个高度灵活的串行通信设备。

串口接口通过三个引脚与其它设备相连。任何 USART 双向通信至少需要两个引脚：接受输入（RX）和数据输出（TX）

- RX：接收数据串行输入。通过过采样技术来区别数据和噪音，从而恢复数据。
- TX：发送数据输出。当发送器被禁止时，输出引脚恢复到它的I/O端口配置。当发送器被激活，并且不发送数据时， TX引脚处于高电平。在单线和智能卡模式里，此I/O口被同时用于数据的发送和接收。

串口发送接收有三种基本方式

- 轮询（阻塞模式）：
    - 阻塞发送：使用 `HAL_UART_Transmit` 函数，使用超时机制管理
    - 阻塞接受：使用 `HAL_UART_Receive` 函数，使用超时机制管理
- 中断模式：
    - 中断发送：使用 `HAL_UART_Transmit_IT` 函数
    - 中断接收：使用 `HAL_UART_Receive_IT` 函数
- DMA
    - DMA 发送：使用 `HAL_UART_Transmit_DMA` 函数
    - DMA 接收：使用 `HAL_UART_Transmit_DMA` 函数

### 流控

流控概念来源于 RS232 这个标准，在 RS232 中包含了串口与流控的定义。其中 RS 就是 Recommend Standard 的缩写。没有被委员会制定，所以不同厂商做 RS232 时标准不太一样

**存在意义**

当两台设备进行串口通讯时，它们对数据处理速度不同。如果接收端数据缓冲区满了就会丢失后续发送过来的数据。使用流控是为了当接收端数据处理能力饱和时就发出信号使得发送端停止发送，直到接收端处理量减少，发送继续发送的信号给发送端，发送端才会继续发送数据

**硬件流控**

硬件流控一般通过 CTS（Clear to Send）和 RTS（Request to Send）两个引脚实现，如下

![OIP-C.jpg](./OIP-C.jpg)

对于 USART1 给 USART2 发送数据时，USART1 会使得 RTS 引脚有效，表明其想要发送请求发送的信息给 USART2，而 USART1 会检测对应的来自于 USART2 的 CTS 引脚，直到 CTS 引脚有效才会真正开始发送数据。并且接下来在发送每个字节之前都会检测对应的 CTS 是否有效，如果有效才会继续传输数据，否则就不能再发送了。如果在这期间 USART2 的 CTS 引脚一直有效，那么当 USART1 发送结束之后，就会把 RTS 设置为无效，表示数据发送完毕了

**软件流控**

软件流控就是以一个特殊的字符或者报文来表示接收端已经不能接收新的数据了，基本的流程就是在接收端接收数据很多的时候或者主动发送给发送端一个特殊字符或者报文，当发送端接收到这个特殊字符之后就不能再发送数据了

软件流控很容易实现，所以渐渐地代替掉了硬件流控

**STM32 的硬件流控**

部分型号的 STM32 的 USART 具有硬件流控的功能，可以实现 RS232 的硬件流控。还有部分型号的 STM32 具有 RS485 的硬件流控功能。

### 阻塞模式

**发送函数**

```c
HAL_StatusTypeDef HAL_UART_Transmit(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
```

该函数的作用就是将数据通过 `UART` 发送到外部的设备或者其他设备，其中的参数含义如下

- `huart` 串口的句柄，表示要使用的串口外设
- `pData` 要发送的数据的首地址
- `Size` 要发送的数据的大小，以字节为单位
- `Timeout` 发送时的超时时长，单位为毫秒。如果在指定的时间内发送成功则返回 `HAL_OK` ，否则返回 `HAL_TIMEOUT` 。如果设置为 `HAL_MAX_DELAY` ，则处理器会一直等待直到数据发送完成再执行下一条语句

**接收函数**

```c
HAL_StatusTypeDef HAL_UART_Receive(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size, uint32_t Timeout);
```

作用是从串口接收数据，并将接收到的数据存储在指定的缓冲区中

- `huart` 要使用的串口句柄地址
- `pData` 要接收的数据缓冲区首地址
- `Size` 要接收的数据长度，以字节为单位
- `Timeout` 超时时间，如上

**配置**

这里选择串口 1，模式选择 `Asymchronous` 异步，下面的参数不需改变，直接生成代码即可

![1722497658060.png](./1722497658060.png)

波特率设置为 115200，字长设置为 8 位，停止位设置为 1 位，其他都按照默认参数即可。而且下面的数据方向一定要是 `Receive and Transmit` 表示发送和接收

**代码实现和测试**

发送：

```c
	uint8_t data[] = "hello";
  while (1)
  {		
	  HAL_UART_Transmit(&huart1, data, sizeof(data), 1000);
		HAL_Delay(1000);
  }
```

接收：

```c
  while (1)
  {
		if(HAL_UART_Receive(&huart1, data, sizeof(data), 1000) == HAL_OK)
			HAL_GPIO_WritePin(GPIOH, GPIO_PIN_10, GPIO_PIN_SET);
		HAL_Delay(500);
		HAL_GPIO_WritePin(GPIOH, GPIO_PIN_10, GPIO_PIN_RESET);
		HAL_Delay(1000);
  }
```

可以使用 USB 转串口与 vofa 来测试

### 中断模式

**发送函数**

```c
HAL_StatusTypeDef HAL_UART_Transmit_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
```

这个函数作用是启动 `UART` 传输并以非阻塞的方式发送一定数量的数据，参数如下

- `huart` 要使用的串口句柄地址
- `pData` 发送缓冲区的首地址，存放要发送的数据
- `Size` 要发送的缓冲区的长度，也就是要发送的字节数

**接收函数**

```c
HAL_StatusTypeDef HAL_UART_Receive_IT(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
```

这个函数用于通过中断接收串口发送的消息，其中参数如下

- `huart` 要使用的串口句柄地址
- `pData` 接收缓冲区的地址，存放接收到的数据
- `Size` 接收缓冲区的长度，也就是要接收到的字节数

中断模式的函数的前三个参数和阻塞的方式完全一致，只是没有超时时间管理，因为中断方式配置完成寄存器之后不会占用 CPU 了，会在接收或者发送完成之后触发中断，所以不需要时间管理

**配置**

只需要在上述串口配置完成的基础上，打开 NVIC Setting，勾选 Enable 即可

![1722511680604.png](./1722511680604.png)

**代码实现和测试**

首先在初始化之后，循环之前添加一个开启一次接收中断，表示接收完成 `sizeof(data)` 个字节之后会进入接收完成中断，也就是 `HAL_UART_RxCpltCallback` 函数中，然后在这个函数中执行中断发送任务，也就是接收完成之后执行发送

```c
HAL_UART_Receive_IT(&huart1, data, sizeof(data));

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
	if(huart->Instance == USART1) {
		HAL_UART_Transmit_IT(&huart1, data, sizeof(data));
	}
	HAL_UART_Receive_IT(&huart1, data, sizeof(data));
}
```

### DMA 模式

**介绍**

DMA （Direct Memory Access 直接访问内存）是一种计算机系统中用于数据传输的机制。允许数据在外设和内存之间传输而不通过 CPU，从而减轻了 CPU 的负担，提高数据传输的效率

**发送函数**

```c
HAL_StatusTypeDef HAL_UART_Transmit_DMA(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
```

用于利用 DMA 来发送数据，参数如下

- `huart` 要使用的串口句柄地址
- `pData` 发送缓冲区的首地址，存放要发送的数据
- `Size` 要发送的缓冲区的长度，也就是要发送的字节数

**接收函数**

```c
HAL_StatusTypeDef HAL_UART_Receive_DMA(UART_HandleTypeDef *huart, uint8_t *pData, uint16_t Size);
```

这个函数用于利用 DMA 来接收数据，将数据直接写入到 `pData` 中，其中参数如下

- `huart` 要使用的串口句柄地址
- `pData` 接收缓冲区的地址，存放接收到的数据
- `Size` 接收缓冲区的长度，也就是要接收到的字节数

**配置**

首先是配置 DMA，如下，需要点击 `Add` 来添加 DMA 接收或者发送。这里将 DMA 的模式设置为 `Circular` 表示循环接收数据，可以多次接收数据，而 `normal` 只能接收一次数据

![1722513180696.png](./1722513180696.png)

首先是必须要开启中断，也就是上述的中断发送中的中断使能，如下

![1722515577449.png](./1722515577449.png)

**代码实现和测试**

由于 DMA 接收结束之后也是会直接触发串口接收完成中断的，所以这里与上述的串口中断相似

```c
HAL_UART_Receive_DMA(&huart1, data, sizeof(data));

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
	if(huart->Instance == USART1) {
		HAL_UART_Transmit_DMA(&huart1, data, sizeof(data));
	}
	HAL_UART_Receive_DMA(&huart1, data, sizeof(data));
}
```

### 总结

在中断和 DMA 模式下，都需要在初始化完成之后开启一次中断或者 DMA 接收，并且在接收回调函数中重启一次中断或者 DMA 接收才能实现循环接收。

**串口配置时模式选择**

| Mode                         | 描述             | 硬件引脚     | 外设支持    |
| ---------------------------- | ---------------- | ------------ | ----------- |
| Asynchronous                 | 异步模式         | TXD，RXD     | USART，UART |
| Synchronous                  | 同步模式         | TXD，RXD，CK | USART       |
| Single Wire (Half-Duplex)    | 半双工单线模式   | TXD          | USART，UART |
| Multiprocessor Communication | 多处理器通讯模式 | TXD，RXD     | USART，UART |
| IrDA                         | 红外解码通信     | TXD，RXD     | USART，UART |
| LIN                          | 总线通信         | TXD，RXD     | USART，UART |
| SmartCard                    | 智能卡模式       | TXD          | USART，UART |
| SmartCard with Card Clock    | 带时钟智能卡模式 | TXD，CK      | USART       |

## CAN 配置

### 介绍

CAN 就是 Controller Area Network 控制器局域网，是 ISO 国际标准化的串行通信协议。

### 特点

- 多主控制：总线空闲时，所有单元都可发送消息，而且两个以上的单元同时开始发送消息时，根据标识符决定优先级，并且对各消息 ID 的每个位进行逐个比较。比较之后优先级高的可继续发送消息，反之则立即停止发送而进行接收工作
- 系统柔软性：连接总线的单元，没有类似的地址的信息，只有发送消息的 ID 的分别，所以在总线上添加单元时，已连接的其他单元的软硬件和应用层都不需改变
- 速度快，距离远：短距离（<40m）时最高 1Mbps，最远可以达到 10km（速率<5kbps）
- 具有错误检测，错误通知和错误恢复功能：所有单元都可以检测错误，检测出错的单元会立即通知其他单元，正在发送消息的单元一旦被检测到错误会强制结束当前的发送。强制结束发送的单元会不断反复地重新发送此消息直到发送成功为止。
- 故障封闭功能：CAN 可以判断出错误的类型时总线上暂时的数据错误（外部噪声）还是持续的数据错误（单元内故障，驱动器故障，断线）。所以当总线上发生持续的数据错误时，可以将引起此故障的单元从总线上隔离
- 连接节点多：CAN 总线是可以同时连接多个单元的总线。可连接的单元总线数量理论上是没有限制的，但是实际上可连接的单元数收到总线上的时间延迟以及电气负载的限制。降低通信速度，可连接的单元数增加提高通讯速度，则可连接的单元数减少

### 数据帧

数据帧是 CAN 通信中 5 中类型帧中最常用且最复杂的，数据帧由 7 个段组成

- 帧起始：表示数据帧开始的段。标准帧和扩展帧都是由1个位的**显性电平**表示帧起始。
- 仲裁段：表示该帧优先级的段。标准帧的格式与扩展帧格式在这里是有区别的
    - 标准帧：包括 `ID` 和 `RTR` ，标准帧一般为 11 位
    - 扩展帧：包含 `ID` ， `RTR` ， `SRR` 和 `IDE` ，扩展帧一般为 32 位，实际上是 29 位
    - `ID` ：高位在前，低位在后，设置的 ID
    - `RTR` ：远程请求位
        - 0：数据帧
        - 1：远程帧
    - `SRR` ：替代远程请求位，设置为 1
    - `IDE` ：标识符选择位
        - 0：标准标识符
        - 1：扩展标识符
- 控制段：表示数据的字节数以及保留位的段。由 6 个位构成，表示数据段的字节数，标准帧和扩展帧的控制段会有点不同，如图中所示
    - `r0, r1` ：保留位。必须以显性电平发送，但是接收可以是隐性电平
    - `DLC` ：数据长度码，范围 0~8 字节，表示接收或者发送的数据长度
    - `IDE` ：标识符选择位
        - 0：标准标识符
        - 1：扩展标识符
- 数据段：数据的内容，一帧可发送 0~8 个字节的数据，从最高位开始发送。标准帧和扩展帧在这个段的格式完全一样
- `CRC` 段：检查帧的传输错误的段。用于检查帧的传输错误，由 15 个位的 `CRC` 顺序和 1 个位的 `CRC` 界定符（用于分隔的位），标准帧和扩展帧在这个段的格式一致。 `CRC` 的值计算范围包括：帧起始，仲裁段，控制段和数据段，接收方以相同的算法计算 `CRC` 的值来进行比较，不一致会通报错误
- `ACK` 段：表示确认正常接收的段，此段用来确认是否正常接收，由 `ACK Slot` 和 `ACK` 界定符两个位组成。标准帧和扩展帧在这个段的格式也是相同的
    - 发送单元的 `ACK` 段会发送两个隐性位
    - 接受单元的 `ACK` 段接收到正确消息的单元在 `ACK Slot` 发送显性位，通知发送单元正常接收结束
    - 上述的两种就是 发送 `ACK` 和 返回 `ACK`
- 帧结束：表示数据帧结束的段，由7个位的隐性位组成。标准帧和扩展帧在这个段格式完全一样

![ef2b92439005f8c953457925221f2042.png](./ef2b92439005f8c953457925221f2042.png)

### CRC 检测

CRC 校验的基础就是多项式，生成多项式就是发送方和接收方约定的一个除数，发送方和接收方都使用这个相同的除数进行模 2 运算，结算结果相同则说明传输数据没问题，而如果计算结果不同则可能出现了问题，就是为了保证传输数据的可靠性

这里的 CAN 通信采取的是 CRC-15 的校验，多项式表示为

$$
X^{15}+X^{14}+X^{10}+X^8+X^7+X^4+X^3+X^0
$$

也就是设置的除数就是 `1100010110011001` ，然后根据需要传输的数据使用模 2 除法算出余数来作为 `CRC` 段的数据

例如，要传输的数据（也就是帧起始，仲裁段，控制段和数据段）如下

- 帧起始：1 显性电平
- 仲裁段：
    - ID：0x200
    - RTR：0
- 控制段：
    - IDE：0
    - r0：1
    - DLC：1
- 数据段：只有 8 位，为 `0x0A`

最后可以得到一串数字，为 `1 01000000000 0 0 1 1 00001010` 进行模 2 除法（[模 2 除法计算器](https://www.23bei.com/tool/744.html)）可以得到余数为 `0011010011101001` ，这个就是 `CRC` 段的值

### ACK 段分析

CAN总线是一种基于广播的通讯方式，为了保证总线上的每一个正常节点都能正确的接收到报文，报文的发送者要求每一个接收节点在报文发送结束前要作出应答，这就是 ACK 段存在的意义

在一帧报文中 ACK 段长度为 2 个位，包含应答间隙 `ACK Slot` 和应答界定符 `ACK Delimter` 

- `ACK Slot` 所有接收到匹配 `CRC` 序列的单元会在应答间隙期间用一个显性的位写入发送器的隐性位来做出回答
- `ACK Delimter` 它是 `ACK` 的第二个位，并且是一个必须是隐性的位，所以应答间隙被两个位所包含，就是 `CRC` 界定符位和 `ACK` 界定位

当一条消息发送出去之后，总线上并没有 ACK 应答，那么发送器就会发送一个错误标志，并且发送错误计数器自增 8，节点就会对报文进行重新发送，不断循环。在发送错误计数器累计到 128 时，由错误主动转为错误被动状态

**ACK 出错原因**

- 总线上只有一个有效节点，发送报文的节点在发送出一帧报文之后会检测总线上应答间隙的状态，如果检测到应答间隙为隐性位，表示该报文没有得到 `ACK` ，也就是发送失败，需要重新发送，直到该节点关闭
- 波特率不匹配或节点没有初始化，导致没有 `ACK`
- 总线线缆短路，断路，反接
- 高速 CAN 总线上接的节点不是高速 CAN 而是容错低速 CAN，导致不匹配

### 总线状态

CAN 通信控制根据 CAN_L 和 CAN_H 两个引脚上的电位差来判断总线电平。总线电平分为显性和隐形电平，总线的状态必定是其中之一。发送方通过使得总线电平发生变化来将消息传递

- 隐性电平：CAN_H 与 CAN_L 电平之差为 2V 左右
- 显性电平：CAN_H 与 CAN_L 电平之差为 0V

需要注意的是，只要有一个单元输出显性电平，那这个总线上就是显性电平，只有所有的单元都输出隐性电平，总线上才会使隐性电平。

另外在 CAN 总线的起止端都有一个 120Ω 的终端电阻，用来做阻抗匹配以减少回波反射

![5d24b76083e4c215483d35920405f1ca.png](./5d24b76083e4c215483d35920405f1ca.png)

当然 CAN_L 与 CAN_H 在隐性电平时也不是 0V，而是大约 2.5V

CAN 通信是以如下 5 种类型的帧来进行发送的

| 帧类型 | 用途                                         |
| ------ | -------------------------------------------- |
| 数据帧 | 用于发送单元向接收单元传送数据的帧           |
| 遥控帧 | 用于接收单元向相同 ID 的发送单元请求数据的帧 |
| 错误帧 | 用于当检测出错误时向其他单元通知错误的帧     |
| 过载帧 | 用于接收单元通知其尚未做好准备接收的帧       |
| 间隔帧 | 用于将数据帧及遥控帧与前面的帧分离开来的帧   |

### CAN 工作模式

CAN 有三种主要工作模式：**初始化，正常和睡眠。**硬件复位之后，CAN 进入睡眠模式以降低功耗，同时 CANTX 上的内部上拉电阻激活。软件将 `MCR` 寄存器的 `INRQ` 或 `SLEEP` 位置 1，以请求 CAN 进入初始化或睡眠模式。一旦进入该模式，CAN 即将 `MSR` 寄存器的 `INAK` 或 `SLAK` 位置 1，以确认该模式，同时禁止内部上拉电阻。如果 `INAK` 和 `SLAK` 均未置 1，则 CAN 将处于正常模式。进入正常模式之前，CAN 必须始终在 CAN 总线上实现同步。**为了进行同步，CAN 将等待 CAN 总线空闲**（即已监测到 CAN_RX 上的 11 个隐性位）

- **初始化模式**：当硬件处于初始化模式时，可以进行软件初始化
    - 进入初始化模式：设置 `MCR` 寄存器的 `INRQ` 为 1，请求 CAN 进入初始化模式，等待硬件对 `MSR` 寄存器的 `INAK` 置位来确认进入了初始化模式。进入初始化模式不会改变配置寄存器
    - 退出初始化模式：设置 `MCR` 寄存器的 `INRQ` 为 0，请求 CAN 退出初始化模式，等待硬件对 `MSR` 寄存器的 `INAK` 复位来确认退出了初始化模式
    - 初始化模式中：禁止报文发送和接收，CAN_TX 引脚输出隐性位。软件对 CAN 的初始化必须至少包括位时间特性（CAN_BTR）和控制（CAN_MCR）这2个寄存器。
- **正常模式**：一旦初始化完成，软件必须向硬件请求进入正常模式，这样才能在 CAN 总线上进行同步，并开始接收和发送
    - 进入正常模式：在初始化完成之后，软件应该让硬件进入正常模式，以便正常接收和发送报文。软件可以通过对 `MCR` 寄存器的 `INRQ` 位清零请求进入正常模式，然后要等待硬件对 `MSR` 寄存器的 `INAK` 复位来确认进入了正常模式。
    - CAN 进入正常模式之后，并与 CAN 总线上的数据传输实现同步后，即可参与总线活动。执行这一步需要等待出现一个由 11 个连续隐性位（总线空闲状态）组成的序列
- **睡眠模式**：为降低能耗功耗，CAN 具有低功耗模式，该模式下，CAN 时钟停止，但软件仍可访问 CAN 邮箱
    - 进入睡眠模式：软件将 `MCR` 寄存器的 `SLEEP` 位置位而发出请求之后即可进入该模式
    - 睡眠模式中：CAN 始终将停止，但是软件仍可访问 CAN 邮箱
    - 退出睡眠模式：软件通过设置 `MCR` 的 `INRQ` 位为 1 来请求进入初始化模式，则必须同时将 `SLEEP` 清零。软件将 `SLEEP` 清零之后或者检测到 CAN 总线活动时，CAN 即被唤醒。检测到 CAN 总线活动时，如果 `MCR` 寄存器的 `AWUM` 置位，硬件将通过清零 `SLEEP` 位来自动执行唤醒序列。如果 `AWUM` 位清零，在发生唤醒中断时，软件必须将 `SLEEP` 位清零才能退出睡眠模式
    - 如果使能唤醒中断，即使 CAN 自动执行唤醒序列，一旦检测到 CAN 总线活动，也会发生唤醒中断
    - 将 `SLEEP` 清零之后，一旦 CAN 与总线同步，就会退出睡眠模式。一旦硬件将 `SLAK` 清零，即会退出睡眠模式

### CAN 测试模式

可以通过 `CAN_BTR` 寄存器的 `SILM` 位和 `LBKM` 位来选择测试模式，这些位都必须在 CAN 处于初始化模式时进行配置。选择测试模式后，必须复位 CAN_MCR 寄存器中的 INRQ 位才能进入正常模式

- **静默模式**：可以通过将 `CAN_BTR` 寄存器的 `SILM` 位置位，将其置于静默模式。在该模式下可接受有效数据帧和有效遥控帧，但仅在 CAN 总线上发送隐性位，无法启动发送。如果 CAN 必须发送一个显性位（ACK 位、溢出标志、活动错误标志），该位将在内部被**改道发送**，以便 CAN 内核可以监视该显性位，但 CAN 总线可以保持隐性状态。静默模式可用于分析 CAN 总线上的流量，同时又不会因发送显性位（确认位、错误帧）对其造成影响
- **环回模式**：可以通过将 `CAN_BTR` 寄存器的 `LBKM` 位置位来设置为回环模式。该模式下，CAN 会将自身发送的消息作为接收的消息来处理并存储（如果这些消息通过了验收筛选）在接收邮箱中。该模式为自检功能提供。为了不受外部事件的影响，CAN 内核在环回模式下将忽略确认错误（在数据/远程帧的确认时隙不对显性位采样）
- **静默模式与环回模式组合**：可以通过将 `CAN_BTR` 寄存器中 `LBKM` 和 `SILM` 置位，将回环模式与静默模式组合起来。该模式可用于热自检。也就是 CAN 可以像在环回模式下进行检测，同时又不会影响到与 CAN_TX 和 CAN_RX 引脚相连的运行中的 CAN 系统。此模式下 CAN_RX 引脚与 CAN 断开连接，而 CAN_TX 保持隐性

### 错误分析

**错误状态寄存器 ESR**

![1352362566_9834.jpg](./1352362566_9834.jpg)

- `REC[7:0]` 接收错误计数器
- `TEC[7:0]` 发送错误计数器
- 保留位，必须保持复位
- `LEC[2:0]` 上一个错误代码
    - 000：无错误
    - 001：位置填充错误：通讯线缆上传输信号违反“位填充”规则时发生该错误
    - 010：格式错误：传输的数据帧与任何一种合法的帧格式不符合时发生的错误
    - 011：确认错误 ACK：发送节点在 `ACK` 段没有接送到应答信号时发生该错误
    - 100：隐性位错
    - 101：显性位错
    - 110：CRC 错误：发送节点计算得到的 CRC 值与接收到的 CRC 值不同时发送该结果
    - 111：由软件设置
- `BOFF` 总线关闭标志，此位由硬件在进入关闭状态时置位， `TEC` 上溢（超过 255）时，进入总线关闭状态，此时硬件将其置位
- `EPVF` 错误被动标志：达到错误被动极限（接收错误计数器或者发送错误计数器>127）时由硬件置位
- `EWGF` 错误警告标志：达到警告极限（接收错误计数器或发送错误计数器>95）时由硬件置位

**三种错误状态**

![fe093b32a65e1723b4d4e065870936bb.png](./fe093b32a65e1723b4d4e065870936bb.png)

- 发送错误 `TEC` （主动错误）：如果接收期间发生错误，该计数器将以 1 或者 8 递增，具体取决于 CAN 标准所定义的错误状态，每次成功接收，该计数器按 1 递减，如果其数值大于 128，则复位为 120，计数器值超过 127 时，控制器进入错误被动状态。如果接收计数器的值为 0，则保持为 0。如果在于 127，则接收计数器的值应设置为 119~127 之间的值
- 接收错误 `REC` （被动错误）：如果发送阶段发送节后发送一个错误标志时，错误计数器以 8 累增。当成功发送一帧报文之后，发送计数器应自减 1，直到为 0
- 离线：当发送错误向上溢出时（超过255），此节点进入离线状态。处于离线状态的节点不会对总线产生任何影响，它将不会发送消息帧，ACK，错误帧，过载帧等，至于会不会接收总线上的数据，取决于此节点的实现。当一个处于离线状态下的节点接收到 128 次连续 11 位隐性位时，将变成主动错误状态，且同时设置发送错误计数器和接收错误计数器为0

**离线恢复**

当 `TEC` 大于 255 时，CAN 进入离线状态，同时 `BOFF` 位被置位，在离线状态下的 CAN 无法接收和发送报文

根据 `MCR` 寄存器中的 `ABOM` 位的设置，CAN 可以自动或者在软件的请求下，从离线状态恢复到错误主动状态。在这两种情况下，CAN 必须等待一个 CAN 标准所描述的恢复过程，也就是 CAN_RX 引脚上检测到 128 次连续 11 个隐性位

- 如果 `ABOM=1` ，CAN 进入离线状态之后，就自动开启恢复过程
- 如果 `ABOM=0` ，CAN 软件必须先请求 CAN 进入然后再退出初始化模式，随后恢复过程才被开启。在初始化模式下，CAN 不会监视 CAN_RX 引脚的状态，这样就不能完成恢复过程。为了完成恢复过程 CAN 必须工作在正常模式下

### CAN 发送

**发送邮箱标识符寄存器**

![ce644b856af22c2a654967236da69dc6.png](./ce644b856af22c2a654967236da69dc6.png)

- `STID` 标准标识符
- `EXID` 扩展标志符
- `IDE` 标识符扩展，用于定义邮箱中消息的标识符类型
    - 0：标准标识符
    - 1：扩展标识符
- `RTR` 远程发送请求
    - 0：数据帧
    - 1：遥控帧
- `TXRQ` 发送邮箱请求。由软件置位，用于请求发送相应邮箱的内容。邮箱变为空之后，此位由硬件清零

**发送优先级**

- 标识符决定
    - 当有超过 1 个发送邮箱在挂号时，根据标识符**由小到大**来决定优先级，如果相等则邮箱号小的先发送
- 发送请求次序决定
    - 通过对 `MCR` 寄存器的 `TXFP` 位置位，可以把邮箱配置为**发送 FIFO。**在此模式下，发送优先级由请求次序决定

**发送位时序**

![ade9d895c00103ae7fdbb2d30bc2e04f.png](./ade9d895c00103ae7fdbb2d30bc2e04f.png)

**位速率**：由发送单元在非同步情况下发送的每秒钟的位数称为位速率。一个位一般可以分为如下四段

- **同步段（SS）**：多个连接在总线上的单元通过此段实现时序调整，同步进行接收和发送的工作。由隐性电平到显性电平的边沿或由显性电平到隐性电平的边沿最好出现在此段中
- **传播时间段（PTS）**：用于吸收网络上的物理延迟的段。所谓的网络的物理延迟指的是发送单元的输出延迟，总线上信号的传播延迟，接收单元的输入延迟。这个段的时间是以上各个延迟时间的和的两倍
- **相位缓冲段 1（PBS1）**：当信号边沿不包含于此段时，可在此进行补偿。由于各个单元以各自独立的时钟工作，细微的时钟误差会累计，PBS 段可吸收此误差。通过对缓冲段加减 SJW 来吸收误差
- **相位缓冲段 2（PBS2）**：如上
- **再同步补偿宽度（SJW）**：因时钟频率偏差，传送延迟等，各个单元有同步误差。SJW 为补偿此误差的最大值

这些段由 Time Quantum（时间段 Tq）的最小时间单位构成。

**总线仲裁**

同时有多个单元发送数据时，总线仲裁过程如下

![52bea4dc69352c3f8ef01846455f5f94.png](./52bea4dc69352c3f8ef01846455f5f94.png)

1. 总线空闲时，最先发送的单元获得发送优先权，一旦发送，其他单元无法抢占
2. 如果多个单元同时发送，则连续输出显性电平多的单元具有较高优先级
3. 仲裁对比时根据仲裁段来对比，仲裁段是位于帧起始之后的

**发送流程**

1. 选择一个空置的邮箱（ `TME` 寄存器为 1）
2. 设置标识符，数据长度和发送数据
3. 设置 `TIR` 寄存器的 `TXRQ` 位为 1，请求发送
4. 邮箱挂号，等待成为最高优先级
5. 预定发送，对于取消发送可以将对应 CAN 的 `TIR` 寄存器中对应邮箱的 `ABRQ` 标志位置位，即可取消发送 
6. 发送
7. 邮箱空置（成功或失败都会邮箱空置，除非设置发送失败重复发送）

### CAN 接收

每个 CAN 中有两个 `FIFO` ，每个 `FIFO` 中包含三个邮箱，这样设计允许CAN单元最多可以缓存六个接收到的报文。‌这种设计使得CAN单元能够有效地处理和存储接收到的数据，‌确保数据的完整性和可靠性

**FIFO 寄存器 CAN_RF0R/CAN_RF1R**

![ceb26f63273e737fcad282ca3d82dcbe.png](./ceb26f63273e737fcad282ca3d82dcbe.png)

- `RFOM` 释放 `FIFO` 释放邮箱，由软件置位用于释放 FIFO 的邮箱。当 `FIFO` 中至少有一条消息挂起时才能释放邮箱。当 `FIFO` 为空时，该位置位无用。如果 `FIFO` 中至少有两条消息挂起时，软件必须释放邮箱，才能访问下一条消息。每次邮箱释放之后，此位由硬件清零
- `FOVR` 当 `FIFO` 填满之后，如果接收到新的消息并且通过了过滤器，此位将由硬件置位，由软件清空
- `FULL` 当 `FIFO` 存储了三条消息之后，由硬件置位，由软件清空
- `FMP` 用于指示接收 `FIFO` 中挂起的消息数。硬件每向 `FIFO` 中存储一条新消息时， `FMP` 就会增加。软件每次将 `RFOM` 置位来释放出邮箱， `FMP` 就会减少
- 保留位：必须保持复位

需要注意的是

- `CAN_RF0R` 用于 `FIFO0` 的控制
- `CAN_RF1R` 用于 `FIFO1` 的控制

**接收流程**

1. 收到有效报文，存入到 `FIFO` 的一个邮箱，这个由硬件控制，不需要软件控制
2. 每个 `FIFO` 中最多有三个邮箱， `FIFO` 接收到的报文数可以从 `CAN_RFR` 寄存器的 `FMP[1:0]` 控制位读取，只要 `FMP` 不为 0，就可以从 FIFO 中读取收到的报文
3. 软件读取邮箱内容时，通过将 `CAN_RFR` 寄存器的 `RFOM` 位设置为 1，从而将邮箱释放
4. 当 `FIFO` 中的三个邮箱都是满的，下一个有效的报文就会导致溢出，并且这个报文会丢失
5. 启用 `FIFO` 锁定功能需要将 `CAN_MCR` 寄存器的 `RFLM` 位置位，那么邮箱满了之后新收到的报文就被丢弃，从 `FIFO` 中读取到的是最早收到的三个报文。否则 `FIFO` 中最后收到的报文会被新报文所覆盖，而新报文不会丢失

**接收中断**

当 `FIFO` 存入，存满，溢出报文时，分别会产生中断

**接收消息储存位置**

数据存在于 `RDLR` 和 `RDHR` 寄存器，标志 ID 和 `RTR` 储存在 `RIR` 中，数据数 `DLC` 和接收时通过的过滤器号 `FMI` 储存在 `RDTR` 中

### CAN 过滤器

**筛选器组 i 寄存器 CAN_FiRx**

![b9b24526ec727290185d84c58d2a2ef0.png](./b9b24526ec727290185d84c58d2a2ef0.png)

STM32 芯片中共有 28 个筛选器寄存器

- `FB[31:0]` 筛选器位，标识符，寄存器每一位用于指定预期标识符相应位的级别
    - 0：需要显性位
    - 1：需要隐性位
- 掩码：寄存器的每一位用于指定相关标识符寄存器的位是否必须与预期标识符的相应位匹配
    - 0：无关，不使用此位进行比较
    - 1：必须匹配，传入标识符的此位必须与筛选器相应标识符寄存器中指定的级别相同

每个筛选器组的 **CAN_FiRx** 都由2个32位寄存器构成，即 **CAN_FiR1** 和 **CAN_FiR2**。根据过滤器位宽和模式的不同设置，这两个寄存器的功能也不尽相同

**筛选器激活寄存器 CAN_FA1R**

![1722598496089.png](./1722598496089.png)

- 保留位：必须保持复位
- `FACTx` 筛选器激活。软件将此位置位可激活筛选器，要修改该筛选器寄存器必须将 `FACTx` 清零或者将 `FMR` 寄存器的 `FINIT` 置位
    - 0：未激活
    - 1：激活

**筛选器分配寄存器 CAN_FFA1R**

![1722598552361.png](./1722598552361.png)

只有当把 `FMR` 寄存器设置了筛选器初始化模式 `FINIT=1` 时，才能对此寄存器执行写操作

- 保留位：必须保持复位
- `FFAx` 筛选器 x 的筛选器 `FIFO` 分配，通过此筛选器的消息将存储在指定的 `FIFO` 中
    - 0：筛选器分配到 `FIFO0`
    - 1：筛选器分配到 `FIFO1`

**过滤器配置**

在对 CAN 过滤器组（模式，位宽，FIFO关联，激活和过滤器组）初始化之前，软件要对 `FMR` 寄存器的 `FINIT` 位置位，当 `FINIT` 被置位之后，报文的接收被禁止。对过滤器的初始化可以在非初始化模式之下进行。可以先对过滤器

不需要在初始化模式下进行过滤器初值的设置，但必须在它处在非激活状态下完成（相应的 `FACT` 位为 0）。而过滤器的位宽和模式的设置，则必须在初始化模式中进入正常模式前完成

### 配置

![1722606027699.png](./1722606027699.png)

这里打开 CAN1，直接使能 CAN1，然后配置参数。由于使用的是 STM32F407 的芯片，时钟树配置中，CAN1 存在于 APB1 总线上，并且时钟频率为 42MHz

配置时，预分频系数选择 7，相位缓冲段 1 设置为 4，相位缓冲段 2 设置为 1，所以就可以得到 CAN 通信的频率为 `42 / 7 / (4 + 1 + 1) = 1 MHz`  

然后下面的几个基础的参数分别为

- DBF：冻结调试，一般是当 STM32 芯片处于程序调试模式时才使用的，平时使用并不影响
    - `ENABLE` 设置 CAN 处于禁止收发的状态，仍然可以访问接收 FIFO 中的数据
    - `DISABLE` 设置 CAN 处于工作状态
- TTCM：时间触发模式
    - `ENABLE` 使能时间触发模式，它用于配置 CAN 的时间触发通信模式，在此模式下，CAN 使用它内部定时器产生时间戳，并把它保存在 CAN_RDTxR、CAN_TDTxR 寄存器中。内部定时器在每个 CAN 位时间累加，在接收和发送的帧起始位被采样，并生成时间戳。利用它可以实现 ISO 11898-4 CAN 标准的分时同步通信功能
    - `DISABLE` 禁用时间触发模式
- ABOM：自动离线管理
    - `ENABLE` 使能自动离线管理。当节点检测到它发送错误或接收错误超过一定值时，会自动进入离线状态，如果启用这个模式，那么会在进入离线状态之后，适当（等待总线空闲）的自动恢复
    - `DISABLE` 禁用自动离线管理
- AWUM：自动唤醒
    - `ENABLE` 启用自动唤醒功能，CAN 外设可以使用软件进入低功耗的睡眠模式，如果使能了这个自动唤醒功能，当 CAN 检测到总线活动的时候，会自动唤醒
    - `DISABLE` 禁用自动唤醒功能
- NART：自动重传
    - `ENABLE` 启用报文自动重传功能，设置这个功能后，当报文发送失败时会自动重传至成功为止。若不使用这个功能，无论发送结果如何，消息只发送一次
    - `DISABLE` 禁用报文自动重传功能
- RFLM：锁定模式
    - `ENABLE` 启用 FIFO 锁定模式，该功能用于锁定接收 FIFO。锁定后，当接收 FIFO 溢出时，会丢弃下一个接收的报文
    - `DISABLE` 不启用 FIFO 锁定模式，不锁定接收 FIFO，则下一个接收到的报文会覆盖原报文
- TXFP：报文发送优先级判定方法
    - `ENABLE` 报文发送优先级的判定方法，以报文存入发送邮箱的先后顺序来发送
    - `DISABLE` 按照报文 ID 的优先级来发送，ID 越小优先级越高

这里是开启环回模式进行测试，正常通信选 normal。然后生成代码

**相关函数**

```c
HAL_CAN_Start	//开启CAN通讯
HAL_CAN_Stop	//关闭CAN通讯
HAL_CAN_RequestSleep	//尝试进入休眠模式
HAL_CAN_WakeUp	//从休眠模式中唤醒
HAL_CAN_IsSleepActive	//检查是否成功进入休眠模式
HAL_CAN_AddTxMessage	//向 Tx 邮箱中增加一个消息,并且激活对应的传输请求
HAL_CAN_AbortTxRequest	//请求中断传输
HAL_CAN_IsTxMessagePending	//检查是否有传输请求在指定的 Tx 邮箱上等待
HAL_CAN_GetRxMessage	//从Rx FIFO 收取一个 CAN 帧
HAL_CAN_ActivateNotification // 使能中断函数
```

**中断**

- `CANx_SCE_IRQHandler` CAN 总线状态改变中断，通过打开这个中断，配合代码可以精确的监测 CAN 总线的故障情况，实际上就是检测 CAN 的 `ESR` 寄存器。只要CAN总线发送状态改变，就会触发中断。可以非常灵敏的检测CAN总线断线，短路等等故障
- `CAN1_TX_IRQHandler` CAN 发送完成中断
- `CAN1_RX0_IRQHandler` CAN 接收中断，用于 FIFO0
- `CAN1_RX1_IRQHandler` CAN 接收中断，用于 FIFO1

**代码配置**

CubeMX 生成的代码之后，需要在 CAN 初始化函数中开启 CAN 通信和使能 CAN 中断，如下

```c
HAL_CAN_Start(&hcan1);
HAL_CAN_ActivateNotification(&hcan1,CAN_IT_RX_FIFO0_MSG_PENDING);
```

同时也需要配置 CAN 的过滤器

```c
CAN_FilterTypeDef sFilterConfig;
// 32位的屏蔽位模式的过滤器
sFilterConfig.FilterActivation = ENABLE;
sFilterConfig.FilterBank = 0;
sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
sFilterConfig.FilterIdHigh = (id << 3) & 0xf0;
sFilterConfig.FilterIdLow = (id << 3) & 0x0f;
sFilterConfig.FilterMaskIdHigh = (mask_id << 3) & 0xf0;
sFilterConfig.FilterMaskIdLow = (mask_id << 3) & 0x0f;
HAL_CAN_ConfigFilter(&hcan1,&sFilterConfig);

// 16位的屏蔽位模式的过滤器
sFilterConfig.FilterActivation = ENABLE;
sFilterConfig.FilterBank = 1;
sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
sFilterConfig.FilterScale = CAN_FILTERSCALE_16BIT;
sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
sFilterConfig.FilterIdHigh = (id1 << 5);
sFilterConfig.FilterIdLow = (id2 << 5);
sFilterConfig.FilterMaskIdHigh = (mask_id1 << 5);
sFilterConfig.FilterMaskIdLow = (mask_id2 << 5);
HAL_CAN_ConfigFilter(&hcan1,&sFilterConfig);

// 32位的列表模式的过滤器
sFilterConfig.FilterActivation = ENABLE;
sFilterConfig.FilterBank = 1;
sFilterConfig.FilterMode = CAN_FILTERMODE_IDLIST;
sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
sFilterConfig.FilterIdHigh = ((id1 << 3) >> 16) & 0x0ff;
sFilterConfig.FilterIdLow = (id1 << 3) & 0x0ff;
sFilterConfig.FilterMaskIdHigh = (id2 << 3) & 0xf0;
sFilterConfig.FilterMaskIdLow = (id2 << 3) & 0x0f;
HAL_CAN_ConfigFilter(&hcan1,&sFilterConfig);

// 16位的列表模式的过滤器
sFilterConfig.FilterActivation = ENABLE;
sFilterConfig.FilterBank = 1;
sFilterConfig.FilterMode = CAN_FILTERMODE_IDLIST;
sFilterConfig.FilterScale = CAN_FILTERSCALE_16BIT;
sFilterConfig.FilterFIFOAssignment = CAN_FILTER_FIFO0;
sFilterConfig.FilterIdHigh = (id1 << 5);
sFilterConfig.FilterIdLow = (id2 << 5);
sFilterConfig.FilterMaskIdHigh = (id3 << 5);
sFilterConfig.FilterMaskIdLow = (id4 << 5);
HAL_CAN_ConfigFilter(&hcan1,&sFilterConfig);
```

其中

- `FilterActivation` CAN 过滤器使能状态
- `FilterBank` 过滤器号，两个 CAN 共享 28 个过滤器，也就是 0~27 号过滤器，但是实际上两个 CAN 分别配置的过滤器不共享。也就是 CAN1 使用了这个过滤器的位置之后，CAN2 就不能使用了
- `FilterMode` 过滤器工作模式
    - `CAN_FILTERMODE_IDMASK` 掩码模式，寄存器的每一位用于指定相关标识符寄存器的位是否必须与预期标识符的相应位匹配
    - `CAN_FILTERMODE_IDLIST` 列表模式，接收到的报文 ID 必须与设置的报文 ID 一致
- `FilterScale` 过滤器中 ID 的长度
    - `CAN_FILTERSCALE_32BIT` ID 长度为 32 位，设置之后下面的 ID 会由 `FilterIdHigh` 和 `FilterIdLow` 连接而成，相应的掩码也会由 `FilterMaskIdHigh` 和 `FilterMaskIdLow` 组成。这里就意味着使用扩展帧，扩展帧长度为 29 位，所以使用时，需要左移 3 位
    - `CAN_FILTERSCALE_16BIT` ID 长度为 16 位，设置之后就以同时设置 2 个掩码匹配规则，高低位分别对应作为掩码。这意味着使用标准帧，标准帧长度为 11 位，所以需要左移 5 位
- `FilterFIFOAssignment` 设置当前过滤器作用于哪一个 `FIFO`
- `FilterIdHigh` 和 `FilterIdLow` ID
- `FilterMaskIdHigh` 和 `FilterMaskIdLow` 掩码 ID

另外，一定需要重新接收到消息的回调函数

```c
void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan) {
	if (hcan->Instance == CAN1) {
		CAN_RxHeaderTypeDef RXHeader;
		uint8_t RXmessage[8];
		HAL_CAN_GetRxMessage(hcan,CAN_FILTER_FIFO0,&RXHeader,RXmessage);
	}
}
```

还有 CAN 的发送函数，示例如下

```c
void CAN_senddata(CAN_HandleTypeDef *hcan)
{
	CAN_TxHeaderTypeDef TXHeader;
	uint8_t TXmessage[8] = {1, 2, 3, 4, 5, 6, 7, 8}; 
	uint32_t pTxMailbox = 0;
	
	TXHeader.StdId=0x00000000;
	TXHeader.ExtId=0x12345000;
	TXHeader.DLC=8;
	TXHeader.IDE=CAN_ID_EXT;
	TXHeader.RTR=CAN_RTR_DATA;
	TXHeader.TransmitGlobalTime = DISABLE;
	HAL_CAN_AddTxMessage(hcan,&TXHeader,TXmessage,&pTxMailbox);
}
```

由于是环回模式，所以自己会接收到自己的消息，也就是会进入到接收中断中

## FreeRtos

### 配置

![1722651467184.png](./1722651467184.png)

在 CubeMX 中找到 FREERTOS，使能并且选择版本为 CMSIS_V2，然后进行参数配置，这里可以看到有很多可设置的参数，如下

- Tasks and Queues：任务与队列，用于配置任务体以及消息队列
- Timers and Semaphores：软件定时器与信号量，用于配置内核对象
- Mutexes：互斥量，用于配置内核对象
- Events：时间，用于配置内核对象
- FreeRTOS Heap Usage：查看用户任务和系统任务的堆占用
- Config Parameters：系统的参数配置
- Include Parameters：系统的功能裁剪
- Advanced Settings：CubeMX 生成代码预配置项
- User Constants：用户常量定义

### Config parameters

系统的参数配置

- **API：**显示 FreeRtos 调用的接口版本
- **Version：**显示 FreeRtos 的版本信息
- **MPU/FPU：**开启 MPU 或 FPU
    - **FPU**：浮点运算单元，是专门运算浮点数的处理器，以前 FPU 是一种单独的芯片，之后集成在 CPU 中
    - **MPU**：微处理器单元，就是把很多 CPU 集成在一起并行处理数据的芯片。但这里没有用到
- **Kernel Setting：**FreeRTOS 调度内核设置
    - **USE_PREEMPTION：**是 RTOS 调度方式的选择
        - 1：使用抢占式调度器，内核会在每个时钟节拍中断中进行任务切换
        - 0：使用携程，会在某任务进入了函数 `taskYIELD` ，调用了使任务阻塞的 API 或者程序中定义了在中断中执行上下文切换这三种情况下进行任务切换
    - **CPU_CLOCK_HZ：**CPU 时钟频率，默认使用晶振通过时钟树后获得的时钟频率
    - **TICK_RATE_HZ：**是 RTOS 的心跳时钟频率，最大为 1000，也就是心跳时钟 1ms 跳动一次
    - **MAX_PRIORITIES：**是 RTOS 任务的最高优先级设置，默认为 56。一般优先级表是 32 位，这里用了两个就是 64 位，其中的 8 位用于系统任务的优先级处理，任务优先级号越大，任务优先级越大
    - **MINIMAL_STACK_SIZE：**设置分配给空闲任务的堆栈大小，这里使用字来指定，所以默认的 128 个子就是 128 * 4 个字节。这里可根据具体情况修改
    - **MAX_TASK_NAME_LEN：**任务的名称，就是任务名字的字符串的大小，这里默认的 16 字节是足够的
    - **USE_16_BIT_TICKS：**使用存放 Tick 周期的计数器数字位宽，默认为 Disable 就是 16 位
    - **IDLE_SHOULD_YIELD：**任务是否应该调度切换
        - 1：当有另一个空闲优先级任务处于 Ready 状态时，空闲任务将不会执行它定义的功能的不止一次迭代，而不会让位于另一个任务。这确保当应用程序任务处于空闲状态时，在空闲任务中花费的时间最少，即同在空闲优先级下，空闲任务优先级更高，不会被抢占，不会以时间片运行
        - 0：空闲任务永远不会让位于另一个任务，只在被抢占时才会离开运行状态
    - **USE_MUTEXES：**开启系统构建过程中的互斥量
    - **USE_RECURSIVE_MUTEXES：**开启系统构建过程中的递归互斥量
    - **USE_COUNTING_SEMAPHORES：**开启系统构建过程中的信号量
    - **QUEUE_REGISTRY_SIZE：**队列注册表的大小，可以用于管理队列名称和队列实体，方便运行中进行查看与管理，默认为 8
    - **USE_APPLICATION_TASK_TAG：**使能时给任务一个 TAG 标签，便于用户使用
    - **ENABLE_BACKWARD_COMPATIBILITY：**一个兼容性使能，使能后 FreeRTOS 8.0.0 之后的版本可以通过宏定义使用 8.0.0 版本之前的函数接口，默认使能
    - **USE_PORT_OPTIMISED_TASK_SELECTION：**查找下一个任务方式的选择，查找下一个就绪任务就是查找优先级表，对优先级表进行导 0 算法，分为通用切换或针对性切换
        - 1：针对性切换使用处理器自带的导 0 指令，使用汇编编写，切换效率高，但兼容性差
        - 0：默认，使用通用切换，使用 C 编写，执行效率低，但兼容性好
    - **USE_TICKLESS_IDLE：**使能后生成两个空函数 `PreSleepProcessing` 和 `PostSleepProcessing` ，用户可以编写代码进入低功耗模式
    - **USE_TASK_NOTIFICATIONS：**任务通知使能，每个 RTOS 任务都有一个 32 位的通知值，RTOS 任务通知是一个直接发送给任务的事件，它可以解除接收任务的阻塞，并可选地更新接收任务的通知值
        - 1：开启
        - 0：关闭，可以为每个任务节省 8 个字节的内存空间
    - **RECORD_STACK_HIGH_ADDRESS：**记录任务堆栈入口地址到 TCB，1 使能，0 关闭
- **Memory management setting：**内存管理设置
    - **Memory Allocation：**内存分配方式，此处是默认，动态和静态都可以
    - **TOTAL_HEAP_SIZE：**内存堆的分配大小。堆本质就是一个数组，此处是设置堆数组的大小，设置时需要考虑最小要满足所有任务的使用要求，最大不能超过系统分配上限
    - **Memory Management scheme：**内存分配方式，有 heap_1~heap_5 5 种。其中 1，2，4，5 都是先建立一个数组堆，从数组中申请，用完再释放，与 C 语言中 `malloc` 和 `free` 使用链表的方式不一样，该方法在 MCU 中更安全稳定，此处默认使用的是 heap_4。具体申请和释放内存方式在 hea_4.c 中可看到
- **Hool function related definitions：**钩子函数配置。钩子函数是一种回调函数，用于在任务执行一次之后或者某些事件发生后执行的函数，该配置项里面有五个选项，控制5种不同功能的钩子函数开启，当然用户也可以在代码中自己定义
    - **USE_IDLE_HOOK：**使能之后，系统生成一个空的回调函数 `void vApplicationIdleHook(void)` ，由用户编写函数主体。每当空闲任务执行一次，钩子函数被执行一次
    - **USE_TICK_HOOK：**使能后，系统生成一个空回调函数 `void vApplicationTickHook(void)` ，由用户编写函数主体。每个 TICK 周期，钩子函数执行一次
    - **USE_MALLOC_FAILED_HOOK：**使能之后，系统生成一个空的回调函数 ****`void vApplicationMallocFailedHook(void)` ，由用户编写函数主体，当申请动态内存失败时，钩子函数会执行一次
    - **USE_DAEMON_TASK_STARTUP_HOOK：**使能之后，系统生成一个空回调函数 `void vApplicationDaemonTaskStartupHook(void)` ，任务刚启动时，钩子函数会执行一次
    - **CHECK_FOR_STACK_OVERFLOW：**使能之后，系统生成一个空回调函数 `void vApplicationStackOverflowHook(xTaskHandle xTask, signed char *pcTaskName)` ，任务栈溢出时，钩子函数会执行一次，传入任务 TCB 和任务名称
- **Run time and task stats gathering related definitions：**任务运行跟踪配置
    - **GENERATE_RUN_TIME_STATS：**开启时间统计功能，在调用 `vTaskGetRunTimeStats()` 函数时，将任务运行时间信息保存到可读列表中
    - **USE_TRACE_FACILITY：**使能后会包含额外的结构成员和函数以帮助执行可视化和跟踪，默认开启，方便 MDK 软件工具调试使用
    - **USE_STATS_FORMATTING_FUNCTIONS：**使能后会生成 `vTaskList()` 和 `vTaskGetRunTimeStats()` 函数用于获取任务运行状态
- **Co-routine related definitions：**协程配置。
    - **USE_CO_ROUTINES：**是否开启协程。开启之后需要用户手动创建协程，实际上协程很少用到了，而且 FreeRTOS 已经不会更新和维护协程了
    - **MAX_CO_ROUTINE_PRIORITIES：**协程最大优先级
- **Software timer definitions：**软件定时器配置
    - **USE_TIMERS：**默认开启软件定时器任务
    - **TIMER_TASK_PRIORITY：**软件定时器任务优先级
    - **TIMER_QUEUE_LENGTH：**定时器任务队列长度，FreeRTOS 是通过队列来发送控制命令给定时器任务，叫做定时器命令队列
    - **TMER_TASK_STACK_DEPTH：**软件定时器任务堆栈大小
- **Interrupt nesting behaviour configuration：**中断优先级设置
    - **LIBRARY_LOWEST_INTERRUPT_PRIORITY：**用来设置最低优先级的，FreeRTOS 使用的 4 位优先级，对应 16 位优先级，对应的最低优先级为 15
    - **LIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY：**设置FreeRTOS 系统可管理的最大优先级，也就是设置阈值优先级，这个大家可以自由设置，这里设置为 5，也就是高于 5 的优先级（优先级数小于 5）不归 FreeRTOS 管理

### Include Parameters

用于内核裁剪，裁剪掉不必要的功能，精简系统功能，减少资源占用，主要功能如下

- **vTaskPrioritySet**：改变某个任务的任务优先级
- **uxTaskPriorityGet**：查询某个任务的优先级。
- **vTaskDelete**：删除任务
- **vTaskCleanUpResources**：回收任务删除后的资源如RAM等等
- **vTaskSuspend**：挂起任务
- **vTaskDelayUntil**：阻塞延时一段绝对时间（绝对延时去去除程序执行时间，执行更精准）
- **vTaskDelay**：阻塞延时一段相对时间
- **xTaskGetSchedulerState**：获取任务调度器的状态，开启或未开启
- **xTaskResumeFromISR**：在中断服务函数中恢复一个任务的运行
- **xQueueGetMutexHolder**：获取信号量的队列拥有者，返回拥有此信号量的队列
- **xSemaphoreGetMutexHolder**：查询拥有互斥锁的任务，返回任务控制块
- **pcTaskGetTaskName**：获取任务名称
- **uxTaskGetStackHighWaterMark**：获取任务的堆栈的历史剩余最小值，FreeRTOS 中叫做“高水位线”
- **xTaskGetCurrentTaskHandle**：此函数用于获取当前任务的任务句柄，就是获取当前任务控制块
- **eTaskGetState**：此函数用于查询某个任务的运行壮态，比如：运行态、阻塞态、挂起态、就绪态等
- **xEventGroupSetBitFromISR**：在中断服务函数中将指定的事件位清零
- **xTimerPendFunctionCall**：定时器守护任务的回调函数（定时器守护任务使用到一个命令队列，只要向队列发送信号就可以执行相应代码，可以实现“中断推迟处理”功能）
- **xTaskAbortDelay**：中止延时函数，该函数能立即解除任务的阻塞状态，将任务插入就绪列表中
- **xTaskGetHandle**：此函数根据任务名字获取的任务句柄（控制块）

### Tasks and Queues

**任务**

任务是操作系统运行的基本单元，也是资源分配的基本单元，直接在任务那里点击 Add 就可以添加新任务，任务的参数如下

- **Task Name**：任务名称，保存在 TCB 结构体中，设置时自己起名字
- **Priority**：任务优先级，任务的调度等级，根据自己创建任务的紧急程度设定。比如通信任务不能被打断，可以设计较高优先级
- **Stack Size（Words）**：设定给任务分配的内存大小，单位是字，对于32位单片机来说占 4 个字节
- **Entry Function**：任务实体，即任务的运行函数名
- **Code Generation**：代码生成模式
    - **As weak**：产生一个用 __weak 修饰的弱定义任务函数，用户可自己在进行定义；
    - **As external**：产生一个外部引用的任务函数，用户需要自己定义该函数；
    - **Default**：产生一个默认格式的任务函数，用户需要在该函数内实现自己功能
- **Parameter**：传入的参数，保持默认就行
- **Allocation**：内存分配方式
    - **Static**：静态方式是直接在RAM占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

设置完成之后即可直接生成代码，可以在 `freertos.c` 文件中找到任务的开启和定义

需要注意的是，**任务必须是死循环**

**任务用户调用接口函数**

- `osThreadNew` ：创建新任务
- `osThreadGetName` ：获取任务名称
- `osThreadGetId` ：获取当前任务的控制块（TCB）
- `osThreadGetState` ：获取当前任务的运行状态
- `osThreadGetStackSize` ：获取任务的堆栈大小
- `osThreadGetStackSpace` ：获取任务剩余的堆栈大小
- `osThreadSetPriority` ：设定任务优先级
- `osThreadGetPriority` ：获取任务优先级
- `osThreadYield` ：切换控制权给下一个任务
- `osThreadSuspend` ：挂起任务
- `osThreadResume` ：恢复任务（挂起多少次恢复多少次）
- `osThreadDetach` ：分离任务，方便任务结束进行回收
- `osThreadJoin` ：等待指定的任务停止
- `osThreadExit` ：停止当前任务
- `osThreadTerminate` ：停止指定任务
- `osThreadGetCount` ：获取激活的任务数量
- `osThreadEnumerate` ：列举激活的任务

**队列**

队列，又称为消息队列，用于任务间的数据通信，传输数据，在操作系统里面，直接使用全局变量传输数据十分危险，看似正常运行，但不知道啥时候就会因为寄存器或者内存等等原因引起崩溃，所以引入消息，队列的概念，任务发送数据到队列，需要接受消息的任务挂起在队列的挂起列表，等待消息的到来。直接在 CubeMX 中点击 Add 创建队列，其中队列的参数如下

- **Queue Name**：队列名称（自己设定）
- **Queue Size**：消息队列大小
- **Item Size**：队列传输类型，保持默认16 位就行
- **Allocation**：队列内存的分配方式
    - **Static**：静态方式是直接在RAM占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

然后点击生成代码，之后就可在 `freertos.c` 中系统初始化函数中看到队列的初始化

初始化函数会在一开始被调用，对 FreeRTOS 系统和内核对象进行初始化，初始化后系统就可以进行调度和使用内核对象，CubeMX 生成的代码自动将创建的内核对象放到初始化函数内，所以在任务和中断中直接使用就可以。

**队列用户调用函数接口**

- `osMessageQueueNew` ：创建并初始化一个新的队列
- `osMessageQueueGetName` ：获取队列的名字
- `osMessageQueuePut` ：发送一条消息到队列
    - `osStatus_t osMessageQueuePut (osMessageQueueId_t mq_id, const void *msg_ptr, uint8_t msg_prio, uint32_t timeout)`
    - `mq_id` ：传入队列的句柄
    - `msg_ptr` ：指向需要发送的消息内容的指针
    - `msg_prio` ：本次发送消息的优先级，但是实际中并没有用到
    - `timeout` ：发送消息的超时时间，设置为 0 表示一直等待发送成功
    - `osStatus_t` ：返回执行结果
        - `osOK` ：执行正常
        - `osError` ：系统错误
        - `osErrorTimeout` ：执行超时
        - `osErrorResource` ：资源不可用
        - `osErrorParameter` ：参数无效
        - `osErrorNoMemory` ：内存不足
        - `osErrorISR` ：不允许在中断调用
        - `osStatusReserved` ：防止编译器优化项，不需要在意
- `osMessageQueueGet` ：从队列等待一条消息
    - `osStatus_t osMessageQueueGet (osMessageQueueId_t mq_id, void *msg_ptr, uint8_t *msg_prio, uint32_t timeout)`
    - `mq_id` ：接受队列的句柄
    - `msg_ptr` ：用于接受消息内容的指针
    - `msg_prio` ：存放接受消息的优先级（目前API未加入功能）
    - `timeout` ：接受消息的超时时间（设置为10代表，当前任务挂起在挂起列表，直到接收成功时恢复，或者 10 个 TICK 等待周期到达然后任务强行恢复，不再等待，为 0 则是不等待，等待期间任务挂起在内核对象的挂起队列）
    - `osStatus_t` ：返回执行结果，同上
- `osMessageQueueGetCapacity` ：获取队列传输消息的峰值
- `osMessageQueueGetMsgSize` ：获取队列使用内存池的最大峰值
- `osMessageQueueGetCount` ：获取队列的消息数量
- `osMessageQueueGetSpace` ：获取队列剩余的可用空槽
- `osMessageQueueReset` ：清空队列
- `osMessageQueueDelete` ：删除队列

### Timers and Semaphores

创建定时器和信号量

**创建定时器**

软件定时器本质上就是设置一段时间，当设置的时间到达之后就执行指定的回调函数，回调函数的两次执行间隔叫做定时器的定时周期

在 CubeMX 中直接在 Timers 中点击 Add 即可添加定时器，具体参数如下

- **Timer Name**：设置定时器的名称
- **Callback**：设定定时器的回调函数体
- **Type**：设定定时器的执行类型
    - **osTimerPeriodic**：定时器周期执行回调函数
    - **osTimerOnce**：定时器只执行一次回调函数
- **Code Generation Option**：代码生成模式
    - **As weak**：产生一个用 __weak 修饰的弱定义任务函数，用户可自己在进行定义
    - **As external**：产生一个外部引用的任务函数，用户需要自己定义该函数
    - **Default**：产生一个默认格式的任务函数，用户需要在该函数内实现自己的功能
- **Parameter**：传入参数，保持默认 NULL 就行
- **Allocation**：软件定时器内存的分配方式，一般使用动态
    - **Static**：静态方式是直接在RAM占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

之后直接生成代码即可在 `freertos.c` 文件中看到定时器创建之后获得的句柄和生成的回调函数

之后就是 CubeMX 提供的定时器接口函数以及功能

- `osTimerNew` ：新建定时器，返回定时器控制句柄
- `osTimerGetName` ：获取定时器名称
- `osTimerStart` ：设置定时器周期，启动定时器
    - `osStatus_t osTimerStart (osTimerId_t timer_id, uint32_t ticks)`
    - `timer_id` 需要启动的定时器句柄
    - `ticks` 设置定时器的运行周期
- `osTimerStop` ：停止定时器
    - `osStatus_t osTimerStop (osTimerId_t timer_id)`
    - `timer_id` 需要启动的定时器句柄
- `osTimerIsRunning` ：检测定时器是否在运行
- `osTimerDelete` ：删除定时器

软件定时器是由软件定时器维护任务进行维护的，检测各个定时器的状态，进行处理

**创建信号量**

信号量是 RTOS 的一个内核对象，该对象有一个队列表示该信号量拥有的信号数目，任何任务都可以对这个信号数目进行获取和释放，获取时信号 - 1，释放时信号 + 1，为 0 时不能继续获取，此时有任务想要继续获取信号量的话，任务会挂起在该内核对象的挂起列表，等到信号可以获取时进行恢复，根据这个特性，信号量常用于控制对共享资源的访问和任务同步

在配置界面中可以看到有两个信号量添加页面

- Binary Semaphores 是二值信号量，仅有一个 token，用于同步一个操作
- Counting Semaphores 是计数信号量，有多个 token，可用于同步多个操作

在对应的信号量中点击 Add 即可创建信号量，可以看到初始的一些配置

Binary Semaphores 配置

- **Semaphore Name**：信号量名称
- **Allocation**：内存分配方式，一般使用动态
    - **Static**：静态方式是直接在RAM占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

Counting Semaphore 配置

- **Semaphore Name**：信号量名称
- **Count**：计数信号量的最大数目
- **Allocation**：内存分配方式，一般使用动态
    - **Static**：静态方式是直接在 RAM 占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

之后直接生成代码即可在 `freertos.c` 文件中看到信号量创建之后获得的句柄和生成的回调函数

之后就是 CubeMX 提供的信号量操作接口函数以及功能

- `osSemaphoreNew` ：创建新的信号量
- `osSemaphoreGetName` ：获取信号量的名称
- `osSemaphoreAcquire` ：获取信号量
    - `osStatus_t osSemaphoreAcquire (osSemaphoreId_t semaphore_id, uint32_t timeout)`
    - `semaphore_id` ：传入要获取信号量的控制句柄
    - `timeout` ：获取等待时间（等待期间任务挂起在内核对象的挂起队列）
    - `osStatus_t` ：返回 `osOK` 即为获取成功
- `osSemaphoreRelease` ：释放信号量
    - `osStatus_t osSemaphoreRelease (osSemaphoreId_t semaphore_id)`
    - `semaphore_id` ：传入要释放的信号量控制句柄
    - `osStatus_t` ：返回 `osOK` 即为释放成功
- `osSemaphoreGetCount` ：获取当前可用信号量的数目
- `osSemaphoreDelete` ：删除信号量

二值信号量和计数信号量的操作基本一致，没用区别，只是用有的信号队列最大数目不同而已。同时注意信号量在使用过程中会出现优先级反转的 Bug，使用时需要注意

### Mutexs

互斥量其实就是一个拥有优先级继承的二值信号量，互斥信号量适合用于那些需要互斥访问的应用中，在互斥访问中互斥信号量相当于一个钥匙，当任务想要使用资源的时候就必须先获得这个钥匙，当使用完资源以后就必须归还这个钥匙，这样其他的任务就可以拿着这个钥匙去使用资源，与信号量不同的是，互斥量的释放必须由获取他的任务进行释放，如果不释放，可能会造成死锁

CubeMX 中提供了两种信号量

- Mutexs：普通信号量，只能获取一次，重复获取无效
- Recursive Mutexs：递归信号量，可以获取多次，同样也需要释放多次才能让出使用权

直接点击 Add 然后配置参数，如下

- **Mutex Name**：互斥量名称
- **Allocation**：内存分配方式，一般使用动态
    - **Static**：静态方式是直接在RAM占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

两种信号量配置方式一样，参数也一样，只是在用法上不同。生成代码之后，会在 `freertos.c` 文件中看到互斥量初始化完成，并且生成了对应的控制句柄

CubeMX 提供了信号量的使用接口，如下

- `osMutexNew` ：创建互斥量
- `osMutexGetName` ：获取互斥量名称
- `osMutexAcquire` ：任务获取互斥量
    - `osStatus_t osMutexAcquire (osMutexId_t mutex_id, uint32_t timeout)`
    - `mutex_id` ：互斥量控制句柄
    - `timeout` ：获取互斥量时的等待时间（等待期间任务挂起在内核对象的挂起队列）
    - `osStatus_t` ：返回 `osOK` 即为获取成功
- `osMutexRelease` ：任务释放互斥量
    - `osStatus_t osMutexRelease (osMutexId_t mutex_id)`
    - `mutex_id` ：互斥量控制句柄
    - `osStatus_t` ：返回 `osOK` 即为释放成功
- `osMutexGetOwner` ：获取互斥量的拥有任务的任务 TCB
- `osMutexDelete` ：删除互斥量

### Events

任务间的同步除了信号量还有时间标志组，信号的同步通常是一对一的同步，有的时候系统需要多对一的同步，比如同时满足5个按键按下时，任务启动，如果使用信号会很占据资源，所以 RTOS 引入了事件标志组来满足这一需求

在 CubeMX 中直接点击 Add 创建事件标志组，其中参数如下

- **Event flags Name**：事件标志组名称
- **Allocation**：内存分配方式，一般使用动态
    - **Static**：静态方式是直接在 RAM 占据一个静态空间
    - **Dynamic**：动态方则是在初始配置的内存池大小数组中动态申请、释放空间

配置完成后，生成代码，在系统初始化内，可以看到生成事件标志组控制句柄

CubeMX 提供的事件操作接口如下

- `osEventFlagsNew` ：创建事件标志组
- `osEventFlagsGetName` ：获取事件标志组名称
- `osEventFlagsSet` ：设置事件标志组
    - `uint32_t osEventFlagsSet (osEventFlagsId_t ef_id, uint32_t flags)`
    - `ef_id` ：事件标志组控制句柄
    - `flags` ：事件位
- `osEventFlagsClear` ：清除事件标志组
- `osEventFlagsGet` ：获取当前事件组标志信息
- `osEventFlagsWait` ：等待事件标志组触发
    - `uint32_t osEventFlagsWait (osEventFlagsId_t ef_id, uint32_t flags, uint32_t options, uint32_t timeout)`
    - `ef_id` ：事件标志组控制句柄
    - `flags` ：等待的事件位
    - `options` ：等待事件位的操作
        - `osFlagsWaitAny` ：等待的事件位有任意一个等到就恢复任务
        - `osFlagsWaitAll` ：等待的事件位全部等到才恢复任务
        - `osFlagsNoClear` ：等待成功后不清除所等待的标志位（默认清除）
    - `timeout` ：等待事件组的等待时间（等待期间任务挂起在内核对象的挂起队列）
- `osEventFlagsDelete` ：删除事件标志组

### User Constants

用户常量，将不变的量转化为常量保存，可以节省 RAM 资源空间，因为常量和变量的保存位置不同

### 任务通知

FreeRTOS 的每个任务都有一个 32 位的通知值，任务控制块中的成员变量 ulNotifiedValue 就是这个通知值。任务通知是一个事件，假如某个任务通知的接收任务因为等待任务通知而阻塞的话，向这个接收任务发送任务通知以后就会解除这个任务的阻塞状态，CubeMX内没有提供相关的配置项，但在其生成的 FreeRTOS 接口里面有相关函数进行配置，函数声明位于 `cmsis_os2.h` 中

函数接口如下

- `osThreadFlagsSet` ：设置任务的通知标志
    - `uint32_t osThreadFlagsSet (osThreadId_t thread_id, uint32_t flags)`
    - `thread_id` ：任务控制块
    - `flags` ：设置的标志
- `osThreadFlagsClear` ：清除任务通知
- `osThreadFlagsGet` ：获取任务标志
- `osThreadFlagsWait` ：等待特定的任务标志
    - `uint32_t osThreadFlagsWait (uint32_t flags, uint32_t options, uint32_t timeout)`
    - `flags` ：设置的标志
    - `options` ：设置功能
        - `osFlagsWaitAny` ：等待的事件位有任意一个等到就恢复任务
        - `osFlagsWaitAll` ：等待的事件位全部等到才恢复任务
        - `osFlagsNoClear` ：等待成功后不清除所等待的标志位（默认清除）
    - `timeout` ：超时时间

任务通知其实个任务事件标志组使用上没有多大的区别，但他们两个的实现原理不同，同时任务通知对资源的占用更少。根据 FreeRTOS 官方的统计，使用任务通知替代二值信号量的时候任务解除阻塞的时间要快 45%，并且需要的 RAM 也更少

### 系统内核配置

CubeMX 生成的代码中封装了一系列内核配置函数，如下

- `osKernelInitialize` ：初始化RTOS的内核
- `osKernelGetInfo` ：获取RTOS的信息
- `osKernelGetState` ：获取当前内核的运行状态
- `osKernelStart` ：启动内核调度
- `osKernelLock` ：锁内核调度器
- `osKernelUnlock` ：解锁内核调度器
- `osKernelRestoreLock` ：恢复RTOS内核调度器锁状态
- `osKernelSuspend` ：挂起任务
- `osKernelResume` ：恢复任务
- `osKernelGetTickCount` ：用于获取系统当前运行的时钟节拍数
- `osKernelGetTickFreq` ：用于获取系统当前运行的时钟节拍的分频频率
- `osKernelGetSysTimerCount` ：获取系统时钟 `SysTick` 的计数值
- `osKernelGetSysTimerFreq` ：获取系统时钟 `SysTick` 的频率

### FreeRTOS 特点

- **实时性：**FreeRTOS可以配置成为一个硬实时操作系统内核，也可以配置为非实时型内核，甚至于部分任务是实时性的，部分不是
- **任务数量：**FreeRTOS对任务数没有限制，同一优先级也可以有多个任务
- **抢占式或协作式调度算法：**任务调度既可以为抢占式也可以为协作式。采用协作式调度算法后，一个处于运行态任务除非主动要求任务切换，否则是不会被调度出运行态的
- **任务调度的时间点：**调度器会在每次定时中断到来时决定任务调度，同时外部异步事件也会引起调度器任务调度
- **调度算法：**任务调度算法首先满足高优先级任务最先执行，当多于 1 个任务具有相同的高优先级时，采用 round robin 算法调度
- **优先级翻转：**FreeRTOS 没有提供优先级继承机制或其他的避免优先级翻转的方法
- **任务间通信：**FreeRTOS 支持队列和几种基本的任务同步机制
    - **消息队列：**任务间传递信息可以采用队列方式，FreeRTOS实现的队列机制传递信息是采用传值方式，因此对于传递大量数据效率有些低。但可以通过传递指针的方式提高效率。中断处理函数中读写队列都是非阻塞型的。任务中读写队列可以为阻塞型也可以配置非阻塞型。当配置为阻塞型时可以指定一个阻塞的最大时间限（Timeout）。
    - **任务间同步：**FreeRTOS 支持基本的信号量功能。FreeRTOS 采用队列来实现信号量的功能，可以认为一个值为n的信号量就是一个长度为n的队列，队列中每个元素的大小为0。这样的队列并不会浪费宝贵的内存空间。
    - **对于死锁（Deadlock）的处理：** FreeRTOS 并没有实现一种可以完全避免死锁的机制。只是通过指定一个阻塞的最大时间限（Timeout）来减少死锁现象的发生。或者说是给出了当死锁现象发生时解锁的可能。当然能不能真的解锁要依赖于使用者的处理代码是否合适。
    - **临界区：**FreeRTOS 采用开关中断的方式实现临界区保护。任务代码中临界区可以嵌套，FreeRTOS 会自动记录每个任务中临界区嵌套的层数。
    - **暂停调度：**与进入临界区类似，FreeRTOS 可以通过暂时关闭任务调度来保证任务代码不被更高优先级的其他任务打断，与临界区不同，关闭任务调度并不会关闭中断，这样中断处理函数仍会照常的执行。
- **内存分配：**FreeRTOS 提供了多种内存动态分配的方法，具体程序中需要选择其中一种。最简单的内存分配方式提供了一种非常简单的固定内存分配算法，这种方式下只支持内存的分配，不支持分配内存的回收。因此，任务建立后就不能被删除。其他几种内存分配算法支持分配内存的回收，有的方法支持邻接内存块的合并，有些不支持。对于 uCOS-II 中内存分配的方法，既保证了实时性，也具有一定的灵活性。FreeRTOS 中提供的几种方式，实时性好的功能上有缺陷，功能上完善的实时性却不好

### FreeRTOS 任务调度机制

**调度策略**

- **可抢占：**在可抢占式调度中，任务可以被更高优先级的任务抢占。当一个高优先级任务变得可用时，它可以打断当前正在执行的低优先级任务，从而使系统立即切换到高优先级任务执行，无论被抢占的任务是否已经执行完其时间片。它确保了高优先级任务能够及时响应，并在需要时立即执行，不受低优先级任务的阻碍。通过设置 `configUSE_PREEMPTION` 来决定是否启动抢占
- **时间片轮转：**操作系统为每个任务分配一个时间片，即预定义的时间量。在时间片轮转调度方式下，每个任务可以执行一个时间片，然后系统将控制权移交给下一个就绪的任务。如果一个任务在其时间片结束前没有完成，系统会暂停该任务，将控制权交给下一个就绪的任务。时间片的大小可以根据应用程序的需要进行调整。这种调度方式有助于确保任务之间的公平性，避免某些任务长时间占用处理器，同时允许多个任务分享处理时间。通过设置 `configUSE_TIME_SLICING` 来决定是否启用时间片轮转
- **组合应用：**可抢占和时间片轮转调度方式可以结合使用。这样可以实现灵活的任务管理，确保高优先级任务能够抢占低优先级任务，并且为任务提供公平的处理器时间，从而有效地管理系统资源

**实现核心**

任务管理使用就绪链表，阻塞链表和挂起链表来管理任务的状态和调度。这些链表用于维护不同状态的任务列表

- **就绪链表：**就绪链表包含所有处于就绪状态的任务。就绪状态的任务就是已经准备好运行，但由于当前执行的任务正在占用 CPU 资源，它们暂时无法立即执行。这些任务按照优先级被组织在就绪链表中。当当前正在执行的任务释放 CPU（例如，由于时间片用完，任务阻塞或挂起等原因）时，调度器从就绪链表中选择优先级最高的任务来执行
- **阻塞链表：**阻塞链表包含那些由于某种原因而无法立即执行的任务。这些原因可能包括等待某个事件、资源不可用、延时等情况。当任务处于阻塞状态时，它们不会被调度器所执行。这些任务会在特定条件满足之后重新放入就绪链表，等待调度器选择其执行
- **挂起链表：**挂起链表包含已被显式挂起的任务。当任务被挂起时，它们暂时停止运行，不再参与调度。这些任务不会出现在就绪链表或阻塞链表中，因为它们被明确地挂起，不参与任务调度。

这些链表是 FreeRTOS 内部任务管理的一部分，并且开发者可以通过 FreeRTOS 提供的 API 函数来管理和操作任务的状态以及链表中的任务

### FreeRTOS 任务释放 CPU 的几种情况

1. **任务主动让出CPU：**任务可以调用 `osDelay` 函数或者 `osDelayUntil` 函数，将自己挂起一段时间，以便其他任务能够运行。这种方式是任务主动放弃CPU的一种方式。
2. **阻塞等待事件：**任务可以调用 FreeRTOS 提供的阻塞函数，如 `osMessageQueueGet` 和 `osSemaphoreAcquire` 等，来等待特定事件的发生。当任务在等待某个事件时，它会被置于阻塞状态，从而释放 CPU，直到事件发生后才会被唤醒。
3. **时间片轮转：**如果使用了时间片轮转调度策略，任务会在其时间片用尽时自动释放CPU，允许其他任务运行。时间片轮转是一种公平分配CPU时间的策略，每个任务都有一个小的时间片来执行，然后被放回就绪队列，等待下一次执行。
4. **任务进入阻塞状态：**任务在执行过程中，如果发生某些阻塞事件，如等待一个队列满足条件和等待互斥信号量等，会自动进入阻塞状态，这时会释放 CPU。一旦阻塞条件得到满足，任务将被重新置于就绪状态。

### FreeRTOS 使能之后黄色感叹号

**When FreeRTOS is used, it is strongly recommanded to use HAL timebase source other than the Systick**

HAL函数如果是阻塞型呼叫，內部会用到 `HAL_Delay()` ，FreeRTOS应该还是使用 `SystTick` 。如果使用的时基操作來源一样，怕有不可预期问题出现，故选择其他定时器

### FreeRTOS 任务只能启动几个

这是因为 FreeRTOS 配置中的 **TOTAL_HEAP_SIZE** 设置的有点小，导致给栈分配的内存不够了，以至于分配失败而任务也不能运行

## 外部中断

外部中断就是 GPOIO 引脚对外部的输入信号进行检测，对应的上升/下降沿会触发中断（具体看设置情况）

### 寄存器

![b09cf826aa892f58f507ed7c68774f6b.png](./b09cf826aa892f58f507ed7c68774f6b.png)

外部中断的功能可以配置以下六个寄存器：

- 中断屏蔽寄存器（EXTI_IMR）
- 事件屏蔽寄存器（EXTI_EMR）
- 上升沿触发选择寄存器（EXTI_RTSR）
- 下降沿触发选择寄存器（EXTI_FTSR）
- 软件中断事件寄存器（EXTI_SWIER）
- 挂起寄存器（EXTI_PR）

### EXTI

![5b66279347282125064682f5b1ed4b00.png](./5b66279347282125064682f5b1ed4b00.png)

**EXTI 支持配置 20 个 中断和事件屏蔽位：**

- `EXTI_Line0~EXTI_Line15` ：GPIO 对应的引脚所触发的中断
- `EXTI_Line16` ：连接到 PVD 输出
- `EXTI_Line17` ：连接到 RTC 闹钟事件
- `EXTI_Line18` ：连接到 USB 唤醒事件
- `EXTI_Line19` ：连接到以太网唤醒事件，只适用于互联网型产品

**中断服务函数的映射关系**

| GPIO                    | IRQN           | IRQN_Handler         |
| ----------------------- | -------------- | -------------------- |
| GPIO_Pin0               | EXTI0_IRQn     | EXTI0_IRQHandler     |
| GPIO_Pin1               | EXTI1_IRQn     | EXTI1_IRQHandler     |
| GPIO_Pin2               | EXTI2_IRQn     | EXTI2_IRQHandler     |
| GPIO_Pin3               | EXTI3_IRQn     | EXTI3_IRQHandler     |
| GPIO_Pin4               | EXTI4_IRQn     | EXTI4_IRQHandler     |
| GPIO_Pin5 — GPIO_Pin9   | EXTI9_5_IRQn   | EXTI9_5_IRQHandler   |
| GPIO_Pin10 — GPIO_Pin15 | EXTI15_10_IRQn | EXTI15_10_IRQHandler |

### 配置

![1723366952195.png](./1723366952195.png)

首先是在右侧选中引脚，将其设置为 `GPIO_EXTI0` 模式，然后左侧开始进行配置

**GPIO 配置**

- GPIO mode：触发中断的模式
    - External Interrupt Mode with Rising edge trigger detection：外部中断模式上升沿触发
    - External Interrupt Mode with Falling edge trigger detection：外部中断模式下降沿触发
    - External Interrupt Mode with Rising/Falling edge trigger detection：外部中断模式上升和下降沿触发
    - External Event Mode with Rising edge trigger detection：外部事件模式上升沿触发
    - External Event Mode with Falling edge trigger detection：外部事件模式下降沿触发
    - External Event Mode with Rising/Falling edge trigger detection：外部事件模式上升和下降沿触发
- GPIO Pull-up/Pull-down：GPIO 的上拉下拉模式
- User Label：用户别名

**中断配置**

这里所用到的中断就是 `EXTI0_IQRHandler` 中断函数。从 `stm32f4xx_it.c` 中找到 HAL 库中的函数，然后可以通过实现 `void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)` 函数来实现中断回调函数

## SPI

### 介绍

SPI 就是 Serial Peripheral interface 的缩写，就是串行外围设备接口。它是一种高速的，全双工的，同步的通信总线，并且在芯片的管脚上只占用四根线，节约了芯片的管脚

SPI 是全双工且 SPI 没有速度限制，一般的实现通常能达到甚至超过 10 Mbps

### 主从模式

SPI 分为主，从两种模式。一个 SPI 通信系统需要包含一个主设备和一个或多个从设备。其中提供时钟的是主设备，接收时钟的设备为从设备。SPI 接口的读写操作都是由主设备发起的。当存在多个从设备时，通过各自的片选信号进行管理

### 硬件信号线

![88e54ca6f01bbead519e39e3f28d0a22.png](./88e54ca6f01bbead519e39e3f28d0a22.png)

SPI 接口一般使用四条信号线通信：SDI 数据输入，SDO 数据输出，SCK 时钟，CS 片选。如下

- `MISO` 主设备输入/从设备输出引脚。该引脚在从模式下发送数据，在主模式下接收数据
- `MOSI` 主设备输出/从设备输入引脚。该引脚在主模式下发送数据，在从模式下接收数据
- `SCLK` 串行时钟信号，由主设备产生
- `CS/SS` 从设备片选信号，由主设备控制。它的功能是用来作为片选引脚，也就是选择指定的从设备。可以让主设备单独地与特定从设备通信，避免总线上的冲突

### 发送接收原理

1. 通讯开始：NSS 线信号由高变低，对应的从机被选中，开始通讯
2. 通讯结束：NSS 线信号由低变高，对应的从机取消被选中状态

SPI 主机和从机都有一个串行移位寄存器，主机通过向它的 SPI 串行寄存器写入一个字节来发起一次传输

1. 首先拉低对应的 SS 信号线，表示与该设备进行通信
2. 主机通过发送 SCLK 时钟信号来告诉从机写数据或读数据
3. 主机将要发送的数据写入到发送数据缓存区，缓存区经过移位寄存器，串行移位寄存器通过 MOSI 信号线将字节一位一位的传送给从机，同时 MISO 接口接收到的数据经过移位寄存器移动到接收缓冲区
4. 从机也将自己的串行移位寄存器中的内容通过 MISO 信号线返回给主机。同时通过 MOSI 信号线接收主机发送的数据，以此来实现交换两个移位寄存器中的内容

![120d59bba98628fa7bdfe66157a26e3b.png](./120d59bba98628fa7bdfe66157a26e3b.png)

SPI 只有主从模式之分，没有读和写的说法，外设的写操作和读操作是同步完成的。如果只进行写操作，主机只需要忽略接收到的字节即可。如果主机想要读取从机的一个字节，就必须发送一个空字节来引发从机的传输。也就是发送数据时必然会收到一个数据，想要接收到一个数据就必须先发送一个数据

### 工作模式

根据 SPI 的时钟极性 CPOL 和相位 CPHA 不同，SPI 有四种工作模式

时钟极性定义了时钟空闲时电平

- CPOL=0：时钟空闲时为低电平
- CPOL=1：时钟空闲时为高电平

时钟相位定义数据的采集事件

- CPHA=0：在时钟的第一个跳变沿（上升沿或者下降沿）进行数据采样
- CPHA=1：在时钟的第二个跳变沿进行数据采样

上面经过组合之后可以得到，但是需要注意的是，主从机必须选用同一种模式

| SPI 模式 | CPOL | CPHA | 空闲时 SCK 时钟 | 采样时刻         |
| -------- | ---- | ---- | --------------- | ---------------- |
| 0        | 0    | 0    | 低电平          | 第一个边沿（奇） |
| 1        | 0    | 1    | 低电平          | 第二个边沿（偶） |
| 2        | 1    | 0    | 高电平          | 第一个边沿（奇） |
| 3        | 1    | 1    | 高电平          | 第二个边沿（偶） |

SPI 工作在 3 种模式下，分别是运行，等待和停止

- 运行模式（Run Mode）这是基本的操作模式
- 等待模式（Wait Mode）SPI 工作在等待模式是一种可配置的低功耗模式，可以通过 SPICR2 寄存器的 SPISWAI 位进行控制。在等待模式下，如果 SPISWAI 位清 0，SPI 操作类似于运行模式。如果 SPISWAI 位置 1，SPI 进入低功耗状态，并且 SPI 时钟将关闭。如果 SPI 配置为主机，所有的传输将停止，但是会在 CPU 进入运行模式后重新开始。如果 SPI 配置为从机，会继续接收和传输一个字节，这样就保证从机与主机同步
- 停止模式（Stop Mode）为了降低功耗，SPI 在停止模式是不活跃的。如果 SPI 配置为主机，正在进行的传输会停止，但是在 CPU 进入运行模式后会重新开始。如果 SPI 配置为从机，会继续接受和发送一个字节，这样就保证了从机与主机同步

### 配置

![1723783409366.png](./1723783409366.png)

这里使用大疆C板做配置。将 SPI 模式设置为全双工主模式，其他模式为

- Full-Duplex Master：全双工主模式
- Full-Duplex Slave：全双工从模式
- Half-Duplex Master：半双工主模式
- Half-Duplex Slave：半双工从模式
- Receive Only Master：只接收主模式
- Receive Only Slave：只接收从模式
- Transmit Only Master：只发送主模式
- Transmit Only Slave：只发送从模式

在 STM32 上有硬件 NSS 片选信号，可以选择使能，也可以使用其他 IO 口接到芯片的 NSS 上进行代替。其中

- SPI1 的片选 NSS 为 PA4
- SPI2 的片选 NSS 为 PB12

如果片选引脚没有连接这两个引脚，则需要选择软件片选

对应的软件片选，需要设置对应的引脚，比如这里就是选择 PA4 和 PB0 作为软件片选引脚，将其设置为输出模式

> **NSS 管脚以及片选信号：**
> 
> 
> 作为主设备 NSS 管脚为高电平，从设备 NSS 管脚为低电平。当 NSS 管脚为低电平时，该 SPI 设备被选中，可以和主设备进行通信
> 
> 在 STM32 中，每个 SPI 控制器的 NSS 信号引脚都具有两种功能，即输入和输出
> 
> - 输入：就是 NSS 管脚的信号给自己
> - 输出：就是将 NSS 的信号送出去给从机
> 
> 对于NSS的输入，又分为软件输入和硬件输入。
> 
> - 软件输入：NSS 分为内部管脚和外部管脚，通过设置 spi_cr1 寄存器的 ssm 位和 ssi 位都为 1 可以设置 NSS 管脚为软件输入模式且内部管脚提供的电平为高电平，其中 SSM 位为使能软件输入位。SSI 位为设置内部管脚电平位。同理通过设置 SSM 和 SSI 位分别为 1 和 0，则此时的 NSS 管脚为软件输入模式但内部管脚提供的电平为 0。若从设备是一个其他的带有 SPI 接口的芯片，并不能选择 NSS 管脚的方式，则可以有两种办法，一种是将 NSS 管脚直接接低电平，另一种就是通过主设备的任何一个 GPIO 口去输出低电平选中从设备
> - 硬件输入：主机接高电平，从机接低电平

这里对 SPI 的参数设置如下

- Frame Format：框架格式
    - Motorola
    - TI
- Data Size：数据长度
    - 8 Bit
    - 16 Bit
- First Bit：对齐格式
    - MSB：高位先行
    - LSB：低位先行
- Prescaler：预分频，用于控制波特率
- Baud Rate：波特率=16MHz/Prescaler
- Clock Polarity：CPOL
- Clock Phase：CPHA
- CRC Calculation：是否启用 CRC 校验
- NSS Signal Type：片选形式，硬件实现还是软件实现

![1723805210121.png](./1723805210121.png)

在 NVIC Settings 中启用 SPI global interrupt，这就打开了 SPI 的中断

![1723805247964.png](./1723805247964.png)

然后在 DMA Settings 中添加 DMA 的接收和发送数据流，如上。其中有个 DMA 接收参数

- DMA 模式，有两种模式，这里与 UART 是一致的
    - Normal：只进行一次接收或者发送
    - Circular：进行多次接收或者发送

配置 DMA 时一定要将 SPI 的中断打开

### 三种接收和发送

**阻塞式**

```c
// 发送数据
HAL_StatusTypeDef  HAL_SPI_Transmit(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// 接收数据
HAL_StatusTypeDef  HAL_SPI_Receive(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// 全双工：发送并且接收数据
HAL_StatusTypeDef HAL_SPI_TransmitReceive(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData, uint16_t Size, uint32_t Timeout);
```

**中断式**

```c
// 发送数据
HAL_StatusTypeDef HAL_SPI_Transmit_IT(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size);
// 接收数据
HAL_StatusTypeDef HAL_SPI_Receive_IT(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size);
// 发送和接收数据，发送接收成功之后进入中断函数
HAL_StatusTypeDef HAL_SPI_TransmitReceive_IT(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData, uint16_t Size);
```

**DMA 式**

```c
// DMA 发送数据
HAL_StatusTypeDef HAL_SPI_Transmit_DMA(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size);
// DMA 接收数据
HAL_StatusTypeDef HAL_SPI_Receive_DMA(SPI_HandleTypeDef *hspi, uint8_t *pData, uint16_t Size);
// DMA 发送并且接收数据，发送接收成功之后进入中断函数
HAL_StatusTypeDef HAL_SPI_TransmitReceive_DMA(SPI_HandleTypeDef *hspi, uint8_t *pTxData, uint8_t *pRxData, uint16_t Size);
```

## IIC

### 简介

IIC 总线是一种由飞利浦公司开发的串行总线。是两条串行的总线，由一根数据线和一根时钟线组成。IIC 上可以连接多个设备，每个器件都有一个唯一的地址识别。同一事件只能有一个主设备，其他的为从设备

STM32 的 IIC 外设可用作通讯的主机以及从机，支持 100Kbs 和 400Kbs 的传输速率，支持 DMA 传输数据，并且具有数据校验功能。需要注意的是 IIC 是为了与低速设备通信而发明的，所以传输速率上比不上 SPI

### 硬件物理层

IIC 一共有两条线，一条是双向的数据线 SDA，一条是串行的时钟线 SCL

所有接到 IIC 总线设备上的串行数据线 SDA 都接在总线的 SDA 上，所有设备的时钟线 SCL 接到总线的 SCL 上。IIC 总线上的每一个设备都对应着唯一的地址

### 传输的起始信号和停止信号

- 起始信号：SCL 保持高电平，SDA 由高电平转为低电平之后，延时 SCL 变为低电平
- 停止信号：SCL 保持高电平，SDA 由低电平变为高电平

![e82e8e8b36bd6946d3302f04990ce726.png](./e82e8e8b36bd6946d3302f04990ce726.png)

### 传输数据的有效性

IIC 信号在传输过程中，当 SCL=1 高电平时，数据线 SDA 必须保持稳定状态，不允许由电平跳变，只有在时钟线上的信号为低电平期间，数据线上的高电平或低电平状态才允许变化。SCL=1 时，数据线 SDA 的任何电平变换都会被看作是总线的起始或者停止信号。也就是在 IIC 传输数据的过程中，SCL 时钟线会频繁的转换电平，以保证数据的传输

### 传输应答信号

每当主机向从机发送完一个字节的数据时，主机都需要等待从机给出一个应答信号以确认从机是否接收到了数据

![d07bc471cd6af605225a9367ada7cb56.png](./d07bc471cd6af605225a9367ada7cb56.png)

**应答信号**：主机 SCL 拉高，读取从机 SDA 电平，为低电平表示产生应答

- 当应答信号为低电平时，规定为有效应答位，表示接收器已经成功接收到了该字节
- 当应答信号为高电平时，规定为非应答位，一般表示接收器接收该字节没有成功

应答出现在每一次主机完成一个字节的数据传输之后紧跟着的时钟周期，低电平表示应答，高电平表示非应答

![6ad0b5d5ffa681f2abdf177fff47de0d.png](./6ad0b5d5ffa681f2abdf177fff47de0d.png)

### 模式选择

IIC 在工作室可以选用以下四种模式之一

- 从发送器
- 从接收器
- 主发送器
- 主接收器

默认情况下，设备以从模式工作。接口在生成起始位后会自动由从模式切换为主模式，并在出现仲裁丢失或生成停止位时从主模式切换为从模式，从而实现多主模式功能

### 通信流程

在主模式下，IIC 接口会启动数据传输并生成时钟信号。串行数据传输始终是在出现起始信号时开始，在出现停止信号时结束。起始位和停止位均在主模式下由软件生成

在从模式下，该接口能够识别其自身地址（7 或 10 位）以及广播呼叫地址。广播呼叫地址检测可由软件使能或禁止

数据和地址均以 8 位字节传输，MSB 在前。起始信号后紧紧跟随这地址字节，7 位地址占据一个字节，10 位地址占据两个字节。地址始终在主模式下传送

在传输 8 个时钟周期之后的第 9 个时钟脉冲期间中，接收器必须向发送器发送一个应答信号。应答信号可被软件使能或禁止

### 从模式

默认情况下，IIC 接口在从模式下工作，要将工作模式由默认的从模式切换为主模式，需要生成一个起始位

为了时序正确，必须在 `IIC_CR2` 寄存器中对外设输入时钟进行编程。外设时钟的频率下限为

- 标准模式：2 MHz
- 快速模式：4 MHz

**从模式工作流程**

1. 检测到起始信号之后，便会立即接收到 SDA 线上的地址，并将其存放在移位寄存器中。之后会将其与接口地址或者广播呼叫地址比较（10 位地址模式下需要包含头序列 11110xx0，其中 xx 表示该地址的两个最高有效位）
2. 头或地址不匹配：接口会忽略并且等待下一个起始信号
3. 头匹配，只针对 10 位地址模式：如果 ACK 位置位，则接口会生成一个应答脉冲并且等待 8 位从地址
4. 地址匹配
    - 发送应答脉冲
    - ADDR 位会由硬件置位，并且在 ITEVFEN 位置位时生成一个中断
    - 如果 ENDUAL=1，则软件必须读取 DUALF 位的状态来核对那些从地址进行了应答
5. 在 10 位模式下，完成地址序列接收之后，从模式时钟处于接收模式。在接收到重复起始信号以及一个匹配地址位和最低有效位置位的头序列（11110xx1）之后，会进入发送模式
6. TRA 位指示从设备是处于接收模式还是处于发送模式

**从发送器**

1. 在接收到地址并且将 ADDR 清零之后，从设备会通过内部移位寄存器将 DR 寄存器中的字节发送到 SDA 线上
2. 从设备会延长 SCL 低电平时间，直到 ADDR 位清零并且 DR 寄存器中填满待发送的数据为止。然后进行数据的发送
3. 接收到应答脉冲时：TxE 位会由硬件置 1 并在 ITEVFEN 和 ITBUFEN 位均置 1 时生成一个中断
4. 如果在下一次数据传输结束之前 TxE 位已置 1 但某些数据尚未写入 IIC_DR 寄存器，则BTF 位会置 1，而接口会一直延长 SCL 低电平，直到通过软件对 IIC_SR1 读操作，以及对 `IIC_DR` 写操作后，把 BTF 清零为止

**从接收器**

1. 在接收到地址并将 ADDR 位清零后，从设备会通过内部移位寄存器接收 SDA 线中的字节并将其保存到 DR 寄存器。在每个字节接收完成后
    - 发出应答脉冲
    - RxNE 位会由硬件置 1 并在 ITEVFEN 和 ITBUFEN 位均置 1 时生成一个中断
2. 如果在下一次数据接收结束之前 RxNE 位已置 1 但 DR 寄存器中的数据尚未读取，则 BTF 位会置 1，而接口会一直延长 SCL 低电平，直到软件通过读取 `IIC_DR` 寄存器来把 BTF 清零

**关闭从设备通信**

传输完最后一个数据字节之后，主设备会生成一个停止位。接口会检测此条件并将 STOPF 位置 1 并在 ITEVFEN 位置 1 时生成一个中断

通过先读取 SR1 寄存器然后写入 CR1 寄存器的方式将 STOPF 位清零

### 主模式

在主模式下，IIC 接口会启动数据传输并生成时钟信号。串行数据传输始终是在出现起始信号时开始，在出现停止信号时结束。只要通过 START 位在总线上生成了起始信号，即会选中主模式

在主模式下需要

- 在 `IIC_CR2` 寄存器中对外设输入时钟进行编程，以生成正确的时序
- 配置时钟控制寄存器
- 配置上升时间寄存器
- 对 `IIC_CR1` 寄存器进行编程以便使能外设
- 将 `IIC_CR1` 寄存器的 START 置位以生成起始信号
- START 置位之后，接口会在 BUSY 位清零之后生成一个起始位并切换到主模式。在主设备下的接口会在当前字节传输结束后生成一个重复起始位
- 接下来从地址会通过内部移位寄存器发送到 SDA 线
    - 10 位寻址模式下：发送头序列，然后发送从地址，再次发送头序列来决定进入发送模式还是接收模式，最低有效位置位进入接收模式
    - 7 位寻址模式下：发送一个地址字节，最低有效位决定进入发送模式还是接收模式，最低有效位置位进入接收模式

**主发送器**

1. 在发送出地址并将 ADDR 清零后，主设备会通过内部移位寄存器将 DR 寄存器中的字节发送到 SDA 线
2. 主设备会一直等待，直到首个数据字节被写入 `IIC_DR` 为止
3. 接收到应答脉冲后，TxE 位会由硬件置 1 并在 ITEVFEN 和 ITBUFEN 位均置 1 时生成一个中断
4. 如果在上一次数据传输结束之前 TxE 位已置 1 但数据字节尚未写入 DR 寄存器，则 BTF 位会置 1，而接口会一直延长 SCL 低电平，等待IIC_DR 寄存器被写入，以将 BTF 清零
5. 通信结束：当最后一个字节写入 DR 寄存器后，软件会将 STOP 位置 1 以生成一个停止信号，接口会自动返回从模式（M/SL 位清零）

**主接收器**

1. 完成地址传输并将 ADDR 位清零后，IIC 接口会进入主接收模式。在此模式下，接口会通过
2. 内部移位寄存器接收 SDA 线中的字节并将其保存到 DR 寄存器。在每个字节传输结束后，接口都会依次：
    - 发出应答脉冲（如果 ACK 位置 1）
    - RxNE 位置 1 并在 ITEVFEN 和 ITBUFEN 位均置 1 时生成一个中断
3. 如果在上一次数据接收结束之前 RxNE 位已置 1 但 DR 寄存器中的数据尚未读取，则 BTF 位会由硬件置 1，而接口会一直延长 SCL 低电平，等待 IIC_DR 寄存器被写入，以将 BTF 清零
4. 通信结束：主设备会针对自从设备接收的最后一个字节发送 NACK。在接收到此 NACK 之后，从设备会释放对 SCL 和 SDA 线的控制。随后，主设备可发送一个停止位/重复起始位
    - 为了在最后一个接收数据字节后生成非应答脉冲，必须在读取倒数第二个数据字节后（倒数第二个 RxNE 事件之后）立即将 ACK 位清零
    - 要生成停止位/重复起始位，软件必须在读取倒数第二个数据字节后（倒数第二个 RxNE事件之后）将 STOP/START 位置 1
    - 在只接收单个字节的情况下，会在 EV6 期间（在 ADDR 标志清零之前）禁止应答并在
    - EV6 之后生成停止位。生成停止位后，接口会自动返回从模式（M/SL 位清零）

### 错误情况

- 总线错误：当 IIC 接口在传输地址或数据期间检测到外部停止位或起始位时，会出现此错误
- 应答失败：当接口检测到未应答脉冲会出现此错误
- 仲裁丢失：当 IIC 接口检测到仲裁丢失时会出现此错误
- 上溢/下溢错误：当时钟延长已禁止且 IIC 接口正在接收数据时，从模式中可能出现上溢错误。接口已经收到一个字节 (RxNE=1)，但是收到下一个字节之前 DR 中的数据未被读走

### 配置

![1723811086152.png](./1723811086152.png)

将 I2C 设置为 I2C，设置模式的话一般设置为主模式，参数设置如下

- I2C Speed Mode：I2C 速度模式
    - Standard Mode：标准模式
    - Fast Mode：快速模式
- I2C Clock Speed：I2C 时钟速度，这里设置为最大
- Clock No Stretch Mode：禁止时钟延长。clock stretching 通过将 SCL 线拉低来暂停一个传输，直到释放 SCL 线为高电平，传输才继续进行。clock stretching 是可选的，实际上大多数从设备不包括 SCL 驱动，所以它们不能 stretch 时钟
- Primary Address Length selection：从设备地址长度
    - 7 bit
    - 10 bit
- Dual Address Acknowledged：双地址确认
- Primary Slave Address：从设备初始地址
- General Call Address Detection：地址检测

```c
// I2C 主设备发送
HAL_StatusTypeDef HAL_I2C_Master_Transmit(I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 主设备接收
HAL_StatusTypeDef HAL_I2C_Master_Receive(I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 从设备发送
HAL_StatusTypeDef HAL_I2C_Slave_Transmit(I2C_HandleTypeDef *hi2c, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 从设备接收
HAL_StatusTypeDef HAL_I2C_Slave_Receive(I2C_HandleTypeDef *hi2c, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 向从机地址中写数据
HAL_StatusTypeDef HAL_I2C_Mem_Write(I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint16_t MemAddress, uint16_t MemAddSize, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 从从机地址中读数据
HAL_StatusTypeDef HAL_I2C_Mem_Read(I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint16_t MemAddress, uint16_t MemAddSize, uint8_t *pData, uint16_t Size, uint32_t Timeout);
// I2C 设备是否准备好了
HAL_StatusTypeDef HAL_I2C_IsDeviceReady(I2C_HandleTypeDef *hi2c, uint16_t DevAddress, uint32_t Trials, uint32_t Timeout);
```

## DWT 软件定时器

在 Cortex-M 里面有一个外设叫 DWT(Data Watchpoint and Trace)，是用于系统调试及跟踪，它有一个 32 位的寄存器叫 CYCCNT， 它是一个向上的计数器，记录的是内核时钟运行的个数，内核时钟跳动一次，该计数器就加 1，精度非常高，取决于内核的频率是多少

DWT 使用之前必须使能 DBG 的系统跟踪，DWT 的控制使能位在 DEMCR 寄存器的 bit24

### DEMCR

![DWT003.png](./DWT003.png)

### DWT_CYCCNT 寄存器

使能 `DWT_CYCCNT` 寄存器之前，先清 0。其基地址是 0xE0001004，复位默认值是 0，可读写类型。所以往 0xE0001004 这个地址写就将 `DWT_CYCCNT` 清 0 了

### DWT_CTRL 寄存器

![e24cb1ef08785c82245b9fcb40b48eff.png](./e24cb1ef08785c82245b9fcb40b48eff.png)

### 配置

使能过程

1. 先使能 DWT 外设，由内核调试寄存器 `DEM_CR` 的位 24 控制，写 1 使能
2. 使能 `CYCCNT` 寄存器之前，先清 0
3. 使能 `CYCCNT` 寄存器，由 `DWT_CTRL` 的位 0 控制，写 1 使能

### 代码实现

头文件

```c
#pragma once

#ifdef __cplusplus
extern "C" {
#endif

#include <stdint.h>

typedef enum {
  dwt_time_unit_s,
  dwt_time_unit_ms,
  dwt_time_unit_us,
} dwt_time_unit;

typedef struct dwt_time_typedef {
  uint32_t us;
  uint32_t ms;
  uint32_t s;
} dwt_time_typedef;

void dwt_init(uint32_t);
void dwt_update();
void dwt_delay(float, dwt_time_unit);
float dwt_get_delta_time(uint32_t *, dwt_time_unit);

#ifdef __cplusplus
}
#endif
```

源文件

```c
#include "bsp_dwt.h"
#include "stm32f4xx.h"

dwt_time_typedef dwt_timer;

static float cpu_freq[3];// s,ms,us
static uint32_t dwt_timer_round;
static uint32_t dwt_last_tick;

void dwt_init(uint32_t freq) {
  DWT->CYCCNT = (uint32_t) 0u;
  CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
  DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
  cpu_freq[0] = freq;
  cpu_freq[1] = freq / 1000;
  cpu_freq[2] = freq / 1000000;
  dwt_timer_round = 0;
}

void dwt_update() {
  volatile uint32_t now_tick = DWT->CYCCNT;
  if (now_tick < dwt_last_tick) ++dwt_timer_round;
  dwt_last_tick = now_tick;
  dwt_timer.s = now_tick / cpu_freq[0] + 25.565281517857142857142857142857 * dwt_timer_round;
  dwt_timer.ms = now_tick / cpu_freq[1] + 25565.281517857142857142857142857 * dwt_timer_round;
  dwt_timer.us = now_tick / cpu_freq[2] + 25565281.517857142857142857142857 * dwt_timer_round;
}

void dwt_delay(float delay, dwt_time_unit unit) {
  uint32_t tickstart = DWT->CYCCNT;
  while ((DWT->CYCCNT - tickstart) < delay * (float) cpu_freq[unit]);
}

float dwt_get_delta_time(uint32_t *last_tick, dwt_time_unit unit) {
  volatile uint32_t now_tick = DWT->CYCCNT;
  uint32_t delta_tick = now_tick - *last_tick;
  if (delta_tick < 0) delta_tick += UINT32_MAX;
  float delta_time = (float) delta_tick / cpu_freq[unit];
  *last_tick = now_tick;
  dwt_update();
  return delta_time;
}
```
