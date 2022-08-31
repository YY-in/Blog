---
title: 单片机原理
date: 2022-01-22 01:01:10
tags: stm32
categories: 硬件
description: 基于STM32对于单片机原理基础的术语描述
---
# 单片机原理

以STM32F103为芯片手册为基础，简单解释单片机的基本术语，配图出自洋桃电子

# 1.基本概念

微控制器=微处理器=单片机

## ARM

### ARM处理器

Acon RISC Machine，本身是32位设计，但也配备16位指令集，一般比等价位32位代码节省35%

|Cortex-M0|低成本和简单性能|
|---|---|
|Cortex-M3|性能效率|
|Cortex-M4|有效的数字信号控制|



- F0 低功耗：电池供电设备
- F1日常：工控、医疗类等中段产品
- F4 高性能：彩屏驱动

嵌入式设计：用最恰当的方案、合适的芯片、完成产品设计

### ARM公司

英国ARM公司是全球领先的半导体知识产权（IP）提供商。全世界超过95%的智能手机和平板电脑都采用ARM架构  。ARM设计了大量高性价比、耗能低的RISC处理器、相关技术及软件。2014年基于ARM技术的全年全球出货量是120亿颗，从诞生到现在为止基于ARM技术的芯片有600亿颗 [2]  。技术具有性能高、成本低和能耗省的特点。在智能机、平板电脑、嵌入控制、多媒体数字等处理器领域拥有主导地位。

## RISC-精简指令集

Reduce Instruction Set Computer 涉及到计算机原理

特点：

- 所有指令格式和周期相同
- 采用流水线技术

复杂指令集：把原来由转件实现的、常用的功能改用硬件指令系统实现，以此提高计算机的执行速度

精简指令集：尽量简化计算机指令功能，只保留那些功能简单、能在一个节拍器中完成的指令，而把较复杂的功能用一段子程序实现。以此使得指令的平均执行周期减少，从而提高计算机的工作主屏，同时使用大量通用寄存器提高子程序执行的速度。

## 32位处理器

计算机中位数指的是CPU一次能处理的最大位数。1bit=8 byte

位数带来的差异：

- 运算速度差异：一次指令可以进行更多的计算
- 内存差异：增加内存寻址，类比电话号码，位数越多所查找的内存越大

# 2.STM32

[](https://www.st.com/content/st_com/zh.html)[ ST意法半导体官网](https://www.st.com/content/st_com/zh.html)

![STM32产品](http://inews.gtimg.com/newsapp_ls/0/14407754419/0)

![STM32命名规范](http://inews.gtimg.com/newsapp_ls/0/14407780611/0)

# 3.内核与存储器

 [STM32F103X8-B 中文数据手册](https://oneindex.yyin.top/share/pdf/STM32F103X8-B数据手册（中文）-20150727-CD00161566_ZHV10.pdf)

 [STM32F103X8-B 英文数据手册](https://oneindex.yyin.top/share/pdf/STM32F103X8-B%E6%95%B0%E6%8D%AE%E6%89%8B%E5%86%8C%EF%BC%88%E8%8B%B1%E6%96%87%EF%BC%89-20160119-CD00161566_ENV17.pdf)

![](http://inews.gtimg.com/newsapp_ls/0/14408393411/0)

![存储过程](https://www.hualigs.cn/image/61dcea6c67cef.jpg)

# 4.时钟、复位和电源管理

时钟：给单片机提供一个基准时间脉冲 ，它会产生一个方波信号，在这个信号之下每一个 方波周期之内会运行一条指令，时钟越快频率越高单片机的速度越快

内部RC震荡器：由内部电容电阻产生的振荡器，高速时钟提供给ARM内核的系统时钟，低速时钟提供给外部RTC

外部晶体振荡器：石英晶振，体积较大，无法集成在单片机内部

PLL：锁相环，产生分频，将一个频率切分成几个，以此达到倍频的效果，用于调节单片机的工作频率

RTC振荡器：实时时钟

![时钟类型与对比](https://www.hualigs.cn/image/61dcf1872508a.jpg)

![时钟与系统工作流程](https://qiniu.yyin.top/20220120193656.png)

# 5.低功耗和ADC

## 1.低功耗

1. 低功耗模式：睡眠、停机、待机
	|工作模式|关掉功能|唤醒方式|
	|---|---|---|
	|睡眠模式|ARM内核|所有内部、外部功能的中断/事件|
	|停机模式|ARM内核          内部所有功能    PLL分频器、HSE      |外部中断输入接口EXIT（16个I/O之一）   电源电压监控中断PVD                                         RTC闹钟到时                                             USB唤醒信号            |
	|待机模式|ARM内核                      内部所有功能                                  PLL分频器、HSE   SRAM内容消失|NRST接口的外部复位信号（外部信号）                                  独立看门狗IWDG复位                                           专用唤醒WKUP引脚                                            RTC闹钟到时|
	
	
2. Vbat:单片机上的电池接口，为RTC实时时钟提供走时的电能，为后备寄存器供电，时寄存器的数据不会丢失

## 2.ADC(数模转换器)

ADC可以实现单次或扫描转换，在扫描模式下，自动进行在选定的一组模拟输入上的转换。

ADC可以使用DMA（独立数据转存）操作

![普通模式：采集一个外部电压值，CPU通过程序向ADC发送指令、让ADC初始化采集数据，这时候ADC采集的数据将会发送给ARM内核，ARM内核收到值后，存放到SRAM。但转换频繁，CPU不断工作。                       DMA模式：解放CPU，让CPU空闲下来处理其他事情，将ADC的值直接通过DMA通道，放入SRAM中](https://qiniu.yyin.top/20220120200724.png)

# 6.DMA和IO端口

DMA可以管理存储器到存储器、设备到存储器、和存储器到设备的数据传输;

DMA控制器支持环形缓冲区管理，避免了控制器传输到达缓冲区结尾时所产生的中断

每个通道都有专门的硬件DMA请求逻辑，同时可以由软件触发每个通道；传输长度、传输源地址和目标地址都可以通过软件单独设置

DMA可以用于主要的外设：SPI、i2c、USART、通用、基本和高级控制定时器TIMx和ADC



![](https://qiniu.yyin.top/640)

![](https://qiniu.yyin.top/fbfebd1c82a4dd532b2c212299f59c30.jpg)

重点：除了具有模拟功能的端口之外，所有GPIO端口都能与其他功能共用，都有大电流通过的能力

![](https://qiniu.yyin.top/20220121005508.png)

![](https://qiniu.yyin.top/08c39db1a17d1c6fd0fa0583dd68d0ba.png)

![](https://qiniu.yyin.top/11b31dc21695c9b42e395660d2ca284a.png)

模拟输入：断开下拉电阻和上拉电阻以及输出口，只保留输入模拟值

浮空输入：断开下拉电阻和上拉电阻以及输出口，只保留输入数字值

下拉输入：上拉就是将不确定的信号通过一个电阻钳位在高电平，电阻同时起限流作

上拉输入：下拉电阻是直接接到地上，接二极管的时候电阻末端是低电平

推挽输出：输出大电流，可以驱动外部元器件，但是没法输入

开漏输出：输出电平但没有太大推动力

![](https://qiniu.yyin.top/30b3c3d7d71f5a5bce07fa9e8d80310b.jpg)

黑点左上正向

7：复位

8 VSSA：电源负极

9 VDDA：电源正极

44：BOOT0 启动

# 7. 调试模式和定时器

## 1.调试模式

![](https://qiniu.yyin.top/1b232f6d1678f93dfd0c05248fe30abb.jpg)

![简化调试说明](https://qiniu.yyin.top/2d2ec6d34b866a64b64e268c16d44a9b.jpg)

## 2. 定时器

![](https://qiniu.yyin.top/edd9e9d138348d322ff8fa6a2a85b46e.png)

![](https://qiniu.yyin.top/微信图片_20220121022141.jpg)

![](https://qiniu.yyin.top/af7e7164e94a0688588eec4096bcee44.jpg)

看门狗：表示一个独立的定时器，对CUP进行监控，如果单片机出现任何意外的情况，比如CUP程序出现错误，或者CUP的电压过低，看门狗就会使单片机复位，是单片机回到第一条程序进行执行，使得从错误中跳离出来。

看门狗独立的计时，CUP必须每个一段时间让程序计时器清零，程序会让看门狗复位，这就是喂狗

![](https://qiniu.yyin.top/9cbecbeaa85cd83754ba88be7dc02722.png)

![](https://qiniu.yyin.top/197117973a6742086689df561bea1536.png)

用户可以自己确定使用哪一个时钟

![](https://qiniu.yyin.top/c87880ccab2523763d29285194e453c8.png)

分时处理，高速切换任务

利用滴答定时器，隔离这些任务

![](https://qiniu.yyin.top/20220121231428.png)

普通定时器：用户程序中的精准定时；

高级定时器：处理精准定时，还可以做电机控制：

窗口看门狗：专用的看门狗定时器或者普通的定时器

独立看门狗：高稳定性的看门狗定时器，给单片机产生复位信号

# 8. I2c和 USART接口

## 1.I2C总线

总线：一个主设备挂载多个从设备，通过一条通信线通信

主模式：单片机，主动发送指令的一方

从设备：设备，被动接受指令发送数据的一方

标准和快速模式：通信协议

![](https://qiniu.yyin.top/31812a0aa3aeaa2abfbb2711c2c04a29.jpg)

SCL：时钟同步线

SDA：数据传输线

通过地址的方式完成设备通信

## 2.通用同步/异步收发器(USART)

![](https://qiniu.yyin.top/e5055fe9651fdf4238e4d413ef526c66.jpg)

![单片机之间的通信](https://qiniu.yyin.top/0793258d9f5d91e9779f892562ee8c78.jpg)

# 9.SPI、CAN、USB接口

## 1.SPI接口

![](https://qiniu.yyin.top/f8d4014c9be551728709bb4132639359.jpg)

SD卡和TF卡就是SPI接口的

![SPI设备电连接图](https://qiniu.yyin.top/e74248ab4494e014b7a34fb24cb5c115.jpg)

除了CS接口，其他都可以并联

与I2C地址查询不同，SPI设备粗暴通过单独的接口完成通信，通过电平切换设备
我们想和那个接口通信就要将那个接口设置成低电平，低电平启动，高电平禁止

优点：通信更快

缺点：设备数量限制

## 2.CAN接口

![](https://qiniu.yyin.top/b93fcf8b486a644e153a4b74b942730e.jpg)

协议很复杂，但功能强大，很稳定

![CAN设备连接关系图](https://qiniu.yyin.top/1e04074a9ca62e3ca5df5b6497b4ee88.jpg)

## 3.USB接口

比SPI慢

![](https://qiniu.yyin.top/cd3f5d95d508598767a589dd82645799.jpg)

![](https://qiniu.yyin.top/36e5d99d1cc3dbfe288ce6f6ddbe330f.png)

协议非常复杂

# 10. CRC校验和芯片ID 

单片机的辅助功能

## 1.CRC计算单元



![](https://qiniu.yyin.top/cf571eed38fee559fda0ed6e93b0e386.jpg)

![](https://qiniu.yyin.top/bf4f288b3df1808601722e892f87a297.jpg)

## 2.芯片ID

![](https://qiniu.yyin.top/a2effe764bafb87df3b6a22289cf5465.jpg)

- 用来作产品序列号
- 用来作密码，提高安全性
- 用来保护程序的不可复制

# 11.总结

![](https://qiniu.yyin.top/5f1d04007d58ca0262acb9767108626b.jpg)

![](https://qiniu.yyin.top/2cde956414b192cc37bfd61152e0a7de.jpg)

中断：单片机内核在正常处理任务，如果外部来了信号，我们就需要中断当前任务去处理其他任务，在中断过多时，中断控制器可以让中断任务分级，让CPU按照用户自定义的顺序处理中断

![](https://qiniu.yyin.top/3a25894d61b65b08e5dc77c49ffdfe4d.jpg)

![](https://qiniu.yyin.top/219e11c654cc60f79144cde1f715920d.jpg)

APB2：高速外围总线

APB1：低速外围总线

![](https://qiniu.yyin.top/20220121152131.png)

![](https://qiniu.yyin.top/20220121152338.png)

# 12.最小系统板

![](https://qiniu.yyin.top/20220121231338.png)

![](https://qiniu.yyin.top/3781c0f7c6cbf686fea270140c0aff22.jpg)
















