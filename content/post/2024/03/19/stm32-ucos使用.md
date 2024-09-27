---
title: "stm32-ucos使用"
date: 2024-03-19 09:55:32
categories: "单片机"
tags: ["stm32f4", "ucosii"]
summary: "基于stm32系列的操作系统ucosii的一些基本使用"
---

### 计算机操作系统

计算机只由硬件构成的叫做“裸机“，不能工作，必须有软件

1. 多任务
    
    把一个大任务分解为几个小任务，那在一个任务需要等待I/O时就可以交出CPU的使用权去运行其他的任务，可以极大的提高CPU的利用效率。
    
2. 内核类型
    - 可剥夺型内核
        
        总是运行优先级别最高的任务，即使CPU正在运行某一个低优先级的任务，当高优先级的任务准备就绪时，就会剥夺低优先级的任务的CPU的使用权。
        
    - 不可剥夺型内核
        
        总是优先级别高的任务最先获得CPU的使用权，要求每个任务都能主动放弃CPU的使用权。
        
3. 任务切换时间
    
    多任务系统的任务之间的切换是需要时间的。操作系统的任务调度器就是做这项工作的。调度器在进行任务切换时需要一段时间，这段时间的长短也影响系统实时性，任务调度器进行任务切换所用的时间不能受到应用程序中其他因素（任务数目等）的影响。
    
4. 终端延时
    
    外部事件的发生会以中断申请信号的形式通知CPU，然后才运行中断服务程序来处理该事件，自CPU响应中断到CPU转向中断服务程序之间所用的时间叫做终端延时，也影响系统的实时性。
    

### ucosii使用

1. 用户应用程序的结构
    
    ```c
    void task1(void *pdata);
    void task2(void* pdata);
    ...
    void main()
    {
    		OSInit(); // 初始化ucosii
    		OSTaskCreate(task1,....); // 创建任务
    		OSStart(); // 启动任务
    }
    ```
    
    使用 `OSStart()`函数启动各项任务之后，任务就交给操作系统管理和调度了。
    
2. 系统任务
    1. 空闲任务
        
        为了使CPU在没有用户任务可执行的时候有事可做，ucosii提供了一个空闲任务的系统任务，用户可以对空闲任务进行增加操作等，并且一个用户的应用程序必须使用这个空闲任务，不能用软件删除。
        
        ```c
        void OS_TaskIdle(void* p_arg){
        		#if OS_CRITICAL_METHOD == 3u 
        				OS_CPU_SR  cpu_sr = 0u; // 为CPU状态寄存器分配存储空间
        		#endif
        
            p_arg = p_arg; // 不使用会出现警告，有些编译器出现报错
            for (;;) {
                OS_ENTER_CRITICAL();// 进入临界区，关闭所有中断
                OSIdleCtr++;
                OS_EXIT_CRITICAL(); // 开放中断
                OSTaskIdleHook(); // 调用用户的任务的HOOK
            }
        }
        ```
        
    2. 统计任务
        
        系统的统计任务 `OSTaskStat()` 可以每秒计算一次CPU在单位时间内被使用的时间，并且把结果以百分比的形式存放在变量 `OSCPUUsage` 中
        
        用户是否使用这个统计任务，可以根据应用程序的实际需要选择，如果要使用就必须把定义在头文件 `OS_CFG.h` 中的系统配置常数 `OS_TASK_STAT_EN` 设置为1，并且在创建统计任务之前调用函数 `OSStatInit()` 对统计任务初始化。
        
3. 任务优先级别和优先权
    
    ucosii的每个任务都必须具有一个唯一的优先级别。ucosii把任务的优先权分为64个优先级别，每一个优先级别都用一个数字表示。数字0表示任务的优先级别最高，数字越大则表示任务的优先级别越高。
    
    通常程序的任务数小于64。用户可以在需要的时候在文件 `OS_CFG.h` 文件中通过给表示最低优先级别的常数 `OS_LOWEST_PRIO` 赋值来定义程序中优先级别的数目。一旦被定义就意味着系统中可以使用的优先级别为 `0 ~ OS_LOWEST_PRIO` 一共OS_LOWEST_PRIO+1个，也限制了程序的任务数量。系统总是把 `OS_LOWEST_PRIO` 自动给空闲任务，把 `OS_LOWEST_PRIO-1` 给统计任务（如果使用的话）。
    
4. 任务堆栈
    1. 任务堆栈的创建
        
        为了方便定义任务堆栈，文件 `OS_CPU.h` 中专门定义了一个数据类型 `OS_STK` 
        
        ```c
        typedef unsigned int OS_STK; // 类型长度为16位 2个字节
        // 使用
        #define TASK_STK_SIZE 512  // 定义堆栈长度 1024 个字节
        OS_STK TaskStk[TASK_STK_SIZE]; // 定义一个数组作为任务堆栈
        ```
        
        创建任务函数
        
        ```c
        INT8U  OSTaskCreate (void   (*task)(void *p_arg), // 指向任务的指针
                             void    *p_arg, // 传递给任务的参数
                             OS_STK  *ptos, // 任务堆栈栈顶的指针
                             INT8U    prio) // 指定任务的优先级别参数
        ```
        
5. 任务创建
    
    程序通过函数 `OSTaskCreate()` 来创建一个任务
    
    函数对于创建任务的优先级别进行一系列判断，确定该优先级别合法并且未被使用之后，就调用函数 `OSTaskStkInit` 和 `OS_TCBInit` 对任务堆栈和任务的控制块进行初始化，初始化成功之后，把任务计数器加一之外还要判断ucosii的核心是否处于运行状态（`OSRunning == 1`），如果正在运行，调用 `OSSched()` 进行任务调度。
    
    ```c
    INT8U  OSTaskCreate (void   (*task)(void *p_arg),
                         void    *p_arg,
                         OS_STK  *ptos,
                         INT8U    prio)
    {
        OS_STK     *psp;
        INT8U       err;
    #if OS_CRITICAL_METHOD == 3u                 /* Allocate storage for CPU status register               */
        OS_CPU_SR   cpu_sr = 0u;
    #endif
    
    #ifdef OS_SAFETY_CRITICAL_IEC61508
        if (OSSafetyCriticalStartFlag == OS_TRUE) {
            OS_SAFETY_CRITICAL_EXCEPTION();
            return (OS_ERR_ILLEGAL_CREATE_RUN_TIME);
        }
    #endif
    
    #if OS_ARG_CHK_EN > 0u
        if (prio > OS_LOWEST_PRIO) {             /* Make sure priority is within allowable range           */
            return (OS_ERR_PRIO_INVALID);
        }
    #endif
        OS_ENTER_CRITICAL();
        if (OSIntNesting > 0u) {                 /* Make sure we don't create the task from within an ISR  */
            OS_EXIT_CRITICAL();
            return (OS_ERR_TASK_CREATE_ISR);
        }
        if (OSTCBPrioTbl[prio] == (OS_TCB *)0) { /* Make sure task doesn't already exist at this priority  */
            OSTCBPrioTbl[prio] = OS_TCB_RESERVED;/* Reserve the priority to prevent others from doing ...  */
                                                 /* ... the same thing until task is created.              */
            OS_EXIT_CRITICAL();
            psp = OSTaskStkInit(task, p_arg, ptos, 0u);             /* Initialize the task's stack         */
            err = OS_TCBInit(prio, psp, (OS_STK *)0, 0u, 0u, (void *)0, 0u);
            if (err == OS_ERR_NONE) {
                if (OSRunning == OS_TRUE) {      /* Find highest priority task if multitasking has started */
                    OS_Sched();
                }
            } else {
                OS_ENTER_CRITICAL();
                OSTCBPrioTbl[prio] = (OS_TCB *)0;/* Make this priority available to others                 */
                OS_EXIT_CRITICAL();
            }
            return (err);
        }
        OS_EXIT_CRITICAL();
        return (OS_ERR_PRIO_EXIST);
    }
    ```
    
6. 信号量和事件标志
    1. 信号量
        
        信号量是用于同步和互斥的工具。信号量的值只能是0或正整数。信号量的值表示当前可用的资源数。当进程占有一个资源时，信号量减1，当进程释放一个资源时，信号量加1。如果信号量的值已经为0，那么等待进程将会被阻塞，直到有一个进程释放了一个资源，将信号量的值加1。
        
        - 创建信号量
            
            ```c
            OS_EVENT *OSSemCreate(INT16U cnt); //信号量计数器初始值
            ```
            
        - 请求信号量
            
            ```c
            // 请求失败进入等待状态
            void  OSSemPend (OS_EVENT  *pevent,  // 信号量指针
                             INT32U     timeout, // 等待时间限制
                             INT8U     *perr) // 错误信息
            
            // 请求失败继续运行
            INT16U  OSSemAccept (OS_EVENT *pevent) // 信号量指针
            ```
            
        - 发送信号量
            
            ```c
            INT8U  OSSemPost (OS_EVENT *pevent) // 信号量指针
            ```
            
        - 删除信号量
            
            ```c
            OS_EVENT  *OSSemDel (OS_EVENT  *pevent, // 信号量指针
                                 INT8U      opt, // 删除条件选项
                                 INT8U     *perr) // 错误信息
            opt:
            1. OS_DEL_NO_PEND 等待任务表中没有等待任务时删除
            2. OSDEL_ALLWAYS 无论有无等待任务都删除
            ！只能在任务中删除信号量，不能在中断中删除
            ```
            
        - 查询状态
            
            ```c
            INT8U  OSSemQuery (OS_EVENT     *pevent,
                               OS_SEM_DATA  *p_sem_data) // 存储信号量状态的结构
            
            typedef struct os_sem_data {
                INT16U  OSCnt;                          /* Semaphore count                                         */
                OS_PRIO OSEventTbl[OS_EVENT_TBL_SIZE];  /* List of tasks waiting for event to occur                */
                OS_PRIO OSEventGrp;                     /* Group corresponding to tasks waiting for event to occur */
            } OS_SEM_DATA;
            ```
            
    2. 事件标志
        
        事件标志是一种进程同步工具，它可以用于同步进程之间的时间。事件标志的值只能为0或1。当事件标志的值为0时，等待进程将会被阻塞。当事件标志的值为1时，等待进程将会被唤醒，继续运行。事件标志主要用于等待某一个事件的发生，例如等待一个条件成立，或者等待一个时间的到来。
        
7. 时间管理
    
    ucosii提供了以下三种时间管理方法：
    
    1. 任务延迟
        
        函数 `OSTimeDly()` 可以使当前执行的任务进入延迟状态，延迟的时间是以系统时钟节拍为单位的。
        
    2. 任务延迟直到事件标志被设置
        
        函数 `OSTimeDlyHMSM()` 可以使当前执行的任务进入延迟状态，延迟时间是以时、分、秒、毫秒表示的，当设定的时间到达或事件标志被设置时，任务将会恢复执行。
        
    3. 获取系统时钟
        
        函数 `OSTimeGet()` 可以获取当前系统时钟的值，该值是以时钟节拍计数的。可以用于时间戳或超时计数等。
        
8. 任务通信与同步
    1. 队列
        
        队列是一种任务通信的机制，用于在多个任务之间传递数据。队列的大小是固定的，队列可以存储指定类型的数据，每个数据的大小必须相同。队列分为FIFO队列和优先级队列。
        
    2. 信号量和事件标志
        
        信号量和事件标志是用于同步和互斥的工具。信号量的值只能是0或正整数。信号量的值表示当前可用的资源数。当进程占有一个资源时，信号量减1，当进程释放一个资源时，信号量加1。如果信号量的值已经为0，那么等待进程将会被阻塞，直到有一个进程释放了一个资源，将信号量的值加1。事件标志是一种进程同步工具，它可以用于同步进程之间的时间。事件标志的值只能为0或1。当事件标志的值为0时，等待进程将会被阻塞。当事件标志的值为1时，等待进程将会被唤醒，继续运行。事件标志主要用于等待某一个事件的发生，例如等待一个条件成立，或者等待一个时间的到来。
        
    3. 互斥量
        
        互斥量是一种用于保护共享资源的机制。当一个任务获得了互斥量的所有权，其他任务将无法访问共享资源，只有当该任务释放互斥量的所有权时，其他任务才能再次访问共享资源。
        
9. 中断处理
    
    中断是一种异步事件，当外设发生中断时，CPU会暂停当前的任务，转而执行中断的处理程序。ucosii提供了中断服务例程（ISR）的支持，可以在ISR中使用ucosii的信号量、消息队列和事件标志等机制来完成各种任务。需要注意的是，中断服务程序应该尽量短小，以保证系统的实时性。
    
10. 常见问题解决
    1. 任务堆栈不够用
        
        如果任务的堆栈不够用，会导致任务运行异常，可以增加任务堆栈的大小或者减少任务中使用的局部变量的数量。
        
    2. 任务优先级别设置不当
        
        如果任务的优先级别设置不当，会导致低优先级的任务无法得到CPU的使用权，从而导致系统异常。可以通过调整任务的优先级别来解决该问题。
        
    3. 队列、信号量和事件标志使用不当
        
        如果队列、信号量和事件标志使用不当，会导致任务之间的同步和通信出现问题，可以通过检查代码来解决该问题。
        
11. 任务挂起和恢复
    
    ```c
    // 任务挂起函数
    INT8U OSTaskSuspend(INT8U prio)； // prio是挂起任务的优先级别
    																	// 参数为 OS_PRIO_SELF 时为挂起自身
    																	// 根据具体的情况返回报错信息
    /*
    OS_NO_ERR 无错误
    OS_TASK_PEND_IDLE
    OS_PRIO_INVALID
    OS_TASK_SUSPEND_PRIO
    */
    
    // 任务恢复函数
    INT8U  OSTaskResume (INT8U prio)；
    /*
    OS_NO_ERR 无错误
    OS_TASK_NOT_SUSPEND
    OS_PRIO_INVALID
    OS_TASK_RESUME_PRIO
    */
    ```
    
12. 任务调度器上锁和解锁
    
    调度器上锁解锁函数 `OSSchedLock (void)`， `OSSchedUnlock (void)` 用于禁止任务调度，让cpu执行当前任务保持cpu 的控制权，解锁后可以进行调度。
    
    实现原理很简单，对全局变量锁定嵌套计数器 `OSLockNesting` 进行操作， `OSLockNesting` 记录了上锁函数 `OSSchedLock (void)` 的调用次数， `OSSchedLock (void)` 中对变量进行加一操作， `OSSchedUnlock (void)` 对变量进行减一操作，在引起任务调度的函数中进行判断，若变量  `OSLockNesting` 的值大于0，说明任务调度上锁，进行任务调度的函数中进行判断 `if (OSLockNesting > 0u)` ，后 `return` ，不进行任务调度。但要满足一个条件，调用者不是中断服务子函数。
    
    上锁和解锁要成对使用！因为上锁以后系统就会被锁住，其他任务都不能运行，这些函数包括 `OSFlagPend` 、 `OSMboxPend` 、 `OSMutexPend` 、 `OSQPend` 、 `OSSemPend` 等。
    
    例如在延时函数中不再进行任务调度，当前有中断函数运行或任务锁,直接返回，当前有嵌套锁定,直接返回。
    
    ```c
    if (OSIntNesting > 0u) {   /* See if trying to call from an IS */
            return;}
        if (OSLockNesting > 0u) {  /* See if called with scheduler locked */
    return;}
    ```