---
title: '内部温度实验'
sidebar_position: 12
---

# 内部温度实验

## 前言

上一章实验讲解了ADC类，本章将学习ADC的一个专用通道，用于读取芯片内部的温度信息。通过本章的学习，读者将学习到使用ADC类读取芯片内部信息。

## ADC介绍

### 1，ADC 简介

有关ADC的介绍，请见[ADC实验的ADC介绍](adc.md#adc介绍)

## ADC模块介绍

有关ADC模块的介绍，请见[ADC实验的ADC模块介绍](adc.md#adc模块介绍)

## 硬件设计

### 例程功能

1. 启用ADC3引脚的GPIO29的ADC功能，通过该引脚可以读取芯片内部输出温度的模拟值，通过这个模拟值通过计算便可转化为温度值。

### 硬件资源

1. 正点原子1.14寸SPI LCD模块

    LCD_BL - GPIO25
   
    LCD_DC - GPIO8
   
    SPI_MOSI - GPIO11
   
    SPI_SCK - GPIO10
   
    LCD_CS - GPIO9

2. ADC

   ADC3 - CPIO29

### 原理图

本章实验内容，主要讲解RP2350A内部ADC模块的使用，无需关注原理图。

##  实验代码

``` python
from machine import ADC,Pin,Timer
import LCD,time

"""
 * @brief       定时器中断回调函数
 * @param       无
 * @retval      无
"""
def tick(timer):
    global led
    led.toggle()
    

"""
 * @brief       ADC取平均值
 * @param       times：次数
 * @retval      返回：ADC平均值
"""
def adc_get_result_average(times):
    
    temp_val = 0
    
    for i in range(0,times):
        temp_val += adc.read_u16()
        
    return temp_val / times

"""
 * @brief       程序入口
 * @param       无
 * @retval      无
"""
if __name__ == '__main__':
    lcd = LCD.LCD_init()
    adc = ADC(4)
    led = Pin("LED", Pin.OUT)
    tim = Timer()
    
    tim.init(freq=1, mode=Timer.PERIODIC, callback=tick)
    lcd.string("ATK-DNRP2350AM", 10, 10, lcd.red)
    lcd.string("ADC TEST", 10, 30, lcd.red)
    lcd.string("ATOM@ALIENTEK", 10, 50, lcd.red)
    while True:
        # 读取ADC值
        adcdata = adc_get_result_average(20)
        # 转换为电压值
        number = float(adcdata * (3.3 / 65535))
        temperature = 27 - (number - 0.706)/0.001721
        print("TEMP:{} ".format(temperature))
        lcd.hcolor_fill(10, 70, 20 * 8, 16, lcd.white)
        lcd.string("TEMP:{} ".format(temperature), 10, 70, lcd.red) # 显示芯片内部温度值
        lcd.display()
        time.sleep(1)
```

可以看到，本章实验和上一章非常相似，函数adc_get_result_average(times)和上一章讲解的一样，用于获取times次ADC采集的平均值，同时还声明了函数tick，用于指示系统运行，接着在main函数中创建lcd、adc、led对象，最后在一个while循环不断转换芯片内部输出的电压值，并通过计算将其转化为温度值，并在屏幕上显示。

## 运行验证

将DNRP2350AM开发板连接到Thonny，然后添加需要运行的实验例程，并点击Thonny左上角的“运行当前脚本”绿色按钮后，此时，可以看到LED灯不停闪烁，屏幕上显示ADC采集到芯片内部的温度值，这和理论结果一致。

