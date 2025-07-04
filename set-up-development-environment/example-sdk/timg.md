---
title: '定时器实验'
sidebar_position: 6
---

# 定时器实验

## 前言

通用定时器可用于准确设定时间间隔、在一定间隔后触发（周期或非周期的）中断或充当硬件时钟。通过本章的学习，开发者将学习到通用定时器的使用。

## 定时器简介

RP2350A上的系统定时器外设为系统提供微秒级时间基准，并基于此时基生成中断。RP2350A包含两个系统定时器实例：TIMER0和TIMER1。这使得两个独立控制的定时器可分别部署于不同的安全域中。

特性	          | 说明
|---------------------|-----------
计数器架构  | 64位计数器，每微秒递增一次
寄存器读取机制 | 通过一对锁存寄存器实现无竞争读取（支持32位总线操作）
中断触发系统  | 4个独立警报器，匹配计数器低32位值后触发IRQ中断

## 硬件设计

### 例程功能

实现现象：程序运行后配置通用定时器，在一定的周期内触发报警事件。

### 硬件资源

1. LED：
     LED-GPIO3
2. 通用定时器：
	 TIMER0和TIMER1

### 原理图

本章实验使用的通用定时器为RP2350A的片上资源，因此没有对应的连接原理图。

## 程序设计

### Timer函数解析

PICO-SDK提供了一套API函数来配置通用定时器，开发者可以在```pico-sdk\src\common\pico_time```路径下找到相关的time.c和time.h文件。在time.h头文件中，你可以找到RP2350A的所有定时器函数定义。接下来，作者将介绍一些常用的定时器函数，这些函数的描述及其作用如下：

#### 添加报警回调函数

该函数用于在指定的毫秒数后触发一个回调函数。它基于 alarm_pool_add_alarm_in_ms 函数实现，并使用了默认的 alarm_pool。其函数原型如下所示：

```add_alarm_in_ms(uint32_t ms, alarm_callback_t callback, void *user_data, bool fire_if_past)```

【参数】

ms:延迟时间，单位为毫秒

callback:回调函数指针，当定时器到期时调用

user_data:传递给回调函数的用户数据，可以是任意类型的数据结构

fire_if_past:如果定时器时间已过，是否立即触发回调函数,1.true：立即触发 2.false：不触发

【返回值】

类型：alarm_id_t

说明：返回一个定时器ID，可用于后续取消定时器

#### 添加一个重复触发的定时器

该函数用于用于创建一个重复触发的定时器，时间间隔以毫秒为单位。它基于 alarm_pool_add_repeating_timer_us 函数实现，并使用了默认的 alarm_pool。其函数原型如下所示：

```add_repeating_timer_ms(int32_t delay_ms, repeating_timer_callback_t callback, void *user_data, repeating_timer_t *out)```

【参数】

delay_ms:延迟时间，单位为毫秒

callback:回调函数指针，当定时器到期时调用

user_data:传递给回调函数的用户数据，可以是任意类型的数据结构

out:输出参数，用于存储定时器的状态信息

【返回值】

类型：bool

说明：true：定时器创建成功

false：定时器创建失败（通常是由于资源不足）

### TIMER驱动解析

在SDK版本的06_timg例程中，作者在06_timg\BSP路径下新增了一个TIMG文件夹，用于存放timg.c和timg.h这两个文件。其中，timg.h文件负责声明GPTIM相关的函数和变量，而timg.c文件则实现了TIMER的驱动代码。下面，我们将详细解析这两个文件的实现内容。

#### 1,timg.h文件

```
/* 函数声明 */
void timg_init(void);
```

#### 2,timg.c文件

```
static struct repeating_timer timer;   /* 静态变量，确保生命周期 */

/**
 * @brief       定时器组回调函数
 * @param       t: 传入参数
 * @retval      默认返回1
 */
bool timer_callback(struct repeating_timer *t) 
{
    LED_TOGGLE();
    printf("Timer test\n");
    return 1;   /* 继续重复 */
}

/**
 * @brief       初始化定时器
 * @param       无 
 * @retval      无
 */
void timg_init(void) 
{
    if (!add_repeating_timer_ms(1000, timer_callback, NULL, &timer))
    {
        printf("Timer initialization failure！\n");
    }
}
```

对于大多数通用定时器使用场景而言，应在启动定时器之前设置警报动作，但不包括简单的挂钟场景，该场景仅需自由运行的定时器。

### CMakeLists.txt文件

打开本章节的实验（06_timg），在整个工程文件下包含了一个CMakeLists.txt文件（该文件的作用，作者已经在第四章中阐述过）。此文件的作用是将BSP文件夹下的驱动程序添加到构建系统中，确保在编译项目工程时能够调用这些驱动程序。关于该实验的CMakeLists.txt文件的具体内容与上一章节并没有什么太大的不同，因此不再赘述。

###  实验应用代码

打开main.c文件，该文件定义了工程入口函数，名为main。该函数代码如下。
```
/**
 * @brief       程序入口
 * @param       无
 * @retval      无
 */
int main()
{
    uint8_t key;

    stdio_init_all();           /* 初始化标准库 */
    led_init();                 /* 初始化LED */
    timg_init();                /* 初始化定时器 */

    while (1) 
    {
        tight_loop_contents();  /* 空循环，保持程序运行 */
    }
}
```
本实验的实验代码很简单，在完成初始化后，执行一个空循环，以减少CPU空闲时间，保证程序正常运行。

## 下载验证

在完成编译和烧录操作后，可以看到板子上的LED在闪烁，在一定周期内串口打印输出定时器报警事件。


