---
title: '定时器实验'
sidebar_position: 5
---

# 定时器实验

## 前言

本章将介绍machine模块中的Timer类，即定时器类。通过本章的学习，读者将学习到machine模块中Timer类的使用。

## Timer模块介绍

### 概述

RP2350A上的系统定时器外设为系统提供微秒级时间基准，并基于此时基生成中断。RP2350A包含两个系统定时器实例：TIMER0和TIMER1。这使得两个独立控制的定时器可分别部署于不同的安全域中。

特性            | 说明
|---------------------|-----------
计数器架构  | 64位计数器，每微秒递增一次
寄存器读取机制 | 通过一对锁存寄存器实现无竞争读取（支持32位总线操作）
中断触发系统  | 4个独立警报器，匹配计数器低32位值后触发IRQ中断

### API描述

Timer类位于machine模块下

#### 构造函数

```python
timer = Timer()
```

构建一个timer对象，可以不传任何参数

#### init

```python
Timer.init(mode=Timer.PERIODIC, freq=1, period=-1, callback=None, tick_hz=1000000)
```

初始化定时器参数

【参数】

- mode：运行模式，单次或周期，可选参数
- freq：Timer运行频率，支持浮点，单位Hz，可选参数，优先级高于`period`
- period：Timer运行周期，单位ms，可选参数
- callback：超时回调函数，必须设置，要带一个参数
- tick_hz：定时器频率，可选参数

【返回值】

无

更多用法请阅读MicroPython官方API手册：

https://docs.micropython.org/en/latest/library/machine.Timer.html#machine.Timer

## 硬件设计

### 例程功能

1. 创建一个超时周期为1000毫秒的周期定时器，并再其超时回调函数中控制LED灯切换亮灭状态

### 硬件资源

1. LED

   ​	LED - GPIO3

### 原理图

本章实验内容，主要讲解Timer模块的使用，无需关注原理图。

##  实验代码

``` python
from machine import Pin, Timer

"""
 * @brief       定时器中断回调函数
 * @param       无
 * @retval      无
"""
def tick(timer):
    global led
    led.toggle()

"""
 * @brief       程序入口
 * @param       无
 * @retval      无
"""
if __name__ == '__main__':
    led = Pin("LED", Pin.OUT)
    tim = Timer()
    tim.init(freq=1, mode=Timer.PERIODIC, callback=tick)
    
    while True:
        pass
```

可以看到，首先构建一个LED对象和定时器对象，然后初始化定时器超时时间为1秒、周期性运行、回调函数为tick，最后进入一个空循环。

当发生超时中断时，系统进入到tick函数，将led灯翻转一次。

## 运行验证

将DNRP2350AM开发板连接到Thonny，然后添加需要运行的实验例程，并点击Thonny左上角的“运行当前脚本”绿色按钮后，此时，可以看到板载的LED每隔1秒亮灭一次，这是因为在定时器中断每隔1秒会将LED灯的引脚翻转一次，这与理论推断的结果一致。
